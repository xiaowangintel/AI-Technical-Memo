# dequantize.cuh — Code Analysis / 代码分析

## Source / 来源
- **File**: `csrc/quantization/gguf/dequantize.cuh`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Converts GGUF quantized blocks into dense CUDA tensors by combining per-format dequant kernels with a runtime dispatcher. / [CN] 通过按格式的反量化内核和运行时分发器，把 GGUF 量化块转换成稠密 CUDA 张量。

## Line-by-Line Analysis / 逐行分析
### Basic q4/q5/q8 dequantizers (L3-L78)
```cpp
static __device__ __forceinline__ void dequantize_q4_0(...) {
    const dfloat d = x[ib].d;
    const int vui = x[ib].qs[iqs];
    v.x = __int2half_rn(vui & 0xF);
    v.y = __int2half_rn(vui >> 4);
    v = __hsub2(v, __floats2half2_rn(8.0f, 8.0f));
    v = __hmul2(v, {d, d});
}
```
**EN:** These helpers decode the compact bit layout of the classic GGUF formats. Q4/Q5 rebuild packed nibbles (plus high bits for Q5), then apply delta/min logic to produce two half values at once.
**CN:** 这些辅助函数负责解码经典 GGUF 格式的紧凑位布局。Q4/Q5 需要重建打包的 4-bit 值（Q5 还要加高位），再结合 delta/min 生成两个 half 值。

### Generic block launcher for simple formats (L80-L99)
```cpp
template <int qk, int qr, dequantize_kernel_t dequantize_kernel, typename dst_t>
static __global__ void dequantize_block(...) {
    const int ib = i/qk;
    const int iqs = (i%qk)/qr;
    ...
    dequantize_kernel(vx, ib, iqs, v);
    y[iybs + iqs + 0]        = convert_from_half<dst_t>(v.x);
    y[iybs + iqs + y_offset] = convert_from_half<dst_t>(v.y);
}
```
**EN:** This generic kernel handles formats where each packed element expands into two outputs. It computes the source block index, asks a format-specific helper to decode the pair, then writes the converted result to the dense output row.
**CN:** 这个通用内核处理“一个打包单元展开成两个输出”的格式。它先算出源块索引，再调用格式专用解码函数，最后把结果写入稠密输出行。

### K-quant dequantization kernels (L101-L256)
```cpp
template<typename dst_t>
static __global__ void dequantize_block_q4_K(...) {
    ...
    get_scale_min_k4(is + 0, x[i].scales, sc, m);
    const half d1 = __hmul(dall, __int2half_rn(sc));
    const half m1 = __hmul(dmin, __int2half_rn(m));
    ...
    y[l + 0] = convert_from_half<dst_t>(__hsub(__hmul(d1, ...), m1));
}
```
**EN:** `q2_K` through `q6_K` are not simple nibble unpackers: they reconstruct block-local scales, mins, and high-bit masks. The kernels are specialized because each K-quant family stores metadata differently.
**CN:** `q2_K` 到 `q6_K` 不是简单的 nibble 解包器，它们还要重建块内 scale、min 和高位掩码。由于每个 K 系列量化的元数据编码方式不同，所以这里必须写成专门内核。

### IQ-family lookup-table reconstruction (L258-L435)
```cpp
template<typename dst_t>
static __global__ void dequantize_block_iq2_xxs(...) {
    const uint8_t  * grid = (const uint8_t *)(iq2xxs_grid + aux8[il]);
    const uint8_t signs = ksigns_iq2xs[(aux32 >> 7*il) & 127];
    for (int j = 0; j < 8; ++j) y[j] = d * grid[j] * (signs & kmask_iq2xs[j] ? -1.f : 1.f);
}
```
**EN:** The IQ formats rely on precomputed grids and sign tables from `ggml-common.h`. Instead of unpacking plain linear quant values, they decode table indices, sign bits, and small scale fields to reconstruct approximate floating-point values.
**CN:** IQ 格式依赖 `ggml-common.h` 中的预计算网格和符号表。它们不是直接解包线性量化值，而是通过表索引、符号位和小尺度字段来重建近似浮点值。

### Launch helpers and runtime type dispatch (L437-L571)
```cpp
template<typename dst_t>
static to_cuda_ggml_t<dst_t> ggml_get_to_cuda(int64_t type) {
    switch (type) {
        case 2:  return dequantize_block_cuda<QK4_0, QR4_0, dequantize_q4_0>;
        ...
        case 29: return dequantize_row_iq1_m_cuda;
        default: return nullptr;
    }
}
```
**EN:** The bottom of the file converts GGUF numeric type IDs into callable CUDA entry points. This is the bridge used by `gguf_kernel.cu` to dequantize arbitrary formats without hard-coding every kernel at the call site.
**CN:** 文件末尾把 GGUF 的数值类型 ID 映射为可调用的 CUDA 入口函数。这是 `gguf_kernel.cu` 的桥接层，使调用方无需在现场写死所有反量化内核。

## Key Concepts / 关键概念
- Pairwise half reconstruction for classic formats / 经典格式的双 half 重建
- Specialized K-quant metadata decoding / K 系列量化元数据解码
- IQ lookup-grid reconstruction / IQ 查表重建
- Runtime mapping from GGUF type ID to CUDA function / GGUF 类型 ID 到 CUDA 函数的运行时映射

## Dependencies / 依赖关系
- `ggml-common.h`: quant block structs, grid tables, sign tables, `convert_from_half`
- CUDA half math: `__hmul`, `__hsub`, `__half2float`
- `gguf_kernel.cu`: calls `ggml_get_to_cuda` for public dequantization
