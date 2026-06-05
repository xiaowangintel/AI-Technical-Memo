# test_mamba_state_scatter_triton.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/unit/layers/test_mamba_state_scatter_triton.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates mamba state scatter triton behavior in SGLang's unit / layers area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 单元 / 层 领域中与 mamba state scatter triton 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-1: module imports and dependencies / 模块导入与依赖
```python
from sglang.test.ci.ci_register import register_cuda_ci
```
**EN:** This block imports the modules needed by the rest of the file, including `sglang.test.ci.ci_register`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `sglang.test.ci.ci_register`。

### Lines 3-3: CI registration and metadata / CI 注册与元数据
```python
register_cuda_ci(est_time=7, stage="base-b", runner_config="1-gpu-small")
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through register_cuda_ci.
**CN:** 该代码块通过 register_cuda_ci 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 5-8: module imports and dependencies / 模块导入与依赖
```python
import os
import unittest

import torch
```
**EN:** This block imports the modules needed by the rest of the file, including `os`, `unittest`, `torch`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `os`, `unittest`, `torch`。

### Lines 10-18: module-level constants and configuration / 模块级常量与配置
```python
try:
    from sglang.srt.layers.attention.mamba.mamba_state_scatter_triton import (
        fused_mamba_state_scatter_with_mask,
    )

    _FUSED_IMPORT_ERROR = None
except Exception as e:  # pragma: no cover
    fused_mamba_state_scatter_with_mask = None
    _FUSED_IMPORT_ERROR = e
```
**EN:** This block defines shared names such as `_FUSED_IMPORT_ERROR`, `fused_mamba_state_scatter_with_mask`. These values centralize configuration that later tests or helpers reuse.
**CN:** 该代码块定义了 `_FUSED_IMPORT_ERROR`, `fused_mamba_state_scatter_with_mask` 等共享名称，用于集中保存后续测试或辅助逻辑会复用的配置。

### Lines 21-31: function dtype from str / 函数 dtype from str
```python
def _dtype_from_str(name: str) -> torch.dtype:
    mapping = {
        "bfloat16": torch.bfloat16,
        "float16": torch.float16,
        "float32": torch.float32,
    }
    if name not in mapping:
        raise ValueError(
            f"Unsupported dtype string {name!r}. Supported: {sorted(mapping.keys())}"
        )
    return mapping[name]
```
**EN:** This block implements `_dtype_from_str` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_dtype_from_str`，承担模块行为中的一个聚焦逻辑片段。

### Lines 34-38: function ref scatter / 函数 ref scatter
```python
def _ref_scatter(dst, src, dst_indices, src_indices, step_indices):
    """Reference implementation using PyTorch advanced indexing."""
    # dst: [L, C, E]
    # src: [L, S, D, E]
    dst[:, dst_indices] = src[:, src_indices, step_indices].to(dst.dtype, copy=False)
```
**EN:** Reference implementation using PyTorch advanced indexing. This block implements `_ref_scatter` and captures one focused piece of the module's behavior.
**CN:** Reference implementation using PyTorch advanced indexing. 该代码块实现 `_ref_scatter`，承担模块行为中的一个聚焦逻辑片段。

### Lines 41-103: function ref update like / 函数 ref update like
```python
def _ref_update_like(
    ssm_states,
    intermediate_ssm,
    conv_states,
    intermediate_conv,
    *,
    state_indices_tensor,
    step_indices_raw,
    mamba_track_indices=None,
    mamba_steps_to_track=None,
):
    """Reference implementation using PyTorch advanced indexing for correctness verification."""
    total_requests = step_indices_raw.shape[0]
    intermediate_state_indices = torch.arange(
        total_requests, dtype=torch.int32, device=step_indices_raw.device
    )

    valid_mask = step_indices_raw >= 0
    dst_state_indices = state_indices_tensor[valid_mask].to(torch.int64)
    src_state_indices = intermediate_state_indices[valid_mask].to(torch.int64)
    last_steps = step_indices_raw[valid_mask].to(torch.int64)

    # Only scatter if there are valid indices (but don't early return -
    # mamba_track_indices processing is independent)
    if dst_state_indices.numel() > 0:
        _ref_scatter(
            ssm_states,
            intermediate_ssm,
            dst_state_indices,
            src_state_indices,
            last_steps,
        )
        _ref_scatter(
            conv_states,
            intermediate_conv,
            dst_state_indices,
            src_state_indices,
            last_steps,
        )

    if mamba_track_indices is not None:
        assert mamba_steps_to_track is not None
        track_mask = mamba_steps_to_track >= 0
        if not track_mask.any():
            return
        dst_track_indices = mamba_track_indices[track_mask].to(torch.int64)
        src_track_indices = intermediate_state_indices[track_mask].to(torch.int64)
        track_steps = mamba_steps_to_track[track_mask].to(torch.int64)

        _ref_scatter(
            ssm_states,
            intermediate_ssm,
            dst_track_indices,
            src_track_indices,
            track_steps,
        )
        _ref_scatter(
            conv_states,
            intermediate_conv,
            dst_track_indices,
            src_track_indices,
            track_steps,
        )
```
**EN:** Reference implementation using PyTorch advanced indexing for correctness verification. This block implements `_ref_update_like` and captures one focused piece of the module's behavior.
**CN:** Reference implementation using PyTorch advanced indexing for correctness verification. 该代码块实现 `_ref_update_like`，承担模块行为中的一个聚焦逻辑片段。

### Lines 106-145: function fused update like / 函数 fused update like
```python
def _fused_update_like(
    ssm_states,
    intermediate_ssm,
    conv_states,
    intermediate_conv,
    *,
    state_indices_tensor,
    step_indices_raw,
    mamba_track_indices=None,
    mamba_steps_to_track=None,
):
    """Matches the fully fused logic that avoids index_select and nonzero calls."""
    # Use fully fused kernel that handles masking internally
    fused_mamba_state_scatter_with_mask(
        ssm_states,
        intermediate_ssm,
        state_indices_tensor,
        step_indices_raw,
    )
    fused_mamba_state_scatter_with_mask(
        conv_states,
        intermediate_conv,
        state_indices_tensor,
        step_indices_raw,
    )

    if mamba_track_indices is not None:
        assert mamba_steps_to_track is not None
        fused_mamba_state_scatter_with_mask(
            ssm_states,
            intermediate_ssm,
            mamba_track_indices,
            mamba_steps_to_track,
        )
        fused_mamba_state_scatter_with_mask(
            conv_states,
            intermediate_conv,
            mamba_track_indices,
            mamba_steps_to_track,
        )
```
**EN:** Matches the fully fused logic that avoids index_select and nonzero calls. This block implements `_fused_update_like` and captures one focused piece of the module's behavior.
**CN:** Matches the fully fused logic that avoids index_select and nonzero calls. 该代码块实现 `_fused_update_like`，承担模块行为中的一个聚焦逻辑片段。

### Lines 148-161: function time cuda ms / 函数 time cuda ms
```python
def _time_cuda_ms(fn, iters=50, warmup=10):
    """Measure average CUDA time (ms) using CUDA events."""
    for _ in range(warmup):
        fn()
    torch.cuda.synchronize()

    start = torch.cuda.Event(enable_timing=True)
    end = torch.cuda.Event(enable_timing=True)
    start.record()
    for _ in range(iters):
        fn()
    end.record()
    torch.cuda.synchronize()
    return start.elapsed_time(end) / iters
```
**EN:** Measure average CUDA time (ms) using CUDA events. This block implements `_time_cuda_ms` and captures one focused piece of the module's behavior.
**CN:** Measure average CUDA time (ms) using CUDA events. 该代码块实现 `_time_cuda_ms`，承担模块行为中的一个聚焦逻辑片段。

### Lines 164-164: class TestMambaStateScatterCorrectness declaration / 类 TestMambaStateScatterCorrectness 声明
```python
class TestMambaStateScatterCorrectness(unittest.TestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `unittest.TestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `unittest.TestCase`。

### Lines 165-242: test case fused matches reference / 测试用例 fused matches reference
```python
    @unittest.skipUnless(torch.cuda.is_available(), "CUDA is required for this test.")
    def test_fused_matches_reference(self):
        """Test that fused_mamba_state_scatter_with_mask matches the reference."""
        if fused_mamba_state_scatter_with_mask is None:
            self.skipTest(
                f"fused_mamba_state_scatter_with_mask import failed: {_FUSED_IMPORT_ERROR}"
            )

        torch.manual_seed(42)
        device = torch.device("cuda")

        # Keep sizes moderate so this test is quick.
        L = 8
        B = 32
        C = 49
        D = 5
        ssm_elems = 1024
        conv_elems = 512

        ssm_states0 = torch.randn(
            (L, C, ssm_elems), device=device, dtype=torch.bfloat16
        )
        conv_states0 = torch.randn(
            (L, C, conv_elems), device=device, dtype=torch.bfloat16
        )
        intermediate_ssm = torch.randn(
            (L, B, D, ssm_elems), device=device, dtype=torch.bfloat16
        )
        intermediate_conv = torch.randn(
            (L, B, D, conv_elems), device=device, dtype=torch.bfloat16
        )

        # unique cache lines (no duplicates) to avoid nondeterministic write order
        state_indices_tensor = torch.randperm(C, device=device, dtype=torch.int64)[
            :B
        ].to(torch.int32)

        step_indices_raw = torch.randint(0, D, (B,), device=device, dtype=torch.int64)
        # set ~10% invalid
        invalid = torch.rand((B,), device=device) < 0.1
        step_indices_raw[invalid] = -1

        # Optional track update
        mamba_track_indices = torch.randperm(C, device=device, dtype=torch.int64)[:B]
        mamba_steps_to_track = torch.randint(
            0, D, (B,), device=device, dtype=torch.int64
        )
        track_invalid = torch.rand((B,), device=device) < 0.7
        mamba_steps_to_track[track_invalid] = -1

        ssm_ref = ssm_states0.clone()
        conv_ref = conv_states0.clone()
        ssm_fused = ssm_states0.clone()
        conv_fused = conv_states0.clone()

        _ref_update_like(
            ssm_ref,
            intermediate_ssm,
            conv_ref,
            intermediate_conv,
            state_indices_tensor=state_indices_tensor,
            step_indices_raw=step_indices_raw,
            mamba_track_indices=mamba_track_indices,
            mamba_steps_to_track=mamba_steps_to_track,
        )
        _fused_update_like(
            ssm_fused,
            intermediate_ssm,
            conv_fused,
            intermediate_conv,
            state_indices_tensor=state_indices_tensor,
            step_indices_raw=step_indices_raw,
            mamba_track_indices=mamba_track_indices,
            mamba_steps_to_track=mamba_steps_to_track,
        )

        torch.testing.assert_close(ssm_fused, ssm_ref)
        torch.testing.assert_close(conv_fused, conv_ref)
```
**EN:** Test that fused_mamba_state_scatter_with_mask matches the reference. This test exercises `test_fused_matches_reference` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that fused_mamba_state_scatter_with_mask matches the reference. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_fused_matches_reference`。

### Lines 245-245: class TestMambaStateScatterPerf declaration / 类 TestMambaStateScatterPerf 声明
```python
class TestMambaStateScatterPerf(unittest.TestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `unittest.TestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `unittest.TestCase`。

### Lines 246-325: test case perf report old vs fused (part 1/2) / 测试用例 perf report old vs fused（第 1/2 部分）
```python
    @unittest.skipUnless(torch.cuda.is_available(), "CUDA is required for this test.")
    def test_perf_report_old_vs_fused(self):
        """Optional microbenchmark comparing baseline vs fused kernel.

        Enable with: SGLANG_RUN_MAMBA_SCATTER_PERF_TEST=1
        """
        if os.environ.get("SGLANG_RUN_MAMBA_SCATTER_PERF_TEST", "0") != "1":
            self.skipTest("Set SGLANG_RUN_MAMBA_SCATTER_PERF_TEST=1 to run perf test.")
        if fused_mamba_state_scatter_with_mask is None:
            self.skipTest(
                f"fused_mamba_state_scatter_with_mask import failed: {_FUSED_IMPORT_ERROR}"
            )

        torch.manual_seed(0)
        device = torch.device("cuda")

        # Parameterize sizes via env vars so we can match a real model more closely.
        L = int(os.environ.get("SGLANG_MAMBA_SCATTER_LAYERS", "32"))
        B = int(os.environ.get("SGLANG_MAMBA_SCATTER_BATCH", "48"))
        C = int(os.environ.get("SGLANG_MAMBA_SCATTER_CACHE", "49"))
        D = int(os.environ.get("SGLANG_MAMBA_SCATTER_DRAFT_TOKENS", "5"))
        ssm_elems = int(os.environ.get("SGLANG_MAMBA_SCATTER_SSM_ELEMS", "4096"))
        conv_elems = int(os.environ.get("SGLANG_MAMBA_SCATTER_CONV_ELEMS", "512"))
        invalid_ratio = float(
            os.environ.get("SGLANG_MAMBA_SCATTER_INVALID_RATIO", "0.0")
        )
        track_ratio = float(os.environ.get("SGLANG_MAMBA_SCATTER_TRACK_RATIO", "0.0"))
        ssm_dtype = _dtype_from_str(
            os.environ.get("SGLANG_MAMBA_SCATTER_SSM_DTYPE", "bfloat16")
        )
        conv_dtype = _dtype_from_str(
            os.environ.get("SGLANG_MAMBA_SCATTER_CONV_DTYPE", "bfloat16")
        )

        # Use zeros for dst so each iteration overwrites the same memory.
        ssm_states = torch.zeros((L, C, ssm_elems), device=device, dtype=ssm_dtype)
        conv_states = torch.zeros((L, C, conv_elems), device=device, dtype=conv_dtype)
        intermediate_ssm = torch.randn(
            (L, B, D, ssm_elems), device=device, dtype=ssm_dtype
        )
        intermediate_conv = torch.randn(
            (L, B, D, conv_elems), device=device, dtype=conv_dtype
        )

        state_indices_tensor = torch.randperm(C, device=device, dtype=torch.int64)[
            :B
        ].to(torch.int32)
        step_indices_raw = torch.randint(0, D, (B,), device=device, dtype=torch.int64)
        if invalid_ratio > 0:
            invalid = torch.rand((B,), device=device) < invalid_ratio
            step_indices_raw[invalid] = -1

        mamba_track_indices = None
        mamba_steps_to_track = None
        if track_ratio > 0:
            mamba_track_indices = torch.randperm(C, device=device, dtype=torch.int64)[
                :B
            ]
            mamba_steps_to_track = torch.randint(
                0, D, (B,), device=device, dtype=torch.int64
            )
            track_invalid = torch.rand((B,), device=device) >= track_ratio
            mamba_steps_to_track[track_invalid] = -1

        def ref_fn():
            _ref_update_like(
                ssm_states,
                intermediate_ssm,
                conv_states,
                intermediate_conv,
                state_indices_tensor=state_indices_tensor,
                step_indices_raw=step_indices_raw,
                mamba_track_indices=mamba_track_indices,
                mamba_steps_to_track=mamba_steps_to_track,
            )

        def fused_fn():
            _fused_update_like(
                ssm_states,
                intermediate_ssm,
```
**EN:** Optional microbenchmark comparing baseline vs fused kernel. This test exercises `test_perf_report_old_vs_fused` by arranging inputs, invoking the relevant path, and checking the expected outcome. This chunk continues part 1 of the same logical block.
**CN:** Optional microbenchmark comparing baseline vs fused kernel. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_perf_report_old_vs_fused`。 这一段对应同一逻辑块的第 1 部分。

### Lines 326-354: test case perf report old vs fused (part 2/2) / 测试用例 perf report old vs fused（第 2/2 部分）
```python
                conv_states,
                intermediate_conv,
                state_indices_tensor=state_indices_tensor,
                step_indices_raw=step_indices_raw,
                mamba_track_indices=mamba_track_indices,
                mamba_steps_to_track=mamba_steps_to_track,
            )

        # Warm up JIT compilation for triton kernels (and caches for torch indexing)
        ref_fn()
        fused_fn()
        torch.cuda.synchronize()

        ref_ms = _time_cuda_ms(ref_fn)
        fused_ms = _time_cuda_ms(fused_fn)

        num_valid = int((step_indices_raw >= 0).sum().item())
        ratio = fused_ms / ref_ms if ref_ms > 0 else float("inf")
        speedup = ref_ms / fused_ms if fused_ms > 0 else float("inf")

        # Print a concise report
        print(
            "\n[MambaStateScatterPerf]\n"
            f"  shapes: L={L} B={B} C={C} D={D} ssm_elems={ssm_elems} conv_elems={conv_elems}\n"
            f"  dtypes: ssm={ssm_dtype} conv={conv_dtype}\n"
            f"  valid: {num_valid}/{B}  invalid_ratio={invalid_ratio}  track_ratio={track_ratio}\n"
            f"  ref_total_ms (baseline):  {ref_ms:.4f}\n"
            f"  fused_total_ms:           {fused_ms:.4f}  (ratio={ratio:.3f}x, speedup={speedup:.2f}x)\n"
        )
```
**EN:** Optional microbenchmark comparing baseline vs fused kernel. This test exercises `test_perf_report_old_vs_fused` by arranging inputs, invoking the relevant path, and checking the expected outcome. This chunk continues part 2 of the same logical block.
**CN:** Optional microbenchmark comparing baseline vs fused kernel. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_perf_report_old_vs_fused`。 这一段对应同一逻辑块的第 2 部分。

### Lines 357-358: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":  # pragma: no cover
    unittest.main()
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `_dtype_from_str`: This block implements `_dtype_from_str` and captures one focused piece of the module's behavior. / 该代码块实现 `_dtype_from_str`，承担模块行为中的一个聚焦逻辑片段。
- `_ref_scatter`: Reference implementation using PyTorch advanced indexing. / 该代码块实现 `_ref_scatter`，承担模块行为中的一个聚焦逻辑片段。
- `_ref_update_like`: Reference implementation using PyTorch advanced indexing for correctness verification. / 该代码块实现 `_ref_update_like`，承担模块行为中的一个聚焦逻辑片段。
- `_fused_update_like`: Matches the fully fused logic that avoids index_select and nonzero calls. / 该代码块实现 `_fused_update_like`，承担模块行为中的一个聚焦逻辑片段。
- `_time_cuda_ms`: Measure average CUDA time (ms) using CUDA events. / 该代码块实现 `_time_cuda_ms`，承担模块行为中的一个聚焦逻辑片段。
- `TestMambaStateScatterCorrectness`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestMambaStateScatterPerf`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestMambaStateScatterCorrectness.test_fused_matches_reference`: Test that fused_mamba_state_scatter_with_mask matches the reference. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_fused_matches_reference`。
- `TestMambaStateScatterPerf.test_perf_report_old_vs_fused`: Optional microbenchmark comparing baseline vs fused kernel. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_perf_report_old_vs_fused`。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `os`, `unittest`
- **Third-party modules / 第三方模块**: `torch`
- **Internal modules / 内部模块**: `sglang.test.ci.ci_register`

- **Total lines / 总行数**: 358
