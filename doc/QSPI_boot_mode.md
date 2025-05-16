# QSPI boot mode adjustments to Petalinux

## Prerequisites

- Supported Linux OS for Petalinux 2024.1 (refer to  [UG1144])
- All required packages as listed in the [Petalinux 2024.1 Release Notes](https://adaptivesupport.amd.com/s/article/000036178?language=en_US)
- Petalinux 2024.1 installation (refer to  [UG1144])
- Reference Design documentation for the hardware combination in use (module and baseboard combination)
- Successful Petalinux build as described in the [BSP usage documentation](BSP.md)

## Petalinux QSPI boot mode adjustments

The QSPI flash size on Enclustra modules is 64 MBytes. The only exception are Andromeda modules with a size of 128 MBytes. The Enclustra BSPs are configured to use the following QSPI flash layout:

For Mercury and Mars modules:

| partition name | partition offset | partition size  | partition size in MB |
|----------------|------------------|-----------------|----------------------|
| boot           | 0x0              | 0x1f00000       | 31                   |
| bootenv        | 0x1f00000        | 0x80000         | 0.5                  |
| bootscr        | 0x1f80000        | 0x80000         | 0.5                  |
| kernel         | 0x2000000        | 0x2000000       | 32                   |

For Andromeda modules:

| partition name | partition offset | partition size  | partition size in MB |
|----------------|------------------|-----------------|----------------------|
| boot           | 0x0              | 0x3f00000       | 63                   |
| bootenv        | 0x3f00000        | 0x80000         | 0.5                  |
| bootscr        | 0x3f80000        | 0x80000         | 0.5                  |
| kernel         | 0x4000000        | 0x4000000       | 64                   |

Note: The QSPI flash layout for Andromeda_XZU65_ST1 was identical with the layout for Mercury and Mars modules in release 2024.1_v1.0.1 and earlier.

The rootfs uses the non-persistent ramfs type and is packaged into the generated `image.ub` file. The necessary adjustments for the QSPI offsets and the fit image are set via the following variables in the Petalinux project configuration:

```bash
# For Zynq Ultrascale+ devices (XU* modules)
CONFIG_SUBSYSTEM_FLASH_PSU_QSPI_0_BANKLESS_PART0_NAME="boot"
CONFIG_SUBSYSTEM_FLASH_PSU_QSPI_0_BANKLESS_PART0_SIZE=0x1f00000
CONFIG_SUBSYSTEM_FLASH_PSU_QSPI_0_BANKLESS_PART1_NAME="bootenv"
CONFIG_SUBSYSTEM_FLASH_PSU_QSPI_0_BANKLESS_PART1_SIZE=0x80000
CONFIG_SUBSYSTEM_FLASH_PSU_QSPI_0_BANKLESS_PART2_NAME="bootscr"
CONFIG_SUBSYSTEM_FLASH_PSU_QSPI_0_BANKLESS_PART2_SIZE=0x80000
CONFIG_SUBSYSTEM_FLASH_PSU_QSPI_0_BANKLESS_PART3_NAME="kernel"
CONFIG_SUBSYSTEM_FLASH_PSU_QSPI_0_BANKLESS_PART3_SIZE=0x2000000
CONFIG_SUBSYSTEM_UBOOT_QSPI_FIT_IMAGE_OFFSET=0x2000000
CONFIG_SUBSYSTEM_UBOOT_QSPI_FIT_IMAGE_SIZE=0x2000000

# For Zynq 7000 (ZX* modules)
CONFIG_SUBSYSTEM_FLASH_PS7_QSPI_0_BANKLESS_PART0_NAME="boot"
CONFIG_SUBSYSTEM_FLASH_PS7_QSPI_0_BANKLESS_PART0_SIZE=0x1f00000
CONFIG_SUBSYSTEM_FLASH_PS7_QSPI_0_BANKLESS_PART1_NAME="bootenv"
CONFIG_SUBSYSTEM_FLASH_PS7_QSPI_0_BANKLESS_PART1_SIZE=0x80000
CONFIG_SUBSYSTEM_FLASH_PS7_QSPI_0_BANKLESS_PART2_NAME="bootscr"
CONFIG_SUBSYSTEM_FLASH_PS7_QSPI_0_BANKLESS_PART2_SIZE=0x80000
CONFIG_SUBSYSTEM_FLASH_PS7_QSPI_0_BANKLESS_PART3_NAME="kernel"
CONFIG_SUBSYSTEM_FLASH_PS7_QSPI_0_BANKLESS_PART3_SIZE=0x2000000
CONFIG_SUBSYSTEM_UBOOT_QSPI_FIT_IMAGE_OFFSET=0x2000000
```
Additional configurations are set in the respective `u-boot.cfg` fragments in the meta-enclustra-module layer.

## Programming the QSPI flash
Program the QSPI flash on the module as described in the respective reference design documentation provided on github (Section **"Programming the QSPI flash"**) using the generated binaries found in **PETALINUX_PROJECT_FOLDER -> images -> linux**:
- Offset 0: BOOT.BIN
- Offset 0x1f80000: boot.scr
- Offset 0x2000000: image.ub

The following bash script provides an example using the `program_flash` utility provided by Vivado/Vitis (source the `settings64.sh` first to make the utility available):
```bash
#!/bin/bash -ex
default_parameter="-flash_type qspi-x4-single -fsbl zynqmp_fsbl.elf -cable type xilinx_tcf url TCP:127.0.0.1:3121"
program_flash -f BOOT.BIN $default_parameter
program_flash -f boot.scr -offset 0x1f80000 $default_parameter
program_flash -f image.ub -offset 0x2000000 $default_parameter
```

Alternatively, [UG1144] provides information for using tftpboot for programming the QSPI flash. As an example the following workflow can be used:
1. Prepare an SD card as described in the [SD boot mode documentation](SD_boot_mode.md).
2. Boot from the SD card.
3. Interrupt the boot process in u-boot.
4. Use tftpboot to program the QSPI flash with the individual binaries.

## Booting from QSPI flash
Follow the steps described in the respective reference design documentation provided on github (Section **Booting from the QSPI Flash**) to set up the hardware to boot from QSPI. Boot from QSPI into Linux. The default credentials are **Username: root, Password: root**.

[UG1144]: https://docs.amd.com/viewer/book-attachment/MVyApcmU3R9Mm97zSMBTWg/A1uhF~YnkvK0u6G775Tu_Q
