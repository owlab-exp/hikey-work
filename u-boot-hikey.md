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
cd u-boot
```
### Change HiKey specific configurations for 2G RAM/8G EMMC 
```
vi include/configs/hikey.h
```
| Before | After |
|--------|-------|
|#define CONFIG_BOOTARGS "console=**ttyAMA0**,115200n8 root=/dev/mmcblk0p9 rw" | #define CONFIG_BOOTARGS "console=**ttyAMA3**,115200n8 root=/dev/mmcblk0p9 rw" |
|#define PHYS_SDRAM_1_SIZE        0x3EFFFFFF /* 1024M - 16M */| #define PHYS_SDRAM_1_SIZE       0x7EFFFFFF /* 2048M - 16M |

(According to `./board/hisilicon/hikey/README`)
```
mkdir -p ../bin
make CROSS_COMPILE=${CC} hikey_defconfig
make CROSS_COMPILE=${CC}
cp u-boot.bin ../bin/u-boot-hikey.bin
```
