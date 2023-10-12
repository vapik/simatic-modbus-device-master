# Modbus RTU Master device-oriented library for S7-1200/S7-1500
> Last version: v0.0.4

## Short description

The library is a wrap of internal library [Modbus_Master](https://support.industry.siemens.com/cs/mdm/109742272?c=86342756619&lc=en-WW).
 
In the library, requests are grouped for each network device separately, and devices are sequentially polled.
Each network device has a local data buffer.

If you need to poll several devices, this library will help you reduce the amount of code, organize the polling order.

Before diving into the library I recommend to read: 
Description of *Modbus_Master*: [Overview of the Modbus RTU communication (S7-1200, S7-1500)](https://support.industry.siemens.com/cs/mdm/109742272?c=58089698955&lc=en-WW)

Example: [link How do you establish MODBUS-RTU communication?](https://support.industry.siemens.com/cs/attachments/47756141/47756141_Description.pdf)

## Contents

- [Example](/docs/example.md)
- [License](/docs/license.md)
