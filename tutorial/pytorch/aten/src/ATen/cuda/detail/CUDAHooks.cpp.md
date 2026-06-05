# CUDAHooks.cpp — Code Analysis / 代码分析
## Source / 来源
- File: `aten/src/ATen/cuda/detail/CUDAHooks.cpp`
- Repository: `pytorch/pytorch`
- Purpose (EN): This file implements CUDA backend support, with primary focus on `_Initializer`, `c10::cuda::_internal`, `at::cuda::detail`.
- 用途（中文）: 该文件实现CUDA 后端支持，核心关注对象是 `_Initializer`, `c10::cuda::_internal`, `at::cuda::detail`。

## Line-by-Line Analysis / 逐行分析

### Lines 1-20
```cpp
#include <ATen/cuda/detail/CUDAHooks.h>

#include <ATen/cuda/CUDAGeneratorImpl.h>
#include <ATen/Context.h>
#include <ATen/DeviceGuard.h>
#include <ATen/DynamicLibrary.h>
#include <ATen/cuda/CUDAConfig.h>
#include <ATen/cuda/CUDADevice.h>
#include <ATen/cuda/Exceptions.h>
#include <ATen/cuda/PeerToPeerAccess.h>
#include <ATen/cuda/PinnedMemoryAllocator.h>
#include <ATen/cuda/nvrtc_stub/ATenNVRTC.h>
#include <ATen/detail/CUDAHooksInterface.h>
#include <ATen/native/cuda/CuFFTPlanCache.h>
#include <c10/util/Exception.h>
#include <c10/util/env.h>
#include <c10/cuda/CUDACachingAllocator.h>
#include <c10/cuda/CUDAFunctions.h>
#include <c10/util/irange.h>

```
- EN: This block sets up header dependencies and the initial compilation context. The file brings required declarations into scope before entering the main ATen logic.
- CN: 该代码块设置头文件依赖与初始编译上下文。文件在进入主要 ATen 逻辑前先引入所需声明。

### Lines 21-38
```cpp
#if AT_CUDNN_ENABLED()
#include <ATen/cudnn/cudnn-wrapper.h>
#include <cudnn_frontend.h>
#endif

#if AT_MAGMA_ENABLED()
#include <magma_v2.h>
#endif

#if defined(USE_ROCM)
#include <miopen/version.h>
#include <hipblaslt/hipblaslt-version.h>
#endif

#ifndef USE_ROCM
#include <ATen/cuda/detail/LazyNVRTC.h>
#endif

```
- EN: This block sets up header dependencies and the initial compilation context. The file brings required declarations into scope before entering the main ATen logic.
- CN: 该代码块设置头文件依赖与初始编译上下文。文件在进入主要 ATen 逻辑前先引入所需声明。

### Lines 39-58
```cpp

#include <sstream>
#include <cstddef>
#include <memory>

namespace c10::cuda::_internal {
void setHasPrimaryContext(bool (*func)(DeviceIndex));
}

namespace at::cuda::detail {

const at::cuda::NVRTC& nvrtc();
DeviceIndex current_device();

static void (*magma_init_fn)() = nullptr;

void set_magma_init_fn(void (*fn)()) {
  magma_init_fn = fn;
}

```
- EN: Focus symbols: `c10::cuda::_internal`, `at::cuda::detail`, `setHasPrimaryContext`, `bool`, `nvrtc`. This block sets up header dependencies and the initial compilation context. The file brings required declarations into scope before entering the main ATen logic.
- CN: 关注符号：`c10::cuda::_internal`, `at::cuda::detail`, `setHasPrimaryContext`, `bool`, `nvrtc`。该代码块设置头文件依赖与初始编译上下文。文件在进入主要 ATen 逻辑前先引入所需声明。

### Lines 59-74
```cpp
namespace {
bool _hasPrimaryContext(DeviceIndex device_index) {
  TORCH_CHECK(device_index >= 0 && device_index < at::cuda::device_count(),
              "hasPrimaryContext expects a valid device index, but got device_index=", static_cast<int>(device_index));
  unsigned int ctx_flags = 0;
  // In standalone tests of cuDevicePrimaryCtxGetState, I've seen the "active" argument end up with weird
  // (garbage-looking nonzero) values when the context is not active, unless I initialize it to zero.
  int ctx_is_active = 0;
  AT_CUDA_DRIVER_CHECK(nvrtc().cuDevicePrimaryCtxGetState(device_index, &ctx_flags, &ctx_is_active));
  return ctx_is_active == 1;
}

// Register hasPrimaryContext back to c10::cuda
struct _Initializer {
  _Initializer() {
      c10::cuda::_internal::setHasPrimaryContext(_hasPrimaryContext);
```
- EN: Focus symbols: `_Initializer`, `_hasPrimaryContext`, `TORCH_CHECK`, `device_count`, `AT_CUDA_DRIVER_CHECK`. This block declares the main types, aliases, or compile-time descriptors used by this file. These declarations define the shape of the API and the data carried through later code.
- CN: 关注符号：`_Initializer`, `_hasPrimaryContext`, `TORCH_CHECK`, `device_count`, `AT_CUDA_DRIVER_CHECK`。该代码块声明此文件使用的核心类型、别名或编译期描述对象。这些声明决定了 API 的形状以及后续代码中流动的数据结构。

### Lines 75-92
```cpp
  }
  ~_Initializer() {
      c10::cuda::_internal::setHasPrimaryContext(nullptr);
  }
} initializer;
} // anonymous namespace


// NB: deleter is dynamic, because we need it to live in a separate
// compilation unit (alt is to have another method in hooks, but
// let's not if we don't need to!)
void CUDAHooks::init() const {
  C10_LOG_API_USAGE_ONCE("aten.init.cuda");

  const auto num_devices = c10::cuda::device_count_ensure_non_zero();
  c10::cuda::CUDACachingAllocator::init(num_devices);
  at::cuda::detail::init_p2p_access_cache(num_devices);

```
- EN: Focus symbols: `~_Initializer`, `setHasPrimaryContext`, `init`, `C10_LOG_API_USAGE_ONCE`, `device_count_ensure_non_zero`. This block declares or implements callable behavior for this module. Inputs are translated into lower-level operations, state updates, or returned values here.
- CN: 关注符号：`~_Initializer`, `setHasPrimaryContext`, `init`, `C10_LOG_API_USAGE_ONCE`, `device_count_ensure_non_zero`。该代码块声明或实现该模块的可调用行为。这里把输入转换为更底层操作、状态更新或返回值。

### Lines 93-110
```cpp
#if AT_MAGMA_ENABLED()
  TORCH_INTERNAL_ASSERT(magma_init_fn != nullptr, "Cannot initialize magma, init routine not set");
  magma_init_fn();
#endif
}

const Generator& CUDAHooks::getDefaultGenerator(DeviceIndex device_index) const {
  return at::cuda::detail::getDefaultCUDAGenerator(device_index);
}

Generator CUDAHooks::getNewGenerator(DeviceIndex device_index) const {
  return make_generator<at::CUDAGeneratorImpl>(device_index);
}

Device CUDAHooks::getDeviceFromPtr(void* data) const {
  return at::cuda::getDeviceFromPtr(data);
}

```
- EN: Focus symbols: `TORCH_INTERNAL_ASSERT`, `magma_init_fn`, `getDefaultGenerator`, `getDefaultCUDAGenerator`, `getNewGenerator`. This block implements callable behavior together with runtime validation. The checks reject unsupported states before delegating work to lower-level helpers.
- CN: 关注符号：`TORCH_INTERNAL_ASSERT`, `magma_init_fn`, `getDefaultGenerator`, `getDefaultCUDAGenerator`, `getNewGenerator`。该代码块实现可调用行为并包含运行时校验。这些检查会在把工作下发给更底层辅助函数前拒绝不支持的状态。

### Lines 111-126
```cpp
bool CUDAHooks::isPinnedPtr(const void* data) const {
  // First check if driver is broken/missing, in which case PyTorch CPU
  // functionalities should still work, we should report `false` here.
  if (!at::cuda::is_available()) {
    return false;
  }
  // cudaPointerGetAttributes grabs context on the current device, so we set
  // device to one that already has context, if exists.
  at::OptionalDeviceGuard device_guard;
  auto primary_ctx_device_index = getDeviceIndexWithPrimaryContext();
  if (primary_ctx_device_index.has_value()) {
    device_guard.reset_device(at::Device(at::DeviceType::CUDA, *primary_ctx_device_index));
  }
  cudaPointerAttributes attr{};
  // We do not believe that CUDA needs mutable access to the data
  // here.
```
- EN: Focus symbols: `isPinnedPtr`, `is_available`, `getDeviceIndexWithPrimaryContext`, `has_value`, `reset_device`. This block declares or implements callable behavior for this module. Inputs are translated into lower-level operations, state updates, or returned values here.
- CN: 关注符号：`isPinnedPtr`, `is_available`, `getDeviceIndexWithPrimaryContext`, `has_value`, `reset_device`。该代码块声明或实现该模块的可调用行为。这里把输入转换为更底层操作、状态更新或返回值。

### Lines 127-143
```cpp
  cudaError_t err = cudaPointerGetAttributes(&attr, data);
#if !defined(USE_ROCM)
  if (err == cudaErrorInvalidValue) {
    (void)cudaGetLastError(); // clear CUDA error
    return false;
  }
  AT_CUDA_CHECK(err);
#else
  // HIP throws hipErrorUnknown here
  if (err != cudaSuccess) {
    (void)cudaGetLastError(); // clear HIP error
    return false;
  }
#endif
  return attr.type == cudaMemoryTypeHost;
}

```
- EN: Focus symbols: `cudaPointerGetAttributes`, `cudaGetLastError`, `AT_CUDA_CHECK`. This block declares or implements callable behavior for this module. Inputs are translated into lower-level operations, state updates, or returned values here.
- CN: 关注符号：`cudaPointerGetAttributes`, `cudaGetLastError`, `AT_CUDA_CHECK`。该代码块声明或实现该模块的可调用行为。这里把输入转换为更底层操作、状态更新或返回值。

### Lines 144-159
```cpp
bool CUDAHooks::hasCUDA() const {
  return at::cuda::is_available();
}

bool CUDAHooks::hasMAGMA() const {
#if AT_MAGMA_ENABLED()
  return true;
#else
  return false;
#endif
}

bool CUDAHooks::hasCuDNN() const {
  return AT_CUDNN_ENABLED();
}

```
- EN: Focus symbols: `hasCUDA`, `is_available`, `hasMAGMA`, `hasCuDNN`, `AT_CUDNN_ENABLED`. This block declares or implements callable behavior for this module. Inputs are translated into lower-level operations, state updates, or returned values here.
- CN: 关注符号：`hasCUDA`, `is_available`, `hasMAGMA`, `hasCuDNN`, `AT_CUDNN_ENABLED`。该代码块声明或实现该模块的可调用行为。这里把输入转换为更底层操作、状态更新或返回值。

### Lines 160-179
```cpp
bool CUDAHooks::hasCuSOLVER() const {
#if defined(CUDART_VERSION) && defined(CUSOLVER_VERSION)
  return true;
#elif AT_ROCM_ENABLED()
  return true;
#else
  return false;
#endif
}

bool CUDAHooks::hasCuBLASLt() const {
#if defined(CUDART_VERSION)
  return true;
#elif AT_ROCM_ENABLED()
  return true;
#else
  return false;
#endif
}

```
- EN: Focus symbols: `hasCuSOLVER`, `hasCuBLASLt`. This block declares or implements callable behavior for this module. Inputs are translated into lower-level operations, state updates, or returned values here.
- CN: 关注符号：`hasCuSOLVER`, `hasCuBLASLt`。该代码块声明或实现该模块的可调用行为。这里把输入转换为更底层操作、状态更新或返回值。

### Lines 180-195
```cpp

bool CUDAHooks::hasCKSDPA() const {
#if !defined(USE_ROCM)
    return false;
#elif defined(USE_ROCM) && defined(USE_ROCM_CK_SDPA)
    return true;
#else
    return false;
#endif
}

bool CUDAHooks::hasCKGEMM() const {
#if !defined(USE_ROCM)
    return false;
#elif defined(USE_ROCM) && defined(USE_ROCM_CK_GEMM)
    return true;
```
- EN: Focus symbols: `hasCKSDPA`, `hasCKGEMM`. This block declares or implements callable behavior for this module. Inputs are translated into lower-level operations, state updates, or returned values here.
- CN: 关注符号：`hasCKSDPA`, `hasCKGEMM`。该代码块声明或实现该模块的可调用行为。这里把输入转换为更底层操作、状态更新或返回值。

### Lines 196-211
```cpp
#else
    return false;
#endif
}

bool CUDAHooks::hasROCM() const {
  // Currently, this is same as `compiledWithMIOpen`.
  // But in future if there are ROCm builds without MIOpen,
  // then `hasROCM` should return true while `compiledWithMIOpen`
  // should return false
  return AT_ROCM_ENABLED();
}

#if defined(USE_DIRECT_NVRTC)
static std::pair<std::unique_ptr<at::DynamicLibrary>, at::cuda::NVRTC*> load_nvrtc() {
  return std::make_pair(nullptr, at::cuda::load_nvrtc());
```
- EN: Focus symbols: `hasROCM`, `AT_ROCM_ENABLED`, `load_nvrtc`, `make_pair`. This block declares or implements callable behavior for this module. Inputs are translated into lower-level operations, state updates, or returned values here.
- CN: 关注符号：`hasROCM`, `AT_ROCM_ENABLED`, `load_nvrtc`, `make_pair`。该代码块声明或实现该模块的可调用行为。这里把输入转换为更底层操作、状态更新或返回值。

### Lines 212-227
```cpp
}
#elif !defined(USE_ROCM)
static std::pair<std::unique_ptr<at::DynamicLibrary>, at::cuda::NVRTC*> load_nvrtc() {
  return std::make_pair(nullptr, &at::cuda::detail::lazyNVRTC);
}
#else
static std::pair<std::unique_ptr<at::DynamicLibrary>, at::cuda::NVRTC*> load_nvrtc() {
#if defined(_WIN32)
  std::string libcaffe2_nvrtc = "caffe2_nvrtc.dll";
#elif defined(__APPLE__)
  std::string libcaffe2_nvrtc = "libcaffe2_nvrtc.dylib";
#else
  std::string libcaffe2_nvrtc = "libcaffe2_nvrtc.so";
#endif
  std::unique_ptr<at::DynamicLibrary> libnvrtc_stub(
      new at::DynamicLibrary(libcaffe2_nvrtc.c_str()));
```
- EN: Focus symbols: `load_nvrtc`, `make_pair`, `libnvrtc_stub`, `DynamicLibrary`, `c_str`. This block declares or implements callable behavior for this module. Inputs are translated into lower-level operations, state updates, or returned values here.
- CN: 关注符号：`load_nvrtc`, `make_pair`, `libnvrtc_stub`, `DynamicLibrary`, `c_str`。该代码块声明或实现该模块的可调用行为。这里把输入转换为更底层操作、状态更新或返回值。

### Lines 228-243
```cpp
  auto fn = (at::cuda::NVRTC * (*)()) libnvrtc_stub->sym("load_nvrtc");
  return std::make_pair(std::move(libnvrtc_stub), fn());
}
#endif

const at::cuda::NVRTC& nvrtc() {
  // must hold onto DynamicLibrary otherwise it will unload
  static auto handle = load_nvrtc();
  return *handle.second;
}

const at::cuda::NVRTC& CUDAHooks::nvrtc() const {
  return at::cuda::detail::nvrtc();
}

DeviceIndex current_device() {
```
- EN: Focus symbols: `sym`, `make_pair`, `move`, `fn`, `nvrtc`. This block declares or implements callable behavior for this module. Inputs are translated into lower-level operations, state updates, or returned values here.
- CN: 关注符号：`sym`, `make_pair`, `move`, `fn`, `nvrtc`。该代码块声明或实现该模块的可调用行为。这里把输入转换为更底层操作、状态更新或返回值。

### Lines 244-262
```cpp
  c10::DeviceIndex device = 0;
  cudaError_t err = c10::cuda::GetDevice(&device);
  if (err == cudaSuccess) {
    return device;
  }
  return -1;
}

/**
 * DEPRECATED: use getCurrentDevice() instead
 */
DeviceIndex CUDAHooks::current_device() const {
  return at::cuda::detail::current_device();
}

bool CUDAHooks::hasPrimaryContext(DeviceIndex device_index) const {
  return _hasPrimaryContext(device_index);
}

```
- EN: Focus symbols: `GetDevice`, `getCurrentDevice`, `current_device`, `hasPrimaryContext`, `_hasPrimaryContext`. This block declares or implements callable behavior for this module. Inputs are translated into lower-level operations, state updates, or returned values here.
- CN: 关注符号：`GetDevice`, `getCurrentDevice`, `current_device`, `hasPrimaryContext`, `_hasPrimaryContext`。该代码块声明或实现该模块的可调用行为。这里把输入转换为更底层操作、状态更新或返回值。

### Lines 263-278
```cpp
Allocator* CUDAHooks::getPinnedMemoryAllocator() const {
  return at::cuda::getPinnedMemoryAllocator();
}

Allocator* CUDAHooks::getCUDADeviceAllocator() const {
  return at::cuda::getCUDADeviceAllocator();
}

bool CUDAHooks::compiledWithCuDNN() const {
  return AT_CUDNN_ENABLED();
}

bool CUDAHooks::compiledWithMIOpen() const {
  return AT_ROCM_ENABLED();
}

```
- EN: Focus symbols: `getPinnedMemoryAllocator`, `getCUDADeviceAllocator`, `compiledWithCuDNN`, `AT_CUDNN_ENABLED`, `compiledWithMIOpen`. This block declares or implements callable behavior for this module. Inputs are translated into lower-level operations, state updates, or returned values here.
- CN: 关注符号：`getPinnedMemoryAllocator`, `getCUDADeviceAllocator`, `compiledWithCuDNN`, `AT_CUDNN_ENABLED`, `compiledWithMIOpen`。该代码块声明或实现该模块的可调用行为。这里把输入转换为更底层操作、状态更新或返回值。

### Lines 279-294
```cpp
bool CUDAHooks::supportsDilatedConvolutionWithCuDNN() const {
#if AT_CUDNN_ENABLED()
  if (!hasCUDA()) {
    return false;
  }
  // NOTE: extra parenthesis around numbers disable clang warnings about
  // dead code
  return true;
#else
  return false;
#endif
}

bool CUDAHooks::supportsDepthwiseConvolutionWithCuDNN() const {
#if AT_CUDNN_ENABLED()
  if (!hasCUDA()) {
```
- EN: Focus symbols: `supportsDilatedConvolutionWithCuDNN`, `hasCUDA`, `supportsDepthwiseConvolutionWithCuDNN`. This block declares or implements callable behavior for this module. Inputs are translated into lower-level operations, state updates, or returned values here.
- CN: 关注符号：`supportsDilatedConvolutionWithCuDNN`, `hasCUDA`, `supportsDepthwiseConvolutionWithCuDNN`。该代码块声明或实现该模块的可调用行为。这里把输入转换为更底层操作、状态更新或返回值。

### Lines 295-310
```cpp
    return false;
  }
  cudaDeviceProp* prop = at::cuda::getCurrentDeviceProperties();
  // Check for Volta cores
  if (prop->major >= 7) {
    return true;
  } else {
    return false;
  }
#else
  return false;
#endif
}

bool CUDAHooks::supportsBFloat16ConvolutionWithCuDNNv8() const {
#if AT_CUDNN_ENABLED()
```
- EN: Focus symbols: `getCurrentDeviceProperties`, `supportsBFloat16ConvolutionWithCuDNNv8`. This block declares or implements callable behavior for this module. Inputs are translated into lower-level operations, state updates, or returned values here.
- CN: 关注符号：`getCurrentDeviceProperties`, `supportsBFloat16ConvolutionWithCuDNNv8`。该代码块声明或实现该模块的可调用行为。这里把输入转换为更底层操作、状态更新或返回值。

### Lines 311-326
```cpp
  if (!hasCUDA()) {
    return false;
  }
  cudaDeviceProp* prop = at::cuda::getCurrentDeviceProperties();
  // Check for Volta cores
  if (prop->major >= 8) {
    return true;
  } else {
    return false;
  }
#else
  return false;
#endif
}

bool CUDAHooks::supportsBFloat16RNNWithCuDNN() const {
```
- EN: Focus symbols: `hasCUDA`, `getCurrentDeviceProperties`, `supportsBFloat16RNNWithCuDNN`. This block declares or implements callable behavior for this module. Inputs are translated into lower-level operations, state updates, or returned values here.
- CN: 关注符号：`hasCUDA`, `getCurrentDeviceProperties`, `supportsBFloat16RNNWithCuDNN`。该代码块声明或实现该模块的可调用行为。这里把输入转换为更底层操作、状态更新或返回值。

### Lines 327-342
```cpp
#if AT_CUDNN_ENABLED() && (CUDNN_VERSION >= 91300)
  if (!hasCUDA()) {
    return false;
  }
  cudaDeviceProp* prop = at::cuda::getCurrentDeviceProperties();
  // Check for Volta cores
  if (prop->major >= 8) {
    return true;
  } else {
    return false;
  }
#else
  return false;
#endif
}

```
- EN: Focus symbols: `hasCUDA`, `getCurrentDeviceProperties`. This block declares or implements callable behavior for this module. Inputs are translated into lower-level operations, state updates, or returned values here.
- CN: 关注符号：`hasCUDA`, `getCurrentDeviceProperties`。该代码块声明或实现该模块的可调用行为。这里把输入转换为更底层操作、状态更新或返回值。

### Lines 343-362
```cpp
long CUDAHooks::versionCuDNN() const {
#if AT_CUDNN_ENABLED()
  return CUDNN_VERSION;
#else
  TORCH_CHECK(false, "Cannot query CuDNN version if ATen_cuda is not built with CuDNN");
#endif
}

long CUDAHooks::versionRuntimeCuDNN() const {
#if AT_CUDNN_ENABLED()
#ifndef USE_STATIC_CUDNN
  return cudnnGetVersion();
#else
  return CUDNN_VERSION;
#endif
#else
  TORCH_CHECK(false, "Cannot query CuDNN version if ATen_cuda is not built with CuDNN");
#endif
}

```
- EN: Focus symbols: `versionCuDNN`, `TORCH_CHECK`, `versionRuntimeCuDNN`, `cudnnGetVersion`. This block implements callable behavior together with runtime validation. The checks reject unsupported states before delegating work to lower-level helpers.
- CN: 关注符号：`versionCuDNN`, `TORCH_CHECK`, `versionRuntimeCuDNN`, `cudnnGetVersion`。该代码块实现可调用行为并包含运行时校验。这些检查会在把工作下发给更底层辅助函数前拒绝不支持的状态。

### Lines 363-380
```cpp
long CUDAHooks::versionCuDNNFrontend() const {
#if AT_CUDNN_ENABLED()
  return CUDNN_FRONTEND_VERSION;
#else
  TORCH_CHECK(false, "Cannot query CuDNN Frontend version if ATen_cuda is not built with CuDNN");
#endif
}

long CUDAHooks::versionMIOpen() const {
#if AT_ROCM_ENABLED()
  return MIOPEN_VERSION_MAJOR * 10000 +
         MIOPEN_VERSION_MINOR * 100 +
         MIOPEN_VERSION_PATCH;
#else
  TORCH_CHECK(false, "Cannot query MIOpen version if ATen_cuda is not built with ROCm");
#endif
}

```
- EN: Focus symbols: `versionCuDNNFrontend`, `TORCH_CHECK`, `versionMIOpen`. This block implements callable behavior together with runtime validation. The checks reject unsupported states before delegating work to lower-level helpers.
- CN: 关注符号：`versionCuDNNFrontend`, `TORCH_CHECK`, `versionMIOpen`。该代码块实现可调用行为并包含运行时校验。这些检查会在把工作下发给更底层辅助函数前拒绝不支持的状态。

### Lines 381-400
```cpp
long CUDAHooks::versionHipBLASLt() const {
#if AT_ROCM_ENABLED()
  return HIPBLASLT_VERSION_MAJOR * 10000 +
         HIPBLASLT_VERSION_MINOR * 100 +
         HIPBLASLT_VERSION_PATCH;
#else
  TORCH_CHECK(false, "Cannot query HipBLASLt version if ATen_cuda is not built with ROCm");
#endif
}

long CUDAHooks::versionCUDART() const {
#ifdef CUDART_VERSION
  return CUDART_VERSION;
#else
  TORCH_CHECK(
    false,
    "Cannot query CUDART version because CUDART is not available");
#endif
}

```
- EN: Focus symbols: `versionHipBLASLt`, `TORCH_CHECK`, `versionCUDART`. This block implements callable behavior together with runtime validation. The checks reject unsupported states before delegating work to lower-level helpers.
- CN: 关注符号：`versionHipBLASLt`, `TORCH_CHECK`, `versionCUDART`。该代码块实现可调用行为并包含运行时校验。这些检查会在把工作下发给更底层辅助函数前拒绝不支持的状态。

### Lines 401-416
```cpp
bool CUDAHooks::hasCUDART() const {
#ifdef CUDART_VERSION
  return true;
#else
  return false;
#endif
}

std::string CUDAHooks::showConfig() const {
  std::ostringstream oss;

  int runtimeVersion = 0;
  AT_CUDA_CHECK(cudaRuntimeGetVersion(&runtimeVersion));

  auto printCudaStyleVersion = [&](size_t v) {
#ifdef USE_ROCM
```
- EN: Focus symbols: `hasCUDART`, `showConfig`, `AT_CUDA_CHECK`, `cudaRuntimeGetVersion`. This block declares or implements callable behavior for this module. Inputs are translated into lower-level operations, state updates, or returned values here.
- CN: 关注符号：`hasCUDART`, `showConfig`, `AT_CUDA_CHECK`, `cudaRuntimeGetVersion`。该代码块声明或实现该模块的可调用行为。这里把输入转换为更底层操作、状态更新或返回值。

### Lines 417-433
```cpp
    // HIP_VERSION value format was changed after ROCm v4.2 to include the patch number
    if(v < 500) {
      // If major=xx, minor=yy then format -> xxyy
      oss << (v / 100) << '.' << (v % 10);
    }
    else {
      // If major=xx, minor=yy & patch=zzzzz then format -> xxyyzzzzz
      oss << (v / 10000000) << '.' << (v / 100000 % 100) << '.' << (v % 100000);
    }
#else
    oss << (v / 1000) << '.' << (v / 10 % 100);
    if (v % 10 != 0) {
      oss << '.' << (v % 10);
    }
#endif
  };

```
- EN: This block declares or implements callable behavior for this module. Inputs are translated into lower-level operations, state updates, or returned values here.
- CN: 该代码块声明或实现该模块的可调用行为。这里把输入转换为更底层操作、状态更新或返回值。

### Lines 434-451
```cpp
#if !defined(USE_ROCM)
  oss << "  - CUDA Runtime ";
#else
  oss << "  - HIP Runtime ";
#endif
  printCudaStyleVersion(runtimeVersion);
  oss << '\n';

  // TODO: Make HIPIFY understand CUDART_VERSION macro
#if !defined(USE_ROCM)
  if (runtimeVersion != CUDART_VERSION) {
    oss << "  - Built with CUDA Runtime ";
    printCudaStyleVersion(CUDART_VERSION);
    oss << '\n';
  }
  oss << "  - NVCC architecture flags: " << NVCC_FLAGS_EXTRA << '\n';
#endif

```
- EN: Focus symbols: `printCudaStyleVersion`. This block declares or implements callable behavior for this module. Inputs are translated into lower-level operations, state updates, or returned values here.
- CN: 关注符号：`printCudaStyleVersion`。该代码块声明或实现该模块的可调用行为。这里把输入转换为更底层操作、状态更新或返回值。

### Lines 452-467
```cpp
#if !defined(USE_ROCM)
#if AT_CUDNN_ENABLED()


  auto printCudnnStyleVersion = [&](size_t v) {
    oss << (v / 1000) << '.' << (v / 100 % 10);
    if (v % 100 != 0) {
      oss << '.' << (v % 100);
    }
  };

  size_t cudnnVersion = cudnnGetVersion();
  oss << "  - CuDNN ";
  printCudnnStyleVersion(cudnnVersion);
  size_t cudnnCudartVersion = cudnnGetCudartVersion();
  if (cudnnCudartVersion != CUDART_VERSION) {
```
- EN: Focus symbols: `cudnnGetVersion`, `printCudnnStyleVersion`, `cudnnGetCudartVersion`. This block declares or implements callable behavior for this module. Inputs are translated into lower-level operations, state updates, or returned values here.
- CN: 关注符号：`cudnnGetVersion`, `printCudnnStyleVersion`, `cudnnGetCudartVersion`。该代码块声明或实现该模块的可调用行为。这里把输入转换为更底层操作、状态更新或返回值。

### Lines 468-487
```cpp
    oss << "  (built against CUDA ";
    printCudaStyleVersion(cudnnCudartVersion);
    oss << ')';
  }
  oss << '\n';
  if (cudnnVersion != CUDNN_VERSION) {
    oss << "    - Built with CuDNN ";
    printCudnnStyleVersion(CUDNN_VERSION);
    oss << '\n';
  }
#endif
#else
  // TODO: Check if miopen has the functions above and unify
  oss << "  - MIOpen " << MIOPEN_VERSION_MAJOR << '.' << MIOPEN_VERSION_MINOR << '.' << MIOPEN_VERSION_PATCH << '\n';
#endif

#if AT_MAGMA_ENABLED()
  oss << "  - Magma " << MAGMA_VERSION_MAJOR << '.' << MAGMA_VERSION_MINOR << '.' << MAGMA_VERSION_MICRO << '\n';
#endif

```
- EN: Focus symbols: `printCudaStyleVersion`, `printCudnnStyleVersion`. This block declares or implements callable behavior for this module. Inputs are translated into lower-level operations, state updates, or returned values here.
- CN: 关注符号：`printCudaStyleVersion`, `printCudnnStyleVersion`。该代码块声明或实现该模块的可调用行为。这里把输入转换为更底层操作、状态更新或返回值。

### Lines 488-507
```cpp
  return oss.str();
}

double CUDAHooks::batchnormMinEpsilonCuDNN() const {
#if AT_CUDNN_ENABLED()
  return CUDNN_BN_MIN_EPSILON;
#else
  TORCH_CHECK(false,
      "Cannot query CUDNN_BN_MIN_EPSILON if ATen_cuda is not built with CuDNN");
#endif
}

int64_t CUDAHooks::cuFFTGetPlanCacheMaxSize(DeviceIndex device_index) const {
  return at::native::detail::cufft_get_plan_cache_max_size_impl(device_index);
}

void CUDAHooks::cuFFTSetPlanCacheMaxSize(DeviceIndex device_index, int64_t max_size) const {
  at::native::detail::cufft_set_plan_cache_max_size_impl(device_index, max_size);
}

```
- EN: Focus symbols: `str`, `batchnormMinEpsilonCuDNN`, `TORCH_CHECK`, `cuFFTGetPlanCacheMaxSize`, `cufft_get_plan_cache_max_size_impl`. This block implements callable behavior together with runtime validation. The checks reject unsupported states before delegating work to lower-level helpers.
- CN: 关注符号：`str`, `batchnormMinEpsilonCuDNN`, `TORCH_CHECK`, `cuFFTGetPlanCacheMaxSize`, `cufft_get_plan_cache_max_size_impl`。该代码块实现可调用行为并包含运行时校验。这些检查会在把工作下发给更底层辅助函数前拒绝不支持的状态。

### Lines 508-526
```cpp
int64_t CUDAHooks::cuFFTGetPlanCacheSize(DeviceIndex device_index) const {
  return at::native::detail::cufft_get_plan_cache_size_impl(device_index);
}

void CUDAHooks::cuFFTClearPlanCache(DeviceIndex device_index) const {
  at::native::detail::cufft_clear_plan_cache_impl(device_index);
}

/**
 * DEPRECATED: use deviceCount() instead
 */
int CUDAHooks::getNumGPUs() const {
  return at::cuda::device_count();
}

DeviceIndex CUDAHooks::deviceCount() const {
  return at::cuda::device_count();
}

```
- EN: Focus symbols: `cuFFTGetPlanCacheSize`, `cufft_get_plan_cache_size_impl`, `cuFFTClearPlanCache`, `cufft_clear_plan_cache_impl`, `deviceCount`. This block declares or implements callable behavior for this module. Inputs are translated into lower-level operations, state updates, or returned values here.
- CN: 关注符号：`cuFFTGetPlanCacheSize`, `cufft_get_plan_cache_size_impl`, `cuFFTClearPlanCache`, `cufft_clear_plan_cache_impl`, `deviceCount`。该代码块声明或实现该模块的可调用行为。这里把输入转换为更底层操作、状态更新或返回值。

### Lines 527-542
```cpp
DeviceIndex CUDAHooks::getCurrentDevice() const {
  return at::cuda::detail::current_device();
}

#ifdef USE_ROCM
bool CUDAHooks::isGPUArch(const std::vector<std::string>& archs, DeviceIndex device_index) const {
  hipDeviceProp_t* prop;
  if (device_index == -1){
      prop = at::cuda::getCurrentDeviceProperties();
  } else {
      prop = at::cuda::getDeviceProperties(device_index);
  }

  std::string device_arch = prop->gcnArchName;
  for (std::string arch : archs) {
      size_t substring = device_arch.find(arch);
```
- EN: Focus symbols: `getCurrentDevice`, `current_device`, `isGPUArch`, `getCurrentDeviceProperties`, `getDeviceProperties`. This block declares or implements callable behavior for this module. Inputs are translated into lower-level operations, state updates, or returned values here.
- CN: 关注符号：`getCurrentDevice`, `current_device`, `isGPUArch`, `getCurrentDeviceProperties`, `getDeviceProperties`。该代码块声明或实现该模块的可调用行为。这里把输入转换为更底层操作、状态更新或返回值。

### Lines 543-562
```cpp
      if (substring != std::string::npos) {
          return true;
      }
  }
  return false;
}

const std::vector<std::string>& CUDAHooks::getHipblasltPreferredArchs() const {
  static const std::vector<std::string> archs = {
    "gfx90a", "gfx942",
#if ROCM_VERSION >= 60400
    "gfx1200", "gfx1201",
#endif
#if ROCM_VERSION >= 70000
    "gfx950"
#endif
  };
  return archs;
}

```
- EN: Focus symbols: `getHipblasltPreferredArchs`. This block declares or implements callable behavior for this module. Inputs are translated into lower-level operations, state updates, or returned values here.
- CN: 关注符号：`getHipblasltPreferredArchs`。该代码块声明或实现该模块的可调用行为。这里把输入转换为更底层操作、状态更新或返回值。

### Lines 563-579
```cpp
const std::vector<std::string>& CUDAHooks::getHipblasltSupportedArchs() const {
  static const std::vector<std::string> archs = {
    "gfx90a", "gfx942",
#if ROCM_VERSION >= 60300
    "gfx1100", "gfx1101", "gfx1103", "gfx1200", "gfx1201", "gfx908",
#endif
#if ROCM_VERSION >= 70000
    "gfx950", "gfx1150", "gfx1151",
#endif
#if ROCM_VERSION >= 70200
    "gfx1250"
#endif
  };
  return archs;
}
#endif

```
- EN: Focus symbols: `getHipblasltSupportedArchs`. This block declares or implements callable behavior for this module. Inputs are translated into lower-level operations, state updates, or returned values here.
- CN: 关注符号：`getHipblasltSupportedArchs`。该代码块声明或实现该模块的可调用行为。这里把输入转换为更底层操作、状态更新或返回值。

### Lines 580-591
```cpp
void CUDAHooks::deviceSynchronize(DeviceIndex device_index) const {
  at::DeviceGuard device_guard(at::Device(at::DeviceType::CUDA, device_index));
  c10::cuda::device_synchronize();
}

// Sigh, the registry doesn't support namespaces :(
using at::CUDAHooksRegistry;
using at::RegistererCUDAHooksRegistry;

REGISTER_CUDA_HOOKS(CUDAHooks)

} // namespace at::cuda::detail
```
- EN: Focus symbols: `at::cuda::detail`, `deviceSynchronize`, `device_guard`, `Device`, `device_synchronize`. This block declares or implements callable behavior for this module. Inputs are translated into lower-level operations, state updates, or returned values here.
- CN: 关注符号：`at::cuda::detail`, `deviceSynchronize`, `device_guard`, `Device`, `device_synchronize`。该代码块声明或实现该模块的可调用行为。这里把输入转换为更底层操作、状态更新或返回值。

## Key Concepts / 关键概念
- CUDA backend support / CUDA 后端支持
- Conditional compilation / 条件编译
- Type-system design / 类型系统设计
- Runtime validation / 运行时校验
- Backend/device dispatch / 后端/设备分发
- ATen/C10 internal layering / ATen/C10 内部分层

## Dependencies / 依赖关系
- Direct internal includes / 直接内部依赖: `ATen/cuda/detail/CUDAHooks.h`, `ATen/cuda/CUDAGeneratorImpl.h`, `ATen/Context.h`, `ATen/DeviceGuard.h`, `ATen/DynamicLibrary.h`, `ATen/cuda/CUDAConfig.h`, `ATen/cuda/CUDADevice.h`, `ATen/cuda/Exceptions.h`, `ATen/cuda/PeerToPeerAccess.h`, `ATen/cuda/PinnedMemoryAllocator.h`, `ATen/cuda/nvrtc_stub/ATenNVRTC.h`, `ATen/detail/CUDAHooksInterface.h`
- External/system includes / 外部或系统头: `cudnn_frontend.h`, `magma_v2.h`, `miopen/version.h`, `hipblaslt/hipblaslt-version.h`, `sstream`, `cstddef`, `memory`
- Companion translation units / 配套实现或声明文件: `aten/src/ATen/cuda/detail/CUDAHooks.h`
- Inferred semantic dependencies / 推断出的语义依赖: device dispatch / 设备分发; runtime validation / 运行时校验; namespace scoping / 命名空间作用域
