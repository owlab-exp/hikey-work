# From USB Drive

```
setenv bootargs console=ttyAMA3,115200 root=/dev/sda2 rootwait rw quiet efi=noruntime
usb reset
usb part
fatls usb 0:1

fatload usb 0:1 0x00080000 image

fatload usb 0:1 0x02000000 hi6220-hikey.dtb

booti 0x00080000 - 0x02000000
```
# From TFTPD

sudo mount -o loop,rw,sync,offset=32256 hikey-jessie_developer_20151130-387mg mnt-img
docker run -it --rm --net=host -v $(pwd)/mnt-img:/var/lib/tftpboot --name tftpd drerik/tftpd
```
setenv bootargs console=ttyAMA3,115200 root=/dev/sda2 rootwait rw quiet efi=noruntime
setenv serverip 192.168.0.200

tftpboot 0x00080000 Image
tftpboot 0x02000000 hi6220-hikey.dtb
booti 0x00080000 - 0x02000000
```
- /working/aarch64-devel/hikey-uefi/u-boot/include/configs/hikey.h
