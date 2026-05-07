环境检查

```
cd myproject/rk356x_linux5.1/buildroot/output/rockchip_rk3568/host
source set_qt_env.sh

which qmake
qmake -v
pkg-config --modversion Qt5Core
pkg-config --modversion Qt5Gui
pkg-config --modversion Qt5Widgets
which qtwaylandscanner
qmake -query
```



编译arm

```
创建cpp
编写pro文件
qmake .pro文件生成Makefile文件
	/home/yxw/myproject/rk356x_linux5.1/buildroot/output/rockchip_rk3568/host/bin/qmake
make编译生成app

lsblk
sudo mount /dev/sdb1 /mnt/usb/
ls /dev/mmcblk1*
mount -t vfat /dev/mmcblk1p1 /mnt/udisk
```



###### buildroot中添加图标

```
```





killall weston
weston &
