## Hi6220V100 Application Function#Memory Control
```
Hi6220 V100 integrates the following three MMC modules:
z MMC0: Connects to eMMC and supports 1-bit, 4-bit, and 8-bit modes.
z MMC1: Connects to the SD 3.0 card and supports 1-bit and 4-bit modes (3 V and 1.8 V
supported for the I/O device).
z MMC2: Connects to the slave device and supports 1-bit and 4-bit modes.
```

```
4-bit and 8-bit HS200 modes defined in the eMMC 4.5 protocol, maximum interface
frequency of 150 MHz
```

```
4-bit DDR transfers defined in the SD 3.0 and MMC 4.41 protocols
```

## linux/arch/arm64/boot/dts/hi6220.dtsi
```
1029 dwmmc_0: dwmmc0@f723d000 {
1030     │   │   │compatible = "hisilicon,hi6220-dw-mshc";
1031     │   │   │num-slots = <0x1>;
1032     │   │   │board-mmc-bus-clk = <0x0>;
1033     │   │   │cap-mmc-highspeed;
1034     │   │   │non-removable;
1035     │   │   │reg = <0x0 0xf723d000 0x0 0x1000>;
1036     │   │   │interrupts = <0x0 0x48 0x4>;
1037     │   │   │clocks = <&clock_sys HI6220_MMC0_CIUCLK>, <&clock_sys HI6220_MMC0_CLK>;
1038     │   │   │clock-names = "ciu", "biu";
1039     │   │   │vmmc-supply = <&ldo19>;
1040     │   │};
1041 
1042 dwmmc_1: dwmmc1@f723e000 {
1043         compatible = "hisilicon,hi6220-dw-mshc";
1044         num-slots = <0x1>;
1045         board-mmc-bus-clk = <0x0>;
1046         card-detect-delay = <200>;
1047         hisilicon,peripheral-syscon = <&ao_ctrl>;
1048         cap-sd-highspeed;
1049         sd-uhs-sdr12;
1050         sd-uhs-sdr25;
1051         sd-uhs-sdr50;
1052         reg = <0x0 0xf723e000 0x0 0x1000>;
1053         interrupts = <0x0 0x49 0x4>;
1054         clocks = <&clock_sys HI6220_MMC1_CIUCLK>, <&clock_sys HI6220_MMC1_CLK>;
1055         clock-names = "ciu", "biu";
1056         vqmmc-supply = <&ldo7>;
1057         vmmc-supply = <&ldo10>;
1058     };
```
## linux/drivers/clk/hisilicon/clk-hi6220.c 
```
 33     { HI6220_MMC1_PAD,  "mmc1_pad", NULL, CLK_IS_ROOT, 100000000, },
 34     { HI6220_MMC2_PAD,  "mmc2_pad", NULL, CLK_IS_ROOT, 100000000, },
 35     { HI6220_MMC0_PAD,  "mmc0_pad", NULL, CLK_IS_ROOT, 200000000, },
```

```
 46 static struct hisi_fixed_factor_clock hi6220_fixed_factor_clks[] __initdata = {
 47     { HI6220_300M,         "clk_300m",    "syspll",          1, 4, 0, },
 48     { HI6220_150M,         "clk_150m",    "clk_300m",        1, 2, 0, },
 49     { HI6220_PICOPHY_SRC,  "picophy_src", "clk_150m",        1, 4, 0, },
 50     { HI6220_MMC0_SRC_SEL, "mmc0srcsel",  "mmc0_sel",        1, 8, 0, },
 51     { HI6220_MMC1_SRC_SEL, "mmc1srcsel",  "mmc1_sel",        1, 8, 0, },
 52     { HI6220_MMC2_SRC_SEL, "mmc2srcsel",  "mmc2_sel",        1, 8, 0, },
 53     { HI6220_VPU_CODEC,    "vpucodec",    "codec_jpeg_aclk", 1, 2, 0, },
 54     { HI6220_MMC0_SMP,     "mmc0_sample", "mmc0_sel",        1, 8, 0, },
 55     { HI6220_MMC1_SMP,     "mmc1_sample", "mmc1_sel",        1, 8, 0, },
 56     { HI6220_MMC2_SMP,     "mmc2_sample", "mmc2_sel",        1, 8, 0, },
 57 };
```

```
166 static struct hisi_gate_clock hi6220_separated_gate_clks_sys[] __initdata = {
167     { HI6220_MMC0_CLK,      "mmc0_clk",      "mmc0_rst_clk",       CLK_SET_RATE_PARENT|CLK_IGNORE_UNUSED, 0x200, 0,  0, },
168     { HI6220_MMC0_CIUCLK,   "mmc0_ciuclk",   "mmc0_smp_in",    CLK_SET_RATE_PARENT|CLK_IGNORE_UNUSED, 0x200, 0,  0, },
169     { HI6220_MMC1_CLK,      "mmc1_clk",      "mmc1_rst_clk",       CLK_SET_RATE_PARENT|CLK_IGNORE_UNUSED, 0x200, 1,  0, },
170     { HI6220_MMC1_CIUCLK,   "mmc1_ciuclk",   "mmc1_smp_in",    CLK_SET_RATE_PARENT|CLK_IGNORE_UNUSED, 0x200, 1,  0, },
171     { HI6220_MMC2_CLK,      "mmc2_clk",      "mmc2_rst_clk",       CLK_SET_RATE_PARENT|CLK_IGNORE_UNUSED, 0x200, 2,  0, },
172     { HI6220_MMC2_CIUCLK,   "mmc2_ciuclk",   "mmc2_smp_in",    CLK_SET_RATE_PARENT|CLK_IGNORE_UNUSED, 0x200, 2,  0, },
```

```
228 static struct hi6220_divider_clock hi6220_div_clks_sys[] __initdata = {
229     { HI6220_CLK_BUS,     "clk_bus",     "clk_300m",      CLK_SET_RATE_PARENT, 0x490, 0,  4, 7, },
230     { HI6220_MMC0_DIV,    "mmc0_div",    "mmc0_syspll",   CLK_SET_RATE_PARENT, 0x494, 0,  6, 7, },
231     { HI6220_MMC1_DIV,    "mmc1_div",    "mmc1_syspll",   CLK_SET_RATE_PARENT, 0x498, 0,  6, 7, },
232     { HI6220_MMC2_DIV,    "mmc2_div",    "mmc2_syspll",   CLK_SET_RATE_PARENT, 0x49c, 0,  6, 7, },
233     { HI6220_HIFI_DIV,    "hifi_div",    "hifi_sel",      CLK_SET_RATE_PARENT, 0x4a0, 0,  4, 7, },
234     { HI6220_BBPPLL0_DIV, "bbppll0_div", "bbppll_sel",    CLK_SET_RATE_PARENT, 0x4a0, 8,  6, 15,},
235     { HI6220_CS_DAPB,     "cs_dapb",     "picophy_src",   CLK_SET_RATE_PARENT, 0x4a0, 24, 2, 31,},
236     { HI6220_CS_ATB_DIV,  "cs_atb_div",  "cs_atb_syspll", CLK_SET_RATE_PARENT, 0x4a4, 0,  4, 7, },
237 };
```
## linux/drivers/clk/hisilicon/clk-hi6220.c
```
 30 static struct hisi_fixed_rate_clock hi6220_fixed_rate_clks[] __initdata = {
 31     { HI6220_REF32K,    "ref32k",   NULL, CLK_IS_ROOT, 32764,     },
 32     { HI6220_CLK_TCXO,  "clk_tcxo", NULL, CLK_IS_ROOT, 19200000,  },
 33     { HI6220_MMC1_PAD,  "mmc1_pad", NULL, CLK_IS_ROOT, 100000000, },
 34     { HI6220_MMC2_PAD,  "mmc2_pad", NULL, CLK_IS_ROOT, 100000000, },
 35     { HI6220_MMC0_PAD,  "mmc0_pad", NULL, CLK_IS_ROOT, 200000000, },
 ```
