> **导语**：此为个人对I2C总线协议的一点总结，如若想深度了解I2C总线协议可参考相关文档
> * [**I2C英文权威手册**](http://www.nxp.com/documents/user_manual/UM10204.pdf)
> * [**I2C中文文档**](http://download.csdn.net/detail/u011137631/9725420)

## I2C总线实用性
* **简单的双向两线总线，硬件效益最大、电路简单、硬件兼容性好。**
* **真正的多主机总线，数十个IC可连接在一路I2C总线接口上通信。**

## I2C接口电路结构

![**标准模式器件和快速模式器件连接到 I2C 总线**](http://upload-images.jianshu.io/upload_images/4859654-8654aeb3ce50431b.PNG?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

> * 一个OD门输出和一个BUFFER门输入连接到一起构成了SDA或者SCL接口。
> * I2C协议规定这种与门结构构成的电路总线空闲时电平为高电平。

## I2C通信遵循的协议
> **两根线的I2C总线如何使上面的多主机实现正常通信使用以下方式来解决**
> * **时钟同步的握手机制**，可以使主、从机器通信数据正确。
> * **仲裁机制**，可以使任意时刻只有一个主机获得I2C总线使用权。
> * **从机地址码**，给了通信时寻找指定设备的钥匙。

### 1、SCL高电平期间，SDA值稳定的情况
* **正常传输数据期间SCL高电平期间SDA数据保持稳定，SCL低电平期间，SDA数据允许修改，如下图。**

![**I2C总线的位传输**](http://upload-images.jianshu.io/upload_images/4859654-db536a9ed70f7f44.PNG?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 2、SCL高电平期间，SDA值变化的几种情况
#### ①起始和停止条件产生时

![起始和停止条件时序](http://upload-images.jianshu.io/upload_images/4859654-cc375f2ea46b0b74.PNG?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### ②两主机仲裁时
* 当 SCL 线是高电平时，仲裁在 SDA 线发生，这样，在其他主机发送低电平时，发送高电平的主机将断开它的数据输出级，因为总线上的电平与它自己的电平不相同；
* 下图显示了两个主机的仲裁过程，当然，可能包含更多的内容 （由连接到总线的主机数量决定 ）此时，产生 DATA1 的主机的内部数据电平与 SDA 线的实际电平有一些差别，如果关断数据输出，这就意味着总线连接了一个高输出电平，这不会影响由赢得仲裁的主机初始化的数据传输。
![两主机仲裁](http://upload-images.jianshu.io/upload_images/4859654-980b002d9f39ba9a.PNG?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 3、时钟同步和应答
>I2C总线上有许多主机，每一个时钟频率可能不一样。在某一个主机发起并获得I2C总线通信时需要有时钟同步机制

#### ①仲裁过程中的时钟同步

![仲裁过程中的时钟同步](http://upload-images.jianshu.io/upload_images/4859654-af603811efdb2edd.PNG?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
* 所有主机在 SCL 线上产生它们自己的时钟来传输 I2C 总线上的报文。数据只在时钟的高电平周期有效。因此 需要一个确定的时钟进行逐位仲裁。
* 时钟同步通过线与连接 I2C 接口到 SCL 线来执行，这就是说 SCL 线的高到低切换会使器件开始数它们的低电平周期，而且一旦器件的时钟变低电平， 它会使 SCL 线保持这种状态直到到达时钟的高电平 （见
图 ）。但是 如果另一个时钟仍处于低电平周期，这个时钟的低到高切换不会改变 SCL 线的状态。 因此SCL 线被有最长低电平周期的器件保持低电平。此时，低电平周期短的器件会进入高电平的等待状态。
* 当所有有关的器件数完了它们的低电平周期后，时钟线被释放并变成高电平之后，器件时钟和 SCL线的状态没有差别，而且所有器件会开始数它们的高电平周期 首先完成高电平周期的器件会再次将 SCL线拉低。
* 这样，产生的同步 SCL 时钟的低电平周期由低电平时钟周期最长的器件决定，而高电平周期由高电平时钟周期最短的器件决定。

#### ②应答机制
* **每次传输完一个8位的字节后，I2C上需要产生一个应答信号（SCL高电平，SDA低电平）。**
![I2C总线相应](http://upload-images.jianshu.io/upload_images/4859654-590bf91f08b7ea20.PNG?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### ③从机地址格式
![**7位从机地址格式**](http://upload-images.jianshu.io/upload_images/4859654-531dfad16f81b4c9.PNG?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
* 数据的传输遵循图所示的格式，在起始条件 （S） 后，发送了一个从机地址。这个地址共有 7 位，紧接着的第 8 位是数据方向位 （R/ W）-- 0 表示发送（写），1 表示请求数据 （读）。
* 数据传输一般由主机产生的停止位（P）终止。但是，如果主机仍希望在总线上通讯，它可以产生重复起始条件（Sr）和寻址另一个从机，而不是首先产生一个停止条件。在这种传输中，可能有不同的读 写格式结合。
![主机 发送器用 7 位地址寻址从机接收器 传输方向不变](http://upload-images.jianshu.io/upload_images/4859654-0969f6e211535365.PNG?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![在第一个字节后主机立即读从机](http://upload-images.jianshu.io/upload_images/4859654-da81676516c654b2.PNG?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![复合格式](http://upload-images.jianshu.io/upload_images/4859654-d547a30386667dd5.PNG?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
