# qdq_util.cuh — Code Analysis / 代码分析

## Source / 来源
- **File**: `csrc/quantization/gptq/qdq_util.cuh`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Provides tiny device helpers for GPTQ quantize/dequantize code, including bit-casts between half types and integer storage plus basic packed-bit extraction. / [CN] 为 GPTQ 量化/反量化代码提供小型设备端辅助函数，包括 half 类型与整数存储之间的位重解释，以及基础的打包位提取操作。

## Line-by-Line Analysis / 逐行分析

### Bit-cast unions
```cpp
union half2_uint32 {
  uint32_t as_uint32;
  half2 as_half2;
  __device__ half2_uint32(uint32_t val) : as_uint32(val) {}
  __device__ half2_uint32(half2 val) : as_half2(val) {}
};

union half_uint16 {
  uint16_t as_uint16;
  half as_half;
  __device__ half_uint16(uint16_t val) : as_uint16(val) {}
  __device__ half_uint16(half val) : as_half(val) {}
};
```
**EN:** The two unions make it cheap to reinterpret `half2` as `uint32_t` and `half` as `uint16_t` inside device code. GPTQ kernels use this when moving between arithmetic values and packed storage representations.
**CN:** 这两个 union 让设备代码可以低成本地把 `half2` 重解释为 `uint32_t`，以及把 `half` 重解释为 `uint16_t`。GPTQ 内核在算术值与打包存储表示之间切换时会用到它们。

### Scale and dequant helpers
```cpp
__forceinline__ __device__ half dq_scale(const int qs, const half max_scale) {
  int qs_i = qs + 1;
  half qs_h = __int2half_rn(qs_i * qs_i);
  qs_h = __hmul(qs_h, max_scale);
  return qs_h;
}

__forceinline__ __device__ half dq(const int q, const int qzero,
                                   const half scale) {
  return __hmul(__int2half_rn(q - qzero), scale);
}

__forceinline__ __device__ half dq_ns(const int q, const int qzero) {
  // return __hsub(__int2half_rn(q), __int2half_rn(qzero));
  return __int2half_rn(q - qzero);
}
```
**EN:** `dq_scale` reconstructs the effective scale from a quantized scale index, while `dq` and `dq_ns` dequantize integer codes with and without applying the scale. These are the basic arithmetic building blocks behind GPTQ weight reconstruction.
**CN:** `dq_scale` 从量化后的 scale 索引重建有效 scale，`dq` 与 `dq_ns` 则分别在“带 scale”和“不带 scale”的情况下把整数码字反量化。这些函数是 GPTQ 权重重建的基础算术组件。

### Packed-bit extraction
```cpp
__forceinline__ __device__ int exb(const uint32_t q, const int shift,
                                   const int mask) {
  return (int)((q >> shift) & mask);
}

__forceinline__ __device__ int exb(const uint32_t q1, const uint32_t q0,
                                   const int shift, const int mask) {
  return (int)(__funnelshift_rc(q0, q1, shift) & mask);
}
```
**EN:** The two `exb` overloads extract a bitfield either from one 32-bit word or from a logical field crossing two adjacent words via `__funnelshift_rc`. They are used when quantized values are not naturally aligned to a single machine word.
**CN:** 两个 `exb` 重载分别处理“位域完全位于单个 32-bit 字内”以及“位域跨越两个相邻字”的情况；后者借助 `__funnelshift_rc` 完成。这在量化值不能自然对齐到单个机器字时非常有用。

## Key Concepts / 关键概念
- EN: The helpers are minimal on purpose: GPTQ kernels call them in very hot loops.
  CN: 这些辅助函数刻意保持极简，因为 GPTQ 内核会在非常热的循环中调用它们。
- EN: `dq_ns` separates zero-point correction from scale multiplication.
  CN: `dq_ns` 把零点修正与 scale 乘法拆开处理。
- EN: Cross-word extraction is required for awkward bit-widths such as 3-bit packing.
  CN: 对于 3-bit 这类不规则位宽，跨字提取是必需的。

## Dependencies / 依赖关系
- EN: Used together with `matrix_view.cuh` in GPTQ CUDA kernels.
  CN: 在 GPTQ CUDA 内核中与 `matrix_view.cuh` 配合使用。
- EN: Depends only on CUDA half/bit-manipulation intrinsics already available in the including translation unit.
  CN: 仅依赖包含该头文件的翻译单元中已可用的 CUDA half/位操作 intrinsic。
