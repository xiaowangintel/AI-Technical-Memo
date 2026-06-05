# test_aux_loader.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/debug_utils/comparator/aligner/token_aligner/test_aux_loader.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module focuses on token aligner aux loader in SGLang. It captures the setup, helper logic, and expectations required by this scenario. / 该测试模块用于分析 SGLang 中与 token aligner aux loader 相关的实现或行为。 它概括了该场景所需的初始化、辅助逻辑与预期结果。

## Line-by-Line Analysis / 逐行分析
### Lines 1-20: Import dependencies
```python
import sys
from pathlib import Path

import polars as pl
import pytest
import torch

from sglang.srt.debug_utils.comparator.aligner.token_aligner.smart.aux_loader import (
    _detect_plugin,
    _ensure_dims_in_metas,
    _load_and_align_aux_tensor,
    _load_non_tensor_aux,
)
from sglang.srt.debug_utils.comparator.aligner.token_aligner.smart.aux_plugins import (
    _MegatronPlugin,
    _SGLangPlugin,
)
from sglang.srt.debug_utils.comparator.log_sink import LogSink
from sglang.srt.debug_utils.comparator.output_types import ErrorLog, InfoLog
from sglang.test.ci.ci_register import register_cpu_ci
```
**EN:** This block imports the libraries, fixtures, and helpers that the rest of the file relies on.
**CN:** 该代码块导入了此文件后续逻辑依赖的库、夹具与辅助工具。

### Lines 22-22: Register CI metadata
```python
register_cpu_ci(est_time=15, suite="base-a-test-cpu", nightly=True)
```
**EN:** This block registers the test case in the CI matrix together with execution-time or runner metadata.
**CN:** 该代码块把测试用例注册到 CI 矩阵中，并附带执行耗时或运行器元数据。

### Lines 24-25: Define module constants
```python
_sglang_plugin = _SGLangPlugin()
_megatron_plugin = _MegatronPlugin()
```
**EN:** This block stores configuration values or reusable constants for the rest of the module.
**CN:** 该代码块保存模块其余部分会复用的配置值或常量。

### Lines 28-40: Define helper: save pt
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

### Lines 43-60: Define helper: make df from filenames
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

### Lines 63-63: Define class TestEnsureDimsInMetas
```python
class TestEnsureDimsInMetas:
```
**EN:** This declaration introduces the `TestEnsureDimsInMetas` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestEnsureDimsInMetas` 测试类，并说明它通过继承承担的职责。

### Lines 64-64: Document the class `TestEnsureDimsInMetas`
```python
    """Tests for _ensure_dims_in_metas."""
```
**EN:** This block stores descriptive text that explains the intent of the surrounding class `TestEnsureDimsInMetas`.
**CN:** 该代码块保存说明性文本，用于解释周围class `TestEnsureDimsInMetas`的设计意图。

### Lines 66-74: Define helper: make meta
```python
    def _make_meta(self, *, cp_size: int = 1, cp_rank: int = 0) -> dict:
        return {
            "sglang_parallel_info": {
                "tp_rank": 0,
                "tp_size": 1,
                "cp_rank": cp_rank,
                "cp_size": cp_size,
            }
        }
```
**EN:** This helper function encapsulates reusable logic inside `TestEnsureDimsInMetas` so the scenario stays organized.
**CN:** 该辅助函数封装了可复用逻辑，供 `TestEnsureDimsInMetas` 内部调用，从而让场景结构更清晰。

### Lines 76-82: Run test: no cp returns metas unchanged
```python
    def test_no_cp_returns_metas_unchanged(self):
        """Without CP parallelism, metas are returned as-is."""
        metas: list[dict] = [self._make_meta(cp_size=1)]
        result = _ensure_dims_in_metas(
            name="input_ids", plugin=_sglang_plugin, metas=metas, ndim=1
        )
        assert result is metas
```
**EN:** This test method exercises no cp returns metas unchanged and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 no cp returns metas unchanged 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 84-90: Run test: dims already present returns metas unchanged
```python
    def test_dims_already_present_returns_metas_unchanged(self):
        """If dims is already in meta, metas are returned as-is."""
        metas: list[dict] = [{**self._make_meta(cp_size=2, cp_rank=0), "dims": "t"}]
        result = _ensure_dims_in_metas(
            name="input_ids", plugin=_sglang_plugin, metas=metas, ndim=1
        )
        assert result is metas
```
**EN:** This test method exercises dims already present returns metas unchanged and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 dims already present returns metas unchanged 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 92-103: Run test: cp sharded sglang input ids infers dims
```python
    def test_cp_sharded_sglang_input_ids_infers_dims(self):
        """CP + input_ids in sglang infers dims 't[cp:zigzag]'."""
        metas: list[dict] = [
            self._make_meta(cp_size=2, cp_rank=0),
            self._make_meta(cp_size=2, cp_rank=1),
        ]
        result = _ensure_dims_in_metas(
            name="input_ids", plugin=_sglang_plugin, metas=metas, ndim=1
        )
        assert result is not metas
        assert result[0]["dims"] == "t[cp:zigzag]"
        assert result[1]["dims"] == "t[cp:zigzag]"
```
**EN:** This test method exercises cp sharded sglang input ids infers dims and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 cp sharded sglang input ids infers dims 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 105-114: Run test: cp sharded sglang positions infers dims
```python
    def test_cp_sharded_sglang_positions_infers_dims(self):
        """CP + positions in sglang infers dims 't[cp:zigzag]'."""
        metas: list[dict] = [
            self._make_meta(cp_size=2, cp_rank=0),
            self._make_meta(cp_size=2, cp_rank=1),
        ]
        result = _ensure_dims_in_metas(
            name="positions", plugin=_sglang_plugin, metas=metas, ndim=1
        )
        assert result[0]["dims"] == "t[cp:zigzag]"
```
**EN:** This test method exercises cp sharded sglang positions infers dims and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 cp sharded sglang positions infers dims 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 116-125: Run test: cp sharded megatron input ids infers dims 1d
```python
    def test_cp_sharded_megatron_input_ids_infers_dims_1d(self):
        """CP + input_ids in megatron (1D) infers dims 't[cp:zigzag]'."""
        metas: list[dict] = [
            {"megatron_parallel_info": {"cp_rank": 0, "cp_size": 2}},
            {"megatron_parallel_info": {"cp_rank": 1, "cp_size": 2}},
        ]
        result = _ensure_dims_in_metas(
            name="input_ids", plugin=_megatron_plugin, metas=metas, ndim=1
        )
        assert result[0]["dims"] == "t[cp:zigzag]"
```
**EN:** This test method exercises cp sharded megatron input ids infers dims 1d and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 cp sharded megatron input ids infers dims 1d 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 127-136: Run test: cp sharded megatron input ids infers dims 2d
```python
    def test_cp_sharded_megatron_input_ids_infers_dims_2d(self):
        """CP + input_ids in megatron (2D) infers dims 'b s[cp:zigzag]'."""
        metas: list[dict] = [
            {"megatron_parallel_info": {"cp_rank": 0, "cp_size": 2}},
            {"megatron_parallel_info": {"cp_rank": 1, "cp_size": 2}},
        ]
        result = _ensure_dims_in_metas(
            name="input_ids", plugin=_megatron_plugin, metas=metas, ndim=2
        )
        assert result[0]["dims"] == "b s[cp:zigzag]"
```
**EN:** This test method exercises cp sharded megatron input ids infers dims 2d and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 cp sharded megatron input ids infers dims 2d 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 138-147: Run test: cp non sharded name returns metas unchanged
```python
    def test_cp_non_sharded_name_returns_metas_unchanged(self):
        """CP + non-sharded tensor name (seq_lens) returns metas as-is."""
        metas: list[dict] = [
            self._make_meta(cp_size=2, cp_rank=0),
            self._make_meta(cp_size=2, cp_rank=1),
        ]
        result = _ensure_dims_in_metas(
            name="seq_lens", plugin=_sglang_plugin, metas=metas, ndim=1
        )
        assert result is metas
```
**EN:** This test method exercises cp non sharded name returns metas unchanged and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 cp non sharded name returns metas unchanged 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 149-164: Run test: unknown plugin returns metas unchanged
```python
    def test_unknown_plugin_returns_metas_unchanged(self):
        """CP + plugin with empty cp_sharded_names returns metas as-is."""

        class _DummyPlugin(_SGLangPlugin):
            @property
            def cp_sharded_names(self) -> frozenset[str]:
                return frozenset()

        metas: list[dict] = [
            self._make_meta(cp_size=2, cp_rank=0),
            self._make_meta(cp_size=2, cp_rank=1),
        ]
        result = _ensure_dims_in_metas(
            name="input_ids", plugin=_DummyPlugin(), metas=metas, ndim=1
        )
        assert result is metas
```
**EN:** This test method exercises unknown plugin returns metas unchanged and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 unknown plugin returns metas unchanged 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 167-167: Define class TestDetectPlugin
```python
class TestDetectPlugin:
```
**EN:** This declaration introduces the `TestDetectPlugin` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestDetectPlugin` 测试类，并说明它通过继承承担的职责。

### Lines 168-177: Run test: discriminating names sglang
```python
    def test_discriminating_names_sglang(self, tmp_path: Path) -> None:
        fn: str = _save_pt(
            tmp_path, name="seq_lens", step=0, rank=0, value=torch.tensor([3])
        )
        df: pl.DataFrame = _make_df_from_filenames([fn])

        result = _detect_plugin(df, dump_path=tmp_path)

        assert result is not None
        assert result.name == "sglang"
```
**EN:** This test method exercises discriminating names sglang and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 discriminating names sglang 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 179-193: Run test: fallback to meta based detection
```python
    def test_fallback_to_meta_based_detection(self, tmp_path: Path) -> None:
        fn: str = _save_pt(
            tmp_path,
            name="input_ids",
            step=0,
            rank=0,
            value=torch.tensor([1, 2, 3]),
            meta={"sglang_parallel_info": {"tp_rank": 0, "tp_size": 1}},
        )
        df: pl.DataFrame = _make_df_from_filenames([fn])

        result = _detect_plugin(df, dump_path=tmp_path)

        assert result is not None
        assert result.name == "sglang"
```
**EN:** This test method exercises fallback to meta based detection and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 fallback to meta based detection 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 195-203: Run test: returns none no match
```python
    def test_returns_none_no_match(self, tmp_path: Path) -> None:
        fn: str = _save_pt(
            tmp_path, name="unrelated_tensor", step=0, rank=0, value=torch.tensor([1])
        )
        df: pl.DataFrame = _make_df_from_filenames([fn])

        result = _detect_plugin(df, dump_path=tmp_path)

        assert result is None
```
**EN:** This test method exercises returns none no match and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 returns none no match 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 206-206: Define class TestLoadNonTensorAux
```python
class TestLoadNonTensorAux:
```
**EN:** This declaration introduces the `TestLoadNonTensorAux` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestLoadNonTensorAux` 测试类，并说明它通过继承承担的职责。

### Lines 207-227: Run test: multi rank mismatch warning
```python
    def test_multi_rank_mismatch_warning(self, tmp_path: Path) -> None:
        fn0: str = _save_pt(tmp_path, name="rids", step=0, rank=0, value=["req_A"])
        fn1: str = _save_pt(tmp_path, name="rids", step=0, rank=1, value=["req_B"])
        df: pl.DataFrame = _make_df_from_filenames([fn0, fn1])

        sink = LogSink()
        with sink.context() as warnings:
            from unittest.mock import patch

            with patch(
                "sglang.srt.debug_utils.comparator.aligner.token_aligner.smart.aux_loader.log_sink",
                sink,
            ):
                result = _load_non_tensor_aux(
                    name="rids", step=0, df=df, dump_path=tmp_path
                )

        assert result == ["req_A"]
        assert len(warnings) == 1
        assert isinstance(warnings[0], ErrorLog)
        assert "rids_mismatch" in warnings[0].category
```
**EN:** This test method exercises multi rank mismatch warning and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 multi rank mismatch warning 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 229-234: Run test: no rows returns none
```python
    def test_no_rows_returns_none(self, tmp_path: Path) -> None:
        df: pl.DataFrame = _make_df_from_filenames([])

        result = _load_non_tensor_aux(name="rids", step=0, df=df, dump_path=tmp_path)

        assert result is None
```
**EN:** This test method exercises no rows returns none and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 no rows returns none 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 237-237: Define class TestLoadAndAlignAuxTensor
```python
class TestLoadAndAlignAuxTensor:
```
**EN:** This declaration introduces the `TestLoadAndAlignAuxTensor` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestLoadAndAlignAuxTensor` 测试类，并说明它通过继承承担的职责。

### Lines 238-291: Run test: multi rank no dims emits warning
```python
    def test_multi_rank_no_dims_emits_warning(self, tmp_path: Path) -> None:
        fn0: str = _save_pt(
            tmp_path,
            name="input_ids",
            step=0,
            rank=0,
            value=torch.tensor([1, 2, 3]),
            meta={
                "sglang_parallel_info": {
                    "tp_rank": 0,
                    "tp_size": 2,
                    "cp_rank": 0,
                    "cp_size": 1,
                }
            },
        )
        fn1: str = _save_pt(
            tmp_path,
            name="input_ids",
            step=0,
            rank=1,
            value=torch.tensor([4, 5, 6]),
            meta={
                "sglang_parallel_info": {
                    "tp_rank": 1,
                    "tp_size": 2,
                    "cp_rank": 0,
                    "cp_size": 1,
                }
            },
        )
        df: pl.DataFrame = _make_df_from_filenames([fn0, fn1])

        sink = LogSink()
        with sink.context() as warnings:
            from unittest.mock import patch

            with patch(
                "sglang.srt.debug_utils.comparator.aligner.token_aligner.smart.aux_loader.log_sink",
                sink,
            ):
                result = _load_and_align_aux_tensor(
                    name="input_ids",
                    step=0,
                    df=df,
                    dump_path=tmp_path,
                    plugin=_sglang_plugin,
                )

        assert result is not None
        assert torch.equal(result, torch.tensor([1, 2, 3]))
        assert len(warnings) == 1
        assert isinstance(warnings[0], InfoLog)
        assert "aux_no_dims" in warnings[0].category
```
**EN:** This test method exercises multi rank no dims emits warning and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 multi rank no dims emits warning 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 294-294: Define class TestLoadNonTensorAuxDp
```python
class TestLoadNonTensorAuxDp:
```
**EN:** This declaration introduces the `TestLoadNonTensorAuxDp` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestLoadNonTensorAuxDp` 测试类，并说明它通过继承承担的职责。

### Lines 295-295: Document the class `TestLoadNonTensorAuxDp`
```python
    """DP filtering in _load_non_tensor_aux."""
```
**EN:** This block stores descriptive text that explains the intent of the surrounding class `TestLoadNonTensorAuxDp`.
**CN:** 该代码块保存说明性文本，用于解释周围class `TestLoadNonTensorAuxDp`的设计意图。

### Lines 297-339: Run test: dp2 non tensor returns value
```python
    def test_dp2_non_tensor_returns_value(self, tmp_path: Path) -> None:
        """DP=2 non-tensor aux: both ranks have same value, filter keeps all (non-tensor)."""
        fn0: str = _save_pt(
            tmp_path,
            name="rids",
            step=0,
            rank=0,
            value=["req_A"],
            meta={
                "sglang_parallel_info": {
                    "dp_rank": 0,
                    "dp_size": 2,
                }
            },
        )
        fn1: str = _save_pt(
            tmp_path,
            name="rids",
            step=0,
            rank=1,
            value=["req_A"],
            meta={
                "sglang_parallel_info": {
                    "dp_rank": 1,
                    "dp_size": 2,
                }
            },
        )
        df: pl.DataFrame = _make_df_from_filenames([fn0, fn1])

        sink = LogSink()
        with sink.context():
            from unittest.mock import patch

            with patch(
                "sglang.srt.debug_utils.comparator.aligner.token_aligner.smart.aux_loader.log_sink",
                sink,
            ):
                result = _load_non_tensor_aux(
                    name="rids", step=0, df=df, dump_path=tmp_path
                )

        assert result == ["req_A"]
```
**EN:** This test method exercises dp2 non tensor returns value and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 dp2 non tensor returns value 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 342-342: Define class TestLoadAndAlignAuxTensorDp
```python
class TestLoadAndAlignAuxTensorDp:
```
**EN:** This declaration introduces the `TestLoadAndAlignAuxTensorDp` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestLoadAndAlignAuxTensorDp` 测试类，并说明它通过继承承担的职责。

### Lines 343-343: Document the class `TestLoadAndAlignAuxTensorDp`
```python
    """DP filtering in _load_and_align_aux_tensor."""
```
**EN:** This block stores descriptive text that explains the intent of the surrounding class `TestLoadAndAlignAuxTensorDp`.
**CN:** 该代码块保存说明性文本，用于解释周围class `TestLoadAndAlignAuxTensorDp`的设计意图。

### Lines 345-384: Run test: dp2 tensor one empty
```python
    def test_dp2_tensor_one_empty(self, tmp_path: Path) -> None:
        """DP=2 tensor aux: rank 0 has data, rank 1 empty -> returns rank 0 tensor."""
        fn0: str = _save_pt(
            tmp_path,
            name="input_ids",
            step=0,
            rank=0,
            value=torch.tensor([10, 20, 30]),
            meta={
                "sglang_parallel_info": {
                    "dp_rank": 0,
                    "dp_size": 2,
                }
            },
        )
        fn1: str = _save_pt(
            tmp_path,
            name="input_ids",
            step=0,
            rank=1,
            value=torch.tensor([]),
            meta={
                "sglang_parallel_info": {
                    "dp_rank": 1,
                    "dp_size": 2,
                }
            },
        )
        df: pl.DataFrame = _make_df_from_filenames([fn0, fn1])

        result = _load_and_align_aux_tensor(
            name="input_ids",
            step=0,
            df=df,
            dump_path=tmp_path,
            plugin=_sglang_plugin,
        )

        assert result is not None
        assert torch.equal(result, torch.tensor([10, 20, 30]))
```
**EN:** This test method exercises dp2 tensor one empty and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 dp2 tensor one empty 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 387-388: Expose unittest entrypoint
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
- Internal modules / 内部模块: `sglang.srt.debug_utils.comparator.aligner.token_aligner.smart.aux_loader`, `sglang.srt.debug_utils.comparator.aligner.token_aligner.smart.aux_plugins`, `sglang.srt.debug_utils.comparator.log_sink`, `sglang.srt.debug_utils.comparator.output_types`, `sglang.test.ci.ci_register`
- External and stdlib modules / 外部与标准库模块: `pathlib`, `polars`, `pytest`, `sys`, `torch`, `unittest.mock`
- Notable symbols / 关键符号: None / 无
