
----
#### js调oc 并返回值给js

简书说明 https://www.jianshu.com/p/83fc78132d03

![js调oc 并返回值给js.gif](https://upload-images.jianshu.io/upload_images/2384741-af013bb82347e671.gif?imageMogr2/auto-orient/strip)

##### 1.js 点击‘获取定位’

```
<input type="button" value="获取定位" onclick="locationClick()" />

function locationClick() {
                window.webkit.messageHandlers.Location.postMessage(null);
            }
```


#####2 .ios 内注册 js  “Location” 命令

>  -- (void)viewWillAppear:(BOOL)animated
{
    [super viewWillAppear:animated];
    // addScriptMessageHandler 很容易导致循环引用
    // 控制器 强引用了WKWebView,WKWebView copy(强引用了）configuration， configuration copy （强引用了）userContentController
    // userContentController 强引用了 self （控制器）
    [self.webView.configuration.userContentController addScriptMessageHandler:self name:@"Location"];
}

##### 3.  ios 得到 js 发送的“Location” 命令
```
#pragma mark - WKScriptMessageHandler
- (void)userContentController:(WKUserContentController *)userContentController didReceiveScriptMessage:(WKScriptMessage *)message
{
//    message.body  --  Allowed types are NSNumber, NSString, NSDate, NSArray,NSDictionary, and NSNull.
    NSLog(@"body:%@",message.body);
if ([message.name isEqualToString:@"Location"]) {
        [self getLocation];
    } 
}
```
#### 4. js 接收ios 结果回调
```
function setLocation(location) {
                asyncAlert(location);
                document.getElementById("returnValue").value = location;
            }
```
##### 5. ios 将结果传给 js
```
- (void)getLocation
{
    // 获取位置信息
    
    // 将结果返回给js
    NSString *jsStr = [NSString stringWithFormat:@"setLocation('%@')",@"广东省深圳市南山区学府路XXXX号"];
    [self.webView evaluateJavaScript:jsStr completionHandler:^(id _Nullable result, NSError * _Nullable error) {
        NSLog(@"%@----%@",result, error);
    }];
   
}
```


js ------- index.html
```
<html>
    <head>
        <meta http-equiv="Content-Type" content="text/html; charset=utf8">
            <script language="javascript">
            var ctuapp_share_img="www.baidu.com";
            function scanClick() {
                window.webkit.messageHandlers.ScanAction.postMessage(null);
            }
                
            function shareClick() {
                window.webkit.messageHandlers.Share.postMessage({title:'测试分享的标题',content:'测试分享的内容',url:'http://m.rblcmall.com/share/openShare.htm?share_uuid=shdfxdfdsfsdfs&share_url=http://m.rblcmall.com/store_index_32787.htm&imagePath=http://c.hiphotos.baidu.com/image/pic/item/f3d3572c11dfa9ec78e256df60d0f703908fc12e.jpg'});
            }
            
            function locationClick() {
                window.webkit.messageHandlers.Location.postMessage(null);
            }
            
            function setLocation(location) {
                asyncAlert(location);
                document.getElementById("returnValue").value = location;
            }
            
            function getQRCode(result) {
                asyncAlert(result);
                document.getElementById("returnValue").value = result;
            }
            
            function colorClick() {
                window.webkit.messageHandlers.Color.postMessage([67,205,128,0.5]);
            }
            
            function payClick() {
                window.webkit.messageHandlers.Pay.postMessage({order_no:'201511120981234',channel:'wx',amount:1,subject:'粉色外套'});
            }
            
            function payResult(str) {
                asyncAlert(str);
                document.getElementById("returnValue").value = str;
            }
            
            function shareResult(channel_id,share_channel,share_url) {
                var content = channel_id+","+share_channel+","+share_url;
                asyncAlert(content);
                document.getElementById("returnValue").value = content;
            }

            function shake() {
                window.webkit.messageHandlers.Shake.postMessage(null);
            }
            
            function goBack() {
                window.webkit.messageHandlers.GoBack.postMessage(null);
            }
            
            function playSound() {
                window.webkit.messageHandlers.PlaySound.postMessage('shake_sound_male.wav');
            }
            
            function asyncAlert(content) {
                setTimeout(function(){
                           alert(content);
                           },1);
            }
            
            
            </script>
            </head>
    
    <body>
        <h1>这是按钮调用</h1>
        <input type="button" value="扫一扫" onclick="scanClick()" />
        <input type="button" value="获取定位" onclick="locationClick()" />
        <input type="button" value="修改背景色" onclick="colorClick()" />
        <input type="button" value="分享" onclick="shareClick()" />
        <input type="button" value="支付" onclick="payClick()" />
        <input type="button" value="摇一摇" onclick="shake()" />
        <input type="button" value="返回" onclick="goBack()" />
        <input type="button" value="播放声音" onclick="playSound()" />
        
        <h1>这是文件上传</h1>

        <input type="file" />
        
        <h1>这是回调结果展示区</h1>
        <textarea id ="returnValue" type="value" rows="5" cols="40">
        
        </textarea>
        
        <h4>竖直方向的表头：</h4>
        <table border="1" style="width:90%;height:600px">
            <tr>
                <th>姓名</th>
                <td>Bill Gates</td>
            </tr>
            <tr>
                <th>电话</th>
                <td>555 77 854</td>
            </tr>
            <tr>
                <th>传真</th>
                <td>555 77 855</td>
            </tr>
        </table>
        
        
    </body>
</html>

```

ios --- WKWebViewController
```
//
//  WKWebViewController.m
//  JS_OC_URL
//
//  Created by Harvey on 16/8/4.
//  Copyright © 2016年 Haley. All rights reserved.
//

#import <WebKit/WebKit.h>
#import <AVFoundation/AVFoundation.h>

#import "WKWebViewController.h"
#import "HLAudioPlayer.h"

@interface WKWebViewController ()<WKUIDelegate,WKScriptMessageHandler>

@property (strong, nonatomic)   WKWebView                   *webView;
@property (strong, nonatomic)   UIProgressView              *progressView;

@end

@implementation WKWebViewController

- (void)viewDidLoad {
    [super viewDidLoad];
    // Do any additional setup after loading the view.
    self.title = @"MessageHandler";
    
    UIBarButtonItem *rightItem = [[UIBarButtonItem alloc] initWithBarButtonSystemItem:UIBarButtonSystemItemCancel target:self action:@selector(rightClick)];
    self.navigationItem.rightBarButtonItem = rightItem;
    
    [self initWKWebView];
    
    [self initProgressView];
    
    [self.webView addObserver:self forKeyPath:@"estimatedProgress" options:NSKeyValueObservingOptionNew context:nil];
}

- (void)viewWillAppear:(BOOL)animated
{
    [super viewWillAppear:animated];
    // addScriptMessageHandler 很容易导致循环引用
    // 控制器 强引用了WKWebView,WKWebView copy(强引用了）configuration， configuration copy （强引用了）userContentController
    // userContentController 强引用了 self （控制器）
    [self.webView.configuration.userContentController addScriptMessageHandler:self name:@"ScanAction"];
    [self.webView.configuration.userContentController addScriptMessageHandler:self name:@"Location"];
    [self.webView.configuration.userContentController addScriptMessageHandler:self name:@"Share"];
    [self.webView.configuration.userContentController addScriptMessageHandler:self name:@"Color"];
    [self.webView.configuration.userContentController addScriptMessageHandler:self name:@"Pay"];
    [self.webView.configuration.userContentController addScriptMessageHandler:self name:@"Shake"];
    [self.webView.configuration.userContentController addScriptMessageHandler:self name:@"GoBack"];
    [self.webView.configuration.userContentController addScriptMessageHandler:self name:@"PlaySound"];
}

- (void)viewWillDisappear:(BOOL)animated
{
    [super viewWillDisappear:animated];
    
    // 因此这里要记得移除handlers
    [self.webView.configuration.userContentController removeScriptMessageHandlerForName:@"ScanAction"];
    [self.webView.configuration.userContentController removeScriptMessageHandlerForName:@"Location"];
    [self.webView.configuration.userContentController removeScriptMessageHandlerForName:@"Share"];
    [self.webView.configuration.userContentController removeScriptMessageHandlerForName:@"Color"];
    [self.webView.configuration.userContentController removeScriptMessageHandlerForName:@"Pay"];
    [self.webView.configuration.userContentController removeScriptMessageHandlerForName:@"Shake"];
    [self.webView.configuration.userContentController removeScriptMessageHandlerForName:@"GoBack"];
    [self.webView.configuration.userContentController removeScriptMessageHandlerForName:@"PlaySound"];
}

- (void)initWKWebView
{
    WKWebViewConfiguration *configuration = [[WKWebViewConfiguration alloc] init];
    
    WKPreferences *preferences = [WKPreferences new];
    preferences.javaScriptCanOpenWindowsAutomatically = YES;
    preferences.minimumFontSize = 40.0;
    configuration.preferences = preferences;
    
    self.webView = [[WKWebView alloc] initWithFrame:self.view.frame configuration:configuration];
    
    
//    NSString *urlStr = @"http://www.baidu.com";
//    NSURLRequest *request = [NSURLRequest requestWithURL:[NSURL URLWithString:urlStr]];
//    [self.webView loadRequest:request];
    
    NSString *urlStr = [[NSBundle mainBundle] pathForResource:@"index.html" ofType:nil];
    NSURL *fileURL = [NSURL fileURLWithPath:urlStr];
    [self.webView loadFileURL:fileURL allowingReadAccessToURL:fileURL];
    
    self.webView.UIDelegate = self;
    [self.view addSubview:self.webView];
}

- (void)initProgressView
{
    CGFloat kScreenWidth = [[UIScreen mainScreen] bounds].size.width;
    UIProgressView *progressView = [[UIProgressView alloc] initWithFrame:CGRectMake(0, 0, kScreenWidth, 2)];
    progressView.tintColor = [UIColor redColor];
    progressView.trackTintColor = [UIColor lightGrayColor];
    [self.view addSubview:progressView];
    self.progressView = progressView;
}

- (void)rightClick
{
    [self goBack];
}

- (void)dealloc
{
    NSLog(@"%s",__FUNCTION__);
    [self.webView removeObserver:self forKeyPath:@"estimatedProgress"];
}

#pragma mark - private method
- (void)getLocation
{
    // 获取位置信息
    
    // 将结果返回给js
    NSString *jsStr = [NSString stringWithFormat:@"setLocation('%@')",@"广东省深圳市南山区学府路XXXX号"];
    [self.webView evaluateJavaScript:jsStr completionHandler:^(id _Nullable result, NSError * _Nullable error) {
        NSLog(@"%@----%@",result, error);
    }];
   
}

- (void)shareWithParams:(NSDictionary *)tempDic
{
    if (![tempDic isKindOfClass:[NSDictionary class]]) {
        return;
    }
    
    NSString *title = [tempDic objectForKey:@"title"];
    NSString *content = [tempDic objectForKey:@"content"];
    NSString *url = [tempDic objectForKey:@"url"];
    // 在这里执行分享的操作
    
    // 将分享结果返回给js
    NSString *jsStr = [NSString stringWithFormat:@"shareResult('%@','%@','%@')",title,content,url];
    [self.webView evaluateJavaScript:jsStr completionHandler:^(id _Nullable result, NSError * _Nullable error) {
        NSLog(@"%@----%@",result, error);
    }];
}

- (void)changeBGColor:(NSArray *)params
{
    if (![params isKindOfClass:[NSArray class]]) {
        return;
    }
    
    if (params.count < 4) {
        return;
    }
    
    CGFloat r = [params[0] floatValue];
    CGFloat g = [params[1] floatValue];
    CGFloat b = [params[2] floatValue];
    CGFloat a = [params[3] floatValue];
    
    self.view.backgroundColor = [UIColor colorWithRed:r/255.0 green:g/255.0 blue:b/255.0 alpha:a];
}

- (void)payWithParams:(NSDictionary *)tempDic
{
    if (![tempDic isKindOfClass:[NSDictionary class]]) {
        return;
    }
    NSString *orderNo = [tempDic objectForKey:@"order_no"];
    long long amount = [[tempDic objectForKey:@"amount"] longLongValue];
    NSString *subject = [tempDic objectForKey:@"subject"];
    NSString *channel = [tempDic objectForKey:@"channel"];
    NSLog(@"%@---%lld---%@---%@",orderNo,amount,subject,channel);
    
    // 支付操作
    
    // 将支付结果返回给js
    NSString *jsStr = [NSString stringWithFormat:@"payResult('%@')",@"支付成功"];
    [self.webView evaluateJavaScript:jsStr completionHandler:^(id _Nullable result, NSError * _Nullable error) {
        NSLog(@"%@----%@",result, error);
    }];
}

- (void)shakeAction
{
    AudioServicesPlaySystemSound (kSystemSoundID_Vibrate);
    [HLAudioPlayer playMusic:@"shake_sound_male.wav"];
}

- (void)goBack
{
    [self.webView goBack];
}

- (void)playSound:(NSString *)fileName
{
    if (![fileName isKindOfClass:[NSString class]]) {
        return;
    }
    
    [HLAudioPlayer playMusic:fileName];
}

#pragma mark - KVO
// 计算wkWebView进度条
- (void)observeValueForKeyPath:(NSString *)keyPath ofObject:(id)object change:(NSDictionary *)change context:(void *)context {
    if (object == self.webView && [keyPath isEqualToString:@"estimatedProgress"]) {
        CGFloat newprogress = [[change objectForKey:NSKeyValueChangeNewKey] doubleValue];
        if (newprogress == 1) {
            [self.progressView setProgress:1.0 animated:YES];
            dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(0.7 * NSEC_PER_SEC)), dispatch_get_main_queue(), ^{
                self.progressView.hidden = YES;
                [self.progressView setProgress:0 animated:NO];
            });
            
        }else {
            self.progressView.hidden = NO;
            [self.progressView setProgress:newprogress animated:YES];
        }
    }
}

#pragma mark - WKUIDelegate
- (void)webView:(WKWebView *)webView runJavaScriptAlertPanelWithMessage:(NSString *)message initiatedByFrame:(WKFrameInfo *)frame completionHandler:(void (^)(void))completionHandler
{
    
    UIAlertController *alert = [UIAlertController alertControllerWithTitle:@"提醒" message:message preferredStyle:UIAlertControllerStyleAlert];
    [alert addAction:[UIAlertAction actionWithTitle:@"知道了" style:UIAlertActionStyleCancel handler:^(UIAlertAction * _Nonnull action) {
        completionHandler();
    }]];
    
    [self presentViewController:alert animated:YES completion:nil];
}

#pragma mark - WKScriptMessageHandler
- (void)userContentController:(WKUserContentController *)userContentController didReceiveScriptMessage:(WKScriptMessage *)message
{
//    message.body  --  Allowed types are NSNumber, NSString, NSDate, NSArray,NSDictionary, and NSNull.
    NSLog(@"body:%@",message.body);
if ([message.name isEqualToString:@"Location"]) {
        [self getLocation];
    }
}

@end

```

