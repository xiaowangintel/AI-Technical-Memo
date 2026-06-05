# answer_extraction.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `benchmark/reasoning_benchmark/answer_extraction.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This Python module focuses on reasoning answer extraction. It primarily loads, filters, or serializes benchmark datasets and computes evaluation scores and aggregate statistics. / 该 Python 模块聚焦于 reasoning answer extraction 相关流程。它主要用于加载、筛选或序列化基准测试数据集、计算评测分数与聚合统计结果。

## Line-by-Line Analysis / 逐行分析
### Lines 3-5: Imports and setup / 导入与初始化
```python
import re

import regex
```
**EN:** This block imports required libraries and establishes module-level constants or helpers used later in the file.
**CN:** 该代码块导入所需库，并建立后续逻辑会使用到的模块级常量或辅助配置。

### Lines 8-37: Function `_fix_fracs` / 函数 `_fix_fracs`
```python
def _fix_fracs(string):
    substrs = string.split("\\frac")
    new_str = substrs[0]
    if len(substrs) > 1:
        substrs = substrs[1:]
        for substr in substrs:
            new_str += "\\frac"
            if len(substr) > 0 and substr[0] == "{":
                new_str += substr
            else:
                try:
                    assert len(substr) >= 2
                except:
                    return string
                a = substr[0]
                b = substr[1]
                if b != "{":
                    if len(substr) > 2:
                        post_substr = substr[2:]
                        new_str += "{" + a + "}{" + b + "}" + post_substr
                    else:
                        new_str += "{" + a + "}{" + b + "}"
                else:
                    if len(substr) > 2:
                        post_substr = substr[2:]
                        new_str += "{" + a + "}" + b + post_substr
                    else:
                        new_str += "{" + a + "}" + b
    string = new_str
    return string
```
**EN:** `_fix_fracs` is a function that implements the core logic for this scope. It returns `string` to the caller. Notable calls include `string.split`, `len`.
**CN:** `_fix_fracs` 是一个函数，用于实现当前作用域的核心逻辑。它会向调用方返回 `string`。其中较关键的调用包括 `string.split`, `len`。

### Lines 40-54: Function `_fix_a_slash_b` / 函数 `_fix_a_slash_b`
```python
def _fix_a_slash_b(string):
    if len(string.split("/")) != 2:
        return string
    a = string.split("/")[0]
    b = string.split("/")[1]
    try:
        if "sqrt" not in a:
            a = int(a)
        if "sqrt" not in b:
            b = int(b)
        assert string == "{}/{}".format(a, b)
        new_string = "\\frac{" + str(a) + "}{" + str(b) + "}"
        return new_string
    except:
        return string
```
**EN:** `_fix_a_slash_b` is a function that implements the core logic for this scope. It returns `string` to the caller. Notable calls include `len`, `string.split`, `int`.
**CN:** `_fix_a_slash_b` 是一个函数，用于实现当前作用域的核心逻辑。它会向调用方返回 `string`。其中较关键的调用包括 `len`, `string.split`, `int`。

### Lines 57-60: Function `_fix_sqrt` / 函数 `_fix_sqrt`
```python
def _fix_sqrt(string):
    _string = re.sub(r"\\sqrt(-?[0-9.a-zA-Z]+)", r"\\sqrt{\1}", string)
    _string = re.sub(r"\\sqrt\s+(\w+)$", r"\\sqrt{\1}", _string)
    return _string
```
**EN:** `_fix_sqrt` is a function that implements the core logic for this scope. It returns `_string` to the caller. Notable calls include `re.sub`.
**CN:** `_fix_sqrt` 是一个函数，用于实现当前作用域的核心逻辑。它会向调用方返回 `_string`。其中较关键的调用包括 `re.sub`。

### Lines 63-66: Function `_fix_tan` / 函数 `_fix_tan`
```python
def _fix_tan(string):
    _string = re.sub(r"\\tan(-?[0-9.a-zA-Z]+)", r"\\tan{\1}", string)
    _string = re.sub(r"\\tan\s+(\w+)$", r"\\tan{\1}", _string)
    return _string
```
**EN:** `_fix_tan` is a function that implements the core logic for this scope. It returns `_string` to the caller. Notable calls include `re.sub`.
**CN:** `_fix_tan` 是一个函数，用于实现当前作用域的核心逻辑。它会向调用方返回 `_string`。其中较关键的调用包括 `re.sub`。

### Lines 69-179: Function `strip_string` / 函数 `strip_string`
```python
def strip_string(string):
    string = str(string).strip()
    # linebreaks
    string = string.replace("\n", "")

    # right "."
    string = string.rstrip(".")

    # remove inverse spaces
    string = string.replace("\\!", "")
    # string = string.replace("\\ ", "")

    # replace \\ with \
    # string = string.replace("\\\\", "\\")
    # string = string.replace("\\\\", "\\")

    if string.startswith("\\text{") and string.endswith("}"):
        string = string.split("{", 1)[1][:-1]

    # replace tfrac and dfrac with frac
    string = string.replace("tfrac", "frac")
    string = string.replace("dfrac", "frac")
    string = string.replace("cfrac", "frac")

    # remove \left and \right
    string = string.replace("\\left", "")
    string = string.replace("\\right", "")

    # Remove unit: miles, dollars if after is not none
    _string = re.sub(r"\\text{.*?}$", "", string).strip()
    if _string != "" and _string != string:
        # print("Warning: unit not removed: '{}' -> '{}'".format(string, _string))
        string = _string

    # Remove circ (degrees)
    string = string.replace("^{\\circ}", "").strip()
    string = string.replace("^\\circ", "").strip()

    string = regex.sub(r"\{(c|m)?m\}(\^(2|3))?", "", string).strip()
    string = regex.sub(r"p\.m\.$", "", string).strip()
    string = regex.sub(r"(\d)\s*t$", r"\1", string).strip()

    # remove dollar signs
    string = string.replace("\\$", "")
    string = string.replace("$", "")

    # string = string.replace("\\text", "")
    string = string.replace("x\\in", "")

    # remove percentage
    string = string.replace("\\%", "%")
    string = string.replace("\%", "%")
    # string = string.replace("%", "")

    # " 0." equivalent to " ." and "{0." equivalent to "{." Alternatively, add "0" if "." is the start of the string
    string = string.replace(" .", " 0.")
    string = string.replace("{.", "{0.")

    # cdot
    string = string.replace("\\cdot", "")

    # inf
    string = string.replace("infinity", "\\infty")
    if "\\infty" not in string:
        string = string.replace("inf", "\\infty")
    string = string.replace("+\\inity", "\\infty")

    # and
    # string = string.replace("and", "")
    string = string.replace("\\mathbf", "")
    string = string.replace("\\mathrm", "")

    # use regex to remove \mbox{...}
    string = re.sub(r"\\mbox{.*?}", "", string)

    # quote
    string.replace("'", "")
    string.replace('"', "")

    # i, j
    if "j" in string and "i" not in string:
        string = string.replace("j", "i")

    # replace a.000b where b is not number or b is end, with ab, use regex
    string = re.sub(r"(\d+)\.0+([^\d])", r"\1\2", string)
    string = re.sub(r"(\d+)\.0+$", r"\1", string)

    # if empty, return empty string
    if len(string) == 0:
        return string
    if string[0] == ".":
        string = "0" + string

    # to consider: get rid of e.g. "k = " or "q = " at beginning
    # if len(string.split("=")) == 2:
    #     if len(string.split("=")[0]) <= 2:
    #         string = string.split("=")[1]

    string = _fix_sqrt(string)
    string = _fix_tan(string)
    string = string.replace(" ", "")

    # \frac1b or \frac12 --> \frac{1}{b} and \frac{1}{2}, etc. Even works with \frac1{72} (but not \frac{72}1). Also does a/b --> \\frac{a}{b}
    string = _fix_fracs(string)

    # NOTE: X/Y changed to \frac{X}{Y} in dataset, but in simple cases fix in case the model output is X/Y
    string = _fix_a_slash_b(string)

    string = regex.sub(r"(\\|,|\.)+$", "", string)

    return string
```
**EN:** `strip_string` is a function that loads, filters, or serializes benchmark datasets. It returns `string` to the caller. Notable calls include `str(string).strip`, `string.replace`, `string.rstrip`.
**CN:** `strip_string` 是一个函数，用于加载、筛选或序列化基准测试数据集。它会向调用方返回 `string`。其中较关键的调用包括 `str(string).strip`, `string.replace`, `string.rstrip`。

### Lines 182-197: Function `extract_boxed_answers` / 函数 `extract_boxed_answers`
```python
def extract_boxed_answers(text):
    answers = []
    for piece in text.split("boxed{")[1:]:
        n = 0
        for i in range(len(piece)):
            if piece[i] == "{":
                n += 1
            elif piece[i] == "}":
                n -= 1
                if n < 0:
                    if i + 1 < len(piece) and piece[i + 1] == "%":
                        answers.append(piece[: i + 1])
                    else:
                        answers.append(piece[:i])
                    break
    return answers
```
**EN:** `extract_boxed_answers` is a function that computes evaluation scores and aggregate statistics. It returns `answers` to the caller. Notable calls include `text.split`, `range`, `len`.
**CN:** `extract_boxed_answers` 是一个函数，用于计算评测分数与聚合统计结果。它会向调用方返回 `answers`。其中较关键的调用包括 `text.split`, `range`, `len`。

### Lines 200-211: Function `extract_program_output` / 函数 `extract_program_output`
```python
def extract_program_output(pred_str):
    """
    extract output between the last ```output\n...\n```
    """
    if "```output" not in pred_str:
        return ""
    if "```output" in pred_str:
        pred_str = pred_str.split("```output")[-1]
    if "```" in pred_str:
        pred_str = pred_str.split("```")[0]
    output = pred_str.strip()
    return output
```
**EN:** `extract_program_output` is a function that implements the core logic for this scope. The docstring frames it as: extract output between the last ```output ... It returns `output` to the caller. Notable calls include `pred_str.strip`, `pred_str.split`.
**CN:** `extract_program_output` 是一个函数，用于实现当前作用域的核心逻辑。文档字符串进一步说明了该逻辑的用途与调用语义。它会向调用方返回 `output`。其中较关键的调用包括 `pred_str.strip`, `pred_str.split`。

### Lines 214-250: Function `extract_answer` / 函数 `extract_answer`
```python
def extract_answer(pred_str, exhaust=False):
    pred = []
    if "final answer is $" in pred_str and "$. I hope" in pred_str:
        tmp = pred_str.split("final answer is $", 1)[1]
        pred = [tmp.split("$. I hope", 1)[0].strip()]
    elif "boxed" in pred_str:
        pred = extract_boxed_answers(pred_str)
    elif "he answer is" in pred_str:
        pred = [pred_str.split("he answer is")[-1].strip()]
    else:
        program_output = extract_program_output(pred_str)
        if program_output != "":
            # fall back to program
            pred.append(program_output)
        else:  # use the last number
            pattern = "-?\d*\.?\d+"
            answers = re.findall(pattern, pred_str.replace(",", ""))
            if len(answers) >= 1:
                last_ans = answers[-1]
            else:
                last_ans = ""
            if last_ans:
                pred.append(last_ans)

    # multiple line
    _pred = []
    for each_ans in pred:
        each_ans = each_ans.strip().split("\n")[0]
        each_ans = each_ans.lstrip(":")
        each_ans = each_ans.rstrip(".")
        each_ans = each_ans.rstrip("/")
        each_ans = strip_string(each_ans)
        _pred.append(each_ans)
    if exhaust:
        return _pred
    else:
        return _pred[-1] if _pred else ""
```
**EN:** `extract_answer` is a function that computes evaluation scores and aggregate statistics. It returns `_pred` to the caller. Notable calls include `each_ans.lstrip`, `each_ans.rstrip`, `strip_string`.
**CN:** `extract_answer` 是一个函数，用于计算评测分数与聚合统计结果。它会向调用方返回 `_pred`。其中较关键的调用包括 `each_ans.lstrip`, `each_ans.rstrip`, `strip_string`。

### Lines 253-269: Function `extract_math_answer` / 函数 `extract_math_answer`
```python
def extract_math_answer(question, reasoning, task):
    answer = []
    for ans in extract_answer(reasoning, exhaust=True):
        if "separated by commas" in question and all(ch not in ans for ch in "()[]"):
            answer.extend([a.strip() for a in ans.split(",")])
        elif regex.search(r"\\text\{\s*and\s*\}", ans):
            answer.extend(
                [
                    a.strip()
                    for a in regex.sub(r"\\text\{\s*and\s*\}", "[SEP]", ans).split(
                        "[SEP]"
                    )
                ]
            )
        else:
            answer.append(ans.strip())
    return answer
```
**EN:** `extract_math_answer` is a function that computes evaluation scores and aggregate statistics. It returns `answer` to the caller. Notable calls include `extract_answer`, `all`, `answer.extend`.
**CN:** `extract_math_answer` 是一个函数，用于计算评测分数与聚合统计结果。它会向调用方返回 `answer`。其中较关键的调用包括 `extract_answer`, `all`, `answer.extend`。

## Key Concepts / 关键概念
- **Benchmarking / 基准测试**: Measures performance-related signals such as latency, throughput, or speedup. / 衡量延迟、吞吐或加速比等性能信号。
- **Datasets / 数据集**: Loads or reshapes benchmark inputs from structured files. / 从结构化文件中加载或整理基准测试输入。
- **Metrics / 指标**: Aggregates quality signals in addition to raw runtime. / 除运行时之外还汇总质量指标。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `re`
- **Third-party / 第三方依赖**: `regex`
