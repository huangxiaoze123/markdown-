<h1 style="text-align: center;font-size:40px"> cmake笔记 </h1>

# 一、简介

- 1、CMake是一个跨平台的安装（编译）工具，可以用简单的语句来描述所有平台的安装(编译过程)。他能够输出各种各样的makefile或者project文件，能测试编译器所支持的C++特性,类似UNIX下的automake。只是 CMake 的组态档取名为 CMakeLists.txt。Cmake 并不直接建构出最终的软件，而是产生标准的建构档（如 Unix 的 Makefile 或 Windows Visual C++ 的 projects/workspaces），然后再依一般的建构方式使用。这使得熟悉某个集成开发环境（IDE）的开发者可以用标准的方式建构他的软件，这种可以使用各平台的原生建构系统的能力是 CMake 和 SCons 等其他类似系统的区别之处。
简单来说就是当多个人用不同的语言或者编译器开发同一个项目，最终需要输出一个可执行文件或者共享库（dll,so等），这个时候就可以利用CMake了（如果你不使用CMake，你可能就需要利用gcc/g++去逐个编译），所有的操作都是通过变异CMakeLists.txt来完成的。

# 二、语法介绍
 
- 1、PROJECT关键字
    - project关键字可以用来指定工程的名字和支持的语言，默认支持所有语言
- 2、SET关键字
    - SET关键字用来显示指定的变量
- 3、MESSAGE关键字
    - MESSAGE关键字主要用于向终端输出用户自定义的信息，主要包含三种信息

      - SEND_ERROR，产生错误，生成过程被跳过
      - STATUS，输出前缀为–的信息
      - FATAL_ERROR，立即终止所有cmake过程
- 4、ADD_EXECUTABLE关键字
    - ADD_EXECUTABLE关键字用于生成可执行文件

# 三、CMake语法的基本原则与注意事项

## 1、基本原则

- 变量使用${}方式取值，但是在 IF 控制语句中是直接使用变量名

- 指令(参数 1 参数 2…) 参数使用括弧括起，参数之间使用空格或分号分开。 以上面的 ADD_EXECUTABLE 指令为例，如果存在另外一个 func.cpp 源文件就要写成：ADD_EXECUTABLE(hello main.cpp func.cpp)或者ADD_EXECUTABLE(hello main.cpp;func.cpp)

- 指令是大小写无关的，参数和变量是大小写相关的。但推荐你全部使用大写指令

## 2、注意事项

- SET(SRC_LIST main.cpp) 可以写成 SET(SRC_LIST “main.cpp”)，如果源文件名中含有空格m ain.cpp，就必须要加双引号

- ADD_EXECUTABLE(hello main) 后缀可以不写，他会自动去找.c和.cpp，最好不要这样写，可能会有这两个文件main.cpp和main

# 四、内部构建与外部构建

- 内部构建，生产的临时文件特别多，不方便清理

- 外部构建，就会把生成的临时文件放在build目录下，不会对源文件有任何影响强烈使用外部构建方式

# 四、使用外部共享库和头文件

## 1、解决make后头文件找不到

- 第一种：include <hello/hello.h>

- 第二种：利用关键字：INCLUDE_DIRECTORIES 这条指令可以用来向工程添加多个特定的头文件搜索路径，路径之间用空格分割

在CMakeLists.txt中加入头文件搜索路径

## 2、解决引用的函数问题

报错信息：undefined reference to `HelloFunc()’

关键字：LINK_DIRECTORIES 添加非标准的共享库搜索路径

指定第三方库所在路径，LINK_DIRECTORIES(/home/myproject/libs)

关键字：TARGET_LINK_LIBRARIES 添加需要链接的共享库

TARGET_LINK_LIBRARIES的时候，只需要给出动态链接库的名字就行了。

在CMakeLists.txt中插入链接共享库，主要要插在executable的后面

注意：64位的虚拟机需要执行一下mv操作, 就可以执行./hello 了

## 3、链接静态库

TARGET_LINK_LIBRARIES(main libhello.a)

特殊的环境变量 CMAKE_INCLUDE_PATH 和 CMAKE_LIBRARY_PATH

注意：这两个是环境变量而不是 cmake 变量，可以在linux的bash中进行设置

我们上面例子中使用了绝对路径INCLUDE_DIRECTORIES(/usr/include/hello)来指明include路径的位置

我们还可以使用另外一种方式，使用环境变量export CMAKE_INCLUDE_PATH=/usr/include/hello

补充：生产debug版本的方法：
cmake … -DCMAKE_BUILD_TYPE=debug