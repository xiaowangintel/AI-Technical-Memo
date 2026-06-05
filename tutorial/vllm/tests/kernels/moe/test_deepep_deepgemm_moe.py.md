# test_deepep_deepgemm_moe.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/kernels/moe/test_deepep_deepgemm_moe.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Pytest module for kernel-level behavior in kernels / moe / test_deepep_deepgemm_moe, focusing on correctness, edge cases, and comparisons against trusted reference paths. / 面向 kernels / moe / test_deepep_deepgemm_moe 的内核级 pytest 模块，重点覆盖正确性、边界情况以及与可信参考路径的对照。

## Line-by-Line Analysis / 逐行分析
### Module docstring (lines 3-7)
```python
"""
Test DeepEP + DeepGEMM integration
DeepGEMM are gemm kernels specialized for the
fp8 block-quantized case.
"""
```
**EN:** This docstring states the file-level intent and helps readers understand the role of the module before they inspect the implementation.
**CN:** 这个文档字符串说明了文件级意图，让读者在查看实现前先理解模块职责。

### Imports and shared setup (lines 9-38)
```python
import dataclasses
from contextlib import contextmanager

import pytest
import torch.distributed
from torch.distributed import ProcessGroup
from typing_extensions import ParamSpec

from vllm.config import VllmConfig, set_current_vllm_config
from vllm.forward_context import set_forward_context
from vllm.model_executor.layers.fused_moe.activation import MoEActivation
from vllm.model_executor.layers.fused_moe.config import (
    FusedMoEQuantConfig,
    fp8_w8a8_moe_quant_config,
)
from vllm.model_executor.layers.fused_moe.fused_moe import fused_experts
from vllm.model_executor.layers.fused_moe.modular_kernel import FusedMoEKernel
from vllm.utils.deep_gemm import (
    get_mk_alignment_for_contiguous_layout,
    is_deep_gemm_e8m0_used,
    is_deep_gemm_supported,
)
from vllm.utils.import_utils import has_deep_ep, has_deep_gemm
from vllm.utils.math_utils import next_power_of_2
from vllm.utils.torch_utils import set_random_seed
from vllm.v1.worker.workspace import init_workspace_manager

from ...utils import multi_gpu_test
from .parallel_utils import ProcessGroupInfo, parallel_launch
from .utils import make_dummy_moe_config, make_test_weights
```
**EN:** This block imports the modules that the rest of the file relies on. It pulls in external libraries such as dataclasses, contextlib, pytest, torch.distributed; shared test helpers from ...utils, .parallel_utils, .utils; and vLLM components like vllm.config, vllm.forward_context, vllm.model_executor.layers.fused_moe.activation, vllm.model_executor.layers.fused_moe.config.
**CN:** 该代码块导入后续逻辑依赖的模块。 主要包括外部库，例如 dataclasses、contextlib、pytest、torch.distributed；共享测试辅助模块，例如 ...utils、.parallel_utils、.utils；vLLM 内部组件，例如 vllm.config、vllm.forward_context、vllm.model_executor.layers.fused_moe.activation、vllm.model_executor.layers.fused_moe.config。

### Top-level block starting at line 40 (lines 40-48)
```python
if has_deep_ep():
    from vllm.model_executor.layers.fused_moe.prepare_finalize.deepep_ht import (
        DeepEPHTPrepareAndFinalize,
    )
    from vllm.model_executor.layers.fused_moe.prepare_finalize.deepep_ll import (
        DeepEPLLPrepareAndFinalize,
    )

    from .parallel_utils import DeepEPHTArgs, DeepEPLLArgs, make_deepep_a2a
```
**EN:** This top-level `If` block performs supporting work that the surrounding tests depend on.
**CN:** 这个顶层 `If` 代码块执行周边测试所依赖的辅助逻辑。

### Top-level block starting at line 50 (lines 50-56)
```python
if has_deep_gemm():
    from vllm.model_executor.layers.fused_moe.experts.batched_deep_gemm_moe import (
        BatchedDeepGemmExperts,
    )
    from vllm.model_executor.layers.fused_moe.experts.deep_gemm_moe import (
        DeepGemmExperts,
    )
```
**EN:** This top-level `If` block performs supporting work that the surrounding tests depend on.
**CN:** 这个顶层 `If` 代码块执行周边测试所依赖的辅助逻辑。

### Constants and module state (lines 58-68)
```python
requires_deep_ep = pytest.mark.skipif(
    not has_deep_ep(),
    reason="Requires deep_ep kernels",
)

requires_deep_gemm = pytest.mark.skipif(
    not is_deep_gemm_supported(),
    reason="Requires deep_gemm kernels",
)

P = ParamSpec("P")
```
**EN:** This block centralizes shared constants and parameter grids, including P. Those values keep later pytest scenarios consistent and make the test matrix easier to audit.
**CN:** 该代码块集中定义共享常量与参数网格，例如 P。这些值让后续 pytest 场景保持一致，也便于检查测试矩阵是否完整。

### Function `with_dp_metadata` (lines 71-85)
```python
@contextmanager
def with_dp_metadata(M: int, world_size: int):
    num_tokens_across_dp = torch.tensor([M] * world_size, device="cpu", dtype=torch.int)

    vllm_config = VllmConfig()
    vllm_config.parallel_config.data_parallel_size = world_size
    vllm_config.parallel_config.enable_expert_parallel = True

    with set_forward_context(
        None,
        vllm_config,
        num_tokens=M,
        num_tokens_across_dp=num_tokens_across_dp,
    ):
        yield
```
**EN:** This helper function implements the shared logic for with dp metadata. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 该辅助函数实现了 with dp metadata 所需的共享逻辑。 它把周边模块中的共用逻辑封装成可复用单元。

### Function `make_block_quant_fp8_weights` (lines 88-100)
```python
def make_block_quant_fp8_weights(
    e: int,
    n: int,
    k: int,
    block_size: list[int],
) -> tuple[torch.Tensor, torch.Tensor, torch.Tensor, torch.Tensor]:
    """
    Return weights w1q, w2q, w1_scale, w2_scale
    """
    (_, w1q, w1_scale, _), (_, w2q, w2_scale, _) = make_test_weights(
        e, n, k, torch.bfloat16, torch.float8_e4m3fn, block_shape=block_size
    )
    return w1q, w2q, w1_scale, w2_scale
```
**EN:** This helper function implements the shared logic for block quant FP8 weights. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 该辅助函数实现了 block quant FP8 weights 所需的共享逻辑。 它把周边模块中的共用逻辑封装成可复用单元。

### Class `TestConfig` (lines 103-114)
```python
@dataclasses.dataclass
class TestConfig:
    topk: int
    m: int
    k: int
    n: int
    num_experts: int
    per_act_token_quant: bool
    block_size: list[int]
    # configs for testing low-latency kernels
    low_latency: bool
    use_fp8_dispatch: bool | None = False
```
**EN:** This dataclass packages the fields needed to describe TestConfig. Keeping the inputs structured makes parameter sets explicit and easy to pass between helpers.
**CN:** 这个 dataclass 把描述 TestConfig 所需的字段组织在一起，使参数集合更明确，也更容易在辅助函数之间传递。

### Class `TestTensors` (lines 117-124)
```python
@dataclasses.dataclass
class TestTensors:
    rank_tokens: torch.Tensor  # all ranks make this many tokens
    rank_token_scales: torch.Tensor | None
    topk: torch.Tensor
    topk_weights: torch.Tensor
    config: TestConfig
```
**EN:** This dataclass packages the fields needed to describe TestTensors. Keeping the inputs structured makes parameter sets explicit and easy to pass between helpers.
**CN:** 这个 dataclass 把描述 TestTensors 所需的字段组织在一起，使参数集合更明确，也更容易在辅助函数之间传递。

### Method `TestTensors.make` (lines 125-156)
```python
    @staticmethod
    def make(config: TestConfig, rank) -> "TestTensors":
        dtype = torch.bfloat16
        topk, m, k = (config.topk, config.m, config.k)

        fp8_info = torch.finfo(torch.float8_e4m3fn)
        fp8_max, fp8_min = fp8_info.max, fp8_info.min
        device = torch.accelerator.current_device_index()
        rank_tokens = torch.randn((m, k), device=device, dtype=dtype) / 10.0
        rank_tokens = rank_tokens.clamp(min=fp8_min, max=fp8_max)
        rank_token_scales = None

        topk_ids = torch.randint(
            low=0,
            high=config.num_experts,
            size=(m, topk),
            device=device,
        ).to(dtype=torch.int64)

        topk_weights = torch.randn(
            topk_ids.shape,
            dtype=torch.float32,
            device=device,
        )

        return TestTensors(
            rank_tokens=rank_tokens,
            rank_token_scales=rank_token_scales,
            topk=topk_ids,
            topk_weights=topk_weights,
            config=config,
        )
```
**EN:** This method on `TestTensors` implements make. It keeps the surrounding module logic factored into a reusable unit.
**CN:** `TestTensors` 中的这个方法实现了 make。 它把周边模块中的共用逻辑封装成可复用单元。

### Function `make_ll_modular_kernel` (lines 159-197)
```python
def make_ll_modular_kernel(
    pg: ProcessGroup,
    pgi: ProcessGroupInfo,
    max_tokens_per_rank: int,
    dp_size: int,
    hidden_size: int,
    q_dtype: torch.dtype | None,
    test_config: TestConfig,
    quant_config: FusedMoEQuantConfig,
) -> FusedMoEKernel:
    assert test_config.low_latency
    assert test_config.use_fp8_dispatch is not None

    a2a: DeepEPLLPrepareAndFinalize = make_deepep_a2a(
        pg=pg,
        pgi=pgi,
        dp_size=dp_size,
        deepep_ht_args=None,
        deepep_ll_args=DeepEPLLArgs(
            max_tokens_per_rank=max_tokens_per_rank,
            hidden_size=hidden_size,
            num_experts=test_config.num_experts,
            use_fp8_dispatch=test_config.use_fp8_dispatch,
        ),
        q_dtype=q_dtype,
        block_shape=test_config.block_size,
    )

    fused_experts = BatchedDeepGemmExperts(
        max_num_tokens=max_tokens_per_rank,
        num_dispatchers=pgi.world_size // dp_size,
        quant_config=quant_config,
        moe_config=make_dummy_moe_config(),
    )
    return FusedMoEKernel(
        prepare_finalize=a2a,
        fused_experts=fused_experts,
        inplace=False,
    )
```
**EN:** This helper function implements the shared logic for ll modular kernel. assertions at the end lock in the intended behavior or graph shape.
**CN:** 该辅助函数实现了 ll modular kernel 所需的共享逻辑。 结尾处的断言会固定预期行为或计算图形态。

### Function `make_ht_modular_kernel` (lines 200-230)
```python
def make_ht_modular_kernel(
    pg: ProcessGroup,
    pgi: ProcessGroupInfo,
    dp_size: int,
    num_local_experts: int,
    q_dtype: torch.dtype | None,
    test_config: TestConfig,
    quant_config: FusedMoEQuantConfig,
) -> FusedMoEKernel:
    assert not test_config.low_latency
    assert test_config.use_fp8_dispatch is None

    a2a: DeepEPHTPrepareAndFinalize = make_deepep_a2a(
        pg=pg,
        pgi=pgi,
        dp_size=dp_size,
        deepep_ht_args=DeepEPHTArgs(num_local_experts=num_local_experts),
        deepep_ll_args=None,
        q_dtype=q_dtype,
        block_shape=test_config.block_size,
    )

    fused_experts = DeepGemmExperts(
        moe_config=make_dummy_moe_config(),
        quant_config=quant_config,
    )
    return FusedMoEKernel(
        prepare_finalize=a2a,
        fused_experts=fused_experts,
        inplace=False,
    )
```
**EN:** This helper function implements the shared logic for ht modular kernel. assertions at the end lock in the intended behavior or graph shape.
**CN:** 该辅助函数实现了 ht modular kernel 所需的共享逻辑。 结尾处的断言会固定预期行为或计算图形态。

### Function `make_modular_kernel` (lines 233-271)
```python
def make_modular_kernel(
    pg: ProcessGroup,
    pgi: ProcessGroupInfo,
    dp_size: int,
    num_local_experts: int,
    test_tensors: TestTensors,
    quant_config: FusedMoEQuantConfig,
) -> FusedMoEKernel:
    q_dtype = torch.float8_e4m3fn
    test_config = test_tensors.config

    mk: FusedMoEKernel
    # Make modular kernel
    if test_config.low_latency:
        max_tokens_per_rank = max(64, next_power_of_2(test_tensors.rank_tokens.size(0)))
        hidden_size = test_tensors.rank_tokens.size(-1)

        mk = make_ll_modular_kernel(
            pg=pg,
            pgi=pgi,
            max_tokens_per_rank=max_tokens_per_rank,
            dp_size=dp_size,
            hidden_size=hidden_size,
            q_dtype=q_dtype,
            test_config=test_config,
            quant_config=quant_config,
        )
    else:
        mk = make_ht_modular_kernel(
            pg,
            pgi,
            dp_size,
            num_local_experts,
            q_dtype,
            test_config,
            quant_config=quant_config,
        )

    return mk
```
**EN:** This helper function implements the shared logic for modular kernel. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 该辅助函数实现了 modular kernel 所需的共享逻辑。 它把周边模块中的共用逻辑封装成可复用单元。

### Function `deepep_deepgemm_moe_impl` (lines 274-329)
```python
def deepep_deepgemm_moe_impl(
    pg: ProcessGroup,
    pgi: ProcessGroupInfo,
    dp_size: int,
    test_tensors: TestTensors,
    w1: torch.Tensor,
    w2: torch.Tensor,
    w1_scale: torch.Tensor | None,
    w2_scale: torch.Tensor | None,
) -> torch.Tensor:
    test_config = test_tensors.config
    num_experts = test_config.num_experts
    num_local_experts = w1.size(0)

    def build_expert_map():
        num_local_experts = w1.size(0)
        expert_map = torch.full((num_experts,), fill_value=-1, dtype=torch.int32)
        s = pgi.rank * num_local_experts
        e = s + num_local_experts
        expert_map[s:e] = torch.tensor(list(range(num_local_experts)))
        device = torch.accelerator.current_device_index()
        return expert_map.to(device=device, dtype=torch.int32)

    quant_config = fp8_w8a8_moe_quant_config(
        w1_scale=w1_scale,
        w2_scale=w2_scale,
        # Low-Latency kernels can't dispatch scales.
        a1_scale=(None if test_config.low_latency else test_tensors.rank_token_scales),
        block_shape=test_config.block_size,
    )

    # Make modular kernel
    mk: FusedMoEKernel = make_modular_kernel(
        pg=pg,
        pgi=pgi,
        dp_size=dp_size,
        num_local_experts=num_local_experts,
        test_tensors=test_tensors,
        quant_config=quant_config,
    )

    with with_dp_metadata(
        M=test_tensors.rank_tokens.size(0), world_size=pgi.world_size
    ):
        out = mk.apply(
            hidden_states=test_tensors.rank_tokens,
            w1=w1,
            w2=w2,
            topk_weights=test_tensors.topk_weights,
            topk_ids=test_tensors.topk,
            activation=MoEActivation.SILU,
            global_num_experts=num_experts,
            expert_map=build_expert_map(),
            apply_router_weight_on_input=False,
        )
    return out
```
**EN:** This helper function implements the shared logic for deepep deepgemm MoE impl. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 该辅助函数实现了 deepep deepgemm MoE impl 所需的共享逻辑。 它把周边模块中的共用逻辑封装成可复用单元。

### Function `triton_impl` (lines 332-358)
```python
def triton_impl(
    a: torch.Tensor,
    topk_ids: torch.Tensor,
    topk_weights: torch.Tensor,
    w1: torch.Tensor,
    w2: torch.Tensor,
    w1_scale: torch.Tensor,
    w2_scale: torch.Tensor,
    a1_scale: torch.Tensor,
    block_shape: list[int],
):
    quant_config = fp8_w8a8_moe_quant_config(
        w1_scale=w1_scale,
        w2_scale=w2_scale,
        a1_scale=a1_scale,
        block_shape=block_shape,
    )

    return fused_experts(
        hidden_states=a,
        w1=w1,
        w2=w2,
        topk_weights=topk_weights,
        topk_ids=topk_ids,
        inplace=False,
        quant_config=quant_config,
    )
```
**EN:** This helper function implements the shared logic for triton impl. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 该辅助函数实现了 triton impl 所需的共享逻辑。 它把周边模块中的共用逻辑封装成可复用单元。

### Function `_test_deepep_deepgemm_moe` (lines 361-424)
```python
def _test_deepep_deepgemm_moe(
    pgi: ProcessGroupInfo,
    dp_size: int,
    config: TestConfig,
    w1: torch.Tensor,
    w2: torch.Tensor,
    w1_scale: torch.Tensor,
    w2_scale: torch.Tensor,
):
    device = torch.device(f"cuda:{pgi.local_rank}")
    init_workspace_manager(device)

    set_random_seed(pgi.rank)

    device = torch.accelerator.current_device_index()
    w1 = w1.to(device=device)
    w2 = w2.to(device=device)
    w1_scale = w1_scale.to(device=device)
    w2_scale = w2_scale.to(device=device)

    pg = torch.distributed.new_group(list(range(pgi.world_size)))
    test_tensors = TestTensors.make(config, pgi.rank)
    block_shape = [w1.size(1) // w1_scale.size(1), w1.size(2) // w1_scale.size(2)]

    with set_current_vllm_config(VllmConfig()):
        # Reference
        triton_moe = triton_impl(
            a=test_tensors.rank_tokens,
            topk_ids=test_tensors.topk,
            topk_weights=test_tensors.topk_weights,
            w1=w1,
            w2=w2,
            w1_scale=w1_scale,
            w2_scale=w2_scale,
            a1_scale=test_tensors.rank_token_scales,
            block_shape=block_shape,
        )

        # Slice experts for this rank.
        num_local_experts = config.num_experts // pgi.world_size
        e_start = num_local_experts * pgi.rank
        e_end = e_start + num_local_experts
        w1_ep = w1[e_start:e_end]
        w2_ep = w2[e_start:e_end]
        w1_scale_ep = w1_scale[e_start:e_end]
        w2_scale_ep = w2_scale[e_start:e_end]

        deepep_moe = deepep_deepgemm_moe_impl(
            pg,
            pgi,
            dp_size,
            test_tensors,
            w1_ep,
            w2_ep,
            w1_scale_ep,
            w2_scale_ep,
        )

    torch.testing.assert_close(
        triton_moe,
        deepep_moe,
        atol=6e-2,
        rtol=6e-2,
    )
```
**EN:** This helper function implements the shared logic for test deepep deepgemm MoE. numeric results are compared against a reference with explicit tolerances. randomness is controlled so the scenario remains reproducible.
**CN:** 该辅助函数实现了 test deepep deepgemm MoE 所需的共享逻辑。 数值结果会在显式容差下与参考结果进行比较；代码会控制随机性以保证场景可复现。

### Constants and module state (lines 427-440)
```python
MNKs = [
    (8, 128, 128),
    (8, 128, 512),
    (3, 1024, 2048),
    (32, 128, 1024),
    (45, 512, 2048),
    (64, 1024, 1024),
    (129, 128, 256),
    (129, 1024, 2048),
    (222, 1024, 2048),
]

TOPKS = [2, 6]
NUM_EXPERTS = [32]
```
**EN:** This block centralizes shared constants and parameter grids, including TOPKS, NUM_EXPERTS. Those values keep later pytest scenarios consistent and make the test matrix easier to audit.
**CN:** 该代码块集中定义共享常量与参数网格，例如 TOPKS、NUM_EXPERTS。这些值让后续 pytest 场景保持一致，也便于检查测试矩阵是否完整。

### Function `test_ht_deepep_deepgemm_moe` (lines 443-497)
```python
@pytest.mark.parametrize("mnk", MNKs)
@pytest.mark.parametrize("num_experts", NUM_EXPERTS)
@pytest.mark.parametrize("topk", TOPKS)
@pytest.mark.parametrize("world_dp_size", [(2, 1)])
@multi_gpu_test(num_gpus=2)
@requires_deep_ep
@requires_deep_gemm
def test_ht_deepep_deepgemm_moe(
    mnk: tuple[int, int, int],
    num_experts: int,
    topk: int,
    world_dp_size: tuple[int, int],
    disable_deepgemm_ue8m0,
    workspace_init,
):
    """
    Tests for High-Throughput DeepEP + DeepGemm integration.
    """

    m, n, k = mnk
    set_random_seed(7)

    if topk > num_experts:
        pytest.skip(f"Skipping test: topk={topk} > E={num_experts}")

    block_m = get_mk_alignment_for_contiguous_layout()[0]
    block_size = [block_m, block_m]

    world_size, dp_size = world_dp_size
    config = TestConfig(
        topk=topk,
        m=m,
        k=k,
        n=n,
        num_experts=num_experts,
        per_act_token_quant=False,
        block_size=block_size,
        low_latency=False,
        use_fp8_dispatch=None,
    )

    w1, w2, w1_scale, w2_scale = make_block_quant_fp8_weights(
        num_experts, n, k, block_size
    )

    parallel_launch(
        world_size,
        _test_deepep_deepgemm_moe,
        dp_size,
        config,
        w1,
        w2,
        w1_scale,
        w2_scale,
    )
```
**EN:** This pytest case verifies ht deepep deepgemm MoE. It is parameterized across 4 input dimensions so one definition covers many scenarios. it consumes fixtures or inputs such as mnk, num_experts, topk, world_dp_size. unsupported hardware, backend, or configuration combinations are skipped early. randomness is controlled so the scenario remains reproducible.
**CN:** 该 pytest 用例验证 ht deepep deepgemm MoE 的行为。 它通过 4 组参数化输入覆盖多种场景；它会使用诸如 mnk、num_experts、topk、world_dp_size 等 fixture 或输入；不支持的硬件、后端或配置组合会被提前跳过；代码会控制随机性以保证场景可复现。

### Constants and module state (lines 500-510)
```python
MNKs = [
    (1, 128, 2560),
    (2, 128, 2560),
    (3, 1024, 2560),
    (32, 128, 2560),
    (45, 512, 2560),
    (64, 1024, 2560),
    (222, 1024, 2560),
]
# Fix tests for USE_FP8_DISPATCH=True
USE_FP8_DISPATCH = [False]
```
**EN:** This block centralizes shared constants and parameter grids, including USE_FP8_DISPATCH. Those values keep later pytest scenarios consistent and make the test matrix easier to audit.
**CN:** 该代码块集中定义共享常量与参数网格，例如 USE_FP8_DISPATCH。这些值让后续 pytest 场景保持一致，也便于检查测试矩阵是否完整。

### Function `test_ll_deepep_deepgemm_moe` (lines 513-569)
```python
@pytest.mark.parametrize("mnk", MNKs)
@pytest.mark.parametrize("num_experts", NUM_EXPERTS)
@pytest.mark.parametrize("topk", TOPKS)
@pytest.mark.parametrize("use_fp8_dispatch", USE_FP8_DISPATCH)
@pytest.mark.parametrize("block_size", [[128, 128]])
@pytest.mark.parametrize("world_dp_size", [(2, 1)])
@multi_gpu_test(num_gpus=2)
@requires_deep_ep
@requires_deep_gemm
def test_ll_deepep_deepgemm_moe(
    mnk: tuple[int, int, int],
    num_experts: int,
    topk: int,
    use_fp8_dispatch: bool,
    block_size: list[int],
    world_dp_size: tuple[int, int],
    disable_deepgemm_ue8m0,
    workspace_init,
):
    """
    Tests for Low-Latency DeepEP + DeepGemm integration.
    """
    assert not is_deep_gemm_e8m0_used()

    m, n, k = mnk
    set_random_seed(7)

    if topk > num_experts:
        pytest.skip(f"Skipping test: topk={topk} > E={num_experts}")

    world_size, dp_size = world_dp_size
    config = TestConfig(
        topk=topk,
        m=m,
        k=k,
        n=n,
        num_experts=num_experts,
        per_act_token_quant=False,
        block_size=block_size,
        low_latency=True,
        use_fp8_dispatch=use_fp8_dispatch,
    )

    w1, w2, w1_scale, w2_scale = make_block_quant_fp8_weights(
        num_experts, n, k, block_size
    )

    parallel_launch(
        world_size,
        _test_deepep_deepgemm_moe,
        dp_size,
        config,
        w1,
        w2,
        w1_scale,
        w2_scale,
    )
```
**EN:** This pytest case verifies ll deepep deepgemm MoE. It is parameterized across 6 input dimensions so one definition covers many scenarios. it consumes fixtures or inputs such as mnk, num_experts, topk, use_fp8_dispatch. unsupported hardware, backend, or configuration combinations are skipped early. assertions at the end lock in the intended behavior or graph shape.
**CN:** 该 pytest 用例验证 ll deepep deepgemm MoE 的行为。 它通过 6 组参数化输入覆盖多种场景；它会使用诸如 mnk、num_experts、topk、use_fp8_dispatch 等 fixture 或输入；不支持的硬件、后端或配置组合会被提前跳过；结尾处的断言会固定预期行为或计算图形态。

## Key Concepts / 关键概念
- **Pytest parameterization / pytest 参数化:** The module expands one definition into many concrete scenarios through parametrized inputs. / 该模块通过参数化输入把单个定义扩展成多组具体场景。
- **Behavioral assertions / 行为断言:** Assertions check tensor values, graph structure, or expected error conditions. / 断言用于检查张量结果、计算图结构或预期的报错路径。
- **Platform guards / 平台保护:** Hardware, backend, or version checks prevent unsupported combinations from running. / 硬件、后端或版本检查会阻止不受支持的组合继续执行。
- **Reference comparisons / 参考实现对照:** Optimized kernels are commonly checked against simpler reference implementations or fallback paths. / 优化后的内核通常会与更直接的参考实现或回退路径进行比对。

## Dependencies / 依赖关系
- `dataclasses`
- `contextlib -> contextmanager`
- `pytest`
- `torch.distributed`
- `torch.distributed -> ProcessGroup`
- `typing_extensions -> ParamSpec`
- `vllm.config -> VllmConfig, set_current_vllm_config`
- `vllm.forward_context -> set_forward_context`
- `vllm.model_executor.layers.fused_moe.activation -> MoEActivation`
- `vllm.model_executor.layers.fused_moe.config -> FusedMoEQuantConfig, fp8_w8a8_moe_quant_config`
- `vllm.model_executor.layers.fused_moe.fused_moe -> fused_experts`
- `vllm.model_executor.layers.fused_moe.modular_kernel -> FusedMoEKernel`
- `vllm.utils.deep_gemm -> get_mk_alignment_for_contiguous_layout, is_deep_gemm_e8m0_used, is_deep_gemm_supported`
- `vllm.utils.import_utils -> has_deep_ep, has_deep_gemm`
- `vllm.utils.math_utils -> next_power_of_2`
- `vllm.utils.torch_utils -> set_random_seed`
