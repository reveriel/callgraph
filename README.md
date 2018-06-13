# Call Graph Generator
<!-- TOC -->

- [Call Graph Generator](#call-graph-generator)
    - [Call Graph](#call-graph)
    - [设计](#设计)
    - [评价](#评价)
    - [问题](#问题)
    - [相关工具](#相关工具)
    - [相关工具简介.](#相关工具简介)
        - [Egypt](#egypt)
        - [ncc](#ncc)
        - [KcacheGrind](#kcachegrind)
        - [Graphviz](#graphviz)
        - [CodeViz](#codeviz)
        - [doxygen](#doxygen)
        - [Understand](#understand)
        - [tceetree/CScope](#tceetreecscope)
        - [DMS Software Reengineering Toolkit](#dms-software-reengineering-toolkit)
        - [Smatch](#smatch)
        - [cflow](#cflow)
        - [CppDepends](#cppdepends)
        - [Code Graph (Visual Studio plugin)](#code-graph-visual-studio-plugin)
    - [Papers](#papers)

<!-- /TOC -->

## Call Graph

构建函数调用图的目的可以是作为后面的过程间优化的基础, 或者是帮助程序员理解程序.

## 设计

多层次的调用图.

- 项目(project)
- 文件夹/库(folder/lib)
- 文件(module/file)
- [类(Class)]
- 函数(function)
- 行(line)

比如 File - Func - Line

- File a, File b.
   文件的调用关系: 两个文件a,b 中 a中存在一行代码调用了 b 中定义的函数.
(问题: 定义和声明如何处理?)
-  File a, Func b.
  a 中存在一行代码调用了 b.
-  Func a, Func b.
  a 中存在一行代码调用了 b.
-  Line a, Func b.
   a 调用了 b.

交互式, 点击节点可以展开. File > Func > Line.

class 对象的 method 组织聚集到一起, 同文件的函数聚集到一起.


## 评价

- 效率.
- 准确度.
- 可读性. 是否有助于理解程序, 比如能够让程序员一眼看出软件的体系结构和设计模式.
- 可用性, 是否需要很多的配置工作.

## 问题

- presentation: huge call graph

  函数

- indirect call, function pointer

  对于通过函数指针的函数调用需要准确的指针分析. 指针分析除了在构建调用图上的问题上会用到, 对于编译器内部的各种优化也是很有用的.
  在调用图问题上, 指针分析的特别之处在于跨模块的分析. (具体一点?)

- reflection, dynamic library

  语言支持的动态调用, 其被调用的函数的函数名可能是程序中的一个字符串变量, 难以在运行前确定.

- C++ virtual function calls

  同样是指针分析的问题.

- build system

  大的 project 可能编译为了编译就需要很多配置, 或者使用了某些 build tool, 在不同的配置下可能有不同的 call graph.

- inline function

  内联函数可能在编译后消失, 但是也应该在图中体现.

- macro function 的表示, 条件编译

  C/C++ 程序中很多代码看起来是函数调用, 实际调用的是一段宏. 但是对于用户来说, 宏和真正的函数是看不出区别的, 一般也是将这样的宏看做函数.

  条件编译的宏会导致编译出不同的代码, 这个问题可能已经被 cpp 预处理器解决了.

- C++ template

## 相关工具

C project 函数调用图生成.

- [Egypt](http://www.gson.org/egypt/) (free software) (tiny)
- [ncc](http://students.ceid.upatras.gr/~sxanth/ncc/) (free) (small)
- [KcacheGrind](https://kcachegrind.github.io/html/Home.html) (GPL) (dynamic)
- [Graphviz](http://www.graphviz.org/) (GPL) (C++)
- [CodeViz](https://github.com/petersenna/codeviz) (GPL) (small)
- [doxygen](http://www.stack.nl/~dimitri/doxygen/) (free)
- [Understand](https://scitools.com/features/) (commercial)
- [tceetree/cscope](https://sourceforge.net/projects/tceetree/) (free) (small)
- [DMS Software Reengineering Toolkit](http://www.semanticdesigns.com/Products/DMS/DMSToolkit.html) (commercial)
- [GNU cflow](http://www.gnu.org/s/cflow/) (GPL) (small)
- [CppDepend](https://www.cppdepend.com ) (commercial)
- [Code Graph](https://marketplace.visualstudio.com/items?itemName=YaobinOuyang.CodeAtlas) (Visual Studio Plugin)

ref :

- [Tools to get a pictorial function call graph of code | stackoverflow](https://stackoverflow.com/questions/517589/tools-to-get-a-pictorial-function-call-graph-of-code)
- [Generate calling graph for C++ code | stackoverflow](https://stackoverflow.com/questions/5373714/generate-calling-graph-for-c-code)

## 相关工具简介.

C 的 Call graph 工具很多都是比较老的小工具. C++ Call Graph 生成工具很少.
应该说是, 生成完整 Call Graph 的工具很少, 因为稍微大一点的 project 的 call
graph 就很复杂了.

比如 CodeViz 提供 一个脚本 `gengraph` , 输入一些函数的集合,
输出包含这些函数的子图. Doxygen, CppDepend, Code Graph, Understand 这些
商业的或常用的工具生成的 call graph 都是以一个函数为中心, 包含一定深度
的 caller 或 callee 的图.

cflow, tceetree/cscope, ncc 生成完整的 call graph, 但是生成的都是文本格式
的调用树. cscope 的本意也是构建一个 index, 能够快速从一个函数找到它的 caller
或者 callee.

对于一个 project 完整的结构的图, 一般称作 dependency graph. 节点为模块, 类等
较大的结构, 关系为依赖关系, 或者说是 def-use 关系, 调用关系是其中的一种.
这样的工具一般还可以检测到不当的软件结构比如循环依赖.


###  Egypt

Egypt takes advantage of GCC's capability to dump an intermediate representation of the program being compiled into a file (a RTL file); this file is much easier to extract information from than a C source file. Egypt extracts information about function calls from the RTL file and massages it into the format used by Graphviz.

用 gcc 分析代码, graphviz 生成图.  100 行 perl 代码. 不支持 函数指针分析.
实际上做的就是从 RTL 文件里面找 `call` 和 `symbol_ref` 这两指令.

不支持跨文件. 因为 gcc 生成的 RTL 是单文件的. gcc 不会链接 RTL. 链接是 ld 的事(吧).

### ncc

ncc is a compiler that produces program analysis information. ncc is a decent replacement of **cflow** and **cscope** able to analyse any program using the gcc compiler. The program also includes a graphical call-graph navigator and source browser which is extremely practical for hacking and comprehending large projects.

支持函数指针, see `doc/fptrs.c` and `doc/frag.c`. 支持多文件, 大项目.

The default output (at stdout) is that for every function defined in hello.c
the following things are reported :

    - function calls
    - pointer to function assigned values
    - use of global variables
    - use of members of structures

### KcacheGrind

The profiling tool **Callgrind** and the profile data visualization **KCachegrind**

Callgrind uses runtime instrumentation via the Valgrind framework for its cache simulation and call-graph generation. This way, even shared libraries and dynamically opened plugins can be profiled. The data files generated by Callgrind can be loaded into KCachegrind for browsing the performance results.

Callgrind 是 Valgrind 的一部分. Callgrind
定义了 Callgrind Format Specification. KCachegrind 将这种文件可视化.

[Callgrind Format Specification](http://valgrind.org/docs/manual/cl-format.html)

### Graphviz

note: 在图较大的时候, 使用  `neato` 比 `dot` 好. graphviz 提供了 C API `<gvc.h>`.

没有找到 call graph 功能.

### CodeViz

Perl 脚本, 大约几千行代码. C/C++ 支持.

`genfull` 脚本生成 project 完整的 call graph (dot 格式). 针对call graph 过大的
情况, 提供了 `gengraph` 脚本生成一些函数的生成子图.

有两种工作方式.
1. 依赖打 patch 的 gcc, 或者是使用 ncc (生成 `.nccout` 文件). patch 后的 gcc 会对每个文件生成一个 `.cdepn` 文件, 包括call, line num, macro 等信息.  无法正确处理同名函数.
2. objdump 处理二进制文件.

### doxygen

You can configure doxygen to extract the code structure from undocumented source files. This is very useful to quickly find your way in large source distributions. Doxygen can also visualize the relations between the various elements by means of include dependency graphs, inheritance diagrams, and collaboration diagrams, which are all generated automatically.

[Graphs and diagrams](http://www.stack.nl/~dimitri/doxygen/manual/diagrams.html)

- class hierarchy
- inheritance graph
- include dependency graph
- inverse include dependency graph
- A graph is drawn for each documented class and struct that shows the inheritance relations with base classes and the usage relations with other structs and classes
- a graphical **call graph** is drawn for each function showing the functions that the function directly or indirectly calls
-  a graphical **caller graph** is drawn for each function showing the functions that the function is directly or indirectly called by.

这个 call graph 是局部的, 以一个 function 为起点的图. 应该是因为整个项目的 call graph 可能会非常大. doxygen 使用 dot 生成图片, 太大的图片无法在浏览器中显示.

Note : The completeness (and correctness) of the call graph depends on the doxygen code parser which is not perfect.

### Understand

Understand 是一个类似于 IDE 的工具, 集成了多分析工具, 帮助快速理解代码.
可以生成CFG, Dependency Graph, 等多种图. 支持 C/C++ 和 函数指针分析.

[Understand, Visualize Your Code](https://scitools.com/feature-category/graphing/)

功能介绍页面 [graphing](https://scitools.com/feature-category/graphing/) 里面
没有提到 call graph.

[Cluster Call Graphs](https://scitools.com/clustered-call-graphs/), 给了一些call graph 的例子.
The interactive Cluster Call Graphs show the function call graph, organized by file. There are several variants of this graph: Call, Call-by, Butterfly and Internal Call. They can also be accessed from the function, class, file or architecture level. These graphs can all be accessed from the Graphical View right click menu for the entity.

### tceetree/CScope

[tceetree](https://sourceforge.net/p/tceetree/wiki/Home/). The purpose of the
project is generating a function call tree for a software application written
in C. This utility takes as input an uncompressed
[CScope](http://cscope.sourceforge.net/) output file. With a few options, an
output DOT language file can be generated.

A fuzzy parser used by cscope supports C, but is flexible enough to be useful for C++ and Java.
cscope outputs a function's caller and callee. tceetree 完全就是读取这个 output
然后输出 dot.

cscope 的代码看起来只是单纯的字符串匹配.

ref:
 [Bug 1508830 - cscope fails to find functions with function pointer formal parameters](https://bugzilla.redhat.com/show_bug.cgi?id=1508830)

### DMS Software Reengineering Toolkit

DMS: Generalized Compiler Infrastructure
A very simple model (see Figure below) of DMS is that of an extremely generalized compiler, having

- a parser (producing compiler-like data structures capturing code),
- a set of semantic analyzers,
including a variety of pattern matching (using surface syntax) engines
- a set of compiler data structure modification engines,
including a source-to-source program transformation engine (using surface syntax)
- and final output formatting components (converting compiler data structures back to valid source code rather than binary code),

[Call Graphs](http://www.semanticdesigns.com/Products/DMS/FlowAnalysis.html)

The call graph data structure is first constructed, and then a rendering step using DOT is use to display arbitrary sub graphs of the full graph to ensure that the rendered graph is of reasonable size for people to grasp. The rendered subgraph is parameterized by a root function name, call width, call depth, and a set of "stop" function names. Call depth controls how deep a chain of calls may go in the rendered graph. Call width controls how many callees are rendered for a particular function node when there are too many indirect call targets. The stop functions terminate the call graph if encountered, and are usually chosen because those functions do not contain details of interest.


### Smatch

[Smatch](http://smatch.sourceforge.net/)

### cflow

[document](http://www.gnu.org/software/cflow/manual/cflow.html)

GNU cflow analyzes a collection of C source files and prints a graph, charting control flow within the program.

The program is able to produce two kind of graphs: direct and reverse. Direct graph begins with the main function (main), and displays recursively all functions called by it. In contrast, reverse graph is a set of subgraphs, charting for each function its callers, in the reverse order. Due to their tree-like appearance, graphs can also be called trees.

产生的结果是 tree 一样的形式, 可以使用 `cflow2vcg` 转换成图片.
这里也体现了一个设计上的问题: 表达 Call Graph 是应该用树还是用图(合并相同节点)
来表达.

lex/yacc 写的 C Parser. 没有指针分析功能.

### CppDepends

Cpp 代码分析的工具.

[文档 Dependency Graph1](https://www.cppdepend.com/Doc_VS_Arch) 介绍了其支持生成的各种图.

包含多个层次, projects 的依赖, namespaces 的依赖图,  一个namespace 中 types 的依赖, type 里面 methods and fields 之间的依赖. 双击高层次图中的节点即可查看下一个层次的图. 使用节点大小表示代码量大小, 边的粗度表示依赖的强弱(可从多种指标中选择).

除了使用图来表示, 还可以以邻接矩阵的形式展现, 更适合展示复杂的图.

产生的 CallGraph 以一个函数为中心, 向 caller, callee 方向查询指定深度. 类似 doxygen.

[博客 A picture is worth a thousand words: Visualize your C/C++ Projects](http://cppdepend.com/blog/?p=27) 介绍了所有可视化功能.

### Code Graph (Visual Studio plugin)


## Papers

- [KernelGraph: Understanding the kernel in a graph](http://journals.sagepub.com/doi/pdf/10.1177/1473871617743239)

- [Static Call Graph Generator for C++ using Debugging Information](https://ieeexplore.ieee.org/document/4425846/)

