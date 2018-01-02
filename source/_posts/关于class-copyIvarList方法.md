---
title: 关于class_copyIvarList方法
date: 2017-12-19 17:49:51
tags: [iOS, Runtime]
categories: iOS
---

在`runtime.h`中，你可以通过其中的`class_copyIvarList`方法来获取实例变量。具体的实现如下（记得导入头文件`#import <objc/runtime.h>`）

```objc
- (NSArray *)ivarArray:(Class)cls {
    unsigned int ivarCount = 0;
    Ivar *ivars = class_copyIvarList(cls, &ivarCount);
    if (stuIvarCount == 0) {
        return nil;
    }
    NSMutableArray *arr = [[NSMutableArray alloc] initWithCapacity:ivarCount];
    for (int i = 0;i<ivarCount;i++) {
        Ivar ivar = ivars[i];
        NSString *ivarName = [NSString stringWithUTF8String:ivar_getName(ivar)];
        NSLog(@"%@",ivarName);
        [arr addObject:ivarName];
    }
    free(ivars);
    return arr;
}
```

如上面代码。其中`cls`就是你要获取实例变量的类，`ivarCount`用来承载要获取类的实例变量的个数。打印出来的`ivarName`就是`cls`的实例变量。接下来对这个方法进行解析:
首先看一下里面的`Ivar`,先看一下定义：

```objc
/// An opaque type that represents an instance variable.
typedef struct objc_ivar *Ivar;
struct objc_ivar {
    char * _Nullable ivar_name                               OBJC2_UNAVAILABLE;  //变量名字
    char * _Nullable ivar_type                               OBJC2_UNAVAILABLE;   //变量类型
    int ivar_offset                                          OBJC2_UNAVAILABLE; //偏移量
    #ifdef __LP64__
    int space                                                OBJC2_UNAVAILABLE;  //存储空间
    #endif
}
```
`Ivar`是一个叫做`objc_ivar`的结构体指针,其中的 `ifdef`判断是判断当前设备是否是64位设备，这里可以延伸出一个方法：

```objc
//判断当前设备是否是64位设备，也可以用这个方法判断是否是32位设备
- (BOOL)is64Bit {
    #if defined(__LP64__) && __LP64__
        return YES;
    #else
        return NO;
    #endif
}
```
```objc
OBJC_EXPORT Ivar _Nonnull * _Nullable
class_copyIvarList(Class _Nullable cls, unsigned int * _Nullable outCount) 
    OBJC_AVAILABLE(10.5, 2.0, 9.0, 1.0, 2.0);
```

`class_copyIvarList`的注释如下：
{% asset_img pic1.png class_copyIvarList %}

它返回的是一个`Ivar`的数组，这个数组里面包含了你要查看类的所有实例变量，但是不包括从父类继承过来的。如果你传入的类没有实例变量或者该`class`为`Nil`,那么该方法返回的就是`NULL`，`count`值也就变成了0。有一点需要注意：你**必须使用`free()`方法**将该数组释放。
然后就是通过`for`循环遍历，通过`ivar _ getName`拿到`ivarName`。
以上便是对`clas_copyIvarList`的介绍。
它还有一个最常用的使用方式（在开发中经常用到的）：根据字典或者`json`字符串转化为`model`，在网络请求返回数据时经常用到。使用方法就是自己写一个基类的`model`，然后让项目中用到的`model`都继承自此基类，基类中的关键代码如下：

```objc
+ (instancetype)km_modelFromDic:(NSDictionary *)dataDic {
    id model = [[self alloc] init];  
    unsigned int count = 0;
    Ivar *ivarsA = class_copyIvarList(self, &count);
    if (count == 0) {
        return model;
    }
    for (int i = 0;i < count; i++) {
        Ivar iv = ivarsA[i];
        NSString *ivarName = [NSString stringWithUTF8String:ivar_getName(iv)];
        ivarName = [ivarName substringFromIndex:1];
        id value = dataDic[ivarName];
        [model setValue:value forKey:ivarName];
    }
    free(ivarsA);
    return model;
}
```
这里是把字典转成`model`，先用`class_copyIvar`获取该`model`的所有实例变量,然后通过`kvc`对属性进行赋值。最终返回`model`。这里有个点需要注意以下几点：

 - 你的`model`的属性名称要和服务端返回的数据一致，比如你的`model`有个属性叫做`name`，那么你服务端返回的数据字典里面的对应属性也要叫做`name`，因为这个方法是根据属性从字典里面拿数据的。你也可以做一个映射，让自定义的实例变量名称映射到服务端提供的变量名称。
 - 实现里面有个`substringFromIndex：`操作，其目的就是把使用该方法拿到的实例变量前面的" _ "去掉。所以你最好使用 @property 进行属性声明，并且不要去修改自动生成的实例变量。（`@property` = `getter` + `setter` + `_ ivar`，这里的 `_ ivar`其实就是编译器帮我们生成的实例变量）  
 
接下来你可以尝试去获取`UILabel`的实例变量列表：
```objc
[self ivarArray:[UILabel class]]
```
你会发现拿到的结果是这样的：

```
(
    "_size",
    "_highlightedColor",
    "_numberOfLines",
    "_measuredNumberOfLines",
    "_baselineReferenceBounds",
    "_lastLineBaseline",
    "_previousBaselineOffsetFromBottom",
    "_firstLineBaseline",
    "_previousFirstLineBaseline",
    "_minimumScaleFactor",
    "_content",
    "_synthesizedAttributedText",
    "_defaultAttributes",
    "_fallbackColorsForUserInterfaceStyle",
    "_minimumFontSize",
    "_lineSpacing",
    "_layout",
    "_scaledMetrics",
    "_cachedIntrinsicContentSize",
    "_contentsFormat",
    "_cuiCatalog",
    "_cuiStyleEffectConfiguration",
    "_textLabelFlags",
    "_adjustsFontForContentSizeCategory",
    "__textColorFollowsTintColor",
    "_preferredMaxLayoutWidth",
    "_multilineContextWidth",
    "__visualStyle"
)
```

但是跳转到`UILabel.h`，你会发现里面有好多的属性不包含在我们根据该方法得出的属性数组里面，而且使用该方法得到的属性在`UILabel.h`里面并没有。这个是什么原因呢？
先看一下好多`UILabel`里面的属性没有在数组里面打印的问题：猜想应该是在`UILabel.m`里面使用了 `@dynamic`。导致没有自动生成`getter`、`setter`和`ivar`，所以没有在数组里面包含。

> `@synthsize`：如果没有手动实现`setter`/`getter`方法那么会自动生成，自动生成`_var`变量。如果不写，默认生成`getter`/`setter`和`_var`。你也可以使用该关键字自己设置自动变量的名称。
`@dynamic`告诉编译器：属性的`setter`/`getter`需要用户自己实现，不自动生成，而且也不会产生`_var`变量。

也就是说在`UILabel`里面虽然有个`text`的属性，也许在`UILabel.m`里面已经包含：
```objc
@dynamic text;
```
这样的话在实现里面没有产生实例变量，只是手动实现了`getter`和`setter`，所以就不会显示`text`属性在刚才得到的数组里面了。
至于数组中有`UILabel.h`里面没有的变量，这个就好理解了，有可能在`UILabel.m`里面添加了一些实例变量或者在运行时添加了这些实例变量。

除此方法之外，你还可以使用`class_copyPropertyList`方法，这个是拿到的所有用 `@property` 声明的属性,包括在`.m`里面添加的属性（所以打印出来的可能要比真实在`.h`里面看到的多），具体实现和上面的获取方法类似：

```objc
- (NSArray *)propertyArr:(Class)cls {
    unsigned count = 0;
    objc_property_t *properties = class_copyPropertyList(cls, &count);
    if (count == 0) {
        return nil;
    }
    NSMutableArray *arr = [[NSMutableArray alloc] initWithCapacity:count];
    for (int i = 0; i < count; i ++) {
        objc_property_t property = properties[i];
        NSString *propertyName = [NSString stringWithUTF8String:property_getName(property)] ;
        [arr addObject:propertyName];
    }
    free(properties);
    return arr;
}
```
其中的`copyPropertyList`方法解释如下：
{% asset_img pic2.png copyPropertyList %}

记得使用过后也要调用`free`去释放数组。（PS：在源代码中暂未找到`objc_property`结构体的说明）因此，你可以通过使用该方法来实现字典或者`json`字符串转`model`操作：

```objc
+ (instancetype)km_modelFromDic:(NSDictionary *)dataDic {
    id model = [[self alloc] init];
    unsigned int count = 0;
    objc_property_t *properties = class_copyPropertyList([self class], &count);
    if (count == 0) {
        return model;
    }
    for (int i = 0;i < count; i++) {
        objc_property_t property = properties[i];
        NSString *propertyName = [NSString stringWithUTF8String:property_getName(property)];
        id value = dataDic[propertyName];
        [model setValue:value forKey:propertyName];
    }
    free(properties);
    return model;
}
```
两种方式均可实现字典到`model`的转换操作。