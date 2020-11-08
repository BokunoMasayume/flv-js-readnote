# flv.js 解码器

解码flv文件, 可解码的视频编码为AVC(H.264), 可解码的音频为AAC和MP3, 将音视频解码为裸流通过_videoTrack和_audioTrack送给编码器, 编码为mp4.

## _videoTrack 和 _audioTrack

### _audioTrack对象
```js
_audioTrack = {
    type: 'audio',
    id: 2,
    sequenceNumber: 由编码器计算和使用
    samples: [
        aacSample 0,
        aacSample 1,
        //...
    ],
    length: sample数
}

aacSample = {
    unit: 音频数据包 直接是flv tag body AACAUDIODATA 中拿掉头一个表示类型字节后的数据块,
    length: 音频数据包长度,
    dts:解码中使用,告诉解码器何时解码该帧 解码时间戳 flv tag header中的时间戳计算而来,
    pts:播放中使用,告诉解码器何时播放该帧 播放时间戳 flv tag header中的时间戳计算而来,
}
```

### _videoTrack对象
```js
_videoTrack = {
    type: 'video',
    id: 1,
    sequenceNumber: 由编码器计算和使用
    samples: [
        avcSample 0,
        avcSample 1,
        //...
    ],
    length: samples个数
}

avcSample = {
    units: [nalUnit, nalUnit, ...],
    length: units data部分的总长度,
    isKeyframe: 由VIDEODATA的头部字段和nalu中的type字段标明
    dts: 解码中使用,告诉解码器何时解码该帧 解码时间戳 flv tag header中的时间戳计算而来,
    cts: 在VIDEODATA的头部字段标明,
    pts: (dts+cts)
    fileposition: 当为关键帧时存在, 该帧的偏移\(字节\)
}

nalUnit = {
    type: nalu类型
    data: nalu数据, nalu size + nalu payload \(h.264视频编码类型为AVC1时, nalu没有startcode, 用naluSzie代替\)
}
```

## _audioMetadata 和 _videoMetadata

### _audioMetadata对象字段
```js
{
    type: 'audio',
    id: 2,
    timescale: 1000 ,// 正常速率, 1000
    duration: 时长,
    audioSampleRate: 采样率,
    channelCount: 声道数, mono or stereo
    codec: misc.codec,
    originalCodec: misc.originalCodec,
    config: misc.config, // aac
    refSampleDuration,
    

}

```