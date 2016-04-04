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
```
lftp -c get ftp://ftp.denx.de/pub/u-boot/u-boot-2016.03.tar.bz2
mkdir u-boot
tar --strip-components=1 -C ${PWD}/u-boot -xvjf u-boot-2016.03.tar.bz2
cd u-boot
make CROSS_COMPILE=${CC} hikey_defconfig
make CROSS_COMPILE=${CC}
```
