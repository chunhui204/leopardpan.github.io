文本检测和一般目标检测的不同文本线是一个sequence（字符、字符的一部分、多字符组成的一个sequence），而不是一般目标检测中只有一个独立的目标。
这既是优势，也是难点。优势体现在同一文本线上不同字符可以互相利用上下文，可以用sequence的方法比如RNN来表示。难点体现在要检测出一个完整的文本线，
同一文本线上不同字符可能差异大，距离远，要作为一个整体检测出来难度比单个目标更大。（字符与背景物体很像的话，容易迷惑网络使框的尺寸不能完全覆盖文本行或尺寸过大）我认为难点还体现在如果以文本块为检测目标的话，还要分离出文本行，
这个分离的方法很难设计；如果直接以文本行为检测对象，不仅向上面说的同一文本行内方差比较大难以把框预测全，而且由于倾斜或弯曲，遮挡，小目标，复杂背景等目标检测的通用难点，
检测难度也比文本块检测要大。

****
有一点思路：能不能用STN网络代替RPN做候选区域提取的工作，如果能在保证recall下降很小的情况下大幅度降低候选框的数量，就能解决frcnn的速度瓶颈，
而且他的图像整形的功能你能降低rcnn的分类难度，唯一担心的是STN能否保证较高的recall。不知道把localization network高的复杂些能否解决这个问题。
****

# STN-OCR: A single Neural Network for Text Detection and Text Recognition

![](/images/STNT1.PNG)

在检测网络中，首先resnet变体，然后平均池化后接BLSTM（考虑到文本是序列结构为了使用字符间的上下文信息），作为STN的定位网络。由于STN只能预测一个感兴趣区域，所以本文使用了N（文中没说明）个STN，但有个疑问，STN能否利用ground truth bbox的监督信息？每个STN的ROI生成是独立的，很容易造成多个STN检测了同一个字符区域，但是有的字符并未覆盖到，即使增加STN数目也起不到作用，如何保证STN能将所有字符区域都检测出来？再就是N的数目是固定的，过少会造成漏检过多计算量会增加，所以能否根据特定图片决定N，这也是作者展望的部分。

识别网络使用的是resnet网络。

对端到端的检测识别网络，训练是非常困难的，很难协调两部分是模型收敛。作者在没有使用预训练模型的情况下检测结果多于两行，分不开，这就说明检测网络优化的很差。本文可以看做吧STN用于文本检测的尝试，是一种很好的思路。

利用BLSTM的序列化处理以及STN的整形作用的文本识别：Robust Scene Text Recognition with Automatic Rectification