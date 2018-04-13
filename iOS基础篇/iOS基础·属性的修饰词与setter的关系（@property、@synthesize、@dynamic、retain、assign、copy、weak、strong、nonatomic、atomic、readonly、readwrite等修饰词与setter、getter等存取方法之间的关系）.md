# iOS基础·属性的修饰词与setter的关系（@property、@synthesize、@dynamic、retain、assign、copy、weak、strong、nonatomic、atomic、readonly、readwrite等修饰词与setter、getter等存取方法之间的关系）


> 很多人讲属性修饰词的时候，喜欢从字面或者定义的角度介绍它们间的区别。这篇文章，我们侧重从修饰词对setter方法的影响直接展示区别。

## 1. 实例变量：命名区别于全局变量和局部变量
---
#### 1.1 命名法则：
  - 以下划线`_`作为实例变量名字的前缀，如`_student`
  - 这样，可以很容易地通过下划线区分实例变量与其它变量（全局变量，局部变量，静态变量等）
 
#### 1.2 声明位置：
  - 在.h头文件中
  - 或者，在.m实现文件的***类拓展*** 中

#### 1.3 声明形式:
  - 头文件，写在类似`@interface Person : NSObject {...}`这样的花括号`{...}`里面 
  - 实现文件的类拓展中，写在类似`@interface Person ()<UIScrollViewDelegate> {...} `这样的花括号`{...}`里面
 #### 1.4 声明示例：
```
@interface TYPagerController ()<UIScrollViewDelegate> {
    NSInteger   _countOfControllers;
    BOOL        _needLayoutContentView;
    CGFloat     _preOffsetX;
    float    _heightInMeters;

    struct {
        unsigned int transitionFromIndexToIndex :1;
        unsigned int transitionFromIndexToIndexProgress :1;
    }_delegateFlags;
    
    struct {
        unsigned int transitionFromIndexToIndex :1;
        unsigned int transitionFromIndexToIndexProgress :1;
    }_methodFlags;
}
```

#### 1.5 setter、getter方法
可以自己手动为实例变量在***头文件*** 中声明setter、getter方法，并在***实现文件***中实现setter、getter方法。你也可以不声明不实现，但不要再企图调用setter、getter方法了，甚至点语法。
```
//一个例子
//实现getter
- (float)heightInMeters{
    return _heightInMeters;
}
//实现setter
- (float)setHeightInMeters:(float)h{
    _heightInMeters = h;
}
```
#### 1.6 调用setter、getter方法
如果你实现了setter，getter方法，才可以调用***存取方法***，例如：
```
//调用getter
float temp = [self heightInMeters];
//调用setter
[self setHeightInMeters:10.0];
```
#### 1.7 点语法
如果你实现了setter，getter方法，才可以使用***点语法*** 简化调用***存取方法***。
```
//调用getter
float temp = self.heightInMeters;
//调用setter
self.heightInMeters = 10.0;
```
#### 1.8 实例变量类型：
   - 1）可以是简单的C类型，如  `int  _sudentNum;`，`float  _heightInMeters;`
> 这种实例变量及其值会在声明对象的内部保存。

   - 2）可以是指针，用来指向其他对象，如`NSString *tempStr`，`CMPersonModel *personModel`等等。这种属性叫***对象实例变量***。

> 这种变量，声明的对象内部仅保存指向相应实例对象的指针（对象地址），而不保存实例对象本身。实例对象本身由堆负责保存，管理机制由ARC负责。
#### 1.9 继承特性：
   - 子类继承不了父类写在***类拓展*** 中的示例变量

## 2. 属性：自动声明实例变量和存取方法，并实现存取方法
---
#### 2.1 声明位置：
   - 声明头文件
   - 或者实现文件的类拓展中
#### 2.2 声明形式：
   - 写在@interface与@end之间，花括号`{...}`之外
   - 必须有@property修饰词修饰，后面可选择性地添加其他修饰词如(nonatomic, strong) 等
#### 2.3 声明示例：
```
#import <UIKit/UIKit.h>
@interface TMAddCategoryViewController ()
@property (nonatomic, strong) NSMutableArray *dataSource;
@end
```
#### 2.4 存取方法：编译器会自动声明和实现
  - @property会让编译器自动声明相应的实例变量和存取方法，并实现存取方法。除非你用其它关键词修饰，专门告诉编译器做什么其它特殊处理。
  - 即使自动生成存取方法，遇到一些需求时，你也可以再自己重写存取方法。一般添加数据模型示例对象的时候，喜欢重写getter方法，设置一些默认值，这种叫懒加载。
  - 有一些***例外***，不会自动生成存取方法：
> 1. 同时重写了getter setter
> 2. 重写只读属性的 getter
> 3. 使用了@dynamic
> 4. @protocol 中定义的属性
> 5. category 中定义的属性
> 6. 重载的属性：当你在子类中重载父类的属性，你必须用 @synthesize 手动合成

#### 2.5 示例：重写getter
```
- (NSMutableArray *)dataSource
{
    if (!_dataSource) {
        _dataSource = [NSMutableArray array];
        RLMResults *results = [[TMDataBaseManager defaultManager] queryAddCategorysWithPaymentType:self.paymentType];
        for (TMAddCategory *category in results) {
            [_dataSource addObject:category];
        }
    }
    return _dataSource;
}
```
注：RLMResults是第三方数据库框架Realm中的一个类名
#### 2.6 示例：重写setter
```
- (void)setDataSource:(id<iCarouselDataSource>)dataSource
{
    if (_dataSource != dataSource)
    {
        _dataSource = dataSource;
        if (_dataSource)
        {
            [self reloadData];
        }
    }
}
```
#### 2.7 重写setter和getter导致的特别情况：
@property声明的属性，编译器是否会合成存取方法和成员变量有如下三种特别情况
- 若手动实现了setter方法，编译器就只会自动生成getter方法
- 若手动实现了getter方法，编译器就只会自动生成setter方法
- 若同时手动实现了setter和getter方法，编译器就不会自动生成不存在的***成员变量*** 。

#### 2.8 编译器自动实现的存取方法有什么弱点?
- @property只会生成最简单的getter/setter方法，而不会进行数据判断

#### 2.9 指定所生成的方法的方法名称
- getter=你定制的getter方法名称
- setter=你定义的setter方法名称(注意setter方法必须要有 :)
- 示例：
```
@property(getter=isMarried)BOOL married;
```
通常BOOL类型的属性的getter方法要以is开头

#### 2.10 继承特性：
  - 父类声明在***头文件*** 中的属性，子类无法继承这些属性声明的实例变量，只能看到属性自动生成的存取方法。


## 3. 修饰词：@synthesize 与 @dynamic
> 修饰词：告诉编译器是否或怎样自动给属性生成存取方法
---
@property有两个对应的修饰词，一个是@synthesize，一个是@dynamic。如果@synthesize和@dynamic都没写，那么默认的就是`@syntheszie var = _var;`。 显然，这两个修饰的功能是互斥的。

#### 3.1 @synthesize 与 @dynamic
###### 3.1.1 位置
@dynamic或者@synthesize，写在.m文件的@implementation中。
###### 3.1.2 功能区分
理解这两个修饰词的功能，可以先看看这两个单词的意思。
-  synthesize 与 dynamic 英文意思
    - synthesize  ['sɪnθəsaɪz]  v. 合成；综合
    -  dynamic  [daɪ'næmɪk]  adj. 动态的；动力的；动力学的；有活力的
###### 3.1.3 单词区分
 **单词混淆**: 多线程的概念里面有个关键词@synchronized跟@synthesize容易视觉混淆，这里也看看两个单词的意思。
- synthesize 与 synchronized 单词比较
  - synthesize  ['sɪnθəsaɪz]  v. 合成；综合
  - synchronized  ['sɪŋkrənaɪzd] adj. 同步的；同步化的 v. 使协调（synchronize的过去分词）；同时发生；校准
#### 3.2 @synthesize 
###### 3.2.1 介绍
定义属性后，编译器会自动编写访问这些属性所需的方法，此过程叫做***自动合成*** (autosynthesis)。需要强调的是，这个过程由编译器在编译期执行，所以编辑器里看不到这些“合成方法”(synthesized method)的源代码。除了生成方法代码 getter、setter 之外，编译器还要自动向类中添加适当类型的实例变量，并且在属性名前面加下划线，以此作为实例变量的名字。
###### 3.2.2 用法
- 多个属性可以通过一行@synthesize搞定,多个属性之间用逗号连接
- 可以在类的实现代码里通过@synthesize语法来指定**实例变量**的名字。
```
@synthesize name = realName；
```
对于上面的实例变量则为生成的是realName而不是name，方法也对应改变。定义的实例变量是根据`@synthesize name = realName;`来定的。用的比较多的情况是：
```
@synthesize name = _name;
```
上述代码的意思是，在@property 声明的name，在`setter/getter`方法中使用`NSObject * _name;`这个实例变量来赋值与返回。
###### 3.2.3 三种写法比较
```
@synthesize age = _age;
```
- setter和getter实现中会访问成员变量_age
- 如果成员变量_age不存在，就会自动生成一个@private的成员变量_age
```
@synthesize age;//等效下面
@synthesize age = age;
```
- setter和getter实现中会访问@synthesize后同名成员变量age
- 如果成员变量age不存在，就会自动生成一个@private的成员变量age
###### 3.2.4 用法场景
当你在子类中重载了父类中的属性，你必须使用@synthesize来手动合成实例变量。

#### 3.3 @dynamic
###### 3.3.1 介绍
- @dynamic告诉编译器：属性的setter与getter方法由用户自己实现，不自动生成。（当然对于readonly的属性只需提供getter即可）。
###### 3.3.2 崩溃
- 假如一个属性被声明为@dynamic var，然后你没有提供@setter方法和@getter方法，编译的时候没问题，但是当程序运行到instance.var = someVar，由于缺setter方法会导致程序崩溃；或者当运行到 someVar = var时，由于缺getter方法同样会导致崩溃。
- 编译时没问题，运行时才执行相应的方法，这就是所谓的***动态绑定***。

#### 3.4. 注意的事
- @synthesize 仅仅是一个 ***clang*** 的 Objective-C 语言扩展 (autosynthesis of properties), 然后clang恰好是 Xcode 的默认编译器. 
- 如果编译器换成了 ***gcc***, 那么这个特性也就不复存在了。

## 4. 其它修饰词
---
很多人讲这些修饰词的时候，喜欢从字面或者定义的角度介绍它们间的区别。这篇文章，我们从修饰词对setter方法的影响直接展示区别。
> retain、assign、copy、weak、strong、nonatomic、atomic、readonly、readwrite

假设为了修饰一个属性nameStr，代码如下：
```
#import <UIKit/UIKit.h>
@interface CMViewController ()
@property (nonatomic, strong) NSString *nameStr;
@end
```
其中，当括号内的修饰词(nonatomic, strong)换成下面各种修饰词的时候，分别分析一下setter方法（有些修饰词修饰字符串并不合适，但这里仅为分析区别）。
#### 4.1 assign
###### 4.1.1 基本特性
- assign (默认)：直接赋值，不更改引用计数。

###### 4.1.2 对setter的影响
- assign修饰词对setter的影响：
```
- (void) setName:(NSString *)newValue{
  nameStr = newValue;
}
```
###### 4.1.3 使用场景
- 常用于基本数据类型（NSInteger）和C数据类型（int、float、double、char以及id类型。
- 这个修饰符不会牵涉到内存管理，但是如果是对象类型，可能会导致内存泄漏或者EXC_BAD_ACCESS错误。
- 除了assign以外的其他修饰符，是必须用于修饰OC对象的。
###### 4.1.4 危险场景
- assign修饰的对象销毁后不会把该对象的指针置nil。对象已经被销毁，但指针还在痴痴的指向它，这就成了野指针，这是比较危险的。所以assign修饰的OC属性是非常危险的，比如，一些老的第三方框架用assign修饰的delegate属性经常会导致崩溃。

#### 4.2 retain
###### 4.2.1 基本特性
- retain: 指针拷贝。指针拷贝后，地址不变，内容不变，引用计数器加1。

###### 4.2.2 对setter的影响
- retain修饰词对setter的影响：
```
- (void) setName:(NSString *)newValue{
  if (nameStr !=newValue){
     [nameStr release]
     nameStr = [newValue retain];
  }
}
```
###### 4.2.3 使用场景
* 适用NSObject和其子类。MRC下，用于修饰多数的OC类型的对象。ARC下，一般功能被strong取代。
* 不能用于基本的数据类型或者Core Foundation的对象(retain操作会使对象的引用计数器加1,但是基本的数据类型或者Core Foundation对象压根就没有引用计数器，所以无法进行retain操作。换言之:基本数据类型或者CF不是指针，不是指针就无法进行retain操作。对象即指针嘛)。

#### 4.3 copy
###### 4.3.1 基本特点
- copy是内容拷贝。释放旧对象，然后建立一个索引计数为1的对象。
- strong修饰的属性在赋值时不会调用copy，而copy修饰的属性在赋值相当于自动多调用了一次copy方法。
###### 4.3.2 对setter的影响
- copy修饰词对setter的影响：
```
- (void) setName:(NSString *)newValue{
  if (nameStr !=newValue){
     [nameStr release]
     nameStr = [newValue copy];
  }
}
```
###### 4.3.3 使用场景
copy的使用场景为，实现了NSCopying protocol的类，我想获取它的值，但是我又不想在原对象上改变，于是深赋值一份新的值给你，让你来自由操作。
* 用于修饰block。
* 用于含有可深拷贝的mutable子类的类，如NSString，NSArray，NSSet，NSDictionary，NSData的，NSCharacterSet，NSIndexSet。
* 但是NSMutableArray这样的不可以，Mutable的不能用copy，不然初始化会有问题。


#### 4.4 weak
weak、strong是ARC出现后才出现的概念，但这并不代表weak、strong这两个修饰都不能在MRC模式下使用。事实上，strong在MRC模式依旧可使用。
###### 4.4.1 基本特性
- weak 用来修饰强引用的属性，类似于对应原来的assign。
- weak是一种弱引用,并不会使对象的引用计数加1,可以避免循环引用的问题。
- 不保留传入的对象。如果该对象被释放，那么相应的实例变量会被自动赋为nil，不会变为悬空指针(也称野指针)。悬空指针指向的是不再存在的对象，向悬空指针发送消息通常会导致程序崩溃。
###### 4.4.2 两种模式下
- MRC模式
  - weak: MRC模式下无法使用
- ARC模式
  - weak: 弱引用，不会使对象的引用计数器加1。
###### 4.4.3 与assign的区别
- weak修饰的对象销毁的时候，指针会自动设置为nil。而assign不会。
- assign可以用于非OC对象，而weak必须用于OC对象。
###### 4.4.4 使用场景

* 用于修饰UI控件。
* UI控件拖到xib/Storyboard后，系统自动为控件赋了strong，所以拖到代码就用weak就行了。
* 代理属性。`@property (nonatomic, weak) id delegate;  // 修饰代理属性`
###### 4.5.5 对setter的影响
* weak修饰词对setter的影响：假设nameStr和newValue都是用weak修饰的属性
```
[nameStr release]
nameStr = newValue;
```

#### 4.5 strong
###### 4.5.1 基本特性
strong 用来修饰强引用的属性，类似于对应原来的retain。
###### 4.5.2 两种模式
- MRC模式
  - strong: 与retain等价
- ARC模式
   - strong: 强引用(它使对象的引用计数加1)
###### 4.5.3 使用场景
* 当要保住某个对象的命,让这个对象可以用于其他的方法时(即在某段时间内要经常用到这个对象,又不想每次用到这个对象都要重新alloc),此时你要把这个对象变为强指针,即变为strong，让strong强引用着这个对象,使这个对象不会被释放。
* 用于修饰NSMutableArray，NSMutableDictionary等copy无法修饰的属性。
* 一般的指针变量默认就是strong类型的,所以我们对于strong变量不加__strong修饰。
```
NSString *name = self.nameField.text;  // 等价
__strong NSString *name = self.nameField.text;  // 等价
```
###### 4.5.4 对setter的影响
* strong修饰词对setter的影响：假设nameStr和newValue都是用strong修饰的属性
```
[newValue retain]
[nameStr release]
nameStr = newValue;
```
#### 4.6 读写属性
读写性修饰符——readwrite、readonly



######  4.6.1 readwrite
* readwrite(默认):  可读可写(系统自动创建getter 和 setter 方法)
######  4.6.1 readonly
* readonly: 只读,系统只会生成 getter方法

#### 4.7 原子属性
######  4.7.1 atomic
* 1.原子属性，声明的属性默认就是atomic.所以底层默认为属性的setter方法加锁，目的就是防止多(条)线程访问同一个内存地址，造成数据错误。
* 2.在多线程环境下,原子操作非常有必要,因为它能提供多线程安全，如果没有原子操作,可能引起异常。--->线程保护
* 3.需要消耗大量的资源。
######  4.7.2 nonatomic
* 1.非原子属性，不会为setter方法加锁。
* 2.没有涉及多线程的编程时,用nonatomic。
* 3.不会消耗大量的资源，所以会提高性能。

## 5. @property的几个例子
```
@property (nonatomic, readonly) CGFloat itemWidth;
@property (nonatomic, assign) double brightness;
@property (nonatomic, assign, getter=isOpenMenu) BOOL openMenu;
@property (nonatomic, strong) UILabel *newsBooksLabel;
@property (nonatomic, strong) UITableView *tableView;
@property (nonatomic, weak) IBOutlet UIButton *nextButton;
@property (nonatomic, copy) void (^cancelBtnBlock)();
@property (nonatomic, weak) id<TMSideCellDelegate> sideCellDelegate;
```
## 6. 推荐阅读
- 深拷贝浅拷贝的区别
  - http://www.jianshu.com/p/59d39e614718
  - http://www.jianshu.com/p/4e5fde48fcda
  - http://www.jianshu.com/p/6b9f3a79cc34
- 修饰词与block引起的循环引用
  - http://www.jianshu.com/p/bb23544f72df
  - http://www.jianshu.com/p/701da54bd78c