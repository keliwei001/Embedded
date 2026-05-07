`uname -m`：Linux查看系统



`ls /dev/mmcblk*`	//存储

`dmesg | grep -i "mmc\|sdhci\|tf"`	//查看sdmmc相关日志



`root@rk3568-buildroot:/# cat /sys/class/graphics/fb0/modes`
`U:1200x1920p-0`	//表示支持 1200×1920 像素的分辨率
`root@rk3568-buildroot:/# cat /sys/class/graphics/fb0/virtual_size`
`1200,1920`		//与实际分辨率一致，说明无虚拟扩展。

