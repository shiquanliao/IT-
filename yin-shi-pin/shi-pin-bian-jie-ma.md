# 视频编解码

| 格式 | 亮度分辨率 |
| :--- | :--- |
| SQCIF | 128x96 |
| QCIF | 176x144 |
| CIF | 352x288 |
| 4CIF | 704x576 |
| SD | 720x576 |
| HD | 1280x720 |

### 帧和场

一帧图像包含两场: 顶层, 底场

> **逐行图像是指：**一帧图像的两场在同一时间得到，ttop=tbot。 **隔行图像是指：**一帧图像的两场在不同时间得到， ttop≠tbot。

### 编码层次的组成

* 序列（Sequence）
* 图像组（Group of Pictures，GOP）
* 图像（Picture）
* 条带（Slice）
* 宏块（Macroblock，MB）
* 块\(Block\)

> P帧：需要参考前面的一个 **I帧或者P帧**预测。
>
> B帧：需要参考前面一个**I帧或者P帧**以及后面的一个**P帧**进行预测。

{% hint style="warning" %}
IDR帧和I帧的区别

 IDR帧的全称为（Instantaneous decoding refresh picture）,因为H264采用了**多帧预测**，所以I帧之后的P帧可能会参考I帧之前的帧_，这就使得在随机访问的时候不能以找到I帧作为参考条件_。

而IDR帧就是一种特殊的I帧。
{% endhint %}

### PTS和DTS

DTS\(Decoding Time stamp\): 主要是用在**AVPackage**中（用于编码）

PTS\(Presentation Time stamp\): 主要是用在**AVFrame**中（用于显示）

### GOP

两个I帧之间形成的一组图片，就是GOP（Group Of Picture）。通常在设置参数的时候，必须要设置gop\_size的值，**代表的是两个I帧之间的帧数目**。

I帧的压缩率为7，P帧的压缩率为20，B帧的压缩率为50.





### 视频编解码关键技术

* 预测：通过帧内预测和帧间预测降低视频图像的空间冗余和时间冗余。
* 变换：通过从时域到频域的变换，去除相邻数据之间的相关性，即去除空间冗余。
* 量化：通过用更粗糙的数据表示精细的数据来降低编码的数据量，或者通过去除人眼不敏感的信息来降低编码数据量。
* 扫描：将二维变换量化数据重新组织成一维的数据序列。
* 熵编码：根据待编码数据的概率特性减少编码冗余。

**Y\(luma\), U\(chroma blue\), V\(chroma red\)**

