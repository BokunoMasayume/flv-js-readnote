# flv.js 阅读笔记

## 简介

flv.js将flv格式的视频数据转换为mp4的格式的, flv和mp4都是‘盒子格式’, 内部的具体视频和音频编码数据是不需要改变的.
因此flv.js的主要工作就是从网络中获取flv格式的视频数据, 解析参数信息, 和音视频裸流, 然后组合成mp4的格式, 通过mse传递给HTMLMediaElement(e.g. video) 

## 主要功能模块划分

- 解码 demux
- 编码 remux
- 网络接收 io
- 核心 core

## 组织方式

内个本体和controller是什么关系

## 目录

- [flv格式](./flv-format.md)
- [mre](./media-source-extension.md)
- [flv.js文档](./flvdoc.md)