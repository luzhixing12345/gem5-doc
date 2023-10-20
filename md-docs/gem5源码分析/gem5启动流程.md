
# gem5启动流程

整个模拟器是由 C++ 与 python 混合编程, 这里期望读者具有 Python C API 的使用经验, 也要求读者具备一定的现代 C++ 语法阅读能力

**开始本节阅读之前, 请务必阅读 pybind11 部分并了解其如何实现 python 与 C++ 之前的模块链接**

gem5 程序运行的主入口是位于 `src/sim/main.cc` 的 main 函数, 其实现如下所示

```cpp
int main(int argc, char **argv) {
    // 初始化一些变了
    initSignals();

    // Convert argv[0] to a wchar_t string, using python's locale and cleanup
    // functions.
    std::unique_ptr<wchar_t[], decltype(&PyMem_RawFree)> program(
        Py_DecodeLocale(argv[0], nullptr),
        &PyMem_RawFree);

    // This can help python find libraries at run time relative to this binary.
    // It's probably not necessary, but is mostly harmless and might be useful.
    Py_SetProgramName(program.get());

    py::scoped_interpreter guard(true, argc, argv);

    auto importer = py::module_::import("importer");
    importer.attr("install")();

    try {
        py::module_::import("m5").attr("main")();
    } catch (py::error_already_set &e) {
        if (e.matches(PyExc_SystemExit))
            return e.value().attr("code").cast<int>();

        std::cerr << e.what();
        return 1;
    }

    return 0;
}
```

Py_DecodeLocale是Python提供的一个函数,用于将字符串转换为本地编码(Locale encoding)下的宽字符形式.本地编码是指当前操作系统的默认字符编码,例如在Windows系统上,它通常是GBK或者UTF-8(取决于系统设置).该函数的函数原型如下

```c
wchar_t* Py_DecodeLocale(const char *arg, size_t *size);
```

> 其中 arg 是要转换的字符串,size是指向输出大小的指针.函数返回一个指向宽字符字符串的指针.如果转换失败,则返回NULL,并将errno设置为相应的错误代码.

该函数通常用于将命令行参数或环境变量等本地编码下的字符串转换为Python程序中使用的Unicode字符串.在Python 3中,字符串默认为Unicode编码.因此,**在与Python C API一起使用时,需要将本地编码的字符串转换为Unicode字符串.**