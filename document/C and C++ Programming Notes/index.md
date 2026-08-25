
本手册面向计算机硬件方向的学生，对硬件感兴趣的同学和学习C/C++的新生。

学习计算机硬件需要毅力和耐心，以及浓厚的兴趣，而不是速学速通。因此该讲义会偏”细水长流“，尽量把涉及的拓展知识讲一讲。


> 知识点推荐资源

<a href="https://www.runoob.com/cplusplus/cpp-tutorial.html">C++ 教程 | 菜鸟教程</a>
<a href="https://wizardforcel.gitbooks.io/lcthw/content/preface.html"> 笨办法学C</a>

---

> 涉及的讲义

一生一芯 v26.07讲义
- <a href="https://ysyx.oscc.cc/docs/2607/e/2.html">E2 C语言程序设计 | 官方文档</a>
一生一芯 v24.07讲义
- <a href="https://ysyx.oscc.cc/docs/2407/e/4.html">E4 从C代码到二进制程序 | 官方文档</a>

---

# 指针

指针的初始化：
```c
//对于地址：
data = 0x5ffe60; //让指针指向"0x5ffe60"这个位置，这个位置可以是空闲or被占用；但是由于权限与安全限制，不能直接指向并且写入下面的值

//可以这样写：
data = (int*)malloc(4) //申请一块4byte的空间，malloc函数返回这个空间的首位bit的地址，这里将无类型指针转为 int* 让data接收
//or 
data = (int*)malloc(sizeof(int)) //int类型占用4bytes，所以sizeof(int) == 4

//-----------------------------------------------
//对于值：(不能直接读写未申请空间的地址)
*data = 3.14 //让指针指向的某个地址的值赋值为3.14(事实上是先把3.14这个值赋值给type为double的隐藏变量，然后再把这个值截断传递给data指向的区域。)((可以理解为把这个区块0x5ffe60的比特赋值为 0x00000003 )
```

回顾：
```c
printf("%p %p %d %p", data, data, *data, *&data)//这里*&data就是*(&data)，取得&data这一块地方的东西（指针），可以简单理解为&*相互抵消
```


> Linux C编程一站式学习习题补充
# 参数的传递

> <a href="https://akaedu.github.io/book/">Linux C编程一站式学习</a>
> 章节3-3 课后习题
## 习题1
在调用函数时，参数`通过复制（默认）`或`传递其引用`来传递。

```c
void increment(int x)
{
	x = x + 1;
}

int main(void)
{
	int i = 1, j = 2;
	increment(i); 
	increment(j); 
	return 0;
}
```

| 序号  | 操作（对应行）            | 内存中的变量                  |     |     |
| --- | ------------------ | ----------------------- | --- | --- |
| 1   | 声明并复制i, j          | i=1, j=2                |     |     |
| 2   | increment(i)--入栈   | i=1, j=2, increment_x=1 |     |     |
| 3   | increment(i)--执行自加 | i=1, j=2, increment_x=2 |     |     |
| 4   | increment(i)--出栈   | i=1, j=2                |     |     |
| 5   | increment(j)--入栈   | i=1, j=2, increment_x=1 |     |     |
| 6   | increment(j)--执行自加 | i=1, j=2, increment_x=2 |     |     |
| 7   | increment(j)--出栈   | i=1, j=2                |     |     |
| 8   | return 0; 之后       | 空                       |     |     |

## 习题2
如果在一个程序中调用了`printf`函数却不包含头文件，例如`int main(void) { printf("\n"); }`，编译时会报警告：`warning: incompatible implicit declaration of built-in function ‘printf’`。请分析错误原因。

- **头文件与实现的区分：** 我们要明确一点：`<stdio.h>` 头文件中只包含了 `printf` 的**函数声明（原型）**，用于编译期的类型检查；而 `printf` 的真正**代码实现**位于系统的标准C库（libc）中，由链接器负责接入。
    
- **早期C标准（C89/C90）的处理机制 —— 为什么是 Warning？**
    <a href="https://hypothes.is/a/A0jzssywEemSol95zyJm3w">Hypothesis</a>
    - **隐式声明（Implicit Declaration）：** 当编译器看到一个没有提前声明的函数时，它不会立刻报错，而是按照规则自动生成一个“隐式声明”，即假定该函数返回 `int` 类型，且不限制参数类型（类似于 `int printf();`）。
        
    - **内建函数冲突（Built-in Function）：** 现代编译器（如GCC）为了优化性能，内部自带了 `printf` 等常用标准库函数的模型（即内建函数）。编译器内部知道 `printf` 的真实原型是 `int printf(const char * restrict format, ...);`。
        
    - **报错原因总结：** 编译器自己生成的“隐式声明”，与它内部记录的“内建函数原型”在参数类型（比如缺乏 `restrict` 限定符和不定参数标识）上**不兼容（incompatible）**，因此抛出此特定警告。
        
- **现代C标准（C99及以后）的变化 —— 为什么变成了 Error？**
    <a href="https://www.cnblogs.com/litifeng/articles/5638574.html">Linux C一站式编程习题答案 - 立体风 - 博客园</a> 
    - 现代C语言标准（C99开始，并在现代编译器如 GCC 14 / Clang 15 中严格执行）**彻底废除了隐式函数声明**。
        
    - 在现代环境下编译相同的代码，不再是报警告，而是直接引发编译终止的**致命错误 (Error)**。报错信息通常为：`error: implicit declaration of function 'printf' is invalid in C99`（不允许隐式声明函数）。这就强制要求程序员在使用任何函数前，必须严格提供对应的头文件或显式声明，保障了代码的类型安全。

<a href="https://www.open-std.org/jtc1/sc22/wg14/">ISO/IEC JTC1/SC22/WG14 - C</a>
<a href="https://www.open-std.org/jtc1/sc22/wg14/www/docs/n3096.pdf">ISO/IEC 9899:2023</a>


# 增量式开发

> <a href="https://akaedu.github.io/book/">Linux C编程一站式学习</a>
> 章节5-2

除了教材的例子，我们可以把要实现的东西作为一个类，类里面放变量（性质）和函数（方法）

例如：
```c
class Enemy;
class Game1{ //class Game是声明，大括号里面是定义的内容。结构体struct也是这样，和class同源。不同之处：struct默认public，class默认private.
// game1.h实现的内容
    //实体
    Gui gui;
    std::list<Enemy*> enemies;
	    //计时器
    int cycle;
    int bombertick;
public:
	Game(int _row, int _col, int _tick);
    ~Game();
    void spawnEnemy();
//more..........

// game1.C实现的内容
void Game1::spawnEnemy(){//补足敌人
    if (cycle % (7*tick) == 0){
        if (gunboatCount < maxgunboat) {
            spawnObjects("Gunboat");
            gunboatCount++;
        }
        if (destroyerCount < maxdestroyer){
            spawnObjects("Destroyer");
            destroyerCount++;
        }
        if (cruiserCount < maxcruiser) {
            spawnObjects("Cruiser");
            cruiserCount++;
        }
    }
}
//more and more..........
};

//如果要实现多游戏切换，可以考虑再加一个类game2，然后实现一个gameloader选择一个游戏（其实就是array里面存 指向类的实例的指针，用的时候调用就行
```

为了管理方便 : ) ，当然也可以把函数的实现和变量的定义分开来写（见上文注释）

# 递归
> <a href="https://akaedu.github.io/book/">Linux C编程一站式学习</a>
> 章节5-3
## 知识点
教材讲得挺好的，或许有些不好理解

## 汉诺塔问题

<a href="https://zh.wikipedia.org/wiki/%E6%B1%89%E8%AF%BA%E5%A1%94">汉诺塔 - 维基百科，自由的百科全书</a>

实现

```c
#include <iostream>
using namespace std;

void Towers(int n,char a,char b,char c){ //目标：将n个盘 a->b->c
	if(n==1){
		cout<<"Move disk "<<n<<" from"<<a<<" to "<<c<<endl;
	}
	else{
		Towers(n-1,a,c,b); // 将n-1个盘 a->b（经过c）
		cout<<"Move disk "<<n<<" from"<<a<<" to "<<c<<endl; //将底下的1个盘 a->c
		Towers(n-1,b,a,c); //将那n-1个盘 b->c (经过a）
	}
}
int main(int argc, char *argv[]) {
	int n;
	cin>>n;
	Towers(n,'A','B','C');
	cout<< endl;
	return 0;
}
```

# 小项目_matrix

以下是一份约100lines的代码：

```c
#include <stdio.h>
#include <stdlib.h>
#include <stdbool.h>

// ---------- 矩阵结构体 ----------
typedef struct {
    int rows;
    int cols;
    int *data;      // 长度为 rows * cols
} Matrix;

typedef a;
struct Matrixs{
    int a;
};

// ---------- 创建 / 销毁 ----------
Matrix* matrix_create(int rows, int cols) {
    Matrix *m = (Matrix*)malloc(sizeof(Matrix));
    if (!m) return NULL;
    m->rows = rows;
    m->cols = cols;
    m->data = (int*)malloc(rows * cols * sizeof(int));
    if (!m->data) {
        free(m);
        return NULL;
    }
    return m;
}

void matrix_destroy(Matrix *m) {
    if (m) {
        free(m->data);
        free(m);
    }
}

define MAT_AT(m, i, j) ((m)->data[(i) * (m)->cols + (j)])

bool matrix_read(Matrix *m) {
    for (int i = 0; i < m->rows; ++i) {
        for (int j = 0; j < m->cols; ++j) {
            if (scanf("%d", &MAT_AT(m, i, j)) != 1)
                return false;
        }
    }
    return true;
}

// ---------- 打印矩阵 ----------
void matrix_print(const Matrix *m, const char *name) {
    printf("%s : (row %d, col %d)\n\n", name, m->rows, m->cols);
    for (int i = 0; i < m->rows; ++i) {
        for (int j = 0; j < m->cols; ++j) {
            printf(" %d", MAT_AT(m, i, j));
        }
        printf("\n");
    }
    printf("\n");
}

Matrix* matrix_multiply(const Matrix *A, const Matrix *B) {
    if (A->cols != B->rows) {
        printf("Dimension mismatch: A.cols(%d) != B.rows(%d)\n", A->cols, B->rows);
        return NULL;
    }

    Matrix *C = matrix_create(A->rows, B->cols);
    if (!C) return NULL;

    int *c_ptr = C->data;
    for (int i = 0; i < A->rows; ++i) {
        for (int j = 0; j < B->cols; ++j) {
            int sum = 0;
            // 计算 C[i][j]
            for (int k = 0; k < A->cols; ++k) {
                sum += MAT_AT(A, i, k) * MAT_AT(B, k, j);
            }
            *c_ptr++ = sum;
        }
    }
    return C;
}

// ---------- 主程序 ----------
int main() {
    int ar, ac, br, bc;
    if (scanf("%d %d %d %d", &ar, &ac, &br, &bc) != 4) {
        fprintf(stderr, "Failed to read dimensions.\n");
        return 1;
    }

    Matrix *A = matrix_create(ar, ac);
    Matrix *B = matrix_create(br, bc);
    if (!A || !B) {
        fprintf(stderr, "Memory allocation failed.\n");
        matrix_destroy(A);
        matrix_destroy(B);
        return 1;
    }

    if (!matrix_read(A) || !matrix_read(B)) {
        fprintf(stderr, "Failed to read matrix data.\n");
        matrix_destroy(A);
        matrix_destroy(B);
        return 1;
    }

    Matrix *C = matrix_multiply(A, B);
    if (!C) {
        matrix_destroy(A);
        matrix_destroy(B);
        return 1;
    }

    matrix_print(A, "matrix_a");
    matrix_print(B, "matrix_b");
    matrix_print(C, "matrix_c");

    matrix_destroy(A);
    matrix_destroy(B);
    matrix_destroy(C);
    return 0;
}
```

《Linux C编程一站式学习》提到，阅读代码不能像看小说或者看题目一样从头读到尾，需要从主干到枝叶；在学习过程中，可以通过画函数之间的关系图、看函数和变量名帮助理解代码的行为。

## main函数是程序入口
我们知道，在C/C++中，main函数是程序入口，终端每次运行C/C++编译后的可执行文件，会从main函数执行，以main函数返回数值而宣告程序的结束。所以阅读代码可以从main函数开始，看他里面定义了什么变量，调用了什么函数。

可以看到，main函数里面使用了`scanf`这一标准输入函数，`fprintf`这一输出函数(专门用于输出报错信息)

## 一个合格的程序，至少要有输出，最好要有输入
为什么？可以上网搜。
根据这一点，程序的重要关注部分就是==输入的格式、输出的内容==。或许有时候看不到输出的内容，他们被封装在其他函数里面，让这个函数专门用于实现输出功能。比如，从关键词"print"可以看出，这个程序的`matrix_print`很可能实现了一部分输出内容，此时溯源定位原函数位置，发现确实如此。


| **类比** | 输入(Input) | 输出(Output)      |
| ------ | --------- | --------------- |
| 程序     |           |                 |
| 函数     | 传参        | 返回值/直接修改传入的参数引用 |
| AI模型   | 张量        | 张量              |
|        |           |                 |

---
## 这么多return看起来眼花缭乱

有人说，main函数里面这么多return，返回值还不一样，究竟返回第几个呢？

首先，一个合格的代码不会只运行第一个return，让后面的代码永远得不到运行。

注意看，前面4个return都被包裹在if控制下的代码块里面。（代码块里面的变量和函数在程序执行到末尾时就会清除）当一种条件成立，进入代码块运行相应代码并return结束main函数，后面的代码块不会被运行。这样就变相实现了if--else的功能，也更美观。

在工程中，==单独列出的if条件用于排除一些特例，而循环结构更能使得程序千变万化，实现更丰富的功能。==因此阅读过程中， 看完的if分支即可选择性忽略，在心中给代码进行剪枝以突出代码核心功能。

---
## 程序跳读：看完main看有关的函数
暂时忽略掉if-else分支后，我们把注意力集中在`matrix_create` `matrix_print` `matrix_destroy` 这三个函数。

### matrix_create()
虽然他见缝插针在if代码块之间，但是却很重要。
我们可以通过`ctrl+f`定位到所有地方，找定义函数的语法以跳到这个函数的位置。

在这个函数中，我们可以看到：`if xxxxx, return NULL;` 这其实是一种“检查不通过，返回零值表示不存在、不合法的情况。（在python则可以用 return None)

- `Matrix *m = (Matrix*)malloc(sizeof(Matrix));`
这里是一个等式，先看等号右边。
等号右边是malloc动态开辟内存的语法，官方定义这种函数传入开辟空间的大小(单位为byte，这里传入大小`sizeof(Matrix)`），返回开辟空间的第一个内存地址。由于默认为`void*`，需要通过(Matrix*)实现转换。
等号左边是新定义的指针m，规定需要指向结构体Matrix.

**易错**：把`(Matrix*)` 写成`(Matrix)` .这样就试图把指针`void*`转换为结构体`Matrix`，很可能报错。

- `m->data = (int*)malloc(rows * cols * sizeof(int));`
类似，开辟大小为`rows * cols * sizeof(int)`的空间P，传给指针data(这里data属于实例M的对象，其中指针m指向他)，调用这个data需要`(*m).data`（先访问指针m指向的结构体，再访问所属的对象data, 简化语法后就是这样`m->data` 。

可以理解为：
```c
int &*v = (int*)malloc(rows * cols * sizeof(int));
m->data = v;
```
<br>
<p align="center">
  <img src="resource/Pasted image 20260706085943.png" alt="Pasted image 20260706085943.png" width="100%" />
</p>
<br>


如果注释够多，我们也对矩阵定义熟悉的话，就可以知道matrix_create()可以创建一个矩阵，把开辟的空间data、row和col用结构体打包，方便管理和调用。


---
### matrix_read()
一开始我想先讲`matrix_multiply()`，但是发现后面也有宏定义`MATAT`，那么还是先从代码量少的matrix_read开始。

---
### matrix_multiply()

multiply：乘法。

这里假设读者学过矩阵。我们看矩阵一般就是一个有行有列的数字阵。代码也是希望这么维护的，从row, col, `MATAT`都能看出来。这里维护一个二维矩阵，进行乘法。

把变量k的for循环提到最外层，减少跳跃读取，增加缓存命中率。



# 继承与多态

实现继承和多态，C语言需要全手工模拟虚函数表（vtable）和虚指针（vptr），而C++的语法便于实现继承和多态。
## 继承

面向对象程序设计中最重要的一个概念是继承。C语言是面向过程语言，

定义
- `依据另一个类`（基类/父类，Base/Parent/Super），来`定义一个类`（派生类，Derived）就是继承。
作用
- 达到了`重用代码功能`和`提高执行效率`的效果，更容易创建和维护一个应用程序。

## 多态

产生条件
- 当类之间存在层次结构，并且类之间是通过继承关联。
作用
- 允许使用基类指针或引用来调用子类的重写方法，从而使得同一接口可以表现不同的行为。也就是说，`程序可以通过基类指针或引用来操作不同类型的对象`，而不需要显式区分对象类型。这样可以使代码更具扩展性，在增加新的形状类时不需要修改主程序。

---
## 举例

### 例子1：Person && Student
virtual关键字暗示程序让该代码使用多态，多态这一机制使得程序可以通过基类Person的指针来调用Student的属性（变量）和方法（函数）。
```c
#include <iostream>
using namespace std;

class Person {
    int DOB; //day of birth
    int MOB; //month of birth
    int YOB; //year of birth
public:
    Person(int d, int m, int y)
        : DOB(d), MOB(m), YOB(y) {}
    virtual void dailyActivity() { cout << "None\n"; } 
	~Person() {}
};

class Student : public Person {
    int SID;
public:
    Student(int d, int m, int y, int s)
        : Person(d, m, y), SID(s) {}
    void dailyActivity() { cout << "Study\n"; }
    void study() { cout << "In class\n"; }
};

int main () {
	//p1, p2, s1, s2的类 和 指针的指向的类是对应的，p3是没有对应的
    Person p1 (10, 10, 1980);
    p1.dailyActivity(); //直接访问类对象
	Person* p2 = new Person (1, 1, 1981); //以指针调用类
    p2-> dailyActivity();   
    
    Student s1 (9, 10, 2006, 1001);
    s1.dailyActivity();        
    Student* s2 = new Student (10, 30, 2006, 1002);
    s2-> dailyActivity();   

    // Polymorphism impact (多态的影响)
    // 核心对比：同样是基类指针，这次输出了子类的行为
    Person* p3 = new Student (11, 3, 2006, 1003);
    p3-> dailyActivity();         // 

    delete p2;
    delete s2;
    delete p3;
    
    return 0;
}
```

输出依次为：
None
None
Study
Study
None (未加入virtual) / Study（加入virtual关键字）

---
### 例子2：不同的车

```c
#include <iostream>
#include <utility>
#include <cmath>
#include <algorithm>

// 1.基类车
class Vehicle {

protected:
    double weight; 
    int wheels; 
    double pos_x, pos_y; 
    double current_oil, max_oil;

public:
    Vehicle(double w, double mo) : weight(w), wheels(4), pos_x(0), pos_y(0), current_oil(mo), max_oil(mo) {}
    virtual ~Vehicle() {} // 父类必须有虚析构

// 加入 virtual关键词，开启多态机制
	virtual void set_value(int val) {}
	virtual int get_value() {return 0;}
	virtual void cut_down(int loss) {}
	
	virtual void reduce_oil(double delta_oil) {current_oil -= delta_oil;}
	virtual void report_oil() {printf("Oil: %.2f / %.2f\n", current_oil, max_oil);}
    
    virtual void run(double dx, double dy) {
	    double dis = sqrt(dx*dx + dy*dy);
	    //可增加油不够的情况，然后改动dx, dy
        reduce_oil(dis*0.05);
        
        pos_x += dx; pos_y += dy;
        current_oil -= (dx * dx + dy * dy) * 0.01;
        std::cout << "移动至 (" << pos_x << ", " << pos_y << ")" << std::endl;
    }
};

// 2.轿车
class Car : public Vehicle {
private:
	int value;
public:
	Car(double w, double mo) : Vehicle(w, mo), value(100000) {}
	Car(double w, double mo, int v) : Vehicle(w, mo), value(v) {}
	~Car() {}
	
	//这里用到的函数，为了让外界以指向Vehicle的指针统一管理，需要手动在基类实现
	virtual void set_value(int val) override {value = val;} 
	virtual int get_value() override {return value;} 
	
	// 重写虚函数
    virtual void run(double dx, double dy) {
        Vehicle::run(dx, dy);
    }
    
};

// 3.跑车
class RaceCar : public Vehicle {
private:
    int tire_durability;
public:
    RaceCar(double w, double mo) : Vehicle(w, mo), tire_durability(1000) {}

    virtual void cut_down(int loss) {
        tire_durability = std::max(0, tire_durability - loss);
        std::cout << "  -> 轮胎磨损，剩余耐久: " << tire_durability << std::endl;
    }

    // 重写虚函数
    virtual void run(double dx, double dy) {
	    //可以直接移用base class实现过的run函数
        Vehicle::run(dx, dy);
        cut_down(50);
         //也可以重新实现一个更加符合跑车（省油）的油耗计算公式
        /*
        double dis = sqrt(dx*dx + dy*dy);
        reduce_oil(dis*0.03);
        
         pos_x += dx; pos_y += dy;
        current_oil -= (dx * dx + dy * dy) * 0.01;
        std::cout << "移动至 (" << pos_x << ", " << pos_y << ")" << std::endl;
        */
    }
};

int main() {
    RaceCar myRaceCar(1000.0, 50.0);
    Car mycar(1000.0, 50.0, 123456);
    
    Vehicle* rPtr = &myRaceCar; //race_car_ptr
    Vehicle* cPtr = &mycar; //car_ptr 
    
    rPtr->run(10.0, 10.0); 
    cPtr->run(10.0, 10.0);
    rPtr->report_oil();
    cPtr->report_oil();
    
    printf("%d\n", cPtr->get_value());
    cPtr->set_value(100000);
    printf("%d\n", cPtr->get_value());
    return 0;
}
```
(代码块1-1)


#### 前置理解

##### 1.访问控制修饰符
在 C++ 中，访问控制修饰符（`private` / `protected` / `public`）规定了谁有权限读写类的成员：
- **`private`（私有）：** 只有**类自己**的成员函数和友元可以访问。即使是亲儿子（派生类），也完全被挡在门外。
- **`protected`（受保护）：** **类自己**以及它的所有派生类（子类、孙子类等）**的成员函数都可以访问，但**类外部（如 `main` 函数）依然无法访问。
- **`public`（公有）：** 任何人（内部、子类、外部 `main` 函数）都可以自由访问。
- 
代码中的protected可以但不建议改成private+友元关键词friend，后者一般用于特殊情况，需要让某个变量单独公开被外界访问。

举例：
```c
class Person {
private:
    int dateOfBirth;
    // `声明`一个外部函数是我的“好朋友”，允许它访问我的 private 变量
    friend void printBirthDate(const Person& p);
    // `声明`另一个类是我的“好朋友”，那个类里的所有方法都能访问我的 private 变量
    friend class Staff; 
    
    // 声明输出函数 作用1：方便debug，让这两个函数拥有Person类的"public"权限。
    friend std::ostream& operator<<(std::ostream& os, const Person& p) { 
	    os << "DOB: " << p.dateOfBirth; 
	}
	friend void myDebugPrint(const Person& p);
};
void myDebugPrint(const Person& p) { // 因为是朋友，所以可以直接无视 private 限制. 
	cout << "Age: " << p.secretAge << endl; 
}
```

##### 2.virtual的作用
- 使用virtual关键字，开启多态功能。
- 多态功能：让`程序可以通过基类指针或引用来操作不同类型的对象`。见下面的例子2

##### 3.override/虚函数

<a href="https://en.cppreference.com/w/cpp/language/override">override specifier</a>

"In a member function declaration or definition, `override` specifier ensures that the function is virtual and is overriding a virtual function from a base class. The program is ill-formed (a compile-time error is generated) if this is not true."

作用：为了确保正在重写基类（父类）中的某个虚函数
#### 对代码块1-1的理解

- 代码块1-1实现了Vehicle, Car, RaceCar这三个类，其中Car, RaceCar继承自Vehicle，共用protected部分的属性——重量weight，轮子数量，坐标posx/y，当前油量/最大油量。
- Car在Vehicle的基础上，增加了估价value部分，并且增加了修改和调用的函数接口set_value和get_value。
- RaceCar在Vehicle的基础上，增加了轮胎磨损机制，并且在重载run函数的时候调用。
- main函数实例化了两辆车，并且在外部访问操作而非类似 `vPtr->value = 100;` 直接修改实例对象。

### 例子3：类的生命周期

预测程序结果：

```c
#include <iostream>
using namespace std;

class Person {
    int DOB;
public:
    Person(int d) : DOB(d) {}
    virtual ~Person() {cout << "PD\n";}
    virtual void dailyActivity() {cout << "None\n";}
    int getDOB() {return DOB;}
};

class Staff : public Person {
    int salary;
public:
    ~Staff() {cout << "SD\n";}
    Staff(int d, int s) : Person(d), salary(s) {}
    void dailyActivity() {cout << "Staff\n";}
    void setSalary(int s) {salary = s;}
};

class Faculty:public Staff {
    int courseID;
public:
    ~Faculty(){ cout << "FD\n";}
    Faculty(int d, int s, int c) : Staff(d, s), courseID(c) {}
    void dailyActivity() {cout << "Faculty\n";}
    void setCourse(int c) {courseID = c;}
};

int main () {
    Person* p = new Staff(1, 1000);
    p->dailyActivity();
    cout << p->getDOB() << endl;

    Staff* s = new Faculty(10, 2000, 101);
    s->dailyActivity();

    Faculty* f = new Faculty(10, 2000, 101);
    f->dailyActivity();

    delete p;
    delete s;
    delete f;
    return 0;
}

```

output:

>> Staff
>> 1
>> Faculty (加入多态，类的“上限”随指向的类，反之，随min(指针属性, 指向的类) )
>> Faculty

delete部分：
>> SD
>> PD

- **构造时**：先递归初始化基类 `Person`，再初始化成员，最后执行派生类 `Staff` 的构造函数体。
- **析构时**：先执行派生类 `Staff` 的析构函数体，再析构派生类的成员，最后才调用基类 `Person` 的析构函数。(按照栈的顺序)
>> FD
>> SD
>> PD

>> FD
>> SD
>> PD



## 手动实现虚函数表和虚指针 (选看)

> 可以自行STFW~

示例：
- `vtable` 是**针对每一个多态类（Class）独立创建的一张只读静态表格**，所有同类对象共享它。
- `vptr` 是**每个具体对象（Object）体内隐藏的一个指针**，它专心指向该类对应的 `vtable`。 这样做的最大收益是**节省内存**：无论类里声明了 1 个还是 100 个虚函数，每个实例对象所付出的额外空间代价永远只有这 1 个 `vptr` 指针的大小（64位系统下固定为 8 字节）。

```c
#include <stdio.h>

struct _Base; // 前置声明

// 1. 真正的虚函数表（所有虚函数的集中登记表）
typedef struct _Base_VTable {
    void (*func)(struct _Base*);
    // 后续如果有 func2, func3... 都可以继续加在这里，而不会增加 Base 对象自身的体积
} Base_VTable;

// 2. 基类结构体：包含一个指向虚表的指针 (vptr)
typedef struct _Base {
    const Base_VTable* vptr; 
} Base;

// 3. 派生类结构体
typedef struct _Derived {
    Base base;        // 继承基类（内含 vptr）
    int derivedData;  // 派生类特有数据
} Derived;

// -------- 派生类函数实现与虚表 --------
void Derived_func(Base* b) {
    Derived* d = (Derived*)b; // 向下转型
    printf("Derived function, data: %d\n", d->derivedData);
}

static const Base_VTable g_derived_vtable = {
    .func = Derived_func 
}; //只读

void Derived_init(Derived* d, int data) {
    d->base.vptr = &g_derived_vtable; // 挂载派生类的虚表
    d->derivedData = data;
}

int main() {
    Derived d;
    Derived_init(&d, 10); // 构造派生类对象

    // 向上转型为基类指针
    Base* b = (Base*) &d;

    // 标准的虚函数派发路径：对象 -> vptr -> vtable -> func
    b->vptr->func(b);

    return 0;
}
```

### 前置理解

#### 1.强制类型转换
`Base* b = (Base*) &d;` 可行;
- 指针类型转换的时候 : 数据方面，改变的是指向的空间大小，解读规则和指针计算与地址移动的规则；
<br>
<p align="center">
  <img src="resource/Pasted image 20260815113828.png" alt="Pasted image 20260815113828.png" width="100%" />
</p>
<br>

#### 2. 指定初始化器 (Designated Initializers)

在 C 语言结构体初始化时出现的 `.func = Base_func`：

```c
static const Base_VTable g_base_vtable = {
    .func = Base_func
}; //虚函数表 指针地址常量

void Base_init(Base* b) {
    b->vptr = &g_base_vtable;
}
```

运行时调用：`b->vptr->func(b)`
- b->vptr 指向 Base_VTable的静态映射表
- func(b) 查找 .func 映射到的地址 (0x00401122)

#### 3. 函数指针语法 (Function Pointers)

在虚函数表中出现的 `void (*func)(struct _Base*);`：

```c
typedef struct _Base_VTable {
    void (*func)(struct _Base*);
} Base_VTable;
```

这是 **C 和 C++ 专门用来声明“函数指针”的独立语法规则**。
- **`(*func)`：告诉编译器，`func` 本身是个**指针变量**，而不是一个返回值为指针的普通函数。
- **`(struct _Base*)` 参数**：声明这个函数的参数

#### 4.不完全声明 && typedef语法
避免“先有鸡先有蛋”的问题
```c
struct _Base; // 前置声明
typedef struct xxx{//声明Base_VTable 需要用到_Base
    void (*func)(struct _Base*);
} Base_VTable;

typedef struct _Base { // 声明Base 需要用到 Base_VTable
    const Base_VTable* vptr; 
} Base;
```

由于历史遗留问题，为了调用简洁，在C语言一般在声明结构体的时候补一个typedef。

`typedef origin_name global_name`
上文：
- `origin_name = struct{.....}` 
- `global_name = Base_VTable`

所以可以这样写：
```c
struct Base;
struct Base_VTable{
    void (*func)(struct Base*);
};
struct Base {
    const struct Base_VTable* vptr; 
};
//后面把Base_VTable替换为 struct Base_VTable，比较麻烦
```

## 引用来源

<a href="https://www.runoob.com/cplusplus/cpp-inheritance.html">C++ 继承 | 菜鸟教程</a>

---


# 集成编译工具：Make

> How To Make ファイル
## 0. 常见的工程项目结构

一个规范的 C/C++ 工程项目通常包含以下结构：

```
LICENSE       # 如果在项目中发布源码，建议包含一份开源协议。未指定协议时他人通常无权使用。
README.md     # 项目说明文档（Markdown 格式）。
Makefile      # 项目的核心构建自动化脚本。
bin/          # 存放最终生成的可执行程序（通常初始为空，由 Makefile 生成）。
build/        # 存放中间编译产物（如 .o 库文件，通常初始为空）。
src/          # 存放源代码文件（如 .c 和 .h）。
tests/        # 存放自动化测试代码。
assets/       # 存放项目静态资源（图片、数据等）。
```

---

## Why choose makefile ?

### 1. 便于一键使用编译命令
- 帮助我们在几天后或几个月后还能记得怎么编译并使用项目。
- 方便别人一键部署源代码并且使用
	- 为了节约空间和时间，传输项目一般不会包括可执行文件。
	- 类似的思想：requirement.txt，里面可能放了很多的pip install xxx，便于到手后灵活使用（比如过几天给电脑腾出空间了再使用）
	
```makefile
app: main.o hello.o
	gcc main.o hello.o -o app

main.o: main.c
	gcc -c main.c -o main.o

hello.o: hello.c
	gcc -c hello.c -o hello.o

clean:
	rm -f main.o hello.o app
```

---
### 2. 写好脚本，先构思再行动
- 写脚本方便管理要运行的指令，能更好把握计算机的行为 （你喜欢以下行为吗）

```bash
gcc -o main main.o sub1.o sub2.o
gcc -c sub1.o sub1.c
# 报错---找不到源文件 or 编译错误
# 频繁按 Up 方向键翻找上一条命令...
......
./main -g
```

- 看看更高效的工作流，这样方便一些吧！
```bash
pico main.c
make
./main
```

- 与shell结合一下
```shell
source ./test.sh #这里面可以包装指令，构建程序行为，也方便工程传递，开箱即用
```
test.sh:
```shell
make clean
make
./main [自定义参数]
```

---
### 3. 分离相同命令，复用行为
#### 请记住基本的指令行为：
```
目标(target)...: 依赖(prerequisites)...
	命令(command)
#注意是<Tab>不是4个空格字符；Vscode会正确翻译成<Tab>而不是4个空格的。
```

- 和选项1的原始代码看看有何不同 (看变量的引入和百分号%的使用)

```
TARGET = mygame 
all: $(TARGET)

$(TARGET): main.o Game.o Gui.o Battleship.o Shell.o Enemy.o Gunboat.o Destroyer.o Island.o Cruiser.o Bomber.o Missile.o Pack.o
	g++ -Wall -g -o $(TARGET) main.o Game.o Gui.o Battleship.o Shell.o Enemy.o Gunboat.o Destroyer.o Island.o Cruiser.o Bomber.o Missile.o Pack.o -lncurses

%.o: %.C
	g++ -Wall -g -c $< -o $@

clean:
	rm -f main.o Game.o Gui.o Battleship.o Shell.o Enemy.o Gunboat.o Destroyer.o Island.o Cruiser.o Bomber.o Missile.o Pack.o mygame
```

- **模式匹配（`%.o: %.C`）：** 彻底消灭了所有形如 `main.o: main.C`、`Game.o: Game.C` 的重复规则。无论未来项目扩展到几十还是上百个 `.C` 文件，全都会自动套用这同一套编译==行为==。 (行为：比如这里的-g, -Wall.....)
    
- **自动化变量（`$<` 与 `$@`）：**
    
    - `$<` 代表当前触发规则的**第一个依赖**（即具体对应的 `.C` 源文件）。
        
    - `$@` 代表当前触发规则的**目标**（即对应的 `.o` 文件）。
        
- **目标变量解耦（`TARGET`）：** 将最终可执行程序的文件名提取为 `TARGET = mygame`。后续更换项目名称时，只需修改顶部一处配置。


- ⚠️错误示范如下：
```
TARGET = mygame

all: $(TARGET)
$(TARGET): main.o Game.o Gui.o Battleship.o Shell.o Enemy.o Gunboat.o Destroyer.o Island.o Cruiser.o Bomber.o Missile.o Pack.o
	g++ -Wall -g -o $(TARGET) main.C Game.C Gui.C Battleship.C Shell.C Enemy.C Gunboat.C Destroyer.C Island.C Cruiser.C Bomber.C Missile.C Pack.C -lncurses

%.o: %.C
	g++ -Wall -g -c $< -o $@

clean:

	rm -f main.o Game.o Gui.o Battleship.o Shell.o Enemy.o Gunboat.o Destroyer.o Island.o Cruiser.o Bomber.o Missile.o Pack.o

	rm -f main.C Game.C Gui.C Battleship.C Shell.C Enemy.C Gunboat.C Destroyer.C Island.C Cruiser.C Bomber.C Missile.C Pack.C
```

---

### 4.用变量名简洁表达

```
SRCS = main.C Game.C Gui.C Battleship.C Shell.C Enemy.C Gunboat.C Destroyer.C Island.C Cruiser.C Bomber.C Missile.C Pack.C
OBJS = $(SRCS:.C=.o) #这里是把SRC里面提到的.C文件映射为.o文件，方便表述
TARGET = mygame

all: $(TARGET)

$(TARGET): $(OBJS)
	g++ -Wall -g -o $(TARGET) $(OBJS) -lncurses

%.o: %.C
	g++ -Wall -g -c $< -o $@

clean:
	rm -f $(OBJS) $(TARGET)

# 这里可以理解为把all clean作为保留词，不当作一个文件名all.C (不加也行)
# 没谁会起名这种"general"的名字吧！
.PHONY: all clean 
```

- 假如还想偷懒或者不想在文件的“正文”看到具体的参数（比如选用的编译工具g++, 编译模式-g....)
```
CXX = g++
CXXFLAGS = -Wall -g
LDFLAGS = -lncurses

SRCS = main.C Game.C Gui.C Battleship.C Shell.C Enemy.C Gunboat.C Destroyer.C Island.C Cruiser.C Bomber.C Missile.C Pack.C

OBJS = $(SRCS:.C=.o)
TARGET = mygame

all: $(TARGET)

$(TARGET): $(OBJS)
	$(CXX) $(CXXFLAGS) -o $(TARGET) $(OBJS) $(LDFLAGS)

%.o: %.C
	$(CXX) $(CXXFLAGS) -c $< -o $@

clean:
	rm -f $(OBJS) $(TARGET)
```

==注意==：
选项1讲了“最初的makefile"用于偷懒，
选项2讲了假设构建好makefile和shell文件后的使用方法
选项3, 4讲了用模式匹配（隐式规则）和变量修饰来使得makefile更简洁可读（但是不一定什么都需要用变量名包装）
选项3只是 作用于模式复用，而真正的Target, Source还是需要自己写出来。

这里介绍一下自动search项目文件的方法，需要的同学可以STFW or STFAI

### 5. wildcard 函数自动搜集源文件
- `$(wildcard *.c)`：获取目录下所有的 `.c` 文件列表（例如 `main.c hello.c`）。
- `$(patsubst %.c, %.o, $(SRCS))`：把列表中所有的 `.c` 替换为 `.o`（例如 `main.o hello.o`）。

- 那么，上述的变量SRC和OBJS可以这么改：
```
SRCS = $(wildcard *.c)
OBJS = $(patsubst %.c, %.o, $(SRCS))
```

### 6.其他小技巧
- 在命令前加 `@`，可以避免命令被终端打印出来，只输出执行命令里面的具体内容的反馈。
```shell
clean:
	@echo "Cleaning up build artifacts..."
	@rm -f $(OBJS) $(TARGET)
	@echo "Zzzzzz..."
```


### 7. Warning & Error
| **现象 / 报错信息**                                          | **公式层面的根本原因**                                               | **Make 的底层思考逻辑**                                 | **解决/排查方向**                                         |
| ------------------------------------------------------ | ----------------------------------------------------------- | ------------------------------------------------ | --------------------------------------------------- |
| **`'app' is up to date.`**                             | `target` 存在，且其时间戳**新于或等于**所有 `prerequisites`。               | “所有的源文件都没改过，`app` 已经是最新产物，不需要重新编译。”              | 若修改了 `.h` 仍提示此报错，说明 Makefile 未将 `.h` 写进依赖关系。        |
| **`missing separator. Stop.`**                         | `command` 行开头**没有使用严格的 `Tab` 键**（误用了空格）。                    | “我无法识别这一行是属于上面 `target` 的命令，语法分隔失败。”             | 检查命令行首，确保是 `\t` 而非空格。                               |
| **`No rule to make target 'X', needed by 'Y'. Stop.`** | 目标 `Y` 需要依赖 `X`，但磁盘上**无 `X` 文件**，且 Makefile 中**无 `X:` 规则**。 | “我要造 `Y` 必须先拿到 `X`，但我找不到 `X`，也不知道怎么造 `X`，依赖链断裂。” | 检查 `X` 是否拼写错误、文件是否被删、或是否漏写了 `%.o: %.c` 这类规则。        |
| **`'clean' is up to date.`** (运行 `make clean` 却不清理)    | 规则写为 `clean:`（依赖为空），且**当前目录下恰好存在名为 `clean` 的文件/文件夹**。       | “`clean` 目标存在，且没有依赖文件比它更新，我认为它已是最新，跳过命令。”        | 在 Makefile 中添加 `.PHONY: clean` 显式声明其为伪目标，强制忽略时间戳检查。 |

至于makefile调用makefile等其他进阶行为，请STFW or STFAI
至于为啥不使用shell来记录编译文件而是用makefile，原因至少有：makefile可以check哪些源码没有被修过，就不重复编译那一部分源码。makefile在编译时是多线程的（自动维护），编译量一多了效果自然可观。

makefile的知识，现在看起来似懂非懂没关系，至少要会用整洁的脚本定义一批指令，完成一个小任务，写脚本的时候，多想多构思，等到XPU跑起来，能解放双手亿会会，那才是真的爽！

（那些奇怪的语法和指令，用到了再说吧！）
（别找我，自行STFW和ATFAI吧！）

