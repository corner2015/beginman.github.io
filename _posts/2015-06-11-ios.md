---
layout: post
title: "iOS 图文混排"
description: "iOS 图文混排 M80AttributedLabel的一些修改"
category: "iOS"
tags: [iOS]
---
{% include JB/setup %}

<ul>
    <li>作者：<a href="http://weibo.com/xpmozong" target="blank">寞踪</a></li>
    <li>本文地址：http://xpmozong.github.io/ios/2014/12/21/ios-tuwenhunpai/</li>
    <li>转载请注明出处</li>
</ul>

1、M80AttributedLabel的URL、号码、邮箱的正则匹配不是很好，我做了一些修改。

    #pragma mark - 正则匹配电话号码，网址链接，Email地址
    + (NSArray *)addRexArr:(NSString *)text
    {
        NSString *parten = @"\\d{3}-\\d{8}|\\d{3}-\\d{7}|\\d{4}-\\d{8}|\\d{4}-\\d{7}|1+[358]+\\d{9}|\\d{8}|\\d{7}";
        NSArray *resultArray = [text componentsMatchedByRegex:parten];
        NSMutableArray *phoneArray = [[NSMutableArray alloc] initWithArray:resultArray];
        NSMutableArray *urlArray = [[NSMutableArray alloc] init];
        NSMutableArray *emailArray = [[NSMutableArray alloc] init];
        
        NSString *parten4 = @"[^\\s]*(\\.)(com|cn|net|org|biz|info|cc|tv|name|中国|mobi|me|asia|co|so|tel)[a-zA-Z\\d\\/]*";
        NSArray *resultArray4 = [text componentsMatchedByRegex:parten4];
        for (NSString *str in resultArray4) {
            BOOL isMatch = NO;
            
            NSString *parten3 = @"(https?)+://[^\\s]*[a-zA-Z\\d\\.]*(\\.)(com|cn|net)[a-zA-Z\\d\\/]*";
            NSArray *resultArray3 = [str componentsMatchedByRegex:parten3];
            for (NSString *s in resultArray3) {
                isMatch = YES;
                [urlArray addObject:s];
            }
            
            NSString *parten2 = @"(www\\.)[^\\s]*(\\.)(com|cn|net|org|biz|info|cc|tv|name|中国|mobi|me|asia|co|so|tel)[a-zA-Z\\d\\/]*";
            NSArray *resultArray2 = [str componentsMatchedByRegex:parten2];
            for (NSString *s in resultArray2) {
                isMatch = YES;
                [urlArray addObject:s];
            }
            
            NSString *parten4 = @"[^\\s]*[a-zA-Z\\d\\.]*(@)[a-z\\d]*(\\.)(com|cn|net|org|biz|info|cc|tv|name|mobi|me|asia|co|so|tel)";
            NSArray *resultArray4 = [str componentsMatchedByRegex:parten4];
            for (NSString *s in resultArray4) {
                isMatch = YES;
                [emailArray addObject:s];
            }
            
            if (isMatch == NO) {
                [urlArray addObject:str];
            }
            
            for (NSString *p in resultArray) {
                NSArray *resultArray3 = [str componentsMatchedByRegex:p];
                if ([resultArray3 count] > 0) {
                    [phoneArray removeObject:p];
                }
            }
        }

        NSMutableArray *array = [[NSMutableArray alloc] init];
        [array addObject:urlArray];
        [array addObject:phoneArray];
        [array addObject:emailArray];
        
        return array;
    }

调用的时候，如下

    NSArray *rexArray = [M80AttributedLabel addRexArr:label.labelText];
    NSArray *httpArr = [rexArray objectAtIndex:0];
    NSArray *phoneNumArr = [rexArray objectAtIndex:1];
    NSArray *emailArr = [rexArray objectAtIndex:2];
    if ([emailArr count]) {
        for (NSString *emailStr in emailArr) {
            [label addCustomLink:[NSURL URLWithString:emailStr] forRange:[label.labelText rangeOfString:emailStr] linkType:LinkTypeEmail];
        }
    }
    if ([phoneNumArr count]) {
        for (NSString *phoneNum in phoneNumArr) {
            [label addCustomLink:[NSURL URLWithString:phoneNum] forRange:[label.labelText rangeOfString:phoneNum] linkType:LinkTypePhone];
        }
    }
    if ([httpArr count]) {
        for (NSString *httpStr in httpArr) {
            [label addCustomLink:[NSURL URLWithString:httpStr] forRange:[label.labelText rangeOfString:httpStr] linkType:LinkTypeURL];
        }
    }
    

2、还有表情的添加，原来只能单个加图片，如果图文都有，比如：开心哦[大笑]，哦也！   其中[大笑]是一个笑的图片。如何分割显示呢？

    NSRegularExpression *regex = [[NSRegularExpression alloc]
                                  initWithPattern:@"(.*?)(\\[*+\\]|\\Z)"
                                  options:NSRegularExpressionCaseInsensitive|NSRegularExpressionDotMatchesLineSeparators
                                  error:nil];
    NSArray* chunks = [regex matchesInString:text options:0
                                       range:NSMakeRange(0, [text length])];
    for (NSTextCheckingResult *b in chunks) {
        NSString *bstr = [text substringWithRange:b.range];
        if (bstr.length > 0) {
            NSString *imgName = [_m_emojiDictionary objectForKey:bstr];
            if (imgName) {
                UIImage *image = [UIImage imageNamed:imgName];
                [label appendImage:image
                           maxSize:CGSizeMake(24, 24)
                            margin:UIEdgeInsetsZero
                         alignment:M80ImageAlignmentCenter];
            }else{
                NSArray *array = [bstr componentsSeparatedByString:@"["];
                int i = 0;
                for (NSString *str in array) {
                    if (i == 0) {
                        [label appendText:str];
                    }else{
                        NSString *astr = [NSString stringWithFormat:@"[%@", str];
                        NSString *imgName = [_m_emojiDictionary objectForKey:astr];
                        if (imgName) {
                            UIImage *image = [UIImage imageNamed:imgName];
                            [label appendImage:image
                                       maxSize:CGSizeMake(24, 24)
                                        margin:UIEdgeInsetsZero
                                     alignment:M80ImageAlignmentCenter];
                        }else{
                            [label appendText:astr];
                        }
                    }
                    i++;
                }
            }
        }
    }

详情代码参考，<a href="https://github.com/xpmozong/Socket" target="_blank">https://github.com/xpmozong/Socket</a>，WeiWei这个项目。

