# cpp_flex_attention_template.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `torch/_inductor/codegen/cpp_flex_attention_template.py`
- **Repository / 仓库**: `pytorch/pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: This module generates backend-specific code for lowered kernels. It defines classes such as `CppFlexAttentionTemplate`.
- **用途（中文）**: 该模块为下沉后的内核生成后端相关代码。其中定义了 `CppFlexAttentionTemplate` 等类。

## Line-by-Line Analysis / 逐行分析
### Lines 1-28 / 第 1-28 行
````python
# mypy: allow-untyped-defs
import contextlib
import logging
import re
from unittest.mock import patch

import sympy

import torch
import torch.utils

from ...utils._ordered_set import OrderedSet
from .. import ir
from ..ir import TensorBox
from ..select_algorithm import DataProcessorTemplateWrapper
from ..utils import parallel_num_threads
from ..virtualized import V
from .cpp_template import CppTemplate
from .cpp_utils import GemmBlocking


log = logging.getLogger(__name__)

# TODO: reuse cpp codegen to generate below pointwise/reduction kernels
SOFTMAX_FUSIONS = r"""
// 1) out = exp(a - val)
// 2) val = sum(out)
template <typename T1, typename T2>
````
- **EN**: Imports dependencies such as `contextlib`, `logging`, `re`, `unittest.mock`, `sympy`, `torch`, and `...+9` for the logic in this range. Uses symbolic expressions to reason about indices, shapes, or algebraic properties. Initializes or updates values such as `log`, and `SOFTMAX_FUSIONS`.
- **CN**: 这里导入了 `contextlib`、`logging`、`re`、`unittest.mock`、`sympy`、`torch`、`另有9项` 等依赖，为后续逻辑提供基础能力。借助符号表达式来推理索引、形状或代数性质。初始化或更新了 `log`、`SOFTMAX_FUSIONS` 等值。

### Lines 29-56 / 第 29-56 行
````python
inline void {{kernel_name}}_exp_reduce_sum_fusion_kernel(
    T1* a,
    const int& size,
    T2* out,
    T1& val) {
  auto vec_size = at::vec::Vectorized<T1>::size();
  auto vec_max = at::vec::Vectorized<T1>(val);
  T1 tmp_sum = 0;
  auto vec_tmp_sum = at::vec::Vectorized<T1>(tmp_sum);
  for (long i = 0; i < vec_size * (size / vec_size); i += vec_size) {
    auto tmp0 = at::vec::Vectorized<T1>::loadu(a + i);
    auto tmp1 = tmp0 - vec_max;
    auto tmp2 = tmp1.exp_u20();
    vec_tmp_sum += tmp2;
    at::native::_store(out + i, tmp2);
  }
  tmp_sum = at::vec::vec_reduce_all<T1>(
      [](at::vec::Vectorized<T1>& x, at::vec::Vectorized<T1>& y) {
        return x + y;
      },
      vec_tmp_sum);
  for (long i = vec_size * (size / vec_size); i < size; i++) {
    auto tmp0 = a[i];
    auto tmp1 = tmp0 - val;
    auto tmp2 = exp(tmp1);
    tmp_sum += tmp2;
    out[i] = tmp2;
  }
````
- **EN**: Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `at`, and `tmp_sum`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `at`、`tmp_sum` 等值。

### Lines 57-84 / 第 57-84 行
````python
  val = tmp_sum;
}

// 1) out = a * scale
// 2) max = max(out)
template <typename scalar_t>
inline void {{kernel_name}}_mul_reduce_max_fusion_kernel(
    const scalar_t* a,
    const scalar_t& scale,
    const int& size,
    scalar_t* out,
    scalar_t& max) {
  auto vec_size = at::vec::Vectorized<scalar_t>::size();
  auto vec_scale = at::vec::Vectorized<scalar_t>(scale);
  scalar_t tmp_max = -std::numeric_limits<scalar_t>::infinity();
  auto vec_tmp_max = at::vec::Vectorized<scalar_t>(tmp_max);
  for (long i = 0; i < vec_size * (size / vec_size); i += vec_size) {
    auto tmp0 = at::vec::Vectorized<scalar_t>::loadu(a + i);
    auto tmp1 = tmp0 * vec_scale;
    vec_tmp_max = at::vec::maximum(vec_tmp_max, tmp1);
    at::native::_store(out + i, tmp1);
  }
  for (long i = vec_size * (size / vec_size); i < size; i++) {
    auto tmp0 = a[i];
    auto tmp1 = tmp0 * scale;
    tmp_max = std::max(tmp_max, tmp1);
    out[i] = tmp1;
  }
````
- **EN**: Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `val`, `vec_tmp_max`, `at`, and `tmp_max`.
- **CN**: 包含分支、循环或上下文管理等控制流。初始化或更新了 `val`、`vec_tmp_max`、`at`、`tmp_max` 等值。

### Lines 85-112 / 第 85-112 行
````python
  max = std::max(
      tmp_max,
      at::vec::vec_reduce_all<scalar_t>(
          [](at::vec::Vectorized<scalar_t>& x, at::vec::Vectorized<scalar_t>& y) {
            return at::vec::maximum(x, y);
          },
          vec_tmp_max));
}

template <typename scalar_t>
static inline scalar_t* {{kernel_name}}_conditional_data_ptr(scalar_t* ptr, scalar_t* ptr2) {
  TORCH_CHECK(ptr2 == nullptr);
  return ptr;
}

template <typename scalar_t,
          typename std::enable_if_t<c10::is_reduced_floating_point_v<scalar_t>, int> = 0>
static inline scalar_t* {{kernel_name}}_conditional_data_ptr(float* ptr, scalar_t* ptr2) {
  return ptr2;
}

template <typename scalar_t>
inline void {{kernel_name}}_fill_stub(scalar_t* data, scalar_t val, int64_t size) {
  using Vec = at::vec::Vectorized<scalar_t>;
  Vec data_vec = Vec(val);
  int64_t d = 0;
  for (; d < size - (size % Vec::size()); d += Vec::size()) {
    data_vec.store(data + d);
````
- **EN**: Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `max`, and `at`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `max`、`at` 等值。

### Lines 113-140 / 第 113-140 行
````python
  }
  #if !defined(_MSC_VER) && !defined(COMPILING_FOR_MIN_SIZE)
  # pragma unroll
  #endif
  for (; d < size; d++) {
    data[d] = val;
  }
}

// out = a * scale
template <typename scalar_t>
inline void {{kernel_name}}_mul_scale_kernel(
    scalar_t* a,
    scalar_t scale,
    int64_t size) {
  auto vec_size = at::vec::Vectorized<scalar_t>::size();
  auto vec_scale = at::vec::Vectorized<scalar_t>(scale);
  for (int64_t i = 0; i < vec_size * (size / vec_size); i += vec_size) {
    auto tmp0 = at::vec::Vectorized<scalar_t>::loadu(a + i);
    auto tmp1 = tmp0 * vec_scale;
    at::native::_store(a + i, tmp1);
  }
  for (int64_t i = vec_size * (size / vec_size); i < size; i++) {
    auto tmp0 = a[i];
    auto tmp1 = tmp0 * scale;
    a[i] = tmp1;
  }
}
````
- **EN**: Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `at`.
- **CN**: 包含分支、循环或上下文管理等控制流。初始化或更新了 `at` 等值。

### Lines 141-168 / 第 141-168 行
````python

"""

BRGEMM_PACK_FUNCTIONS = r"""
template <typename scalar_t>
inline void {{kernel_name}}_copy_value_with_pad(
    const scalar_t* value_ptr,
    scalar_t* dst_ptr,
    int64_t rows,
    int64_t cols,
    int64_t prows,
    int64_t pcols,
    int64_t ldi) {
  auto vec_size = at::vec::Vectorized<scalar_t>::size();
  int64_t i = 0;
  for (; i < rows; i++) {
    int64_t j = 0;
    for (; j < cols - (cols % vec_size); j += vec_size) {
      auto vec_v =
          at::vec::Vectorized<scalar_t>::loadu(value_ptr + i * ldi + j);
      vec_v.store(dst_ptr + i * pcols + j);
    }

    if (j < cols) {
      auto vec_v = at::vec::Vectorized<scalar_t>::loadu(
          value_ptr + i * ldi + j, cols - j);
      vec_v.store(dst_ptr + i * pcols + j, cols - j);
    }
````
- **EN**: Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `BRGEMM_PACK_FUNCTIONS`, and `at`.
- **CN**: 包含分支、循环或上下文管理等控制流。初始化或更新了 `BRGEMM_PACK_FUNCTIONS`、`at` 等值。

### Lines 169-196 / 第 169-196 行
````python

    // col padding
    auto psize = pcols - cols;
    if (psize > 0) {
      auto zero_vec = at::vec::Vectorized<scalar_t>(0);
      int64_t pj = 0;
      for (; pj < psize - (psize % vec_size); pj += vec_size) {
        zero_vec.store(dst_ptr + i * pcols + cols + pj);
      }
      if (pj < psize) {
        zero_vec.store(dst_ptr + i * pcols + cols + pj, psize - pj);
      }
    }
  }
  // row padding
  for (; i < prows; i++) {
    auto zero_vec = at::vec::Vectorized<scalar_t>(0);
    int64_t j = 0;
    for (; j < pcols - (pcols % vec_size); j += vec_size) {
      zero_vec.store(dst_ptr + i * pcols + j);
    }
    if (j < pcols) {
      zero_vec.store(dst_ptr + i * pcols + j, pcols - j);
    }

  }
}
"""
````
- **EN**: Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 包含分支、循环或上下文管理等控制流。

### Lines 197-224 / 第 197-224 行
````python

MICRO_GEMM_TEMPLATE = r"""
GEMM_DEFINE
"""

ALLOCATE_BUFFER = r"""
  int64_t {{buffer_name}}_dtype_itemsize = c10::is_reduced_floating_point_v<{{buffer_dtype}}> ? 2 : 4;
  auto& {{buffer_name}}_allocator = *at::getCPUAllocator();
  auto {{buffer_name}}_work_data = {{buffer_name}}_allocator.allocate({{buffer_size}}*{{buffer_name}}_dtype_itemsize);
  void* {{buffer_name}}_data_ptr = {{buffer_name}}_work_data.get();
  {{buffer_dtype}}* {{buffer_name}} = ({{buffer_dtype}}*){{buffer_name}}_data_ptr;
"""

FLEX_ATTENTION_TEMPLATE = r"""
{{template.header().getvalue()}}
#include <ATen/native/cpu/utils.h>
#include <ATen/native/CPUBlas.h>
#include <ATen/Context.h>
{{template.codegen_micro_gemm(kernel.kernel_name)}}
{{template.codegen_softmax_fusion(kernel.kernel_name)}}
{{template.codegen_brgemm_pack_function(kernel.kernel_name)}}
{%- set kernel_args = {"query": query, "key": key, "value": value,
                       "kv_num_blocks": kv_num_blocks, "kv_indices": kv_indices,
                       "full_kv_num_blocks": full_kv_num_blocks, "full_kv_indices": full_kv_indices } %}
{%- set kernel_args = template.update_kernel_args(kernel_args) %}

extern "C"
{{kernel.def_kernel(inputs=kernel_args, outputs={"output": output}, extra_sizevars=template.extra_sizevars)}}
````
- **EN**: Handles device-specific behavior, capabilities, or dispatch decisions. Initializes or updates values such as `MICRO_GEMM_TEMPLATE`, `ALLOCATE_BUFFER`, and `FLEX_ATTENTION_TEMPLATE`.
- **CN**: 处理与设备相关的行为、能力信息或派发决策。初始化或更新了 `MICRO_GEMM_TEMPLATE`、`ALLOCATE_BUFFER`、`FLEX_ATTENTION_TEMPLATE` 等值。

### Lines 225-252 / 第 225-252 行
````python
{
  {{ kernel.maybe_codegen_profile() }}
  int64_t qBlockSize = {{qBlockSize}};
  int64_t kvBlockSize = {{kvBlockSize}};
  int64_t num_thread = {{num_thread}};

  // dtypes of kernel and internal buffers
  using scalar_t = {{kernel.dtype(query)}};
  constexpr bool is_reduced_type = c10::is_reduced_floating_point_v<scalar_t>;
  using accum_t = at::opmath_type<{{kernel.dtype(query)}}>;
  using Vec = at::vec::Vectorized<accum_t>;
  accum_t scaling_factor = {{scale}};
  int64_t batchSize = {{kernel.size(query, 0)}};
  int64_t qSize = {{kernel.size(query, 1)}};
  int64_t num_head = {{kernel.size(query, 2)}};
  int64_t headSize = {{kernel.size(query, 3)}};
  int64_t batchSize_k = {{kernel.size(key, 0)}};
  int64_t num_head_k = {{kernel.size(key, 2)}};
  int64_t headSize_v = {{kernel.size(value, 3)}};
  bool is_broadcast_bs_kv = batchSize != batchSize_k;
  bool is_broadcast_head_kv = num_head != num_head_k;
  int64_t gqa_shards = num_head / num_head_k;
  int64_t bs_shards = batchSize / batchSize_k;

  int64_t batchSize_kvi = {{kernel.size(kv_indices, 0)}};
  int64_t num_head_kvi = {{kernel.size(kv_indices, 1)}};
  int64_t block_num_kvi = {{kernel.size(kv_indices, 3)}};
  bool is_broadcast_bs_kvi = batchSize != batchSize_kvi;
````
- **EN**: Tracks dtype-specific behavior or constants that affect numeric execution.
- **CN**: 追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。

### Lines 253-280 / 第 253-280 行
````python
  bool is_broadcast_head_kvi = num_head != num_head_kvi;
  int64_t gqa_shards_kvi = num_head / num_head_kvi;
  int64_t bs_shards_kvi = batchSize / batchSize_kvi;

  int64_t kviStrideB = {{kernel.stride(kv_indices, 0)}};
  int64_t kviStrideH = {{kernel.stride(kv_indices, 1)}};
  int64_t kviStrideQ = {{kernel.stride(kv_indices, 2)}};

  int64_t num_kviStrideB = {{kernel.stride(kv_num_blocks, 0)}};
  int64_t num_kviStrideH = {{kernel.stride(kv_num_blocks, 1)}};

{%- if has_full_kv_block %}
  int64_t full_kviStrideB = {{kernel.stride(full_kv_indices, 0)}};
  int64_t full_kviStrideH = {{kernel.stride(full_kv_indices, 1)}};
  int64_t full_kviStrideQ = {{kernel.stride(full_kv_indices, 2)}};

  int64_t full_num_kviStrideB = {{kernel.stride(full_kv_num_blocks, 0)}};
  int64_t full_num_kviStrideH = {{kernel.stride(full_kv_num_blocks, 1)}};
  auto full_kv_indices_data = full_kv_indices;
  auto full_kv_num_blocks_data = full_kv_num_blocks;
{%- endif %}

  auto kv_num_blocks_data = kv_num_blocks;
  auto kv_indices_data = kv_indices;

  // Strides
  int64_t qStrideB = {{kernel.stride(query, 0)}};
  int64_t qStrideM = {{kernel.stride(query, 1)}};
````
- **EN**: Implements the local expressions and calls needed for this part of the module.
- **CN**: 实现了该模块这一部分所需的局部表达式与调用逻辑。

### Lines 281-308 / 第 281-308 行
````python
  int64_t qStrideH = {{kernel.stride(query, 2)}};
  int64_t kStrideB = {{kernel.stride(key, 0)}};
  int64_t kStrideN = {{kernel.stride(key, 1)}};
  int64_t kStrideH = {{kernel.stride(key, 2)}};
  int64_t vStrideB = {{kernel.stride(value, 0)}};
  int64_t vStrideN = {{kernel.stride(value, 1)}};
  int64_t vStrideH = {{kernel.stride(value, 2)}};
  int64_t oStrideB = {{kernel.stride(output, 0)}};
  int64_t oStrideM = {{kernel.stride(output, 2)}};
  int64_t oStrideH = {{kernel.stride(output, 1)}};

  int64_t kvSize = {{kernel.size(key, 1)}};

  int64_t qSplitSize = qBlockSize;
  int64_t kvSplitSize = kvBlockSize;


  qSplitSize = qSplitSize > qSize ? qSize : qSplitSize;
  kvSplitSize = kvSplitSize > kvSize ? kvSize : kvSplitSize;
  int64_t qSlice = (qSize + qSplitSize - 1) / qSplitSize;
  int64_t kvSlice = (kvSize + kvSplitSize - 1) / kvSplitSize;
  int64_t kvTail = (kvSize - 1) % kvSplitSize + 1;

  bool need_pack = false;
  // Whether pack is needed for BFloat16/Half
  if (is_reduced_type) {
    // check platform ability
    need_pack = std::is_same_v<scalar_t, at::BFloat16> ? at::native::cpublas::could_pack(at::kBFloat16)
````
- **EN**: Tracks dtype-specific behavior or constants that affect numeric execution. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `qSplitSize`, `kvSplitSize`, and `need_pack`.
- **CN**: 追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含分支、循环或上下文管理等控制流。初始化或更新了 `qSplitSize`、`kvSplitSize`、`need_pack` 等值。

### Lines 309-336 / 第 309-336 行
````python
                                                       : at::native::cpublas::could_pack(at::kHalf);
  }
  if (need_pack) {
    // When the number of gemm is greater than the number of pack,
    // the pack overhead can be overlapped.
    int64_t thresh_size = 64;
    need_pack = kvSize >= thresh_size && qSize >= thresh_size;
    if (need_pack) {
      double pack_size = batchSize * num_head * kvSize * headSize;
      double qs_per_thread = (batchSize * num_head * qSlice + num_thread - 1) / num_thread;
      double gemm_size_per_thread = qs_per_thread * qSplitSize * kvSize * headSize;
      need_pack = gemm_size_per_thread / pack_size >= 4;
    }
  }
  // Pad is needed for packing when K is not even
  bool headSize_even = headSize % 2 == 0;
  int64_t eheadSize = need_pack && !headSize_even ? headSize + 1: headSize;
  int64_t ekvSplitSize = need_pack && (kvSplitSize % 2 != 0) ? kvSplitSize + 1 : kvSplitSize;
  int64_t ekvTail = need_pack && (kvTail % 2 != 0) ? kvTail + 1 : kvTail;
  int64_t kv_padding_size = (kvSize - 1) / kvSplitSize * ekvSplitSize + ekvTail;

  // Allocate per thread temp buf (accumulate type)
  int64_t _size_per_thread =
      /* qk     */ qSplitSize * kvSplitSize +
      /* qk_max */ qSplitSize +
      /* qk_sum */ qSplitSize +
      /* dst    */ qSplitSize * headSize_v;

````
- **EN**: Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `need_pack`.
- **CN**: 包含分支、循环或上下文管理等控制流。初始化或更新了 `need_pack` 等值。

### Lines 337-364 / 第 337-364 行
````python
  // Inputs/outputs buffers
  const scalar_t* q_data = query;
  const scalar_t* k_data = key;
  const scalar_t* v_data = value;
  scalar_t* out_data = output;

  // Buffers to store accum results, padding query and transpose/packing key/value
  {{template.codegen_allocate_buffer("buf_data", "accum_t", "num_thread*_size_per_thread")}}
  {{template.codegen_allocate_buffer("buf_reduced_data", "scalar_t", "num_thread*qSplitSize*ekvSplitSize")}}
  {{template.codegen_allocate_buffer("key_reorder_ptr", "scalar_t", "batchSize_k*num_head_k*eheadSize*kvSize")}}
  {{template.codegen_allocate_buffer("value_reorder_ptr", "scalar_t", "batchSize_k*num_head_k*kv_padding_size*headSize_v")}}
  {{template.codegen_allocate_buffer("transpose_buffer_ptr", "scalar_t", "num_thread*kvSplitSize*headSize")}}
  {{template.codegen_allocate_buffer("query_padding_ptr", "scalar_t", "num_thread*qSplitSize*eheadSize")}}
  if (need_pack) {
    // Pack K, V
    at::parallel_for(0, batchSize_k * num_head_k * kvSlice, 1, [&](int64_t begin, int64_t end) {
      int ompIdx = at::get_thread_num();
      int64_t i = 0, j = 0, l = 0, n = 0;
      scalar_t* transpose_ptr = need_pack? transpose_buffer_ptr + ompIdx * kvSplitSize * headSize : nullptr;
      at::native::data_index_init(begin, i, batchSize_k, j, num_head_k, l, kvSlice);
      for ([[maybe_unused]] auto z : c10::irange(begin, end)) {
        n = l * kvSplitSize;
        int64_t cur_kvSplitSize = std::min(kvSplitSize, kvSize - n);
        auto k_addr =
              k_data + i * kStrideB + j * kStrideH + n * kStrideN;
        auto v_addr =
              v_data + i * vStrideB + j * vStrideH + n * vStrideN;
        // transpose [cur_kvSplitSize, headSize] -> [headSize, cur_kvSplitSize]
````
- **EN**: Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `at`, and `n`.
- **CN**: 包含分支、循环或上下文管理等控制流。初始化或更新了 `at`、`n` 等值。

### Lines 365-392 / 第 365-392 行
````python
        at::native::utils::transpose<uint16_t>(
          cur_kvSplitSize,
          headSize,
          /* src_ptr */
          reinterpret_cast<const uint16_t*>(k_addr),
          /* ld_src */ kStrideN,
          /* dst */ reinterpret_cast<uint16_t*>(transpose_ptr),
          /* ld_dst */ cur_kvSplitSize);

        // Pack [headSize, cur_kvSplitSize]
        at::vec::pack_vnni2(
          /* src */ reinterpret_cast<const uint16_t*>(transpose_ptr),
          /* dst */ reinterpret_cast<uint16_t*>(key_reorder_ptr + i * num_head_k * eheadSize * kvSize +
                  j * eheadSize * kvSize + n * eheadSize),
          /* ld_src */ cur_kvSplitSize,
          /* K */ headSize,
          /* N */ cur_kvSplitSize);

        // Pack [cur_kvSplitSize, headSize_v]
        at::vec::pack_vnni2(
          /* src */ reinterpret_cast<const uint16_t*>(v_addr),
          /* dst */ reinterpret_cast<uint16_t*>(value_reorder_ptr +
                  i * num_head_k * kv_padding_size * headSize_v +
                  j * kv_padding_size * headSize_v + n * headSize_v),
          /* ld_src */ vStrideN,
          /* K */ cur_kvSplitSize,
          /* N */ headSize_v);
      // Move to the next query
````
- **EN**: Initializes or updates values such as `at`.
- **CN**: 初始化或更新了 `at` 等值。

### Lines 393-420 / 第 393-420 行
````python
      at::native::data_index_step(i, batchSize_k, j, num_head_k, l, kvSlice);
      }
    });
  }
  // Attention loop below
  at::parallel_for(0, batchSize * num_head * qSlice, 1, [&](int64_t begin, int64_t end) {
    int64_t i = 0, j = 0, k = 0;
    at::native::data_index_init(begin, i, batchSize, j, num_head, k, qSlice);
    int ompIdx = at::get_thread_num();
    accum_t* buf_ptr = buf_data + ompIdx * _size_per_thread;
    accum_t* qk_data = buf_ptr;
    accum_t* qk_max_data = qk_data + qSplitSize * kvSplitSize;
    accum_t* qk_sum_data = qk_max_data + qSplitSize;
    accum_t* dst_data = qk_sum_data + qSplitSize;
    scalar_t *qk_reduced_data =
        is_reduced_type
            ? buf_reduced_data + ompIdx * qSplitSize * ekvSplitSize
            : nullptr;
    scalar_t* query_t_padding_ptr = (!headSize_even && need_pack)
            ? query_padding_ptr + ompIdx * qSplitSize * eheadSize
            : nullptr;

    for ([[maybe_unused]] auto z : c10::irange(begin, end)) {
      auto i_kvi = is_broadcast_bs_kvi ? i/bs_shards_kvi : i;
      auto j_kvi = is_broadcast_head_kvi ? j/gqa_shards_kvi : j;
      auto kv_logical_num_data = kv_num_blocks_data + i_kvi * num_kviStrideB +
                              j_kvi * num_kviStrideH + k;
      int kv_indice_num = *kv_logical_num_data;
````
- **EN**: Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `at`.
- **CN**: 包含分支、循环或上下文管理等控制流。初始化或更新了 `at` 等值。

### Lines 421-448 / 第 421-448 行
````python
      std::vector<int> kv_indice_list(kv_indice_num);
      for(int kv_i = 0; kv_i < kv_indice_num; kv_i++){
        auto kv_logical_data = kv_indices_data + i_kvi * kviStrideB +
                                  j_kvi * kviStrideH + k*kviStrideQ + kv_i;
        kv_indice_list[kv_i] = *kv_logical_data;
      }
      bool is_skip_kv = kv_indice_num > 0 ? false : true;
{%- if has_full_kv_block %}
      auto full_kv_logical_num_data = full_kv_num_blocks_data + i_kvi * num_kviStrideB +
                              j_kvi * num_kviStrideH + k;
      int full_kv_indice_num = *full_kv_logical_num_data;
      std::vector<int> full_kv_indice_list(full_kv_indice_num);
      for(int kv_i = 0; kv_i < full_kv_indice_num; kv_i++){
        auto full_kv_logical_data = full_kv_indices_data + i_kvi * full_kviStrideB +
                                  j_kvi * full_kviStrideH + k*full_kviStrideQ + kv_i;
        full_kv_indice_list[kv_i] = *full_kv_logical_data;
      }
      is_skip_kv = kv_indice_num + full_kv_indice_num > 0 ? false : true;
{%- endif %}
      int64_t m = k * qSplitSize;
      int64_t cur_qSplitSize = std::min(qSplitSize, qSize - m);
      if (!is_skip_kv){
        // Initialize max and sum
        {{kernel.kernel_name}}_fill_stub(qk_max_data,
            -std::numeric_limits<accum_t>::infinity(), cur_qSplitSize);
        {{kernel.kernel_name}}_fill_stub(qk_sum_data,
            static_cast<accum_t>(0), cur_qSplitSize);

````
- **EN**: Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `std`, and `is_skip_kv`.
- **CN**: 包含分支、循环或上下文管理等控制流。初始化或更新了 `std`、`is_skip_kv` 等值。

### Lines 449-476 / 第 449-476 行
````python
        if (!headSize_even && need_pack) {
          // Pad query if headSize is not even
          {{kernel.kernel_name}}_copy_value_with_pad<scalar_t>(
            q_data + i * qStrideB + j * qStrideH + m * qStrideM,
            query_t_padding_ptr,
            cur_qSplitSize,
            headSize,
            cur_qSplitSize,
            eheadSize,
            qStrideM
          );
        }
      }

{%- if has_full_kv_block %}
      for (int64_t n_idx = 0; n_idx < kv_indice_num + full_kv_indice_num ; n_idx += 1) {
        auto n = n_idx < kv_indice_num ? kv_indice_list[n_idx]*kvSplitSize : full_kv_indice_list[n_idx - kv_indice_num]*kvSplitSize;
{%- else %}
      for (int64_t n_idx = 0; n_idx < kv_indice_num ; n_idx += 1) {
        auto n = kv_indice_list[n_idx]*kvSplitSize;
{%- endif %}

        auto cur_n = n/kvSplitSize;
        int64_t cur_kvSplitSize = std::min(kvSplitSize, kvSize - n);
        int64_t cur_ekvSplitSize = (need_pack && cur_kvSplitSize % 2 != 0) ? cur_kvSplitSize + 1 : cur_kvSplitSize;

        // Calculate scale * q @ k.T
        auto i_kv = is_broadcast_bs_kv ? i/bs_shards : i;
````
- **EN**: Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 包含分支、循环或上下文管理等控制流。

### Lines 477-504 / 第 477-504 行
````python
        auto j_kv = is_broadcast_head_kv ? j/gqa_shards : j;

        if (!need_pack) {
          auto k_addr =
              k_data + i_kv * kStrideB + j_kv * kStrideH + n * kStrideN;

          {{kernel.kernel_name}}_kernel_micro_gemm_transpose_b<static_cast<bool>(false)>(
              q_data + i * qStrideB + j * qStrideH +
                  m * qStrideM,
              k_addr,
              qk_data,
              cur_qSplitSize,
              cur_kvSplitSize,
              headSize,
              qStrideM,
              kStrideN,
              cur_kvSplitSize);

        } else {
          at::native::cpublas::brgemm(
              cur_qSplitSize,
              cur_kvSplitSize,
              eheadSize,
              headSize_even ? qStrideM : eheadSize,
              cur_kvSplitSize,
              cur_kvSplitSize,
              false,
              !headSize_even
````
- **EN**: Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `at`.
- **CN**: 包含分支、循环或上下文管理等控制流。初始化或更新了 `at` 等值。

### Lines 505-532 / 第 505-532 行
````python
                  ? query_t_padding_ptr
                  : q_data + i * qStrideB + j * qStrideH + m * qStrideM,
              key_reorder_ptr + i_kv * num_head_k * eheadSize * kvSize +
                  j_kv * eheadSize * kvSize + n * eheadSize,
              qk_data,
              need_pack);
        }

        {{kernel.kernel_name}}_mul_scale_kernel<accum_t>(qk_data, scaling_factor, cur_qSplitSize*cur_kvSplitSize);

{%- if score_mod and mask_mod %}
        // TODO: reduce the number of calls of q_idx and kv_idx initialization
        std::vector<int64_t> q_idx(cur_qSplitSize);
        for (int64_t i = 0; i < cur_qSplitSize; ++i) {
          q_idx[i] = m + i;
        }

        std::vector<int64_t> kv_idx(cur_kvSplitSize);
        for (int64_t i = 0; i < cur_kvSplitSize; ++i) {
          kv_idx[i] = n + i;
        }

        std::vector<int64_t> b_idx = {i};
        std::vector<int64_t> h_idx = {j};

        accum_t* in_ptr0 = qk_data;

        auto in_ptr1 = b_idx.data();
````
- **EN**: Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `std`.
- **CN**: 包含分支、循环或上下文管理等控制流。初始化或更新了 `std` 等值。

### Lines 533-560 / 第 533-560 行
````python
        auto in_ptr2 = h_idx.data();
        auto in_ptr3 = q_idx.data();
        auto in_ptr4 = kv_idx.data();

        // apply score mod function
        {
            {{ template.generate_other_buffer("score_others", 0, "len_score_other", kernel.args) }}
            accum_t* out_ptr{{score_buf_idx}} = in_ptr0;
            {{ template.modification(score_mod, score_buf_name, score_buf_idx)|indent(12, false) }}
        }

        if ((std::find(kv_indice_list.begin(), kv_indice_list.end(), cur_n) != kv_indice_list.end()) ){
          // Apply block mask, fill unused with -inf
          {
              {{ template.generate_other_buffer("mask_others", -1, "len_mask_other", kernel.args) }}
              accum_t* out_ptr{{mask_buf_idx}} = in_ptr0;
              {{ template.modification(mask_mod, mask_buf_name, mask_buf_idx)|indent(12, false) }}
          }
        }

{%- endif %}
        // Update coefficients with Softmax
        accum_t tmp_max = 0, tmp_sum = 0, exp_tmp = 0;
        for (int64_t row = 0; row < cur_qSplitSize; ++row) {
          // apply scaling factor and max per row in fusion
          {{kernel.kernel_name}}_mul_reduce_max_fusion_kernel(
              qk_data + row * cur_kvSplitSize,
              static_cast<accum_t>(1),
````
- **EN**: Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 包含分支、循环或上下文管理等控制流。

### Lines 561-588 / 第 561-588 行
````python
              cur_kvSplitSize,
              qk_data + row * cur_kvSplitSize,
              tmp_max);
          tmp_max = qk_max_data[row] > tmp_max ? qk_max_data[row] : tmp_max;
          if (tmp_max == -std::numeric_limits<accum_t>::infinity()) {
            // to avoid `nan = exp2f(-inf - (-inf))`
            {{kernel.kernel_name}}_fill_stub(
              {{kernel.kernel_name}}_conditional_data_ptr(qk_data, qk_reduced_data) + row * cur_ekvSplitSize,
              static_cast<scalar_t>(0), cur_kvSplitSize);
          } else {
            tmp_sum = tmp_max;
            // qk <- exp(qk - max) and sum per row
            {{kernel.kernel_name}}_exp_reduce_sum_fusion_kernel(
              qk_data + row * cur_kvSplitSize, cur_kvSplitSize,
              {{kernel.kernel_name}}_conditional_data_ptr(qk_data, qk_reduced_data) + row * cur_ekvSplitSize,
              tmp_sum);
            // exp_tmp <- exp(max[row] - max)
            exp_tmp = std::exp(qk_max_data[row] - tmp_max);
            // sum[row] <- sum + exp_tmp * sum[row]
            qk_sum_data[row] = tmp_sum + exp_tmp * qk_sum_data[row];
            // max[row] <- max
            qk_max_data[row] = tmp_max;
            // dst <- dst * exp_tmp
            if (n_idx > 0) {
              at::vec::map<accum_t>(
              [exp_tmp](Vec x) { return x * Vec(exp_tmp); },
              dst_data + row * headSize_v,
              dst_data + row * headSize_v,
````
- **EN**: Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `tmp_max`, `tmp_sum`, `exp_tmp`, and `at`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `tmp_max`、`tmp_sum`、`exp_tmp`、`at` 等值。

### Lines 589-616 / 第 589-616 行
````python
              headSize_v);
            }
          }
          if (need_pack && cur_kvSplitSize % 2 != 0) {
            // Pad: [qSplitSize, cur_kvSplitSize] -> [qSplitSize, cur_kvSplitSize + 1]
            *(qk_reduced_data + row * (1 + cur_kvSplitSize) + cur_kvSplitSize) = scalar_t(0);
          }
        }
        // Calculate Softmax(q @ k.T) @ v
        if (!need_pack) {
          auto v_addr =
              v_data + i_kv * vStrideB + j_kv * vStrideH + n * vStrideN;
          // Fallback Half brgemm is slower than micro gemm
          if (!std::is_same_v<scalar_t, at::Half>) {
            at::native::cpublas::brgemm(
                  cur_qSplitSize,
                  headSize_v,
                  cur_ekvSplitSize,
                  cur_ekvSplitSize,
                  vStrideN,
                  headSize_v,
                  n_idx > 0,
                  {{kernel.kernel_name}}_conditional_data_ptr(qk_data, qk_reduced_data),
                  v_addr,
                  dst_data,
                  need_pack);
          } else {
            if (n_idx > 0) {
````
- **EN**: Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `at`.
- **CN**: 包含分支、循环或上下文管理等控制流。初始化或更新了 `at` 等值。

### Lines 617-644 / 第 617-644 行
````python
              {{kernel.kernel_name}}_kernel_micro_gemm<static_cast<bool>(true)>(
                {{kernel.kernel_name}}_conditional_data_ptr(qk_data, qk_reduced_data),
                v_addr,
                dst_data,
                cur_qSplitSize,
                headSize_v,
                cur_ekvSplitSize,
                cur_ekvSplitSize,
                vStrideN,
                headSize_v);
            } else {
              {{kernel.kernel_name}}_kernel_micro_gemm<static_cast<bool>(false)>(
                {{kernel.kernel_name}}_conditional_data_ptr(qk_data, qk_reduced_data),
                v_addr,
                dst_data,
                cur_qSplitSize,
                headSize_v,
                cur_ekvSplitSize,
                cur_ekvSplitSize,
                vStrideN,
                headSize_v);
            }
          }
        } else {
          int64_t psize = n / kvSplitSize * ekvSplitSize;
          at::native::cpublas::brgemm(
              cur_qSplitSize,
              headSize_v,
````
- **EN**: Initializes or updates values such as `at`.
- **CN**: 初始化或更新了 `at` 等值。

### Lines 645-672 / 第 645-672 行
````python
              cur_ekvSplitSize,
              cur_ekvSplitSize,
              headSize_v,
              headSize_v,
              n_idx > 0,
              qk_reduced_data,
              value_reorder_ptr +
                  i_kv * num_head_k * kv_padding_size * headSize_v +
                  j_kv * kv_padding_size * headSize_v + psize * headSize_v,
              dst_data,
              need_pack);
        }
      }

      // dst <- dst / sum[row]
      // reorder MHA output with strides
      for (int64_t row = 0; row < cur_qSplitSize; ++row) {
        // Row sums for full masked out rows are 0, we set them to 1
        // in order to avoid NaNs in the output and instead set fully
        // masked out rows to 0
        qk_max_data[row] = qk_max_data[row] == -std::numeric_limits<accum_t>::infinity() ? 0 : qk_max_data[row];
        qk_sum_data[row] = qk_sum_data[row] == 0 ? 1 : qk_sum_data[row];
        accum_t sum_reciprocal = 1 / qk_sum_data[row];
        at::vec::map<scalar_t>(
            [sum_reciprocal, is_skip_kv](Vec x) { return  is_skip_kv ? Vec(0.0) : x * Vec(sum_reciprocal); },
            out_data + i * oStrideB + j * oStrideH + m * oStrideM + row * oStrideM,
            dst_data + row * headSize_v,
            headSize_v);
````
- **EN**: Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `at`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `at` 等值。

### Lines 673-700 / 第 673-700 行
````python
      }

      // Move to the next query
      at::native::data_index_step(i, batchSize, j, num_head, k, qSlice);
    }

    at::native::cpublas::brgemm_release(need_pack);

  });
}
"""


class CppFlexAttentionTemplate(CppTemplate):
    def __init__(
        self,
        input_nodes,
        layout: ir.Layout,
        scale,
        score_mod,
        mask_mod,
        kv_block_size,
        q_block_size,
        has_other_buffer,
        no_full_kv_block,
        fake_buffers,
        len_score_other,
        len_mask_other,
````
- **EN**: Introduces class `CppFlexAttentionTemplate`, function `__init__`. Initializes or updates values such as `at`, and `layout`.
- **CN**: 这里定义了类`CppFlexAttentionTemplate`、函数`__init__`。初始化或更新了 `at`、`layout` 等值。

### Lines 701-728 / 第 701-728 行
````python
        kernel_input_name_to_buffer,
        block_vars,
    ) -> None:
        assert layout.dtype in [torch.float, torch.bfloat16, torch.float16]
        super().__init__("flex_attention", input_nodes, layout, parallel_num_threads())
        self.scale = scale
        self.score_mod = score_mod
        self.mask_mod = mask_mod
        self.score_buf_name = (
            V.graph.register_buffer(self.score_mod) if self.score_mod else None
        )
        self.mask_buf_name = (
            V.graph.register_buffer(self.mask_mod) if self.mask_mod else None
        )

        def get_idx(buf_name):
            match = re.search(r"\d+", buf_name)
            assert match, f"incorrect score buf name: {buf_name}"
            return match.group()

        self.score_buf_idx = (
            get_idx(self.score_buf_name) if self.score_buf_name else None
        )
        self.mask_buf_idx = get_idx(self.mask_buf_name) if self.mask_buf_name else None
        self.kv_block_size = kv_block_size
        self.q_block_size = q_block_size
        self.has_other_buffer = has_other_buffer
        self.no_full_kv_block = no_full_kv_block
````
- **EN**: Introduces function `get_idx`. Tracks dtype-specific behavior or constants that affect numeric execution. Builds or updates a registry/mapping so later code can dispatch by name or capability. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了函数`get_idx`。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 729-756 / 第 729-756 行
````python
        self.other_buffer_input_offset = 2
        if self.no_full_kv_block:
            self.other_buffer_input_offset = 0
        self.fake_buffers = fake_buffers
        self.len_score_other = len_score_other
        self.len_mask_other = len_mask_other
        self.kernel_input_name_to_buffer = kernel_input_name_to_buffer
        self.block_vars = block_vars
        self.extra_sizevars = list(
            OrderedSet(
                val
                for val in self.kernel_input_name_to_buffer.values()
                if isinstance(val, sympy.Symbol)
            )
        )
        self.other_buf_start_idx = 5
        self.score_mod_other_buffers = (
            self.input_nodes[
                self.other_buf_start_idx
                + self.other_buffer_input_offset : self.other_buf_start_idx
                + self.other_buffer_input_offset
                + self.len_score_other
            ]
            if self.has_other_buffer
            else None
        )
        self.mask_mod_other_buffers = (
            self.input_nodes[
````
- **EN**: Uses symbolic expressions to reason about indices, shapes, or algebraic properties. Contains control-flow that branches, iterates, or scopes resource management. This range continues the implementation of function `CppFlexAttentionTemplate.__init__`.
- **CN**: 借助符号表达式来推理索引、形状或代数性质。包含分支、循环或上下文管理等控制流。这一段延续了函数`CppFlexAttentionTemplate.__init__` 的具体实现。

### Lines 757-784 / 第 757-784 行
````python
                self.other_buf_start_idx
                + self.other_buffer_input_offset
                + self.len_score_other :
            ]
            if self.has_other_buffer
            else None
        )
        self.other_ptr_data = {}  # type: ignore[var-annotated]

    def update_kernel_args(self, kernel_args):
        kernel_args.update(
            {
                key: value
                for key, value in self.kernel_input_name_to_buffer.items()
                if not isinstance(value, sympy.Symbol)
            }
        )
        return kernel_args

    def generate_other_buffer(self, buf_list, start_offset, len_attr, kernel_args):
        kernel_input_name_to_buffer_name = {
            key: value if isinstance(value, sympy.Symbol) else value.get_name()
            for key, value in self.kernel_input_name_to_buffer.items()
        }

        def get_arg(name):
            return kernel_input_name_to_buffer_name.get(name)

````
- **EN**: Introduces function `update_kernel_args`, function `generate_other_buffer`, function `get_arg`. Uses symbolic expressions to reason about indices, shapes, or algebraic properties. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`update_kernel_args`、函数`generate_other_buffer`、函数`get_arg`。借助符号表达式来推理索引、形状或代数性质。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 785-812 / 第 785-812 行
````python
        def get_arg_name(name):
            if isinstance(get_arg(name), sympy.Symbol):
                return kernel_args.sizevars.get(get_arg(name))
            return kernel_args.input_buffers.get(get_arg(name))

        if not self.has_other_buffer:
            return ""

        if start_offset == -1:
            start_offset = self.len_score_other

        length = getattr(self, len_attr)
        for i in range(length):
            pointer = f"in_ptr{self.other_buf_start_idx + start_offset + i}"
            buffer_key = f"{buf_list}_{i}"
            if pointer not in self.other_ptr_data:
                self.other_ptr_data[pointer] = (
                    get_arg_name(buffer_key),
                    get_arg(buffer_key),
                )

        return "\n".join(
            f"auto {ptr} = {name};" for ptr, (name, _) in self.other_ptr_data.items()
        )

    def modification(self, subgraph_buffer, output_name, output_idx):
        assert isinstance(subgraph_buffer, ir.ComputedBuffer)
        subgraph_buffer_data = subgraph_buffer.data
````
- **EN**: Introduces function `get_arg_name`, function `modification`. Uses symbolic expressions to reason about indices, shapes, or algebraic properties. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`get_arg_name`、函数`modification`。借助符号表达式来推理索引、形状或代数性质。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 813-840 / 第 813-840 行
````python
        from ..loop_body import LoopBody
        from ..utils import sympy_index_symbol_with_prefix, SymT
        from ..virtualized import V
        from .cpp import CppKernelProxy, KernelGroup, ParallelDepth

        kernel_group = KernelGroup()
        kernel_input_args = {
            "score": "in_ptr0",
            "b": "in_ptr1",
            "h": "in_ptr2",
            "q_idx": "in_ptr3",
            "kv_idx": "in_ptr4",
        }
        if self.has_other_buffer:
            kernel_input_args.update(
                {arg: ptr for ptr, (_, arg) in self.other_ptr_data.items()}
            )

        kernel_output_args = {output_name: f"out_ptr{output_idx}"}

        args = kernel_group.args
        for name, inp in kernel_input_args.items():
            args.input_buffers[name] = inp

        for name, inp in kernel_output_args.items():
            args.output_buffers[name] = inp

        for name in self.extra_sizevars:
````
- **EN**: Imports dependencies such as `..loop_body`, `..utils`, `..virtualized`, and `.cpp` for the logic in this range. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `kernel_group`, `kernel_input_args`, `kernel_output_args`, and `args`. This range continues the implementation of function `CppFlexAttentionTemplate.modification`.
- **CN**: 这里导入了 `..loop_body`、`..utils`、`..virtualized`、`.cpp` 等依赖，为后续逻辑提供基础能力。包含分支、循环或上下文管理等控制流。初始化或更新了 `kernel_group`、`kernel_input_args`、`kernel_output_args`、`args` 等值。这一段延续了函数`CppFlexAttentionTemplate.modification` 的具体实现。

### Lines 841-868 / 第 841-868 行
````python
            args.sizevars[name] = f"k{name}"

        kernel_group.args = args

        cpp_kernel_proxy = CppKernelProxy(kernel_group)
        bodies = []
        var_sizes_list = []
        var_sizes = tuple(subgraph_buffer.get_size())
        var_ranges = {
            sympy_index_symbol_with_prefix(SymT.INDEX, i): sz
            for i, sz in enumerate(var_sizes)
        }

        dst_layout = subgraph_buffer.get_layout()
        output_index = dst_layout.make_indexer()([*var_ranges.keys()])

        def fn(*args):
            V.ops.store(
                output_name,
                output_index,
                subgraph_buffer_data.make_loader()(args).value,
            )

        body = LoopBody(
            fn,
            (list(var_ranges.keys())),
            var_ranges,
            list(var_ranges.keys()),
````
- **EN**: Introduces function `fn`. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `cpp_kernel_proxy`, `bodies`, `var_sizes_list`, `var_sizes`, `var_ranges`, `dst_layout`, and `...+2`.
- **CN**: 这里定义了函数`fn`。包含分支、循环或上下文管理等控制流。初始化或更新了 `cpp_kernel_proxy`、`bodies`、`var_sizes_list`、`var_sizes`、`var_ranges`、`dst_layout`、`另有2项` 等值。

### Lines 869-896 / 第 869-896 行
````python
            tuple(),
        )

        from ..loop_body import MemoryUsageType

        assert all(
            mem.buffer_name in kernel_group.args.input_buffers
            for mem in body.memory_usage[MemoryUsageType.LOAD]
        ), (
            "All the buffers in the score and mask subgraph should be in kernel_group.args.input_buffers"
        )

        bodies.append(body)
        var_sizes_list.append((var_sizes, ()))

        cpp_kernel_proxy.codegen_loop_bodies(bodies, var_sizes_list)

        def max_parallel_depth():
            return ParallelDepth(parallel_depth=0, start_depth=0)

        # This loop is not parallelized since it is not the outermost loop.
        with patch.object(
            cpp_kernel_proxy.loop_nest, "max_parallel_depth", max_parallel_depth
        ):
            kernel_group.finalize_kernel(cpp_kernel_proxy, [])
        output_code = kernel_group.loops_code.getvalue()

        var_q_symbol, var_kv_symbol = self.block_vars
````
- **EN**: Imports dependencies such as `..loop_body` for the logic in this range. Introduces function `max_parallel_depth`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里导入了 `..loop_body` 等依赖，为后续逻辑提供基础能力。这里定义了函数`max_parallel_depth`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 897-924 / 第 897-924 行
````python
        # See [Note] Handle the case where the split sizes are not statically known.
        # We don't know the value of qBlockSize and rkvBlockSize during compilation time
        # thus we've represented them by symbols.
        # We change the symbol strings back to "cur_qSplitSize" and "cur_kvSplitSize"
        # in the generated code thus they'll be filled with the real value during runtime.
        if var_q_symbol in kernel_group.args.sizevars:
            output_code = output_code.replace(
                kernel_group.args.sizevars[var_q_symbol], "cur_qSplitSize"
            )
        if var_kv_symbol in kernel_group.args.sizevars:
            output_code = output_code.replace(
                kernel_group.args.sizevars[var_kv_symbol], "cur_kvSplitSize"
            )

        return output_code

    @staticmethod
    def add_choices(
        choices,
        input_nodes,
        layout,
        scale,
        score_mod,
        mask_mod,
        kv_block_size,
        q_block_size,
        has_other_buffer,
        no_full_kv_block,
````
- **EN**: Introduces function `add_choices`. Applies decorators to register behavior or alter how the following definition is constructed. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`add_choices`。使用装饰器来注册行为，或改变后续定义的构造方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 925-952 / 第 925-952 行
````python
        fake_buffers,
        len_score_other,
        len_mask_other,
        kernel_input_name_to_buffer,
        block_vars,
    ):
        def preprocessor(input_nodes, layout):
            return input_nodes, layout

        def postprocessor(output):
            return output

        template = DataProcessorTemplateWrapper(
            CppFlexAttentionTemplate,
            preprocessor,
            postprocessor,
            input_nodes=input_nodes,
            layout=layout,
            scale=scale,
            score_mod=score_mod,
            mask_mod=mask_mod,
            kv_block_size=kv_block_size,
            q_block_size=q_block_size,
            has_other_buffer=has_other_buffer,
            no_full_kv_block=no_full_kv_block,
            fake_buffers=fake_buffers,
            len_score_other=len_score_other,
            len_mask_other=len_mask_other,
````
- **EN**: Introduces function `preprocessor`, function `postprocessor`. Includes returns or checks that define the contract of this code path. Initializes or updates values such as `template`, `input_nodes`, `layout`, `scale`, `score_mod`, `mask_mod`, and `...+7`.
- **CN**: 这里定义了函数`preprocessor`、函数`postprocessor`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。初始化或更新了 `template`、`input_nodes`、`layout`、`scale`、`score_mod`、`mask_mod`、`另有7项` 等值。

### Lines 953-980 / 第 953-980 行
````python
            kernel_input_name_to_buffer=kernel_input_name_to_buffer,
            block_vars=block_vars,
        )
        template.maybe_append_choice(choices)
        return template

    def apply_score_mod(self, score, b, h, q_idx, kv_idx):
        return self.score_mod.graph_module(score, b, h, q_idx, kv_idx).item()

    def render(  # type: ignore[override,return]
        self,
        kernel,
        template_buffer_node: ir.CppTemplateBuffer | None = None,
        epilogue_nodes: list[ir.IRNode] | None = None,
        **kwargs,
    ) -> str:
        if epilogue_nodes is not None and epilogue_nodes != []:
            raise NotImplementedError(
                "Unsupported for `epilogue_nodes` in CppFlexAttentionTemplate."
            )
        # Query (Batch x Num_heads  x Q_seq_len  x Dim_per_head)
        #     -> (Batch x Q_seq_len  x Num_heads  x Dim_per_head)
        #  Key   (Batch x Num_heads  x KV_seq_len x Dim_per_head)
        #     -> (Batch x KV_seq_len x Num_heads  x Dim_per_head)
        #  Value (Batch x Num_heads  x KV_seq_len x Dim_per_head)
        #     -> (Batch x KV_seq_len x Num_heads  x Dim_per_head)

        query = kernel.permute(self.input_nodes[0], [0, 2, 1, 3])
````
- **EN**: Introduces function `apply_score_mod`, function `render`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `kernel_input_name_to_buffer`, `block_vars`, `template_buffer_node`, `epilogue_nodes`, and `query`.
- **CN**: 这里定义了函数`apply_score_mod`、函数`render`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `kernel_input_name_to_buffer`、`block_vars`、`template_buffer_node`、`epilogue_nodes`、`query` 等值。

### Lines 981-1008 / 第 981-1008 行
````python
        key = kernel.permute(self.input_nodes[1], [0, 2, 1, 3])
        value = kernel.permute(self.input_nodes[2], [0, 2, 1, 3])
        self.accumulate_dtype = torch.float
        self.input_dtype = query.layout.dtype

        num_threads = parallel_num_threads()
        assert isinstance(self.output_node, ir.IRNode)
        buf_out = TensorBox.create(self.output_node)
        if template_buffer_node is not None:
            buf_out = template_buffer_node
        options = dict(
            query=query,
            key=key,
            value=value,
            kv_num_blocks=self.input_nodes[3],
            kv_indices=self.input_nodes[4],
            full_kv_num_blocks=(
                self.input_nodes[5] if not self.no_full_kv_block else None
            ),
            full_kv_indices=self.input_nodes[6] if not self.no_full_kv_block else None,
            score_mod_other_buffers=self.score_mod_other_buffers,
            mask_mod_other_buffers=self.mask_mod_other_buffers,
            scale=self.scale,
            has_full_kv_block=not self.no_full_kv_block,
            accumulate_dtype=self.accumulate_dtype,
            query_dtype=self.input_dtype,
            kvBlockSize=self.kv_block_size,
            qBlockSize=self.q_block_size,
````
- **EN**: Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `key`, `value`, `num_threads`, `buf_out`, `options`, `query`, and `...+12`.
- **CN**: 追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `key`、`value`、`num_threads`、`buf_out`、`options`、`query`、`另有12项` 等值。

### Lines 1009-1036 / 第 1009-1036 行
````python
            template=self,
            output=buf_out,
            kernel=kernel,
            num_thread=num_threads,
            score_mod=self.score_mod,
            mask_mod=self.mask_mod,
            score_buf_name=self.score_buf_name,
            mask_buf_name=self.mask_buf_name,
            score_buf_idx=self.score_buf_idx,
            mask_buf_idx=self.mask_buf_idx,
        )
        with contextlib.ExitStack() as stack:
            for buf in self.fake_buffers:
                stack.enter_context(
                    patch.object(V.graph, "get_dtype", self._fake_get_dtype(buf))
                )
            return self._template_from_string(FLEX_ATTENTION_TEMPLATE).render(**options)

    def codegen_softmax_fusion(self, kernel_name: str):
        # TODO: use inductor IR to rewrite those fusions
        return self._template_from_string(SOFTMAX_FUSIONS).render(
            dict(kernel_name=kernel_name)
        )

    def codegen_brgemm_pack_function(self, kernel_name: str):
        # TODO: make them general for common bmm templates
        return self._template_from_string(BRGEMM_PACK_FUNCTIONS).render(
            dict(kernel_name=kernel_name)
````
- **EN**: Introduces function `codegen_softmax_fusion`, function `codegen_brgemm_pack_function`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `template`, `output`, `kernel`, `num_thread`, `score_mod`, `mask_mod`, and `...+4`.
- **CN**: 这里定义了函数`codegen_softmax_fusion`、函数`codegen_brgemm_pack_function`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `template`、`output`、`kernel`、`num_thread`、`score_mod`、`mask_mod`、`另有4项` 等值。

### Lines 1037-1064 / 第 1037-1064 行
````python
        )

    def codegen_allocate_buffer(self, buffer_name: str, buffer_dtype, buffer_size):
        return self._template_from_string(ALLOCATE_BUFFER).render(
            dict(
                buffer_name=buffer_name,
                buffer_dtype=buffer_dtype,
                buffer_size=buffer_size,
            )
        )

    def micro_gemm_define(self, kernel_name: str):
        from torch._inductor.codegen.cpp_gemm_template import (
            CppTemplateKernel,
            parallel_num_threads,
        )
        from torch._inductor.codegen.cpp_micro_gemm import CppMicroGemmFP32Vec
        from torch._inductor.virtualized import V

        micro_gemm_trans = CppMicroGemmFP32Vec(
            kernel_name + "_kernel_micro_gemm_transpose_b",
            self.input_dtype,
            self.input_dtype,
            self.accumulate_dtype,
            self.accumulate_dtype,
            GemmBlocking(1, 16, 1),
            1,
            True,
````
- **EN**: Imports dependencies such as `torch._inductor.codegen.cpp_gemm_template`, `torch._inductor.codegen.cpp_micro_gemm`, and `torch._inductor.virtualized` for the logic in this range. Introduces function `codegen_allocate_buffer`, function `micro_gemm_define`. Includes returns or checks that define the contract of this code path. Initializes or updates values such as `buffer_name`, `buffer_dtype`, `buffer_size`, and `micro_gemm_trans`.
- **CN**: 这里导入了 `torch._inductor.codegen.cpp_gemm_template`、`torch._inductor.codegen.cpp_micro_gemm`、`torch._inductor.virtualized` 等依赖，为后续逻辑提供基础能力。这里定义了函数`codegen_allocate_buffer`、函数`micro_gemm_define`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。初始化或更新了 `buffer_name`、`buffer_dtype`、`buffer_size`、`micro_gemm_trans` 等值。

### Lines 1065-1089 / 第 1065-1089 行
````python
            True,
        )

        micro_gemm = CppMicroGemmFP32Vec(
            kernel_name + "_kernel_micro_gemm",
            self.input_dtype,
            self.input_dtype,
            self.accumulate_dtype,
            self.accumulate_dtype,
            GemmBlocking(1, 16, 1),
            1,
            True,
            False,
        )

        with V.set_graph_handler(V.graph):
            kernel = CppTemplateKernel("cpp_micro_gemm", parallel_num_threads())
            code_trans = micro_gemm_trans.codegen_define(kernel)
            code = micro_gemm.codegen_define(kernel)
        return code + code_trans

    def codegen_micro_gemm(self, kernel_name: str):
        micro_gemm = self.micro_gemm_define(kernel_name)
        GEMM_SOURCE_CODE = MICRO_GEMM_TEMPLATE.replace("GEMM_DEFINE", micro_gemm)
        return self._template_from_string(GEMM_SOURCE_CODE).render()
````
- **EN**: Introduces function `codegen_micro_gemm`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `micro_gemm`, `kernel`, `code_trans`, `code`, and `GEMM_SOURCE_CODE`.
- **CN**: 这里定义了函数`codegen_micro_gemm`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `micro_gemm`、`kernel`、`code_trans`、`code`、`GEMM_SOURCE_CODE` 等值。

## Key Concepts / 关键概念
- **EN**: Generates backend-specific code for lowered kernels  
  **CN**: 为下沉后的内核生成后端相关代码
- **EN**: Symbolic shape or index reasoning  
  **CN**: 符号化形状或索引推理
- **EN**: Graph-level transformations  
  **CN**: 图级变换
- **EN**: Operator lowering  
  **CN**: 算子下沉
- **EN**: Profiling data handling  
  **CN**: 性能画像数据处理
- **EN**: Kernel construction or dispatch  
  **CN**: 内核构建或派发
- **EN**: Dtype propagation or validation  
  **CN**: 数据类型传播或校验
- **EN**: Primary classes: `CppFlexAttentionTemplate`  
  **CN**: 主要类：`CppFlexAttentionTemplate`

## Dependencies / 依赖关系
- **Standard library / 标准库**: `contextlib`, `logging`, `re`, `unittest.mock`
- **Third-party / 第三方**: `sympy`
- **PyTorch/Internal / PyTorch 内部**: `torch`, `torch.utils`, `...utils._ordered_set`, `..`, `..ir`, `..select_algorithm`, `..utils`, `..virtualized`, `.cpp_template`, `.cpp_utils`, `..loop_body`, `.cpp`, `torch._inductor.codegen.cpp_gemm_template`, `torch._inductor.codegen.cpp_micro_gemm`, `torch._inductor.virtualized`
