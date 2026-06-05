# upload_test_stats_running_jobs.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `tools/stats/upload_test_stats_running_jobs.py`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Implements statistics collection and reporting helpers for build, CI, or repository health signals.
- **Purpose (CN)**: 实现统计采集与报告辅助逻辑，用于构建、CI 或仓库健康度信号。
## Line-by-Line Analysis / 逐行分析

### Lines 1-5
```python
import sys
import time
from functools import cache
from typing import Any
```
- **EN**: This block assembles the Python-side dependencies, importing Python standard-library modules such as sys, time, functools, and 1 more.
- **CN**: 这一段组织 Python 侧依赖，引入了Python 标准库模块，如 sys、time、functools 等共 4 项。

### Lines 6-9
```python
from tools.stats.test_dashboard import upload_additional_info
from tools.stats.upload_stats_lib import get_s3_resource
```
- **EN**: This block assembles the Python-side dependencies, importing internal PyTorch modules such as tools.stats.test_dashboard, tools.stats.upload_stats_lib.
- **CN**: 这一段组织 Python 侧依赖，引入了内部 PyTorch 模块，如 tools.stats.test_dashboard、tools.stats.upload_stats_lib。

### Lines 10-13
```python
BUCKET_PREFIX = "workflows_failing_pending_upload"


@cache
```
- **EN**: Configuration constants such as BUCKET_PREFIX centralize defaults so later functions share the same policy knobs.
- **CN**: BUCKET_PREFIX 等配置常量集中定义默认值，使后续函数共享同一套策略开关。

### Lines 14-17
```python
def get_bucket() -> Any:
    return get_s3_resource().Bucket("gha-artifacts")
```
- **EN**: This chunk defines `get_bucket`, which implements a focused step inside the ci statistics pipeline. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段定义了 `get_bucket`，其作用是实现CI 统计流水线中的一个关键步骤。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 18-25
```python
def delete_obj(key: str) -> None:
    # Does not raise error if key does not exist
    get_bucket().delete_objects(
        Delete={
            "Objects": [{"Key": key}],
            "Quiet": True,
        }
    )
```
- **EN**: This chunk defines `delete_obj`, which implements a focused step inside the ci statistics pipeline. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. Assertions and explicit failures stop invalid states from propagating deeper into the workflow.
- **CN**: 这一段定义了 `delete_obj`，其作用是实现CI 统计流水线中的一个关键步骤。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 断言与显式失败逻辑可阻止非法状态继续向工作流更深层传播。

### Lines 26-33
```python


def put_object(key: str) -> None:
    get_bucket().put_object(
        Key=key,
        Body=b"",
    )
```
- **EN**: This chunk defines `put_object`, which implements a focused step inside the ci statistics pipeline.
- **CN**: 这一段定义了 `put_object`，其作用是实现CI 统计流水线中的一个关键步骤。

### Lines 34-40
```python

def do_upload(workflow_id: int) -> None:
    workflow_attempt = 1
    # Flush stdout so that any errors in upload show up last in the logs.
    sys.stdout.flush()
    upload_additional_info(workflow_id, workflow_attempt)
```
- **EN**: This chunk defines `do_upload`, which parses or loads structured input into tool-friendly data structures.
- **CN**: 这一段定义了 `do_upload`，其作用是把结构化输入解析或加载为工具可处理的数据结构。

### Lines 41-46
```python

def get_workflow_ids(pending: bool = False) -> list[int]:
    prefix = f"{BUCKET_PREFIX}/{'pending/' if pending else ''}"
    objs = get_bucket().objects.filter(Prefix=prefix)
    return [int(obj.key.split("/")[-1].split(".")[0]) for obj in objs]
```
- **EN**: This chunk defines `get_workflow_ids`, which implements a focused step inside the ci statistics pipeline. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段定义了 `get_workflow_ids`，其作用是实现CI 统计流水线中的一个关键步骤。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 47-51
```python

def read_s3(pending: bool = False) -> None:
    while True:
        workflows = get_workflow_ids(pending)
        if not workflows:
```
- **EN**: This chunk defines `read_s3`, which parses or loads structured input into tool-friendly data structures. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段定义了 `read_s3`，其作用是把结构化输入解析或加载为工具可处理的数据结构。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 52-57
```python
            if pending:
                break
            # Wait for more stuff to show up
            print("Sleeping for 60 seconds")
            time.sleep(60)
        for workflow_id in workflows:
```
- **EN**: This chunk continues `read_s3` and expands its internal control flow or data movement. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段延续了 `read_s3`，进一步展开其内部控制流或数据流转。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 58-61
```python
            print(f"Processing {workflow_id}")
            put_object(f"{BUCKET_PREFIX}/pending/{workflow_id}.txt")
            delete_obj(f"{BUCKET_PREFIX}/{workflow_id}.txt")
            try:
```
- **EN**: This chunk continues `read_s3` and expands its internal control flow or data movement. Exception-handling paths convert tool failures into clearer control flow or user-facing diagnostics.
- **CN**: 这一段延续了 `read_s3`，进一步展开其内部控制流或数据流转。 异常处理路径把工具失败转换为更清晰的控制流或面向用户的诊断信息。

### Lines 62-66
```python
                do_upload(workflow_id)
            except Exception as e:
                print(f"Failed to upload {workflow_id}: {e}")
            delete_obj(f"{BUCKET_PREFIX}/pending/{workflow_id}.txt")
```
- **EN**: This chunk continues `read_s3` and expands its internal control flow or data movement. Exception-handling paths convert tool failures into clearer control flow or user-facing diagnostics.
- **CN**: 这一段延续了 `read_s3`，进一步展开其内部控制流或数据流转。 异常处理路径把工具失败转换为更清晰的控制流或面向用户的诊断信息。

### Lines 67-72
```python

if __name__ == "__main__":
    # Workflows in the pending folder were previously in progress of uploading
    # but failed to complete, so we need to retry them.
    read_s3(pending=True)
    read_s3()
```
- **EN**: This chunk continues `read_s3` and expands its internal control flow or data movement. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段延续了 `read_s3`，进一步展开其内部控制流或数据流转。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

## Key Concepts / 关键概念

- **CI statistics**
  - EN: This file belongs to the ci statistics layer and should be read in the context of neighboring scripts in the same subtree.
  - CN: 该文件属于CI 统计层，应结合同一子目录中的相邻脚本一起理解。
- **Statistics reporting**
  - EN: The implementation aggregates measurements and turns them into summaries or dashboards.
  - CN: 该实现聚合度量数据，并将其转化为摘要或仪表板。
- **Type-shape surface**
  - EN: The file models public Python typing information, stubs, or developer-facing signatures.
  - CN: 该文件建模公共 Python 类型信息、类型桩或面向开发者的签名。
- **BUCKET_PREFIX**
  - EN: `BUCKET_PREFIX` is one of the main local symbols exposed or implemented here.
  - CN: `BUCKET_PREFIX` 是此处暴露或实现的主要局部符号之一。
- **get_bucket**
  - EN: `get_bucket` is one of the main local symbols exposed or implemented here.
  - CN: `get_bucket` 是此处暴露或实现的主要局部符号之一。
## Dependencies / 依赖关系

- **Internal PyTorch Python modules / 内部 PyTorch Python 模块**: `tools.stats.test_dashboard`, `tools.stats.upload_stats_lib`
- **Python standard library / Python 标准库**: `sys`, `time`, `functools`, `typing`
- **Primary symbols in this file / 本文件核心符号**: `BUCKET_PREFIX`, `get_bucket`, `delete_obj`, `put_object`, `do_upload`, `get_workflow_ids`, `read_s3`
