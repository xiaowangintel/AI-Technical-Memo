# test_dataset_tools.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/unit/auto_benchmark/test_dataset_tools.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates dataset tools behavior in SGLang's unit / auto benchmark area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 单元 / auto benchmark 领域中与 dataset tools 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-5: module imports and dependencies / 模块导入与依赖
```python
import json
import sys
import unittest
from pathlib import Path
from types import SimpleNamespace
```
**EN:** This block imports the modules needed by the rest of the file, including `json`, `sys`, `unittest`, `pathlib`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `json`, `sys`, `unittest`, `pathlib`。

### Lines 7-10: module-level constants and configuration / 模块级常量与配置
```python
CURRENT_DIR = Path(__file__).resolve().parent
PARENT_DIR = CURRENT_DIR.parent
if str(PARENT_DIR) not in sys.path:
    sys.path.insert(0, str(PARENT_DIR))
```
**EN:** This block defines shared names such as `CURRENT_DIR`, `PARENT_DIR`. These values centralize configuration that later tests or helpers reuse.
**CN:** 该代码块定义了 `CURRENT_DIR`, `PARENT_DIR` 等共享名称，用于集中保存后续测试或辅助逻辑会复用的配置。

### Lines 12-16: module imports and dependencies / 模块导入与依赖
```python
from auto_benchmark import AutoBenchmarkTestCase

from sglang.auto_benchmark_lib import infer_backend, prepare_dataset
from sglang.benchmark.datasets.autobench import sample_autobench_requests
from sglang.test.ci.ci_register import register_amd_ci, register_cuda_ci
```
**EN:** This block imports the modules needed by the rest of the file, including `auto_benchmark`, `sglang.auto_benchmark_lib`, `sglang.benchmark.datasets.autobench`, `sglang.test.ci.ci_register`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `auto_benchmark`, `sglang.auto_benchmark_lib`, `sglang.benchmark.datasets.autobench`, `sglang.test.ci.ci_register`。

### Lines 18-19: CI registration and metadata / CI 注册与元数据
```python
register_cuda_ci(est_time=6, stage="base-b", runner_config="1-gpu-small")
register_amd_ci(est_time=6, suite="stage-b-test-1-gpu-small-amd")
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through register_cuda_ci, register_amd_ci.
**CN:** 该代码块通过 register_cuda_ci, register_amd_ci 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 22-22: class TestAutoBenchmarkDatasetTools declaration / 类 TestAutoBenchmarkDatasetTools 声明
```python
class TestAutoBenchmarkDatasetTools(AutoBenchmarkTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `AutoBenchmarkTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `AutoBenchmarkTestCase`。

### Lines 23-47: test case prepare custom autobench dataset / 测试用例 prepare custom autobench dataset
```python
    def test_prepare_custom_autobench_dataset(self):
        dataset_path = self._write_autobench_jsonl()
        output_path = self.tmpdir_path / "prepared.autobench.jsonl"

        prepared_path, rows, summary = prepare_dataset(
            dataset_cfg={
                "kind": "custom",
                "path": dataset_path,
                "num_prompts": 2,
            },
            tokenizer_path=str(self.tokenizer_dir),
            model=None,
            output_path=str(output_path),
        )

        self.assertEqual(prepared_path, str(output_path))
        self.assertEqual(summary["num_requests"], 2)
        self.assertTrue(Path(prepared_path).exists())
        converted_rows = sample_autobench_requests(
            dataset_path=prepared_path,
            num_requests=0,
            tokenizer=self.tokenizer,
        )
        self.assertEqual(len(rows), 2)
        self.assertEqual(len(converted_rows), 2)
```
**EN:** This test exercises `test_prepare_custom_autobench_dataset` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_prepare_custom_autobench_dataset`。

### Lines 49-63: test case invalid json like prompt falls back to plain text / 测试用例 invalid json like prompt falls back to plain text
```python
    def test_invalid_json_like_prompt_falls_back_to_plain_text(self):
        path = self.tmpdir_path / "jsonlike.autobench.jsonl"
        path.write_text(
            json.dumps({"prompt": "[not actually json", "output_len": 8}) + "\n",
            encoding="utf-8",
        )

        rows = sample_autobench_requests(
            dataset_path=str(path),
            num_requests=0,
            tokenizer=self.tokenizer,
        )

        self.assertEqual(len(rows), 1)
        self.assertEqual(rows[0].prompt, "[not actually json")
```
**EN:** This test exercises `test_invalid_json_like_prompt_falls_back_to_plain_text` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_invalid_json_like_prompt_falls_back_to_plain_text`。

### Lines 65-82: test case prepare sharegpt dataset / 测试用例 prepare sharegpt dataset
```python
    def test_prepare_sharegpt_dataset(self):
        sharegpt_path = self._write_sharegpt_json()
        output_path = self.tmpdir_path / "sharegpt.autobench.jsonl"

        prepared_path, rows, summary = prepare_dataset(
            dataset_cfg={
                "kind": "sharegpt",
                "path": sharegpt_path,
                "num_prompts": 2,
            },
            tokenizer_path=str(self.tokenizer_dir),
            model=None,
            output_path=str(output_path),
        )

        self.assertEqual(prepared_path, str(output_path))
        self.assertEqual(summary["num_requests"], 2)
        self.assertEqual(len(rows), 2)
```
**EN:** This test exercises `test_prepare_sharegpt_dataset` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_prepare_sharegpt_dataset`。

### Lines 84-91: test case prepare custom dataset requires path / 测试用例 prepare custom dataset requires path
```python
    def test_prepare_custom_dataset_requires_path(self):
        with self.assertRaisesRegex(ValueError, "dataset.path is required"):
            prepare_dataset(
                dataset_cfg={"kind": "custom"},
                tokenizer_path=str(self.tokenizer_dir),
                model=None,
                output_path=str(self.tmpdir_path / "missing.autobench.jsonl"),
            )
```
**EN:** This test exercises `test_prepare_custom_dataset_requires_path` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_prepare_custom_dataset_requires_path`。

### Lines 93-100: test case infer backend / 测试用例 infer backend
```python
    def test_infer_backend(self):
        prompt_rows = [SimpleNamespace(prompt="tok_1 tok_2")]
        chat_rows = [SimpleNamespace(prompt=[{"role": "user", "content": "tok_1"}])]
        token_id_rows = [SimpleNamespace(prompt=[1, 2, 3])]

        self.assertEqual(infer_backend("auto", prompt_rows), "sglang-oai")
        self.assertEqual(infer_backend("auto", chat_rows), "sglang-oai-chat")
        self.assertEqual(infer_backend("auto", token_id_rows), "sglang")
```
**EN:** This test exercises `test_infer_backend` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_infer_backend`。

### Lines 103-104: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `TestAutoBenchmarkDatasetTools`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestAutoBenchmarkDatasetTools.test_prepare_custom_autobench_dataset`: This test exercises `test_prepare_custom_autobench_dataset` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_prepare_custom_autobench_dataset`。
- `TestAutoBenchmarkDatasetTools.test_invalid_json_like_prompt_falls_back_to_plain_text`: This test exercises `test_invalid_json_like_prompt_falls_back_to_plain_text` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_invalid_json_like_prompt_falls_back_to_plain_text`。
- `TestAutoBenchmarkDatasetTools.test_prepare_sharegpt_dataset`: This test exercises `test_prepare_sharegpt_dataset` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_prepare_sharegpt_dataset`。
- `TestAutoBenchmarkDatasetTools.test_prepare_custom_dataset_requires_path`: This test exercises `test_prepare_custom_dataset_requires_path` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_prepare_custom_dataset_requires_path`。
- `TestAutoBenchmarkDatasetTools.test_infer_backend`: This test exercises `test_infer_backend` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_infer_backend`。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `json`, `sys`, `unittest`, `pathlib`, `types`
- **Third-party modules / 第三方模块**: `auto_benchmark`
- **Internal modules / 内部模块**: `sglang.auto_benchmark_lib`, `sglang.benchmark.datasets.autobench`, `sglang.test.ci.ci_register`

- **Total lines / 总行数**: 104
