# test_platform_interface.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/unit/platforms/test_platform_interface.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates platform interface behavior in SGLang's unit / platforms area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 单元 / platforms 领域中与 platform interface 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-6: supporting statements / 辅助语句
```python
"""
Unit tests for SGLang platform abstraction layer.

Tests DeviceMixin, SRTPlatform, PlatformEnum, CpuArchEnum, DeviceCapability,
and the platform discovery / lazy initialization mechanism.
"""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 8-22: module imports and dependencies / 模块导入与依赖
```python
from unittest.mock import MagicMock, patch

import torch

from sglang.srt.platforms import _load_platform_class, _resolve_platform
from sglang.srt.platforms.cuda import CudaDeviceMixin, CudaSRTPlatform
from sglang.srt.platforms.device_mixin import (
    CpuArchEnum,
    DeviceCapability,
    DeviceMixin,
    PlatformEnum,
)
from sglang.srt.platforms.interface import SRTPlatform
from sglang.test.ci.ci_register import register_cpu_ci
from sglang.test.test_utils import CustomTestCase
```
**EN:** This block imports the modules needed by the rest of the file, including `unittest.mock`, `torch`, `sglang.srt.platforms`, `sglang.srt.platforms.cuda`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `unittest.mock`, `torch`, `sglang.srt.platforms`, `sglang.srt.platforms.cuda`。

### Lines 24-24: CI registration and metadata / CI 注册与元数据
```python
register_cpu_ci(est_time=7, suite="base-a-test-cpu")
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through register_cpu_ci.
**CN:** 该代码块通过 register_cpu_ci 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 25-31: supporting source context / 辅助源码上下文
```python


# ---------------------------------------------------------------------------
# Helpers: factory functions to reduce boilerplate
# ---------------------------------------------------------------------------


```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 32-46: function make device mixin / 函数 make device mixin
```python
def _make_device_mixin(enum, name, dtype):
    """Create a concrete DeviceMixin subclass for testing."""

    class M(DeviceMixin):
        _enum = enum
        device_name = name
        device_type = dtype

        def get_device_total_memory(self, device_id=0):
            return 10**9

        def get_current_memory_usage(self, device=None):
            return 5 * 10**8

    return M()
```
**EN:** Create a concrete DeviceMixin subclass for testing. This block implements `_make_device_mixin` and captures one focused piece of the module's behavior.
**CN:** Create a concrete DeviceMixin subclass for testing. 该代码块实现 `_make_device_mixin`，承担模块行为中的一个聚焦逻辑片段。

### Lines 49-49: class _StubPlatform declaration / 类 _StubPlatform 声明
```python
class _StubPlatform(SRTPlatform):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `SRTPlatform`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `SRTPlatform`。

### Lines 50-54: class-level constants and configuration for `_StubPlatform` / 类级常量与配置
```python
    """Concrete SRTPlatform with minimal defaults for testing overrides."""

    _enum = PlatformEnum.CUDA
    device_name = "cuda"
    device_type = "cuda"
```
**EN:** This block defines shared names such as `_enum`, `device_name`, `device_type`. These values centralize configuration that later tests or helpers reuse.
**CN:** 该代码块定义了 `_enum`, `device_name`, `device_type` 等共享名称，用于集中保存后续测试或辅助逻辑会复用的配置。

### Lines 56-57: helper routine get device total memory / 辅助流程 get device total memory
```python
    def get_device_total_memory(self, device_id=0):
        return 10**9
```
**EN:** This helper encapsulates `get_device_total_memory` so the surrounding tests can reuse setup, execution, or validation logic.
**CN:** 该辅助函数封装了 `get_device_total_memory`，以便周围测试复用准备、执行或校验逻辑。

### Lines 59-60: helper routine get current memory usage / 辅助流程 get current memory usage
```python
    def get_current_memory_usage(self, device=None):
        return 5 * 10**8
```
**EN:** This helper encapsulates `get_current_memory_usage` so the surrounding tests can reuse setup, execution, or validation logic.
**CN:** 该辅助函数封装了 `get_current_memory_usage`，以便周围测试复用准备、执行或校验逻辑。

### Lines 62-63: helper routine get default attention backend / 辅助流程 get default attention backend
```python
    def get_default_attention_backend(self):
        return "flashinfer"
```
**EN:** This helper encapsulates `get_default_attention_backend` so the surrounding tests can reuse setup, execution, or validation logic.
**CN:** 该辅助函数封装了 `get_default_attention_backend`，以便周围测试复用准备、执行或校验逻辑。

### Lines 65-66: helper routine get graph runner cls / 辅助流程 get graph runner cls
```python
    def get_graph_runner_cls(self):
        return object
```
**EN:** This helper encapsulates `get_graph_runner_cls` so the surrounding tests can reuse setup, execution, or validation logic.
**CN:** 该辅助函数封装了 `get_graph_runner_cls`，以便周围测试复用准备、执行或校验逻辑。

### Lines 68-69: helper routine get mha kv pool cls / 辅助流程 get mha kv pool cls
```python
    def get_mha_kv_pool_cls(self):
        return object
```
**EN:** This helper encapsulates `get_mha_kv_pool_cls` so the surrounding tests can reuse setup, execution, or validation logic.
**CN:** 该辅助函数封装了 `get_mha_kv_pool_cls`，以便周围测试复用准备、执行或校验逻辑。

### Lines 71-72: helper routine get mla kv pool cls / 辅助流程 get mla kv pool cls
```python
    def get_mla_kv_pool_cls(self):
        return object
```
**EN:** This helper encapsulates `get_mla_kv_pool_cls` so the surrounding tests can reuse setup, execution, or validation logic.
**CN:** 该辅助函数封装了 `get_mla_kv_pool_cls`，以便周围测试复用准备、执行或校验逻辑。

### Lines 74-75: helper routine get nsa kv pool cls / 辅助流程 get nsa kv pool cls
```python
    def get_nsa_kv_pool_cls(self):
        return object
```
**EN:** This helper encapsulates `get_nsa_kv_pool_cls` so the surrounding tests can reuse setup, execution, or validation logic.
**CN:** 该辅助函数封装了 `get_nsa_kv_pool_cls`，以便周围测试复用准备、执行或校验逻辑。

### Lines 77-78: helper routine get paged allocator cls / 辅助流程 get paged allocator cls
```python
    def get_paged_allocator_cls(self):
        return object
```
**EN:** This helper encapsulates `get_paged_allocator_cls` so the surrounding tests can reuse setup, execution, or validation logic.
**CN:** 该辅助函数封装了 `get_paged_allocator_cls`，以便周围测试复用准备、执行或校验逻辑。

### Lines 80-81: helper routine get piecewise backend cls / 辅助流程 get piecewise backend cls
```python
    def get_piecewise_backend_cls(self):
        return object
```
**EN:** This helper encapsulates `get_piecewise_backend_cls` so the surrounding tests can reuse setup, execution, or validation logic.
**CN:** 该辅助函数封装了 `get_piecewise_backend_cls`，以便周围测试复用准备、执行或校验逻辑。

### Lines 84-92: function make platform ep / 函数 make platform ep
```python
def _make_platform_ep(name, load_fn=None):
    """Create a mock entry point for platform plugins."""
    ep = MagicMock()
    ep.name = name
    if load_fn is not None:
        ep.load.return_value = load_fn
    else:
        ep.load.return_value = MagicMock()
    return ep
```
**EN:** Create a mock entry point for platform plugins. This block implements `_make_platform_ep` and captures one focused piece of the module's behavior.
**CN:** Create a mock entry point for platform plugins. 该代码块实现 `_make_platform_ep`，承担模块行为中的一个聚焦逻辑片段。

### Lines 93-99: supporting source context / 辅助源码上下文
```python


# ---------------------------------------------------------------------------
# PlatformEnum & CpuArchEnum
# ---------------------------------------------------------------------------


```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 100-100: class TestPlatformEnum declaration / 类 TestPlatformEnum 声明
```python
class TestPlatformEnum(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 101-101: supporting statements / 辅助语句
```python
    """Tests for PlatformEnum enumeration."""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 103-117: test case all expected values exist / 测试用例 all expected values exist
```python
    def test_all_expected_values_exist(self):
        expected = {
            "CUDA",
            "ROCM",
            "CPU",
            "XPU",
            "MUSA",
            "NPU",
            "TPU",
            "MPS",
            "OOT",
            "UNSPECIFIED",
        }
        actual = {member.name for member in PlatformEnum}
        self.assertEqual(actual, expected)
```
**EN:** This test exercises `test_all_expected_values_exist` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_all_expected_values_exist`。

### Lines 120-120: class TestCpuArchEnum declaration / 类 TestCpuArchEnum 声明
```python
class TestCpuArchEnum(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 121-121: supporting statements / 辅助语句
```python
    """Tests for CpuArchEnum enumeration."""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 123-126: test case all expected values exist / 测试用例 all expected values exist
```python
    def test_all_expected_values_exist(self):
        expected = {"X86", "ARM", "UNSPECIFIED"}
        actual = {member.name for member in CpuArchEnum}
        self.assertEqual(actual, expected)
```
**EN:** This test exercises `test_all_expected_values_exist` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_all_expected_values_exist`。

### Lines 127-133: supporting source context / 辅助源码上下文
```python


# ---------------------------------------------------------------------------
# DeviceCapability
# ---------------------------------------------------------------------------


```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 134-134: class TestDeviceCapability declaration / 类 TestDeviceCapability 声明
```python
class TestDeviceCapability(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 135-135: supporting statements / 辅助语句
```python
    """Tests for DeviceCapability custom logic (formatting, conversion)."""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 137-139: test case as version str / 测试用例 as version str
```python
    def test_as_version_str(self):
        self.assertEqual(DeviceCapability(major=9, minor=0).as_version_str(), "9.0")
        self.assertEqual(DeviceCapability(major=8, minor=9).as_version_str(), "8.9")
```
**EN:** This test exercises `test_as_version_str` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_as_version_str`。

### Lines 141-144: test case to int / 测试用例 to int
```python
    def test_to_int(self):
        self.assertEqual(DeviceCapability(major=9, minor=0).to_int(), 90)
        self.assertEqual(DeviceCapability(major=8, minor=9).to_int(), 89)
        self.assertEqual(DeviceCapability(major=0, minor=0).to_int(), 0)
```
**EN:** This test exercises `test_to_int` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_to_int`。

### Lines 145-151: supporting source context / 辅助源码上下文
```python


# ---------------------------------------------------------------------------
# DeviceMixin
# ---------------------------------------------------------------------------

# Platform identity test data: (enum, name, dtype, true_method)
```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 152-170: module-level constants and configuration / 模块级常量与配置
```python
_PLATFORM_IDENTITY = [
    (PlatformEnum.CUDA, "cuda", "cuda", "is_cuda"),
    (PlatformEnum.ROCM, "rocm", "hip", "is_rocm"),
    (PlatformEnum.CPU, "cpu", "cpu", "is_cpu"),
    (PlatformEnum.XPU, "xpu", "xpu", "is_xpu"),
    (PlatformEnum.MUSA, "musa", "musa", "is_musa"),
    (PlatformEnum.NPU, "npu", "npu", "is_npu"),
    (PlatformEnum.TPU, "tpu", "tpu", "is_tpu"),
    (PlatformEnum.MPS, "mps", "mps", "is_mps"),
]

# is_cuda_alike test data: (enum, name, dtype, expected)
_CUDA_ALIKE = [
    (PlatformEnum.CUDA, "cuda", "cuda", True),
    (PlatformEnum.ROCM, "rocm", "hip", True),
    (PlatformEnum.MUSA, "musa", "musa", True),
    (PlatformEnum.CPU, "cpu", "cpu", False),
    (PlatformEnum.NPU, "npu", "npu", False),
]
```
**EN:** This block defines shared names such as `_PLATFORM_IDENTITY`, `_CUDA_ALIKE`. These values centralize configuration that later tests or helpers reuse.
**CN:** 该代码块定义了 `_PLATFORM_IDENTITY`, `_CUDA_ALIKE` 等共享名称，用于集中保存后续测试或辅助逻辑会复用的配置。

### Lines 173-173: class TestDeviceMixin declaration / 类 TestDeviceMixin 声明
```python
class TestDeviceMixin(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 174-174: supporting statements / 辅助语句
```python
    """Tests for DeviceMixin base class."""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 176-181: test case platform identity methods / 测试用例 platform identity methods
```python
    def test_platform_identity_methods(self):
        """Each platform type returns True for its identity method."""
        for enum_val, name, dtype, method in _PLATFORM_IDENTITY:
            with self.subTest(method=method, enum=enum_val.name):
                mixin = _make_device_mixin(enum_val, name, dtype)
                self.assertTrue(getattr(mixin, method)())
```
**EN:** Each platform type returns True for its identity method. This test exercises `test_platform_identity_methods` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Each platform type returns True for its identity method. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_platform_identity_methods`。

### Lines 183-188: test case is cuda alike / 测试用例 is cuda alike
```python
    def test_is_cuda_alike(self):
        """is_cuda_alike is True for CUDA/ROCM/MUSA, False otherwise."""
        for enum_val, name, dtype, expected in _CUDA_ALIKE:
            with self.subTest(enum=enum_val.name):
                mixin = _make_device_mixin(enum_val, name, dtype)
                self.assertEqual(mixin.is_cuda_alike(), expected)
```
**EN:** is_cuda_alike is True for CUDA/ROCM/MUSA, False otherwise. This test exercises `test_is_cuda_alike` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** is_cuda_alike is True for CUDA/ROCM/MUSA, False otherwise. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_is_cuda_alike`。

### Lines 190-194: test case is out of tree / 测试用例 is out of tree
```python
    def test_is_out_of_tree(self):
        oot = _make_device_mixin(PlatformEnum.OOT, "custom", "custom")
        self.assertTrue(oot.is_out_of_tree())
        cuda = _make_device_mixin(PlatformEnum.CUDA, "cuda", "cuda")
        self.assertFalse(cuda.is_out_of_tree())
```
**EN:** This test exercises `test_is_out_of_tree` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_is_out_of_tree`。

### Lines 196-212: test case get cpu architecture / 测试用例 get cpu architecture
```python
    @patch("platform.machine")
    def test_get_cpu_architecture(self, mock_machine):
        """get_cpu_architecture maps common strings to CpuArchEnum."""
        cases = [
            ("x86_64", CpuArchEnum.X86),
            ("amd64", CpuArchEnum.X86),
            ("i386", CpuArchEnum.X86),
            ("i686", CpuArchEnum.X86),
            ("X86_64", CpuArchEnum.X86),  # case insensitive
            ("arm64", CpuArchEnum.ARM),
            ("aarch64", CpuArchEnum.ARM),
            ("unknown_arch", CpuArchEnum.UNSPECIFIED),
        ]
        for machine_str, expected in cases:
            with self.subTest(machine=machine_str):
                mock_machine.return_value = machine_str
                self.assertEqual(DeviceMixin.get_cpu_architecture(), expected)
```
**EN:** get_cpu_architecture maps common strings to CpuArchEnum. This test exercises `test_get_cpu_architecture` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** get_cpu_architecture maps common strings to CpuArchEnum. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_get_cpu_architecture`。

### Lines 213-219: supporting source context / 辅助源码上下文
```python


# ---------------------------------------------------------------------------
# SRTPlatform
# ---------------------------------------------------------------------------


```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 220-220: class TestSRTPlatform declaration / 类 TestSRTPlatform 声明
```python
class TestSRTPlatform(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 221-221: supporting statements / 辅助语句
```python
    """Tests for SRTPlatform base class and default behaviors."""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 223-226: test case compile backend signature compatibility / 测试用例 compile backend signature compatibility
```python
    def test_compile_backend_signature_compatibility(self):
        """get_compile_backend accepts mode keyword arg without error."""
        base = SRTPlatform()
        self.assertEqual(base.get_compile_backend(mode="npugraph_ex"), "inductor")
```
**EN:** get_compile_backend accepts mode keyword arg without error. This test exercises `test_compile_backend_signature_compatibility` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** get_compile_backend accepts mode keyword arg without error. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_compile_backend_signature_compatibility`。

### Lines 228-232: test case base device identity stays unspecified / 测试用例 base device identity stays unspecified
```python
    def test_base_device_identity_stays_unspecified(self):
        """The abstract SRT base should not claim any concrete in-tree device."""
        base = SRTPlatform()
        self.assertFalse(base.is_cuda())
        self.assertFalse(base.is_cuda_alike())
```
**EN:** The abstract SRT base should not claim any concrete in-tree device. This test exercises `test_base_device_identity_stays_unspecified` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** The abstract SRT base should not claim any concrete in-tree device. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_base_device_identity_stays_unspecified`。

### Lines 235-235: class TestCudaDeviceMixin declaration / 类 TestCudaDeviceMixin 声明
```python
class TestCudaDeviceMixin(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 236-236: supporting statements / 辅助语句
```python
    """Tests for CUDA device operation defaults."""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 238-240: test case default get device returns cuda device / 测试用例 default get device returns cuda device
```python
    def test_default_get_device_returns_cuda_device(self):
        base = CudaSRTPlatform()
        self.assertEqual(base.get_device(2), torch.device("cuda", 2))
```
**EN:** This test exercises `test_default_get_device_returns_cuda_device` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_default_get_device_returns_cuda_device`。

### Lines 242-246: test case cuda platform identity / 测试用例 cuda platform identity
```python
    def test_cuda_platform_identity(self):
        base = CudaSRTPlatform()
        self.assertTrue(base.is_cuda())
        self.assertTrue(base.is_cuda_alike())
        self.assertIsInstance(base, CudaDeviceMixin)
```
**EN:** This test exercises `test_cuda_platform_identity` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_cuda_platform_identity`。

### Lines 248-255: test case default get device total memory uses cuda / 测试用例 default get device total memory uses cuda
```python
    @patch("torch.cuda.get_device_properties")
    def test_default_get_device_total_memory_uses_cuda(
        self, mock_get_device_properties
    ):
        mock_get_device_properties.return_value.total_memory = 123
        base = CudaSRTPlatform()
        self.assertEqual(base.get_device_total_memory(1), 123)
        mock_get_device_properties.assert_called_once_with(1)
```
**EN:** This test exercises `test_default_get_device_total_memory_uses_cuda` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_default_get_device_total_memory_uses_cuda`。

### Lines 257-264: test case default get current memory usage uses cuda / 测试用例 default get current memory usage uses cuda
```python
    @patch("torch.cuda.max_memory_allocated", return_value=456)
    def test_default_get_current_memory_usage_uses_cuda(
        self, mock_max_memory_allocated
    ):
        base = CudaSRTPlatform()
        device = torch.device("cuda", 1)
        self.assertEqual(base.get_current_memory_usage(device), 456.0)
        mock_max_memory_allocated.assert_called_once_with(device)
```
**EN:** This test exercises `test_default_get_current_memory_usage_uses_cuda` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_default_get_current_memory_usage_uses_cuda`。

### Lines 266-271: test case default set device uses cuda / 测试用例 default set device uses cuda
```python
    @patch("torch.cuda.set_device")
    def test_default_set_device_uses_cuda(self, mock_set_device):
        base = CudaSRTPlatform()
        device = torch.device("cuda", 1)
        base.set_device(device)
        mock_set_device.assert_called_once_with(device)
```
**EN:** This test exercises `test_default_set_device_uses_cuda` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_default_set_device_uses_cuda`。

### Lines 273-277: test case default get device name uses cuda / 测试用例 default get device name uses cuda
```python
    @patch("torch.cuda.get_device_name", return_value="NVIDIA H100")
    def test_default_get_device_name_uses_cuda(self, mock_get_device_name):
        base = CudaSRTPlatform()
        self.assertEqual(base.get_device_name(1), "NVIDIA H100")
        mock_get_device_name.assert_called_once_with(1)
```
**EN:** This test exercises `test_default_get_device_name_uses_cuda` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_default_get_device_name_uses_cuda`。

### Lines 279-284: test case default get device uuid uses cuda / 测试用例 default get device uuid uses cuda
```python
    @patch("torch.cuda.get_device_properties")
    def test_default_get_device_uuid_uses_cuda(self, mock_get_device_properties):
        mock_get_device_properties.return_value.uuid = "1234"
        base = CudaSRTPlatform()
        self.assertEqual(base.get_device_uuid(1), "1234")
        mock_get_device_properties.assert_called_once_with(1)
```
**EN:** This test exercises `test_default_get_device_uuid_uses_cuda` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_default_get_device_uuid_uses_cuda`。

### Lines 286-290: test case default get device capability uses cuda / 测试用例 default get device capability uses cuda
```python
    @patch("torch.cuda.get_device_capability", return_value=(9, 0))
    def test_default_get_device_capability_uses_cuda(self, mock_get_device_capability):
        base = CudaSRTPlatform()
        self.assertEqual(base.get_device_capability(1), DeviceCapability(9, 0))
        mock_get_device_capability.assert_called_once_with(1)
```
**EN:** This test exercises `test_default_get_device_capability_uses_cuda` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_default_get_device_capability_uses_cuda`。

### Lines 292-296: test case default empty cache uses cuda / 测试用例 default empty cache uses cuda
```python
    @patch("torch.cuda.empty_cache")
    def test_default_empty_cache_uses_cuda(self, mock_empty_cache):
        base = CudaSRTPlatform()
        base.empty_cache()
        mock_empty_cache.assert_called_once_with()
```
**EN:** This test exercises `test_default_empty_cache_uses_cuda` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_default_empty_cache_uses_cuda`。

### Lines 298-302: test case default synchronize uses cuda / 测试用例 default synchronize uses cuda
```python
    @patch("torch.cuda.synchronize")
    def test_default_synchronize_uses_cuda(self, mock_synchronize):
        base = CudaSRTPlatform()
        base.synchronize()
        mock_synchronize.assert_called_once_with()
```
**EN:** This test exercises `test_default_synchronize_uses_cuda` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_default_synchronize_uses_cuda`。

### Lines 304-308: test case default get available memory uses cuda / 测试用例 default get available memory uses cuda
```python
    @patch("torch.cuda.mem_get_info", return_value=(123, 456), create=True)
    def test_default_get_available_memory_uses_cuda(self, mock_mem_get_info):
        base = CudaSRTPlatform()
        self.assertEqual(base.get_available_memory(1), (123, 456))
        mock_mem_get_info.assert_called_once_with(1)
```
**EN:** This test exercises `test_default_get_available_memory_uses_cuda` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_default_get_available_memory_uses_cuda`。

### Lines 310-312: test case default distributed backend is nccl / 测试用例 default distributed backend is nccl
```python
    def test_default_distributed_backend_is_nccl(self):
        base = CudaSRTPlatform()
        self.assertEqual(base.get_torch_distributed_backend_str(), "nccl")
```
**EN:** This test exercises `test_default_distributed_backend_is_nccl` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_default_distributed_backend_is_nccl`。

### Lines 314-325: test case default seed everything seeds cuda / 测试用例 default seed everything seeds cuda
```python
    @patch("torch.cuda.manual_seed_all")
    @patch("torch.manual_seed")
    @patch("sglang.srt.platforms.device_mixin.np.random.seed")
    @patch("sglang.srt.platforms.device_mixin.random.seed")
    def test_default_seed_everything_seeds_cuda(
        self, mock_random_seed, mock_np_seed, mock_torch_seed, mock_cuda_seed
    ):
        CudaSRTPlatform.seed_everything(123)
        mock_random_seed.assert_called_once_with(123)
        mock_np_seed.assert_called_once_with(123)
        mock_torch_seed.assert_called_once_with(123)
        mock_cuda_seed.assert_called_once_with(123)
```
**EN:** This test exercises `test_default_seed_everything_seeds_cuda` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_default_seed_everything_seeds_cuda`。

### Lines 327-331: test case cuda srt platform capabilities / 测试用例 cuda srt platform capabilities
```python
    def test_cuda_srt_platform_capabilities(self):
        base = CudaSRTPlatform()
        self.assertTrue(base.supports_fp8())
        self.assertTrue(base.support_cuda_graph())
        self.assertTrue(base.support_piecewise_cuda_graph())
```
**EN:** This test exercises `test_cuda_srt_platform_capabilities` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_cuda_srt_platform_capabilities`。

### Lines 334-334: class TestSRTPlatformOverrides declaration / 类 TestSRTPlatformOverrides 声明
```python
class TestSRTPlatformOverrides(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 335-335: supporting statements / 辅助语句
```python
    """Tests for SRTPlatform method overrides via plugins."""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 337-346: test case custom get dispatch key name / 测试用例 custom get dispatch key name
```python
    def test_custom_get_dispatch_key_name(self):
        class P(_StubPlatform):
            _enum = PlatformEnum.NPU
            device_name = "npu"
            device_type = "npu"

            def get_dispatch_key_name(self):
                return "npu"

        self.assertEqual(P().get_dispatch_key_name(), "npu")
```
**EN:** This test exercises `test_custom_get_dispatch_key_name` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_custom_get_dispatch_key_name`。

### Lines 348-357: test case custom get compile backend / 测试用例 custom get compile backend
```python
    def test_custom_get_compile_backend(self):
        class P(_StubPlatform):
            _enum = PlatformEnum.NPU
            device_name = "npu"
            device_type = "npu"

            def get_compile_backend(self, mode=None):
                return "inductor"

        self.assertEqual(P().get_compile_backend(mode="npugraph_ex"), "inductor")
```
**EN:** This test exercises `test_custom_get_compile_backend` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_custom_get_compile_backend`。

### Lines 358-364: supporting source context / 辅助源码上下文
```python


# ---------------------------------------------------------------------------
# Platform Discovery: _resolve_platform
# ---------------------------------------------------------------------------


```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 365-365: class TestResolvePlatformWithEnv declaration / 类 TestResolvePlatformWithEnv 声明
```python
class TestResolvePlatformWithEnv(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 366-366: supporting statements / 辅助语句
```python
    """Tests for _resolve_platform when SGLANG_PLATFORM is set."""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 368-380: test case selected plugin activates / 测试用例 selected plugin activates
```python
    @patch("sglang.srt.platforms.entry_points")
    @patch("sglang.srt.platforms.envs")
    def test_selected_plugin_activates(self, mock_envs, mock_ep):
        """When SGLANG_PLATFORM matches an entry point, it activates that plugin."""
        mock_envs.SGLANG_PLATFORM.get.return_value = "my_hardware"
        plugin_fn = MagicMock(return_value="pkg.Mod:MyPlatform")
        mock_ep.return_value = [_make_platform_ep("my_hardware", plugin_fn)]
        with patch("sglang.srt.platforms._load_platform_class") as mock_load:
            mock_instance = MagicMock()
            mock_load.return_value = MagicMock(return_value=mock_instance)
            result = _resolve_platform()
            mock_load.assert_called_once_with("pkg.Mod:MyPlatform")
            self.assertEqual(result, mock_instance)
```
**EN:** When SGLANG_PLATFORM matches an entry point, it activates that plugin. This test exercises `test_selected_plugin_activates` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** When SGLANG_PLATFORM matches an entry point, it activates that plugin. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_selected_plugin_activates`。

### Lines 382-389: test case selected plugin not found / 测试用例 selected plugin not found
```python
    @patch("sglang.srt.platforms.entry_points")
    @patch("sglang.srt.platforms.envs")
    def test_selected_plugin_not_found(self, mock_envs, mock_ep):
        """When SGLANG_PLATFORM names a nonexistent plugin, raise RuntimeError."""
        mock_envs.SGLANG_PLATFORM.get.return_value = "nonexistent"
        mock_ep.return_value = []
        with self.assertRaises(RuntimeError):
            _resolve_platform()
```
**EN:** When SGLANG_PLATFORM names a nonexistent plugin, raise RuntimeError. This test exercises `test_selected_plugin_not_found` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** When SGLANG_PLATFORM names a nonexistent plugin, raise RuntimeError. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_selected_plugin_not_found`。

### Lines 391-399: test case selected plugin hardware unavailable / 测试用例 selected plugin hardware unavailable
```python
    @patch("sglang.srt.platforms.entry_points")
    @patch("sglang.srt.platforms.envs")
    def test_selected_plugin_hardware_unavailable(self, mock_envs, mock_ep):
        """When activate() returns None, hardware is not available."""
        mock_envs.SGLANG_PLATFORM.get.return_value = "my_hardware"
        plugin_fn = MagicMock(return_value=None)
        mock_ep.return_value = [_make_platform_ep("my_hardware", plugin_fn)]
        with self.assertRaises(RuntimeError):
            _resolve_platform()
```
**EN:** When activate() returns None, hardware is not available. This test exercises `test_selected_plugin_hardware_unavailable` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** When activate() returns None, hardware is not available. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_selected_plugin_hardware_unavailable`。

### Lines 401-409: test case selected plugin load exception / 测试用例 selected plugin load exception
```python
    @patch("sglang.srt.platforms.entry_points")
    @patch("sglang.srt.platforms.envs")
    def test_selected_plugin_load_exception(self, mock_envs, mock_ep):
        """When ep.load() or activate() throws, exception is re-raised."""
        mock_envs.SGLANG_PLATFORM.get.return_value = "my_hardware"
        plugin_fn = MagicMock(side_effect=ImportError("missing dep"))
        mock_ep.return_value = [_make_platform_ep("my_hardware", plugin_fn)]
        with self.assertRaises(ImportError):
            _resolve_platform()
```
**EN:** When ep.load() or activate() throws, exception is re-raised. This test exercises `test_selected_plugin_load_exception` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** When ep.load() or activate() throws, exception is re-raised. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_selected_plugin_load_exception`。

### Lines 411-425: test case other plugins not loaded / 测试用例 other plugins not loaded
```python
    @patch("sglang.srt.platforms.entry_points")
    @patch("sglang.srt.platforms.envs")
    def test_other_plugins_not_loaded(self, mock_envs, mock_ep):
        """When SGLANG_PLATFORM is set, other plugins are not imported."""
        mock_envs.SGLANG_PLATFORM.get.return_value = "target_hw"
        target_fn = MagicMock(return_value="pkg.Mod:TargetPlatform")
        other_ep = _make_platform_ep("other_hw")  # default load returns MagicMock
        target_ep = _make_platform_ep("target_hw", target_fn)
        mock_ep.return_value = [other_ep, target_ep]
        with patch("sglang.srt.platforms._load_platform_class") as mock_load:
            mock_load.return_value = MagicMock(return_value=MagicMock())
            _resolve_platform()
            # Only the target entry point should be loaded
            target_ep.load.assert_called_once()
            other_ep.load.assert_not_called()
```
**EN:** When SGLANG_PLATFORM is set, other plugins are not imported. This test exercises `test_other_plugins_not_loaded` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** When SGLANG_PLATFORM is set, other plugins are not imported. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_other_plugins_not_loaded`。

### Lines 428-428: class TestResolvePlatformAutoDiscover declaration / 类 TestResolvePlatformAutoDiscover 声明
```python
class TestResolvePlatformAutoDiscover(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 429-429: supporting statements / 辅助语句
```python
    """Tests for _resolve_platform auto-discovery when SGLANG_PLATFORM is not set."""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 431-439: test case is cuda available excludes rocm / 测试用例 is cuda available excludes rocm
```python
    @patch("sglang.srt.platforms.torch")
    def test_is_cuda_available_excludes_rocm(self, mock_torch):
        """ROCm exposes torch.cuda, but should not use the CUDA platform identity."""
        mock_torch.cuda.is_available.return_value = True
        mock_torch.version.hip = "6.0"

        import sglang.srt.platforms as plat_mod

        self.assertFalse(plat_mod._is_cuda_available())
```
**EN:** ROCm exposes torch.cuda, but should not use the CUDA platform identity. This test exercises `test_is_cuda_available_excludes_rocm` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** ROCm exposes torch.cuda, but should not use the CUDA platform identity. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_is_cuda_available_excludes_rocm`。

### Lines 441-453: test case single plugin activates / 测试用例 single plugin activates
```python
    @patch("sglang.srt.platforms.load_plugins_by_group")
    @patch("sglang.srt.platforms.envs")
    def test_single_plugin_activates(self, mock_envs, mock_load):
        """When exactly one plugin activates, return its platform instance."""
        mock_envs.SGLANG_PLATFORM.get.return_value = ""
        plugin_fn = MagicMock(return_value="pkg.Mod:MyPlatform")
        mock_load.return_value = {"my_hw": (plugin_fn, "my-hw-dist")}
        with patch("sglang.srt.platforms._load_platform_class") as mock_resolve:
            mock_instance = MagicMock()
            mock_resolve.return_value = MagicMock(return_value=mock_instance)
            result = _resolve_platform()
            mock_resolve.assert_called_once_with("pkg.Mod:MyPlatform")
            self.assertEqual(result, mock_instance)
```
**EN:** When exactly one plugin activates, return its platform instance. This test exercises `test_single_plugin_activates` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** When exactly one plugin activates, return its platform instance. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_single_plugin_activates`。

### Lines 455-466: test case no plugin activates cuda fallback / 测试用例 no plugin activates cuda fallback
```python
    @patch("sglang.srt.platforms.load_plugins_by_group")
    @patch("sglang.srt.platforms._is_cuda_available")
    @patch("sglang.srt.platforms.envs")
    def test_no_plugin_activates_cuda_fallback(
        self, mock_envs, mock_is_cuda_available, mock_load
    ):
        """When CUDA is available and no plugin activates, return CUDA defaults."""
        mock_envs.SGLANG_PLATFORM.get.return_value = ""
        mock_is_cuda_available.return_value = True
        mock_load.return_value = {}
        result = _resolve_platform()
        self.assertIsInstance(result, CudaSRTPlatform)
```
**EN:** When CUDA is available and no plugin activates, return CUDA defaults. This test exercises `test_no_plugin_activates_cuda_fallback` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** When CUDA is available and no plugin activates, return CUDA defaults. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_no_plugin_activates_cuda_fallback`。

### Lines 468-480: test case no plugin no cuda activates base fallback / 测试用例 no plugin no cuda activates base fallback
```python
    @patch("sglang.srt.platforms.load_plugins_by_group")
    @patch("sglang.srt.platforms._is_cuda_available")
    @patch("sglang.srt.platforms.envs")
    def test_no_plugin_no_cuda_activates_base_fallback(
        self, mock_envs, mock_is_cuda_available, mock_load
    ):
        """When no plugin or CUDA is available, return the abstract base platform."""
        mock_envs.SGLANG_PLATFORM.get.return_value = ""
        mock_is_cuda_available.return_value = False
        mock_load.return_value = {}
        result = _resolve_platform()
        self.assertIsInstance(result, SRTPlatform)
        self.assertNotIsInstance(result, CudaSRTPlatform)
```
**EN:** When no plugin or CUDA is available, return the abstract base platform. This test exercises `test_no_plugin_no_cuda_activates_base_fallback` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** When no plugin or CUDA is available, return the abstract base platform. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_no_plugin_no_cuda_activates_base_fallback`。

### Lines 482-497: test case no plugin rocm does not activate cuda fallback / 测试用例 no plugin rocm does not activate cuda fallback
```python
    @patch("sglang.srt.platforms.load_plugins_by_group")
    @patch("sglang.srt.platforms.torch")
    @patch("sglang.srt.platforms.envs")
    def test_no_plugin_rocm_does_not_activate_cuda_fallback(
        self, mock_envs, mock_torch, mock_load
    ):
        """ROCm exposes torch.cuda but must not use the CUDA fallback platform."""
        mock_envs.SGLANG_PLATFORM.get.return_value = ""
        mock_torch.cuda.is_available.return_value = True
        mock_torch.version.hip = "6.0"
        mock_load.return_value = {}

        result = _resolve_platform()

        self.assertIsInstance(result, SRTPlatform)
        self.assertNotIsInstance(result, CudaSRTPlatform)
```
**EN:** ROCm exposes torch.cuda but must not use the CUDA fallback platform. This test exercises `test_no_plugin_rocm_does_not_activate_cuda_fallback` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** ROCm exposes torch.cuda but must not use the CUDA fallback platform. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_no_plugin_rocm_does_not_activate_cuda_fallback`。

### Lines 499-508: test case multiple plugins activate raises / 测试用例 multiple plugins activate raises
```python
    @patch("sglang.srt.platforms.load_plugins_by_group")
    @patch("sglang.srt.platforms.envs")
    def test_multiple_plugins_activate_raises(self, mock_envs, mock_load):
        """When multiple plugins activate, raise RuntimeError."""
        mock_envs.SGLANG_PLATFORM.get.return_value = ""
        fn1 = MagicMock(return_value="pkg1.Mod:Platform1")
        fn2 = MagicMock(return_value="pkg2.Mod:Platform2")
        mock_load.return_value = {"hw1": (fn1, "hw1-dist"), "hw2": (fn2, "hw2-dist")}
        with self.assertRaises(RuntimeError):
            _resolve_platform()
```
**EN:** When multiple plugins activate, raise RuntimeError. This test exercises `test_multiple_plugins_activate_raises` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** When multiple plugins activate, raise RuntimeError. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_multiple_plugins_activate_raises`。

### Lines 510-526: test case plugin exception does not crash / 测试用例 plugin exception does not crash
```python
    @patch("sglang.srt.platforms.load_plugins_by_group")
    @patch("sglang.srt.platforms.envs")
    def test_plugin_exception_does_not_crash(self, mock_envs, mock_load):
        """When a plugin's activate() throws, it is skipped, others continue."""
        mock_envs.SGLANG_PLATFORM.get.return_value = ""
        bad_fn = MagicMock(side_effect=RuntimeError("broken"))
        good_fn = MagicMock(return_value="pkg.Mod:GoodPlatform")
        mock_load.return_value = {
            "bad": (bad_fn, "bad-dist"),
            "good": (good_fn, "good-dist"),
        }
        with patch("sglang.srt.platforms._load_platform_class") as mock_resolve:
            mock_instance = MagicMock()
            mock_resolve.return_value = MagicMock(return_value=mock_instance)
            result = _resolve_platform()
            mock_resolve.assert_called_once_with("pkg.Mod:GoodPlatform")
            self.assertEqual(result, mock_instance)
```
**EN:** When a plugin's activate() throws, it is skipped, others continue. This test exercises `test_plugin_exception_does_not_crash` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** When a plugin's activate() throws, it is skipped, others continue. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_plugin_exception_does_not_crash`。

### Lines 528-544: test case plugin returns none is skipped / 测试用例 plugin returns none is skipped
```python
    @patch("sglang.srt.platforms.load_plugins_by_group")
    @patch("sglang.srt.platforms.envs")
    def test_plugin_returns_none_is_skipped(self, mock_envs, mock_load):
        """When a plugin's activate() returns None, it is skipped (hardware unavailable)."""
        mock_envs.SGLANG_PLATFORM.get.return_value = ""
        none_fn = MagicMock(return_value=None)
        good_fn = MagicMock(return_value="pkg.Mod:GoodPlatform")
        mock_load.return_value = {
            "unavailable": (none_fn, "unavail-dist"),
            "good": (good_fn, "good-dist"),
        }
        with patch("sglang.srt.platforms._load_platform_class") as mock_resolve:
            mock_instance = MagicMock()
            mock_resolve.return_value = MagicMock(return_value=mock_instance)
            result = _resolve_platform()
            # Only the good plugin activated; single activation succeeds
            mock_resolve.assert_called_once_with("pkg.Mod:GoodPlatform")
```
**EN:** When a plugin's activate() returns None, it is skipped (hardware unavailable). This test exercises `test_plugin_returns_none_is_skipped` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** When a plugin's activate() returns None, it is skipped (hardware unavailable). 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_plugin_returns_none_is_skipped`。

### Lines 545-551: supporting source context / 辅助源码上下文
```python


# ---------------------------------------------------------------------------
# Platform Discovery: _load_platform_class
# ---------------------------------------------------------------------------


```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 552-552: class TestLoadPlatformClass declaration / 类 TestLoadPlatformClass 声明
```python
class TestLoadPlatformClass(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 553-553: supporting statements / 辅助语句
```python
    """Tests for _load_platform_class qualname resolution."""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 555-560: test case valid subclass / 测试用例 valid subclass
```python
    @patch("sglang.srt.platforms.pkgutil.resolve_name")
    def test_valid_subclass(self, mock_resolve):
        """Valid SRTPlatform subclass resolves successfully."""
        mock_resolve.return_value = type("MyPlatform", (SRTPlatform,), {})
        result = _load_platform_class("pkg.Mod:MyPlatform")
        self.assertTrue(issubclass(result, SRTPlatform))
```
**EN:** Valid SRTPlatform subclass resolves successfully. This test exercises `test_valid_subclass` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Valid SRTPlatform subclass resolves successfully. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_valid_subclass`。

### Lines 562-567: test case non subclass raises type error / 测试用例 non subclass raises type error
```python
    @patch("sglang.srt.platforms.pkgutil.resolve_name")
    def test_non_subclass_raises_type_error(self, mock_resolve):
        """Non-SRTPlatform class raises TypeError."""
        mock_resolve.return_value = str
        with self.assertRaises(TypeError):
            _load_platform_class("builtins.str")
```
**EN:** Non-SRTPlatform class raises TypeError. This test exercises `test_non_subclass_raises_type_error` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Non-SRTPlatform class raises TypeError. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_non_subclass_raises_type_error`。

### Lines 569-574: test case non type raises type error / 测试用例 non type raises type error
```python
    @patch("sglang.srt.platforms.pkgutil.resolve_name")
    def test_non_type_raises_type_error(self, mock_resolve):
        """Non-type object raises TypeError."""
        mock_resolve.return_value = "not a class"
        with self.assertRaises(TypeError):
            _load_platform_class("something")
```
**EN:** Non-type object raises TypeError. This test exercises `test_non_type_raises_type_error` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Non-type object raises TypeError. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_non_type_raises_type_error`。

### Lines 575-581: supporting source context / 辅助源码上下文
```python


# ---------------------------------------------------------------------------
# Platform Discovery: current_platform lazy init
# ---------------------------------------------------------------------------


```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 582-582: class TestCurrentPlatformLazyInit declaration / 类 TestCurrentPlatformLazyInit 声明
```python
class TestCurrentPlatformLazyInit(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 583-583: supporting statements / 辅助语句
```python
    """Tests for current_platform lazy initialization via module __getattr__."""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 585-590: setUp setup routine / setUp 初始化流程
```python
    def setUp(self):
        """Reset module-level cache before each test."""
        import sglang.srt.platforms as plat_mod

        self._saved_platform = plat_mod._current_platform
        plat_mod._current_platform = None
```
**EN:** Reset module-level cache before each test. This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** Reset module-level cache before each test. 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 592-596: tearDown cleanup routine / tearDown 清理流程
```python
    def tearDown(self):
        """Restore original _current_platform after each test."""
        import sglang.srt.platforms as plat_mod

        plat_mod._current_platform = self._saved_platform
```
**EN:** Restore original _current_platform after each test. This routine releases resources and restores state after the related tests finish.
**CN:** Restore original _current_platform after each test. 该流程会在相关测试结束后释放资源并恢复状态。

### Lines 598-607: test case first access triggers resolve / 测试用例 first access triggers resolve
```python
    @patch("sglang.srt.platforms._resolve_platform")
    def test_first_access_triggers_resolve(self, mock_resolve):
        """First access to current_platform calls _resolve_platform."""
        mock_instance = MagicMock(spec=SRTPlatform)
        mock_resolve.return_value = mock_instance
        import sglang.srt.platforms as plat_mod

        result = plat_mod.current_platform
        mock_resolve.assert_called_once()
        self.assertEqual(result, mock_instance)
```
**EN:** First access to current_platform calls _resolve_platform. This test exercises `test_first_access_triggers_resolve` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** First access to current_platform calls _resolve_platform. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_first_access_triggers_resolve`。

### Lines 609-618: test case subsequent access uses cache / 测试用例 subsequent access uses cache
```python
    @patch("sglang.srt.platforms._resolve_platform")
    def test_subsequent_access_uses_cache(self, mock_resolve):
        """Subsequent accesses return cached instance without re-resolving."""
        mock_instance = MagicMock(spec=SRTPlatform)
        mock_resolve.return_value = mock_instance
        import sglang.srt.platforms as plat_mod

        _ = plat_mod.current_platform
        _ = plat_mod.current_platform
        mock_resolve.assert_called_once()
```
**EN:** Subsequent accesses return cached instance without re-resolving. This test exercises `test_subsequent_access_uses_cache` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Subsequent accesses return cached instance without re-resolving. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_subsequent_access_uses_cache`。

### Lines 620-625: test case other attribute raises error / 测试用例 other attribute raises error
```python
    def test_other_attribute_raises_error(self):
        """Accessing non-existent module attribute raises AttributeError."""
        import sglang.srt.platforms as plat_mod

        with self.assertRaises(AttributeError):
            _ = plat_mod.nonexistent_attribute
```
**EN:** Accessing non-existent module attribute raises AttributeError. This test exercises `test_other_attribute_raises_error` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Accessing non-existent module attribute raises AttributeError. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_other_attribute_raises_error`。

### Lines 628-631: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    import unittest

    unittest.main()
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `_make_device_mixin`: Create a concrete DeviceMixin subclass for testing. / 该代码块实现 `_make_device_mixin`，承担模块行为中的一个聚焦逻辑片段。
- `_StubPlatform`: Concrete SRTPlatform with minimal defaults for testing overrides. / 用于组织相关测试、夹具或辅助方法。
- `_make_platform_ep`: Create a mock entry point for platform plugins. / 该代码块实现 `_make_platform_ep`，承担模块行为中的一个聚焦逻辑片段。
- `TestPlatformEnum`: Tests for PlatformEnum enumeration. / 用于组织相关测试、夹具或辅助方法。
- `TestCpuArchEnum`: Tests for CpuArchEnum enumeration. / 用于组织相关测试、夹具或辅助方法。
- `TestDeviceCapability`: Tests for DeviceCapability custom logic (formatting, conversion). / 用于组织相关测试、夹具或辅助方法。
- `TestDeviceMixin`: Tests for DeviceMixin base class. / 用于组织相关测试、夹具或辅助方法。
- `TestSRTPlatform`: Tests for SRTPlatform base class and default behaviors. / 用于组织相关测试、夹具或辅助方法。
- `_StubPlatform.get_device_total_memory`: This helper encapsulates `get_device_total_memory` so the surrounding tests can reuse setup, execution, or validation logic. / 该辅助函数封装了 `get_device_total_memory`，以便周围测试复用准备、执行或校验逻辑。
- `_StubPlatform.get_current_memory_usage`: This helper encapsulates `get_current_memory_usage` so the surrounding tests can reuse setup, execution, or validation logic. / 该辅助函数封装了 `get_current_memory_usage`，以便周围测试复用准备、执行或校验逻辑。
- `_StubPlatform.get_default_attention_backend`: This helper encapsulates `get_default_attention_backend` so the surrounding tests can reuse setup, execution, or validation logic. / 该辅助函数封装了 `get_default_attention_backend`，以便周围测试复用准备、执行或校验逻辑。
- `_StubPlatform.get_graph_runner_cls`: This helper encapsulates `get_graph_runner_cls` so the surrounding tests can reuse setup, execution, or validation logic. / 该辅助函数封装了 `get_graph_runner_cls`，以便周围测试复用准备、执行或校验逻辑。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `unittest.mock`
- **Third-party modules / 第三方模块**: `torch`
- **Internal modules / 内部模块**: `sglang.srt.platforms`, `sglang.srt.platforms.cuda`, `sglang.srt.platforms.device_mixin`, `sglang.srt.platforms.interface`, `sglang.test.ci.ci_register`, `sglang.test.test_utils`

- **Total lines / 总行数**: 631
