# ios-Programming-thought
编程思想的由来：在开发中我们会遇见各种各样的需求，经常会思考如何快速的完成这些需求，这样就会慢慢形成快速完成这些需求的思想。下面我来总结一下我们开发中常用的编程思想。

1、面向过程编程思想

处理事情以过程为核心，一步一步的实现。C语言编程就是面向过程编程。

2、面向对象编程思想

万物皆对象。典型代表OC等。

3、链式编程思想

是将多个操作（多行代码）通过点号(.)链接在一起成为一句代码,使代码可读性好。a(1).b(2).c(3)

链式编程特点：方法的返回值是block,block必须有返回值（本身对象），block参数（需要操作的值
代表：masonry框架
为了便于理解我们练习写一个计算器。首先创建一个计算机类CalculateManager。平常我们用masonry的时候设置它的属性都会返回一个block，并且block的返回值为make本事，所以我们定义一个方法。
@interface CalculateManager : NSObject

@property (nonatomic,assign) int result;

-(CalculateManager *(^)(int))add;

@end
实现为：

-(CalculateManager *(^)(int))add{
    return ^CalculateManager *(int value){
        _result+=value;
        return self;
    };
}
并把最后的计算结果存储的result变量里面用于返回获取结果。
我们是想让NSObject及其子类都拥有计算能力，所以写一个Category：NSObject (Calculate)。写一个方法参数为一个Block，该block的参数为上面的计算机类对象，所有的计算都在这个block里面执行。

+ (int)cyx_makeCalcuclate:(void(^)(CalculateManager *))block;
实现为：

//将所有的计算代码都放在这里
+ (int)cyx_makeCalcuclate:(void(^)(CalculateManager *))block{

    CalculateManager * manager = [CalculateManager new];
    block(manager);
    return manager.result;
}
用法为：

- (void)viewDidLoad {
    [super viewDidLoad];
    // Do any additional setup after loading the view, typically from a nib.
    int result =  [NSObject cyx_makeCalcuclate:^(CalculateManager *manager) {
       //在这里面实现存放计算代码
        //5+5
        manager.add(5).add(3);
    }];
    NSLog(@"%d",result);
}
这样就实现了。 附demo：https://github.com/SionChen/ChainProgramming

4、响应式编程思想

不需要考虑调用顺序，只需要知道考虑结果，类似于蝴蝶效应，产生一个事件，会影响很多东西，这些事件像流一样的传播出去，然后影响结果，借用面向对象的一句话，万物皆是流。

代表：KVO运用。
我们自己模仿实现一下KVO。在写之前我们首先了解一下KVO的底层实现：
基本的原理：
当观察某对象 A 时，KVO 机制动态创建一个对象A当前类的子类，并为这个新的子类重写了被观察属性 keyPath 的 setter 方法。setter 方法随后负责通知观察对象属性的改变状况。

深入剖析：
Apple 使用了 isa 混写（isa-swizzling）来实现 KVO 。当观察对象A时，KVO机制动态创建一个新的名为：NSKVONotifying_A 的新类，该类继承自对象A的本类，且 KVO 为 NSKVONotifying_A 重写观察属性的 setter 方法，setter 方法会负责在调用原 setter 方法之前和之后，通知所有观察对象属性值的更改情况。
参考文章：https://www.jianshu.com/p/e59bb8f59302
话不多说我们开始写，首先创建一个person类定义一个实例变量：

@interface Person : NSObject
{
    @public
    NSString * _name;
}

@property (nonatomic,copy) NSString *name;
@end
创建一个NSObject的Category用于给所有NSObject及其子类新增 添加监听方法：

@interface NSObject (KVO)
- (void)cyx_addObserver:(NSObject *)observer forKeyPath:(NSString *)keyPath options:(NSKeyValueObservingOptions)options context:(nullable void *)context;
@end
NSString * const cyx_key = @"observer";

@implementation NSObject (KVO)
-(void)cyx_addObserver:(NSObject *)observer forKeyPath:(NSString *)keyPath options:(NSKeyValueObservingOptions)options context:(void *)context
{


    objc_setAssociatedObject(self, (__bridge const void *)(cyx_key), observer, OBJC_ASSOCIATION_RETAIN_NONATOMIC);

    //修改isa 指针
    object_setClass(self, [SonPerson class]);
}
这里利用runtime修改isa指针，修改调用方法时寻找方法的类。这里我们修改到SonPerson类。并在SonPerson类里面实现监听方法。

extern NSString * const cyx_key;
@implementation SonPerson

-(void)setName:(NSString *)name{
    [super setName:name];

    NSObject * observer = objc_getAssociatedObject(self, cyx_key);

    [observer observeValueForKeyPath:@"name" ofObject:self change:nil context:nil];
}
这里也用到了runtime 里面 objc_getAssociatedObject 和objc_setAssociatedObject动态存储方法。
好了那我们来用一下试一下效果吧。

#import "ViewController.h"
#import "Person.h"
#import "NSObject+KVO.h"
@interface ViewController ()

@property (nonatomic,strong) Person *p;
@end

@implementation ViewController

- (void)viewDidLoad {
    [super viewDidLoad];
    // Do any additional setup after loading the view, typically from a nib.
    Person * p = [[Person alloc] init];
    [p cyx_addObserver:self forKeyPath:@"name" options:0 context:nil];
    _p = p;
}
-(void)observeValueForKeyPath:(NSString *)keyPath ofObject:(id)object change:(NSDictionary<NSKeyValueChangeKey,id> *)change context:(void *)context{
    NSLog(@"%@",_p.name);
}

-(void)touchesBegan:(NSSet<UITouch *> *)touches withEvent:(UIEvent *)event{

    static int i=0;
    i++;
    _p.name = [NSString stringWithFormat:@"%d",i];
    //_p -> _name = [NSString stringWithFormat:@"%d",i];

}
输出：

2018-04-21 11:15:17.974785+0800 04-响应式编程思想[1882:274712] 1
2018-04-21 11:15:18.293700+0800 04-响应式编程思想[1882:274712] 2
2018-04-21 11:15:18.687331+0800 04-响应式编程思想[1882:274712] 3
2018-04-21 11:15:19.036166+0800 04-响应式编程思想[1882:274712] 4
2018-04-21 11:15:19.396075+0800 04-响应式编程思想[1882:274712] 5
2018-04-21 11:15:19.699907+0800 04-响应式编程思想[1882:274712] 6
2018-04-21 11:15:19.981256+0800 04-响应式编程思想[1882:274712] 7
demo:https://github.com/SionChen/ReactiveProgramming

5、函数式编程思想

是把操作尽量写成一系列嵌套的函数或者方法调用。

函数式编程本质:就是往方法中传入Block,方法中嵌套Block调用，把代码聚合起来管理
函数式编程特点：每个方法必须有返回值（本身对象）,把函数或者Block当做参数,block参数（需要操作的值）block返回值（操作结果）
代表：ReactiveCocoa。
  我们用函数式编程实现，写一个加法计算器，创建一个计算机类CaluculateManager。
@interface CaluculateManager : NSObject
@property (nonatomic,assign) int result;
//计算
-(instancetype)caluculate:(int(^)(int))caluculateBlock;
@end
@implementation CaluculateManager

-(instancetype)caluculate:(int (^)(int))caluculateBlock{
    _result = caluculateBlock(_result);
    return self;
}
@end
使用：

- (void)viewDidLoad {
    [super viewDidLoad];
    // Do any additional setup after loading the view, typically from a nib.


    CaluculateManager * manager = [CaluculateManager new];
    int result = [[manager caluculate:^int(int results){
        results+=5;
        results*=5;
        return results;
    }] result];
    NSLog(@"%d",result);
}
附demo：https://github.com/SionChen/FunctionalProgramming。欢迎指教。

作者：一条咸鱼哇
链接：https://www.jianshu.com/p/3b11b0c71b0c
