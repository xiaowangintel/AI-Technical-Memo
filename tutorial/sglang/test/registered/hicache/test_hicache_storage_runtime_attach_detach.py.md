# test_hicache_storage_runtime_attach_detach.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/hicache/test_hicache_storage_runtime_attach_detach.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates hicache storage runtime attach detach behavior in SGLang's hicache area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 hicache 领域中与 hicache storage runtime attach detach 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-10: supporting statements / 辅助语句
```python
"""
E2E check for HiCache storage runtime attach/detach.

This test launches an SGLang server with hierarchical cache enabled but WITHOUT
any storage backend at startup, then attaches/detaches a storage backend via the
HTTP endpoints.

Usage:
    python3 -m pytest test/registered/hicache/test_hicache_storage_runtime_attach_detach.py -v
"""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 12-29: module imports and dependencies / 模块导入与依赖
```python
import json
import os
import tempfile
import time
import unittest
from urllib import error, request

from sglang.srt.utils import kill_process_tree
from sglang.test.ci.ci_register import register_cuda_ci
from sglang.test.test_utils import (
    DEFAULT_MODEL_NAME_FOR_TEST,
    DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
    DEFAULT_URL_FOR_TEST,
    CustomTestCase,
    find_available_port,
    popen_launch_server,
)
from sglang.utils import wait_for_http_ready
```
**EN:** This block imports the modules needed by the rest of the file, including `json`, `os`, `tempfile`, `time`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `json`, `os`, `tempfile`, `time`。

### Lines 31-31: CI registration and metadata / CI 注册与元数据
```python
register_cuda_ci(est_time=139, stage="base-b", runner_config="2-gpu-large")
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through register_cuda_ci.
**CN:** 该代码块通过 register_cuda_ci 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 34-34: class TestHiCacheStorageRuntimeAttachDetach declaration / 类 TestHiCacheStorageRuntimeAttachDetach 声明
```python
class TestHiCacheStorageRuntimeAttachDetach(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 35-63: setUpClass setup routine / setUpClass 初始化流程
```python
    @classmethod
    def setUpClass(cls):
        cls.temp_dir = tempfile.mkdtemp()
        cls.model = DEFAULT_MODEL_NAME_FOR_TEST
        # Use a per-test-class available port to reduce flakiness / conflicts.
        default_port = int(DEFAULT_URL_FOR_TEST.rsplit(":", 1)[1])
        cls.base_url = f"http://127.0.0.1:{find_available_port(default_port)}"

        cls.other_args = [
            "--enable-hierarchical-cache",
            "--mem-fraction-static",
            "0.6",
            "--hicache-ratio",
            "1.2",
            "--hicache-size",
            "100",
            "--page-size",
            "64",
            "--enable-cache-report",
            # NOTE: do NOT pass --hicache-storage-backend* here
        ]

        cls.env = {
            **os.environ,
            # File backend uses this env var to decide where to store cache pages.
            "SGLANG_HICACHE_FILE_BACKEND_STORAGE_DIR": cls.temp_dir,
            # Make runs less flaky for CI/dev.
            "SGLANG_ENABLE_DETERMINISTIC_INFERENCE": "1",
        }
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 65-69: tearDownClass cleanup routine / tearDownClass 清理流程
```python
    @classmethod
    def tearDownClass(cls):
        import shutil

        shutil.rmtree(cls.temp_dir, ignore_errors=True)
```
**EN:** This routine releases resources and restores state after the related tests finish.
**CN:** 该流程会在相关测试结束后释放资源并恢复状态。

### Lines 71-80: method wait for server ready / 方法 wait for server ready
```python
    @classmethod
    def _wait_for_server_ready(
        cls, base_url: str, timeout: int = 60, process=None
    ) -> bool:
        wait_for_http_ready(
            url=f"{base_url}/health",
            timeout=timeout,
            process=process,
        )
        return True
```
**EN:** This block implements `_wait_for_server_ready` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_wait_for_server_ready`，承担模块行为中的一个聚焦逻辑片段。

### Lines 82-90: method http get / 方法 http get
```python
    @staticmethod
    def _http_get(url: str, timeout: int = 10, headers: dict | None = None):
        try:
            req = request.Request(url, headers=headers or {}, method="GET")
            with request.urlopen(req, timeout=timeout) as resp:
                return resp.getcode(), resp.read().decode("utf-8", errors="replace")
        except error.HTTPError as e:
            body = e.read().decode("utf-8", errors="replace")
            return e.code, body
```
**EN:** This block implements `_http_get` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_http_get`，承担模块行为中的一个聚焦逻辑片段。

### Lines 92-105: method http post json / 方法 http post json
```python
    @staticmethod
    def _http_post_json(url: str, payload: dict | None = None, timeout: int = 30):
        data = None
        headers = {}
        if payload is not None:
            data = json.dumps(payload).encode("utf-8")
            headers["Content-Type"] = "application/json"
        req = request.Request(url, data=data, headers=headers, method="POST")
        try:
            with request.urlopen(req, timeout=timeout) as resp:
                return resp.getcode(), resp.read().decode("utf-8", errors="replace")
        except error.HTTPError as e:
            body = e.read().decode("utf-8", errors="replace")
            return e.code, body
```
**EN:** This block implements `_http_post_json` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_http_post_json`，承担模块行为中的一个聚焦逻辑片段。

### Lines 107-125: method http post json with headers / 方法 http post json with headers
```python
    @staticmethod
    def _http_post_json_with_headers(
        url: str,
        payload: dict | None = None,
        timeout: int = 30,
        headers: dict | None = None,
    ):
        data = None
        all_headers = dict(headers or {})
        if payload is not None:
            data = json.dumps(payload).encode("utf-8")
            all_headers["Content-Type"] = "application/json"
        req = request.Request(url, data=data, headers=all_headers, method="POST")
        try:
            with request.urlopen(req, timeout=timeout) as resp:
                return resp.getcode(), resp.read().decode("utf-8", errors="replace")
        except error.HTTPError as e:
            body = e.read().decode("utf-8", errors="replace")
            return e.code, body
```
**EN:** This block implements `_http_post_json_with_headers` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_http_post_json_with_headers`，承担模块行为中的一个聚焦逻辑片段。

### Lines 127-145: method http put json with headers / 方法 http put json with headers
```python
    @staticmethod
    def _http_put_json_with_headers(
        url: str,
        payload: dict | None = None,
        timeout: int = 30,
        headers: dict | None = None,
    ):
        data = None
        all_headers = dict(headers or {})
        if payload is not None:
            data = json.dumps(payload).encode("utf-8")
            all_headers["Content-Type"] = "application/json"
        req = request.Request(url, data=data, headers=all_headers, method="PUT")
        try:
            with request.urlopen(req, timeout=timeout) as resp:
                return resp.getcode(), resp.read().decode("utf-8", errors="replace")
        except error.HTTPError as e:
            body = e.read().decode("utf-8", errors="replace")
            return e.code, body
```
**EN:** This block implements `_http_put_json_with_headers` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_http_put_json_with_headers`，承担模块行为中的一个聚焦逻辑片段。

### Lines 147-158: method http delete with headers / 方法 http delete with headers
```python
    @staticmethod
    def _http_delete_with_headers(
        url: str, timeout: int = 30, headers: dict | None = None
    ):
        all_headers = dict(headers or {})
        req = request.Request(url, headers=all_headers, method="DELETE")
        try:
            with request.urlopen(req, timeout=timeout) as resp:
                return resp.getcode(), resp.read().decode("utf-8", errors="replace")
        except error.HTTPError as e:
            body = e.read().decode("utf-8", errors="replace")
            return e.code, body
```
**EN:** This block implements `_http_delete_with_headers` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_http_delete_with_headers`，承担模块行为中的一个聚焦逻辑片段。

### Lines 160-165: method get backend status / 方法 get backend status
```python
    def _get_backend_status(self, base_url: str, headers: dict | None = None):
        code, body = self._http_get(
            f"{base_url}/hicache/storage-backend", timeout=10, headers=headers
        )
        self.assertEqual(code, 200, body)
        return json.loads(body)
```
**EN:** This block implements `_get_backend_status` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_get_backend_status`，承担模块行为中的一个聚焦逻辑片段。

### Lines 167-187: method attach backend / 方法 attach backend
```python
    def _attach_backend(
        self,
        base_url: str,
        backend: str,
        extra_cfg: dict,
        prefetch_policy: str = "timeout",
        write_policy: str = "write_through",
        headers: dict | None = None,
    ):
        payload = {
            "hicache_storage_backend": backend,
            "hicache_storage_backend_extra_config_json": json.dumps(extra_cfg),
            "hicache_storage_prefetch_policy": prefetch_policy,
            "hicache_write_policy": write_policy,
        }
        return self._http_put_json_with_headers(
            f"{base_url}/hicache/storage-backend",
            payload,
            timeout=30,
            headers=headers,
        )
```
**EN:** This block implements `_attach_backend` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_attach_backend`，承担模块行为中的一个聚焦逻辑片段。

### Lines 189-194: method detach backend / 方法 detach backend
```python
    def _detach_backend(self, base_url: str, headers: dict | None = None):
        return self._http_delete_with_headers(
            f"{base_url}/hicache/storage-backend",
            timeout=30,
            headers=headers,
        )
```
**EN:** This block implements `_detach_backend` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_detach_backend`，承担模块行为中的一个聚焦逻辑片段。

### Lines 196-275: test case runtime attach detach (part 1/3) / 测试用例 runtime attach detach（第 1/3 部分）
```python
    def test_runtime_attach_detach(self):
        # Phase A: WITHOUT --admin-api-key, ADMIN_FORCE endpoints must be forbidden (403).
        process1 = popen_launch_server(
            self.model,
            self.base_url,
            timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
            other_args=self.other_args,
            env=self.env,
        )
        try:
            self._wait_for_server_ready(self.base_url, process=process1)

            code_info, _body_info = self._http_get(
                f"{self.base_url}/hicache/storage-backend", timeout=10
            )
            self.assertEqual(code_info, 400)
            code_attach_no_admin, _body_attach_no_admin = self._attach_backend(
                base_url=self.base_url, backend="file", extra_cfg={}
            )
            self.assertEqual(code_attach_no_admin, 400)
            code_detach_no_admin, _body_detach_no_admin = self._detach_backend(
                self.base_url
            )
            self.assertEqual(code_detach_no_admin, 400)
        finally:
            kill_process_tree(process1.pid)
            time.sleep(2)

        # Phase B: WITH --admin-api-key, must provide Authorization: Bearer <admin_key>.
        admin_key = "sglang-test-admin-key"
        base_url2 = f"http://127.0.0.1:{find_available_port(int(self.base_url.rsplit(':', 1)[1]) + 1)}"
        other_args2 = list(self.other_args) + ["--admin-api-key", admin_key]
        process2 = popen_launch_server(
            self.model,
            base_url2,
            timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
            other_args=other_args2,
            env=self.env,
        )
        try:
            self._wait_for_server_ready(base_url2, process=process2)

            # 1) Initially disabled (but unauthorized without admin key)
            code_info2_unauth, _ = self._http_get(
                f"{base_url2}/hicache/storage-backend", timeout=10
            )
            self.assertEqual(code_info2_unauth, 401)

            admin_headers = {"Authorization": f"Bearer {admin_key}"}
            status0 = self._get_backend_status(base_url2, headers=admin_headers)
            self.assertIsNone(status0.get("hicache_storage_backend"))

            # 2) Attach should succeed when idle
            extra_cfg = {
                "hicache_storage_pass_prefix_keys": True,
                # keep knobs small and stable
                "prefetch_threshold": 256,
                "prefetch_timeout_base": 3,
                "prefetch_timeout_per_ki_token": 0.01,
            }

            # Unauthorized attach must fail.
            code_attach_unauth, _ = self._attach_backend(
                base_url=base_url2, backend="file", extra_cfg=extra_cfg
            )
            self.assertEqual(code_attach_unauth, 401)

            code_attach, body_attach = self._attach_backend(
                base_url=base_url2,
                backend="file",
                extra_cfg=extra_cfg,
                prefetch_policy="timeout",
                write_policy="write_back",
                headers=admin_headers,
            )
            self.assertEqual(code_attach, 200, f"{code_attach} - {body_attach}")

            status1 = self._get_backend_status(base_url2, headers=admin_headers)
            self.assertEqual(status1.get("hicache_storage_backend"), "file")
            self.assertEqual(
```
**EN:** This test exercises `test_runtime_attach_detach` by arranging inputs, invoking the relevant path, and checking the expected outcome. This chunk continues part 1 of the same logical block.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_runtime_attach_detach`。 这一段对应同一逻辑块的第 1 部分。

### Lines 276-355: test case runtime attach detach (part 2/3) / 测试用例 runtime attach detach（第 2/3 部分）
```python
                status1.get("hicache_storage_backend_extra_config"),
                json.dumps(extra_cfg),
            )
            self.assertEqual(status1.get("hicache_storage_prefetch_policy"), "timeout")
            self.assertEqual(status1.get("hicache_write_policy"), "write_back")

            # 3) Attach again succeeds with policies updated
            code_attach_again, body_attach_again = self._attach_backend(
                base_url=base_url2,
                backend="file",
                extra_cfg=extra_cfg,
                prefetch_policy="wait_complete",
                write_policy="write_through_selective",
                headers=admin_headers,
            )
            self.assertEqual(
                code_attach_again, 200, f"{code_attach_again} - {body_attach_again}"
            )

            status2 = self._get_backend_status(base_url2, headers=admin_headers)
            self.assertEqual(
                status2.get("hicache_storage_backend_extra_config"),
                json.dumps(extra_cfg),
            )
            self.assertEqual(
                status2.get("hicache_storage_prefetch_policy"), "wait_complete"
            )
            self.assertEqual(
                status2.get("hicache_write_policy"), "write_through_selective"
            )

            # 4) Attach again with different backend should be rejected
            code_attach_again, body_attach_again = self._attach_backend(
                base_url=base_url2,
                backend="mooncake",
                extra_cfg=extra_cfg,
                headers=admin_headers,
            )
            self.assertNotEqual(code_attach_again, 200, body_attach_again)

            # 5) Detach should succeed and be idempotent
            code_detach, body_detach = self._detach_backend(
                base_url2, headers=admin_headers
            )
            self.assertEqual(code_detach, 200, f"{code_detach} - {body_detach}")
            status3 = self._get_backend_status(base_url2, headers=admin_headers)
            self.assertIsNone(status3.get("hicache_storage_backend"))
            self.assertEqual(
                status3.get("hicache_storage_prefetch_policy"), "wait_complete"
            )
            self.assertEqual(
                status3.get("hicache_write_policy"), "write_through_selective"
            )

            code_detach_again, body_detach_again = self._detach_backend(
                base_url2, headers=admin_headers
            )
            self.assertEqual(
                code_detach_again,
                200,
                f"{code_detach_again} - {body_detach_again}",
            )

            # 6) Re-attach after detach should succeed
            code_attach2, body_attach2 = self._attach_backend(
                base_url=base_url2,
                backend="file",
                extra_cfg=extra_cfg,
                headers=admin_headers,
            )
            self.assertEqual(code_attach2, 200, f"{code_attach2} - {body_attach2}")
            status4 = self._get_backend_status(base_url2, headers=admin_headers)
            self.assertEqual(status4.get("hicache_storage_backend"), "file")
            self.assertEqual(
                status4.get("hicache_storage_backend_extra_config"),
                json.dumps(extra_cfg),
            )
            self.assertEqual(status4.get("hicache_storage_prefetch_policy"), "timeout")
            self.assertEqual(status4.get("hicache_write_policy"), "write_through")

```
**EN:** This test exercises `test_runtime_attach_detach` by arranging inputs, invoking the relevant path, and checking the expected outcome. This chunk continues part 2 of the same logical block.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_runtime_attach_detach`。 这一段对应同一逻辑块的第 2 部分。

### Lines 356-363: test case runtime attach detach (part 3/3) / 测试用例 runtime attach detach（第 3/3 部分）
```python
            # Cleanup: detach for test isolation
            code_detach2, body_detach2 = self._detach_backend(
                base_url2, headers=admin_headers
            )
            self.assertEqual(code_detach2, 200, f"{code_detach2} - {body_detach2}")
        finally:
            kill_process_tree(process2.pid)
            time.sleep(2)
```
**EN:** This test exercises `test_runtime_attach_detach` by arranging inputs, invoking the relevant path, and checking the expected outcome. This chunk continues part 3 of the same logical block.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_runtime_attach_detach`。 这一段对应同一逻辑块的第 3 部分。

### Lines 366-367: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    unittest.main(verbosity=2)
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `TestHiCacheStorageRuntimeAttachDetach`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestHiCacheStorageRuntimeAttachDetach.setUpClass`: This routine prepares shared fixtures, models, or runtime state before the assertions execute. / 该流程会在断言执行前准备共享夹具、模型或运行状态。
- `TestHiCacheStorageRuntimeAttachDetach.tearDownClass`: This routine releases resources and restores state after the related tests finish. / 该流程会在相关测试结束后释放资源并恢复状态。
- `TestHiCacheStorageRuntimeAttachDetach._wait_for_server_ready`: This block implements `_wait_for_server_ready` and captures one focused piece of the module's behavior. / 该代码块实现 `_wait_for_server_ready`，承担模块行为中的一个聚焦逻辑片段。
- `TestHiCacheStorageRuntimeAttachDetach._http_get`: This block implements `_http_get` and captures one focused piece of the module's behavior. / 该代码块实现 `_http_get`，承担模块行为中的一个聚焦逻辑片段。
- `TestHiCacheStorageRuntimeAttachDetach._http_post_json`: This block implements `_http_post_json` and captures one focused piece of the module's behavior. / 该代码块实现 `_http_post_json`，承担模块行为中的一个聚焦逻辑片段。
- `TestHiCacheStorageRuntimeAttachDetach._http_post_json_with_headers`: This block implements `_http_post_json_with_headers` and captures one focused piece of the module's behavior. / 该代码块实现 `_http_post_json_with_headers`，承担模块行为中的一个聚焦逻辑片段。
- `TestHiCacheStorageRuntimeAttachDetach._http_put_json_with_headers`: This block implements `_http_put_json_with_headers` and captures one focused piece of the module's behavior. / 该代码块实现 `_http_put_json_with_headers`，承担模块行为中的一个聚焦逻辑片段。
- `TestHiCacheStorageRuntimeAttachDetach._http_delete_with_headers`: This block implements `_http_delete_with_headers` and captures one focused piece of the module's behavior. / 该代码块实现 `_http_delete_with_headers`，承担模块行为中的一个聚焦逻辑片段。
- `TestHiCacheStorageRuntimeAttachDetach._get_backend_status`: This block implements `_get_backend_status` and captures one focused piece of the module's behavior. / 该代码块实现 `_get_backend_status`，承担模块行为中的一个聚焦逻辑片段。
- `TestHiCacheStorageRuntimeAttachDetach._attach_backend`: This block implements `_attach_backend` and captures one focused piece of the module's behavior. / 该代码块实现 `_attach_backend`，承担模块行为中的一个聚焦逻辑片段。
- `TestHiCacheStorageRuntimeAttachDetach._detach_backend`: This block implements `_detach_backend` and captures one focused piece of the module's behavior. / 该代码块实现 `_detach_backend`，承担模块行为中的一个聚焦逻辑片段。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `json`, `os`, `tempfile`, `time`, `unittest`, `urllib`
- **Internal modules / 内部模块**: `sglang.srt.utils`, `sglang.test.ci.ci_register`, `sglang.test.test_utils`, `sglang.utils`

- **Total lines / 总行数**: 367
