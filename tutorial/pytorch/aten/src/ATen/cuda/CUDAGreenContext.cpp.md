# CUDAGreenContext.cpp — Code Analysis / 代码分析
## Source / 来源
- File: `aten/src/ATen/cuda/CUDAGreenContext.cpp`
- Repository: `pytorch/pytorch`
- Purpose (EN): This file implements CUDA backend support, with primary focus on `is`, `HAS_CUDA_GREEN_CONTEXT`, `HAS_CUDA_WORKQUEUE_SUPPORT`.
- 用途（中文）: 该文件实现CUDA 后端支持，核心关注对象是 `is`, `HAS_CUDA_GREEN_CONTEXT`, `HAS_CUDA_WORKQUEUE_SUPPORT`。

## Line-by-Line Analysis / 逐行分析

### Lines 1-13
```cpp
#include <ATen/cuda/CUDAGreenContext.h>

#if defined(CUDA_VERSION) && CUDA_VERSION >= 12080 && !defined(USE_ROCM) && defined(PYTORCH_C10_DRIVER_API_SUPPORTED)
#include <c10/cuda/driver_api.h>
#include <stdexcept>
#include <vector>
#define HAS_CUDA_GREEN_CONTEXT() 1
#else
#define HAS_CUDA_GREEN_CONTEXT() 0
// Suppress unused private field warnings as this class is not supposed to be called
C10_DIAGNOSTIC_PUSH_AND_IGNORED_IF_DEFINED("-Wunused-private-field")
#endif

```
- EN: Focus symbols: `is`, `HAS_CUDA_GREEN_CONTEXT`, `C10_DIAGNOSTIC_PUSH_AND_IGNORED_IF_DEFINED`. This block sets up header dependencies and the initial compilation context. The file brings required declarations into scope before entering the main ATen logic.
- CN: 关注符号：`is`, `HAS_CUDA_GREEN_CONTEXT`, `C10_DIAGNOSTIC_PUSH_AND_IGNORED_IF_DEFINED`。该代码块设置头文件依赖与初始编译上下文。文件在进入主要 ATen 逻辑前先引入所需声明。

### Lines 14-23
```cpp
#if defined(CUDA_VERSION) && CUDA_VERSION >= 13010 && HAS_CUDA_GREEN_CONTEXT()
#define HAS_CUDA_WORKQUEUE_SUPPORT() 1
#else
#define HAS_CUDA_WORKQUEUE_SUPPORT() 0
#endif

namespace at::cuda {

GreenContext::GreenContext(
    uint32_t device_id,
```
- EN: Focus symbols: `HAS_CUDA_WORKQUEUE_SUPPORT`, `at::cuda`, `GreenContext`. This block executes runtime branching, validation, or state updates. This is where the file reacts to dynamic conditions instead of only describing static structure.
- CN: 关注符号：`HAS_CUDA_WORKQUEUE_SUPPORT`, `at::cuda`, `GreenContext`。该代码块执行运行时分支、校验或状态更新。这里体现了文件对动态条件的响应，而不只是静态结构描述。

### Lines 24-34
```cpp
    std::optional<uint32_t> num_sms,
    std::optional<int32_t> workqueue_scope,
    std::optional<uint32_t> workqueue_concurrency_limit) {
#if HAS_CUDA_GREEN_CONTEXT()
  TORCH_CHECK(
      num_sms.has_value() || workqueue_scope.has_value(),
      "At least one of num_sms or workqueue_scope must be specified");
  TORCH_CHECK(
      !workqueue_concurrency_limit.has_value() || workqueue_scope.has_value(),
      "workqueue_concurrency_limit requires workqueue_scope to be set");

```
- EN: Focus symbols: `TORCH_CHECK`, `has_value`. This block implements callable behavior together with runtime validation. The checks reject unsupported states before delegating work to lower-level helpers.
- CN: 关注符号：`TORCH_CHECK`, `has_value`。该代码块实现可调用行为并包含运行时校验。这些检查会在把工作下发给更底层辅助函数前拒绝不支持的状态。

### Lines 35-45
```cpp
  int driver_version;
  C10_CUDA_CHECK(cudaDriverGetVersion(&driver_version));
  TORCH_CHECK(
      driver_version >= 12080, "cuda driver too old to use green context!");
  CUcontext pctx = nullptr;
  C10_CUDA_DRIVER_CHECK(c10::cuda::DriverAPI::get()->cuCtxGetCurrent_(&pctx));
  if (C10_UNLIKELY(!pctx)) {
    TORCH_WARN(
        "Attempted to create a green context but"
        " there was no primary context! Creating a primary context...");

```
- EN: Focus symbols: `C10_CUDA_CHECK`, `cudaDriverGetVersion`, `TORCH_CHECK`, `C10_CUDA_DRIVER_CHECK`, `get`. This block implements callable behavior together with runtime validation. The checks reject unsupported states before delegating work to lower-level helpers.
- CN: 关注符号：`C10_CUDA_CHECK`, `cudaDriverGetVersion`, `TORCH_CHECK`, `C10_CUDA_DRIVER_CHECK`, `get`。该代码块实现可调用行为并包含运行时校验。这些检查会在把工作下发给更底层辅助函数前拒绝不支持的状态。

### Lines 46-55
```cpp
    cudaFree(nullptr);
  }

  CUdevice device;
  device_id_ = device_id;
  C10_CUDA_DRIVER_CHECK(
      c10::cuda::DriverAPI::get()->cuDeviceGet_(&device, device_id));

  std::vector<CUdevResource> resources;

```
- EN: Focus symbols: `cudaFree`, `C10_CUDA_DRIVER_CHECK`, `get`, `cuDeviceGet_`. This block declares or implements callable behavior for this module. Inputs are translated into lower-level operations, state updates, or returned values here.
- CN: 关注符号：`cudaFree`, `C10_CUDA_DRIVER_CHECK`, `get`, `cuDeviceGet_`。该代码块声明或实现该模块的可调用行为。这里把输入转换为更底层操作、状态更新或返回值。

### Lines 56-69
```cpp
  // --- SM resource ---
  if (num_sms.has_value()) {
    CUdevResource sm_resource;
    C10_CUDA_DRIVER_CHECK(c10::cuda::DriverAPI::get()->cuDeviceGetDevResource_(
        device, &sm_resource, CU_DEV_RESOURCE_TYPE_SM));

    TORCH_CHECK(
        *num_sms > 0 && *num_sms <= sm_resource.sm.smCount,
        "Invalid number of SMs requested for green context: ",
        *num_sms,
        " (device has ",
        sm_resource.sm.smCount,
        " SMs)");

```
- EN: Focus symbols: `has_value`, `C10_CUDA_DRIVER_CHECK`, `get`, `cuDeviceGetDevResource_`, `TORCH_CHECK`. This block implements callable behavior together with runtime validation. The checks reject unsupported states before delegating work to lower-level helpers.
- CN: 关注符号：`has_value`, `C10_CUDA_DRIVER_CHECK`, `get`, `cuDeviceGetDevResource_`, `TORCH_CHECK`。该代码块实现可调用行为并包含运行时校验。这些检查会在把工作下发给更底层辅助函数前拒绝不支持的状态。

### Lines 70-79
```cpp
    // Split resources
    std::vector<CUdevResource> split_result(1);
    unsigned int nb_groups = 1;
    CUdevResource remaining;

    C10_CUDA_DRIVER_CHECK(
        c10::cuda::DriverAPI::get()->cuDevSmResourceSplitByCount_(
            split_result.data(),
            &nb_groups,
            &sm_resource,
```
- EN: Focus symbols: `split_result`, `C10_CUDA_DRIVER_CHECK`, `get`, `cuDevSmResourceSplitByCount_`, `data`. This block declares or implements callable behavior for this module. Inputs are translated into lower-level operations, state updates, or returned values here.
- CN: 关注符号：`split_result`, `C10_CUDA_DRIVER_CHECK`, `get`, `cuDevSmResourceSplitByCount_`, `data`。该代码块声明或实现该模块的可调用行为。这里把输入转换为更底层操作、状态更新或返回值。

### Lines 80-89
```cpp
            &remaining,
            0, // default flags
            *num_sms));
    TORCH_CHECK(nb_groups == 1, "Failed to create single SM resource group");
    resources.push_back(split_result[0]);
  }

  // --- Workqueue config resource ---
  if (workqueue_scope.has_value()) {
#if HAS_CUDA_WORKQUEUE_SUPPORT()
```
- EN: Focus symbols: `TORCH_CHECK`, `push_back`, `has_value`. This block implements callable behavior together with runtime validation. The checks reject unsupported states before delegating work to lower-level helpers.
- CN: 关注符号：`TORCH_CHECK`, `push_back`, `has_value`。该代码块实现可调用行为并包含运行时校验。这些检查会在把工作下发给更底层辅助函数前拒绝不支持的状态。

### Lines 90-99
```cpp
    TORCH_CHECK(
        driver_version >= 13010, "cuda driver too old to use workqueue configuration!");
    CUdevResource wq_resource{};
    C10_CUDA_DRIVER_CHECK(c10::cuda::DriverAPI::get()->cuDeviceGetDevResource_(
        device, &wq_resource, CU_DEV_RESOURCE_TYPE_WORKQUEUE_CONFIG));

    wq_resource.wqConfig.sharingScope =
        static_cast<CUdevWorkqueueConfigScope>(*workqueue_scope);
    if (workqueue_concurrency_limit.has_value()) {
      wq_resource.wqConfig.wqConcurrencyLimit = *workqueue_concurrency_limit;
```
- EN: Focus symbols: `TORCH_CHECK`, `C10_CUDA_DRIVER_CHECK`, `get`, `cuDeviceGetDevResource_`, `has_value`. This block implements callable behavior together with runtime validation. The checks reject unsupported states before delegating work to lower-level helpers.
- CN: 关注符号：`TORCH_CHECK`, `C10_CUDA_DRIVER_CHECK`, `get`, `cuDeviceGetDevResource_`, `has_value`。该代码块实现可调用行为并包含运行时校验。这些检查会在把工作下发给更底层辅助函数前拒绝不支持的状态。

### Lines 100-109
```cpp
    }
    resources.push_back(wq_resource);
#else
    TORCH_CHECK(
        false,
        "Workqueue configuration for green contexts requires CUDA 13.1+!");
#endif
  }

  // Generate resource descriptor
```
- EN: Focus symbols: `push_back`, `TORCH_CHECK`. This block implements callable behavior together with runtime validation. The checks reject unsupported states before delegating work to lower-level helpers.
- CN: 关注符号：`push_back`, `TORCH_CHECK`。该代码块实现可调用行为并包含运行时校验。这些检查会在把工作下发给更底层辅助函数前拒绝不支持的状态。

### Lines 110-122
```cpp
  CUdevResourceDesc desc;
  C10_CUDA_DRIVER_CHECK(
      c10::cuda::DriverAPI::get()->cuDevResourceGenerateDesc_(
          &desc,
          resources.data(),
          static_cast<unsigned int>(resources.size())));

  // Create green context
  // CU_GREEN_CTX_DEFAULT_STREAM is required per docs:
  // https://docs.nvidia.com/cuda/cuda-driver-api/group__CUDA__GREEN__CONTEXTS.html
  C10_CUDA_DRIVER_CHECK(c10::cuda::DriverAPI::get()->cuGreenCtxCreate_(
      &green_ctx_, desc, device, CU_GREEN_CTX_DEFAULT_STREAM));

```
- EN: Focus symbols: `C10_CUDA_DRIVER_CHECK`, `get`, `cuDevResourceGenerateDesc_`, `data`, `size`. This block declares or implements callable behavior for this module. Inputs are translated into lower-level operations, state updates, or returned values here.
- CN: 关注符号：`C10_CUDA_DRIVER_CHECK`, `get`, `cuDevResourceGenerateDesc_`, `data`, `size`。该代码块声明或实现该模块的可调用行为。这里把输入转换为更底层操作、状态更新或返回值。

### Lines 123-132
```cpp
  // Convert to regular context
  C10_CUDA_DRIVER_CHECK(
      c10::cuda::DriverAPI::get()->cuCtxFromGreenCtx_(&context_, green_ctx_));
  TORCH_CHECK(context_, "Green ctx conversion to regular ctx failed!");
#else
  TORCH_CHECK(false, "Green Context is only supported on CUDA 12.8+!");
#endif
}

std::unique_ptr<GreenContext> GreenContext::create(
```
- EN: Focus symbols: `C10_CUDA_DRIVER_CHECK`, `get`, `cuCtxFromGreenCtx_`, `TORCH_CHECK`, `create`. This block implements callable behavior together with runtime validation. The checks reject unsupported states before delegating work to lower-level helpers.
- CN: 关注符号：`C10_CUDA_DRIVER_CHECK`, `get`, `cuCtxFromGreenCtx_`, `TORCH_CHECK`, `create`。该代码块实现可调用行为并包含运行时校验。这些检查会在把工作下发给更底层辅助函数前拒绝不支持的状态。

### Lines 133-142
```cpp
    std::optional<uint32_t> device_id,
    std::optional<uint32_t> num_sms,
    std::optional<int32_t> workqueue_scope,
    std::optional<uint32_t> workqueue_concurrency_limit) {
#if HAS_CUDA_GREEN_CONTEXT()
  if (!device_id.has_value()) {
    device_id = at::cuda::current_device();
  }
  return std::unique_ptr<GreenContext>(new GreenContext(
      device_id.value(), num_sms, workqueue_scope, workqueue_concurrency_limit));
```
- EN: Focus symbols: `has_value`, `current_device`, `GreenContext`, `value`. This block declares or implements callable behavior for this module. Inputs are translated into lower-level operations, state updates, or returned values here.
- CN: 关注符号：`has_value`, `current_device`, `GreenContext`, `value`。该代码块声明或实现该模块的可调用行为。这里把输入转换为更底层操作、状态更新或返回值。

### Lines 143-152
```cpp
#else
  TORCH_CHECK(false, "Green Context is only supported on CUDA 12.8+!");
#endif
}

uint32_t GreenContext::max_workqueue_concurrency(
    std::optional<uint32_t> device_id) {
#if HAS_CUDA_WORKQUEUE_SUPPORT()
  int driver_version;
  C10_CUDA_CHECK(cudaDriverGetVersion(&driver_version));
```
- EN: Focus symbols: `TORCH_CHECK`, `max_workqueue_concurrency`, `C10_CUDA_CHECK`, `cudaDriverGetVersion`. This block implements callable behavior together with runtime validation. The checks reject unsupported states before delegating work to lower-level helpers.
- CN: 关注符号：`TORCH_CHECK`, `max_workqueue_concurrency`, `C10_CUDA_CHECK`, `cudaDriverGetVersion`。该代码块实现可调用行为并包含运行时校验。这些检查会在把工作下发给更底层辅助函数前拒绝不支持的状态。

### Lines 153-162
```cpp
  TORCH_CHECK(
      driver_version >= 13010, "cuda driver too old to use workqueue configuration!");
  if (!device_id.has_value()) {
    device_id = at::cuda::current_device();
  }
  CUdevice device;
  C10_CUDA_DRIVER_CHECK(
      c10::cuda::DriverAPI::get()->cuDeviceGet_(&device, device_id.value()));
  CUdevResource wq_resource;
  C10_CUDA_DRIVER_CHECK(c10::cuda::DriverAPI::get()->cuDeviceGetDevResource_(
```
- EN: Focus symbols: `TORCH_CHECK`, `has_value`, `current_device`, `C10_CUDA_DRIVER_CHECK`, `get`. This block implements callable behavior together with runtime validation. The checks reject unsupported states before delegating work to lower-level helpers.
- CN: 关注符号：`TORCH_CHECK`, `has_value`, `current_device`, `C10_CUDA_DRIVER_CHECK`, `get`。该代码块实现可调用行为并包含运行时校验。这些检查会在把工作下发给更底层辅助函数前拒绝不支持的状态。

### Lines 163-172
```cpp
      device, &wq_resource, CU_DEV_RESOURCE_TYPE_WORKQUEUE_CONFIG));
  return wq_resource.wqConfig.wqConcurrencyLimit;
#else
  TORCH_CHECK(false, "Workqueue configuration requires CUDA 13.1+!");
#endif
}

  // Implement move operations
#if HAS_CUDA_GREEN_CONTEXT()
  GreenContext::GreenContext(GreenContext&& other) noexcept
```
- EN: Focus symbols: `TORCH_CHECK`, `GreenContext`. This block implements callable behavior together with runtime validation. The checks reject unsupported states before delegating work to lower-level helpers.
- CN: 关注符号：`TORCH_CHECK`, `GreenContext`。该代码块实现可调用行为并包含运行时校验。这些检查会在把工作下发给更底层辅助函数前拒绝不支持的状态。

### Lines 173-185
```cpp
      : device_id_(std::exchange(other.device_id_, -1)),
        green_ctx_(std::exchange(other.green_ctx_, nullptr)),
        context_(std::exchange(other.context_, nullptr)),
        parent_stream_(std::exchange(other.parent_stream_, nullptr)) {
        curr_stream_idx_.exchange(other.curr_stream_idx_);
        std::swap(this->green_ctx_streams_, other.green_ctx_streams_);
  }
#else
  GreenContext::GreenContext(GreenContext&& other) noexcept {
    TORCH_CHECK(false, "Green Context move constructor is only supported on CUDA 12.8+!");
  }
#endif

```
- EN: Focus symbols: `device_id_`, `exchange`, `green_ctx_`, `context_`, `parent_stream_`. This block implements callable behavior together with runtime validation. The checks reject unsupported states before delegating work to lower-level helpers.
- CN: 关注符号：`device_id_`, `exchange`, `green_ctx_`, `context_`, `parent_stream_`。该代码块实现可调用行为并包含运行时校验。这些检查会在把工作下发给更底层辅助函数前拒绝不支持的状态。

### Lines 186-195
```cpp
  GreenContext& GreenContext::operator=(GreenContext&& other) noexcept {
#if HAS_CUDA_GREEN_CONTEXT()
    if (this != &other) {
      // Clean up current resources
      if (green_ctx_) {
        CUcontext current = nullptr;
        C10_CUDA_DRIVER_CHECK(
            c10::cuda::DriverAPI::get()->cuCtxGetCurrent_(&current));
        if (current == context_) {
          TORCH_CHECK(
```
- EN: Focus symbols: `C10_CUDA_DRIVER_CHECK`, `get`, `cuCtxGetCurrent_`, `TORCH_CHECK`. This block implements callable behavior together with runtime validation. The checks reject unsupported states before delegating work to lower-level helpers.
- CN: 关注符号：`C10_CUDA_DRIVER_CHECK`, `get`, `cuCtxGetCurrent_`, `TORCH_CHECK`。该代码块实现可调用行为并包含运行时校验。这些检查会在把工作下发给更底层辅助函数前拒绝不支持的状态。

### Lines 196-205
```cpp
              false,
              "attempting to overwrite current green ctx "
              "when it is active!");
        }
        C10_CUDA_DRIVER_CHECK(c10::cuda::DriverAPI::get()->cuGreenCtxDestroy_(green_ctx_));
      }

      // Take ownership of other's resources
      device_id_ = std::exchange(other.device_id_, -1);
      green_ctx_ = std::exchange(other.green_ctx_, nullptr);
```
- EN: Focus symbols: `C10_CUDA_DRIVER_CHECK`, `get`, `cuGreenCtxDestroy_`, `exchange`. This block declares or implements callable behavior for this module. Inputs are translated into lower-level operations, state updates, or returned values here.
- CN: 关注符号：`C10_CUDA_DRIVER_CHECK`, `get`, `cuGreenCtxDestroy_`, `exchange`。该代码块声明或实现该模块的可调用行为。这里把输入转换为更底层操作、状态更新或返回值。

### Lines 206-216
```cpp
      context_ = std::exchange(other.context_, nullptr);
      parent_stream_ = std::exchange(other.parent_stream_, nullptr);
      curr_stream_idx_.exchange(other.curr_stream_idx_);
      std::swap(this->green_ctx_streams_, other.green_ctx_streams_);
    }
    return *this;
#else
    TORCH_CHECK(false, "Green Context is only supported on CUDA 12.8+!");
#endif
  }

```
- EN: Focus symbols: `exchange`, `swap`, `TORCH_CHECK`. This block implements callable behavior together with runtime validation. The checks reject unsupported states before delegating work to lower-level helpers.
- CN: 关注符号：`exchange`, `swap`, `TORCH_CHECK`。该代码块实现可调用行为并包含运行时校验。这些检查会在把工作下发给更底层辅助函数前拒绝不支持的状态。

### Lines 217-226
```cpp
  GreenContext::~GreenContext() noexcept{
#if HAS_CUDA_GREEN_CONTEXT()
    C10_CUDA_DRIVER_CHECK(
        c10::cuda::DriverAPI::get()->cuGreenCtxDestroy_(green_ctx_));
#else
    TORCH_CHECK(false, "Green Context is only supported on CUDA 12.8+!");
#endif
  }

  // Make this context current
```
- EN: Focus symbols: `~GreenContext`, `C10_CUDA_DRIVER_CHECK`, `get`, `cuGreenCtxDestroy_`, `TORCH_CHECK`. This block implements callable behavior together with runtime validation. The checks reject unsupported states before delegating work to lower-level helpers.
- CN: 关注符号：`~GreenContext`, `C10_CUDA_DRIVER_CHECK`, `get`, `cuGreenCtxDestroy_`, `TORCH_CHECK`。该代码块实现可调用行为并包含运行时校验。这些检查会在把工作下发给更底层辅助函数前拒绝不支持的状态。

### Lines 227-236
```cpp
  void GreenContext::setContext() {
#if HAS_CUDA_GREEN_CONTEXT()
    auto current_stream = c10::cuda::getCurrentCUDAStream();
    parent_stream_ = current_stream.stream();

    at::cuda::CUDAEvent ev;
    ev.record(current_stream);

    CUcontext current = nullptr;
    C10_CUDA_DRIVER_CHECK(
```
- EN: Focus symbols: `setContext`, `getCurrentCUDAStream`, `stream`, `record`, `C10_CUDA_DRIVER_CHECK`. This block declares or implements callable behavior for this module. Inputs are translated into lower-level operations, state updates, or returned values here.
- CN: 关注符号：`setContext`, `getCurrentCUDAStream`, `stream`, `record`, `C10_CUDA_DRIVER_CHECK`。该代码块声明或实现该模块的可调用行为。这里把输入转换为更底层操作、状态更新或返回值。

### Lines 237-246
```cpp
        c10::cuda::DriverAPI::get()->cuCtxGetCurrent_(&current));
    if (!current) {
      C10_CUDA_DRIVER_CHECK(
          c10::cuda::DriverAPI::get()->cuCtxSetCurrent_(context_));
    } else {
      C10_CUDA_DRIVER_CHECK(
          c10::cuda::DriverAPI::get()->cuCtxPushCurrent_(context_));
    }
    // setContext API uses default stream
    // see GreenContext::Stream() for side-stream creation
```
- EN: Focus symbols: `get`, `cuCtxGetCurrent_`, `C10_CUDA_DRIVER_CHECK`, `cuCtxSetCurrent_`, `cuCtxPushCurrent_`. This block declares or implements callable behavior for this module. Inputs are translated into lower-level operations, state updates, or returned values here.
- CN: 关注符号：`get`, `cuCtxGetCurrent_`, `C10_CUDA_DRIVER_CHECK`, `cuCtxSetCurrent_`, `cuCtxPushCurrent_`。该代码块声明或实现该模块的可调用行为。这里把输入转换为更底层操作、状态更新或返回值。

### Lines 247-256
```cpp
    auto green_ctx_stream = c10::cuda::getDefaultCUDAStream();
    ev.block(green_ctx_stream);
    c10::cuda::setCurrentCUDAStream(c10::cuda::CUDAStream(green_ctx_stream));
#else
    TORCH_CHECK(false, "Green Context is only supported on CUDA 12.8+!");
#endif
  }

  void GreenContext::popContext() {
#if HAS_CUDA_GREEN_CONTEXT()
```
- EN: Focus symbols: `getDefaultCUDAStream`, `block`, `setCurrentCUDAStream`, `CUDAStream`, `TORCH_CHECK`. This block implements callable behavior together with runtime validation. The checks reject unsupported states before delegating work to lower-level helpers.
- CN: 关注符号：`getDefaultCUDAStream`, `block`, `setCurrentCUDAStream`, `CUDAStream`, `TORCH_CHECK`。该代码块实现可调用行为并包含运行时校验。这些检查会在把工作下发给更底层辅助函数前拒绝不支持的状态。

### Lines 257-266
```cpp
    // see above note about stream being hardcoded to the default stream
    at::cuda::CUDAEvent ev;
    ev.record(c10::cuda::getCurrentCUDAStream());
    CUcontext popped;
    C10_CUDA_DRIVER_CHECK(
        c10::cuda::DriverAPI::get()->cuCtxPopCurrent_(&popped));
    TORCH_INTERNAL_ASSERT(
        popped == context_, "expected popped context to be the current ctx");
    auto parent_stream = c10::cuda::getStreamFromExternal(parent_stream_, device_id_);
    ev.block(parent_stream);
```
- EN: Focus symbols: `record`, `getCurrentCUDAStream`, `C10_CUDA_DRIVER_CHECK`, `get`, `cuCtxPopCurrent_`. This block implements callable behavior together with runtime validation. The checks reject unsupported states before delegating work to lower-level helpers.
- CN: 关注符号：`record`, `getCurrentCUDAStream`, `C10_CUDA_DRIVER_CHECK`, `get`, `cuCtxPopCurrent_`。该代码块实现可调用行为并包含运行时校验。这些检查会在把工作下发给更底层辅助函数前拒绝不支持的状态。

### Lines 267-276
```cpp
    c10::cuda::setCurrentCUDAStream(parent_stream);
#else
    TORCH_CHECK(false, "Green Context is only supported on CUDA 12.8+!");
#endif
  }

  CUDAStream GreenContext::Stream() {
#if HAS_CUDA_GREEN_CONTEXT()
    curr_stream_idx_++;
    auto idx = curr_stream_idx_ % kStreamPerGreenContextPool;
```
- EN: Focus symbols: `setCurrentCUDAStream`, `TORCH_CHECK`, `Stream`. This block implements callable behavior together with runtime validation. The checks reject unsupported states before delegating work to lower-level helpers.
- CN: 关注符号：`setCurrentCUDAStream`, `TORCH_CHECK`, `Stream`。该代码块实现可调用行为并包含运行时校验。这些检查会在把工作下发给更底层辅助函数前拒绝不支持的状态。

### Lines 277-286
```cpp
    if (curr_stream_idx_ < kStreamPerGreenContextPool) {
       CUstream green_ctx_side_stream;
       C10_CUDA_DRIVER_CHECK(c10::cuda::DriverAPI::get()->cuGreenCtxStreamCreate_(
         &green_ctx_side_stream, green_ctx_, CU_STREAM_NON_BLOCKING, 0));
       // implies we leak side-streams, but this has precedent in e.g., c10/cuda/CUDAStream.cpp
       // if we do not have any statically allocated GreenContexts, would it be safe to
       // destroy these streams in a destructor?
       green_ctx_streams_[idx] = green_ctx_side_stream;
       return c10::cuda::getStreamFromExternal(green_ctx_side_stream, device_id_);
    }
```
- EN: Focus symbols: `C10_CUDA_DRIVER_CHECK`, `get`, `cuGreenCtxStreamCreate_`, `getStreamFromExternal`. This block declares or implements callable behavior for this module. Inputs are translated into lower-level operations, state updates, or returned values here.
- CN: 关注符号：`C10_CUDA_DRIVER_CHECK`, `get`, `cuGreenCtxStreamCreate_`, `getStreamFromExternal`。该代码块声明或实现该模块的可调用行为。这里把输入转换为更底层操作、状态更新或返回值。

### Lines 287-292
```cpp
    return c10::cuda::getStreamFromExternal(green_ctx_streams_[idx], device_id_);
#else
    TORCH_CHECK(false, "Green Context is only supported on CUDA 12.8+!");
#endif
  }
} // namespace at::cuda
```
- EN: Focus symbols: `at::cuda`, `getStreamFromExternal`, `TORCH_CHECK`. This block implements callable behavior together with runtime validation. The checks reject unsupported states before delegating work to lower-level helpers.
- CN: 关注符号：`at::cuda`, `getStreamFromExternal`, `TORCH_CHECK`。该代码块实现可调用行为并包含运行时校验。这些检查会在把工作下发给更底层辅助函数前拒绝不支持的状态。

## Key Concepts / 关键概念
- CUDA backend support / CUDA 后端支持
- Conditional compilation / 条件编译
- Type-system design / 类型系统设计
- Runtime validation / 运行时校验
- ATen/C10 internal layering / ATen/C10 内部分层

## Dependencies / 依赖关系
- Direct internal includes / 直接内部依赖: `ATen/cuda/CUDAGreenContext.h`, `c10/cuda/driver_api.h`
- External/system includes / 外部或系统头: `stdexcept`, `vector`
- Companion translation units / 配套实现或声明文件: `aten/src/ATen/cuda/CUDAGreenContext.h`
- Inferred semantic dependencies / 推断出的语义依赖: runtime validation / 运行时校验; namespace scoping / 命名空间作用域
