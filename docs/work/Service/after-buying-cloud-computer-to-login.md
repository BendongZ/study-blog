# 购买云主机ESC之后登录

### 1.购买云主机之后，你就可以到你的管理控制台查看你的服务器 信息啦

![service](/work/Service/Service15.png)

你可以管理设置你的服务器信息，当然你还可以登录到远程主机中，可视化操作前提是你的服务器是Windows系统的，你的公网IP：47.93.287.54

### 2.最开始你并不知道你的远程主机的登录密码，故你就自己修改了
在如上界面最右边有个更多，点击选择重置密码

![service](/work/Service/Service16.png)

### 3.然后填写你的想填的密码提交，之后重启你的云主机，即可生效

![service](/work/Service/Service17.png)

### 4.如果你想操作你自己的云主机的话，有两种方式
第一种：Windows和Mac用户一样的，在管理界面右边点击远程连接

![service](/work/Service/Service18.png)

进入如下页面

![service](/work/Service/Service19.png)

这个远程连接密码，当你是第一次进入的话，会给你显示出来的，所以你需要记住，下次进来的时候就只有框了，输入密码，即可进入

第二种：Windows用户，使用windows自带的远程连接工具，win+r之后输入mstsc

![service](/work/Service/Service20.png)

确定进入如下页面

![service](/work/Service/Service21.png)

最后连接即可

Mac用户，大致过程一样，工具的话使用 rdc for mac下载一个工具，配置也差不多，但是要连接上windows操作系统的云主机的话，需要先去配置云主机中两个东西，
解决方法：

开启组策略中远程桌面链接安全层。

- 1、开始-运行-gpedit.msc，进入组策略编辑器；
- 2、找到左侧边栏计算机配置-管理模板-Windows组件-远程桌面服务-远程桌面会话主机-安全项；
- 3、修改以下两项：
    - A,远程（RDP）连接要求使用指定的安全层，改为启用，安全层选择RDP
    - B,要求使用网络级别的身份验证对远程连接的用户进行身份验证，改为禁用；
- 4、关闭组策略编辑器，重启计算机。
完成之后即可连接了