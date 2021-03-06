## Arm64 Grub on U-Boot

## ARM Grub on U-Boot
### grub-core/Makefile.core.def
#### arm_efi
```
  arm_efi_ldflags          = '-Wl,-r,-d';
  arm_efi_stripflags       = '--strip-unneeded -K start -R .note -R .comment -R .note.gnu.gold-version';
```
```
  arm_efi_startup = kern/arm/efi/startup.S;
```
#### arm_uboot
```
  arm_uboot_ldflags       = '-Wl,-r,-d';
  arm_uboot_stripflags    = '--strip-unneeded -K start -R .note -R .comment -R .note.gnu.gold-version';
```
```
  arm_uboot_startup = kern/arm/uboot/startup.S;
```
#### arm_efi > arm_uboot
```
  arm_efi = kern/arm/efi/init.c;
  arm_efi = kern/arm/efi/misc.c;
```
#### module - name: boot
```
enable = arm_efi;
...
enable = arm_uboot;
```
#### module - name: reboot
```
arm_efi = lib/efi/reboot.c;
...
uboot = lib/uboot/reboot.c;
```
#### module - name: mmap
```
enable = arm_efi;
```
#### module - name: setjmp
```
  extra_dist = lib/arm/setjmp.S;
```
#### module - name: linux
```
  arm = loader/arm/linux.c;
```
#### arm_uboot only sections
kernel
```
  arm_uboot_ldflags       = '-Wl,-r,-d';
  arm_uboot_stripflags    = '--strip-unneeded -K start -R .note -R .comment -R .note.gnu.gold-version';
...
  arm_uboot_startup = kern/arm/uboot/startup.S;
```
module = boot
```
  enable = arm_uboot;
```
#### (arm64_uboot sections)
```
  arm64_uboot_ldflags    =
  arm64_uboot_stripflags =
...
  arm64_uboot_startup = kern/arm64/uboot/startup.S
```
module = boot
```
  enable = arm64_boot;
```
