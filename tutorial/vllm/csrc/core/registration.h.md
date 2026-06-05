# registration.h — Code Analysis / 代码分析

## Source / 来源
- **File**: `csrc/core/registration.h`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Provide macro helpers for token concatenation, stringification, Torch library macro expansion, and Python extension module registration. / [CN] 提供用于 token 拼接、字符串化、Torch 库宏展开以及 Python 扩展模块注册的宏辅助。

## Line-by-Line Analysis / 逐行分析
### Preprocessor utility macros / 预处理器工具宏
```cpp
#define _CONCAT(A, B) A##B
#define CONCAT(A, B) _CONCAT(A, B)

#define _STRINGIFY(A) #A
#define STRINGIFY(A) _STRINGIFY(A)
```
**EN:** These are standard two-level preprocessor helpers. The outer macro forces expansion before token pasting or stringification, which is necessary when the input itself is another macro rather than a literal token.
**CN:** 这是标准的“两层式”预处理器辅助宏。外层宏会先触发一次展开，然后再执行 token 拼接或字符串化；当输入本身还是另一个宏时，这种写法尤其必要。

### Torch registration macro wrappers / Torch 注册宏包装器
```cpp
// A version of the TORCH_LIBRARY macro that expands the NAME, i.e. so NAME
// could be a macro instead of a literal token.
#define TORCH_LIBRARY_EXPAND(NAME, MODULE) TORCH_LIBRARY(NAME, MODULE)

// A version of the TORCH_LIBRARY_IMPL macro that expands the NAME, i.e. so NAME
// could be a macro instead of a literal token.
#define TORCH_LIBRARY_IMPL_EXPAND(NAME, DEVICE, MODULE)   TORCH_LIBRARY_IMPL(NAME, DEVICE, MODULE)
```
**EN:** PyTorch’s registration macros do not always behave as desired when the library name is itself a macro. These wrappers solve that by inserting an extra expansion layer, making macro-generated names usable with `TORCH_LIBRARY` and `TORCH_LIBRARY_IMPL`.
**CN:** 当库名本身也是一个宏时，PyTorch 的注册宏并不总能按预期工作。这两个包装器通过增加一层展开，确保用宏生成的名字也能正确传给 `TORCH_LIBRARY` 和 `TORCH_LIBRARY_IMPL`。

### Python extension entry point / Python 扩展入口点
```cpp
#define REGISTER_EXTENSION(NAME)                                                 PyMODINIT_FUNC CONCAT(PyInit_, NAME)() {                                         static struct PyModuleDef module = {PyModuleDef_HEAD_INIT,                                                         STRINGIFY(NAME), nullptr, 0, nullptr};     return PyModule_Create(&module);                                             }
```
**EN:** `REGISTER_EXTENSION(NAME)` emits the `PyInit_<NAME>` symbol required by Python’s import machinery. The generated function builds a minimal `PyModuleDef` and returns a module object, which is enough to let the shared library load and trigger its static Torch registration side effects.
**CN:** `REGISTER_EXTENSION(NAME)` 会生成 Python 导入机制要求的 `PyInit_<NAME>` 符号。该函数构造一个最小化的 `PyModuleDef` 并返回模块对象，这足以让共享库被导入，同时触发其中静态的 Torch 注册副作用。

## Key Concepts / 关键概念
- **EN:** Two-level macros are required when you want macro arguments to expand before concatenation or stringification.  
  **CN:** 当你希望宏参数先展开、再参与拼接或字符串化时，就必须使用两层宏。
- **EN:** This header smooths over friction between C preprocessor rules and PyTorch’s macro-based registration APIs.  
  **CN:** 这个头文件的作用，是缓和 C 预处理器规则与 PyTorch 宏式注册 API 之间的配合摩擦。
- **EN:** The generated Python module is intentionally minimal; the real payload is usually static registration side effects elsewhere in the binary.  
  **CN:** 生成的 Python 模块本身刻意保持最小化；真正重要的通常是二进制中其他位置触发的静态注册副作用。
- **EN:** `REGISTER_EXTENSION` is about importability, not about implementing Python-level methods in this file.  
  **CN:** `REGISTER_EXTENSION` 主要解决“可被 import”问题，而不是在这个文件里实现 Python 层方法。

## Dependencies / 依赖关系
- **EN:** `<Python.h>` supplies `PyMODINIT_FUNC`, `PyModuleDef`, and `PyModule_Create`.  
  **CN:** `<Python.h>` 提供 `PyMODINIT_FUNC`、`PyModuleDef` 和 `PyModule_Create`。
- **EN:** The wrappers assume PyTorch registration macros such as `TORCH_LIBRARY` are visible at the include site.  
  **CN:** 这些包装器假定包含此头文件的位置已经能看到 `TORCH_LIBRARY` 之类的 PyTorch 注册宏。
- **EN:** The macro-generated init symbol must match the shared library/module name expected by Python import.  
  **CN:** 宏生成的初始化符号必须与 Python 导入时期望的共享库/模块名一致。
