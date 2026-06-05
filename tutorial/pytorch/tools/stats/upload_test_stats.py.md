# upload_test_stats.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `tools/stats/upload_test_stats.py`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Implements statistics collection and reporting helpers for build, CI, or repository health signals.
- **Purpose (CN)**: 实现统计采集与报告辅助逻辑，用于构建、CI 或仓库健康度信号。
## Line-by-Line Analysis / 逐行分析

### Lines 1-12
```python
from __future__ import annotations

import argparse
import os
import re
import sys
import xml.etree.ElementTree as ET
from multiprocessing import cpu_count, Pool
from pathlib import Path
from tempfile import TemporaryDirectory
from typing import Any
```
- **EN**: This block assembles the Python-side dependencies, importing Python standard-library modules such as __future__, argparse, os, and 7 more. Argument-parsing logic turns command-line inputs into structured parameters for the tool core. Filesystem handling code normalizes paths and keeps generated or discovered artifacts in predictable locations.
- **CN**: 这一段组织 Python 侧依赖，引入了Python 标准库模块，如 __future__、argparse、os 等共 10 项。 参数解析逻辑把命令行输入转换为工具核心可以直接消费的结构化参数。 文件系统处理逻辑负责规范化路径，并把生成或发现的产物放在可预测的位置。

### Lines 13-22
```python
from tools.stats.test_dashboard import upload_additional_info
from tools.stats.upload_stats_lib import (
    download_s3_artifacts,
    get_job_id,
    remove_nan_inf,
    unzip,
    upload_to_s3,
    upload_workflow_stats_to_s3,
)
```
- **EN**: This block assembles the Python-side dependencies, importing internal PyTorch modules such as tools.stats.test_dashboard, tools.stats.upload_stats_lib.
- **CN**: 这一段组织 Python 侧依赖，引入了内部 PyTorch 模块，如 tools.stats.test_dashboard、tools.stats.upload_stats_lib。

### Lines 23-35
```python

def should_upload_full_test_run(head_branch: str | None, head_repository: str) -> bool:
    """Return True if we should upload the full test_run dataset.

    Rules:
    - Only for the main repository (pytorch/pytorch)
    - If head_branch is 'main', or a tag of form 'trunk/{40-hex-sha}'
    """
    is_trunk_tag = bool(re.fullmatch(r"trunk/[0-9a-fA-F]{40}", (head_branch or "")))
    return head_repository == "pytorch/pytorch" and (
        head_branch == "main" or is_trunk_tag
    )
```
- **EN**: This docstring block explains the tool entry point, generation contract, or usage expectations before executable logic begins. This chunk defines `should_upload_full_test_run`, which parses or loads structured input into tool-friendly data structures. Subprocess orchestration bridges this script to external build tools, linters, or system commands. Pattern-matching logic extracts structured facts from loosely formatted text inputs. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段文档字符串在可执行逻辑开始前说明了工具入口、生成约定或使用方式。 这一段定义了 `should_upload_full_test_run`，其作用是把结构化输入解析或加载为工具可处理的数据结构。 子进程编排把该脚本连接到外部构建工具、lint 或系统命令。 模式匹配逻辑从格式较松散的文本输入中提取结构化事实。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 36-46
```python

def parse_xml_report(
    tag: str,
    report: Path,
    workflow_id: int,
    workflow_run_attempt: int,
    job_id: int | None = None,
) -> list[dict[str, Any]]:
    """Convert a test report xml file into a JSON-serializable list of test cases."""
    print(f"Parsing {tag}s for test report: {report}")
```
- **EN**: This docstring block explains the tool entry point, generation contract, or usage expectations before executable logic begins. This chunk defines `parse_xml_report`, which parses or loads structured input into tool-friendly data structures. Structured-data parsing keeps repository metadata or intermediate reports in machine-readable form. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently.
- **CN**: 这一段文档字符串在可执行逻辑开始前说明了工具入口、生成约定或使用方式。 这一段定义了 `parse_xml_report`，其作用是把结构化输入解析或加载为工具可处理的数据结构。 结构化数据解析使仓库元数据或中间报告保持机器可读形式。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。

### Lines 47-59
```python
    if job_id is None:
        job_id = get_job_id(report)
        print(f"Found job id: {job_id}")

    test_cases: list[dict[str, Any]] = []

    root = ET.parse(report)
    for test_case in root.iter(tag):
        case = process_xml_element(test_case)
        case["workflow_id"] = workflow_id
        case["workflow_run_attempt"] = workflow_run_attempt
        case["job_id"] = job_id
```
- **EN**: This chunk continues `parse_xml_report` and expands its internal control flow or data movement. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段延续了 `parse_xml_report`，进一步展开其内部控制流或数据流转。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 60-72
```python
        # [invoking file]
        # The name of the file that the test is located in is not necessarily
        # the same as the name of the file that invoked the test.
        # For example, `test_jit.py` calls into multiple other test files (e.g.
        # jit/test_dce.py). For sharding/test selection purposes, we want to
        # record the file that invoked the test.
        #
        # To do this, we leverage an implementation detail of how we write out
        # tests (https://bit.ly/3ajEV1M), which is that reports are created
        # under a folder with the same name as the invoking file.
        case["invoking_file"] = report.parent.name
        test_cases.append(case)
```
- **EN**: This chunk continues `parse_xml_report` and expands its internal control flow or data movement.
- **CN**: 这一段延续了 `parse_xml_report`，进一步展开其内部控制流或数据流转。

### Lines 73-81
```python
    return test_cases


def process_xml_element(
    element: ET.Element, output_numbers: bool = True
) -> dict[str, Any]:
    """Convert a test suite element into a JSON-serializable dict."""
    ret: dict[str, Any] = {}
```
- **EN**: This docstring block explains the tool entry point, generation contract, or usage expectations before executable logic begins. This chunk defines `process_xml_element`, which implements a focused step inside the ci statistics pipeline. Structured-data parsing keeps repository metadata or intermediate reports in machine-readable form. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段文档字符串在可执行逻辑开始前说明了工具入口、生成约定或使用方式。 这一段定义了 `process_xml_element`，其作用是实现CI 统计流水线中的一个关键步骤。 结构化数据解析使仓库元数据或中间报告保持机器可读形式。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 82-91
```python
    # Convert attributes directly into dict elements.
    # e.g.
    #     <testcase name="test_foo" classname="test_bar"></testcase>
    # becomes:
    #     {"name": "test_foo", "classname": "test_bar"}
    ret.update(element.attrib)

    # The XML format encodes all values as strings. Convert to ints/floats if
    # possible to make aggregation possible in SQL.
    if output_numbers:
```
- **EN**: This chunk continues `process_xml_element` and expands its internal control flow or data movement. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段延续了 `process_xml_element`，进一步展开其内部控制流或数据流转。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 92-100
```python
        for k, v in ret.items():
            try:
                ret[k] = int(v)
            except ValueError:
                try:
                    ret[k] = float(v)
                except ValueError:
                    pass
```
- **EN**: This chunk continues `process_xml_element` and expands its internal control flow or data movement. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Exception-handling paths convert tool failures into clearer control flow or user-facing diagnostics.
- **CN**: 这一段延续了 `process_xml_element`，进一步展开其内部控制流或数据流转。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 异常处理路径把工具失败转换为更清晰的控制流或面向用户的诊断信息。

### Lines 101-110
```python
    # Convert inner and outer text into special dict elements.
    # e.g.
    #     <testcase>my_inner_text</testcase> my_tail
    # becomes:
    #     {"text": "my_inner_text", "tail": " my_tail"}
    if element.text and element.text.strip():
        ret["text"] = element.text
    if element.tail and element.tail.strip():
        ret["tail"] = element.tail
```
- **EN**: This chunk continues `process_xml_element` and expands its internal control flow or data movement. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段延续了 `process_xml_element`，进一步展开其内部控制流或数据流转。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 111-123
```python
    # Convert child elements recursively, placing them at a key:
    # e.g.
    #     <testcase>
    #       <foo>hello</foo>
    #       <foo>world</foo>
    #       <bar>another</bar>
    #     </testcase>
    # becomes
    #    {
    #       "foo": [{"text": "hello"}, {"text": "world"}],
    #       "bar": {"text": "another"}
    #    }
    for child in element:
```
- **EN**: This chunk continues `process_xml_element` and expands its internal control flow or data movement. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently.
- **CN**: 这一段延续了 `process_xml_element`，进一步展开其内部控制流或数据流转。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。

### Lines 124-133
```python
        if child.tag not in ret:
            ret[child.tag] = process_xml_element(child)
        else:
            # If there are multiple tags with the same name, they should be
            # coalesced into a list.
            if not isinstance(ret[child.tag], list):
                ret[child.tag] = [ret[child.tag]]
            ret[child.tag].append(process_xml_element(child))
    return ret
```
- **EN**: This chunk continues `process_xml_element` and expands its internal control flow or data movement. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段延续了 `process_xml_element`，进一步展开其内部控制流或数据流转。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 134-144
```python

def get_tests(workflow_run_id: int, workflow_run_attempt: int) -> list[dict[str, Any]]:
    with TemporaryDirectory() as temp_dir:
        print("Using temporary directory:", temp_dir)
        os.chdir(temp_dir)

        # Download and extract all the reports (both GHA and S3)
        s3_paths = download_s3_artifacts(
            "test-report", workflow_run_id, workflow_run_attempt
        )
        for path in s3_paths:
```
- **EN**: This chunk defines `get_tests`, which implements a focused step inside the ci statistics pipeline. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently.
- **CN**: 这一段定义了 `get_tests`，其作用是实现CI 统计流水线中的一个关键步骤。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。

### Lines 145-162
```python
            unzip(path)

        # Parse the reports and transform them to JSON
        test_cases = []
        mp = Pool(cpu_count())
        for xml_report in Path(".").glob("**/*.xml"):
            test_cases.append(
                mp.apply_async(
                    parse_xml_report,
                    args=(
                        "testcase",
                        xml_report,
                        workflow_run_id,
                        workflow_run_attempt,
                    ),
                )
            )
        mp.close()
```
- **EN**: This chunk continues `get_tests` and expands its internal control flow or data movement. Filesystem handling code normalizes paths and keeps generated or discovered artifacts in predictable locations. Structured-data parsing keeps repository metadata or intermediate reports in machine-readable form. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently.
- **CN**: 这一段延续了 `get_tests`，进一步展开其内部控制流或数据流转。 文件系统处理逻辑负责规范化路径，并把生成或发现的产物放在可预测的位置。 结构化数据解析使仓库元数据或中间报告保持机器可读形式。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。

### Lines 163-178
```python
        mp.join()
        test_cases = [tc.get() for tc in test_cases]
        flattened = [item for sublist in test_cases for item in sublist]
        return flattened


def backfill_test_jsons_while_running(
    workflow_run_id: int, workflow_run_attempt: int
) -> None:
    # The bucket name name is a bit misleading, usually the jsons should be
    # uploaded while the job is running, but that won't happen if the job
    # doesn't have permissions to write to the bucket or if there was an error
    with TemporaryDirectory() as temp_dir:
        print("Using temporary directory:", temp_dir)
        os.chdir(temp_dir)
```
- **EN**: This chunk defines `backfill_test_jsons_while_running`, which orchestrates command-line execution and forwards parsed arguments into the core logic. Structured-data parsing keeps repository metadata or intermediate reports in machine-readable form. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段定义了 `backfill_test_jsons_while_running`，其作用是编排命令行执行，并把解析后的参数转入核心逻辑。 结构化数据解析使仓库元数据或中间报告保持机器可读形式。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 179-187
```python
        # Download and extract all the reports (both GHA and S3)
        s3_xmls = download_s3_artifacts(
            "test-report", workflow_run_id, workflow_run_attempt
        )

        s3_jsons = download_s3_artifacts(
            "test-jsons", workflow_run_id, workflow_run_attempt
        )
```
- **EN**: This chunk continues `backfill_test_jsons_while_running` and expands its internal control flow or data movement. Structured-data parsing keeps repository metadata or intermediate reports in machine-readable form.
- **CN**: 这一段延续了 `backfill_test_jsons_while_running`，进一步展开其内部控制流或数据流转。 结构化数据解析使仓库元数据或中间报告保持机器可读形式。

### Lines 188-197
```python
        # Unzip artifacts and save their locations
        unzipped_xml_dirs = [unzip(path) for path in s3_xmls]
        unzipped_json_dirs = [unzip(path) for path in s3_jsons]

        all_existing_jsons = []
        for unzipped_dir in unzipped_json_dirs:
            all_existing_jsons.extend(
                [
                    str(Path(json_report).relative_to(unzipped_dir))
                    for json_report in unzipped_dir.glob("**/*.json")
```
- **EN**: This chunk continues `backfill_test_jsons_while_running` and expands its internal control flow or data movement. Filesystem handling code normalizes paths and keeps generated or discovered artifacts in predictable locations. Structured-data parsing keeps repository metadata or intermediate reports in machine-readable form. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently.
- **CN**: 这一段延续了 `backfill_test_jsons_while_running`，进一步展开其内部控制流或数据流转。 文件系统处理逻辑负责规范化路径，并把生成或发现的产物放在可预测的位置。 结构化数据解析使仓库元数据或中间报告保持机器可读形式。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。

### Lines 198-208
```python
                ]
            )

        for unzipped_dir in unzipped_xml_dirs:
            for xml in unzipped_dir.glob("**/*.xml"):
                corresponding_json = str(
                    xml.with_suffix(".json").relative_to(
                        unzipped_dir / "test" / "test-reports"
                    )
                )
                if corresponding_json in all_existing_jsons:
```
- **EN**: This chunk continues `backfill_test_jsons_while_running` and expands its internal control flow or data movement. Filesystem handling code normalizes paths and keeps generated or discovered artifacts in predictable locations. Structured-data parsing keeps repository metadata or intermediate reports in machine-readable form. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段延续了 `backfill_test_jsons_while_running`，进一步展开其内部控制流或数据流转。 文件系统处理逻辑负责规范化路径，并把生成或发现的产物放在可预测的位置。 结构化数据解析使仓库元数据或中间报告保持机器可读形式。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 209-226
```python
                    print(f"Skipping upload for existing test json for {xml}")
                    continue
                # print(f"Uploading missing test json for {xml}")
                job_id = get_job_id(xml)
                test_cases = parse_xml_report(
                    "testcase",
                    xml,
                    workflow_run_id,
                    workflow_run_attempt,
                    job_id,
                )
                json_file = xml.with_suffix(".json")
                s3_key = (
                    json_file.relative_to(unzipped_dir / "test" / "test-reports")
                    .as_posix()
                    .replace("/", "_")
                )
                s3_key = f"test_jsons_while_running/{workflow_run_id}/{job_id}/{s3_key}"
```
- **EN**: This chunk continues `backfill_test_jsons_while_running` and expands its internal control flow or data movement. Structured-data parsing keeps repository metadata or intermediate reports in machine-readable form. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently.
- **CN**: 这一段延续了 `backfill_test_jsons_while_running`，进一步展开其内部控制流或数据流转。 结构化数据解析使仓库元数据或中间报告保持机器可读形式。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。

### Lines 227-235
```python
                upload_to_s3("gha-artifacts", s3_key, remove_nan_inf(test_cases))


def summarize_test_cases(test_cases: list[dict[str, Any]]) -> list[dict[str, Any]]:
    """Group test cases by classname, file, and job_id. We perform the aggregation
    manually instead of using the `test-suite` XML tag because xmlrunner does
    not produce reliable output for it.
    """
```
- **EN**: This docstring block explains the tool entry point, generation contract, or usage expectations before executable logic begins. This chunk defines `summarize_test_cases`, which implements a focused step inside the ci statistics pipeline. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently.
- **CN**: 这一段文档字符串在可执行逻辑开始前说明了工具入口、生成约定或使用方式。 这一段定义了 `summarize_test_cases`，其作用是实现CI 统计流水线中的一个关键步骤。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。

### Lines 236-246
```python
    def get_key(test_case: dict[str, Any]) -> Any:
        return (
            test_case.get("file"),
            test_case.get("classname"),
            test_case["job_id"],
            test_case["workflow_id"],
            test_case["workflow_run_attempt"],
            # [see: invoking file]
            test_case["invoking_file"],
        )
```
- **EN**: This chunk defines `get_key`, which implements a focused step inside the ci statistics pipeline. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段定义了 `get_key`，其作用是实现CI 统计流水线中的一个关键步骤。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 247-263
```python
    def init_value(test_case: dict[str, Any]) -> dict[str, Any]:
        return {
            "file": test_case.get("file"),
            "classname": test_case.get("classname"),
            "job_id": test_case["job_id"],
            "workflow_id": test_case["workflow_id"],
            "workflow_run_attempt": test_case["workflow_run_attempt"],
            # [see: invoking file]
            "invoking_file": test_case["invoking_file"],
            "tests": 0,
            "failures": 0,
            "errors": 0,
            "skipped": 0,
            "successes": 0,
            "time": 0.0,
        }
```
- **EN**: This chunk defines `init_value`, which implements a focused step inside the ci statistics pipeline. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段定义了 `init_value`，其作用是实现CI 统计流水线中的一个关键步骤。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 264-272
```python
    ret = {}
    for test_case in test_cases:
        key = get_key(test_case)
        if key not in ret:
            ret[key] = init_value(test_case)

        ret[key]["tests"] += 1

        if "failure" in test_case:
```
- **EN**: This chunk continues `init_value` and expands its internal control flow or data movement. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段延续了 `init_value`，进一步展开其内部控制流或数据流转。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 273-283
```python
            ret[key]["failures"] += 1
        elif "error" in test_case:
            ret[key]["errors"] += 1
        elif "skipped" in test_case:
            ret[key]["skipped"] += 1
        else:
            ret[key]["successes"] += 1

        ret[key]["time"] += test_case["time"]
    return list(ret.values())
```
- **EN**: This chunk continues `init_value` and expands its internal control flow or data movement. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段延续了 `init_value`，进一步展开其内部控制流或数据流转。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 284-301
```python

if __name__ == "__main__":
    parser = argparse.ArgumentParser(description="Upload test stats to s3")
    parser.add_argument(
        "--workflow-run-id",
        required=True,
        help="id of the workflow to get artifacts from",
    )
    parser.add_argument(
        "--workflow-run-attempt",
        type=int,
        required=True,
        help="which retry of the workflow this is",
    )
    parser.add_argument(
        "--head-branch",
        required=True,
        help="Head branch of the workflow",
```
- **EN**: This chunk continues `init_value` and expands its internal control flow or data movement. Argument-parsing logic turns command-line inputs into structured parameters for the tool core. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段延续了 `init_value`，进一步展开其内部控制流或数据流转。 参数解析逻辑把命令行输入转换为工具核心可以直接消费的结构化参数。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 302-311
```python
    )
    parser.add_argument(
        "--head-repository",
        required=True,
        help="Head repository of the workflow",
    )
    args = parser.parse_args()

    print(f"Workflow id is: {args.workflow_run_id}")
```
- **EN**: This chunk continues `init_value` and expands its internal control flow or data movement. Argument-parsing logic turns command-line inputs into structured parameters for the tool core.
- **CN**: 这一段延续了 `init_value`，进一步展开其内部控制流或数据流转。 参数解析逻辑把命令行输入转换为工具核心可以直接消费的结构化参数。

### Lines 312-320
```python
    test_cases = get_tests(args.workflow_run_id, args.workflow_run_attempt)

    # Flush stdout so that any errors in the upload show up last in the logs.
    sys.stdout.flush()

    # For PRs, only upload a summary of test_runs. This helps lower the
    # volume of writes we do to the HUD backend database.
    test_case_summary = summarize_test_cases(test_cases)
```
- **EN**: This chunk continues `init_value` and expands its internal control flow or data movement.
- **CN**: 这一段延续了 `init_value`，进一步展开其内部控制流或数据流转。

### Lines 321-332
```python
    upload_workflow_stats_to_s3(
        args.workflow_run_id,
        args.workflow_run_attempt,
        "test_run_summary",
        remove_nan_inf(test_case_summary),
    )

    # Separate out the failed test cases.
    # Uploading everything is too data intensive most of the time,
    # but these will be just a tiny fraction.
    failed_tests_cases = []
    for test_case in test_cases:
```
- **EN**: This chunk continues `init_value` and expands its internal control flow or data movement. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently.
- **CN**: 这一段延续了 `init_value`，进一步展开其内部控制流或数据流转。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。

### Lines 333-342
```python
        if "rerun" in test_case or "failure" in test_case or "error" in test_case:
            failed_tests_cases.append(test_case)

    upload_workflow_stats_to_s3(
        args.workflow_run_id,
        args.workflow_run_attempt,
        "failed_test_runs",
        remove_nan_inf(failed_tests_cases),
    )
```
- **EN**: This chunk continues `init_value` and expands its internal control flow or data movement. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段延续了 `init_value`，进一步展开其内部控制流或数据流转。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 343-354
```python
    backfill_test_jsons_while_running(args.workflow_run_id, args.workflow_run_attempt)

    # Upload full test_run only for trusted refs (main or trunk/{sha} tags)
    if should_upload_full_test_run(args.head_branch, args.head_repository):
        # For jobs on main branch, upload everything.
        upload_workflow_stats_to_s3(
            args.workflow_run_id,
            args.workflow_run_attempt,
            "test_run",
            remove_nan_inf(test_cases),
        )
```
- **EN**: This chunk continues `init_value` and expands its internal control flow or data movement. Subprocess orchestration bridges this script to external build tools, linters, or system commands. Structured-data parsing keeps repository metadata or intermediate reports in machine-readable form. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段延续了 `init_value`，进一步展开其内部控制流或数据流转。 子进程编排把该脚本连接到外部构建工具、lint 或系统命令。 结构化数据解析使仓库元数据或中间报告保持机器可读形式。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 355-355
```python
    upload_additional_info(args.workflow_run_id, args.workflow_run_attempt)
```
- **EN**: This chunk continues `init_value` and expands its internal control flow or data movement.
- **CN**: 这一段延续了 `init_value`，进一步展开其内部控制流或数据流转。

## Key Concepts / 关键概念

- **CI statistics**
  - EN: This file belongs to the ci statistics layer and should be read in the context of neighboring scripts in the same subtree.
  - CN: 该文件属于CI 统计层，应结合同一子目录中的相邻脚本一起理解。
- **Statistics reporting**
  - EN: The implementation aggregates measurements and turns them into summaries or dashboards.
  - CN: 该实现聚合度量数据，并将其转化为摘要或仪表板。
- **CLI interface**
  - EN: The implementation exposes a command-line entry point and converts arguments into tool actions.
  - CN: 该实现暴露命令行入口，并把参数转换为工具动作。
- **Filesystem coordination**
  - EN: The code spends effort on path normalization, file discovery, or directory layout management.
  - CN: 代码重点处理路径规范化、文件发现或目录布局管理。
- **Type-shape surface**
  - EN: The file models public Python typing information, stubs, or developer-facing signatures.
  - CN: 该文件建模公共 Python 类型信息、类型桩或面向开发者的签名。
- **should_upload_full_test_run**
  - EN: `should_upload_full_test_run` is one of the main local symbols exposed or implemented here.
  - CN: `should_upload_full_test_run` 是此处暴露或实现的主要局部符号之一。
## Dependencies / 依赖关系

- **Internal PyTorch Python modules / 内部 PyTorch Python 模块**: `tools.stats.test_dashboard`, `tools.stats.upload_stats_lib`
- **Python standard library / Python 标准库**: `__future__`, `argparse`, `os`, `re`, `sys`, `xml.etree.ElementTree`, `multiprocessing`, `pathlib`, `tempfile`, `typing`
- **Primary symbols in this file / 本文件核心符号**: `should_upload_full_test_run`, `parse_xml_report`, `process_xml_element`, `get_tests`, `backfill_test_jsons_while_running`, `summarize_test_cases`
