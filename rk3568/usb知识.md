usb全称为Universal Serial Bus（通用串行总线）。usb是一个标准。

​	usb标准为了满足各种设备的不同需求，使用了非常多不同种类的接口，但是这些接口都遵循机器标准

![image-20250826140009803](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20250826140009803.png)



大部分时候：

![image-20250826143534142](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20250826143534142.png)



![image-20250826145615231](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20250826145615231.png)

![image-20250826145734436](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20250826145734436.png)



###### HOST和OTG

​	USB设备分为host（主设备）和slave（从设备），host和slave连接时才能实现数据的传输

​	USB HOST：具有主机功能的设备，控制和管理usb总线，通常是计算机平板电脑等

​	USB SLAVE：具有从设备功能的设备，受usb主机控制，依赖于usb主机设备进行数据传输和通信，键盘鼠标u盘等

​	USB OTG（USB On-The-Go）:支持otg（即插即用）的设备，允许在主设备和从设备进行切换，能直接与其他usb设备进行通信，无需传统主机设备，如摄像机与电脑/打印机相连



![image-20250826160316664](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20250826160316664.png)

![image-20250826160240549](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20250826160240549.png)

![image-20250826160403072](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20250826160403072.png)



![image-20250826170739153](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20250826170739153.png)



![image-20250826173810216](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20250826173810216.png)



###### USB拓扑结构

​	主从设备之间，从设备只能通过集线器（hub）与主机连接

​	根集线器：与主机直接连接的集线器叫根集线器，普通用户外接的叫普通集线器

​	集线的作用：为了扩展更多的接口，不能无限扩展，USB2.0协议中最多扩展7层，每一层所有设备相加不能超过127个

