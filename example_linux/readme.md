### 1. 前提条件

webrtc源码目录：/root/smbDir/webrtc-checkout/src
webrtc源码编译：https://shiftregister.cn/index.php/compile-webrtc/

### 2. 示例 g711_test

#### 2.1原始素材处理

查看原始音频信息，采样率44100 Hz，双声道stereo

```shell
ffprobe.exe 1024.mp4
    Stream #0:1(und): Audio: aac (LC) (mp4a / 0x6134706D), 44100 Hz, stereo, fltp, 130 kb/s (default)
```


将音频转码为单声道16000hz

```shell
#-ac 1 设置声道数为1
#-ar 48000 设置采样率为48000Hz
ffmpeg -i 1024.mp4 -ac 1 -ar 16000 1024_16.mp4
```

将音频解码为PCM

```shell
ffmpeg -i 1024_16.mp4 -f s16le -acodec pcm_s16le 1024.pcm
```

播放PCM音频

```shell
ffplay -ar 16000 -channels 1 -f s16le -i 1024.pcm
```

#### 2.2 音频相关概念 

1、AAC 
一个AAC原始帧包含某段时间内1024个采样点相关数据，用1024主要是因为AAC是用的1024点的mdct。
音频帧的播放时间 = 一个AAC帧对应的采样样本的个数 / 采样频率(单位为s)。
采样率(samplerate)为 44100Hz，表示每秒 44100个采样点, 所以，根据公式,
音频帧的播放时长 = 一个AAC帧对应的采样点个数 / 采样频率
则，当前一帧的播放时间 = 1024 * 1000/44100= 22.32ms(单位为ms)

2、MP3 
mp3 每帧均为1152个字节， 则：
每帧播放时长 = 1152 * 1000 / sample_rate （单位：ms）
例如：sample_rate = 44100HZ时， 计算出的时长为26.122ms，这就是经常听到的mp3每帧播放时间固定为26ms的由来。

3、音频帧大小的计算
假设音频采样率 = 8000，采样通道 = 2，位深度 = 8，采样间隔 = 20ms
首先我们计算一秒钟总的数据量，采样间隔采用20ms的话，说明每秒钟需采集50次(1s=1000ms)，那么总的数据量计算为
一秒钟总的数据量 = 8000 * 2 * 8/8 = 16000（Byte）
所以每帧音频数据大小 = 16000/50 = 320（Byte）
每个通道样本数 = 320/2 = 160（Byte）

#### 2.3 程序运行

程序运行参数

```shell
./g711_test 480 A 1024.pcm 1024_encode_decode.pcm lawA.g711
```

结果验证

```shell
#-ac: 音频通道数
#-ar：音频采样率
#-f： 文件格式 
ffplay -ar 16000 -channels 1 -f s16le -i 1024_encode_decode.pcm  
ffplay -i lawA.g711 -f alaw  -ac 1  -ar 16000
```

### 3. 示例 turnserver

CMake使用Ninja加速C++代码构建过程

https://zhongpan.tech/2019/06/26/008-cmake-with-ninja/

ninja -C out/Default chrome命令



https://note.qidong.name/tags/ninja/

https://www.jianshu.com/p/d118615c1943

https://note.qidong.name/2017/08/ninja-syntax/

