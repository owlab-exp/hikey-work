usb reset

setenv bootargs console=ttyAMA3,115200 root=/dev/sda2 rootwait rw quiet efi=noruntime

fatload usb 0:1 0x10000000 image

fatload usb 0:1 0x13000000 hi6220-hikey.dtb

booti 0x10000000 - 0x13000000

OR

sudo mount -o loop,rw,sync,offset=32256 hikey-jessie_developer_20151130-387mg mnt-img
docker run -it --rm --net=host -v $(pwd)/mnt-img:/var/lib/tftpboot --name tftpd drerik/tftpd

setenv serverip 192.168.0.200

tftpboot 0x10000000 Image
tftpboot 0x13000000 hi6220-hikey.dtb
booti 0x10000000 - 0x13000000
