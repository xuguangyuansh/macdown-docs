# React Native — 3、RN混合开发之 XDE
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;前面文章讲了用 react-native-cli 创建 RN 混合开发项目，但有一个问题是编辑项目后，模拟器不能自动加载，需要 Cmd+R 来手动触发，因此，这里来介绍另外一款工具，expo 的 xde。
## 安装 xde
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;浏览器打开地址 [https://expo.io/tools](https://expo.io/tools)（或鼠标点击搜索框，会向下展开一个区域，其中也有 XDE），将页面托到下半部分，可以找到 Expo XDE，目前支持 MacOS，Windows，Linux 三个平台。

<center>![expo-xde][]</center>

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;点击其中的 `Download from GitHub` 连接，在新页面选择 .dmg 文件进行下载

<center>![xde-download][]</center>

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;之后，安装方法就是 双击此 xde-***.dmg 文件，再拖动 Expo XDE.app 到 Applications 中即可。

<center>![xde-install][]</center>

## 注册 XDE 账号

<center>![xde-register][]</center>

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;按提示信息注册 Expo 账号，成功后使用 Expo 账号登录 XDE。

## 使用 XDE 创建 RN 混合开发项目
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;登录 xde 后，点击 “create new project...”，填写项目名称，修改路径等，最后点击 create。

<center>![xde-create-new][]</center>

<center>![xde-create][]</center>

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;稍等片刻项目就创建好了，出现以下信息说明项目已经启动，点击 Device 并选择 ios，可以启动 iphone x 模拟器，

<center>![xde-run-ios][]</center>

<center>![xde-iphone][]</center>

## 编辑项目
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;用 WebStorm 打开刚刚创建的 rn 混合项目 myxde，并在其中的 App.js 中编写 react 代码，这次 iphone 模拟器中又可以自动更新渲染了。

## 移动 Iphone 窗口
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;与 crna 时一样，Iphone x 窗口移动时经常失败，可点住尽量靠近边缘位置，能提升一些成功率。

## Iphone 的物理键
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;笔者 OS X 版本 10.13.4，所在时间 2018-05-25，此 iphone x 模拟器周围的 电源，音量，静音 按键匀可以鼠标点击。


[expo-xde]:
[xde-download]:
[xde-install]:
[xde-register]:
[xde-create-new]:
[xde-create]:
[xde-run-ios]:
[xde-iphone]:











