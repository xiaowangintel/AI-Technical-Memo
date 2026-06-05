# test_aiter_allreduce_fusion_amd.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/ops/test_aiter_allreduce_fusion_amd.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates aiter allreduce fusion amd behavior in SGLang's ops area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 ops 领域中与 aiter allreduce fusion amd 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-11: module imports and dependencies / 模块导入与依赖
```python
import csv
import os
import subprocess
import sys
import tempfile
import unittest
from pathlib import Path

import torch

from sglang.test.ci.ci_register import register_amd_ci
```
**EN:** This block imports the modules needed by the rest of the file, including `csv`, `os`, `subprocess`, `sys`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `csv`, `os`, `subprocess`, `sys`。

### Lines 13-15: CI registration and metadata / CI 注册与元数据
```python
register_amd_ci(est_time=240, suite="stage-c-test-large-8-gpu-amd")

HIDDEN_DIMS = [2880, 4096, 5120, 6144, 7168, 8192]
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through register_amd_ci.
**CN:** 该代码块通过 register_amd_ci 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 18-97: function run residual accuracy check (part 1/2) / 函数 run residual accuracy check（第 1/2 部分）
```python
def _run_residual_accuracy_check():
    """Distributed entry point: bit-exact residual accuracy across 1-stage/2-stage.

    Regression test for the 1-stage kernel accuracy bug (ROCm/aiter#2586):
    allreduce_fusion_kernel_1stage accumulated in f32 and added the residual
    before rounding to bf16, while the unfused path rounds allreduce to bf16
    first.  The 1-ULP divergence compounded across layers and caused a -2.6pp
    GSM8K regression.

    Must be launched via torchrun (multi-GPU).
    """
    import torch.distributed as dist

    from sglang.srt.distributed.communication_op import (
        tensor_model_parallel_all_reduce,
        tensor_model_parallel_fused_allreduce_rmsnorm,
    )
    from sglang.srt.distributed.parallel_state import (
        destroy_distributed_environment,
        destroy_model_parallel,
        init_distributed_environment,
        initialize_model_parallel,
        set_custom_all_reduce,
    )

    rank = int(os.environ.get("RANK", "0"))
    world_size = int(os.environ.get("WORLD_SIZE", "1"))
    local_rank = int(os.environ.get("LOCAL_RANK", str(rank)))
    torch.cuda.set_device(local_rank % torch.cuda.device_count())
    device = torch.device(f"cuda:{local_rank % torch.cuda.device_count()}")

    set_custom_all_reduce(True)
    init_distributed_environment(
        world_size=world_size,
        rank=rank,
        local_rank=local_rank,
        distributed_init_method="env://",
        backend="nccl",
    )
    initialize_model_parallel(tensor_model_parallel_size=world_size)

    dtype = torch.bfloat16
    eps = 1e-6

    all_pass = True
    test_cases = [(m, n) for n in HIDDEN_DIMS for m in [1, 4, 8, 16, 32, 64, 128]]

    prev_n = None
    for m, n in test_cases:
        if n != prev_n:
            prev_n = n
            weight = torch.ones((n,), dtype=dtype, device=device)
            if rank == 0:
                print(f"\nhidden_dim={n}:")

        torch.manual_seed(1234 + rank * 17 + m)
        x = torch.randn((m, n), dtype=torch.float32, device=device).to(dtype)
        residual = torch.randn((m, n), dtype=torch.float32, device=device).to(dtype)
        zero_res = torch.zeros((m, n), dtype=dtype, device=device)

        dist.barrier()
        torch.cuda.synchronize()

        fused_zero = tensor_model_parallel_fused_allreduce_rmsnorm(
            x.clone(), zero_res.clone(), weight, eps
        )
        torch.cuda.synchronize()
        if fused_zero is None:
            if rank == 0:
                print(f"  {m:>5d}x{n}: SKIP (fused unavailable)")
            continue
        _, fused_ar = fused_zero

        dist.barrier()
        torch.cuda.synchronize()

        fused_random = tensor_model_parallel_fused_allreduce_rmsnorm(
            x.clone(), residual.clone(), weight, eps
        )
        torch.cuda.synchronize()
```
**EN:** Distributed entry point: bit-exact residual accuracy across 1-stage/2-stage. This block implements `_run_residual_accuracy_check` and captures one focused piece of the module's behavior. This chunk continues part 1 of the same logical block.
**CN:** Distributed entry point: bit-exact residual accuracy across 1-stage/2-stage. 该代码块实现 `_run_residual_accuracy_check`，承担模块行为中的一个聚焦逻辑片段。 这一段对应同一逻辑块的第 1 部分。

### Lines 98-140: function run residual accuracy check (part 2/2) / 函数 run residual accuracy check（第 2/2 部分）
```python
        _, fused_res = fused_random

        dist.barrier()
        torch.cuda.synchronize()

        unfused_ar = tensor_model_parallel_all_reduce(x.clone())
        torch.cuda.synchronize()

        expected = fused_ar + residual
        diff = (fused_res.float() - expected.float()).abs()
        ar_diff = (fused_ar.float() - unfused_ar.float()).abs()
        max_diff = diff.max().item()
        frac_nonzero = (diff > 0).float().mean().item()

        nbytes = m * n * dtype.itemsize
        stage = "1-stage" if nbytes <= 128 * 1024 else "2-stage"
        passed = max_diff == 0.0

        if not passed:
            all_pass = False

        if rank == 0:
            status = "PASS" if passed else "FAIL"
            print(
                f"  {m:>5d}x{n} ({stage:>7s}): max_diff={max_diff:.6e}  "
                f"frac_nonzero={frac_nonzero:.4f}  "
                f"AR_exact={'yes' if ar_diff.max().item() == 0 else 'no':>3s}  "
                f"[{status}]"
            )

    dist.barrier()
    destroy_model_parallel()
    destroy_distributed_environment()

    if rank == 0:
        print()
        if all_pass:
            print("ALL PASSED: fused residual output is bit-identical to unfused path.")
        else:
            print(
                "FAILED: fused residual output diverges from unfused path for some shapes."
            )
        sys.exit(0 if all_pass else 1)
```
**EN:** Distributed entry point: bit-exact residual accuracy across 1-stage/2-stage. This block implements `_run_residual_accuracy_check` and captures one focused piece of the module's behavior. This chunk continues part 2 of the same logical block.
**CN:** Distributed entry point: bit-exact residual accuracy across 1-stage/2-stage. 该代码块实现 `_run_residual_accuracy_check`，承担模块行为中的一个聚焦逻辑片段。 这一段对应同一逻辑块的第 2 部分。

### Lines 143-144: class TestAiterAllreduceFusionAmd declaration / 类 TestAiterAllreduceFusionAmd 声明
```python
class TestAiterAllreduceFusionAmd(unittest.TestCase):

```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `unittest.TestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `unittest.TestCase`。

### Lines 145-147: method gpu count / 方法 gpu count
```python
    @staticmethod
    def _gpu_count():
        return torch.cuda.device_count() if torch.cuda.is_available() else 0
```
**EN:** This block implements `_gpu_count` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_gpu_count`，承担模块行为中的一个聚焦逻辑片段。

### Lines 149-214: method run benchmark / 方法 run benchmark
```python
    def _run_benchmark(self, nproc, prefill_shapes, decode_shapes):
        """Run the benchmark subprocess and return parsed CSV rows."""
        repo_root = Path(__file__).resolve().parents[3]
        benchmark_script = (
            repo_root
            / "benchmark"
            / "kernels"
            / "all_reduce"
            / "benchmark_fused_ar_rms_amd.py"
        )
        self.assertTrue(
            benchmark_script.exists(),
            f"Missing benchmark script: {benchmark_script}",
        )

        with tempfile.TemporaryDirectory(prefix="aiter_fused_ar_rms_") as tmpdir:
            csv_path = Path(tmpdir) / "fused_ar_rms_check.csv"
            cmd = [
                sys.executable,
                "-m",
                "torch.distributed.run",
                "--standalone",
                f"--nproc_per_node={nproc}",
                str(benchmark_script),
                "--dtype",
                "bf16",
                "--prefill-shapes",
                prefill_shapes,
                "--decode-shapes",
                decode_shapes,
                "--warmup",
                "3",
                "--iters",
                "15",
                "--repeats",
                "2",
                "--csv-out",
                str(csv_path),
            ]

            env = os.environ.copy()
            result = subprocess.run(
                cmd,
                cwd=str(repo_root),
                env=env,
                stdout=subprocess.PIPE,
                stderr=subprocess.STDOUT,
                text=True,
                timeout=1200,
            )

            if result.returncode != 0:
                self.fail(
                    "Benchmark command failed.\n"
                    f"Return code: {result.returncode}\n"
                    f"Command: {' '.join(cmd)}\n"
                    f"Output:\n{result.stdout}"
                )

            self.assertTrue(csv_path.exists(), f"CSV output not found: {csv_path}")

            with open(csv_path, "r", encoding="utf-8") as f:
                rows = list(csv.DictReader(f))

            self.assertGreater(len(rows), 0, "CSV contains no rows.")
            return rows
```
**EN:** Run the benchmark subprocess and return parsed CSV rows. This block implements `_run_benchmark` and captures one focused piece of the module's behavior.
**CN:** Run the benchmark subprocess and return parsed CSV rows. 该代码块实现 `_run_benchmark`，承担模块行为中的一个聚焦逻辑片段。

### Lines 216-222: method assert correctness / 方法 assert correctness
```python
    def _assert_correctness(self, rows):
        bad_rows = [r for r in rows if r["correctness_ok"] != "True"]
        self.assertEqual(
            [],
            bad_rows,
            f"Found correctness failures: {bad_rows}",
        )
```
**EN:** This block implements `_assert_correctness` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_assert_correctness`，承担模块行为中的一个聚焦逻辑片段。

### Lines 224-256: test case fused ar rms benchmark / 测试用例 fused ar rms benchmark
```python
    def test_fused_ar_rms_benchmark(self):
        if self._gpu_count() < 8:
            self.skipTest("This test requires at least 8 GPUs.")

        rows = self._run_benchmark(
            nproc=8,
            prefill_shapes="128x7168,512x7168,2048x7168,4096x7168,5120x7168",
            decode_shapes="1x7168,8x7168,64x7168,512x7168",
        )

        eager_rows = [r for r in rows if r["mode"] == "eager"]
        graph_rows = [r for r in rows if r["mode"] == "graph"]
        self.assertGreater(len(eager_rows), 0, "Missing eager rows in CSV.")
        self.assertGreater(len(graph_rows), 0, "Missing graph rows in CSV.")

        self._assert_correctness(rows)

        self.assertTrue(
            any(r["fused_available"] == "True" for r in eager_rows),
            "Expected at least one eager row with fused_available=True.",
        )
        self.assertTrue(
            any(r["fused_available"] == "True" for r in graph_rows),
            "Expected at least one graph row with fused_available=True.",
        )

        large_eager_rows = [
            r for r in eager_rows if int(r["bytes_per_rank"]) > 64 * 1024 * 1024
        ]
        self.assertTrue(
            any(r["fused_available"] == "False" for r in large_eager_rows),
            "Expected fused fallback for oversized eager shape(s) under default gate.",
        )
```
**EN:** This test exercises `test_fused_ar_rms_benchmark` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_fused_ar_rms_benchmark`。

### Lines 258-284: test case fused ar rms multi hidden dim / 测试用例 fused ar rms multi hidden dim
```python
    def test_fused_ar_rms_multi_hidden_dim(self):
        """Correctness across hidden_dims from various models (TP=4)."""
        nproc = min(self._gpu_count(), 4)
        if nproc < 2:
            self.skipTest("This test requires at least 2 GPUs.")

        # hidden_dims: 2880 (GPT-OSS), 4096 (Qwen3.5), 5120, 6144 (Mixtral),
        # 7168 (DeepSeek), 8192 (Llama-70B)
        decode = ",".join(f"{m}x{n}" for n in HIDDEN_DIMS for m in [1, 4, 16])
        prefill = ",".join(f"128x{n}" for n in HIDDEN_DIMS)

        rows = self._run_benchmark(
            nproc=nproc,
            prefill_shapes=prefill,
            decode_shapes=decode,
        )

        self._assert_correctness(rows)

        fused_rows = [r for r in rows if r["fused_available"] == "True"]
        self.assertEqual(
            len(fused_rows),
            len(rows),
            f"Expected fused available for all shapes, but {len(rows) - len(fused_rows)} "
            f"rows were not fused: "
            f"{[r['shape'] for r in rows if r['fused_available'] != 'True']}",
        )
```
**EN:** Correctness across hidden_dims from various models (TP=4). This test exercises `test_fused_ar_rms_multi_hidden_dim` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Correctness across hidden_dims from various models (TP=4). 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_fused_ar_rms_multi_hidden_dim`。

### Lines 286-328: test case fused ar rms residual accuracy / 测试用例 fused ar rms residual accuracy
```python
    def test_fused_ar_rms_residual_accuracy(self):
        """Bit-exact residual accuracy across 1-stage and 2-stage paths.

        Regression test for ROCm/aiter#2586.  Launches this file itself via
        torchrun with --residual-accuracy to run the distributed check.
        """
        nproc = min(self._gpu_count(), 4)
        if nproc < 2:
            self.skipTest("This test requires at least 2 GPUs.")

        cmd = [
            sys.executable,
            "-m",
            "torch.distributed.run",
            "--standalone",
            f"--nproc_per_node={nproc}",
            __file__,
            "--residual-accuracy",
        ]

        result = subprocess.run(
            cmd,
            cwd=str(Path(__file__).resolve().parents[3]),
            env=os.environ.copy(),
            stdout=subprocess.PIPE,
            stderr=subprocess.STDOUT,
            text=True,
            timeout=300,
        )

        if result.returncode != 0:
            self.fail(
                "Residual accuracy check failed.\n"
                f"Return code: {result.returncode}\n"
                f"Command: {' '.join(cmd)}\n"
                f"Output:\n{result.stdout}"
            )

        self.assertIn(
            "ALL PASSED",
            result.stdout,
            f"Expected 'ALL PASSED' in output, got:\n{result.stdout}",
        )
```
**EN:** Bit-exact residual accuracy across 1-stage and 2-stage paths. This test exercises `test_fused_ar_rms_residual_accuracy` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Bit-exact residual accuracy across 1-stage and 2-stage paths. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_fused_ar_rms_residual_accuracy`。

### Lines 331-335: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    if "--residual-accuracy" in sys.argv:
        _run_residual_accuracy_check()
    else:
        unittest.main()
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `_run_residual_accuracy_check`: Distributed entry point: bit-exact residual accuracy across 1-stage/2-stage. / 该代码块实现 `_run_residual_accuracy_check`，承担模块行为中的一个聚焦逻辑片段。
- `TestAiterAllreduceFusionAmd`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestAiterAllreduceFusionAmd._gpu_count`: This block implements `_gpu_count` and captures one focused piece of the module's behavior. / 该代码块实现 `_gpu_count`，承担模块行为中的一个聚焦逻辑片段。
- `TestAiterAllreduceFusionAmd._run_benchmark`: Run the benchmark subprocess and return parsed CSV rows. / 该代码块实现 `_run_benchmark`，承担模块行为中的一个聚焦逻辑片段。
- `TestAiterAllreduceFusionAmd._assert_correctness`: This block implements `_assert_correctness` and captures one focused piece of the module's behavior. / 该代码块实现 `_assert_correctness`，承担模块行为中的一个聚焦逻辑片段。
- `TestAiterAllreduceFusionAmd.test_fused_ar_rms_benchmark`: This test exercises `test_fused_ar_rms_benchmark` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_fused_ar_rms_benchmark`。
- `TestAiterAllreduceFusionAmd.test_fused_ar_rms_multi_hidden_dim`: Correctness across hidden_dims from various models (TP=4). / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_fused_ar_rms_multi_hidden_dim`。
- `TestAiterAllreduceFusionAmd.test_fused_ar_rms_residual_accuracy`: Bit-exact residual accuracy across 1-stage and 2-stage paths. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_fused_ar_rms_residual_accuracy`。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `csv`, `os`, `subprocess`, `sys`, `tempfile`, `unittest`, `pathlib`
- **Third-party modules / 第三方模块**: `torch`
- **Internal modules / 内部模块**: `sglang.test.ci.ci_register`

- **Total lines / 总行数**: 335
