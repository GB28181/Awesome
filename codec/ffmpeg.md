
## 介绍

[FFmpeg](https://www.ffmpeg.org/download.html) 是一款非常强大的音视频处理工具，它可以完成对视频的编码，解码以及整合等等功能。支持命令行操作。

# 举个栗子
* Cut video file into a smaller clip

`ffmpeg -i input.mp4 -ss 00:00:50.0 -codec copy -t 20 output.mp4`
* Split a video into multiple parts

`ffmpeg -i video.mp4 -t 00:00:50 -c copy small-1.mp4 -ss 00:00:50 -codec copy small-2.mp4`

* Convert video from one format to another

`ffmpeg -i youtube.flv -c:v libx264 filename.mp4`

`ffmpeg -i video.wmv -c:v libx264 -preset ultrafast video.mp4`

* [More](https://www.labnol.org/internet/useful-ffmpeg-commands/28490/)

## 参考资料

[Useful FFmpeg Commands](https://www.labnol.org/internet/useful-ffmpeg-commands/28490/)

---

来源：[ffmpeg处理视频](https://github.com/ivonzhang/my-fed-essay/wiki/FFmpeg%E5%A4%84%E7%90%86%E8%A7%86%E9%A2%91) | 
编辑：[Edward](https://github.com/crazybber) 
