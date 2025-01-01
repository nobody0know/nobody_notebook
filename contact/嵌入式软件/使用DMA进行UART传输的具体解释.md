# 通过寄存器配置DMA进行USART传输的具体解析

## 前情提要

HAL库使用cubemx初始化USART并开启DMA传输，设置好是否使用缓冲区，DMA工作模式，以及数据流向模式（一般RX是设置外设到储存，TX是储存到外设）

然后自己可以写一个函数作为使能串口DMA的init函数或者直接丢到main的while(1)前

## DMA部分寄存器解析以及串口相关寄存器解析

### 配置时用到的USART寄存器中的位

通过设置控制寄存器 USART_CR3

第六位和第七位的DMAR和DMAT使能发送和接收器

> <mark>状态寄存器 (USART_SR)</mark>
> Status register
> 偏移地址：0x00
> 复位值：0x00C0 0000
> 
> <mark>位 5 RXNE：读取数据寄存器不为空 (Read data register not empty)</mark>
> 当 RDR 移位寄存器的内容已传输到 USART_DR 寄存器时，该位由硬件置 1。如果 USART_CR1 寄存器中 RXNEIE = 1，则会生成中断。通过对 USART_DR 寄存器执行读入 操作将该位清零。RXNE 标志也可以通过向该位写入零来清零。建议仅在多缓冲区通信时使 
> 用此清零序列。
> 0：未接收到数据
> 1：已准备好读取接收到的数据
> <mark>位 4 IDLE：检测到空闲线路 (IDLE line detected)</mark>
> 检测到空闲线路时，该位由硬件置 1。如果 USART_CR1 寄存器中 IDLEIE = 1，则会生成中 断。该位由软件序列清零（读入 USART_SR 寄存器，然后读入 USART_DR 寄存器）。
> 0：未检测到空闲线路
> 1：检测到空闲线路
> 注意：直到 RXNE 位本身已置 1 时（即，当出现新的空闲线路时）IDLE 位才会被再次置 1。
> 
> <mark>位 0 PE：奇偶校验错误 (Parity error)</mark>
> 当在接收器模式下发生奇偶校验错误时，该位由硬件置 1。
> 
> <mark>该位由软件序列清零</mark>
> 
> <mark>（读取状态寄存器，然后对 USART_DR 数据寄存器执行读或写访问）。</mark>
> 
> 将 PE 位清零前软件必须等待 RXNE 标志被置 1。 
> 如果 USART_CR1 寄存器中 PEIE = 1，则会生成中断。
> 0：无奇偶校验错误 
> 1：奇偶校验错误
> 
> 
> 
> <mark>控制寄存器 3 (USART_CR3)</mark>
> Control register 3
> 偏移地址：0x14
> 复位值：0x0000 0000
> 
>  <mark>位7 DMAT： DMA 使能发送器 (DMA enable transmitter)</mark>
> 该位由软件置 1/ 复位。
> 1：针对发送使能 DMA 模式。
> 0：针对发送禁止 DMA 模式。
> <mark>位6 DMAR： DMA 使能接收器 (DMA enable receiver)</mark>
> 该位由软件置 1/ 复位。
> 1：针对接收使能 DMA 模式 
> 0：针对接收禁止 DMA 模式

```c
    //使能DMA串口接收和发送
    SET_BIT(huart1.Instance->CR3, USART_CR3_DMAR);
    SET_BIT(huart1.Instance->CR3, USART_CR3_DMAT);
```

> <mark>数据寄存器 (USART_DR)</mark>
> Data register
> 偏移地址：0x04
> 复位值：0xXXXX XXXX 
> 
> 位 31:9 保留，必须保持复位值
> <mark>位 8:0 DR[8:0]：数据值</mark>
> 包含接收到数据字符或已发送的数据字符，具体取决于所执行的操作是“读取”操作还是“写入”操作。
> 因为数据寄存器包含两个寄存器，一个用于发送 (TDR)，一个用于接收 (RDR)，因此它具有 双重功能（读和写）。
> TDR 寄存器在内部总线和输出移位寄存器之间提供了并行接口。
> RDR 寄存器在输入移位寄存器和内部总线之间提供了并行接口。
> 在使能奇偶校验位的情况下（USART_CR1 寄存器中的 PCE 位被置 1）进行发送时，由于 MSB 的写入值（位 7 或位 8，具体取决于数据长度）会被奇偶校验位所取代，因此该值不 起任何作用。
> 在使能奇偶校验位的情况下进行接收时，从 MSB 位中读取的值为接收到的奇偶校验位。

串口外设每一次的收发原数据都收在DR寄存器中

### 配置时用到的DMA寄存器中的位

> <mark>DMA 数据流 x 配置寄存器 (DMA_SxCR)</mark> (x = 0..7)
> DMA stream x configuration register
> 此寄存器用于配置相关数据流。
> 偏移地址：0x10 + 0x18 × 数据流编号
> 复位值：0x0000 0000
> 
> <mark>位 0 EN：数据流使能/读作低电平时数据流就绪标志</mark> (Stream enable / flag stream ready when read low)
> 此位由软件置 1 和清零。
> 0：禁止数据流 
> 1：使能数据流
> 以下情况下，此位可由硬件清零：
> — DMA 传输结束时（准备好配置数据流）
> — AHB 主总线出现传输错误时
> — 存储器 AHB 端口上的 FIFO 阈值与突发大小不兼容时
> 此位读作 0 时，软件可以对配置和 FIFO 位寄存器编程。EN 位读作 1 时，禁止向这些寄存 器执行写操作。
> 注意：将 EN 位置“1”以启动新传输之前，DMA_LISR 或 DMA_HISR 寄存器中与数据流向对应的事件标志必须清零。

> <mark>DMA 数据流 x 外设地址寄存器 (DMA_SxPAR)</mark> (x = 0..7)
> DMA stream x peripheral address register
> 偏移地址：0x18 + 0x18 × 数据流编号
> 复位值：0x0000 0000
> 
> <mark>位 31:0 PAR[31:0]：外设地址 (Peripheral address)</mark>
> 读/写数据的外设数据寄存器的基址。
> 这些位受到写保护，只有 DMA_SxCR 寄存器中的 EN 为“0”时才可以写入。

> <mark>DMA 数据流 x 存储器 0 地址寄存器 (DMA_SxM0AR)</mark> (x = 0..7)
> DMA stream x memory 0 address register
> 偏移地址：0x1C + 0x18 × 数据流编号
> 复位值：0x0000 0000
> 
> <mark>位 31:0 M0A[31:0]：存储器 0 地址 (Memory 0 address)</mark>
> 读/写数据的存储区 0 的基址。
> 这些位受到写保护，只有在以下情况下才可以写入：
> — 禁止数据流（DMA_SxCR 寄存器中的位 EN=“0”）或
> — 使能数据流（DMA_SxCR 寄存器中的 EN=“1”）并且 DMA_SxCR 寄存器中的 
>       位 CT =“1”（在双缓冲区模式下）。

> <mark>DMA 数据流 x 存储器 1 地址寄存器 (DMA_SxM1AR)</mark> (x = 0..7)
> DMA stream x memory 1 address register
> 偏移地址：0x20 + 0x18 × 数据流编号
> 复位值：0x0000 0000
> 
> <mark>位 31:0 M1A[31:0]：存储器 1 地址（用于双缓冲区模式）</mark>(Memory 1 address (used in case of Double buffer mode))
> 读/写数据的存储区 1 的基址。
> 此寄存器仅用于双缓冲区模式。
> 这些位受到写保护，只有在以下情况下才可以写入：
> — 禁止数据流（DMA_SxCR 寄存器中的位 EN=“0”）或
> — 使能数据流（DMA_SxCR 寄存器中的 EN=“1”）并且 DMA_SxCR 寄存器中的位      CT =“0”。

## 操作流程

### USART DMA RX init配置

```c
SET_BIT(huart1.Instance->CR3, USART_CR3_DMAR);
__HAL_UART_ENABLE_IT(&huart1, UART_IT_IDLE);//空闲中断

__HAL_DMA_DISABLE(&hdma_usart1_rx);

//第一次调用时硬件会将CR的位0 EN位清零
//意思为DMA准备好配置数据流0&1=0
//此次用于为前面那个disable的成功与否做处理
//若成功则跳出循环，失败则继续disable
while (hdma_usart1_rx.Instance->CR&DMA_SxCR_EN)
{
     __HAL_DMA_DISABLE(&hdma_usart1_rx);
}

__HAL_DMA_CLEAR_FLAG(&hdma_usart1_rx,DMA_HISR_TCIF5);

hdma_usart1_rx.Instance->PAR=(uint32_t)&(USART1->DR);
hdma_usart1_rx.Instance->M0AR=(uint32_t)(rx_buf);//rx_buf为自定义的储存地址
hdma_usart1_rx.Instance->NDTR=dma_buf_num;

__HAL_DMA_ENABLE(&hdma_usart1_rx);//使能串口dma接收
```

## USART DMA RX 使用配置

```c
//在中断回调函数中，或定义原中断回调函数为__weak然后重写它
void USART1_IRQHandler(void)
{
    static volatile uint8_t res;
    if(USART1->SR & UART_FLAG_IDLE)
    {
        __HAL_UART_CLEAR_PEFLAG(&huart1);//读取UART1-SR 和UART1-DR; 清除中断标志位

        __HAL_DMA_DISABLE(huart1.hdmarx); //失能dma_rx

        Vision_read_data(&usart1_receive_buf[0]);//解析数据信息

        memset(&usart1_receive_buf[0],0,VISION_BUFFER_SIZE);//置0

        __HAL_DMA_CLEAR_FLAG(huart1.hdmarx,DMA_HISR_TCIF5); //清除传输完成标志位

        __HAL_DMA_SET_COUNTER(huart1.hdmarx, VISION_BUFFER_SIZE);//设置DMA 搬运数据大小 单位为字节

        __HAL_DMA_ENABLE(huart1.hdmarx); //使能DMAR

    }


}
```

### USART DMA TX init配置

```c
//init流程
      SET_BIT(huart1.Instance->CR3, USART_CR3_DMAT);
    //失效DMA
    __HAL_DMA_DISABLE(&hdma_usart1_tx);

//第一次调用时硬件会将CR的位0 EN位清零
//意思为DMA准备好配置数据流0&1=0
//此次用于为前面那个disable的成功与否做处理
//若成功则跳出循环，失败则继续disable
    while(hdma_usart1_tx.Instance->CR & DMA_SxCR_EN)
    {
        __HAL_DMA_DISABLE(&hdma_usart1_tx);
    }

    hdma_usart1_tx.Instance->PAR = (uint32_t) & (USART1->DR);
//为了后面使用usart1_tx_enable重设发送储存器的地址，所以设定为null
    hdma_usart1_tx.Instance->M0AR = (uint32_t)(NULL);
    hdma_usart1_tx.Instance->NDTR = 0;


```

## USART DMA TX使用配置

```c
//发送函数
void usart1_tx_dma_enable(uint8_t *data, uint16_t len)
{
    //disable DMA
    //失效DMA
    __HAL_DMA_DISABLE(&hdma_usart1_tx);

    while(hdma_usart1_tx.Instance->CR & DMA_SxCR_EN)
    {
        __HAL_DMA_DISABLE(&hdma_usart1_tx);
    }

    __HAL_DMA_CLEAR_FLAG(&hdma_usart1_tx, DMA_HISR_TCIF7);

    //重设储存器地址为要发送的数据的地址,以及数据长度
    hdma_usart1_tx.Instance->M0AR = (uint32_t)(data);
    __HAL_DMA_SET_COUNTER(&hdma_usart1_tx, len);

    __HAL_DMA_ENABLE(&hdma_usart1_tx);
}
```
