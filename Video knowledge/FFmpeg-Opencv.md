<font color=#FF0000 size=4> <p align="center">FFmpeg、FFprobe、FFplay</p></font>

//读取本地视频文件推理至媒体服务器
root@dwj:/bin# ffmpeg -re -i "/root/123.mp4" -vcodec copy -f flv "rtmp://10.3.9.106:1935/live/2"

//使用硬件加速解码文件
root@dwj:/bin# ffmpeg -y -vsync 0 -hwaccel cuda -hwaccel_output_format cuda -i input.mp4 -c:a copy -c:v h264_nvenc -b:v 5M output.mp4

//硬件解码视频文件到原始格式视频
root@dwj:/bin# ffmpeg -y -vsync 0 -c:v h264_cuvid -i /opt/bin/input.mp4 output.yuv

//编码原始格式视频到其他格式
root@dwj:/bin# ffmpeg -y -vsync 0 -s 640x480 -i output.yuv -c:v h264_nvenc outputxx.mp4

//查看ffmpeg支持能力
```
ffmpeg -h encoder=h264_nvenc
ffmpeg -h decoder=h264_cuvid
ffmpeg -decoders |grep "cuvid"
ffmpeg -formats|grep "mp2"
```

<font color=#FF0000 size=4> <p align="center">Opencv</p></font>
