
## iOS 基于环信SDK实现即时通讯-文字聊天


这里介绍集成环信SDK3.0自定义聊天页面，后面有练习项目地址    
首先到环信官网下载环信SDK、由于后续会实现语音、视频，我这里使用的是带有语音的SDK     

下载完成后把HyphenateFullSDK文件夹拉入项目：   

![](https://cwos111509sina.github.io/Blog/OC/Source/iOS-基于环信SDK实现即时通讯-文字聊天/chat_0.jpeg)  
在Embedded Binaries目录下点击加号添加Hyphenate.framework:   
![](https://cwos111509sina.github.io/Blog/OC/Source/iOS-基于环信SDK实现即时通讯-文字聊天/chat_1.jpeg)  

在AppDelegate.m导入环信库文件   
#import <Hyphenate/Hyphenate.h>   
编译通过。  

在- (BOOL)application:(UIApplication )application didFinishLaunchingWithOptions:(NSDictionary )launchOptions 方法中注册环信  
```
  EMOptions *options = [EMOptions optionsWithAppkey:@"1156170310178775#emchattext"];//这里的key是自己在环信注册应用时环信给的appkey
  [[EMClient sharedClient] initializeSDKWithOptions:options];
```
![](https://cwos111509sina.github.io/Blog/OC/Source/iOS-基于环信SDK实现即时通讯-文字聊天/chat_2.jpeg)  
在- (void)applicationDidEnterBackground:(UIApplication *)application 程序进入后台方法中调用环信方法  
```
    [[EMClient sharedClient] applicationDidEnterBackground:application];
```
在- (void)applicationWillEnterForeground:(UIApplication *)application 程序即将从后台返回方法中调用环信方法  
```
    [[EMClient sharedClient] applicationWillEnterForeground:application];
```

这样环信SDK集成工作已经做完，下面介绍实现聊天需要调用的实现方法：

1、登录、注册、退出方法  

```
//登陆方法
 EMError *error = [[EMClient sharedClient] loginWithUsername:@"账号" password:@"密码"];
        if (!error) {
            NSLog(@"登录成功");
            [[EMClient sharedClient].options setIsAutoLogin:YES];//设置自动登录

        }else{
            NSLog(@"登录失败 ： error = %@",error.errorDescription);
        }

//注册方法
EMError *error = [[EMClient sharedClient] registerWithUsername:@"账号" password:@"密码"];
        if (error == nil) {
            NSLog(@"注册成功");
        }else{
            NSLog(@"注册失败 ： error = %@",error.errorDescription);
        }
//退出
[[EMClient sharedClient] logout:YES completion:^(EMError *aError) {
        if (!aError.code) {

        }
    }];
``` 
2、创建聊天

EMConversation 聊天会话实例，发送获取消息。。。

下面介绍用到及调用的方法  
```
- (void)viewDidLoad {
    [super viewDidLoad];

    _conversation = [[EMClient sharedClient].chatManager getConversation:@"会话ID传入对方环信Id即可" type:EMConversationTypeChat createIfNotExist:YES];

    //移除消息回调
    [[EMClient sharedClient].chatManager removeDelegate:self];

    //注册消息回调
    [[EMClient sharedClient].chatManager addDelegate:self delegateQueue:nil];

    // Do any additional setup after loading the view.
}


[_conversation loadMessagesStartFromId:@"会话ID（同上）" count:20 searchDirection:EMMessageSearchDirectionUp completion:^(NSArray *aMessages, EMError *aError) {

        if (!aError) {
            NSLog(@"获取本地消息内容成功");
        }else{

            NSLog(@"获取本地消息内容失败：error = %@",aError.errorDescription);
        }

    }];


/*
接收消息回调消息类型(接收到别人发送的消息可以在这里进行相应的处理：添加数据、修改视图。。。)

EMMessageBodyTypeText   = 1,    文本类型 
EMMessageBodyTypeImage,         图片类型 EMMessageBodyTypeVideo,         视频类型 
EMMessageBodyTypeLocation,      位置类型 EMMessageBodyTypeVoice,        语音类型 
EMMessageBodyTypeFile,          文件类型
EMMessageBodyTypeCmd,           命令类型

*/
-(void)messagesDidReceive:(NSArray *)aMessages{
    for (EMMessage *message in aMessages) {
        if (message.body.type == EMMessageBodyTypeText) {
            NSLog(@"接收到文字消息");
        }

    }

}


//发送消息方法

//发送内容
    EMTextMessageBody *body = [[EMTextMessageBody alloc] initWithText:@"文字消息类型（发送文字）"];
    NSString *from = [[EMClient sharedClient] currentUsername];

    //生成Message
    EMMessage *message = [[EMMessage alloc] initWithConversationID:@"会话ID(同上)" body:body ext:nil];

    message.chatType = EMChatTypeChat;// 设置为单聊消息

    [[EMClient sharedClient].chatManager sendMessage:message progress:nil completion:^(EMMessage *message, EMError *error) {

        if (!error) {
            NSLog(@"消息发送成功");
        }else{
            NSLog(@"消息发送失败 ： error = %@",error.errorDescription);
        }

    }];
```

[项目地址](https://github.com/cwos111509sina/EMChatText.git)  

如果运行报下面错误   

![](https://cwos111509sina.github.io/Blog/OC/Source/iOS-基于环信SDK实现即时通讯-文字聊天/chat_3.jpeg)  
需要自己下载环信SDK文件中的HyphoenateFullSDK/Hyphoenate.framework/Hyphoenate放到EMChatText项目HyphoenateFullSDK/Hyphoenate.framework目录下，下图：   
![](https://cwos111509sina.github.io/Blog/OC/Source/iOS-基于环信SDK实现即时通讯-文字聊天/chat_4.jpeg)

[返回首页](https://cwos111509sina.github.io/Blog/)
