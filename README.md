# A-simple-drawing-language-interpreter
### 背景

这是一个简单绘图语言解释器。

2021年学习编译原理时，通过此项目加深对编译器构造原理和方法的理解，巩固所学知识。

目的：

1. 会用正规式设计简单语言的词法
2. 会用产生式设计简单语言的语法
3. 会用递归下降子程序编写语言的解释器

### 安装

本项目使用C++编写，使用Visual Studio 2019在Windows10下开发。

代码结构：

- scanner 词法分析器的全部代码

- parser 语法分析器的全部代码（仅语法分析）

- semantics 语义分析（计算），应用于在 parser 代码中，以构成“解释器”

- errlog 错误处理，信息输出

- ui_main 人机界面+主程序，包括两种实现：

  - 文本交互界面（console），主函数是 main()

  - 图形界面（WinGUI），主函数是 WinMain()

### 简单绘图语言规则

1. 5种语句：

   - 循环绘图（FOR-DRAW）

   - 比例设置（SCALE）

   - 角度旋转（ROT）

   - 坐标平移（ORIGIN）

   - 注释        （-- 或 //）

2. 屏幕（窗口）的坐标系

   - 左上角为原点
   - x方向从左向右增长
   - y方向从上到下增长（与一般的坐标系方向相反）


3. 源程序举例 

```
--------------- 函数f(t)=t的图形
origin is (100, 300);										 -- 设置原点的偏移量
rot is 0;																 -- 设置旋转角度(不旋转)
scale is (1, 1);												 -- 设置横坐标和纵坐标的比例
for T from 0 to 200 step 1 draw (t, 0);	 -- 横坐标的轨迹（纵坐标为0）
for T from 0 to 150 step 1 draw (0, -t); -- 纵坐标的轨迹（横坐标为0）
for T from 0 to 120 step 1 draw (t, -t); -- 函数f(t)=t的轨迹 
```

默认值：

```
    origin is (0, 0); 
    rot is 0;
    scale is (1, 1);
```

### 程序分析

1. 词法分析器（目录 scanner）

1.1 源文件 & 编译

- scanner.h  词法分析器对外提供的接口声明

- scanner.c  词法分析器的实现

- dfa.c  定义了 表驱动型 DFA 及其操作，共 scanner.c 使用

- scannermain.c  词法分析测试主程序，仅测试词法分析时使用

1.2  采用“表驱动”模式实现，定义在文件 dfa.c 中。

- 将 DFA 的使用抽象为三个接口操作，从而将实现细节封装起来。当构造“直接编码型”词法分析器时，仅需要修改这三个函数的实现即可。
- 将表驱动型 DFA 的状态转换矩阵当作“稀疏矩阵”，并采用了基于三元组的“压缩”存储策略。

1.3  错误处理策略

- 当打开文件失败时，仅通过 InitScanner() 的返回值向调用者报告。
- 对于词法分析发现的错误，统一由 GetToken() 返回一个种类为 ERRTOKEN 的记号，而该错误的显示、处理均交由调用者负责。词法分析测试主程序中，凡是输出记号类别为 ERRTOKEN 的都是这些错误。

2. 语法分析器

2.1 源文件 & 编译

- ​	parsermain.c  语法分析测试主程序，仅测试语法分析时使用.
- ​	parser.h  语法分析器对外提供的接口声明
- ​	parser.c  语法分析器的实现

2.2 实现模式:  这里实现的是“递归下降的语法分析器”。

2.3 关于语法树

​	仅为各类表达式构造语法树，且一个语句分析结束后，

​	该语句对应的全部语法树会被销毁（参见 statement 的递归下降函数）。

2.4  程序输出

​	错误信息输出：写入文件 error.log，也会显示到屏幕上。

​	其他信息输出：写入文件 parser.log ，并不输出到屏幕上。

2.5 错误处理策略

​	但凡发现语法错误、词法错误，则显示错误信息之后，立即结束程序（很简单，也很粗暴）。

3. 语义分析与解释器

​	这个实现将 语法分析、语义分析与绘图操作交织在一起，所以该模块定义了必要的语义子程序 semantics.c，并对 parser/parser.c 中，四个语句（*Statement）对应的递归下降子程序体中，增加了语义计算操作。 

3.1 源文件

​	-- 下面两个文件是语义计算的辅助程序，为 parser.c 服务。

​		semantics.h  语义计算的辅助函数声明

​		semantics.c  语义计算函数的实现

​	-- 下面两个文件是语法分析器的，需要配合语义分析进行修改：

​		parser.h  语法分析器对外、对语义分析提供的接口声明

​		parser.c  {添加了语义动作}的语法分析器，它 #include "semantics.h"

3.2  程序输出

​	错误信息输出：写入文件 error.log。

​	其他信息输出：写入文件 parser.log ，并不输出到屏幕上。

3.3 错误处理策略

​	但凡发现语法错误、词法错误，则显示错误信息之后，立即结束程序（很简单，也很粗暴）。
