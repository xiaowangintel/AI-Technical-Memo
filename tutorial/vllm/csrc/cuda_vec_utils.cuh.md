# cuda_vec_utils.cuh — Code Analysis / 代码分析

## Source / 来源
- **File**: `csrc/cuda_vec_utils.cuh`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Provides vector types, packed-type converters, alignment checks, and low-level 32/128/256-bit load-store primitives used by performance-critical CUDA kernels. / [CN] 提供高性能 CUDA 内核所需的向量类型、packed 类型转换、对齐检查以及 32/128/256 位底层读写原语。

## Line-by-Line Analysis / 逐行分析
### Architecture feature flag and vector traits / 架构特性开关与向量 traits
```cpp
#if !defined(USE_ROCM) && defined(__CUDA_ARCH__) && __CUDA_ARCH__ >= 1000 && \
    defined(CUDA_VERSION) && CUDA_VERSION >= 12090
  #define VLLM_256B_PTX_ENABLED 1
#else
  #define VLLM_256B_PTX_ENABLED 0
#endif

namespace vllm {

// ============================================================
// Types and traits
// ============================================================

// 256-bit (32-byte) aligned vector type: 8 x uint32_t
struct alignas(32) u32x8_t {
  uint32_t d[8];
};

// VecTraits — select between 128-bit (int4) and 256-bit
// (u32x8_t) vector types at compile time.
template <bool support_256>
struct VecTraits;

template <>
struct VecTraits<true> {
  static constexpr int ARCH_MAX_VEC_SIZE = 32;
  using vec_t = u32x8_t;
};

template <>
struct VecTraits<false> {
  static constexpr int ARCH_MAX_VEC_SIZE = 16;
  using vec_t = int4;
};
```
**EN:** `VLLM_256B_PTX_ENABLED` enables Blackwell-era 256-bit PTX only when both architecture and toolkit are new enough. `VecTraits` then maps a compile-time boolean to either 32-byte `u32x8_t` vectors or 16-byte `int4` vectors.
**CN:** `VLLM_256B_PTX_ENABLED` 只有在架构和工具链都足够新时才启用 Blackwell 时代的 256 位 PTX。随后 `VecTraits` 会把编译期布尔值映射到 32 字节的 `u32x8_t` 或 16 字节的 `int4` 向量类型。

### Packed and CUDA scalar type conversion / Packed 与 CUDA 标量类型转换
```cpp
// PackedTypeConverter — map between CUDA scalar and packed types
//   half  <-> half2,  __nv_bfloat16 <-> __nv_bfloat162, etc.
template <typename T>
struct PackedTypeConverter {
  static_assert(sizeof(T) == 0,
                "PackedTypeConverter is not specialized for this type.");
};

template <>
struct PackedTypeConverter<half2> {
  using Type = half;
};

template <>
struct PackedTypeConverter<half> {
  using Type = half2;
};

template <>
struct PackedTypeConverter<__nv_bfloat162> {
  using Type = __nv_bfloat16;
};

template <>
struct PackedTypeConverter<__nv_bfloat16> {
  using Type = __nv_bfloat162;
};

template <>
struct PackedTypeConverter<float> {
  using Type = float2;
};

template <>
struct PackedTypeConverter<float2> {
  using Type = float;
};

template <>
struct PackedTypeConverter<c10::Half> {
  using Type = half2;
};

template <>
struct PackedTypeConverter<c10::BFloat16> {
  using Type = __nv_bfloat162;
};

// CUDATypeConverter — map PyTorch scalar types to CUDA scalar
//   c10::Half -> half,  c10::BFloat16 -> __nv_bfloat16
template <typename T>
struct CUDATypeConverter {
  using Type = T;
};

template <>
struct CUDATypeConverter<c10::Half> {
  using Type = half;
};

template <>
struct CUDATypeConverter<c10::BFloat16> {
  using Type = __nv_bfloat16;
};

// PackedVec — typed vector container for packed element access.
//   Derives alignment and element count from VecTraits.
//   Type is the CUDA scalar type (e.g. half, __nv_bfloat16).
template <class Type, bool use_256b>
struct alignas(VecTraits<use_256b>::ARCH_MAX_VEC_SIZE) PackedVec {
  static constexpr int NUM_ELTS =
      VecTraits<use_256b>::ARCH_MAX_VEC_SIZE /
      sizeof(typename PackedTypeConverter<Type>::Type);
  typename PackedTypeConverter<Type>::Type elts[NUM_ELTS];
};
```
**EN:** `PackedTypeConverter` and `CUDATypeConverter` bridge PyTorch scalar wrappers (`c10::Half`, `c10::BFloat16`) with CUDA-native scalar/packed types. `PackedVec` then uses those traits to define an aligned container with the correct number of packed elements.
**CN:** `PackedTypeConverter` 与 `CUDATypeConverter` 在 PyTorch 的标量包装类型（`c10::Half`、`c10::BFloat16`）和 CUDA 原生标量/packed 类型之间建立桥梁。`PackedVec` 再利用这些 traits 定义对齐正确且 packed 元素数合适的容器。

### 128-bit and 256-bit load/store primitives / 128 位与 256 位读写原语
```cpp
// 256-bit load / store — SM100+ only (PTX v8 instructions).
__device__ __forceinline__ void ld256(u32x8_t& val, const u32x8_t* ptr) {
#if VLLM_256B_PTX_ENABLED
  asm volatile("ld.global.nc.v8.u32 {%0,%1,%2,%3,%4,%5,%6,%7}, [%8];\n"
               : "=r"(val.d[0]), "=r"(val.d[1]), "=r"(val.d[2]), "=r"(val.d[3]),
                 "=r"(val.d[4]), "=r"(val.d[5]), "=r"(val.d[6]), "=r"(val.d[7])
               : "l"(ptr));
#else
  assert(false && "ld256 requires SM100+ with CUDA 12.9+");
#endif
}

__device__ __forceinline__ void st256(u32x8_t& val, u32x8_t* ptr) {
#if VLLM_256B_PTX_ENABLED
  asm volatile("st.global.v8.u32 [%0], {%1,%2,%3,%4,%5,%6,%7,%8};\n"
               :
               : "l"(ptr), "r"(val.d[0]), "r"(val.d[1]), "r"(val.d[2]),
                 "r"(val.d[3]), "r"(val.d[4]), "r"(val.d[5]), "r"(val.d[6]),
                 "r"(val.d[7])
               : "memory");
#else
  assert(false && "st256 requires SM100+ with CUDA 12.9+");
#endif
}

// Generic ld256 / st256 for any 32-byte aligned type (e.g. PackedVec).
// Non-template overloads above are preferred for u32x8_t.
template <typename T>
__device__ __forceinline__ void ld256(T& val, const T* ptr) {
  static_assert(sizeof(T) == 32, "ld256 requires a 32-byte type");
  ld256(reinterpret_cast<u32x8_t&>(val), reinterpret_cast<const u32x8_t*>(ptr));
}

template <typename T>
__device__ __forceinline__ void st256(T& val, T* ptr) {
  static_assert(sizeof(T) == 32, "st256 requires a 32-byte type");
  st256(reinterpret_cast<u32x8_t&>(val), reinterpret_cast<u32x8_t*>(ptr));
}
```
**EN:** These helpers expose raw vector transactions. The 256-bit versions use inline PTX and assert if compiled for unsupported targets, while the 128-bit helpers reuse `int4` plus `__ldg` for a broadly available fast path.
**CN:** 这些辅助函数直接暴露底层向量事务。256 位版本使用内联 PTX，并在不支持的目标上通过断言阻止误用；128 位辅助函数则复用 `int4` 和 `__ldg`，提供更通用的快速路径。

### Cache-streaming and predicated loads / Cache-streaming 与带谓词的读取
```cpp
// 256-bit cache-streaming (.cs) load / store  — SM100+ only.
__forceinline__ __device__ u32x8_t ld256_cs(const u32x8_t* addr) {
#if VLLM_256B_PTX_ENABLED
  u32x8_t val;
  asm volatile("ld.global.cs.v8.u32 {%0,%1,%2,%3,%4,%5,%6,%7}, [%8];"
               : "=r"(val.d[0]), "=r"(val.d[1]), "=r"(val.d[2]), "=r"(val.d[3]),
                 "=r"(val.d[4]), "=r"(val.d[5]), "=r"(val.d[6]), "=r"(val.d[7])
               : "l"(addr));
  return val;
#else
  assert(false && "ld256_cs requires SM100+ with CUDA 12.9+");
  return u32x8_t{};
#endif
}

__forceinline__ __device__ void st256_cs(u32x8_t* addr, u32x8_t val) {
#if VLLM_256B_PTX_ENABLED
  asm volatile(
      "st.global.cs.v8.u32 [%0], {%1,%2,%3,%4,%5,%6,%7,%8};" ::"l"(addr),
      "r"(val.d[0]), "r"(val.d[1]), "r"(val.d[2]), "r"(val.d[3]), "r"(val.d[4]),
      "r"(val.d[5]), "r"(val.d[6]), "r"(val.d[7]));
#else
  assert(false && "st256_cs requires SM100+ with CUDA 12.9+");
#endif
}

// 32-bit load / store.
__device__ __forceinline__ int ld32(const int* addr) { return __ldg(addr); }

__device__ __forceinline__ void st32(int* addr, int val) { *addr = val; }

// 32-bit cache-streaming (.cs) load / store.
// Falls back to ld32/st32 on ROCm (no .cs hint).
__forceinline__ __device__ int ld32_cs(const int* addr) {
  int val;
#ifndef USE_ROCM
  asm volatile("ld.global.cs.b32 %0, [%1];" : "=r"(val) : "l"(addr));
#else
  val = ld32(addr);
#endif
  return val;
}

__forceinline__ __device__ void st32_cs(int* addr, int val) {
#ifndef USE_ROCM
  asm volatile("st.global.cs.b32 [%0], %1;" ::"l"(addr), "r"(val));
#else
  st32(addr, val);
#endif
}

// 128-bit cache-streaming (.cs) load / store.
// Falls back to ld128/st128 on ROCm (no .cs hint).
__forceinline__ __device__ int4 ld128_cs(const int4* addr) {
  int4 val;
#ifndef USE_ROCM
  asm volatile("ld.global.cs.v4.u32 {%0,%1,%2,%3}, [%4];"
               : "=r"(val.x), "=r"(val.y), "=r"(val.z), "=r"(val.w)
               : "l"(addr));
#else
  ld128(val, addr);
#endif
  return val;
}

__forceinline__ __device__ void st128_cs(int4* addr, int4 val) {
#ifndef USE_ROCM
  asm volatile("st.global.cs.v4.u32 [%0], {%1,%2,%3,%4};" ::"l"(addr),
               "r"(val.x), "r"(val.y), "r"(val.z), "r"(val.w));
#else
  st128(val, addr);
#endif
}

// Predicated 256-bit / 128-bit cache-global (.cg) loads.
// Returns zero if pred is false.  SM100+ only.
__device__ __forceinline__ void ld256_cg_or_zero(u32x8_t& val, const void* ptr,
                                                 bool pred) {
#if VLLM_256B_PTX_ENABLED
  asm volatile(
      "{\n"
      "  .reg .pred pr;\n"
      "  setp.ne.u32 pr, %8, 0;\n"
      "  mov.u32 %0, 0;\n"
      "  mov.u32 %1, 0;\n"
      "  mov.u32 %2, 0;\n"
      "  mov.u32 %3, 0;\n"
      "  mov.u32 %4, 0;\n"
      "  mov.u32 %5, 0;\n"
      "  mov.u32 %6, 0;\n"
      "  mov.u32 %7, 0;\n"
      "  @pr ld.global.cg.v8.u32 {%0,%1,%2,%3,%4,%5,%6,%7}, [%9];\n"
      "}\n"
      : "=r"(val.d[0]), "=r"(val.d[1]), "=r"(val.d[2]), "=r"(val.d[3]),
        "=r"(val.d[4]), "=r"(val.d[5]), "=r"(val.d[6]), "=r"(val.d[7])
      : "r"((int)pred), "l"(ptr));
#else
  assert(false && "ld256_cg_or_zero requires SM100+ with CUDA 12.9+");
#endif
}

__device__ __forceinline__ void ld128_cg_or_zero(uint4& val, const void* ptr,
                                                 bool pred) {
#ifndef USE_ROCM
  uint32_t r0, r1, r2, r3;

  asm volatile(
      "{\n"
      "  .reg .pred pr;\n"
      "  setp.ne.u32 pr, %4, 0;\n"
      "  mov.u32 %0, 0;\n"
      "  mov.u32 %1, 0;\n"
      "  mov.u32 %2, 0;\n"
      "  mov.u32 %3, 0;\n"
      "  @pr ld.global.cg.v4.u32 {%0,%1,%2,%3}, [%5];\n"
      "}\n"
      : "=r"(r0), "=r"(r1), "=r"(r2), "=r"(r3)
      : "r"((int)pred), "l"(ptr));

  val = uint4{r0, r1, r2, r3};
#else
  assert(false && "ld128_cg_or_zero is not supported on ROCm");
#endif
}
```
**EN:** The `.cs` helpers request streaming cache behavior for 32/128/256-bit accesses, while `ld256_cg_or_zero` and `ld128_cg_or_zero` support predicated “load or zero” behavior. These forms are useful when kernels want explicit cache hints or need boundary-safe vector loads.
**CN:** `.cs` 辅助函数为 32/128/256 位访问请求 streaming cache 行为，而 `ld256_cg_or_zero` 与 `ld128_cg_or_zero` 则支持“带谓词的加载，否则置零”。当内核需要显式 cache hint 或需要边界安全的向量加载时，这些形式非常有用。

### Alignment helpers and packed arithmetic / 对齐辅助与 packed 算术
```cpp
__host__ __device__ __forceinline__ bool is_16byte_aligned(const void* ptr) {
  return (reinterpret_cast<uintptr_t>(ptr) & 15) == 0;
}

__host__ __device__ __forceinline__ bool is_32byte_aligned(const void* ptr) {
  return (reinterpret_cast<uintptr_t>(ptr) & 31) == 0;
}

// ============================================================
// Packed type conversion and arithmetic
// ============================================================

template <typename packed_t>
__device__ __forceinline__ float2 cast_to_float2(const packed_t& val) {
  if constexpr (std::is_same_v<packed_t, __nv_bfloat162>) {
    return __bfloat1622float2(val);
  } else if constexpr (std::is_same_v<packed_t, __half2>) {
    return __half22float2(val);
  } else if constexpr (std::is_same_v<packed_t, float2>) {
    return float2(val);
  }
}

template <typename packed_t>
__device__ __forceinline__ packed_t cast_to_packed(const float2& val) {
  if constexpr (std::is_same_v<packed_t, __nv_bfloat162>) {
    return __float22bfloat162_rn(val);
  } else if constexpr (std::is_same_v<packed_t, __half2>) {
    return __float22half2_rn(val);
  } else if constexpr (std::is_same_v<packed_t, float2>) {
    return float2(val);
  }
}

template <typename packed_t>
__device__ __forceinline__ packed_t packed_mul(const packed_t& x,
                                               const packed_t& y) {
  if constexpr (std::is_same_v<packed_t, __nv_bfloat162> ||
                std::is_same_v<packed_t, __half2>) {
    return __hmul2(x, y);
  } else if constexpr (std::is_same_v<packed_t, float2>) {
    return make_float2(x.x * y.x, x.y * y.y);
  }
}

}  // namespace vllm
```
**EN:** The last block supplies cheap alignment checks and generic conversions between packed registers and `float2`, then defines `packed_mul` so higher-level kernels can write type-agnostic vector math.
**CN:** 最后一段提供了廉价的对齐检查，以及 packed 寄存器与 `float2` 之间的通用转换，再通过 `packed_mul` 让上层内核可以编写与具体类型无关的向量乘法逻辑。

## Key Concepts / 关键概念
- **EN:** This header is the common substrate for many fast paths in vLLM CUDA code.
  **CN:** 这个头文件是 vLLM 大量 CUDA 快速路径的公共底座。
- **EN:** Type traits are used to keep kernels generic while still emitting architecture-specific vector instructions.
  **CN:** 通过类型 traits，内核既能保持泛型写法，又能发出与架构相关的向量指令。
- **EN:** Alignment checks are critical because many vectorized kernels only enable the fast path when pointers satisfy 16- or 32-byte alignment.
  **CN:** 对齐检查非常关键，因为许多向量化内核只有在指针满足 16 或 32 字节对齐时才会启用快速路径。

## Dependencies / 依赖关系
- **EN:** Depends on CUDA/HIP vector types such as `int4`, `float2`, `half2`, and BF16 pair types.
  **CN:** 依赖 `int4`、`float2`、`half2` 以及 BF16 成对类型等 CUDA/HIP 向量类型。
- **EN:** Higher-level consumers include `activation_kernels.cu`, `concat_mla_q.cuh`, and several fused-attention kernels.
  **CN:** 上层使用者包括 `activation_kernels.cu`、`concat_mla_q.cuh` 以及多个融合注意力内核。
