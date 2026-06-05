# Atomic.cuh — Code Analysis / 代码分析
## Source / 来源
- File: `aten/src/ATen/cuda/Atomic.cuh`
- Repository: `pytorch/pytorch`
- Purpose (EN): This file declares CUDA backend support, with primary focus on `AtomicFPOp`, `Atomic`, `ATOMIC_INTEGER_IMPL`.
- 用途（中文）: 该文件声明CUDA 后端支持，核心关注对象是 `AtomicFPOp`, `Atomic`, `ATOMIC_INTEGER_IMPL`。

## Line-by-Line Analysis / 逐行分析

### Lines 1-16
```cpp
#pragma once

#include <cuda.h>
#include <c10/util/Half.h>
#include <c10/util/BFloat16.h>

#include <ATen/NumericUtils.h>

#if !(defined(USE_ROCM) || ((defined(__CUDA_ARCH__) && (__CUDA_ARCH__ < 800))))
#include <cuda_bf16.h>
#endif

template <typename T>
struct AtomicFPOp;

template <>
```
- EN: Focus symbols: `AtomicFPOp`. This block sets up header dependencies and the initial compilation context. The file brings required declarations into scope before entering the main ATen logic.
- CN: 关注符号：`AtomicFPOp`。该代码块设置头文件依赖与初始编译上下文。文件在进入主要 ATen 逻辑前先引入所需声明。

### Lines 17-32
```cpp
struct AtomicFPOp<at::Half> {
  template <typename func_t>
  inline __device__ at::Half operator() (at::Half *address, at::Half val, const func_t& func) {
    unsigned int * address_as_ui =
      (unsigned int *) ((char *)address - ((size_t)address & 2));
    unsigned int old = *address_as_ui;
    unsigned int assumed;

    at::Half hsum;
    do {
      assumed = old;
      hsum.x = (size_t)address & 2 ? (old >> 16) : (old & 0xffff);
      hsum = func(hsum, val);
      old = (size_t)address & 2 ? (old & 0xffff) | (hsum.x << 16) : (old & 0xffff0000) | hsum.x;
      old = atomicCAS(address_as_ui, assumed, old);
    } while (assumed != old);
```
- EN: Focus symbols: `AtomicFPOp`, `operator`, `func`, `atomicCAS`. This block declares the main types, aliases, or compile-time descriptors used by this file. These declarations define the shape of the API and the data carried through later code.
- CN: 关注符号：`AtomicFPOp`, `operator`, `func`, `atomicCAS`。该代码块声明此文件使用的核心类型、别名或编译期描述对象。这些声明决定了 API 的形状以及后续代码中流动的数据结构。

### Lines 33-48
```cpp
    hsum.x = (size_t)address & 2 ? (old >> 16) : (old & 0xffff);
    return hsum;
  }
};

template <>
struct AtomicFPOp<at::BFloat16> {
  template <typename func_t>
  inline __device__ at::BFloat16 operator() (at::BFloat16 *address, at::BFloat16 val, const func_t& func) {
    unsigned int * address_as_ui =
      (unsigned int *) ((char *)address - ((size_t)address & 2));
    unsigned int old = *address_as_ui;
    unsigned int assumed;

    at::BFloat16 bsum;
    do {
```
- EN: Focus symbols: `AtomicFPOp`, `operator`. This block declares the main types, aliases, or compile-time descriptors used by this file. These declarations define the shape of the API and the data carried through later code.
- CN: 关注符号：`AtomicFPOp`, `operator`。该代码块声明此文件使用的核心类型、别名或编译期描述对象。这些声明决定了 API 的形状以及后续代码中流动的数据结构。

### Lines 49-67
```cpp
      assumed = old;
      bsum.x = (size_t)address & 2 ? (old >> 16) : (old & 0xffff);
      bsum = func(bsum, val);
      old = (size_t)address & 2 ? (old & 0xffff) | (bsum.x << 16) : (old & 0xffff0000) | bsum.x;
      old = atomicCAS(address_as_ui, assumed, old);
    } while (assumed != old);
    bsum.x = (size_t)address & 2 ? (old >> 16) : (old & 0xffff);
    return bsum.x;
  }
};

template <>
struct AtomicFPOp<double> {
  template <typename func_t>
  inline __device__ double operator() (double * address, double val, const func_t& func) {
    unsigned long long int* address_as_ull = (unsigned long long int*)address;
    unsigned long long int old = *address_as_ull;
    unsigned long long int assumed;

```
- EN: Focus symbols: `AtomicFPOp`, `func`, `atomicCAS`, `operator`. This block declares the main types, aliases, or compile-time descriptors used by this file. These declarations define the shape of the API and the data carried through later code.
- CN: 关注符号：`AtomicFPOp`, `func`, `atomicCAS`, `operator`。该代码块声明此文件使用的核心类型、别名或编译期描述对象。这些声明决定了 API 的形状以及后续代码中流动的数据结构。

### Lines 68-83
```cpp
    do {
      assumed = old;
      old = atomicCAS(address_as_ull, assumed, func(val, assumed));
      // Note: uses integer comparison to avoid hang in case of NaN (since NaN != NaN)
    } while (assumed != old);

    return __longlong_as_double(old);
  }
};

#define ATOMIC_INTEGER_IMPL(NAME)                                                                                      \
template <typename T, size_t n>                                                                                        \
struct Atomic##NAME##IntegerImpl;                                                                                      \
                                                                                                                       \
template<typename T>                                                                                                   \
struct Atomic##NAME##IntegerImpl<T, 1> {                                                                               \
```
- EN: Focus symbols: `Atomic`, `ATOMIC_INTEGER_IMPL`, `atomicCAS`, `func`, `__longlong_as_double`. This block declares the main types, aliases, or compile-time descriptors used by this file. These declarations define the shape of the API and the data carried through later code.
- CN: 关注符号：`Atomic`, `ATOMIC_INTEGER_IMPL`, `atomicCAS`, `func`, `__longlong_as_double`。该代码块声明此文件使用的核心类型、别名或编译期描述对象。这些声明决定了 API 的形状以及后续代码中流动的数据结构。

### Lines 84-99
```cpp
  template <typename func_t>                                                                                           \
  inline __device__ void operator()(T *address, T val, const func_t& func) {                                           \
    size_t offset = (size_t)address & 3;                                                                               \
    uint32_t * address_as_ui = (uint32_t *)((char *)address - offset);                                                 \
    uint32_t old = *address_as_ui;                                                                                     \
    uint32_t shift = offset * 8;                                                                                       \
    uint32_t old_byte;                                                                                                 \
    uint32_t newval;                                                                                                   \
    uint32_t assumed;                                                                                                  \
                                                                                                                       \
    do {                                                                                                               \
      assumed = old;                                                                                                   \
      old_byte = (old >> shift) & 0xff;                                                                                \
      newval = static_cast<uint8_t>(func(val, static_cast<T>(old_byte)));                                              \
      newval = (old & ~(0x000000ff << shift)) | (newval << shift);                                                     \
      old = atomicCAS(address_as_ui, assumed, newval);                                                                 \
```
- EN: Focus symbols: `operator`, `func`, `~`, `atomicCAS`. This block builds generic/template machinery for multiple dtypes, devices, or call patterns. Templates let ATen share one design across many concrete instantiations.
- CN: 关注符号：`operator`, `func`, `~`, `atomicCAS`。该代码块构建适配多种数据类型、设备或调用模式的模板机制。模板让 ATen 能在多个具体实例之间共享同一套设计。

### Lines 100-115
```cpp
    } while (assumed != old);                                                                                          \
  }                                                                                                                    \
};                                                                                                                     \
                                                                                                                       \
template<typename T>                                                                                                   \
struct Atomic##NAME##IntegerImpl<T, 2> {                                                                               \
  template <typename func_t>                                                                                           \
  inline __device__ void operator()(T *address, T val, const func_t& func) {                                           \
    size_t offset = (size_t)address & 2;                                                                               \
    uint32_t * address_as_ui = (uint32_t *)((char *)address - offset);                                                 \
    bool is_32_align = offset;                                                                                         \
    uint32_t old = *address_as_ui;                                                                                     \
    uint32_t old_bytes;                                                                                                \
    uint32_t newval;                                                                                                   \
    uint32_t assumed;                                                                                                  \
                                                                                                                       \
```
- EN: Focus symbols: `Atomic`, `operator`. This block declares the main types, aliases, or compile-time descriptors used by this file. These declarations define the shape of the API and the data carried through later code.
- CN: 关注符号：`Atomic`, `operator`。该代码块声明此文件使用的核心类型、别名或编译期描述对象。这些声明决定了 API 的形状以及后续代码中流动的数据结构。

### Lines 116-131
```cpp
    do {                                                                                                               \
      assumed = old;                                                                                                   \
      old_bytes = is_32_align ? old >> 16 : old & 0xffff;                                                              \
      newval = static_cast<uint16_t>(func(val, static_cast<T>(old_bytes)));                                            \
      newval = is_32_align ? (old & 0xffff) | (newval << 16) : (old & 0xffff0000) | newval;                            \
      old = atomicCAS(address_as_ui, assumed, newval);                                                                 \
    } while (assumed != old);                                                                                          \
  }                                                                                                                    \
};                                                                                                                     \
                                                                                                                       \
template<typename T>                                                                                                   \
struct Atomic##NAME##IntegerImpl<T, 4> {                                                                               \
  template <typename func_t>                                                                                           \
  inline __device__ void operator()(T *address, T val, const func_t& func) {                                           \
    uint32_t * address_as_ui = (uint32_t *) (address);                                                                 \
    uint32_t old = *address_as_ui;                                                                                     \
```
- EN: Focus symbols: `Atomic`, `func`, `atomicCAS`, `operator`. This block declares the main types, aliases, or compile-time descriptors used by this file. These declarations define the shape of the API and the data carried through later code.
- CN: 关注符号：`Atomic`, `func`, `atomicCAS`, `operator`。该代码块声明此文件使用的核心类型、别名或编译期描述对象。这些声明决定了 API 的形状以及后续代码中流动的数据结构。

### Lines 132-147
```cpp
    uint32_t newval;                                                                                                   \
    uint32_t assumed;                                                                                                  \
                                                                                                                       \
    do {                                                                                                               \
      assumed = old;                                                                                                   \
      newval = static_cast<uint32_t>(func(val, static_cast<T>(old)));                                                  \
      old = atomicCAS(address_as_ui, assumed, newval);                                                                 \
    } while (assumed != old);                                                                                          \
  }                                                                                                                    \
};                                                                                                                     \
                                                                                                                       \
template<typename T>                                                                                                   \
struct Atomic##NAME##IntegerImpl<T, 8> {                                                                               \
  template <typename func_t>                                                                                           \
  inline __device__ void operator()(T *address, T val, const func_t& func) {                                           \
    unsigned long long * address_as_ui = (unsigned long long *) (address);                                             \
```
- EN: Focus symbols: `Atomic`, `func`, `atomicCAS`, `operator`. This block declares the main types, aliases, or compile-time descriptors used by this file. These declarations define the shape of the API and the data carried through later code.
- CN: 关注符号：`Atomic`, `func`, `atomicCAS`, `operator`。该代码块声明此文件使用的核心类型、别名或编译期描述对象。这些声明决定了 API 的形状以及后续代码中流动的数据结构。

### Lines 148-163
```cpp
    unsigned long long old = *address_as_ui;                                                                           \
    unsigned long long newval;                                                                                         \
    unsigned long long assumed;                                                                                        \
                                                                                                                       \
    do {                                                                                                               \
      assumed = old;                                                                                                   \
      newval = static_cast<uint64_t>(func(val, static_cast<T>(old)));                                                  \
      old = atomicCAS(address_as_ui, assumed, newval);                                                                 \
    } while (assumed != old);                                                                                          \
  }                                                                                                                    \
};


# define GPU_ATOMIC_INTEGER(NAME, OP, DTYPE)                                                                           \
inline __device__ void gpuAtomic##NAME(DTYPE *address, DTYPE val) {                                             \
Atomic##NAME##IntegerImpl<DTYPE, sizeof(DTYPE)>()(address,                                                             \
```
- EN: Focus symbols: `func`, `atomicCAS`, `NAME`. This block executes runtime branching, validation, or state updates. This is where the file reacts to dynamic conditions instead of only describing static structure.
- CN: 关注符号：`func`, `atomicCAS`, `NAME`。该代码块执行运行时分支、校验或状态更新。这里体现了文件对动态条件的响应，而不只是静态结构描述。

### Lines 164-181
```cpp
                                                      val,                                                             \
                                                      [](DTYPE a, DTYPE b) {                                           \
                                                          return OP;                                                   \
                                                      });                                                              \
}                                                                                                                      \

ATOMIC_INTEGER_IMPL(Add)
GPU_ATOMIC_INTEGER(Add, a || b, bool)

// Don't instantiate gpuAtomicAdd with the macro as it seems non-standard (see int32, int64)
inline __device__ void gpuAtomicAdd(uint8_t *address, uint8_t val) {
  AtomicAddIntegerImpl<uint8_t, sizeof(uint8_t)>()(address,
                                                   val,
                                                   [](uint8_t a, uint8_t b) {
                                                      return a + b;
                                                   });
}

```
- EN: Focus symbols: `ATOMIC_INTEGER_IMPL`, `GPU_ATOMIC_INTEGER`, `gpuAtomicAdd`. This block declares or implements callable behavior for this module. Inputs are translated into lower-level operations, state updates, or returned values here.
- CN: 关注符号：`ATOMIC_INTEGER_IMPL`, `GPU_ATOMIC_INTEGER`, `gpuAtomicAdd`。该代码块声明或实现该模块的可调用行为。这里把输入转换为更底层操作、状态更新或返回值。

### Lines 182-201
```cpp
inline  __device__ void gpuAtomicAdd(int8_t *address, int8_t val) {
  AtomicAddIntegerImpl<int8_t, sizeof(int8_t)>()(address,
                                                 val,
                                                 [](int8_t a, int8_t b) {
                                                   return a + b;
                                                 });
}

inline  __device__ void gpuAtomicAdd(int16_t *address, int16_t val) {
  AtomicAddIntegerImpl<int16_t, sizeof(int16_t)>()(address,
                                                   val,
                                                   [](int16_t a, int16_t b) {
                                                     return a + b;
                                                   });
}

inline __device__ int32_t gpuAtomicAdd(int32_t *address, int32_t val) {
  return atomicAdd(address, val);
}

```
- EN: Focus symbols: `gpuAtomicAdd`, `atomicAdd`. This block declares or implements callable behavior for this module. Inputs are translated into lower-level operations, state updates, or returned values here.
- CN: 关注符号：`gpuAtomicAdd`, `atomicAdd`。该代码块声明或实现该模块的可调用行为。这里把输入转换为更底层操作、状态更新或返回值。

### Lines 202-221
```cpp
inline __device__ void gpuAtomicAdd(int64_t *address, int64_t val) {
#if defined(USE_ROCM)
  __atomic_fetch_add(address, val, __ATOMIC_RELAXED);
#else
  static_assert(sizeof(unsigned long long int) == sizeof(int64_t), "bitwidth change is not allowed");
  atomicAdd(reinterpret_cast<unsigned long long int *>(address), static_cast<unsigned long long int>(val));
#endif
}

inline  __device__ at::Half gpuAtomicAdd(at::Half *address, at::Half val) {
#if defined(USE_ROCM) || ((defined(__CUDA_ARCH__) && (__CUDA_ARCH__ < 700)))
  return AtomicFPOp<at::Half>()(address, val,
                                [](at::Half hsum, at::Half val) {
                                  return hsum + val;
                                });
#else
  return atomicAdd(reinterpret_cast<__half*>(address), val);
#endif
}

```
- EN: Focus symbols: `gpuAtomicAdd`, `__atomic_fetch_add`, `static_assert`, `atomicAdd`. This block declares or implements callable behavior for this module. Inputs are translated into lower-level operations, state updates, or returned values here.
- CN: 关注符号：`gpuAtomicAdd`, `__atomic_fetch_add`, `static_assert`, `atomicAdd`。该代码块声明或实现该模块的可调用行为。这里把输入转换为更底层操作、状态更新或返回值。

### Lines 222-237
```cpp
inline __device__ at::BFloat16 gpuAtomicAdd(at::BFloat16 *address, at::BFloat16 val) {
#if defined(USE_ROCM) || ((defined(__CUDA_ARCH__) && (__CUDA_ARCH__ < 800)))
return AtomicFPOp<at::BFloat16>()(address, val,
                                  [](at::BFloat16 bsum, at::BFloat16 val) {
                                    return bsum + val;
                                  });
#else
  __nv_bfloat16 r = atomicAdd(reinterpret_cast<__nv_bfloat16*>(address), *reinterpret_cast<__nv_bfloat16*>(&val));
  return *reinterpret_cast<c10::BFloat16*>(&r);
#endif
}

#if defined(__CUDA_ARCH__) && (__CUDA_ARCH__ < 600)
// from CUDA C Programmic Guide
inline __device__ double atomicAdd(double* address, double val)
#if defined(__clang__) && defined(__CUDA__)
```
- EN: Focus symbols: `gpuAtomicAdd`, `atomicAdd`. This block declares or implements callable behavior for this module. Inputs are translated into lower-level operations, state updates, or returned values here.
- CN: 关注符号：`gpuAtomicAdd`, `atomicAdd`。该代码块声明或实现该模块的可调用行为。这里把输入转换为更底层操作、状态更新或返回值。

### Lines 238-253
```cpp
#pragma GCC diagnostic push
#pragma GCC diagnostic ignored "-Wgcc-compat"
    __attribute__((enable_if(true, "")))
#pragma GCC diagnostic pop
#endif
{

  return AtomicFPOp<double>()(address, val,
                              [](double val, unsigned long long int assumed) {
                                return __double_as_longlong(val + __longlong_as_double(assumed));
                              });
}
#elif defined(USE_ROCM) || !(defined(__CUDA_ARCH__))

/* Note [hip-clang differences to hcc]
 * ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
```
- EN: Focus symbols: `__attribute__`, `enable_if`, `__double_as_longlong`, `__longlong_as_double`. This block declares or implements callable behavior for this module. Inputs are translated into lower-level operations, state updates, or returned values here.
- CN: 关注符号：`__attribute__`, `enable_if`, `__double_as_longlong`, `__longlong_as_double`。该代码块声明或实现该模块的可调用行为。这里把输入转换为更底层操作、状态更新或返回值。

### Lines 254-271
```cpp
 * The upcoming hip-clang compiler for ROCm differs from hcc in a few details.
 * It exports the __HIP__ macro, we can hence differentiate between hcc and
 * hip-clang. In the below, hcc only received support for atomicAdd with double
 * typing after work week 18312. hip-clang had support from the first version.
 * In general, the code-visible differences between hip-clang and hcc will be
 * minimal.
 */

#if defined(USE_ROCM) && __hcc_workweek__ < 18312 && !__HIP__
  // This needs to be defined for the host side pass
  inline  __device__  double atomicAdd(double *address, double val) { }
#endif
#endif

inline __device__ double gpuAtomicAdd(double *address, double val) {
  return atomicAdd(address, val);
}

```
- EN: Focus symbols: `atomicAdd`, `gpuAtomicAdd`. This block declares or implements callable behavior for this module. Inputs are translated into lower-level operations, state updates, or returned values here.
- CN: 关注符号：`atomicAdd`, `gpuAtomicAdd`。该代码块声明或实现该模块的可调用行为。这里把输入转换为更底层操作、状态更新或返回值。

### Lines 272-291
```cpp
inline __device__ float gpuAtomicAdd(float *address, float val) {
  return atomicAdd(address, val);
}

template<typename T>
inline __device__ void gpuAtomicAdd(c10::complex<T> *address, c10::complex<T> val) {
  gpuAtomicAdd(&address->real_, val.real_);
  gpuAtomicAdd(&address->imag_, val.imag_);
}

/* Note [gpuAtomicAdd vs atomicAdd]
 * ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
 * Some extensions such as torchvision call atomicAdd()
 * directly and require non-library provided data type support. Only for these, we
 * continue to provide atomicAdd overloads.
 */
inline __device__ at::Half atomicAdd(at::Half *address, at::Half val) {
  return gpuAtomicAdd(address, val);
}

```
- EN: Focus symbols: `gpuAtomicAdd`, `atomicAdd`. This block builds generic/template machinery for multiple dtypes, devices, or call patterns. Templates let ATen share one design across many concrete instantiations.
- CN: 关注符号：`gpuAtomicAdd`, `atomicAdd`。该代码块构建适配多种数据类型、设备或调用模式的模板机制。模板让 ATen 能在多个具体实例之间共享同一套设计。

### Lines 292-311
```cpp
inline __device__ at::BFloat16 atomicAdd(at::BFloat16 *address, at::BFloat16 val) {
  return gpuAtomicAdd(address, val);
}

inline __device__ void atomicAdd(uint8_t *address, uint8_t val) {
  gpuAtomicAdd(address, val);
}

inline  __device__ void atomicAdd(int8_t *address, int8_t val) {
  gpuAtomicAdd(address, val);
}

inline  __device__ void atomicAdd(int16_t *address, int16_t val) {
  gpuAtomicAdd(address, val);
}

inline __device__ void atomicAdd(int64_t *address, int64_t val) {
  gpuAtomicAdd(address, val);
}

```
- EN: Focus symbols: `atomicAdd`, `gpuAtomicAdd`. This block declares or implements callable behavior for this module. Inputs are translated into lower-level operations, state updates, or returned values here.
- CN: 关注符号：`atomicAdd`, `gpuAtomicAdd`。该代码块声明或实现该模块的可调用行为。这里把输入转换为更底层操作、状态更新或返回值。

### Lines 312-327
```cpp
inline __device__ void atomicAdd(bool *address, bool val) {
  gpuAtomicAdd(address, val);
}

/* Note [explicitly non-returning atomics]
 * ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
 * AMD's MI100 (gfx908) provides an optimized fp32 atomicAdd, exposed via atomicAddNoRet().
 * Due to compiler limitations, callers must opt-in to guarantee the optimized instruction.
 * This non-returning atomicAddNoRet cannot be used to implement the returning atomicAdd,
 * therefore we need a new API 'gpuAtomicAddNoReturn'.
 */
template<typename T>
inline __device__ void gpuAtomicAddNoReturn(c10::complex<T> *address, c10::complex<T> val) { gpuAtomicAdd(address, val); }
inline __device__ void gpuAtomicAddNoReturn(uint8_t *address, uint8_t val) { gpuAtomicAdd(address, val); }
inline __device__ void gpuAtomicAddNoReturn(int8_t *address, int8_t val) { gpuAtomicAdd(address, val); }
inline __device__ void gpuAtomicAddNoReturn(int16_t *address, int16_t val) { gpuAtomicAdd(address, val); }
```
- EN: Focus symbols: `atomicAdd`, `gpuAtomicAdd`, `MI100`, `atomicAddNoRet`, `gpuAtomicAddNoReturn`. This block builds generic/template machinery for multiple dtypes, devices, or call patterns. Templates let ATen share one design across many concrete instantiations.
- CN: 关注符号：`atomicAdd`, `gpuAtomicAdd`, `MI100`, `atomicAddNoRet`, `gpuAtomicAddNoReturn`。该代码块构建适配多种数据类型、设备或调用模式的模板机制。模板让 ATen 能在多个具体实例之间共享同一套设计。

### Lines 328-343
```cpp
inline __device__ void gpuAtomicAddNoReturn(int32_t *address, int32_t val) { gpuAtomicAdd(address, val); }
inline __device__ void gpuAtomicAddNoReturn(int64_t *address, int64_t val) { gpuAtomicAdd(address, val); }
inline __device__ void gpuAtomicAddNoReturn(bool *address, bool val) { gpuAtomicAdd(address, val); }
inline __device__ void gpuAtomicAddNoReturn(at::Half *address, at::Half val) { gpuAtomicAdd(address, val); }
inline __device__ void gpuAtomicAddNoReturn(at::BFloat16 *address, at::BFloat16 val) { gpuAtomicAdd(address, val); }

/* Note [HIP unsafeAtomicAdd]
 * ~~~~~~~~~~~~~~~~~~~~~~~~~~
 * Use unsafeAtomicAdd instead of atomicAdd for fp32 and fp64.
 * On HIP, atomicAdd is always correct but is a slow CAS loop.
 * unsafeAtomicAdd will use HW instructions and is much faster,
 * but the caller must guarantee the pointer is GPU memory.
 * If the pointer is system memory, the result is a silent no-op.
 * This guarantee is upheld by all PyTorch uses of unsafeAtomicAdd.
 * AMD HIP atomic header file is named amd_hip_atomic.h and is
 * under the LLVM compiler directory.
```
- EN: Focus symbols: `gpuAtomicAddNoReturn`, `gpuAtomicAdd`. This block connects nearby declarations and implementation details. This glue code keeps the surrounding abstraction consistent and usable.
- CN: 关注符号：`gpuAtomicAddNoReturn`, `gpuAtomicAdd`。该代码块连接相邻的声明与实现细节。这些胶水代码让周边抽象保持一致并可被使用。

### Lines 344-360
```cpp
 */
#if defined(USE_ROCM)
inline __device__ void gpuAtomicAddNoReturn(float *address, float val) {
#if defined(__gfx908__)
  atomicAddNoRet(address, val);
#else
  (void)unsafeAtomicAdd(address, val);
#endif
}
inline __device__ void gpuAtomicAddNoReturn(double *address, double val) { (void)unsafeAtomicAdd(address, val); }
#else
inline __device__ void gpuAtomicAddNoReturn(float *address, float val) { gpuAtomicAdd(address, val); }
inline __device__ void gpuAtomicAddNoReturn(double *address, double val) { gpuAtomicAdd(address, val); }
#endif

// Atomic multiplication implementation.

```
- EN: Focus symbols: `gpuAtomicAddNoReturn`, `atomicAddNoRet`, `unsafeAtomicAdd`, `gpuAtomicAdd`. This block declares or implements callable behavior for this module. Inputs are translated into lower-level operations, state updates, or returned values here.
- CN: 关注符号：`gpuAtomicAddNoReturn`, `atomicAddNoRet`, `unsafeAtomicAdd`, `gpuAtomicAdd`。该代码块声明或实现该模块的可调用行为。这里把输入转换为更底层操作、状态更新或返回值。

### Lines 361-376
```cpp
ATOMIC_INTEGER_IMPL(Mul)
GPU_ATOMIC_INTEGER(Mul, a * b, uint8_t)
GPU_ATOMIC_INTEGER(Mul, a * b, int8_t)
GPU_ATOMIC_INTEGER(Mul, a * b, int16_t)
GPU_ATOMIC_INTEGER(Mul, a * b, int32_t)
GPU_ATOMIC_INTEGER(Mul, a * b, int64_t)

inline __device__ at::Half gpuAtomicMul(at::Half * address, at::Half val) {
  return AtomicFPOp<at::Half>()(address, val,
                                [](at::Half bsum, at::Half val) {
                                  return bsum * val;
                                });
}

inline __device__ at::BFloat16 gpuAtomicMul(at::BFloat16 * address, at::BFloat16 val) {
  return AtomicFPOp<at::BFloat16>()(address, val,
```
- EN: Focus symbols: `ATOMIC_INTEGER_IMPL`, `GPU_ATOMIC_INTEGER`, `gpuAtomicMul`. This block declares or implements callable behavior for this module. Inputs are translated into lower-level operations, state updates, or returned values here.
- CN: 关注符号：`ATOMIC_INTEGER_IMPL`, `GPU_ATOMIC_INTEGER`, `gpuAtomicMul`。该代码块声明或实现该模块的可调用行为。这里把输入转换为更底层操作、状态更新或返回值。

### Lines 377-394
```cpp
                                    [](at::BFloat16 bsum, at::BFloat16 val) {
                                      return bsum * val;
                                    });
}

inline __device__ double gpuAtomicMul(double * address, double val) {
  return AtomicFPOp<double>()(address, val,
                              [](double val, unsigned long long int assumed) {
                                return __double_as_longlong(val * __longlong_as_double(assumed));
                              });
}

// Dont use a templated function for this since the addition function defaults to the CUDA built-in.
inline __device__ float gpuAtomicMul (float * address, float val) {
  unsigned int* address_as_ull = (unsigned int*)address;
  unsigned int old = *address_as_ull;
  unsigned int assumed;

```
- EN: Focus symbols: `gpuAtomicMul`, `__double_as_longlong`, `__longlong_as_double`. This block declares or implements callable behavior for this module. Inputs are translated into lower-level operations, state updates, or returned values here.
- CN: 关注符号：`gpuAtomicMul`, `__double_as_longlong`, `__longlong_as_double`。该代码块声明或实现该模块的可调用行为。这里把输入转换为更底层操作、状态更新或返回值。

### Lines 395-414
```cpp
  do {
    assumed = old;
    old = atomicCAS(address_as_ull, assumed,
                    __float_as_int(val *
                                   __int_as_float(assumed)));

    // Note: uses integer comparison to avoid hang in case of NaN (since NaN != NaN)
  } while (assumed != old);

  return __int_as_float(old);
}

// Atomic maximum implementation.

template <typename T>
__host__ __device__ T safe_max(T a, T b) {
  T max = at::_isnan(b) ? b : std::max<T>(a, b);
  return max;
}

```
- EN: Focus symbols: `atomicCAS`, `__float_as_int`, `__int_as_float`, `safe_max`, `_isnan`. This block builds generic/template machinery for multiple dtypes, devices, or call patterns. Templates let ATen share one design across many concrete instantiations.
- CN: 关注符号：`atomicCAS`, `__float_as_int`, `__int_as_float`, `safe_max`, `_isnan`。该代码块构建适配多种数据类型、设备或调用模式的模板机制。模板让 ATen 能在多个具体实例之间共享同一套设计。

### Lines 415-430
```cpp
ATOMIC_INTEGER_IMPL(Max)
GPU_ATOMIC_INTEGER(Max, safe_max(a, b), uint8_t)
GPU_ATOMIC_INTEGER(Max, safe_max(a, b), int8_t)
GPU_ATOMIC_INTEGER(Max, safe_max(a, b), int16_t)
GPU_ATOMIC_INTEGER(Max, safe_max(a, b), int32_t)
GPU_ATOMIC_INTEGER(Max, safe_max(a, b), int64_t)

inline __device__ at::Half gpuAtomicMax(at::Half * address, at::Half val) {
  return AtomicFPOp<at::Half>()(address, val,
                                [](at::Half bsum, at::Half val) {
                                  return safe_max(bsum, val);
                                });
}

inline __device__ at::BFloat16 gpuAtomicMax(at::BFloat16 * address, at::BFloat16 val) {
  return AtomicFPOp<at::BFloat16>()(address, val,
```
- EN: Focus symbols: `ATOMIC_INTEGER_IMPL`, `GPU_ATOMIC_INTEGER`, `safe_max`, `gpuAtomicMax`. This block declares or implements callable behavior for this module. Inputs are translated into lower-level operations, state updates, or returned values here.
- CN: 关注符号：`ATOMIC_INTEGER_IMPL`, `GPU_ATOMIC_INTEGER`, `safe_max`, `gpuAtomicMax`。该代码块声明或实现该模块的可调用行为。这里把输入转换为更底层操作、状态更新或返回值。

### Lines 431-448
```cpp
                                    [](at::BFloat16 bsum, at::BFloat16 val) {
                                      return safe_max(bsum, val);
                                    });
}

inline __device__ double gpuAtomicMax(double * address, double val) {
  return AtomicFPOp<double>()(address, val,
                              [](double val, unsigned long long int assumed) {
                                return __double_as_longlong(safe_max(val, __longlong_as_double(assumed)));
                              });
}

// Dont use a templated function for this since the addition function defaults to the CUDA built-in.
inline __device__ float gpuAtomicMax(float * address, float val) {
  unsigned int* address_as_ull = (unsigned int*)address;
  unsigned int old = *address_as_ull;
  unsigned int assumed;

```
- EN: Focus symbols: `safe_max`, `gpuAtomicMax`, `__double_as_longlong`, `__longlong_as_double`. This block declares or implements callable behavior for this module. Inputs are translated into lower-level operations, state updates, or returned values here.
- CN: 关注符号：`safe_max`, `gpuAtomicMax`, `__double_as_longlong`, `__longlong_as_double`。该代码块声明或实现该模块的可调用行为。这里把输入转换为更底层操作、状态更新或返回值。

### Lines 449-467
```cpp
  do {
    assumed = old;
    old = atomicCAS(address_as_ull, assumed,
                    __float_as_int(safe_max(val, __int_as_float(assumed))));

    // Note: uses integer comparison to avoid hang in case of NaN (since NaN != NaN)
  } while (assumed != old);

  return __int_as_float(old);
}

// Atomic minimum implementation.

template <typename T>
__host__ __device__ T safe_min(T a, T b) {
  T min = at::_isnan(b) ? b : std::min<T>(a, b);
  return min;
}

```
- EN: Focus symbols: `atomicCAS`, `__float_as_int`, `safe_max`, `__int_as_float`, `safe_min`. This block builds generic/template machinery for multiple dtypes, devices, or call patterns. Templates let ATen share one design across many concrete instantiations.
- CN: 关注符号：`atomicCAS`, `__float_as_int`, `safe_max`, `__int_as_float`, `safe_min`。该代码块构建适配多种数据类型、设备或调用模式的模板机制。模板让 ATen 能在多个具体实例之间共享同一套设计。

### Lines 468-483
```cpp
ATOMIC_INTEGER_IMPL(Min)
GPU_ATOMIC_INTEGER(Min, safe_min(a, b), uint8_t)
GPU_ATOMIC_INTEGER(Min, safe_min(a, b), int8_t)
GPU_ATOMIC_INTEGER(Min, safe_min(a, b), int16_t)
GPU_ATOMIC_INTEGER(Min, safe_min(a, b), int32_t)
GPU_ATOMIC_INTEGER(Min, safe_min(a, b), int64_t)

inline __device__ at::Half gpuAtomicMin(at::Half * address, at::Half val) {
  return AtomicFPOp<at::Half>()(address, val,
                                [](at::Half bsum, at::Half val) {
                                  return safe_min(bsum, val);
                                });
}

inline __device__ at::BFloat16 gpuAtomicMin(at::BFloat16 * address, at::BFloat16 val) {
  return AtomicFPOp<at::BFloat16>()(address, val,
```
- EN: Focus symbols: `ATOMIC_INTEGER_IMPL`, `GPU_ATOMIC_INTEGER`, `safe_min`, `gpuAtomicMin`. This block declares or implements callable behavior for this module. Inputs are translated into lower-level operations, state updates, or returned values here.
- CN: 关注符号：`ATOMIC_INTEGER_IMPL`, `GPU_ATOMIC_INTEGER`, `safe_min`, `gpuAtomicMin`。该代码块声明或实现该模块的可调用行为。这里把输入转换为更底层操作、状态更新或返回值。

### Lines 484-501
```cpp
                                    [](at::BFloat16 bsum, at::BFloat16 val) {
                                      return safe_min(bsum, val);
                                    });
}

inline __device__ double gpuAtomicMin(double * address, double val) {
  return AtomicFPOp<double>()(address, val,
                              [](double val, unsigned long long int assumed) {
                                return __double_as_longlong(safe_min(val, __longlong_as_double(assumed)));
                              });
}

// Dont use a templated function for this since the addition function defaults to the CUDA built-in.
inline __device__ float gpuAtomicMin(float * address, float val) {
  unsigned int* address_as_ull = (unsigned int*)address;
  unsigned int old = *address_as_ull;
  unsigned int assumed;

```
- EN: Focus symbols: `safe_min`, `gpuAtomicMin`, `__double_as_longlong`, `__longlong_as_double`. This block declares or implements callable behavior for this module. Inputs are translated into lower-level operations, state updates, or returned values here.
- CN: 关注符号：`safe_min`, `gpuAtomicMin`, `__double_as_longlong`, `__longlong_as_double`。该代码块声明或实现该模块的可调用行为。这里把输入转换为更底层操作、状态更新或返回值。

### Lines 502-511
```cpp
  do {
    assumed = old;
    old = atomicCAS(address_as_ull, assumed,
                    __float_as_int(safe_min(val, __int_as_float(assumed))));

    // Note: uses integer comparison to avoid hang in case of NaN (since NaN != NaN)
  } while (assumed != old);

  return __int_as_float(old);
}
```
- EN: Focus symbols: `atomicCAS`, `__float_as_int`, `safe_min`, `__int_as_float`. This block declares or implements callable behavior for this module. Inputs are translated into lower-level operations, state updates, or returned values here.
- CN: 关注符号：`atomicCAS`, `__float_as_int`, `safe_min`, `__int_as_float`。该代码块声明或实现该模块的可调用行为。这里把输入转换为更底层操作、状态更新或返回值。

## Key Concepts / 关键概念
- CUDA backend support / CUDA 后端支持
- C++ templates and specialization / C++ 模板与特化
- Conditional compilation / 条件编译
- Type-system design / 类型系统设计
- ATen/C10 internal layering / ATen/C10 内部分层

## Dependencies / 依赖关系
- Direct internal includes / 直接内部依赖: `c10/util/Half.h`, `c10/util/BFloat16.h`, `ATen/NumericUtils.h`
- External/system includes / 外部或系统头: `cuda.h`, `cuda_bf16.h`
- Inferred semantic dependencies / 推断出的语义依赖: C++ templates / C++ 模板
