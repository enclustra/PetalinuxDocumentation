# NAND boot mode adjustments to Petalinux

## Prerequisites

- Supported Linux OS for Petalinux 2020.1
- Petalinux 2020.1 installation and all required packages (for more details please refer to [UG1144])
- Reference Design documentation for the hardware combination in use (module and baseboard combination)

## Adjusting project for NAND boot mode

The equipped NAND flash on Enclustra modules is typically 512 MBytes large. No adjustments need to be made to the default Petalinux project configuration.

## Program NAND

Please refer to the reference design documentation for your module and baseboard configuration to see the various methods of programming the NAND memory. Additionally, Xilinx [UG1144] provides information for using tftpboot for programming the various memories. Further, have a look at [Zynq-7000: NAND Checklist](https://www.xilinx.com/support/answers/59311.html).

Booting directly from the NAND Flash memory is not supported. The BOOT.BIN including the PMUFW, FSBL, Bitstream and U-Boot have to be started from SD Card (MMC) or QSPI Flash (e.g. [QSPI_boot_mode.md](QSPI_boot_mode.md)). When U-Boot is booted it can load and boot the Linux system stored on the NAND Flash memory [EBE NAND flash](https://enclustra.github.io/ebe-docs/user-doc-xilinx/index_xilinx.html#nand-flash).

This means that only the "image.ub" for programming to the NAND flash is left, which contains the kernel rootfs and the kernel device tree.

The following steps must be performed.

1. Programm the QSPI (optionally SD):

 Have a look at the detailed instructions: [QSPI_boot_mode.md](QSPI_boot_mode.md) or [SD_boot_mode.md](SD_boot_mode.md).

2. Programm the NAND

 For detailed instructions have a look at the user manuals:
    * Mercury ZX1 chapter [3.10 NAND Flash Programming](https://download.enclustra.com/public_files/SoC_Modules/Mercury_ZX1/Mercury_ZX1_User_Manual_V05.pdf)
    * Mars ZX3 chapter [3.9 NAND Flash Programming](https://download.enclustra.com/public_files/SoC_Modules/Mars_ZX3/Mars_ZX3_User_Manual_V07.pdf)
    * Mars ZX5 chapter [3.10 NAND Flash Programming](https://download.enclustra.com/public_files/SoC_Modules/Mercury_ZX5/Mercury_ZX5_User_Manual_V05.pdf)

> **_NOTE:_** Please note that Vivado Hardware Manager does not support the NAND flash type equipped on the Mercury ZX1 SoC module.

[UG1144]: https://www.xilinx.com/support/documentation/sw_manuals/xilinx2020_1/ug1144-petalinux-tools-reference-guide.pdf
