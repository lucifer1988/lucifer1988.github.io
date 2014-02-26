---
layout: post
title: "强力的日志分析工具-NSLogger"
date: 2014-02-25 17:00:29 +0800
comments: true
categories: iOS
description: "强力的日志分析工具-NSLogger"
keywords: NSLogger,日志分析
published: true
---
>NSLogger出现了，在Florent Pillet的打造下，一个开源强力的输出工具给了log这一古老的工作崭新的生命。标签输出，优先级查找，直接输出图像，多线程标记，时序控制，甚至是通过网络log到别人的终端或者是从别人的终端程序中记录log。在这里，只有想不到没有做不到，堪称是史上最为强大的logger。  
><p align="right">--OneV's Den</p>

NSLogger是一款强力的日志记录和分析工具，其强大的功能，可以完全替代Xcode自带的Debugger，本文将介绍一些其主要特点和用法。git地址：<https://github.com/fpillet/NSLogger>

<!--more-->

##安装

NSLogger由两部分组成，一是需要加入工程中的组件代码，二是查看和管理日志的Mac客户端。组件代码可通过CocoaPods安装，也可通过手动添加（手动添加需要引入CFNetwok.framework和SystemConfiguration.framework）。Mac客户端NSLoggerViewer的源码包含在了组件当中，可以自己生成，也可以[点击这里](http://doruby.com/assets/NSLoggerViewer.zip)下载生成好的客户端，NSLoggerViewer实际运行图：

<img src="/images/NSLoggerViewer.png">

<!--more-->

##特点

* 标签输出
* 自定义优先级
* 直接输出图片
* 多线程标记
* 时序控制
* 远程记录

<!--more-->

##配置
首先要将LoggerClient.h头文件import进来，通过LoggerSetOptions()配置Logger的一些属性：

```
enum {
  kLoggerOption_LogToConsole               = 0x01,
  kLoggerOption_BufferLogsUntilConnection  = 0x02,
  kLoggerOption_BrowseBonjour              = 0x04,
  kLoggerOption_BrowseOnlyLocalDomain      = 0x08,
  kLoggerOption_UseSSL                     = 0x10,
  kLoggerOption_CaptureSystemConsole       = 0x20
};

void LoggerSetOptions(Logger *logger, uint32_t flags);
```
一般使用默认Logger，第一个参数传入NULL就行，至于第二个参数主要是一些功能开关选项，将需要开启的功能相或后作为第二个参数即可，详细参数说明[点击](https://github.com/fpillet/NSLogger/wiki/NSLogger-API)，实例：

```
		LoggerSetOptions(NULL,					
						 kLoggerOption_BufferLogsUntilConnection |
						 kLoggerOption_UseSSL |
						 kLoggerOption_CaptureSystemConsole|
						 kLoggerOption_BrowseBonjour|
						 kLoggerOption_BrowseOnlyLocalDomain : 0));
```
NSLogger支持TCP和Bonjour两种方式连接终端设备，Bonjour连接一般不需要配置，如果要是使用TCP连接，要通过LoggerSetViewerHost()配置IP地址和端口（同时需配置NSLoggerViewer，在Preferences的Network中，勾选 “Listen for loggers on TCP port”打开监听）：

```
void LoggerSetViewerHost(Logger *aLogger, CFStringRef host, UInt32 port);
```
同样，使用默认Logger，第一参数传NULL，实例：

```
LoggerSetViewerHost(NULL, (__bridge CFStringRef)@"192.168.11.38", (UInt32)50000);
```
以上代码放到```- (void)applicationDidFinishLaunching:(UIApplication *)application```统一配置。

<!--more-->

##使用

使用NSLogger基本方法和NSLog并无本质差别，只是开发者可以添加一些标签和级别参数，以供NSLoggerViewer端的日志过滤。

```
void LogMessage(NSString *tag, int level, NSString *format, ...);
```
同时也支持添加文件名、方法名、行号、变量名等参数：

```
void LogMessageF(const char *file, int line, const char *function, NSString *tag, int level, NSString *format, ...);
void LogMessage_va(NSString *tag, int level, NSString *format, va_list args);
```
NSLogger支持直接打印二进制数据：

```
void LogData(NSString *tag, int level, NSData *data);
void LogDataF(const char *file, int line, const char *function, NSString *tag, int level, NSData *data);
```
NSLogger最大的优点，支持直接打印图片，而且可以指定打印图片的大小：

```
void LogImageData(NSString *tag, int level, int width, int height, NSData *data);
void LogImageDataF(const char *file, int line, const char *function, NSString *tag, int level, int width, int height, NSData *data);
```

<!--more-->

##Tips

* 直接使用默认打印函数过于繁琐，可结合需求自己定义宏来定义打印方法：

```
#ifdef DEBUG
#define LOG_NETWORK(level, ...) LogMessageF(FILE,LINE,FUNCTION,"network",level,__VA_ARGS__) 
#define LOG_GENERAL(level, ...) LogMessageF(__FILE__,__LINE__,__FUNCTION__,“general”,level,VA_ARGS)
#define LOG_GRAPHICS(level, ...) LogMessageF(FILE,LINE,FUNCTION,@"graphics",level,VA_ARGS)
#define LOG_TRACE(...) LogMessageF( __FILE__,__LINE__,__FUNCTION__, NULL, 0, __VA_ARGS__)
#else
#define LOG_NETWORK(...) do{}while(0)
#define LOG_GENERAL(...) do{}while(0)
#define LOG_GRAPHICS(...) do{}while(0)
#define LOG_TRACE(...) do{}while(0)
```
* 如果程序启动后，没有数据发送到NSLoggerViewer，可以先clean一下。
* 通过NSLoggerViewer当中Tools功能下```Add Mark```(**Cmd-M**)可以在日志列表底部快速添加一个时间戳标记，而使用```Add Mark With Title```(**shift-Cmd-M**)可以添加自定义标题的标记，通过这些标记将日志按照需要进行分块。  
<img src="/images/NSLoggerViewer_Marker.png">

##总结
本文介绍了NSLogger的一些基本用法和技巧，以后还会介绍一些其他的调试工具，不过个人感觉工具再好，不能真正结合自己的项目用起来，也没有太大意义，所以还是在平时能多去试试这些工具，这样才能利用到它们为我们真正做一些事情。



                                                  
