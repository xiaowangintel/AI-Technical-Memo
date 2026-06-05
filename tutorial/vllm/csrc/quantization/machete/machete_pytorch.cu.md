# machete_pytorch.cu — Code Analysis / 代码分析

## Source / 来源
- **File**: `csrc/quantization/machete/machete_pytorch.cu`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Exposes Machete prepacking, schedule querying, and matrix multiplication as PyTorch custom operators. / [CN] 将 Machete 的预打包、调度查询和矩阵乘法能力封装为 PyTorch 自定义算子。

## Line-by-Line Analysis / 逐行分析
### [1-10] Launcher includes and namespace setup
```cpp
#include "machete_mm_launcher.cuh"
#include "machete_prepack_launcher.cuh"
#include "core/scalar_type.hpp"
#include "core/registration.h"

namespace machete {
using namespace vllm;
```
**EN:** This file is intentionally thin: all heavy lifting lives in the launcher and kernel headers. It mainly translates PyTorch-facing types into the internal dispatch structures.
**CN:** 该文件本身很薄，真正的实现都在 launcher 和 kernel 头文件里。这里的主要职责是把 PyTorch 接口参数翻译成内部调度结构。

### [11-28] `supported_schedules` wrapper
```cpp
std::vector<std::string> supported_schedules(
    at::ScalarType a_type, int64_t b_type_id,
    std::optional<at::ScalarType> maybe_group_scales_type,
    ...,
    std::optional<at::ScalarType> maybe_out_type) {
  ScalarType const b_type = ScalarType::from_id(b_type_id);
  return supported_schedules_dispatch({
      .a_type = a_type,
      .b_type = b_type,
      ...
  });
}
```
**EN:** The API receives a serialized `b_type_id` from Python, converts it into vLLM’s internal `ScalarType`, and forwards all optional quantization metadata to the schedule-discovery dispatcher. This lets higher layers query which CUTLASS schedule combinations are legal before launching a kernel.
**CN:** 该接口从 Python 接收序列化的 `b_type_id`，将其转换为 vLLM 内部 `ScalarType`，并把所有可选量化元数据转发给调度查询分发器。这样上层就能在真正发射内核之前先知道哪些 CUTLASS 调度组合是可用的。

### [30-50] `mm` entry point for runtime execution
```cpp
torch::Tensor mm(torch::Tensor const& A, torch::Tensor const& B,
                 int64_t b_type_id,
                 std::optional<at::ScalarType> const& maybe_out_type,
                 ...,
                 std::optional<std::string> maybe_schedule) {
  ScalarType const b_type = ScalarType::from_id(b_type_id);
  return mm_dispatch({.A = A,
                      .B = B,
                      .b_type = b_type,
                      ...,
                      .maybe_schedule = maybe_schedule});
}
```
**EN:** `mm` packages the dense input tensor, the prepacked/quantized weight tensor, and all optional scale tensors into `MMArgs`, then delegates the real kernel selection to `mm_dispatch`. The wrapper does no computation itself; it is a stable ABI boundary between Python and templated CUDA code.
**CN:** `mm` 会把稠密输入张量、预打包/量化权重张量以及各种可选缩放张量组装成 `MMArgs`，再交给 `mm_dispatch` 做真正的内核选择。这个包装函数本身不计算，只充当 Python 与模板化 CUDA 代码之间的稳定 ABI 边界。

### [52-61] `prepack_B` entry point
```cpp
torch::Tensor prepack_B(
    torch::Tensor const& B, at::ScalarType const& a_type, int64_t b_type_id,
    std::optional<at::ScalarType> const& maybe_group_scales_type) {
  ScalarType const b_type = ScalarType::from_id(b_type_id);
  return prepack_B_dispatch(
      {.B = B,
       .a_type = a_type,
       .b_type = b_type,
       .maybe_group_scales_type = maybe_group_scales_type});
}
```
**EN:** Machete requires a special weight layout for fast SM90 tensor-core execution. `prepack_B` is the operator that converts a regular packed weight tensor into that runtime layout ahead of inference.
**CN:** Machete 为了在 SM90 Tensor Core 上高效执行，需要一种特殊的权重布局。`prepack_B` 就是在推理前把普通打包权重转换成该运行时布局的操作符。

### [63-71] Register custom operators into Torch
```cpp
TORCH_LIBRARY_IMPL_EXPAND(TORCH_EXTENSION_NAME, CUDA, m) {
  m.impl("machete_prepack_B", &prepack_B);
  m.impl("machete_mm", &mm);
}

TORCH_LIBRARY_IMPL(TORCH_EXTENSION_NAME, CatchAll, m) {
  m.impl("machete_supported_schedules", &supported_schedules);
}
```
**EN:** CUDA-backed operators are registered in the CUDA dispatch key, while `supported_schedules` uses `CatchAll` because it has no tensor inputs and therefore cannot be routed by device. This split is common for mixed operator/query APIs.
**CN:** 真正依赖 CUDA 张量的算子注册在 CUDA dispatch key 下，而 `supported_schedules` 因为没有张量输入，无法按设备分发，所以放在 `CatchAll`。这种拆分是“执行 + 查询”混合接口中的常见做法。

## Key Concepts / 关键概念
- **EN:** This file is an API adapter, not a compute implementation.
- **CN:** 该文件是 API 适配层，而不是计算实现层。
- **EN:** `ScalarType::from_id` bridges Python-friendly integer identifiers and internal quantized type metadata.
- **CN:** `ScalarType::from_id` 用来桥接 Python 友好的整数类型编号与内部量化类型元数据。

## Dependencies / 依赖关系
- **EN:** Depends on `machete_mm_launcher.cuh` and `machete_prepack_launcher.cuh` for actual dispatch logic.
- **CN:** 依赖 `machete_mm_launcher.cuh` 和 `machete_prepack_launcher.cuh` 提供真实的分发逻辑。
- **EN:** Uses Torch registration macros from `core/registration.h`.
- **CN:** 使用 `core/registration.h` 中的 Torch 注册宏。
