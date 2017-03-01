title: Android中商业化播放器需要的功能
tags: [Android,MediaPlayer]
date: 2017-02-09 14:53:26
description:
---

商业化的播放器除了提供Android系统播放器支持的功能外，还有广告播放，seekBar中的截图、播放能力配置等功能

# 基本播放功能
基本功能是Android播放器可以提供
## 基本操作
开始、暂停、拖动

## 信息提示
开始播放、结束播放、Buffer进度、播放位置、拖动完成

## 错误提示
播放失败

## 支持格式
- 音频：mp3 wma wav ogg flac ape alac aac
- 视频：3gp asf avi dat flv m2ts mkv mov mp4 mpeg mpg rm rmvb ts vob wmv m4v mpe mts tp
- 字幕：srt ssa ass sub
- 编解码：h264/h265

## 播放能力
- 最大清晰度：4K
- 最高帧率：60fps
- 多音轨切换：DTS Dolby

## 流媒体协议
- HLS
- RTMP

# 非基本播放功能
特色功能需要自己开发基于FFmpeg的播放器

## 无缝切流
前置广告和中插广告需要和正片直接进行切流动作，切流前能预加载下一个流，保证平滑切换
## seekBar中的截图

## loading信息
- 下载速度
- 下载进度

## 播放配置功能
不同的机器播放能力不同，有的可以播放1080p，有的可以播放h265，这些需要一个本地默认配置和在线配置中心来处理这个问题

## 降级功能
一些机型上面自主播放器效果不好，需要使用系统播放器

## 缓存功能
能够根据业务需要缓存之前播放的数据


# 性能要求

## 稳定性
- Money时长：3*24
- 长时间播放：3*24 
     
## 资源消耗
- CPU：30~40%
- 内存：150M

## 快速响应
- 起播速度
- seek播放速度
- 切换清晰度速度


<font color="#FF0000">版权声明：本文为博主原创文章，转载请注明出处</font>
