# test_fp4_kv_cache_quant_method.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/unit/layers/quantization/test_fp4_kv_cache_quant_method.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates fp4 kv cache quant method behavior in SGLang's unit / layers / quantization area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 单元 / 层 / quantization 领域中与 fp4 kv cache quant method 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-1: supporting statements / 辅助语句
```python
"""Unit tests for FP4 KV cache quantization strategy pattern — no server, no model loading."""
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

### Lines 7-11: module imports and dependencies / 模块导入与依赖
```python
import unittest

import torch

from sglang.test.test_utils import CustomTestCase
```
**EN:** This block imports the modules needed by the rest of the file, including `unittest`, `torch`, `sglang.test.test_utils`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `unittest`, `torch`, `sglang.test.test_utils`。

### Lines 14-20: function skip if no blackwell nvfp4 / 函数 skip if no blackwell nvfp4
```python
def skip_if_no_blackwell_nvfp4(func):
    """Skip test if Blackwell NVFP4 is not available."""
    from sglang.srt.utils import is_blackwell

    return unittest.skipUnless(
        is_blackwell(), "Blackwell (SM100/SM120) with CUDA >= 12.8 is required"
    )(func)
```
**EN:** Skip test if Blackwell NVFP4 is not available. This block implements `skip_if_no_blackwell_nvfp4` and captures one focused piece of the module's behavior.
**CN:** Skip test if Blackwell NVFP4 is not available. 该代码块实现 `skip_if_no_blackwell_nvfp4`，承担模块行为中的一个聚焦逻辑片段。

### Lines 23-23: class TestKVCacheQuantRegistry declaration / 类 TestKVCacheQuantRegistry 声明
```python
class TestKVCacheQuantRegistry(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 24-24: supporting statements / 辅助语句
```python
    """Test the registry and factory function."""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 26-32: test case registry contains nvfp4 and mxfp4 / 测试用例 registry contains nvfp4 and mxfp4
```python
    def test_registry_contains_nvfp4_and_mxfp4(self):
        from sglang.srt.layers.quantization.fp4_kv_cache_quant_method import (
            FP4_KV_CACHE_QUANT_REGISTRY,
        )

        self.assertIn("nvfp4", FP4_KV_CACHE_QUANT_REGISTRY)
        self.assertIn("blockfp4", FP4_KV_CACHE_QUANT_REGISTRY)
```
**EN:** This test exercises `test_registry_contains_nvfp4_and_mxfp4` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_registry_contains_nvfp4_and_mxfp4`。

### Lines 34-44: test case factory nvfp4 / 测试用例 factory nvfp4
```python
    def test_factory_nvfp4(self):
        from sglang.srt.layers.quantization.fp4_kv_cache_quant_method import (
            NVFP4KVMethod,
            get_fp4_kv_cache_quant_method,
        )

        method = get_fp4_kv_cache_quant_method(
            "nvfp4", num_layers=4, device="cpu", sm_version=120
        )
        self.assertIsInstance(method, NVFP4KVMethod)
        self.assertEqual(method.name, "nvfp4")
```
**EN:** This test exercises `test_factory_nvfp4` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_factory_nvfp4`。

### Lines 46-54: test case factory mxfp4 / 测试用例 factory mxfp4
```python
    def test_factory_mxfp4(self):
        from sglang.srt.layers.quantization.fp4_kv_cache_quant_method import (
            BlockFP4KVMethod,
            get_fp4_kv_cache_quant_method,
        )

        method = get_fp4_kv_cache_quant_method("blockfp4")
        self.assertIsInstance(method, BlockFP4KVMethod)
        self.assertEqual(method.name, "blockfp4")
```
**EN:** This test exercises `test_factory_mxfp4` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_factory_mxfp4`。

### Lines 56-62: test case factory unknown raises / 测试用例 factory unknown raises
```python
    def test_factory_unknown_raises(self):
        from sglang.srt.layers.quantization.fp4_kv_cache_quant_method import (
            get_fp4_kv_cache_quant_method,
        )

        with self.assertRaises(ValueError):
            get_fp4_kv_cache_quant_method("unknown_method")
```
**EN:** This test exercises `test_factory_unknown_raises` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_factory_unknown_raises`。

### Lines 65-65: class TestNVFP4KVMethod declaration / 类 TestNVFP4KVMethod 声明
```python
class TestNVFP4KVMethod(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 66-66: supporting statements / 辅助语句
```python
    """Test NVFP4KVMethod buffer creation and properties."""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 68-77: test case properties / 测试用例 properties
```python
    def test_properties(self):
        from sglang.srt.layers.quantization.fp4_kv_cache_quant_method import (
            NVFP4KVMethod,
        )

        m = NVFP4KVMethod(num_layers=4, device="cpu", sm_version=120)
        self.assertEqual(m.name, "nvfp4")
        self.assertEqual(m.SCALE_BLOCK_SIZE, 16)
        self.assertTrue(m.needs_dequant_workspace())
        self.assertTrue(m.needs_global_scale())
```
**EN:** This test exercises `test_properties` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_properties`。

### Lines 79-100: test case create buffers shapes / 测试用例 create buffers shapes
```python
    def test_create_buffers_shapes(self):
        from sglang.srt.layers.quantization.fp4_kv_cache_quant_method import (
            NVFP4KVMethod,
        )

        m = NVFP4KVMethod(num_layers=4, device="cpu", sm_version=120)
        size, heads, dim, layers = 64, 8, 128, 4
        bufs = m.create_buffers(size, heads, dim, layers, "cpu")

        self.assertEqual(len(bufs["k_buffer"]), layers)
        self.assertEqual(len(bufs["v_buffer"]), layers)
        self.assertEqual(len(bufs["k_scale_buffer"]), layers)
        self.assertEqual(len(bufs["v_scale_buffer"]), layers)

        # FP4 packed: (size, heads, dim//2)
        self.assertEqual(bufs["k_buffer"][0].shape, (size, heads, dim // 2))
        # Block scales: (size, heads, dim//16)
        self.assertEqual(bufs["k_scale_buffer"][0].shape, (size, heads, dim // 16))
        # Dequant workspace: (size, heads, dim), FP8
        self.assertEqual(bufs["dq_k_buffer"].shape, (size, heads, dim))
        self.assertEqual(bufs["dq_k_buffer"].dtype, torch.float8_e4m3fn)
        self.assertEqual(bufs["store_dtype"], torch.uint8)
```
**EN:** This test exercises `test_create_buffers_shapes` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_create_buffers_shapes`。

### Lines 102-110: test case compute cell size / 测试用例 compute cell size
```python
    def test_compute_cell_size(self):
        from sglang.srt.layers.quantization.fp4_kv_cache_quant_method import (
            NVFP4KVMethod,
        )

        m = NVFP4KVMethod(num_layers=4, device="cpu")
        cell = m.compute_cell_size(head_num=8, head_dim=128, num_layers=4, kv_size=1)
        # FP4: 8*64*4*2 = 4096, scales: 8*8*4*2 = 512, dq: 8*128*2 = 2048
        self.assertEqual(cell, 4096 + 512 + 2048)
```
**EN:** This test exercises `test_compute_cell_size` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_compute_cell_size`。

### Lines 112-121: test case scales init / 测试用例 scales init
```python
    def test_scales_init(self):
        from sglang.srt.layers.quantization.fp4_kv_cache_quant_method import (
            NVFP4KVMethod,
        )

        m = NVFP4KVMethod(num_layers=4, device="cpu")
        # Default scales should be 1.0
        self.assertTrue(torch.all(m.k_scales_gpu == 1.0))
        self.assertTrue(torch.all(m.v_scales_gpu == 1.0))
        self.assertEqual(len(m.k_scales_gpu), 4)
```
**EN:** This test exercises `test_scales_init` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_scales_init`。

### Lines 123-171: test case quantize dequantize roundtrip / 测试用例 quantize dequantize roundtrip
```python
    @skip_if_no_blackwell_nvfp4
    def test_quantize_dequantize_roundtrip(self):
        """Test NVFP4 quantize→dequantize roundtrip on CUDA."""
        from sglang.srt.layers.quantization.fp4_kv_cache_quant_method import (
            NVFP4KVMethod,
        )

        major, minor = torch.cuda.get_device_capability()
        m = NVFP4KVMethod(num_layers=1, device="cuda", sm_version=major * 10 + minor)

        size, heads, dim = 32, 8, 128
        bufs = m.create_buffers(size, heads, dim, 1, "cuda")

        # Create random input
        k = torch.randn(4, heads, dim, dtype=torch.bfloat16, device="cuda")
        v = torch.randn(4, heads, dim, dtype=torch.bfloat16, device="cuda")
        loc = torch.arange(4, device="cuda")

        # Quantize
        m.quantize_and_store(
            bufs["k_buffer"][0],
            bufs["v_buffer"][0],
            bufs["k_scale_buffer"][0],
            bufs["v_scale_buffer"][0],
            loc,
            k,
            v,
            k_scale=m.k_scales_gpu[0:1],
            v_scale=m.v_scales_gpu[0:1],
        )

        # Dequantize
        k_fp4 = bufs["k_buffer"][0][loc]
        k_scales = bufs["k_scale_buffer"][0][loc]
        v_fp4 = bufs["v_buffer"][0][loc]
        v_scales = bufs["v_scale_buffer"][0][loc]
        k_out, v_out = m.dequantize_prev_kv(k_fp4, k_scales, v_fp4, v_scales, 0)

        # Check shapes
        self.assertEqual(k_out.shape, (4, heads, dim))
        self.assertEqual(k_out.dtype, torch.float8_e4m3fn)

        # Check roundtrip error is bounded (FP4 is very lossy, ~20% relative error)
        k_ref = k.float()
        k_rec = k_out.float()
        rel_error = (k_ref - k_rec).abs().mean() / k_ref.abs().mean()
        self.assertLess(
            rel_error, 0.5, f"NVFP4 roundtrip error too high: {rel_error:.3f}"
        )
```
**EN:** Test NVFP4 quantize→dequantize roundtrip on CUDA. This test exercises `test_quantize_dequantize_roundtrip` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test NVFP4 quantize→dequantize roundtrip on CUDA. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_quantize_dequantize_roundtrip`。

### Lines 174-174: class TestBlockFP4KVMethod declaration / 类 TestBlockFP4KVMethod 声明
```python
class TestBlockFP4KVMethod(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 175-175: supporting statements / 辅助语句
```python
    """Test BlockFP4KVMethod buffer creation and roundtrip."""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 177-185: test case properties / 测试用例 properties
```python
    def test_properties(self):
        from sglang.srt.layers.quantization.fp4_kv_cache_quant_method import (
            BlockFP4KVMethod,
        )

        m = BlockFP4KVMethod()
        self.assertEqual(m.name, "blockfp4")
        self.assertTrue(m.needs_dequant_workspace())
        self.assertFalse(m.needs_global_scale())
```
**EN:** This test exercises `test_properties` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_properties`。

### Lines 187-199: test case create buffers shapes / 测试用例 create buffers shapes
```python
    def test_create_buffers_shapes(self):
        from sglang.srt.layers.quantization.fp4_kv_cache_quant_method import (
            BlockFP4KVMethod,
        )

        m = BlockFP4KVMethod()
        size, heads, dim, layers = 64, 8, 128, 4
        bufs = m.create_buffers(size, heads, dim, layers, "cpu")

        self.assertEqual(len(bufs["k_buffer"]), layers)
        self.assertEqual(bufs["k_buffer"][0].shape, (size, heads, dim // 2))
        # MXFP4 flattens head dims for scales
        self.assertEqual(bufs["k_scale_buffer"][0].shape, (size, (heads * dim) // 16))
```
**EN:** This test exercises `test_create_buffers_shapes` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_create_buffers_shapes`。

### Lines 201-234: test case quantize dequantize roundtrip cpu / 测试用例 quantize dequantize roundtrip cpu
```python
    def test_quantize_dequantize_roundtrip_cpu(self):
        """Test MXFP4 quantize→dequantize roundtrip on CPU."""
        from sglang.srt.layers.quantization.fp4_kv_cache_quant_method import (
            BlockFP4KVMethod,
        )

        m = BlockFP4KVMethod()
        size, heads, dim = 32, 8, 128
        bufs = m.create_buffers(size, heads, dim, 1, "cpu")

        k = torch.randn(4, heads, dim, dtype=torch.bfloat16)
        v = torch.randn(4, heads, dim, dtype=torch.bfloat16)
        loc = torch.arange(4)

        # Quantize
        m.quantize_and_store(
            bufs["k_buffer"][0],
            bufs["v_buffer"][0],
            bufs["k_scale_buffer"][0],
            bufs["v_scale_buffer"][0],
            loc,
            k,
            v,
        )

        # Dequantize
        k_fp4 = bufs["k_buffer"][0][loc]
        k_scales = bufs["k_scale_buffer"][0][loc]
        v_fp4 = bufs["v_buffer"][0][loc]
        v_scales = bufs["v_scale_buffer"][0][loc]
        k_out, v_out = m.dequantize_prev_kv(k_fp4, k_scales, v_fp4, v_scales, 0)

        self.assertEqual(k_out.shape, (4, heads, dim))
        self.assertEqual(k_out.dtype, torch.float8_e4m3fn)
```
**EN:** Test MXFP4 quantize→dequantize roundtrip on CPU. This test exercises `test_quantize_dequantize_roundtrip_cpu` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test MXFP4 quantize→dequantize roundtrip on CPU. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_quantize_dequantize_roundtrip_cpu`。

### Lines 237-237: class TestBlockFP4KVQuantizeUtil declaration / 类 TestBlockFP4KVQuantizeUtil 声明
```python
class TestBlockFP4KVQuantizeUtil(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 238-238: supporting statements / 辅助语句
```python
    """Test the existing MXFP4 BlockFP4KVQuantizeUtil roundtrip."""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 240-251: test case roundtrip cpu / 测试用例 roundtrip cpu
```python
    def test_roundtrip_cpu(self):
        from sglang.srt.layers.quantization.kvfp4_tensor import BlockFP4KVQuantizeUtil

        x = torch.randn(4, 8, 128, dtype=torch.bfloat16)
        packed, scales = BlockFP4KVQuantizeUtil.batched_quantize(x)
        reconstructed = BlockFP4KVQuantizeUtil.batched_dequantize(packed, scales)

        self.assertEqual(reconstructed.shape, x.shape)
        rel_error = (
            x.float() - reconstructed.float()
        ).abs().mean() / x.float().abs().mean()
        self.assertLess(rel_error, 0.5)
```
**EN:** This test exercises `test_roundtrip_cpu` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_roundtrip_cpu`。

### Lines 254-254: class TestFP4KVCacheRecipe declaration / 类 TestFP4KVCacheRecipe 声明
```python
class TestFP4KVCacheRecipe(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 255-255: supporting statements / 辅助语句
```python
    """Test enum."""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 257-261: test case enum values / 测试用例 enum values
```python
    def test_enum_values(self):
        from sglang.srt.layers.quantization.kvfp4_tensor import FP4KVCacheRecipe

        self.assertEqual(FP4KVCacheRecipe.MXFP4.value, 1)
        self.assertEqual(FP4KVCacheRecipe.NVFP4.value, 2)
```
**EN:** This test exercises `test_enum_values` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_enum_values`。

### Lines 264-265: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `skip_if_no_blackwell_nvfp4`: Skip test if Blackwell NVFP4 is not available. / 该代码块实现 `skip_if_no_blackwell_nvfp4`，承担模块行为中的一个聚焦逻辑片段。
- `TestKVCacheQuantRegistry`: Test the registry and factory function. / 用于组织相关测试、夹具或辅助方法。
- `TestNVFP4KVMethod`: Test NVFP4KVMethod buffer creation and properties. / 用于组织相关测试、夹具或辅助方法。
- `TestBlockFP4KVMethod`: Test BlockFP4KVMethod buffer creation and roundtrip. / 用于组织相关测试、夹具或辅助方法。
- `TestBlockFP4KVQuantizeUtil`: Test the existing MXFP4 BlockFP4KVQuantizeUtil roundtrip. / 用于组织相关测试、夹具或辅助方法。
- `TestFP4KVCacheRecipe`: Test enum. / 用于组织相关测试、夹具或辅助方法。
- `TestKVCacheQuantRegistry.test_registry_contains_nvfp4_and_mxfp4`: This test exercises `test_registry_contains_nvfp4_and_mxfp4` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_registry_contains_nvfp4_and_mxfp4`。
- `TestKVCacheQuantRegistry.test_factory_nvfp4`: This test exercises `test_factory_nvfp4` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_factory_nvfp4`。
- `TestKVCacheQuantRegistry.test_factory_mxfp4`: This test exercises `test_factory_mxfp4` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_factory_mxfp4`。
- `TestKVCacheQuantRegistry.test_factory_unknown_raises`: This test exercises `test_factory_unknown_raises` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_factory_unknown_raises`。
- `TestNVFP4KVMethod.test_properties`: This test exercises `test_properties` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_properties`。
- `TestNVFP4KVMethod.test_create_buffers_shapes`: This test exercises `test_create_buffers_shapes` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_create_buffers_shapes`。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `unittest`
- **Third-party modules / 第三方模块**: `torch`
- **Internal modules / 内部模块**: `sglang.test.ci.ci_register`, `sglang.test.test_utils`

- **Total lines / 总行数**: 265
