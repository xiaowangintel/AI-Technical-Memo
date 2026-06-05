# upload_stats_lib.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `tools/stats/upload_stats_lib.py`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Implements statistics collection and reporting helpers for build, CI, or repository health signals.
- **Purpose (CN)**: 实现统计采集与报告辅助逻辑，用于构建、CI 或仓库健康度信号。
## Line-by-Line Analysis / 逐行分析

### Lines 1-13
```python
from __future__ import annotations

import gzip
import io
import json
import math
import os
import time
import zipfile
from functools import lru_cache
from pathlib import Path
from typing import Any, cast, TYPE_CHECKING
```
- **EN**: This block assembles the Python-side dependencies, importing Python standard-library modules such as __future__, gzip, io, and 8 more. Filesystem handling code normalizes paths and keeps generated or discovered artifacts in predictable locations. Structured-data parsing keeps repository metadata or intermediate reports in machine-readable form.
- **CN**: 这一段组织 Python 侧依赖，引入了Python 标准库模块，如 __future__、gzip、io 等共 11 项。 文件系统处理逻辑负责规范化路径，并把生成或发现的产物放在可预测的位置。 结构化数据解析使仓库元数据或中间报告保持机器可读形式。

### Lines 14-23
```python
import boto3  # type: ignore[import]
import requests


if TYPE_CHECKING:
    from collections.abc import Callable


PYTORCH_REPO = "https://api.github.com/repos/pytorch/pytorch"
```
- **EN**: This block assembles the Python-side dependencies, importing Python standard-library modules such as collections.abc; external packages such as boto3  # type: ignore[import], requests. Configuration constants such as PYTORCH_REPO centralize defaults so later functions share the same policy knobs. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段组织 Python 侧依赖，引入了Python 标准库模块，如 collections.abc；外部依赖包，如 boto3  # type: ignore[import]、requests。 PYTORCH_REPO 等配置常量集中定义默认值，使后续函数共享同一套策略开关。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 24-32
```python

@lru_cache
def get_s3_resource() -> Any:
    return boto3.resource("s3")


GHA_ARTIFACTS_BUCKET = "gha-artifacts"
```
- **EN**: This chunk defines `get_s3_resource`, which implements a focused step inside the ci statistics pipeline. Configuration constants such as GHA_ARTIFACTS_BUCKET centralize defaults so later functions share the same policy knobs. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段定义了 `get_s3_resource`，其作用是实现CI 统计流水线中的一个关键步骤。 GHA_ARTIFACTS_BUCKET 等配置常量集中定义默认值，使后续函数共享同一套策略开关。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 33-43
```python
# NB: In CI, a flaky test is usually retried 3 times, then the test file would be rerun
# 2 more times
MAX_RETRY_IN_NON_DISABLED_MODE = 3 * 3


def _get_request_headers() -> dict[str, str]:
    return {
        "Accept": "application/vnd.github.v3+json",
        "Authorization": "token " + os.environ["GITHUB_TOKEN"],
    }
```
- **EN**: This chunk defines `_get_request_headers`, which implements a focused step inside the ci statistics pipeline. Configuration constants such as MAX_RETRY_IN_NON_DISABLED_MODE centralize defaults so later functions share the same policy knobs. Structured-data parsing keeps repository metadata or intermediate reports in machine-readable form. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段定义了 `_get_request_headers`，其作用是实现CI 统计流水线中的一个关键步骤。 MAX_RETRY_IN_NON_DISABLED_MODE 等配置常量集中定义默认值，使后续函数共享同一套策略开关。 结构化数据解析使仓库元数据或中间报告保持机器可读形式。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 44-52
```python

def _get_artifact_urls(prefix: str, workflow_run_id: int) -> dict[Path, str]:
    """Get all workflow artifacts with 'test-report' in the name."""
    response = requests.get(
        f"{PYTORCH_REPO}/actions/runs/{workflow_run_id}/artifacts?per_page=100",
        headers=_get_request_headers(),
    )
    artifacts = response.json()["artifacts"]
    while "next" in response.links:
```
- **EN**: This docstring block explains the tool entry point, generation contract, or usage expectations before executable logic begins. This chunk defines `_get_artifact_urls`, which implements a focused step inside the ci statistics pipeline. Structured-data parsing keeps repository metadata or intermediate reports in machine-readable form. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently.
- **CN**: 这一段文档字符串在可执行逻辑开始前说明了工具入口、生成约定或使用方式。 这一段定义了 `_get_artifact_urls`，其作用是实现CI 统计流水线中的一个关键步骤。 结构化数据解析使仓库元数据或中间报告保持机器可读形式。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。

### Lines 53-63
```python
        response = requests.get(
            response.links["next"]["url"], headers=_get_request_headers()
        )
        artifacts.extend(response.json()["artifacts"])

    artifact_urls = {}
    for artifact in artifacts:
        if artifact["name"].startswith(prefix):
            artifact_urls[Path(artifact["name"])] = artifact["archive_download_url"]
    return artifact_urls
```
- **EN**: This chunk continues `_get_artifact_urls` and expands its internal control flow or data movement. Structured-data parsing keeps repository metadata or intermediate reports in machine-readable form. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段延续了 `_get_artifact_urls`，进一步展开其内部控制流或数据流转。 结构化数据解析使仓库元数据或中间报告保持机器可读形式。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 64-76
```python

def _download_artifact(
    artifact_name: Path, artifact_url: str, workflow_run_attempt: int
) -> Path:
    # [Artifact run attempt]
    # All artifacts on a workflow share a single namespace. However, we can
    # re-run a workflow and produce a new set of artifacts. To avoid name
    # collisions, we add `-runattempt1<run #>-` somewhere in the artifact name.
    #
    # This code parses out the run attempt number from the artifact name. If it
    # doesn't match the one specified on the command line, skip it.
    atoms = str(artifact_name).split("-")
    for atom in atoms:
```
- **EN**: This chunk defines `_download_artifact`, which parses or loads structured input into tool-friendly data structures. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently.
- **CN**: 这一段定义了 `_download_artifact`，其作用是把结构化输入解析或加载为工具可处理的数据结构。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。

### Lines 77-86
```python
        if atom.startswith("runattempt"):
            found_run_attempt = int(atom[len("runattempt") :])
            if workflow_run_attempt != found_run_attempt:
                print(
                    f"Skipping {artifact_name} as it is an invalid run attempt. "
                    f"Expected {workflow_run_attempt}, found {found_run_attempt}."
                )

    print(f"Downloading {artifact_name}")
```
- **EN**: This chunk continues `_download_artifact` and expands its internal control flow or data movement. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段延续了 `_download_artifact`，进一步展开其内部控制流或数据流转。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 87-104
```python
    response = requests.get(artifact_url, headers=_get_request_headers())
    with open(artifact_name, "wb") as f:
        f.write(response.content)
    return artifact_name


def download_s3_artifacts(
    prefix: str,
    workflow_run_id: int,
    workflow_run_attempt: int,
    job_id: int | None = None,
) -> list[Path]:
    bucket = get_s3_resource().Bucket(GHA_ARTIFACTS_BUCKET)
    objs = bucket.objects.filter(
        Prefix=f"pytorch/pytorch/{workflow_run_id}/{workflow_run_attempt}/artifact/{prefix}"
    )
    found_one = False
    paths = []
```
- **EN**: This chunk defines `download_s3_artifacts`, which parses or loads structured input into tool-friendly data structures. Filesystem handling code normalizes paths and keeps generated or discovered artifacts in predictable locations. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段定义了 `download_s3_artifacts`，其作用是把结构化输入解析或加载为工具可处理的数据结构。 文件系统处理逻辑负责规范化路径，并把生成或发现的产物放在可预测的位置。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 105-116
```python
    for obj in objs:
        object_name = Path(obj.key).name
        # target an artifact for a specific job_id if provided, otherwise skip the download.
        if job_id is not None and str(job_id) not in object_name:
            continue
        found_one = True
        p = Path(Path(obj.key).name)
        print(f"Downloading {p}")
        with open(p, "wb") as f:
            f.write(obj.get()["Body"].read())
        paths.append(p)
```
- **EN**: This chunk continues `download_s3_artifacts` and expands its internal control flow or data movement. Filesystem handling code normalizes paths and keeps generated or discovered artifacts in predictable locations. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段延续了 `download_s3_artifacts`，进一步展开其内部控制流或数据流转。 文件系统处理逻辑负责规范化路径，并把生成或发现的产物放在可预测的位置。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 117-125
```python
    if not found_one:
        print(
            "::warning title=s3 artifacts not found::"
            "Didn't find any test reports in s3, there might be a bug!"
        )
    return paths


def download_gha_artifacts(
```
- **EN**: This chunk defines `download_gha_artifacts`, which parses or loads structured input into tool-friendly data structures. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段定义了 `download_gha_artifacts`，其作用是把结构化输入解析或加载为工具可处理的数据结构。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 126-134
```python
    prefix: str, workflow_run_id: int, workflow_run_attempt: int
) -> list[Path]:
    artifact_urls = _get_artifact_urls(prefix, workflow_run_id)
    paths = []
    for name, url in artifact_urls.items():
        paths.append(_download_artifact(Path(name), url, workflow_run_attempt))
    return paths
```
- **EN**: This chunk continues `download_gha_artifacts` and expands its internal control flow or data movement. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段延续了 `download_gha_artifacts`，进一步展开其内部控制流或数据流转。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 135-144
```python
def upload_to_dynamodb(
    dynamodb_table: str,
    repo: str,
    docs: list[Any],
    generate_partition_key: Callable[[str, dict[str, Any]], str] | None,
) -> None:
    print(f"Writing {len(docs)} documents to DynamoDB {dynamodb_table}")
    # https://boto3.amazonaws.com/v1/documentation/api/latest/guide/dynamodb.html#batch-writing
    with boto3.resource("dynamodb").Table(dynamodb_table).batch_writer() as batch:
        for doc in docs:
```
- **EN**: This chunk defines `upload_to_dynamodb`, which parses or loads structured input into tool-friendly data structures. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently.
- **CN**: 这一段定义了 `upload_to_dynamodb`，其作用是把结构化输入解析或加载为工具可处理的数据结构。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。

### Lines 145-153
```python
            if generate_partition_key:
                doc["dynamoKey"] = generate_partition_key(repo, doc)
            # This is to move away the _event_time field from Rockset, which we cannot use when
            # reimport the data
            doc["timestamp"] = int(round(time.time() * 1000))
            batch.put_item(Item=doc)


def upload_to_s3(
```
- **EN**: This chunk defines `upload_to_s3`, which parses or loads structured input into tool-friendly data structures. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段定义了 `upload_to_s3`，其作用是把结构化输入解析或加载为工具可处理的数据结构。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 154-163
```python
    bucket_name: str,
    key: str,
    docs: list[dict[str, Any]],
) -> None:
    print(f"Writing {len(docs)} documents to S3 {bucket_name}/{key}")
    body = io.StringIO()
    for doc in docs:
        json.dump(doc, body)
        body.write("\n")
```
- **EN**: This chunk continues `upload_to_s3` and expands its internal control flow or data movement. Structured-data parsing keeps repository metadata or intermediate reports in machine-readable form. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently.
- **CN**: 这一段延续了 `upload_to_s3`，进一步展开其内部控制流或数据流转。 结构化数据解析使仓库元数据或中间报告保持机器可读形式。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。

### Lines 164-173
```python
    get_s3_resource().Object(
        f"{bucket_name}",
        f"{key}",
    ).put(
        Body=gzip.compress(body.getvalue().encode()),
        ContentEncoding="gzip",
        ContentType="application/json",
    )
    print(f"Done! Finish writing document to S3 {bucket_name}/{key} ")
```
- **EN**: This chunk continues `upload_to_s3` and expands its internal control flow or data movement. Structured-data parsing keeps repository metadata or intermediate reports in machine-readable form.
- **CN**: 这一段延续了 `upload_to_s3`，进一步展开其内部控制流或数据流转。 结构化数据解析使仓库元数据或中间报告保持机器可读形式。

### Lines 174-191
```python

def read_from_s3(
    bucket_name: str,
    key: str,
) -> list[dict[str, Any]]:
    print(f"Reading from s3://{bucket_name}/{key}")
    body = (
        get_s3_resource()
        .Object(
            f"{bucket_name}",
            f"{key}",
        )
        .get()["Body"]
        .read()
    )
    results = gzip.decompress(body).decode().split("\n")
    return [json.loads(result) for result in results if result]
```
- **EN**: This chunk defines `read_from_s3`, which parses or loads structured input into tool-friendly data structures. Structured-data parsing keeps repository metadata or intermediate reports in machine-readable form. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段定义了 `read_from_s3`，其作用是把结构化输入解析或加载为工具可处理的数据结构。 结构化数据解析使仓库元数据或中间报告保持机器可读形式。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 192-201
```python

def remove_nan_inf(old: Any) -> Any:
    # Casta NaN, inf, -inf to string from float since json.dumps outputs invalid
    # json with them
    def _helper(o: Any) -> Any:
        if isinstance(o, float) and (math.isinf(o) or math.isnan(o)):
            return str(o)
        if isinstance(o, list):
            return [_helper(v) for v in o]
        if isinstance(o, dict):
```
- **EN**: This chunk defines `_helper`, which implements a focused step inside the ci statistics pipeline. Structured-data parsing keeps repository metadata or intermediate reports in machine-readable form. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段定义了 `_helper`，其作用是实现CI 统计流水线中的一个关键步骤。 结构化数据解析使仓库元数据或中间报告保持机器可读形式。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 202-210
```python
            return {_helper(k): _helper(v) for k, v in o.items()}
        if isinstance(o, tuple):
            return tuple(_helper(v) for v in o)
        return o

    return _helper(old)


def upload_workflow_stats_to_s3(
```
- **EN**: This chunk defines `upload_workflow_stats_to_s3`, which parses or loads structured input into tool-friendly data structures. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段定义了 `upload_workflow_stats_to_s3`，其作用是把结构化输入解析或加载为工具可处理的数据结构。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 211-219
```python
    workflow_run_id: int,
    workflow_run_attempt: int,
    collection: str,
    docs: list[dict[str, Any]],
) -> None:
    bucket_name = "ossci-raw-job-status"
    key = f"{collection}/{workflow_run_id}/{workflow_run_attempt}"
    upload_to_s3(bucket_name, key, docs)
```
- **EN**: This chunk continues `upload_workflow_stats_to_s3` and expands its internal control flow or data movement.
- **CN**: 这一段延续了 `upload_workflow_stats_to_s3`，进一步展开其内部控制流或数据流转。

### Lines 220-235
```python

def upload_file_to_s3(
    file_name: str,
    bucket: str,
    key: str,
) -> None:
    """
    Upload a local file to S3
    """
    print(f"Upload {file_name} to s3://{bucket}/{key}")
    boto3.client("s3").upload_file(
        file_name,
        bucket,
        key,
    )
```
- **EN**: This docstring block explains the tool entry point, generation contract, or usage expectations before executable logic begins. This chunk defines `upload_file_to_s3`, which parses or loads structured input into tool-friendly data structures.
- **CN**: 这一段文档字符串在可执行逻辑开始前说明了工具入口、生成约定或使用方式。 这一段定义了 `upload_file_to_s3`，其作用是把结构化输入解析或加载为工具可处理的数据结构。

### Lines 236-246
```python

def unzip(p: Path) -> Path:
    """Unzip the provided zipfile to a similarly-named directory.

    Returns None if `p` is not a zipfile.

    Looks like: /tmp/test-reports.zip -> /tmp/unzipped-test-reports/

    Returns the path to the unzipped directory.
    """
    if not p.is_file():
```
- **EN**: This docstring block explains the tool entry point, generation contract, or usage expectations before executable logic begins. This chunk defines `unzip`, which implements a focused step inside the ci statistics pipeline. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段文档字符串在可执行逻辑开始前说明了工具入口、生成约定或使用方式。 这一段定义了 `unzip`，其作用是实现CI 统计流水线中的一个关键步骤。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 247-255
```python
        raise AssertionError(f"{p} is not a file")
    unzipped_dir = p.with_name("unzipped-" + p.stem)
    print(f"Extracting {p} to {unzipped_dir}")

    with zipfile.ZipFile(p, "r") as zip:
        zip.extractall(unzipped_dir)

    return unzipped_dir
```
- **EN**: This chunk continues `unzip` and expands its internal control flow or data movement. Assertions and explicit failures stop invalid states from propagating deeper into the workflow. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段延续了 `unzip`，进一步展开其内部控制流或数据流转。 断言与显式失败逻辑可阻止非法状态继续向工作流更深层传播。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 256-267
```python

def is_rerun_disabled_tests(
    report: Path,
    workflow_run_id: int,
    workflow_run_attempt: int,
    tests: dict[str, dict[str, int]],
) -> bool:
    """
    Check if the test report is coming from rerun_disabled_tests workflow where
    each test is run multiple times
    """
    if all(
```
- **EN**: This docstring block explains the tool entry point, generation contract, or usage expectations before executable logic begins. This chunk defines `is_rerun_disabled_tests`, which orchestrates command-line execution and forwards parsed arguments into the core logic. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段文档字符串在可执行逻辑开始前说明了工具入口、生成约定或使用方式。 这一段定义了 `is_rerun_disabled_tests`，其作用是编排命令行执行，并把解析后的参数转入核心逻辑。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 268-276
```python
        t.get("num_green", 0) + t.get("num_red", 0) > MAX_RETRY_IN_NON_DISABLED_MODE
        for t in tests.values()
    ):
        return True
    job_id = get_job_id(report)
    job_name = get_job_name(job_id, workflow_run_id, workflow_run_attempt)
    return job_name is not None and "rerun_disabled_tests" in job_name
```
- **EN**: This chunk continues `is_rerun_disabled_tests` and expands its internal control flow or data movement. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段延续了 `is_rerun_disabled_tests`，进一步展开其内部控制流或数据流转。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 277-287
```python
def get_job_id(report: Path) -> int | None:
    # [Job id in artifacts]
    # Retrieve the job id from the report path. In our GHA workflows, we append
    # the job id to the end of the report name, so `report` looks like:
    #     unzipped-test-reports-foo_5596745227/test/test-reports/foo/TEST-foo.xml
    # and we want to get `5596745227` out of it.
    try:
        return int(report.parts[0].rpartition("_")[2])
    except ValueError:
        return None
```
- **EN**: This chunk defines `get_job_id`, which implements a focused step inside the ci statistics pipeline. Exception-handling paths convert tool failures into clearer control flow or user-facing diagnostics. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段定义了 `get_job_id`，其作用是实现CI 统计流水线中的一个关键步骤。 异常处理路径把工具失败转换为更清晰的控制流或面向用户的诊断信息。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 288-296
```python

@lru_cache
def get_job_name(
    id: int | None, workflow_id: int | None, workflow_run_attempt: int | None
) -> str | None:
    if id is None:
        return None
    try:
        if workflow_id is None:
```
- **EN**: This chunk defines `get_job_name`, which implements a focused step inside the ci statistics pipeline. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. Exception-handling paths convert tool failures into clearer control flow or user-facing diagnostics. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段定义了 `get_job_name`，其作用是实现CI 统计流水线中的一个关键步骤。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 异常处理路径把工具失败转换为更清晰的控制流或面向用户的诊断信息。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 297-305
```python
            response = requests.get(
                f"{PYTORCH_REPO}/actions/jobs/{id}",
                headers=_get_request_headers(),
            )
            if response.status_code != 200:
                return None
            return cast(str, response.json()["name"])
        else:
```
- **EN**: This chunk continues `get_job_name` and expands its internal control flow or data movement. Structured-data parsing keeps repository metadata or intermediate reports in machine-readable form. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段延续了 `get_job_name`，进一步展开其内部控制流或数据流转。 结构化数据解析使仓库元数据或中间报告保持机器可读形式。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 306-317
```python
            @lru_cache
            def _get_jobs(workflow_id: int) -> dict[int, str]:
                jobs: dict[int, str] = {}
                # Paginate
                page = 1
                while True:
                    response = requests.get(
                        f"{PYTORCH_REPO}/actions/runs/{workflow_id}/attempts/{workflow_run_attempt}/jobs",
                        headers=_get_request_headers(),
                        params={"page": page, "per_page": 100},
                    )
                    if response.status_code != 200:
```
- **EN**: This chunk defines `_get_jobs`, which implements a focused step inside the ci statistics pipeline. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段定义了 `_get_jobs`，其作用是实现CI 统计流水线中的一个关键步骤。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 318-329
```python
                        return jobs
                    for job in response.json()["jobs"]:
                        jobs[job["id"]] = job["name"]
                    if "next" not in response.links:
                        break
                    page += 1
                return jobs

            jobs = _get_jobs(workflow_id)
            return jobs[id]
    except Exception:
        return None
```
- **EN**: This chunk continues `_get_jobs` and expands its internal control flow or data movement. Structured-data parsing keeps repository metadata or intermediate reports in machine-readable form. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. Exception-handling paths convert tool failures into clearer control flow or user-facing diagnostics. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段延续了 `_get_jobs`，进一步展开其内部控制流或数据流转。 结构化数据解析使仓库元数据或中间报告保持机器可读形式。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 异常处理路径把工具失败转换为更清晰的控制流或面向用户的诊断信息。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

## Key Concepts / 关键概念

- **CI statistics**
  - EN: This file belongs to the ci statistics layer and should be read in the context of neighboring scripts in the same subtree.
  - CN: 该文件属于CI 统计层，应结合同一子目录中的相邻脚本一起理解。
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
- **PYTORCH_REPO**
  - EN: `PYTORCH_REPO` is one of the main local symbols exposed or implemented here.
  - CN: `PYTORCH_REPO` 是此处暴露或实现的主要局部符号之一。
## Dependencies / 依赖关系

- **Python standard library / Python 标准库**: `__future__`, `gzip`, `io`, `json`, `math`, `os`, `time`, `zipfile`, `functools`, `pathlib`, `typing`, `collections.abc`
- **External packages / 外部依赖包**: `boto3  # type: ignore[import]`, `requests`
- **Primary symbols in this file / 本文件核心符号**: `PYTORCH_REPO`, `get_s3_resource`, `GHA_ARTIFACTS_BUCKET`, `MAX_RETRY_IN_NON_DISABLED_MODE`, `_get_request_headers`, `_get_artifact_urls`, `_download_artifact`, `download_s3_artifacts`, `download_gha_artifacts`, `upload_to_dynamodb`, `upload_to_s3`, `read_from_s3`
