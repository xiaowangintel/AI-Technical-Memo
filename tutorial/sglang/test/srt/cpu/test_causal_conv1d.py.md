# test_causal_conv1d.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/srt/cpu/test_causal_conv1d.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates causal conv1d behavior in SGLang's srt / cpu area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 SRT / CPU 领域中与 causal conv1d 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-9: module imports and dependencies / 模块导入与依赖
```python
import unittest
from typing import Optional

import sgl_kernel  # noqa: F401
import torch
import torch.nn.functional as F
from utils import parametrize, precision

from sglang.test.test_utils import CustomTestCase
```
**EN:** This block imports the modules needed by the rest of the file, including `unittest`, `typing`, `sgl_kernel`, `torch`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `unittest`, `typing`, `sgl_kernel`, `torch`。

### Lines 11-18: module-level constants and configuration / 模块级常量与配置
```python
causal_conv1d_weight_pack = torch.ops.sgl_kernel.causal_conv1d_weight_pack
causal_conv1d_fwd = torch.ops.sgl_kernel.causal_conv1d_fwd_cpu
causal_conv1d_update = torch.ops.sgl_kernel.causal_conv1d_update_cpu


torch.manual_seed(1234)

PAD_SLOT_ID = -1
```
**EN:** This block defines shared names such as `causal_conv1d_weight_pack`, `causal_conv1d_fwd`, `causal_conv1d_update`, `PAD_SLOT_ID`. These values centralize configuration that later tests or helpers reuse.
**CN:** 该代码块定义了 `causal_conv1d_weight_pack`, `causal_conv1d_fwd`, `causal_conv1d_update`, `PAD_SLOT_ID` 等共享名称，用于集中保存后续测试或辅助逻辑会复用的配置。

### Lines 21-60: function causal conv1d ref / 函数 causal conv1d ref
```python
def causal_conv1d_ref(
    x: torch.Tensor,
    weight: torch.Tensor,
    bias: Optional[torch.Tensor] = None,
    initial_states: Optional[torch.Tensor] = None,
    return_final_states: bool = False,
    final_states_out: Optional[torch.Tensor] = None,
    activation: Optional[str] = "silu",
):
    """
    x: (batch, dim, seqlen)
    weight: (dim, width)
    bias: (dim,)
    initial_states: (batch, dim, width - 1)
    final_states_out: (batch, dim, width - 1)

    out: (batch, dim, seqlen)
    """
    if activation not in [None, "silu", "swish"]:
        raise NotImplementedError("activation must be None, silu, or swish")
    dtype_in = x.dtype
    x = x.to(weight.dtype)
    seqlen = x.shape[-1]
    dim, width = weight.shape
    if initial_states is None:
        out = F.conv1d(x, weight.unsqueeze(1), bias, padding=width - 1, groups=dim)
    else:
        x = torch.cat([initial_states, x], dim=-1)
        out = F.conv1d(x, weight.unsqueeze(1), bias, padding=0, groups=dim)
    out = out[..., :seqlen]
    if return_final_states:
        final_states = F.pad(x, (width - 1 - x.shape[-1], 0)).to(
            dtype_in
        )  # (batch, dim, width - 1)
        if final_states_out is not None:
            final_states_out.copy_(final_states)
        else:
            final_states_out = final_states
    out = (out if activation is None else F.silu(out)).to(dtype=dtype_in)
    return (out, None) if not return_final_states else (out, final_states_out)
```
**EN:** x: (batch, dim, seqlen) weight: (dim, width) bias: (dim,) initial_states: (batch, dim, width - 1) final_states_out: (batch, dim, width - 1) out: (batch, dim, seqlen) This block implements `causal_conv1d_ref` and captures one focused piece of the module's behavior.
**CN:** x: (batch, dim, seqlen) weight: (dim, width) bias: (dim,) initial_states: (batch, dim, width - 1) final_states_out: (batch, dim, width - 1) out: (batch, dim, seqlen) 该代码块实现 `causal_conv1d_ref`，承担模块行为中的一个聚焦逻辑片段。

### Lines 63-94: function causal conv1d update ref / 函数 causal conv1d update ref
```python
def causal_conv1d_update_ref(
    x, conv_state, weight, bias=None, activation=None, cache_seqlens=None
):
    """
    x: (batch, dim) or (batch, dim, seqlen)
    conv_state: (batch, dim, state_len), where state_len >= width - 1
    weight: (dim, width)
    bias: (dim,)
    cache_seqlens: (batch,), dtype int32.
        If not None, the conv_state is treated as a circular buffer.
        The conv_state will be updated by copying x to the
        conv_state starting at the index
        @cache_seqlens % state_len before performing the convolution.

    out: (batch, dim) or (batch, dim, seqlen)
    """
    if activation not in [None, "silu", "swish"]:
        raise NotImplementedError("activation must be None, silu, or swish")

    x = x.unsqueeze(-1)
    batch, dim, seqlen = x.shape
    width = weight.shape[1]
    state_len = conv_state.shape[-1]

    x_new = torch.cat([conv_state, x], dim=-1)
    conv_state.copy_(x_new[:, :, -state_len:])
    out = F.conv1d(x_new, weight.unsqueeze(1), bias, padding=0, groups=dim)[
        :, :, -seqlen:
    ]

    out = out.squeeze(-1)
    return out if activation is None else F.silu(out)
```
**EN:** x: (batch, dim) or (batch, dim, seqlen) conv_state: (batch, dim, state_len), where state_len >= width - 1 weight: (dim, width) bias: (dim,) cache_seqlens: (batch,), dtype int32. This block implements `causal_conv1d_update_ref` and captures one focused piece of the module's behavior.
**CN:** x: (batch, dim) or (batch, dim, seqlen) conv_state: (batch, dim, state_len), where state_len >= width - 1 weight: (dim, width) bias: (dim,) cache_seqlens: (batch,), dtype int32. 该代码块实现 `causal_conv1d_update_ref`，承担模块行为中的一个聚焦逻辑片段。

### Lines 97-97: class TestCausalConv1d declaration / 类 TestCausalConv1d 声明
```python
class TestCausalConv1d(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 98-98: class-level constants and configuration for `TestCausalConv1d` / 类级常量与配置
```python
    activation = "silu"
```
**EN:** This block defines shared names such as `activation`. These values centralize configuration that later tests or helpers reuse.
**CN:** 该代码块定义了 `activation` 等共享名称，用于集中保存后续测试或辅助逻辑会复用的配置。

### Lines 100-158: test case causal conv1d / 测试用例 causal conv1d
```python
    @parametrize(
        batch=[1, 1024],
        dim=[96, 512],
        seqlen=[2, 36],
        width=[4],
        has_bias=[True, False],
        has_initial_state=[True, False],
    )
    def test_causal_conv1d(
        self,
        batch,
        dim,
        seqlen,
        width,
        has_bias,
        has_initial_state,
        dtype=torch.bfloat16,
        prepack=True,
    ):
        x = torch.randn(batch, seqlen, dim).to(dtype).transpose_(-1, -2)
        weight = torch.randn(dim, width).to(dtype)
        bias = torch.randn(dim).to(dtype) if has_bias else None

        if has_initial_state:
            initial_states = torch.randn(batch, dim, width - 1, dtype=dtype)
            has_initial_state_tensor = torch.ones(batch, dtype=torch.bool)
        else:
            initial_states = None
            has_initial_state_tensor = None

        packed_weight = causal_conv1d_weight_pack(weight) if prepack else weight

        out_ref, final_states_ref = causal_conv1d_ref(
            x,
            weight,
            bias,
            initial_states,
            return_final_states=has_initial_state,
            activation=self.activation,
        )

        out = causal_conv1d_fwd(
            x,
            packed_weight,
            bias,
            initial_states,
            None,
            None,
            has_initial_state_tensor,
            self.activation in ["silu"],
            PAD_SLOT_ID,
            prepack,
        )

        atol = rtol = precision[dtype]
        torch.testing.assert_close(out_ref, out, atol=atol, rtol=rtol)
        torch.testing.assert_close(
            final_states_ref, initial_states, atol=atol, rtol=rtol
        )
```
**EN:** This test exercises `test_causal_conv1d` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_causal_conv1d`。

### Lines 160-239: test case causal conv1d varlen (part 1/2) / 测试用例 causal conv1d varlen（第 1/2 部分）
```python
    @parametrize(
        batch=[11],
        dim=[96],
        max_seqlen=[66],
        width=[4],
    )
    def test_causal_conv1d_varlen(
        self,
        batch,
        dim,
        max_seqlen,
        width,
        has_bias=False,
        dtype=torch.bfloat16,
        prepack=False,
    ):
        total_entries = batch + 3

        seqlens = torch.randint(1, max_seqlen, (batch + 1,))
        seqlens[0] = 0
        # 1 or 2 must test
        seqlens[-2] = 2

        query_start_loc = torch.cumsum(seqlens, dim=0).to(torch.int32)

        seqlen = query_start_loc[-1].item()
        x = torch.randn(seqlen, dim, dtype=dtype).transpose_(-1, -2)
        weight = torch.randn(dim, width, dtype=dtype)
        bias = torch.randn(dim, dtype=dtype) if has_bias else None

        final_states = torch.randn(total_entries, dim, width - 1, dtype=dtype)
        final_states_ref = final_states.clone()

        has_initial_states = torch.randint(0, 2, (batch,), dtype=torch.bool).fill_(
            False
        )
        state_indices = torch.randperm(total_entries, dtype=torch.int32)[:batch]

        out_ref = []
        out_ref_b = []

        return_final_states = final_states is not None
        splits = torch.split(x, seqlens[1:].tolist(), dim=1)
        for i, x_s in enumerate(splits):
            out_ref_b.append(
                causal_conv1d_ref(
                    x_s.unsqueeze(0),
                    weight,
                    bias,
                    activation=self.activation,
                    return_final_states=return_final_states,
                    final_states_out=(
                        final_states_ref[state_indices[i]].unsqueeze(0)
                        if return_final_states
                        else None
                    ),
                    initial_states=(
                        final_states_ref[state_indices[i]].unsqueeze(0)
                        if has_initial_states[i]
                        else None
                    ),
                )
            )
        out_ref.append(torch.cat([t[0] for t in out_ref_b], dim=2))
        out_ref_tensor = torch.cat(out_ref, dim=0).squeeze(0)

        out = causal_conv1d_fwd(
            x,
            weight,
            bias,
            final_states,
            query_start_loc,
            state_indices,
            has_initial_states,
            self.activation in ["silu"],
            PAD_SLOT_ID,
            prepack,
        )

        atol = rtol = precision[dtype]
```
**EN:** This test exercises `test_causal_conv1d_varlen` by arranging inputs, invoking the relevant path, and checking the expected outcome. This chunk continues part 1 of the same logical block.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_causal_conv1d_varlen`。 这一段对应同一逻辑块的第 1 部分。

### Lines 240-241: test case causal conv1d varlen (part 2/2) / 测试用例 causal conv1d varlen（第 2/2 部分）
```python
        torch.testing.assert_close(out_ref_tensor, out, atol=atol, rtol=rtol)
        torch.testing.assert_close(final_states_ref, final_states, atol=atol, rtol=rtol)
```
**EN:** This test exercises `test_causal_conv1d_varlen` by arranging inputs, invoking the relevant path, and checking the expected outcome. This chunk continues part 2 of the same logical block.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_causal_conv1d_varlen`。 这一段对应同一逻辑块的第 2 部分。

### Lines 243-279: test case causal conv1d update / 测试用例 causal conv1d update
```python
    @parametrize(
        batch=[11],
        dim=[32, 64, 96],
        width=[4],
    )
    def test_causal_conv1d_update(
        self, batch, dim, width, has_bias=False, dtype=torch.bfloat16, prepack=True
    ):
        x = torch.randn(batch, dim).to(dtype)
        conv_state = torch.randn(batch, dim, width - 1, dtype=dtype)
        weight = torch.randn(dim, width).to(dtype)
        bias = torch.randn(dim).to(dtype) if has_bias else None

        packed_weight = causal_conv1d_weight_pack(weight) if prepack else weight

        conv_state_ref = conv_state.clone()
        out_ref = causal_conv1d_update_ref(
            x, conv_state_ref, weight, bias, activation=self.activation
        )

        cache_seqlens = None
        conv_state_indices = None
        out = causal_conv1d_update(
            x,
            conv_state,
            packed_weight,
            bias,
            self.activation in ["silu"],
            cache_seqlens,
            conv_state_indices,
            PAD_SLOT_ID,
            prepack,
        )

        atol = rtol = precision[dtype]
        torch.testing.assert_close(out_ref, out, atol=atol, rtol=rtol)
        torch.testing.assert_close(conv_state_ref, conv_state, atol=atol, rtol=rtol)
```
**EN:** This test exercises `test_causal_conv1d_update` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_causal_conv1d_update`。

### Lines 281-323: test case causal conv1d update with batch gather / 测试用例 causal conv1d update with batch gather
```python
    @parametrize(
        batch=[7],
        dim=[96],
        width=[4],
    )
    def test_causal_conv1d_update_with_batch_gather(
        self, batch, dim, width, has_bias=False, dtype=torch.bfloat16, prepack=True
    ):
        total_entries = batch + 3

        x = torch.randn(batch, dim).to(dtype=dtype)

        conv_state_indices = torch.randperm(total_entries)[:batch].to(dtype=torch.int32)
        conv_state = torch.randn(total_entries, dim, width - 1, dtype=dtype)

        weight = torch.randn(dim, width).to(dtype=dtype)
        bias = torch.randn(dim).to(dtype=dtype) if has_bias else None
        conv_state_ref = conv_state[conv_state_indices, :]

        packed_weight = causal_conv1d_weight_pack(weight) if prepack else weight

        out_ref = causal_conv1d_update_ref(
            x, conv_state_ref, weight, bias, activation=self.activation
        )

        cache_seqlens = None
        out = causal_conv1d_update(
            x,
            conv_state,
            packed_weight,
            bias,
            self.activation in ["silu"],
            cache_seqlens,
            conv_state_indices,
            PAD_SLOT_ID,
            prepack,
        )

        atol = rtol = precision[dtype]
        torch.testing.assert_close(out_ref, out, atol=atol, rtol=rtol)
        torch.testing.assert_close(
            conv_state_ref, conv_state[conv_state_indices, :], atol=atol, rtol=rtol
        )
```
**EN:** This test exercises `test_causal_conv1d_update_with_batch_gather` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_causal_conv1d_update_with_batch_gather`。

### Lines 326-327: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `causal_conv1d_ref`: x: (batch, dim, seqlen) weight: (dim, width) bias: (dim,) initial_states: (batch, dim, width - 1) final_states_out: (batch, dim, width - 1) out: (batch, dim, seqlen) / 该代码块实现 `causal_conv1d_ref`，承担模块行为中的一个聚焦逻辑片段。
- `causal_conv1d_update_ref`: x: (batch, dim) or (batch, dim, seqlen) conv_state: (batch, dim, state_len), where state_len >= width - 1 weight: (dim, width) bias: (dim,) cache_seqlens: (batch,), dtype int32. / 该代码块实现 `causal_conv1d_update_ref`，承担模块行为中的一个聚焦逻辑片段。
- `TestCausalConv1d`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestCausalConv1d.test_causal_conv1d`: This test exercises `test_causal_conv1d` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_causal_conv1d`。
- `TestCausalConv1d.test_causal_conv1d_varlen`: This test exercises `test_causal_conv1d_varlen` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_causal_conv1d_varlen`。
- `TestCausalConv1d.test_causal_conv1d_update`: This test exercises `test_causal_conv1d_update` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_causal_conv1d_update`。
- `TestCausalConv1d.test_causal_conv1d_update_with_batch_gather`: This test exercises `test_causal_conv1d_update_with_batch_gather` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_causal_conv1d_update_with_batch_gather`。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `unittest`, `typing`
- **Third-party modules / 第三方模块**: `sgl_kernel`, `torch`, `torch.nn.functional`, `utils`
- **Internal modules / 内部模块**: `sglang.test.test_utils`

- **Total lines / 总行数**: 327
