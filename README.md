

文章大纲，便于浏览

1. [01-iOS程序启动过程](https://github.com/liberalisman/2018-Interview-Preparation#01-ios-app-startup-process)
2. [02-浅拷贝-深拷贝](https://github.com/liberalisman/2018-Interview-Preparation#02-shallowcopy-deepcopy)
3. [03-View的生命周期](https://github.com/liberalisman/2018-Interview-Preparation#03-view的生命周期)
4. [04-@property](https://github.com/liberalisman/2018-Interview-Preparation#04-property)
5. [05-事件传递和事件响应](https://github.com/liberalisman/iOS-Summary-Part1#05-事件传递和事件响应)
6. [06-KVC](https://github.com/liberalisman/iOS-Summary-Part1#06-kvc)
7. [07-KVO](https://github.com/liberalisman/iOS-Summary-Part1#07-kvo)
8. [08-iOS数据持久化方案](https://github.com/liberalisman/iOS-Summary-Part1#08-ios数据持久化方案)






## 01-iOS-App-startup-process

###一、启动完整过程

![](http://okhqmtd8q.bkt.clouddn.com/image/jpg/%E7%A8%8B%E5%BA%8F%E5%90%AF%E5%8A%A8%E6%B5%81%E7%A8%8B.png)

1.`main`函数

2.`UIApplicationMain`函数

* 创建`UIApplication`对象
* 创建`UIApplication的delegate`对象

3.`delegate`对象开始处理(监听)系统事件(没有storyboard)

* 程序启动完毕的时候, 就会调用代理的:`didFinishLaunchingWithOptions:`方法
* 在`application:didFinishLaunchingWithOptions`:中创建`UIWindow` 创建和设置`UIWindow`的`rootViewController`
* 显示窗口

4.根据`Info.plist`获得最主要`storyboard`的文件名,加载最主要的`storyboard`(有storyboard)

* 创建`UIWindow`
* 创建和设置`UIWindow`的`rootViewController`
* 显示窗口


###二、程序启动原理


1.`main`函数中执行了一个`UIApplicationMain`这个函数

```objc
int UIApplicationMain(int argc, char *argv[], NSString *principalClassName, NSString *delegateClassName);

argc、argv：直接传递给UIApplicationMain进行相关处理即可
```

2.`principalClassName`：指定应用程序类名（app的象征），该类必须是`UIApplication`(或子类)。如果为`nil`,则用`UIApplication`类作为默认值

3.`delegateClassName`：指定应用程序的代理类，该类必须遵守`UIApplicationDelegate`协议

4.`UIApplicationMain`函数会根据`principalClassName`创建`UIApplication`对象，根据`delegateClassName`创建一个`delegate`对象，并将该`delegate`对象赋值给`UIApplication`对象中的`delegate`属性

5.接着会建立应用程序的`Main Runloop`（事件循环），进行事件的处理(首先会在程序完毕后调用`delegate`对象的`application:didFinishLaunchingWithOptions`:方法)

程序正常退出时`UIApplicationMain`函数才返回

```objc
int main(int argc, char * argv[]){ @autoreleasepool { 

/**
* argc: 系统或者用户传入的参数个数
* argv: 系统或者用户传入的实际参数 
* 1.根据传入的第三个参数创建UIApplication对象 
* 2.根据传入的第四个产生创建UIApplication对象的代理
* 3.设置刚刚创建出来的代理对象为UIApplication的代理 
* 4.开启一个事件循环 
**/ 
return UIApplicationMain(argc, argv, @"UIApplication", @"YYAppDelegate"); }}

```

启动与代理：
![](http://okhqmtd8q.bkt.clouddn.com/image/jpg/%E7%A8%8B%E5%BA%8F%E5%90%AF%E5%8A%A8%E4%B8%8E%E4%BB%A3%E7%90%86.png)

## 02-ShallowCopy-DeepCopy

简要总结一下什么是浅拷贝，什么是深拷贝

> 深拷贝就是内容拷贝（分为单层拷贝、完全拷贝），深拷贝的之所以分为两类，主要是针对集合类

> 浅拷贝就是指针拷贝

####一.系统对象的 copy/mutableCopy
 

```objc
NSString *string = @"LiMing";
    
NSString *copyString = [string copy];
    
NSString *mutableString = [string mutableCopy];
    
NSLog(@"string = %p",string);
    
NSLog(@"copyString = %p",copyString);
    
NSLog(@"mutableString = %p ",mutableString);

结论：
1.string 和 copyString 他们只是二个不同的指针，指向内存中的同一块地址，copy 只是指针复制
2.string 和 mutableString 打印出来的地址不同，是因为两个指针指向的地址本就不同，mutableCopy 是内容复制

注意：其他对象 NSArray 、NSMutableArray 、NSDictionary 、NSMutableDictionary 一样适用
```

规律可以从这张图看出来

![](http://okhqmtd8q.bkt.clouddn.com/image/jpg/%E6%B7%B1%E6%8B%B7%E8%B4%9D-%E6%B5%85%E6%8B%B7%E8%B4%9D-01)

![](http://okhqmtd8q.bkt.clouddn.com/image/jpg/%E6%B7%B1%E6%8B%B7%E8%B4%9D-%E6%B5%85%E6%8B%B7%E8%B4%9D-02)

####二.自定义对象实现 Copy-MutableCopy

* copy

```objc
GZQPerson *person = [[GZQPerson alloc] init];
person.age = 20;
person.name = @"GZQ";
GZQPerson *copyP = [person copy];  // 这里崩溃
```

崩溃：
![](http://okhqmtd8q.bkt.clouddn.com/image/jpg/%E6%B7%B1%E6%8B%B7%E8%B4%9D-%E6%B5%85%E6%8B%B7%E8%B4%9D-03.png)

看崩溃信息GZQPerson应该先实现：

```objc
- (id)copyWithZone:(NSZone *)zone;
```
测试：

```objc
#import "GZQPerson.h"

@interface GZQPerson ()<NSCopying,NSMutableCopying>

@end

@implementation GZQPerson

- (id)copyWithZone:(NSZone *)zone {

    GZQPerson *person = [[[self class] allocWithZone:zone] init];
    person.age = self.age;
    person.name = self.name;
    return person;
}

- (id)mutableCopyWithZone:(NSZone *)zone {

    GZQPerson *person = [[[self class] allocWithZone:zone] init];
    person.age = self.age;
    person.name = self.name;
    return person;
}

@end


```



```objc
#import "ViewController.h"
#import "GZQPerson.h"

@interface ViewController ()

@end

@implementation ViewController

- (void)viewDidLoad {
    
    [super viewDidLoad];

    GZQPerson *person = [[GZQPerson alloc] init];
    person.age = 20;
    person.name = @"GZQ";
    GZQPerson *copyP = [person copy];
    
    NSLog(@"copyP=%p",copyP);
    NSLog(@"person=%p",person);
    NSLog(@"person=%p",copyP.name);
    NSLog(@"person=%p",person.name);
    
}
@end
```

![](http://okhqmtd8q.bkt.clouddn.com/image/jpg/%E6%B7%B1%E6%8B%B7%E8%B4%9D-%E6%B5%85%E6%8B%B7%E8%B4%9D-04.png)

可以看出虽然指针的地址不同，但是存储的地址是一致的。



####三.copy 本质

`property copy` 实际上就对name干了这个：

```objc
#import <Foundation/Foundation.h>

property copy 实际上就对name干了这个：

- (void)setName:(NSString *)name
{
    _name = [name copy];
}
```

`strong`是不执行`Copy`操作的

```objc
@property (nonatomic, strong) NSString *name;

NSMutableString *string = [NSMutableString stringWithFormat:@"深拷贝-浅拷贝"];

GZQPerson *person = [[GZQPerson alloc] init];
person.name = string;

// 可以改变person.name的值，因为其内部没有生成新的对象
[string appendString:@"LALALA"];

NSLog(@"name = %@", person.name);
```

####四.集合类 Copy MutableCopy 操作

> 单层深复制，也就是我们经常说的深复制，我这里说的单层深复制是对于集合类所说的(即NSArray,NSDictionary,NSSet)，单层深复制指的是只复制了该集合类的最外层，里边的元素没有复制，(即这两个集合类的地址不一样，但是两个集合里所存储的元素的地址是一样的)
> 
完全复制，指的是完全复制整个集合类，也就是说两个集合地址不一样，里边所存储的元素地址也不一样


实现多层完全拷贝也很简单

```objc
 NSArray *copyArray = [[NSArray alloc] initWithArray:array copyItems:YES];  // 完全复制
```
## 03-View的生命周期

* 读懂这一张图即可
![](http://okhqmtd8q.bkt.clouddn.com/image/jpg/View%E7%94%9F%E5%91%BD%E5%91%A8%E6%9C%9F.png)



## 04-@property
> @property 的本质是什么？ivar、getter、setter 是如何生成并添加到这个类中的


@property 的本质.

```objc
@property = ivar + getter + setter;
```

下面解释下：

> “属性” (property)有两大概念：ivar（实例变量）、存取方法（access method ＝ getter + setter）。

>“属性” (property)作为 Objective-C 的一项特性，主要的作用就在于封装对象中的数据。 Objective-C 对象通常会把其所需要的数据保存为各种实例变量。实例变量一般通过“存取方法”(access method)来访问。其中，“获取方法” (getter)用于读取变量值，而“设置方法” (setter)用于写入变量值。这个概念已经定型，并且经由“属性”这一特性而成为 Objective-C 2.0 的一部分。 而在正规的 Objective-C 编码风格中，存取方法有着严格的命名规范。 正因为有了这种严格的命名规范，所以 Objective-C 这门语言才能根据名称自动创建出存取方法。其实也可以把属性当做一种关键字，其表示:

编译器会自动写出一套存取方法，用以访问给定类型中具有给定名称的变量。 所以你也可以这么说：

```objc
@property = getter + setter;
```

例如下面这个类：

```objc
@interface Person : NSObject
@property NSString *firstName;
@property NSString *lastName;
@end
```
上述代码写出来的类与下面这种写法等效：

```objc
@interface Person : NSObject
- (NSString *)firstName;
- (void)setFirstName:(NSString *)firstName;
- (NSString *)lastName;
- (void)setLastName:(NSString *)lastName;
@end
```

`property`在`runtime`中是`objc_property_t`定义如下:

```objc
typedef struct objc_property *objc_property_t;
```

而`objc_property`是一个结构体，包括`name`和`attributes`，定义如下：

```objc
struct property_t {
    const char *name;
    const char *attributes;
};
```
而`attributes`本质是`objc_property_attribute_t`，定义了`property`的一些属性，定义如下：

```objc
/// Defines a property attribute
typedef struct {
    const char *name;           /**< The name of the attribute */
    const char *value;          /**< The value of the attribute (usually empty) */
} objc_property_attribute_t;
```


>而attributes的具体内容是什么呢？其实，包括：类型，原子性，内存语义和对应的实例变量。

例如：我们定义一个`string`的`property`

```objc
@property (nonatomic, copy) NSString *string;
```

通过 `property_getAttributes(property)`获取到`attributes`并打印出来之后的结果为

```objc
T@"NSString",C,N,V_string
```

其中`T`就代表类型，可参阅`Type Encodings`，`C`就代表`Copy`，`N`代表`nonatomic`，`V`就代表对于的实例变量。

>ivar、getter、setter 是如何生成并添加到这个类中的?

**“自动合成”( autosynthesis)**

>完成属性定义后，编译器会自动编写访问这些属性所需的方法，此过程叫做“自动合成”(autosynthesis)。需要强调的是，这个过程由编译 器在编译期执行，所以编辑器里看不到这些“合成方法”(synthesized method)的源代码。除了生成方法代码 getter、setter 之外，编译器还要自动向类中添加适当类型的实例变量，并且在属性名前面加下划线，以此作为实例变量的名字。在前例中，会生成两个实例变量，其名称分别为 _firstName 与 _lastName。也可以在类的实现代码里通过@synthesize 语法来指定实例变量的名字.

```objc
@implementation Person
@synthesize firstName = _myFirstName;
@synthesize lastName = _myLastName;
@end
```

**我为了搞清属性是怎么实现的,曾经反编译过相关的代码,他大致生成了五个东西**

```objc
1. OBJC_IVAR_$类名$属性名称 ：该属性的“偏移量” (offset)，这个偏移量是“硬编码” (hardcode)，表示该变量距离存放对象的内存区域的起始地址有多远。
2. setter 与 getter 方法对应的实现函数
3. ivar_list ：成员变量列表
4. method_list ：方法列表
5. prop_list ：属性列表
也就是说我们每次在增加一个属性,系统都会在 ivar_list 中添加一个成员变量的描述,在 method_list 中增加 setter 与 getter 方法的描述,在属性列表中增加一个属性的描述,然后计算该属性在对象中的偏移量,然后给出 setter 与 getter 方法对应的实现,在 setter 方法中从偏移量的位置开始赋值,在 getter 方法中从偏移量开始取值,为了能够读取正确字节数,系统对象偏移量的指针类型进行了类型强转.
```



**属性可以拥有的特质分为四类:**

*  原子性--- nonatomic 特质,在默认情况下，由编译器合成的方法会通过锁定机制确保其原子性(atomicity)。如果属性具备 nonatomic 特质，则不使用自旋锁。请注意，尽管没有名为“atomic”的特质(如果某属性不具备 nonatomic 特质，那它就是“原子的” ( atomic) )，但是仍然可以在属性特质中写明这一点，编译器不会报错。若是自己定义存取方法，那么就应该遵从与属性特质相符的原子性。

*  读/写权限---readwrite(读写)、readonly (只读)

*  内存管理语义---assign、strong、 weak、unsafe_unretained、copy
*  方法名---getter=<name> 、setter=<name>

**getter=<name>的样式：**

```objc
@property (nonatomic, getter=isOn) BOOL on;
     
(`setter=`这种不常用，也不推荐使用。故不在这里给出写法。）
```

**setter=<name>一般用在特殊的情境下，比如**：

>在数据反序列化、转模型的过程中，服务器返回的字段如果以 init 开头，所以你需要定义一个 init 开头的属性，但默认生成的 setter 与 getter 方法也会以 init 开头，而编译器会把所有以 init 开头的方法当成初始化方法，而初始化方法只能返回 self 类型，因此编译器会报错。

**这时你就可以使用下面的方式来避免编译器报错：**

```objc
@property(nonatomic, strong, getter=p_initBy, setter=setP_initBy:)NSString *initBy;
```

**另外也可以用关键字进行特殊说明，来避免编译器报错**：

```objc
@property(nonatomic, readwrite, copy, null_resettable) NSString *initBy;

- (NSString *)initBy __attribute__((objc_method_family(none)));

1. 不常用的：nonnull,null_resettable,nullable

注意：很多人会认为如果属性具备 nonatomic 特质，则不使用 “同步锁”。其实在属性设置方法中使用的是自旋锁，自旋锁相关代码如下：

static inline void reallySetProperty(id self, SEL _cmd, id newValue, ptrdiff_t offset, bool atomic, bool copy, bool mutableCopy)
{
    if (offset == 0) 
    {
        object_setClass(self, newValue);
        return;
    }

    id oldValue;
    id *slot = (id*) ((char*)self + offset);

    if (copy) 
    {
        newValue = [newValue copyWithZone:nil];
    } 
    else if (mutableCopy) 
    {
        newValue = [newValue mutableCopyWithZone:nil];
    } 
    else 
    {
        if (*slot == newValue) return;
        newValue = objc_retain(newValue);
    }

    if (!atomic) 
    {
        oldValue = *slot;
        *slot = newValue;
    } 
    else 
    {
        spinlock_t& slotlock = PropertyLocks[slot];
        slotlock.lock();
        oldValue = *slot;
        *slot = newValue;        
        slotlock.unlock();
    }

    objc_release(oldValue);
}

void objc_setProperty(id self, SEL _cmd, ptrdiff_t offset, id newValue, BOOL atomic, signed char shouldCopy) 
{
    bool copy = (shouldCopy && shouldCopy != MUTABLE_COPY);
    bool mutableCopy = (shouldCopy == MUTABLE_COPY);
    reallySetProperty(self, _cmd, newValue, offset, atomic, copy, mutableCopy);
}
```

## 05-事件传递和事件响应

这部分知识如果自己总结，篇幅较长。可以参考[以下文章](http://www.jianshu.com/p/2e074db792ba)


## 06-KVC

**Key-Value Coding (KVC)**


> KVC（Key-value coding）键值编码，单看这个名字可能不太好理解。其实翻译一下就很简单了，就是指iOS的开发中，可以允许开发者通过Key名直接访问对象的属性，或者给对象的属性赋值。而不需要调用明确的存取方法。这样就可以在运行时动态在访问和修改对象的属性。而不是在编译时确定，这也是iOS开发中的黑魔法之一。很多高级的iOS开发技巧都是基于KVC实现的。目前网上关于KVC的文章在非常多，有的只是简单地说了下用法，有的讲得深入但是在使用场景和最佳实践没有说明，我写下这遍文章就是给大家详解一个最完整最详细的KVC。

**KVC在iOS中的定义**

无论是`Swift`还是`Objective-C`，`KVC`的定义都是对`NSObject`的扩展来实现的(`Objective-C`中有个显式的`NSKeyValueCoding`类别名，而`Swift`没有，也不需要)所以对于所有继承了`NSObject`在类型，都能使用`KVC`(一些纯`Swift`类和结构体是不支持`KVC`的)，下面是`KVC`最为重要的四个方法

```objc
- (nullable id)valueForKey:(NSString *)key;                          //直接通过Key来取值
- (void)setValue:(nullable id)value forKey:(NSString *)key;          //通过Key来设值
- (nullable id)valueForKeyPath:(NSString *)keyPath;                  //通过KeyPath来取值
- (void)setValue:(nullable id)value forKeyPath:(NSString *)keyPath;  //通过KeyPath来设值
```


当然`NSKeyValueCoding`类别中还有其他的一些方法，下面列举一些

```objc
+ (BOOL)accessInstanceVariablesDirectly;
//默认返回YES，表示如果没有找到Set<Key>方法的话，会按照_key，_iskey，key，iskey的顺序搜索成员，设置成NO就不这样搜索
- (BOOL)validateValue:(inout id __nullable * __nonnull)ioValue forKey:(NSString *)inKey error:(out NSError **)outError;
//KVC提供属性值确认的API，它可以用来检查set的值是否正确、为不正确的值做一个替换值或者拒绝设置新值并返回错误原因。
- (NSMutableArray *)mutableArrayValueForKey:(NSString *)key;
//这是集合操作的API，里面还有一系列这样的API，如果属性是一个NSMutableArray，那么可以用这个方法来返回
- (nullable id)valueForUndefinedKey:(NSString *)key;
//如果Key不存在，且没有KVC无法搜索到任何和Key有关的字段或者属性，则会调用这个方法，默认是抛出异常
- (void)setValue:(nullable id)value forUndefinedKey:(NSString *)key;
//和上一个方法一样，只不过是设值。
- (void)setNilValueForKey:(NSString *)key;
//如果你在SetValue方法时面给Value传nil，则会调用这个方法
- (NSDictionary<NSString *, id> *)dictionaryWithValuesForKeys:(NSArray<NSString *> *)keys;
//输入一组key,返回该组key对应的Value，再转成字典返回，用于将Model转到字典。
```

上面的这些方法在碰到特殊情况或者有特殊需求还是会用到的，所以也是可以了解一下。后面的代码示例会有讲到其中的一些方法。
同时苹果对一些容器类比如NSArray或者NSSet等，KVC有着特殊的实现。建议有基础的或者英文好的开发者直接去看苹果的官方文档，相信你会对KVC的理解更上一个台阶。

**KVC是怎么寻找Key的**

KVC是怎么使用的，我相信绝大多数的开发者都很清楚，我在这里就不再写简单的使用KVC来设值和取值的代码了，首页我们来探讨KVC在内部是按什么样的顺序来寻找key的。
当调用`setValue：`属性值 `forKey：``@”name“`的代码时，底层的执行机制如下：


* 程序优先调用`set<Key>:`属性值方法，代码通过`setter`方法完成设置。注意，这里的`<key>`是指成员变量名，首字母大清写要符合`KVC`的全名规则，下同

* 如果没有找到`setName：`方法，`KVC`机制会检查`+ (BOOL)accessInstanceVariablesDirectly`方法有没有返回`YES`，默认该方法会返回`YES`，如果你重写了该方法让其返回`NO`的话，那么在这一步KVC会执行`setValue：forUNdefinedKey：`方法，不过一般开发者不会这么做。所以KVC机制会搜索该类里面有没有名为`_<key>`的成员变量，无论该变量是在类接口部分定义，还是在类实现部分定义，也无论用了什么样的访问修饰符，只在存在以`_<key>`命名的变量，`KVC`都可以对该成员变量赋值。

* 如果该类即没有`set<Key>：`方法，也没有`_<key>`成员变量，`KVC`机制会搜索`_is<Key>`的成员变量，

* 和上面一样，如果该类即没有`set<Key>：`方法，也没有`_<key>`和`_is<Key>`成员变量，`KVC`机制再会继续搜索`<key>`和`is<Key>`的成员变量。再给它们赋值。

* 如果上面列出的方法或者成员变量都不存在，系统将会执行该对象的`setValue：forUNdefinedKey：`方法，默认是抛出异常。

如果开发者想让这个类禁用`KVC`里，那么重写`+ (BOOL)accessInstanceVariablesDirectly`方法让其返回NO即可，这样的话如果`KVC`没有找到`set<Key>:`属性名时，会直接用`setValue：forUNdefinedKey：`方法。


## 07-KVO

`KVO`，全称为`Key-Value Observing`，是iOS中的一种设计模式，用于检测对象的某些属性的实时变化情况并作出响应。当应用场景比较复杂时，多个地方存在crash的危险。

首先，假设我们的目标是在一个`UITableViewController`内对`tableview`的`contentOffset`进行实时监测，很容易地使用`KVO`来实现为。

在初始化方法中加入：

```OBJC
[_tableView addObserver:self forKeyPath:@"contentOffset" options:NSKeyValueObservingOptionNew context:nil];

// 在dealloc中移除KVO监听：
[_tableView removeObserver:self forKeyPath:@"contentOffset" context:nil];

// 添加默认的响应回调方法：
- (void)observeValueForKeyPath:(NSString *)keyPath 
                      ofObject:(id)object
                        change:(NSDictionary *)change 
                       context:(void *)context
{
    [self doSomethingWhenContentOffsetChanges];
}
```

好了，`KVO`实现就到此完美结束了，开玩笑，肯定没这么简单的，这样的代码太粗糙了，当你在`controller`中添加多个`KVO`时，所有的回调都是走同上述函数，那就必须对触发回调函数的来源进行判断。判断如下：

```objc
- (void)observeValueForKeyPath:(NSString *)keyPath 
                      ofObject:(id)object
                        change:(NSDictionary *)change 
                       context:(void *)context
{
    if (object == _tableView && [keyPath isEqualToString:@"contentOffset"]) 
    {
        [self doSomethingWhenContentOffsetChanges];
    }
}
```

你以为这样就结束了吗？答案是否定的！我们假设当前类(在例子中为`UITableViewController`)还有父类，并且父类也有自己绑定了一些其他`KVO`呢？我们看到，上述回调函数体中只有一个判断，如果这个`if`不成立，这次`KVO`事件的触发就会到此中断了。但事实上，若当前类无法捕捉到这个`KVO`，那很有可能是在他的`superClass`，或者`super-superClass...`中，上述处理砍断了这个链。合理的处理方式应该是这样的：

```objc
- (void)observeValueForKeyPath:(NSString *)keyPath 
                      ofObject:(id)object
                        change:(NSDictionary *)change 
                       context:(void *)context
{
    if (object == _tableView && [keyPath isEqualToString:@"contentOffset"]) 
    {
        [self doSomethingWhenContentOffsetChanges];
    } 
    else 
    {
        [super observeValueForKeyPath:keyPath ofObject:object change:change context:context];
    }
}
```

这样就结束了吗？答案仍旧是否定的。潜在的问题有可能出现在`dealloc`中对`KVO`的注销上。`KVO`的一种缺陷(其实不能称为缺陷，应该称为特性)是，当对同一个`keypath`进行两次`removeObserver`时会导致程序`crash`，这种情况常常出现在父类有一个`kvo`，父类在`dealloc`中`remove`了一次，子类又`remove`了一次的情况下。不要以为这种情况很少出现！当你封装`framework`开源给别人用或者多人协作开发时是有可能出现的，而且这种`crash`很难发现。不知道你发现没，目前的代码中`context`字段都是`nil`，那能否利用该字段来标识出到底`kvo`是`superClass`注册的，还是`self`注册的？

回答是可以的。我们可以分别在父类以及本类中定义各自的`context`字符串，比如在本类中定义`context`为`@"ThisIsMyKVOContextNotSuper"`;然后在`dealloc`中`remove observer`时指定移除的自身添加的`observer`。这样iOS就能知道移除的是自己的`kvo`，而不是父类中的`kvo`，避免二次`remove`造成`crash`。

## 08-iOS数据持久化方案

### 存储方案
* plist文件（属性列表）
* preference（偏好设置）
* NSKeyedArchiver（归档）
* SQLite 3
* CoreData

### 沙盒

> iOS程序默认情况下只能访问程序自己的目录，这个目录被称为“沙盒”。

#### 1.结构

沙盒的目录结构如下：

```objc
"应用程序包"
Documents
Library
    Caches
    Preferences
tmp
```

#### 2.目录特性

> 虽然沙盒中有这么多文件夹，但是每个文件夹都不尽相同，都有各自的特性。所以在选择存放目录时，一定要认真选择适合的目录。

"应用程序包": 这里面存放的是应用程序的**源文件**，包括**资源文件**和**可执行文件**。

* Documents: 最常用的目录，iTunes同步该应用时会同步此文件夹中的内容，适合存储重要数据。
 
```objc
  NSString *path = [[NSBundle mainBundle] bundlePath];
  NSLog(@"%@", path);
```

* Library/Caches: iTunes不会同步此文件夹，适合存储体积大，不需要备份的非重要数据。

```objc
  NSString *path = NSSearchPathForDirectoriesInDomains(NSDocumentDirectory, NSUserDomainMask, YES).firstObject;
  NSLog(@"%@", path);
```

* Library/Preferences: iTunes同步该应用时会同步此文件夹中的内容，通常保存应用的设置信息。

```objc
  NSString *path = NSSearchPathForDirectoriesInDomains(NSCachesDirectory, NSUserDomainMask, YES).firstObject;
  NSLog(@"%@", path);
```

* tmp: iTunes不会同步此文件夹，系统可能在应用没运行时就删除该目录下的文件，所以此目录适合保存应用中的一些临时文件，用完就删除。

```objc
  NSString *path = NSTemporaryDirectory();
  NSLog(@"%@", path);
```

### plist文件

> plist文件是将某些特定的类，通过XML文件的方式保存在目录中。

可以被序列化的类型只有如下几种：

```objc
NSArray;
NSMutableArray;
NSDictionary;
NSMutableDictionary;
NSData;
NSMutableData;
NSString;
NSMutableString;
NSNumber;
NSDate;
```

#### 1.获得文件路径

```objc
NSString *path = NSSearchPathForDirectoriesInDomains(NSDocumentDirectory, NSUserDomainMask, YES).firstObject;
NSString *fileName = [path stringByAppendingPathComponent:@"123.plist"];
```
    
#### 2.存储

```objc
NSArray *array = @[@"123", @"456", @"789"];
[array writeToFile:fileName atomically:YES];
```

#### 3.读取

```objc
NSArray *result = [NSArray arrayWithContentsOfFile:fileName];
NSLog(@"%@", result);
```

#### 4.注意

```objc
// 只有以上列出的类型才能使用plist文件存储。
// 存储时使用writeToFile: atomically:方法。 其中atomically表示是否需要先写入一个辅助文件，再把辅助文件拷贝到目标文件地址。这是更安全的写入文件方法，一般都写YES。
// 读取时使用arrayWithContentsOfFile:方法。
```

### Preference

#### 1.使用方法

```objc
//1.获得NSUserDefaults文件
NSUserDefaults *userDefaults = [NSUserDefaults standardUserDefaults];

//2.向文件中写入内容
[userDefaults setObject:@"AAA" forKey:@"a"];
[userDefaults setBool:YES forKey:@"sex"];
[userDefaults setInteger:21 forKey:@"age"];

//2.1立即同步
[userDefaults synchronize];

//3.读取文件
NSString *name = [userDefaults objectForKey:@"a"];
BOOL sex = [userDefaults boolForKey:@"sex"];
NSInteger age = [userDefaults integerForKey:@"age"];
NSLog(@"%@, %d, %ld", name, sex, age);
```

#### 2.注意

```objc
// 偏好设置是专门用来保存应用程序的配置信息的，一般不要在偏好设置中保存其他数据。
// 如果没有调用synchronize方法，系统会根据I/O情况不定时刻地保存到文件中。所以如果需要立即写入文件的就必须调用synchronize方法。
// 偏好设置会将所有数据保存到同一个文件中。即preference目录下的一个以此应用包名来命名的plist文件。
```


### NSKeyedArchiver

> 归档在iOS中是另一种形式的序列化，只要遵循了NSCoding协议的对象都可以通过它实现序列化。由于决大多数支持存储数据的Foundation和Cocoa Touch类都遵循了NSCoding协议，因此，对于大多数类来说，归档相对而言还是比较容易实现的。

#### 1.遵循NSCoding协议

> NSCoding协议声明了两个方法，这两个方法都是必须实现的。一个用来说明如何将对象编码到归档中，另一个说明如何进行解档来获取一个新对象。

遵循协议和设置属性

```objc
  //1.遵循NSCoding协议 
  @interface Person : NSObject   //2.设置属性
  @property (strong, nonatomic) UIImage *avatar;
  @property (copy, nonatomic) NSString *name;
  @property (assign, nonatomic) NSInteger age;
  @end
```

实现协议方法

```objc
  //解档
  - (id)initWithCoder:(NSCoder *)aDecoder {
      if ([super init]) {
          self.avatar = [aDecoder decodeObjectForKey:@"avatar"];
          self.name = [aDecoder decodeObjectForKey:@"name"];
          self.age = [aDecoder decodeIntegerForKey:@"age"];
      }
      return self;
  }
  
  //归档
  - (void)encodeWithCoder:(NSCoder *)aCoder {
      [aCoder encodeObject:self.avatar forKey:@"avatar"];
      [aCoder encodeObject:self.name forKey:@"name"];
      [aCoder encodeInteger:self.age forKey:@"age"];
  }

```
  
  
**特别注意**

```objc
如果需要归档的类是某个自定义类的子类时，就需要在归档和解档之前先实现父类的归档和解档方法。即 [super encodeWithCoder:aCoder] 和 [super initWithCoder:aDecoder] 方法;
```

#### 2.使用

需要把对象归档是调用`NSKeyedArchiver`的工厂方法 `archiveRootObject: toFile: `方法。

```objc
  NSString *file = [NSSearchPathForDirectoriesInDomains(NSDocumentDirectory, NSUserDomainMask, YES).firstObject stringByAppendingPathComponent:@"person.data"];
  Person *person = [[Person alloc] init];
  person.avatar = self.avatarView.image;
  person.name = self.nameField.text;
  person.age = [self.ageField.text integerValue];
  [NSKeyedArchiver archiveRootObject:person toFile:file];
```

需要从文件中解档对象就调用`NSKeyedUnarchiver`的一个工厂方法 `unarchiveObjectWithFile:` 即可。

```objc
  NSString *file = [NSSearchPathForDirectoriesInDomains(NSDocumentDirectory, NSUserDomainMask, YES).firstObject stringByAppendingPathComponent:@"person.data"];
  Person *person = [NSKeyedUnarchiver unarchiveObjectWithFile:file];
  if (person) 
  {
     self.avatarView.image = person.avatar;
     self.nameField.text = person.name;
     self.ageField.text = [NSString stringWithFormat:@"%ld", person.age];
  }
```
  
#### 3.注意

```objc
必须遵循并实现NSCoding协议
保存文件的扩展名可以任意指定
继承时必须先调用父类的归档解档方法
```


### SQLite3

> 之前的所有存储方法，都是覆盖存储。如果想要增加一条数据就必须把整个文件读出来，然后修改数据后再把整个内容覆盖写入文件。所以它们都不适合存储大量的内容。

#### 1.字段类型

表面上·SQLite·将数据分为以下几种类型：

```objc
integer : 整数
real : 实数（浮点数）
text : 文本字符串
blob : 二进制数据，比如文件，图片之类的
```

实际上`SQLite`是无类型的。即不管你在创表时指定的字段类型是什么，存储是依然可以存储任意类型的数据。而且在创表时也可以不指定字段类型。`SQLite`之所以什么类型就是为了良好的编程规范和方便开发人员交流，所以平时在使用时最好设置正确的字段类型！主键必须设置成`integer`


#### 2. 准备工作

准备工作就是导入依赖库啦，在`iOS`中要使用`SQLite3`，需要添加库文件：`libsqlite3.dylib`并导入主头文件，这是一个`C语言`的库，所以直接使用`SQLite3`还是比较麻烦的。

#### 3.使用

##### 1.创建数据库并打开

操作数据库之前必须先指定数据库文件和要操作的表，所以使用`SQLite3`，首先要打开数据库文件，然后指定或创建一张表。

```objc
//  打开数据库并创建一个表
- (void)openDatabase 
{
   //1.设置文件名
   NSString *filename = [NSSearchPathForDirectoriesInDomains(NSDocumentDirectory, NSUserDomainMask, YES).firstObject stringByAppendingPathComponent:@"person.db"];
   
   //2.打开数据库文件，如果没有会自动创建一个文件
   NSInteger result = sqlite3_open(filename.UTF8String, &_sqlite3);
   
   if (result == SQLITE_OK) 
   {
       NSLog(@"打开数据库成功！");
       
       //3.创建一个数据库表
       char *errmsg = NULL;
       sqlite3_exec(_sqlite3, "CREATE TABLE IF NOT EXISTS t_person(id integer primary key autoincrement, name text, age integer)", NULL, NULL, &errmsg);
       
       if (errmsg) 
       {
           NSLog(@"错误：%s", errmsg);
       } 
       else 
       {
           NSLog(@"创表成功！");
       }
   } 
   else 
   {
       NSLog(@"打开数据库失败！");
   }
}
```

##### 2.执行指令
使用 `sqlite3_exec()` 方法可以执行任何`SQL`语句，比如`创表、更新、插入和删除`操作。但是一般不用它执行查询语句，因为它不会返回查询到的数据。

```objc
// 往表中插入1000条数据
- (void)insertData 
{
    NSString *nameStr;
    NSInteger age;
    
    for (NSInteger i = 0; i < 1000; i++) 
    {
      nameStr = [NSString stringWithFormat:@"Bourne-%d", arc4random_uniform(10000)];
      age = arc4random_uniform(80) + 20;
      NSString *sql = [NSString stringWithFormat:@"INSERT INTO t_person (name, age) VALUES('%@', '%ld')", nameStr, age];
      char *errmsg = NULL;
      sqlite3_exec(_sqlite3, sql.UTF8String, NULL, NULL, &errmsg);
      if (errmsg) 
      {
          NSLog(@"错误：%s", errmsg);
      }
    }
    NSLog(@"插入完毕！");   
}
```

##### 3.查询指令
前面说过一般不使用 sqlite3_exec() 方法查询数据。因为查询数据必须要获得查询结果，所以查询相对比较麻烦。示例代码如下：

```objc
// sqlite3_prepare_v2() : 检查sql的合法性
// sqlite3_step() : 逐行获取查询结果，不断重复，直到最后一条记录
// sqlite3_coloum_xxx() : 获取对应类型的内容，iCol对应的就是SQL语句中字段的顺序，从0开始。根据实际查询字段的属性，使用sqlite3_column_xxx取得对应的内容即可。
// sqlite3_finalize() : 释放stmt

// 从表中读取数据到数组中
- (void)readData 
{
   NSMutableArray *mArray = [NSMutableArray arrayWithCapacity:1000];
   char *sql = "select name, age from t_person;";
   sqlite3_stmt *stmt;
   NSInteger result = sqlite3_prepare_v2(_sqlite3, sql, -1, &stmt, NULL);
   
   if (result == SQLITE_OK) 
   {
       while (sqlite3_step(stmt) == SQLITE_ROW) 
       {
           char *name = (char *)sqlite3_column_text(stmt, 0);
           NSInteger age = sqlite3_column_int(stmt, 1);
           //创建对象
           Person *person = [Person personWithName:[NSString stringWithUTF8String:name] Age:age];
           [mArray addObject:person];
       }
       self.dataList = mArray;
   }
   sqlite3_finalize(stmt);
}
```

#### 4.总结

总得来说，`SQLite3`的使用还是比较麻烦的，因为都是些`c语言`的函数，理解起来有些困难。不过在一般开发过程中，使用的都是第三方开源库 `FMDB`，封装了这些基本的`c语言`方法，使得我们在使用时更加容易理解，提高开发效率。

### FMDB

#### 1.简介

> FMDB是iOS平台的SQLite数据库框架，它是以OC的方式封装了SQLite的C语言API，它相对于cocoa自带的C语言框架有如下的优点:
使用起来更加面向对象，省去了很多麻烦、冗余的C语言代码
对比苹果自带的Core Data框架，更加轻量级和灵活
提供了多线程安全的数据库操作方法，有效地防止数据混乱


#### 2.核心类

**FMDB有三个主要的类：**

```objc
// FMDatabase
一个FMDatabase对象就代表一个单独的SQLite数据库，用来执行SQL语句

// FMResultSet
使用FMDatabase执行查询后的结果集

// FMDatabaseQueue
用于在多线程中执行多个查询或更新，它是线程安全的
```

#### 3.打开数据库

> 和c语言框架一样，FMDB通过指定SQLite数据库文件路径来创建FMDatabase对象，但FMDB更加容易理解，使用起来更容易，使用之前一样需要导入sqlite3.dylib。打开数据库方法如下：

```objc
NSString *path = [NSSearchPathForDirectoriesInDomains(NSDocumentDirectory, NSUserDomainMask, YES).firstObject stringByAppendingPathComponent:@"person.db"];
FMDatabase *database = [FMDatabase databaseWithPath:path]; 
   
if (![database open]) 
{
    NSLog(@"数据库打开失败！");
}
```
**值得注意的是，Path的值可以传入以下三种情况：**

```objc
// 具体文件路径，如果不存在会自动创建
// 空字符串@""，会在临时目录创建一个空的数据库，当FMDatabase连接关闭时，数据库文件也被删除
// nil，会创建一个内存中临时数据库，当FMDatabase连接关闭时，数据库会被销毁
```
#### 4.更新

> 在FMDB中，除查询以外的所有操作，都称为“更新”, 如：create、drop、insert、update、delete等操作，使用executeUpdate:方法执行更新：

```objc
//常用方法有以下 3 种：   
- (BOOL)executeUpdate:(NSString*)sql, ...

- (BOOL)executeUpdateWithFormat:(NSString*)format, ...

- (BOOL)executeUpdate:(NSString*)sql withArgumentsInArray:(NSArray *)arguments

//示例
[database executeUpdate:@"CREATE TABLE IF NOT EXISTS t_person(id integer primary key autoincrement, name text, age integer)"]; 
  
//或者  
[database executeUpdate:@"INSERT INTO t_person(name, age) VALUES(?, ?)", @"Bourne", [NSNumber numberWithInt:42]];
```
#### 5.查询

查询方法也有3种，使用起来相当简单：

```objc
- (FMResultSet *)executeQuery:(NSString*)sql, ...
- (FMResultSet *)executeQueryWithFormat:(NSString*)format, ...
- (FMResultSet *)executeQuery:(NSString *)sql withArgumentsInArray:(NSArray *)arguments
```

查询示例：

```objc
//1.执行查询
FMResultSet *result = [database executeQuery:@"SELECT * FROM t_person"];

//2.遍历结果集
while ([result next]) 
{
    NSString *name = [result stringForColumn:@"name"];
    int age = [result intForColumn:@"age"];
}
```

#### 6.线程安全

> 在多个线程中同时使用一个 `FMDatabase` 实例是不明智的。不要让多个线程分享同一个`FMDatabase`实例，它无法在多个线程中同时使用。 如果在多个线程中同时使用一个`FMDatabase`实例，会造成数据混乱等问题。所以，请使用 `FMDatabaseQueue`，它是线程安全的。以下是使用方法：

创建队列。

```objc
FMDatabaseQueue *queue = [FMDatabaseQueue databaseQueueWithPath:aPath];

// 使用队列
[queue inDatabase:^(FMDatabase *database)
{    
          [database executeUpdate:@"INSERT INTO t_person(name, age) VALUES (?, ?)", @"Bourne_1", [NSNumber numberWithInt:1]];    
          [database executeUpdate:@"INSERT INTO t_person(name, age) VALUES (?, ?)", @"Bourne_2", [NSNumber numberWithInt:2]];    
          [database executeUpdate:@"INSERT INTO t_person(name, age) VALUES (?, ?)", @"Bourne_3", [NSNumber numberWithInt:3]];      
          FMResultSet *result = [database executeQuery:@"select * from t_person"];    
         while([result next]) {   
         }    
}];

// 而且可以轻松地把简单任务包装到事务里：
[queue inTransaction:^(FMDatabase *database, BOOL *rollback) {    
          [database executeUpdate:@"INSERT INTO t_person(name, age) VALUES (?, ?)", @"Bourne_1", [NSNumber numberWithInt:1]];    
          [database executeUpdate:@"INSERT INTO t_person(name, age) VALUES (?, ?)", @"Bourne_2", [NSNumber numberWithInt:2]];    
          [database executeUpdate:@"INSERT INTO t_person(name, age) VALUES (?, ?)", @"Bourne_3", [NSNumber numberWithInt:3]];      
          FMResultSet *result = [database executeQuery:@"select * from t_person"];    
             while([result next]) {   
             }   
           //回滚
           *rollback = YES;  
    }];
```

FMDatabaseQueue 后台会建立系列化的`GCD`队列，并执行你传给`GCD`队列的块。这意味着 你从多线程同时调用调用方法，`GCD`也会按它接收的块的顺序来执行了。

