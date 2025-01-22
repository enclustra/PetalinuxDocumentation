# Known issues

## 1. USB host not working on PE1 base boards

### Affected modules:
All variants and revisions of the following modules are affected:
- Mercury+ XU1
- Mercury XU5
- Mercury+ XU6
- Mercury+ XU61
- Mercury+ XU7
- Mercury+ XU8
- Mercury+ XU9

### Description
USB PHY 0 is enabled by default in all reference designs and Linux BSPs but it can not be used in combination with PE1 base boards. The USB data signals are routed from the module to the USB hub device on the PE1 base board through multiplexers. The USB_ID signal controlling one of the multiplexer devices is routed to the USB 1 PHY which affects the signal.

### Workaround
If USB is required, the Vivado design and Linux devicetree need to be modified to enable USB PHY 1. On some modules USB PHY 1 and ETH 1 share the same MIO pins and therefore, ETH 1 is not available when using USB 1.

## 2. USB initialization problems with Zynq 7000 modules

### Affected modules:
All variants and revisions of the following modules are affected:
- Mercury ZX1
- Mercury ZX5
- Mars ZX2
- Mars ZX3

### Description
There is no dedicated reset for the USB PHY. This can result in failures to initialize the USB interface.

### Workaround
An FSBL patch can be added to the Petalinux project as desribed in this [forum thread](https://adaptivesupport.amd.com/s/question/0D52E00006hpYs7SAE/144-usb-not-working-any-more?language=en_US).

## 3. U-Boot gets stuck while booting on modules with DS28 EEPROM

### Affected modules:
All revisions of modules equipped with a DS28 EEPROM are affected in combination with BSP files of release 2024.1:
- Mercury ZX1 R1
- Mercury ZX5 R1, R2
- Mars ZX3 R1 -R6

### Description
Because modules can have two different types of EEPROM devices, U-Boot always tries to read the MAC address from the atsha204a device as a first attempt.
If that is not successful, a second attemp is started to read the MAC address from the DS28 EEPROM.
The problem is that the atsha204a driver hangs in the wakeup sequence when the device does not answer to an I2C read request.

### Workaround
The U-Boot patch for the atsha204a driver needs to be replaced as described in following issue: https://github.com/enclustra/Mercury_ZX5_PE1_Reference_Design/issues/1 .
