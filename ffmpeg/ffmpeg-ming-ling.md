# ffmpeg命令

### 命令分类

* 基本信息查询

  -version, -demuxers, -muxers, -devices, -codes, -decoders, -encoders, -bsfs, -formats, -protocols, -filters, -pix_fmts,_ sample\_fmts, -layouts, -colors.

* 录制命令

  ```text
  // 录制
  ffmpeg -f avfoundation -i 1 -r 30 out.yuv
  ffmpeg -f avfoundation -i :0 out.wav

  // 查看支持的设备
  ffmpeg -f avfoundation -list_devices true -i ""
  ```

* 分解/复用

  ```text
  ffmpeg -i out.mp4 -vcodec copy -acodec copy out.flv
  ffmpeg -i f35.mov -an -vcodec copy out.264
  ffmpeg -i f35.mov -vn -acodec copy out.aac
  ```

* 处理原始数据

  ```text
  ffmpeg -i f35.mp4 -an -c:v rawvideo -pix_fmt yuv420p out.yuv
  ffmpeg -i out.mp4 -vn -ar 44100 -ac 2 -f s16le out.pcm
  ```

* 裁剪/合并

  ```text
  // 裁剪
  ffmpeg -i f35.mp4 -ss 00:00:10 -t 10 out.ts

  // 合并
  ffmpeg -f concat -i inputs.txt out.flv
  inputs.txt 内容为 'file filename'格式， 合并多个文件
  inputs.txt内容
  file '1.ts'
  file '2.ts'
  ```

* 图像/视频

  ```text
  // 视频转图片
  ffmpeg -i in.flv -r 1 -f image2 image-%3d.jpeg 

  // 图片转视频
  ffmpeg -i imge-%3d.jpeg out.mp4
  ```

* 直播

  ```text
  ffmpeg -re -i out.mp4 -c copy -f flv 
  flv rtmp://server/live/streamName
  ```

* 滤镜\(水印，画中画，倍数\)

  ```text
  ffmpeg -i f35.mp4 -vf crop=in_w-200:in_h-200
  -c:v libx264 -c:a copy out.mp4
  ```

