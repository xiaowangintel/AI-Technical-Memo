# dump_comparator.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/debug_utils/dump_comparator.py`
- **Repository**: `sgl-project/sglang`
- **Purpose**: This module belongs to SGLang's SRT debug utilities and focuses on shared debugging utilities. It mainly implements the core logic needed by this part of the debug toolchain. / 该模块属于 SGLang 的 SRT 调试工具，聚焦于共享调试工具。它主要用于实现该调试工具链所需的核心逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-6: Document the module intent / 说明模块意图
```python
"""Simplified dump comparator — a self-contained single-file script for comparing
two dump directories tensor-by-tensor.

For advanced features (unshard, token alignment, per-dimension annotations), see the
full ``comparator/`` package: ``python -m sglang.srt.debug_utils.comparator``.
"""
```
**EN:** The docstring states the module intent, so readers can understand the debugging scenario before reading the implementation details.
**CN:** 文档字符串先说明模块意图，让读者在进入实现细节之前就能理解对应的调试场景。

### Lines 8-17: Import dependencies and shared types / 导入依赖与共享类型
```python
import argparse
import functools
import re
from dataclasses import dataclass
from pathlib import Path
from typing import Callable, List, Optional

import torch

from sglang.srt.debug_utils.dumper import get_truncated_value
```
**EN:** This import block pulls in the standard library, third-party packages, and neighboring SGLang helpers that the rest of the file builds on.
**CN:** 这一段导入了标准库、第三方包以及相邻的 SGLang 辅助模块，后续实现都会建立在这些依赖之上。

### Lines 20-84: Implement function `main` / 实现函数 `main`
```python
def main(args):
    import polars as pl

    from sglang.srt.debug_utils.dump_loader import find_row, read_meta

    df_target = read_meta(args.target_path)
    df_target = df_target.filter(
        (pl.col("step") >= args.start_step) & (pl.col("step") <= args.end_step)
    )
    if args.filter:
        df_target = df_target.filter(pl.col("filename").str.contains(args.filter))
    assert all(c in df_target.columns for c in ["rank", "step", "dump_index", "name"])

    df_baseline = read_meta(args.baseline_path)
    print("df_target", df_target)
    print("df_baseline", df_baseline)

    tensor_dim_descs: List[TensorDimDesc] = _get_tensor_dim_descs()

    for row in df_target.iter_rows(named=True):
        path_target = Path(args.target_path) / row["filename"]

        tensor_dim_desc: Optional[TensorDimDesc] = None
        if tensor_dim_descs:
            matched: list[TensorDimDesc] = [
                desc
                for desc in tensor_dim_descs
                if re.search(desc.pattern, row["filename"]) is not None
            ]
            if matched:
                tensor_dim_desc = matched[0]

        row_baseline = find_row(
            df_baseline,
            conditions=dict(
                step=row["step"],
                **{
                    k: v
                    for k, v in row.items()
                    if k not in ["step", "dump_index", "filename"]
                },
            ),
        )

        if row_baseline is None:
            print(f"Skip: target={str(path_target)} since no baseline")
            x_target = _load_object(path_target)
            if x_target is not None:
                print(f"x_target(sample)={get_truncated_value(x_target)}")
            continue

        path_baseline = Path(args.baseline_path) / row_baseline["filename"]
        print(
            f"Check:\n"
            f"target={str(path_target)} (duplicate_index={row['duplicate_index']})\n"
            f"baseline={str(path_baseline)} (duplicate_index={row_baseline['duplicate_index']})"
        )
        check_tensor_pair(
            path_baseline=path_baseline,
            path_target=path_target,
            diff_threshold=args.diff_threshold,
            name=row["name"],
            tensor_dim_desc=tensor_dim_desc,
        )
        print()
```
**EN:** Function `main` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps.
**CN:** 函数 `main` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。

### Lines 87-185: Implement function `check_tensor_pair` / 实现函数 `check_tensor_pair`
```python
def check_tensor_pair(
    path_baseline,
    path_target,
    diff_threshold: float = 1e-3,
    name="",
    tensor_dim_desc: Optional["TensorDimDesc"] = None,
):
    x_baseline = _load_object(path_baseline)
    x_target = _load_object(path_target)

    if x_baseline is None or x_target is None:
        print(
            f"Skip comparison because of None: x_baseline={x_baseline}, x_target={x_target}"
        )
        return

    print(
        f"Raw "
        f"[shape] {x_baseline.shape} vs {x_target.shape}\t"
        f"[{'' if x_baseline.dtype == x_target.dtype else '🟠'}dtype] {x_baseline.dtype} vs {x_target.dtype}"
    )

    if tensor_dim_desc is not None:
        import einops

        x_baseline = einops.rearrange(
            x_baseline,
            tensor_dim_desc.baseline_desc + " -> " + tensor_dim_desc.target_desc,
        )
        if tensor_dim_desc.baseline_cropper is not None:
            print("Apply baseline_cropper")
            x_baseline = tensor_dim_desc.baseline_cropper(x_baseline)

    x_baseline, x_target = _comparison_preprocessor(x_baseline, x_target, name=name)
    x_baseline = _try_unify_shape(x_baseline, target_shape=x_target.shape)

    print(
        f"After preprocessor "
        f"[shape] {x_baseline.shape} vs {x_target.shape}\t"
        f"[dtype] {x_baseline.dtype} vs {x_target.dtype}"
    )

    x_baseline_original_dtype = x_baseline.dtype
    x_target_original_dtype = x_target.dtype

    x_target = x_target.float()
    x_baseline = x_baseline.float()

    for name, fn in [
        ("mean", torch.mean),
# ... truncated for brevity ...
    needs_print = diff_info["max_abs_diff"] > 1e-3

    if (x_baseline_original_dtype != x_target_original_dtype) and (
        (
            downcast_dtype := _compute_smaller_dtype(
                x_baseline_original_dtype, x_target_original_dtype
            )
        )
        is not None
    ):
        _compute_and_print_diff(
            x_baseline=x_baseline.to(downcast_dtype),
            x_target=x_target.to(downcast_dtype),
            diff_threshold=diff_threshold,
            prefix_text=f"When downcast to {downcast_dtype}: ",
        )

    if needs_print:
        print(f"x_baseline(sample)={get_truncated_value(x_baseline)}")
        print(f"x_target(sample)={get_truncated_value(x_target)}")
```
**EN:** Function `check_tensor_pair` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps. The snippet is abbreviated to keep the analysis readable.
**CN:** 函数 `check_tensor_pair` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。 为了保持分析可读性，这里的代码片段做了节选。

### Lines 188-212: Implement helper `_compute_and_print_diff` / 实现辅助函数 `_compute_and_print_diff`
```python
def _compute_and_print_diff(
    x_baseline, x_target, diff_threshold: float, prefix_text=""
):
    raw_abs_diff = (x_target - x_baseline).abs()

    max_abs_diff = raw_abs_diff.max().item()
    mean_abs_diff = raw_abs_diff.mean().item()
    rel_diff = _calc_rel_diff(x_target, x_baseline)

    rel_diff_marker: str = "❌" if rel_diff > diff_threshold else "✅"
    print(
        prefix_text
        + f"{rel_diff_marker} rel_diff={rel_diff}\t"
        + f"max_abs_diff={max_abs_diff}\t"
        + f"mean_abs_diff={mean_abs_diff}"
    )

    max_diff_coord = _argmax_coord(raw_abs_diff)
    print(
        f"max_abs_diff happens at coord={max_diff_coord} with "
        f"baseline={x_baseline[max_diff_coord].item()} "
        f"target={x_target[max_diff_coord].item()}"
    )

    return dict(max_abs_diff=max_abs_diff)
```
**EN:** Function `_compute_and_print_diff` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps.
**CN:** 函数 `_compute_and_print_diff` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。

### Lines 215-217: Implement helper `_argmax_coord` / 实现辅助函数 `_argmax_coord`
```python
def _argmax_coord(x: torch.Tensor) -> tuple:
    flat_idx = x.argmax()
    return tuple(idx.item() for idx in torch.unravel_index(flat_idx, x.shape))
```
**EN:** Function `_argmax_coord` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps.
**CN:** 函数 `_argmax_coord` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。

### Lines 220-225: Implement helper `_compute_smaller_dtype` / 实现辅助函数 `_compute_smaller_dtype`
```python
def _compute_smaller_dtype(dtype_a, dtype_b):
    info_dict = {
        (torch.float32, torch.bfloat16): torch.bfloat16,
        # ... add more ...
    }
    return info_dict.get((dtype_a, dtype_b)) or info_dict.get((dtype_b, dtype_a))
```
**EN:** Function `_compute_smaller_dtype` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps.
**CN:** 函数 `_compute_smaller_dtype` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。

### Lines 228-238: Implement helper `_try_unify_shape` / 实现辅助函数 `_try_unify_shape`
```python
def _try_unify_shape(x: torch.Tensor, target_shape):
    x_shape = x.shape
    num_dim_to_remove = len(x_shape) - len(target_shape)
    if (x_shape[num_dim_to_remove:] == target_shape) and all(
        val == 1 for val in x_shape[:num_dim_to_remove]
    ):
        out = functools.reduce(lambda a, _: a.squeeze(0), range(num_dim_to_remove), x)
        print(f"Unify shape: {x_shape} -> {out.shape} (to match {target_shape})")
        return out

    return x
```
**EN:** Function `_try_unify_shape` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps.
**CN:** 函数 `_try_unify_shape` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。

### Lines 242-246: Implement helper `_calc_rel_diff` / 实现辅助函数 `_calc_rel_diff`
```python
def _calc_rel_diff(x: torch.Tensor, y: torch.Tensor):
    x, y = x.double(), y.double()
    denominator = (x * x + y * y).sum()
    sim = 2 * (x * y).sum() / denominator
    return 1 - sim
```
**EN:** Function `_calc_rel_diff` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps.
**CN:** 函数 `_calc_rel_diff` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。

### Lines 249-262: Implement helper `_load_object` / 实现辅助函数 `_load_object`
```python
def _load_object(path):
    try:
        x = torch.load(path, weights_only=False)
    except Exception as e:
        print(f"Skip load {path} since error {e}")
        return None

    if isinstance(x, dict) and "value" in x:
        x = x["value"]

    if not isinstance(x, torch.Tensor):
        print(f"Skip load {path} since {type(x)=} is not a Tensor ({x=})")
        return None
    return x.cuda()
```
**EN:** Function `_load_object` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps.
**CN:** 函数 `_load_object` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。

### Lines 265-267: Implement helper `_comparison_preprocessor` / 实现辅助函数 `_comparison_preprocessor`
```python
def _comparison_preprocessor(x_baseline, x_target, name):
    """Customization endpoint. Can insert arbitrary adhoc postprocessing logic here."""
    return x_baseline, x_target
```
**EN:** Function `_comparison_preprocessor` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps.
**CN:** 函数 `_comparison_preprocessor` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。

### Lines 271-271: Define class `TensorDimDesc` and class context / 定义类 `TensorDimDesc`及类上下文
```python
class TensorDimDesc:
```
**EN:** This section introduces `TensorDimDesc`, including its inheritance and any class-level context, so later fields or methods have a clear abstraction boundary.
**CN:** 这一部分引入了 `TensorDimDesc`，包括其继承关系和类级上下文，从而为后续字段或方法建立清晰的抽象边界。

### Lines 272-275: Declare fields for `TensorDimDesc` such as `pattern`, `baseline_desc`, `target_desc`, `baseline_cropper` / 为 `TensorDimDesc` 声明字段，例如 `pattern`, `baseline_desc`, `target_desc`, `baseline_cropper`
```python
    pattern: str
    baseline_desc: str
    target_desc: str
    baseline_cropper: Optional[Callable[[torch.Tensor], torch.Tensor]] = None
```
**EN:** These lines declare the state carried by `TensorDimDesc`. The annotated fields show what data instances expose to the surrounding debugging pipeline.
**CN:** 这些代码声明了 `TensorDimDesc` 持有的状态。带注解的字段展示了实例会向周边调试流水线暴露哪些数据。

### Lines 278-281: Implement helper `_get_tensor_dim_descs` / 实现辅助函数 `_get_tensor_dim_descs`
```python
def _get_tensor_dim_descs() -> List[TensorDimDesc]:
    """Customization endpoint. Return a list of TensorDimDesc to rearrange baseline
    dimensions to match target layout via einops before comparison."""
    return []
```
**EN:** Function `_get_tensor_dim_descs` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps.
**CN:** 函数 `_get_tensor_dim_descs` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。

### Lines 284-296: Provide a command-line entrypoint / 提供命令行入口
```python
if __name__ == "__main__":
    # python -m sglang.srt.debug_utils.dump_comparator --baseline-path ... --target-path ...
    parser = argparse.ArgumentParser()
    parser.add_argument("--baseline-path", type=str)
    parser.add_argument("--target-path", type=str)
    parser.add_argument("--start-step", type=int, default=0)
    parser.add_argument("--end-step", type=int, default=1000000)
    parser.add_argument("--diff-threshold", type=float, default=1e-3)
    parser.add_argument(
        "--filter", type=str, default=None, help="Regex to filter filenames"
    )
    args = parser.parse_args()
    main(args)
```
**EN:** This conditional branch selects behavior based on runtime state, optional inputs, or developer-facing entry conditions.
**CN:** 该条件分支会根据运行时状态、可选输入或面向开发者的入口条件来选择不同的行为。

## Key Concepts / 关键概念
- **Core symbols / 核心符号**: `TensorDimDesc`, `main`, `check_tensor_pair`, `_compute_and_print_diff`, `_argmax_coord`, `_compute_smaller_dtype`, `_try_unify_shape`, `_calc_rel_diff`, `_load_object`, `_comparison_preprocessor`, `_get_tensor_dim_descs`
- **Module role / 模块角色**: Shared debugging utilities / 共享调试工具
- **Implementation focus / 实现重点**: Implements the core logic needed by this part of the debug toolchain / 实现该调试工具链所需的核心逻辑

## Dependencies / 依赖关系
- **Standard library / 标准库**: `argparse`, `functools`, `re`, `dataclasses`, `pathlib`, `typing`
- **Third-party / 第三方**: `torch`, `polars`, `einops`
- **Internal / 内部**: `sglang.srt.debug_utils.dumper`, `sglang.srt.debug_utils.dump_loader`
