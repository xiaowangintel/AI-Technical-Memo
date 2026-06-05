# evt_load_xe_sm80_90.py — Code Analysis / 代码分析

## Source / 来源
- Path / 路径: `test/python/cutlass/evt/evt_load_xe_sm80_90.py`
- **EN:** Stress-tests EVT load nodes with full tensors, batched tensors, row/column/scalar broadcasts, and mixed scalar arithmetic patterns.
- **CN:** 重点压力测试 EVT load 节点，覆盖整张量、批量张量、行/列/标量广播以及混合标量算术模式。

## Line-by-Line Analysis / 逐行分析

### Lines 33-46
```python
"""
Unit test for load nodes in SM90
"""

import logging
import unittest

import cutlass_cppgen
from cutlass_cppgen.backend import *
from cutlass_cppgen.epilogue import *

from utils.evt_testbed import EVTTestBed, EVTTestCaseBase

cutlass_cppgen.set_log_level(logging.WARNING)
```
**EN:** This opening block establishes the module context. The module docstring states the purpose: Unit test for load nodes in SM90 It imports the libraries needed for code generation, testing, and tensor/runtime support. `cutlass_cppgen.set_log_level(logging.WARNING)` reduces generator-side logging noise during test runs.
**CN:** 这一开头代码块建立了模块的上下文。 模块文档字符串直接说明了用途：Unit test for load nodes in SM90 它导入了代码生成、测试以及张量/运行时支持所需的库。 `cutlass_cppgen.set_log_level(logging.WARNING)` 会降低测试运行期间生成器侧的日志噪声。

### Lines 49-51
```python
@unittest.skipIf(device_cc() not in [12, 20, 80, 86, 89, 90], "This unittest is only supported on CC [12, 20, 80, 86, 89, 90]")
class TestEVTLoad(EVTTestCaseBase):
```
**EN:** `TestEVTLoad` is the main class introduced in this block. It inherits from `EVTTestCaseBase`, so the class reuses shared helpers or unittest behavior. The decorator(s) `unittest.skipIf(device_cc() not in [12, 20, 80, 86, 89, 90], 'This unittest is only supported on CC [12, 20, 80, 86, 89, 90]')` gate execution to supported environments. That base class supplies common element types, fake tensor descriptors, and standard problem-size sweeps for all contained tests.
**CN:** `TestEVTLoad` 是此处引入的主要类。 它继承自 `EVTTestCaseBase`，因此会复用共享辅助逻辑或 unittest 行为。 装饰器 `unittest.skipIf(device_cc() not in [12, 20, 80, 86, 89, 90], 'This unittest is only supported on CC [12, 20, 80, 86, 89, 90]')` 用来把执行限制在受支持的环境中。 该基类为内部所有测试提供统一的元素类型、伪张量描述符以及标准问题尺寸扫描逻辑。

### Lines 52-72
```python
    def test_tensor_load(self):
        """
        Load extra tensor with shape [m, n]
        """
        def evt_tensor_load(accum, C, aux, aux_batch):
            D = accum + C + aux + aux_batch
            return D

        for m, n, k, l in self.get_problem_sizes(8):
            example_inputs = {
                "accum": self.fake_tensor(self.element, (l, m, n)),
                "C": self.fake_tensor(self.element, (l, m, n)),
                "aux": self.fake_tensor(self.element, (m, n)),
                "aux_batch": self.fake_tensor(np.float32, (l, m, n)),
                "D": self.fake_tensor(self.element, (l, m, n)),
            }

            launcher = EVTTestBed(self.element, evt_tensor_load, example_inputs)
            input_keys = ["C", "aux", "aux_batch"]
            result_keys = ["D"]
            launcher.verify((m, n, k), input_keys, result_keys, l)
```
**EN:** `test_tensor_load(self)` is a method inside `TestEVTLoad`. The docstring describes it as: Load extra tensor with shape [m, n] It defines the inner visitor `evt_tensor_load`; the key traced steps are `D = accum + C + aux + aux_batch; return D`. `example_inputs` prepares runtime placeholders/values for `accum`, `C`, `aux`, `aux_batch`, `D`. `input_keys=['C', 'aux', 'aux_batch']` and `result_keys=['D']` tell the launcher which values are supplied and which outputs must be validated. The surrounding control flow iterates over `self.get_problem_sizes(8)` to cover multiple shapes/configurations. The method ultimately routes execution through `EVTTestBed.verify`, which compares the generated CUTLASS path against a PyTorch-based reference.
**CN:** `test_tensor_load(self)` 是一个位于 `TestEVTLoad` 内部的方法。 文档字符串将其描述为：Load extra tensor with shape [m, n] 它定义了内部 visitor `evt_tensor_load`；被 trace 的关键步骤是 `D = accum + C + aux + aux_batch; return D`。 `example_inputs` 为 `accum`、`C`、`aux`、`aux_batch`、`D` 准备运行时占位符或实际数值。 `input_keys=['C', 'aux', 'aux_batch']` 与 `result_keys=['D']` 告诉启动器哪些值由外部提供、哪些输出需要验证。 外层控制流会遍历 `self.get_problem_sizes(8)`，以覆盖多种形状或配置。 该方法最终会通过 `EVTTestBed.verify` 执行，它会把生成的 CUTLASS 路径与基于 PyTorch 的参考实现进行比较。

### Lines 74-94
```python
    def test_row_broadcast(self):
        """
        Load extra tensor with shape [1, n]
        """
        def evt_row_broadcast(accum, C, bias, bias_batch):
            D = accum + C + bias + bias_batch
            return D

        for m, n, k, l in self.get_problem_sizes(8):
            example_inputs = {
                "accum": self.fake_tensor(self.element, (l, m, n)),
                "C": self.fake_tensor(self.element, (l, m, n)),
                "bias": self.fake_tensor(self.element, (n,)),
                "bias_batch": self.fake_tensor(np.float32, (l, 1, n)),
                "D": self.fake_tensor(self.element, (l, m, n)),
            }

            launcher = EVTTestBed(self.element, evt_row_broadcast, example_inputs)
            input_keys = ["C", "bias", "bias_batch"]
            result_keys = ["D"]
            launcher.verify((m, n, k), input_keys, result_keys, l)
```
**EN:** `test_row_broadcast(self)` is a method inside `TestEVTLoad`. The docstring describes it as: Load extra tensor with shape [1, n] It defines the inner visitor `evt_row_broadcast`; the key traced steps are `D = accum + C + bias + bias_batch; return D`. `example_inputs` prepares runtime placeholders/values for `accum`, `C`, `bias`, `bias_batch`, `D`. `input_keys=['C', 'bias', 'bias_batch']` and `result_keys=['D']` tell the launcher which values are supplied and which outputs must be validated. The surrounding control flow iterates over `self.get_problem_sizes(8)` to cover multiple shapes/configurations. The method ultimately routes execution through `EVTTestBed.verify`, which compares the generated CUTLASS path against a PyTorch-based reference.
**CN:** `test_row_broadcast(self)` 是一个位于 `TestEVTLoad` 内部的方法。 文档字符串将其描述为：Load extra tensor with shape [1, n] 它定义了内部 visitor `evt_row_broadcast`；被 trace 的关键步骤是 `D = accum + C + bias + bias_batch; return D`。 `example_inputs` 为 `accum`、`C`、`bias`、`bias_batch`、`D` 准备运行时占位符或实际数值。 `input_keys=['C', 'bias', 'bias_batch']` 与 `result_keys=['D']` 告诉启动器哪些值由外部提供、哪些输出需要验证。 外层控制流会遍历 `self.get_problem_sizes(8)`，以覆盖多种形状或配置。 该方法最终会通过 `EVTTestBed.verify` 执行，它会把生成的 CUTLASS 路径与基于 PyTorch 的参考实现进行比较。

### Lines 97-117
```python
    def test_column_broadcast(self):
        """
        Load extra tensor with shape [m, 1]
        """
        def evt_column_broadcast(accum, C, bias, bias_batch):
            D = accum + C + bias + bias_batch
            return D

        for m, n, k, l in self.get_problem_sizes(8):
            example_inputs = {
                "accum": self.fake_tensor(self.element, (l, m, n)),
                "C": self.fake_tensor(self.element, (l, m, n)),
                "bias": self.fake_tensor(self.element, (m, 1)),
                "bias_batch": self.fake_tensor(np.float32, (l, m, 1)),
                "D": self.fake_tensor(self.element, (l, m, n)),
            }

            launcher = EVTTestBed(self.element, evt_column_broadcast, example_inputs)
            input_keys = ["C", "bias", "bias_batch"]
            result_keys = ["D"]
            launcher.verify((m, n, k), input_keys, result_keys, l)
```
**EN:** `test_column_broadcast(self)` is a method inside `TestEVTLoad`. The docstring describes it as: Load extra tensor with shape [m, 1] It defines the inner visitor `evt_column_broadcast`; the key traced steps are `D = accum + C + bias + bias_batch; return D`. `example_inputs` prepares runtime placeholders/values for `accum`, `C`, `bias`, `bias_batch`, `D`. `input_keys=['C', 'bias', 'bias_batch']` and `result_keys=['D']` tell the launcher which values are supplied and which outputs must be validated. The surrounding control flow iterates over `self.get_problem_sizes(8)` to cover multiple shapes/configurations. The method ultimately routes execution through `EVTTestBed.verify`, which compares the generated CUTLASS path against a PyTorch-based reference.
**CN:** `test_column_broadcast(self)` 是一个位于 `TestEVTLoad` 内部的方法。 文档字符串将其描述为：Load extra tensor with shape [m, 1] 它定义了内部 visitor `evt_column_broadcast`；被 trace 的关键步骤是 `D = accum + C + bias + bias_batch; return D`。 `example_inputs` 为 `accum`、`C`、`bias`、`bias_batch`、`D` 准备运行时占位符或实际数值。 `input_keys=['C', 'bias', 'bias_batch']` 与 `result_keys=['D']` 告诉启动器哪些值由外部提供、哪些输出需要验证。 外层控制流会遍历 `self.get_problem_sizes(8)`，以覆盖多种形状或配置。 该方法最终会通过 `EVTTestBed.verify` 执行，它会把生成的 CUTLASS 路径与基于 PyTorch 的参考实现进行比较。

### Lines 119-139
```python
    def test_scalar_broadcast(self):
        """
        Load extra tensor with shape [1, 1]
        """
        def evt_scalar_broadcast(accum, C, alpha, alpha_batch):
            D = accum + C + alpha + alpha_batch
            return D

        for m, n, k, l in self.get_problem_sizes(8):
            example_inputs = {
                "accum": self.fake_tensor(self.element, (l, m, n)),
                "C": self.fake_tensor(self.element, (l, m, n)),
                "alpha": 0.5,
                "alpha_batch": self.fake_tensor(np.float32, (l, 1, 1)),
                "D": self.fake_tensor(self.element, (l, m, n)),
            }

            launcher = EVTTestBed(self.element, evt_scalar_broadcast, example_inputs)
            input_keys = ["C", "alpha", "alpha_batch"]
            result_keys = ["D"]
            launcher.verify((m, n, k), input_keys, result_keys, l)
```
**EN:** `test_scalar_broadcast(self)` is a method inside `TestEVTLoad`. The docstring describes it as: Load extra tensor with shape [1, 1] It defines the inner visitor `evt_scalar_broadcast`; the key traced steps are `D = accum + C + alpha + alpha_batch; return D`. `example_inputs` prepares runtime placeholders/values for `accum`, `C`, `alpha`, `alpha_batch`, `D`. `input_keys=['C', 'alpha', 'alpha_batch']` and `result_keys=['D']` tell the launcher which values are supplied and which outputs must be validated. The surrounding control flow iterates over `self.get_problem_sizes(8)` to cover multiple shapes/configurations. The method ultimately routes execution through `EVTTestBed.verify`, which compares the generated CUTLASS path against a PyTorch-based reference.
**CN:** `test_scalar_broadcast(self)` 是一个位于 `TestEVTLoad` 内部的方法。 文档字符串将其描述为：Load extra tensor with shape [1, 1] 它定义了内部 visitor `evt_scalar_broadcast`；被 trace 的关键步骤是 `D = accum + C + alpha + alpha_batch; return D`。 `example_inputs` 为 `accum`、`C`、`alpha`、`alpha_batch`、`D` 准备运行时占位符或实际数值。 `input_keys=['C', 'alpha', 'alpha_batch']` 与 `result_keys=['D']` 告诉启动器哪些值由外部提供、哪些输出需要验证。 外层控制流会遍历 `self.get_problem_sizes(8)`，以覆盖多种形状或配置。 该方法最终会通过 `EVTTestBed.verify` 执行，它会把生成的 CUTLASS 路径与基于 PyTorch 的参考实现进行比较。

### Lines 141-162
```python
    def test_mixed_broadcast_combinations(self):
        """
        Test combinations of different broadcast patterns
        """
        def evt_mixed_broadcast(accum, C, row_bias, col_bias, scalar_alpha):
            D = accum + C + row_bias + col_bias + scalar_alpha
            return D

        for m, n, k, l in self.get_problem_sizes(8):
            example_inputs = {
                "accum": self.fake_tensor(self.element, (l, m, n)),
                "C": self.fake_tensor(self.element, (l, m, n)),
                "row_bias": self.fake_tensor(self.element, (n,)),
                "col_bias": self.fake_tensor(self.element, (m, 1)),
                "scalar_alpha": 0.25,
                "D": self.fake_tensor(self.element, (l, m, n)),
            }

            launcher = EVTTestBed(self.element, evt_mixed_broadcast, example_inputs)
            input_keys = ["C", "row_bias", "col_bias", "scalar_alpha"]
            result_keys = ["D"]
            launcher.verify((m, n, k), input_keys, result_keys, l)
```
**EN:** `test_mixed_broadcast_combinations(self)` is a method inside `TestEVTLoad`. The docstring describes it as: Test combinations of different broadcast patterns It defines the inner visitor `evt_mixed_broadcast`; the key traced steps are `D = accum + C + row_bias + col_bias + scalar_alpha; return D`. `example_inputs` prepares runtime placeholders/values for `accum`, `C`, `row_bias`, `col_bias`, `scalar_alpha`, `D`. `input_keys=['C', 'row_bias', 'col_bias', 'scalar_alpha']` and `result_keys=['D']` tell the launcher which values are supplied and which outputs must be validated. The surrounding control flow iterates over `self.get_problem_sizes(8)` to cover multiple shapes/configurations. The method ultimately routes execution through `EVTTestBed.verify`, which compares the generated CUTLASS path against a PyTorch-based reference.
**CN:** `test_mixed_broadcast_combinations(self)` 是一个位于 `TestEVTLoad` 内部的方法。 文档字符串将其描述为：Test combinations of different broadcast patterns 它定义了内部 visitor `evt_mixed_broadcast`；被 trace 的关键步骤是 `D = accum + C + row_bias + col_bias + scalar_alpha; return D`。 `example_inputs` 为 `accum`、`C`、`row_bias`、`col_bias`、`scalar_alpha`、`D` 准备运行时占位符或实际数值。 `input_keys=['C', 'row_bias', 'col_bias', 'scalar_alpha']` 与 `result_keys=['D']` 告诉启动器哪些值由外部提供、哪些输出需要验证。 外层控制流会遍历 `self.get_problem_sizes(8)`，以覆盖多种形状或配置。 该方法最终会通过 `EVTTestBed.verify` 执行，它会把生成的 CUTLASS 路径与基于 PyTorch 的参考实现进行比较。

### Lines 164-186
```python
    def test_multiple_tensor_loads(self):
        """
        Test loading multiple full tensors with different operations
        """
        def evt_multiple_tensors(accum, C, aux1, aux2, aux3):
            temp = accum + C + aux1
            D = temp * aux2 - aux3
            return D

        for m, n, k, l in self.get_problem_sizes(8):
            example_inputs = {
                "accum": self.fake_tensor(self.element, (l, m, n)),
                "C": self.fake_tensor(self.element, (l, m, n)),
                "aux1": self.fake_tensor(self.element, (l, m, n)),
                "aux2": self.fake_tensor(self.element, (l, m, n)),
                "aux3": self.fake_tensor(self.element, (l, m, n)),
                "D": self.fake_tensor(self.element, (l, m, n)),
            }

            launcher = EVTTestBed(self.element, evt_multiple_tensors, example_inputs)
            input_keys = ["C", "aux1", "aux2", "aux3"]
            result_keys = ["D"]
            launcher.verify((m, n, k), input_keys, result_keys, l)
```
**EN:** `test_multiple_tensor_loads(self)` is a method inside `TestEVTLoad`. The docstring describes it as: Test loading multiple full tensors with different operations It defines the inner visitor `evt_multiple_tensors`; the key traced steps are `temp = accum + C + aux1; D = temp * aux2 - aux3; return D`. `example_inputs` prepares runtime placeholders/values for `accum`, `C`, `aux1`, `aux2`, `aux3`, `D`. `input_keys=['C', 'aux1', 'aux2', 'aux3']` and `result_keys=['D']` tell the launcher which values are supplied and which outputs must be validated. The surrounding control flow iterates over `self.get_problem_sizes(8)` to cover multiple shapes/configurations. The method ultimately routes execution through `EVTTestBed.verify`, which compares the generated CUTLASS path against a PyTorch-based reference.
**CN:** `test_multiple_tensor_loads(self)` 是一个位于 `TestEVTLoad` 内部的方法。 文档字符串将其描述为：Test loading multiple full tensors with different operations 它定义了内部 visitor `evt_multiple_tensors`；被 trace 的关键步骤是 `temp = accum + C + aux1; D = temp * aux2 - aux3; return D`。 `example_inputs` 为 `accum`、`C`、`aux1`、`aux2`、`aux3`、`D` 准备运行时占位符或实际数值。 `input_keys=['C', 'aux1', 'aux2', 'aux3']` 与 `result_keys=['D']` 告诉启动器哪些值由外部提供、哪些输出需要验证。 外层控制流会遍历 `self.get_problem_sizes(8)`，以覆盖多种形状或配置。 该方法最终会通过 `EVTTestBed.verify` 执行，它会把生成的 CUTLASS 路径与基于 PyTorch 的参考实现进行比较。

### Lines 188-209
```python
    def test_batch_and_broadcast_combination(self):
        """
        Test combination of batched tensors with different broadcast patterns
        """
        def evt_batch_broadcast_combo(accum, C, batch_tensor, row_bias, scalar_alpha):
            D = accum + C + batch_tensor + row_bias + scalar_alpha
            return D

        for m, n, k, l in self.get_problem_sizes(8):
            example_inputs = {
                "accum": self.fake_tensor(self.element, (l, m, n)),
                "C": self.fake_tensor(self.element, (l, m, n)),
                "batch_tensor": self.fake_tensor(np.float32, (l, m, n)),
                "row_bias": self.fake_tensor(self.element, (n,)),
                "scalar_alpha": 1.5,
                "D": self.fake_tensor(self.element, (l, m, n)),
            }

            launcher = EVTTestBed(self.element, evt_batch_broadcast_combo, example_inputs)
            input_keys = ["C", "batch_tensor", "row_bias", "scalar_alpha"]
            result_keys = ["D"]
            launcher.verify((m, n, k), input_keys, result_keys, l)
```
**EN:** `test_batch_and_broadcast_combination(self)` is a method inside `TestEVTLoad`. The docstring describes it as: Test combination of batched tensors with different broadcast patterns It defines the inner visitor `evt_batch_broadcast_combo`; the key traced steps are `D = accum + C + batch_tensor + row_bias + scalar_alpha; return D`. `example_inputs` prepares runtime placeholders/values for `accum`, `C`, `batch_tensor`, `row_bias`, `scalar_alpha`, `D`. `input_keys=['C', 'batch_tensor', 'row_bias', 'scalar_alpha']` and `result_keys=['D']` tell the launcher which values are supplied and which outputs must be validated. The surrounding control flow iterates over `self.get_problem_sizes(8)` to cover multiple shapes/configurations. The method ultimately routes execution through `EVTTestBed.verify`, which compares the generated CUTLASS path against a PyTorch-based reference.
**CN:** `test_batch_and_broadcast_combination(self)` 是一个位于 `TestEVTLoad` 内部的方法。 文档字符串将其描述为：Test combination of batched tensors with different broadcast patterns 它定义了内部 visitor `evt_batch_broadcast_combo`；被 trace 的关键步骤是 `D = accum + C + batch_tensor + row_bias + scalar_alpha; return D`。 `example_inputs` 为 `accum`、`C`、`batch_tensor`、`row_bias`、`scalar_alpha`、`D` 准备运行时占位符或实际数值。 `input_keys=['C', 'batch_tensor', 'row_bias', 'scalar_alpha']` 与 `result_keys=['D']` 告诉启动器哪些值由外部提供、哪些输出需要验证。 外层控制流会遍历 `self.get_problem_sizes(8)`，以覆盖多种形状或配置。 该方法最终会通过 `EVTTestBed.verify` 执行，它会把生成的 CUTLASS 路径与基于 PyTorch 的参考实现进行比较。

### Lines 211-234
```python
    def test_complex_arithmetic_combination(self):
        """
        Test complex arithmetic operations with multiple load patterns
        """
        def evt_complex_arithmetic(accum, C, row_scale, col_bias, batch_offset):
            scaled = accum * row_scale
            biased = scaled + col_bias
            D = biased + C + batch_offset
            return D

        for m, n, k, l in self.get_problem_sizes(8):
            example_inputs = {
                "accum": self.fake_tensor(self.element, (l, m, n)),
                "C": self.fake_tensor(self.element, (l, m, n)),
                "row_scale": self.fake_tensor(self.element, (n,)),
                "col_bias": self.fake_tensor(self.element, (m, 1)),
                "batch_offset": self.fake_tensor(np.float32, (l, 1, 1)),
                "D": self.fake_tensor(self.element, (l, m, n)),
            }

            launcher = EVTTestBed(self.element, evt_complex_arithmetic, example_inputs)
            input_keys = ["C", "row_scale", "col_bias", "batch_offset"]
            result_keys = ["D"]
            launcher.verify((m, n, k), input_keys, result_keys, l)
```
**EN:** `test_complex_arithmetic_combination(self)` is a method inside `TestEVTLoad`. The docstring describes it as: Test complex arithmetic operations with multiple load patterns It defines the inner visitor `evt_complex_arithmetic`; the key traced steps are `scaled = accum * row_scale; biased = scaled + col_bias; D = biased + C + batch_offset; return D`. `example_inputs` prepares runtime placeholders/values for `accum`, `C`, `row_scale`, `col_bias`, `batch_offset`, `D`. `input_keys=['C', 'row_scale', 'col_bias', 'batch_offset']` and `result_keys=['D']` tell the launcher which values are supplied and which outputs must be validated. The surrounding control flow iterates over `self.get_problem_sizes(8)` to cover multiple shapes/configurations. The method ultimately routes execution through `EVTTestBed.verify`, which compares the generated CUTLASS path against a PyTorch-based reference.
**CN:** `test_complex_arithmetic_combination(self)` 是一个位于 `TestEVTLoad` 内部的方法。 文档字符串将其描述为：Test complex arithmetic operations with multiple load patterns 它定义了内部 visitor `evt_complex_arithmetic`；被 trace 的关键步骤是 `scaled = accum * row_scale; biased = scaled + col_bias; D = biased + C + batch_offset; return D`。 `example_inputs` 为 `accum`、`C`、`row_scale`、`col_bias`、`batch_offset`、`D` 准备运行时占位符或实际数值。 `input_keys=['C', 'row_scale', 'col_bias', 'batch_offset']` 与 `result_keys=['D']` 告诉启动器哪些值由外部提供、哪些输出需要验证。 外层控制流会遍历 `self.get_problem_sizes(8)`，以覆盖多种形状或配置。 该方法最终会通过 `EVTTestBed.verify` 执行，它会把生成的 CUTLASS 路径与基于 PyTorch 的参考实现进行比较。

### Lines 236-259
```python
    def test_all_broadcast_patterns_combined(self):
        """
        Test all broadcast patterns in a single operation
        """
        def evt_all_broadcasts(accum, C, full_tensor, row_bias, col_bias, scalar_alpha, batch_tensor):
            D = accum + C + full_tensor + row_bias + col_bias + scalar_alpha + batch_tensor
            return D

        for m, n, k, l in self.get_problem_sizes(8):  # Reduced iterations due to complexity
            example_inputs = {
                "accum": self.fake_tensor(self.element, (l, m, n)),
                "C": self.fake_tensor(self.element, (l, m, n)),
                "full_tensor": self.fake_tensor(self.element, (m, n)),
                "row_bias": self.fake_tensor(self.element, (n,)),
                "col_bias": self.fake_tensor(self.element, (m, 1)),
                "scalar_alpha": 0.1,
                "batch_tensor": self.fake_tensor(np.float32, (l, m, n)),
                "D": self.fake_tensor(self.element, (l, m, n)),
            }

            launcher = EVTTestBed(self.element, evt_all_broadcasts, example_inputs)
            input_keys = ["C", "full_tensor", "row_bias", "col_bias", "scalar_alpha", "batch_tensor"]
            result_keys = ["D"]
            launcher.verify((m, n, k), input_keys, result_keys, l)
```
**EN:** `test_all_broadcast_patterns_combined(self)` is a method inside `TestEVTLoad`. The docstring describes it as: Test all broadcast patterns in a single operation It defines the inner visitor `evt_all_broadcasts`; the key traced steps are `D = accum + C + full_tensor + row_bias + col_bias + scalar_alpha + batch_tensor; return D`. `example_inputs` prepares runtime placeholders/values for `accum`, `C`, `full_tensor`, `row_bias`, `col_bias`, `scalar_alpha`, `batch_tensor`, `D`. `input_keys=['C', 'full_tensor', 'row_bias', 'col_bias', 'scalar_alpha', 'batch_tensor']` and `result_keys=['D']` tell the launcher which values are supplied and which outputs must be validated. The surrounding control flow iterates over `self.get_problem_sizes(8)` to cover multiple shapes/configurations. The method ultimately routes execution through `EVTTestBed.verify`, which compares the generated CUTLASS path against a PyTorch-based reference.
**CN:** `test_all_broadcast_patterns_combined(self)` 是一个位于 `TestEVTLoad` 内部的方法。 文档字符串将其描述为：Test all broadcast patterns in a single operation 它定义了内部 visitor `evt_all_broadcasts`；被 trace 的关键步骤是 `D = accum + C + full_tensor + row_bias + col_bias + scalar_alpha + batch_tensor; return D`。 `example_inputs` 为 `accum`、`C`、`full_tensor`、`row_bias`、`col_bias`、`scalar_alpha`、`batch_tensor`、`D` 准备运行时占位符或实际数值。 `input_keys=['C', 'full_tensor', 'row_bias', 'col_bias', 'scalar_alpha', 'batch_tensor']` 与 `result_keys=['D']` 告诉启动器哪些值由外部提供、哪些输出需要验证。 外层控制流会遍历 `self.get_problem_sizes(8)`，以覆盖多种形状或配置。 该方法最终会通过 `EVTTestBed.verify` 执行，它会把生成的 CUTLASS 路径与基于 PyTorch 的参考实现进行比较。

### Lines 261-282
```python
    def test_scalar_and_tensor_load_combination(self):
        """
        Combination of scalar broadcast and tensor load
        D = accum + C + aux_tensor + scalar_alpha
        """
        def evt_scalar_tensor_combo(accum, C, aux_tensor, scalar_alpha):
            D = accum + C + aux_tensor + scalar_alpha
            return D

        for m, n, k, l in self.get_problem_sizes(8):
            example_inputs = {
                "accum": self.fake_tensor(self.element, (l, m, n)),
                "C": self.fake_tensor(self.element, (l, m, n)),
                "aux_tensor": self.fake_tensor(self.element, (m, n)),
                "scalar_alpha": 0.5,
                "D": self.fake_tensor(self.element, (l, m, n)),
            }

            launcher = EVTTestBed(self.element, evt_scalar_tensor_combo, example_inputs)
            input_keys = ["C", "aux_tensor", "scalar_alpha"]
            result_keys = ["D"]
            launcher.verify((m, n, k), input_keys, result_keys, l)
```
**EN:** `test_scalar_and_tensor_load_combination(self)` is a method inside `TestEVTLoad`. The docstring describes it as: Combination of scalar broadcast and tensor load D = accum + C + aux_tensor + scalar_alpha It defines the inner visitor `evt_scalar_tensor_combo`; the key traced steps are `D = accum + C + aux_tensor + scalar_alpha; return D`. `example_inputs` prepares runtime placeholders/values for `accum`, `C`, `aux_tensor`, `scalar_alpha`, `D`. `input_keys=['C', 'aux_tensor', 'scalar_alpha']` and `result_keys=['D']` tell the launcher which values are supplied and which outputs must be validated. The surrounding control flow iterates over `self.get_problem_sizes(8)` to cover multiple shapes/configurations. The method ultimately routes execution through `EVTTestBed.verify`, which compares the generated CUTLASS path against a PyTorch-based reference.
**CN:** `test_scalar_and_tensor_load_combination(self)` 是一个位于 `TestEVTLoad` 内部的方法。 文档字符串将其描述为：Combination of scalar broadcast and tensor load D = accum + C + aux_tensor + scalar_alpha 它定义了内部 visitor `evt_scalar_tensor_combo`；被 trace 的关键步骤是 `D = accum + C + aux_tensor + scalar_alpha; return D`。 `example_inputs` 为 `accum`、`C`、`aux_tensor`、`scalar_alpha`、`D` 准备运行时占位符或实际数值。 `input_keys=['C', 'aux_tensor', 'scalar_alpha']` 与 `result_keys=['D']` 告诉启动器哪些值由外部提供、哪些输出需要验证。 外层控制流会遍历 `self.get_problem_sizes(8)`，以覆盖多种形状或配置。 该方法最终会通过 `EVTTestBed.verify` 执行，它会把生成的 CUTLASS 路径与基于 PyTorch 的参考实现进行比较。

### Lines 284-306
```python
    def test_tensor_multiplication_with_scalar(self):
        """
        Tensor load with scalar multiplication
        D = (accum + C) * aux_tensor * scalar_scale
        """
        def evt_tensor_mult_scalar(accum, C, aux_tensor, scalar_scale):
            temp = accum + C
            D = temp * aux_tensor * scalar_scale
            return D

        for m, n, k, l in self.get_problem_sizes(8):
            example_inputs = {
                "accum": self.fake_tensor(self.element, (l, m, n)),
                "C": self.fake_tensor(self.element, (l, m, n)),
                "aux_tensor": self.fake_tensor(self.element, (m, n)),
                "scalar_scale": 2.0,
                "D": self.fake_tensor(self.element, (l, m, n)),
            }

            launcher = EVTTestBed(self.element, evt_tensor_mult_scalar, example_inputs)
            input_keys = ["C", "aux_tensor", "scalar_scale"]
            result_keys = ["D"]
            launcher.verify((m, n, k), input_keys, result_keys, l)
```
**EN:** `test_tensor_multiplication_with_scalar(self)` is a method inside `TestEVTLoad`. The docstring describes it as: Tensor load with scalar multiplication D = (accum + C) * aux_tensor * scalar_scale It defines the inner visitor `evt_tensor_mult_scalar`; the key traced steps are `temp = accum + C; D = temp * aux_tensor * scalar_scale; return D`. `example_inputs` prepares runtime placeholders/values for `accum`, `C`, `aux_tensor`, `scalar_scale`, `D`. `input_keys=['C', 'aux_tensor', 'scalar_scale']` and `result_keys=['D']` tell the launcher which values are supplied and which outputs must be validated. The surrounding control flow iterates over `self.get_problem_sizes(8)` to cover multiple shapes/configurations. The method ultimately routes execution through `EVTTestBed.verify`, which compares the generated CUTLASS path against a PyTorch-based reference.
**CN:** `test_tensor_multiplication_with_scalar(self)` 是一个位于 `TestEVTLoad` 内部的方法。 文档字符串将其描述为：Tensor load with scalar multiplication D = (accum + C) * aux_tensor * scalar_scale 它定义了内部 visitor `evt_tensor_mult_scalar`；被 trace 的关键步骤是 `temp = accum + C; D = temp * aux_tensor * scalar_scale; return D`。 `example_inputs` 为 `accum`、`C`、`aux_tensor`、`scalar_scale`、`D` 准备运行时占位符或实际数值。 `input_keys=['C', 'aux_tensor', 'scalar_scale']` 与 `result_keys=['D']` 告诉启动器哪些值由外部提供、哪些输出需要验证。 外层控制流会遍历 `self.get_problem_sizes(8)`，以覆盖多种形状或配置。 该方法最终会通过 `EVTTestBed.verify` 执行，它会把生成的 CUTLASS 路径与基于 PyTorch 的参考实现进行比较。

### Lines 308-332
```python
    def test_batched_tensor_with_scalar_operations(self):
        """
        Batched tensor load combined with scalar operations
        D = (accum + aux_tensor) * alpha + aux_batch * beta
        """
        def evt_batched_tensor_scalar(accum, C, aux_tensor, aux_batch, alpha, beta):
            temp = accum + aux_tensor
            D = temp * alpha + C + aux_batch * beta
            return D

        for m, n, k, l in self.get_problem_sizes(8):
            example_inputs = {
                "accum": self.fake_tensor(self.element, (l, m, n)),
                "C": self.fake_tensor(self.element, (l, m, n)),
                "aux_tensor": self.fake_tensor(self.element, (m, n)),
                "aux_batch": self.fake_tensor(np.float32, (l, m, n)),
                "alpha": 1.3,
                "beta": 0.6,
                "D": self.fake_tensor(self.element, (l, m, n)),
            }

            launcher = EVTTestBed(self.element, evt_batched_tensor_scalar, example_inputs)
            input_keys = ["C", "aux_tensor", "aux_batch", "alpha", "beta"]
            result_keys = ["D"]
            launcher.verify((m, n, k), input_keys, result_keys, l)
```
**EN:** `test_batched_tensor_with_scalar_operations(self)` is a method inside `TestEVTLoad`. The docstring describes it as: Batched tensor load combined with scalar operations D = (accum + aux_tensor) * alpha + aux_batch * beta It defines the inner visitor `evt_batched_tensor_scalar`; the key traced steps are `temp = accum + aux_tensor; D = temp * alpha + C + aux_batch * beta; return D`. `example_inputs` prepares runtime placeholders/values for `accum`, `C`, `aux_tensor`, `aux_batch`, `alpha`, `beta`, `D`. `input_keys=['C', 'aux_tensor', 'aux_batch', 'alpha', 'beta']` and `result_keys=['D']` tell the launcher which values are supplied and which outputs must be validated. The surrounding control flow iterates over `self.get_problem_sizes(8)` to cover multiple shapes/configurations. The method ultimately routes execution through `EVTTestBed.verify`, which compares the generated CUTLASS path against a PyTorch-based reference.
**CN:** `test_batched_tensor_with_scalar_operations(self)` 是一个位于 `TestEVTLoad` 内部的方法。 文档字符串将其描述为：Batched tensor load combined with scalar operations D = (accum + aux_tensor) * alpha + aux_batch * beta 它定义了内部 visitor `evt_batched_tensor_scalar`；被 trace 的关键步骤是 `temp = accum + aux_tensor; D = temp * alpha + C + aux_batch * beta; return D`。 `example_inputs` 为 `accum`、`C`、`aux_tensor`、`aux_batch`、`alpha`、`beta`、`D` 准备运行时占位符或实际数值。 `input_keys=['C', 'aux_tensor', 'aux_batch', 'alpha', 'beta']` 与 `result_keys=['D']` 告诉启动器哪些值由外部提供、哪些输出需要验证。 外层控制流会遍历 `self.get_problem_sizes(8)`，以覆盖多种形状或配置。 该方法最终会通过 `EVTTestBed.verify` 执行，它会把生成的 CUTLASS 路径与基于 PyTorch 的参考实现进行比较。

### Lines 334-357
```python
    def test_tensor_scalar_division_combination(self):
        """
        Division operations with tensor loads and scalars
        D = (accum + aux_tensor) / divisor + C * multiplier
        """
        def evt_tensor_scalar_division(accum, C, aux_tensor, divisor, multiplier):
            temp = accum + aux_tensor
            D = temp / divisor + C * multiplier
            return D

        for m, n, k, l in self.get_problem_sizes(8):
            example_inputs = {
                "accum": self.fake_tensor(self.element, (l, m, n)),
                "C": self.fake_tensor(self.element, (l, m, n)),
                "aux_tensor": self.fake_tensor(self.element, (m, n)),
                "divisor": 2.0,
                "multiplier": 1.5,
                "D": self.fake_tensor(self.element, (l, m, n)),
            }

            launcher = EVTTestBed(self.element, evt_tensor_scalar_division, example_inputs)
            input_keys = ["C", "aux_tensor", "divisor", "multiplier"]
            result_keys = ["D"]
            launcher.verify((m, n, k), input_keys, result_keys, l)
```
**EN:** `test_tensor_scalar_division_combination(self)` is a method inside `TestEVTLoad`. The docstring describes it as: Division operations with tensor loads and scalars D = (accum + aux_tensor) / divisor + C * multiplier It defines the inner visitor `evt_tensor_scalar_division`; the key traced steps are `temp = accum + aux_tensor; D = temp / divisor + C * multiplier; return D`. `example_inputs` prepares runtime placeholders/values for `accum`, `C`, `aux_tensor`, `divisor`, `multiplier`, `D`. `input_keys=['C', 'aux_tensor', 'divisor', 'multiplier']` and `result_keys=['D']` tell the launcher which values are supplied and which outputs must be validated. The surrounding control flow iterates over `self.get_problem_sizes(8)` to cover multiple shapes/configurations. The method ultimately routes execution through `EVTTestBed.verify`, which compares the generated CUTLASS path against a PyTorch-based reference.
**CN:** `test_tensor_scalar_division_combination(self)` 是一个位于 `TestEVTLoad` 内部的方法。 文档字符串将其描述为：Division operations with tensor loads and scalars D = (accum + aux_tensor) / divisor + C * multiplier 它定义了内部 visitor `evt_tensor_scalar_division`；被 trace 的关键步骤是 `temp = accum + aux_tensor; D = temp / divisor + C * multiplier; return D`。 `example_inputs` 为 `accum`、`C`、`aux_tensor`、`divisor`、`multiplier`、`D` 准备运行时占位符或实际数值。 `input_keys=['C', 'aux_tensor', 'divisor', 'multiplier']` 与 `result_keys=['D']` 告诉启动器哪些值由外部提供、哪些输出需要验证。 外层控制流会遍历 `self.get_problem_sizes(8)`，以覆盖多种形状或配置。 该方法最终会通过 `EVTTestBed.verify` 执行，它会把生成的 CUTLASS 路径与基于 PyTorch 的参考实现进行比较。

### Lines 359-385
```python
    def test_complex_scalar_tensor_expression(self):
        """
        Complex expression combining multiple scalars and tensor loads
        D = (accum * scale1 + aux1) * scale2 + (C + aux2) * scale3
        """
        def evt_complex_scalar_tensor(accum, C, aux1, aux2, scale1, scale2, scale3):
            temp1 = accum * scale1 + aux1
            temp2 = C + aux2
            D = temp1 * scale2 + temp2 * scale3
            return D

        for m, n, k, l in self.get_problem_sizes(8):
            example_inputs = {
                "accum": self.fake_tensor(self.element, (l, m, n)),
                "C": self.fake_tensor(self.element, (l, m, n)),
                "aux1": self.fake_tensor(self.element, (m, n)),
                "aux2": self.fake_tensor(self.element, (m, n)),
                "scale1": 0.8,
                "scale2": 1.2,
                "scale3": 0.9,
                "D": self.fake_tensor(self.element, (l, m, n)),
            }

            launcher = EVTTestBed(self.element, evt_complex_scalar_tensor, example_inputs)
            input_keys = ["C", "aux1", "aux2", "scale1", "scale2", "scale3"]
            result_keys = ["D"]
            launcher.verify((m, n, k), input_keys, result_keys, l)
```
**EN:** `test_complex_scalar_tensor_expression(self)` is a method inside `TestEVTLoad`. The docstring describes it as: Complex expression combining multiple scalars and tensor loads D = (accum * scale1 + aux1) * scale2 + (C + aux2) * scale3 It defines the inner visitor `evt_complex_scalar_tensor`; the key traced steps are `temp1 = accum * scale1 + aux1; temp2 = C + aux2; D = temp1 * scale2 + temp2 * scale3; return D`. `example_inputs` prepares runtime placeholders/values for `accum`, `C`, `aux1`, `aux2`, `scale1`, `scale2`, `scale3`, `D`. `input_keys=['C', 'aux1', 'aux2', 'scale1', 'scale2', 'scale3']` and `result_keys=['D']` tell the launcher which values are supplied and which outputs must be validated. The surrounding control flow iterates over `self.get_problem_sizes(8)` to cover multiple shapes/configurations. The method ultimately routes execution through `EVTTestBed.verify`, which compares the generated CUTLASS path against a PyTorch-based reference.
**CN:** `test_complex_scalar_tensor_expression(self)` 是一个位于 `TestEVTLoad` 内部的方法。 文档字符串将其描述为：Complex expression combining multiple scalars and tensor loads D = (accum * scale1 + aux1) * scale2 + (C + aux2) * scale3 它定义了内部 visitor `evt_complex_scalar_tensor`；被 trace 的关键步骤是 `temp1 = accum * scale1 + aux1; temp2 = C + aux2; D = temp1 * scale2 + temp2 * scale3; return D`。 `example_inputs` 为 `accum`、`C`、`aux1`、`aux2`、`scale1`、`scale2`、`scale3`、`D` 准备运行时占位符或实际数值。 `input_keys=['C', 'aux1', 'aux2', 'scale1', 'scale2', 'scale3']` 与 `result_keys=['D']` 告诉启动器哪些值由外部提供、哪些输出需要验证。 外层控制流会遍历 `self.get_problem_sizes(8)`，以覆盖多种形状或配置。 该方法最终会通过 `EVTTestBed.verify` 执行，它会把生成的 CUTLASS 路径与基于 PyTorch 的参考实现进行比较。

### Lines 387-413
```python
    def test_multiple_scalars_with_tensor_load_corrected(self):
        """
        CORRECTED: Multiple scalar broadcasts combined with tensor load
        D = accum * alpha + C * beta + aux_tensor + gamma
        """
        def evt_multi_scalar_tensor(accum, C, aux_tensor, alpha, beta, gamma):
            temp1 = accum * alpha
            temp2 = C * beta
            temp3 = aux_tensor + gamma  # Add scalar to tensor first
            D = temp1 + temp2 + temp3
            return D

        for m, n, k, l in self.get_problem_sizes(8):
            example_inputs = {
                "accum": self.fake_tensor(self.element, (l, m, n)),
                "C": self.fake_tensor(self.element, (l, m, n)),
                "aux_tensor": self.fake_tensor(self.element, (m, n)),  # Full tensor, not broadcast
                "alpha": 1.2,
                "beta": 0.8,
                "gamma": 0.3,
                "D": self.fake_tensor(self.element, (l, m, n)),
            }

            launcher = EVTTestBed(self.element, evt_multi_scalar_tensor, example_inputs)
            input_keys = ["C", "aux_tensor", "alpha", "beta", "gamma"]
            result_keys = ["D"]
            launcher.verify((m, n, k), input_keys, result_keys, l)
```
**EN:** `test_multiple_scalars_with_tensor_load_corrected(self)` is a method inside `TestEVTLoad`. The docstring describes it as: CORRECTED: Multiple scalar broadcasts combined with tensor load D = accum * alpha + C * beta + aux_tensor + gamma It defines the inner visitor `evt_multi_scalar_tensor`; the key traced steps are `temp1 = accum * alpha; temp2 = C * beta; temp3 = aux_tensor + gamma; D = temp1 + temp2 + temp3`. `example_inputs` prepares runtime placeholders/values for `accum`, `C`, `aux_tensor`, `alpha`, `beta`, `gamma`, `D`. `input_keys=['C', 'aux_tensor', 'alpha', 'beta', 'gamma']` and `result_keys=['D']` tell the launcher which values are supplied and which outputs must be validated. The surrounding control flow iterates over `self.get_problem_sizes(8)` to cover multiple shapes/configurations. The method ultimately routes execution through `EVTTestBed.verify`, which compares the generated CUTLASS path against a PyTorch-based reference.
**CN:** `test_multiple_scalars_with_tensor_load_corrected(self)` 是一个位于 `TestEVTLoad` 内部的方法。 文档字符串将其描述为：CORRECTED: Multiple scalar broadcasts combined with tensor load D = accum * alpha + C * beta + aux_tensor + gamma 它定义了内部 visitor `evt_multi_scalar_tensor`；被 trace 的关键步骤是 `temp1 = accum * alpha; temp2 = C * beta; temp3 = aux_tensor + gamma; D = temp1 + temp2 + temp3`。 `example_inputs` 为 `accum`、`C`、`aux_tensor`、`alpha`、`beta`、`gamma`、`D` 准备运行时占位符或实际数值。 `input_keys=['C', 'aux_tensor', 'alpha', 'beta', 'gamma']` 与 `result_keys=['D']` 告诉启动器哪些值由外部提供、哪些输出需要验证。 外层控制流会遍历 `self.get_problem_sizes(8)`，以覆盖多种形状或配置。 该方法最终会通过 `EVTTestBed.verify` 执行，它会把生成的 CUTLASS 路径与基于 PyTorch 的参考实现进行比较。

### Lines 415-440
```python
    def test_multiple_tensor_loads_with_scalars_corrected(self):
        """
        CORRECTED: Multiple tensor loads combined with scalar operations
        D = (aux1 * scalar1) + (aux2 * scalar2) + accum + C
        """
        def evt_multi_tensor_scalar(accum, C, aux1, aux2, scalar1, scalar2):
            scaled1 = aux1 * scalar1
            scaled2 = aux2 * scalar2
            D = scaled1 + scaled2 + accum + C
            return D

        for m, n, k, l in self.get_problem_sizes(8):
            example_inputs = {
                "accum": self.fake_tensor(self.element, (l, m, n)),
                "C": self.fake_tensor(self.element, (l, m, n)),
                "aux1": self.fake_tensor(self.element, (m, n)),  # Full tensor
                "aux2": self.fake_tensor(self.element, (m, n)),  # Full tensor
                "scalar1": 1.5,
                "scalar2": 0.7,
                "D": self.fake_tensor(self.element, (l, m, n)),
            }

            launcher = EVTTestBed(self.element, evt_multi_tensor_scalar, example_inputs)
            input_keys = ["C", "aux1", "aux2", "scalar1", "scalar2"]
            result_keys = ["D"]
            launcher.verify((m, n, k), input_keys, result_keys, l)
```
**EN:** `test_multiple_tensor_loads_with_scalars_corrected(self)` is a method inside `TestEVTLoad`. The docstring describes it as: CORRECTED: Multiple tensor loads combined with scalar operations D = (aux1 * scalar1) + (aux2 * scalar2) + accum + C It defines the inner visitor `evt_multi_tensor_scalar`; the key traced steps are `scaled1 = aux1 * scalar1; scaled2 = aux2 * scalar2; D = scaled1 + scaled2 + accum + C; return D`. `example_inputs` prepares runtime placeholders/values for `accum`, `C`, `aux1`, `aux2`, `scalar1`, `scalar2`, `D`. `input_keys=['C', 'aux1', 'aux2', 'scalar1', 'scalar2']` and `result_keys=['D']` tell the launcher which values are supplied and which outputs must be validated. The surrounding control flow iterates over `self.get_problem_sizes(8)` to cover multiple shapes/configurations. The method ultimately routes execution through `EVTTestBed.verify`, which compares the generated CUTLASS path against a PyTorch-based reference.
**CN:** `test_multiple_tensor_loads_with_scalars_corrected(self)` 是一个位于 `TestEVTLoad` 内部的方法。 文档字符串将其描述为：CORRECTED: Multiple tensor loads combined with scalar operations D = (aux1 * scalar1) + (aux2 * scalar2) + accum + C 它定义了内部 visitor `evt_multi_tensor_scalar`；被 trace 的关键步骤是 `scaled1 = aux1 * scalar1; scaled2 = aux2 * scalar2; D = scaled1 + scaled2 + accum + C; return D`。 `example_inputs` 为 `accum`、`C`、`aux1`、`aux2`、`scalar1`、`scalar2`、`D` 准备运行时占位符或实际数值。 `input_keys=['C', 'aux1', 'aux2', 'scalar1', 'scalar2']` 与 `result_keys=['D']` 告诉启动器哪些值由外部提供、哪些输出需要验证。 外层控制流会遍历 `self.get_problem_sizes(8)`，以覆盖多种形状或配置。 该方法最终会通过 `EVTTestBed.verify` 执行，它会把生成的 CUTLASS 路径与基于 PyTorch 的参考实现进行比较。

### Lines 442-466
```python
    def test_scalar_tensor_arithmetic_chain_corrected(self):
        """
        CORRECTED: Chain of arithmetic operations with scalars and tensors
        temp1 = accum * alpha, temp2 = aux_tensor + beta, D = temp1 + C + temp2
        """
        def evt_scalar_tensor_chain(accum, C, aux_tensor, alpha, beta):
            temp1 = accum * alpha
            temp2 = aux_tensor + beta
            D = temp1 + C + temp2
            return D

        for m, n, k, l in self.get_problem_sizes(8):
            example_inputs = {
                "accum": self.fake_tensor(self.element, (l, m, n)),
                "C": self.fake_tensor(self.element, (l, m, n)),
                "aux_tensor": self.fake_tensor(self.element, (m, n)),  # Full tensor
                "alpha": 0.9,
                "beta": 0.1,
                "D": self.fake_tensor(self.element, (l, m, n)),
            }

            launcher = EVTTestBed(self.element, evt_scalar_tensor_chain, example_inputs)
            input_keys = ["C", "aux_tensor", "alpha", "beta"]
            result_keys = ["D"]  # ← FIXED: Added missing result_keys
            launcher.verify((m, n, k), input_keys, result_keys, l)
```
**EN:** `test_scalar_tensor_arithmetic_chain_corrected(self)` is a method inside `TestEVTLoad`. The docstring describes it as: CORRECTED: Chain of arithmetic operations with scalars and tensors temp1 = accum * alpha, temp2 = aux_tensor + beta, D = temp1 + C + temp2 It defines the inner visitor `evt_scalar_tensor_chain`; the key traced steps are `temp1 = accum * alpha; temp2 = aux_tensor + beta; D = temp1 + C + temp2; return D`. `example_inputs` prepares runtime placeholders/values for `accum`, `C`, `aux_tensor`, `alpha`, `beta`, `D`. `input_keys=['C', 'aux_tensor', 'alpha', 'beta']` and `result_keys=['D']` tell the launcher which values are supplied and which outputs must be validated. The surrounding control flow iterates over `self.get_problem_sizes(8)` to cover multiple shapes/configurations. The method ultimately routes execution through `EVTTestBed.verify`, which compares the generated CUTLASS path against a PyTorch-based reference.
**CN:** `test_scalar_tensor_arithmetic_chain_corrected(self)` 是一个位于 `TestEVTLoad` 内部的方法。 文档字符串将其描述为：CORRECTED: Chain of arithmetic operations with scalars and tensors temp1 = accum * alpha, temp2 = aux_tensor + beta, D = temp1 + C + temp2 它定义了内部 visitor `evt_scalar_tensor_chain`；被 trace 的关键步骤是 `temp1 = accum * alpha; temp2 = aux_tensor + beta; D = temp1 + C + temp2; return D`。 `example_inputs` 为 `accum`、`C`、`aux_tensor`、`alpha`、`beta`、`D` 准备运行时占位符或实际数值。 `input_keys=['C', 'aux_tensor', 'alpha', 'beta']` 与 `result_keys=['D']` 告诉启动器哪些值由外部提供、哪些输出需要验证。 外层控制流会遍历 `self.get_problem_sizes(8)`，以覆盖多种形状或配置。 该方法最终会通过 `EVTTestBed.verify` 执行，它会把生成的 CUTLASS 路径与基于 PyTorch 的参考实现进行比较。

### Lines 468-490
```python
    def test_simple_scalar_arithmetic(self):
        """
        NEW: Simple scalar arithmetic that should work
        D = accum * alpha + C + beta
        """
        def evt_simple_scalar_arithmetic(accum, C, alpha, beta):
            temp = accum * alpha
            D = temp + C + beta
            return D

        for m, n, k, l in self.get_problem_sizes(8):
            example_inputs = {
                "accum": self.fake_tensor(self.element, (l, m, n)),
                "C": self.fake_tensor(self.element, (l, m, n)),
                "alpha": 1.5,
                "beta": 0.5,
                "D": self.fake_tensor(self.element, (l, m, n)),
            }

            launcher = EVTTestBed(self.element, evt_simple_scalar_arithmetic, example_inputs)
            input_keys = ["C", "alpha", "beta"]
            result_keys = ["D"]
            launcher.verify((m, n, k), input_keys, result_keys, l)
```
**EN:** `test_simple_scalar_arithmetic(self)` is a method inside `TestEVTLoad`. The docstring describes it as: NEW: Simple scalar arithmetic that should work D = accum * alpha + C + beta It defines the inner visitor `evt_simple_scalar_arithmetic`; the key traced steps are `temp = accum * alpha; D = temp + C + beta; return D`. `example_inputs` prepares runtime placeholders/values for `accum`, `C`, `alpha`, `beta`, `D`. `input_keys=['C', 'alpha', 'beta']` and `result_keys=['D']` tell the launcher which values are supplied and which outputs must be validated. The surrounding control flow iterates over `self.get_problem_sizes(8)` to cover multiple shapes/configurations. The method ultimately routes execution through `EVTTestBed.verify`, which compares the generated CUTLASS path against a PyTorch-based reference.
**CN:** `test_simple_scalar_arithmetic(self)` 是一个位于 `TestEVTLoad` 内部的方法。 文档字符串将其描述为：NEW: Simple scalar arithmetic that should work D = accum * alpha + C + beta 它定义了内部 visitor `evt_simple_scalar_arithmetic`；被 trace 的关键步骤是 `temp = accum * alpha; D = temp + C + beta; return D`。 `example_inputs` 为 `accum`、`C`、`alpha`、`beta`、`D` 准备运行时占位符或实际数值。 `input_keys=['C', 'alpha', 'beta']` 与 `result_keys=['D']` 告诉启动器哪些值由外部提供、哪些输出需要验证。 外层控制流会遍历 `self.get_problem_sizes(8)`，以覆盖多种形状或配置。 该方法最终会通过 `EVTTestBed.verify` 执行，它会把生成的 CUTLASS 路径与基于 PyTorch 的参考实现进行比较。

### Lines 492-513
```python
    def test_scalar_only_operations(self):
        """
        NEW: Test with only scalar operations
        D = accum * scale + offset
        """
        def evt_scalar_only(accum, C, scale, offset):
            D = accum * scale + C + offset
            return D

        for m, n, k, l in self.get_problem_sizes(8):
            example_inputs = {
                "accum": self.fake_tensor(self.element, (l, m, n)),
                "C": self.fake_tensor(self.element, (l, m, n)),
                "scale": 2.0,
                "offset": 1.0,
                "D": self.fake_tensor(self.element, (l, m, n)),
            }

            launcher = EVTTestBed(self.element, evt_scalar_only, example_inputs)
            input_keys = ["C", "scale", "offset"]
            result_keys = ["D"]
            launcher.verify((m, n, k), input_keys, result_keys, l)
```
**EN:** `test_scalar_only_operations(self)` is a method inside `TestEVTLoad`. The docstring describes it as: NEW: Test with only scalar operations D = accum * scale + offset It defines the inner visitor `evt_scalar_only`; the key traced steps are `D = accum * scale + C + offset; return D`. `example_inputs` prepares runtime placeholders/values for `accum`, `C`, `scale`, `offset`, `D`. `input_keys=['C', 'scale', 'offset']` and `result_keys=['D']` tell the launcher which values are supplied and which outputs must be validated. The surrounding control flow iterates over `self.get_problem_sizes(8)` to cover multiple shapes/configurations. The method ultimately routes execution through `EVTTestBed.verify`, which compares the generated CUTLASS path against a PyTorch-based reference.
**CN:** `test_scalar_only_operations(self)` 是一个位于 `TestEVTLoad` 内部的方法。 文档字符串将其描述为：NEW: Test with only scalar operations D = accum * scale + offset 它定义了内部 visitor `evt_scalar_only`；被 trace 的关键步骤是 `D = accum * scale + C + offset; return D`。 `example_inputs` 为 `accum`、`C`、`scale`、`offset`、`D` 准备运行时占位符或实际数值。 `input_keys=['C', 'scale', 'offset']` 与 `result_keys=['D']` 告诉启动器哪些值由外部提供、哪些输出需要验证。 外层控制流会遍历 `self.get_problem_sizes(8)`，以覆盖多种形状或配置。 该方法最终会通过 `EVTTestBed.verify` 执行，它会把生成的 CUTLASS 路径与基于 PyTorch 的参考实现进行比较。

### Lines 515-537
```python
    def test_multiple_aux_tensors_different_types(self):
            """
            Load multiple auxiliary tensors with different element types
            Tests type conversion and multiple tensor loads
            """
            def evt_multiple_aux_tensors(accum, C, aux_fp32, aux_batch, scalar_scale):
                D = accum * scalar_scale + C + aux_fp32 + aux_batch
                return D

            for m, n, k, l in self.get_problem_sizes(8):
                example_inputs = {
                    "accum": self.fake_tensor(self.element, (l, m, n)),
                    "C": self.fake_tensor(self.element, (l, m, n)),
                    "aux_fp32": self.fake_tensor(np.float32, (m, n)),
                    "aux_batch": self.fake_tensor(self.element, (l, m, n)),
                    "scalar_scale": 0.25,
                    "D": self.fake_tensor(self.element, (l, m, n)),
                }

                launcher = EVTTestBed(self.element, evt_multiple_aux_tensors, example_inputs)
                input_keys = ["C", "aux_fp32", "aux_batch", "scalar_scale"]
                result_keys = ["D"]
                launcher.verify((m, n, k), input_keys, result_keys, l)
```
**EN:** `test_multiple_aux_tensors_different_types(self)` is a method inside `TestEVTLoad`. The docstring describes it as: Load multiple auxiliary tensors with different element types Tests type conversion and multiple tensor loads It defines the inner visitor `evt_multiple_aux_tensors`; the key traced steps are `D = accum * scalar_scale + C + aux_fp32 + aux_batch; return D`. `example_inputs` prepares runtime placeholders/values for `accum`, `C`, `aux_fp32`, `aux_batch`, `scalar_scale`, `D`. `input_keys=['C', 'aux_fp32', 'aux_batch', 'scalar_scale']` and `result_keys=['D']` tell the launcher which values are supplied and which outputs must be validated. The surrounding control flow iterates over `self.get_problem_sizes(8)` to cover multiple shapes/configurations. The method ultimately routes execution through `EVTTestBed.verify`, which compares the generated CUTLASS path against a PyTorch-based reference.
**CN:** `test_multiple_aux_tensors_different_types(self)` 是一个位于 `TestEVTLoad` 内部的方法。 文档字符串将其描述为：Load multiple auxiliary tensors with different element types Tests type conversion and multiple tensor loads 它定义了内部 visitor `evt_multiple_aux_tensors`；被 trace 的关键步骤是 `D = accum * scalar_scale + C + aux_fp32 + aux_batch; return D`。 `example_inputs` 为 `accum`、`C`、`aux_fp32`、`aux_batch`、`scalar_scale`、`D` 准备运行时占位符或实际数值。 `input_keys=['C', 'aux_fp32', 'aux_batch', 'scalar_scale']` 与 `result_keys=['D']` 告诉启动器哪些值由外部提供、哪些输出需要验证。 外层控制流会遍历 `self.get_problem_sizes(8)`，以覆盖多种形状或配置。 该方法最终会通过 `EVTTestBed.verify` 执行，它会把生成的 CUTLASS 路径与基于 PyTorch 的参考实现进行比较。

### Lines 539-560
```python
    def test_tensor_load_with_multiply(self):
        """
        Load full auxiliary tensor and use with multiplication operations
        Tests tensor load with element-wise multiply instead of just addition
        """
        def evt_tensor_load_multiply(accum, C, scale_matrix, offset_matrix):
            D = (accum + C) * scale_matrix + offset_matrix
            return D

        for m, n, k, l in self.get_problem_sizes(8):
            example_inputs = {
                "accum": self.fake_tensor(self.element, (l, m, n)),
                "C": self.fake_tensor(self.element, (l, m, n)),
                "scale_matrix": self.fake_tensor(self.element, (m, n)),
                "offset_matrix": self.fake_tensor(self.element, (m, n)),
                "D": self.fake_tensor(self.element, (l, m, n)),
            }

            launcher = EVTTestBed(self.element, evt_tensor_load_multiply, example_inputs)
            input_keys = ["C", "scale_matrix", "offset_matrix"]
            result_keys = ["D"]
            launcher.verify((m, n, k), input_keys, result_keys, l)
```
**EN:** `test_tensor_load_with_multiply(self)` is a method inside `TestEVTLoad`. The docstring describes it as: Load full auxiliary tensor and use with multiplication operations Tests tensor load with element-wise multiply instead of just addition It defines the inner visitor `evt_tensor_load_multiply`; the key traced steps are `D = (accum + C) * scale_matrix + offset_matrix; return D`. `example_inputs` prepares runtime placeholders/values for `accum`, `C`, `scale_matrix`, `offset_matrix`, `D`. `input_keys=['C', 'scale_matrix', 'offset_matrix']` and `result_keys=['D']` tell the launcher which values are supplied and which outputs must be validated. The surrounding control flow iterates over `self.get_problem_sizes(8)` to cover multiple shapes/configurations. The method ultimately routes execution through `EVTTestBed.verify`, which compares the generated CUTLASS path against a PyTorch-based reference.
**CN:** `test_tensor_load_with_multiply(self)` 是一个位于 `TestEVTLoad` 内部的方法。 文档字符串将其描述为：Load full auxiliary tensor and use with multiplication operations Tests tensor load with element-wise multiply instead of just addition 它定义了内部 visitor `evt_tensor_load_multiply`；被 trace 的关键步骤是 `D = (accum + C) * scale_matrix + offset_matrix; return D`。 `example_inputs` 为 `accum`、`C`、`scale_matrix`、`offset_matrix`、`D` 准备运行时占位符或实际数值。 `input_keys=['C', 'scale_matrix', 'offset_matrix']` 与 `result_keys=['D']` 告诉启动器哪些值由外部提供、哪些输出需要验证。 外层控制流会遍历 `self.get_problem_sizes(8)`，以覆盖多种形状或配置。 该方法最终会通过 `EVTTestBed.verify` 执行，它会把生成的 CUTLASS 路径与基于 PyTorch 的参考实现进行比较。

### Lines 562-584
```python
    def test_multiple_scalar_broadcast(self):
        """
        Load and broadcast multiple scalar values with different operations
        Tests scalar broadcast with multiplication, addition, and division
        """
        def evt_multiple_scalars(accum, C, alpha, beta, gamma):
            D = (accum * alpha + C * beta) / gamma
            return D

        for m, n, k, l in self.get_problem_sizes(8):
            example_inputs = {
                "accum": self.fake_tensor(self.element, (l, m, n)),
                "C": self.fake_tensor(self.element, (l, m, n)),
                "alpha": 1.5,
                "beta": 2.0,
                "gamma": 0.5,
                "D": self.fake_tensor(self.element, (l, m, n)),
            }

            launcher = EVTTestBed(self.element, evt_multiple_scalars, example_inputs)
            input_keys = ["C", "alpha", "beta", "gamma"]
            result_keys = ["D"]
            launcher.verify((m, n, k), input_keys, result_keys, l)
```
**EN:** `test_multiple_scalar_broadcast(self)` is a method inside `TestEVTLoad`. The docstring describes it as: Load and broadcast multiple scalar values with different operations Tests scalar broadcast with multiplication, addition, and division It defines the inner visitor `evt_multiple_scalars`; the key traced steps are `D = (accum * alpha + C * beta) / gamma; return D`. `example_inputs` prepares runtime placeholders/values for `accum`, `C`, `alpha`, `beta`, `gamma`, `D`. `input_keys=['C', 'alpha', 'beta', 'gamma']` and `result_keys=['D']` tell the launcher which values are supplied and which outputs must be validated. The surrounding control flow iterates over `self.get_problem_sizes(8)` to cover multiple shapes/configurations. The method ultimately routes execution through `EVTTestBed.verify`, which compares the generated CUTLASS path against a PyTorch-based reference.
**CN:** `test_multiple_scalar_broadcast(self)` 是一个位于 `TestEVTLoad` 内部的方法。 文档字符串将其描述为：Load and broadcast multiple scalar values with different operations Tests scalar broadcast with multiplication, addition, and division 它定义了内部 visitor `evt_multiple_scalars`；被 trace 的关键步骤是 `D = (accum * alpha + C * beta) / gamma; return D`。 `example_inputs` 为 `accum`、`C`、`alpha`、`beta`、`gamma`、`D` 准备运行时占位符或实际数值。 `input_keys=['C', 'alpha', 'beta', 'gamma']` 与 `result_keys=['D']` 告诉启动器哪些值由外部提供、哪些输出需要验证。 外层控制流会遍历 `self.get_problem_sizes(8)`，以覆盖多种形状或配置。 该方法最终会通过 `EVTTestBed.verify` 执行，它会把生成的 CUTLASS 路径与基于 PyTorch 的参考实现进行比较。

### Lines 586-608
```python
    def test_tensor_load_fused_activation(self):
        """
        Load auxiliary tensor with activation-like operations
        Tests tensor load with max operation (ReLU-like pattern)
        """
        def evt_tensor_load_activation(accum, C, bias_matrix, threshold):
            temp = accum + C + bias_matrix
            D = maximum(temp, threshold)
            return D

        for m, n, k, l in self.get_problem_sizes(8):
            example_inputs = {
                "accum": self.fake_tensor(self.element, (l, m, n)),
                "C": self.fake_tensor(self.element, (l, m, n)),
                "bias_matrix": self.fake_tensor(self.element, (m, n)),
                "threshold": 0.0,
                "D": self.fake_tensor(self.element, (l, m, n)),
            }

            launcher = EVTTestBed(self.element, evt_tensor_load_activation, example_inputs)
            input_keys = ["C", "bias_matrix", "threshold"]
            result_keys = ["D"]
            launcher.verify((m, n, k), input_keys, result_keys, l)
```
**EN:** `test_tensor_load_fused_activation(self)` is a method inside `TestEVTLoad`. The docstring describes it as: Load auxiliary tensor with activation-like operations Tests tensor load with max operation (ReLU-like pattern) It defines the inner visitor `evt_tensor_load_activation`; the key traced steps are `temp = accum + C + bias_matrix; D = maximum(temp, threshold); return D`. `example_inputs` prepares runtime placeholders/values for `accum`, `C`, `bias_matrix`, `threshold`, `D`. `input_keys=['C', 'bias_matrix', 'threshold']` and `result_keys=['D']` tell the launcher which values are supplied and which outputs must be validated. The surrounding control flow iterates over `self.get_problem_sizes(8)` to cover multiple shapes/configurations. The method ultimately routes execution through `EVTTestBed.verify`, which compares the generated CUTLASS path against a PyTorch-based reference.
**CN:** `test_tensor_load_fused_activation(self)` 是一个位于 `TestEVTLoad` 内部的方法。 文档字符串将其描述为：Load auxiliary tensor with activation-like operations Tests tensor load with max operation (ReLU-like pattern) 它定义了内部 visitor `evt_tensor_load_activation`；被 trace 的关键步骤是 `temp = accum + C + bias_matrix; D = maximum(temp, threshold); return D`。 `example_inputs` 为 `accum`、`C`、`bias_matrix`、`threshold`、`D` 准备运行时占位符或实际数值。 `input_keys=['C', 'bias_matrix', 'threshold']` 与 `result_keys=['D']` 告诉启动器哪些值由外部提供、哪些输出需要验证。 外层控制流会遍历 `self.get_problem_sizes(8)`，以覆盖多种形状或配置。 该方法最终会通过 `EVTTestBed.verify` 执行，它会把生成的 CUTLASS 路径与基于 PyTorch 的参考实现进行比较。

### Lines 610-634
```python
    def test_scalar_broadcast_complex_expression(self):
        """
        Scalar broadcast in complex nested expression
        Tests scalar values used in multiple sub-expressions
        """
        def evt_scalar_complex(accum, C, scale1, scale2, offset):
            temp1 = accum * scale1
            temp2 = C * scale2
            D = (temp1 + temp2) * offset
            return D

        for m, n, k, l in self.get_problem_sizes(8):
            example_inputs = {
                "accum": self.fake_tensor(self.element, (l, m, n)),
                "C": self.fake_tensor(self.element, (l, m, n)),
                "scale1": 0.8,
                "scale2": 1.2,
                "offset": 0.5,
                "D": self.fake_tensor(self.element, (l, m, n)),
            }

            launcher = EVTTestBed(self.element, evt_scalar_complex, example_inputs)
            input_keys = ["C", "scale1", "scale2", "offset"]
            result_keys = ["D"]
            launcher.verify((m, n, k), input_keys, result_keys, l)
```
**EN:** `test_scalar_broadcast_complex_expression(self)` is a method inside `TestEVTLoad`. The docstring describes it as: Scalar broadcast in complex nested expression Tests scalar values used in multiple sub-expressions It defines the inner visitor `evt_scalar_complex`; the key traced steps are `temp1 = accum * scale1; temp2 = C * scale2; D = (temp1 + temp2) * offset; return D`. `example_inputs` prepares runtime placeholders/values for `accum`, `C`, `scale1`, `scale2`, `offset`, `D`. `input_keys=['C', 'scale1', 'scale2', 'offset']` and `result_keys=['D']` tell the launcher which values are supplied and which outputs must be validated. The surrounding control flow iterates over `self.get_problem_sizes(8)` to cover multiple shapes/configurations. The method ultimately routes execution through `EVTTestBed.verify`, which compares the generated CUTLASS path against a PyTorch-based reference.
**CN:** `test_scalar_broadcast_complex_expression(self)` 是一个位于 `TestEVTLoad` 内部的方法。 文档字符串将其描述为：Scalar broadcast in complex nested expression Tests scalar values used in multiple sub-expressions 它定义了内部 visitor `evt_scalar_complex`；被 trace 的关键步骤是 `temp1 = accum * scale1; temp2 = C * scale2; D = (temp1 + temp2) * offset; return D`。 `example_inputs` 为 `accum`、`C`、`scale1`、`scale2`、`offset`、`D` 准备运行时占位符或实际数值。 `input_keys=['C', 'scale1', 'scale2', 'offset']` 与 `result_keys=['D']` 告诉启动器哪些值由外部提供、哪些输出需要验证。 外层控制流会遍历 `self.get_problem_sizes(8)`，以覆盖多种形状或配置。 该方法最终会通过 `EVTTestBed.verify` 执行，它会把生成的 CUTLASS 路径与基于 PyTorch 的参考实现进行比较。

### Lines 636-637
```python
if __name__ == '__main__':
    unittest.main()
```
**EN:** This conditional block is guarded by `__name__ == '__main__'`. It acts as the executable entry point and runs `unittest.main()` when the file is invoked as a script.
**CN:** 该条件块由 `__name__ == '__main__'` 控制。 当文件以脚本方式运行时，它就是可执行入口，并会执行 `unittest.main()`。

## Key Concepts / 关键概念
- **EN:** Load patterns: full tensor, batched tensor, row broadcast, column broadcast, and scalar broadcast.
  **CN:** 加载模式：整张量、批量张量、行广播、列广播和标量广播。
- **EN:** Combinations of broadcasts with arithmetic chains and activation-like operations.
  **CN:** 将多种广播与算术链、类激活操作组合起来测试。
- **EN:** Mixed element types such as the base element type plus `np.float32` auxiliaries.
  **CN:** 混合元素类型，例如基础元素类型配合 `np.float32` 辅助张量。
- **EN:** Regression-oriented “corrected” cases that encode supported expression forms.
  **CN:** 面向回归的“corrected”用例，用于固定已验证可支持的表达式形式。

## Dependencies / 依赖关系
- `import logging`
  - **EN:** Participates in this module’s runtime support and test infrastructure.
  - **CN:** 参与该模块的运行时支持与测试基础设施。
- `import unittest`
  - **EN:** Supplies the Python unit-test framework and result/runner abstractions.
  - **CN:** 提供 Python 单元测试框架以及结果/运行器抽象。
- `import cutlass_cppgen`
  - **EN:** Provides the main CUTLASS Python API used to build plans, trace visitors, or emit code.
  - **CN:** 提供主要的 CUTLASS Python API，用于构建 plan、trace visitor 或发射代码。
- `from cutlass_cppgen.backend import *`
  - **EN:** Brings in backend helpers such as architecture queries, data types, and convenience symbols used by the tests.
  - **CN:** 引入后端辅助内容，如架构查询、数据类型以及测试中使用的便捷符号。
- `from cutlass_cppgen.epilogue import *`
  - **EN:** Imports the epilogue DSL operators used inside EVT expressions, such as activation, reduction, and layout helpers.
  - **CN:** 导入 EVT 表达式中使用的 epilogue DSL 算子，例如激活、归约和布局辅助函数。
- `from utils.evt_testbed import EVTTestBed, EVTTestCaseBase`
  - **EN:** Imports the shared EVT launcher and testcase base used by the test modules.
  - **CN:** 导入测试模块复用的 EVT 启动器与 testcase 基类。
- `device_cc()` runtime check
  - **EN:** Controls architecture-specific skips so unsupported devices do not execute invalid cases.
  - **CN:** 控制架构相关跳过逻辑，避免不支持的设备执行无效用例。
- `utils.evt_testbed.EVTTestBed`
  - **EN:** Shared harness that compiles the CUTLASS plan, runs the kernel, and checks the reference output.
  - **CN:** 共享测试支架：负责编译 CUTLASS plan、运行内核并检查参考输出。
