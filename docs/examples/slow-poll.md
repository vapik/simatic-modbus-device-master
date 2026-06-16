# Example
> Requirements: 
> Hardware: CPU 1214C - *6ES7 214-1AG40-0XB0*, CB1241 - *6ES7 241-1CH30-1XB0*
> 
> Software: TIA Portal V17 or higher
> 
> Download TIA-project: [link](/tia/generated/LVT_ModbusDeviceRtu_slow-poll_v0.0.1.zap17)

When multiple devices are on the same bus, a fault on one device will slow down the polling of all devices.
This example implements a mechanism called "slow-poll":
We read 1 register in 3 indentical devices. If a device fails to respond 3 times, a slowPollTimer is activated and polling is blocked until timer expires.

## Configuration
### Devices
The DB *ModbusRtuMaster_DevicesDb* contains array of devices *devices[0..2]*.
Every device contain request 1 (*requests[0]*): Read register 40001, place value in *device1.buffer[0]*.

![example read](/docs/images/slow-poll/example_read.png)

## Code
The main logic is located in the function block ***ModbusRtu_Master***

![example read](/docs/images/slow-poll/ModbusRtu_Master.png)
![subroutines](/docs/images/slow-poll/subroutines.png)

### Initializing the serial port
Initialization must occur once when the controller is started; for this, the *FirstScan* system flag is used.

If initialization is completed successfully, set token to poll the first device (i = 0)

```
REGION Serial port initialization
     IF "FirstScan" THEN
         #mbCommLoad.MODE := 4;
     END_IF;
    
     #mbCommLoad.RETRIES := #com.retries;
    
     #mbCommLoad(REQ := "FirstScan",
                 "PORT" := #com."port",
                 BAUD := #com.baud,
                 PARITY := #com.parity,
                 MB_DB := #mbMaster.MB_DB,
                 RESP_TO := #com.timeout,
                 DONE => #com.done,
                 ERROR => #com.error,
                 STATUS => #com.status);
    
     #initTrig(CLK := #com.done);
    
     IF #initTrig.Q THEN
         #i := 0;
     END_IF;
END_REGION
```

### Polling devices
To call instance of FB *"LVT_ModbusDeviceRtu_Master"* you need to send a single pulse to the *start* input.

```
#start := NOT #inst_LModbusDeviceRtu_Master.done;
```

Poll of device *devices[i]* starts.

When all requests is completed the output *done* will be set to *true* once.
We start copying data from *devices[i].buffer* to *DataDb.devices[i]* (FC *"ModbusRtu_Device_HandleData"*).

```
#data.param1 := #mdata.buffer[0];
```

If a request fail then *devices[i].requests[x].status.errCount* increased by 1.
We calculate amount of all counters, if the amount exceeds *ERRORS_THRESHOLD* we set *error* flag.

```
#statQuantityErrors := 0;
FOR #i := 0 TO #MAX_REQUEST_INDEX - 1 DO
    #statQuantityErrors += #mdata.requests[#i].status.errorCounter;
END_FOR;

// Set error status
#data.error := #statQuantityErrors >= #ERRORS_THRESHOLD;
```

Set the next token (*i := i + 1*).

```
IF #inst_LModbusDeviceRtu_Master.done THEN
     "ModbusRtu_Device1_MapData"(mdata := "ModbusRtuMaster_DevicesDb".device1,
                                 data := "DataDb".device);
     #i := #i + 1;
END_IF;
```

### Slow poll
If flag *error* is true then *slowPollTimers[j]* is started and flag *allowPolles[i]* is false.
```
REGION Slow poll
    
    FOR #j := 0 TO #MAX_DEV_INDEX DO
        
        // Update Slow Poll timers
        #slowPollTimers[#j](IN := "DataDb".devices[#j].error,
                            PT := #SLOW_POLL_TIME);
        
        // Polling is allowed if there are no error or timer is ended.        
        #allowPolles[#j] := NOT ("DataDb".devices[#j].error AND #slowPollTimers[#j].Q);
        
    END_FOR;
    
END_REGION
```

If flag *allowPolles[i]* is false then we don't call FB *"LVT_ModbusDeviceRtu_Master"* and skip the step until timer *slowPollTimers[j]* finishes.

```
IF #allowPolles[#i] THEN
            #stat_LModbusDeviceRtu_Master(...);
            ...            
        ELSE
            #i := #i + 1;
        END_IF;
```

Tokens have run out, set the initial one and start the process again
```
     ELSE
         #i := 0;
```
