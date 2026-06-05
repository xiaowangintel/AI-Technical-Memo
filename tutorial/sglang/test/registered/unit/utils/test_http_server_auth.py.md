# test_http_server_auth.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/unit/utils/test_http_server_auth.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates http server auth behavior in SGLang's unit / utils area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 单元 / 工具 领域中与 http server auth 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-6: supporting statements / 辅助语句
```python
"""
Unit tests for HTTP server admin auth.

Usage:
    python3 -m pytest test/registered/unit/utils/test_http_server_auth.py -v
"""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 8-11: module imports and dependencies / 模块导入与依赖
```python
import unittest

from sglang.srt.utils.auth import AuthLevel, decide_request_auth
from sglang.test.ci.ci_register import register_cpu_ci
```
**EN:** This block imports the modules needed by the rest of the file, including `unittest`, `sglang.srt.utils.auth`, `sglang.test.ci.ci_register`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `unittest`, `sglang.srt.utils.auth`, `sglang.test.ci.ci_register`。

### Lines 13-13: CI registration and metadata / CI 注册与元数据
```python
register_cpu_ci(est_time=6, suite="base-a-test-cpu")
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through register_cpu_ci.
**CN:** 该代码块通过 register_cpu_ci 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 16-16: class TestHttpServerAdminAuth declaration / 类 TestHttpServerAdminAuth 声明
```python
class TestHttpServerAdminAuth(unittest.TestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `unittest.TestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `unittest.TestCase`。

### Lines 17-34: method decide / 方法 decide
```python
    def _decide(
        self,
        *,
        method: str,
        path: str,
        authorization_header: str | None,
        api_key: str | None,
        admin_api_key: str | None,
        auth_level: AuthLevel,
    ):
        return decide_request_auth(
            method=method,
            path=path,
            authorization_header=authorization_header,
            api_key=api_key,
            admin_api_key=admin_api_key,
            auth_level=auth_level,
        )
```
**EN:** This block implements `_decide` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_decide`，承担模块行为中的一个聚焦逻辑片段。

### Lines 36-69: test case no keys configured / 测试用例 no keys configured
```python
    def test_no_keys_configured(self):
        # No keys configured -> NORMAL + ADMIN_OPTIONAL are open (legacy),
        # but ADMIN_FORCE must be rejected (403) explicitly.
        self.assertTrue(
            self._decide(
                method="GET",
                path="/v1/models",
                authorization_header=None,
                api_key=None,
                admin_api_key=None,
                auth_level=AuthLevel.NORMAL,
            ).allowed
        )
        self.assertTrue(
            self._decide(
                method="POST",
                path="/admin_optional_demo",
                authorization_header=None,
                api_key=None,
                admin_api_key=None,
                auth_level=AuthLevel.ADMIN_OPTIONAL,
            ).allowed
        )

        d = self._decide(
            method="POST",
            path="/admin_force_demo",
            authorization_header=None,
            api_key=None,
            admin_api_key=None,
            auth_level=AuthLevel.ADMIN_FORCE,
        )
        self.assertFalse(d.allowed)
        self.assertEqual(d.error_status_code, 403)
```
**EN:** This test exercises `test_no_keys_configured` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_no_keys_configured`。

### Lines 71-126: test case api key only / 测试用例 api key only
```python
    def test_api_key_only(self):
        # api_key configured -> NORMAL requires api_key (legacy).
        self.assertFalse(
            self._decide(
                method="GET",
                path="/v1/models",
                authorization_header=None,
                api_key="user",
                admin_api_key=None,
                auth_level=AuthLevel.NORMAL,
            ).allowed
        )
        self.assertTrue(
            self._decide(
                method="GET",
                path="/v1/models",
                authorization_header="Bearer user",
                api_key="user",
                admin_api_key=None,
                auth_level=AuthLevel.NORMAL,
            ).allowed
        )

        # ADMIN_OPTIONAL requires api_key when only api_key is configured.
        self.assertFalse(
            self._decide(
                method="POST",
                path="/admin_optional_demo",
                authorization_header="Bearer wrong",
                api_key="user",
                admin_api_key=None,
                auth_level=AuthLevel.ADMIN_OPTIONAL,
            ).allowed
        )
        self.assertTrue(
            self._decide(
                method="POST",
                path="/admin_optional_demo",
                authorization_header="Bearer user",
                api_key="user",
                admin_api_key=None,
                auth_level=AuthLevel.ADMIN_OPTIONAL,
            ).allowed
        )

        # ADMIN_FORCE must be rejected even if api_key is configured (403).
        d = self._decide(
            method="POST",
            path="/admin_force_demo",
            authorization_header="Bearer user",
            api_key="user",
            admin_api_key=None,
            auth_level=AuthLevel.ADMIN_FORCE,
        )
        self.assertFalse(d.allowed)
        self.assertEqual(d.error_status_code, 403)
```
**EN:** This test exercises `test_api_key_only` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_api_key_only`。

### Lines 128-183: test case admin api key only / 测试用例 admin api key only
```python
    def test_admin_api_key_only(self):
        # admin_api_key only:
        # - normal endpoints open
        # - optional/force endpoints require admin_api_key
        self.assertTrue(
            self._decide(
                method="GET",
                path="/v1/models",
                authorization_header="Bearer user",
                api_key=None,
                admin_api_key="admin",
                auth_level=AuthLevel.NORMAL,
            ).allowed
        )
        self.assertTrue(
            self._decide(
                method="GET",
                path="/v1/models",
                authorization_header=None,
                api_key=None,
                admin_api_key="admin",
                auth_level=AuthLevel.NORMAL,
            ).allowed
        )

        # Optional endpoints require admin_api_key when admin_api_key is configured.
        self.assertTrue(
            self._decide(
                method="POST",
                path="/admin_optional_demo",
                authorization_header="Bearer admin",
                api_key=None,
                admin_api_key="admin",
                auth_level=AuthLevel.ADMIN_OPTIONAL,
            ).allowed
        )
        self.assertFalse(
            self._decide(
                method="POST",
                path="/admin_optional_demo",
                authorization_header="Bearer user",
                api_key=None,
                admin_api_key="admin",
                auth_level=AuthLevel.ADMIN_OPTIONAL,
            ).allowed
        )

        d = self._decide(
            method="POST",
            path="/admin_force_demo",
            authorization_header="Bearer admin",
            api_key=None,
            admin_api_key="admin",
            auth_level=AuthLevel.ADMIN_FORCE,
        )
        self.assertTrue(d.allowed)
```
**EN:** This test exercises `test_admin_api_key_only` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_admin_api_key_only`。

### Lines 185-250: test case with both api keys / 测试用例 with both api keys
```python
    def test_with_both_api_keys(self):
        # both api_key and admin_api_key configured:
        # - normal endpoints require api_key
        # - optional endpoints require admin_api_key (api_key is NOT accepted)
        # - force endpoints require admin_api_key
        self.assertTrue(
            self._decide(
                method="GET",
                path="/v1/models",
                authorization_header="Bearer user",
                api_key="user",
                admin_api_key="admin",
                auth_level=AuthLevel.NORMAL,
            ).allowed
        )
        self.assertFalse(
            self._decide(
                method="GET",
                path="/v1/models",
                authorization_header="Bearer admin",
                api_key="user",
                admin_api_key="admin",
                auth_level=AuthLevel.NORMAL,
            ).allowed
        )
        # Optional endpoints must require admin_api_key when both keys are configured.
        self.assertFalse(
            self._decide(
                method="POST",
                path="/admin_optional_demo",
                authorization_header="Bearer user",
                api_key="user",
                admin_api_key="admin",
                auth_level=AuthLevel.ADMIN_OPTIONAL,
            ).allowed
        )
        self.assertTrue(
            self._decide(
                method="POST",
                path="/admin_optional_demo",
                authorization_header="Bearer admin",
                api_key="user",
                admin_api_key="admin",
                auth_level=AuthLevel.ADMIN_OPTIONAL,
            ).allowed
        )
        self.assertFalse(
            self._decide(
                method="POST",
                path="/admin_force_demo",
                authorization_header="Bearer user",
                api_key="user",
                admin_api_key="admin",
                auth_level=AuthLevel.ADMIN_FORCE,
            ).allowed
        )
        self.assertTrue(
            self._decide(
                method="POST",
                path="/admin_force_demo",
                authorization_header="Bearer admin",
                api_key="user",
                admin_api_key="admin",
                auth_level=AuthLevel.ADMIN_FORCE,
            ).allowed
        )
```
**EN:** This test exercises `test_with_both_api_keys` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_with_both_api_keys`。

### Lines 252-263: test case options is always allowed / 测试用例 options is always allowed
```python
    def test_options_is_always_allowed(self):
        # CORS preflight should never be blocked.
        self.assertTrue(
            self._decide(
                method="OPTIONS",
                path="/v1/models",
                authorization_header=None,
                api_key="user",
                admin_api_key="admin",
                auth_level=AuthLevel.ADMIN_FORCE,
            ).allowed
        )
```
**EN:** This test exercises `test_options_is_always_allowed` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_options_is_always_allowed`。

### Lines 265-292: test case health and metrics are always allowed / 测试用例 health and metrics are always allowed
```python
    def test_health_and_metrics_are_always_allowed(self):
        # Health/metrics endpoints are always public by design, regardless of auth level / keys.
        combos = [
            dict(api_key=None, admin_api_key=None),
            dict(api_key="user", admin_api_key=None),
            dict(api_key=None, admin_api_key="admin"),
            dict(api_key="user", admin_api_key="admin"),
        ]
        paths_allowed = [
            "/health",
            "/health_generate",
            "/metrics",
            "/metrics/",
            "/metrics/prometheus",
        ]
        for keys in combos:
            for path in paths_allowed:
                self.assertTrue(
                    self._decide(
                        method="GET",
                        path=path,
                        authorization_header=None,
                        api_key=keys["api_key"],
                        admin_api_key=keys["admin_api_key"],
                        auth_level=AuthLevel.ADMIN_FORCE,
                    ).allowed,
                    msg=f"expected allowed for {path=} with {keys=}",
                )
```
**EN:** This test exercises `test_health_and_metrics_are_always_allowed` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_health_and_metrics_are_always_allowed`。

### Lines 295-296: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `TestHttpServerAdminAuth`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestHttpServerAdminAuth._decide`: This block implements `_decide` and captures one focused piece of the module's behavior. / 该代码块实现 `_decide`，承担模块行为中的一个聚焦逻辑片段。
- `TestHttpServerAdminAuth.test_no_keys_configured`: This test exercises `test_no_keys_configured` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_no_keys_configured`。
- `TestHttpServerAdminAuth.test_api_key_only`: This test exercises `test_api_key_only` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_api_key_only`。
- `TestHttpServerAdminAuth.test_admin_api_key_only`: This test exercises `test_admin_api_key_only` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_admin_api_key_only`。
- `TestHttpServerAdminAuth.test_with_both_api_keys`: This test exercises `test_with_both_api_keys` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_with_both_api_keys`。
- `TestHttpServerAdminAuth.test_options_is_always_allowed`: This test exercises `test_options_is_always_allowed` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_options_is_always_allowed`。
- `TestHttpServerAdminAuth.test_health_and_metrics_are_always_allowed`: This test exercises `test_health_and_metrics_are_always_allowed` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_health_and_metrics_are_always_allowed`。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `unittest`
- **Internal modules / 内部模块**: `sglang.srt.utils.auth`, `sglang.test.ci.ci_register`

- **Total lines / 总行数**: 296
