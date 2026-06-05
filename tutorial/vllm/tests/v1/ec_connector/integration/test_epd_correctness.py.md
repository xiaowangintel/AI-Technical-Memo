# test_epd_correctness.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/v1/ec_connector/integration/test_epd_correctness.py`
- **Repository**: vllm-project/vllm
- **Purpose**: EPD Correctness Test Tests that EPD (Encoder-Prefill-Decode) disaggregation produces the same outputs as a baseline single instance. / 该文件的文档字符串表明其用途：`epd correctness test tests that epd (encoder-prefill-decode) disaggregation produces the same outputs as a baseline single instance`。

## Line-by-Line Analysis / 逐行分析
### Module header / 模块头 (lines 1-2)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project
```
**EN:** File header with 2 non-empty line(s) of license or metadata comments.
**CN:** 文件头包含 2 行非空的许可证或元数据注释。

### Module docstring / 模块说明 (lines 3-23)
```python
"""
EPD Correctness Test

Tests that EPD (Encoder-Prefill-Decode) disaggregation produces the same
outputs as a baseline single instance.

Usage:
    # Baseline mode (saves outputs):
    python test_epd_correctness.py \
        --service_url http://localhost:8000 \
        --model_name Qwen/Qwen2.5-VL-3B-Instruct \
        --mode baseline \
        --baseline_file .vllm_epd_baseline.txt

    # Disagg mode (compares outputs):
    python test_epd_correctness.py \
        --service_url http://localhost:8000 \
        --model_name Qwen/Qwen2.5-VL-3B-Instruct \
        --mode disagg \
        --baseline_file .vllm_epd_baseline.txt
"""
```
**EN:** Module docstring that declares the scope of the file: EPD Correctness Test Tests that EPD (Encoder-Prefill-Decode) disaggregation produces the same outputs as a baseline single instance.
**CN:** 模块文档字符串直接说明了文件范围：`epd correctness test tests that epd (encoder-prefill-decode) disaggregation produces the same outputs as a baseline single instance`。

### Imports and setup / 导入与设置 (lines 25-34)
```python
import argparse
import json
import os
import time

import openai
import requests

from vllm.assets.image import ImageAsset
from vllm.multimodal.utils import encode_image_url
```
**EN:** Imports the libraries needed to build the test harness. External dependencies include `openai, requests`. vLLM modules under test include `vllm.assets.image, vllm.multimodal.utils`.
**CN:** 该代码块导入构建测试环境所需的库。 外部依赖包括 `openai, requests`。 被测试的 vLLM 模块包括 `vllm.assets.image, vllm.multimodal.utils`。

### Module state / 模块级状态 (lines 36-93)
```python
MAX_OUTPUT_LEN = 256

# Sample prompts with multimodal content
image_1 = ImageAsset("stop_sign").pil_image.resize((1280, 720))
image_2 = ImageAsset("cherry_blossom").pil_image.resize((1280, 720))
image_local_path = f"{os.path.dirname(os.path.abspath(__file__))}/hato.jpg"
SAMPLE_PROMPTS_MM: list[dict] = [
    {
        "messages": [
            {
                "role": "user",
                "content": [
                    {
                        "type": "image_url",
                        "image_url": {"url": encode_image_url(image_1)},
                    },
# ... excerpt omitted for brevity ...
            {"role": "user", "content": "Explain quantum computing in simple terms."}
        ],
        "description": "Text-only explanation request",
    },
]
```
**EN:** Defines module-level constants, feature gates, or shared state. Representative names: `MAX_OUTPUT_LEN, image_1, image_2, image_local_path, SAMPLE_PROMPTS_MM, SAMPLE_PROMPTS_TEXT`. Shared setup calls include `pil_image.resize, path.dirname, ImageAsset, path.abspath, encode_image_url`.
**CN:** 定义模块级常量、特性开关或共享状态。 代表性名称：`MAX_OUTPUT_LEN, image_1, image_2, image_local_path, SAMPLE_PROMPTS_MM, SAMPLE_PROMPTS_TEXT`。 共享初始化调用包括 `pil_image.resize, path.dirname, ImageAsset, path.abspath, encode_image_url`。

### check_vllm_server (lines 96-121)
```python
def check_vllm_server(url: str, timeout=5, retries=10) -> bool:
    """Check if the vLLM server is ready.

    Args:
        url: The URL to check (usually /health or /healthcheck endpoint)
        timeout: Timeout in seconds for each request
        retries: Number of retries if the server is not ready

    Returns:
        True if the server is ready, False otherwise
    """
    for attempt in range(retries):
        try:
            response = requests.get(url, timeout=timeout)
            if response.status_code == 200:
                print(f"Server is ready at {url}")
                return True
            else:
                print(
                    f"Attempt {attempt + 1}/{retries}: Server returned "
                    f"status code {response.status_code}"
                )
        except requests.exceptions.RequestException as e:
            print(f"Attempt {attempt + 1}/{retries}: Error connecting: {e}")
        time.sleep(2)  # Wait before retrying
    return False
```
**EN:** Helper function `check_vllm_server` encapsulates reusable logic for `check vllm server`. Inputs: `url, timeout, retries`. Key calls include `range, time.sleep, requests.get, print`.
**CN:** 辅助函数 `check_vllm_server` 封装了与 `check vllm server` 相关的可复用逻辑。 输入参数：`url, timeout, retries`。 关键调用包括 `range, time.sleep, requests.get, print`。

### run_chat_completion (lines 124-151)
```python
def run_chat_completion(
    base_url: str,
    model_name: str,
    messages: list,
    max_tokens: int = MAX_OUTPUT_LEN,
) -> str:
    """Run a chat completion request.

    Args:
        base_url: Base URL of the vLLM server
        model_name: Name of the model
        messages: Messages for chat completion
        max_tokens: Maximum tokens to generate

    Returns:
        Generated text content
    """
    client = openai.OpenAI(api_key="EMPTY", base_url=base_url)

    completion = client.chat.completions.create(
        model=model_name,
        messages=messages,
        max_tokens=max_tokens,
        temperature=0.0,
        seed=42,
    )

    return completion.choices[0].message.content
```
**EN:** Helper function `run_chat_completion` encapsulates reusable logic for `run chat completion`. Inputs: `base_url, model_name, messages, max_tokens`. Key calls include `openai.OpenAI, completions.create`.
**CN:** 辅助函数 `run_chat_completion` 封装了与 `run chat completion` 相关的可复用逻辑。 输入参数：`base_url, model_name, messages, max_tokens`。 关键调用包括 `openai.OpenAI, completions.create`。

### main (lines 154-296)
```python
def main():
    """Main test function."""
    parser = argparse.ArgumentParser(
        description="EPD correctness test - compare disagg vs baseline"
    )

    parser.add_argument(
        "--service_url",
        type=str,
        required=True,
        help="The vLLM service URL (e.g., http://localhost:8000)",
        "--model_name",
        help="Model name",
    # ... excerpt omitted for brevity ...
        assert isinstance(baseline_outputs, dict), "Baseline outputs should be a dict"
        assert len(baseline_outputs) == len(output_strs), (
            assert key in output_strs, f"{key} not in disagg outputs"
                print(f"  Disagg:   {disagg_output}")
                all_match = False
        assert all_match, "❌❌Disagg outputs do not match baseline!❌❌"
        if all_match:
            print("\n✅ All outputs match! Test PASSED")
```
**EN:** Helper function `main` encapsulates reusable logic for `main`. Key calls include `argparse.ArgumentParser, parser.add_argument, parser.parse_args, print, enumerate, check_vllm_server`. It includes 4 internal assertion(s) to guard assumptions.
**CN:** 辅助函数 `main` 封装了与 `main` 相关的可复用逻辑。 关键调用包括 `argparse.ArgumentParser, parser.add_argument, parser.parse_args, print, enumerate, check_vllm_server`。 其中包含 4 个内部断言，用于保护前置假设。

### Module state / 模块级状态 (lines 299-300)
```python
if __name__ == "__main__":
    main()
```
**EN:** Defines module-level constants, feature gates, or shared state. Conditional logic is used to adapt the suite to optional dependencies or platform differences. Shared setup calls include `main`.
**CN:** 定义模块级常量、特性开关或共享状态。 条件逻辑用于适配可选依赖或平台差异。 共享初始化调用包括 `main`。

## Key Concepts / 关键概念
- **EN:** Connector contracts and transfer paths
- **CN:** 连接器契约与传输路径
- **EN:** Multimodal input bookkeeping
- **CN:** 多模态输入管理

## Dependencies / 依赖关系
- **EN:** External libraries: `openai, requests`.
- **CN:** 外部库：`openai, requests`。
- **EN:** vLLM modules under test: `vllm.assets.image, vllm.multimodal.utils`.
- **CN:** 被测试的 vLLM 模块：`vllm.assets.image, vllm.multimodal.utils`。
- **EN:** Standard-library support: `argparse, json, os, time`.
- **CN:** 标准库支持：`argparse, json, os, time`。
