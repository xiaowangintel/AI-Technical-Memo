# quick_all_reduce.cuh — Code Analysis / 代码分析

## Source / 来源
- **File**: `sgl-kernel/csrc/allreduce/quick_all_reduce.cuh`
- **Repository**: `sgl-project/sglang`
- **Purpose**: Handles multi-GPU all-reduce communication kernels and their supporting runtime logic. This header primarily defines interfaces, templates, constants, and inline helpers shared by compilation units. / 处理多 GPU all-reduce 通信内核及其配套运行时逻辑。 该头文件主要定义编译单元共享的接口、模板、常量和内联辅助逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-24: Namespace and shared declarations
```cpp
#pragma once

#include <hip/hip_runtime.h>

#include "quick_all_reduce_base.h"

namespace quickreduce {

struct CodecBase {
  const int thread;
  const int rank;
  const int group_leader;
  __quickreduce_device_inline__ CodecBase(int thread, int rank)
      : thread(thread), rank(rank), group_leader((threadIdx.x / kThreadGroupSize) * kThreadGroupSize) {
    set_fp16_ovfl(true);
  }
};

// Default full precision codec.
template <typename T, int world_size>
struct CodecFP : public CodecBase {
  static constexpr int kWorldSize = world_size;
  static constexpr int kRankAtoms = kAtoms / kWorldSize;
```
**EN:** This section organizes shared declarations under namespace scope, keeping related symbols together and reducing naming conflicts.
**CN:** 本段把共享声明组织在命名空间作用域下，使相关符号保持聚合并减少命名冲突。

### Lines 25-48: Templates, aliases, and constants
```cpp
  // Codec tile size process by this workgroup.
  // Each thread processes atoms of f16x8_t (16B).
  static constexpr int kRankTransmittedTileSize = kBlockSize * kRankAtoms * sizeof(int32x4_t);
  static_assert(kRankTransmittedTileSize % 16 == 0, "kRankTransmittedTileSize must be 16B aligned.");

  // Total tile size for the collective communication.
  static constexpr int kTransmittedTileSize = kRankTransmittedTileSize * kWorldSize;

  __quickreduce_device_inline__ CodecFP(int thread, int rank) : CodecBase(thread, rank) {}

  __quickreduce_device_inline__ void send(int32x4_t* __restrict__ send_buffer, const int32x4_t* __restrict__ data) {
    for (int i = 0; i < kRankAtoms; i++) {
      __builtin_nontemporal_store(data[i], send_buffer + thread);
      send_buffer += kAtomStride;
    }
  }

  __quickreduce_device_inline__ void recv(int32x4_t** __restrict__ recv_buffer, int32x4_t* __restrict__ data) {
    for (int i = 0; i < kRankAtoms; i++) {
      data[i] = __builtin_nontemporal_load(*recv_buffer + thread);
      *recv_buffer += kAtomStride;
    }
  }
};
```
**EN:** This section defines `CodecFP`, `send`, `recv`, giving the file reusable compile-time abstractions, aliases, and specialization points.
**CN:** 本段定义了`CodecFP`、`send`、`recv`等内容，为文件提供可复用的编译期抽象、类型别名和特化入口。

### Lines 49-72: Types and data layout
```cpp

// Int4 symmetric quantization codec.
// We quantize the FP16 data to block-scaled Int4 in blocks of 4 *
// kThreadGroupSize.
template <typename T, int world_size>
struct CodecQ4 : public CodecBase {
  static constexpr int kWorldSize = world_size;

  // Codec tile size process by this workgroup.
  // Each threads processes a fragment of fp16x8_t (16B),
  // into a int4x8_t (4B) and a fp16 scale shared among 32 values.
  static constexpr int kRankAtoms = kAtoms / kWorldSize;
  static constexpr int kRankTileStride = 1152;
  static constexpr int kRankTileScaleOffset = 1024;
  static constexpr int kRankTransmittedTileSize = kRankTileStride * kRankAtoms;
  static_assert(kRankTransmittedTileSize % 16 == 0, "kRankTransmittedTileSize must be 16B aligned.");

  static constexpr int kRankBufferTileStride = kRankTileStride / sizeof(int32x4_t);

  // Total tile size for the collective communication.
  static constexpr int kTransmittedTileSize = kRankTransmittedTileSize * kWorldSize;

  // Constants configuration
```
**EN:** This section defines `CodecQ4`, `static_assert`, `kWorldSize`, packaging state, metadata, or memory layout information that later kernels and wrappers consume.
**CN:** 本段定义了`CodecQ4`、`static_assert`、`kWorldSize`等内容，用于封装后续内核与封装层会使用的状态、元数据或内存布局信息。

### Lines 73-97: Templates, aliases, and constants
```cpp
  // {-1/8.0h, -1/8.0h}, f16x2_t
  static constexpr int kScaleFactor = std::is_same<T, half>::value ? 0xB000B000 : 0xBE00BE00;

  // {1e-7, 1e-7}, f16x2_t
  static constexpr int kScaleEpsilon = std::is_same<T, half>::value ? 0x00010001 : 0x33D733D7;

  // {-8, -8}, f16x2_t
  static constexpr int kRangeMin = std::is_same<T, half>::value ? 0xC800C800 : 0xC100C100;

  // {+7, +7}, f16x2_t
  static constexpr int kRangeMax = std::is_same<T, half>::value ? 0x47004700 : 0x40E040E0;

  // {+8, +8}, int16x2_t
  static constexpr int kRangeBias = 0x00080008;

  __quickreduce_device_inline__ CodecQ4(int thread, int rank) : CodecBase(thread, rank) {}

  __quickreduce_device_inline__ void send(int32x4_t* __restrict__ send_buffer, const int32x4_t* __restrict__ data) {
    for (int k = 0; k < kRankAtoms; k++) {
      int32x4_t const atom = data[k];

      // Compute the absolute maximum of the atom in the thread group
      // In 2 blocks of values, upper/lower halves of the f16x2_t
      int wblockmax = group_abs_max<T>(atom);
```
**EN:** This section defines `CodecQ4`, `send`, `kScaleFactor`, giving the file reusable compile-time abstractions, aliases, and specialization points.
**CN:** 本段定义了`CodecQ4`、`send`、`kScaleFactor`等内容，为文件提供可复用的编译期抽象、类型别名和特化入口。

### Lines 98-120: Control flow and branching
```cpp
      // Derive scales
      int decoding_scale;
      int encoding_scale;
      decoding_scale = packed_mul<T>(wblockmax, kScaleFactor);
      encoding_scale = packed_add<T>(decoding_scale, kScaleEpsilon);
      encoding_scale = packed_rcp<T>(encoding_scale);

      // Apply scales to get quantized values
      int32x4_t w;
      for (int i = 0; i < 4; i++) {
        w[i] = packed_mul<T>(atom[i], encoding_scale);
        w[i] = packed_max<T>(w[i], kRangeMin);
        w[i] = packed_min<T>(w[i], kRangeMax);
      }

      // Convert from f16x2_t to uint16x2_t
      int32x4_t q;
      {
        int16_t* qi = reinterpret_cast<int16_t*>(&q);
        T* wh = reinterpret_cast<T*>(&w);
        for (int i = 0; i < 8; i++)
          qi[i] = (int16_t)rintf(T2float_cast(wh[i]));
```
**EN:** This section drives `rintf` through branches and loops, deciding which execution path or data-handling strategy is used.
**CN:** 本段通过分支和循环推进`rintf`相关逻辑，决定采用哪条执行路径或数据处理策略。

### Lines 121-141: Device helpers and synchronization
```cpp
        for (int i = 0; i < 4; i++) {
          q[i] = packed_add<int16_t>(q[i], kRangeBias);
        }
      }

      // Pack 8 x q4 into int32_t
      int qw = q[0] | (q[1] << 4) | (q[2] << 8) | (q[3] << 12);

      // Write quantized atom to send_buffer
      // note: only the group leader stores the scale
      uint8_t* atom_ptr = reinterpret_cast<uint8_t*>(send_buffer + k * kRankBufferTileStride);
      int32_t* qw_ptr = reinterpret_cast<int32_t*>(atom_ptr) + thread;
      int* qs_ptr = reinterpret_cast<int*>(atom_ptr + kRankTileScaleOffset) + (thread / 8);

      __builtin_nontemporal_store(qw, qw_ptr);
      if (threadIdx.x == group_leader) {
        __builtin_nontemporal_store(decoding_scale, qs_ptr);
      }
    }
  }
```
**EN:** This section implements `__builtin_nontemporal_store`, low-level device-side helpers for arithmetic, synchronization, communication, or memory movement.
**CN:** 本段实现了`__builtin_nontemporal_store`等底层设备侧辅助逻辑，用于算术处理、同步、通信或内存搬运。

### Lines 142-160: Templates, aliases, and constants
```cpp
  __quickreduce_device_inline__ void recv(int32x4_t** __restrict__ recv_buffer, int32x4_t* __restrict__ data) {
    for (int k = 0; k < kRankAtoms; k++) {
      // Directly read quantized atom from recv_buffer
      uint8_t* atom_ptr = reinterpret_cast<uint8_t*>(*recv_buffer);
      int32_t* qw_ptr = reinterpret_cast<int32_t*>(atom_ptr) + thread;
      int* qs_ptr = reinterpret_cast<int*>(atom_ptr + kRankTileScaleOffset) + (thread / 8);

      int32_t qw = __builtin_nontemporal_load(qw_ptr);
      int qs = __builtin_nontemporal_load(qs_ptr);

      *recv_buffer += kRankBufferTileStride;

      // Unpack q4 into f16x8_t
      int32x4_t w;
      {
        static constexpr uint kMask000F = 0x000F000F;
        static constexpr uint kHalf2_1024 = 0x64006400;  // {1024.0, 1024.0}, fp16x2_t
        static uint constexpr kHalf2_1032 = 0xE408E408;  // {-1032.0, -1032.0}, fp16x2_t
```
**EN:** This section defines `recv`, `__builtin_nontemporal_load`, `kMask000F`, giving the file reusable compile-time abstractions, aliases, and specialization points.
**CN:** 本段定义了`recv`、`__builtin_nontemporal_load`、`kMask000F`等内容，为文件提供可复用的编译期抽象、类型别名和特化入口。

### Lines 161-184: Templates, aliases, and constants
```cpp
        for (int i = 0; i < 4; i++) {
          if constexpr (std::is_same<T, half>::value) {
            int32_t q4 = ((qw >> (i * 4)) & kMask000F) | kHalf2_1024;
            w[i] = packed_add<half>(q4, kHalf2_1032);
          } else {
            int32_t int16_2 = (qw >> (i * 4)) & kMask000F;
            int16_t low = static_cast<int16_t>(int16_2 & 0xFFFF);
            int16_t high = static_cast<int16_t>((int16_2 >> 16) & 0xFFFF);
            nv_bfloat16 bf_low = __float2bfloat16(static_cast<float>(low));
            nv_bfloat16 bf_high = __float2bfloat16(static_cast<float>(high));
            nv_bfloat162 bf2 = __halves2bfloat162(bf_low, bf_high);
            int32_t packed_bf16 = *reinterpret_cast<int32_t*>(&bf2);
            w[i] = packed_add<nv_bfloat16>(packed_bf16, kRangeMin);
          }
        }
      }

      // Apply decoding scales
      for (int i = 0; i < 4; i++) {
        w[i] = packed_mul<T>(w[i], qs);
      }

      data[k] = w;
    }
```
**EN:** This section defines `__float2bfloat16`, `__halves2bfloat162`, giving the file reusable compile-time abstractions, aliases, and specialization points.
**CN:** 本段定义了`__float2bfloat16`、`__halves2bfloat162`等内容，为文件提供可复用的编译期抽象、类型别名和特化入口。

### Lines 185-209: Types and data layout
```cpp
  }
};

// Int6 symmetric quantization codec.
// We quantize the FP16 data to block-scaled Int6 in blocks of 4 *
// kThreadGroupSize.
template <typename T, int world_size>
struct CodecQ6 : public CodecBase {
  static constexpr int kWorldSize = world_size;

  // Codec tile size process by this workgroup.
  // Each threads processes a fragment of fp16x8_t (16B),
  // into a int6x8_t (4B + 2B) and a fp16 scale shared among 32 values.
  static constexpr int kRankAtoms = kAtoms / kWorldSize;
  static constexpr int kRankTileStride = 1664;
  static constexpr int kRankTileQ2Offset = 1024;
  static constexpr int kRankTileScaleOffset = 1536;
  static constexpr int kRankTransmittedTileSize = kRankTileStride * kRankAtoms;
  static_assert(kRankTransmittedTileSize % 16 == 0, "kRankTransmittedTileSize must be 16B aligned.");

  static constexpr int kRankBufferTileStride = kRankTileStride / sizeof(int32x4_t);

  // Total tile size for the collective communication.
  static constexpr int kTransmittedTileSize = kRankTransmittedTileSize * kWorldSize;
```
**EN:** This section defines `CodecQ6`, `static_assert`, `kWorldSize`, packaging state, metadata, or memory layout information that later kernels and wrappers consume.
**CN:** 本段定义了`CodecQ6`、`static_assert`、`kWorldSize`等内容，用于封装后续内核与封装层会使用的状态、元数据或内存布局信息。

### Lines 210-232: Templates, aliases, and constants
```cpp
  // Constants configuration

  // {-1/32.0h, -1/32.0h}, fp16x2_t
  static constexpr int kScaleFactor = std::is_same<T, half>::value ? 0xA800A800 : 0xBD00BD00;

  // {1e-7, 1e-7}, fp16x2_t
  static constexpr int kScaleEpsilon = std::is_same<T, half>::value ? 0x00010001 : 0x33D733D7;

  // {-32, -32}, fp16x2_t
  static constexpr int kRangeMin = std::is_same<T, half>::value ? 0xD000D000 : 0xC200C200;

  // {+31, +31}, fp16x2_t
  static constexpr int kRangeMax = std::is_same<T, half>::value ? 0x4FC04FC0 : 0x41F841F8;

  // {+32, +32}, int16x2_t
  static constexpr int kRangeBias = 0x00200020;

  __quickreduce_device_inline__ CodecQ6(int thread, int rank) : CodecBase(thread, rank) {}

  __quickreduce_device_inline__ void send(int32x4_t* __restrict__ send_buffer, const int32x4_t* __restrict__ data) {
    for (int k = 0; k < kRankAtoms; k++) {
      int32x4_t const atom = data[k];
```
**EN:** This section defines `CodecQ6`, `send`, `kScaleFactor`, giving the file reusable compile-time abstractions, aliases, and specialization points.
**CN:** 本段定义了`CodecQ6`、`send`、`kScaleFactor`等内容，为文件提供可复用的编译期抽象、类型别名和特化入口。

### Lines 233-259: Control flow and branching
```cpp
      // Compute the absolute maximum of the atom in the thread group
      // In 2 blocks of values, upper/lower halves of the f16x2_t
      int wblockmax = group_abs_max<T>(atom);

      // Derive scales
      int decoding_scale;
      int encoding_scale;
      decoding_scale = packed_mul<T>(wblockmax, kScaleFactor);
      encoding_scale = packed_add<T>(decoding_scale, kScaleEpsilon);
      encoding_scale = packed_rcp<T>(encoding_scale);

      // Apply scales to get quantized values
      int32x4_t w;
      for (int i = 0; i < 4; i++) {
        w[i] = packed_mul<T>(atom[i], encoding_scale);
        w[i] = packed_max<T>(w[i], kRangeMin);
        w[i] = packed_min<T>(w[i], kRangeMax);
      }

      // Convert from f16x2_t to uint16x2_t
      int32x4_t q;
      {
        int16_t* qi = reinterpret_cast<int16_t*>(&q);
        T* wh = reinterpret_cast<T*>(&w);
        for (int i = 0; i < 8; i++)
          qi[i] = (int16_t)rintf(T2float_cast(wh[i]));
```
**EN:** This section drives `rintf` through branches and loops, deciding which execution path or data-handling strategy is used.
**CN:** 本段通过分支和循环推进`rintf`相关逻辑，决定采用哪条执行路径或数据处理策略。

### Lines 260-282: Control flow and branching
```cpp
        for (int i = 0; i < 4; i++) {
          q[i] = packed_add<int16_t>(q[i], kRangeBias);
        }
      }

      // Pack 8 x q6 into int32_t + int16_t
      uint32_t q4w;
      uint16_t q2w = 0;
      q4w = (q[0] & 0x000F000F) | ((q[1] & 0x000F000F) << 4) | ((q[2] & 0x000F000F) << 8) | ((q[3] & 0x000F000F) << 12);
      {
        int16_t* tw = reinterpret_cast<int16_t*>(&q);
#pragma unroll
        for (int i = 0; i < 8; i++) {
          q2w |= (tw[i] >> 4) << (i * 2);
        }
      }
      // Write quantized atom to send_buffer
      // note: only the group leader stores the scale
      uint8_t* atom_ptr = reinterpret_cast<uint8_t*>(send_buffer + k * kRankBufferTileStride);
      uint32_t* q4w_ptr = reinterpret_cast<uint32_t*>(atom_ptr) + thread;
      uint16_t* q2w_ptr = reinterpret_cast<uint16_t*>(atom_ptr + kRankTileQ2Offset) + thread;
      int* qs_ptr = reinterpret_cast<int*>(atom_ptr + kRankTileScaleOffset) + (thread / 8);
```
**EN:** This section drives the surrounding logic through branches and loops, deciding which execution path or data-handling strategy is used.
**CN:** 本段通过分支和循环推进相关逻辑相关逻辑，决定采用哪条执行路径或数据处理策略。

### Lines 283-304: Device helpers and synchronization
```cpp
      __builtin_nontemporal_store(q4w, q4w_ptr);
      __builtin_nontemporal_store(q2w, q2w_ptr);
      if (threadIdx.x == group_leader) {
        __builtin_nontemporal_store(decoding_scale, qs_ptr);
      }
    }
  }

  __quickreduce_device_inline__ void recv(int32x4_t** __restrict__ recv_buffer, int32x4_t* __restrict__ data) {
    for (int k = 0; k < kRankAtoms; k++) {
      // Directly read quantized atom from recv_buffer
      uint8_t* atom_ptr = reinterpret_cast<uint8_t*>(*recv_buffer);
      uint32_t* q4w_ptr = reinterpret_cast<uint32_t*>(atom_ptr) + thread;
      uint16_t* q2w_ptr = reinterpret_cast<uint16_t*>(atom_ptr + kRankTileQ2Offset) + thread;
      int* qs_ptr = reinterpret_cast<int*>(atom_ptr + kRankTileScaleOffset) + (thread / 8);

      uint32_t q4w = __builtin_nontemporal_load(q4w_ptr);
      uint16_t q2w = __builtin_nontemporal_load(q2w_ptr);
      int qs = __builtin_nontemporal_load(qs_ptr);

      *recv_buffer += kRankBufferTileStride;
```
**EN:** This section implements `recv`, `__builtin_nontemporal_store`, `__builtin_nontemporal_load`, low-level device-side helpers for arithmetic, synchronization, communication, or memory movement.
**CN:** 本段实现了`recv`、`__builtin_nontemporal_store`、`__builtin_nontemporal_load`等底层设备侧辅助逻辑，用于算术处理、同步、通信或内存搬运。

### Lines 305-331: Device helpers and synchronization
```cpp
      // Unpack q6 into fp16x8_t
      int32x4_t w;
      {
        static uint constexpr kMask000F = 0x000F000F;
        static uint constexpr kHalf2_1024 = 0x64006400;  // {1024.0, 1024.0}, fp16x2_t
        static uint constexpr kHalf2_1056 = 0xE420E420;  // {-1056.0, -1056.0}, fp16x2_t

#pragma unroll
        for (int i = 0; i < 4; i++) {
          int32_t q4 = q4w & kMask000F;
          int32_t q2 = (q2w & 0x3) | ((q2w & 0xC) << 14);
          q4w >>= 4;
          q2w >>= 4;
          if constexpr (std::is_same<T, half>::value) {
            int32_t q6 = q4 | (q2 << 4) | kHalf2_1024;
            asm volatile("v_pk_add_f16 %0, %1, %2" : "=v"(w[i]) : "v"(q6), "v"(kHalf2_1056));
          } else {
            int32_t int16_2 = q4 | (q2 << 4);
            int16_t low = static_cast<int16_t>(int16_2 & 0xFFFF);
            int16_t high = static_cast<int16_t>((int16_2 >> 16) & 0xFFFF);

            nv_bfloat16 bf_low = __float2bfloat16(static_cast<float>(low));
            nv_bfloat16 bf_high = __float2bfloat16(static_cast<float>(high));
            nv_bfloat162 bf2 = __halves2bfloat162(bf_low, bf_high);
            int32_t packed_bf16 = *reinterpret_cast<int32_t*>(&bf2);
            w[i] = packed_add<nv_bfloat16>(packed_bf16, kRangeMin);
          }
```
**EN:** This section implements `volatile`, `__float2bfloat16`, `__halves2bfloat162`, low-level device-side helpers for arithmetic, synchronization, communication, or memory movement.
**CN:** 本段实现了`volatile`、`__float2bfloat16`、`__halves2bfloat162`等底层设备侧辅助逻辑，用于算术处理、同步、通信或内存搬运。

### Lines 332-352: Types and data layout
```cpp
        }
      }

      // Apply decoding scales
      for (int i = 0; i < 4; i++) {
        w[i] = packed_mul<T>(w[i], qs);
      }

      // That's pretty much it...
      data[k] = w;
    }
  }
};

// Int8 symmetric quantization codec.
// We quantize the FP16 data to block-scaled Int8 in blocks of 4 *
// kThreadGroupSize.
template <typename T, int world_size>
struct CodecQ8 : public CodecBase {
  static constexpr int kWorldSize = world_size;
```
**EN:** This section defines `CodecQ8`, `kWorldSize`, packaging state, metadata, or memory layout information that later kernels and wrappers consume.
**CN:** 本段定义了`CodecQ8`、`kWorldSize`等内容，用于封装后续内核与封装层会使用的状态、元数据或内存布局信息。

### Lines 353-374: Templates, aliases, and constants
```cpp
  // Codec tile size process by this workgroup.
  // Each threads processes a fragment of f16x8_t (16B),
  // into a int8x8_t (8B) and a f16 scale shared among 32 values.
  static constexpr int kRankAtoms = kAtoms / kWorldSize;
  static constexpr int kRankTileStride = 2176;
  static constexpr int kRankTileScaleOffset = 2048;
  static constexpr int kRankTransmittedTileSize = kRankTileStride * kRankAtoms;
  static_assert(kRankTransmittedTileSize % 16 == 0, "kRankTileSize must be 16B aligned.");

  static constexpr int kRankBufferTileStride = kRankTileStride / sizeof(int32x4_t);

  // Total tile size for the collective communication.
  static constexpr int kTransmittedTileSize = kRankTransmittedTileSize * kWorldSize;

  // Constants configuration

  // {-1/128.0h, -1/128.0h}, f16x2_t
  static constexpr int kScaleFactor = std::is_same<T, half>::value ? 0xA000A000 : 0xBC00BC00;

  // {1e-7, 1e-7}, f16x2_t
  static constexpr int kScaleEpsilon = std::is_same<T, half>::value ? 0x00010001 : 0x33D733D7;
```
**EN:** This section defines `static_assert`, `kRankAtoms`, `kRankTileStride`, giving the file reusable compile-time abstractions, aliases, and specialization points.
**CN:** 本段定义了`static_assert`、`kRankAtoms`、`kRankTileStride`等内容，为文件提供可复用的编译期抽象、类型别名和特化入口。

### Lines 375-398: Templates, aliases, and constants
```cpp
  // {-128, -128}, f16x2_t
  static constexpr int kRangeMin = std::is_same<T, half>::value ? 0xD800D800 : 0xC300C300;
  // {+127, +127}, f16x2_t
  static constexpr int kRangeMax = std::is_same<T, half>::value ? 0x57F057F0 : 0x42FE42FE;

  // {+128, +128}, int16x2_t
  static constexpr int kRangeBias = 0x00800080;

  __quickreduce_device_inline__ CodecQ8(int thread, int rank) : CodecBase(thread, rank) {}

  __quickreduce_device_inline__ void send(int32x4_t* __restrict__ send_buffer, int32x4_t const* __restrict__ data) {
    for (int k = 0; k < kRankAtoms; k++) {
      int32x4_t const atom = data[k];
      // Compute the absolute maximum of the atom in the thread group
      // In 2 blocks of values, upper/lower halves of the f16x2_t
      int wblockmax = group_abs_max<T>(atom);

      // Derive scales
      int decoding_scale;
      int encoding_scale;
      decoding_scale = packed_mul<T>(wblockmax, kScaleFactor);
      encoding_scale = packed_add<T>(decoding_scale, kScaleEpsilon);
      encoding_scale = packed_rcp<T>(encoding_scale);
```
**EN:** This section defines `CodecQ8`, `send`, `kRangeMin`, giving the file reusable compile-time abstractions, aliases, and specialization points.
**CN:** 本段定义了`CodecQ8`、`send`、`kRangeMin`等内容，为文件提供可复用的编译期抽象、类型别名和特化入口。

### Lines 399-424: Control flow and branching
```cpp
      // Apply scales to get quantized values
      int32x4_t w;
      for (int i = 0; i < 4; i++) {
        w[i] = packed_mul<T>(atom[i], encoding_scale);
        w[i] = packed_max<T>(w[i], kRangeMin);
        w[i] = packed_min<T>(w[i], kRangeMax);
      }

      // Convert from f16x2_t to uint16x2_t
      int32x4_t q;
      {
        int16_t* qi = reinterpret_cast<int16_t*>(&q);
        T* wh = reinterpret_cast<T*>(&w);
        for (int i = 0; i < 8; i++)
          qi[i] = (int16_t)rintf(T2float_cast(wh[i]));

        for (int i = 0; i < 4; i++) {
          q[i] = packed_add<int16_t>(q[i], kRangeBias);
        }
      }

      // Pack 8 x q8 into int32x2_t
      int32x2_t qw;
      qw[0] = q[0] | (q[1] << 8);
      qw[1] = q[2] | (q[3] << 8);
```
**EN:** This section drives `rintf` through branches and loops, deciding which execution path or data-handling strategy is used.
**CN:** 本段通过分支和循环推进`rintf`相关逻辑，决定采用哪条执行路径或数据处理策略。

### Lines 425-449: Device helpers and synchronization
```cpp
      // Write quantized atom to send_buffer
      // note: only the group leader stores the scale
      uint8_t* atom_ptr = reinterpret_cast<uint8_t*>(send_buffer + k * kRankBufferTileStride);
      int32x2_t* qw_ptr = reinterpret_cast<int32x2_t*>(atom_ptr) + thread;
      int* qs_ptr = reinterpret_cast<int*>(atom_ptr + kRankTileScaleOffset) + (thread / 8);

      __builtin_nontemporal_store(qw, qw_ptr);
      if (threadIdx.x == group_leader) {
        __builtin_nontemporal_store(decoding_scale, qs_ptr);
      }
    }
  }

  __quickreduce_device_inline__ void recv(int32x4_t** __restrict__ recv_buffer, int32x4_t* __restrict__ data) {
    for (int k = 0; k < kRankAtoms; k++) {
      // Directly read quantized atom from recv_buffer
      uint8_t* atom_ptr = reinterpret_cast<uint8_t*>(*recv_buffer);
      int32x2_t* qw_ptr = reinterpret_cast<int32x2_t*>(atom_ptr) + thread;
      int* qs_ptr = reinterpret_cast<int*>(atom_ptr + kRankTileScaleOffset) + (thread / 8);

      int32x2_t qw = __builtin_nontemporal_load(qw_ptr);
      int qs = __builtin_nontemporal_load(qs_ptr);

      *recv_buffer += kRankBufferTileStride;
```
**EN:** This section implements `recv`, `__builtin_nontemporal_store`, `__builtin_nontemporal_load`, low-level device-side helpers for arithmetic, synchronization, communication, or memory movement.
**CN:** 本段实现了`recv`、`__builtin_nontemporal_store`、`__builtin_nontemporal_load`等底层设备侧辅助逻辑，用于算术处理、同步、通信或内存搬运。

### Lines 450-475: Templates, aliases, and constants
```cpp
      // Unpack q8 into fp16x8_t
      int32x4_t w;
      {
        static uint constexpr kMask00FF = 0x00FF00FF;

        // {1024.0, 1024.0}, fp16x2_t
        static uint constexpr kHalf2_1024 = 0x64006400;

        // {-1152.0, -1152.0}, fp16x2_t
        static uint constexpr kHalf2_1152 = 0xE480E480;

#pragma unroll
        for (int i = 0; i < 4; i++) {
          if constexpr (std::is_same<T, half>::value) {
            int32_t q8 = ((qw[i / 2] >> ((i % 2) * 8)) & kMask00FF) | kHalf2_1024;
            w[i] = packed_add<half>(q8, kHalf2_1152);
          } else {
            int32_t int16_2 = (qw[i / 2] >> ((i % 2) * 8)) & kMask00FF;
            int16_t low = static_cast<int16_t>(int16_2 & 0xFFFF);
            int16_t high = static_cast<int16_t>((int16_2 >> 16) & 0xFFFF);
            nv_bfloat16 bf_low = __float2bfloat16(static_cast<float>(low));
            nv_bfloat16 bf_high = __float2bfloat16(static_cast<float>(high));
            nv_bfloat162 bf2 = __halves2bfloat162(bf_low, bf_high);
            int32_t packed_bf16 = *reinterpret_cast<int32_t*>(&bf2);
            w[i] = packed_add<nv_bfloat16>(packed_bf16, kRangeMin);
          }
```
**EN:** This section defines `__float2bfloat16`, `__halves2bfloat162`, `kMask00FF`, giving the file reusable compile-time abstractions, aliases, and specialization points.
**CN:** 本段定义了`__float2bfloat16`、`__halves2bfloat162`、`kMask00FF`等内容，为文件提供可复用的编译期抽象、类型别名和特化入口。

### Lines 476-495: Types and data layout
```cpp
        }
      }

      // Apply decoding scales
      for (int i = 0; i < 4; i++) {
        w[i] = packed_mul<T>(w[i], qs);
      }

      data[k] = w;
    }
  }
};

// Twoshot All Reduce
template <typename T, class Codec, bool cast_bf2half>
struct AllReduceTwoshot {
  static_assert(sizeof(T) == 2);

  static constexpr int kWorldSize = Codec::kWorldSize;
```
**EN:** This section defines `Codec`, `AllReduceTwoshot`, `static_assert`, packaging state, metadata, or memory layout information that later kernels and wrappers consume.
**CN:** 本段定义了`Codec`、`AllReduceTwoshot`、`static_assert`等内容，用于封装后续内核与封装层会使用的状态、元数据或内存布局信息。

### Lines 496-518: Device helpers and synchronization
```cpp
  __device__ static void
  run(T const* __restrict__ input,
      T* __restrict__ output,
      uint32_t const N,                    // number of elements
      int const block,                     // block index
      int const rank,                      // rank index
      uint8_t** __restrict__ buffer_list,  // communication buffers
      uint32_t const data_offset,          // offset to start of the data buffer
      uint32_t flag_color,
      int64_t data_size_per_phase) {
    // Topology
    int thread = threadIdx.x + threadIdx.y * kWavefront;
    uint8_t* rank_buffer = buffer_list[rank];
    Codec codec(thread, rank);
    int block_id = blockIdx.x;
    int grid_size = gridDim.x;
    // --------------------------------------------------------
    // Read input into registers
    int32x4_t tA[kAtoms];

    BufferResource src_buffer(const_cast<T*>(input), N * sizeof(T));
    uint32_t src_offset = block * kTileSize + thread * sizeof(int32x4_t);
```
**EN:** This section implements `run`, `codec`, `src_buffer`, low-level device-side helpers for arithmetic, synchronization, communication, or memory movement.
**CN:** 本段实现了`run`、`codec`、`src_buffer`等底层设备侧辅助逻辑，用于算术处理、同步、通信或内存搬运。

### Lines 519-542: Templates, aliases, and constants
```cpp
    for (int i = 0; i < kAtoms; i++) {
      tA[i] = buffer_load_dwordx4(src_buffer.descriptor, src_offset, 0, 0);
      src_offset += kAtomStride * sizeof(int32x4_t);
      if constexpr (cast_bf2half) {
        const nv_bfloat162* bf_buf = reinterpret_cast<const nv_bfloat162*>(&tA[i]);
        half2 half_buf[4];
#pragma unroll
        for (int j = 0; j < 4; ++j) {
          float2 f = __bfloat1622float2(bf_buf[j]);
          half_buf[j] = __float22half2_rn(f);
        }
        tA[i] = *reinterpret_cast<const int32x4_t*>(half_buf);
      }
    }

    // --------------------------------------------------------
    // Phase-1A: Write segment data into the communication buffer of the target
    // rank responsible for this segment.
    uint32_t comm_data0_offset = data_offset + block_id * Codec::kTransmittedTileSize;
    uint32_t comm_data1_offset = data_size_per_phase + comm_data0_offset;

    uint32_t comm_flags0_offset = block_id * (kWorldSize * sizeof(uint32_t));
    uint32_t comm_flags1_offset = (data_offset / 2) + comm_flags0_offset;
```
**EN:** This section defines `buffer_load_dwordx4`, `__bfloat1622float2`, `__float22half2_rn`, giving the file reusable compile-time abstractions, aliases, and specialization points.
**CN:** 本段定义了`buffer_load_dwordx4`、`__bfloat1622float2`、`__float22half2_rn`等内容，为文件提供可复用的编译期抽象、类型别名和特化入口。

### Lines 543-567: Device helpers and synchronization
```cpp
    for (int r = 0; r < kWorldSize; r++) {
      int32x4_t* send_buffer =
          reinterpret_cast<int32x4_t*>(buffer_list[r] + comm_data0_offset + rank * Codec::kRankTransmittedTileSize);
      codec.send(send_buffer, &tA[r * Codec::kRankAtoms]);
    }

    __syncthreads();
    if (thread < kWorldSize) {
      int r = thread;
      uint32_t* flag_ptr = reinterpret_cast<uint32_t*>(buffer_list[r] + comm_flags0_offset + rank * sizeof(uint32_t));
      set_sync_flag(flag_ptr, flag_color);
    }
    // --------------------------------------------------------
    // Phase-1B: Reduce the segment data from the communication buffers.
    int32x4_t tR[Codec::kRankAtoms] = {};
    {
      // Read the data from the communication buffer.
      int32x4_t* recv_buffer = reinterpret_cast<int32x4_t*>(rank_buffer + comm_data0_offset);
      uint32_t* flag_ptr = reinterpret_cast<uint32_t*>(rank_buffer + comm_flags0_offset);

      for (int r = 0; r < kWorldSize; r++) {
        // Wait for the flags to be set.
        if (thread == 0) {
          wait_sync_flag(&flag_ptr[r], flag_color);
        }
```
**EN:** This section implements `send`, `__syncthreads`, `set_sync_flag`, low-level device-side helpers for arithmetic, synchronization, communication, or memory movement.
**CN:** 本段实现了`send`、`__syncthreads`、`set_sync_flag`等底层设备侧辅助逻辑，用于算术处理、同步、通信或内存搬运。

### Lines 568-591: Device helpers and synchronization
```cpp
        __syncthreads();

        // note: we reuse tA as temp buffer here
        codec.recv(&recv_buffer, tA);

        for (int i = 0; i < Codec::kRankAtoms; i++) {
          packed_assign_add<T>(&tR[i], &tA[i]);
        }
      }
    }

    // Phase-2: Write the reduced segment to every other rank
    for (int r = 0; r < kWorldSize; r++) {
      int32x4_t* send_buffer =
          reinterpret_cast<int32x4_t*>(buffer_list[r] + comm_data1_offset + rank * Codec::kRankTransmittedTileSize);
      codec.send(send_buffer, tR);
    }

    __syncthreads();
    if (thread < kWorldSize) {
      int r = thread;
      uint32_t* flag_ptr = reinterpret_cast<uint32_t*>(buffer_list[r] + comm_flags1_offset + rank * sizeof(uint32_t));
      set_sync_flag(flag_ptr, flag_color);
    }
```
**EN:** This section implements `__syncthreads`, `recv`, `send`, low-level device-side helpers for arithmetic, synchronization, communication, or memory movement.
**CN:** 本段实现了`__syncthreads`、`recv`、`send`等底层设备侧辅助逻辑，用于算术处理、同步、通信或内存搬运。

### Lines 592-615: Device helpers and synchronization
```cpp

    // Phase-2: Read the gather segments from the rank's communication buffer.
    {
      // Read the data from the communication buffer.
      int32x4_t* recv_buffer = reinterpret_cast<int32x4_t*>(rank_buffer + comm_data1_offset);
      uint32_t* flag_ptr = reinterpret_cast<uint32_t*>(rank_buffer + comm_flags1_offset);

      for (int r = 0; r < kWorldSize; r++) {
        // Wait for the flags to be set.
        if (thread == 0) {
          wait_sync_flag(&flag_ptr[r], flag_color);
        }
        __syncthreads();

        // Gather all reduced and final rank segments into tA.
        codec.recv(&recv_buffer, &tA[r * Codec::kRankAtoms]);
      }
    }

    // --------------------------------------------------------
    // Write the result to output.
    BufferResource dst_buffer(output, N * sizeof(T));
    uint32_t dst_offset = block * kTileSize + thread * sizeof(int32x4_t);
```
**EN:** This section implements `wait_sync_flag`, `__syncthreads`, `recv`, low-level device-side helpers for arithmetic, synchronization, communication, or memory movement.
**CN:** 本段实现了`wait_sync_flag`、`__syncthreads`、`recv`等底层设备侧辅助逻辑，用于算术处理、同步、通信或内存搬运。

### Lines 616-633: Templates, aliases, and constants
```cpp
    for (int i = 0; i < kAtoms; i++) {
      if constexpr (cast_bf2half) {
        const half2* half_buf = reinterpret_cast<const half2*>(&tA[i]);
        nv_bfloat162 bf16_buf[4];
#pragma unroll
        for (int j = 0; j < 4; ++j) {
          float2 f = __half22float2(half_buf[j]);
          bf16_buf[j] = __float22bfloat162_rn(f);
        }
        buffer_store_dwordx4(*reinterpret_cast<const int32x4_t*>(bf16_buf), dst_buffer.descriptor, dst_offset, 0, 0);
      } else {
        buffer_store_dwordx4(tA[i], dst_buffer.descriptor, dst_offset, 0, 0);
      }
      dst_offset += kAtomStride * sizeof(int32x4_t);
    }
  }
};
```
**EN:** This section defines `__half22float2`, `__float22bfloat162_rn`, `buffer_store_dwordx4`, giving the file reusable compile-time abstractions, aliases, and specialization points.
**CN:** 本段定义了`__half22float2`、`__float22bfloat162_rn`、`buffer_store_dwordx4`等内容，为文件提供可复用的编译期抽象、类型别名和特化入口。

### Lines 634-634: Local implementation details
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

## Dependencies / 依赖关系
- **Internal headers / 内部头文件**: `quick_all_reduce_base.h`
- **External headers / 外部头文件**: `hip/hip_runtime.h`
- **Path context / 路径上下文**: allreduce / quick_all_reduce.cuh
