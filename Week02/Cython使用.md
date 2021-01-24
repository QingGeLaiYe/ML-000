## Cython语法

Cython的开发是为了使编写Python的C扩展变得更容易，并允许将现有的Python代码转换为C。此外，Cython允许将优化的代码与Python应用程序一起提供，而无需外部依赖。

### 变量类型

一些变量类型在用Cython使用的是Python的类型相呼应，如 `int`，`float`和`long`。其他Cython变量类型也可以在C中找到，如`char`或`struct`，如声明`unsigned long`。

### 在`cdef`和`cpdef`函数类型

该`cdef`关键字指示使用用Cython或C型。它也用于定义函数，就像在Python中一样。

使用Python`def`关键字用Cython编写的函数对其他Python代码可见，但是会导致性能下降。**使用该`cdef`关键字的函数仅对其他Cython或C代码可见，但执行速度更快**。如果具有只能从Cython模块内部调用的函数，使用`cdef`。

`cpdef`与Python代码和C代码兼容，从而使C代码可以全速访问声明的函数。不过，这种便利需要付出一定的代价：与相比， **`cpdef`函数产生更多的代码，并且调用开销略多`cdef`**。

### 其他Cython关键字

Cython中的其他关键字可以控制程序流程和行为的某些方面，而Python中则没有这些方面：

- `gil`**和`nogil`。**这些是上下文管理器，用于描述需要（`with gil:`）或不需要（`with nogil:`）Python的全局解释器锁（GIL ）的代码段。无需调用Python API的C代码可以在一个`nogil`块中更快地运行，尤其是当它执行长时间运行的操作（例如从网络连接读取数据）时。
- **`cimport`。** 这将指导Cython导入C数据类型，函数，变量和扩展类型。例如，使用NumPy中C模块的Cython应用程序用于`cimport`访问这些功能。
- `**include**`**。**这将一个Cython文件的源代码放在另一个C中，与C中的方式大致相同。
- **`ctypedef`。**用于引用外部C头文件中的类型定义。
- **`extern`。**与`cdef`一起使用，以指代其他模块中的C函数或变量。
- **`public/api`。**用于在Cython模块中进行声明，其他C代码可以看到这些声明。
- **`inline`。**用来表示给定的函数应该内联，或者为了快速起见，在每次使用时将其代码放置在调用函数的主体中。例如，`f`上面的代码示例中的函数可以修饰`inline`以减少其函数调用开销，因为它仅在一个地方使用。（请注意，C编译器可能会自动执行其自身的内联，但`inline`可以让显式指定是否应内联某些东西。）

Cython代码倾向于以增量方式编写-首先编写有效的Python代码，然后添加Cython装饰以加快速度。因此，可以根据需要选择Cython的扩展关键字语法。

## 编译Cython

要构建一个有效的Cython程序，我们需要三件事：

1. Python解释器。如果可以，请使用最新的发行版本。
2. Cython程序包。通过`pip`包管理器将Cython添加到Python ：`pip install cython`
3. AC编译器。

## 如何使用Cython

为了获得最佳结果，请使用Cython优化以下Python函数：

1. 在紧密的循环中运行的功能，或者在单个“热点”中需要大量处理时间的功能。
2. 执行数字操作的函数。
3. 与可以用纯C表示的对象一起工作的函数，例如基本数字类型，数组或结构，而不是诸如列表，字典或元组之类的Python对象类型。

传统上，Python在循环和数字操作方面的效率低于其他未解释的语言。装饰代码的次数越多，表明它应使用可以转换为C的基数值类型，则数字处理的速度就越快。

在Cython中使用Python对象类型本身并不是问题。使用Python对象的Cython函数仍会编译，并且在性能不是首要考虑因素的情况下，Python对象可能更可取。但是任何使用Python对象的代码都将受到Python运行时性能的限制，因为Cython会生成代码来直接处理Python的API和ABI。

Cython优化的另一个值得关注的目标是直接与C库交互的Python代码。可以跳过Python“包装器”代码并直接与库连接。

然而，用Cython也不会自动生成这些库中的正确调用接口。将需要Cython通过声明引用库头文件中的函数签名`cdef extern from`。请注意，如果没有头文件，Cython会很宽容地允许声明近似于原始头的外部函数签名。但是为了安全起见，请尽可能使用原件。

Cython可以立即使用的一个外部C库是NumPy。要利用Cython对NumPy数组的快速访问，请使用`cimport numpy`（可选用with`as np`保持其命名空间不同），然后使用`cdef`语句声明NumPy变量，例如`cdef np.array`或`np.ndarray`。

## Cython分析

改善应用程序性能的第一步是对其进行概要分析-生成有关执行期间花费时间的详细报告。Python提供了用于生成代码配置文件的内置机制。Cython不仅参与了这些机制，而且拥有自己的分析工具。

Python自己的探查器会`cProfile`生成报告，以显示在给定的Python程序中哪些功能占用的时间最多。默认情况下，Cython代码不会显示在那些报告中，但是您可以通过在文件顶部插入要包含在性能分析中的函数的编译器指令来对Cython代码]进行`.pyx`性能分析：

```
＃cython：profile = True
```

在Cython生成的C代码上启用逐行跟踪，但这会带来很多开销，因此默认情况下处于关闭状态。

请注意，性能分析会影响性能，因此请确保针对要交付生产的代码关闭性能分析。

Cython还可以生成代码报告，以指示将给定`.pyx`文件中的多少转换为C，以及剩余的Python代码。要查看实际效果，请`setup.py`在我们的示例中编辑文件，并在顶部添加以下两行：

```
import Cython.Compiler.Options
Cython.Compiler.Options.annotate = True
```

删除`.c`项目中生成的文件，然后重新运行`setup.py`脚本以重新编译所有内容。完成后，您应该在共享.pyx文件名称的同一目录中看到一个HTML文件，在本例中为 `num.html`。打开HTML文件，您将看到仍然依赖于Python的代码部分以黄色突出显示。您可以单击黄色区域以查看Cython生成的基础C代码。

