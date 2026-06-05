# compat.cuh — Code Analysis / 代码分析

## Source / 来源
- **File**: `csrc/quantization/gptq/compat.cuh`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Supplies compatibility `atomicAdd` implementations for `half` and `half2` on older CUDA architectures and ROCm builds. / [CN] 为旧版 CUDA 架构和部分 ROCm 环境提供 `half` 与 `half2` 的兼容 `atomicAdd` 实现。

## Line-by-Line Analysis / 逐行分析
### [10-28] CAS-based fallback for `half`
```cpp
__device__ __forceinline__ void atomicAdd_half(half* address, half val) {
  unsigned int* address_as_ui =
      (unsigned int*)((char*)address - ((size_t)address & 2));
  unsigned int old = *address_as_ui;
  unsigned int assumed;

  do {
    assumed = old;
    __half_raw hsum;
    hsum.x = (size_t)address & 2 ? (old >> 16) : (old & 0xffff);
    half tmpres = __hadd(hsum, val);
    ...
    old = atomicCAS(address_as_ui, assumed, old);
  } while (assumed != old);
}
```
**EN:** Native `atomicAdd(half*)` is unavailable on some targets, so the code updates a packed 32-bit word with `atomicCAS`. It selects the low or high 16-bit lane depending on address alignment, computes the new half sum, and retries until the compare-and-swap succeeds.
**CN:** 某些目标平台没有原生 `atomicAdd(half*)`，所以这里用 `atomicCAS` 修改承载该 half 的 32 位字。它先根据地址对齐判断使用高 16 位还是低 16 位，再计算新的半精度和并循环重试，直到 CAS 成功。

### [30-42] CAS-based fallback for `half2`
```cpp
__device__ __forceinline__ void atomicAdd_half2(half2* address, half2 val) {
  unsigned int* address_as_ui = (unsigned int*)address;
  unsigned int old = *address_as_ui;
  unsigned int assumed;
  do {
    assumed = old;
    half2 old_val = *((half2*)&old);
    half2 new_val = __hadd2(old_val, val);
    old = atomicCAS(address_as_ui, assumed, *((unsigned int*)&new_val));
  } while (assumed != old);
}
```
**EN:** `half2` already occupies a full 32-bit word, so the fallback is simpler: reinterpret the word as `half2`, add the new vector, and swap the whole word atomically.
**CN:** `half2` 正好占满一个 32 位字，因此兼容实现更直接：把该字重解释为 `half2`，完成向量加法后，再整体进行原子交换。

### [44-60] Architecture-gated wrapper overloads
```cpp
#if defined(__CUDA_ARCH__) ||             (defined(USE_ROCM) && (HIP_VERSION_MAJOR * 100 + HIP_VERSION_MINOR) < 713)
  #if __CUDA_ARCH__ < 700 || defined(USE_ROCM)

__device__ __forceinline__ void atomicAdd(half* address, half val) {
  atomicAdd_half(address, val);
}
...
```
**EN:** The wrapper only activates when the backend lacks the corresponding native half atomic. On newer CUDA devices, the built-in overloads remain in place; on ROCm or older architectures, these compatibility overloads seamlessly replace them.
**CN:** 这些包装重载只会在底层后端缺少原生 half 原子操作时启用。对于较新的 CUDA 设备，仍然使用系统内建实现；而在 ROCm 或旧架构上，则自动切换到这里的兼容版本。

## Key Concepts / 关键概念
- **EN:** `atomicCAS` is the standard fallback technique for custom atomic updates on packed data.
- **CN:** 对于打包数据，自定义原子更新的常见回退方案就是 `atomicCAS`。
- **EN:** Address alignment matters for `half`, because two halves share the same 32-bit storage unit.
- **CN:** `half` 的地址对齐非常关键，因为两个 half 共用一个 32 位存储单元。

## Dependencies / 依赖关系
- **EN:** Relies on CUDA/ROCm half intrinsics such as `__hadd`, `__hadd2`, and `atomicCAS`.
- **CN:** 依赖 CUDA/ROCm 的 `__hadd`、`__hadd2`、`atomicCAS` 等 half 内建函数。
- **EN:** Used by `q_gemm.cu` when kernels accumulate partial results into FP16 outputs.
- **CN:** 在 `q_gemm.cu` 中被用于将部分结果累加到 FP16 输出时的原子更新。
