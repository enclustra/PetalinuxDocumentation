# Known issues

## 1. USB host not working on PE1 base boards

### Affected modules:
All variants and revisions of the following modules are affected:
- Mercury+ XU1
- Mercury XU5
- Mercury+ XU6
- Mercury+ XU7
- Mercury+ XU8
- Mercury+ XU9

### Description
USB PHY 0 is enabled by default in all reference designs and Linux BSPs but it can not be used in combination with PE1 base boards. The USB data signals are routed from the module to the USB hub device on the PE1 base board through multiplexers. The USB_ID signal controlling one of the multiplexer devices is routed to the USB 1 PHY which affects the signal.

### Workaround
If USB is required, the Vivado design and Linux devicetree need to be modified to enable USB PHY 1. On some modules USB PHY 1 and ETH 1 share the same MIO pins and therefore, ETH 1 is not available when using USB 1.
