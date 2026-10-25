#DynamicX课堂笔记
#日期2026.10.22
#智能感知工程 康宸斌

#Cmake笔记

**注意：project为可执行程序名称，my_libs为我创建的目录，该笔记中会记录g++如何工作，Makefile、Cmake、Catkin是什么和怎么工作的**

#**这些东西是什么**
在代码程序编写中，g++、Makefile和Cmake是三种编写工具。

1.若将编写比作盖房子，g++是一块一块砖头盖即用g++编译每一个文件，当文件数量多起来时，g++则会显得繁琐且容易出错。
2.Makefile则相当于包工头，根据提供的图纸（Makefile这个纯文本文件）进行编写。
3.Cmake：Cmake相当于总设计师，根据提供的需求说明书（CmakeList.txt)，自动根据当前使用的系统，帮忙画出Makefile能执行的图纸。本质上就是一个自动化的生成图纸的工具，省去了制定符合使用系统格式的图纸的时间和工作。
4.Catkin是ROS（机器人操作系统）里面的构建系统，是rosbuild的替代者，是基于Cmake进行拓展，为了方便管理ROS的众多依赖包和复杂工程结构的系统。

#一、g++怎么编写文件

**单个文件编写**：`g++ main.cpp -o project` 
`-o`代表输出（output)，`project`贼是生成的可执行程序的名称。`-o`如果缺失，则会默认生成一个a.out的文件

**多个文件编写**:`g++ main.cpp test.cpp -o project`

**头文件搜索**：`g++ main.cpp test.cpp -I./include -o project`
`-I`(include)即去`./include`这个目录中找寻头文件。

**链接外部库**：
1.系统自带的库：`g++ main.cpp -lm -o project`
-l(link)表示链接，对于系统自带的库如`lib`则去/user/lib等标准目录中去找，-lm则是链接`libm.so`（数学库）。
2.自己下载的非系统自带的库`g++ main.cpp -L./my_libs -limage -o project`
-L则是去指定目录即我创建的`my_libs`目录中去找`libimage.so`文件。
**补充**：在Linux系统中，库文件通常以lib开头，以.so（动态库）或.a(静态库）结尾。`libm.so`中实际是lib+m+.so组成。

**调试与优化**：`g++ main.cpp -g -o project`
`-g`表示在可执行文件中加入调试信息，方便用gdb调试时查看源码。
##补充gdb相关信息：
###1.gdb是什么
gdb（GNU Debugger即GNU调试器）可以称之为程序运行是的“X光机”用于随时暂停程序来查看程序中变量的变化。
###2.gdb常用指令
启动调试：gdb ./main
设置断点：（gdb) break 10
运行到第10行时让它停止
运行程序：（gdb) run或者r
单步执行：
执行下一行代码，当遇到函数时不进去：（gdb) next或n
会进入函数：(gdb) step或s
打印变量的值：（gdb） print 变量名或写为（gdb） p 变量名
退出：（gdb） quit或q

#二、Makefile如何工作
**注意：g++这行代码前面一定用tab缩进**
如：
`makefile`
```
my_project:main.cpp
    g++ main.cpp -o my_project 
 clean:
    rm -f my_project
 install:
    cp my_project /usr/local/bin/
```
##1.在目录中找Makefile或makefile这个图纸并默认访问里面的第一个文件。
##2.若只想要默认则输入`make`，若要指定则输入`make clean`这样跟文件名。
##3.查找依赖 如`my_project`后面冒号接的便是该文件的依赖`main.cpp`。
##4.比对修改时间 ：跳过未修改文件，直接执行修改文件节约时间（编译大型项目快速的主要原因）。

##工作特性
###1.可使用变量替换
如‵ａ = g++`后面就可以使用$(a)来代替g++这一个编译名称。
###2.具有模式化规则
如：

```
#最终执行目标：生成依赖main.o和math.o生成my_project这一可执行文件
my_project:main.o math.o
    g++ main.o math.o -o my_projrct
#将所有.cpp文件生成.o文件
%o: %.cpp
    g++ -c $< -o $@
#清理规则:删除所有.o文件和最终的可执行程序（可用于清理编译过程中编译失败后产生的.o垃圾文件）
clean：
    rm-f *.o my_projrct
```
第一行表示任意名字即如果需要生成.o文件则去寻找.c文件生成。`%`表示任意名字，如果make发现需要main.o时，`%`就是main。
第二行表示不管遇到哪个.cpp都用g++编译成.o文件。`$<`和`$@`叫做自动变量（专属于Makefile）。`$<`代表第一个依赖文件（如上文的main.cpp)，`$@`代表目标文件（如main.o）。

#三Cmake如何工作
##1.建立CmakeLists.txt文件
```
# 1. 声明要求的 CMake 最低版本3.10
cmake_minimum_required(VERSION 3.10)

# 2. 声明项目名称
project(MyProject)

# 3. 添加可执行文件（目标名 源文件）
add_executable(my_program main.cpp math.cpp)
```
**注解**：`project（Myproject）`中Myproject会被存入一个Cmake的名为PROJECT_NAME内置变量中。
如：
```
project(MyProject) 
add_executable(${PROJECT_NAME} main.cpp) 
# 这里的 ${PROJECT_NAME} 就自动替换成了 MyProject
```
**好处**：如果想更改项目名称只用更改第一行，大大节约了时间和降低了出错的可能性。
##2.建立文件夹build
```
mkdir build
cd build
```
**注意**：在源码目录下建立文件夹————外部构建，使源码目录保持干净。
##3.在文件夹里执行命令
```
# 第一步：让 CMake 根据上一级目录的 CMakeLists.txt，生成 Makefile
cmake ..

# 第二步：调用 make 包工头，执行真正的编译工作
make
```
###Cmake的核心语法
1.set(）：定义变量
2.include_directories():告诉编译器去哪里找头文件（.h) -> g++ -I。
3.target_link_libraries():告诉链接器链接什么库 -> g++ -l。
4.find_package():自动补全所需的头文件和库文件路径，大大节约了时间和事情的繁琐性。

#资料补充

