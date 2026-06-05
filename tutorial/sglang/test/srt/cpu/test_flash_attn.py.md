# test_flash_attn.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/srt/cpu/test_flash_attn.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates flash attn behavior in SGLang's srt / cpu area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 SRT / CPU 领域中与 flash attn 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-8: module imports and dependencies / 模块导入与依赖
```python
import unittest

import sgl_kernel  # noqa: F401
import torch
import torch.nn.functional as F
from utils import parametrize, precision

from sglang.test.test_utils import CustomTestCase
```
**EN:** This block imports the modules needed by the rest of the file, including `unittest`, `sgl_kernel`, `torch`, `torch.nn.functional`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `unittest`, `sgl_kernel`, `torch`, `torch.nn.functional`。

### Lines 10-12: module-level constants and configuration / 模块级常量与配置
```python
flash_attn_varlen_func = torch.ops.sgl_kernel.flash_attn_varlen_func

torch.manual_seed(1234)
```
**EN:** This block defines shared names such as `flash_attn_varlen_func`. These values centralize configuration that later tests or helpers reuse.
**CN:** 该代码块定义了 `flash_attn_varlen_func` 等共享名称，用于集中保存后续测试或辅助逻辑会复用的配置。

### Lines 15-46: function flash attn varlen ref / 函数 flash attn varlen ref
```python
def flash_attn_varlen_ref(
    q,
    k,
    v,
    cu_seqlens_q,
    cu_seqlens_k,
    is_causal,
    enable_gqa,
):
    cu_q = cu_seqlens_q.tolist()
    cu_k = cu_seqlens_k.tolist()
    batch = len(cu_k) - 1

    # [T, H, D] -> [1, H, T, D]
    q, k, v = [x.unsqueeze(0).transpose(1, 2) for x in [q, k, v]]

    B, H, T, D = q.shape
    out = torch.empty(B, H, T, v.size(-1), dtype=q.dtype)
    for b in range(batch):
        start_q, end_q = cu_q[b], cu_q[b + 1]
        start_k, end_k = cu_k[b], cu_k[b + 1]

        out[:, :, start_q:end_q, :] = F.scaled_dot_product_attention(
            q[:, :, start_q:end_q, :],
            k[:, :, start_k:end_k, :],
            v[:, :, start_k:end_k, :],
            is_causal=is_causal,
            enable_gqa=enable_gqa,
        )

    # [1, H, T, D] -> [T, H, D]
    return out.transpose(1, 2).squeeze(0)
```
**EN:** This block implements `flash_attn_varlen_ref` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `flash_attn_varlen_ref`，承担模块行为中的一个聚焦逻辑片段。

### Lines 47-49: supporting source context / 辅助源码上下文
```python


# faster version ref kernel for non varlen case
```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 50-77: function flash attn non varlen ref / 函数 flash attn non varlen ref
```python
def flash_attn_non_varlen_ref(
    q,
    k,
    v,
    cu_seqlens_q,
    cu_seqlens_k,
    is_causal,
    enable_gqa,
):
    cu_q = cu_seqlens_q.tolist()
    cu_k = cu_seqlens_k.tolist()
    batch = len(cu_k) - 1

    B_T, H, D = q.shape
    T = B_T // batch

    # [T, H, D] -> [1, H, T, D]
    q, k, v = [x.reshape(batch, T, H, D).transpose(1, 2) for x in [q, k, v]]

    out = F.scaled_dot_product_attention(
        q,
        k,
        v,
        is_causal=is_causal,
        enable_gqa=enable_gqa,
    )
    # [B, H, T, D] -> [B * T, H, D]
    return out.transpose(1, 2).reshape(batch * T, H, D)
```
**EN:** This block implements `flash_attn_non_varlen_ref` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `flash_attn_non_varlen_ref`，承担模块行为中的一个聚焦逻辑片段。

### Lines 80-81: class TestFlashAttn declaration / 类 TestFlashAttn 声明
```python
class TestFlashAttn(CustomTestCase):

```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 82-141: test case flash attn varlen / 测试用例 flash attn varlen
```python
    @parametrize(
        batch=[4],
        max_seqlen_q=[35, 96],
        max_seqlen_k=[35, 96],
        num_heads=[16],
        num_heads_kv=[16, 2],
        head_dim=[32, 48],  # test when D is not 32x
        head_dim_v=[32],
        is_causal=[True, False],
    )
    def test_flash_attn_varlen(
        self,
        batch,
        max_seqlen_q,
        max_seqlen_k,
        num_heads,
        num_heads_kv,
        head_dim,
        head_dim_v,
        is_causal,
    ):
        dtype = torch.bfloat16

        # random seqlens for k and kv
        seqlens_q = torch.randint(1, max_seqlen_q, (batch,), dtype=torch.int32)
        seqlens_k = torch.randint(1, max_seqlen_k, (batch,), dtype=torch.int32)
        cu_seqlens_q = torch.zeros((batch + 1,), dtype=torch.int32)
        cu_seqlens_k = torch.zeros((batch + 1,), dtype=torch.int32)
        cu_seqlens_q[1:] = torch.cumsum(seqlens_q, 0)
        cu_seqlens_k[1:] = torch.cumsum(seqlens_k, 0)

        sum_seqlen_q = seqlens_q.sum().item()
        sum_seqlen_k = seqlens_k.sum().item()
        q = torch.randn(sum_seqlen_q, num_heads, head_dim).to(dtype)
        k = torch.randn(sum_seqlen_k, num_heads_kv, head_dim).to(dtype)
        v = torch.randn(sum_seqlen_k, num_heads_kv, head_dim_v).to(dtype)

        out_ref = flash_attn_varlen_ref(
            q,
            k,
            v,
            cu_seqlens_q,
            cu_seqlens_k,
            is_causal=is_causal,
            enable_gqa=num_heads != num_heads_kv,
        )

        out = flash_attn_varlen_func(
            q,
            k,
            v,
            cu_seqlens_q,
            cu_seqlens_k,
            seqlens_q.max().item(),
            seqlens_k.max().item(),
            is_causal,
        )

        atol = rtol = precision[dtype]
        torch.testing.assert_close(out_ref, out, atol=atol, rtol=rtol)
```
**EN:** This test exercises `test_flash_attn_varlen` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_flash_attn_varlen`。

### Lines 142-143: supporting source context / 辅助源码上下文
```python

    # test with large size to capture overflow issue
```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 144-204: test case flash attn large size / 测试用例 flash attn large size
```python
    @parametrize(
        batch=[4097],
        max_seqlen_q=[4097],
        max_seqlen_k=[4097],
        num_heads=[4],
        num_heads_kv=[4],
        head_dim=[32],
        head_dim_v=[32],
        is_causal=[False],
    )
    def test_flash_attn_large_size(
        self,
        batch,
        max_seqlen_q,
        max_seqlen_k,
        num_heads,
        num_heads_kv,
        head_dim,
        head_dim_v,
        is_causal,
    ):
        dtype = torch.bfloat16

        # test the non varlen case
        seqlens_q = torch.full((batch,), max_seqlen_q, dtype=torch.int32)
        seqlens_k = torch.full((batch,), max_seqlen_k, dtype=torch.int32)

        cu_seqlens_q = torch.zeros((batch + 1,), dtype=torch.int32)
        cu_seqlens_k = torch.zeros((batch + 1,), dtype=torch.int32)
        cu_seqlens_q[1:] = torch.cumsum(seqlens_q, 0)
        cu_seqlens_k[1:] = torch.cumsum(seqlens_k, 0)

        sum_seqlen_q = seqlens_q.sum().item()
        sum_seqlen_k = seqlens_k.sum().item()
        q = torch.randn(sum_seqlen_q, num_heads, head_dim).to(dtype)
        k = torch.randn(sum_seqlen_k, num_heads_kv, head_dim).to(dtype)
        v = torch.randn(sum_seqlen_k, num_heads_kv, head_dim_v).to(dtype)

        out_ref = flash_attn_non_varlen_ref(
            q,
            k,
            v,
            cu_seqlens_q,
            cu_seqlens_k,
            is_causal=is_causal,
            enable_gqa=num_heads != num_heads_kv,
        )

        out = flash_attn_varlen_func(
            q,
            k,
            v,
            cu_seqlens_q,
            cu_seqlens_k,
            seqlens_q.max().item(),
            seqlens_k.max().item(),
            is_causal,
        )

        atol = rtol = precision[dtype]
        torch.testing.assert_close(out_ref, out, atol=atol, rtol=rtol)
```
**EN:** This test exercises `test_flash_attn_large_size` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_flash_attn_large_size`。

### Lines 206-230: method test flash attn large seq causal mask once / 方法 test flash attn large seq causal mask once
```python
    def _test_flash_attn_large_seq_causal_mask_once(self, seqlens):
        dtype = torch.bfloat16
        num_heads = 8
        num_heads_kv = 2
        head_dim = 64

        seqlens_t = torch.tensor(seqlens, dtype=torch.int32)
        cu_seqlens = torch.zeros(len(seqlens) + 1, dtype=torch.int32)
        cu_seqlens[1:] = torch.cumsum(seqlens_t, 0)
        total = cu_seqlens[-1].item()
        max_seqlen = seqlens_t.max().item()

        q = torch.randn(total, num_heads, head_dim, dtype=dtype)
        k = torch.randn(total, num_heads_kv, head_dim, dtype=dtype)
        v = torch.randn(total, num_heads_kv, head_dim, dtype=dtype)

        out_ref = flash_attn_varlen_ref(
            q, k, v, cu_seqlens, cu_seqlens, is_causal=True, enable_gqa=True
        )
        out = flash_attn_varlen_func(
            q, k, v, cu_seqlens, cu_seqlens, max_seqlen, max_seqlen, True
        )

        atol = rtol = precision[dtype]
        torch.testing.assert_close(out_ref, out, atol=atol, rtol=rtol)
```
**EN:** This block implements `_test_flash_attn_large_seq_causal_mask_once` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_test_flash_attn_large_seq_causal_mask_once`，承担模块行为中的一个聚焦逻辑片段。

### Lines 232-238: test case flash attn large seq causal mask / 测试用例 flash attn large seq causal mask
```python
    def test_flash_attn_large_seq_causal_mask(self):
        # Non-varlen path: single sequence, has_varlen_sequences returns False
        # → dispatches to flash_attn_kernel_impl.
        self._test_flash_attn_large_seq_causal_mask_once([5000])
        # Varlen path: sequences with different lengths, has_varlen_sequences
        # returns True → dispatches to flash_attn_varlen_kernel_impl
        self._test_flash_attn_large_seq_causal_mask_once([5000, 4999])
```
**EN:** This test exercises `test_flash_attn_large_seq_causal_mask` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_flash_attn_large_seq_causal_mask`。

### Lines 241-242: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `flash_attn_varlen_ref`: This block implements `flash_attn_varlen_ref` and captures one focused piece of the module's behavior. / 该代码块实现 `flash_attn_varlen_ref`，承担模块行为中的一个聚焦逻辑片段。
- `flash_attn_non_varlen_ref`: This block implements `flash_attn_non_varlen_ref` and captures one focused piece of the module's behavior. / 该代码块实现 `flash_attn_non_varlen_ref`，承担模块行为中的一个聚焦逻辑片段。
- `TestFlashAttn`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestFlashAttn.test_flash_attn_varlen`: This test exercises `test_flash_attn_varlen` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_flash_attn_varlen`。
- `TestFlashAttn.test_flash_attn_large_size`: This test exercises `test_flash_attn_large_size` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_flash_attn_large_size`。
- `TestFlashAttn._test_flash_attn_large_seq_causal_mask_once`: This block implements `_test_flash_attn_large_seq_causal_mask_once` and captures one focused piece of the module's behavior. / 该代码块实现 `_test_flash_attn_large_seq_causal_mask_once`，承担模块行为中的一个聚焦逻辑片段。
- `TestFlashAttn.test_flash_attn_large_seq_causal_mask`: This test exercises `test_flash_attn_large_seq_causal_mask` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_flash_attn_large_seq_causal_mask`。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `unittest`
- **Third-party modules / 第三方模块**: `sgl_kernel`, `torch`, `torch.nn.functional`, `utils`
- **Internal modules / 内部模块**: `sglang.test.test_utils`

- **Total lines / 总行数**: 242
