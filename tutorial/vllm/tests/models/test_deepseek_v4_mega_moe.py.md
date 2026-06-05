# test_deepseek_v4_mega_moe.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/models/test_deepseek_v4_mega_moe.py`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Covers model-facing behavior. The file defines 4 test(s), 0 fixture(s), and 0 helper/class block(s) to validate this area. / [CN] 该文件覆盖面向模型的行为。它定义了 4 个测试、0 个 fixture，以及 0 个辅助函数/类块，用于验证这一领域。

## Line-by-Line Analysis / 逐行分析
### Imports and setup / 导入与初始化: module imports (L4-L14)
```python
from types import SimpleNamespace

import pytest
import torch

from vllm.model_executor.models.deepseek_v4 import (
    DeepseekV4MegaMoEExperts,
    _stage_deepseek_v4_mega_moe_inputs,
    make_deepseek_v4_expert_params_mapping,
)
from vllm.platforms import current_platform
```
**EN:** Imports standard-library modules such as `types.SimpleNamespace`, third-party packages like `pytest`, `torch`, project helpers such as `vllm.model_executor.models.deepseek_v4.DeepseekV4MegaMoEExperts`, `vllm.model_executor.models.deepseek_v4._stage_deepseek_v4_mega_moe_inputs`, `vllm.model_executor.models.deepseek_v4.make_deepseek_v4_expert_params_mapping`.
**CN:** 导入标准库模块（如 `types.SimpleNamespace`）、第三方包（如 `pytest`、`torch`）、项目内辅助模块（如 `vllm.model_executor.models.deepseek_v4.DeepseekV4MegaMoEExperts`、`vllm.model_executor.models.deepseek_v4._stage_deepseek_v4_mega_moe_inputs`、`vllm.model_executor.models.deepseek_v4.make_deepseek_v4_expert_params_mapping`）。

### Module setup / 模块级配置: pytestmark (L16-L19)
```python
pytestmark = pytest.mark.skipif(
    not current_platform.is_cuda(),
    reason="DeepSeek V4 MegaMoE requires CUDA",
)
```
**EN:** This block defines shared constants, registries, or module-level configuration such as `pytestmark`. Later helpers and tests reuse these values to keep scenarios concise. It also applies module-wide pytest markers.
**CN:** 该代码块定义共享常量、注册表或模块级配置，例如 `pytestmark`。后续辅助函数和测试会复用这些值，以减少重复。 它还会应用模块级的 pytest 标记。

### Test / 测试: test_deepseek_v4_mega_moe_expert_mapping (L22-L32)
```python
def test_deepseek_v4_mega_moe_expert_mapping():
    mapping = make_deepseek_v4_expert_params_mapping(2)

    assert mapping == [
        ("experts.w13_", "experts.0.w1.", 0, "w1"),
        ("experts.w2_", "experts.0.w2.", 0, "w2"),
        ("experts.w13_", "experts.0.w3.", 0, "w3"),
        ("experts.w13_", "experts.1.w1.", 1, "w1"),
        ("experts.w2_", "experts.1.w2.", 1, "w2"),
        ("experts.w13_", "experts.1.w3.", 1, "w3"),
    ]
```
**EN:** This test validates `test_deepseek_v4_mega_moe_expert_mapping`. The main assertion is `mapping == [('experts.w13_', 'experts.0.w1.', 0, 'w1'), ('experts.w2_', 'experts.0.w2.', 0, 'w2'), ('experts.w13_', 'experts.0.w3.', 0, '...`.
**CN:** 这个测试验证 `test_deepseek_v4_mega_moe_expert_mapping`。 核心断言是 `mapping == [('experts.w13_', 'experts.0.w1.', 0, 'w1'), ('experts.w2_', 'experts.0.w2.', 0, 'w2'), ('experts.w13_', 'experts.0.w3.', 0, '...`。

### Test / 测试: test_deepseek_v4_mega_moe_ue8m0_uint8_to_float (L35-L44)
```python
def test_deepseek_v4_mega_moe_ue8m0_uint8_to_float():
    raw = torch.tensor([0, 126, 127, 128], dtype=torch.uint8)

    decoded = DeepseekV4MegaMoEExperts._ue8m0_uint8_to_float(raw)

    assert torch.equal(decoded.view(torch.int32), raw.to(torch.int32) << 23)
    assert decoded[0].item() == 0.0
    assert decoded[1].item() == 0.5
    assert decoded[2].item() == 1.0
    assert decoded[3].item() == 2.0
```
**EN:** This test validates `test_deepseek_v4_mega_moe_ue8m0_uint8_to_float`. The main assertion is `torch.equal(decoded.view(torch.int32), raw.to(torch.int32) << 23)` and `decoded[0].item() == 0.0`.
**CN:** 这个测试验证 `test_deepseek_v4_mega_moe_ue8m0_uint8_to_float`。 核心断言是 `torch.equal(decoded.view(torch.int32), raw.to(torch.int32) << 23)` and `decoded[0].item() == 0.0`。

### Test / 测试: test_deepseek_v4_mega_moe_weight_loader_uses_ep_expert_ownership (L47-L106)
```python
def test_deepseek_v4_mega_moe_weight_loader_uses_ep_expert_ownership():
    vllm_config = SimpleNamespace(
        scheduler_config=SimpleNamespace(max_num_batched_tokens=4)
    )
    experts = DeepseekV4MegaMoEExperts(
        vllm_config,
        num_experts=4,
        num_local_experts=2,
        experts_start_idx=2,
        top_k=2,
        hidden_size=128,
        intermediate_size=128,
    )

    nonlocal_weight = torch.ones(128, 64, dtype=torch.uint8)
    assert (
        experts.weight_loader(
            experts.w13_weight,
# ... 34 lines omitted for brevity ...
        expert_id=2,
        return_success=True,
    )

    assert torch.equal(experts.w13_weight[0, :128], w1)
    assert torch.equal(experts.w13_weight[0, 128:], w3)
    assert torch.equal(experts.w2_weight[0], w2)
    assert torch.count_nonzero(experts.w13_weight[1]) == 0
```
**EN:** This test validates `test_deepseek_v4_mega_moe_weight_loader_uses_ep_expert_ownership`. The main assertion is `experts.weight_loader(experts.w13_weight, nonlocal_weight, 'experts.w13_weight', shard_id='w1', expert_id=1, return_success=True) is False` and `experts.weight_loader(experts.w13_weight, w1, 'experts.w13_weight', shard_id='w1', expert_id=2, return_success=True)`.
**CN:** 这个测试验证 `test_deepseek_v4_mega_moe_weight_loader_uses_ep_expert_ownership`。 核心断言是 `experts.weight_loader(experts.w13_weight, nonlocal_weight, 'experts.w13_weight', shard_id='w1', expert_id=1, return_success=True) is False` and `experts.weight_loader(experts.w13_weight, w1, 'experts.w13_weight', shard_id='w1', expert_id=2, return_success=True)`。

### Test / 测试: test_deepseek_v4_mega_moe_fused_input_staging_is_bitwise_exact (L109-L184)
```python
@pytest.mark.skipif(
    not torch.cuda.is_available(),
    reason="DeepSeek V4 MegaMoE fused input staging requires CUDA.",
)
def test_deepseek_v4_mega_moe_fused_input_staging_is_bitwise_exact():
    from vllm.third_party.deep_gemm.utils import per_token_cast_to_fp8

    device = torch.device("cuda")
    num_tokens = 7
    hidden_size = 256
    top_k = 8

    generator = torch.Generator(device=device)
    generator.manual_seed(0)
    hidden_states = (
        torch.randn(
            num_tokens,
            hidden_size,
# ... 50 lines omitted for brevity ...

    assert torch.equal(fused_x.view(torch.uint8), ref_x.view(torch.uint8))
    assert torch.equal(fused_x_sf, ref_x_sf)
    assert torch.equal(fused_topk_idx, ref_topk_idx)
    assert torch.equal(
        fused_topk_weights.view(torch.uint8),
        ref_topk_weights.view(torch.uint8),
    )
```
**EN:** This test validates `test_deepseek_v4_mega_moe_fused_input_staging_is_bitwise_exact`. Relevant pytest markers include `skipif`. The main assertion is `torch.equal(fused_x.view(torch.uint8), ref_x.view(torch.uint8))` and `torch.equal(fused_x_sf, ref_x_sf)`.
**CN:** 这个测试验证 `test_deepseek_v4_mega_moe_fused_input_staging_is_bitwise_exact`。 相关的 pytest 标记包括 `skipif`。 核心断言是 `torch.equal(fused_x.view(torch.uint8), ref_x.view(torch.uint8))` and `torch.equal(fused_x_sf, ref_x_sf)`。

## Key Concepts / 关键概念
- **EN:** The checks cover model loading, execution, registry data, or backend compatibility.
  **CN:** 这些检查覆盖模型加载、执行、注册表数据或后端兼容性。
- **EN:** Assertions encode the acceptance criteria for each scenario.
  **CN:** 断言定义了每个场景的验收标准。

## Dependencies / 依赖关系
- **Stdlib / 标准库**: `types.SimpleNamespace`
- **Third-party / 第三方**: `pytest`, `torch`
- **Project / 项目内**: `vllm.model_executor.models.deepseek_v4.DeepseekV4MegaMoEExperts`, `vllm.model_executor.models.deepseek_v4._stage_deepseek_v4_mega_moe_inputs`, `vllm.model_executor.models.deepseek_v4.make_deepseek_v4_expert_params_mapping`, `vllm.platforms.current_platform`, `vllm.third_party.deep_gemm.utils.per_token_cast_to_fp8`
