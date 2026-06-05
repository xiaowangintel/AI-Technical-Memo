# cpp_grouped_gemm_template.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `torch/_inductor/codegen/cpp_grouped_gemm_template.py`
- **Repository / 仓库**: `pytorch/pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: This module generates backend-specific code for lowered kernels. It defines classes such as `CppGroupedGemmTemplate`. It exposes functions such as `get_deduplicated_act`.
- **用途（中文）**: 该模块为下沉后的内核生成后端相关代码。其中定义了 `CppGroupedGemmTemplate` 等类。同时提供 `get_deduplicated_act` 等函数。

## Line-by-Line Analysis / 逐行分析
### Lines 1-20 / 第 1-20 行
````python
import contextlib
import logging
import os
from collections.abc import Callable
from typing import Any, cast, TypeVar
from unittest.mock import patch

import torch
import torch.utils
from torch.utils._ordered_set import OrderedSet

from ..._dynamo.utils import counters
from .. import config, ir
from ..kernel.mm_common import mm_args
from ..select_algorithm import ChoiceCaller, DataProcessorTemplateWrapper
from ..utils import parallel_num_threads
from ..virtualized import V
from .cpp import get_export_declaration
from .cpp_gemm_template import (
    CppGemmTemplate,
````
- **EN**: Imports dependencies such as `contextlib`, `logging`, `os`, `collections.abc`, `typing`, `unittest.mock`, and `...+11` for the logic in this range.
- **CN**: 这里导入了 `contextlib`、`logging`、`os`、`collections.abc`、`typing`、`unittest.mock`、`另有11项` 等依赖，为后续逻辑提供基础能力。

### Lines 21-40 / 第 21-40 行
````python
    expand_bias,
    gen_2d_view_of_epilogue_buf,
    prune_tensors,
    transpose_w,
)
from .cpp_micro_gemm import CppMicroGemmAMX, create_micro_gemm
from .cpp_template_kernel import CppTemplateKernel
from .cpp_utils import (
    create_epilogue_with_attr,
    DTYPE_TO_CPP,
    GemmBlocking,
    get_gemm_template_output_and_compute_dtype,
)


log = logging.getLogger(__name__)

GEMM_TEMPLATE = r"""
{{template.header().getvalue()}}
{{micro_gemm.codegen_define(kernel)}}
````
- **EN**: Imports dependencies such as `.cpp_micro_gemm`, `.cpp_template_kernel`, and `.cpp_utils` for the logic in this range. Initializes or updates values such as `log`, and `GEMM_TEMPLATE`.
- **CN**: 这里导入了 `.cpp_micro_gemm`、`.cpp_template_kernel`、`.cpp_utils` 等依赖，为后续逻辑提供基础能力。初始化或更新了 `log`、`GEMM_TEMPLATE` 等值。

### Lines 41-60 / 第 41-60 行
````python

extern "C" {{export_declaration}}
{{kernel.def_kernel(inputs=kernel_args, outputs=Y_list, aliases=aliases)}}
{
    {{kernel.maybe_codegen_profile()}}
    {{ template.codegen_blocks(
        num_threads, N, K, micro_gemm, is_dynamic_M, kernel, GemmOuts[0], config, L1_cache_size, L2_cache_size, X_list[0], W_list[0]
    ) }}
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
````
- **EN**: Manipulates cached state or cache paths to avoid rebuilding the same work repeatedly.
- **CN**: 这一段处理缓存状态或缓存路径，以避免重复构建相同工作。

### Lines 61-80 / 第 61-80 行
````python
{%- endif %}
        {{ micro_gemm.codegen_init(kernel) }}
{%- set acc_buf_name_list=[] %}
{%- set acc_buf_name_prefix = "local_acc_buf_" %}
{%- for gemm_idx in range(0, gemm_grouped_num, 1) %}
    {%- set acc_buf_name = acc_buf_name_prefix + gemm_idx|string %}
    {{ kernel.define_buffer(acc_buf_name, ["Mc_blocks*Mr", "Nc_blocks*Nr"], acc_buf_dtype) }}
    {%- set acc_buf_name_list=acc_buf_name_list.append(acc_buf_name) %}
{%- endfor %}
        for (int64_t mc_block_id = 0; mc_block_id < num_Mc_blocks_per_thread; mc_block_id++) {
            {{ template.codegen_m_loop_params()|indent(12, false) }}
            for (int64_t nc = n_block_start; nc < n_block_end; nc += Nc_blocks) {
                {{ template.codegen_n_loop_params()|indent(16, false) }}
{%- set acc_list=[] %}
{%- for gemm_idx in range(0, gemm_grouped_num, 1) %}
    {%- set acc_list = acc_list.append( kernel.local_buffers[acc_buf_name_list[gemm_idx]] ) %}
    {{ kernel.reinit_buffer_if_null(acc_buf_name_list[gemm_idx]) }}
{%- endfor %}
                for (int64_t kc = k_block_start; kc < k_block_end; kc += Kc_blocks) {
                    int64_t k_start = kc * Kr;
````
- **EN**: Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 包含分支、循环或上下文管理等控制流。

### Lines 81-100 / 第 81-100 行
````python
                    int64_t k_end = std::min(std::min(kc + Kc_blocks, k_block_end) * Kr, K);
{%- set tile_X_list=[] %}
{%- for gemm_idx in range(0, gemm_grouped_num, 1) %}
    {%- set tile_X_list = tile_X_list.append( kernel.slice_nd(X_list[gemm_idx], [("m_start", "m_end"), ("k_start", "k_end")]) ) %}
{%- endfor %}
                    for (int64_t nci = nc; nci < nc_block_end; nci++) {
{%- set tile_W_3d_list=[] %}
{%- set tile_W_list=[] %}
{%- set acc_slice_list=[] %}
{%- for gemm_idx in range(0, gemm_grouped_num, 1) %}
    {%- set acc_slice_list = acc_slice_list.append(
        kernel.slice_nd(acc_list[gemm_idx], [("0", "m_end - m_start"), ("(nci - nc)*Nr", "(nci - nc + 1)*Nr")])
    ) %}
    {%- set tile_W_3d_list = tile_W_3d_list.append(
        kernel.slice_nd(W_list[gemm_idx], [("nci", "nci + 1"), ("k_start", "k_end"), ()])
    ) %}
{%- endfor %}
{%- for gemm_idx in range(0, gemm_grouped_num, 1) %}
    {%- set tile_W_list = tile_W_list.append(
        kernel.view(tile_W_3d_list[gemm_idx], ["k_end - k_start", micro_gemm.register_blocking.block_n])
````
- **EN**: Builds or updates a registry/mapping so later code can dispatch by name or capability. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。包含分支、循环或上下文管理等控制流。

### Lines 101-120 / 第 101-120 行
````python
    ) %}
{%- endfor %}
                        if (kc == k_block_start) {
                            {%- for gemm_idx in range(0, gemm_grouped_num, 1) %}
                                {{ micro_gemm.codegen_call(
                                    kernel, tile_X_list[gemm_idx], tile_W_list[gemm_idx], acc_slice_list[gemm_idx], accum=False
                                )|indent(28, false) }}
                            {%- endfor %}
                        } else {
                            {%- for gemm_idx in range(0, gemm_grouped_num, 1) %}
                                {{ micro_gemm.codegen_call(
                                    kernel, tile_X_list[gemm_idx], tile_W_list[gemm_idx], acc_slice_list[gemm_idx], accum=True
                                )|indent(28, false) }}
                            {%- endfor %}
                        }
                    }
                }
                {
{%- set tile_acc_list = [] %}
{%- set tile_Y_list = [] %}
````
- **EN**: Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 包含分支、循环或上下文管理等控制流。

### Lines 121-140 / 第 121-140 行
````python
{%- for gemm_idx in range(0, gemm_grouped_num, 1) %}
    {%- set tile_acc_list = tile_acc_list.append(
        kernel.slice_nd(acc_list[gemm_idx], [("0", "m_end - m_start"), ("0", "n_end - n_start")])
    ) %}
    {%- set tile_Y_list = tile_Y_list.append(
        kernel.slice_nd(Y_2d_list[gemm_idx], [("m_start", "m_end"), ("n_start", "n_end")])
    ) %}
{%- endfor %}
                    {{ kernel.store_outputs(
                        tile_Y_list,
                        tile_acc_list,
                        GemmOuts,
                        epilogue_nodes,
                        offsets=("m_start", "n_start"),
                        reindexers=reindexers,
                        multi_output_buffers=multi_output_buffers
                    )|indent(20, false)
                    }}
                }
            }
````
- **EN**: Initializes or updates values such as `offsets`, `reindexers`, and `multi_output_buffers`.
- **CN**: 初始化或更新了 `offsets`、`reindexers`、`multi_output_buffers` 等值。

### Lines 141-160 / 第 141-160 行
````python
        }
        {{ micro_gemm.codegen_finalize(kernel) }}
    }
}
"""


def get_deduplicated_act(act_mapping: dict[int, ir.IRNode]) -> list[ir.IRNode]:
    act_deduplicated = []
    act_deduplicated_name: OrderedSet[str] = OrderedSet()
    for act_idx in range(len(act_mapping.values())):
        act = act_mapping[act_idx]
        if act.get_name() not in act_deduplicated_name:
            act_deduplicated.append(act)
            act_deduplicated_name.add(act.get_name())
    return act_deduplicated


class CppGroupedGemmTemplate(CppGemmTemplate):
    def __init__(
````
- **EN**: Introduces function `get_deduplicated_act`, class `CppGroupedGemmTemplate`, function `__init__`. Builds or updates a registry/mapping so later code can dispatch by name or capability. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`get_deduplicated_act`、类`CppGroupedGemmTemplate`、函数`__init__`。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 161-180 / 第 161-180 行
````python
        self,
        input_nodes: list[ir.IRNode],
        layout: ir.Layout,
        num_threads: int,
        register_blocking: GemmBlocking,
        beta: int = 1,
        alpha: int = 1,
        has_bias: bool = False,
        epilogue_creator: Callable[[ir.Buffer], ir.Pointwise] | None = None,
        act_mapping: dict[int, ir.IRNode] | None = None,
        gemm_grouped_num: int = 1,
    ) -> None:
        """
        Template for Group of GEMMs:
        * Each GEMM has the same dimensions (m, n, k) and the same leading dimensions (lda, ldb, ldc)
          for their A, B, and C matrices.
        * Each GEMM has distinct or shared activations, has distinct weight, has unique bias or no bias, has distinct epilogues.
        * In the current implementation, the outputs of all GEMMs are accumulated using pointwise epilogues.
          This behavior can be extended in the future if needed.
        """
````
- **EN**: Builds or updates a registry/mapping so later code can dispatch by name or capability. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `input_nodes`, `layout`, `num_threads`, `register_blocking`, `beta`, `alpha`, and `...+4`. This range continues the implementation of function `CppGroupedGemmTemplate.__init__`.
- **CN**: 构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。包含分支、循环或上下文管理等控制流。初始化或更新了 `input_nodes`、`layout`、`num_threads`、`register_blocking`、`beta`、`alpha`、`另有4项` 等值。这一段延续了函数`CppGroupedGemmTemplate.__init__` 的具体实现。

### Lines 181-200 / 第 181-200 行
````python
        super().__init__(
            input_nodes,
            layout,
            num_threads,
            register_blocking,
            beta,
            alpha,
            has_bias,
            epilogue_creator,
        )
        self.act_mapping = act_mapping
        self.gemm_grouped_num = gemm_grouped_num
        # pyrefly: ignore [bad-override]
        self.output_node: list[ir.Buffer] = [
            ir.Buffer(name="buf_out" + str(idx), layout=layout)
            for idx in range(gemm_grouped_num)
        ]

    @classmethod
    # pyrefly: ignore [bad-override]
````
- **EN**: Applies decorators to register behavior or alter how the following definition is constructed. Builds or updates a registry/mapping so later code can dispatch by name or capability. Contains control-flow that branches, iterates, or scopes resource management. This range continues the implementation of function `CppGroupedGemmTemplate.__init__`.
- **CN**: 使用装饰器来注册行为，或改变后续定义的构造方式。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。包含分支、循环或上下文管理等控制流。这一段延续了函数`CppGroupedGemmTemplate.__init__` 的具体实现。

### Lines 201-220 / 第 201-220 行
````python
    def add_choices(
        cls,
        choices: list[ChoiceCaller],
        layout: ir.Layout,
        input_nodes: list[ir.IRNode],
        beta: int = 1,
        alpha: int = 1,
        has_bias: tuple[bool, ...] = (False, False),
        trans_w: bool = False,
        input_indices: list[int] | None = None,
        epilogue_creator: Callable[[ir.Buffer], ir.Pointwise] | None = None,
        act_mapping: dict[int, ir.IRNode] | None = None,  # gemm idx to its act buf
    ) -> DataProcessorTemplateWrapper:
        # Input nodes order: x, optional[x1], ... w0, w1, ... optional[b0], optional[b1], ...
        gemm_grouped_num = len(has_bias)
        assert act_mapping
        act_deduplicated = get_deduplicated_act(act_mapping)
        wgt_start_idx = len(act_deduplicated)
        bias_start_idx = wgt_start_idx + gemm_grouped_num
        input_indices = list(range(len(input_nodes)))
````
- **EN**: Introduces function `add_choices`. Builds or updates a registry/mapping so later code can dispatch by name or capability. Includes returns or checks that define the contract of this code path. Initializes or updates values such as `choices`, `layout`, `input_nodes`, `beta`, `alpha`, `has_bias`, and `...+8`.
- **CN**: 这里定义了函数`add_choices`。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。初始化或更新了 `choices`、`layout`、`input_nodes`、`beta`、`alpha`、`has_bias`、`另有8项` 等值。

### Lines 221-240 / 第 221-240 行
````python

        _T = TypeVar("_T", ir.IRNode, torch.Tensor)
        _U = TypeVar("_U", ir.Layout, torch.Tensor)

        def reorder_and_filter(
            inputs: list[_T],
            layout_or_out: _U,
        ) -> tuple[list[_T], _U]:
            assert input_indices is not None, "input_indices must be set"
            return [inputs[idx] for idx in input_indices], layout_or_out

        new_inputs, new_layout = reorder_and_filter(input_nodes, layout)

        def maybe_to_dense(
            inputs: list[_T],
            layout_or_out: _U,
        ) -> tuple[list[_T], _U]:
            new_inputs = list(inputs)
            for idx in range(wgt_start_idx, wgt_start_idx + gemm_grouped_num):
                if isinstance(inputs[idx], torch.Tensor):
````
- **EN**: Introduces function `reorder_and_filter`, function `maybe_to_dense`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `_T`, `_U`, `inputs`, `layout_or_out`, and `new_inputs`.
- **CN**: 这里定义了函数`reorder_and_filter`、函数`maybe_to_dense`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `_T`、`_U`、`inputs`、`layout_or_out`、`new_inputs` 等值。

### Lines 241-260 / 第 241-260 行
````python
                    W = inputs[idx]
                    assert isinstance(W, torch.Tensor), "W must be a torch.Tensor"
                    # pyrefly: ignore [unsupported-operation]
                    new_inputs[idx] = W.to_dense() if W.is_mkldnn else W
            return new_inputs, layout_or_out

        def normalize_shapes(
            inputs: list[_T],
            layout_or_out: _U,
        ) -> tuple[list[_T], _U]:
            new_inputs: list[_T] = list(inputs)
            if not trans_w:
                return new_inputs, layout_or_out
            X = new_inputs[0]
            for wgt_idx in range(wgt_start_idx, wgt_start_idx + gemm_grouped_num):
                new_input = new_inputs[wgt_idx]
                new_inputs[wgt_idx] = transpose_w(new_input, trans_w)
            for bias_idx in range(bias_start_idx, len(new_inputs)):
                # pyrefly: ignore [bad-argument-type]
                new_bias = expand_bias(new_inputs[bias_idx], X)
````
- **EN**: Introduces function `normalize_shapes`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `W`, `inputs`, `layout_or_out`, `new_inputs`, `X`, `new_input`, and `...+1`.
- **CN**: 这里定义了函数`normalize_shapes`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `W`、`inputs`、`layout_or_out`、`new_inputs`、`X`、`new_input`、`另有1项` 等值。

### Lines 261-280 / 第 261-280 行
````python
                assert new_bias is not None
                # pyrefly: ignore [unsupported-operation]
                new_inputs[bias_idx] = new_bias
            return new_inputs, layout_or_out

        num_threads = parallel_num_threads()
        new_inputs, _ = normalize_shapes(*maybe_to_dense(new_inputs, new_layout))
        m, n, k, *_ = mm_args(new_inputs[0], new_inputs[wgt_start_idx])
        output_dtype, compute_dtype = get_gemm_template_output_and_compute_dtype(
            new_inputs[0].get_dtype()
        )
        micro_gemm = create_micro_gemm(
            "micro_gemm",
            m,
            n,
            k,
            input_dtype=new_inputs[0].get_dtype(),
            input2_dtype=new_inputs[wgt_start_idx].get_dtype(),
            output_dtype=output_dtype,
            compute_dtype=compute_dtype,
````
- **EN**: Includes returns or checks that define the contract of this code path. Initializes or updates values such as `num_threads`, `micro_gemm`, `input_dtype`, `input2_dtype`, `output_dtype`, and `compute_dtype`. This range continues the implementation of function `CppGroupedGemmTemplate.add_choices`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。初始化或更新了 `num_threads`、`micro_gemm`、`input_dtype`、`input2_dtype`、`output_dtype`、`compute_dtype` 等值。这一段延续了函数`CppGroupedGemmTemplate.add_choices` 的具体实现。

### Lines 281-300 / 第 281-300 行
````python
            alpha=alpha,
            num_threads=num_threads,
        )
        assert micro_gemm is not None
        _, block_n, _ = micro_gemm.register_blocking
        new_size, padded_n = cls.get_padded_size(
            n, block_n, k, should_block_weight=True
        )
        padding = padded_n - n

        def pack_weight(
            inputs: list[_T],
            layout_or_out: _U,
        ) -> tuple[list[_T], _U]:
            new_W_list = []
            new_inputs = list(inputs)
            W_list = new_inputs[wgt_start_idx : wgt_start_idx + gemm_grouped_num]
            for W in W_list:
                blocked_w = cls.block_weight(W, new_size, padding)
                new_W_list.append(cls.pack_vnni_weight(blocked_w, micro_gemm, new_size))
````
- **EN**: Introduces function `pack_weight`. Builds or updates a registry/mapping so later code can dispatch by name or capability. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`pack_weight`。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 301-320 / 第 301-320 行
````python
            new_inputs[wgt_start_idx : wgt_start_idx + gemm_grouped_num] = new_W_list
            return new_inputs, layout_or_out

        def preprocessor(
            inputs: list[_T],
            layout: _U,
        ) -> tuple[list[_T], _U]:
            return pack_weight(
                *normalize_shapes(*maybe_to_dense(*reorder_and_filter(inputs, layout)))
            )

        def postprocessor(output: _T) -> _T:
            if isinstance(output, ir.TensorBox):
                template_buffer = ir.InputsKernel.unwrap_storage_for_input(output)
                assert isinstance(template_buffer, ir.CppTemplateBuffer)
                new_input_nodes, _ = reorder_and_filter(input_nodes, layout)
                W_nodes = new_input_nodes[
                    wgt_start_idx : wgt_start_idx + gemm_grouped_num
                ]
                W_tensor = []
````
- **EN**: Introduces function `preprocessor`, function `postprocessor`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `inputs`, `layout`, `template_buffer`, `W_nodes`, `wgt_start_idx`, and `W_tensor`.
- **CN**: 这里定义了函数`preprocessor`、函数`postprocessor`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `inputs`、`layout`、`template_buffer`、`W_nodes`、`wgt_start_idx`、`W_tensor` 等值。

### Lines 321-340 / 第 321-340 行
````python
                for W_node in W_nodes:
                    assert W_node.get_name() in V.graph.constants
                    # pyrefly: ignore [bad-argument-type]
                    W_tensor.append(V.graph.constants[W_node.get_name()])
                # pyrefly: ignore [unsupported-operation]
                new_input_nodes[wgt_start_idx : wgt_start_idx + gemm_grouped_num] = (
                    W_tensor  # type: ignore[assignment]
                )
                new_input_nodes, _ = pack_weight(
                    *normalize_shapes(*maybe_to_dense(new_input_nodes, layout))
                )
                # Prune unused tensors
                prune_tensors(input_nodes, new_input_nodes)
                for idx in range(wgt_start_idx, wgt_start_idx + gemm_grouped_num):
                    W_packed = new_input_nodes[idx]
                    assert isinstance(W_packed, torch.Tensor)
                    W_packed_constant = V.graph.add_tensor_constant(W_packed)
                    template_buffer.inputs[idx] = (
                        ir.InputsKernel.unwrap_storage_for_input(W_packed_constant)
                    )
````
- **EN**: Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `W_packed`, and `W_packed_constant`. This range continues the implementation of function `CppGroupedGemmTemplate.add_choices.postprocessor`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `W_packed`、`W_packed_constant` 等值。这一段延续了函数`CppGroupedGemmTemplate.add_choices.postprocessor` 的具体实现。

### Lines 341-360 / 第 341-360 行
````python

            return output

        template = DataProcessorTemplateWrapper(
            CppGroupedGemmTemplate,
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
            act_mapping=act_mapping,
            gemm_grouped_num=gemm_grouped_num,
        )
        template.maybe_append_choice(choices)
        return template
````
- **EN**: Builds or updates a registry/mapping so later code can dispatch by name or capability. Includes returns or checks that define the contract of this code path. Initializes or updates values such as `template`, `input_nodes`, `layout`, `num_threads`, `register_blocking`, `beta`, and `...+5`. This range continues the implementation of function `CppGroupedGemmTemplate.add_choices`.
- **CN**: 构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。初始化或更新了 `template`、`input_nodes`、`layout`、`num_threads`、`register_blocking`、`beta`、`另有5项` 等值。这一段延续了函数`CppGroupedGemmTemplate.add_choices` 的具体实现。

### Lines 361-380 / 第 361-380 行
````python

    def render(  # type: ignore[override,return,no-untyped-def]
        self,
        kernel: CppTemplateKernel,
        template_buffer_node: ir.CppTemplateBuffer | None = None,
        flag_template_buffer_has_other_users: bool | None = None,
        epilogue_nodes: list[ir.IRNode] | None = None,
        **kwargs,
    ) -> str:
        assert self.act_mapping
        act_deduplicated = get_deduplicated_act(self.act_mapping)
        wgt_start_idx = len(act_deduplicated)
        bias_start_idx = wgt_start_idx + self.gemm_grouped_num
        X_list = list(self.act_mapping.values())
        W_list = self.input_nodes[wgt_start_idx : wgt_start_idx + self.gemm_grouped_num]
        inp_list = []
        cur_idx = bias_start_idx
        for inp_idx in range(self.gemm_grouped_num):
            inp = None
            # pyrefly: ignore [bad-index, index-error]
````
- **EN**: Introduces function `render`. Builds or updates a registry/mapping so later code can dispatch by name or capability. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`render`。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 381-400 / 第 381-400 行
````python
            if self.has_bias[inp_idx]:
                inp = self.input_nodes[cur_idx]
                cur_idx += 1
            inp_list.append(inp)

        Y_list = self.output_node
        multi_output_buffers = None
        if template_buffer_node is not None:
            W_list = template_buffer_node.inputs[
                wgt_start_idx : wgt_start_idx + self.gemm_grouped_num
            ]
            assert isinstance(template_buffer_node.outputs, list)
            Y_list = template_buffer_node.outputs
            counters["inductor"]["cpp_grouped_gemm_template"] += 1
            multi_output_buffers = template_buffer_node.outputs

        template_buffer = Y_list[0]
        fake_buffers: list[ir.Buffer] = []
        Y_2d_list = Y_list
        output_dtype, compute_dtype = get_gemm_template_output_and_compute_dtype(
````
- **EN**: Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `inp`, `Y_list`, `multi_output_buffers`, `W_list`, `wgt_start_idx`, `template_buffer`, and `...+2`. This range continues the implementation of function `CppGroupedGemmTemplate.render`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `inp`、`Y_list`、`multi_output_buffers`、`W_list`、`wgt_start_idx`、`template_buffer`、`另有2项` 等值。这一段延续了函数`CppGroupedGemmTemplate.render` 的具体实现。

### Lines 401-420 / 第 401-420 行
````python
            X_list[0].get_dtype()
        )
        micro_gemm = create_micro_gemm(
            f"{kernel.kernel_name}_micro_gemm",
            self.m,
            self.n,
            self.k,
            input_dtype=X_list[0].get_dtype(),
            input2_dtype=W_list[0].get_dtype(),
            output_dtype=output_dtype,
            compute_dtype=compute_dtype,
            alpha=self.alpha,
            num_threads=self.num_threads,
        )
        assert micro_gemm is not None
        assert self.register_blocking == micro_gemm.register_blocking
        self.log_blockings()
        if isinstance(micro_gemm, CppMicroGemmAMX):
            counters["inductor"]["cpp_micro_gemm_amx_counter"] += 1

````
- **EN**: Builds or updates a registry/mapping so later code can dispatch by name or capability. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `micro_gemm`, `input_dtype`, `input2_dtype`, `output_dtype`, `compute_dtype`, `alpha`, and `...+1`.
- **CN**: 构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `micro_gemm`、`input_dtype`、`input2_dtype`、`output_dtype`、`compute_dtype`、`alpha`、`另有1项` 等值。

### Lines 421-440 / 第 421-440 行
````python
        L1_cache_size = torch.cpu.get_capabilities().get(
            "l1d_cache_size", 0
        )  # per core cache size in Bytes
        assert L1_cache_size > 0, f"Expect L1_cache_size > 0 but got {L1_cache_size}"

        L2_cache_size = torch.cpu.get_capabilities().get(
            "l2_cache_size", 0
        )  # per core cache size in Bytes
        assert L2_cache_size > 0, f"Expect L2_cache_size > 0 but got {L2_cache_size}"

        epilogues: list[ir.IRNode] = []
        reindexers: list[Callable[[list[Any]], list[Any]] | None] = []
        gemm_output_buffers: list[ir.Buffer] = []
        for out_buf_idx in range(self.gemm_grouped_num):
            gemm_output_name = f"{template_buffer.get_name()}_GemmOut" + str(
                out_buf_idx
            )
            gemm_output_buffers.append(
                ir.Buffer(name=gemm_output_name, layout=template_buffer.layout)
            )
````
- **EN**: Manipulates cached state or cache paths to avoid rebuilding the same work repeatedly. Handles device-specific behavior, capabilities, or dispatch decisions. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这一段处理缓存状态或缓存路径，以避免重复构建相同工作。处理与设备相关的行为、能力信息或派发决策。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 441-460 / 第 441-460 行
````python

        assert not self.epilogue_creator, (
            "epilogue_creator is not supported yet in Grouped GEMM Template"
        )

        kernel_args: dict[str, ir.IRNode | None] = {}
        for x_idx in range(wgt_start_idx):
            kernel_args["X" + str(x_idx)] = act_deduplicated[x_idx]
        for w_idx in range(self.gemm_grouped_num):
            kernel_args["W" + str(w_idx)] = W_list[w_idx]
        for inp_idx in range(self.gemm_grouped_num):
            kernel_args["inp" + str(inp_idx)] = inp_list[inp_idx]

        def _bias_add_epilogue(buf: ir.IRNode, inp: ir.IRNode) -> ir.Pointwise:
            return create_epilogue_with_attr(
                buf, "bias_add", other=inp, beta=self.beta, dtype=self.layout.dtype
            )

        for gemm_idx, inp in enumerate(inp_list):
            if inp:
````
- **EN**: Introduces function `_bias_add_epilogue`. Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`_bias_add_epilogue`。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 461-480 / 第 461-480 行
````python
                buffer_name = Y_list[gemm_idx].get_name()
                epilogues.append(
                    ir.ComputedBuffer(
                        name=buffer_name,
                        layout=template_buffer.layout,
                        data=_bias_add_epilogue(gemm_output_buffers[gemm_idx], inp),
                    )
                )
                reindexers.append(None)

        if epilogue_nodes:
            epilogues.extend(epilogue_nodes)
            for epilogue_node in epilogue_nodes:
                Y = cast(ir.Buffer, epilogue_node)
                _, reindexers = gen_2d_view_of_epilogue_buf(
                    Y,
                    template_buffer,
                    [
                        epilogue_node,
                    ],
````
- **EN**: Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `buffer_name`, `name`, `layout`, `data`, and `Y`. This range continues the implementation of function `CppGroupedGemmTemplate.render`.
- **CN**: 包含分支、循环或上下文管理等控制流。初始化或更新了 `buffer_name`、`name`、`layout`、`data`、`Y` 等值。这一段延续了函数`CppGroupedGemmTemplate.render` 的具体实现。

### Lines 481-500 / 第 481-500 行
````python
                    reindexers,
                    default_reindexers=[
                        None,
                    ],
                )

        options = dict(
            N=self.n,
            K=self.k,
            PADDED_N=self.padded_n,
            aliases={},
            beta=self.beta,
            alpha=self.alpha,
            num_threads=self.num_threads,
            micro_gemm=micro_gemm,
            is_dynamic_M=self.is_dynamic_M,
            template=self,
            kernel=kernel,
            export_declaration=get_export_declaration(),
            acc_buf_dtype=torch.float,
````
- **EN**: Initializes or updates values such as `default_reindexers`, `options`, `N`, `K`, `PADDED_N`, `aliases`, and `...+9`. This range continues the implementation of function `CppGroupedGemmTemplate.render`.
- **CN**: 初始化或更新了 `default_reindexers`、`options`、`N`、`K`、`PADDED_N`、`aliases`、`另有9项` 等值。这一段延续了函数`CppGroupedGemmTemplate.render` 的具体实现。

### Lines 501-520 / 第 501-520 行
````python
            DTYPE_TO_CPP=DTYPE_TO_CPP,
            L1_cache_size=L1_cache_size,
            L2_cache_size=L2_cache_size,
            config=config,
            epilogue_nodes=epilogues,
            GemmOuts=gemm_output_buffers,
            reindexers=reindexers,
            kernel_args=kernel_args,
            X_list=X_list,
            W_list=W_list,
            gemm_grouped_num=self.gemm_grouped_num,
            Y_list={"Y" + str(idx): Y for idx, Y in enumerate(Y_list)},
            Y_2d_list=Y_2d_list,
            multi_output_buffers=multi_output_buffers,
            cpu_count=os.cpu_count(),
        )
        with contextlib.ExitStack() as stack:
            stack.enter_context(
                patch.object(V.graph, "get_dtype", self._fake_get_dtype(fake_buffers))
            )
````
- **EN**: Manipulates cached state or cache paths to avoid rebuilding the same work repeatedly. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `DTYPE_TO_CPP`, `L1_cache_size`, `L2_cache_size`, `config`, `epilogue_nodes`, `GemmOuts`, and `...+9`. This range continues the implementation of function `CppGroupedGemmTemplate.render`.
- **CN**: 这一段处理缓存状态或缓存路径，以避免重复构建相同工作。包含分支、循环或上下文管理等控制流。初始化或更新了 `DTYPE_TO_CPP`、`L1_cache_size`、`L2_cache_size`、`config`、`epilogue_nodes`、`GemmOuts`、`另有9项` 等值。这一段延续了函数`CppGroupedGemmTemplate.render` 的具体实现。

### Lines 521-521 / 第 521-521 行
````python
            return self._template_from_string(GEMM_TEMPLATE).render(**options)
````
- **EN**: Includes returns or checks that define the contract of this code path. This range continues the implementation of function `CppGroupedGemmTemplate.render`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。这一段延续了函数`CppGroupedGemmTemplate.render` 的具体实现。

## Key Concepts / 关键概念
- **EN**: Generates backend-specific code for lowered kernels  
  **CN**: 为下沉后的内核生成后端相关代码
- **EN**: Compilation/runtime caching  
  **CN**: 编译或运行期缓存
- **EN**: Graph-level transformations  
  **CN**: 图级变换
- **EN**: Profiling data handling  
  **CN**: 性能画像数据处理
- **EN**: Kernel construction or dispatch  
  **CN**: 内核构建或派发
- **EN**: Dtype propagation or validation  
  **CN**: 数据类型传播或校验
- **EN**: Primary classes: `CppGroupedGemmTemplate`  
  **CN**: 主要类：`CppGroupedGemmTemplate`
- **EN**: Primary functions: `get_deduplicated_act`  
  **CN**: 主要函数：`get_deduplicated_act`

## Dependencies / 依赖关系
- **Standard library / 标准库**: `contextlib`, `logging`, `os`, `collections.abc`, `typing`, `unittest.mock`
- **Third-party / 第三方**: None / 无
- **PyTorch/Internal / PyTorch 内部**: `torch`, `torch.utils`, `torch.utils._ordered_set`, `..._dynamo.utils`, `..`, `..kernel.mm_common`, `..select_algorithm`, `..utils`, `..virtualized`, `.cpp`, `.cpp_gemm_template`, `.cpp_micro_gemm`, `.cpp_template_kernel`, `.cpp_utils`
