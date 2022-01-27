<font color=#FF0000 size=4> <p align="center">FFmpeg、FFprobe、FFplay</p></font>

//读取本地视频文件推理至媒体服务器
root@dwj:/bin# ffmpeg -re -i "/root/123.mp4" -vcodec copy -f flv "rtmp://10.3.9.106:1935/live/2"

//拉取流媒体视频保存本地视频文件
root@dwj:/bin# ffmpeg -re -i "rtmp://10.3.9.106:1935/live/2" -vcodec copy -f flv "/root/123.flv"

//使用硬件加速解码文件
root@dwj:/bin# ffmpeg -y -vsync 0 -hwaccel cuda -hwaccel_output_format cuda -i input.mp4 -c:a copy -c:v h264_nvenc -b:v 5M output.mp4

//硬件解码视频文件到原始格式视频
root@dwj:/bin# ffmpeg -y -vsync 0 -c:v h264_cuvid -i /opt/bin/input.mp4 output.yuv

//编码原始格式视频到其他格式
root@dwj:/bin# ffmpeg -y -vsync 0 -s 640x480 -i output.yuv -c:v h264_nvenc outputxx.mp4

//播放视频1.mp4,将width和height设置为500|300,设置窗体标题栏显示名称gjsy
root@dwj:/bin# ffplay -i 1.mp4 -x 500 -y 300 -window_title "gjsy"

//nodisp窗体将被隐藏，但声音正常能播放，noborder窗体无边框显示，fs全屏显示
root@dwj:/bin# ffplay -i 1.mp4 -fs -nodisp -noborder

//ffmpeg取消控制台打印信息，不打印log日志
root@dwj:/bin# ffmpeg -y -vsync 0 -c:v h264_cuvid -i /opt/bin/input.mp4 output.yuv -loglevel quiet

//ffmpeg不打印log日志代码实现
```
#include "libavutil/log.h"
av_log_set_level(AV_LOG_QUIET);
```

//查看ffmpeg支持能力
```
ffmpeg -h encoder=h264_nvenc
ffmpeg -h decoder=h264_cuvid
ffmpeg -decoders |grep "cuvid"
ffmpeg -formats|grep "mp2"
```

<font color=#FF0000 size=4> <p align="center">Opencv</p></font>
