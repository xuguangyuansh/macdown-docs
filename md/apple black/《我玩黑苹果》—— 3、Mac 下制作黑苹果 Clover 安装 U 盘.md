# 我玩黑苹果 — 3、Mac下制作黑苹果 Clover 安装 U盘
## 1、原版镜像下载
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;有两种方式：1）来到苹果官网 https://www.apple.com/cn/，依次点击 技术支持 -> Mac -> 前往 macOS 支持 -> 下载 macOS High Sierra，最终跳转回 App Store 进行下载；2）打开 App Store，在页面右侧点击 “macOS High Sierra” 进行下载，下载完成后镜像会存在于 应用程序 列表中。<br/>

<center>![osx-download][]</center>

<center>![osx-download-over][]</center>

## 2、下载Clover
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;这个没什么难度，到 [https://www.sourceforge.net/projects/cloverefiboot/](https://www.sourceforge.net/projects/cloverefiboot/) 下载即可。<br/>

<center>![clover-download][]</center>

## 3、格式化U盘
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;使用磁盘工具，将 U盘分成一个区，名称：OSX；格式：Mac OS 扩展（日志式）；并将 U盘转 GUID分区（笔者的 U盘在 win 时已是 GPT，就跳过了这步，也不知道怎么搞）。

<center>![format-u][]</center>

## 4、写入镜像到U盘
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;使用命令

```
sudo /Applications/Install\ macOS\ High\ Sierra.app/Contents/Resources/createinstallmedia --volume /Volumes/OSX --applicationpath /Applications/Install\ macOS\ High\ Sierra.app --nointeraction
```

解释一下：<br/>

- 第一个 `/Applications/Install\ macOS\ High\ Sierra.app/`：是 OS X 的镜像位置，此处用于指明路径，最终我们要使用内部的 createinstallmedia 工具，该工具也可以提前拷贝到其他路径使用。
- `--volume`：指定拷贝到哪个地址。
- `/Volumes/OSX`：计划拷贝的目标目录，我们的 U盘 插入 Mac 后会默认挂载到 `/Volumes/` 目录，我们刚刚又在磁盘工具中将其改名为 OSX，所以这里也自动变成了 OSX 目录。
- `--applicationpath`：指定待拷贝程序所在目录。
- 第二个`/Applications/Install\ macOS\ High\ Sierra.app`：待拷贝的程序，也就是我们的 OSX 镜像。
- `--nointeraction`：所有提问使用默认选项，进尔拷贝过程将不会再有提示。

## 5、安装Clover引导
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;在终端输入 `mkdir /Volumes/EFI`，双击我们下载的 `Clover_v2.4k_r4497.pkg`，如果下载的是 .zip 文件，需要先手动解压，之后按下面方式操作：

<center>![clover-change-button][]</center>

<center>![clover-change-disk][]</center>

<center>![clover-custom-button][]</center>

<center>![clover-custom-1][]</center>

<center>![clover-custom-2][]</center>

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;最后，点击 安装。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;解释：由于 Mac 下默认不显示
 EFI 分区，而 Clover 在安装时会创建 EFI 分区，因此我们先建一个 EFI 目录，这样 Clover 在安装时会自动挂载 EFI 分区到 EFI 目录，方便我们后续向 EFI 中拷贝 config.plist、DSDT.aml、SSDT.aml 和 kext 等文件，比较方便。
 
## 附录 1
>1、`DSDT.aml`和`SSDT.aml`放在`/EFI/CLOVER/ACPI/patched`文件夹下；
>2、`kext`放在`/EFI/CLOVER/kexts/Other`文件夹下或者系统对应版本的文件夹下，一般只放`FakeSMC`、`AppleACPIPS2Nub`和`ApplePS2Controller`就够了。

## 附录 2
> 之后要想修改的话可以用以下命令挂载EFI分区：<br/>

```
diskutil list (用来查看分区列表)
mkdir /Volumes/EFI (新建EFI挂载点)
sudo mount -t msdos /dev/disk3s1 /Volumes/EFI/ (挂载EFI分区，disk3s1 根据EFI分区来写)
```

> 下边还有几个备用命令：

```
sudo mount_hfs /dev/disk3s1 /volumes/EFI (挂载hfs分区)
umount /Volumes/EFI  (推出分区)
umount –f /volumes/EFI  (强制推出分区)
newfs_msdos -v Fat32 -F 32 /dev/rdisk3s1  (格式化EFI分区为fat32)
su (获取root权限)
```

## 6、完成
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;到此安装盘就制作完成了。补充两点：<br/>
   第一，语言设置成 `zh_CN:0` 安装界面会显示为中文(默认的 `en:0` 为英文)；
   第二，开机 U盘启动之后选择 U盘上的安装分区，按空格，选择 Boot Mac OS X without caches an with extra kexts 启动。


[osx-download]:
[osx-download-over]:
[clover-download]:
[format-u]:
[clover-change-button]:
[clover-change-disk]:
[clover-custom-button]:
[clover-custom-1]:
[clover-custom-2]:















