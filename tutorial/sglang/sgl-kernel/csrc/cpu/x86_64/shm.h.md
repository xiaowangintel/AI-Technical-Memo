# shm.h — Code Analysis / 代码分析

## Source / 来源
- **File**: `sgl-kernel/csrc/cpu/x86_64/shm.h`
- **Repository**: `sgl-project/sglang`
- **Purpose**: Provides CPU-side reference implementations, preprocessing, or fallback execution paths. This header primarily defines interfaces, templates, constants, and inline helpers shared by compilation units. / 提供 CPU 侧参考实现、预处理或回退执行路径。 该头文件主要定义编译单元共享的接口、模板、常量和内联辅助逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-22: Local implementation details
```cpp
#pragma once
#include <immintrin.h>

// Reduce functions down below use vectorized algorithm, the number of bytes
// processed each iteration depends on vector length.  256bit vector ==> 32
// bytes, 512bit vector ==> 64 bytes If you change implementation of
// reduce_bf16_buffers, etc. , check whether this number needs to be changed
#define VECTOR_LENGTH_IN_BYTES 32

inline __m512 cvt_bf16_to_fp32(const __m256i src) __attribute__((target("avx512bw")));
inline __m512 cvt_bf16_to_fp32(const __m256i src) {
  auto y = _mm512_cvtepu16_epi32(src);
  return _mm512_castsi512_ps(_mm512_bslli_epi128(y, 2));
}

inline __m256i cvt_fp32_to_bf16(const __m512 src) __attribute__((target("avx512bw")));
inline __m256i cvt_fp32_to_bf16(const __m512 src) {
  __m512i value = _mm512_castps_si512(src);
  __m512i nan = _mm512_set1_epi32(0xffff);
  auto mask_value = _mm512_cmp_ps_mask(src, src, _CMP_ORD_Q);
  __m512i ones = _mm512_set1_epi32(0x1);
  __m512i vec_bias = _mm512_set1_epi32(0x7fff);
```
**EN:** This section fills in the local implementation details around `cvt_bf16_to_fp32`, `cvt_fp32_to_bf16`, `_mm512_cvtepu16_epi32`, completing the behavior required by the file.
**CN:** 本段补充了`cvt_bf16_to_fp32`、`cvt_fp32_to_bf16`、`_mm512_cvtepu16_epi32`周边的局部实现细节，以完成该文件所需的具体行为。

### Lines 23-44: Local implementation details
```cpp
  // uint32_t lsb = (input >> 16) & 1;
  auto t_value = _mm512_and_si512(_mm512_srli_epi32(value, 16), ones);
  // uint32_t rounding_bias = 0x7fff + lsb;
  t_value = _mm512_add_epi32(t_value, vec_bias);
  // input += rounding_bias;
  t_value = _mm512_add_epi32(t_value, value);
  // input = input >> 16;
  t_value = _mm512_srli_epi32(t_value, 16);
  // Check NaN before converting back to bf16
  t_value = _mm512_mask_blend_epi32(mask_value, nan, t_value);
  return _mm512_cvtusepi32_epi16(t_value);
}

inline __m512 cvt_fp16_to_fp32(const __m256i src) __attribute__((target("avx512bw")));
inline __m512 cvt_fp16_to_fp32(const __m256i src) {
  return _mm512_cvtph_ps(src);
}

inline __m256i cvt_fp32_to_fp16(const __m512 src) __attribute__((target("avx512bw")));
inline __m256i cvt_fp32_to_fp16(const __m512 src) {
  return _mm512_cvtps_ph(src, (_MM_FROUND_TO_NEAREST_INT | _MM_FROUND_NO_EXC));
}
```
**EN:** This section fills in the local implementation details around `cvt_fp16_to_fp32`, `cvt_fp32_to_fp16`, `_mm512_and_si512`, completing the behavior required by the file.
**CN:** 本段补充了`cvt_fp16_to_fp32`、`cvt_fp32_to_fp16`、`_mm512_and_si512`周边的局部实现细节，以完成该文件所需的具体行为。

### Lines 45-66: Control flow and branching
```cpp

#define CVT_ADD_BF16(x)                                                                  \
  do {                                                                                   \
    auto in##x##_val = cvt_bf16_to_fp32(_mm256_loadu_si256((__m256i*)(buffers[x] + i))); \
    inout_val = _mm512_add_ps(inout_val, in##x##_val);                                   \
  } while (0)

__attribute__((target("avx512bw"))) inline void
reduce_bf16_buffers(int start_elements, int num_elements, char* to_buffer, char** buffers, int world_size) {
  const int element_size = 2;
  const int vector_length = VECTOR_LENGTH_IN_BYTES / element_size;
  int main_elements = num_elements - (num_elements % vector_length);
  int remain_elements = num_elements % vector_length;

  // process aligned part
#pragma omp parallel for
  for (int i = start_elements * element_size; i < (start_elements + main_elements) * element_size;
       i += VECTOR_LENGTH_IN_BYTES) {
    auto inout_val = cvt_bf16_to_fp32(_mm256_loadu_si256((__m256i*)(buffers[0] + i)));
    switch (world_size) {
      case 16:
        CVT_ADD_BF16(15);
```
**EN:** This section drives `cvt_bf16_to_fp32`, `_mm512_add_ps`, `CVT_ADD_BF16` through branches and loops, deciding which execution path or data-handling strategy is used.
**CN:** 本段通过分支和循环推进`cvt_bf16_to_fp32`、`_mm512_add_ps`、`CVT_ADD_BF16`相关逻辑，决定采用哪条执行路径或数据处理策略。

### Lines 67-88: Control flow and branching
```cpp
      case 15:
        CVT_ADD_BF16(14);
      case 14:
        CVT_ADD_BF16(13);
      case 13:
        CVT_ADD_BF16(12);
      case 12:
        CVT_ADD_BF16(11);
      case 11:
        CVT_ADD_BF16(10);
      case 10:
        CVT_ADD_BF16(9);
      case 9:
        CVT_ADD_BF16(8);
      case 8:
        CVT_ADD_BF16(7);
      case 7:
        CVT_ADD_BF16(6);
      case 6:
        CVT_ADD_BF16(5);
      case 5:
        CVT_ADD_BF16(4);
```
**EN:** This section drives `CVT_ADD_BF16` through branches and loops, deciding which execution path or data-handling strategy is used.
**CN:** 本段通过分支和循环推进`CVT_ADD_BF16`相关逻辑，决定采用哪条执行路径或数据处理策略。

### Lines 89-112: Runtime integration and dispatch
```cpp
      case 4:
        CVT_ADD_BF16(3);
      case 3:
        CVT_ADD_BF16(2);
      case 2:
        CVT_ADD_BF16(1);
      case 1:
        break;
      default:
        for (int j = 1; j < world_size; j++) {
          auto in_val = cvt_bf16_to_fp32(_mm256_loadu_si256((__m256i*)(buffers[j] + i)));
          inout_val = _mm512_add_ps(inout_val, in_val);
        }
    }
    _mm256_storeu_si256((__m256i*)(to_buffer + i), cvt_fp32_to_bf16(inout_val));
  }

  // process remaining part
  int i = (start_elements + main_elements) * element_size;
  while (remain_elements > 0) {
    float val = 0.0f;
    for (int j = 0; j < world_size; j++) {
      val += *(at::BFloat16*)(buffers[j] + i);
    }
```
**EN:** This section uses `CVT_ADD_BF16`, `cvt_bf16_to_fp32`, `_mm512_add_ps` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`CVT_ADD_BF16`、`cvt_bf16_to_fp32`、`_mm512_add_ps`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 113-131: Runtime integration and dispatch
```cpp
    *(at::BFloat16*)(to_buffer + i) = val;
    remain_elements--;
    i += element_size;
  }
}

#define CVT_ADD_FP16(x)                                                                  \
  do {                                                                                   \
    auto in##x##_val = cvt_fp16_to_fp32(_mm256_loadu_si256((__m256i*)(buffers[x] + i))); \
    inout_val = _mm512_add_ps(inout_val, in##x##_val);                                   \
  } while (0)

__attribute__((target("avx512bw"))) inline void
reduce_fp16_buffers(int start_elements, int num_elements, char* to_buffer, char** buffers, int world_size) {
  const int element_size = 2;
  const int vector_length = VECTOR_LENGTH_IN_BYTES / element_size;
  int main_elements = num_elements - (num_elements % vector_length);
  int remain_elements = num_elements % vector_length;
```
**EN:** This section uses `cvt_fp16_to_fp32`, `_mm512_add_ps`, `element_size` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`cvt_fp16_to_fp32`、`_mm512_add_ps`、`element_size`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 132-153: Control flow and branching
```cpp
  // process aligned part
#pragma omp parallel for
  for (int i = start_elements * element_size; i < (start_elements + main_elements) * element_size;
       i += VECTOR_LENGTH_IN_BYTES) {
    auto inout_val = cvt_fp16_to_fp32(_mm256_loadu_si256((__m256i*)(buffers[0] + i)));
    switch (world_size) {
      case 16:
        CVT_ADD_FP16(15);
      case 15:
        CVT_ADD_FP16(14);
      case 14:
        CVT_ADD_FP16(13);
      case 13:
        CVT_ADD_FP16(12);
      case 12:
        CVT_ADD_FP16(11);
      case 11:
        CVT_ADD_FP16(10);
      case 10:
        CVT_ADD_FP16(9);
      case 9:
        CVT_ADD_FP16(8);
```
**EN:** This section drives `cvt_fp16_to_fp32`, `CVT_ADD_FP16` through branches and loops, deciding which execution path or data-handling strategy is used.
**CN:** 本段通过分支和循环推进`cvt_fp16_to_fp32`、`CVT_ADD_FP16`相关逻辑，决定采用哪条执行路径或数据处理策略。

### Lines 154-175: Control flow and branching
```cpp
      case 8:
        CVT_ADD_FP16(7);
      case 7:
        CVT_ADD_FP16(6);
      case 6:
        CVT_ADD_FP16(5);
      case 5:
        CVT_ADD_FP16(4);
      case 4:
        CVT_ADD_FP16(3);
      case 3:
        CVT_ADD_FP16(2);
      case 2:
        CVT_ADD_FP16(1);
      case 1:
        break;
      default:
        for (int j = 1; j < world_size; j++) {
          auto in_val = cvt_fp16_to_fp32(_mm256_loadu_si256((__m256i*)(buffers[j] + i)));
          inout_val = _mm512_add_ps(inout_val, in_val);
        }
    }
```
**EN:** This section drives `CVT_ADD_FP16`, `cvt_fp16_to_fp32`, `_mm512_add_ps` through branches and loops, deciding which execution path or data-handling strategy is used.
**CN:** 本段通过分支和循环推进`CVT_ADD_FP16`、`cvt_fp16_to_fp32`、`_mm512_add_ps`相关逻辑，决定采用哪条执行路径或数据处理策略。

### Lines 176-197: Runtime integration and dispatch
```cpp
    _mm256_storeu_si256((__m256i*)(to_buffer + i), cvt_fp32_to_fp16(inout_val));
  }

  // process remaining part
  int i = (start_elements + main_elements) * element_size;
  while (remain_elements > 0) {
    float val = 0.0f;
    for (int j = 0; j < world_size; j++) {
      val += *(at::Half*)(buffers[j] + i);
    }
    *(at::Half*)(to_buffer + i) = val;
    remain_elements--;
    i += element_size;
  }
}

#define CVT_ADD_F32(x)                                            \
  do {                                                            \
    auto in##x##_val = _mm256_loadu_ps((float*)(buffers[x] + i)); \
    inout_val = _mm256_add_ps(inout_val, in##x##_val);            \
  } while (0)
```
**EN:** This section uses `_mm256_storeu_si256`, `_mm256_loadu_ps`, `_mm256_add_ps` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`_mm256_storeu_si256`、`_mm256_loadu_ps`、`_mm256_add_ps`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 198-219: Control flow and branching
```cpp
__attribute__((target("avx512bw"))) inline void
reduce_fp32_buffers(int start_elements, int num_elements, char* to_buffer, char** buffers, int world_size) {
  const int element_size = 4;
  const int vector_length = VECTOR_LENGTH_IN_BYTES / element_size;
  int main_elements = num_elements - (num_elements % vector_length);
  int remain_elements = num_elements % vector_length;

  // process aligned part
#pragma omp parallel for
  for (int i = start_elements * element_size; i < (start_elements + main_elements) * element_size;
       i += VECTOR_LENGTH_IN_BYTES) {
    auto inout_val = _mm256_loadu_ps((float*)(buffers[0] + i));
    switch (world_size) {
      case 16:
        CVT_ADD_F32(15);
      case 15:
        CVT_ADD_F32(14);
      case 14:
        CVT_ADD_F32(13);
      case 13:
        CVT_ADD_F32(12);
      case 12:
```
**EN:** This section drives `__attribute__`, `_mm256_loadu_ps`, `CVT_ADD_F32` through branches and loops, deciding which execution path or data-handling strategy is used.
**CN:** 本段通过分支和循环推进`__attribute__`、`_mm256_loadu_ps`、`CVT_ADD_F32`相关逻辑，决定采用哪条执行路径或数据处理策略。

### Lines 220-247: Control flow and branching
```cpp
        CVT_ADD_F32(11);
      case 11:
        CVT_ADD_F32(10);
      case 10:
        CVT_ADD_F32(9);
      case 9:
        CVT_ADD_F32(8);
      case 8:
        CVT_ADD_F32(7);
      case 7:
        CVT_ADD_F32(6);
      case 6:
        CVT_ADD_F32(5);
      case 5:
        CVT_ADD_F32(4);
      case 4:
        CVT_ADD_F32(3);
      case 3:
        CVT_ADD_F32(2);
      case 2:
        CVT_ADD_F32(1);
      case 1:
        break;
      default:
        for (int j = 1; j < world_size; j++) {
          auto in_val = _mm256_loadu_ps((float*)(buffers[j] + i));
          inout_val = _mm256_add_ps(inout_val, in_val);
        }
```
**EN:** This section drives `CVT_ADD_F32`, `_mm256_loadu_ps`, `_mm256_add_ps` through branches and loops, deciding which execution path or data-handling strategy is used.
**CN:** 本段通过分支和循环推进`CVT_ADD_F32`、`_mm256_loadu_ps`、`_mm256_add_ps`相关逻辑，决定采用哪条执行路径或数据处理策略。

### Lines 248-272: Control flow and branching
```cpp
    }
    _mm256_storeu_ps((float*)(to_buffer + i), inout_val);
  }

  // process remaining part
  int i = (start_elements + main_elements) * element_size;
  while (remain_elements > 0) {
    float val = 0.0f;
    for (int j = 0; j < world_size; j++) {
      val += *(float*)(buffers[j] + i);
    }
    *(float*)(to_buffer + i) = val;
    remain_elements--;
    i += element_size;
  }
}

__attribute__((target("avx512bw"))) inline void parallel_memcpy(void* to, void* from, size_t n_bytes) {
  auto aligned_bytes = n_bytes - (n_bytes % VECTOR_LENGTH_IN_BYTES);
  // process aligned part
#pragma omp parallel for
  for (size_t i = 0; i < aligned_bytes; i += VECTOR_LENGTH_IN_BYTES) {
    auto val = _mm256_loadu_si256((__m256i*)((char*)from + i));
    _mm256_storeu_si256((__m256i*)((char*)to + i), val);
  }
```
**EN:** This section drives `__attribute__`, `_mm256_storeu_ps`, `_mm256_loadu_si256` through branches and loops, deciding which execution path or data-handling strategy is used.
**CN:** 本段通过分支和循环推进`__attribute__`、`_mm256_storeu_ps`、`_mm256_loadu_si256`相关逻辑，决定采用哪条执行路径或数据处理策略。

### Lines 273-279: Control flow and branching
```cpp

  // process remaining part
  for (size_t i = aligned_bytes; i < n_bytes; i++) {
    *((char*)to + i) = *((char*)from + i);
  }
}
```
**EN:** This section drives the surrounding logic through branches and loops, deciding which execution path or data-handling strategy is used.
**CN:** 本段通过分支和循环推进相关逻辑相关逻辑，决定采用哪条执行路径或数据处理策略。

### Lines 280-280: Local implementation details
```cpp
#undef VECTOR_LENGTH_IN_BYTES
```
**EN:** This section fills in the local implementation details around the surrounding logic, completing the behavior required by the file.
**CN:** 本段补充了相关逻辑周边的局部实现细节，以完成该文件所需的具体行为。

## Key Concepts / 关键概念
- **Low-level systems code / 底层系统代码**: Focuses on performance-sensitive implementation details close to the hardware. / 关注贴近硬件、对性能敏感的实现细节。

## Dependencies / 依赖关系
- **Internal headers / 内部头文件**: None explicitly listed / 未显式列出
- **External headers / 外部头文件**: `immintrin.h`
- **Path context / 路径上下文**: cpu / x86_64 / shm.h
