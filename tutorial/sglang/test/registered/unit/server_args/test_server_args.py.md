# test_server_args.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/unit/server_args/test_server_args.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates server args behavior in SGLang's unit / server args area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 单元 / server args 领域中与 server args 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-12: module imports and dependencies / 模块导入与依赖
```python
import json
import tempfile
import unittest
from unittest.mock import MagicMock, patch

from sglang.srt.arg_groups.speculative_hook import handle_speculative_decoding
from sglang.srt.server_args import PortArgs, ServerArgs, prepare_server_args
from sglang.test.ci.ci_register import register_cpu_ci
from sglang.test.test_utils import (
    DEFAULT_SMALL_MODEL_NAME_FOR_TEST_QWEN,
    CustomTestCase,
)
```
**EN:** This block imports the modules needed by the rest of the file, including `json`, `tempfile`, `unittest`, `unittest.mock`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `json`, `tempfile`, `unittest`, `unittest.mock`。

### Lines 14-18: CI registration and metadata / CI 注册与元数据
```python
register_cpu_ci(est_time=10, suite="base-a-test-cpu")

# Mock get_device() so all tests run on CPU-only CI runners
_mock_device = patch("sglang.srt.server_args.get_device", return_value="cuda")
_mock_device.start()
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through register_cpu_ci, patch, start.
**CN:** 该代码块通过 register_cpu_ci, patch, start 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 21-21: class TestPrepareServerArgs declaration / 类 TestPrepareServerArgs 声明
```python
class TestPrepareServerArgs(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 22-35: test case prepare server args / 测试用例 prepare server args
```python
    def test_prepare_server_args(self):
        server_args = prepare_server_args(
            [
                "--model-path",
                DEFAULT_SMALL_MODEL_NAME_FOR_TEST_QWEN,
                "--json-model-override-args",
                '{"rope_scaling": {"factor": 2.0, "rope_type": "linear"}}',
            ]
        )
        self.assertEqual(server_args.model_path, DEFAULT_SMALL_MODEL_NAME_FOR_TEST_QWEN)
        self.assertEqual(
            json.loads(server_args.json_model_override_args),
            {"rope_scaling": {"factor": 2.0, "rope_type": "linear"}},
        )
```
**EN:** This test exercises `test_prepare_server_args` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_prepare_server_args`。

### Lines 38-38: class TestLoadBalanceMethod declaration / 类 TestLoadBalanceMethod 声明
```python
class TestLoadBalanceMethod(unittest.TestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `unittest.TestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `unittest.TestCase`。

### Lines 39-41: test case non pd defaults to round robin / 测试用例 non pd defaults to round robin
```python
    def test_non_pd_defaults_to_round_robin(self):
        server_args = ServerArgs(model_path="dummy", disaggregation_mode="null")
        self.assertEqual(server_args.load_balance_method, "round_robin")
```
**EN:** This test exercises `test_non_pd_defaults_to_round_robin` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_non_pd_defaults_to_round_robin`。

### Lines 43-45: test case pd prefill defaults to follow bootstrap room / 测试用例 pd prefill defaults to follow bootstrap room
```python
    def test_pd_prefill_defaults_to_follow_bootstrap_room(self):
        server_args = ServerArgs(model_path="dummy", disaggregation_mode="prefill")
        self.assertEqual(server_args.load_balance_method, "follow_bootstrap_room")
```
**EN:** This test exercises `test_pd_prefill_defaults_to_follow_bootstrap_room` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_pd_prefill_defaults_to_follow_bootstrap_room`。

### Lines 47-49: test case pd decode defaults to round robin / 测试用例 pd decode defaults to round robin
```python
    def test_pd_decode_defaults_to_round_robin(self):
        server_args = ServerArgs(model_path="dummy", disaggregation_mode="decode")
        self.assertEqual(server_args.load_balance_method, "round_robin")
```
**EN:** This test exercises `test_pd_decode_defaults_to_round_robin` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_pd_decode_defaults_to_round_robin`。

### Lines 51-65: test case pd decode radix cache rejects hisparse / 测试用例 pd decode radix cache rejects hisparse
```python
    def test_pd_decode_radix_cache_rejects_hisparse(self):
        with self.assertRaises(ValueError) as context:
            ServerArgs(
                model_path="dummy",
                disaggregation_mode="decode",
                disaggregation_decode_enable_radix_cache=True,
                disaggregation_transfer_backend="nixl",
                enable_hisparse=True,
            )

        self.assertIn(
            "--disaggregation-decode-enable-radix-cache is incompatible with "
            "--enable-hisparse",
            str(context.exception),
        )
```
**EN:** This test exercises `test_pd_decode_radix_cache_rejects_hisparse` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_pd_decode_radix_cache_rejects_hisparse`。

### Lines 67-75: test case pd decode radix cache allows mooncake / 测试用例 pd decode radix cache allows mooncake
```python
    def test_pd_decode_radix_cache_allows_mooncake(self):
        server_args = ServerArgs(
            model_path="dummy",
            disaggregation_mode="decode",
            disaggregation_decode_enable_radix_cache=True,
            disaggregation_transfer_backend="mooncake",
        )

        self.assertFalse(server_args.disable_radix_cache)
```
**EN:** This test exercises `test_pd_decode_radix_cache_allows_mooncake` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_pd_decode_radix_cache_allows_mooncake`。

### Lines 77-87: test case pd decode radix cache rejects unknown backend / 测试用例 pd decode radix cache rejects unknown backend
```python
    def test_pd_decode_radix_cache_rejects_unknown_backend(self):
        with self.assertRaises(ValueError) as context:
            ServerArgs(
                model_path="dummy",
                disaggregation_mode="decode",
                disaggregation_decode_enable_radix_cache=True,
                disaggregation_transfer_backend="fake",
            )

        self.assertIn("('nixl', 'mooncake')", str(context.exception))
        self.assertIn("'fake'", str(context.exception))
```
**EN:** This test exercises `test_pd_decode_radix_cache_rejects_unknown_backend` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_pd_decode_radix_cache_rejects_unknown_backend`。

### Lines 90-90: class TestPortArgs declaration / 类 TestPortArgs 声明
```python
class TestPortArgs(unittest.TestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `unittest.TestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `unittest.TestCase`。

### Lines 91-110: test case init new with nccl port none / 测试用例 init new with nccl port none
```python
    @patch("sglang.srt.server_args.get_free_port")
    @patch("sglang.srt.server_args.tempfile.NamedTemporaryFile")
    def test_init_new_with_nccl_port_none(self, mock_temp_file, mock_get_free_port):
        """Test that get_free_port() is called when nccl_port is None"""
        mock_temp_file.return_value.name = "temp_file"
        mock_get_free_port.return_value = 45678  # Mock ephemeral port

        # Use MagicMock here to verify get_free_port is called
        server_args = MagicMock()
        server_args.nccl_port = None
        server_args.enable_dp_attention = False
        server_args.tokenizer_worker_num = 1

        port_args = PortArgs.init_new(server_args)

        # Verify get_free_port was called
        mock_get_free_port.assert_called_once()

        # Verify the returned port is used
        self.assertEqual(port_args.nccl_port, 45678)
```
**EN:** Test that get_free_port() is called when nccl_port is None This test exercises `test_init_new_with_nccl_port_none` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that get_free_port() is called when nccl_port is None 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_init_new_with_nccl_port_none`。

### Lines 112-126: test case init new standard case / 测试用例 init new standard case
```python
    @patch("sglang.srt.server_args.tempfile.NamedTemporaryFile")
    def test_init_new_standard_case(self, mock_temp_file):
        mock_temp_file.return_value.name = "temp_file"

        server_args = ServerArgs(model_path="dummy")
        server_args.port = 30000
        server_args.nccl_port = None
        server_args.enable_dp_attention = False

        port_args = PortArgs.init_new(server_args)

        self.assertTrue(port_args.tokenizer_ipc_name.startswith("ipc://"))
        self.assertTrue(port_args.scheduler_input_ipc_name.startswith("ipc://"))
        self.assertTrue(port_args.detokenizer_ipc_name.startswith("ipc://"))
        self.assertIsInstance(port_args.nccl_port, int)
```
**EN:** This test exercises `test_init_new_standard_case` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_init_new_standard_case`。

### Lines 128-144: test case init new with single node dp attention / 测试用例 init new with single node dp attention
```python
    def test_init_new_with_single_node_dp_attention(self):

        server_args = ServerArgs(model_path="dummy")
        server_args.port = 30000
        server_args.nccl_port = None
        server_args.enable_dp_attention = True
        server_args.nnodes = 1
        server_args.dist_init_addr = None

        port_args = PortArgs.init_new(server_args)

        self.assertTrue(port_args.tokenizer_ipc_name.startswith("tcp://127.0.0.1:"))
        self.assertTrue(
            port_args.scheduler_input_ipc_name.startswith("tcp://127.0.0.1:")
        )
        self.assertTrue(port_args.detokenizer_ipc_name.startswith("tcp://127.0.0.1:"))
        self.assertIsInstance(port_args.nccl_port, int)
```
**EN:** This test exercises `test_init_new_with_single_node_dp_attention` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_init_new_with_single_node_dp_attention`。

### Lines 146-161: test case init new with dp rank / 测试用例 init new with dp rank
```python
    def test_init_new_with_dp_rank(self):
        server_args = ServerArgs(model_path="dummy")
        server_args.port = 30000
        server_args.nccl_port = None
        server_args.enable_dp_attention = True
        server_args.nnodes = 1
        server_args.dist_init_addr = "192.168.1.1:25000"

        worker_ports = [25006, 25007, 25008, 25009]
        port_args = PortArgs.init_new(server_args, dp_rank=2, worker_ports=worker_ports)

        self.assertTrue(port_args.scheduler_input_ipc_name.endswith(":25008"))

        self.assertTrue(port_args.tokenizer_ipc_name.startswith("tcp://192.168.1.1:"))
        self.assertTrue(port_args.detokenizer_ipc_name.startswith("tcp://192.168.1.1:"))
        self.assertIsInstance(port_args.nccl_port, int)
```
**EN:** This test exercises `test_init_new_with_dp_rank` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_init_new_with_dp_rank`。

### Lines 163-179: test case init new with ipv4 address / 测试用例 init new with ipv4 address
```python
    def test_init_new_with_ipv4_address(self):
        server_args = ServerArgs(model_path="dummy")
        server_args.port = 30000
        server_args.nccl_port = None

        server_args.enable_dp_attention = True
        server_args.nnodes = 2
        server_args.dist_init_addr = "192.168.1.1:25000"

        port_args = PortArgs.init_new(server_args)

        self.assertTrue(port_args.tokenizer_ipc_name.startswith("tcp://192.168.1.1:"))
        self.assertTrue(
            port_args.scheduler_input_ipc_name.startswith("tcp://192.168.1.1:")
        )
        self.assertTrue(port_args.detokenizer_ipc_name.startswith("tcp://192.168.1.1:"))
        self.assertIsInstance(port_args.nccl_port, int)
```
**EN:** This test exercises `test_init_new_with_ipv4_address` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_init_new_with_ipv4_address`。

### Lines 181-193: test case init new with malformed ipv4 address / 测试用例 init new with malformed ipv4 address
```python
    def test_init_new_with_malformed_ipv4_address(self):
        server_args = ServerArgs(model_path="dummy")
        server_args.port = 30000
        server_args.nccl_port = None

        server_args.enable_dp_attention = True
        server_args.nnodes = 2
        server_args.dist_init_addr = "192.168.1.1"

        with self.assertRaises(ValueError) as context:
            PortArgs.init_new(server_args)

        self.assertIn("Missing port", str(context.exception))
```
**EN:** This test exercises `test_init_new_with_malformed_ipv4_address` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_init_new_with_malformed_ipv4_address`。

### Lines 195-205: test case init new with malformed ipv4 address invalid port / 测试用例 init new with malformed ipv4 address invalid port
```python
    def test_init_new_with_malformed_ipv4_address_invalid_port(self):
        server_args = ServerArgs(model_path="dummy")
        server_args.port = 30000
        server_args.nccl_port = None

        server_args.enable_dp_attention = True
        server_args.nnodes = 2
        server_args.dist_init_addr = "192.168.1.1:abc"

        with self.assertRaises(ValueError):
            PortArgs.init_new(server_args)
```
**EN:** This test exercises `test_init_new_with_malformed_ipv4_address_invalid_port` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_init_new_with_malformed_ipv4_address_invalid_port`。

### Lines 208-208: class TestSSLArgs declaration / 类 TestSSLArgs 声明
```python
class TestSSLArgs(unittest.TestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `unittest.TestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `unittest.TestCase`。

### Lines 209-214: test case default ssl fields are none / 测试用例 default ssl fields are none
```python
    def test_default_ssl_fields_are_none(self):
        server_args = ServerArgs(model_path="dummy")
        self.assertIsNone(server_args.ssl_keyfile)
        self.assertIsNone(server_args.ssl_certfile)
        self.assertIsNone(server_args.ssl_ca_certs)
        self.assertIsNone(server_args.ssl_keyfile_password)
```
**EN:** This test exercises `test_default_ssl_fields_are_none` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_default_ssl_fields_are_none`。

### Lines 216-219: test case ssl keyfile without certfile raises / 测试用例 ssl keyfile without certfile raises
```python
    def test_ssl_keyfile_without_certfile_raises(self):
        with self.assertRaises(ValueError) as context:
            ServerArgs(model_path="dummy", ssl_keyfile="key.pem")
        self.assertIn("--ssl-certfile", str(context.exception))
```
**EN:** This test exercises `test_ssl_keyfile_without_certfile_raises` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_ssl_keyfile_without_certfile_raises`。

### Lines 221-224: test case ssl certfile without keyfile raises / 测试用例 ssl certfile without keyfile raises
```python
    def test_ssl_certfile_without_keyfile_raises(self):
        with self.assertRaises(ValueError) as context:
            ServerArgs(model_path="dummy", ssl_certfile="cert.pem")
        self.assertIn("--ssl-keyfile", str(context.exception))
```
**EN:** This test exercises `test_ssl_certfile_without_keyfile_raises` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_ssl_certfile_without_keyfile_raises`。

### Lines 226-232: test case ssl both keyfile and certfile accepted / 测试用例 ssl both keyfile and certfile accepted
```python
    @patch("os.path.isfile", return_value=True)
    def test_ssl_both_keyfile_and_certfile_accepted(self, _mock_isfile):
        server_args = ServerArgs(
            model_path="dummy", ssl_keyfile="key.pem", ssl_certfile="cert.pem"
        )
        self.assertEqual(server_args.ssl_keyfile, "key.pem")
        self.assertEqual(server_args.ssl_certfile, "cert.pem")
```
**EN:** This test exercises `test_ssl_both_keyfile_and_certfile_accepted` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_ssl_both_keyfile_and_certfile_accepted`。

### Lines 234-236: test case url returns http without ssl / 测试用例 url returns http without ssl
```python
    def test_url_returns_http_without_ssl(self):
        server_args = ServerArgs(model_path="dummy")
        self.assertTrue(server_args.url().startswith("http://"))
```
**EN:** This test exercises `test_url_returns_http_without_ssl` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_url_returns_http_without_ssl`。

### Lines 238-240: test case url rewrites all interfaces to loopback / 测试用例 url rewrites all interfaces to loopback
```python
    def test_url_rewrites_all_interfaces_to_loopback(self):
        server_args = ServerArgs(model_path="dummy", host="0.0.0.0")
        self.assertEqual(server_args.url(), "http://127.0.0.1:30000")
```
**EN:** This test exercises `test_url_rewrites_all_interfaces_to_loopback` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_url_rewrites_all_interfaces_to_loopback`。

### Lines 242-244: test case url rewrites empty host to loopback / 测试用例 url rewrites empty host to loopback
```python
    def test_url_rewrites_empty_host_to_loopback(self):
        server_args = ServerArgs(model_path="dummy", host="")
        self.assertEqual(server_args.url(), "http://127.0.0.1:30000")
```
**EN:** This test exercises `test_url_rewrites_empty_host_to_loopback` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_url_rewrites_empty_host_to_loopback`。

### Lines 246-251: test case url returns https with ssl / 测试用例 url returns https with ssl
```python
    @patch("os.path.isfile", return_value=True)
    def test_url_returns_https_with_ssl(self, _mock_isfile):
        server_args = ServerArgs(
            model_path="dummy", ssl_keyfile="key.pem", ssl_certfile="cert.pem"
        )
        self.assertTrue(server_args.url().startswith("https://"))
```
**EN:** This test exercises `test_url_returns_https_with_ssl` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_url_returns_https_with_ssl`。

### Lines 253-272: test case ssl cli args parsed / 测试用例 ssl cli args parsed
```python
    @patch("os.path.isfile", return_value=True)
    def test_ssl_cli_args_parsed(self, _mock_isfile):
        server_args = prepare_server_args(
            [
                "--model-path",
                "dummy",
                "--ssl-keyfile",
                "key.pem",
                "--ssl-certfile",
                "cert.pem",
                "--ssl-ca-certs",
                "ca.pem",
                "--ssl-keyfile-password",
                "secret",
            ]
        )
        self.assertEqual(server_args.ssl_keyfile, "key.pem")
        self.assertEqual(server_args.ssl_certfile, "cert.pem")
        self.assertEqual(server_args.ssl_ca_certs, "ca.pem")
        self.assertEqual(server_args.ssl_keyfile_password, "secret")
```
**EN:** This test exercises `test_ssl_cli_args_parsed` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_ssl_cli_args_parsed`。

### Lines 274-276: test case ssl verify without ssl / 测试用例 ssl verify without ssl
```python
    def test_ssl_verify_without_ssl(self):
        server_args = ServerArgs(model_path="dummy")
        self.assertIs(server_args.ssl_verify(), True)
```
**EN:** This test exercises `test_ssl_verify_without_ssl` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_ssl_verify_without_ssl`。

### Lines 278-283: test case ssl verify with ssl no ca / 测试用例 ssl verify with ssl no ca
```python
    @patch("os.path.isfile", return_value=True)
    def test_ssl_verify_with_ssl_no_ca(self, _mock_isfile):
        server_args = ServerArgs(
            model_path="dummy", ssl_keyfile="key.pem", ssl_certfile="cert.pem"
        )
        self.assertIs(server_args.ssl_verify(), False)
```
**EN:** This test exercises `test_ssl_verify_with_ssl_no_ca` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_ssl_verify_with_ssl_no_ca`。

### Lines 285-293: test case ssl verify with ssl and ca / 测试用例 ssl verify with ssl and ca
```python
    @patch("os.path.isfile", return_value=True)
    def test_ssl_verify_with_ssl_and_ca(self, _mock_isfile):
        server_args = ServerArgs(
            model_path="dummy",
            ssl_keyfile="key.pem",
            ssl_certfile="cert.pem",
            ssl_ca_certs="ca.pem",
        )
        self.assertEqual(server_args.ssl_verify(), "ca.pem")
```
**EN:** This test exercises `test_ssl_verify_with_ssl_and_ca` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_ssl_verify_with_ssl_and_ca`。

### Lines 295-298: test case ssl ca certs without certfile raises / 测试用例 ssl ca certs without certfile raises
```python
    def test_ssl_ca_certs_without_certfile_raises(self):
        with self.assertRaises(ValueError) as context:
            ServerArgs(model_path="dummy", ssl_ca_certs="ca.pem")
        self.assertIn("--ssl-ca-certs", str(context.exception))
```
**EN:** This test exercises `test_ssl_ca_certs_without_certfile_raises` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_ssl_ca_certs_without_certfile_raises`。

### Lines 300-303: test case ssl keyfile password without certfile raises / 测试用例 ssl keyfile password without certfile raises
```python
    def test_ssl_keyfile_password_without_certfile_raises(self):
        with self.assertRaises(ValueError) as context:
            ServerArgs(model_path="dummy", ssl_keyfile_password="secret")
        self.assertIn("--ssl-keyfile-password", str(context.exception))
```
**EN:** This test exercises `test_ssl_keyfile_password_without_certfile_raises` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_ssl_keyfile_password_without_certfile_raises`。

### Lines 305-312: test case ssl keyfile not found raises / 测试用例 ssl keyfile not found raises
```python
    def test_ssl_keyfile_not_found_raises(self):
        with self.assertRaises(ValueError) as context:
            ServerArgs(
                model_path="dummy",
                ssl_keyfile="/nonexistent/key.pem",
                ssl_certfile="/nonexistent/cert.pem",
            )
        self.assertIn("not found", str(context.exception))
```
**EN:** This test exercises `test_ssl_keyfile_not_found_raises` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_ssl_keyfile_not_found_raises`。

### Lines 314-322: test case ssl certfile not found raises / 测试用例 ssl certfile not found raises
```python
    def test_ssl_certfile_not_found_raises(self):
        with tempfile.NamedTemporaryFile(suffix=".pem") as keyfile:
            with self.assertRaises(ValueError) as context:
                ServerArgs(
                    model_path="dummy",
                    ssl_keyfile=keyfile.name,
                    ssl_certfile="/nonexistent/cert.pem",
                )
            self.assertIn("SSL certificate file not found", str(context.exception))
```
**EN:** This test exercises `test_ssl_certfile_not_found_raises` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_ssl_certfile_not_found_raises`。

### Lines 324-336: test case ssl ca certs not found raises / 测试用例 ssl ca certs not found raises
```python
    def test_ssl_ca_certs_not_found_raises(self):
        with tempfile.NamedTemporaryFile(suffix=".pem") as keyfile:
            with tempfile.NamedTemporaryFile(suffix=".pem") as certfile:
                with self.assertRaises(ValueError) as context:
                    ServerArgs(
                        model_path="dummy",
                        ssl_keyfile=keyfile.name,
                        ssl_certfile=certfile.name,
                        ssl_ca_certs="/nonexistent/ca.pem",
                    )
                self.assertIn(
                    "SSL CA certificates file not found", str(context.exception)
                )
```
**EN:** This test exercises `test_ssl_ca_certs_not_found_raises` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_ssl_ca_certs_not_found_raises`。

### Lines 338-340: test case enable ssl refresh default false / 测试用例 enable ssl refresh default false
```python
    def test_enable_ssl_refresh_default_false(self):
        server_args = ServerArgs(model_path="dummy")
        self.assertFalse(server_args.enable_ssl_refresh)
```
**EN:** This test exercises `test_enable_ssl_refresh_default_false` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_enable_ssl_refresh_default_false`。

### Lines 342-346: test case enable ssl refresh without ssl raises / 测试用例 enable ssl refresh without ssl raises
```python
    def test_enable_ssl_refresh_without_ssl_raises(self):
        with self.assertRaises(ValueError) as context:
            ServerArgs(model_path="dummy", enable_ssl_refresh=True)
        self.assertIn("--enable-ssl-refresh", str(context.exception))
        self.assertIn("--ssl-certfile", str(context.exception))
```
**EN:** This test exercises `test_enable_ssl_refresh_without_ssl_raises` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_enable_ssl_refresh_without_ssl_raises`。

### Lines 348-356: test case enable ssl refresh with ssl accepted / 测试用例 enable ssl refresh with ssl accepted
```python
    @patch("os.path.isfile", return_value=True)
    def test_enable_ssl_refresh_with_ssl_accepted(self, _mock_isfile):
        server_args = ServerArgs(
            model_path="dummy",
            ssl_keyfile="key.pem",
            ssl_certfile="cert.pem",
            enable_ssl_refresh=True,
        )
        self.assertTrue(server_args.enable_ssl_refresh)
```
**EN:** This test exercises `test_enable_ssl_refresh_with_ssl_accepted` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_enable_ssl_refresh_with_ssl_accepted`。

### Lines 358-371: test case enable ssl refresh cli flag / 测试用例 enable ssl refresh cli flag
```python
    @patch("os.path.isfile", return_value=True)
    def test_enable_ssl_refresh_cli_flag(self, _mock_isfile):
        server_args = prepare_server_args(
            [
                "--model-path",
                "dummy",
                "--ssl-keyfile",
                "key.pem",
                "--ssl-certfile",
                "cert.pem",
                "--enable-ssl-refresh",
            ]
        )
        self.assertTrue(server_args.enable_ssl_refresh)
```
**EN:** This test exercises `test_enable_ssl_refresh_cli_flag` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_enable_ssl_refresh_cli_flag`。

### Lines 374-374: class TestHiCacheArgs declaration / 类 TestHiCacheArgs 声明
```python
class TestHiCacheArgs(unittest.TestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `unittest.TestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `unittest.TestCase`。

### Lines 375-379: method make args / 方法 make args
```python
    def _make_args(self, **overrides) -> ServerArgs:
        args = ServerArgs(model_path="dummy")
        for key, value in overrides.items():
            setattr(args, key, value)
        return args
```
**EN:** This block implements `_make_args` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_make_args`，承担模块行为中的一个聚焦逻辑片段。

### Lines 381-392: method assert hicache fields / 方法 assert hicache fields
```python
    def _assert_hicache_fields(
        self,
        args: ServerArgs,
        *,
        expected_io_backend: str,
        expected_mem_layout: str,
        expected_decode_backend: str | None = None,
    ):
        self.assertEqual(args.hicache_io_backend, expected_io_backend)
        self.assertEqual(args.hicache_mem_layout, expected_mem_layout)
        if expected_decode_backend is not None:
            self.assertEqual(args.decode_attention_backend, expected_decode_backend)
```
**EN:** This block implements `_assert_hicache_fields` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_assert_hicache_fields`，承担模块行为中的一个聚焦逻辑片段。

### Lines 394-449: test case hicache io backend and mem layout compatibility / 测试用例 hicache io backend and mem layout compatibility
```python
    def test_hicache_io_backend_and_mem_layout_compatibility(self):
        cases = [
            {
                "name": "kernel_with_page_first_direct",
                "overrides": {
                    "enable_hierarchical_cache": True,
                    "hicache_io_backend": "kernel",
                    "hicache_mem_layout": "page_first_direct",
                },
                "expected_io_backend": "direct",
                "expected_mem_layout": "page_first_direct",
            },
            {
                "name": "direct_with_page_first",
                "overrides": {
                    "enable_hierarchical_cache": True,
                    "hicache_io_backend": "direct",
                    "hicache_mem_layout": "page_first",
                },
                "expected_io_backend": "direct",
                "expected_mem_layout": "page_first_direct",
            },
            {
                "name": "mooncake_with_layer_first",
                "overrides": {
                    "enable_hierarchical_cache": True,
                    "hicache_storage_backend": "mooncake",
                    "hicache_io_backend": "direct",
                    "hicache_mem_layout": "layer_first",
                },
                "expected_io_backend": "direct",
                "expected_mem_layout": "page_first_direct",
            },
            {
                "name": "fa3_kernel_with_explicit_decode_backend",
                "overrides": {
                    "enable_hierarchical_cache": True,
                    "hicache_io_backend": "kernel",
                    "hicache_mem_layout": "page_first",
                    "attention_backend": "triton",
                    "decode_attention_backend": "fa3",
                },
                "expected_io_backend": "direct",
                "expected_mem_layout": "page_first_direct",
            },
        ]

        for case in cases:
            with self.subTest(case=case["name"]):
                args = self._make_args(**case["overrides"])
                args._handle_hicache()
                self._assert_hicache_fields(
                    args,
                    expected_io_backend=case["expected_io_backend"],
                    expected_mem_layout=case["expected_mem_layout"],
                )
```
**EN:** This test exercises `test_hicache_io_backend_and_mem_layout_compatibility` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_hicache_io_backend_and_mem_layout_compatibility`。

### Lines 451-465: test case decode attention backend with implicit fa3 / 测试用例 decode attention backend with implicit fa3
```python
    @patch.object(ServerArgs, "use_mla_backend", return_value=False)
    @patch("sglang.srt.server_args.is_flashinfer_available", return_value=False)
    def test_decode_attention_backend_with_implicit_fa3(
        self, _mock_flashinfer, _mock_use_mla_backend
    ):
        args = self._make_args(
            enable_hierarchical_cache=True,
            hicache_io_backend="kernel",
            attention_backend="fa3",
            decode_attention_backend=None,
        )

        args._handle_hicache()

        self.assertEqual(args.decode_attention_backend, "triton")
```
**EN:** This test exercises `test_decode_attention_backend_with_implicit_fa3` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_decode_attention_backend_with_implicit_fa3`。

### Lines 468-468: class TestNgramExternalSamArgs declaration / 类 TestNgramExternalSamArgs 声明
```python
class TestNgramExternalSamArgs(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 469-489: test case prepare server args parses external sam args / 测试用例 prepare server args parses external sam args
```python
    def test_prepare_server_args_parses_external_sam_args(self):
        server_args = prepare_server_args(
            [
                "--model-path",
                "dummy",
                "--speculative-algorithm",
                "NGRAM",
                "--speculative-ngram-external-corpus-path",
                "/tmp/ngram-corpus.jsonl",
                "--speculative-ngram-external-sam-budget",
                "4",
                "--speculative-ngram-external-corpus-max-tokens",
                "128",
            ]
        )
        self.assertEqual(
            server_args.speculative_ngram_external_corpus_path,
            "/tmp/ngram-corpus.jsonl",
        )
        self.assertEqual(server_args.speculative_ngram_external_sam_budget, 4)
        self.assertEqual(server_args.speculative_ngram_external_corpus_max_tokens, 128)
```
**EN:** This test exercises `test_prepare_server_args_parses_external_sam_args` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_prepare_server_args_parses_external_sam_args`。

### Lines 491-498: method make dummy ngram args / 方法 make dummy ngram args
```python
    def _make_dummy_ngram_args(self, **overrides):
        args = ServerArgs(model_path="dummy")
        args.speculative_algorithm = "NGRAM"
        args.speculative_num_draft_tokens = 12
        args.device = "cuda"
        for key, value in overrides.items():
            setattr(args, key, value)
        return args
```
**EN:** This block implements `_make_dummy_ngram_args` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_make_dummy_ngram_args`，承担模块行为中的一个聚焦逻辑片段。

### Lines 500-508: test case external sam budget must fit draft budget / 测试用例 external sam budget must fit draft budget
```python
    def test_external_sam_budget_must_fit_draft_budget(self):
        args = self._make_dummy_ngram_args(
            speculative_num_draft_tokens=4,
            speculative_ngram_external_corpus_path="/tmp/ngram-corpus.jsonl",
            speculative_ngram_external_sam_budget=4,
        )
        with self.assertRaises(ValueError) as context:
            handle_speculative_decoding(args)
        self.assertIn("speculative_num_draft_tokens - 1", str(context.exception))
```
**EN:** This test exercises `test_external_sam_budget_must_fit_draft_budget` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_external_sam_budget_must_fit_draft_budget`。

### Lines 510-518: test case external corpus max tokens must be positive / 测试用例 external corpus max tokens must be positive
```python
    def test_external_corpus_max_tokens_must_be_positive(self):
        args = self._make_dummy_ngram_args(
            speculative_ngram_external_corpus_path="/tmp/ngram-corpus.jsonl",
            speculative_ngram_external_sam_budget=2,
            speculative_ngram_external_corpus_max_tokens=0,
        )
        with self.assertRaises(ValueError) as context:
            handle_speculative_decoding(args)
        self.assertIn("external-corpus-max-tokens", str(context.exception))
```
**EN:** This test exercises `test_external_corpus_max_tokens_must_be_positive` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_external_corpus_max_tokens_must_be_positive`。

### Lines 521-521: class TestDeepEPWaterfillArgs declaration / 类 TestDeepEPWaterfillArgs 声明
```python
class TestDeepEPWaterfillArgs(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 522-533: test case waterfill enforces shared experts fusion / 测试用例 waterfill enforces shared experts fusion
```python
    def test_waterfill_enforces_shared_experts_fusion(self):
        server_args = ServerArgs(
            model_path="dummy",
            moe_a2a_backend="deepep",
            enable_deepep_waterfill=True,
            disable_shared_experts_fusion=True,
        )
        # dummy-model path short-circuits __post_init__; invoke the handler directly.
        server_args._handle_a2a_moe()

        self.assertFalse(server_args.disable_shared_experts_fusion)
        self.assertTrue(server_args.enforce_shared_experts_fusion)
```
**EN:** This test exercises `test_waterfill_enforces_shared_experts_fusion` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_waterfill_enforces_shared_experts_fusion`。

### Lines 535-545: test case waterfill overrides moe a2a backend to deepep / 测试用例 waterfill overrides moe a2a backend to deepep
```python
    def test_waterfill_overrides_moe_a2a_backend_to_deepep(self):
        server_args = ServerArgs(
            model_path="dummy",
            moe_a2a_backend="none",
            enable_deepep_waterfill=True,
        )
        # dummy-model path short-circuits __post_init__; invoke the handler directly.
        server_args._handle_a2a_moe()

        self.assertEqual(server_args.moe_a2a_backend, "deepep")
        self.assertTrue(server_args.enforce_shared_experts_fusion)
```
**EN:** This test exercises `test_waterfill_overrides_moe_a2a_backend_to_deepep` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_waterfill_overrides_moe_a2a_backend_to_deepep`。

### Lines 547-559: test case waterfill supports deepep low latency mode / 测试用例 waterfill supports deepep low latency mode
```python
    def test_waterfill_supports_deepep_low_latency_mode(self):
        server_args = ServerArgs(
            model_path="dummy",
            moe_a2a_backend="deepep",
            enable_deepep_waterfill=True,
            deepep_mode="low_latency",
        )
        # dummy-model path short-circuits __post_init__; invoke the handler directly.
        server_args._handle_a2a_moe()

        self.assertEqual(server_args.deepep_mode, "low_latency")
        self.assertFalse(server_args.disable_cuda_graph)
        self.assertTrue(server_args.enforce_shared_experts_fusion)
```
**EN:** This test exercises `test_waterfill_supports_deepep_low_latency_mode` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_waterfill_supports_deepep_low_latency_mode`。

### Lines 562-562: class TestPrefillOnlyDisableKvCache declaration / 类 TestPrefillOnlyDisableKvCache 声明
```python
class TestPrefillOnlyDisableKvCache(unittest.TestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `unittest.TestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `unittest.TestCase`。

### Lines 563-574: supporting statements / 辅助语句
```python
    """Validation for --prefill-only-disable-kv-cache.

    The flag wires NoOpMHATokenToKVPool, which is only safe when:
      - the engine is in embedding mode (fa_skip_kv_cache active in FA backend),
      - chunked_prefill_size == -1 (no inter-chunk K/V reuse),
      - disable_radix_cache (radix cache otherwise indexes empty pool slots),
      - no context-parallel attention (CP writes to the pool via set_kv_buffer),
      - no HiSparse (uses a different pool family),
      - kv_cache_dtype != fp4_e2m1 (FP4 pool is a separate allocation path).
    All other configurations must be rejected at __post_init__ time so users
    get a clear error before model load.
    """
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 576-585: method base kwargs / 方法 base kwargs
```python
    def _base_kwargs(self, **overrides):
        kwargs = dict(
            model_path="dummy",
            is_embedding=True,
            chunked_prefill_size=-1,
            disable_radix_cache=True,
            prefill_only_disable_kv_cache=True,
        )
        kwargs.update(overrides)
        return kwargs
```
**EN:** This block implements `_base_kwargs` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_base_kwargs`，承担模块行为中的一个聚焦逻辑片段。

### Lines 587-589: test case valid minimal config constructs / 测试用例 valid minimal config constructs
```python
    def test_valid_minimal_config_constructs(self):
        sa = ServerArgs(**self._base_kwargs())
        self.assertTrue(sa.prefill_only_disable_kv_cache)
```
**EN:** This test exercises `test_valid_minimal_config_constructs` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_valid_minimal_config_constructs`。

### Lines 591-593: test case rejects when not embedding / 测试用例 rejects when not embedding
```python
    def test_rejects_when_not_embedding(self):
        with self.assertRaisesRegex(ValueError, "requires --is-embedding"):
            ServerArgs(**self._base_kwargs(is_embedding=False))
```
**EN:** This test exercises `test_rejects_when_not_embedding` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_rejects_when_not_embedding`。

### Lines 595-597: test case rejects when chunked prefill size not minus one / 测试用例 rejects when chunked prefill size not minus one
```python
    def test_rejects_when_chunked_prefill_size_not_minus_one(self):
        with self.assertRaisesRegex(ValueError, "--chunked-prefill-size=-1"):
            ServerArgs(**self._base_kwargs(chunked_prefill_size=8192))
```
**EN:** This test exercises `test_rejects_when_chunked_prefill_size_not_minus_one` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_rejects_when_chunked_prefill_size_not_minus_one`。

### Lines 599-601: test case rejects when radix cache enabled / 测试用例 rejects when radix cache enabled
```python
    def test_rejects_when_radix_cache_enabled(self):
        with self.assertRaisesRegex(ValueError, "--disable-radix-cache"):
            ServerArgs(**self._base_kwargs(disable_radix_cache=False))
```
**EN:** This test exercises `test_rejects_when_radix_cache_enabled` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_rejects_when_radix_cache_enabled`。

### Lines 603-605: test case rejects attn cp size greater than one / 测试用例 rejects attn cp size greater than one
```python
    def test_rejects_attn_cp_size_greater_than_one(self):
        with self.assertRaisesRegex(ValueError, "--attn-cp-size"):
            ServerArgs(**self._base_kwargs(attn_cp_size=2, tp_size=2))
```
**EN:** This test exercises `test_rejects_attn_cp_size_greater_than_one` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_rejects_attn_cp_size_greater_than_one`。

### Lines 607-609: test case rejects prefill context parallel / 测试用例 rejects prefill context parallel
```python
    def test_rejects_prefill_context_parallel(self):
        with self.assertRaisesRegex(ValueError, "--enable-prefill-context-parallel"):
            ServerArgs(**self._base_kwargs(enable_prefill_context_parallel=True))
```
**EN:** This test exercises `test_rejects_prefill_context_parallel` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_rejects_prefill_context_parallel`。

### Lines 611-613: test case rejects hisparse / 测试用例 rejects hisparse
```python
    def test_rejects_hisparse(self):
        with self.assertRaisesRegex(ValueError, "--enable-hisparse"):
            ServerArgs(**self._base_kwargs(enable_hisparse=True))
```
**EN:** This test exercises `test_rejects_hisparse` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_rejects_hisparse`。

### Lines 615-617: test case rejects fp4 kv cache / 测试用例 rejects fp4 kv cache
```python
    def test_rejects_fp4_kv_cache(self):
        with self.assertRaisesRegex(ValueError, "fp4_e2m1"):
            ServerArgs(**self._base_kwargs(kv_cache_dtype="fp4_e2m1"))
```
**EN:** This test exercises `test_rejects_fp4_kv_cache` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_rejects_fp4_kv_cache`。

### Lines 620-621: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `TestPrepareServerArgs`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestLoadBalanceMethod`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestPortArgs`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestSSLArgs`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestHiCacheArgs`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestNgramExternalSamArgs`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestDeepEPWaterfillArgs`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestPrefillOnlyDisableKvCache`: Validation for --prefill-only-disable-kv-cache. / 用于组织相关测试、夹具或辅助方法。
- `TestPrepareServerArgs.test_prepare_server_args`: This test exercises `test_prepare_server_args` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_prepare_server_args`。
- `TestLoadBalanceMethod.test_non_pd_defaults_to_round_robin`: This test exercises `test_non_pd_defaults_to_round_robin` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_non_pd_defaults_to_round_robin`。
- `TestLoadBalanceMethod.test_pd_prefill_defaults_to_follow_bootstrap_room`: This test exercises `test_pd_prefill_defaults_to_follow_bootstrap_room` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_pd_prefill_defaults_to_follow_bootstrap_room`。
- `TestLoadBalanceMethod.test_pd_decode_defaults_to_round_robin`: This test exercises `test_pd_decode_defaults_to_round_robin` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_pd_decode_defaults_to_round_robin`。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `json`, `tempfile`, `unittest`, `unittest.mock`
- **Internal modules / 内部模块**: `sglang.srt.arg_groups.speculative_hook`, `sglang.srt.server_args`, `sglang.test.ci.ci_register`, `sglang.test.test_utils`

- **Total lines / 总行数**: 621
