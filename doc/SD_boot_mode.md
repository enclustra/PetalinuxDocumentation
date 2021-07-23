# SD boot mode

## Prerequisites

- Supported Linux OS for Petalinux 2020.2
- Petalinux 2020.2 installation and all required packages (for more details please refer to [UG1144])
- Reference Design documentation for the hardware combination in use (module and baseboard combination)


## Create an SD Card with the boot images

The Petalinux tools reference guide [UG1144] from Xilinx describes how to create an SD card to boot from. This is a short description of the required steps to create a bootable SD card. 

1. Files needed for SD boot: (files are in PETALINUX_PROJECT_FOLDER -> images -> linux)
    - BOOT.BIN
    - boot.scr
    - Image
    - rootfs.tar.gz
2. Create two partitions on the SD Card. The first partition is **FAT32** formated, the second **EXT4**. [UG1144] descibes how to create these partitions.
3. Copy the files **BOOT.BIN, boot.scr and Image** to the first (FAT32) partition.
4. Extract the file **rootfs.tar.gz** to the second partition: 'tar -xvf rootfs.tar.gz -C /path/to/second/partition'
5. Unmount all partitions and remove the SD card.



## Boot from SD Card

1. Prepare the module and baseboard for SD boot mode

A detailed description of this step can be found in the reference design documentation.

2. Mount the module on the baseboard and set the DIP switches correctly.
3. Boot from the SD card.

A detailed description of these steps can be found in the reference design documentation.
1. Insert the SD card.
2. Plug in the USB cable and connect to a serial terminal.
3. Plug in the power jack.
4. The serial terminal shows the boot log of U-Boot and Linux. The default login for a Petalinux project is: **Username: root, Password: root**

[UG1144]: (https://www.xilinx.com/support/documentation/sw_manuals/xilinx2020_2/ug1144-petalinux-tools-reference-guide.pdf)