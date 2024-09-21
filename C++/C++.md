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

| 打开方式    | 解释                       |
| ----------- | -------------------------- |
| ios::in     | 读文件                     |
| ios::out    | 写文件                     |
| ios::ate    | 打开文件的初始位置：文件尾 |
| ios::app    | 追加方式写文件             |
| ios::trunc  | 如果文件存在先删除，再创建 |
| ios::binary | 以二进制方式打开文件       |

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

