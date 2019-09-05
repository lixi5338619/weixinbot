# weixinbot

网页版微信公众号监听机器人   v-1.0.0
---

- 运行 run_wxbot.py 文件，扫码登陆后，即可监听公众号文章并采集提交;    
- wxpy.pkl 文件记录登陆信息，短时间不需要进行重复扫码登陆;
- bot.log 记录日志信息，登陆时间 采集信息 提交信息等;

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
  
- - - 

  
  
- - -
#### 监控设置：    
开启 smtp 服务，登出时邮件通知。   
开启 @回复，可自行查询监听状况。

- - -
#### 代理ip：    
提取类型： 每次请求提取一个https隧道IP。     
使用时间： 每天早9点晚12点,自动开启代理ip。   
代理规则： 代理ip存活时间15分钟。       
消耗预算： 每日大约需要消耗60个IP。

- - -
#### 注意事项：
```log         

linux下参数设置：
bot = Bot(console_qr=True  # 终端显示二维码 )  
# console_qr 在大部分Linux 系统中可设为`True`或 2，
# MacOS Terminal 的默认白底配色中，应设为 -2。

Send_email = True   # 开启 登出邮件通知
config_smtp中设置 收发邮件对象


linux下itchat 源码修改事项：
路径:/usr/local/lib/python3.5/site-packages/
注释了 group.py 106行 用户初始化 log 信息
创建了 itchat/components/config 文件 config_smtp.py
增加了 itchat/components/login.py 中 SMTP 服务


linux启动事项：  
启动报错:lOG OUT。 解决方法：删除 wx.pkl 关闭缓存

使用 screen命令 实现当前窗口与任务分离：
新建窗口        screen -S wxbot
会话分离        Ctrl+A D(即按住Ctrl，依次再按A,D)
恢复会话窗口     screen -r wxbot
杀死会话窗口     screen -X -S 10289 quit 
```
- - -
- - -
- - -
09/05 更新：开启 smtp 服务，登出邮件通知。
  ![在这里插入图片描述](https://img-blog.csdnimg.cn/20190905131229885.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzU4MjEwMQ==,size_16,color_FFFFFF,t_70)
