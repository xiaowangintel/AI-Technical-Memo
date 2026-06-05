# cpp_gemm_template.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `torch/_inductor/codegen/cpp_gemm_template.py`
- **Repository / 仓库**: `pytorch/pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: This module generates backend-specific code for lowered kernels. It defines classes such as `CppGemmTemplate`, `CppWoqInt4GemmTemplateMeta`, and `CppWoqInt4GemmTemplate`. It exposes functions such as `_is_int8_gemm`, `get_padded_n`, `transpose_w`, `expand_bias`, `prune_tensors`, and `gen_2d_view_of_epilogue_buf`.
- **用途（中文）**: 该模块为下沉后的内核生成后端相关代码。其中定义了 `CppGemmTemplate`、`CppWoqInt4GemmTemplateMeta`、`CppWoqInt4GemmTemplate` 等类。同时提供 `_is_int8_gemm`、`get_padded_n`、`transpose_w`、`expand_bias`、`prune_tensors`、`gen_2d_view_of_epilogue_buf` 等函数。

## Line-by-Line Analysis / 逐行分析
### Lines 1-28 / 第 1-28 行
````python
# mypy: allow-untyped-defs
import contextlib
import logging
import math
import os
from collections.abc import Callable
from functools import lru_cache
from typing import Any, cast, TypeVar
from unittest.mock import patch

import torch
import torch.utils
from torch.utils._ordered_set import OrderedSet

from ..._dynamo.utils import counters
from .. import config, ir, lowering as L
from ..kernel.mm_common import mm_args
from ..select_algorithm import DataProcessorTemplateWrapper
from ..utils import (
    has_free_symbols,
    is_same_mkldnn_tensor,
    is_same_tensor,
    parallel_num_threads,
)
from ..virtualized import ops, V
from .cpp import get_export_declaration
from .cpp_micro_gemm import (
    CppMicroBrgemm,
````
- **EN**: Imports dependencies such as `contextlib`, `logging`, `math`, `os`, `collections.abc`, `functools`, and `...+13` for the logic in this range. Manipulates cached state or cache paths to avoid rebuilding the same work repeatedly.
- **CN**: 这里导入了 `contextlib`、`logging`、`math`、`os`、`collections.abc`、`functools`、`另有13项` 等依赖，为后续逻辑提供基础能力。这一段处理缓存状态或缓存路径，以避免重复构建相同工作。

### Lines 29-56 / 第 29-56 行
````python
    CppMicroGemm,
    CppMicroGemmAMX,
    CppMicroGemmFP32Vec,
    create_micro_gemm,
    is_int8_woq_gemm_small_m_dim_corner_case,
    LayoutType,
)
from .cpp_template import CppTemplate
from .cpp_template_kernel import CppTemplateKernel
from .cpp_utils import (
    create_epilogue_with_attr,
    DTYPE_TO_CPP,
    GemmBlocking,
    get_gemm_template_output_and_compute_dtype,
)


log = logging.getLogger(__name__)

GEMM_TEMPLATE_INIT_BLOCKING_BASIC_BLOCK = r"""
    constexpr int64_t num_threads = {{num_threads}};
    constexpr int64_t N = {{N}};
    constexpr int64_t K = {{K}};
    constexpr int64_t Mr = {{micro_gemm.register_blocking.block_m}};
    constexpr int64_t Nr = {{micro_gemm.register_blocking.block_n}};
    constexpr int64_t Kr = {{micro_gemm.register_blocking.block_k}};
    constexpr int64_t Nr_blocks = (N + Nr - 1) / Nr;
    constexpr int64_t Kr_blocks = (K + Kr - 1) / Kr;
````
- **EN**: Imports dependencies such as `.cpp_template`, `.cpp_template_kernel`, and `.cpp_utils` for the logic in this range. Builds or updates a registry/mapping so later code can dispatch by name or capability. Initializes or updates values such as `log`, and `GEMM_TEMPLATE_INIT_BLOCKING_BASIC_BLOCK`.
- **CN**: 这里导入了 `.cpp_template`、`.cpp_template_kernel`、`.cpp_utils` 等依赖，为后续逻辑提供基础能力。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。初始化或更新了 `log`、`GEMM_TEMPLATE_INIT_BLOCKING_BASIC_BLOCK` 等值。

### Lines 57-84 / 第 57-84 行
````python
{%- if is_dynamic_M %}
    const int64_t M = {{kernel.size(GemmOut, 0)}};
    const int64_t Mr_blocks = (M + Mr - 1) / Mr;
{%- else %}
    constexpr int64_t M = {{kernel.size(GemmOut, 0)}};
    constexpr int64_t Mr_blocks = (M + Mr - 1) / Mr;
{%- endif %}
"""

GEMM_TEMPLATE_INIT_BLOCKING_EXTENDED = r"""
{%- if is_dynamic_M %}
    {%- if num_threads > 1 %}
    int64_t Mt_blocks, Nt_blocks, Kt_blocks;
    mm_get_thread_blocking(num_threads, {{config.cpp.gemm_max_k_slices}}, M, N, K, Mr, Nr, Kr, Mt_blocks, Nt_blocks, Kt_blocks);
    {%- else %}
    const auto Mt_blocks = Mr_blocks;
    const auto Nt_blocks = Nr_blocks;
    const auto Kt_blocks = Kr_blocks;
    {%- endif %}
    int64_t Mc_blocks, Nc_blocks, Kc_blocks;
    uint32_t L1_cache_size = {{L1_cache_size}};
    uint32_t L2_cache_size = {{L2_cache_size}};
    mm_get_cache_blocking<{{kernel.dtype(X)}}, {{kernel.dtype(W)}}>(
        num_threads,
        M,
        N,
        K,
        Mr,
````
- **EN**: Manipulates cached state or cache paths to avoid rebuilding the same work repeatedly. Tracks dtype-specific behavior or constants that affect numeric execution. Initializes or updates values such as `GEMM_TEMPLATE_INIT_BLOCKING_EXTENDED`.
- **CN**: 这一段处理缓存状态或缓存路径，以避免重复构建相同工作。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。初始化或更新了 `GEMM_TEMPLATE_INIT_BLOCKING_EXTENDED` 等值。

### Lines 85-112 / 第 85-112 行
````python
        Nr,
        Kr,
        Mt_blocks,
        Nt_blocks,
        Kt_blocks,
        Mc_blocks,
        Nc_blocks,
        Kc_blocks,
        L1_cache_size,
        L2_cache_size
    );
    const int64_t num_Mc_blocks = (Mr_blocks + Mc_blocks - 1) / Mc_blocks;
    const int64_t num_Nc_blocks = (Nr_blocks + Nc_blocks - 1) / Nc_blocks;
    const int64_t num_Mt_blocks = (Mr_blocks + Mt_blocks - 1) / Mt_blocks;
    const int64_t num_Nt_blocks = (Nr_blocks + Nt_blocks - 1) / Nt_blocks;
    const int64_t num_Kt_blocks = (Kr_blocks + Kt_blocks - 1) / Kt_blocks;
{%- else %}
    constexpr int64_t Mt_blocks = {{template.thread_blocking(num_threads).block_m}};
    constexpr int64_t Nt_blocks = {{template.thread_blocking(num_threads).block_n}};
    constexpr int64_t Kt_blocks = {{template.thread_blocking(num_threads).block_k}};
    constexpr int64_t Mc_blocks = {{template.cache_blocking(num_threads).block_m}};
    constexpr int64_t Nc_blocks = {{template.cache_blocking(num_threads).block_n}};
    constexpr int64_t Kc_blocks = {{template.cache_blocking(num_threads).block_k}};
    constexpr int64_t num_Mc_blocks = (Mr_blocks + Mc_blocks - 1) / Mc_blocks;
    constexpr int64_t num_Nc_blocks = (Nr_blocks + Nc_blocks - 1) / Nc_blocks;
    constexpr int64_t num_Mt_blocks = (Mr_blocks + Mt_blocks - 1) / Mt_blocks;
    constexpr int64_t num_Nt_blocks = (Nr_blocks + Nt_blocks - 1) / Nt_blocks;
    constexpr int64_t num_Kt_blocks = (Kr_blocks + Kt_blocks - 1) / Kt_blocks;
````
- **EN**: Manipulates cached state or cache paths to avoid rebuilding the same work repeatedly.
- **CN**: 这一段处理缓存状态或缓存路径，以避免重复构建相同工作。

### Lines 113-140 / 第 113-140 行
````python
{%- endif %}
{%- if is_woq_int4 %}
    int64_t group_size = *q_group_size;
{%- endif %}

    // make sure all partitions are assigned
    {{kernel.assert_function}}(
        Mt_blocks * Nt_blocks * Kt_blocks * {{num_threads}} >= Mr_blocks * Nr_blocks * Kr_blocks,
        "Not all partitions are assigned."
    );
"""

GEMM_TEMPLATE_MULTI_THREADS_PARAMS = r"""
const int tid = omp_get_thread_num();
const int64_t k_group_id = tid / num_Kt_blocks;
const int64_t k_slice_id = tid % num_Kt_blocks;
const int64_t n_group_id = k_group_id / num_Nt_blocks;
const int64_t n_slice_id = k_group_id % num_Nt_blocks;
const int64_t k_block_start = k_slice_id * Kt_blocks;
const int64_t k_block_end = std::min(k_block_start + Kt_blocks, Kr_blocks);
const int64_t n_block_start = n_slice_id * Nt_blocks;
const int64_t n_block_end = std::min(n_block_start + Nt_blocks, Nr_blocks);
const int64_t m_block_start = std::min(n_group_id * Mt_blocks, Mr_blocks);
const int64_t m_block_end = std::min(m_block_start + Mt_blocks, Mr_blocks);
const int64_t num_Mc_blocks_per_thread = (m_block_end - m_block_start + Mc_blocks - 1) / Mc_blocks;
"""

GEMM_TEMPLATE_SINGLE_THREAD_PARAMS = r"""
````
- **EN**: Initializes or updates values such as `GEMM_TEMPLATE_MULTI_THREADS_PARAMS`, and `GEMM_TEMPLATE_SINGLE_THREAD_PARAMS`.
- **CN**: 初始化或更新了 `GEMM_TEMPLATE_MULTI_THREADS_PARAMS`、`GEMM_TEMPLATE_SINGLE_THREAD_PARAMS` 等值。

### Lines 141-168 / 第 141-168 行
````python
constexpr int tid = 0;
constexpr int64_t k_group_id = 0;
constexpr int64_t k_slice_id = 0;
constexpr int64_t n_group_id = 0;
constexpr int64_t n_slice_id = 0;
constexpr int64_t m_block_start = 0;
constexpr int64_t n_block_start = 0;
constexpr int64_t n_block_end = Nr_blocks;
constexpr int64_t k_block_start = 0;
constexpr int64_t k_block_end = Kr_blocks;
{%- if is_dynamic_M %}
const int64_t num_Mc_blocks_per_thread = num_Mc_blocks;
const int64_t m_block_end = Mr_blocks;
{%- else %}
constexpr int64_t num_Mc_blocks_per_thread = num_Mc_blocks;
constexpr int64_t m_block_end = Mr_blocks;
{%- endif %}
"""

GEMM_TEMPLATE_M_LOOP_PARAMS = r"""
const int64_t my_mc_block_id = (mc_block_id + n_slice_id) % num_Mc_blocks_per_thread;
const int64_t mc = m_block_start + my_mc_block_id * Mc_blocks;
const int64_t m_start = mc * Mr;
const int64_t m_end = std::min(std::min(mc + Mc_blocks, m_block_end) * Mr, M);
const int64_t m_size = m_end - m_start;
"""

GEMM_TEMPLATE_N_LOOP_PARAMS = r"""
````
- **EN**: Initializes or updates values such as `GEMM_TEMPLATE_M_LOOP_PARAMS`, and `GEMM_TEMPLATE_N_LOOP_PARAMS`.
- **CN**: 初始化或更新了 `GEMM_TEMPLATE_M_LOOP_PARAMS`、`GEMM_TEMPLATE_N_LOOP_PARAMS` 等值。

### Lines 169-196 / 第 169-196 行
````python
const int64_t n_start = nc * Nr;
const int64_t n_end = std::min(std::min(nc + Nc_blocks, n_block_end) * Nr, N);
const int64_t n_size = n_end - n_start;
// NB: assume we pad N, nc_block_end won't exceed padded N here.
const int64_t nc_block_end = std::min(nc + Nc_blocks, n_block_end);
"""

GEMM_TEMPLATE_MICROKERNEL_DEF = r"""
{{template.header().getvalue()}}

{{micro_gemm.codegen_define(kernel)}}
"""

GEMM_TEMPLATE_STUB_DEF = r"""
{%- if x_scale is not none %}
    {%- set kernel_args = {"X": X, "W": W, "inp": inp, "x_scale": x_scale, "x_zp": x_zp, "w_scale": w_scale, "w_zp": w_zp,} %}
{%- elif is_woq_int4 %}
    {%- set kernel_args = {"X": X, "W": W, "q_group_size": q_group_size, "qscale_and_zeros": qscale_and_zeros} %}
{%- else %}
    {%- set kernel_args = {"X": X, "W": W, "inp": inp} %}
{%- endif %}

extern "C" {{export_declaration}}
{{kernel.def_kernel(inputs=kernel_args, outputs={"Y": Y}, aliases=aliases)}}
"""

GEMM_TEMPLATE = r"""
{{ template.codegen_gemm_stub_def() }}
````
- **EN**: Initializes or updates values such as `GEMM_TEMPLATE_MICROKERNEL_DEF`, `GEMM_TEMPLATE_STUB_DEF`, and `GEMM_TEMPLATE`.
- **CN**: 初始化或更新了 `GEMM_TEMPLATE_MICROKERNEL_DEF`、`GEMM_TEMPLATE_STUB_DEF`、`GEMM_TEMPLATE` 等值。

### Lines 197-224 / 第 197-224 行
````python
{
    {{ kernel.maybe_codegen_profile() }}
    {{ template.codegen_blocks(
        num_threads, N, K, micro_gemm, is_dynamic_M, kernel, GemmOut, config, L1_cache_size, L2_cache_size, X, W
    ) }}

{%- if maybe_k_slicing %}
    std::unique_ptr<std::unique_ptr<{{DTYPE_TO_CPP[acc_buf_dtype]}}[]>[]> local_buf_ptrs;
    if (num_Kt_blocks > 1) {
        local_buf_ptrs.reset(new std::unique_ptr<{{DTYPE_TO_CPP[acc_buf_dtype]}}[]>[num_Mc_blocks * num_Nc_blocks * num_Kt_blocks]);
    }
{%- endif %}

{%- if num_threads > 1 %}
    {%- set use_dynamic_threads = ((config.cpp.threads < 1) and (num_threads == cpu_count)) or config.cpp.dynamic_threads %}
    {%- if use_dynamic_threads %}
    #pragma omp parallel
    {%- else %}
    #pragma omp parallel num_threads({{num_threads}})
    {%- endif %}
    {
        {{ template.codegen_multi_threads_params()|indent(8, false) }}
{%- else %}
    {
        {{ template.codegen_single_thread_params(is_dynamic_M)|indent(8, false) }}
{%- endif %}
        {{ micro_gemm.codegen_init(kernel) }}
{%- if use_local_acc %}
````
- **EN**: Manipulates cached state or cache paths to avoid rebuilding the same work repeatedly. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `std`.
- **CN**: 这一段处理缓存状态或缓存路径，以避免重复构建相同工作。包含分支、循环或上下文管理等控制流。初始化或更新了 `std` 等值。

### Lines 225-252 / 第 225-252 行
````python
    {%- set acc_buf_name = "local_acc_buf" %}
        {{ kernel.define_buffer(acc_buf_name, ["Mc_blocks*Mr", "Nc_blocks*Nr"], acc_buf_dtype) }}
{%- endif %}
        for (int64_t mc_block_id = 0; mc_block_id < num_Mc_blocks_per_thread; mc_block_id++) {
            {{ template.codegen_m_loop_params()|indent(12, false) }}
            for (int64_t nc = n_block_start; nc < n_block_end; nc += Nc_blocks) {
                {{ template.codegen_n_loop_params()|indent(16, false) }}
{%- if use_local_acc %}
    {%- set acc = kernel.local_buffers[acc_buf_name] %}
                {{ kernel.reinit_buffer_if_null(acc_buf_name) }}
{%- else %}
    {%- set acc = kernel.slice_nd(GemmOut, [("m_start", "m_end"), ("n_start", "n_end")]) %}
{%- endif %}
                for (int64_t kc = k_block_start; kc < k_block_end; kc += Kc_blocks) {
                    int64_t k_start = kc * Kr;
                    int64_t k_end = std::min(std::min(kc + Kc_blocks, k_block_end) * Kr, K);
{%- set tile_X = kernel.slice_nd(X, [("m_start", "m_end"), ("k_start", "k_end")]) %}
                    for (int64_t nci = nc; nci < nc_block_end; nci++) {
{%- set acc_slice = kernel.slice_nd(acc, [("0", "m_end - m_start"), ("(nci - nc)*Nr", "(nci - nc + 1)*Nr")]) %}
{%- if template.should_block_weights and not is_woq_int4 %}
{%- set tile_W_3d = kernel.slice_nd(W, [("nci", "nci + 1"), ("k_start", "k_end"), ()]) %}
{%- set tile_W = kernel.view(tile_W_3d, ["k_end - k_start", micro_gemm.register_blocking.block_n]) %}
{%- else %}
    {%- if is_woq_int4 %}
        {%- set tile_W = kernel.slice_nd(W, [("nci * Nr", "(nci + 1) * Nr"), ("k_start * Nr / 2", "k_end * Nr / 2")]) %}
        {%- set tile_qparam = kernel.slice_nd(
            qscale_and_zeros, [("k_start // group_size", "k_end // group_size"), ("nci * Nr", "(nci + 1) * Nr"), ()]) %}
    {%- else %}
````
- **EN**: Builds or updates a registry/mapping so later code can dispatch by name or capability. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。包含分支、循环或上下文管理等控制流。

### Lines 253-280 / 第 253-280 行
````python
        {%- set tile_W = kernel.slice_nd(W, [("k_start", "k_end"), ("n_start", "n_start + n_size")]) %}
        {%- set tile_qparam = None %}
    {%- endif %}
{%- endif %}
                        if (kc == k_block_start) {
                            {{ micro_gemm.codegen_call(kernel,
                                                       tile_X,
                                                       tile_W,
                                                       acc_slice,
                                                       accum=False,
                                                       qscale_and_zeros=tile_qparam)|indent(28, false)
                            }}
                        } else {
                            {{ micro_gemm.codegen_call(kernel,
                                                       tile_X,
                                                       tile_W,
                                                       acc_slice,
                                                       accum=True,
                                                       qscale_and_zeros=tile_qparam)|indent(28, false)
                            }}
                        }
                    }
                }
{%- if maybe_k_slicing %}
                if (num_Kt_blocks > 1) {
                    const int64_t mxn_cache_block_id = (mc / Mc_blocks) * num_Nc_blocks + nc;
                    local_buf_ptrs[mxn_cache_block_id * num_Kt_blocks + k_slice_id].reset(
                        {{ kernel.release_buffer(acc_buf_name) }});
````
- **EN**: Manipulates cached state or cache paths to avoid rebuilding the same work repeatedly. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `accum`, and `qscale_and_zeros`.
- **CN**: 这一段处理缓存状态或缓存路径，以避免重复构建相同工作。包含分支、循环或上下文管理等控制流。初始化或更新了 `accum`、`qscale_and_zeros` 等值。

### Lines 281-308 / 第 281-308 行
````python
                } else
{%- endif %}
                {
{%- set tile_Y = kernel.slice_nd(Y_2d, [("m_start", "m_end"), ("n_start", "n_end")]) %}
{%- set tile_acc = kernel.slice_nd(acc, [("0", "m_end - m_start"), ("0", "n_end - n_start")]) %}
                    {{ kernel.store_output(
                        tile_Y, tile_acc, GemmOut, epilogue_nodes, offsets=("m_start", "n_start"), reindexers=reindexers
                    )|indent(20, false)
                    }}
                }
            }
        }
{%- if maybe_k_slicing %}
        if (num_Kt_blocks > 1) {
            #pragma omp barrier
            for (int64_t mc = m_block_start; mc < m_block_end; mc += Mc_blocks) {
                // We slice M-dim and each thread in the k-slicing group works on a slice
                const int64_t m_start_unsliced = mc * Mr;
                const int64_t m_end_unsliced = std::min(std::min(mc + Mc_blocks, m_block_end) * Mr, M);
                const int64_t m_size_unsliced = m_end_unsliced - m_start_unsliced;
                const int64_t m_slice_size = (m_size_unsliced + num_Kt_blocks - 1) / num_Kt_blocks;
                const int64_t m_start = std::min(m_start_unsliced + m_slice_size * k_slice_id, m_end_unsliced);
                const int64_t m_end = std::min(m_start_unsliced + m_slice_size * (k_slice_id + 1), m_end_unsliced);
                const int64_t m_size = m_end - m_start;
                const int64_t m_offset = m_start - m_start_unsliced;
                for (int64_t nc = n_block_start; nc < n_block_end; nc += Nc_blocks) {
                    const int64_t n_start = nc * Nr;
                    const int64_t n_end = std::min(std::min(nc + Nc_blocks, n_block_end) * Nr, N);
````
- **EN**: Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 包含分支、循环或上下文管理等控制流。

### Lines 309-336 / 第 309-336 行
````python
                    const int64_t n_size = n_end - n_start;
                    const int64_t mxn_cache_block_id = (mc / Mc_blocks) * num_Nc_blocks + nc;
                    auto {{acc_buf_name}} = local_buf_ptrs[mxn_cache_block_id * num_Kt_blocks].get();
                    for (int64_t other_slice = 1; other_slice < num_Kt_blocks; other_slice++) {
                        auto other_acc = local_buf_ptrs[mxn_cache_block_id * num_Kt_blocks + other_slice].get();
                        for (int64_t m = m_offset; m < m_offset + m_size; m++) {
                            #pragma omp simd
                            for (int64_t n = 0; n < n_size; n++) {
                                {{acc_buf_name}}[m*Nr + n] += other_acc[m*Nr + n];
                            }
                        }
                    }
    {%- set tile_acc_m_slice = kernel.slice_nd(tile_acc, [("m_offset", "m_offset + m_end - m_start"), ()]) %}
                    {{ kernel.store_output(
                        tile_Y, tile_acc_m_slice, GemmOut, epilogue_nodes, offsets=("m_start", "n_start"), reindexers=reindexers
                    )|indent(20, false)
                    }}
                }
            }
        }
{%- endif %}
        {{ micro_gemm.codegen_finalize(kernel) }}
    }
}
"""

SMALL_M_GEMM_TEMPLATE = r"""
{{ template.codegen_gemm_stub_def() }}
````
- **EN**: Manipulates cached state or cache paths to avoid rebuilding the same work repeatedly. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `SMALL_M_GEMM_TEMPLATE`.
- **CN**: 这一段处理缓存状态或缓存路径，以避免重复构建相同工作。包含分支、循环或上下文管理等控制流。初始化或更新了 `SMALL_M_GEMM_TEMPLATE` 等值。

### Lines 337-364 / 第 337-364 行
````python
{
    {{ kernel.maybe_codegen_profile() }}
    {{ template.codegen_blocks(
        num_threads, N, K, micro_gemm, is_dynamic_M, kernel, GemmOut, config, L1_cache_size, L2_cache_size, X, W
    ) }}
    # pragma omp parallel
    {
        #pragma omp for nowait
        for (int64_t nr_block_id = 0; nr_block_id < Nr_blocks; nr_block_id++) {
            // Handle one output M * Nr block in each thread
            int64_t n_start = nr_block_id * Nr;
            int64_t n_end = (nr_block_id + 1) * Nr;
{%- if use_local_acc %}
    {%- set acc_buf_name = "local_acc_buf" %}
            {{ kernel.define_stack_allocated_buffer(acc_buf_name, ["M", "Nr"], acc_buf_dtype) }}
    {%- set acc = kernel.local_buffers[acc_buf_name] %}
{%- else %}
    {%- set acc = kernel.slice_nd(GemmOut, [(0, "M"), ("n_start", "n_end")]) %}
{%- endif %}
            for (int64_t kr_block_id = 0; kr_block_id < Kr_blocks; kr_block_id++) {
                // this loop is not parallelized
                int64_t k_start = kr_block_id * Kr;
                int64_t k_end = std::min((kr_block_id + 1) * Kr, K);
{%- set tile_X = kernel.slice_nd(X, [(0, "M"), ("k_start", "k_end")]) %}
{%- set tile_W_3d = kernel.slice_nd(W, [("nr_block_id", "nr_block_id + 1"), ("k_start", "k_end"), ()]) %}
{%- set tile_W = kernel.view(tile_W_3d, ["k_end - k_start", micro_gemm.register_blocking.block_n]) %}
                if C10_UNLIKELY(kr_block_id == 0) {
                    {{ micro_gemm.codegen_call(kernel, tile_X, tile_W, acc, accum=False, prefetch=True)|indent(20, false) }}
````
- **EN**: Manipulates cached state or cache paths to avoid rebuilding the same work repeatedly. Builds or updates a registry/mapping so later code can dispatch by name or capability. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这一段处理缓存状态或缓存路径，以避免重复构建相同工作。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。包含分支、循环或上下文管理等控制流。

### Lines 365-392 / 第 365-392 行
````python
                } else if C10_UNLIKELY(k_end == K) {
                    {{ micro_gemm.codegen_call(kernel, tile_X, tile_W, acc, accum=True, prefetch=False)|indent(20, false) }}
                } else {
                    {{ micro_gemm.codegen_call(kernel, tile_X, tile_W, acc, accum=True, prefetch=True)|indent(20, false) }}
                }
            }
{%- set tile_Y = kernel.slice_nd(Y_2d, [("0", "M"), ("n_start", "n_end")]) %}
{%- set tile_acc = kernel.slice_nd(acc, [("0", "M"), ("0", "n_end - n_start")]) %}
            {{ kernel.store_output(
                tile_Y, tile_acc, GemmOut, epilogue_nodes, offsets=("0", "n_start"), reindexers=reindexers
            )|indent(20, false) }}
        }
    }
}
"""


def _is_int8_gemm(inputs):
    return (
        isinstance(inputs[0], ir.IRNode)
        and inputs[0].get_dtype() in [torch.uint8, torch.int8]
    ) or (
        isinstance(inputs[0], torch.Tensor)
        and inputs[0].dtype in [torch.uint8, torch.int8]
    )


def get_padded_n(n, block_n):
````
- **EN**: Introduces function `_is_int8_gemm`, function `get_padded_n`. Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了函数`_is_int8_gemm`、函数`get_padded_n`。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 393-420 / 第 393-420 行
````python
    return (n + block_n - 1) // block_n * block_n


_T = TypeVar("_T", ir.IRNode, torch.Tensor)


def transpose_w(W: _T, trans_w: bool) -> _T:
    """
    Transpose W based on the trans_w flag.
    """
    if isinstance(W, ir.IRNode):
        if trans_w:
            if not isinstance(W, ir.TensorBox):
                # pyrefly: ignore [bad-assignment]
                W = ir.TensorBox(W)
            W = L.permute(W, [1, 0])
    else:
        if trans_w:
            assert isinstance(W, torch.Tensor)
            # pyrefly: ignore [bad-assignment]
            W = W.transpose(0, 1)
    # pyrefly: ignore [bad-return]
    return W


def expand_bias(B: _T | None, X: _T) -> _T | None:
    """
    Expand Bias to the same size of X.
````
- **EN**: Introduces function `transpose_w`, function `expand_bias`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `_T`, `W`, and `else`.
- **CN**: 这里定义了函数`transpose_w`、函数`expand_bias`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `_T`、`W`、`else` 等值。

### Lines 421-448 / 第 421-448 行
````python
    """
    if B is not None:
        if isinstance(B, ir.IRNode):
            if not isinstance(B, ir.TensorBox):
                # pyrefly: ignore [bad-assignment]
                B = ir.TensorBox(B)
            assert hasattr(X, "get_size")
            # pyrefly: ignore [missing-attribute]
            B = L.expand(B, (X.get_size()[0], B.get_size()[-1]))
        else:
            assert isinstance(B, torch.Tensor)
            assert isinstance(X, torch.Tensor)
            # pyrefly: ignore [bad-assignment]
            B = B.expand(X.shape[0], B.shape[-1])
    return B


def prune_tensors(input_nodes: list[ir.IRNode], new_input_nodes: list[ir.IRNode]):
    """
    Prune unused tensors from `V.graph` since the GEMM Template use new packed weight.
    """

    def share_storage(base_tensor: torch.Tensor, comp_tensor: torch.Tensor):
        return base_tensor.is_mkldnn == comp_tensor.is_mkldnn and (
            is_same_tensor(base_tensor, comp_tensor)
            or is_same_mkldnn_tensor(base_tensor, comp_tensor)
        )

````
- **EN**: Introduces function `prune_tensors`, function `share_storage`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `B`, and `else`.
- **CN**: 这里定义了函数`prune_tensors`、函数`share_storage`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `B`、`else` 等值。

### Lines 449-476 / 第 449-476 行
````python
    def get_candidates(input_nodes, new_input_nodes):
        # Only Constant Buffer like weight and bias might be changed in GEMM Template.
        # The Inductor IR Node may changed, but still share the storage. For example:
        # bias in bfloat16 case which only do the expand
        return [
            node
            for node in input_nodes
            if (
                node not in new_input_nodes
                and isinstance(node, (ir.TensorBox, ir.StorageBox))
                and node.get_name() in V.graph.constants
                and not any(
                    (
                        isinstance(new_node, (ir.TensorBox, ir.StorageBox))
                        and new_node.get_name() in V.graph.constants
                        and share_storage(
                            V.graph.constants[node.get_name()],
                            V.graph.constants[new_node.get_name()],
                        )
                    )
                    for new_node in new_input_nodes
                )
            )
        ]

    for candidate_node in get_candidates(input_nodes, new_input_nodes):
        # By using the new packed weight for the GEMM template, we can prune the
        # old weight if it has no other users. This saves memory but makes the FX graph
````
- **EN**: Introduces function `get_candidates`. Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`get_candidates`。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 477-504 / 第 477-504 行
````python
        # non-retraceable. To support retracing, we can add a repack node to the
        # FX graph. For example:
        # mkldnn._linear_pointwise <- repack_linear_wgt <- packed_wgt_for_template
        candidate_tensor_users = 0
        candidate_tensor = V.graph.constants[candidate_node.get_name()]
        for node in reversed(V.graph.graph.nodes):
            # Case may happen when the candidate tensor is used by more than 1 get_attr node
            # https://github.com/pytorch/pytorch/issues/134998
            if node.op == "get_attr" and hasattr(
                V.graph.module, node.target
            ):  # candidate tensor might already be deleted
                comp_tensor = getattr(V.graph.module, node.target)
                if isinstance(comp_tensor, torch.Tensor) and share_storage(
                    candidate_tensor, comp_tensor
                ):
                    candidate_tensor_users += 1

        for node in reversed(V.graph.graph.nodes):
            # The get_attr node has only 1 user fx node
            # The candidate tensor has been used by only 1 get_attr node
            if (
                node.op == "get_attr"
                and node.target == candidate_node.get_name()
                and len(node.users) == 1
                and candidate_tensor_users == 1
            ):
                del V.graph.constants[node.target]
                delattr(V.graph.module, node.target)
````
- **EN**: Preserves reference comments that point to design notes, specifications, or vendor datasheets. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `candidate_tensor_users`, `candidate_tensor`, and `comp_tensor`. This range continues the implementation of function `prune_tensors`.
- **CN**: 保留了指向设计说明、规范或厂商数据手册的注释引用。包含分支、循环或上下文管理等控制流。初始化或更新了 `candidate_tensor_users`、`candidate_tensor`、`comp_tensor` 等值。这一段延续了函数`prune_tensors` 的具体实现。

### Lines 505-532 / 第 505-532 行
````python
                delattr(V.graph.graph.owning_module, node.target)
                counters["inductor"]["select_algorithm_weight_prune"] += 1


def gen_2d_view_of_epilogue_buf(
    Y: ir.Buffer,
    template_buffer: ir.Buffer,
    epilogue_nodes: list[ir.IRNode],
    reindexers: list[Callable[[list[Any]], list[Any]] | None],
    default_reindexers: list[Callable[[list[Any]], list[Any]] | None],
) -> tuple[
    ir.Buffer | ir.ReinterpretView,
    list[Callable[[list[Any]], list[Any]] | None],
]:
    """
    The dimension and the indexing could be different between the GEMM output, i.e. `template_buffer`, which is
    2D with MxN) and the output from the template after epilogues, i.e. `Y`. In the GEMM template code,
    we are not aware of the dimension and the indexing of the epilogues and always work on 2D tiles according to
    the indexing of the GEMM output.
    In this function, we return a 2D buffer (`Y_2d`) according to GEMM output (reinterpreted from `Y` if needed) and
    build a reindexer that converts the indexing of `Y` into `Y_2d`.
    """
    Y_2d: ir.Buffer | ir.ReinterpretView = Y
    if (
        Y.get_size() == template_buffer.get_size()
        and Y.get_stride() == template_buffer.get_stride()
    ):
        reindexers.extend(default_reindexers)
````
- **EN**: Introduces function `gen_2d_view_of_epilogue_buf`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `Y`, `template_buffer`, `epilogue_nodes`, `reindexers`, `default_reindexers`, and `Y_2d`.
- **CN**: 这里定义了函数`gen_2d_view_of_epilogue_buf`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `Y`、`template_buffer`、`epilogue_nodes`、`reindexers`、`default_reindexers`、`Y_2d` 等值。

### Lines 533-560 / 第 533-560 行
````python
        Y_2d = Y
    else:

        def get_reindexer(epilogue_node, default_reindexer=None):
            # From template_buffer to epilogue_node_ordered (ordered by stride decreasingly, in dense format), for example:
            #   template_buffer:
            #       size (324, 512), stride (512, 1)
            #   epilogue_node_ordered (ordered by stride decreasingly, in dense format):
            #       size (1, 18, 18, 512), stride (165888, 9216, 512, 1)
            stride_order = list(
                ir.get_stride_order(
                    V.graph.sizevars.guarding_hints_or_throw(epilogue_node.get_stride())
                )
            )
            fill_order = ir.stride_order2fill_order(stride_order)
            reversed_fill_order = list(reversed(fill_order))
            size_with_stride_ordered_decreasingly = [
                epilogue_node.get_size()[i] for i in reversed_fill_order
            ]
            reshape_reindex = ir.View.dynamic_reshape_indexer(
                size_with_stride_ordered_decreasingly,
                template_buffer.get_size(),
            )
            if default_reindexer:
                reshape_reindex = ir.fuse_reindexing(reshape_reindex, default_reindexer)

            # From epilogue_node_ordered (ordered by stride decreasingly, in dense format) to epilogue_node, for example:
            #   epilogue_node_ordered (ordered by stride decreasingly, in dense format):
````
- **EN**: Introduces function `get_reindexer`. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `Y_2d`, `else`, `stride_order`, `fill_order`, `reversed_fill_order`, `size_with_stride_ordered_decreasingly`, and `...+1`.
- **CN**: 这里定义了函数`get_reindexer`。包含分支、循环或上下文管理等控制流。初始化或更新了 `Y_2d`、`else`、`stride_order`、`fill_order`、`reversed_fill_order`、`size_with_stride_ordered_decreasingly`、`另有1项` 等值。

### Lines 561-588 / 第 561-588 行
````python
            #       size (1, 18, 18, 512), stride (165888, 9216, 512, 1)
            #   epilogue_node:
            #       size (1, 18, 18, 512), stride (165888, 1, 9216, 512)
            from_stride_ordered_decreasingly_to_epilogue_node_order = [
                (len(stride_order) - 1) - stride_order[i]
                for i in range(len(stride_order))
            ]
            stride_reindex = ir.same_reorder(
                from_stride_ordered_decreasingly_to_epilogue_node_order
            )

            reindexer = ir.fuse_reindexing(stride_reindex, reshape_reindex)  # type: ignore[var-annotated]
            return reindexer

        if default_reindexers is None:
            default_reindexers = [None] * len(epilogue_nodes)
        new_reindexers = [
            get_reindexer(epilogue_node, default_reindexer)
            for epilogue_node, default_reindexer in zip(
                epilogue_nodes, default_reindexers
            )
        ]
        reindexers.extend(new_reindexers)
        if isinstance(Y, ir.BaseView):
            storage = ir.StorageBox(Y.unwrap_view())
        else:
            assert isinstance(Y, ir.Buffer)
            storage = ir.StorageBox(Y)
````
- **EN**: Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `from_stride_ordered_decreasingly_to_epilogue_node_order`, `stride_reindex`, `reindexer`, `default_reindexers`, `new_reindexers`, `storage`, and `...+1`. This range continues the implementation of function `gen_2d_view_of_epilogue_buf`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `from_stride_ordered_decreasingly_to_epilogue_node_order`、`stride_reindex`、`reindexer`、`default_reindexers`、`new_reindexers`、`storage`、`另有1项` 等值。这一段延续了函数`gen_2d_view_of_epilogue_buf` 的具体实现。

### Lines 589-616 / 第 589-616 行
````python
        Y_2d = ir.ReinterpretView(data=storage, layout=template_buffer.get_layout())
    return Y_2d, reindexers


class CppGemmTemplate(CppTemplate):
    """
    GEMM Template for Inductor CPP Backend.
    """

    def __init__(
        self,
        input_nodes,
        layout: ir.Layout,
        num_threads: int,
        register_blocking: GemmBlocking,
        beta=1,
        alpha=1,
        has_bias=False,
        epilogue_creator: Callable[[ir.Buffer], ir.Pointwise] | None = None,
        should_block_weights: bool = True,
        name="packed_gemm",
    ) -> None:
        assert layout.dtype in [
            torch.float,
            torch.bfloat16,
            torch.half,
            torch.uint8,
            torch.int8,
````
- **EN**: Introduces class `CppGemmTemplate`, function `__init__`. Tracks dtype-specific behavior or constants that affect numeric execution. Builds or updates a registry/mapping so later code can dispatch by name or capability. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了类`CppGemmTemplate`、函数`__init__`。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 617-644 / 第 617-644 行
````python
        ]
        super().__init__(
            name,
            input_nodes,
            layout,
            num_threads,
            epilogue_creator=epilogue_creator,
        )
        self.beta = beta
        self.alpha = alpha
        self.has_bias = has_bias
        self.register_blocking = register_blocking
        m, n = layout.size[-2:]
        k = input_nodes[0].get_size()[-1]
        self.m, self.n, self.k = m, n, k
        self.padded_n = get_padded_n(n, self.register_blocking.block_n)
        self.is_dynamic_M = has_free_symbols((m,))
        self.should_block_weights = should_block_weights
        self.thread_blocking = self.make_thread_blocking_cache()
        self.cache_blocking = self.make_cache_blocking_cache()

    def make_thread_blocking_cache(self):
        cache = lru_cache()(self._thread_blocking)

        def thread_blocking(num_threads: int) -> GemmBlocking:
            return cache(num_threads)

        return thread_blocking
````
- **EN**: Introduces function `make_thread_blocking_cache`, function `thread_blocking`. Manipulates cached state or cache paths to avoid rebuilding the same work repeatedly. Builds or updates a registry/mapping so later code can dispatch by name or capability. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了函数`make_thread_blocking_cache`、函数`thread_blocking`。这一段处理缓存状态或缓存路径，以避免重复构建相同工作。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 645-672 / 第 645-672 行
````python

    def _thread_blocking(self, num_threads: int) -> GemmBlocking:
        """
        NOTE [Thread blocking in Cpp GEMM]
        We use simple heuristics to decide the thread blocking:
        1. Make sure all threads are occupied as much as possible.
        2. For (m, n) blocks, favor more square-sized thread blocks for better data reuse.
        3. If (m, n) blocks cannot occupy all the threads, we consider k-slicing.
        TODO(jgong5): allow tuning various blocking options
        """

        def get_factors(number):
            factors = []
            for i in range(int(number**0.5), 0, -1):
                if number % i == 0:
                    factors.append(number // i)
                    factors.append(i)
            return factors

        def get_blocking(m_factor, n_factor, k_factor, m_blocks, n_blocks, k_blocks):
            thread_block_k = math.ceil(k_blocks / k_factor)
            thread_block_n = math.ceil(n_blocks / n_factor)
            thread_block_m = math.ceil(m_blocks / m_factor)
            return GemmBlocking(thread_block_m, thread_block_n, thread_block_k)

        assert not self.is_dynamic_M, (
            "Unable to determine thread blocking for dynamic M."
        )
````
- **EN**: Introduces function `_thread_blocking`, function `get_factors`, function `get_blocking`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `factors`, `thread_block_k`, `thread_block_n`, and `thread_block_m`.
- **CN**: 这里定义了函数`_thread_blocking`、函数`get_factors`、函数`get_blocking`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `factors`、`thread_block_k`、`thread_block_n`、`thread_block_m` 等值。

### Lines 673-700 / 第 673-700 行
````python
        register_blocking = self.register_blocking
        m_blocks = math.ceil(self.m / register_blocking.block_m)
        n_blocks = math.ceil(self.n / register_blocking.block_n)
        k_blocks = math.ceil(self.k / register_blocking.block_k)
        factors = get_factors(num_threads)
        assert len(factors) > 0

        if config.cpp.gemm_thread_factors is not None:
            factors = [int(i) for i in config.cpp.gemm_thread_factors.split(",")]
            assert len(factors) == 3
            assert math.prod(factors) == self.num_threads
            return get_blocking(
                factors[0], factors[1], factors[2], m_blocks, n_blocks, k_blocks
            )

        # we favor square-sized thread blocks for good data reuse
        def get_better_blocking(blocking, best_blocking):
            if best_blocking is None:
                best_blocking = blocking
            else:
                block_m_size = blocking.block_m * register_blocking.block_m
                block_n_size = blocking.block_n * register_blocking.block_n
                best_block_m_size = best_blocking.block_m * register_blocking.block_m
                best_block_n_size = best_blocking.block_n * register_blocking.block_n
                if blocking.block_k > best_blocking.block_k:
                    best_blocking = blocking
                elif (
                    blocking.block_k == best_blocking.block_k
````
- **EN**: Introduces function `get_better_blocking`. Builds or updates a registry/mapping so later code can dispatch by name or capability. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`get_better_blocking`。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 701-728 / 第 701-728 行
````python
                    and block_m_size + block_n_size
                    < best_block_m_size + best_block_n_size
                ):
                    best_blocking = blocking
            return best_blocking

        best_blocking = None
        # check if we can have a thread-blocking to occupy all threads without k-slicing
        for n_factor in factors:
            m_factor = num_threads // n_factor
            if n_blocks >= n_factor and m_blocks >= m_factor:
                blocking = get_blocking(
                    m_factor, n_factor, 1, m_blocks, n_blocks, k_blocks
                )
                best_blocking = get_better_blocking(blocking, best_blocking)

        if best_blocking is None:
            for k_factor in factors:
                if k_blocks >= k_factor and (
                    config.cpp.gemm_max_k_slices == 0
                    or k_factor <= config.cpp.gemm_max_k_slices
                ):
                    n_factors = get_factors(num_threads // k_factor)
                    for n_factor in n_factors:
                        m_factor = (num_threads // k_factor) // n_factor
                        if n_blocks >= n_factor and m_blocks >= m_factor:
                            blocking = get_blocking(
                                m_factor,
````
- **EN**: Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `best_blocking`, `m_factor`, `blocking`, and `n_factors`. This range continues the implementation of function `CppGemmTemplate._thread_blocking`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `best_blocking`、`m_factor`、`blocking`、`n_factors` 等值。这一段延续了函数`CppGemmTemplate._thread_blocking` 的具体实现。

### Lines 729-756 / 第 729-756 行
````python
                                n_factor,
                                k_factor,
                                m_blocks,
                                n_blocks,
                                k_blocks,
                            )
                            best_blocking = get_better_blocking(blocking, best_blocking)

        if best_blocking is None:
            for n_factor in factors:
                m_factor = num_threads // n_factor
                if n_blocks >= n_factor or m_blocks >= m_factor:
                    blocking = get_blocking(
                        m_factor, n_factor, 1, m_blocks, n_blocks, k_blocks
                    )
                    best_blocking = get_better_blocking(blocking, best_blocking)

        assert best_blocking is not None
        return best_blocking

    def make_cache_blocking_cache(self):
        cache = lru_cache()(self._cache_blocking)

        def cache_blocking(num_threads: int) -> GemmBlocking:
            return cache(num_threads)

        return cache_blocking

````
- **EN**: Introduces function `make_cache_blocking_cache`, function `cache_blocking`. Manipulates cached state or cache paths to avoid rebuilding the same work repeatedly. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`make_cache_blocking_cache`、函数`cache_blocking`。这一段处理缓存状态或缓存路径，以避免重复构建相同工作。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 757-784 / 第 757-784 行
````python
    def _cache_blocking(self, num_threads: int) -> GemmBlocking:
        def get_cache_blocking(register_blocking, thread_blocking):
            Mr = register_blocking.block_m
            Nr = register_blocking.block_n
            Kr = register_blocking.block_k

            Mt_blocks = thread_blocking.block_m
            Nt_blocks = thread_blocking.block_n
            Kt_blocks = thread_blocking.block_k

            if config.cpp.gemm_cache_blocking is not None:
                blockings = [int(i) for i in config.cpp.gemm_cache_blocking.split(",")]
                assert len(blockings) == 3
                Mc_blocks, Nc_blocks, Kc_blocks = blockings
                return (
                    min(Mc_blocks, Mt_blocks),
                    min(Nc_blocks, Nt_blocks),
                    min(Kc_blocks, Kt_blocks),
                )

            # The ratios below are empirically determined to decide
            # the effective sizes of L1 and L2.
            # TODO: tune the factor here
            L1_limit_factor = 0.8
            L2_limit_factor = 0.5

            L1_cache_size = torch.cpu.get_capabilities().get(
                "l1d_cache_size", 0
````
- **EN**: Introduces function `_cache_blocking`, function `get_cache_blocking`. Manipulates cached state or cache paths to avoid rebuilding the same work repeatedly. Handles device-specific behavior, capabilities, or dispatch decisions. Builds or updates a registry/mapping so later code can dispatch by name or capability.
- **CN**: 这里定义了函数`_cache_blocking`、函数`get_cache_blocking`。这一段处理缓存状态或缓存路径，以避免重复构建相同工作。处理与设备相关的行为、能力信息或派发决策。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。

### Lines 785-812 / 第 785-812 行
````python
            )  # per core cache size in Bytes
            assert L1_cache_size > 0, (
                f"Expect L1_cache_size > 0 but got {L1_cache_size}"
            )
            L1 = L1_cache_size * L1_limit_factor

            L2_cache_size = torch.cpu.get_capabilities().get(
                "l2_cache_size", 0
            )  # per core cache size in Bytes
            assert L2_cache_size > 0, (
                f"Expect L2_cache_size > 0 but got {L2_cache_size}"
            )
            L2 = L2_cache_size * L2_limit_factor

            def get_num_byte(dtype):
                return torch.tensor([], dtype=dtype).element_size()

            dtype_A = self.input_nodes[0].get_dtype()
            dtype_B = self.input_nodes[1].get_dtype()
            num_byte_A = get_num_byte(dtype_A)
            num_byte_B = get_num_byte(dtype_B)
            if dtype_A is torch.bfloat16 and dtype_B is torch.int8 and Kr != 1:
                # We will cache dequantized weights (BF16) in L1D for AMX micro-kernel.
                # In this case, the choice of the micro-kernel being used can't be decoupled from
                # the cache blocking.
                # TODO: Decouple the choice of micro-kernel from cache blocking
                num_byte_B *= num_byte_A

````
- **EN**: Introduces function `get_num_byte`. Manipulates cached state or cache paths to avoid rebuilding the same work repeatedly. Handles device-specific behavior, capabilities, or dispatch decisions. Tracks dtype-specific behavior or constants that affect numeric execution.
- **CN**: 这里定义了函数`get_num_byte`。这一段处理缓存状态或缓存路径，以避免重复构建相同工作。处理与设备相关的行为、能力信息或派发决策。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。

### Lines 813-840 / 第 813-840 行
````python
            # NOTE [CPP GEMM Cache Blocking Algorithm]
            # Our overall strategy is to
            # 1) Make cache blocks of B L1-reside and reused by multiple rows of A, i.e. Mc.
            #    Here, B is Kc x Nr where Nr is a single register block. We use L1 size to
            #    decide Kc. We want to make Mc large enough to better reuse B.
            # 2) Make cache blocks of A L2-reside, which would limit Mc. We want to reuse A
            #    along N, where we have two sub-strategies (see notes below) to decide Mc and Nc.

            # Step 1: Decide Kc assuming B block is L1-reside.
            size_cache_B = Kr * Kt_blocks * Nr * num_byte_B

            Kc_blocks = Kt_blocks
            if size_cache_B > L1:
                Kc_blocks = math.floor(L1 / (Kr * Nr * num_byte_B))

            if (
                config.cpp.use_small_dequant_buffer
                and dtype_A is torch.bfloat16
                and Mt_blocks == 1
            ):
                if dtype_B is torch.uint8:
                    # A16W4
                    # Make a small dequant_B buffer for woq int4 [q_group_size, Nr]
                    # Since when Mt_blocks == 1, L1-reside B block can't be reused by A.
                    if Kc_blocks * Kr >= self.q_group_size():
                        Kc_blocks = self.q_group_size() // Kr

                elif dtype_B is torch.int8:
````
- **EN**: Manipulates cached state or cache paths to avoid rebuilding the same work repeatedly. Tracks dtype-specific behavior or constants that affect numeric execution. Builds or updates a registry/mapping so later code can dispatch by name or capability. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这一段处理缓存状态或缓存路径，以避免重复构建相同工作。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。包含分支、循环或上下文管理等控制流。

### Lines 841-868 / 第 841-868 行
````python
                    # A16W8
                    # Make A, B, C buffer in L1
                    A_buf_size_div_K = self.m * num_byte_A
                    B_buf_size_div_K = Nr * num_byte_B
                    # assume acc in float32/int32 and Mc_blocks = Nc_blocks = 1
                    C_buf_size = Mr * Nr * 4
                    K_block_size = (L1 - C_buf_size) // (
                        A_buf_size_div_K + B_buf_size_div_K
                    )
                    if Kc_blocks * Kr >= K_block_size:
                        Kc_blocks = (K_block_size + Kr - 1) // Kr

            # Step 2: Decide Mc assuming A block is L2-reside.
            min_Mc_ratio = 2  # TODO(jgong5): something to tune?
            min_Mc_blocks = math.ceil(min_Mc_ratio * Mr / Nr)
            assert min_Mc_blocks >= 1
            Kt_bytes = Kt_blocks * Kr * num_byte_A
            if min_Mc_blocks * Mr * Kt_bytes < L2:
                # Strategy 1: A (Mc x Kt) resides in L2 and reused by all Nt
                # when Nc_blocks is kept 1. Mc should be large enough (>= min_Mc_blocks)
                # to reuse B (Kc x Nr) in L1. This makes C (Mc x Nr) small enough to reside
                # in L1.
                Mc_blocks = min(Mt_blocks, math.floor(L2 / (Mr * Kt_bytes)))
                Nc_blocks = 1
            else:
                # Strategy 2: Kt is too large to hold A (Mc x Kt) in L2, we reuse
                # A (Mc x Kc) in L2 by B (Kc x Nc). C (Mc x Nc) resides in L2.
                Mc_blocks = Mt_blocks
````
- **EN**: Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `A_buf_size_div_K`, `B_buf_size_div_K`, `C_buf_size`, `K_block_size`, `Kc_blocks`, `min_Mc_ratio`, and `...+5`.
- **CN**: 追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `A_buf_size_div_K`、`B_buf_size_div_K`、`C_buf_size`、`K_block_size`、`Kc_blocks`、`min_Mc_ratio`、`另有5项` 等值。

### Lines 869-896 / 第 869-896 行
````python
                Nc_blocks = min(math.ceil(Mc_blocks * Mr / Nr), Nt_blocks)
                Nc_bytes = Nc_blocks * Nr * 4  # assume C or acc is float32/int32
                Kc_bytes = Kc_blocks * Kr * num_byte_A
                if Mc_blocks * Mr * (Kc_bytes + Nc_bytes) > L2:
                    # The following is the solution for 4*Mc*Nc + Mc*Kc_bytes = L2,
                    # assuming Mc == Nc for good data reuse.
                    M_max = (math.sqrt(Kc_bytes * Kc_bytes + 16 * L2) - Kc_bytes) / 8
                    if M_max < Mc_blocks * Mr:
                        Mc_blocks = math.floor(M_max / Mr)
                        Nc_blocks = min(math.ceil(Mc_blocks * Mr / Nr), Nt_blocks)

            return Mc_blocks, Nc_blocks, Kc_blocks

        assert not self.is_dynamic_M, (
            "Unable to determine cache blocking for dynamic M."
        )
        register_blocking = self.register_blocking
        thread_blocking = self.thread_blocking(num_threads)

        return GemmBlocking(*get_cache_blocking(register_blocking, thread_blocking))

    def log_blockings(self):
        log.debug(f"Register blocking: {self.register_blocking}")  # noqa: G004
        if self.is_dynamic_M:
            # thread and cache blockings are determined at runtime for dynamic shapes
            return
        log.debug(
            f"Cache blocking: {self.cache_blocking(self.num_threads)}"  # noqa: G004
````
- **EN**: Introduces function `log_blockings`. Manipulates cached state or cache paths to avoid rebuilding the same work repeatedly. Tracks dtype-specific behavior or constants that affect numeric execution. Builds or updates a registry/mapping so later code can dispatch by name or capability.
- **CN**: 这里定义了函数`log_blockings`。这一段处理缓存状态或缓存路径，以避免重复构建相同工作。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。

### Lines 897-924 / 第 897-924 行
````python
        )
        thread_blocking = self.thread_blocking(self.num_threads)
        log.debug(f"Thread blocking: {thread_blocking}")  # noqa: G004

        def get_occupancy():
            m_blocks = math.ceil(self.m / self.register_blocking.block_m)
            n_blocks = math.ceil(self.n / self.register_blocking.block_n)
            k_blocks = math.ceil(self.k / self.register_blocking.block_k)
            m = math.ceil(m_blocks / thread_blocking.block_m)
            n = math.ceil(n_blocks / thread_blocking.block_n)
            k = math.ceil(k_blocks / thread_blocking.block_k)
            return (m, n, k)

        log.debug(
            f"Number of threads: {self.num_threads}, occupancy: {get_occupancy()}"  # noqa: G004
        )

    def maybe_k_slicing(self):
        if self.num_threads == 1:
            return False
        if self.is_dynamic_M:
            # TODO(jgong5): perhaps use size hint to decide?
            return True
        register_blocking = self.register_blocking
        k_blocks = math.ceil(self.k / register_blocking.block_k)
        thread_blocking = self.thread_blocking(self.num_threads)
        return k_blocks > thread_blocking.block_k

````
- **EN**: Introduces function `get_occupancy`, function `maybe_k_slicing`. Builds or updates a registry/mapping so later code can dispatch by name or capability. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`get_occupancy`、函数`maybe_k_slicing`。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 925-952 / 第 925-952 行
````python
    @classmethod
    def add_choices(
        cls,
        choices,
        layout,
        input_nodes,
        beta=1,
        alpha=1,
        has_bias=False,
        trans_w=False,
        input_indices=None,
        epilogue_creator: Callable[[ir.Buffer], ir.Pointwise] | None = None,
        act_mapping: dict[int, ir.IRNode] | None = None,
    ):
        """
        Add choices for the GEMM template.
        """
        # Fast path to save the epilogue calculation when x_scale/x_zp/w_scale are constant
        use_int8_fast_compensation_path = _is_int8_gemm(input_nodes) and all(
            (
                isinstance(input_nodes[idx], ir.TensorBox)
                and isinstance(input_nodes[idx].data.data, ir.ConstantBuffer)
            )
            for idx in [1, 2, 4]
        )

        if input_indices is None:
            input_indices = list(range(len(input_nodes)))
````
- **EN**: Introduces function `add_choices`. Applies decorators to register behavior or alter how the following definition is constructed. Builds or updates a registry/mapping so later code can dispatch by name or capability. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`add_choices`。使用装饰器来注册行为，或改变后续定义的构造方式。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。包含分支、循环或上下文管理等控制流。

### Lines 953-980 / 第 953-980 行
````python

        def reorder_and_filter(inputs, layout_or_out):
            if has_bias:
                assert len(input_indices) >= 3
                # Assume the input order is [inp, x, w] and we reorder it to [x, w, inp]
                inp_idx = input_indices[0]
                x_idx = input_indices[1]
                w_idx = input_indices[2]
                return [
                    inputs[x_idx],
                    inputs[w_idx],
                    inputs[inp_idx],
                    *[inputs[idx] for idx in input_indices[3:]],
                ], layout_or_out
            elif len(inputs) >= len(input_indices):
                assert len(input_indices) >= 2
                return [inputs[idx] for idx in input_indices], layout_or_out
            else:
                # For when input is used for x and w, i.e. X@X.T or similar
                # Assumes the first input is the only input
                assert len(inputs) == 1
                return [inputs[0]] * len(input_indices), layout_or_out

        new_inputs, new_layout = reorder_and_filter(input_nodes, layout)
        is_mkldnn_wgt = (
            new_inputs[1].get_name() in V.graph.constants
            and V.graph.constants[new_inputs[1].get_name()].is_mkldnn
        )
````
- **EN**: Introduces function `reorder_and_filter`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `inp_idx`, `x_idx`, `w_idx`, `else`, and `is_mkldnn_wgt`.
- **CN**: 这里定义了函数`reorder_and_filter`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `inp_idx`、`x_idx`、`w_idx`、`else`、`is_mkldnn_wgt` 等值。

### Lines 981-1008 / 第 981-1008 行
````python
        if is_mkldnn_wgt:
            # It shouldn't happen as viewing an mkldnn tensor, we can extend the
            # implementation if it does.
            assert not isinstance(new_inputs[1], ir.BaseView)
        # Note that the layout of MKLDNN Tensor is with the wrong stride
        view_size = new_inputs[1].layout.size
        view_stride = new_inputs[1].layout.stride
        view_offset = new_inputs[1].layout.offset

        def maybe_to_dense(inputs, layout_or_out):
            new_inputs = list(inputs)
            if isinstance(inputs[1], torch.Tensor):
                W = inputs[1]
                new_inputs[1] = W.to_dense() if W.is_mkldnn else W
            return new_inputs, layout_or_out

        def normalize_shapes(inputs, layout_or_out):
            new_inputs = list(inputs)
            if not is_mkldnn_wgt and isinstance(new_inputs[1], torch.Tensor):
                if has_free_symbols(view_size):
                    # If batch size B is dynamic, we need to set the batch size and possibly stride
                    assert not has_free_symbols(view_size[1:])
                    view_size[:] = V.graph.sizevars.guarding_hints_or_throw(view_size)
                    view_stride[:] = V.graph.sizevars.guarding_hints_or_throw(
                        view_stride
                    )
                # With the assumptation that W is the storage of unwrap view
                # thus view it back here
````
- **EN**: Introduces function `maybe_to_dense`, function `normalize_shapes`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `view_size`, `view_stride`, `view_offset`, `new_inputs`, and `W`.
- **CN**: 这里定义了函数`maybe_to_dense`、函数`normalize_shapes`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `view_size`、`view_stride`、`view_offset`、`new_inputs`、`W` 等值。

### Lines 1009-1036 / 第 1009-1036 行
````python
                new_inputs[1] = new_inputs[1].as_strided(
                    view_size, view_stride, view_offset
                )

            if not trans_w:
                return new_inputs, layout_or_out
            X = new_inputs[0]
            W = new_inputs[1]
            B = new_inputs[2] if has_bias else None
            W = transpose_w(W, trans_w)
            B = expand_bias(B, X)  # type:ignore[arg-type]
            new_inputs[1] = W
            if B is not None:
                new_inputs[2] = B
            return new_inputs, layout_or_out

        # TODO(jgong5): decide proper number of threads per problem size
        num_threads = parallel_num_threads()
        new_inputs, _ = normalize_shapes(*maybe_to_dense(new_inputs, new_layout))
        m, n, k, *_ = mm_args(
            new_inputs[0],
            new_inputs[1],
            mat2_transposed=cls.is_woq_int4(),
            use_4x2_dim=cls.is_woq_int4(),
        )
        output_dtype, compute_dtype = get_gemm_template_output_and_compute_dtype(
            new_inputs[0].get_dtype()
        )
````
- **EN**: Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `X`, `W`, `B`, `num_threads`, `mat2_transposed`, and `use_4x2_dim`. This range continues the implementation of function `CppGemmTemplate.add_choices.normalize_shapes`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `X`、`W`、`B`、`num_threads`、`mat2_transposed`、`use_4x2_dim` 等值。这一段延续了函数`CppGemmTemplate.add_choices.normalize_shapes` 的具体实现。

### Lines 1037-1064 / 第 1037-1064 行
````python
        micro_gemm = create_micro_gemm(
            "micro_gemm",
            m,
            n,
            k,
            input_dtype=new_inputs[0].get_dtype(),
            input2_dtype=new_inputs[1].get_dtype(),
            output_dtype=output_dtype,
            compute_dtype=compute_dtype,
            alpha=alpha,
            num_threads=num_threads,
            use_ref=not cls.is_woq_int4(),
            q_group_size=cls.q_group_size(),
        )
        assert micro_gemm is not None
        pre_block_weights = cls.check_if_block_weight(new_inputs[1], micro_gemm)
        micro_gemm.use_local_vnni_blocking(not pre_block_weights)
        only_one_input = (
            input_nodes[0] == input_nodes[1] if len(input_nodes) > 1 else False
        ) and not pre_block_weights  # If weights are blocked, use the second input

        def preprocessor(inputs, layout):
            new_inputs, new_layout = normalize_shapes(
                *maybe_to_dense(*reorder_and_filter(inputs, layout))
            )
            if only_one_input and isinstance(new_inputs[0], torch.Tensor):
                return new_inputs[1:], new_layout
            return cls.prep_weight(
````
- **EN**: Introduces function `preprocessor`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `micro_gemm`, `input_dtype`, `input2_dtype`, `output_dtype`, `compute_dtype`, `alpha`, and `...+5`.
- **CN**: 这里定义了函数`preprocessor`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `micro_gemm`、`input_dtype`、`input2_dtype`、`output_dtype`、`compute_dtype`、`alpha`、`另有5项` 等值。

### Lines 1065-1092 / 第 1065-1092 行
````python
                new_inputs,
                new_layout,
                # pyrefly: ignore [bad-argument-type]
                micro_gemm,
                pre_block_weights,
                use_int8_fast_compensation_path,
            )

        def postprocessor(output):
            if isinstance(output, ir.TensorBox):
                # prepack the weight as input to the template buffer
                template_buffer = ir.InputsKernel.unwrap_storage_for_input(output)
                assert isinstance(template_buffer, ir.CppTemplateBuffer)
                new_input_nodes, _ = reorder_and_filter(input_nodes, layout)

                W_node = new_input_nodes[1]
                if W_node.get_name() not in V.graph.constants:
                    return output
                W = V.graph.constants[W_node.get_name()]
                new_input_nodes[1] = W
                new_input_nodes, new_layout = normalize_shapes(
                    *maybe_to_dense(new_input_nodes, layout)
                )
                new_input_nodes, _ = cls.prep_weight(
                    new_input_nodes,
                    new_layout,
                    # pyrefly: ignore [bad-argument-type]
                    micro_gemm,
````
- **EN**: Introduces function `postprocessor`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `template_buffer`, `W_node`, and `W`.
- **CN**: 这里定义了函数`postprocessor`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `template_buffer`、`W_node`、`W` 等值。

### Lines 1093-1120 / 第 1093-1120 行
````python
                    pre_block_weights,
                    use_int8_fast_compensation_path,
                    skip_int8_compensation=True,
                )
                W_packed = new_input_nodes[1]
                W_packed_constant = V.graph.add_tensor_constant(W_packed)
                new_input_nodes[1] = W_packed_constant

                # Prune unused tensors
                prune_tensors(input_nodes, new_input_nodes)

                template_buffer.inputs[1] = ir.InputsKernel.unwrap_storage_for_input(
                    W_packed_constant
                )
            return output

        template = DataProcessorTemplateWrapper(
            cls,
            preprocessor,
            postprocessor,
            input_nodes=input_nodes,
            layout=layout,
            num_threads=num_threads,
            register_blocking=micro_gemm.register_blocking,
            beta=beta,
            alpha=alpha,
            has_bias=has_bias,
            epilogue_creator=epilogue_creator,
````
- **EN**: Builds or updates a registry/mapping so later code can dispatch by name or capability. Includes returns or checks that define the contract of this code path. Initializes or updates values such as `skip_int8_compensation`, `W_packed`, `W_packed_constant`, `template`, `input_nodes`, `layout`, and `...+6`. This range continues the implementation of function `CppGemmTemplate.add_choices.postprocessor`.
- **CN**: 构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。初始化或更新了 `skip_int8_compensation`、`W_packed`、`W_packed_constant`、`template`、`input_nodes`、`layout`、`另有6项` 等值。这一段延续了函数`CppGemmTemplate.add_choices.postprocessor` 的具体实现。

### Lines 1121-1148 / 第 1121-1148 行
````python
            should_block_weights=pre_block_weights,
            name=micro_gemm.__class__.__name__,
        )
        template.maybe_append_choice(choices)
        return template

    @staticmethod
    def get_padded_size(n, block_n, k, should_block_weight):
        padded_n = get_padded_n(n, block_n)
        # We assume that all GEMM weight tensors should be blocked and padded
        new_size = [padded_n // block_n, k, block_n]
        return new_size, padded_n

    @staticmethod
    def _maybe_remove_storage_offset(node: ir.IRNode):
        if node.get_layout().offset == 0:
            return node
        # node may be contiguous but still have a non-zero storage offset.
        # GEMM_TEMPLATE emits code like:
        #   W.data_ptr[node.offset + ...]
        # but runtime W.data_ptr (after normalize_shapes()) already includes this offset.
        # To avoid double-offsetting, we remove the offset in the node also in the generated code.
        #   W.data_ptr[...]
        return ir.ExternKernel.copy_input(node)

    @classmethod
    def prep_weight(
        cls,
````
- **EN**: Introduces function `get_padded_size`, function `_maybe_remove_storage_offset`, function `prep_weight`. Applies decorators to register behavior or alter how the following definition is constructed. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`get_padded_size`、函数`_maybe_remove_storage_offset`、函数`prep_weight`。使用装饰器来注册行为，或改变后续定义的构造方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 1149-1176 / 第 1149-1176 行
````python
        inputs,
        layout: ir.Layout,
        micro_gemm: CppMicroGemm,
        should_block_weight: bool,
        use_int8_fast_compensation_path: bool = False,
        skip_int8_compensation: bool = False,
    ):
        """
        NOTE Weight prep consists of 2 separate steps:
        1. Blocking the weight tensor into a 3D shape: [n//block_n, k, block_n]
           This is always done if the weight tensor is constant, i.e. for all GEMM and some BMM.
           For BMM, we also block non-contiguous weight tensors, since they would be reshaped anyway.
           This assumes that blocked, contiguous weights will be more efficient for the GEMM kernel,
           and is worth the overhead of reshape and blocking.

           This blocking includes additional padding, when n is not a multiple of block_n.
           This padding allows a more efficient microkernel implementation. For BMM, this is only done
           if reshape would happen anyway, i.e.  if the weight tensor is constant, is not contiguous,
           or is using AMX VNNI layout.
        2. Packing the weight tensor into a VNNI-friendly shape. For constant input,
           this is done at the same time as the weight blocking.

        At compile time, the constant weight tensors are blocked and packed. For non-constant tensors (e.g. BMM)
        which will be blocked (non-contiguous or VNNI-layout tensors), the weight tensor is blocked and packed at runtime.

        CppBmmTemplate overrides the methods get_padded_size, and block_weight in order to accommodate
        an additional dimension for the batch size and to determine if the weight tensor should be blocked.
        """
````
- **EN**: Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `layout`, `micro_gemm`, `should_block_weight`, `use_int8_fast_compensation_path`, and `skip_int8_compensation`. This range continues the implementation of function `CppGemmTemplate.prep_weight`.
- **CN**: 包含分支、循环或上下文管理等控制流。初始化或更新了 `layout`、`micro_gemm`、`should_block_weight`、`use_int8_fast_compensation_path`、`skip_int8_compensation` 等值。这一段延续了函数`CppGemmTemplate.prep_weight` 的具体实现。

### Lines 1177-1204 / 第 1177-1204 行
````python
        W = inputs[1]
        new_inputs = list(inputs)
        if cls.is_woq_int4():
            assert (
                len(W.get_size()) == 2
                if isinstance(W, ir.IRNode)
                else len(W.shape) == 2
            )
            n, k = W.get_size() if isinstance(W, ir.IRNode) else W.shape
        else:
            k, n = W.get_size()[-2:] if isinstance(W, ir.IRNode) else W.shape[-2:]
        _, block_n, _ = micro_gemm.register_blocking
        new_size, padded_n = cls.get_padded_size(n, block_n, k, should_block_weight)
        padding = padded_n - n

        if should_block_weight and not cls.is_woq_int4():
            blocked_w = cls.block_weight(W, new_size, padding)
            new_inputs[1] = cls.pack_vnni_weight(blocked_w, micro_gemm, new_size)
        elif should_block_weight:
            assert cls.is_woq_int4()
            new_inputs[1] = cls.block_weight(W, new_size, padding)
        elif isinstance(W, ir.IRNode):
            # Require W layout to be fixed & contiguous, happens inplace.
            ir.ExternKernel.require_contiguous(W)
            new_inputs[1] = cls._maybe_remove_storage_offset(W)

        if not skip_int8_compensation and _is_int8_gemm(new_inputs):
            BCompensate = None
````
- **EN**: Builds or updates a registry/mapping so later code can dispatch by name or capability. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `W`, `new_inputs`, `else`, `padding`, `blocked_w`, and `BCompensate`.
- **CN**: 构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `W`、`new_inputs`、`else`、`padding`、`blocked_w`、`BCompensate` 等值。

### Lines 1205-1232 / 第 1205-1232 行
````python
            x_w_scale = None

            def _get_compensation_node(W, use_int8_fast_compensation_path):
                BCompensate = V.graph.add_tensor_constant(
                    V.graph.constants[W.get_name() + "_BMatrixCompens"],
                    W.get_name() + "_BMatrixCompens",
                )
                x_w_scale = None
                if use_int8_fast_compensation_path:
                    x_w_scale = V.graph.add_tensor_constant(
                        V.graph.constants[W.get_name() + "_x_w_compens"],
                        W.get_name() + "_x_w_compens",
                    )
                return BCompensate, x_w_scale

            if use_int8_fast_compensation_path:
                # new_inputs has been reordered: [x, w, optional[bias], x_scale, x_zp, w_scale, w_zp]
                x_scale = new_inputs[-4]
                x_zp = new_inputs[-3]
                w_scale = new_inputs[-2]
                if isinstance(W, ir.IRNode):
                    BCompensate, x_w_scale = _get_compensation_node(
                        W, use_int8_fast_compensation_path
                    )
                else:
                    # Use the original W, not the blocked_w in new_inputs[1] to calculate BCompensate
                    BCompensate = torch.sum(W.to_dense().to(torch.float), dim=0)  # type: ignore[assignment]
                    assert all(
````
- **EN**: Introduces function `_get_compensation_node`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `x_w_scale`, `BCompensate`, `x_scale`, `x_zp`, `w_scale`, and `else`.
- **CN**: 这里定义了函数`_get_compensation_node`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `x_w_scale`、`BCompensate`、`x_scale`、`x_zp`、`w_scale`、`else` 等值。

### Lines 1233-1260 / 第 1233-1260 行
````python
                        isinstance(item, torch.Tensor)
                        for item in (x_scale, x_zp, w_scale)
                    )
                    BCompensate = BCompensate * x_scale * w_scale * x_zp
                    x_w_scale = x_scale * w_scale
                new_inputs.append(BCompensate)
                new_inputs.append(x_w_scale)
            else:
                if isinstance(W, ir.IRNode):
                    BCompensate, _ = _get_compensation_node(
                        W, use_int8_fast_compensation_path
                    )
                else:
                    # Use the original W, not the blocked_w in new_inputs[1] to calculate BCompensate
                    BCompensate = torch.sum(W.to_dense().to(torch.float), dim=0)  # type: ignore[assignment]
                new_inputs.append(BCompensate)
        return new_inputs, layout

    @staticmethod
    def check_if_block_weight(W, micro_gemm):
        return True

    @classmethod
    def block_weight(cls, W, new_size, padding):
        # These are separated into two methods to allow subclasses to override them separately
        if isinstance(W, ir.IRNode):
            if W.get_name() in V.graph.constants:
                # Create a new buffer, representing the constant blocked tensor
````
- **EN**: Introduces function `check_if_block_weight`, function `block_weight`. Applies decorators to register behavior or alter how the following definition is constructed. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`check_if_block_weight`、函数`block_weight`。使用装饰器来注册行为，或改变后续定义的构造方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 1261-1288 / 第 1261-1288 行
````python
                blocked_w = ir.Buffer(
                    name=W.get_name(),  # Borrow the registered buffer name
                    layout=ir.FixedLayout(
                        W.get_device_or_error(),
                        W.get_dtype(),
                        new_size,
                        ir.FlexibleLayout.contiguous_strides(new_size),
                        0,
                    ),
                )
            else:
                if not isinstance(W, ir.TensorBox):
                    W = ir.TensorBox(W)
                permute_dims = list(range(len(new_size)))
                permute_dims[-2], permute_dims[-3] = permute_dims[-3], permute_dims[-2]
                permute_size = list(new_size)
                permute_size[-2], permute_size[-3] = permute_size[-3], permute_size[-2]
                blocked_w = L.constant_pad_nd(W, (0, padding))
                blocked_w = L.permute(
                    L.view(blocked_w, permute_size),  # type: ignore[arg-type]
                    permute_dims,
                )
        else:
            assert isinstance(W, torch.Tensor)
            # Pad the weight tensor and reshape it into a 3D blocked shape
            blocked_size = list(new_size)
            blocked_size[-2], blocked_size[-3] = blocked_size[-3], blocked_size[-2]
            blocked_w = (
````
- **EN**: Builds or updates a registry/mapping so later code can dispatch by name or capability. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `blocked_w`, `name`, `layout`, `else`, `W`, `permute_dims`, and `...+2`.
- **CN**: 构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `blocked_w`、`name`、`layout`、`else`、`W`、`permute_dims`、`另有2项` 等值。

### Lines 1289-1316 / 第 1289-1316 行
````python
                torch.nn.functional.pad(W, (0, padding))  # type: ignore[assignment]
                .reshape(*blocked_size)
                .transpose(-3, -2)
                .contiguous()
            )
        return blocked_w

    @classmethod
    def pack_vnni_weight(cls, W, micro_gemm, new_size):
        # WOQ INT4 weights are reordered in microkernel so do not pack them here
        should_pack = (
            micro_gemm.get_b_layout() != LayoutType.NORMAL
            and not micro_gemm.is_woq_int4()
        )

        # These are separated into two methods to allow subclasses to override them separately
        if isinstance(W, ir.IRNode):
            if isinstance(W, ir.Buffer) and W.get_name() in V.graph.constants:
                return W
            k = new_size[-2]
            if not isinstance(W, ir.TensorBox):
                W = ir.TensorBox(W)
            if should_pack:
                permute_dims = list(range(len(new_size) + 1))
                permute_dims[-1], permute_dims[-2] = permute_dims[-2], permute_dims[-1]
                vnni_size = 4 if micro_gemm.get_b_layout() == LayoutType.VNNI4 else 2
                vnni_view_size = list(new_size)
                vnni_view_size[-2] = k // vnni_size
````
- **EN**: Introduces function `pack_vnni_weight`. Applies decorators to register behavior or alter how the following definition is constructed. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`pack_vnni_weight`。使用装饰器来注册行为，或改变后续定义的构造方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 1317-1344 / 第 1317-1344 行
````python
                vnni_view_size.insert(-1, vnni_size)
                W = L.view(
                    L.permute(L.view(W, vnni_view_size), permute_dims),
                    new_size,
                )
            W = ir.ExternKernel.realize_input(W)
            W = ir.ExternKernel.require_contiguous(W)
            return W
        else:
            k = new_size[-2]
            # Apply VNNI packing to the weight tensor
            if should_pack:
                # TODO: Move VNNI weight packing for non-constant tensors into the template,
                # to improve cache locality and avoid full-tensor copy.
                layout_str = (
                    "VNNI4"
                    if micro_gemm.get_b_layout() == LayoutType.VNNI4
                    else "VNNI2"
                )
                assert micro_gemm.get_b_layout() in [
                    LayoutType.VNNI2,
                    LayoutType.VNNI4,
                ], f"We only support {layout_str} for now"
                vnni_size = 4 if micro_gemm.get_b_layout() == LayoutType.VNNI4 else 2
                assert k % vnni_size == 0, (
                    f"k should be divisible by vnni_size for {layout_str} layout"
                )
                vnni_view_size = list(new_size)
````
- **EN**: Manipulates cached state or cache paths to avoid rebuilding the same work repeatedly. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `W`, `else`, `k`, `layout_str`, `vnni_size`, and `vnni_view_size`.
- **CN**: 这一段处理缓存状态或缓存路径，以避免重复构建相同工作。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `W`、`else`、`k`、`layout_str`、`vnni_size`、`vnni_view_size` 等值。

### Lines 1345-1372 / 第 1345-1372 行
````python
                vnni_view_size[-2] = k // vnni_size
                vnni_view_size.insert(-1, vnni_size)
                W = W.view(vnni_view_size).transpose(-1, -2).contiguous().view(new_size)
            # normalize stride to be "contiguous_strides" per size
            # this avoids the problems in L.view during template codegen
            new_stride = [1]
            for sz in reversed(W.shape[1:]):
                new_stride.insert(0, new_stride[0] * sz)
            W = W.as_strided(W.shape, new_stride)
            return W

    def get_default_reindexers(self, epilogue_nodes):
        return [None] * len(epilogue_nodes)

    def get_options(
        self,
        kernel: CppTemplateKernel,
        template_buffer_node: ir.CppTemplateBuffer | None = None,
        flag_template_buffer_has_other_users: bool | None = None,
        epilogue_nodes: list[ir.IRNode] | None = None,
    ) -> dict[str, Any]:
        assert len(self.input_nodes) >= 2

        int8_gemm = self.input_nodes[0].get_dtype() in [torch.uint8, torch.int8]
        x_scale = None
        x_zp = None
        w_scale = None
        w_zp = None
````
- **EN**: Introduces function `get_default_reindexers`, function `get_options`. Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`get_default_reindexers`、函数`get_options`。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 1373-1400 / 第 1373-1400 行
````python
        inp = None
        q_group_size_node = None
        qscale_and_zeros = None
        if int8_gemm:
            X, W = self.input_nodes[0], self.input_nodes[1]
            bias_idx = 2 if self.has_bias else 1
            inp = self.input_nodes[bias_idx] if self.has_bias else None
            x_scale = self.input_nodes[bias_idx + 1]
            x_zp = self.input_nodes[bias_idx + 2]
            w_scale = self.input_nodes[bias_idx + 3]
            w_zp = self.input_nodes[bias_idx + 4]
            Y = self.output_node
        elif self.is_woq_int4():
            X, W = self.input_nodes[0], self.input_nodes[1]
            Y = self.output_node
            q_group_size_node = self.input_nodes[2]
            qscale_and_zeros = self.input_nodes[3]
        else:
            X, W = self.input_nodes[0], self.input_nodes[1]
            Y = self.output_node
            inp = self.input_nodes[2] if self.has_bias else None

        template_buffer_has_other_users = None

        if template_buffer_node is not None:
            # Use the updated prepacked weight buffer
            W = template_buffer_node.inputs[1]
            Y = template_buffer_node
````
- **EN**: Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `inp`, `q_group_size_node`, `qscale_and_zeros`, `bias_idx`, `x_scale`, `x_zp`, and `...+6`. This range continues the implementation of function `CppGemmTemplate.get_options`.
- **CN**: 包含分支、循环或上下文管理等控制流。初始化或更新了 `inp`、`q_group_size_node`、`qscale_and_zeros`、`bias_idx`、`x_scale`、`x_zp`、`另有6项` 等值。这一段延续了函数`CppGemmTemplate.get_options` 的具体实现。

### Lines 1401-1428 / 第 1401-1428 行
````python

            assert flag_template_buffer_has_other_users is not None
            template_buffer_has_other_users = flag_template_buffer_has_other_users

        template_buffer = Y
        gemm_output_buffer = template_buffer

        epilogues: list[ir.IRNode] = []
        reindexers: list[Callable[[list[Any]], list[Any]] | None] = []
        epilogue_creators: list[Callable[[ir.Buffer], ir.Pointwise]] = []
        fake_buffers: list[ir.Buffer] = []
        Y_aliases: OrderedSet[str] = OrderedSet()

        use_local_acc = (
            self.layout.dtype != torch.float
            or template_buffer_has_other_users
            or int8_gemm
            or self.padded_n != self.n
            or self.maybe_k_slicing()
            or (epilogue_nodes and epilogue_nodes[-1].get_dtype() != self.layout.dtype)
        )

        # TODO(jgong5): for int8 gemm, bias-add is handled outside of gemm template,
        # but we'd better move it here to align with fp.
        if inp is not None and self.beta != 0 and not int8_gemm:
            # add an epilogue for bias add
            def _bias_add_epilogue(buf):
                return create_epilogue_with_attr(
````
- **EN**: Introduces function `_bias_add_epilogue`. Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`_bias_add_epilogue`。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 1429-1456 / 第 1429-1456 行
````python
                    buf, "bias_add", other=inp, beta=self.beta, dtype=self.layout.dtype
                )

            epilogue_creators.append(_bias_add_epilogue)

        if self.epilogue_creator is not None:
            epilogue_creators.append(self.epilogue_creator)

        # When the GEMM output buffer is localized but it has users other than the epilogue nodes,
        # we need to copy the value in the GEMM output local buffer to a global buffer.
        def need_copy_from_local_to_global_buffer_epilogue(
            use_local_acc, template_buffer_has_other_users, epilogue_creators
        ):
            # The GEMM output buffer is a global buffer, thus copy is not needed.
            if not use_local_acc:
                return False

            # The possible value of template_buffer_has_other_users is (None, False, True)
            # It is None when generating the gemm template during autotune and it will have value during scheduler codegen.
            # extra copy_from_local_to_global_buffer_epilogue is not needed in either of the below two cases:
            #   1. template_buffer_has_other_users is None (i.e. when doing the codegen during autotune)
            #   2. template_buffer_has_other_users is False, which means it's safe to keep the value in the
            #       GEMM output buffer in local buffer only (no users outside of the epilogues will use its value).
            if not template_buffer_has_other_users:
                return False

            # When bias is not None or self.epilogue_creator is not None,
            # there will be epilogue_creators after the GEMM.
````
- **EN**: Introduces function `need_copy_from_local_to_global_buffer_epilogue`. Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`need_copy_from_local_to_global_buffer_epilogue`。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 1457-1484 / 第 1457-1484 行
````python
            # The GEMM output buffer is localized while
            # the output buffer of the epilogue_creators is a global buffer.
            if epilogue_creators:
                return False

            return True

        if need_copy_from_local_to_global_buffer_epilogue(
            use_local_acc, template_buffer_has_other_users, epilogue_creators
        ):

            def copy_from_local_to_global_buffer_epilogue(input_buffer: ir.Buffer):
                dtype = self.layout.dtype
                input_loader = input_buffer.make_loader()

                def copy_inner(index):
                    input = input_loader(index)
                    result = ops.to_dtype(input, dtype)
                    return result

                return ir.Pointwise(
                    device=input_buffer.get_device_or_error(),
                    dtype=self.layout.dtype,
                    inner_fn=copy_inner,
                    ranges=input_buffer.get_size(),
                )

            epilogue_creators.append(copy_from_local_to_global_buffer_epilogue)
````
- **EN**: Introduces function `copy_from_local_to_global_buffer_epilogue`, function `copy_inner`. Handles device-specific behavior, capabilities, or dispatch decisions. Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了函数`copy_from_local_to_global_buffer_epilogue`、函数`copy_inner`。处理与设备相关的行为、能力信息或派发决策。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 1485-1512 / 第 1485-1512 行
````python

        # NOTE [How CPP GEMM template epilogues are organized]
        #   gemm_output_buffer
        #     --> zero or more in-template epilogues (created by `epilogue_creators`) -->
        #   template_buffer
        #     --> zero or more out-of-template epilogues (`epilogue_nodes`) -->
        #   Y
        if epilogue_creators:
            assert isinstance(template_buffer, ir.IRNode)
            gemm_output_name = f"{template_buffer.get_name()}_GemmOut"
            gemm_output_buffer = ir.Buffer(
                name=gemm_output_name,
                # pyrefly: ignore [missing-attribute]
                layout=template_buffer.layout,
            )
            current_input_buffer = gemm_output_buffer
            for i, creator in enumerate(epilogue_creators):
                if i == len(epilogue_creators) - 1:
                    buffer_name = template_buffer.get_name()
                else:
                    buffer_name = f"{gemm_output_name}_epilogue_{i}"
                epilogues.append(
                    ir.ComputedBuffer(
                        name=buffer_name,
                        # pyrefly: ignore [missing-attribute]
                        layout=template_buffer.layout,
                        data=creator(current_input_buffer),
                    )
````
- **EN**: Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `gemm_output_name`, `gemm_output_buffer`, `name`, `layout`, `current_input_buffer`, `buffer_name`, and `...+2`. This range continues the implementation of function `CppGemmTemplate.get_options`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `gemm_output_name`、`gemm_output_buffer`、`name`、`layout`、`current_input_buffer`、`buffer_name`、`另有2项` 等值。这一段延续了函数`CppGemmTemplate.get_options` 的具体实现。

### Lines 1513-1540 / 第 1513-1540 行
````python
                )
                fake_buffers.append(current_input_buffer)
                Y_aliases.add(current_input_buffer.get_name())
                reindexers.append(None)
                if i < len(epilogue_creators) - 1:
                    current_input_buffer = ir.Buffer(
                        name=buffer_name,
                        # pyrefly: ignore [missing-attribute]
                        layout=template_buffer.layout,
                    )

        assert isinstance(Y, (ir.Buffer, ir.ReinterpretView))
        Y_2d: ir.Buffer | ir.ReinterpretView = Y

        if epilogue_nodes:
            if not template_buffer_has_other_users:
                assert isinstance(template_buffer, ir.IRNode)
                Y_aliases.add(template_buffer.get_name())
            epilogues.extend(epilogue_nodes)
            assert Y.get_numel() == epilogues[-1].get_numel()
            Y = cast(ir.Buffer, epilogues[-1])
            assert isinstance(template_buffer, ir.Buffer)
            Y_2d, reindexers = gen_2d_view_of_epilogue_buf(
                Y,
                template_buffer,
                epilogue_nodes,
                reindexers,
                default_reindexers=self.get_default_reindexers(epilogue_nodes),
````
- **EN**: Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `current_input_buffer`, `name`, `layout`, `Y_2d`, `Y`, and `default_reindexers`. This range continues the implementation of function `CppGemmTemplate.get_options`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `current_input_buffer`、`name`、`layout`、`Y_2d`、`Y`、`default_reindexers` 等值。这一段延续了函数`CppGemmTemplate.get_options` 的具体实现。

### Lines 1541-1568 / 第 1541-1568 行
````python
            )

        output_dtype, compute_dtype = get_gemm_template_output_and_compute_dtype(
            X.get_dtype()
        )
        micro_gemm = create_micro_gemm(
            f"{kernel.kernel_name}_micro_gemm",
            self.m,
            self.n,
            self.k,
            input_dtype=X.get_dtype(),
            input2_dtype=W.get_dtype(),
            output_dtype=output_dtype,
            compute_dtype=compute_dtype,
            alpha=self.alpha,
            num_threads=self.num_threads,
            use_ref=not self.is_woq_int4(),
            q_group_size=self.q_group_size(),
        )
        assert micro_gemm is not None
        micro_gemm.use_local_vnni_blocking(not self.should_block_weights)
        assert self.register_blocking == micro_gemm.register_blocking
        self.log_blockings()
        if isinstance(micro_gemm, CppMicroGemmAMX):
            counters["inductor"]["cpp_micro_gemm_amx_counter"] += 1
        if isinstance(micro_gemm, CppMicroBrgemm):
            counters["inductor"]["cpp_micro_brgemm_counter"] += 1

````
- **EN**: Builds or updates a registry/mapping so later code can dispatch by name or capability. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `micro_gemm`, `input_dtype`, `input2_dtype`, `output_dtype`, `compute_dtype`, `alpha`, and `...+3`.
- **CN**: 构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `micro_gemm`、`input_dtype`、`input2_dtype`、`output_dtype`、`compute_dtype`、`alpha`、`另有3项` 等值。

### Lines 1569-1596 / 第 1569-1596 行
````python
        L1_cache_size = torch.cpu.get_capabilities().get(
            "l1d_cache_size", 0
        )  # per core cache size in Bytes
        assert L1_cache_size > 0, f"Expect L1_cache_size > 0 but got {L1_cache_size}"

        L2_cache_size = torch.cpu.get_capabilities().get(
            "l2_cache_size", 0
        )  # per core cache size in Bytes
        assert L2_cache_size > 0, f"Expect L2_cache_size > 0 but got {L2_cache_size}"

        options = dict(
            X=X,
            W=W,
            inp=inp,
            Y=Y,
            N=self.n,
            K=self.k,
            PADDED_N=self.padded_n,
            GemmOut=gemm_output_buffer,
            aliases={alias: Y.get_name() for alias in Y_aliases},
            beta=self.beta,
            alpha=self.alpha,
            num_threads=self.num_threads,
            micro_gemm=micro_gemm,
            is_dynamic_M=self.is_dynamic_M,
            template=self,
            kernel=kernel,
            export_declaration=get_export_declaration(),
````
- **EN**: Manipulates cached state or cache paths to avoid rebuilding the same work repeatedly. Handles device-specific behavior, capabilities, or dispatch decisions. Includes returns or checks that define the contract of this code path. Initializes or updates values such as `L1_cache_size`, `L2_cache_size`, `options`, `X`, `W`, `inp`, and `...+14`.
- **CN**: 这一段处理缓存状态或缓存路径，以避免重复构建相同工作。处理与设备相关的行为、能力信息或派发决策。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。初始化或更新了 `L1_cache_size`、`L2_cache_size`、`options`、`X`、`W`、`inp`、`另有14项` 等值。

### Lines 1597-1624 / 第 1597-1624 行
````python
            epilogue_nodes=epilogues,
            reindexers=reindexers,
            Y_2d=Y_2d,
            use_local_acc=use_local_acc,
            maybe_k_slicing=self.maybe_k_slicing(),
            x_scale=x_scale,
            x_zp=x_zp,
            w_scale=w_scale,
            w_zp=w_zp,
            acc_buf_dtype=torch.int32 if int8_gemm else torch.float,
            DTYPE_TO_CPP=DTYPE_TO_CPP,
            L1_cache_size=L1_cache_size,
            L2_cache_size=L2_cache_size,
            config=config,
            fake_buffers=fake_buffers,
            is_woq_int4=self.is_woq_int4(),
            q_group_size=q_group_size_node,
            qscale_and_zeros=qscale_and_zeros,
            cpu_count=os.cpu_count(),
        )
        return options

    def is_int8_woq_gemm_small_m_dim(
        self,
        X: ir.ReinterpretView,
        W: ir.ReinterpretView,
        N,
        K,
````
- **EN**: Introduces function `is_int8_woq_gemm_small_m_dim`. Manipulates cached state or cache paths to avoid rebuilding the same work repeatedly. Includes returns or checks that define the contract of this code path. Initializes or updates values such as `epilogue_nodes`, `reindexers`, `Y_2d`, `use_local_acc`, `maybe_k_slicing`, `x_scale`, and `...+15`.
- **CN**: 这里定义了函数`is_int8_woq_gemm_small_m_dim`。这一段处理缓存状态或缓存路径，以避免重复构建相同工作。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。初始化或更新了 `epilogue_nodes`、`reindexers`、`Y_2d`、`use_local_acc`、`maybe_k_slicing`、`x_scale`、`另有15项` 等值。

### Lines 1625-1652 / 第 1625-1652 行
````python
        micro_gemm,
    ):
        """Use SMALL_M_GEMM_TEMPLATE"""
        return (
            isinstance(micro_gemm, CppMicroGemmFP32Vec)
            and is_int8_woq_gemm_small_m_dim_corner_case(
                micro_gemm, X.get_size()[0], N, K
            )
            and X.get_dtype() is torch.bfloat16
            and W.get_dtype() is torch.int8
        )

    def render(  # type: ignore[override, return]
        self,
        kernel: CppTemplateKernel,
        template_buffer_node: ir.CppTemplateBuffer | None = None,
        flag_template_buffer_has_other_users: bool | None = None,
        epilogue_nodes: list[ir.IRNode] | None = None,
        **kwargs,
    ) -> str:
        options = self.get_options(
            kernel=kernel,
            template_buffer_node=template_buffer_node,
            flag_template_buffer_has_other_users=flag_template_buffer_has_other_users,
            epilogue_nodes=epilogue_nodes,
        )
        self.render_options = options

````
- **EN**: Introduces function `render`. Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path. Initializes or updates values such as `kernel`, `template_buffer_node`, `flag_template_buffer_has_other_users`, `epilogue_nodes`, and `options`.
- **CN**: 这里定义了函数`render`。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。初始化或更新了 `kernel`、`template_buffer_node`、`flag_template_buffer_has_other_users`、`epilogue_nodes`、`options` 等值。

### Lines 1653-1680 / 第 1653-1680 行
````python
        with contextlib.ExitStack() as stack:
            for buf in options["fake_buffers"]:
                stack.enter_context(
                    patch.object(V.graph, "get_dtype", self._fake_get_dtype(buf))
                )
            if not options["is_dynamic_M"] and self.is_int8_woq_gemm_small_m_dim(
                options["X"],
                options["W"],
                options["N"],
                options["K"],
                options["micro_gemm"],
            ):
                template_str = SMALL_M_GEMM_TEMPLATE
            else:
                template_str = GEMM_TEMPLATE
            return self._template_from_string(template_str).render(**options)

    def codegen_blocks(
        self,
        num_threads,
        N,
        K,
        micro_gemm,
        is_dynamic_M,
        kernel,
        GemmOut,
        config,
        L1_cache_size,
````
- **EN**: Introduces function `codegen_blocks`. Manipulates cached state or cache paths to avoid rebuilding the same work repeatedly. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`codegen_blocks`。这一段处理缓存状态或缓存路径，以避免重复构建相同工作。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 1681-1708 / 第 1681-1708 行
````python
        L2_cache_size,
        X,
        W,
    ):
        options = dict(
            num_threads=num_threads,
            N=N,
            K=K,
            micro_gemm=micro_gemm,
            is_dynamic_M=is_dynamic_M,
            kernel=kernel,
            GemmOut=GemmOut,
            config=config,
            L1_cache_size=L1_cache_size,
            L2_cache_size=L2_cache_size,
            template=self,
            X=X,
            W=W,
            is_woq_int4=self.is_woq_int4(),
        )
        template_str = GEMM_TEMPLATE_INIT_BLOCKING_BASIC_BLOCK
        if not (
            not is_dynamic_M
            and self.is_int8_woq_gemm_small_m_dim(X, W, N, K, micro_gemm)
        ):
            template_str += GEMM_TEMPLATE_INIT_BLOCKING_EXTENDED
        return self._template_from_string(template_str).render(options)

````
- **EN**: Manipulates cached state or cache paths to avoid rebuilding the same work repeatedly. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `options`, `num_threads`, `N`, `K`, `micro_gemm`, `is_dynamic_M`, and `...+10`.
- **CN**: 这一段处理缓存状态或缓存路径，以避免重复构建相同工作。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `options`、`num_threads`、`N`、`K`、`micro_gemm`、`is_dynamic_M`、`另有10项` 等值。

### Lines 1709-1736 / 第 1709-1736 行
````python
    def codegen_microkernel_def(self):
        return self._template_from_string(GEMM_TEMPLATE_MICROKERNEL_DEF).render(
            self.render_options
        )

    def codegen_gemm_stub_def(self):
        microkernel = self.codegen_microkernel_def()
        return microkernel + self._template_from_string(GEMM_TEMPLATE_STUB_DEF).render(
            self.render_options
        )

    def codegen_multi_threads_params(self):
        return self._template_from_string(GEMM_TEMPLATE_MULTI_THREADS_PARAMS).render()

    def codegen_single_thread_params(self, is_dynamic_M):
        options = dict(
            is_dynamic_M=is_dynamic_M,
        )
        return self._template_from_string(GEMM_TEMPLATE_SINGLE_THREAD_PARAMS).render(
            options
        )

    def codegen_m_loop_params(self):
        return self._template_from_string(GEMM_TEMPLATE_M_LOOP_PARAMS).render()

    def codegen_n_loop_params(self):
        return self._template_from_string(GEMM_TEMPLATE_N_LOOP_PARAMS).render()

````
- **EN**: Introduces function `codegen_microkernel_def`, function `codegen_gemm_stub_def`, function `codegen_multi_threads_params`, function `codegen_single_thread_params`, function `codegen_m_loop_params`, function `codegen_n_loop_params`. Includes returns or checks that define the contract of this code path. Initializes or updates values such as `microkernel`, `options`, and `is_dynamic_M`.
- **CN**: 这里定义了函数`codegen_microkernel_def`、函数`codegen_gemm_stub_def`、函数`codegen_multi_threads_params`、函数`codegen_single_thread_params`、函数`codegen_m_loop_params`、函数`codegen_n_loop_params`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。初始化或更新了 `microkernel`、`options`、`is_dynamic_M` 等值。

### Lines 1737-1764 / 第 1737-1764 行
````python
    @classmethod
    def is_woq_int4(cls):
        return False

    @classmethod
    def q_group_size(cls):
        return None


class CppWoqInt4GemmTemplateMeta(type):
    def __getitem__(cls, q_group_size):
        class CppWoqInt4GemmTemplateInstance(CppGemmTemplate):
            def __init__(
                self,
                *args,
                **kwargs,
            ) -> None:
                super().__init__(
                    *args,
                    **kwargs,
                )

            @classmethod
            def is_woq_int4(cls):
                return True

            @classmethod
            def q_group_size(cls):
````
- **EN**: Introduces function `is_woq_int4`, function `q_group_size`, class `CppWoqInt4GemmTemplateMeta`, function `__getitem__`, class `CppWoqInt4GemmTemplateInstance`, function `__init__`, function `is_woq_int4`, function `q_group_size`. Applies decorators to register behavior or alter how the following definition is constructed. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了函数`is_woq_int4`、函数`q_group_size`、类`CppWoqInt4GemmTemplateMeta`、函数`__getitem__`、类`CppWoqInt4GemmTemplateInstance`、函数`__init__`、函数`is_woq_int4`、函数`q_group_size`。使用装饰器来注册行为，或改变后续定义的构造方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 1765-1792 / 第 1765-1792 行
````python
                return q_group_size

            @staticmethod
            def check_if_block_weight(W, micro_gemm):
                # For WOQ INT4, weight is already packed
                # However, for AMX microkernel, we want to change the blocking of weight
                from .cpp_micro_gemm import CppMicroGemmWoQInt4Amx

                return isinstance(micro_gemm, CppMicroGemmWoQInt4Amx)

            @classmethod
            def block_weight(cls, W, new_size, padding):
                # This method is called only if AMX microkernels are used.
                # In this case, we unpack and repack weight so that block_n=32
                # the format of packed weight is described here:
                # https://github.com/pytorch/pytorch/blob/32eee8ed225d9f10fbbcb38c24b8b44c24c0c97c/aten/src/ATen/native/cpu/int4mm_kernel.cpp#L583
                if isinstance(W, ir.IRNode):
                    # in this case, we do nothing
                    ir.ExternKernel.require_contiguous(W)
                    blocked_w = W
                else:
                    # in this case, we unpack and repack weight
                    assert isinstance(W, torch.Tensor)
                    assert W.dim() == 2
                    N = W.size(0)
                    K = W.size(-1) * 2
                    G = cls.q_group_size()
                    # x and qscales_and_zeros are in bfloat16 instead of float to use the optimized kernel
````
- **EN**: Imports dependencies such as `.cpp_micro_gemm` for the logic in this range. Introduces function `check_if_block_weight`, function `block_weight`. Preserves reference comments that point to design notes, specifications, or vendor datasheets. Applies decorators to register behavior or alter how the following definition is constructed.
- **CN**: 这里导入了 `.cpp_micro_gemm` 等依赖，为后续逻辑提供基础能力。这里定义了函数`check_if_block_weight`、函数`block_weight`。保留了指向设计说明、规范或厂商数据手册的注释引用。使用装饰器来注册行为，或改变后续定义的构造方式。

### Lines 1793-1820 / 第 1793-1820 行
````python
                    # so that the unpacking process is faster
                    x = torch.eye(K).bfloat16()
                    # Here we use scale=1 and qzero=8 because we want to unpack weight
                    # without dequantizing it. The qzero here is 8 instead of 0 because
                    # int4 values are converted to [-7, 8] in the _weight_int4pack_mm_for_cpu kernel:
                    # https://github.com/pytorch/pytorch/blob/32eee8ed225d9f10fbbcb38c24b8b44c24c0c97c/aten/src/ATen/native/cpu/int4mm_kernel.cpp#L95
                    qscales_and_zeros = (
                        torch.tensor([1.0, 8.0])
                        .bfloat16()
                        .expand(K // G, N, 2)
                        .contiguous()
                    )
                    # shape: [K, N]
                    unpacked_w = torch.ops.aten._weight_int4pack_mm_for_cpu(
                        x,
                        W,
                        G,
                        qscales_and_zeros,
                    ).to(torch.uint8)
                    block_n = 32
                    # shape: [N // block_n, K, block_n]
                    w_blocked = (
                        unpacked_w.view(K, N // block_n, block_n)
                        .permute(1, 0, 2)
                        .contiguous()
                    )
                    # pack 2 int4 -> 1 int8
                    # block_n: [a0, a1, ..., a15, b0, b1, ..., b15]
````
- **EN**: Preserves reference comments that point to design notes, specifications, or vendor datasheets. Handles device-specific behavior, capabilities, or dispatch decisions. Tracks dtype-specific behavior or constants that affect numeric execution. Initializes or updates values such as `x`, `qscales_and_zeros`, `unpacked_w`, `block_n`, and `w_blocked`.
- **CN**: 保留了指向设计说明、规范或厂商数据手册的注释引用。处理与设备相关的行为、能力信息或派发决策。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。初始化或更新了 `x`、`qscales_and_zeros`、`unpacked_w`、`block_n`、`w_blocked` 等值。

### Lines 1821-1837 / 第 1821-1837 行
````python
                    # -> [(a0 & 0xf) | (b0 << 4), (a1 & 0xf) | (b1 << 4), ...]
                    # shape: [N // block_n, K, 2, block_n // 2]
                    w_blocked = w_blocked.view(N // block_n, K, 2, block_n // 2)
                    # shape: [N // block_n, K, block_n // 2]
                    w_blocked_packed = (w_blocked[:, :, 0, :] & 0xF) | (
                        w_blocked[:, :, 1, :] << 4
                    )
                    # shape: [N, K // 2]
                    blocked_w = w_blocked_packed.view(N, K // 2)

                return blocked_w

        return CppWoqInt4GemmTemplateInstance


class CppWoqInt4GemmTemplate(metaclass=CppWoqInt4GemmTemplateMeta):
    pass
````
- **EN**: Introduces class `CppWoqInt4GemmTemplate`. Includes returns or checks that define the contract of this code path. Initializes or updates values such as `w_blocked`, `w_blocked_packed`, and `blocked_w`.
- **CN**: 这里定义了类`CppWoqInt4GemmTemplate`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。初始化或更新了 `w_blocked`、`w_blocked_packed`、`blocked_w` 等值。

## Key Concepts / 关键概念
- **EN**: Generates backend-specific code for lowered kernels  
  **CN**: 为下沉后的内核生成后端相关代码
- **EN**: Compilation/runtime caching  
  **CN**: 编译或运行期缓存
- **EN**: Graph-level transformations  
  **CN**: 图级变换
- **EN**: Scheduling decisions  
  **CN**: 调度决策
- **EN**: Operator lowering  
  **CN**: 算子下沉
- **EN**: Profiling data handling  
  **CN**: 性能画像数据处理
- **EN**: Device capability modeling  
  **CN**: 设备能力建模
- **EN**: Autotuning support  
  **CN**: 自动调优支持

## Dependencies / 依赖关系
- **Standard library / 标准库**: `contextlib`, `logging`, `math`, `os`, `collections.abc`, `functools`, `typing`, `unittest.mock`
- **Third-party / 第三方**: None / 无
- **PyTorch/Internal / PyTorch 内部**: `torch`, `torch.utils`, `torch.utils._ordered_set`, `..._dynamo.utils`, `..`, `..kernel.mm_common`, `..select_algorithm`, `..utils`, `..virtualized`, `.cpp`, `.cpp_micro_gemm`, `.cpp_template`, `.cpp_template_kernel`, `.cpp_utils`
