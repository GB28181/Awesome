# Media Source Extensions（简称MSE）使用简介

## 设计目的

* 运行JS处理流媒体。
* 这个特性允许JavaScript去动态地为<audio>和<video>创建媒体流。
* 它定义了一个MediaSource对象来给HTMLMediaElement提供媒体数据的源。

## 适用场景
* 播放流媒体（flv / m3u8）
* H5直播

## MSE基本数据流

Download ---》 **Response.arrayBuffer**(适用fetch/xhr等异步获取流媒体数据) ---》 **SourceBuffer**(添加到MediaSource的buffer中) ---》 `<vedio/>` or `<autio/>`

综上，MSE处理的三个基本要素：
1. MediaSource.SourceBuffer资源，通过请求获取并解析到SourceBuffer中
2. MediaSource 的实例对象
3. Video / Audio 媒体元素

# MSE示例化后的状态（readyState）

* `open` MSE实例已经绑定到了媒体元素上，等待接受数据或者正在接受数据（对应事件`sourceopen`）
* `closed` MSE实例未绑定到了媒体元素上（对应事件`sourceclosed`）
* `ended` MSE实例已绑定到媒体元素上，并且所有数据已经接受完毕（对应事件`sourceended`）

## 举个栗子：播放视频

### 思路
* vedio元素
* 创建MSE示例
* 创建`SourceBuffer`
* fetch异步获取视频流
* 将fetch的视频流数据，通过`sourceBuffer.appendBuffer`添加到`sourcebuffer`中
* 监听sourcebuffer事件`sourceBuffer.addEventListener('updateend', callback)`，在回调中处理播放器是否可以播放视频。

### 代码coding（来自Google Developer）
```
var videoMp4 = document.querySelector('.js-player-mp4');  // 获取video元素

  if (window.MediaSource) {

    var mediaSource = new MediaSource(); // 创建mediaSource实例

    videoMp4.src = URL.createObjectURL(mediaSource); // 将mediasource实例关联到video元素上

    mediaSource.addEventListener('sourceopen', sourceOpen);

  } else {

  console.log("The Media Source Extensions API is not supported.")

  }

  function sourceOpen(e) {

    URL.revokeObjectURL(videoMp4.src);

    // 设置：媒体的编码类型
    var mime = 'video/webm; codecs="vorbis, vp8"';

    var mediaSource = e.target;

    var sourceBuffer = mediaSource.addSourceBuffer(mime); // 创建sourcebuffer实例，用于存放媒体数据

    var videoUrl = './video/avegers3.webm';

    fetch(videoUrl) // 异步获取视频数据

      .then(function(response) {

        return response.arrayBuffer();

      })
      .then(function(arrayBuffer) {

      sourceBuffer.addEventListener('updateend', function(e) {

        if (!sourceBuffer.updating && mediaSource.readyState === 'open') {

          mediaSource.endOfStream(); // readyState在该方法执行后将会变为ended

          videoMp4.play().then(function() { // 数据已经ready完毕，可以播放了

          }).catch(function(err) {

            log('.js-log-mp4', err)

          });

        }
      });

      sourceBuffer.appendBuffer(arrayBuffer); // 将异步获取的视频数据添加到sourcebuffer中

      });
  }

文章来源：[Media Source Extensions（简称MSE）使用简介
](https://github.com/ivonzhang/my-fed-essay/wiki/Media-Source-Extensions%EF%BC%88%E7%AE%80%E7%A7%B0MSE%EF%BC%89%E4%BD%BF%E7%94%A8%E7%AE%80%E4%BB%8B) | 编辑：[Edward](https://github.com/crazybber)
