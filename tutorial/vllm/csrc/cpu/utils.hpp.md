# utils.hpp — Code Analysis / 代码分析

## Source / 来源
- **File**: `csrc/cpu/utils.hpp`
- **Repository**: vllm-project/vllm
- **Purpose**: Defines shared CPU helper utilities used across kernels. / 定义跨 CPU 内核复用的通用辅助工具。

## Line-by-Line Analysis / 逐行分析
### Preamble and includes (lines 1-8)
```cpp
#ifndef UTILS_HPP
#define UTILS_HPP

#include <atomic>
#include <unistd.h>
#include <ATen/cpu/Utils.h>

#include "cpu/cpu_types.hpp"
```
**EN:** This opening block pulls in the headers, feature macros, and compile-time aliases that the rest of the file depends on.
**CN:** 这一开头代码块引入了后续实现依赖的头文件、特性宏以及编译期别名。

### Function / Kernel: get_isa (lines 13-21)
```cpp
inline ISA get_isa(const std::string& isa) {
  if (isa == "amx") {
    return ISA::AMX;
  } else if (isa == "vec") {
    return ISA::VEC;
  } else {
    TORCH_CHECK(false, "Invalid isa type: " + isa);
  }
}
```
**EN:** This function provides a reusable piece of the file’s main compute pipeline, usually handling validation, indexing, or a fused math step.
**CN:** 该函数是本文件主计算流水线中的一个可复用环节，通常负责校验、索引计算或融合数学步骤。

### Struct: Counter (lines 45-54)
```cpp
struct Counter {
  std::atomic<int64_t> counter;
  char _padding[56];

  Counter() : counter(0) {}

  void reset_counter() { counter.store(0); }

  int64_t acquire_counter() { return counter++; }
};
```
**EN:** This type defines shared state, data layout, or a reusable helper abstraction for the rest of the file.
**CN:** 该类型定义了共享状态、数据布局或供本文件其他部分复用的辅助抽象。

### Function / Kernel: get_available_l2_size (lines 56-85)
```cpp
inline int64_t get_available_l2_size() {
#if defined(__s390x__) || defined(__powerpc__)
  static int64_t size = []() {
    uint32_t l2_cache_size = 0;
    auto caps = at::cpu::get_cpu_capabilities();
    auto it = caps.find("l2_cache_size");
    if (it != caps.end()) {
      l2_cache_size = static_cast<uint32_t>(it->second.toInt());
    }
    if (l2_cache_size == 0) {
      long sys_l2 = sysconf(_SC_LEVEL2_CACHE_SIZE);
      if (sys_l2 > 0) {
        l2_cache_size = static_cast<uint32_t>(sys_l2);
      }
// ...
  return size;
#endif
}
```
**EN:** This helper converts data between storage formats while preserving the layout assumptions expected by later compute kernels.
**CN:** 该辅助函数在不同存储格式之间进行转换，同时保持后续计算内核所需的数据布局假设。

### Function / Kernel: round_up (lines 87-91)
```cpp
template <int32_t alignment_v, typename T>
inline T round_up(T size) {
  T alignment = alignment_v;
  return (((size + alignment - 1) / alignment) * alignment);
}
```
**EN:** This function provides a reusable piece of the file’s main compute pipeline, usually handling validation, indexing, or a fused math step.
**CN:** 该函数是本文件主计算流水线中的一个可复用环节，通常负责校验、索引计算或融合数学步骤。

### Function / Kernel: round_down (lines 93-97)
```cpp
template <int32_t alignment_v, typename T>
inline T round_down(T size) {
  T alignment = alignment_v;
  return (size / alignment) * alignment;
}
```
**EN:** This function provides a reusable piece of the file’s main compute pipeline, usually handling validation, indexing, or a fused math step.
**CN:** 该函数是本文件主计算流水线中的一个可复用环节，通常负责校验、索引计算或融合数学步骤。

### Function / Kernel: print_logits (lines 99-114)
```cpp
template <typename T>
inline void print_logits(const char* name, T* ptr, int32_t row, int32_t col,
                         int32_t stride) {
  std::stringstream ss;
  ss << std::fixed << std::setprecision(5) << name << ": [\n";
  auto* curr_logits_buffer = ptr;
  for (int32_t m = 0; m < row; ++m) {
    for (int32_t n = 0; n < col; ++n) {
      ss << curr_logits_buffer[n] << ", ";
    }
    ss << "\n";
    curr_logits_buffer += stride;
  }
  ss << "]\n";
  std::printf("%s", ss.str().c_str());
}
```
**EN:** This function provides a reusable piece of the file’s main compute pipeline, usually handling validation, indexing, or a fused math step.
**CN:** 该函数是本文件主计算流水线中的一个可复用环节，通常负责校验、索引计算或融合数学步骤。

### Class: ScratchPadManager (lines 116-138)
```cpp
class ScratchPadManager {
 public:
  static constexpr size_t allocation_unit = 4 * 1024;  // 4KB

  static ScratchPadManager* get_scratchpad_manager();

  ScratchPadManager();

  template <typename T>
  T* get_data() {
    return reinterpret_cast<T*>(ptr_);
  }

  static size_t round(size_t size) {
// ...
  size_t size_;
  void* ptr_;
};
```
**EN:** This type defines shared state, data layout, or a reusable helper abstraction for the rest of the file.
**CN:** 该类型定义了共享状态、数据布局或供本文件其他部分复用的辅助抽象。

## Key Concepts / 关键概念
- Intel AMX tiling / Intel AMX 分块
- SIMD vectorization / SIMD 向量化
- Template-based specialization / 基于模板的专用化

## Dependencies / 依赖关系
- **Internal headers / 内部头文件**: `cpu/cpu_types.hpp`
- **External libraries / 外部库**: C++ standard library or platform support, PyTorch / ATen
- **Runtime coupling / 运行时耦合**: Uses templates to specialize code paths at compile time / 通过模板在编译期专用化代码路径
