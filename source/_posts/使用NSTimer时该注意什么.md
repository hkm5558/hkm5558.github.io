---
title: 使用NSTimer时该注意什么
date: 2018-01-03 17:57:29
tags: [iOS, Timer]
categories: Question
---

**NSTimer的常用API**

```objc
+ (NSTimer *)timerWithTimeInterval:(NSTimeInterval)ti target:(id)aTarget selector:(SEL)aSelector userInfo:(nullable id)userInfo repeats:(BOOL)yesOrNo;
  ///Initializes a timer object with the specified object and selector.
+ (NSTimer *)scheduledTimerWithTimeInterval:(NSTimeInterval)ti target:(id)aTarget selector:(SEL)aSelector userInfo:(nullable id)userInfo repeats:(BOOL)yesOrNo;
 /// Creates a timer and schedules it on the current run loop in the default mode.
```
根据官方文档方法一和方法二的区别在于：使用方法一创建的`Timer`不会添加到`NSRunLoop`需要手动添加；使用方法二会自动添加到主线程的`RunLoop`中。

```objc
- (void)fire; 
  ///Causes the timer's message to be sent to its target.
- (void)invalidate;
  ///Stops the timer from ever firing again and requests its removal from its run loop.
```
`fire`方法可以立即触发`Timer`对象的`target`方法；`invalidate`会停止`Timer`并将其从`runloop`中移除

**NSRunLoop & NSTimer**
当使用`NSTimer`的`scheduledTimerWithTimeInterval`方法时，`NSTimer`的实例会被加入到当前线程的`RunLoop`中，模式为默认模式`NSDefaultRunLoopMode`。
```objc
- (void)viewDidLoad
{
    [super viewDidLoad];
        
    NSLog(@"主线程 %@", [NSThread currentThread]);
    NSTimer *timer = [NSTimer timerWithTimeInterval:2.0 target:self selector:@selector(doSomething) userInfo:nil repeats:YES];
    //使用NSRunLoopCommonModes模式，把timer加入到当前Run Loop中。
    [[NSRunLoop currentRunLoop] addTimer:timer forMode:NSRunLoopCommonModes];
}

- (void)doSomething
{
    NSLog(@"Timer %@", [NSThread currentThread]);
    //控制台输出结果：上下打印出来的线程都是主线程
}

```
如果当你线程是主线程也就是UI线程时，某些UI事件（`UIScrollView`的滑动操作），会将`RunLoop`切换到`NSEventTrackingRunLoopMode`模式。
在这个过程中，默认的`NSDefaultRunLoopMode`模式中注册的事件是不会被执行的。也就是说，此时使用`scheduledTimerWithTimeInterval`添加到`RunLoop`中的`Timer`就不会执行。这是一个在开发中经常碰到的场景，如何解决这个问题？我们可以在创建完一个`Timer`后使用`NSRunLoop`的`addTimer:forMode:`方法，使用`NSRunLoopCommonModes`。这是一个占位用的`Mode`，可以在不同情况下扮演不同的角色——`NSDefaultRunLoopMode` & `UITrackingRunLoopMode`。

**NSTimer循环引用问题**
`NSTimer`为什么会造成循环引用？
在开发中我们经常将`Timer`作为控制器的属性来使用，这样一来控制器对`Timer`进行了强引用。在`target`-`action`这个过程中，`Timer`又对`self`做了强引用，这就是导致循环引用的原因了。在网上有非常多的解决方案，总结了一下有下面几种。

 - 方法一：在`viewWillDisappear`中执行下面的操作。  
 
```objc
 - (void)viewWillDisappear:(BOOL)animated {
  [super viewWillDisappear:animated];

  [_timer invalidate];
  _timer = nil;
}
```
苹果文档中关于`target`强引用的解释
```objc
 - (NSTimer *)timerWithTimeInterval:(NSTimeInterval)ti target:(id)target selector:(SEL)aSelector userInfo:(id)userInfo repeats:(BOOL)repeats

  Parameters
  
  ti 
  The number of seconds between firings of the timer. If ti is less than or equal to 0.0, this method chooses the nonnegative value of 0.1 milliseconds instead.
  
  target 
  The object to which to send the message specified by aSelector when the timer fires. The timer maintains a strong reference to this object until it (the timer) is invalidated.
```
`The timer maintains a strong reference to this object until it (the timer) is invalidated.`苹果文档说`invalidate`以后`timer`就不再保有对`target`的强引用了。所以解决循环引用的关键在于`invalidate`方法有没有执行。下面`_timer = nil`这句话的意义是，`invalidate`方法执行以后`Timer`就不能复用了，为了防止在其之后其他地方再次使用`Timer`，在这里将其置为nil。

 - 方法二：引入一个代理对象，让其弱引用`self`，参考<a href="https://github.com/ibireme/YYKit/blob/3869686e0e560db0b27a7140188fad771e271508/YYKit/Utility/YYWeakProxy.m" target="_blank">YYWeakProxy</a>
没引入代理(`Proxy`)之前是：`self` -> `timer` -> `self`这样的循环引用。在引入代理(`Proxy`)之后是：`self` -> `timer` -> `proxy` ··> self。这样一来就打破了之前的循环引用。

 - 方法三：使用`YYTimer`和`MSWeakTimer`等`GCD`实现的`Timer`。

