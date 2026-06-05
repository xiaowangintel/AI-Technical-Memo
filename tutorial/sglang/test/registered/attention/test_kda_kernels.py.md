# test_kda_kernels.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/attention/test_kda_kernels.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module focuses on attention kda kernels in SGLang. It captures the setup, helper logic, and expectations required by this scenario. / 该测试模块用于分析 SGLang 中与 attention kda kernels 相关的实现或行为。 它概括了该场景所需的初始化、辅助逻辑与预期结果。

## Line-by-Line Analysis / 逐行分析
### Lines 1-15: Import dependencies
```python
import unittest

import torch

from sglang.srt.layers.attention.fla.cumsum import chunk_local_cumsum
from sglang.srt.layers.attention.fla.fused_sigmoid_gating_recurrent import (
    fused_sigmoid_gating_delta_rule_update,
)
from sglang.srt.layers.attention.fla.index import prepare_chunk_indices
from sglang.srt.layers.attention.fla.kda import (
    fused_recurrent_kda,
    kda_gate_chunk_cumsum,
)
from sglang.srt.utils.common import get_device
from sglang.test.ci.ci_register import register_cuda_ci
```
**EN:** This block imports the libraries, fixtures, and helpers that the rest of the file relies on. It also registers the case for CUDA CI coverage.
**CN:** 该代码块导入了此文件后续逻辑依赖的库、夹具与辅助工具。 其中还会将该用例注册到 CUDA CI 覆盖范围。

### Lines 17-17: Register CI metadata
```python
register_cuda_ci(est_time=12, stage="base-b", runner_config="1-gpu-large")
```
**EN:** This block registers the test case in the CI matrix together with execution-time or runner metadata. It also registers the case for CUDA CI coverage.
**CN:** 该代码块把测试用例注册到 CI 矩阵中，并附带执行耗时或运行器元数据。 其中还会将该用例注册到 CUDA CI 覆盖范围。

### Lines 20-24: Define class TestKDAFusedSigmoidGatingRecurrent
```python
@unittest.skipIf(
    not (torch.cuda.is_available() or torch.xpu.is_available()),
    "Test requires CUDA or XPU",
)
class TestKDAFusedSigmoidGatingRecurrent(unittest.TestCase):
```
**EN:** This declaration introduces the `TestKDAFusedSigmoidGatingRecurrent` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestKDAFusedSigmoidGatingRecurrent` 测试类，并说明它通过继承承担的职责。

### Lines 25-81: Prepare per-test state (part 1)
```python
    def setUp(self):
        self.device = get_device()
        self.token_num = 4
        self.query_start_loc = torch.tensor([0, 1, 2, 3, 4], device=self.device)
        self.cache_indices = torch.tensor([0, 2, 5, 8], device=self.device)
        self.local_num_heads = 8
        self.head_dim = 128
        self.cache_len = 64

        self.A_log = torch.randn(
            1, 1, self.local_num_heads, 1, dtype=torch.float32, device=self.device
        )
        self.a = torch.randn(
            1,
            self.token_num,
            self.local_num_heads * self.head_dim,
            dtype=torch.bfloat16,
            device=self.device,
        )
        self.dt_bias = torch.randn(
            self.local_num_heads * self.head_dim,
            dtype=torch.bfloat16,
            device=self.device,
        )
        self.softplus_beta = 1.0
        self.softplus_threshold = 20.0
        self.q = torch.randn(
            1,
            self.token_num,
            self.local_num_heads,
            self.head_dim,
            dtype=torch.bfloat16,
            device=self.device,
        )
        self.k = torch.randn(
            1,
            self.token_num,
            self.local_num_heads,
            self.head_dim,
            dtype=torch.bfloat16,
            device=self.device,
        )
        self.v = torch.randn(
            1,
            self.token_num,
            self.local_num_heads,
            self.head_dim,
            dtype=torch.bfloat16,
            device=self.device,
        )
        self.beta = torch.randn(
            1,
            self.token_num,
            self.local_num_heads,
            dtype=torch.bfloat16,
            device=self.device,
        )
```
**EN:** This setup method initializes the state required before each individual test executes.
**CN:** 该初始化方法会在每个独立测试执行前准备所需状态。

### Lines 83-90: Prepare per-test state (part 2)
```python
        self.ssm_states = torch.zeros(
            self.cache_len,
            self.local_num_heads,
            self.head_dim,
            self.head_dim,
            dtype=torch.float32,
            device=self.device,
        )
```
**EN:** This setup method initializes the state required before each individual test executes.
**CN:** 该初始化方法会在每个独立测试执行前准备所需状态。

### Lines 92-110: Define helper: run fused
```python
    def run_fused(self):
        ssm_states = self.ssm_states.clone()
        core_attn_out = fused_sigmoid_gating_delta_rule_update(
            A_log=self.A_log,
            dt_bias=self.dt_bias,
            q=self.q,
            k=self.k,
            v=self.v,
            a=self.a,
            b=self.beta,
            initial_state_source=ssm_states,
            initial_state_indices=self.cache_indices,
            cu_seqlens=self.query_start_loc,
            use_qk_l2norm_in_kernel=True,
            softplus_beta=self.softplus_beta,
            softplus_threshold=self.softplus_threshold,
            is_kda=True,
        )
        return core_attn_out, ssm_states[self.cache_indices]
```
**EN:** This helper function encapsulates reusable logic inside `TestKDAFusedSigmoidGatingRecurrent` so the scenario stays organized.
**CN:** 该辅助函数封装了可复用逻辑，供 `TestKDAFusedSigmoidGatingRecurrent` 内部调用，从而让场景结构更清晰。

### Lines 112-134: Define helper: run kda
```python
    def run_kda(self):
        b = self.beta.float().sigmoid()
        # Reference gate activation using torch ops:
        #   g = -exp(A_log) * softplus(raw_g + dt_bias)
        H, K = self.local_num_heads, self.head_dim
        raw_g = self.a.float()  # [1, T, H*K]
        if self.dt_bias is not None:
            raw_g = raw_g + self.dt_bias.float()
        g = -torch.exp(
            self.A_log.float().view(1, 1, H, 1)
        ) * torch.nn.functional.softplus(raw_g.view(1, -1, H, K))
        initial_state = self.ssm_states[self.cache_indices].clone()
        core_attn_out, last_state = fused_recurrent_kda(
            q=self.q,
            k=self.k,
            v=self.v,
            g=g,
            beta=b,
            initial_state=initial_state,
            use_qk_l2norm_in_kernel=True,
            cu_seqlens=self.query_start_loc,
        )
        return core_attn_out, last_state
```
**EN:** This helper function encapsulates reusable logic inside `TestKDAFusedSigmoidGatingRecurrent` so the scenario stays organized.
**CN:** 该辅助函数封装了可复用逻辑，供 `TestKDAFusedSigmoidGatingRecurrent` 内部调用，从而让场景结构更清晰。

### Lines 136-145: Run test: kda fused sigmoid gating recurrent
```python
    def test_kda_fused_sigmoid_gating_recurrent(self):
        core_attn_out, last_state = self.run_fused()
        core_attn_out_ref, last_state_ref = self.run_kda()
        abs_diff_out = (core_attn_out - core_attn_out_ref).abs().max()
        abs_diff_state = (last_state - last_state_ref).abs().max()
        print(f"{abs_diff_out=}, {abs_diff_state=}")
        self.assertTrue(
            torch.allclose(core_attn_out, core_attn_out_ref, rtol=1e-3, atol=1e-4)
        )
        self.assertTrue(torch.allclose(last_state, last_state_ref))
```
**EN:** This test method exercises kda fused sigmoid gating recurrent and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 kda fused sigmoid gating recurrent 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 148-149: Define class TestKDAGateChunkCumsum
```python
@unittest.skipIf(not torch.cuda.is_available(), "Test requires CUDA")
class TestKDAGateChunkCumsum(unittest.TestCase):
```
**EN:** This declaration introduces the `TestKDAGateChunkCumsum` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestKDAGateChunkCumsum` 测试类，并说明它通过继承承担的职责。

### Lines 150-150: Document the class `TestKDAGateChunkCumsum`
```python
    """Test kda_gate_chunk_cumsum against torch reference (gate activation + cumsum)."""
```
**EN:** This block stores descriptive text that explains the intent of the surrounding class `TestKDAGateChunkCumsum`.
**CN:** 该代码块保存说明性文本，用于解释周围class `TestKDAGateChunkCumsum`的设计意图。

### Lines 152-152: Declare TestKDAGateChunkCumsum configuration
```python
    CHUNK_SIZE = 64
```
**EN:** This block defines class-level settings that are shared across the `TestKDAGateChunkCumsum` test methods.
**CN:** 该代码块定义了 `TestKDAGateChunkCumsum` 各测试方法共享的类级配置。

### Lines 154-168: Define helper: ref gate cumsum
```python
    def _ref_gate_cumsum(self, raw_g, A_log, dt_bias, cu_seqlens, chunk_size):
        """Reference: torch gate activation then chunk_local_cumsum."""
        B, T, H, K = raw_g.shape
        g = raw_g.float()
        if dt_bias is not None:
            g = g + dt_bias.float().view(1, 1, H, K)
        g = -torch.exp(A_log.float().view(1, 1, H, 1)) * torch.nn.functional.softplus(g)
        chunk_indices = (
            prepare_chunk_indices(cu_seqlens, chunk_size)
            if cu_seqlens is not None
            else None
        )
        return chunk_local_cumsum(
            g, chunk_size=chunk_size, cu_seqlens=cu_seqlens, chunk_indices=chunk_indices
        )
```
**EN:** This helper function encapsulates reusable logic inside `TestKDAGateChunkCumsum` so the scenario stays organized.
**CN:** 该辅助函数封装了可复用逻辑，供 `TestKDAGateChunkCumsum` 内部调用，从而让场景结构更清晰。

### Lines 170-201: Define helper: run case
```python
    def _run_case(self, B, T_per_seq, H, K, use_bias, use_varlen):
        T = B * T_per_seq
        torch.manual_seed(42)
        raw_g = torch.randn(1, T, H, K, dtype=torch.bfloat16, device="cuda")
        A_log = torch.randn(H, dtype=torch.float32, device="cuda") * 0.5
        dt_bias = (
            torch.randn(H * K, dtype=torch.float32, device="cuda") * 0.1
            if use_bias
            else None
        )
        cu_seqlens = (
            torch.arange(
                0, (B + 1) * T_per_seq, T_per_seq, dtype=torch.long, device="cuda"
            )
            if use_varlen
            else None
        )

        out_fused = kda_gate_chunk_cumsum(
            raw_g,
            A_log=A_log,
            chunk_size=self.CHUNK_SIZE,
            dt_bias=dt_bias,
            cu_seqlens=cu_seqlens,
        )
        out_ref = self._ref_gate_cumsum(
            raw_g, A_log, dt_bias, cu_seqlens, self.CHUNK_SIZE
        )

        max_diff = (out_fused - out_ref).abs().max().item()
        rel_diff = max_diff / (out_ref.abs().mean().item() + 1e-8)
        return max_diff, rel_diff
```
**EN:** This helper function encapsulates reusable logic inside `TestKDAGateChunkCumsum` so the scenario stays organized.
**CN:** 该辅助函数封装了可复用逻辑，供 `TestKDAGateChunkCumsum` 内部调用，从而让场景结构更清晰。

### Lines 203-209: Run test: varlen with bias
```python
    def test_varlen_with_bias(self):
        max_diff, rel_diff = self._run_case(
            B=4, T_per_seq=256, H=16, K=128, use_bias=True, use_varlen=True
        )
        self.assertLess(
            max_diff, 1e-3, f"max_diff={max_diff:.2e}, rel_diff={rel_diff:.2e}"
        )
```
**EN:** This test method exercises varlen with bias and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 varlen with bias 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 211-217: Run test: varlen no bias
```python
    def test_varlen_no_bias(self):
        max_diff, rel_diff = self._run_case(
            B=4, T_per_seq=256, H=16, K=128, use_bias=False, use_varlen=True
        )
        self.assertLess(
            max_diff, 1e-3, f"max_diff={max_diff:.2e}, rel_diff={rel_diff:.2e}"
        )
```
**EN:** This test method exercises varlen no bias and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 varlen no bias 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 219-225: Run test: fixed len with bias
```python
    def test_fixed_len_with_bias(self):
        max_diff, rel_diff = self._run_case(
            B=4, T_per_seq=256, H=16, K=128, use_bias=True, use_varlen=False
        )
        self.assertLess(
            max_diff, 1e-3, f"max_diff={max_diff:.2e}, rel_diff={rel_diff:.2e}"
        )
```
**EN:** This test method exercises fixed len with bias and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 fixed len with bias 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 227-233: Run test: single seq long
```python
    def test_single_seq_long(self):
        max_diff, rel_diff = self._run_case(
            B=1, T_per_seq=2048, H=16, K=128, use_bias=True, use_varlen=True
        )
        self.assertLess(
            max_diff, 1e-3, f"max_diff={max_diff:.2e}, rel_diff={rel_diff:.2e}"
        )
```
**EN:** This test method exercises single seq long and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 single seq long 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 235-241: Run test: small head dim
```python
    def test_small_head_dim(self):
        max_diff, rel_diff = self._run_case(
            B=4, T_per_seq=128, H=8, K=64, use_bias=True, use_varlen=True
        )
        self.assertLess(
            max_diff, 1e-3, f"max_diff={max_diff:.2e}, rel_diff={rel_diff:.2e}"
        )
```
**EN:** This test method exercises small head dim and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 small head dim 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

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

## Dependencies / 依赖关系
- Internal modules / 内部模块: `sglang.srt.layers.attention.fla.cumsum`, `sglang.srt.layers.attention.fla.fused_sigmoid_gating_recurrent`, `sglang.srt.layers.attention.fla.index`, `sglang.srt.layers.attention.fla.kda`, `sglang.srt.utils.common`, `sglang.test.ci.ci_register`
- External and stdlib modules / 外部与标准库模块: `torch`, `unittest`
- Notable symbols / 关键符号: `register_cuda_ci`, `unittest.main`
