# quick_all_reduce_base.h — Code Analysis / 代码分析

## Source / 来源
- **File**: `sgl-kernel/csrc/allreduce/quick_all_reduce_base.h`
- **Repository**: `sgl-project/sglang`
- **Purpose**: Handles multi-GPU all-reduce communication kernels and their supporting runtime logic. This header primarily defines interfaces, templates, constants, and inline helpers shared by compilation units. / 处理多 GPU all-reduce 通信内核及其配套运行时逻辑。 该头文件主要定义编译单元共享的接口、模板、常量和内联辅助逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-20: Headers and compile-time setup
```cpp
#pragma once

#include <hip/hip_bf16.h>
#include <hip/hip_fp16.h>
#include <hip/hip_runtime.h>

#include <cstdint>

#define __quickreduce_device_inline__ __device__ __forceinline__
#define __quickreduce_launch_bounds_two_shot__ __launch_bounds__(256, 4)
#define __quickreduce_launch_bounds_one_shot__ __launch_bounds__(512, 4)

namespace quickreduce {

typedef __hip_bfloat16 nv_bfloat16;
typedef __hip_bfloat162 nv_bfloat162;

using int32x2_t = __attribute__((__vector_size__(2 * sizeof(int)))) int;
using int32x4_t = __attribute__((__vector_size__(4 * sizeof(int)))) int;
```
**EN:** This section prepares the file's headers, macros, and compile-time switches so later code can rely on the right platform and API definitions.
**CN:** 本段准备文件依赖的头文件、宏和编译期开关，使后续代码能够依赖正确的平台与 API 定义。

### Lines 21-46: Templates, aliases, and constants
```cpp
// Setup acquire-release semantics for vector memory reads (mubuf instruction)
// as per architecture.
#if defined(__gfx942__)
// CDNA3: Scope bits sc0, sc1
#define MUBUF_ACQUIRE 16
#define MUBUF_RELEASE 16
#elif (defined(__gfx908__) || defined(__gfx90a__))
// CDNA1 and CDNA2 - glc bit
#define MUBUF_ACQUIRE 1
#define MUBUF_RELEASE 0
#endif

static constexpr int kNegOne = 0xBC00BC00;  // {-1, -1}, fp16x2_t

// Number of atoms (4xf16x2_t) processed by a single thread
static constexpr int kAtoms = 8;

// We use a workgroup of 256 threads
static constexpr int kBlockSize = 256;
static constexpr int kAtomStride = kBlockSize;

// Size and atom stride of source/destination data that the block will
// process.
// Workgroup scope = Tile = (256 threads x 8 atoms x 16B)
static constexpr int kTileSize = kBlockSize * kAtoms * sizeof(int32x4_t);
```
**EN:** This section defines `kNegOne`, `kAtoms`, `kBlockSize`, giving the file reusable compile-time abstractions, aliases, and specialization points.
**CN:** 本段定义了`kNegOne`、`kAtoms`、`kBlockSize`等内容，为文件提供可复用的编译期抽象、类型别名和特化入口。

### Lines 47-70: Templates, aliases, and constants
```cpp
// Max number of blocks. 304 CUs on MI300
static constexpr int kMaxNumBlocks = 304 * 4;

// Standard CDNA wavefront size.
static constexpr int kWavefront = 64;

// 256 thread, 4 wavefronts.
static dim3 constexpr kBlockTwoShot = {kWavefront, kBlockSize / kWavefront, 1};

// Number of threads in a group for quantization
// It corresponds to 32 F16 elements in quantization block
static constexpr int kThreadGroupSize = 8;

// Methods
__quickreduce_device_inline__ __host__ unsigned long divceil(unsigned long x, unsigned long y) {
  return ((x + y - 1) / y);
}

union BufferResource {
  __quickreduce_device_inline__ constexpr BufferResource() : config(0x00020000U) {}

  __quickreduce_device_inline__ constexpr BufferResource(void* buffer_address, uint32_t buffer_size)
      : address(buffer_address), range(buffer_size), config(0x00020000U) {}
```
**EN:** This section defines `divceil`, `BufferResource`, `kMaxNumBlocks`, giving the file reusable compile-time abstractions, aliases, and specialization points.
**CN:** 本段定义了`divceil`、`BufferResource`、`kMaxNumBlocks`等内容，为文件提供可复用的编译期抽象、类型别名和特化入口。

### Lines 71-95: Types and data layout
```cpp
  int32x4_t descriptor;
  struct {
    void* address;  // 8B, out of which first 48b is address, and 16b is stride
    // (unused)
    uint32_t range;   // Byte range for the buffer resource
    uint32_t config;  // Constant, DFMT=32b
  };
};

__quickreduce_device_inline__ static int32x4_t buffer_load_dwordx4(
    int32x4_t srsrc, int32_t voffset, int32_t soffset, int32_t aux) __asm("llvm.amdgcn.raw.buffer.load.v4i32");

__quickreduce_device_inline__ static void
buffer_store_dwordx4(int32x4_t data, int32x4_t srsrc, int32_t voffset, int32_t soffset, int32_t aux) __asm(
    "llvm.amdgcn.raw.buffer.store.v4i32");

__quickreduce_device_inline__ static void set_fp16_ovfl(bool const value) {
#if defined(__gfx942__)
  if (value) {
    asm volatile("s_setreg_imm32_b32 0xdc1, 1;" ::);
  } else {
    asm volatile("s_setreg_imm32_b32 0xdc1, 0;" ::);
  }
#endif
}
```
**EN:** This section defines `set_fp16_ovfl`, `defined`, `buffer_load_dwordx4`, packaging state, metadata, or memory layout information that later kernels and wrappers consume.
**CN:** 本段定义了`set_fp16_ovfl`、`defined`、`buffer_load_dwordx4`等内容，用于封装后续内核与封装层会使用的状态、元数据或内存布局信息。

### Lines 96-122: Device helpers and synchronization
```cpp
union bf162_int_union {
  int i;
  nv_bfloat162 bf2;
};

template <typename T>
__quickreduce_device_inline__ void packed_assign_add(int32x4_t* A, int32x4_t* B);

template <>
__quickreduce_device_inline__ void packed_assign_add<half>(int32x4_t* A, int32x4_t* B) {
  int32x4_t& tR_fragment = A[0];
  int32x4_t& tA_fragment = B[0];

  asm volatile("v_pk_add_f16 %0, %1, %2" : "=v"(tR_fragment[0]) : "v"(tR_fragment[0]), "v"(tA_fragment[0]));
  asm volatile("v_pk_add_f16 %0, %1, %2" : "=v"(tR_fragment[1]) : "v"(tR_fragment[1]), "v"(tA_fragment[1]));
  asm volatile("v_pk_add_f16 %0, %1, %2" : "=v"(tR_fragment[2]) : "v"(tR_fragment[2]), "v"(tA_fragment[2]));
  asm volatile("v_pk_add_f16 %0, %1, %2" : "=v"(tR_fragment[3]) : "v"(tR_fragment[3]), "v"(tA_fragment[3]));
}

template <>
__quickreduce_device_inline__ void packed_assign_add<nv_bfloat16>(int32x4_t* A, int32x4_t* B) {
  nv_bfloat162* tA = reinterpret_cast<nv_bfloat162*>(A);
  nv_bfloat162* tB = reinterpret_cast<nv_bfloat162*>(B);
#pragma unroll
  for (int i = 0; i < 4; i++) {
    tA[i] = __hadd2(tA[i], tB[i]);
  }
```
**EN:** This section implements `packed_assign_add`, `volatile`, `__hadd2`, low-level device-side helpers for arithmetic, synchronization, communication, or memory movement.
**CN:** 本段实现了`packed_assign_add`、`volatile`、`__hadd2`等底层设备侧辅助逻辑，用于算术处理、同步、通信或内存搬运。

### Lines 123-146: Device helpers and synchronization
```cpp
}

template <typename T>
__quickreduce_device_inline__ int packed_max(int a, int b);

template <>
__quickreduce_device_inline__ int packed_max<half>(int a, int b) {
  int result;
  asm volatile("v_pk_max_f16 %0, %1, %2" : "=v"(result) : "v"(a), "v"(b));
  return result;
}

template <>
__quickreduce_device_inline__ int packed_max<nv_bfloat16>(int a, int b) {
  bf162_int_union A, B, R;
  A.i = a;
  B.i = b;
  R.bf2 = __hmax2(A.bf2, B.bf2);
  return R.i;
}

template <typename T>
__quickreduce_device_inline__ int packed_min(int a, int b);
```
**EN:** This section implements `packed_max`, `volatile`, `__hmax2`, low-level device-side helpers for arithmetic, synchronization, communication, or memory movement.
**CN:** 本段实现了`packed_max`、`volatile`、`__hmax2`等底层设备侧辅助逻辑，用于算术处理、同步、通信或内存搬运。

### Lines 147-171: Device helpers and synchronization
```cpp
template <>
__quickreduce_device_inline__ int packed_min<half>(int a, int b) {
  int result;
  asm volatile("v_pk_min_f16 %0, %1, %2" : "=v"(result) : "v"(a), "v"(b));
  return result;
}

template <>
__quickreduce_device_inline__ int packed_min<nv_bfloat16>(int a, int b) {
  bf162_int_union A, B, R;
  A.i = a;
  B.i = b;
  R.bf2 = __hmin2(A.bf2, B.bf2);
  return R.i;
}

template <typename T>
__quickreduce_device_inline__ int packed_abs_max(int a, int b);

template <>
__quickreduce_device_inline__ int packed_abs_max<half>(int a, int b) {
  half2 wmaxh2 = __builtin_bit_cast(half2, a);
  half2 wminh2 = __builtin_bit_cast(half2, b);
  half2 wblockmaxh2;
```
**EN:** This section implements `volatile`, `__hmin2`, `packed_abs_max`, low-level device-side helpers for arithmetic, synchronization, communication, or memory movement.
**CN:** 本段实现了`volatile`、`__hmin2`、`packed_abs_max`等底层设备侧辅助逻辑，用于算术处理、同步、通信或内存搬运。

### Lines 172-195: Device helpers and synchronization
```cpp
  wblockmaxh2.x = __hgt(__habs(wmaxh2.x), __habs(wminh2.x)) ? wmaxh2.x : wminh2.x;
  wblockmaxh2.y = __hgt(__habs(wmaxh2.y), __habs(wminh2.y)) ? wmaxh2.y : wminh2.y;
  return __builtin_bit_cast(int, wblockmaxh2);
}

template <>
__quickreduce_device_inline__ int packed_abs_max<nv_bfloat16>(int a, int b) {
  bf162_int_union A, B, R;
  A.i = a;
  B.i = b;
  R.bf2.x = __hgt(__habs(A.bf2.x), __habs(B.bf2.x)) ? A.bf2.x : B.bf2.x;
  R.bf2.y = __hgt(__habs(A.bf2.y), __habs(B.bf2.y)) ? A.bf2.y : B.bf2.y;
  return R.i;
}

template <typename T>
__quickreduce_device_inline__ int packed_add(int a, int b);

template <>
__quickreduce_device_inline__ int packed_add<half>(int a, int b) {
  int result;
  asm volatile("v_pk_add_f16 %0, %1, %2" : "=v"(result) : "v"(a), "v"(b));
  return result;
}
```
**EN:** This section implements `__builtin_bit_cast`, `packed_add`, `volatile`, low-level device-side helpers for arithmetic, synchronization, communication, or memory movement.
**CN:** 本段实现了`__builtin_bit_cast`、`packed_add`、`volatile`等底层设备侧辅助逻辑，用于算术处理、同步、通信或内存搬运。

### Lines 196-219: Device helpers and synchronization
```cpp

template <>
__quickreduce_device_inline__ int packed_add<nv_bfloat16>(int a, int b) {
  bf162_int_union A, B, R;
  A.i = a;
  B.i = b;
  R.bf2 = __hadd2(A.bf2, B.bf2);
  return R.i;
}

template <>
__quickreduce_device_inline__ int packed_add<int16_t>(int a, int b) {
  int result;
  asm volatile("v_pk_add_i16 %0, %1, %2" : "=v"(result) : "v"(a), "v"(b));
  return result;
}

template <typename T>
__quickreduce_device_inline__ int packed_sub(int a, int b);

template <>
__quickreduce_device_inline__ int packed_sub<half>(int a, int b) {
  int result;
```
**EN:** This section implements `__hadd2`, `volatile`, `packed_sub`, low-level device-side helpers for arithmetic, synchronization, communication, or memory movement.
**CN:** 本段实现了`__hadd2`、`volatile`、`packed_sub`等底层设备侧辅助逻辑，用于算术处理、同步、通信或内存搬运。

### Lines 220-243: Device helpers and synchronization
```cpp
  // MI300 lacks packed fp16 sub instruction. So we do -1 * min + max
  asm volatile("v_pk_fma_f16 %0, %1, %2 %3" : "=v"(result) : "v"(kNegOne), "v"(b), "v"(a));
  return result;
}

template <>
__quickreduce_device_inline__ int packed_sub<nv_bfloat16>(int a, int b) {
  bf162_int_union A, B, R;
  A.i = a;
  B.i = b;
  R.bf2 = __hsub2(A.bf2, B.bf2);
  return R.i;
}

template <typename T>
__quickreduce_device_inline__ int packed_mul(int a, int b);

template <>
__quickreduce_device_inline__ int packed_mul<half>(int a, int b) {
  int result;
  asm volatile("v_pk_mul_f16 %0, %1, %2" : "=v"(result) : "v"(a), "v"(b));
  return result;
}
```
**EN:** This section implements `volatile`, `__hsub2`, `packed_mul`, low-level device-side helpers for arithmetic, synchronization, communication, or memory movement.
**CN:** 本段实现了`volatile`、`__hsub2`、`packed_mul`等底层设备侧辅助逻辑，用于算术处理、同步、通信或内存搬运。

### Lines 244-267: Templates, aliases, and constants
```cpp
template <>
__quickreduce_device_inline__ int packed_mul<nv_bfloat16>(int a, int b) {
  nv_bfloat162* tA = reinterpret_cast<nv_bfloat162*>(&a);
  nv_bfloat162* tB = reinterpret_cast<nv_bfloat162*>(&b);
  nv_bfloat162 tR = __hmul2(*tA, *tB);
  return *(reinterpret_cast<int*>(&tR));
}

template <typename T>
__quickreduce_device_inline__ int packed_rcp(int a);

template <>
__quickreduce_device_inline__ int packed_rcp<half>(int a) {
  return __builtin_bit_cast(int, h2rcp(__builtin_bit_cast(half2, a)));
}

template <>
__quickreduce_device_inline__ int packed_rcp<nv_bfloat16>(int a) {
  bf162_int_union A, R;
  A.i = a;
  R.bf2 = h2rcp(A.bf2);
  return R.i;
}
```
**EN:** This section defines `__hmul2`, `packed_rcp`, `__builtin_bit_cast`, giving the file reusable compile-time abstractions, aliases, and specialization points.
**CN:** 本段定义了`__hmul2`、`packed_rcp`、`__builtin_bit_cast`等内容，为文件提供可复用的编译期抽象、类型别名和特化入口。

### Lines 268-292: Device helpers and synchronization
```cpp
// changes dtype
__quickreduce_device_inline__ float T2float_cast(half a) {
  return __half2float(a);
}

__quickreduce_device_inline__ float T2float_cast(nv_bfloat16 a) {
  return __bfloat162float(a);
}

template <typename T>
__quickreduce_device_inline__ int group_abs_max(int32x4_t atom) {
  const int group_leader = (threadIdx.x / kThreadGroupSize) * kThreadGroupSize;

  int wmax, wmin, wblockmax;
  int a, b;
  a = packed_max<T>(atom[0], atom[1]);
  b = packed_max<T>(atom[2], atom[3]);

  wmax = packed_max<T>(a, b);

  a = packed_min<T>(atom[0], atom[1]);
  b = packed_min<T>(atom[2], atom[3]);

  wmin = packed_min<T>(a, b);
```
**EN:** This section implements `T2float_cast`, `group_abs_max`, `__half2float`, low-level device-side helpers for arithmetic, synchronization, communication, or memory movement.
**CN:** 本段实现了`T2float_cast`、`group_abs_max`、`__half2float`等底层设备侧辅助逻辑，用于算术处理、同步、通信或内存搬运。

### Lines 293-316: Device helpers and synchronization
```cpp
  // Reduce the max among a group of threads
  // Note: This is basically 2 blocks of values setup as the
  // upper/lower halves of the f16x2_t
  for (int i = 1; i < kThreadGroupSize; i <<= 1) {
    int x = __shfl_down(wmax, i);
    wmax = packed_max<T>(wmax, x);

    int y = __shfl_down(wmin, i);
    wmin = packed_min<T>(wmin, y);
  }
  wblockmax = packed_abs_max<T>(wmax, wmin);
  // Share with the cohort
  wblockmax = __shfl(wblockmax, group_leader);
  return wblockmax;
}

__quickreduce_device_inline__ void set_sync_flag(uint32_t* flag_ptr, uint32_t flag) {
  __atomic_store_n(flag_ptr, flag, __ATOMIC_RELEASE);
}

__quickreduce_device_inline__ void wait_sync_flag(uint32_t* flag_ptr, uint32_t flag) {
  while (__atomic_load_n(flag_ptr, __ATOMIC_RELAXED) != flag) {
  }
}
```
**EN:** This section implements `set_sync_flag`, `wait_sync_flag`, `__shfl_down`, low-level device-side helpers for arithmetic, synchronization, communication, or memory movement.
**CN:** 本段实现了`set_sync_flag`、`wait_sync_flag`、`__shfl_down`等底层设备侧辅助逻辑，用于算术处理、同步、通信或内存搬运。

### Lines 317-317: Local implementation details
```cpp

```
**EN:** This section fills in the local implementation details around the surrounding logic, completing the behavior required by the file.
**CN:** 本段补充了相关逻辑周边的局部实现细节，以完成该文件所需的具体行为。

### Lines 318-318: Local implementation details
```cpp
}  // namespace quickreduce
```
**EN:** This section fills in the local implementation details around the surrounding logic, completing the behavior required by the file.
**CN:** 本段补充了相关逻辑周边的局部实现细节，以完成该文件所需的具体行为。

## Key Concepts / 关键概念
- **Device helpers / 设备侧辅助函数**: Uses inline device routines to structure low-level operations. / 使用内联设备函数组织底层操作。
- **Template specialization / 模板特化**: Relies on C++ templates to specialize code paths by type or shape. / 依赖 C++ 模板按类型或形状特化代码路径。
- **HIP runtime / HIP 运行时**: Uses HIP APIs or AMD-specific intrinsics for portability. / 使用 HIP API 或 AMD 特定 intrinsic 实现可移植性。
- **Quantization / 量化**: Handles low-precision representations and conversion logic. / 处理低精度表示及其转换逻辑。
- **Collective communication / 集合通信**: Coordinates reductions across devices or ranks. / 在设备或 rank 之间协调归约。
- **Synchronization / atomics / 同步与原子操作**: Uses atomics or explicit synchronization to coordinate parallel work. / 使用原子操作或显式同步协调并行工作。

## Dependencies / 依赖关系
- **Internal headers / 内部头文件**: None explicitly listed / 未显式列出
- **External headers / 外部头文件**: `hip/hip_bf16.h`, `hip/hip_fp16.h`, `hip/hip_runtime.h`, `cstdint`
- **Path context / 路径上下文**: allreduce / quick_all_reduce_base.h
