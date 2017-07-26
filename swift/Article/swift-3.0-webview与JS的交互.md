
##  swift 3.0 webview与JS的交互


最近接触了个web App项目，手痒就选择了Swift来写，这里主要介绍一些swift中webview和js交互的事项

首先import JavaScriptCore

然后定义需要和JS交互的协议内容（JS需要调用的方法）

／*

@objc protocol （@objc必须添加）

goToClipboar（js调用的本地方法名称）

(urlStr : NSString)（js传过来的内容，格式不定）

*／

@objc protocol /协议名称根据喜好/ : JSExport{

func goToClipboard(urlStr : NSString)

}  
``` 
import JavaScriptCore
@objc protocol JSObjcDelegate : JSExport{
    
    func goToClipboard(urlStr : NSString)
}

```


上面是与js交互的方法，下面开始webview需要设置内容的介绍

首先遵守协议，实例化JSContext，实现协议方法 

```
class webViewPbj : UIWebView , UIWebViewDelegate , JSObjcDelegate {
  
  var jsContext : JSContext?
  
  internal func goToClipboard(urlStr : NSString){
  
  
 }
}
```

然后去webViewDidFinishLoad方法里面进行关联 


```

func webViewdidFinishLoad(_ webView: UIWebView){
  jsContext = webView.value(forKeyPath:"documentView.webView.mainFrame.javaScriptContext") as! JSContext!
  jsContext!.setObject(self.forKeyedSubscript: "myObj" as (NSCopying & NSObjectProtocol)!)
}

```


到这里webview与JS交互内容就结束了。刚刚接触，还有很多不足，欢迎指导。下面完整代码



```
import JavaScriptCore

@objc protocol JSObjcDelegate : JSExport{

func goToClipboard(urlStr : NSString)//点击顶部返回按键

}

class webViewObj: UIWebView , UIWebViewDelegate , JSObjcDelegate{

var jsContext : JSContext?

func webViewDidFinishLoad(_ webView: UIWebView) {

jsContext = webView.value(forKeyPath:"documentView.webView.mainFrame.javaScriptContext") as! JSContext!

jsContext!.setObject(self, forKeyedSubscript: "myObj" as (NSCopying & NSObjectProtocol)!)

}

internal func goToClipboard(urlStr: NSString) {

print("goToClipboard")

}

}

```



[返回首页](https://cwos111509sina.github.io/Blog/)
