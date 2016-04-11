### Call Tracing of USB Speed Initialization - `struct dwc2_hsotg *hsotg`
- Last: `dwc2_init_fs_ls_pclk_sel` > `val = HCFG_FSLSPCLKSEL_48_MHZ` or `val = HCFG_FSLSPCLKSEL_30_60_MHZ`
- Cond: 
     95     if ((hsotg->hw_params.hs_phy_type == GHWCFG2_HS_PHY_TYPE_ULPI &&
     96          hsotg->hw_params.fs_phy_type == GHWCFG2_FS_PHY_TYPE_DEDICATED &&
     97          hsotg->core_params->ulpi_fs_ls > 0) ||
     98         hsotg->core_params->phy_type == DWC2_PHY_TYPE_PARAM_FS) {
     99         /* Full speed PHY */
     100         val = HCFG_FSLSPCLKSEL_48_MHZ;
     101     } else {
     102         /* High speed PHY running at full speed or high speed */
     103         val = HCFG_FSLSPCLKSEL_30_60_MHZ;
     104     }
- Prev: `dwc2_core_host_init` > `dwc2_init_fs_ls_pclk_sel(hsotg)`
- Prev: 

### 96boards-linux/drivers/usb/dwc2/core.c
By reviewing **u-boot/drivers/usb/host/dwc2.c** and **dwc2.h**. **dwc2.h** has a line `#undef CONFIG_DWC2_DFLT_SPEED_FULL      /* Do not force DWC2 to FS */`.
```
 662     /* Initialize Host Configuration Register */
 663     dwc2_init_fs_ls_pclk_sel(hsotg);
 664     /* HJ to make USB default speed HIGH, not FULL
 665     if (hsotg->core_params->speed == DWC2_SPEED_PARAM_FULL) {
 666         hcfg = readl(hsotg->regs + HCFG);
 667         hcfg |= HCFG_FSLSSUPP;
 668         writel(hcfg, hsotg->regs + HCFG);
 669     }
 670     */
 ```
 ==> USB default speed to HIGH, not FULL of previous!
 
### u-boot-2016.03/drivers/mmc/hi6220_dw_mmc.c
```
#define MMC0_DEFAULT_FREQ       25000000 /* 25MHz */
```
==> Bad result!
```
15 /* from 
 16  * git clone https://github.com/96boards/u-boot/tree/hikey-rebase
 17  * git checkout hikey-rebase
 18  */
 19 #define DWMMC_MAX_FREQ          50000000 
 20 #define DWMMC_MIN_FREQ          378000
 21 
 22 /* Source clock is configured to 100MHz by ATF bl1*/
 23 #define MMC0_DEFAULT_FREQ       100000000 
 24 #define MMC1_DEFAULT_FREQ       50000000  
```
```
 61     switch(index) {
 62        case 0:
 63           host->bus_hz = MMC0_DEFAULT_FREQ;
 64           printf("DWMMC%d's default frequency -> %d\n", index, MMC0_DEFAULT_FREQ);
 65           break;
 66        case 1: /* SD Card */
 67           host->bus_hz = MMC1_DEFAULT_FREQ;
 68           printf("DWMMC%d's default frequency -> %d\n", index, MMC1_DEFAULT_FREQ);
 69           break;
 70        default:
 71           host->bus_hz = MMC0_DEFAULT_FREQ;
 72     }
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
