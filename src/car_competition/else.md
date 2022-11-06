# 其他篇

一些没长到需要单独写一页的地方

## 串口

所有的单片机都有串口，串口是（我觉得）最好用的与单片机交互的方式。

在智能车的比赛中，逐飞等竞赛相关公司提供了单片机的代码框架，我们可以很好的使用`printf`进行窗口输出。如果不想使用现成的超级臃肿的代码框架，我们也可以使用几行代码自己写一个。

```c
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

#### VOFA的JustFloat协议

```c
void UartFloat(float *pData, uint32_t number) {
	const uint8_t tail[4] = {0x00, 0x00, 0x80, 0x7f};
	memcpy(Uart_TransmitBuffer,pData,sizeof(float) * number);
	memcpy(&Uart_TransmitBuffer[sizeof(float) * number],tail,4);

	exDMA_SetMemoryAddress(DMA1_Channel2, Uart_TransmitBuffer);
	exDMA_SetTransmitLen(DMA1_Channel2, sizeof(float) * number + 4);
	DMA_Cmd(DMA1_Channel2, ENABLE);
}
```

## 代码编辑器

众所周知，Keil的调试功能和编译功能是比较好用的，而编辑器及其难用。我们使用Clion作为代码编辑器，他强大的代码补全和提示功能可以极大的加快写代码的速度。

MM32与STM32相似，Clion的配置可以参考稚晖君的[配置CLion用于STM32开发【优雅の嵌入式开发】]([配置CLion用于STM32开发【优雅の嵌入式开发】 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/145801160))

#### 代码版本管理

如果你不是一个人写所有代码的话，该注意一下代码版本管理了。你也不想桌面上全部是不同版本的代码吧：）

去了解一下git



## 调参界面



## 轮胎

