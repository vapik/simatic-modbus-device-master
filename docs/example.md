# Example
> Requirements: 
> Hardware: CPU 1214C - *6ES7 214-1AG40-0XB0*, CB1241 - *6ES7 241-1CH30-1XB0*
> 
> Software: TIA Portal V15.1 or higher
> 
> Download TIA-project: [link](/tia/generated/LMBMD_ModbusDeviceRtu_0.0.4.zap15_1)

In the example we read 3 registers from *Device1* and write these values to *Device2*.

Reading data from device *Device1* is performed in 2 requests: read 40001, read 40002,40003.

Writing to the *Device2* device is performed in one request: writing to 40003, 40004, 40005.

## Configuration
### Device1
The DB *ModbusRtuMaster_DevicesDb.device1* contains the query configuration for *Device1*.

Request 1 (*requests[0]*): Read register 40001, place value in *device1.buffer[0]*.

Request 2 (*requests[1]*): Read register 40002..40003, place values in *device1.buffer[1]..device1.buffer[2]*

![example read](/docs/images/example_read.png)

### Device2
The DB *ModbusRtuMaster_DevicesDb.device2* contains the query configuration for *Device2*.

Request 1 (*requests[0]*): Write from *device1.buffer[0]..device1.buffer[2]* to registers 40003..40005.

![example write](/docs/images/example_write.png)

## Code
The main logic is located in the function block ***ModbusRtu_Master***

![example read](/docs/images/ModbusRtu_Master.png.png)
![subroutines](/docs/images/subroutines.png)

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

### Poll Device1
To call instance of FB *"LMBMD_ModbusDeviceRtu_Master"* you need to send a single pulse to the *start* input.

```
#start := NOT #inst_LModbusDeviceRtu_Master.done;
```

Poll of device *Device1* starts.

When all requests is completed the output *done* will be set to *true* once.

We start copying data from *device1.buffer* to *DataDb.device* (FC *"ModbusRtu_Device1_MapData"*).

Set the next token (*i := i + 1*).

```
IF #inst_LModbusDeviceRtu_Master.done THEN
     "ModbusRtu_Device1_MapData"(mdata := "ModbusRtuMaster_DevicesDb".device1,
                                 data := "DataDb".device);
     #i := #i + 1;
END_IF;
```

```
//"ModbusRtu_Device1_MapData"
#data.param1 := #mdata.buffer[0];
#data.param2 := #mdata.buffer[1];
#data.param3 := #mdata.buffer[1];
```


### Poll Device2
Before starting polling of Device2, the data to be written is moved from *DataDb.device* to *device2.buffer* (FC *ModbusRtu_Device2_MapData*).

```
//"ModbusRtu_Device2_MapData"
#mdata.buffer[0] := #data.param1;
#mdata.buffer[1] := #data.param2;
#mdata.buffer[2] := #data.param3;
```

Poll of device *Device2* starts.

When all requests is completed the output *done* will be set to *true* once.

Set the next token (*i := i + 1*)

```
         IF #inst_LModbusDeviceRtu_Master.done THEN
             #i := #i + 1;
         END_IF;
```

Tokens have run out, set the initial one and start the process again
```
     ELSE
         #i := 0;
```
