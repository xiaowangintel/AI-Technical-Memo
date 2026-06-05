# CUDAGraph.cpp — Code Analysis / 代码分析
## Source / 来源
- File: `aten/src/ATen/cuda/CUDAGraph.cpp`
- Repository: `pytorch/pytorch`
- Purpose (EN): This file implements CUDA backend support, with primary focus on `is_user_created`, `at::cuda`, `is_graph_capture_active`.
- 用途（中文）: 该文件实现CUDA 后端支持，核心关注对象是 `is_user_created`, `at::cuda`, `is_graph_capture_active`。

## Line-by-Line Analysis / 逐行分析

### Lines 1-18
```cpp
#include <ATen/core/CachingHostAllocator.h>
#include <ATen/cuda/CUDAContextLight.h>
#include <ATen/cuda/CUDAGeneratorImpl.h>
#include <ATen/cuda/CUDAGraph.h>
#include <ATen/cuda/CUDAGraphsUtils.cuh>
#include <ATen/cuda/Exceptions.h>
#include <ATen/cuda/MemPool.h>
#include <ATen/Functions.h>
#include <c10/cuda/CUDAAllocatorConfig.h>
#include <c10/cuda/CUDAFunctions.h>

#include <cstddef>
#include <optional>

namespace at::cuda {

static bool _cuda_graphs_debug = false;

```
- EN: Focus symbols: `at::cuda`. This block sets up header dependencies and the initial compilation context. The file brings required declarations into scope before entering the main ATen logic.
- CN: 关注符号：`at::cuda`。该代码块设置头文件依赖与初始编译上下文。文件在进入主要 ATen 逻辑前先引入所需声明。

### Lines 19-37
```cpp
// To support stream capture across multiple threads, we use a global
// hashmap mapping cuda stream capture IDs to CUDAGraph objects. This
// was originally a thread_local std::stack<CUDAGraph*>, but that was
// not acceptable since stream capture does span threads in certain
// circumstances (in particular, during autograd).
static std::mutex _currently_capturing_graphs_mutex;
static ska::flat_hash_map<CaptureId_t, CUDAGraph*> _currently_capturing_graphs;


#if defined(USE_ROCM)
// Returns true when at least one CUDAGraph capture is currently active in this
// process. Uses the same mutex-protected capture map as capture lifecycle
// bookkeeping.
bool is_graph_capture_active() {
  std::unique_lock<std::mutex> lock(_currently_capturing_graphs_mutex);
  return !_currently_capturing_graphs.empty();
}
#endif // defined(USE_ROCM)

```
- EN: Focus symbols: `is_graph_capture_active`, `lock`, `empty`. This block declares or implements callable behavior for this module. Inputs are translated into lower-level operations, state updates, or returned values here.
- CN: 关注符号：`is_graph_capture_active`, `lock`, `empty`。该代码块声明或实现该模块的可调用行为。这里把输入转换为更底层操作、状态更新或返回值。

### Lines 38-53
```cpp
MempoolId_t graph_pool_handle() {
  // Sets just the second value, to distinguish it from MempoolId_ts created from
  // cudaStreamGetCaptureInfo id_s in capture_begin.
  return at::cuda::MemPool::graph_pool_handle();
}

/**
 * Note [CUDA Graph Wrapper Class]
 * ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
 * Q: Why do we need graph capture and launch bindings in Pytorch?
 *    Why can't they live in a user extension, for example?
 *
 * A1: Convenience.
 * A2: To ensure valid numerics on replay, some native CUDA ops (like RNG ops with
 *     CPU statefulness) need cooperation from the capture and replay bindings
 *     (see Note [CUDA Graph-safe RNG states] in CUDAGeneratorImpl.h).
```
- EN: Focus symbols: `graph_pool_handle`, `ops`. This block declares or implements callable behavior for this module. Inputs are translated into lower-level operations, state updates, or returned values here.
- CN: 关注符号：`graph_pool_handle`, `ops`。该代码块声明或实现该模块的可调用行为。这里把输入转换为更底层操作、状态更新或返回值。

### Lines 54-70
```cpp
 *
 *     We can't expect users to know about this cooperation.  If users write capture
 *     bindings naively in an extension, they likely won't interact with the native
 *     ops properly.  Their graphs would yield invalid numerics on replay.
 */

/**
 * Note [Interaction with CUDA graph capture] in CUDACachingAllocator.cpp
 * describes memory management for captures.
 */

CUDAGraph::CUDAGraph(bool keep_graph)
  // CUDAStreams may not be default-constructed.
  : capture_stream_(at::cuda::getCurrentCUDAStream()),
    keep_graph_(keep_graph) {
}

```
- EN: Focus symbols: `CUDAGraph`, `capture_stream_`, `getCurrentCUDAStream`, `keep_graph_`. This block declares or implements callable behavior for this module. Inputs are translated into lower-level operations, state updates, or returned values here.
- CN: 关注符号：`CUDAGraph`, `capture_stream_`, `getCurrentCUDAStream`, `keep_graph_`。该代码块声明或实现该模块的可调用行为。这里把输入转换为更底层操作、状态更新或返回值。

### Lines 71-86
```cpp
void CUDAGraph::register_generator_state(
    c10::intrusive_ptr<at::CUDAGeneratorState> state) {
  captured_generator_states_[std::move(state)] = 0;
}

void CUDAGraph::register_generator_state(const at::Generator& generator) {
  c10::intrusive_ptr<CUDAGeneratorImpl> cuda_gen =
      dynamic_intrusive_pointer_cast<CUDAGeneratorImpl>(
          generator.getIntrusivePtr());
  cuda_gen->register_graph(this);
}


template <>
std::function<bool(cudaStream_t)> CUDAGraph::create_allocate_filter<cudaStream_t>() const {
  return [this](cudaStream_t stream) {
```
- EN: Focus symbols: `register_generator_state`, `move`, `getIntrusivePtr`, `register_graph`, `bool`. This block builds generic/template machinery for multiple dtypes, devices, or call patterns. Templates let ATen share one design across many concrete instantiations.
- CN: 关注符号：`register_generator_state`, `move`, `getIntrusivePtr`, `register_graph`, `bool`。该代码块构建适配多种数据类型、设备或调用模式的模板机制。模板让 ATen 能在多个具体实例之间共享同一套设计。

### Lines 87-105
```cpp
    auto capture_id_opt = c10::cuda::captureIdMayInitCtx(stream);
    return capture_id_opt.has_value() && capture_id_opt.value() == capture_id_;
  };
}

template <>
std::function<bool(c10::Stream)> CUDAGraph::create_allocate_filter<c10::Stream>() const {
  return [this](c10::Stream stream) {
    cudaStream_t cuda_stream = CUDAStream(CUDAStream::UNCHECKED, stream);
    auto capture_id_opt = c10::cuda::captureIdMayInitCtx(cuda_stream);
    return capture_id_opt.has_value() && capture_id_opt.value() == capture_id_;
  };
}

void CUDAGraph::capture_begin(MempoolId_t pool/*={0,0}*/, cudaStreamCaptureMode capture_mode) {
  TORCH_CHECK(!has_graph_exec_,
              "This CUDAGraph instance already owns a captured graph. "
              "To capture a new graph, create a new instance.");

```
- EN: Focus symbols: `captureIdMayInitCtx`, `has_value`, `value`, `bool`, `CUDAStream`. This block builds generic/template machinery for multiple dtypes, devices, or call patterns. Templates let ATen share one design across many concrete instantiations.
- CN: 关注符号：`captureIdMayInitCtx`, `has_value`, `value`, `bool`, `CUDAStream`。该代码块构建适配多种数据类型、设备或调用模式的模板机制。模板让 ATen 能在多个具体实例之间共享同一套设计。

### Lines 106-122
```cpp
  capture_mode_ = capture_mode;

  // default generator is always registered
  auto* gen = get_generator_or_default<CUDAGeneratorImpl>(
      std::nullopt, cuda::detail::getDefaultCUDAGenerator());
  gen->register_graph(this);

  auto stream = at::cuda::getCurrentCUDAStream();

  TORCH_CHECK(stream != at::cuda::getDefaultCUDAStream(),
              "CUDA graphs must be captured on a non-default stream. "
              "(However, after capture, it's ok to replay them on the "
              "default stream.)");

  capture_stream_ = stream;
  capture_dev_ = c10::cuda::current_device();

```
- EN: Focus symbols: `getDefaultCUDAGenerator`, `register_graph`, `getCurrentCUDAStream`, `TORCH_CHECK`, `getDefaultCUDAStream`. This block implements callable behavior together with runtime validation. The checks reject unsupported states before delegating work to lower-level helpers.
- CN: 关注符号：`getDefaultCUDAGenerator`, `register_graph`, `getCurrentCUDAStream`, `TORCH_CHECK`, `getDefaultCUDAStream`。该代码块实现可调用行为并包含运行时校验。这些检查会在把工作下发给更底层辅助函数前拒绝不支持的状态。

### Lines 123-138
```cpp
#if defined(USE_ROCM)
  // hipBLASLt handles are per-(device, stream) on ROCm and lazily created.
  // Ensure the handle for the intended capture stream exists before
  // capture begins, because hipblasLtCreate performs internal allocations
  // that are not allowed once stream capture is active.
  if (at::globalContext().blasPreferredBackend() == at::BlasBackend::Cublaslt) {
    (void)at::cuda::getCurrentCUDABlasLtHandle();
  }
#endif

  if (pool.first != 0 || pool.second != 0) {
    // Either value being nonzero means the user supplied a pool to share.
    // But only one should be nonzero.
    // If pool was created by another graph's capture_begin, first should be nonzero.
    // If pool was created by graph_pool_handle, second should be nonzero.
    TORCH_INTERNAL_ASSERT(!(pool.first && pool.second));
```
- EN: Focus symbols: `globalContext`, `blasPreferredBackend`, `getCurrentCUDABlasLtHandle`, `TORCH_INTERNAL_ASSERT`. This block implements callable behavior together with runtime validation. The checks reject unsupported states before delegating work to lower-level helpers.
- CN: 关注符号：`globalContext`, `blasPreferredBackend`, `getCurrentCUDABlasLtHandle`, `TORCH_INTERNAL_ASSERT`。该代码块实现可调用行为并包含运行时校验。这些检查会在把工作下发给更底层辅助函数前拒绝不支持的状态。

### Lines 139-158
```cpp
    mempool_id_ = pool;
  } else {
    // User did not ask us to share a mempool. Create graph pool handle using is_user_created=false.
    // Sets just the first value, to distinguish it from MempoolId_ts created by graph_pool_handle().
    mempool_id_ = at::cuda::MemPool::graph_pool_handle(false);
    TORCH_INTERNAL_ASSERT(mempool_id_.first > 0);
  }

  // Addendum: beginAllocateStreamToPool is now called before cudaStreamBeginCapture to prevent an
  // autograd thread's free() call triggering an invalid cudaEventRecord in the caching allocator
  // due to the capture status being updated _after_ a capture had already started.
  c10::cuda::CUDACachingAllocator::beginAllocateToPool(capture_dev_, mempool_id_, create_allocate_filter<cudaStream_t>());

  at::getHostAllocator(at::kCUDA)->begin_allocate_to_pool(mempool_id_, create_allocate_filter<c10::Stream>());

  // cudaStreamCaptureModeGlobal is the most conservative option to
  // prevent potentially unsafe CUDA API calls during capture.  See
  // https://docs.nvidia.com/cuda/cuda-runtime-api/group__CUDART__STREAM.html#group__CUDART__STREAM_1g9d0535d93a214cbf126835257b16ba85
  AT_CUDA_CHECK(cudaStreamBeginCapture(capture_stream_, capture_mode));

```
- EN: Focus symbols: `is_user_created`, `graph_pool_handle`, `TORCH_INTERNAL_ASSERT`, `beginAllocateToPool`, `getHostAllocator`. This block declares the main types, aliases, or compile-time descriptors used by this file. These declarations define the shape of the API and the data carried through later code.
- CN: 关注符号：`is_user_created`, `graph_pool_handle`, `TORCH_INTERNAL_ASSERT`, `beginAllocateToPool`, `getHostAllocator`。该代码块声明此文件使用的核心类型、别名或编译期描述对象。这些声明决定了 API 的形状以及后续代码中流动的数据结构。

### Lines 159-177
```cpp
  auto capture_id_opt = c10::cuda::captureIdMayInitCtx(stream);
  TORCH_INTERNAL_ASSERT(capture_id_opt.has_value(),
      "Stream should be actively capturing after cudaStreamBeginCapture");
  capture_id_ = capture_id_opt.value();

  {
    std::lock_guard<std::mutex> lock(_currently_capturing_graphs_mutex);
    _currently_capturing_graphs.emplace(capture_id_, this);
  }

  for (auto& [generator_state, wholegraph_increment] :
       captured_generator_states_) {
    generator_state->init_capture_state(capture_id_);
  }
}

void CUDAGraph::capture_end() {
  auto stream = at::cuda::getCurrentCUDAStream();

```
- EN: Focus symbols: `captureIdMayInitCtx`, `TORCH_INTERNAL_ASSERT`, `has_value`, `value`, `lock`. This block implements callable behavior together with runtime validation. The checks reject unsupported states before delegating work to lower-level helpers.
- CN: 关注符号：`captureIdMayInitCtx`, `TORCH_INTERNAL_ASSERT`, `has_value`, `value`, `lock`。该代码块实现可调用行为并包含运行时校验。这些检查会在把工作下发给更底层辅助函数前拒绝不支持的状态。

### Lines 178-196
```cpp
  TORCH_CHECK(stream.stream() == capture_stream_.stream(),
              "Capture must end on the same stream it began on.");

  // Capture is over once cudaStreamEndCapture returns (success or failure).
  // Clear bookkeeping before propagating the return status so watchdog-side
  // checks cannot observe stale "capture active" state on error paths.
  cudaError_t endCaptureErr = cudaStreamEndCapture(capture_stream_, &graph_);
  {
    std::unique_lock<std::mutex> lock(_currently_capturing_graphs_mutex);
    TORCH_CHECK(
        _currently_capturing_graphs.count(capture_id_),
        "capture_end() called before capture_begin().");
    _currently_capturing_graphs.erase(capture_id_);
  }

  c10::cuda::CUDACachingAllocator::endAllocateToPool(capture_dev_, mempool_id_);
  at::getHostAllocator(at::kCUDA)->end_allocate_to_pool(mempool_id_);
  AT_CUDA_CHECK(endCaptureErr);

```
- EN: Focus symbols: `TORCH_CHECK`, `stream`, `cudaStreamEndCapture`, `lock`, `count`. This block implements callable behavior together with runtime validation. The checks reject unsupported states before delegating work to lower-level helpers.
- CN: 关注符号：`TORCH_CHECK`, `stream`, `cudaStreamEndCapture`, `lock`, `count`。该代码块实现可调用行为并包含运行时校验。这些检查会在把工作下发给更底层辅助函数前拒绝不支持的状态。

### Lines 197-212
```cpp
  TORCH_CHECK(graph_ != nullptr, "Invalid capture.");

  for (auto& [generator_state, wholegraph_increment] :
       captured_generator_states_) {
    wholegraph_increment = generator_state->capture_epilogue(capture_id_);
  }

  size_t numCUDAGraphNodes = 0;
  AT_CUDA_CHECK(cudaGraphGetNodes(graph_, nullptr, &numCUDAGraphNodes));
  if (numCUDAGraphNodes == 0) {
      TORCH_WARN("The CUDA Graph is empty. This usually means that the graph was ",
                 "attempted to be captured on wrong device or stream.");
  }

  capture_ended_ = true;
  has_graph_ = true;
```
- EN: Focus symbols: `TORCH_CHECK`, `capture_epilogue`, `AT_CUDA_CHECK`, `cudaGraphGetNodes`, `TORCH_WARN`. This block implements callable behavior together with runtime validation. The checks reject unsupported states before delegating work to lower-level helpers.
- CN: 关注符号：`TORCH_CHECK`, `capture_epilogue`, `AT_CUDA_CHECK`, `cudaGraphGetNodes`, `TORCH_WARN`。该代码块实现可调用行为并包含运行时校验。这些检查会在把工作下发给更底层辅助函数前拒绝不支持的状态。

### Lines 213-228
```cpp
  if (!keep_graph_) {
    instantiate();
    if (!_cuda_graphs_debug) {
      AT_CUDA_CHECK(cudaGraphDestroy(graph_));
    }
    has_graph_ = false;
  }
}

void CUDAGraph::instantiate() {
  TORCH_CHECK(capture_ended_, "capture_end() must have been called before calling instantiate");

  if (has_graph_exec_) {
    TORCH_CHECK(keep_graph_, "instantiate() is intended to be called by the user only when keep_graph=true");
    AT_CUDA_CHECK(cudaGraphExecDestroy(graph_exec_));
  }
```
- EN: Focus symbols: `instantiate`, `AT_CUDA_CHECK`, `cudaGraphDestroy`, `TORCH_CHECK`, `capture_end`. This block implements callable behavior together with runtime validation. The checks reject unsupported states before delegating work to lower-level helpers.
- CN: 关注符号：`instantiate`, `AT_CUDA_CHECK`, `cudaGraphDestroy`, `TORCH_CHECK`, `capture_end`。该代码块实现可调用行为并包含运行时校验。这些检查会在把工作下发给更底层辅助函数前拒绝不支持的状态。

### Lines 229-244
```cpp
  // In typical graph usage some tensors (e.g. the tensors used for graph IO) are not freed
  // between replays.
  // If Pytorch compiles and runs with a CUDA 11.4+ toolkit, there's a chance the allocator backend
  // is cudaMallocAsync.
  // cudaMallocAsync is generally graph-safe, but if some tensors are not freed between replays,
  // the graph's internal bookkeeping requires that we instantiate with
  // cudaGraphInstantiateFlagAutoFreeOnLaunch. See
  // cudaGraphLaunch
  // https://docs.nvidia.com/cuda/cuda-runtime-api/group__CUDART__GRAPH.html#group__CUDART__GRAPH_1g1accfe1da0c605a577c22d9751a09597
  // cudaGraphInstantiateWithFlags
  // https://docs.nvidia.com/cuda/cuda-runtime-api/group__CUDART__GRAPH.html#group__CUDART__GRAPH_1ga2c652a24ba93e52b99a47bec0888233
#if !defined(USE_ROCM)
    AT_CUDA_CHECK(cudaGraphInstantiateWithFlags(&graph_exec_,
                                                graph_,
                                                cudaGraphInstantiateFlagAutoFreeOnLaunch | cudaGraphInstantiateFlagUseNodePriority));
#else
```
- EN: Focus symbols: `AT_CUDA_CHECK`, `cudaGraphInstantiateWithFlags`. This block declares or implements callable behavior for this module. Inputs are translated into lower-level operations, state updates, or returned values here.
- CN: 关注符号：`AT_CUDA_CHECK`, `cudaGraphInstantiateWithFlags`。该代码块声明或实现该模块的可调用行为。这里把输入转换为更底层操作、状态更新或返回值。

### Lines 245-262
```cpp
    AT_CUDA_CHECK(cudaGraphInstantiateWithFlags(&graph_exec_,
                                                graph_,
                                                cudaGraphInstantiateFlagAutoFreeOnLaunch));
#endif
  has_graph_exec_ = true;
}

void CUDAGraph::replay() {
  TORCH_CHECK(capture_ended_,
              "Called CUDAGraph::replay without a preceding successful capture.");

  if (!has_graph_exec_) {
    TORCH_INTERNAL_ASSERT(keep_graph_);
    instantiate();
  }

  c10::OptionalDeviceGuard device_guard{capture_stream_.device()};

```
- EN: Focus symbols: `AT_CUDA_CHECK`, `cudaGraphInstantiateWithFlags`, `replay`, `TORCH_CHECK`, `TORCH_INTERNAL_ASSERT`. This block implements callable behavior together with runtime validation. The checks reject unsupported states before delegating work to lower-level helpers.
- CN: 关注符号：`AT_CUDA_CHECK`, `cudaGraphInstantiateWithFlags`, `replay`, `TORCH_CHECK`, `TORCH_INTERNAL_ASSERT`。该代码块实现可调用行为并包含运行时校验。这些检查会在把工作下发给更底层辅助函数前拒绝不支持的状态。

### Lines 263-278
```cpp
  for (auto& [generator_state, wholegraph_increment] :
       captured_generator_states_) {
    generator_state->replay_prologue(capture_id_, wholegraph_increment);
  }
  // graph_exec_ may be replayed in any stream.
  AT_CUDA_CHECK(cudaGraphLaunch(graph_exec_, at::cuda::getCurrentCUDAStream()));
}

void CUDAGraph::enable_debug_mode() {
  _cuda_graphs_debug = true;
}

void CUDAGraph::debug_dump(const std::string& debug_path) {
  if (_cuda_graphs_debug || keep_graph_) {
    TORCH_WARN("DEBUG: calling debug_dump()");
    if (has_graph_) {
```
- EN: Focus symbols: `replay_prologue`, `AT_CUDA_CHECK`, `cudaGraphLaunch`, `getCurrentCUDAStream`, `enable_debug_mode`. This block declares or implements callable behavior for this module. Inputs are translated into lower-level operations, state updates, or returned values here.
- CN: 关注符号：`replay_prologue`, `AT_CUDA_CHECK`, `cudaGraphLaunch`, `getCurrentCUDAStream`, `enable_debug_mode`。该代码块声明或实现该模块的可调用行为。这里把输入转换为更底层操作、状态更新或返回值。

### Lines 279-296
```cpp
      TORCH_WARN("DEBUG: calling cudaGraphDebugDotPrint() with ", debug_path);
      C10_CUDA_CHECK_WARN(cudaGraphDebugDotPrint(graph_, debug_path.c_str(), cudaGraphDebugDotFlagsVerbose)); // most verbose output
      if (!keep_graph_) {
        AT_CUDA_CHECK(cudaGraphDestroy(graph_));
        has_graph_ = false;
      }
    }
  } else {
    TORCH_WARN("CUDA Graphs debug not enabled, set with [graph].enable_debug_mode()");
  }
}

cudaGraph_t CUDAGraph::raw_cuda_graph() {
  TORCH_CHECK(keep_graph_, "You cannot access the raw cudaGraph_t instance unless CUDAGraph was initialized with keep_graph=true");
  TORCH_CHECK(has_graph_, "You cannot access the raw cudaGraph_t instance until capture_end() has been called");
  return graph_;
}

```
- EN: Focus symbols: `TORCH_WARN`, `cudaGraphDebugDotPrint`, `C10_CUDA_CHECK_WARN`, `c_str`, `AT_CUDA_CHECK`. This block implements callable behavior together with runtime validation. The checks reject unsupported states before delegating work to lower-level helpers.
- CN: 关注符号：`TORCH_WARN`, `cudaGraphDebugDotPrint`, `C10_CUDA_CHECK_WARN`, `c_str`, `AT_CUDA_CHECK`。该代码块实现可调用行为并包含运行时校验。这些检查会在把工作下发给更底层辅助函数前拒绝不支持的状态。

### Lines 297-312
```cpp
cudaGraphExec_t CUDAGraph::raw_cuda_graph_exec() {
  TORCH_CHECK(
      has_graph_exec_,
      "You cannot access the raw cudaGraphExec_t instance until instantiate() has been called");
  return graph_exec_;
}

void CUDAGraph::reset() {
  // I'd prefer these checks throw exceptions, not print warnings,
  // but the destructor calls reset(), and at least one CI build
  // refuses to compile with a throwing destructor.
  //
  // Instead of calling reset() in the destructor to clean up, I could
  // call reset() in the __del__ method of a thin Python wrapper,
  // in which case reset would be allowed to throw exceptions.
  // But Stackoverflow does not like user-defined __del__.
```
- EN: Focus symbols: `raw_cuda_graph_exec`, `TORCH_CHECK`, `instantiate`, `reset`. This block implements callable behavior together with runtime validation. The checks reject unsupported states before delegating work to lower-level helpers.
- CN: 关注符号：`raw_cuda_graph_exec`, `TORCH_CHECK`, `instantiate`, `reset`。该代码块实现可调用行为并包含运行时校验。这些检查会在把工作下发给更底层辅助函数前拒绝不支持的状态。

### Lines 313-331
```cpp
  // __del__ prevents Graph instances from EVER being garbage collected
  // if they participate in a reference cycle.
  // And exceptions thrown in __del__ only print a warning anyway.
  //
  // Calling reset() in the C++ destructor, with warnings instead of exceptions
  // if calls fail, is the compromise we chose.
  //
  // If capture_begin, the capture, or capture_end failed at some point, this CUDAGraph, the generator,
  // and the allocator could end up in all kinds of weird states depending where failure occurred.
  // If the user catches the failure exception in a script, or is running in REPL or (god forbid)
  // a Jupyter notebook, I don't see an easy way for reset() to gracefully fix all such possible error states.

  if (capture_id_ != 0) {
    for (auto& [generator_state, wholegraph_increment] : captured_generator_states_) {
      generator_state->remove_capture_state(capture_id_);
    }
  }
  captured_generator_states_.clear();

```
- EN: Focus symbols: `remove_capture_state`, `clear`. This block declares or implements callable behavior for this module. Inputs are translated into lower-level operations, state updates, or returned values here.
- CN: 关注符号：`remove_capture_state`, `clear`。该代码块声明或实现该模块的可调用行为。这里把输入转换为更底层操作、状态更新或返回值。

### Lines 332-347
```cpp
  if (capture_id_ != 0) {
    std::lock_guard<std::mutex> lock(_currently_capturing_graphs_mutex);
    _currently_capturing_graphs.erase(capture_id_);
    capture_id_ = 0;
  }

  if (capture_ended_) {
    // Clean up cuBLAS workspaces allocated on the capture stream, otherwise live allocations prevent
    // private pool cleanup
    clearCublasWorkspacesForStream(capture_stream_.stream());

    // notifyCaptureDestroy may throw. How should we handle this?
    c10::cuda::CUDACachingAllocator::releasePool(capture_dev_, mempool_id_);
    at::getHostAllocator(at::kCUDA)->release_pool(mempool_id_);
    capture_ended_ = false;
  }
```
- EN: Focus symbols: `lock`, `erase`, `clearCublasWorkspacesForStream`, `stream`, `releasePool`. This block declares or implements callable behavior for this module. Inputs are translated into lower-level operations, state updates, or returned values here.
- CN: 关注符号：`lock`, `erase`, `clearCublasWorkspacesForStream`, `stream`, `releasePool`。该代码块声明或实现该模块的可调用行为。这里把输入转换为更底层操作、状态更新或返回值。

### Lines 348-364
```cpp
  if (has_graph_) {
    C10_CUDA_CHECK_WARN(cudaGraphDestroy(graph_));
    has_graph_ = false;
  }
  if (has_graph_exec_) {
    C10_CUDA_CHECK_WARN(cudaGraphExecDestroy(graph_exec_));
    has_graph_exec_ = false;
  }
}

// Returns an id another graph's capture_begin can use to share the same memory pool as this graph.
MempoolId_t CUDAGraph::pool() {
  TORCH_CHECK(capture_ended_,
              "Called CUDAGraph::pool() without a preceding successful capture.");
  return mempool_id_;
}

```
- EN: Focus symbols: `C10_CUDA_CHECK_WARN`, `cudaGraphDestroy`, `cudaGraphExecDestroy`, `pool`, `TORCH_CHECK`. This block implements callable behavior together with runtime validation. The checks reject unsupported states before delegating work to lower-level helpers.
- CN: 关注符号：`C10_CUDA_CHECK_WARN`, `cudaGraphDestroy`, `cudaGraphExecDestroy`, `pool`, `TORCH_CHECK`。该代码块实现可调用行为并包含运行时校验。这些检查会在把工作下发给更底层辅助函数前拒绝不支持的状态。

### Lines 365-380
```cpp
CUDAGraph::~CUDAGraph() {
  reset();

// There are recent HIP changes where hipGraphExecDestroy doesn't immediately free memory.
// They wait for next sync point in order to free the memory, this is to ensure that all
// hipGraphLaunch are finished before we release any memory. This feature was enabled in rocm6.2.
// We need to ensure all async operations finish before deleting the object.
#if defined(USE_ROCM)
  if (capture_dev_ != UNDEFINED_DEVICE) // check if capture_dev_ contains the real device id
  {
    AT_CUDA_CHECK(cudaSetDevice(capture_dev_));
    AT_CUDA_CHECK(cudaDeviceSynchronize());
  }
#endif
}

```
- EN: Focus symbols: `~CUDAGraph`, `reset`, `AT_CUDA_CHECK`, `cudaSetDevice`, `cudaDeviceSynchronize`. This block declares or implements callable behavior for this module. Inputs are translated into lower-level operations, state updates, or returned values here.
- CN: 关注符号：`~CUDAGraph`, `reset`, `AT_CUDA_CHECK`, `cudaSetDevice`, `cudaDeviceSynchronize`。该代码块声明或实现该模块的可调用行为。这里把输入转换为更底层操作、状态更新或返回值。

### Lines 381-399
```cpp
CUDAGraph* CUDAGraph::get_currently_capturing_graph() {
  std::unique_lock<std::mutex> lock(_currently_capturing_graphs_mutex);
  auto capture_id_opt = c10::cuda::currentStreamCaptureIdMayInitCtx();
  TORCH_CHECK(
      capture_id_opt.has_value(),
      "The current stream is not currently capturing.");
  TORCH_CHECK(
      _currently_capturing_graphs.count(capture_id_opt.value()),
      "get_currently_capturing_graph() can be used only between capture_begin() and capture_end(). Did you use a stream without making it depend upon the original stream used for capture?");
  return _currently_capturing_graphs.at(capture_id_opt.value());
}

void CUDAGraph::begin_capture_to_if_node(
    const at::Tensor& scalar_cuda_pred_tensor) {
#if !defined(USE_ROCM) && (defined(CUDA_VERSION) && CUDA_VERSION >= 12040)
  TORCH_CHECK(
      !has_graph_exec_,
      "This CUDAGraph instance already owns a captured graph.");

```
- EN: Focus symbols: `get_currently_capturing_graph`, `lock`, `currentStreamCaptureIdMayInitCtx`, `TORCH_CHECK`, `has_value`. This block implements callable behavior together with runtime validation. The checks reject unsupported states before delegating work to lower-level helpers.
- CN: 关注符号：`get_currently_capturing_graph`, `lock`, `currentStreamCaptureIdMayInitCtx`, `TORCH_CHECK`, `has_value`。该代码块实现可调用行为并包含运行时校验。这些检查会在把工作下发给更底层辅助函数前拒绝不支持的状态。

### Lines 400-415
```cpp
  TORCH_CHECK(!c10::cuda::CUDACachingAllocator::CUDAAllocatorConfig::graph_capture_record_stream_reuse(), "'graph_capture_record_stream_reuse:True' allocator config does not work with conditional control flow in a cuda graph today. See issue #175001 for updates");

  cudaStreamCaptureStatus status{};
  cudaGraph_t currently_capturing_graph{};
  AT_CUDA_CHECK(cudaStreamGetCaptureInfo(
      getCurrentCUDAStream(), &status, nullptr, &currently_capturing_graph));
  TORCH_CHECK(
      status == cudaStreamCaptureStatusActive,
      "capture_begin() must be called before begin_capture_to_if_node()");
  cudaGraphConditionalHandle handle{};
  AT_CUDA_CHECK(cudaGraphConditionalHandleCreate(
      &handle, currently_capturing_graph, 0, 0));

  set_conditional_handle(handle, scalar_cuda_pred_tensor);

  const cudaGraphNode_t* dependencies{};
```
- EN: Focus symbols: `TORCH_CHECK`, `graph_capture_record_stream_reuse`, `AT_CUDA_CHECK`, `cudaStreamGetCaptureInfo`, `getCurrentCUDAStream`. This block implements callable behavior together with runtime validation. The checks reject unsupported states before delegating work to lower-level helpers.
- CN: 关注符号：`TORCH_CHECK`, `graph_capture_record_stream_reuse`, `AT_CUDA_CHECK`, `cudaStreamGetCaptureInfo`, `getCurrentCUDAStream`。该代码块实现可调用行为并包含运行时校验。这些检查会在把工作下发给更底层辅助函数前拒绝不支持的状态。

### Lines 416-431
```cpp
  const cudaGraphEdgeData* dependency_edges{};
  size_t num_dependencies = 0;
#if CUDA_VERSION >= 13000
  AT_CUDA_CHECK(cudaStreamGetCaptureInfo(
      getCurrentCUDAStream(),
      &status,
      nullptr,
      &currently_capturing_graph,
      &dependencies,
      &dependency_edges,
      &num_dependencies));
#else
  AT_CUDA_CHECK(cudaStreamGetCaptureInfo_v3(
      getCurrentCUDAStream(),
      &status,
      nullptr,
```
- EN: Focus symbols: `AT_CUDA_CHECK`, `cudaStreamGetCaptureInfo`, `getCurrentCUDAStream`, `cudaStreamGetCaptureInfo_v3`. This block declares or implements callable behavior for this module. Inputs are translated into lower-level operations, state updates, or returned values here.
- CN: 关注符号：`AT_CUDA_CHECK`, `cudaStreamGetCaptureInfo`, `getCurrentCUDAStream`, `cudaStreamGetCaptureInfo_v3`。该代码块声明或实现该模块的可调用行为。这里把输入转换为更底层操作、状态更新或返回值。

### Lines 432-447
```cpp
      &currently_capturing_graph,
      &dependencies,
      &dependency_edges,
      &num_dependencies
  ));
#endif
  TORCH_CHECK(status == cudaStreamCaptureStatusActive);

  cudaGraphNodeParams params{};
  params.type = cudaGraphNodeTypeConditional;
  params.conditional.handle = handle;
  params.conditional.type = cudaGraphCondTypeIf;
  params.conditional.size = 1;

  cudaGraphNode_t cond_node{};
#if CUDA_VERSION >= 13000
```
- EN: Focus symbols: `TORCH_CHECK`. This block implements callable behavior together with runtime validation. The checks reject unsupported states before delegating work to lower-level helpers.
- CN: 关注符号：`TORCH_CHECK`。该代码块实现可调用行为并包含运行时校验。这些检查会在把工作下发给更底层辅助函数前拒绝不支持的状态。

### Lines 448-465
```cpp
  AT_CUDA_CHECK(cudaGraphAddNode(
      &cond_node,
      currently_capturing_graph,
      dependencies,
      dependency_edges,
      num_dependencies,
      &params));
#else
  AT_CUDA_CHECK(cudaGraphAddNode_v2(
      &cond_node,
      currently_capturing_graph,
      dependencies,
      dependency_edges,
      num_dependencies,
      &params));
#endif
  cudaGraph_t if_node_child_graph = params.conditional.phGraph_out[0];

```
- EN: Focus symbols: `AT_CUDA_CHECK`, `cudaGraphAddNode`, `cudaGraphAddNode_v2`. This block declares or implements callable behavior for this module. Inputs are translated into lower-level operations, state updates, or returned values here.
- CN: 关注符号：`AT_CUDA_CHECK`, `cudaGraphAddNode`, `cudaGraphAddNode_v2`。该代码块声明或实现该模块的可调用行为。这里把输入转换为更底层操作、状态更新或返回值。

### Lines 466-485
```cpp
#if CUDA_VERSION >= 13000
  AT_CUDA_CHECK(cudaStreamUpdateCaptureDependencies(
getCurrentCUDAStream(), &cond_node, nullptr, 1, cudaStreamSetCaptureDependencies));
#else
  AT_CUDA_CHECK(cudaStreamUpdateCaptureDependencies_v2(
getCurrentCUDAStream(), &cond_node, nullptr, 1, cudaStreamSetCaptureDependencies));
#endif

  CUDAStream child_stream = getStreamFromPool();
  conditional_graph_capture_ids_.push(0);

  c10::cuda::CUDACachingAllocator::endAllocateToPool(capture_dev_, mempool_id_);
  at::getHostAllocator(at::kCUDA)->end_allocate_to_pool(mempool_id_);
  c10::cuda::CUDACachingAllocator::beginAllocateToPool(
      capture_dev_, mempool_id_, create_child_allocate_filter());
  auto filter = create_child_allocate_filter();
  at::getHostAllocator(at::kCUDA)->begin_allocate_to_pool(mempool_id_, [filter](c10::Stream stream) {
    return filter(CUDAStream(CUDAStream::UNCHECKED, stream));
  });

```
- EN: Focus symbols: `AT_CUDA_CHECK`, `cudaStreamUpdateCaptureDependencies`, `getCurrentCUDAStream`, `cudaStreamUpdateCaptureDependencies_v2`, `getStreamFromPool`. This block declares or implements callable behavior for this module. Inputs are translated into lower-level operations, state updates, or returned values here.
- CN: 关注符号：`AT_CUDA_CHECK`, `cudaStreamUpdateCaptureDependencies`, `getCurrentCUDAStream`, `cudaStreamUpdateCaptureDependencies_v2`, `getStreamFromPool`。该代码块声明或实现该模块的可调用行为。这里把输入转换为更底层操作、状态更新或返回值。

### Lines 486-501
```cpp
  AT_CUDA_CHECK(cudaStreamBeginCaptureToGraph(
      child_stream, if_node_child_graph, nullptr, nullptr, 0, capture_mode_));

  auto child_capture_id_opt = c10::cuda::captureIdMayInitCtx(child_stream);
  TORCH_INTERNAL_ASSERT(child_capture_id_opt.has_value(),
      "Child stream should be actively capturing after cudaStreamBeginCaptureToGraph");
  conditional_graph_capture_ids_.top() = child_capture_id_opt.value();

  conditional_node_streams_.emplace(child_stream);

  {
    std::unique_lock<std::mutex> lock(_currently_capturing_graphs_mutex);
    _currently_capturing_graphs.emplace(
        conditional_graph_capture_ids_.top(), this);
  }

```
- EN: Focus symbols: `AT_CUDA_CHECK`, `cudaStreamBeginCaptureToGraph`, `captureIdMayInitCtx`, `TORCH_INTERNAL_ASSERT`, `has_value`. This block implements callable behavior together with runtime validation. The checks reject unsupported states before delegating work to lower-level helpers.
- CN: 关注符号：`AT_CUDA_CHECK`, `cudaStreamBeginCaptureToGraph`, `captureIdMayInitCtx`, `TORCH_INTERNAL_ASSERT`, `has_value`。该代码块实现可调用行为并包含运行时校验。这些检查会在把工作下发给更底层辅助函数前拒绝不支持的状态。

### Lines 502-517
```cpp
#else // !defined(USE_ROCM) && (defined(CUDA_VERSION) && CUDA_VERSION >= 12040)
  AT_ERROR(
      __func__,
      " CUDA Graphs conditional nodes are not supported for cuda version < 12.4");
  return;
#endif
}

void CUDAGraph::end_capture_to_conditional_node() {
#if !defined(USE_ROCM) && (defined(CUDA_VERSION) && CUDA_VERSION >= 12040)
  TORCH_INTERNAL_ASSERT(
      !conditional_graph_capture_ids_.empty(),
      "Missing capture ID for conditional node.");

  CaptureId_t child_capture_id = conditional_graph_capture_ids_.top();
  bool rng_or_generators_changed = false;
```
- EN: Focus symbols: `AT_ERROR`, `end_capture_to_conditional_node`, `TORCH_INTERNAL_ASSERT`, `empty`, `top`. This block implements callable behavior together with runtime validation. The checks reject unsupported states before delegating work to lower-level helpers.
- CN: 关注符号：`AT_ERROR`, `end_capture_to_conditional_node`, `TORCH_INTERNAL_ASSERT`, `empty`, `top`。该代码块实现可调用行为并包含运行时校验。这些检查会在把工作下发给更底层辅助函数前拒绝不支持的状态。

### Lines 518-533
```cpp
  for (const auto& [generator_state, wholegraph_increment] :
       captured_generator_states_) {
    if (generator_state->get_capture_state(child_capture_id) != nullptr) {
      rng_or_generators_changed = true;
      break;
    }
  }

  {
    std::unique_lock<std::mutex> lock(_currently_capturing_graphs_mutex);
    TORCH_CHECK(
        _currently_capturing_graphs.count(child_capture_id),
        "capture_end() called before capture_begin().");
    _currently_capturing_graphs.erase(child_capture_id);
  }

```
- EN: Focus symbols: `get_capture_state`, `lock`, `TORCH_CHECK`, `count`, `capture_end`. This block implements callable behavior together with runtime validation. The checks reject unsupported states before delegating work to lower-level helpers.
- CN: 关注符号：`get_capture_state`, `lock`, `TORCH_CHECK`, `count`, `capture_end`。该代码块实现可调用行为并包含运行时校验。这些检查会在把工作下发给更底层辅助函数前拒绝不支持的状态。

### Lines 534-549
```cpp
  CUDAStream stream = conditional_node_streams_.top().current_stream();
  AT_CUDA_CHECK(cudaStreamEndCapture(stream.stream(), nullptr));
  conditional_node_streams_.pop();
  conditional_graph_capture_ids_.pop();

  c10::cuda::CUDACachingAllocator::endAllocateToPool(capture_dev_, mempool_id_);
  at::getHostAllocator(at::kCUDA)->end_allocate_to_pool(mempool_id_);
  if (conditional_graph_capture_ids_.empty()) {
    c10::cuda::CUDACachingAllocator::beginAllocateToPool(
        capture_dev_, mempool_id_, create_allocate_filter<cudaStream_t>());
    at::getHostAllocator(at::kCUDA)->begin_allocate_to_pool(mempool_id_, create_allocate_filter<c10::Stream>());
  } else {
    c10::cuda::CUDACachingAllocator::beginAllocateToPool(
        capture_dev_, mempool_id_, create_child_allocate_filter());
    auto filter = create_child_allocate_filter();
    at::getHostAllocator(at::kCUDA)->begin_allocate_to_pool(mempool_id_, [filter](c10::Stream stream) {
```
- EN: Focus symbols: `top`, `current_stream`, `AT_CUDA_CHECK`, `cudaStreamEndCapture`, `stream`. This block declares or implements callable behavior for this module. Inputs are translated into lower-level operations, state updates, or returned values here.
- CN: 关注符号：`top`, `current_stream`, `AT_CUDA_CHECK`, `cudaStreamEndCapture`, `stream`。该代码块声明或实现该模块的可调用行为。这里把输入转换为更底层操作、状态更新或返回值。

### Lines 550-565
```cpp
      return filter(CUDAStream(CUDAStream::UNCHECKED, stream));
    });
  }
  constexpr const char* rng_with_conditional_nodes_error =
      "RNG within data-dependent conditional nodes is not supported yet.";
  TORCH_CHECK(!rng_or_generators_changed, rng_with_conditional_nodes_error);

#else // !defined(USE_ROCM) && (defined(CUDA_VERSION) && CUDA_VERSION >= 12040)
  AT_ERROR(
      __func__,
      " CUDA Graphs conditional nodes are not supported for cuda version < 12.4");
#endif
}

std::function<bool(cudaStream_t)> CUDAGraph::create_child_allocate_filter() {
#if !defined(USE_ROCM) && (defined(CUDA_VERSION) && CUDA_VERSION >= 12040)
```
- EN: Focus symbols: `filter`, `CUDAStream`, `TORCH_CHECK`, `AT_ERROR`, `bool`. This block implements callable behavior together with runtime validation. The checks reject unsupported states before delegating work to lower-level helpers.
- CN: 关注符号：`filter`, `CUDAStream`, `TORCH_CHECK`, `AT_ERROR`, `bool`。该代码块实现可调用行为并包含运行时校验。这些检查会在把工作下发给更底层辅助函数前拒绝不支持的状态。

### Lines 566-579
```cpp
  return [&current_capture_id = conditional_graph_capture_ids_.top()](cudaStream_t stream) {
      auto capture_id_opt = c10::cuda::captureIdMayInitCtx(stream);
      return capture_id_opt.has_value() && capture_id_opt.value() == current_capture_id;
  };
#else // !defined(USE_ROCM) && (defined(CUDA_VERSION) && CUDA_VERSION >= 12040)
  AT_ERROR(
      __func__,
      " CUDA Graphs conditional nodes are not supported for cuda version < 12.4");
  return std::function<bool(cudaStream_t)>();
#endif
}


} // namespace at::cuda
```
- EN: Focus symbols: `at::cuda`, `top`, `captureIdMayInitCtx`, `has_value`, `value`. This block declares or implements callable behavior for this module. Inputs are translated into lower-level operations, state updates, or returned values here.
- CN: 关注符号：`at::cuda`, `top`, `captureIdMayInitCtx`, `has_value`, `value`。该代码块声明或实现该模块的可调用行为。这里把输入转换为更底层操作、状态更新或返回值。

## Key Concepts / 关键概念
- CUDA backend support / CUDA 后端支持
- C++ templates and specialization / C++ 模板与特化
- Conditional compilation / 条件编译
- Runtime validation / 运行时校验
- ATen/C10 internal layering / ATen/C10 内部分层

## Dependencies / 依赖关系
- Direct internal includes / 直接内部依赖: `ATen/core/CachingHostAllocator.h`, `ATen/cuda/CUDAContextLight.h`, `ATen/cuda/CUDAGeneratorImpl.h`, `ATen/cuda/CUDAGraph.h`, `ATen/cuda/CUDAGraphsUtils.cuh`, `ATen/cuda/Exceptions.h`, `ATen/cuda/MemPool.h`, `ATen/Functions.h`, `c10/cuda/CUDAAllocatorConfig.h`, `c10/cuda/CUDAFunctions.h`
- External/system includes / 外部或系统头: `cstddef`, `optional`
- Companion translation units / 配套实现或声明文件: `aten/src/ATen/cuda/CUDAGraph.h`, `aten/src/ATen/cuda/CUDAGraph.cu`
- Inferred semantic dependencies / 推断出的语义依赖: tensor abstractions / 张量抽象; C++ templates / C++ 模板; runtime validation / 运行时校验; namespace scoping / 命名空间作用域
