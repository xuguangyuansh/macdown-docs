# 容器与容器云 — x、Mac 下 Docker 安装

## 1、使用 Docker.dmg 安装
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;下载并运行 docker.dmg：
[https://download.docker.com/mac/stable/Docker.dmg](https://download.docker.com/mac/stable/Docker.dmg)。<br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;这种安装方式虽然简单，且可通过迅雷进行下载以快速部署，但 Mac 主机却无法 ping 通容器实例，因为 docker 原生只适用于 Linux，其余所有系统，匀是先在主机上虚拟一台 linux，再在该虚拟机中安装 docker。而使用 docker.dmg 部署时由于其不会为 Mac 主机创建虚拟网卡，因此 Mac 到这台虚拟的 linux 系统的网络是不通的，进而无法通过网络访问容器的服务，但仍然可以通过端口映射来访问，不过有些服务必须要和客户端处于不同网络，这种情况下就不得不使用 Homebrew + virtualbox 的方式来部署 docker。

## 2、使用 Homebrew 安装
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;1）安装 Homebrew

```
ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;2）安装 VirtualBox
[https://download.virtualbox.org/virtualbox/5.2.14/VirtualBox-5.2.14-123301-OSX.dmg
](https://download.virtualbox.org/virtualbox/5.2.14/VirtualBox-5.2.14-123301-OSX.dmg
)。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;3）安装 docker、docker-machine

```
brew install docker boot2docker docker-machine docker-compose
```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4）创建默认 linux

```
docker-machine create --driver virtualbox default
```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;5）进入 linux docker 环境

```
docker-machine ssh default
```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;之后就可以进入 docker 操作了，如 docker pull ...，docker ps -a 等，而且该 linux 环境还可以通过 virtualbox 虚拟网卡与主机通信。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;6）若要直接从宿主机发起 docker 命令，需要编辑 ~/.zshrc 或 ~/.bashrc 文件，在末尾添加 `eval "$(docker-machine env default)"`

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;7）开机启动虚拟容器主机：

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;在用户目录 ~/ 创建 `com.docker.machine.default.plist` 文件。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
    <dict>
        <key>EnvironmentVariables</key>
        <dict>
            <key>PATH</key>
            <string>/usr/bin:/bin:/usr/sbin:/sbin:/usr/local/bin</string>
        </dict>
        <key>Label</key>
        <string>com.docker.machine.default</string>
        <key>ProgramArguments</key>
        <array>
            <string>/usr/local/bin/docker-machine</string>
            <string>start</string>
            <string>default</string>
        </array>
        <key>RunAtLoad</key>
        <true/>
    </dict>
</plist>
```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;之后将其拷贝到`/Library/LaunchAgents`目录，并附 755 权限，并使用`launchctl load`进行加载。

```
cp com.docker.machine.default.plist /Library/LaunchAgents/
cd /Library/LaunchAgents/
sudo chmod 755 com.docker.machine.default.plist
launchctl load com.docker.machine.default.plist
```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;每次修改 plist 文件，匀需重新 `launchctl unload && launchctl load`。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;load 以后，可使用`launchctl list | grep docker`来查看是否成功。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;至此，docker-machine default 已可以随 Mac 系统登录而启动。

注：boot2docker 虚拟主机的默认用户/密码：docker/tcuser

参考资料：<br/>
https://gist.github.com/andystanton/257fab335b242bc2658b
https://stackoverflow.com/questions/39226654/start-docker-machine-on-boot
https://andy.stanton.is/writing/about/running-a-local-docker-machine-at-startup-on-osx/

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;此外，还要以安装 kitematic，这是 docker 推出的 GUI 工具，使操作 docker 的方式变得更简单直观。

## 3、Mac ping 通 docker 容器
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;通过 virtualbox 安装的容器主机，是可以通过 virtualbox 的虚拟网卡和 Mac 主机通信的，但容器主机内的容器实例仍是无法与 Mac 主机通信的。处理方法如下：假设 virtualbox 容器主机的 ip 为 192.168.99.100，容器主机内的容器实例 ip 段为 172.17.0.0/16，则需要为系统添加一条路由，即告诉系统所有到 172.17.0.0/16 的数据，都从 192.168.99.100 这个网卡发出去，mac 下 `sudo route -n add -net 172.17.0.0/16 192.168.99.100`，这样 Mac 主机就可以 ping 通位于虚拟容器主机内处于 172.17.0.0/16 网段的容器实例了。

参考资料：https://blog.csdn.net/Mr0o0rM/article/details/80683115


## 4、docker-machine 常用命令
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;查看当前的machine：```docker-machine ls```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;创建一个docker-machine：<br/>
```docker-machine create --driver virtualbox default```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;更改环境变量，使得本地 docker 指向docker-machine，需要执行命令：<br/>
```eval "$(docker-machine env default)"```

- `help` 查看帮助信息
- `active` 查看活动的Docker主机
- `config` 输出连接的配置信息
- `create` 创建一个Docker主机
- `env` 显示连接到某个主机需要的环境变量
- `inspect` 输出主机更新信息
- `ip` 获取Docker主机地址
- `kill` 停止某个Docker主机
- `ls` 列出所有管理的Docker主机
- `regenerate-certs` 为某个主机重新成功TLS认证信息
- `restart` 重启Docker主机
- `rm` 删除Docker主机
- `scp` 在Docker主机之间复制文件
- `ssh` SSH到主机上执行命令
- `start` 启动一个主机
- `status` 查看一个主机状态
- `stop` 停止一个主机
- `upgrade` 更新主机Docker版本为最新
- `url` 获取主机的URL

参考资料：https://www.jianshu.com/p/5d3f6b40b132







