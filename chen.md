rm -rf gt9xx,将厂商gt9xx放到yxw@yxw-virtual-machine:~/myproject/rk356x_linux5.1/kernel/drivers/input/touchscreen/gt9xx



![image-20250811152411364](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20250811152411364.png)



修改参数



在touchscreen的Kconfig中添加source "drivers/input/touchscreen/gt9xx/Kconfig" 



