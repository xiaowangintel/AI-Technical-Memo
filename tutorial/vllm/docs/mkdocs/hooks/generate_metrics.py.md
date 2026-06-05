# generate_metrics.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `docs/mkdocs/hooks/generate_metrics.py`
- **Repository**: `vllm-project/vllm`
- **Purpose**: Build-time documentation hook or generator for MkDocs / 面向 MkDocs 的构建期文档 hook 或生成器

## Line-by-Line Analysis / 逐行分析
### Module setup
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project
import ast
import logging
from pathlib import Path
from typing import Literal

logger = logging.getLogger("mkdocs")

ROOT_DIR = Path(__file__).parent.parent.parent.parent
DOCS_DIR = ROOT_DIR / "docs"
GENERATED_METRICS_DIR = DOCS_DIR / "generated" / "metrics"

# Files to scan for metric definitions - each will generate a separate table
METRIC_SOURCE_FILES = [
    {"path": "vllm/v1/metrics/loggers.py", "output": "general.inc.md"},
    {
        "path": "vllm/v1/spec_decode/metrics.py",
        "output": "spec_decode.inc.md",
    },
    {
        "path": "vllm/distributed/kv_transfer/kv_connector/v1/nixl/stats.py",
        "output": "nixl_connector.inc.md",
    },
    {"path": "vllm/v1/metrics/perf.py", "output": "perf.inc.md"},
]
```
**EN:** The opening block establishes imports, constants, globals, and module-level context for later hooks or helpers.
**CN:** 开头部分建立导入、常量、全局变量以及后续 hook/辅助函数所需的模块上下文。

### MetricExtractor
```python
class MetricExtractor(ast.NodeVisitor):
    """AST visitor to extract metric definitions."""

    def __init__(self):
        self.metrics: list[dict[str, str]] = []

    def visit_Call(self, node: ast.Call) -> None:
        """Visit function calls to find metric class instantiations."""
        metric_type = self._get_metric_type(node)
        if metric_type:
            name = self._extract_kwarg(node, "name")
            documentation = self._extract_kwarg(node, "documentation")

            if name:
                self.metrics.append(
                    {
                        "name": name,
                        "type": metric_type,
                        "documentation": documentation or "",
                    }
                )

        self.generic_visit(node)

    def _get_metric_type(self, node: ast.Call) -> str | None:
        """Determine if this call creates a metric and return its type."""
        metric_type_map = {
            "_gauge_cls": "gauge",
            "_counter_cls": "counter",
            "_histogram_cls": "histogram",
        }
        if isinstance(node.func, ast.Attribute):
            return metric_type_map.get(node.func.attr)
        return None

    def _extract_kwarg(self, node: ast.Call, key: str) -> str | None:
        """Extract a keyword argument value from a function call."""
        for keyword in node.keywords:
            if keyword.arg == key:
                return self._get_string_value(keyword.value)
        return None

    def _get_string_value(self, node: ast.AST) -> str | None:
        """Extract string value from an AST node."""
        if isinstance(node, ast.Constant):
            return str(node.value) if node.value is not None else None
        return None
```
**EN:** This class pulls structured data out of source text or syntax trees.
**CN:** 该class负责从源码文本或语法树中提取结构化信息。

### extract_metrics_from_file
```python
def extract_metrics_from_file(filepath: Path) -> list[dict[str, str]]:
    """Parse a Python file and extract all metric definitions."""
    try:
        with open(filepath, encoding="utf-8") as f:
            source = f.read()

        tree = ast.parse(source, filename=str(filepath))
        extractor = MetricExtractor()
        extractor.visit(tree)
        return extractor.metrics
    except Exception as e:
        raise RuntimeError(f"Failed to parse {filepath}: {e}") from e
```
**EN:** This function pulls structured data out of source text or syntax trees.
**CN:** 该function负责从源码文本或语法树中提取结构化信息。

### generate_markdown_table
```python
def generate_markdown_table(metrics: list[dict[str, str]]) -> str:
    """Generate a markdown table from extracted metrics."""
    if not metrics:
        return "No metrics found.\n"

    # Sort by type, then by name
    metrics_sorted = sorted(metrics, key=lambda m: (m["type"], m["name"]))

    lines = []
    lines.append("| Metric Name | Type | Description |")
    lines.append("|-------------|------|-------------|")

    for metric in metrics_sorted:
        name = metric["name"]
        metric_type = metric["type"].capitalize()
        doc = metric["documentation"].replace("\n", " ").strip()
        lines.append(f"| `{name}` | {metric_type} | {doc} |")

    return "\n".join(lines) + "\n"
```
**EN:** This function assembles generated documentation or derived output from parsed inputs.
**CN:** 该function会根据解析后的输入组装生成文档或派生输出。

### on_startup
```python
def on_startup(command: Literal["build", "gh-deploy", "serve"], dirty: bool):
    """Generate metrics documentation tables from source files."""
    logger.info("Generating metrics documentation")

    # Create generated directory if it doesn't exist
    GENERATED_METRICS_DIR.mkdir(parents=True, exist_ok=True)

    total_metrics = 0
    for source_config in METRIC_SOURCE_FILES:
        source_path = source_config["path"]
        output_file = source_config["output"]

        filepath = ROOT_DIR / source_path
        if not filepath.exists():
            raise FileNotFoundError(f"Metrics source file not found: {filepath}")

        logger.debug("Extracting metrics from: %s", source_path)
        metrics = extract_metrics_from_file(filepath)
        logger.debug("Found %d metrics in %s", len(metrics), source_path)

        # Generate and write the markdown table for this source
        table_content = generate_markdown_table(metrics)
        output_path = GENERATED_METRICS_DIR / output_file
        with open(output_path, "w", encoding="utf-8") as f:
            f.write(table_content)

        total_metrics += len(metrics)
        logger.info(
            "Generated metrics table: %s (%d metrics)",
            output_path.relative_to(ROOT_DIR),
            len(metrics),
        )

    logger.info(
        "Total metrics generated: %d across %d files",
        total_metrics,
        len(METRIC_SOURCE_FILES),
    )
```
**EN:** This function is a MkDocs hook entrypoint that is executed during build/render events.
**CN:** 该function是 MkDocs 的钩子入口，会在构建或渲染事件中执行。

## Key Concepts / 关键概念
- **EN:** The file is build-time tooling: it shapes generated docs or page rendering rather than model execution.
  **CN:** 该文件属于构建期工具：它影响生成文档或页面渲染，而不是模型执行本身。
- **EN:** MkDocs hook functions are the entrypoints that connect the module to the documentation build lifecycle.
  **CN:** MkDocs hook 函数是把该模块接入文档构建生命周期的入口。
- **EN:** Classes in the file encapsulate reusable parsing, formatting, or preprocessing behavior.
  **CN:** 文件中的类封装了可复用的解析、格式化或预处理行为。
- **EN:** Generation/extraction helpers turn source material into derived Markdown content.
  **CN:** 生成/提取辅助函数会把源码材料转换成派生的 Markdown 内容。

## Dependencies / 依赖关系
- **EN:** `ast` is imported directly and participates in the hook or generation flow.
  **CN:** `ast` 被直接导入，并参与该 hook 或生成流程。
- **EN:** `logging` is imported directly and participates in the hook or generation flow.
  **CN:** `logging` 被直接导入，并参与该 hook 或生成流程。
- **EN:** `pathlib` is imported directly and participates in the hook or generation flow.
  **CN:** `pathlib` 被直接导入，并参与该 hook 或生成流程。
- **EN:** `typing` is imported directly and participates in the hook or generation flow.
  **CN:** `typing` 被直接导入，并参与该 hook 或生成流程。
