# Scroll To Top

苹果并没有提供相应的API，但是在`UIScrollView`、`UIWindow`找到了相应的私有API。

```swift
extension UIScrollView {
    
    @objc public func extension_scrollToTopIfPossible(animated: Bool) {
        let selector = NSSelectorFromString("_scrollToTopIfPossible:")
        guard self.responds(to: selector) else {
            return
        }
        self.performSelector(onMainThread: selector, with: animated ? true : nil, waitUntilDone: true)
    }
}
```

或者
```Objective-C
@implementation UIWindow (SCROLL)

- (void)performScrollToTop {
    
    SEL selector = NSSelectorFromString(@"_scrollToTopViewsUnderScreenPointIfNecessary:resultHandler:");
    if ([self respondsToSelector:selector] == false) {
        return;
    }
    
    NSMethodSignature *signature = [UIWindow instanceMethodSignatureForSelector:selector];
    if (signature == nil) {
        return;
    }
    
    NSInvocation *invocation = [NSInvocation invocationWithMethodSignature:[UIWindow instanceMethodSignatureForSelector:selector]];
    [invocation setTarget:self];
    [invocation setSelector:selector];
    CGRect statusBarFrame = UIApplication.sharedApplication.statusBarFrame;
    CGPoint point = CGPointMake(statusBarFrame.size.width / 2.0, statusBarFrame.size.height + 1.0);
    [invocation setArgument:&point atIndex:2];
    [invocation invoke];
}

@end
```

因为API并未公开，所以可能回应为调用私有API被拒，可以将API字符串进行混淆处理。

