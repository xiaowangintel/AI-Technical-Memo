# ggml-common.h — Code Analysis / 代码分析

## Source / 来源
- **File**: `csrc/quantization/gguf/ggml-common.h`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Defines the GGUF quant block layouts, lookup tables, function pointer types, and low-level helpers shared by all CUDA quantization kernels. / [CN] 定义所有 CUDA 量化内核共享的 GGUF 量化块布局、查找表、函数指针类型和底层辅助函数。

## Line-by-Line Analysis / 逐行分析
### Global constants and execution parameters (L2-L10)
```cpp
#define QK_K 256
#define K_QUANTS_PER_ITERATION 2
#define WARP_SIZE_GGUF 32
#define CUDA_DEQUANTIZE_BLOCK_SIZE 256
#define GGML_CUDA_MMV_Y 1
```
**EN:** These macros define the shared geometry assumptions used across dequantization, GEMV, GEMM, and MoE kernels. `QK_K=256` is especially important because the K-quant formats are built around 256-value super-blocks.
**CN:** 这些宏定义了反量化、GEMV、GEMM 和 MoE 内核共用的几何假设。尤其是 `QK_K=256`，因为 K 系列量化都是围绕 256 个值的 super-block 设计的。

### Standard GGUF block structs (L17-L65)
```cpp
typedef struct { half d; uint8_t qs[QK4_0 / 2]; } block_q4_0;
typedef struct { half2 dm; uint8_t qs[QK4_1 / 2]; } block_q4_1;
typedef struct { half d; uint8_t qh[4]; uint8_t qs[QK5_0 / 2]; } block_q5_0;
typedef struct { half2 ds; int8_t qs[QK8_0]; } block_q8_1;
```
**EN:** This section defines the memory ABI for the classic q4/q5/q8 GGUF formats. The comments show exactly which metadata fields exist: pure scale (`d`), scale+min (`dm`), high-bit side channels (`qh`), or activation sum (`ds.y`).
**CN:** 这一段定义了经典 q4/q5/q8 GGUF 格式的内存 ABI。注释明确说明了每种格式具备哪些元数据：纯 scale（`d`）、scale+min（`dm`）、高位旁路（`qh`）或激活求和（`ds.y`）。

### K-quant and IQ block families (L67-L191)
```cpp
typedef struct { uint8_t scales[QK_K/16]; uint8_t qs[QK_K/4]; half2 dm; } block_q2_K;
typedef struct { uint8_t hmask[QK_K/8]; uint8_t qs[QK_K/4]; uint8_t scales[K_SCALE_SIZE]; half d; } block_q3_K;
typedef struct { half d; uint16_t qs[QK_K/8]; } block_iq2_xxs;
typedef struct { half d; uint8_t qs[QK_K/4]; uint8_t qh[QK_K/32]; uint8_t signs[QK_K/8]; uint8_t scales[IQ3S_N_SCALE]; } block_iq3_s;
```
**EN:** These structs show why later kernels become specialized: K-quants and IQ formats encode scale/min/sign information in very different ways. This header is the canonical contract that all decode and dot-product code relies on.
**CN:** 这些结构体解释了后续内核为何必须专门化：K 系列量化和 IQ 格式对 scale/min/sign 的编码方式差异很大。这个头文件就是所有解码和点积代码依赖的统一契约。

### Lookup-table resources (L193-L1060, excerpt)
```cpp
static const __device__ uint64_t iq2xxs_grid[256] = { ... };
static const __device__ uint64_t iq2xs_grid[512] = { ... };
static const __device__ uint64_t iq1s_grid_gpu[2048] = { ... };
static const __device__ uint8_t kmask_iq2xs[8] = {1, 2, 4, 8, 16, 32, 64, 128};
static const __device__ int8_t kvalues_iq4nl[16] = {-127, -104, ... , 113};
```
**EN:** Most of the file is device-resident lookup data for IQ formats. These tables trade memory for speed: the kernels can decode small table indices instead of reconstructing non-linear quant values procedurally.
**CN:** 文件的大部分内容都是给 IQ 格式使用的设备端查表数据。它通过“用内存换速度”的方式，让内核直接根据小索引查值，而不是运行时按规则重建非线性量化值。

### Function-pointer ABI and type conversion helpers (L1063-L1096)
```cpp
typedef void (*dequantize_kernel_t)(const void * vx, const int ib, const int iqs, dfloat2 & v);
typedef float (*vec_dot_q_cuda_t)(const void * __restrict__ vbq, const block_q8_1 * __restrict__ bq8_1, const int & iqs);
...
template<> __device__ __forceinline__ float convert_from_half<float>(half val) {
    return __half2float(val);
}
```
**EN:** These typedefs are what allow higher-level templates to pass kernels and dot-product routines as arguments. `convert_from_half` also centralizes output conversion for `half`, `float`, and `BFloat16` targets.
**CN:** 这些 typedef 让更高层模板能够把解码函数和点积函数当参数传递。`convert_from_half` 则统一了 `half`、`float` 和 `BFloat16` 的输出转换逻辑。

### ROCm compatibility shims (L1098-L1150)
```cpp
static __device__ __forceinline__ int __dp4a(const int a, const int b, int c) {
#if __has_builtin(__builtin_amdgcn_sdot4)
    c = __builtin_amdgcn_sdot4(a, b, c, false);
#else
    ...
#endif
}
```
**EN:** CUDA-specific integer vector intrinsics are reimplemented here for ROCm when needed. That keeps the rest of the quant kernels mostly portable between NVIDIA and AMD backends.
**CN:** 这里为 ROCm 补上了 CUDA 风格的整数向量 intrinsic。这样一来，其余量化内核的大部分逻辑就能在 NVIDIA 和 AMD 后端之间共享。

## Key Concepts / 关键概念
- Shared ABI for GGUF quant block layouts / GGUF 量化块布局的共享 ABI
- Device lookup tables for IQ formats / IQ 格式的设备端查表
- Function-pointer typedefs enabling templated dispatch / 支持模板分发的函数指针类型
- ROCm compatibility layer for integer SIMD helpers / 面向整数 SIMD 辅助函数的 ROCm 兼容层

## Dependencies / 依赖关系
- CUDA half types and intrinsics / CUDA half 类型与 intrinsic
- `c10::BFloat16` for mixed output conversion
- Used by `dequantize.cuh`, `vecdotq.cuh`, `mmvq.cuh`, `mmq.cuh`, `moe.cuh`, and `moe_vec.cuh`
