### u-boot-2016.03/drivers/mmc/hi6220_dw_mmc.c
```
#define MMC0_DEFAULT_FREQ       25000000 /* 25MHz */
```
## Boot from u-boot shell
### From USB Drive
```
setenv bootargs console=ttyAMA3,115200 root=/dev/sda2 rootwait rw quiet efi=noruntime
usb reset
usb part
fatls usb 0:1

fatload usb 0:1 0x00080000 image

fatload usb 0:1 0x02000000 hi6220-hikey.dtb

booti 0x00080000 - 0x02000000
```
### From TFTPD

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

## Make kernel image for u-boot
On a running HiKey, if mkimage is absent, then `sudo apt-get install u-boot-tools`.
```
mkimage -A arm64 -O linux -T kernel -C none -a 0x00080000 -e 0x02000000 -n 'Linux linaro-alip-1 3.18.0-linaro-hikey' -d /boot/Image uImage
```
## mmcinfo when MMC is recoginzed
```
=> mmcinfo
Device: HiKey DWMMC
Manufacturer ID: 74
OEM: 4a60
Name: USDU1 
Tran Speed: 50000000
Rd Block Len: 512
SD version 3.0
High Capacity: Yes
Capacity: 15 GiB
Bus Width: 4-bit
Erase Group Size: 512 Bytes
```
