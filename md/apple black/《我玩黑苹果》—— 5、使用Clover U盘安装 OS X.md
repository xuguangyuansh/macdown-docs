# 我玩黑苹果 — 5、使用Clover U盘安装 OS X

## 1、目标机准备
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;准备一块空的磁盘：<br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;1）在Win 或 pe 下用 DiskGenius 转为 GUID 分区表，由于 10.11 后的 OS X，安装时的 磁盘工具 已不再支持该转换，因此需在 win 下完成；<br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;2）删除所有分区；

## 2、开始安装
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;配置好后，重启，优盘快捷启动，记得一定选的是 UEFI 优盘的。<br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;这里放一张图，与新版 Clover 可能对应不上，但可用于说明策略：

<center>![v-f-x][]</center>

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;根据以往小伙伴的经验，一般选第四个（-v模式）或者倒数第二个（不加载注入的驱动）。

> 切记，安装等待它自动重启，最后1秒会很久，安装重启还是优盘引导，到引导界面没有看到 MAC 系统盘就在进安装盘进行二次安装（不需要再摸盘直接在安装，一般进去后就会自动安装第二次）原版基本都要二次安装。











<div style="display:none;">
[v-f-x]:

</div>
