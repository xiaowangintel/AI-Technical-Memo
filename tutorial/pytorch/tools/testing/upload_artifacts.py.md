# upload_artifacts.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `tools/testing/upload_artifacts.py`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Provides shared testing infrastructure, test-selection helpers, or execution utilities for PyTorch CI.
- **Purpose (CN)**: 提供 PyTorch CI 使用的共享测试基础设施、测试选择辅助逻辑或执行工具。
## Line-by-Line Analysis / 逐行分析

### Lines 1-10
```python
import glob
import gzip
import json
import os
import time
import zipfile
from functools import lru_cache
from pathlib import Path
from typing import Any
```
- **EN**: This block assembles the Python-side dependencies, importing Python standard-library modules such as glob, gzip, json, and 6 more. Filesystem handling code normalizes paths and keeps generated or discovered artifacts in predictable locations. Structured-data parsing keeps repository metadata or intermediate reports in machine-readable form.
- **CN**: 这一段组织 Python 侧依赖，引入了Python 标准库模块，如 glob、gzip、json 等共 9 项。 文件系统处理逻辑负责规范化路径，并把生成或发现的产物放在可预测的位置。 结构化数据解析使仓库元数据或中间报告保持机器可读形式。

### Lines 11-19
```python
from filelock import FileLock, Timeout

from tools.stats.upload_test_stats import parse_xml_report


REPO_ROOT = Path(__file__).resolve().parent.parent.parent
LAST_UPDATED = 0.0
LOG_BUCKET_PREFIX = "temp_logs"
```
- **EN**: This block assembles the Python-side dependencies, importing internal PyTorch modules such as tools.stats.upload_test_stats; external packages such as filelock. Configuration constants such as REPO_ROOT, LAST_UPDATED, LOG_BUCKET_PREFIX centralize defaults so later functions share the same policy knobs.
- **CN**: 这一段组织 Python 侧依赖，引入了内部 PyTorch 模块，如 tools.stats.upload_test_stats；外部依赖包，如 filelock。 REPO_ROOT、LAST_UPDATED、LOG_BUCKET_PREFIX 等配置常量集中定义默认值，使后续函数共享同一套策略开关。

### Lines 20-28
```python

@lru_cache(maxsize=1)
def get_s3_resource() -> Any:
    import boto3  # type: ignore[import]

    return boto3.client("s3")


def zip_artifact(file_name: str, paths: list[str]) -> None:
```
- **EN**: This block assembles the Python-side dependencies, importing external packages such as boto3  # type: ignore[import]. This chunk defines `zip_artifact`, which implements a focused step inside the testing infrastructure pipeline. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段组织 Python 侧依赖，引入了外部依赖包，如 boto3  # type: ignore[import]。 这一段定义了 `zip_artifact`，其作用是实现测试基础设施流水线中的一个关键步骤。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 29-37
```python
    """Zip the files in the paths listed into file_name. The paths will be used
    in a glob and should be relative to REPO_ROOT."""

    with zipfile.ZipFile(file_name, "w") as f:
        for path in paths:
            for file in glob.glob(f"{REPO_ROOT}/{path}", recursive=True):
                f.write(file, os.path.relpath(file, REPO_ROOT))
```
- **EN**: This docstring block explains the tool entry point, generation contract, or usage expectations before executable logic begins. This chunk continues `zip_artifact` and expands its internal control flow or data movement. Filesystem handling code normalizes paths and keeps generated or discovered artifacts in predictable locations. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently.
- **CN**: 这一段文档字符串在可执行逻辑开始前说明了工具入口、生成约定或使用方式。 这一段延续了 `zip_artifact`，进一步展开其内部控制流或数据流转。 文件系统处理逻辑负责规范化路径，并把生成或发现的产物放在可预测的位置。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。

### Lines 38-47
```python
def concated_logs() -> str:
    """Concatenate all the logs in the test-reports directory into a single string."""
    logs = []
    for log_file in glob.glob(
        f"{REPO_ROOT}/test/test-reports/**/*.log", recursive=True
    ):
        logs.append(f"=== {log_file} ===")
        with open(log_file) as f:
            # For every line, prefix with fake timestamp for log classifier
            for line in f:
```
- **EN**: This docstring block explains the tool entry point, generation contract, or usage expectations before executable logic begins. This chunk defines `concated_logs`, which implements a focused step inside the testing infrastructure pipeline. Filesystem handling code normalizes paths and keeps generated or discovered artifacts in predictable locations. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently.
- **CN**: 这一段文档字符串在可执行逻辑开始前说明了工具入口、生成约定或使用方式。 这一段定义了 `concated_logs`，其作用是实现测试基础设施流水线中的一个关键步骤。 文件系统处理逻辑负责规范化路径，并把生成或发现的产物放在可预测的位置。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。

### Lines 48-59
```python
                line = line.rstrip("\n")  # Remove any trailing newline
                logs.append(f"2020-01-01T00:00:00.0000000Z {line}")
    return "\n".join(logs)


def upload_to_s3_artifacts(failed: bool) -> None:
    """Upload the file to S3."""
    workflow_id = os.environ.get("GITHUB_RUN_ID")
    workflow_run_attempt = os.environ.get("GITHUB_RUN_ATTEMPT")
    file_suffix = os.environ.get("ARTIFACTS_FILE_SUFFIX")
    job_id = os.environ.get("JOB_ID")
    if not workflow_id or not workflow_run_attempt or not file_suffix:
```
- **EN**: This docstring block explains the tool entry point, generation contract, or usage expectations before executable logic begins. This chunk defines `upload_to_s3_artifacts`, which parses or loads structured input into tool-friendly data structures. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段文档字符串在可执行逻辑开始前说明了工具入口、生成约定或使用方式。 这一段定义了 `upload_to_s3_artifacts`，其作用是把结构化输入解析或加载为工具可处理的数据结构。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 60-74
```python
        print(
            "GITHUB_RUN_ID, GITHUB_RUN_ATTEMPT, or ARTIFACTS_FILE_SUFFIX not set, not uploading"
        )
        return

    test_reports_zip_path = f"{REPO_ROOT}/test-reports-{file_suffix}.zip"
    zip_artifact(
        test_reports_zip_path,
        ["test/test-reports/**/*.xml", "test/test-reports/**/*.csv"],
    )
    test_logs_zip_path = f"{REPO_ROOT}/logs-{file_suffix}.zip"
    zip_artifact(test_logs_zip_path, ["test/test-reports/**/*.log"])
    jsons_zip_path = f"{REPO_ROOT}/test-jsons-{file_suffix}.zip"
    zip_artifact(jsons_zip_path, ["test/test-reports/**/*.json"])
```
- **EN**: This chunk continues `upload_to_s3_artifacts` and expands its internal control flow or data movement. Structured-data parsing keeps repository metadata or intermediate reports in machine-readable form. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段延续了 `upload_to_s3_artifacts`，进一步展开其内部控制流或数据流转。 结构化数据解析使仓库元数据或中间报告保持机器可读形式。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 75-92
```python
    s3_prefix = f"pytorch/pytorch/{workflow_id}/{workflow_run_attempt}/artifact"
    get_s3_resource().upload_file(
        test_reports_zip_path,
        "gha-artifacts",
        f"{s3_prefix}/{Path(test_reports_zip_path).name}",
    )
    get_s3_resource().upload_file(
        test_logs_zip_path,
        "gha-artifacts",
        f"{s3_prefix}/{Path(test_logs_zip_path).name}",
    )
    get_s3_resource().upload_file(
        test_logs_zip_path,
        "gha-artifacts",
        f"{s3_prefix}/{Path(jsons_zip_path).name}",
    )
    get_s3_resource().put_object(
        Body=b"",
```
- **EN**: This chunk continues `upload_to_s3_artifacts` and expands its internal control flow or data movement. Structured-data parsing keeps repository metadata or intermediate reports in machine-readable form.
- **CN**: 这一段延续了 `upload_to_s3_artifacts`，进一步展开其内部控制流或数据流转。 结构化数据解析使仓库元数据或中间报告保持机器可读形式。

### Lines 93-108
```python
        Bucket="gha-artifacts",
        Key=f"workflows_failing_pending_upload/{workflow_id}.txt",
    )
    if job_id and failed:
        logs = concated_logs()
        # Put logs into bucket so log classifier can access them. We cannot get
        # the actual GH logs so this will have to be a proxy.
        print(f"Uploading logs for {job_id} to S3")
        get_s3_resource().put_object(
            Body=gzip.compress(logs.encode("utf-8")),
            Bucket="gha-artifacts",
            Key=f"{LOG_BUCKET_PREFIX}/{job_id}",
            ContentType="text/plain",
            ContentEncoding="gzip",
        )
```
- **EN**: This chunk continues `upload_to_s3_artifacts` and expands its internal control flow or data movement. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段延续了 `upload_to_s3_artifacts`，进一步展开其内部控制流或数据流转。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 109-118
```python

def zip_and_upload_artifacts(failed: bool) -> None:
    # not thread safe but correctness of the LAST_UPDATED var doesn't really
    # matter for this
    # Upload if a test failed or every 20 minutes
    global LAST_UPDATED

    if failed or time.time() - LAST_UPDATED > 20 * 60:
        start = time.time()
        try:
```
- **EN**: This chunk defines `zip_and_upload_artifacts`, which parses or loads structured input into tool-friendly data structures. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. Exception-handling paths convert tool failures into clearer control flow or user-facing diagnostics.
- **CN**: 这一段定义了 `zip_and_upload_artifacts`，其作用是把结构化输入解析或加载为工具可处理的数据结构。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 异常处理路径把工具失败转换为更清晰的控制流或面向用户的诊断信息。

### Lines 119-128
```python
            upload_to_s3_artifacts(failed=failed)
            LAST_UPDATED = time.time()
        except Exception as e:
            print(f"Failed to upload artifacts: {e}")
        print(f"Uploading artifacts took {time.time() - start:.2f} seconds")


def trigger_upload_test_stats_intermediate_workflow() -> None:
    import requests
```
- **EN**: This block assembles the Python-side dependencies, importing external packages such as requests. This chunk defines `trigger_upload_test_stats_intermediate_workflow`, which parses or loads structured input into tool-friendly data structures. Configuration constants such as LAST_UPDATED centralize defaults so later functions share the same policy knobs. Exception-handling paths convert tool failures into clearer control flow or user-facing diagnostics.
- **CN**: 这一段组织 Python 侧依赖，引入了外部依赖包，如 requests。 这一段定义了 `trigger_upload_test_stats_intermediate_workflow`，其作用是把结构化输入解析或加载为工具可处理的数据结构。 LAST_UPDATED 等配置常量集中定义默认值，使后续函数共享同一套策略开关。 异常处理路径把工具失败转换为更清晰的控制流或面向用户的诊断信息。

### Lines 129-146
```python
    # The GITHUB_TOKEN cannot trigger workflow so this isn't used for now
    print("Triggering upload_test_stats_intermediate workflow")
    x = requests.post(
        "https://api.github.com/repos/pytorch/pytorch/actions/workflows/upload_test_stats_intermediate.yml/dispatches",
        headers={
            "Accept": "application/vnd.github.v3+json",
            "Authorization": f"Bearer {os.environ.get('GITHUB_TOKEN')}",
        },
        json={
            "ref": "main",
            "inputs": {
                "workflow_run_id": os.environ.get("GITHUB_RUN_ID"),
                "workflow_run_attempt": os.environ.get("GITHUB_RUN_ATTEMPT"),
            },
        },
    )
    print(x.text)
```
- **EN**: This chunk continues `trigger_upload_test_stats_intermediate_workflow` and expands its internal control flow or data movement. Structured-data parsing keeps repository metadata or intermediate reports in machine-readable form. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently.
- **CN**: 这一段延续了 `trigger_upload_test_stats_intermediate_workflow`，进一步展开其内部控制流或数据流转。 结构化数据解析使仓库元数据或中间报告保持机器可读形式。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。

### Lines 147-156
```python

def parse_xml_and_upload_json() -> None:
    """
    Parse xml test reports that do not yet have a corresponding json report
    uploaded to s3, and upload the json reports to s3. Use filelock to avoid
    uploading the same file from multiple processes.
    """
    try:
        job_id: int | None = int(os.environ.get("JOB_ID", 0))
        if job_id == 0:
```
- **EN**: This docstring block explains the tool entry point, generation contract, or usage expectations before executable logic begins. This chunk defines `parse_xml_and_upload_json`, which parses or loads structured input into tool-friendly data structures. Structured-data parsing keeps repository metadata or intermediate reports in machine-readable form. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. Exception-handling paths convert tool failures into clearer control flow or user-facing diagnostics.
- **CN**: 这一段文档字符串在可执行逻辑开始前说明了工具入口、生成约定或使用方式。 这一段定义了 `parse_xml_and_upload_json`，其作用是把结构化输入解析或加载为工具可处理的数据结构。 结构化数据解析使仓库元数据或中间报告保持机器可读形式。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 异常处理路径把工具失败转换为更清晰的控制流或面向用户的诊断信息。

### Lines 157-168
```python
            job_id = None
    except (ValueError, TypeError):
        job_id = None

    try:
        for xml_file in glob.glob(
            f"{REPO_ROOT}/test/test-reports/**/*.xml", recursive=True
        ):
            xml_path = Path(xml_file)
            json_file = xml_path.with_suffix(".json")
            lock = FileLock(str(json_file) + ".lock")
```
- **EN**: This chunk continues `parse_xml_and_upload_json` and expands its internal control flow or data movement. Filesystem handling code normalizes paths and keeps generated or discovered artifacts in predictable locations. Structured-data parsing keeps repository metadata or intermediate reports in machine-readable form. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Exception-handling paths convert tool failures into clearer control flow or user-facing diagnostics.
- **CN**: 这一段延续了 `parse_xml_and_upload_json`，进一步展开其内部控制流或数据流转。 文件系统处理逻辑负责规范化路径，并把生成或发现的产物放在可预测的位置。 结构化数据解析使仓库元数据或中间报告保持机器可读形式。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 异常处理路径把工具失败转换为更清晰的控制流或面向用户的诊断信息。

### Lines 169-181
```python
            try:
                lock.acquire(timeout=0)  # immediately fails if already locked
                if json_file.exists():
                    continue  # already uploaded
                test_cases = parse_xml_report(
                    "testcase",
                    xml_path,
                    int(os.environ.get("GITHUB_RUN_ID", "0")),
                    int(os.environ.get("GITHUB_RUN_ATTEMPT", "0")),
                    job_id,
                )
                line_by_line_jsons = "\n".join([json.dumps(tc) for tc in test_cases])
```
- **EN**: This chunk continues `parse_xml_and_upload_json` and expands its internal control flow or data movement. Structured-data parsing keeps repository metadata or intermediate reports in machine-readable form. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. Exception-handling paths convert tool failures into clearer control flow or user-facing diagnostics.
- **CN**: 这一段延续了 `parse_xml_and_upload_json`，进一步展开其内部控制流或数据流转。 结构化数据解析使仓库元数据或中间报告保持机器可读形式。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 异常处理路径把工具失败转换为更清晰的控制流或面向用户的诊断信息。

### Lines 182-196
```python
                gzipped = gzip.compress(line_by_line_jsons.encode("utf-8"))
                s3_key = (
                    json_file.relative_to(REPO_ROOT / "test/test-reports")
                    .as_posix()
                    .replace("/", "_")
                )

                get_s3_resource().put_object(
                    Body=gzipped,
                    Bucket="gha-artifacts",
                    Key=f"test_jsons_while_running/{os.environ.get('GITHUB_RUN_ID')}/{job_id}/{s3_key}",
                    ContentType="application/json",
                    ContentEncoding="gzip",
                )
```
- **EN**: This chunk continues `parse_xml_and_upload_json` and expands its internal control flow or data movement. Structured-data parsing keeps repository metadata or intermediate reports in machine-readable form.
- **CN**: 这一段延续了 `parse_xml_and_upload_json`，进一步展开其内部控制流或数据流转。 结构化数据解析使仓库元数据或中间报告保持机器可读形式。

### Lines 197-205
```python
                # We don't need to save the json file locally, but doing so lets us
                # track which ones have been uploaded already. We could probably also
                # check S3
                with open(json_file, "w") as f:
                    f.write(line_by_line_jsons)
            except Timeout:
                continue  # another process is working on this file
            finally:
                if lock.is_locked:
```
- **EN**: This chunk continues `parse_xml_and_upload_json` and expands its internal control flow or data movement. Filesystem handling code normalizes paths and keeps generated or discovered artifacts in predictable locations. Structured-data parsing keeps repository metadata or intermediate reports in machine-readable form. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. Exception-handling paths convert tool failures into clearer control flow or user-facing diagnostics.
- **CN**: 这一段延续了 `parse_xml_and_upload_json`，进一步展开其内部控制流或数据流转。 文件系统处理逻辑负责规范化路径，并把生成或发现的产物放在可预测的位置。 结构化数据解析使仓库元数据或中间报告保持机器可读形式。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 异常处理路径把工具失败转换为更清晰的控制流或面向用户的诊断信息。

### Lines 206-216
```python
                    lock.release()
    except Exception as e:
        print(f"Failed to parse and upload json test reports: {e}")


def upload_adhoc_failure_json(invoking_file: str, current_failure: str) -> None:
    """
    manually upload a json to s3 indicating that the entire test file failed
    since xml was probably not generated in this case
    """
    try:
```
- **EN**: This docstring block explains the tool entry point, generation contract, or usage expectations before executable logic begins. This chunk defines `upload_adhoc_failure_json`, which parses or loads structured input into tool-friendly data structures. Structured-data parsing keeps repository metadata or intermediate reports in machine-readable form. Exception-handling paths convert tool failures into clearer control flow or user-facing diagnostics.
- **CN**: 这一段文档字符串在可执行逻辑开始前说明了工具入口、生成约定或使用方式。 这一段定义了 `upload_adhoc_failure_json`，其作用是把结构化输入解析或加载为工具可处理的数据结构。 结构化数据解析使仓库元数据或中间报告保持机器可读形式。 异常处理路径把工具失败转换为更清晰的控制流或面向用户的诊断信息。

### Lines 217-230
```python
        job_id = int(os.environ["JOB_ID"])
        workflow_id = int(os.environ["GITHUB_RUN_ID"])
    except Exception as e:
        print(f"Failed to get job_id or workflow_id: {e}")
        return

    split_failure = current_failure.split("::")
    if len(split_failure) >= 2:
        className = split_failure[-2]
        testName = split_failure[-1]
    else:
        testName = current_failure
        className = ""
```
- **EN**: This chunk continues `upload_adhoc_failure_json` and expands its internal control flow or data movement. Pattern-matching logic extracts structured facts from loosely formatted text inputs. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. Exception-handling paths convert tool failures into clearer control flow or user-facing diagnostics. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段延续了 `upload_adhoc_failure_json`，进一步展开其内部控制流或数据流转。 模式匹配逻辑从格式较松散的文本输入中提取结构化事实。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 异常处理路径把工具失败转换为更清晰的控制流或面向用户的诊断信息。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 231-248
```python
    message = "The test file failed but pytest did not generate xml.  The most likely cause is a segfault"
    j = {
        "invoking_file": invoking_file,
        "file": f"{invoking_file}.py",
        "name": testName,
        "classname": className,
        "workflow_id": workflow_id,
        "workflow_run_attempt": os.environ.get("GITHUB_RUN_ATTEMPT"),
        "job_id": job_id,
        "failure": {"message": message, "text": message},
    }
    gzipped = gzip.compress(json.dumps(j).encode("utf-8"))
    s3_key = f"{invoking_file.replace('/', '_')}_{os.urandom(8).hex()}.json"
    get_s3_resource().put_object(
        Body=gzipped,
        Bucket="gha-artifacts",
        Key=f"test_jsons_while_running/{workflow_id}/{job_id}/{s3_key}",
        ContentType="application/json",
```
- **EN**: This chunk continues `upload_adhoc_failure_json` and expands its internal control flow or data movement. Structured-data parsing keeps repository metadata or intermediate reports in machine-readable form.
- **CN**: 这一段延续了 `upload_adhoc_failure_json`，进一步展开其内部控制流或数据流转。 结构化数据解析使仓库元数据或中间报告保持机器可读形式。

### Lines 249-250
```python
        ContentEncoding="gzip",
    )
```
- **EN**: This chunk continues `upload_adhoc_failure_json` and expands its internal control flow or data movement.
- **CN**: 这一段延续了 `upload_adhoc_failure_json`，进一步展开其内部控制流或数据流转。

## Key Concepts / 关键概念

- **Testing infrastructure**
  - EN: This file belongs to the testing infrastructure layer and should be read in the context of neighboring scripts in the same subtree.
  - CN: 该文件属于测试基础设施层，应结合同一子目录中的相邻脚本一起理解。
- **GitHub automation**
  - EN: The file integrates with pull requests, workflow metadata, or repository automation flows.
  - CN: 该文件与 Pull Request、工作流元数据或仓库自动化流程集成。
- **Statistics reporting**
  - EN: The implementation aggregates measurements and turns them into summaries or dashboards.
  - CN: 该实现聚合度量数据，并将其转化为摘要或仪表板。
- **Filesystem coordination**
  - EN: The code spends effort on path normalization, file discovery, or directory layout management.
  - CN: 代码重点处理路径规范化、文件发现或目录布局管理。
- **Type-shape surface**
  - EN: The file models public Python typing information, stubs, or developer-facing signatures.
  - CN: 该文件建模公共 Python 类型信息、类型桩或面向开发者的签名。
- **REPO_ROOT**
  - EN: `REPO_ROOT` is one of the main local symbols exposed or implemented here.
  - CN: `REPO_ROOT` 是此处暴露或实现的主要局部符号之一。
## Dependencies / 依赖关系

- **Internal PyTorch Python modules / 内部 PyTorch Python 模块**: `tools.stats.upload_test_stats`
- **Python standard library / Python 标准库**: `glob`, `gzip`, `json`, `os`, `time`, `zipfile`, `functools`, `pathlib`, `typing`
- **External packages / 外部依赖包**: `filelock`, `boto3  # type: ignore[import]`, `requests`
- **Primary symbols in this file / 本文件核心符号**: `REPO_ROOT`, `LAST_UPDATED`, `LOG_BUCKET_PREFIX`, `get_s3_resource`, `zip_artifact`, `concated_logs`, `upload_to_s3_artifacts`, `zip_and_upload_artifacts`, `trigger_upload_test_stats_intermediate_workflow`, `parse_xml_and_upload_json`, `upload_adhoc_failure_json`
