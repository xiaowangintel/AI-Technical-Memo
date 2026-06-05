# minimax_reduce_rms_kernel.h — Code Analysis / 代码分析

## Source / 来源
- **File**: `csrc/minimax_reduce_rms_kernel.h`
- **Repository**: `vllm-project/vllm`
- **Purpose**: [EN] Declares the parameter structure and type traits used by the MiniMax all-reduce RMSNorm CUDA kernels. / [CN] 声明 MiniMax all-reduce RMSNorm CUDA 内核所使用的参数结构和类型特征。

## Line-by-Line Analysis / 逐行分析

### Core includes and namespace / 核心头文件与命名空间
```cpp
#include <cuda_bf16.h>
#include <cuda_fp16.h>

#include <torch/types.h>

namespace vllm {
namespace tensorrt_llm {
```
**EN:** The header only needs CUDA scalar types and PyTorch scalar metadata. It places the API under `vllm::tensorrt_llm`, signaling that these kernels are tied to the TensorRT-LLM MiniMax path.
**CN:** 该头文件只依赖 CUDA 标量类型和 PyTorch 的标量元信息，并将接口放在 `vllm::tensorrt_llm` 命名空间下，表明这些内核面向 TensorRT-LLM 的 MiniMax 路径。

### Access-width trait / 访问宽度特征
```cpp
template <typename DType>
struct ElemsPerAccess;

template <>
struct ElemsPerAccess<half> {
  static constexpr int value = 8;
  using vec_type = float4;
};
...
template <typename DType>
static constexpr int kElemsPerAccess = ElemsPerAccess<DType>::value;
```
**EN:** `ElemsPerAccess` encodes how many elements one vectorized memory transaction should handle for each dtype. Half and BF16 use 8 elements per access, while FP32 uses 4, but all map to `float4`-sized vector moves.
**CN:** `ElemsPerAccess` 描述每种数据类型在一次向量化访存中应该处理多少元素。half 和 BF16 每次访问 8 个元素，FP32 每次访问 4 个元素，但它们都映射到 `float4` 大小的向量移动。

### Kernel parameter bundle / 内核参数打包结构
```cpp
struct MiniMaxReduceRMSParams {
  int nranks{};
  int rank{};
  at::ScalarType dtype{at::ScalarType::Undefined};
  int size_q{};
  int hidden_dim{};
  int size_k{};
  int hidden_dim_k{};
  int stride_q{};
  int stride_k{};
  int stride_q_out{};
  int stride_k_out{};
  void** workspace{};
  void* allreduce_in{};
  void* rms_norm_out{};
  void* rms_gamma{};
  void* allreduce_in_k{};
  void* rms_norm_out_k{};
  void* rms_gamma_k{};
  float rms_eps{};
  cudaStream_t stream{};
};
```
**EN:** This struct is the complete launch contract. It carries tensor-parallel metadata (`rank`, `nranks`), Q/K sizes and strides, workspace pointers used for cross-rank synchronization, gamma weights, output buffers, epsilon, and the CUDA stream.
**CN:** 这个结构体就是完整的启动契约。它包含张量并行元数据（`rank`、`nranks`）、Q/K 的尺寸与步长、跨 rank 同步所需的 workspace 指针、gamma 权重、输出缓冲区、epsilon 以及 CUDA stream。

### Public entry point / 对外入口
```cpp
void minimax_reduce_rms_op(MiniMaxReduceRMSParams const& params);
```
**EN:** The `.cu` file implements this single entry point and performs all dtype/rank/kernel-shape dispatch internally. The header therefore stays lightweight and stable for callers.
**CN:** `.cu` 文件实现了这个统一入口，并在内部完成数据类型、rank 数量和内核形状的分发，因此该头文件对调用方来说保持轻量且稳定。

## Key Concepts / 关键概念
- **EN:** The header separates policy-free data description from kernel implementation details.  
  **CN:** 该头文件把“纯数据描述”与具体内核实现细节分离开来。
- **EN:** `ElemsPerAccess` is important because the implementation relies heavily on `float4`-sized vector loads/stores.  
  **CN:** `ElemsPerAccess` 很关键，因为实现严重依赖 `float4` 尺寸的向量化读写。
- **EN:** The presence of both Q and K fields shows that the runtime can normalize either Q alone or fused Q/K slices from a wider QKV tensor.  
  **CN:** 参数里同时存在 Q 和 K 字段，说明运行时既支持只处理 Q，也支持从更宽的 QKV 张量里融合处理 Q/K。

## Dependencies / 依赖关系
- **EN:** Consumed by `csrc/minimax_reduce_rms_kernel.cu`, which interprets every field in `MiniMaxReduceRMSParams`.  
  **CN:** 由 `csrc/minimax_reduce_rms_kernel.cu` 使用，并在那里解释 `MiniMaxReduceRMSParams` 的全部字段。
- **EN:** Depends on CUDA half/BF16 definitions and PyTorch `at::ScalarType`.  
  **CN:** 依赖 CUDA 的 half/BF16 定义以及 PyTorch 的 `at::ScalarType`。
- **EN:** Serves as the bridge between higher-level torch extension wrappers and the low-level launchers.  
  **CN:** 它是高层 torch 扩展包装函数与底层 CUDA launcher 之间的桥梁。
