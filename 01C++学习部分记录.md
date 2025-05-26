- ## 菱形继承

virtual修饰，代表虚继承，会使A_1多占4个字节的内存，用来存放虚指针。使用virtual修饰继承关系，确保共享的基类在派生类中仅存在 一个副本。

![image-20250507100001133](C:\Users\xinmojun\AppData\Roaming\Typora\typora-user-images\image-20250507100001133.png)

# 六 多态

## 1.联编 

### 1.动态联编（动态约束 or 晚期联编）

​	出现在程序运行的时候

### 2.静态联编（静态约束 or 早期联编）

​	在写代码的时候就确定了

### 3.实现动态联编的条件：

**先决条件：**

1： 要有一个拥有虚函数成员的那么一个类。

2：要有继承关系，拥有虚函数的这个类得是父类。



3：通过基类指针或引用调用

## 2.多态

1. 多态：多种状态 多种形态

2. 虚函数以及虚函数的特点：
   虚函数：virtual 返回值类型 函数名称（参数表）
   虚函数特点：
   1. 动态绑定：虚函数允许通过**基类指针或引用**调用派生类的函数实现。程序在**运行时**根据对象的实际类型决定调用哪个版本的函数，而非编译时确定。

   2.虚函数表机制：
        虚函数表：每个包含虚函数的类都有一个虚函数表，存储该类所有虚函数的地址。

   ​     虚指针（vptr）：对象实例中包含指向虚函数表的指针，调用虚函数时通过vptr查表找到具体函数地址。

   3.派生类覆盖规则

   基类声明为`virtual`的函数，派生类**可重写**（override），且**自动继承虚函数属性**（即使不显式加`virtual`关键字）。

   要求函数签名（函数名、参数列表、返回类型）完全一致（协变返回类型除外）。

   4.默认实现与灵活性

   - 基类虚函数可提供默认实现，派生类可选择覆盖或继承基类行为。
   - 与纯虚函数不同，包含虚函数的类仍是具体类，可直接实例化。

   5.使用限制

   1. 

      不能为虚函数的情形：

      - 构造函数（对象未完全初始化时无法确定虚表）
      - 静态成员函数（无this指针，与对象无关）
      - 友元函数（非成员函数）

   2. 

      内联函数冲突：虚函数动态绑定机制与内联函数的编译期展开冲突，但虚函数本身可被编译器优化为内联。

   6.设计意义

   - 在面向对象设计中，虚函数是多态性的技术基础，广泛用于接口抽象、设计模式（如工厂方法、策略模式）。
   - 允许程序调用编译时未知的函数，增强代码扩展性。

**###**虚函数调用需通过虚表间接寻址，相比普通函数有**额外开销**，但现代编译器（如GCC、Clang）会通过**虚函数内联优化（devirtualization）** 减少开销

通过类的继承和虚函数实现多态例子：

```C++
#include <iostream>
using namespace std;

class Father
{
public:
	Father();
	~Father();

	virtual void test_func();

private:

};

class son_1 : public Father
{
public:
	son_1();
	~son_1();
	void test_func();

private:

};

class son_2 : public Father
{
public:
	son_2();
	~son_2();
	void test_func();
private:

};

int main()
{
	Father obj_father;
	son_1 obj_son1;
	son_2 obj_son2;
	Father* p_father;
	p_father = &obj_father;
	p_father->test_func();

	p_father = &obj_son1;
	p_father->test_func();

	p_father = &obj_son2;
	p_father->test_func();

	return 0;
}

/** Father **/
void Father::test_func()
{
	cout << "Father::test_func()" << endl;
}

Father::Father()
{
}

Father::~Father()
{
}

/** son_1 **/
void son_1::test_func()
{
	cout << "son_1::test_func()" << endl;
}

son_1::son_1()
{
}

son_1::~son_1()
{
}

/** son_2 **/
void son_2::test_func()
{
	cout << "son_2::test_func()" << endl;
}

son_2::son_2()
{
}

son_2::~son_2()
{
}
```

多态使用过程中，内存泄漏的场景：

1. **基类析构函数未声明为虚函数**
   当通过基类指针删除派生类对象时，若基类析构函数非虚，编译器会执行静态绑定，仅调用基类的析构函数，而不会调用派生类的析构函数。这导致派生类中独有的资源（如动态内存、文件句柄等）无法释放。例如：

   ```cpp
   Base* obj = new Derived();
   delete obj;  // 若基类析构非虚，仅调用Base::~Base()
   ```

   此时，`Derived`类中通过`new`分配的内存或资源会泄漏。

2. **多态对象未正确释放**
   即使基类析构函数是虚的，若程序逻辑未触发`delete`操作（如异常导致提前退出、循环中遗漏释放等），仍会导致内存未释放。例如：

   ```cpp
   try {
       Base* obj = new Derived();
       throw std::runtime_error("Error");
       delete obj;  // 未执行到此处
   } catch (...) { /* 未处理obj释放 */ }
   ```

### 二、为何必须将析构函数声明为虚函数？

1. **动态绑定确保正确析构链**
   虚析构函数通过虚函数表（vtable）实现动态绑定。当通过基类指针调用`delete`时，实际执行流程为：
   - 查找对象的虚表，定位到派生类的析构函数
   - 调用`Derived::~Derived()`释放派生类资源
   - 自动调用Base::~Base() 释放基类资源
   - 这一机制保证了完整的析构链，避免资源泄漏。

**2.****多态设计的必要约束**
若一个类可能被继承并通过基类指针操作，其析构函数必须为虚函数。否则，违反多态对象生命周期管理的原则，导致派生类资源无法释放。例如：

```cpp
class AbstractBase {
public:
    virtual ~AbstractBase() = default;  // 虚析构确保派生类正确释放
    virtual void process() = 0;
};
```

## 3.纯虚函数

纯粹的虚函数（是一种特殊的虚函数）没有函数体的虚函数

**纯虚函数**：virtual 返回值类型 函数名称（参数表）= 0；     **这样就定义了一个纯虚函数**

1. 自己不去实现这个函数体，让子类去重写父类虚函数的时候，来实现这个函数体。

2. 析构函数写成纯虚函数的话，还是要在这个类里实现一下。

**抽象类**：只要一个普通类有一个或多个纯虚函数，就称为抽象类。

1. 抽象类不能实例化对象（有纯虚函数，类的定义不完整），但是可以定义指针（为子类去服务）。
2. 抽象类的特性可以遗传，只要子类没有把父类的纯虚函数全部实例化，那么子类也是抽象类。



虚函数 可由 子类重写，也可以加virtual后，孙子类也可以继续重写虚函数。子类不加virtual，在孙子类里也可以重写虚函数。

## 4.final

1. 权限掠夺者
2. 掠夺函数权限：阻止重写
3. 掠夺类的权限：阻止派生

**2.掠夺函数权限，在函数上使用的例子**

```C++
class Son : public Father
{
public: 
	Son();
	~Son();
	void test_func() final;

private:

};
```

```C++
#include <iostream>
using namespace std;

class Father
{
public:
	Father();
	~Father();

	virtual void test_func();

private:

};

class Son : public Father
{
public: 
	Son();
	~Son();
	void test_func() final;

private:

};

class G_Son : public Son
{
public:
	G_Son();
	~G_Son();
	// 子类说明虚函数是final，那么孙子类就不能重写test_func了
	//void test_func();
private:

};

int main()
{
	Father obj_father;
	Son obj_Son;
	G_Son obj_G_Son;
	Father* p_father;
    
	p_father = &obj_father;
	p_father->test_func();

	p_father = &obj_Son;
	p_father->test_func();
	return 0;
}

/** Father **/
void Father::test_func()
{
	cout << "Father::test_func()" << endl;
}

Father::Father()
{
}

Father::~Father()
{
}

/** Son **/
void Son::test_func()
{
	cout << "Son::test_func()" << endl;
}

Son::Son()
{
}

Son::~Son()
{
}

/** G_Son **/

G_Son::G_Son()
{
}

G_Son::~G_Son()
{
}
```

**3.掠夺类的权限**

```C++
class Father final
{
public:
	Father();
	~Father();

	virtual void test_func();

private:

};
```

Fazther就没有资格再做基类了，它不可能再有子类了。

# 七 运算符重载

目的：

1. 掌握运算符重载规则
2. 能够自主实现运算符重载
3. 通过重载运算符，扩展运算符的功能

内容：

1. 基础语法
2. 规则详解
3. 重载>>和<<
4. 注意事项

重点：

1. 基础规则
2. 注意事项

## 1.基础语法

1.重载的概念：

​	重新赋予它一个含义（内容）

**2.语法规则：**

​	返回值类型 函数名 （形参列表）{}

​	**函数名**：operator运算符名称

**3.在类中重载运算符例子：**

```C++
#include <iostream>
using namespace std;

class MyComplex
{
public:
	MyComplex(double real=0.0, double imag=0.0);
	~MyComplex();
	void Display() const;
	// 返回值类型 函数名 （形参列表）{}
	// **函数名**：operator运算符名称
	MyComplex operator+(MyComplex other)
	{
		MyComplex obj;
		obj.o_real = this->o_real + other.o_real;
		obj.o_imag = this->o_imag + other.o_imag;
		return obj;
	}

private:
	double o_real; // 实部
	double o_imag; // 虚部
};

int main(void)
{
	MyComplex obj_1(3,5);
	obj_1.Display();

	MyComplex obj_2(4, 6);
	obj_2.Display();

	MyComplex obj_3;
	obj_3.Display();

	obj_3 = obj_1 + obj_2;
	obj_3.Display();

	return 0;
}

/** MyComplex **/
MyComplex::MyComplex(double real, double imag)
	:o_real(real), o_imag(imag)
{
}

MyComplex::~MyComplex()
{
}
void MyComplex::Display() const
{
	cout << "(" << o_real << "+" << o_imag << "i)" << endl;
}
```

```C++
#include <iostream>
using namespace std;

class MyComplex
{
public:
	MyComplex(double real=0.0, double imag=0.0);
	~MyComplex();
	void Display() const;
	// 返回值类型 函数名 （形参列表）{}
	// **函数名**：operator运算符名称
	/*MyComplex operator+(MyComplex other)
	{
		MyComplex obj;
		obj.o_real = this->o_real + other.o_real;
		obj.o_imag = this->o_imag + other.o_imag;
		return obj;
	}*/
	MyComplex operator+(const MyComplex& other) const;

private:
	double o_real; // 实部
	double o_imag; // 虚部
};



int main(void)
{
	MyComplex obj_1(3,5);
	obj_1.Display();

	MyComplex obj_2(4, 6);
	obj_2.Display();

	MyComplex obj_3;
	obj_3.Display();

	obj_3 = obj_1 + obj_2;
	obj_3.Display();

	return 0;
}

/** MyComplex **/
MyComplex::MyComplex(double real, double imag)
	:o_real(real), o_imag(imag)
{
}

MyComplex::~MyComplex()
{
}
void MyComplex::Display() const
{
	cout << "(" << o_real << "+" << o_imag << "i)" << endl;
}
MyComplex MyComplex::operator+(const MyComplex& other) const
{
	return MyComplex(this->o_real + other.o_real, this->o_imag + other.o_imag);
}
```

**4.在全局重载运算符：**

```C++
#include <iostream>
using namespace std;

class MyComplex
{
public:
	MyComplex(double real=0.0, double imag=0.0);
	~MyComplex();
	void Display() const;
	// 返回值类型 函数名 （形参列表）{}
	// **函数名**：operator运算符名称
	/*MyComplex operator+(MyComplex other)
	{
		MyComplex obj;
		obj.o_real = this->o_real + other.o_real;
		obj.o_imag = this->o_imag + other.o_imag;
		return obj;
	}*/
	MyComplex operator+(const MyComplex& other) const;
	friend MyComplex operator+(const MyComplex& obj1, const int num);
private:
	double o_real; // 实部
	double o_imag; // 虚部
};

MyComplex operator+(const MyComplex& obj1, const int num);

int main(void)
{
	MyComplex obj_1(3,5);
	obj_1.Display();

	MyComplex obj_2(4, 6);
	obj_2.Display();

	MyComplex obj_3;
	obj_3.Display();

	obj_3 = obj_1 + obj_2;
	obj_3.Display();

	obj_3 = obj_3 + 2; // 使用全局重载运算符
	obj_3.Display();

	return 0;
}

/** MyComplex **/
MyComplex::MyComplex(double real, double imag)
	:o_real(real), o_imag(imag)
{
}

MyComplex::~MyComplex()
{
}
void MyComplex::Display() const
{
	cout << "(" << o_real << "+" << o_imag << "i)" << endl;
}
MyComplex MyComplex::operator+(const MyComplex& other) const
{
	return MyComplex(this->o_real + other.o_real, this->o_imag + other.o_imag);
}


/*****/
MyComplex operator+(const MyComplex& obj1, const int num)
{
	return MyComplex(obj1.o_real + num, obj1.o_imag);
}
```

## 2.规则详解

**1.并不是所有的运算符都可以重载。**

**基础类不能重载的运算符**包括：

1. **成员访问运算符`.`**
   - 用于访问类/结构体成员（如`obj.member`）
   - 若允许重载，会破坏对象成员的直接访问语义
2. **成员指针运算符`.*`和`->***`**
   - 用于通过成员指针访问对象（如`ptr->*mem_ptr`）
   - 涉及底层指针操作，重载会导致不可预测的副作用
3. **作用域解析运算符`::`**
   - 用于限定命名空间或类的成员（如`Class::static_member`）
   - 由编译器在编译期静态解析，与运行时多态无关
4. **条件运算符`?:`**
   - 三元运算符（如`a ? b : c`）
   - 语法结构复杂，重载易引发逻辑混乱

**编译期相关不可重载运算符**包括：

1. **`sizeof`运算符**
   - 计算类型或对象的内存大小（如`sizeof(int)`）
   - 由编译器在编译期直接计算，无法动态修改
2. **类型信息运算符`typeid`**
   - 获取对象的类型信息（如`typeid(obj).name()`）
   - 与运行时类型识别（RTTI）紧密相关，不可自定义

**2.重载不能改变运算符的优先级和结合性。**

**3.重载不会改变运算符的用法。**

（之前是单目运算符，重载之后还是单目运算符。之前是双目运算符，重载之后还是双目运算符。）（之前操作数在左边，重载之后操作数还是在左边。之前在右边，重载之后操作数还是在右边。）

**4.运算符重载函数不能有默认的参数。**

**5.运算符重载函数可以作为类的成员函数，也可以作为全局函数。**

| **特性**     | **成员函数**                      | **全局函数**                   |
| :----------- | :-------------------------------- | :----------------------------- |
| 参数传递     | 隐式 `this` + 显式参数            | 显式传递左右操作数             |
| 访问私有成员 | 直接访问                          | 需声明为友元                   |
| 适用运算符   | `=`, `[]`, `()`, `->`, 单目运算符 | `<<`, `>>`, 需对称性的运算符   |
| 左操作数类型 | 必须为本类对象                    | 可以是任意类型（包括内置类型） |

**6.箭头运算符->  、 下标运算符[] 、 赋值运算符= 、 函数调用运算符()      只能以成员函数的形式重载**

## 3.重载>>与<<

1.作为友元函数重载（全局重载）

2.istream：输入

3.ostream：输出

```C++
#include <iostream>
using namespace std;

class MyComplex
{
public:
	MyComplex(double real=0.0, double imag=0.0);
	~MyComplex();
	friend istream& operator>>(istream& in, MyComplex& com);
	friend ostream& operator<<(ostream& out, MyComplex& com);
private:
	double o_real; // 实部
	double o_imag; // 虚部
};

istream& operator>>(istream& in, MyComplex& com)
{
	in >> com.o_real >> com.o_imag;
	return in;
}
ostream& operator<<(ostream& out, MyComplex& com)
{
	out << "(" << com.o_real << "+" << com.o_imag << "i)";
	return out;
}

int main(void)
{
	MyComplex obj;
	MyComplex obj1;
	operator>>(operator>>(cin, obj), obj1); // cin>>obj>>obj1;
	cout << obj << endl << obj1 << endl;
	return 0;
}

/** MyComplex **/
MyComplex::MyComplex(double real, double imag)
	:o_real(real), o_imag(imag)
{
}

MyComplex::~MyComplex()
{
}

```

## 4.注意事项

1.运算发重载的语法很简单，关键是规则。

2.可以显示调用，也可以隐式调用。

显示调用（直接调用函数）

隐式调用（使用运算符调用）

3.注意与友元的联合使用

# 模板

目的：

1.了解并理解模板的概念。

2.熟悉函数模板的使用。

2.熟悉类模板的使用。

内容：

模板的概念。

**函数模板。**

**类模板。**

模板与友元

## 1.模板的概念

1.模板与泛型编程

**泛型编程**：编写和类型无关的逻辑代码

**模板**：实现代码重用的一种工具。可以实现类型参数化的操作

**模板是泛型编程的基础。**

2.模板分类

​	函数模板：模板的一种，使用函数模板可以定义出模板函数。

​	类模板：模板的一种，使用类模板可以定义出模板类。

## 2.函数模板

1.函数模板是什么

​	是一种写代码的方法、工具。可以写出一些不完整的模板函数

2.函数模板的定义语法

​	通过 函数模板 定义 模板函数

​	**template <typename Type1,....,typename Type_n>**
​	**返回值类型 函数名(形参列表)**
​	**{**
​		**函数体;**
​	**}**

​	**template**：定义模板的关键字
​	**typename**：声明类型参数的关键字
​	<>  ：类型的参数列表

​	显式调用：**函数名<类型列表>(实参列表)**

​	隐式调用：**函数名(实参列表)**

3.函数模板示例代码

```C++
#include <iostream>
using namespace std;

template <typename T1>
T1 add(T1 a, T1 b)
{
	return a + b;
}

template <typename TT1,typename TT2>
void test(TT1 tt1,TT2 tt2)
{
	cout << "tt1 = " << tt1 << endl 
		<< "tt2 = " << tt2 << endl;
}

int main()
{
	test(1, 2);
	cout << add<int>(6, 2.3) << endl;
	cout << add<double>(6, 2.3) << endl;
	return 0;
}

/*
通过 函数模板 定义 模板函数

template <typename Type1,....,typename Type_n>
返回值类型 函数名(形参列表)
{
	函数体;
}

template：定义模板的关键字
typename：声明类型参数的关键字
<>  ：类型的参数列表
调用：
	函数名<类型列表>(实参列表)
*/
```

4.函数模板和普通函数对比

- 类型转换与参数匹配

​	**普通函数**支持**隐式类型转换**（如`int`与`char`相加时，自动将`char`转为ASCII码值），参数类型不严格匹配时仍可调用。

​	**模板函数** **自动类型推导**：要求参数类型**完全一致**，否则编译报错。**显式指定类型**：允许隐式转换，如`add<int>(10, 'c')`会将`'c'`转为`int`后调用。

- 调用规则与优先级

​	**普通函数优先**当普通函数和模板均匹配时，编译器优先调用**普通函数**（即使模板更高效）。

​	**强制调用模板**通过空模板参数列表`func<>(10)`或显式类型`func<int>(10)`强制使用模板。

​	**模板更好匹配时优先**若模板参数匹配更精准（如无需类型转换），则选择模板。



- **重载能力**普通函数和模板均可重载，但模板的重载需基于参数数量或类型。

| **场景**                 | **推荐方式**       | **原因**                 |
| :----------------------- | :----------------- | ------------------------ |
| 类型严格匹配且需复用逻辑 | 函数模板           | 泛用性强，支持多种类型   |
| 需隐式类型转换           | 普通函数           | 避免模板显式指定的复杂性 |
| 高性能或特定类型优化     | 普通函数或模板特化 | 减少模板实例化开销       |
| 需要强制类型控制         | 模板显式实例化     | 精确控制参数类型         |

**建议**：优先使用函数模板提升灵活性，但在需要隐式转换或特定优化时选择普通函数。

5.模板的局限性

## 3.类模板

示例：

```C++
#include <iostream>
using namespace std;

template <class Type1 = int, class Type2 = double>
//template <class Type1, class Type2>
class MyData
{
public:
	MyData(Type1 n = 0, Type2 v = 0) :num(n), val(v) {}
	Type1 GetNum() { return num; }
	Type2 GetVal() { return val; }
	void SetNum(Type1 n) { num = n; }
	void SetNum(Type2 v) { val = v; }

	void ShowData();

private:
	Type1 num;
	Type2 val;
};
template <class Type1, class Type2>
void MyData<Type1, Type2>::ShowData()
{
	cout << "num = " << num << endl << "val = " << val << endl;
}

int main()
{
	MyData<> data_1(3, 3.14);
	data_1.ShowData();

	MyData<char> data_2(65, 3.14);
	data_2.ShowData();
	
	return 0;
}
/*

template <类型参数列表>
class 模板类名
{
	成员：
};

类型参数列表：<class T1,.....,class Tn>
成员：T1 name;........
*/
```

4.类模板做函数参数

​	1.方法一 **指定传入类型（显式实例化）**       **定义方式**：在函数参数列表中直接声明类模板的**具体类型参数**。

```c++
void test_Func_1(MyData<int, double>& obj)
{
	obj.ShowData();
}
```

​	2. 方法二 **参数模板化（部分模板化）**         **定义方式**：将类模板的模板参数**作为函数模板参数**传递，保留泛型特性。

```C++
template <typename TT1, typename TT2>
void test_Func_2(MyData<TT1, TT2>& obj)
{
	obj.ShowData();
}
```

​	3.方法三**整个类模板化**                    **定义方式**：将整个类模板的实例化类型**作为函数模板参数**传递。

```C++
template <class T>
void test_Func_3(T& obj)
{
	obj.ShowData();
}
```

### **对比与适用场景**

| **方法**     | **优点**             | **缺点**                   | **适用场景**                 |
| :----------- | :------------------- | :------------------------- | :--------------------------- |
| 指定传入类型 | 类型明确，安全性高   | 灵活性低，无法适配其他类型 | 已知具体类型，需类型安全保证 |
| 参数模板化   | 部分泛化，复用性较强 | 需处理多模板参数推导       | 需适配不同参数的同类模板对象 |
| 整个类模板化 | 完全泛化，灵活性最高 | 类型检查较弱，调试复杂度高 | 处理多种类模板实例或未知类型 |

```C++
#include <iostream>
using namespace std;

template <class Type1 = int, class Type2 = double>
//template <class Type1, class Type2>
class MyData
{
public:
	MyData(Type1 n = 0, Type2 v = 0) :num(n), val(v) {}
	Type1 GetNum() { return num; }
	Type2 GetVal() { return val; }
	void SetNum(Type1 n) { num = n; }
	void SetNum(Type2 v) { val = v; }

	void ShowData();

private:
	Type1 num;
	Type2 val;
};
template <class Type1, class Type2>
void MyData<Type1, Type2>::ShowData()
{
	cout << "num = " << num << " " << "val = " << val << endl;
}

void test_Func_1(MyData<int, double>& obj)
{
	obj.ShowData();
}

template <typename TT1, typename TT2>
void test_Func_2(MyData<TT1, TT2>& obj)
{
	obj.ShowData();
}

template <class T>
void test_Func_3(T& obj)
{
	obj.ShowData();
}

int main()
{
	MyData<> data_1(3, 3.14); 
	MyData<char> data_2(65, 3.1415);
	MyData<char, int> data_3(66, 2);
	test_Func_1(data_1);

	test_Func_2(data_1);
	test_Func_2(data_2);
	test_Func_2<char,double>(data_2);

	test_Func_3(data_1);
	test_Func_3(data_2);
	test_Func_3(data_3);
	test_Func_3<MyData<char, int>>(data_3);
	
	return 0;
}
```

5.类模板和继承

```C++
#include <iostream>
using namespace std;

template <class F_Type>
class Father
{
public:
	F_Type m_F_val;
};

// 模板类派生普通类
class Son : public Father<int>
{
public:
	int m_S_val;
};

// 模板类派生模板类
template <class S_Type1, class S_Type2>
class Son_1 : public Father<S_Type1>
{
public:
	S_Type2 m_S_val;
};

int main()
{
	Son_1<int, double> obj_son;
	obj_son.m_F_val;
	obj_son.m_S_val;
	return 0;
}
```

6.注意事项

**延迟实例化特性**
	模板代码本身不是完整的类型或函数，而是编译器生成具体代码的“蓝图”。当编译器遇到模板的具体实例化请求（如 `MyClass<int> obj;`）时，**必须能立即访问模板的完整定义**，才能根据类型参数生成对应的二进制代码

所以**写模板类的时候，要把成员函数的声明和实现，写到同一个文件里。**

**默认方案**：将模板的声明和实现统一写入头文件（`.h` 或 `.hpp`），如 STL 的实现方式。

## 4.模板类与友元函数

模板类的友元函数 一：

```c++
#include <iostream>
using namespace std;

template <class T>
class A
{
public:
	A(T a = 0) : m_a(a) {}
private:
	T m_a;
	// 在模板类里直接声明友元函数
	friend void showData(A<int>& a)
	{
		cout << a.m_a << endl;
	}
};

int main()
{
	A<int> a(666);
	showData(a);
	return 0;
}
```

模板类的友元函数 二：

```c++
#include <iostream>
using namespace std;

template <class T>
class A;
template <typename TT>
void showData2(A<TT>& a);

template <class T>
class A
{
public:
	A(T a = 0) : m_a(a) {}
private:
	T m_a;

	// 在类里定义
	friend void showData2<>(A<T>& a);
};
// 在类外实现
template <typename TT>
void showData2(A<TT>& a)
{
	cout << a.m_a << endl;
}
int main()
{
	A<int> a(666);
	showData2(a);
	return 0;
}
```

# 知识点补充

目的：

1.知道C++异常时的处理，及其过程。

2.知道流的概念，会使用对文件进行操作。

3.知道C++11新标准。

内容：

异常处理

文件流

11标准

重点难点：

难点：文件流

重点：11标准

## 1.异常处理机制

1.程序异常：在它执行期间内发生了一些问题。

2.C++中的异常：

​	throw：抛出异常

​	try：尝试一些可能会有异常的代码

​	catch：接收抛出的异常

使用语法：

try

{

​	直接或者间接有 throw

}

catch（接收异常提示）

{

​	处理

}

例子：

```c++
#include <iostream>
using namespace std;

double test(double a, double b)
{
	if (b == 0.0)
	{
		// 有异常
		// throw 抛出的异常信息；
		// 抛出的异常信息：支持多种类型
		throw "这里有问题！";
		// 如歌抛出异常 后面的语句 不会执行
	}
	return a / b;
}

int  main()
{
	try
	{
		cout << test(9.0, 15) << endl;
	}
	
	catch (const char* str)
	{
		cout << str << endl;
	}
	catch (int num)
	{
		cout << num << endl;
	}
	catch (...) // 不知道抛出什么类型错误
	{
		cout << "不对劲。" << endl;
	}
	

	return 0;
}
```

3.使用自定义异常

C++异常处理的共同父类：exception

![image-20250509142729557](C:\Users\xinmojun\AppData\Roaming\Typora\typora-user-images\image-20250509142729557.png)

## 2.文件流

1.流的概念

​	先有流的概念，才有无结构化的传递。

2.fstream的使用（C++中常用的文件操作类）

3.常用的成员函数

![image-20250509160144858](C:\Users\xinmojun\AppData\Roaming\Typora\typora-user-images\image-20250509160144858.png)

```c++
#include <iostream>
using namespace std;
#include <fstream>

int  main()
{
	fstream obj;

	obj.open("test_1.txt",ios::out);
	obj.put('S');
	char ch = 'A';
	obj.put(ch);
	obj.close();
	//obj.is_open();
	//obj.eof(); // end of file
	obj.open("test_1.txt", ios::in);
	obj.get(ch);
	cout << ch << endl;
	ch = obj.get();
	cout << ch << endl;
	obj.close();

	return 0;
}
```

二进制格式操作：

```c++
#include <iostream>
using namespace std;
#include <fstream>

int  main()
{
	fstream obj;

	obj.open("test_2.txt",ios::out);
	int num = 99;
	obj.write((const char*)&num, sizeof(int));
	obj.close();
	//obj.is_open();
	//obj.eof(); // end of file
	obj.open("test_2.txt", ios::in);
	int val = 0;
	obj.read((char*)&val, sizeof(int));
	obj.close();
	cout << "val = " << val << endl;

	return 0;
}
```

4.使用重载的<<   >> 

```c++
#include <iostream>
using namespace std;
#include <fstream>

int  main()
{
	fstream obj;

	obj.open("test_3.txt",ios::out);
	obj << "勇敢困难,不怕牛牛！" << endl;
	obj << "123" << endl;
	obj.close();

	char str[64];
	int num = 0;
	obj.open("test_3.txt", ios::in);
	obj >> str;
	obj >> num;
	obj.close();
	cout << str << endl;
	cout << num << endl;

	return 0;
}
```

