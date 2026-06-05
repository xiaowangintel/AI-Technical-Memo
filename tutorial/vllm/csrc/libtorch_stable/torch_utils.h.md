# torch_utils.h — Code Analysis / 代码分析

## Source / 来源
- **File**: `csrc/libtorch_stable/torch_utils.h`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Utility header providing stable ABI-compatible device property queries and CUDA stream/cuBLAS handle access for vLLM CUDA kernels. / [CN] 提供稳定 ABI 兼容的设备属性查询和 CUDA 流/cuBLAS 句柄访问的实用头文件，供 vLLM CUDA 内核使用。

## Line-by-Line Analysis / 逐行分析

### Header Guard and Includes / 头文件保护和包含
```cpp
#pragma once
#include <torch/csrc/inductor/aoti_torch/c/shim.h>
#include <torch/csrc/stable/accelerator.h>
#include <torch/csrc/stable/ops.h>
#include <torch/csrc/stable/tensor.h>
#include <torch/headeronly/util/shim_utils.h>
#include <cublas_v2.h>
#include <cuda_runtime.h>
#include <deque>
#include <mutex>
#include <string>
#include <vector>
```
**EN:** Includes PyTorch stable ABI headers, CUDA runtime/cuBLAS APIs, and STL containers for caching. The stable ABI headers ensure cross-version compatibility.  
**CN:** 包含 PyTorch 稳定 ABI 头文件、CUDA 运行时/cuBLAS API 和用于缓存的 STL 容器。稳定 ABI 头文件确保跨版本兼容性。

### Custom Check Macro / 自定义检查宏
```cpp
// Stable ABI equivalent of TORCH_CHECK_NOT_IMPLEMENTED.
#define STD_TORCH_CHECK_NOT_IMPLEMENTED(cond, ...) \
  STD_TORCH_CHECK(cond, "NotImplementedError: ", __VA_ARGS__)
```
**EN:** Defines a stable ABI-compatible macro for checking unimplemented features, wrapping `STD_TORCH_CHECK` with a "NotImplementedError" prefix.  
**CN:** 定义稳定 ABI 兼容的宏以检查未实现的功能，用 "NotImplementedError" 前缀包装 `STD_TORCH_CHECK`。

### Device Properties Cache / 设备属性缓存
```cpp
// Device properties cache for stable ABI compatibility.
// Uses raw CUDA/HIP APIs instead of ATen functions.
// Using inline ensures a single instance across all translation units.
inline std::deque<std::once_flag> device_flags;
inline std::vector<cudaDeviceProp> device_properties;
inline std::once_flag vectors_init_flag;
```
**EN:** Global inline variables (C++17) for caching device properties. Uses `std::once_flag` for thread-safe lazy initialization per device, avoiding repeated CUDA API calls.  
**CN:** 用于缓存设备属性的全局内联变量（C++17）。使用 `std::once_flag` 实现每设备的线程安全延迟初始化，避免重复的 CUDA API 调用。

### Vector Initialization / 向量初始化
```cpp
inline void do_init_device_vectors() {
  int device_count;
  cudaError_t err = cudaGetDeviceCount(&device_count);
  if (err != cudaSuccess) {
    STD_TORCH_CHECK(false, "cudaGetDeviceCount failed: " +
                               std::string(cudaGetErrorString(err)));
  }
  device_flags.resize(device_count);
  device_properties.resize(device_count);
}

inline void initDeviceVectors() {
  std::call_once(vectors_init_flag, do_init_device_vectors);
}
```
**EN:** Initializes device property caches exactly once using `std::call_once`. Queries total device count via raw CUDA API and resizes storage vectors accordingly.  
**CN:** 使用 `std::call_once` 恰好初始化一次设备属性缓存。通过原始 CUDA API 查询总设备数并相应地调整存储向量大小。

### Device Property Initialization / 设备属性初始化
```cpp
inline void initDeviceProperty(int device_index) {
  cudaDeviceProp device_prop{};
  cudaError_t err = cudaGetDeviceProperties(&device_prop, device_index);
  if (err != cudaSuccess) {
    STD_TORCH_CHECK(false, "cudaGetDeviceProperties failed: " +
                               std::string(cudaGetErrorString(err)));
  }
  device_properties[device_index] = device_prop;
}
```
**EN:** Fetches properties for a specific device using `cudaGetDeviceProperties` and caches the result. Called exactly once per device via `std::call_once`.  
**CN:** 使用 `cudaGetDeviceProperties` 获取特定设备的属性并缓存结果。通过 `std::call_once` 对每个设备恰好调用一次。

### get_device_prop Function / get_device_prop 函数
```cpp
// Get device properties using raw CUDA/HIP APIs (stable ABI compatible).
// Caches results per device so cudaGetDeviceProperties is called at most once
// per device.
inline cudaDeviceProp* get_device_prop() {
  initDeviceVectors();
  int device_index;
  cudaError_t err = cudaGetDevice(&device_index);
  if (err != cudaSuccess) {
    STD_TORCH_CHECK(
        false, "cudaGetDevice failed: " + std::string(cudaGetErrorString(err)));
  }
  STD_TORCH_CHECK(device_index >= 0 && static_cast<size_t>(device_index) <
                                           device_properties.size(),
                  "CUDA device index " + std::to_string(device_index) +
                      " out of range [0, " +
                      std::to_string(device_properties.size()) + ")");

  std::call_once(device_flags[device_index], initDeviceProperty, device_index);
  return &device_properties[device_index];
}
```
**EN:** Returns cached device properties for the current CUDA device. Key features:
- Uses raw CUDA APIs (`cudaGetDevice`) instead of ATen functions for stable ABI compatibility
- Lazy initialization: properties fetched on first access per device
- Thread-safe via `std::call_once`
- Bounds checking to prevent out-of-range access

**CN:** 返回当前 CUDA 设备的缓存设备属性。关键特性：
- 使用原始 CUDA API（`cudaGetDevice`）而非 ATen 函数以实现稳定 ABI 兼容性
- 延迟初始化：每个设备首次访问时获取属性
- 通过 `std::call_once` 实现线程安全
- 边界检查以防止越界访问

### get_current_cuda_stream Function / get_current_cuda_stream 函数
```cpp
// Utility to get the current CUDA stream for a given device using stable APIs.
// Returns a cudaStream_t for use in kernel launches.
inline cudaStream_t get_current_cuda_stream(int32_t device_index = -1) {
  void* stream_ptr = nullptr;
  TORCH_ERROR_CODE_CHECK(
      aoti_torch_get_current_cuda_stream(device_index, &stream_ptr));
  return reinterpret_cast<cudaStream_t>(stream_ptr);
}
```
**EN:** Retrieves the current CUDA stream for kernel launches using PyTorch's stable AOTI (Ahead-Of-Time Inductor) API. The default `device_index = -1` queries the current device. Returns a raw `cudaStream_t` for use in `<<<grid, block, smem, stream>>>` kernel launches.  
**CN:** 使用 PyTorch 的稳定 AOTI（提前感应器）API 检索用于内核启动的当前 CUDA 流。默认 `device_index = -1` 查询当前设备。返回原始 `cudaStream_t` 用于 `<<<grid, block, smem, stream>>>` 内核启动。

### get_current_cuda_blas_handle Function / get_current_cuda_blas_handle 函数
```cpp
// Utility to get the current cuBLAS handle using stable APIs.
inline cublasHandle_t get_current_cuda_blas_handle() {
  void* blas_handle_ptr = nullptr;
  TORCH_ERROR_CODE_CHECK(torch_get_current_cuda_blas_handle(&blas_handle_ptr));
  return reinterpret_cast<cublasHandle_t>(blas_handle_ptr);
}
```
**EN:** Retrieves the current cuBLAS handle for BLAS operations using stable APIs. Essential for kernels performing matrix operations via cuBLAS calls (e.g., GEMM operations in quantized matrix multiplication).  
**CN:** 使用稳定 API 检索用于 BLAS 操作的当前 cuBLAS 句柄。对于通过 cuBLAS 调用执行矩阵操作的内核（例如，量化矩阵乘法中的 GEMM 操作）至关重要。

## Key Concepts / 关键概念

### EN:
- **Stable ABI Compatibility**: All utilities use raw CUDA APIs or PyTorch stable APIs instead of ATen functions to avoid ABI breakage across PyTorch versions
- **Lazy Initialization Pattern**: Device properties fetched on-demand using `std::call_once` for thread-safe, efficient caching
- **Per-Device Caching**: Separate `std::once_flag` for each device enables independent lazy initialization
- **Inline Variables (C++17)**: Global caches declared `inline` ensure single definition across translation units without violating ODR
- **Error Handling**: All CUDA API calls checked with `STD_TORCH_CHECK`, providing descriptive error messages
- **Resource Reuse**: Stream and cuBLAS handle queries leverage PyTorch's internal resource management for optimal GPU utilization

### CN:
- **稳定 ABI 兼容性**：所有实用工具使用原始 CUDA API 或 PyTorch 稳定 API，而非 ATen 函数，以避免跨 PyTorch 版本的 ABI 破坏
- **延迟初始化模式**：使用 `std::call_once` 按需获取设备属性，实现线程安全、高效的缓存
- **每设备缓存**：每个设备的独立 `std::once_flag` 实现独立的延迟初始化
- **内联变量（C++17）**：声明为 `inline` 的全局缓存确保在翻译单元间单一定义而不违反 ODR
- **错误处理**：所有 CUDA API 调用都通过 `STD_TORCH_CHECK` 检查，提供描述性错误消息
- **资源重用**：流和 cuBLAS 句柄查询利用 PyTorch 的内部资源管理以实现最佳 GPU 利用率

## Dependencies / 依赖关系

### EN:
- **PyTorch Stable ABI**:
  - `torch/csrc/inductor/aoti_torch/c/shim.h` - AOTI C shim for stream access
  - `torch/csrc/stable/accelerator.h`, `ops.h`, `tensor.h` - stable tensor/op APIs
  - `torch/headeronly/util/shim_utils.h` - header-only utilities
- **CUDA Libraries**:
  - `cuda_runtime.h` - CUDA runtime API (`cudaGetDevice`, `cudaGetDeviceProperties`, `cudaStream_t`)
  - `cublas_v2.h` - cuBLAS library types (`cublasHandle_t`)
- **Standard Library**:
  - `<deque>`, `<vector>` - storage for device flags and properties
  - `<mutex>` - `std::call_once`, `std::once_flag` for thread safety
  - `<string>` - error message construction
- **Callers**: Used by all vLLM CUDA kernels for device queries and stream access:
  - Quantization kernels (`per_token_group_quant.cu`)
  - CUTLASS GEMM wrappers
  - Custom kernel implementations throughout `csrc/`

### CN:
- **PyTorch 稳定 ABI**：
  - `torch/csrc/inductor/aoti_torch/c/shim.h` - 用于流访问的 AOTI C 填充层
  - `torch/csrc/stable/accelerator.h`、`ops.h`、`tensor.h` - 稳定张量/操作 API
  - `torch/headeronly/util/shim_utils.h` - 仅头文件实用工具
- **CUDA 库**：
  - `cuda_runtime.h` - CUDA 运行时 API（`cudaGetDevice`、`cudaGetDeviceProperties`、`cudaStream_t`）
  - `cublas_v2.h` - cuBLAS 库类型（`cublasHandle_t`）
- **标准库**：
  - `<deque>`、`<vector>` - 设备标志和属性的存储
  - `<mutex>` - `std::call_once`、`std::once_flag` 用于线程安全
  - `<string>` - 错误消息构造
- **调用者**：所有 vLLM CUDA 内核用于设备查询和流访问：
  - 量化内核（`per_token_group_quant.cu`）
  - CUTLASS GEMM 包装器
  - 整个 `csrc/` 中的自定义内核实现
