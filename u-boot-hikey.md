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
