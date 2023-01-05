# Petalinux eMMC boot mode

## Preparation
Download the BSP for the SD card boot mode from the available reference design BSPs in the release section of the respective github repository and build the project with `petalinux-build`. Package the generated binaries into an MMC image with `petalinux-package --wic`. Write the generated image file to an SD card. This image will be used to later write the eMMC image onto the eMMC memory on the module. If the ext4 partition is too small to fit the eMMC image, you can resize the ext4 partition on the SD card with tools like gparted.

## BSP changes

Create the project as desribed in the BSP usage documentation.
Run `petalinux-config` command to open the project configuration and apply the following changes:

- navigate to `Subsystem AUTO Hardware Settings -> SD/SDIO Settings -> Primary SD/SDIO` and change `psu_sd_1` to `psu_sd_0`
- navigate to `Image Packaging Configuration` and change `/dev/mmcblk1p2` to `/dev/mmcblk0p2`
- save and exit the configuration

## Boot image generation
Rebuild the project with `petalinux-build` and wait for completion.
Package the generated binaries into an MMC image with `petalinux-package --wic`.

## Deployment
Copy the generated image to the SD card with `sudo cp petalinux-sdimage.wic <path-to-rootfs-on-SD-card>/home/petalinux/`. Unmount the SD card.
Set SD card boot mode as described in the respective reference design documentation and boot into Linux. The generated image should be under `/home/petalinux/`. Write the image to eMMC with `dd if=petalinux-sdimage.wic of=/dev/mmcblk0`. Wait for completion, this step takes time. After completion you can use `shutdown now` to exit Linux.
Disconnect power from the board and set the eMMC boot mode as described in the respective reference design documentation. The generated eMMC should boot into Linux successfully.