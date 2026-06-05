# test_awq_int4_to_int8.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/kernels/test_awq_int4_to_int8.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Pytest module for kernel-level behavior in kernels / test_awq_int4_to_int8, focusing on correctness, edge cases, and comparisons against trusted reference paths. / 面向 kernels / test_awq_int4_to_int8 的内核级 pytest 模块，重点覆盖正确性、边界情况以及与可信参考路径的对照。

## Line-by-Line Analysis / 逐行分析
### Module docstring (lines 3-17)
```python
"""
Unit tests for AWQ INT4 W4A8 GEMM pipeline (SGLang kernel migration).

Part 1: Weight packing tests
  - convert_weight_packed_scale_zp correctness

Part 2: INT4 W4A8 GEMM tests
  - int4_scaled_mm_cpu correctness w.r.t. float reference
  - Bias, 3D input, various shapes

Part 3: create_weights shapes

cmd:
    VLLM_CPU_INT4_W4A8=1 python -m pytest tests/kernels/test_awq_int4_to_int8.py -v -s
"""
```
**EN:** This docstring states the file-level intent and helps readers understand the role of the module before they inspect the implementation.
**CN:** 这个文档字符串说明了文件级意图，让读者在查看实现前先理解模块职责。

### Imports and shared setup (lines 19-31)
```python
import numpy as np
import pytest
import torch

from vllm._custom_ops import (
    CPUQuantAlgo,
    convert_weight_packed_scale_zp,
    int4_scaled_mm_cpu,
)
from vllm.model_executor.layers.quantization.utils.quant_utils import (
    pack_cols,
)
from vllm.platforms import current_platform
```
**EN:** This block imports the modules that the rest of the file relies on. It pulls in external libraries such as numpy, pytest, torch; and vLLM components like vllm._custom_ops, vllm.model_executor.layers.quantization.utils.quant_utils, vllm.platforms.
**CN:** 该代码块导入后续逻辑依赖的模块。 主要包括外部库，例如 numpy、pytest、torch；vLLM 内部组件，例如 vllm._custom_ops、vllm.model_executor.layers.quantization.utils.quant_utils、vllm.platforms。

### Top-level block starting at line 33 (lines 33-34)
```python
if not current_platform.is_cpu():
    pytest.skip("skipping CPU-only tests", allow_module_level=True)
```
**EN:** This top-level `If` block performs supporting work that the surrounding tests depend on.
**CN:** 这个顶层 `If` 代码块执行周边测试所依赖的辅助逻辑。

### Function `make_awq_checkpoint_data` (lines 37-83)
```python
def make_awq_checkpoint_data(K, N, group_size, seed=42):
    """Create synthetic AWQ checkpoint data in packed int32 format.

    Returns:
        packed_qweight: [K, N//8] int32 (AWQ interleaved + packed)
        packed_qzeros:  [num_groups, N//8] int32 (AWQ interleaved + packed)
        scales:         [num_groups, N] float32
        float_ref:      [K, N] float32, reference dequantized weights
        weight_int4_orig: [K, N] int32, original int4 values (0-15)
        zeros_int4_orig:  [num_groups, N] int32, original zero points (0-15)
    """
    rng = np.random.RandomState(seed)
    num_groups = K // group_size

    weight_int4_orig = torch.from_numpy(
        rng.randint(0, 16, size=(K, N)).astype(np.int32)
    )
    zeros_int4_orig = torch.from_numpy(
        rng.randint(0, 16, size=(num_groups, N)).astype(np.int32)
    )
    scales = torch.from_numpy((rng.randn(num_groups, N) * 0.05).astype(np.float32))

    scales_exp = scales.repeat_interleave(group_size, dim=0)
    zeros_exp = zeros_int4_orig.repeat_interleave(group_size, dim=0)
    float_ref = (weight_int4_orig.float() - zeros_exp.float()) * scales_exp

    awq_interleave = [0, 2, 4, 6, 1, 3, 5, 7]
    weight_interleaved = (
        weight_int4_orig.reshape(-1, 8)[:, awq_interleave].reshape(K, N).contiguous()
    )
    packed_qweight = pack_cols(weight_interleaved, 4, K, N)

    zeros_interleaved = (
        zeros_int4_orig.reshape(-1, 8)[:, awq_interleave]
        .reshape(num_groups, N)
        .contiguous()
    )
    packed_qzeros = pack_cols(zeros_interleaved, 4, num_groups, N)

    return (
        packed_qweight,
        packed_qzeros,
        scales,
        float_ref,
        weight_int4_orig,
        zeros_int4_orig,
    )
```
**EN:** This helper function implements the shared logic for awq checkpoint data. randomness is controlled so the scenario remains reproducible.
**CN:** 该辅助函数实现了 awq checkpoint data 所需的共享逻辑。 代码会控制随机性以保证场景可复现。

### Class `TestConvertWeightPackedScaleZp` (lines 86-88)
```python
class TestConvertWeightPackedScaleZp:
    """Tests for convert_weight_packed_scale_zp weightpacking."""
```
**EN:** This helper class groups the state and behavior needed for TestConvertWeightPackedScaleZp. That structure makes repeated setup, graph inspection, or reusable assertions easier to manage.
**CN:** 这个辅助类把 TestConvertWeightPackedScaleZp 所需的状态与行为收拢到一起，使重复初始化、计算图检查或可复用断言更容易管理。

### Method `TestConvertWeightPackedScaleZp.test_packing_output_shapes` (lines 89-127)
```python
    @pytest.mark.parametrize(
        "K,N,group_size",
        [
            (128, 128, 128),
            (256, 256, 128),
            (512, 256, 64),
        ],
    )
    def test_packing_output_shapes(self, K, N, group_size):
        """Packed outputs should have expected shapes."""
        (packed_qweight, packed_qzeros, scales, _, _, _) = make_awq_checkpoint_data(
            K, N, group_size
        )

        blocked_w, blocked_zp, blocked_s = convert_weight_packed_scale_zp(
            packed_qweight,
            packed_qzeros,
            scales,
            CPUQuantAlgo.AWQ,
        )

        block_n = 32
        Nc = N // block_n

        assert blocked_w.dim() >= 2, (
            f"blocked_w should have >= 2 dims, got {blocked_w.dim()}"
        )
        assert blocked_s.size(0) == Nc, (
            f"Expected Nc={Nc} scale blocks, got {blocked_s.size(0)}"
        )
        assert blocked_zp.size(0) == Nc, (
            f"Expected Nc={Nc} qzeros blocks, got {blocked_zp.size(0)}"
        )

        print(
            f"  [PASS] packing shapes K={K}, N={N}, gs={group_size}: "
            f"blocked_w={list(blocked_w.shape)}, "
            f"blocked_s={list(blocked_s.shape)}, blocked_zp={list(blocked_zp.shape)}"
        )
```
**EN:** This method on `TestConvertWeightPackedScaleZp` checks packing output shapes. It is parameterized across 1 input dimension so one definition covers many scenarios. it consumes fixtures or inputs such as K, N, group_size. assertions at the end lock in the intended behavior or graph shape.
**CN:** `TestConvertWeightPackedScaleZp` 中的这个方法用于检查 packing output shapes。 它通过 1 组参数化输入覆盖多种场景；它会使用诸如 K、N、group_size 等 fixture 或输入；结尾处的断言会固定预期行为或计算图形态。

### Class `TestInt4ScaledMmCpu` (lines 130-132)
```python
class TestInt4ScaledMmCpu:
    """Tests for int4_scaled_mm_cpu GEMM kernel."""
```
**EN:** This helper class groups the state and behavior needed for TestInt4ScaledMmCpu. That structure makes repeated setup, graph inspection, or reusable assertions easier to manage.
**CN:** 这个辅助类把 TestInt4ScaledMmCpu 所需的状态与行为收拢到一起，使重复初始化、计算图检查或可复用断言更容易管理。

### Method `TestInt4ScaledMmCpu.test_gemm_vs_float_reference` (lines 133-173)
```python
    @pytest.mark.parametrize(
        "M,K,N,group_size",
        [
            (1, 128, 128, 128),
            (4, 256, 256, 128),
            (16, 512, 256, 64),
            (32, 256, 512, 128),
            (64, 512, 512, 128),
        ],
    )
    def test_gemm_vs_float_reference(self, M, K, N, group_size):
        """INT4 W4A8 GEMM should approximate float matmul."""
        (packed_qweight, packed_qzeros, scales, float_ref, _, _) = (
            make_awq_checkpoint_data(K, N, group_size)
        )

        blocked_w, blocked_zp, blocked_s = convert_weight_packed_scale_zp(
            packed_qweight,
            packed_qzeros,
            scales,
            CPUQuantAlgo.AWQ,
        )

        x = torch.randn(M, K, dtype=torch.bfloat16)
        out = int4_scaled_mm_cpu(x, blocked_w, blocked_zp, blocked_s, None)

        ref_out = torch.mm(x.float(), float_ref)

        abs_diff = (out.float() - ref_out).abs()
        mean_abs = abs_diff.mean().item()
        pct95 = torch.quantile(abs_diff, 0.95).item()
        ref_mag = ref_out.abs().mean().item() + 1e-6
        mean_rel = mean_abs / ref_mag

        assert mean_rel < 0.05, (
            f"Mean relative error {mean_rel:.4f} exceeds 5% threshold"
        )
        assert pct95 < ref_mag * 0.15, (
            f"95th-pctile abs_diff {pct95:.4f} exceeds 15% of ref magnitude"
        )
        print(f"  [PASS] INT4 GEMM correct: M={M}, K={K}, N={N}")
```
**EN:** This method on `TestInt4ScaledMmCpu` checks gemm vs float reference. It is parameterized across 1 input dimension so one definition covers many scenarios. it consumes fixtures or inputs such as M, K, N, group_size. assertions at the end lock in the intended behavior or graph shape.
**CN:** `TestInt4ScaledMmCpu` 中的这个方法用于检查 gemm vs float reference。 它通过 1 组参数化输入覆盖多种场景；它会使用诸如 M、K、N、group_size 等 fixture 或输入；结尾处的断言会固定预期行为或计算图形态。

### Method `TestInt4ScaledMmCpu.test_gemm_with_bias` (lines 175-203)
```python
    @pytest.mark.parametrize("M", [1, 8, 32])
    def test_gemm_with_bias(self, M):
        """INT4 W4A8 GEMM with bias should match reference."""
        K, N, group_size = 256, 128, 128
        (packed_qweight, packed_qzeros, scales, float_ref, _, _) = (
            make_awq_checkpoint_data(K, N, group_size)
        )

        blocked_w, blocked_zp, blocked_s = convert_weight_packed_scale_zp(
            packed_qweight,
            packed_qzeros,
            scales,
            CPUQuantAlgo.AWQ,
        )

        bias = torch.randn(N, dtype=torch.float32)
        x = torch.randn(M, K, dtype=torch.bfloat16)

        out = int4_scaled_mm_cpu(x, blocked_w, blocked_zp, blocked_s, bias)

        ref_out = torch.mm(x.float(), float_ref) + bias
        abs_diff = (out.float() - ref_out).abs()
        mean_abs = abs_diff.mean().item()
        ref_mag = ref_out.abs().mean().item() + 1e-6
        mean_rel = mean_abs / ref_mag
        assert mean_rel < 0.05, (
            f"Mean relative error {mean_rel:.4f} with bias exceeds 5%"
        )
        print(f"  [PASS] INT4 GEMM with bias: M={M}")
```
**EN:** This method on `TestInt4ScaledMmCpu` checks gemm with bias. It is parameterized across 1 input dimension so one definition covers many scenarios. it consumes fixtures or inputs such as M. assertions at the end lock in the intended behavior or graph shape.
**CN:** `TestInt4ScaledMmCpu` 中的这个方法用于检查 gemm with bias。 它通过 1 组参数化输入覆盖多种场景；它会使用诸如 M 等 fixture 或输入；结尾处的断言会固定预期行为或计算图形态。

### Method `TestInt4ScaledMmCpu.test_gemm_3d_input` (lines 205-235)
```python
    def test_gemm_3d_input(self):
        """apply() reshapes 3D input [B, S, K] -> [B*S, K] -> back to 3D."""
        K, N, group_size = 256, 128, 128
        (packed_qweight, packed_qzeros, scales, float_ref, _, _) = (
            make_awq_checkpoint_data(K, N, group_size)
        )

        blocked_w, blocked_zp, blocked_s = convert_weight_packed_scale_zp(
            packed_qweight,
            packed_qzeros,
            scales,
            CPUQuantAlgo.AWQ,
        )

        B, S = 2, 8
        x_3d = torch.randn(B, S, K, dtype=torch.bfloat16)
        x_2d = x_3d.reshape(-1, K)

        out_2d = int4_scaled_mm_cpu(x_2d, blocked_w, blocked_zp, blocked_s, None)
        out_3d = out_2d.reshape(B, S, N)

        ref_out = torch.mm(x_2d.float(), float_ref).reshape(B, S, N)

        assert out_3d.shape == (B, S, N)
        abs_diff = (out_3d.float() - ref_out).abs()
        mean_abs = abs_diff.mean().item()
        ref_mag = ref_out.abs().mean().item() + 1e-6
        mean_rel = mean_abs / ref_mag

        assert mean_rel < 0.05, f"Mean relative error {mean_rel:.4f} for 3D exceeds 5%"
        print(f"  [PASS] 3D input [{B},{S},{K}] -> output [{B},{S},{N}]")
```
**EN:** This method on `TestInt4ScaledMmCpu` checks gemm 3d input. assertions at the end lock in the intended behavior or graph shape.
**CN:** `TestInt4ScaledMmCpu` 中的这个方法用于检查 gemm 3d input。 结尾处的断言会固定预期行为或计算图形态。

### Method `TestInt4ScaledMmCpu.test_gemm_fp16_input` (lines 237-262)
```python
    def test_gemm_fp16_input(self):
        """INT4 GEMM should also work with fp16 input."""
        K, N, group_size, M = 256, 256, 128, 8
        (packed_qweight, packed_qzeros, scales, float_ref, _, _) = (
            make_awq_checkpoint_data(K, N, group_size)
        )

        blocked_w, blocked_zp, blocked_s = convert_weight_packed_scale_zp(
            packed_qweight,
            packed_qzeros,
            scales,
            CPUQuantAlgo.AWQ,
        )

        x = torch.randn(M, K, dtype=torch.float16)
        out = int4_scaled_mm_cpu(x, blocked_w, blocked_zp, blocked_s, None)

        ref_out = torch.mm(x.float(), float_ref)
        abs_diff = (out.float() - ref_out).abs()
        ref_mag = ref_out.abs().mean().item() + 1e-6
        mean_rel = abs_diff.mean().item() / ref_mag

        assert mean_rel < 0.05, (
            f"Mean relative error {mean_rel:.4f} for fp16 exceeds 5%"
        )
        print(f"  [PASS] fp16 input M={M}, K={K}, N={N}")
```
**EN:** This method on `TestInt4ScaledMmCpu` checks gemm FP16 input. assertions at the end lock in the intended behavior or graph shape.
**CN:** `TestInt4ScaledMmCpu` 中的这个方法用于检查 gemm FP16 input。 结尾处的断言会固定预期行为或计算图形态。

### Class `TestCreateWeightsUnchanged` (lines 265-267)
```python
class TestCreateWeightsUnchanged:
    """Create_weights should still produce correct int4 placeholder shapes."""
```
**EN:** This helper class groups the state and behavior needed for TestCreateWeightsUnchanged. That structure makes repeated setup, graph inspection, or reusable assertions easier to manage.
**CN:** 这个辅助类把 TestCreateWeightsUnchanged 所需的状态与行为收拢到一起，使重复初始化、计算图检查或可复用断言更容易管理。

### Method `TestCreateWeightsUnchanged.test_int4_placeholder_shapes` (lines 268-288)
```python
    @pytest.mark.parametrize(
        "K,N,group_size",
        [
            (128, 128, 128),
            (256, 256, 128),
            (512, 256, 64),
        ],
    )
    def test_int4_placeholder_shapes(self, K, N, group_size):
        """Verify qweight, qzeros, scales shapes."""
        pack_factor = 8
        num_groups = K // group_size

        qweight = torch.empty(K, N // pack_factor, dtype=torch.int32)
        qzeros = torch.empty(num_groups, N // pack_factor, dtype=torch.int32)
        scales = torch.empty(num_groups, N, dtype=torch.bfloat16)

        assert qweight.shape == (K, N // pack_factor)
        assert qzeros.shape == (num_groups, N // pack_factor)
        assert scales.shape == (num_groups, N)
        print(f"  [PASS] create_weights shapes: K={K}, N={N}, gs={group_size}")
```
**EN:** This method on `TestCreateWeightsUnchanged` checks int4 placeholder shapes. It is parameterized across 1 input dimension so one definition covers many scenarios. it consumes fixtures or inputs such as K, N, group_size. assertions at the end lock in the intended behavior or graph shape.
**CN:** `TestCreateWeightsUnchanged` 中的这个方法用于检查 int4 placeholder shapes。 它通过 1 组参数化输入覆盖多种场景；它会使用诸如 K、N、group_size 等 fixture 或输入；结尾处的断言会固定预期行为或计算图形态。

## Key Concepts / 关键概念
- **Pytest parameterization / pytest 参数化:** The module expands one definition into many concrete scenarios through parametrized inputs. / 该模块通过参数化输入把单个定义扩展成多组具体场景。
- **Behavioral assertions / 行为断言:** Assertions check tensor values, graph structure, or expected error conditions. / 断言用于检查张量结果、计算图结构或预期的报错路径。
- **Platform guards / 平台保护:** Hardware, backend, or version checks prevent unsupported combinations from running. / 硬件、后端或版本检查会阻止不受支持的组合继续执行。
- **Reference comparisons / 参考实现对照:** Optimized kernels are commonly checked against simpler reference implementations or fallback paths. / 优化后的内核通常会与更直接的参考实现或回退路径进行比对。

## Dependencies / 依赖关系
- `numpy`
- `pytest`
- `torch`
- `vllm._custom_ops -> CPUQuantAlgo, convert_weight_packed_scale_zp, int4_scaled_mm_cpu`
- `vllm.model_executor.layers.quantization.utils.quant_utils -> pack_cols`
- `vllm.platforms -> current_platform`
