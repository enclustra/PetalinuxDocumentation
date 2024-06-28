# BSP usage documentation

1. [Prerequisites](#prerequisites)
2. [Petalinux project creation and build with BSP file](#petalinux-project-creation-and-build-with-bsp-file)
3. [Updating the hardware file used by the Petalinux project](#updating-the-hardware-file-used-by-the-petalinux-project)
4. [Accelerating Petalinux builds](#accelerating-petalinux-builds)
5. [Changes to Petalinux default configurations](#changes-to-petalinux-default-configurations)
    1. [Kernel changes](#kernel-changes)
    2. [U-boot changes](#u-boot-changes)
    3. [Device-tree changes](#device-tree-changes)
    4. [Root file system changes](#root-file-system-changes)
6. [CPU frequency](#cpu-frequency)

## Prerequisites

- Supported Linux OS for Petalinux 2022.1
- Petalinux 2022.1 installation and all required packages (for more details please refer to  [UG1144])

## Petalinux project creation and build with BSP file

The provided **BSP** file follows the naming convention of `Petalinux_${MODULE_NAME}_${BASEBOARD_NAME}_${BOOT_MODE}.bsp`. Using the Mercury ME-XU5-2EG-1I-D11E as an example in combination with the PE1 baseboard and SD card boot mode the file will have the name: `Petalinux_ME-XU5-2EG-1I-D11E_PE1_SD.bsp`

- Open a console and source the Petalinux environment script: `source /<path-to-petalinux-installation-dir>/settings.sh`

- To create a Petalinux project from the provided bsp file use: `petalinux-create --type project -s <path-to-bsp-file>.bsp`

- The created project has the following structure
    ```
    PETALINUX_PROJECT_FOLDER
    ├── components
    ├── config.project
    └── project-spec
    ├── attributes
    ├── configs
    │   ├── busybox
    │   │   └── inetd.conf
    │   ├── config
    │   ├── init-ifupdown
    │   │   └── interfaces
    │   ├── rootfs_config
    │   └── systemd-conf
    │       └── wired.network
    └── meta-user
        ├── conf
        │   ├── layer.conf
        │   ├── petalinuxbsp.conf
        │   └── user-rootfsconfig
        ├── COPYING.MIT
        ├── README
        ├── recipes-apps
        ├── recipes-bsp
        │   ├── device-tree
        │   │   ├── device-tree.bbappend
        │   │   └── files
        │   │       ├── pl-custom.dtsi
        │   │       ├── system-user.dtsi
        │   │       └── <Enclustra specific device tree files>
        │   ├── u-boot
        │   │   ├── files
        │   │   │   ├── 0001-ubifs-distroboot-support.patch
        │   │   │   ├── 0008-Enclustra-MAC-address-readout-from-EEPROM.patch
        │   │   │   ├── 0010-Enclustra-Zynqmp-Board-Patch.patch
        │   │   │   ├── 0012-Bugfix-for-atsha204a-driver.patch
        │   │   │   ├── 0020-Enclustra-ds28-eeprom-fix.patch
        │   │   │   ├── 0030-zynq-qspi.patch
        │   │   │   ├── 0040-emmc.patch
        │   │   │   ├── 0050-Xilinx-PHY.patch
        │   │   │   ├── 0060-env-qspi-boot-avoid-ubifs.patch
        │   │   │   ├── bsp.cfg
        │   │   │   ├── platform-top.h
        │   │   │   └── u-boot.cfg
        │   │   └── u-boot-xlnx_%.bbappend
        │   └── uboot-device-tree
        │       ├── files
        │       │   └── system-user.dtsi
        │       └── uboot-device-tree.bbappend
        ├── recipes-core
        │   └── init-ifupdown
        │       ├── files
        │       │   └── interfaces_dual_eth
        │       └── init-ifupdown_%.bbappend
        └── recipes-kernel
            └── linux
                ├── linux-xlnx
                │   ├── 0010-RTL8211F.patch
                │   ├── bsp.cfg
                │   └── kernel.cfg
                └── linux-xlnx_%.bbappend
    ```

- Use `petalinux-build` to build the project

- Use the `petalinux-package` command to generate the boot components:
    - for ZynqMP:
        ```
        petalinux-package --boot --fsbl images/linux/zynqmp_fsbl.elf --u-boot images/linux/u-boot.elf --pmufw images/linux/pmufw.elf --fpga images/linux/system.bit --force
        ```
    - for Zynq:
        ```
        petalinux-package --boot --fsbl images/linux/zynq_fsbl.elf --fpga images/linux/system.bit --u-boot --force
        ```

    Refer to [UG1144] for the correct arguments.

- The boot components will be located in **PETALINUX_PROJECT_FOLDER -> images -> linux**.

- Run the created images on hardware
    - For SD boot mode, follow: [SD boot mode](SD_boot_mode.md)
    - For QSPI boot mode, follow: [QSPI boot mode](QSPI_boot_mode.md)
    - For NAND boot mode, follow: [NAND boot mode](NAND_boot_mode.md)
    - For eMMC boot mode, follow: [eMMC boot mode](EMMC_boot_mode.md)

## Updating the hardware file (.xsa) used by the Petalinux project

The provided `xsa` file (**PETALINUX_PROJECT_FOLDER -> project-spec -> hw_description**) is the hardware archive generated from the corresponding Enclustra reference design. To change the `xsa` file used by Petalinux please follow these steps:
- Make your changes to the hardware in Vivado
- Generate the bitstream and export the hardware including the bitstream
- Then update the hardware description in Petalinux: `petalinux-config --get-hw-description=<path-to-exported-xsa-file>`
- If needed, make changes to the Petalinux config corresponding to the updated hardware
- Rebuild the project: `petalinux-build`
- Finally, use `petalinux-package --boot ...` to regenerate the boot files

## Accelerating Petalinux builds
Petalinux fetches all necessary sources during the build process and afterwards uses incremental builds. If a lot of builds need to be run, the process can be accelerated by downloading these sources into a separate directory and referencing this directory in **PETALINUX_PROJECT_FOLDER -> project-spec -> meta-user -> conf -> petalinuxbsp.conf**
- Download the cache files from https://www.xilinx.com/support/download/index.html/content/xilinx/en/downloadNav/embedded-design-tools.html under **PetaLinux Tools sstate-cache artifacts** to a convenient directory outside of the Petalinux project
- Using `petalinux-config` within the Petalinux project, navigate to **Yocto Settings -> Add pre-mirror url**
- Enter the path to downloads directory. Prepend the path with `file://`
- Navigate to **Yocto Settings -> Local sstate feeds and settings**
- Enter the path to the installed sstate cache directory corresponding to your SoC architecture
- Save and exit

Complete offline builds are also possible by following the steps from https://xilinx-wiki.atlassian.net/wiki/spaces/A/pages/60129817/Xilinx+Yocto+Builds+without+an+Internet+Connection

## Changes to Petalinux default configurations
The provided **BSPs** contain the following modifications to the default Petalinux configuration and sources.

### Kernel changes
The following patches are added to the default Petalinux kernel:
| Patch                                                        | Functionality                                                             |
|--------------------------------------------------------------|---------------------------------------------------------------------------|
| [0010-RTL8211F.patch](../patches/kernel/0010-RTL8211F.patch) | Add center tap config for RTL8211F Ethernet PHY equipped on some modules. |

Further changes:
- QSPI
    - Reduced kernel size by disabling non mandatory drivers from the default Petalinux kernel config. The changes to the default kernel configuration are located in **PETALINUX_PROJECT_FOLDER -> project-spec -> meta-user -> recipes-kernel -> linux -> linux-xlnx -> kernel.cfg**.

### U-boot changes
The following patches are added to the default Petalinux u-boot:
| Patch                                                                                                                                | Functionality                                                                                                                                   |
|--------------------------------------------------------------------------------------------------------------------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------|
| [0008-Enclustra-MAC-address-readout-from-EEPROM.patch](../patches/uboot/Common/0008-Enclustra-MAC-address-readout-from-EEPROM.patch) | Add functionality to read Enclustra MAC address from EEPROM                                                                                     |
| [0012-Bugfix-for-atsha204a-driver.patch](../patches/uboot/Common/0012-Bugfix-for-atsha204a-driver.patch)                             | Prevent ATSHA EEPROM from timing out                                                                                                            |
| [0020-Enclustra-ds28-eeprom-fix.patch](../patches/uboot/Common/0020-Enclustra-ds28-eeprom-fix.patch)                                 | Add support for DS28 EEPROM                                                                                                                     |
| [0030-zynq-qspi.patch](../patches/uboot/Common/0030-zynq-qspi.patch)                                                                 | Fix read instruction for QSPI                                                                                                                   |
| [0040-emmc.patch](../patches/uboot/Common/0040-emmc.patch)                                                                           | - Change initialization flow for eMMC ([origin](https://github.com/enclustra-bsp/xilinx-uboot/commit/cafd52079faed3db66809c3244a393995b14d5ce)) <br> - Get "max-frequency" from the device-tree <br> - Support card detect via gpio|
| [0050-Xilinx-PHY.patch](../patches/uboot/Common/0050-Xilinx-PHY.patch)                                                               | In case the Xilinx device-tree entry is missing, try the normal way to configure the phy instead of writing an error                            |

Changes for MPSoC modules (XU*):
| Patch                                                                                                                      | Functionality                                                                                    |
|----------------------------------------------------------------------------------------------------------------------------|--------------------------------------------------------------------------------------------------|
| [0010-Enclustra-Zynqmp-Board-Patch.patch](../patches/uboot/zynqMP/0010-Enclustra-Zynqmp-Board-Patch.patch)                 | - Read and set MAC address <br> - U-boot command to read mac-address                             |
| [0060-env-qspi-boot-avoid-ubifs.patch](../patches/uboot/zynqMP/0060-env-qspi-boot-avoid-ubifs.patch)                       | Revert to old boot flow just sourcing a u-boot script                                            |
| [0070-Change-u-boot-script-load-address.patch](../patches/uboot/zynqMP/0070-Change-u-boot-script-load-address.patch)       | Load u-boot script at lower address to support devices with smaller PS DDR memory                |

Changes for SoC modules (ZX*):
| Patch                                                                                                     | Functionality                                                        |
|-----------------------------------------------------------------------------------------------------------|----------------------------------------------------------------------|
| [0010-Enclustra-Zynq-Board-Patch.patch](../patches/uboot/zynq/0010-Enclustra-Zynq-Board-Patch.patch)      | - Read and set MAC address <br> - U-boot command to read mac-address <br> - U-boot command to switch MIO pins between NAND and QSPI|

Further changes:
- The changes to the default u-boot configuration are located in:
   - **PETALINUX_PROJECT_FOLDER -> project-spec -> meta-user -> recipes-bsp -> u-boot -> files -> u-boot.cfg**
- QSPI
    - changes to adjust Petalinux for QSPI boot mode. More details are available in the [QSPI boot mode chapter](./QSPI_boot_mode.md)

### Device tree changes
The provided **BSP** uses Enclustra specific device tree include files to make necessary adjustments to the automatically generated Petalinux device tree. The corresponding device tree include files are located in **PETALINUX_PROJECT_FOLDER -> project-spec -> meta-user -> recipes-bsp -> device-tree -> files**. Any additional changes should be made in those files. The Enclustra include files are seperated into module and baseboard specific device tree files.

### Root file system changes
The following packages are added to the default Petalinux root file system configuration:
- phytool (read PHY registers for RGMII delay check)
- hdparm (memory read speed for SD card, eMMC, USB)
- memtester (DDR memory test)
- i2c-tools (PS I2C tests)
- iperf3 (network benchmarking tool)

## CPU frequency
Depending on the device and its speed grade, a different maximum CPU frequency results. The possible cpu frequencies for scaling are fix in the device tree ([zynqmp.dtsi](https://github.com/Xilinx/linux-xlnx/blob/master/arch/arm64/boot/dts/xilinx/zynqmp.dtsi)/[zynq-7000.dtsi](https://github.com/Xilinx/linux-xlnx/blob/master/arch/arm/boot/dts/zynq-7000.dtsi)) and may not match. Therefore, do not be distracted by the following warning, by default the device runs at its maximum frequency.
```
[    5.323088] cpu cpu0: dev_pm_opp_set_rate: failed to find current OPP for freq 1333333320 (-34)
[    5.340367] cpu cpu0: dev_pm_opp_set_rate: failed to find current OPP for freq 1333333320 (-34)
```
For more information about CPU frequency scaling please visit <https://xilinx-wiki.atlassian.net/wiki/spaces/A/pages/18841831/CPU+frequency+scaling>.

[UG1144]: <https://www.xilinx.com/support/documentation/sw_manuals/xilinx2022_1/ug1144-petalinux-tools-reference-guide.pdf>
