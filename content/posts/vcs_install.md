+++

title = 'VCS与Verdi安装指南'
date = 2025-06-22
summary = '详细说明VCS和Verdi工具的安装配置流程及常见问题解决方法'

+++

# 安装环境准备

推荐方案：创建专用用户`tools`，所有EDA工具安装在该用户家目录下，避免权限冲突。

# VCS环境配置

以下为VCS和Verdi所需的环境变量设置：

```bash
export SYNOPSYS="/home/tools/synopsys"
export VCS_TARGET_ARCH="amd64"
export DVE_HOME="$SYNOPSYS/vcs/O-2018.09-SP2/gui/dve"
export PATH="$SYNOPSYS/vcs-mx/O-2018.09-SP2/bin:"$PATH
export VCS_HOME="$SYNOPSYS/vcs-mx/O-2018.09-SP2"

export VCS_ARCH_OVERRIDE="linux"
#verdi
export PATH="$SYNOPSYS/verdi/Verdi_O-2018.09-SP2/bin:"$PATH
export VERDI_HOME="$SYNOPSYS/verdi/Verdi_O-2018.09-SP2"
export LD_LIBRARY_PATH="$SYNOPSYS/verdi/Verdi_O-2018.09-SP2/share/PLI/lib/LINUX64":$LD_LIBRARY_PATH
export VERDI_DIR="$SYNOPSYS/verdi/Verdi_O-2018.09-SP2"
export NOVAS_INST_DIR="$SYNOPSYS/verdi/Verdi_O-2018.09-SP2"
export NPI_PLATFORM="LINUX64_GNU_472"
export LD_LIBRARY_PATH="$NOVAS_INST_DIR/share/NPI/lib/LINUX64_GNU_520":$LD_LIBRARY_PATH
export NOVAS_HOME="$SYNOPSYS/verdi/Verdi_O-2018.09-SP2"

#LICENSE
export SNPSLMD_LICENSE_FILE=27000@`hostname`
export LM_LICENSE_FILE="$SYNOPSYS/license/Synopsys.dat"

#SCL
alias lmli="$SYNOPSYS/scl/2018.06/linux64/bin/lmgrd -c $SYNOPSYS/license/Synopsys.dat"
export PATH="$SYNOPSYS/scl/2018.06/linux64/bin:"$PATH

# alias dve="dve -full64 &"
# alias vcs64="vcs -full64"
# alias vcs="vcs -full64"
```

# License文件生成

在Windows系统中使用scl_keygen.exe生成License文件时，需更改以下参数：

| 参数项 | 说明 |
|--------|------|
| HOST ID Daemon | 主机MAC地址（通过`lmhostid`获取） |
| HOST ID Feature | 同上 |
| HOST Name | 主机名（通过`hostname`获取） |

注意：

- 多网卡设备执行`lmhostid`会出现多个MAC地址，仅需填写一个MAC地址
- 避免使用USB网卡等可插拔设备的MAC地址

# License服务自启动

创建systemd服务单元文件：

```bash
$ mkdir -p ~/.config/systemd/user/
$ vi ~/.config/systemd/user/lmgrd.service
[Unit]
Description=FlexLM license server daemon(VCS)
Wants=network-online.target
After=network-online.target systemd-user-sessions.service

[Service]
Type=forking
WorkingDirectory=/home/tools/synopsys
ExecStart=/home/tools/synopsys/scl/2018.06/linux64/bin/lmgrd -c /home/tools/synopsys/license/Synopsys.dat
ExecStop=/home/tools/synopsys/scl/2018.06/linux64/bin/lmdown -q

[Install]
WantedBy=default.target
```

服务管理命令：

```bash
# 启用用户级服务
sudo loginctl enable-linger tools # 选项允许你在用户会话结束后，仍然保持用户的systemd 实例，使得用户定义的单元可以在用户退出后继续运行。
# 使用systemctl --user命令需要避免使用su命令更改用户
systemctl --user start lmgrd.service
systemctl --user enable lmgrd.service

# 查看服务状态
systemctl --user status lmgrd.service
# 实时查看lmgrd.service输出的日志（Ctrl+C退出）
journalctl --user -u -f lmgrd.service

# 停止服务
systemctl --user stop lmgrd.service
```


# 常见问题解决

1. Permission Denied: locksnpslmd文件出现

```bash
$ sudo rm /var/tmp/lock*
```

2. (lmgrd) Failed to open the TCP port number in the license.

停止lmgrd.service，然后等待1分钟
