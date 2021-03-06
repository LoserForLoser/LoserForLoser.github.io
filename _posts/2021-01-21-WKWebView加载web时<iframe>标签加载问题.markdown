---
layout: post
title:  "WKWebView加载web时<iframe>标签加载问题"
date:   2021-01-21 09:30:30 +0800
categories: jekyll update
---
之前做项目的时候有需求是加载 web 并做交互

按照那个之前的思路就是简单的创建一个 webView (xib 拖的)
```
@property (weak, nonatomic) IBOutlet WKWebView *webView;

- (void)viewDidLoad {
    [super viewDidLoad];
    
    @weakify(self)
    self.navigationItem.leftBarButtonItem = [AppTheme backItemWithHandler:^(id sender) {
        @strongify(self)
        if (self.webView.canGoBack) {
            self.tabView.hidden = YES;
            self.tabViewHeightLC.constant = 0;
            self.webState = WKWebStateWebView;
            [self.webView goBack];
        } else {
            [self.navigationController popViewControllerAnimated:YES];
        }
    }];
    
    self.webView.configuration.allowsInlineMediaPlayback = YES;
    self.webView.navigationDelegate = self;
//    self.webView.allowsBackForwardNavigationGestures = YES;
    self.webView.scrollView.delegate = self;
    [self.webView addObserver:self forKeyPath:@"estimatedProgress" options:NSKeyValueObservingOptionNew context:nil];
    
    NSURL *url = [NSURL URLWithString:[self stringHandle:self.webURL]];
    NSURLRequest *request = [[NSURLRequest alloc] initWithURL:url];
    [self.webView loadRequest:request];
}

#pragma mark - WKNavigationDelegat

......

#pragma mark - WKScriptMessageHandler

......

```

但是在运行打开 web 的时候由于 web 里面添加了 <iframe> 标签进行部分模块资源的二次加载，造成 webView 的 reload 十分影响用户体验

为此在经过思考和询问前端的同学以后，决定通过 JS 注入的方式让移除/禁用掉 <iframe> 标签来预防此事

```
// 可以加个判断在主 web URL 加载完成后进行注入， <iframe> 标签是在当前 web 加载完成后立即加载
- (void)webView:(WKWebView *)webView decidePolicyForNavigationAction:(WKNavigationAction *)navigationAction decisionHandler:(void (^)(WKNavigationActionPolicy))decisionHandler {
    [self.webView evaluateJavaScript:@"document.querySelectorAll('iframe').forEach(function (item) { item.remove() })" completionHandler:nil];
}
```

运行之后发现效果非常不错！

但是！

如果有同学之前查过相关问题的话会发现另一个问题：

好多视频播放的网站他的视频播放地址是放在 <iframe> 标签里的！

这就很头疼了……

但是在我查 <iframe> 标签问题的一个偶然的机会下发现了这么一个东西：

** WKNavigationAction **

恰好就在 - (void)webView:(WKWebView *)webView decidePolicyForNavigationAction:(WKNavigationAction *)navigationAction decisionHandler:(void (^)(WKNavigationActionPolicy))decisionHandler; 中有 WKNavigationAction，经过阅读后了解到：
> 这个方法的参数WKNavigationAction中有两个属性: sourceFrame和targetFrame, 分别代表这个action的出处和目标。类型是WKFrameInfo。WKFrameInfo有 一个mainFrame的属性，正是这个属性标记着这个frame是在主frame里还是新开一个frame。如果targetFrame的mainFrame属性为NO，表明这个WKNavigationAction将会新开一个页面。

好的！皮卡丘，就决定是你了！

通过对 sourceFrame.mainFrame 和 targetFrame.mainFrame 进行打印后发现正常的打开/跳转 web 页面出现的是 YES & NO 或 NO & YES 这种形式，而<iframe> 标签内链接返回的是 NO & NO 这种形式，因此改为：

```
// 可以加个判断在主 web URL 加载完成后进行注入， <iframe> 标签是在当前 web 加载完成后立即加载
- (void)webView:(WKWebView *)webView decidePolicyForNavigationAction:(WKNavigationAction *)navigationAction decisionHandler:(void (^)(WKNavigationActionPolicy))decisionHandler {
    BOOL sourceIsMain = navigationAction.sourceFrame.mainFrame;
    BOOL targetIsMain = navigationAction.targetFrame.mainFrame;
    if !sourceIsMain && !targetIsMain {
        // 不允许跳转
        NSLog(@"不允许跳转");
        decisionHandler(WKNavigationActionPolicyCancel);
    } else {
        // 允许跳转
        NSLog(@"允许跳转");
        decisionHandler(WKNavigationActionPolicyAllow);
    }
}
```

再次运行：完美拦截 <iframe> 标签加载造成的多余跳转，且 <iframe> 标签内视频正常播放！

至此，有关 <iframe> 标签加载造成的问题就圆满解决了！

（题外话：在 web 浏览器上点击链接后是新开个标签打开页面还是在当前页面加载是通过是否有 js 的 window.open 来决定的，担心 <iframe> 标签与此同理或近似，但实际测试时发现我们项目新链接的打开时走不走 window.open 的都有切都能正常跳转，因此推测 <iframe> 标签应该与此不同，故可放心使用）

此外，经过近几次项目使用后发现，WKWebView 比 UIWebView 是真的香，但是好多东西一轻量化以后就不好找了，或者说不读官方文档不懂得就多了，与君共勉
