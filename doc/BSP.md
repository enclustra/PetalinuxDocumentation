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

## Prerequisites

- Supported Linux OS for Petalinux 2024.1 (refer to  [UG1144])
- All required packages as listed in the [Petalinux 2024.1 Release Notes](https://adaptivesupport.amd.com/s/article/000036178?language=en_US)
- Petalinux 2024.1 installation (refer to  [UG1144])

## Petalinux project creation and build with BSP file

The provided **BSP** file follows the naming convention of `Petalinux_${MODULE_NAME}_${BASEBOARD_NAME}_${BOOT_MODE}.bsp`. Using the Mercury ME-XU5-2EG-1I-D11E as an example in combination with the PE1 baseboard and SD card boot mode the file will have the name: `Petalinux_ME-XU5-2EG-1I-D11E_PE1_SD.bsp`

- Open a console and source the Petalinux environment script: `source /<path-to-petalinux-installation-dir>/settings.sh`

- To create a Petalinux project from the provided bsp file use: `petalinux-create project -s <path-to-bsp-file>.bsp`

- The created project has the following structure
    ```
    PETALINUX_PROJECT_FOLDER
    ├── components
    │   └── plnx_workspace
    └── project-spec
        ├── attributes
        ├── configs
        │   ├── busybox
        │   │   └── inetd.conf
        │   ├── config
        │   ├── flash_parts.txt
        │   ├── gen-machineconf.log
        │   ├── init-ifupdown
        │   │   └── interfaces
        │   ├── plnx_syshw_data
        │   ├── rootfs_config
        │   └── systemd-conf
        ├── hw-description
        ├── meta-enclustra
        │   ├── meta-enclustra-baseboard
        │   │   ├── conf
        │   │   │   ├── layer.conf
        │   │   │   └── machine
        │   │   │       ├── pe1-generic.conf
        │   │   │       ├── pe3-generic.conf
        │   │   │       ├── refdes-xu1-pe1.conf
        │   │   │       ├── refdes-xu1-pe3.conf
        │   │   │       ├── refdes-xu1-st1.conf
        │   │   │       ├── refdes-xu3-st3.conf
        │   │   │       ├── refdes-xu5-pe1.conf
        │   │   │       ├── refdes-xu5-pe3.conf
        │   │   │       ├── refdes-xu5-st1.conf
        │   │   │       ├── refdes-xu61-pe1.conf
        │   │   │       ├── refdes-xu61-pe3.conf
        │   │   │       ├── refdes-xu61-st1.conf
        │   │   │       ├── refdes-xu6-pe1.conf
        │   │   │       ├── refdes-xu6-pe3.conf
        │   │   │       ├── refdes-xu6-st1.conf
        │   │   │       ├── refdes-xu7-pe1.conf
        │   │   │       ├── refdes-xu7-pe3.conf
        │   │   │       ├── refdes-xu7-st1.conf
        │   │   │       ├── refdes-xu8-pe1.conf
        │   │   │       ├── refdes-xu8-pe3.conf
        │   │   │       ├── refdes-xu8-st1.conf
        │   │   │       ├── refdes-xu9-pe1.conf
        │   │   │       ├── refdes-xu9-pe3.conf
        │   │   │       ├── refdes-xu9-st1.conf
        │   │   │       ├── refdes-xzu65-pe5.conf
        │   │   │       ├── refdes-xzu65-st1.conf
        │   │   │       ├── refdes-xzu80-pe5.conf
        │   │   │       ├── refdes-xzu90-pe5.conf
        │   │   │       ├── refdes-zx1-pe1.conf
        │   │   │       ├── refdes-zx1-pe3.conf
        │   │   │       ├── refdes-zx1-st1.conf
        │   │   │       ├── refdes-zx2-st3.conf
        │   │   │       ├── refdes-zx3-st3.conf
        │   │   │       ├── refdes-zx5-pe1.conf
        │   │   │       ├── refdes-zx5-pe3.conf
        │   │   │       ├── refdes-zx5-st1.conf
        │   │   │       ├── st1-generic.conf
        │   │   │       └── st3-generic.conf
        │   │   ├── COPYING.MIT
        │   │   ├── README.md
        │   │   └── recipes-bsp
        │   │       └── device-tree
        │   │           ├── device-tree.bbappend
        │   │           └── files
        │   │               ├── common
        │   │               │   ├── zynq_enclustra_mars_st3.dtsi
        │   │               │   ├── zynq_enclustra_mercury_pe1.dtsi
        │   │               │   ├── zynq_enclustra_mercury_pe3.dtsi
        │   │               │   ├── zynq_enclustra_mercury_st1.dtsi
        │   │               │   ├── zynqmp_enclustra_andromeda_pe5.dtsi
        │   │               │   ├── zynqmp_enclustra_mars_st3.dtsi
        │   │               │   ├── zynqmp_enclustra_mercury_pe1.dtsi
        │   │               │   ├── zynqmp_enclustra_mercury_pe3.dtsi
        │   │               │   └── zynqmp_enclustra_mercury_st1.dtsi
        │   │               ├── refdes-xu1-pe1
        │   │               │   └── system-user.dtsi
        │   │               ├── refdes-xu1-pe3
        │   │               │   └── system-user.dtsi
        │   │               ├── refdes-xu1-st1
        │   │               │   └── system-user.dtsi
        │   │               ├── refdes-xu3-st3
        │   │               │   └── system-user.dtsi
        │   │               ├── refdes-xu5-pe1
        │   │               │   └── system-user.dtsi
        │   │               ├── refdes-xu5-pe3
        │   │               │   └── system-user.dtsi
        │   │               ├── refdes-xu5-st1
        │   │               │   └── system-user.dtsi
        │   │               ├── refdes-xu61-pe1
        │   │               │   └── system-user.dtsi
        │   │               ├── refdes-xu61-pe3
        │   │               │   └── system-user.dtsi
        │   │               ├── refdes-xu61-st1
        │   │               │   └── system-user.dtsi
        │   │               ├── refdes-xu6-pe1
        │   │               │   └── system-user.dtsi
        │   │               ├── refdes-xu6-pe3
        │   │               │   └── system-user.dtsi
        │   │               ├── refdes-xu6-st1
        │   │               │   └── system-user.dtsi
        │   │               ├── refdes-xu7-pe1
        │   │               │   └── system-user.dtsi
        │   │               ├── refdes-xu7-pe3
        │   │               │   └── system-user.dtsi
        │   │               ├── refdes-xu7-st1
        │   │               │   └── system-user.dtsi
        │   │               ├── refdes-xu8-pe1
        │   │               │   └── system-user.dtsi
        │   │               ├── refdes-xu8-pe3
        │   │               │   └── system-user.dtsi
        │   │               ├── refdes-xu8-st1
        │   │               │   └── system-user.dtsi
        │   │               ├── refdes-xu9-pe1
        │   │               │   └── system-user.dtsi
        │   │               ├── refdes-xu9-pe3
        │   │               │   └── system-user.dtsi
        │   │               ├── refdes-xu9-st1
        │   │               │   └── system-user.dtsi
        │   │               ├── refdes-xzu65-pe5
        │   │               │   └── system-user.dtsi
        │   │               ├── refdes-xzu65-st1
        │   │               │   └── system-user.dtsi
        │   │               ├── refdes-xzu80-pe5
        │   │               │   └── system-user.dtsi
        │   │               ├── refdes-xzu90-pe5
        │   │               │   └── system-user.dtsi
        │   │               ├── refdes-zx1-pe1
        │   │               │   └── system-user.dtsi
        │   │               ├── refdes-zx1-pe3
        │   │               │   └── system-user.dtsi
        │   │               ├── refdes-zx1-st1
        │   │               │   └── system-user.dtsi
        │   │               ├── refdes-zx2-st3
        │   │               │   └── system-user.dtsi
        │   │               ├── refdes-zx3-st3
        │   │               │   └── system-user.dtsi
        │   │               ├── refdes-zx5-pe1
        │   │               │   └── system-user.dtsi
        │   │               ├── refdes-zx5-pe3
        │   │               │   └── system-user.dtsi
        │   │               └── refdes-zx5-st1
        │   │                   └── system-user.dtsi
        │   └── meta-enclustra-module
        │       ├── conf
        │       │   ├── layer.conf
        │       │   └── machine
        │       │       ├── andromeda-generic.conf
        │       │       ├── mars-generic.conf
        │       │       ├── mercury-generic.conf
        │       │       ├── xu1-module.conf
        │       │       ├── xu3-module.conf
        │       │       ├── xu5-module.conf
        │       │       ├── xu61-module.conf
        │       │       ├── xu6-module.conf
        │       │       ├── xu7-module.conf
        │       │       ├── xu8-module.conf
        │       │       ├── xu9-module.conf
        │       │       ├── xzu65-module.conf
        │       │       ├── xzu80-module.conf
        │       │       ├── xzu90-module.conf
        │       │       ├── zx1-module.conf
        │       │       ├── zx2-module.conf
        │       │       ├── zx3-module.conf
        │       │       └── zx5-module.conf
        │       ├── COPYING.MIT
        │       ├── README.md
        │       ├── recipes-bsp
        │       │   ├── device-tree
        │       │   │   ├── device-tree.bbappend
        │       │   │   └── files
        │       │   │       ├── zynq_enclustra_common.dtsi
        │       │   │       ├── zynq_enclustra_mars_zx2.dtsi
        │       │   │       ├── zynq_enclustra_mars_zx3.dtsi
        │       │   │       ├── zynq_enclustra_mercury_zx1.dtsi
        │       │   │       ├── zynq_enclustra_mercury_zx5.dtsi
        │       │   │       ├── zynq_enclustra_nand_parts.dtsi
        │       │   │       ├── zynqmp_enclustra_andromeda_xzu65.dtsi
        │       │   │       ├── zynqmp_enclustra_andromeda_xzu80.dtsi
        │       │   │       ├── zynqmp_enclustra_andromeda_xzu90.dtsi
        │       │   │       ├── zynqmp_enclustra_common.dtsi
        │       │   │       ├── zynqmp_enclustra_mars_xu3.dtsi
        │       │   │       ├── zynqmp_enclustra_mercury_xu1.dtsi
        │       │   │       ├── zynqmp_enclustra_mercury_xu5.dtsi
        │       │   │       ├── zynqmp_enclustra_mercury_xu61.dtsi
        │       │   │       ├── zynqmp_enclustra_mercury_xu6.dtsi
        │       │   │       ├── zynqmp_enclustra_mercury_xu7.dtsi
        │       │   │       ├── zynqmp_enclustra_mercury_xu8.dtsi
        │       │   │       └── zynqmp_enclustra_mercury_xu9.dtsi
        │       │   └── u-boot
        │       │       ├── files
        │       │       │   ├── andromeda
        │       │       │   │   └── formfactor.cfg
        │       │       │   ├── common
        │       │       │   │   ├── 0008-Enclustra-MAC-address-readout-from-EEPROM.patch
        │       │       │   │   ├── 0012-Atsha204a-upstream-fixes.patch
        │       │       │   │   ├── 0020-Enclustra-ds28-eeprom-fix.patch
        │       │       │   │   ├── 0030-zynq-qspi.patch
        │       │       │   │   ├── 0040-emmc.patch
        │       │       │   │   └── 0050-update-ksz9131-driver.patch
        │       │       │   ├── mars
        │       │       │   │   └── formfactor.cfg
        │       │       │   ├── mercury
        │       │       │   │   └── formfactor.cfg
        │       │       │   ├── zynq
        │       │       │   │   ├── 0010-Enclustra-Zynq-Board-Patch.patch
        │       │       │   │   ├── 0011-Fix-zynq_qspi_fill_tx_fifo.patch
        │       │       │   │   ├── qspi
        │       │       │   │   │   ├── u-boot.cfg
        │       │       │   │   │   └── u-boot-nand.cfg
        │       │       │   │   └── sd
        │       │       │   │       ├── u-boot.cfg
        │       │       │   │       └── u-boot-nand.cfg
        │       │       │   └── zynqmp
        │       │       │       ├── 0010-Enclustra-Zynqmp-Board-Patch.patch
        │       │       │       ├── emmc
        │       │       │       │   └── u-boot.cfg
        │       │       │       ├── qspi
        │       │       │       │   └── u-boot.cfg
        │       │       │       └── sd
        │       │       │           └── u-boot.cfg
        │       │       └── u-boot-xlnx_%.bbappend
        │       ├── recipes-kernel
        │       │   └── linux
        │       │       ├── linux-xlnx
        │       │       │   ├── 0001-RTC.patch
        │       │       │   ├── 0010-RTL8211F.patch
        │       │       │   ├── bsp.cfg
        │       │       │   ├── zynq
        │       │       │   │   ├── qspi
        │       │       │   │   │   └── kernel.cfg
        │       │       │   │   └── sd
        │       │       │   │       └── kernel.cfg
        │       │       │   └── zynqmp
        │       │       │       ├── emmc
        │       │       │       │   └── kernel.cfg
        │       │       │       ├── qspi
        │       │       │       │   └── kernel.cfg
        │       │       │       └── sd
        │       │       │           └── kernel.cfg
        │       │       └── linux-xlnx_%.bbappend
        │       └── recipes-multimedia
        │           └── vcu
        │               └── kernel-module-vcu_%.bbappend
        └── meta-user
    ```
    Compared to previous releases, release 2024.1 uses its own enclustra user layers, `meta-enclustra-baseboard` and `meta-enclustra-module` to add support for Enclustra hardware. The build target is defined via the `Yocto Machine Name`, `Yocto Include Machine Name` and `Yocto Additional Overrides`. The packaged BSPs provided in the release section of each reference design repository are already set up to build the respective target.

- Use `petalinux-build` to build the project

- Use the `petalinux-package` command to generate the boot components:
    - for ZynqMP:
        ```
        petalinux-package boot --fsbl images/linux/zynqmp_fsbl.elf --u-boot images/linux/u-boot.elf --pmufw images/linux/pmufw.elf --fpga images/linux/system.bit --force
        ```
    - for Zynq:
        ```
        petalinux-package boot --fsbl images/linux/zynq_fsbl.elf --fpga images/linux/system.bit --u-boot --force
        ```

    The complete overview over all arguments is given in [UG1144].

- The boot components will be located in **PETALINUX_PROJECT_FOLDER -> images -> linux**.

- Run the created images on hardware
    - For SD boot mode, follow: [SD boot mode](SD_boot_mode.md)
    - For QSPI boot mode, follow: [QSPI boot mode](QSPI_boot_mode.md)
    - For NAND boot mode, follow: [NAND boot mode](NAND_boot_mode.md)
    - For eMMC boot mode, follow: [eMMC boot mode](EMMC_boot_mode.md)

## Updating the hardware file (.xsa) used by the Petalinux project

The provided `xsa` file (**PETALINUX_PROJECT_FOLDER -> project-spec -> hw_description**) is the hardware archive generated from the corresponding Enclustra reference design. To change the `xsa` file used by Petalinux follow these steps:
1. Make your changes to the hardware in Vivado.
2. Generate the bitstream and export the hardware including the bitstream.
3. Update the hardware description in Petalinux: `petalinux-config --get-hw-description=<path-to-exported-xsa-file>`

> **_Note:_**  It is recommended to run `petalinux-build -x mrproper` before updating the reference XSA file in order to avoid build errors related to leftover files from previous builds.

4. Make changes to the Petalinux project if needed.
5. Rebuild the project: `petalinux-build`
6. Use the `petalinux-package boot ...` command to regenerate the boot files.

## Accelerating Petalinux builds
Petalinux fetches all necessary sources during the build process and afterwards uses incremental builds. If a lot of builds need to be run, the process can be accelerated by downloading these sources into a separate directory and referencing this directory in **PETALINUX_PROJECT_FOLDER -> project-spec -> meta-user -> conf -> petalinuxbsp.conf**
1. Download the cache files from the [AMD download page](https://www.origin.xilinx.com/support/download/index.html/content/xilinx/en/downloadNav/embedded-design-tools/2024-1.html) under **PetaLinux Tools sstate-cache Artifacts** to a convenient directory outside of the Petalinux project.
2. Using `petalinux-config` within the Petalinux project, navigate to **Yocto Settings -> Add pre-mirror url**.
3. Enter the path to the downloads directory. Prepend the path with `file://`.
4. Navigate to **Yocto Settings -> Local sstate feeds and settings**.
5. Enter the path to the installed sstate cache directory corresponding to your SoC architecture.
6. Save and exit.

Complete offline builds are also possible by following the steps described in the [AMD confluence wiki](https://xilinx-wiki.atlassian.net/wiki/spaces/A/pages/60129817/Xilinx+Yocto+Builds+without+an+Internet+Connection).

## Changes to Petalinux default configurations
The provided Enclustra meta layer contain the following modifications.

### Kernel changes
The following patches are added to the default Petalinux kernel:
| Patch                                                        | Functionality                                                             |
|--------------------------------------------------------------|---------------------------------------------------------------------------|
| 0001-RTC.patch                                               | Ensure that the calibration register is written during probing            |
| 0010-RTL8211F.patch                                          | Add center tap config for RTL8211F Ethernet PHY equipped on some modules. |

The kernel configuration fragments are located in **PETALINUX_PROJECT_FOLDER -> project-spec -> meta-enclustra -> meta-enclustra-module -> recipes-kernel -> linux -> linux-xlnx -> <ARCH> -> <BOOT_MODE> -> kernel.cfg**.

### U-boot changes
The following patches are added to the default Petalinux u-boot:
| Patch                                                | Functionality                                                                    |
|------------------------------------------------------|----------------------------------------------------------------------------------|
| 0008-Enclustra-MAC-address-readout-from-EEPROM.patch | Add functionality to read Enclustra MAC address from EEPROM                      |
| 0012-Atsha204a-upstream-fixes.patch                  | Upstream fixes for the atsha204a driver                                          |
| 0020-Enclustra-ds28-eeprom-fix.patch                 | Add support for DS28 EEPROM                                                      |
| 0030-zynq-qspi.patch                                 | Fix read instruction for QSPI                                                    |
| 0040-emmc.patch                                      | - Change initialization flow for eMMC ([origin](https://github.com/enclustra-bsp/xilinx-uboot/commit/cafd52079faed3db66809c3244a393995b14d5ce)) <br> - Get "max-frequency" from the device-tree <br> - Support card detect via gpio |
| 0050-update-ksz9131-driver.patch                     | Update PHY driver to newer version to support RGMII delay configuration          |

Changes for MPSoC modules (XU*):
| Patch                                                | Functionality                                                                   |
|------------------------------------------------------|---------------------------------------------------------------------------------|
| 0010-Enclustra-Zynqmp-Board-Patch.patch              | - Read and set MAC address <br> - U-boot command to read mac-address            |

Changes for SoC modules (ZX*):
| Patch                                                | Functionality                                                                   |
|------------------------------------------------------|---------------------------------------------------------------------------------|
| 0010-Enclustra-Zynq-Board-Patch.patch                | - Read and set MAC address <br> - U-boot command to read mac-address <br> - U-boot command to switch MIO pins between NAND and QSPI |

The u-boot configuration fragments are located in **PETALINUX_PROJECT_FOLDER -> project-spec -> meta-enclustra -> meta-enclustra-module -> recipes-bsp -> u-boot -> <ARCH> -> <BOOT_MODE> -> u-boot.cfg**.

### Device tree changes
The meta-enclustra-module layer contains the device tree include files for all supported Enclustra modules. The meta-enclustra-baseboard layer contains the device tree include files for all supported Enclustra baseboards. In addition, the reference design device tree include file (`system-user.dtsi`) combining the module and baseboard device tree files for the repsective target are in the meta-enclustra-baseboard layer as well. The meta-enclustra layer is added as a user layer on top of the Petalinux meta-user layer and thus the `system-user.dtsi` file contents of the Enclustra layer take precendence over the default `system-user.dtsi` file in the meta-user layer.

### Root file system changes
The following changes are applied to the default Petalinux root file system configuration:
```bash
CONFIG_haveged=y
# CONFIG_nfs-utils is not set
CONFIG_i2c-tools=y
# CONFIG_openssh-sftp-server is not set
CONFIG_hdparm=y
CONFIG_strace=y
CONFIG_sysstat=y
CONFIG_resize-part=y
CONFIG_packagegroup-petalinux-display-debug=y
CONFIG_imagefeature-debug-tweaks=y
CONFIG_ADD_EXTRA_USERS="root:root;petalinux:petalinux;"
CONFIG_phytool=y
CONFIG_memtester=y
CONFIG_iperf3=y
CONFIG_usbutils=y
```

[UG1144]: <https://docs.amd.com/r/2024.1-English/ug1144-petalinux-tools-reference-guide/Overview>
