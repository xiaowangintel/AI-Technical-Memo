# test_flash_attn.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/cpu/test_flash_attn.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module focuses on cpu flash attn in SGLang. It captures the setup, helper logic, and expectations required by this scenario. / 该测试模块用于分析 SGLang 中与 cpu flash attn 相关的实现或行为。 它概括了该场景所需的初始化、辅助逻辑与预期结果。

## Line-by-Line Analysis / 逐行分析
### Lines 1-9: Import dependencies
```python
import unittest

import sgl_kernel  # noqa: F401
import torch
import torch.nn.functional as F
from utils import parametrize, precision

from sglang.test.ci.ci_register import register_cpu_ci
from sglang.test.test_utils import CustomTestCase
```
**EN:** This block imports the libraries, fixtures, and helpers that the rest of the file relies on.
**CN:** 该代码块导入了此文件后续逻辑依赖的库、夹具与辅助工具。

### Lines 11-11: Register CI metadata
```python
register_cpu_ci(est_time=10, suite="base-b-test-cpu")
```
**EN:** This block registers the test case in the CI matrix together with execution-time or runner metadata.
**CN:** 该代码块把测试用例注册到 CI 矩阵中，并附带执行耗时或运行器元数据。

### Lines 13-13: Define module constants
```python
flash_attn_varlen_func = torch.ops.sgl_kernel.flash_attn_varlen_func
```
**EN:** This block stores configuration values or reusable constants for the rest of the module.
**CN:** 该代码块保存模块其余部分会复用的配置值或常量。

### Lines 15-15: Implement expr logic
```python
torch.manual_seed(1234)
```
**EN:** This block contains supporting statements that connect the surrounding setup and assertions.
**CN:** 该代码块包含连接周边初始化与断言的辅助语句。

### Lines 18-49: Define helper: flash attn varlen ref
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
**EN:** This helper function encapsulates reusable logic so the scenario stays organized.
**CN:** 该辅助函数封装了可复用逻辑，从而让场景结构更清晰。

### Lines 53-80: Define helper: flash attn non varlen ref
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
**EN:** This helper function encapsulates reusable logic so the scenario stays organized.
**CN:** 该辅助函数封装了可复用逻辑，从而让场景结构更清晰。

### Lines 83-83: Define class TestFlashAttn
```python
class TestFlashAttn(CustomTestCase):
```
**EN:** This declaration introduces the `TestFlashAttn` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestFlashAttn` 测试类，并说明它通过继承承担的职责。

### Lines 85-144: Run test: flash attn varlen
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
**EN:** This test method exercises flash attn varlen and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 flash attn varlen 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 147-204: Run test: flash attn large size (part 1)
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
```
**EN:** This test method exercises flash attn large size and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 flash attn large size 场景，并验证观测到的行为是否符合预期契约。

### Lines 206-207: Run test: flash attn large size (part 2)
```python
        atol = rtol = precision[dtype]
        torch.testing.assert_close(out_ref, out, atol=atol, rtol=rtol)
```
**EN:** This test method exercises flash attn large size and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 flash attn large size 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 209-233: Define helper: test flash attn large seq causal mask once
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
**EN:** This helper function encapsulates reusable logic inside `TestFlashAttn` so the scenario stays organized. It also checks concrete expectations with assertions.
**CN:** 该辅助函数封装了可复用逻辑，供 `TestFlashAttn` 内部调用，从而让场景结构更清晰。 其中还会通过断言检查明确的预期。

### Lines 235-241: Run test: flash attn large seq causal mask
```python
    def test_flash_attn_large_seq_causal_mask(self):
        # Non-varlen path: single sequence, has_varlen_sequences returns False
        # → dispatches to flash_attn_kernel_impl.
        self._test_flash_attn_large_seq_causal_mask_once([5000])
        # Varlen path: sequences with different lengths, has_varlen_sequences
        # returns True → dispatches to flash_attn_varlen_kernel_impl
        self._test_flash_attn_large_seq_causal_mask_once([5000, 4999])
```
**EN:** This test method exercises flash attn large seq causal mask and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 flash attn large seq causal mask 场景，并验证观测到的行为是否符合预期契约。

### Lines 244-245: Expose unittest entrypoint
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
- External and stdlib modules / 外部与标准库模块: `sgl_kernel`, `torch`, `torch.nn.functional`, `unittest`, `utils`
- Notable symbols / 关键符号: `CustomTestCase`, `unittest.main`
