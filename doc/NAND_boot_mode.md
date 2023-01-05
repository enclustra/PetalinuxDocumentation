# NAND boot mode adjustments to Petalinux

## Prerequisites

- Supported Linux OS for Petalinux 2022.1
- Petalinux 2022.1 installation and all required packages (for more details please refer to [UG1144])
- Reference Design documentation for the hardware combination in use (module and baseboard combination)

## Adjusting project for NAND boot mode

The equipped NAND flash on Enclustra modules is typically 512 MBytes large. No adjustments need to be made to the default Petalinux project configuration.

## Program NAND

Please refer to the reference design documentation for your module and baseboard configuration to see the various methods of programming the NAND memory. Additionally, Xilinx [UG1144] provides information for using tftpboot for programming the various memories. Further, have a look at [Zynq-7000: NAND Checklist](https://www.xilinx.com/support/answers/59311.html).  

The current BSP does not support booting directly from the NAND Flash memory. The BOOT.BIN including the FSBL, Bitstream and U-Boot has to be started from SD Card (MMC) or QSPI Flash (e.g. [QSPI_boot_mode.md](QSPI_boot_mode.md)). When U-Boot is booted it can load and boot the Linux system stored on the NAND Flash memory [EBE NAND flash](https://github.com/enclustra-bsp/bsp-xilinx/blob/master/documentation/4_Deployment.md#nand-flash).

This means that only the "image.ub" for programming to the NAND flash is left, which contains the kernel, rootfs in ramfs (rootfs only included in QSPI boot BSP) and the kernel device tree.

The following steps must be performed.  

1. Boot from QSPI (optionally SD), and interrupt the u-boot by hitting SPACE. Have a look at the detailed instructions: [QSPI boot mode](QSPI_boot_mode.md) or [SD boot mode](SD_boot_mode.md).  

2. Setup tftp-server and load the image.ub (FIT image) from the tftp server into the RAM. For example setup the tftp server on 10.1.10.222, the board then can be configured to have 10.1.10.100. on address 0x3000000.  

```
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

3. Switch the storage device to NAND, and write the image to NAND. Note: NAND always needs to be erased before writing! There is a NAND partition configured in the device tree, which we will use for the example.   

```
Zynq> zx_set_storage NAND
    Flash MIO pins configured to NAND mode

Zynq> nand device 0
Zynq> nand erase.part nand-linux
Zynq> nand write 0x3000000 nand-linux 0x2000000
    NAND write: device 0 offset 0x0, size 0x2000000
     33554432 bytes written: OK

```

4. If writing was successful, the NAND memory is persistent and the board could be powercycled. In our example we continue from a stopped u-boot shell and load the image.ub from NAND into RAM.  

```
Zynq> nand read 0x3000000 nand-linux 0x2000000
    NAND read: device 0 offset 0x2000000, size 0x2000000
     33554432 bytes read: OK
```

5. Boot into the loaded image in RAM.  

```
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

It is possible to switch BOOT mode to NAND, but is out of the scope of this provided Petalinux BSP. The basic steps are similiar. In addition some adjustments to the devicetree (partitioning), bootloader and the linux boot arguments for a rootfs in NAND will then be necessary.  


For further detailed instructions have a look at the user manuals:  
- Mercury ZX1 chapter [3.10 NAND Flash Programming](https://download.enclustra.com/public_files/SoC_Modules/Mercury_ZX1/Mercury_ZX1_User_Manual_V07.pdf)
- Mars ZX3 chapter [3.9 NAND Flash Programming](https://download.enclustra.com/public_files/SoC_Modules/Mars_ZX3/Mars_ZX3_User_Manual_V9.pdf)
- Mercury ZX5 chapter [3.10 NAND Flash Programming](https://download.enclustra.com/public_files/SoC_Modules/Mercury_ZX5/Mercury_ZX5_User_Manual_V06.pdf)

> **_NOTE:_** Please note that Vivado Hardware Manager does not support the NAND flash type equipped on the Enclustra modules.

[UG1144]: https://www.xilinx.com/support/documentation/sw_manuals/xilinx2022_1/ug1144-petalinux-tools-reference-guide.pdf
