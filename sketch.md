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