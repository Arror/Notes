# UIImageNibPlaceholder

```swift
@import UIKit;
@import ObjectiveC.runtime;
@import ImageDemo;

id (*UIImageNibPlaceholderInitWithCoder)(id, SEL, NSCoder *) = NULL;
id (*UIImageNibPlaceholderInitWithOtherImage)(id, SEL, UIImage *) = NULL;

static id HookedImageNibPlaceholderInitWithCoder(id self, SEL _cmd, NSCoder *coder) {
    NSString *name = (NSString *)[coder decodeObjectForKey:@"UIResourceName"];
    if ([name isEqualToString:@"wechat"]) {
        UIImage *image = [UIImage imageNamed:name inBundle:[NSBundle bundleForClass:DDImage.class] compatibleWithTraitCollection:nil];
        if (image == nil) {
            return nil;
        } else {
            return UIImageNibPlaceholderInitWithOtherImage(self, _cmd, image);
        }
    } else {
        return UIImageNibPlaceholderInitWithCoder(self, _cmd, coder);
    }
}

@interface UIImageHook: NSObject

@end

@implementation UIImageHook

+ (void)load {
    static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{
        Class clazz = NSClassFromString(@"UIImageNibPlaceholder");
        {
            SEL sel = NSSelectorFromString(@"initWithCoder:");
            IMP imp = (IMP)HookedImageNibPlaceholderInitWithCoder;
            Method method = class_getInstanceMethod(clazz, sel);
            UIImageNibPlaceholderInitWithCoder = (void *)method_getImplementation(method);
            method_setImplementation(method, imp);
        }
        {
            SEL sel = NSSelectorFromString(@"_initWithOtherImage:");
            Method method = class_getInstanceMethod(clazz, sel);
            UIImageNibPlaceholderInitWithOtherImage = (void *)method_getImplementation(method);
        }
    });
}

@end
```
