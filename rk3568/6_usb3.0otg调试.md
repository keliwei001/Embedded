```swift
#按键控制
gpio-keys {
        compatible = "gpio-keys";
        pinctrl-names = "default";
        pinctrl-0 = <&otg_switch_key>;

        otg-switch-key {
            label = "OTG Switch Key";
            linux,code = <KEY_F13>; // 使用一个未使用的键值
            gpios = <&gpio0 RK_PA5 GPIO_ACTIVE_LOW>; // 低电平有效
        };
    };

	gpio-keys {
        otg_switch_key: otg-switch-key {
            rockchip,pins = <0 RK_PA5 RK_FUNC_GPIO &pcfg_pull_up>;
        };
    };

```



```

vi /usr/bin/usb_otg_switch.sh
chmod +x /usr/bin/usb_otg_switch.sh

#!/bin/sh
USB_DEBUG_PATH="/sys/kernel/debug/usb/fcc00000.dwc3"
MODE_FILE="$USB_DEBUG_PATH/mode"

# 输入设备
INPUT_DEVICE="/dev/input/event5"

# 当前模式状态文件
STATE_FILE="/tmp/usb_otg_state"

# 日志文件
LOG_FILE="/var/log/usb-otg-switch.log"

# 初始化状态
init_state() {
    if [ -f "$MODE_FILE" ]; then
        current_mode=$(cat $MODE_FILE 2>/dev/null | tr -d '\n')
        echo "$current_mode" > $STATE_FILE
        echo "$(date): Initialized state to $current_mode" >> $LOG_FILE
    else
        echo "device" > $STATE_FILE
        echo "$(date): Mode file not found, defaulting to device" >> $LOG_FILE
    fi
}

# 切换USB模式的函数
switch_usb_mode() {
    current_mode=$(cat $STATE_FILE 2>/dev/null)
    
    if [ "$current_mode" = "device" ]; then
        new_mode="host"
        echo "$(date): Switching to HOST mode" >> $LOG_FILE
        echo "Switching to HOST mode"
    else
        new_mode="device"
        echo "$(date): Switching to DEVICE mode" >> $LOG_FILE
        echo "Switching to DEVICE mode"
    fi
    
    # 执行模式切换
    echo $new_mode > $MODE_FILE 2>/dev/null
    
    if [ $? -eq 0 ]; then
        echo $new_mode > $STATE_FILE
        echo "$(date): Mode switched to: $new_mode" >> $LOG_FILE
        echo "Mode switched to: $new_mode"
        
        # 添加额外的处理（可选）
        if [ "$new_mode" = "host" ]; then
            echo "$(date): Host mode activated" >> $LOG_FILE
            echo "Host mode activated"
        else
            echo "$(date): Device mode activated" >> $LOG_FILE
            echo "Device mode activated"
        fi
    else
        echo "$(date): Failed to switch mode to: $new_mode" >> $LOG_FILE
        echo "Failed to switch mode to: $new_mode"
    fi
}

# 监听按键事件
listen_for_key() {
    echo "$(date): Starting USB OTG switch daemon" >> $LOG_FILE
    echo "$(date): Listening for KEY_F13 events on $INPUT_DEVICE" >> $LOG_FILE
    echo "$(date): Current mode: $(cat $STATE_FILE 2>/dev/null)" >> $LOG_FILE
    
    echo "Starting USB OTG switch daemon..."
    echo "Listening for KEY_F13 events on $INPUT_DEVICE"
    echo "Current mode: $(cat $STATE_FILE 2>/dev/null)"
    
    # 检查输入设备是否存在
    if [ ! -e "$INPUT_DEVICE" ]; then
        echo "$(date): Error: Input device $INPUT_DEVICE not found" >> $LOG_FILE
        echo "Error: Input device $INPUT_DEVICE not found"
        exit 1
    fi
    
    # 监听按键事件
    evtest "$INPUT_DEVICE" | while read line; do
        # 检测按键按下事件 (value 1)
        if echo "$line" | grep -q "code 183 (KEY_F13), value 1"; then
            echo "$(date): OTG switch key pressed" >> $LOG_FILE
            echo "OTG switch key pressed"
            switch_usb_mode
        fi
    done
}

# 主程序
case "$1" in
    start)
        init_state
        listen_for_key
        ;;
    stop)
        echo "$(date): Stopping USB OTG switch daemon" >> $LOG_FILE
        echo "Stopping USB OTG switch daemon"
        pkill -f "usb_otg_switch.sh"
        ;;
    status)
        current_mode=$(cat $STATE_FILE 2>/dev/null || echo "unknown")
        actual_mode=$(cat $MODE_FILE 2>/dev/null || echo "unknown")
        echo "Stored mode: $current_mode"
        echo "Actual mode: $actual_mode"
        ;;
    switch)
        init_state
        switch_usb_mode
        ;;
    *)
        echo "Usage: $0 {start|stop|status|switch}"
        echo "  start   - Start listening for key events"
        echo "  stop    - Stop listening daemon"
        echo "  status  - Show current USB mode status"
        echo "  switch  - Manually switch USB mode"
        exit 1
        ;;
esac
```



```
root@rk3568-buildroot:/# cat /etc/init.d/S99usbotgswitch
#!/bin/sh
DAEMON="/usr/bin/usb_otg_switch.sh"
NAME="usb-otg-switch"
PIDFILE="/var/run/$NAME.pid"
DESC="USB OTG Mode Switch"

case "$1" in
    start)
        echo "Starting $DESC: $NAME"
        if [ -f $PIDFILE ] && kill -0 $(cat $PIDFILE) 2>/dev/null; then
            echo "$NAME is already running"
            exit 1
        fi
        start-stop-daemon -S -b -m -p $PIDFILE -x $DAEMON -- start
        ;;
    stop)
        echo "Stopping $DESC: $NAME"
        start-stop-daemon -K -p $PIDFILE 2>/dev/null
        rm -f $PIDFILE 2>/dev/null
        ;;
    restart)
        echo "Restarting $DESC: $NAME"
        $0 stop
        sleep 1
        $0 start
        ;;
    status)
        if [ -f $PIDFILE ] && kill -0 $(cat $PIDFILE) 2>/dev/null; then
            echo "$NAME is running"
            $DAEMON status
        else
            echo "$NAME is not running"
        fi
        ;;
    *)
        echo "Usage: $0 {start|stop|restart|status}"
        exit 1
        ;;
esac

exit 0

```

```
chmod +x /etc/init.d/S99usbotgswitch
```



```
BR2_ROOTFS_POST_BUILD_SCRIPT="board/rockchip/common/post-build.sh board/rockchip/rk3566_rk3568/fs-overlay/post-build.sh"
```



![image-20250901164719138](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20250901164719138.png)

```
#!/bin/bash

# 设置 USB OTG 开关脚本的执行权限
if [ -f "${TARGET_DIR}/etc/init.d/S99usbotgswitch" ]; then
    echo "Setting execute permission for S99usbotgswitch"
    chmod +x "${TARGET_DIR}/etc/init.d/S99usbotgswitch"
else
    echo "Warning: S99usbotgswitch not found in ${TARGET_DIR}/etc/init.d/"
fi

```



测试脚本

```
# 创建日志目录
mkdir -p /var/log

# 测试脚本功能
/usr/bin/usb_otg_switch.sh status
/usr/bin/usb_otg_switch.sh switch

# 测试启动脚本
/etc/init.d/S99usbotgswitch status
/etc/init.d/S99usbotgswitch start

# 查看日志
cat /var/log/usb-otg-switch.log

# 检查进程
ps aux | grep usb_otg_switch
```



测试按键

```
# 在前台测试
/usr/bin/usb_otg_switch.sh start

# 在另一个终端查看状态
/usr/bin/usb_otg_switch.sh status
```



出现问题

```
# 检查USB模式文件是否存在
ls -la /sys/kernel/debug/usb/fcc00000.dwc3/mode

# 检查输入设备
ls -la /dev/input/event5

# 手动测试模式切换
echo "device" > /sys/kernel/debug/usb/fcc00000.dwc3/mode
echo "host" > /sys/kernel/debug/usb/fcc00000.dwc3/mode

# 检查当前模式
cat /sys/kernel/debug/usb/fcc00000.dwc3/mode
```



USB OTG主切换脚本流程图

```
┌─────────────────────────────────────────────────────────────┐
│                   USB OTG模式切换主流程                      │
└─────────────────────────────────────────────────────────────┘
                               │
                               ▼
┌─────────────────────────────────────────────────────────────┐
│                       启动参数解析                           │
│  case "$1" in                                               │
│      start|stop|status|switch)                              │
└─────────────────────────────────────────────────────────────┘
                               │
                               ▼
                      ┌─────────────────┐
                      │    start        │
                      └─────────────────┘
                               │
                               ▼
┌─────────────────────────────────────────────────────────────┐
│                      初始化状态 (init_state)                 │
│ 1. 加载必要驱动 (libcomposite, usb_f_ffs等)                 │
│ 2. 强制设置为HOST模式: echo "host" > $MODE_FILE             │
│ 3. 禁用gadget: echo "" > $GADGET_PATH/UDC                   │
│ 4. 设置状态文件: echo "host" > $STATE_FILE                  │
│ 5. 记录日志并检查状态                                        │
└─────────────────────────────────────────────────────────────┘
                               │
                               ▼
┌─────────────────────────────────────────────────────────────┐
│                    监听按键事件 (listen_for_key)             │
│ 1. 检查输入设备是否存在                                      │
│ 2. 使用evtest监听/dev/input/event5                          │
│ 3. 过滤KEY_F13按键事件 (code 183, value 1)                  │
└─────────────────────────────────────────────────────────────┘
                               │
                               ▼
                      ┌─────────────────┐
                      │ 检测到KEY_F13   │
                      │   按键按下      │
                      └─────────────────┘
                               │
                               ▼
┌─────────────────────────────────────────────────────────────┐
│                    模式切换 (switch_usb_mode)                │
└─────────────────────────────────────────────────────────────┘
                               │
                      ┌─────────────────┐
                      │ 当前模式判断      │
                      └─────────────────┘
                               │
               ┌───────────────┴───────────────┐
               ▼                               ▼
┌─────────────────────────────┐    ┌─────────────────────────────┐
│  当前是DEVICE模式           │    │  当前是HOST模式             │
│  切换到HOST模式             │    │  切换到DEVICE模式           │
├─────────────────────────────┤    ├─────────────────────────────┤
│ 1. 禁用gadget               │    │ 1. 先切换到HOST模式         │
│ 2. 等待1秒                  │    │ 2. 等待1秒                  │
│ 3. 设置HOST模式             │    │ 3. 设置DEVICE模式           │
│ 4. 更新状态文件             │    │ 4. 等待2秒让控制器稳定       │
│ 5. 记录日志                 │    │ 5. 启用gadget               │
│                             │    │ 6. 更新状态文件             │
│                             │    │ 7. 记录日志                 │
└─────────────────────────────┘    └─────────────────────────────┘
                               │
                               ▼
┌─────────────────────────────────────────────────────────────┐
│                      状态检查 (check_usb_status)             │
│ 1. 记录当前时间戳                                           │
│ 2. 检查存储的模式状态                                       │
│ 3. 检查实际的模式文件                                       │
│ 4. 检查gadget绑定状态                                       │
│ 5. 写入日志文件                                             │
└─────────────────────────────────────────────────────────────┘
                               │
                               ▼
                      ┌─────────────────┐
                      │  返回监听状态   │
                      │  等待下次按键   │
                     
                     
 启动 → 初始化 → 监听按键 → 检测按键 → 切换模式 → 更新状态 → 记录日志 → 返回监听 → (循环)
```



​	USB Gadget是Linux kernel中一个框架，启用该框架时允许设备成为从设备：提供设备功能描述，管理USB端点配置，处理数据传输，与主机协商通信协议

```
┌─────────────────────────────────────────────────┐
│               USB Gadget 架构                   │
└─────────────────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────┐
│                Gadget 核心驱动                   │
│   (dwc3, udc_core, libcomposite)                │
└─────────────────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────┐
│                Function 驱动                     │
│  (f_mass_storage, f_ether, f_acm, f_ffs等)      │
└─────────────────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────┐
│                ConfigFS 配置                     │
│  (/sys/kernel/config/usb_gadget/)               │
└─────────────────────────────────────────────────┘


# Vendor和Product ID
cat /sys/kernel/config/usb_gadget/rockchip/idVendor    # 0x2207
cat /sys/kernel/config/usb_gadget/rockchip/idProduct   # 0x0006

# 设备描述信息
cat /sys/kernel/config/usb_gadget/rockchip/strings/0x409/manufacturer   # rockchip
cat /sys/kernel/config/usb_gadget/rockchip/strings/0x409/product        # rk3xxx
cat /sys/kernel/config/usb_gadget/rockchip/strings/0x409/serialnumber   # d256f05d48ad48b8

# 使用FFS（FunctionFS）功能 - 通常是ADB
/sys/kernel/config/usb_gadget/rockchip/functions/ffs.adb


# 功能绑定到配置
/sys/kernel/config/usb_gadget/rockchip/configs/b.1/f-ffs.adb

# UDC控制器绑定
cat /sys/kernel/config/usb_gadget/rockchip/UDC  # fcc00000.dwc3
```

```
Gadget工作流程
# 1. 确保模式为device
echo "device" > /sys/kernel/debug/usb/fcc00000.dwc3/mode
# 2. 绑定到UDC控制器
echo "fcc00000.dwc3" > /sys/kernel/config/usb_gadget/rockchip/UDC
# 3. 电脑识别到USB设备

禁用Gadget流程：
# 1. 从UDC控制器解绑
echo "" > /sys/kernel/config/usb_gadget/rockchip/UDC
# 2. 可以切换到host模式
echo "host" > /sys/kernel/debug/usb/fcc00000.dwc3/mode

注意：
时序：先切换到host，后切换到device，控制器稳定后绑定Gadget
检测绑定：检测是否绑定成功
状态同步：内核状态和用户状态一致
```



S99usbotgswitch

```
#!/bin/sh
DAEMON="/usr/bin/usb_otg_switch.sh"
NAME="usb-otg-switch"
PIDFILE="/var/run/$NAME.pid"
DESC="USB OTG Mode Switch"

case "$1" in
    start)
        echo "Starting $DESC: $NAME"
        
        # 确保DAEMON脚本有执行权限
        if [ ! -x "$DAEMON" ]; then
            echo "Setting execute permission for $DAEMON"
            chmod +x "$DAEMON"
        fi
        
        # 确保本脚本有执行权限
        if [ ! -x "/etc/init.d/S99usbotgswitch" ]; then
            echo "Setting execute permission for init script"
            chmod +x "/etc/init.d/S99usbotgswitch"
        fi
        
        if [ -f $PIDFILE ] && kill -0 $(cat $PIDFILE) 2>/dev/null; then
            echo "$NAME is already running"
            exit 1
        fi
        start-stop-daemon -S -b -m -p $PIDFILE -x $DAEMON -- start
        ;;
    stop)
        echo "Stopping $DESC: $NAME"
        start-stop-daemon -K -p $PIDFILE 2>/dev/null
        rm -f $PIDFILE 2>/dev/null
        ;;
    restart)
        echo "Restarting $DESC: $NAME"
        $0 stop
        sleep 1
        $0 start
        ;;
    status)
        # 确保DAEMON脚本有执行权限
        if [ ! -x "$DAEMON" ]; then
            chmod +x "$DAEMON"
        fi
        
        if [ -f $PIDFILE ] && kill -0 $(cat $PIDFILE) 2>/dev/null; then
            echo "$NAME is running"
            $DAEMON status
        else
            echo "$NAME is not running"
        fi
        ;;
    *)
        echo "Usage: $0 {start|stop|restart|status}"
        exit 1
        ;;
esac

exit 0
```

usb_otg_switch.sh

```
#!/bin/sh

# USB控制器路径
USB_DEBUG_PATH="/sys/kernel/debug/usb/fcc00000.dwc3"
MODE_FILE="$USB_DEBUG_PATH/mode"
UDC_PATH="/sys/class/udc/fcc00000.dwc3"

# 通过设备名称查找输入设备
find_input_device() {
    # 查找gpio-keys设备
    for dev in /dev/input/event*; do
        if [ -e "$dev" ]; then
            local name=$(evtest --query "$dev" name 2>/dev/null || echo "")
            if echo "$name" | grep -qi "gpio-keys"; then
                echo "$dev"
                return 0
            fi
        fi
    done
    echo ""
}

# 当前模式状态文件
STATE_FILE="/tmp/usb_otg_state"

# 日志文件
LOG_FILE="/var/log/usb-otg-switch.log"

# 现有的gadget配置
GADGET_PATH="/sys/kernel/config/usb_gadget/rockchip"

# 动态获取输入设备
INPUT_DEVICE=$(find_input_device)

# 如果没找到，使用默认的event5作为后备
if [ -z "$INPUT_DEVICE" ] || [ ! -e "$INPUT_DEVICE" ]; then
    INPUT_DEVICE="/dev/input/event5"
    echo "$(date): Warning: Using fallback device $INPUT_DEVICE" >> $LOG_FILE
fi

# USB状态检查
check_usb_status() {
    echo "=== USB Status $(date) ===" >> $LOG_FILE
    echo "Stored mode: $(cat $STATE_FILE 2>/dev/null || echo 'unknown')" >> $LOG_FILE
    if [ -f "$MODE_FILE" ]; then
        echo "Actual mode: $(cat $MODE_FILE 2>/dev/null)" >> $LOG_FILE
    else
        echo "Actual mode: Mode file not available" >> $LOG_FILE
    fi
    echo "UDC status: $(test -e $UDC_PATH && echo 'Exists' || echo 'Missing')" >> $LOG_FILE
    echo "Gadget UDC: $(cat $GADGET_PATH/UDC 2>/dev/null || echo 'Not bound')" >> $LOG_FILE
    echo "Input device: $INPUT_DEVICE" >> $LOG_FILE
    echo "=== End Status ===" >> $LOG_FILE
}

# 启用gadget功能
enable_gadget() {
    echo "$(date): Enabling USB gadget" >> $LOG_FILE
    
    # 确保gadget目录存在
    if [ ! -d "$GADGET_PATH" ]; then
        echo "$(date): Error: Gadget path $GADGET_PATH does not exist" >> $LOG_FILE
        return 1
    fi
    
    # 绑定到UDC控制器
    echo "fcc00000.dwc3" > $GADGET_PATH/UDC 2>/dev/null
    
    if [ $? -eq 0 ]; then
        echo "$(date): USB gadget enabled successfully" >> $LOG_FILE
        return 0
    else
        echo "$(date): Failed to enable USB gadget" >> $LOG_FILE
        return 1
    fi
}

# 禁用gadget功能
disable_gadget() {
    echo "$(date): Disabling USB gadget" >> $LOG_FILE
    
    # 从UDC控制器解绑
    echo "" > $GADGET_PATH/UDC 2>/dev/null
    
    echo "$(date): USB gadget disabled" >> $LOG_FILE
}

# 初始化状态
init_state() {
    echo "$(date): Initializing USB OTG state" >> $LOG_FILE
    
    # 确保必要的内核模块加载
    modprobe libcomposite 2>/dev/null
    modprobe usb_f_ffs 2>/dev/null
    
    if [ -f "$MODE_FILE" ]; then
        # 强制设置为host模式并禁用gadget
        echo "host" > $MODE_FILE 2>/dev/null
        sleep 1
        disable_gadget
        echo "host" > $STATE_FILE
        echo "$(date): Initialized to HOST mode" >> $LOG_FILE
    else
        echo "host" > $STATE_FILE
        echo "$(date): Mode file not found, defaulting to HOST" >> $LOG_FILE
    fi
    
    check_usb_status
}

# 切换USB模式
switch_usb_mode() {
    current_mode=$(cat $STATE_FILE 2>/dev/null || echo "host")
    
    if [ "$current_mode" = "device" ]; then
        # 切换到host模式
        new_mode="host"
        echo "$(date): Switching to HOST mode" >> $LOG_FILE
        
        # 先禁用gadget
        disable_gadget
        
        # 等待一下确保gadget完全禁用
        sleep 1
        
        # 切换到host模式
        echo "host" > $MODE_FILE 2>/dev/null
        
        if [ $? -eq 0 ]; then
            echo "host" > $STATE_FILE
            echo "$(date): Successfully switched to HOST mode" >> $LOG_FILE
            echo "Switched to HOST mode"
        else
            echo "$(date): Failed to switch to HOST mode" >> $LOG_FILE
            echo "Failed to switch to HOST mode"
        fi
        
    else
        # 切换到device模式
        new_mode="device"
        echo "$(date): Switching to DEVICE mode" >> $LOG_FILE
        
        # 先确保在host模式
        echo "host" > $MODE_FILE 2>/dev/null
        sleep 1
        
        # 切换到device模式
        echo "device" > $MODE_FILE 2>/dev/null
        
        if [ $? -eq 0 ]; then
            echo "device" > $STATE_FILE
            echo "$(date): USB controller set to DEVICE mode" >> $LOG_FILE
            
            # 等待控制器稳定
            sleep 2
            
            # 启用gadget
            if enable_gadget; then
                echo "$(date): Successfully switched to DEVICE mode with gadget enabled" >> $LOG_FILE
                echo "Switched to DEVICE mode"
            else
                echo "$(date): Switched to DEVICE mode but failed to enable gadget" >> $LOG_FILE
                echo "Switched to DEVICE mode (gadget activation failed)"
            fi
        else
            echo "$(date): Failed to switch to DEVICE mode" >> $LOG_FILE
            echo "Failed to switch to DEVICE mode"
        fi
    fi
    
    check_usb_status
}

# 监听按键事件
listen_for_key() {
    echo "$(date): Starting USB OTG switch daemon" >> $LOG_FILE
    echo "$(date): Using input device: $INPUT_DEVICE" >> $LOG_FILE
    
    # 重新确认设备状态
    if [ ! -e "$INPUT_DEVICE" ]; then
        echo "$(date): Error: Input device $INPUT_DEVICE not found, re-scanning..." >> $LOG_FILE
        INPUT_DEVICE=$(find_input_device)
        if [ -z "$INPUT_DEVICE" ] || [ ! -e "$INPUT_DEVICE" ]; then
            echo "$(date): Error: No suitable input device found after re-scan" >> $LOG_FILE
            echo "Available input devices:" >> $LOG_FILE
            for dev in /dev/input/event*; do
                if [ -e "$dev" ]; then
                    local name=$(evtest --query "$dev" name 2>/dev/null || echo "Unknown")
                    echo "  $dev: $name" >> $LOG_FILE
                fi
            done
            exit 1
        fi
        echo "$(date): Found new device: $INPUT_DEVICE" >> $LOG_FILE
    fi
    
    check_usb_status
    
    echo "$(date): Listening for KEY_F13 events on $INPUT_DEVICE" >> $LOG_FILE
    
    # 使用evtest监听
    evtest --grab "$INPUT_DEVICE" | while read line; do
        if echo "$line" | grep -q "code 183 (KEY_F13).*value 1"; then
            echo "$(date): OTG switch key pressed (KEY_F13)" >> $LOG_FILE
            switch_usb_mode
        fi
    done
}

# 手动测试函数
test_switch() {
    echo "$(date): Manual mode switch test" >> $LOG_FILE
    switch_usb_mode
}

# 主程序
case "$1" in
    start)
        init_state
        listen_for_key
        ;;
    stop)
        echo "$(date): Stopping USB OTG switch daemon" >> $LOG_FILE
        pkill -f "usb_otg_switch.sh"
        # 确保回到host模式
        echo "host" > $MODE_FILE 2>/dev/null
        disable_gadget
        ;;
    status)
        check_usb_status
        echo "Stored mode: $(cat $STATE_FILE 2>/dev/null || echo 'unknown')"
        if [ -f "$MODE_FILE" ]; then
            echo "Actual mode: $(cat $MODE_FILE 2>/dev/null)"
        else
            echo "Actual mode: Mode file not available"
        fi
        echo "Gadget UDC: $(cat $GADGET_PATH/UDC 2>/dev/null || echo 'Not bound')"
        echo "Input device: $INPUT_DEVICE"
        ;;
    switch)
        test_switch
        ;;
    enable-gadget)
        enable_gadget
        ;;
    disable-gadget)
        disable_gadget
        ;;
    find-device)
        # 查找输入设备
        echo "Searching for input devices..."
        for dev in /dev/input/event*; do
            if [ -e "$dev" ]; then
                name=$(evtest --query "$dev" name 2>/dev/null || echo "Unknown")
                echo "$dev: $name"
            fi
        done
        found_device=$(find_input_device)
        echo "Selected device: $found_device"
        ;;
    *)
        echo "Usage: $0 {start|stop|status|switch|enable-gadget|disable-gadget|find-device}"
        echo "  start           - Start listening for key events"
        echo "  stop            - Stop the daemon"
        echo "  status          - Show current USB mode status"
        echo "  switch          - Manually switch USB mode"
        echo "  enable-gadget   - Enable USB gadget function"
        echo "  disable-gadget  - Disable USB gadget function"
        echo "  find-device     - Find available input devices"
        exit 1
        ;;
esac

exit 0

```

