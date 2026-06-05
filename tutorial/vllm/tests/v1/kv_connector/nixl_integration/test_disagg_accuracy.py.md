# test_disagg_accuracy.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/v1/kv_connector/nixl_integration/test_disagg_accuracy.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Validates `disagg accuracy` behavior and regressions in the v1 stack. / 验证 v1 栈中 `disagg accuracy` 相关行为与回归。

## Line-by-Line Analysis / 逐行分析
### Module header / 模块头 (lines 1-2)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project
```
**EN:** File header with 2 non-empty line(s) of license or metadata comments.
**CN:** 文件头包含 2 行非空的许可证或元数据注释。

### Imports and setup / 导入与设置 (lines 3-9)
```python
import argparse
import json
import os
import time

import openai
import requests
```
**EN:** Imports the libraries needed to build the test harness. External dependencies include `openai, requests`.
**CN:** 该代码块导入构建测试环境所需的库。 外部依赖包括 `openai, requests`。

### Module state / 模块级状态 (lines 11-24)
```python
MAX_OUTPUT_LEN = 30

SAMPLE_PROMPTS = (
    "Red Hat is the best company in the world to work for because it works on "
    "open source software, which means that all the contributions are "
    "delivered to the community. As a result, when working on projects like "
    "vLLM we are able to meet many amazing people from various organizations "
    "like AMD, Google, NVIDIA, ",
    "We hold these truths to be self-evident, that all men are created equal, "
    "that they are endowed by their Creator with certain unalienable Rights, "
    "that among these are Life, Liberty and the pursuit of Happiness.--That "
    "to secure these rights, Governments are instituted among Men, deriving "
    "their just powers from the consent of the governed, ",
)
```
**EN:** Defines module-level constants, feature gates, or shared state. Representative names: `MAX_OUTPUT_LEN, SAMPLE_PROMPTS`.
**CN:** 定义模块级常量、特性开关或共享状态。 代表性名称：`MAX_OUTPUT_LEN, SAMPLE_PROMPTS`。

### check_vllm_server (lines 27-53)
```python
def check_vllm_server(url: str, timeout=5, retries=3) -> bool:
    """
    Checks if the vLLM server is ready by sending a GET request to the
    /health endpoint.

    Args:
        url (str): The base URL of the vLLM server.
        timeout (int): Timeout in seconds for the request.
        retries (int): Number of retries if the server is not ready.

    Returns:
        bool: True if the server is ready, False otherwise.
    """
    for attempt in range(retries):
        try:
            response = requests.get(url, timeout=timeout)
            if response.status_code == 200:
                return True
            else:
                print(
                    f"Attempt {attempt + 1}: Server returned status code "
                    "{response.status_code}"
                )
        except requests.exceptions.RequestException as e:
            print(f"Attempt {attempt + 1}: Error connecting to server: {e}")
        time.sleep(1)  # Wait before retrying
    return False
```
**EN:** Helper function `check_vllm_server` encapsulates reusable logic for `check vllm server`. Inputs: `url, timeout, retries`. Key calls include `range, time.sleep, requests.get, print`.
**CN:** 辅助函数 `check_vllm_server` 封装了与 `check vllm server` 相关的可复用逻辑。 输入参数：`url, timeout, retries`。 关键调用包括 `range, time.sleep, requests.get, print`。

### run_simple_prompt (lines 56-80)
```python
def run_simple_prompt(
    base_url: str, model_name: str, input_prompt: str, use_chat_endpoint: bool
) -> str:
    client = openai.OpenAI(api_key="EMPTY", base_url=base_url)
    if use_chat_endpoint:
        completion = client.chat.completions.create(
            model=model_name,
            messages=[
                {"role": "user", "content": [{"type": "text", "text": input_prompt}]}
            ],
            max_completion_tokens=MAX_OUTPUT_LEN,
            temperature=0.0,
            seed=42,
        )
        return completion.choices[0].message.content
    else:
        completion = client.completions.create(
            model=model_name,
            prompt=input_prompt,
            max_tokens=MAX_OUTPUT_LEN,
            temperature=0.0,
            seed=42,
        )

        return completion.choices[0].text
```
**EN:** Helper function `run_simple_prompt` encapsulates reusable logic for `run simple prompt`. Inputs: `base_url, model_name, input_prompt, use_chat_endpoint`. Key calls include `openai.OpenAI, completions.create`.
**CN:** 辅助函数 `run_simple_prompt` 封装了与 `run simple prompt` 相关的可复用逻辑。 输入参数：`base_url, model_name, input_prompt, use_chat_endpoint`。 关键调用包括 `openai.OpenAI, completions.create`。

### main (lines 83-176)
```python
def main():
    """
    This script demonstrates how to accept two optional string arguments
    ("service_url" and "file_name") from the command line, each with a
    default value of an empty string, using the argparse module.
    parser = argparse.ArgumentParser(description="vLLM client script")

    parser.add_argument(
        "--service_url",  # Name of the first argument
        type=str,
        required=True,
        help="The vLLM service URL.",
    )
        "--model_name",  # Name of the first argument
    # ... excerpt omitted for brevity ...
        assert isinstance(baseline_outputs, dict)
        assert len(baseline_outputs) == len(output_strs)
        for prompt, output in baseline_outputs.items():
            assert prompt in output_strs, f"{prompt} not included"
            assert output == output_strs[prompt], (
                f"baseline_output: {output} != PD output: {output_strs[prompt]}"
            )
```
**EN:** Helper function `main` encapsulates reusable logic for `main`. Key calls include `argparse.ArgumentParser, parser.add_argument, parser.parse_args, vars, dict, enumerate`. It includes 4 internal assertion(s) to guard assumptions.
**CN:** 辅助函数 `main` 封装了与 `main` 相关的可复用逻辑。 关键调用包括 `argparse.ArgumentParser, parser.add_argument, parser.parse_args, vars, dict, enumerate`。 其中包含 4 个内部断言，用于保护前置假设。

### Module state / 模块级状态 (lines 179-180)
```python
if __name__ == "__main__":
    main()
```
**EN:** Defines module-level constants, feature gates, or shared state. Conditional logic is used to adapt the suite to optional dependencies or platform differences. Shared setup calls include `main`.
**CN:** 定义模块级常量、特性开关或共享状态。 条件逻辑用于适配可选依赖或平台差异。 共享初始化调用包括 `main`。

## Key Concepts / 关键概念
- **EN:** Connector contracts and transfer paths
- **CN:** 连接器契约与传输路径

## Dependencies / 依赖关系
- **EN:** External libraries: `openai, requests`.
- **CN:** 外部库：`openai, requests`。
- **EN:** Standard-library support: `argparse, json, os, time`.
- **CN:** 标准库支持：`argparse, json, os, time`。
