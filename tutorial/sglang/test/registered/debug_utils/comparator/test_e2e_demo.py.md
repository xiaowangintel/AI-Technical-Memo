# test_e2e_demo.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/debug_utils/comparator/test_e2e_demo.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module focuses on comparator e2e demo in SGLang. It captures the setup, helper logic, and expectations required by this scenario. / 该测试模块用于分析 SGLang 中与 comparator e2e demo 相关的实现或行为。 它概括了该场景所需的初始化、辅助逻辑与预期结果。

## Line-by-Line Analysis / 逐行分析
### Lines 1-7: Document the module
```python
"""Minimal demo: run the comparator on synthetic data and print its output.

This is NOT a correctness test suite.
The sole purpose is to let a new user run ``pytest -s test_e2e_demo.py``
and immediately see what comparator text output looks like (passed, failed,
skipped in one shot).  Correctness is verified via the JSONL report file.
"""
```
**EN:** This block stores descriptive text that explains the intent of the surrounding module.
**CN:** 该代码块保存说明性文本，用于解释周围module的设计意图。

### Lines 9-27: Import dependencies
```python
from __future__ import annotations

import sys
from pathlib import Path
from typing import Dict, List, Optional

import pytest
import torch

import sglang.srt.debug_utils.dumper as _dumper_module
from sglang.srt.debug_utils.comparator.entrypoint import parse_args, run
from sglang.srt.debug_utils.comparator.output_types import (
    AnyRecord,
    ComparisonErrorRecord,
    SummaryRecord,
    parse_record_json,
)
from sglang.srt.debug_utils.dumper import DumperConfig, _Dumper
from sglang.test.ci.ci_register import register_cpu_ci
```
**EN:** This block imports the libraries, fixtures, and helpers that the rest of the file relies on.
**CN:** 该代码块导入了此文件后续逻辑依赖的库、夹具与辅助工具。

### Lines 29-29: Register CI metadata
```python
register_cpu_ci(est_time=10, suite="default", nightly=True)
```
**EN:** This block registers the test case in the CI matrix together with execution-time or runner metadata.
**CN:** 该代码块把测试用例注册到 CI 矩阵中，并附带执行耗时或运行器元数据。

### Lines 31-31: Define module constants
```python
_EXP_NAME = "demo_exp"
```
**EN:** This block stores configuration values or reusable constants for the rest of the module.
**CN:** 该代码块保存模块其余部分会复用的配置值或常量。

### Lines 35-90: Run test: demo (part 1)
```python
def test_demo(tmp_path: Path) -> None:
    """Passed + failed + skipped + sharded + errored in a single demo file."""
    torch.manual_seed(0)
    good_tensor = torch.randn(4, 8)
    sharded_full = torch.randn(2, 8, 16)

    baseline_dir = tmp_path / "baseline"
    target_dir = tmp_path / "target"
    baseline_dir.mkdir()
    target_dir.mkdir()

    # Step 1: simple tensors (single rank, no parallelism)
    _dump_single(baseline_dir, name="my_good_tensor", tensor=good_tensor)
    _dump_single(baseline_dir, name="my_bad_tensor", tensor=torch.randn(4, 8))

    _dump_single(
        target_dir, name="my_good_tensor", tensor=good_tensor + torch.randn(4, 8) * 1e-5
    )
    _dump_single(target_dir, name="my_bad_tensor", tensor=torch.randn(4, 8) * 100)
    _dump_single(target_dir, name="my_orphan_tensor", tensor=torch.randn(4, 8))

    # Step 2: sharded tensor (BSHD) — baseline: TP=2 on h, target: CP=2 zigzag + SP=2 on s
    sharded_target = sharded_full + torch.randn_like(sharded_full) * 1e-5
    _dump_tp_sharded(
        baseline_dir, name="my_sharded_tensor", full_tensor=sharded_full, tp_size=2
    )
    _dump_cp_zigzag_sp_sharded(
        target_dir,
        name="my_sharded_tensor",
        full_tensor=sharded_target,
        cp_size=2,
        sp_size=2,
    )

    # Step 3: bad dims — target says h[cp] but parallel_info has tp → undeclared axis error
    bad_dims_tensor = torch.randn(2, 8, 16)
    for tp_rank, shard in enumerate(bad_dims_tensor.chunk(2, dim=-1)):
        _dump_rank(
            baseline_dir,
            rank=tp_rank,
            name="my_bad_dims_tensor",
            tensor=shard,
            dims="b s h[tp]",
            parallel_info={"tp_rank": tp_rank, "tp_size": 2},
        )
        _dump_rank(
            target_dir,
            rank=tp_rank,
            name="my_bad_dims_tensor",
            tensor=shard,
            dims="b s h[cp]",
            parallel_info={"tp_rank": tp_rank, "tp_size": 2},
        )

    baseline_exp = baseline_dir / _EXP_NAME
    target_exp = target_dir / _EXP_NAME
```
**EN:** This test method exercises demo and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 demo 场景，并验证观测到的行为是否符合预期契约。

### Lines 92-102: Run test: demo (part 2)
```python
    # Step 4: run normal, then verbose
    for verbosity in ("normal", "verbose"):
        report_path = tmp_path / f"report_{verbosity}.jsonl"
        _run(
            baseline_exp,
            target_exp,
            report_path=report_path,
            output_format="text",
            verbosity=verbosity,
        )
        _assert_summary(report_path, passed=2, failed=1, skipped=1, errored=1)
```
**EN:** This test method exercises demo and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 demo 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 104-109: Run test: demo (part 3)
```python
    # Step 5: verify error record content
    records = _read_report(tmp_path / "report_verbose.jsonl")
    errors = [r for r in records if isinstance(r, ComparisonErrorRecord)]
    assert len(errors) == 1
    assert "tp" in errors[0].exception_message
    assert "--override-dims" in errors[0].traceback_str
```
**EN:** This test method exercises demo and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 demo 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 115-123: Define helper: assert summary
```python
def _assert_summary(
    report_path: Path, *, passed: int, failed: int, skipped: int, errored: int = 0
) -> None:
    records = _read_report(report_path)
    summary = next(r for r in records if isinstance(r, SummaryRecord))
    assert summary.passed == passed
    assert summary.failed == failed
    assert summary.skipped == skipped
    assert summary.errored == errored
```
**EN:** This helper function encapsulates reusable logic so the scenario stays organized. It also checks concrete expectations with assertions.
**CN:** 该辅助函数封装了可复用逻辑，从而让场景结构更清晰。 其中还会通过断言检查明确的预期。

### Lines 126-127: Define helper: dump single
```python
def _dump_single(directory: Path, *, name: str, tensor: torch.Tensor) -> None:
    _dump_rank(directory, rank=0, name=name, tensor=tensor)
```
**EN:** This helper function encapsulates reusable logic so the scenario stays organized.
**CN:** 该辅助函数封装了可复用逻辑，从而让场景结构更清晰。

### Lines 130-147: Define helper: dump tp sharded
```python
def _dump_tp_sharded(
    directory: Path,
    *,
    name: str,
    full_tensor: torch.Tensor,
    tp_size: int,
) -> None:
    """Dump TP-sharded tensor: dims="b s h[tp]", shard along last dim."""
    shards = list(full_tensor.chunk(tp_size, dim=-1))
    for tp_rank, shard in enumerate(shards):
        _dump_rank(
            directory,
            rank=tp_rank,
            name=name,
            tensor=shard,
            dims="b s h[tp]",
            parallel_info={"tp_rank": tp_rank, "tp_size": tp_size},
        )
```
**EN:** This helper function encapsulates reusable logic so the scenario stays organized.
**CN:** 该辅助函数封装了可复用逻辑，从而让场景结构更清晰。

### Lines 150-188: Define helper: dump cp zigzag sp sharded
```python
def _dump_cp_zigzag_sp_sharded(
    directory: Path,
    *,
    name: str,
    full_tensor: torch.Tensor,
    cp_size: int,
    sp_size: int,
) -> None:
    """Dump CP-zigzag+SP sharded tensor: dims="b s[cp:zigzag,sp] h", shard seq dim."""
    seq_dim = 1
    num_chunks = cp_size * 2
    natural_chunks = list(full_tensor.chunk(num_chunks, dim=seq_dim))

    zigzag_order: List[int] = []
    for i in range(cp_size):
        zigzag_order.append(i)
        zigzag_order.append(num_chunks - 1 - i)

    zigzagged = torch.cat([natural_chunks[idx] for idx in zigzag_order], dim=seq_dim)
    cp_chunks = list(zigzagged.chunk(cp_size, dim=seq_dim))

    rank = 0
    for cp_rank in range(cp_size):
        sp_chunks = list(cp_chunks[cp_rank].chunk(sp_size, dim=seq_dim))
        for sp_rank in range(sp_size):
            _dump_rank(
                directory,
                rank=rank,
                name=name,
                tensor=sp_chunks[sp_rank],
                dims="b s[cp:zigzag,sp] h",
                parallel_info={
                    "cp_rank": cp_rank,
                    "cp_size": cp_size,
                    "sp_rank": sp_rank,
                    "sp_size": sp_size,
                },
            )
            rank += 1
```
**EN:** This helper function encapsulates reusable logic so the scenario stays organized.
**CN:** 该辅助函数封装了可复用逻辑，从而让场景结构更清晰。

### Lines 191-210: Define helper: dump rank
```python
def _dump_rank(
    directory: Path,
    *,
    rank: int,
    name: str,
    tensor: torch.Tensor,
    dims: Optional[str] = None,
    parallel_info: Optional[Dict[str, int]] = None,
) -> None:
    with pytest.MonkeyPatch.context() as mp:
        mp.setattr(_dumper_module, "_get_rank", lambda: rank)
        dumper = _Dumper(
            config=DumperConfig(enable=True, dir=str(directory), exp_name=_EXP_NAME)
        )
        static_meta: Dict[str, object] = {"world_rank": rank, "world_size": 1}
        if parallel_info is not None:
            static_meta["sglang_parallel_info"] = parallel_info
        dumper.__dict__["_static_meta"] = static_meta
        dumper.dump(name, tensor, dims=dims)
        dumper.step()
```
**EN:** This helper function encapsulates reusable logic so the scenario stays organized.
**CN:** 该辅助函数封装了可复用逻辑，从而让场景结构更清晰。

### Lines 213-239: Define helper: run
```python
def _run(
    baseline_path: Path,
    target_path: Path,
    *,
    report_path: Path,
    output_format: str = "text",
    verbosity: str = "normal",
) -> int:
    argv = [
        "--baseline-path",
        str(baseline_path),
        "--target-path",
        str(target_path),
        "--output-format",
        output_format,
        "--verbosity",
        verbosity,
        "--preset",
        "sglang_dev",
        "--report-path",
        str(report_path),
    ]
    print(
        f"\n  $ python -m sglang.srt.debug_utils.comparator {' '.join(argv)}\n",
        flush=True,
    )
    return run(parse_args(argv))
```
**EN:** This helper function encapsulates reusable logic so the scenario stays organized.
**CN:** 该辅助函数封装了可复用逻辑，从而让场景结构更清晰。

### Lines 242-245: Define helper: read report
```python
def _read_report(report_path: Path) -> List[AnyRecord]:
    return [
        parse_record_json(line) for line in report_path.read_text().strip().splitlines()
    ]
```
**EN:** This helper function encapsulates reusable logic so the scenario stays organized.
**CN:** 该辅助函数封装了可复用逻辑，从而让场景结构更清晰。

### Lines 248-249: Expose unittest entrypoint
```python
if __name__ == "__main__":
    sys.exit(pytest.main([__file__, "-s", "-v"]))
```
**EN:** This conditional keeps the module executable as a standalone unittest target.
**CN:** 该条件语句使模块可以作为独立的 unittest 目标直接运行。

## Key Concepts / 关键概念
- The file is structured as an automated regression test for a focused scenario. / 该文件按聚焦场景组织为自动化回归测试。
- CI registration keeps the case visible to automated test pipelines. / CI 注册使该用例能够被自动化测试流水线发现。
- Assertions encode the behavioral contract that the scenario must satisfy. / 断言编码了该场景必须满足的行为契约。

## Dependencies / 依赖关系
- Internal modules / 内部模块: `sglang.srt.debug_utils.comparator.entrypoint`, `sglang.srt.debug_utils.comparator.output_types`, `sglang.srt.debug_utils.dumper`, `sglang.test.ci.ci_register`
- External and stdlib modules / 外部与标准库模块: `__future__`, `pathlib`, `pytest`, `sys`, `torch`, `typing`
- Notable symbols / 关键符号: None / 无
