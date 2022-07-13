## 关于作者

微信公众号：

![](../img/wechat.jpg)

## const含义

常类型是指使用类型修饰符**const**说明的类型，常类型的变量或对象的值是不能被更新的。

## const作用

+ 可以定义常量

```cpp
const int a=100;
```

+ 类型检查

    + const常量与`#define`宏定义常量的区别：
    > ~~<u>**const常量具有类型，编译器可以进行安全检查；#define宏定义没有数据类型，只是简单的字符串替换，不能进行安全检查。**</u>~~感谢两位大佬指出这里问题，见：[issue](https://github.com/Light-City/CPlusPlusThings/issues/5)

    + const定义的变量只有类型为整数或枚举，且以常量表达式初始化时才能作为常量表达式。
    + 其他情况下它只是一个 `const` 限定的变量，不要将与常量混淆。

+ 防止修改，起保护作用，增加程序健壮性

```cpp
void f(const int i){
    i++; //error!
}
```

+ 可以节省空间，避免不必要的内存分配

    + const定义常量从汇编的角度来看，只是给出了对应的内存地址，而不是像`#define`一样给出的是立即数。
    + const定义的常量在程序运行过程中只有一份拷贝，而`#define`定义的常量在内存中有若干个拷贝。

## const对象默认为文件局部变量

<p><font style="color:red">注意：非const变量默认为extern。要使const变量能够在其他文件中访问，必须在文件中显式地指定它为extern。</font></p>


```cpp
// 非const变量在不同文件的访问
// file1.cpp
int ext; //非const,不用extern,不必初始化

// file2.cpp
#include<iostream>
extern int ext;
int main(void){ std::cout<<(ext+10)<<std::endl; }
```


```cpp
// const常量在不同文件的访问
//extern_file1.cpp
//const int ext=12; //必须初始化,没有extern所以不能被其他文件访问
extern const int ext=12; //必须初始化,有extern所以可以被其他文件访问

//extern_file2.cpp
#include<iostream>
extern const int ext;
int main(void){ std::cout<<ext<<std::endl; }
```

## 定义常量

```cpp
const int b = 10;
b = 0; // error: 常量b不能修改
const string s = "helloworld"; //ok
const int i,j=0 // error: 常量i未初始化
```

## 指针与const

与指针相关的const有四种：

```cpp
const char * a; //指向const char对象的 指针(指向常量的指针)
char const * a; //同上
char * const a; //指向char对象的 const指针(常指针)
const char * const a; //指向const char对象的 const指针
```

具体使用如下：

###  **指向常量的指针**

```cpp
const int *ptr; //指向const int对象的 指针，指针可变(可以不初始化)
*ptr = 10; //error 因为const int不能被修改
```

```cpp
const int p = 10;
const void * vp = &p; //ok
void *vp = &p; //error,不能使用void *指针保存const int对象的地址
```

```cpp
int val = 3;  //非const变量
const int *ptr; //指向const int对象的指针
ptr = &val; //ok
*ptr = 10; //error,无法通过ptr来修改val的值

int *ptr1 = &val; //想修改val就定义非const的指针
*ptr1=4;
cout<<*ptr<<endl; //访问可以用const int指针
```

### **常指针**

```cpp
#include<iostream>
using namespace std;
int main(void){
    int num=0;
    int * const ptr=&num; //const指针必须初始化！且const指针的值不能修改
    ptr = NULL; //error const指针不能修改
    *ptr=2; //ok
    cout<<*ptr<<endl;
}
```

```cpp
#include<iostream>
using namespace std;
int main(void){
    const int num=0;
    int * const ptr=&num; //error 指向'int对象'的const指针不能指向'const int对象'
    cout<<*ptr<<endl;
}
```

### **指向常量的常指针**

```cpp
const int a = 3;
const int * const ptr = &a;
```

## 函数中使用const

### const修饰函数返回值

```cpp
const int func1(); //跟const修饰普通变量以及指针的含义基本相同：
//无意义，因为参数返回本身就是赋值给其他的变量！

const int* func2(); //const int对象不可变

int *const func2(); //指针本身不可变
```

### const修饰函数参数

```cpp
void func(const int var); // var的值在函数内不可变
void func(int *const var); // var指针不能指向其它地方

void StringCopy(char *dst, const char *src); //src指针所指的内容为常量(不可变),企图修改编译就报错

void func(const A &a); //推荐使用，参数为引用，为了增加效率同时防止修改
```

- `void func(A a);`效率低,因为调用时会调用复制构造函数创建a,函数退出又会析构a(复制构造+析构比较耗时)
- `void func(A &a);`效率高,因为“引用传递”仅借用一下参数的别名,不产生临时对象,但是函数内可能修改对象内容
- `void func(const A &a)`这样即高效又防止函数内修改了对象内容

`void func(int x)`不用改为`void func(const int &x)`因为内部数据类型的参数不存在构造、析构的过程，而复制也非常快，“值传递”和“引用传递”的效率几乎相当。

**为了高效**
- 非内部数据类型的传参用'const 引用传递':`void func(const A &a)`
- 内部数据类型的传参用'值传递':`void func(int x)`

**解决了两个面试问题**

+ 如果函数需要传入一个指针，是否需要为该指针加上const，把const加在指针不同的位置有什么区别；
+ 如果写的函数需要传入的参数是一个复杂类型的实例，传入值参数或者引用参数有什么区别，什么时候需要为传入的引用参数加上const。

## 类中使用const

在一个类中，任何不会修改数据成员的函数都应该声明为const类型。如果在编写const成员函数时，不慎修改 数据成员，或者调用了其它非const成员函数，编译器将指出错误，这无疑会提高程序的健壮性。

使用const关键字进行说明的成员函数，称为常成员函数。只有常成员函数才有资格操作常量或常对象，没有使用const关键字进行说明的成员函数不能用来操作常对象。


const对象只能访问const成员函数,而非const对象可以访问任意的成员函数,包括const成员函数.

例如：

```cpp
//apple.cpp
class Apple {
private:
    int people[100];
public:
    Apple(int i);
    const int apple_number; //const成员数据

    void take(int num) const; //const成员函数
    int add();
    int add(int num) const; //const成员函数
    int getCount() const; //const成员函数
};

//main.cpp
#include<iostream>
#include"apple.cpp"
using namespace std;

Apple::Apple(int i):apple_number(i) //const成员数据必须在构造函数初始化
{
}
int Apple::add(){ //非const成员函数,可以调用所有成员函数
    take(1);
    return 0;
}
int Apple::add(int num) const{ //const成员函数,只能调用const成员函数
    take(num);
    return num;
}
void Apple::take(int num) const { //const成员函数,只能调用const成员函数
    cout<<"take func "<<num<<endl;
}
int Apple::getCount() const { //const成员函数,只能调用const成员函数
    take(1);
    add();  // error,add()是非const成员函数
    return apple_number;
}
int main(void){
    Apple a(2);
    cout<<a.getCount()<<endl;
    a.add(10);
    return 0;
}
```
> 编译： g++ -o main main.cpp apple.cpp


初始化const成员数据，除了'在构造函数初始化'外，也可以这样

1. 先将常量定义与static结合，也就是：
```cpp
static const int apple_number;
```
2. 然后在外面初始化：
```cpp
const int Apple::apple_number=10;
```

如果使用c++11(加`-std=c++11`)进行编译，直接可以在定义出初始化
```cpp
static const int apple_number=10;
// 或者
const int apple_number=10;
```

这两种都在c++11中支持！

类内static成员变量，c++11不能进行声明并初始化，需要
1. 先在类中声明：
```cpp
static int ap; //类中static成员变量只是声明,不能初始化
```
2. 再在类实现文件中初始化
```cpp
int Apple::ap=666; //类的实现进行初始化
```
