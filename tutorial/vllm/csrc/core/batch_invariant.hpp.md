# batch_invariant.hpp — Code Analysis / 代码分析

## Source / 来源
- **File**: `csrc/core/batch_invariant.hpp`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Expose a tiny helper that checks whether the `VLLM_BATCH_INVARIANT` environment variable is enabled, caching the result for the lifetime of the process. / [CN] 提供一个很小的辅助函数，用于检查 `VLLM_BATCH_INVARIANT` 环境变量是否开启，并在进程生命周期内缓存结果。

## Line-by-Line Analysis / 逐行分析
### Includes and namespace / 头文件与命名空间
```cpp
#pragma once
#include <cstdlib>
#include <string>

namespace vllm {
```
**EN:** The header is intentionally lightweight: it only needs C standard library access for `std::getenv` / `std::atoi` and `std::string` for the environment-variable key.
**CN:** 这个头文件刻意保持轻量：它只需要 C 标准库来使用 `std::getenv` / `std::atoi`，以及 `std::string` 来表示环境变量名。

### Cached environment lookup / 缓存式环境变量读取
```cpp
inline bool vllm_is_batch_invariant() {
  static bool cached = []() {
    std::string env_key = "VLLM_BATCH_INVARIANT";
    const char* val = std::getenv(env_key.c_str());
    return (val && std::atoi(val) != 0) ? 1 : 0;
  }();
  return cached;
}
```
**EN:** The function reads the environment variable only once by storing the result in a local `static` initialized by a lambda. Any non-zero string accepted by `std::atoi` enables the flag; missing or zero-valued variables disable it.
**CN:** 这个函数通过 lambda 初始化的局部 `static` 变量实现“只读取一次”环境变量。只要 `std::atoi` 解析出的值非零，就视为开启；变量缺失或值为 0 则视为关闭。

## Key Concepts / 关键概念
- **EN:** Function-local statics are used as a one-time cache.  
  **CN:** 函数内静态变量被用作一次性缓存。
- **EN:** The helper converts a process-wide environment knob into a cheap branchable boolean.  
  **CN:** 这个辅助函数把进程级环境开关转换成可低成本分支使用的布尔值。
- **EN:** Because the value is cached, changing the environment after first call has no effect inside the running process.  
  **CN:** 由于结果被缓存，首次调用之后再修改环境变量不会影响当前进程内的返回值。

## Dependencies / 依赖关系
- **EN:** `<cstdlib>` supplies `std::getenv` and `std::atoi`.  
  **CN:** `<cstdlib>` 提供 `std::getenv` 和 `std::atoi`。
- **EN:** `<string>` is used to hold the environment-variable name before querying it.  
  **CN:** `<string>` 用于在查询前保存环境变量名。
