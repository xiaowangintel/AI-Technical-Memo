# quick_all_reduce.h — Code Analysis / 代码分析

## Source / 来源
- **File**: `sgl-kernel/csrc/allreduce/quick_all_reduce.h`
- **Repository**: `sgl-project/sglang`
- **Purpose**: Handles multi-GPU all-reduce communication kernels and their supporting runtime logic. This header primarily defines interfaces, templates, constants, and inline helpers shared by compilation units. / 处理多 GPU all-reduce 通信内核及其配套运行时逻辑。 该头文件主要定义编译单元共享的接口、模板、常量和内联辅助逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-21: Templates, aliases, and constants
```cpp
#pragma once

#include <hip/hip_runtime.h>

#include <vector>

#include "quick_all_reduce.cuh"

#define HIP_CHECK(err)                                                                               \
  do {                                                                                               \
    hipError_t err_ = (err);                                                                         \
    if (err_ != hipSuccess) {                                                                        \
      std::printf("HIP error %d at %s:%d. %s\n", err_, __FILE__, __LINE__, hipGetErrorString(err_)); \
      throw std::runtime_error("HIP error");                                                         \
    }                                                                                                \
  } while (0)

namespace quickreduce {
using fptr_t = int64_t;
static_assert(sizeof(void*) == sizeof(fptr_t));
```
**EN:** This section defines `printf`, `runtime_error`, `static_assert`, giving the file reusable compile-time abstractions, aliases, and specialization points.
**CN:** 本段定义了`printf`、`runtime_error`、`static_assert`等内容，为文件提供可复用的编译期抽象、类型别名和特化入口。

### Lines 22-42: Kernel implementation
```cpp
template <typename AllReduceKernel, typename T>
__global__ __quickreduce_launch_bounds_two_shot__ static void allreduce_prototype_twoshot(
    T const* A,
    T* B,
    uint32_t N,
    uint32_t num_blocks,
    int rank,
    uint8_t** dbuffer_list,
    uint32_t data_offset,
    uint32_t flag_color,
    int64_t data_size_per_phase) {
  int block = blockIdx.x;
  int grid = gridDim.x;

  while (block < num_blocks) {
    AllReduceKernel::run(A, B, N, block, rank, dbuffer_list, data_offset, flag_color, data_size_per_phase);
    block += grid;
    flag_color++;
  }
}
```
**EN:** This section implements `allreduce_prototype_twoshot`, `run`, the GPU work units and launch-facing code that carry out the file's core computation.
**CN:** 本段实现了`allreduce_prototype_twoshot`、`run`等 GPU 工作单元及其启动相关代码，承担该文件的核心计算。

### Lines 43-64: Kernel implementation
```cpp
#define TWOSHOT_DISPATCH(__codec)                                         \
  if (world_size == 2) {                                                  \
    using LineCodec = __codec<T, 2>;                                      \
    using AllReduceKernel = AllReduceTwoshot<T, LineCodec, cast_bf2half>; \
    hipLaunchKernelGGL(                                                   \
        (allreduce_prototype_twoshot<AllReduceKernel, T>),                \
        dim3(grid),                                                       \
        dim3(kBlockTwoShot),                                              \
        0,                                                                \
        stream,                                                           \
        A,                                                                \
        B,                                                                \
        N,                                                                \
        num_blocks,                                                       \
        rank,                                                             \
        dbuffer_list,                                                     \
        data_offset,                                                      \
        flag_color,                                                       \
        this->kMaxProblemSize);                                           \
  } else if (world_size == 4) {                                           \
    using LineCodec = __codec<T, 4>;                                      \
    using AllReduceKernel = AllReduceTwoshot<T, LineCodec, cast_bf2half>; \
```
**EN:** This section implements `TWOSHOT_DISPATCH`, `hipLaunchKernelGGL`, `LineCodec`, the GPU work units and launch-facing code that carry out the file's core computation.
**CN:** 本段实现了`TWOSHOT_DISPATCH`、`hipLaunchKernelGGL`、`LineCodec`等 GPU 工作单元及其启动相关代码，承担该文件的核心计算。

### Lines 65-86: Kernel implementation
```cpp
    hipLaunchKernelGGL(                                                   \
        (allreduce_prototype_twoshot<AllReduceKernel, T>),                \
        dim3(grid),                                                       \
        dim3(kBlockTwoShot),                                              \
        0,                                                                \
        stream,                                                           \
        A,                                                                \
        B,                                                                \
        N,                                                                \
        num_blocks,                                                       \
        rank,                                                             \
        dbuffer_list,                                                     \
        data_offset,                                                      \
        flag_color,                                                       \
        this->kMaxProblemSize);                                           \
  } else if (world_size == 8) {                                           \
    using LineCodec = __codec<T, 8>;                                      \
    using AllReduceKernel = AllReduceTwoshot<T, LineCodec, cast_bf2half>; \
    hipLaunchKernelGGL(                                                   \
        (allreduce_prototype_twoshot<AllReduceKernel, T>),                \
        dim3(grid),                                                       \
        dim3(kBlockTwoShot),                                              \
```
**EN:** This section implements `hipLaunchKernelGGL`, `LineCodec`, `AllReduceKernel`, the GPU work units and launch-facing code that carry out the file's core computation.
**CN:** 本段实现了`hipLaunchKernelGGL`、`LineCodec`、`AllReduceKernel`等 GPU 工作单元及其启动相关代码，承担该文件的核心计算。

### Lines 87-110: Types and data layout
```cpp
        0,                                                                \
        stream,                                                           \
        A,                                                                \
        B,                                                                \
        N,                                                                \
        num_blocks,                                                       \
        rank,                                                             \
        dbuffer_list,                                                     \
        data_offset,                                                      \
        flag_color,                                                       \
        this->kMaxProblemSize);                                           \
  }

enum QuickReduceQuantLevel {
  F16 = 0,
  INT8 = 1,
  INT6 = 2,
  INT4 = 3,
};

struct DeviceComms {
  // Max problem size is 2GB (in bytes) or half of uint32_t max value.
  int64_t kMaxProblemSize = static_cast<int64_t>(std::numeric_limits<int32_t>::max()) + 1;
```
**EN:** This section defines `QuickReduceQuantLevel`, `DeviceComms`, packaging state, metadata, or memory layout information that later kernels and wrappers consume.
**CN:** 本段定义了`QuickReduceQuantLevel`、`DeviceComms`等内容，用于封装后续内核与封装层会使用的状态、元数据或内存布局信息。

### Lines 111-130: Templates, aliases, and constants
```cpp
  // Max TP-8
  static int constexpr kMaxWorldSize = 8;

  bool initialized = false;
  uint32_t flag_color = 1;
  int world_size;
  int rank;

  uint8_t* dbuffer;
  uint8_t** dbuffer_list;
  hipIpcMemHandle_t buffer_ipc_handle;
  std::vector<hipIpcMemHandle_t> all_buffer_ipc_handles;
  std::vector<uint8_t*> buffer_list;
  uint32_t data_offset;

  DeviceComms() : initialized(false), world_size(1), rank(0) {}
  ~DeviceComms() {
    destroy();
  }
```
**EN:** This section defines `DeviceComms`, `destroy`, `kMaxWorldSize`, giving the file reusable compile-time abstractions, aliases, and specialization points.
**CN:** 本段定义了`DeviceComms`、`destroy`、`kMaxWorldSize`等内容，为文件提供可复用的编译期抽象、类型别名和特化入口。

### Lines 131-151: Control flow and branching
```cpp
  void init(int world_size, int rank, std::optional<int64_t> max_problem_size = std::nullopt) {
    destroy();
    this->world_size = world_size;
    this->rank = rank;
    if (max_problem_size.has_value() && max_problem_size.value() > 0) {
      this->kMaxProblemSize = max_problem_size.value();
    }
    // Allocate buffer size for worst case: F16 2-stage buffer.
    uint32_t flags_buffer_size = 2 * world_size * kMaxNumBlocks * sizeof(uint32_t);
    static int64_t data_buffer_size = 2 * this->kMaxProblemSize;
    int64_t total_buffer_size = flags_buffer_size + data_buffer_size;
    data_offset = flags_buffer_size;
    HIP_CHECK(hipExtMallocWithFlags((void**)&dbuffer, total_buffer_size, hipDeviceMallocUncached));

    // Clear the flags buffer.
    HIP_CHECK(hipMemset(dbuffer, 0, flags_buffer_size));

    // Device-side list of IPC buffers.
    buffer_list.resize(world_size);
    HIP_CHECK(hipMalloc(&dbuffer_list, world_size * sizeof(uint8_t*)));
```
**EN:** This section drives `init`, `destroy`, `value` through branches and loops, deciding which execution path or data-handling strategy is used.
**CN:** 本段通过分支和循环推进`init`、`destroy`、`value`相关逻辑，决定采用哪条执行路径或数据处理策略。

### Lines 152-176: Control flow and branching
```cpp
    // Create IPC handles for rank's communication buffer.
    all_buffer_ipc_handles.resize(world_size);
    HIP_CHECK(hipIpcGetMemHandle(&buffer_ipc_handle, dbuffer));

    initialized = true;
  }
  int get_world_size() {
    return world_size;
  }
  int get_rank() {
    return rank;
  }
  bool status() {
    return initialized;
  }
  hipIpcMemHandle_t const get_handle() {
    return buffer_ipc_handle;
  }

  void destroy() {
    if (initialized) {
      for (int i = 0; i < world_size; i++) {
        if (i != rank) {
          HIP_CHECK(hipIpcCloseMemHandle(dbuffer_list[i]));
        }
```
**EN:** This section drives `get_world_size`, `get_rank`, `status` through branches and loops, deciding which execution path or data-handling strategy is used.
**CN:** 本段通过分支和循环推进`get_world_size`、`get_rank`、`status`相关逻辑，决定采用哪条执行路径或数据处理策略。

### Lines 177-200: Control flow and branching
```cpp
      }

      HIP_CHECK(hipFree(dbuffer));
      HIP_CHECK(hipFree(dbuffer_list));

      initialized = false;
    }
  }

  void open_ipc_handles(std::vector<hipIpcMemHandle_t> const& ipc_handles) {
    assert(ipc_handles.size() == all_buffer_ipc_handles.size());
    for (int i = 0; i < world_size; i++) {
      all_buffer_ipc_handles[i] = ipc_handles[i];
    }

    // Open device memory access to the IPC communication buffers.
    // Note: For our own rank, we do not need to open a handle.
    for (int i = 0; i < world_size; i++) {
      if (i != rank) {
        HIP_CHECK(
            hipIpcOpenMemHandle((void**)&buffer_list[i], all_buffer_ipc_handles[i], hipIpcMemLazyEnablePeerAccess));
      } else {
        buffer_list[i] = dbuffer;
      }
```
**EN:** This section drives `open_ipc_handles`, `HIP_CHECK`, `assert` through branches and loops, deciding which execution path or data-handling strategy is used.
**CN:** 本段通过分支和循环推进`open_ipc_handles`、`HIP_CHECK`、`assert`相关逻辑，决定采用哪条执行路径或数据处理策略。

### Lines 201-222: Runtime integration and dispatch
```cpp
    }

    HIP_CHECK(hipMemcpy(dbuffer_list, buffer_list.data(), world_size * sizeof(uint8_t*), hipMemcpyHostToDevice));
  }

  template <typename T, bool cast_bf2half>
  void allreduce(T const* A, T* B, uint32_t N, int quant_level, hipStream_t stream) {
    if (world_size != 2 && world_size != 4 && world_size != 8) {
      throw std::runtime_error("All Reduce not supported for world_size = " + std::to_string(world_size));
    }

    // Configuration.
    uint32_t msg_size = N * sizeof(T);
    uint32_t num_blocks = divceil(msg_size, kTileSize);
    uint32_t grid = min(kMaxNumBlocks, num_blocks);
    auto quant_level_ = static_cast<QuickReduceQuantLevel>(quant_level);
    switch (quant_level_) {
      case QuickReduceQuantLevel::INT8:
        TWOSHOT_DISPATCH(CodecQ8)
        break;
      case QuickReduceQuantLevel::INT6:
        TWOSHOT_DISPATCH(CodecQ6)
```
**EN:** This section uses `allreduce`, `HIP_CHECK`, `runtime_error` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`allreduce`、`HIP_CHECK`、`runtime_error`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 223-236: Control flow and branching
```cpp
        break;
      case QuickReduceQuantLevel::INT4:
        TWOSHOT_DISPATCH(CodecQ4)
        break;
      default:
        TWOSHOT_DISPATCH(CodecFP)
        break;
    }
    HIP_CHECK(cudaGetLastError());
    // Rotate the flag color.
    flag_color += divceil(N, grid);
  }
};
```
**EN:** This section drives `HIP_CHECK`, `divceil` through branches and loops, deciding which execution path or data-handling strategy is used.
**CN:** 本段通过分支和循环推进`HIP_CHECK`、`divceil`相关逻辑，决定采用哪条执行路径或数据处理策略。

### Lines 237-237: Local implementation details
```cpp
}  // namespace quickreduce
```
**EN:** This section fills in the local implementation details around the surrounding logic, completing the behavior required by the file.
**CN:** 本段补充了相关逻辑周边的局部实现细节，以完成该文件所需的具体行为。

## Key Concepts / 关键概念
- **GPU kernel programming / GPU 内核编程**: Defines executable device-side work launched from the host. / 定义由主机启动、在设备侧执行的工作。
- **Template specialization / 模板特化**: Relies on C++ templates to specialize code paths by type or shape. / 依赖 C++ 模板按类型或形状特化代码路径。
- **CUDA runtime / CUDA 运行时**: Uses CUDA APIs, streams, or intrinsics for NVIDIA GPUs. / 使用 CUDA API、stream 或 intrinsic 支持 NVIDIA GPU。
- **HIP runtime / HIP 运行时**: Uses HIP APIs or AMD-specific intrinsics for portability. / 使用 HIP API 或 AMD 特定 intrinsic 实现可移植性。
- **Quantization / 量化**: Handles low-precision representations and conversion logic. / 处理低精度表示及其转换逻辑。
- **Collective communication / 集合通信**: Coordinates reductions across devices or ranks. / 在设备或 rank 之间协调归约。

## Dependencies / 依赖关系
- **Internal headers / 内部头文件**: `quick_all_reduce.cuh`
- **External headers / 外部头文件**: `hip/hip_runtime.h`, `vector`
- **Path context / 路径上下文**: allreduce / quick_all_reduce.h
