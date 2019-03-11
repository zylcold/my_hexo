---
title: Javascript<=>Native交互
date: 2018-12-27 10:04:27
tags: 
- iOS
- Javascript
---

[toc]

## Javascript => Native 交互


### 拦截URL Scheme
> 监听特定跳转，拦截并处理

#### Web端

```js
window.location.href = "url_scheme://event_name?param1=foo&param2=foo2"

//URL Scheme是一种页面内跳转协议,也可以被称为URLRouter
//url_scheme 拦截协议
//event_name 执行事件
//param1=foo&param2=foo2 参数列表
```

#### Native端

1. 实现WebView的跳转代理

```objc
self.wkWebView.navigationDelegate = self;
//or
self.uiWebView.delegate = self;
```
    
    
2. 监听URL Scheme调用
    
```swift

//webView
func webView(_ webView: UIWebView, shouldStartLoadWith request: URLRequest, 
        navigationType: UIWebViewNavigationType) -> Bool {
    guard let url = request.url else { return true }
    //解析url
    let urlComponents = NSURLComponents(url: url, resolvingAgainstBaseURL: true)
    //比较scheme
    if urlComponents?.scheme == "url_scheme" {
        //解析Query
        let params = parseUrlQueryToParams(query: urlComponents?.query)
        //开始处理
        startHandleEvent(event: urlComponents?.host, params: params)
        return false
    }
    return true
}


//wkWebView

func webView(_ webView: WKWebView, decidePolicyFor 
      navigationAction: WKNavigationAction, 
       decisionHandler: @escaping (WKNavigationActionPolicy) -> Void) {
    guard let url = navigationAction.request.url else {
        decisionHandler(.allow)
        return
    }
    var isGoAhead = true
    let urlComponents = NSURLComponents(url: url, resolvingAgainstBaseURL: true)
    //比较scheme
    if urlComponents?.scheme == "url_scheme" {
        //解析Query
        let params = parseUrlQueryToParams(query: urlComponents?.query)
        //开始处理
        startHandleEvent(event: urlComponents?.host, params: params)
        isGoAhead = false
    }

    decisionHandler(isGoAhead ? .allow : .cancel)
}

```
    
<!-- more -->

### 注入方法

> 拿到WebView的JS上下文，向上下文中注入方法。

#### Web端

```js
    foo.sayHello("zyl") //(1)注入对象
    sayHello("zyl") //(2)注入方法
```

#### Native端

1. 获取WebView的JS上下文
   
```swift
func webViewDidFinishLoad(_ webView: UIWebView) {
    self.webContent = webView.value(forKey: "documentView.webView.mainFrame.javaScriptContext") as? JSContext
    
}
```
2. 注入方法

```swift

//创建协议
protocol FooJSExports: JSExport {
    func sayHello(name: String) -> Void
}
//创建对象
class Foo: NSObject, FooJSExports {
    func sayHello(name: String) {
        print("Hello \(name)")
    }
}

//注入
func injectJSEvent() {
    guard let jsContent = self.webContent else {
        return
    }
    
    //(1)注入对象
    jsContent.setObject(Foo(), forKeyedSubscript: "foo" as NSString)

    //(2)注入函数
    jsContent.setObject({
        (name:String) in
        print("Hello \(name)")
    }, forKeyedSubscript: "sayHello" as NSString)
}


```


### MessageHandler
#### Web端

```js
window.webkit.messageHandlers.<name>.postMessage(<messageBody>)
//e.g
//window.webkit.messageHandlers.Share.postMessage(<messageBody>)
// 在`WKScriptMessageHandler`协议中，
//name -> mssage是`WKScriptMessage`类型，name属性。
//messageBody -> mssage是`WKScriptMessage`类型，body属性。
//body 的类型：
//Allowed types are NSNumber, NSString, NSDate, NSArray, NSDictionary, and NSNull.
```
#### Native端
1. 创建WKWebViewConfiguration

```swift
let config = WKWebViewConfiguration()
let preferences = WKPreferences()
preferences.javaScriptCanOpenWindowsAutomatically = true;
preferences.minimumFontSize = 40.0;
config.preferences = preferences
self.wkWebView = WKWebView(frame: self.view.bounds, configuration: config)
```

2. 添加响应事件

```swift
self.wkWebView?.configuration.userContentController.add(self, name: "sayHello")
```

3. 实现协议，添加处理

```swift
extension ViewController: WKScriptMessageHandler {
    func userContentController(_ userContentController: WKUserContentController, 
                                    didReceive message: WKScriptMessage) {
        if message.name == "sayHello" {
            print("Hello \(message.body as? String ?? "null")")
        }
    }
}
```


### 拦截TextInputPanel事件
#### Web端

```js
function callNativeApp(){
    try {
        var name = "sayHello"
        var data = {name:"zyl"}
        var res = prompt("YLbridge_"+name, JSON.stringify(type))
    	retrun res
    } catch(err) {
        console.log('The native context does not exist yet')
    }
}
```

#### Native端

```swift
func webView(_ webView: WKWebView, runJavaScriptTextInputPanelWithPrompt prompt: String, 
           defaultText: String?, initiatedByFrame frame: WKFrameInfo, 
     completionHandler: @escaping (String?) -> Void) {
    //判断前缀
    guard prompt.hasPrefix("YLbridge_") else {
        completionHandler(defaultText)
        return
    }
    //判断方法
    if prompt == "YLbridge_sayName" {
        //获取参数
        if let data = defaultText?.data(using: .utf8), 
           let param = (try? JSONSerialization.jsonObject(with: data, options: .allowFragments)) 
               as? [NSString: Any]
        {
            //执行
            print("Hello \(param["name"] as? String ?? "")")
            //返回结果
            completionHandler("succeed");
        }else {
            completionHandler("not param");
        }
    }
}
```

### 总结

| 方式 |实现 (JS-Nv)| 维护/拓展  | 同步 | 致命伤 |
| --- | ---|--- | --- | --- |
| 拦截URLScheme | 10/8 | 5 |  不支持| 扩展困难
| 注入方法 | 10/9| 8 | 支持 | 仅UIWebView，循环引用
| MessageHandler | 8/9 | 7 | 不支持 |仅WKWebView,循环引用
| 拦截TextInputPanel事件 | 7/7 | 6 | 支持 |实现困难


## Native => Javascript 交互
### WebView

```swift

self.webView.stringByEvaluatingJavaScript(from: "jsSayHello(\("zyl"))")
        
self.wkWebView?.evaluateJavaScript("jsSayHello(\("zyl"))") { (res, error) in
    print("res \(String(describing: res))")
}
```

### JSContent

```swift
let value = jsContent.objectForKeyedSubscript("jsSayHello")?.call(withArguments: ["zyl"])
print("res \(value?.toString() ?? "")")
```