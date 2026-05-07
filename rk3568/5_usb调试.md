###### usb驱动配置

```
dmesg | grep -E "ehci|ohci|USB"
```

```
#配置好usb相关驱动
yxw@yxw-virtual-machine:~/myproject/rk356x_linux5.1/kernel$ grep -E "CONFIG_USB=y|CONFIG_USB_EHCI_HCD=y|CONFIG_USB_OHCI_HCD=y" .config
CONFIG_USB=y
CONFIG_USB_EHCI_HCD=y
CONFIG_USB_OHCI_HCD=y
yxw@yxw-virtual-machine:~/myproject/rk356x_linux5.1/kernel$ grep CONFIG_USB_STORAGE .config
CONFIG_USB_STORAGE=y
# CONFIG_USB_STORAGE_DEBUG is not set
# CONFIG_USB_STORAGE_REALTEK is not set
CONFIG_USB_STORAGE_DATAFAB=y
CONFIG_USB_STORAGE_FREECOM=y
CONFIG_USB_STORAGE_ISD200=y
CONFIG_USB_STORAGE_USBAT=y
CONFIG_USB_STORAGE_SDDR09=y
CONFIG_USB_STORAGE_SDDR55=y
CONFIG_USB_STORAGE_JUMPSHOT=y
CONFIG_USB_STORAGE_ALAUDA=y
CONFIG_USB_STORAGE_ONETOUCH=y
CONFIG_USB_STORAGE_KARMA=y
CONFIG_USB_STORAGE_CYPRESS_ATACB=y
CONFIG_USB_STORAGE_ENE_UB6250=y
yxw@yxw-virtual-machine:~/myproject/rk356x_linux5.1/kernel$ grep CONFIG_USB_HID .config
CONFIG_USB_HID=y
CONFIG_USB_HIDDEV=y
yxw@yxw-virtual-machine:~/myproject/rk356x_linux5.1/kernel$
```

```
#插上u盘后新出现日志
dmesg | grep -E "ehci|ohci|USB"

[  489.551506] usb 1-1: new high-speed USB device number 2 using ehci-platform
[  489.700210] usb 1-1: New USB device found, idVendor=0781, idProduct=5591, bcdDevice= 1.00
[  489.700255] usb 1-1: New USB device strings: Mfr=1, Product=2, SerialNumber=3
[  489.700278] usb 1-1: Manufacturer:  USB
[  489.701620] usb-storage 1-1:1.0: USB Mass Storage device detected
[  490.716595] scsi 1:0:0:0: Direct-Access      USB      SanDisk 3.2Gen1 1.00 PQ: 0 ANSI: 6

#new high-speed USB device：U 盘被识别为高速设备（符合 USB 2.0 标准）；
#sd 0:0:0:0: [sda]：U 盘被分配为块设备 sda；
#sda: sda1：U 盘包含一个分区 sda1。

```



```
#U 盘设备：/dev/sda（总容量 60GB）；
#U 盘分区：/dev/sda1（NTFS 格式，可挂载分区，标记为引导分区）。

root@rk3568-buildroot:/# fdisk -l
Found valid GPT with protective MBR; using GPT

Disk /dev/mmcblk0: 15269888 sectors, 3360M
Logical sector size: 512
Disk identifier (GUID): 3f4c0000-0000-462d-8000-31f4000026f7
Partition table holds up to 128 entries
First usable sector is 34, last usable sector is 15269854

Number  Start (sector)    End (sector)  Size Name
     1           16384           24575 4096K uboot
     2           24576           32767 4096K misc
     3           32768          163839 64.0M boot
     4          163840          425983  128M recovery
     5          425984          491519 32.0M backup
     6          491520        13074431 6144M rootfs
     7        13074432        13336575  128M oem
     8        13336576        15269823  943M userdata
Disk /dev/sda: 60 GB, 63909113344 bytes, 124822487 sectors
7769 cylinders, 255 heads, 63 sectors/track
Units: sectors of 1 * 512 = 512 bytes

Device  Boot StartCHS    EndCHS        StartLBA     EndLBA    Sectors  Size Id Type
/dev/sda1 *  90,61,36    1023,254,63    1449728  124822486  123372759 58.8G  7 HPFS/NTFS
root@rk3568-buildroot:/#
```

###### 检测

```
# 列出 U 盘中的文件（访问实际挂载路径 /mnt/udisk）
ls /mnt/udisk

# 在 U 盘中创建测试文件
echo "RK3568 USB NTFS Test Success" > /mnt/udisk/rk_test.txt

# 读取测试文件，确认写入成功
cat /mnt/udisk/rk_test.txt

#卸载U盘
umount /mnt/udisk

```

