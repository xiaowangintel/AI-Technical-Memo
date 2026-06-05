# DataLoader.cpp — Code Analysis / 代码分析

## Source / 来源

- **File**: `torch/csrc/DataLoader.cpp`
- **Repository**: `pytorch/pytorch`
- **Purpose**: This file implements logic for `DataLoader.cpp` inside the core torch/csrc bridge layer between Python objects, ATen runtime, and low-level storage/type wrappers, with emphasis on runtime glue. / 该文件在连接 Python 对象、ATen 运行时与底层存储/类型封装的 torch/csrc 核心桥接层中针对 `DataLoader.cpp` 实现逻辑，重点涉及运行时胶水逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-2: Header dependencies / 头文件依赖
```cpp
#include <torch/csrc/DataLoader.h>

```
- **EN**: Pulls in the headers required by this translation unit, including interfaces from the core torch/csrc bridge layer between Python objects, ATen runtime, and low-level storage/type wrappers.
- **CN**: 引入该翻译单元所需的头文件，包括来自连接 Python 对象、ATen 运行时与底层存储/类型封装的 torch/csrc 核心桥接层的接口。

### Lines 3-9: Comments and documentation / 注释与文档
```cpp
// Together with `torch/utils/data/_utils/signal_handling.py`, the following
// is an effort to do our best to provide some error message to users when a
// worker dies due to error / critical signals.
//
// See NOTE [ Signal handling in multiprocessing data loading ] for more
// details.

```
- **EN**: Documents licensing terms, invariants, or developer intent that contextualize the surrounding implementation.
- **CN**: 说明许可证条款、不变量或开发者意图，为周围实现提供上下文。

### Lines 10-13: Comments and documentation / 注释与文档
```cpp
// TODO: The following don't work on Windows. Specifically, sigaction, waitid
// calls, and SIGCHLD handler. Currently, dummy implementations are provided
// for Windows.

```
- **EN**: Documents licensing terms, invariants, or developer intent that contextualize the surrounding implementation.
- **CN**: 说明许可证条款、不变量或开发者意图，为周围实现提供上下文。

### Lines 14-15: Preprocessor configuration / 预处理配置
```cpp
#ifndef _WIN32

```
- **EN**: Applies compile-time gating, macros, or platform-specific configuration so the file builds correctly under different feature sets.
- **CN**: 应用编译期开关、宏或平台特定配置，使该文件能在不同功能组合下正确构建。

### Lines 16-18: Header dependencies / 头文件依赖
```cpp
#include <torch/csrc/Exceptions.h>
#include <torch/csrc/utils/python_numbers.h>

```
- **EN**: Pulls in the headers required by this translation unit, including interfaces from the core torch/csrc bridge layer between Python objects, ATen runtime, and low-level storage/type wrappers.
- **CN**: 引入该翻译单元所需的头文件，包括来自连接 Python 对象、ATen 运行时与底层存储/类型封装的 torch/csrc 核心桥接层的接口。

### Lines 19-21: Header dependencies / 头文件依赖
```cpp
#include <c10/util/irange.h>
#include <fmt/format.h>

```
- **EN**: Pulls in the headers required by this translation unit, including interfaces from the core torch/csrc bridge layer between Python objects, ATen runtime, and low-level storage/type wrappers.
- **CN**: 引入该翻译单元所需的头文件，包括来自连接 Python 对象、ATen 运行时与底层存储/类型封装的 torch/csrc 核心桥接层的接口。

### Lines 22-27: Header dependencies / 头文件依赖
```cpp
#include <sys/wait.h>
#include <csignal>
#include <map>
#include <set>
#include <sstream>

```
- **EN**: Pulls in the headers required by this translation unit, including interfaces from the core torch/csrc bridge layer between Python objects, ATen runtime, and low-level storage/type wrappers.
- **CN**: 引入该翻译单元所需的头文件，包括来自连接 Python 对象、ATen 运行时与底层存储/类型封装的 torch/csrc 核心桥接层的接口。

### Lines 28-29: Using declarations / using 声明
```cpp
using namespace torch;

```
- **EN**: Introduces aliases or selected names to simplify later references inside this file.
- **CN**: 引入别名或选定名称，以简化文件后续代码中的引用。

### Lines 30-50: Supporting statements / 辅助语句
```cpp
// Critical signal handlers should be registered on worker processes before
// doing work.
// The handler will raise default handler so that the kill information will be
// retrieved from main process.
// Python handle is _set_worker_signal_handlers().
#define SIGNAL_HANDLER(SIGNAL, HANDLER_NAME, ERROR_MSG)                    \
  static void HANDLER_NAME(int sig, siginfo_t* info, void* ctx) {          \
    auto _w =                                                              \
        write(STDERR_FILENO, ERROR_MSG, sizeof(ERROR_MSG) / sizeof(char)); \
    (void)_w;                                                              \
    struct sigaction sa{};                                                 \
    sa.sa_handler = SIG_DFL;                                               \
    sa.sa_flags = 0;                                                       \
    if (sigemptyset(&sa.sa_mask) != 0 ||                                   \
        sigaction(SIGNAL, &sa, nullptr) != 0) {                            \
      _exit(EXIT_FAILURE);                                                 \
    } else {                                                               \
      raise(SIGNAL);                                                       \
    }                                                                      \
  }

```
- **EN**: Contains local statements, constants, or helper glue that support the surrounding control flow and data movement.
- **CN**: 包含局部语句、常量或辅助胶水代码，用于支撑周围的控制流与数据传递。

### Lines 51-68: Supporting statements / 辅助语句
```cpp
// signal(2) is really not portable. So use sigaction.
// http://man7.org/linux/man-pages/man2/signal.2.html
static void setSignalHandler(
    int signal,
    void (*handler)(int, siginfo_t*, void*),
    struct sigaction* old_sa_ptr) {
  struct sigaction sa{};
  sa.sa_sigaction = handler;
  sa.sa_flags = SA_RESTART | SA_SIGINFO | SA_NOCLDSTOP | SA_NODEFER;
  if (sigemptyset(&sa.sa_mask) != 0 ||
      sigaction(signal, &sa, old_sa_ptr) != 0) {
    std::ostringstream oss;
    oss << "An error occurred while setting handler for " << strsignal(signal)
        << '.';
    TORCH_CHECK(false, oss.str());
  }
}

```
- **EN**: Contains local statements, constants, or helper glue that support the surrounding control flow and data movement.
- **CN**: 包含局部语句、常量或辅助胶水代码，用于支撑周围的控制流与数据传递。

### Lines 69-82: Supporting statements / 辅助语句
```cpp
SIGNAL_HANDLER(
    SIGBUS,
    handler_SIGBUS,
    "ERROR: Unexpected bus error encountered in worker. "
    "This might be caused by insufficient shared memory (shm).\n")
SIGNAL_HANDLER(
    SIGSEGV,
    handler_SIGSEGV,
    "ERROR: Unexpected segmentation fault encountered in worker.\n")
SIGNAL_HANDLER(
    SIGFPE,
    handler_SIGFPE,
    "ERROR: Unexpected floating-point exception encountered in worker.\n")

```
- **EN**: Contains local statements, constants, or helper glue that support the surrounding control flow and data movement.
- **CN**: 包含局部语句、常量或辅助胶水代码，用于支撑周围的控制流与数据传递。

### Lines 83-104: Supporting statements / 辅助语句
```cpp
// When an error happened in DataLoader methods and Python starts to exit, the
// error trace will keep the loader alive, and Python may kill the children
// processes first before deleting the loader object. Then the cleaning up
// methods in DataLoader.__del__ are not yet called, and SIGCHILD will print an
// error saying a worker is killed by SIGTERM. So we suppress SIGTERM from main
// loader process here to avoid this by _exit(EXIT_SUCCESS). Note that if we
// exit with nonzero code, the loader SIGCHLD handler may report RuntimeError
// again, and then it defeats the whole purpose.
static void handler_SIGTERM(int sig, siginfo_t* info, void* ctx) {
  if (info->si_pid == getppid()) {
    _exit(EXIT_SUCCESS);
  }
  struct sigaction sa{};
  sa.sa_handler = SIG_DFL;
  sa.sa_flags = 0;
  if (sigemptyset(&sa.sa_mask) != 0 || sigaction(SIGTERM, &sa, nullptr) != 0) {
    _exit(EXIT_FAILURE);
  } else {
    raise(SIGTERM);
  }
}

```
- **EN**: Contains local statements, constants, or helper glue that support the surrounding control flow and data movement.
- **CN**: 包含局部语句、常量或辅助胶水代码，用于支撑周围的控制流与数据传递。

### Lines 105-107: Function `NOLINTNEXTLINE` / 函数 `NOLINTNEXTLINE`
```cpp
// NOLINTNEXTLINE(misc-use-internal-linkage)
__attribute__((weak)) void setDataLoaderSignalHandlers() {}

```
- **EN**: Implements `NOLINTNEXTLINE`, one of the operational units in this file for the core torch/csrc bridge layer between Python objects, ATen runtime, and low-level storage/type wrappers.
- **CN**: 实现 `NOLINTNEXTLINE`，它是该文件中服务于连接 Python 对象、ATen 运行时与底层存储/类型封装的 torch/csrc 核心桥接层的一个运行单元。

### Lines 108-120: Supporting statements / 辅助语句
```cpp
static PyObject* THPModule_setWorkerSignalHandlers(
    PyObject* module,
    PyObject* arg) {
  HANDLE_TH_ERRORS
  setSignalHandler(SIGBUS, &handler_SIGBUS, nullptr);
  setSignalHandler(SIGSEGV, &handler_SIGSEGV, nullptr);
  setSignalHandler(SIGTERM, &handler_SIGTERM, nullptr);
  setSignalHandler(SIGFPE, &handler_SIGFPE, nullptr);
  setDataLoaderSignalHandlers();
  Py_RETURN_NONE;
  END_HANDLE_TH_ERRORS
}

```
- **EN**: Contains local statements, constants, or helper glue that support the surrounding control flow and data movement.
- **CN**: 包含局部语句、常量或辅助胶水代码，用于支撑周围的控制流与数据传递。

### Lines 121-122: Supporting statements / 辅助语句
```cpp
static std::map<int64_t, std::set<pid_t>> worker_pids = {};

```
- **EN**: Contains local statements, constants, or helper glue that support the surrounding control flow and data movement.
- **CN**: 包含局部语句、常量或辅助胶水代码，用于支撑周围的控制流与数据传递。

### Lines 123-127: Supporting statements / 辅助语句
```cpp
static PyObject* THPModule_errorIfAnyWorkerFails(
    PyObject* module,
    PyObject* noargs) {
  HANDLE_TH_ERRORS

```
- **EN**: Contains local statements, constants, or helper glue that support the surrounding control flow and data movement.
- **CN**: 包含局部语句、常量或辅助胶水代码，用于支撑周围的控制流与数据传递。

### Lines 128-151: Supporting statements / 辅助语句
```cpp
  // Only check the pids we care about
  for (auto& w : worker_pids) {
    auto& pid_set = w.second;
    for (auto worker_pid : pid_set) {
      // Use waitid rather than waitpid so that we can set NOWAIT, and that
      // Python and other handlers can get whatever info they want about the
      // child.
      siginfo_t infop{};
      infop.si_pid = 0;
      auto error =
          waitid(P_PID, worker_pid, &infop, WEXITED | WNOHANG | WNOWAIT);
      // ignore errors and case with no waitable child
      if (error < 0 || infop.si_pid == 0)
        continue;
      if (infop.si_code == CLD_EXITED &&
          infop.si_status != EXIT_SUCCESS) { // exit with error
        auto error_msg = fmt::format(
            "DataLoader worker (pid {}) exited unexpectedly with exit code {}. "
            "Details are lost due to multiprocessing. Rerunning with "
            "num_workers=0 may give better error trace.",
            worker_pid,
            infop.si_status);
        // This is necessary. Otherwise, the runtime error will kill the other
        // workers, and trigger this again.
```
- **EN**: Contains local statements, constants, or helper glue that support the surrounding control flow and data movement.
- **CN**: 包含局部语句、常量或辅助胶水代码，用于支撑周围的控制流与数据传递。

### Lines 152-175: Supporting statements / 辅助语句
```cpp
        pid_set.clear();
        TORCH_CHECK(false, error_msg);
      } else if (
          infop.si_code == CLD_KILLED ||
          infop.si_code == CLD_DUMPED) { // killed by signal
        auto error_msg = fmt::format(
            "DataLoader worker (pid {}) is killed by signal: {}. ",
            worker_pid,
            strsignal(infop.si_status));
        if (infop.si_status == SIGBUS) {
          error_msg +=
              "It is possible that dataloader's workers are out of shared memory. "
              "Please try to raise your shared memory limit.";
        }
        // This is necessary. Otherwise, the runtime error will kill the other
        // workers, and trigger this again.
        pid_set.clear();
        TORCH_CHECK(false, error_msg);
      }
    }
  }
  Py_RETURN_NONE;
  END_HANDLE_TH_ERRORS
}
```
- **EN**: Contains local statements, constants, or helper glue that support the surrounding control flow and data movement.
- **CN**: 包含局部语句、常量或辅助胶水代码，用于支撑周围的控制流与数据传递。

### Lines 176-199: Supporting statements / 辅助语句
```cpp

// We don't want to exit on any SIGCHLD from any child. child_pids is a tuple
// of pids we are interested in.
static PyObject* THPModule_setWorkerPIDs(PyObject* module, PyObject* args) {
  HANDLE_TH_ERRORS
  TORCH_CHECK_TYPE(
      PyTuple_GET_SIZE(args) == 2,
      "_set_worker_pids expects exactly 2 arguments.");
  int64_t key = THPUtils_unpackLong(PyTuple_GET_ITEM(args, 0));
  TORCH_CHECK_VALUE(
      worker_pids.find(key) == worker_pids.end(),
      "_set_worker_pids should be called only once for each _BaseDataLoaderIter.");
  PyObject* child_pids = PyTuple_GET_ITEM(args, 1);
  TORCH_CHECK_TYPE(
      PyTuple_Check(child_pids),
      "_set_worker_pids expects a tuple for child_pids, but got ",
      Py_TYPE(child_pids)->tp_name,
      ".");
  std::set<pid_t> pids_set = {};
  auto size = PyTuple_GET_SIZE(child_pids);
  for (const auto idx : c10::irange(size)) {
    PyObject* obj = PyTuple_GET_ITEM(child_pids, idx);
    pids_set.insert(static_cast<pid_t>(THPUtils_unpackLong(obj)));
  }
```
- **EN**: Contains local statements, constants, or helper glue that support the surrounding control flow and data movement.
- **CN**: 包含局部语句、常量或辅助胶水代码，用于支撑周围的控制流与数据传递。

### Lines 200-202: Supporting statements / 辅助语句
```cpp

  worker_pids[key] = pids_set;

```
- **EN**: Contains local statements, constants, or helper glue that support the surrounding control flow and data movement.
- **CN**: 包含局部语句、常量或辅助胶水代码，用于支撑周围的控制流与数据传递。

### Lines 203-206: Supporting statements / 辅助语句
```cpp
  Py_RETURN_NONE;
  END_HANDLE_TH_ERRORS
}

```
- **EN**: Contains local statements, constants, or helper glue that support the surrounding control flow and data movement.
- **CN**: 包含局部语句、常量或辅助胶水代码，用于支撑周围的控制流与数据传递。

### Lines 207-211: Supporting statements / 辅助语句
```cpp
static PyObject* THPModule_removeWorkerPIDs(
    PyObject* module,
    PyObject* loader_id) {
  HANDLE_TH_ERRORS

```
- **EN**: Contains local statements, constants, or helper glue that support the surrounding control flow and data movement.
- **CN**: 包含局部语句、常量或辅助胶水代码，用于支撑周围的控制流与数据传递。

### Lines 212-219: Supporting statements / 辅助语句
```cpp
  int64_t key = THPUtils_unpackLong(loader_id);
  auto it = worker_pids.find(key);
  TORCH_CHECK_VALUE(
      it != worker_pids.end(),
      "Cannot find worker information for _BaseDataLoaderIter with id ",
      key);
  worker_pids.erase(it);

```
- **EN**: Contains local statements, constants, or helper glue that support the surrounding control flow and data movement.
- **CN**: 包含局部语句、常量或辅助胶水代码，用于支撑周围的控制流与数据传递。

### Lines 220-223: Supporting statements / 辅助语句
```cpp
  Py_RETURN_NONE;
  END_HANDLE_TH_ERRORS
}

```
- **EN**: Contains local statements, constants, or helper glue that support the surrounding control flow and data movement.
- **CN**: 包含局部语句、常量或辅助胶水代码，用于支撑周围的控制流与数据传递。

### Lines 224-225: Supporting statements / 辅助语句
```cpp
#undef SIGNAL_HANDLER

```
- **EN**: Contains local statements, constants, or helper glue that support the surrounding control flow and data movement.
- **CN**: 包含局部语句、常量或辅助胶水代码，用于支撑周围的控制流与数据传递。

### Lines 226-228: Preprocessor configuration / 预处理配置
```cpp
#else
// dummy implementations for windows

```
- **EN**: Applies compile-time gating, macros, or platform-specific configuration so the file builds correctly under different feature sets.
- **CN**: 应用编译期开关、宏或平台特定配置，使该文件能在不同功能组合下正确构建。

### Lines 229-234: Supporting statements / 辅助语句
```cpp
static PyObject* THPModule_setWorkerSignalHandlers(
    PyObject* module,
    PyObject* _ignored) {
  Py_RETURN_NONE;
}

```
- **EN**: Contains local statements, constants, or helper glue that support the surrounding control flow and data movement.
- **CN**: 包含局部语句、常量或辅助胶水代码，用于支撑周围的控制流与数据传递。

### Lines 235-238: Function `THPModule_setWorkerPIDs` / 函数 `THPModule_setWorkerPIDs`
```cpp
static PyObject* THPModule_setWorkerPIDs(PyObject* module, PyObject* _ignored) {
  Py_RETURN_NONE;
}

```
- **EN**: Implements `THPModule_setWorkerPIDs` as part of the Python/C++ bridge for the core torch/csrc bridge layer between Python objects, ATen runtime, and low-level storage/type wrappers, handling argument conversion, safety checks, or wrapper behavior.
- **CN**: 将 `THPModule_setWorkerPIDs` 实现为连接 Python 对象、ATen 运行时与底层存储/类型封装的 torch/csrc 核心桥接层中的 Python/C++ 桥接逻辑，负责参数转换、安全检查或封装行为。

### Lines 239-244: Supporting statements / 辅助语句
```cpp
static PyObject* THPModule_removeWorkerPIDs(
    PyObject* module,
    PyObject* _ignored) {
  Py_RETURN_NONE;
}

```
- **EN**: Contains local statements, constants, or helper glue that support the surrounding control flow and data movement.
- **CN**: 包含局部语句、常量或辅助胶水代码，用于支撑周围的控制流与数据传递。

### Lines 245-250: Supporting statements / 辅助语句
```cpp
static PyObject* THPModule_errorIfAnyWorkerFails(
    PyObject* module,
    PyObject* _ignored) {
  Py_RETURN_NONE;
}

```
- **EN**: Contains local statements, constants, or helper glue that support the surrounding control flow and data movement.
- **CN**: 包含局部语句、常量或辅助胶水代码，用于支撑周围的控制流与数据传递。

### Lines 251-252: Preprocessor configuration / 预处理配置
```cpp
#endif

```
- **EN**: Applies compile-time gating, macros, or platform-specific configuration so the file builds correctly under different feature sets.
- **CN**: 应用编译期开关、宏或平台特定配置，使该文件能在不同功能组合下正确构建。

### Lines 253-265: Function `NOLINTNEXTLINE` / 函数 `NOLINTNEXTLINE`
```cpp
// NOLINTNEXTLINE(cppcoreguidelines-avoid-c-arrays,cppcoreguidelines-avoid-non-const-global-variables,modernize-avoid-c-arrays)
PyMethodDef DataLoaderMethods[] = {
    {"_set_worker_signal_handlers",
     THPModule_setWorkerSignalHandlers,
     METH_NOARGS,
     nullptr},
    {"_set_worker_pids", THPModule_setWorkerPIDs, METH_VARARGS, nullptr},
    {"_remove_worker_pids", THPModule_removeWorkerPIDs, METH_O, nullptr},
    {"_error_if_any_worker_fails",
     THPModule_errorIfAnyWorkerFails,
     METH_NOARGS,
     nullptr},
    {nullptr, nullptr, 0, nullptr}};
```
- **EN**: Implements `NOLINTNEXTLINE` as part of the Python/C++ bridge for the core torch/csrc bridge layer between Python objects, ATen runtime, and low-level storage/type wrappers, handling argument conversion, safety checks, or wrapper behavior.
- **CN**: 将 `NOLINTNEXTLINE` 实现为连接 Python 对象、ATen 运行时与底层存储/类型封装的 torch/csrc 核心桥接层中的 Python/C++ 桥接逻辑，负责参数转换、安全检查或封装行为。

## Key Concepts / 关键概念
- Python/C++ runtime bridge / Python/C++ 运行时桥接
- Python bindings / Python 绑定

## Dependencies / 依赖关系
### Internal / 内部
- `torch/csrc/DataLoader.h`
- `torch/csrc/Exceptions.h`
- `torch/csrc/utils/python_numbers.h`
- `c10/util/irange.h`
- `sys/wait.h`
### External / 外部
- `fmt/format.h`
- `csignal`
- `map`
- `set`
- `sstream`
