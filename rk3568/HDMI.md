

```
检测HDMI连接
cat /sys/class/drm/card0-HDMI-A-1/status

# 查看当前显示状态
cat /sys/class/drm/card0-HDMI-A-1/enabled
cat /sys/class/drm/card0-HDMI-A-1/modes

# 检查DRM设备
ls /dev/dri/

# 查看VOP2状态
dmesg | grep vop2
cat /sys/kernel/debug/dri/0/state

# 确保显示相关电源域正常
cat /sys/kernel/debug/pm_genpd/pm_genpd_summary

# 设置不同的分辨率测试
echo 1280x720 > /sys/class/drm/card0-HDMI-A-1/mode

手动关闭DSI显示
echo off > /sys/class/drm/card0-DSI-1/status

cat /sys/class/drm/card0-HDMI-A-1/modes
echo 2560x1600 > /sys/class/drm/card0-HDMI-A-1/mode

cat /sys/kernel/debug/dri/0/summary
cat /sys/kernel/debug/dw-hdmi/status

//查看视频信息
ffmpeg -i 3443801.mp4 2>&1 | grep -E "Stream.*Video"

gst-launch-1.0 filesrc location=344380.mp4 ! decodebin ! autovideosink

gst-launch-1.0 multifilesrc location=/mnt/usb/picture/pixnio-13430140-2560x1600.jpg loop=true \
  ! jpegparse ! mppjpegdec ! waylandsink
  
gst-launch-1.0 filesrc location=/mnt/usb/3443801.mp4 ! qtdemux ! h264parse ! mppvideodec ! videoconvert ! textoverlay text="2560x1600@90Hz / 4K@30fps" halignment=left valignment=top shaded-background=true ! autovideosink
  
gst-launch-1.0 filesrc location=/mnt/usb/344380.mp4 ! qtdemux ! h264parse ! mppvideodec ! videoscale ! video/x-raw,width=2560,height=1600 ! textoverlay text="2560x1600@90Hz / 4K@29.97fps" halignment=left valignment=top shaded-background=true ! waylandsink  

modetest -M rockchip


gst-launch-1.0 filesrc location=344380.mp4 ! \
    qtdemux ! h264parse ! mppvideodec ! \
    textoverlay text="2560x1600, 60 fps" \
        valignment=top halignment=left \
        font-desc="Sans, 10" \
        shaded-background=true \
        color=0xFFFFFF00 ! \
    waylandsink
    
    
    
gst-launch-1.0 filesrc location=344380.mp4 ! decodebin ! \
videoconvert ! textoverlay text="2560x1600@90Hz" valignment=top halignment=left font-desc="Sans, 10" ! \
autovideosink
```



情况：播放视频的时候左上角会显示屏幕分辨率等信息

```
gst-launch-1.0 filesrc location=344380.mp4 ! decodebin ! autovideosink
该模式下播放视频时硬件解码直出，能正常播放，
gst-launch-1.0 filesrc location=/mnt/usb/3443801.mp4 ! qtdemux ! h264parse ! mppvideodec ! videoconvert ! textoverlay text="2560x1600@90Hz / 4K@30fps" halignment=left valignment=top shaded-background=true ! autovideosink
该模式下播放因为要使用到textoverlay等插件，硬解码会转为cpu解码将相关文字渲染到视频上，cpu性能不够视频会卡

解决办法：
gst-launch-1.0 filesrc location=344380.mp4 ! decodebin ! autovideosink播放视频，播放分辨率用osd程序，具体用脚本实现
```

