# test_cutlass_w16a16_moe.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/manual/quant/test_cutlass_w16a16_moe.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This manual test module covers the `cutlass w16a16 moe` scenario in `test/manual/quant`. It uses SGLang's shared test infrastructure to configure models or services and verify expected accuracy and functional behavior. / 该手动测试模块覆盖 `test/manual/quant` 中的 `cutlass w16a16 moe` 场景。它利用 SGLang 的共享测试基础设施配置模型或服务，并验证预期的精度与功能表现。

## Line-by-Line Analysis / 逐行分析
### Lines 1-24: Constants and scenario settings / 常量与场景配置
```python
# SPDX-License-Identifier: Apache-2.0
import pytest
import torch
from flashinfer.fused_moe import cutlass_fused_moe as flashinfer_cutlass_fused_moe

from sglang.srt.layers.activation import SiluAndMul
from sglang.srt.layers.moe.topk import TopKConfig, select_experts
from sglang.srt.server_args import ServerArgs, set_global_server_args_for_scheduler

MNK_FACTORS = [
    (2, 1024, 1024),
    (2, 1024, 1536),
    (2, 3072, 1024),
    (2, 3072, 1536),
    (64, 1024, 1024),
    (64, 1024, 1536),
    (64, 3072, 1024),
    (64, 2048, 1024),
    (224, 1024, 1024),
    (224, 1024, 1536),
]


# Reference implementation of torch_moe for unquantized weights
```
**EN:** This range defines constants and per-scenario settings that parameterize the later execution path.
**CN:** 这一部分定义常量和场景参数，为后续执行路径提供配置。

### Lines 25-49: Helper routines around torch_moe_reference / 辅助例程
```python
def torch_moe_reference(a, w13, w2, score, topk):
    B, D = a.shape

    set_global_server_args_for_scheduler(ServerArgs(model_path="dummy"))

    # Flip w13 layout
    dim = -2
    size = w13.size(dim)
    assert size % 2 == 0, f"Expected even size in dim {dim}, got {size}"
    half = size // 2
    # Reorder weight
    w1, w3 = w13.split(half, dim=dim)
    w13 = torch.cat([w3, w1], dim=dim).contiguous()

    a = a.view(B, -1, D).repeat(1, topk, 1).reshape(-1, D)
    out = torch.zeros(B * topk, w2.shape[1], dtype=a.dtype, device=a.device)
    score = torch.softmax(score, dim=-1, dtype=torch.float32)
    topk_weight, topk_ids = torch.topk(score, topk)
    topk_weight = topk_weight.view(-1)
    topk_ids = topk_ids.view(-1)

    for i in range(w13.shape[0]):
        mask = topk_ids == i
        if mask.sum():
            out[mask] = SiluAndMul()(a[mask] @ w13[i].transpose(0, 1)) @ w2[
```
**EN:** This range implements helper routine(s) `torch_moe_reference` so setup, transformation, or validation logic can be reused cleanly. Assertions in this block enforce the intended outcome. Representative call sites include `set_global_server_args_for_scheduler`, `ServerArgs`, `size` and `split`.
**CN:** 这一部分实现辅助例程，使初始化、转换或校验逻辑能够被整洁地复用。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 50-55: Scenario logic / 场景逻辑
```python
                i
            ].transpose(0, 1)

    return (
        out.view(B, -1, w2.shape[1]) * topk_weight.view(B, -1, 1).to(out.dtype)
    ).sum(dim=1)
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `transpose`, `view` and `to`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 56-61: Scenario logic / 场景逻辑
```python


@pytest.mark.parametrize("m,n,k", MNK_FACTORS)
@pytest.mark.parametrize("e", [40, 64, 256])
@pytest.mark.parametrize("topk", [1, 6, 8])
@torch.inference_mode()
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `parametrize` and `inference_mode`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 62-86: Test routines around test_flashinfer_bf16_cutlass_moe / 测试例程
```python
def test_flashinfer_bf16_cutlass_moe(m: int, n: int, k: int, e: int, topk: int):
    """
    Test the bf16 cutlass moe API.

    Args:
        m: number of tokens
        n: intermediate size
        k: hidden size
        e: number of experts
        topk: top-k experts per token
    """
    torch.manual_seed(7)

    dtype = torch.bfloat16

    # Create unquantized weights
    a = torch.randn((m, k), device="cuda", dtype=dtype) / 10

    # w13: fused gate_up projection [num_experts, 2*intermediate, hidden]
    # FlashInfer CUTLASS expects [up, gate] layout
    w13 = torch.randn((e, 2 * n, k), device="cuda", dtype=dtype) / 10

    # w2: down projection [num_experts, hidden, intermediate]
    w2 = torch.randn((e, k, n), device="cuda", dtype=dtype) / 10
```
**EN:** This range defines concrete test routine(s) `test_flashinfer_bf16_cutlass_moe`. The logic drives the target scenario and encodes the expected acceptance criteria. Representative call sites include `manual_seed` and `randn`.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 87-111: Scenario logic / 场景逻辑
```python
    # Generate router scores
    score = torch.randn((m, e), device="cuda", dtype=dtype)

    # Get topk routing
    topk_output = select_experts(
        hidden_states=a,
        router_logits=score,
        topk_config=TopKConfig(top_k=topk, renormalize=False),
    )
    topk_weights, topk_ids, _ = topk_output

    # Test: Call FlashInfer CUTLASS fused_moe (unquantized version)
    test_output = flashinfer_cutlass_fused_moe(
        input=a,
        token_selected_experts=topk_ids,
        token_final_scales=topk_weights,
        fc1_expert_weights=w13,
        fc2_expert_weights=w2,
        output_dtype=dtype,
        quant_scales=None,
    )[0]

    # Reference: Torch implementation
    torch_output = torch_moe_reference(a, w13, w2, score, topk)
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `randn`, `select_experts`, `TopKConfig` and `fused_moe`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 112-113: Assertions and result checks / 断言与结果检查
```python
    # Compare outputs
    torch.testing.assert_close(torch_output, test_output, rtol=1e-2, atol=1e-2)
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Assertions in this block enforce the intended outcome. Representative call sites include `assert_close`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 114-118: Script entry point / 脚本入口
```python


if __name__ == "__main__":
    # Run a simple test case
    test_flashinfer_bf16_cutlass_moe(224, 1024, 1024, 8, 2)
```
**EN:** This range exposes the module as a directly runnable script, usually by delegating to a test runner or helper entry point. Representative call sites include `test_flashinfer_bf16_cutlass_moe`.
**CN:** 这一部分把模块暴露为可直接运行的脚本，通常会委托给测试运行器或辅助入口。 其中还会调用若干代表性的函数来串联完整流程。

## Key Concepts / 关键概念
- Pytest markers / Pytest 标记
- Accuracy evaluation / 精度评测
- Multi-GPU orchestration / 多 GPU 编排
- Quantization configuration / 量化配置
- Token-level inspection / Token 级分析
- Disaggregated serving / 解耦式服务

## Dependencies / 依赖关系
- **Standard Library / 标准库**: None / 无
- **Third-party / 第三方库**: `flashinfer.fused_moe`, `pytest`, `torch`
- **Project Modules / 项目模块**: `sglang.srt.layers.activation`, `sglang.srt.layers.moe.topk`, `sglang.srt.server_args`
