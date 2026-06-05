# test_numa_utils.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/utils/test_numa_utils.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates numa utils behavior in SGLang's utils area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 工具 领域中与 numa utils 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-10: module imports and dependencies / 模块导入与依赖
```python
import ctypes
import unittest
from unittest.mock import MagicMock, patch

from sglang.srt.utils.numa_utils import (
    _is_numa_available,
    _query_numa_node_for_gpu,
    get_numa_node_if_available,
)
from sglang.test.ci.ci_register import register_cpu_ci, register_cuda_ci
```
**EN:** This block imports the modules needed by the rest of the file, including `ctypes`, `unittest`, `unittest.mock`, `sglang.srt.utils.numa_utils`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `ctypes`, `unittest`, `unittest.mock`, `sglang.srt.utils.numa_utils`。

### Lines 12-14: CI registration and metadata / CI 注册与元数据
```python
register_cpu_ci(est_time=7, suite="base-a-test-cpu")
register_cuda_ci(est_time=10, stage="base-c", runner_config="4-gpu-gb200")
register_cuda_ci(est_time=10, stage="base-c", runner_config="8-gpu-b200")
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through register_cpu_ci, register_cuda_ci.
**CN:** 该代码块通过 register_cpu_ci, register_cuda_ci 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 17-17: class TestIsNumaAvailable declaration / 类 TestIsNumaAvailable 声明
```python
class TestIsNumaAvailable(unittest.TestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `unittest.TestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `unittest.TestCase`。

### Lines 18-18: supporting statements / 辅助语句
```python
    """Tests for _is_numa_available on both NUMA and non-NUMA systems."""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 20-22: test case returns false when not cuda / 测试用例 returns false when not cuda
```python
    @patch("sglang.srt.utils.numa_utils._is_cuda", False)
    def test_returns_false_when_not_cuda(self):
        self.assertFalse(_is_numa_available())
```
**EN:** This test exercises `test_returns_false_when_not_cuda` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_returns_false_when_not_cuda`。

### Lines 24-27: test case returns false when no numa nodes / 测试用例 returns false when no numa nodes
```python
    @patch("sglang.srt.utils.numa_utils._is_cuda", True)
    @patch("os.path.isdir", return_value=False)
    def test_returns_false_when_no_numa_nodes(self, _mock_isdir):
        self.assertFalse(_is_numa_available())
```
**EN:** This test exercises `test_returns_false_when_no_numa_nodes` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_returns_false_when_no_numa_nodes`。

### Lines 29-38: test case returns false when affinity constrained / 测试用例 returns false when affinity constrained
```python
    @patch("sglang.srt.utils.numa_utils._is_cuda", True)
    @patch("os.path.isdir", return_value=True)
    @patch("sglang.srt.utils.numa_utils.psutil")
    def test_returns_false_when_affinity_constrained(self, mock_psutil, _mock_isdir):
        mock_process = MagicMock()
        mock_process.cpu_affinity.return_value = [0, 1]
        mock_psutil.Process.return_value = mock_process
        mock_psutil.cpu_count.return_value = 128

        self.assertFalse(_is_numa_available())
```
**EN:** This test exercises `test_returns_false_when_affinity_constrained` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_returns_false_when_affinity_constrained`。

### Lines 40-54: test case returns true on numa system with full affinity / 测试用例 returns true on numa system with full affinity
```python
    @patch("sglang.srt.utils.numa_utils._can_set_mempolicy", return_value=True)
    @patch("sglang.srt.utils.numa_utils.shutil.which", return_value="/usr/bin/numactl")
    @patch("sglang.srt.utils.numa_utils._is_cuda", True)
    @patch("os.path.isdir", return_value=True)
    @patch("sglang.srt.utils.numa_utils.psutil")
    def test_returns_true_on_numa_system_with_full_affinity(
        self, mock_psutil, _mock_isdir, _mock_which, _mock_mempolicy
    ):
        all_cpus = list(range(128))
        mock_process = MagicMock()
        mock_process.cpu_affinity.return_value = all_cpus
        mock_psutil.Process.return_value = mock_process
        mock_psutil.cpu_count.return_value = 128

        self.assertTrue(_is_numa_available())
```
**EN:** This test exercises `test_returns_true_on_numa_system_with_full_affinity` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_returns_true_on_numa_system_with_full_affinity`。

### Lines 56-70: test case returns false when mempolicy not permitted / 测试用例 returns false when mempolicy not permitted
```python
    @patch("sglang.srt.utils.numa_utils._can_set_mempolicy", return_value=False)
    @patch("sglang.srt.utils.numa_utils.shutil.which", return_value="/usr/bin/numactl")
    @patch("sglang.srt.utils.numa_utils._is_cuda", True)
    @patch("os.path.isdir", return_value=True)
    @patch("sglang.srt.utils.numa_utils.psutil")
    def test_returns_false_when_mempolicy_not_permitted(
        self, mock_psutil, _mock_isdir, _mock_which, _mock_mempolicy
    ):
        all_cpus = list(range(128))
        mock_process = MagicMock()
        mock_process.cpu_affinity.return_value = all_cpus
        mock_psutil.Process.return_value = mock_process
        mock_psutil.cpu_count.return_value = 128

        self.assertFalse(_is_numa_available())
```
**EN:** This test exercises `test_returns_false_when_mempolicy_not_permitted` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_returns_false_when_mempolicy_not_permitted`。

### Lines 72-87: test case isdir called with node1 path / 测试用例 isdir called with node1 path
```python
    @patch("sglang.srt.utils.numa_utils._can_set_mempolicy", return_value=True)
    @patch("sglang.srt.utils.numa_utils.shutil.which", return_value="/usr/bin/numactl")
    @patch("sglang.srt.utils.numa_utils._is_cuda", True)
    @patch("os.path.isdir", return_value=True)
    @patch("sglang.srt.utils.numa_utils.psutil")
    def test_isdir_called_with_node1_path(
        self, mock_psutil, mock_isdir, _mock_which, _mock_mempolicy
    ):
        all_cpus = list(range(8))
        mock_process = MagicMock()
        mock_process.cpu_affinity.return_value = all_cpus
        mock_psutil.Process.return_value = mock_process
        mock_psutil.cpu_count.return_value = 8

        _is_numa_available()
        mock_isdir.assert_called_with("/sys/devices/system/node/node1")
```
**EN:** This test exercises `test_isdir_called_with_node1_path` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_isdir_called_with_node1_path`。

### Lines 90-90: class TestQueryNumaNodeForGpu declaration / 类 TestQueryNumaNodeForGpu 声明
```python
class TestQueryNumaNodeForGpu(unittest.TestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `unittest.TestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `unittest.TestCase`。

### Lines 91-91: supporting statements / 辅助语句
```python
    """Tests for _query_numa_node_for_gpu with mocked pynvml."""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 93-114: test case single node affinity / 测试用例 single node affinity
```python
    @patch(
        "sglang.srt.utils.numa_utils.glob.glob",
        return_value=[
            "/sys/devices/system/node/node0",
            "/sys/devices/system/node/node1",
        ],
    )
    def test_single_node_affinity(self, _mock_glob):
        c_ulong_bits = ctypes.sizeof(ctypes.c_ulong) * 8
        # Bitmask: bit 0 set -> node 0
        node_set = [1]

        mock_pynvml = MagicMock()
        mock_pynvml.nvmlDeviceGetMemoryAffinity.return_value = node_set
        mock_pynvml.NVML_AFFINITY_SCOPE_NODE = 0

        with patch.dict("sys.modules", {"pynvml": mock_pynvml}):
            result = _query_numa_node_for_gpu(0)

        self.assertEqual(result, [0])
        mock_pynvml.nvmlInit.assert_called_once()
        mock_pynvml.nvmlShutdown.assert_called_once()
```
**EN:** This test exercises `test_single_node_affinity` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_single_node_affinity`。

### Lines 116-134: test case second node affinity / 测试用例 second node affinity
```python
    @patch(
        "sglang.srt.utils.numa_utils.glob.glob",
        return_value=[
            "/sys/devices/system/node/node0",
            "/sys/devices/system/node/node1",
        ],
    )
    def test_second_node_affinity(self, _mock_glob):
        # Bitmask: bit 1 set -> node 1
        node_set = [2]

        mock_pynvml = MagicMock()
        mock_pynvml.nvmlDeviceGetMemoryAffinity.return_value = node_set
        mock_pynvml.NVML_AFFINITY_SCOPE_NODE = 0

        with patch.dict("sys.modules", {"pynvml": mock_pynvml}):
            result = _query_numa_node_for_gpu(1)

        self.assertEqual(result, [1])
```
**EN:** This test exercises `test_second_node_affinity` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_second_node_affinity`。

### Lines 136-156: test case multiple node affinity / 测试用例 multiple node affinity
```python
    @patch(
        "sglang.srt.utils.numa_utils.glob.glob",
        return_value=[
            "/sys/devices/system/node/node0",
            "/sys/devices/system/node/node1",
            "/sys/devices/system/node/node2",
            "/sys/devices/system/node/node3",
        ],
    )
    def test_multiple_node_affinity(self, _mock_glob):
        # Bitmask: bits 1 and 3 set -> nodes 1, 3 (binary: ...1010 = 10)
        node_set = [0b1010]

        mock_pynvml = MagicMock()
        mock_pynvml.nvmlDeviceGetMemoryAffinity.return_value = node_set
        mock_pynvml.NVML_AFFINITY_SCOPE_NODE = 0

        with patch.dict("sys.modules", {"pynvml": mock_pynvml}):
            result = _query_numa_node_for_gpu(0)

        self.assertEqual(result, [1, 3])
```
**EN:** This test exercises `test_multiple_node_affinity` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_multiple_node_affinity`。

### Lines 158-175: test case no affinity / 测试用例 no affinity
```python
    @patch(
        "sglang.srt.utils.numa_utils.glob.glob",
        return_value=[
            "/sys/devices/system/node/node0",
            "/sys/devices/system/node/node1",
        ],
    )
    def test_no_affinity(self, _mock_glob):
        node_set = [0]

        mock_pynvml = MagicMock()
        mock_pynvml.nvmlDeviceGetMemoryAffinity.return_value = node_set
        mock_pynvml.NVML_AFFINITY_SCOPE_NODE = 0

        with patch.dict("sys.modules", {"pynvml": mock_pynvml}):
            result = _query_numa_node_for_gpu(0)

        self.assertEqual(result, [])
```
**EN:** This test exercises `test_no_affinity` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_no_affinity`。

### Lines 177-193: test case nvml shutdown called on success / 测试用例 nvml shutdown called on success
```python
    @patch(
        "sglang.srt.utils.numa_utils.glob.glob",
        return_value=[
            "/sys/devices/system/node/node0",
            "/sys/devices/system/node/node1",
        ],
    )
    def test_nvml_shutdown_called_on_success(self, _mock_glob):
        node_set = [1]
        mock_pynvml = MagicMock()
        mock_pynvml.nvmlDeviceGetMemoryAffinity.return_value = node_set
        mock_pynvml.NVML_AFFINITY_SCOPE_NODE = 0

        with patch.dict("sys.modules", {"pynvml": mock_pynvml}):
            _query_numa_node_for_gpu(0)

        mock_pynvml.nvmlShutdown.assert_called_once()
```
**EN:** This test exercises `test_nvml_shutdown_called_on_success` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_nvml_shutdown_called_on_success`。

### Lines 196-196: class TestGetNumaNodeIfAvailable declaration / 类 TestGetNumaNodeIfAvailable 声明
```python
class TestGetNumaNodeIfAvailable(unittest.TestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `unittest.TestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `unittest.TestCase`。

### Lines 197-197: supporting statements / 辅助语句
```python
    """Tests for get_numa_node_if_available combining _is_numa_available + _query_numa_node_for_gpu."""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 199-202: method make server args / 方法 make server args
```python
    def _make_server_args(self, numa_node=None):
        args = MagicMock()
        args.numa_node = numa_node
        return args
```
**EN:** This block implements `_make_server_args` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_make_server_args`，承担模块行为中的一个聚焦逻辑片段。

### Lines 204-209: test case returns explicit numa node from server args / 测试用例 returns explicit numa node from server args
```python
    def test_returns_explicit_numa_node_from_server_args(self):
        args = self._make_server_args(numa_node=[2, 3, 0, 1])
        self.assertEqual(get_numa_node_if_available(args, 0), 2)
        self.assertEqual(get_numa_node_if_available(args, 1), 3)
        self.assertEqual(get_numa_node_if_available(args, 2), 0)
        self.assertEqual(get_numa_node_if_available(args, 3), 1)
```
**EN:** This test exercises `test_returns_explicit_numa_node_from_server_args` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_returns_explicit_numa_node_from_server_args`。

### Lines 211-214: test case returns none when numa not available / 测试用例 returns none when numa not available
```python
    @patch("sglang.srt.utils.numa_utils._is_numa_available", return_value=False)
    def test_returns_none_when_numa_not_available(self, _mock_avail):
        args = self._make_server_args(numa_node=None)
        self.assertIsNone(get_numa_node_if_available(args, 0))
```
**EN:** This test exercises `test_returns_none_when_numa_not_available` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_returns_none_when_numa_not_available`。

### Lines 216-220: test case returns none when query returns empty / 测试用例 returns none when query returns empty
```python
    @patch("sglang.srt.utils.numa_utils._query_numa_node_for_gpu", return_value=[])
    @patch("sglang.srt.utils.numa_utils._is_numa_available", return_value=True)
    def test_returns_none_when_query_returns_empty(self, _mock_avail, _mock_gpu):
        args = self._make_server_args(numa_node=None)
        self.assertIsNone(get_numa_node_if_available(args, 0))
```
**EN:** This test exercises `test_returns_none_when_query_returns_empty` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_returns_none_when_query_returns_empty`。

### Lines 222-226: test case returns queried single node / 测试用例 returns queried single node
```python
    @patch("sglang.srt.utils.numa_utils._query_numa_node_for_gpu", return_value=[1])
    @patch("sglang.srt.utils.numa_utils._is_numa_available", return_value=True)
    def test_returns_queried_single_node(self, _mock_avail, _mock_gpu):
        args = self._make_server_args(numa_node=None)
        self.assertEqual(get_numa_node_if_available(args, 0), 1)
```
**EN:** This test exercises `test_returns_queried_single_node` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_returns_queried_single_node`。

### Lines 228-232: test case returns first node when multiple found / 测试用例 returns first node when multiple found
```python
    @patch("sglang.srt.utils.numa_utils._query_numa_node_for_gpu", return_value=[0, 2])
    @patch("sglang.srt.utils.numa_utils._is_numa_available", return_value=True)
    def test_returns_first_node_when_multiple_found(self, _mock_avail, _mock_gpu):
        args = self._make_server_args(numa_node=None)
        self.assertEqual(get_numa_node_if_available(args, 0), 0)
```
**EN:** This test exercises `test_returns_first_node_when_multiple_found` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_returns_first_node_when_multiple_found`。

### Lines 234-240: test case logs warning when multiple nodes / 测试用例 logs warning when multiple nodes
```python
    @patch("sglang.srt.utils.numa_utils._query_numa_node_for_gpu", return_value=[0, 2])
    @patch("sglang.srt.utils.numa_utils._is_numa_available", return_value=True)
    def test_logs_warning_when_multiple_nodes(self, _mock_avail, _mock_gpu):
        args = self._make_server_args(numa_node=None)
        with self.assertLogs("sglang.srt.utils.numa_utils", level="WARNING") as cm:
            get_numa_node_if_available(args, 0)
        self.assertTrue(any("Multiple NUMA nodes" in msg for msg in cm.output))
```
**EN:** This test exercises `test_logs_warning_when_multiple_nodes` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_logs_warning_when_multiple_nodes`。

### Lines 242-249: test case explicit server args takes precedence / 测试用例 explicit server args takes precedence
```python
    @patch("sglang.srt.utils.numa_utils._is_numa_available", return_value=True)
    @patch("sglang.srt.utils.numa_utils._query_numa_node_for_gpu", return_value=[1])
    def test_explicit_server_args_takes_precedence(self, _mock_gpu, _mock_avail):
        args = self._make_server_args(numa_node=[5, 6])
        result = get_numa_node_if_available(args, 0)
        self.assertEqual(result, 5)
        _mock_avail.assert_not_called()
        _mock_gpu.assert_not_called()
```
**EN:** This test exercises `test_explicit_server_args_takes_precedence` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_explicit_server_args_takes_precedence`。

### Lines 252-262: function get gpu name / 函数 get gpu name
```python
def _get_gpu_name():
    try:
        import pynvml

        pynvml.nvmlInit()
        handle = pynvml.nvmlDeviceGetHandleByIndex(0)
        name = pynvml.nvmlDeviceGetName(handle)
        pynvml.nvmlShutdown()
        return name
    except Exception:
        return ""
```
**EN:** This block implements `_get_gpu_name` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_get_gpu_name`，承担模块行为中的一个聚焦逻辑片段。

### Lines 265-265: module-level constants and configuration / 模块级常量与配置
```python
_gpu_name = _get_gpu_name()
```
**EN:** This block defines shared names such as `_gpu_name`. These values centralize configuration that later tests or helpers reuse.
**CN:** 该代码块定义了 `_gpu_name` 等共享名称，用于集中保存后续测试或辅助逻辑会复用的配置。

### Lines 268-269: class TestGB200NumaTopology declaration / 类 TestGB200NumaTopology 声明
```python
@unittest.skipUnless("GB200" in _gpu_name, "Requires GB200 hardware")
class TestGB200NumaTopology(unittest.TestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `unittest.TestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `unittest.TestCase`。

### Lines 270-270: supporting statements / 辅助语句
```python
    """Hardware test validating expected NUMA topology on GB200 (2 NUMA nodes, 4 GPUs)."""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 272-275: method make server args / 方法 make server args
```python
    def _make_server_args(self):
        args = MagicMock()
        args.numa_node = None
        return args
```
**EN:** This block implements `_make_server_args` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_make_server_args`，承担模块行为中的一个聚焦逻辑片段。

### Lines 277-286: test case gpu numa mapping / 测试用例 gpu numa mapping
```python
    def test_gpu_numa_mapping(self):
        expected = {0: 0, 1: 0, 2: 1, 3: 1}
        args = self._make_server_args()
        for gpu_id, expected_node in expected.items():
            result = get_numa_node_if_available(args, gpu_id)
            self.assertEqual(
                result,
                expected_node,
                f"GPU {gpu_id}: expected NUMA node {expected_node}, got {result}",
            )
```
**EN:** This test exercises `test_gpu_numa_mapping` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_gpu_numa_mapping`。

### Lines 289-290: class TestB200NumaTopology declaration / 类 TestB200NumaTopology 声明
```python
@unittest.skipUnless("B200" in _gpu_name, "Requires B200 hardware")
class TestB200NumaTopology(unittest.TestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `unittest.TestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `unittest.TestCase`。

### Lines 291-291: supporting statements / 辅助语句
```python
    """Hardware test validating expected NUMA topology on B200 (2 NUMA nodes, 8 GPUs)."""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 293-296: method make server args / 方法 make server args
```python
    def _make_server_args(self):
        args = MagicMock()
        args.numa_node = None
        return args
```
**EN:** This block implements `_make_server_args` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_make_server_args`，承担模块行为中的一个聚焦逻辑片段。

### Lines 298-307: test case gpu numa mapping / 测试用例 gpu numa mapping
```python
    def test_gpu_numa_mapping(self):
        expected = {0: 0, 1: 0, 2: 0, 3: 0, 4: 1, 5: 1, 6: 1, 7: 1}
        args = self._make_server_args()
        for gpu_id, expected_node in expected.items():
            result = get_numa_node_if_available(args, gpu_id)
            self.assertEqual(
                result,
                expected_node,
                f"GPU {gpu_id}: expected NUMA node {expected_node}, got {result}",
            )
```
**EN:** This test exercises `test_gpu_numa_mapping` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_gpu_numa_mapping`。

### Lines 310-311: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `TestIsNumaAvailable`: Tests for _is_numa_available on both NUMA and non-NUMA systems. / 用于组织相关测试、夹具或辅助方法。
- `TestQueryNumaNodeForGpu`: Tests for _query_numa_node_for_gpu with mocked pynvml. / 用于组织相关测试、夹具或辅助方法。
- `TestGetNumaNodeIfAvailable`: Tests for get_numa_node_if_available combining _is_numa_available + _query_numa_node_for_gpu. / 用于组织相关测试、夹具或辅助方法。
- `_get_gpu_name`: This block implements `_get_gpu_name` and captures one focused piece of the module's behavior. / 该代码块实现 `_get_gpu_name`，承担模块行为中的一个聚焦逻辑片段。
- `TestGB200NumaTopology`: Hardware test validating expected NUMA topology on GB200 (2 NUMA nodes, 4 GPUs). / 用于组织相关测试、夹具或辅助方法。
- `TestB200NumaTopology`: Hardware test validating expected NUMA topology on B200 (2 NUMA nodes, 8 GPUs). / 用于组织相关测试、夹具或辅助方法。
- `TestIsNumaAvailable.test_returns_false_when_not_cuda`: This test exercises `test_returns_false_when_not_cuda` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_returns_false_when_not_cuda`。
- `TestIsNumaAvailable.test_returns_false_when_no_numa_nodes`: This test exercises `test_returns_false_when_no_numa_nodes` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_returns_false_when_no_numa_nodes`。
- `TestIsNumaAvailable.test_returns_false_when_affinity_constrained`: This test exercises `test_returns_false_when_affinity_constrained` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_returns_false_when_affinity_constrained`。
- `TestIsNumaAvailable.test_returns_true_on_numa_system_with_full_affinity`: This test exercises `test_returns_true_on_numa_system_with_full_affinity` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_returns_true_on_numa_system_with_full_affinity`。
- `TestIsNumaAvailable.test_returns_false_when_mempolicy_not_permitted`: This test exercises `test_returns_false_when_mempolicy_not_permitted` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_returns_false_when_mempolicy_not_permitted`。
- `TestIsNumaAvailable.test_isdir_called_with_node1_path`: This test exercises `test_isdir_called_with_node1_path` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_isdir_called_with_node1_path`。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `ctypes`, `unittest`, `unittest.mock`
- **Internal modules / 内部模块**: `sglang.srt.utils.numa_utils`, `sglang.test.ci.ci_register`

- **Total lines / 总行数**: 311
