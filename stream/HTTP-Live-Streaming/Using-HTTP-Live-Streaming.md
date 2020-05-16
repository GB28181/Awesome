# 使用HTTP流媒体

## 下载工具

目前有许多已有的工具帮助你搭建一个HTTP流媒体服务。这些工具包括一个媒体流分段器、一个媒体文件分段器、一个流验证器、一个id3标签生成器和一个播放列表生成器。
这些工具时常会有更新，所以你应该在Apple开发者网站下载HTTP流媒体工具的最新版本。如果你是iOS开发者计划的成员你就可以获取这些软件。可以通过登录到[developer.apple.com](developer.apple.com)并使用搜索功能以获取相应软件。

### 媒体流分段器
`mediastreamsegmenter`命令行工具接受MPEG-2传输流为输入并生成一系列适合HTTP流媒体使用的等长的文件。这个工具同时可以生成索引文件（或者叫播放列表），加密媒体，生成解密密钥*(注：此处原文为encryption keys，疑应为decryption keys)*，优化文件以节省开销，和生成多个供选择的流的必要文件。详细信息请先确认安装了工具，然后使用`man mediastreamsegmenter`命令。
使用示例：`mediastreamsegmenter -s 3 -D -f /Library/WebServer/Documents/stream 239.4.1.5:20103`
这个示例从`239.4.1.5:20103`捕获实时流并从中生成媒体段文件和索引文件。索引文件包含当前的三个媒体段文件的列表(`-s 3)`。媒体段文件在使用了`-D`参数后会被删除。索引文件和媒体段文件会被存储在路径`/Library/WebServer/Documents/stream`中。

### 媒体文件分段器
`mediafilesegmenter`命令行工具接受编码过的媒体文件作为输入，将其转换为MPEG-2传输流并生成一系列适合HTTP流媒体使用的等长的文件。媒体文件分段器同时也可以生成索引文件和解密密钥。文件分段器使用方式与流分段器很相似，只是接受的输入是现有文件而不是从编码器生成的流。详情请使用`man mediafilesegmenter`命令。

### 媒体流验证器
`variantplaylistcreator`命令行工具会使用`mediafilesegmenter`的输出创建一个主索引文件（或播放列表）。这个索引文件列出了其他可选比特率的索引文件。`mediafilesegmenter`必须使用`-generate-variant-playlist`参数以生成变体播放列表生成器所需的输出。详情请使用`man variantplaylistcreator`命令。

### 媒体标签生成器
`id3taggenerator`命令行工具生成ID3元数据标签。这些标签可以写到文件中或插入传输中的视频流分段中。详情请请见[Adding Timed Metadata](https://developer.apple.com/library/ios/documentation/NetworkingInternet/Conceptual/StreamingMediaGuide/UsingHTTPLiveStreaming/UsingHTTPLiveStreaming.html#//apple_ref/doc/uid/TP40008332-CH102-SW4)

## 会话类型
HTTP流媒体协议支持两种会话：事件（实时广播）和视频点播（VOD）。

### VOD会话
对于VOD对话，媒体文件在整个播放时间中是持续可用的。索引文件是固定的并且包含从播放开始是的完整的文件列表。这种会话允许客户端访问整个程序。
VOD也可以用于发送“罐装”的媒体。HTTP流媒体为VOD提供了逐步下载的优点，例如支持媒体加密和在不同的比特率的流之间切换以调整连接速度。（QuickTime也通过逐步下载支持多数据速率但是QuickTime电影不支持播放中动态切换不同的速率）。

### 实时会话
实时会话（活动——可以被表示为一个完整的活动记录或者一个包含用户可以查看的有限时间长度的滑动窗口。
对于实时会话，当媒体文件被创建并且变成可用时，索引文件会被更新。新的索引文件列出新的媒体文件。旧的媒体文件可以从索引中移除并废弃，表示一个持续的流的移动窗口。这种类型的会话适合持续的广播。或者可以仅仅向索引文件中加入新的媒体晚间，这种会话可以在活动完成后轻松的被转换为VOD。
创建一个即时支持VOD的活动的广播是可行的。若要将实时广播转换成VOD，不要从服务器移除旧媒体文件及其在索引文件中的链接地址。在活动结束后在索引文件中加入`#EXT-X-ENDLIST`标签。这允许晚一些进入直播的客户端仍然能够看到完整的活动，同时也能无需其他工作即可存档重播。
如果你的播放列表包含`EXT-X-PLAYLIST-TYPE`标签，你应该将值从`EVENT`转换为`VOD`。

## 内容保护
含流段的媒体文件可以被单独加密。当使用加密时，相应的密钥文件会出现在索引文件中，以使客户机可以检索该密钥用于解密。

当一个密钥文件中在索引文件中列出，密钥文件包含必须用于解密在索引文件中列出的后续媒体文件的密钥。目前，HTTP实时流支持使用16字节密钥的AES-128加密。密钥文件的格式是用二进制格式这16八位位组的填充阵列。

Apple提供的媒体流分段器提供三种加密和支持加密的配置模式。

第一种模式允许您指定磁盘上现有的密钥文件的路径。在这种模式下，分段器将现有密钥文件的URL插入在索引文件中。它使用该密钥加密所有媒体文件。

第二种模式指示分段器生成一个随机密钥文件，并将其保存在指定的位置，并在索引文件中引用它。所有的媒体文件都采用这种随机产生的密钥加密。

第三模式指示分段器为每n个媒体段产生一个新的随机密钥文件，将其保存在一个指定的位置，并在索引文件中引用它。这种模式被称为密钥旋转。每个组的n个文件使用不同的密钥加密。

> 备注：所有媒体文件可以使用相同的密钥进行加密，或每隔一段时间申请神的密钥。理论极限是每媒体文件中的一个密钥，但由于每个媒体密钥增加了一个文件请求和传输请求用于呈现随后的媒体段，周期性更新密钥可以比每段都更新密钥更小的影响系统性能。

你可以使用HTTP或HTTPS提供密钥文件。你也可以选择保护使用自己的基于会话的认证方案密钥文件的交付。有关详细信息，请参阅[Serving Key Files Securely Over HTTPS](https://developer.apple.com/library/ios/documentation/NetworkingInternet/Conceptual/StreamingMediaGuide/DeployingHTTPLiveStreaming/DeployingHTTPLiveStreaming.html#//apple_ref/doc/uid/TP40008332-CH2-SW2)。

密钥文件需要的初始化向量（IV）以解密加密媒体。IV可以像密钥一样周期性改变。

## 缓存和分发协议

HTTPS通常用于提供密钥文件。它也可用于递送媒体段文件和索引文件，但当重视扩展性时不推荐这样做，因为HTTPS请求经常绕过web服务器缓存，导致所有内容请求被导向服务器，这与边缘网络分发系统的初衷相悖。

正是由于这个原因，确保您使用的任何内容分发网络了解到，.M3U8索引文件被缓存的时间不会超过一个为直播准备的媒体段的持续时间。其索引文件是动态变化的。

## 备用流

一个主索引文件可能会引用内容的备用流。引用可以用于支持不同质量级别不同的带宽或设备下的同一内容的多个流的递送。 HTTP实时流支持动态如果可用带宽变化流之间的切换。客户端软件使用启发式算法以确定画面切花的合适时间。目前，这些启发式基于在测量网络吞吐量的最新趋势。

主索引文件指向媒体备用流，包括其他索引文件专门标记列表，如图2-1所示。

![](https://developer.apple.com/library/ios/documentation/NetworkingInternet/Conceptual/StreamingMediaGuide/art/indexing_2x.png)

主索引文件和辅助索引文件都以.M3U8播放列表格式存储。主索引文件只需下载一次，但对于直播节目备用索引文件需要定期重新加载。在主索引文件中列出的第一个备用流第一个被使。在此之后，客户端可以根据可用带宽选择备用流。

注意，客户端可能会选择在任何时间改变到备用流，比如当移动设备进入或离开一个WiFi热点。所有的候补应该使用相同的音频，让流之间平滑过渡。

你可以使用`variantplaylistcreator`创建一组备用流。也可以在使用`mediafilesegmentertool`或`mediastreamsegmenter`工具时指定`-generate-variant-playlist`选项。详见[下载工具](https://developer.apple.com/library/ios/documentation/NetworkingInternet/Conceptual/StreamingMediaGuide/UsingHTTPLiveStreaming/UsingHTTPLiveStreaming.html#//apple_ref/doc/uid/TP40008332-CH102-SW3)

带使用备用流是，牢记以下注意事项是非常重要的：

- 该变体的播放列表的第一个条目，当用户加入该流时被播放，并用作测试的一部分，以确定哪些流是最合适的。其他条目的顺序是无关紧要的。
- 如果可能的话，编码变体足以提供在大范围的连接速度最优质的流。例如，编码在150 kbps的，350 kbps的，550 kbps的，900 kbps的1500 kbps的变种。
- 如果可能的话，在变体播放列表和独立的.M3U8播放列表文件中使用相对路径名。
- 备用流的视频横纵比必须完全一样，但备用流可以有不同的像素尺寸，只要它们具有相同的纵横比。例如，同为4：3的宽高比可以有400×300和800×600的尺寸。
- 包含在`EXT-X-STREAM-INF`的`RESOLUTION`域应帮助客户端选择合适的流。
- 如果你是一个iOS应用程序开发人员，你可以查询用户的设备，以确定的初始连接是网络或WiFi，并选择合适的主索引文件。
  当媒体流被播放时为确保用户具有良好的体验，不论初始的网络连接，你应该有多于一个的由不用的备用索引文件组成的主索引文件。
  蜂窝网络播放列表推荐使用150K流。
  Wi-Fi网络播放列表推荐使用240K或440K流。

> 备注：关于确定iOS设备的网络连接类型的细节，请参考[实例代码](https://developer.apple.com/library/ios/samplecode/Reachability/Introduction/Intro.html#//apple_ref/doc/uid/DTS40007324)

- 当你指定流变体的比特率时，`BANDWIDTH`参数紧密地匹配由一个给定流所需的实际带宽是很重要的。如果实际的带宽要求与`BANDWIDTH`参数显著不同，数据流的自动切换可能无法平稳甚至正确的被执行。

## 蜂窝网络下的视频

当您发送视频到移动设备，如iPhone或iPad，客户端的互联网连接可能在任何时间移动到或离开蜂窝网络。

HTTP流媒体允许客户端在动态网络带宽的变化中选择备用流，以当设备在蜂窝网络和WiFi间切换时提供最佳的流，例如，3G和EDGE连接之间切换。这是渐进式下载的一个显著优势。

我们强烈建议您使用HTTP流媒体提供视频给所有的蜂窝网络设备，甚至是视频点播，为观众在变化的条件下尽可能提供最佳的体验。

此外，你应该为蜂窝网络的客户端提供在64 Kbps或更少较慢的数据连接使用的备用流。如果你不能在64Kbps或更低速率下提供可接受质量的视频，您应该提供一个仅音频流，或者用静止图像的音频。

为蜂窝网络准备的链接分辨率推荐为400x224(16:9)或400x300(4:3)（参见[Preparing Media for Delivery to iOS-Based Devices](https://developer.apple.com/library/ios/documentation/NetworkingInternet/Conceptual/StreamingMediaGuide/UsingHTTPLiveStreaming/UsingHTTPLiveStreaming.html#//apple_ref/doc/uid/TP40008332-CH102-SW8)）。

## 应用程序需求

> 警告：在App Store中提交分发的应用必须符合下列要求

如果您的应用程序通过蜂窝网络传送视频，并且视频持续超过10分钟或在5分钟内使用超过5MB数据，你必须使用HTTP实时流。 （渐进式下载可用于更小的片段）。

如果您的应用程序通过蜂窝网络使用HTTP流媒体，您需要在64 Kbps或较低的带宽（低带宽流可能仅音频或静止图像音频）提供至少一个流。

这些要求适用于在App Store提交分发上使用苹果产品的iOS应用。不符合要求的应用程序基于苹果公司的决定可能会被拒绝或删除。

## 冗余流

如果您的播放列表包含备用流，它们不仅可以用于带宽或设备交替，也可以用于故障回退。从iOS 3.1开始，如果客户端无法加载一个流的索引文件（例如由于404错误），客户端尝试切换到备用流。

在对一个索引加载失败的情况下，客户端会选择网络连接支持的最高带宽的备用流。如果有多个相同的带宽的备用流，客户端按播放列表中列出的顺序选择。

您可以使用此功能来提供冗余流，使即使发生严重的局部故障，如服务器崩溃或内容分发节点下线的情况媒体依然能够送达到客户端。

为了支持冗余流，通常会创建一个流或多个备用带宽数据流，并生成一个播放列表文件。然后在独立的服务器或内容分发服务上创建一个平行流或组流。将备用流加入播放列表以保证各个带宽的备用流在主要流后面列出。例如，如果主流来自于服务器ALPHA，备份流来自服务器BETA，您的播放列表文件可能是这个样子：

```
#EXTM3U

#EXT-X-STREAM-INF:PROGRAM-ID=1, BANDWIDTH=200000, RESOLUTION=720x480

http://ALPHA.mycompany.com/lo/prog_index.m3u8

#EXT-X-STREAM-INF:PROGRAM-ID=1, BANDWIDTH=200000, RESOLUTION=720x480

http://BETA.mycompany.com/lo/prog_index.m3u8

 

#EXT-X-STREAM-INF:PROGRAM-ID=1, BANDWIDTH=500000, RESOLUTION=1920x1080

http://ALPHA.mycompany.com/md/prog_index.m3u8

#EXT-X-STREAM-INF:PROGRAM-ID=1, BANDWIDTH=500000, RESOLUTION=1920x1080

http://BETA.mycompany.com/md/prog_index.m3u8
```

注意，备用流和主要流在播放列表中混合存储。备用流列在对应带宽的主要流的后面。

你不被局限于单一的备用流集。例如在上面的例子中，ALPHA和BETA可接着GAMMA。同样的，你不需要提供一个完整的并行组流。例如可以在备用服务器上提供一个单一的低带宽流。

## 添加定时的元数据

您可以各种元数据添加到媒体流段。例如，您可以将专辑封面，艺术家的名字和歌曲标题添加到音频流。另举一个例子，您可以将当前击球手的名字和统计信息添加到棒球比赛的视频中。

如果仅音频流包括图像作为元数据，苹果客户端软件会自动显示它。目前，由苹果公司提供的客户端软件会自动显示的唯一元数据是伴随纯音频流的静止图像。

但是如果你正在写自己的客户端软件，无论是使用MPMoviePlayerController或AVPlayerItem，都可以使用timedMetaData属性访问流的元数据。

如果你正在写自己的分割器，可以阅读有关[HTTP流媒体中的定时元数据](https://developer.apple.com/library/ios/documentation/AudioVideo/Conceptual/HTTP_Live_Streaming_Metadata_Spec/Introduction/Introduction.html#//apple_ref/doc/uid/TP40010435)。

如果您在使用苹果的工具，您可以使用流分段或文件分段器的-F命令行选项指定元数据以添加定时的元数据。指定的元数据源可以是ID3格式的文件或图像文件（JPEG或PNG）。元数据中指定这种方式会自动插入到每一个媒体片段。

它被一个给定的时间偏移并插入到媒体流，所以它被称为定时元数据。定时元数据可以选择性地被插入一个给定的时间后的所有的段。

若要添加定时元数据到实时流，使用id3taggenerator工具，其输出设置为流分段。该工具生成ID3元数据，并将其列入输出流中的流分段。

标签生成器可通过脚本运行，例如，在指定的时间以指定的间隔插入元数据。新的定时元数据会自动替换任何现有的元数据。

一旦元数据被插入到一媒体段，它便会一直存在。例如如果实况广播在视频点播中被重新利用，它会保留原始广播期间插入的任何元数据。

加入定时元数据使用文件分段创建流是略微复杂一些。

1. 首先生成元数据样例。你可以生成使用id3taggenerator命令行工具生成ID3元数据并将输出保存到文件。
2. 接下来创建一个元数据宏文件——一个包含数据插入时间、元数据类型和元数据路径及文件名信息的文本文件。
   例如下列的元数据宏文件会在1.2秒时插入一副图片，在10秒时插入ID3标签
   
	```
	1.2 picture /meta/images/picture.jpg
	10 id3 /meta/id3/title.id3
	```

3. 最后当你调用媒体文件分段器时，使用-M选项指定元数据宏文件。

更多细节请参考mediastreamsegmenter, mediafilesegmenter, 和id3taggenerator的man页面。

## 添加隐藏式字幕

HTTP实时流支持流内的字幕。如果您使用的是流分段器，就需要将CEA-608字幕按ATSC A /72添加到MPEG-2传输流（在主视频基本流中）。如果你使用的文件分割器，你应该将媒体封装在QuickTime影片文件中，并添加字幕轨道（'CLCP'）。如果你正在编写一个应用程序，AVFoundation框架支持播放字幕。

直播还支持多种字幕和网络视频文本轨道（WebVTT插入）格式的隐藏字幕轨道。表2-1中的示例显示了在主播放列表中的两个字幕轨道。

*表2-1*

```
#EXTM3U

 

#EXT-X-MEDIA:TYPE=CLOSED-CAPTIONS,GROUP-ID="cc",NAME="CC1",LANGUAGE="en",DEFAULT=YES,AUTOSELECT=YES,INSTREAM-ID="CC1"

#EXT-X-MEDIA:TYPE=CLOSED-CAPTIONS,GROUP-ID="cc",NAME="CC2",LANGUAGE="sp",AUTOSELECT=YES,INSTREAM-ID="CC2"

 

#EXT-X-STREAM-INF:BANDWIDTH=1000000,SUBTITLES="subs",CLOSED-CAPTIONS="cc"

x.m3u8
```

在编码过程中，WebVTT文件像音频和视频媒体一样被分成段。由此产生的媒体播放列表包括段持续时间与相关的视频正确的点同步文本。

直播字幕的高级功能包括语义元数据，CSS样式，和简单的动画。更多信息和WebVTT的实现参考WWDC 2012: What's New in HTTP Live Streaming和[WebVTT: The Web Video Text Tracks Format specification.](http://dev.w3.org/html5/webvtt/)

## 准备传输至iOS设备的媒体

基于IOS的设备中使用的数据流的建议的编码器设置示下列四个表所示。对于实时流，这些设置应该在您的硬件或软件编码器可用。如果要从主文件重新编码为视频点播，你可以使用一个视频编辑工具，如Compressor。

为文件分段的文件格式可以是一个使用指定的编码的QuickTime电影，MPEG-4视频，或MP3音频。

流分段器所分个的流格式必须是MPEG基本音频和视频流，打包成MPEG-2传输流，并使用以下的编码。音频技术和视频技术的列表支持压缩格式。

- H.264压缩编码
	- H.264 Baseline 3.0: 所有设备
	- H.264 Baseline 3.1: iPhone 3G或更高, and 第二代iPod touch或更高.
	- H.264 Main profile 3.1: iPad（所有版本）, Apple TV 2 或更高, iPhone 4 或更高.
	- H.264 Main Profile 4.0: Apple TV 3 或更高, iPad 2 或更高, and iPhone 4S 或更高
	- H.264 High Profile 4.0: Apple TV 3 或更高, iPad 2 或更高, and iPhone 4S 或更高.
	- H.264 High Profile 4.1: iPad 2 或更高 and iPhone 4S 或更高.
- 200kbps以下的速率推荐10fps帧率。300kbps以下的速率推荐12到15fps帧率。其他的速率推荐29.97fps帧率。
- 音频用以下的一种编码
	- HE-AAC or AAC-LC, stereo
	- MP3 (MPEG-1 Audio Layer 3), stereo 
- 在使用支持AC-3输入的环绕式音频接收器或电视时，可以选择为Apple TV或AirPlay传输至Apple TV准备AC-3音轨。
- 在所有情况下推荐最小为22.05kHz的采样率和40kbps的比特率，使用Wi-Fi时推荐更高的采样率和比特率。

*表2-1 最低主编码器设置，16：9横纵比*

连接种类|像素|总比特率|视频比特率|关键帧
-----|-----|-------|--------|------
蜂窝网络|400 x 224|64 kbps|仅音频|无
蜂窝网络|400 x 224|150 kbps|110 kbps|30
蜂窝网络|400 x 224|240 kbps|200 kbps|45
蜂窝网络|400 x 224|440 kbps|400 kbps|90
WiFi|640 x 360|640 kbps|600 kbps|90

*表2-2 最低主编码器设置，4：3横纵比*

连接种类|像素|总比特率|视频比特率|关键帧
-----|-----|-------|--------|------
蜂窝网络|400 x 300|64 kbps|仅音频|无
蜂窝网络|400 x 300|150 kbps|110 kbps|30
蜂窝网络|400 x 300|240 kbps|200 kbps|45
蜂窝网络|400 x 300|440 kbps|400 kbps|90
WiFi|640 x 480|640 kbps|600 kbps|90

*表2-3 扩展主编码器设置，16：9横纵比*

连接种类|像素|总比特率|视频比特率|关键帧
-----|-----|-------|--------|------
WiFi|640 x 360|1240 kbps|1200 kbps|90
WiFi|960 x 540|1840 kbps|1800 kbps|90
WiFi|1280 x 720|2540 kbps|2500 kbps|90
WiFi|1280 x 720|4540 kbps|4500 kbps|90


*表2-4 扩展主编码器设置，4：3横纵比*

连接种类|像素|总比特率|视频比特率|关键帧
-----|-----|-------|--------|------
WiFi|640 x 480|1240 kbps|1200 kbps|90
WiFi|960 x 720|1840 kbps|1800 kbps|90
WiFi|960 x 720|2540 kbps|2500 kbps|90
WiFi|1280 x 960|4540 kbps|4500 kbps|90

*表2-5 扩展高级编码器设置，16：9横纵比*

连接种类|像素|总比特率|视频比特率|关键帧
-----|-----|-------|--------|------
WiFi|1920 x 1080|12000 kbps|11000 kbps|90
WiFi|1920 x 1080|25000 kbps|24000 kbps|90
WiFi|1920 x 1080|40000 kbps|39000 kbps|90

## 样例流

在Apple开发者网站上有一系列供测试用的HTTP流。这些样例展示了.M3U8索引文件、.ts媒体段文件的合适格式。这些流可以在[HTTP Live Streaming Resources](https://developer.apple.com/streaming/)被找到。

