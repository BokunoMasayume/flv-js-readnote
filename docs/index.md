# flv.js 阅读笔记

## 简介

flv.js将flv格式的视频数据转换为mp4的格式的, flv和mp4都是‘盒子格式’, 内部的具体视频和音频编码数据是不需要改变的.
因此flv.js的主要工作就是从网络中获取flv格式的视频数据, 解析参数信息, 和音视频裸流, 然后组合成mp4的格式, 通过MediaSource传递给HTMLMediaElement(e.g. video) 
flv.js 只当视频编码为h.264, 音频编码为AAC时有效, 这也是一般的情况.

## 主要功能模块划分

- 解码 demux
- 编码 remux
- 网络接收 io
- 核心 core

## 组织方式

// TODO 结构图


## 媒体数据的流通路径

配置选择的loader -> IOController -> FLVDemuxer -> MP4Remuxer -> TransmuxingController -> Transmuxer[-Worker] -> FlvPlayer -> MSEController

IOController的onLoaderChunkArrival方法注入配置选择的loader的onDataArrival方法传递数据

IOController 和 FLVDemuxer 通过把FLVDemuxer的parseChunks方法注入IOController的onDAtaArrival方法传递数据

FLVDemuxer 和 MP4Remuxer 通过把MP4Remuxer的remux方法注入FLVDemuxer的onDataAvailable方法传递数据

MP4Remuxer 和 TransmuxingController 通过把TransmuxingController的onRemuxerMediaSegmentArrival方法注入MP4Remuxer的onMediaSegment方法传递数据

TransmuxingController 和 Transmuxer通过MEDIA_SEGMENT事件(Transmuxer中定义TransmuxingController对象的监听事件)传递数据

Transmuxer 和 FlvPlayer通过MEDIA_SEGMENT事件(FlvPlayer中定义Transmuxer对象的监听事件)传递数据

FlvPlayer 和 MSEController 通过FlvPlayer调用MSEController 的appendMediaSegment方法传递数据

数据由MSEController最终送给MediaSource中, 在HTMLMediaElement中播放

## 媒体数据处理流程

IOLoader请求媒体数据,每拿到一块数据, 调用onDataArrival句柄,该句柄传来的数据经由IOController的onLoaderChunkArrival做一些缓存处理后交TransmuxingController的onInitChunkArrival句柄处理.

在onInitChunkArrival中, 通过扫描数据头部, 如发现flv文件头, **FLVDemuxer**和**MP4Remuxer**被创建, 随后IOController移交给demuxer管理, 数据由原本的从iocontroller流向transmuxingcontroller变更为由iocontroller流向demuxer的parseChunks方法处理.另外transmuxingController还会给demuxer注入error, mediaInfo, metaDataArrival, scriptDataArrived事件处理句柄, 给remuxer注入initSegment, mediaSegment事件处理句柄; 并将remuxer的muxer方法注入demuxer的dataavailable句柄, ontrackMetadataReceived注入trackMetadata句柄, 使经由demuxer处理的数据流入remuxer.

当数据通过demuxer的parseChunks方法流入demxuer后, parseChunks将数据按照flv tag分割, 并解析每个tag的header, 并根据header的信息确认每个flv tag的类型(audio, video, script)并将该tag的body部分分发给相应的处理方法处理.

- script data处理句柄: 最特殊的script data tag是flv文件的第一个tag, 它是[amf格式](./flv-format.md#SCRIPTDATA)的, 包含了音视频相关的元数据, 当接收到的script tag是这个时, demuxer调用onMetaDataArrived处理, 具体的处理方式经由transmuxingController的onMetaDataArrived和transmuxer的onMetaDataArruved最终触发FlvPlayer的METADATA_ARRIVED事件, flv.js中无flvPlayer该事件接收者, 基本就随风飘散了. 对所有的script data 处理后都会调用onScriptDataArrived方法处理,该句柄经由transmuxingController的onScriptDataArrived方法, transmuer的onScriptDataArrived方法, 最终触发flvPlayer的SCRIPTDATA_ARRIVED事件.

- audio data处理句柄: audio tag的body部分的结构为AUDIODATA, 在该结构的头部(第一个字节)可以得到音频的采样率, 声道数和编码方式(aac或mp3), 然后根据编码方式, 送给aac处理句柄和mp3处理句柄, 抽取出音频数据部分和元数据信息, 并将抽取到的音频数据推入_audioTrack队列中等待被mp4编码器消费,将元数据用onTrackMetadata送给remuxer的onTrackMetadataReceived方法处理, 之后经由onInitSegment方法送给transmuxing-controller的onRemuxerInitSegmentArrival方法, 最后在mse-controller的appendInitSegment方法里, 为media source 附加相应mime type的sourceBuffer. 

- video data处理句柄: video tag的body部分的结构为VIDEODATA, 其头部(第一个字节)描述了帧类型和编码方式, 这里只支持avc编码, 当确定该tag为avc编码的视频数据后, 会根据数据部分的AVCVIDEOPACKET结构, 从中抽取出以nalu为单位的视频数据和元数据信息, 并将抽取出的视频数据推入_videoTrack队列中等待被mp4编码器消费, 将元数据用onTrackMetadata送给remuxer的onTrackMetadataReceived方法处理, 之后经由onInitSegment方法送给transmuxing-controller的onRemuxerInitSegmentArrival方法, 最后在mse-controller的appendInitSegment方法里, 为media source 附加相应mime type的sourceBuffer. 

当该tag的body部分被相应的处理方法处理完成后, parseChunks会调用_onDataAvailable句柄, 将_audioTrack和_videoTrack送出, 该句柄由remuxer的remux方法提供, 在这里, 音视频的轨道分别被_remuxVideo和_remuxAudio方法消费. 在这里, 首先会矫正各个sample的dts, 进行音视频同步,  并计算它们的duration, 然后把它们封装成一整个mdat box, 并在前面加上一个moof box, 通过onMediaSegment方法, 触发MEDIA_SEGMENT事件, 推送给mse使用. 

mse-controller 使用appendMediaSegment方法处理接收到的fmp4格式的数据, 在该方法内部将接收到的数据放在pendingSegments对列中等到消费, 并调用_doApppendSegments方法, 在这里将数据调用source buffer上的appendBuffer装入相应的buffer里, 然后就会被media source连接的video元素播放了.







### flvPlayer初始化相关流程

1. flvPlayer实例化
    
    - 传入媒体资源url
    - 确定loadedmetadata seek, canplay, stalled, onvProgress事件句柄
    - 实例上字段初始化

2. 连接媒体element
    - 用1中句柄监听媒体element上的相应事件
    - 初始化MSEController实例,并监听update_end, buffer_full, source_open事件, 并调用其**attachMediaElement**方法将该实例创建的media source连接到媒体元素,

3. 初始化加载数据流程
    - 初始化Transmuxer实例,并监听其上init_segment, media_segment, loading_complete, recovered_early_eof, io_error, demux_error, media_info, metadata_arrived, scriptdata_arrived, statistics_info, recommend_seekpoint事件, 并调用其**open**方法

4. 开始播放
    调用媒体元素**play**方法

### MSEController初始化相关流程

1. 确定sourceOpen, sourceEnded, sourceClose, sourceBufferError, sourceBufferUpdateEnd事件的句柄
2. 连接媒体元素, 创建MeidaSource实例, 并用1中句柄监听其上相应事件, 最后将ms附在媒体元素src属性上.

### Transmuser初始化相关流程

1. 创建TransmuxingController实例, 并监听其上io_error, demux_error, init_segment, media_segment, loading_complete, recovered_early_eof, media_info, metadata_arrived, scriptdata_arrived, statistics_info, recomment_seekpoint事件
2. 调用transmuxingcontroller实例的**start**方法

### TransmuxingController初始化相关流程

1. 媒体元素上创建segments:[{timestampBase, cors, withCredentials, duration, filesize, url}]
2. start: 
    - 调用 _loadSegment(segment)
        -   创建IOController, 为其注入error, seeked, complete, redirect, recoveredEarlyEof事件句柄
        - 注入dataarrival事件句柄
        - 调用其**open**方法

### IOController初始化相关流程

1. 选择seekHandler,并创建其实例
2. 选择loader
3. 创建loader实例, 注入其contentLengthKnwon, URLRedirect, dataArrival, complete, error事件句柄
4. 调用open方法, , 其中调用loader的open方法

### Loader初始化相关流程

请求媒体数据

## io

### 可选loader

- customLoader
- WebSocketLoader
- FetchStreamLoader
- MozChunkedLoader
- RangeLoader

flv.js在io-controller.js的selectLoader方法显示的逻辑表示, 默认优先的loader是WebSocketLoader 和 FetchStreamLoader

## [解码](./flvjs_demux.md)

## [编码](./flvjs_remux.md)

## seek实现(./seek.md)

## 详细文档

- [带注释版flv.js源码](https://github.com/BokunoMasayume/flv.js/tree/note/src)
- [flv格式](./flv-format.md)
- [mp4格式](./mp4-format.md)
- [mre](./media-source-extension.md)
- [flv.js使用文档](./flvdoc.md)
- [杂项](./sketch.md)

## [参考](./reference.md)


