flv ->  ISO BMFF (mp4 format) -> media source extension


## iso base media file format :
定义了一种用于基于时间的多媒体文件(比如视频和音频)的通用结构. 它被设计为可伸缩, 可拓展的格式, 呈现形式可能是本地或通过网络或其他的流式分发机制. 文件的格式被设计为与任何特定的网络协议相独立的, 以对它们提供通用的支持. 它被用做其他媒体文件格式的基础(比如mp4)

iso/iec base media file format 是直接基于苹果的quickTime container format. 第一个mp4文件格式规范就是在2001年发布的quicktime format 规范的基础上创造的. mp4有名 iso/iec 14496-14, 后来mp4文件格式被通用化为iso base media file format (iso/iec 14496-12:2004 或 iso/iec 15444-12:2004)

### 技术细节

iso/iec base media file format 包含时序, 结构和媒体信息以支持时序媒体数据, 比如音视频展示. 文件结构是面向对象的. 一个文件可以被简单的分解为基础的对象. 而这些对象的结构取决于它们的类型.

遵守这个格式的文件由一系列被称为“boxwes"的对象组成. 所有的数据都被包含在boxes中, 除此之外文件中不包含其他文件. box是一个由唯一类型标识符和长度定义的面向对象的结构块. 它在有些规范中被称为原子

一个动作序列(presentation or motion sequence)可能被多个文件包含. 所有的timing and framing(position and size)信息必须在iso base media file中. 其他的辅助文件可以使用任何格式.

### file type box 
为了识别一个基于 iso/iec base media file format 的文件遵守的规范, brands 被用来做文件格式中的标识符. 它们被设置在一个名为file type box(ftyp)的box里.这个box必须放在文件的起始处. brand可以指明使用的编码类型, 每种类型的编码数据是如何存储的, 文件应用的约束和拓展, 文件的兼容性和意图. brands是四个可打印字符. 一个file type box 包含两种brands, 一个是主brand,指明文件的最佳使用规范, 主brand附带有副brand,它是4字节的整数标示子版本号. 另一种是‘compatible-brands’ 兼容brand, 指明该文件兼容的多种规范. 每个文件都应该包含file type box, 但有时因为和早期规范的兼容原因, 文件可能遵循iso base media file format 但不包含file type box, 这时候他们要被当作有一个默认的ftyp: mp41

一个基于该规范的多媒体文件结构可能和多个具体的规范兼容

流
提示轨, 不同协议提示轨不同, 媒体数据本身不需要重新格式化, 

## iso bmff 字节流格式

这个规范定义了一个基于iso base media file format 的 media source extensions 字节流格式规范

## media source extensions



# YUV(YC_bC_r)

人类视觉系统（HDV）对亮度比彩色更敏感，因此可以把亮度信息从彩色信息分离处来，并使之具有更高的清晰度，彩色信息的清晰度较低些，可显著压缩带宽，实现视频压缩的一部分，人的感觉却没有不同。
如果亮度分量用Y表示，色度用Cb，Cr表示，则由大量实验得出：
Y = .299R + .587G + .114B
C_b = .564(B - Y)
C_r = .713(R - Y)

反之, RGB则是
R = Y + 1.402C_r
G = Y - .334C_b - .714C_r
B = Y + 1.772C_b

# mpeg-4 
mpeg-4标准主要包含音视频对象编码工具集和编码对象句法语言两个部分

mpeg-4标准的编码基于对象
mpeg-4具有很好的拓展性, 可进行时域和空域的拓展.
mpeg-4有多种算法, 可根据需要选择, 例如区域编码有DCT, SADCT, OWT等.

mpeg-4 为支持高效压缩, 基于内容交互和基于内容分级拓展, 以基于内容的方式表示视频数据, 引入AVO的概念实现基于内容的表示

## AVO及数据结构
AVO的基本单位是原始AV对象, 可能是一个没有背景的说话的人, 也可能是这个人的语音或背景音乐等. 它具有高效编码, 高效存储传播及可交互操作的特性.
mpeg-4就是围绕AV对象的编码,存储,传输和组合制定的. mpeg-4对AV对象的主要操作如下:
- 采用AV对象表示音视频或其组合内容
- 组合已有AV对象, 通过自然混合编码SNHC组织.
- 客队AV对象数据多路合成和同步, 以便选择合适网络传输数据
- 允许用户对AV对象进行交互操作
- 支持AV对象知识产权和保护

mpeg-4 是第一个使用户可在接收端对画面进行操作和交互访问的编码标准

对于画面:在mpeg-4校验模型中, VO主要定义为画面中分割出来的不同物体, 并由三类信息描述: 运动信息, 形状信息, 纹理信息

# h.264
mpeg和vceg已经联合开发了一个比早期研发的mpeg和h.263性能更好的视频压缩编码标准, 这就是被命名为AVC的,也被称为itu-y h.264建议和mpeg-4的第10部分的标准. 在这里, 就简称它为h.264/avc或h.264.
h.264不仅具有优异的压缩性能, 还具有良好的网络亲和性. 
和mpeg-4的重点是灵活性不同,h.264着重在压缩的高效率和传输的高可靠性

## h.264编解码器
h.264并不明确规定一个编解码器如何实现, 而是规定了一个编了码的视频比特流的句法, 和该比特流的解码方法

