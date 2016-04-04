
# Compile u-boot for Hikey
## on Debian Jessie 8 on Hikey

```
mkdir -p working
cd working
lftp -c get ftp://ftp.denx.de/pub/u-boot/u-boot-latest.tar.bz2
tar xvjf u-boot-latest.tar.bz2
cd u-boot-2016.01
make hikey_defconfig
make
```
### Notes
- As of writing, the u-boot-latest.tar.bz2 -> u-boot-2016.01.tar.bz2

## Cross-compilation on Ubuntu 14.04 x86_64
```
mkdir working
cd working
lftp -c get http://releases.linaro.org/15.02/components/toolchain/binaries/aarch64-linux-gnu/gcc-linaro-4.9-2015.02-3-x86_64_aarch64-linux-gnu.tar.xz
mkdir arm64-tc
tar --strip-components=1 -C ${PWD}/arm64-tc -xvf gcc-linaro-4.9-2015.02-3-x86_64_aarch64-linux-gnu.tar.xz
export CC=${PWD}/arm64-tc/bin/aarch64-linux-gnu-
lftp -c get ftp://ftp.denx.de/pub/u-boot/u-boot-2016.03.tar.bz2
mkdir u-boot
tar --strip-components=1 -C ${PWD}/u-boot -xvjf u-boot-2016.03.tar.bz2
cd u-boot
make CROSS_COMPILE=${CC} hikey_defconfig
make CROSS_COMPILE=${CC}
```
