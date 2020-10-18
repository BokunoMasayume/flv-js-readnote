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

内个本体和controller是什么关系


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

## io

### 可选loader

- customLoader
- WebSocketLoader
- FetchStreamLoader
- MozChunkedLoader
- RangeLoader

flv.js在io-controller.js的selectLoader方法显示的逻辑表示, 默认优先的loader是WebSocketLoader 和 FetchStreamLoader

## 其他

- [flv格式](./flv-format.md)
- [mre](./media-source-extension.md)
- [flv.js文档](./flvdoc.md)