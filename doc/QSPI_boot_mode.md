# QSPI boot mode adjustments to Petalinux

## Prerequisites

- Supported Linux OS for Petalinux 2020.2
- Petalinux 2020.2 installation and all required packages (for more details please refer to [UG1144])
- Reference Design documentation for the hardware combination in use (module and baseboard combination)

## Adjusting project for QSPI boot mode

The equipped QSPI flash on Enclustra modules is typically 64 MBytes large. If the Petalinux project was created with the *_QSPI.bsp file, the following adjustments are already done and can be skipped. The default Petalinux project configuration needs the following adjustments:

1. Run `petalinux-config`
2. Under `Subsystem AUTO Hardware settings:`
    - `Flash Settings`: create all partitions as needed, e. g.
        | partition name | partition size  | partition size in MB |
        |----------------|-----------------|----------------------|
        | boot           | 0x1f00000       | 31                   |
        | bootenv        | 0x80000         | 0.5                  |
        | bootscr        | 0x80000         | 0.5                  |
        | kernel         | 0x1000000       | 16                   |
        | rootfs.jffs2   | 0x1000000       | 16                   |
    - Under `Advanced bootable images storage Settings`:
        - `boot image settings -> images storage media`: set to `primary flash`
        - `kernel image settings -> image storage media`: set to `primary flash`
    - Note that the default Xilinx kernel is too big for this specific configuration and a reduced kernel configuration is used to fit the kernel onto the QSPI memory.
3. Run `petalinux-config -c u-boot` and set `ARM architecture -> Boot script offset` corresponding to your offset address. The offset address can be found in the Petalinux project folder `components/plnx_workspace/device-tree/device-tree/system-conf.dtsi` under the dts entry for the `bootscr` partition. For the above example configuration the correct offset is `0x1f80000`.
4. Adjust the values for the kernel offset and the FIT image size in `project-spec/meta-user/recipes-bsp/u-boot/u-boot-zynq-scr.bbappend`:
    - `QSPI_KERNEL_OFFSET` according to your configuration, in this case `0x2000000`
    - `QSPI_FIT_IMAGE_SIZE` according to the size of your kernel partition, in this case `0x1000000`
5. Run `petalinux-build`
6. After the build is finished use `petalinux-package` command with the according flags to generate the boot components (assuming console is open in Petalinux project root dir):
    - for zynqMP modules:
        `petalinux-package --boot --fsbl images/linux/zynqmp_fsbl.elf --u-boot images/linux/u-boot.elf --pmufw images/linux/pmufw.elf --fpga images/linux/system.bit --force`
    - for zynq modules:
        `petalinux-package --boot --fsbl images/linux/zynq_fsbl.elf --fpga images/linux/system.bit --u-boot --force`

## Program QSPI flash

Please refer to the reference design documentation for your module and baseboard configuration to see the various methods of programming the QSPI flash memory. Additionally, Xilinx [UG1144] provides information for using tftpboot for programming the QSPI flash.

As an example Vivado is used below, which has some additional steps:
1. Files needed for programming:
    - BOOT.BIN
    - image.ub
    - boot.scr
2. Rename all files which do not have the `BIN` ending to `<filename>.bin` (e.g. `image.ub` -> `image.bin`).
3. Follow the steps from the reference design documentation for QSPI flash programming to set up Vivado correctly
4. Program each individual file in the following order:
    - BOOT.BIN with `Erase` and `Program` box selected and offset is `0`
    - bootscr.bin with `Program` box selected (remember to deselect `Erase`) and offset according to your partition configuration. In our example `0x01f80000`.
    - image.bin with `Program` box selected (remember to deselect `Erase`) and offset according to your partition configuration. In our example `0x2000000`.
5. After every partition has been programmed, the board should boot Petalinux successfully from QSPI.
6. The default Linux login for a Petalinux project is: **Username: root, Password: root**

As a simple script:
```
#!/bin/bash -ex
default_parameter="-flash_type qspi-x4-single -fsbl zynqmp_fsbl.elf -cable type xilinx_tcf url TCP:127.0.0.1:3121"
program_flash -f BOOT.BIN $default_parameter
program_flash -f boot.scr -offset 0x1f80000 $default_parameter
program_flash -f image.ub -offset 0x2000000 $default_parameter
```
[UG1144]: https://www.xilinx.com/support/documentation/sw_manuals/xilinx2020_2/ug1144-petalinux-tools-reference-guide.pdf