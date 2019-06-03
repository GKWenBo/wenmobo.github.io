---
title: iOS 文件获取与选择
tags: [iCloud,iOS,GitHub]
date: 2018-11-06 22:26:54
permalink:
categories: iOS
description: 最近开发遇到一个需求，就是要从手机客户端通过云信IM发送文件，最开始想到的是通过云信发送文件是没有问题的，但似乎获取从手机获取文件没那么方便，安卓手机是可以轻松去选取系统文件，iOS相对没那么开放，但这也是iOS相比安卓安全的原因之一。除了系统开发的图片文件和视频文件，就不能获取其他格式文件吗？答案是否定的，自己留意了微信，发现有发送文件功能，微信点击发送文件，就会跳转到一个系统提供的文件拾取界面。自己从前也没有接触过，查阅了相关资料，自己也做下记录，下面开始介绍iOS获取文件的几种方式吧！
image:
---
<p class="description"></p>

<!-- more -->

### 一、需求分析

> 最近开发遇到一个需求，就是要从手机客户端通过云信IM发送文件，最开始想到的是通过云信发送文件是没有问题的，但似乎获取从手机获取文件没那么方便，安卓手机是可以轻松去选取系统文件，iOS相对没那么开放，但这也是iOS相比安卓安全的原因之一。除了系统开发的图片文件和视频文件，就不能获取其他格式文件吗？答案是否定的，自己留意了微信，发现有发送文件功能，微信点击发送文件，就会跳转到一个系统提供的文件拾取界面。自己从前也没有接触过，查阅了相关资料，自己也做下记录，下面开始介绍iOS获取文件的几种方式吧！

### 二、通过**WBUIDocumentPickerController**拾取

- iOS 新增的文件拾取类，API也比较的简单，通过简单的几句代码就可以实现文件拾取功能

```
UIDocumentPickerViewController *documentPickerViewController = [[UIDocumentPickerViewController alloc]initWithDocumentTypes:self.documentTypes
                                                                                                                         inMode:UIDocumentPickerModeOpen];
    documentPickerViewController.delegate = self;
    [self presentViewController:documentPickerViewController
                       animated:YES
                     completion:nil];
```

- 创建的时候需传入类型，更多UTI类型可到苹果官网查看：[https://developer.apple.com/library/archive/documentation/Miscellaneous/Reference/UTIRef/Articles/System-DeclaredUniformTypeIdentifiers.html#//apple_ref/doc/uid/TP40009259-SW1](https://developer.apple.com/library/archive/documentation/Miscellaneous/Reference/UTIRef/Articles/System-DeclaredUniformTypeIdentifiers.html#//apple_ref/doc/uid/TP40009259-SW1)

```
- (NSArray *)documentTypes {
    if (!_documentTypes) {
        _documentTypes = @[@"public.content",
                           @"public.text",
                           @"public.source-code",
                           @"public.image",
                           @"public.audiovisual-content",
                           @"com.adobe.pdf",
                           @"com.apple.keynote.key",
                           @"com.microsoft.word.doc",
                           @"com.microsoft.excel.xls",
                           @"com.microsoft.powerpoint.ppt",
                           @"public.rtf",
                           @"public.html",
                           @""];
    }
    return _documentTypes;
}
```

- 代理方法

  ```
  // MARK:UIDocumentPickerDelegate
  /*  < iOS 11 API > */
  - (void)documentPicker:(UIDocumentPickerViewController *)controller didPickDocumentsAtURLs:(NSArray<NSURL *> *)urls {
      NSLog(@"%s",__func__);
  }
  
  /*  < iOS 8 API > */
  - (void)documentPicker:(UIDocumentPickerViewController *)controller didPickDocumentAtURL:(NSURL *)url {
      NSLog(@"%s",__func__);
      
      NSArray *array = [[url absoluteString] componentsSeparatedByString:@"/"];
      NSString *fileName = [array lastObject];
      fileName = [fileName stringByRemovingPercentEncoding];
      
      if ([WBiCloudManager iCloudEnable]) {
          [WBiCloudManager wb_downloadWithDocumentURL:url
                                       completedBlock:^(id obj) {
                                           NSData *data = obj;
                                           NSString *path = [NSHomeDirectory() stringByAppendingString:[NSString stringWithFormat:@"/Documents/%@",fileName]];
                                           /*  < 写入沙盒 > */
                                           [data writeToFile:path
                                                  atomically:YES];
                                       }];
      }
  }
  
  - (void)documentPickerWasCancelled:(UIDocumentPickerViewController *)controller {
      NSLog(@"%s",__func__);
  }
  ```

  **注意**：我们从文件拾取器选择了一个文件，有可能这个文件还没有从iCloud同步到本地，这时我们需要子类化一个**UIDocument**对象，去做文件相关的操作，在使用前还需判断是否开启iCloud服务：

  ```
  //如果url不为空，说明可以使用iCloud相关功能api
  + (NSURL *)iCloudURLForIdentifier:(NSString *)identifier {
      NSURL *url = nil;
      NSFileManager *fileManager = [NSFileManager defaultManager];
      url = [fileManager URLForUbiquityContainerIdentifier:identifier];
      return url;
  }
  ```

  关于**WBUIDocumentPickerController**的使用，我写了一个详细的demo，在文章末尾我也为贴出GitHub地址。

  ### 三、UIDocumentBrowserViewController

  > Important
  >
  > Always assign the document browser as your app's root view controller. Don't place the document browser in a navigation controller, tab bar, or split view, and don't present the document browser modally.
  >
  > If you want to present a document browser from another location in your view hierarchy, use a [`UIDocumentPickerViewController`](https://developer.apple.com/documentation/uikit/uidocumentpickerviewcontroller?language=objc) instead.

  iOS 11新出的API，用来浏览本地系统文件，iCloud文件。一般的项目可能用不到这个类，使用的时候注意遵循官方文档说明，将UIDocumentPickerViewController设置为窗口根控制器。这里也不做过多说明了，自己也写了个小demo，感兴趣的朋友可下载demo查看。

  ### 四、通过三方APP拷贝文件到自己的项目

  通过这种方式获取文件，之前自己已经接触过了，比如像QQ，微信打开一个文件选择用自己的APP打开，自己APP先要在 **info.plist**中配置支持文件类型，然后就能将文件拷贝到自己的项目，主要用到的方法是**AppDelegate**中

  ```
  - (BOOL)application:(UIApplication *)app openURL:(NSURL *)url options:(NSDictionary<UIApplicationOpenURLOptionsKey,id> *)options {
      NSLog(@"url  %@",[url absoluteString]);
      
      /*  < 三方App拷贝到自己App处理 > */
      if ([[url absoluteString] containsString:@"file"]) {
          NSArray *array = [[url absoluteString] componentsSeparatedByString:@"/"];
          NSString *fileName = [array lastObject];
          fileName = [fileName stringByRemovingPercentEncoding];
          
          NSString *path = [NSHomeDirectory() stringByAppendingString:[NSString stringWithFormat:@"/Documents/Inbox/%@",fileName]];
          
          NSData *data = [NSData dataWithContentsOfFile:path];
      }
      return YES;
  }
  ```

### 五、结语

> 自己暂时了解到的文件获取方式就这些了，以后有新的方式再做补充吧，文章如果有描述不对的地方还请批评指正，如果你有更多更好的方式，欢迎一起交流学习。既然有发送文件，就会有预览文件，接下来我也会写一篇文件预览的文章。好了，最后贴出demo地址吧！

### 六、Demo

[WBDocumentBrowserDemo](https://github.com/wenmobo/WBDocumentBrowserDemo)

### 参考文章

1、[https://developer.apple.com/documentation/uikit/view_controllers/adding_a_document_browser_to_your_app?language=objc](https://developer.apple.com/documentation/uikit/view_controllers/adding_a_document_browser_to_your_app?language=objc)

2、[https://developer.apple.com/documentation/uikit/view_controllers/building_a_document_browser-based_app?language=objc](https://developer.apple.com/documentation/uikit/view_controllers/building_a_document_browser-based_app?language=objc)

3、[iOS - 从iCloud，QQ，微信获取文件](https://www.jianshu.com/p/1d07fab48e67)