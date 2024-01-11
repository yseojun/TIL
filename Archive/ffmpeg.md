#### 비디오 오디오 분리
```sh
ffmpeg -i input.mp4 -c:v copy -an video.mp4 -c:a libmp3lame audio.mp3
```
---
#### MPEG-DASH 생성
```sh
ffmpeg -i input.mp4 -c:v libx264 -b:v 5M -s 1920x1080 -profile:v high -level 4.2 -c:a aac -b:a 192k output_1080p.mp4
ffmpeg -i input.mp4 -c:v libx264 -b:v 1M -s 854x480 -profile:v main -level 3.1 -c:a aac -b:a 96k output_480p.mp4


mp4fragment output_480p.mp4 output_480p_f.mp4
mp4fragment output_480p.mp4 output_720p_f.mp4
mp4fragment output_480p.mp4 output_1080p_f.mp4

mp4dash -o video_dash output_480p_f.mp4 output_720p_f.mp4 output_1080p_f.mp4
```
