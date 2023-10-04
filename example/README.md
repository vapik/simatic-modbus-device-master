# Example for LMBMD_ModbusDeviceRtu
## Read holding registers from Device1 and write the data to Device2

In the example we read 2 holding registers (40001, 40002) from Device1 (addr: 1), map the data to local Db, write the data to Device2 (40003, 40004).
TIA V15 project: [link](about:blank)

## Hardware
CPU1214C DC/DC/DC: 6ES7 214-1AG40-0XB0

CB 1241 (RS485): 6ES7 241-1CH30-1XB0

## Descrition
### Code
- Main.scl [OB] - OB1 organization block;
- ModbusRtu_Master.scl [FB] - Init serial port, send requests, mapping data;
- ModbusRtu_Device1_MapData.scl [FC]- Copy data from "Device Map Memory" (device1.buffer.readBuffer) to local Db (DataDb.device);
- ModbusRtu_Device2_MapData [FC] - Copy data from local Db (DataDb.device) to (device2.buffer.writeBuffer).

### Data
- DataDb.db [DB] - "Device" data;
- inst_ModbusRtu_Master.db [DB] - instance of ModbusRtu_Master;
- ModbusRtuMaster_DeviceDb.db [DB] - the block have 2 "LMBMD_ModbusDeviceRtu_Data" structues: device1, device2;
- Modbus_Comm_Load_DB.db [DB] - instance of "Modbus_Comm_Load";
- Modbus_Master_DB.db [DB] - instance of "Modbus_Master".

### Types
- DeviceData.udt - structure of device data.

> Don't forget to add "LMBMD" library from "src" folder to TIA Portal project