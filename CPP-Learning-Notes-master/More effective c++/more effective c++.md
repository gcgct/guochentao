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


