<h1 style="text-align: center;font-size:40px"> live555笔记 </h1>

# 一、编译

- 1、
  ```c++
  CROSS_COMPILE?=		/home/wangyc/ARMTool/RK3566/sysroots/x86_64-pokysdk-linux/usr/bin/aarch64-poky-linux/aarch64-poky-linux-
COMPILE_OPTS =		--sysroot=/home/wangyc/ARMTool/RK3566/sysroots/aarch64-poky-linux $(INCLUDES) -I/home/wangyc/ARMTool/RK3566/include -I. -I/opt/pushpullcbb/openssl/rk3566/include -DSOCKLEN_T=socklen_t -DNO_SSTREAM=1 -D_LARGEFILE_SOURCE=1 -D_FILE_OFFSET_BITS=64 -DNOVA_ENABLE_OFFICIAL_SRTP -g -std=c++17
C =			c
C_COMPILER =		$(CROSS_COMPILE)gcc --sysroot=/home/wangyc/ARMTool/RK3566/sysroots/aarch64-poky-linux 
C_FLAGS =		$(COMPILE_OPTS)
CPP =			cpp
CPLUSPLUS_COMPILER =	$(CROSS_COMPILE)g++
CPLUSPLUS_FLAGS =	$(COMPILE_OPTS) -Wall -DBSD=1
OBJ =			o
LINK =			$(CROSS_COMPILE)g++ --sysroot=/home/wangyc/ARMTool/RK3566/sysroots/aarch64-poky-linux -o
LINK_OPTS =		
CONSOLE_LINK_OPTS =	$(LINK_OPTS)
LIBRARY_LINK =		$(CROSS_COMPILE)ar cr 
LIBRARY_LINK_OPTS =	$(LINK_OPTS)
LIB_SUFFIX =	a
PREFIX = /home/wangyc/softEncoder/live555/live555_20200819/build_rk3566/
LIBS_FOR_CONSOLE_APPLICATION = -lssl -lcrypto
LIBS_FOR_GUI_APPLICATION =
EXE =

  ```

