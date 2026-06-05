# test_cutlass_moe.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/kernels/moe/test_cutlass_moe.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Pytest module for kernel-level behavior in kernels / moe / test_cutlass_moe, focusing on correctness, edge cases, and comparisons against trusted reference paths. / 面向 kernels / moe / test_cutlass_moe 的内核级 pytest 模块，重点覆盖正确性、边界情况以及与可信参考路径的对照。

## Line-by-Line Analysis / 逐行分析
### Imports and shared setup (lines 3-30)
```python
import copy
import dataclasses
from math import prod

import pytest
import torch

import vllm.model_executor.layers.fused_moe.modular_kernel as mk
from tests.kernels.moe.utils import make_dummy_moe_config
from vllm import _custom_ops as ops
from vllm.config import ParallelConfig, VllmConfig, set_current_vllm_config
from vllm.model_executor.layers.fused_moe import fused_experts, fused_topk
from vllm.model_executor.layers.fused_moe.activation import MoEActivation
from vllm.model_executor.layers.fused_moe.all2all_utils import (
    maybe_make_prepare_finalize,
)
from vllm.model_executor.layers.fused_moe.config import (
    FUSED_MOE_UNQUANTIZED_CONFIG,
    FusedMoEQuantConfig,
    fp8_w8a8_moe_quant_config,
)
from vllm.model_executor.layers.fused_moe.experts.cutlass_moe import (
    CutlassExpertsFp8,
    run_cutlass_moe_fp8,
)
from vllm.model_executor.layers.fused_moe.utils import moe_kernel_quantize_input
from vllm.platforms import current_platform
from vllm.utils.torch_utils import set_random_seed
```
**EN:** This block imports the modules that the rest of the file relies on. It pulls in external libraries such as copy, dataclasses, math, pytest; shared test helpers from tests.kernels.moe.utils; and vLLM components like vllm.model_executor.layers.fused_moe.modular_kernel, vllm, vllm.config, vllm.model_executor.layers.fused_moe.
**CN:** 该代码块导入后续逻辑依赖的模块。 主要包括外部库，例如 copy、dataclasses、math、pytest；共享测试辅助模块，例如 tests.kernels.moe.utils；vLLM 内部组件，例如 vllm.model_executor.layers.fused_moe.modular_kernel、vllm、vllm.config、vllm.model_executor.layers.fused_moe。

### Constants and module state (lines 32-52)
```python
NUM_EXPERTS = [40, 64]
TOP_KS = [6, 8]

MNK_FACTORS = [
    (2, 1024, 1024),
    (2, 3072, 1024),
    (2, 3072, 1536),
    (7, 3072, 1536),
    (64, 1024, 1024),
    (64, 1024, 1536),
    (64, 3072, 1024),
    (224, 1024, 1024),
    (224, 3072, 1024),
    (224, 3072, 1536),
    (32768, 1024, 1024),
    # These sizes trigger wrong answers.
    # (7232, 2048, 5120),
    # (40000, 2048, 5120),
]

vllm_config = VllmConfig(parallel_config=ParallelConfig(pipeline_parallel_size=1))
```
**EN:** This block centralizes shared constants and parameter grids, including NUM_EXPERTS, TOP_KS, MNK_FACTORS. Those values keep later pytest scenarios consistent and make the test matrix easier to audit.
**CN:** 该代码块集中定义共享常量与参数网格，例如 NUM_EXPERTS、TOP_KS、MNK_FACTORS。这些值让后续 pytest 场景保持一致，也便于检查测试矩阵是否完整。

### Class `MOETensors` (lines 55-64)
```python
@dataclasses.dataclass
class MOETensors:
    a: torch.Tensor
    w1: torch.Tensor
    w2: torch.Tensor
    ab_strides1: torch.Tensor
    c_strides1: torch.Tensor
    ab_strides2: torch.Tensor
    c_strides2: torch.Tensor
```
**EN:** This dataclass packages the fields needed to describe MOETensors. Keeping the inputs structured makes parameter sets explicit and easy to pass between helpers.
**CN:** 这个 dataclass 把描述 MOETensors 所需的字段组织在一起，使参数集合更明确，也更容易在辅助函数之间传递。

### Method `MOETensors.make_moe_tensors` (lines 65-84)
```python
    @staticmethod
    def make_moe_tensors(
        m: int, k: int, n: int, e: int, dtype: torch.dtype
    ) -> "MOETensors":
        a = torch.randn((m, k), device="cuda", dtype=dtype) / 10
        w1 = torch.randn((e, 2 * n, k), device="cuda", dtype=dtype) / 10
        w2 = torch.randn((e, k, n), device="cuda", dtype=dtype) / 10
        ab_strides1 = torch.full((e,), k, device="cuda", dtype=torch.int64)
        c_strides1 = torch.full((e,), 2 * n, device="cuda", dtype=torch.int64)
        ab_strides2 = torch.full((e,), n, device="cuda", dtype=torch.int64)
        c_strides2 = torch.full((e,), k, device="cuda", dtype=torch.int64)
        return MOETensors(
            a=a,
            w1=w1,
            w2=w2,
            ab_strides1=ab_strides1,
            c_strides1=c_strides1,
            ab_strides2=ab_strides2,
            c_strides2=c_strides2,
        )
```
**EN:** This method on `MOETensors` implements MoE tensors. It keeps the surrounding module logic factored into a reusable unit.
**CN:** `MOETensors` 中的这个方法实现了 MoE tensors。 它把周边模块中的共用逻辑封装成可复用单元。

### Class `MOETensors8Bit` (lines 87-100)
```python
@dataclasses.dataclass
class MOETensors8Bit(MOETensors):
    # quantized
    a_q: torch.Tensor | None = None  # a -> a_q
    w1_q: torch.Tensor | None = None  # w1 -> w1_q
    w2_q: torch.Tensor | None = None  # w2 -> w2_q
    a_scale: torch.Tensor | None = None
    w1_scale: torch.Tensor | None = None
    w2_scale: torch.Tensor | None = None
    # dequantized
    a_d: torch.Tensor | None = None  # a -> a_q -> a_d
    w1_d: torch.Tensor | None = None  # w1 -> w1_q -> w1_d
    w2_d: torch.Tensor | None = None  # w2 -> w2_q -> w2_d
```
**EN:** This dataclass packages the fields needed to describe MOETensors8Bit. Keeping the inputs structured makes parameter sets explicit and easy to pass between helpers.
**CN:** 这个 dataclass 把描述 MOETensors8Bit 所需的字段组织在一起，使参数集合更明确，也更容易在辅助函数之间传递。

### Method `MOETensors8Bit.make_moe_tensors_8bit` (lines 101-156)
```python
    @staticmethod
    def make_moe_tensors_8bit(
        m: int, k: int, n: int, e: int, per_act_token: bool, per_out_channel: bool
    ) -> "MOETensors8Bit":
        dtype = torch.half
        q_dtype = torch.float8_e4m3fn

        moe_tensors_fp16 = MOETensors.make_moe_tensors(m, k, n, e, dtype)

        # a -> a_q, w1 -> w1_q, w2 -> w2_q
        n_b_scales = 2 * n if per_out_channel else 1
        k_b_scales = k if per_out_channel else 1
        # Get the right scale for tests.
        a_q, a_scale = ops.scaled_fp8_quant(
            moe_tensors_fp16.a, None, use_per_token_if_dynamic=per_act_token
        )

        w1_q = torch.empty((e, 2 * n, k), device="cuda", dtype=q_dtype)
        w2_q = torch.empty((e, k, n), device="cuda", dtype=q_dtype)

        w1_scale = torch.empty((e, n_b_scales, 1), device="cuda", dtype=torch.float32)
        w2_scale = torch.empty((e, k_b_scales, 1), device="cuda", dtype=torch.float32)
        for expert in range(e):
            w1_q[expert], w1_scale[expert] = ops.scaled_fp8_quant(
                moe_tensors_fp16.w1[expert], use_per_token_if_dynamic=per_out_channel
            )
            w2_q[expert], w2_scale[expert] = ops.scaled_fp8_quant(
                moe_tensors_fp16.w2[expert], use_per_token_if_dynamic=per_out_channel
            )

        # a_q -> a_d, w1_q -> w1_d, w2_q -> w2_d
        a_d = a_q.float().mul(a_scale).to(dtype)
        w1_d = torch.empty_like(moe_tensors_fp16.w1)
        w2_d = torch.empty_like(moe_tensors_fp16.w2)
        for expert in range(e):
            w1_d[expert] = (w1_q[expert].float() * w1_scale[expert]).half()
            w2_d[expert] = (w2_q[expert].float() * w2_scale[expert]).half()

        return MOETensors8Bit(
            a=moe_tensors_fp16.a,
            w1=moe_tensors_fp16.w1,
            w2=moe_tensors_fp16.w2,
            ab_strides1=moe_tensors_fp16.ab_strides1,
            c_strides1=moe_tensors_fp16.c_strides1,
            ab_strides2=moe_tensors_fp16.ab_strides2,
            c_strides2=moe_tensors_fp16.c_strides2,
            a_q=a_q,
            w1_q=w1_q,
            w2_q=w2_q,
            a_scale=a_scale,
            w1_scale=w1_scale,
            w2_scale=w2_scale,
            a_d=a_d,
            w1_d=w1_d,
            w2_d=w2_d,
        )
```
**EN:** This method on `MOETensors8Bit` implements MoE tensors 8bit. It keeps the surrounding module logic factored into a reusable unit.
**CN:** `MOETensors8Bit` 中的这个方法实现了 MoE tensors 8bit。 它把周边模块中的共用逻辑封装成可复用单元。

### Function `run_with_expert_maps` (lines 159-221)
```python
def run_with_expert_maps(
    num_experts: int,
    num_local_experts: int,
    quant_config: FusedMoEQuantConfig,
    **cutlass_moe_kwargs,
):
    def slice_experts():
        slice_params = [
            "w1",
            "w2",
        ]
        full_tensors = {
            k: v
            for k, v in cutlass_moe_kwargs.items()
            if k in slice_params and k in cutlass_moe_kwargs
        }

        for i in range(0, num_experts, num_local_experts):
            s, e = i, i + num_local_experts

            # make expert map
            expert_map = [-1] * num_experts
            expert_map[s:e] = list(range(num_local_experts))
            expert_map = torch.tensor(expert_map, dtype=torch.int32, device="cuda")

            # update cutlass moe arg with expert_map
            cutlass_moe_kwargs["expert_map"] = expert_map
            # update cutlass moe arg tensors
            for k, t in full_tensors.items():
                cutlass_moe_kwargs[k] = t[s:e]

            new_quant_config = copy.deepcopy(quant_config)
            new_quant_config._w1.scale = quant_config.w1_scale[s:e]
            new_quant_config._w2.scale = quant_config.w2_scale[s:e]

            yield cutlass_moe_kwargs, new_quant_config

    out_tensor = torch.zeros_like(cutlass_moe_kwargs["hidden_states"])
    for kwargs, new_quant_config in slice_experts():
        w2 = kwargs["w2"]
        a = kwargs["hidden_states"]
        moe_config = make_dummy_moe_config(
            num_experts=w2.shape[0],
            hidden_dim=w2.shape[1],
            intermediate_size_per_partition=w2.shape[2],
            in_dtype=a.dtype,
        )
        kernel = mk.FusedMoEKernel(
            maybe_make_prepare_finalize(
                moe=moe_config,
                quant_config=new_quant_config,
                allow_new_interface=True,
                use_monolithic=False,
            ),
            CutlassExpertsFp8(
                moe_config=moe_config,
                quant_config=new_quant_config,
            ),
            inplace=False,
        )
        out_tensor = out_tensor + kernel.apply(**kwargs)

    return out_tensor
```
**EN:** This helper function implements the shared logic for run with expert maps. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 该辅助函数实现了 run with expert maps 所需的共享逻辑。 它把周边模块中的共用逻辑封装成可复用单元。

### Function `run_8_bit` (lines 224-297)
```python
def run_8_bit(
    moe_tensors: MOETensors8Bit,
    topk_weights: torch.Tensor,
    topk_ids: torch.Tensor,
    per_act_token: bool,
    per_out_ch: bool,
    num_local_experts: int | None = None,
) -> torch.Tensor:
    assert not any(
        [
            t is None
            for t in [
                moe_tensors.w1_q,
                moe_tensors.w2_q,
                moe_tensors.w1_scale,
                moe_tensors.w2_scale,
                moe_tensors.a_scale,
            ]
        ]
    )

    quant_config = fp8_w8a8_moe_quant_config(
        w1_scale=moe_tensors.w1_scale,
        w2_scale=moe_tensors.w2_scale,
        per_act_token_quant=per_act_token,
        per_out_ch_quant=per_out_ch,
        # Set to moe_tensors.a_scale iff static scales + per tensor.
        # This is not currently being tested.
        a1_scale=None,
    )

    kwargs = {
        "hidden_states": moe_tensors.a,
        "w1": moe_tensors.w1_q,  # type: ignore[union-attr]
        "w2": moe_tensors.w2_q,  # type: ignore[union-attr]
        "topk_weights": topk_weights,
        "topk_ids": topk_ids,
        "global_num_experts": moe_tensors.w1_q.shape[0],  # type: ignore[union-attr]
        "activation": MoEActivation.SILU,
        "expert_map": None,
        "apply_router_weight_on_input": False,
    }

    num_experts = moe_tensors.w1.size(0)  # type: ignore[attr-defined]
    with_ep = num_local_experts is not None or num_local_experts == num_experts
    if not with_ep:
        moe_config = make_dummy_moe_config(
            num_experts=moe_tensors.w2_q.shape[0],  # type: ignore[union-attr]
            hidden_dim=moe_tensors.w2_q.shape[1],  # type: ignore[union-attr]
            intermediate_size_per_partition=moe_tensors.w2_q.shape[2],  # type: ignore[union-attr]
            in_dtype=moe_tensors.a.dtype,
        )
        kernel = mk.FusedMoEKernel(
            maybe_make_prepare_finalize(
                moe=moe_config,
                quant_config=quant_config,
                allow_new_interface=True,
                use_monolithic=False,
            ),
            CutlassExpertsFp8(
                moe_config=moe_config,
                quant_config=quant_config,
            ),
            inplace=False,
        )
        return kernel.apply(**kwargs)

    assert num_local_experts is not None
    return run_with_expert_maps(
        num_experts,
        num_local_experts,  # type: ignore[arg-type]
        quant_config,
        **kwargs,
    )
```
**EN:** This helper function implements the shared logic for run 8 bit. assertions at the end lock in the intended behavior or graph shape.
**CN:** 该辅助函数实现了 run 8 bit 所需的共享逻辑。 结尾处的断言会固定预期行为或计算图形态。

### Function `test_cutlass_moe_8_bit_no_graph` (lines 300-352)
```python
@pytest.mark.parametrize("m,n,k", MNK_FACTORS)
@pytest.mark.parametrize("e", NUM_EXPERTS)
@pytest.mark.parametrize("topk", TOP_KS)
@pytest.mark.parametrize("per_act_token", [True, False])
@pytest.mark.parametrize("per_out_ch", [True, False])
@pytest.mark.skipif(
    (lambda x: x is None or not ops.cutlass_group_gemm_supported(x.to_int()))(
        current_platform.get_device_capability()
    ),
    reason="Grouped gemm is not supported on this GPU type.",
)
def test_cutlass_moe_8_bit_no_graph(
    m: int,
    n: int,
    k: int,
    e: int,
    topk: int,
    per_act_token: bool,
    per_out_ch: bool,
    monkeypatch,
    workspace_init,
    ep_size: int | None = None,
):
    set_random_seed(7)
    with set_current_vllm_config(vllm_config):
        mt = MOETensors8Bit.make_moe_tensors_8bit(m, k, n, e, per_act_token, per_out_ch)

        score = torch.randn((m, e), device="cuda", dtype=torch.half)
        topk_weights, topk_ids, _ = fused_topk(mt.a, score, topk, renormalize=False)

        # Note that we are using the dequantized versions of the tensors.
        # Using a, w1 and w2 directly results in minor output differences.

        quant_config = FUSED_MOE_UNQUANTIZED_CONFIG
        triton_output = fused_experts(
            mt.a_d, mt.w1_d, mt.w2_d, topk_weights, topk_ids, quant_config=quant_config
        )

        if ep_size is not None:
            assert e % ep_size == 0, "Cannot distribute experts evenly"
            number_local_experts = e // ep_size
        else:
            number_local_experts = None

        cutlass_output = run_8_bit(
            mt, topk_weights, topk_ids, per_act_token, per_out_ch, number_local_experts
        )

        # Note 5.5 only needed for larger problem sizes, 5 works ok for
        # the rest.
        torch.testing.assert_close(
            triton_output, cutlass_output, atol=5.5e-2, rtol=1e-2
        )
```
**EN:** This pytest case verifies cutlass MoE 8 bit no graph. It is parameterized across 5 input dimensions so one definition covers many scenarios. it consumes fixtures or inputs such as m, n, k, e. unsupported hardware, backend, or configuration combinations are skipped early. numeric results are compared against a reference with explicit tolerances.
**CN:** 该 pytest 用例验证 cutlass MoE 8 bit no graph 的行为。 它通过 5 组参数化输入覆盖多种场景；它会使用诸如 m、n、k、e 等 fixture 或输入；不支持的硬件、后端或配置组合会被提前跳过；数值结果会在显式容差下与参考结果进行比较。

### Function `test_cutlass_moe_8_bit_cuda_graph` (lines 355-404)
```python
@pytest.mark.parametrize("m,n,k", MNK_FACTORS)
@pytest.mark.parametrize("e", NUM_EXPERTS)
@pytest.mark.parametrize("topk", TOP_KS)
@pytest.mark.parametrize("per_act_token", [True, False])
@pytest.mark.parametrize("per_out_ch", [True, False])
@pytest.mark.skipif(
    (lambda x: x is None or not ops.cutlass_group_gemm_supported(x.to_int()))(
        current_platform.get_device_capability()
    ),
    reason="Grouped gemm is not supported on this GPU type.",
)
def test_cutlass_moe_8_bit_cuda_graph(
    m: int,
    n: int,
    k: int,
    e: int,
    topk: int,
    per_act_token: bool,
    per_out_ch: bool,
    monkeypatch,
    workspace_init,
):
    set_random_seed(7)
    with set_current_vllm_config(vllm_config):
        dtype = torch.half

        mt = MOETensors8Bit.make_moe_tensors_8bit(m, k, n, e, per_act_token, per_out_ch)

        score = torch.randn((m, e), device="cuda", dtype=dtype)
        topk_weights, topk_ids, _ = fused_topk(mt.a, score, topk, renormalize=False)

        # Note that we are using the dequantized versions of the tensors.
        # Using a, w1 and w2 directly results in minor output differences.
        quant_config = FUSED_MOE_UNQUANTIZED_CONFIG
        triton_output = fused_experts(
            mt.a_d, mt.w1_d, mt.w2_d, topk_weights, topk_ids, quant_config=quant_config
        )

        stream = torch.cuda.Stream()
        graph = torch.cuda.CUDAGraph()
        with torch.cuda.graph(graph, stream=stream):
            cutlass_output = run_8_bit(
                mt, topk_weights, topk_ids, per_act_token, per_out_ch
            )

        torch.accelerator.synchronize()
        graph.replay()
        torch.accelerator.synchronize()

        torch.testing.assert_close(triton_output, cutlass_output, atol=9e-2, rtol=1e-2)
```
**EN:** This pytest case verifies cutlass MoE 8 bit CUDA graph. It is parameterized across 5 input dimensions so one definition covers many scenarios. it consumes fixtures or inputs such as m, n, k, e. unsupported hardware, backend, or configuration combinations are skipped early. numeric results are compared against a reference with explicit tolerances.
**CN:** 该 pytest 用例验证 cutlass MoE 8 bit CUDA graph 的行为。 它通过 5 组参数化输入覆盖多种场景；它会使用诸如 m、n、k、e 等 fixture 或输入；不支持的硬件、后端或配置组合会被提前跳过；数值结果会在显式容差下与参考结果进行比较。

### Function `test_cutlass_moe_8_bit_EP` (lines 407-444)
```python
@pytest.mark.parametrize("m", [64])
@pytest.mark.parametrize("n", [1024])
@pytest.mark.parametrize("k", [4096])
@pytest.mark.parametrize("e", [16])
@pytest.mark.parametrize("topk", [1, 8])
@pytest.mark.parametrize("per_act_token", [True])
@pytest.mark.parametrize("per_out_channel", [True])
@pytest.mark.parametrize("ep_size", [1, 2, 4, 8, 16])
@pytest.mark.skipif(
    (lambda x: x is None or not ops.cutlass_group_gemm_supported(x.to_int()))(
        current_platform.get_device_capability()
    ),
    reason="Grouped gemm is not supported on this GPU type.",
)
def test_cutlass_moe_8_bit_EP(
    m: int,
    n: int,
    k: int,
    e: int,
    topk: int,
    per_act_token: bool,
    per_out_channel: bool,
    ep_size: int,
    monkeypatch,
    workspace_init,
):
    test_cutlass_moe_8_bit_no_graph(
        m,
        n,
        k,
        e,
        topk,
        per_act_token,
        per_out_channel,
        monkeypatch,
        workspace_init,
        ep_size,
    )
```
**EN:** This pytest case verifies cutlass MoE 8 bit EP. It is parameterized across 8 input dimensions so one definition covers many scenarios. it consumes fixtures or inputs such as m, n, k, e. unsupported hardware, backend, or configuration combinations are skipped early.
**CN:** 该 pytest 用例验证 cutlass MoE 8 bit EP 的行为。 它通过 8 组参数化输入覆盖多种场景；它会使用诸如 m、n、k、e 等 fixture 或输入；不支持的硬件、后端或配置组合会被提前跳过。

### Constants and module state (lines 447-451)
```python
LARGE_MNK_FACTORS = [
    (1, 8192, 5120, 31),
    (32768, 1024, 1024, 16),
    (65536, 512, 1024, 16),
]
```
**EN:** This block centralizes shared constants and parameter grids, including LARGE_MNK_FACTORS. Those values keep later pytest scenarios consistent and make the test matrix easier to audit.
**CN:** 该代码块集中定义共享常量与参数网格，例如 LARGE_MNK_FACTORS。这些值让后续 pytest 场景保持一致，也便于检查测试矩阵是否完整。

### Function `test_cutlass_moe_8_bit_EP_large` (lines 454-488)
```python
@pytest.mark.parametrize("m,n,k,topk", LARGE_MNK_FACTORS)
@pytest.mark.parametrize("e", [128])
@pytest.mark.parametrize("per_act_token", [False])
@pytest.mark.parametrize("per_out_channel", [True])
@pytest.mark.parametrize("ep_size", [8])
@pytest.mark.skipif(
    (lambda x: x is None or not ops.cutlass_group_gemm_supported(x.to_int()))(
        current_platform.get_device_capability()
    ),
    reason="Grouped gemm is not supported on this GPU type.",
)
def test_cutlass_moe_8_bit_EP_large(
    m: int,
    n: int,
    k: int,
    e: int,
    topk: int,
    per_act_token: bool,
    per_out_channel: bool,
    ep_size: int,
    monkeypatch,
    workspace_init,
):
    test_cutlass_moe_8_bit_no_graph(
        m,
        n,
        k,
        e,
        topk,
        per_act_token,
        per_out_channel,
        monkeypatch,
        workspace_init,
        ep_size,
    )
```
**EN:** This pytest case verifies cutlass MoE 8 bit EP large. It is parameterized across 5 input dimensions so one definition covers many scenarios. it consumes fixtures or inputs such as m, n, k, e. unsupported hardware, backend, or configuration combinations are skipped early.
**CN:** 该 pytest 用例验证 cutlass MoE 8 bit EP large 的行为。 它通过 5 组参数化输入覆盖多种场景；它会使用诸如 m、n、k、e 等 fixture 或输入；不支持的硬件、后端或配置组合会被提前跳过。

### Function `test_run_cutlass_moe_fp8` (lines 491-595)
```python
@pytest.mark.parametrize("m,n,k,topk", [(1, 8192, 5120, 31)])
@pytest.mark.parametrize("e", [128])
@pytest.mark.parametrize("per_act_token", [False])
@pytest.mark.parametrize("per_out_channel", [True])
@pytest.mark.parametrize("ep_size", [8])
@pytest.mark.skipif(
    (lambda x: x is None or not ops.cutlass_group_gemm_supported(x.to_int()))(
        current_platform.get_device_capability()
    ),
    reason="Grouped gemm is not supported on this GPU type.",
)
def test_run_cutlass_moe_fp8(
    m: int,
    n: int,
    k: int,
    e: int,
    topk: int,
    per_act_token: bool,
    per_out_channel: bool,
    ep_size: int,
    workspace_init,
):
    set_random_seed(7)
    with set_current_vllm_config(vllm_config):
        mt = MOETensors8Bit.make_moe_tensors_8bit(
            m, k, n, e, per_act_token, per_out_channel
        )

        score = torch.randn((m, e), device="cuda", dtype=torch.half)
        topk_weights, topk_ids, _ = fused_topk(mt.a, score, topk, renormalize=False)
        # we want to make sure there is at least one token that's generated in
        # this expert shard and at least one token that's NOT generated in this
        # expert shard
        topk_ids[0][0] = -1
        topk_ids[0][1] = 1

        workspace13_shape = (m * topk, max(2 * n, k))
        workspace2_shape = (m * topk, max(n, k))
        output_shape = (m, k)

# ... excerpt ...
            mt.w1_q,
            mt.w2_q,
            topk_ids,
            activation,
            global_num_experts,
            expert_map,
            mt.w1_scale,
            mt.w2_scale,
            a1q_scale,
            None,
            ab_strides1,
            ab_strides2,
            c_strides1,
            c_strides2,
            workspace13,
            workspace2,
            None,
            mt.a.dtype,
            per_act_token,
            per_out_channel,
            False,
            topk_weights,
        )

        workspace13.random_()
        output_random_workspace = torch.empty(
            output_shape, device="cuda", dtype=mt.a.dtype
        )
        func(output_random_workspace)

        workspace13.fill_(0)
        output_zero_workspace = torch.zeros(
            output_shape, device="cuda", dtype=mt.a.dtype
        )
        func(output_zero_workspace)

        torch.testing.assert_close(
            output_random_workspace, output_zero_workspace, atol=5e-3, rtol=1e-3
        )
```
**EN:** This pytest case verifies run cutlass MoE FP8. It is parameterized across 5 input dimensions so one definition covers many scenarios. it consumes fixtures or inputs such as m, n, k, e. unsupported hardware, backend, or configuration combinations are skipped early. numeric results are compared against a reference with explicit tolerances. Only the key portions are shown here because the block is large.
**CN:** 该 pytest 用例验证 run cutlass MoE FP8 的行为。 它通过 5 组参数化输入覆盖多种场景；它会使用诸如 m、n、k、e 等 fixture 或输入；不支持的硬件、后端或配置组合会被提前跳过；数值结果会在显式容差下与参考结果进行比较。 由于该代码块较大，这里只展示关键片段。

## Key Concepts / 关键概念
- **Pytest parameterization / pytest 参数化:** The module expands one definition into many concrete scenarios through parametrized inputs. / 该模块通过参数化输入把单个定义扩展成多组具体场景。
- **Behavioral assertions / 行为断言:** Assertions check tensor values, graph structure, or expected error conditions. / 断言用于检查张量结果、计算图结构或预期的报错路径。
- **Platform guards / 平台保护:** Hardware, backend, or version checks prevent unsupported combinations from running. / 硬件、后端或版本检查会阻止不受支持的组合继续执行。
- **Reference comparisons / 参考实现对照:** Optimized kernels are commonly checked against simpler reference implementations or fallback paths. / 优化后的内核通常会与更直接的参考实现或回退路径进行比对。

## Dependencies / 依赖关系
- `copy`
- `dataclasses`
- `math -> prod`
- `pytest`
- `torch`
- `vllm.model_executor.layers.fused_moe.modular_kernel`
- `tests.kernels.moe.utils -> make_dummy_moe_config`
- `vllm -> _custom_ops`
- `vllm.config -> ParallelConfig, VllmConfig, set_current_vllm_config`
- `vllm.model_executor.layers.fused_moe -> fused_experts, fused_topk`
- `vllm.model_executor.layers.fused_moe.activation -> MoEActivation`
- `vllm.model_executor.layers.fused_moe.all2all_utils -> maybe_make_prepare_finalize`
- `vllm.model_executor.layers.fused_moe.config -> FUSED_MOE_UNQUANTIZED_CONFIG, FusedMoEQuantConfig, fp8_w8a8_moe_quant_config`
- `vllm.model_executor.layers.fused_moe.experts.cutlass_moe -> CutlassExpertsFp8, run_cutlass_moe_fp8`
- `vllm.model_executor.layers.fused_moe.utils -> moe_kernel_quantize_input`
- `vllm.platforms -> current_platform`
