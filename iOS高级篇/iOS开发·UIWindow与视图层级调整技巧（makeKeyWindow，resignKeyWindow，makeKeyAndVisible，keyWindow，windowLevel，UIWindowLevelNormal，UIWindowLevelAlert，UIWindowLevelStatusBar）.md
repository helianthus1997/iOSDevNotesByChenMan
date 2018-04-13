# iOS开发·UIWindow与视图层级调整技巧（makeKeyWindow，resignKeyWindow，makeKeyAndVisible，keyWindow，windowLevel，UIWindowLevelNormal，UIWindowLevelAlert，UIWindowLevelStatusBar）

> iOS开发过程中，多人开发或者导入第三方框架的时候，可能碰到UIWindow层级冲突的问题。

例如，很多人习惯在keyWindow上添加一个自定义浮层视图，但是，当自己或者其它第三方框架曾经调高过其它自定义UIWindow属性windowLevel，或者有其它同级windowLevel的UIWindow后来改变过显示状态（如.hidden=NO，makeKeyAndVisible等），而且又**没有** 设将其设置为keyWindow，结果导致正在显示的UIWindow不是keyWindow，从而导致添加到keyWindow上自定义视图无法显示（被覆盖了）。

![如何查看App的UIWindow层级](http://upload-images.jianshu.io/upload_images/1283539-934bcf12bebfe5ae.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


## 一. 为App初始化一个默认UIWindow对象

在AppDelegate.m中需要初始化一个window属性，作为后面往App添加视图的容器

##### 1. 初始化操作写在如下UIApplicationDelegate代理方法中
```
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {
```
##### 2. 一个初始化window操作示例如下，具体根据产品需求设置
```
self.window = [[UIWindow alloc]initWithFrame:[UIScreen mainScreen].bounds];
self.window.backgroundColor = [UIColor whiteColor];
[self.window makeKeyAndVisible];
```
##### 3. 接下来需指定window的rootViewController
```
//判断该用首次欢迎页or密码登录页等等
if ([CommonUtils isStringNilOrEmpty:[CommonUtils getStrValueInUDWithKey:kIsGuide]]) {
    [CommonUtils saveStrValueInUD:@"1" forKey:kGuide];
    // 欢迎页置为rootViewController
    self.window.rootViewController = [[WHGuidePagesController alloc]initWithNibName:@"WHGuidePagesController" bundle:nil];
}else{
    // 检查各种密码决定登录方式，并分别设置rootViewController
    [self chooseVerifyMethod];
}
```
##### 4. 注意点：rootViewController属性
- 目前只有UIWindow**有**rootViewController这个**属性**，不要跟UINavigationController里面的根视图概念混淆。
- UINavigationController其实**并没有** rootViewController这个属性！也就没有自带的setter方法。要设置其根视图只能通过如下方法，而不能通过属性的setter方法和点语法设置根视图。
```
- (instancetype)initWithRootViewController:(UIViewController *)rootViewController; 
// Convenience method pushes the root view controller without animation.
```
##### 5. 大多数APP的视图层级关系（以有底部TabBar的App为例）
-  1). [UIApplication sharedApplication].keyWindow为UIWindow对象。比如，获取APP的keyWindow并往上添加视图的代码：
```
[[UIApplication sharedApplication].keyWindow addSubview:self.signView];
```
 - 2). 假设APP的keyWindow对象为uiWindow，则uiWindow.rootViewController为UITabBarController对象（也只有UIWindow可以用点语法设置根视图）。比如，为设置rootViewController代码：
```
self.window.rootViewController = customTabBarVC;//AppDelegate.m里面
```
- 3). UITabBarController对象的viewControllers包含UINavigationController对象。设置其viewControllers的方法：
```
- (void)setViewControllers:(NSArray<__kindof UIViewController *> * __nullable)viewControllers animated:(BOOL)animated;
```
 - 4). UINavigationController对象的rootViewController为UIViewController对象。初始化其rootViewController的方法为：
```
- (instancetype)initWithRootViewController:(UIViewController *)rootViewController;
```

##### 6. 获取keyWindow(它并不一定是当前最上层显示的window)的rootViewController
可以通过如下方法找到当前UIWindow的rootViewController，前提是当keyWindow真的显示在最上层。
```
#pragma mark - 获取根视图的(导航、标签)视图控制器
+ (UINavigationController *)getRootVCformViewController
{
    UIViewController *rootVC = [UIApplication sharedApplication].keyWindow.rootViewController;
    UINavigationController *nav = nil;
    if ([rootVC isKindOfClass:[UITabBarController class]]) {
        UITabBarController *tabbar = (UITabBarController *)rootVC;
        NSInteger index = tabbar.selectedIndex;
        nav = tabbar.childViewControllers[index];
    }else if ([rootVC isKindOfClass:[UINavigationController class]]) {
        nav = (UINavigationController *)rootVC;
    }else if ([rootVC isKindOfClass:[UIViewController class]]) {
        NSLog(@"This no UINavigationController...");
    }
    return nav;
}
```

## 二. 在自定义的UIWindow添加自定义视图

假设想为一个APP添加一个手势验证的页面，当进入APP弹出这个手势验证页面。如果不想影响原来的UIWindow，可以考虑新建一个UIWindow并覆盖原来的UIWindow，并往新建的UIWindow上添加各种手势相关的视图及控制器。但在手势验证完后，务必销毁这个自定义的UIWindow，否则可能导致看不见的UIWindow越积越多。

##### 1. 自定义UIWindow
```
_window = [[UIWindow alloc]initWithFrame:[[UIScreen mainScreen] bounds]];
_window.hidden = NO;
[self.window makeKeyAndVisible];
```
##### 2. 指定自定义视图控制器
```
UIViewController *vc = [[UIViewController alloc]init];
_window.rootViewController = vc;
```
##### 3. 销毁自定义UIWindow

自定义视图用完后，记得要销毁自定义的UIWindow，否则导致APP以后会有越来越多没用到的UIWindow，即使再也没有显示过它们，但是可以用调试工具看到许多废弃的window。可参考方法如下
```
- (void)dismiss {
    
    [self.window resignKeyWindow];
    self.window.windowLevel = -1000;
    self.window.hidden = YES;
    [self.window.rootViewController dismissViewControllerAnimated:YES completion:nil];
    
    self.window = nil;
}
```
## 三. UIWindow的显示特性

#### 1. 相同windowLevel下，调整UIWindow显示层的基本方法

##### 1). 显示相关属性：hidden 
- 如果仅仅想显示一个UIWindow
```
customWindow.hidden = NO;
```
> PS: 虽然设置自己的hidden即可显示出来，但上述方法并不会"自动"影响之前显示的UIWindow对象的hidden属性。如果，之前UIWindow的hidden = NO，设置新UIWindow的hidden将旧UIWindow覆盖后，旧UIWindow的hidden属性依旧为NO。
- 如果仅仅想隐藏一个UIWindow
```
customWindow.hidden = YES;
```
> PS: 如果你没有专门设置过hidden属性，系统默认为YES。上述代码会将UIWindow绝对隐藏，不管有没其他UIWindow覆盖。当也没有其它非隐藏的UIWindow的时候，APP屏幕完全黑屏。

- 如果想显示一个UIWindow，同时设置为keyWindow，并将其显示在同一windowLevel的其它任何UIWindow之上
```
- (void)makeKeyAndVisible
```
> PS: 上述方法真的会将其显示在同一windowLevel的其它任何UIWindow之上！显示最上层的UIWindow以最后执行过该代码的UIWindow为准。

##### 2). 显示相关方法：makeKeyAndVisible的作用

```
[self.window makeKeyAndVisible];
```
其执行效果包括 **但不限于** 执行了如下代码（因为还会覆盖同level的所有window）：
```
[self.window makeKeyWindow];
self.window.hidden = NO;
```
讲真，makeKeyAndVisible真的会自动改变hidden属性值为NO。
##### 3). UIWindow对象的hidden属性默认值
- 默认值：YES

> PS：如果你仅仅创建一个UIWindow，而又不专门设置hidden属性（或者makeKeyAndVisible），系统默认分配的默认值为true。例如，我们把影响到hidden属性的方法屏蔽掉：
```
self.window = [[UIWindow alloc]initWithFrame:[UIScreen mainScreen].bounds];
self.window.backgroundColor = [UIColor whiteColor];
// [self.window makeKeyAndVisible];
// [self.window makeKeyWindow];
// self.window.hidden = NO;
```
再来打印hidden属性如下：
```
po self.window.isHidden
true
```

##### 4). 误区：关于keyWindow的混淆易错点

设置keyWindow与否并**不** 影响视图层级**显示**，仅来接收键盘及其它非触摸事件。如果没有专门设置过keyWindow的hiden为NO，而且也没有其它非隐藏的UIWindow，那么APP会黑屏。

- 如果仅仅设置为keyWindow
```
- (void)makeKeyWindow
```
- 如果仅仅解除为keyWindow
```
- (void)resignKeyWindow
```

> app的keyWindow与是否在最上层显示没有任何关系。比如，你如果想通过`[[UIApplication sharedApplication] keyWindow]`获取正在显示的UIWindow是**极其不准确** 的。有时候通过这个代码获取的如果真的是正在显示的UIWindow，仅仅是因为碰巧而已。

##### 5). 警惕点：有多个hidden属性=NO的UIWindow，该显示谁？

如上所见，makeKeyAndVisible与hidden的setter方法均可以改变hidden的值，但有个问题，经过多次调整，可能有多个UIWindow的hidden都为NO，那么应该显示谁？
- 对于hidden的setter方法，最终显示的以**最后** 执行过 **.hidden=NO** 的UIWindow为准，且执行 **.hidden=NO** 之前hidden的值为YES。（hidden如果是从NO改为NO的**不** 算 **最后** 改变UIWindow的显示状态）
- 对于makeKeyAndVisible方法，最终显示的以**最后** 执行过 **makeKeyAndVisible** 的UIWindow为准。
- 对于先后分别用makeKeyAndVisible方法和hidden的setter方法，还是先后分别用hidden的setter方法和makeKeyAndVisible方法，结局同样以最后改变显示状态的UIWindow为准。

### 2. 基于windowLevel，调整UIWindow显示层的拓展方法

先去UIWindow.h里面看看UIWindowLevel的定义：
```
typedef CGFloat UIWindowLevel;
UIKIT_EXTERN const UIWindowLevel UIWindowLevelNormal;
UIKIT_EXTERN const UIWindowLevel UIWindowLevelAlert;
UIKIT_EXTERN const UIWindowLevel UIWindowLevelStatusBar __TVOS_PROHIBITED;
```


例如，在手势相关类中调整自定义的UIWindow层级
```
[self.window makeKeyAndVisible]; 
_window.windowLevel = UIWindowLevelAlert;
```
- 打印代表UIWindowLevelAlert层级的数据值
```
(lldb) po self.window.windowLevel
2000
```
- 同理，打印代表UIWindowLevelStatusBar层级的数据值
```
(lldb) po self.window.windowLevel
1000
```
- 同理，打印代表UIWindowLevelNormal层级的数据值
```
(lldb) po self.window.windowLevel
0
```
**小结：**
>1. windowLevel数值越大的显示在窗口栈的越上面
>2. **显示层的优先级** 为： UIWindowLevelAlert > UIWindowLevelStatusBar > UIWindowLevelNormal
>3. 系统给UIWindow默认的windowLevel为UIWindowLevelNormal

![Xcode查看窗口栈](http://upload-images.jianshu.io/upload_images/1283539-c10057e96a4ec7aa.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


## 四. UIWindow常见操作方法总结

##### 1. 获取App所有window的windows数组
```
 [[UIApplication sharedApplication] windows]
```
例如，第三方加载动画框架KVNProcess中KVNProgress.m文件会有一段这样的代码：
```
- (void)addToCurrentWindow
{
	UIWindow *currentWindow = nil;
	
	NSEnumerator *frontToBackWindows = [[[UIApplication sharedApplication] windows] reverseObjectEnumerator];
	
	for (UIWindow *window in frontToBackWindows) {
		if (window.windowLevel == UIWindowLevelNormal) {
			currentWindow = window;
			break;
		}
	}
	
	if (self.superview != currentWindow) {
		[self addToView:currentWindow];
	}
}
```
##### 2. keyWindow
```
[[UIApplication sharedApplication] keyWindow]
```
例如，第三方下拉菜单框架FFDropDownMenu的FFDropDownMenuView.m文件中有这样一段代码：
```
UIWindow *keyWindow = [UIApplication sharedApplication].keyWindow;
[keyWindow addSubview:self];
```
这段代码的目的是添加到最上层UIWindow，但实际操作是把自己的视图添加到keyWindow上。其实，如果我们在编写代码时严谨地保证keyWindow是显示在最上层的UIWindow，这样写没有问题。但如果：自己或者其它第三方框架曾经调高过其它UIWindow属性windowLevel，或者有同级windowLevel的其它UIWindow后来改变过显示状态（如.hidden=NO，makeKeyAndVisible等），可能会导致下拉菜单的弹出视图无法显示（被覆盖）。

##### 3. 获取AppDelegate单例的window属性
专门获取AppDelegate.m文件中的window属性，不包含其它其定义的window
```
[[[UIApplication sharedApplication] delegate] window]
```
拓展一下，获取AppDelegate单例的方法为
```
+ (AppDelegate *)sharedDelegate
{
    return (AppDelegate *)[[UIApplication sharedApplication] delegate];
}
```


---
## 附. 调试打印例子
- 启动APP，AppDelegate.m中的window属性
```
(lldb) po self.window
<UIWindow: 0x15fd24390; frame = (0 0; 320 568); gestureRecognizers = <NSArray: 0x1700567a0>; layer = <UIWindowLayer: 0x170233700>>
```
- 跳转手势，GestureScreen.m中的window属性
```
(lldb) po _window
<UIWindow: 0x15fd29160; frame = (0 0; 320 568); gestureRecognizers = <NSArray: 0x170057f70>; layer = <UIWindowLayer: 0x1702345c0>>
```
- 此时，可查看所有window
```
(lldb) po [[UIApplication sharedApplication] windows]
<__NSArrayM 0x17405c290>(
<UIWindow: 0x15fd24390; frame = (0 0; 320 568); gestureRecognizers = <NSArray: 0x1700567a0>; layer = <UIWindowLayer: 0x170233700>>,
<UIWindow: 0x15fd29160; frame = (0 0; 320 568); gestureRecognizers = <NSArray: 0x170057f70>; layer = <UIWindowLayer: 0x1702345c0>>
)
```
- 此时，断点在手势相关类中，也可专门查看AppDelegate.m中的window属性：假设```UIWindow *delegateWindow = [[[UIApplication sharedApplication] delegate] window];``` 打印如下
```
(lldb) po delegateWindow
<UIWindow: 0x15fd24390; frame = (0 0; 320 568); gestureRecognizers = <NSArray: 0x1700567a0>; layer = <UIWindowLayer: 0x170233700>>
```


