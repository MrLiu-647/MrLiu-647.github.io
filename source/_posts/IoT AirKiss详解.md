---
title: IoT AirKiss协议详解
date: 2018-07-11
categories:
- IoT
---

# AirKiss(飞吻)技术原理简介：
802.11 是 IEEE 制定的无线局域网协议，802.11 以 802.2 的逻辑链路控制封 装来携带 IP 封包，因此能够以 802.2 SNAP 格式接收无线网络数据。如果开启 wifi 芯片的混杂模式监听空间中的无线信号，并以 802.2 SNAP 格式从数据链路层截取数据，就会得到如下图所示的数据包:

![](https://upload-images.jianshu.io/upload_images/3407530-d968c50da7f28a8c.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



DA字段表示目标mac地址，SA字段表示源mac地址，Length字段表示后面数据的长度，LLC字段表示LLC头，SNAP字段包括3bytes的厂商代码和2bytes的协议类型标识，DATA字段为负载，对于加密信道来说是密文的，FCS字段表示帧检验序列。

从无线信号监听方的角度来说，不管无线信道有没有加密，DA、SA、Length、LLC、SNAP、FCS字段总是暴露的，因此信号监听方便有了从这6个字段获取信息的可能。但从发送方的角度来说，由于操作系统的限制(比如ISO或者Android)，DA、SA、LLC、SNAP、FCS五个字段的控制需要很高的控制权限，发送方一般是很难拿到的。***因此只剩下Length这一字段，发送方可以通过改变其所需要发送数据包的长度进行很方便的控制。***所以，只要制定出一套利用长度编码的通信协议，就可利用802.2 SNAP 数据包中的Length字段进行信息传递。

***在实际应用中，我们采用UDP广播包作为信息的载体。信息发送方向空间中发送一系列的UDP广播包，其中每一包的长度(即Length字段)都按照Air Kiss通信协议进行编码，信息接收方利用混杂模式监听空间中的无线信号，并从数据链路层截取802.2 SNAP格式数据包，便可得到已编码的Length字段，随后接收方便可根据Air Kiss通信协议解析出需要的信息。***

整个过程如下图所示：

![](https://upload-images.jianshu.io/upload_images/3407530-0d3a1a5119228d4d.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# AirKiss通信协议：

## 物理层协议：

在信号载体方面，采用wifi无线信号进行信息传递，1-14全信道支持。

在信号编码方面，802.2 SNAP 数据包中的Length字段为数据发送方唯一可控字段，因此Air Kiss通信协议利用发送数据包的长度进行编码。由于受到MTU的限制，Length字段最大可编码位数为10bit。但实际测试过程中发现，UDP包长度与丢包率、乱序率成正比。因此本协议中，我们把Length字段编码位数限制在9bit，即UDP广播包的发送长度不大于512字节。

我们身处的无线网络环境有可能及其复杂，很有可能在同一个空间中存在多个AP，而这些AP又分布在相同或者不同的信道上，这样接收者一开始是不知道发送方在1-14哪个信道上发送信息，而且同一个信道上也可能会有很多设备在发送UDP广播包。在这种情况下，接收方监听到的数据包是海量的。必须从海量的数据信息中定位出发送方所在的信道和发送方的mac地址。另外，由于在UDP广播包发送过程中，一个UDP层的数据包，要经过IP层、数据链路层的封装，并且通过加密(加密方式包括WPA2、WPA、WEP三种)后才会被发送出去，所以发送方发送UDP广播包的长度与接收方监听SNAP包中的Length字段值存在差异，这就需要进行转义。然而，由于底层加密方式的不确定性，使得这个差异值也具有不确定性。

为解决这两个问题，在发送链路层数据（见下节）之前，***需要先发送400ms的前导域（400ms = 8 * 50ms，即如果设备端以50ms的频率切换信道，则可以覆盖8个信道，因为一般用户环境不用监听14个信道，所以覆盖8个信道足已)。***前导域由4个字节组成，其值固定为｛1,2,3,4｝。接收方在接收到这些前导域数据包后，利用SNAP包中的Length字段与之相减，从而获取到这个差异值。

举个例子，接受方通过监听，在链路层截获802.2 SNAP格式的前导数据包，其Length字段的值分别为53，54，55，56，那差异值就能确定为53-1=52。之后接收方接收到数据之后都用SNAP包的Length字段值减去52，即能得到实际的信息数据。

## 链路层协议：

![](https://upload-images.jianshu.io/upload_images/3407530-70f59671837567dd.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

链路层数据结构可分为两类，**control字段**与**data字段**，magic code、prefix code、sequence header field属于control字段，data field属于data字段。control字段与data字段以第8bit位（最高位）加以区别，该位为1表示data field字段，为0表示control 字段。在control字段中，magic code字段与prefix code字段完全相同，magic code字段与sequence header字段通过第7bit位加以区分，该位为1表示sequence header字段，为0表示magic code字段。

以下分别对各个字段进行详细介绍。

- magic code字段

magic code字段的数据结构如下图所示：

![](https://upload-images.jianshu.io/upload_images/3407530-6b7ee8c342bc2568.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



magic code由4个9bits组成，每个9bits的高5位为magic code字段，低4位为information字段。前两个9bits的information字段分别装载要发送数据长度的高4位和低4位，后面两个9bits的information字段分别装载要发送ssid的crc8值的高4位和低4位。

***这里单独传输ssid的crc8字段是对整个传输过程所做的优化。在研究中我们发现，在信息传输之前先对AP进行扫描，通过获取的beacon可以得知无线环境中所有非隐藏AP的ssid、rssi以及信道。在传输过程中，接收方先从magic code field中获取目标AP ssid 的crc8值，然后再和事先扫描所得到的ssid的crc8值进行比对，如果发现相同值，那么在接下来的接收过程中接收方就不用再接收ssid信息，这就大大加快了传输的时间。***

在传输过程中，需要发送**5个magic code字段**。这里重复发送的原因是magic code中的字段很重要，接收端可以通过对比多次接收的结果来保证正确性。

- prefix code字段

prefix code字段的数据结构如下图所示：

![](https://upload-images.jianshu.io/upload_images/3407530-cd0a9c34c4207d38.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



prefix code由4个9bits组成，每个9bits的高5位为magic code字段，低4位为information字段。前两个9bits的information字段分别装载要发送密码的长度的高4位和低4位，后面两个9bits的information字段分别装载发送密码长度的crc8值的高4位和低4位。

prefix code有两个作用，首先是表示数据序列的正式开始，其次告诉接收端发送密码的长度，以便接收方在接收完数据后，对数据进行分割解密。

- sequence header字段：

![](https://upload-images.jianshu.io/upload_images/3407530-4a1a6695f32de02f.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


我们把待发送的数据以4为粒度进行划分，每4个数据组成一个sequence，以sequence为单位进行数据的发送。每个sequence都由sequence header字段和data 字段组成。最后一个sequence如果不够4个数据，不用补全。

sequence header字段由两个9bits组成，第一个的低7位装载的是从本sequence index开始到本sequence结束发送的所有数据的crc8的低7位值（计算过程中不计入字段标识位，因此sequence index最高位需补0），在接收完一个sequence的数据之后，需进行crc8值的效验，如果不相同，证明该sequence的数据接收出错，应该丢弃。

- data字段：

data字段的数据结构如下图所示：

![](https://upload-images.jianshu.io/upload_images/3407530-cd4addf7ced603bf.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

data字段由4个9bits组成，每个9bits的第8位为控制位，固定为1，其余的8位装载需要传输的数据

## 应用层协议：
送方所要发送的数据由三部分组成：密码、随机数、ssid。其中随机数的作用是，当数据接收方连上AP之后，立即发送以该随机数为内容的UDP广播包，当发送方收到该广播包后就能确认接收方已经准确接收到所有数据。密码和ssid都’\0’结尾，***并且分别用AES进行加密，再发送。***这三部分数据的发送顺序为先发送密码，再发送随机数，最后发送ssid，如下图所示：

![](https://upload-images.jianshu.io/upload_images/3407530-142c1116cd4fdb39.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



# Air Kiss 通信协议性能分析

## 纠错能力分析

Air Kiss技术中的通信模型可以抽象为错误率为0-5%的单向的信道，所需要传递信息的最大长度为68bytes。在这种情况下，如果不采用纠错算法，就很难保证在有限次数内完成信息的发送。

Air Kiss采用了累积纠错算法来保证在有限次内完成传输过程。累积纠错算法的理论基础为：多轮数据发送过程中，在同一位数据上发生错误的概率是很低的。因此可以累积多轮的数据传递结果进行分析，其中一轮中某一位错误数据有很大的概率能其它轮中找到其对应的正确值，这样就能保证在有限次内完成信息的发送。

假定需要传递信息的长度为68bytes，我们计算了在最坏的情况下，使用累积纠错算法与不使用累积纠错算法信息发送成功的概率与发送次数的关系，结果如下表所示：

![](https://upload-images.jianshu.io/upload_images/3407530-80344634bc22c04b.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 基本性能分析
Air Kiss技术的传输速率取决于信息发送方UDP广播包的发送速率，目前广播包的发送频率为5ms发送一个，因此其传输速率为200bytes/s。在不计算magic code field的情况下，负载效率为66.7%。

从纠错能力的分析可能，如果发送信息长度为最长的68bytes，那么在***最坏情况下，最多需要5次就可以完成信息发送，最大的传输时间需要2.039s。***

参考资料：
[百度文献](https://wenku.baidu.com/view/0e825981ad02de80d5d8409c.html)
[Air Kiss(飞吻)技术实现方案](https://blog.csdn.net/flyingcys/article/details/50248537)
