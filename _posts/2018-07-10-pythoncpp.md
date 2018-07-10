---
layout: post
title: C/C++ 调用Python
categories: 工程
description: C/C++ 调用Python
keywords: Python
---

## 目录
- 前言
- 官方文档
- 环境搭建
- 编译链接
- Demo
- 解释器
- 初始化
- GIL
- Object
- 一切皆对象
- 从Python代码中获取Object
- C/C++与Object转换
- 函数调用
- 引用计数
- 参考资料

## 前言
最近项目中遇到需要用C++调用python代码的情况，在网上搜索后发现中文资料比较少。因此借此机会一边学习一边整理成文档，方便后续查阅。

## 官方文档
教程：https://docs.python.org/2/extending/embedding.html
API：https://docs.python.org/2/c-api/index.html

## 环境搭建
### 编译链接
使用python提供的C/C++接口，需要包含python安装目录下的头文件Python.h
编译、链接时需要指定头文件、python库的地址，不过不需要我们自己操心，python提供了一个脚本，可以自动推荐编译、链接参数：
Bash
python-config --cflags
python-config --ldflags

### Demo
动过运行一个简单的demo，可以验证链路是否打通。
C++
```
#include <Python.h>
int main(int argc, char *argv[]) {
  Py_Initialize();
  PyRun_SimpleString("print('hello world')\n");
  Py_Finalize();
  return 0;
}
// g++ main.cpp -I$PYTHON_PATH/include/python2.7 -lpython2.7
// 输出 hello world
```

## 解释器
### 初始化
在调用python API时，首先需要初始化全局解释器，并且在使用完后销毁。在我们的业务场景下，需要解释器常驻内存，因此Py_Initialize在系统初始化时调用，Py_Finalize在析构函数中调用。

C++
```
void Py_Initialize(void)
int Py_IsInitialized(void)
void Py_Finalize()
```

初始化Python后，可以通过int PyRun_SimpleString(const char *command)函数令解释器执行任意python代码。这种叫做高层接口。高层接口虽然方便，但很难与C/C++交换数据。所以对于复杂需求，应该使用低层接口。虽然需要多写很多C代码，但可以灵活的实现很多复杂功能。

从操作步骤上看，C++调用Python低层接口可以分为几个阶段
- 初始化Python解释器
- 从C++到Python转换数据
- 用转换后的数据做参数调用Python函数
- 把函数返回值转换为C++数据结构

### GIL
在使用python解释器时，要注意GIL（全局解释锁）的工作原理以及对性能的影响。GIL保证在任意时刻只有一个线程在解释器中运行。在多线程环境中，python解释器工作原理如下：
Plain Text
```
1. 设置GIL
2. 切换到一个线程去运行
3. 运行：
    a. 指定数量的字节码指令，或者
    b. 线程主动让出控制（可以调用time.sleep(0)）
4. 把线程设置为睡眠状态
5. 解锁GIL
6. 再次重复以上所有步骤 
```

对性能的影响：
假如有一段两个线程的python代码，运行在一个两核CPU上。由于GIL的存在，两个线程无法真正并行执行，CPU占用率总是低于50%。

GIL是一个历史遗留问题，导致CPython多线程不能利用多个CPU内核的计算能力。为了利用多核，通常使用多进程的方法，或是通过Python调用C代码，由C来实现多线程。

注意，当在C/C++创建的线程中调用Python时，GIL需要通过函数PyGILState_Ensure()和PyGILState_Release(）手动获取、释放。
C++
```
PyGILState_STATE gstate;
gstate = PyGILState_Ensure();

/* Perform Python actions here. */
result = CallSomeFunction();
/* evaluate result or handle exception */

/* Release the thread. No Python API allowed beyond this point. */
PyGILState_Release(gstate);
```

## Object
###一切皆对象
在python中有一句话叫做“一切皆对象”，这句话可以结合源码更好的进行理解。在python里，一切变量、函数、类等，在解释器中执行时，都会在在堆中新建一个对象，并将名字绑定在对象上。
Python
```
i = 1              -----新建一个PyIntObject对象,然后绑定到i上
s = "abcde"        -----新建一个PyStringObject对象，绑定到s上
def foo(): pass    -----新建一个PyFunctionObject对象， 绑定到foo上
class C(object): pass    -----新建一个类对象，绑定到C上
instance = C()           -----新建一个实例对象，绑定到instance上
l = [1,2]                -----新建一个PyListObject对象，绑定到l上
t = (1,2)                -----新建一个PyTupleObject对象，绑定到t上
```

在Python/C API中，使用指向堆中对象的指针PyObject*对这些对象进行进行管理。因此，python中的大多数语句，都可以通过对PyObject指针调用各种函数来实现。

### 从Python代码中获取Object
如上一节所述，既然一切皆对象，那我们就可以在C/C++中获取到python代码中的对象。

C++
```
// Python内建函数import，导入一个Python模块。
PyObject* PyImport_ImportModule(char *name) 

// Python语句o.attr_name，返回对象o中检索attr_name属性或方法。
PyObject* PyObject_GetAttrString(PyObject *o, char*attr_name) 
```

### C/C++与Object转换
可以通过调用Py_BuildValue，通过传递格式字符串和变长参数，将C/C++变量构造为变量或元组。
C
```
PyObject* Py_BuildValue(const char *format, ...)
// 更多参数查阅参考官方文档
// s 将C字符串转换为Python字符串对象。
// i 将C int转换为Python整数对象。
// d 将C double转换为Python浮点数。
```

此外也可以直接调用下面一系列函数，显式将C/C++变量转换为python变量。
C++
```
// 基本变量
PyObject* PyLong_FromLong(long v)
PyObject* PyBool_FromLong(long v)
PyObject* PyFloat_FromDouble(double v)

// python元组
PyObject* PyTuple_New(Py_ssize_t len)
int PyTuple_SetItem(PyObject *p, Py_ssize_t pos, PyObject *o)
```
使用例
C++
```
// 通过set item的方式构造tuple
PyObject* args = PyTuple_New(3);
PyObject* arg1 = Py_BuildValue("i", 100); // 整数参数
PyObject* arg2 = Py_BuildValue("f", 3.14); // 浮点数参数
PyObject* arg3 = Py_BuildValue("s", "hello"); // 字符串参数
PyTuple_SetItem(args, 0, arg1);
PyTuple_SetItem(args, 1, arg2);
PyTuple_SetItem(args, 2, arg3);

// 通过buildvalue直接构造tuple
PyObject* args = Py_BuildValue("(ifs)", 100, 3.14, "hello");
PyObject* args = Py_BuildValue("()"); // 无参函数
```

PyObject也可以转换为C++变量
C++
```
// 使用一系列库函数转换基本变量
long PyLong_AsLong(PyObject *obj)
long PyInt_AsLong(PyObject *obj)
double PyFloat_AsDouble(PyObject *obj)
string PyString_AsString(PyObject *obj)

// 元组
Py_ssize_t PyTuple_Size(PyObject *p)
PyObject* PyTuple_GetItem(PyObject *p, Py_ssize_t pos)
```

### 函数调用
由于Python中一切皆对象，因此函数、方法等调用都可以通过PyObject_Call系列函数完成。
C++
```
// callable(*args, **kwargs)
PyObject* PyObject_Call(PyObject *callable, PyObject *args, PyObject *kwargs)
PyObject* PyObject_CallObject(PyObject *callable, PyObject *args)

// callable(arg1, arg2, ...)
PyObject* PyObject_CallFunction(PyObject *callable, const char *format, ...)
PyObject* PyObject_CallFunctionObjArgs(PyObject *callable, ..., NULL)

// obj.name(arg1, arg2, ...)
PyObject* PyObject_CallMethod(PyObject *obj, const char *name, const char *format, ...)
PyObject* PyObject_CallMethodObjArgs(PyObject *obj, PyObject *name, ..., NULL)
```

可以发现，函数参数通常有两种类型，一种是直接传递tuple、map两类PyObject，另一种是通过格式字符串与变长参数，直接将C/C++变量解析成参数。

还用一种结合两种参数的方法，通过PyArg_ParseTuple()、PyArg_ParseTupleAndKeywords()和PyArg_Parse()三个函数，可以用格式字符串将C/C++变量构造为Python元组、字典或变量，以便后续函数调用。

## 引用计数
### 内存管理
Python使用引用计数与垃圾回收来管理内存。对于每个对象，可以理解为有一个对象实体以及若干对该实体的引用（指向对象的指针）。引用计数通过记录对象被引用的次数来管理对象，增加对对象的引用会使引用计数加一，减少对对象的引用使引用计数减一，当一个对象引用计数为0时释放该对象占用的内存。由于引用计数无法处理循环引用的情况，还会有垃圾回收机制来处理循环引用的对象。可以认为Python解释器会周期的调用垃圾回收。
所有的python对象，PyObject都有对象类型和引用计数。对象类型确定它是什么样的对象 (如，整数、列表或用户定义的函数)。对于每个已知类型，都有一个宏来检查对象是否属于该类型。例如，如果指向的对象是 Python 列表, 则 PyList_Check (a) 为 true。

### Python/C API 中的引用计数
API中使用两个宏Py_INCREF(x) 和 Py_DECREF(x)来处理引用计数的增加和减少。当计数达到零时，Py_DECREF() 会释放对象。如果是列表等复合对象类型，Py_DECREF还会递减对象中包含的其他对象的引用计数。如果忘记减少引用计数将会造成内存泄漏。
有一个概念叫做引用的所有权，拥有所有权代表该引用不使用时需要调用Py_DECREF。所有权可以被传递，获得所有权的新引用也需要在不使用时调用Py_DECREF。
此外还有一个概念叫做“借用(borrow)”的引用，类似C++中std::weak_ptr。借用的引用不能长期持有对象，没有所有权。对象原持有者释放对象后，借用引用再访问被释放的内存会有风险。借用的引用可以通过调用Py_INCREF()改为真正持有引用。
所有权传递常发生在函数调用：
- 作为返回值：当函数返回一个引用给调用方时，分为两种情况：调用方获得引用所有权或借用了引用而没有获得所有权。
  - 以 PyObject_、PyNumber_、PySequence_ 或 PyMapping_ 开头的函数总是传递所有权，新增它们返回的对象的引用计数，调用方要在使用完后调用Py_DECREF ()。
  - 而PyTuple_GetItem ()、PyList_GetItem ()、PyDict_GetItem () 和 PyDict_GetItemString () 都返回从元组、列表或字典中借用的引用。
- 作为参数：当引用作为参数传递给函数时，有两种情况：函数窃取引用或没有窃取。窃取引用代表调用方不需要再处理该引用。
  - 很少有函数窃取引用，常用的是PyList_SetItem()和PyTuple_SetItem()。
  - 而PyObject_SetItem()、PyDict_SetItem()不窃取引用。

用一个例子来说明所有权传递问题，分别用PyList_GetItem()、 PySequence_GetItem()实现对list中所有数字求和：
C++
```
long
sum_list(PyObject *list)
{
    Py_ssize_t i, n;
    long total = 0, value;
    PyObject *item;

    n = PyList_Size(list);
    if (n < 0)
        return -1; /* Not a list */
    for (i = 0; i < n; i++) {
        item = PyList_GetItem(list, i); /* Can't fail */
        if (!PyLong_Check(item)) continue; /* Skip non-integers */
        value = PyLong_AsLong(item);
        if (value == -1 && PyErr_Occurred())
            /* Integer too big to fit in a C long, bail out */
            return -1;
        total += value;
    }
    return total;
}
C++
long
sum_sequence(PyObject *sequence)
{
    Py_ssize_t i, n;
    long total = 0, value;
    PyObject *item;
    n = PySequence_Length(sequence);
    if (n < 0)
        return -1; /* Has no length */
    for (i = 0; i < n; i++) {
        item = PySequence_GetItem(sequence, i);
        if (item == NULL)
            return -1; /* Not a sequence, or other failure */
        if (PyLong_Check(item)) {
            value = PyLong_AsLong(item);
            Py_DECREF(item);
            if (value == -1 && PyErr_Occurred())
                /* Integer too big to fit in a C long, bail out */
                return -1;
            total += value;
        }
        else {
            Py_DECREF(item); /* Discard reference ownership */
        }
    }
    return total;
}
```

## 参考资料
- 嵌套python解释器
- 浅析 C++ 调用 Python 模块
- C/C++与python互相调用
- python 一切皆对象
- 深入理解 GIL：如何写出高性能及线程安全的 Python 代码
- Python高级特性：全局解释器锁GIL基本概念
- python引用计数和gc垃圾回收

