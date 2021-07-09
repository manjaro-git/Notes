# CPP 笔记

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

默认拷贝就是单纯的按照元素逐个拷贝的(element-wise copy).

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