### Customizing new and delete

**49. 了解new-handler的行为 （Understand the behavior of the new-handler)**

当new无法申请到新的内存的时候，会不断的调用new-handler，直到找到足够的内存,new_handler是一个错误处理函数：
    namespace std{
        typedef void(*new_handler)();
        new_handler set_new_handler(new_handler p) throw();
    }

一个设计良好的new-handler要做下面的事情：

+ 让更多内存可以被使用
+ 安装另一个new-handler，如果目前这个new-handler无法取得更多可用内存，或许他知道另外哪个new-handler有这个能力，然后用那个new-handler替换自己
+ 卸除new-handler
+ 抛出bad_alloc的异常
+ 不返回，调用abort或者exit

new-handler无法给每个class进行定制，但是可以重写new运算符，设计出自己的new-handler
此时这个new应该类似于下面的实现方式：
    

    void* Widget::operator new(std::size_t size) throw(std::bad_alloc){
        NewHandlerHolder h(std::set_new_handler(currentHandler));      // 安装Widget的new-handler
        return ::operator new(size);                                   //分配内存或者抛出异常，恢复global new-handler
    }

总结：

+ set_new_handler允许客户制定一个函数，在内存分配无法获得满足时被调用
+ Nothrow new是一个没什么用的东西

**50. 了解new和delete的合理替换时机 （Understand when it makes sense to replace new and delete)**

+ 用来检测运用上的错误，如果new的内存delete的时候失败掉了就会导致内存泄漏，定制的时候可以进行检测和定位对应的失败位置
+ 为了强化效率（传统的new是为了适应各种不同需求而制作的，所以效率上就很中庸）
+ 可以收集使用上的统计数据
+ 为了增加分配和归还内存的速度
+ 为了降低缺省内存管理器带来的空间额外开销
+ 为了弥补缺省分配器中的非最佳对齐位
+ 为了将相关对象成簇集中起来

**51. 编写new和delete时需固守常规（Adhere to convention when writing new and delete)**

+ 重写new的时候要保证49条的情况，要能够处理0bytes内存申请等所有意外情况
+ 重写delete的时候，要保证删除null指针永远是安全的

**52. 写了placement new也要写placement delete（Write placement delete if you write placement new)**

如果operator new接受的参数除了一定会有的size_t之外还有其他的参数，这个就是所谓的palcement new

void* operator new(std::size_t, void* pMemory) throw(); //placement new
static void operator delete(void* pMemory) throw();     //palcement delete，此时要注意名称遮掩问题

#### 杂项讨论 (Miscellany)

**53. 不要轻忽编译器的警告（Pay attention to compiler warnings)**

+ 严肃对待编译器发出的warning， 努力在编译器最高警告级别下无warning
+ 同时不要过度依赖编译器的警告，因为不同的编译器对待事情的态度可能并不相同，换一个编译器警告信息可能就没有了

**54. 让自己熟悉包括TR1在内的标准程序库 （Familiarize yourself with the standard library, including TR1)**

其实感觉这一条已经有些过时了，不过虽然过时，但是很多地方还是有用的

+ smart pointers
+ tr1::function ： 表示任何callable entity（可调用物，只任何函数或者函数对象）
+ tr1::bind是一种stl绑定器
+ Hash tables例如set，multisets， maps等
+ 正则表达式
+ tuples变量组
+ tr1::array：本质是一个STL化的数组
+ tr1::mem_fn:语句构造上与程艳函数指针一样的东西
+ tr1::reference_wrapper： 一个让references的行为更像对象的东西
+ 随机数生成工具
+ type traits

**55. 让自己熟悉Boost （Familiarize yourself with Boost)**

主要是因为boost是一个C++开发者贡献的程序库，代码相对比较好