wifi请求时钟，主板pcie发送时钟。

```
WL_REG_ON_GPIO2_B1
WL_HOST_WAKE_GPIO2_B2

4G_RST_GPIO3_D0
4G_DISABLE_GPIO3_D3

BT_REG_ON_GPIO2_B7
BT_WAKE_GPIO2_C1
BT_HOST_WAKE_GPIO2_C0


UART1_RTSN_M0		GPIO2_B5
```



```
PCIE20_WAKEN_M1 => GPIO2_D1
PCIE20_CLKREQN_M1 => GPIO2_D0

WL_HOST_WAKE => GPIO2_B2
WL_REG_ON => GPIO2_B1
PCIE20_REFCLKN => 
PCIE20_REFCLKP =>

WL_GPIO_2 => GPIO3_D3
WL_GPIO_1 => GPIO3_D0
PCIE20_RXN =>
PCIE20_RXP =>
PCIE20_TXN =>
PCIE20_TXP =>

// 下面四个 测试引脚，并没有接出来
BT_PCM_CLK => 
BT_PCM_OUT =>
BT_PCM_IN =>
BT_PCM_SYNC =>


BT_HOST_WAKE => GPIO2_C0
BT_WAKE => GPIO2_C1
BT_REG_ON => GPIO2_B7
PCIE_PME_L => GPIO3_D1

4G_DISABLE_GPIO4_C2 => GPIO4_C2
PCIE20_PERSTN_M1 => GPIO3_C1
UART1_RTSN_M0 => GPIO2_B5
UART1_CTSN_M0 => GPIO2_B6

USB2_HOST3_MINIPCIE_DM =>
USB2_HOST3_MINIPCIE_DP =>

UART1_RX_M0 => GPIO2_B3
UART1_TX_M0 => UART1_TX_M0
```



```
echo 0 > /sys/class/rfkill/rfkill0/state
echo 0 > /proc/bluetooth/sleep/btwrite
sleep 0.2
echo 1 > /sys/class/rfkill/rfkill0/state
echo 1 > /proc/bluetooth/sleep/btwrite
sleep 0.2
brcm_patchram_plus1 --bd_addr_rand --enable_hci --no2bytes --use_baudrate_for_download --tosleep 200000 --baudrate 1500000 --patchram /system/etc/firmware/BCM4362A2.hcd /dev/ttyS1 &

hciconfig -a

# 确保hci0已启动
hciconfig hci0 up
# 开启扫描模式
hciconfig hci0 piscan
#关闭扫描模式
hciconfig hci0 noscan
# 扫描周围蓝牙设备
hcitool scan

hciconfig hci0 leadv 3

bluetoothctl
# 在bluetoothctl中执行：
#   power on			#启动
#	discoverable on		#可发现模式
#	pairable on			#配对模式
#   agent on			#代理
#   scan on				#扫描
#   devices
#	exit
#	default-agent		#设置当前代理为默认

# 查看蓝牙控制器信息
root@rk3568-buildroot:/# hcitool dev
Devices:
	hci0	77:F2:D3:51:F5:F7
root@rk3568-buildroot:/# 

	#关闭BT电源，等同于rfkill block操作
echo 0 > /sys/class/rfkill/rfkill0/state 
echo 0 > /proc/bluetooth/sleep/btwrite
killall brcm_patchram_plus1 #务必要杀掉brcm_patchram_plus1进程，因为打开时会再次执行，否则会冲突


/etc/init.d/S99bluetooth
#!/bin/sh
echo 0 > /sys/class/rfkill/rfkill0/state # 下电
echo 0 > /proc/bluetooth/sleep/btwrite
sleep 1
killall brcm_patchram_plus1
sleep 2
echo 1 > /sys/class/rfkill/rfkill0/state # 上电
echo 1 > /proc/bluetooth/sleep/btwrite
sleep 2

brcm_patchram_plus1 --bd_addr_rand --enable_hci --no2bytes --use_baudrate_for_download --tosleep 200000 --baudrate 1500000 --patchram /system/etc/firmware/BCM4362A2.hcd /dev/ttyS1 &

brcm_patchram_plus1 --bd_addr_rand --enable_hci --no2bytes --use_baudrate_for_download --tosleep 200000 --baudrate 3000000 --patchram /system/etc/firmware/BCM4362A2_001.003.006.1126.1235_CL1.hcd /dev/ttyS1 &
```



```
#寻找hcd固件
find /lib/firmware -type f -name "*.hcd"
dmesg | grep -i bluetooth
```





```
root@rk3568-buildroot:/# cat /sys/class/rfkill/rfkill0/state
0
root@rk3568-buildroot:/# echo 0 > /sys/class/rfkill/rfkill0/state
te
echo 1 > /proc/bluetooth/sleep/btwrite
sleep 0.2
brcm_patchra[   20.884016] [BT_RFKILL]: bt shut off power
m_plus1 --bd_addr_rand --enable_hci --no2bytes --use_baudrate_for_download --tosleep 200000 --baudrate 1500000 --patchram /system/etc/firmware/BCM4362A2.hcd /dev/ttyS1 &root@rk3568-buildroot:/# echo 0 > /proc/bluetooth/sleep/btwrite
root@rk3568-buildroot:/# sleep 0.2
root@rk3568-buildroot:/# echo 1 > /sys/class/rfkill/rfkill0/state
[   21.120245] [BT_RFKILL]: rfkill_rk_set_power: set bt wake_host high!
[   21.145521] [BT_RFKILL]: rfkill_rk_set_power: set bt wake_host input!
[   21.145604] [BT_RFKILL]: ENABLE UART_RTS
[   21.252192] [BT_RFKILL]: DISABLE UART_RTS
[   21.252306] [BT_RFKILL]: bt turn on power
[   21.252353] [BT_RFKILL]: Request irq for bt wakeup host
root@rk3568-buildroot:/# [   21.252413] [BT_RFKILL]: ** disable irq
echo 1 > /proc/bluetooth/sleep/btwrite
root@rk3568-buildroot:/# sleep 0.2

system/etc/firmware/BCM4362A2.hcd /dev/ttyS1 &00 --baudrate 1500000 --patchram /s
[1] 962
root@rk3568-buildroot:/# ###AMPAK FW Auto detection patch version = [v1.1 20161117]###
cp: cannot stat '/data/cfg/device_info.txt': No such file or directory
FW folder path = /system/etc/firmware
[   21.980865] of_dma_request_slave_channel: dma-names property of node '/serial@fe650000' missing or empty
[   21.980922] dw-apb-uart fe650000.serial: failed to request DMA, use interrupt mode
proc_resetchip id = BCM4362A2
FW path = /system/etc/firmware/BCM4362A2.hcd
send hci_download_minidriverproc_resetDone setting line discpline
N: [pulseaudio] bluez5-util.c: Could not find org.bluez.BatteryProviderManager1.RegisterBatteryProvider(), is bluetoothd started with experimental features enabled (-E flag)?

root@rk3568-buildroot:/# hciconfig -a
hci0:	Type: Primary  Bus: UART
	BD Address: 62:4E:28:10:EB:8C  ACL MTU: 1021:8  SCO MTU: 64:1
	UP RUNNING 
	RX bytes:902 acl:0 sco:0 events:65 errors:0
	TX bytes:3339 acl:0 sco:0 commands:65 errors:0
	Features: 0xbf 0xfe 0xcf 0xfe 0xdb 0xff 0x7b 0x87
	Packet type: DM1 DM3 DM5 DH1 DH3 DH5 HV1 HV2 HV3 
	Link policy: RSWITCH SNIFF 
	Link mode: PERIPHERAL ACCEPT 
	Name: 'BlueZ 5.68'
	Class: 0x6c0000
	Service Classes: Rendering, Capturing, Audio, Telephony
	Device Class: Miscellaneous, 
	HCI Version:  (0xc)  Revision: 0x495
	LMP Version:  (0xc)  Subversion: 0x1111
	Manufacturer: Broadcom Corporation (15)

root@rk3568-buildroot:/#
```



###### 连接WiFi

```
ifconfig wlan0 up
iw dev wlan0 scan		#确认有无BSS（AP）输出
wpa_passphrase "YNKJ-5G" "ynkj88888888" >> /etc/wpa_supplicant.conf
wpa_supplicant -B -i wlan0 -c /etc/wpa_supplicant.conf
udhcpc -i wlan0			#通过DHCP协议自动获取IP地址、网关、DNS等网络配置
ifconfig wlan0

#成功能看到
inet addr:192.168.1.105  Bcast:192.168.1.255  Mask:255.255.255.0

#避免eth0路由干扰
route del default dev eth0 2>/dev/null

wpa_cli -i wlan0 status		#检查当前网络状态
ps aux | grep wpa_supplicant
killall wpa_supplicant
wpa_supplicant -B -i wlan0 -c /etc/wpa_supplicant.conf

#扫描周边ap		好：-20到-50，偏弱：-50到-70，差：-70到-90
wpa_cli -i wlan0 -p /var/run/wpa_supplicant scan
wpa_cli -i wlan0 -p /var/run/wpa_supplicant scan_results
```









hci0解释

```
hci0:	Type: Primary  Bus: UART
	BD Address: 62:4E:28:10:EB:8C  ACL MTU: 1021:8  SCO MTU: 64:1
		#Asynchronous Connection-Less和Synchronous Connection-Oriented
		#MTU网络最大传输单元
		#ACL MTU	1021:8 表示主机/控制器的 ACL 数据包最大有效载荷（字节）
		#SCO MTU	表示语音（SCO）包的大小，SCO 用于语音、HFP 音频路径（窄延时）
	UP RUNNING PSCAN ISCAN
		#UP RUNNING：网络接口层面已启用并正常运行（HCI 已启动）
		#PSCAN = Page Scan（是否响应 page 请求以建立连接）
		#ISCAN = Inquiry Scan（是否响应 inquiry 扫描，以便被发现）
		#noscan（都关闭），pscan（可连接不可发现），iscan（被发现不连接），piscan（被发现被连接）
	RX bytes:902 acl:0 sco:0 events:65 errors:0
	TX bytes:3339 acl:0 sco:0 commands:65 errors:0
		#统计信息：接收/发送的字节数、ACL/SCO 包计数、事件与错误数。用于判断链路是否有丢包或高错误率。
	Features: 0xbf 0xfe 0xcf 0xfe 0xdb 0xff 0x7b 0x87
		#位掩码，表示控制器支持的一组 HCI 特性（Encryption、Three-slot packets、Slot offset、Interlaced、Secure Simple Pairing、LE 支持等）。需参考特定 Bluetooth Core Spec 的 Feature bits 表
	Packet type: DM1 DM3 DM5 DH1 DH3 DH5 HV1 HV2 HV3 
		#支持的 基础包类型（BR/EDR）及 SCO：
		#DMx/DHx：DM = 数据包（with encryption?），DH = 数据高速包（no FEC），数字越大表示支持三/五时隙包，影响吞吐量与延迟。
		#HVx：SCO（语音）包类型（HV1/HV2/HV3）。
		#模式/影响：是否支持 3-slot/5-slot 决定单连接时的吞吐能力；SCO 类型决定语音延迟与可靠性。
	Link policy: RSWITCH SNIFF 
		#Link policy：控制连接时主从如何管理链路、节电策略等。常见选项：
		#RSWITCH（Role Switch allowed）— 允许角色切换（从/主的切换）。
		#SNIFF — 支持 SNIFF 模式（低功耗周期性轮询以节省能量）。
		#······
		#切换/使用：策略通常在建立连接或通过 L2CAP/HCI 命令设置（由蓝牙堆栈自动或应用触发）。
	Link mode: PERIPHERAL ACCEPT 
		#Link mode 描述设备角色与连接接受策略
		#PERIPHERAL：常用于 BLE 周边角色（在经典 BR/EDR 中也有“role”概念，表明当前行为是从设备或主设备）。
		#ACCEPT：表示接受来自远端的连接请求（如果配合 PSCAN/ISCAN）。
		#在 BLE 中常见的角色有 PERIPHERAL / CENTRAL（但在 BR/EDR 语境下也有 Master/Slave 或 Role Switch）。
	Name: 'BlueZ 5.68'
		#设备名字	更改：hciconfig hci0 name 'MyDevice'
	Class: 0x6c0000
	Service Classes: Rendering, Capturing, Audio, Telephony
	Device Class: Miscellaneous, 
		#lass of Device (CoD)：一个 24-bit 值，表示设备类别（比如音频、电话、电脑等）与提供的服务类别。远端扫描时会看到这个分类，有助于过滤设备类型。
		#通过 hciconfig hci0 class <hex> 设置（谨慎使用）。
	HCI Version:  (0xc)  Revision: 0x495
		#HCI Version：控制器实现的 Bluetooth Core Spec 版本（比如 0xc = 12 表示 Bluetooth 4.x/5.x 的某个版本，需对照规范表）。
		#Revision：芯片厂商的内部固件/硬件修订号。
	LMP Version:  (0xc)  Subversion: 0x1111
		#LMP Version：Link Manager Protocol 版本，与 HCI Version 相关，表示控制器支持的协商级别。
		#Subversion：厂商特定的子版本号。
	Manufacturer: Broadcom Corporation (15)
		#控制器厂商标识符（这里是 Broadcom）。有助于判断使用哪套 vendor 驱动/固件（例如 .hcd 文件）。

```





```
&wireless_wlan {
	pinctrl-names = "default";
	pinctrl-0 = <&wifi_host_wake_irq>;
	wifi_chip_type = "ap6275p";
	WIFI,poweren_gpio = <&gpio2 RK_PB1 GPIO_ACTIVE_HIGH>;
	WIFI,host_wake_irq = <&gpio3 RK_PD4 GPIO_ACTIVE_HIGH>;
	wl_reg_on-gpios = <&gpio2 RK_PB1 GPIO_ACTIVE_HIGH>;
	wl_host_wake-gpios = <&gpio2 RK_PB2 GPIO_ACTIVE_HIGH>;
	//ref_wifi_clk = <&cru CLK_PCIE20_AUX_NDFT>;
	 ref_wifi_clk = <&pcie2x1 4>;
};
```





```
root@rk3568-buildroot:/# dmesg | grep wlan
[    3.364856] [WLAN_RFKILL]: Enter rfkill_wlan_init
[    3.365609] [WLAN_RFKILL]: Enter rfkill_wlan_probe
[    3.365668] [WLAN_RFKILL]: wlan_platdata_parse_dt: wifi_chip_type = ap6275p
[    3.365679] [WLAN_RFKILL]: wlan_platdata_parse_dt: wifi power remain
[    3.365688] [WLAN_RFKILL]: wlan_platdata_parse_dt: wifi power controled by gpio.
[    3.365716] [WLAN_RFKILL]: wlan_platdata_parse_dt: WIFI,poweren_gpio = 73 flags = 0.
[    3.365746] [WLAN_RFKILL]: wlan_platdata_parse_dt: WIFI,host_wake_irq = 74, flags = 0.
[    3.365760] [WLAN_RFKILL]: wlan_platdata_parse_dt: The ref_wifi_clk not found !
[    3.365769] [WLAN_RFKILL]: rfkill_wlan_probe: init gpio
[    3.475998] [WLAN_RFKILL]: Exit rfkill_wlan_probe
[    7.214753] [dhd] STATIC-MSG) dhd_static_buf_init : 101.10.361.31 (wlan=r892223-20230427-1)
[    7.215281] [dhd] STATIC-MSG) dhd_init_wlan_mem : prealloc ok for index 0: 2369536(2314K)
[    7.215478] [dhd] dhd_wlan_init_gpio: WL_HOST_WAKE=-1, oob_irq=126, oob_irq_flags=0x4
[    7.215497] [dhd] dhd_wlan_init_gpio: WL_REG_ON=-1
[    7.216102] [dhd] STATIC-MSG) dhd_wlan_mem_prealloc : section 3, size 139264
[    7.216174] [dhd] STATIC-MSG) dhd_wlan_mem_prealloc : section 4, size 0
[    7.219629] [dhd] STATIC-MSG) dhd_wlan_mem_prealloc : section 7, size 43776
[    7.220835] [dhd] STATIC-MSG) dhd_wlan_mem_prealloc : section 0, size 5152
[    7.306116] [dhd] STATIC-MSG) dhd_wlan_mem_prealloc : section 5, size 65536
[    7.312709] [dhd] STATIC-MSG) dhd_wlan_mem_prealloc : section 19, size 65688
[    7.559498] [dhd] STATIC-MSG) dhd_wlan_mem_prealloc : section 9, size 32896
[    7.925674] [dhd] Register interface [wlan0]  MAC: 9c:b8:b4:5f:0a:82
[    7.926365] [dhd] [wlan0] wl_android_wifi_off :  g_wifi_on=1 force_off=1
[    7.931688] [dhd] [wlan0] wl_android_wifi_off : out
[    7.934288] [dhd] Register interface [wlan1]  MAC: 9e:b8:b4:5f:0a:82
[    8.213613] [dhd] [wlan0] dhd_open : Enter
[    8.213671] [dhd] [wlan0] wl_android_wifi_on : in g_wifi_on=0
[    8.747486] [dhd] STATIC-MSG) dhd_wlan_mem_prealloc : section 9, size 32896
[    8.988698] [dhd] [wlan0] wl_android_wifi_on : Success
[    9.048618] [dhd] [wlan0] wl_cfg80211_up : Roam channel cache enabled
[    9.052539] [dhd] [wlan0] dhd_open : Exit ret=0
[    9.052575] [dhd] [wlan0] dhd_pri_open : tx queue started
[    9.052630] [dhd] [wlan0] custom_xps_map_set : Done. mapping cpu




root@rk3568-buildroot:/# wpa_passphrase "YNKJ-5G" "ynkj88888888" > /etc/wpa_supplicant.conf
root@rk3568-buildroot:/# wpa_supplicant -B -i wlan0 -c /etc/wpa_supplicant.conf
Successfully initialized wpa_supplicant
		#wpa_supplicant 启动成功
nl80211: kernel reports: Match already configured
		#那些重复的信息只是内核在注册 NL80211 匹配项时的提示，不影响连接，常见于部分 dhd 驱动版本。
nl80211: kernel reports: Match already configured
nl80211: kernel reports: Match already configured
nl80211: kernel reports: Match already configured
nl80211: kernel reports: Match already configured
nl80211: kernel reports: Match already configured
nl80211: kernel reports: Match already configured
nl80211: kernel reports: Match already configured
nl80211: kernel reports: Match already configured
nl80211: kernel reports: Match already configured
nl80211: kernel reports: Match already configured
nl80211: kernel reports: Match already configured
nl80211: kernel reports: Match already configured
nl80211: kernel reports: Match already configured
nl80211: kernel reports: Match already configured
nl80211: kernel reports: Match already configured
nl80211: kernel reports: Match already configured
nl80211: kernel reports: Match already configured
nl80211: kernel reports: Match already configured
nl80211: kernel reports: Match already configured
nl80211: kernel reports: Match already configured
nl80211: kernel reports: Match already configured
nl80211: kernel reports: Match already configured
nl80211: kernel reports: Match already configured
nl80211: kernel reports: Registration to specific type not supported
root@rk3568-buildroot:/# udhcpc -i wlan0
udhcpc: started, v1.36.0
udhcpc: broadcasting discover
udhcpc: broadcasting select for 192.168.1.29, server 192.168.1.1
udhcpc: lease of 192.168.1.29 obtained from 192.168.1.1, lease time 86400
		#DHCP 成功分配 IP
		#表明 WLAN 已成功关联（association success）并完成了 IP 获取流程。
deleting routers
adding dns 192.168.1.1
root@rk3568-buildroot:/# ifconfig wlan0
wlan0     Link encap:Ethernet  HWaddr 9C:B8:B4:5F:0A:82  
          inet addr:192.168.1.29  Bcast:192.168.1.255  Mask:255.255.255.0
          inet6 addr: fe80::641a:2b2f:3a22:b9a1/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          #网络已 UP，链路稳定。
          RX packets:11 errors:0 dropped:0 overruns:0 frame:0
          TX packets:6 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000 
          RX bytes:1258 (1.2 KiB)  TX bytes:1112 (1.0 KiB)

root@rk3568-buildroot:/#
```

