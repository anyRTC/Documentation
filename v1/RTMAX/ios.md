## 一、快速开始

### 集成指南

#### 适用范围

本集成文档适用于iOS RTMaxEngine SDK 3.0.0 ~ 3.0.3版本。

#### 准备环境

- Xcode 9.0+。
- iOS 8.0+ 真机（iPhone 或 iPad）。
- 请确保你的项目已设置有效的开发者签名。

#### 导入SDK

**CocoaPods导入**

* 通过 Cocoapods 下载地址：

```
pod 'RTMaxEngine'
```
* 如果需要安装指定版本则使用以下方式（以 3.0.1.8 版本为例）：

```
pod 'RTMaxEngine', '~> 3.0.1.8'
```

**手动导入**

* 前往GitHub[下载Demo](https://github.com/anyRTC/AR-Talk-iOS)，找到RTMaxEngine.framework；

* 在Xcode中选择“Add files to 'Your project name'...”，将RTMaxEngine.framework添加到你的工程目录中

![ios_max_01](/assets/images/ios/ios_max_01.png)
* 打开General->Embedded Binaries中添加RTMaxEngine.framework

![ios_max_02](/assets/images/ios/ios_max_02.png)

#### 权限说明

使用RTMaxEngine SDK 前，需要对设备进行授权。打开 info.plist ，点击 + 图标开始添加：

* 添加设备使用「网络」的权限
```
<key>NSAppTransportSecurity</key>
<dict>
<key>NSAllowsArbitraryLoads</key>
<true/>
</dict>
```

* 添加设备使用「相机」的权限
```
<key>NSCameraUsageDescription</key>
<string>XXX请求访问相机用于...</string>
```

* 添加设备使用「麦克风」的权限

```
<key>NSMicrophoneUsageDescription</key>
<string>XXX请求访问麦克风用于...</string>
```

#### 后台模式(Background Modes)

勾选Audio, AirPlay and Picture in Picture

![ios_max_03](/assets/images/ios/ios_max_03.png)

### 开发指南

#### 1. 初始化SDK

集成SDK后，还需对SDK进行初始化操作，建议在AppDelegate中完成。

##### 1.1 导入头文件

```
#import <RTMaxEngine/RTMaxEngineSDK.h>
```

##### 1.2 配置开发者信息

调用initEngine:token:方法配置开发者信息，开发者信息可在anyRTC管理后台中获得，详见[创建anyRTC账号](https://docs.anyrtc.io)


**示例代码：**

```
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {
// Override point for customization after application launch.
//配置开发者信息
[ARMaxEngine initEngine:appID token:token];
return YES;
}
```

#### 2. 加入房间

##### 2.1 实例化对讲调度对象

调用initWithDelegate:方法实例化对讲调度对象，需实现ARMaxKitDelegate中的回调方法。

**示例代码：**

```
self.maxKit = [[ARMaxKit alloc] initWithDelegate:self];
```

##### 2.2 加入对讲组

调用joinTalkGroupByToken:groupId:userId:userData:方法加入对讲组，第一个参数token为令牌，可为空，具体用法可参考[安全指南](https://docs.anyrtc.io/v1/security/服务级安全设置指南.html)。

**示例代码：**

```
NSDictionary *dict  = [[NSDictionary alloc] initWithObjectsAndKeys:self.userId,@"userId",nil];
[self.maxKit joinTalkGroupByToken:nil groupId:@"123456789" userId:self.userId userData:[RMCommons fromDicToJSONStr:dict]];
```

##### 2.3 抢麦

调用applyTalk:方法抢麦，参数priority为抢麦用户级别，数值越大，级别越小。当返回值为0时，代表抢麦成功。

**示例代码：**

```
[self.maxKit applyTalk:1];
```

##### 2.4 设置本地视频采集窗口

调用setLocalVideoCapturer:option:方法设置本地视频采集窗口，第二个参数option为配置信息，包括视频帧率、码率、方向等信息。

**示例代码：**

```
//配置信息
ARVideoConfig *config = [[ARVideoConfig alloc] init];
//视频质量
config.videoProfile = ARVideoProfile480x640;
ARMaxOption *option = ARMaxOption.defaultOption;
option.videoConfig = config;
//设置本地视频采集窗口
[self.maxKit setLocalVideoCapturer:videoView option:option];
```

##### 2.5 呼叫相关

* 发起呼叫（makeCall:type:userData:）

* 视频监看（monitorVideo:userData:）

* 视频上报（reportVideo:）

##### 2.6 离开对讲组

调用leaveTalkGroup方法释放对讲调度资源。

**示例代码：**

```
[self.maxKit leaveTalkGroup];
```

## 二、API接口文档

### ARMaxEngine 接口类

#### 1. 配置开发者信息

**定义**

```
+ (void)initEngine:(NSString *)appId token:(NSString *)token;
```
**参数**

参数名 | 类型 | 描述
---|:---:|---
appId | NSString | appId
token | NSString | token

**说明**

该方法为配置开发者信息，上述参数均可在[https://www.anyrtc.io/ ](https://www.anyrtc.io/)应用管理中获得，建议在AppDelegate.m调用。

#### 2. 配置私有云

**定义**

```
+ (void)configServerForPriCloud:(NSString *)address port:(int)port;
```
**参数**

参数名 | 类型 | 描述
---|:---:|---
address | NSString | 私有云服务地址
port | int | 私有云服务端口

**说明**

配置私有云信息，当使用私有云时才需要进行配置，默认无需配置。

#### 3. 获取SDK版本号

**定义**

```
+ (NSString *)getSDKVersion;
```
**返回值**

RTMaxEngine SDK版本号

#### 4. 设置打印日志级别

**定义**

```
+ (void)setLogLevel:(ARLogModel)levelModel;
```
**参数**

参数名 | 类型 | 描述
---|:---:|---
levelModel | ARLogModel | 日志级别

### ARMaxKit 接口类

#### 1. 实例化ARMaxKit对象

**定义**

```
- (instancetype)initWithDelegate:(id<ARMaxKitDelegate>)delegate;
```

**参数**

参数名 | 类型 | 描述
---|:---:|---
delegate | id<ARMaxKitDelegate> | 对讲调度相关回调代理

**返回**

对讲调度对象。

#### 2. 加入对讲组

**定义**

```
- (int)joinTalkGroupByToken:(NSString * _Nullable)token groupId:(NSString *)groupId userId:(NSString *)userId userData:(NSString *)userData;
```

**参数**

参数名 | 类型 | 描述
---|:---:|---
token | NSString | 客户端向自己服务申请获得，参考企业级安全指南
groupId | NSString | 对讲组id（同一个anyrtc平台的appid内保持唯一性）
userId | NSString | 用户的第三方平台的用户Id
userData | NSString | 用户的自定义数据

**返回值**

-1为groupId为空，0为成功，2为userData 大于512字节。


#### 3. 切换对讲组

**定义**

```
- (int)switchTalkGroup:(NSString *)groupId userData:(NSString *)userData;
```

**参数**

参数名 | 类型 | 描述
---|:---:|---
groupId | NSString | 对讲组id（同一个anyrtc平台的appid内保持唯一性）
userData | NSString | 用户的自定义数据

**返回值**

-2为切换对讲组失败，-1为groupId为空，0为成功，2为userData 大于512字节。


#### 4. 离开对讲组

**定义**

```
- (void)leaveTalkGroup;
```

**说明**

该方法为释放对讲调度资源

#### 5. 切换对讲组

**定义**

```
- (int)applyTalk:(int)priority;
```

**参数**

参数名 | 类型 | 描述
---|:---:|---
priority | int | 申请抢麦用户的级别（0权限最大（数值越大，权限越小）；除0意外，可以后台设置0-10之间的抢麦权限大小）

**返回值**

0为成功，-1为未登录，-2为正在对讲中，-3为资源还在释放中，-4为 操作太过频繁。

#### 6. 取消抢麦/下麦

**定义**

```
- (void)cancleTalk;
```

#### 7. 设置只有对讲模式

**定义**

```
- (void)setOnlyTalkMode:(BOOL)enable;
```

**参数**

参数名 | 类型 | 描述
---|:---:|---
enable | BOOL | YES:只有对讲；NO:包含所有功能

#### 8. 设置本地视频采集窗口

**定义**

```
- (void)setLocalVideoCapturer:(UIView *)render option:(ARMaxOption *)option;
```

**参数**

参数名 | 类型 | 描述
---|:---:|---
render | UIView | 视频显示对象
option | ARMaxOption | 视频配置

#### 9. 切换前后摄像头

**定义**

```
- (void)switchCamera;
```

#### 10. 设置音量检测

**定义**

```
- (void)setAudioActiveCheck:(BOOL)on;
```

**参数**

参数名 | 类型 | 描述
---|:---:|---
on | BOOL | YES为打开音量检测；NO为关闭音量检测

**说明**

默认打开

#### 11. 获取音频检测是否打开

**定义**

```
- (BOOL)audioActiveCheck;
```

#### 12. 设置本地音频是否传输

**定义**

```
- (void)setLocalAudioEnable:(BOOL)enable;
```

**参数**

参数名 | 类型 | 描述
---|:---:|---
enable | BOOL | YES为传输音频,NO为不传输音频

**说明**

默认传输

#### 13. 设置本地视频是否传输

**定义**

```
- (void)setLocalVideoEnable:(BOOL)enable;
```

**参数**

参数名 | 类型 | 描述
---|:---:|---
enable | BOOL | YES为传输视频，NO为不传输视频，默认传输

#### 14. 设置录像

**定义**

```
- (BOOL)setRecordPath:(NSString *)filePath talkPath:(NSString *)talkPath talkP2PPath:(NSString *)talkP2PPath;
```

**参数**

参数名 | 类型 | 描述
---|:---:|---
filePath | NSString | 呼叫通话录音的保存路径（文件夹路径）
talkPath | NSString | 对讲录音的保存路径（文件夹路径）
talkP2PPath | NSString | 强插P2P录像的保存路径（文件夹路径）

**返回值**

0为设置失败，1为设置成功。

#### 15. 设置远端用户是否可以接收自己的音视频

**定义**

```
- (void)setRemoteAVEnable:(NSString *)userId audio:(BOOL)enable video:(BOOL)enable;
```

**参数**

参数名 | 类型 | 描述
---|:---:|---
userId | NSString | 远端用户userId
enable | BOOL | YES：接收音频，NO：不接收音频
enable | BOOL | YES：接收视频，NO：不接收视频

#### 16. 设置不接收指定通道的音视频

**定义**

```
- (void)setRemotePeerAVEnable:(NSString *)peerId audio:(BOOL)enable video:(BOOL)enable;
```

**参数**

参数名 | 类型 | 描述
---|:---:|---
peerId | NSString | 视频通道的peerId
enable | BOOL | YES：接收音频，NO：不接收音频
enable | BOOL | YES：接收视频，NO：不接收视频

#### 17. 启用/关闭扬声器播放

**定义**

```
- (BOOL)setEnableSpeakerphone:(BOOL)enableSpeaker;
```

**参数**

参数名 | 类型 | 描述
---|:---:|---
enableSpeaker | BOOL | YES切换到外放，NO切换到听筒。如果设备连接了耳机，则语音路由走耳机

#### 18. 查询扬声器启用状态

**定义**

```
- (BOOL)isSpeakerphoneEnabled;
```

#### 19. 重置音频录音和播放

**定义**

```
- (void)doRestartAudioRecord;
```

**说明**

使用AVplayer播放后调用该方法。


#### 20. 打开或者关闭网络状态回调

**定义**

```
- (void)setNetworkStatus:(BOOL)enable;
```

**参数**

参数名 | 类型 | 描述
---|:---:|---
enable | BOOL | YES打开网络状态回调，NO关闭网络状态回调，默认关闭

#### 21. 获取网络状态是否打开

**定义**

```
- (BOOL)networkStatusEnabled;
```

**返回值**

YES网络状态打开，NO网络状态关闭。

#### 22. 关闭P2P通话

**定义**

```
- (void)closeP2PTalk;
```

**说明**

在控制台强插对讲后，关闭和控制台之间的P2P通话。

#### 23. 发起呼叫

**定义**

```
- (int)makeCall:(NSString *)userId type:(int)type userData:(NSString *)userData;
```

**参数**

参数名 | 类型 | 描述
---|:---:|---
userId | NSString | 被呼叫用户Id
type | int | 呼叫类型：0视频，1音频
userData | NSString | 用户的自定义数据

**返回值**

0为调用OK，-1未登录，-2没有通话，-3视频资源占用中，-5:本操作不支持自己对自己，-6:会话未创建（没有被呼叫用户）。

#### 24. 邀请用户

**定义**

```
- (int)inviteCall:(NSString *)userId userData:(NSString *)userData;
```

**参数**

参数名 | 类型 | 描述
---|:---:|---
userId | NSString | 被邀请用户的Id
userData | NSString | 用户的自定义数据

**返回值**

0为调用OK，-1未登录，-2没有通话，-3视频资源占用中，-5:本操作不支持自己对自己，-6:会话未创建（没有被呼叫用户）。

#### 25. 主叫端结束某一路正在进行的通话

**定义**

```
- (void)endCall:(NSString *)userId;
```

**参数**

参数名 | 类型 | 描述
---|:---:|---
userId | NSString | 结束的这路视频的用户ID

#### 26. 同意呼叫

**定义**

```
- (int)acceptCall:(NSString *)callId;
```

**参数**

参数名 | 类型 | 描述
---|:---:|---
callId | NSString | 呼叫请求时收到的Id

**说明**

0调用OK，-1未登录，-2会话不存在，-3视频资源占用中。


#### 27. 拒绝通话

**定义**

```
- (void)rejectCall:(NSString *)callId;
```

**参数**

参数名 | 类型 | 描述
---|:---:|---
callId | NSString | 呼叫的Id

#### 28. 被叫端离开通话

**定义**

```
- (void)leaveCall;
```

**说明**

调用该方法后，把本地视频窗口从父视图上删除。


#### 29. 设置显示其他端视频窗口

**定义**

```
- (void)setRemoteVideoRender:(UIView *)render pubId:(NSString *)pubId;
```

**参数**

参数名 | 类型 | 描述
---|:---:|---
render | UIView |  显示对方视频的窗口
pubId | NSString |  RTC服务生成流的ID (用于标识与会者发布的流)

**说明**

该方法用于与会者接通后，与会者视频接通回调中（onRTCRemoteOpenVideoRender）使用。


#### 30. 发起视频监看（或者收到视频上报请求时查看视频）

**定义**

```
- (int)monitorVideo:(NSString *)userId userData:(NSString *)userData;
```

**参数**

参数名 | 类型 | 描述
---|:---:|---
userId | NSString |  被监看用户Id
userData | NSString | 用户的自定义数据

**返回值**

0调用OK，-1未登录，-5本操作不支持自己对自己。

#### 31. 同意别人视频监看

**定义**

```
- (int)acceptVideoMonitor:(NSString *)userId;
```

**参数**

参数名 | 类型 | 描述
---|:---:|---
userId | NSString |  被监看用户Id

**说明**

该方法为被监看方调用，在接收到回掉（onRTCVideoMonitorRequest）后调用

**返回值**

0调用OK，-1未登录，-3:视频资源占用中，-5本操作不支持自己对自己。

#### 32. 拒绝监看

**定义**

```
-(void)rejectVideoMonitor:(NSString *)userId;
```

**参数**

参数名 | 类型 | 描述
---|:---:|---
userId | NSString |  发起人的用户Id

#### 33. 监看发起者关闭视频监看（谁监看谁关闭）

**定义**

```
- (void)closeVideoMonitor:(NSString *)userId;
```

**参数**

参数名 | 类型 | 描述
---|:---:|---
userId | NSString |  被监看的用户Id

**说明**

关闭成功会有回掉（onRTCRemoteCloseVideoRender），把显示的视图从父视图上删除掉


#### 34. 视频上报（接收端收到上报请求时，调用monitorVideo进行视频查看）

**定义**

```
- (int)reportVideo:(NSString *)userId;
```

**参数**

参数名 | 类型 | 描述
---|:---:|---
userId | NSString |  上报对象的用户的Id

**返回值**

0调用OK，-1未登录，-3:视频资源占用中，-5本操作不支持自己对自己。


#### 35. 上报者关闭上报

**定义**

```
- (void)closeReportVideo;
```

**说明**

上报功能只能自己关闭。


#### 36. 发送消息

**定义**

```
- (BOOL)sendUserMessage:(NSString *)userName userHeader:(NSString *)userHeaderUrl content:(NSString *)content;
```

**参数**

参数名 | 类型 | 描述
---|:---:|---
userName | NSString | 用户昵称(最大256字节)，不能为空，否则发送失败
userHeaderUrl | NSString | 用户头像(最大512字节)，可选
content | NSString | 消息内容(最大1024字节)不能为空，否则发送失败

**返回值**

YES/NO 发送成功/发送失败

**说明**

如果加入RTC（joinTalkGroupByToken）没有设置userId，发送失败


### ARMaxKitDelegate 回调类

#### 1. 加入群组成功回调

**定义**

```
- (void)onRTCJoinMaxGroupOk:(NSString*)groupId;
```

**参数**

参数名 | 类型 | 描述
---|:---:|---
groupId | NSString | 组ID

#### 2. 加入群组失败回调

**定义**

```
- (void)onRTCJoinMaxGroupFailed:(NSString*)groupId code:(ARMaxCode)code reason:(NSString*)reason;
```

**参数**

参数名 | 类型 | 描述
---|:---:|---
groupId | NSString | 组ID
code | ARMaxCode | 错误码
reason | NSString | 错误原因,rtc错误，或者token错误（错误值自己平台定义）

#### 3. 临时离开群组回调

**定义**

```
- (void)onRTCTempLeaveMaxGroup:(ARMaxCode)code;
```

**参数**

参数名 | 类型 | 描述
---|:---:|---
code | ARMaxCode | 错误码；0是正常退出；100：网络错误；207：强制退出。其他值参考错误码

#### 4. 离开群组回调

**定义**

```
- (void)onRTCLeaveMaxGroup:(ARMaxCode)code;
```

**参数**

参数名 | 类型 | 描述
---|:---:|---
code | ARMaxCode | 错误码；0是正常退出；100：网络错误；207：强制退出。其他值参考错误码

#### 5. 申请语音上麦成功回调

**定义**

```
- (void)onRTCApplyTalkOk;
```

#### 6. 申请语音上麦失败回调

**定义**

```
- (void)onRTCApplyTalkClosed:(ARMaxCode)code userId:(NSString *)userId userData:(NSString *)userData;
```

**参数**

参数名 | 类型 | 描述
---|:---:|---
code | ARMaxCode | 错误码；0是正常退出；100：网络错误；207：强制退出。其他值参考错误码
userId | NSString | 用户Id
userData | NSString | 用户自定义数据

#### 7. 其他人上麦成功回调

**定义**

```
- (void)onRTCTalkOn:(NSString*)userId userData:(NSString*)userData;
```

**参数**

参数名 | 类型 | 描述
---|:---:|---
userId | NSString | 用户Id
userData | NSString | 用户自定义数据

#### 8. 其他人下麦回调

**定义**

```
- (void)onRTCTalkClosed:(ARMaxCode)code userId:(NSString *)userId userData:(NSString *)userData;
```

**参数**
参数名 | 类型 | 描述
---|:---:|---
code | ARMaxCode | 错误码 0：正常结束对讲；其他参考错误码
userId | NSString | 用户Id
userData | NSString | 用户自定义数据

#### 9. SDK向应用层获取当前的网络状态

**定义**

```
- (BOOL)onRTCCheckNetSignalIsBad;
```

**说明**

如果是2G、3G网络，是弱网；如果是4G和WIFI，为非弱网。SDK为弱网传输做了深度优化

**返回值**

YES弱网，NO非弱网。

#### 10. 加入对讲组成功/切换对讲组成功回调

**定义**

```
- (void)onRTCJoinTalkGroupOK:(NSString *)groupId;
```

**参数**

参数名 | 类型 | 描述
---|:---:|---
groupId | NSString | 组ID

#### 11. 加入对讲组失败/切换对讲组失败回调

**定义**

```
- (void)onRTCJoinTalkGroupFailed:(NSString *)groupId code:(ARMaxCode)code reason:(NSString *)reason;
```

**参数**

参数名 | 类型 | 描述
---|:---:|---
groupId | NSString | 组ID
code | ARMaxCode | 错误码
reason | NSString | 错误原因,rtc错误，或者token错误（错误值自己平台定义）

#### 12. 临时断开群组回调

**定义**

```
- (void)onRTCTempLeaveTalkGroup:(ARMaxCode)code;
```

**参数**

参数名 | 类型 | 描述
---|:---:|---
code | ARMaxCode | 错误码

#### 13. 离开对讲组回调

**定义**

```
- (void)onRTCLeaveTalkGroup:(ARMaxCode)code;
```

**参数**

参数名 | 类型 | 描述
---|:---:|---
code | ARMaxCode | 错误码；0是正常退出；100：网络错误；207：强制退出。其他值参考错误码

#### 14. 当用户处于对讲状态时，控制台强制发起P2P通话的回调

**定义**

```
- (void)onRTCTalkP2POn:(NSString *)userId userData:(NSString *)userData;
```

**参数**

参数名 | 类型 | 描述
---|:---:|---
userId | NSString | 用户Id
userData | NSString | 用户自定义数据

#### 15. 与控制台的P2P讲话结束回调

**定义**

```
- (void)onRTCTalkP2POff:(NSString *)userData;
```

**参数**

参数名 | 类型 | 描述
---|:---:|---
userData | NSString | 用户自定义数据

#### 16. 收到监看请求回调

**定义**

```
- (void)onRTCVideoMonitorRequest:(NSString *)userId userData:(NSString *)userData;
```

**参数**

参数名 | 类型 | 描述
---|:---:|---
userId | NSString | 用户Id
userData | NSString | 用户自定义数据

#### 17. 视频监看结束回调

**定义**

```
- (void)onRTCVideoMonitorClose:(NSString*)userId userData:(NSString*)userData;
```

**参数**

参数名 | 类型 | 描述
---|:---:|---
userId | NSString | 用户Id
userData | NSString | 用户自定义数据

#### 18. 视频监看请求结果回调

**定义**

```
- (void)onRTCVideoMonitorResult:(NSString *)userId code:(ARMaxCode)code userData:(NSString *)userData;
```

**参数**

参数名 | 类型 | 描述
---|:---:|---
userId | NSString | 用户Id
code | ARMaxCode | 错误码
userData | NSString | 用户自定义数据

#### 19. 收到视频上报请求回调

**定义**

```
- (void)onRTCVideoReportRequest:(NSString *)userId userData:(NSString *)userData;
```

**参数**

参数名 | 类型 | 描述
---|:---:|---
userId | NSString | 用户Id
userData | NSString | 用户自定义数据

#### 20. 收到视频上报结束回调

**定义**

```
- (void)onRTCVideoReportClose:(NSString *)userId;
```

**参数**

参数名 | 类型 | 描述
---|:---:|---
userId | NSString | 用户Id

#### 21. 主叫方发起通话成功回调

**定义**

```
- (void)onRTCMakeCallOK:(NSString *)callId;
```

**参数**

参数名 | 类型 | 描述
---|:---:|---
callId | NSString | 呼叫Id

#### 22. 主叫方收到被叫方同意通话回调

**定义**

```
- (void)onRTCAcceptCall:(NSString *)userId userData:(NSString *)userData;
```

**参数**

参数名 | 类型 | 描述
---|:---:|---
userId | NSString | 用户Id
userData | NSString | 用户自定义数据

#### 23. 主叫方收到被叫方通话拒绝的回调

**定义**

```
- (void)onRTCRejectCall:(NSString *)userId code:(ARMaxCode)code userData:(NSString *)userData;
```

**参数**

参数名 | 类型 | 描述
---|:---:|---
userId | NSString | 用户Id
code | ARMaxCode | 错误码
userData | NSString | 用户自定义数据

#### 24. 被叫方调用leaveCall方法时，主叫方收到的回调

**定义**

```
- (void)onRTCLeaveCall:(NSString *)userId;
```

**参数**

参数名 | 类型 | 描述
---|:---:|---
userId | NSString | 用户Id

#### 25. 主叫方收到通话结束（被叫方和邀请发已全部退出或者主叫方挂断所有参与者）回调

**定义**

```
- (void)onRTCReleaseCall:(NSString *)callId;
```

**参数**

参数名 | 类型 | 描述
---|:---:|---
callId | NSString | 呼叫Id

#### 26. 被叫方收到通话请求回调

**定义**

```
- (void)onRTCMakeCall:(NSString *)callId callType:(int)callType userId:(NSString *)userId userData:(NSString *)userData;
```

**参数**

参数名 | 类型 | 描述
---|:---:|---
callId | NSString | 呼叫Id
callType | int | 呼叫类型:0:视频;1:音频
userId | NSString | 用户Id
userData | NSString | 用户自定义数据

#### 27. 被叫方收到主叫方挂断通话(收到该回掉，把本地视图从父视图删除)回调

**定义**

```
- (void)onRTCEndCall:(NSString *)callId userId:(NSString *)userId code:(ARMaxCode)code;
```

**参数**

参数名 | 类型 | 描述
---|:---:|---
callId | NSString | 呼叫Id
userId | NSString | 用户Id
code | ARMaxCode | 错误码

#### 28. 其他与会者加入（音视频）回调

**定义**

```
-(void)onRTCRemoteOpenVideoRender:(NSString *)peerId pubId:(NSString *)pubId userId:(NSString *)userId userData:(NSString *)userData;
```

**参数**

参数名 | 类型 | 描述
---|:---:|---
peerId | NSString | RTC服务生成的标识Id (用于标识与会者，每次加入会议随机生成)；
pubId | NSString | RTC服务生成流的Id (用于标识与会者发布的流)；
userId | NSString | 用户Id
userData | NSString | 用户自定义数据

#### 29. 其他与会者离开（音视频）回调

**定义**

```
-(void)onRTCCloseRemoteVideoRender:(NSString *)peerId pubId:(NSString *)pubId userId:(NSString *)userId;
```

**参数**

参数名 | 类型 | 描述
---|:---:|---
peerId | NSString | RTC服务生成的标识Id (用于标识与会者，每次加入会议随机生成)；
pubId | NSString | RTC服务生成流的Id (用于标识与会者发布的流)；
userId | NSString | 用户Id


#### 30. 其他与会者加入（音频）回调

**定义**

```
- (void)onRTCOpenRemoteAudioTrack:(NSString *)peerId userId:(NSString *)userId userData:(NSString *)userData;
```

**参数**

参数名 | 类型 | 描述
---|:---:|---
peerId | NSString | RTC服务生成的标识Id (用于标识与会者，每次加入会议随机生成)；
userId | NSString | 用户Id
userData | NSString | 用户自定义数据

#### 31. 其他与会者离开（音频）回调

**定义**

```
- (void)onRTCCloseRemoteAudioTrack:(NSString *)peerId userId:(NSString *)userId;
```

**参数**

参数名 | 类型 | 描述
---|:---:|---
peerId | NSString | RTC服务生成的标识Id (用于标识与会者，每次加入会议随机生成)；
userId | NSString | 用户Id

#### 32. 本地视频第一帧

**定义**

```
-(void)onRTCFirstLocalVideoFrame:(CGSize)size;
```

**参数**

参数名 | 类型 | 描述
---|:---:|---
size | CGSize | 视频窗口大小

#### 33. 远程视频第一帧

**定义**

```
- (void)onRTCFirstRemoteVideoFrame:(CGSize)size pubId:(NSString *)pubId;
```

**参数**

参数名 | 类型 | 描述
---|:---:|---
size | CGSize | 视频窗口大小
pubId | NSString | RTC服务生成流的Id (用于标识与会者发布的流)

#### 34. 本地窗口大小的回调

**定义**

```
- (void)onRTCLocalVideoViewChanged:(CGSize)size;
```

**参数**

参数名 | 类型 | 描述
---|:---:|---
size | CGSize | 视频窗口大小

#### 35. 远程窗口大小的回调

**定义**

```
- (void)onRTCRemoteVideoViewChanged:(CGSize)size pubId:(NSString *)pubId;
```

**参数**

参数名 | 类型 | 描述
---|:---:|---
size | CGSize | 视频窗口大小
pubId | NSString | RTC服务生成流的Id (用于标识与会者发布的流)


#### 36. 其他与会者对音视频的操作回调

**定义**

```
- (void)onRTCRemoteAVStatus:(NSString *)peerId audio:(BOOL)audio video:(BOOL)video;
```

**参数**

参数名 | 类型 | 描述
---|:---:|---
peerId | NSString | RTC服务生成的标识Id (用于标识与会者，每次加入会议随机生成)
audio | BOOL | YES为打开音频，NO为关闭音频
video | BOOL | YES为打开视频，NO为关闭视频

#### 37. 别人对自己音视频的操作回调

**定义**

```
- (void)onRTCLocalAVStatus:(BOOL)audio video:(BOOL)video;
```

**参数**

参数名 | 类型 | 描述
---|:---:|---
audio | BOOL | YES为打开音频，NO为关闭音频
video | BOOL | YES为打开视频，NO为关闭视频


#### 38. 其他与会者的声音大小回调

**定义**

```
- (void)onRTCRemoteAudioActive:(NSString *)peerId userId:(NSString *)userId audioLevel:(int)level showTime:(int)time;
```
**参数**

参数名 | 类型 | 描述
---|:---:|---
peerId | NSString | RTCP服务生成的通道Id (用于标识通道，每次发布随机生成)
userId | NSString | 自己平台的用户Id
level | int | 音频大小（0~100）
time | int | 音频检测在nTime毫秒内不会再回调该方法（单位：毫秒）

**说明**

与会者关闭音频后（setLocalAudioEnable为NO）,该回调将不再回调。对方关闭音频检测后（setAudioActiveCheck为NO）,该回调也将不再回调。

#### 39. 本地声音大小回调

**定义**

```
- (void)onRTCLocalAudioActive:(int)level showTime:(int)time;
```
**参数**

参数名 | 类型 | 描述
---|:---:|---
level | int | 音频大小（0~100）
time | int | 音频检测在nTime毫秒内不会再回调该方法（单位：毫秒）

**说明**

本地关闭音频后（setLocalAudioEnable为NO）,该回调将不再回调。本地关闭音频检测后（setAudioActiveCheck为NO）,该回调也将不再回调。

#### 40. 其他与会者网络质量回调

**定义**

```
- (void)onRTCRemoteNetworkStatus:(NSString *)peerId userId:(NSString *)userId netSpeed:(int)netSpeed packetLost:(int)packetLost netQuality:(ARNetQuality)netQuality;
```
**参数**

参数名 | 类型 | 描述
---|:---:|---
peerId | NSString | RTCP服务生成的通道Id (用于标识通道，每次发布随机生成)
userId | NSString | 用户平台Id
netSpeed | int | 网络上行
packetLost | int | 丢包率
netQuality | ARNetQuality | 网络质量

#### 41. 本地网络质量回调

**定义**

```
- (void)onRTCLocalNetworkStatus:(int)netSpeed packetLost:(int)packetLost netQuality:(ARNetQuality)netQuality;
```
**参数**

参数名 | 类型 | 描述
---|:---:|---
netSpeed | int | 网络上行
packetLost | int | 丢包率
netQuality | ARNetQuality | 网络质量

#### 42. 流量状态

**定义**

```
- (void)onRTCNetStatsAll:(int)allBytes sendBytes:(int)sendBytes recvBytes:(int)recvBytes detail:(NSString*)details;
```
**参数**

参数名 | 类型 | 描述
---|:---:|---
allBytes | int | 总的流量
sendBytes | int | 发送的流量
recvBytes | int | 接收的流量
details | NSString | 详情

#### 43. 录像地址回调信息回调

**定义**

```
- (void)onRTCGotRecordFile:(int)recordType userData:(NSString *)userData filePath:(NSString *)filePath;
```
**参数**

参数名 | 类型 | 描述
---|:---:|---
recordType | int | 录音的类型（0/1/2/3：对讲本地录音/对讲远端录音/强插P2P录音/语音通话呼叫录音）
userData | NSString | 用户自定义数据
filePath | NSString | 录音文件的路径

#### 44. 收到消息回调

**定义**

```
- (void)onRTCUserMessage:(NSString *)userId userName:(NSString *)userName userHeader:(NSString *)userHeaderUrl content:(NSString *)content;
```
**参数**

参数名 | 类型 | 描述
---|:---:|---
userId | NSString | 发送消息者在自己平台下的Id
userName | NSString | 发送消息者的昵称
userHeaderUrl | NSString | 发送者的头像
content | NSString | 消息内容

#### 45. 当前对讲组在线人数回调

**定义**

```
- (void)onRTCMemberNum:(int)num;
```

**参数**

参数名 | 类型 | 描述
---|:---:|---
num | int | 当前在线人员数量

#### 46. 第三方用户平台自定义消息通知

**定义**

```
- (void)onRTCUserDataNotify:(NSString*)userData;
```

**参数**

参数名 | 类型 | 描述
---|:---:|---
userData | NSString | 自定义消息内容


## 三、更新日志

**Version 3.0.1.7 （2019-10-15）**

* 修复bug

**Version 3.0.0 （2019-05-15）**

* SDK版本升级3.0，API接口变更，更加简洁规范
* 更改SDK配置项，配置更简单
* 断网重连优化
* 分辨率选项增加，帧率可配置
* 添加用户服务级token安全认证，服务更安全

**Version 2.0.0 （2017-09-30）**

* SDK版本升级2.0，梳理、完善SDK




