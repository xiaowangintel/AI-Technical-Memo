# eval_utils.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `benchmark/reasoning_benchmark/eval_utils.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This Python module focuses on reasoning eval. It primarily organizes benchmark execution and reporting. / 该 Python 模块聚焦于 reasoning eval 相关流程。它主要用于组织基准测试执行与结果汇报。

## Line-by-Line Analysis / 逐行分析
### Lines 3-8: Imports and setup / 导入与初始化
```python
from math import isclose

import regex
from sympy import N, simplify
from sympy.parsing.latex import parse_latex
from sympy.parsing.sympy_parser import parse_expr
```
**EN:** This block imports required libraries and establishes module-level constants or helpers used later in the file.
**CN:** 该代码块导入所需库，并建立后续逻辑会使用到的模块级常量或辅助配置。

### Lines 11-25: Function `parse_digits` / 函数 `parse_digits`
```python
def parse_digits(num):
    # format: 234.23 || 23%
    num = regex.sub(",", "", str(num))
    try:
        return float(num)
    except:
        if num.endswith("%"):
            num = num[:-1]
            if num.endswith("\\"):
                num = num[:-1]
            try:
                return float(num) / 100
            except:
                pass
    return None
```
**EN:** `parse_digits` is a function that implements the core logic for this scope. It returns `None` to the caller. Notable calls include `regex.sub`, `str`, `float`.
**CN:** `parse_digits` 是一个函数，用于实现当前作用域的核心逻辑。它会向调用方返回 `None`。其中较关键的调用包括 `regex.sub`, `str`, `float`。

### Lines 28-30: Function `is_digit` / 函数 `is_digit`
```python
def is_digit(num):
    # paired with parse_digits
    return parse_digits(num) is not None
```
**EN:** `is_digit` is a function that implements the core logic for this scope. It returns `parse_digits(num) is not None` to the caller. Notable calls include `parse_digits`.
**CN:** `is_digit` 是一个函数，用于实现当前作用域的核心逻辑。它会向调用方返回 `parse_digits(num) is not None`。其中较关键的调用包括 `parse_digits`。

### Lines 33-56: Function `symbolic_equal` / 函数 `symbolic_equal`
```python
def symbolic_equal(a, b):
    def _parse(s):
        for f in [parse_latex, parse_expr]:
            try:
                return f(s)
            except:
                pass
        return s

    a = _parse(a)
    b = _parse(b)

    try:
        if simplify(a - b) == 0:
            return True
    except:
        pass

    try:
        if isclose(N(a), N(b), abs_tol=1e-3):
            return True
    except:
        pass
    return False
```
**EN:** `symbolic_equal` is a function that implements the core logic for this scope. It returns `False` to the caller. Notable calls include `_parse`, `isclose`, `simplify`.
**CN:** `symbolic_equal` 是一个函数，用于实现当前作用域的核心逻辑。它会向调用方返回 `False`。其中较关键的调用包括 `_parse`, `isclose`, `simplify`。

### Lines 59-206: Function `math_equal` / 函数 `math_equal`
```python
def math_equal(prediction, reference, include_percentage=True, is_close=True):
    """
    Exact match of math if and only if:
    1. numerical equal: both can convert to float and are equal
    2. symbolic equal: both can convert to sympy expression and are equal
    """
    if str(prediction) == str(reference):
        return True

    try:  # 1. numerical equal
        if is_digit(prediction) and is_digit(reference):
            prediction = parse_digits(prediction)
            reference = parse_digits(reference)
            # number questions
            if include_percentage:
                gt_result = [reference / 100, reference, reference * 100]
            else:
                gt_result = [reference]
            for item in gt_result:
                try:
                    if is_close:
                        if isclose(item, prediction, abs_tol=1e-3):
                            return True
                    else:
                        if item == prediction:
                            return True
                except Exception:
                    continue
            return False
    except:
        pass

    if not prediction and prediction not in [0, False]:
        return False

    # 2. symbolic equal
    reference = str(reference).strip()
    prediction = str(prediction).strip()

    if (
        regex.match(r"(\(|\[).+(\)|\])", prediction) is not None
        and regex.match(r"(\(|\[).+(\)|\])", reference) is not None
    ):
        pred_parts = prediction[1:-1].split(",")
        ref_parts = reference[1:-1].split(",")
        if len(pred_parts) == len(ref_parts):
            if all(
                [
                    math_equal(
                        pred_parts[i], ref_parts[i], include_percentage, is_close
                    )
                    for i in range(len(pred_parts))
                ]
            ):
                return True

    # Add back matrix comparison
    if (
        (
            prediction.startswith("\\begin{pmatrix}")
            or prediction.startswith("\\begin{bmatrix}")
        )
        and (
            prediction.endswith("\\end{pmatrix}")
            or prediction.endswith("\\end{bmatrix}")
        )
        and (
            reference.startswith("\\begin{pmatrix}")
            or reference.startswith("\\begin{bmatrix}")
        )
        and (
            reference.endswith("\\end{pmatrix}") or reference.endswith("\\end{bmatrix}")
        )
    ):
        pred_lines = [
            line.strip()
            for line in prediction[
                len("\\begin{pmatrix}") : -len("\\end{pmatrix}")
            ].split("\\\\")
            if line.strip()
        ]
        ref_lines = [
            line.strip()
            for line in reference[
                len("\\begin{pmatrix}") : -len("\\end{pmatrix}")
            ].split("\\\\")
            if line.strip()
        ]
        matched = True
        if len(pred_lines) == len(ref_lines):
            for pred_line, ref_line in zip(pred_lines, ref_lines):
                pred_parts = pred_line.split("&")
                ref_parts = ref_line.split("&")
                if len(pred_parts) == len(ref_parts):
                    if not all(
                        [
                            math_equal(
                                pred_parts[i],
                                ref_parts[i],
                                include_percentage,
                                is_close,
                            )
                            for i in range(len(pred_parts))
                        ]
                    ):
                        matched = False
                        break
                else:
                    matched = False
                if not matched:
                    break
        else:
            matched = False
        if matched:
            return True

    # Add back equation comparison
    if prediction.count("=") == 1 and reference.count("=") == 1:
        pred = prediction.split("=")
        pred = f"{pred[0].strip()} - ({pred[1].strip()})"
        ref = reference.split("=")
        ref = f"{ref[0].strip()} - ({ref[1].strip()})"
        if symbolic_equal(pred, ref) or symbolic_equal(f"-({pred})", ref):
            return True
    elif (
        prediction.count("=") == 1
        and len(prediction.split("=")[0].strip()) <= 2
        and "=" not in reference
    ):
        if math_equal(
            prediction.split("=")[1], reference, include_percentage, is_close
        ):
            return True
    elif (
        reference.count("=") == 1
        and len(reference.split("=")[0].strip()) <= 2
        and "=" not in prediction
    ):
        if math_equal(
            prediction, reference.split("=")[1], include_percentage, is_close
        ):
            return True

    # symbolic equal with sympy
    if symbolic_equal(prediction, reference):
        return True

    return False
```
**EN:** `math_equal` is a function that implements the core logic for this scope. The docstring frames it as: Exact match of math if and only if: 1. It returns `False` to the caller. Notable calls include `str(reference).strip`, `str(prediction).strip`, `symbolic_equal`.
**CN:** `math_equal` 是一个函数，用于实现当前作用域的核心逻辑。文档字符串进一步说明了该逻辑的用途与调用语义。它会向调用方返回 `False`。其中较关键的调用包括 `str(reference).strip`, `str(prediction).strip`, `symbolic_equal`。

## Key Concepts / 关键概念
- **Benchmarking / 基准测试**: Measures performance-related signals such as latency, throughput, or speedup. / 衡量延迟、吞吐或加速比等性能信号。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `math`
- **Third-party / 第三方依赖**: `regex`, `sympy`, `sympy.parsing.latex`, `sympy.parsing.sympy_parser`
