+++

title = 'VNC Server 自启动配置指南'
date = 2025-06-24
tags = ["软件安装教程"]
summary = '本文介绍了如何配置 VNC Server 作为 systemd 用户服务实现开机自启动，包括 GNOME Flashback 桌面环境的 xstartup 配置、systemd 服务单元文件的创建，以及常用的服务管理命令。适用于需要远程桌面访问的 Linux 用户，确保 VNC 服务在系统启动后自动运行。'

+++

## 1. 配置 xstartup 文件

编辑 `~/.vnc/xstartup` 文件，配置 GNOME Flashback 桌面环境：

```bash
unset SESSION_MANAGER
unset DBUS_SESSION_BUS_ADDRESS
export XKL_XMODMAP_DISABLE=1
export XDG_CURRENT_DESKTOP="GNOME-Flashback:GNOME"
export XDG_MENU_PREFIX="gnome-flashback-"
[ -x /etc/vnc/xstartup ] && exec /etc/vnc/xstartup
[ -r $HOME/.Xresources ] && xrdb $HOME/.Xresources
xsetroot -solid grey
vncconfig -iconic &
gnome-session --session=gnome-flashback-metacity --disable-acceleration-check &
```

## 2. 创建 systemd 用户服务

编辑 `~/.config/systemd/user/vncserver.service` 文件，添加 systemd 服务单元文件：

```bash
[Unit]
Description=vncserver startup
Wants=network-online.target
After=network-online.target systemd-user-sessions.service

[Service]
Type=forking
ExecStart=vncserver -geometry 2560x1440
ExecStop=vncserver -kill :1

[Install]
WantedBy=default.target
```

## 3. 服务管理命令

常用 systemd 用户服务管理命令：

```bash
# 该命令允许你在用户会话结束后，仍然保持用户的systemd实例，使得用户定义的单元可以在用户退出后继续运行。
sudo loginctl enable-linger <user_name> 

# 启动服务
systemctl --user start vncserver.service

# 设置开机自启
systemctl --user enable vncserver.service

# 查看服务状态
systemctl --user status vncserver.service

# 停止服务
systemctl --user stop vncserver.service

# 关闭开机自启
systemctl --user disable vncserver.service
```

## 注意事项

1. 确保已安装 `tigervnc-server` 和 `gnome-flashback` 软件包
2. 首次运行 vncserver 需要设置访问密码
3. 分辨率参数 `2560x1440` 可根据显示器配置调整
