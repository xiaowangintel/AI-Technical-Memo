# test_network_address.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/utils/test_network_address.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates network address behavior in SGLang's utils area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 工具 领域中与 network address 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-7: module imports and dependencies / 模块导入与依赖
```python
import socket
import unittest
from unittest.mock import patch

from sglang.srt.server_args import PortArgs, ServerArgs
from sglang.srt.utils.network import NetworkAddress
from sglang.test.ci.ci_register import register_cpu_ci
```
**EN:** This block imports the modules needed by the rest of the file, including `socket`, `unittest`, `unittest.mock`, `sglang.srt.server_args`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `socket`, `unittest`, `unittest.mock`, `sglang.srt.server_args`。

### Lines 9-13: CI registration and metadata / CI 注册与元数据
```python
register_cpu_ci(est_time=7, suite="base-a-test-cpu")

# Mock get_device() so ServerArgs tests run on CPU-only CI runners
_mock_device = patch("sglang.srt.server_args.get_device", return_value="cuda")
_mock_device.start()
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through register_cpu_ci, patch, start.
**CN:** 该代码块通过 register_cpu_ci, patch, start 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 16-16: class TestNetworkAddressIPv4 declaration / 类 TestNetworkAddressIPv4 声明
```python
class TestNetworkAddressIPv4(unittest.TestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `unittest.TestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `unittest.TestCase`。

### Lines 17-22: test case basic properties / 测试用例 basic properties
```python
    def test_basic_properties(self):
        na = NetworkAddress("127.0.0.1", 30000)
        self.assertEqual(na.host, "127.0.0.1")
        self.assertEqual(na.port, 30000)
        self.assertFalse(na.is_ipv6)
        self.assertEqual(na.family, socket.AF_INET)
```
**EN:** This test exercises `test_basic_properties` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_basic_properties`。

### Lines 24-27: test case to url / 测试用例 to url
```python
    def test_to_url(self):
        na = NetworkAddress("10.0.0.1", 8080)
        self.assertEqual(na.to_url(), "http://10.0.0.1:8080")
        self.assertEqual(na.to_url("https"), "https://10.0.0.1:8080")
```
**EN:** This test exercises `test_to_url` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_to_url`。

### Lines 29-32: test case to tcp / 测试用例 to tcp
```python
    def test_to_tcp(self):
        self.assertEqual(
            NetworkAddress("10.0.0.1", 25000).to_tcp(), "tcp://10.0.0.1:25000"
        )
```
**EN:** This test exercises `test_to_tcp` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_to_tcp`。

### Lines 34-37: test case to host port str / 测试用例 to host port str
```python
    def test_to_host_port_str(self):
        self.assertEqual(
            NetworkAddress("192.168.1.1", 443).to_host_port_str(), "192.168.1.1:443"
        )
```
**EN:** This test exercises `test_to_host_port_str` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_to_host_port_str`。

### Lines 39-42: test case to bind tuple / 测试用例 to bind tuple
```python
    def test_to_bind_tuple(self):
        self.assertEqual(
            NetworkAddress("0.0.0.0", 30000).to_bind_tuple(), ("0.0.0.0", 30000)
        )
```
**EN:** This test exercises `test_to_bind_tuple` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_to_bind_tuple`。

### Lines 44-45: test case str / 测试用例 str
```python
    def test_str(self):
        self.assertEqual(str(NetworkAddress("127.0.0.1", 30000)), "127.0.0.1:30000")
```
**EN:** This test exercises `test_str` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_str`。

### Lines 48-48: class TestNetworkAddressIPv6 declaration / 类 TestNetworkAddressIPv6 声明
```python
class TestNetworkAddressIPv6(unittest.TestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `unittest.TestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `unittest.TestCase`。

### Lines 49-54: test case basic properties / 测试用例 basic properties
```python
    def test_basic_properties(self):
        na = NetworkAddress("::1", 30000)
        self.assertEqual(na.host, "::1")
        self.assertEqual(na.port, 30000)
        self.assertTrue(na.is_ipv6)
        self.assertEqual(na.family, socket.AF_INET6)
```
**EN:** This test exercises `test_basic_properties` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_basic_properties`。

### Lines 56-57: test case to url / 测试用例 to url
```python
    def test_to_url(self):
        self.assertEqual(NetworkAddress("::1", 8080).to_url(), "http://[::1]:8080")
```
**EN:** This test exercises `test_to_url` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_to_url`。

### Lines 59-62: test case to url custom scheme / 测试用例 to url custom scheme
```python
    def test_to_url_custom_scheme(self):
        na = NetworkAddress("2001:db8::1", 443)
        self.assertEqual(na.to_url("https"), "https://[2001:db8::1]:443")
        self.assertEqual(na.to_url("instance"), "instance://[2001:db8::1]:443")
```
**EN:** This test exercises `test_to_url_custom_scheme` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_to_url_custom_scheme`。

### Lines 64-65: test case to tcp / 测试用例 to tcp
```python
    def test_to_tcp(self):
        self.assertEqual(NetworkAddress("::1", 25000).to_tcp(), "tcp://[::1]:25000")
```
**EN:** This test exercises `test_to_tcp` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_to_tcp`。

### Lines 67-68: test case to host port str / 测试用例 to host port str
```python
    def test_to_host_port_str(self):
        self.assertEqual(NetworkAddress("::1", 443).to_host_port_str(), "[::1]:443")
```
**EN:** This test exercises `test_to_host_port_str` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_to_host_port_str`。

### Lines 70-71: test case to bind tuple raw / 测试用例 to bind tuple raw
```python
    def test_to_bind_tuple_raw(self):
        self.assertEqual(NetworkAddress("::1", 30000).to_bind_tuple(), ("::1", 30000))
```
**EN:** This test exercises `test_to_bind_tuple_raw` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_to_bind_tuple_raw`。

### Lines 73-78: test case full ipv6 address / 测试用例 full ipv6 address
```python
    def test_full_ipv6_address(self):
        na = NetworkAddress("2001:0db8:85a3:0000:0000:8a2e:0370:7334", 80)
        self.assertTrue(na.is_ipv6)
        self.assertEqual(
            na.to_url(), "http://[2001:0db8:85a3:0000:0000:8a2e:0370:7334]:80"
        )
```
**EN:** This test exercises `test_full_ipv6_address` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_full_ipv6_address`。

### Lines 80-81: test case str / 测试用例 str
```python
    def test_str(self):
        self.assertEqual(str(NetworkAddress("::1", 30000)), "[::1]:30000")
```
**EN:** This test exercises `test_str` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_str`。

### Lines 84-84: class TestNetworkAddressHostname declaration / 类 TestNetworkAddressHostname 声明
```python
class TestNetworkAddressHostname(unittest.TestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `unittest.TestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `unittest.TestCase`。

### Lines 85-90: test case hostname / 测试用例 hostname
```python
    def test_hostname(self):
        na = NetworkAddress("my-server", 8080)
        self.assertFalse(na.is_ipv6)
        self.assertEqual(na.family, socket.AF_INET)
        self.assertEqual(na.to_url(), "http://my-server:8080")
        self.assertEqual(na.to_tcp(), "tcp://my-server:8080")
```
**EN:** This test exercises `test_hostname` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_hostname`。

### Lines 92-95: test case localhost / 测试用例 localhost
```python
    def test_localhost(self):
        na = NetworkAddress("localhost", 30000)
        self.assertFalse(na.is_ipv6)
        self.assertEqual(na.to_url(), "http://localhost:30000")
```
**EN:** This test exercises `test_localhost` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_localhost`。

### Lines 98-98: class TestNetworkAddressParse declaration / 类 TestNetworkAddressParse 声明
```python
class TestNetworkAddressParse(unittest.TestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `unittest.TestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `unittest.TestCase`。

### Lines 99-101: test case parse ipv4 / 测试用例 parse ipv4
```python
    def test_parse_ipv4(self):
        na = NetworkAddress.parse("127.0.0.1:8000")
        self.assertEqual(na, NetworkAddress("127.0.0.1", 8000))
```
**EN:** This test exercises `test_parse_ipv4` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_parse_ipv4`。

### Lines 103-104: test case parse ipv4 high port / 测试用例 parse ipv4 high port
```python
    def test_parse_ipv4_high_port(self):
        self.assertEqual(NetworkAddress.parse("10.0.0.1:65535").port, 65535)
```
**EN:** This test exercises `test_parse_ipv4_high_port` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_parse_ipv4_high_port`。

### Lines 106-109: test case parse ipv6 loopback / 测试用例 parse ipv6 loopback
```python
    def test_parse_ipv6_loopback(self):
        na = NetworkAddress.parse("[::1]:8000")
        self.assertEqual(na, NetworkAddress("::1", 8000))
        self.assertTrue(na.is_ipv6)
```
**EN:** This test exercises `test_parse_ipv6_loopback` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_parse_ipv6_loopback`。

### Lines 111-113: test case parse ipv6 full / 测试用例 parse ipv6 full
```python
    def test_parse_ipv6_full(self):
        na = NetworkAddress.parse("[2001:db8::1]:30000")
        self.assertEqual(na, NetworkAddress("2001:db8::1", 30000))
```
**EN:** This test exercises `test_parse_ipv6_full` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_parse_ipv6_full`。

### Lines 115-117: test case parse ipv6 all interfaces / 测试用例 parse ipv6 all interfaces
```python
    def test_parse_ipv6_all_interfaces(self):
        na = NetworkAddress.parse("[::]:8080")
        self.assertEqual(na, NetworkAddress("::", 8080))
```
**EN:** This test exercises `test_parse_ipv6_all_interfaces` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_parse_ipv6_all_interfaces`。

### Lines 119-121: test case parse hostname / 测试用例 parse hostname
```python
    def test_parse_hostname(self):
        na = NetworkAddress.parse("my-server:9000")
        self.assertEqual(na, NetworkAddress("my-server", 9000))
```
**EN:** This test exercises `test_parse_hostname` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_parse_hostname`。

### Lines 123-125: test case parse fqdn / 测试用例 parse fqdn
```python
    def test_parse_fqdn(self):
        na = NetworkAddress.parse("node1.cluster.local:25000")
        self.assertEqual(na, NetworkAddress("node1.cluster.local", 25000))
```
**EN:** This test exercises `test_parse_fqdn` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_parse_fqdn`。

### Lines 127-129: test case roundtrip ipv4 / 测试用例 roundtrip ipv4
```python
    def test_roundtrip_ipv4(self):
        na = NetworkAddress("10.0.0.1", 8080)
        self.assertEqual(NetworkAddress.parse(na.to_host_port_str()), na)
```
**EN:** This test exercises `test_roundtrip_ipv4` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_roundtrip_ipv4`。

### Lines 131-133: test case roundtrip ipv6 / 测试用例 roundtrip ipv6
```python
    def test_roundtrip_ipv6(self):
        na = NetworkAddress("::1", 30000)
        self.assertEqual(NetworkAddress.parse(na.to_host_port_str()), na)
```
**EN:** This test exercises `test_roundtrip_ipv6` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_roundtrip_ipv6`。

### Lines 135-137: test case roundtrip hostname / 测试用例 roundtrip hostname
```python
    def test_roundtrip_hostname(self):
        na = NetworkAddress("my-host", 443)
        self.assertEqual(NetworkAddress.parse(na.to_host_port_str()), na)
```
**EN:** This test exercises `test_roundtrip_hostname` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_roundtrip_hostname`。

### Lines 140-140: class TestNetworkAddressParseErrors declaration / 类 TestNetworkAddressParseErrors 声明
```python
class TestNetworkAddressParseErrors(unittest.TestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `unittest.TestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `unittest.TestCase`。

### Lines 141-143: test case empty / 测试用例 empty
```python
    def test_empty(self):
        with self.assertRaises(ValueError):
            NetworkAddress.parse("")
```
**EN:** This test exercises `test_empty` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_empty`。

### Lines 145-147: test case no port / 测试用例 no port
```python
    def test_no_port(self):
        with self.assertRaises(ValueError):
            NetworkAddress.parse("127.0.0.1")
```
**EN:** This test exercises `test_no_port` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_no_port`。

### Lines 149-151: test case bare ipv6 ambiguous / 测试用例 bare ipv6 ambiguous
```python
    def test_bare_ipv6_ambiguous(self):
        with self.assertRaises(ValueError):
            NetworkAddress.parse("::1:8000")
```
**EN:** This test exercises `test_bare_ipv6_ambiguous` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_bare_ipv6_ambiguous`。

### Lines 153-155: test case missing closing bracket / 测试用例 missing closing bracket
```python
    def test_missing_closing_bracket(self):
        with self.assertRaises(ValueError):
            NetworkAddress.parse("[::1:8000")
```
**EN:** This test exercises `test_missing_closing_bracket` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_missing_closing_bracket`。

### Lines 157-159: test case invalid ipv6 in brackets / 测试用例 invalid ipv6 in brackets
```python
    def test_invalid_ipv6_in_brackets(self):
        with self.assertRaises(ValueError):
            NetworkAddress.parse("[not-ipv6]:8000")
```
**EN:** This test exercises `test_invalid_ipv6_in_brackets` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_invalid_ipv6_in_brackets`。

### Lines 161-163: test case bracket no port / 测试用例 bracket no port
```python
    def test_bracket_no_port(self):
        with self.assertRaises(ValueError):
            NetworkAddress.parse("[::1]")
```
**EN:** This test exercises `test_bracket_no_port` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_bracket_no_port`。

### Lines 165-167: test case invalid port string / 测试用例 invalid port string
```python
    def test_invalid_port_string(self):
        with self.assertRaises(ValueError):
            NetworkAddress.parse("127.0.0.1:abc")
```
**EN:** This test exercises `test_invalid_port_string` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_invalid_port_string`。

### Lines 169-171: test case port out of range / 测试用例 port out of range
```python
    def test_port_out_of_range(self):
        with self.assertRaises(ValueError):
            NetworkAddress.parse("127.0.0.1:70000")
```
**EN:** This test exercises `test_port_out_of_range` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_port_out_of_range`。

### Lines 173-175: test case negative port / 测试用例 negative port
```python
    def test_negative_port(self):
        with self.assertRaises(ValueError):
            NetworkAddress.parse("127.0.0.1:-1")
```
**EN:** This test exercises `test_negative_port` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_negative_port`。

### Lines 177-179: test case empty host / 测试用例 empty host
```python
    def test_empty_host(self):
        with self.assertRaises(ValueError):
            NetworkAddress.parse(":8000")
```
**EN:** This test exercises `test_empty_host` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_empty_host`。

### Lines 182-182: class TestNetworkAddressBracketStripping declaration / 类 TestNetworkAddressBracketStripping 声明
```python
class TestNetworkAddressBracketStripping(unittest.TestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `unittest.TestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `unittest.TestCase`。

### Lines 183-186: test case strip brackets / 测试用例 strip brackets
```python
    def test_strip_brackets(self):
        na = NetworkAddress("[::1]", 8000)
        self.assertEqual(na.host, "::1")
        self.assertTrue(na.is_ipv6)
```
**EN:** This test exercises `test_strip_brackets` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_strip_brackets`。

### Lines 188-190: test case no brackets / 测试用例 no brackets
```python
    def test_no_brackets(self):
        na = NetworkAddress("::1", 8000)
        self.assertEqual(na.host, "::1")
```
**EN:** This test exercises `test_no_brackets` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_no_brackets`。

### Lines 192-195: test case ipv4 passthrough / 测试用例 ipv4 passthrough
```python
    def test_ipv4_passthrough(self):
        na = NetworkAddress("127.0.0.1", 30000)
        self.assertEqual(na.host, "127.0.0.1")
        self.assertFalse(na.is_ipv6)
```
**EN:** This test exercises `test_ipv4_passthrough` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_ipv4_passthrough`。

### Lines 197-199: test case hostname passthrough / 测试用例 hostname passthrough
```python
    def test_hostname_passthrough(self):
        na = NetworkAddress("myhost", 30000)
        self.assertEqual(na.host, "myhost")
```
**EN:** This test exercises `test_hostname_passthrough` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_hostname_passthrough`。

### Lines 202-202: class TestNetworkAddressImmutability declaration / 类 TestNetworkAddressImmutability 声明
```python
class TestNetworkAddressImmutability(unittest.TestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `unittest.TestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `unittest.TestCase`。

### Lines 203-208: test case frozen / 测试用例 frozen
```python
    def test_frozen(self):
        na = NetworkAddress("127.0.0.1", 30000)
        with self.assertRaises(AttributeError):
            na.host = "0.0.0.0"
        with self.assertRaises(AttributeError):
            na.port = 8080
```
**EN:** This test exercises `test_frozen` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_frozen`。

### Lines 210-215: test case hashable / 测试用例 hashable
```python
    def test_hashable(self):
        a = NetworkAddress("::1", 8000)
        b = NetworkAddress("::1", 8000)
        self.assertEqual(a, b)
        self.assertEqual(hash(a), hash(b))
        self.assertEqual(len({a, b}), 1)
```
**EN:** This test exercises `test_hashable` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_hashable`。

### Lines 217-220: test case inequality / 测试用例 inequality
```python
    def test_inequality(self):
        a = NetworkAddress("127.0.0.1", 8000)
        b = NetworkAddress("127.0.0.1", 8001)
        self.assertNotEqual(a, b)
```
**EN:** This test exercises `test_inequality` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_inequality`。

### Lines 223-223: class TestPortArgsIPv6 declaration / 类 TestPortArgsIPv6 声明
```python
class TestPortArgsIPv6(unittest.TestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `unittest.TestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `unittest.TestCase`。

### Lines 224-224: supporting statements / 辅助语句
```python
    """PortArgs.init_new() IPv6 address parsing via NetworkAddress.parse()."""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 226-243: test case init new with ipv6 address / 测试用例 init new with ipv6 address
```python
    def test_init_new_with_ipv6_address(self):
        server_args = ServerArgs(model_path="dummy")
        server_args.port = 30000
        server_args.nccl_port = None
        server_args.enable_dp_attention = True
        server_args.nnodes = 2
        server_args.dist_init_addr = "[2001:db8::1]:25000"

        port_args = PortArgs.init_new(server_args)

        self.assertTrue(port_args.tokenizer_ipc_name.startswith("tcp://[2001:db8::1]:"))
        self.assertTrue(
            port_args.scheduler_input_ipc_name.startswith("tcp://[2001:db8::1]:")
        )
        self.assertTrue(
            port_args.detokenizer_ipc_name.startswith("tcp://[2001:db8::1]:")
        )
        self.assertIsInstance(port_args.nccl_port, int)
```
**EN:** This test exercises `test_init_new_with_ipv6_address` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_init_new_with_ipv6_address`。

### Lines 245-255: test case init new with invalid ipv6 address / 测试用例 init new with invalid ipv6 address
```python
    def test_init_new_with_invalid_ipv6_address(self):
        server_args = ServerArgs(model_path="dummy")
        server_args.port = 30000
        server_args.nccl_port = None
        server_args.enable_dp_attention = True
        server_args.nnodes = 2
        server_args.dist_init_addr = "[invalid-ipv6]:25000"

        with self.assertRaises(ValueError) as context:
            PortArgs.init_new(server_args)
        self.assertIn("Invalid IPv6 address inside brackets", str(context.exception))
```
**EN:** This test exercises `test_init_new_with_invalid_ipv6_address` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_init_new_with_invalid_ipv6_address`。

### Lines 257-267: test case init new with malformed ipv6 address missing bracket / 测试用例 init new with malformed ipv6 address missing bracket
```python
    def test_init_new_with_malformed_ipv6_address_missing_bracket(self):
        server_args = ServerArgs(model_path="dummy")
        server_args.port = 30000
        server_args.nccl_port = None
        server_args.enable_dp_attention = True
        server_args.nnodes = 2
        server_args.dist_init_addr = "[2001:db8::1:25000"

        with self.assertRaises(ValueError) as context:
            PortArgs.init_new(server_args)
        self.assertIn("Missing closing bracket", str(context.exception))
```
**EN:** This test exercises `test_init_new_with_malformed_ipv6_address_missing_bracket` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_init_new_with_malformed_ipv6_address_missing_bracket`。

### Lines 269-279: test case init new with malformed ipv6 address missing port / 测试用例 init new with malformed ipv6 address missing port
```python
    def test_init_new_with_malformed_ipv6_address_missing_port(self):
        server_args = ServerArgs(model_path="dummy")
        server_args.port = 30000
        server_args.nccl_port = None
        server_args.enable_dp_attention = True
        server_args.nnodes = 2
        server_args.dist_init_addr = "[2001:db8::1]"

        with self.assertRaises(ValueError) as context:
            PortArgs.init_new(server_args)
        self.assertIn("Expected ':port' after closing bracket", str(context.exception))
```
**EN:** This test exercises `test_init_new_with_malformed_ipv6_address_missing_port` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_init_new_with_malformed_ipv6_address_missing_port`。

### Lines 281-291: test case init new with malformed ipv6 address invalid port / 测试用例 init new with malformed ipv6 address invalid port
```python
    def test_init_new_with_malformed_ipv6_address_invalid_port(self):
        server_args = ServerArgs(model_path="dummy")
        server_args.port = 30000
        server_args.nccl_port = None
        server_args.enable_dp_attention = True
        server_args.nnodes = 2
        server_args.dist_init_addr = "[2001:db8::1]:abcde"

        with self.assertRaises(ValueError) as context:
            PortArgs.init_new(server_args)
        self.assertIn("Invalid port number", str(context.exception))
```
**EN:** This test exercises `test_init_new_with_malformed_ipv6_address_invalid_port` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_init_new_with_malformed_ipv6_address_invalid_port`。

### Lines 293-303: test case init new with malformed ipv6 address wrong separator / 测试用例 init new with malformed ipv6 address wrong separator
```python
    def test_init_new_with_malformed_ipv6_address_wrong_separator(self):
        server_args = ServerArgs(model_path="dummy")
        server_args.port = 30000
        server_args.nccl_port = None
        server_args.enable_dp_attention = True
        server_args.nnodes = 2
        server_args.dist_init_addr = "[2001:db8::1]#25000"

        with self.assertRaises(ValueError) as context:
            PortArgs.init_new(server_args)
        self.assertIn("Expected ':port' after closing bracket", str(context.exception))
```
**EN:** This test exercises `test_init_new_with_malformed_ipv6_address_wrong_separator` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_init_new_with_malformed_ipv6_address_wrong_separator`。

### Lines 306-306: class TestServerArgsIPv6Url declaration / 类 TestServerArgsIPv6Url 声明
```python
class TestServerArgsIPv6Url(unittest.TestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `unittest.TestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `unittest.TestCase`。

### Lines 307-307: supporting statements / 辅助语句
```python
    """ServerArgs.url() IPv6 formatting (moved from test_server_args.py)."""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 309-311: test case url rewrites ipv6 all interfaces to loopback / 测试用例 url rewrites ipv6 all interfaces to loopback
```python
    def test_url_rewrites_ipv6_all_interfaces_to_loopback(self):
        server_args = ServerArgs(model_path="dummy", host="::")
        self.assertEqual(server_args.url(), "http://[::1]:30000")
```
**EN:** This test exercises `test_url_rewrites_ipv6_all_interfaces_to_loopback` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_url_rewrites_ipv6_all_interfaces_to_loopback`。

### Lines 313-321: test case url returns https with ssl and ipv6 / 测试用例 url returns https with ssl and ipv6
```python
    @patch("os.path.isfile", return_value=True)
    def test_url_returns_https_with_ssl_and_ipv6(self, _mock_isfile):
        server_args = ServerArgs(
            model_path="dummy",
            host="::1",
            ssl_keyfile="key.pem",
            ssl_certfile="cert.pem",
        )
        self.assertEqual(server_args.url(), "https://[::1]:30000")
```
**EN:** This test exercises `test_url_returns_https_with_ssl_and_ipv6` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_url_returns_https_with_ssl_and_ipv6`。

### Lines 324-325: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `TestNetworkAddressIPv4`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestNetworkAddressIPv6`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestNetworkAddressHostname`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestNetworkAddressParse`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestNetworkAddressParseErrors`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestNetworkAddressBracketStripping`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestNetworkAddressImmutability`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestPortArgsIPv6`: PortArgs.init_new() IPv6 address parsing via NetworkAddress.parse(). / 用于组织相关测试、夹具或辅助方法。
- `TestNetworkAddressIPv4.test_basic_properties`: This test exercises `test_basic_properties` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_basic_properties`。
- `TestNetworkAddressIPv4.test_to_url`: This test exercises `test_to_url` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_to_url`。
- `TestNetworkAddressIPv4.test_to_tcp`: This test exercises `test_to_tcp` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_to_tcp`。
- `TestNetworkAddressIPv4.test_to_host_port_str`: This test exercises `test_to_host_port_str` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_to_host_port_str`。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `socket`, `unittest`, `unittest.mock`
- **Internal modules / 内部模块**: `sglang.srt.server_args`, `sglang.srt.utils.network`, `sglang.test.ci.ci_register`

- **Total lines / 总行数**: 325
