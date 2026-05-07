

```
~/myproject/rk356x_linux5.1/buildroot/output/rockchip_rk3568/host/bin/aarch64-linux-gcc \
--sysroot=$SYSROOT \
-I$SYSROOT/usr/include \
-I$SYSROOT/usr/include/libdrm \
osd_demo.c -o osd_demo -ldrm


yxw@yxw-virtual-machine:~/myproject/osd/osd$ export SYSROOT=/home/yxw/myproject/rk356x_linux5.1/buildroot/output/rockchip_rk3568/host/aarch64-buildroot-linux-gnu/sysroot

~/myproject/rk356x_linux5.1/buildroot/output/rockchip_rk3568/host/bin/aarch64-linux-gcc \
    --sysroot=$SYSROOT \
    -I$SYSROOT/usr/include/libdrm \
    osd_demo.c \
    -o osd_demo \
    -ldrm

```



