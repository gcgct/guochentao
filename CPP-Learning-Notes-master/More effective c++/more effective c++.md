#### 一、基础议题

**1. 区分指针和引用**

引用必须指向一个对象，而不是空值，下面是一个危险的例子：
    

    char* pc = 0;  //设置指针为空值
    char& rc = *pc;//让引用指向空值，很危险！！！

下面的情况下使用指针：

+ 存在不指向任何对象的可能
+ 需要能够在不同的时刻指向不同的对象
  其他情况应该使用引用

**2. 优先考虑C++风格的类型转换**

上本书effective c++ 说过了，第27条

**3. 决不要把多态用于数组**

主要是考虑以下写法：
    

    class BST{...}
    class BalancedBST : public BST{...}
    
    void printBSTArray(const BST array[]){
        for(auto i : array){
            std::cout << *i;
        }
    }
    
    BalancedBST bBSTArray[10];
    printBSTArray(bBSTArray);

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
    

    typedef EP* PEP;
    PEP bestPieces[10];
    PEP *bestPieces = new PEP[10];然后使用的时候再重新new来进行初始化

#### 二、运算符

**5. 小心用户自定义的转换函数**

因为可能会出现一些无法理解的并且也是无能为力的运算，而且在不需要这些类型转换函数的时候，仍然可能会调用这些转换，例如下面的代码：
    

    // 有理数类
    class Rational{
    public:
        Rational(int numerator = 0, int denominator = 1)
        operator double() const;
    }
    
    Rational r(1, 2);
    double d = 0.5 * r; //将r转换成了double进行计算
    
    cout << r; //会调用最接近的类型转换函数double，将r转换成double打印出来，而不是想要的1/2，

上面问题的解决方法是，把double变成
    

    double asDouble() const;这样就可以直接用了

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
