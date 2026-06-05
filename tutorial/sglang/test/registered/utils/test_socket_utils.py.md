# test_socket_utils.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/utils/test_socket_utils.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates socket utils behavior in SGLang's utils area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 工具 领域中与 socket utils 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-16: module imports and dependencies / 模块导入与依赖
```python
import os
import socket
import unittest
from unittest.mock import patch

from sglang.srt.utils.network import (
    _get_addrinfos_for_bind,
    bind_port,
    get_free_port,
    get_open_port,
    is_port_available,
    try_bind_socket,
)
from sglang.test.ci.ci_register import register_cpu_ci
from sglang.test.test_utils import CustomTestCase
from sglang.utils import normalize_base_url, release_port, reserve_port
```
**EN:** This block imports the modules needed by the rest of the file, including `os`, `socket`, `unittest`, `unittest.mock`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `os`, `socket`, `unittest`, `unittest.mock`。

### Lines 18-18: CI registration and metadata / CI 注册与元数据
```python
register_cpu_ci(est_time=7, suite="base-a-test-cpu")
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through register_cpu_ci.
**CN:** 该代码块通过 register_cpu_ci 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 21-21: class TestTryBindSocket declaration / 类 TestTryBindSocket 声明
```python
class TestTryBindSocket(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 22-30: test case bind ephemeral port / 测试用例 bind ephemeral port
```python
    def test_bind_ephemeral_port(self):
        """try_bind_socket() with port=0 should bind to an OS-assigned port."""
        sock = try_bind_socket()
        try:
            port = sock.getsockname()[1]
            self.assertGreater(port, 0)
            self.assertLessEqual(port, 65535)
        finally:
            sock.close()
```
**EN:** try_bind_socket() with port=0 should bind to an OS-assigned port. This test exercises `test_bind_ephemeral_port` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** try_bind_socket() with port=0 should bind to an OS-assigned port. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_bind_ephemeral_port`。

### Lines 32-39: test case bind specific port / 测试用例 bind specific port
```python
    def test_bind_specific_port(self):
        """try_bind_socket(port=N) should bind to that exact port."""
        port = get_free_port()
        sock = try_bind_socket(port=port)
        try:
            self.assertEqual(sock.getsockname()[1], port)
        finally:
            sock.close()
```
**EN:** try_bind_socket(port=N) should bind to that exact port. This test exercises `test_bind_specific_port` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** try_bind_socket(port=N) should bind to that exact port. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_bind_specific_port`。

### Lines 41-49: test case bind with listen / 测试用例 bind with listen
```python
    def test_bind_with_listen(self):
        """try_bind_socket(listen=True) should return a listening socket."""
        sock = try_bind_socket(listen=True)
        try:
            # A listening socket has a valid bound address
            port = sock.getsockname()[1]
            self.assertGreater(port, 0)
        finally:
            sock.close()
```
**EN:** try_bind_socket(listen=True) should return a listening socket. This test exercises `test_bind_with_listen` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** try_bind_socket(listen=True) should return a listening socket. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_bind_with_listen`。

### Lines 51-58: test case bind with host / 测试用例 bind with host
```python
    def test_bind_with_host(self):
        """try_bind_socket(host='127.0.0.1') should bind to localhost."""
        sock = try_bind_socket(host="127.0.0.1")
        try:
            addr = sock.getsockname()
            self.assertEqual(addr[0], "127.0.0.1")
        finally:
            sock.close()
```
**EN:** try_bind_socket(host='127.0.0.1') should bind to localhost. This test exercises `test_bind_with_host` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** try_bind_socket(host='127.0.0.1') should bind to localhost. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_bind_with_host`。

### Lines 60-68: test case bind occupied port raises / 测试用例 bind occupied port raises
```python
    def test_bind_occupied_port_raises(self):
        """try_bind_socket should raise OSError if port is occupied."""
        sock1 = try_bind_socket(host="127.0.0.1", reuse_addr=False)
        try:
            port = sock1.getsockname()[1]
            with self.assertRaises(OSError):
                try_bind_socket(host="127.0.0.1", port=port, reuse_addr=False)
        finally:
            sock1.close()
```
**EN:** try_bind_socket should raise OSError if port is occupied. This test exercises `test_bind_occupied_port_raises` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** try_bind_socket should raise OSError if port is occupied. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_bind_occupied_port_raises`。

### Lines 70-76: test case returns correct family / 测试用例 returns correct family
```python
    def test_returns_correct_family(self):
        """Returned socket should be AF_INET or AF_INET6."""
        sock = try_bind_socket()
        try:
            self.assertIn(sock.family, (socket.AF_INET, socket.AF_INET6))
        finally:
            sock.close()
```
**EN:** Returned socket should be AF_INET or AF_INET6. This test exercises `test_returns_correct_family` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Returned socket should be AF_INET or AF_INET6. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_returns_correct_family`。

### Lines 78-88: test case gaierror fallback / 测试用例 gaierror fallback
```python
    def test_gaierror_fallback(self):
        """_get_addrinfos_for_bind should fall back to AF_INET on gaierror."""
        with patch(
            "sglang.srt.utils.network.socket.getaddrinfo",
            side_effect=socket.gaierror("mocked"),
        ):
            infos = _get_addrinfos_for_bind()
            self.assertEqual(len(infos), 1)
            family, socktype, _, _, sockaddr = infos[0]
            self.assertEqual(family, socket.AF_INET)
            self.assertEqual(sockaddr[0], "0.0.0.0")
```
**EN:** _get_addrinfos_for_bind should fall back to AF_INET on gaierror. This test exercises `test_gaierror_fallback` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** _get_addrinfos_for_bind should fall back to AF_INET on gaierror. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_gaierror_fallback`。

### Lines 90-97: test case gaierror fallback preserves host / 测试用例 gaierror fallback preserves host
```python
    def test_gaierror_fallback_preserves_host(self):
        """Fallback should use the provided host, not default to 0.0.0.0."""
        with patch(
            "sglang.srt.utils.network.socket.getaddrinfo",
            side_effect=socket.gaierror("mocked"),
        ):
            infos = _get_addrinfos_for_bind(host="10.0.0.1", port=8080)
            self.assertEqual(infos[0][4], ("10.0.0.1", 8080))
```
**EN:** Fallback should use the provided host, not default to 0.0.0.0. This test exercises `test_gaierror_fallback_preserves_host` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Fallback should use the provided host, not default to 0.0.0.0. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_gaierror_fallback_preserves_host`。

### Lines 100-100: class TestSocketUtilities declaration / 类 TestSocketUtilities 声明
```python
class TestSocketUtilities(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 101-104: test case is port available / 测试用例 is port available
```python
    def test_is_port_available(self):
        """is_port_available should return True for a free port."""
        port = get_free_port()
        self.assertTrue(is_port_available(port))
```
**EN:** is_port_available should return True for a free port. This test exercises `test_is_port_available` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** is_port_available should return True for a free port. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_is_port_available`。

### Lines 106-113: test case is port available occupied / 测试用例 is port available occupied
```python
    def test_is_port_available_occupied(self):
        """is_port_available should return False for an occupied port."""
        sock = try_bind_socket(port=0, reuse_addr=False, listen=True)
        try:
            port = sock.getsockname()[1]
            self.assertFalse(is_port_available(port))
        finally:
            sock.close()
```
**EN:** is_port_available should return False for an occupied port. This test exercises `test_is_port_available_occupied` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** is_port_available should return False for an occupied port. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_is_port_available_occupied`。

### Lines 115-119: test case get free port / 测试用例 get free port
```python
    def test_get_free_port(self):
        """get_free_port should return a valid port number."""
        port = get_free_port()
        self.assertGreater(port, 0)
        self.assertLessEqual(port, 65535)
```
**EN:** get_free_port should return a valid port number. This test exercises `test_get_free_port` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** get_free_port should return a valid port number. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_get_free_port`。

### Lines 121-128: test case bind port / 测试用例 bind port
```python
    def test_bind_port(self):
        """bind_port should return a listening socket."""
        port = get_free_port()
        sock = bind_port(port)
        try:
            self.assertEqual(sock.getsockname()[1], port)
        finally:
            sock.close()
```
**EN:** bind_port should return a listening socket. This test exercises `test_bind_port` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** bind_port should return a listening socket. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_bind_port`。

### Lines 130-134: test case get open port / 测试用例 get open port
```python
    def test_get_open_port(self):
        """get_open_port should return a valid port number."""
        port = get_open_port()
        self.assertGreater(port, 0)
        self.assertLessEqual(port, 65535)
```
**EN:** get_open_port should return a valid port number. This test exercises `test_get_open_port` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** get_open_port should return a valid port number. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_get_open_port`。

### Lines 136-141: test case get open port with env var / 测试用例 get open port with env var
```python
    def test_get_open_port_with_env_var(self):
        """get_open_port should respect SGLANG_PORT env var."""
        free_port = get_free_port()
        with patch.dict(os.environ, {"SGLANG_PORT": str(free_port)}):
            port = get_open_port()
            self.assertEqual(port, free_port)
```
**EN:** get_open_port should respect SGLANG_PORT env var. This test exercises `test_get_open_port_with_env_var` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** get_open_port should respect SGLANG_PORT env var. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_get_open_port_with_env_var`。

### Lines 143-153: test case get open port env var occupied increments / 测试用例 get open port env var occupied increments
```python
    def test_get_open_port_env_var_occupied_increments(self):
        """get_open_port should increment if SGLANG_PORT is occupied."""
        sock = try_bind_socket(port=0, reuse_addr=False, listen=True)
        try:
            occupied_port = sock.getsockname()[1]
            with patch.dict(os.environ, {"SGLANG_PORT": str(occupied_port)}):
                port = get_open_port()
                # Should skip the occupied port and return a higher one
                self.assertGreater(port, occupied_port)
        finally:
            sock.close()
```
**EN:** get_open_port should increment if SGLANG_PORT is occupied. This test exercises `test_get_open_port_env_var_occupied_increments` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** get_open_port should increment if SGLANG_PORT is occupied. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_get_open_port_env_var_occupied_increments`。

### Lines 156-156: class TestReservePort declaration / 类 TestReservePort 声明
```python
class TestReservePort(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 157-165: test case reserve port returns port and socket / 测试用例 reserve port returns port and socket
```python
    def test_reserve_port_returns_port_and_socket(self):
        """reserve_port should return a (port, socket) tuple."""
        port, sock = reserve_port("127.0.0.1")
        try:
            self.assertGreaterEqual(port, 30000)
            self.assertLess(port, 40000)
            self.assertEqual(sock.getsockname()[1], port)
        finally:
            release_port(sock)
```
**EN:** reserve_port should return a (port, socket) tuple. This test exercises `test_reserve_port_returns_port_and_socket` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** reserve_port should return a (port, socket) tuple. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_reserve_port_returns_port_and_socket`。

### Lines 167-174: test case reserve port custom range / 测试用例 reserve port custom range
```python
    def test_reserve_port_custom_range(self):
        """reserve_port should respect custom start/end range."""
        port, sock = reserve_port("127.0.0.1", start=40000, end=41000)
        try:
            self.assertGreaterEqual(port, 40000)
            self.assertLess(port, 41000)
        finally:
            release_port(sock)
```
**EN:** reserve_port should respect custom start/end range. This test exercises `test_reserve_port_custom_range` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** reserve_port should respect custom start/end range. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_reserve_port_custom_range`。

### Lines 176-185: test case reserve port holds port / 测试用例 reserve port holds port
```python
    def test_reserve_port_holds_port(self):
        """The reserved port should not be available until released."""
        port, sock = reserve_port("127.0.0.1")
        try:
            # Verify port is held by trying to bind the same family explicitly
            with self.assertRaises(OSError):
                s = try_bind_socket(host="127.0.0.1", port=port, reuse_addr=False)
                s.close()
        finally:
            release_port(sock)
```
**EN:** The reserved port should not be available until released. This test exercises `test_reserve_port_holds_port` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** The reserved port should not be available until released. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_reserve_port_holds_port`。

### Lines 187-194: test case reserve port no free port raises / 测试用例 reserve port no free port raises
```python
    def test_reserve_port_no_free_port_raises(self):
        """reserve_port should raise RuntimeError if no port is available."""
        with patch(
            "sglang.srt.utils.network.try_bind_socket",
            side_effect=OSError("mocked"),
        ):
            with self.assertRaises(RuntimeError):
                reserve_port("127.0.0.1", start=50000, end=50002)
```
**EN:** reserve_port should raise RuntimeError if no port is available. This test exercises `test_reserve_port_no_free_port_raises` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** reserve_port should raise RuntimeError if no port is available. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_reserve_port_no_free_port_raises`。

### Lines 197-197: class TestNormalizeBaseUrl declaration / 类 TestNormalizeBaseUrl 声明
```python
class TestNormalizeBaseUrl(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 198-201: test case ipv4 host / 测试用例 ipv4 host
```python
    def test_ipv4_host(self):
        """normalize_base_url should produce http://host:port for IPv4."""
        url = normalize_base_url("127.0.0.1", 8080)
        self.assertEqual(url, "http://127.0.0.1:8080")
```
**EN:** normalize_base_url should produce http://host:port for IPv4. This test exercises `test_ipv4_host` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** normalize_base_url should produce http://host:port for IPv4. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_ipv4_host`。

### Lines 203-206: test case ipv6 host / 测试用例 ipv6 host
```python
    def test_ipv6_host(self):
        """normalize_base_url should bracket IPv6 addresses."""
        url = normalize_base_url("::1", 8080)
        self.assertEqual(url, "http://[::1]:8080")
```
**EN:** normalize_base_url should bracket IPv6 addresses. This test exercises `test_ipv6_host` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** normalize_base_url should bracket IPv6 addresses. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_ipv6_host`。

### Lines 208-211: test case hostname / 测试用例 hostname
```python
    def test_hostname(self):
        """normalize_base_url should work with hostnames."""
        url = normalize_base_url("localhost", 3000)
        self.assertEqual(url, "http://localhost:3000")
```
**EN:** normalize_base_url should work with hostnames. This test exercises `test_hostname` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** normalize_base_url should work with hostnames. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_hostname`。

### Lines 213-220: test case deprecated scheme passthrough / 测试用例 deprecated scheme passthrough
```python
    def test_deprecated_scheme_passthrough(self):
        """normalize_base_url should pass through host with scheme (deprecated)."""
        import warnings

        with warnings.catch_warnings():
            warnings.simplefilter("ignore", DeprecationWarning)
            url = normalize_base_url("http://myhost", 9000)
        self.assertEqual(url, "http://myhost:9000")
```
**EN:** normalize_base_url should pass through host with scheme (deprecated). This test exercises `test_deprecated_scheme_passthrough` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** normalize_base_url should pass through host with scheme (deprecated). 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_deprecated_scheme_passthrough`。

### Lines 223-224: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `TestTryBindSocket`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestSocketUtilities`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestReservePort`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestNormalizeBaseUrl`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestTryBindSocket.test_bind_ephemeral_port`: try_bind_socket() with port=0 should bind to an OS-assigned port. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_bind_ephemeral_port`。
- `TestTryBindSocket.test_bind_specific_port`: try_bind_socket(port=N) should bind to that exact port. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_bind_specific_port`。
- `TestTryBindSocket.test_bind_with_listen`: try_bind_socket(listen=True) should return a listening socket. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_bind_with_listen`。
- `TestTryBindSocket.test_bind_with_host`: try_bind_socket(host='127.0.0.1') should bind to localhost. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_bind_with_host`。
- `TestTryBindSocket.test_bind_occupied_port_raises`: try_bind_socket should raise OSError if port is occupied. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_bind_occupied_port_raises`。
- `TestTryBindSocket.test_returns_correct_family`: Returned socket should be AF_INET or AF_INET6. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_returns_correct_family`。
- `TestTryBindSocket.test_gaierror_fallback`: _get_addrinfos_for_bind should fall back to AF_INET on gaierror. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_gaierror_fallback`。
- `TestTryBindSocket.test_gaierror_fallback_preserves_host`: Fallback should use the provided host, not default to 0.0.0.0. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_gaierror_fallback_preserves_host`。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `os`, `socket`, `unittest`, `unittest.mock`
- **Internal modules / 内部模块**: `sglang.srt.utils.network`, `sglang.test.ci.ci_register`, `sglang.test.test_utils`, `sglang.utils`

- **Total lines / 总行数**: 224
