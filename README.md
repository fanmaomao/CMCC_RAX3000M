# CMCC_RAX3000M
RAX3000M是中国移动于2023年推出的一款高性价比路由器。

不仅美观(https://www.red-dot.org/project/rax3000m-wifi6-65030)，
硬件配置更是家用百元级产品中的佼佼者：
```text
SoC: MediaTek Filogic 820 MT7981B 双核1.3GHz
RAM: 512MB
Flash: NAND版本128MB；EMMC版本64GB
WiFi: WiFi6-AX3000M规格
四个千兆网口
一个USB3.0插口
```

更不可思议的是这款路由器支持**免拆刷机**！[滑稽+狗头]

## 刷机
### 开启SSH
参考自：
- https://www.right.com.cn/forum/thread-8320480-1-1.html
- https://www.right.com.cn/forum/thread-8316001-1-1.html

```bash
export CMCC_PASSWD=$CmDc#RaX30O0M@\!$
# 解包
openssl aes-256-cbc -d -pbkdf2 -k $CMCC_PASSWD -in cfg_export_config_file.conf -out temp.tar.gz

tar -xzf temp.tar.gz

# 重新压包
tar -zcf - etc/ | openssl aes-256-cbc -pbkdf2 -k $CMCC_PASSWD -out export_config_new.conf
```

### 备份原厂固件
```bash
dd if=/dev/mtd0 | gzip >/tmp/mtd0_spi0.0.bin.gz
dd if=/dev/mtd1 of=/tmp/mtd1_BL2.bin
dd if=/dev/mtd2 of=/tmp/mtd2_u-boot-env.bin
dd if=/dev/mtd3 of=/tmp/mtd3_Factory.bin
dd if=/dev/mtd4 of=/tmp/mtd4_mtd4_FIP.bin
dd if=/dev/mtd5 of=/tmp/mtd5_ubi.bin
dd if=/dev/mtd6 of=/tmp/mtd6_plugins.bin
dd if=/dev/mtd7 of=/tmp/mtd7_fwk.bin
dd if=/dev/mtd8 of=/tmp/mtd8_fwk2.bin
```
### 刷入uBoot
```bash
mtd write /tmp/mt7981_cmcc_rax3000m-fip-fixed-parts.bin FIP
```

### 刷入OpenWRT



