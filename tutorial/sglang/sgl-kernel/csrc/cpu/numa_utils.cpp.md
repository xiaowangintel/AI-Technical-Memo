# numa_utils.cpp — Code Analysis / 代码分析

## Source / 来源
- **File**: `sgl-kernel/csrc/cpu/numa_utils.cpp`
- **Repository**: `sgl-project/sglang`
- **Purpose**: Provides CPU-side reference implementations, preprocessing, or fallback execution paths. This source file primarily implements kernels, host-side wrappers, bindings, or launch logic. / 提供 CPU 侧参考实现、预处理或回退执行路径。 该源文件主要实现内核、主机侧封装、绑定或启动逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-18: Headers and compile-time setup
```cpp
#include <numa.h>
#include <sched.h>
#include <sys/syscall.h>
#include <sys/types.h>
#include <unistd.h>

#include <string>

#include "common.h"

std::string init_cpu_threads_env(const std::string& cpu_ids) {
  bitmask* omp_cpu_mask = numa_parse_cpustring(cpu_ids.c_str());
  TORCH_CHECK(omp_cpu_mask->size > 0);
  std::vector<int> omp_cpu_ids;
  omp_cpu_ids.reserve(omp_cpu_mask->size);

  constexpr int group_size = 8 * sizeof(*omp_cpu_mask->maskp);
```
**EN:** This section prepares the file's headers, macros, and compile-time switches so later code can rely on the right platform and API definitions.
**CN:** 本段准备文件依赖的头文件、宏和编译期开关，使后续代码能够依赖正确的平台与 API 定义。

### Lines 19-38: Runtime integration and dispatch
```cpp
  for (int offset = 0; offset < omp_cpu_mask->size; offset += group_size) {
    unsigned long group_mask = omp_cpu_mask->maskp[offset / group_size];
    int i = 0;
    while (group_mask) {
      if (group_mask & 1) {
        omp_cpu_ids.emplace_back(offset + i);
      }
      ++i;
      group_mask >>= 1;
    }
  }

  // Memory node binding
  if (numa_available() != -1) {
    TORCH_CHECK(!omp_cpu_ids.empty(), "Cannot bind memory, no CPUs specified.");
    int mem_node_id_st = numa_node_of_cpu(omp_cpu_ids.front());
    int mem_node_id_ed = numa_node_of_cpu(omp_cpu_ids.back());
    if (mem_node_id_st > mem_node_id_ed) {
      std::swap(mem_node_id_st, mem_node_id_ed);
    }
```
**EN:** This section uses `emplace_back`, `TORCH_CHECK`, `numa_node_of_cpu` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`emplace_back`、`TORCH_CHECK`、`numa_node_of_cpu`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 39-56: Runtime integration and dispatch
```cpp

    bitmask* mask =
        numa_parse_nodestring((std::to_string(mem_node_id_st) + "-" + std::to_string(mem_node_id_ed)).c_str());
    bitmask* src_mask = numa_get_membind();

    int pid = getpid();

    // move all existing pages to the specified numa node.
    *(src_mask->maskp) = *(src_mask->maskp) ^ *(mask->maskp);
    int page_num = numa_migrate_pages(pid, src_mask, mask);
    if (page_num == -1) {
      TORCH_WARN(false, "numa_migrate_pages failed. errno: " + std::to_string(errno));
    }

    // restrict memory allocation node.
    numa_set_membind(mask);
    numa_set_strict(1);
  }
```
**EN:** This section uses `numa_parse_nodestring`, `numa_get_membind`, `getpid` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`numa_parse_nodestring`、`numa_get_membind`、`getpid`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 57-77: Runtime integration and dispatch
```cpp

  // OMP threads binding
  omp_set_num_threads((int)omp_cpu_ids.size());
  at::set_num_threads((int)omp_cpu_ids.size());
  TORCH_CHECK_EQ(omp_cpu_ids.size(), at::get_num_threads());
  TORCH_CHECK_EQ(omp_cpu_ids.size(), omp_get_max_threads());

  std::vector<std::pair<int, int>> thread_core_mapping;
  thread_core_mapping.reserve(omp_cpu_ids.size());
  omp_lock_t writelock;
  omp_init_lock(&writelock);

#pragma omp parallel for schedule(static, 1)
  for (size_t i = 0; i < omp_cpu_ids.size(); ++i) {
    cpu_set_t mask;
    CPU_ZERO(&mask);
    CPU_SET(omp_cpu_ids[i], &mask);
    int ret = sched_setaffinity(0, sizeof(cpu_set_t), &mask);
    if (ret == -1) {
      TORCH_CHECK(false, "sched_setaffinity failed. errno: " + std::to_string(errno));
    }
```
**EN:** This section uses `omp_set_num_threads`, `set_num_threads`, `TORCH_CHECK_EQ` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`omp_set_num_threads`、`set_num_threads`、`TORCH_CHECK_EQ`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 78-95: Control flow and branching
```cpp

    omp_set_lock(&writelock);
    thread_core_mapping.emplace_back(syscall(SYS_gettid), omp_cpu_ids[i]);
    omp_unset_lock(&writelock);
  }

  omp_destroy_lock(&writelock);

  numa_free_nodemask(omp_cpu_mask);

  std::stringstream ss;
  ss << "OMP threads binding of Process " << getpid() << ":\n";
  std::sort(
      thread_core_mapping.begin(), thread_core_mapping.end(), [](auto&& a, auto&& b) { return a.second < b.second; });
  for (auto&& item : thread_core_mapping) {
    ss << "\t"
       << "OMP tid: " << item.first << ", core " << item.second << "\n";
  }
```
**EN:** This section drives `std::sort`, `omp_set_lock`, `emplace_back` through branches and loops, deciding which execution path or data-handling strategy is used.
**CN:** 本段通过分支和循环推进`std::sort`、`omp_set_lock`、`emplace_back`相关逻辑，决定采用哪条执行路径或数据处理策略。

### Lines 96-98: Local implementation details
```cpp

  return ss.str();
}
```
**EN:** This section fills in the local implementation details around `str`, completing the behavior required by the file.
**CN:** 本段补充了`str`周边的局部实现细节，以完成该文件所需的具体行为。

## Key Concepts / 关键概念
- **Low-level systems code / 底层系统代码**: Focuses on performance-sensitive implementation details close to the hardware. / 关注贴近硬件、对性能敏感的实现细节。

## Dependencies / 依赖关系
- **Internal headers / 内部头文件**: `common.h`
- **External headers / 外部头文件**: `numa.h`, `sched.h`, `sys/syscall.h`, `sys/types.h`, `unistd.h`, `string`
- **Path context / 路径上下文**: cpu / numa_utils.cpp
