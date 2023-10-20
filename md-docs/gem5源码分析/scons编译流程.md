
# scons编译流程

解决所有基础环境之后我们先来介绍一下 gem5 的编译过程, 即 scons 的编译流程

[SCons](https://github.com/SCons/scons): a Software Construction Tool, 是一个开源的软件构建工具, 意在为 C、C++ 和 FORTRAN 内置的可靠、自动的依赖性分析。不再需要“make depend”或“make clean”来获取所有依赖项。通过用户定义的其他语言或文件类型的依赖扫描器，可以轻松扩展依赖分析

值得一提的是 SCons 是向下兼容 Python 语法的, 你可以查看根目录下的 SConstruct Sconsopts 这两个文件, 不难发现其语法和运行都是依赖python解释器工作的

![20230331234936](https://raw.githubusercontent.com/learner-lu/picbed/master/20230331234936.png)

当然你也可能会敏锐且惊讶的发现文件并没有全部高亮和提示, 仍有不少部分显示来自 Python 的报错信息. 这是因为这并不完全是 Python的语法, 也有一些来自 SCons 的内置命令(内置关键字) 并没有成功被 Python 解析. 目前并没有可用的 Vscode 扩展足以解决这个问题, 且这是一个[历史遗留的issue](https://github.com/SCons/scons/issues/3210), 且 SCons 团队也曾经试图向 Vscode 官方[反映过这个问题](https://github.com/microsoft/vscode/issues/105623)

当然你可以使用如下引入来引入部分智能提示, 但仍有部分 GlobalDefaultEnvironmentFunctions 之类指令存在语法报错

```python
from SCons.Script import *
```

除此之外你可能会发现找不到 gem5_scons 模块, 这需要你为 Pylance 扩展手动添加一个选项指明路径

![20230401001534](https://raw.githubusercontent.com/learner-lu/picbed/master/20230401001534.png)

Anyway, 我们至少可以开始阅读了

不过需要注意的是, 并不建议直接阅读 SConstruct 这种构建的源代码, 因为其中涉及很多细节和 SCons 自带的语法以及运行原理, 深入研究 SCons 并不是当务之急, 将其作为一个简单的工具使用即可, 我们只需要从宏观上去了解 gem5 系统是如何构建/运行的即可

## gem5 目录结构

基于gem5的研究大多需要更改源代码重新编译，因此确认系统环境没问题后，每一个项目内最好有各自对应的调整编译好的gem5，同时，官方教程给的代码与命令行有很多相对路径是固定的，因此最好保持和教程一致的文件结构，并且在相应位置编译gem5

我们先来看一下 gem5 的目录结构, 删除了部分无关文件/文件夹之后的根核心目录结构如下所示

```bash
├── build         构建得到的二进制文件
├── build_opts    构建可选参数
├── build_tools   构建时工具
├── configs       [一些模拟配置脚本]
├── ext           构建所需的依赖库
├── include       一些可选宏?
├── m5out         [使用gem5模拟系统后的输出结果]
├── SConsopts     SCons构建选项
├── SConstruct    SCons构建入口
├── site_scons    gem5修改的SCons构建依赖
├── src           [源代码主目录]
├── system        模拟系统的固件或引导程序
├── tests         测试相关
└── util          实用脚本程序
```

> [目录结构](http://old.gem5.org/Source_Code.html)

其中比较重要的部分已经使用 `[]` 括起来了, 分别是 `configs` `src` 这两部分是后续修改的核心位置, 我们会在后文分三部分详细描述如何

- 在 `config` 中编写py脚本以完成相应任务
- 在 `src` 中修改源代码并构建自己的系统
- 分析 `m5out` 输出结果的具体内容, 这里保存着使用 gem5 模拟之后的所有结果

---

当我们使用 scons 构建的时候

```bash
scons build/X86/gem5.opt -j4
```

其中第一个参数 `build/X86/gem5.opt` 指明了三部分内容, 编译结果保存在 build/ 目录下, 编译可选的指令集及架构如下

```bash
# build_opts/

├── ALL
├── ARM
├── ARM_MESI_Three_Level
├── ARM_MESI_Three_Level_HTM
├── ARM_MOESI_hammer
├── Garnet_standalone
├── GCN3_X86
├── MIPS
├── NULL
├── NULL_MESI_Two_Level
├── NULL_MOESI_CMP_directory
├── NULL_MOESI_CMP_token
├── NULL_MOESI_hammer
├── POWER
├── RISCV
├── SPARC
├── VEGA_X86
├── X86
├── X86_MESI_Two_Level
├── X86_MI_example
└── X86_MOESI_AMD_Base
```

> 我们可以在文件中看到一些key=value的参数, 比如 `PROTOCOL` 等

最后一个参数决定以什么程度编译 gem5, 可选项如下

- `gem5.debug` 关闭了优化。保证变量不会被优化掉，功能不会被意料外的内联（inlined），以及控制流行为正常。该版本与gdb类的工具合作良好，然而关闭优化会造成该版本明显慢于其它版本。当使用gdb或valgrind等工具并且不希望任何细节被模糊掉时应该选择该版本，否则建议选择其它版本。
- `gem5.opt` 打开优化的同时保留了部分调试功能（例如，断言和DPRINTFs）。该版本良好地平衡了模拟速度与调试观察，是所有环境中最优的版本。
- `gem5.fast` 打开优化并关闭调试部分。最优的速度，代价是不能进行运行时错误检查与调试输出。如果确信所有功能可以正确运行并想要获得峰值性能，建议使用该版本。
- `gem5.prof` 类似于gem5.fast，但仍然保留了一些功能（instrumentation）可以用于gprof分析工具。该版本不常用，但可以用于找出gem5中应当被注意的部分以提升性能。
- `gem5.perf` 同gem5.prof，但是instrumentation使用google perftools，允许被google-pprof分析。该分析版本是gem5.prof的补充，可能可以在所有基于Linux的系统中替换gem5.prof。

> 简而言之, 编译选opt, 确定没问题了选fast

与此同时 SConstruct 中也提供了一些 options 的选项, 不过这大多数都是控制输出等等, 并不是我们关心的问题

接着 scons 做了一些诸如版本/缓存/字节校验/python模块的工作, 之后根据 site_sons 中 gem5 特定的一些函数引入之类的工作, 确定没有问题了递归搜索所有目录, 找到其中子目录下的 SConstruct 做进一步的编译, 我们可以在 src 下的子文件夹中看到 一个一个的 SConstruct, 其中就包含了当前文件夹内部的编译依赖关系的. 总而言之经过 SCons 内部的依赖处理和检查, 开始编译了

随后将众多.o链接为 gem5.opt 可执行文件, 完成了整个系统的构建

## 具体的编译流程与细节

细心的朋友可能会发现, 在 build/X86 下除了编译得到的 gem5.opt 文件还有 gem5py gem5py_m5 两个可执行文件, 我们找到 gem5py 对应的源码位置 src/python/gem5py.cc

```cpp
int main(int argc, const char **argv) {
    py::scoped_interpreter guard;
    // Embedded python doesn't set up sys.argv, so we'll do that ourselves.
    py::list py_argv;
    auto sys = py::module::import("sys");
    if (py::hasattr(sys, "argv")) {
        // sys.argv already exists, so grab that.
        py_argv = sys.attr("argv");
    } else {
        // sys.argv doesn't exist, so create it.
        sys.add_object("argv", py_argv);
    }
    // Clear out argv just in case it has something in it.
    py_argv.attr("clear")();
    if (argc < 2) {
        std::cerr << "Usage: gem5py SCRIPT [arg] ..." << std::endl;
        std::exit(1);
    }
    // Fill it with our argvs.
    for (int i = 1; i < argc; i++) py_argv.append(argv[i]);
    // Actually call the script.
    py::eval_file(argv[1]);
    return 0;
}
```

大致浏览一下发现并不难理解, 就是简单的利用 pybind11 做了一个 python 解释器的封装, 调用此程序与直接调用 python 没什么太大区别, 我们可以创建一个简单的 python 文件做一次验证

```python
import sys
print(sys.argv)
```

```bash
kamilu@kamilu:~/gem5-cxl$ ./build/X86/gem5py a.py 123 456
['a.py', '123', '456']
```

我们可以在 scons 编译的一开头看到此文件的编译输出

```bash
 [     CXX] X86/python/gem5py.cc -> .pyo
 [     CXX] X86/python/embedded.cc -> .pyo
 [     CXX] X86/python/importer.cc -> .pyo
 [VER TAGS]  -> X86/sim/tags.cc
 [     CXX] X86/sim/init.cc -> .o
 [     CXX] X86/sim/main.cc -> .o
 [     CXX] X86/python/embedded.cc -> .o
 [     CXX] X86/python/importer.cc -> .o
 [    LINK]  -> X86/gem5py
```

如果我们稍加修改 gem5py.cc 让其在编译时输出 argv 的相关信息, 可以看到具体的编译流程, 诸如此类大量的编译信息

```bash
 [ TRACING]  -> X86/debug/RubyDma.cc
 [ TRACING]  -> X86/debug/DistEthernet.cc
build/X86/gem5py build_tools/debugflagcc.py build/X86/debug/DistEthernet.cc DistEthernet 
 [EMBED PY] X86/python/gem5/utils/multiprocessing/__init__.py -> .cc
 [EMBED PY] X86/dev/serial/Serial.py -> .cc
build/X86/gem5py build_tools/debugflagcc.py build/X86/debug/RubyDma.cc RubyDma 
build/X86/gem5py build_tools/marshal.py build/X86/python/gem5/utils/multiprocessing/__init__.py.cc build/X86/python/gem5/utils/multiprocessing/__init__.py gem5.utils.multiprocessing /home/kamilu/gem5/src/python/gem5/utils/multiprocessing/__init__.py 
build/X86/gem5py build_tools/marshal.py build/X86/dev/serial/Serial.py.cc build/X86/dev/serial/Serial.py m5.objects.Serial /home/kamilu/gem5/src/dev/serial/Serial.py 
 [ TRACING]  -> X86/debug/Event.hh
build/X86/gem5py build_tools/debugflaghh.py build/X86/debug/Event.hh Event  False  
 [ TRACING]  -> X86/debug/MC146818.cc
 [ TRACING]  -> X86/debug/Branch.hh
build/X86/gem5py build_tools/debugflagcc.py build/X86/debug/MC146818.cc MC146818 
 [EMBED PY] X86/python/gem5/components/cachehierarchies/classic/caches/__init__.py -> .cc
build/X86/gem5py build_tools/debugflaghh.py build/X86/debug/Branch.hh Branch  False  
build/X86/gem5py build_tools/marshal.py build/X86/python/gem5/components/cachehierarchies/classic/caches/__init__.py.cc build/X86/python/gem5/components/cachehierarchies/classic/caches/__init__.py gem5.components.cachehierarchies.classic.caches /home/kamilu/gem5/src/python/gem5/components/cachehierarchies/classic/caches/__init__.py 
 [ TRACING]  -> X86/debug/CacheComp.hh
 [EMBED PY] X86/mem/MemCtrl.py -> .cc
 [ TRACING]  -> X86/debug/EthernetCksum.hh
 [ TRACING]  -> X86/debug/MemCtrl.cc
build/X86/gem5py build_tools/debugflaghh.py build/X86/debug/CacheComp.hh CacheComp  False  
build/X86/gem5py build_tools/marshal.py build/X86/mem/MemCtrl.py.cc build/X86/mem/MemCtrl.py m5.objects.MemCtrl /home/kamilu/gem5/src/mem/MemCtrl.py 
build/X86/gem5py build_tools/debugflaghh.py build/X86/debug/EthernetCksum.hh EthernetCksum  False  
build/X86/gem5py build_tools/debugflagcc.py build/X86/debug/MemCtrl.cc MemCtrl 
 [ TRACING]  -> X86/debug/MinorTiming.hh
 [ TRACING]  -> X86/debug/RubyProtocol.cc
build/X86/gem5py build_tools/debugflaghh.py build/X86/debug/MinorTiming.hh MinorTiming Extra timing for instructions False  
build/X86/gem5py build_tools/debugflagcc.py build/X86/debug/RubyProtocol.cc RubyProtocol 
......
```

接着过了一段时间又出现了 `gem5py_m5`, 然后就是关于 gem5py_m5 的构建

```bash
 [ TRACING]  -> X86/debug/TimeSync.hh
 [ TRACING]  -> X86/debug/Fetch.hh
 [ TRACING]  -> X86/debug/KvmIO.cc
build/X86/gem5py build_tools/debugflaghh.py build/X86/debug/TimeSync.hh TimeSync  False  
build/X86/gem5py build_tools/debugflaghh.py build/X86/debug/Fetch.hh Fetch  False  
build/X86/gem5py build_tools/debugflagcc.py build/X86/debug/KvmIO.cc KvmIO 
build/X86/gem5py build_tools/marshal.py build/X86/python/gem5/utils/multiprocessing/_command_line.py.cc build/X86/python/gem5/utils/multiprocessing/_command_line.py gem5.utils.multiprocessing._command_line /home/kamilu/gem5/src/python/gem5/utils/multiprocessing/_command_line.py 
 [    LINK]  -> X86/gem5py_m5
 ...

 [SO Param] m5.objects.ThermalModel, ThermalReference -> X86/python/_m5/param_ThermalReference.cc
 [SO Param] m5.objects.BaseMinorCPU, MinorFUPool -> X86/params/MinorFUPool.hh
build/X86/gem5py_m5 build_tools/sim_object_param_struct_cc.py m5.objects.ThermalModel build/X86/python/_m5/param_ThermalReference.cc True 
build/X86/gem5py_m5 build_tools/sim_object_param_struct_hh.py m5.objects.BaseMinorCPU build/X86/params/MinorFUPool.hh 
 [SO Param] m5.objects.SMBios, X86SMBiosSMBiosTable -> X86/params/X86SMBiosSMBiosTable.hh
 [SO Param] m5.objects.Uart, Uart8250 -> X86/params/Uart8250.hh
build/X86/gem5py_m5 build_tools/sim_object_param_struct_hh.py m5.objects.Uart build/X86/params/Uart8250.hh 
build/X86/gem5py_m5 build_tools/sim_object_param_struct_hh.py m5.objects.SMBios build/X86/params/X86SMBiosSMBiosTable.hh 
 [SO Param] m5.objects.X86SeWorkload, X86EmuLinux -> X86/python/_m5/param_X86EmuLinux.cc
 [SO Param] m5.objects.Tags, CompressedTags -> X86/python/_m5/param_CompressedTags.cc
 [SO Param] m5.objects.TlmBridge, TlmToGem5BridgeBase -> X86/python/_m5/param_TlmToGem5BridgeBase.cc
build/X86/gem5py_m5 build_tools/sim_object_param_struct_cc.py m5.objects.X86SeWorkload build/X86/python/_m5/param_X86EmuLinux.cc True 
build/X86/gem5py_m5 build_tools/sim_object_param_struct_cc.py m5.objects.Tags build/X86/python/_m5/param_CompressedTags.cc True 
build/X86/gem5py_m5 build_tools/sim_object_param_struct_cc.py m5.objects.TlmBridge build/X86/python/_m5/param_TlmToGem5BridgeBase.cc True
```

到此为止不难得出两个结论, 1. `gem5py` `gem5py_m5` 在 py cpp 文件直接做了很多相关的文件转换工作, 2. `build_tools` 这个目录似乎很重要

> 关于 build_tools 这个目录里的代码, 我的评价是**可读性极差, 极其不优雅, 着实有些难绷...**



## 结语

正如笔者开头所说: "并不建议直接阅读 SConstruct 这种构建的源代码, 因为其中涉及很多细节和 SCons 自带的语法以及运行原理, 深入研究 SCons 并不是当务之急, 将其作为一个简单的工具使用即可, 我们只需要从宏观上去了解 gem5 系统是如何构建/运行的即可", 本文也只是简单的介绍了一下 gem5 的根目录结构以及相关选项的含义, 没有深入 SCons 的部分.

我们会在后文对于源代码的修改/添加的时候引入对于 SConstruct 文件的修改, 结合修改实例去做一个更为贴近现实的讲解, 相比于大而泛的空谈 SCons 构建过程, 不如照猫画虎照葫芦画瓢来的更直观
