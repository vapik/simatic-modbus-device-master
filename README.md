# Device-oriented Modbus RTU Master for S7-1200/S7-1500
The library show you how to organize polling modbus devices.
If you have several devices you don't need to code every request and its order, requests are contained in cfg structure for each device separately.
Function block **LMBMD_ModbusDeviceRtu_Master** make requests, put data to memory (different for each device), provide signal "done" when polling is finished.
You need only make an order for polling each device and make a function for mapping memory to inner blocks.
Take a look at **example** folder

>The detailded description will be sooon.



