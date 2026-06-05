# test_abort_with_metrics.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/scheduler/test_abort_with_metrics.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates abort with metrics behavior in SGLang's scheduler area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 scheduler 领域中与 abort with metrics 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-10: supporting statements / 辅助语句
```python
"""
Unit test for _PureASGIDispatch: verify that the ASGI ``receive`` callable
is passed through untouched so that request.is_disconnected() works.

Background: @app.middleware("http") wraps handlers with BaseHTTPMiddleware
whose call_next() replaces the ASGI ``receive``, breaking
request.is_disconnected() and preventing non-streaming abort on client
disconnect.  _PureASGIDispatch fixes this.  The existing test_abort.py
already covers the full e2e abort flow.
"""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 12-19: module imports and dependencies / 模块导入与依赖
```python
import asyncio
import unittest

from starlette.requests import Request

from sglang.srt.utils.http_middleware_patch import _PureASGIDispatch
from sglang.test.ci.ci_register import register_cpu_ci
from sglang.test.test_utils import CustomTestCase
```
**EN:** This block imports the modules needed by the rest of the file, including `asyncio`, `unittest`, `starlette.requests`, `sglang.srt.utils.http_middleware_patch`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `asyncio`, `unittest`, `starlette.requests`, `sglang.srt.utils.http_middleware_patch`。

### Lines 21-32: CI registration and metadata / CI 注册与元数据
```python
register_cpu_ci(est_time=7, suite="base-a-test-cpu")

_HTTP_SCOPE = {
    "type": "http",
    "asgi": {"version": "3.0"},
    "http_version": "1.1",
    "method": "POST",
    "path": "/test",
    "query_string": b"",
    "root_path": "",
    "headers": [],
}
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through register_cpu_ci.
**CN:** 该代码块通过 register_cpu_ci 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 35-35: class TestPureASGIDispatchReceivePassthrough declaration / 类 TestPureASGIDispatchReceivePassthrough 声明
```python
class TestPureASGIDispatchReceivePassthrough(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 36-36: supporting statements / 辅助语句
```python
    """Verify _PureASGIDispatch passes ``receive`` through untouched."""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 38-62: method run with receive / 方法 run with receive
```python
    @staticmethod
    async def _run_with_receive(receive_msg):
        """Invoke _PureASGIDispatch and return request.is_disconnected()."""
        result = {}

        async def dispatch(request: Request, call_next):
            result["disconnected"] = await request.is_disconnected()
            await call_next(request)

        async def inner_app(scope, receive, send):
            await send({"type": "http.response.start", "status": 200, "headers": []})
            await send({"type": "http.response.body", "body": b""})

        middleware = _PureASGIDispatch(inner_app, dispatch=dispatch)

        async def receive():
            return receive_msg

        sent = []

        async def send(msg):
            sent.append(msg)

        await middleware(_HTTP_SCOPE, receive, send)
        return result["disconnected"]
```
**EN:** Invoke _PureASGIDispatch and return request.is_disconnected(). This block implements `_run_with_receive` and captures one focused piece of the module's behavior.
**CN:** Invoke _PureASGIDispatch and return request.is_disconnected(). 该代码块实现 `_run_with_receive`，承担模块行为中的一个聚焦逻辑片段。

### Lines 64-68: test case is disconnected on client disconnect / 测试用例 is disconnected on client disconnect
```python
    def test_is_disconnected_on_client_disconnect(self):
        """receive() -> http.disconnect: is_disconnected() must return True."""
        self.assertTrue(
            asyncio.run(self._run_with_receive({"type": "http.disconnect"}))
        )
```
**EN:** receive() -> http.disconnect: is_disconnected() must return True. This test exercises `test_is_disconnected_on_client_disconnect` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** receive() -> http.disconnect: is_disconnected() must return True. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_is_disconnected_on_client_disconnect`。

### Lines 70-74: test case not disconnected when connected / 测试用例 not disconnected when connected
```python
    def test_not_disconnected_when_connected(self):
        """receive() -> http.request: is_disconnected() must return False."""
        self.assertFalse(
            asyncio.run(self._run_with_receive({"type": "http.request", "body": b""}))
        )
```
**EN:** receive() -> http.request: is_disconnected() must return False. This test exercises `test_not_disconnected_when_connected` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** receive() -> http.request: is_disconnected() must return False. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_not_disconnected_when_connected`。

### Lines 77-78: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `TestPureASGIDispatchReceivePassthrough`: Verify _PureASGIDispatch passes ``receive`` through untouched. / 用于组织相关测试、夹具或辅助方法。
- `TestPureASGIDispatchReceivePassthrough._run_with_receive`: Invoke _PureASGIDispatch and return request.is_disconnected(). / 该代码块实现 `_run_with_receive`，承担模块行为中的一个聚焦逻辑片段。
- `TestPureASGIDispatchReceivePassthrough.test_is_disconnected_on_client_disconnect`: receive() -> http.disconnect: is_disconnected() must return True. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_is_disconnected_on_client_disconnect`。
- `TestPureASGIDispatchReceivePassthrough.test_not_disconnected_when_connected`: receive() -> http.request: is_disconnected() must return False. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_not_disconnected_when_connected`。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `asyncio`, `unittest`
- **Third-party modules / 第三方模块**: `starlette.requests`
- **Internal modules / 内部模块**: `sglang.srt.utils.http_middleware_patch`, `sglang.test.ci.ci_register`, `sglang.test.test_utils`

- **Total lines / 总行数**: 78
