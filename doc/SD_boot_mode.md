# SD boot mode

## Prerequisites

- Supported Linux OS for Petalinux 2024.1 (refer to  [UG1144])
- All required packages as listed in the [Petalinux 2024.1 Release Notes](https://adaptivesupport.amd.com/s/article/000036178?language=en_US)
- Petalinux 2024.1 installation (refer to  [UG1144])
- Reference Design documentation for the hardware combination in use (module and baseboard combination)
- Successful Petalinux build as described in the [BSP usage documentation](BSP.md)

## Preparing an SD card for booting
Follow these steps to build the SD card image:
1. Create the `BOOT.BIN` as described in the [BSP usage documentation](BSP.md).
2. Use the `petalinux-package wic` command to create an SD card image. The default image will be created in **PETALINUX_PROJECT_FOLDER -> images -> linux -> petalinux-sdimage.wic** with a size of 6GB. It is possible to change the generated image size by passing arguments via the `--wks` flag. Refer to [UG1144] for details.
3. Write the `petalinux-sdimage.wic` image to an SD card.

## Booting from SD Card
Boot from the SD card as described in the respective reference design documentation (Section **"Booting from the SD Card"**) using the default credentials **Username: root, Password: root**.

[UG1144]: https://docs.amd.com/viewer/book-attachment/MVyApcmU3R9Mm97zSMBTWg/A1uhF~YnkvK0u6G775Tu_Q
