# React Native — x1、分离expo项目
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;简单说，分离后就可以进入 rn 原生混合开发模式。也由此引出了 expo kit，这里以官方的口吻介绍下 expo kit。<br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;expo kit 是一个允许你用 expo 平台把已经创建的 expo 项目作为更大的标准 native 项目一部分的 objective-c 和 java 类库。当把 expo 项目分离后，就可以用 xcode 或者 AndroidStudio 或者 react-native init 打开项目。<br/>

## 1、为何要detache呢
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;当我们用 expo 创建项目后，如果要调用移动端平台的原生模块，这在 expo 里是做不到的，因为 expo 是纯 js 写的，调用不到 ios 或 Android 层。然而某些情况下，一些高级开发者，需要用到 native 的功能，这就超出了 expo 提供的范围，通过把 expo 项目 detache，分离出来的项目可以用 xcode 打开，也可以用 AndroidStudio 运行，并且分离出来的项目对 expo kit 仍有依赖关系，也就是仍然依赖 expo sdk，这样 expo 提供的 api 就可以继续使用了。<br/>

## 2、开始分离
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;1）使用 Xde 创建项目。<br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;2）进入项目目录，执行：<br/>

```
exp detach
```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<br/>

<center>![][]</center>

<div style="display:none;">
[]:
[]:
[]:
</div>













