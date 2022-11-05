# 基本篇

一些比较基础但是重要的地方

## 串口

所有的单片机都有串口，串口是（我觉得）最好用的与单片机交互的方式。

在智能车的比赛中，逐飞等竞赛相关公司提供了单片机的代码框架，我们可以很好的使用`printf`进行窗口输出。如果不想使用现成的超级臃肿的代码框架，我们也可以使用几行代码自己写一个。

```C
void UartPrintf(const char *fmt, ...) {
	va_list ap;
	char dataBuf[512];
	va_start(ap, fmt);
	vsprintf((char *) dataBuf, fmt, ap);
	va_end(ap);
    while(!UART_GetFlagStatus(UART1,  UART_FLAG_TXEMPTY))
        ;
    UART_PUTBUFF(UART_1, (uint8_t *) dataBuf, strlen(dataBuf));
}
```

在调试的过程中，如果单片机的调试功能不好用的话，勤用串口打log，这会比呆呆的看代码有用的多。

## 调参界面



## 代码版本管理

如果你不是一个人写所有代码的话，该注意一下代码版本管理了。你也不想桌面上全部是不同版本的代码吧：）

去了解一下git

