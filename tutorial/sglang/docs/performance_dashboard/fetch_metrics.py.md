# fetch_metrics.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `docs/performance_dashboard/fetch_metrics.py`
- **Repository**: `sgl-project/sglang`
- **Purpose**: Fetch and process SGLang nightly test metrics from GitHub Actions artifacts. This script fetches consolidated metrics from GitHub Actions workflow runs and outputs them as JSON for the performance dashboard. / 该 Python 模块用于实现与 Fetch 指标 相关的文档工具链逻辑，包括配置、数据处理、自动化脚本或发布辅助功能。

## Structure Overview / 结构概览
- **EN:** The module contains 10 import statements, 0 class definitions, and 9 function definitions.
- **CN:** 该模块包含 10 处导入、0 个类定义以及 9 个函数定义。

## Imports / 导入依赖
- `argparse`
- `datetime`
- `io`
- `json`
- `os`
- `pathlib`
- `requests`
- `sys`
- `typing`
- `zipfile`

## Symbols / 主要符号
- **`get_github_token()`** — Get GitHub token from environment or gh CLI. / 该函数定义了 `get_github_token()`，封装了脚本中的一个关键处理步骤。
- **`get_headers(token)`** — Get request headers with optional authentication. / 该函数定义了 `get_headers(token)`，封装了脚本中的一个关键处理步骤。
- **`fetch_workflow_runs(token, days, event)`** — Fetch completed workflow runs from GitHub Actions. / 该函数定义了 `fetch_workflow_runs(token, days, event)`，封装了脚本中的一个关键处理步骤。
- **`fetch_run_artifacts(token, run_id)`** — Fetch artifacts for a specific workflow run. / 该函数定义了 `fetch_run_artifacts(token, run_id)`，封装了脚本中的一个关键处理步骤。
- **`download_artifact(token, artifact_id)`** — Download an artifact by ID. / 该函数定义了 `download_artifact(token, artifact_id)`，封装了脚本中的一个关键处理步骤。
- **`extract_metrics_from_zip(zip_content)`** — Extract metrics JSON from a zip file. / 该函数定义了 `extract_metrics_from_zip(zip_content)`，封装了脚本中的一个关键处理步骤。
- **`fetch_metrics_for_run(token, run)`** — Fetch metrics for a single workflow run. / 该函数定义了 `fetch_metrics_for_run(token, run)`，封装了脚本中的一个关键处理步骤。
- **`fetch_single_run(token, run_id)`** — Fetch metrics for a single run by ID. / 该函数定义了 `fetch_single_run(token, run_id)`，封装了脚本中的一个关键处理步骤。
- **`main()`** — This function defines `main()` and is used to orchestrate the overall script flow. / 该函数定义了 `main()`，封装了脚本中的一个关键处理步骤。

## Line-by-Line Analysis / 逐行分析

### Lines 2-12: Expr
**EN:** This block provides module-level documentation or contextual notes.
**CN:** 这部分提供模块级说明文字，用于解释脚本用途、输入输出或使用方式。

### Lines 14-14: Import
**EN:** These lines import dependencies such as argparse to support the module.
**CN:** 这些代码导入模块依赖，用于支撑后续的配置、数据处理、网络访问或命令行逻辑。

### Lines 15-15: Import
**EN:** These lines import dependencies such as io to support the module.
**CN:** 这些代码导入模块依赖，用于支撑后续的配置、数据处理、网络访问或命令行逻辑。

### Lines 16-16: Import
**EN:** These lines import dependencies such as json to support the module.
**CN:** 这些代码导入模块依赖，用于支撑后续的配置、数据处理、网络访问或命令行逻辑。

### Lines 17-17: Import
**EN:** These lines import dependencies such as os to support the module.
**CN:** 这些代码导入模块依赖，用于支撑后续的配置、数据处理、网络访问或命令行逻辑。

### Lines 18-18: Import
**EN:** These lines import dependencies such as sys to support the module.
**CN:** 这些代码导入模块依赖，用于支撑后续的配置、数据处理、网络访问或命令行逻辑。

### Lines 19-19: Import
**EN:** These lines import dependencies such as zipfile to support the module.
**CN:** 这些代码导入模块依赖，用于支撑后续的配置、数据处理、网络访问或命令行逻辑。

### Lines 20-20: ImportFrom
**EN:** These lines import dependencies such as datetime to support the module.
**CN:** 这些代码导入模块依赖，用于支撑后续的配置、数据处理、网络访问或命令行逻辑。

### Lines 21-21: ImportFrom
**EN:** These lines import dependencies such as pathlib to support the module.
**CN:** 这些代码导入模块依赖，用于支撑后续的配置、数据处理、网络访问或命令行逻辑。

### Lines 22-22: ImportFrom
**EN:** These lines import dependencies such as typing to support the module.
**CN:** 这些代码导入模块依赖，用于支撑后续的配置、数据处理、网络访问或命令行逻辑。

### Lines 24-24: Import
**EN:** These lines import dependencies such as requests to support the module.
**CN:** 这些代码导入模块依赖，用于支撑后续的配置、数据处理、网络访问或命令行逻辑。

### Lines 26-26: Assign
**EN:** These lines define module-level values such as GITHUB_REPO.
**CN:** 这些代码定义模块级常量或默认参数，供后续逻辑复用。

### Lines 27-27: Assign
**EN:** These lines define module-level values such as WORKFLOW_NAME.
**CN:** 这些代码定义模块级常量或默认参数，供后续逻辑复用。

### Lines 28-28: Assign
**EN:** These lines define module-level values such as ARTIFACT_PREFIX.
**CN:** 这些代码定义模块级常量或默认参数，供后续逻辑复用。

### Lines 31-52: get_github_token()
**EN:** This function defines `get_github_token()`. Get GitHub token from environment or gh CLI.
**CN:** 该函数定义了 `get_github_token()`，封装了脚本中的一个关键处理步骤。

### Lines 55-62: get_headers(token)
**EN:** This function defines `get_headers(token)`. Get request headers with optional authentication.
**CN:** 该函数定义了 `get_headers(token)`，封装了脚本中的一个关键处理步骤。

### Lines 65-94: fetch_workflow_runs(token, days, event)
**EN:** This function defines `fetch_workflow_runs(token, days, event)`. Fetch completed workflow runs from GitHub Actions.
**CN:** 该函数定义了 `fetch_workflow_runs(token, days, event)`，封装了脚本中的一个关键处理步骤。

### Lines 97-104: fetch_run_artifacts(token, run_id)
**EN:** This function defines `fetch_run_artifacts(token, run_id)`. Fetch artifacts for a specific workflow run.
**CN:** 该函数定义了 `fetch_run_artifacts(token, run_id)`，封装了脚本中的一个关键处理步骤。

### Lines 107-125: download_artifact(token, artifact_id)
**EN:** This function defines `download_artifact(token, artifact_id)`. Download an artifact by ID.
**CN:** 该函数定义了 `download_artifact(token, artifact_id)`，封装了脚本中的一个关键处理步骤。

### Lines 128-141: extract_metrics_from_zip(zip_content)
**EN:** This function defines `extract_metrics_from_zip(zip_content)`. Extract metrics JSON from a zip file.
**CN:** 该函数定义了 `extract_metrics_from_zip(zip_content)`，封装了脚本中的一个关键处理步骤。

### Lines 144-181: fetch_metrics_for_run(token, run)
**EN:** This function defines `fetch_metrics_for_run(token, run)`. Fetch metrics for a single workflow run.
**CN:** 该函数定义了 `fetch_metrics_for_run(token, run)`，封装了脚本中的一个关键处理步骤。

### Lines 184-192: fetch_single_run(token, run_id)
**EN:** This function defines `fetch_single_run(token, run_id)`. Fetch metrics for a single run by ID.
**CN:** 该函数定义了 `fetch_single_run(token, run_id)`，封装了脚本中的一个关键处理步骤。

### Lines 195-268: main()
**EN:** This function defines `main()` and is used to orchestrate the overall script flow.
**CN:** 该函数定义了 `main()`，封装了脚本中的一个关键处理步骤。

### Lines 271-272: If
**EN:** This conditional block handles branching logic or the script entry point.
**CN:** 该条件分支负责处理不同执行路径，或作为脚本主入口控制流程。

## Dependencies / 依赖关系
- `argparse`
- `datetime`
- `io`
- `json`
- `os`
- `pathlib`
- `requests`
- `sys`
- `typing`
- `zipfile`
