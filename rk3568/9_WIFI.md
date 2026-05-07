wifi请求时钟，主板pcie发送时钟

##### 开发

###### 硬件确认

```
#ap6275p，使用pcie2接口
WL_REG_ON_GPIO2_B1		#开机高电平
WL_HOST_WAKE_GPIO2_B2

#pcie2长供电，无需设备树配置
PCIE20_PERSTN_M1 	GPIO3_C1
PCIE20_WAKEN_M1		GPIO2_D1
PCIE20_CLKREQN_M1 	GPIO2_D0


	wireless-wlan {
		wifi_host_wake_irq: wifi-host-wake-irq {
			rockchip,pins = <2 RK_PB2 RK_FUNC_GPIO &pcfg_pull_none>;
		};
		wifi_reset_gpio: wifi_reset_gpio{
			rockchip,pins = <2 RK_PB1 RK_FUNC_GPIO &pcfg_pull_up>;
		};
		wifi_poweren_gpio: wifi_poweren_gpio{
			rockchip,pins = <2 RK_PB1 RK_FUNC_GPIO &pcfg_pull_up>;
		};
	};
	
&wireless_wlan {
	compatible = "wlan-platdata";
	pinctrl-names = "default";
	pinctrl-0 = <&wifi_host_wake_irq>;

	keep_wifi_power_on;
	WIFI,poweren_gpio = <&gpio2 RK_PB1 GPIO_ACTIVE_HIGH>;		//73
	WIFI,reset_gpio = <&gpio2 RK_PB1 GPIO_ACTIVE_HIGH>;			//73
	WIFI,host_wake_irq = <&gpio2 RK_PB2 GPIO_ACTIVE_HIGH>;		//74
};


&pcie2x1 {
	pinctrl-names = "default";
	//pinctrl-0 = <&pcie20m1_pins>;
	pinctrl-0 = <&pcie20_custom_reset>;
	//pinctrl-0 = <&pcie20m1_pins &pcie20_custom_reset>;
	//pinctrl-0 = <&sata_pins>;
	reset-gpios = <&gpio3 RK_PC1 GPIO_ACTIVE_HIGH>;		//113
	//vpcie3v3-supply = <&vcc3v3_pcie>;
	clkreq-gpios = <&gpio2 RK_PD0 GPIO_ACTIVE_LOW>;		//88
	wake-gpios = <&gpio2 RK_PD1 GPIO_ACTIVE_LOW>;		//89
	status = "okay";
};
```



###### 内核配置

```
CONFIG_RFKILL=y
CONFIG_RFKILL_RK=y
CONFIG_MMC=y
CONFIG_PWRSEQ_SIMPLE=y
CONFIG_MMC_DW=y
CONFIG_MMC_DW_PLTFM=y
CONFIG_MMC_DW_ROCKCHIP=y
CONFIG_WIRELESS=y
CONFIG_WIRELESS_EXT=y
CONFIG_WEXT_CORE=y
CONFIG_WEXT_PROC=y
CONFIG_WEXT_PRIV=y
CONFIG_CFG80211=y
CONFIG_CFG80211_REQUIRE_SIGNED_REGDB=y
CONFIG_CFG80211_USE_KERNEL_REGDB_KEYS=y
CONFIG_CFG80211_DEFAULT_PS=y
CONFIG_CFG80211_CRDA_SUPPORT=y
# CONFIG_CFG80211_WEXT is not set
CONFIG_MAC80211=y
CONFIG_MAC80211_HAS_RC=y
CONFIG_MAC80211_RC_MINSTREL=y
CONFIG_MAC80211_RC_DEFAULT_MINSTREL=y
CONFIG_MAC80211_RC_DEFAULT="minstrel_ht"
CONFIG_MAC80211_STA_HASH_MAX_SIZE=0
```



###### 编译驱动和涉及文件

```
external/rkwifibt/drivers/
bcmdhd
bin/arm*/		#预置厂商的bin工具，wl正基模块配置工具，rtwpriv rtk模块的测试工具
conf/*conf 		#预置的wpa_supplicant.conf/dnsmasq.conf用于sta/ap模式
S36wifibt-init.sh script/*sh 		#Buildroot系统wifibt初始化脚本，包括连接测试等等
firmware/* 		#每个厂家对应的各个型号的WiFi/BT的firmware文件

ap6275p中Wi-Fi/BT的firmware名字：
```



###### WIFI/BT启动脚本

```
#启动脚本流程：
#/etc/init.d/S36wifibt-init.sh
# -> wifibt-init.sh
# -> wifibt-util.sh
external/rkwifibt/scripts/S36wifibt-init.sh
external/rkwifibt/scripts/

#根据VID/PID自动识别Wi-Fi/BT的型号
wifibt-util.sh

#根据上面识别的型号加载WiFi的ko并初始化蓝牙
wifibt-init.sh #start_wifibt/start_bt_brcm/start_bt_rtk_usb&uart etc.
```



##### 测试

```
ifconfig wlan0 up
iw dev wlan0 scan		#确认有无BSS（AP）输出
wpa_passphrase "YNKJ-5G" "ynkj88888888" >> /etc/wpa_supplicant.conf
wpa_supplicant -B -i wlan0 -c /etc/wpa_supplicant.conf
udhcpc -i wlan0			#通过DHCP协议自动获取IP地址、网关、DNS等网络配置
ifconfig wlan0

```



