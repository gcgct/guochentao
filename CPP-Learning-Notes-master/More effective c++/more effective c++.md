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