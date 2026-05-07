# tp驱动

1，驱动移植

将厂商给的驱动代码移植进kernel中。

![image-20250811145932573](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20250811145932573.png)







上电时显示触摸正常，休眠唤醒后显示正常触摸失效，lcd和触摸共用电源





```
pr_info("gtp_resume: Enter resume function\n"); 
```
