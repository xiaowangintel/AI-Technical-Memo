# LeftRight.h — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `c10/util/LeftRight.h`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Provides low-level c10 utilities for ownership, containers, hashing, type traits, errors, and portability.
- **Purpose (CN)**: 提供 c10 的底层工具能力，包括所有权管理、容器、哈希、类型萃取、错误处理与可移植性支持。

## Line-by-Line Analysis / 逐行分析
### Lines 1-18
```cpp
#pragma once

#include <c10/macros/Macros.h>
#include <c10/util/Synchronized.h>
#include <array>
#include <atomic>
#include <mutex>
#include <thread>

namespace c10 {

namespace detail {

struct IncrementRAII final {
 public:
  explicit IncrementRAII(std::atomic<int32_t>* counter) : _counter(counter) {
    _counter->fetch_add(1);
  }
```
- **EN**: This block assembles the compilation dependencies, pulling in local PyTorch/c10 headers such as c10/macros/Macros.h, c10/util/Synchronized.h; standard-library headers such as array, atomic, mutex, and 1 more. The preprocessor guard keeps declarations single-instanced when this header is included transitively. The namespace declarations place the code inside c10, detail, matching the surrounding subsystem. It introduces or extends IncrementRAII, which define the main data structures or interfaces for this portion of the file. This chunk defines `fetch_add`, which implements a reusable low-level helper for higher-level runtime code. Synchronization primitives protect shared state and make concurrent updates deterministic enough for runtime use. Container logic organizes metadata, caches, or lookup state so surrounding code can access it efficiently.
- **CN**: 这一段组织编译依赖，引入了本地 PyTorch/c10 头文件，如 c10/macros/Macros.h、c10/util/Synchronized.h；标准库头文件，如 array、atomic、mutex 等共 4 项。 预处理器保护用于避免头文件在传递包含时被重复展开。 命名空间声明把代码放入 c10、detail 中，与周边子系统保持一致。 它引入或扩展了 IncrementRAII，这些类型定义了本段涉及的主要数据结构或接口。 这一段定义了 `fetch_add`，其作用是实现供更高层运行时代码复用的底层辅助逻辑。 同步原语用于保护共享状态，并让并发更新在运行时场景下保持足够确定。 容器逻辑用于组织元数据、缓存或查找状态，从而让周边代码能够高效访问。

### Lines 20-32
```cpp
  ~IncrementRAII() {
    _counter->fetch_sub(1);
  }
  IncrementRAII(IncrementRAII&&) = delete;
  IncrementRAII& operator=(IncrementRAII&&) = delete;

 private:
  std::atomic<int32_t>* _counter;

  C10_DISABLE_COPY_AND_ASSIGN(IncrementRAII);
};

} // namespace detail
```
- **EN**: This chunk defines `fetch_sub`, which implements a reusable low-level helper for higher-level runtime code. Preprocessor and exported-macro usage adapts the code to compiler, platform, and visibility constraints. Synchronization primitives protect shared state and make concurrent updates deterministic enough for runtime use. Assertions and failure paths make invalid states fail early instead of silently propagating corruption.
- **CN**: 这一段定义了 `fetch_sub`，其作用是实现供更高层运行时代码复用的底层辅助逻辑。 预处理器与导出宏的使用让代码适配编译器、平台以及可见性约束。 同步原语用于保护共享状态，并让并发更新在运行时场景下保持足够确定。 断言与失败路径让非法状态尽早暴露，而不是悄悄传播错误。

### Lines 34-51
```cpp
// LeftRight wait-free readers synchronization primitive
// https://hal.archives-ouvertes.fr/hal-01207881/document
//
// LeftRight is quite easy to use (it can make an arbitrary
// data structure permit wait-free reads), but it has some
// particular performance characteristics you should be aware
// of if you're deciding to use it:
//
//  - Reads still incur an atomic write (this is how LeftRight
//    keeps track of how long it needs to keep around the old
//    data structure)
//
//  - Writes get executed twice, to keep both the left and right
//    versions up to date.  So if your write is expensive or
//    nondeterministic, this is also an inappropriate structure
//
// LeftRight is used fairly rarely in PyTorch's codebase.  If you
// are still not sure if you need it or not, consult your local
```
- **EN**: Conditional branches split fast paths, error cases, and special-case invariants.
- **CN**: 条件分支用于区分快速路径、错误场景以及特殊情况不变量。

### Lines 52-62
```cpp
// C++ expert.
//
template <class T>
class LeftRight final {
 public:
  template <class... Args>
  explicit LeftRight(const Args&... args)
      : _counters{{{0}, {0}}},
        _foregroundCounterIndex(0),
        _foregroundDataIndex(0),
        _data{{T{args...}, T{args...}}} {}
```
- **EN**: It introduces or extends T, LeftRight, which define the main data structures or interfaces for this portion of the file. This chunk defines `LeftRight`, which implements a reusable low-level helper for higher-level runtime code. Template machinery keeps the implementation reusable across scalar types, pointer targets, or backend-specific policies.
- **CN**: 它引入或扩展了 T、LeftRight，这些类型定义了本段涉及的主要数据结构或接口。 这一段定义了 `LeftRight`，其作用是实现供更高层运行时代码复用的底层辅助逻辑。 模板机制让实现能够在标量类型、指针目标或后端策略之间复用。

### Lines 64-81
```cpp
  // Copying and moving would not be threadsafe.
  // Needs more thought and careful design to make that work.
  LeftRight(const LeftRight&) = delete;
  LeftRight(LeftRight&&) noexcept = delete;
  LeftRight& operator=(const LeftRight&) = delete;
  LeftRight& operator=(LeftRight&&) noexcept = delete;

  ~LeftRight() {
    // wait until any potentially running writers are finished
    {
      std::unique_lock<std::mutex> lock(_writeMutex);
    }

    // wait until any potentially running readers are finished
    while (_counters[0].load() != 0 || _counters[1].load() != 0) {
      std::this_thread::yield();
    }
  }
```
- **EN**: This chunk defines `yield`, which implements a reusable low-level helper for higher-level runtime code. Synchronization primitives protect shared state and make concurrent updates deterministic enough for runtime use. Looping logic walks collections, device tables, or metadata arrays so the same rule is applied systematically.
- **CN**: 这一段定义了 `yield`，其作用是实现供更高层运行时代码复用的底层辅助逻辑。 同步原语用于保护共享状态，并让并发更新在运行时场景下保持足够确定。 循环逻辑会遍历集合、设备表或元数据数组，从而把同一规则系统地应用到每个元素。

### Lines 83-99
```cpp
  template <typename F>
  auto read(F&& readFunc) const {
    detail::IncrementRAII _increment_counter(
        &_counters[_foregroundCounterIndex.load()]);

    return std::forward<F>(readFunc)(_data[_foregroundDataIndex.load()]);
  }

  // Throwing an exception in writeFunc is ok but causes the state to be either
  // the old or the new state, depending on if the first or the second call to
  // writeFunc threw.
  template <typename F>
  auto write(F&& writeFunc) {
    std::unique_lock<std::mutex> lock(_writeMutex);

    return _write(std::forward<F>(writeFunc));
  }
```
- **EN**: This chunk defines `_write`, which implements a reusable low-level helper for higher-level runtime code. Template machinery keeps the implementation reusable across scalar types, pointer targets, or backend-specific policies. Synchronization primitives protect shared state and make concurrent updates deterministic enough for runtime use. Conditional branches split fast paths, error cases, and special-case invariants. The tail returns accumulated results or hands the updated state back to the caller.
- **CN**: 这一段定义了 `_write`，其作用是实现供更高层运行时代码复用的底层辅助逻辑。 模板机制让实现能够在标量类型、指针目标或后端策略之间复用。 同步原语用于保护共享状态，并让并发更新在运行时场景下保持足够确定。 条件分支用于区分快速路径、错误场景以及特殊情况不变量。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 101-118
```cpp
 private:
  template <class F>
  auto _write(const F& writeFunc) {
    /*
     * Assume, A is in background and B in foreground. In simplified terms, we
     * want to do the following:
     * 1. Write to A (old background)
     * 2. Switch A/B
     * 3. Write to B (new background)
     *
     * More detailed algorithm (explanations on why this is important are below
     * in code):
     * 1. Write to A
     * 2. Switch A/B data pointers
     * 3. Wait until A counter is zero
     * 4. Switch A/B counters
     * 5. Wait until B counter is zero
     * 6. Write to B
```
- **EN**: It introduces or extends F, which define the main data structures or interfaces for this portion of the file. This chunk defines `_write`, which implements a reusable low-level helper for higher-level runtime code. Template machinery keeps the implementation reusable across scalar types, pointer targets, or backend-specific policies.
- **CN**: 它引入或扩展了 F，这些类型定义了本段涉及的主要数据结构或接口。 这一段定义了 `_write`，其作用是实现供更高层运行时代码复用的底层辅助逻辑。 模板机制让实现能够在标量类型、指针目标或后端策略之间复用。

### Lines 119-136
```cpp
     */

    auto localDataIndex = _foregroundDataIndex.load();

    // 1. Write to A
    _callWriteFuncOnBackgroundInstance(writeFunc, localDataIndex);

    // 2. Switch A/B data pointers
    localDataIndex = localDataIndex ^ 1;
    _foregroundDataIndex = localDataIndex;

    /*
     * 3. Wait until A counter is zero
     *
     * In the previous write run, A was foreground and B was background.
     * There was a time after switching _foregroundDataIndex (B to foreground)
     * and before switching _foregroundCounterIndex, in which new readers could
     * have read B but incremented A's counter.
```
- **EN**: This chunk declares `_callWriteFuncOnBackgroundInstance`, which implements a reusable low-level helper for higher-level runtime code.
- **CN**: 这一段声明了 `_callWriteFuncOnBackgroundInstance`，其作用是实现供更高层运行时代码复用的底层辅助逻辑。

### Lines 137-153
```cpp
     *
     * In this current run, we just switched _foregroundDataIndex (A back to
     * foreground), but before writing to the new background B, we have to make
     * sure A's counter was zero briefly, so all these old readers are gone.
     */
    auto localCounterIndex = _foregroundCounterIndex.load();
    _waitForBackgroundCounterToBeZero(localCounterIndex);

    /*
     * 4. Switch A/B counters
     *
     * Now that we know all readers on B are really gone, we can switch the
     * counters and have new readers increment A's counter again, which is the
     * correct counter since they're reading A.
     */
    localCounterIndex = localCounterIndex ^ 1;
    _foregroundCounterIndex = localCounterIndex;
```
- **EN**: This chunk declares `_waitForBackgroundCounterToBeZero`, which converts one representation into another form used by nearby runtime code. Switch-based dispatch selects specialized behavior for enum variants, backend kinds, or opcode-like cases.
- **CN**: 这一段声明了 `_waitForBackgroundCounterToBeZero`，其作用是把一种表示转换为附近运行时代码使用的另一种形式。 基于 switch 的分发会为枚举分支、后端类型或类似 opcode 的场景选择专门行为。

### Lines 155-166
```cpp
    /*
     * 5. Wait until B counter is zero
     *
     * This waits for all the readers on B that came in while both data and
     * counter for B was in foreground, i.e. normal readers that happened
     * outside of that brief gap between switching data and counter.
     */
    _waitForBackgroundCounterToBeZero(localCounterIndex);

    // 6. Write to B
    return _callWriteFuncOnBackgroundInstance(writeFunc, localDataIndex);
  }
```
- **EN**: This chunk declares `_callWriteFuncOnBackgroundInstance`, which implements a reusable low-level helper for higher-level runtime code. Looping logic walks collections, device tables, or metadata arrays so the same rule is applied systematically. The tail returns accumulated results or hands the updated state back to the caller.
- **CN**: 这一段声明了 `_callWriteFuncOnBackgroundInstance`，其作用是实现供更高层运行时代码复用的底层辅助逻辑。 循环逻辑会遍历集合、设备表或元数据数组，从而把同一规则系统地应用到每个元素。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 168-185
```cpp
  template <class F>
  auto _callWriteFuncOnBackgroundInstance(
      const F& writeFunc,
      uint8_t localDataIndex) {
    try {
      return writeFunc(_data[localDataIndex ^ 1]);
    } catch (...) {
      // recover invariant by copying from the foreground instance
      _data[localDataIndex ^ 1] = _data[localDataIndex];
      // rethrow
      throw;
    }
  }

  void _waitForBackgroundCounterToBeZero(uint8_t counterIndex) {
    while (_counters[counterIndex ^ 1].load() != 0) {
      std::this_thread::yield();
    }
```
- **EN**: It introduces or extends F, which define the main data structures or interfaces for this portion of the file. This chunk defines `yield`, which implements a reusable low-level helper for higher-level runtime code. Template machinery keeps the implementation reusable across scalar types, pointer targets, or backend-specific policies. Looping logic walks collections, device tables, or metadata arrays so the same rule is applied systematically. Assertions and failure paths make invalid states fail early instead of silently propagating corruption. The tail returns accumulated results or hands the updated state back to the caller.
- **CN**: 它引入或扩展了 F，这些类型定义了本段涉及的主要数据结构或接口。 这一段定义了 `yield`，其作用是实现供更高层运行时代码复用的底层辅助逻辑。 模板机制让实现能够在标量类型、指针目标或后端策略之间复用。 循环逻辑会遍历集合、设备表或元数据数组，从而把同一规则系统地应用到每个元素。 断言与失败路径让非法状态尽早暴露，而不是悄悄传播错误。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 186-201
```cpp
  }

  mutable std::array<std::atomic<int32_t>, 2> _counters;
  std::atomic<uint8_t> _foregroundCounterIndex;
  std::atomic<uint8_t> _foregroundDataIndex;
  std::array<T, 2> _data;
  std::mutex _writeMutex;
};

// RWSafeLeftRightWrapper is API compatible with LeftRight and uses a
// read-write lock to protect T (data).
template <class T>
class RWSafeLeftRightWrapper final {
 public:
  template <class... Args>
  explicit RWSafeLeftRightWrapper(const Args&... args) : data_{args...} {}
```
- **EN**: It introduces or extends T, RWSafeLeftRightWrapper, which define the main data structures or interfaces for this portion of the file. This chunk defines `RWSafeLeftRightWrapper`, which implements a reusable low-level helper for higher-level runtime code. Template machinery keeps the implementation reusable across scalar types, pointer targets, or backend-specific policies. Synchronization primitives protect shared state and make concurrent updates deterministic enough for runtime use. Container logic organizes metadata, caches, or lookup state so surrounding code can access it efficiently.
- **CN**: 它引入或扩展了 T、RWSafeLeftRightWrapper，这些类型定义了本段涉及的主要数据结构或接口。 这一段定义了 `RWSafeLeftRightWrapper`，其作用是实现供更高层运行时代码复用的底层辅助逻辑。 模板机制让实现能够在标量类型、指针目标或后端策略之间复用。 同步原语用于保护共享状态，并让并发更新在运行时场景下保持足够确定。 容器逻辑用于组织元数据、缓存或查找状态，从而让周边代码能够高效访问。

### Lines 203-216
```cpp
  // RWSafeLeftRightWrapper is not copyable or moveable since LeftRight
  // is not copyable or moveable.
  RWSafeLeftRightWrapper(const RWSafeLeftRightWrapper&) = delete;
  RWSafeLeftRightWrapper(RWSafeLeftRightWrapper&&) noexcept = delete;
  RWSafeLeftRightWrapper& operator=(const RWSafeLeftRightWrapper&) = delete;
  RWSafeLeftRightWrapper& operator=(RWSafeLeftRightWrapper&&) noexcept = delete;
  ~RWSafeLeftRightWrapper() = default;

  template <typename F>
  // NOLINTNEXTLINE(cppcoreguidelines-missing-std-forward)
  auto read(F&& readFunc) const {
    return data_.withLock(
        [&readFunc](T const& data) { return std::forward<F>(readFunc)(data); });
  }
```
- **EN**: This chunk defines `forward<F>`, which implements a reusable low-level helper for higher-level runtime code. Template machinery keeps the implementation reusable across scalar types, pointer targets, or backend-specific policies. The tail returns accumulated results or hands the updated state back to the caller.
- **CN**: 这一段定义了 `forward<F>`，其作用是实现供更高层运行时代码复用的底层辅助逻辑。 模板机制让实现能够在标量类型、指针目标或后端策略之间复用。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 218-229
```cpp
  template <typename F>
  // NOLINTNEXTLINE(cppcoreguidelines-missing-std-forward)
  auto write(F&& writeFunc) {
    return data_.withLock(
        [&writeFunc](T& data) { return std::forward<F>(writeFunc)(data); });
  }

 private:
  c10::Synchronized<T> data_;
};

} // namespace c10
```
- **EN**: This chunk defines `forward<F>`, which implements a reusable low-level helper for higher-level runtime code. Template machinery keeps the implementation reusable across scalar types, pointer targets, or backend-specific policies. The tail returns accumulated results or hands the updated state back to the caller.
- **CN**: 这一段定义了 `forward<F>`，其作用是实现供更高层运行时代码复用的底层辅助逻辑。 模板机制让实现能够在标量类型、指针目标或后端策略之间复用。 末尾会返回累积结果，或把更新后的状态交还给调用方。


## Key Concepts / 关键概念
- **Utility layer**
  - EN: Provides low-level c10 utilities for ownership, containers, hashing, type traits, errors, and portability.
  - CN: 提供 c10 的底层工具能力，包括所有权管理、容器、哈希、类型萃取、错误处理与可移植性支持。
- **IncrementRAII**
  - EN: `IncrementRAII` is one of the dominant symbols declared or implemented in this file.
  - CN: `IncrementRAII` 是本文件声明或实现的关键符号之一。
- **T**
  - EN: `T` is one of the dominant symbols declared or implemented in this file.
  - CN: `T` 是本文件声明或实现的关键符号之一。
- **Error handling**
  - EN: Surfaces invalid states through assertions, exceptions, and diagnostic messages.
  - CN: 通过断言、异常与诊断消息暴露非法状态。
- **Compiler portability**
  - EN: Normalizes compiler, platform, and visibility differences behind shared macros.
  - CN: 通过共享宏统一编译器、平台与可见性差异。

## Dependencies / 依赖关系
- **Internal includes / 内部依赖**: `c10/macros/Macros.h`、`c10/util/Synchronized.h`
- **Third-party includes / 第三方依赖**: 无
- **Standard includes / 标准库依赖**: `array`、`atomic`、`mutex`、`thread`
- **System includes / 系统依赖**: 无
- **Namespaces / 命名空间**: `c10`、`detail`
- **Representative symbols / 代表性符号**: `IncrementRAII`、`T`、`LeftRight`、`F`、`RWSafeLeftRightWrapper`、`fetch_add`、`~IncrementRAII`、`fetch_sub`、`~LeftRight`、`lock`
