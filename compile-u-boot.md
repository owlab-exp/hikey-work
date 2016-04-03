
# Compile u-boot on Hikey

    ```
    mkdir working
    cd working
    lftp -c get ftp://ftp.denx.de/pub/u-boot/u-boot-latest.tar.bz2
    tar xvjf u-boot-latest.tar.bz2
    cd u-boot-2016.01
    make hikey_defconfig
    make
    ```
# Notes
- As of writing, the u-boot-latest.tar.bz2 -> u-boot-2016.01.tar.bz2
