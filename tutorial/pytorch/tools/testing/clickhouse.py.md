# clickhouse.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `tools/testing/clickhouse.py`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Provides shared testing infrastructure, test-selection helpers, or execution utilities for PyTorch CI.
- **Purpose (CN)**: 提供 PyTorch CI 使用的共享测试基础设施、测试选择辅助逻辑或执行工具。
## Line-by-Line Analysis / 逐行分析

### Lines 1-5
```python
import json
import os
from functools import lru_cache
from typing import Any
```
- **EN**: This block assembles the Python-side dependencies, importing Python standard-library modules such as json, os, functools, and 1 more. Structured-data parsing keeps repository metadata or intermediate reports in machine-readable form.
- **CN**: 这一段组织 Python 侧依赖，引入了Python 标准库模块，如 json、os、functools 等共 4 项。 结构化数据解析使仓库元数据或中间报告保持机器可读形式。

### Lines 6-9
```python
import clickhouse_connect  # type: ignore[import]


@lru_cache(maxsize=1)
```
- **EN**: This block assembles the Python-side dependencies, importing external packages such as clickhouse_connect  # type: ignore[import].
- **CN**: 这一段组织 Python 侧依赖，引入了外部依赖包，如 clickhouse_connect  # type: ignore[import]。

### Lines 10-17
```python
def get_clickhouse_client() -> Any:
    endpoint = os.environ["CLICKHOUSE_ENDPOINT"]
    # I cannot figure out why these values aren't being handled automatically
    # when it is fine in the lambda
    endpoint = endpoint.removeprefix("https://")
    endpoint = endpoint.removesuffix(":8443")
    return clickhouse_connect.get_client(
        host=endpoint,
```
- **EN**: This chunk defines `get_clickhouse_client`, which orchestrates command-line execution and forwards parsed arguments into the core logic. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段定义了 `get_clickhouse_client`，其作用是编排命令行执行，并把解析后的参数转入核心逻辑。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 18-24
```python
        user=os.environ["CLICKHOUSE_USERNAME"],
        password=os.environ["CLICKHOUSE_PASSWORD"],
        secure=True,
        interface="https",
        port=8443,
    )
```
- **EN**: This chunk continues `get_clickhouse_client` and expands its internal control flow or data movement.
- **CN**: 这一段延续了 `get_clickhouse_client`，进一步展开其内部控制流或数据流转。

### Lines 25-30
```python

def query_clickhouse(query: str, params: dict[str, Any]) -> list[dict[str, Any]]:
    """
    Queries ClickHouse.  Returns datetime in YYYY-MM-DD HH:MM:SS format.
    """
```
- **EN**: This docstring block explains the tool entry point, generation contract, or usage expectations before executable logic begins. This chunk defines `query_clickhouse`, which orchestrates command-line execution and forwards parsed arguments into the core logic.
- **CN**: 这一段文档字符串在可执行逻辑开始前说明了工具入口、生成约定或使用方式。 这一段定义了 `query_clickhouse`，其作用是编排命令行执行，并把解析后的参数转入核心逻辑。

### Lines 31-34
```python
    def convert_to_json_list(res: bytes) -> list[dict[str, Any]]:
        rows = []
        for row in res.decode().split("\n"):
            if row:
```
- **EN**: This chunk defines `convert_to_json_list`, which implements a focused step inside the testing infrastructure pipeline. Structured-data parsing keeps repository metadata or intermediate reports in machine-readable form. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段定义了 `convert_to_json_list`，其作用是实现测试基础设施流水线中的一个关键步骤。 结构化数据解析使仓库元数据或中间报告保持机器可读形式。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 35-39
```python
                rows.append(json.loads(row))
        return rows

    res = get_clickhouse_client().raw_query(query, params, fmt="JSONEachRow")
    return convert_to_json_list(res)
```
- **EN**: This chunk continues `convert_to_json_list` and expands its internal control flow or data movement. Structured-data parsing keeps repository metadata or intermediate reports in machine-readable form. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段延续了 `convert_to_json_list`，进一步展开其内部控制流或数据流转。 结构化数据解析使仓库元数据或中间报告保持机器可读形式。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

## Key Concepts / 关键概念

- **Testing infrastructure**
  - EN: This file belongs to the testing infrastructure layer and should be read in the context of neighboring scripts in the same subtree.
  - CN: 该文件属于测试基础设施层，应结合同一子目录中的相邻脚本一起理解。
- **Type-shape surface**
  - EN: The file models public Python typing information, stubs, or developer-facing signatures.
  - CN: 该文件建模公共 Python 类型信息、类型桩或面向开发者的签名。
- **get_clickhouse_client**
  - EN: `get_clickhouse_client` is one of the main local symbols exposed or implemented here.
  - CN: `get_clickhouse_client` 是此处暴露或实现的主要局部符号之一。
- **query_clickhouse**
  - EN: `query_clickhouse` is one of the main local symbols exposed or implemented here.
  - CN: `query_clickhouse` 是此处暴露或实现的主要局部符号之一。
## Dependencies / 依赖关系

- **Python standard library / Python 标准库**: `json`, `os`, `functools`, `typing`
- **External packages / 外部依赖包**: `clickhouse_connect  # type: ignore[import]`
- **Primary symbols in this file / 本文件核心符号**: `get_clickhouse_client`, `query_clickhouse`
