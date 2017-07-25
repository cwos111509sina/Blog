

##  iOS webview与Js的交互。以及修改cookie、header头

<br>

第一部分：webview与Js的交互   

第二部分：设定cookie   

第三部分：修改header头   

（觉得有用可以给个星星）  

一：交互  

```

#import "webView.h"

#import <JavaScriptCore/JavaScriptCore.h>//系统支持库

@protocol JSObjcDelegate <JSExport>//定义web与JS交互的协议
-(void)goToNextActivity:(NSString *)urlStr;//JS需要调用的方法，参数可有可无，根据需要设定
@end

@interface webView ()<UIWebViewDelegate,JSObjcDelegate>//遵守协议
@property (nonatomic,strong)JSContext * jsContext;//创建JSContext对象 我把它当作上下文对象，用于连接JS和OC


@end

@implementation webView


-(instancetype)initWithFrame:(CGRect)frame url:(NSURL *)url{

    self = [super initWithFrame:frame];

    if (self) {
        NSURLRequest *request = [NSURLRequest requestWithURL:url];
        self.delegate = self;

        [self loadRequest:request];
    }

    return self;
}

-(void)webViewDidFinishLoad:(UIWebView *)webView{

    self.jsContext = [webView valueForKeyPath:@"documentView.webView.mainFrame.javaScriptContext"];

    self.jsContext[@"myObj"] = self;//这里把self赋值给self.jsContext的myObj对象，那么在JS中就可以通过myObj调用oc方法了
    self.jsContext.exceptionHandler = ^(JSContext *context, JSValue *exceptionValue) {
        context.exception = exceptionValue;
        NSLog(@"异常信息：%@", exceptionValue);
    };

}


//webview调用方法
-(void)goToNextActivity:(NSString *)urlStr{

    NSLog(@"goToNextActivity str:%@",urlStr);


}


@end
```

二：cookie  
这里是每次请求的时候都进行设定，所以方法写在了web开始加载的方法里面

```
- (BOOL)webView:(UIWebView *)webView shouldStartLoadWithRequest:(NSURLRequest *)request navigationType:(UIWebViewNavigationType)navigationType{

    NSDictionary *dic = @{@"TOKEN":[DEFAULTS objectForKey:@"token"],@"UID":[DEFAULTS objectForKey:@"userID"]};
    [dic enumerateKeysAndObjectsUsingBlock:^(id key, id obj, BOOL *stop) {
        // 设定 cookie
        NSHTTPCookie *cookie = [NSHTTPCookie cookieWithProperties:
                                [NSDictionary dictionaryWithObjectsAndKeys:
                                 [request.URL host], NSHTTPCookieDomain,
                                 [request.URL path], NSHTTPCookiePath,
                                 key,NSHTTPCookieName,
                                 obj,NSHTTPCookieValue,
                                 nil]];

        [[NSHTTPCookieStorage sharedHTTPCookieStorage] setCookie:cookie];
//        NSLog(@"cookie = %@",cookie);

    }];


    return YES;

}
```
三：header   
同样是写在了webview开始加载的方法里面

```
-(BOOL)webView:(UIWebView *)webView shouldStartLoadWithRequest:(NSURLRequest *)request navigationType:(UIWebViewNavigationType)navigationType{

    NSMutableURLRequest *mutableRequest = [request mutableCopy];
//这里加了一些判断，避免死循环，因为修改完成后webview需要重新加载request （其中DEFAULTS 是NSUserDefaults存了一些后台程序返回的内容）
    if (!request.allHTTPHeaderFields[@"CITY"]) {

        [mutableRequest addValue:[DEFAULTS objectForKey:@"city"] forHTTPHeaderField:@"CITY"];

        [self loadRequest:request];
    }else if(![[DEFAULTS objectForKey:@"city"] isEqualToString:request.allHTTPHeaderFields[@"CITY"]]) {

        [mutableRequest setValue:[DEFAULTS objectForKey:@"city"] forHTTPHeaderField:@"CITY"];

        request = [mutableRequest copy];
        [self loadRequest:request];
    }


    return YES;

}
```

[返回首页](https://cwos111509sina.github.io/Blog/)

