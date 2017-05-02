---
layout: post
title: Xcode8代码格式化插件
categories: [Xcode插件]
tags: [工具]
date: 2016-04-05 19:51:03
comments: true
---

> Xcode 8前，开发者可以通过运行时注入代码来实现添加插件，甚至在著名的插件管理工具Alcatraz上提交和分发插件。
> 但是，Xcode 8成为这种开始方式的终结者，因为它提供了自家的Xcode Source Editor Extension方式开发插件。
> 接下来，将详细介绍如何在Xcode8下编写插件，以编写代码格式化插件为例。不足之处，欢迎补充，先看下最终效果吧。

### 最终效果
**final Effects**  
![final Effects]({{ site.url }}/img/blogImg/codeFormat_gif.gif)  

### 具体代码实现
1.打开Xcode创建mac程序 -> macOS -> Cocoa Application，接着设置工程参数即可
![ ]({{ site.url }}/img/blogImg/codeFormat_1.png)   
![ ]({{ site.url }}/img/blogImg/codeFormat_2.png) 

接下来添加Target
![ ]({{ site.url }}/img/blogImg/codeFormat_3.png)   
![ ]({{ site.url }}/img/blogImg/codeFormat_4.png) 

配置Target名称等信息，Finish完成，此时会弹出Scheme对话框，点击Activate启用。文件结构如下
![ ]({{ site.url }}/img/blogImg/codeFormat_5.png)   

**认识插件**
SourceEditorExtension类和插件的生命周期和配置有关，实现了XCSourceEditorExtension协议中的一个方法extensionDidFinishLaunching() 和一个变量commandDefinitions，默认是被注释掉的。
方法extensionDidFinishLaunching()是指刚刚加载好插件但还未点击插件按钮时，可以执行某些准备工作。
变量commandDefinitions返回字典类型的数组，可以为每个插件重写名字、标识符和自定义类名等信息，相当于设置后面要介绍的的Info.plist文件中对应的XCSourceEditorCommandName、XCSourceEditorCommandIdentifier和XCSourceEditorCommandClassName信息。
SourceEditorCommand类实现了XCSourceEditorCommand协议中的perform方法，点击插件按钮所执行的具体逻辑就是在这个方法中完成的，因此是实现插件功能的核心。
({{ site.url }}/img/blogImg/codeFormat_7.png)  

思路，获取需要格式化类文件的 当前代码数组，数组元素为每一行代码，对对象转化成字符串，对字符串进行操作，从而实现格式化代码。
具体代码如下，

SourceEditorCommand.m文件代码
```
#import "SourceEditorCommand.h"
#import "HandleFormatCodeMethod.h"

@interface SourceEditorCommand ()

@property (strong, nonatomic) NSMutableArray *allLinesCodeMArr;
@property (strong, nonatomic) HandleFormatCodeMethod *codeMethod;

@end
```

```
- (void)performCommandWithInvocation:(XCSourceEditorCommandInvocation *)invocation completionHandler:(void (^)(NSError * _Nullable nilOrError))completionHandler
{
//    [self handleLLVMStyleFormatWithInvocation:invocation completionHandler:completionHandler];
    
    [self formatAllClassFileCode:invocation];
    completionHandler(nil);
}


- (NSMutableArray *)allLinesCodeMArr
{
    if (!_allLinesCodeMArr) {
        _allLinesCodeMArr = [NSMutableArray array];
    }
    return _allLinesCodeMArr;
}


- (HandleFormatCodeMethod *)codeMethod
{
    if (!_codeMethod) {
        _codeMethod = [[HandleFormatCodeMethod alloc] init];
    }
    return _codeMethod;
}


- (void)formatAllClassFileCode:(XCSourceEditorCommandInvocation *)invocation
{
    if (invocation.buffer.lines.count == 0) {
        return;
    }
    
    [self.allLinesCodeMArr removeAllObjects];
    //处理NSTaggedPointerString 转化为NSString
    for (int i = 0; i < invocation.buffer.lines.count; i++) {
        id stringValue = [invocation.buffer.lines objectAtIndex:i];
        NSString *stringPointerValue = [NSString stringWithFormat:@"%@",stringValue];
        [_allLinesCodeMArr addObject:stringPointerValue];
    }
    
    self.codeMethod.allLinesCodeMArr = self.allLinesCodeMArr;
    self.codeMethod.invocation = invocation;
    
    //处理implementation之后所有的换行
    [self.codeMethod handlEunnecessaryNewLine];
    
    //处理#import之前要预留一行空格的问题
    [self.codeMethod handleBeforeFirstImportHasOnlyOneNewLine];
    
    //处理最后一个#import与第一个@interface之间仅保留一个换行
    [self.codeMethod handleBetweenTheLastImportAndNextCodeHasOnlyOneNewLine];

    //处理@interface前后都有一个换行
    [self.codeMethod handleBefroreInterfaceHasOnlyOneNewLine];
    [self.codeMethod handleLaterInterfaceHasOnlyOneNewLine];

    //处理implementation前后都有一个换行
    [self.codeMethod handleBefroreImplementationHasOnlyOneNewLine];
    [self.codeMethod handleLaterImplementationHasOnlyOneNewLine];
    
    //处理@end前后都有一个换行
    [self.codeMethod handleBefroreEndHasOnlyOneNewLine];
    [self.codeMethod handleLaterEndHasOnlyOneNewLine];
    
    //处理@property属性申明时候的间距
    [self.codeMethod handlePropertyLineSpaceMargin];
    
    //处理每个方法开始的第一个花括号另起一行
    [self.codeMethod handleMethodStartLocationTheFirstBraceNewLine];
    
    //处理每个方法前有2个换行 每个方法后有2个换行
    [self.codeMethod handleBeforeEveryMethodAndLaterEveryMethodHasOnlyTwoNewLine];
    
    //处理#pragma 或者 #warning前后各空一行
    [self.codeMethod handleBeforeAndLaterPragmaOrWarningHasNewLine];
    
}
```

```
- (void)handleLLVMStyleFormatWithInvocation:(XCSourceEditorCommandInvocation *)invocation completionHandler:(void (^)(NSError * _Nullable nilOrError))completionHandler
{
    NSString *pathStr = [[NSBundle mainBundle] pathForResource:@"clang_format" ofType:nil];
    
    NSFileManager *fileManager = [NSFileManager defaultManager];
    if ([fileManager fileExistsAtPath:pathStr]) {
        NSLog(@"=====fileExistsAtPath======");
    }
    
    
    NSPipe *errorPipe = [[NSPipe alloc] init];
    NSPipe *outputPipe = [[NSPipe alloc] init];
    NSPipe *inputPipe = [[NSPipe alloc] init];

    NSTask *task = [[NSTask alloc] init];
    task.standardError = errorPipe;
    task.standardOutput = outputPipe;
    task.launchPath = pathStr;
    task.arguments = @[@"-style=llvm"];
    task.standardInput = inputPipe;
    
    NSFileHandle *stdinHandle = inputPipe.fileHandleForWriting;
    
    NSString *stdin = invocation.buffer.completeBuffer;
    
    NSData *data = [stdin dataUsingEncoding:NSUTF8StringEncoding];
    [stdinHandle writeData:data];
    [stdinHandle closeFile];
    [task launch];
    [task waitUntilExit];
    [errorPipe.fileHandleForReading readDataToEndOfFile];
    NSData *outputData = [outputPipe.fileHandleForReading readDataToEndOfFile];
    NSString *resultStr = [[NSString alloc] initWithData:outputData encoding:NSUTF8StringEncoding];
    
    
    //--------------------------------------------
    if ([invocation.buffer.contentUTI isEqualToString:@"public.objective-c-source"]) {
        [invocation.buffer.lines removeAllObjects];
        NSArray *lines = [resultStr componentsSeparatedByString:@"\n"];
        [invocation.buffer.lines addObjectsFromArray:lines];
        [invocation.buffer.selections removeAllObjects];
        XCSourceTextRange *range = [[XCSourceTextRange alloc] init];
        range.start = XCSourceTextPositionMake(0, 0);
        range.end = XCSourceTextPositionMake(0, 0);
        [invocation.buffer.selections addObject:range];
    };
}
```

HandleFormatCodeMethod.h中写具体的处理方法，再次只展示.h文件
```

#import <Foundation/Foundation.h>
#import <XcodeKit/XcodeKit.h> 

@interface HandleFormatCodeMethod : NSObject

@property (strong, nonatomic) NSMutableArray *allLinesCodeMArr;
@property (strong, nonatomic) XCSourceEditorCommandInvocation *invocation;




/**
 处理implementation之后所有的换行
 */
- (void)handlEunnecessaryNewLine;

/**
 处理成第一个#import之前必须有唯一的一个换行
 */
- (void)handleBeforeFirstImportHasOnlyOneNewLine;


/**
 处理最后一个#import与第一个interface之间仅保留一个换行
 */
- (void)handleBetweenTheLastImportAndNextCodeHasOnlyOneNewLine;


/**
 处理interface前只有一个换行
 */
- (void)handleBefroreInterfaceHasOnlyOneNewLine;


/**
 处理interface后只有一个换行
 */
- (void)handleLaterInterfaceHasOnlyOneNewLine;


/**
 处理implementation前只有一个换行
 */
- (void)handleBefroreImplementationHasOnlyOneNewLine;


/**
 处理implementation后只有一个换行
 */
- (void)handleLaterImplementationHasOnlyOneNewLine;


/**
 处理end前只有一个换行
 */
- (void)handleBefroreEndHasOnlyOneNewLine;


/**
 处理end后只有一个换行
 */
- (void)handleLaterEndHasOnlyOneNewLine;


/**
 处理每个方法前仅有2个换行  处理每个方法后仅有2个换行
 */
- (void)handleBeforeEveryMethodAndLaterEveryMethodHasOnlyTwoNewLine;


/**
 处理每个方法开始的第一个花括号另起一行
 */
- (void)handleMethodStartLocationTheFirstBraceNewLine;


/**
 处理@property属性申明时候的间距
 */
- (void)handlePropertyLineSpaceMargin;



/**
 处理#pragma 或者 #warning前后各空一行
 */
- (void)handleBeforeAndLaterPragmaOrWarningHasNewLine;


@end
```

//完整项目代码:https://github.com/maxzhang123/codeFormat
//转载请注明出处，谢谢


