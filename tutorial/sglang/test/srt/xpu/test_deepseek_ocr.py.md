# test_deepseek_ocr.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/srt/xpu/test_deepseek_ocr.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates deepseek ocr behavior in SGLang's srt / xpu area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 SRT / xpu 领域中与 deepseek ocr 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-3: supporting statements / 辅助语句
```python
"""
python3 -m unittest test_deepseek_ocr.py
"""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 5-19: module imports and dependencies / 模块导入与依赖
```python
import json
import os
import unittest
from pathlib import Path

import requests

from sglang.srt.utils import kill_process_tree
from sglang.srt.utils.hf_transformers import get_tokenizer
from sglang.test.test_utils import (
    DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
    DEFAULT_URL_FOR_TEST,
    CustomTestCase,
    popen_launch_server,
)
```
**EN:** This block imports the modules needed by the rest of the file, including `json`, `os`, `unittest`, `pathlib`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `json`, `os`, `unittest`, `pathlib`。

### Lines 22-22: class TestDeepSeekOCR declaration / 类 TestDeepSeekOCR 声明
```python
class TestDeepSeekOCR(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 23-47: setUpClass setup routine / setUpClass 初始化流程
```python
    @classmethod
    def setUpClass(cls):
        cls.model = "deepseek-ai/DeepSeek-OCR"
        cls.tokenizer = get_tokenizer(cls.model)
        cls.base_url = DEFAULT_URL_FOR_TEST
        cls.image_path = str(
            (Path(__file__).resolve().parents[3] / "examples/assets/example_image.png")
        )
        if not os.path.exists(cls.image_path):
            raise FileNotFoundError(f"Image not found: {cls.image_path}")
        cls.common_args = [
            "--device",
            "xpu",
            "--attention-backend",
            "intel_xpu",
        ]
        os.environ["SGLANG_USE_SGL_XPU"] = "1"
        cls.process = popen_launch_server(
            cls.model,
            cls.base_url,
            timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
            other_args=[
                *cls.common_args,
            ],
        )
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 49-58: tearDownClass cleanup routine / tearDownClass 清理流程
```python
    @classmethod
    def tearDownClass(cls):
        """Fixture that is run once after all tests in the class."""
        if hasattr(cls, "process") and cls.process:
            cls.process.terminate()
            try:
                cls.process.wait(timeout=30)
            except Exception:
                # Force kill if it didn't exit cleanly in time
                kill_process_tree(cls.process.pid)
```
**EN:** Fixture that is run once after all tests in the class. This routine releases resources and restores state after the related tests finish.
**CN:** Fixture that is run once after all tests in the class. 该流程会在相关测试结束后释放资源并恢复状态。

### Lines 60-72: helper routine get request json / 辅助流程 get request json
```python
    def get_request_json(self, max_new_tokens=32, n=1):
        response = requests.post(
            self.base_url + "/generate",
            json={
                "text": "<image>\n<|grounding|>Convert the document to pure text.",
                "image_data": self.image_path,
                "sampling_params": {
                    "temperature": 0 if n == 1 else 0.5,
                    "max_new_tokens": max_new_tokens,
                },
            },
        )
        return response.json()
```
**EN:** This helper encapsulates `get_request_json` so the surrounding tests can reuse setup, execution, or validation logic.
**CN:** 该辅助函数封装了 `get_request_json`，以便周围测试复用准备、执行或校验逻辑。

### Lines 74-103: helper routine run decode / 辅助流程 run decode
```python
    def run_decode(
        self,
        max_new_tokens=128,
        n=1,
    ):

        ret = self.get_request_json(max_new_tokens=max_new_tokens, n=n)
        print(json.dumps(ret, indent=2))

        def assert_one_item(item):
            if item["meta_info"]["finish_reason"]["type"] == "stop":
                self.assertEqual(
                    item["meta_info"]["finish_reason"]["matched"],
                    self.tokenizer.eos_token_id,
                )
            elif item["meta_info"]["finish_reason"]["type"] == "length":
                self.assertEqual(
                    len(item["output_ids"]), item["meta_info"]["completion_tokens"]
                )
                self.assertEqual(len(item["output_ids"]), max_new_tokens)

        # Determine whether to assert a single item or multiple items based on n
        if n == 1:
            assert_one_item(ret)
        else:
            self.assertEqual(len(ret), n)
            for i in range(n):
                assert_one_item(ret[i])

        print("=" * 100)
```
**EN:** This helper encapsulates `run_decode` so the surrounding tests can reuse setup, execution, or validation logic.
**CN:** 该辅助函数封装了 `run_decode`，以便周围测试复用准备、执行或校验逻辑。

### Lines 105-106: test case moe / 测试用例 moe
```python
    def test_moe(self):
        self.run_decode()
```
**EN:** This test exercises `test_moe` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_moe`。

### Lines 109-110: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `TestDeepSeekOCR`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestDeepSeekOCR.setUpClass`: This routine prepares shared fixtures, models, or runtime state before the assertions execute. / 该流程会在断言执行前准备共享夹具、模型或运行状态。
- `TestDeepSeekOCR.tearDownClass`: Fixture that is run once after all tests in the class. / 该流程会在相关测试结束后释放资源并恢复状态。
- `TestDeepSeekOCR.get_request_json`: This helper encapsulates `get_request_json` so the surrounding tests can reuse setup, execution, or validation logic. / 该辅助函数封装了 `get_request_json`，以便周围测试复用准备、执行或校验逻辑。
- `TestDeepSeekOCR.run_decode`: This helper encapsulates `run_decode` so the surrounding tests can reuse setup, execution, or validation logic. / 该辅助函数封装了 `run_decode`，以便周围测试复用准备、执行或校验逻辑。
- `TestDeepSeekOCR.test_moe`: This test exercises `test_moe` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_moe`。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `json`, `os`, `unittest`, `pathlib`
- **Third-party modules / 第三方模块**: `requests`
- **Internal modules / 内部模块**: `sglang.srt.utils`, `sglang.srt.utils.hf_transformers`, `sglang.test.test_utils`

- **Total lines / 总行数**: 110
