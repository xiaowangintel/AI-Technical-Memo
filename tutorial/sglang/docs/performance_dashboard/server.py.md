# server.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `docs/performance_dashboard/server.py`
- **Repository**: `sgl-project/sglang`
- **Purpose**: Simple development server for the SGLang Performance Dashboard. Serves the static HTML/JS files 2. / 该 Python 模块用于实现与 服务端 相关的文档工具链逻辑，包括配置、数据处理、自动化脚本或发布辅助功能。

## Structure Overview / 结构概览
- **EN:** The module contains 16 import statements, 1 class definitions, and 8 function definitions.
- **CN:** 该模块包含 16 处导入、1 个类定义以及 8 个函数定义。

## Imports / 导入依赖
- `argparse`
- `datetime`
- `hashlib`
- `hmac`
- `http.server`
- `io`
- `json`
- `os`
- `pathlib`
- `requests`
- `secrets`
- `socketserver`
- `threading`
- `time`
- `urllib.parse`
- `zipfile`

## Symbols / 主要符号
- **`DashboardHandler`** — HTTP request handler for the dashboard. / 该类定义了 `DashboardHandler`，用于组织相关状态与行为。
- **`hash_password(password)`** — Hash a password using SHA-256 for constant-time comparison. / 该函数定义了 `hash_password(password)`，封装了脚本中的一个关键处理步骤。
- **`create_auth_token()`** — Create a new session token. / 该函数定义了 `create_auth_token()`，封装了脚本中的一个关键处理步骤。
- **`verify_auth_token(token)`** — Verify a session token is valid and not expired. / 该函数定义了 `verify_auth_token(token)`，封装了脚本中的一个关键处理步骤。
- **`get_github_token()`** — Get GitHub token from environment or gh CLI. / 该函数定义了 `get_github_token()`，封装了脚本中的一个关键处理步骤。
- **`fetch_metrics_from_github(days)`** — Fetch metrics from GitHub Actions artifacts. / 该函数定义了 `fetch_metrics_from_github(days)`，封装了脚本中的一个关键处理步骤。
- **`update_cache_async()`** — Update the metrics cache in background with thread safety. / 该函数定义了 `update_cache_async()`，封装了脚本中的一个关键处理步骤。
- **`start_periodic_refresh(interval_hours)`** — Start a background thread that refreshes the cache periodically. / 该函数定义了 `start_periodic_refresh(interval_hours)`，封装了脚本中的一个关键处理步骤。
- **`main()`** — This function defines `main()` and is used to orchestrate the overall script flow. / 该函数定义了 `main()`，封装了脚本中的一个关键处理步骤。

## Line-by-Line Analysis / 逐行分析

### Lines 2-18: Expr
**EN:** This block provides module-level documentation or contextual notes.
**CN:** 这部分提供模块级说明文字，用于解释脚本用途、输入输出或使用方式。

### Lines 20-20: Import
**EN:** These lines import dependencies such as argparse to support the module.
**CN:** 这些代码导入模块依赖，用于支撑后续的配置、数据处理、网络访问或命令行逻辑。

### Lines 21-21: Import
**EN:** These lines import dependencies such as hashlib to support the module.
**CN:** 这些代码导入模块依赖，用于支撑后续的配置、数据处理、网络访问或命令行逻辑。

### Lines 22-22: Import
**EN:** These lines import dependencies such as hmac to support the module.
**CN:** 这些代码导入模块依赖，用于支撑后续的配置、数据处理、网络访问或命令行逻辑。

### Lines 23-23: Import
**EN:** These lines import dependencies such as http.server to support the module.
**CN:** 这些代码导入模块依赖，用于支撑后续的配置、数据处理、网络访问或命令行逻辑。

### Lines 24-24: Import
**EN:** These lines import dependencies such as io to support the module.
**CN:** 这些代码导入模块依赖，用于支撑后续的配置、数据处理、网络访问或命令行逻辑。

### Lines 25-25: Import
**EN:** These lines import dependencies such as json to support the module.
**CN:** 这些代码导入模块依赖，用于支撑后续的配置、数据处理、网络访问或命令行逻辑。

### Lines 26-26: Import
**EN:** These lines import dependencies such as os to support the module.
**CN:** 这些代码导入模块依赖，用于支撑后续的配置、数据处理、网络访问或命令行逻辑。

### Lines 27-27: Import
**EN:** These lines import dependencies such as secrets to support the module.
**CN:** 这些代码导入模块依赖，用于支撑后续的配置、数据处理、网络访问或命令行逻辑。

### Lines 28-28: Import
**EN:** These lines import dependencies such as socketserver to support the module.
**CN:** 这些代码导入模块依赖，用于支撑后续的配置、数据处理、网络访问或命令行逻辑。

### Lines 29-29: Import
**EN:** These lines import dependencies such as threading to support the module.
**CN:** 这些代码导入模块依赖，用于支撑后续的配置、数据处理、网络访问或命令行逻辑。

### Lines 30-30: Import
**EN:** These lines import dependencies such as time to support the module.
**CN:** 这些代码导入模块依赖，用于支撑后续的配置、数据处理、网络访问或命令行逻辑。

### Lines 31-31: Import
**EN:** These lines import dependencies such as zipfile to support the module.
**CN:** 这些代码导入模块依赖，用于支撑后续的配置、数据处理、网络访问或命令行逻辑。

### Lines 32-32: ImportFrom
**EN:** These lines import dependencies such as datetime to support the module.
**CN:** 这些代码导入模块依赖，用于支撑后续的配置、数据处理、网络访问或命令行逻辑。

### Lines 33-33: ImportFrom
**EN:** These lines import dependencies such as pathlib to support the module.
**CN:** 这些代码导入模块依赖，用于支撑后续的配置、数据处理、网络访问或命令行逻辑。

### Lines 34-34: ImportFrom
**EN:** These lines import dependencies such as urllib.parse to support the module.
**CN:** 这些代码导入模块依赖，用于支撑后续的配置、数据处理、网络访问或命令行逻辑。

### Lines 36-36: Import
**EN:** These lines import dependencies such as requests to support the module.
**CN:** 这些代码导入模块依赖，用于支撑后续的配置、数据处理、网络访问或命令行逻辑。

### Lines 38-38: Assign
**EN:** These lines define module-level values such as GITHUB_REPO.
**CN:** 这些代码定义模块级常量或默认参数，供后续逻辑复用。

### Lines 39-39: Assign
**EN:** These lines define module-level values such as WORKFLOW_NAME.
**CN:** 这些代码定义模块级常量或默认参数，供后续逻辑复用。

### Lines 40-40: Assign
**EN:** These lines define module-level values such as ARTIFACT_PREFIX.
**CN:** 这些代码定义模块级常量或默认参数，供后续逻辑复用。

### Lines 43-43: Assign
**EN:** These lines define module-level values such as cache_lock.
**CN:** 这些代码定义模块级常量或默认参数，供后续逻辑复用。

### Lines 44-48: Assign
**EN:** These lines define module-level values such as metrics_cache.
**CN:** 这些代码定义模块级常量或默认参数，供后续逻辑复用。

### Lines 50-50: Assign
**EN:** These lines define module-level values such as CACHE_TTL.
**CN:** 这些代码定义模块级常量或默认参数，供后续逻辑复用。

### Lines 51-51: Assign
**EN:** These lines define module-level values such as REQUEST_TIMEOUT.
**CN:** 这些代码定义模块级常量或默认参数，供后续逻辑复用。

### Lines 54-59: Assign
**EN:** These lines define module-level values such as auth_config.
**CN:** 这些代码定义模块级常量或默认参数，供后续逻辑复用。

### Lines 60-60: Assign
**EN:** These lines define module-level values such as auth_lock.
**CN:** 这些代码定义模块级常量或默认参数，供后续逻辑复用。

### Lines 61-61: Assign
**EN:** These lines define module-level values such as AUTH_TOKEN_TTL.
**CN:** 这些代码定义模块级常量或默认参数，供后续逻辑复用。

### Lines 64-66: hash_password(password)
**EN:** This function defines `hash_password(password)`. Hash a password using SHA-256 for constant-time comparison.
**CN:** 该函数定义了 `hash_password(password)`，封装了脚本中的一个关键处理步骤。

### Lines 69-79: create_auth_token()
**EN:** This function defines `create_auth_token()`. Create a new session token.
**CN:** 该函数定义了 `create_auth_token()`，封装了脚本中的一个关键处理步骤。

### Lines 82-92: verify_auth_token(token)
**EN:** This function defines `verify_auth_token(token)`. Verify a session token is valid and not expired.
**CN:** 该函数定义了 `verify_auth_token(token)`，封装了脚本中的一个关键处理步骤。

### Lines 95-114: get_github_token()
**EN:** This function defines `get_github_token()`. Get GitHub token from environment or gh CLI.
**CN:** 该函数定义了 `get_github_token()`，封装了脚本中的一个关键处理步骤。

### Lines 117-216: fetch_metrics_from_github(days)
**EN:** This function defines `fetch_metrics_from_github(days)`. Fetch metrics from GitHub Actions artifacts.
**CN:** 该函数定义了 `fetch_metrics_from_github(days)`，封装了脚本中的一个关键处理步骤。

### Lines 219-234: update_cache_async()
**EN:** This function defines `update_cache_async()`. Update the metrics cache in background with thread safety.
**CN:** 该函数定义了 `update_cache_async()`，封装了脚本中的一个关键处理步骤。

### Lines 237-249: start_periodic_refresh(interval_hours)
**EN:** This function defines `start_periodic_refresh(interval_hours)`. Start a background thread that refreshes the cache periodically.
**CN:** 该函数定义了 `start_periodic_refresh(interval_hours)`，封装了脚本中的一个关键处理步骤。

### Lines 252-358: DashboardHandler
**EN:** This class defines `DashboardHandler`. HTTP request handler for the dashboard.
**CN:** 该类定义了 `DashboardHandler`，用于组织相关状态与行为。

### Lines 361-418: main()
**EN:** This function defines `main()` and is used to orchestrate the overall script flow.
**CN:** 该函数定义了 `main()`，封装了脚本中的一个关键处理步骤。

### Lines 421-422: If
**EN:** This conditional block handles branching logic or the script entry point.
**CN:** 该条件分支负责处理不同执行路径，或作为脚本主入口控制流程。

## Dependencies / 依赖关系
- `argparse`
- `datetime`
- `hashlib`
- `hmac`
- `http.server`
- `io`
- `json`
- `os`
- `pathlib`
- `requests`
- `secrets`
- `socketserver`
- `threading`
- `time`
- `urllib.parse`
- `zipfile`
