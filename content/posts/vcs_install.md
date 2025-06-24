+++

title = 'VCS&Verdi安装教程'
date = 2025-06-22
draft = true
summary = '本文阐述VCS&Verdi安装教程与安装最佳实践。'

+++

# 安装目录与安装用户

推荐使用专用用户<tools>，然后将所有工具安装到用户的家目录中

# VCS安装教程

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
export SNPSLMD_LICENSE_FILE=27000@hg
export LM_LICENSE_FILE="$SYNOPSYS/license/Synopsys.dat"

#SCL
alias lmli="$SYNOPSYS/scl/2018.06/linux64/bin/lmgrd -c $SYNOPSYS/license/Synopsys.dat"
export PATH="$SYNOPSYS/scl/2018.06/linux64/bin:"$PATH

# alias dve="dve -full64 &"
# alias vcs64="vcs -full64"
# alias vcs="vcs -full64"
```

# lmli进程自启动

```bash
$ mkdir -p .config/systemd/user/
$ vi .config/systemd/user/lmgrd.service
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

```bash
# 使用systemctl --user命令需要避免使用su命令更改用户
sudo loginctl enable-linger tools # 选项允许你在用户会话结束后，仍然保持用户的systemd 实例，使得用户定义的单元可以在用户退出后继续运行。
$ systemctl --user start lmgrd.service 
$ systemctl --user enable lmgrd.service 
$ systemctl --user status lmgrd.service
$ systemctl --user stop lmgrd.service
```

如果lmgrd.service执行时出现Permission Denied: locksnpslmd文件出现
```bash
$ sudo rm /var/tmp/lock*
```

(lmgrd) Failed to open the TCP port number in the license.
关闭
```bash
lmdown -q
```
然后等待1分钟