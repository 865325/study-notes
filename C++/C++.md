## 基础知识

### 注释

- 单行注释：// 描述信息
	- 放在代码上方或语句尾，对该行代码进行说明
- 多行注释：/\*描述信息*/
	- 放在代码上方，对代码做整体说明

### 常量

```c
// 通常在文件上方定义，表示一个常量
#define 常量名	常量值

// 不可修改
const 数据类型 常量名 = 常量值
```

### 标识符

标识符不能是关键字，只能由字母、数字和下划线组成，第一个字符必须为字母或下划线

### 数据类型

| 数据类型  | 占用空间               | 取值范围        |
| --------- | ---------------------- | --------------- |
| short     | 2                      | -2^15 ~ 2^15-1  |
| int       | 4                      | -2^31 ~ 2^31-1  |
| long      | window(4), linux(4或8) | -2^31 ~ 2^31-1  |
| long long | 8                      | -2^63 ~ 2^63-1  |
| float     | 4                      | 7位有效数字     |
| double    | 8                      | 15~16位有效数字 |
| char      | 1                      | -128-127        |
| bool      | 1                      | 0 或 1          |

### 三目运算符

```c
// 表达式1正确的话执行表达式2并返回结果，否则表达式3
表达式1 ? 表达式2 : 表达式3
```

### switch语句

```c
// 表达式只能为整型或者字符型
switch(表达式){
    case 结果1: 执行语句;break;
    case 结果2: 执行语句;break;
    ...
    default: 执行语句;break;
}
```

### 指针

指针存放的是地址，通过 * 操作符可以操作指针指向的内存，该过程叫做解决引用

指针所占内存空间由操作系统的位数所决定，如32位操作系统下是4字节

```c
// 空指针，指向内存中编号为0的空间，该空间不可被访问
int *p = NULL;

// 野指针，指向非法（未分配）的内存空间
int *p = (int *)0x1100;

// 常量指针——const修饰指针，指针指向可以改变，指针指向的值不可以改变
const int * p = &a;
p = &b;		// 正确
*p = 100;	// 错误

// 指针常量——const修饰常量，指针指向不可以改变，指针指向的值可以改变
int * const p = &a;
p = &b;		// 错误
*p = 100;	// 正确

// const既修饰常量又修饰指针，都不可以改变
const int * const p = &a;
p = &b;		// 错误
*p = 100;	// 错误
```

## 核心编程

### 内存划分

![img](.\assets\73a96d54c02c919cf7d4823a152356c2.png)

### 引用

作用：给变量起别名

语法：数据类型 &别名 = 原名

注意：引用必须初始化；引用不可以更改

引用作为函数参数的效果和地址传递一样，可以修改原值

引用可以作为函数的返回值存在的，但不要返回局部变量的引用

引用的本质在C++内部实现是一个指针常量

### 函数提高

#### 函数默认参数

形参可以有默认值

如果某个位置参数有默认值，那么从该位置往后，必须有默认值

如果函数声明有默认值，那么函数实现的时候就不能有默认参数

```c
// 声明
int func(int a = 10, int b = 10);

// 实现
int func(int a, int b){
    return a + b;
}
```

#### 函数占位参数

函数列表中可以有占位参数，用于占位，调用函数必须填补该位置

占位参数也可以有默认参数

```c
void func(int, int a);
```

#### 函数重载

- 作用域相同
- 函数名称相同
- 函数参数**类型**不同 or **个数**不同 or **顺序**不同
- 函数的返回值不可以作为重载的条件

注意事项

- 引用作为重载条件

```c
// int 和 const int 不能作为重载条件
void func(int &a);
void func(const int &a);

int main(){
    int a = 100;
    func(a);	// 调用无const
    func(10);	// 调用有const
}
```

- 函数默认参数作为重载条件

```c
void func(int a);
void func(int a, int b = 10);

int main(){
    # func(10);	// 有歧义，需避免
}
```

### 类和对象

#### 封装

将属性和行为作为一个整体，且对其加以权限控制

访问权限有三种

- public		公共权限
- protected 保护权限
- private       私有权限

#### 构造函数

主要作用在于创建对象时为对象的成员属性赋值，构造函数由编译器自动调用，无须手动调用

- 无返回值也不需要写void
- 函数名称与类名一致
- 构造函数可以有参数，可重载
- 程序创建对象时自动调用构造，无须手动调用，且只会调动一次

##### 分类

- 按参数划分：有参构造和无参构造
- 按类型划分：普通构造和拷贝构造

##### 调用方式

- 括号法
- 显示法
- 隐式转换法

```c
class Person {
public:
    //无参（默认）构造函数
    Person() {
        cout << "无参构造函数!" << endl;
    }
    //有参构造函数
    Person(int a) {
        age = a;
        cout << "有参构造函数!" << endl;
    }
    //拷贝构造函数
    Person(const Person& p) {
        age = p.age;
        cout << "拷贝构造函数!" << endl;
    }
    //析构函数
    ~Person() {
        cout << "析构函数!" << endl;
    }
public:
    int age;
};


//调用构造函数
void test02() {
    //调用无参构造函数
    Person p;
    //2.1 括号法，常用
    Person p1(10);
    //注意1：调用无参构造函数不能加括号，如果加了编译器认为这是一个函数声明
    //Person p2();
    //2.2 显式法
    Person p2 = Person(10);
    Person p3 = Person(p2);
    //Person(10)单独写就是匿名对象 当前行结束之后，马上析构
    //2.3 隐式转换法
    Person p4 = 10; // Person p4 = Person(10);
    Person p5 = p4; // Person p5 = Person(p4);
    //注意2：不能利用 拷贝构造函数 初始化匿名对象 编译器认为是对象声明
    //Person p5(p4);
}

```

##### 默认情况下，c++编译器至少给一个类添加3个函数

- 默认构造函数(无参，函数体为空) 
- 默认析构函数(无参，函数体为空) 
- 默认拷贝构造函数，对属性进行值拷贝

##### 构造函数调用规则如下

- 如果用户定义有参构造函数，c++不在提供默认无参构造，但是会提供默认拷贝构造

- 如果用户定义拷贝构造函数，c++不会再提供其他构造函数

##### 深拷贝和浅拷贝

浅拷贝：简单的复制拷贝操作

深拷贝：在堆区申请空间，进行拷贝操作

##### 初始化列表

构造函数用于初始化属性

```c
构造函数():属性1(值1), 属性2(值2)...{}
```

##### 对象成员与类的构造和析构顺序

假设B类中有对象A作为成员，即A为对象成员

那么创建B时，A先构造

销毁B时，B先被析构

##### 静态成员

静态成员变量

- 所有对象共享同一份数据
- 在编译阶段分配内存
- 类内声明，类外初始化

静态成员函数

- 所有对象共享同一个函数
- 静态成员函数只能访问静态成员变量

#### this指针

this指针指向被调用的成员函数所属的对象

在类的非静态成员函数中返回对象本身，可以用return *this

##### 空指针访问成员函数

类对象没有被分配存储空间，也可以调用成员函数

因为在C++中，类的成员函数并不占内存空间，成员函数的调用最终都会被编译器转化为一个全局函数的调用，成员函数只是个地址。所以即使类对象的指针为空，也可以正常调用。所以，对于对于成员函数的调用不会有问题

然而，因为成员变量是占用内存的，而test 对象的指针并没有指向一块有效的内存区域，所以，this指针是nullptr。所以，当使用这个空this指针访问具体的内存（比如成员变量a）时，就会出现段错误。

所以，如果一个成员函数没有访问任何成员变量，请将这个成员函数设置为全局函数，如果设置为成员函数，当使用一个类对象的空指针也能访问成功，就会使得类对象失效的问题难以暴露出来

##### const修饰成员函数

常函数

- 成员函数后加const，称之为常函数
- 常函数内不可以修改成员属性
- 成员属性声明时加关键字mutable后，在常函数中依然可以被修改

常对象

- 声明对象前加const，称之为常对象
- 常对象只能调用常函数

#### 友元

让一个函数或者类访问另一个类中私有成员，关键字 friend

##### 全局函数做友元

```c
class Building{
    // 告诉编译器 goodGay全局函数 是 Building类的好朋友，可以访问类中的私有内容
    friend void goodGay(Building *building);

public:
    Building(){
        this->m_SittingRoom = "客厅";
        this->m_BedRoom = "卧室";
    }

public:
    string m_SittingRoom; // 客厅
private:
    string m_BedRoom; // 卧室
};
void goodGay(Building *building){
    cout << "好基友正在访问： " << building->m_SittingRoom << endl;
    cout << "好基友正在访问： " << building->m_BedRoom << endl;
}
```

##### 类做友元

```c
class Building;
class goodGay{
public:
    goodGay();
    void visit();

private:
    Building *building;
};
class Building{
    // 告诉编译器 goodGay类是Building类的好朋友，可以访问到Building类中私有内容
    friend class goodGay;

public:
    Building();

public:
    string m_SittingRoom; // 客厅
private:
    string m_BedRoom; // 卧室
};
Building::Building(){
    this->m_SittingRoom = "客厅";
    this->m_BedRoom = "卧室";
}
goodGay::goodGay(){
    building = new Building;
}
void goodGay::visit(){
    cout << "好基友正在访问" << building->m_SittingRoom << endl;
    cout << "好基友正在访问" << building->m_BedRoom << endl;
}
```

##### 成员函数做友元

```c
class Building;
class goodGay{
public:
    goodGay();
    void visit(); // 只让visit函数作为Building的好朋友，可以发访问Building中私有内容
    void visit2();

private:
    Building *building;
};
class Building{
    // 告诉编译器 goodGay类中的visit成员函数 是Building好朋友，可以访问私有内容
    friend void goodGay::visit();

public:
    Building();

public:
    string m_SittingRoom; // 客厅
private:
    string m_BedRoom; // 卧室
};
Building::Building(){
    this->m_SittingRoom = "客厅";
    this->m_BedRoom = "卧室";
}
goodGay::goodGay(){
    building = new Building;
}
void goodGay::visit(){
    cout << "好基友正在访问" << building->m_SittingRoom << endl;
    cout << "好基友正在访问" << building->m_BedRoom << endl;
}
void goodGay::visit2(){
    cout << "好基友正在访问" << building->m_SittingRoom << endl;
    // cout << "好基友正在访问" << building->m_BedRoom << endl;
}
```

#### 运算符重载

##### 加号运算符重载

```c
class Person {
public:
    Person operator+(const Person& p) {
        ...
    }
}
// 全局函数实现 + 号运算符重载
Person operator+(const Person& p1, const Person& p2) {
    ...
}
// 运算符重载，可以发生函数重载
Person operator+(const Person& p1, int val) {
    ...
}
```

##### 左移运算符重载

```c
class Person {
    //成员函数无法实现，因为会出现 p << cout 不是我们想要的效果
	friend ostream& operator<<(ostream& out, Person& p)
}

// 全局函数实现左移重载
// ostream对象只能有一个
ostream &operator<<(ostream &out, const Person &p) {
    ...
}
```

##### 递增运算符重载

```c
class MyInteger {
public:
    MyInteger() {
        m_Num = 0;
    }
    // 前置++
    MyInteger &operator++() {
        // 先++
        m_Num++;
        // 再返回
        return *this;
    }
    // 后置++
    MyInteger operator++(int) {
        // 先返回
        MyInteger temp = *this; // 记录当前本身的值，然后让本身的值加1，但是返回的是以前的值，达到先返回后++； 
        m_Num++;
        return temp;
    }

private:
    int m_Num;
};
```

##### 赋值运算符重载

```c
class MyInteger {
public:
    Person& operator=(Person &p) {
        ...
        return *this;
    }
}
```

##### 关系运算符重载

```c
class MyInteger {
public:
    bool operator==(Person & p) {
    	if (...) return true;
    	else return false;
    }
}
```

##### 函数调用运算符重载

重载()，重载后的使用方法很像函数，因此称之为仿函数。彷函数没有固定写法，非常灵活

```c
class MyPrint {
public:
    void operator()(string text) {
    	cout << text << endl;
    }
};
```

#### 继承

继承语法：class 子类 : 继承方式 父类

![image-20240801181529976](assets\image-20240801181529976.png)

父类的所有成员（包括私有成员）都被子类继承下去了，只是私有成员由编译器隐藏无法访问

继承中先调用父类构造函数，再调用子类构造函数，析构顺序与构造相反

##### 继承同名成员处理方式

- 子类对象访问子类同名成员	直接访问即可
- 子类对象访问父类同名成员    需要加作用域    子类对象.父类名::同名成员
- 子类访问父类同名静态成员    两种访问方式
	- 通过对象    子类对象.父类名::同名静态成员
	- 通过类名    子类名::父类名::同名静态成员

##### 多继承

```c
class 子类: 继承方式 父类1, 继承方式 父类2...
```

##### 菱形（钻石）继承

两个派生类继承同一个基类A，又有某个类B同时继承两个派生类

菱形继承带来的主要问题是子类B继承了来自基类A的两份相同的数据，导致资源浪费

利用虚继承可以解决该问题

```c
//继承前加virtual关键字后，变为虚继承
//此时公共的父类Animal称为虚基类
class Sheep : virtual public Animal {};
class Tuo : virtual public Animal {};
class SheepTuo : public Sheep, public Tuo {};
```

#### 多态

- 静态多态：函数重载和运算符重载属于静态多态，在编译阶段绑定函数地址
- 动态多态：派生类和虚函数实现运行时多态，在运行阶段绑定函数地址

##### 虚函数

在基类定义一个未实现的函数名，通过基类访问派生类定义的同名函数

```c
class A  
{  
public:  
    void foo()  
    {  
        printf("1\n");  
    }  
    virtual void fun()  
    {  
        printf("2\n");  
    }  
};  
class B : public A  
{  
public:  
    void foo()  //隐藏：派生类的函数屏蔽了与其同名的基类函数
    {  
        printf("3\n");  
    }  
    void fun()  //多态、覆盖
    {  
        printf("4\n");  
    }  
};  
int main(void)  
{  
    A a;  
    B b;  
    A *p = &a;  
    p->foo();  //输出1
    p->fun();  //输出2
    p = &b;  
    p->foo();  //取决于指针类型，输出1
    p->fun();  //取决于对象类型，输出4，体现了多态
    return 0;  
}
```

为了减少程序运行的错误，重写的虚函数都建议加上override，明确表示派生类的这个虚函数是重写虚类的

```c
class Base {
public:
    virtual void Show(int x); // 虚函数
};
 
class Derived : public Base {
public:
    virtual void Show(int x) const override; // const 属性不一样，新的虚函数 
};
```

如果不希望某个类被继承，或不希望某个虚函数被重写，则可以在类名和虚函数后加上final关键字

##### 纯虚函数和抽象类

在多态中，一般基类的虚函数实现是没有意义的，主要是调用子类重写的内容，因此可以将虚函数改为纯虚函数，virtual 函数 = 0;

当类中有纯虚函数后，这个类也被称为抽象类

抽象类无法被实例化，且子类必须重写抽象类中的纯虚函数，否则也属于抽象类
##### 虚析构和纯虚析构

在多态使用时，如果子类有属性开辟到堆区，那么父类指针在释放时无法调用到子类的析构方法

可以将父类的析构方法改为虚析构或者纯虚析构，拥有纯虚析构的类也属于抽象类

### 文件操作

#### 文件打开方式

| 打开方式        | 解释            |
| ----------- | ------------- |
| ios::in     | 读文件           |
| ios::out    | 写文件           |
| ios::ate    | 打开文件的初始位置：文件尾 |
| ios::app    | 追加方式写文件       |
| ios::trunc  | 如果文件存在先删除，再创建 |
| ios::binary | 以二进制方式打开文件    |

#### 文本文件

##### 写文件

```c
#include <fstream>
void write() {
    ofstream ofs;
    ofs.open("test.txt", ios::out);
    ofs << "姓名：张三" << endl;
    ofs << "性别：男" << endl;
    ofs << "年龄：18" << endl;
    ofs.close();
}
```

##### 读文件

```c
void read()
{
    ifstream ifs;
    ifs.open("test.txt", ios::in);
    if (!ifs.is_open())
    {
        cout << "文件打开失败" << endl;
        return;
    }
    // 第一种方式
    char buf[1024] = { 0 };
    while (ifs >> buf)
    {
     cout << buf << endl;
    }
    // 第二种
    char buf[1024] = { 0 };
    while (ifs.getline(buf,sizeof(buf)))
    {
     cout << buf << endl;
    }
    // 第三种
    string buf;
    while (getline(ifs, buf))
    {
     cout << buf << endl;
    }
    char c;
    while ((c = ifs.get()) != EOF)
    {
        cout << c;
    }
    ifs.close();
}
```

#### 二进制文件
##### 写文件

```c
// 二进制文件 写文件
void write()
{
    // 1、包含头文件
    // 2、创建输出流对象
    ofstream ofs("person.txt", ios::out | ios::binary);
    // 3、打开文件
    // ofs.open("person.txt", ios::out | ios::binary);
    Person p = {"张三", 18};
    // 4、写文件
    ofs.write((const char *)&p, sizeof(p));
    // 5、关闭文件
    ofs.close();
}
```

##### 读文件

```c
void read()
{
    ifstream ifs("person.txt", ios::in | ios::binary);
    if (!ifs.is_open())
    {
        cout << "文件打开失败" << endl;
    }
    Person p;
    ifs.read((char *)&p, sizeof(p));
    cout << "姓名： " << p.m_Name << " 年龄： " << p.m_Age << endl;
}
```
## 提高编程

### 模板

通用摸具，有助于提高复用性
#### 函数模板

C++有一种编程思想为泛型编程，主要用到的技术就是模板

C++提供两种模板机制：函数模板

```c++
template<typename T>
函数声明或定义
```

##### 普通函数和函数模板的调用规则

-   如果函数模板和普通模板都可以实现，优先普通函数
-   可以通过空模板参数列表来强制调用函数模板
-   函数模板也可以发生重载
-   如果函数模板可以更好匹配，优先函数模板

##### 具体化模板

```c++
template<class T>
void func(T a, T b)
{
	cout << "普通模板" <<endl;
}

template<> void func(int *a, int *b)
{
	cout << "具体化模板" << endl;
}

//具体化，显示具体化的原型和定意思以template<>开头，并通过名称来指出类型
int main()
{
	// 函数模板
	func(1, 2);
	int a[5], b[5];
	// 具体化模板
	func(a, b);
}
```

#### 类模板

```c++
// 类模板
template <class NameType, class AgeType = int>
class Person
{
public:
	Person(NameType name, AgeType age)
	{
		this->mName = name;
		this->mAge = age;
	}
	void showPerson() { cout << "name: " << this->mName << " age: " << this->mAge << endl; }

public:
	NameType mName;
	AgeType mAge;
};
```

与函数模板的区别：

-   类模板没有自动类型推导的使用方式
-   类模板在模板参数列表中可以有默认参数

成员函数创建时机不同：

-   普通类中的成员函数一开始就创建
-   类模板中的成员函数在调用时才创建

##### 类模板对象做函数参数

一共三种传入方式

-   指定传入的类型	   ——直接显示对象的数据类型
-   参数模板化		   ——将对象中的参数变为模板进行传递
-   整个类模板                   ——将这个对象类型模板化进行传递

```c++
//1、指定传入的类型
void printPerson1(Person<string, int> &p);

//2、参数模板化
template <class T1, class T2>
void printPerson2(Person<T1, T2>&p);

//3、整个类模板化 
template<class T>
void printPerson3(T & p);
```

##### 类模板与继承

-   当父类是一个类模板时，子类在声明时，要指定父类中T的类型
-   如果不指定，编译器无法给子类分配内存
-   如果想灵活指定父类中T的类型，子类也要变为类模板

```c++
template <class T>
class Base
{
	T m;
};

// class Son:public Base //错误，c++编译需要给子类分配内存，必须知道父类中T的类型才可 以向下继承
class Son : public Base<int> // 必须指定一个类型 
{ };

//类模板继承类模板 ,可以用T2指定父类中的T类型
template<class T1, class T2>
class Son2 :public Base<T2>
{ };
```

##### 类模板成员函数类外实现

类模板中成员函数类外实现时，需要加上模板参数列表

```c++
// 类模板中成员函数类外实现
template <class T1, class T2>
class Person
{
public:
	// 成员函数类内声明
	Person(T1 name, T2 age);
	void showPerson();

public:
	T1 m_Name;
	T2 m_Age;
};

// 构造函数 类外实现
template <class T1, class T2>
Person<T1, T2>::Person(T1 name, T2 age)
{
	this->m_Name = name;
	this->m_Age = age;
}
```

##### 类模板分文件编写

类模板中成员函数的创建时机是在调用阶段，导致分文件编写时无法链接

解决方法：

-   直接包含 .cpp 源文件，而非 .h 文件
-   将声明和实现写到同一个文件里，并更改后缀名为 .hpp，hpp是约定名称，非强制

##### 类模板和友元函数

全局函数类内实现 - 直接在类内声明友元即可

全局函数类外实现 - 需要提前让编译器知道全局函数的存在，即提前声明

### STL

STL(Standard Template Library,标准模板库)，从广义上分为: 容器(container) 算法(algorithm) 迭代器(iterator)，容器和算法之间通过迭代器进行无缝连接

#### 六大组件

-   容器：各种数据结构，如vector、list、deque、set、map等,用来存放数据
-   算法：各种常用的算法，如sort、find、copy、for_each等
-   迭代器：扮演了容器与算法之间的胶合剂
-   仿函数：行为类似函数，可作为算法的某种策略
-   适配器：一种用来修饰容器或者仿函数或迭代器接口的东西
-   空间配置器：负责空间的配置与管理

#### 容器

是将运用最广泛的一些数据结构实现出来，常用的数据结构：数组, 链表,树, 栈, 队列, 集合, 映射表 等

这些容器分为序列式容器和关联式容器两种

-   序列式容器:强调值的排序，序列式容器中的每个元素均有固定的位置
-   关联式容器:二叉树结构，各元素之间没有严格的物理上的顺序关系

#### 算法

算法是使用有限的步骤，解决逻辑或数学上的问题

算法分为质变算法和非质变算法

-   质变算法：是指运算过程中会更改区间内的元素的内容。例如拷贝，替换，删除等等
-   非质变算法：是指运算过程中不会更改区间内的元素内容，例如查找、计数、遍历、寻找极值等等

#### 迭代器

容器和算法之间粘合剂，提供一种方法，使之能够依序寻访某个容器所含的各个元素，而又无需暴露该容器的内部表示方式。每个容器都有自己专属的迭代器

迭代器种类

| 种类           | 功能                                   |
| -------------- | -------------------------------------- |
| 输入迭代器     | 对数据进行只读访问                     |
| 输出迭代器     | 对数据进行只写访问                     |
| 前向迭代器     | 读写操作，并能向前推进迭代器           |
| 双向迭代器     | 读写操作，并能向前向后操作             |
| 随机访问迭代器 | 读写操作，可以以跳跃的方式访问任意数据 |

常用迭代器为双向迭代器和随机访问迭代器

##### 使用方法

```c++
// 创建vector容器对象，并且通过模板参数指定容器中存放的数据的类型
vector<int> v;
// 向容器中放数据
v.push_back(10);
v.push_back(20);
v.push_back(30);
v.push_back(40);
// 每一个容器都有自己的迭代器，迭代器是用来遍历容器中的元素
// v.begin()返回迭代器，这个迭代器指向容器中第一个数据
// v.end()返回迭代器，这个迭代器指向容器元素的最后一个元素的下一个位置
// vector<int>::iterator 拿到vector<int>这种容器的迭代器类型
vector<int>::iterator pBegin = v.begin();
vector<int>::iterator pEnd = v.end();
// 第一种遍历方式：
while (pBegin != pEnd)
{
    cout << *pBegin << endl;
    pBegin++;
}
// 第二种遍历方式：
for (vector<int>::iterator it = v.begin(); it != v.end(); it++)
{
    cout << *it << endl;
}
cout << endl;
// 第三种遍历方式：
// 使用STL提供标准遍历算法 头文件 algorithm
for_each(v.begin(), v.end(), MyPrint); // void MyPrint(int val);
```

#### string容器

string是一个类，类内部封装了char\*，管理这个字符串，是一个char\*型的容器

##### 构造函数

```c++
string();					//创建一个空的字符串 例如: string str;
string(const char* s);		//使用字符串s初始化
string(const string& str); 	//使用一个string对象初始化另一个string对象
string(int n, char c); 		//使用n个字符c初始化
```

##### 赋值操作

```c++
string& operator=(const char* s); 		//char*类型字符串 赋值给当前的字符串
string& operator=(const string &s);		//把字符串s赋给当前的字符串
string& operator=(char c); 				//字符赋值给当前的字符串
string& assign(const char *s); 			//把字符串s赋给当前的字符串
string& assign(const char *s, int n); 	//把字符串s的前n个字符赋给当前的字符串
string& assign(const string &s); 		//把字符串s赋给当前字符串
string& assign(int n, char c); 			//用n个字符c赋给当前字符串
```

##### 字符串拼接

```c++
string& operator+=(const char* str); 	//重载+=操作符
string& operator+=(const char c); 		//重载+=操作符
string& operator+=(const string& str); 	//重载+=操作符
string& append(const char *s); 			//把字符串s连接到当前字符串结尾
string& append(const char *s, int n); 	//把字符串s的前n个字符连接到当前字符串结尾
string& append(const string &s); 		//同operator+=(const string& str)
string& append(const string &s, int pos, int n); //字符串s中从pos开始的n个字符连接到字符串结尾
```

##### 查找

```c++
int find(const string& str, int pos = 0) const; 	//查找str第一次出现位置,从pos开始查找。找不到返回 string::npos, 以下也一样
int find(const char* s, int pos = 0) const; 		//查找s第一次出现位置,从pos开始查找
int find(const char* s, int pos, int n) const; 		//从pos位置查找s的前n个字符第一次位置
int find(const char c, int pos = 0) const; 			//查找字符c第一次出现位置
int rfind(const string& str, int pos = npos) const; //查找str最后一次位置,从pos开始查找
int rfind(const char* s, int pos = npos) const; 	//查找s最后一次出现位置,从pos开始查找
int rfind(const char* s, int pos, int n) const; 	//从pos查找s的前n个字符最后一次位置
int rfind(const char c, int pos = 0) const; 		//查找字符c最后一次出现位置
```

##### 替换

```c++
string& replace(int pos, int n, const string& str);	//替换从pos开始n个字符为字符串str
string& replace(int pos, int n,const char* s); 		//替换从pos开始的n个字符为字符串s
```

##### 字符串比较

```c++
int compare(const string &s) const; //与字符串s比较, > 1, = 0, < -1
int compare(const char *s) const; 	//与字符串s比较
```

##### 字符存取

```c++
char& operator[](int n); 	//通过[]方式取字符
char& at(int n); 			//通过at方法获取字符
```

##### 插入和删除

```c++
string& insert(int pos, const char* s); 	//插入字符串
string& insert(int pos, const string& str); //插入字符串
string& insert(int pos, int n, char c); 	//在指定位置插入n个字符c
string& erase(int pos, int n = npos); 		//删除从Pos开始的n个字符
```

##### 获取子串

```c++
string substr(int pos = 0, int n = npos) const;	//返回由pos开始的n个字符组成的字符串
```

#### vector容器

vector数据结构和数组非常相似，也称为单端数组

不同之处在于数组是静态空间，而vector可以动态扩展

并不是在原空间之后续接新空间，而是找更大的内存空间，然后将原数据拷贝新空间，释放原空间

vector容器的迭代器是支持随机访问的迭代器

##### 构造函数

```c++
vector<T> v; 				//采用模板实现类实现，默认构造函数
vector(v.begin(), v.end()); //将v[begin(), end())区间中的元素拷贝给本身
vector(n, elem); 			//构造函数将n个elem拷贝给本身
vector(const vector &vec); 	//拷贝构造函数
```

##### 赋值操作

```c++
vector& operator=(const vector &vec); 	//重载等号操作符
assign(beg, end); 						//将[beg, end)区间中的数据拷贝赋值给本身
assign(n, elem); 						//将n个elem拷贝赋值给本身
```

##### 容量和大小

```c++
empty(); 				//判断容器是否为空
capacity(); 			//容器的容量
size(); 				//返回容器中元素的个数
resize(int num); 		//重新指定容器的长度为num，若容器变长，则以默认值填充新位置。如果容器变短，则末尾超出容器长度的元素被删除。
resize(int num, elem); 	//重新指定容器的长度为num，若容器变长，则以elem值填充新位置。如果容器变短，则末尾超出容器长度的元素被删除
```

##### 插入和删除

```c++
push_back(ele); 									//尾部插入元素ele
pop_back(); 										//删除最后一个元素
insert(const_iterator pos, ele); 					//迭代器指向位置pos插入元素ele
insert(const_iterator pos, int count,ele); 			//迭代器指向位置pos插入count个元素ele
erase(const_iterator pos); 							//删除迭代器指向的元素
erase(const_iterator start, const_iterator end); 	//删除迭代器从start到end之间的元素
clear(); 											//删除容器中所有元素
```

##### 数据存取

```c++
at(int idx); 	//返回索引idx所指的数据
operator[]; 	//返回索引idx所指的数据
front(); 		//返回容器中第一个数据元素
back(); 		//返回容器中最后一个数据元素
```

##### 互换容器

```c++
swap(vec); // 将vec与本身的元素互换，实现两个容器内元素进行互换

// 收缩内存，即capacity=size
// vector<int>(v)创建匿名对象，其值与v相等，但capacity=size
// swap后，v的内存收缩
vector<int>(v).swap(v);
```

##### 预留空间

```c++
reserve(int len); 	// 容器预留len个元素长度，预留位置不初始化，元素不可访问，减少扩充空间的时间花销
```

#### deque容器

双端数组，可以对头端进行插入删除操作

vector对于头部的插入删除效率低，数据量越大，效率越低

deque相对而言，对头部的插入删除速度回比vector快

vector访问元素时的速度会比deque快,这和两者内部实现有关

##### 构造函数

```c++
deque<T> deqT; 				//默认构造形式
deque(beg, end); 			//构造函数将[beg, end)区间中的元素拷贝给本身
deque(n, elem); 			//构造函数将n个elem拷贝给本身
deque(const deque &deq); 	//拷贝构造函数
```

##### 赋值操作

```c++
deque& operator=(const deque &deq); //重载等号操作符
assign(beg, end); 					//将[beg, end)区间中的数据拷贝赋值给本身。
assign(n, elem); 					//将n个elem拷贝赋值给本身
```

##### 容器大小操作

```c++
deque.empty(); 				//判断容器是否为空
deque.size(); 				//返回容器中元素的个数
deque.resize(num); 			//重新指定容器的长度为num,若容器变长，则以默认值填充新位置。如果容器变短，则末尾超出容器长度的元素被删除。
deque.resize(num, elem); 	//重新指定容器的长度为num,若容器变长，则以elem值填充新位置。如果容器变短，则末尾超出容器长度的元素被删除。
```

##### 插入和删除

```c++
push_back(elem); 		//在容器尾部添加一个数据
push_front(elem); 		//在容器头部插入一个数据
pop_back(); 			//删除容器最后一个数据
pop_front(); 			//删除容器第一个数据
insert(pos,elem); 		//在pos位置插入一个elem元素的拷贝，返回新数据的位置
insert(pos,n,elem); 	//在pos位置插入n个elem数据，无返回值
insert(pos,beg,end);	//在pos位置插入[beg,end)区间的数据，无返回值
clear(); 				//清空容器的所有数据
erase(beg,end); 		//删除[beg,end)区间的数据，返回下一个数据的位置
erase(pos); 			//删除pos位置的数据，返回下一个数据的位置
```

##### 数据存取

```c++
at(int idx); 	//返回索引idx所指的数据
operator[]; 	//返回索引idx所指的数据
front(); 		//返回容器中第一个数据元素
back(); 		//返回容器中最后一个数据元素
```

#### stack容器

stack是一种先进后出(First In Last Out,FILO)的数据结构，它只有一个出口

栈中只有顶端的元素才可以被外界使用，因此栈不允许有遍历行为

##### 构造函数

```c++
stack<T> stk; 				//stack采用模板类实现， stack对象的默认构造形式
stack(const stack &stk); 	//拷贝构造函数
```

##### 赋值操作

```c++
stack& operator=(const stack &stk); //重载等号操作符
```

##### 数据存取

```c++
push(elem); 	//向栈顶添加元素
pop(); 			//从栈顶移除第一个元素，无返回值
top(); 			//返回栈顶元素
```

##### 大小操作

```c++
empty(); 		//判断堆栈是否为空
size(); 		//返回栈的大小
```

#### queue 容器

queue是一种先进先出(First In First Out,FIFO)的数据结构，它有两个出口

队列容器允许从一端新增元素，从另一端移除元素

队列中只有队头和队尾才可以被外界使用，因此队列不允许有遍历行为

##### 构造函数

```c++
queue<T> que; 				//queue采用模板类实现，queue对象的默认构造形式
queue(const queue &que); 	//拷贝构造函数
```

##### 赋值操作

```c++
queue& operator=(const queue &que); //重载等号操作符
```

##### 数据存取

```c++
push(elem); 	//往队尾添加元素
pop(); 			//从队头移除第一个元素
back(); 		//返回最后一个元素
front(); 		//返回第一个元素
```

##### 大小操作

```c++
empty(); 		//判断队列是否为空
size(); 		//返回队列的大小
```

#### list容器

物理存储单元上非连续的存储结构，数据元素的逻辑顺序是通过链表中的指针链接实现的

STL中的链表是一个双向循环链表

List有一个重要的性质，插入操作和删除操作都不会造成原有list迭代器的失效，这在vector是不成立的

##### 构造函数

```c++
list<T> lst; 			//list采用采用模板类实现,对象的默认构造形式
list(beg,end); 			//构造函数将[beg, end)区间中的元素拷贝给本身
list(n,elem); 			//构造函数将n个elem拷贝给本身
list(const list &lst); 	//拷贝构造函数
```

##### 赋值和交换

```c++
assign(beg, end); 					//将[beg, end)区间中的数据拷贝赋值给本身。
assign(n, elem); 					//将n个elem拷贝赋值给本身。
list& operator=(const list &lst); 	//重载等号操作符
swap(lst); 							//将lst与本身的元素互换
```

##### 大小操作

```c++
size(); 			//返回容器中元素的个数
empty(); 			//判断容器是否为空
resize(num); 		//重新指定容器的长度为num，若容器变长，则以默认值填充新位置。如果容器变短，则末尾超出容器长度的元素被删除
resize(num, elem); 	//重新指定容器的长度为num，若容器变长，则以elem值填充新位置。
```

#####  插入和删除

```c++
push_back(elem);		//在容器尾部加入一个元素
pop_back();				//删除容器中最后一个元素
push_front(elem);		//在容器开头插入一个元素
pop_front();			//从容器开头移除第一个元素
insert(pos,elem);		//在pos位置插elem元素的拷贝，返回新数据的位置
insert(pos,n,elem);		//在pos位置插入n个elem数据，无返回值
insert(pos,beg,end);	//在pos位置插入[beg,end)区间的数据，无返回值
clear();				//移除容器的所有数据
erase(beg,end);			//删除[beg,end)区间的数据，返回下一个数据的位置
erase(pos);				//删除pos位置的数据，返回下一个数据的位置
remove(elem);			//删除容器中所有与elem值匹配的元素
```

##### 数据存取

```c++
front(); 	//返回第一个元素
back(); 	//返回最后一个元素，不支持 at 和 []
```

##### 反转和排序

```c++
reverse(); 	//反转链表
sort(); 	//链表排序，可以加入cmp

L.sort([](int x, int y){
    return x < y;
});
```

#### set/ multiset 容器

所有元素都会在插入时自动被排序

set/multiset属于关联式容器，底层结构是用二叉树实现

set不允许容器中有重复的元素 multiset允许容器中有重复的元素

##### 构造和赋值

```c++
set<T> st; 						//默认构造函数：
set(const set &st); 			//拷贝构造函数
set& operator=(const set &st); 	//重载等号操作符
```

##### 大小和交换

```c++
size(); 	//返回容器中元素的数目
empty(); 	//判断容器是否为空
swap(st); 	//交换两个集合容器
```

##### 插入和删除

```c++
insert(elem); 		//在容器中插入元素。
clear(); 			//清除所有元素
erase(pos); 		//删除pos迭代器所指的元素，返回下一个元素的迭代器。
erase(beg, end); 	//删除区间[beg,end)的所有元素 ，返回下一个元素的迭代器。
erase(elem); 		//删除容器中值为elem的元素
```

##### 查找和统计

```c++
find(key); 	//查找key是否存在,若存在，返回该键的元素的迭代器；若不存在，返回set.end();
count(key); //统计key的元素个数
```

##### set和multiset区别

set不可以插入重复数据，而multiset可以

set插入数据的同时会返回插入结果，表示插入是否成功

multiset不会检测数据，因此可以插入重复数据

##### 容器排序

利用仿函数，可以改变排序规则

对于自定义数据类型，set必须指定排序规则才可以插入数据

```\
class MyCompare
{
public:
	bool operator()(int v1, int v2)
	{
		return v1 > v2;
	} 
};

set<int, MyCompare> s;
```

#### map/ multimap容器

map中所有元素都是pair

pair中第一个元素为key（键值），起到索引作用，第二个元素为value（实值）

所有元素都会根据元素的键值自动排序

map/multimap属于关联式容器，底层结构是用二叉树实现

##### 构造和赋值

```c++
map<T1, T2> mp; 				//map默认构造函数
map(const map &mp); 			//拷贝构造函数
map& operator=(const map &mp); 	//重载等号操作符
```

##### 大小和交换

```c++
size(); 	//返回容器中元素的数目
empty(); 	//判断容器是否为空
swap(st); 	//交换两个集合容器
```

##### 插入和删除

```c++
insert(elem); 		//在容器中插入元素
clear(); 			//清除所有元素
erase(pos); 		//删除pos迭代器所指的元素，返回下一个元素的迭代器
erase(beg, end); 	//删除区间[beg,end)的所有元素 ，返回下一个元素的迭代器
erase(key); 		//删除容器中值为key的元素
```

##### 查找和统计

```c++
find(key); 	//查找key是否存在,若存在，返回该键的元素的迭代器；若不存在，返回set.end();
count(key); //统计key的元素个数
```

##### 容器排序

map容器默认排序规则为 按照key值进行 从小到大排序

利用仿函数，可以改变排序规则

对于自定义数据类型，map必须要指定排序规则,同set容器

### 函数对象

#### 概念

重载函数调用操作符的类，其对象常称为函数对象

函数对象使用重载的()时，行为类似函数调用，也叫仿函数

函数对象(仿函数)是一个类，不是一个函数

使用方法：

-   函数对象在使用时，可以像普通函数那样调用, 可以有参数，可以有返回值
-   函数对象超出普通函数的概念，函数对象可以有自己的状态
-   函数对象可以作为参数传递

#### 谓词

返回bool类型的仿函数称为谓词

如果operator()接受一个参数，那么叫做一元谓词

如果operator()接受两个参数，那么叫做二元谓词

##### 内建函数对象

STL内建了一些函数对象

-   算术仿函数
-   关系仿函数
-   逻辑仿函数

这些仿函数所产生的对象，用法和一般函数完全相同

使用内建函数对象，需要引入头文件 #include \<functional>

##### 算术仿函数

```c++
template<class T> T plus<T> 		//加法仿函数
template<class T> T minus<T> 		//减法仿函数
template<class T> T multiplies<T> 	//乘法仿函数
template<class T> T divides<T> 		//除法仿函数
template<class T> T modulus<T> 		//取模仿函数
template<class T> T negate<T> 		//取反仿函数，一元运算
   
negate<int> n;
cout << n(50) << endl;
plus<int> p;
cout << p(10, 20) << endl;
```

##### 关系仿函数

实现关系对比

```c++
template<class T> bool equal_to<T> 			//等于
template<class T> bool not_equal_to<T> 		//不等于
template<class T> bool greater<T> 			//大于
template<class T> bool greater_equal<T> 	//大于等于
template<class T> bool less<T> 				//小于
template<class T> bool less_equal<T> 		//小于等于
    
sort(v.begin(), v.end(), greater<int>());	//使用大于仿函数，进行排序
```

##### 逻辑仿函数

实现逻辑运算

```c++
template<class T> bool logical_and<T> 	//逻辑与
template<class T> bool logical_or<T> 	//逻辑或
template<class T> bool logical_not<T> 	//逻辑非
```

#### 常用算法

算法主要是由头文件\<algorithm>, \<functional>, \<numeric>组成

-   \<algorithm> 是所有STL头文件中最大的一个，范围涉及到比较、 交换、查找、遍历操作、复制、 修改等等

-   \<numeric> 体积很小，只包括几个在序列上面进行简单数学运算的模板函数
-   \<functional> 定义了一些模板类,用以声明函数对象

##### 常用遍历算法

```c++
for_each(iterator beg, iterator end, _func);	// 遍历容器元素，执行__func函数

// beg1 源容器开始迭代器
// end1 源容器结束迭代器
// beg2 目标容器开始迭代器
// _func 函数或者函数对象
transform(iterator beg1, iterator end1, iterator beg2, _func);
```

##### 常用查找算法

```c++
// 按值查找元素，找到返回指定位置迭代器，找不到返回结束迭代器位置
find(iterator beg, iterator end, value);

// 按值查找元素，找到返回指定位置迭代器，找不到返回结束迭代器位置
// _Pred 函数或者谓词（返回bool类型的仿函数）
find_if(iterator beg, iterator end, _Pred);	

// 查找相邻重复元素,返回相邻元素的第一个位置的迭代器
adjacent_find(iterator beg, iterator end);

// 查找指定的元素，查到 返回true 否则false。注意: 在无序序列中不可用
bool binary_search(iterator beg, iterator end, value);

// 统计元素出现次数
count(iterator beg, iterator end, value);

// 按条件统计元素出现次数
count_if(iterator beg, iterator end, _Pred);
```

##### 常用排序算法

```c++
// 对容器内元素进行排序
sort(iterator beg, iterator end, _Pred);

// 指定范围内的元素随机调整次序
random_shuffle(iterator beg, iterator end);

// 容器元素合并，并存储到另一容器中
// 注意: 两个容器必须是有序的
merge(iterator beg1, iterator end1, iterator beg2, iterator end2, iterator
dest);

// 反转指定范围的元素
reverse(iterator beg, iterator end);
```

##### 常用拷贝和替换算法

```c++
// 容器内指定范围的元素拷贝到另一容器中
copy(iterator beg, iterator end, iterator dest);

// 将容器内指定范围的旧元素修改为新元素
replace(iterator beg, iterator end, oldvalue, newvalue);

// 按条件替换元素，满足条件的替换成指定元素
// _pred 谓词
replace_if(iterator beg, iterator end, _pred, newvalue);

// 互换两个容器的元素
swap(container c1, container c2);
```

##### 常用算术生成算法

使用时包含的头文件为 #include \<numeric>

```c++
// 计算容器元素累计总和
accumulate(iterator beg, iterator end, value);

// 向容器中填充指定的元素
fill(iterator beg, iterator end, value);
```

##### 常用集合算法

```c++
// 求两个集合的交集
// 注意:两个集合必须是有序序列
set_intersection(iterator beg1, iterator end1, iterator beg2, iterator end2, iterator dest);

// 求两个集合的并集
set_union(iterator beg1, iterator end1, iterator beg2, iterator end2, iterator dest);

// 求两个集合的差集
set_difference(iterator beg1, iterator end1, iterator beg2, iterator end2, iterator dest);
```

