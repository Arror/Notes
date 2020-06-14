# Named Color (iOS 11)

在iOS11.0之后，苹果提供了在xcassets中管理颜色的方式。在代码中我们可以通过一下方式使用我们定义的颜色。

```objc
[UIColor colorNamed:@"Color Name"];
[UIColor colorNamed:@"Color Name" inBundle:bundle compatibleWithTraitCollection:traitCollection];
```
```swift
UIColor(named: "Color Name")
UIColor(named: "Color Name", in: bundle, compatibleWith: traitCollection)
```

在`Interface Builde`r上面我们可以通过色板去选择我们定义的颜色，在`Named Color`分组。`Xcode`会列举出工程中所有定义的颜色。


在`Interface Builder`使用`Named Color`没有记录`Bundle`信息，所以`Interface Builder`会在其所在的`Bundle`中通过`Name`去寻找该颜色，如果没找到就是使用`Fallback Color`（`Fallback Color`是使用颜色时直接写入`Interface Builder`上面的当时该颜色的值）。

那么问题来了，当`Interface Builder`和颜色分别在不同的`Bundle`时，我们在`Interface Builder`依然可以使用该颜色，但是如果我们在某个变更中修改了该颜色，并且没有更新`Interface Builder`上的`Fallback Color`（打开`Interface Builder`就会自动更新），那么在运行时`Interface Builder`会因为找不到对应的颜色而使用`Fallback Color`，而`Fallback Color`是修改之前的。我们会看到颜色的修改没有生效。

我们不希望我每次修改颜色之后，还要去检查所有的`Interface Builder`去更新`Fallback Color`。同时又想享受`Named Color`给我们带来的便利。我们需要一些手段来帮助我们解决这个问题。

## 方案

1. 我们要统一管理颜色，一般情况下是在私有库中定义，保证所有颜色定义在同一个`Bundle`中。
2. 当`Interface Builder`通过`Name`去寻找颜色的时候我们做一次重定向。

可根据具体工程环境做相应调整

`CODE_GENERATED_MACRO`是所有我们定义的颜色的名称`@"a" @"b", @"c"`的宏，我们可以通过脚本生成，同时也避免硬编码带来书写上的失误。另外，我们可以通过脚本来生成颜色的定义，可以在代码中有更好的使用体验。


```objc
#import <UIKit/UIKit.h>
#import <objc/runtime.h>

@interface __Private__ColorNamesStorage : NSObject

@property(nonatomic, strong) NSBundle *bundle;
@property(nonatomic, strong) NSSet<NSString *> *names;

@end

static __Private__ColorNamesStorage *_shareStorage;

@implementation __Private__ColorNamesStorage

- (instancetype)init {
    if (self = [super init]) {
        _bundle = [NSBundle bundleForClass:[__Private__ColorNamesStorage class]];
        _names = [NSSet setWithObjects:CODE_GENERATED_MACRO, nil];
    }
    return self;
}

+ (__Private__ColorNamesStorage *)sharedInstance {
    static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{
        _shareStorage = [[self alloc] init];
    });
    return _shareStorage;
}

@end

@implementation UIColor (Named)

+ (void)load {
    static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{
        Method o = class_getClassMethod(self, @selector(colorNamed:inBundle:compatibleWithTraitCollection:));
        Method s = class_getClassMethod(self, @selector(swizzled_colorNamed:inBundle:compatibleWithTraitCollection:));
        method_exchangeImplementations(o, s);
    });
}

+ (UIColor *)swizzled_colorNamed:(NSString *)name inBundle:(NSBundle *)bundle compatibleWithTraitCollection:(UITraitCollection *)traitCollection {
    NSBundle *targetBundle = [[__Private__ColorNamesStorage sharedInstance].names containsObject:name ?: @""] ? [__Private__ColorNamesStorage sharedInstance].bundle : bundle;
    return [self swizzled_colorNamed:name inBundle:targetBundle compatibleWithTraitCollection:traitCollection];
}

@end

```