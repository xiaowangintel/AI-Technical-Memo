# test_bundle_matcher.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/debug_utils/comparator/test_bundle_matcher.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module focuses on comparator bundle matcher in SGLang. It captures the setup, helper logic, and expectations required by this scenario. / 该测试模块用于分析 SGLang 中与 comparator bundle matcher 相关的实现或行为。 它概括了该场景所需的初始化、辅助逻辑与预期结果。

## Line-by-Line Analysis / 逐行分析
### Lines 1-14: Import dependencies
```python
import sys
from typing import Any

import polars as pl
import pytest

from sglang.srt.debug_utils.comparator.bundle_matcher import (
    TensorBundleInfo,
    TensorFileInfo,
    _rows_to_tensor_infos,
    match_bundles,
)
from sglang.srt.debug_utils.comparator.utils import Pair
from sglang.test.ci.ci_register import register_cpu_ci
```
**EN:** This block imports the libraries, fixtures, and helpers that the rest of the file relies on.
**CN:** 该代码块导入了此文件后续逻辑依赖的库、夹具与辅助工具。

### Lines 16-16: Register CI metadata
```python
register_cpu_ci(est_time=15, suite="base-a-test-cpu", nightly=True)
```
**EN:** This block registers the test case in the CI matrix together with execution-time or runner metadata.
**CN:** 该代码块把测试用例注册到 CI 矩阵中，并附带执行耗时或运行器元数据。

### Lines 19-38: Define helper: make row
```python
def _make_row(
    *,
    name: str,
    step: int = 0,
    rank: int = 0,
    layer_id: int | None = None,
    filename: str | None = None,
) -> dict[str, Any]:
    if filename is None:
        layer_part: str = f"___layer_id={layer_id}" if layer_id is not None else ""
        filename = f"name={name}___step={step}___rank={rank}{layer_part}.pt"
    row: dict[str, Any] = {
        "name": name,
        "step": step,
        "rank": rank,
        "filename": filename,
    }
    if layer_id is not None:
        row["layer_id"] = layer_id
    return row
```
**EN:** This helper function encapsulates reusable logic so the scenario stays organized.
**CN:** 该辅助函数封装了可复用逻辑，从而让场景结构更清晰。

### Lines 41-51: Define helper: make df
```python
def _make_df(rows: list[dict[str, Any]]) -> pl.DataFrame:
    if not rows:
        return pl.DataFrame(rows)

    all_keys: set[str] = set()
    for row in rows:
        all_keys.update(row.keys())
    normalized: list[dict[str, Any]] = [
        {k: row.get(k, None) for k in all_keys} for row in rows
    ]
    return pl.DataFrame(normalized)
```
**EN:** This helper function encapsulates reusable logic so the scenario stays organized.
**CN:** 该辅助函数封装了可复用逻辑，从而让场景结构更清晰。

### Lines 54-54: Define class TestMatchBundles
```python
class TestMatchBundles:
```
**EN:** This declaration introduces the `TestMatchBundles` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestMatchBundles` 测试类，并说明它通过继承承担的职责。

### Lines 55-67: Run test: single tensor single step
```python
    def test_single_tensor_single_step(self) -> None:
        target_df: pl.DataFrame = _make_df([_make_row(name="t_a")])
        baseline_df: pl.DataFrame = _make_df([_make_row(name="t_a")])

        results: list[Pair[TensorBundleInfo]] = match_bundles(
            dfs=Pair(x=baseline_df, y=target_df),
            skip_keys={"filename"},
        )

        assert len(results) == 1
        assert len(results[0].x) == 1
        assert len(results[0].y) == 1
        assert results[0].y[0].name == "t_a"
```
**EN:** This test method exercises single tensor single step and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 single tensor single step 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 69-91: Run test: multiple names separate bundles
```python
    def test_multiple_names_separate_bundles(self) -> None:
        target_df: pl.DataFrame = _make_df(
            [
                _make_row(name="t_a"),
                _make_row(name="t_b"),
            ]
        )
        baseline_df: pl.DataFrame = _make_df(
            [
                _make_row(name="t_a"),
                _make_row(name="t_b"),
            ]
        )

        results: list[Pair[TensorBundleInfo]] = match_bundles(
            dfs=Pair(x=baseline_df, y=target_df),
            skip_keys={"filename"},
        )

        assert len(results) == 2
        result_names: list[str] = [r.y[0].name for r in results]
        assert "t_a" in result_names
        assert "t_b" in result_names
```
**EN:** This test method exercises multiple names separate bundles and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 multiple names separate bundles 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 93-113: Run test: skip rank groups across ranks
```python
    def test_skip_rank_groups_across_ranks(self) -> None:
        target_df: pl.DataFrame = _make_df(
            [
                _make_row(name="t_a", rank=0),
                _make_row(name="t_a", rank=1),
            ]
        )
        baseline_df: pl.DataFrame = _make_df(
            [
                _make_row(name="t_a", rank=0),
                _make_row(name="t_a", rank=1),
            ]
        )

        results: list[Pair[TensorBundleInfo]] = match_bundles(
            dfs=Pair(x=baseline_df, y=target_df),
            skip_keys={"filename", "rank"},
        )

        assert len(results) == 1
        assert len(results[0].y) == 2
```
**EN:** This test method exercises skip rank groups across ranks and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 skip rank groups across ranks 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 115-137: Run test: baseline missing tensor
```python
    def test_baseline_missing_tensor(self) -> None:
        target_df: pl.DataFrame = _make_df(
            [
                _make_row(name="t_a"),
                _make_row(name="t_extra"),
            ]
        )
        baseline_df: pl.DataFrame = _make_df(
            [
                _make_row(name="t_a"),
            ]
        )

        results: list[Pair[TensorBundleInfo]] = match_bundles(
            dfs=Pair(x=baseline_df, y=target_df),
            skip_keys={"filename"},
        )

        assert len(results) == 2
        extra_pair: Pair[TensorBundleInfo] = [
            r for r in results if r.y[0].name == "t_extra"
        ][0]
        assert extra_pair.x == []
```
**EN:** This test method exercises baseline missing tensor and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 baseline missing tensor 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 139-148: Run test: empty target returns empty
```python
    def test_empty_target_returns_empty(self) -> None:
        target_df: pl.DataFrame = _make_df([])
        baseline_df: pl.DataFrame = _make_df([_make_row(name="t_a")])

        results: list[Pair[TensorBundleInfo]] = match_bundles(
            dfs=Pair(x=baseline_df, y=target_df),
            skip_keys={"filename"},
        )

        assert results == []
```
**EN:** This test method exercises empty target returns empty and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 empty target returns empty 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 150-170: Run test: skip step groups across steps
```python
    def test_skip_step_groups_across_steps(self) -> None:
        target_df: pl.DataFrame = _make_df(
            [
                _make_row(name="t_a", step=0),
                _make_row(name="t_a", step=1),
            ]
        )
        baseline_df: pl.DataFrame = _make_df(
            [
                _make_row(name="t_a", step=0),
                _make_row(name="t_a", step=1),
            ]
        )

        results: list[Pair[TensorBundleInfo]] = match_bundles(
            dfs=Pair(x=baseline_df, y=target_df),
            skip_keys={"filename", "step"},
        )

        assert len(results) == 1
        assert len(results[0].y) == 2
```
**EN:** This test method exercises skip step groups across steps and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 skip step groups across steps 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 173-173: Define class TestMatchBundlesPipelineParallel
```python
class TestMatchBundlesPipelineParallel:
```
**EN:** This declaration introduces the `TestMatchBundlesPipelineParallel` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestMatchBundlesPipelineParallel` 测试类，并说明它通过继承承担的职责。

### Lines 174-174: Document the class `TestMatchBundlesPipelineParallel`
```python
    """Tests verifying that PP works correctly with the existing matching logic."""
```
**EN:** This block stores descriptive text that explains the intent of the surrounding class `TestMatchBundlesPipelineParallel`.
**CN:** 该代码块保存说明性文本，用于解释周围class `TestMatchBundlesPipelineParallel`的设计意图。

### Lines 176-176: Declare TestMatchBundlesPipelineParallel configuration
```python
    LOGICAL_SKIP_KEYS: set[str] = {"filename", "rank", "dump_index", "recompute_status"}
```
**EN:** This block defines class-level settings that are shared across the `TestMatchBundlesPipelineParallel` test methods.
**CN:** 该代码块定义了 `TestMatchBundlesPipelineParallel` 各测试方法共享的类级配置。

### Lines 178-195: Run test: same layer id different ranks match
```python
    def test_same_layer_id_different_ranks_match(self) -> None:
        """SGLang PP=2 rank 0 (layers 0-31) vs Megatron PP=4 rank 2 (layers 16-31):
        layer_id=20 should match regardless of world rank."""
        target_df: pl.DataFrame = _make_df(
            [_make_row(name="hidden", rank=0, layer_id=20)]
        )
        baseline_df: pl.DataFrame = _make_df(
            [_make_row(name="hidden", rank=2, layer_id=20)]
        )

        results: list[Pair[TensorBundleInfo]] = match_bundles(
            dfs=Pair(x=baseline_df, y=target_df),
            skip_keys=self.LOGICAL_SKIP_KEYS,
        )

        assert len(results) == 1
        assert len(results[0].x) == 1
        assert len(results[0].y) == 1
```
**EN:** This test method exercises same layer id different ranks match and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 same layer id different ranks match 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 197-210: Run test: layer id none non layer tensors match
```python
    def test_layer_id_none_non_layer_tensors_match(self) -> None:
        """Non-layer tensors (embedding, lm_head) have no layer_id.
        They should match across different PP ranks."""
        target_df: pl.DataFrame = _make_df([_make_row(name="embed_tokens", rank=0)])
        baseline_df: pl.DataFrame = _make_df([_make_row(name="embed_tokens", rank=0)])

        results: list[Pair[TensorBundleInfo]] = match_bundles(
            dfs=Pair(x=baseline_df, y=target_df),
            skip_keys=self.LOGICAL_SKIP_KEYS,
        )

        assert len(results) == 1
        assert len(results[0].x) == 1
        assert len(results[0].y) == 1
```
**EN:** This test method exercises layer id none non layer tensors match and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 layer id none non layer tensors match 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 212-268: Run test: different pp sizes layer and non layer bundles
```python
    def test_different_pp_sizes_layer_and_non_layer_bundles(self) -> None:
        """SGLang PP=2 TP=2 (4 ranks) vs Megatron PP=4 TP=2 (8 ranks).
        Layer tensors match by (name, layer_id); non-layer tensors match by name.
        All ranks are grouped into the same bundle when rank is skipped."""
        target_df: pl.DataFrame = _make_df(
            [
                # SGLang: pp_stage=0 has ranks 0,1 (TP=2)
                _make_row(name="hidden", rank=0, layer_id=20),
                _make_row(name="hidden", rank=1, layer_id=20),
                # SGLang: embedding on pp_stage=0
                _make_row(name="embed_tokens", rank=0),
                _make_row(name="embed_tokens", rank=1),
                # SGLang: lm_head on pp_stage=1, ranks 2,3
                _make_row(name="lm_head", rank=2),
                _make_row(name="lm_head", rank=3),
            ]
        )
        baseline_df: pl.DataFrame = _make_df(
            [
                # Megatron: pp_stage=1 has ranks 2,3 for layer 20
                _make_row(name="hidden", rank=2, layer_id=20),
                _make_row(name="hidden", rank=3, layer_id=20),
                # Megatron: embedding on pp_stage=0
                _make_row(name="embed_tokens", rank=0),
                _make_row(name="embed_tokens", rank=1),
                # Megatron: lm_head on pp_stage=3, ranks 6,7
                _make_row(name="lm_head", rank=6),
                _make_row(name="lm_head", rank=7),
            ]
        )

        results: list[Pair[TensorBundleInfo]] = match_bundles(
            dfs=Pair(x=baseline_df, y=target_df),
            skip_keys=self.LOGICAL_SKIP_KEYS,
        )

        assert len(results) == 3
        names_to_pairs: dict[str, Pair[TensorBundleInfo]] = {}
        for pair in results:
            key: str = pair.y[0].name
            layer_suffix: str = ""
            if "layer_id" in target_df.columns:
                row_match = [
                    r
                    for r in target_df.to_dicts()
                    if r["filename"] == pair.y[0].filename
                ]
                if row_match and row_match[0].get("layer_id") is not None:
                    layer_suffix = f"_{row_match[0]['layer_id']}"
            names_to_pairs[key + layer_suffix] = pair

        assert len(names_to_pairs["hidden_20"].x) == 2
        assert len(names_to_pairs["hidden_20"].y) == 2
        assert len(names_to_pairs["embed_tokens"].x) == 2
        assert len(names_to_pairs["embed_tokens"].y) == 2
        assert len(names_to_pairs["lm_head"].x) == 2
        assert len(names_to_pairs["lm_head"].y) == 2
```
**EN:** This test method exercises different pp sizes layer and non layer bundles and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 different pp sizes layer and non layer bundles 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 270-294: Run test: unmatched layer id creates empty baseline
```python
    def test_unmatched_layer_id_creates_empty_baseline(self) -> None:
        """If target has a layer_id that baseline doesn't, the baseline side
        should be empty (not incorrectly matched to a different layer)."""
        target_df: pl.DataFrame = _make_df(
            [
                _make_row(name="hidden", rank=0, layer_id=10),
                _make_row(name="hidden", rank=0, layer_id=20),
            ]
        )
        baseline_df: pl.DataFrame = _make_df(
            [
                _make_row(name="hidden", rank=0, layer_id=10),
            ]
        )

        results: list[Pair[TensorBundleInfo]] = match_bundles(
            dfs=Pair(x=baseline_df, y=target_df),
            skip_keys=self.LOGICAL_SKIP_KEYS,
        )

        assert len(results) == 2
        matched: list[Pair[TensorBundleInfo]] = [r for r in results if r.x]
        unmatched: list[Pair[TensorBundleInfo]] = [r for r in results if not r.x]
        assert len(matched) == 1
        assert len(unmatched) == 1
```
**EN:** This test method exercises unmatched layer id creates empty baseline and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 unmatched layer id creates empty baseline 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 296-322: Run test: pp1 vs pp gt1 matches by layer id
```python
    def test_pp1_vs_pp_gt1_matches_by_layer_id(self) -> None:
        """PP=1 (all layers on 1 rank) vs PP>1 (layers split across ranks).
        Should match correctly by layer_id regardless of rank."""
        target_df: pl.DataFrame = _make_df(
            [
                # PP=1: all on rank 0
                _make_row(name="hidden", rank=0, layer_id=0),
                _make_row(name="hidden", rank=0, layer_id=1),
            ]
        )
        baseline_df: pl.DataFrame = _make_df(
            [
                # PP=2: layer 0 on rank 0, layer 1 on rank 1
                _make_row(name="hidden", rank=0, layer_id=0),
                _make_row(name="hidden", rank=1, layer_id=1),
            ]
        )

        results: list[Pair[TensorBundleInfo]] = match_bundles(
            dfs=Pair(x=baseline_df, y=target_df),
            skip_keys=self.LOGICAL_SKIP_KEYS,
        )

        assert len(results) == 2
        for pair in results:
            assert len(pair.x) == 1
            assert len(pair.y) == 1
```
**EN:** This test method exercises pp1 vs pp gt1 matches by layer id and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 pp1 vs pp gt1 matches by layer id 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 325-325: Define class TestRowsToTensorInfos
```python
class TestRowsToTensorInfos:
```
**EN:** This declaration introduces the `TestRowsToTensorInfos` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestRowsToTensorInfos` 测试类，并说明它通过继承承担的职责。

### Lines 326-333: Run test: filters extra columns
```python
    def test_filters_extra_columns(self) -> None:
        rows: list[dict[str, Any]] = [
            {"filename": "a.pt", "name": "t_a", "step": 0, "rank": 7}
        ]
        infos: list[TensorFileInfo] = _rows_to_tensor_infos(rows)

        assert len(infos) == 1
        assert infos[0] == TensorFileInfo(filename="a.pt", name="t_a", step=0)
```
**EN:** This test method exercises filters extra columns and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 filters extra columns 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 335-337: Run test: empty rows
```python
    def test_empty_rows(self) -> None:
        infos: list[TensorFileInfo] = _rows_to_tensor_infos([])
        assert infos == []
```
**EN:** This test method exercises empty rows and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 empty rows 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 340-341: Expose unittest entrypoint
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
- Internal modules / 内部模块: `sglang.srt.debug_utils.comparator.bundle_matcher`, `sglang.srt.debug_utils.comparator.utils`, `sglang.test.ci.ci_register`
- External and stdlib modules / 外部与标准库模块: `polars`, `pytest`, `sys`, `typing`
- Notable symbols / 关键符号: None / 无
