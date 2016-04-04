usb reset

setenv bootargs console=ttyAMA3,115200 root=/dev/sda2 rootwait rw quiet efi=noruntime

fatload usb 0:1 0x10000000 image

fatload usb 0:1 0x13000000 hi6220-hikey.dtb

booti 0x10000000 - 0x13000000
