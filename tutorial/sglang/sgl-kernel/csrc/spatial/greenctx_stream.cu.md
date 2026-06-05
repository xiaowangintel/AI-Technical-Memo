# greenctx_stream.cu — Code Analysis / 代码分析

## Source / 来源
- **File**: `sgl-kernel/csrc/spatial/greenctx_stream.cu`
- **Repository**: `sgl-project/sglang`
- **Purpose**: Contains low-level kernel code and support utilities for the sgl-kernel backend. This source file primarily implements kernels, host-side wrappers, bindings, or launch logic. / 包含 sgl-kernel 后端的底层内核代码和配套工具。 该源文件主要实现内核、主机侧封装、绑定或启动逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-18: Templates, aliases, and constants
```cpp
// Documentation: https://docs.nvidia.com/cuda/cuda-driver-api/group__CUDA__GREEN__CONTEXTS.html
#include <torch/all.h>

#include <cstdlib>

#include "cuda_utils.h"
#include "greenctx_stream.h"

static int CUDA_DRIVER_VERSION;

using PFN_cuGreenCtxStreamCreate = CUresult(CUDAAPI*)(CUstream*, CUgreenCtx, unsigned int, int);

auto probe_cuGreenCtxStreamCreate() -> PFN_cuGreenCtxStreamCreate {
  static PFN_cuGreenCtxStreamCreate pfn = nullptr;
  CUDA_DRV(cuGetProcAddress("cuGreenCtxStreamCreate", reinterpret_cast<void**>(&pfn), CUDA_DRIVER_VERSION, 0, nullptr));
  return pfn;
}
```
**EN:** This section defines `probe_cuGreenCtxStreamCreate`, `CUresult`, `CUDA_DRV`, giving the file reusable compile-time abstractions, aliases, and specialization points.
**CN:** 本段定义了`probe_cuGreenCtxStreamCreate`、`CUresult`、`CUDA_DRV`等内容，为文件提供可复用的编译期抽象、类型别名和特化入口。

### Lines 19-35: Local implementation details
```cpp
static std::vector<int64_t> create_greenctx_stream_fallback(CUgreenCtx gctx[2]) {
  CUstream streamA, streamB;
  CUcontext ctx;

  CUDA_DRV(cuCtxFromGreenCtx(&ctx, gctx[0]));
  CUDA_DRV(cuCtxPushCurrent(ctx));
  CUDA_DRV(cuStreamCreate(&streamA, CU_STREAM_NON_BLOCKING));
  CUDA_DRV(cuCtxPopCurrent(nullptr));

  CUDA_DRV(cuCtxFromGreenCtx(&ctx, gctx[1]));
  CUDA_DRV(cuCtxPushCurrent(ctx));
  CUDA_DRV(cuStreamCreate(&streamB, CU_STREAM_NON_BLOCKING));
  CUDA_DRV(cuCtxPopCurrent(nullptr));

  return {(int64_t)streamA, (int64_t)streamB};
}
```
**EN:** This section fills in the local implementation details around `create_greenctx_stream_fallback`, `CUDA_DRV`, completing the behavior required by the file.
**CN:** 本段补充了`create_greenctx_stream_fallback`、`CUDA_DRV`周边的局部实现细节，以完成该文件所需的具体行为。

### Lines 36-54: Runtime integration and dispatch
```cpp
inline void destroy_green_context(CUgreenCtx gctx) {
  if (!gctx) return;
  CUDA_DRV(cuGreenCtxDestroy(gctx));
}

static std::vector<int64_t> create_greenctx_stream_direct_dynamic(CUgreenCtx gctx[2]) {
  // This symbol is introduced in CUDA 12.5
  const static auto pfn = probe_cuGreenCtxStreamCreate();
  if (!pfn) {
    TORCH_WARN("cuGreenCtxStreamCreate(cuda>=12.5) is not available, using fallback");
    return create_greenctx_stream_fallback(gctx);
  }

  CUstream streamA, streamB;
  CUDA_DRV(pfn(&streamA, gctx[0], CU_STREAM_NON_BLOCKING, 0));
  CUDA_DRV(pfn(&streamB, gctx[1], CU_STREAM_NON_BLOCKING, 0));

  return {(int64_t)streamA, (int64_t)streamB};
}
```
**EN:** This section uses `destroy_green_context`, `create_greenctx_stream_direct_dynamic`, `CUDA_DRV` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`destroy_green_context`、`create_greenctx_stream_direct_dynamic`、`CUDA_DRV`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 55-71: Runtime integration and dispatch
```cpp

std::vector<int64_t> create_greenctx_stream_by_value(int64_t smA, int64_t smB, int64_t device) {
  CUDA_DRV(cuDriverGetVersion(&CUDA_DRIVER_VERSION));

  CUgreenCtx gctx[3];
  CUdevResourceDesc desc[3];
  CUdevResource input;
  CUdevResource resources[4];

  TORCH_CHECK(smA > 0 && smB > 0, "SM counts must be positive");

  CUDA_DRV(cuDeviceGetDevResource((CUdevice)device, &input, CU_DEV_RESOURCE_TYPE_SM));

  const unsigned minCount = static_cast<unsigned>(smA + smB);
  const unsigned minCountA = static_cast<unsigned>(smA);
  TORCH_CHECK(minCount <= input.sm.smCount, "Not enough SMs available for the requested configuration");
```
**EN:** This section uses `create_greenctx_stream_by_value`, `CUDA_DRV`, `TORCH_CHECK` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`create_greenctx_stream_by_value`、`CUDA_DRV`、`TORCH_CHECK`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 72-90: Local implementation details
```cpp
  unsigned nbGroups = 1;
  CUDA_DRV(cuDevSmResourceSplitByCount(&resources[2], &nbGroups, &input, &resources[3], 0, minCount));
  CUDA_DRV(cuDevResourceGenerateDesc(&desc[2], &resources[2], 1));
  CUDA_DRV(cuGreenCtxCreate(&gctx[2], desc[2], (CUdevice)device, CU_GREEN_CTX_DEFAULT_STREAM));
  CUDA_DRV(cuGreenCtxGetDevResource(gctx[2], &input, CU_DEV_RESOURCE_TYPE_SM));
  nbGroups = 1;
  CUDA_DRV(cuDevSmResourceSplitByCount(&resources[0], &nbGroups, &input, &resources[1], 0, minCountA));
  CUDA_DRV(cuDevResourceGenerateDesc(&desc[0], &resources[0], 1));
  CUDA_DRV(cuGreenCtxCreate(&gctx[0], desc[0], (CUdevice)device, CU_GREEN_CTX_DEFAULT_STREAM));
  CUDA_DRV(cuDevResourceGenerateDesc(&desc[1], &resources[1], 1));
  CUDA_DRV(cuGreenCtxCreate(&gctx[1], desc[1], (CUdevice)device, CU_GREEN_CTX_DEFAULT_STREAM));

  const int smCountA = resources[0].sm.smCount;
  const int smCountB = resources[1].sm.smCount;

  std::vector<int64_t> streams = create_greenctx_stream_direct_dynamic(gctx);

  destroy_green_context(gctx[2]);
```
**EN:** This section fills in the local implementation details around `CUDA_DRV`, `create_greenctx_stream_direct_dynamic`, `destroy_green_context`, completing the behavior required by the file.
**CN:** 本段补充了`CUDA_DRV`、`create_greenctx_stream_direct_dynamic`、`destroy_green_context`周边的局部实现细节，以完成该文件所需的具体行为。

### Lines 91-98: Local implementation details
```cpp
  std::vector<int64_t> vec = {
      streams[0],  // streamA
      streams[1],  // streamB
      (int64_t)smCountA,
      (int64_t)smCountB};

  return vec;
}
```
**EN:** This section fills in the local implementation details around the surrounding logic, completing the behavior required by the file.
**CN:** 本段补充了相关逻辑周边的局部实现细节，以完成该文件所需的具体行为。

## Key Concepts / 关键概念
- **PyTorch extension / PyTorch 扩展**: Bridges low-level kernels with PyTorch tensor/runtime abstractions. / 把底层内核与 PyTorch 张量/运行时抽象连接起来。
- **CUDA runtime / CUDA 运行时**: Uses CUDA APIs, streams, or intrinsics for NVIDIA GPUs. / 使用 CUDA API、stream 或 intrinsic 支持 NVIDIA GPU。

## Dependencies / 依赖关系
- **Internal headers / 内部头文件**: `cuda_utils.h`, `greenctx_stream.h`
- **External headers / 外部头文件**: `torch/all.h`, `cstdlib`
- **Path context / 路径上下文**: spatial / greenctx_stream.cu
