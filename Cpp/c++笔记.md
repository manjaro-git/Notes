# CPP 笔记

## 零. 基本语法

### 1. static_assert



##  一. 面向对象

### 1. Class

#### 1.1 class basic

+ 类是一个用户定义的类型
+ 内部有数据成员和成员函数
+ 公有函数是接口，私有函数是实现细节
+ struct 是默认访问控制为public的class
+ 类是一个将数据及处理数据的函数包含起来的命名空间
+ 成员函数定义四种基本的使用：初始化，拷贝，移动，销毁

**成员数据和成员函数**

数据成员是类的成员，成员函数是将类和数据连接起来的纽带。

成员函数是唯一可访问私有数据成员和成员函数的函数集合(不考虑友元)。

**默认拷贝**

默认的，类支持两种拷贝:拷贝的构造和拷贝的赋值。

默认拷贝就是单纯的**按照元素逐个拷贝**的(element-wise copy).

**访问控制**

+ public
+ private
+ protected

对类设置访问控制有以下好处:

1. 便于调试
2. 便于用户学习接口
3. 如果功能改变，不需要改变接口，那么依赖于接口的那些代码都不需要改变。
4. **良好的结构设计会产生更好的代码**，这回减少我们在debug上花费的时间。

notes: 对私有成员的保护是依赖于对类成员名字的使用的限制，但是我们仍然可以通过地址操作和显式的类型转换来访问数据成员。



**class 和 struct **

class X{...}; 这是类的定义，但是由于历史原因，这其实指的是类的声明。所以，类的定义是可以通过#include的方式包含在不同的源文件中，并且不会违反one-definition rule。

下面两种定义等价:

~~~C++
struct S{...};
class S{public:...};
//具体怎么使用，依赖于个人爱好，但是一般的规则是，struct只用于一些简单的数据结构。
~~~



**构造器**

使用init()函数来构造一个类是非常不优雅的。因为我们没办法说明一个类是必须被初始化的，程序员可能会忘记初始化一个类或者将一个类初始化两次。更好的方法是显式的声明一个函数来初始化对象:constructor.constructor 和类名相同。

~~~C++
class Date{
  int d,m,y;
public:
    Data(int dd, int mm, int yy);
};
~~~

构造器定义了类的初始化，我们可以使用{}-initializer 符号:

Data today = Data{23, 6, 1983};

Data xmas {22, 5, 1000};

**default arguments**

为了满足多种初始化的需求，我们通常需要定义多个constructor，但也可以使用default arguments来减少构造函数的使用。

~~~C++
class Data{
  public:
    Data(int dd =0, int mm =0, int yy=0);
};

Data::Data(int dd, int mm, int yy){
    d = dd;
    m = mm;
    y = yy;
}
~~~



conclusion: 给定了构造器，那么其他的成员函数处理的一定是已经初始化的数据。



**显式构造器 (explicit Constructor)**

默认的，一个构造器(可能有多个默认参数)被单个参数值**调用**,这个行为，可以被编译器解释为一种隐式转换(参数类型->类的类型)。具体例子如下:

~~~C++
Consider Data:
void my_fct(Data d);
void f()
{
    Data d{15};
    my_fct(15); // 模糊 15->Data->my_fct(Data)
    d =15;   // 模糊 15->Data->d
}
~~~



我们可以避免这种隐式转换， 只要对构造函数的声明使用explicit就行。

具体如下:

~~~C++
class Data{
  int d,m,y;
 public:
    explicit Data(int dd=0, int mm =0, int yy=0);
};

Data d1 = {15}; // error:explicit 
~~~

notes:如果一个构造函数使用explicit声明，而它定义在类外，那么类外的定义就不能再加上explicit。



**类内初始化**

减少构造函数数量的方法除了默认参数，还有就是进行数据成员的类内初始化。

下面的例子值得看一下:

~~~C++
class Data{
private:
    int d{today.d};
    int m{today.m};
    int y{today.y};
    Data(int dd):d(dd){}
    
    //上面的构造函数等价于:
    Data(int dd):d(dd),m(today.m), y(today.y){ }
};
~~~



**类内的函数定义 **

+ 类内函数定义，会变成inline函数，适合小，不常更改，经常使用的函数

+ 函数类内定义和类定义一样，可通过#include粘贴多出

+ 类内的成员声明是顺序无关的

+ ~~~C++
  #include<iostream>
  class Data{
      int d,m,y;
   public:
   	Data(int dd):d(dd){
          
      }
     // inline 函数，顺序无关
      void add_date(int n){d += n;}
  }
  ~~~

**const 成员函数**

如果在函数列表后面加上const，这样的函数就是const 成员函数。如下面:

~~~python
class Data{
public:
    void add_month()const{
        
    }
};
~~~

const 成员函数可以被**const**对象，和**non-const**对象调用，但是如果是非const成员函数，那么只能被非const对象调用。

const是函数的一种属性，如果要声明定义一个const(成员)函数，那么不管是声明还是定义，都必然要在参数列表后面加上const。

#### 1.2  const 和 mutability 

很多时候，我们对于一个数据可能大多时候保持不变，但是偶尔需要改变，或者说对于用户而言，并不直接改变，但是在无法观测到的地方需要改变，这就要考虑类内数据的可变性问题。

**小对象，小数据的改变**

如果是比较小的对象，且类内数据只有少数需要更改，可以使用mutable关键字,如下：

~~~c++
class Data{
    mutable double m_Data;
 public:
    Data(int data):m_Data(data){
        
    }
    
    void Change(int n)const{ m_Data += data;}
};

void main(){
    const Data cd(10);
    cd.Change(20); // ok,m_Data为mutable，即使cd为const，也可调用更改
}
~~~



**大对象，多数据的改变**

mutable这种方式，只适合小对象的小部分数据，如果是比较复杂的数据，可以使用cache的方法:

~~~c++
struct Cache{
double data,
bool valid;
std::string str;
};

class Data{
    Cache* c;
public:
    Data(Cache *c_init):c(c_init){}
    void Change()cosnt{
        c->data = 100;
        c->vaild = false;
        c->str = "Cache";
    }
};
~~~

对于Data来说，Cache* c 并没有改变，然而其实我们还是修改了data，valid等值，这种方法可进一步扩展，称为“惰性求值"

### 1.3 this 与 static

**this**



**static 成员**


##  二. 标准库和泛型编程



##  三. 工程

C++ Programming Web Resource ：[3D Game Engine Programming](https://www.3dgep.com/tag/filesystem/)

C/C++ MemoryManagement : [Building your own memory manager for C/C++ projects – IBM Developer](https://developer.ibm.com/tutorials/au-memorymanager/?mhsrc=ibmsearch_a&mhq= )

###  1. OpenGL
###  2. Hazel
###  3. Premake

premake 的官方文档网址:[Premake](https://premake.github.io/docs/postbuildcommands)

~~~Lua
workspace "MyWorkSpace"
	configurations{"Debug", "Release"}
	startproject "Sanbox"
outputdir = "%{cfg.system}-%{cfg.architecture}-%{cfg.buildcfg}"

include "Hazel/vendor/GLFW"
include "Hazel/vendor/Glad"


project "Hazel"
	location "Hazel"
	kind "SharedLib"
	language "C++"
	files
	{
    	"**.h",
    	"**.cpp"
	}
	includedirs
	{
    	"Hazel/src"
	}
	
	pchheader ("hzpch.h")
	pchsource ("hzpch.cpp")	

	targetdir ("bin/" .. outputdir .. "/%{prj.name}")
	objdir ("int-bin/" .. outputdir .. "/%{prj.name}")
	postbuildcommands{"{COPY} %{cfg.buildtarget.relpath} bin/".. outputdir .. "/sandbox" }
	
	filter {"configurations:Debug","system:windows"}
		pic "on"
		staticruntime "off"
		defines {"HZ_PLATFORM_WINDOWS"}
		symbols "On"
	filter "configurations:Release"
		staticruntime "off"
		defines "NDEBUG"
		optimize "on"
		pic "on"
project "sandbox"
	kind "ConsoleApp"
	language "C++"
	location "sandbox"
	
	links "Hazel"

	includedirs
	{
    		
	}

	files
	{
    	
	}
	
	targetdir ="bin/" .. outputdir .. "/%{prj.name}"
	objdir = "int-bin/" .. outputdir .. "/%{prj.name}"
	filter {"configurations:Debug"}
		staticruntime "on"
		defines "DEBUG"
		symbols "On"
	
~~~



###  4. GLFW
###  5. Spdlog