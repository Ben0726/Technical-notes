>  **LED数码管或者LCD12864等显示器在显示温度采样值时，需要的是稳定的能反映温度实际变化的显示效果。ADC采样频率快，直接采样输出的值往往有不小的跳变，这时候需要对其数据做一些数字滤波处理。常用的方法有：平均值法、多次采样用冒泡法取中间一段数值法、平滑平均线法。冒泡法处理时间太长，这里主要用了平均值和平滑平均线两种方法一起处理。**

**一、算法部分**
---
> **平滑平均线法**可解决一般均值法的欠灵敏和滞后性，同时滤除毛刺。用此法对ADC采集的数据进行ADC滤波后，在LCD12864上显示稳定。

 **具体做法如下：**
1. 在一个ADC通道上采集多次，获得采样平均值a，用a初始化含有假定100个数据的数组b[100]中所有的数据(这样b[]数组的平均值就是a)；
2. 后续每次运行一次ADC采样都会得到一个新的采样平均值A，同时把数组b的数据左移（b[0]=b[1]~~~b[98]=b[99]）,然后把A的值赋值给b[99]；
3. 如此每次运行一次采样就会平滑的更新一次数组b的平均值B，也就得到了数字滤波后的平滑平均值B，平均值B就可以拿去做显示，效果稳定；
4. **实际程序处理中是：第一次得到的采样平均值A0赋值给b[0]，如此顺序，第100次采样得到的平均值A99赋值给b[99]，之后又重新依次更新b[0]到b[99]的值，如此循环，这样做可以减少运算量，同时达到左移的效果**

**平滑平均值代码**
```c
/**
  * @brief  ADC采样所得到的温度
  * @param None
  * @retval : None
  */
void ADC1_Tempera(void)
{
	u32 i,k,m,n;
	static u32 ValueCount;
	for(i=0;i<ADC_ChannelCount;i++)
	{
		/*
		*@平均值滤波
		*/
		ValueCount=0;
		for(k=0;k<ADC1_Count;k++)//取中间的100次值(ADC1_Count)
		{
			ValueCount+=ADGetValue[k][i];
			ADC_Filter[i]=(ValueCount/ADC1_Count);//求取一个ADC通道的平均值
		}
		if(Tflag==0)
		{
			for(n=0;n<SmoothNum;n++)
			{
				ADC_SmoothFilter[i][n]=ADC_Filter[i];//获得第一次平滑值
			}
			ADC_SmoothCount[i]=ADC_Filter[i];//获取第一次平滑平均值线
		}
		/*
		*@平滑滤波
		*/
		if(Tflag==1)
		{
			ADC_SmoothFilter[i][Smoth_Num[i]]=ADC_Filter[i];//把数值循环放置
			Smoth_Num[i]++;
			if(Smoth_Num[i]==SmoothNum)
				Smoth_Num[i]=0;
			for(n=0;n<SmoothNum;n++)//平滑滤波
			{
				ADC_SmoothCount[i] += ADC_SmoothFilter[i][n];
			}
			ADC_SmoothCount[i]=(ADC_SmoothCount[i]/SmoothNum);//获取平滑平均值线
		}
	}
	for(m=0;m<3;m++)
	{
		TMonitor[m]=(((2361*ADC_SmoothCount[2*m])/(ADC_SmoothCount[2*m+1])-2597)+TemperaSet[m+4]);//已经扩大10倍的值转化为温度数据,低温,高温,环境温度 TemperaSet[m+4]为温度补偿系数
		if(Tflag==0)
			TMonitorDis[m]=TMonitor[m];
		/*
		*@显示处理缓慢变化
		*/
		if(TMonitorDis[m]<(TMonitor[m]-1))
			TMonitorDis[m] += 1;
		if(TMonitorDis[m]>(TMonitor[m]+1))
			TMonitorDis[m] -= 1;
	}
	Tflag=1;
}
```
**代码注解**
* ADC_ChannelCount为采样通道数，其值为6，表示ADC1有6个采样通道。
* ADC1_Count为每一个通道在执行一次采样函数中需要采样的次数，此值为100。
* ADGetValue[ADC1_Count][ADC_ChannelCount]数组内存中存储的是6个通道中每个通道的100个数据，即每执行一次函数void ADC1_Tempera(void)就更新一次这段内存的所有数据。
* **批量采样工作过程是：定时器触发一次后依次采样通道（1-2-3-4-5-6），第二次定时器触发后依次采样通道（1-2-3-4-5-6），如此连续转换，循环扫描；所以要想每个通道都采样100次，那么就需要触发转换100次，这样定义一段内存存储数据的数组就应该是ADGetValue[100][6]，这样在i=0~99情况下100个数据ADGetValue[i][0]表示为通道1在采样了100次后所得到的数据。**

**ADC通道配置**
---
**DMA模式配置要点**

* STM32的ADC1作为DMA的外设源，其地址为16位，如果变量值不是16位那么所得到的数据会混乱，但是DMA配置里是32位，所以需要强制转换为32位。
```c
vu16 ADGetValue[][];//AD采样值,ADC_ChannelCount个通道,每个通道采样ADC1_Count次
DMA_InitStructure.DMA_MemoryBaseAddr = (u32)&ADGetValue;//设置DMA内存地址，ADC转换结果直接放入该地址
```
*  使用定时器外部触发转换，ADC六个通道采样，循环采样模式、连续转换。
```c
ADC_InitStructure.ADC_Mode = ADC_Mode_Independent; 		//使用独立模式，扫描模式
	ADC_InitStructure.ADC_ScanConvMode = ENABLE;			//循环扫描模式
	ADC_InitStructure.ADC_ContinuousConvMode = ENABLE; 		//连续转换模式
	ADC_InitStructure.ADC_ExternalTrigConv = ADC_ExternalTrigConv_T1_CC1; //外部定时器触发转换
```

**ADC配置代码**
```c
/**
  * @brief  配置采样通道端口 使能GPIO时钟	 设置ADC采样PA0端口信号
  * @param None
  * @retval : None
  */
void ADC1_GPIO_Config(void)
{ 
	GPIO_InitTypeDef ADC_PORT;    
	RCC_APB2PeriphClockCmd(ADC_ClK, ENABLE);	  
	ADC_PORT.GPIO_Pin = ADC_Pin;				//ADC引脚选择,6个通道
	ADC_PORT.GPIO_Mode = GPIO_Mode_AIN;		    //GPIO设置为模拟输入
	GPIO_Init(ADC_CONTROL, &ADC_PORT);   
}
/**
  * @brief  配置ADC1的工作模式为DMA模式
  * @param None
  * @retval : None
  */
 void ADC1_Mode_Config(void)
 {
	ADC_InitTypeDef ADC_InitStructure;	
	RCC_APB2PeriphClockCmd(RCC_APB2Periph_ADC1, ENABLE);	//使能ADC1时钟 
	/* ADC1 configuration */
	ADC_InitStructure.ADC_Mode = ADC_Mode_Independent; 		//使用独立模式，扫描模式
	ADC_InitStructure.ADC_ScanConvMode = ENABLE;			//循环扫描模式
	ADC_InitStructure.ADC_ContinuousConvMode = ENABLE; 		//连续转换模式
	ADC_InitStructure.ADC_ExternalTrigConv = ADC_ExternalTrigConv_T1_CC1; //外部定时器触发转换
	ADC_InitStructure.ADC_DataAlign = ADC_DataAlign_Right;//数据右对齐
	ADC_InitStructure.ADC_NbrOfChannel = ADC_ChannelCount;  //有ADC_ChannelCount个转换通道
	ADC_Init(ADC1, &ADC_InitStructure);
	/* 以下为配置通道数总共6路,分别是AD10,AD11,AD12,AD13,AD14,AD15 */
	/* ADC1 regular channel10 configuration */ 
	ADC_RegularChannelConfig(ADC1, ADC_Channel_10, 1, ADC_SampleTime_55Cycles5); //通道10,作为第1个采样目标,采样周期239.5个时钟周期
	/* ADC1 regular channel11 configuration */ 
	ADC_RegularChannelConfig(ADC1, ADC_Channel_11, 2, ADC_SampleTime_55Cycles5); //通道11,作为第2个采样目标,采样周期55.5个时钟周期
	/* ADC1 regular channel12 configuration */ 
	ADC_RegularChannelConfig(ADC1, ADC_Channel_12, 3, ADC_SampleTime_55Cycles5); //通道12,作为第3个采样目标,采样周期55.5个时钟周期
	/* ADC1 regular channel13 configuration */ 
	ADC_RegularChannelConfig(ADC1, ADC_Channel_13, 4, ADC_SampleTime_55Cycles5); //通道13,作为第4个采样目标,采样周期55.5个时钟周期
	/* ADC1 regular channel14 configuration */ 
	ADC_RegularChannelConfig(ADC1, ADC_Channel_14, 5, ADC_SampleTime_55Cycles5); //通道14,作为第5个采样目标,采样周期55.5个时钟周期
	/* ADC1 regular channel15 configuration */ 
	ADC_RegularChannelConfig(ADC1, ADC_Channel_15, 6, ADC_SampleTime_55Cycles5); //通道15,作为第6个采样目标,采样周期55.5个时钟周期

	/* Enable ADC1 DMA */
	ADC_DMACmd(ADC1, ENABLE);	 //使能ADC的DMA
	/*使能ADC1外部触发*/
	ADC_ExternalTrigConvCmd(ADC1,ENABLE);
	/* Enable ADC1 */
	ADC_Cmd(ADC1, ENABLE); //使能ADC1

	/* Enable ADC1 reset calibaration register */   
	ADC_ResetCalibration(ADC1);
	/* Check the end of ADC1 reset calibration register */
	while(ADC_GetResetCalibrationStatus(ADC1));

	/* Start ADC1 calibaration */
	ADC_StartCalibration(ADC1);
	/* Check the end of ADC1 calibration */
	while(ADC_GetCalibrationStatus(ADC1));
	 
	/* Start ADC1 Software Conversion */ 
	ADC_SoftwareStartConvCmd(ADC1, ENABLE);  //开始转换
	
}
/**
  * @brief  配置DMA
  * @param None
  * @retval : None
  */
 void DMA_Config(void)
 {
	DMA_InitTypeDef DMA_InitStructure;
	RCC_AHBPeriphClockCmd(RCC_AHBPeriph_DMA1, ENABLE); //使能DMA1时钟
	/* DMA1 channel1 configuration */
	DMA_DeInit(DMA1_Channel1);  //指定DMA1通道1
	DMA_InitStructure.DMA_PeripheralBaseAddr = ADC1_DR_Address;//设置DMA外设地址
	DMA_InitStructure.DMA_MemoryBaseAddr = (u32)&ADGetValue;//设置DMA内存地址，ADC转换结果直接放入该地址
	DMA_InitStructure.DMA_DIR = DMA_DIR_PeripheralSRC; //外设设置为数据传输的来源
	DMA_InitStructure.DMA_BufferSize = (ADC_ChannelCount* ADC1_Count);	//DMA缓冲区设置为ADC_ChannelCount* ADC1_Count,有6个通道；
	DMA_InitStructure.DMA_PeripheralInc = DMA_PeripheralInc_Disable;//外设地址不增加
	DMA_InitStructure.DMA_MemoryInc = DMA_MemoryInc_Enable;//内存地址增加
	DMA_InitStructure.DMA_PeripheralDataSize = DMA_PeripheralDataSize_HalfWord;//外设源的数据宽度半字,ADC1_DR的值16位
	DMA_InitStructure.DMA_MemoryDataSize = DMA_MemoryDataSize_HalfWord;
	DMA_InitStructure.DMA_Mode = DMA_Mode_Circular;//循环模式
	DMA_InitStructure.DMA_Priority = DMA_Priority_High;
	DMA_InitStructure.DMA_M2M = DMA_M2M_Disable;
	DMA_Init(DMA1_Channel1, &DMA_InitStructure);

	/* Enable DMA channel1 */
	DMA_Cmd(DMA1_Channel1, ENABLE);  //使能DMA通道
	//DMA_ITConfig(DMA1_Channel1, DMA_IT_TC, ENABLE); //使能DMA传输完成中断 
}

/**
  * @brief  初始化ADC1
  * @param None
  * @retval : None
  */
void ADC1_Init(void)
{
	ADC1_GPIO_Config();
	ADC1_Mode_Config();
	DMA_Config();
	TIM1_Mode_Config();//开启定时器1，产生PWN1外部触发源
}
```
