# nccl.cpp — Code Analysis / 代码分析

## Source / 来源

- **File**: `torch/csrc/cuda/nccl.cpp`
- **Repository**: `pytorch/pytorch`
- **Purpose**: This file implements logic for `nccl.cpp` inside the CUDA backend bindings, memory helpers, streams, events, and communication utilities, with emphasis on nccl integration, cuda backend integration. / 该文件在CUDA 后端绑定、内存辅助逻辑、流、事件与通信工具中针对 `nccl.cpp` 实现逻辑，重点涉及NCCL 集成、CUDA 后端集成。

## Line-by-Line Analysis / 逐行分析

### Lines 1-4: Header dependencies / 头文件依赖
```cpp
#include <ATen/core/functional.h>
#include <torch/csrc/cuda/device_set.h>
#include <torch/csrc/cuda/nccl.h>

```
- **EN**: Pulls in the headers required by this translation unit, including interfaces from the CUDA backend bindings, memory helpers, streams, events, and communication utilities.
- **CN**: 引入该翻译单元所需的头文件，包括来自CUDA 后端绑定、内存辅助逻辑、流、事件与通信工具的接口。

### Lines 5-11: Header dependencies / 头文件依赖
```cpp
#include <ATen/ATen.h>
#include <c10/cuda/CUDAException.h>
#include <c10/cuda/CUDAGuard.h>
#include <c10/util/Exception.h>
#include <c10/util/hash.h>
#include <c10/util/irange.h>

```
- **EN**: Pulls in the headers required by this translation unit, including interfaces from the CUDA backend bindings, memory helpers, streams, events, and communication utilities.
- **CN**: 引入该翻译单元所需的头文件，包括来自CUDA 后端绑定、内存辅助逻辑、流、事件与通信工具的接口。

### Lines 12-13: Header dependencies / 头文件依赖
```cpp
#include <nccl.h>

```
- **EN**: Pulls in the headers required by this translation unit, including interfaces from the CUDA backend bindings, memory helpers, streams, events, and communication utilities.
- **CN**: 引入该翻译单元所需的头文件，包括来自CUDA 后端绑定、内存辅助逻辑、流、事件与通信工具的接口。

### Lines 14-19: Header dependencies / 头文件依赖
```cpp
#include <sched.h>
#include <limits>
#include <sstream>
#include <type_traits>
#include <unordered_map>

```
- **EN**: Pulls in the headers required by this translation unit, including interfaces from the CUDA backend bindings, memory helpers, streams, events, and communication utilities.
- **CN**: 引入该翻译单元所需的头文件，包括来自CUDA 后端绑定、内存辅助逻辑、流、事件与通信工具的接口。

### Lines 20-23: Preprocessor configuration / 预处理配置
```cpp
#if (NCCL_MAJOR > 2) || ((NCCL_MAJOR == 2) && (NCCL_MINOR >= 13))
#define NCCL_HAS_REMOTE_ERROR 1
#endif

```
- **EN**: Applies compile-time gating, macros, or platform-specific configuration so the file builds correctly under different feature sets.
- **CN**: 应用编译期开关、宏或平台特定配置，使该文件能在不同功能组合下正确构建。

### Lines 24-27: Preprocessor configuration / 预处理配置
```cpp
#if (NCCL_MAJOR > 2) || ((NCCL_MAJOR == 2) && (NCCL_MINOR >= 14))
#define NCCL_HAS_COMM_NONBLOCKING 1
#endif

```
- **EN**: Applies compile-time gating, macros, or platform-specific configuration so the file builds correctly under different feature sets.
- **CN**: 应用编译期开关、宏或平台特定配置，使该文件能在不同功能组合下正确构建。

### Lines 28-31: Function `to_nccl_comm` / 函数 `to_nccl_comm`
```cpp
ncclComm_t* to_nccl_comm(torch::cuda::nccl::ncclComm_t* var) {
  return reinterpret_cast<ncclComm_t*>(var);
}

```
- **EN**: Implements `to_nccl_comm`, one of the operational units in this file for the CUDA backend bindings, memory helpers, streams, events, and communication utilities.
- **CN**: 实现 `to_nccl_comm`，它是该文件中服务于CUDA 后端绑定、内存辅助逻辑、流、事件与通信工具的一个运行单元。

### Lines 32-35: Function `to_nccl_comm` / 函数 `to_nccl_comm`
```cpp
ncclComm_t to_nccl_comm(torch::cuda::nccl::ncclComm_t var) {
  return reinterpret_cast<ncclComm_t>(var);
}

```
- **EN**: Implements `to_nccl_comm`, one of the operational units in this file for the CUDA backend bindings, memory helpers, streams, events, and communication utilities.
- **CN**: 实现 `to_nccl_comm`，它是该文件中服务于CUDA 后端绑定、内存辅助逻辑、流、事件与通信工具的一个运行单元。

### Lines 36-39: Function `to_nccl_unique_id` / 函数 `to_nccl_unique_id`
```cpp
ncclUniqueId* to_nccl_unique_id(torch::cuda::nccl::ncclUniqueId* var) {
  return reinterpret_cast<ncclUniqueId*>(var);
}

```
- **EN**: Implements `to_nccl_unique_id`, one of the operational units in this file for the CUDA backend bindings, memory helpers, streams, events, and communication utilities.
- **CN**: 实现 `to_nccl_unique_id`，它是该文件中服务于CUDA 后端绑定、内存辅助逻辑、流、事件与通信工具的一个运行单元。

### Lines 40-63: Function `to_nccl_result` / 函数 `to_nccl_result`
```cpp
ncclResult_t to_nccl_result(torch::cuda::nccl::ncclResult var) {
  switch (var) {
    case torch::cuda::nccl::ncclResult::Success:
      return ncclResult_t::ncclSuccess;
    case torch::cuda::nccl::ncclResult::UnhandledCudaError:
      return ncclResult_t::ncclUnhandledCudaError;
    case torch::cuda::nccl::ncclResult::SystemError:
      return ncclResult_t::ncclSystemError;
    case torch::cuda::nccl::ncclResult::InternalError:
      return ncclResult_t::ncclInternalError;
    case torch::cuda::nccl::ncclResult::InvalidArgument:
      return ncclResult_t::ncclInvalidArgument;
    case torch::cuda::nccl::ncclResult::InvalidUsage:
      return ncclResult_t::ncclInvalidUsage;
#ifdef NCCL_HAS_REMOTE_ERROR
    case torch::cuda::nccl::ncclResult::RemoteError:
      return ncclResult_t::ncclRemoteError;
#endif
#ifdef NCCL_HAS_COMM_NONBLOCKING
    case torch::cuda::nccl::ncclResult::InProgress:
      return ncclResult_t::ncclInProgress;
#endif
    case torch::cuda::nccl::ncclResult::NumResults:
      return ncclResult_t::ncclNumResults;
```
- **EN**: Implements `to_nccl_result`, one of the operational units in this file for the CUDA backend bindings, memory helpers, streams, events, and communication utilities.
- **CN**: 实现 `to_nccl_result`，它是该文件中服务于CUDA 后端绑定、内存辅助逻辑、流、事件与通信工具的一个运行单元。

### Lines 64-68: Supporting statements / 辅助语句
```cpp
    default:
      TORCH_CHECK(false, "Unconvertible NCCL type");
  }
}

```
- **EN**: Contains local statements, constants, or helper glue that support the surrounding control flow and data movement.
- **CN**: 包含局部语句、常量或辅助胶水代码，用于支撑周围的控制流与数据传递。

### Lines 69-92: Function `from_nccl_result` / 函数 `from_nccl_result`
```cpp
torch::cuda::nccl::ncclResult from_nccl_result(ncclResult_t var) {
  switch (var) {
    case ncclSuccess:
      return torch::cuda::nccl::ncclResult::Success;
    case ncclUnhandledCudaError:
      return torch::cuda::nccl::ncclResult::UnhandledCudaError;
    case ncclSystemError:
      return torch::cuda::nccl::ncclResult::SystemError;
    case ncclInternalError:
      return torch::cuda::nccl::ncclResult::InternalError;
    case ncclInvalidArgument:
      return torch::cuda::nccl::ncclResult::InvalidArgument;
    case ncclInvalidUsage:
      return torch::cuda::nccl::ncclResult::InvalidUsage;
#ifdef NCCL_HAS_REMOTE_ERROR
    case ncclRemoteError:
      return torch::cuda::nccl::ncclResult::RemoteError;
#endif
#ifdef NCCL_HAS_COMM_NONBLOCKING
    case ncclInProgress:
      return torch::cuda::nccl::ncclResult::InProgress;
#endif
    case ncclNumResults:
      return torch::cuda::nccl::ncclResult::NumResults;
```
- **EN**: Implements `from_nccl_result`, one of the operational units in this file for the CUDA backend bindings, memory helpers, streams, events, and communication utilities.
- **CN**: 实现 `from_nccl_result`，它是该文件中服务于CUDA 后端绑定、内存辅助逻辑、流、事件与通信工具的一个运行单元。

### Lines 93-97: Supporting statements / 辅助语句
```cpp
    default:
      TORCH_CHECK(false, "Unconvertible NCCL type");
  }
}

```
- **EN**: Contains local statements, constants, or helper glue that support the surrounding control flow and data movement.
- **CN**: 包含局部语句、常量或辅助胶水代码，用于支撑周围的控制流与数据传递。

### Lines 98-121: Function `to_nccl_data_type` / 函数 `to_nccl_data_type`
```cpp
ncclDataType_t to_nccl_data_type(c10::ScalarType type) {
  switch (type) {
    case at::kFloat:
      return ncclDataType_t::ncclFloat;
    case at::kHalf:
      return ncclDataType_t::ncclHalf;
    case at::kDouble:
      return ncclDataType_t::ncclDouble;
    case at::kLong:
      return ncclDataType_t::ncclInt64;
    case at::kInt:
      return ncclDataType_t::ncclInt;
    case at::kChar:
      return ncclDataType_t::ncclChar;
    // NOLINTNEXTLINE(*-narrowing-conversions, bugprone-branch-clone)
    case at::kByte:
      return ncclDataType_t::ncclUint8;
    case at::kBool:
      return ncclDataType_t::ncclUint8;
    case at::kFloat8_e4m3fnuz:
      return ncclDataType_t::ncclUint8;
    case at::kFloat8_e5m2fnuz:
      return ncclDataType_t::ncclUint8;
    case at::kFloat8_e4m3fn:
```
- **EN**: Implements `to_nccl_data_type`, one of the operational units in this file for the CUDA backend bindings, memory helpers, streams, events, and communication utilities.
- **CN**: 实现 `to_nccl_data_type`，它是该文件中服务于CUDA 后端绑定、内存辅助逻辑、流、事件与通信工具的一个运行单元。

### Lines 122-133: Supporting statements / 辅助语句
```cpp
      return ncclDataType_t::ncclUint8;
    case at::kFloat8_e5m2:
      return ncclDataType_t::ncclUint8;
#if HAS_NCCL_BF16_DATATYPE
    case at::kBFloat16:
      return ncclDataType_t::ncclBfloat16;
#endif
    default:
      TORCH_CHECK(false, "Unconvertible NCCL type ", type);
  }
}

```
- **EN**: Contains local statements, constants, or helper glue that support the surrounding control flow and data movement.
- **CN**: 包含局部语句、常量或辅助胶水代码，用于支撑周围的控制流与数据传递。

### Lines 134-143: Function `to_nccl_data_type` / 函数 `to_nccl_data_type`
```cpp
ncclDataType_t to_nccl_data_type(const at::Tensor& t) {
  if (!t.is_cuda()) {
    TORCH_CHECK(
        false,
        "NCCL only supports CUDA tensors, but got a tensor on ",
        t.device());
  }
  return to_nccl_data_type(t.scalar_type());
}

```
- **EN**: Implements `to_nccl_data_type`, one of the operational units in this file for the CUDA backend bindings, memory helpers, streams, events, and communication utilities.
- **CN**: 实现 `to_nccl_data_type`，它是该文件中服务于CUDA 后端绑定、内存辅助逻辑、流、事件与通信工具的一个运行单元。

### Lines 144-147: Function `to_nccl_red_op` / 函数 `to_nccl_red_op`
```cpp
ncclRedOp_t to_nccl_red_op(int var) {
  return (ncclRedOp_t)var;
}

```
- **EN**: Implements `to_nccl_red_op`, one of the operational units in this file for the CUDA backend bindings, memory helpers, streams, events, and communication utilities.
- **CN**: 实现 `to_nccl_red_op`，它是该文件中服务于CUDA 后端绑定、内存辅助逻辑、流、事件与通信工具的一个运行单元。

### Lines 148-149: Namespace scope / 命名空间作用域
```cpp
namespace torch::cuda::nccl {

```
- **EN**: Opens or organizes namespace scopes so related symbols live under the expected PyTorch component hierarchy.
- **CN**: 打开或组织命名空间作用域，使相关符号位于预期的 PyTorch 组件层级中。

### Lines 150-151: Using declarations / using 声明
```cpp
using namespace at;

```
- **EN**: Introduces aliases or selected names to simplify later references inside this file.
- **CN**: 引入别名或选定名称，以简化文件后续代码中的引用。

### Lines 152-153: Namespace scope / 命名空间作用域
```cpp
namespace detail {

```
- **EN**: Opens or organizes namespace scopes so related symbols live under the expected PyTorch component hierarchy.
- **CN**: 打开或组织命名空间作用域，使相关符号位于预期的 PyTorch 组件层级中。

### Lines 154-157: Function `NCCL_CHECK` / 函数 `NCCL_CHECK`
```cpp
static void NCCL_CHECK(ncclResult_t result) {
  NCCL_CHECK(from_nccl_result(result));
}

```
- **EN**: Implements `NCCL_CHECK`, one of the operational units in this file for the CUDA backend bindings, memory helpers, streams, events, and communication utilities.
- **CN**: 实现 `NCCL_CHECK`，它是该文件中服务于CUDA 后端绑定、内存辅助逻辑、流、事件与通信工具的一个运行单元。

### Lines 158-167: Supporting statements / 辅助语句
```cpp
// TODO(eqy): can this duplication be avoided from NCCLUtils.cpp?
bool nccl_use_nonblocking() {
  static bool nccl_use_nonblocking_ =
      c10::utils::check_env("TORCH_NCCL_USE_COMM_NONBLOCKING") == true;
  if (nccl_use_nonblocking_) {
    TORCH_WARN("Using experimental non-blocking NCCL communicator.");
  }
  return nccl_use_nonblocking_;
}

```
- **EN**: Contains local statements, constants, or helper glue that support the surrounding control flow and data movement.
- **CN**: 包含局部语句、常量或辅助胶水代码，用于支撑周围的控制流与数据传递。

### Lines 168-183: Supporting statements / 辅助语句
```cpp
// Default value: 30 minutes
static int nccl_nonblocking_timeout() {
  static int timeout = -2; // -2 means not initialized
  if (timeout == -2) {
    const auto val = c10::utils::get_env("TORCH_NCCL_NONBLOCKING_TIMEOUT");
    if (val && !val.value().empty()) {
      // NOLINTNEXTLINE(*-narrowing-conversions)
      timeout = strtol(val->c_str(), nullptr, 0);
    } else {
      // Default value consistent with kBackendDefaultTimeout
      timeout = 30 * 60;
    }
  }
  return timeout;
}

```
- **EN**: Contains local statements, constants, or helper glue that support the surrounding control flow and data movement.
- **CN**: 包含局部语句、常量或辅助胶水代码，用于支撑周围的控制流与数据传递。

### Lines 184-207: Function `NCCL_CHECK_TIMEOUT` / 函数 `NCCL_CHECK_TIMEOUT`
```cpp
static void NCCL_CHECK_TIMEOUT(ncclResult status, ncclComm_t comm) {
#ifdef NCCL_HAS_COMM_NONBLOCKING
  ncclResult_t result = to_nccl_result(status);
  auto startTimepoint = std::chrono::steady_clock::now();
  while (result == ncclInProgress) {
    auto currentTimepoint = std::chrono::steady_clock::now();
    auto timeElapsed = std::chrono::duration_cast<std::chrono::seconds>(
                           currentTimepoint - startTimepoint)
                           .count();
    TORCH_CHECK(
        timeElapsed <= nccl_nonblocking_timeout(),
        "NCCL timeout when waiting for nonblocking call to become successful.");
    sched_yield(); // yield to other threads
    ncclCommGetAsyncError(to_nccl_comm(comm), &result);
  }
  if (result != ncclSuccess) {
    throw_nccl_error(from_nccl_result(result));
  }
#else
  TORCH_INTERNAL_ASSERT(
      false, "NCCL COMM NONBLOCKING USED WITH UNSUPPORTED NCCL VERSION.");
#endif
}

```
- **EN**: Implements `NCCL_CHECK_TIMEOUT`, one of the operational units in this file for the CUDA backend bindings, memory helpers, streams, events, and communication utilities.
- **CN**: 实现 `NCCL_CHECK_TIMEOUT`，它是该文件中服务于CUDA 后端绑定、内存辅助逻辑、流、事件与通信工具的一个运行单元。

### Lines 208-211: Function `NCCL_CHECK_TIMEOUT` / 函数 `NCCL_CHECK_TIMEOUT`
```cpp
static void NCCL_CHECK_TIMEOUT(ncclResult_t result, ncclComm_t comm) {
  NCCL_CHECK_TIMEOUT(from_nccl_result(result), comm);
}

```
- **EN**: Implements `NCCL_CHECK_TIMEOUT`, one of the operational units in this file for the CUDA backend bindings, memory helpers, streams, events, and communication utilities.
- **CN**: 实现 `NCCL_CHECK_TIMEOUT`，它是该文件中服务于CUDA 后端绑定、内存辅助逻辑、流、事件与通信工具的一个运行单元。

### Lines 212-235: Supporting statements / 辅助语句
```cpp
static void NCCL_CHECK_TIMEOUT(
    ncclResult status,
    std::vector<ncclComm_t>& comms) {
#ifdef NCCL_HAS_COMM_NONBLOCKING
  ncclResult_t result = to_nccl_result(status);
  auto startTimepoint = std::chrono::steady_clock::now();
  if (result == ncclInProgress) {
    for (const auto i : c10::irange(comms.size())) {
      do {
        auto currentTimepoint = std::chrono::steady_clock::now();
        auto timeElapsed = std::chrono::duration_cast<std::chrono::seconds>(
                               currentTimepoint - startTimepoint)
                               .count();
        TORCH_CHECK(
            timeElapsed <= nccl_nonblocking_timeout(),
            "NCCL timeout when waiting for nonblocking call to become successful.");
        sched_yield(); // yield to other threads
        ncclCommGetAsyncError(to_nccl_comm(comms[i]), &result);
      } while (result == ncclInProgress);
      if (result != ncclSuccess) {
        break; /* fall through to failed case */
      }
    }
  }
```
- **EN**: Contains local statements, constants, or helper glue that support the surrounding control flow and data movement.
- **CN**: 包含局部语句、常量或辅助胶水代码，用于支撑周围的控制流与数据传递。

### Lines 236-244: Supporting statements / 辅助语句
```cpp
  if (result != ncclSuccess) {
    throw_nccl_error(from_nccl_result(result));
  }
#else
  TORCH_INTERNAL_ASSERT(
      false, "NCCL COMM NONBLOCKING USED WITH UNSUPPORTED NCCL VERSION.");
#endif
}

```
- **EN**: Contains local statements, constants, or helper glue that support the surrounding control flow and data movement.
- **CN**: 包含局部语句、常量或辅助胶水代码，用于支撑周围的控制流与数据传递。

### Lines 245-250: Supporting statements / 辅助语句
```cpp
static void NCCL_CHECK_TIMEOUT(
    ncclResult_t result,
    std::vector<ncclComm_t>& comms) {
  NCCL_CHECK_TIMEOUT(from_nccl_result(result), comms);
}

```
- **EN**: Contains local statements, constants, or helper glue that support the surrounding control flow and data movement.
- **CN**: 包含局部语句、常量或辅助胶水代码，用于支撑周围的控制流与数据传递。

### Lines 251-257: Function `throw_nccl_error` / 函数 `throw_nccl_error`
```cpp
void throw_nccl_error(torch::cuda::nccl::ncclResult status) {
  std::ostringstream err;
  err << "NCCL Error " << static_cast<int>(status) << ": "
      << ncclGetErrorString(to_nccl_result(status));
  TORCH_CHECK(false, err.str());
}

```
- **EN**: Implements `throw_nccl_error`, one of the operational units in this file for the CUDA backend bindings, memory helpers, streams, events, and communication utilities.
- **CN**: 实现 `throw_nccl_error`，它是该文件中服务于CUDA 后端绑定、内存辅助逻辑、流、事件与通信工具的一个运行单元。

### Lines 258-281: Type declaration / 类型声明
```cpp
struct NcclCommList {
  // NOLINTNEXTLINE(*array*)
  std::unique_ptr<ncclComm_t[]> comms;
  size_t ndevices;
  NcclCommList(const std::vector<int>& devices)
      : comms(new ncclComm_t[devices.size()]), ndevices(devices.size()) {
    NCCL_CHECK(ncclCommInitAll(
        to_nccl_comm(comms.get()),
        static_cast<int>(devices.size()),
        devices.data()));
  }
  NcclCommList(NcclCommList&& foo) = default;
  // NOLINTNEXTLINE(bugprone-exception-escape)
  ~NcclCommList() {
    if (comms) {
      for (const auto i : c10::irange(ndevices)) {
        int dummy_var = 0;
        if (C10_CUDA_ERROR_HANDLED(cudaGetDevice(&dummy_var)) != cudaSuccess) {
          /* there are cases when this destructor is called after the
           CUDA driver is already unloaded from the process.
           In these cases, skip ncclCommDestroy */
          return;
        }
        comm_destroy(comms[i]);
```
- **EN**: Declares a core helper type, wrapper, or policy object that structures the rest of the implementation.
- **CN**: 声明一个核心辅助类型、封装对象或策略对象，用于组织后续实现。

### Lines 282-289: Supporting statements / 辅助语句
```cpp
      }
    }
  }
  ArrayRef<ncclComm_t> ref() const {
    return ArrayRef<ncclComm_t>(comms.get(), ndevices);
  }
};

```
- **EN**: Contains local statements, constants, or helper glue that support the surrounding control flow and data movement.
- **CN**: 包含局部语句、常量或辅助胶水代码，用于支撑周围的控制流与数据传递。

### Lines 290-294: Using declarations / using 声明
```cpp
using device_list = std::vector<int>;
// accesses to this object have to be guarded by THC's CudaFreeMutex
static std::unordered_map<device_list, NcclCommList, c10::hash<device_list>>
    _communicators;

```
- **EN**: Introduces aliases or selected names to simplify later references inside this file.
- **CN**: 引入别名或选定名称，以简化文件后续代码中的引用。

### Lines 295-306: Function `get_communicators` / 函数 `get_communicators`
```cpp
ArrayRef<ncclComm_t> get_communicators(TensorList inputs) {
  static auto get_device = [](const at::Tensor& t) -> int {
    return t.get_device();
  };
  device_list devices = fmap(inputs, get_device);
  auto it = _communicators.find(devices);
  if (it == _communicators.end()) {
    it = _communicators.emplace(devices, devices).first;
  }
  return it->second.ref();
}

```
- **EN**: Implements `get_communicators`, one of the operational units in this file for the CUDA backend bindings, memory helpers, streams, events, and communication utilities.
- **CN**: 实现 `get_communicators`，它是该文件中服务于CUDA 后端绑定、内存辅助逻辑、流、事件与通信工具的一个运行单元。

### Lines 307-318: Supporting statements / 辅助语句
```cpp
static void check_tensor(
    const at::Tensor& input,
    const std::optional<at::Tensor>& output,
    size_t input_multiplier,
    size_t output_multiplier,
    int64_t ref_numel,
    ScalarType ref_dtype) {
  auto check_one = [&](const at::Tensor& tensor) {
    TORCH_CHECK(
        tensor.is_cuda() && !tensor.is_sparse(),
        "input and output elements have to be cuda dense Tensors");

```
- **EN**: Contains local statements, constants, or helper glue that support the surrounding control flow and data movement.
- **CN**: 包含局部语句、常量或辅助胶水代码，用于支撑周围的控制流与数据传递。

### Lines 319-322: Supporting statements / 辅助语句
```cpp
    TORCH_CHECK(
        ref_dtype == tensor.scalar_type(),
        "all inputs and outputs must be of the same Tensor dtype");

```
- **EN**: Contains local statements, constants, or helper glue that support the surrounding control flow and data movement.
- **CN**: 包含局部语句、常量或辅助胶水代码，用于支撑周围的控制流与数据传递。

### Lines 323-326: Supporting statements / 辅助语句
```cpp
    TORCH_CHECK(
        tensor.is_contiguous(), "all inputs and outputs have to be contiguous");
  };

```
- **EN**: Contains local statements, constants, or helper glue that support the surrounding control flow and data movement.
- **CN**: 包含局部语句、常量或辅助胶水代码，用于支撑周围的控制流与数据传递。

### Lines 327-328: Supporting statements / 辅助语句
```cpp
  check_one(input);

```
- **EN**: Contains local statements, constants, or helper glue that support the surrounding control flow and data movement.
- **CN**: 包含局部语句、常量或辅助胶水代码，用于支撑周围的控制流与数据传递。

### Lines 329-333: Supporting statements / 辅助语句
```cpp
  // all inputs must be same size
  TORCH_CHECK(
      input.numel() == ref_numel,
      "all inputs must have the same number of elements");

```
- **EN**: Contains local statements, constants, or helper glue that support the surrounding control flow and data movement.
- **CN**: 包含局部语句、常量或辅助胶水代码，用于支撑周围的控制流与数据传递。

### Lines 334-336: Supporting statements / 辅助语句
```cpp
  if (output) {
    check_one(*output);

```
- **EN**: Contains local statements, constants, or helper glue that support the surrounding control flow and data movement.
- **CN**: 包含局部语句、常量或辅助胶水代码，用于支撑周围的控制流与数据传递。

### Lines 337-341: Supporting statements / 辅助语句
```cpp
    // inputs and outputs must be on same device respectively
    TORCH_CHECK(
        input.get_device() == output->get_device(),
        "input and output must be on the same device");

```
- **EN**: Contains local statements, constants, or helper glue that support the surrounding control flow and data movement.
- **CN**: 包含局部语句、常量或辅助胶水代码，用于支撑周围的控制流与数据传递。

### Lines 342-347: Supporting statements / 辅助语句
```cpp
    TORCH_CHECK(
        output->numel() * output_multiplier == ref_numel * input_multiplier,
        "output must be of size input_size * size_multiplier");
  }
}

```
- **EN**: Contains local statements, constants, or helper glue that support the surrounding control flow and data movement.
- **CN**: 包含局部语句、常量或辅助胶水代码，用于支撑周围的控制流与数据传递。

### Lines 348-355: Supporting statements / 辅助语句
```cpp
void check_inputs(
    TensorList inputs,
    TensorList outputs,
    size_t input_multiplier,
    size_t output_multiplier) {
  // len(inputs) == len(outputs)
  size_t len = inputs.size();

```
- **EN**: Contains local statements, constants, or helper glue that support the surrounding control flow and data movement.
- **CN**: 包含局部语句、常量或辅助胶水代码，用于支撑周围的控制流与数据传递。

### Lines 356-357: Supporting statements / 辅助语句
```cpp
  TORCH_CHECK(len != 0, "input sequence can't be empty");

```
- **EN**: Contains local statements, constants, or helper glue that support the surrounding control flow and data movement.
- **CN**: 包含局部语句、常量或辅助胶水代码，用于支撑周围的控制流与数据传递。

### Lines 358-364: Supporting statements / 辅助语句
```cpp
  if (len != outputs.size()) {
    std::stringstream err;
    err << "inputs and outputs sequences have to be of the same length, but got input of length "
        << len << " and output of length " << outputs.size();
    TORCH_CHECK(false, err.str());
  }

```
- **EN**: Contains local statements, constants, or helper glue that support the surrounding control flow and data movement.
- **CN**: 包含局部语句、常量或辅助胶水代码，用于支撑周围的控制流与数据传递。

### Lines 365-368: Supporting statements / 辅助语句
```cpp
  device_set devices;
  int64_t numel = inputs[0].numel();
  auto dtype = inputs[0].scalar_type();

```
- **EN**: Contains local statements, constants, or helper glue that support the surrounding control flow and data movement.
- **CN**: 包含局部语句、常量或辅助胶水代码，用于支撑周围的控制流与数据传递。

### Lines 369-372: Supporting statements / 辅助语句
```cpp
  for (const auto i : c10::irange(len)) {
    const auto& input = inputs[i];
    auto output = outputs[i];

```
- **EN**: Contains local statements, constants, or helper glue that support the surrounding control flow and data movement.
- **CN**: 包含局部语句、常量或辅助胶水代码，用于支撑周围的控制流与数据传递。

### Lines 373-375: Supporting statements / 辅助语句
```cpp
    check_tensor(
        input, output, input_multiplier, output_multiplier, numel, dtype);

```
- **EN**: Contains local statements, constants, or helper glue that support the surrounding control flow and data movement.
- **CN**: 包含局部语句、常量或辅助胶水代码，用于支撑周围的控制流与数据传递。

### Lines 376-383: Supporting statements / 辅助语句
```cpp
    auto input_device = input.get_device();
    // inputs must be on unique devices
    TORCH_CHECK(
        !devices.test(input_device), "inputs must be on unique devices");
    devices.set(input_device);
  }
}

```
- **EN**: Contains local statements, constants, or helper glue that support the surrounding control flow and data movement.
- **CN**: 包含局部语句、常量或辅助胶水代码，用于支撑周围的控制流与数据传递。

### Lines 384-391: Supporting statements / 辅助语句
```cpp
void check_inputs(
    TensorList inputs,
    const at::Tensor& output,
    int root,
    int input_multiplier,
    int output_multiplier) {
  auto len = inputs.size();

```
- **EN**: Contains local statements, constants, or helper glue that support the surrounding control flow and data movement.
- **CN**: 包含局部语句、常量或辅助胶水代码，用于支撑周围的控制流与数据传递。

### Lines 392-393: Supporting statements / 辅助语句
```cpp
  TORCH_CHECK(len > 0, "input sequence can't be empty");

```
- **EN**: Contains local statements, constants, or helper glue that support the surrounding control flow and data movement.
- **CN**: 包含局部语句、常量或辅助胶水代码，用于支撑周围的控制流与数据传递。

### Lines 394-397: Supporting statements / 辅助语句
```cpp
  device_set devices;
  int64_t numel = inputs[0].numel();
  auto dtype = inputs[0].scalar_type();

```
- **EN**: Contains local statements, constants, or helper glue that support the surrounding control flow and data movement.
- **CN**: 包含局部语句、常量或辅助胶水代码，用于支撑周围的控制流与数据传递。

### Lines 398-400: Supporting statements / 辅助语句
```cpp
  for (const auto i : c10::irange(len)) {
    const auto& input = inputs[i];

```
- **EN**: Contains local statements, constants, or helper glue that support the surrounding control flow and data movement.
- **CN**: 包含局部语句、常量或辅助胶水代码，用于支撑周围的控制流与数据传递。

### Lines 401-410: Supporting statements / 辅助语句
```cpp
    check_tensor(
        input,
        i == static_cast<std::remove_cv_t<decltype(i)>>(root)
            ? std::optional<at::Tensor>{output}
            : std::nullopt,
        input_multiplier,
        output_multiplier,
        numel,
        dtype);

```
- **EN**: Contains local statements, constants, or helper glue that support the surrounding control flow and data movement.
- **CN**: 包含局部语句、常量或辅助胶水代码，用于支撑周围的控制流与数据传递。

### Lines 411-418: Supporting statements / 辅助语句
```cpp
    auto input_device = input.get_device();
    // inputs must be on unique devices
    TORCH_CHECK(
        !devices.test(input_device), "inputs must be on unique devices");
    devices.set(input_device);
  }
}

```
- **EN**: Contains local statements, constants, or helper glue that support the surrounding control flow and data movement.
- **CN**: 包含局部语句、常量或辅助胶水代码，用于支撑周围的控制流与数据传递。

### Lines 419-420: Supporting statements / 辅助语句
```cpp
} // namespace detail

```
- **EN**: Contains local statements, constants, or helper glue that support the surrounding control flow and data movement.
- **CN**: 包含局部语句、常量或辅助胶水代码，用于支撑周围的控制流与数据传递。

### Lines 421-426: Function `AutoNcclGroup` / 函数 `AutoNcclGroup`
```cpp
AutoNcclGroup::AutoNcclGroup() : comm_(nullptr), comm_nonblocking_(false) {
#if defined(NCCL_MAJOR) && (NCCL_MAJOR < 2)
  // nccl < 2.0 cannot be called concurrently with cudaFree
  (c10::cuda::getFreeMutex())->lock();
#endif

```
- **EN**: Implements `AutoNcclGroup`, one of the operational units in this file for the CUDA backend bindings, memory helpers, streams, events, and communication utilities.
- **CN**: 实现 `AutoNcclGroup`，它是该文件中服务于CUDA 后端绑定、内存辅助逻辑、流、事件与通信工具的一个运行单元。

### Lines 427-431: Preprocessor configuration / 预处理配置
```cpp
#if defined(NCCL_MAJOR) && (NCCL_MAJOR >= 2)
  detail::NCCL_CHECK(ncclGroupStart());
#endif
}

```
- **EN**: Applies compile-time gating, macros, or platform-specific configuration so the file builds correctly under different feature sets.
- **CN**: 应用编译期开关、宏或平台特定配置，使该文件能在不同功能组合下正确构建。

### Lines 432-438: Function `AutoNcclGroup` / 函数 `AutoNcclGroup`
```cpp
AutoNcclGroup::AutoNcclGroup(ncclComm_t comm, bool comm_nonblocking)
    : comm_(comm), comm_nonblocking_(comm_nonblocking) {
#if defined(NCCL_MAJOR) && (NCCL_MAJOR < 2)
  // nccl < 2.0 cannot be called concurrently with cudaFree
  (c10::cuda::getFreeMutex())->lock();
#endif

```
- **EN**: Implements `AutoNcclGroup`, one of the operational units in this file for the CUDA backend bindings, memory helpers, streams, events, and communication utilities.
- **CN**: 实现 `AutoNcclGroup`，它是该文件中服务于CUDA 后端绑定、内存辅助逻辑、流、事件与通信工具的一个运行单元。

### Lines 439-443: Preprocessor configuration / 预处理配置
```cpp
#if defined(NCCL_MAJOR) && (NCCL_MAJOR >= 2)
  detail::NCCL_CHECK(ncclGroupStart());
#endif
}

```
- **EN**: Applies compile-time gating, macros, or platform-specific configuration so the file builds correctly under different feature sets.
- **CN**: 应用编译期开关、宏或平台特定配置，使该文件能在不同功能组合下正确构建。

### Lines 444-457: Function `NOLINTNEXTLINE` / 函数 `NOLINTNEXTLINE`
```cpp
// NOLINTNEXTLINE(bugprone-exception-escape)
AutoNcclGroup::~AutoNcclGroup() noexcept(false) {
#if defined(NCCL_MAJOR) && (NCCL_MAJOR >= 2)
  if (comm_nonblocking_ && comm_ != nullptr) {
    detail::NCCL_CHECK_TIMEOUT(ncclGroupEnd(), comm_);
  } else {
    detail::NCCL_CHECK(ncclGroupEnd());
  }
#endif
#if defined(NCCL_MAJOR) && (NCCL_MAJOR < 2)
  (c10::cuda::getFreeMutex())->unlock();
#endif
}

```
- **EN**: Implements `NOLINTNEXTLINE`, one of the operational units in this file for the CUDA backend bindings, memory helpers, streams, events, and communication utilities.
- **CN**: 实现 `NOLINTNEXTLINE`，它是该文件中服务于CUDA 后端绑定、内存辅助逻辑、流、事件与通信工具的一个运行单元。

### Lines 458-476: Function `is_available` / 函数 `is_available`
```cpp
bool is_available(TensorList tensors) {
#ifdef USE_NCCL
  device_set devices;
  for (auto& tensor : tensors) {
    if (!tensor.is_cuda() || tensor.is_sparse())
      return false;
    if (!tensor.is_contiguous())
      return false;
    auto device = tensor.get_device();
    if (devices[device])
      return false;
    devices[device] = true;
  }
  return true;
#else
  return false;
#endif
}

```
- **EN**: Implements `is_available`, one of the operational units in this file for the CUDA backend bindings, memory helpers, streams, events, and communication utilities.
- **CN**: 实现 `is_available`，它是该文件中服务于CUDA 后端绑定、内存辅助逻辑、流、事件与通信工具的一个运行单元。

### Lines 477-489: Function `version` / 函数 `version`
```cpp
std::uint64_t version() {
#if defined(NCCL_MAJOR)
  constexpr std::uint64_t ver = (((uint64_t)NCCL_MAJOR) << 32) |
      (((uint64_t)NCCL_MINOR) << 16) | ((uint64_t)NCCL_PATCH);
  return ver;
#elif defined(USE_NCCL)
  // return major version "1"
  return ((uint64_t)1) << 32;
#else
  return 0;
#endif
}

```
- **EN**: Implements `version`, one of the operational units in this file for the CUDA backend bindings, memory helpers, streams, events, and communication utilities.
- **CN**: 实现 `version`，它是该文件中服务于CUDA 后端绑定、内存辅助逻辑、流、事件与通信工具的一个运行单元。

### Lines 490-497: Function `version_suffix` / 函数 `version_suffix`
```cpp
const char* version_suffix() {
#if defined(NCCL_SUFFIX)
  return NCCL_SUFFIX;
#else
  return "";
#endif
}

```
- **EN**: Implements `version_suffix`, one of the operational units in this file for the CUDA backend bindings, memory helpers, streams, events, and communication utilities.
- **CN**: 实现 `version_suffix`，它是该文件中服务于CUDA 后端绑定、内存辅助逻辑、流、事件与通信工具的一个运行单元。

### Lines 498-506: Function `get_unique_id` / 函数 `get_unique_id`
```cpp
void get_unique_id(ncclUniqueId& id) {
#ifdef USE_NCCL
  using namespace torch::cuda::nccl::detail;
  NCCL_CHECK(ncclGetUniqueId(to_nccl_unique_id(&id)));
#else
  TORCH_CHECK(false, "PyTorch built without NCCL support");
#endif
}

```
- **EN**: Implements `get_unique_id`, one of the operational units in this file for the CUDA backend bindings, memory helpers, streams, events, and communication utilities.
- **CN**: 实现 `get_unique_id`，它是该文件中服务于CUDA 后端绑定、内存辅助逻辑、流、事件与通信工具的一个运行单元。

### Lines 507-519: Function `comm_init_rank` / 函数 `comm_init_rank`
```cpp
ncclComm_t comm_init_rank(int nranks, const ncclUniqueId& comm_id, int rank) {
#ifdef USE_NCCL
  using namespace torch::cuda::nccl::detail;
  ncclComm_t comm = nullptr;
  ncclUniqueId id = comm_id;
  NCCL_CHECK(ncclCommInitRank(
      to_nccl_comm(&comm), nranks, *(to_nccl_unique_id(&id)), rank));
  return comm;
#else
  return nullptr;
#endif
}

```
- **EN**: Implements `comm_init_rank`, one of the operational units in this file for the CUDA backend bindings, memory helpers, streams, events, and communication utilities.
- **CN**: 实现 `comm_init_rank`，它是该文件中服务于CUDA 后端绑定、内存辅助逻辑、流、事件与通信工具的一个运行单元。

### Lines 520-530: Function `comm_destroy` / 函数 `comm_destroy`
```cpp
void comm_destroy(ncclComm_t comm) {
  /*
   * TODO(T30279827) Temporarily disable calling ncclCommDestroy
   * Calling ncclCommDestroy while program exiting is undefined
   * according to Nvidia, and lead to segfault in NCCL 2
   * (whether it is called before or after the CUDA runtime destructor).
   * Temporarily disable it in destructor to avoid segfault.
   * Following up with Nvidia for long term solution.
   */
  return;

```
- **EN**: Implements `comm_destroy`, one of the operational units in this file for the CUDA backend bindings, memory helpers, streams, events, and communication utilities.
- **CN**: 实现 `comm_destroy`，它是该文件中服务于CUDA 后端绑定、内存辅助逻辑、流、事件与通信工具的一个运行单元。

### Lines 531-536: Preprocessor configuration / 预处理配置
```cpp
#ifdef USE_NCCL
  using namespace torch::cuda::nccl::detail;
  NCCL_CHECK(ncclCommDestroy(to_nccl_comm(comm)));
#endif
}

```
- **EN**: Applies compile-time gating, macros, or platform-specific configuration so the file builds correctly under different feature sets.
- **CN**: 应用编译期开关、宏或平台特定配置，使该文件能在不同功能组合下正确构建。

### Lines 537-542: Namespace scope / 命名空间作用域
```cpp
namespace {
// NCCL changed the numerical type used for count between NCCL1 and NCCL2.
// So we use the following struct, which gets the type of the second argument
// of T, if T is a function type, with ncclBcast, to get that type statically
// and programmatically.

```
- **EN**: Opens or organizes namespace scopes so related symbols live under the expected PyTorch component hierarchy.
- **CN**: 打开或组织命名空间作用域，使相关符号位于预期的 PyTorch 组件层级中。

### Lines 543-545: Type declaration / 类型声明
```cpp
template <typename T>
struct GetSecondArgType;

```
- **EN**: Declares a core helper type, wrapper, or policy object that structures the rest of the implementation.
- **CN**: 声明一个核心辅助类型、封装对象或策略对象，用于组织后续实现。

### Lines 546-550: Type declaration / 类型声明
```cpp
template <typename R, typename Arg0, typename Arg1, typename... Args>
struct GetSecondArgType<R(Arg0, Arg1, Args...)> {
  typedef std::decay_t<Arg1> type;
};

```
- **EN**: Declares a core helper type, wrapper, or policy object that structures the rest of the implementation.
- **CN**: 声明一个核心辅助类型、封装对象或策略对象，用于组织后续实现。

### Lines 551-553: Supporting statements / 辅助语句
```cpp
constexpr auto count_max =
    std::numeric_limits<GetSecondArgType<decltype(ncclBcast)>::type>::max();

```
- **EN**: Contains local statements, constants, or helper glue that support the surrounding control flow and data movement.
- **CN**: 包含局部语句、常量或辅助胶水代码，用于支撑周围的控制流与数据传递。

### Lines 554-573: Supporting statements / 辅助语句
```cpp
// Since NCCL 2.12.10, NCCL supports send/recv 0 byte:
// https://github.com/NVIDIA/nccl/issues/696. The issue of skipping send/recv
// is that it can cause deadlock when a rank send and recv 0 bytes so it's
// completely skipping the collective, causing mismatch across ranks
#if defined(NCCL_MAJOR) && \
    ((NCCL_MAJOR > 2) || ((NCCL_MAJOR == 2) && (NCCL_MINOR > 13)))
template <typename T>
constexpr bool _nccl_should_send_recv([[maybe_unused]] T _unused_) {
  return true;
}
#else
// old NCCL uses 0 byte message for synchronization
// Avoid send/recv when message size is zero
template <typename T>
inline bool _nccl_should_send_recv(T value) {
  return value != 0;
}
#endif
} // namespace

```
- **EN**: Contains local statements, constants, or helper glue that support the surrounding control flow and data movement.
- **CN**: 包含局部语句、常量或辅助胶水代码，用于支撑周围的控制流与数据传递。

### Lines 574-577: Function `get_max_count` / 函数 `get_max_count`
```cpp
size_t get_max_count() {
  return count_max;
}

```
- **EN**: Implements `get_max_count`, one of the operational units in this file for the CUDA backend bindings, memory helpers, streams, events, and communication utilities.
- **CN**: 实现 `get_max_count`，它是该文件中服务于CUDA 后端绑定、内存辅助逻辑、流、事件与通信工具的一个运行单元。

### Lines 578-587: Supporting statements / 辅助语句
```cpp
void broadcast(
    TensorList tensors,
    const stream_list& streams,
    const comm_list& user_comms) {
#ifdef USE_NCCL
  using namespace torch::cuda::nccl::detail;
  check_inputs(tensors, tensors, 1, 1);
  auto data_type = to_nccl_data_type(tensors[0]);
  int64_t numel = tensors[0].numel();

```
- **EN**: Contains local statements, constants, or helper glue that support the surrounding control flow and data movement.
- **CN**: 包含局部语句、常量或辅助胶水代码，用于支撑周围的控制流与数据传递。

### Lines 588-590: Function `empty` / 函数 `empty`
```cpp
  const auto comms = user_comms.empty() ? get_communicators(tensors)
                                        : ArrayRef<ncclComm_t>(user_comms);

```
- **EN**: Implements `empty`, one of the operational units in this file for the CUDA backend bindings, memory helpers, streams, events, and communication utilities.
- **CN**: 实现 `empty`，它是该文件中服务于CUDA 后端绑定、内存辅助逻辑、流、事件与通信工具的一个运行单元。

### Lines 591-614: Supporting statements / 辅助语句
```cpp
  AutoNcclGroup nccl_group_guard;
  at::cuda::OptionalCUDAGuard device_guard;
  for (size_t i = 0, num_tensors = tensors.size(); i < num_tensors; i++) {
    auto device = tensors[i].get_device();
    device_guard.set_index(device);
    // Default to the current stream
    const auto stream = (streams.empty() || !streams[i])
        ? at::cuda::getCurrentCUDAStream(device).stream()
        : streams[i]->stream();
    TORCH_CHECK(
        static_cast<uint64_t>(numel) <= static_cast<uint64_t>(count_max),
        "Broadcast tensor has ",
        numel,
        " elements, which exceeds the "
        "maximum NCCL supports (",
        count_max,
        ")");
    ncclComm_t comm = comms[i];
    NCCL_CHECK(ncclBcast(
        tensors[i].mutable_data_ptr(),
        numel,
        data_type,
        0,
        to_nccl_comm(comm),
```
- **EN**: Contains local statements, constants, or helper glue that support the surrounding control flow and data movement.
- **CN**: 包含局部语句、常量或辅助胶水代码，用于支撑周围的控制流与数据传递。

### Lines 615-621: Supporting statements / 辅助语句
```cpp
        stream));
  }
#else
  TORCH_CHECK(false, "PyTorch built without NCCL support");
#endif
}

```
- **EN**: Contains local statements, constants, or helper glue that support the surrounding control flow and data movement.
- **CN**: 包含局部语句、常量或辅助胶水代码，用于支撑周围的控制流与数据传递。

### Lines 622-633: Supporting statements / 辅助语句
```cpp
void reduce(
    const std::vector<at::Tensor>& inputs,
    at::Tensor& output,
    int32_t root,
    int32_t op,
    const stream_list& streams,
    const comm_list& user_comms) {
#ifdef USE_NCCL
  using namespace torch::cuda::nccl::detail;
  TORCH_CHECK(
      root >= 0 && static_cast<size_t>(root) < inputs.size(), "invalid root");

```
- **EN**: Contains local statements, constants, or helper glue that support the surrounding control flow and data movement.
- **CN**: 包含局部语句、常量或辅助胶水代码，用于支撑周围的控制流与数据传递。

### Lines 634-636: Supporting statements / 辅助语句
```cpp
  check_inputs(inputs, output, root, 1, 1);
  const auto len = inputs.size();

```
- **EN**: Contains local statements, constants, or helper glue that support the surrounding control flow and data movement.
- **CN**: 包含局部语句、常量或辅助胶水代码，用于支撑周围的控制流与数据传递。

### Lines 637-638: Supporting statements / 辅助语句
```cpp
  auto data_type = to_nccl_data_type(inputs[0]);

```
- **EN**: Contains local statements, constants, or helper glue that support the surrounding control flow and data movement.
- **CN**: 包含局部语句、常量或辅助胶水代码，用于支撑周围的控制流与数据传递。

### Lines 639-642: Supporting statements / 辅助语句
```cpp
  const auto count = inputs[0].numel();
  auto comms_ref = user_comms.empty() ? get_communicators(inputs)
                                      : ArrayRef<ncclComm_t>(user_comms);

```
- **EN**: Contains local statements, constants, or helper glue that support the surrounding control flow and data movement.
- **CN**: 包含局部语句、常量或辅助胶水代码，用于支撑周围的控制流与数据传递。

### Lines 643-652: Supporting statements / 辅助语句
```cpp
  AutoNcclGroup nccl_group_guard;
  at::cuda::OptionalCUDAGuard device_guard;
  for (const auto i : c10::irange(len)) {
    auto device = inputs[i].device().index();
    device_guard.set_index(device);
    // Default to the current stream
    const auto stream = (streams.empty() || !streams[i])
        ? at::cuda::getCurrentCUDAStream(device).stream()
        : streams[i]->stream();

```
- **EN**: Contains local statements, constants, or helper glue that support the surrounding control flow and data movement.
- **CN**: 包含局部语句、常量或辅助胶水代码，用于支撑周围的控制流与数据传递。

### Lines 653-670: Supporting statements / 辅助语句
```cpp
    ncclComm_t comm = comms_ref[i];
    NCCL_CHECK(ncclReduce(
        inputs[i].const_data_ptr(),
        static_cast<std::remove_cv_t<decltype(i)>>(root) == i
            ? output.mutable_data_ptr()
            : nullptr,
        count,
        data_type,
        to_nccl_red_op(op),
        root,
        to_nccl_comm(comm),
        stream));
  }
#else
  TORCH_CHECK(false, "PyTorch built without NCCL support");
#endif
}

```
- **EN**: Contains local statements, constants, or helper glue that support the surrounding control flow and data movement.
- **CN**: 包含局部语句、常量或辅助胶水代码，用于支撑周围的控制流与数据传递。

### Lines 671-679: Supporting statements / 辅助语句
```cpp
void reduce(
    std::vector<at::Tensor>& inputs,
    int32_t root,
    int32_t op,
    const stream_list& streams,
    const comm_list& user_comms) {
  reduce(inputs, /*output=*/inputs[root], root, op, streams, user_comms);
}

```
- **EN**: Contains local statements, constants, or helper glue that support the surrounding control flow and data movement.
- **CN**: 包含局部语句、常量或辅助胶水代码，用于支撑周围的控制流与数据传递。

### Lines 680-690: Supporting statements / 辅助语句
```cpp
void all_reduce(
    const std::vector<at::Tensor>& inputs,
    std::vector<at::Tensor>& outputs,
    int32_t op,
    const stream_list& streams,
    const comm_list& user_comms) {
#ifdef USE_NCCL
  using namespace torch::cuda::nccl::detail;
  check_inputs(inputs, outputs, 1, 1);
  const auto len = inputs.size();

```
- **EN**: Contains local statements, constants, or helper glue that support the surrounding control flow and data movement.
- **CN**: 包含局部语句、常量或辅助胶水代码，用于支撑周围的控制流与数据传递。

### Lines 691-692: Supporting statements / 辅助语句
```cpp
  auto data_type = to_nccl_data_type(inputs[0]);

```
- **EN**: Contains local statements, constants, or helper glue that support the surrounding control flow and data movement.
- **CN**: 包含局部语句、常量或辅助胶水代码，用于支撑周围的控制流与数据传递。

### Lines 693-696: Supporting statements / 辅助语句
```cpp
  const auto count = inputs[0].numel();
  auto comms_ref = user_comms.empty() ? get_communicators(inputs)
                                      : ArrayRef<ncclComm_t>(user_comms);

```
- **EN**: Contains local statements, constants, or helper glue that support the surrounding control flow and data movement.
- **CN**: 包含局部语句、常量或辅助胶水代码，用于支撑周围的控制流与数据传递。

### Lines 697-706: Supporting statements / 辅助语句
```cpp
  AutoNcclGroup nccl_group_guard;
  at::cuda::OptionalCUDAGuard device_guard;
  for (const auto i : c10::irange(len)) {
    auto device = inputs[i].device().index();
    device_guard.set_index(device);
    // Default to the current stream
    const auto stream = (streams.empty() || !streams[i])
        ? at::cuda::getCurrentCUDAStream(device).stream()
        : streams[i]->stream();

```
- **EN**: Contains local statements, constants, or helper glue that support the surrounding control flow and data movement.
- **CN**: 包含局部语句、常量或辅助胶水代码，用于支撑周围的控制流与数据传递。

### Lines 707-721: Supporting statements / 辅助语句
```cpp
    ncclComm_t comm = comms_ref[i];
    NCCL_CHECK(ncclAllReduce(
        inputs[i].const_data_ptr(),
        outputs[i].mutable_data_ptr(),
        count,
        data_type,
        to_nccl_red_op(op),
        to_nccl_comm(comm),
        stream));
  }
#else
  TORCH_CHECK(false, "PyTorch built without NCCL support");
#endif
}

```
- **EN**: Contains local statements, constants, or helper glue that support the surrounding control flow and data movement.
- **CN**: 包含局部语句、常量或辅助胶水代码，用于支撑周围的控制流与数据传递。

### Lines 722-732: Supporting statements / 辅助语句
```cpp
void reduce_scatter(
    const std::vector<at::Tensor>& inputs,
    std::vector<at::Tensor>& outputs,
    int32_t op,
    const stream_list& streams,
    const comm_list& user_comms) {
#ifdef USE_NCCL
  using namespace torch::cuda::nccl::detail;
  const auto len = inputs.size();
  check_inputs(inputs, outputs, 1, len);

```
- **EN**: Contains local statements, constants, or helper glue that support the surrounding control flow and data movement.
- **CN**: 包含局部语句、常量或辅助胶水代码，用于支撑周围的控制流与数据传递。

### Lines 733-734: Supporting statements / 辅助语句
```cpp
  auto data_type = to_nccl_data_type(inputs[0]);

```
- **EN**: Contains local statements, constants, or helper glue that support the surrounding control flow and data movement.
- **CN**: 包含局部语句、常量或辅助胶水代码，用于支撑周围的控制流与数据传递。

### Lines 735-738: Supporting statements / 辅助语句
```cpp
  const auto count = inputs[0].numel() / len;
  auto comms_ref = user_comms.empty() ? get_communicators(inputs)
                                      : ArrayRef<ncclComm_t>(user_comms);

```
- **EN**: Contains local statements, constants, or helper glue that support the surrounding control flow and data movement.
- **CN**: 包含局部语句、常量或辅助胶水代码，用于支撑周围的控制流与数据传递。

### Lines 739-748: Supporting statements / 辅助语句
```cpp
  AutoNcclGroup nccl_group_guard;
  at::cuda::OptionalCUDAGuard device_guard;
  for (const auto i : c10::irange(len)) {
    auto device = inputs[i].device().index();
    device_guard.set_index(device);
    // Default to the current stream
    const auto stream = (streams.empty() || !streams[i])
        ? at::cuda::getCurrentCUDAStream(device).stream()
        : streams[i]->stream();

```
- **EN**: Contains local statements, constants, or helper glue that support the surrounding control flow and data movement.
- **CN**: 包含局部语句、常量或辅助胶水代码，用于支撑周围的控制流与数据传递。

### Lines 749-763: Supporting statements / 辅助语句
```cpp
    ncclComm_t comm = comms_ref[i];
    NCCL_CHECK(ncclReduceScatter(
        inputs[i].const_data_ptr(),
        outputs[i].mutable_data_ptr(),
        count,
        data_type,
        to_nccl_red_op(op),
        to_nccl_comm(comm),
        stream));
  }
#else
  TORCH_CHECK(false, "PyTorch built without NCCL support");
#endif
}

```
- **EN**: Contains local statements, constants, or helper glue that support the surrounding control flow and data movement.
- **CN**: 包含局部语句、常量或辅助胶水代码，用于支撑周围的控制流与数据传递。

### Lines 764-773: Supporting statements / 辅助语句
```cpp
void all_gather(
    const std::vector<at::Tensor>& inputs,
    std::vector<at::Tensor>& outputs,
    const stream_list& streams,
    const comm_list& user_comms) {
#ifdef USE_NCCL
  using namespace torch::cuda::nccl::detail;
  const auto len = inputs.size();
  check_inputs(inputs, outputs, len, 1);

```
- **EN**: Contains local statements, constants, or helper glue that support the surrounding control flow and data movement.
- **CN**: 包含局部语句、常量或辅助胶水代码，用于支撑周围的控制流与数据传递。

### Lines 774-775: Supporting statements / 辅助语句
```cpp
  auto data_type = to_nccl_data_type(inputs[0]);

```
- **EN**: Contains local statements, constants, or helper glue that support the surrounding control flow and data movement.
- **CN**: 包含局部语句、常量或辅助胶水代码，用于支撑周围的控制流与数据传递。

### Lines 776-779: Supporting statements / 辅助语句
```cpp
  const auto count = inputs[0].numel();
  auto comms_ref = user_comms.empty() ? get_communicators(inputs)
                                      : ArrayRef<ncclComm_t>(user_comms);

```
- **EN**: Contains local statements, constants, or helper glue that support the surrounding control flow and data movement.
- **CN**: 包含局部语句、常量或辅助胶水代码，用于支撑周围的控制流与数据传递。

### Lines 780-789: Supporting statements / 辅助语句
```cpp
  AutoNcclGroup nccl_group_guard;
  at::cuda::OptionalCUDAGuard device_guard;
  for (const auto i : c10::irange(len)) {
    auto device = inputs[i].device().index();
    device_guard.set_index(device);
    // Default to the current stream
    const auto stream = (streams.empty() || !streams[i])
        ? at::cuda::getCurrentCUDAStream(device).stream()
        : streams[i]->stream();

```
- **EN**: Contains local statements, constants, or helper glue that support the surrounding control flow and data movement.
- **CN**: 包含局部语句、常量或辅助胶水代码，用于支撑周围的控制流与数据传递。

### Lines 790-813: Supporting statements / 辅助语句
```cpp
    ncclComm_t comm = comms_ref[i];
#if defined(NCCL_MAJOR) && (NCCL_MAJOR >= 2)
    NCCL_CHECK(ncclAllGather(
        inputs[i].const_data_ptr(),
        outputs[i].mutable_data_ptr(),
        count,
        data_type,
        to_nccl_comm(comm),
        stream));
#else
    NCCL_CHECK(ncclAllGather(
        inputs[i].const_data_ptr(),
        count,
        data_type,
        outputs[i].mutable_data_ptr(),
        to_nccl_comm(comm),
        stream));
#endif
  }
#else
  TORCH_CHECK(false, "PyTorch built without NCCL support");
#endif
}

```
- **EN**: Contains local statements, constants, or helper glue that support the surrounding control flow and data movement.
- **CN**: 包含局部语句、常量或辅助胶水代码，用于支撑周围的控制流与数据传递。

### Lines 814-824: Supporting statements / 辅助语句
```cpp
void all2all_single_equal_split(
    at::Tensor& input,
    at::Tensor& output,
    int size,
    ncclComm_t _comm,
    at::cuda::CUDAStream& stream) {
#ifdef USE_NCCL
#if defined(NCCL_MAJOR) && \
    ((NCCL_MAJOR > 2) || ((NCCL_MAJOR == 2) && (NCCL_MINOR >= 7)))
  using namespace torch::cuda::nccl::detail;

```
- **EN**: Contains local statements, constants, or helper glue that support the surrounding control flow and data movement.
- **CN**: 包含局部语句、常量或辅助胶水代码，用于支撑周围的控制流与数据传递。

### Lines 825-848: Supporting statements / 辅助语句
```cpp
  auto type = to_nccl_data_type(input);
  size_t count = input.numel() / size;
  [[maybe_unused]] size_t rankdiff = input.nbytes() / size;
  const auto* sendbuff = reinterpret_cast<const char*>(input.const_data_ptr());
  auto* recvbuff = reinterpret_cast<char*>(output.mutable_data_ptr());
  auto comm = to_nccl_comm(_comm);
#if defined(USE_ROCM) || defined(NCCL_ALLTOALL_SUPPORTED)
  // NCCL_ALLTOALL_SUPPORTED is used so NCCL can differentiate send/recv
  // operations issued as a part of the collective (e.g. alltoall) vs those
  // inside traditional p2p operations.
  NCCL_CHECK(ncclAllToAll(sendbuff, recvbuff, count, type, comm, stream));
#elif NCCL_VERSION_CODE >= NCCL_VERSION(2, 28, 0)
  NCCL_CHECK(ncclAlltoAll(sendbuff, recvbuff, count, type, comm, stream));
#else
  int numranks = 0;
  NCCL_CHECK(ncclCommCount(comm, &numranks));
  NCCL_CHECK(ncclGroupStart());
  for (const auto r : c10::irange(numranks)) {
    if (_nccl_should_send_recv(count)) {
      NCCL_CHECK(
          ncclSend(sendbuff + r * rankdiff, count, type, r, comm, stream));
      NCCL_CHECK(
          ncclRecv(recvbuff + r * rankdiff, count, type, r, comm, stream));
    }
```
- **EN**: Contains local statements, constants, or helper glue that support the surrounding control flow and data movement.
- **CN**: 包含局部语句、常量或辅助胶水代码，用于支撑周围的控制流与数据传递。

### Lines 849-863: Supporting statements / 辅助语句
```cpp
  }
#ifndef NCCL_HAS_COMM_NONBLOCKING
  NCCL_CHECK(ncclGroupEnd());
#else
  NCCL_CHECK_TIMEOUT(ncclGroupEnd(), _comm);
#endif
#endif
#else
  TORCH_CHECK(false, "all2all is only supported for NCCL lib version >= 2.7.0");
#endif
#else
  TORCH_CHECK(false, "PyTorch built without NCCL support");
#endif
}

```
- **EN**: Contains local statements, constants, or helper glue that support the surrounding control flow and data movement.
- **CN**: 包含局部语句、常量或辅助胶水代码，用于支撑周围的控制流与数据传递。

### Lines 864-879: Supporting statements / 辅助语句
```cpp
void all2all_single_unequal_split(
    void* sendbuff,
    const size_t* sendcounts,
    const size_t* senddispls,
    void* recvbuff,
    const size_t* recvcounts,
    const size_t* recvdispls,
    size_t size,
    c10::ScalarType _type,
    ncclComm_t _comm,
    at::cuda::CUDAStream& stream) {
#ifdef USE_NCCL
#if defined(NCCL_MAJOR) && \
    ((NCCL_MAJOR > 2) || ((NCCL_MAJOR == 2) && (NCCL_MINOR >= 7)))
  using namespace torch::cuda::nccl::detail;

```
- **EN**: Contains local statements, constants, or helper glue that support the surrounding control flow and data movement.
- **CN**: 包含局部语句、常量或辅助胶水代码，用于支撑周围的控制流与数据传递。

### Lines 880-903: Supporting statements / 辅助语句
```cpp
  auto type = to_nccl_data_type(_type);
  auto comm = to_nccl_comm(_comm);
#if defined(USE_ROCM) || defined(NCCL_ALLTOALLV_SUPPORTED)
  // NCCL_ALLTOALLV_SUPPORTED is used so NCCL can differentiate send/recv
  // operations issued as a part of the collective (e.g. alltoallv) vs those
  // inside traditional p2p operations.
  NCCL_CHECK(ncclAllToAllv(
      sendbuff,
      sendcounts,
      senddispls,
      recvbuff,
      recvcounts,
      recvdispls,
      type,
      comm,
      stream.stream()));
#else
  int numranks = 0;
  NCCL_CHECK(ncclCommCount(comm, &numranks));
  NCCL_CHECK(ncclGroupStart());
  for (const auto r : c10::irange(numranks)) {
    if (_nccl_should_send_recv(sendcounts[r])) {
      NCCL_CHECK(ncclSend(
          ((char*)sendbuff) + senddispls[r] * size,
```
- **EN**: Contains local statements, constants, or helper glue that support the surrounding control flow and data movement.
- **CN**: 包含局部语句、常量或辅助胶水代码，用于支撑周围的控制流与数据传递。

### Lines 904-927: Supporting statements / 辅助语句
```cpp
          sendcounts[r],
          type,
          r,
          comm,
          stream));
    }
    if (_nccl_should_send_recv(recvcounts[r])) {
      NCCL_CHECK(ncclRecv(
          ((char*)recvbuff) + recvdispls[r] * size,
          recvcounts[r],
          type,
          r,
          comm,
          stream));
    }
  }
#ifndef NCCL_HAS_COMM_NONBLOCKING
  NCCL_CHECK(ncclGroupEnd());
#else
  NCCL_CHECK_TIMEOUT(ncclGroupEnd(), _comm);
#endif
#endif
#else
  TORCH_CHECK(false, "all2all is only supported for NCCL lib version >= 2.7.0");
```
- **EN**: Contains local statements, constants, or helper glue that support the surrounding control flow and data movement.
- **CN**: 包含局部语句、常量或辅助胶水代码，用于支撑周围的控制流与数据传递。

### Lines 928-933: Preprocessor configuration / 预处理配置
```cpp
#endif
#else
  TORCH_CHECK(false, "PyTorch built without NCCL support");
#endif
}

```
- **EN**: Applies compile-time gating, macros, or platform-specific configuration so the file builds correctly under different feature sets.
- **CN**: 应用编译期开关、宏或平台特定配置，使该文件能在不同功能组合下正确构建。

### Lines 934-944: Supporting statements / 辅助语句
```cpp
void all2all(
    std::vector<at::Tensor>& outputTensors,
    std::vector<at::Tensor>& inputTensors,
    ncclComm_t _comm,
    at::cuda::CUDAStream& stream) {
#ifdef USE_NCCL
#if defined(NCCL_MAJOR) && \
    ((NCCL_MAJOR > 2) || ((NCCL_MAJOR == 2) && (NCCL_MINOR >= 7)))
  using namespace torch::cuda::nccl::detail;
  auto comm = to_nccl_comm(_comm);

```
- **EN**: Contains local statements, constants, or helper glue that support the surrounding control flow and data movement.
- **CN**: 包含局部语句、常量或辅助胶水代码，用于支撑周围的控制流与数据传递。

### Lines 945-949: Supporting statements / 辅助语句
```cpp
  NCCL_CHECK(ncclGroupStart());
  for (const int r : c10::irange(static_cast<int>(outputTensors.size()))) {
    at::Tensor& input = inputTensors[r];
    at::Tensor& output = outputTensors[r];

```
- **EN**: Contains local statements, constants, or helper glue that support the surrounding control flow and data movement.
- **CN**: 包含局部语句、常量或辅助胶水代码，用于支撑周围的控制流与数据传递。

### Lines 950-973: Supporting statements / 辅助语句
```cpp
    if (_nccl_should_send_recv(input.numel())) {
      NCCL_CHECK(ncclSend(
          input.const_data_ptr(),
          input.numel(),
          to_nccl_data_type(input),
          r,
          comm,
          stream.stream()));
    }
    if (_nccl_should_send_recv(output.numel())) {
      NCCL_CHECK(ncclRecv(
          output.mutable_data_ptr(),
          output.numel(),
          to_nccl_data_type(output),
          r,
          comm,
          stream.stream()));
    }
  }
#ifndef NCCL_HAS_COMM_NONBLOCKING
  NCCL_CHECK(ncclGroupEnd());
#else
  NCCL_CHECK_TIMEOUT(ncclGroupEnd(), _comm);
#endif
```
- **EN**: Contains local statements, constants, or helper glue that support the surrounding control flow and data movement.
- **CN**: 包含局部语句、常量或辅助胶水代码，用于支撑周围的控制流与数据传递。

### Lines 974-981: Preprocessor configuration / 预处理配置
```cpp
#else
  TORCH_CHECK(false, "all2all is only supported for NCCL lib version >= 2.7.0");
#endif
#else
  TORCH_CHECK(false, "PyTorch built without NCCL support");
#endif
}

```
- **EN**: Applies compile-time gating, macros, or platform-specific configuration so the file builds correctly under different feature sets.
- **CN**: 应用编译期开关、宏或平台特定配置，使该文件能在不同功能组合下正确构建。

### Lines 982-1005: Supporting statements / 辅助语句
```cpp
void send(
    const at::Tensor& input,
    ncclComm_t comm,
    at::cuda::CUDAStream stream,
    int dst) {
#ifdef USE_NCCL
#if defined(NCCL_MAJOR) && \
    ((NCCL_MAJOR > 2) || ((NCCL_MAJOR == 2) && (NCCL_MINOR >= 7)))
  using namespace torch::cuda::nccl::detail;
#ifndef NCCL_HAS_COMM_NONBLOCKING
  NCCL_CHECK(ncclSend(
      input.const_data_ptr(),
      input.numel(),
      to_nccl_data_type(input),
      dst,
      to_nccl_comm(comm),
      stream.stream()));
#else
  NCCL_CHECK_TIMEOUT(
      ncclSend(
          input.const_data_ptr(),
          input.numel(),
          to_nccl_data_type(input),
          dst,
```
- **EN**: Contains local statements, constants, or helper glue that support the surrounding control flow and data movement.
- **CN**: 包含局部语句、常量或辅助胶水代码，用于支撑周围的控制流与数据传递。

### Lines 1006-1017: Supporting statements / 辅助语句
```cpp
          to_nccl_comm(comm),
          stream.stream()),
      comm);
#endif
#else
  TORCH_CHECK(false, "Send is only supported for NCCL lib version >= 2.7.0");
#endif
#else
  TORCH_CHECK(false, "PyTorch built without NCCL support");
#endif
}

```
- **EN**: Contains local statements, constants, or helper glue that support the surrounding control flow and data movement.
- **CN**: 包含局部语句、常量或辅助胶水代码，用于支撑周围的控制流与数据传递。

### Lines 1018-1041: Supporting statements / 辅助语句
```cpp
void recv(
    at::Tensor& output,
    ncclComm_t comm,
    at::cuda::CUDAStream stream,
    int src) {
#ifdef USE_NCCL
#if defined(NCCL_MAJOR) && \
    ((NCCL_MAJOR > 2) || ((NCCL_MAJOR == 2) && (NCCL_MINOR >= 7)))
  using namespace torch::cuda::nccl::detail;
#ifndef NCCL_HAS_COMM_NONBLOCKING
  NCCL_CHECK(ncclRecv(
      output.mutable_data_ptr(),
      output.numel(),
      to_nccl_data_type(output),
      src,
      to_nccl_comm(comm),
      stream.stream()));
#else
  NCCL_CHECK_TIMEOUT(
      ncclRecv(
          output.mutable_data_ptr(),
          output.numel(),
          to_nccl_data_type(output),
          src,
```
- **EN**: Contains local statements, constants, or helper glue that support the surrounding control flow and data movement.
- **CN**: 包含局部语句、常量或辅助胶水代码，用于支撑周围的控制流与数据传递。

### Lines 1042-1053: Supporting statements / 辅助语句
```cpp
          to_nccl_comm(comm),
          stream.stream()),
      comm);
#endif
#else
  TORCH_CHECK(false, "Recv is only supported for NCCL lib version >= 2.7.0");
#endif
#else
  TORCH_CHECK(false, "PyTorch built without NCCL support");
#endif
}

```
- **EN**: Contains local statements, constants, or helper glue that support the surrounding control flow and data movement.
- **CN**: 包含局部语句、常量或辅助胶水代码，用于支撑周围的控制流与数据传递。

### Lines 1054-1064: Supporting statements / 辅助语句
```cpp
void gather(
    const at::Tensor& inputs,
    std::vector<at::Tensor>& outputs,
    ncclComm_t _comm,
    at::cuda::CUDAStream& stream,
    int32_t root) {
#ifdef USE_NCCL
#if defined(NCCL_MAJOR) && \
    ((NCCL_MAJOR > 2) || ((NCCL_MAJOR == 2) && (NCCL_MINOR >= 7)))
  using namespace torch::cuda::nccl::detail;

```
- **EN**: Contains local statements, constants, or helper glue that support the surrounding control flow and data movement.
- **CN**: 包含局部语句、常量或辅助胶水代码，用于支撑周围的控制流与数据传递。

### Lines 1065-1069: Supporting statements / 辅助语句
```cpp
  auto comm = to_nccl_comm(_comm);
  int numranks = 0, cur_rank = 0;
  NCCL_CHECK(ncclCommCount(comm, &numranks));
  NCCL_CHECK(ncclCommUserRank(comm, &cur_rank));

```
- **EN**: Contains local statements, constants, or helper glue that support the surrounding control flow and data movement.
- **CN**: 包含局部语句、常量或辅助胶水代码，用于支撑周围的控制流与数据传递。

### Lines 1070-1073: Supporting statements / 辅助语句
```cpp
  size_t count = inputs.numel();
  auto type = to_nccl_data_type(inputs);
  const auto* sendbuff = reinterpret_cast<const char*>(inputs.const_data_ptr());

```
- **EN**: Contains local statements, constants, or helper glue that support the surrounding control flow and data movement.
- **CN**: 包含局部语句、常量或辅助胶水代码，用于支撑周围的控制流与数据传递。

### Lines 1074-1075: Supporting statements / 辅助语句
```cpp
  NCCL_CHECK(ncclGroupStart());

```
- **EN**: Contains local statements, constants, or helper glue that support the surrounding control flow and data movement.
- **CN**: 包含局部语句、常量或辅助胶水代码，用于支撑周围的控制流与数据传递。

### Lines 1076-1094: Supporting statements / 辅助语句
```cpp
  if (cur_rank == root) {
    for (const auto r : c10::irange(numranks)) {
      if (r != root) {
        auto* recvbuff = reinterpret_cast<char*>(outputs[r].mutable_data_ptr());
        NCCL_CHECK(ncclRecv(recvbuff, count, type, r, comm, stream));
      } else {
        // on its own rank, simply copy from the input
        outputs[r].copy_(inputs);
      }
    }
  } else {
    NCCL_CHECK(ncclSend(sendbuff, count, type, root, comm, stream));
  }
#ifndef NCCL_HAS_COMM_NONBLOCKING
  NCCL_CHECK(ncclGroupEnd());
#else
  NCCL_CHECK_TIMEOUT(ncclGroupEnd(), _comm);
#endif

```
- **EN**: Contains local statements, constants, or helper glue that support the surrounding control flow and data movement.
- **CN**: 包含局部语句、常量或辅助胶水代码，用于支撑周围的控制流与数据传递。

### Lines 1095-1102: Preprocessor configuration / 预处理配置
```cpp
#else
  TORCH_CHECK(false, "gather is only supported for NCCL lib version >= 2.7.0");
#endif
#else
  TORCH_CHECK(false, "PyTorch built without NCCL support");
#endif
}

```
- **EN**: Applies compile-time gating, macros, or platform-specific configuration so the file builds correctly under different feature sets.
- **CN**: 应用编译期开关、宏或平台特定配置，使该文件能在不同功能组合下正确构建。

### Lines 1103-1113: Supporting statements / 辅助语句
```cpp
void scatter(
    const std::vector<at::Tensor>& inputs,
    at::Tensor& outputs,
    ncclComm_t _comm,
    at::cuda::CUDAStream& stream,
    int32_t root) {
#ifdef USE_NCCL
#if defined(NCCL_MAJOR) && \
    ((NCCL_MAJOR > 2) || ((NCCL_MAJOR == 2) && (NCCL_MINOR >= 7)))
  using namespace torch::cuda::nccl::detail;

```
- **EN**: Contains local statements, constants, or helper glue that support the surrounding control flow and data movement.
- **CN**: 包含局部语句、常量或辅助胶水代码，用于支撑周围的控制流与数据传递。

### Lines 1114-1137: Supporting statements / 辅助语句
```cpp
  auto comm = to_nccl_comm(_comm);
  int numranks = 0, cur_rank = 0;
#ifndef NCCL_HAS_COMM_NONBLOCKING
  NCCL_CHECK(ncclCommCount(comm, &numranks));
  NCCL_CHECK(ncclCommUserRank(comm, &cur_rank));
#else
  NCCL_CHECK_TIMEOUT(ncclCommCount(comm, &numranks), _comm);
  NCCL_CHECK_TIMEOUT(ncclCommUserRank(comm, &cur_rank), _comm);
#endif
  NCCL_CHECK(ncclGroupStart());
  if (cur_rank == root) {
    for (const auto r : c10::irange(numranks)) {
      if (r != root) {
        size_t send_count = inputs[r].numel();
        auto send_type = to_nccl_data_type(inputs[r]);
        const auto* sendbuff =
            reinterpret_cast<const char*>(inputs[r].const_data_ptr());
        NCCL_CHECK(ncclSend(sendbuff, send_count, send_type, r, comm, stream));
      } else {
        // on its own rank, simply copy it to the output
        outputs.copy_(inputs[r]);
      }
    }
  } else {
```
- **EN**: Contains local statements, constants, or helper glue that support the surrounding control flow and data movement.
- **CN**: 包含局部语句、常量或辅助胶水代码，用于支撑周围的控制流与数据传递。

### Lines 1138-1155: Supporting statements / 辅助语句
```cpp
    size_t recv_count = outputs.numel();
    auto recv_type = to_nccl_data_type(outputs);
    auto* recvbuff = reinterpret_cast<char*>(outputs.mutable_data_ptr());
    NCCL_CHECK(ncclRecv(recvbuff, recv_count, recv_type, root, comm, stream));
  }
#ifndef NCCL_HAS_COMM_NONBLOCKING
  NCCL_CHECK(ncclGroupEnd());
#else
  NCCL_CHECK_TIMEOUT(ncclGroupEnd(), _comm);
#endif
#else
  TORCH_CHECK(false, "scatter is only supported for NCCL lib version >= 2.7.0");
#endif
#else
  TORCH_CHECK(false, "PyTorch built without NCCL support");
#endif
}

```
- **EN**: Contains local statements, constants, or helper glue that support the surrounding control flow and data movement.
- **CN**: 包含局部语句、常量或辅助胶水代码，用于支撑周围的控制流与数据传递。

### Lines 1156-1156: Supporting statements / 辅助语句
```cpp
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
- `ATen/core/functional.h`
- `torch/csrc/cuda/device_set.h`
- `torch/csrc/cuda/nccl.h`
- `ATen/ATen.h`
- `c10/cuda/CUDAException.h`
- `c10/cuda/CUDAGuard.h`
- `c10/util/Exception.h`
- `c10/util/hash.h`
- `c10/util/irange.h`
### External / 外部
- `nccl.h`
- `sched.h`
- `limits`
- `sstream`
- `type_traits`
- `unordered_map`
