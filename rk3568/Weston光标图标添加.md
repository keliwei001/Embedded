weston合成器在加载拖放操作的光标图标文件遇到问题

```
could not load cursor 'dnd-move'
could not load cursor 'dnd-copy'
could not load cursor 'dnd-none'
could not load cursor 'dnd-move'
could not load cursor 'dnd-copy'
could not load cursor 'dnd-none'
```



分析：

weston没找到相关光标图标

缺少`dnd-move`, `dnd-copy`, `dnd-none` 等光标主题文件

光标图标文件未正确安装或路径配置错误

```
root@rk3568-buildroot:/# find /usr -name "dnd-*" -type f
/usr/share/icons/Adwaita/cursors/dnd-no-drop
/usr/share/icons/Adwaita/cursors/dnd-move
/usr/share/icons/Adwaita/cursors/dnd-ask
/usr/share/icons/Adwaita/cursors/dnd-link
/usr/share/icons/Adwaita/cursors/dnd-none
/usr/share/icons/Adwaita/cursors/dnd-copy
root@rk3568-buildroot:/# find /usr -name "*cursor*" -type d
/usr/share/icons/whiteglass/cursors
/usr/share/icons/redglass/cursors
/usr/share/icons/handhelds/cursors
/usr/share/icons/Adwaita/cursors
root@rk3568-buildroot:/# 
root@rk3568-buildroot:/# root@rk3568-buildroot:/# ls /usr/share/icons/ 		Adwaita hicolor icon_glmark2.png icon_simple-egl.png whiteglass handhelds icon_camera.png icon_player.png redglass

# 创建符号链接到正确位置
mkdir -p /usr/share/icons/Adwaita/cursors/
ln -sf /path/to/actual/cursors/* /usr/share/icons/Adwaita/cursors/

```



添加该文件，

```
buildroot/board/rockchip/rk3566_rk3568/fs-overlay/etc/xdg/weston/weston.ini
```

```
yxw@yxw-virtual-machine:~/myproject/rk356x_linux5.1$ cat buildroot/board/rockchip/rk3566_rk3568/fs-overlay/etc/xdg/weston/weston.ini 
[core]
backend=drm-backend.so

# Allow running without input devices
require-input=false

# Allow running without output devices
require-outputs=none

# Disable screen idle timeout by default
idle-time=0

# The repaint-window is used to calculate repaint delay(ms) after flipped.
#   value <= 0: delay = abs(value)
#   value > 0: delay = vblank_duration - value
repaint-window=-1

# Allow blending with lower drm planes
# gbm-format=argb8888

[shell]
# top(default)|bottom|left|right|none, none to disable panel
# panel-position=none

cursor-theme=Adwaita

# Scale panel size
panel-scale=3

# Set cursor size
cursor-size=32

# none|minutes(default)|minutes-24h|seconds|seconds-24h
# clock-format=minutes-24h
clock-with-date=false

# Disable screen locking
locking=false

[libinput]
# Uncomment below to enable touch screen calibrator(weston-touch-calibrator)
# touchscreen_calibrator=true
# calibration_helper=/bin/weston-calibration-helper.sh

[keyboard]
# Comment this to enable vt switching
vt-switching=false

# Configs for auto key repeat
# repeat-rate=40
# repeat-delay=400
```



添加该文件

```
buildroot/board/rockchip/rk3566_rk3568/fs-overlay/etc/init.d/S99weston 
chmod +x buildroot/board/rockchip/rk3566_rk3568/fs-overlay/etc/init.d/S99weston 
```

```
yxw@yxw-virtual-machine:~/myproject/rk356x_linux5.1$ cat buildroot/board/rockchip/rk3566_rk3568/fs-overlay/etc/init.d/S99weston 
#!/bin/sh
PATH=/sbin:/bin:/usr/sbin:/usr/bin

export XDG_RUNTIME_DIR=/tmp/xdg
mkdir -p $XDG_RUNTIME_DIR
chmod 700 $XDG_RUNTIME_DIR

export XCURSOR_THEME=Adwaita
export XCURSOR_SIZE=32
export XCURSOR_PATH=/usr/share/icons

start-stop-daemon --start --background --make-pidfile --pidfile /var/run/weston.pid \
  --exec /usr/bin/weston -- --backend=drm-backend.so --use-pixman

yxw@yxw-virtual-machine:~/myproject/rk356x_linux5.1$ 








yxw@yxw-virtual-machine:~/myproject/rk356x_linux5.1/buildroot/board/rockchip/rk3566_rk3568/fs-overlay/etc/init.d$ cat S99weston 
#!/bin/sh
PATH=/sbin:/bin:/usr/sbin:/usr/bin

export XDG_RUNTIME_DIR=/tmp/xdg
mkdir -p $XDG_RUNTIME_DIR
chmod 700 $XDG_RUNTIME_DIR

export XCURSOR_THEME=Adwaita
export XCURSOR_SIZE=32
export XCURSOR_PATH=/usr/share/icons

echo "[S99weston] Starting weston with cursor theme=$XCURSOR_THEME, size=$XCURSOR_SIZE"
echo "[S99weston] XCURSOR_PATH=$XCURSOR_PATH" 

# 直接用 'env' 显式传递变量（避免丢失）
env \
  XDG_RUNTIME_DIR=$XDG_RUNTIME_DIR \
  XCURSOR_THEME=$XCURSOR_THEME \
  XCURSOR_SIZE=$XCURSOR_SIZE \
  XCURSOR_PATH=$XCURSOR_PATH \
  start-stop-daemon --start --background --make-pidfile --pidfile /var/run/weston.pid \
  --exec /usr/bin/weston -- --backend=drm-backend.so --use-pixman --log=/tmp/weston.log








```



问题

```
创建的weston.ini文件不生效
yxw@yxw-virtual-machine:~/myproject/rk356x_linux5.1/buildroot$ find -name weston.ini
./board/rockchip/common/overlays/10-weston/etc/xdg/weston/weston.ini
./board/rockchip/rk3566_rk3568/fs-overlay/etc/xdg/weston/weston.ini
./output/rockchip_rk3568/target/etc/xdg/weston/weston.ini
./output/rockchip_rk3568/build/weston-13.0.0/build/ivi-shell/weston.ini
./output/rockchip_rk3568/build/weston-13.0.0/build/compositor/weston.ini
./output/rockchip_rk3568_recovery/target/etc/xdg/weston/weston.ini

Buildroot 在打包 rootfs 时，先合并了你的 fs-overlay，然后又被 10-weston 覆盖掉。

解决：
创建buildroot/board/rockchip/rk3566_rk3568/post-build.sh
chmod +x board/rockchip/rk3566_rk3568/post-build.sh
编译后检测
buildroot/configs/rockchip_rk3568_defconfig中有类似内容：BR2_ROOTFS_POST_BUILD_SCRIPT="board/rockchip/rk3566_rk3568/post-build.sh"
没有则加上

在buildroot/board/rockchip/common/post-build.sh文件末尾加上配置补丁
# --- Weston 配置补丁 ---
echo ">>> Copying weston.ini from rk3566_rk3568 overlay"
if [ -f board/rockchip/rk3566_rk3568/fs-overlay/etc/xdg/weston/weston.ini ]; then
    install -Dm0644 "board/rockchip/rk3566_rk3568/fs-overlay/etc/xdg/weston/weston.ini" \
        "${TARGET_DIR}/etc/xdg/weston/weston.ini"
fi

cat output/rockchip_rk3568/target/etc/xdg/weston/weston.ini | grep cursor
应输出：
cursor-theme=Adwaita
cursor-size=32
```

