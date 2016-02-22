---
layout: post
title: "Objectivec-C Tips 1"
date: 2016-02-22 10:59:43 +0800
comments: true
categories: Objectivec-C,tip
published: true
---
开个坑吧，这一篇专门记一下开发中遇到的小问题，以及解决方案，作为一个开发备忘录。

<!--more-->

## Tip1.UITextField中文键盘输入监听

问题：监听UITextField的输入我们一般使用*-(BOOL)textField:(UITextField *)textField shouldChangeCharactersInRange:(NSRange)range replacementString:(NSString*)string*来监听Field的输入，但如果使用中文键盘，会有问题：只能监听用户在拼汉字时输入的字母，而用户在选汉字时则不会进入回调，而我们恰恰需要监听用户最终输入的汉字。

解决方案：注册对@“UITextFieldTextDidChangeNotification”事件的监听，该通知会在用户选汉字时发出，我们可以利用这一通知来做对Field的中文长度和内容的判断，注意要在监听者的dealloc方法中移除监听。

```objectivec
//1.注册UITextFieldTextDidChangeNotification监听
[[NSNotificationCenter defaultCenter]addObserver:self selector:@selector(textFiledEditChanged:)
                                            name:@"UITextFieldTextDidChangeNotification" 
                                          object:myTextField];
```

```objectivec
//2.监听处理
#define kMaxLength 25
-(void)textFiledEditChanged:(NSNotification *)obj{
   UITextField *textField = (UITextField *)obj.object;
   
   NSString *toBeString = textField.text;
   NSString *lang = [[textField textInputMode] primaryLanguage]; // 键盘输入模式
   if ([lang isEqualToString:@"zh-Hans"]) { // 简体中文输入，包括简体拼音，健体五笔，简体手写
       UITextRange *selectedRange = [textField markedTextRange];
       // 获取高亮部分
       UITextPosition *position = [textField positionFromPosition:selectedRange.start offset:0];
       // 没有高亮选择的字，则对已输入的文字进行字数统计和限制
       if (!position) {
           if (toBeString.length > kMaxLength) {
               textField.text = [toBeString substringToIndex:kMaxLength];
           }
       }
       // 有高亮选择的字符串，则暂不对文字进行统计和限制
       else {
       }
   }
   // 中文输入法以外的处理
   else {
   }
}
```

```objectivec
-(void)dealloc{
   [[NSNotificationCenter defaultCenter]removeObserver:self
                                                  name:@"UITextFieldTextDidChangeNotification"
                                                object:_albumNameTextField];
}
```