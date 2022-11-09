# 其他篇

一些没长到需要单独写一页的地方。

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

MM32与STM32相似，Clion的配置可以参考稚晖君的[配置CLion用于STM32开发【优雅の嵌入式开发】](https://zhuanlan.zhihu.com/p/145801160))

#### 代码版本管理

如果你不是一个人写所有代码的话，该注意一下代码版本管理了。你也不想桌面上全部是不同版本的代码吧：）

去了解一下git

## 调参界面

速度，二值化阈值等参数是需要在比赛时随时调整的。如果使用英飞凌单片机的话使用电脑编译下载一次需要至少1分钟，使用MM32也需要30秒。这个时长是无法接受的。如果能在小车上使用按键修改参数的话，那会极大的节省时间。

## 轮胎

讲一讲我们处理轮胎的方式和效果

- 轮胎面软化：有用。但是需要大量的轮胎软化剂，氪金玩家可以多买点然后把轮胎泡软化剂里。

- 磨轮胎面：效果一般。不过超级难磨，如果学长没有传下来的话建议咸鱼买一对。
- 里面多填一份海绵：有用。更重要的是保持海绵平整，龙邱有环状的海绵可以购买尝试。看到往年有队伍在轮胎里多填充两份海绵，我们尝试过后发现多填充两份会导致海绵无法平整，反而得不偿失。
- 对轮胎进行封边：有用。主要是为了防止轮胎甩出来。
- 保持轮胎干净：极有用。在轮胎面软化过后，只要有一点点的灰就会疯狂打滑。比赛时需要时刻保持轮胎面清洁。我们比赛时使用WD-40并在每次发车前涂抹，清洁效果极好。
