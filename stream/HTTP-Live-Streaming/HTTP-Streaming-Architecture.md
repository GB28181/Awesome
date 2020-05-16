#HTTP串流结构

HTTP流媒体允许你从普通网络服务器传送实时或者预先录制的音频或视频到任何运行iOS3.0及更高版本的设备（包括iPad和Apple TV）或者任何安装了Safari 4.0或更高版本的电脑。串流同时包含有加密和认证支持。

##概览

概念上，HTTP流媒体由三部分组成：服务器组件，分发组件，客户端软件。

**服务器组件**负责取得媒体的输入流并且将其编码，转换成适合传输的格式，然后为分发准备转换好的媒体数据。

**分发组件**由标准网络服务器组成。它们负责接收客户端请求并向客户端分发准备好的媒体数据和关联资源。对于大规模的分发，边际网络或者其他内容分发网络也可以被使用。

**客户端软件**负责确定需要请求的媒体数据，下载这些资源，然后将其重新组合以保证能够展示给用户完整的串流。客户端软件包含在iOS 3.0及更高版本和安装了Safari 4.0及更高版本的电脑上。

在典型的配置中，硬件编码器接受音频-视频输入，编码为H.264视频和AAC音频，然后以MPEG-2
传输流输出。这个传输流将会被流分段软件分为一系列的小媒体文件。这些媒体文件会被放置在网络服务器上。分段器同时创建并维护一个包含这些媒体文件列表的索引。这份索引的链接会在网络服务器上被发布。客户端软件读取索引，然后按顺序请求列表上的媒体文件然后不间断的播放媒体片段。

一个简单的HTTP串流配置如图1-1。

*图1-1*
![Figure 1-1](https://developer.apple.com/library/ios/documentation/NetworkingInternet/Conceptual/StreamingMediaGuide/art/transport_stream_2x.png)

输入可以是实时的或者是一个预先录制好的源。典型的编码是MPEG-4（H.264视频和AAC音频），并由已有的硬件封装为MPEG-2传输流。该MPEG-2传输流之后会被分成若干段并被保存为一系列的`.ts`媒体文件。这项工作可以借助例如Apple stream segmenter这类软件完成。

仅包含音频的串流可以是一系列的MPEG基本音频文件。编码包括含ADTS头的AAC、MP3和AC-3。

分段器同时会创建一个索引文件。索引文件包含一个媒体文件列表，同时还保存有元数据。这份索引是一个`.M3U8`播放列表文件。索引文件的链接经由客户端访问，之后客户端会按序请求被索引的文件。

##服务器组件

服务器需要有一个媒体编码器（可以是现有的硬件），以及能够将编码过的媒体文件分歌声段并存储为文件的方法。这种方法可以是如Apple提供的媒体流分段器软件，或者一个完整的第三方解决方案。

###媒体编码器

媒体编码器从音频-视频设备接收实时信号，将媒体编码并为传输做好包装。编码应该被设定为客户端设备支持的格式，如H.264视频和HE-AAC音频。现阶段支持的传输格式为MPEG-2视频-音频传输流，及仅音频的MPEG简单流。

编码器通过本地网络将编码过的媒体以MPEG-2传输流发送给流分段器。MPEG-2传输流不应与MPEG-2视频压缩格式相混淆。传输流是可以使用多种不同压缩格式包装格式。[视频技术](https://developer.apple.com/library/ios/documentation/Miscellaneous/Conceptual/iPhoneOSTechOverview/MediaLayer/MediaLayer.html#//apple_ref/doc/uid/TP40007898-CH9-SW6)和[音频技术](https://developer.apple.com/library/ios/documentation/Miscellaneous/Conceptual/iPhoneOSTechOverview/MediaLayer/MediaLayer.html#//apple_ref/doc/uid/TP40007898-CH9-SW2)列出了支持的压缩格式

>**重要：** 视频编码器不应该在编码中更改诸如视频尺寸或编译码器类型等串流设置。如果更改设置不可避免，则务必只在段的边界时更改设置，并且比如为下一个段设置`EXT-X-DISCONTINUITY`标签。

###流分段器

流分段器是一个从本地网络读取传输流并将其分成一系列等长的小媒体文件的软件。尽管每个段是独立的文件，这个视频文件是从一个可以被无缝重建为一个持续的流被建立的。

分段器同时会创建一个包含每个独立媒体文件的索引。每当分段器完成一个新媒体文件的创建，索引文件就会被更新。索引文件用于追踪媒体文件的的可用性和存放位置。分段器在分段流程中同时可以对每个媒体段进行加密并创建一个密钥文件。

媒体段文件被保存为`.ts`文件（MPEG-2传输流文件）。索引文件被保存为`.M3U8`播放列表。

###文件分段器

如果你已经有了使用被支持的编码器编码过的媒体文件，此时可以使用文件分段器将其包装为MPEG-2传输流并分成等长的小段。文件分段器允许你使用现有的音频和视频文件库通过HTTP流媒体来传输视频。文件分段器所作的任务与流分段器相同，只是输入从流变为了文件。

##媒体段文件

媒体段文件通常基于编码器的输入由流分段器产生，包含一系列含有MPEG-2传输流小段的`.ts`文件。该传输流编码为H.264视频和AAC/MP3/AC-3音频。对于仅音频的广播，分段器可以创建简单音频流，编码包括带ADTS头的AAC、MP3和AC-3。

##索引文件（播放列表）

索引文件通常由流分段器和文件分段器产生，以`.M3U8`播放列表格式存储。`M3U8`是一个用于MP3播放列表的`.m3u`的扩展格式。

>备注：由于索引文件是`.m3u`格式的扩展，以及系统同时支持`.mp3`音频媒体文件，客户端软件同时也可能兼容用于网络广播串流的普通MP3播放列表。

下面是一个`.M3U8`格式索引文件的简单例子。例子中分段器从一个完整的流创建了三个未加密的10秒长度的媒体文件。

```
#EXT-X-VERSION:3
#EXTM3U
#EXT-X-TARGETDURATION:10
#EXT-X-MEDIA-SEQUENCE:1
 
# Old-style integer duration; avoid for newer clients.
#EXTINF:10,
http://media.example.com/segment0.ts
 
# New-style floating-point duration; use for modern clients.
#EXTINF:10.0,
http://media.example.com/segment1.ts
#EXTINF:9.5,
http://media.example.com/segment2.ts
#EXT-X-ENDLIST
```

为了保证最大的精确度，当你向支持版本3及更高的协议的客户端传送播放列表时应该将所有持续时间设置为浮点数（较老版本客户端仅支持整形变量）。当你使用浮点数时，必须执行协议版本，否则应该遵守协议版本1。

>备注：你可以使用Apple提供的分段器并以MPEG-4视频或者AAC/MP3音频作为源来产生一系列示例播放列表。详情请见[媒体分段器](https://developer.apple.com/library/ios/documentation/NetworkingInternet/Conceptual/StreamingMediaGuide/UsingHTTPLiveStreaming/UsingHTTPLiveStreaming.html#//apple_ref/doc/uid/TP40008332-CH102-SW7)。

索引文件同时也可能包含加密密钥文件和为不同带宽准备的其他索引文件的链接。关于索引文件格式的详情，请参见IETF的互联网草案：[HTTP Live Streaming specification](http://tools.ietf.org/html/draft-pantos-http-live-streaming)。

索引文件通常由创建媒体段文件的分段器同时生成。或者只要符合已发布的规范，即可独立创建`.M3U8`文件和媒体段文件。例如对于仅音频的广播，你可以通过文本编辑器创建`.M3U8`文件并李处一系列现存的`.MP3`文件。

##分发组件

分发系统是一个通过HTTP向客户端媒体文件和索引文件的网络服务器或者网络缓存系统。传输内容不需要定制化的服务器模块，并且不需要对服务器进行大量设置。

推荐配置一般仅限于指定`.M3U8`和`.MP3`文件的MIME类型关联。

详情请见[部署HTTP流媒体](https://developer.apple.com/library/ios/documentation/NetworkingInternet/Conceptual/StreamingMediaGuide/DeployingHTTPLiveStreaming/DeployingHTTPLiveStreaming.html#//apple_ref/doc/uid/TP40008332-CH2-SW3)。

##客户端组件

客户端软件基于标志了流的链接来获取索引文件。索引文件按序表明有效媒体文件位置、解密密钥和是否有其他的有效的源可供使用。对于选定的串流，客户端下载按队列下载每个可用的媒体文件。每个文件包含串流的连续段。当客户端下载了足够的数据之后会将重新组合的流播放给用户。

客户端有责任获取解密密钥、认证或为认证提供用户交互界面以及按需解密媒体文件。

这个流程会持续知道客户端在索引文件中检测到了`#EXT-X-ENDLIST`标签。如果`#EXT-X-ENDLIST`标签没有出现，这个索引文件是一个持续广播的一部分。在持续广播中，客户端会周期性加载更新版本的索引文件。之后会在更新过的索引文件中寻找心的媒体文件和解密密钥并将这些链接加入下载队列。


