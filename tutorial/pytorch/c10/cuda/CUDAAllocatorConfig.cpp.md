# CUDAAllocatorConfig.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `c10/cuda/CUDAAllocatorConfig.cpp`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Implements memory-allocation interfaces, policies, and configuration shared by c10 runtime components.
- **Purpose (CN)**: 实现 c10 运行时组件共享的内存分配接口、策略与配置。

## Line-by-Line Analysis / 逐行分析
### Lines 1-18
```cpp
#include <c10/cuda/CUDAAllocatorConfig.h>
#include <c10/cuda/CUDACachingAllocator.h>

#if !defined(USE_ROCM) && defined(PYTORCH_C10_DRIVER_API_SUPPORTED)
#include <c10/cuda/driver_api.h>
#endif

namespace c10::cuda::CUDACachingAllocator {

size_t CUDAAllocatorConfig::parseAllocatorConfig(
    const c10::CachingAllocator::ConfigTokenizer& tokenizer,
    size_t i,
    bool& used_cudaMallocAsync) {
  // Ease burden on ROCm users by allowing either cuda or hip tokens.
  // cuda token is broken up to prevent hipify matching it.
#define PYTORCH_TOKEN1 \
  "cud"                \
  "aMallocAsync"
```
- **EN**: This block assembles the compilation dependencies, pulling in local PyTorch/c10 headers such as c10/cuda/CUDAAllocatorConfig.h, c10/cuda/CUDACachingAllocator.h, c10/cuda/driver_api.h. The preprocessor guard keeps declarations single-instanced when this header is included transitively. The namespace declarations place the code inside c10::cuda::CUDACachingAllocator, matching the surrounding subsystem. This chunk defines `parseAllocatorConfig`, which manages allocation, reuse, or release decisions for runtime memory. Memory-management logic tracks ownership, requested sizes, reuse opportunities, or allocator configuration. Device- or stream-related details connect this code to backend execution context and asynchronous ordering rules. Conditional branches split fast paths, error cases, and special-case invariants. Assertions and failure paths make invalid states fail early instead of silently propagating corruption.
- **CN**: 这一段组织编译依赖，引入了本地 PyTorch/c10 头文件，如 c10/cuda/CUDAAllocatorConfig.h、c10/cuda/CUDACachingAllocator.h、c10/cuda/driver_api.h。 预处理器保护用于避免头文件在传递包含时被重复展开。 命名空间声明把代码放入 c10::cuda::CUDACachingAllocator 中，与周边子系统保持一致。 这一段定义了 `parseAllocatorConfig`，其作用是管理运行时内存的分配、复用或释放决策。 内存管理逻辑会跟踪所有权、申请尺寸、复用机会或分配器配置。 与设备或流相关的细节把这里的代码连接到后端执行上下文与异步顺序规则。 条件分支用于区分快速路径、错误场景以及特殊情况不变量。 断言与失败路径让非法状态尽早暴露，而不是悄悄传播错误。

### Lines 19-36
```cpp
#define PYTORCH_TOKEN2 "hipMallocAsync"
  tokenizer.checkToken(++i, ":");
  i++; // Move to the value after the colon
#ifdef USE_ROCM
  TORCH_CHECK_VALUE(
      ((tokenizer[i] == "native") || (tokenizer[i] == PYTORCH_TOKEN1) ||
       (tokenizer[i] == PYTORCH_TOKEN2)),
      "Unknown allocator backend, "
      "options are native, " PYTORCH_TOKEN1 ", and " PYTORCH_TOKEN2);
  used_cudaMallocAsync =
      (tokenizer[i] == PYTORCH_TOKEN1 || tokenizer[i] == PYTORCH_TOKEN2);
  TORCH_INTERNAL_ASSERT(
      tokenizer[i] == get()->name() ||
          (tokenizer[i] == PYTORCH_TOKEN1 && get()->name() == PYTORCH_TOKEN2),
      "Allocator backend parsed at runtime != "
      "allocator backend parsed at load time, ",
      tokenizer[i],
      " != ",
```
- **EN**: The preprocessor guard keeps declarations single-instanced when this header is included transitively. This chunk declares `checkToken`, which validates assumptions and reports invalid states early. Preprocessor and exported-macro usage adapts the code to compiler, platform, and visibility constraints. Memory-management logic tracks ownership, requested sizes, reuse opportunities, or allocator configuration. Device- or stream-related details connect this code to backend execution context and asynchronous ordering rules. Assertions and failure paths make invalid states fail early instead of silently propagating corruption.
- **CN**: 预处理器保护用于避免头文件在传递包含时被重复展开。 这一段声明了 `checkToken`，其作用是校验前提条件并尽早报告非法状态。 预处理器与导出宏的使用让代码适配编译器、平台以及可见性约束。 内存管理逻辑会跟踪所有权、申请尺寸、复用机会或分配器配置。 与设备或流相关的细节把这里的代码连接到后端执行上下文与异步顺序规则。 断言与失败路径让非法状态尽早暴露，而不是悄悄传播错误。

### Lines 37-54
```cpp
      get()->name());
#else // USE_ROCM
  TORCH_CHECK_VALUE(
      ((tokenizer[i] == "native") || (tokenizer[i] == PYTORCH_TOKEN1)),
      "Unknown allocator backend, "
      "options are native and " PYTORCH_TOKEN1);
  used_cudaMallocAsync = (tokenizer[i] == PYTORCH_TOKEN1);
  TORCH_INTERNAL_ASSERT(
      tokenizer[i] == get()->name(),
      "Allocator backend parsed at runtime != "
      "allocator backend parsed at load time, ",
      tokenizer[i],
      " != ",
      get()->name());
  if (used_cudaMallocAsync) {
#if CUDA_VERSION >= 11040
    int version = 0;
    C10_CUDA_CHECK(cudaDriverGetVersion(&version));
```
- **EN**: This chunk defines `get`, which implements a focused piece of backend/runtime support logic. Preprocessor and exported-macro usage adapts the code to compiler, platform, and visibility constraints. Memory-management logic tracks ownership, requested sizes, reuse opportunities, or allocator configuration. Device- or stream-related details connect this code to backend execution context and asynchronous ordering rules. Conditional branches split fast paths, error cases, and special-case invariants. Assertions and failure paths make invalid states fail early instead of silently propagating corruption.
- **CN**: 这一段定义了 `get`，其作用是实现一段聚焦的后端/运行时支持逻辑。 预处理器与导出宏的使用让代码适配编译器、平台以及可见性约束。 内存管理逻辑会跟踪所有权、申请尺寸、复用机会或分配器配置。 与设备或流相关的细节把这里的代码连接到后端执行上下文与异步顺序规则。 条件分支用于区分快速路径、错误场景以及特殊情况不变量。 断言与失败路径让非法状态尽早暴露，而不是悄悄传播错误。

### Lines 55-70
```cpp
    TORCH_CHECK(
        version >= 11040,
        "backend:cudaMallocAsync requires CUDA runtime "
        "11.4 or newer, but cudaDriverGetVersion returned ",
        version);
#else // CUDA_VERSION >= 11040
    TORCH_CHECK(
        false,
        "backend:cudaMallocAsync requires PyTorch to be built with "
        "CUDA 11.4 or newer, but CUDA_VERSION is ",
        CUDA_VERSION);
#endif // CUDA_VERSION >= 11040
  }
#endif // USE_ROCM
  return i;
}
```
- **EN**: This chunk continues `get` and expands its control flow, data movement, or edge-case handling. Preprocessor and exported-macro usage adapts the code to compiler, platform, and visibility constraints. Memory-management logic tracks ownership, requested sizes, reuse opportunities, or allocator configuration. Device- or stream-related details connect this code to backend execution context and asynchronous ordering rules. Assertions and failure paths make invalid states fail early instead of silently propagating corruption. The tail returns accumulated results or hands the updated state back to the caller.
- **CN**: 这一段延续了 `get`，进一步展开其控制流、数据流转或边界处理逻辑。 预处理器与导出宏的使用让代码适配编译器、平台以及可见性约束。 内存管理逻辑会跟踪所有权、申请尺寸、复用机会或分配器配置。 与设备或流相关的细节把这里的代码连接到后端执行上下文与异步顺序规则。 断言与失败路径让非法状态尽早暴露，而不是悄悄传播错误。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 72-89
```cpp
void CUDAAllocatorConfig::parseArgs(const std::string& env) {
  bool used_cudaMallocAsync = false;
  bool used_native_specific_option = false;

  c10::CachingAllocator::ConfigTokenizer tokenizer(env);
  for (size_t i = 0; i < tokenizer.size(); i++) {
    const auto& key = tokenizer[i];
    if (key == "backend") {
      i = parseAllocatorConfig(tokenizer, i, used_cudaMallocAsync);
    } else if (
        // ROCm build's hipify step will change "cuda" to "hip", but for ease of
        // use, accept both. We must break up the string to prevent hipify here.
        key == "release_lock_on_hipmalloc" ||
        key ==
            "release_lock_on_c"
            "udamalloc") {
      used_native_specific_option = true;
      tokenizer.checkToken(++i, ":");
```
- **EN**: This chunk defines `checkToken`, which validates assumptions and reports invalid states early. Memory-management logic tracks ownership, requested sizes, reuse opportunities, or allocator configuration. Device- or stream-related details connect this code to backend execution context and asynchronous ordering rules. Looping logic walks collections, device tables, or metadata arrays so the same rule is applied systematically. Conditional branches split fast paths, error cases, and special-case invariants.
- **CN**: 这一段定义了 `checkToken`，其作用是校验前提条件并尽早报告非法状态。 内存管理逻辑会跟踪所有权、申请尺寸、复用机会或分配器配置。 与设备或流相关的细节把这里的代码连接到后端执行上下文与异步顺序规则。 循环逻辑会遍历集合、设备表或元数据数组，从而把同一规则系统地应用到每个元素。 条件分支用于区分快速路径、错误场景以及特殊情况不变量。

### Lines 90-107
```cpp
      m_release_lock_on_cudamalloc = tokenizer.toBool(++i);
    } else if (
        // ROCm build's hipify step will change "cuda" to "hip", but for ease of
        // use, accept both. We must break up the string to prevent hipify here.
        key == "pinned_use_hip_host_register" ||
        key ==
            "pinned_use_c"
            "uda_host_register") {
      i = parsePinnedUseCudaHostRegister(tokenizer, i);
      used_native_specific_option = true;
    } else if (key == "pinned_num_register_threads") {
      i = parsePinnedNumRegisterThreads(tokenizer, i);
      used_native_specific_option = true;
    } else if (key == "pinned_reserve_segment_size_mb") {
      i = parsePinnedReserveSegmentSize(tokenizer, i);
      used_native_specific_option = true;
    } else if (key == "graph_capture_record_stream_reuse") {
      i = parseGraphCaptureRecordStreamReuse(tokenizer, i);
```
- **EN**: This chunk defines `parseGraphCaptureRecordStreamReuse`, which manages device or stream context while preserving execution invariants. Memory-management logic tracks ownership, requested sizes, reuse opportunities, or allocator configuration. Device- or stream-related details connect this code to backend execution context and asynchronous ordering rules. Looping logic walks collections, device tables, or metadata arrays so the same rule is applied systematically. Conditional branches split fast paths, error cases, and special-case invariants.
- **CN**: 这一段定义了 `parseGraphCaptureRecordStreamReuse`，其作用是管理设备或流上下文，同时保持执行不变量。 内存管理逻辑会跟踪所有权、申请尺寸、复用机会或分配器配置。 与设备或流相关的细节把这里的代码连接到后端执行上下文与异步顺序规则。 循环逻辑会遍历集合、设备表或元数据数组，从而把同一规则系统地应用到每个元素。 条件分支用于区分快速路径、错误场景以及特殊情况不变量。

### Lines 108-125
```cpp
      used_native_specific_option = true;
    } else if (key == "per_process_memory_fraction") {
      i = parsePerProcessMemoryFraction(tokenizer, i);
      used_native_specific_option = true;
    } else if (key == "pinned_free_catch_all") {
      i = parsePinnedFreeCatchAll(tokenizer, i);
      used_native_specific_option = true;
    } else if (key == "throw_on_cudamalloc_oom") {
      i = parseThrowOnCudaMallocOom(tokenizer, i);
      used_native_specific_option = true;
    } else {
      const auto& keys =
          c10::CachingAllocator::AcceleratorAllocatorConfig::getKeys();
      TORCH_CHECK_VALUE(
          keys.find(key) != keys.end(),
          "Unrecognized key '",
          key,
          "' in CUDA allocator config.");
```
- **EN**: This chunk defines `getKeys`, which implements a focused piece of backend/runtime support logic. Preprocessor and exported-macro usage adapts the code to compiler, platform, and visibility constraints. Memory-management logic tracks ownership, requested sizes, reuse opportunities, or allocator configuration. Device- or stream-related details connect this code to backend execution context and asynchronous ordering rules. Container logic organizes metadata, caches, or lookup state so surrounding code can access it efficiently. Conditional branches split fast paths, error cases, and special-case invariants. Assertions and failure paths make invalid states fail early instead of silently propagating corruption.
- **CN**: 这一段定义了 `getKeys`，其作用是实现一段聚焦的后端/运行时支持逻辑。 预处理器与导出宏的使用让代码适配编译器、平台以及可见性约束。 内存管理逻辑会跟踪所有权、申请尺寸、复用机会或分配器配置。 与设备或流相关的细节把这里的代码连接到后端执行上下文与异步顺序规则。 容器逻辑用于组织元数据、缓存或查找状态，从而让周边代码能够高效访问。 条件分支用于区分快速路径、错误场景以及特殊情况不变量。 断言与失败路径让非法状态尽早暴露，而不是悄悄传播错误。

### Lines 126-140
```cpp
      // Skip the key and its value
      i = tokenizer.skipKey(i);
    }

    if (i + 1 < tokenizer.size()) {
      tokenizer.checkToken(++i, ",");
    }
  }

  if (used_cudaMallocAsync && used_native_specific_option) {
    TORCH_WARN(
        "backend:cudaMallocAsync ignores max_split_size_mb,"
        "roundup_power2_divisions, and garbage_collect_threshold.");
  }
}
```
- **EN**: This chunk defines `checkToken`, which validates assumptions and reports invalid states early. Preprocessor and exported-macro usage adapts the code to compiler, platform, and visibility constraints. Memory-management logic tracks ownership, requested sizes, reuse opportunities, or allocator configuration. Device- or stream-related details connect this code to backend execution context and asynchronous ordering rules. Conditional branches split fast paths, error cases, and special-case invariants.
- **CN**: 这一段定义了 `checkToken`，其作用是校验前提条件并尽早报告非法状态。 预处理器与导出宏的使用让代码适配编译器、平台以及可见性约束。 内存管理逻辑会跟踪所有权、申请尺寸、复用机会或分配器配置。 与设备或流相关的细节把这里的代码连接到后端执行上下文与异步顺序规则。 条件分支用于区分快速路径、错误场景以及特殊情况不变量。

### Lines 142-156
```cpp
size_t CUDAAllocatorConfig::parsePinnedUseCudaHostRegister(
    const c10::CachingAllocator::ConfigTokenizer& tokenizer,
    size_t i) {
  tokenizer.checkToken(++i, ":");
  m_pinned_use_cuda_host_register = tokenizer.toBool(++i);
  return i;
}

size_t CUDAAllocatorConfig::parseGraphCaptureRecordStreamReuse(
    const c10::CachingAllocator::ConfigTokenizer& tokenizer,
    size_t i) {
  tokenizer.checkToken(++i, ":");
  m_graph_capture_record_stream_reuse = tokenizer.toBool(++i);
  return i;
}
```
- **EN**: This chunk defines `parseGraphCaptureRecordStreamReuse`, which manages device or stream context while preserving execution invariants. Memory-management logic tracks ownership, requested sizes, reuse opportunities, or allocator configuration. Device- or stream-related details connect this code to backend execution context and asynchronous ordering rules. The tail returns accumulated results or hands the updated state back to the caller.
- **CN**: 这一段定义了 `parseGraphCaptureRecordStreamReuse`，其作用是管理设备或流上下文，同时保持执行不变量。 内存管理逻辑会跟踪所有权、申请尺寸、复用机会或分配器配置。 与设备或流相关的细节把这里的代码连接到后端执行上下文与异步顺序规则。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 158-168
```cpp
double CUDAAllocatorConfig::parsePerProcessMemoryFraction(
    const c10::CachingAllocator::ConfigTokenizer& tokenizer,
    size_t i) {
  tokenizer.checkToken(++i, ":");
  double val_env = tokenizer.toDouble(++i);
  TORCH_CHECK_VALUE(
      val_env >= 0.0 && val_env <= 1.0,
      "per_process_memory_fraction is invalid, set it in [0.0, 1.0]");
  m_per_process_memory_fraction = val_env;
  return i;
}
```
- **EN**: This chunk defines `toDouble`, which converts one representation into another form used by nearby runtime code. Preprocessor and exported-macro usage adapts the code to compiler, platform, and visibility constraints. Memory-management logic tracks ownership, requested sizes, reuse opportunities, or allocator configuration. Device- or stream-related details connect this code to backend execution context and asynchronous ordering rules. Container logic organizes metadata, caches, or lookup state so surrounding code can access it efficiently. Assertions and failure paths make invalid states fail early instead of silently propagating corruption. The tail returns accumulated results or hands the updated state back to the caller.
- **CN**: 这一段定义了 `toDouble`，其作用是把一种表示转换为附近运行时代码使用的另一种形式。 预处理器与导出宏的使用让代码适配编译器、平台以及可见性约束。 内存管理逻辑会跟踪所有权、申请尺寸、复用机会或分配器配置。 与设备或流相关的细节把这里的代码连接到后端执行上下文与异步顺序规则。 容器逻辑用于组织元数据、缓存或查找状态，从而让周边代码能够高效访问。 断言与失败路径让非法状态尽早暴露，而不是悄悄传播错误。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 170-187
```cpp
size_t CUDAAllocatorConfig::parsePinnedNumRegisterThreads(
    const c10::CachingAllocator::ConfigTokenizer& tokenizer,
    size_t i) {
  tokenizer.checkToken(++i, ":");
  size_t val2 = tokenizer.toSizeT(++i);
  TORCH_CHECK_VALUE(
      llvm::isPowerOf2_64(val2),
      "Number of register threads has to be power of 2, got ",
      val2);
  auto maxThreads = CUDAAllocatorConfig::pinned_max_register_threads();
  TORCH_CHECK_VALUE(
      val2 <= maxThreads,
      "Number of register threads should be less than or equal to ",
      maxThreads,
      ", got ",
      val2);
  m_pinned_num_register_threads = val2;
  return i;
```
- **EN**: This chunk defines `pinned_max_register_threads`, which implements a focused piece of backend/runtime support logic. Preprocessor and exported-macro usage adapts the code to compiler, platform, and visibility constraints. Memory-management logic tracks ownership, requested sizes, reuse opportunities, or allocator configuration. Device- or stream-related details connect this code to backend execution context and asynchronous ordering rules. Assertions and failure paths make invalid states fail early instead of silently propagating corruption. The tail returns accumulated results or hands the updated state back to the caller.
- **CN**: 这一段定义了 `pinned_max_register_threads`，其作用是实现一段聚焦的后端/运行时支持逻辑。 预处理器与导出宏的使用让代码适配编译器、平台以及可见性约束。 内存管理逻辑会跟踪所有权、申请尺寸、复用机会或分配器配置。 与设备或流相关的细节把这里的代码连接到后端执行上下文与异步顺序规则。 断言与失败路径让非法状态尽早暴露，而不是悄悄传播错误。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 188-199
```cpp
}

size_t CUDAAllocatorConfig::parsePinnedReserveSegmentSize(
    const c10::CachingAllocator::ConfigTokenizer& tokenizer,
    size_t i) {
  tokenizer.checkToken(++i, ":");
  size_t val2 = tokenizer.toSizeT(++i);
  TORCH_CHECK_VALUE(
      val2 > 0, "Pinned reserve segment size has to be greater than 0");
  m_pinned_reserve_segment_size_mb = val2;
  return i;
}
```
- **EN**: This chunk defines `toSizeT`, which converts one representation into another form used by nearby runtime code. Preprocessor and exported-macro usage adapts the code to compiler, platform, and visibility constraints. Memory-management logic tracks ownership, requested sizes, reuse opportunities, or allocator configuration. Device- or stream-related details connect this code to backend execution context and asynchronous ordering rules. Assertions and failure paths make invalid states fail early instead of silently propagating corruption. The tail returns accumulated results or hands the updated state back to the caller.
- **CN**: 这一段定义了 `toSizeT`，其作用是把一种表示转换为附近运行时代码使用的另一种形式。 预处理器与导出宏的使用让代码适配编译器、平台以及可见性约束。 内存管理逻辑会跟踪所有权、申请尺寸、复用机会或分配器配置。 与设备或流相关的细节把这里的代码连接到后端执行上下文与异步顺序规则。 断言与失败路径让非法状态尽早暴露，而不是悄悄传播错误。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 201-218
```cpp
size_t CUDAAllocatorConfig::parsePinnedFreeCatchAll(
    const c10::CachingAllocator::ConfigTokenizer& tokenizer,
    size_t i) {
  tokenizer.checkToken(++i, ":");
  m_pinned_free_catch_all = tokenizer.toBool(++i);
  return i;
}

size_t CUDAAllocatorConfig::parseThrowOnCudaMallocOom(
    const c10::CachingAllocator::ConfigTokenizer& tokenizer,
    size_t i) {
  // Format: throw_on_cudamalloc_oom:true or throw_on_cudamalloc_oom:false
  // When enabled, throws OOM error before calling cudaMalloc if the allocation
  // would likely fail due to insufficient memory.
  tokenizer.checkToken(++i, ":");
  m_throw_on_cudamalloc_oom = tokenizer.toBool(++i);
  return i;
}
```
- **EN**: This chunk defines `parseThrowOnCudaMallocOom`, which manages allocation, reuse, or release decisions for runtime memory. Memory-management logic tracks ownership, requested sizes, reuse opportunities, or allocator configuration. Device- or stream-related details connect this code to backend execution context and asynchronous ordering rules. Container logic organizes metadata, caches, or lookup state so surrounding code can access it efficiently. Conditional branches split fast paths, error cases, and special-case invariants. The tail returns accumulated results or hands the updated state back to the caller.
- **CN**: 这一段定义了 `parseThrowOnCudaMallocOom`，其作用是管理运行时内存的分配、复用或释放决策。 内存管理逻辑会跟踪所有权、申请尺寸、复用机会或分配器配置。 与设备或流相关的细节把这里的代码连接到后端执行上下文与异步顺序规则。 容器逻辑用于组织元数据、缓存或查找状态，从而让周边代码能够高效访问。 条件分支用于区分快速路径、错误场景以及特殊情况不变量。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 220-222
```cpp
REGISTER_ALLOCATOR_CONFIG_PARSE_HOOK(CUDAAllocatorConfig)

} // namespace c10::cuda::CUDACachingAllocator
```
- **EN**: This chunk continues `parseThrowOnCudaMallocOom` and expands its control flow, data movement, or edge-case handling. Memory-management logic tracks ownership, requested sizes, reuse opportunities, or allocator configuration. Device- or stream-related details connect this code to backend execution context and asynchronous ordering rules.
- **CN**: 这一段延续了 `parseThrowOnCudaMallocOom`，进一步展开其控制流、数据流转或边界处理逻辑。 内存管理逻辑会跟踪所有权、申请尺寸、复用机会或分配器配置。 与设备或流相关的细节把这里的代码连接到后端执行上下文与异步顺序规则。


## Key Concepts / 关键概念
- **CUDA runtime support**
  - EN: Implements CUDA-facing c10 runtime support, including allocators, streams, guards, and device helpers.
  - CN: 实现面向 CUDA 的 c10 运行时支持，包括分配器、流、守卫以及设备辅助逻辑。
- **parseAllocatorConfig**
  - EN: `parseAllocatorConfig` is one of the dominant symbols declared or implemented in this file.
  - CN: `parseAllocatorConfig` 是本文件声明或实现的关键符号之一。
- **checkToken**
  - EN: `checkToken` is one of the dominant symbols declared or implemented in this file.
  - CN: `checkToken` 是本文件声明或实现的关键符号之一。
- **Memory allocation**
  - EN: Tracks how raw memory is requested, cached, released, and reported.
  - CN: 跟踪原始内存如何被申请、缓存、释放以及上报。
- **Stream semantics**
  - EN: Coordinates asynchronous execution ordering and per-stream resource usage.
  - CN: 协调异步执行顺序以及按流划分的资源使用。
- **CUDA integration**
  - EN: Connects c10 abstractions to CUDA allocators, streams, and device/runtime APIs.
  - CN: 把 c10 抽象连接到 CUDA 分配器、流以及设备/运行时 API。

## Dependencies / 依赖关系
- **Internal includes / 内部依赖**: `c10/cuda/CUDAAllocatorConfig.h`、`c10/cuda/CUDACachingAllocator.h`、`c10/cuda/driver_api.h`
- **Third-party includes / 第三方依赖**: 无
- **Standard includes / 标准库依赖**: 无
- **System includes / 系统依赖**: 无
- **Namespaces / 命名空间**: `c10::cuda::CUDACachingAllocator`
- **Representative symbols / 代表性符号**: `parseAllocatorConfig`、`checkToken`、`parseArgs`、`tokenizer`、`size`、`toBool`、`parsePinnedUseCudaHostRegister`、`parsePinnedNumRegisterThreads`、`parsePinnedReserveSegmentSize`、`parseGraphCaptureRecordStreamReuse`
