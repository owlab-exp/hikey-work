## Prepare Tool Chain
```
mkdir working
cd working
lftp -c get http://releases.linaro.org/15.02/components/toolchain/binaries/aarch64-linux-gnu/gcc-linaro-4.9-2015.02-3-x86_64_aarch64-linux-gnu.tar.xz
lftp -c get http://releases.linaro.org/15.02/components/toolchain/binaries/arm-linux-gnueabihf/gcc-linaro-4.9-2015.02-3-x86_64_arm-linux-gnueabihf.tar.xz
mkdir arm64-tc arm-tc
tar --strip-components=1 -C ${PWD}/arm64-tc -xvf gcc-linaro-4.9-2015.02-3-x86_64_aarch64-linux-gnu.tar.xz
tar --strip-components=1 -C ${PWD}/arm-tc -xvf gcc-linaro-4.9-2015.02-3-x86_64_arm-linux-gnueabihf.tar.xz
export PATH="${PWD}/arm-tc/bin:${PWD}/arm64-tc/bin:$PATH"
export CC=${PWD}/arm64-tc/bin/aarch64-linux-gnu-
```
## Build U-Boot
### Downlaod
```
lftp -c get ftp://ftp.denx.de/pub/u-boot/u-boot-2016.03.tar.bz2
mkdir u-boot
tar --strip-components=1 -C ${PWD}/u-boot -xvjf u-boot-2016.03.tar.bz2
```
### Change HiKey specific configurations for 2G RAM/8G EMMC 
```
vi ./u-boot/include/configs/hikey.h
```
| Before | After |
|--------|-------|
|#define CONFIG_BOOTARGS "console=**ttyAMA0**,115200n8 root=/dev/mmcblk0p9 rw" | #define CONFIG_BOOTARGS "console=**ttyAMA3**,115200n8 root=/dev/mmcblk0p9 rw" |
|#define PHYS_SDRAM_1_SIZE        0x3EFFFFFF /* 1024M - 16M */| #define PHYS_SDRAM_1_SIZE       0x7EFFFFFF /* 2048M - 16M |
| | #define CONFIG_CMD_EXT4 /* EXT4 support */ |
| | #define CONFIG_SYS_BOOTM_LEN (16 << 20) /* Increase max image size */ |
| | #define CONFIG_DWC2_DFLT_SPEED_FULL |

### Build
Refer to the include README (`./u-boot/board/hisilicon/hikey/README`)
```
mkdir -p ../bin
cd u-boot
make CROSS_COMPILE=${CC} hikey_config
make CROSS_COMPILE=${CC}
cd ..
cp ./u-boot/u-boot.bin ./bin/u-boot-hikey.bin
```
## ARM Trusted Firmware (ATF) & l-loader
This is also from the README included in u-boot source.

### Download BL30 mcu binary
```
wget -P ./bin https://builds.96boards.org/releases/hikey/linaro/binaries/15.05/mcuimage.bin
```
### Build ATF
```
https://github.com/96boards/arm-trusted-firmware.git
cd ./arm-trusted-firmware
cp ../u-boot/board/hisilicon/hikey/build-tf.mak .
make -f  build-tf.mak build
cd ..
```
### Get l-loader & build ptable.img
```
git clone https://github.com/96boards/l-loader.git
cd l-loader
ln -s ../bin/bl1-hikey.bin bl1.bin
ln -s ../bin/fip-hikey.bin fip.bin
arm-linux-gnueabihf-gcc -c -o start.o start.S
arm-linux-gnueabihf-gcc -c -o debug.o debug.S
arm-linux-gnueabihf-ld -Bstatic -Tl-loader.lds -Ttext 0xf9800800 start.o debug.o -o loader
arm-linux-gnueabihf-objcopy -O binary loader temp
python gen_loader.py -o ../bin/l-loader.bin --img_loader=temp --img_bl1=bl1.bin
sudo PTABLE=linux-8g bash -x generate_ptable.sh
```
The last output will be like;
```
+ PTABLE=linux-8g
+ SECTOR_SIZE=512
++ mktemp /tmp/linux-8g.XXXXXX
+ TEMP_FILE=/tmp/linux-8g.CgCoGp
+ case ${PTABLE} in
+ SECTOR_NUMBER=15269888
++ expr 15269888 - 33
+ BK_PTABLE_LBA=15269855
+ echo 15269855
15269855
+ case ${PTABLE} in
+ dd if=/dev/zero of=/tmp/linux-8g.CgCoGp bs=512 count=15269888
15269888+0 records in
15269888+0 records out
7818182656 bytes (7.8 GB) copied, 69.99 s, 112 MB/s
+ sgdisk -U 2CB85345-6A91-4043-8203-723F0D28FBE8 -v /tmp/linux-8g.CgCoGp
Creating new GPT entries.

No problems found. 15269821 free sectors (7.3 GiB) available in 1
segments, the largest of which is 15269821 (7.3 GiB) in size.
Warning: The kernel is still using the old partition table.
The new table will be used at the next reboot.
The operation has completed successfully.
+ sgdisk -n 1:0:+1M -t 1:0700 -u 1:496847AB-56A1-4CD5-A1AD-47F4ACF055C9 -c 1:vrl /tmp/linux-8g.CgCoGp
Warning: The kernel is still using the old partition table.
The new table will be used at the next reboot.
The operation has completed successfully.
+ sgdisk -n 2:0:+1M -t 2:0700 -u 2:61A36FC1-8EFB-4899-84D8-B61642EFA723 -c 2:vrl_backup /tmp/linux-8g.CgCoGp
Warning: The kernel is still using the old partition table.
The new table will be used at the next reboot.
The operation has completed successfully.
+ sgdisk -n 3:0:+1M -t 3:0700 -u 3:65007411-962D-4781-9B2C-51DD7DF22CC3 -c 3:mcuimage /tmp/linux-8g.CgCoGp
Warning: The kernel is still using the old partition table.
The new table will be used at the next reboot.
The operation has completed successfully.
+ sgdisk -n 4:0:+8M -t 4:EF02 -u 4:496847AB-56A1-4CD5-A1AD-47F4ACF055C9 -c 4:fastboot /tmp/linux-8g.CgCoGp
Warning: The kernel is still using the old partition table.
The new table will be used at the next reboot.
The operation has completed successfully.
+ sgdisk -n 5:0:+2M -t 5:0700 -u 5:00354BCD-BBCB-4CB3-B5AE-CDEFCB5DAC43 -c 5:nvme /tmp/linux-8g.CgCoGp
Warning: The kernel is still using the old partition table.
The new table will be used at the next reboot.
The operation has completed successfully.
+ sgdisk -n 6:0:+64M -t 6:EF00 -u 6:5C0F213C-17E1-4149-88C8-8B50FB4EC70E -c 6:boot /tmp/linux-8g.CgCoGp
Warning: The kernel is still using the old partition table.
The new table will be used at the next reboot.
The operation has completed successfully.
+ sgdisk -n 7:0:+256M -t 7:0700 -u 7:BED8EBDC-298E-4A7A-B1F1-2500D98453B7 -c 7:reserved /tmp/linux-8g.CgCoGp
Warning: The kernel is still using the old partition table.
The new table will be used at the next reboot.
The operation has completed successfully.
+ sgdisk -n 8:0:+256M -t 8:8301 -u 8:A092C620-D178-4CA7-B540-C4E26BD6D2E2 -c 8:cache /tmp/linux-8g.CgCoGp
Warning: The kernel is still using the old partition table.
The new table will be used at the next reboot.
The operation has completed successfully.
+ sgdisk -n -E -t 9:8300 -u 9:FC56E345-2E8E-49AE-B2F8-5B9D263FE377 -c 9:system /tmp/linux-8g.CgCoGp
Warning: The kernel is still using the old partition table.
The new table will be used at the next reboot.
The operation has completed successfully.
+ dd if=/tmp/linux-8g.CgCoGp of=prm_ptable.img bs=512 count=34
34+0 records in
34+0 records out
17408 bytes (17 kB) copied, 0.00033875 s, 51.4 MB/s
+ dd if=/tmp/linux-8g.CgCoGp of=sec_ptable.img skip=15269855 bs=512 count=33
33+0 records in
33+0 records out
16896 bytes (17 kB) copied, 0.000280356 s, 60.3 MB/s
+ rm -f /tmp/linux-8g.CgCoGp
```
### Generate ptable.img
This is adapted from `u-boot/board/hisilicon/hikey/README
```
python gen_loader.py -o ../bin/ptable.img --img_prm_ptable=./prm_ptable.img --img_sec_ptable=./sec_ptable.img
cd ..
```

## Flashing all
### Get Recovery tool
```
git clone https://github.com/96boards/burn-boot.git
```
### Get nvme.img
```
wget -P ./bin https://builds.96boards.org/releases/hikey/linaro/binaries/latest/nvme.img
```
### Get boo-fat.uefi.img and debian
```
wget https://builds.96boards.org/releases/hikey/linaro/debian/latest/boot-fat.uefi.img.gz
wget https://builds.96boards.org/releases/hikey/linaro/debian/latest/hikey-jessie_developer_20151130-387-8g.emmc.img.gz
gunzip boot-fat.uefi.img.gz
gunzip hikey-jessie_developer_20151130-387-8g.emmc.img.gz
```
### Prepare the board
1. Power off the board
2. Close jumper 1-2 and jumpter 3-4
3. Connect USB OTG port to you PC
4. Power on

### Run the recovery tool
The /dev/ttyUSB0 might be different in your PC. For example, /dev/ttyUSB1.
```
sudo ./burn-boot/hisi-idt.py -d /dev/ttyUSB0 --img1=./bin/l-loader.bin
```
if succeeded, `sudo fastboot devices` will show a message like following:
```
0123456789ABCDEF	fastboot
```
### Flash images to the eMMC.
```
sudo fastboot flash ptable ./bin/ptable.img
sudo fastboot flash fastboot ./bin/fip-hikey.bin
sudo fastboot flash nvme ./bin/nvme.img
```
### And also flash Linux
```
sudo fastboot flash boot boot-fat.uefi.img
sudo fastboot flash system hikey-jessie_developer_20151130-387-8g.emmc.img
```
### Post flash task
1. Power off the board
2. Open jumper 3-4
3. Disconnect the OTG cable from the board
4. Power on
