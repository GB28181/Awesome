Usage
编译SRS，需要切换到Develop分支，并开启gb28181功能：

git checkout develop &&
./configure --with-gb28181 && 
make clean && make
然后使用配置文件conf/push.gb28181.conf 启动：

./objs/srs -c conf/push.gb28181.conf 
Remark: 一定要修改配置文件中的host配置，改成你的服务器的IP，摄像头能访问到的这个IP。后续会改进为自动获取，目前还需要修改配置。

然后按下面的操作配置摄像头，推流到SRS：

![配置IPC](https://user-images.githubusercontent.com/2777660/78043273-bb749800-73a5-11ea-97e8-01f01077692c.png)

最后，观看RTMP流：rtmp://localhost:1935/live/34020000001320000001

Remark: 海康摄像头默认会连接萤石云，可以在APP上看到摄像头的内网IP，然后访问这个IP进入管理页面。其他摄像头也有对应办法，可以配置摄像头使用GB28181推流。

Remark：如果看不了流，请确认你的服务器IP，还有流名称也就是视频通道编码ID是否正确。

Remark: 默认不开启声音，如果需要开启声音，要打开配置audio_enable on;，并且在摄像头的音频配置中开启。

Remark: 若需要开启音频，需要将编码设置为AAC，同时采样率设置为44100HZ，否则Flash可能无法观看。
