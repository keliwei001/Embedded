Ubuntu中：

```
sudo ps -e |grep ssh
#如果没有ssh显示，输入
sudo service ssh start
sudo service ssh status
```

![image-20251022093821820](C:\Users\Administrator\Desktop\md\vscode通过ssh连接Ubuntu.assets\image-20251022093821820.png)

然后：

```
ifconfig
```

![image-20251022094011483](C:\Users\Administrator\Desktop\md\vscode通过ssh连接Ubuntu.assets\image-20251022094011483.png)



whidows中：window+r打开

![image-20251022094411402](C:\Users\Administrator\Desktop\md\vscode通过ssh连接Ubuntu.assets\image-20251022094411402.png)

![image-20251022094443941](C:\Users\Administrator\Desktop\md\vscode通过ssh连接Ubuntu.assets\image-20251022094443941.png)

打开vscode，打开扩展，搜索ssh，安装ssh

![image-20251022094535641](C:\Users\Administrator\Desktop\md\vscode通过ssh连接Ubuntu.assets\image-20251022094535641.png)

 安装完ssh

 打开管理，找到设置

![image-20251022094836608](C:\Users\Administrator\Desktop\md\vscode通过ssh连接Ubuntu.assets\image-20251022094836608.png)

![image-20251022094854180](C:\Users\Administrator\Desktop\md\vscode通过ssh连接Ubuntu.assets\image-20251022094854180.png)

![image-20251022094912353](C:\Users\Administrator\Desktop\md\vscode通过ssh连接Ubuntu.assets\image-20251022094912353.png)

![image-20251022094924850](C:\Users\Administrator\Desktop\md\vscode通过ssh连接Ubuntu.assets\image-20251022094924850.png)



###### 使用

打开远程资源管理器

点击加号（+）

![image-20251022095126539](C:\Users\Administrator\Desktop\md\vscode通过ssh连接Ubuntu.assets\image-20251022095126539.png)

打开扩展

输入ssh xxx(ubuntu名称)@xxxxxx(ip地址）

```
ssh yxw@192.168.1.18
```

![image-20251022095357419](C:\Users\Administrator\Desktop\md\vscode通过ssh连接Ubuntu.assets\image-20251022095357419.png)

选第一个，选择connect，输入密码

![image-20251022095508669](C:\Users\Administrator\Desktop\md\vscode通过ssh连接Ubuntu.assets\image-20251022095508669.png)