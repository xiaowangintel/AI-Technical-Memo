# shm.h — Code Analysis / 代码分析

## Source / 来源
- **File**: `sgl-kernel/csrc/cpu/aarch64/shm.h`
- **Repository**: `sgl-project/sglang`
- **Purpose**: Provides CPU-side reference implementations, preprocessing, or fallback execution paths. This header primarily defines interfaces, templates, constants, and inline helpers shared by compilation units. / 提供 CPU 侧参考实现、预处理或回退执行路径。 该头文件主要定义编译单元共享的接口、模板、常量和内联辅助逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-23: Local implementation details
```cpp
#pragma once
#include <arm_neon.h>

#define VECTOR_LENGTH_IN_BYTES 16

__attribute__((target("+bf16"))) inline float32x4x2_t cvt_bf16_to_fp32(const bfloat16x8_t src) {
  float32x4x2_t y;
  y.val[0] = vcvtq_low_f32_bf16(src);
  y.val[1] = vcvtq_high_f32_bf16(src);
  return y;
}

__attribute__((target("+bf16"))) inline bfloat16x8_t cvt_fp32_to_bf16(const float32x4x2_t src) {
  return vcvtq_high_bf16_f32(vcvtq_low_bf16_f32(src.val[0]), src.val[1]);
}

__attribute__((target("+bf16"))) inline void
reduce_bf16_buffers(int start_elements, int num_elements, char* to_buffer, char** buffers, int world_size) {
  const int element_size = 2;
  const int vector_length = VECTOR_LENGTH_IN_BYTES / element_size;
  int main_elements = num_elements - (num_elements % vector_length);
  int remain_elements = num_elements % vector_length;
```
**EN:** This section fills in the local implementation details around `__attribute__`, `vcvtq_low_f32_bf16`, `vcvtq_high_f32_bf16`, completing the behavior required by the file.
**CN:** 本段补充了`__attribute__`、`vcvtq_low_f32_bf16`、`vcvtq_high_f32_bf16`周边的局部实现细节，以完成该文件所需的具体行为。

### Lines 24-47: Control flow and branching
```cpp
  // process aligned part
#pragma omp parallel for
  for (int i = start_elements * element_size; i < (start_elements + main_elements) * element_size;
       i += VECTOR_LENGTH_IN_BYTES) {
    float32x4x2_t inout_val = cvt_bf16_to_fp32(vld1q_bf16((const bfloat16_t*)(buffers[0] + i)));
    for (int j = 1; j < world_size; j++) {
      const float32x4x2_t in_val = cvt_bf16_to_fp32(vld1q_bf16((const bfloat16_t*)(buffers[j] + i)));
      inout_val.val[0] = vaddq_f32(inout_val.val[0], in_val.val[0]);
      inout_val.val[1] = vaddq_f32(inout_val.val[1], in_val.val[1]);
    }
    vst1q_bf16((bfloat16_t*)(to_buffer + i), cvt_fp32_to_bf16(inout_val));
  }

  // process remaining part
  int i = (start_elements + main_elements) * element_size;
  while (remain_elements > 0) {
    float val = 0.0f;
    for (int j = 0; j < world_size; j++) {
      val += vcvtah_f32_bf16(*(bfloat16_t*)(buffers[j] + i));
    }
    *(bfloat16_t*)(to_buffer + i) = vcvth_bf16_f32(val);
    remain_elements--;
    i += element_size;
  }
```
**EN:** This section drives `cvt_bf16_to_fp32`, `vaddq_f32`, `vst1q_bf16` through branches and loops, deciding which execution path or data-handling strategy is used.
**CN:** 本段通过分支和循环推进`cvt_bf16_to_fp32`、`vaddq_f32`、`vst1q_bf16`相关逻辑，决定采用哪条执行路径或数据处理策略。

### Lines 48-67: Control flow and branching
```cpp
}

inline void reduce_fp16_buffers(int start_elements, int num_elements, char* to_buffer, char** buffers, int world_size) {
  const int element_size = 2;
  const int vector_length = VECTOR_LENGTH_IN_BYTES / element_size;
  int main_elements = num_elements - (num_elements % vector_length);
  int remain_elements = num_elements % vector_length;

  // process aligned part
#pragma omp parallel for
  for (int i = start_elements * element_size; i < (start_elements + main_elements) * element_size;
       i += VECTOR_LENGTH_IN_BYTES) {
    float16x8_t inout_val = vld1q_f16((const float16_t*)(buffers[0] + i));
    for (int j = 1; j < world_size; j++) {
      const float16x8_t in_val = vld1q_f16((const float16_t*)(buffers[j] + i));
      inout_val = vaddq_f16(inout_val, in_val);
    }
    vst1q_f16((float16_t*)(to_buffer + i), inout_val);
  }
```
**EN:** This section drives `reduce_fp16_buffers`, `vld1q_f16`, `vaddq_f16` through branches and loops, deciding which execution path or data-handling strategy is used.
**CN:** 本段通过分支和循环推进`reduce_fp16_buffers`、`vld1q_f16`、`vaddq_f16`相关逻辑，决定采用哪条执行路径或数据处理策略。

### Lines 68-86: Control flow and branching
```cpp
  // process remaining part
  int i = (start_elements + main_elements) * element_size;
  while (remain_elements > 0) {
    float16_t val = 0.0f;
    for (int j = 0; j < world_size; j++) {
      val = vaddh_f16(val, *(float16_t*)(buffers[j] + i));
    }
    *(float16_t*)(to_buffer + i) = val;
    remain_elements--;
    i += element_size;
  }
}

inline void reduce_fp32_buffers(int start_elements, int num_elements, char* to_buffer, char** buffers, int world_size) {
  const int element_size = 4;
  const int vector_length = VECTOR_LENGTH_IN_BYTES / element_size;
  int main_elements = num_elements - (num_elements % vector_length);
  int remain_elements = num_elements % vector_length;
```
**EN:** This section drives `reduce_fp32_buffers`, `vaddh_f16`, `element_size` through branches and loops, deciding which execution path or data-handling strategy is used.
**CN:** 本段通过分支和循环推进`reduce_fp32_buffers`、`vaddh_f16`、`element_size`相关逻辑，决定采用哪条执行路径或数据处理策略。

### Lines 87-109: Control flow and branching
```cpp
  // process aligned part
#pragma omp parallel for
  for (int i = start_elements * element_size; i < (start_elements + main_elements) * element_size;
       i += VECTOR_LENGTH_IN_BYTES) {
    float32x4_t inout_val = vld1q_f32((const float*)(buffers[0] + i));
    for (int j = 1; j < world_size; j++) {
      const float32x4_t in_val = vld1q_f32((const float*)(buffers[j] + i));
      inout_val = vaddq_f32(inout_val, in_val);
    }
    vst1q_f32((float32_t*)(to_buffer + i), inout_val);
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
```
**EN:** This section drives `vld1q_f32`, `vaddq_f32`, `vst1q_f32` through branches and loops, deciding which execution path or data-handling strategy is used.
**CN:** 本段通过分支和循环推进`vld1q_f32`、`vaddq_f32`、`vst1q_f32`相关逻辑，决定采用哪条执行路径或数据处理策略。

### Lines 110-126: Control flow and branching
```cpp
}

inline void parallel_memcpy(void* to, void* from, size_t n_bytes) {
  auto aligned_bytes = n_bytes - (n_bytes % VECTOR_LENGTH_IN_BYTES);
  // process aligned part
#pragma omp parallel for
  for (size_t i = 0; i < aligned_bytes; i += VECTOR_LENGTH_IN_BYTES) {
    const uint8x16_t val = vld1q_u8((uint8_t*)from + i);
    vst1q_u8((uint8_t*)to + i, val);
  }

  // process remaining part
  for (size_t i = aligned_bytes; i < n_bytes; i++) {
    *((uint8_t*)to + i) = *((uint8_t*)from + i);
  }
}
```
**EN:** This section drives `parallel_memcpy`, `vld1q_u8`, `vst1q_u8` through branches and loops, deciding which execution path or data-handling strategy is used.
**CN:** 本段通过分支和循环推进`parallel_memcpy`、`vld1q_u8`、`vst1q_u8`相关逻辑，决定采用哪条执行路径或数据处理策略。

### Lines 127-127: Local implementation details
```cpp
#undef VECTOR_LENGTH_IN_BYTES
```
**EN:** This section fills in the local implementation details around the surrounding logic, completing the behavior required by the file.
**CN:** 本段补充了相关逻辑周边的局部实现细节，以完成该文件所需的具体行为。

## Key Concepts / 关键概念
- **Low-level systems code / 底层系统代码**: Focuses on performance-sensitive implementation details close to the hardware. / 关注贴近硬件、对性能敏感的实现细节。

## Dependencies / 依赖关系
- **Internal headers / 内部头文件**: None explicitly listed / 未显式列出
- **External headers / 外部头文件**: `arm_neon.h`
- **Path context / 路径上下文**: cpu / aarch64 / shm.h
