# FFmpeg  
- 查看媒体信息  
ffmpeg -hide_banner -i input.mp4   
只查看不转码（会打印流信息）  
- 仅换封装容器（不转码，最快）
ffmpeg -i input.mov -c copy output.mp4
  
 - 指定编码器转码
ffmpeg -i input.mkv -c:v libx264 -c:a aac output.mp4  
  常见硬件编码器
H.264：-c:v h264_nvenc / h264_qsv / h264_amf / h264_videotoolbox
H.265：-c:v hevc_nvenc / hevc_qsv / hevc_videotoolbox