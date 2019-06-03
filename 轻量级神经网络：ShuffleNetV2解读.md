## 轻量级神经网络：ShuffleNetV2解读

R.JD [CVer](javascript:void(0);) *昨天*

点击上方“**CVer**”，选择加"星标"或“置顶”

重磅干货，第一时间送达![img](https://mmbiz.qpic.cn/mmbiz_jpg/ow6przZuPIENb0m5iawutIf90N2Ub3dcPuP2KXHJvaR1Fv2FnicTuOy3KcHuIEJbd9lUyOibeXqW8tEhoJGL98qOw/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

> 作者：R.JD
>
> https://zhuanlan.zhihu.com/p/67009992
>
> 本文已授权，未经允许，不得二次转载

其实，我个人是比较偏向于工程师风格，工程导向，也许是和本科读的机械专业有关吧。这就让我养成了一个习惯：什么东西我都想知道是什么？ 为什么？ 怎么用？

最近一直在思考一个问题，大规模的神经网络和小规模的神经的网络的之间有什么区别？ 为什么有了大规模神经网络的精度后还要去研究小规模的神经网络呢？

当然是要恰饭的嘛！ 你个大网络虽然浓眉大眼精度高，但我要怎么用你？我为了用你我好要给你准备很多的存储空间，GPU，机房，散热吗？成本会很高的鸭，会没钱吃饭的鸭！没办法，我只能投入轻量级网络的怀抱里咯。

正应了这句老话：



![img](https://mmbiz.qpic.cn/mmbiz_jpg/yNnalkXE7oVzxgOkibWVOUXyq2RCcRsvhRvG9HYciard6VbGUiacJ4VeibkSMdybPkVZBszJwoWYFPMvLJGD7XmykQ/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



废话说完了，进入正题。

论文名：ShuffleNet V2: Practical Guidelines for Efficient CNN Architecture Design 论文地址：arxiv.org/abs/1807.1116 会议：ECCV2018

这篇文章来自来自旷视和清华研究组，被ECCV2018所收录。为什么直接介绍shuffleNetV2，而不是从V1开始？因为V2的效果一定是比V1好的，并且V2是从V1上发展而来，这样我们就可以既学习了V2又学习了V1的不足。对照的学习，这样我jio得效率是最高的。

论文主要分为三部分：



![img](https://mmbiz.qpic.cn/mmbiz_jpg/yNnalkXE7oVzxgOkibWVOUXyq2RCcRsvhRrazWahtnS9Hmr854GWfc4libj2iboO3icS6ZlTSOUDnCZ5Ho79VwBD5A/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



首先谈一谈目前模型用的指标FLOPs，之后讨论了一下高效网络设计的设计准则，最后在shuffleNetV1上运用设计准则设计出shuffleNetV2。

## 摘要

【摘要】目前，网络架构设计主要由计算复杂度的间接度量（如FLOPs）测量。然而，直接度量（例如，速度）还取决于诸如存储器访问成本和平台特性的其他因素。因此，这项工作建议评估目标平台上的直接度量，而不仅仅考虑FLOPs。基于一系列对照实验，这项工作为高效的网络设计提供了几个实用指南。此外，提出了一种称为ShuffleNetV2的新架构。利用消融实验验证了我们的模型在速度和准确度方面是最好的。

## Section 1 FLOPs?

### 什么是FLOPs

首先，先介绍一下FLOPs。



![img](https://mmbiz.qpic.cn/mmbiz_jpg/yNnalkXE7oVzxgOkibWVOUXyq2RCcRsvh3Z77w44BBfGia9HiaibpCqSluMJRPOUqfibA5V2ChI6MG0czPxfoTjGhPw/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



FLOPS： 全大写，指每秒浮点运算次数，可以理解为计算的速度。是衡量硬件性能的一个指标。（硬件）

FLOPs： s小写，指浮点运算数，理解为计算量。可以用来衡量算法/模型的复杂度。（模型） 在论文中常用GFLOPs（1 GFLOPs = 10^9 FLOPs）

2017年，ICIR收录了NVIDIA发表的一篇论文，在这篇论文的附录里，NVIDIA详细的讲解了一下FLOPs的计算。



![img](https://mmbiz.qpic.cn/mmbiz_jpg/yNnalkXE7oVzxgOkibWVOUXyq2RCcRsvhCC1b3bG6ohOnhOY3ydZeCtI5R6iaL0upLzONXmD12wndCaauibL5UyZw/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



对于卷积层来说：



![img](https://mmbiz.qpic.cn/mmbiz_jpg/yNnalkXE7oVzxgOkibWVOUXyq2RCcRsvhQ13pcdqFCXorIbUiakW2ZbTZfgUSBNxKLbBP5lcbA2ccgBRt9ocHic8Q/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



有人将其简化了一下：



![img](https://mmbiz.qpic.cn/mmbiz_jpg/yNnalkXE7oVzxgOkibWVOUXyq2RCcRsvhlXOhpicdbTD2YaCRic63IorskwiaRpib9Zl4KV5IdS79WEMETXB2AkQWPA/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



感兴趣的同学可以移步CNN模型所需的计算力（flops）和参数（parameters）数量是怎么计算的？ - 知乎

https://www.zhihu.com/question/65305385

在此推荐一个神器（pytorch）：torchstat

https://github.com/Swall0w/torchstat

可以用来计算pytorch构建的网络的参数，空间大小，MAdd，FLOPs等指标，简单好用。

比如：我想知道alexnet的网络的一些参数。

只需要：

```
from torchstat import stat
import torchvision.models as models

model = model.alexnet()
stat(model, (3, 224, 224))
```

就能得到结果啦：



![img](https://mmbiz.qpic.cn/mmbiz_jpg/yNnalkXE7oVzxgOkibWVOUXyq2RCcRsvh29H5WQylYAsCD1WosjCMh9fYQXqcgsaVDlskCqGY5aWjygPFIfk81w/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



可以看到每一层都有结果！兼职是神器呀。

再附上一个常用网络的参数：



![img](https://mmbiz.qpic.cn/mmbiz_jpg/yNnalkXE7oVzxgOkibWVOUXyq2RCcRsvhIlDkjyWRcsjYOSrzsZSY85ibXC2AfbtlibhhzcR0hicD3h5pJSHmQr0BQ/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



来源：https://github.com/sovrasov/flops-counter.pytorch

以及：

![img](https://mmbiz.qpic.cn/mmbiz_jpg/yNnalkXE7oVzxgOkibWVOUXyq2RCcRsvhBLM04pJaEeoFYr3pO5cibibj1nCwCHZTpAW0hDgnx1ZibulyHibvic3cFVQ/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



跑的有点远，收！



![img](https://mmbiz.qpic.cn/mmbiz_png/yNnalkXE7oVzxgOkibWVOUXyq2RCcRsvhrRgesQ4Bq8qQtibEQe1WFcFqL2d290iaTZSAyYwzzLYVVltNsfRbdUFA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



作者认为FLOPs是一种简介的测量指标，是一个近似值，并不是我们真正关心的。我们需要的是直接的指标，比如速度和延迟。



![img](https://mmbiz.qpic.cn/mmbiz_jpg/yNnalkXE7oVzxgOkibWVOUXyq2RCcRsvhIccNx7UNCqN2qxZDIFrQqd6z7UkL1AG8x03Botib1ibR1R9Uy2qYS8Jw/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



图c是经典小网络在GPU上的MFLOPs与速度（Batches/sec）的关系。 图d是经典小网络在ARM上的MFLOPs与速度（Batches/sec）的关系。

我们可以发现，具有相似的FLOPs的网络，执行的速度却不一样。有的相差还挺大。

作者在此提出了第一个观点：因此，使用FLOP作为计算复杂度的唯一指标是不充分的。

### 为什么不能只用FLOPs作为指标呢？

作者认为有如下几个原因：

1) FLOPs没有考虑几个对速度有相当大影响的重要因素。 2）计算平台的不同。

### 1) FLOPs没有考虑几个对速度有相当大影响的重要因素

MAC和并行度

### MAC

比如：MAC(内存访问成本)，计算机在进行计算时候要加载到缓存中，然后再计算，这个加载过程是需要时间的。其中，分组卷积（group convolution）是对MAC消耗比较多的操作。

什么是分组卷积？

### group convolution 分组卷积

分组卷积最早是出现在AlexNet，当时这么做是为了解决显存不够的问题。AlexNet使用的GPU型号是NVIDIA GTX 580，只有1.5GB的显存可以用，但是这个模型需要3GB的RAM才能进行训练。



![img](https://mmbiz.qpic.cn/mmbiz_jpg/yNnalkXE7oVzxgOkibWVOUXyq2RCcRsvhPQEc5629m25ZrLibKsxvL1fw64W2cdbDqGwbrsNw0Z2CicauxZ26XjQw/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



可以看到，AlexNet有两个部分，上面是在一个GPU上操作的，下面是在另外一个GPU上的操作，最后再进行特征融合。将两部分的卷积核组进行可视化，得到如下图：



![img](https://mmbiz.qpic.cn/mmbiz_jpg/yNnalkXE7oVzxgOkibWVOUXyq2RCcRsvhdyAtYsc9dPZ3UWPibicF3bbZUpI8KuBoAX1GhcHqlKICKxLM1HpibQsfw/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



AlexNet也指出，不管怎么样学习。最后卷积核组始终会分成两个独立的任务，上面是黑白过滤器，下面则是彩色过滤器

我们一般的卷积操作如下图，卷积会对输入数据的整体一起做卷积操作，即输入数据：H1×W1×C1；而卷积核大小为h1×w1，一共有C2个，然后卷积得到的输出数据就是H2×W2×C2。*这里假设输出和输出的WH不变*。



![img](https://mmbiz.qpic.cn/mmbiz_jpg/yNnalkXE7oVzxgOkibWVOUXyq2RCcRsvhtttjVpSzBej705RcIdrKEw50loywic36m0R0zrBeKibVCN5Pd9NHJkkQ/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



但是，当我门对一般的卷积进行“切割”之后，就是分组卷积了（如下图）。



![img](https://mmbiz.qpic.cn/mmbiz_jpg/yNnalkXE7oVzxgOkibWVOUXyq2RCcRsvhAvoWvbCDzefMnicbmdRicK50LglUgnzHCiayjUSznDQoo8WFWLYgggibhw/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



输入数据被分成了2个group（g=2），卷积核(filters)被分成了2个group,每一个group都只有原来一半的feature map。用每组的卷积核同它们对应组内的输入数据进行卷积操作，得到了两组不同的输出数据，再用通道合并（concatenate*首尾相接*）的方式组合起来，最终的输出数据的通道仍是C2。需要注意的是，这种分组只是在Channel上进行划分，即某几个通道编为一组。

也就是说，分组数g决定以后，就将并行的运算g个相同的卷积过程。每个过程也就是每组，输入数据为H1×W1×C1/g，卷积核大小为h1×w1×C1/g，一共有C2/g个，输出数据为H2×W2×C2/g。

> 总之，Group convolution是一种卷积操作，先切分channel，然后分组卷积，运算上没有什么特别的地方，只是单纯的通道分组处理，降低复杂度。

这样就可以带来一个好处：降低参数

我们用`C1×K×K×C2` 近似的计算参数量。



![img](https://mmbiz.qpic.cn/mmbiz_jpg/yNnalkXE7oVzxgOkibWVOUXyq2RCcRsvhvTY8UTNEQ7vuX0dg6PicLhNrUr29jdkRoEeLY3IyNyQianzQRIw7Oib2w/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



假设输入通道为256，输出通道也为256，kernel size为3×3，一般卷积参数为256×3×3×256。

分组卷积之后的参数量如下图：



![img](https://mmbiz.qpic.cn/mmbiz_jpg/yNnalkXE7oVzxgOkibWVOUXyq2RCcRsvhyx6awBpZ7qISOzGc6BGWWYZdSxKUcgWiaxOI7fFJ5n1bt9KsiaUddGoA/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



实施分组卷积时（g=8），每个组的输入channel和输出channel均为32（256/8），参数为8×32×3×3×32，是原来的八分之一。

什么分组卷积会消耗MCA？

### 并行度

第二个对速度有相当大影响的重要因素就是模型的并行度。

在相同的FLOPs下，具有高并行度的模型可能比具有低并行度的另一个模型快得多。如果网络的并行度较高，那么速度就会有显著的提升。

### 2）计算平台的不同

不同的运行平台，得到的FLOPs也不相同。有的平台会对操作进行优化，比如：cudnn加强了对3×3conv计算的优化。这样一来，不同平台的FLOPs大小确实没有什么说服力。

### 效率对比准则

通过以上的分析，作者提出了2个网络执行效率对比的设计准则：

1 使用直接度量方式如速度代替FLOPs。

2 要在目标计算平台上计算，不然结果不具有代表性。

## Section 2 高效网络设计实用指南

在这一节中，作者首先分析了两个具有代表性的最先进网络的运行时性能（ShuffleNetV1，MobileNetV2）。然后，推出了四个高效网络设计指南，*这些指南不仅仅考虑了FLOP*。

### 分析ShuffleNetV1，MobileNetV2

作者分别在GPU和CPU上去对ShuffleNetV1，MobileNetV2的运行时间进行了测试。



![img](https://mmbiz.qpic.cn/mmbiz_jpg/yNnalkXE7oVzxgOkibWVOUXyq2RCcRsvh7lv51Y6CGicvWeiaHAGcIlib3IoC4ictsdp8NMtDLX0DC4Z0kiaiau7SAa4g/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



- GPU使用单个NVIDIA GeForce GTX 1080Ti。卷积库是CUDNN 7.0。
- CPU使用高通骁龙 810.

测试结果如下：



![img](https://mmbiz.qpic.cn/mmbiz_jpg/yNnalkXE7oVzxgOkibWVOUXyq2RCcRsvh8rMTfia075Wf6SuB8EiamjHvriaPryeeibqZpiaBiceUcnr6gGSIRHB3TAFg/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



可以看到，整个运行时被分解用于不同的操作。处理器在运算的时候，不光只是进行卷积运算，也在进行其他的运算，特别是在GPU上，卷积运算只占了运算时间的一般左右。



![img](https://mmbiz.qpic.cn/mmbiz_jpg/yNnalkXE7oVzxgOkibWVOUXyq2RCcRsvhnVpMnDUkeJXU3d8fgicageWJVGn8cD1zuVRTGqDFqXhhRL5St3ibwyibg/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



我们将卷积部分认为是FLOPs操作。虽然这部分消耗的时间最多，但其他操作包括数据IO，数据混洗和逐元素操作（AddTensor，ReLU等）也占用了相当多的时间。因此，再次确认了模型使用FLOPs指标对实际运行时间的估计不够准确。

基于这一观察，作者从几个不同的方面对运行时间（或速度）进行了详细分析，并为高效的网络架构设计提出了几个实用指南。

### 四个高效网络设计指南

### G1 输入输出具有相同channel的时候，内存消耗是最小的

由于目前主流网络均使用的depthwise separable convolutions，其中pointwise convolution（同上）（如1×1卷积）占据了很大一块的运算量（dwc和pwc之后写文章详述，在此就不说了）。所以作者以1×1卷积为例,假设feature map的大小为h×w输入输出channel分别为c1和c2,那么：



![img](https://mmbiz.qpic.cn/mmbiz_jpg/yNnalkXE7oVzxgOkibWVOUXyq2RCcRsvhzicarG2pv38DJrh8bIRiaUyba4k2ibdwgibmBQmLohBstQHBmwez8blLzw/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



由均值不等式，我们可得：



![img](https://mmbiz.qpic.cn/mmbiz_jpg/yNnalkXE7oVzxgOkibWVOUXyq2RCcRsvhA1tuTYXJpCCkhhGMD04CAz5BDyYibiaCPicnJYx0kk8yKuhcXNh1CTjWQ/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



当 c1 = c2 时取得最小值。

来人，上实验：

![img](https://mmbiz.qpic.cn/mmbiz_jpg/yNnalkXE7oVzxgOkibWVOUXyq2RCcRsvhZg1eMKbA1Btnicia6uiaOHnO2u9Je3icYnSGjOlOrCK9aW6MpLwsKGVu7w/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

为了验证这一个想法，作者设计了这个实验。实验网络是由10个block堆叠组成，每个block包含2个1×1卷积层，第一个卷积层的输入输出通道分别是c1和c2，第二个卷积层相反（c2，c1）。4行结果分别表示不同的c1:c2比例，但是每种比例的FLOPs都是相同的。

可以看出当c1和c2比例越接近时，速度越快，尤其是在c1:c2比例为1:1时速度最快。这与G1所提出的当c1和c2相等时MAC达到最小值相所对应。

### G2 过多的分组卷积操作会增大MAC，从而使模型速度变慢

之前有提到，分组卷积操作会减少参数，这样一来网络的计算量也就减少了。但是呢，认为网络的计算量减少，不代表模型的速度也会减少。MAC主要的消耗来源就来自分组卷积，分组卷积一多，MAC消耗的越多，模型速度也就变慢了。

和前面同理，带group操作的1×1卷积的FLOPs如下所示，多了一个除数g，g表示分组数量。这是因为每个卷积核都只和c1/g个通道的输入特征做卷积。



![img](https://mmbiz.qpic.cn/mmbiz_jpg/yNnalkXE7oVzxgOkibWVOUXyq2RCcRsvhGp4iaoBWWw75iaB4MrTX2qIhIxIrPM47sPRbKSgOLZ2CcWGUBEicfLbIQ/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



MAC计算同理，和前面不同的是这里卷积核的存储量除g。

MAC和B之间的关系如下：



![img](https://mmbiz.qpic.cn/mmbiz_jpg/yNnalkXE7oVzxgOkibWVOUXyq2RCcRsvhQYiauN8Q30FZXd8uZgpEyvyfu1ictYxnBUSBgp66eE9lUXuk6rsia8fkA/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



可以看出在B不变时，g越大，MAC也越大。

上实验！



![img](https://mmbiz.qpic.cn/mmbiz_jpg/yNnalkXE7oVzxgOkibWVOUXyq2RCcRsvhnDmfcBeVucCqhf0iaA3rKuSjxkc1sgJqnEnh76SCgPvicOnVJCDOEbWw/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



很清楚的看到，g越小，速度越快。因此，作者建议应根据目标平台和任务仔细选择组号。虽然组卷积能增加模型的准确度，但是作者认为盲目使用较大的组号是不明智的，因为这将会使得计算成本增加带来的缺点大于准确度增加带来的优点。

### G3 模型中的分支数量越少，模型速度越快

作者认为，模型中的网络结构太复杂（分支和基本单元过多）会降低网络的并行程度，模型速度越慢。

文章用了一个词：fragment，翻译过来就是分裂的意思，可以简单理解为网络的单元或者支路数量。



![img](https://mmbiz.qpic.cn/mmbiz_jpg/yNnalkXE7oVzxgOkibWVOUXyq2RCcRsvhayic3ibQAW3ibQkhH0KlUkCMzppZlnmvM0TlugNriax3jfHJUspEV6Yicaw/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



为了研究fragment对模型速度的影响，作者做了第三个实验。具体地，每个模型由block组成，每个block由1到卷积组成，分别将它们重复10次。



![img](https://mmbiz.qpic.cn/mmbiz_jpg/yNnalkXE7oVzxgOkibWVOUXyq2RCcRsvhyBRlZQf3Nd4ZjvM216zj3kwAd432cHmm6ebcMa0CGibQkwZlXYICoqg/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



得到的结果如下：



![img](https://mmbiz.qpic.cn/mmbiz_jpg/yNnalkXE7oVzxgOkibWVOUXyq2RCcRsvhib6FVMYOJzqxOoMKLpZcicPzicZo0WnkBt6KmOwNZUcicECGlibtaWtfCzA/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



其中， 2-fragment-series表示一个block中有2个卷积层串行，也就是简单的叠加； 4-fragment-parallel表示一个block中有4个卷积层并行，类似Inception的整体设计。 可以看出在相同FLOPs的情况下，单卷积层（1-fragment）的速度最快。

### G4 Element-wise操作不能被忽略

Element-wise包括Add/Relu/short-cut/depthwise convolution等操作。

再回到之前的运行时间图：



![img](https://mmbiz.qpic.cn/mmbiz_jpg/yNnalkXE7oVzxgOkibWVOUXyq2RCcRsvhNtXyxypgdNtF0OfRFxVJLtYvGjriaT7NJIK5S3r3GsfqdLEsYUic8crA/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



元素操作类型操作虽然FLOPs非常低，但是带来的时间消耗还是非常明显的，尤其是在GPU上。元素操作操作虽然基本上不增加FLOPs，但是所带来的时间消耗占比却不可忽视。也即Small FLOPs heavy MAC。

于是作者做了一个实验，采用的是Resnet50的瓶颈结构（bottleneck），除去跨层链接shortcut和 ReLU：







得到结果如下：







可以看到，在 GPU 和 ARM 结构上都获得了接近 20% 的提速。

### 总结设计指南

把作者提出的四个高效网络设计指南再总结一下：

（1） 卷积层使用相同的输入输出通道数。

（2） 注意到使用大的分组数所带来的坏处。

（3） 减少分支以及所包含的基本单元。

（4） 减少Element-wise操作。

## Section 3 ShuffleNet V2

终于到了最后的部分了！

这一部分，作者根据之前所提出的设计指南，在shufflenetV1的基础上进行修改，得到了shuffleNetV2。首先，简要的回顾一下V1，看看V1的网络结构中有什么地方违反了四条设计指南，再进行修改。

### 回顾ShuﬄeNet v1

shuffleNet在resnext单元基础上进行改进。



![img](https://mmbiz.qpic.cn/mmbiz_jpg/yNnalkXE7oVzxgOkibWVOUXyq2RCcRsvh6cy4ib4NJfKvqT7S0BTicEiaDOb7rtSQ0wRV0liaMdbLIZIiaAnltw4aGnA/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



shuffleNet主要拥有两个创新点：

（1）pointwise group convolution 逐点分组卷积

（2）channel shuffle

原因：

1。逐点卷积占了很大的计算量————> 逐点分组卷积

2。不同组之间特征通信问题 ————> channel shuffle

逐点组卷积，就是带分组的卷积核为1×1的卷积，也就是说逐点组卷积是卷积核为1×1的分组卷积。

### 什么是channel shuffle?

分组会导致信息的丢失，为了解决这个问题，shuffleNetV1给出的方法就是交换通道，如下图：







因为在同一组中不同的通道蕴含的信息可能是相同的，如果不进行通道交换的话，学出来的特征会非常局限。如果在不同的组之后交换一些通道，那么就能交换信息，使得各个组的信息更丰富，能提取到的特征自然就更多，这样是有利于得到更好的结果。如图c所示，每组中都有其他所有组的特征。







于是shuffleNetV1从最初的a到了b。首先用带group的1×1卷积代替原来的1×1卷积，同时跟一个channel shuffle操作。然后是3×3 dwc，然后去掉了两个ReLU层，这个在Xception和mobileNetV2中有所介绍。

### V1有何不妥？



![img](https://mmbiz.qpic.cn/mmbiz_jpg/yNnalkXE7oVzxgOkibWVOUXyq2RCcRsvhQsIo2VbDNt1H9ICiaKjscwlq9Lia7c7qH8T9S6ias8hAXvQCFt2ofo0vA/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



如Section 2所述，逐点组卷积增加了MAC违背了G2。这个成本不可忽视，特别是对于轻量级模型。另外，使用太多分组也违背了 G3。



![img](https://mmbiz.qpic.cn/mmbiz_jpg/yNnalkXE7oVzxgOkibWVOUXyq2RCcRsvhe1PPUcweSdOFhsqXV7H3iaCEUibWIPUZiaPWFEogicib0a1AgfNPMIMQibHA/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



瓶颈结构违背了G1与多单位违背了G3。



![img](https://mmbiz.qpic.cn/mmbiz_jpg/yNnalkXE7oVzxgOkibWVOUXyq2RCcRsvh4WNldArDrfuELKOQyKJAgzDhmgwibLJVPjxzxIlyqd8ZbBicXHzvbCtA/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



为了实现较高的模型容量和效率，关键问题是如何保持大量且同样宽的通道，既没有密集卷积也没有太多的分组Add操作是元素级加法操作也不可取违反了G4。因此，为了实现较高的模型容量和效率，关键问题是如何保持大量且同样宽的通道，既没有密集卷积也没有太多的分组。

为此，作者引入了channel split

### shuffleNetV2



![img](https://mmbiz.qpic.cn/mmbiz_jpg/yNnalkXE7oVzxgOkibWVOUXyq2RCcRsvhUz5HEkgoKIEw0UIXRybK6HUFrNAFo9eOOiaNx0ooXDCRIbOF5quzskw/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



左图是V1，右图是V2。

1.在每个单元的开始，通过Channel split将c特征通道的输入被分为两支，分别带有 c−c' 和c'个通道。按照准则G3，一个分支的结构仍然保持不变。另一个分支由三个卷积组成， 为满足G1，令输入和输出通道相同。与 ShuffleNet V1 不同的是，两个 1×1 卷积不再是组卷积(GConv)，因为Channel Split分割操作已经产生了两个组。

2.卷积之后，把两个分支拼接(Concat)起来，从而通道数量保持不变 (G1)，而且也没有Add操作（element-wise操作）（G4）。然后进行与ShuffleNetV1相同的Channel Shuﬄe操作来保证两个分支间能进行信息交流。

3.depthwise convolution保留

上述构建模块被重复堆叠以构建整个网络，被称之为 ShuﬄeNet V2。基于上述分析，本文得出结论：由于对上述四个准则的遵循，shuffleNetV2架构设计非常的高效。

V2（左）和V1（右）网络整体结构如下图：



![img](https://mmbiz.qpic.cn/mmbiz_jpg/yNnalkXE7oVzxgOkibWVOUXyq2RCcRsvhpaoSdnmk4NeAIA6uF1vF3M0or29nS3SRVjfLEibtqAxQgicxiawaOQ4Gg/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



作者在多个模型的测试对比结果如下：



![img](https://mmbiz.qpic.cn/mmbiz_jpg/yNnalkXE7oVzxgOkibWVOUXyq2RCcRsvhbxnVAIIvzvseZEnoza56WgO6LR3Oh7K9s7GCmVWLibZqZmg0v5JuBKA/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



> 从FLOPs、精度、速度上进行详细对比。实验中不少结果都和前面几点发现吻合，比如MobileNet v1在GPU上速度最快，主要使用了DW卷积以及逐点卷积，使得网络结构简单，没有太多复杂的支路结构，符合G3；IGCV2和IGCV3因为group操作较多（违反G2），所以整体速度较慢；最后的几个通过自动搜索构建的网络结构，和前面的第3点发现对应，因为支路较多，所以速度较慢，符合G3的标准，另外IGCV2与IGCV3的速度较慢主要是因为使用了较多的group。

shuffleNetV2和其他几个轻量级网络在GPU和ARM两个平台上的对比如下：



![img](https://mmbiz.qpic.cn/mmbiz_jpg/yNnalkXE7oVzxgOkibWVOUXyq2RCcRsvh5ibBSuVicibgDQSOicZRF0cGiaAyO3QE3KKDhrPM7YlBTP0hmRmz8j0dY4w/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



可以看到，速度和准确度都是最好的。

ShuﬄeNet V2 在 COCO 目标检测任务上的性能与其他小网络对比：



![img](https://mmbiz.qpic.cn/mmbiz_jpg/yNnalkXE7oVzxgOkibWVOUXyq2RCcRsvhGIZ2WEEDBsx3ydzzqs6Wxpibhd7wLiaOngsxFESqBBD5dqn8BmWdibBBw/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



最后，作者夹带了点小“私活”：



![img](https://mmbiz.qpic.cn/mmbiz_jpg/yNnalkXE7oVzxgOkibWVOUXyq2RCcRsvh44nM1X08AJvBtGvE775K65mlBsOeM0aOPVBWuEyDPazT283HC1eWvg/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

**CVer学术交流群**



扫码添加CVer助手，可申请加入**CVer-目标检测交流群、图像分割、目标跟踪、人脸检测&识别、OCR、超分辨率、SLAM、医疗影像、Re-ID和GAN等群。****一定要备注：研究方向+地点+学校/公司+昵称**（如目标检测+上海+上交+卡卡）

![img](https://mmbiz.qpic.cn/mmbiz_png/yNnalkXE7oWwGaQHYUGCaoicqoQQalGZXe6jnkb9FIicAvM7PslNXrjExITE9dAMibnWkiaTH5e5MNVMVKfiavI2ibsw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

▲长按加群



这么硬的**论文精读**，麻烦给我一个在**在看**



![img](https://mmbiz.qpic.cn/mmbiz_png/e1jmIzRpwWg3jTWCAZ4BrnvIuN20lLkhIjtg4GRSDhTk9NpeF0GGTJwUpKPatscIQU7Ndj9hgl8BPpGj2BJoFw/640?tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

▲长按关注我们

**麻烦给我一个在看****！**

[阅读原文](https://mp.weixin.qq.com/s?__biz=MzUxNjcxMjQxNg==&mid=2247489582&idx=3&sn=e61a9ea5715855815381169e01243033&chksm=f9a26aa1ced5e3b77c190b53feac278d035976855efc2f230ec824f351802c7c09ce9db87b19&mpshare=1&scene=1&srcid=&key=45382ee80ea50780c6b44eb62c42c25d33fc11d5d62309f3b5a0f7e8435c456001ff0b04ce8b5dca176c93a1ff521876d507d6319b6b1994677b703e8b3e0406fd08b89a87e8b42f79423c6bd3af7e87&ascene=1&uin=MjMzNDA2ODYyNQ%3D%3D&devicetype=Windows+10&version=62060833&lang=zh_CN&pass_ticket=%2BtDZ2Al0VM5wjz5XAzAxV1jJFwepKB91N4744YqAfvwEIleHxJyeJlLibQdxfrJN##)





微信扫一扫
关注该公众号