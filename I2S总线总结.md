# 导语： I2S总线是专门开发传输音频信号的总线

## 一、自然界声音频率的范围

>![自然界声音频率范围.png](http://upload-images.jianshu.io/upload_images/4859654-488c294eccd74972.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
* 如上图所示，自然界声音的频率范围最大为20KHz，根据香浓采样定律，采样频率只要大于2*Fs，采集的数据即可无失真的还原原始信号，常见的CD，采样率为44.1KHz；这样当采样频率达到48KHz时即可无失真采样还原所有的声音信号。

## 二、数字音频的质量
>* **数字音频的质量取决于**：**采样频率**和**量化位数**这两个参数，为了保真，在时间变化方向上取样点尽量密，取样率要高；在幅度取值上尽量细，量化比特率要高，直接的结果就是存储容量及传输信道容量要求的压力大；
* **音频信号的传输率= 取样频率 × 样本的量化比特数 × 通道数。**
![**传输速率与数据量举例**](http://upload-images.jianshu.io/upload_images/4859654-5fba7f4776315224.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 三、I2S总线简述
> * I2S总线主要有3个信号：串行时钟BCLK、帧时钟LRCK、串行数据SDATA；有时为了使系统间能够更好地同步，还需要另外传输一个信号MCLK，称为主时钟，也叫系统时钟（Sys Clock），是采样频率的256倍或384倍。

* 串行时钟BCLK：也叫SCLK，即对应数字音频的每一位数据，SCLK都有1个脉冲；SCLK的频率=2×采样频率×采样位数，2倍是因为I2S有左右两个通道。
* 帧时钟LRCK：用于切换左右声道的数据。LRCK为“1”表示正在传输的是左声道的数据，为“0”则表示正在传输的是右声道的数据；LRCK的频率等于采样频率。
* 串行数据SDATA：就是用二进制补码表示的音频数据。

## 四、数据信号特点
>* I2S格式的信号无论有多少位有效数据，数据的最高位总是出现在LRCK变化（也就是一帧开始）后的第2个SCLK脉冲处。这就使得接收端与发送端的有效位数可以不同。如果接收端能处理的有效位数少于发送端，可以放弃数据帧中多余的低位数据；如果接收端能处理的有效位数多于发送端，可以自行补足剩余的位。这种同步机制使得数字音频设备的互连更加方便，而且不会造成数据错位。

* **左对齐格式**

![左对齐格式](http://upload-images.jianshu.io/upload_images/4859654-8d7089b67a52675f.PNG?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

* **右对齐格式**

![右对齐格式](http://upload-images.jianshu.io/upload_images/4859654-1f4db673c3c8e977.PNG?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
