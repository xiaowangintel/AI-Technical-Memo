# vecdotq.cuh — Code Analysis / 代码分析

## Source / 来源
- **File**: `csrc/quantization/gguf/vecdotq.cuh`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Implements the low-level quantized dot-product math and shared-memory tile helpers used by GGUF GEMV, GEMM, and MoE kernels. / [CN] 实现 GGUF 的 GEMV、GEMM 和 MoE 内核所依赖的底层量化点积数学与共享内存 tile 辅助函数。

## Line-by-Line Analysis / 逐行分析
### Integer unpack helpers and VDR definitions (L3-L44)
```cpp
static __device__ __forceinline__ int get_int_from_int8_aligned(const int8_t * x8, const int & i32) {
    return *((const int *) (x8 + sizeof(int) * i32));
}
#define VDR_Q4_0_Q8_1_MMVQ 2
#define VDR_Q4_0_Q8_1_MMQ  4
```
**EN:** The file starts by turning packed byte streams into 32-bit chunks that can be consumed by SIMD integer dot-product instructions. The VDR macros describe how much contiguous work one thread handles in each kernel family.
**CN:** 文件开头先把打包字节流变成 32 位整块，以便交给 SIMD 整数点积指令处理。VDR 宏用于描述每个线程在不同内核家族中连续处理多少数据。

### Core DP4A-based dot-product implementations (L46-L198)
```cpp
template <int vdr> static __device__ __forceinline__ float vec_dot_q4_0_q8_1_impl(...) {
    ...
    sumi = __dp4a(vi0, u[2*i+0], sumi);
    sumi = __dp4a(vi1, u[2*i+1], sumi);
    return d4 * (sumi * ds8f.x - (8*vdr/QI4_0) * ds8f.y);
}
```
**EN:** The classic q4/q5/q8 paths convert packed quants into `int` lanes and rely on `__dp4a` to do four-way integer multiply-accumulate. The returned float is not just a raw dot product; it also applies scale, min, or sum corrections required by the quant format.
**CN:** 经典 q4/q5/q8 路径会先把打包量化值展开成整型 lane，再依赖 `__dp4a` 做四路整数乘加。返回值也不只是原始点积，还会补上该量化格式所需的 scale、min 或 sum 修正。

### K-quant inner-product logic (L203-L392)
```cpp
static __device__ __forceinline__ float vec_dot_q2_K_q8_1_impl_mmq(...) {
    ...
    sumi_d_sc = __dp4a(v[i], u[i], sumi_d_sc);
    sumi_m    = __dp4a(m,    u[i], sumi_m);
    ...
    return d8 * (dm2f.x*sumi_d - dm2f.y*sumi_m);
}
```
**EN:** K-quants need extra arithmetic because their blocks carry block-local scales and mins. The implementations split the dot product into a data term and a min/offset correction term, then recombine them with the super-block metadata.
**CN:** K 系列量化需要更多算术步骤，因为它们的块里还存有局部 scale 和 min。实现上通常把点积分成“数据项”和“min/偏移修正项”两部分，再结合 super-block 元数据合并。

### MMVQ/MMQ adapters and tile loaders (L493-L1406)
```cpp
static __device__ __forceinline__ float vec_dot_q4_0_q8_1(...) { ... }

template <int mmq_y> static __device__ __forceinline__ void allocate_tiles_q4_0(...) {
    __shared__ int tile_x_qs[...];
    __shared__ float tile_x_d[...];
}

template <int mmq_y, int nwarps, bool need_check>
static __device__ __forceinline__ void load_tiles_q4_0(...) { ... }
```
**EN:** The middle of the file adapts the pure dot-product math to two execution styles: vector kernels (`mmvq`) and tiled matrix kernels (`mmq`/`moe`). `allocate_tiles_*` reserves the shared-memory layout, while `load_tiles_*` repacks quant blocks into a form that the matrix kernel can reuse efficiently.
**CN:** 文件中段把纯点积数学适配到两种执行风格：向量内核（`mmvq`）和分块矩阵内核（`mmq`/`moe`）。`allocate_tiles_*` 负责定义共享内存布局，`load_tiles_*` 则把量化块重排成矩阵内核更容易复用的形式。

### IQ-format dot products via lookup tables (L1531-L1812)
```cpp
static __device__ __forceinline__ float vec_dot_iq2_xxs_q8_1(...) {
    const uint8_t * grid = (const uint8_t *)(iq2xxs_grid + aux8[l]);
    const uint8_t signs = ksigns_iq2xs[aux32 & 127];
    ...
}
```
**EN:** The IQ paths are the most table-driven part of the file. Instead of extracting plain 2/3/4-bit values, they decode table indices, sign masks, and tiny scale fields, then perform the dot product against Q8_1 blocks.
**CN:** IQ 路径是本文件里最依赖查表的部分。它们不是直接提取普通的 2/3/4-bit 值，而是先解出表索引、符号掩码和小 scale 字段，再与 Q8_1 块做点积。

## Key Concepts / 关键概念
- `__dp4a`-based integer dot products / 基于 `__dp4a` 的整数点积
- Shared low-level backend for GEMV/GEMM/MoE / GEMV/GEMM/MoE 共用的底层后端
- Tile allocation and repacking for matrix kernels / 面向矩阵内核的 tile 分配与重排
- Table-driven IQ decoding / 基于查表的 IQ 解码

## Dependencies / 依赖关系
- `ggml-common.h`: block structs, tables, helper typedefs, `__dp4a` shims
- Used by `mmvq.cuh`, `mmq.cuh`, `moe.cuh`, and `moe_vec.cuh`
- CUDA/ROCm integer SIMD and half arithmetic intrinsics
