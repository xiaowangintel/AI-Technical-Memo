# test_auth.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/unit/utils/test_auth.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates auth behavior in SGLang's unit / utils area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 单元 / 工具 领域中与 auth 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-1: supporting statements / 辅助语句
```python
"""Unit tests for srt/utils/auth.py — no server, no model loading."""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 3-12: module imports and dependencies / 模块导入与依赖
```python
import unittest

from sglang.srt.utils.auth import (
    AuthDecision,
    AuthLevel,
    auth_level,
    decide_request_auth,
)
from sglang.test.ci.ci_register import register_cpu_ci
from sglang.test.test_utils import CustomTestCase
```
**EN:** This block imports the modules needed by the rest of the file, including `unittest`, `sglang.srt.utils.auth`, `sglang.test.ci.ci_register`, `sglang.test.test_utils`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `unittest`, `sglang.srt.utils.auth`, `sglang.test.ci.ci_register`, `sglang.test.test_utils`。

### Lines 14-14: CI registration and metadata / CI 注册与元数据
```python
register_cpu_ci(1.0, "base-a-test-cpu")
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through register_cpu_ci.
**CN:** 该代码块通过 register_cpu_ci 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 17-17: class TestAuthDecision declaration / 类 TestAuthDecision 声明
```python
class TestAuthDecision(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 18-21: test case allowed default / 测试用例 allowed default
```python
    def test_allowed_default(self):
        decision = AuthDecision(allowed=True)
        self.assertTrue(decision.allowed)
        self.assertEqual(decision.error_status_code, 401)
```
**EN:** This test exercises `test_allowed_default` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_allowed_default`。

### Lines 23-26: test case not allowed with custom status / 测试用例 not allowed with custom status
```python
    def test_not_allowed_with_custom_status(self):
        decision = AuthDecision(allowed=False, error_status_code=403)
        self.assertFalse(decision.allowed)
        self.assertEqual(decision.error_status_code, 403)
```
**EN:** This test exercises `test_not_allowed_with_custom_status` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_not_allowed_with_custom_status`。

### Lines 28-31: test case frozen / 测试用例 frozen
```python
    def test_frozen(self):
        decision = AuthDecision(allowed=True)
        with self.assertRaises(AttributeError):
            decision.allowed = False
```
**EN:** This test exercises `test_frozen` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_frozen`。

### Lines 34-34: class TestAuthLevel declaration / 类 TestAuthLevel 声明
```python
class TestAuthLevel(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 35-38: test case enum values / 测试用例 enum values
```python
    def test_enum_values(self):
        self.assertEqual(AuthLevel.NORMAL.value, "normal")
        self.assertEqual(AuthLevel.ADMIN_OPTIONAL.value, "admin_optional")
        self.assertEqual(AuthLevel.ADMIN_FORCE.value, "admin_force")
```
**EN:** This test exercises `test_enum_values` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_enum_values`。

### Lines 40-43: test case is string enum / 测试用例 is string enum
```python
    def test_is_string_enum(self):
        self.assertIsInstance(AuthLevel.NORMAL, str)
        # str mixin allows direct comparison with string values
        self.assertEqual(AuthLevel.NORMAL, "normal")
```
**EN:** This test exercises `test_is_string_enum` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_is_string_enum`。

### Lines 46-46: class TestAuthLevelDecorator declaration / 类 TestAuthLevelDecorator 声明
```python
class TestAuthLevelDecorator(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 47-52: test case decorator sets auth level / 测试用例 decorator sets auth level
```python
    def test_decorator_sets_auth_level(self):
        @auth_level(AuthLevel.ADMIN_FORCE)
        def my_endpoint():
            pass

        self.assertEqual(my_endpoint._auth_level, AuthLevel.ADMIN_FORCE)
```
**EN:** This test exercises `test_decorator_sets_auth_level` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_decorator_sets_auth_level`。

### Lines 54-59: test case decorator preserves function / 测试用例 decorator preserves function
```python
    def test_decorator_preserves_function(self):
        @auth_level(AuthLevel.NORMAL)
        def my_endpoint():
            return 42

        self.assertEqual(my_endpoint(), 42)
```
**EN:** This test exercises `test_decorator_preserves_function` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_decorator_preserves_function`。

### Lines 62-62: class TestDecideRequestAuth declaration / 类 TestDecideRequestAuth 声明
```python
class TestDecideRequestAuth(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 63-63: supporting statements / 辅助语句
```python
    """Tests for the pure decide_request_auth function."""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 64-66: supporting source context / 辅助源码上下文
```python

    # ==================== Always-Allowed Paths ====================

```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 67-76: test case options method always allowed / 测试用例 options method always allowed
```python
    def test_options_method_always_allowed(self):
        decision = decide_request_auth(
            method="OPTIONS",
            path="/v1/chat/completions",
            authorization_header=None,
            api_key="secret",
            admin_api_key="admin-secret",
            auth_level=AuthLevel.ADMIN_FORCE,
        )
        self.assertTrue(decision.allowed)
```
**EN:** This test exercises `test_options_method_always_allowed` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_options_method_always_allowed`。

### Lines 78-87: test case health path always allowed / 测试用例 health path always allowed
```python
    def test_health_path_always_allowed(self):
        decision = decide_request_auth(
            method="GET",
            path="/health",
            authorization_header=None,
            api_key="secret",
            admin_api_key=None,
            auth_level=AuthLevel.NORMAL,
        )
        self.assertTrue(decision.allowed)
```
**EN:** This test exercises `test_health_path_always_allowed` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_health_path_always_allowed`。

### Lines 89-98: test case health subpath always allowed / 测试用例 health subpath always allowed
```python
    def test_health_subpath_always_allowed(self):
        decision = decide_request_auth(
            method="GET",
            path="/health_generate",
            authorization_header=None,
            api_key="secret",
            admin_api_key=None,
            auth_level=AuthLevel.NORMAL,
        )
        self.assertTrue(decision.allowed)
```
**EN:** This test exercises `test_health_subpath_always_allowed` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_health_subpath_always_allowed`。

### Lines 100-109: test case metrics path always allowed / 测试用例 metrics path always allowed
```python
    def test_metrics_path_always_allowed(self):
        decision = decide_request_auth(
            method="GET",
            path="/metrics",
            authorization_header=None,
            api_key="secret",
            admin_api_key=None,
            auth_level=AuthLevel.NORMAL,
        )
        self.assertTrue(decision.allowed)
```
**EN:** This test exercises `test_metrics_path_always_allowed` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_metrics_path_always_allowed`。

### Lines 110-112: supporting source context / 辅助源码上下文
```python

    # ==================== NORMAL Auth Level ====================

```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 113-122: test case normal no keys configured / 测试用例 normal no keys configured
```python
    def test_normal_no_keys_configured(self):
        decision = decide_request_auth(
            method="POST",
            path="/v1/chat/completions",
            authorization_header=None,
            api_key=None,
            admin_api_key=None,
            auth_level=AuthLevel.NORMAL,
        )
        self.assertTrue(decision.allowed)
```
**EN:** This test exercises `test_normal_no_keys_configured` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_normal_no_keys_configured`。

### Lines 124-133: test case normal with api key correct / 测试用例 normal with api key correct
```python
    def test_normal_with_api_key_correct(self):
        decision = decide_request_auth(
            method="POST",
            path="/v1/chat/completions",
            authorization_header="Bearer my-api-key",
            api_key="my-api-key",
            admin_api_key=None,
            auth_level=AuthLevel.NORMAL,
        )
        self.assertTrue(decision.allowed)
```
**EN:** This test exercises `test_normal_with_api_key_correct` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_normal_with_api_key_correct`。

### Lines 135-144: test case normal with api key wrong / 测试用例 normal with api key wrong
```python
    def test_normal_with_api_key_wrong(self):
        decision = decide_request_auth(
            method="POST",
            path="/v1/chat/completions",
            authorization_header="Bearer wrong-key",
            api_key="my-api-key",
            admin_api_key=None,
            auth_level=AuthLevel.NORMAL,
        )
        self.assertFalse(decision.allowed)
```
**EN:** This test exercises `test_normal_with_api_key_wrong` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_normal_with_api_key_wrong`。

### Lines 146-155: test case normal with api key missing header / 测试用例 normal with api key missing header
```python
    def test_normal_with_api_key_missing_header(self):
        decision = decide_request_auth(
            method="POST",
            path="/v1/chat/completions",
            authorization_header=None,
            api_key="my-api-key",
            admin_api_key=None,
            auth_level=AuthLevel.NORMAL,
        )
        self.assertFalse(decision.allowed)
```
**EN:** This test exercises `test_normal_with_api_key_missing_header` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_normal_with_api_key_missing_header`。

### Lines 157-167: test case normal only admin key configured / 测试用例 normal only admin key configured
```python
    def test_normal_only_admin_key_configured(self):
        """When only admin_api_key is configured, normal endpoints allow all."""
        decision = decide_request_auth(
            method="POST",
            path="/v1/chat/completions",
            authorization_header=None,
            api_key=None,
            admin_api_key="admin-secret",
            auth_level=AuthLevel.NORMAL,
        )
        self.assertTrue(decision.allowed)
```
**EN:** When only admin_api_key is configured, normal endpoints allow all. This test exercises `test_normal_only_admin_key_configured` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** When only admin_api_key is configured, normal endpoints allow all. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_normal_only_admin_key_configured`。

### Lines 168-170: supporting source context / 辅助源码上下文
```python

    # ==================== ADMIN_FORCE Auth Level ====================

```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 171-182: test case admin force no admin key configured / 测试用例 admin force no admin key configured
```python
    def test_admin_force_no_admin_key_configured(self):
        """ADMIN_FORCE without admin_api_key configured returns 403."""
        decision = decide_request_auth(
            method="POST",
            path="/admin/endpoint",
            authorization_header="Bearer my-api-key",
            api_key="my-api-key",
            admin_api_key=None,
            auth_level=AuthLevel.ADMIN_FORCE,
        )
        self.assertFalse(decision.allowed)
        self.assertEqual(decision.error_status_code, 403)
```
**EN:** ADMIN_FORCE without admin_api_key configured returns 403. This test exercises `test_admin_force_no_admin_key_configured` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** ADMIN_FORCE without admin_api_key configured returns 403. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_admin_force_no_admin_key_configured`。

### Lines 184-193: test case admin force correct admin key / 测试用例 admin force correct admin key
```python
    def test_admin_force_correct_admin_key(self):
        decision = decide_request_auth(
            method="POST",
            path="/admin/endpoint",
            authorization_header="Bearer admin-secret",
            api_key="my-api-key",
            admin_api_key="admin-secret",
            auth_level=AuthLevel.ADMIN_FORCE,
        )
        self.assertTrue(decision.allowed)
```
**EN:** This test exercises `test_admin_force_correct_admin_key` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_admin_force_correct_admin_key`。

### Lines 195-205: test case admin force wrong admin key / 测试用例 admin force wrong admin key
```python
    def test_admin_force_wrong_admin_key(self):
        decision = decide_request_auth(
            method="POST",
            path="/admin/endpoint",
            authorization_header="Bearer wrong-key",
            api_key="my-api-key",
            admin_api_key="admin-secret",
            auth_level=AuthLevel.ADMIN_FORCE,
        )
        self.assertFalse(decision.allowed)
        self.assertEqual(decision.error_status_code, 401)
```
**EN:** This test exercises `test_admin_force_wrong_admin_key` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_admin_force_wrong_admin_key`。

### Lines 207-217: test case admin force api key not accepted / 测试用例 admin force api key not accepted
```python
    def test_admin_force_api_key_not_accepted(self):
        """ADMIN_FORCE rejects api_key, only accepts admin_api_key."""
        decision = decide_request_auth(
            method="POST",
            path="/admin/endpoint",
            authorization_header="Bearer my-api-key",
            api_key="my-api-key",
            admin_api_key="admin-secret",
            auth_level=AuthLevel.ADMIN_FORCE,
        )
        self.assertFalse(decision.allowed)
```
**EN:** ADMIN_FORCE rejects api_key, only accepts admin_api_key. This test exercises `test_admin_force_api_key_not_accepted` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** ADMIN_FORCE rejects api_key, only accepts admin_api_key. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_admin_force_api_key_not_accepted`。

### Lines 218-220: supporting source context / 辅助源码上下文
```python

    # ==================== ADMIN_OPTIONAL Auth Level ====================

```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 221-230: test case admin optional no keys configured / 测试用例 admin optional no keys configured
```python
    def test_admin_optional_no_keys_configured(self):
        decision = decide_request_auth(
            method="POST",
            path="/admin/optional",
            authorization_header=None,
            api_key=None,
            admin_api_key=None,
            auth_level=AuthLevel.ADMIN_OPTIONAL,
        )
        self.assertTrue(decision.allowed)
```
**EN:** This test exercises `test_admin_optional_no_keys_configured` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_admin_optional_no_keys_configured`。

### Lines 232-241: test case admin optional only api key correct / 测试用例 admin optional only api key correct
```python
    def test_admin_optional_only_api_key_correct(self):
        decision = decide_request_auth(
            method="POST",
            path="/admin/optional",
            authorization_header="Bearer my-api-key",
            api_key="my-api-key",
            admin_api_key=None,
            auth_level=AuthLevel.ADMIN_OPTIONAL,
        )
        self.assertTrue(decision.allowed)
```
**EN:** This test exercises `test_admin_optional_only_api_key_correct` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_admin_optional_only_api_key_correct`。

### Lines 243-252: test case admin optional only api key wrong / 测试用例 admin optional only api key wrong
```python
    def test_admin_optional_only_api_key_wrong(self):
        decision = decide_request_auth(
            method="POST",
            path="/admin/optional",
            authorization_header="Bearer wrong-key",
            api_key="my-api-key",
            admin_api_key=None,
            auth_level=AuthLevel.ADMIN_OPTIONAL,
        )
        self.assertFalse(decision.allowed)
```
**EN:** This test exercises `test_admin_optional_only_api_key_wrong` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_admin_optional_only_api_key_wrong`。

### Lines 254-263: test case admin optional only admin key correct / 测试用例 admin optional only admin key correct
```python
    def test_admin_optional_only_admin_key_correct(self):
        decision = decide_request_auth(
            method="POST",
            path="/admin/optional",
            authorization_header="Bearer admin-secret",
            api_key=None,
            admin_api_key="admin-secret",
            auth_level=AuthLevel.ADMIN_OPTIONAL,
        )
        self.assertTrue(decision.allowed)
```
**EN:** This test exercises `test_admin_optional_only_admin_key_correct` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_admin_optional_only_admin_key_correct`。

### Lines 265-275: test case admin optional both keys requires admin / 测试用例 admin optional both keys requires admin
```python
    def test_admin_optional_both_keys_requires_admin(self):
        """When both keys configured, ADMIN_OPTIONAL requires admin_api_key."""
        decision = decide_request_auth(
            method="POST",
            path="/admin/optional",
            authorization_header="Bearer my-api-key",
            api_key="my-api-key",
            admin_api_key="admin-secret",
            auth_level=AuthLevel.ADMIN_OPTIONAL,
        )
        self.assertFalse(decision.allowed)
```
**EN:** When both keys configured, ADMIN_OPTIONAL requires admin_api_key. This test exercises `test_admin_optional_both_keys_requires_admin` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** When both keys configured, ADMIN_OPTIONAL requires admin_api_key. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_admin_optional_both_keys_requires_admin`。

### Lines 277-286: test case admin optional both keys admin accepted / 测试用例 admin optional both keys admin accepted
```python
    def test_admin_optional_both_keys_admin_accepted(self):
        decision = decide_request_auth(
            method="POST",
            path="/admin/optional",
            authorization_header="Bearer admin-secret",
            api_key="my-api-key",
            admin_api_key="admin-secret",
            auth_level=AuthLevel.ADMIN_OPTIONAL,
        )
        self.assertTrue(decision.allowed)
```
**EN:** This test exercises `test_admin_optional_both_keys_admin_accepted` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_admin_optional_both_keys_admin_accepted`。

### Lines 287-289: supporting source context / 辅助源码上下文
```python

    # ==================== Bearer Token Edge Cases ====================

```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 290-299: test case malformed authorization header / 测试用例 malformed authorization header
```python
    def test_malformed_authorization_header(self):
        decision = decide_request_auth(
            method="POST",
            path="/v1/chat/completions",
            authorization_header="NotBearer my-api-key",
            api_key="my-api-key",
            admin_api_key=None,
            auth_level=AuthLevel.NORMAL,
        )
        self.assertFalse(decision.allowed)
```
**EN:** This test exercises `test_malformed_authorization_header` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_malformed_authorization_header`。

### Lines 301-310: test case empty authorization header / 测试用例 empty authorization header
```python
    def test_empty_authorization_header(self):
        decision = decide_request_auth(
            method="POST",
            path="/v1/chat/completions",
            authorization_header="",
            api_key="my-api-key",
            admin_api_key=None,
            auth_level=AuthLevel.NORMAL,
        )
        self.assertFalse(decision.allowed)
```
**EN:** This test exercises `test_empty_authorization_header` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_empty_authorization_header`。

### Lines 312-321: test case bearer case insensitive / 测试用例 bearer case insensitive
```python
    def test_bearer_case_insensitive(self):
        decision = decide_request_auth(
            method="POST",
            path="/v1/chat/completions",
            authorization_header="BEARER my-api-key",
            api_key="my-api-key",
            admin_api_key=None,
            auth_level=AuthLevel.NORMAL,
        )
        self.assertTrue(decision.allowed)
```
**EN:** This test exercises `test_bearer_case_insensitive` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_bearer_case_insensitive`。

### Lines 324-325: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `TestAuthDecision`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestAuthLevel`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestAuthLevelDecorator`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestDecideRequestAuth`: Tests for the pure decide_request_auth function. / 用于组织相关测试、夹具或辅助方法。
- `TestAuthDecision.test_allowed_default`: This test exercises `test_allowed_default` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_allowed_default`。
- `TestAuthDecision.test_not_allowed_with_custom_status`: This test exercises `test_not_allowed_with_custom_status` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_not_allowed_with_custom_status`。
- `TestAuthDecision.test_frozen`: This test exercises `test_frozen` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_frozen`。
- `TestAuthLevel.test_enum_values`: This test exercises `test_enum_values` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_enum_values`。
- `TestAuthLevel.test_is_string_enum`: This test exercises `test_is_string_enum` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_is_string_enum`。
- `TestAuthLevelDecorator.test_decorator_sets_auth_level`: This test exercises `test_decorator_sets_auth_level` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_decorator_sets_auth_level`。
- `TestAuthLevelDecorator.test_decorator_preserves_function`: This test exercises `test_decorator_preserves_function` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_decorator_preserves_function`。
- `TestDecideRequestAuth.test_options_method_always_allowed`: This test exercises `test_options_method_always_allowed` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_options_method_always_allowed`。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `unittest`
- **Internal modules / 内部模块**: `sglang.srt.utils.auth`, `sglang.test.ci.ci_register`, `sglang.test.test_utils`

- **Total lines / 总行数**: 325
