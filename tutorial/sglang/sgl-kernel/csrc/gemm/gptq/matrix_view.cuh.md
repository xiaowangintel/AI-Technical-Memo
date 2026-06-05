# matrix_view.cuh — Code Analysis / 代码分析

## Source / 来源
- **File**: `sgl-kernel/csrc/gemm/gptq/matrix_view.cuh`
- **Repository**: `sgl-project/sglang`
- **Purpose**: Provides GEMM / matrix multiplication kernels, tiling strategies, or low-level launch wrappers. This header primarily defines interfaces, templates, constants, and inline helpers shared by compilation units. / 提供 GEMM / 矩阵乘内核、分块策略或底层启动封装。 该头文件主要定义编译单元共享的接口、模板、常量和内联辅助逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-22: Types and data layout
```cpp
/*
Adapted from https://github.com/turboderp/exllamav2 and
https://github.com/turboderp/exllama
*/

#ifndef _matrix_view_cuh
#define _matrix_view_cuh

#include <cuda_fp16.h>
#include <cuda_runtime.h>

#include "qdq_util.cuh"

namespace sglang {
namespace gptq {

class MatrixView_half {
 public:
  const half* data;
  const int height;
  const int width;
```
**EN:** This section defines `MatrixView_half`, `data`, `height`, packaging state, metadata, or memory layout information that later kernels and wrappers consume.
**CN:** 本段定义了`MatrixView_half`、`data`、`height`等内容，用于封装后续内核与封装层会使用的状态、元数据或内存布局信息。

### Lines 23-47: Device helpers and synchronization
```cpp
  __device__ __forceinline__ MatrixView_half(const half* data, const int height, const int width)
      : data(data), height(height), width(width) {}

  __device__ __forceinline__ half item(int row, int column) const {
    return data[row * width + column];
  }
  __device__ __forceinline__ half2 item_half2(int row, int column) const {
    return ((half2*)data)[(row * width + column) / 2];
  }
  __device__ __forceinline__ half2 item_half2half2(int row, int column) const {
    return __half2half2(data[row * width + column]);
  }
  __device__ __forceinline__ const half* item_ptr(int row, int column) const {
    return &data[row * width + column];
  }

  __device__ __forceinline__ void item4(half (&items)[4], int row, int column) const {
    half2* ptr = (half2*)item_ptr(row, column);
    half2 i01 = ptr[0];
    half2 i23 = ptr[1];
    items[0] = __low2half(i01);
    items[1] = __high2half(i01);
    items[2] = __low2half(i23);
    items[3] = __high2half(i23);
  }
```
**EN:** This section implements `MatrixView_half`, `item`, `item_half2`, low-level device-side helpers for arithmetic, synchronization, communication, or memory movement.
**CN:** 本段实现了`MatrixView_half`、`item`、`item_half2`等底层设备侧辅助逻辑，用于算术处理、同步、通信或内存搬运。

### Lines 48-68: Device helpers and synchronization
```cpp
  __device__ __forceinline__ void item4_f(float (&items)[4], int row, int column) const {
    half2* ptr = (half2*)item_ptr(row, column);
    half2 i01 = ptr[0];
    half2 i23 = ptr[1];
    items[0] = __half2float(__low2half(i01));
    items[1] = __half2float(__high2half(i01));
    items[2] = __half2float(__low2half(i23));
    items[3] = __half2float(__high2half(i23));
  }

  __device__ __forceinline__ void item4_h2(half2 (&items)[4], int row, int column) const {
    half2* ptr = (half2*)item_ptr(row, column);
    half2 i01 = ptr[0];
    half2 i23 = ptr[1];
    items[0] = __half2half2(__low2half(i01));
    items[1] = __half2half2(__high2half(i01));
    items[2] = __half2half2(__low2half(i23));
    items[3] = __half2half2(__high2half(i23));
  }
};
```
**EN:** This section implements `item4_f`, `item4_h2`, `item_ptr`, low-level device-side helpers for arithmetic, synchronization, communication, or memory movement.
**CN:** 本段实现了`item4_f`、`item4_h2`、`item_ptr`等底层设备侧辅助逻辑，用于算术处理、同步、通信或内存搬运。

### Lines 69-89: Types and data layout
```cpp
class MatrixView_half_rw {
 public:
  half* data;
  const int height;
  const int width;

  __device__ __forceinline__ MatrixView_half_rw(half* data, const int height, const int width)
      : data(data), height(height), width(width) {}

  __device__ __forceinline__ half item(int row, int column) const {
    return data[row * width + column];
  }
  __device__ __forceinline__ half2 item_half2(int row, int column) const {
    return ((half2*)data)[(row * width + column) / 2];
  }
  __device__ __forceinline__ half* item_ptr(int row, int column) {
    return &data[row * width + column];
  }
  __device__ __forceinline__ void set(int row, int column, half value) {
    data[row * width + column] = value;
  }
```
**EN:** This section defines `MatrixView_half_rw`, `item`, `item_half2`, packaging state, metadata, or memory layout information that later kernels and wrappers consume.
**CN:** 本段定义了`MatrixView_half_rw`、`item`、`item_half2`等内容，用于封装后续内核与封装层会使用的状态、元数据或内存布局信息。

### Lines 90-111: Types and data layout
```cpp
  __device__ __forceinline__ void set_half2(int row, int column, half2 value) {
    ((half2*)data)[(row * width + column) / 2] = value;
  }

  __device__ __forceinline__ void set4(int row, int column, half v0, half v1, half v2, half v3) {
    half2 v01 = __halves2half2(v0, v1);
    half2 v23 = __halves2half2(v2, v3);
    half2* ptr = (half2*)item_ptr(row, column);
    ptr[0] = v01;
    ptr[1] = v23;
  }
};

class MatrixView_q4_row {
 public:
  const uint32_t* data;
  const int height;
  const int width;

  __device__ __forceinline__ MatrixView_q4_row(const uint32_t* data, const int height, const int width)
      : data(data), height(height), width(width) {}
```
**EN:** This section defines `MatrixView_q4_row`, `set_half2`, `set4`, packaging state, metadata, or memory layout information that later kernels and wrappers consume.
**CN:** 本段定义了`MatrixView_q4_row`、`set_half2`、`set4`等内容，用于封装后续内核与封装层会使用的状态、元数据或内存布局信息。

### Lines 112-133: Device helpers and synchronization
```cpp
  __device__ __forceinline__ int item(int row, int column) const {
    int shift = (column & 0x07) * 4;
    return (data[row * width / 8 + column / 8] >> shift) & 0x0f;
  }

  __device__ __forceinline__ void item2(int (&items)[2], int row, int column) const {
    int shift = (column & 0x07) * 4;
    uint32_t d = data[row * width / 8 + column / 8] >> shift;
    items[0] = d & 0x0f;
    items[1] = (d >> 4) & 0x0f;
  }

  __device__ __forceinline__ void item4(int (&items)[4], int row, int column) const {
    int shift = (column & 0x07) * 4;
    uint32_t d = data[row * width / 8 + column / 8] >> shift;
    items[0] = d & 0x0f;
    items[1] = (d >> 4) & 0x0f;
    items[2] = (d >> 8) & 0x0f;
    items[3] = (d >> 12) & 0x0f;
  }
};
```
**EN:** This section implements `item`, `item2`, `item4`, low-level device-side helpers for arithmetic, synchronization, communication, or memory movement.
**CN:** 本段实现了`item`、`item2`、`item4`等底层设备侧辅助逻辑，用于算术处理、同步、通信或内存搬运。

### Lines 134-155: Types and data layout
```cpp
class MatrixView_q4_column {
 public:
  const uint32_t* data;
  const int height;
  const int width;

  __device__ __forceinline__ MatrixView_q4_column(const uint32_t* data, const int height, const int width)
      : data(data), height(height), width(width) {}

  __device__ __forceinline__ int item(int row, int column) const {
    int shift = (row & 0x07) * 4;
    return (data[row / 8 * width + column] >> shift) & 0x0f;
  }

  __device__ __forceinline__ uint32_t item_uint32_t(int row, int column) {
    return data[row / 8 * width + column];
  }
  __device__ __forceinline__ const uint32_t* item_uint32_ptr(int row, int column) {
    return &data[row / 8 * width + column];
  }
};
```
**EN:** This section defines `MatrixView_q4_column`, `item`, `item_uint32_t`, packaging state, metadata, or memory layout information that later kernels and wrappers consume.
**CN:** 本段定义了`MatrixView_q4_column`、`item`、`item_uint32_t`等内容，用于封装后续内核与封装层会使用的状态、元数据或内存布局信息。

### Lines 156-176: Types and data layout
```cpp
class MatrixView_q2_row {
 public:
  const uint32_t* data;
  const int height;
  const int width;

  __device__ __forceinline__ MatrixView_q2_row(const uint32_t* data, const int height, const int width)
      : data(data), height(height), width(width) {}

  __device__ __forceinline__ int item(int row, int column) const {
    int shift = (column & 0x0f) * 2;
    return (data[row * width / 16 + column / 16] >> shift) & 0x03;
  }

  __device__ __forceinline__ void item2(int (&items)[2], int row, int column) const {
    int shift = (column & 0x0f) * 2;
    uint32_t d = data[row * width / 16 + column / 16] >> shift;
    items[0] = d & 0x03;
    items[1] = (d >> 2) & 0x03;
  }
```
**EN:** This section defines `MatrixView_q2_row`, `item`, `item2`, packaging state, metadata, or memory layout information that later kernels and wrappers consume.
**CN:** 本段定义了`MatrixView_q2_row`、`item`、`item2`等内容，用于封装后续内核与封装层会使用的状态、元数据或内存布局信息。

### Lines 177-199: Types and data layout
```cpp
  __device__ __forceinline__ void item4(int (&items)[4], int row, int column) const {
    int shift = (column & 0x0f) * 2;
    uint32_t d = data[row * width / 16 + column / 16] >> shift;
    items[0] = d & 0x03;
    items[1] = (d >> 2) & 0x03;
    items[2] = (d >> 4) & 0x03;
    items[3] = (d >> 6) & 0x03;
  }
};

class MatrixView_q3_row {
 public:
  const uint32_t* data;
  const int height;
  const int width;

  __device__ __forceinline__ MatrixView_q3_row(const uint32_t* data, const int height, const int width)
      : data(data), height(height), width(width) {}

  __device__ __forceinline__ int item(int row, int column) const {
    int z_w = column * 3 / 32;
    int z_mod = column & 0x1f;
```
**EN:** This section defines `MatrixView_q3_row`, `item4`, `item`, packaging state, metadata, or memory layout information that later kernels and wrappers consume.
**CN:** 本段定义了`MatrixView_q3_row`、`item4`、`item`等内容，用于封装后续内核与封装层会使用的状态、元数据或内存布局信息。

### Lines 200-221: Device helpers and synchronization
```cpp
    if (z_mod == 10) {
      return (data[row * width * 3 / 32 + z_w] >> 30) | ((data[row * width * 3 / 32 + (z_w + 1)] << 2) & 0x4);
    } else if (z_mod == 21) {
      return (data[row * width * 3 / 32 + z_w] >> 31) | ((data[row * width * 3 / 32 + (z_w + 1)] << 1) & 0x6);
    } else if (z_mod < 10) {
      return (data[row * width * 3 / 32 + z_w] >> (z_mod * 3)) & 0x07;
    } else if (z_mod < 21) {
      return (data[row * width * 3 / 32 + z_w] >> (z_mod * 3 - 32)) & 0x07;
    } else {
      return (data[row * width * 3 / 32 + z_w] >> (z_mod * 3 - 64)) & 0x07;
    }
  }

  __device__ __forceinline__ void item4(int (&items)[4], int row, int column) const {
    int shift = (column & 0x1f);
    uint32_t d;
    if (shift <= 4) {
      d = data[row * width / 32 * 3 + column * 3 / 32] >> (shift * 3);
    } else if (shift == 8) {
      d = (data[row * width / 32 * 3 + column * 3 / 32] >> 24) |
          ((data[row * width / 32 * 3 + column * 3 / 32 + 1] & 0x0f) << 8);
    } else if (shift <= 16) {
```
**EN:** This section implements `item4`, low-level device-side helpers for arithmetic, synchronization, communication, or memory movement.
**CN:** 本段实现了`item4`等底层设备侧辅助逻辑，用于算术处理、同步、通信或内存搬运。

### Lines 222-244: Types and data layout
```cpp
      d = data[row * width / 32 * 3 + column * 3 / 32] >> (shift * 3 - 32);
    } else if (shift == 20) {
      d = (data[row * width / 32 * 3 + column * 3 / 32] >> 28) |
          ((data[row * width / 32 * 3 + column * 3 / 32 + 1] & 0xff) << 4);
    } else {
      d = data[row * width / 32 * 3 + column * 3 / 32] >> (shift * 3 - 64);
    }
    items[0] = d & 0x07;
    items[1] = (d >> 3) & 0x07;
    items[2] = (d >> 6) & 0x07;
    items[3] = (d >> 9) & 0x07;
  }
};

class MatrixView_q8_row {
 public:
  const uint32_t* data;
  const int height;
  const int width;

  __device__ __forceinline__ MatrixView_q8_row(const uint32_t* data, const int height, const int width)
      : data(data), height(height), width(width) {}
```
**EN:** This section defines `MatrixView_q8_row`, `data`, `height`, packaging state, metadata, or memory layout information that later kernels and wrappers consume.
**CN:** 本段定义了`MatrixView_q8_row`、`data`、`height`等内容，用于封装后续内核与封装层会使用的状态、元数据或内存布局信息。

### Lines 245-266: Device helpers and synchronization
```cpp
  __device__ __forceinline__ int item(int row, int column) const {
    int shift = (column & 0x03) * 8;
    return (data[row * width / 4 + column / 4] >> shift) & 0xff;
  }

  __device__ __forceinline__ void item2(int (&items)[2], int row, int column) const {
    int shift = (column & 0x03) * 8;
    uint32_t d = data[row * width / 4 + column / 4] >> shift;
    items[0] = d & 0xff;
    items[1] = (d >> 8) & 0xff;
  }

  __device__ __forceinline__ void item4(int (&items)[4], int row, int column) const {
    int shift = (column & 0x03) * 2;
    uint32_t d = data[row * width / 4 + column / 4] >> shift;
    items[0] = d & 0xff;
    items[1] = (d >> 8) & 0xff;
    items[2] = (d >> 16) & 0xff;
    items[3] = (d >> 24) & 0xff;
  }
};
```
**EN:** This section implements `item`, `item2`, `item4`, low-level device-side helpers for arithmetic, synchronization, communication, or memory movement.
**CN:** 本段实现了`item`、`item2`、`item4`等底层设备侧辅助逻辑，用于算术处理、同步、通信或内存搬运。

### Lines 267-269: Local implementation details
```cpp
}  // namespace gptq
}  // namespace sglang
#endif
```
**EN:** This section fills in the local implementation details around the surrounding logic, completing the behavior required by the file.
**CN:** 本段补充了相关逻辑周边的局部实现细节，以完成该文件所需的具体行为。

## Key Concepts / 关键概念
- **Device helpers / 设备侧辅助函数**: Uses inline device routines to structure low-level operations. / 使用内联设备函数组织底层操作。
- **CUDA runtime / CUDA 运行时**: Uses CUDA APIs, streams, or intrinsics for NVIDIA GPUs. / 使用 CUDA API、stream 或 intrinsic 支持 NVIDIA GPU。

## Dependencies / 依赖关系
- **Internal headers / 内部头文件**: `qdq_util.cuh`
- **External headers / 外部头文件**: `cuda_fp16.h`, `cuda_runtime.h`
- **Path context / 路径上下文**: gemm / gptq / matrix_view.cuh
