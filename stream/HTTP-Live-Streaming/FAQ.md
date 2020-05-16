# 常见问题

1. 何种类型的编码器被支持？
	协议规范不限制编码器选择。不过，目前苹果实施应开发含有H.264视频和AAC音频（HE-AAC或AAC-LC）的MPEG-2传输流编码器。与通过UDP传输流的广播软件兼容的编码器应该也与现在Apple提供分段器兼容。
	
2. 支持的视频和音频详细格式？
	尽管协议规范不限制视频和音频格式，目前Apple支持一下格式：
	- 视频
		- H.264 Baseline Level 3.0, Baseline Level 3.1, Main Level 3.1, and High Profile Level 4.1.
	- 音频
		- HE-AAC or AAC-LC up to 48 kHz, stereo audio
		- MP3 (MPEG-1 Audio Layer 3) 8 kHz to 48 kHz, stereo audio
		- AC-3 (for Apple TV, in pass-through mode only)

3. 媒体文件应持续多长时间？
	要考虑的主要因素是，短段导致索引文件更频繁的刷新，这可能会给客户不必要的网络开销。长段将延长广播和初始启动时间的固有延迟。每个媒体文件持续10秒似乎可以保持大多数广播内容的合理平衡。
	
4. 在一个持续的会话期间，应该在索引文件上列出多少文件？
	一般推荐数量为3，但是可用数量更大。
	当选择最佳数量要考虑的很重要的一点是做播放/暂停和搜索操作时，在实时会话中提供的文件数量限制客户的行为。列表中的多个文件，时间越长，客户端越可以被暂停，而不会在广播中失去它的位置，进一步的回加入流时，以及更广泛的时间范围内，客户端可以寻求新的客户端开始广播。权衡是一个较长的索引文件会增加网络开销，在实时播放过程中，客户端都定期刷新索引文件，因此它确实增加了，即使索引文件通常很小。

5. 何种数据比率被支持？
	一个内容提供者选择一个流的数据率最大的被目标客户端平台和预期的网络拓扑影响。流协议本身不限制可使用的数据传输率。当前的实现的基于到iPhone的音频、视频流测试使用的数据比率低至64 Kbps和高达3 Mbps。仅音频流建议以64Kbps作为选择通过慢速蜂窝连接流传输。

	对于推荐的数据传输速率，请参阅[准备传输至iOS设备的媒体](https://developer.apple.com/library/ios/documentation/NetworkingInternet/Conceptual/StreamingMediaGuide/UsingHTTPLiveStreaming/UsingHTTPLiveStreaming.html#//apple_ref/doc/uid/TP40008332-CH102-SW8)

	> 备注：如果数据比率溢出了可用带宽，在启动前会有更高延迟，并且可语段可能需要暂停以缓冲个多数据。如果广播使用了提供滑动窗口访问数据的索引文件，这种情况下客户端会被逐渐落下，并导致丢掉一个或多个段。在VOD模式下，不会有段被丢掉，但是不充足的带宽会导致慢启动和在数据缓冲时周期性的失速。

6. .ts文件是什么
	.ts文件包含了一个MPEG-2传输流。这是一个封装的一系列编码媒体样本的文件格式——一般是音频与食品。文件格式支持多种压缩格式，包括MP3音频，AAC音频，H.264视频，等等。目前并非所有的压缩格式在苹果的HTTP实时流被支持。 有关当前支持的格式列表，请参阅[媒体编码器](https://developer.apple.com/library/ios/documentation/NetworkingInternet/Conceptual/StreamingMediaGuide/HTTPStreamingArchitecture/HTTPStreamingArchitecture.html#//apple_ref/doc/uid/TP40008332-CH101-SW3)。

	MPEG-2传输流是容器，并且不应该与MPEG-2压缩混淆。

7. .M3U8文件是什么
	一个.M3U8文件是一个可扩展的播放列表文件的格式。这是一个包含UTF-8编码的文本的m3u播放列表。 m3u文件格式，是适用于承载的媒体文件的URL列表的事实上的标准播放列表格式。这是作为HTTP实时流索引文件的格式。详情请见：[IETF Internet-Draft of the HTTP Live Streaming specification](http://tools.ietf.org/html/draft-pantos-http-live-streaming)
	
8. 客户端软件如何确定合适切换流？
	当前实现客户端实现是在播放时观察有效带宽。如果较高质量流可用并且带宽足以支持它，客户端会切换到更高的质量。如果一个低质量的流可用并且当前的带宽不足以支持当前流，客户端会切换到一个较低的质量。
	> 备注：对于备用流之间的无缝转换，流的音频部分应该在所有版本相同。
	
9. 我可以在那里找到苹果的媒体流分段器？
	媒体流分段器、文件流分段器和其他工具会经常更新，所以你应该在苹果开发者网站上下载最新版本的工具。参见[下载工具](https://developer.apple.com/library/ios/documentation/NetworkingInternet/Conceptual/StreamingMediaGuide/UsingHTTPLiveStreaming/UsingHTTPLiveStreaming.html#//apple_ref/doc/uid/TP40008332-CH102-SW3)
	
10. 对于苹果的分段器和典型的含备用的HTTP流，推荐设置是什么？
	参见[准备传输至iOS设备的媒体](https://developer.apple.com/library/ios/documentation/NetworkingInternet/Conceptual/StreamingMediaGuide/UsingHTTPLiveStreaming/UsingHTTPLiveStreaming.html#//apple_ref/doc/uid/TP40008332-CH102-SW8)
	这些设置是当前建议。也有一定的要求。在ISO/ IEC13818的传输流定义必须包含H.264（MPEG-4，第10部分）视频和AAC或MPEG音频。目前的mediastreamsegmenter工具，仅支持MPEG-2传输流。如果使用AAC音频，它必须有ADTS头标。 H.264视频接入单元必须使用一个访问单元定界符的NAL，并且必须是独特的PES包。

	该分割器也有一些用户可配置的设置。您可以通过从终端应用程序中输入man mediastreamsegmenter获得的命令行参数及其含义列表。 媒体段的建议长度为10秒，如果未指定目标的持续时间则是默认的。

11. 我如何指定何种解码器或H.264配置文件以播放我的流？
	使用`EXT-X-STREAM_INF`标签的`CODECS`参数。当这个参数被表示是，他必须包含所有需要的解码器和设定以播放流。下列的数值目前可以被识别。
		
	类型|参数  
	---|---
	AAC-LC |"mp4a.40.2"
HE-AAC|"mp4a.40.5"
MP3|"mp4a.40.34"
H.264 Baseline Profile level 3.0|"avc1.42001e" or "avc1.66.30" Note: Use "avc1.66.30" for compatibility with iOS versions 3.0 to 3.1.2.
H.264 Baseline Profile level 3.1 |"avc1.42001f"
H.264 Main Profile level 3.0|"avc1.4d001e" or "avc1.77.30" Note: Use "avc1.77.30" for compatibility with iOS versions 3.0 to 3.12.
H.264 Main Profile level 3.1|"avc1.4d001f"
H.264 Main Profile level 4.0|"avc1.4d0028"
H.264 High Profile level 3.1|"avc1.64001f"
H.264 High Profile level 4.0|"avc1.640028"
H.264 High Profile level 4.1|"avc1.640029"

	参数必须被包含在引用中。如果多个值被指定，一组引用会被使用以包含全部的值，并且这些值用逗号分开。
	
	```
	#EXTM3U
	#EXT-X-STREAM-INF:PROGRAM-ID=1, BANDWIDTH=500000,RESOLUTION=720x480
	mid_video_index.M3U8
	#EXT-X-STREAM-INF:PROGRAM-ID=1, BANDWIDTH=800000, RESOLUTION=1280x720
	wifi_video_index.M3U8
	#EXT-X-STREAM-INF:PROGRAM-ID=1, BANDWIDTH=3000000, CODECS="avc1.4d001e,mp4a.40.5", RESOLUTION=1920x1080
	h264main_heaac_index.M3U8
	#EXT-X-STREAM-INF:PROGRAM-ID=1, BANDWIDTH=64000, CODECS="mp4a.40.5"
	aacaudio_index.M3U8
	```
	
12. 我如何从视频音频输入创建仅音频流？
	在使用分段器时加入`-audio-only`参数。
	
13. 我如何想仅音频流加入一个静态图片？
	在引用流时使用`-meta-file`参数活在使用文件分段器时使用`-meta-type=picture`以向每个段中加入图像。例如，下面的例子会将一个名为poster.jpg的图片插入到由track01.mp3生成的音频流的每个段中。
	`mediafilesegmenter -f /Dir/outputFile -a --meta-file=poster.jpg --meta-type=picture track01.mp3`
	请记住图像没十秒钟会被重新发送，所以请尽可能保证文件足够小。
	
14. 我如何将一个仅音频流切换到视频音频流？
	同时使用`EXT-X-STREAM-INF`的`CODECS`和`BANDWIDTH`参数。
	`BANDWIDTH`参数指定了每个流所需的带宽。如果带宽足够音频使用，但不够最低质量的视频使用，客户端会写换到音频流。
	如果`OCDECS`参数被包括，它必须列出播放流所需的所有编码。如果只有一种音频编码被指定，流会被标记为仅音频。目前不需要指定流是仅音频的，所以`CODECS`参数是可选的。
	下面是一个例子，参数为快速链接速率为500Kbps，慢速链接速率为150Kbps，极慢连接速度为64Kbps。所有流都应该使用同样的64Kbps音频来保证不同流之间不会有显著差异。
	
	```
	#EXTM3U
	#EXT-X-STREAM-INF:PROGRAM-ID=1, BANDWIDTH=500000, RESOLUTION=1920x1080
	mid_video_index.M3U8

	#EXT-X-STREAM-INF:PROGRAM-ID=1, BANDWIDTH=150000, RESOLUTION=720x480
3g_video_index.M3U8

	#EXT-X-STREAM-INF:PROGRAM-ID=1, BANDWIDTH=64000, CODECS="mp4a.40.5"

	aacaudio_index.M3U8
	```

15. 服务器的硬件要求或者推荐要求是什么？
	硬件需求请参考问题1.
	苹果流分段器可以在任何Intel平台的Mac上运行。我们推荐使用有两个网络接口的Mac，例如Mac Pro或XServer。一个接口用于从本地网络接收编码的流，另一个接口提供访问网络的带宽。
	
16. HTTP实时流支持DRM么？
	不支持。但是媒体可以被加密，密钥访问可以在客户端向HTTPS服务器请求时通过认证限制访问。
	
17. 何种客户端平台被支持
	iPhone、iPad、iPod touch（需要iOS3.0或更高）、Apple TV（version2或更高）以及Mac OS X电脑。
	
18. 协议规范可以在那里找到？
	协议规范是一份IETF互联网草案，详见[http://tools.ietf.org/html/draft-pantos-http-live-streaming](http://tools.ietf.org/html/draft-pantos-http-live-streaming)

19. 客户端会缓存内容么？
	索引文件包含指示客户端是否需要缓存内容。否则客户端会缓存数据以减少搜寻数据时的开销。
	
20. 这是一个实时的配送系统么？
	不。它具有对应于含有流段中的媒体文件的大小和持续时间的固有等待时间。至少一个区段必须完全下被载，才能在客户端查看，并且可能需要确保段之间的无缝转换。此外，在编码器和分割器必须从输入的文件创建;这个文件的持续时间是之前媒体可供下载的最小的延迟。与推荐设置典型的延迟是30秒左右。

21. 等待时间是什么
	对于推荐设置来说大约30秒。参考问题15.
	
22. 我需要一个硬件编码器么？
	不需要。根据协议规范，使用软件编码器也是可行的。
	
23. 使用RTP/RSTP有何种优点？
	HTTP不太可能由路由器，NAT或防火墙的设置被禁止。没有端口需要被打开时默认情况下通常关闭。内容因此更容易得到通过对客户端的多个位置，无需特别设置。 HTTP也被更多的内容分发网络，它可以在较大的分销模式对成本的支持。一般情况下，更多的可用硬件和软件的工作原理未修改并打算与HTTP比RTP / RTSP。专长于使用自定义工具，如PHP HTTP内容交付也比较普遍。
	此外，HTTP实时流是在Safari支持和iOS的媒体播放器框架。不支持RTSP流。

24. 为什么我的流比特率比视频音频比特率总和稍高？
	MPEG-2传输流可能包含更多数据。他们利用固定大小的数据包时，包的内容比缺省包大小较小时会被填充。编码器和多路复用器实现他们的效率在包装媒体数据到这些固定的数据包大小有所不同。填充量可以与帧速率，采样率，和分辨率而改变。

25. 我如何减少数据超出并降低比特率？
	使用更有效的编码以降低超支，或调整编码设置。
	
26. 左右的媒体文件都需要是同一个MPEG-2传输流的一部分么？
	不需要。你可以混合来自不同流的文件。通过`EXT-X-DISCONTINUITY`标签来区分。参考协议规格来获取更多细节。为了取得最佳效果，所有媒体文件应有同样的分辨率。
	
27. 我可以在哪里找到关于设置HTTP视频/音频服务器的帮助？
	您可以访问[苹果开发者论坛](http://devforums.apple.com/)
	也可以查看[Best Practices for Creating and Deploying HTTP Live Streaming Media for the iPhone and iPad](
https://developer.apple.com/library/ios/technotes/tn2224/_index.html#//apple_ref/doc/uid/DTS40009745)


