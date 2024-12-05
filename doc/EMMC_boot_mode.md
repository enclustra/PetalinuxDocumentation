# Petalinux eMMC boot mode

## Prerequisites

- Supported Linux OS for Petalinux 2024.1 (refer to  [UG1144])
- All required packages as listed in the [Petalinux 2024.1 Release Notes](https://adaptivesupport.amd.com/s/article/000036178?language=en_US)
- Petalinux 2024.1 installation (refer to  [UG1144])
- Reference Design documentation for the hardware combination in use (module and baseboard combination)
- Successful Petalinux build as described in the [BSP usage documentation](BSP.md)

## Preparing an SD card image
Download the BSP for the SD card boot mode from the available reference design BSPs in the release section of the respective github repository and build the project as described in the [BSP usage documentation](BSP.md) and [SD boot mode documentation](SD_boot_mode.md). The SD card image will be used to write the eMMC image into the eMMC memory on the module. If the `ext4` partition is too small to fit the eMMC image, the ext4 partition on the SD card can be extended with tools like `resize-part` or `gparted`.

## Building the eMMC image
Follow these steps to build the eMMC image:
1. Create the `BOOT.BIN` as described in the [BSP usage documentation](BSP.md) using the provided `*_EMMC.bsp`.
2. Use the `petalinux-package wic` command to create an MMC image. The default image will be created in **PETALINUX_PROJECT_FOLDER -> images -> linux -> petalinux-sdimage.wic** with a size of 6GB. It is possible to change the generated image size by passing arguments via the `--wks` flag. Refer to [UG1144] for details.

## Programming the eMMC flash
Follow these steps to program the eMMC flash with the image generated in the previous section:
1. Copy the generated image to the SD card with `sudo cp petalinux-sdimage.wic <path-to-rootfs-on-SD-card>/home/petalinux/`.
2. Unmount the SD card.
3. Boot from the SD card as described in the respective reference design documentation (Section **"Booting from the SD Card"**) using the default credentials **Username: root, Password: root**.
5. The generated image should be in the `/home/petalinux/` directory. Write the image to eMMC with `dd if=petalinux-sdimage.wic of=/dev/mmcblk0`.
6. Wait for completion. This step can take some time.
7. After completion, you can use `shutdown now` to exit Linux.
8. Remove the SD card from the SD card slot.
9. Unpower the board and boot from eMMC as described in the respective reference design documentation (Section **Booting from the eMMC**).

[UG1144]: https://docs.amd.com/viewer/book-attachment/MVyApcmU3R9Mm97zSMBTWg/A1uhF~YnkvK0u6G775Tu_Q