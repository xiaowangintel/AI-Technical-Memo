# blake3_avx2.c — Code Analysis / 代码分析

## Source / 来源
- File: `llvm/lib/Support/BLAKE3/blake3_avx2.c`
- Repository: `llvm-project`
- Purpose (EN): Note that clang-format doesn't like the name "xor" for some reason.
- Purpose (CN): 该文件位于 LLVM 的 `Support/BLAKE3` 目录中，主要实现与 `blake3_avx2` 相关的接口、数据结构和辅助逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-40

```c
#include "blake3_impl.h"

#include <immintrin.h>

#define DEGREE 8

INLINE __m256i loadu(const uint8_t src[32]) {
  return _mm256_loadu_si256((const __m256i *)src);
}

INLINE void storeu(__m256i src, uint8_t dest[16]) {
  _mm256_storeu_si256((__m256i *)dest, src);
}

INLINE __m256i addv(__m256i a, __m256i b) { return _mm256_add_epi32(a, b); }

// Note that clang-format doesn't like the name "xor" for some reason.
INLINE __m256i xorv(__m256i a, __m256i b) { return _mm256_xor_si256(a, b); }

INLINE __m256i set1(uint32_t x) { return _mm256_set1_epi32((int32_t)x); }

INLINE __m256i rot16(__m256i x) {
  return _mm256_shuffle_epi8(
      x, _mm256_set_epi8(13, 12, 15, 14, 9, 8, 11, 10, 5, 4, 7, 6, 1, 0, 3, 2,
                         13, 12, 15, 14, 9, 8, 11, 10, 5, 4, 7, 6, 1, 0, 3, 2));
}

INLINE __m256i rot12(__m256i x) {
  return _mm256_or_si256(_mm256_srli_epi32(x, 12), _mm256_slli_epi32(x, 32 - 12));
}

INLINE __m256i rot8(__m256i x) {
  return _mm256_shuffle_epi8(
      x, _mm256_set_epi8(12, 15, 14, 13, 8, 11, 10, 9, 4, 7, 6, 5, 0, 3, 2, 1,
                         12, 15, 14, 13, 8, 11, 10, 9, 4, 7, 6, 5, 0, 3, 2, 1));
}

INLINE __m256i rot7(__m256i x) {
  return _mm256_or_si256(_mm256_srli_epi32(x, 7), _mm256_slli_epi32(x, 32 - 7));
}
```
- EN: Introduces the file banner, comments, and the initial setup for this translation unit.
  CN: 这一段给出文件横幅、注释说明以及该编译单元的初始设置。
- EN: Brings in 2 direct dependencies, including `blake3_impl.h`, `immintrin.h`.
  CN: 引入了 2 个直接依赖，其中包括 `blake3_impl.h`, `immintrin.h`。
- EN: This section centers on `loadu`, `_mm256_loadu_si256`, `storeu` and implements the local control flow and bookkeeping.
  CN: 这一段主要围绕 `loadu`, `_mm256_loadu_si256`, `storeu` 等符号展开，负责实现局部控制流程与状态维护。

### Lines 41-80

```c

INLINE void round_fn(__m256i v[16], __m256i m[16], size_t r) {
  v[0] = addv(v[0], m[(size_t)MSG_SCHEDULE[r][0]]);
  v[1] = addv(v[1], m[(size_t)MSG_SCHEDULE[r][2]]);
  v[2] = addv(v[2], m[(size_t)MSG_SCHEDULE[r][4]]);
  v[3] = addv(v[3], m[(size_t)MSG_SCHEDULE[r][6]]);
  v[0] = addv(v[0], v[4]);
  v[1] = addv(v[1], v[5]);
  v[2] = addv(v[2], v[6]);
  v[3] = addv(v[3], v[7]);
  v[12] = xorv(v[12], v[0]);
  v[13] = xorv(v[13], v[1]);
  v[14] = xorv(v[14], v[2]);
  v[15] = xorv(v[15], v[3]);
  v[12] = rot16(v[12]);
  v[13] = rot16(v[13]);
  v[14] = rot16(v[14]);
  v[15] = rot16(v[15]);
  v[8] = addv(v[8], v[12]);
  v[9] = addv(v[9], v[13]);
  v[10] = addv(v[10], v[14]);
  v[11] = addv(v[11], v[15]);
  v[4] = xorv(v[4], v[8]);
  v[5] = xorv(v[5], v[9]);
  v[6] = xorv(v[6], v[10]);
  v[7] = xorv(v[7], v[11]);
  v[4] = rot12(v[4]);
  v[5] = rot12(v[5]);
  v[6] = rot12(v[6]);
  v[7] = rot12(v[7]);
  v[0] = addv(v[0], m[(size_t)MSG_SCHEDULE[r][1]]);
  v[1] = addv(v[1], m[(size_t)MSG_SCHEDULE[r][3]]);
  v[2] = addv(v[2], m[(size_t)MSG_SCHEDULE[r][5]]);
  v[3] = addv(v[3], m[(size_t)MSG_SCHEDULE[r][7]]);
  v[0] = addv(v[0], v[4]);
  v[1] = addv(v[1], v[5]);
  v[2] = addv(v[2], v[6]);
  v[3] = addv(v[3], v[7]);
  v[12] = xorv(v[12], v[0]);
  v[13] = xorv(v[13], v[1]);
```
- EN: This section centers on `round_fn` and implements the local control flow and bookkeeping.
  CN: 这一段主要围绕 `round_fn` 等符号展开，负责实现局部控制流程与状态维护。

### Lines 81-120

```c
  v[14] = xorv(v[14], v[2]);
  v[15] = xorv(v[15], v[3]);
  v[12] = rot8(v[12]);
  v[13] = rot8(v[13]);
  v[14] = rot8(v[14]);
  v[15] = rot8(v[15]);
  v[8] = addv(v[8], v[12]);
  v[9] = addv(v[9], v[13]);
  v[10] = addv(v[10], v[14]);
  v[11] = addv(v[11], v[15]);
  v[4] = xorv(v[4], v[8]);
  v[5] = xorv(v[5], v[9]);
  v[6] = xorv(v[6], v[10]);
  v[7] = xorv(v[7], v[11]);
  v[4] = rot7(v[4]);
  v[5] = rot7(v[5]);
  v[6] = rot7(v[6]);
  v[7] = rot7(v[7]);

  v[0] = addv(v[0], m[(size_t)MSG_SCHEDULE[r][8]]);
  v[1] = addv(v[1], m[(size_t)MSG_SCHEDULE[r][10]]);
  v[2] = addv(v[2], m[(size_t)MSG_SCHEDULE[r][12]]);
  v[3] = addv(v[3], m[(size_t)MSG_SCHEDULE[r][14]]);
  v[0] = addv(v[0], v[5]);
  v[1] = addv(v[1], v[6]);
  v[2] = addv(v[2], v[7]);
  v[3] = addv(v[3], v[4]);
  v[15] = xorv(v[15], v[0]);
  v[12] = xorv(v[12], v[1]);
  v[13] = xorv(v[13], v[2]);
  v[14] = xorv(v[14], v[3]);
  v[15] = rot16(v[15]);
  v[12] = rot16(v[12]);
  v[13] = rot16(v[13]);
  v[14] = rot16(v[14]);
  v[10] = addv(v[10], v[15]);
  v[11] = addv(v[11], v[12]);
  v[8] = addv(v[8], v[13]);
  v[9] = addv(v[9], v[14]);
  v[5] = xorv(v[5], v[10]);
```
- EN: This range contains supporting statements, comments, or structural glue code.
  CN: 这一段主要包含辅助语句、注释或结构性胶水代码。

### Lines 121-160

```c
  v[6] = xorv(v[6], v[11]);
  v[7] = xorv(v[7], v[8]);
  v[4] = xorv(v[4], v[9]);
  v[5] = rot12(v[5]);
  v[6] = rot12(v[6]);
  v[7] = rot12(v[7]);
  v[4] = rot12(v[4]);
  v[0] = addv(v[0], m[(size_t)MSG_SCHEDULE[r][9]]);
  v[1] = addv(v[1], m[(size_t)MSG_SCHEDULE[r][11]]);
  v[2] = addv(v[2], m[(size_t)MSG_SCHEDULE[r][13]]);
  v[3] = addv(v[3], m[(size_t)MSG_SCHEDULE[r][15]]);
  v[0] = addv(v[0], v[5]);
  v[1] = addv(v[1], v[6]);
  v[2] = addv(v[2], v[7]);
  v[3] = addv(v[3], v[4]);
  v[15] = xorv(v[15], v[0]);
  v[12] = xorv(v[12], v[1]);
  v[13] = xorv(v[13], v[2]);
  v[14] = xorv(v[14], v[3]);
  v[15] = rot8(v[15]);
  v[12] = rot8(v[12]);
  v[13] = rot8(v[13]);
  v[14] = rot8(v[14]);
  v[10] = addv(v[10], v[15]);
  v[11] = addv(v[11], v[12]);
  v[8] = addv(v[8], v[13]);
  v[9] = addv(v[9], v[14]);
  v[5] = xorv(v[5], v[10]);
  v[6] = xorv(v[6], v[11]);
  v[7] = xorv(v[7], v[8]);
  v[4] = xorv(v[4], v[9]);
  v[5] = rot7(v[5]);
  v[6] = rot7(v[6]);
  v[7] = rot7(v[7]);
  v[4] = rot7(v[4]);
}

INLINE void transpose_vecs(__m256i vecs[DEGREE]) {
  // Interleave 32-bit lanes. The low unpack is lanes 00/11/44/55, and the high
  // is 22/33/66/77.
```
- EN: This section centers on `transpose_vecs` and implements the local control flow and bookkeeping.
  CN: 这一段主要围绕 `transpose_vecs` 等符号展开，负责实现局部控制流程与状态维护。

### Lines 161-200

```c
  __m256i ab_0145 = _mm256_unpacklo_epi32(vecs[0], vecs[1]);
  __m256i ab_2367 = _mm256_unpackhi_epi32(vecs[0], vecs[1]);
  __m256i cd_0145 = _mm256_unpacklo_epi32(vecs[2], vecs[3]);
  __m256i cd_2367 = _mm256_unpackhi_epi32(vecs[2], vecs[3]);
  __m256i ef_0145 = _mm256_unpacklo_epi32(vecs[4], vecs[5]);
  __m256i ef_2367 = _mm256_unpackhi_epi32(vecs[4], vecs[5]);
  __m256i gh_0145 = _mm256_unpacklo_epi32(vecs[6], vecs[7]);
  __m256i gh_2367 = _mm256_unpackhi_epi32(vecs[6], vecs[7]);

  // Interleave 64-bit lanes. The low unpack is lanes 00/22 and the high is
  // 11/33.
  __m256i abcd_04 = _mm256_unpacklo_epi64(ab_0145, cd_0145);
  __m256i abcd_15 = _mm256_unpackhi_epi64(ab_0145, cd_0145);
  __m256i abcd_26 = _mm256_unpacklo_epi64(ab_2367, cd_2367);
  __m256i abcd_37 = _mm256_unpackhi_epi64(ab_2367, cd_2367);
  __m256i efgh_04 = _mm256_unpacklo_epi64(ef_0145, gh_0145);
  __m256i efgh_15 = _mm256_unpackhi_epi64(ef_0145, gh_0145);
  __m256i efgh_26 = _mm256_unpacklo_epi64(ef_2367, gh_2367);
  __m256i efgh_37 = _mm256_unpackhi_epi64(ef_2367, gh_2367);

  // Interleave 128-bit lanes.
  vecs[0] = _mm256_permute2x128_si256(abcd_04, efgh_04, 0x20);
  vecs[1] = _mm256_permute2x128_si256(abcd_15, efgh_15, 0x20);
  vecs[2] = _mm256_permute2x128_si256(abcd_26, efgh_26, 0x20);
  vecs[3] = _mm256_permute2x128_si256(abcd_37, efgh_37, 0x20);
  vecs[4] = _mm256_permute2x128_si256(abcd_04, efgh_04, 0x31);
  vecs[5] = _mm256_permute2x128_si256(abcd_15, efgh_15, 0x31);
  vecs[6] = _mm256_permute2x128_si256(abcd_26, efgh_26, 0x31);
  vecs[7] = _mm256_permute2x128_si256(abcd_37, efgh_37, 0x31);
}

INLINE void transpose_msg_vecs(const uint8_t *const *inputs,
                               size_t block_offset, __m256i out[16]) {
  out[0] = loadu(&inputs[0][block_offset + 0 * sizeof(__m256i)]);
  out[1] = loadu(&inputs[1][block_offset + 0 * sizeof(__m256i)]);
  out[2] = loadu(&inputs[2][block_offset + 0 * sizeof(__m256i)]);
  out[3] = loadu(&inputs[3][block_offset + 0 * sizeof(__m256i)]);
  out[4] = loadu(&inputs[4][block_offset + 0 * sizeof(__m256i)]);
  out[5] = loadu(&inputs[5][block_offset + 0 * sizeof(__m256i)]);
  out[6] = loadu(&inputs[6][block_offset + 0 * sizeof(__m256i)]);
```
- EN: This section centers on `transpose_msg_vecs` and implements the local control flow and bookkeeping.
  CN: 这一段主要围绕 `transpose_msg_vecs` 等符号展开，负责实现局部控制流程与状态维护。

### Lines 201-240

```c
  out[7] = loadu(&inputs[7][block_offset + 0 * sizeof(__m256i)]);
  out[8] = loadu(&inputs[0][block_offset + 1 * sizeof(__m256i)]);
  out[9] = loadu(&inputs[1][block_offset + 1 * sizeof(__m256i)]);
  out[10] = loadu(&inputs[2][block_offset + 1 * sizeof(__m256i)]);
  out[11] = loadu(&inputs[3][block_offset + 1 * sizeof(__m256i)]);
  out[12] = loadu(&inputs[4][block_offset + 1 * sizeof(__m256i)]);
  out[13] = loadu(&inputs[5][block_offset + 1 * sizeof(__m256i)]);
  out[14] = loadu(&inputs[6][block_offset + 1 * sizeof(__m256i)]);
  out[15] = loadu(&inputs[7][block_offset + 1 * sizeof(__m256i)]);
  for (size_t i = 0; i < 8; ++i) {
    _mm_prefetch((const void *)&inputs[i][block_offset + 256], _MM_HINT_T0);
  }
  transpose_vecs(&out[0]);
  transpose_vecs(&out[8]);
}

INLINE void load_counters(uint64_t counter, bool increment_counter,
                          __m256i *out_lo, __m256i *out_hi) {
  const __m256i mask = _mm256_set1_epi32(-(int32_t)increment_counter);
  const __m256i add0 = _mm256_set_epi32(7, 6, 5, 4, 3, 2, 1, 0);
  const __m256i add1 = _mm256_and_si256(mask, add0);
  __m256i l = _mm256_add_epi32(_mm256_set1_epi32((int32_t)counter), add1);
  __m256i carry = _mm256_cmpgt_epi32(_mm256_xor_si256(add1, _mm256_set1_epi32(0x80000000)), 
                                     _mm256_xor_si256(   l, _mm256_set1_epi32(0x80000000)));
  __m256i h = _mm256_sub_epi32(_mm256_set1_epi32((int32_t)(counter >> 32)), carry);
  *out_lo = l;
  *out_hi = h;
}

static
void blake3_hash8_avx2(const uint8_t *const *inputs, size_t blocks,
                       const uint32_t key[8], uint64_t counter,
                       bool increment_counter, uint8_t flags,
                       uint8_t flags_start, uint8_t flags_end, uint8_t *out) {
  __m256i h_vecs[8] = {
      set1(key[0]), set1(key[1]), set1(key[2]), set1(key[3]),
      set1(key[4]), set1(key[5]), set1(key[6]), set1(key[7]),
  };
  __m256i counter_low_vec, counter_high_vec;
  load_counters(counter, increment_counter, &counter_low_vec,
```
- EN: This section centers on `_mm_prefetch`, `transpose_vecs`, `load_counters` and performs utility computation and state updates.
  CN: 这一段主要围绕 `_mm_prefetch`, `transpose_vecs`, `load_counters` 等符号展开，负责执行工具性计算并更新状态。
- EN: This range iterates over collections, ranges, or records.
  CN: 这一段遍历集合、区间或记录。

### Lines 241-280

```c
                &counter_high_vec);
  uint8_t block_flags = flags | flags_start;

  for (size_t block = 0; block < blocks; block++) {
    if (block + 1 == blocks) {
      block_flags |= flags_end;
    }
    __m256i block_len_vec = set1(BLAKE3_BLOCK_LEN);
    __m256i block_flags_vec = set1(block_flags);
    __m256i msg_vecs[16];
    transpose_msg_vecs(inputs, block * BLAKE3_BLOCK_LEN, msg_vecs);

    __m256i v[16] = {
        h_vecs[0],       h_vecs[1],        h_vecs[2],     h_vecs[3],
        h_vecs[4],       h_vecs[5],        h_vecs[6],     h_vecs[7],
        set1(IV[0]),     set1(IV[1]),      set1(IV[2]),   set1(IV[3]),
        counter_low_vec, counter_high_vec, block_len_vec, block_flags_vec,
    };
    round_fn(v, msg_vecs, 0);
    round_fn(v, msg_vecs, 1);
    round_fn(v, msg_vecs, 2);
    round_fn(v, msg_vecs, 3);
    round_fn(v, msg_vecs, 4);
    round_fn(v, msg_vecs, 5);
    round_fn(v, msg_vecs, 6);
    h_vecs[0] = xorv(v[0], v[8]);
    h_vecs[1] = xorv(v[1], v[9]);
    h_vecs[2] = xorv(v[2], v[10]);
    h_vecs[3] = xorv(v[3], v[11]);
    h_vecs[4] = xorv(v[4], v[12]);
    h_vecs[5] = xorv(v[5], v[13]);
    h_vecs[6] = xorv(v[6], v[14]);
    h_vecs[7] = xorv(v[7], v[15]);

    block_flags = flags;
  }

  transpose_vecs(h_vecs);
  storeu(h_vecs[0], &out[0 * sizeof(__m256i)]);
  storeu(h_vecs[1], &out[1 * sizeof(__m256i)]);
```
- EN: This section centers on `transpose_msg_vecs`, `round_fn`, `transpose_vecs` and implements the local control flow and bookkeeping.
  CN: 这一段主要围绕 `transpose_msg_vecs`, `round_fn`, `transpose_vecs` 等符号展开，负责实现局部控制流程与状态维护。
- EN: In this range, the code iterates over collections, ranges, or records and checks conditions and handles edge cases.
  CN: 在这一段中，代码遍历集合、区间或记录，并检查条件并处理边界情况。

### Lines 281-320

```c
  storeu(h_vecs[2], &out[2 * sizeof(__m256i)]);
  storeu(h_vecs[3], &out[3 * sizeof(__m256i)]);
  storeu(h_vecs[4], &out[4 * sizeof(__m256i)]);
  storeu(h_vecs[5], &out[5 * sizeof(__m256i)]);
  storeu(h_vecs[6], &out[6 * sizeof(__m256i)]);
  storeu(h_vecs[7], &out[7 * sizeof(__m256i)]);
}

#if !defined(BLAKE3_NO_SSE41)
void blake3_hash_many_sse41(const uint8_t *const *inputs, size_t num_inputs,
                            size_t blocks, const uint32_t key[8],
                            uint64_t counter, bool increment_counter,
                            uint8_t flags, uint8_t flags_start,
                            uint8_t flags_end, uint8_t *out);
#else
void blake3_hash_many_portable(const uint8_t *const *inputs, size_t num_inputs,
                               size_t blocks, const uint32_t key[8],
                               uint64_t counter, bool increment_counter,
                               uint8_t flags, uint8_t flags_start,
                               uint8_t flags_end, uint8_t *out);
#endif

void blake3_hash_many_avx2(const uint8_t *const *inputs, size_t num_inputs,
                           size_t blocks, const uint32_t key[8],
                           uint64_t counter, bool increment_counter,
                           uint8_t flags, uint8_t flags_start,
                           uint8_t flags_end, uint8_t *out) {
  while (num_inputs >= DEGREE) {
    blake3_hash8_avx2(inputs, blocks, key, counter, increment_counter, flags,
                      flags_start, flags_end, out);
    if (increment_counter) {
      counter += DEGREE;
    }
    inputs += DEGREE;
    num_inputs -= DEGREE;
    out = &out[DEGREE * BLAKE3_OUT_LEN];
  }
#if !defined(BLAKE3_NO_SSE41)
  blake3_hash_many_sse41(inputs, num_inputs, blocks, key, counter,
                         increment_counter, flags, flags_start, flags_end, out);
```
- EN: This section centers on `storeu`, `blake3_hash_many_sse41`, `blake3_hash_many_portable` and performs utility computation and state updates.
  CN: 这一段主要围绕 `storeu`, `blake3_hash_many_sse41`, `blake3_hash_many_portable` 等符号展开，负责执行工具性计算并更新状态。
- EN: In this range, the code iterates over collections, ranges, or records and checks conditions and handles edge cases.
  CN: 在这一段中，代码遍历集合、区间或记录，并检查条件并处理边界情况。

### Lines 321-326

```c
#else
  blake3_hash_many_portable(inputs, num_inputs, blocks, key, counter,
                            increment_counter, flags, flags_start, flags_end,
                            out);
#endif
}
```
- EN: This section centers on `blake3_hash_many_portable` and performs utility computation and state updates.
  CN: 这一段主要围绕 `blake3_hash_many_portable` 等符号展开，负责执行工具性计算并更新状态。

## Key Concepts / 关键概念
- Domain / 领域: LLVM support utilities / LLVM 支撑工具
- Core symbols / 核心符号: `loadu`, `_mm256_loadu_si256`, `storeu`, `_mm256_storeu_si256` / 该文件围绕这些类型或函数组织主要逻辑。
- Data flow / 数据流: Formatting, emission, and outward serialization. / 重点关注格式化、输出与序列化。
- Integration / 集成关系: Connects file-local logic to adjacent LLVM headers, support types, and subsystems. / 将文件内逻辑连接到相邻的 LLVM 头文件、支撑类型与子系统。

## Dependencies / 依赖关系
- LLVM headers / LLVM 头文件: None / 无
- Standard library / 标准库: None / 无
- Other/system headers / 其他或系统头文件: `blake3_impl.h`, `immintrin.h`
- Related symbols / 相关符号: `loadu`, `_mm256_loadu_si256`, `storeu`, `_mm256_storeu_si256`, `addv`
