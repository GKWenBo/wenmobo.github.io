---
title: iOS优秀三方开源库整理，了解一下
tags: [iOS,GitHub,Objective-C,Swift]
date: 2018-07-08 11:00:36
permalink:
categories:
- iOS
description: 在自己工作之余，收集整理了一些优秀的三方开源框架，自己整理的这些三方开源库涵盖了iOS开发面很多方面的知识。非常感谢这些开源库的作者们，正是因为这些库，提高了我们的开发效率，同样也是我们学习进步的源泉。现将这个整理工程文件分享出来，希望能给需要的朋友一些帮助，同时也自己也做下收集记录。
image: https://ws3.sinaimg.cn/large/006tNc79ly1ft2a1l4r4cj31ic0a2gn5.jpg
---
<p class="description"></p>

<!-- more -->

### 一、前言
> 在自己工作之余，收集整理了一些优秀的三方开源框架，自己整理的这些三方开源库涵盖了iOS开发面很多方面的知识。非常感谢这些开源库的作者们，正是因为这些库，提高了我们的开发效率，同样也是我们学习进步的源泉。现将这个整理工程文件分享出来，希望能给需要的朋友一些帮助，同时也自己也做下收集记录。

**Github整理的地址**：    
[WBCollectOCThirdLib](https://github.com/wenmobo/WBCollectOCThirdLib)     
[WBCollectSwfitThirdLib](https://github.com/wenmobo/WBCollectSwfitThirdLib)

### 二、Objective-C三方开源库
- 表格侧滑菜单  
        [MGSwipeTableCell](https://github.com/MortimerGoro/MGSwipeTableCell) - **6300 star**  
        [SWTableViewCell](https://github.com/CEWendel/SWTableViewCell) - **7088 star**    
        [ZJSwipeTableView](https://github.com/jasnig/ZJSwipeTableView) - **7 star**

- 表格高度缓存库    

    - [FDTemplateLayoutCell](https://github.com/forkingdog/UITableView-FDTemplateLayoutCell) - **8992 star**   

      Template auto layout cell for automatically UITableViewCell height calculating

    ​    [HYBMasonryAutoCellHeight](https://github.com/CoderJackyHuang/HYBMasonryAutoCellHeight) - **595 star**

- 表格刷新控件  
        [MJRefresh](https://github.com/CoderMJLee/MJRefresh) - **12006 star**  
        [KafkaRefresh](https://github.com/OpenFeyn/KafkaRefresh) - **627 star**

- 弹出菜单（类似微信弹出菜单）  
        [FTPopOverMenu](https://github.com/liufengting/FTPopOverMenu) - **712 star**    
        [kxmenu](https://github.com/kolyvan/kxmenu) - **1776 star**  
        [PopMenuTableView](https://github.com/KongPro/PopMenuTableView) - **217 star**  

- 导航栏    
        [FDFullscreenPopGesture](https://github.com/forkingdog/FDFullscreenPopGesture) - **5010 star**  
        [KMNavigationBarTransition](https://github.com/MoZhouqi/KMNavigationBarTransition) - **2549 star**  
        [RTRootNavigationController](https://github.com/rickytan/RTRootNavigationController) - **1333 star**    
        [WRNavigationBar](https://github.com/wangrui460/WRNavigationBar) - **1852 star**

- 动画  
        [lottie-ios](https://github.com/airbnb/lottie-ios) - **14103 star**  
        [pop](https://github.com/facebook/pop) - **19064 star**     
        [LSAnimator](https://github.com/Lision/LSAnimator) **1238 star**

- 分段控件  
        [HMSegmentedControl](https://github.com/HeshamMegid/HMSegmentedControl) - **3392 star**  

- 富文本编辑    
        [ZSSRichTextEditor](https://github.com/nnhubbard/ZSSRichTextEditor) - **2891 star**

- 弹幕  
        [HJDanmakuDemo](https://github.com/panghaijiao/HJDanmakuDemo) - **717 star**    

- 滚动视图  
        [SwipeView](https://github.com/nicklockwood/SwipeView) - **2611 star**

- 滚动视图嵌套  
        [HJTabViewController](https://github.com/panghaijiao/HJTabViewController) - **191 star**    
        [LTScrollView](https://github.com/gltwy/LTScrollView) - **236 star**

- 红点提示  
        [JSBadgeView](https://github.com/JaviSoto/JSBadgeView) - **1209 star**   
        [WZLBadge](https://github.com/weng1250/WZLBadge) - **1603 star**

- 键盘  

    - [IQKeyboardManager](https://github.com/hackiftekhar/IQKeyboardManager) - **11697 star**  

      Codeless drop-in universal library allows to prevent issues of keyboard sliding up and cover UITextField/UITextView. Neither need to write any code nor any setup required and much more.一款强大键盘管理库。

    ​    [MMNumberKeyboard](https://github.com/matmartinez/MMNumberKeyboard) - **911 star**  
        [TPKeyboardAvoiding](https://github.com/michaeltyson/TPKeyboardAvoiding) - **5568 star**    

- 界面布局 

    - [FlexLib](https://github.com/zhenglibao/FlexLib) - **496 star**  

    - [Masonry](https://github.com/SnapKit/Masonry) - **16526 star**  

      Harness the power of AutoLayout NSLayoutConstraints with a simplified, chainable and expressive syntax. Supports iOS and OSX Auto Layout.对苹果原生AutoLayout的封装，链式语法，纯代码开发必备布局库。

    - [SDAutoLayout](https://github.com/gsdios/SDAutoLayout) - **5241 star**   

      One line of code to implement automatic layout. 一行代码搞定自动布局！支持Cell和Tableview高度自适应，Label和ScrollView内容自适应，致力于做最简单易用的AutoLayout库。The most easy way for autoLayout. Based on runtime.

    - [WHC_AutoLayoutKit](https://github.com/netyouli/WHC_AutoLayoutKit) - **786 star**

    - [MyLinearLayout](https://github.com/youngsoft/MyLinearLayout) - **2990 star** 

       MyLayout是一套iOS界面视图布局框架。MyLayout的内核是基于对UIView的layoutSubviews方法的重载以及对子视图的bounds和center属性的设置而实现的。MyLayout功能强大而且简单易用，它集成了:iOS Autolayout和SizeClass、android的5大布局体系、HTML/CSS的浮动定位技术以及flex-box和bootstrap框架等市面上主流的平台的界面布局功能，同时提供了一套非常简单和完备的多屏幕尺寸适配的解决方案。之前自己布局一直用Frame、Masonry，Xib布局，最近也在学习这款强大的布局框架。

- 进度指示器    
        [DACircularProgress](https://github.com/danielamitay/DACircularProgress) - **2307 star**    
        [SDProgressView](https://github.com/gsdios/SDProgressView) - **378 star**

- 开发模式  
        [KVOController](https://github.com/facebook/KVOController) - **6524 star**   

- 控制器切换    
        [DWQListOfDifferentOrderStatus](https://github.com/DevelopmentEngineer-DWQ/DWQListOfDifferentOrderStatus) **12 star**   
        [HYPageView](https://github.com/runlhy/HYPageView) - **74 star**  
        [SGPagingView](https://github.com/kingsic/SGPagingView) - **822 star**    
        [WMPageController](https://github.com/wangmchn/WMPageController) - **2229 star**    
        [ZJScrollPageView](https://github.com/jasnig/ZJScrollPageView) - **847 star**

- 数据存储  
        [fmdb](https://github.com/ccgus/fmdb) - **12533 star**   
        [BGFMDB](https://github.com/huangzhibiao/BGFMDB) - **771 star**  
        [JKDBModel](https://github.com/Haley-Wong/JKDBModel) - **683 star**  
        [JRDB](https://github.com/scubers/JRDB) - **480 star**  
        [LKDBHelper-SQLite-ORM](https://github.com/li6185377/LKDBHelper-SQLite-ORM) - **980 star**

- 数据转模型  
        [MJExtension](https://github.com/CoderMJLee/MJExtension) - **7667 star**   
        [YYModel](https://github.com/ibireme/YYModel) - **3589 star**  
        [Mantle](https://github.com/Mantle/Mantle)  - **11023 star**    
        [jsonmodel](https://github.com/jsonmodel/jsonmodel) - **6559 star**  
        [GDataXML-HTML](https://github.com/graetzer/GDataXML-HTML) - **261 star**

- 搜索  
        [PYSearch](https://github.com/ko1o/PYSearch) - **2976 star**

- 提示框架  
        [MBProgressHUD](https://github.com/jdg/MBProgressHUD/tree/master) - **14618 star**  
        [SVProgressHUD](https://github.com/SVProgressHUD/SVProgressHUD) - **11046 star**    
        [JGProgressHUD](https://github.com/JonasGessner/JGProgressHUD) - **2257 star**  
        [KSToastView](https://github.com/c0ming/KSToastView) - **91 star**  
        [MMPopupView](https://github.com/adad184/MMPopupView) - **1910 star**   
        [SCLAlertView](https://github.com/dogo/SCLAlertView) - **3084 star**   
        [Toast](https://github.com/scalessec/Toast) - **3085 star**

- 图表绘制  
        [AAChartKit](https://github.com/AAChartModel/AAChartKit) - **2241 star**   
        [JHChart](https://github.com/China131/JHChart) - **508 star**  
        [ZFChart](https://github.com/Zirkfied/ZFChart) - **670 star**  
        [DVPieChart](https://github.com/FireMou/DVPieChart) - **63 star**  
        [DVLineChart](https://github.com/FireMou/DVLineChart) - **56 star**

- 图片缓存框架  
        [SDWebImage](https://github.com/rs/SDWebImage) - **22089 star**  
        [FLAnimatedImage](https://github.com/Flipboard/FLAnimatedImage) - **6691 star**     
        [iOS-WebP](https://github.com/seanooi/iOS-WebP) - **739 star**  
        [YYWebImage](https://github.com/ibireme/YYWebImage) - **3195 star**  

- 图片浏览  
        [MWPhotoBrowser](https://github.com/mwaterfall/MWPhotoBrowser) - **8238 star**  
        [PYPhotoBrowser](https://github.com/ko1o/PYPhotoBrowser) - **1782 star**   
        [SDPhotoBrowser](https://github.com/gsdios/SDPhotoBrowser) - **962 star**    
        [STPhotoBrowser](https://github.com/STShenZhaoliang/STPhotoBrowser) - **299 star**  
        [KSPhotoBrowser](https://github.com/skx926/KSPhotoBrowser) - **457 star**

- 图片轮播  
        [SDCycleScrollView](https://github.com/gsdios/SDCycleScrollView) - **4921 star**    
        [HYBLoopScrollView](https://github.com/CoderJackyHuang/HYBLoopScrollView) - **615 star**    
        [TXScrollLabelView](https://github.com/tingxins/TXScrollLabelView) - **497 star**

- 图片拾取  
        [TZImagePickerController](https://github.com/banchichen/TZImagePickerController) - **5065 star**    
        [CTAssetsPickerController](https://github.com/chiunam/CTAssetsPickerController) - **2142 star**     
        [DNImagePicker](https://github.com/AwesomeDennis/DNImagePicker) - **365 star**  
        [HXWeiboPhotoPicker](https://github.com/KeenTeam1990/HXWeiboPhotoPicker) - **7 star**

- 3D效果图  
        [HelloPanoramaGL](https://github.com/heroims/HelloPanoramaGL) - **39 star**

- 网络请求  
        [AFNetworking](https://github.com/AFNetworking/AFNetworking) - **31246 star**    
        [YTKNetwork](https://github.com/yuantiku/YTKNetwork) - **5385 star**   
        [PPNetworkHelper](https://github.com/jkpang/PPNetworkHelper) - **1180 star**     
        [HYBNetworking](https://github.com/CoderJackyHuang/HYBNetworking) - **541 star**    
        [SJNetwork](https://github.com/knightsj/SJNetwork) - **153 star**

- 网络状态监测  
        [Reachability](https://github.com/tonymillion/Reachability) - **6665 star**

- 文件下载  
        [TWRDownloadManager](https://github.com/chasseurmic/TWRDownloadManager) - **366 star**  
        [ZFDownload](https://github.com/renzifeng/ZFDownload) - **291 star**

- 旋转木马  
        [iCarousel](https://github.com/nicklockwood/iCarousel) - **10628 star**  
        [NewPagedFlowView](https://github.com/PageGuo/NewPagedFlowView) - **512 star**

- 音视频    
        [ijkplayer](https://github.com/Bilibili/ijkplayer) - **19602 star**  
        [ZFPlayer](https://github.com/renzifeng/ZFPlayer) - **4539 star**  
        [WMPlayer](https://github.com/zhengwenming/WMPlayer) - **2397 star**   
        [TBPlayer](https://github.com/suifengqjn/TBPlayer) - **1125 star**  
        [TTAVPlayer](https://github.com/tangdiforx/TTAVPlayer) - **118 star**

- 占位图        
        [DZNEmptyDataSet](https://github.com/dzenbot/DZNEmptyDataSet) - **10439 star**  
        [LYEmptyView](https://github.com/dev-liyang/LYEmptyView) - **657 star**

- C语言扩展库   
        [libextobjc](https://github.com/jspahrsummers/libextobjc) - **3936 star**   

- Socket编程    
        [CocoaAsyncSocket](https://github.com/robbiehanson/CocoaAsyncSocket) - **10193 star**   
        [SocketRocket](https://github.com/facebook/SocketRocket) - **7833 star**   
        [socket.io](https://github.com/socketio/socket.io) - **42419 star**  
        [MQTTKit](https://github.com/mobile-web-messaging/MQTTKit) - **407 star**

- 内存泄露检测工具  

    - [MLeaksFinder](https://github.com/Tencent/MLeaksFinder) - **3568 star**   

      腾讯开源内存泄漏检测框架，非常好用，值得推荐。

    - [FBRetainCycleDetector](https://github.com/facebook/FBRetainCycleDetector) - **3068 star**

      iOS library to help detecting retain cycles in runtime.

- YYKit     
        [YYKit](https://github.com/ibireme/YYKit) - **12185 star**

- LOG工具   
        [CocoaLumberjack](https://github.com/CocoaLumberjack/CocoaLumberjack) - **10382 star**

- OC与JS交互    
        [WebViewJavascriptBridge](https://github.com/marcuswestin/WebViewJavascriptBridge) - **11017 star**

- 输入框占位符  
        [UITextView-Placeholder](https://github.com/devxoul/UITextView-Placeholder) - **797 star**  
        [RPFloatingPlaceholders](https://github.com/iwasrobbed/RPFloatingPlaceholders) - **1115 star**  
        [SZTextView](https://github.com/glaszig/SZTextView) - **652 star**  

- 分类  
        [JKCategories](https://github.com/shaojiankui/JKCategories) - **2770 star**

- 图像处理  
        [GPUImage](https://github.com/BradLarson/GPUImage) - **17606 star**

- iOS开发知识集合   

    - [iOS-Tips](https://github.com/awesome-tips/iOS-Tips) - **2187 star**   

       iOS知识小集，`iOS知识小集`的初衷是希望用300字左右（外加代码和效果展示）来说明一个小知识点，这样读者可以在上下班路上，花个2分钟就能了解一个iOS开发的小知识。

- 面试题集锦    

    - [iOSInterviewQuestions](https://github.com/ChenYilong/iOSInterviewQuestions) - **6809 star**  

      iOS面试题集锦（附答案），分为两篇[《招聘一个靠谱的 iOS》—参考答案（上）](https://github.com/ChenYilong/iOSInterviewQuestions/blob/master/01%E3%80%8A%E6%8B%9B%E8%81%98%E4%B8%80%E4%B8%AA%E9%9D%A0%E8%B0%B1%E7%9A%84iOS%E3%80%8B%E9%9D%A2%E8%AF%95%E9%A2%98%E5%8F%82%E8%80%83%E7%AD%94%E6%A1%88/%E3%80%8A%E6%8B%9B%E8%81%98%E4%B8%80%E4%B8%AA%E9%9D%A0%E8%B0%B1%E7%9A%84iOS%E3%80%8B%E9%9D%A2%E8%AF%95%E9%A2%98%E5%8F%82%E8%80%83%E7%AD%94%E6%A1%88%EF%BC%88%E4%B8%8A%EF%BC%89.md)   、[《招聘一个靠谱的 iOS》—参考答案（下）](https://github.com/ChenYilong/iOSInterviewQuestions/blob/master/01%E3%80%8A%E6%8B%9B%E8%81%98%E4%B8%80%E4%B8%AA%E9%9D%A0%E8%B0%B1%E7%9A%84iOS%E3%80%8B%E9%9D%A2%E8%AF%95%E9%A2%98%E5%8F%82%E8%80%83%E7%AD%94%E6%A1%88/%E3%80%8A%E6%8B%9B%E8%81%98%E4%B8%80%E4%B8%AA%E9%9D%A0%E8%B0%B1%E7%9A%84iOS%E3%80%8B%E9%9D%A2%E8%AF%95%E9%A2%98%E5%8F%82%E8%80%83%E7%AD%94%E6%A1%88%EF%BC%88%E4%B8%8B%EF%BC%89.md)，面试前值得学习与了解。

    - [iOS-InterviewQuestion-collection](https://github.com/liberalisman/iOS-InterviewQuestion-collection) - **1019 star** 

      iOS 开发者在面试过程中，常见的一些面试题，建议尽量弄懂了原理，并且多实践。

- 三方开源库分析    

    - [analyze](https://github.com/Draveness/analyze) - **6077 star**

      主要记录了Draveness大神阅读开源框架源代码的心得，主要框架包括**SDWebImage**、**MBProgressHUD**、**Masonry**、**AFNetworking**、**KVOController**等，有兴趣的朋友可以到GitHub阅读。

### 三、Swift三方开源库
- 动画  

    - [NVActivityIndicatorView](https://github.com/ninjaprox/NVActivityIndicatorView) - **7098 star**

      一组极棒的加载动画集合。

- 网络请求  

    - [Alamofire](https://github.com/Alamofire/Alamofire) - **28292 star**

      Swift优雅的HTTP网络请求库。

- 占位图    
        [SkeletonView](https://github.com/Juanpe/SkeletonView) - **5083 star**

- 二维码扫描    
        [EFQRCode](https://github.com/EyreFree/EFQRCode) - **2751 star**

- 布局框架  
        [SnapKit](https://github.com/SnapKit/SnapKit) - **12978 star**  

- 图表绘制  
        [Charts](https://github.com/danielgindi/Charts) - **18605 star**
### 四、结语
> 上面这些三方开源库按照自己的理解分类整理了一遍，其中OC语言库居多，自己现在也是基于OC开发，Swift收集的相对较少，这些库也是自己现在所了解到的，当然还有很多优秀的三方库自己也未发现和接触，我以后会不断在这篇博客中更新优秀的三方开源库。