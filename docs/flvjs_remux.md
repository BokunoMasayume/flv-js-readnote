# flv.js 编码器

编码为fmp4文件

```js
segment = {
    type: 'audio'|'video',
    data: moof+mdat
    sampleCount:采样数目,
    info: 包含着数据的pts, dts信息
}
```