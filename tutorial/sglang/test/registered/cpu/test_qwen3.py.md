# test_qwen3.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/cpu/test_qwen3.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module focuses on cpu qwen3 in SGLang. It captures the setup, helper logic, and expectations required by this scenario. / 该测试模块用于分析 SGLang 中与 cpu qwen3 相关的实现或行为。 它概括了该场景所需的初始化、辅助逻辑与预期结果。

## Line-by-Line Analysis / 逐行分析
### Lines 1-7: Import dependencies
```python
import unittest

import torch
from utils import precision

from sglang.test.ci.ci_register import register_cpu_ci
from sglang.test.test_utils import CustomTestCase
```
**EN:** This block imports the libraries, fixtures, and helpers that the rest of the file relies on.
**CN:** 该代码块导入了此文件后续逻辑依赖的库、夹具与辅助工具。

### Lines 9-9: Register CI metadata
```python
register_cpu_ci(est_time=10, suite="base-b-test-cpu")
```
**EN:** This block registers the test case in the CI matrix together with execution-time or runner metadata.
**CN:** 该代码块把测试用例注册到 CI 矩阵中，并附带执行耗时或运行器元数据。

### Lines 11-11: Implement expr logic
```python
torch.manual_seed(1234)
```
**EN:** This block contains supporting statements that connect the surrounding setup and assertions.
**CN:** 该代码块包含连接周边初始化与断言的辅助语句。

### Lines 14-56: Define helper: fix query key value ordering reshape cat
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
**EN:** This helper function encapsulates reusable logic so the scenario stays organized.
**CN:** 该辅助函数封装了可复用逻辑，从而让场景结构更清晰。

### Lines 59-84: Define helper: fix query key value ordering reshape cat contiguous
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
**EN:** This helper function encapsulates reusable logic so the scenario stays organized.
**CN:** 该辅助函数封装了可复用逻辑，从而让场景结构更清晰。

### Lines 87-87: Define class TestQwen3
```python
class TestQwen3(CustomTestCase):
```
**EN:** This declaration introduces the `TestQwen3` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestQwen3` 测试类，并说明它通过继承承担的职责。

### Lines 88-114: Run test: fused qkvzba split reshape cat
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
**EN:** This test method exercises fused qkvzba split reshape cat and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 fused qkvzba split reshape cat 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 116-148: Run test: fused qkvzba split reshape cat contiguous
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
**EN:** This test method exercises fused qkvzba split reshape cat contiguous and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 fused qkvzba split reshape cat contiguous 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 151-152: Expose unittest entrypoint
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This conditional keeps the module executable as a standalone unittest target. It also keeps a standalone unittest entrypoint available.
**CN:** 该条件语句使模块可以作为独立的 unittest 目标直接运行。 其中还会保留可独立运行的 unittest 入口。

## Key Concepts / 关键概念
- The file is structured as an automated regression test for a focused scenario. / 该文件按聚焦场景组织为自动化回归测试。
- CI registration keeps the case visible to automated test pipelines. / CI 注册使该用例能够被自动化测试流水线发现。
- Assertions encode the behavioral contract that the scenario must satisfy. / 断言编码了该场景必须满足的行为契约。
- Shared base classes centralize common setup, assertions, and diagnostics. / 共享基类集中封装了通用初始化、断言与诊断逻辑。

## Dependencies / 依赖关系
- Internal modules / 内部模块: `sglang.test.ci.ci_register`, `sglang.test.test_utils`
- External and stdlib modules / 外部与标准库模块: `torch`, `unittest`, `utils`
- Notable symbols / 关键符号: `CustomTestCase`, `unittest.main`
