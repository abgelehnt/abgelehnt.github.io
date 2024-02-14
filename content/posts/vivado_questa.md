+++

title = 'Vivado联合QuestaSim仿真环境搭建'
date = 2024-02-13T22:16:12+08:00
draft = true
summary = '使用Vivado编写FPGA设计代码后，通常需要对代码进行验证。本文将阐述Vivado工程的QuestaSim仿真验证环境搭建方法。'

+++

使用Vivado编写FPGA设计代码后，通常需要对代码进行验证。本文将阐述Vivado工程的QuestaSim仿真验证环境搭建方法。

# 编译Xilinx IP

打开工程环境

# 编译DUT代码

``` 
TODO:目录
```

# 编辑Makefile

``` makefile
.PHONY : all clean
TODO
```

## QuestaSim常用命令参数详解

``` bash
vlib lib/tb_lib
vmap 
vlog -work tb_lib \
	+incdir+tb_system \
	tb.sv
```

# 搭建简单的UVM验证环境

``` systemverilog
initial begin
    
end
```

# 添加TC

``` systemverilog
TC
```

# 添加自动化测试支持

``` makefile
```

# 获取覆盖率报告

``` makefile
```



