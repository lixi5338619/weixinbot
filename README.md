# weixinbot

网页版微信公众号监听机器人   v-1.0.0
---
 操作指南:   
微信群内 @bot, 并       
回复'查看' -> 可获取当前抓取数量   
回复'来源' -> 可查看已监听到的公众号      
回复'login' -> 可查看本次bot登陆时间     
回复'help' -> 可获取操作指南


- - -
### 登陆逻辑图：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190902171231751.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzU4MjEwMQ==,size_16,color_FFFFFF,t_70)
- - -
### 登陆的过程按顺序为:

- 获取二维码uuid
- 获取二维码
- 判断是否已经登陆成功
- 获取初始化数据
- 更新微信相关信息（通讯录、手机登陆状态）
- 循环扫描新信息（开启心跳）

- - -

### 登陆方法简介: 
|方法名|    |  简介 文件地址: itchat/core.py|
|:---: |:---:|:---|
|get_QRuuid| |获取生成二维码所需的uuid，并返回。|
|check_login| |判断是否已经登陆成功，返回扫描的状态码。|
|show_mobile_login| |在手机上显示登录状态。|
|web_init| |获取微信用户信息以及心跳所需要的数据。|
|start_receiving| |循环扫描是否有新的消息，开启心跳包。|

core.py文件中的Core类定义了itchat的所有接口。
但是只定义了接口，方法全部在component包中实现重构。

文件地址:  itchat/components/login.py
- - -

### 登陆接口介绍：

- 通过抓包，发现携带appid和fun通过get请求
对https://login.weixin.qq.com/进行访问，可获得uuid

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190902154143385.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzU4MjEwMQ==,size_16,color_FFFFFF,t_70)

uuid如下图所示：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190902154213220.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzU4MjEwMQ==,size_16,color_FFFFFF,t_70)
- - -


- 2.**通过uuid- 获取二维码图片。**(并保存，此处省略)
请求接口：
 https://login.weixin.qq.com/1/ + uuid


- 3.**判断是否已经登陆成功**

   ```Request URL: https://login.wx.qq.com/cgi-bin/mmwebwx-bin/login?loginicon=true&uuid=oYdNqgD3wQ==&tip=0&r=251883138&_=1567410890538```

	接口参数：uuid = uuid 	。r = int(时间戳) / 1579 。  	_ = int(时间戳)
	
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190902155941156.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzU4MjEwMQ==,size_16,color_FFFFFF,t_70)


4.**获取登陆url信息**

   完成登录时(扫描qrcode)，获取 loginInfo_url
   
   syncUrl和fileUploadingUrl将被获取	  
   将生成deviceid和msgid	 	
   获取skey, wxsid, wxuin, pass_ticket
   
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190902161519154.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzU4MjEwMQ==,size_16,color_FFFFFF,t_70)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190902161624539.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzU4MjEwMQ==,size_16,color_FFFFFF,t_70)


- 5.**初始化信息**
接口：    url = '%s/webwxinit?r=%s' % （loginInfo['url'], int(时间戳))
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190902162306630.png)


- 6.**更新微信相关信息（登陆状态）**
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190902162700692.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzU4MjEwMQ==,size_16,color_FFFFFF,t_70)


- 7.**循环扫描新信息（开启心跳监听）**

    **多线程的事件监听器模式。**

   1, 创建线程并启动,在创建线程的位置设置一个标记
 
   2,创建队列保存线程
   
   3,遍历队列中的线程 ,并得到标记
   ![在这里插入图片描述](https://img-blog.csdnimg.cn/20190902164902851.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzU4MjEwMQ==,size_16,color_FFFFFF,t_70)
  
  
  - - -
  **csdn**: https://blog.csdn.net/weixin_43582101/article/details/100306681
