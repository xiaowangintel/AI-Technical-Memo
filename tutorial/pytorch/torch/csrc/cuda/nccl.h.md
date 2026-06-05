# nccl.h — Code Analysis / 代码分析

## Source / 来源

- **File**: `torch/csrc/cuda/nccl.h`
- **Repository**: `pytorch/pytorch`
- **Purpose**: This file declares interfaces for `nccl.h` inside the CUDA backend bindings, memory helpers, streams, events, and communication utilities, with emphasis on nccl integration, cuda backend integration. / 该文件在CUDA 后端绑定、内存辅助逻辑、流、事件与通信工具中针对 `nccl.h` 声明接口，重点涉及NCCL 集成、CUDA 后端集成。

## Line-by-Line Analysis / 逐行分析

### Lines 1-2: Preprocessor configuration / 预处理配置
```cpp
#pragma once

```
- **EN**: Applies compile-time gating, macros, or platform-specific configuration so the file builds correctly under different feature sets.
- **CN**: 应用编译期开关、宏或平台特定配置，使该文件能在不同功能组合下正确构建。

### Lines 3-5: Header dependencies / 头文件依赖
```cpp
#include <ATen/ATen.h>
#include <ATen/cuda/CUDAContext.h>

```
- **EN**: Pulls in the headers required by this translation unit, including interfaces from the CUDA backend bindings, memory helpers, streams, events, and communication utilities.
- **CN**: 引入该翻译单元所需的头文件，包括来自CUDA 后端绑定、内存辅助逻辑、流、事件与通信工具的接口。

### Lines 6-9: Header dependencies / 头文件依赖
```cpp
#include <cstddef>
#include <optional>
#include <vector>

```
- **EN**: Pulls in the headers required by this translation unit, including interfaces from the CUDA backend bindings, memory helpers, streams, events, and communication utilities.
- **CN**: 引入该翻译单元所需的头文件，包括来自CUDA 后端绑定、内存辅助逻辑、流、事件与通信工具的接口。

### Lines 10-20: Supporting statements / 辅助语句
```cpp
// NCCL BFloat16 is enabled only for CUDA 11+ and NCCL versions 2.10+, or for
// HIP 3.1+
#if defined(__CUDA_BF16_TYPES_EXIST__)
#define HAS_NCCL_BF16_DATATYPE \
  ((NCCL_MAJOR > 2) || (NCCL_MAJOR == 2) && (NCCL_MINOR >= 10))
#elif defined(USE_ROCM) && (TORCH_HIP_VERSION >= 301)
#define HAS_NCCL_BF16_DATATYPE 1
#else
#define HAS_NCCL_BF16_DATATYPE 0
#endif

```
- **EN**: Contains local statements, constants, or helper glue that support the surrounding control flow and data movement.
- **CN**: 包含局部语句、常量或辅助胶水代码，用于支撑周围的控制流与数据传递。

### Lines 21-22: Namespace scope / 命名空间作用域
```cpp
namespace torch::cuda::nccl {

```
- **EN**: Opens or organizes namespace scopes so related symbols live under the expected PyTorch component hierarchy.
- **CN**: 打开或组织命名空间作用域，使相关符号位于预期的 PyTorch 组件层级中。

### Lines 23-26: Comments and documentation / 注释与文档
```cpp
/* The following are copied from <nccl.h> and redefined in torch::cuda::nccl
 * namespace */
/* pytorch should only use the following definition within pytorch scope */

```
- **EN**: Documents licensing terms, invariants, or developer intent that contextualize the surrounding implementation.
- **CN**: 说明许可证条款、不变量或开发者意图，为周围实现提供上下文。

### Lines 27-30: Supporting statements / 辅助语句
```cpp
/* Opaque handle to communicator to ncclComm*, this will reinterpret as ncclComm
 * in nccl.cpp */
typedef void* ncclComm_t;

```
- **EN**: Contains local statements, constants, or helper glue that support the surrounding control flow and data movement.
- **CN**: 包含局部语句、常量或辅助胶水代码，用于支撑周围的控制流与数据传递。

### Lines 31-38: Supporting statements / 辅助语句
```cpp
/** redefine nccl unique ID in torch scope. this should be identical to native
 * nccl impp. */
#define NCCL_UNIQUE_ID_BYTES 128
typedef struct {
  // NOLINTNEXTLINE(*array*)
  char internal[NCCL_UNIQUE_ID_BYTES];
} ncclUniqueId;

```
- **EN**: Contains local statements, constants, or helper glue that support the surrounding control flow and data movement.
- **CN**: 包含局部语句、常量或辅助胶水代码，用于支撑周围的控制流与数据传递。

### Lines 39-51: Supporting statements / 辅助语句
```cpp
/* Error type */
enum class ncclResult {
  Success = 0,
  UnhandledCudaError = 1,
  SystemError = 2,
  InternalError = 3,
  InvalidArgument = 4,
  InvalidUsage = 5,
  RemoteError = 6,
  InProgress = 7,
  NumResults = 8
};

```
- **EN**: Contains local statements, constants, or helper glue that support the surrounding control flow and data movement.
- **CN**: 包含局部语句、常量或辅助胶水代码，用于支撑周围的控制流与数据传递。

### Lines 52-54: Supporting statements / 辅助语句
```cpp
/* Reduction operation selector */
enum class ncclRedOp { Sum = 0, Prod = 1, Max = 2, Min = 3, NumOps = 4 };

```
- **EN**: Contains local statements, constants, or helper glue that support the surrounding control flow and data movement.
- **CN**: 包含局部语句、常量或辅助胶水代码，用于支撑周围的控制流与数据传递。

### Lines 55-74: Supporting statements / 辅助语句
```cpp
/* Data types */
enum class ncclDataType {
  Int8 = 0,
  Char = 0,
  Uint8 = 1,
  Int32 = 2,
  Int = 2,
  Uint32 = 3,
  Int64 = 4,
  Uint64 = 5,
  Float16 = 6,
  Half = 6,
  Float32 = 7,
  Float = 7,
  Float64 = 8,
  Double = 8,
  Bfloat16 = 9,
  NumTypes = 10
};

```
- **EN**: Contains local statements, constants, or helper glue that support the surrounding control flow and data movement.
- **CN**: 包含局部语句、常量或辅助胶水代码，用于支撑周围的控制流与数据传递。

### Lines 75-85: Supporting statements / 辅助语句
```cpp
// RAII helper class to manage NCCL group API and CUDA free mutex.
// The destructor is allowed to throw since this helper class only
// manages group and lock lifetimes.
struct TORCH_CUDA_CPP_API AutoNcclGroup {
  AutoNcclGroup();
  AutoNcclGroup(ncclComm_t comm, bool comm_nonblocking);
  ~AutoNcclGroup() noexcept(false);
  ncclComm_t comm_;
  bool comm_nonblocking_;
};

```
- **EN**: Contains local statements, constants, or helper glue that support the surrounding control flow and data movement.
- **CN**: 包含局部语句、常量或辅助胶水代码，用于支撑周围的控制流与数据传递。

### Lines 86-89: Supporting statements / 辅助语句
```cpp
// NOTE: this is exposed only so that python_nccl.cpp can some of these helpers.
// Don't use them outside of these files.
namespace detail {

```
- **EN**: Contains local statements, constants, or helper glue that support the surrounding control flow and data movement.
- **CN**: 包含局部语句、常量或辅助胶水代码，用于支撑周围的控制流与数据传递。

### Lines 90-91: Supporting statements / 辅助语句
```cpp
TORCH_CUDA_CPP_API void throw_nccl_error(ncclResult status);

```
- **EN**: Contains local statements, constants, or helper glue that support the surrounding control flow and data movement.
- **CN**: 包含局部语句、常量或辅助胶水代码，用于支撑周围的控制流与数据传递。

### Lines 92-97: Function `NCCL_CHECK` / 函数 `NCCL_CHECK`
```cpp
inline void NCCL_CHECK(ncclResult status) {
  if (status != ncclResult::Success) {
    throw_nccl_error(status);
  }
}

```
- **EN**: Implements `NCCL_CHECK`, one of the operational units in this file for the CUDA backend bindings, memory helpers, streams, events, and communication utilities.
- **CN**: 实现 `NCCL_CHECK`，它是该文件中服务于CUDA 后端绑定、内存辅助逻辑、流、事件与通信工具的一个运行单元。

### Lines 98-111: Supporting statements / 辅助语句
```cpp
TORCH_CUDA_CPP_API at::ArrayRef<ncclComm_t> get_communicators(
    at::TensorList inputs);
TORCH_CUDA_CPP_API void check_inputs(
    at::TensorList inputs,
    at::TensorList outputs,
    size_t input_multiplier,
    size_t output_multiplier);
TORCH_CUDA_CPP_API void check_inputs(
    at::TensorList inputs,
    const at::Tensor& output,
    int root,
    size_t input_multiplier,
    size_t output_multiplier);

```
- **EN**: Contains local statements, constants, or helper glue that support the surrounding control flow and data movement.
- **CN**: 包含局部语句、常量或辅助胶水代码，用于支撑周围的控制流与数据传递。

### Lines 112-113: Supporting statements / 辅助语句
```cpp
} // namespace detail

```
- **EN**: Contains local statements, constants, or helper glue that support the surrounding control flow and data movement.
- **CN**: 包含局部语句、常量或辅助胶水代码，用于支撑周围的控制流与数据传递。

### Lines 114-116: Using declarations / using 声明
```cpp
using comm_list = std::vector<ncclComm_t>;
using stream_list = std::vector<std::optional<at::cuda::CUDAStream>>;

```
- **EN**: Introduces aliases or selected names to simplify later references inside this file.
- **CN**: 引入别名或选定名称，以简化文件后续代码中的引用。

### Lines 117-119: Supporting statements / 辅助语句
```cpp
TORCH_CUDA_CPP_API std::uint64_t version();
TORCH_CUDA_CPP_API const char* version_suffix();

```
- **EN**: Contains local statements, constants, or helper glue that support the surrounding control flow and data movement.
- **CN**: 包含局部语句、常量或辅助胶水代码，用于支撑周围的控制流与数据传递。

### Lines 120-121: Supporting statements / 辅助语句
```cpp
bool is_available(at::TensorList tensors);

```
- **EN**: Contains local statements, constants, or helper glue that support the surrounding control flow and data movement.
- **CN**: 包含局部语句、常量或辅助胶水代码，用于支撑周围的控制流与数据传递。

### Lines 122-126: Supporting statements / 辅助语句
```cpp
TORCH_CUDA_CPP_API void get_unique_id(ncclUniqueId& id);
TORCH_CUDA_CPP_API ncclComm_t
comm_init_rank(int nranks, const ncclUniqueId& comm_id, int rank);
TORCH_CUDA_CPP_API void comm_destroy(ncclComm_t comm);

```
- **EN**: Contains local statements, constants, or helper glue that support the surrounding control flow and data movement.
- **CN**: 包含局部语句、常量或辅助胶水代码，用于支撑周围的控制流与数据传递。

### Lines 127-131: Supporting statements / 辅助语句
```cpp
TORCH_CUDA_CPP_API void broadcast(
    at::TensorList tensors,
    const stream_list& streams = {},
    const comm_list& user_comms = {});

```
- **EN**: Contains local statements, constants, or helper glue that support the surrounding control flow and data movement.
- **CN**: 包含局部语句、常量或辅助胶水代码，用于支撑周围的控制流与数据传递。

### Lines 132-133: Supporting statements / 辅助语句
```cpp
size_t get_max_count();

```
- **EN**: Contains local statements, constants, or helper glue that support the surrounding control flow and data movement.
- **CN**: 包含局部语句、常量或辅助胶水代码，用于支撑周围的控制流与数据传递。

### Lines 134-141: Supporting statements / 辅助语句
```cpp
TORCH_CUDA_CPP_API void reduce(
    const std::vector<at::Tensor>& inputs,
    at::Tensor& output,
    int32_t root = 0,
    int32_t op = static_cast<int>(ncclRedOp::Sum),
    const stream_list& streams = {},
    const comm_list& user_comms = {});

```
- **EN**: Contains local statements, constants, or helper glue that support the surrounding control flow and data movement.
- **CN**: 包含局部语句、常量或辅助胶水代码，用于支撑周围的控制流与数据传递。

### Lines 142-148: Supporting statements / 辅助语句
```cpp
TORCH_CUDA_CPP_API void reduce(
    std::vector<at::Tensor>& inputs,
    int32_t root = 0,
    int32_t op = static_cast<int>(ncclRedOp::Sum),
    const stream_list& streams = {},
    const comm_list& user_comms = {});

```
- **EN**: Contains local statements, constants, or helper glue that support the surrounding control flow and data movement.
- **CN**: 包含局部语句、常量或辅助胶水代码，用于支撑周围的控制流与数据传递。

### Lines 149-155: Supporting statements / 辅助语句
```cpp
TORCH_CUDA_CPP_API void all_reduce(
    const std::vector<at::Tensor>& inputs,
    std::vector<at::Tensor>& outputs,
    int32_t op = static_cast<int>(ncclRedOp::Sum),
    const stream_list& streams = {},
    const comm_list& user_comms = {});

```
- **EN**: Contains local statements, constants, or helper glue that support the surrounding control flow and data movement.
- **CN**: 包含局部语句、常量或辅助胶水代码，用于支撑周围的控制流与数据传递。

### Lines 156-162: Supporting statements / 辅助语句
```cpp
TORCH_CUDA_CPP_API void reduce_scatter(
    const std::vector<at::Tensor>& inputs,
    std::vector<at::Tensor>& outputs,
    int32_t op = static_cast<int>(ncclRedOp::Sum),
    const stream_list& streams = {},
    const comm_list& user_comms = {});

```
- **EN**: Contains local statements, constants, or helper glue that support the surrounding control flow and data movement.
- **CN**: 包含局部语句、常量或辅助胶水代码，用于支撑周围的控制流与数据传递。

### Lines 163-169: Supporting statements / 辅助语句
```cpp
TORCH_CUDA_CPP_API void scatter(
    const std::vector<at::Tensor>& inputs,
    at::Tensor& outputs,
    ncclComm_t comm,
    at::cuda::CUDAStream& stream,
    int32_t root = 0);

```
- **EN**: Contains local statements, constants, or helper glue that support the surrounding control flow and data movement.
- **CN**: 包含局部语句、常量或辅助胶水代码，用于支撑周围的控制流与数据传递。

### Lines 170-175: Supporting statements / 辅助语句
```cpp
TORCH_CUDA_CPP_API void all_gather(
    const std::vector<at::Tensor>& inputs,
    std::vector<at::Tensor>& outputs,
    const stream_list& streams = {},
    const comm_list& user_comms = {});

```
- **EN**: Contains local statements, constants, or helper glue that support the surrounding control flow and data movement.
- **CN**: 包含局部语句、常量或辅助胶水代码，用于支撑周围的控制流与数据传递。

### Lines 176-182: Supporting statements / 辅助语句
```cpp
TORCH_CUDA_CPP_API void gather(
    const at::Tensor& inputs,
    std::vector<at::Tensor>& outputs,
    ncclComm_t comm,
    at::cuda::CUDAStream& stream,
    int32_t root = 0);

```
- **EN**: Contains local statements, constants, or helper glue that support the surrounding control flow and data movement.
- **CN**: 包含局部语句、常量或辅助胶水代码，用于支撑周围的控制流与数据传递。

### Lines 183-189: Supporting statements / 辅助语句
```cpp
TORCH_CUDA_CPP_API void all2all_single_equal_split(
    at::Tensor& input,
    at::Tensor& output,
    int size,
    ncclComm_t comm,
    at::cuda::CUDAStream& stream);

```
- **EN**: Contains local statements, constants, or helper glue that support the surrounding control flow and data movement.
- **CN**: 包含局部语句、常量或辅助胶水代码，用于支撑周围的控制流与数据传递。

### Lines 190-201: Supporting statements / 辅助语句
```cpp
TORCH_CUDA_CPP_API void all2all_single_unequal_split(
    void* sendbuff,
    const size_t* sendcounts,
    const size_t* senddispls,
    void* recvbuff,
    const size_t* recvcounts,
    const size_t* recvdispls,
    size_t size,
    c10::ScalarType type,
    ncclComm_t comm,
    at::cuda::CUDAStream& stream);

```
- **EN**: Contains local statements, constants, or helper glue that support the surrounding control flow and data movement.
- **CN**: 包含局部语句、常量或辅助胶水代码，用于支撑周围的控制流与数据传递。

### Lines 202-207: Supporting statements / 辅助语句
```cpp
TORCH_CUDA_CPP_API void all2all(
    std::vector<at::Tensor>& outputTensors,
    std::vector<at::Tensor>& inputTensors,
    ncclComm_t _comm,
    at::cuda::CUDAStream& stream);

```
- **EN**: Contains local statements, constants, or helper glue that support the surrounding control flow and data movement.
- **CN**: 包含局部语句、常量或辅助胶水代码，用于支撑周围的控制流与数据传递。

### Lines 208-213: Supporting statements / 辅助语句
```cpp
TORCH_CUDA_CPP_API void send(
    const at::Tensor& input,
    ncclComm_t comm,
    at::cuda::CUDAStream stream,
    int dst);

```
- **EN**: Contains local statements, constants, or helper glue that support the surrounding control flow and data movement.
- **CN**: 包含局部语句、常量或辅助胶水代码，用于支撑周围的控制流与数据传递。

### Lines 214-219: Supporting statements / 辅助语句
```cpp
TORCH_CUDA_CPP_API void recv(
    at::Tensor& output,
    ncclComm_t comm,
    at::cuda::CUDAStream stream,
    int src);
} // namespace torch::cuda::nccl
```
- **EN**: Contains local statements, constants, or helper glue that support the surrounding control flow and data movement.
- **CN**: 包含局部语句、常量或辅助胶水代码，用于支撑周围的控制流与数据传递。

## Key Concepts / 关键概念
- CUDA backend integration / CUDA 后端集成
- NCCL integration / NCCL 集成
- ATen runtime interfaces / ATen 运行时接口

## Dependencies / 依赖关系
### Internal / 内部
- `ATen/ATen.h`
- `ATen/cuda/CUDAContext.h`
### External / 外部
- `cstddef`
- `optional`
- `vector`
