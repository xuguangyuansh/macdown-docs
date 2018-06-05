# 我玩黑苹果 — 5、Config.plist 文件详解

## 1、目标机准备
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;准备一块空的磁盘：<br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;1）在Win 或 pe 下用 DiskGenius 转为 GUID 分区表，由于 10.11 后的 OS X，安装时的 磁盘工具 已不再支持该转换，因此需在 win 下完成；<br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;2）删除所有分区；

## 2、安装前注意
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;1）设置 UEFI 开机的，以下文件自行 四选一 使用，其余全部移除，或全部移除。

```
/EFI/CLOVER/drivers64UEFI/OsxLowMemFixDrv-64.efi
/EFI/CLOVER/drivers64UEFI/OsxAptioFix2Drv-64.efi
/EFI/CLOVER/drivers64UEFI/OsxAptioFixDrv-64.efi
/EFI/CLOVER/drivers64UEFI/OsxAptioFixDrv256m-64.efi
```
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;2）在 config.plist 中删掉 slide=XXX (XXX代表数值)。如果仍旧无法启动, 表明内核请求分配的空间仍旧太大, 将 CsrActiveConfig 设置成 0x67，解决以下问题：

```
OsxAptioFixDrv-64.efi
Error - requested memory exceeds our allocated relocation block
OsxAptioFix2Drv-64.efi
Error loading kernel cache
```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;没出问题前先不要改动

```
<key>RtVariables</key>
<dict>
	<key>CsrActiveConfig</key>
	<string>0x67</string>
</dict>
```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;3）台式机关闭安全启动 台式路径 boot -> secure boot -> os type -> other os<br/>

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4）个别BIOS默认不是开启 UEFI   只要看到有 EFI 字眼都是可以支持 UEFI 的。<br/>

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;5）fakeSMC.KEXT 这个驱动是必须要的，还有建议加 USB 驱动。<br/>

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;6）台式机 有独显的安装需要加代码屏蔽显卡，安装好进系统后再删除 `nv_disable=1` 添加 `nvda_drv=1` 加载 NV WebDriver 驱动。

```
<key>Boot</key>
	<dict>
		<key>Arguments</key>
		<string>dart=0 npci=0x2000 kext-dev-mode=1 rootless=0 nv_disable=1 </string>
```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;7）到引导界面没有看到 MAC 系统盘就在进安装盘进行二次安装（不需要再抹盘直接安装，一般进去后就会自动安装第二次）原版基本都要二次安装。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;8）DSDT、SSDT 等 安装中不需要放置，等安装成功能用 U 盘的 Clover 进 Mac 系统后，将 Clover 安装到主机硬盘，再添加 DSDT、SSDT 等进行微调。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;9）config.plist 一些基本原则：<br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(1) 如果你不知道这个参数是干什么的，或者他的值应该是多少，那么直接从 config 里删除掉这个参数。<br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(2) 不要设定你不知道的参数以及参数所对应的值。<br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(3) 任何参数都需要一个值，宁可删掉这个参数，也不要留空不填（不填写这个参数的值）。<br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(4) 在 Clover 引导界面，进入 Option 设置，可以方便地临时修改各个参数的设定，请善用此功能。<br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(5) 新手在引导系统安装时，config.plist 的参数尽可能从简。<br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(6) 很多 -v 五国可以通过 Clover 的 DSDT Fixes 来解决，但是写多了 Fixes 可能会导致 AppleSMCD 等错误。<br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(7) 因显卡驱动等原因而卡死在最后，你需要认真设置 Config 的显卡部分。<br/>

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;


## 3、开始安装
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;高频率内存方面，笔者只了解到 芝奇、海盗船。幻光戟 与 复仇者 加 蓝屏 关键字，都有较多搜索结果，因此这里笔者选了 铂金统治者，类似结果没搜到，而且据说是特挑的颗粒。
















