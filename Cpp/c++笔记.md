# CPP 笔记

##  一. 面向对象


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