## 简介
`wilddog-client-arduino` 是wilddog在`arduino`上的实现库，通过该库，Arduino可以轻松访问和同步云端数据。目前`wilddog-client-arduino` 仅支持Arduino Yun，用户向`wilddog`云发送的请求是通过Arduino Yun的联网模块ar9331转发到云端的，我们的库有两部分，如下：

	├── lib
	├── platform

	`lib`:Arduino IDE中的库，包含库代码、供用户调用的接口及示例.
	`platform`:不同平台下的实现，在Arduino Yun中是守护进程及依赖库的ipk安装文件.

## 使用步骤
	
###第一步 安装

####1、配置ArduinoYun

注意，如果Arduino Yun为出厂状态，需要进行配置，可参看doc目录下的`ArduinoYun-Configuration.md`，如果之前已经有过配置，可以略过。

####2、安装Wilddog库

#####在联网模块中安装守护进程及依赖库

ssh远程登陆到Arduino Yun联网模块，使用lrzsz命令搭配SercureCRT进行ftp传输，将platform/ArduinoYun/目录下的ipk下载到Arduino Yun并安装：

	root@Arduino:~# opkg update
	root@Arduino:~# opkg install lrzsz
	root@Arduino:~# rz	（在弹出的窗口中找到并选择libwilddog_xx.ipk和wilddogArduinoYun_xx.ipk，点击确定）
	root@Arduino:~# chmod +x *.ipk
	root@Arduino:~/arduinoYun# opkg install libwilddog_xx.ipk wilddogArduinoYun_xx.ipk

有以下可执行命令表明安装成功

	wilddogd      wilddog_watch
		
**注**：卸载守护进程及依赖库:

	root@Arduino:~# opkg remove --force-remove libwilddog 
	root@Arduino:~# opkg remove --force-remove wilddogArduinoYun

#####在Arduino IDE中安装wilddog库

	1、把`wilddog` 放置到Arduino IDE的`libraries`目录下.
	2、更新库，打开Arduino IDE，点击`项目-->管理库`，IDE会自动更新库，并在选择框里输入`wilddog`，出现下图说明库安装成功.
	
![](./doc/res/arduino_ide_updata.png )

###第二步 创建账号和应用

[**注册**](https://www.wilddog.com/account/signup)并登录Wilddog账号，进入控制面板。在控制面板中，添加一个新的应用。

你会获得一个独一无二的应用`URL` `https://<appId>.wilddogio.com/`，在同步和存取数据的时候，你的数据将保存在这个`URL`下。

###第三步 使用
我们在`libraries\Wilddog\examples`下提供了丰富的范例供用户学习和测试。

####API接口

new Wilddog

	定义：new Wilddog

	说明：初始化一个Wilddog节点

	返回：一个wilddog节点

getValue()

	定义：int getValue(CallBackFunc f_callback,void *arg)

	说明：设置节点的值
	
	参数：f_callback 回调函数
		 arg 用户自定义参数（可为NULL）
	
	返回：返回 0:成功 <0:失败。

setValue()

	定义：int setValue(const char *p_data,CallBackFunc f_callback,void *arg)
  
	说明：向云端设置该节点的值
	
	参数：p_data 节点的值，为json字符串
		 f_callback 回调函数
		 arg 用户自定义参数（可为NULL）

	返回：返回 0:成功 <0:失败。

push()

	定义：int push(const char *p_data,CallBackFunc f_callback,void *arg)

	说明：在当前节点下生成一个子节点，并返回子节点的引用。子节点的key利用服务端的当前时间生成。

	参数：p_data 节点的值，为json字符串
		 f_callback 回调函数
		 arg 用户自定义参数（可为NULL）

	返回：返回 0:成功 <0:失败。

removeValue()

	定义：int removeValue(CallBackFunc f_callback,void *arg)

	说明：删除当前节点

	参数：f_callback 回调函数
		 arg 用户自定义参数（可为NULL）

	返回：返回 0:成功 <0:失败。

addObserver()

	定义：int addObserver(Wilddog_EventType_T event,CallBackFunc f_callback,void *arg)

	说明：监听节点下的某个事件,注册回调函数

	参数：event 事件类型（目前只能设为1）
		 f_callback 回调函数
		 arg 用户自定义参数（可为NULL）

	返回：返回 0:成功 <0:失败。

removeObserver()

	定义：int removeObserver(Wilddog_EventType_T event)

	说明：取消监听事件。取消之前用addObserver()注册的回调函数

	参数：event 事件类型（目前只能设为1）

	返回：返回 0:成功 <0:失败。

auth()

	定义：int auth(const char *p_auth,const char *p_host,CallBackFunc onAuth,void *arg)

	说明：发送auth数据到服务器进行认证

	参数：p_auth auth数据
		 p_host 节点的host（如appid.wilddogio.com）
		 onAuth 回调函数
		 arg 用户自定义参数（可为NULL）

trySync()

	定义：void trySync()

	说明：通过调用wilddog_trySync来向Wilddog云端同步数据。每次调用都会处理来自云端的推送和请求超时的重发、长连接的维持 ，以及触发用户注册的回调函数。
