# 介绍

这个规范允许js动态构建video和audio的媒体流. 它定义了一个*MediaSource*对象, 该对象可作为HTMLMediaElement的媒体数据源. MediaSource 对象有一个或多个*SourceBuffer*对象. 应用给SourceBuffer添加数据段,也可以基于系统性能或其他因素改变添加数据的quality. 来自SourceBuffer的数据被管理为正在解码播放的音频, 视频, 文本数据的track buffers
![mre pipeline model](./assets/img/mre_pipeline_model.svg)

## 接口

### MediaSource

代表了将由HTMLMediaElement对象播放的媒体资源

#### 静态方法

- MediaSource.isTypeSupported(MIMEType)
    返回一个布尔值表明给定的mime类型是否被当前浏览器支持, 这意味着是否可以成功创建这个mime类型的sourcebuffer对象.

#### 属性

- sourceBuffers
    返回一个SourceBufferList对象, 包含这个MediaSource实例的SourceBuffer对象列表
- activeSourceBuffers
    返回一个SourceBufferList对象, 包含了这个实例的sourceBuffers中的SourceBuffer的子集: 当前被选中的视频轨(video track), 启用的音频轨(audio track)和字幕轨(text tracks)的对象列表
- readyState
    返回一个包含当前meidaSource状态的集合
    - closed: 当前没有附着meida元素
    - open: 已附着并准备接收sourceBuffer对象和其中的数据
    - ended: 已附着但这个流已经没MediaSource.endOfStream()关闭
- duration
    获取和设置当前正在推流媒体的持续时间

- onsourceopen

- onsourceended

- onsourceclose

#### 方法

- addSourceBuffer(MIMEType)
    创建一个带有指定mime类型的新SourceBuffer并添加到MediaSource的SourceBuffers列表
- removeSourceBuffer(sb)
- endOfStream(error?:EndOfStreamError)
    用来表明流结束了.
    'network' : 表示因为网络错误而终止
    'decode' : 表示因为解码错误而终止
- setLiveSeekableRange(start, end)
    改变HTMLElement.seekable的行为.
- clearLiveSeekableRange()

#### 事件

- sourceopen
    readyState 从cloesd -> open 或 ended -> open
- sourceended
    readyState open -> ended
- sourceclose
    readyState open -> closed 或 ended -> closed


#### 用法

createObjectURL(your_mediasource_instance), 把得到的url作为src使用.



### SourceBuffer

代表了一个经由MediaSource对象被传入HTMLMediaElement的媒体块

#### 属性

- mode 
    'segments' 或 'sequence'
    segments: 用timestamps
    sequence: 不用timestamps
- updating
    只读, 指示appendBuffer()或remove()操作的异步过程仍在被处理中, 初始值为false
- buffered: TimeRanges
    在sourceBuffer中缓存的片段的时间范围.

- timestampOffset

- audioTracks: AudioTrackList
    只读, 该对象创建的AudioTrack对象的列表

        AudioTrack {
            readonly id
            readonly kind
            readonly label
            readonly language
            enabled
        }

- videoTracks
    只读, 该对象创建的VideoTrack对象的列表
        
        VideoTrack {
            readonly id
            readonly kind
            readonly label
            readonly language
            selected
        }
- textTracks
    只读, 该对象创建的TextTrack对象的列表
- appendWindowStart
    appendWindow : 一个演示时间戳范围(窗口)
- appendWindowEnd

- onupdatestart

- onupdate

- onupdateend

- onerror

- onabort

#### 方法

- appendBuffer(data: BufferSource)
    给其添加数据
- abort()

- remove(start, end)
    移除指定时间范围的媒体数据
...

#### 用法

source buffer内部状态:  
    - waiting_for_segment
    - parsing_init_segment
    - parsing_media_segment

### SourceBufferList 

列出多个SourceBuffer对象的简单的容器列表

### Initialization Segment

包含解码媒体段的初始化信息的一系列比特

### Media Segment

一系列比特的包含打包好的带时间戳的媒体数据, 媒体段总是和最近的初始化段相关联




### VideoPlaybackQuality

包含了有关正在被播放的视频的质量信息, 由 HTMLVideoElement.getVideoPlaybackQuality() 方法返回

...

### TrackDefault 

为在媒体块的初始化阶段没有包含类型, 标签和语言信息的轨道提供一个包含这些信息的SourceBuffer.

...

### TrackDefaultList

列出多个TrackDefault对象的简单的容器列表

