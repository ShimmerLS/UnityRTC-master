# 服务端程序免费私有部署

以下服务端均完全免费(采用C语言开发)，无鉴权，可用于腾讯云，阿里云或局域网内部署，现已开放:


| 服务端        | 功能           | 备注  |
| ------------- |:-------------  |:-----|
| voipServer    | 一对一视频通话 					  | 需要搭配msgServer使用 |
| msgServer     | 单聊（如文字聊天），私信，信令      |    |
| chatDBServer  | 离线消息存储      				  |     |
| groupServer   | 群聊      					      | 如果只需要单聊，不需要群聊的话，不用启动    |
| chatRoomServer| 多人聊天室      					  |     |
| liveSrcServer | 多人视频会议  RTMP推流      		  |     |
| liveVdnServer | 互动连麦直播，vdn分发网络		      |     |
| liveProxyServer | RTSP 拉流服务端     				      |     |
| videoRecServer | 录制录像功能     				      |     |
| groupPushHttpProxy | 系统消息及群操作功能     				      |     |


 web-supported目录里面是支持web端的服务端程序与自签名证书。do-not-support-web目录里面的服务端程序不支持web端。

**支持CentOS 64bit，Ubuntu 64bit**。Windows上请自行安装虚拟机(请使用桥接)或docker测试。

#### 支持功能：

* 多人同时在线聊天：由于基于webrtc的p2p连接，每两个人之间都有一个peerconnection，只能少数几人互通，否则性能会有问题
* 支持android和ios
* 语音扬声器模式与游戏背景音共存
* 语音断线重连

### MutiRTC_Unity

  unity工程，基于版本5.3.3f1。包含一个简单的多人实时语音聊天室场景。语音模块以平台sdk形式集成进unity，包括安卓和ios的语音sdk，详见plugins目录。<br>
  可支持多人视频(videotrack)和文字聊天(datachannel)，暂时屏蔽了

### ICE Server
  ICE Server 用来实现p2p穿墙或可靠传输。
  基于 https://github.com/coturn/coturn ，具体环境的搭建参考网上资料。如果只是测试，可以用第三方的服务器，比如：http://numb.viagenie.ca/ 可以申请账号，demo中有提供申请好的账号，如果是局域网同一wifi，不需要ice server
## 使用
### MutiRTC_Unity
* Main Camera 上的Main脚本中修改Host/Port为自己的信令服务器ip地址和端口，IceServers修改为自己搭建或申请的stun/turn地址
* 打包app，先init，再输入房间号join

 ### 部署步骤（请切换为root用户或者用sudo执行）：

第1步：下载服务端程序： git clone https://github.com/starrtc/starrtc-server.git
		
		然后进入相应目录，直接执行chmod +x *.sh && ./start.sh 即部署成功！如果想单独运行，请继续下面的步骤。

第2步：进入相应目录，给所有服务端程序加可执行权限: chmod +x *Server  

第3步：部署各服务端程序，具体如下：

其中.log后缀文件为日志文件，可通过命令tail -f xxx.log查看相关日志。

voip服务端部署
==
```java
后台启动：
nohup ./voipServer > voipServer.log 2>&1 &

刚开始为了验证是否启动成功，可以不后台启动，而是通过运行 ./voipServer 直接看输出日志是否成功，成功了以后就可以后台启动。
```
注：也需要部署msgServer,用于传输呼叫，接听等消息。

IM服务端部署
==
IM全套服务，分为3个服务端程序，分别是:

消息服务端msgServer、离线消息数据服务端chatDBServer，群管理服务端groupServer，分别启动即可。

只需要单聊的，不需要启动groupServer。

可以保持自己原有的im系统不变，用我们的im系统作为voip等服务的信令服务。
```java
后台启动：
nohup ./msgServer     > msgServer.log 2>&1 &
nohup ./chatDBServer  > chatDBServer.log 2>&1 &
nohup ./groupServer   > groupServer.log 2>&1 &
```

chatRoom服务端部署
==
```java
后台启动：
nohup ./chatRoomServer > chatRoomServer.log 2>&1 &
```

liveSrc服务端部署
==
```java  
后台启动：
nohup ./liveSrcServer > liveSrcServer.log 2>&1 &
```

RTMP推流测试:可打开安卓客户端，新建一个会议室，点击RTMP推流，填上RTMP URL后，点击推流即可。然后用其它第3方播放器如VLC就可以打开该RTMP URL观看会议画面了。

同理，可以在直播间推流，用vlc打开就可以观看直播了。

liveVdn服务端部署
==
互动直播，观众不限人数
```java  
后台启动：
nohup ./liveVdnServer > liveVdnServer.log 2>&1 &
```

录制服务端(videoRecServer)部署
==
目前用于liveSrcServer和voipServer的视频录像功能，目前为测试版，输出为ts文件，支持自定义切片或不切片，音频只支持AAC格式。

videoRecServer默认是切片模式，30s一片，若不切片，请在程序同级目录中新建starrtc.conf文本文件，写入recSegMode=off，即关闭切片模式，不切片的时候切片序号一直为0。

文件目录格式为：

在线会议或互动直播:

./RECFOLDER/liveChannels/用户名/resSessionId_用户名_切片序号.ts，如./RECFOLDER/liveChannels/tom/1573119917990_tom_0.ts

一对一视频通话(VOIP):

./RECFOLDER/voips/用户名/resSessionId_用户名_切片序号.ts，如./RECFOLDER/voips/tom/1573119917990_tom_0.ts

其中，sessionId在移动端SDK中获取得到，详见android文档。



```java  
后台启动：
nohup ./videoRecServer > videoRecServer.log 2>&1 &
```

系统消息及群操作功能服务
==
用户使用AEC高级模式的情况下使用，比如给某用户发送系统消息(例如购买消费成功通知)，或给某个群的全部用户发送群系统消息(例如某人进群、退群)。

请注意该服务仅供内网其他服务使用，不要将19922端口暴露到外网！

```java 
push系统消息:
toUsers：需要发送消息的所有用户，用逗号隔开
msg： 需要发送的文本内容
digest： 需要发送的文本内容的摘要，用于用户不在线时的push推送使用
http://www.xxx.com:19922/pushSystemMsgToUsers?toUsers=userId1,userId2,userId3,...&msg=xxxx&digest=xxxx

push群消息(全员):   
http://www.xxx.com:19922/pushGroupMsg?groupId=xxx&msg=xxxx
```

下面五个和群有关的接口，在客户端sdk同样有实现，但通过这些接口，服务端可以主动给群服务器同步群成员，或对群成员进行其他操作，请您根据实际需求来选取合适的群成员同步策略。
```java 
同步群成员:	
groupId: 群id
groupList:   所有群成员，用逗号隔开，不传groupList表示清空这个群的成员
ignoreList： 对该群设置了消息免打扰的群成员id，用逗号隔开
http://www.xxx.com:19922/syncGroupList?groupId=xxx&groupList=userId1,userId2,userId3,...&ignoreList=userId1,userIdx,...

添加群成员:   
addedUsers: 要添加进的群的所有用户id，用逗号隔开
http://www.xxx.com:19922/addUsersToGroup?groupId=xxx&addedUsers=userId1,userId2,userId3,...

删除群成员:   
deledUsers: 需要从群内删除的所有用户id，用逗号隔开
http://www.xxx.com:19922/delUsersFromGroup?groupId=xxx&deledUsers=userId1,userId2,userId3,...

设置免打扰:	
ignoreList: 对该群设置消息免打扰(不接收群消息)的所有用户id，用逗号隔开
http://www.xxx.com:19922/setPushIgnore?groupId=xxx&ignoreList=userId1,userIdx,...

取消免打扰:	
ignoreList: 对该群取消免打扰(接收群消息)的所有用户id，用逗号隔开
http://www.xxx.com:19922/unsetPushIgnore?groupId=xxx&ignoreList=userId1,userIdx,...

```





拉流服务端部署
==
用于拉取第三方rtsp流(RTMP流暂未开放)，转换为starRTC协议后转发到liveSrcServer，
然后就可以在各终端(Android,iOS,PC和web)的在线会议或互动直播中播放这个流了。

```java  
后台启动：
nohup ./liveProxyServer > liveProxyServer.log 2>&1 &
```

测试方法：首先找到一个可以正常播放的rtsp流（也可以使用示例程序里面的默认测试流），
然后可以打开安卓示例程序，打开设置-》第3方流测试-》新建一个流，填一下名字，和流的rstp地址（也可以不填直接使用默认的测试流），
同时选择该流是在直播中播放，还是在会议中播放。 然后去直播间或会议室就可以看到拉的视频流画面了。


也可以自己使用HTTP方式调用：

- 1 创建channelId并拉流（streamType暂时只支持rtsp），接口返回channelId：

http://www.xxx.com:19932/push?streamType=rtsp&streamUrl=rtsp://184.72.239.149/vod/mp4://BigBuckBunny_175k.mov&roomLiveType=0&roomId=xxxx&extra=xxxxx 

其中roomId和extra为可选参数

- 2 拉流到指定的channelId：

http://www.xxx.com:19932/push?streamType=rtsp&streamUrl=rtsp://184.72.239.149/vod/mp4://BigBuckBunny_175k.mov&channelId=xxxx

- 3 停止拉流（不删除channelId，仍在列表中存在）：

http://www.xxx.com:19932/close?channelId=xxxx

- 4 停止拉流，同时删除channelId：

http://www.xxx.com:19932/delete?channelId=xxxx

需要开放端口
====
| 服务端        | 端口           | web端需开放端口            | 
| ------------- |:-------------  |:-------------  |
| msgServer      | 19903(tcp)     | 29991(tcp):https信任测试   | 
| voipServer     | 10086(udp)  44446(udp):P2P通讯     | 10087(tcp):websocket 10088(udp):webrtc   29992(tcp):https信任测试| 
| chatRoomServer | 19906(tcp)      | 29993(tcp):https信任测试  | 
| liveSrcServer  | 19931(udp)       | 19934(tcp):websocket 19935(udp):webrtc 29994(tcp):https信任测试 |
| liveVdnServer  | 19928(udp)     	 | 19940(tcp):websocket 19941(udp):webrtc 29995(tcp):https信任测试	|
| liveProxyServer |19932(tcp)   	 |   |


测试方法
=====
下载[客户端示例程序](https://docs.starrtc.com/en/download/)，

打开"设置->服务器配置"，然后填写你自己的服务器ip即可（注意不要修改端口号，如果是域名不需要添加“http://”前缀）。


客户端开发
=====
基于私有部署服务端开发自己的客户端，参见[开发文档](https://docs.starrtc.com/zh-cn/docs/android-3b.html)，

示例代码参见：https://docs.starrtc.com/en/download/

服务端开发
=====
打开配置文件starrtc.conf，修改里面的aecurl的值（目前不支持https地址），开发请参考server-api目录里面的示例代码。

更新记录
=====
https://github.com/starrtc/starrtc-server/wiki/Changelog

参考
==
[端口连接性测试](https://github.com/starrtc/starrtc-server/wiki/TCP%E4%B8%8EUDP%E7%AB%AF%E5%8F%A3%E8%BF%9E%E6%8E%A5%E6%80%A7%E6%B5%8B%E8%AF%95)

[阿里云修改安全组规则](https://help.aliyun.com/document_detail/101471.html)

[腾讯云安全组操作指南](https://cloud.tencent.com/document/product/213/18197)

