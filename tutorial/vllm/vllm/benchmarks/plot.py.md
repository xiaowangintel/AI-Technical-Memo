# plot.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/benchmarks/plot.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Generate plots for benchmark results. / 该文件的核心目的为：Generate plots for benchmark results.

## Line-by-Line Analysis / 逐行分析
### Module docstring (lines 3-3)
```python
"""Generate plots for benchmark results."""
```
**EN:** This opening docstring states the module's intent, expected usage, and any important assumptions before the implementation details begin.
**CN:** 开头的文档字符串先说明模块意图、预期用法以及实现展开前需要了解的重要假设。

### Imports (lines 5-8)
```python
from pathlib import Path

from typing import Any

from vllm.utils.import_utils import PlaceholderModule
```
**EN:** This import block pulls in standard-library helpers, third-party packages, and vLLM internals used later in the file. It establishes the building blocks for the remaining logic.
**CN:** 这一组导入语句引入了标准库辅助工具、第三方依赖以及后续要使用的 vLLM 内部模块，为剩余逻辑建立基础。

### Function `generate_timeline_plot` (lines 25-113)
```python
def generate_timeline_plot(
    results: list[dict[str, Any]],
    output_path: Path,
    colors: list[str] | None = None,
    itl_thresholds: list[float] | None = None,
    labels: list[str] | None = None,
) -> None:
    """
    Generate an HTML timeline plot from benchmark results.

    Args:
        results: List of per-request result dictionaries containing:
            - start_time: Request start time (seconds)
            - ttft: Time to first token (seconds)
            - itl: List of inter-token latencies (seconds)
            - latency: Total request latency (seconds)
            - prompt_len: Number of prompt tokens
            - output_tokens: Number of output tokens
        output_path: Path where the HTML file will be saved
        colors: List of colors for ITL categories (default: green, orange, red, black)
        itl_thresholds: ITL thresholds in seconds (default: [1.0, 4.0, 6.0])
        labels: Labels for ITL categories (default based on thresholds)
    """

    # Set defaults
    # ... omitted for brevity ...
    pio.write_html(fig, str(output_path))
    print(f"Timeline plot saved to: {output_path}")
```
**EN:** Function `generate_timeline_plot` provides a reusable helper around the module's main workflow. The docstring highlights: Generate an HTML timeline plot from benchmark results. Key calls such as `dict`, `zip`, `construct_timeline_data`, `print`, `px.timeline` show the concrete execution path.
**CN:** Function `generate_timeline_plot` 为模块主流程提供可复用的辅助逻辑。 文档字符串强调：Generate an HTML timeline plot from benchmark results. 像 `dict`, `zip`, `construct_timeline_data`, `print`, `px.timeline` 这样的关键调用展示了该代码块的具体执行路径。

### Function `construct_timeline_data` (lines 116-225)
```python
def construct_timeline_data(
    requests_data: list[dict[str, Any]],
    itl_thresholds: list[float],
    labels: list[str],
) -> list[dict[str, Any]]:
    """
    Construct timeline data from request results.

    Args:
        requests_data: List of per-request result dictionaries
        itl_thresholds: ITL thresholds in seconds
        labels: Labels for ITL categories

    Returns:
        List of timeline segments for plotting
    """

    def tostr(sec_time: float) -> str:
        """Convert seconds to HH:MM:SS.mmm format."""
        h = int(sec_time // 3600)
        assert h < 100, "time seems to last more than 100 hours"
        m = int((sec_time % 3600) // 60)
        s = sec_time % 60
        return f"{h:02d}:{m:02d}:{s:06.3f}"

    # ... omitted for brevity ...

    return timeline_data
```
**EN:** Function `construct_timeline_data` provides a reusable helper around the module's main workflow. The docstring highlights: Construct timeline data from request results. Key calls such as `int`, `request.get`, `enumerate`, `tostr`, `timeline_data.append` show the concrete execution path.
**CN:** Function `construct_timeline_data` 为模块主流程提供可复用的辅助逻辑。 文档字符串强调：Construct timeline data from request results. 像 `int`, `request.get`, `enumerate`, `tostr`, `timeline_data.append` 这样的关键调用展示了该代码块的具体执行路径。

### Function `generate_dataset_stats_plot` (lines 228-316)
```python
def generate_dataset_stats_plot(
    results: list[dict[str, Any]],
    output_path: Path,
) -> None:
    """
    Generate a matplotlib figure with dataset statistics.

    Creates a figure with 4 subplots:
    - Top-left: Prompt tokens distribution (histogram)
    - Top-right: Output tokens distribution (histogram)
    - Bottom-left: Prompt+output tokens distribution (histogram)
    - Bottom-right: Stacked bar chart (request_id vs tokens)

    Args:
        results: List of per-request result dictionaries containing:
            - prompt_len: Number of prompt tokens
            - output_tokens: Number of output tokens
        output_path: Path where the figure will be saved
    """
    # Extract data
    prompt_tokens = []
    output_tokens = []
    total_tokens = []

    for request in results:
    # ... omitted for brevity ...

    print(f"Dataset statistics plot saved to: {output_path}")
```
**EN:** Function `generate_dataset_stats_plot` provides a reusable helper around the module's main workflow. The docstring highlights: Generate a matplotlib figure with dataset statistics. Key calls such as `request.get`, `prompt_tokens.append`, `output_tokens.append`, `total_tokens.append`, `print` show the concrete execution path.
**CN:** Function `generate_dataset_stats_plot` 为模块主流程提供可复用的辅助逻辑。 文档字符串强调：Generate a matplotlib figure with dataset statistics. 像 `request.get`, `prompt_tokens.append`, `output_tokens.append`, `total_tokens.append`, `print` 这样的关键调用展示了该代码块的具体执行路径。

## Key Concepts / 关键概念
- **Benchmarking/data prep / 基准测试/数据准备**
  - **EN:** The file includes utilities for preparing benchmark inputs or measuring runtime behavior.
  - **CN:** 该文件包含用于准备基准测试输入或测量运行时行为的工具。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `from pathlib import Path`, `from typing import Any`
- **vLLM internal / vLLM 内部依赖**: `from vllm.utils.import_utils import PlaceholderModule`
