# numa.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `c10/util/numa.cpp`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Provides low-level c10 utilities for ownership, containers, hashing, type traits, errors, and portability.
- **Purpose (CN)**: 提供 c10 的底层工具能力，包括所有权管理、容器、哈希、类型萃取、错误处理与可移植性支持。

## Line-by-Line Analysis / 逐行分析
### Lines 1-11
```cpp
#include <c10/util/Exception.h>
#include <c10/util/numa.h>

C10_DEFINE_bool(caffe2_cpu_numa_enabled, false, "Use NUMA whenever possible.")

#if defined(__linux__) && defined(C10_USE_NUMA) && !defined(C10_MOBILE)
#include <numa.h>
#include <numaif.h>
#include <unistd.h>
#define C10_ENABLE_NUMA
#endif
```
- **EN**: This block assembles the compilation dependencies, pulling in local PyTorch/c10 headers such as c10/util/Exception.h, c10/util/numa.h; standard-library headers such as numa.h, numaif.h, unistd.h. The preprocessor guard keeps declarations single-instanced when this header is included transitively. Conditional branches split fast paths, error cases, and special-case invariants. Assertions and failure paths make invalid states fail early instead of silently propagating corruption.
- **CN**: 这一段组织编译依赖，引入了本地 PyTorch/c10 头文件，如 c10/util/Exception.h、c10/util/numa.h；标准库头文件，如 numa.h、numaif.h、unistd.h。 预处理器保护用于避免头文件在传递包含时被重复展开。 条件分支用于区分快速路径、错误场景以及特殊情况不变量。 断言与失败路径让非法状态尽早暴露，而不是悄悄传播错误。

### Lines 13-21
```cpp
// This code used to have a lot of VLOGs. However, because allocation might be
// triggered during static initialization, it's unsafe to invoke VLOG here

namespace c10 {

#ifdef C10_ENABLE_NUMA
bool IsNUMAEnabled() {
  return FLAGS_caffe2_cpu_numa_enabled && numa_available() >= 0;
}
```
- **EN**: The namespace declarations place the code inside c10, matching the surrounding subsystem. This chunk defines `IsNUMAEnabled`, which implements a reusable low-level helper for higher-level runtime code. Preprocessor and exported-macro usage adapts the code to compiler, platform, and visibility constraints. Memory-management logic tracks ownership, requested sizes, reuse opportunities, or allocator configuration. Assertions and failure paths make invalid states fail early instead of silently propagating corruption. The tail returns accumulated results or hands the updated state back to the caller.
- **CN**: 命名空间声明把代码放入 c10 中，与周边子系统保持一致。 这一段定义了 `IsNUMAEnabled`，其作用是实现供更高层运行时代码复用的底层辅助逻辑。 预处理器与导出宏的使用让代码适配编译器、平台以及可见性约束。 内存管理逻辑会跟踪所有权、申请尺寸、复用机会或分配器配置。 断言与失败路径让非法状态尽早暴露，而不是悄悄传播错误。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 23-29
```cpp
void NUMABind(int numa_node_id) {
  if (numa_node_id < 0) {
    return;
  }
  if (!IsNUMAEnabled()) {
    return;
  }
```
- **EN**: This chunk defines `NUMABind`, which implements a reusable low-level helper for higher-level runtime code. Conditional branches split fast paths, error cases, and special-case invariants. The tail returns accumulated results or hands the updated state back to the caller.
- **CN**: 这一段定义了 `NUMABind`，其作用是实现供更高层运行时代码复用的底层辅助逻辑。 条件分支用于区分快速路径、错误场景以及特殊情况不变量。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 31-41
```cpp
  TORCH_CHECK(
      numa_node_id <= numa_max_node(),
      "NUMA node id ",
      numa_node_id,
      " is unavailable");

  auto bm = numa_allocate_nodemask();
  numa_bitmask_setbit(bm, numa_node_id);
  numa_bind(bm);
  numa_bitmask_free(bm);
}
```
- **EN**: This chunk declares `numa_bitmask_free`, which manages allocation, reuse, or release decisions for runtime memory. Preprocessor and exported-macro usage adapts the code to compiler, platform, and visibility constraints. Memory-management logic tracks ownership, requested sizes, reuse opportunities, or allocator configuration. Container logic organizes metadata, caches, or lookup state so surrounding code can access it efficiently. Assertions and failure paths make invalid states fail early instead of silently propagating corruption.
- **CN**: 这一段声明了 `numa_bitmask_free`，其作用是管理运行时内存的分配、复用或释放决策。 预处理器与导出宏的使用让代码适配编译器、平台以及可见性约束。 内存管理逻辑会跟踪所有权、申请尺寸、复用机会或分配器配置。 容器逻辑用于组织元数据、缓存或查找状态，从而让周边代码能够高效访问。 断言与失败路径让非法状态尽早暴露，而不是悄悄传播错误。

### Lines 43-54
```cpp
int GetNUMANode(const void* ptr) {
  if (!IsNUMAEnabled()) {
    return -1;
  }
  AT_ASSERT(ptr);

  int numa_node = -1;
  // NOLINTNEXTLINE(cppcoreguidelines-pro-type-const-cast)
  TORCH_CHECK(
      get_mempolicy(
          &numa_node,
          nullptr,
```
- **EN**: This chunk defines `GetNUMANode`, which implements a reusable low-level helper for higher-level runtime code. Preprocessor and exported-macro usage adapts the code to compiler, platform, and visibility constraints. Conditional branches split fast paths, error cases, and special-case invariants. Assertions and failure paths make invalid states fail early instead of silently propagating corruption. The tail returns accumulated results or hands the updated state back to the caller.
- **CN**: 这一段定义了 `GetNUMANode`，其作用是实现供更高层运行时代码复用的底层辅助逻辑。 预处理器与导出宏的使用让代码适配编译器、平台以及可见性约束。 条件分支用于区分快速路径、错误场景以及特殊情况不变量。 断言与失败路径让非法状态尽早暴露，而不是悄悄传播错误。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 55-66
```cpp
          0,
          const_cast<void*>(ptr),
          MPOL_F_NODE | MPOL_F_ADDR) == 0,
      "Unable to get memory policy, errno:",
      errno);
  return numa_node;
}

int GetNumNUMANodes() {
  if (!IsNUMAEnabled()) {
    return -1;
  }
```
- **EN**: This chunk defines `GetNumNUMANodes`, which implements a reusable low-level helper for higher-level runtime code. Conditional branches split fast paths, error cases, and special-case invariants. The tail returns accumulated results or hands the updated state back to the caller.
- **CN**: 这一段定义了 `GetNumNUMANodes`，其作用是实现供更高层运行时代码复用的底层辅助逻辑。 条件分支用于区分快速路径、错误场景以及特殊情况不变量。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 68-78
```cpp
  return numa_num_configured_nodes();
}

void NUMAMove(void* ptr, size_t size, int numa_node_id) {
  if (numa_node_id < 0) {
    return;
  }
  if (!IsNUMAEnabled()) {
    return;
  }
  AT_ASSERT(ptr);
```
- **EN**: This chunk defines `NUMAMove`, which implements a reusable low-level helper for higher-level runtime code. Conditional branches split fast paths, error cases, and special-case invariants. Assertions and failure paths make invalid states fail early instead of silently propagating corruption. The tail returns accumulated results or hands the updated state back to the caller.
- **CN**: 这一段定义了 `NUMAMove`，其作用是实现供更高层运行时代码复用的底层辅助逻辑。 条件分支用于区分快速路径、错误场景以及特殊情况不变量。 断言与失败路径让非法状态尽早暴露，而不是悄悄传播错误。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 80-91
```cpp
  uintptr_t page_start_ptr =
      ((reinterpret_cast<uintptr_t>(ptr)) & ~(getpagesize() - 1));
  // NOLINTNEXTLINE(*-conversions)
  ptrdiff_t offset = reinterpret_cast<uintptr_t>(ptr) - page_start_ptr;
  // Avoid extra dynamic allocation and NUMA api calls
  AT_ASSERT(
      numa_node_id >= 0 &&
      static_cast<unsigned>(numa_node_id) < sizeof(unsigned long) * 8);
  unsigned long mask = 1UL << numa_node_id;
  // NOLINTNEXTLINE(performance-no-int-to-ptr)
  TORCH_CHECK(
      mbind(
```
- **EN**: This chunk declares `reinterpret_cast<uintptr_t>`, which implements a reusable low-level helper for higher-level runtime code. Preprocessor and exported-macro usage adapts the code to compiler, platform, and visibility constraints. Memory-management logic tracks ownership, requested sizes, reuse opportunities, or allocator configuration. Container logic organizes metadata, caches, or lookup state so surrounding code can access it efficiently. Assertions and failure paths make invalid states fail early instead of silently propagating corruption.
- **CN**: 这一段声明了 `reinterpret_cast<uintptr_t>`，其作用是实现供更高层运行时代码复用的底层辅助逻辑。 预处理器与导出宏的使用让代码适配编译器、平台以及可见性约束。 内存管理逻辑会跟踪所有权、申请尺寸、复用机会或分配器配置。 容器逻辑用于组织元数据、缓存或查找状态，从而让周边代码能够高效访问。 断言与失败路径让非法状态尽早暴露，而不是悄悄传播错误。

### Lines 92-99
```cpp
          reinterpret_cast<void*>(page_start_ptr),
          size + offset,
          MPOL_BIND,
          &mask,
          sizeof(mask) * 8,
          MPOL_MF_MOVE | MPOL_MF_STRICT) == 0,
      "Could not move memory to a NUMA node");
}
```
- **EN**: This chunk continues `reinterpret_cast<uintptr_t>` and expands its control flow, data movement, or edge-case handling. Container logic organizes metadata, caches, or lookup state so surrounding code can access it efficiently.
- **CN**: 这一段延续了 `reinterpret_cast<uintptr_t>`，进一步展开其控制流、数据流转或边界处理逻辑。 容器逻辑用于组织元数据、缓存或查找状态，从而让周边代码能够高效访问。

### Lines 101-110
```cpp
int GetCurrentNUMANode() {
  if (!IsNUMAEnabled()) {
    return -1;
  }

  auto n = numa_node_of_cpu(sched_getcpu());
  return n;
}

#else // C10_ENABLE_NUMA
```
- **EN**: This chunk defines `numa_node_of_cpu`, which implements a reusable low-level helper for higher-level runtime code. Preprocessor and exported-macro usage adapts the code to compiler, platform, and visibility constraints. Conditional branches split fast paths, error cases, and special-case invariants. Assertions and failure paths make invalid states fail early instead of silently propagating corruption. The tail returns accumulated results or hands the updated state back to the caller.
- **CN**: 这一段定义了 `numa_node_of_cpu`，其作用是实现供更高层运行时代码复用的底层辅助逻辑。 预处理器与导出宏的使用让代码适配编译器、平台以及可见性约束。 条件分支用于区分快速路径、错误场景以及特殊情况不变量。 断言与失败路径让非法状态尽早暴露，而不是悄悄传播错误。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 112-120
```cpp
bool IsNUMAEnabled() {
  return false;
}

void NUMABind(int /*numa_node_id*/) {}

int GetNUMANode(const void* /*ptr*/) {
  return -1;
}
```
- **EN**: This chunk defines `GetNUMANode`, which implements a reusable low-level helper for higher-level runtime code. The tail returns accumulated results or hands the updated state back to the caller.
- **CN**: 这一段定义了 `GetNUMANode`，其作用是实现供更高层运行时代码复用的底层辅助逻辑。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 122-132
```cpp
int GetNumNUMANodes() {
  return -1;
}

void NUMAMove(void* /*ptr*/, size_t /*size*/, int /*numa_node_id*/) {}

int GetCurrentNUMANode() {
  return -1;
}

#endif // C10_NUMA_ENABLED
```
- **EN**: This chunk defines `GetCurrentNUMANode`, which implements a reusable low-level helper for higher-level runtime code. Preprocessor and exported-macro usage adapts the code to compiler, platform, and visibility constraints. Assertions and failure paths make invalid states fail early instead of silently propagating corruption. The tail returns accumulated results or hands the updated state back to the caller.
- **CN**: 这一段定义了 `GetCurrentNUMANode`，其作用是实现供更高层运行时代码复用的底层辅助逻辑。 预处理器与导出宏的使用让代码适配编译器、平台以及可见性约束。 断言与失败路径让非法状态尽早暴露，而不是悄悄传播错误。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 134-134
```cpp
} // namespace c10
```
- **EN**: This chunk continues `GetCurrentNUMANode` and expands its control flow, data movement, or edge-case handling.
- **CN**: 这一段延续了 `GetCurrentNUMANode`，进一步展开其控制流、数据流转或边界处理逻辑。


## Key Concepts / 关键概念
- **Utility layer**
  - EN: Provides low-level c10 utilities for ownership, containers, hashing, type traits, errors, and portability.
  - CN: 提供 c10 的底层工具能力，包括所有权管理、容器、哈希、类型萃取、错误处理与可移植性支持。
- **IsNUMAEnabled**
  - EN: `IsNUMAEnabled` is one of the dominant symbols declared or implemented in this file.
  - CN: `IsNUMAEnabled` 是本文件声明或实现的关键符号之一。
- **NUMABind**
  - EN: `NUMABind` is one of the dominant symbols declared or implemented in this file.
  - CN: `NUMABind` 是本文件声明或实现的关键符号之一。
- **Error handling**
  - EN: Surfaces invalid states through assertions, exceptions, and diagnostic messages.
  - CN: 通过断言、异常与诊断消息暴露非法状态。

## Dependencies / 依赖关系
- **Internal includes / 内部依赖**: `c10/util/Exception.h`、`c10/util/numa.h`
- **Third-party includes / 第三方依赖**: 无
- **Standard includes / 标准库依赖**: `numa.h`、`numaif.h`、`unistd.h`
- **System includes / 系统依赖**: 无
- **Namespaces / 命名空间**: `c10`
- **Representative symbols / 代表性符号**: `IsNUMAEnabled`、`NUMABind`、`numa_allocate_nodemask`、`numa_bitmask_setbit`、`numa_bind`、`numa_bitmask_free`、`GetNUMANode`、`GetNumNUMANodes`、`numa_num_configured_nodes`、`NUMAMove`
