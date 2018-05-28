# React Native — 1、CRNA
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;由于笔者的个人喜好，Java 只喜欢用来做服务端，而客户端并不喜欢用 java，此因这里通篇都只讲的 Rn for ios，若想了解 Rn for andriod 的，可以整体跳过该系列文章了。<br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Rn for ios，首先要安装 xcode，在 app store 能够搜到。时间较长，建议在晚上下班临走前点上。<br/>

## 安装 yarn
```
sudo npm install -g yarn
yarn config set registry https://registry.npm.taobao.org --global
yarn config set disturl https://npm.taobao.org/dist --global
```

## 安装 create-react-native-app
```
sudo yarn global add create-react-native-app
```
## 创建crna项目
```
create-react-native-app myapp
```

## 启动crna项目
```
cd myapp && yarn start
```
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;当显示 Starting packager... 后，键盘按 q，会生成一个二维码，和一些信息。根据提示信息 键盘按 i，之后会自动打开 xcode ios 模拟器，且是模拟的最新 iphone 产品 iphone x。

## 编辑项目
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;用 WebStorm 打开刚刚创建的 cran 项目 myapp，并在其中的 App.js 中编写 react 代码，ios 模拟器中会自动更新渲染。

## 移动 Iphone 窗口
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;在笔者的开发中，如果打开的是 iphone x 模拟器，其窗口会很难被移动，此时可尝试 点住 尽可能靠近外边界的地方来拖动，成功率会大增加。

## Iphone 的物理键
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;笔者 OS X 版本 10.13.4，所在时间为 2018-05-25，发现该 iphone 模拟器做的是比较精细的，周围的 电源，音量，静音 按键匀是可以鼠标点击的。


















