# test_cutlass_w4a8_moe.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/manual/quant/test_cutlass_w4a8_moe.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This manual test module covers the `cutlass w4a8 moe` scenario in `test/manual/quant`. It uses SGLang's shared test infrastructure to configure models or services and verify expected accuracy and functional behavior. / 该手动测试模块覆盖 `test/manual/quant` 中的 `cutlass w4a8 moe` 场景。它利用 SGLang 的共享测试基础设施配置模型或服务，并验证预期的精度与功能表现。

## Line-by-Line Analysis / 逐行分析
### Lines 1-11: Imports and shared helpers / 导入与共享辅助项
```python
# SPDX-License-Identifier: Apache-2.0

from typing import Optional

import pytest
import torch

from sglang.srt.layers.moe.cutlass_w4a8_moe import cutlass_w4a8_moe
from sglang.srt.layers.moe.topk import TopKConfig, select_experts
```
**EN:** This range imports `typing`, `pytest`, `torch` and `sglang.srt.layers.moe.cutlass_w4a8_moe`. These dependencies provide the test harness, runtime clients, and reusable SGLang helpers used later in the file.
**CN:** 这一部分导入所需模块，为后续测试框架、运行时客户端以及可复用的 SGLang 辅助逻辑提供依赖。

### Lines 12-25: Helper routines around pack_int4_values_to_int8 / 辅助例程
```python
def pack_int4_values_to_int8(int4_values_interleaved: torch.Tensor) -> torch.Tensor:
    if int4_values_interleaved.shape[-1] % 2 != 0:
        raise ValueError(
            "the last dim size of int4_values_interleaved tensor must be even."
        )

    input_tensor_int8 = int4_values_interleaved.to(torch.int8)

    low_nibbles = input_tensor_int8[..., 0::2]
    high_nibbles = input_tensor_int8[..., 1::2]

    packed_tensor = (high_nibbles << 4) | (low_nibbles & 0x0F)

    return packed_tensor.to(torch.int8)
```
**EN:** This range implements helper routine(s) `pack_int4_values_to_int8` so setup, transformation, or validation logic can be reused cleanly. Representative call sites include `ValueError` and `to`.
**CN:** 这一部分实现辅助例程，使初始化、转换或校验逻辑能够被整洁地复用。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 28-49: Helper routines around pack_interleave / 辅助例程
```python
def pack_interleave(num_experts, ref_weight, ref_scale, alignment=4):
    n, k = ref_weight.shape[1], ref_weight.shape[2]

    weight = pack_int4_values_to_int8(ref_weight.cpu()).cuda()
    w_q = weight.view((num_experts, n, k // 2)).view(torch.int8)
    w_q = w_q.contiguous()

    scale_interleaved = ref_scale.reshape(
        ref_scale.shape[0],
        ref_scale.shape[1],
        (ref_scale.shape[2] // alignment),
        alignment,
    )  # [E, N, K/4, 4]
    scale_interleaved = scale_interleaved.permute(0, 2, 1, 3)  # [E, K/4, N, 4]
    scale_interleaved = scale_interleaved.reshape(
        ref_scale.shape[0],
        ref_scale.shape[2] // alignment,
        ref_scale.shape[1] * alignment,
    )  # [E, K/4, N*4]
    w_scale = scale_interleaved.contiguous()

    return w_q, w_scale
```
**EN:** This range implements helper routine(s) `pack_interleave` so setup, transformation, or validation logic can be reused cleanly. Representative call sites include `pack_int4_values_to_int8`, `cpu`, `cuda` and `view`.
**CN:** 这一部分实现辅助例程，使初始化、转换或校验逻辑能够被整洁地复用。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 50-60: Scenario logic / 场景逻辑
```python


@pytest.mark.parametrize("M", [1, 2, 4, 8, 16])
@pytest.mark.parametrize("N", [2048])
@pytest.mark.parametrize("K", [7168])
@pytest.mark.parametrize("E", [256])
@pytest.mark.parametrize("tp_size", [8])
@pytest.mark.parametrize("use_ep_moe", [True, False])
@pytest.mark.parametrize("topk", [8])
@pytest.mark.parametrize("group_size", [128])
@pytest.mark.parametrize("dtype", [torch.bfloat16])
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `parametrize`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 61-85: Test routines around test_cutlass_w4a8_moe / 测试例程
```python
def test_cutlass_w4a8_moe(M, N, K, E, tp_size, use_ep_moe, topk, group_size, dtype):
    if use_ep_moe:
        local_e = E // tp_size
    else:  # tp mode
        local_e = E
        N = N // tp_size

    debug = False
    if debug:
        a = torch.ones((M, K), dtype=dtype, device="cuda") * 0.001
        ref_weight_1 = torch.ones((local_e, N * 2, K), dtype=torch.int8, device="cuda")
        ref_weight_2 = torch.ones((local_e, K, N), dtype=torch.int8, device="cuda")
        a1_scale = torch.ones(1, dtype=torch.float32, device="cuda")
        a2_scale = torch.ones(1, dtype=torch.float32, device="cuda")
        scale_1 = torch.ones(
            (local_e, N * 2, K // group_size), dtype=dtype, device="cuda"
        )
        scale_2 = torch.ones((local_e, K, N // group_size), dtype=dtype, device="cuda")
    else:
        a = torch.randn(M, K, dtype=dtype, device="cuda")
        ref_weight_1 = torch.randint(
            -8, 8, (local_e, N * 2, K), dtype=torch.int8, device="cuda"
        )
        ref_weight_2 = torch.randint(
            -8, 8, (local_e, K, N), dtype=torch.int8, device="cuda"
```
**EN:** This range defines concrete test routine(s) `test_cutlass_w4a8_moe`. The logic drives the target scenario and encodes the expected acceptance criteria. Representative call sites include `ones`, `randn` and `randint`.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 86-110: Scenario logic / 场景逻辑
```python
        )
        affine_coeff = 0.005
        a1_scale = torch.randn(1, dtype=torch.float32, device="cuda")
        a2_scale = torch.randn(1, dtype=torch.float32, device="cuda")
        scale_1 = (
            torch.randn(local_e, N * 2, K // group_size, dtype=dtype, device="cuda")
            * affine_coeff
        )
        scale_2 = (
            torch.randn(local_e, K, N // group_size, dtype=dtype, device="cuda")
            * affine_coeff
        )

    w1_q, w1_scale = pack_interleave(local_e, ref_weight_1, scale_1)
    if use_ep_moe:
        w2_q, w2_scale = pack_interleave(local_e, ref_weight_2, scale_2)
    else:
        w2_q, w2_scale = pack_interleave(local_e, ref_weight_2, scale_2, 1)

    device = "cuda"
    a_strides1 = torch.full((local_e, 3), K, device=device, dtype=torch.int64)
    c_strides1 = torch.full((local_e, 3), 2 * N, device=device, dtype=torch.int64)
    a_strides2 = torch.full((local_e, 3), N, device=device, dtype=torch.int64)
    c_strides2 = torch.full((local_e, 3), K, device=device, dtype=torch.int64)
    b_strides1 = a_strides1
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `randn`, `pack_interleave` and `full`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 111-135: Scenario logic / 场景逻辑
```python
    s_strides13 = c_strides1
    b_strides2 = a_strides2
    s_strides2 = c_strides2

    score = torch.randn((M, E), dtype=dtype, device=device)
    topk_output = select_experts(
        hidden_states=a,
        router_logits=score,
        topk_config=TopKConfig(top_k=topk, renormalize=False),
    )
    topk_weights, topk_ids, _ = topk_output
    expert_map = torch.arange(E, dtype=torch.int32, device=device)
    expert_map[local_e:] = -1

    output = cutlass_moe(
        a,
        w1_q,
        w2_q,
        w1_scale,
        w2_scale,
        topk_weights,
        topk_ids,
        a_strides1,
        b_strides1,
        c_strides1,
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `randn`, `select_experts`, `TopKConfig` and `arange`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 136-160: Scenario logic / 场景逻辑
```python
        a_strides2,
        b_strides2,
        c_strides2,
        s_strides13,
        s_strides2,
        local_e,
        a1_scale,
        a2_scale,
        expert_map,
    )

    ref_output = ref(
        a,
        local_e,
        topk_weights,
        topk_ids,
        ref_weight_1,
        ref_weight_2,
        scale_1,
        scale_2,
        has_pre_quant=True,
        has_alpha=True,
        pre_quant_scale_1=a1_scale,
        pre_quant_scale_2=a2_scale,
        alpha_1=a1_scale,
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `ref`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 161-169: Assertions and result checks / 断言与结果检查
```python
        alpha_2=a2_scale,
    )

    # compare
    torch.cuda.synchronize()

    # compare final output
    torch.testing.assert_close(output, ref_output, rtol=1e-2, atol=0.1)
    print("SUCCESS: Final output tensors are close.")
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Assertions in this block enforce the intended outcome. Representative call sites include `synchronize` and `assert_close`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 172-196: Helper routines around cutlass_moe / 辅助例程
```python
def cutlass_moe(
    a: torch.Tensor,
    w1_q: torch.Tensor,
    w2_q: torch.Tensor,
    w1_scale: torch.Tensor,
    w2_scale: torch.Tensor,
    topk_weights: torch.Tensor,
    topk_ids: torch.Tensor,
    a_strides1: torch.Tensor,
    b_strides1: torch.Tensor,
    c_strides1: torch.Tensor,
    a_strides2: torch.Tensor,
    b_strides2: torch.Tensor,
    c_strides2: torch.Tensor,
    s_strides13: torch.Tensor,
    s_strides2: torch.Tensor,
    num_local_experts: int,
    a1_scale: Optional[torch.Tensor] = None,
    a2_scale: Optional[torch.Tensor] = None,
    expert_map: Optional[torch.Tensor] = None,
    apply_router_weight_on_input: bool = False,
):
    topk_ids = expert_map[topk_ids]
    device = a.device
```
**EN:** This range implements helper routine(s) `cutlass_moe` so setup, transformation, or validation logic can be reused cleanly.
**CN:** 这一部分实现辅助例程，使初始化、转换或校验逻辑能够被整洁地复用。

### Lines 197-221: Scenario logic / 场景逻辑
```python
    expert_offsets = torch.empty(
        (num_local_experts + 1), dtype=torch.int32, device=device
    )
    problem_sizes1 = torch.empty(
        (num_local_experts, 3), dtype=torch.int32, device=device
    )
    problem_sizes2 = torch.empty(
        (num_local_experts, 3), dtype=torch.int32, device=device
    )
    return cutlass_w4a8_moe(
        a,
        w1_q,
        w2_q,
        w1_scale,
        w2_scale,
        topk_weights,
        topk_ids,
        a_strides1,
        b_strides1,
        c_strides1,
        a_strides2,
        b_strides2,
        c_strides2,
        s_strides13,
        s_strides2,
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `empty` and `cutlass_w4a8_moe`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 222-228: Scenario logic / 场景逻辑
```python
        expert_offsets,
        problem_sizes1,
        problem_sizes2,
        a1_scale,
        a2_scale,
        apply_router_weight_on_input,
    )
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 231-255: Helper routines around ref / 辅助例程
```python
def ref(
    x: torch.Tensor,
    num_experts: int,
    topk_weights: torch.Tensor,
    topk_ids: torch.Tensor,
    ref_weight_1: torch.Tensor,
    ref_weight_2: torch.Tensor,
    ref_weight_scale_1: torch.Tensor,
    ref_weight_scale_2: torch.Tensor,
    has_pre_quant: bool = False,
    has_alpha: bool = False,
    pre_quant_scale_1: Optional[torch.Tensor] = None,
    pre_quant_scale_2: Optional[torch.Tensor] = None,
    alpha_1: Optional[torch.Tensor] = None,
    alpha_2: Optional[torch.Tensor] = None,
):
    results = torch.zeros_like(x)
    dtype = x.dtype
    for e_idx in range(num_experts):
        mask = topk_ids == e_idx
        activated_tokens = mask.sum(1).bool()
        act = x[activated_tokens, :]
        if act.shape[0] == 0:
            continue
        final_scale = (topk_weights * mask).sum(1)[activated_tokens].unsqueeze(1)
```
**EN:** This range implements helper routine(s) `ref` so setup, transformation, or validation logic can be reused cleanly. Representative call sites include `zeros_like`, `bool` and `unsqueeze`.
**CN:** 这一部分实现辅助例程，使初始化、转换或校验逻辑能够被整洁地复用。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 256-280: Scenario logic / 场景逻辑
```python

        act = (
            torch.clamp((act / pre_quant_scale_1.float()), -448.0, 448.0)
            .to(torch.float8_e4m3fn)
            .to(dtype)
        )
        w3_w1 = ref_weight_1[e_idx]
        ref_w_scale_repeat = (
            ref_weight_scale_1[e_idx].repeat_interleave(128, dim=1).to(float)
        )
        w3_w1 = (w3_w1.to(float) * ref_w_scale_repeat).to(dtype)
        fc1 = ((torch.matmul(act, w3_w1.T)) * alpha_1).to(torch.float16)

        gate, fc1 = fc1.chunk(2, dim=-1)
        fc1 = fc1 * torch.nn.functional.silu(gate)
        act = torch.clamp((fc1 / pre_quant_scale_2.float()), -448.0, 448.0).to(
            torch.float8_e4m3fn
        )
        act = act.to(dtype)

        w2 = ref_weight_2[e_idx]
        ref_w_scale_repeat = (
            ref_weight_scale_2[e_idx].repeat_interleave(128, dim=1).to(float)
        )
        w2 = (w2.to(float) * ref_w_scale_repeat).to(dtype)
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `clamp`, `to`, `repeat_interleave` and `matmul`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 281-285: Scenario logic / 场景逻辑
```python
        fc2 = (torch.matmul(act, w2.T) * alpha_2).to(torch.float16)

        results[activated_tokens, :] += (fc2 * final_scale).to(results.dtype)

    return results
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `matmul` and `to`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

## Key Concepts / 关键概念
- Pytest markers / Pytest 标记
- Accuracy evaluation / 精度评测
- Multi-GPU orchestration / 多 GPU 编排
- Quantization configuration / 量化配置
- Token-level inspection / Token 级分析
- Disaggregated serving / 解耦式服务

## Dependencies / 依赖关系
- **Standard Library / 标准库**: `typing`
- **Third-party / 第三方库**: `pytest`, `torch`
- **Project Modules / 项目模块**: `sglang.srt.layers.moe.cutlass_w4a8_moe`, `sglang.srt.layers.moe.topk`
