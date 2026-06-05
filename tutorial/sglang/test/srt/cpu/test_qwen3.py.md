# test_qwen3.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/srt/cpu/test_qwen3.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates qwen3 behavior in SGLang's srt / cpu area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 SRT / CPU 领域中与 qwen3 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-6: module imports and dependencies / 模块导入与依赖
```python
import unittest

import torch
from utils import precision

from sglang.test.test_utils import CustomTestCase
```
**EN:** This block imports the modules needed by the rest of the file, including `unittest`, `torch`, `utils`, `sglang.test.test_utils`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `unittest`, `torch`, `utils`, `sglang.test.test_utils`。

### Lines 8-8: supporting statements / 辅助语句
```python
torch.manual_seed(1234)
```
**EN:** This block performs supporting work through calls such as `manual_seed`, preparing state for nearby definitions.
**CN:** 该代码块通过 `manual_seed` 等调用完成辅助性工作，为相邻定义准备状态。

### Lines 11-53: function fix query key value ordering reshape cat / 函数 fix query key value ordering reshape cat
```python
def fix_query_key_value_ordering_reshape_cat(
    mixed_qkvz, mixed_ba, num_k_heads, num_v_heads, attn_tp_size, head_k_dim, head_v_dim
):
    new_tensor_shape_qkvz = mixed_qkvz.size()[:-1] + (
        num_k_heads // attn_tp_size,
        (
            head_k_dim
            + head_k_dim
            + (head_v_dim + head_v_dim) * num_v_heads // num_k_heads
        ),
    )
    new_tensor_shape_ba = mixed_ba.size()[:-1] + (
        num_k_heads // attn_tp_size,
        2 * num_v_heads // num_k_heads,
    )

    mixed_qkvz = mixed_qkvz.view(*new_tensor_shape_qkvz)
    mixed_ba = mixed_ba.view(*new_tensor_shape_ba)

    split_arg_list_qkvz = [
        head_k_dim,
        head_k_dim,
        (num_v_heads // num_k_heads * head_v_dim),
        (num_v_heads // num_k_heads * head_v_dim),
    ]
    split_arg_list_ba = [
        num_v_heads // num_k_heads,
        num_v_heads // num_k_heads,
    ]
    # [b, sq, ng, (hn + hn + np/ng * hn + np/ng + np/ng)]
    # --> [b, sq, ng, hn], [b, sq, ng, hn], [b, sq, ng, np/ng * hn], [b, sq, ng, np/ng * hn], [b, sq, ng, np/ng], [b, sq, ng, np/ng]
    query, key, value, z = torch.split(mixed_qkvz, split_arg_list_qkvz, dim=2)
    b, a = torch.split(mixed_ba, split_arg_list_ba, dim=2)

    # [b, sq, ng, np/ng * hn] -> [b, sq, np, hn]
    value = value.reshape(value.size(0), -1, head_v_dim)
    z = z.reshape(z.size(0), -1, head_v_dim)
    b = b.reshape(b.size(0), num_v_heads // attn_tp_size)
    a = a.reshape(a.size(0), num_v_heads // attn_tp_size)
    query, key, value = map(lambda x: x.reshape(x.shape[0], -1), (query, key, value))
    mixed_qkv = torch.cat((query, key, value), dim=-1)

    return mixed_qkv, z, b, a
```
**EN:** This block implements `fix_query_key_value_ordering_reshape_cat` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `fix_query_key_value_ordering_reshape_cat`，承担模块行为中的一个聚焦逻辑片段。

### Lines 56-81: function fix query key value ordering reshape cat contiguous / 函数 fix query key value ordering reshape cat contiguous
```python
def fix_query_key_value_ordering_reshape_cat_contiguous(
    mixed_qkvz: torch.Tensor,
    mixed_ba: torch.Tensor,
    key_dim: int,
    value_dim: int,
    num_v_heads: int,
    head_v_dim: int,
    attn_tp_size: int,
):
    """
    Derives `query`, `key` and `value` tensors from `mixed_qkvzba`.
    """
    k_tp = key_dim // attn_tp_size
    v_tp = value_dim // attn_tp_size
    nv_tp = num_v_heads // attn_tp_size

    # Directly split, no head group reshape
    query, key, value, z = mixed_qkvz.split([k_tp, k_tp, v_tp, v_tp], dim=-1)
    b, a = mixed_ba.split([nv_tp, nv_tp], dim=-1)

    # value / z reshape to (seq, num_v_heads/tp, head_v_dim)
    value = value.reshape(value.size(0), -1, head_v_dim)
    z = z.reshape(z.size(0), -1, head_v_dim)
    query, key, value = map(lambda x: x.reshape(x.shape[0], -1), (query, key, value))
    mixed_qkv = torch.cat((query, key, value), dim=-1)
    return mixed_qkv, z, b, a
```
**EN:** Derives `query`, `key` and `value` tensors from `mixed_qkvzba`. This block implements `fix_query_key_value_ordering_reshape_cat_contiguous` and captures one focused piece of the module's behavior.
**CN:** Derives `query`, `key` and `value` tensors from `mixed_qkvzba`. 该代码块实现 `fix_query_key_value_ordering_reshape_cat_contiguous`，承担模块行为中的一个聚焦逻辑片段。

### Lines 84-84: class TestQwen3 declaration / 类 TestQwen3 声明
```python
class TestQwen3(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 85-111: test case fused qkvzba split reshape cat / 测试用例 fused qkvzba split reshape cat
```python
    def test_fused_qkvzba_split_reshape_cat(self):
        mixed_qkvz = torch.rand(1024, 12288, dtype=torch.bfloat16)
        mixed_ba = torch.rand(1024, 64, dtype=torch.bfloat16)
        head_k_dim = 128
        head_v_dim = 128
        num_v_heads = 32
        num_k_heads = 16
        attn_tp_size = 1
        mixed_qkv_ref, z_ref, b_ref, a_ref = fix_query_key_value_ordering_reshape_cat(
            mixed_qkvz,
            mixed_ba,
            num_k_heads,
            num_v_heads,
            attn_tp_size,
            head_k_dim,
            head_v_dim,
        )
        num_heads_qk = num_k_heads // attn_tp_size
        num_heads_v = num_v_heads // attn_tp_size
        mixed_qkv, z, b, a = torch.ops.sgl_kernel.fused_qkvzba_split_reshape_cat_cpu(
            mixed_qkvz, mixed_ba, num_heads_qk, num_heads_v, head_k_dim, head_v_dim
        )
        atol = rtol = precision[mixed_qkv.dtype]
        torch.testing.assert_close(mixed_qkv, mixed_qkv_ref, atol=atol, rtol=rtol)
        torch.testing.assert_close(z, z_ref, atol=atol, rtol=rtol)
        torch.testing.assert_close(b, b_ref, atol=atol, rtol=rtol)
        torch.testing.assert_close(a, a_ref, atol=atol, rtol=rtol)
```
**EN:** This test exercises `test_fused_qkvzba_split_reshape_cat` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_fused_qkvzba_split_reshape_cat`。

### Lines 113-145: test case fused qkvzba split reshape cat contiguous / 测试用例 fused qkvzba split reshape cat contiguous
```python
    def test_fused_qkvzba_split_reshape_cat_contiguous(self):
        mixed_qkvz = torch.rand(1, 12288, dtype=torch.bfloat16)
        mixed_ba = torch.rand(1, 64, dtype=torch.bfloat16)
        head_k_dim = 128
        head_v_dim = 128
        num_v_heads = 32
        num_k_heads = 16
        attn_tp_size = 1
        key_dim = head_k_dim * num_k_heads
        value_dim = head_v_dim * num_v_heads
        mixed_qkv_ref, z_ref, b_ref, a_ref = (
            fix_query_key_value_ordering_reshape_cat_contiguous(
                mixed_qkvz,
                mixed_ba,
                key_dim,
                value_dim,
                num_v_heads,
                head_v_dim,
                attn_tp_size,
            )
        )
        num_heads_qk = num_k_heads // attn_tp_size
        num_heads_v = num_v_heads // attn_tp_size
        mixed_qkv, z, b, a = (
            torch.ops.sgl_kernel.fused_qkvzba_split_reshape_cat_contiguous_cpu(
                mixed_qkvz, mixed_ba, num_heads_qk, num_heads_v, head_k_dim, head_v_dim
            )
        )
        atol = rtol = precision[mixed_qkv.dtype]
        torch.testing.assert_close(mixed_qkv, mixed_qkv_ref, atol=atol, rtol=rtol)
        torch.testing.assert_close(z, z_ref, atol=atol, rtol=rtol)
        torch.testing.assert_close(b, b_ref, atol=atol, rtol=rtol)
        torch.testing.assert_close(a, a_ref, atol=atol, rtol=rtol)
```
**EN:** This test exercises `test_fused_qkvzba_split_reshape_cat_contiguous` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_fused_qkvzba_split_reshape_cat_contiguous`。

### Lines 148-149: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `fix_query_key_value_ordering_reshape_cat`: This block implements `fix_query_key_value_ordering_reshape_cat` and captures one focused piece of the module's behavior. / 该代码块实现 `fix_query_key_value_ordering_reshape_cat`，承担模块行为中的一个聚焦逻辑片段。
- `fix_query_key_value_ordering_reshape_cat_contiguous`: Derives `query`, `key` and `value` tensors from `mixed_qkvzba`. / 该代码块实现 `fix_query_key_value_ordering_reshape_cat_contiguous`，承担模块行为中的一个聚焦逻辑片段。
- `TestQwen3`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestQwen3.test_fused_qkvzba_split_reshape_cat`: This test exercises `test_fused_qkvzba_split_reshape_cat` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_fused_qkvzba_split_reshape_cat`。
- `TestQwen3.test_fused_qkvzba_split_reshape_cat_contiguous`: This test exercises `test_fused_qkvzba_split_reshape_cat_contiguous` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_fused_qkvzba_split_reshape_cat_contiguous`。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `unittest`
- **Third-party modules / 第三方模块**: `torch`, `utils`
- **Internal modules / 内部模块**: `sglang.test.test_utils`

- **Total lines / 总行数**: 149
