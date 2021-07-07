---
title: Pixel 2 个人配置
date: 2019-08-06 15:33:05
tag: 
  Android
---



# Pixel 2 个人配置

本文记录了作者在配置自用的Pixel 2 欧版时学到的诸多经验与教训，一来便于自己今后查阅，二来希望能帮助到需要的人。若有疑问，可在下方评论区留言。
 <!-- more -->

## 解锁boot

略，这个网上的教程很多。

## 电信配置

众所周知周知，美版定制版无法电信4G，欧版可以破解电信4G。网上有修改基带文件的做法，但个人找到的最好方法是用Magisk：安装`Pixel 2 VoLTE Enabler for China`和`VoEmabler`模块。

**该方法在Android 10 上暂不能用！**

## 升级方式：

仅针对已经解锁bootloader并刷入了Magisk的Pixel 2！其他情况请酌情参考处理！

### adb OTA升级

Warning：该方法对于在Magisk安装时取消勾选了`keep AVB 2.0/dm-verity`的手机会无效，在 sideload 到40%左右时会终止，并会出现如下提示：

```
Failed to verify package compatibility (result 1): Runtime info and framework compatibility matrix are incompatible: AVB version 0.0 does not match framework matrix 1.0 Installation aborted.
```

虽说不会损坏手机，但也无法升级。仅仅安装Magisk不会出现类似情况。上述情况可以参照下一个方法解决。

>参考链接：  
><https://forum.xda-developers.com/pixel-2-xl/help/january-ota-wont-sideload-strange-error-t3729624>
><https://support.google.com/pixelphone/forum/AAAAb4-OgUs__Cown6bM4A/?hl=en&msgid=Dqpd5yT6AAAJ&gpf=d/msg/phone-by-google/__Cown6bM4A/Dqpd5yT6AAAJ>


**Note：虽说是OTA升级，但还是建议升级前先备份数据！**

#### Step 1: 下载OTA包
[下载地址](https://developers.google.com/android/ota)

下载完整的OTA包，放在电脑上。电脑需确定安装了adb（以下以macOS为例）。

PS: 以上网站似乎不需要科学上网，且使用迅雷下载一般可以满带宽。

#### Step 2: 进入Recovery，刷写OTA包

手机打开 Developer options , 并允许debug。


命令行输入`adb reboot recovery`，手机将进入 Recovery 模式。

再同时按住电源键和音量上键，将显示 Recovery 菜单：在菜单上选择` Apply update from ADB`。之后在电脑端输入：

```
adb sideload ota_file.zip   #此处 ota_file.zip 替换为下载的OTA包
```

等待进度加载完毕即可。


#### Step 3: 恢复Magisk

安装完OTA更新后，在 recovery 选择`reboot to bootloader`，将重启到bootloader；或者也可以选择重启，再在命令行输入：

```
adb reboot bootloader
```

临时 boot TWRP，以刷入Magisk：

**注：从大概2020年初开始，TWRP 无法通过 PIN 码作为密码解锁手机，因此建议在更新前或者更新重启后先删掉锁屏 PIN 码，再 
boot TWRP 并执行下属步骤（包括Step 4）。执行完后可以再加回 PIN 码等加密。**
```
fastboot boot [twrp-img-file].img
```

导入Magisk包到sdcard：

```
adb push [Magisk-vxxx.zip] /sdcard/   #将[Magisk-vxxx.zip]替换为最新的Magisk包
```

再通过 Recovery TWRP 的Install功能，安装导入的Magisk安装包：先输入锁屏的pin码解锁内部存储，再点击`Install`，并选中之前导入的包。

刷入完毕后如果不是电信（需要恢复LTE）就可以选择 reboot ，这时会提示要不要安装 TWRP 的app，如果不需要就选择不安装即可。

完成后建议取消勾选`USB debugging`，确保安全。

>参考链接：  
><https://www.jianshu.com/p/deac7ad245fc>
><https://www.jianshu.com/p/4017e9d6541c>

电信用户还是个老大难，需要增加 Step 4。

#### Step 4：恢复电信 LTE

这里使用的是备份&恢复EFS的方法。备份的工作具体如何做可以参考<https://isteed.cc/posts/4291184207/>，这里只讲恢复：

确保备份被放在了 `/sdcard/TWRP` 里，在TWRP内选择恢复。之后使用 adb 删除 `/data/vendor/radio/` （清除 radio 缓存）。再同上 reboot 即可。


### 刷写原厂包

#### Step 1: 下载原厂包

[下载地址](https://developers.google.com/android/images)

下载完整的原厂包，放在电脑上。电脑需确定安装了adb（以下以macOS为例）。

PS: 以上网站似乎不需要科学上网，且使用迅雷下载一般可以满带宽。

#### Step 2: 适当修改刷写工具

解压下载出来的原厂包，你将得到如下内容：

```
bootloader-walleye-xxxxxx.img
flash-all.bat
flash-all.sh
flash-base.sh
image-walleye-xxxxxx.zip
radio-walleye-xxxxx.img

#其中 walleye 是 Pixel 2 的代号，其他机型将不同！
```

打开`flash-all.sh`,将

```
fastboot -w update image-walleye-xxxxxx.zip
```
改为

```
fastboot update image-walleye-xxxxxx.zip
```
，并建议最好解压`image-walleye-xxxxxx.zip`并删除`userdata.img`来确保不会删除用户数据。

之后，将手机连接电脑，在解压后的文件夹内运行

```
adb reboot bootloader
```

等待出现bootloader界面后，再在解压后的文件夹内运行

```
./flash-all.sh
```

等待完成即可。

期间的输出可参考作者得到的输出：

```
ERROR: Couldn't create a device interface iterator: (e00002bd)
Sending 'bootloader_b' (38692 KB)                  OKAY [  0.267s]
Writing 'bootloader_b'                             (bootloader) Updating: partition:0   @00002000 sz=0000B000
(bootloader) Updating: partition:1   @0000D000 sz=0000B000
(bootloader) Updating: partition:2   @00018000 sz=0000B000
(bootloader) Updating: partition:3   @00023000 sz=0000B000
(bootloader) Updating: partition:4   @0002E000 sz=0000B000
(bootloader) Updating: partition:5   @00039000 sz=0000B000
(bootloader) Updating: pmic          @00044000 sz=0000D000
(bootloader) Updating: xbl           @00051000 sz=003A6000
(bootloader) Updating: abl           @003F7000 sz=00065000
(bootloader) Updating: cmnlib64      @0045C000 sz=0004A000
(bootloader) Updating: cmnlib        @004A6000 sz=00038000
(bootloader) Updating: devcfg        @004DE000 sz=0000F000
(bootloader) Updating: hyp           @004ED000 sz=00043000
(bootloader) Updating: keymaster     @00530000 sz=0004E000
(bootloader) Updating: tz            @0057E000 sz=001D3000
(bootloader) Updating: rpm           @00751000 sz=0003A000
(bootloader) Updating: hosd          @0078B000 sz=01E20000
(bootloader) Updating: lockbooter    @025AB000 sz=00016000
(bootloader) Updating: apdp          @025C1000 sz=00004000
(bootloader) Updating: msadp         @025C5000 sz=00004000
OKAY [  0.756s]
Finished. Total time: 1.025s
ERROR: Couldn't create a device interface iterator: (e00002bd)
rebooting into bootloader                          OKAY [  0.000s]
Finished. Total time: 0.000s
ERROR: Couldn't create a device interface iterator: (e00002bd)
Sending 'radio_b' (60372 KB)                       OKAY [  0.414s]
Writing 'radio_b'                                  (bootloader) Updating: modem         @00001000 sz=03AF3000
OKAY [  0.284s]
Finished. Total time: 0.700s
ERROR: Couldn't create a device interface iterator: (e00002bd)
rebooting into bootloader                          OKAY [  0.000s]
Finished. Total time: 0.000s
ERROR: Couldn't create a device interface iterator: (e00002bd)
extracting android-info.txt (0 MB) to RAM...
--------------------------------------------
Bootloader Version...: mw8998-002.0076.00
Baseband Version.....: g8998-00008-1902121845
Serial Number........: FA79W1A03730
--------------------------------------------
Checking product                                   OKAY [  0.001s]
Checking version-bootloader                        OKAY [  0.001s]
Checking version-baseband                          OKAY [  0.001s]
extracting boot.img (32 MB) to disk... took 0.186s
archive does not contain 'boot.sig'
archive does not contain 'boot_other.img'
extracting dtbo.img (8 MB) to disk... took 0.035s
archive does not contain 'dtbo.sig'
archive does not contain 'dt.img'
archive does not contain 'odm.img'
archive does not contain 'product.img'
archive does not contain 'product-services.img'
archive does not contain 'recovery.img'
archive does not contain 'super.img'
extracting system.img (2210 MB) to disk... took 19.792s
archive does not contain 'system.sig'
extracting system_other.img (321 MB) to disk... took 3.013s
archive does not contain 'system.sig'
extracting vbmeta.img (0 MB) to disk... took 0.000s
archive does not contain 'vbmeta.sig'
extracting vendor.img (350 MB) to disk... took 3.165s
archive does not contain 'vendor.sig'
archive does not contain 'vendor_other.img'
Sending 'boot_b' (32768 KB)                        OKAY [  0.224s]
Writing 'boot_b'                                   OKAY [  0.157s]
Sending 'dtbo_b' (8192 KB)                         OKAY [  0.043s]
Writing 'dtbo_b'                                   OKAY [  0.041s]
Sending sparse 'system_b' 1/5 (524284 KB)          OKAY [  3.749s]
Writing sparse 'system_b' 1/5                      OKAY [  0.000s]
Sending sparse 'system_b' 2/5 (524284 KB)          OKAY [ 13.749s]
Writing sparse 'system_b' 2/5                      OKAY [  0.000s]
Sending sparse 'system_b' 3/5 (524284 KB)          OKAY [  6.039s]
Writing sparse 'system_b' 3/5                      OKAY [  0.000s]
Sending sparse 'system_b' 4/5 (521176 KB)          OKAY [  8.999s]
Writing sparse 'system_b' 4/5                      OKAY [  0.000s]
Sending sparse 'system_b' 5/5 (169428 KB)          OKAY [  3.590s]
Writing sparse 'system_b' 5/5                      OKAY [  0.000s]
Sending 'system_a' (329560 KB)                     OKAY [  3.069s]
Writing 'system_a'                                 OKAY [  0.000s]
Sending 'vbmeta_b' (4 KB)                          OKAY [  3.259s]
Writing 'vbmeta_b'                                 OKAY [  0.001s]
Sending 'vendor_b' (359288 KB)                     OKAY [  2.449s]
Writing 'vendor_b'                                 OKAY [  0.000s]
Setting current slot to 'b'                        OKAY [  1.731s]
Rebooting
Finished. Total time: 73.327s
```

此后系统会自动重启，启动时间略长，不过应该在15min之内。重启之后的系统是没有root/Magisk的，需要再按照 “adb OTA升级” 方法中的Step 3 恢复Magisk和 Step 4 恢复电信LTE。

完成后建议取消勾选`USB debugging`，确保安全。

>参考链接：  
>[Pixel 手机升级保留基带和数据](https://www.jianshu.com/p/b62e174f5e44)


