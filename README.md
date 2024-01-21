# CMCC_RAX3000M
RAX3000M是中国移动于2023年推出的一款高性价比路由器。

不仅美观( https://www.red-dot.org/project/rax3000m-wifi6-65030 )，
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
```bash
# 1. 先在网页上登录到路由器管理后台，导出配置文件

# 2. 从系统日志中发现暴露了密钥
export CMCC_PASSWD='#RaX30O0M@!$'

# 3. 在本地对配置文件解密解压
openssl aes-256-cbc -d -pbkdf2 -k "$CMCC_PASSWD" -in cfg_export_config_file.conf | tar -zxvf -

# 4. 编辑 etc/config/dropbear 文件以开启SSH
# 编辑 etc/shadow 文件以置空root密码

# 5. 重新压包加密
sudo tar -zcvf - etc | openssl aes-256-cbc -pbkdf2 -k "$CMCC_PASSWD" -out new_config_file.conf

# 6. 登录到路由器管理后台，导入新的配置文件

# 7. 确保你的电脑处于192.168.10.1/24网段，使用root账户无需密码即可登录
ssh root@192.168.10.1
```
参考自：
- https://www.right.com.cn/forum/thread-8302668-1-1.html
- https://www.right.com.cn/forum/thread-8316001-1-1.html
- https://www.right.com.cn/forum/thread-8320480-1-1.html

### 备份原厂固件
如果你的是nand版的，由于flash较小，在备份时最好打包一个就导出一个到本地，并删除。
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
> 暂不知道为啥要备份这些文件

如果你使用`scp`下载文件时遇到：
```text
ash: /usr/libexec/sftp-server: not found
scp: Connection closed
```
是因为你的scp版本默认使用sftp协议，但这个路由器系统默认没有开启sftp，添加`- O`选项即可解决。详见：https://forum.openwrt.org/t/ash-usr-libexec-sftp-server-not-found-when-using-scp/125772



### 刷入uBoot
这里推荐使用hanwckf制作的bootloader。
- https://github.com/hanwckf/bl-mt798x
- https://github.com/hanwckf/bl-mt798x/releases/tag/20231124

也可以自行编译，可参考本项目的`.github/workflows`。

```bash
# 1. ssh登录到路由器操作系统
# 2. 上传适用于cmcc_rax3000m的文件到/tmp目录下
# 3. 使用mtd命令写入
mtd write /tmp/mt7981_cmcc_rax3000m-fip-fixed-parts.bin FIP
```
重启路由器，并按住背面的reset直至指示灯变绿，此时路由器使用的就是我们上传uboot了。

### 获取固件
这款路由器对市面上主流的OpenWRT系统都兼容良好。
推荐根据自身需求自行编译系统固件。

编译过程可参考本项目的`.github/workflows`。

### 刷入OpenWRT
刷入uboot后，可以轻松的借助其web页面(192.168.1.1)刷入系统固件。
详见：
[hanwckf's blog](https://cmi.hanwckf.top/p/mt798x-uboot-usage/#failsafe-webui%E4%BD%BF%E7%94%A8%E8%AF%B4%E6%98%8E)
