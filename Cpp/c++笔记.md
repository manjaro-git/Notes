

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

 #### 1.3 this 与 static

**this**



**static 成员**


##  二. 标准库和泛型编程



##  三. 工程

C++ Programming Web Resource ：[3D Game Engine Programming](https://www.3dgep.com/tag/filesystem/)

C/C++ MemoryManagement : [Building your own memory manager for C/C++ projects – IBM Developer](https://developer.ibm.com/tutorials/au-memorymanager/?mhsrc=ibmsearch_a&mhq= )

游戏编程模式: [命令模式 · Design Patterns Revisited · 游戏设计模式 (tkchu.me)](https://gpp.tkchu.me/command.html)

###  1. OpenGL

#### 1.1 Vertex Buffer

GPU中存取数据的内存空间，我们以字节的形式将数据传送给GPU

~~~c++
unsigned int buffer;
glGenBuffers(1, &buffer);
glBindBuffer(GL_ARRAY_BUFFER, buffer); // 将标识符绑定到GL_ARRAY_BUFFER上面
glBUfferData(GL_ARRAY_BUFFER, sizeof(data),data,GL_STATIC_DRAW); // 指定vertex buffer的大小
~~~

#### 1.2 指定属性

OpenGL中每个vertex都是包含许多属性(比如位置，颜色，纹理等)的节点,我们通过glBufferData传递过去的是一堆字节流，我们需要告诉OpenGL怎么理解这段字节流

~~~C++
// 指定属性以及数据的分布
glVertexAttribPointer(GLuint index,GLint size, GLenum type, GLBoolean normalized, GLsizei stride, const GLvoid* pointer)
~~~

上面函数每个参数的一次，可以去docs.gl查找

指定完属性后，由于OpenGL对一个属性的默认为关闭，所以我们还需要打开属性

~~~C++
glEnableVertexAttribArray(index); // index 表示属性下标
~~~

#### 1.3 Shader

Shader是GPU上运行的小程序，我们平时主要编程的，就是VertexShader，和FragmentShader，VertexShader处理节点信息，FragmentShader处理像素渲染信息。

##### 1.3.1 shader 编写

**vertex shader**

~~~glsl
#version 440
layout(location =0) in vec4 position;
void main(){
    gl_Position = position
}
~~~

**fragment shader**

~~~glsl
#version 440
out vec4 color;
void main(){
    color = vec4(1.0f, 1.0f, 0.0f, 1.0f);
}
~~~

##### 1.3.2 CompileShader

1. 获取源码
2. 编译shader

~~~C++
static unsigned int CompileShader(const std::string& source){
    unsigned int VertexShader;
    glShaderSource(&VertexShader, 1, &source.c_str(),NULL);
    glCompileShader(VertexShader);
    // error handle here
    
    return VertexShader;
}
~~~



##### 1.3.3 获取报错信息

~~~C++
unsigned int VertexShader;
// Shader Error Handle
int result;
glGetShaderiv(VertexShader, GL_COMPILE_STATUS，&result);
if(result == 0 ){
    int length;
    glGetShaderiv(VertexShader,GL_INTO_LOG_LENGTH,&length);
    char* message = (char*)alloca(length*sizeof(char));
    glGetShaderib(VertexShader, length,&length,message);
    std::cout<<"Failed to Compile"<< "VertexShader"<<message<<std::endl;
}
~~~



##### 1.3.4 LinkShader

将vertexShader和fragmentShader链接到program里面

~~~C++
static unsigned int CreateProgram(const std::string& VertexSource, const std::string& FragmentSource){
    unsigned int program = glCreateProgram();
    unsigned int VertexShader = CompileShader(VertexSource);
    unsigned int FragmentShader = CompileShader(FragmentSource);
    
    glAttachShader(program, VertexShader);
    glAtttachShader(program, FragmentShader);
    glLinkProgram(program);
    
    // Handle Error
    int result;
    glGetProgramiv(program, GL_LINK_STATUS, &result);
    if(result == 0){
        int length;
        glGetProgramiv(program, GL_INFO_LOG_LENGTH,&length);
        char* message = (char*) alloc(message* sizeof(char));
        glGetProgramInfoLog(program, length, &length, message);
        std::cout<<"Link Program Error: "<<message<<std::endl;
    }
    
    // 删除shader
    glDeleteShader(VertexShader);
    glDeleteShader(FragmentShader);
    
    return program;
}
~~~



#### 1.4 Index Buffer

为了节省空间，我们使用IndexBuffer

~~~C++
unsigned int indices[] = {
    0,1,2,
    2,3,0
}; // index必须是unsigned类型

unsigned int ibo;
glGenBuffers(1, &ibo)
glBindBuffer(GL_ELEMENT_ARRAY_BUFFER, ibo);
glBufferData(GL_ELEMENT_ARRAY_BUFFER, sizeof(indices), indices, GL_STATIC_DRAW);

// render
glDrawElements(GL_TRIANGLES, 6, GL_UNSIGNED_INT, nullptr);
~~~



#### 1.5 OpenGL Error Handle

~~~C++
#define ASSERT(x) if(!(x)) __debugbreak()

#ifdef GLDEBUG
#define glCall(x) glClearError();\
	x;\
	ASSERT(glCheckError(#x , __FILE__, __LINE__));
#else
#define glCall(x) x;
#endif
static void glClearError(){
    while(glGetError() != GL_NO_ERROR);
}

static bool glCheckError(const char* function, const char* file, int line){
    if((GLenum error =glGetError()) != GL_NO_ERROR){
        std::cout<<"OpenGL Error: ("<< error<<") ["<<function<<"] "<<file<<": "<<line<<std::endl;
        return false;
    }
    return true;
}


// 使用debug函数
glCall(glDrawElements(Gl_TRIANGLES, 6, GL_INT, nullptr));
~~~

#### 1.6 uniform

`uniform`变量是除了使用`ertex buffer`以外，可以从cpu传递数据给gpu的方法，下面是要点

+ `uniform` 变量在每次draw函数调用时都会设置一次
+ 先获取`uniform`变量的位置，在设置`uniform`的值

~~~C++
int location = glCall(glGetUniformLocation(program, "u_Color"));
ASSERT(location != -1);
glUniform4f(location,1f,2f,3f,4f);
~~~

下面是`shader`文件的改变:
~~~glsl
uniform vec4 u_Color;  // 改成uniform 变量
void main(){
    color = u_Color;
}
~~~



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



## 四. Desing Patterns

### 1. 命令模式

将命令/请求/事件变成对象。这就是函数回调的面向对象版本

~~~C++
class Command{
public:
    virtual void execute() = 0;
};

class JumpCommand:public Command{
public:
    virtual void execute()override {jump();}
};

class RunCommand:public Command{
public:
    virtual void execute()override {run();}
};
class FightCommand:public Command{
public:
    virtual void execute()override {fight();}
};

class InputHandler{
private:
    Command* button_x;
    Command* button_y;
    Command* button_z;
};
void InputHandler::Handle(){
    if(IsPressed(BUTTONX)) button_x->execute();
    if(IsPressed(BUTTONY)) button_y->execute();
    if(IsPressed(BUTTONZ)) button_z->execute();
}
~~~

**Notes:将抽象的概念具象化,从而更加方便处理**

进一步，将命令作为第一考虑对象，我们可以构建如下的基于命令流的结构:

![image-20210714165553809](images/image-20210714165553809.png)

由输入器或者AI产生一堆命令，通过一个类似队列的命令流，然后传给解释器或者Actor角色对象，它们会消耗或者使用命令。这就是一种对生产者消费者的解耦。

下面是把命令作为第一对象，而游戏角色作为命令对象的参数的例子:

~~~C++
class Role{
public:
    void jump();
    void run();
};

class Command{
public:
    virtual void execute(Role* role) = 0;
};

class JumpCommand:public Command{
public:
    virtual void execute(Role* role) override{
        role->jump();
    }
};

class RunCommand:public Command{
public:
    virtual void execute(Role* role) override{
        role->run();
    }
};

class InputHandler{
public:
    // 将命令作为第一对象返回，我们可以通过这种方法实现一种懒处理
    Command* handle(){
        if(IsPressed(button_x)) return new JumpCommand();
        if(ISPressed(button_y)) return new RunCommand();
        return nullptr;
    }
};

void main(){
    Role *role = GetRole();
    while(true){
        Command* command = handle();
        if(command){
            command->execute(role);
        }
    }
}
~~~



**有了命令作为对象，我们还可以实现撤销的动作，只要实现一个命令列表，用一个当前current指针指向现在的命令，如果要撤销，就移动指针退后，具体情况如下:**

![image-20210714171212603](images/image-20210714171212603.png)

