### Templates and Generic Programming

**41. 了解隐式接口和编译期多态 （Understand implicit interfaces and compile-time polymorphism)**

对于面向对象编程：以显式接口（explicit interfaces）和运行期多态（runtime polymorphism）解决问题：
    

    class Widget {
    public:
        Widget();
        virtual ~Widget();
        virtual std::size_t size() const;
        void swap(Widget& other); //第25条
    }
    
    void doProcessing(Widget& w){
        if(w.size()>10){...}
    }

+ 在上面这段代码中，由于w的类型被声明为Widget，所以w必须支持Widget接口，我们可以在源码中找出这个接口，看看他是什么样子（explicit interface），也就是他在源码中清晰可见
+ 由于Widget的某些成员函数是virtual，w对于那些函数的调用将表现运行期多态，也就是运行期间根据w的动态类型决定调用哪一个函数

在templete编程中：隐式接口（implicit interface）和编译器多态（compile-time polymorphism）更重要：
    

    template<typename T>
    void doProcessing(T& w)
    {
        if(w.size()>10){...}
    }

+ 在上面这段代码中，w必须支持哪一种接口，由template中执行于w身上的操作来决定，例如T必须支持size等函数。这叫做隐式接口
+ 凡涉及到w的任何函数调用，例如operator>，都有可能造成template具现化，使得调用成功，根据不同的T调用具现化出来不同的函数，这叫做编译期多态

**42. 了解typename的双重意义 （Understand the two meanings of typename)**

下面一段代码：
    

    template<typename C>
    void print2nd(const C& container){
        if(container.size() >=2)
            typename C::const_iterator iter(container.begin());//这里的typename表示C::const_iterator是一个类型名称，
                                                               //因为有可能会出现C这个类型里面没有const_iterator这个类型
                                                               //或者C这个类型里面有一个名为const_iterator的变量
    }

所以，在任何时候想要在template中指定一个嵌套从属类型名称（dependent names，依赖于C的类型名称），前面必须添加typename

+ 声明template参数时，前缀关键字class和typename是可以互换的

+ 需要使用typename标识嵌套从属类型名称，但不能在base class lists（基类列）或者member initialization list（成员初始列）内以它作为base class修饰符 

  template<typename T>
  class Derived : public typename Base<T> ::Nested{}//错误的！！！！！

**43. 学习处理模板化基类内的名称 （Know how to access names in templatized base classes)**

原代码：
    

    class CompanyA{
    public:
        void sendCleartext(const std::string& msg);
        ....
    }
    class CompanyB{....}
    
    template <typename Company>
    class MsgSender{
    public:
        void sendClear(const MsgInfo& info){
            std::string msg;
            Company c;
            c.sendCleartext(msg);
        }
    }
    template<typename Company>//想要在发送消息的时候同时写入log，因此有了这个类
    class LoggingMsgSender:public MsgSender<Company>{
        public:
        void sendClearMsg(const MsgInfo& info){
            //记录log
            sendClear(info);//无法通过编译，因为找不到一个特例化的MsgSender<company>
        }
    }

解决方法1（认为不是特别好）：

    template <> // 生成一个全特例化的模板
    class MsgSender<CompanyZ>{  //和一般的template，但是没有sendClear,当Company==CompanyZ的时候就没有sendClear了
    public:
        void sendSecret(const MsgInfo& info){....}
    }

解决方法2（使用this）：

    template<typename Company>
    class LoggingMsgSender:public MsgSender<Company>{
        public:
        void sendClearMsg(const MsgInfo& info){
            //记录log
            this->sendClear(info);//假设sendClear将被继承
        }
    }

解决方法3（使用using）：

    template<typename Company>
    class LoggingMsgSender:public MsgSender<Company>{
        public:
    
        using MsgSender<Company>::sendClear; //告诉编译器，请他假设sendClear位于base class里面
    
        void sendClearMsg(const MsgInfo& info){
            //记录log
            sendClear(info);//假设sendClear将被继承
        }
    }

解决方法4（指明位置）：

    template<typename Company>
    class LoggingMsgSender:public MsgSender<Company>{
        public:
        void sendClearMsg(const MsgInfo& info){
            //记录log
            MsgSender<Company>::sendClear(info);//假设sendClear将被继承
        }
    }

上面那些做法都是对编译器说：base class template的任何特例化版本都支持其一般版本所提供的接口

**44. 将与参数无关的代码抽离templates （Factor parameter-independent code out of templates)**

主要是会让编译器编译出很长的臃肿的二进制码，所以要把参数抽离，看以下代码：
    

    template<typename T, std::size_t n>
    class SquareMatrix{
        public:
        void invert();    //求逆矩阵
    }
    
    SquareMatrix<double, 5> sm1;
    SquareMatrix<double, 10> sm2;
    sm1.invert(); 
    sm2.invert(); //会具现出两个invert并且基本完全相同

修改后的代码：
    

    template<typename T>
    class SquareMatrixBase{
        protected:
        void invert(std::size_t matrixSize);
    }
    
    template<typename T, std::size_t n>
    class SquareMatrix:private SquareMatrixBase<T>{
        private:
        using SquareMatrixBase<T>::invert;  //避免遮掩base版的invert
        public:
        void invert(){ this->invert(n); }   //一个inline调用，调用base class版的invert
    }

当然因为矩阵数据可能会不一样，例如5x5的矩阵和10x10的矩阵计算方式会不一样，输入的矩阵数据也会不一样，采用指针指向矩阵数据的方法会比较好：
    

    template<typename T, std::size_t n>
    class SquareMatrix:: private SquareMatrixBase<T>{
        public:
        SquareMatrix():SquareMatrixBase<T>(n, 0), pData(new T[n*n]){
            this->setDataPtr(pData.get());
        }
        private:
        boost::scoped_array<T> pData; //存在heap里面
    };

总结：

+ templates生成多个classes和多个函数，所以任何template代码都不该与某个造成膨胀的template参数产生依赖关系
+ 因非类型模板参数（non-type template parameters）而造成的代码膨胀，往往可以消除，做法是以函数参数后者class成员变量替换template参数
+ 因类型参数（type parameters）而造成的代码膨胀，往往可以降低，做法是让带有完全相同的二进制表述的具现类型，共享实现码

**45. 运用成员函数模板接受所有兼容类型 （Use member function templates to accept "all compatible types.")**

    Top* pt2 = new Bottom; //将Bottom*转换为Top*是很容易的
    template<typename T>
    class SmartPtr{
        public:
        explicit SmartPtr(T* realPtr);
    };
    SmartPtr<Top> pt2 = SmartPtr<Bottom>(new Bottom);//将SmartPtr<Bottom>转换成SmartPtr<Top>是有些麻烦的

但是我们只是希望SmartPtr<Bottom>转换成SmartPtr<Top>，而不希望SmartPtr<Top>转换成SmartPtr<Bottom>
这种需求可以通过构造模板来实现：
    

    template<typename T>
    class SmartPtr{
    public:
        template<typename U>
        SmartPtr(const SmartPtr<U>& other)  //为了生成copy构造函数
            :heldPtr(other.get()){....}
        T* get() const { return heldPtr; }
    private:
        T* heldPtr;                        //这个SmartPtr持有的内置原始指针
    };

总结:

+ 使用成员函数模板生成“可接受所有兼容类型”的函数
+ 如果还想泛化copy构造函数、操作符重载等，同样需要在前面加上template

**46. 需要类型转换时请为模板定义非成员函数 （Define non-member functions inside templates when type conversions are desired)**

像第24条一样，当我们进行混合类型算术运算的时候，会出现编译通过不了的情况
    

    template<typename T>
    const Rational<T> operator* (const Rational<T>& lhs, const Rational<T>& rhs){....}
    
    Rational<int> oneHalf(1, 2);
    Rational<int> result = oneHalf * 2; //错误，无法通过编译

解决方法：使用friend声明一个函数,进行混合式调用
    

    template<typename T>
    class Rational{
        public:
        friend const Rational operator*(const Rational& lhs, const Rational& rhs){
            return Rational(lhs.numerator()*rhs.numerator(), lhs.denominator() * rhs.denominator());
        }
    };
    template<typename T>
    const Rational<T> operator*(const Rational<T>& lhs, const Rational<T>&rhs){....}

总结：

+ 当我们编写一个class template， 而他所提供的“与此template相关的”函数支持所有参数隐形类型转换时，请将那些函数定义为classtemplate内部的friend函数

**47. 请使用traits classes表现类型信息 （Use traits classes for information about types)**

traits是一种允许你在编译期间取得某些类型信息的技术，或者受是一种协议。这个技术的要求之一是：他对内置类型和用户自定义类型的表现必须是一样的。
    

    template<typename T>
    struct iterator_traits;  //迭代器分类的相关信息
                             //iterator_traits的运作方式是，针对某一个类型IterT，在struct iterator_traits<IterT>内一定声明//某个typedef名为iterator_category。这个typedef 用来确认IterT的迭代器分类
    一个针对deque迭代器而设计的class大概是这样的
    template<....>
    class deque{
        public:
        class iterator{
            public:
            typedef random_access_iterator_tag iterator_category;
        }
    }
    对于用户自定义的iterator_traits，就是有一种“IterT说它自己是什么”的意思
    template<typename IterT>
    struct iterator_traits{
        typedef typename IterT::iterator_category iterator_category;
    }
    //iterator_traits为指针指定的迭代器类型是：
    template<typename IterT>
    struct iterator_traits<IterT*>{
        typedef random_access_iterator_tag iterator_category;
    }

综上所述，设计并实现一个traits class：

+ 确认若干你希望将来可取得的类型相关信息，例如对迭代器而言，我们希望将来可取得其分类
+ 为该信息选择一个名称（例如iterator_category）
+ 提供一个template和一组特化版本（例如iterator_traits)，内含你希望支持的类型相关信息

在设计实现一个traits class以后，我们就需要使用这个traits class：
    

    template<typename IterT, typename DistT>
    void doAdvance(IterT& iter, DistT d, std::random_access_iterator_tag){ iter += d; }//用于实现random access迭代器
    template<typename IterT, typename DistT>
    void doAdvance(IterT& iter, DistT d, std::bidirectional_iterator_tag){ //用于实现bidirectional迭代器
        if(d >=0){
            while(d--)
                ++iter;
        }
        else{
            while(d++)
                --iter;
        }
    }
    
    template<typename IterT, typename DistT>
    void advance(IterT& iter, DistT d){
        doAdvance(iter, d, typename std::iterator_traits<IterT>::iterator_category());
    }

使用一个traits class:

+ 建立一组重载函数（像劳工）或者函数模板（例如doAdvance），彼此间的差异只在于各自的traits参数，令每个函数实现码与其接受traits信息相应
+ 建立一个控制函数（像工头）或者函数模板（例如advance），用于调用上述重载函数并且传递traits class所提供的信息

**48. 认识template元编程 （Be aware of template metaprogramming)**

Template metaprogramming是编写执行于编译期间的程序，因为这些代码运行于编译器而不是运行期，所以效率会很高，同时一些运行期容易出现的问题也容易暴露出来
    

    template<unsigned n>
    struct Factorial{
        enum{
            value = n * Factorial<n-1>::value
        };
    };
    template<>
    struct Factorial<0>{
        enum{ value = 1 };
    };                       //这就是一个计算阶乘的元编程


#### 