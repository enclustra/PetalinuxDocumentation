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

- Supported Linux OS for Petalinux 2020.1
- Petalinux 2020.1 installation and all required packages (for more details please refer to [Xilinx UG 1144](https://www.xilinx.com/support/documentation/sw_manuals/xilinx2020_1/ug1144-petalinux-tools-reference-guide.pdf))

## Petalinux project creation and build with BSP file

The provided **BSP** file follows the naming convention of `${MODULE_NAME}_${BASEBOARD_NAME}_${BOOT_MODE}.bsp`. Using the Mercury ME-XU5-2EG-1I-D11E as an example in combination with the PE1 baseboard and SD card boot mode the file will have the name: `ME-XU5-2EG-1I-D11E_PE1_SD.bsp`

- Open a console and source the Petalinux environment script: `source /<path-to-petalinux-installation-dir>/settings.sh`

- To create a Petalinux project from the provided bsp file use: `petalinux-create --type project -s <path-to-bsp-file>.bsp`

- The created project has the following structure
    ```
    PETALINUX_PROJECT_FOLDER
    ├── components
    │   └── ...
    ├── config.project
    ├── hardware
    │   └── vivado_export
    │       └── <Hardware description file>
    └── project-spec
        ├── attributes
        ├── configs
        │   ├── config
        │   ├── linux-xlnx
        │   │   └── plnx_kernel.cfg
        │   ├── rootfs_config
        │   └── u-boot-xlnx
        │       ├── config.cfg
        │       └── platform-auto.h
        │   ├── ...
        ├── hw-description
        │   ├── ...
        └── meta-user
            ├── conf
            │   ├── petalinuxbsp.conf
            │   ├── ...
            ├── recipes-apps
            │   ├── ...
            └── recipes-bsp
                ├── device-tree
                │   ├── device-tree.bbappend
                │   └── files
                │       ├── pl-custom.dtsi
                │       ├── system-user.dtsi
                │       ├── <Custom device tree include files provided by Enclustra>
                └── u-boot
                    ├── ...
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

    Refer to [Xilinx UG 1144](https://www.xilinx.com/support/documentation/sw_manuals/xilinx2020_1/ug1144-petalinux-tools-reference-guide.pdf) for the correct arguments.

- The boot components will be located in **PETALINUX_PROJECT_FOLDER -> images -> linux**.

- Run the created images on hardware
    - For SD boot mode, follow: [SD_boot_mode.md](SD_boot_mode.md)
    - For QSPI boot mode, follow: [QSPI_boot_mode.md](QSPI_boot_mode.md)
    - For NAND boot mode, follow: [NAND_boot_mode.md](NAND_boot_mode.md)

## Updating the hardware file (.xsa) used by the Petalinux project

The provided `xsa` file (**PETALINUX_PROJECT_FOLDER -> hardware -> vivado_export**) is the hardware archive generated from the corresponding Enclustra reference design. To change the `xsa` file used by Petalinux please follow these steps:
- Make your changes to the hardware in Vivado
- Generate the bitstream and export the hardware including the bitstream
- The easiest way is to replace the provided `xsa` file with the newly generated one in the **PETALINUX_PROJECT_FOLDER -> hardware -> vivado_export folder**
- Then update the hardware description in Petalinux: `petalinux-config --get-hw-description=./hardware/vivado_export`
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
- [si5338.patch](../patches/kernel/si5338.patch)

 Add the driver for the programmable i2c clock generator Si5338. Kernel configuration element is **COMMON_CLK_SI5338**.

Further changes:
- QSPI
    - Reduced kernel size by disabling non mandatory drivers from the default Petalinux kernel config. The changes to the default kernel configuration are located in **PETALINUX_PROJECT_FOLDER -> project-spec -> meta-user -> recipes-kernel -> linux -> linux-xlnx -> kernel.cfg**.

### U-boot changes
The following patches are added to the default Petalinux u-boot:
- **eeprom.patch**

 1. Same for [SoC](patches/uboot/SoC/eeprom.patch) and  [MPSoC](patches/uboot/MPSoC/eeprom.patch)
 2. Add functionality to read the mac-address from the atsha204a secure-eeprom
 3. Prevent atsha204a-i2c from timing out ([origin](https://github.com/enclustra-bsp/xilinx-uboot/commit/b41dce14ba2c7adff6027cf5fbd2121bc02ed1e0))

- **emmc.patch**

 1. Same for [SoC](patches/uboot/SoC/emmc.patch) and [MPSoC](patches/uboot/MPSoC/emmc.patch)
 2. Change the initialization flow for e/MMC cards ([origin](https://github.com/enclustra-bsp/xilinx-uboot/commit/cafd52079faed3db66809c3244a393995b14d5ce))
 3. Support card detect via gpio
 4. Get "max-frequency" from the device-tree

- **phy_delay.patch**

 1. Same for [SoC](patches/uboot/SoC/phy_delay.patch) and [MPSoC](patches/uboot/MPSoC/phy_delay.patch)
 2. When checking whether the value is too large, it now differentiates whether the values are 4 or 5 bits wide

- **[zynq_board.patch](patches/uboot/SoC/zynq_board.patch)**

 1. Read and set mac-address
 2. U-Boot command to read mac-address
 3. Functionality / U-Boot command to swich MIO pins between NAND and QSPI
 4. MGT routing capabilities for cosmos

- **[zynqmp_board.patch](patches/uboot/MPSoC/zynqmp_board.patch)**

 1. Read and set mac-address
 2. U-Boot command to read mac-address

Further changes:
- The changes to the default u-boot configuration are located in:
   - **PETALINUX_PROJECT_FOLDER -> project-spec -> meta-user -> recipes-bsp -> u-boot -> files -> u-boot.cfg**
- QSPI
    - changes to adjust Petalinux for QSPI boot mode. More details are available at [./QSPI_boot_mode.md](./QSPI_boot_mode.md)

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
For more information about [CPU frequency scaling](https://xilinx-wiki.atlassian.net/wiki/spaces/A/pages/18841831/CPU+frequency+scaling).
