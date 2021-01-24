# 第二周：

Premature Optimization Is the Root of All Evil 过早的优化是万恶之源



Python优化方法论

优化重点不在于所谓的 python trick 而在于： 

识别代码瓶颈并找到原因 → Profiler 使用 

首先从宏观上寻找修改策略 

采用更接近于底层的语言（Cython 或者 C++）进行操作 

并行：多进程或者多线程



Python优化原则：

不要乱并行!

并发和并行是两个非常容易混淆的概念。 它们都可以表示两个或多个任务一起执行，但是偏重点有点不同。 并发偏重于多个任务**交替执行**，而多个任务之间有可能还是串行的。 并发是逻辑上的同时发生（simultaneous），而并行是**物理上的同时发生**。

优化按照重要性排序：算法本身 > 算法复杂度 > 并行(没有充分利用多线程)

 

寻找hotspots原因的一般规则：

内存读取往往是瓶颈的核心。 所以如果想不到原因，优先找内存操作。 

SIMD 是提速的一个解决方案。



SIMD:**单指令流多数据流**（英语：**Single Instruction Multiple Data**，[缩写](https://zh.wikipedia.org/wiki/縮寫)：**SIMD**）是一种采用一个控制器来控制多个处理器，同时对一组数据（又称“数据向量”）中的每一个分别执行**相同**的操作从而实现空间上的并行性的技术。

现代 CPU 往往可以同时做多个计算（Register 更大）, 但是操作必须得是一样的（比如说都是乘或加）。 

一些编译器会自动进行优化，一些不会。 

不同 CPU 能够并行计算的大小不一样。所以不同服务 器需要重新编译。

 实现这个要求数组排列连续并符合某些性质。 

实现方式： OpenMP 指令（只能用 C/C++). 

需要 directory。 

或者使用 intrinsics（近乎于汇编程度）。



CPU 的内存读取问题：

内存层次：内存 → Cache → Register，从左到右存储量越来越小，但是速度越来越快。 

内存：外存与CPU进行沟通的桥梁，计算机中所有程序的运行都在内存中进行。

Cache：**缓存（cache**），原始意义是指访问速度比一般随机存取存储器（RAM）快的一种高速存储器，通常它不像系统主存那样使用DRAM技术，而使用昂贵但较快速的SRAM技术。

Register：**寄存器**（Register）是中央处理器内用来暂存指令、数据和地址的电脑存储器。寄存器的存贮容量有限，读写速度非常快。在计算机体系结构里，寄存器存储在已知时间点所作计算的中间结果，通过快速地访问数据来加速计算机程序的运行。寄存器位于存储器层次结构的最顶端，也是CPU可以读写的最快的存储器

CPU 只能在 Register 中工作。 

我们无法控制 Cache,但是 Cache 往往是出事的源泉



Cache问题:

不好读进来： 

 一般来说读内存地址都是成批读临近数据进 Cache 的； 但如果不在一块，就非常麻烦 

 得反复读： 

 一般都是读一块数据进来，但如果读错了就得重新清空 cache。 

Cache miss 

更多问题阅读Vtune-Cookbook (2020). Intel® VTune™ Profiler Performance Analysis Cookbook. 

 https://software.intel.com/content/www/us/en/develop/documentation/vtune-cookbook/top.html



Profiler：

Function Profiler 可以采用 cProfile。

cProfile：

需要在终端中运行命令。 

命令为 python –m cProfile myscript.py 

python –m cProfile myscript.py 

输出为： tottime→ 整体运行时间（不包括调用其他函数） 
                cumtime→ 累计运行时间（包含调用其他函数）

缺点：

 缺乏 call-tree（可以通过其他方法补充） 

缺乏每行代码运行时间（存在其他 line profiler） 

 缺乏其他 profiling 信息，尤其是底层信息 

难以处理多语言情况 

没有图形界面 

很多时候时间不准确



 Line Profiler 选择很多，都不太好

安装命令 pip install line_profiler 

在需要 profile 的函数加上装饰器 @profile 

运行 kernprof -l -v myscript.py

 

Vtune 目前只支持 C++，但理论上 python 是一致的。

付费软件，为 Intel Parallel Studio 一部分。 

官方网站值得探究：https://software.intel.com/content/www/us/en/develop/articles/qualify-for-free-software.html

 运行时先要进入对应 vtune 文件夹运行 source ./vtune-vars.sh。



target leakage：在准备数据的时候，或者数据采样的时候出了问题，误将与结果直接相关的feature纳入了数据集。一般target leakage会导致数据在训练集上表现很好，但是当运用到实际上时，表现会很差。



Cython

官网：https://cython.readthedocs.io/en/latest/src/tutorial/cython_tutorial.html

 Python 的 superset：全部python 代码都可以运行， 但是并不是其长项。 

更好的办法是把它想成 C 

 最大的区别：静态类 

一个额外的作用：代码保护



**Cython的编译**

代码案例：

运行命令 python setup.py install。 

注意：Cython 的建议是每个文件是一个 module。所以建议 install 来配置（docker）。 

同理：Cython 互相引用会出很多问题。

用 intel 编译器：https://github.com/IntelPython/examples/tree/master/cython_icc_mkl



Cython 的加速 trick：cython_example.ipynb

Cython 和 C 的连接：connect_c 文件夹



使用 Cython 一些建议 

所有类型均应该有 type。 

注意 numpy 的排列。 

尽量使用 C++ 自带数据结构。 

通用方法：

numpy 使用 view 传递给 C；

C 使用 openmp 或者Eigen。使用 Map 可以使用类似的 matlab 的语法。但是不支持 OpenMP。 

 所有内存分配和传递都应该在 python 中完成。临时变量用 C++ 类进行构建。



Python 并行和 GIL

由于 Python 中 GIL的存在，使得本质上 python 只能用一个进程进行。 这使得并行 python 十分困难。 python 自带多进程库滥到极点。所以建议使用 ray。 

GIL：



 ray：

ray 长处在于可以调用任何 python 函数，并可以共享 数据（使得不用 copy）。 但是共享的数据只读。 

ray 的语法见 ray 文件夹。

race condition竞态，类似map reduce





OpenMP

OpenMP是一套非常复杂的并行计算模块。 

 整体想法：采用注释的方式使非并行的东西变成并行，（不仅是并行（多线程）还有SIMD） 。

最大问题：对 C++ 支持极差，所以不适合 Eigen（主流矩阵计算库）。

如果要用 C++，请用oneTBB 建议的解决方法：

使用 Intel 编译器 及oneTBB的Denpendency Graph+ Eigen::Map。

oneTBB：偏底层，了解即可。

Cython 只做数据预处理。注意：这样做编译过程将会十分痛苦。 

如果要在 cython 中运用 openmp，只可以用 prange， 并且必须满足各种需求。

OpenMP不能调用任何与python有关的东西



Eigen官网;http://eigen.tuxfamily.org/index.php?title=Main_Page

Eigen map例子：注意col与row

为什么用Eigen？

用Eigrn可自动完成SIMD，大量计算矩阵。Eigen只支持C++库（Python不行）



VTune:调到HPC

注：找个学生账号可免费使用



Cython的两种运用

1.Cython当成简洁的C运用

2.Cython作数据预处理

优化要有个限度，防止出问题

一般而言传递给c的都是numpyarray



出现问题三个小时之内自己解决

多线程能不用尽量不要用

能不并行就不要并行

------

# 