# NAND boot mode adjustments to Petalinux

## Prerequisites

- Supported Linux OS for Petalinux 2024.1 (refer to  [UG1144])
- All required packages as listed in the [Petalinux 2024.1 Release Notes](https://adaptivesupport.amd.com/s/article/000036178?language=en_US)
- Petalinux 2024.1 installation (refer to  [UG1144])
- Reference Design documentation for the hardware combination in use (module and baseboard combination)
- Successful Petalinux build as described in the [BSP usage documentation](BSP.md)

## Adjusting the project for NAND boot mode
The size of the NAND flash on Enclustra modules is 512 MBytes. No adjustments need to be made to the default Petalinux project configuration. The NAND partition layout is given below:

| partition name | partition offset | partition size  | partition size in MB |
|----------------|------------------|-----------------|----------------------|
| kernel         | 0x0              | 0x2000000       | 32                   |
| bootscr        | 0x2000000        | 0x100000        | 1                    |
| rootfs         | 0x2100000        | 0x1DF00000      | 479                  |

## Programming the NAND flash
The BSP does not support booting from the NAND flash memory directly. The initial boot has to be from either SD card or QSPI in order to load a `BOOT.BIN` including the FSBL, bitstream (optional) and u-boot.

Follow these steps to program the `image.ub` containing the ramfs, the kernel and the kernel device tree into the NAND flash:
1. Boot from SD card or QSPI and interrupt the boot process in u-boot by pressing **SPACE**.
2. Setup a tftp-server and load the FIT image (`image.ub`) from the tftp server into RAM. An example is given in the excerpt below:

```shell
Zynq> setenv ipaddr 10.1.10.100
Zynq> setenv serverip 10.1.10.222
Zynq> ping 10.1.10.222
    Using ethernet@e000b000 device
    host 10.1.10.222 is alive

Zynq> tftpboot 0x3000000 image.ub
    Using ethernet@e000b000 device
    TFTP from server 10.1.10.222; our IP address is 10.1.10.100
    Filename 'image.ub'.
    Load address: 0x3000000
    Loading: #################################################################
             #################################################################
             #################################################################
             #################################################################
             #################################################################
             #################################################################
             #################################################################
             #################################################################
             #################################################################
             #################################################################
             #################################################################
             #################################################################
             #######################
             4.2 MiB/s
    done
    Bytes transferred = 11783860 (b3ceb4 hex)
```

3. Switch the storage device to NAND and write the FIT image to NAND.

    > **_Note:_**  NAND flash needs to be erased before writing.

```shell
Zynq> zx_set_storage NAND
    Flash MIO pins configured to NAND mode

Zynq> nand device 0
Zynq> nand erase.part nand-linux
Zynq> nand write 0x3000000 nand-linux 0x2000000
    NAND write: device 0 offset 0x0, size 0x2000000
     33554432 bytes written: OK

```

4. If writing was successful, the NAND memory is persistent and the board can be powercycled. In this example we continue in the active u-boot shell and load the `image.ub` from NAND into RAM.

```shell
Zynq> nand read 0x3000000 nand-linux 0x2000000
    NAND read: device 0 offset 0x2000000, size 0x2000000
     33554432 bytes read: OK
```

5. Boot into the loaded image in RAM.  

```shell
Zynq> bootm 0x3000000
    ## Loading kernel from FIT Image at 03000000 ...
       Using 'conf-system-top.dtb' configuration
       Verifying Hash Integrity ... OK
       Trying 'kernel-1' kernel subimage
         Description:  Linux kernel
         Type:         Kernel Image
         Compression:  uncompressed
         Data Start:   0x030000fc
         Data Size:    3824992 Bytes = 3.6 MiB
         Architecture: ARM
         OS:           Linux
         Load Address: 0x00200000
         Entry Point:  0x00200000
         Hash algo:    sha256
         Hash value:   98a200fa083c2f7227244e50adf76172fddba30a9c11e00e5036bf90de87e0ee
       Verifying Hash Integrity ... sha256+ OK
    ## Loading ramdisk from FIT Image at 03000000 ...
       Using 'conf-system-top.dtb' configuration
    ...

```

> **_NOTE:_** Note that the Vivado Hardware Manager does not support the NAND flash type equipped on Enclustra modules.

[UG1144]: https://docs.amd.com/viewer/book-attachment/MVyApcmU3R9Mm97zSMBTWg/A1uhF~YnkvK0u6G775Tu_Q
