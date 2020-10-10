# 介绍

这个规范允许js动态构建video和audio的媒体流. 它定义了一个*MediaSource*对象, 该对象可作为HTMLMediaElement的媒体数据源. MediaSource 对象有一个或多个*SourceBuffer*对象. 应用给SourceBuffer添加数据段,也可以基于系统性能或其他因素改变添加数据的quality. 来自SourceBuffer的数据被管理为正在解码播放的音频, 视频, 文本数据的track buffers
![mre pipeline model](./assets/img/mre_pipeline_model.svg)

## 定义
