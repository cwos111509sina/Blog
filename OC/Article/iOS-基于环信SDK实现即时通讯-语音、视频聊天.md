

## iOS 基于环信SDK实现即时通讯-语音、视频聊天


这里创建的项目是在[文字聊天项目](https://cwos111509sina.github.io/Blog/iOS-基于环信SDK实现即时通讯-文字聊天)基础上添加的功能，所以可能需要先去链接文章地址查看集成过程，具体项目链接在下面，这里只介绍使用环信SDK集成语音、视频通话:  
需要用到的内容有：  

EMCallManagerDelegate 语音视频代理   
AVFoundation 音频输出   
EMCallSession 会话信息  

在info.plist里面添加：  
```
<key>NSMicrophoneUsageDescription</key>
    <string>是否允许此App使用你的麦克风？</string>
<key>NSCameraUsageDescription</key>
    <string>是否允许此App使用你的相机？</string>
```
添加代理方法为：  
```
    [[EMClient sharedClient].callManager addDelegate:self delegateQueue:nil];
```
使用到的代理方法主要有： 

(void)callDidReceive:(EMCallSession *)aSession 
//用户A拨打用户B用户B会收到这个回调、你希望在哪个页面可以监听被呼叫就把这个方法写在里面，记得遵守协议；

(void)callDidConnect:(EMCallSession *)aSession 
//通话通道完成，可以在这里创建音频输出设备和环境AVAudioSession

(void)callDidAccept:(EMCallSession *)aSession 
//用户B同意用户A的通话请求后，用户A会收到这个回调

(void)callDidEnd:(EMCallSession )aSession reason:(EMCallEndReason)aReason error:(EMError )aError
//用户A或用户B挂断后对方会收到这个回调。或者通话出现错误、双方都会收到该回调

创建一个语音或者视频通话：


```
/*
*  @param aType            通话类型
 *  @param aRemoteName      被呼叫的用户（不能与自己通话）
 *  @param aExt             通话扩展信息，会传给被呼叫方
 *  @param aCompletionBlock 完成的回调
*/
[[EMClient sharedClient].callManager startCall:aType remoteName:aRemoteName ext:aExt completion:^(EMCallSession *aCallSession, EMError *aError) {

       if (!aError) {//创建成功
       }else{
       }

}];
```

同意别人的会话邀请：

```
/*
_callSession.callId 会话ID
*/
 [[EMClient sharedClient].callManager answerIncomingCall:_callSession.callId];
 
```
结束通话：

```
/*
_callSession.callId 会话ID
aReason     挂断原因 （EMCallEndReason） 
*/
[[EMClient sharedClient].callManager endCall:_callSession.callId reason:aReason];
```

上面这些东西已经可以完成一个简单的语音、视频通话需求，具体实现可以到下面链接下载查看

[项目地址](https://github.com/cwos111509sina/EMChatText.git)


[返回首页](https://cwos111509sina.github.io/Blog/)
