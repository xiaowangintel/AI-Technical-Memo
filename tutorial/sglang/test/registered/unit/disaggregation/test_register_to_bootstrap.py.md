# test_register_to_bootstrap.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/unit/disaggregation/test_register_to_bootstrap.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates register to bootstrap behavior in SGLang's unit / disaggregation area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 单元 / disaggregation 领域中与 register to bootstrap 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-1: supporting statements / 辅助语句
```python
"""Unit tests for srt/disaggregation/common/conn — register_to_bootstrap retry logic."""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 3-3: module imports and dependencies / 模块导入与依赖
```python
from sglang.test.ci.ci_register import register_cpu_ci
```
**EN:** This block imports the modules needed by the rest of the file, including `sglang.test.ci.ci_register`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `sglang.test.ci.ci_register`。

### Lines 5-5: CI registration and metadata / CI 注册与元数据
```python
register_cpu_ci(est_time=5, suite="base-a-test-cpu")
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through register_cpu_ci.
**CN:** 该代码块通过 register_cpu_ci 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 7-10: module imports and dependencies / 模块导入与依赖
```python
import unittest
from unittest.mock import MagicMock, call, patch

from sglang.test.test_utils import CustomTestCase
```
**EN:** This block imports the modules needed by the rest of the file, including `unittest`, `unittest.mock`, `sglang.test.test_utils`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `unittest`, `unittest.mock`, `sglang.test.test_utils`。

### Lines 13-13: class TestRegisterToBootstrap declaration / 类 TestRegisterToBootstrap 声明
```python
class TestRegisterToBootstrap(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 14-14: supporting statements / 辅助语句
```python
    """Tests for CommonKVManager.register_to_bootstrap retry/backoff behavior."""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 16-28: test case succeeds on first attempt / 测试用例 succeeds on first attempt
```python
    @patch("sglang.srt.disaggregation.common.conn.time")
    @patch("sglang.srt.disaggregation.common.conn.requests.put")
    def test_succeeds_on_first_attempt(self, mock_put, mock_time):
        mock_time.monotonic.return_value = 0.0
        mock_response = MagicMock()
        mock_response.status_code = 200
        mock_put.return_value = mock_response

        mgr = self._make_manager()
        mgr.register_to_bootstrap()

        mock_put.assert_called_once()
        mock_time.sleep.assert_not_called()
```
**EN:** This test exercises `test_succeeds_on_first_attempt` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_succeeds_on_first_attempt`。

### Lines 30-44: test case succeeds after retries / 测试用例 succeeds after retries
```python
    @patch("sglang.srt.disaggregation.common.conn.time")
    @patch("sglang.srt.disaggregation.common.conn.requests.put")
    def test_succeeds_after_retries(self, mock_put, mock_time):
        mock_time.monotonic.return_value = 0.0
        fail_resp = MagicMock()
        fail_resp.status_code = 503
        success_resp = MagicMock()
        success_resp.status_code = 200
        mock_put.side_effect = [fail_resp, fail_resp, success_resp]

        mgr = self._make_manager()
        mgr.register_to_bootstrap()

        self.assertEqual(mock_put.call_count, 3)
        self.assertEqual(mock_time.sleep.call_count, 2)
```
**EN:** This test exercises `test_succeeds_after_retries` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_succeeds_after_retries`。

### Lines 46-59: test case all retries exhausted / 测试用例 all retries exhausted
```python
    @patch("sglang.srt.disaggregation.common.conn.time")
    @patch("sglang.srt.disaggregation.common.conn.requests.put")
    def test_all_retries_exhausted(self, mock_put, mock_time):
        mock_time.monotonic.return_value = 0.0
        fail_resp = MagicMock()
        fail_resp.status_code = 503
        mock_put.return_value = fail_resp

        mgr = self._make_manager()
        mgr.register_to_bootstrap()

        self.assertEqual(mock_put.call_count, 5)
        # Sleep is only called between attempts, not after the final failure
        self.assertEqual(mock_time.sleep.call_count, 4)
```
**EN:** This test exercises `test_all_retries_exhausted` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_all_retries_exhausted`。

### Lines 61-79: test case exception with nested cause / 测试用例 exception with nested cause
```python
    @patch("sglang.srt.disaggregation.common.conn.time")
    @patch("sglang.srt.disaggregation.common.conn.requests.put")
    def test_exception_with_nested_cause(self, mock_put, mock_time):
        mock_time.monotonic.return_value = 0.0

        root_exc = ConnectionRefusedError("connection refused")
        inner_exc = OSError("os error")
        inner_exc.__cause__ = root_exc
        outer_exc = Exception("wrapped")
        outer_exc.__cause__ = inner_exc

        success_resp = MagicMock()
        success_resp.status_code = 200
        mock_put.side_effect = [outer_exc, success_resp]

        mgr = self._make_manager()
        mgr.register_to_bootstrap()

        self.assertEqual(mock_put.call_count, 2)
```
**EN:** This test exercises `test_exception_with_nested_cause` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_exception_with_nested_cause`。

### Lines 81-96: test case exception with no cause / 测试用例 exception with no cause
```python
    @patch("sglang.srt.disaggregation.common.conn.time")
    @patch("sglang.srt.disaggregation.common.conn.requests.put")
    def test_exception_with_no_cause(self, mock_put, mock_time):
        mock_time.monotonic.return_value = 0.0

        exc = ConnectionError("plain connection error")
        exc.__cause__ = None

        success_resp = MagicMock()
        success_resp.status_code = 200
        mock_put.side_effect = [exc, success_resp]

        mgr = self._make_manager()
        mgr.register_to_bootstrap()

        self.assertEqual(mock_put.call_count, 2)
```
**EN:** This test exercises `test_exception_with_no_cause` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_exception_with_no_cause`。

### Lines 98-113: test case backoff delay exponential / 测试用例 backoff delay exponential
```python
    @patch("sglang.srt.disaggregation.common.conn.time")
    @patch("sglang.srt.disaggregation.common.conn.requests.put")
    def test_backoff_delay_exponential(self, mock_put, mock_time):
        mock_time.monotonic.return_value = 0.0
        fail_resp = MagicMock()
        fail_resp.status_code = 503
        mock_put.return_value = fail_resp

        mgr = self._make_manager()
        mgr.register_to_bootstrap()

        # With monotonic() = 0.0, jitter factor = 0.75 + 0.25 * (0.0 % 1) = 0.75
        # delay = min(1.0 * 2^attempt, 30.0) * 0.75
        # Sleep happens only between attempts (attempt 0..3), not after the final failure
        expected_calls = [call(0.75), call(1.5), call(3.0), call(6.0)]
        self.assertEqual(mock_time.sleep.call_args_list, expected_calls)
```
**EN:** This test exercises `test_backoff_delay_exponential` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_backoff_delay_exponential`。

### Lines 115-138: test case jitter never exceeds max delay / 测试用例 jitter never exceeds max delay
```python
    @patch("sglang.srt.disaggregation.common.conn.time")
    @patch("sglang.srt.disaggregation.common.conn.requests.put")
    def test_jitter_never_exceeds_max_delay(self, mock_put, mock_time):
        """Guard against operator-precedence regressions in the jitter factor.

        The jitter factor must stay in [0.75, 1.0), so a delay capped at
        max_delay must never exceed max_delay after applying jitter.
        """
        # monotonic() returns a value whose fractional part is close to 1.
        # If the parentheses around `time.monotonic() % 1` were dropped, the
        # jitter factor could grow up to ~1.75 and blow past max_delay.
        mock_time.monotonic.return_value = 999.9999
        fail_resp = MagicMock()
        fail_resp.status_code = 503
        mock_put.return_value = fail_resp

        mgr = self._make_manager()
        mgr.register_to_bootstrap()

        max_delay = 30.0
        for sleep_call in mock_time.sleep.call_args_list:
            actual_delay = sleep_call[0][0]
            self.assertLess(actual_delay, max_delay)
            self.assertGreaterEqual(actual_delay, 0.75)
```
**EN:** Guard against operator-precedence regressions in the jitter factor. This test exercises `test_jitter_never_exceeds_max_delay` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Guard against operator-precedence regressions in the jitter factor. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_jitter_never_exceeds_max_delay`。

### Lines 140-170: test case payload contains required fields / 测试用例 payload contains required fields
```python
    @patch("sglang.srt.disaggregation.common.conn.time")
    @patch("sglang.srt.disaggregation.common.conn.requests.put")
    def test_payload_contains_required_fields(self, mock_put, mock_time):
        mock_time.monotonic.return_value = 0.0
        success_resp = MagicMock()
        success_resp.status_code = 200
        mock_put.return_value = success_resp

        mgr = self._make_manager()
        mgr.register_to_bootstrap()

        call_kwargs = mock_put.call_args
        payload = call_kwargs[1]["json"]
        required_fields = [
            "attn_tp_size",
            "attn_tp_rank",
            "attn_cp_size",
            "attn_cp_rank",
            "attn_dp_size",
            "attn_dp_rank",
            "pp_size",
            "pp_rank",
            "system_dp_size",
            "system_dp_rank",
            "rank_ip",
            "rank_port",
            "page_size",
            "kv_cache_dtype",
        ]
        for field in required_fields:
            self.assertIn(field, payload)
```
**EN:** This test exercises `test_payload_contains_required_fields` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_payload_contains_required_fields`。

### Lines 172-184: test case url with dist init addr / 测试用例 url with dist init addr
```python
    @patch("sglang.srt.disaggregation.common.conn.time")
    @patch("sglang.srt.disaggregation.common.conn.requests.put")
    def test_url_with_dist_init_addr(self, mock_put, mock_time):
        mock_time.monotonic.return_value = 0.0
        success_resp = MagicMock()
        success_resp.status_code = 200
        mock_put.return_value = success_resp

        mgr = self._make_manager(dist_init_addr="10.0.0.1:12345")
        mgr.register_to_bootstrap()

        url_used = mock_put.call_args[0][0]
        self.assertIn("10.0.0.1", url_used)
```
**EN:** This test exercises `test_url_with_dist_init_addr` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_url_with_dist_init_addr`。

### Lines 186-222: method make manager / 方法 make manager
```python
    def _make_manager(self, dist_init_addr=None):
        """Create a lightweight mock manager that has the attributes needed
        by register_to_bootstrap, without going through CommonKVManager.__init__
        (which requires zmq, ServerArgs model resolution, etc.)."""
        from sglang.srt.disaggregation.common.conn import CommonKVManager

        mgr = MagicMock(spec=CommonKVManager)
        # Bind the real method to the mock
        mgr.register_to_bootstrap = CommonKVManager.register_to_bootstrap.__get__(
            mgr, CommonKVManager
        )

        # Set attributes that register_to_bootstrap reads
        mgr.dist_init_addr = dist_init_addr
        mgr.bootstrap_host = "127.0.0.1"
        mgr.bootstrap_port = 8765
        mgr.attn_tp_size = 1
        mgr.attn_tp_rank = 0
        mgr.attn_cp_size = 1
        mgr.attn_cp_rank = 0
        mgr.attn_dp_size = 1
        mgr.attn_dp_rank = 0
        mgr.pp_size = 1
        mgr.pp_rank = 0
        mgr.system_dp_size = 1
        mgr.system_dp_rank = 0
        mgr.local_ip = "127.0.0.1"
        mgr.rank_port = 12345

        mgr.kv_args = MagicMock()
        mgr.kv_args.page_size = 16

        mgr.server_args = MagicMock()
        mgr.server_args.kv_cache_dtype = "auto"
        mgr.server_args.load_balance_method = "follow_bootstrap_room"

        return mgr
```
**EN:** Create a lightweight mock manager that has the attributes needed by register_to_bootstrap, without going through CommonKVManager.__init__ (which requires zmq, ServerArgs model resolution, etc.). This block implements `_make_manager` and captures one focused piece of the module's behavior.
**CN:** Create a lightweight mock manager that has the attributes needed by register_to_bootstrap, without going through CommonKVManager.__init__ (which requires zmq, ServerArgs model resolution, etc.). 该代码块实现 `_make_manager`，承担模块行为中的一个聚焦逻辑片段。

### Lines 225-226: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `TestRegisterToBootstrap`: Tests for CommonKVManager.register_to_bootstrap retry/backoff behavior. / 用于组织相关测试、夹具或辅助方法。
- `TestRegisterToBootstrap.test_succeeds_on_first_attempt`: This test exercises `test_succeeds_on_first_attempt` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_succeeds_on_first_attempt`。
- `TestRegisterToBootstrap.test_succeeds_after_retries`: This test exercises `test_succeeds_after_retries` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_succeeds_after_retries`。
- `TestRegisterToBootstrap.test_all_retries_exhausted`: This test exercises `test_all_retries_exhausted` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_all_retries_exhausted`。
- `TestRegisterToBootstrap.test_exception_with_nested_cause`: This test exercises `test_exception_with_nested_cause` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_exception_with_nested_cause`。
- `TestRegisterToBootstrap.test_exception_with_no_cause`: This test exercises `test_exception_with_no_cause` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_exception_with_no_cause`。
- `TestRegisterToBootstrap.test_backoff_delay_exponential`: This test exercises `test_backoff_delay_exponential` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_backoff_delay_exponential`。
- `TestRegisterToBootstrap.test_jitter_never_exceeds_max_delay`: Guard against operator-precedence regressions in the jitter factor. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_jitter_never_exceeds_max_delay`。
- `TestRegisterToBootstrap.test_payload_contains_required_fields`: This test exercises `test_payload_contains_required_fields` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_payload_contains_required_fields`。
- `TestRegisterToBootstrap.test_url_with_dist_init_addr`: This test exercises `test_url_with_dist_init_addr` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_url_with_dist_init_addr`。
- `TestRegisterToBootstrap._make_manager`: Create a lightweight mock manager that has the attributes needed by register_to_bootstrap, without going through CommonKVManager.__init__ (which requires zmq, ServerArgs model resolution, etc.). / 该代码块实现 `_make_manager`，承担模块行为中的一个聚焦逻辑片段。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `unittest`, `unittest.mock`
- **Internal modules / 内部模块**: `sglang.test.ci.ci_register`, `sglang.test.test_utils`

- **Total lines / 总行数**: 226
