# interface.cpp — Code Analysis / 代码分析

## Source / 来源
- **File**: `sgl-kernel/csrc/cpu/interface.cpp`
- **Repository**: `sgl-project/sglang`
- **Purpose**: Provides CPU-side reference implementations, preprocessing, or fallback execution paths. This source file primarily implements kernels, host-side wrappers, bindings, or launch logic. / 提供 CPU 侧参考实现、预处理或回退执行路径。 该源文件主要实现内核、主机侧封装、绑定或启动逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-18: Control flow and branching
```cpp
#include <ATen/record_function.h>
#include <torch/all.h>

#include "shm.h"

// Communication settings
static int world_rank = -1;
static int world_size = -1;

static bool is_initialized = false;

static bool all_ranks_local_p = false;

void initialize(int64_t size, int64_t rank) {
  if (is_initialized) {
    return;
  }
```
**EN:** This section drives `initialize` through branches and loops, deciding which execution path or data-handling strategy is used.
**CN:** 本段通过分支和循环推进`initialize`相关逻辑，决定采用哪条执行路径或数据处理策略。

### Lines 19-35: Control flow and branching
```cpp
  // Check whether all ranks is on the same physical machine.
  // If true, we will use an SHM based low latency allreduce

  auto ls_string = std::getenv("LOCAL_SIZE");
  int ls = 0;
  if (ls_string != NULL) {
    ls = std::stoi(std::getenv("LOCAL_SIZE"));
  }

  if (size >= 1 && size == ls) {
    all_ranks_local_p = true;
  }

  world_size = size;
  world_rank = rank;
  is_initialized = true;
```
**EN:** This section drives `getenv`, `stoi` through branches and loops, deciding which execution path or data-handling strategy is used.
**CN:** 本段通过分支和循环推进`getenv`、`stoi`相关逻辑，决定采用哪条执行路径或数据处理策略。

### Lines 36-52: Runtime integration and dispatch
```cpp
  const char* addr_string = std::getenv("MASTER_ADDR");
  if (addr_string == NULL) {
    addr_string = "";
  }
  const char* port_string = std::getenv("MASTER_PORT");
  if (port_string == NULL) {
    port_string = "";
  }

  if (all_ranks_local_p) {
    shm_initialize(size, rank, addr_string, port_string);
  }
}

void shm_allreduce(torch::Tensor& data, int64_t op) {
  TORCH_CHECK(op == c10d::ReduceOp::SUM, "Only torch.distributed.ReduceOp.SUM is supported");
```
**EN:** This section uses `shm_allreduce`, `getenv`, `shm_initialize` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`shm_allreduce`、`getenv`、`shm_initialize`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 53-70: Runtime integration and dispatch
```cpp
  auto numel = data.numel();
  int data_size = numel * data.element_size();
  all_reduce_outer_loop(data, numel, data_size);

  return;
}

torch::Tensor shm_allgather(torch::Tensor& data, int64_t dim) {
  auto numel = data.numel();
  int data_size = numel * data.element_size();
  if (dim < 0) {
    dim += data.dim();
  }
  std::vector<int64_t> result_shape = data.sizes().vec();
  result_shape[dim] *= world_size;
  torch::Tensor result_tensor = torch::empty(result_shape, data.options());
  return all_gather(result_tensor, data, dim, numel, data_size);
}
```
**EN:** This section uses `shm_allgather`, `numel`, `element_size` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`shm_allgather`、`numel`、`element_size`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

## Key Concepts / 关键概念
- **PyTorch extension / PyTorch 扩展**: Bridges low-level kernels with PyTorch tensor/runtime abstractions. / 把底层内核与 PyTorch 张量/运行时抽象连接起来。
- **Collective communication / 集合通信**: Coordinates reductions across devices or ranks. / 在设备或 rank 之间协调归约。

## Dependencies / 依赖关系
- **Internal headers / 内部头文件**: `shm.h`
- **External headers / 外部头文件**: `ATen/record_function.h`, `torch/all.h`
- **Path context / 路径上下文**: cpu / interface.cpp
