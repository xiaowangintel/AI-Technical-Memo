# matrix_view.cuh — Code Analysis / 代码分析

## Source / 来源
- **File**: `csrc/quantization/gptq/matrix_view.cuh`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Defines lightweight device-side matrix view classes for GPTQ kernels, covering dense half matrices and several packed low-bit layouts. / [CN] 为 GPTQ 内核定义轻量级设备端矩阵视图类，涵盖 dense half 矩阵以及多种打包低比特布局。

## Line-by-Line Analysis / 逐行分析

### Dense half read-only view
```cpp
class MatrixView_half {
 public:
  const half* data;
  const int height;
  const int width;

  __device__ __forceinline__ MatrixView_half(const half* data, const int height,
                                             const int width)
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

  __device__ __forceinline__ void item4(half (&items)[4], int row,
                                        int column) const {
    half2* ptr = (half2*)item_ptr(row, column);
    half2 i01 = ptr[0];
    half2 i23 = ptr[1];
    items[0] = __low2half(i01);
    items[1] = __high2half(i01);
    items[2] = __low2half(i23);
    items[3] = __high2half(i23);
  }
  __device__ __forceinline__ void item4_f(float (&items)[4], int row,
                                          int column) const {
    half2* ptr = (half2*)item_ptr(row, column);
    half2 i01 = ptr[0];
    half2 i23 = ptr[1];
    items[0] = __half2float(__low2half(i01));
    items[1] = __half2float(__high2half(i01));
    items[2] = __half2float(__low2half(i23));
    items[3] = __half2float(__high2half(i23));
  }

  __device__ __forceinline__ void item4_h2(half2 (&items)[4], int row,
                                           int column) const {
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
**EN:** `MatrixView_half` wraps a raw `half*` buffer with height/width metadata and provides scalar, `half2`, and 4-element accessors. The 4-element helpers unpack two `half2` reads into either scalars, floats, or duplicated `half2` values for later vectorized math.
**CN:** `MatrixView_half` 用高度/宽度元数据封装原始 `half*` 缓冲区，并提供标量、`half2` 与 4 元访问接口。4 元辅助函数会把两次 `half2` 读取拆成标量、float，或复制展开后的 `half2`，方便后续向量化计算。

### Dense half read-write view
```cpp
class MatrixView_half_rw {
 public:
  half* data;
  const int height;
  const int width;

  __device__ __forceinline__ MatrixView_half_rw(half* data, const int height,
                                                const int width)
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
  __device__ __forceinline__ void set_half2(int row, int column, half2 value) {
    ((half2*)data)[(row * width + column) / 2] = value;
  }

  __device__ __forceinline__ void set4(int row, int column, half v0, half v1,
                                       half v2, half v3) {
    half2 v01 = __halves2half2(v0, v1);
    half2 v23 = __halves2half2(v2, v3);
    half2* ptr = (half2*)item_ptr(row, column);
    ptr[0] = v01;
    ptr[1] = v23;
  }
};
```
**EN:** `MatrixView_half_rw` adds write access on top of the same addressing scheme. Its `set_half2` and `set4` helpers let kernels update multiple half values in packed form without manual pointer arithmetic every time.
**CN:** `MatrixView_half_rw` 在相同寻址方案上增加了写接口。其 `set_half2` 与 `set4` 辅助函数使内核可以用打包形式更新多个 half 值，而不必每次手工计算指针偏移。

### Packed Q4 row/column views
```cpp
class MatrixView_q4_row {
 public:
  const uint32_t* data;
  const int height;
  const int width;

  __device__ __forceinline__ MatrixView_q4_row(const uint32_t* data,
                                               const int height,
                                               const int width)
      : data(data), height(height), width(width) {}

  __device__ __forceinline__ int item(int row, int column) const {
    int shift = (column & 0x07) * 4;
    return (data[row * width / 8 + column / 8] >> shift) & 0x0f;
  }

  __device__ __forceinline__ void item2(int (&items)[2], int row,
                                        int column) const {
    int shift = (column & 0x07) * 4;
    uint32_t d = data[row * width / 8 + column / 8] >> shift;
    items[0] = d & 0x0f;
    items[1] = (d >> 4) & 0x0f;
  }

  __device__ __forceinline__ void item4(int (&items)[4], int row,
                                        int column) const {
    int shift = (column & 0x07) * 4;
    uint32_t d = data[row * width / 8 + column / 8] >> shift;
    items[0] = d & 0x0f;
    items[1] = (d >> 4) & 0x0f;
    items[2] = (d >> 8) & 0x0f;
    items[3] = (d >> 12) & 0x0f;
  }
};

class MatrixView_q4_column {
 public:
  const uint32_t* data;
  const int height;
  const int width;

  __device__ __forceinline__ MatrixView_q4_column(const uint32_t* data,
                                                  const int height,
                                                  const int width)
      : data(data), height(height), width(width) {}

  __device__ __forceinline__ int item(int row, int column) const {
    int shift = (row & 0x07) * 4;
    return (data[row / 8 * width + column] >> shift) & 0x0f;
  }

  __device__ __forceinline__ uint32_t item_uint32_t(int row, int column) {
    return data[row / 8 * width + column];
  }
  __device__ __forceinline__ const uint32_t* item_uint32_ptr(int row,
                                                             int column) {
    return &data[row / 8 * width + column];
  }
};
```
**EN:** `MatrixView_q4_row` interprets each `uint32_t` as eight 4-bit row-major values, while `MatrixView_q4_column` interprets it as eight column-major values stacked along rows. The two classes differ only in how they compute the bit shift and base word index.
**CN:** `MatrixView_q4_row` 把每个 `uint32_t` 解释为按行存放的 8 个 4-bit 值；`MatrixView_q4_column` 则把同一个 32 位字解释为沿行方向堆叠的列主序 4-bit 值。两者差别主要体现在位移和基础字索引的计算方式上。

### Packed Q2 row view
```cpp
class MatrixView_q2_row {
 public:
  const uint32_t* data;
  const int height;
  const int width;

  __device__ __forceinline__ MatrixView_q2_row(const uint32_t* data,
                                               const int height,
                                               const int width)
      : data(data), height(height), width(width) {}

  __device__ __forceinline__ int item(int row, int column) const {
    int shift = (column & 0x0f) * 2;
    return (data[row * width / 16 + column / 16] >> shift) & 0x03;
  }

  __device__ __forceinline__ void item2(int (&items)[2], int row,
                                        int column) const {
    int shift = (column & 0x0f) * 2;
    uint32_t d = data[row * width / 16 + column / 16] >> shift;
    items[0] = d & 0x03;
    items[1] = (d >> 2) & 0x03;
  }

  __device__ __forceinline__ void item4(int (&items)[4], int row,
                                        int column) const {
    int shift = (column & 0x0f) * 2;
    uint32_t d = data[row * width / 16 + column / 16] >> shift;
    items[0] = d & 0x03;
    items[1] = (d >> 2) & 0x03;
    items[2] = (d >> 4) & 0x03;
    items[3] = (d >> 6) & 0x03;
  }
};
```
**EN:** `MatrixView_q2_row` stores sixteen 2-bit values per `uint32_t`. Its `item2` and `item4` helpers extract multiple adjacent quantized values by shifting once and then masking successive 2-bit fields.
**CN:** `MatrixView_q2_row` 在每个 `uint32_t` 中存放 16 个 2-bit 值。它的 `item2` 与 `item4` 辅助函数通过一次右移后连续掩码，提取多个相邻的量化值。

### Packed Q3 row view
```cpp
class MatrixView_q3_row {
 public:
  const uint32_t* data;
  const int height;
  const int width;

  __device__ __forceinline__ MatrixView_q3_row(const uint32_t* data,
                                               const int height,
                                               const int width)
      : data(data), height(height), width(width) {}

  __device__ __forceinline__ int item(int row, int column) const {
    int z_w = column * 3 / 32;
    int z_mod = column & 0x1f;

    if (z_mod == 10) {
      return (data[row * width * 3 / 32 + z_w] >> 30) |
             ((data[row * width * 3 / 32 + (z_w + 1)] << 2) & 0x4);
    } else if (z_mod == 21) {
      return (data[row * width * 3 / 32 + z_w] >> 31) |
             ((data[row * width * 3 / 32 + (z_w + 1)] << 1) & 0x6);
    } else if (z_mod < 10) {
      return (data[row * width * 3 / 32 + z_w] >> (z_mod * 3)) & 0x07;
    } else if (z_mod < 21) {
      return (data[row * width * 3 / 32 + z_w] >> (z_mod * 3 - 32)) & 0x07;
    } else {
      return (data[row * width * 3 / 32 + z_w] >> (z_mod * 3 - 64)) & 0x07;
    }
  }

  __device__ __forceinline__ void item4(int (&items)[4], int row,
                                        int column) const {
    int shift = (column & 0x1f);
    uint32_t d;
    if (shift <= 4) {
      d = data[row * width / 32 * 3 + column * 3 / 32] >> (shift * 3);
    } else if (shift == 8) {
      d = (data[row * width / 32 * 3 + column * 3 / 32] >> 24) |
          ((data[row * width / 32 * 3 + column * 3 / 32 + 1] & 0x0f) << 8);
    } else if (shift <= 16) {
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
```
**EN:** Q3 packing is more irregular because 3-bit values do not align to 32-bit word boundaries. `MatrixView_q3_row` therefore contains special cases for positions that straddle two adjacent words and reconstructs those elements with cross-word shifts.
**CN:** Q3 打包更不规则，因为 3-bit 值无法自然对齐到 32-bit 边界。于是 `MatrixView_q3_row` 专门处理跨越相邻两个字的特殊位置，并通过跨字位移重建这些元素。

### Packed Q8 row view
```cpp
class MatrixView_q8_row {
 public:
  const uint32_t* data;
  const int height;
  const int width;

  __device__ __forceinline__ MatrixView_q8_row(const uint32_t* data,
                                               const int height,
                                               const int width)
      : data(data), height(height), width(width) {}

  __device__ __forceinline__ int item(int row, int column) const {
    int shift = (column & 0x03) * 8;
    return (data[row * width / 4 + column / 4] >> shift) & 0xff;
  }

  __device__ __forceinline__ void item2(int (&items)[2], int row,
                                        int column) const {
    int shift = (column & 0x03) * 8;
    uint32_t d = data[row * width / 4 + column / 4] >> shift;
    items[0] = d & 0xff;
    items[1] = (d >> 8) & 0xff;
  }

  __device__ __forceinline__ void item4(int (&items)[4], int row,
                                        int column) const {
    int shift = (column & 0x03) * 2;
    uint32_t d = data[row * width / 4 + column / 4] >> shift;
    items[0] = d & 0xff;
    items[1] = (d >> 8) & 0xff;
    items[2] = (d >> 16) & 0xff;
    items[3] = (d >> 24) & 0xff;
  }
};
```
**EN:** `MatrixView_q8_row` is the simplest packed low-bit view: each `uint32_t` holds four 8-bit values. The helpers expose scalar, pair, and 4-element extraction routines used by GPTQ dequantization code.
**CN:** `MatrixView_q8_row` 是最简单的打包低比特视图：每个 `uint32_t` 包含 4 个 8-bit 值。相关辅助函数提供标量、成对以及 4 元提取接口，供 GPTQ 反量化代码使用。

## Key Concepts / 关键概念
- EN: These classes are intentionally tiny wrappers around raw pointers so they inline well inside CUDA kernels.
  CN: 这些类本质上是对原始指针的极薄封装，目的是在 CUDA 内核中更好地内联。
- EN: Different quantized layouts are encoded purely by indexing and bit-extraction rules.
  CN: 不同量化布局完全通过索引规则与位提取规则来表达。
- EN: The read/write half views are used for dense intermediates, while Q2/Q3/Q4/Q8 views expose packed GPTQ weights.
  CN: half 读写视图用于 dense 中间结果，而 Q2/Q3/Q4/Q8 视图用于暴露打包后的 GPTQ 权重。

## Dependencies / 依赖关系
- EN: Includes CUDA half/half2 types and `qdq_util.cuh` for companion dequantization helpers.
  CN: 包含 CUDA 的 half/half2 类型，并依赖 `qdq_util.cuh` 中的配套反量化辅助函数。
- EN: Consumed by GPTQ kernels that need direct bit-level access to packed weight matrices.
  CN: 被需要直接按位访问打包权重矩阵的 GPTQ 内核使用。
