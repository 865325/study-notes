STM32

### GPIO

1. RCC使能，开启GPIO的时钟

	```c
	/* RCC开启GPIO的时钟，这里开启的是GPIOA */
	RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOA, ENABLE);
	```

2. 使用GPIO_Init函数初始化GPIO

      ```c
      /* 配置GPIO初始化参数 */
      GPIO_InitTypeDef GPIO_InitStruct;
      GPIO_InitStruct.GPIO_Mode = GPIO_Mode_Out_PP; 	// 使用推挽输出，开漏输出下高电平没有驱动能力
      // GPIO_InitStruct.GPIO_Mode = GPIO_Mode_IPU;	// 上拉输入，默认是高电平
      GPIO_InitStruct.GPIO_Pin = GPIO_Pin_0;			// 使用的是A_0，所以选择0号引脚
      GPIO_InitStruct.GPIO_Speed = GPIO_Speed_50MHz;	// 时钟频率为50MHz
      GPIO_Init(GPIOA, &GPIO_InitStruct);				// 初始化GPIO，这里是GPIOA的0号引脚，A0
      ```

      ```c
      GPIO_InitStruct.GPIO_Pin = GPIO_Pin_0 | GPIO_Pin_1;		// 0号引脚和1号引脚同时初始化，还可以再|其他需要的引脚
      GPIO_InitStruct.GPIO_Pin = GPIO_Pin_All;		// 同时初始化所有引脚
      ```

3. 使用输出或输入函数控制GPIO口
	
	- 输出模式
	
	```c
	/* 使用输出或输入函数控制GPIO口 */
	GPIO_ResetBits(GPIOA, GPIO_Pin_0);				// 复位A0口，即置A0口电平为0
	GPIO_SetBits(GPIOA, GPIO_Pin_0);				// 置A0口电平为1
	GPIO_WriteBit(GPIOA, GPIO_Pin_0, Bit_SET);		// 设置A0口电平为0，第三个参数为Bit_SET时设置电平为1
	GPIO_Write(GPIOA, ~0x0001);						// 设置A0口低电平，GPIO_Write可以同时设置多个引脚的电平值
	GPIO_Write(GPIOA, ~0x0003);						// 设置A0, A1口低电平
	```
	- 输入模式
	
	```c
	GPIO_ReadInputDataBit(GPIOB, GPIO_Pin_1);		// 读取B1端口值
	GPIO_ReadOutputDataBit(GPIOA, GPIO_Pin_0);		// 读取输出寄存器中A0的值，这时A0是作为输出口

---

### EXTI外部中断

#### 简介

EXTI可以监测指定GPIO口的电平信号，当其指定的GPIO口产生电平变化时，EXTI将立即向NVIC发出中断申请，经过NVIC裁决后即可中断CPU主程序，使CPU执行EXTI对应的中断程序

#### 流程

1. 开启涉及的外设的时钟，GPIO和AFIO

	```c
	// 开启时钟
	RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOB, ENABLE);
	RCC_APB2PeriphClockCmd(RCC_APB2Periph_AFIO, ENABLE);
	```

2. 配置GPIO

	```c
	// 配置GPIO
	GPIO_InitTypeDef GPIO_InitStruct;
	GPIO_InitStruct.GPIO_Mode = GPIO_Mode_IPU;
	GPIO_InitStruct.GPIO_Pin = GPIO_Pin_14;
	GPIO_InitStruct.GPIO_Speed = GPIO_Speed_50MHz;
	GPIO_Init(GPIOB, &GPIO_InitStruct);
	```

3. 配置AFIO

	```c
	// 配置AFIO，PB14引脚的电平信号可以通过AFIO，进入EXTI电路
	GPIO_EXTILineConfig(GPIO_PortSourceGPIOB, GPIO_PinSource14);
	```

4. 配置EXTI

	```c
	// 配置EXTI，将EXTI的第14个线路配置为中断模式
	// 下降沿触发中断，中断响应
	EXTI_InitTypeDef EXTI_InitStruct;
	EXTI_InitStruct.EXTI_Line = EXTI_Line14;					// 选用14号线
	EXTI_InitStruct.EXTI_LineCmd = ENABLE;						// 使能，开启中断
	EXTI_InitStruct.EXTI_Mode = EXTI_Mode_Interrupt;			// 选用中断模式，这里选用中断响应
	EXTI_InitStruct.EXTI_Trigger = EXTI_Trigger_Falling;		// 选择下降沿触发中断
	EXTI_Init(&EXTI_InitStruct);
	```

5. 配置NVIC

	```c
	// 配置NVIC
	NVIC_PriorityGroupConfig(NVIC_PriorityGroup_2);		// 配置优先级分组，两位抢占，两位响应
	NVIC_InitTypeDef NVIC_InitStruct;
	NVIC_InitStruct.NVIC_IRQChannel = EXTI15_10_IRQn;		// 配置中断通道
	NVIC_InitStruct.NVIC_IRQChannelCmd = ENABLE;			// 使能
	NVIC_InitStruct.NVIC_IRQChannelPreemptionPriority = 1;	// 配置抢占优先级
	NVIC_InitStruct.NVIC_IRQChannelSubPriority = 1;			// 配置响应优先级
	NVIC_Init(&NVIC_InitStruct);
	```

6. 重写中断函数，中断函数名字需要和中断通道配合，在startup_stm32f10x_md.s中可以找到

	```c
	void EXTI15_10_IRQHandler(void)
	{
		// 获取中断标志位，判断指定的EXTI中断线是否被触发
		if(EXTI_GetITStatus(EXTI_Line14) == SET)
		{
			// 程序主逻辑
			
			// 清除中断标志位，防止多次触发中断
			EXTI_ClearITPendingBit(EXTI_Line14);
		}
	}
	```

	---
	

### TIM定时中断

#### 简介

- 定时器可以对输入的时钟进行计数，并在计数值达到设定值时触发中断
- 16位计数器、预分频器、自动重装寄存器的时基单元，在72MHz计数时钟下可以实现最大59.65s的定时
	- 预分频器：对内部时钟进行分频，其值与实际分频系数相差1，比如预分频器值为1，则输出频率 = 输入频率 / 2
	- 16位计数器：每来一个上升沿，计数加1
	- 自动重装寄存器：存放预先设置好的阈值，当计数器到达阈值，触发中断且置计算器值为0
- 不仅具备基本的定时中断功能，而且还包含内外时钟源选择、输入捕获、输出比较、编码器接口、主从触发模式等多种功能
- 根据复杂度和应用场景分为了高级定时器、通用定时器、基本定时器三种类型

#### 内部时钟源

1. 开启APB1的时钟，TIM2是APB1总线的外设

	```c
	// 开启APB1的时钟，TIM2是APB1总线的外设
	RCC_APB1PeriphClockCmd(RCC_APB1Periph_TIM2, ENABLE);
	```

2. TIM2选择内部时钟源

	```c
	// TIM2选择内部时钟源，TIM2的时基单元由内部时钟驱动
	TIM_InternalClockConfig(TIM2);
	```

3. 配置时基单元

	```c
	// 配置时基单元
	// 定时频率  = CK_PSC / (PSC + 1) / (ARR + 1)
	// 下面 72M / 7200 / 10000 = 1，定时频率为1HZ，即1s触发定时器一次
	TIM_TimeBaseInitTypeDef TIM_TimeBaseInitStruct;
	TIM_TimeBaseInitStruct.TIM_ClockDivision = TIM_CKD_DIV1;
	TIM_TimeBaseInitStruct.TIM_CounterMode = TIM_CounterMode_Up;	// 计数器模式，向上计数
	TIM_TimeBaseInitStruct.TIM_Period = 10000 - 1;						// ARR自动重装器的值
	TIM_TimeBaseInitStruct.TIM_Prescaler = 7200 - 1;				// PSC预分频器的值
	TIM_TimeBaseInitStruct.TIM_RepetitionCounter = 0;				// 高级定时器才有的，这里直接配置0即可
	TIM_TimeBaseInit(TIM2, &TIM_TimeBaseInitStruct);
	TIM_ClearFlag(TIM2, TIM_IT_Update);								// 清除中断标志位，防止一开始就直接进入中断
	```

4. 使能中断

	```c
	// 使能中断，打通了中断到NVIC的道路
	TIM_ITConfig(TIM2, TIM_IT_Update, ENABLE);
	```

5. 配置NVIC

	```c
	// 配置NVIC
	NVIC_PriorityGroupConfig(NVIC_PriorityGroup_2);			// 配置优先级分组，两位抢占，两位响应
	NVIC_InitTypeDef NVIC_InitStruct;
	NVIC_InitStruct.NVIC_IRQChannel = TIM2_IRQn;			// 配置中断通道
	NVIC_InitStruct.NVIC_IRQChannelCmd = ENABLE;			// 使能
	NVIC_InitStruct.NVIC_IRQChannelPreemptionPriority = 1;	// 配置抢占优先级
	NVIC_InitStruct.NVIC_IRQChannelSubPriority = 1;			// 配置响应优先级
	NVIC_Init(&NVIC_InitStruct);
	```

6. 启动定时器

	```c
	// 启动定时器
	TIM_Cmd(TIM2, ENABLE);
	```

7. 重写中断函数

	```c
	/**
	  * @brief  TIM2的中断函数
	  * @param  
	  * @retval 
	  */
	void TIM2_IRQHandler(void)
	{
		// 检查中断标志位，TIM2的更新中断
		if(TIM_GetITStatus(TIM2, TIM_IT_Update) == SET)
		{
			// 程序主逻辑
			
			// 清除中断标志位，防止多次触发中断
			TIM_ClearITPendingBit(TIM2, TIM_IT_Update);
		}
	}
	```

#### 外部时钟源

步骤与内部时钟源差不多，但第二步选择外部时钟源ETR，TIM2的ETR引脚是PA0，因此预先配置GPIO

```c
// 配置GPIO的PA0口
RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOA, ENABLE);
GPIO_InitTypeDef GPIO_InitStruct;
GPIO_InitStruct.GPIO_Mode = GPIO_Mode_IPU;
GPIO_InitStruct.GPIO_Pin = GPIO_Pin_0;
GPIO_InitStruct.GPIO_Speed = GPIO_Speed_50MHz;
GPIO_Init(GPIOA, &GPIO_InitStruct);

// TIM2选择外部时钟源，ETR引脚的外部时钟模式2
// 外部分频，即外部时钟预先分频，选择不需要分频
// 外部触发的极性，选择上升沿有效
// 外部触发滤波器，以采样频率f采样n个点，n个点一样才有效输出，0x00表示不需要滤波器
TIM_ETRClockMode2Config(TIM2, TIM_ExtTRGPSC_OFF, TIM_ExtTRGPolarity_NonInverted, 0x0F);
```

#### 输出比较

##### 简介

输出比较可以通过比较CNT与CCR寄存器值的关系，来对输出电平进行置1、置0或翻转的操作，用于输出一定频率和占空比的PWM波形

##### PWM（Pulse Width Modulation）脉冲宽度调制

在具有惯性的系统中，可以通过对一系列脉冲的宽度进行调制，来等效地获得所需要的模拟参量，常应用于电机控速等领域

PWM频率： Freq = CK_PSC / (PSC + 1) / (ARR + 1)
PWM占空比： Duty = CCR / (ARR + 1)
PWM分辨率： Reso = 1 / (ARR + 1)

##### 流程

1. 开启APB1的时钟，选择内部时钟，配置时基单元

	```c
	// 开启APB1的时钟，TIM2是APB1总线的外设
	RCC_APB1PeriphClockCmd(RCC_APB1Periph_TIM2, ENABLE);
	
	// TIM2选择内部时钟源，TIM2的时基单元由内部时钟驱动
	TIM_InternalClockConfig(TIM2);
	
	// 配置时基单元
	// PWM频率： Freq = CK_PSC / (PSC + 1) / (ARR + 1)
	// PWM占空比： Duty = CCR / (ARR + 1)
	// PWM分辨率： Reso = 1 / (ARR + 1)
	TIM_TimeBaseInitTypeDef TIM_TimeBaseInitStruct;
	TIM_TimeBaseInitStruct.TIM_ClockDivision = TIM_CKD_DIV1;
	TIM_TimeBaseInitStruct.TIM_CounterMode = TIM_CounterMode_Up;	// 计数器模式，向上计数
	TIM_TimeBaseInitStruct.TIM_Period = 100 - 1;					// ARR自动重装器的值
	TIM_TimeBaseInitStruct.TIM_Prescaler = 720 - 1;				// PSC预分频器的值
	TIM_TimeBaseInitStruct.TIM_RepetitionCounter = 0;				// 高级定时器才有的，这里直接配置0即可
	TIM_TimeBaseInit(TIM2, &TIM_TimeBaseInitStruct);
	```

2. 配置PWM波形输出口

	```c
	// TIM2的OC1对应的TIM2_CH1_ETR默认输出口为PA0
	RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOA, ENABLE);
	GPIO_InitTypeDef GPIO_InitStruct;
	GPIO_InitStruct.GPIO_Mode = GPIO_Mode_AF_PP;		// 复用推挽输出，将输出权交给片上外设，TIM2
	GPIO_InitStruct.GPIO_Pin = GPIO_Pin_0;
	GPIO_InitStruct.GPIO_Speed = GPIO_Speed_50MHz;
	GPIO_Init(GPIOA, &GPIO_InitStruct);
	```
3. 初始化输出比较单元

	```c
	// 初始化输出比较单元，这里使用PA0口，对应OC1
	TIM_OCInitTypeDef TIM_OCInitStruct;
	TIM_OCStructInit(&TIM_OCInitStruct);	// 赋初始值，防止之后配置高级寄存器出现某些变量未赋值的情况
	TIM_OCInitStruct.TIM_OCMode = TIM_OCMode_PWM1;			// 设置输出比较模式，配置成PWM模式
	TIM_OCInitStruct.TIM_OCPolarity = TIM_OCPolarity_High;		// 设置输出比较的极性，这里设置极性不翻转，REF波形直接输出
	TIM_OCInitStruct.TIM_OutputState = TIM_OutputState_Enable;	// 设置输出使能
	TIM_OCInitStruct.TIM_Pulse = 0;			// 设置CCR
	TIM_OC1Init(TIM2, &TIM_OCInitStruct);
	```
4. 启动定时器

	```c
	// 启动定时器
	TIM_Cmd(TIM2, ENABLE);
	```

##### 其他

1. 重新配置CRR的值

	```c
	/**
	  * @brief  更新PWM波形的占空比，通过重新配置CRR的值
	  * @param  CRR的新值
	  * @retval 
	  */
	void PWM_SetCompare(uint16_t compare)
	{
		TIM_SetCompare1(TIM2, compare);		// 重配置TIM2的输出比较通道1
	}
	```
	
2. 重新配置PSC的值

	```c
	/**
      * @brief  更新预分频的值
      * @param  
      * @retval 
      */
    void PWM_SetPrescaler(uint16_t Prescaler)
    {
        // 这里配置为预分频器的值立刻生效
        TIM_PrescalerConfig(TIM2, Prescaler, TIM_PSCReloadMode_Immediate);
    }
   ```
	
3. 引脚重映射（TIM2的CH1可以从PA0挪到PA15引脚上）

	```c
	// 使能AFIO
	RCC_APB2PeriphClockCmd(RCC_APB2Periph_AFIO, ENABLE);
	
	// 将TIM2的CH1从PA0重映射到PA15
	GPIO_PinRemapConfig(GPIO_PartialRemap1_TIM2, ENABLE);
	GPIO_PinRemapConfig(GPIO_Remap_SWJ_JTAGDisable, ENABLE);	// 解除PA15的JTAG的调试端口的复用，使其成为正常的GPIO口
	```


#### 输入捕获

##### 简介

- 输入捕获模式下，当通道输入引脚出现指定电平跳变时，当前CNT的值将被锁存到CCR中，可用于测量PWM波形的频率、占空比、脉冲间隔、电平持续时间等参数

- 可配置为PWMI模式，同时测量频率和占空比

- 可配合主从触发模式，实现硬件全自动测量

![image-1](.\assets\image-1.png)

##### 流程

1.  开启时钟

	```c
	// 开启时钟
	RCC_APB1PeriphClockCmd(RCC_APB1Periph_TIM3, ENABLE);
	RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOA, ENABLE);
	```

2. 配置GPIO

	```c
	// 配置GPIO
	RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOA, ENABLE);
	GPIO_InitTypeDef GPIO_InitStruct;
	GPIO_InitStruct.GPIO_Mode = GPIO_Mode_IPU;
	GPIO_InitStruct.GPIO_Pin = GPIO_Pin_6;
	GPIO_InitStruct.GPIO_Speed = GPIO_Speed_50MHz;
	GPIO_Init(GPIOA, &GPIO_InitStruct);
	```

3. 配置时基单元

	```c
    // 由内部时钟驱动
    TIM_InternalClockConfig(TIM3);

	// 配置时基单元
	TIM_TimeBaseInitTypeDef TIM_TimeBaseInitStruct;
	TIM_TimeBaseInitStruct.TIM_ClockDivision = TIM_CKD_DIV1;
	TIM_TimeBaseInitStruct.TIM_CounterMode = TIM_CounterMode_Up;
	TIM_TimeBaseInitStruct.TIM_Period = 65536 - 1;
	TIM_TimeBaseInitStruct.TIM_Prescaler = 72 - 1;
	TIM_TimeBaseInitStruct.TIM_RepetitionCounter = 0;
	TIM_TimeBaseInit(TIM3, &TIM_TimeBaseInitStruct);
	```

4. 初始化输入捕获单元

	```c
	// 初始化输入捕获单元
	TIM_ICInitTypeDef TIM_ICInitStruct;
	TIM_ICInitStruct.TIM_Channel = TIM_Channel_1;	// 选择TIM3的通道1
	TIM_ICInitStruct.TIM_ICFilter = 0xF;			// 滤波器，去噪声
	TIM_ICInitStruct.TIM_ICPolarity = TIM_ICPolarity_Rising;		// 选择上升沿触发
	TIM_ICInitStruct.TIM_ICPrescaler = TIM_ICPSC_DIV1;				// 选择不分频
	TIM_ICInitStruct.TIM_ICSelection = TIM_ICSelection_DirectTI;	// 选择直连通道输入
	TIM_ICInit(TIM3, &TIM_ICInitStruct);
	```

5. 配置主从触发

	```c
	// 配置TRGI的触发源为TI1FP1
	TIM_SelectInputTrigger(TIM3, TIM_TS_TI1FP1);
	// 配置从模式的触发模式为Reset
	TIM_SelectSlaveMode(TIM3, TIM_SlaveMode_Reset);
	```

6. 开启定时器

	```c
	// 启动定时器
	TIM_Cmd(TIM3, ENABLE);
	```

##### PWMI

流程和之前测量频率差不多，不过第四步初始化时基单元更改为下面代码

```c
// 初始化输入捕获单元
TIM_ICInitTypeDef TIM_ICInitStruct;
TIM_ICInitStruct.TIM_Channel = TIM_Channel_1;	// 选择TIM3的通道1
TIM_ICInitStruct.TIM_ICFilter = 0xF;			// 滤波器，去噪声
TIM_ICInitStruct.TIM_ICPolarity = TIM_ICPolarity_Rising;		// 选择上升沿触发
TIM_ICInitStruct.TIM_ICPrescaler = TIM_ICPSC_DIV1;				// 选择不分频
TIM_ICInitStruct.TIM_ICSelection = TIM_ICSelection_DirectTI;	// 选择直连通道输入
// 配置PWMI，这里把通道2的初始化为与通道1相反的配置（下降沿，交叉通道输入）
// 同时初始化通道1，2
TIM_PWMIConfig(TIM3, &TIM_ICInitStruct);
```

##### 其他

1. 获取频率

	```c
	/**
	  * @brief  返回输入信号的频率，这里采用测周法，𝑓𝑥=𝑓𝑐 / 𝑁
	  * @param  
	  * @retval 
	  */
	uint32_t IC_GetFreq(void)
	{
		return 1000000 / (TIM_GetCapture1(TIM3) + 1);
	}
	```

2. 获取占空比

	```c
	/**
	  * @brief  获取占空比
	  * @param  
	  * @retval 
	  */
	uint32_t IC_GetDuty(void)
	{
		return (TIM_GetCapture2(TIM3) + 1) * 100 / (TIM_GetCapture1(TIM3) + 1);
	}
	```

#### 编码器接口

##### 简介

编码器接口可接收增量（正交）编码器的信号，根据编码器旋转产生的正交信号脉冲，自动控制CNT自增或自减，从而指示编码器的位置、旋转方向和旋转速度

##### 流程

1. 开启时钟

	```c
	// 开启时钟
	RCC_APB1PeriphClockCmd(RCC_APB1Periph_TIM3, ENABLE);
	RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOA, ENABLE);
	```

2. 配置GPIO

	```c
	// 配置GPIO
	RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOA, ENABLE);
	GPIO_InitTypeDef GPIO_InitStruct;
	GPIO_InitStruct.GPIO_Mode = GPIO_Mode_IPU;
	GPIO_InitStruct.GPIO_Pin = GPIO_Pin_6 | GPIO_Pin_7;
	GPIO_InitStruct.GPIO_Speed = GPIO_Speed_50MHz;
	GPIO_Init(GPIOA, &GPIO_InitStruct);
	```

3. 配置时基单元

	```c
	// 配置时基单元
	TIM_TimeBaseInitTypeDef TIM_TimeBaseInitStruct;
	TIM_TimeBaseInitStruct.TIM_ClockDivision = TIM_CKD_DIV1;
	TIM_TimeBaseInitStruct.TIM_CounterMode = TIM_CounterMode_Up;
	TIM_TimeBaseInitStruct.TIM_Period = 65536 - 1;
	TIM_TimeBaseInitStruct.TIM_Prescaler = 1 - 1;
	TIM_TimeBaseInitStruct.TIM_RepetitionCounter = 0;
	TIM_TimeBaseInit(TIM3, &TIM_TimeBaseInitStruct);
	```

4. 初始化输入捕获单元

	```c
	// 初始化输入捕获单元
	TIM_ICInitTypeDef TIM_ICInitStruct;
	TIM_ICStructInit(&TIM_ICInitStruct);
	TIM_ICInitStruct.TIM_Channel = TIM_Channel_1;	// 选择TIM3的通道1
	TIM_ICInitStruct.TIM_ICFilter = 0xF;			// 滤波器，去噪声
	TIM_ICInit(TIM3, &TIM_ICInitStruct);
	
	TIM_ICInitStruct.TIM_Channel = TIM_Channel_2;	// 选择TIM3的通道2
	TIM_ICInitStruct.TIM_ICFilter = 0xF;			// 滤波器，去噪声
	TIM_ICInit(TIM3, &TIM_ICInitStruct);

5. 配置编码器接口

	```c
	// 配置编码器接口
	TIM_EncoderInterfaceConfig(TIM3, TIM_EncoderMode_TI12, TIM_ICPolarity_Rising, TIM_ICPolarity_Rising);
	```

6. 启动定时器

	```c
	// 启动定时器
	TIM_Cmd(TIM3, ENABLE);
	```


### ADC数模转换器

#### 简介

- ADC可以将引脚上连续变化的模拟电压转换为内存中存储的数字变量，建立模拟电路到数字电路的桥梁

- 12位逐次逼近型ADC，1us转换时间

- 输入电压范围：0 - 3.3v，转换结果范围：0 ~ 4095

#### 基本结构图

![image-2](.\assets\image-2.png)

#### 单通道

1. 开启时钟

	```c
	// 开启时钟
	RCC_APB2PeriphClockCmd(RCC_APB2Periph_ADC1, ENABLE);
	RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOA, ENABLE);
	```

2. 配置ADCCLK

	```c
	// 配置ADCCLK
	RCC_ADCCLKConfig(RCC_PCLK2_Div6);	// ADCCLK = 72M / 6 = 12MHz
	```

3. 配置GPIO

	```c
	// 配置GPIO
	GPIO_InitTypeDef GPIO_InitStruct;
	GPIO_InitStruct.GPIO_Mode = GPIO_Mode_AIN;			// 模拟输入
	GPIO_InitStruct.GPIO_Pin = GPIO_Pin_0;
	GPIO_InitStruct.GPIO_Speed = GPIO_Speed_50MHz;
	GPIO_Init(GPIOA, &GPIO_InitStruct);
	```

4. 选择规则组的输入通道

	```c
	// 选择规则组的输入通道
	// 将ADC1的通道0放在规则组的序列1，且配置采样时间为55个ADCCLK
	ADC_RegularChannelConfig(ADC1, ADC_Channel_0, 1, 	ADC_SampleTime_55Cycles5);
	```

5. 初始化ADC

	```c
	// 初始化ADC
	ADC_InitTypeDef ADC_InitStruct;
	ADC_InitStruct.ADC_ContinuousConvMode = DISABLE;	// 单次转换
	ADC_InitStruct.ADC_DataAlign = ADC_DataAlign_Right;	// 数据对齐为右对齐
	ADC_InitStruct.ADC_ExternalTrigConv = ADC_ExternalTrigConv_None;	// 触发控制的触发源，使用软件触发
	ADC_InitStruct.ADC_Mode = ADC_Mode_Independent;		// ADC工作模式为独立模式
	ADC_InitStruct.ADC_NbrOfChannel = 1;				// 规则组通道数目，仅在扫描模式下生效
	ADC_InitStruct.ADC_ScanConvMode = DISABLE;			// 非扫描模式
	ADC_Init(ADC1, &ADC_InitStruct);
	```

6. 启动ADC

	```c
	// 启动ADC
	ADC_Cmd(ADC1, ENABLE);
	```

7. 校准ADC

	```c
	// 校准ADC
	ADC_ResetCalibration(ADC1);
	while(ADC_GetResetCalibrationStatus(ADC1) == SET);
	ADC_StartCalibration(ADC1);
	while(ADC_GetCalibrationStatus(ADC1) == SET);
	```

#### 其他

1. 软件触发中断并获取转换值

	```c
	/**
	  * @brief  软件触发ADC转换
	  * @param  
	  * @retval 
	  */
	uint16_t AD_GetValue(void)
	{
		// 软件触发ADC转换
		ADC_SoftwareStartConvCmd(ADC1, ENABLE);
		// 转换完成标志位
		while(ADC_GetFlagStatus(ADC1, ADC_FLAG_EOC) != SET);
		// 获取转换值
		return ADC_GetConversionValue(ADC1);
	}
	```


### DMA

#### 简介

DMA可以提供外设和存储器或者存储器和存储器之间的高速数据传输，无须CPU干预，节省了CPU的资源

#### DMA转运数据

1. 开启时钟

	```c
	// 开启时钟
	RCC_AHBPeriphClockCmd(RCC_AHBPeriph_DMA1, ENABLE);
	```
	
2. 配置DMA参数

	```c
	// 配置DMA参数
	// 配置 AddrA -> AddrB，转运数据以字节为单位，且设置为自增，即转运完单个数据，下次转运从下个地址开始
	DMA_InitTypeDef DMA_InitStruct;
	DMA_InitStruct.DMA_PeripheralBaseAddr = AddrA;
	DMA_InitStruct.DMA_PeripheralDataSize = DMA_PeripheralDataSize_Byte;
	DMA_InitStruct.DMA_PeripheralInc = DMA_PeripheralInc_Enable;
	DMA_InitStruct.DMA_MemoryBaseAddr = AddrB;
	DMA_InitStruct.DMA_MemoryDataSize = DMA_MemoryDataSize_Byte;
	DMA_InitStruct.DMA_MemoryInc = DMA_MemoryInc_Enable;
	
	DMA_InitStruct.DMA_DIR = DMA_DIR_PeripheralSRC;		// 设置传输方向，选择外设地址作为源地址
	DMA_InitStruct.DMA_BufferSize = Size;		// 设置传输计数器，即传输数据量
	DMA_InitStruct.DMA_Mode = DMA_Mode_Normal;	// 设置传输计数器是否重装，选择不重装（重装即转运完一轮数据，立刻从头再次转运）
	DMA_InitStruct.DMA_M2M = DMA_M2M_Enable;	// 设置触发方式为软件触发
	DMA_InitStruct.DMA_Priority = DMA_Priority_VeryHigh;	// 设置转运优先级
	DMA_Init(DMA1_Channel1, &DMA_InitStruct);
	```
	
3. 启动DMA

	```c
	// 启动DMA
	DMA_Cmd(DMA1_Channel1, ENABLE);
	```

#### DMA+AD多通道

1. 开启时钟

	```c
	/*开启时钟*/
	RCC_APB2PeriphClockCmd(RCC_APB2Periph_ADC1, ENABLE);	//开启ADC1的时钟
	RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOA, ENABLE);	//开启GPIOA的时钟
	RCC_AHBPeriphClockCmd(RCC_AHBPeriph_DMA1, ENABLE);		//开启DMA1的时钟
	```

2. 设置ADC时钟

	```c
	/*设置ADC时钟*/
	RCC_ADCCLKConfig(RCC_PCLK2_Div6);						//选择时钟6分频，ADCCLK = 72MHz / 6 = 12MHz
	```
	
3. GPIO初始化

	```c
   /*GPIO初始化*/
   GPIO_InitTypeDef GPIO_InitStructure;
   GPIO_InitStructure.GPIO_Mode = GPIO_Mode_AIN;
   GPIO_InitStructure.GPIO_Pin = GPIO_Pin_0 | GPIO_Pin_1 | GPIO_Pin_2 | GPIO_Pin_3;
   GPIO_InitStructure.GPIO_Speed = GPIO_Speed_50MHz;
   GPIO_Init(GPIOA, &GPIO_InitStructure);					//将PA0、PA1、PA2和PA3引脚初始化为模拟输入
	```

4. 规则组通道配置

	```c
    /*规则组通道配置*/
    ADC_RegularChannelConfig(ADC1, ADC_Channel_0, 1, ADC_SampleTime_55Cycles5);	//规则组序列1的位置，配置为通道0
    ADC_RegularChannelConfig(ADC1, ADC_Channel_1, 2, ADC_SampleTime_55Cycles5);	//规则组序列2的位置，配置为通道1
    ADC_RegularChannelConfig(ADC1, ADC_Channel_2, 3, ADC_SampleTime_55Cycles5);	//规则组序列3的位置，配置为通道2
    ADC_RegularChannelConfig(ADC1, ADC_Channel_3, 4, ADC_SampleTime_55Cycles5);	//规则组序列4的位置，配置为通道3
	```

5. ADC初始化

	```c
    /*ADC初始化*/
    ADC_InitTypeDef ADC_InitStructure;											//定义结构体变量
    ADC_InitStructure.ADC_Mode = ADC_Mode_Independent;							//模式，选择独立模式，即单独使用ADC1
    ADC_InitStructure.ADC_DataAlign = ADC_DataAlign_Right;						//数据对齐，选择右对齐
    ADC_InitStructure.ADC_ExternalTrigConv = ADC_ExternalTrigConv_None;			//外部触发，使用软件触发，不需要外部触发
    ADC_InitStructure.ADC_ContinuousConvMode = ENABLE;							//连续转换，使能，每转换一次规则组序列后立刻开始下一次转换
    ADC_InitStructure.ADC_ScanConvMode = ENABLE;								//扫描模式，使能，扫描规则组的序列，扫描数量由ADC_NbrOfChannel确定
    ADC_InitStructure.ADC_NbrOfChannel = 4;										//通道数，为4，扫描规则组的前4个通道
    ADC_Init(ADC1, &ADC_InitStructure);											//将结构体变量交给ADC_Init，配置ADC1
	```

6. DMA初始化

	```c
    /*DMA初始化*/
    DMA_InitTypeDef DMA_InitStructure;											//定义结构体变量
    DMA_InitStructure.DMA_PeripheralBaseAddr = (uint32_t)&ADC1->DR;				//外设基地址，给定形参AddrA
    DMA_InitStructure.DMA_PeripheralDataSize = DMA_PeripheralDataSize_HalfWord;	//外设数据宽度，选择半字，对应16为的ADC数据寄存器
    DMA_InitStructure.DMA_PeripheralInc = DMA_PeripheralInc_Disable;			//外设地址自增，选择失能，始终以ADC数据寄存器为源
    DMA_InitStructure.DMA_MemoryBaseAddr = (uint32_t)AD_Value;					//存储器基地址，给定存放AD转换结果的全局数组AD_Value
    DMA_InitStructure.DMA_MemoryDataSize = DMA_MemoryDataSize_HalfWord;			//存储器数据宽度，选择半字，与源数据宽度对应
    DMA_InitStructure.DMA_MemoryInc = DMA_MemoryInc_Enable;						//存储器地址自增，选择使能，每次转运后，数组移到下一个位置
    DMA_InitStructure.DMA_DIR = DMA_DIR_PeripheralSRC;							//数据传输方向，选择由外设到存储器，ADC数据寄存器转到数组
    DMA_InitStructure.DMA_BufferSize = 4;										//转运的数据大小（转运次数），与ADC通道数一致
    DMA_InitStructure.DMA_Mode = DMA_Mode_Circular;								//模式，选择循环模式，与ADC的连续转换一致
    DMA_InitStructure.DMA_M2M = DMA_M2M_Disable;								//存储器到存储器，选择失能，数据由ADC外设触发转运到存储器
    DMA_InitStructure.DMA_Priority = DMA_Priority_Medium;						//优先级，选择中等
    DMA_Init(DMA1_Channel1, &DMA_InitStructure);								//将结构体变量交给DMA_Init，配置DMA1的通道1
	```

7. DMA和ADC使能

	```c
    /*DMA和ADC使能*/
    DMA_Cmd(DMA1_Channel1, ENABLE);							//DMA1的通道1使能
    ADC_DMACmd(ADC1, ENABLE);								//ADC1触发DMA1的信号使能
    ADC_Cmd(ADC1, ENABLE);									//ADC1使能
	```

8. ADC校准

	```c
    /*ADC校准*/
    ADC_ResetCalibration(ADC1);								//固定流程，内部有电路会自动执行校准
    while (ADC_GetResetCalibrationStatus(ADC1) == SET);
    ADC_StartCalibration(ADC1);
    while (ADC_GetCalibrationStatus(ADC1) == SET);
	```

9. ADC触发

	```c
	/*ADC触发*/
	ADC_SoftwareStartConvCmd(ADC1, ENABLE);	//软件触发ADC开始工作，由于ADC处于连续转换模式，故触发一次后ADC就可以一直连续不断地工作
	```

#### 其他

1. 重新设置传输计数器的值，然后重新转运数据

	```c
	/**
	  * @brief  重新设置传输计数器的值，然后重新转运数据
	  * @param  
	  * @retval 
	  */
	void MyDMA_Transfer(void)
	{
		// DMA失能
		DMA_Cmd(DMA1_Channel1, DISABLE);
		
		// 设置传输计数器
		DMA_SetCurrDataCounter(DMA1_Channel1, MyDMA_Size);
		
		// DMA使能
		DMA_Cmd(DMA1_Channel1, ENABLE);
		
		// 等待转运完成
		while(DMA_GetFlagStatus(DMA1_FLAG_TC1) == RESET);
		DMA_ClearFlag(DMA1_FLAG_TC1);
	}
	```

### 串口通信

#### 简介

- 串口是一种应用十分广泛的通讯接口，串口成本低、容易使用、通信线路简单，可实现两个设备的互相通信
- 单片机的串口可以使单片机与单片机、单片机与电脑、单片机与各式各样的模块互相通信，极大地扩展了单片机的应用范围，增强了单片机系统的硬件实力

#### 硬件电路

- 简单双向串口通信有两根通信线（发送端TX和接收端RX）
- TX与RX要交叉连接
- 当只需单向的数据传输时，可以只接一根通信线
- 当电平标准不一致时，需要加电平转换芯片

#### 电平标准

电平标准是数据1和数据0的表达方式，是传输线缆中人为规定的电压与数据的对应关系，串口常用的电平标准有如下三种：

1. TTL电平：+3.3V或+5V表示1，0V表示0（单片机一般使用）
2. RS232电平：-3~-15V表示1，+3~+15V表示0
3. RS485电平：两线压差+2~+6V表示1，-2~-6V表示0（差分信号）

#### 串口参数

- 波特率：串口通信的速率

- 起始位：标志一个数据帧的开始，固定为低电平

- 数据位：数据帧的有效载荷，1为高电平，0为低电平，低位先行（即低位数据先发送，0xA5发送为10100101）

- 校验位：用于数据验证，根据数据位计算得来（奇偶校验）

- 停止位：用于数据帧间隔，固定为高电平

#### 时序图

- 无校验位

![image-3](.\assets\image-3.png) 

- 有校验位（奇偶校验）

![image-4](.\assets\image-4.png) 

例子

![image-5](.\assets\image-5.png) 

#### 外设（USART）

##### 简介

- USART（Universal Synchronous/Asynchronous Receiver/Transmitter）通用同步/异步收发器

- USART是STM32内部集成的硬件外设，可根据数据寄存器的一个字节数据自动生成数据帧时序，从TX引脚发送出去，也可自动接收RX引脚的数据帧时序，拼接为一个字节数据，存放在数据寄存器里

- 自带波特率发生器，最高达4.5Mbits/s(相当于一个分频器)

- 可配置数据位长度（8/9）、停止位长度（0.5/1/1.5/2）

- 可选校验位（无校验/奇校验/偶校验）

##### USART基本结构

![image-20231129102913426](.\assets\image-6.png)

#### 流程

1. 开启时钟

	```c
	/*开启时钟*/
	RCC_APB2PeriphClockCmd(RCC_APB2Periph_USART1, ENABLE);	//开启USART1的时钟
	RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOA, ENABLE);	//开启GPIOA的时钟
	```

2. GPIO初始化

	```c
	/*GPIO初始化*/
	GPIO_InitTypeDef GPIO_InitStructure;
	GPIO_InitStructure.GPIO_Mode = GPIO_Mode_AF_PP;
	GPIO_InitStructure.GPIO_Pin = GPIO_Pin_9;
	GPIO_InitStructure.GPIO_Speed = GPIO_Speed_50MHz;
	GPIO_Init(GPIOA, &GPIO_InitStructure);					//将PA9引脚初始化为复用推挽输出
	
	GPIO_InitStructure.GPIO_Mode = GPIO_Mode_IPU;
	GPIO_InitStructure.GPIO_Pin = GPIO_Pin_10;
	GPIO_InitStructure.GPIO_Speed = GPIO_Speed_50MHz;
	GPIO_Init(GPIOA, &GPIO_InitStructure);					//将PA10引脚初始化为上拉输入
	```

3. USART初始化

	```c
	USART_InitTypeDef USART_InitStructure;					//定义结构体变量
	USART_InitStructure.USART_BaudRate = 9600;				//波特率
	USART_InitStructure.USART_HardwareFlowControl = USART_HardwareFlowControl_None;	//硬件流控制，不需要
	USART_InitStructure.USART_Mode = USART_Mode_Tx | USART_Mode_Rx;	//模式，发送模式和接收模式均选择
	USART_InitStructure.USART_Parity = USART_Parity_No;		//奇偶校验，不需要
	USART_InitStructure.USART_StopBits = USART_StopBits_1;	//停止位，选择1位
	USART_InitStructure.USART_WordLength = USART_WordLength_8b;		//字长，选择8位
	USART_Init(USART1, &USART_InitStructure);				//将结构体变量交给USART_Init，配置USART1
	```

4. 中断输出配置

	```c
	/*中断输出配置*/
	USART_ITConfig(USART1, USART_IT_RXNE, ENABLE);			//开启串口接收数据的中断
	
	/*NVIC中断分组*/
	NVIC_PriorityGroupConfig(NVIC_PriorityGroup_2);			//配置NVIC为分组2
	
	/*NVIC配置*/
	NVIC_InitTypeDef NVIC_InitStructure;					//定义结构体变量
	NVIC_InitStructure.NVIC_IRQChannel = USART1_IRQn;		//选择配置NVIC的USART1线
	NVIC_InitStructure.NVIC_IRQChannelCmd = ENABLE;			//指定NVIC线路使能
	NVIC_InitStructure.NVIC_IRQChannelPreemptionPriority = 1;		//指定NVIC线路的抢占优先级为1
	NVIC_InitStructure.NVIC_IRQChannelSubPriority = 1;		//指定NVIC线路的响应优先级为1
	NVIC_Init(&NVIC_InitStructure);							//将结构体变量交给NVIC_Init，配置NVIC外设
	```

5. USART使能

	```c
	/*USART使能*/
	USART_Cmd(USART1, ENABLE);								//使能USART1，串口开始运行
	```

#### 其他

1. 发送数据

	```c
	/**
	  * 函    数：串口发送一个字节
	  * 参    数：Byte 要发送的一个字节
	  * 返 回 值：无
	  */
	void Serial_SendByte(uint8_t Byte)
	{
		USART_SendData(USART1, Byte);		//将字节数据写入数据寄存器，写入后USART自动生成时序波形
		while (USART_GetFlagStatus(USART1, USART_FLAG_TXE) == RESET);	//等待发送完成
		/*下次写入数据寄存器会自动清除发送完成标志位，故此循环后，无需清除标志位*/
	}
	```

2. 接收数据（采用中断触发）

	```c
	/**
	  * 函    数：USART1中断函数
	  * 参    数：无
	  * 返 回 值：无
	  * 注意事项：此函数为中断函数，无需调用，中断触发后自动执行
	  *           函数名为预留的指定名称，可以从启动文件复制
	  *           请确保函数名正确，不能有任何差异，否则中断函数将不能进入
	  */
	void USART1_IRQHandler(void)
	{
		if (USART_GetITStatus(USART1, USART_IT_RXNE) == SET)		//判断是否是USART1的接收事件触发的中断
		{
			Serial_RxData = USART_ReceiveData(USART1);				//读取数据寄存器，存放在接收的数据变量
			Serial_RxFlag = 1;										//置接收标志位变量为1
			USART_ClearITPendingBit(USART1, USART_IT_RXNE);			//清除USART1的RXNE标志位
																	//读取数据寄存器会自动清除此标志位
																	//如果已经读取了数据寄存器，也可以不执行此代码
		}
	}
	```


### I2C通信

#### 简介

- 2C（Inter IC Bus）是由Philips公司开发的一种通用数据总线

- 两根通信线：SCL（Serial Clock）、SDA（Serial Data）

- 同步，半双工

- 带数据应答

- 支持总线挂载多设备（一主多从、多主多从）

#### 硬件电路

- 所有I2C设备的SCL连在一起，SDA连在一起

- 设备的SCL和SDA均要配置成开漏输出模式

- SCL和SDA各添加一个上拉电阻，阻值一般为4.7KΩ左右

![image-20231130125424963](.\assets\image-7.png)


#### I2C时序基本单元

1. 起始条件：SCL高电平期间，SDA从高电平切换到低电平
![image-20231130125606879](.\assets\image-8.png)

2. 终止条件：SCL高电平期间，SDA从低电平切换到高电平
	![image-20231130125721757](.\assets\image-9.png)
3. 发送一个字节：SCL低电平期间，主机将数据位依次放到SDA线上（高位先行），然后释放SCL，从机将在SCL高电平期间读取数据位，所以SCL高电平期间SDA不允许有数据变化，依次循环上述过程8次，即可发送一个字节
	![image-20231130125816242](.\assets\image-10.png)

4. 接收一个字节：SCL低电平期间，从机将数据位依次放到SDA线上（高位先行），然后释放SCL，主机将在SCL高电平期间读取数据位，所以SCL高电平期间SDA不允许有数据变化，依次循环上述过程8次，即可接收一个字节（主机在接收之前，需要释放SDA）
	![image-20231130130041260](.\assets\image-11.png)

5. 发送应答：主机在接收完一个字节之后，在下一个时钟发送一位数据，数据0表示应答，数据1表示非应答
	![image-20231130130125191](.\assets\image-12.png)

6. 接收应答：主机在发送完一个字节之后，在下一个时钟接收一位数据，判断从机是否应答，数据0表示应答，数据1表示非应答（主机在接收之前，需要释放SDA）
	![image-20231130130207643](.\assets\image-13.png)

#### I2C时序

1. 指定地址写：对于指定设备（Slave Address），在指定地址（Reg Address）下，写入指定数据（Data）
	![image-20231130130317631](.\assets\image-14.png)

2. 当前地址读：对于指定设备（Slave Address），在当前地址指针（读写都会自增）指示的地址下，读取从机数据（Data）
	![image-20231130130443990](.\assets\image-15.png)

3. 指定地址读：对于指定设备（Slave Address），在指定地址（Reg Address）下，读取从机数据（Data）
	![image-20231130130523515](.\assets\image-16.png)
#### 外设

##### 简介

STM32内部集成了硬件I2C收发电路，可以由硬件自动执行时钟生成、起始终止条件生成、应答位收发、数据收发等功能，减轻CPU的负担。支持多主机模型

##### 基本结构图

![image-20231201061231648](.\assets\image-17.png)

##### 主机发送流程图

![image-20231201061538911](.\assets\image-18.png)

##### 主机接收流程图

![image-20231201061558381](.\assets\image-19.png)

#### I2C协议软件模拟

1. 引脚配置层及GPIO初始化

	```c
	/*引脚配置层*/
	
	/**
	  * 函    数：I2C写SCL引脚电平
	  * 参    数：BitValue 协议层传入的当前需要写入SCL的电平，范围0~1
	  * 返 回 值：无
	  * 注意事项：此函数需要用户实现内容，当BitValue为0时，需要置SCL为低电平，当BitValue为1时，需要置SCL为高电平
	  */
	void MyI2C_W_SCL(uint8_t BitValue)
	{
		GPIO_WriteBit(GPIOB, GPIO_Pin_10, (BitAction)BitValue);		//根据BitValue，设置SCL引脚的电平
		Delay_us(10);												//延时10us，防止时序频率超过要求
	}
	
	/**
	  * 函    数：I2C写SDA引脚电平
	  * 参    数：BitValue 协议层传入的当前需要写入SDA的电平，范围0~0xFF
	  * 返 回 值：无
	  * 注意事项：此函数需要用户实现内容，当BitValue为0时，需要置SDA为低电平，当BitValue非0时，需要置SDA为高电平
	  */
	void MyI2C_W_SDA(uint8_t BitValue)
	{
		GPIO_WriteBit(GPIOB, GPIO_Pin_11, (BitAction)BitValue);		//根据BitValue，设置SDA引脚的电平，BitValue要实现非0即1的特性
		Delay_us(10);												//延时10us，防止时序频率超过要求
	}
	
	/**
	  * 函    数：I2C读SDA引脚电平
	  * 参    数：无
	  * 返 回 值：协议层需要得到的当前SDA的电平，范围0~1
	  * 注意事项：此函数需要用户实现内容，当前SDA为低电平时，返回0，当前SDA为高电平时，返回1
	  */
	uint8_t MyI2C_R_SDA(void)
	{
		uint8_t BitValue;
		BitValue = GPIO_ReadInputDataBit(GPIOB, GPIO_Pin_11);		//读取SDA电平
		Delay_us(10);												//延时10us，防止时序频率超过要求
		return BitValue;											//返回SDA电平
	}
	
	/**
	  * 函    数：I2C初始化
	  * 参    数：无
	  * 返 回 值：无
	  * 注意事项：此函数需要用户实现内容，实现SCL和SDA引脚的初始化
	  */
	void MyI2C_Init(void)
	{
		/*开启时钟*/
		RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOB, ENABLE);	//开启GPIOB的时钟
		
		/*GPIO初始化*/
		GPIO_InitTypeDef GPIO_InitStructure;
		GPIO_InitStructure.GPIO_Mode = GPIO_Mode_Out_OD;
		GPIO_InitStructure.GPIO_Pin = GPIO_Pin_10 | GPIO_Pin_11;
		GPIO_InitStructure.GPIO_Speed = GPIO_Speed_50MHz;
		GPIO_Init(GPIOB, &GPIO_InitStructure);					//将PB10和PB11引脚初始化为开漏输出
		
		/*设置默认电平*/
		GPIO_SetBits(GPIOB, GPIO_Pin_10 | GPIO_Pin_11);			//设置PB10和PB11引脚初始化后默认为高电平（释放总线状态）
	}
	```

2. 协议层，依次定义6个I2C时序基本单元

	```c
	/*协议层*/
	
	/**
	  * 函    数：I2C起始
	  * 参    数：无
	  * 返 回 值：无
	  */
	void MyI2C_Start(void)
	{
		MyI2C_W_SDA(1);							//释放SDA，确保SDA为高电平
		MyI2C_W_SCL(1);							//释放SCL，确保SCL为高电平
		MyI2C_W_SDA(0);							//在SCL高电平期间，拉低SDA，产生起始信号
		MyI2C_W_SCL(0);							//起始后把SCL也拉低，即为了占用总线，也为了方便总线时序的拼接
	}
	
	/**
	  * 函    数：I2C终止
	  * 参    数：无
	  * 返 回 值：无
	  */
	void MyI2C_Stop(void)
	{
		MyI2C_W_SDA(0);							//拉低SDA，确保SDA为低电平
		MyI2C_W_SCL(1);							//释放SCL，使SCL呈现高电平
		MyI2C_W_SDA(1);							//在SCL高电平期间，释放SDA，产生终止信号
	}
	
	/**
	  * 函    数：I2C发送一个字节
	  * 参    数：Byte 要发送的一个字节数据，范围：0x00~0xFF
	  * 返 回 值：无
	  */
	void MyI2C_SendByte(uint8_t Byte)
	{
		uint8_t i;
		for (i = 0; i < 8; i ++)				//循环8次，主机依次发送数据的每一位
		{
			MyI2C_W_SDA(Byte & (0x80 >> i));	//使用掩码的方式取出Byte的指定一位数据并写入到SDA线
			MyI2C_W_SCL(1);						//释放SCL，从机在SCL高电平期间读取SDA
			MyI2C_W_SCL(0);						//拉低SCL，主机开始发送下一位数据
		}
	}
	
	/**
	  * 函    数：I2C接收一个字节
	  * 参    数：无
	  * 返 回 值：接收到的一个字节数据，范围：0x00~0xFF
	  */
	uint8_t MyI2C_ReceiveByte(void)
	{
		uint8_t i, Byte = 0x00;					//定义接收的数据，并赋初值0x00，此处必须赋初值0x00，后面会用到
		MyI2C_W_SDA(1);							//接收前，主机先确保释放SDA，避免干扰从机的数据发送
		for (i = 0; i < 8; i ++)				//循环8次，主机依次接收数据的每一位
		{
			MyI2C_W_SCL(1);						//释放SCL，主机机在SCL高电平期间读取SDA
			if (MyI2C_R_SDA() == 1){Byte |= (0x80 >> i);}	//读取SDA数据，并存储到Byte变量
															//当SDA为1时，置变量指定位为1，当SDA为0时，不做处理，指定位为默认的初值0
			MyI2C_W_SCL(0);						//拉低SCL，从机在SCL低电平期间写入SDA
		}
		return Byte;							//返回接收到的一个字节数据
	}
	
	/**
	  * 函    数：I2C发送应答位
	  * 参    数：Byte 要发送的应答位，范围：0~1，0表示应答，1表示非应答
	  * 返 回 值：无
	  */
	void MyI2C_SendAck(uint8_t AckBit)
	{
		MyI2C_W_SDA(AckBit);					//主机把应答位数据放到SDA线
		MyI2C_W_SCL(1);							//释放SCL，从机在SCL高电平期间，读取应答位
		MyI2C_W_SCL(0);							//拉低SCL，开始下一个时序模块
	}
	
	/**
	  * 函    数：I2C接收应答位
	  * 参    数：无
	  * 返 回 值：接收到的应答位，范围：0~1，0表示应答，1表示非应答
	  */
	uint8_t MyI2C_ReceiveAck(void)
	{
		uint8_t AckBit;							//定义应答位变量
		MyI2C_W_SDA(1);							//接收前，主机先确保释放SDA，避免干扰从机的数据发送
		MyI2C_W_SCL(1);							//释放SCL，主机机在SCL高电平期间读取SDA
		AckBit = MyI2C_R_SDA();					//将应答位存储到变量里
		MyI2C_W_SCL(0);							//拉低SCL，开始下一个时序模块
		return AckBit;							//返回定义应答位变量
	}
	```

#### MPU6050

##### 简介

MPU6050是一个6轴姿态传感器，可以测量芯片自身X、Y、Z轴的加速度、角速度参数，通过数据融合，可进一步得到姿态角，常应用于平衡车、飞行器等需要检测自身姿态的场景

##### MPU6050的I2C软件模拟代码

1. 指定位置写

	```c
	/**
	  * 函    数：MPU6050写寄存器
	  * 参    数：RegAddress 寄存器地址，范围：参考MPU6050手册的寄存器描述
	  * 参    数：Data 要写入寄存器的数据，范围：0x00~0xFF
	  * 返 回 值：无
	  */
	void MPU6050_WriteReg(uint8_t RegAddress, uint8_t Data)
	{
		MyI2C_Start();						//I2C起始
		MyI2C_SendByte(MPU6050_ADDRESS);	//发送从机地址，读写位为0，表示即将写入
		MyI2C_ReceiveAck();					//接收应答
		MyI2C_SendByte(RegAddress);			//发送寄存器地址
		MyI2C_ReceiveAck();					//接收应答
		MyI2C_SendByte(Data);				//发送要写入寄存器的数据
		MyI2C_ReceiveAck();					//接收应答
		MyI2C_Stop();						//I2C终止
	}
	```

2. 指定位置读

	```c
	/**
	  * 函    数：MPU6050读寄存器
	  * 参    数：RegAddress 寄存器地址，范围：参考MPU6050手册的寄存器描述
	  * 返 回 值：读取寄存器的数据，范围：0x00~0xFF
	  */
	uint8_t MPU6050_ReadReg(uint8_t RegAddress)
	{
		uint8_t Data;
		
		MyI2C_Start();						//I2C起始
		MyI2C_SendByte(MPU6050_ADDRESS);	//发送从机地址，读写位为0，表示即将写入
		MyI2C_ReceiveAck();					//接收应答
		MyI2C_SendByte(RegAddress);			//发送寄存器地址
		MyI2C_ReceiveAck();					//接收应答
		
		MyI2C_Start();						//I2C重复起始
		MyI2C_SendByte(MPU6050_ADDRESS | 0x01);	//发送从机地址，读写位为1，表示即将读取
		MyI2C_ReceiveAck();					//接收应答
		Data = MyI2C_ReceiveByte();			//接收指定寄存器的数据
		MyI2C_SendAck(1);					//发送应答，给从机非应答，终止从机的数据输出
		MyI2C_Stop();						//I2C终止
		
		return Data;
	}
	```

3. MPU6050初始化

	```c
	/**
	  * 函    数：MPU6050初始化
	  * 参    数：无
	  * 返 回 值：无
	  */
	void MPU6050_Init(void)
	{
		MyI2C_Init();									//先初始化底层的I2C
		
		/*MPU6050寄存器初始化，需要对照MPU6050手册的寄存器描述配置，此处仅配置了部分重要的寄存器*/
		MPU6050_WriteReg(MPU6050_PWR_MGMT_1, 0x01);		//电源管理寄存器1，取消睡眠模式，选择时钟源为X轴陀螺仪
		MPU6050_WriteReg(MPU6050_PWR_MGMT_2, 0x00);		//电源管理寄存器2，保持默认值0，所有轴均不待机
		MPU6050_WriteReg(MPU6050_SMPLRT_DIV, 0x09);		//采样率分频寄存器，配置采样率
		MPU6050_WriteReg(MPU6050_CONFIG, 0x06);			//配置寄存器，配置DLPF
		MPU6050_WriteReg(MPU6050_GYRO_CONFIG, 0x18);	//陀螺仪配置寄存器，选择满量程为±2000°/s
		MPU6050_WriteReg(MPU6050_ACCEL_CONFIG, 0x18);	//加速度计配置寄存器，选择满量程为±16g
	}
	```

4. 获取6轴数据

	```c
	/**
	  * 函    数：MPU6050获取数据
	  * 参    数：AccX AccY AccZ 加速度计X、Y、Z轴的数据，使用输出参数的形式返回，范围：-32768~32767
	  * 参    数：GyroX GyroY GyroZ 陀螺仪X、Y、Z轴的数据，使用输出参数的形式返回，范围：-32768~32767
	  * 返 回 值：无
	  */
	void MPU6050_GetData(int16_t *AccX, int16_t *AccY, int16_t *AccZ, 
							int16_t *GyroX, int16_t *GyroY, int16_t *GyroZ)
	{
		uint8_t DataH, DataL;								//定义数据高8位和低8位的变量
		
		DataH = MPU6050_ReadReg(MPU6050_ACCEL_XOUT_H);		//读取加速度计X轴的高8位数据
		DataL = MPU6050_ReadReg(MPU6050_ACCEL_XOUT_L);		//读取加速度计X轴的低8位数据
		*AccX = (DataH << 8) | DataL;						//数据拼接，通过输出参数返回
		
		DataH = MPU6050_ReadReg(MPU6050_ACCEL_YOUT_H);		//读取加速度计Y轴的高8位数据
		DataL = MPU6050_ReadReg(MPU6050_ACCEL_YOUT_L);		//读取加速度计Y轴的低8位数据
		*AccY = (DataH << 8) | DataL;						//数据拼接，通过输出参数返回
		
		DataH = MPU6050_ReadReg(MPU6050_ACCEL_ZOUT_H);		//读取加速度计Z轴的高8位数据
		DataL = MPU6050_ReadReg(MPU6050_ACCEL_ZOUT_L);		//读取加速度计Z轴的低8位数据
		*AccZ = (DataH << 8) | DataL;						//数据拼接，通过输出参数返回
		
		DataH = MPU6050_ReadReg(MPU6050_GYRO_XOUT_H);		//读取陀螺仪X轴的高8位数据
		DataL = MPU6050_ReadReg(MPU6050_GYRO_XOUT_L);		//读取陀螺仪X轴的低8位数据
		*GyroX = (DataH << 8) | DataL;						//数据拼接，通过输出参数返回
		
		DataH = MPU6050_ReadReg(MPU6050_GYRO_YOUT_H);		//读取陀螺仪Y轴的高8位数据
		DataL = MPU6050_ReadReg(MPU6050_GYRO_YOUT_L);		//读取陀螺仪Y轴的低8位数据
		*GyroY = (DataH << 8) | DataL;						//数据拼接，通过输出参数返回
		
		DataH = MPU6050_ReadReg(MPU6050_GYRO_ZOUT_H);		//读取陀螺仪Z轴的高8位数据
		DataL = MPU6050_ReadReg(MPU6050_GYRO_ZOUT_L);		//读取陀螺仪Z轴的低8位数据
		*GyroZ = (DataH << 8) | DataL;						//数据拼接，通过输出参数返回
	}
	```

5. MPU6050寄存器配置表

	```c
	#ifndef __MPU6050_REG_H
	#define __MPU6050_REG_H
	
	#define	MPU6050_SMPLRT_DIV		0x19
	#define	MPU6050_CONFIG			0x1A
	#define	MPU6050_GYRO_CONFIG		0x1B
	#define	MPU6050_ACCEL_CONFIG	0x1C
	
	#define	MPU6050_ACCEL_XOUT_H	0x3B
	#define	MPU6050_ACCEL_XOUT_L	0x3C
	#define	MPU6050_ACCEL_YOUT_H	0x3D
	#define	MPU6050_ACCEL_YOUT_L	0x3E
	#define	MPU6050_ACCEL_ZOUT_H	0x3F
	#define	MPU6050_ACCEL_ZOUT_L	0x40
	#define	MPU6050_TEMP_OUT_H		0x41
	#define	MPU6050_TEMP_OUT_L		0x42
	#define	MPU6050_GYRO_XOUT_H		0x43
	#define	MPU6050_GYRO_XOUT_L		0x44
	#define	MPU6050_GYRO_YOUT_H		0x45
	#define	MPU6050_GYRO_YOUT_L		0x46
	#define	MPU6050_GYRO_ZOUT_H		0x47
	#define	MPU6050_GYRO_ZOUT_L		0x48
	
	#define	MPU6050_PWR_MGMT_1		0x6B
	#define	MPU6050_PWR_MGMT_2		0x6C
	#define	MPU6050_WHO_AM_I		0x75
	
	#endif
	```


##### MPU6050的I2C硬件模拟代码

代码大体和软件模拟差不多，不过涉及到I2C的部分需要修改，修改如下

1. 指定位置写

	```c
	/**
	  * 函    数：MPU6050写寄存器
	  * 参    数：RegAddress 寄存器地址，范围：参考MPU6050手册的寄存器描述
	  * 参    数：Data 要写入寄存器的数据，范围：0x00~0xFF
	  * 返 回 值：无
	  */
	void MPU6050_WriteReg(uint8_t RegAddress, uint8_t Data)
	{
		I2C_GenerateSTART(I2C2, ENABLE);										//硬件I2C生成起始条件
		MPU6050_WaitEvent(I2C2, I2C_EVENT_MASTER_MODE_SELECT);					//等待EV5
		
		I2C_Send7bitAddress(I2C2, MPU6050_ADDRESS, I2C_Direction_Transmitter);	//硬件I2C发送从机地址，方向为发送
		MPU6050_WaitEvent(I2C2, I2C_EVENT_MASTER_TRANSMITTER_MODE_SELECTED);	//等待EV6
		
		I2C_SendData(I2C2, RegAddress);											//硬件I2C发送寄存器地址
		MPU6050_WaitEvent(I2C2, I2C_EVENT_MASTER_BYTE_TRANSMITTING);			//等待EV8
		
		I2C_SendData(I2C2, Data);												//硬件I2C发送数据
		MPU6050_WaitEvent(I2C2, I2C_EVENT_MASTER_BYTE_TRANSMITTED);				//等待EV8_2
		
		I2C_GenerateSTOP(I2C2, ENABLE);											//硬件I2C生成终止条件
	}
	```

2. 指定位置读

	```c
	/**
	  * 函    数：MPU6050读寄存器
	  * 参    数：RegAddress 寄存器地址，范围：参考MPU6050手册的寄存器描述
	  * 返 回 值：读取寄存器的数据，范围：0x00~0xFF
	  */
	uint8_t MPU6050_ReadReg(uint8_t RegAddress)
	{
		uint8_t Data;
		
		I2C_GenerateSTART(I2C2, ENABLE);										//硬件I2C生成起始条件
		MPU6050_WaitEvent(I2C2, I2C_EVENT_MASTER_MODE_SELECT);					//等待EV5
		
		I2C_Send7bitAddress(I2C2, MPU6050_ADDRESS, I2C_Direction_Transmitter);	//硬件I2C发送从机地址，方向为发送
		MPU6050_WaitEvent(I2C2, I2C_EVENT_MASTER_TRANSMITTER_MODE_SELECTED);	//等待EV6
		
		I2C_SendData(I2C2, RegAddress);											//硬件I2C发送寄存器地址
		MPU6050_WaitEvent(I2C2, I2C_EVENT_MASTER_BYTE_TRANSMITTED);				//等待EV8_2
		
		I2C_GenerateSTART(I2C2, ENABLE);										//硬件I2C生成重复起始条件
		MPU6050_WaitEvent(I2C2, I2C_EVENT_MASTER_MODE_SELECT);					//等待EV5
		
		I2C_Send7bitAddress(I2C2, MPU6050_ADDRESS, I2C_Direction_Receiver);		//硬件I2C发送从机地址，方向为接收
		MPU6050_WaitEvent(I2C2, I2C_EVENT_MASTER_RECEIVER_MODE_SELECTED);		//等待EV6
		
		I2C_AcknowledgeConfig(I2C2, DISABLE);									//在接收最后一个字节之前提前将应答失能
		I2C_GenerateSTOP(I2C2, ENABLE);											//在接收最后一个字节之前提前申请停止条件
		
		MPU6050_WaitEvent(I2C2, I2C_EVENT_MASTER_BYTE_RECEIVED);				//等待EV7
		Data = I2C_ReceiveData(I2C2);											//接收数据寄存器
		
		I2C_AcknowledgeConfig(I2C2, ENABLE);									//将应答恢复为使能，为了不影响后续可能产生的读取多字节操作
		
		return Data;
	}
	```

3. MPU6050初始化

	```c
	/**
	  * 函    数：MPU6050初始化
	  * 参    数：无
	  * 返 回 值：无
	  */
	void MPU6050_Init(void)
	{
		/*开启时钟*/
		RCC_APB1PeriphClockCmd(RCC_APB1Periph_I2C2, ENABLE);		//开启I2C2的时钟
		RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOB, ENABLE);		//开启GPIOB的时钟
		
		/*GPIO初始化*/
		GPIO_InitTypeDef GPIO_InitStructure;
		GPIO_InitStructure.GPIO_Mode = GPIO_Mode_AF_OD;
		GPIO_InitStructure.GPIO_Pin = GPIO_Pin_10 | GPIO_Pin_11;
		GPIO_InitStructure.GPIO_Speed = GPIO_Speed_50MHz;
		GPIO_Init(GPIOB, &GPIO_InitStructure);					//将PB10和PB11引脚初始化为复用开漏输出
		
		/*I2C初始化*/
		I2C_InitTypeDef I2C_InitStructure;						//定义结构体变量
		I2C_InitStructure.I2C_Mode = I2C_Mode_I2C;				//模式，选择为I2C模式
		I2C_InitStructure.I2C_ClockSpeed = 50000;				//时钟速度，选择为50KHz
		I2C_InitStructure.I2C_DutyCycle = I2C_DutyCycle_2;		//时钟占空比，选择Tlow/Thigh = 2
		I2C_InitStructure.I2C_Ack = I2C_Ack_Enable;				//应答，选择使能
		I2C_InitStructure.I2C_AcknowledgedAddress = I2C_AcknowledgedAddress_7bit;	//应答地址，选择7位，从机模式下才有效
		I2C_InitStructure.I2C_OwnAddress1 = 0x00;				//自身地址，从机模式下才有效
		I2C_Init(I2C2, &I2C_InitStructure);						//将结构体变量交给I2C_Init，配置I2C2
		
		/*I2C使能*/
		I2C_Cmd(I2C2, ENABLE);									//使能I2C2，开始运行
		
		/*MPU6050寄存器初始化，需要对照MPU6050手册的寄存器描述配置，此处仅配置了部分重要的寄存器*/
		MPU6050_WriteReg(MPU6050_PWR_MGMT_1, 0x01);				//电源管理寄存器1，取消睡眠模式，选择时钟源为X轴陀螺仪
		MPU6050_WriteReg(MPU6050_PWR_MGMT_2, 0x00);				//电源管理寄存器2，保持默认值0，所有轴均不待机
		MPU6050_WriteReg(MPU6050_SMPLRT_DIV, 0x09);				//采样率分频寄存器，配置采样率
		MPU6050_WriteReg(MPU6050_CONFIG, 0x06);					//配置寄存器，配置DLPF
		MPU6050_WriteReg(MPU6050_GYRO_CONFIG, 0x18);			//陀螺仪配置寄存器，选择满量程为±2000°/s
		MPU6050_WriteReg(MPU6050_ACCEL_CONFIG, 0x18);			//加速度计配置寄存器，选择满量程为±16g
	}
	```

4. 等待事件发生，加入超时机制

	```c
	/**
	  * 函    数：MPU6050等待事件
	  * 参    数：同I2C_CheckEvent
	  * 返 回 值：无
	  */
	void MPU6050_WaitEvent(I2C_TypeDef* I2Cx, uint32_t I2C_EVENT)
	{
		uint32_t Timeout;
		Timeout = 10000;									//给定超时计数时间
		while (I2C_CheckEvent(I2Cx, I2C_EVENT) != SUCCESS)	//循环等待指定事件
		{
			Timeout --;										//等待时，计数值自减
			if (Timeout == 0)								//自减到0后，等待超时
			{
				/*超时的错误处理代码，可以添加到此处*/
				break;										//跳出等待，不等了
			}
		}
	}
	```


### SPI通信

#### 简介

- SPI（Serial Peripheral Interface）是由Motorola公司开发的一种通用数据总线

- 四根通信线：SCK（Serial Clock）、MOSI（Master Output Slave Input）、MISO（Master Input Slave Output）、SS（Slave Select）

- 同步，全双工

- 支持总线挂载多设备（一主多从）

#### 硬件电路

- 所有SPI设备的SCK、MOSI、MISO分别连在一起

- 主机另外引出多条SS控制线，分别接到各从机的SS引脚

- 输出引脚配置为推挽输出，输入引脚配置为浮空或上拉输入

![image-20231207084936113](assets\image-20.png) 

#### 移位示意图

SPI通信通过交换移位寄存器里面的数据达到通信的目的

![image-20231207085125449](assets\image-21.png)

#### 时序基本单元

1. 起始条件：SS从高电平切换到低电平
	![image-20231207085507539](.\assets\image-22.png)

2. 终止条件：SS从低电平切换到高电平
	![image-20231207085521910](.\assets\image-23.png)

3. 交换一个字节（模式0）：

	CPOL=0：空闲状态时，SCK为低电平

	CPHA=0：SCK第一个边沿移入数据，第二个边沿移出数据
	![image-20231207085543653](.\assets\image-24.png)

4. 交换一个字节（模式1）

	CPOL=0：空闲状态时，SCK为低电平

	CPHA=1：SCK第一个边沿移出数据，第二个边沿移入数据
	![image-20231207085625063](.\assets\image-25.png)

5. 交换一个字节（模式2）

	CPOL=1：空闲状态时，SCK为高电平

	CPHA=0：SCK第一个边沿移入数据，第二个边沿移出数据
	![image-20231207090001453](.\assets\image-26.png)

6. 交换一个字节（模式3）

	CPOL=1：空闲状态时，SCK为高电平

	CPHA=1：SCK第一个边沿移出数据，第二个边沿移入数据
	![image-20231207090001453](.\assets\image-27.png)

#### SPI时序（模式0）

1. 发送指令。向SS指定的设备，发送指令（0x06）
	![image-20231207090459399](assets\image-28.png)
2. 指定地址写。向SS指定的设备，发送写指令（0x02），随后在指定地址（Address[23:0]）下，写入指定数据（Data）
	![image-20231207090459399](assets\image-29.png)
3. 指定地址读。向SS指定的设备，发送读指令（0x03），随后在指定地址（Address[23:0]）下，读取从机数据（Data）
	![image-20231207090602391](assets\image-30.png)

#### 外设

##### 简介

- STM32内部集成了硬件SPI收发电路，可以由硬件自动执行时钟生成、数据收发等功能，减轻CPU的负担

- 可配置8位/16位数据帧、高位先行/低位先行

- 时钟频率： fPCLK / (2, 4, 8, 16, 32, 64, 128, 256)

##### SPI基本结构

![image-20231207111231011](assets\image-31.png)

##### 非连续传输流程图

![image-20231207111329056](assets\image-32.png) 

#### SPI协议软件模拟

1. 引脚配置层，配置GPIO口及SS，SLK，MOSI和MISO读写操作

	```c
	/*引脚配置层*/
	
	/**
	  * 函    数：SPI写SS引脚电平
	  * 参    数：BitValue 协议层传入的当前需要写入SS的电平，范围0~1
	  * 返 回 值：无
	  * 注意事项：此函数需要用户实现内容，当BitValue为0时，需要置SS为低电平，当BitValue为1时，需要置SS为高电平
	  */
	void MySPI_W_SS(uint8_t BitValue)
	{
		GPIO_WriteBit(GPIOA, GPIO_Pin_4, (BitAction)BitValue);		//根据BitValue，设置SS引脚的电平
	}
	
	/**
	  * 函    数：SPI写SCK引脚电平
	  * 参    数：BitValue 协议层传入的当前需要写入SCK的电平，范围0~1
	  * 返 回 值：无
	  * 注意事项：此函数需要用户实现内容，当BitValue为0时，需要置SCK为低电平，当BitValue为1时，需要置SCK为高电平
	  */
	void MySPI_W_SCK(uint8_t BitValue)
	{
		GPIO_WriteBit(GPIOA, GPIO_Pin_5, (BitAction)BitValue);		//根据BitValue，设置SCK引脚的电平
	}
	
	/**
	  * 函    数：SPI写MOSI引脚电平
	  * 参    数：BitValue 协议层传入的当前需要写入MOSI的电平，范围0~0xFF
	  * 返 回 值：无
	  * 注意事项：此函数需要用户实现内容，当BitValue为0时，需要置MOSI为低电平，当BitValue非0时，需要置MOSI为高电平
	  */
	void MySPI_W_MOSI(uint8_t BitValue)
	{
		GPIO_WriteBit(GPIOA, GPIO_Pin_7, (BitAction)BitValue);		//根据BitValue，设置MOSI引脚的电平，BitValue要实现非0即1的特性
	}
	
	/**
	  * 函    数：I2C读MISO引脚电平
	  * 参    数：无
	  * 返 回 值：协议层需要得到的当前MISO的电平，范围0~1
	  * 注意事项：此函数需要用户实现内容，当前MISO为低电平时，返回0，当前MISO为高电平时，返回1
	  */
	uint8_t MySPI_R_MISO(void)
	{
		return GPIO_ReadInputDataBit(GPIOA, GPIO_Pin_6);			//读取MISO电平并返回
	}
	
	/**
	  * 函    数：SPI初始化
	  * 参    数：无
	  * 返 回 值：无
	  * 注意事项：此函数需要用户实现内容，实现SS、SCK、MOSI和MISO引脚的初始化
	  */
	void MySPI_Init(void)
	{
		/*开启时钟*/
		RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOA, ENABLE);	//开启GPIOA的时钟
		
		/*GPIO初始化*/
		GPIO_InitTypeDef GPIO_InitStructure;
		GPIO_InitStructure.GPIO_Mode = GPIO_Mode_Out_PP;
		GPIO_InitStructure.GPIO_Pin = GPIO_Pin_4 | GPIO_Pin_5 | GPIO_Pin_7;
		GPIO_InitStructure.GPIO_Speed = GPIO_Speed_50MHz;
		GPIO_Init(GPIOA, &GPIO_InitStructure);					//将PA4、PA5和PA7引脚初始化为推挽输出
		
		GPIO_InitStructure.GPIO_Mode = GPIO_Mode_IPU;
		GPIO_InitStructure.GPIO_Pin = GPIO_Pin_6;
		GPIO_InitStructure.GPIO_Speed = GPIO_Speed_50MHz;
		GPIO_Init(GPIOA, &GPIO_InitStructure);					//将PA6引脚初始化为上拉输入
		
		/*设置默认电平*/
		MySPI_W_SS(1);											//SS默认高电平
		MySPI_W_SCK(0);											//SCK默认低电平
	}
	```

2. 协议层，配置时序基本单元，起始条件，结束条件和交换一个字节

	```c
	/*协议层*/
	
	/**
	  * 函    数：SPI起始
	  * 参    数：无
	  * 返 回 值：无
	  */
	void MySPI_Start(void)
	{
		MySPI_W_SS(0);				//拉低SS，开始时序
	}
	
	/**
	  * 函    数：SPI终止
	  * 参    数：无
	  * 返 回 值：无
	  */
	void MySPI_Stop(void)
	{
		MySPI_W_SS(1);				//拉高SS，终止时序
	}
	
	/**
	  * 函    数：SPI交换传输一个字节，使用SPI模式0
	  * 参    数：ByteSend 要发送的一个字节
	  * 返 回 值：接收的一个字节
	  */
	uint8_t MySPI_SwapByte(uint8_t ByteSend)
	{
		uint8_t i, ByteReceive = 0x00;					//定义接收的数据，并赋初值0x00，此处必须赋初值0x00，后面会用到
		
		for (i = 0; i < 8; i ++)						//循环8次，依次交换每一位数据
		{
			MySPI_W_MOSI(ByteSend & (0x80 >> i));		//使用掩码的方式取出ByteSend的指定一位数据并写入到MOSI线
			MySPI_W_SCK(1);								//拉高SCK，上升沿移出数据
			if (MySPI_R_MISO() == 1){ByteReceive |= (0x80 >> i);}	//读取MISO数据，并存储到Byte变量
																	//当MISO为1时，置变量指定位为1，当MISO为0时，不做处理，指定位为默认的初值0
			MySPI_W_SCK(0);								//拉低SCK，下降沿移入数据
		}
		
		return ByteReceive;								//返回接收到的一个字节数据
	}
	```

#### SPI协议外设模拟

1. SPI写SS引脚电平，SS仍由软件模拟

	```c
	/**
	  * 函    数：SPI写SS引脚电平，SS仍由软件模拟
	  * 参    数：BitValue 协议层传入的当前需要写入SS的电平，范围0~1
	  * 返 回 值：无
	  * 注意事项：此函数需要用户实现内容，当BitValue为0时，需要置SS为低电平，当BitValue为1时，需要置SS为高电平
	  */
	void MySPI_W_SS(uint8_t BitValue)
	{
		GPIO_WriteBit(GPIOA, GPIO_Pin_4, (BitAction)BitValue);		//根据BitValue，设置SS引脚的电平
	}
	```

2. SPI初始化

	```c
	/**
	  * 函    数：SPI初始化
	  * 参    数：无
	  * 返 回 值：无
	  */
	void MySPI_Init(void)
	{
		/*开启时钟*/
		RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOA, ENABLE);	//开启GPIOA的时钟
		RCC_APB2PeriphClockCmd(RCC_APB2Periph_SPI1, ENABLE);	//开启SPI1的时钟
		
		/*GPIO初始化*/
		GPIO_InitTypeDef GPIO_InitStructure;
		GPIO_InitStructure.GPIO_Mode = GPIO_Mode_Out_PP;
		GPIO_InitStructure.GPIO_Pin = GPIO_Pin_4;
		GPIO_InitStructure.GPIO_Speed = GPIO_Speed_50MHz;
		GPIO_Init(GPIOA, &GPIO_InitStructure);					//将PA4引脚初始化为推挽输出
		
		GPIO_InitStructure.GPIO_Mode = GPIO_Mode_AF_PP;
		GPIO_InitStructure.GPIO_Pin = GPIO_Pin_5 | GPIO_Pin_7;
		GPIO_InitStructure.GPIO_Speed = GPIO_Speed_50MHz;
		GPIO_Init(GPIOA, &GPIO_InitStructure);					//将PA5和PA7引脚初始化为复用推挽输出
		
		GPIO_InitStructure.GPIO_Mode = GPIO_Mode_IPU;
		GPIO_InitStructure.GPIO_Pin = GPIO_Pin_6;
		GPIO_InitStructure.GPIO_Speed = GPIO_Speed_50MHz;
		GPIO_Init(GPIOA, &GPIO_InitStructure);					//将PA6引脚初始化为上拉输入
		
		/*SPI初始化*/
		SPI_InitTypeDef SPI_InitStructure;						//定义结构体变量
		SPI_InitStructure.SPI_Mode = SPI_Mode_Master;			//模式，选择为SPI主模式
		SPI_InitStructure.SPI_Direction = SPI_Direction_2Lines_FullDuplex;	//方向，选择2线全双工
		SPI_InitStructure.SPI_DataSize = SPI_DataSize_8b;		//数据宽度，选择为8位
		SPI_InitStructure.SPI_FirstBit = SPI_FirstBit_MSB;		//先行位，选择高位先行
		SPI_InitStructure.SPI_BaudRatePrescaler = SPI_BaudRatePrescaler_128;	//波特率分频，选择128分频
		SPI_InitStructure.SPI_CPOL = SPI_CPOL_Low;				//SPI极性，选择低极性
		SPI_InitStructure.SPI_CPHA = SPI_CPHA_1Edge;			//SPI相位，选择第一个时钟边沿采样，极性和相位决定选择SPI模式0
		SPI_InitStructure.SPI_NSS = SPI_NSS_Soft;				//NSS，选择由软件控制
		SPI_InitStructure.SPI_CRCPolynomial = 7;				//CRC多项式，暂时用不到，给默认值7
		SPI_Init(SPI1, &SPI_InitStructure);						//将结构体变量交给SPI_Init，配置SPI1
		
		/*SPI使能*/
		SPI_Cmd(SPI1, ENABLE);									//使能SPI1，开始运行
		
		/*设置默认电平*/
		MySPI_W_SS(1);											//SS默认高电平
	}
	```

3. SPI起始条件和结束条件

	```c
	/**
	  * 函    数：SPI起始
	  * 参    数：无
	  * 返 回 值：无
	  */
	void MySPI_Start(void)
	{
		MySPI_W_SS(0);				//拉低SS，开始时序
	}
	
	/**
	  * 函    数：SPI终止
	  * 参    数：无
	  * 返 回 值：无
	  */
	void MySPI_Stop(void)
	{
		MySPI_W_SS(1);				//拉高SS，终止时序
	}
	```

4. SPI交换传输一个字节，使用SPI模式0

	```c
	/**
	  * 函    数：SPI交换传输一个字节，使用SPI模式0
	  * 参    数：ByteSend 要发送的一个字节
	  * 返 回 值：接收的一个字节
	  */
	uint8_t MySPI_SwapByte(uint8_t ByteSend)
	{
		while (SPI_I2S_GetFlagStatus(SPI1, SPI_I2S_FLAG_TXE) != SET);	//等待发送数据寄存器空
		
		SPI_I2S_SendData(SPI1, ByteSend);								//写入数据到发送数据寄存器，开始产生时序
		
		while (SPI_I2S_GetFlagStatus(SPI1, SPI_I2S_FLAG_RXNE) != SET);	//等待接收数据寄存器非空
		
		return SPI_I2S_ReceiveData(SPI1);								//读取接收到的数据并返回
	}
	
	```

#### W25Q64

##### 简介

- W25Qxx系列是一种低成本、小型化、使用简单的非易失性存储器，常应用于数据存储、字库存储、固件程序存储等场景

- 存储介质：Nor Flash（闪存）

- 时钟频率：80MHz / 160MHz (Dual SPI) / 320MHz (Quad SPI)

##### 操作注意事项

1. 写入操作时：

	- 写入操作前，必须先进行写使能

	- 每个数据位只能由1改写为0，不能由0改写为1
	- 写入数据前必须先擦除，擦除后，所有数据位变为1
	- 擦除必须按最小擦除单元进行
	- 连续写入多字节时，最多写入一页的数据，超过页尾位置的数据，会回到页首覆盖写入
	- 写入操作结束后，芯片进入忙状态，不响应新的读写操作

2. 读取操作时：
	- 直接调用读取时序，无需使能，无需额外操作，没有页的限制，读取操作结束后不会进入忙状态，但不能在忙状态时读取

##### SPI读写W25Q64

1. W25Q64初始化

	```c
	/**
	  * 函    数：W25Q64初始化
	  * 参    数：无
	  * 返 回 值：无
	  */
	void W25Q64_Init(void)
	{
		MySPI_Init();					//先初始化底层的SPI
	}
	```

2. MPU6050读取ID号

	```c
	/**
	  * 函    数：MPU6050读取ID号
	  * 参    数：MID 工厂ID，使用输出参数的形式返回
	  * 参    数：DID 设备ID，使用输出参数的形式返回
	  * 返 回 值：无
	  */
	void W25Q64_ReadID(uint8_t *MID, uint16_t *DID)
	{
		MySPI_Start();								//SPI起始
		MySPI_SwapByte(W25Q64_JEDEC_ID);			//交换发送读取ID的指令
		*MID = MySPI_SwapByte(W25Q64_DUMMY_BYTE);	//交换接收MID，通过输出参数返回
		*DID = MySPI_SwapByte(W25Q64_DUMMY_BYTE);	//交换接收DID高8位
		*DID <<= 8;									//高8位移到高位
		*DID |= MySPI_SwapByte(W25Q64_DUMMY_BYTE);	//或上交换接收DID的低8位，通过输出参数返回
		MySPI_Stop();								//SPI终止
	}
	```

3. W25Q64写使能

	```c
	/**
	  * 函    数：W25Q64写使能
	  * 参    数：无
	  * 返 回 值：无
	  */
	void W25Q64_WriteEnable(void)
	{
		MySPI_Start();								//SPI起始
		MySPI_SwapByte(W25Q64_WRITE_ENABLE);		//交换发送写使能的指令
		MySPI_Stop();								//SPI终止
	}

4. W25Q64等待忙

	```c
	/**
	  * 函    数：W25Q64等待忙
	  * 参    数：无
	  * 返 回 值：无
	  */
	void W25Q64_WaitBusy(void)
	{
		uint32_t Timeout;
		MySPI_Start();								//SPI起始
		MySPI_SwapByte(W25Q64_READ_STATUS_REGISTER_1);				//交换发送读状态寄存器1的指令
		Timeout = 100000;							//给定超时计数时间
		while ((MySPI_SwapByte(W25Q64_DUMMY_BYTE) & 0x01) == 0x01)	//循环等待忙标志位
		{
			Timeout --;								//等待时，计数值自减
			if (Timeout == 0)						//自减到0后，等待超时
			{
				/*超时的错误处理代码，可以添加到此处*/
				break;								//跳出等待，不等了
			}
		}
		MySPI_Stop();								//SPI终止
	}
	```

5. W25Q64页编程

	```c
	/**
	  * 函    数：W25Q64页编程
	  * 参    数：Address 页编程的起始地址，范围：0x000000~0x7FFFFF
	  * 参    数：DataArray	用于写入数据的数组
	  * 参    数：Count 要写入数据的数量，范围：0~256
	  * 返 回 值：无
	  * 注意事项：写入的地址范围不能跨页
	  */
	void W25Q64_PageProgram(uint32_t Address, uint8_t *DataArray, uint16_t Count)
	{
		uint16_t i;
		
		W25Q64_WriteEnable();						//写使能
		
		MySPI_Start();								//SPI起始
		MySPI_SwapByte(W25Q64_PAGE_PROGRAM);		//交换发送页编程的指令
		MySPI_SwapByte(Address >> 16);				//交换发送地址23~16位
		MySPI_SwapByte(Address >> 8);				//交换发送地址15~8位
		MySPI_SwapByte(Address);					//交换发送地址7~0位
		for (i = 0; i < Count; i ++)				//循环Count次
		{
			MySPI_SwapByte(DataArray[i]);			//依次在起始地址后写入数据
		}
		MySPI_Stop();								//SPI终止
		
		W25Q64_WaitBusy();							//等待忙
	}

6. W25Q64扇区擦除（4KB）

	```c
	/**
	  * 函    数：W25Q64扇区擦除（4KB）
	  * 参    数：Address 指定扇区的地址，范围：0x000000~0x7FFFFF
	  * 返 回 值：无
	  */
	void W25Q64_SectorErase(uint32_t Address)
	{
		W25Q64_WriteEnable();						//写使能
		
		MySPI_Start();								//SPI起始
		MySPI_SwapByte(W25Q64_SECTOR_ERASE_4KB);	//交换发送扇区擦除的指令
		MySPI_SwapByte(Address >> 16);				//交换发送地址23~16位
		MySPI_SwapByte(Address >> 8);				//交换发送地址15~8位
		MySPI_SwapByte(Address);					//交换发送地址7~0位
		MySPI_Stop();								//SPI终止
		
		W25Q64_WaitBusy();							//等待忙
	}
	```

7. W25Q64读取数据

	```c
	/**
	  * 函    数：W25Q64读取数据
	  * 参    数：Address 读取数据的起始地址，范围：0x000000~0x7FFFFF
	  * 参    数：DataArray 用于接收读取数据的数组，通过输出参数返回
	  * 参    数：Count 要读取数据的数量，范围：0~0x800000
	  * 返 回 值：无
	  */
	void W25Q64_ReadData(uint32_t Address, uint8_t *DataArray, uint32_t Count)
	{
		uint32_t i;
		MySPI_Start();								//SPI起始
		MySPI_SwapByte(W25Q64_READ_DATA);			//交换发送读取数据的指令
		MySPI_SwapByte(Address >> 16);				//交换发送地址23~16位
		MySPI_SwapByte(Address >> 8);				//交换发送地址15~8位
		MySPI_SwapByte(Address);					//交换发送地址7~0位
		for (i = 0; i < Count; i ++)				//循环Count次
		{
			DataArray[i] = MySPI_SwapByte(W25Q64_DUMMY_BYTE);	//依次在起始地址后读取数据
		}
		MySPI_Stop();								//SPI终止
	}
	

### BKP备份寄存器&RTC实时时钟

#### Unix时间戳

- Unix 时间戳（Unix Timestamp）定义为从UTC/GMT的1970年1月1日0时0分0秒开始所经过的秒数，不考虑闰秒

- 时间戳存储在一个秒计数器中，秒计数器为32位/64位的整型变量

- 世界上所有时区的秒计数器相同，不同时区通过添加偏移来得到当地时间
- GMT（Greenwich Mean Time）格林尼治标准时间是一种以地球自转为基础的时间计量系统。它将地球自转一周的时间间隔等分为24小时，以此确定计时标准
- UTC（Universal Time Coordinated）协调世界时是一种以原子钟为基础的时间计量系统。它规定铯133原子基态的两个超精细能级间在零磁场下跃迁辐射9,192,631,770周所持续的时间为1秒。当原子钟计时一天的时间与地球自转一周的时间相差超过0.9秒时，UTC会执行闰秒来保证其计时与地球自转的协调一致

#### BKP简介

- BKP可用于存储用户应用程序数据。当VDD（2.0~3.6V）电源被切断，他们仍然由VBAT（1.8~3.6V）维持供电。当系统在待机模式下被唤醒，或系统复位或电源复位时，他们也不会被复位

- TAMPER引脚产生的侵入事件将所有备份寄存器内容清除

- RTC引脚输出RTC校准时钟、RTC闹钟脉冲或者秒脉冲

- 存储RTC时钟校准寄存器

#### BKP基本结构

![image-20231208122610288](assets\image-33.png)

#### RTC简介

- RTC是一个独立的定时器，可为系统提供时钟和日历的功能

- RTC和时钟配置系统处于后备区域，系统复位时数据不清零，VDD（2.0~3.6V）断电后可借助VBAT（1.8~3.6V）供电继续走时

- 32位的可编程计数器，可对应Unix时间戳的秒计数器

- 20位的可编程预分频器，可适配不同频率的输入时钟

- 可选择三种RTC时钟源：
	- HSE时钟除以128（通常为8MHz/128）
	-  LSE振荡器时钟（通常为32.768KHz）
	- LSI振荡器时钟（40KHz）

#### RTC基本结构

![image-20231208122739498](assets\image-34.png)

#### RTC操作注意事项

- 执行以下操作将使能对BKP和RTC的访问：
	- 设置RCC_APB1ENR的PWREN和BKPEN，使能PWR和BKP时钟
	- 设置PWR_CR的DBP，使能对BKP和RTC的访问

- 若在读取RTC寄存器时，RTC的APB1接口曾经处于禁止状态，则软件首先必须等待RTC_CRL寄存器中的RSF位（寄存器同步标志）被硬件置1

- 必须设置RTC_CRL寄存器中的CNF位，使RTC进入配置模式后，才能写入RTC_PRL、RTC_CNT、RTC_ALR寄存器

- 对RTC任何寄存器的写操作，都必须在前一次写操作结束后进行。可以通过查询RTC_CR寄存器中的RTOFF状态位，判断RTC寄存器是否处于更新中。仅当RTOFF状态位是1时，才可以写入RTC寄存器

#### 实时时钟

1. RTC初始化

	```c
	/**
	  * 函    数：RTC初始化
	  * 参    数：无
	  * 返 回 值：无
	  */
	void MyRTC_Init(void)
	{
		/*开启时钟*/
		RCC_APB1PeriphClockCmd(RCC_APB1Periph_PWR, ENABLE);		//开启PWR的时钟
		RCC_APB1PeriphClockCmd(RCC_APB1Periph_BKP, ENABLE);		//开启BKP的时钟
		
		/*备份寄存器访问使能*/
		PWR_BackupAccessCmd(ENABLE);							//使用PWR开启对备份寄存器的访问
		
		if (BKP_ReadBackupRegister(BKP_DR1) != 0xA5A5)			//通过写入备份寄存器的标志位，判断RTC是否是第一次配置
																//if成立则执行第一次的RTC配置
		{
			RCC_LSEConfig(RCC_LSE_ON);							//开启LSE时钟
			while (RCC_GetFlagStatus(RCC_FLAG_LSERDY) != SET);	//等待LSE准备就绪
			
			RCC_RTCCLKConfig(RCC_RTCCLKSource_LSE);				//选择RTCCLK来源为LSE
			RCC_RTCCLKCmd(ENABLE);								//RTCCLK使能
			
			RTC_WaitForSynchro();								//等待同步
			RTC_WaitForLastTask();								//等待上一次操作完成
			
			RTC_SetPrescaler(32768 - 1);						//设置RTC预分频器，预分频后的计数频率为1Hz
			RTC_WaitForLastTask();								//等待上一次操作完成
			
			MyRTC_SetTime();									//设置时间，调用此函数，全局数组里时间值刷新到RTC硬件电路
			
			BKP_WriteBackupRegister(BKP_DR1, 0xA5A5);			//在备份寄存器写入自己规定的标志位，用于判断RTC是不是第一次执行配置
		}
		else													//RTC不是第一次配置
		{
			RTC_WaitForSynchro();								//等待同步
			RTC_WaitForLastTask();								//等待上一次操作完成
		}
	}
	```

2. RTC设置时间

	```c
	uint16_t MyRTC_Time[] = {2023, 1, 1, 23, 59, 55};	//定义全局的时间数组，数组内容分别为年、月、日、时、分、秒
	
	/**
	  * 函    数：RTC设置时间
	  * 参    数：无
	  * 返 回 值：无
	  * 说    明：调用此函数后，全局数组里时间值将刷新到RTC硬件电路
	  */
	void MyRTC_SetTime(void)
	{
		time_t time_cnt;		//定义秒计数器数据类型
		struct tm time_date;	//定义日期时间数据类型
		
		time_date.tm_year = MyRTC_Time[0] - 1900;		//将数组的时间赋值给日期时间结构体
		time_date.tm_mon = MyRTC_Time[1] - 1;
		time_date.tm_mday = MyRTC_Time[2];
		time_date.tm_hour = MyRTC_Time[3];
		time_date.tm_min = MyRTC_Time[4];
		time_date.tm_sec = MyRTC_Time[5];
		
		time_cnt = mktime(&time_date) - 8 * 60 * 60;	//调用mktime函数，将日期时间转换为秒计数器格式
														//- 8 * 60 * 60为东八区的时区调整
		
		RTC_SetCounter(time_cnt);						//将秒计数器写入到RTC的CNT中
		RTC_WaitForLastTask();							//等待上一次操作完成
	}
	```

3. RTC读取时间

	```c
	/**
	  * 函    数：RTC读取时间
	  * 参    数：无
	  * 返 回 值：无
	  * 说    明：调用此函数后，RTC硬件电路里时间值将刷新到全局数组
	  */
	void MyRTC_ReadTime(void)
	{
		time_t time_cnt;		//定义秒计数器数据类型
		struct tm time_date;	//定义日期时间数据类型
		
		time_cnt = RTC_GetCounter() + 8 * 60 * 60;		//读取RTC的CNT，获取当前的秒计数器
														//+ 8 * 60 * 60为东八区的时区调整
		
		time_date = *localtime(&time_cnt);				//使用localtime函数，将秒计数器转换为日期时间格式
		
		MyRTC_Time[0] = time_date.tm_year + 1900;		//将日期时间结构体赋值给数组的时间
		MyRTC_Time[1] = time_date.tm_mon + 1;
		MyRTC_Time[2] = time_date.tm_mday;
		MyRTC_Time[3] = time_date.tm_hour;
		MyRTC_Time[4] = time_date.tm_min;
		MyRTC_Time[5] = time_date.tm_sec;
	}
	```


### PWR电源控制

#### 简介

- PWR（Power Control）电源控制

- PWR负责管理STM32内部的电源供电部分，可以实现可编程电压监测器和低功耗模式的功能

- 可编程电压监测器（PVD）可以监控VDD电源电压，当VDD下降到PVD阀值以下或上升到PVD阀值之上时，PVD会触发中断，用于执行紧急关闭任务

- 低功耗模式包括睡眠模式（Sleep）、停机模式（Stop）和待机模式（Standby），可在系统空闲时，降低STM32的功耗，延长设备使用时间

#### 低功耗模式

![image-20231213081651353](assets\image-35.png)

#### 模式选择

执行WFI（Wait For Interrupt）或者WFE（Wait For Event）指令后，STM32进入低功耗模式

![image-20231213081808454](assets\image-36.png)

#### 睡眠模式

- 执行完WFI/WFE指令后，STM32进入睡眠模式，程序暂停运行，唤醒后程序从暂停的地方继续运行
- SLEEPONEXIT位决定STM32执行完WFI或WFE后，是立刻进入睡眠，还是等STM32从最低优先级的中断处理程序中退出时进入睡眠
- 在睡眠模式下，所有的I/O引脚都保持它们在运行模式时的状态
- WFI指令进入睡眠模式，可被任意一个NVIC响应的中断唤醒
- WFE指令进入睡眠模式，可被唤醒事件唤醒

```c
while (1)
{
    --- 代码部分（这里需要有外部中断，唤醒cpu） ---

    __WFI();	// 进入睡眠模式，可由中断
}
```

#### 停止模式

- 执行完WFI/WFE指令后，STM32进入停止模式，程序暂停运行，唤醒后程序从暂停的地方继续运行
- 1.8V供电区域的所有时钟都被停止，PLL、HSI和HSE被禁止，SRAM和寄存器内容被保留下来
- 在停止模式下，所有的I/O引脚都保持它们在运行模式时的状态
- 当一个中断或唤醒事件导致退出停止模式时，HSI被选为系统时钟
- 当电压调节器处于低功耗模式下，系统从停止模式退出时，会有一段额外的启动延时
- WFI指令进入停止模式，可被任意一个EXTI中断唤醒
- WFE指令进入停止模式，可被任意一个EXTI事件唤醒

```c
/*开启时钟*/
RCC_APB1PeriphClockCmd(RCC_APB1Periph_PWR, ENABLE);		//开启PWR的时钟
//停止模式和待机模式一定要记得开启

while (1)
{
    // 主程序
    PWR_EnterSTOPMode(PWR_Regulator_ON, PWR_STOPEntry_WFI);	//STM32进入停止模式，并等待中断唤醒
    SystemInit();										//唤醒后，要重新配置时钟
}
```

#### 待机模式

- 执行完WFI/WFE指令后，STM32进入待机模式，唤醒后程序从头开始运行
- 整个1.8V供电区域被断电，PLL、HSI和HSE也被断电，SRAM和寄存器内容丢失，只有备份的寄存器和待机电路维持供电
- 在待机模式下，所有的I/O引脚变为高阻态（浮空输入）
- WKUP引脚的上升沿、RTC闹钟事件的上升沿、NRST引脚上外部复位、IWDG复位退出待机模式

```c
// 实时时钟代码的基础上修改的
int main(void)
{
	/*模块初始化*/
	OLED_Init();		//OLED初始化
	MyRTC_Init();		//RTC初始化
	
	/*开启时钟*/
	RCC_APB1PeriphClockCmd(RCC_APB1Periph_PWR, ENABLE);		//开启PWR的时钟
															//停止模式和待机模式一定要记得开启
	
	/*显示静态字符串*/
	OLED_ShowString(1, 1, "CNT :");
	OLED_ShowString(2, 1, "ALR :");
	OLED_ShowString(3, 1, "ALRF:");
	
	/*使能WKUP引脚*/
	PWR_WakeUpPinCmd(ENABLE);						//使能位于PA0的WKUP引脚，WKUP引脚上升沿唤醒待机模式
	
	/*设定闹钟*/
	uint32_t Alarm = RTC_GetCounter() + 10;			//闹钟为唤醒后当前时间的后10s
	RTC_SetAlarm(Alarm);							//写入闹钟值到RTC的ALR寄存器
	OLED_ShowNum(2, 6, Alarm, 10);					//显示闹钟值
	
	while (1)
	{
		OLED_ShowNum(1, 6, RTC_GetCounter(), 10);	//显示32位的秒计数器
		OLED_ShowNum(3, 6, RTC_GetFlagStatus(RTC_FLAG_ALR), 1);		//显示闹钟标志位
		
		OLED_ShowString(4, 1, "Running");			//OLED闪烁Running，指示当前主循环正在运行
		Delay_ms(100);
		OLED_ShowString(4, 1, "       ");
		Delay_ms(100);
		
		OLED_ShowString(4, 9, "STANDBY");			//OLED闪烁STANDBY，指示即将进入待机模式
		Delay_ms(1000);
		OLED_ShowString(4, 9, "       ");
		Delay_ms(100);
		
		OLED_Clear();								//OLED清屏，模拟关闭外部所有的耗电设备，以达到极度省电
		
		PWR_EnterSTANDBYMode();						//STM32进入停止模式，并等待指定的唤醒事件（WKUP上升沿或RTC闹钟）
		/*待机模式唤醒后，程序会重头开始运行*/
	}
}
```



#### 修改主频

在system_stm32f10x.c文件中使用了宏定义配置了系统的主频，需要修改的话解除相应的注释即可

```c
#if defined (STM32F10X_LD_VL) || (defined STM32F10X_MD_VL) || (defined STM32F10X_HD_VL)
/* #define SYSCLK_FREQ_HSE    HSE_VALUE */
 #define SYSCLK_FREQ_24MHz  24000000
#else
/* #define SYSCLK_FREQ_HSE    HSE_VALUE */
/* #define SYSCLK_FREQ_24MHz  24000000 */ 
/* #define SYSCLK_FREQ_36MHz  36000000 */
/* #define SYSCLK_FREQ_48MHz  48000000 */
/* #define SYSCLK_FREQ_56MHz  56000000 */
#define SYSCLK_FREQ_72MHz  72000000
#endif
```

### WDG看门狗

#### 简介

- 看门狗可以监控程序的运行状态，当程序因为设计漏洞、硬件故障、电磁干扰等原因，出现卡死或跑飞现象时，看门狗能及时复位程序，避免程序陷入长时间的罢工状态，保证系统的可靠性和安全性

- 看门狗本质上是一个定时器，当指定时间范围内，程序没有执行喂狗（重置计数器）操作时，看门狗硬件电路就自动产生复位信号

- STM32内置两个看门狗

	- 独立看门狗（IWDG）：独立工作，对时间精度要求较低

	- 窗口看门狗（WWDG）：要求看门狗在精确计时窗口起作用

#### IWDG框图

![image-20231213111122352](assets\image-37.png)

#### IWDG键寄存器

- 键寄存器本质上是控制寄存器，用于控制硬件电路的工作

- 在可能存在干扰的情况下，一般通过在整个键寄存器写入特定值来代替控制寄存器写入一位的功能，以降低硬件电路受到干扰的概率

| **写入键寄存器的值** | **作用**                               |
| -------------------- | -------------------------------------- |
| 0xCCCC               | 启用独立看门狗                         |
| 0xAAAA               | IWDG_RLR中的值重新加载到计数器（喂狗） |
| 0x5555               | 解除IWDG_PR和IWDG_RLR的写保护          |
| 0x5555之外的其他值   | 启用IWDG_PR和IWDG_RLR的写保护          |

#### IWDG超时时间

超时时间：TIWDG = TLSI × PR预分频系数 × (RL + 1)

其中：TLSI = 1 / FLSI

![image-20231213111248845](assets\image-38.png) 

#### WWDG框图

![image-20231213111504940](assets\image-39.png) 

#### WWDG超时时间

超时时间：TWWDG = TPCLK1 × 4096 × WDGTB预分频系数 × (T[5:0] + 1)

窗口时间：TWIN = TPCLK1 × 4096 × WDGTB预分频系数 × (T[5:0] - W[5:0])

其中：TPCLK1 = 1 / FPCLK1

![image-20231213111600526](assets\image-40.png) 

#### IWDG和WWDG对比

|            | **IWDG独立看门狗**           | **WWDG窗口看门狗**                  |
| ---------- | ---------------------------- | ----------------------------------- |
| 复位       | 计数器减到0后                | 计数器T[5:0]减到0后、过早重装计数器 |
| 中断       | 无                           | 早期唤醒中断                        |
| 时钟源     | LSI（40KHz）                 | PCLK1（36MHz）                      |
| 预分频系数 | 4、8、32、64、128、256       | 1、2、4、8                          |
| 计数器     | 12位                         | 6位（有效计数）                     |
| 超时时间   | 0.1ms~26214.4ms              | 113us~58.25ms                       |
| 喂狗方式   | 写入键寄存器，重装固定值RLR  | 直接写入计数器，写多少重装多少      |
| 防误操作   | 键寄存器和写保护             | 无                                  |
| 用途       | 独立工作，对时间精度要求较低 | 要求看门狗在精确计时窗口起作用      |

#### 独立看门狗

```c
int main(void)
{
	/*模块初始化*/
	OLED_Init();						//OLED初始化
	Key_Init();							//按键初始化
	
	/*显示静态字符串*/
	OLED_ShowString(1, 1, "IWDG TEST");
	
	/*判断复位信号来源*/
	if (RCC_GetFlagStatus(RCC_FLAG_IWDGRST) == SET)	//如果是独立看门狗复位
	{
		OLED_ShowString(2, 1, "IWDGRST");			//OLED闪烁IWDGRST字符串
		Delay_ms(500);
		OLED_ShowString(2, 1, "       ");
		Delay_ms(100);
		
		RCC_ClearFlag();							//清除标志位
	}
	else											//否则，即为其他复位
	{
		OLED_ShowString(3, 1, "RST");				//OLED闪烁RST字符串
		Delay_ms(500);
		OLED_ShowString(3, 1, "   ");
		Delay_ms(100);
	}
	
	/*IWDG初始化*/
	IWDG_WriteAccessCmd(IWDG_WriteAccess_Enable);	//独立看门狗写使能
	IWDG_SetPrescaler(IWDG_Prescaler_16);			//设置预分频为16
	IWDG_SetReload(2499);							//设置重装值为2499，独立看门狗的超时时间为1000ms
	IWDG_ReloadCounter();							//重装计数器，喂狗
	IWDG_Enable();									//独立看门狗使能
	
	while (1)
	{
		Key_GetNum();								//调用阻塞式的按键扫描函数，模拟主循环卡死
		
		IWDG_ReloadCounter();						//重装计数器，喂狗
		
		OLED_ShowString(4, 1, "FEED");				//OLED闪烁FEED字符串
		Delay_ms(200);								//喂狗间隔为200+600=800ms
		OLED_ShowString(4, 1, "    ");
		Delay_ms(600);
	}
}
```

#### 窗口看门狗

```c
int main(void)
{
	/*模块初始化*/
	OLED_Init();						//OLED初始化
	Key_Init();							//按键初始化
	
	/*显示静态字符串*/
	OLED_ShowString(1, 1, "WWDG TEST");
	
	/*判断复位信号来源*/
	if (RCC_GetFlagStatus(RCC_FLAG_WWDGRST) == SET)	//如果是窗口看门狗复位
	{
		OLED_ShowString(2, 1, "WWDGRST");			//OLED闪烁WWDGRST字符串
		Delay_ms(500);
		OLED_ShowString(2, 1, "       ");
		Delay_ms(100);
		
		RCC_ClearFlag();							//清除标志位
	}
	else											//否则，即为其他复位
	{
		OLED_ShowString(3, 1, "RST");				//OLED闪烁RST字符串
		Delay_ms(500);
		OLED_ShowString(3, 1, "   ");
		Delay_ms(100);
	}
	
	/*开启时钟*/
	RCC_APB1PeriphClockCmd(RCC_APB1Periph_WWDG, ENABLE);	//开启WWDG的时钟
	
	/*WWDG初始化*/
	WWDG_SetPrescaler(WWDG_Prescaler_8);			//设置预分频为8
	WWDG_SetWindowValue(0x40 | 21);					//设置窗口值，窗口时间为30ms
	WWDG_Enable(0x40 | 54);							//使能并第一次喂狗，超时时间为50ms
	
	while (1)
	{
		Key_GetNum();								//调用阻塞式的按键扫描函数，模拟主循环卡死
		
		OLED_ShowString(4, 1, "FEED");				//OLED闪烁FEED字符串
		Delay_ms(20);								//喂狗间隔为20+20=40ms
		OLED_ShowString(4, 1, "    ");
		Delay_ms(20);
		
		WWDG_SetCounter(0x40 | 54);					//重装计数器，喂狗
	}
}
```

### Flash

#### 简介

- STM32F1系列的FLASH包含程序存储器、系统存储器和选项字节三个部分，通过闪存存储器接口（外设）可以对程序存储器和选项字节进行擦除和编程

- 读写FLASH的用途：

	- 利用程序存储器的剩余空间来保存掉电不丢失的用户数据

	- 通过在程序中编程（IAP），实现程序的自我更新

- 在线编程（In-Circuit Programming – ICP）用于更新程序存储器的全部内容，它通过JTAG、SWD协议或系统加载程序（Bootloader）下载程序

- 在程序中编程（In-Application Programming – IAP）可以使用微控制器支持的任一种通信接口下载程序

#### FLASH解锁

- FPEC共有三个键值：

	- RDPRT键 = 0x000000A5

	- KEY1 = 0x45670123

	- KEY2 = 0xCDEF89AB

- 解锁：

	- 复位后，FPEC被保护，不能写入FLASH_CR

	- 在FLASH_KEYR先写入KEY1，再写入KEY2，解锁

	- 错误的操作序列会在下次复位前锁死FPEC和FLASH_CR

- 加锁：
	- 设置FLASH_CR中的LOCK位锁住FPEC和FLASH_CR

#### 使用指针访问存储器

- 使用指针读指定地址下的存储器：
	- uint16_t Data = *((__IO uint16_t *)(0x08000000));

- 使用指针写指定地址下的存储器：
	- *((__IO uint16_t *)(0x08000000)) = 0x1234;

- 其中：
	- \#define  __IO  volatile

#### 程序存储器编程

![image-20231213212122400](assets\image-41.png) 

#### 程序存储器页擦除

![image-20231213212149160](assets\image-42.png) 

#### 程序存储器全擦除

![image-20231213212220618](assets\image-43.png) 

#### FLASH相关操作

```c
#include "stm32f10x.h"                  // Device header

/**
  * 函    数：FLASH读取一个32位的字
  * 参    数：Address 要读取数据的字地址
  * 返 回 值：指定地址下的数据
  */
uint32_t MyFLASH_ReadWord(uint32_t Address)
{
	return *((__IO uint32_t *)(Address));	//使用指针访问指定地址下的数据并返回
}

/**
  * 函    数：FLASH读取一个16位的半字
  * 参    数：Address 要读取数据的半字地址
  * 返 回 值：指定地址下的数据
  */
uint16_t MyFLASH_ReadHalfWord(uint32_t Address)
{
	return *((__IO uint16_t *)(Address));	//使用指针访问指定地址下的数据并返回
}

/**
  * 函    数：FLASH读取一个8位的字节
  * 参    数：Address 要读取数据的字节地址
  * 返 回 值：指定地址下的数据
  */
uint8_t MyFLASH_ReadByte(uint32_t Address)
{
	return *((__IO uint8_t *)(Address));	//使用指针访问指定地址下的数据并返回
}

/**
  * 函    数：FLASH全擦除
  * 参    数：无
  * 返 回 值：无
  * 说    明：调用此函数后，FLASH的所有页都会被擦除，包括程序文件本身，擦除后，程序将不复存在
  */
void MyFLASH_EraseAllPages(void)
{
	FLASH_Unlock();					//解锁
	FLASH_EraseAllPages();			//全擦除
	FLASH_Lock();					//加锁
}

/**
  * 函    数：FLASH页擦除
  * 参    数：PageAddress 要擦除页的页地址
  * 返 回 值：无
  */
void MyFLASH_ErasePage(uint32_t PageAddress)
{
	FLASH_Unlock();					//解锁
	FLASH_ErasePage(PageAddress);	//页擦除
	FLASH_Lock();					//加锁
}

/**
  * 函    数：FLASH编程字
  * 参    数：Address 要写入数据的字地址
  * 参    数：Data 要写入的32位数据
  * 返 回 值：无
  */
void MyFLASH_ProgramWord(uint32_t Address, uint32_t Data)
{
	FLASH_Unlock();							//解锁
	FLASH_ProgramWord(Address, Data);		//编程字
	FLASH_Lock();							//加锁
}

/**
  * 函    数：FLASH编程半字
  * 参    数：Address 要写入数据的半字地址
  * 参    数：Data 要写入的16位数据
  * 返 回 值：无
  */
void MyFLASH_ProgramHalfWord(uint32_t Address, uint16_t Data)
{
	FLASH_Unlock();							//解锁
	FLASH_ProgramHalfWord(Address, Data);	//编程半字
	FLASH_Lock();							//加锁
}
```

#### FLASH基础上数据

```c
#include "stm32f10x.h"                  // Device header
#include "MyFLASH.h"

#define STORE_START_ADDRESS		0x0800FC00		//存储的起始地址
#define STORE_COUNT				512				//存储数据的个数

uint16_t Store_Data[STORE_COUNT];				//定义SRAM数组

/**
  * 函    数：参数存储模块初始化
  * 参    数：无
  * 返 回 值：无
  */
void Store_Init(void)
{
	/*判断是不是第一次使用*/
	if (MyFLASH_ReadHalfWord(STORE_START_ADDRESS) != 0xA5A5)	//读取第一个半字的标志位，if成立，则执行第一次使用的初始化
	{
		MyFLASH_ErasePage(STORE_START_ADDRESS);					//擦除指定页
		MyFLASH_ProgramHalfWord(STORE_START_ADDRESS, 0xA5A5);	//在第一个半字写入自己规定的标志位，用于判断是不是第一次使用
		for (uint16_t i = 1; i < STORE_COUNT; i ++)				//循环STORE_COUNT次，除了第一个标志位
		{
			MyFLASH_ProgramHalfWord(STORE_START_ADDRESS + i * 2, 0x0000);		//除了标志位的有效数据全部清0
		}
	}
	
	/*上电时，将闪存数据加载回SRAM数组，实现SRAM数组的掉电不丢失*/
	for (uint16_t i = 0; i < STORE_COUNT; i ++)					//循环STORE_COUNT次，包括第一个标志位
	{
		Store_Data[i] = MyFLASH_ReadHalfWord(STORE_START_ADDRESS + i * 2);		//将闪存的数据加载回SRAM数组
	}
}

/**
  * 函    数：参数存储模块保存数据到闪存
  * 参    数：无
  * 返 回 值：无
  */
void Store_Save(void)
{
	MyFLASH_ErasePage(STORE_START_ADDRESS);				//擦除指定页
	for (uint16_t i = 0; i < STORE_COUNT; i ++)			//循环STORE_COUNT次，包括第一个标志位
	{
		MyFLASH_ProgramHalfWord(STORE_START_ADDRESS + i * 2, Store_Data[i]);	//将SRAM数组的数据备份保存到闪存
	}
}

/**
  * 函    数：参数存储模块将所有有效数据清0
  * 参    数：无
  * 返 回 值：无
  */
void Store_Clear(void)
{
	for (uint16_t i = 1; i < STORE_COUNT; i ++)			//循环STORE_COUNT次，除了第一个标志位
	{
		Store_Data[i] = 0x0000;							//SRAM数组有效数据清0
	}
	Store_Save();										//保存数据到闪存
}
```

