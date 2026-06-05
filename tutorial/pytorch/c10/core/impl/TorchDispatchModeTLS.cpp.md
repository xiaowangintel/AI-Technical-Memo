# TorchDispatchModeTLS.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `c10/core/impl/TorchDispatchModeTLS.cpp`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Defines c10 core abstractions such as tensor metadata, dispatch keys, devices, storage, and symbolic values.
- **Purpose (CN)**: 定义 c10 的核心抽象，如张量元数据、分发键、设备、存储以及符号值。

## Line-by-Line Analysis / 逐行分析
### Lines 1-10
```cpp
#include <c10/core/DispatchKey.h>
#include <c10/core/impl/LocalDispatchKeySet.h>
#include <c10/core/impl/TorchDispatchModeTLS.h>
#include <c10/util/irange.h>

#include <utility>

namespace c10::impl {

thread_local static TorchDispatchModeTLS torchDispatchModeState;
```
- **EN**: This block assembles the compilation dependencies, pulling in local PyTorch/c10 headers such as c10/core/DispatchKey.h, c10/core/impl/LocalDispatchKeySet.h, c10/core/impl/TorchDispatchModeTLS.h, and 1 more; standard-library headers such as utility. The namespace declarations place the code inside c10::impl, matching the surrounding subsystem. Dispatch-oriented logic computes or queries backend/functionality state so later calls reach the correct kernel path. Container logic organizes metadata, caches, or lookup state so surrounding code can access it efficiently.
- **CN**: 这一段组织编译依赖，引入了本地 PyTorch/c10 头文件，如 c10/core/DispatchKey.h、c10/core/impl/LocalDispatchKeySet.h、c10/core/impl/TorchDispatchModeTLS.h 等共 4 项；标准库头文件，如 utility。 命名空间声明把代码放入 c10::impl 中，与周边子系统保持一致。 面向分发的逻辑会计算或查询后端/功能状态，从而让后续调用进入正确的内核路径。 容器逻辑用于组织元数据、缓存或查找状态，从而让周边代码能够高效访问。

### Lines 12-22
```cpp
bool TorchDispatchModeTLS::any_modes_set(bool skip_infra_modes) {
  if (!torchDispatchModeState.stack_.empty())
    return true;
  if (!skip_infra_modes) {
    for (const auto i : c10::irange(
             static_cast<size_t>(TorchDispatchModeKey::NUM_MODE_KEYS))) {
      if (torchDispatchModeState.infra_modes_[i] != std::nullopt) {
        return true;
      }
    }
  }
```
- **EN**: This chunk defines `any_modes_set`, which maintains lookup structures and hashing behavior for fast metadata access. Dispatch-oriented logic computes or queries backend/functionality state so later calls reach the correct kernel path. Container logic organizes metadata, caches, or lookup state so surrounding code can access it efficiently. Looping logic walks collections, device tables, or metadata arrays so the same rule is applied systematically. Conditional branches split fast paths, error cases, and special-case invariants. The tail returns accumulated results or hands the updated state back to the caller.
- **CN**: 这一段定义了 `any_modes_set`，其作用是维护查找结构与哈希行为，以便快速访问元数据。 面向分发的逻辑会计算或查询后端/功能状态，从而让后续调用进入正确的内核路径。 容器逻辑用于组织元数据、缓存或查找状态，从而让周边代码能够高效访问。 循环逻辑会遍历集合、设备表或元数据数组，从而把同一规则系统地应用到每个元素。 条件分支用于区分快速路径、错误场景以及特殊情况不变量。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 23-34
```cpp
  return false;
}

void TorchDispatchModeTLS::push_non_infra_mode_onto_stack(
    std::shared_ptr<PyObject_TorchDispatchMode> mode) {
  if (!any_modes_set()) {
    c10::impl::tls_set_dispatch_key_included(DispatchKey::Python, true);
    c10::impl::tls_set_dispatch_key_included(
        DispatchKey::PythonTLSSnapshot, true);
  }
  torchDispatchModeState.stack_.push_back(std::move(mode));
}
```
- **EN**: This chunk defines `push_back`, which implements a focused piece of c10 core logic. Dispatch-oriented logic computes or queries backend/functionality state so later calls reach the correct kernel path. Container logic organizes metadata, caches, or lookup state so surrounding code can access it efficiently. Conditional branches split fast paths, error cases, and special-case invariants. The tail returns accumulated results or hands the updated state back to the caller.
- **CN**: 这一段定义了 `push_back`，其作用是实现一段聚焦的 c10 核心逻辑。 面向分发的逻辑会计算或查询后端/功能状态，从而让后续调用进入正确的内核路径。 容器逻辑用于组织元数据、缓存或查找状态，从而让周边代码能够高效访问。 条件分支用于区分快速路径、错误场景以及特殊情况不变量。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 36-47
```cpp
const std::shared_ptr<PyObject_TorchDispatchMode> TorchDispatchModeTLS::
    pop_stack() {
  std::shared_ptr<PyObject_TorchDispatchMode> out;
  if (!torchDispatchModeState.stack_.empty()) {
    out = torchDispatchModeState.stack_.back();
    torchDispatchModeState.stack_.pop_back();
  } else {
    for (int64_t i =
             static_cast<size_t>(TorchDispatchModeKey::NUM_MODE_KEYS) - 1;
         i >= 0;
         --i) {
      if (torchDispatchModeState.infra_modes_[i].has_value()) {
```
- **EN**: This chunk defines `pop_back`, which implements a focused piece of c10 core logic. Dispatch-oriented logic computes or queries backend/functionality state so later calls reach the correct kernel path. Looping logic walks collections, device tables, or metadata arrays so the same rule is applied systematically. Conditional branches split fast paths, error cases, and special-case invariants.
- **CN**: 这一段定义了 `pop_back`，其作用是实现一段聚焦的 c10 核心逻辑。 面向分发的逻辑会计算或查询后端/功能状态，从而让后续调用进入正确的内核路径。 循环逻辑会遍历集合、设备表或元数据数组，从而把同一规则系统地应用到每个元素。 条件分支用于区分快速路径、错误场景以及特殊情况不变量。

### Lines 48-59
```cpp
        // NOLINTNEXTLINE(bugprone-unchecked-optional-access)
        out = std::move(torchDispatchModeState.infra_modes_[i].value());
        torchDispatchModeState.infra_modes_[i] = std::nullopt;
        break;
      }
    }
  }
  TORCH_CHECK(out, "trying to pop from empty mode stack");
  if (!any_modes_set()) {
    c10::impl::tls_set_dispatch_key_included(DispatchKey::Python, false);
    c10::impl::tls_set_dispatch_key_included(
        DispatchKey::PythonTLSSnapshot, false);
```
- **EN**: This chunk defines `tls_set_dispatch_key_included`, which queries or constructs dispatch-related state used for backend/kernel selection. Preprocessor and exported-macro usage adapts the code to compiler, platform, and visibility constraints. Dispatch-oriented logic computes or queries backend/functionality state so later calls reach the correct kernel path. Container logic organizes metadata, caches, or lookup state so surrounding code can access it efficiently. Conditional branches split fast paths, error cases, and special-case invariants. Assertions and failure paths make invalid states fail early instead of silently propagating corruption.
- **CN**: 这一段定义了 `tls_set_dispatch_key_included`，其作用是查询或构造用于后端/内核选择的分发相关状态。 预处理器与导出宏的使用让代码适配编译器、平台以及可见性约束。 面向分发的逻辑会计算或查询后端/功能状态，从而让后续调用进入正确的内核路径。 容器逻辑用于组织元数据、缓存或查找状态，从而让周边代码能够高效访问。 条件分支用于区分快速路径、错误场景以及特殊情况不变量。 断言与失败路径让非法状态尽早暴露，而不是悄悄传播错误。

### Lines 60-71
```cpp
  }
  return out;
}
const std::
    tuple<std::shared_ptr<PyObject_TorchDispatchMode>, TorchDispatchModeKey>
    TorchDispatchModeTLS::pop_highest_infra_mode() {
  for (int64_t i = static_cast<size_t>(TorchDispatchModeKey::NUM_MODE_KEYS) - 1;
       i >= 0;
       --i) {
    if (torchDispatchModeState.infra_modes_[i].has_value()) {
      // NOLINTNEXTLINE(bugprone-unchecked-optional-access)
      auto out_mode = torchDispatchModeState.infra_modes_[i].value();
```
- **EN**: This chunk defines `pop_highest_infra_mode`, which implements a focused piece of c10 core logic. Dispatch-oriented logic computes or queries backend/functionality state so later calls reach the correct kernel path. Looping logic walks collections, device tables, or metadata arrays so the same rule is applied systematically. Conditional branches split fast paths, error cases, and special-case invariants. The tail returns accumulated results or hands the updated state back to the caller.
- **CN**: 这一段定义了 `pop_highest_infra_mode`，其作用是实现一段聚焦的 c10 核心逻辑。 面向分发的逻辑会计算或查询后端/功能状态，从而让后续调用进入正确的内核路径。 循环逻辑会遍历集合、设备表或元数据数组，从而把同一规则系统地应用到每个元素。 条件分支用于区分快速路径、错误场景以及特殊情况不变量。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 72-81
```cpp
      torchDispatchModeState.infra_modes_[i] = std::nullopt;
      if (!any_modes_set()) {
        c10::impl::tls_set_dispatch_key_included(DispatchKey::Python, false);
        c10::impl::tls_set_dispatch_key_included(
            DispatchKey::PythonTLSSnapshot, false);
      }
      return std::make_tuple(
          std::move(out_mode), static_cast<TorchDispatchModeKey>(i));
    }
  }
```
- **EN**: This chunk defines `make_tuple`, which constructs derived state from the current inputs and invariants. Dispatch-oriented logic computes or queries backend/functionality state so later calls reach the correct kernel path. Container logic organizes metadata, caches, or lookup state so surrounding code can access it efficiently. Conditional branches split fast paths, error cases, and special-case invariants. The tail returns accumulated results or hands the updated state back to the caller.
- **CN**: 这一段定义了 `make_tuple`，其作用是根据当前输入与不变量构建派生状态。 面向分发的逻辑会计算或查询后端/功能状态，从而让后续调用进入正确的内核路径。 容器逻辑用于组织元数据、缓存或查找状态，从而让周边代码能够高效访问。 条件分支用于区分快速路径、错误场景以及特殊情况不变量。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 82-92
```cpp
  TORCH_CHECK(
      false, "Called pop_highest_infra_mode, but no infra modes were active.")
}

const std::shared_ptr<PyObject_TorchDispatchMode>& TorchDispatchModeTLS::
    get_stack_at(int64_t idx) {
  TORCH_CHECK(idx < stack_len(), "Tried to get stack at idx that's too big");
  // Our "logical" stack includes both:
  // - any user modes (the entire torchDispatchModeState.stack_)
  // - any infra modes (members of torchDispatchModeState.infra_modes_ that are
  // not None)
```
- **EN**: This chunk defines `get_stack_at`, which implements a focused piece of c10 core logic. Preprocessor and exported-macro usage adapts the code to compiler, platform, and visibility constraints. Dispatch-oriented logic computes or queries backend/functionality state so later calls reach the correct kernel path. Assertions and failure paths make invalid states fail early instead of silently propagating corruption.
- **CN**: 这一段定义了 `get_stack_at`，其作用是实现一段聚焦的 c10 核心逻辑。 预处理器与导出宏的使用让代码适配编译器、平台以及可见性约束。 面向分发的逻辑会计算或查询后端/功能状态，从而让后续调用进入正确的内核路径。 断言与失败路径让非法状态尽早暴露，而不是悄悄传播错误。

### Lines 94-105
```cpp
  // idx == 0 means the "bottom" of the stack, which starts with any infra
  // modes (iterating from lowest-priority to highest-priority).
  auto curr_idx = idx;
  for (const auto i :
       c10::irange(static_cast<size_t>(TorchDispatchModeKey::NUM_MODE_KEYS))) {
    if (torchDispatchModeState.infra_modes_[i].has_value()) {
      if (curr_idx == 0) {
        // NOLINTNEXTLINE(bugprone-unchecked-optional-access)
        return torchDispatchModeState.infra_modes_[i].value();
      }
      curr_idx -= 1;
    }
```
- **EN**: This chunk continues `get_stack_at` and expands its control flow, data movement, or edge-case handling. Dispatch-oriented logic computes or queries backend/functionality state so later calls reach the correct kernel path. Looping logic walks collections, device tables, or metadata arrays so the same rule is applied systematically. Conditional branches split fast paths, error cases, and special-case invariants. The tail returns accumulated results or hands the updated state back to the caller.
- **CN**: 这一段延续了 `get_stack_at`，进一步展开其控制流、数据流转或边界处理逻辑。 面向分发的逻辑会计算或查询后端/功能状态，从而让后续调用进入正确的内核路径。 循环逻辑会遍历集合、设备表或元数据数组，从而把同一规则系统地应用到每个元素。 条件分支用于区分快速路径、错误场景以及特殊情况不变量。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 106-117
```cpp
  }
  // At this point, we're guaranteed that curr_idx < stack_.size()
  return torchDispatchModeState.stack_[curr_idx];
}

int64_t TorchDispatchModeTLS::stack_len() {
  auto stack_len = static_cast<int64_t>(torchDispatchModeState.stack_.size());
  int64_t infra_modes_len = 0;
  for (const auto i :
       c10::irange(static_cast<size_t>(TorchDispatchModeKey::NUM_MODE_KEYS))) {
    if (torchDispatchModeState.infra_modes_[i] != std::nullopt) {
      infra_modes_len += 1;
```
- **EN**: This chunk defines `static_cast<int64_t>`, which implements a focused piece of c10 core logic. Dispatch-oriented logic computes or queries backend/functionality state so later calls reach the correct kernel path. Looping logic walks collections, device tables, or metadata arrays so the same rule is applied systematically. Conditional branches split fast paths, error cases, and special-case invariants. The tail returns accumulated results or hands the updated state back to the caller.
- **CN**: 这一段定义了 `static_cast<int64_t>`，其作用是实现一段聚焦的 c10 核心逻辑。 面向分发的逻辑会计算或查询后端/功能状态，从而让后续调用进入正确的内核路径。 循环逻辑会遍历集合、设备表或元数据数组，从而把同一规则系统地应用到每个元素。 条件分支用于区分快速路径、错误场景以及特殊情况不变量。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 118-126
```cpp
    }
  }
  return stack_len + infra_modes_len;
}

const std::optional<std::shared_ptr<PyObject_TorchDispatchMode>>
TorchDispatchModeTLS::get_mode(TorchDispatchModeKey mode_key) {
  return torchDispatchModeState.infra_modes_[static_cast<size_t>(mode_key)];
}
```
- **EN**: This chunk defines `get_mode`, which implements a focused piece of c10 core logic. Dispatch-oriented logic computes or queries backend/functionality state so later calls reach the correct kernel path. The tail returns accumulated results or hands the updated state back to the caller.
- **CN**: 这一段定义了 `get_mode`，其作用是实现一段聚焦的 c10 核心逻辑。 面向分发的逻辑会计算或查询后端/功能状态，从而让后续调用进入正确的内核路径。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 128-136
```cpp
void TorchDispatchModeTLS::set_mode(
    const std::shared_ptr<PyObject_TorchDispatchMode>& mode,
    TorchDispatchModeKey mode_key) {
  TORCH_CHECK(
      torchDispatchModeState.infra_modes_[static_cast<size_t>(mode_key)] ==
          std::nullopt,
      "trying to set the current ",
      to_string(mode_key),
      ", but one already exists");
```
- **EN**: This chunk defines `set_mode`, which maintains lookup structures and hashing behavior for fast metadata access. Preprocessor and exported-macro usage adapts the code to compiler, platform, and visibility constraints. Dispatch-oriented logic computes or queries backend/functionality state so later calls reach the correct kernel path. Container logic organizes metadata, caches, or lookup state so surrounding code can access it efficiently. Assertions and failure paths make invalid states fail early instead of silently propagating corruption.
- **CN**: 这一段定义了 `set_mode`，其作用是维护查找结构与哈希行为，以便快速访问元数据。 预处理器与导出宏的使用让代码适配编译器、平台以及可见性约束。 面向分发的逻辑会计算或查询后端/功能状态，从而让后续调用进入正确的内核路径。 容器逻辑用于组织元数据、缓存或查找状态，从而让周边代码能够高效访问。 断言与失败路径让非法状态尽早暴露，而不是悄悄传播错误。

### Lines 138-145
```cpp
  if (!any_modes_set()) {
    c10::impl::tls_set_dispatch_key_included(DispatchKey::Python, true);
    c10::impl::tls_set_dispatch_key_included(
        DispatchKey::PythonTLSSnapshot, true);
  }

  torchDispatchModeState.infra_modes_[static_cast<size_t>(mode_key)] = mode;
}
```
- **EN**: This chunk defines `tls_set_dispatch_key_included`, which queries or constructs dispatch-related state used for backend/kernel selection. Dispatch-oriented logic computes or queries backend/functionality state so later calls reach the correct kernel path. Container logic organizes metadata, caches, or lookup state so surrounding code can access it efficiently. Conditional branches split fast paths, error cases, and special-case invariants.
- **CN**: 这一段定义了 `tls_set_dispatch_key_included`，其作用是查询或构造用于后端/内核选择的分发相关状态。 面向分发的逻辑会计算或查询后端/功能状态，从而让后续调用进入正确的内核路径。 容器逻辑用于组织元数据、缓存或查找状态，从而让周边代码能够高效访问。 条件分支用于区分快速路径、错误场景以及特殊情况不变量。

### Lines 147-158
```cpp
const std::optional<std::shared_ptr<PyObject_TorchDispatchMode>>
TorchDispatchModeTLS::unset_mode(TorchDispatchModeKey mode_key) {
  auto out = torchDispatchModeState.infra_modes_[static_cast<size_t>(mode_key)];
  torchDispatchModeState.infra_modes_[static_cast<size_t>(mode_key)] =
      std::nullopt;
  if (out.has_value() && !any_modes_set()) {
    c10::impl::tls_set_dispatch_key_included(DispatchKey::Python, false);
    c10::impl::tls_set_dispatch_key_included(
        DispatchKey::PythonTLSSnapshot, false);
  }
  return out;
}
```
- **EN**: This chunk defines `tls_set_dispatch_key_included`, which queries or constructs dispatch-related state used for backend/kernel selection. Dispatch-oriented logic computes or queries backend/functionality state so later calls reach the correct kernel path. Container logic organizes metadata, caches, or lookup state so surrounding code can access it efficiently. Conditional branches split fast paths, error cases, and special-case invariants. The tail returns accumulated results or hands the updated state back to the caller.
- **CN**: 这一段定义了 `tls_set_dispatch_key_included`，其作用是查询或构造用于后端/内核选择的分发相关状态。 面向分发的逻辑会计算或查询后端/功能状态，从而让后续调用进入正确的内核路径。 容器逻辑用于组织元数据、缓存或查找状态，从而让周边代码能够高效访问。 条件分支用于区分快速路径、错误场景以及特殊情况不变量。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 160-171
```cpp
const TorchDispatchModeTLS& TorchDispatchModeTLS::get_state() {
  return torchDispatchModeState;
}

void TorchDispatchModeTLS::set_state(TorchDispatchModeTLS state) {
  torchDispatchModeState = std::move(state);
  if (!any_modes_set()) {
    c10::impl::tls_set_dispatch_key_included(DispatchKey::Python, false);
    c10::impl::tls_set_dispatch_key_included(
        DispatchKey::PythonTLSSnapshot, false);
  } else {
    c10::impl::tls_set_dispatch_key_included(DispatchKey::Python, true);
```
- **EN**: This chunk defines `tls_set_dispatch_key_included`, which queries or constructs dispatch-related state used for backend/kernel selection. Dispatch-oriented logic computes or queries backend/functionality state so later calls reach the correct kernel path. Container logic organizes metadata, caches, or lookup state so surrounding code can access it efficiently. Conditional branches split fast paths, error cases, and special-case invariants. The tail returns accumulated results or hands the updated state back to the caller.
- **CN**: 这一段定义了 `tls_set_dispatch_key_included`，其作用是查询或构造用于后端/内核选择的分发相关状态。 面向分发的逻辑会计算或查询后端/功能状态，从而让后续调用进入正确的内核路径。 容器逻辑用于组织元数据、缓存或查找状态，从而让周边代码能够高效访问。 条件分支用于区分快速路径、错误场景以及特殊情况不变量。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 172-182
```cpp
    c10::impl::tls_set_dispatch_key_included(
        DispatchKey::PythonTLSSnapshot, true);
  }
}

// UTIL

bool dispatch_mode_enabled() {
  return !c10::impl::tls_is_dispatch_key_excluded(DispatchKey::Python) &&
      TorchDispatchModeTLS::any_modes_set();
}
```
- **EN**: This chunk defines `tls_is_dispatch_key_excluded`, which queries or constructs dispatch-related state used for backend/kernel selection. Dispatch-oriented logic computes or queries backend/functionality state so later calls reach the correct kernel path. Container logic organizes metadata, caches, or lookup state so surrounding code can access it efficiently. The tail returns accumulated results or hands the updated state back to the caller.
- **CN**: 这一段定义了 `tls_is_dispatch_key_excluded`，其作用是查询或构造用于后端/内核选择的分发相关状态。 面向分发的逻辑会计算或查询后端/功能状态，从而让后续调用进入正确的内核路径。 容器逻辑用于组织元数据、缓存或查找状态，从而让周边代码能够高效访问。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 184-195
```cpp
std::string to_string(TorchDispatchModeKey mode_key) {
  switch (mode_key) {
    case TorchDispatchModeKey::PROXY:
      return "ProxyTorchDispatchMode";
    case TorchDispatchModeKey::FAKE:
      return "FakeTensorMode";
    default:
      return "UNKNOWN_MODE";
  }
}

} // namespace c10::impl
```
- **EN**: This chunk defines `to_string`, which converts one representation into another form used by nearby runtime code. Dispatch-oriented logic computes or queries backend/functionality state so later calls reach the correct kernel path. Switch-based dispatch selects specialized behavior for enum variants, backend kinds, or opcode-like cases. The tail returns accumulated results or hands the updated state back to the caller.
- **CN**: 这一段定义了 `to_string`，其作用是把一种表示转换为附近运行时代码使用的另一种形式。 面向分发的逻辑会计算或查询后端/功能状态，从而让后续调用进入正确的内核路径。 基于 switch 的分发会为枚举分支、后端类型或类似 opcode 的场景选择专门行为。 末尾会返回累积结果，或把更新后的状态交还给调用方。


## Key Concepts / 关键概念
- **c10 core abstractions**
  - EN: Defines c10 core abstractions such as tensor metadata, dispatch keys, devices, storage, and symbolic values.
  - CN: 定义 c10 的核心抽象，如张量元数据、分发键、设备、存储以及符号值。
- **any_modes_set**
  - EN: `any_modes_set` is one of the dominant symbols declared or implemented in this file.
  - CN: `any_modes_set` 是本文件声明或实现的关键符号之一。
- **push_non_infra_mode_onto_stack**
  - EN: `push_non_infra_mode_onto_stack` is one of the dominant symbols declared or implemented in this file.
  - CN: `push_non_infra_mode_onto_stack` 是本文件声明或实现的关键符号之一。
- **Dispatch key system**
  - EN: Maps backend/functionality combinations to runtime kernel-selection behavior.
  - CN: 把后端/功能组合映射到运行时内核选择行为。

## Dependencies / 依赖关系
- **Internal includes / 内部依赖**: `c10/core/DispatchKey.h`、`c10/core/impl/LocalDispatchKeySet.h`、`c10/core/impl/TorchDispatchModeTLS.h`、`c10/util/irange.h`
- **Third-party includes / 第三方依赖**: 无
- **Standard includes / 标准库依赖**: `utility`
- **System includes / 系统依赖**: 无
- **Namespaces / 命名空间**: `c10::impl`
- **Representative symbols / 代表性符号**: `any_modes_set`、`push_non_infra_mode_onto_stack`、`tls_set_dispatch_key_included`、`push_back`、`pop_stack`、`back`、`pop_back`、`pop_highest_infra_mode`、`make_tuple`、`get_stack_at`
