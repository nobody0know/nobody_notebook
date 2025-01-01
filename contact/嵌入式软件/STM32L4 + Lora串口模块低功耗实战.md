# STM32L4 + Lora串口模块低功耗实战

很早就像搞低功耗了，大一时的ESP32物联网开灯器一周一个充电宝的耗电量我一直都印象非常深刻，碰巧遇上物联网通信的课程实验需要使用STM32L4的开发板实现低功耗的功能的选做实验，于是就试试了，先上代码

```c
void enter_low_power_mode()
{
    HAL_UART_DeInit(&huart2);
    HAL_UART_DeInit(&huart1);
    GPIO_InitTypeDef GPIO_InitStruct = {0};

    //GPIO_InitStruct.Pin = GPIO_PIN_3; //PA3 UART2 RX
    GPIO_InitStruct.Pin = GPIO_PIN_1; //唤醒模式下，模块AUX脚即接入到MCU的PE1会提前发出高电平唤醒MCU
    GPIO_InitStruct.Mode = GPIO_MODE_IT_RISING;
    GPIO_InitStruct.Pull = GPIO_PULLUP;
    HAL_GPIO_Init(GPIOE, &GPIO_InitStruct);

    /* EXTI interrupt init*/
    HAL_NVIC_SetPriority(EXTI1_IRQn, 0, 0);
    HAL_NVIC_EnableIRQ(EXTI1_IRQn);

    HAL_PWR_EnterSTOPMode(PWR_LOWPOWERREGULATOR_ON,PWR_STOPENTRY_WFI);

}
```

```c
void HAL_GPIO_EXTI_Callback(uint16_t GPIO_Pin)
{
    if(GPIO_Pin == GPIO_PIN_3)
    {
        HAL_Init();
        SystemClock_Config();
        huart2.gState = HAL_UART_STATE_RESET;
        huart1.gState = HAL_UART_STATE_RESET;
        MX_USART2_UART_Init();
        MX_USART1_UART_Init();
       // HAL_NVIC_DisableIRQ(EXTI3_IRQn);//开了UART2 RX的普通IO中断要关掉，串口init不会给你关掉的
       HAL_NVIC_DisableIRQ(EXTI1_IRQn);
        __HAL_UART_ENABLE_IT(&huart2,UART_IT_RXNE);
        __HAL_UART_ENABLE_IT(&huart2,UART_IT_IDLE);
        __HAL_UART_ENABLE_IT(&huart1,UART_IT_RXNE);
        __HAL_UART_ENABLE_IT(&huart1,UART_IT_IDLE);

    }
}
```
