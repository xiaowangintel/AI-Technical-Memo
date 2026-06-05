# test_thd_seq_lens_loader.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/debug_utils/comparator/aligner/token_aligner/test_thd_seq_lens_loader.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module focuses on token aligner thd seq lens loader in SGLang. It captures the setup, helper logic, and expectations required by this scenario. / 该测试模块用于分析 SGLang 中与 token aligner thd seq lens loader 相关的实现或行为。 它概括了该场景所需的初始化、辅助逻辑与预期结果。

## Line-by-Line Analysis / 逐行分析
### Lines 1-15: Import dependencies
```python
import sys
from pathlib import Path
from unittest.mock import patch

import polars as pl
import pytest
import torch

from sglang.srt.debug_utils.comparator.aligner.token_aligner.concat_steps.thd_seq_lens_loader import (
    load_thd_seq_lens_only,
)
from sglang.srt.debug_utils.comparator.aligner.token_aligner.smart.aux_plugins import (
    _SGLangPlugin,
)
from sglang.test.ci.ci_register import register_cpu_ci
```
**EN:** This block imports the libraries, fixtures, and helpers that the rest of the file relies on.
**CN:** 该代码块导入了此文件后续逻辑依赖的库、夹具与辅助工具。

### Lines 17-17: Register CI metadata
```python
register_cpu_ci(est_time=15, suite="base-a-test-cpu", nightly=True)
```
**EN:** This block registers the test case in the CI matrix together with execution-time or runner metadata.
**CN:** 该代码块把测试用例注册到 CI 矩阵中，并附带执行耗时或运行器元数据。

### Lines 20-32: Define helper: save pt
```python
def _save_pt(
    dump_path: Path,
    *,
    name: str,
    step: int,
    rank: int,
    value: object,
    meta: dict | None = None,
) -> str:
    filename: str = f"name={name}___step={step}___rank={rank}.pt"
    payload: dict = {"value": value, "meta": meta or {}}
    torch.save(payload, dump_path / filename)
    return filename
```
**EN:** This helper function encapsulates reusable logic so the scenario stays organized.
**CN:** 该辅助函数封装了可复用逻辑，从而让场景结构更清晰。

### Lines 35-52: Define helper: make df from filenames
```python
def _make_df_from_filenames(filenames: list[str]) -> pl.DataFrame:
    rows: list[dict] = []
    for fn in filenames:
        parts: dict = {}
        stem: str = fn.removesuffix(".pt")
        for kv in stem.split("___"):
            if "=" in kv:
                k, v = kv.split("=", 1)
                parts[k] = v
        rows.append(
            {
                "filename": fn,
                "name": parts["name"],
                "step": int(parts["step"]),
                "rank": int(parts["rank"]),
            }
        )
    return pl.DataFrame(rows)
```
**EN:** This helper function encapsulates reusable logic so the scenario stays organized.
**CN:** 该辅助函数封装了可复用逻辑，从而让场景结构更清晰。

### Lines 55-55: Define class TestLoadThdSeqLensOnly
```python
class TestLoadThdSeqLensOnly:
```
**EN:** This declaration introduces the `TestLoadThdSeqLensOnly` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestLoadThdSeqLensOnly` 测试类，并说明它通过继承承担的职责。

### Lines 56-56: Document the class `TestLoadThdSeqLensOnly`
```python
    """Tests for load_thd_seq_lens_only."""
```
**EN:** This block stores descriptive text that explains the intent of the surrounding class `TestLoadThdSeqLensOnly`.
**CN:** 该代码块保存说明性文本，用于解释周围class `TestLoadThdSeqLensOnly`的设计意图。

### Lines 58-67: Run test: returns none when no plugin
```python
    def test_returns_none_when_no_plugin(self, tmp_path: Path) -> None:
        """No recognized plugin → returns None."""
        fn: str = _save_pt(
            tmp_path, name="unrelated_tensor", step=0, rank=0, value=torch.tensor([1])
        )
        df: pl.DataFrame = _make_df_from_filenames([fn])

        result = load_thd_seq_lens_only(dump_path=tmp_path, df=df)

        assert result is None
```
**EN:** This test method exercises returns none when no plugin and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 returns none when no plugin 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 69-93: Run test: returns none when no cp sharded names
```python
    def test_returns_none_when_no_cp_sharded_names(self, tmp_path: Path) -> None:
        """Plugin detected but cp_sharded_names is empty → returns None."""

        class _NoCpPlugin(_SGLangPlugin):
            @property
            def cp_sharded_names(self) -> frozenset[str]:
                return frozenset()

        fn: str = _save_pt(
            tmp_path,
            name="seq_lens",
            step=0,
            rank=0,
            value=torch.tensor([3, 5]),
            meta={"sglang_parallel_info": {"tp_rank": 0, "tp_size": 1}},
        )
        df: pl.DataFrame = _make_df_from_filenames([fn])

        with patch(
            "sglang.srt.debug_utils.comparator.aligner.token_aligner.concat_steps.thd_seq_lens_loader._detect_plugin",
            return_value=_NoCpPlugin(),
        ):
            result = load_thd_seq_lens_only(dump_path=tmp_path, df=df)

        assert result is None
```
**EN:** This test method exercises returns none when no cp sharded names and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 returns none when no cp sharded names 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 95-110: Run test: sglang extracts seq lens
```python
    def test_sglang_extracts_seq_lens(self, tmp_path: Path) -> None:
        """SGLang format: seq_lens tensor present → extracts per-seq lengths."""
        fn: str = _save_pt(
            tmp_path,
            name="seq_lens",
            step=0,
            rank=0,
            value=torch.tensor([3, 5]),
            meta={"sglang_parallel_info": {"tp_rank": 0, "tp_size": 1}},
        )
        df: pl.DataFrame = _make_df_from_filenames([fn])

        result = load_thd_seq_lens_only(dump_path=tmp_path, df=df)

        assert result is not None
        assert result == {0: [3, 5]}
```
**EN:** This test method exercises sglang extracts seq lens and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 sglang extracts seq lens 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 112-127: Run test: megatron extracts from cu seqlens
```python
    def test_megatron_extracts_from_cu_seqlens(self, tmp_path: Path) -> None:
        """Megatron format: cu_seqlens_q tensor → derives seq_lens via diff."""
        fn: str = _save_pt(
            tmp_path,
            name="cu_seqlens_q",
            step=0,
            rank=0,
            value=torch.tensor([0, 3, 8], dtype=torch.int64),
            meta={"megatron_parallel_info": {"cp_rank": 0, "cp_size": 2}},
        )
        df: pl.DataFrame = _make_df_from_filenames([fn])

        result = load_thd_seq_lens_only(dump_path=tmp_path, df=df)

        assert result is not None
        assert result == {0: [3, 5]}
```
**EN:** This test method exercises megatron extracts from cu seqlens and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 megatron extracts from cu seqlens 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 129-152: Run test: multi step
```python
    def test_multi_step(self, tmp_path: Path) -> None:
        """Two steps with different seq_lens → returns both in result dict."""
        fn0: str = _save_pt(
            tmp_path,
            name="seq_lens",
            step=0,
            rank=0,
            value=torch.tensor([3, 5]),
            meta={"sglang_parallel_info": {"tp_rank": 0, "tp_size": 1}},
        )
        fn1: str = _save_pt(
            tmp_path,
            name="seq_lens",
            step=1,
            rank=0,
            value=torch.tensor([10, 20, 30]),
            meta={"sglang_parallel_info": {"tp_rank": 0, "tp_size": 1}},
        )
        df: pl.DataFrame = _make_df_from_filenames([fn0, fn1])

        result = load_thd_seq_lens_only(dump_path=tmp_path, df=df)

        assert result is not None
        assert result == {0: [3, 5], 1: [10, 20, 30]}
```
**EN:** This test method exercises multi step and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 multi step 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 154-168: Run test: returns none when seq lens missing
```python
    def test_returns_none_when_seq_lens_missing(self, tmp_path: Path) -> None:
        """Plugin with cp_sharded_names but no seq_lens/cu_seqlens_q tensor → None."""
        fn: str = _save_pt(
            tmp_path,
            name="cu_seqlens_kv",
            step=0,
            rank=0,
            value=torch.tensor([0, 4], dtype=torch.int64),
            meta={"megatron_parallel_info": {"cp_rank": 0, "cp_size": 2}},
        )
        df: pl.DataFrame = _make_df_from_filenames([fn])

        result = load_thd_seq_lens_only(dump_path=tmp_path, df=df)

        assert result is None
```
**EN:** This test method exercises returns none when seq lens missing and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 returns none when seq lens missing 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 171-172: Expose unittest entrypoint
```python
if __name__ == "__main__":
    sys.exit(pytest.main([__file__]))
```
**EN:** This conditional keeps the module executable as a standalone unittest target.
**CN:** 该条件语句使模块可以作为独立的 unittest 目标直接运行。

## Key Concepts / 关键概念
- The file is structured as an automated regression test for a focused scenario. / 该文件按聚焦场景组织为自动化回归测试。
- CI registration keeps the case visible to automated test pipelines. / CI 注册使该用例能够被自动化测试流水线发现。
- Assertions encode the behavioral contract that the scenario must satisfy. / 断言编码了该场景必须满足的行为契约。

## Dependencies / 依赖关系
- Internal modules / 内部模块: `sglang.srt.debug_utils.comparator.aligner.token_aligner.concat_steps.thd_seq_lens_loader`, `sglang.srt.debug_utils.comparator.aligner.token_aligner.smart.aux_plugins`, `sglang.test.ci.ci_register`
- External and stdlib modules / 外部与标准库模块: `pathlib`, `polars`, `pytest`, `sys`, `torch`, `unittest.mock`
- Notable symbols / 关键符号: None / 无
