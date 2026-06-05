# async_util.cuh — Code Analysis / 代码分析

## Source / 来源
- **File**: `csrc/async_util.cuh`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Wraps backend-specific asynchronous copy primitives so kernels can stage data from global memory into shared memory with one interface. / [CN] 封装后端相关的异步拷贝原语，让内核能够通过统一接口把全局内存数据搬运到共享内存。

## Line-by-Line Analysis / 逐行分析
### 16-byte cache-global async copy / 16 字节 cache-global 异步拷贝
```cpp
__device__ __forceinline__ void cp_async_shared_global_16_cg(
    void* smem_ptr, const void* glob_ptr) {
#if defined(USE_ROCM)
  *reinterpret_cast<int4*>(smem_ptr) = *reinterpret_cast<const int4*>(glob_ptr);
#elif defined(__CUDA_ARCH__) && __CUDA_ARCH__ >= 800
  uint32_t smem = static_cast<uint32_t>(__cvta_generic_to_shared(smem_ptr));
  asm volatile("cp.async.cg.shared.global [%0], [%1], 16;\n"
               :
               : "r"(smem), "l"(glob_ptr));
#elif defined(__CUDA_ARCH__)
  *reinterpret_cast<int4*>(smem_ptr) = *reinterpret_cast<const int4*>(glob_ptr);
#else
  (void)smem_ptr;
  (void)glob_ptr;
#endif
}
```
**EN:** On CUDA SM80+, this emits `cp.async.cg.shared.global` for a 16-byte transfer into shared memory. ROCm and older CUDA architectures transparently fall back to a normal `int4` copy, preserving semantics without asynchronous overlap.
**CN:** 在 CUDA SM80+ 上，这里会发出 `cp.async.cg.shared.global`，执行 16 字节到共享内存的异步搬运。ROCm 和更老的 CUDA 架构则自动退回为普通 `int4` 拷贝，语义一致但没有异步重叠能力。

### Variable-size cache-all copy / 可变大小 cache-all 拷贝
```cpp
__device__ __forceinline__ void cp_async_shared_global_ca(void* smem_ptr,
                                                          const void* glob_ptr,
                                                          int size_bytes) {
#if defined(USE_ROCM)
  if (size_bytes == 4) {
    *reinterpret_cast<uint32_t*>(smem_ptr) =
        *reinterpret_cast<const uint32_t*>(glob_ptr);
  } else if (size_bytes == 8) {
    *reinterpret_cast<uint64_t*>(smem_ptr) =
        *reinterpret_cast<const uint64_t*>(glob_ptr);
  } else {
    *reinterpret_cast<int4*>(smem_ptr) =
        *reinterpret_cast<const int4*>(glob_ptr);
  }
#elif defined(__CUDA_ARCH__) && __CUDA_ARCH__ >= 800
  uint32_t smem = static_cast<uint32_t>(__cvta_generic_to_shared(smem_ptr));
  if (size_bytes == 4) {
    asm volatile("cp.async.ca.shared.global [%0], [%1], 4;\n"
                 :
                 : "r"(smem), "l"(glob_ptr));
  } else if (size_bytes == 8) {
    asm volatile("cp.async.ca.shared.global [%0], [%1], 8;\n"
                 :
                 : "r"(smem), "l"(glob_ptr));
  } else {
    asm volatile("cp.async.ca.shared.global [%0], [%1], 16;\n"
                 :
                 : "r"(smem), "l"(glob_ptr));
  }
#elif defined(__CUDA_ARCH__)
  if (size_bytes == 4) {
    *reinterpret_cast<uint32_t*>(smem_ptr) =
        *reinterpret_cast<const uint32_t*>(glob_ptr);
  } else if (size_bytes == 8) {
    *reinterpret_cast<uint64_t*>(smem_ptr) =
        *reinterpret_cast<const uint64_t*>(glob_ptr);
  } else {
    *reinterpret_cast<int4*>(smem_ptr) =
        *reinterpret_cast<const int4*>(glob_ptr);
  }
#else
  (void)smem_ptr;
  (void)glob_ptr;
  (void)size_bytes;
#endif
}
```
**EN:** `cp_async_shared_global_ca` generalizes the operation to 4, 8, or 16 bytes. The CUDA path emits different inline assembly opcodes per size, while the ROCm and legacy paths choose the corresponding scalar/vector load-store sequence.
**CN:** `cp_async_shared_global_ca` 把操作推广到 4、8、16 字节三种大小。CUDA 路径会按大小发出不同的内联汇编指令；ROCm 与旧架构路径则选择对应的标量或向量读写序列。

### Group commit and wait / 分组提交与等待
```cpp
__device__ __forceinline__ void cp_async_commit_group() {
#if defined(__CUDA_ARCH__) && __CUDA_ARCH__ >= 800 && !defined(USE_ROCM)
  asm volatile("cp.async.commit_group;\n" ::);
#endif
}

template <int n>
__device__ __forceinline__ void cp_async_wait_group() {
#if defined(__CUDA_ARCH__) && __CUDA_ARCH__ >= 800 && !defined(USE_ROCM)
  asm volatile("cp.async.wait_group %0;\n" : : "n"(n));
#endif
}
```
**EN:** These wrappers expose `cp.async.commit_group` and `cp.async.wait_group<n>` only when the architecture supports them. They are intentionally no-ops elsewhere so higher-level kernels do not need scattered backend conditionals.
**CN:** 这些包装函数只在架构支持时暴露 `cp.async.commit_group` 与 `cp.async.wait_group<n>`。在其他平台上它们故意成为空操作，从而避免上层内核到处写后端条件分支。

## Key Concepts / 关键概念
- **EN:** The header isolates inline PTX/ISA details from higher-level kernels.
  **CN:** 这个头文件把内联 PTX/ISA 细节从上层内核中隔离出来。
- **EN:** CUDA uses true asynchronous shared-memory prefetch, while ROCm gets a correctness-preserving synchronous fallback.
  **CN:** CUDA 使用真正的异步共享内存预取，而 ROCm 走保持正确性的同步退化路径。
- **EN:** Group commit/wait APIs are the synchronization points that make staged `cp.async` traffic visible to later computation.
  **CN:** 分组提交/等待接口是 `cp.async` 分阶段搬运完成后、让后续计算可见的同步点。

## Dependencies / 依赖关系
- **EN:** Relies on CUDA builtins such as `__cvta_generic_to_shared` and inline PTX `cp.async` instructions.
  **CN:** 依赖 `__cvta_generic_to_shared` 等 CUDA builtin 以及内联 PTX `cp.async` 指令。
- **EN:** Uses `USE_ROCM` and `__CUDA_ARCH__` guards to switch implementations at compile time.
  **CN:** 通过 `USE_ROCM` 与 `__CUDA_ARCH__` 条件编译在不同实现之间切换。
