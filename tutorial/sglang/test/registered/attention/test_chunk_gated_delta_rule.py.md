# test_chunk_gated_delta_rule.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/attention/test_chunk_gated_delta_rule.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module focuses on attention chunk gated delta rule in SGLang. It captures the setup, helper logic, and expectations required by this scenario. / 该测试模块用于分析 SGLang 中与 attention chunk gated delta rule 相关的实现或行为。 它概括了该场景所需的初始化、辅助逻辑与预期结果。

## Line-by-Line Analysis / 逐行分析
### Lines 1-10: Import dependencies
```python
import unittest

import torch

from sglang.srt.layers.attention.fla.chunk import chunk_gated_delta_rule
from sglang.srt.layers.attention.fla.fused_recurrent import (
    fused_recurrent_gated_delta_rule,
)
from sglang.srt.utils import get_device
from sglang.test.ci.ci_register import register_cuda_ci, register_xpu_ci
```
**EN:** This block imports the libraries, fixtures, and helpers that the rest of the file relies on. It also registers the case for CUDA CI coverage.
**CN:** 该代码块导入了此文件后续逻辑依赖的库、夹具与辅助工具。 其中还会将该用例注册到 CUDA CI 覆盖范围。

### Lines 12-13: Register CI metadata
```python
register_cuda_ci(est_time=11, stage="base-b", runner_config="1-gpu-large")
register_xpu_ci(est_time=30, suite="xpu")
```
**EN:** This block registers the test case in the CI matrix together with execution-time or runner metadata. It also registers the case for CUDA CI coverage.
**CN:** 该代码块把测试用例注册到 CI 矩阵中，并附带执行耗时或运行器元数据。 其中还会将该用例注册到 CUDA CI 覆盖范围。

### Lines 16-20: Define class TestChunkGatedDeltaRule
```python
@unittest.skipIf(
    not (torch.cuda.is_available() or torch.xpu.is_available()),
    "Test requires CUDA or XPU",
)
class TestChunkGatedDeltaRule(unittest.TestCase):
```
**EN:** This declaration introduces the `TestChunkGatedDeltaRule` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestChunkGatedDeltaRule` 测试类，并说明它通过继承承担的职责。

### Lines 21-21: Document the class `TestChunkGatedDeltaRule`
```python
    """Test chunk_gated_delta_rule against token-by-token fused_recurrent reference."""
```
**EN:** This block stores descriptive text that explains the intent of the surrounding class `TestChunkGatedDeltaRule`.
**CN:** 该代码块保存说明性文本，用于解释周围class `TestChunkGatedDeltaRule`的设计意图。

### Lines 23-24: Declare TestChunkGatedDeltaRule configuration
```python
    ATOL = 2e-2
    RTOL = 1e-2
```
**EN:** This block defines class-level settings that are shared across the `TestChunkGatedDeltaRule` test methods.
**CN:** 该代码块定义了 `TestChunkGatedDeltaRule` 各测试方法共享的类级配置。

### Lines 26-53: Define helper: run reference
```python
    def _run_reference(self, pool_init, cache_indices, q, k, v, g, beta):
        """Per-batch token-by-token reference using fused_recurrent_gated_delta_rule.

        initial_state shape: [N, H, V, K] (native layout on this branch).
        """
        B = cache_indices.shape[0]
        T_per_seq = q.shape[1] // B
        pool = pool_init.clone()
        h_cur = pool[cache_indices].contiguous().clone()

        o_list = []
        for b in range(B):
            sl = slice(b * T_per_seq, (b + 1) * T_per_seq)
            o_b, h_b = fused_recurrent_gated_delta_rule(
                q=q[0, sl].unsqueeze(0),
                k=k[0, sl].unsqueeze(0),
                v=v[0, sl].unsqueeze(0),
                g=g[0, sl].unsqueeze(0),
                beta=beta[0, sl].unsqueeze(0),
                initial_state=h_cur[b : b + 1],
                output_final_state=True,
                use_qk_l2norm_in_kernel=True,
            )
            o_list.append(o_b)
            h_cur[b] = h_b[0]

        pool[cache_indices] = h_cur
        return torch.cat(o_list, dim=1), pool
```
**EN:** This helper function encapsulates reusable logic inside `TestChunkGatedDeltaRule` so the scenario stays organized.
**CN:** 该辅助函数封装了可复用逻辑，供 `TestChunkGatedDeltaRule` 内部调用，从而让场景结构更清晰。

### Lines 55-70: Define helper: run chunk
```python
    def _run_chunk(self, pool_init, cache_indices, q, k, v, g, beta, cu_seqlens):
        """Run chunk_gated_delta_rule with native [V, K] pool."""
        pool = pool_init.clone()
        o, _, _ = chunk_gated_delta_rule(
            q=q,
            k=k,
            v=v,
            g=g,
            beta=beta,
            initial_state=pool,
            initial_state_indices=cache_indices,
            cu_seqlens=cu_seqlens,
            head_first=False,
            use_qk_l2norm_in_kernel=True,
        )
        return o, pool
```
**EN:** This helper function encapsulates reusable logic inside `TestChunkGatedDeltaRule` so the scenario stays organized.
**CN:** 该辅助函数封装了可复用逻辑，供 `TestChunkGatedDeltaRule` 内部调用，从而让场景结构更清晰。

### Lines 72-127: Define helper: check shape
```python
    def _check_shape(
        self, B, T_per_seq, H, K, V, pool_size, sequential_indices=False, seed=42
    ):
        """Run correctness check for one (B, T_per_seq, H, K, V, pool_size) config."""
        device = get_device()
        dtype = torch.bfloat16
        T = B * T_per_seq

        torch.manual_seed(seed)

        if sequential_indices:
            cache_indices = torch.arange(B, dtype=torch.int32, device=device)
        else:
            perm = torch.randperm(pool_size, device=device)[:B]
            cache_indices = perm.to(torch.int32)

        pool_init = (
            torch.randn(pool_size, H, V, K, dtype=torch.float32, device=device) * 0.1
        )
        cu_seqlens = torch.zeros(B + 1, dtype=torch.long, device=device)
        cu_seqlens[1:] = (
            torch.arange(1, B + 1, dtype=torch.long, device=device) * T_per_seq
        )

        q = torch.randn(1, T, H, K, dtype=dtype, device=device)
        k = torch.randn(1, T, H, K, dtype=dtype, device=device)
        v = torch.randn(1, T, H, V, dtype=dtype, device=device)
        g = torch.nn.functional.logsigmoid(
            torch.randn(1, T, H, dtype=dtype, device=device)
        )
        beta = torch.sigmoid(torch.randn(1, T, H, dtype=dtype, device=device))

        o_ref, pool_ref = self._run_reference(
            pool_init, cache_indices, q, k, v, g, beta
        )
        o_new, pool_new = self._run_chunk(
            pool_init, cache_indices, q, k, v, g, beta, cu_seqlens
        )

        self.assertTrue(
            torch.allclose(
                o_ref.float(), o_new.float(), atol=self.ATOL, rtol=self.RTOL
            ),
            f"Output mismatch: max_diff="
            f"{(o_ref.float() - o_new.float()).abs().max().item():.2e}",
        )

        ref_slots = pool_ref[cache_indices].contiguous()
        new_slots = pool_new[cache_indices].contiguous()
        self.assertTrue(
            torch.allclose(
                ref_slots.float(), new_slots.float(), atol=self.ATOL, rtol=self.RTOL
            ),
            f"State mismatch: max_diff="
            f"{(ref_slots.float() - new_slots.float()).abs().max().item():.2e}",
        )
```
**EN:** This helper function encapsulates reusable logic inside `TestChunkGatedDeltaRule` so the scenario stays organized. It also checks concrete expectations with assertions.
**CN:** 该辅助函数封装了可复用逻辑，供 `TestChunkGatedDeltaRule` 内部调用，从而让场景结构更清晰。 其中还会通过断言检查明确的预期。

### Lines 132-133: Run test: production nt1
```python
    def test_production_nt1(self):
        self._check_shape(B=4, T_per_seq=64, H=16, K=128, V=128, pool_size=32)
```
**EN:** This test method exercises production nt1 and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 production nt1 场景，并验证观测到的行为是否符合预期契约。

### Lines 135-136: Run test: production nt2
```python
    def test_production_nt2(self):
        self._check_shape(B=4, T_per_seq=128, H=16, K=128, V=128, pool_size=32)
```
**EN:** This test method exercises production nt2 and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 production nt2 场景，并验证观测到的行为是否符合预期契约。

### Lines 138-139: Run test: production nt4
```python
    def test_production_nt4(self):
        self._check_shape(B=4, T_per_seq=256, H=16, K=128, V=128, pool_size=32)
```
**EN:** This test method exercises production nt4 and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 production nt4 场景，并验证观测到的行为是否符合预期契约。

### Lines 144-145: Run test: batch 1
```python
    def test_batch_1(self):
        self._check_shape(B=1, T_per_seq=128, H=16, K=128, V=128, pool_size=32)
```
**EN:** This test method exercises batch 1 and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 batch 1 场景，并验证观测到的行为是否符合预期契约。

### Lines 147-148: Run test: batch 2
```python
    def test_batch_2(self):
        self._check_shape(B=2, T_per_seq=128, H=16, K=128, V=128, pool_size=32)
```
**EN:** This test method exercises batch 2 and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 batch 2 场景，并验证观测到的行为是否符合预期契约。

### Lines 150-151: Run test: batch 8
```python
    def test_batch_8(self):
        self._check_shape(B=8, T_per_seq=128, H=16, K=128, V=128, pool_size=64)
```
**EN:** This test method exercises batch 8 and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 batch 8 场景，并验证观测到的行为是否符合预期契约。

### Lines 153-154: Run test: batch 16
```python
    def test_batch_16(self):
        self._check_shape(B=16, T_per_seq=64, H=16, K=128, V=128, pool_size=128)
```
**EN:** This test method exercises batch 16 and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 batch 16 场景，并验证观测到的行为是否符合预期契约。

### Lines 156-157: Run test: batch 32
```python
    def test_batch_32(self):
        self._check_shape(B=32, T_per_seq=32, H=16, K=128, V=128, pool_size=256)
```
**EN:** This test method exercises batch 32 and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 batch 32 场景，并验证观测到的行为是否符合预期契约。

### Lines 162-163: Run test: heads 4
```python
    def test_heads_4(self):
        self._check_shape(B=4, T_per_seq=128, H=4, K=128, V=128, pool_size=32)
```
**EN:** This test method exercises heads 4 and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 heads 4 场景，并验证观测到的行为是否符合预期契约。

### Lines 165-166: Run test: heads 8
```python
    def test_heads_8(self):
        self._check_shape(B=4, T_per_seq=128, H=8, K=128, V=128, pool_size=32)
```
**EN:** This test method exercises heads 8 and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 heads 8 场景，并验证观测到的行为是否符合预期契约。

### Lines 168-169: Run test: heads 32
```python
    def test_heads_32(self):
        self._check_shape(B=4, T_per_seq=128, H=32, K=128, V=128, pool_size=32)
```
**EN:** This test method exercises heads 32 and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 heads 32 场景，并验证观测到的行为是否符合预期契约。

### Lines 171-172: Run test: heads 64
```python
    def test_heads_64(self):
        self._check_shape(B=4, T_per_seq=128, H=64, K=128, V=128, pool_size=32)
```
**EN:** This test method exercises heads 64 and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 heads 64 场景，并验证观测到的行为是否符合预期契约。

### Lines 177-178: Run test: dim 64x64
```python
    def test_dim_64x64(self):
        self._check_shape(B=4, T_per_seq=128, H=16, K=64, V=64, pool_size=32)
```
**EN:** This test method exercises dim 64x64 and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 dim 64x64 场景，并验证观测到的行为是否符合预期契约。

### Lines 180-181: Run test: dim k lt v
```python
    def test_dim_k_lt_v(self):
        self._check_shape(B=4, T_per_seq=128, H=16, K=64, V=128, pool_size=32)
```
**EN:** This test method exercises dim k lt v and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 dim k lt v 场景，并验证观测到的行为是否符合预期契约。

### Lines 183-184: Run test: dim k gt v
```python
    def test_dim_k_gt_v(self):
        self._check_shape(B=4, T_per_seq=128, H=16, K=128, V=64, pool_size=32)
```
**EN:** This test method exercises dim k gt v and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 dim k gt v 场景，并验证观测到的行为是否符合预期契约。

### Lines 186-187: Run test: dim 256x256
```python
    def test_dim_256x256(self):
        self._check_shape(B=4, T_per_seq=128, H=16, K=256, V=256, pool_size=32)
```
**EN:** This test method exercises dim 256x256 and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 dim 256x256 场景，并验证观测到的行为是否符合预期契约。

### Lines 192-193: Run test: seqlen 1
```python
    def test_seqlen_1(self):
        self._check_shape(B=4, T_per_seq=1, H=16, K=128, V=128, pool_size=32)
```
**EN:** This test method exercises seqlen 1 and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 seqlen 1 场景，并验证观测到的行为是否符合预期契约。

### Lines 195-196: Run test: seqlen 7
```python
    def test_seqlen_7(self):
        self._check_shape(B=4, T_per_seq=7, H=16, K=128, V=128, pool_size=32)
```
**EN:** This test method exercises seqlen 7 and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 seqlen 7 场景，并验证观测到的行为是否符合预期契约。

### Lines 198-199: Run test: seqlen 16
```python
    def test_seqlen_16(self):
        self._check_shape(B=4, T_per_seq=16, H=16, K=128, V=128, pool_size=32)
```
**EN:** This test method exercises seqlen 16 and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 seqlen 16 场景，并验证观测到的行为是否符合预期契约。

### Lines 201-202: Run test: seqlen 32
```python
    def test_seqlen_32(self):
        self._check_shape(B=4, T_per_seq=32, H=16, K=128, V=128, pool_size=32)
```
**EN:** This test method exercises seqlen 32 and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 seqlen 32 场景，并验证观测到的行为是否符合预期契约。

### Lines 207-208: Run test: multi chunk nt8
```python
    def test_multi_chunk_nt8(self):
        self._check_shape(B=4, T_per_seq=512, H=16, K=128, V=128, pool_size=32)
```
**EN:** This test method exercises multi chunk nt8 and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 multi chunk nt8 场景，并验证观测到的行为是否符合预期契约。

### Lines 210-211: Run test: large pool
```python
    def test_large_pool(self):
        self._check_shape(B=4, T_per_seq=128, H=16, K=128, V=128, pool_size=512)
```
**EN:** This test method exercises large pool and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 large pool 场景，并验证观测到的行为是否符合预期契约。

### Lines 216-217: Run test: stress
```python
    def test_stress(self):
        self._check_shape(B=32, T_per_seq=128, H=32, K=128, V=128, pool_size=256)
```
**EN:** This test method exercises stress and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 stress 场景，并验证观测到的行为是否符合预期契约。

### Lines 222-231: Run test: seq idx b4
```python
    def test_seq_idx_b4(self):
        self._check_shape(
            B=4,
            T_per_seq=128,
            H=16,
            K=128,
            V=128,
            pool_size=4,
            sequential_indices=True,
        )
```
**EN:** This test method exercises seq idx b4 and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 seq idx b4 场景，并验证观测到的行为是否符合预期契约。

### Lines 233-242: Run test: seq idx b8
```python
    def test_seq_idx_b8(self):
        self._check_shape(
            B=8,
            T_per_seq=128,
            H=16,
            K=128,
            V=128,
            pool_size=8,
            sequential_indices=True,
        )
```
**EN:** This test method exercises seq idx b8 and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 seq idx b8 场景，并验证观测到的行为是否符合预期契约。

### Lines 244-253: Run test: seq idx h32
```python
    def test_seq_idx_h32(self):
        self._check_shape(
            B=4,
            T_per_seq=128,
            H=32,
            K=128,
            V=128,
            pool_size=4,
            sequential_indices=True,
        )
```
**EN:** This test method exercises seq idx h32 and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 seq idx h32 场景，并验证观测到的行为是否符合预期契约。

### Lines 255-264: Run test: seq idx h64
```python
    def test_seq_idx_h64(self):
        self._check_shape(
            B=4,
            T_per_seq=128,
            H=64,
            K=128,
            V=128,
            pool_size=4,
            sequential_indices=True,
        )
```
**EN:** This test method exercises seq idx h64 and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 seq idx h64 场景，并验证观测到的行为是否符合预期契约。

### Lines 266-275: Run test: seq idx stress
```python
    def test_seq_idx_stress(self):
        self._check_shape(
            B=32,
            T_per_seq=128,
            H=32,
            K=128,
            V=128,
            pool_size=32,
            sequential_indices=True,
        )
```
**EN:** This test method exercises seq idx stress and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 seq idx stress 场景，并验证观测到的行为是否符合预期契约。

### Lines 278-279: Expose unittest entrypoint
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

## Dependencies / 依赖关系
- Internal modules / 内部模块: `sglang.srt.layers.attention.fla.chunk`, `sglang.srt.layers.attention.fla.fused_recurrent`, `sglang.srt.utils`, `sglang.test.ci.ci_register`
- External and stdlib modules / 外部与标准库模块: `torch`, `unittest`
- Notable symbols / 关键符号: `register_cuda_ci`, `unittest.main`
