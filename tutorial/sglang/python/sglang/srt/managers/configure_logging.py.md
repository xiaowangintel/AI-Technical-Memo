# configure_logging.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/managers/configure_logging.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module implements configure logging logic for runtime managers and coordination components. It exposes the classes, functions, and helpers that keep this part of the serving stack working. / 该模块实现与 configure logging 相关的逻辑，并服务于 运行时管理与协调组件。它提供支撑这一服务链路所需的类、函数与辅助流程。

## Line-by-Line Analysis / 逐行分析

### Lines 1-14: Provide supporting module logic / 提供辅助模块逻辑
```python
"""
Copyright 2023-2025 SGLang Team
Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
"""
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 16-21: Provide supporting module logic / 提供辅助模块逻辑
```python
"""
Configure the logging settings of a server.

Usage:
python3 -m sglang.srt.managers.configure_logging --url http://localhost:30000
"""
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 23-23: Import runtime dependencies / 导入运行时依赖
```python
import argparse
```
**EN:** This block gathers the standard-library, third-party, and local runtime modules that the rest of the file relies on.
**CN:** 该代码块汇集标准库、第三方库以及本地运行时模块，供后续实现复用。

### Lines 25-25: Import runtime dependencies / 导入运行时依赖
```python
import requests
```
**EN:** This block gathers the standard-library, third-party, and local runtime modules that the rest of the file relies on.
**CN:** 该代码块汇集标准库、第三方库以及本地运行时模块，供后续实现复用。

### Lines 27-62: Run the module as a script / 作为脚本运行模块
```python
if __name__ == "__main__":
    parser = argparse.ArgumentParser()
    parser.add_argument("--url", type=str, default="http://localhost:30000")
    parser.add_argument("--log-requests", action="store_true")
    parser.add_argument("--log-requests-level", type=int, default=3)
    parser.add_argument(
        "--dump-requests-folder", type=str, default="/tmp/sglang_request_dump"
    )
    parser.add_argument("--dump-requests-threshold", type=int, default=1000)
    parser.add_argument(
        "--dump-requests-exclude-meta-keys",
        type=str,
        default=None,
        help=(
            "Comma-separated meta_info keys to strip from each dumped request "
            "(e.g. 'routed_experts,hidden_states'). Pass an empty string to "
            "keep all keys. If not set, the server default is used."
        ),
    )
    args = parser.parse_args()

    payload = {
        "log_requests": args.log_requests,
        "log_requests_level": args.log_requests_level,  # Log full requests
        "dump_requests_folder": args.dump_requests_folder,
        "dump_requests_threshold": args.dump_requests_threshold,
    }
    if args.dump_requests_exclude_meta_keys is not None:
        payload["dump_requests_exclude_meta_keys"] = [
            k.strip()
            for k in args.dump_requests_exclude_meta_keys.split(",")
            if k.strip()
        ]

    response = requests.post(args.url + "/configure_logging", json=payload)
    assert response.status_code == 200
```
**EN:** This block provides the executable entry path used when the module is launched directly.
**CN:** 该代码块提供了模块被直接运行时使用的执行入口。

## Key Concepts / 关键概念
- **Domain focus / 领域焦点**: configure logging / configure logging
- **Control style / 控制方式**: mostly synchronous orchestration and helper composition / 以同步编排与辅助逻辑组合为主

## Dependencies / 依赖关系
- **Standard Library / 标准库**: argparse
- **Third-party / 第三方库**: requests
- **Local Modules / 本地模块**: None / 无
