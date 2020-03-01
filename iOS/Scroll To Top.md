# Scroll To Top

苹果并没有提供相应的API，我使用运行时查看可UIScrollView的所有API，发现了`_scrollToTopIfPossible:`这个方法，该方法有一个`Bool`类型参数。当然我没有验证所有的系统版本，仅仅验证了iOS10及以上系统版本。

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

因为该API并未公开，所以可能回应为调用私有API被拒，可以将API字符串进行混淆处理。