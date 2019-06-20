# Audio PSU Limit
---
###### Author : 冯文运
###### Date: 2019 - 06 - 20
---
## Overview
> * **PSU** : Power support unit;
> * 音响类功放持续输出的功率一般是其额定最大功率的-9dB，而Burst信号输出的最大功率能达到额定最大功率即可，其原因是输出大功率音频信号时间久了之后供电电源电压会下降很厉害甚至烧机，所以一般会设计PSU Limit 功能。

## 一、IPD与IPX对比
> ### 1、PSU控制方式
> * IPD采用ADC芯片实时采集PSU电压做反馈控制，控制比较高效精准；
> * IPX根据PSU给出的两个逻辑电平状态来做反馈控制，效率较差，后续趋向于采用ADC采集PSU电压的方式来做反馈控制
![PSU Control Diagram](https://img-blog.csdnimg.cn/20190618114423867.PNG?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JlbnRlbmdkZW1h,size_16,color_FFFFFF,t_70)
> ### 2、PSU下降波形
> * IPD 每次输出大功率一段时间后PSU电压开始下降，PSU电压先迅速下降到51Vrms左右，然后PSU电压随着功放输出功率指数阶梯式下降的调节再缓慢下降并稳定在46Vrms附近，稳定后的输出功率4R负载下为其额定最大功率的1/8、8R负载下功率1/16；并且在运行期间变换不同大小负载或者输入信号都可迅速一次性调节至稳定。
>![IPD的PSU降压保护](https://img-blog.csdnimg.cn/20190618135358419.PNG?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JlbnRlbmdkZW1h,size_16,color_FFFFFF,t_70)
> * IPX在输出大功率信号一段时间后PSU电压开始下降，迅速下降至56Vrms然后跟随功放输出功率不断变化最后稳定在50Vrms ~ 15Vrms之间后稳定输出，稳定输出功率4R负载下为其额定最大功率的1/8 ~ 1/6左右、8R负载下功率减半；目前IPX是靠两个逻辑电平反馈来控制PSU电压变化，调节效果有滞后性，在变换负载或者刚开始运行时需要先运行一次才能获得所要调节的目标值，获得目标值后若负载不变的情况下若有再次压缩的需要，其压缩调节效果大致和IPD的一致；因为是电平调节，PSU电压处在两个电平中间的位置都会认为是目标范围，因此随着PSU电压稳定值的不同，输出稳定的功率会有一定的大小差异。
> ![IPX的PSU降压保护](https://img-blog.csdnimg.cn/20190618144636334.PNG?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JlbnRlbmdkZW1h,size_16,color_FFFFFF,t_70)
> ### 3、Burst信号输出
>  * 一般使6dBu以下的Burst输入信号要尽可能不被压缩的输出，超过6dBu（1.51Vrms）输入信号其输出可能会被压缩。
>  ![IPD Burst 输出](https://img-blog.csdnimg.cn/201906181552169.PNG?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JlbnRlbmdkZW1h,size_16,color_FFFFFF,t_70)
>  * IPX是电平调节，在PSU掉到上面那根电平就开始调节，为了能让Burst信号全部出来，在硬件上下调了两个电平的参考电压，副作用是会使最后稳定的输出功率变得更大50W左右。
>  ![IPX Burst 输出](https://img-blog.csdnimg.cn/20190618160442523.PNG?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JlbnRlbmdkZW1h,size_16,color_FFFFFF,t_70)
## 二、PSU Limit 程序
> ### 1、流程图
>  * 采用平滑压缩方式，数学公式为S(n) = αS(n-1) +  (1 - α)G， 调节压缩时间系数$\alpha$的大小的时候要注意相邻点压缩的平滑连续性。
> * 逻辑电平表如下
> ![IPX PSU Limit逻辑电平表](https://img-blog.csdnimg.cn/20190620095411581.png)
> * 算法流程图
> ![PSU Limit 算法流程图](https://img-blog.csdnimg.cn/20190619154744360.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JlbnRlbmdkZW1h,size_16,color_FFFFFF,t_70)
> ### 2、调节时间
> * 2402接不同负载满功率输出信号时PSU电压掉到45Vrms以下危险区域的时间如下表所示，不同的负载输出大功率PSU电压下降到危险区域的时间不同
>  ![PSU电压下降时间](https://img-blog.csdnimg.cn/20190619173914880.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JlbnRlbmdkZW1h,size_16,color_FFFFFF,t_70)
> * 动态调整压缩时间控制参数$\alpha$的大小来兼顾不同负载的压缩过程，理论上会比限制压缩时间为单一最短值比如2ms会获得更好的听觉效果，程序中根据输出功率与参考功率的差值来动态调整这一压缩时间，最短为2ms最长为20ms。
> ![alpha调节](https://img-blog.csdnimg.cn/20190619181327546.PNG?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JlbnRlbmdkZW1h,size_16,color_FFFFFF,t_70)
> ### 3、调试过程遇到的破音问题
> * 压缩时间过短，压缩不连续平滑会导致声音突变；前期的做法是目标参考值(VthdBu)一改变对应的压缩比例立刻变化这样会导致相邻两个点的压缩比例差别巨大从而产生明显的声音失真刺耳，后来更改软件在目标参考值有变化时连续平滑的调整压缩比例，使相邻点能被连续平滑的压缩，在压缩时间2ms以上听不出音乐信号的突变；接两个2R负载时，IPD输出的Burst信号会变形失真，IPD新版硬件改善了这个问题。
> * IPX压缩破音图
> ![IPX PSU压缩破音](https://img-blog.csdnimg.cn/20190619185414991.PNG?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JlbnRlbmdkZW1h,size_16,color_FFFFFF,t_70)
> * IPX压缩改善后PSU Limit 波形图
> ![IPX PSU压缩改善](https://img-blog.csdnimg.cn/20190619190208751.PNG?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JlbnRlbmdkZW1h,size_16,color_FFFFFF,t_70)
> * IPX接2R负载满功率输出波形
> ![2R*2负载](https://img-blog.csdnimg.cn/20190619190359613.PNG?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JlbnRlbmdkZW1h,size_16,color_FFFFFF,t_70)
> * IPD的PSU Limit波形
> ![IPD PSU Limit 波形](https://img-blog.csdnimg.cn/20190619190520187.PNG?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JlbnRlbmdkZW1h,size_16,color_FFFFFF,t_70)

> ### 4、改进空间
> * **IPX** 用逻辑电平调节PSU输出功率的方式在调节过程中无法及时准确的获知PSU的实际输出电压值以及预判电压值下降或上升的趋势，这使得在DSP调节了压缩比例后无法及时的获得PSU输出电压下降还是上升，只有等到触发了逻辑电平状态改变的时候DSP才能大致的获知PSU输出电压的状态；由于接不同负载最后稳定输出所要求的功率会不同，这使得目标参考值(VthdBu)无法事先设定，在未获得目标参考值及PSU的输出电压未知的情况下IPX需要震荡调整寻找一个合适的目标参考值；
> * **IPD** 类似采用ADC采集PSU输出电压的方式，这样IPD输出功率压变化引起的PSU电压变化能及时的被DSP获知从而调整压缩系数，这种调节方式比较及时有效；
> * 倾向于用ADC采集PSU电压方式来做PSU Limit调节。
> * 以下是不同负载下，PSU稳定输出1/8最大额定功率表。
> ![PSU输出1/8最大额定功率表](https://img-blog.csdnimg.cn/20190620105650371.PNG?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JlbnRlbmdkZW1h,size_16,color_FFFFFF,t_70)
