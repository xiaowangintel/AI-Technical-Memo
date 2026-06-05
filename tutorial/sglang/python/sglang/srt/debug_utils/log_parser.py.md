# log_parser.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/debug_utils/log_parser.py`
- **Repository**: `sgl-project/sglang`
- **Purpose**: This module belongs to SGLang's SRT debug utilities and focuses on shared debugging utilities. It mainly implements the core logic needed by this part of the debug toolchain. / 该模块属于 SGLang 的 SRT 调试工具，聚焦于共享调试工具。它主要用于实现该调试工具链所需的核心逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-16: Declare module-level symbols such as `_PATTERN_DECODE` / 声明模块级符号，例如 `_PATTERN_DECODE`
```python
_PATTERN_DECODE = (
    r"(\(\w+ pid=(?P<pid>\d+)(?:,\s*ip=(?P<ip>[\d\.]+))?\))?\s*"
    r"\[(?P<time>\d{4}-\d{2}-\d{2} \d{2}:\d{2}:\d{2})"
    r"(?:\s+DP(?P<dp_rank>\d+))?"
    r"(?:\s+TP(?P<tp_rank>\d+))?"
    r"(?:\s+EP(?P<ep_rank>\d+))?"
    r"(?:\s+PP(?P<pp_rank>\d+))?"
    r"\]\s+"
    r"Decode batch( \[\d+\])?,\s+"
    r"#running-req:\s*(?P<num_running_req>\d+),\s+"
    r"#token:\s*(?P<num_token>\d+),\s+"
    r"token usage:\s*(?P<token_usage>[0-9.]+),\s+"
    r".*?"
    r"gen throughput \(token/s\):\s*(?P<gen_throughput>[0-9.]+),\s+"
    r"#queue-req:\s*(?P<queue_req>\d+),"
)
```
**EN:** This block defines reusable module-level symbols such as constants, aliases, or configuration values that simplify the later control flow.
**CN:** 这一段定义了可复用的模块级符号，例如常量、别名或配置值，从而简化后续控制流程。

### Lines 19-46: Implement function `parse` / 实现函数 `parse`
```python
def parse(lines):
    import polars as pl

    df = pl.DataFrame(dict(line=lines.splitlines()))
    df = df.with_columns(info=pl.col("line").str.extract_groups(_PATTERN_DECODE))
    df = df.unnest("info")
    df = df.filter(pl.col("gen_throughput").is_not_null())

    df = df.with_columns(
        pl.col("time").str.strptime(pl.Datetime, "%Y-%m-%d %H:%M:%S"),
        *[
            pl.col(col).cast(dtype)
            for col, dtype in [
                ("pid", pl.Int64),
                ("dp_rank", pl.Int64),
                ("tp_rank", pl.Int64),
                ("ep_rank", pl.Int64),
                ("pp_rank", pl.Int64),
                ("num_running_req", pl.Int64),
                ("num_token", pl.Int64),
                ("token_usage", pl.Float64),
                ("gen_throughput", pl.Float64),
                ("queue_req", pl.Int64),
            ]
            if col in df.columns
        ],
    )
    return df
```
**EN:** Function `parse` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps.
**CN:** 函数 `parse` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。

## Key Concepts / 关键概念
- **Core symbols / 核心符号**: `parse`
- **Module role / 模块角色**: Shared debugging utilities / 共享调试工具
- **Implementation focus / 实现重点**: Implements the core logic needed by this part of the debug toolchain / 实现该调试工具链所需的核心逻辑

## Dependencies / 依赖关系
- **Standard library / 标准库**: None / 无
- **Third-party / 第三方**: `polars`
- **Internal / 内部**: None / 无
