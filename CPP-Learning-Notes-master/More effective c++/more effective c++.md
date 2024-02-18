#### 一、基础议题

**1. 区分指针和引用**

引用必须指向一个对象，而不是空值，下面是一个危险的例子：
    

    char* pc = 0;  //设置指针为空值
    char& rc = *pc;//让引用指向空值，很危险！！！

下面的情况下使用指针：

+ 存在不指向任何对象的可能
+ 需要能够在不同的时刻指向不同的对象
+ 其他情况应该使用引用

**2. 优先考虑C++风格的类型转换**

上本书effective c++ 说过了，第27条

**3. 决不要把多态用于数组**

主要是考虑以下写法：
    

```c++
class BST{...}
class BalancedBST : public BST{...}

void printBSTArray(const BST array[]){
    for(auto i : array){
        std::cout << *i;
    }
}

BalancedBST bBSTArray[10];
printBSTArray(bBSTArray);
```

由于我们之前说的，这种情况下编译器是毫无警告的，而对象在传递过程中是按照声明的大小来传递的，所以每一个元素的间隔是sizeof(BST)此时指针就指向了错误的地方

**4. 避免不必要的默认构造函数**

这里主要是为了防止出现有了对象但是却没有必要的数据，例如：没有id的人
但是主要还是在关键词 *不必要* 上面，必要的默认构造函数，不会造成数据出现遗漏的话，还是可以用的
当然，如果不提供缺省构造函数的话，例如：
    

    class EP{
    public:
        EP(int ID);
    }

这样的代码会让使用者在某些时候非常的难受，特别是当EP类是虚类的时候。

这时可以通过让用户使用

    EP bestP[] = {
        EP(ID1),
        EP(ID2),
        ......
    }

函数数组的方法，或者是指针：
    

```c++
typedef EP* PEP;
PEP bestPieces[10];
PEP *bestPieces = new PEP[10];//然后使用的时候再重新new来进行初始化
```

#### 二、运算符

**5. 小心用户自定义的转换函数**

因为可能会出现一些无法理解的并且也是无能为力的运算，而且在不需要这些类型转换函数的时候，仍然可能会调用这些转换，例如下面的代码：

```c++
// 有理数类
class Rational{
public:
    Rational(int numerator = 0, int denominator = 1)
    operator double() const;
}

Rational r(1, 2);
double d = 0.5 * r; //将r转换成了double进行计算

cout << r; //会调用最接近的类型转换函数double，将r转换成double打印出来，而不是想要的1/2，
```

上面问题的解决方法是，把double变成
    

```c++
double asDouble() const;//这样就可以直接用了
```

但是即使这样做还有可能会出现隐式转换的现象：
    

    template<class T>
    class Array{
    public:
        Array(int size);
        T& operator[](int index);
    };
    
    bool operator==(const Array<int> &lhs, const Array<int> & rhs);
    Array<int> a(10), b(10);
    if(a == b[3]) //想要写 a[3] == b[3]，但是这时候编译器并不会报错，解决方法是使用explicit关键字
    
    explicit Array(int size); 
    if(a == b[3]) // 错误，无法进行隐式转换

其实还有一种很骚的操作：

    class Array { 
    public:  
        class ArraySize {                    // 这个类是新的 
            public: 
            ArraySize(int numElements):theSize(numElements){}
            int size() const { return theSize;}
        private: 
            int theSize;
        };
        Array(int lowBound, int highBound); 
        Array(ArraySize size);                  // 注意新的声明
        ... 
    }; 

这样写的代码在Array<int> a(10);的时候，编译器会先通过类型转换转换成ArraySize，然后再进行构造，虽然麻烦很多，效率也低了很多，但是在一定程度上可以避免隐式转换带来的问题

**6. 区分自增运算符和自减运算符的前缀形式与后缀形式**

这一点主要是要知道前缀和后缀的重载形式是不同的，以及重载的时候不要进行连续重载例如i++++;
因为连续的+号会导致创建很多临时对象，效率会变低

**7. 不要重载"&&"、"||"和","**

主要是因为上面三个符号，大部分的程序员都已经达成共识，先运算前面的一串表达式，再判断后面的一串表达式：
if(expression1 && expression2){} 就会先运算第一个表达式，然后再运算第二个表达式

比较特殊的是逗号操作符：“,“，例如最常见的for循环：
    

    for(int i = 0, j = strlen(s)-1; i < j; i++, j--){}

在这个for循环里面，因为最后一个部分职能使用一个表达式，分开表达式来改变i和j的值是不合法的，用逗号表达式就会先计算出来左边的i++，然后计算出逗号右边的j--

**8. 理解new和delete在不同情形下的含义**

两种new: new 操作符（new operator）和new操作（operator new）的区别

    string *ps = new string("Memory Management"); //使用的是new操作符，这个操作符像sizeof一样是内置的，无法改变
    
    void* operator new(size_t size); // new操作，可以重写这个函数来改变如何分配内存

一般不会直接调用operator new，但是可以像调用其他函数一样调用他：

    void* rawMemory = operator new(sizeof(String));

placement new : placement new 是有一些已经被分配但是没有被处理的内存，需要在这个内存里面构造一个对象，使用placement new 可以实现这个需求，实现方法：
    

    class Widget{
        public:
            Widget(int widgetSize);
        ....
    };
    
    Widget* constructWidgetInBuffer(void *buffer, int widgetSize){
        return new(buffer) Widget(widgetSize);
    }

这样就返回一个指针，指向一个Widget对象，对象在传递给函数的buffer里面分配

同样的道理：
    delete buffer; //指的是先调用buffer的析构函数，然后再释放内存
    operator delete(buffer); //指的是只释放内存，但是不调用析构函数

而placement new 出来的内存，就不应该直接使用delete操作符，因为delete操作符使用operator delete来释放内存，但是包含对象的内存最初不是被operator new分配的，而应该显示调用析构函数来消除构造函数的影响

new[]和delete[]就相当于对每一个数组元素调用构造和析构函数

#### 三、异常

**9. 使用析构函数防止资源泄漏**

原代码：
    

    void processAdoptions(istream& dataSource){
        while(dataSource){
            ALA *pa = readALA(dataSource);
        }
        try{
            pa->processAdoption();
        }
        catch(...){
            delete pa; //在抛出异常的时候避免泄露
            throw;
        }
        delete pa;     //在不抛出异常的时候避免泄露
    }

因为这种情况会需要删除两次pa，代码维护很麻烦，所以需要进行优化：

template<class T>
class auto_ptr{
public:
    auto_ptr(T *p=0):ptr(p){} //保存ptr，指向对象
    ~auto_ptr(){delete prt;}
private:
    T *ptr;    
}

void processAdoptions(istream& dataSource){
    while(dataSource){
        auto_ptr<ALA> pa(readALA(dataSource));
        pa->processAdoption();
    }
}

auto_ptr后面隐藏的思想是：使用一个对象来存储需要被自动释放的资源，然后依靠对象的析构函数来释放资源。
事实上WindowHandle就是这样一个东西

那么这样就引出一个非常重要的规则：资源应该被封装在一个对象里面

**10. 防止构造函数里的资源泄漏**

这一条主要是防止在构造函数中出现异常导致资源泄露：
    

    BookEntry::BookEntry(){
        theImage     = new Image(imageFileName);
        theAudioClip = new AudioClip(audioClipFileName);
    }
    BookEntry::~BookEntry(){
        delete theImage;
    }

如果在构造函数new AudioClip里面出现异常的话，那么~BookEntry析构函数就不会执行，那么NewImage就永远不会被删除，而且因为new BookEntry失败，导致delete BookEntry也无法释放theImage，那么只能在构造函数里面使用异常来避免这个问题
    

    BookEntry::BookEntry(){
        try{
            theImage     = new Image(imageFileName);
            theAudioClip = new AudioClip(audioClipFileName);
        }
        catch(...){
            delete theImage;
            delete theAudioClip;
            //上面一段代码和析构函数里面的一样，所以可以直接封装成一个成员函数cleanup：
            cleanup();
            throw;
        }
    }

更好的做法是将theImage和theAudioClip做成成员来进行封装：
    

    class BookEntry{
    public:......
    private:
        const auto_ptr<Image> theImage;
        const auto_ptr<AudioClip> theAudioClip;
    }

**11. 阻止异常传递到析构函数以外**

如果析构函数抛出异常的话，会导致程序直接调用terminate函数，中止程序而不释放对象，所以不应该让异常传递到析构函数外面，而是应该在析构函数里面直接catch并且处理掉

另外，如果析构函数抛出异常的话，那么析构函数就不会完全运行，就无法完成希望做的一些其他事情例如：
    

    Session::~Session(){
        logDestruction(this);
        endTransaction(); //结束database transaction,如果上面一句失败的话，下面这句就没办法正确执行了
    }

**12. 理解“抛出异常”，“传递参数”和“调用虚函数”之间的不同**

传递参数的函数：

    void f1(Widget w);

catch子句：    

    catch(widget w)... 

上面两行代码的相同点：传递函数参数与异常的途径可以是传值、传递引用或者传递指针

上面两行代码的不同点：系统所需要完成操作的过程是完全不同的。调用函数时程序的控制权还会返回到函数的调用处，但是抛出一个异常时，控制权永远都不会回到抛出异常的地方
三种捕获异常的方法：
    

    catch(Widget w);
    catch(Widget& w);
    catch(const Widget& w);

一个被抛出的对象可以通过普通的引用捕获，它不需要通过指向const对象的引用捕获，但是在函数调用中不允许传递一个临时对象到一个非const引用类型的参数里面
同时异常抛出的时候实际上是抛出对象创建的临时对象的拷贝，

另外一个区别就是在try语句块里面，抛出的异常不会进行类型转换（除了继承类和基类之间的类型转换，和类型化指针转变成无类型指针的变换），例如：
    

    void f(int value){
        try{
            throw value; //value可以是int也可以是double等其他类型的值
        }
        catch(double d){
            ....         //这里只处理double类型的异常，如果遇到int或者其他类型的异常则不予理会
        }
    }

最后一个区别就是，异常catch的时候是按照顺序来的，即如果两个catch并且存在的话，会优先进入到第一个catch里面，但是函数则是匹配最优的

**13. 通过引用捕获异常**

使用指针方式捕获异常：不需要拷贝对象，是最快的,但是，程序员很容易忘记写static，如果忘记写static的话，会导致异常在抛出后，因为离开了作用域而失效：
    

    void someFunction(){
        static exception ex;
        throw &ex;
    }
    void doSomething(){
        try{
            someFunction();
        }
        catch(exception *ex){...}
    }

创建堆对象抛出异常：new exception 不会出现异常失效的问题，但是会出现在捕捉以后是否应该删除他们接受的指针，在哪一个层级删除指针的问题
通过值捕获异常：不会出现上述问题，但是会在被抛出时系统将异常对象拷贝两次，而且会出现派生类和基类的slicing problem，即派生类的异常对象被作为基类异常对象捕获时，会把派生类的一部分切掉，例如：
    

    class exception{
    public:
        virtual const char *what() throw();
    };
    class runtime_error : public exception{...};
    void someFunction(){
        if(true){
            throw runtime_error();
        }
    }
    void doSomething(){
        try{
            someFunction();
        }
        catch(exception ex){
            cerr << ex.what(); //这个时候调用的就是基类的what而不是runtime_error里面的what了，而这个并不是我们想要的
        }
    }

通过引用捕获异常：可以避免上面所有的问题，异常对象也只会被拷贝一次：
    

    void someFunction(){...} //和上面一样
    void doSomething(){
        try{...}             //和上面一样
        catch(exception& ex){
            cerr << ex.what(); //这个时候就是调用的runtime_error而不是基类的exception::what()了，其他和上面其实是一样的
        }
    }

**14. 审慎地使用异常规格（exception specifications）**

异常规格指的是函数指定只能抛出异常的类型：
    

    extern void f1();    //f1可以抛出任意类型的异常
    void f2() throw(int);//f2只能抛出int类型的异常
    void f2() throw(int){ 
         f1();           //编译器会因为f1和f2的异常规格不同而在发出异常的时候调用unexpected
    }

在用模板的时候，会让这种情况更为明显：
    

    template<class T>
    bool operator==(const T& lhs, const T&rhs) throw(){
        return &lhs == &rhs;
    }

这个模板为所有的类型定义了一个操作符函数operator==对于任意一对相同类型的对象，如果有一样的地址，则返回true，否则返回false，单单这么一个函数可能不会抛出异常，但是如果有operator&重载时，operator&可能会抛出异常，这样就违反了异常规则，让程序跳转到unexpected

阻止程序跳转到unexpected的三种方法：
将所有的unexpected异常都替换成UnexpectedException对象：
    

    class UnexpectedException{}; //所有的unexpected异常对象都被替换成这种对象
    void convertUnexpected(){       //如果一个unexpected异常被抛出，这个函数就会被调用
        throw UnexpectedException();
    }
    set_unexpected(convertUnexpected);

替换unexpected函数：

    void convertUnexpected(){ //如果一个unexpected异常被抛出，这个函数被调用
        throw;                //只是重新抛出当前的异常
    }
    set_unexpected(convertUnexpected);//安装convertUnexpected作为unexpected的替代品，此方法应该在所有的异常规格里面包含bad_exception

总结：异常规格应该在加入之前谨慎的考虑它带来的行为是否是我们所希望的

**15. 理解异常处理所付出的代价**

编译器带来的开销（很难消除，因为所有的编译器都是支持异常的
try块语句的开销：大概会降低5%-10%的速度和增加相应的代码尺寸

#### 四、效率

**16. 记住80-20准则**

分别有20%的代码耗用了80%的程序资源，运行时间，内存，磁盘，有80%的维护投入到20%的代码上
用profiler工具来对程序进行分析

**17. 考虑使用延迟计算**

一个延迟计算的例子：

    class String{....}
    String s1 = "Hello";
    String s2 = s1;  //在正常的情况下，这一句需要调用new操作符分配堆内存，然后调用strcpy将s1内的数据拷贝到s2里面。但是我们此时s2并没有被使用，所以我们不需要s2，这个时候如果让s2和s1共享一个值，就可以减小这些开销

使用延迟计算进行读操作和写操作：

    String s = "Homer's Iliad";
    cout << s[3];
    s[3] = 'x';

首先调用operator[] 用来读取string的部分值，但是第二次调用该函数式为了完成写操作。读取效率较高，写入因为需要拷贝，所以效率较低，这个时候可以推迟作出是读操作还是写操作的决定。

延迟策略进行数据库操作：有点类似之前写web 的时候，把数据放在内存和数据库两份，更新的时候只更新内存，然后隔一段时间（或者等到使用的时候）去更新数据库。
在effective c++里面，则是更加专业的将这个操作封装成了一个类，然后把是否更新数据库弄成一个flag。以及使用了mutable关键字，来修改数据

延迟表达式：
    

    Matrix<int> m1(1000, 1000), m2(1000, 1000);
    m3 = m1 + m2;
    因为矩阵的加法计算量太大（1000*1000）次计算，所以可以先用表达式表示m3是m1和m2的和，然后真正需要计算出值的时候再真的进行计算（甚至计算的时候也只计算m3[3][2]这样某一个位置的值）

**18. 分期摊还预期的计算开销（提前计算法）**

例如对于max， min函数，如果被频繁调用的话，就可以专门将min和max缓存城一个m_min成员或者mmax成员，这样就在每次调用的时候直接返回就行了，不需要每次调用的时候就重新计算，这个方法叫做cache

prefetching是另一种方法，例如从磁盘读取数据的时候，一次读取一整块或者整个扇区的数据，因为一次读取一大块要比不同时间读取几个小块要快

**19. 了解临时对象的来源**

通常意义的临时对象指的是 temp = a; a = b; b = temp;中的temp
但是在C++中的临时对象指的是那些看不见的东西，例如：

    size_t countChar(const string& str, char ch);
    char buffer[MAX_STRING_LEN];
    cout << countChar(buffer, c);

对于countChar的调用来说，buffer是一个char的数据，但是其形参是const string，那么就需要建立一个string类型的临时对象，然后用buffer作为参数对这个临时对象进行初始化

另外再如operator+重载函数中，函数的返回值是临时的，因为它没有被命名。

所以在任何时候只要见到函数中的常量引用参数，就存在建立临时对象的可能性

**20. 协助编译器实现返回值优化**

一个返回一整个对象的函数，效率是很低的，因为需要调用对象的析构和构造函数。但是有时候编译器会帮助优化我们的实现：
    

    inline const Rational operator*(const Rational& lhs, const Rational& rhs{
        return Rational(lhs.numerator() * rhs.numerator(), lhs.denominator() * rhs.denominator());
    }

上面这个操作实在是太骚了，初看起来好像是会创建一个Rational的临时对象，但是实际上编译器会把这个临时对象给优化掉，所以就免除了析构和构造的开销，而inline还可以减少函数的调用开销

**21. 通过函数重载避免隐式类型转换**

改代码之前：

    class UPInt{
        public:
        UPInt();
        UPInt(int value);
    }
    const UPInt operator+(const UPInt& lhs, const UPInt& rhs);
    upi3 = upi1 + upi2;
    upi3 = 10 + upi1;  // 会产生隐式类型转换，转换过程中会出现临时对象
    upi3 = upi1 + 10;

改代码之后：
    

    const UPInt operator+(const UPInt& lhs, const UPInt& rhs);
    const UPInt operator+(const UPInt& lhs, int rhs);
    const UPInt operator+(int lhs, const UPInt& rhs);

**22. 考虑使用op=来取代单独的op运算符**

operator+ 和operator+=是不一样的，所以如果想要重载+号，就最好重载+=，那么一个比较好的方法就是把+号用+=来实现，当然如果可以的话，可以使用模板编写：
    

    template<class T>
    const T operator+(const T& lhs, const T& rhs)
    {
        return T(lhs) += rhs;
    }
    template<class T>
    const T operator-(const T& lhs, const T& rhs){
        return T(lhs) -= rhs; 
    }


**23. 考虑使用其他等价的程序库**

例如stdio和iostream两个程序库都有输入输出的功能，但是stdio库则速度更快，iostream则写起来更安全。需要合理的选择应用的替代库

**24. 理解虚函数、多重继承、虚基类以及RTTI所带来的开销**

C++的特性和编译器会很大程度上影响程序的效率，所以我们有必要知道编译器在一个C++特性后面做了些什么事情。

例如虚函数，指向对象的指针或者引用的类型是不重要的，大多数编译器使用的是virtual table(vtbl)和virtual table pointers(vptr)来进行实现

vtbl:  

    class C1{
    public:
        C1();
        virtual ~C1();
        virtual void f1();
        virtual int f2(char c)const;
        virtual void f3(const string& s);
        void f4()const
    }

vtbl的虚拟表类似于下面这样,只有虚函数在里面，非虚函数的f4不在里面：

     ___
    |___| → ~C1()
    |___| → f1()
    |___| → f2()
    |___| → f3()

如果按照上面的这种，每一个虚函数都需要一个地址空间的话，那么如果拥有大量虚函数的类，就会需要大量的地址存储这些东西，这个vtbl放在哪里根据编译器的不同而不同

vptr：

     __________
    |__________| → 存放类的数据
    |__________| → 存放vptr

每一个对象都只存储一个指针，但是在对象很小的时候，多于的vptr将会看起来非常占地方。在使用vptr的时候，编译器会先通过vptr找到对应的vtbl，然后通过vtbl开始找到指向的函数
事实上对于函数：
    

    pC1->f1();

他的本质是：

    (*pC1->vptr[i])(pC1);

在使用多继承的时候，vptr会占用很大的地方，并且非常恶心，所以不要用多继承

RTTI：能够让我们在runtime找到对象的类信息，那么就肯定有一个地方存储了这些信息，这个特性也可以使用vtbl实现，把每一个对象，都添加一个隐形的数据成员type_info，来存储这些东西，从而占用很大的空间

#### 五、技巧

**25. 使构造函数和非成员函数具有虚函数的行为**

    class NewsLetter{
    private:
        static NLComponent *readComponent(istream& str);
        virtual NLComponent *clone() const = 0;
    };
    NewsLetter::NewsLetter(istream& str){
        while(str){
            components.push_back(readComponent(str));
        }
    }
    class TextBlock: public NLComponent{
    public:
        virtual TextBlock*clone()const{
            return new TextBlock(*this);
        }
    }

在上面那段代码当中，readComponent就是一个具有构造函数行为（因为能够创建出新的对象）的函数，我们叫做虚拟构造函数

clone() 叫做虚拟拷贝构造函数,相当于拷贝一个新的对象

通过这种方法，我们上面的NewsLetter构造函数就可以这样：

    NewsLetter::NewsLetter(const NewsLetter& rhs){
        while(str){
            for(list<NLComponent*>::const_iterator it=rhs.component.begin(); it!=rhs.component.end();it++){
                components.push_back((*it)->clone());
            }
        }
    }

这样每一个TextBlock都可以调用他自己的clone，其他的子类也可以调用他们自己对应的clone()

**26. 限制类对象的个数**

比如某个类只应该有一个对象，那么最简单的限制这个个数的方法就是把构造函数放在private域里面，这样每个人都没有权力创建对象

或者做一个约束，每次创建的时候都返回static的对象：
    

    class Printer{
    public:
        friend Printer& thePrinter();或者static Printer& thePrinter();
    private:
        Printer();
        Printer(const Printer& rhs);
    };
    Printer& thePrinter(){
        static Printer p;
        return p;
    }

上面这段代码中，Printer类的构造函数是private，可以阻止建立对象，全局函数thePrinter被声明为类的友元，让thePrinter避免私有构造函数引起的限制

创建对象的环境：
当然还有一个直观的方法来限制对象的个数，就是添加一个名为numObjects的static变量，来记录对象的个数，当然这种方法在出现继承的时候会出现问题（一个Printer和一个继承自Printer的colorPrinter同时存在的时候，就会超出numObjects个数，这个时候就需要限制继承

允许对象来去自由：
如果使用伪构造函数的话，会导致对象销毁后，无法创建新的对象，解决方法就是一起使用上面的伪构造函数和计数器。

一个具有对象计数功能的基类：
如果拥有大量像Printer这样的类需要进行计数，那么较好的方法就是一次性封装所有的计数功能,需要确保每个进行实例计数的类都有一个相互隔离的计数器，所以模板会比较好:
    

    template <class BeingCounted>
    class Counted{
    public:
        class TooManyObjects{};
        static int objectCount(){return numObjects;}
    protected:
        Counted();
        Counted(const Counted& rhs);
        ~Counted(){ --numObjects; }
    private:
        static int numObjects;
        static const size_t maxObjects;
        void init();                 //避免构造函数的代码重复
    };
    
    template<class BeingCounted>
    Counted<BeingCounted>::Counted(){init();}
    
    template<class BeingCounted>
    Counted<BeingCounted>::Counted(const Counted<BeingCounted>&){init();}
    
    template<class BeingCounted>
    void Counted<BeingCounted>::init(){
        if(numObjects >= maxObjects)throw TooManyObjects();
        ++numObjects;
    }
    
    class Printer:private Counted<Printer>{
    public:
        static Printer* makePrinter(); // 伪构造函数
        using Counted<Printer>::objectCount;
        using Counted<Printer>::TooManyObjects;
    }

**27. 要求或禁止对象分配在堆上**

必须在堆中建立对象（程序有自我管理对象的需求）：
禁用隐式的构造函数和析构函数，例如声明成private，或者仅仅让析构函数成为private（副作用小一些），然后创建一个public的destory()方法来调用析构。
遇到继承析构的问题的话(现在的做法无法继承)，也可以将析构函数声明成protected的

判断一个对象是否在堆中：
在构造函数中无法区分是否在堆中，但是在new里面可以做些事情：
    

    class UPNumber{
    public:
        class HeapConstraintViolation{};
        static void* operator new(size_t size);
        UPNumber();
    private:
        static bool onTheHeap;
    };

实际上上面这段代码是跑不了的，因为如果使用new[]创造数组的话就没有办法用了

另一种方法是判断变量所在的地址，因为stack是从高位地址向下的，heap是从地位地址向上的：
    

    bool onHeap(const void *address) { 
        char onTheStack; // 局部栈变量，因为他是新的变量，所以比他小的都在堆或者静态空间里面，比他大的都在栈里面
        return address < &onTheStack; 
    }

禁止堆对象：
重写operator new就行了，例如弄成private

**28. 智能(smart)指针**

auto_ptr：会把值给传出去，原来的指针作废掉
实现dereference（取出指针所指东西的内容）：
    template<class T>
    T& SmartPtr<T>::operaotr*()const{
        return *pointee;
    }

    template<class T>
    T* SmartPtr<T>::operator->()const{
        return pointee;
    }

测试smart pointer是否是NULL：
如果直接使用下面的代码是错误的：
    

    SmartPtr<TreeNode> ptn;
    if(ptn == 0)... //error
    if(ptn)... //error
    if(!ptn)... //error

所以需要进行隐式类型转换操作符，才能够进行上面的操作
    

    template<class T>
    class SmartPtr{
    public:
        operator void*();
    };
    SmartPtr<TreeNode> ptn;
    if(ptn == 0) //现在正确
    if(ptn) //现在正确
    if(!ptn) //现在正确

smart pointer 和继承类/基类的类型转换:

    class MusicProduct{....};
    class Cassette:public MusicProduct{....};
    class CD:public MusicProduct{....};
    displayAndPlay(const SmartPtr<MusicProduct>& pmp, int numTimes);
    
    SmartPtr<Cassette> funMusic(new Cassette("1234"));
    SmartPtr<CD> nightmareMusic(new CD("143"));
    displayAndPlay(funMusic, 10); // 错误!
    displayAndPlay(nightmareMusic, 0); // 错误!

我们可以看到的是，如果没有隐式转换操作符的话，是没有办法进行转换的，那么解决方法就是添加一个操作符,：
    

    class SmartPtr<Cassette>{//或者用模板来代替
    public:
        operator SmartPtr<MusicProduct>(){
            return SmartPtr<MusicProduct>(pointee);
        }
    };

smart pointer 和 const：
    

    SmartPtr<CD> p; //non-const 对象 non-const 指针
    SmartPtr<const CD> p; //const 对象 non-const 指针
    const SmartPtr<CD> p = &goodCD; //non-const 对象 const 指针
    const SmartPtr<const CD> p = &goodCD; //const 对象 const 指针
    
    template<class T>      // 指向const对象的
    class SmartPtrToConst{ //灵巧指针
        ...                // 灵巧指针通常的成员函数
    protected:
        union {
            const T* constPointee; // 让 SmartPtrToConst 访问
            T* pointee; // 让 SmartPtr 访问
        };
    };
    
    template<class T> // 指向 non-const 对象的灵巧指针
    class SmartPtr: public SmartPtrToConst<T> {
        ... // 没有数据成员
    };


**29. 引用计数**
就是一个smart pointer，不讨论了
**30. 代理类**

例子：实现二维数组类：
    

    template<class T>
    class Array2D{
    public:
        Array2D(int dim1, int dim2);
        class Array1D{
        public:
            T& operator[](int index);
            const T& operator[](int index) const;
        };
        Array1D operator[](int index);
        const Array1D operator[](int index) const;
    };
    Array2D<int> data(10, 20);
    cout << data[3][6] //这里面的[][]运算符是通过两次重载实现的

例子：代理类区分[]操作符的读写：

采用延迟计算方法，修改operator[]让他返回一个（代理字符的）proxy对象而不是字符对象本身，并且判断之后这个代理字符怎么被使用，从而判断是读还是写操作
    

    class String{
    public:
        class CharProxy{
        public:
            CharProxy(String& str, int index);
            CharProxy& operator=(const CharProxy& rhs);
            CharProxy& operator=(char c);
            operator char() const;
        private:
            String& theString;
            int charIndex;
        };
        const CharProxy operator[](int index) const;//对于const的Strings
        CharProxy operator[](int index);            //对于non-const的Strings
    
        friend class CharProxy;
    private:
        RCPtr<StringValue> value;
    };

