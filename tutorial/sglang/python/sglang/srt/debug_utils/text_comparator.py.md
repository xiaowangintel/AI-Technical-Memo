# text_comparator.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/debug_utils/text_comparator.py`
- **Repository**: `sgl-project/sglang`
- **Purpose**: This module belongs to SGLang's SRT debug utilities and focuses on text output comparison. It mainly compares textual outputs and highlights mismatches succinctly. / 该模块属于 SGLang 的 SRT 调试工具，聚焦于文本输出比较。它主要用于比较文本输出并简明地标出不一致之处。

## Line-by-Line Analysis / 逐行分析
### Lines 1-6: Import dependencies and shared types / 导入依赖与共享类型
```python
import argparse
import hashlib
import json
from pathlib import Path

import polars as pl
```
**EN:** This import block pulls in the standard library, third-party packages, and neighboring SGLang helpers that the rest of the file builds on.
**CN:** 这一段导入了标准库、第三方包以及相邻的 SGLang 辅助模块，后续实现都会建立在这些依赖之上。

### Lines 8-13: Declare module-level symbols such as `_DESCRIPTION` / 声明模块级符号，例如 `_DESCRIPTION`
```python
_DESCRIPTION = """Compare and find differences to benchmark outputs.

Supported inputs:
* The samples jsonl from `lm_eval --log_samples --output_path FOLDER_NAME`
* The output from `gsm8k/bench_sglang.py --raw-result-file FILE_NAME` (or mmlu)
"""
```
**EN:** This block defines reusable module-level symbols such as constants, aliases, or configuration values that simplify the later control flow.
**CN:** 这一段定义了可复用的模块级符号，例如常量、别名或配置值，从而简化后续控制流程。

### Lines 16-71: Implement function `main` / 实现函数 `main`
```python
def main(args):
    if args.data_type == "simple_evals":
        df_input = _compute_df_input_mode_simple_evals(args)
    else:
        df_input = _transform_df_input(_compute_df_raw(args))

    assert all(
        c in df_input.columns
        for c in ["category", "trial_index", "prompt_id", "prompt", "output", "correct"]
    )

    df_meta = _compute_df_meta(df_input)

    df_correctness_per_trial = df_input.group_by(
        "category", "trial_index", maintain_order=True
    ).agg(pl.col("correct").mean())
    df_correctness_delta = (
        df_meta.group_by("correctness_delta").len().sort("correctness_delta")
    )
    df_good_to_bad = df_meta.filter(pl.col("correctness_delta") < 0)
    df_bad_to_good = df_meta.filter(pl.col("correctness_delta") > 0)

    print(f"Dump output to {args.output_path}")
    Path(args.output_path).write_text(
        json.dumps(
            dict(
                df_meta=df_meta.to_dicts(),
                df_good_to_bad=df_good_to_bad.to_dicts(),
                df_bad_to_good=df_bad_to_good.to_dicts(),
            ),
            indent=4,
        ),
    )

    if not args.disable_print_details:
        with pl.Config(
            fmt_str_lengths=10000,
            tbl_cols=-1,
            tbl_rows=-1,
            tbl_width_chars=-1,
            tbl_formatting="UTF8_FULL",
        ):
            print("====== Correctness per trial ======")
            print(df_correctness_per_trial)

            print(
                "====== Correctness Delta (-1.0 means all-right becomes all-wrong) ======"
            )
            print(df_correctness_delta)

            for name, df in [
                ("Good->Bad", df_good_to_bad),
                ("Bad->Good", df_bad_to_good),
            ]:
                print(f"====== Concrete Examples: {name} ======")
                print(df)
```
**EN:** Function `main` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps.
**CN:** 函数 `main` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。

### Lines 74-80: Implement helper `_compute_df_input_mode_simple_evals` / 实现辅助函数 `_compute_df_input_mode_simple_evals`
```python
def _compute_df_input_mode_simple_evals(args):
    return pl.concat(
        [
            _compute_df_input_one_mode_simple_evals(**info)
            for info in _get_file_infos(args=args)
        ]
    )
```
**EN:** Function `_compute_df_input_mode_simple_evals` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps.
**CN:** 函数 `_compute_df_input_mode_simple_evals` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。

### Lines 83-104: Implement helper `_compute_df_input_one_mode_simple_evals` / 实现辅助函数 `_compute_df_input_one_mode_simple_evals`
```python
def _compute_df_input_one_mode_simple_evals(path, category, trial_index):
    data = json.loads(Path(path).read_text())
    rows = []

    for single_eval_result in data["metadata"]["single_eval_results"]:
        prompt = single_eval_result["example_level_metadata"][
            "actual_queried_prompt_messages"
        ]
        score = single_eval_result["score"]
        assert score in {0.0, 1.0}, f"{score=}"

        row = dict(
            category=category,
            trial_index=trial_index,
            prompt_id=_compute_id_from_object(prompt),
            prompt=json.dumps(prompt),
            output=single_eval_result["example_level_metadata"]["response_text"],
            correct=score == 1.0,
        )
        rows.append(row)

    return pl.DataFrame(rows)
```
**EN:** Function `_compute_df_input_one_mode_simple_evals` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps.
**CN:** 函数 `_compute_df_input_one_mode_simple_evals` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。

### Lines 107-111: Implement helper `_compute_id_from_object` / 实现辅助函数 `_compute_id_from_object`
```python
def _compute_id_from_object(obj):
    if isinstance(obj, pl.Series):
        obj = obj.to_list()
    json_str = json.dumps(obj, sort_keys=True, ensure_ascii=False)
    return hashlib.sha256(json_str.encode("utf-8")).hexdigest()
```
**EN:** Function `_compute_id_from_object` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps.
**CN:** 函数 `_compute_id_from_object` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。

### Lines 114-124: Implement helper `_compute_df_raw` / 实现辅助函数 `_compute_df_raw`
```python
def _compute_df_raw(args):
    return pl.concat(
        [
            _read_df_raw(
                path=info["path"],
                category=info["category"],
                trial_index=info["trial_index"],
            )
            for info in _get_file_infos(args=args)
        ]
    )
```
**EN:** Function `_compute_df_raw` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps.
**CN:** 函数 `_compute_df_raw` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。

### Lines 127-135: Implement helper `_get_file_infos` / 实现辅助函数 `_get_file_infos`
```python
def _get_file_infos(args):
    return [
        dict(path=path, category=category, trial_index=trial_index)
        for category, paths in [
            ("baseline", args.baseline_path),
            ("target", args.target_path),
        ]
        for trial_index, path in enumerate(paths)
    ]
```
**EN:** Function `_get_file_infos` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps.
**CN:** 函数 `_get_file_infos` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。

### Lines 138-141: Implement helper `_read_df_raw` / 实现辅助函数 `_read_df_raw`
```python
def _read_df_raw(path: str, category: str, trial_index: int):
    return pl.read_ndjson(path).with_columns(
        category=pl.lit(category), trial_index=trial_index
    )
```
**EN:** Function `_read_df_raw` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps.
**CN:** 函数 `_read_df_raw` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。

### Lines 144-170: Implement helper `_transform_df_input` / 实现辅助函数 `_transform_df_input`
```python
def _transform_df_input(df: pl.DataFrame):
    if "doc_id" in df.columns:
        print("Transform mode: lm_eval")

        filter_names = df["filter"].unique(maintain_order=True).to_list()
        if len(filter_names) > 1:
            filter_name = filter_names[0]
            print(f"Choose {filter_name=} among {filter_names}")
            df = df.filter(pl.col("filter") == filter_name)

        df = df.select(
            pl.col("category"),
            pl.col("trial_index"),
            prompt_id=pl.col("doc_id"),
            prompt=pl.col("arguments").struct.field("gen_args_0").struct.field("arg_0"),
            output=pl.col("resps").list.get(0).list.get(0),
            correct=pl.col("exact_match").cast(bool),
        )

        return df
    elif "prompt_id" in df.columns:
        print("Transform mode: SGLang bench")
        return df
    else:
        raise Exception(
            f"Unknown data: {df.columns}. You may need to set `--data-type` if using e.g. simple_evals."
        )
```
**EN:** Function `_transform_df_input` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps.
**CN:** 函数 `_transform_df_input` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。

### Lines 173-185: Implement helper `_compute_df_meta` / 实现辅助函数 `_compute_df_meta`
```python
def _compute_df_meta(df_input: pl.DataFrame):
    df_input = df_input.sort("prompt_id", "category", "trial_index")
    df_meta = pl.DataFrame(
        [
            _handle_one_prompt(df_one_prompt)
            for df_one_prompt in df_input.partition_by("prompt_id", maintain_order=True)
        ]
    )
    df_meta = df_meta.with_columns(
        correctness_delta=pl.col("correctness_target") - pl.col("correctness_baseline"),
    )
    df_meta = df_meta.sort("correctness_delta", "output_same_prefix_len")
    return df_meta
```
**EN:** Function `_compute_df_meta` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps.
**CN:** 函数 `_compute_df_meta` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。

### Lines 188-213: Implement helper `_handle_one_prompt` / 实现辅助函数 `_handle_one_prompt`
```python
def _handle_one_prompt(df_one_prompt: pl.DataFrame):
    assert (
        len(set(_compute_id_from_object(obj) for obj in df_one_prompt["prompt"])) == 1
    )

    df_baseline = df_one_prompt.filter(pl.col("category") == "baseline")
    df_target = df_one_prompt.filter(pl.col("category") == "target")

    outputs_baseline = df_baseline["output"].to_list()
    outputs_target = df_target["output"].to_list()

    output_same_prefix_len = max(
        _compute_str_prefix_len(output_baseline, output_target)
        for output_baseline in outputs_baseline
        for output_target in outputs_target
    )

    return dict(
        prompt_id=df_one_prompt[0, "prompt_id"],
        correctness_baseline=df_baseline["correct"].mean(),
        correctness_target=df_target["correct"].mean(),
        output_same_prefix_len=output_same_prefix_len,
        prompt=df_one_prompt[0, "prompt"],
        outputs_baseline=outputs_baseline,
        outputs_target=outputs_target,
    )
```
**EN:** Function `_handle_one_prompt` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps.
**CN:** 函数 `_handle_one_prompt` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。

### Lines 216-221: Implement helper `_compute_str_prefix_len` / 实现辅助函数 `_compute_str_prefix_len`
```python
def _compute_str_prefix_len(a: str, b: str) -> int:
    min_len = min(len(a), len(b))
    for i in range(min_len):
        if a[i] != b[i]:
            return i
    return min_len
```
**EN:** Function `_compute_str_prefix_len` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps.
**CN:** 函数 `_compute_str_prefix_len` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。

### Lines 224-234: Provide a command-line entrypoint / 提供命令行入口
```python
if __name__ == "__main__":
    parser = argparse.ArgumentParser(description=_DESCRIPTION)
    parser.add_argument("--data-type", type=str, default="auto")
    parser.add_argument("--baseline-path", type=str, nargs="+")
    parser.add_argument("--target-path", type=str, nargs="+")
    parser.add_argument(
        "--output-path", type=str, default="/tmp/text_comparator_output.json"
    )
    parser.add_argument("--disable-print-details", action="store_true")
    args = parser.parse_args()
    main(args)
```
**EN:** This conditional branch selects behavior based on runtime state, optional inputs, or developer-facing entry conditions.
**CN:** 该条件分支会根据运行时状态、可选输入或面向开发者的入口条件来选择不同的行为。

## Key Concepts / 关键概念
- **Core symbols / 核心符号**: `main`, `_compute_df_input_mode_simple_evals`, `_compute_df_input_one_mode_simple_evals`, `_compute_id_from_object`, `_compute_df_raw`, `_get_file_infos`, `_read_df_raw`, `_transform_df_input`, `_compute_df_meta`, `_handle_one_prompt`, `_compute_str_prefix_len`
- **Module role / 模块角色**: Text output comparison / 文本输出比较
- **Implementation focus / 实现重点**: Compares textual outputs and highlights mismatches succinctly / 比较文本输出并简明地标出不一致之处

## Dependencies / 依赖关系
- **Standard library / 标准库**: `argparse`, `hashlib`, `json`, `pathlib`
- **Third-party / 第三方**: `polars`
- **Internal / 内部**: None / 无
