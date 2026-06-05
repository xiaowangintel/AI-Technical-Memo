# test_hicache_spec_file_storage.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/hicache/test_hicache_spec_file_storage.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates hicache spec file storage behavior in SGLang's hicache area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 hicache 领域中与 hicache spec file storage 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-6: supporting statements / 辅助语句
```python
"""
E2E test for HiCache file storage with EAGLE3 speculative decoding.

Usage:
    python3 -m pytest test/registered/hicache/test_hicache_spec_file_storage.py -v
"""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 8-31: module imports and dependencies / 模块导入与依赖
```python
import json
import os
import shutil
import tempfile
import time
import unittest
from typing import Dict, List

import psutil
import requests

from sglang.benchmark.utils import get_tokenizer
from sglang.srt.utils import is_hip, kill_process_tree
from sglang.test.ci.ci_register import register_cuda_ci
from sglang.test.test_utils import (
    DEFAULT_DRAFT_MODEL_EAGLE3,
    DEFAULT_TARGET_MODEL_EAGLE3,
    DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
    DEFAULT_URL_FOR_TEST,
    CustomTestCase,
    find_available_port,
    popen_launch_server,
)
from sglang.utils import wait_for_http_ready
```
**EN:** This block imports the modules needed by the rest of the file, including `json`, `os`, `shutil`, `tempfile`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `json`, `os`, `shutil`, `tempfile`。

### Lines 33-33: CI registration and metadata / CI 注册与元数据
```python
register_cuda_ci(est_time=200, stage="extra-a", runner_config="1-gpu-large")
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through register_cuda_ci.
**CN:** 该代码块通过 register_cuda_ci 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 36-37: class TestHiCacheSpecFileStorage declaration / 类 TestHiCacheSpecFileStorage 声明
```python
@unittest.skipIf(is_hip(), "HiCache + EAGLE3 file-storage loadback e2e is CUDA-only.")
class TestHiCacheSpecFileStorage(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 38-47: class-level constants and configuration for `TestHiCacheSpecFileStorage` / 类级常量与配置
```python
    model = DEFAULT_TARGET_MODEL_EAGLE3
    draft_model = DEFAULT_DRAFT_MODEL_EAGLE3

    input_token_len = 1024
    max_new_tokens = 200
    page_size = 64
    min_expected_accept_length = 7.0
    min_second_to_first_accept_ratio = 0.9
    storage_wait_timeout = 30
    first_measure_new_tokens = 128
```
**EN:** This block defines shared names such as `model`, `draft_model`, `input_token_len`, `max_new_tokens`, `page_size`. These values centralize configuration that later tests or helpers reuse.
**CN:** 该代码块定义了 `model`, `draft_model`, `input_token_len`, `max_new_tokens`, `page_size` 等共享名称，用于集中保存后续测试或辅助逻辑会复用的配置。

### Lines 49-98: setUpClass setup routine / setUpClass 初始化流程
```python
    @classmethod
    def setUpClass(cls):
        cls.temp_dir = tempfile.mkdtemp()
        default_port = int(DEFAULT_URL_FOR_TEST.rsplit(":", 1)[1])
        cls.base_url = f"http://127.0.0.1:{find_available_port(default_port)}"

        cls.tokenizer = get_tokenizer(cls.model)
        cls.prompt_input_ids = cls._build_long_repetitive_prompt_ids(
            cls.tokenizer, cls.input_token_len
        )

        extra_config = {
            "hicache_storage_pass_prefix_keys": True,
        }
        cls.other_args = [
            "--enable-hierarchical-cache",
            "--enable-cache-report",
            "--mem-fraction-static",
            "0.3",
            "--hicache-ratio",
            "1.5",
            "--disable-cuda-graph",
            "--page-size",
            str(cls.page_size),
            "--hicache-storage-backend",
            "file",
            "--hicache-storage-prefetch-policy",
            "wait_complete",
            "--hicache-storage-backend-extra-config",
            json.dumps(extra_config),
            "--speculative-algorithm",
            "EAGLE3",
            "--speculative-draft-model-path",
            cls.draft_model,
            "--speculative-num-steps",
            "7",
            "--speculative-eagle-topk",
            "1",
            "--speculative-num-draft-tokens",
            "8",
            "--dtype",
            "float16",
        ]
        cls.env = {
            **os.environ,
            "SGLANG_ALLOW_OVERWRITE_LONGER_CONTEXT_LEN": "1",
            "SGLANG_HICACHE_FILE_BACKEND_STORAGE_DIR": cls.temp_dir,
        }
        cls.process = None
        cls._launch_server()
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 100-113: method launch server / 方法 launch server
```python
    @classmethod
    def _launch_server(cls):
        cls.process = popen_launch_server(
            cls.model,
            cls.base_url,
            timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
            other_args=cls.other_args,
            env=cls.env,
        )
        wait_for_http_ready(
            url=f"{cls.base_url}/health",
            timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
            process=cls.process,
        )
```
**EN:** This block implements `_launch_server` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_launch_server`，承担模块行为中的一个聚焦逻辑片段。

### Lines 115-140: method stop server / 方法 stop server
```python
    @classmethod
    def _stop_server(cls):
        if getattr(cls, "process", None) is None:
            return

        process = cls.process
        try:
            root = psutil.Process(process.pid)
            watched_procs = [root] + root.children(recursive=True)
        except psutil.NoSuchProcess:
            watched_procs = []

        try:
            kill_process_tree(process.pid, wait_timeout=60)
        except RuntimeError:
            non_zombie_procs = []
            for proc in watched_procs:
                try:
                    if proc.is_running() and proc.status() != psutil.STATUS_ZOMBIE:
                        non_zombie_procs.append(proc)
                except psutil.NoSuchProcess:
                    pass
            if non_zombie_procs:
                raise
        finally:
            cls.process = None
```
**EN:** This block implements `_stop_server` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_stop_server`，承担模块行为中的一个聚焦逻辑片段。

### Lines 142-145: method restart server / 方法 restart server
```python
    @classmethod
    def _restart_server(cls):
        cls._stop_server()
        cls._launch_server()
```
**EN:** This block implements `_restart_server` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_restart_server`，承担模块行为中的一个聚焦逻辑片段。

### Lines 147-163: method count file storage pages / 方法 count file storage pages
```python
    @classmethod
    def _count_file_storage_pages(cls):
        try:
            filenames = os.listdir(cls.temp_dir)
        except FileNotFoundError:
            return 0, 0

        target_pages = 0
        draft_pages = 0
        for filename in filenames:
            if not filename.endswith(".bin"):
                continue
            if filename.startswith("d:"):
                draft_pages += 1
            else:
                target_pages += 1
        return target_pages, draft_pages
```
**EN:** This block implements `_count_file_storage_pages` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_count_file_storage_pages`，承担模块行为中的一个聚焦逻辑片段。

### Lines 165-180: method wait for file storage pages / 方法 wait for file storage pages
```python
    @classmethod
    def _wait_for_file_storage_pages(cls):
        min_pages = (cls.input_token_len - 2 * cls.page_size) // cls.page_size
        deadline = time.monotonic() + cls.storage_wait_timeout
        target_pages = draft_pages = 0

        while time.monotonic() < deadline:
            target_pages, draft_pages = cls._count_file_storage_pages()
            if target_pages >= min_pages and draft_pages >= min_pages:
                return target_pages, draft_pages
            time.sleep(0.2)

        raise AssertionError(
            "Timed out waiting for HiCache file storage pages before restart: "
            f"{target_pages=}, {draft_pages=}, {min_pages=}"
        )
```
**EN:** This block implements `_wait_for_file_storage_pages` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_wait_for_file_storage_pages`，承担模块行为中的一个聚焦逻辑片段。

### Lines 182-186: tearDownClass cleanup routine / tearDownClass 清理流程
```python
    @classmethod
    def tearDownClass(cls):
        cls._stop_server()
        if hasattr(cls, "temp_dir"):
            shutil.rmtree(cls.temp_dir, ignore_errors=True)
```
**EN:** This routine releases resources and restores state after the related tests finish.
**CN:** 该流程会在相关测试结束后释放资源并恢复状态。

### Lines 188-190: method encode without special tokens / 方法 encode without special tokens
```python
    @classmethod
    def _encode_without_special_tokens(cls, tokenizer, text: str) -> List[int]:
        return tokenizer.encode(text, add_special_tokens=False)
```
**EN:** This block implements `_encode_without_special_tokens` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_encode_without_special_tokens`，承担模块行为中的一个聚焦逻辑片段。

### Lines 192-220: method build long repetitive prompt ids / 方法 build long repetitive prompt ids
```python
    @classmethod
    def _build_long_repetitive_prompt_ids(cls, tokenizer, target_len: int) -> List[int]:
        bos_ids = (
            [tokenizer.bos_token_id]
            if getattr(tokenizer, "bos_token_id", None) is not None
            else []
        )
        suffix_ids = cls._encode_without_special_tokens(
            tokenizer,
            "\n\nContinue the sequence with only the word apple separated by spaces.\n"
            "Answer: apple apple apple apple",
        )
        repeat_ids = cls._encode_without_special_tokens(tokenizer, " apple")
        if not repeat_ids:
            raise ValueError(
                "Tokenizer produced no ids for the repetitive prompt seed."
            )
        if len(bos_ids) + len(suffix_ids) >= target_len:
            raise ValueError(
                "Prompt suffix is too long: "
                f"{len(bos_ids)=}, {len(suffix_ids)=}, {target_len=}."
            )

        prefix_len = target_len - len(bos_ids) - len(suffix_ids)
        repeats = (prefix_len + len(repeat_ids) - 1) // len(repeat_ids)
        prefix_ids = (repeat_ids * repeats)[:prefix_len]
        prompt_ids = bos_ids + prefix_ids + suffix_ids
        assert len(prompt_ids) == target_len
        return prompt_ids
```
**EN:** This block implements `_build_long_repetitive_prompt_ids` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_build_long_repetitive_prompt_ids`，承担模块行为中的一个聚焦逻辑片段。

### Lines 222-242: method send long prompt / 方法 send long prompt
```python
    def _send_long_prompt(self, max_new_tokens: int = None) -> Dict:
        if max_new_tokens is None:
            max_new_tokens = self.max_new_tokens
        response = requests.post(
            f"{self.base_url}/generate",
            json={
                "input_ids": self.prompt_input_ids,
                "sampling_params": {
                    "temperature": 0,
                    "max_new_tokens": max_new_tokens,
                    "ignore_eos": True,
                },
            },
            timeout=900,
        )
        self.assertEqual(
            response.status_code,
            200,
            f"Request failed: {response.status_code} - {response.text}",
        )
        return response.json()
```
**EN:** This block implements `_send_long_prompt` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_send_long_prompt`，承担模块行为中的一个聚焦逻辑片段。

### Lines 244-251: method get spec accept length / 方法 get spec accept length
```python
    def _get_spec_accept_length(self, response_json: Dict) -> float:
        meta_info = response_json.get("meta_info", {})
        self.assertIn(
            "spec_accept_length",
            meta_info,
            f"Missing spec_accept_length in meta_info: {meta_info}",
        )
        return float(meta_info["spec_accept_length"])
```
**EN:** This block implements `_get_spec_accept_length` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_get_spec_accept_length`，承担模块行为中的一个聚焦逻辑片段。

### Lines 253-299: test case file storage loadback keeps spec accept length / 测试用例 file storage loadback keeps spec accept length
```python
    def test_file_storage_loadback_keeps_spec_accept_length(self):
        first = self._send_long_prompt(max_new_tokens=self.first_measure_new_tokens)
        first_accept_length = self._get_spec_accept_length(first)
        self.assertGreaterEqual(
            first_accept_length,
            self.min_expected_accept_length,
            f"First prompt accept length is too low: {first_accept_length=}",
        )

        target_pages, draft_pages = self._wait_for_file_storage_pages()
        print(f"file_storage_before_restart: {target_pages=}, {draft_pages=}")

        self._restart_server()

        second = self._send_long_prompt()
        second_accept_length = self._get_spec_accept_length(second)
        second_meta = second.get("meta_info", {})
        cached_details = second_meta.get("cached_tokens_details") or {}
        storage_cached_tokens = int(cached_details.get("storage", 0))

        print(
            f"{first_accept_length=:.3f}, {second_accept_length=:.3f}, "
            f"{storage_cached_tokens=}, {cached_details=}"
        )

        self.assertGreaterEqual(
            storage_cached_tokens,
            self.input_token_len - 2 * self.page_size,
            "Expected the second request to load the long prompt KV cache from "
            f"file storage, got {cached_details=}",
        )
        self.assertEqual(
            cached_details.get("storage_backend"),
            "HiCacheFile",
            f"Expected file storage backend in cache report, got {cached_details=}",
        )
        self.assertGreaterEqual(
            second_accept_length,
            self.min_expected_accept_length,
            f"Second prompt accept length is too low: {second_accept_length=}",
        )
        self.assertGreaterEqual(
            second_accept_length,
            first_accept_length * self.min_second_to_first_accept_ratio,
            "Spec accept length dropped after file-storage loadback: "
            f"{first_accept_length=:.3f}, {second_accept_length=:.3f}",
        )
```
**EN:** This test exercises `test_file_storage_loadback_keeps_spec_accept_length` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_file_storage_loadback_keeps_spec_accept_length`。

### Lines 302-303: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    unittest.main(verbosity=2)
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `TestHiCacheSpecFileStorage`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestHiCacheSpecFileStorage.setUpClass`: This routine prepares shared fixtures, models, or runtime state before the assertions execute. / 该流程会在断言执行前准备共享夹具、模型或运行状态。
- `TestHiCacheSpecFileStorage._launch_server`: This block implements `_launch_server` and captures one focused piece of the module's behavior. / 该代码块实现 `_launch_server`，承担模块行为中的一个聚焦逻辑片段。
- `TestHiCacheSpecFileStorage._stop_server`: This block implements `_stop_server` and captures one focused piece of the module's behavior. / 该代码块实现 `_stop_server`，承担模块行为中的一个聚焦逻辑片段。
- `TestHiCacheSpecFileStorage._restart_server`: This block implements `_restart_server` and captures one focused piece of the module's behavior. / 该代码块实现 `_restart_server`，承担模块行为中的一个聚焦逻辑片段。
- `TestHiCacheSpecFileStorage._count_file_storage_pages`: This block implements `_count_file_storage_pages` and captures one focused piece of the module's behavior. / 该代码块实现 `_count_file_storage_pages`，承担模块行为中的一个聚焦逻辑片段。
- `TestHiCacheSpecFileStorage._wait_for_file_storage_pages`: This block implements `_wait_for_file_storage_pages` and captures one focused piece of the module's behavior. / 该代码块实现 `_wait_for_file_storage_pages`，承担模块行为中的一个聚焦逻辑片段。
- `TestHiCacheSpecFileStorage.tearDownClass`: This routine releases resources and restores state after the related tests finish. / 该流程会在相关测试结束后释放资源并恢复状态。
- `TestHiCacheSpecFileStorage._encode_without_special_tokens`: This block implements `_encode_without_special_tokens` and captures one focused piece of the module's behavior. / 该代码块实现 `_encode_without_special_tokens`，承担模块行为中的一个聚焦逻辑片段。
- `TestHiCacheSpecFileStorage._build_long_repetitive_prompt_ids`: This block implements `_build_long_repetitive_prompt_ids` and captures one focused piece of the module's behavior. / 该代码块实现 `_build_long_repetitive_prompt_ids`，承担模块行为中的一个聚焦逻辑片段。
- `TestHiCacheSpecFileStorage._send_long_prompt`: This block implements `_send_long_prompt` and captures one focused piece of the module's behavior. / 该代码块实现 `_send_long_prompt`，承担模块行为中的一个聚焦逻辑片段。
- `TestHiCacheSpecFileStorage._get_spec_accept_length`: This block implements `_get_spec_accept_length` and captures one focused piece of the module's behavior. / 该代码块实现 `_get_spec_accept_length`，承担模块行为中的一个聚焦逻辑片段。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `json`, `os`, `shutil`, `tempfile`, `time`, `unittest`, `typing`
- **Third-party modules / 第三方模块**: `psutil`, `requests`
- **Internal modules / 内部模块**: `sglang.benchmark.utils`, `sglang.srt.utils`, `sglang.test.ci.ci_register`, `sglang.test.test_utils`, `sglang.utils`

- **Total lines / 总行数**: 303
