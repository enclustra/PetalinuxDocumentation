# SD boot mode

## Prerequisites

- Supported Linux OS for Petalinux 2020.1
- Petalinux 2020.1 installation and all required packages (for more details please refer to https://www.xilinx.com/support/documentation/sw_manuals/xilinx2020_1/ug1144-petalinux-tools-reference-guide.pdf)
- Reference Design documentation for the hardware combination in use (module and baseboard combination)


## Create an SD Card with the boot images

The Petalinux tools reference guide (UG1144) from Xilinx describes how to create an SD card to boot from. This is a short description of the required steps to create a bootable SD card. 

- Files needed for SD boot: (files are in PETALINUX_PROJECT_FOLDER -> images -> linux)
    - BOOT.BIN
    - boot.scr
    - Image
    - rootfs.tar.gz
- Create two partitions on the SD Card. The first partition is **FAT32** formated, the second **EXT4**. UG1144 descibes how to create these partitions.
- Copy the files **BOOT.BIN, boot.scr and Image** to the first (FAT32) partition.
- Extract the file **rootfs.tar.gz** to the second partition: 'tar -xvf rootfs.tar.gz -C /path/to/second/partition'
- Unmount all partitions and remove the SD card.



## Boot from SD Card

- Prepare the module and baseboard for SD boot mode

A detailed description of this step can be found in the reference design documentation.

    - Mount the module on the baseboard and set the DIP switches correctly.

- Boot from the SD card.

A detailed description of these steps can be found in the reference design documentation.

    - Insert the SD card.
    - Plug in the USB cable and connect a serial terminal.
    - Plug in the power jack.
    - The serial terminal shows the log of the booting U-Boot and Linux. The default login for a Petalinux project is: **Username: root, Password: root**

