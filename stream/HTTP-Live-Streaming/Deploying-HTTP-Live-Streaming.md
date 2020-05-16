# 部署HTTP流媒体

要实际部署HTTP实时流，你需要创建一个浏览器的HTML网页或客户端应用程序充当接收器。您还需要使用Web服务器和一种方式将实时流编码为MPEG-2传输流或从源文件制作的H.264文件和AAC编码的MP3或MPEG-4媒体。

您可以使用苹果提供的工具来分割流或媒体文件，并生成索引文件和变异的播放列表（见[下载工具](https://developer.apple.com/library/ios/documentation/NetworkingInternet/Conceptual/StreamingMediaGuide/UsingHTTPLiveStreaming/UsingHTTPLiveStreaming.html#//apple_ref/doc/uid/TP40008332-CH102-SW3)）。

您应该使用苹果公司提供的流媒体服务验证您的流，以确保它们与HTTP实时流完全兼容。

您可能要加密数据流，在这种情况下，你可能还需要通过HTTPS安全提供加密密钥文件，这样只有您预期的客户端可以解密。

## 创建HTML页面

最简单分发HTTP实时流媒体的方式是创建一个包含`<video>`标签的的HTML5页面，`.M3U8`播放列表文件作为视频源。一个例子在下文列出。

```
<html>
<head>
    <title>HTTP Live Streaming Example</title>
</head>
<body>
    <video
        src="http://devimages.apple.com/iphone/samples/bipbop/bipbopall.m3u8"
        height="300" width="400"
    >
    </video>
</body>
</html>
```
对于不支持HTML5视频元素，或不支持HTTP实时流的浏览器，您可以在`<video>`和`</video>`之间包含后备代码。例如，您可能回落到渐进式下载影片或使用QuickTime插件的RTSP流。见[Safari浏览器HTML5音频和视频指南](https://developer.apple.com/library/ios/documentation/AudioVideo/Conceptual/Using_HTML5_Audio_Video/Introduction/Introduction.html#//apple_ref/doc/uid/TP40009523)的例子。

## 设置网络服务器

HTTP实时流可以从一个普通的Web服务器提供服务;无需特殊配置，除了被他们的文件扩展名关联送达的MIME类型的文件。

配置以下MIME类型HTTP实时流：

文件扩展名|MIME类型
---------|--------
.M3U8|application/x-mpegURL 或 vnd.apple.mpegURL
.ts|video/MP2T

如果你的Web服务器限制于MIME类型，您可以提供MIME属性为audio/mpegURL的.m3u文件以保证兼容性。

索引文件可以是长期的，也可以频繁地重新下载，但它们是文本文件，并可以非常有效地压缩。您可以通过启用即时的.M3U8索引文件.GZIP压缩降低服务器开销;在HTTP实时流客户端会自动解压压缩索引文件。

有时可能需要缩短.M3U8文件的TTL值，以实现对下游web缓存适当缓存行为，因为这些文件在实况广播经常覆盖，并且对于每个下载请求应下载最新的版本。具体建议内容请与服务提供商确认。对于VOD，索引文件是静态的，只需下载一次，所以缓存是不是一个因素。

## 使你的流生效

`mediastreamvalidator`工具是用于验证HTTP实时流流和服务器的命令行实用程序（详见[下载工具](https://developer.apple.com/library/ios/documentation/NetworkingInternet/Conceptual/StreamingMediaGuide/UsingHTTPLiveStreaming/UsingHTTPLiveStreaming.html#//apple_ref/doc/uid/TP40008332-CH102-SW3)）。

媒体流验证模拟的HTTP实时流媒体会话，并验证索引文件和媒体段符合HTTP实时流规范。它会执行几项检查，以确保可靠的数据流。如果发现任何错误或问题时，会显示详细的诊断报告。

你应该总是在提供一个新的流或备用流前运行验证器。

媒体流验证显示您提供流的列表，然后为每个数据流的时序结果。（这可能需要几分钟的时间来计算实际时间）下面是一个验证器的输出。

```
$ mediastreamvalidator -d iphone http://devimages.apple.com/iphone/samples/bipbop/gear3/prog_index.m3u8

mediastreamvalidator: Beta Version 1.1(130423)

 

Validating http://devimages.apple.com/iphone/samples/bipbop/gear3/prog_index.m3u8

 

--------------------------------------------------------------------------------

http://devimages.apple.com/iphone/samples/bipbop/gear3/prog_index.m3u8

--------------------------------------------------------------------------------

 

Playlist Syntax:     OK

 

Segments:    OK

 

Average segment duration: 9.91 seconds

Segment bitrate: Average: 509.56 kbits/sec, Max: 840.74 kbits/sec

Average segment structural overhead: 97.49 kbits/sec (19.13 %)
```
欲了解更多信息，请参阅[流媒体验证工具的结果解释](https://developer.apple.com/library/ios/technotes/tn2235/_index.html#//apple_ref/doc/uid/DTS40010221)。

## 通过HTTPS安全提供文件密钥

您可以通过加密保护媒体。文件分割器和流分段都有加密选项，你可以设定定期更改加密密钥。有你设定谁与你分享的密钥。

密钥文件需要的初始化向量（IV），以解密加密的媒体解码。IV型可周期性地改变，就像密钥一样。目前对于尽量减少开销的建议是，每3-4小时更换密钥，每50 MB数据更换IV。

尽管被限制了密钥的访问，如果通过HTTP传输，密钥文件还是会被窃听者获得一份复制。一种解决方式是通过HTTPS安全地发送密钥。

在尝试通过HTTPS服务密钥文件前，你应该做一个测试，通过HTTP内部Web服务器提供密钥。这允许您添加HTTPS进来之前调试你的配置。一旦你有一个已知的工作系统，你就可以切换到HTTPS。

下面是通过HTTPS提供HTTP实时流密钥需要满足的三个条件：

- 你需要在你的HTTPS服务器上安装可信机构签署的SSL证书。
- 密钥文件的认证域必须相同作为第一播放列表文件的认证域。做到这一点最简单的方法是为从HTTPS服务器变种播放列表文件变种播放列表文件只需下载一次，所以这应该不会造成过大的负担。其他的播放列表文件可以使用HTTP提供服务。
- 您必须初始化自己的会话供用户进行身份验证，或者你必须在客户端存储凭据——HTTP实时流不提供进行身份验的用户对话。如果你正在编写自己的客户端应用程序，你可以存储凭据，基于Cookie桌和HTTP摘要，并提供在didReceiveAuthenticationChallenge回调的凭证（详情请参阅[使用NSURLConnection](https://developer.apple.com/library/ios/documentation/Cocoa/Conceptual/URLLoadingSystem/Tasks/UsingNSURLConnection.html#//apple_ref/doc/uid/20001836)和[验证的挑战和TLS链验证](https://developer.apple.com/library/ios/documentation/Cocoa/Conceptual/URLLoadingSystem/Articles/AuthenticationChallenges.html#//apple_ref/doc/uid/TP40009507)）。您提供的凭据会被缓存，并通过媒体播放器重复使用。

> **重要：**你必须拥有由认证机构签发的SSL证书以使用HTTPS服务器来提供HTTP实时流

如果您的HTTPS服务器没有被信任的机构签署的SSL证书，你仍然可以通过创建一个自签名的SSL证书管理局和服务器叶证书测试您的设置。将需要认证的证书附加到电子邮件，发送给你想作为一个直播客户端的设备，并点击邮件中的附件，使设备信任服务器。
一个例子记录在[HTTP实时流中的MPEG-2流加密格式](https://developer.apple.com/library/ios/documentation/AudioVideo/Conceptual/HLS_Sample_Encryption/Intro/Intro.html#//apple_ref/doc/uid/TP40012862)


