# test_model_file_verifier.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/utils/test_model_file_verifier.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates model file verifier behavior in SGLang's utils area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 工具 领域中与 model file verifier 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-28: module imports and dependencies / 模块导入与依赖
```python
import hashlib
import json
import os
import shutil
import subprocess
import sys
import tempfile
import unittest
import warnings
from contextlib import nullcontext
from io import StringIO

import requests
from huggingface_hub import snapshot_download

from sglang.srt.utils import kill_process_tree
from sglang.srt.utils.model_file_verifier import (
    IntegrityError,
    compute_sha256,
    generate_checksums,
    verify,
)
from sglang.test.ci.ci_register import register_cuda_ci
from sglang.test.test_utils import (
    DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
    DEFAULT_URL_FOR_TEST,
    popen_launch_server,
)
```
**EN:** This block imports the modules needed by the rest of the file, including `hashlib`, `json`, `os`, `shutil`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `hashlib`, `json`, `os`, `shutil`。

### Lines 29-30: supporting source context / 辅助源码上下文
```python

# Note: AMD registration removed - test_model_file_verifier fails on AMD
```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 31-33: CI registration and metadata / CI 注册与元数据
```python
register_cuda_ci(est_time=120, suite="nightly-1-gpu", nightly=True)

MODEL_NAME = "Qwen/Qwen3-0.6B"
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through register_cuda_ci.
**CN:** 该代码块通过 register_cuda_ci 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 34-38: supporting source context / 辅助源码上下文
```python


# ======== Base Test Classes ========


```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 39-40: class _FakeModelTestCase declaration / 类 _FakeModelTestCase 声明
```python
class _FakeModelTestCase(unittest.TestCase):

```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `unittest.TestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `unittest.TestCase`。

### Lines 41-45: class-level constants and configuration for `_FakeModelTestCase` / 类级常量与配置
```python
    FAKE_FILES = {
        "model.safetensors": b"fake safetensors content " * 100,
        "config.json": b'{"model_type": "llama"}',
        "tokenizer.json": b'{"version": "1.0"}',
    }
```
**EN:** This block defines shared names such as `FAKE_FILES`. These values centralize configuration that later tests or helpers reuse.
**CN:** 该代码块定义了 `FAKE_FILES` 等共享名称，用于集中保存后续测试或辅助逻辑会复用的配置。

### Lines 47-50: setUp setup routine / setUp 初始化流程
```python
    def setUp(self):
        self.test_dir = tempfile.mkdtemp()
        for filename, content in self.FAKE_FILES.items():
            _create_test_file(self.test_dir, filename, content)
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 52-53: tearDown cleanup routine / tearDown 清理流程
```python
    def tearDown(self):
        shutil.rmtree(self.test_dir, ignore_errors=True)
```
**EN:** This routine releases resources and restores state after the related tests finish.
**CN:** 该流程会在相关测试结束后释放资源并恢复状态。

### Lines 56-57: class _RealModelTestCase declaration / 类 _RealModelTestCase 声明
```python
class _RealModelTestCase(unittest.TestCase):

```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `unittest.TestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `unittest.TestCase`。

### Lines 58-60: setUpClass setup routine / setUpClass 初始化流程
```python
    @classmethod
    def setUpClass(cls):
        cls.original_model_path = snapshot_download(MODEL_NAME)
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 62-64: setUp setup routine / setUp 初始化流程
```python
    def setUp(self):
        self.test_dir = tempfile.mkdtemp()
        shutil.copytree(self.original_model_path, self.test_dir, dirs_exist_ok=True)
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 66-67: tearDown cleanup routine / tearDown 清理流程
```python
    def tearDown(self):
        shutil.rmtree(self.test_dir, ignore_errors=True)
```
**EN:** This routine releases resources and restores state after the related tests finish.
**CN:** 该流程会在相关测试结束后释放资源并恢复状态。

### Lines 68-72: supporting source context / 辅助源码上下文
```python


# ======== Unit Tests ========


```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 73-74: class TestModelFileVerifier declaration / 类 TestModelFileVerifier 声明
```python
class TestModelFileVerifier(_FakeModelTestCase):

```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `_FakeModelTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `_FakeModelTestCase`。

### Lines 75-86: test case detect bit rot / 测试用例 detect bit rot
```python
    def test_detect_bit_rot(self):
        checksums_file = os.path.join(self.test_dir, "checksums.json")
        generate_checksums(source=self.test_dir, output_path=checksums_file)

        target_file = os.path.join(self.test_dir, "model.safetensors")
        _flip_bit_in_file(target_file, byte_offset=50, bit_position=3)

        with self.assertRaises(IntegrityError) as ctx:
            verify(model_path=self.test_dir, checksums_source=checksums_file)

        self.assertIn("model.safetensors", str(ctx.exception))
        self.assertIn("mismatch", str(ctx.exception).lower())
```
**EN:** This test exercises `test_detect_bit_rot` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_detect_bit_rot`。

### Lines 88-97: test case detect missing file / 测试用例 detect missing file
```python
    def test_detect_missing_file(self):
        checksums_file = os.path.join(self.test_dir, "checksums.json")
        generate_checksums(source=self.test_dir, output_path=checksums_file)

        os.remove(os.path.join(self.test_dir, "config.json"))

        with self.assertRaises(IntegrityError) as ctx:
            verify(model_path=self.test_dir, checksums_source=checksums_file)

        self.assertIn("config.json", str(ctx.exception))
```
**EN:** This test exercises `test_detect_missing_file` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_detect_missing_file`。

### Lines 99-107: test case compute sha256 / 测试用例 compute sha256
```python
    def test_compute_sha256(self):
        test_file = os.path.join(self.test_dir, "test.bin")
        content = b"hello world"
        with open(test_file, "wb") as f:
            f.write(content)

        result = compute_sha256(file_path=test_file)
        expected = hashlib.sha256(content).hexdigest()
        self.assertEqual(result, expected)
```
**EN:** This test exercises `test_compute_sha256` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_compute_sha256`。

### Lines 109-120: test case parallel checksum computation / 测试用例 parallel checksum computation
```python
    def test_parallel_checksum_computation(self):
        for i in range(10):
            _create_test_file(
                self.test_dir, f"shard_{i}.safetensors", f"content_{i}".encode() * 1000
            )

        checksums_file = os.path.join(self.test_dir, "checksums.json")
        result = generate_checksums(
            source=self.test_dir, output_path=checksums_file, max_workers=4
        )

        self.assertGreaterEqual(len(result.files), 10)
```
**EN:** This test exercises `test_parallel_checksum_computation` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_parallel_checksum_computation`。

### Lines 122-145: test case generated json snapshot / 测试用例 generated json snapshot
```python
    def test_generated_json_snapshot(self):
        checksums_file = os.path.join(self.test_dir, "checksums.json")
        generate_checksums(source=self.test_dir, output_path=checksums_file)

        with open(checksums_file) as f:
            data = json.load(f)

        expected = {
            "files": {
                "config.json": {
                    "sha256": "81dddc8c379baae137d99d24c5fa081d3a5ce52b6a221ddc22fe364711f8beaf",
                    "size": 23,
                },
                "model.safetensors": {
                    "sha256": "eb0c73a48a89fefb6b68dd41af830d75610c885135eac99139373b04705d05f3",
                    "size": 2500,
                },
                "tokenizer.json": {
                    "sha256": "4e3043229142b64d998563bc543ce034e0a2251af5d404995e3afcb8ce8850df",
                    "size": 18,
                },
            }
        }
        self.assertEqual(data, expected)
```
**EN:** This test exercises `test_generated_json_snapshot` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_generated_json_snapshot`。

### Lines 147-164: test case legacy checksums format deprecated / 测试用例 legacy checksums format deprecated
```python
    def test_legacy_checksums_format_deprecated(self):
        legacy_data = {
            "checksums": {
                "model.safetensors": "eb0c73a48a89fefb6b68dd41af830d75610c885135eac99139373b04705d05f3",
                "config.json": "81dddc8c379baae137d99d24c5fa081d3a5ce52b6a221ddc22fe364711f8beaf",
                "tokenizer.json": "4e3043229142b64d998563bc543ce034e0a2251af5d404995e3afcb8ce8850df",
            }
        }
        legacy_file = os.path.join(self.test_dir, "legacy_checksums.json")
        with open(legacy_file, "w") as f:
            json.dump(legacy_data, f)

        with warnings.catch_warnings(record=True) as w:
            warnings.simplefilter("always")
            verify(model_path=self.test_dir, checksums_source=legacy_file)
            self.assertEqual(len(w), 1)
            self.assertTrue(issubclass(w[0].category, DeprecationWarning))
            self.assertIn("deprecated", str(w[0].message).lower())
```
**EN:** This test exercises `test_legacy_checksums_format_deprecated` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_legacy_checksums_format_deprecated`。

### Lines 165-169: supporting source context / 辅助源码上下文
```python


# ======== CLI Tests ========


```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 170-171: class TestModelFileVerifierCLI declaration / 类 TestModelFileVerifierCLI 声明
```python
class TestModelFileVerifierCLI(_FakeModelTestCase):

```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `_FakeModelTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `_FakeModelTestCase`。

### Lines 172-194: test case cli generate / 测试用例 cli generate
```python
    def test_cli_generate(self):
        checksums_file = os.path.join(self.test_dir, "checksums.json")
        result = subprocess.run(
            [
                sys.executable,
                "-m",
                "sglang.srt.utils.model_file_verifier",
                "generate",
                "--model-path",
                self.test_dir,
                "--model-checksum",
                checksums_file,
            ],
            capture_output=True,
            text=True,
        )
        self.assertEqual(result.returncode, 0, f"stderr: {result.stderr}")
        self.assertTrue(os.path.exists(checksums_file))

        with open(checksums_file) as f:
            data = json.load(f)
        self.assertIn("files", data)
        self.assertEqual(len(data["files"]), 3)
```
**EN:** This test exercises `test_cli_generate` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_cli_generate`。

### Lines 196-215: test case cli verify success / 测试用例 cli verify success
```python
    def test_cli_verify_success(self):
        checksums_file = os.path.join(self.test_dir, "checksums.json")
        generate_checksums(source=self.test_dir, output_path=checksums_file)

        result = subprocess.run(
            [
                sys.executable,
                "-m",
                "sglang.srt.utils.model_file_verifier",
                "verify",
                "--model-path",
                self.test_dir,
                "--model-checksum",
                checksums_file,
            ],
            capture_output=True,
            text=True,
        )
        self.assertEqual(result.returncode, 0, f"stderr: {result.stderr}")
        self.assertIn("verified successfully", result.stdout)
```
**EN:** This test exercises `test_cli_verify_success` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_cli_verify_success`。

### Lines 217-243: test case cli verify fails on corruption / 测试用例 cli verify fails on corruption
```python
    def test_cli_verify_fails_on_corruption(self):
        checksums_file = os.path.join(self.test_dir, "checksums.json")
        generate_checksums(source=self.test_dir, output_path=checksums_file)

        target_file = os.path.join(self.test_dir, "model.safetensors")
        _flip_bit_in_file(target_file, byte_offset=50, bit_position=3)

        result = subprocess.run(
            [
                sys.executable,
                "-m",
                "sglang.srt.utils.model_file_verifier",
                "verify",
                "--model-path",
                self.test_dir,
                "--model-checksum",
                checksums_file,
            ],
            capture_output=True,
            text=True,
        )
        self.assertNotEqual(result.returncode, 0)
        combined = result.stdout + result.stderr
        self.assertTrue(
            "IntegrityError" in combined or "mismatch" in combined.lower(),
            f"Expected integrity error, got: {combined}",
        )
```
**EN:** This test exercises `test_cli_verify_fails_on_corruption` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_cli_verify_fails_on_corruption`。

### Lines 244-248: supporting source context / 辅助源码上下文
```python


# ======== HuggingFace Tests ========


```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 249-250: class TestModelFileVerifierHF declaration / 类 TestModelFileVerifierHF 声明
```python
class TestModelFileVerifierHF(_RealModelTestCase):

```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `_RealModelTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `_RealModelTestCase`。

### Lines 251-258: test case generate checksums from hf / 测试用例 generate checksums from hf
```python
    def test_generate_checksums_from_hf(self):
        checksums_file = os.path.join(self.test_dir, "checksums.json")
        result = generate_checksums(source=MODEL_NAME, output_path=checksums_file)

        self.assertTrue(os.path.exists(checksums_file))
        self.assertGreater(len(result.files), 0)
        for filename, file_info in result.files.items():
            self.assertEqual(len(file_info.sha256), 64)
```
**EN:** This test exercises `test_generate_checksums_from_hf` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_generate_checksums_from_hf`。

### Lines 260-261: test case verify with hf checksums source / 测试用例 verify with hf checksums source
```python
    def test_verify_with_hf_checksums_source(self):
        verify(model_path=self.test_dir, checksums_source=MODEL_NAME)
```
**EN:** This test exercises `test_verify_with_hf_checksums_source` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_verify_with_hf_checksums_source`。

### Lines 262-266: supporting source context / 辅助源码上下文
```python


# ======== Real Model E2E Tests ========


```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 267-268: class TestModelFileVerifierWithRealModel declaration / 类 TestModelFileVerifierWithRealModel 声明
```python
class TestModelFileVerifierWithRealModel(_RealModelTestCase):

```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `_RealModelTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `_RealModelTestCase`。

### Lines 269-310: method run server test / 方法 run server test
```python
    def _run_server_test(self, *, corrupt_weights: bool, use_hf_checksum: bool):
        if use_hf_checksum:
            checksum_arg = MODEL_NAME
        else:
            checksums_file = os.path.join(self.test_dir, "checksums.json")
            generate_checksums(source=self.test_dir, output_path=checksums_file)
            checksum_arg = checksums_file

        corrupted_file = None
        if corrupt_weights:
            safetensors_files = [
                f for f in os.listdir(self.test_dir) if f.endswith(".safetensors")
            ]
            self.assertTrue(len(safetensors_files) > 0, "No safetensors files found")
            corrupted_file = safetensors_files[0]
            _flip_bit_in_file(os.path.join(self.test_dir, corrupted_file))

        stdout_io, stderr_io = StringIO(), StringIO()
        ctx = self.assertRaises(Exception) if corrupt_weights else nullcontext()
        with ctx:
            process = popen_launch_server(
                model=self.test_dir,
                base_url=DEFAULT_URL_FOR_TEST,
                timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
                other_args=["--model-checksum", checksum_arg],
                return_stdout_stderr=(stdout_io, stderr_io),
            )

        if corrupt_weights:
            output = stdout_io.getvalue() + stderr_io.getvalue()
            self.assertIn(corrupted_file, output)
            self.assertIn("mismatch", output.lower())
        else:
            try:
                response = requests.post(
                    f"{DEFAULT_URL_FOR_TEST}/generate",
                    json={"text": "Hello", "sampling_params": {"max_new_tokens": 8}},
                )
                self.assertEqual(response.status_code, 200)
                self.assertIn("text", response.json())
            finally:
                kill_process_tree(process.pid)
```
**EN:** This block implements `_run_server_test` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_run_server_test`，承担模块行为中的一个聚焦逻辑片段。

### Lines 312-313: test case server launch with checksum intact / 测试用例 server launch with checksum intact
```python
    def test_server_launch_with_checksum_intact(self):
        self._run_server_test(corrupt_weights=False, use_hf_checksum=False)
```
**EN:** This test exercises `test_server_launch_with_checksum_intact` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_server_launch_with_checksum_intact`。

### Lines 315-316: test case server launch fails with corrupted weights / 测试用例 server launch fails with corrupted weights
```python
    def test_server_launch_fails_with_corrupted_weights(self):
        self._run_server_test(corrupt_weights=True, use_hf_checksum=False)
```
**EN:** This test exercises `test_server_launch_fails_with_corrupted_weights` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_server_launch_fails_with_corrupted_weights`。

### Lines 318-319: test case server launch with hf checksum intact / 测试用例 server launch with hf checksum intact
```python
    def test_server_launch_with_hf_checksum_intact(self):
        self._run_server_test(corrupt_weights=False, use_hf_checksum=True)
```
**EN:** This test exercises `test_server_launch_with_hf_checksum_intact` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_server_launch_with_hf_checksum_intact`。

### Lines 321-322: test case server launch with hf checksum corrupted / 测试用例 server launch with hf checksum corrupted
```python
    def test_server_launch_with_hf_checksum_corrupted(self):
        self._run_server_test(corrupt_weights=True, use_hf_checksum=True)
```
**EN:** This test exercises `test_server_launch_with_hf_checksum_corrupted` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_server_launch_with_hf_checksum_corrupted`。

### Lines 323-327: supporting source context / 辅助源码上下文
```python


# ======== Test Utilities ========


```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 328-332: function create test file / 函数 create test file
```python
def _create_test_file(directory: str, filename: str, content: bytes) -> str:
    path = os.path.join(directory, filename)
    with open(path, "wb") as f:
        f.write(content)
    return path
```
**EN:** This block implements `_create_test_file` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_create_test_file`，承担模块行为中的一个聚焦逻辑片段。

### Lines 335-345: function flip bit in file / 函数 flip bit in file
```python
def _flip_bit_in_file(file_path: str, byte_offset: int = 100, bit_position: int = 0):
    file_size = os.path.getsize(file_path)
    assert (
        byte_offset < file_size
    ), f"byte_offset {byte_offset} >= file_size {file_size}"

    with open(file_path, "r+b") as f:
        f.seek(byte_offset)
        original_byte = f.read(1)[0]
        f.seek(byte_offset)
        f.write(bytes([original_byte ^ (1 << bit_position)]))
```
**EN:** This block implements `_flip_bit_in_file` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_flip_bit_in_file`，承担模块行为中的一个聚焦逻辑片段。

### Lines 348-349: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `_FakeModelTestCase`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `_RealModelTestCase`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestModelFileVerifier`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestModelFileVerifierCLI`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestModelFileVerifierHF`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestModelFileVerifierWithRealModel`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `_create_test_file`: This block implements `_create_test_file` and captures one focused piece of the module's behavior. / 该代码块实现 `_create_test_file`，承担模块行为中的一个聚焦逻辑片段。
- `_flip_bit_in_file`: This block implements `_flip_bit_in_file` and captures one focused piece of the module's behavior. / 该代码块实现 `_flip_bit_in_file`，承担模块行为中的一个聚焦逻辑片段。
- `_FakeModelTestCase.setUp`: This routine prepares shared fixtures, models, or runtime state before the assertions execute. / 该流程会在断言执行前准备共享夹具、模型或运行状态。
- `_FakeModelTestCase.tearDown`: This routine releases resources and restores state after the related tests finish. / 该流程会在相关测试结束后释放资源并恢复状态。
- `_RealModelTestCase.setUpClass`: This routine prepares shared fixtures, models, or runtime state before the assertions execute. / 该流程会在断言执行前准备共享夹具、模型或运行状态。
- `_RealModelTestCase.setUp`: This routine prepares shared fixtures, models, or runtime state before the assertions execute. / 该流程会在断言执行前准备共享夹具、模型或运行状态。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `hashlib`, `json`, `os`, `shutil`, `subprocess`, `sys`, `tempfile`, `unittest`, `warnings`, `contextlib`, `io`
- **Third-party modules / 第三方模块**: `requests`, `huggingface_hub`
- **Internal modules / 内部模块**: `sglang.srt.utils`, `sglang.srt.utils.model_file_verifier`, `sglang.test.ci.ci_register`, `sglang.test.test_utils`

- **Total lines / 总行数**: 349
