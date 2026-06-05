# evt_mixed_xe_sm80_90.py — Code Analysis / 代码分析

## Source / 来源
- Path / 路径: `test/python/cutlass/evt/evt_mixed_xe_sm80_90.py`
- **EN:** Exercises mixed epilogue DAGs that combine aux stores, reductions, reused intermediates, Stream-K swizzling, and batch/no-batch variants.
- **CN:** 测试混合 epilogue DAG：同时包含辅助输出、归约、重复使用的中间值、Stream-K swizzle，以及带批量/不带批量两类场景。

## Line-by-Line Analysis / 逐行分析

### Lines 33-47
```python
"""
Unittest for mixed types of nodes in SM90
"""

import logging
import unittest

import cutlass_cppgen
from cutlass_cppgen.backend import *
from cutlass_cppgen.epilogue import *
from cutlass_cppgen.swizzle import ThreadblockSwizzleStreamK

from utils.evt_testbed import EVTTestBed, EVTTestCaseBase

cutlass_cppgen.set_log_level(logging.WARNING)
```
**EN:** This opening block establishes the module context. The module docstring states the purpose: Unittest for mixed types of nodes in SM90 It imports the libraries needed for code generation, testing, and tensor/runtime support. `cutlass_cppgen.set_log_level(logging.WARNING)` reduces generator-side logging noise during test runs.
**CN:** 这一开头代码块建立了模块的上下文。 模块文档字符串直接说明了用途：Unittest for mixed types of nodes in SM90 它导入了代码生成、测试以及张量/运行时支持所需的库。 `cutlass_cppgen.set_log_level(logging.WARNING)` 会降低测试运行期间生成器侧的日志噪声。

### Lines 50-52
```python
@unittest.skipIf(device_cc() not in [12, 20, 80, 86, 89, 90], "This unittest is only supported on CC [12, 20, 80, 86, 89, 90]")
class TestEVTMixed(EVTTestCaseBase):
```
**EN:** `TestEVTMixed` is the main class introduced in this block. It inherits from `EVTTestCaseBase`, so the class reuses shared helpers or unittest behavior. The decorator(s) `unittest.skipIf(device_cc() not in [12, 20, 80, 86, 89, 90], 'This unittest is only supported on CC [12, 20, 80, 86, 89, 90]')` gate execution to supported environments. That base class supplies common element types, fake tensor descriptors, and standard problem-size sweeps for all contained tests.
**CN:** `TestEVTMixed` 是此处引入的主要类。 它继承自 `EVTTestCaseBase`，因此会复用共享辅助逻辑或 unittest 行为。 装饰器 `unittest.skipIf(device_cc() not in [12, 20, 80, 86, 89, 90], 'This unittest is only supported on CC [12, 20, 80, 86, 89, 90]')` 用来把执行限制在受支持的环境中。 该基类为内部所有测试提供统一的元素类型、伪张量描述符以及标准问题尺寸扫描逻辑。

### Lines 53-72
```python
    def test_same_variable_used_multiple_times(self):
        """
        The same variable z0 is used multiple times
        """
        def evt_aux_store(accum):
            z0 = relu(accum)
            D = z0 + z0
            return z0, D

        for m, n, k, l in self.get_problem_sizes(8):
            example_inputs = {
                "accum": self.fake_tensor(self.element, (l, m, n)),
                "D": self.fake_tensor(self.element, (l, m, n)),
                "z0": self.fake_tensor(self.element, (l, m, n)),
            }

            launcher = EVTTestBed(self.element, evt_aux_store, example_inputs)
            input_keys = ["accum"]
            result_keys = ["z0", "D"]
            launcher.verify((m, n, k), input_keys, result_keys, l)
```
**EN:** `test_same_variable_used_multiple_times(self)` is a method inside `TestEVTMixed`. The docstring describes it as: The same variable z0 is used multiple times It defines the inner visitor `evt_aux_store`; the key traced steps are `z0 = relu(accum); D = z0 + z0; return (z0, D)`. `example_inputs` prepares runtime placeholders/values for `accum`, `D`, `z0`. `input_keys=['accum']` and `result_keys=['z0', 'D']` tell the launcher which values are supplied and which outputs must be validated. The surrounding control flow iterates over `self.get_problem_sizes(8)` to cover multiple shapes/configurations. The method ultimately routes execution through `EVTTestBed.verify`, which compares the generated CUTLASS path against a PyTorch-based reference.
**CN:** `test_same_variable_used_multiple_times(self)` 是一个位于 `TestEVTMixed` 内部的方法。 文档字符串将其描述为：The same variable z0 is used multiple times 它定义了内部 visitor `evt_aux_store`；被 trace 的关键步骤是 `z0 = relu(accum); D = z0 + z0; return (z0, D)`。 `example_inputs` 为 `accum`、`D`、`z0` 准备运行时占位符或实际数值。 `input_keys=['accum']` 与 `result_keys=['z0', 'D']` 告诉启动器哪些值由外部提供、哪些输出需要验证。 外层控制流会遍历 `self.get_problem_sizes(8)`，以覆盖多种形状或配置。 该方法最终会通过 `EVTTestBed.verify` 执行，它会把生成的 CUTLASS 路径与基于 PyTorch 的参考实现进行比较。

### Lines 74-95
```python
    def test_no_lca(self):
        """
        The same variable z0 is used multiple times
        """
        def evt_no_lca(accum, bias):
            E = relu(accum)
            F = E + bias
            tmp_2 = E + 2
            D = tmp_2 + E
            return D

        for m, n, k, l in self.get_problem_sizes(8):
            example_inputs = {
                "accum": self.fake_tensor(self.element, (l, m, n)),
                "D": self.fake_tensor(self.element, (l, m, n)),
                "bias": self.fake_tensor(self.element, (m,1), stride=(1,0)),
            }

            launcher = EVTTestBed(self.element, evt_no_lca, example_inputs)
            input_keys = ["accum", "bias"]
            result_keys = ["D"]
            launcher.verify((m, n, k), input_keys, result_keys, l)
```
**EN:** `test_no_lca(self)` is a method inside `TestEVTMixed`. The docstring describes it as: The same variable z0 is used multiple times It defines the inner visitor `evt_no_lca`; the key traced steps are `E = relu(accum); F = E + bias; tmp_2 = E + 2; D = tmp_2 + E`. `example_inputs` prepares runtime placeholders/values for `accum`, `D`, `bias`. `input_keys=['accum', 'bias']` and `result_keys=['D']` tell the launcher which values are supplied and which outputs must be validated. The surrounding control flow iterates over `self.get_problem_sizes(8)` to cover multiple shapes/configurations. The method ultimately routes execution through `EVTTestBed.verify`, which compares the generated CUTLASS path against a PyTorch-based reference.
**CN:** `test_no_lca(self)` 是一个位于 `TestEVTMixed` 内部的方法。 文档字符串将其描述为：The same variable z0 is used multiple times 它定义了内部 visitor `evt_no_lca`；被 trace 的关键步骤是 `E = relu(accum); F = E + bias; tmp_2 = E + 2; D = tmp_2 + E`。 `example_inputs` 为 `accum`、`D`、`bias` 准备运行时占位符或实际数值。 `input_keys=['accum', 'bias']` 与 `result_keys=['D']` 告诉启动器哪些值由外部提供、哪些输出需要验证。 外层控制流会遍历 `self.get_problem_sizes(8)`，以覆盖多种形状或配置。 该方法最终会通过 `EVTTestBed.verify` 执行，它会把生成的 CUTLASS 路径与基于 PyTorch 的参考实现进行比较。

### Lines 97-130
```python
    def test_mixed_dag(self):
        def evt_mixed_dag(accum, alpha, C, beta, aux, cbias, rbias):
            F = alpha * accum + (beta * C + aux)
            F_row_max = max(F, dim=[0, 1])
            E = relu(F + 1) + cbias + rbias
            E_col_max = max(E, dim=[0, 2])
            D = E + F
            return D, F, F_row_max, E_col_max

        if device_cc() == 80:
            alignments = [2, 4, 8]
        else:
            # Sm90 EVT currently only supports 128-bit alignment
            alignments = [8,]
        for align in alignments:
            for m, n, k, l in self.get_problem_sizes(align):
                example_inputs = {
                    "accum": self.fake_tensor(self.element, (l, m, n)),
                    "alpha": 1.0,
                    "C": self.fake_tensor(self.element, (l, m, n)),
                    "beta": 1.0,
                    "aux": self.fake_tensor(self.element, (l, m, n)),
                    "cbias": self.fake_tensor(self.element, (m, 1)),
                    "rbias": self.fake_tensor(self.element, (n,)),
                    "D": self.fake_tensor(self.element, (l, m, n)),
                    "F": self.fake_tensor(self.element, (l, m, n)),
                    "F_row_max": self.fake_tensor(DataType.f32, (n,)),
                    "E_col_max": self.fake_tensor(DataType.f32, (m, 1))
                }

                launcher = EVTTestBed(self.element, evt_mixed_dag, example_inputs)
                input_keys = ["alpha", "C", "beta", "aux", "cbias", "rbias"]
                result_keys = ["D", "F", "F_row_max", "E_col_max"]
                launcher.verify((m, n, k), input_keys, result_keys, l)
```
**EN:** `test_mixed_dag(self)` is a method inside `TestEVTMixed`. It defines the inner visitor `evt_mixed_dag`; the key traced steps are `F = alpha * accum + (beta * C + aux); F_row_max = max(F, dim=[0, 1]); E = relu(F + 1) + cbias + rbias; E_col_max = max(E, dim=[0, 2])`. `example_inputs` prepares runtime placeholders/values for `accum`, `alpha`, `C`, `beta`, `aux`, `cbias`, `rbias`, `D`.... `input_keys=['alpha', 'C', 'beta', 'aux', 'cbias', 'rbias']` and `result_keys=['D', 'F', 'F_row_max', 'E_col_max']` tell the launcher which values are supplied and which outputs must be validated. The surrounding control flow iterates over `alignments` to cover multiple shapes/configurations. The method ultimately routes execution through `EVTTestBed.verify`, which compares the generated CUTLASS path against a PyTorch-based reference.
**CN:** `test_mixed_dag(self)` 是一个位于 `TestEVTMixed` 内部的方法。 它定义了内部 visitor `evt_mixed_dag`；被 trace 的关键步骤是 `F = alpha * accum + (beta * C + aux); F_row_max = max(F, dim=[0, 1]); E = relu(F + 1) + cbias + rbias; E_col_max = max(E, dim=[0, 2])`。 `example_inputs` 为 `accum`、`alpha`、`C`、`beta`、`aux`、`cbias`、`rbias`、`D` 等 准备运行时占位符或实际数值。 `input_keys=['alpha', 'C', 'beta', 'aux', 'cbias', 'rbias']` 与 `result_keys=['D', 'F', 'F_row_max', 'E_col_max']` 告诉启动器哪些值由外部提供、哪些输出需要验证。 外层控制流会遍历 `alignments`，以覆盖多种形状或配置。 该方法最终会通过 `EVTTestBed.verify` 执行，它会把生成的 CUTLASS 路径与基于 PyTorch 的参考实现进行比较。

### Lines 132-160
```python
    @unittest.skipIf(device_cc() not in [80, 89], "This unittest is for cc 80 and 89 only")
    def test_mixed_dag_float(self):
        def evt_mixed_dag(accum, alpha, C, beta, aux, cbias, rbias):
            F = alpha * accum + (beta * C + aux)
            F_row_max = max(F, dim=[0, 1])
            E = relu(F + 1) + cbias + rbias
            E_col_max = max(E, dim=[0, 2])
            D = E + F
            return D, F, F_row_max, E_col_max

        for align in [3, 2, 4]:
            for m, n, k, l in self.get_problem_sizes(align):
                example_inputs = {
                    "accum": self.fake_tensor(np.float32, (l, m, n)),
                    "alpha": 1.0,
                    "C": self.fake_tensor(np.float32, (l, m, n)),
                    "beta": 1.0,
                    "aux": self.fake_tensor(np.float32, (l, m, n)),
                    "cbias": self.fake_tensor(np.float32, (m, 1)),
                    "rbias": self.fake_tensor(np.float32, (n,)),
                    "D": self.fake_tensor(np.float32, (l, m, n)),
                    "F": self.fake_tensor(np.float32, (l, m, n)),
                    "F_row_max": self.fake_tensor(np.float32, (n,)),
                    "E_col_max": self.fake_tensor(np.float32, (m, 1))
                }
                launcher = EVTTestBed(DataType.f32, evt_mixed_dag, example_inputs)
                input_keys = ["alpha", "C", "beta", "aux", "cbias", "rbias"]
                result_keys = ["D", "F", "F_row_max", "E_col_max"]
                launcher.verify((m, n, k), input_keys, result_keys, l)
```
**EN:** `test_mixed_dag_float(self)` is a method inside `TestEVTMixed`. It defines the inner visitor `evt_mixed_dag`; the key traced steps are `F = alpha * accum + (beta * C + aux); F_row_max = max(F, dim=[0, 1]); E = relu(F + 1) + cbias + rbias; E_col_max = max(E, dim=[0, 2])`. `example_inputs` prepares runtime placeholders/values for `accum`, `alpha`, `C`, `beta`, `aux`, `cbias`, `rbias`, `D`.... `input_keys=['alpha', 'C', 'beta', 'aux', 'cbias', 'rbias']` and `result_keys=['D', 'F', 'F_row_max', 'E_col_max']` tell the launcher which values are supplied and which outputs must be validated. The surrounding control flow iterates over `[3, 2, 4]` to cover multiple shapes/configurations. The method ultimately routes execution through `EVTTestBed.verify`, which compares the generated CUTLASS path against a PyTorch-based reference.
**CN:** `test_mixed_dag_float(self)` 是一个位于 `TestEVTMixed` 内部的方法。 它定义了内部 visitor `evt_mixed_dag`；被 trace 的关键步骤是 `F = alpha * accum + (beta * C + aux); F_row_max = max(F, dim=[0, 1]); E = relu(F + 1) + cbias + rbias; E_col_max = max(E, dim=[0, 2])`。 `example_inputs` 为 `accum`、`alpha`、`C`、`beta`、`aux`、`cbias`、`rbias`、`D` 等 准备运行时占位符或实际数值。 `input_keys=['alpha', 'C', 'beta', 'aux', 'cbias', 'rbias']` 与 `result_keys=['D', 'F', 'F_row_max', 'E_col_max']` 告诉启动器哪些值由外部提供、哪些输出需要验证。 外层控制流会遍历 `[3, 2, 4]`，以覆盖多种形状或配置。 该方法最终会通过 `EVTTestBed.verify` 执行，它会把生成的 CUTLASS 路径与基于 PyTorch 的参考实现进行比较。

### Lines 162-190
```python
    @unittest.skipIf(device_cc() not in [80, 89], "This unittest is for cc 80 and 89 only")
    def test_mixed_dag_stage2(self):
        def evt_mixed_dag(accum, alpha, C, beta, aux, cbias, rbias):
            F = alpha * accum + (beta * C + aux)
            F_row_max = max(F, dim=[0, 1])
            E = relu(F + 1) + cbias + rbias
            E_col_max = max(E, dim=[0, 2])
            D = E + F
            return D, F, F_row_max, E_col_max

        for m, n, k, l in self.get_problem_sizes(8):
            example_inputs = {
                "accum": self.fake_tensor(self.element, (l, m, n)),
                "alpha": 1.0,
                "C": self.fake_tensor(self.element, (l, m, n)),
                "beta": 1.0,
                "aux": self.fake_tensor(self.element, (l, m, n)),
                "cbias": self.fake_tensor(self.element, (m, 1)),
                "rbias": self.fake_tensor(self.element, (n,)),
                "D": self.fake_tensor(self.element, (l, m, n)),
                "F": self.fake_tensor(self.element, (l, m, n)),
                "F_row_max": self.fake_tensor(DataType.f32, (n,)),
                "E_col_max": self.fake_tensor(DataType.f32, (m, 1))
            }

            launcher = EVTTestBed(self.element, evt_mixed_dag, example_inputs, epilogue_stages=2)
            input_keys = ["alpha", "C", "beta", "aux", "cbias", "rbias"]
            result_keys = ["D", "F", "F_row_max", "E_col_max"]
            launcher.verify((m, n, k), input_keys, result_keys, l)
```
**EN:** `test_mixed_dag_stage2(self)` is a method inside `TestEVTMixed`. It defines the inner visitor `evt_mixed_dag`; the key traced steps are `F = alpha * accum + (beta * C + aux); F_row_max = max(F, dim=[0, 1]); E = relu(F + 1) + cbias + rbias; E_col_max = max(E, dim=[0, 2])`. `example_inputs` prepares runtime placeholders/values for `accum`, `alpha`, `C`, `beta`, `aux`, `cbias`, `rbias`, `D`.... `input_keys=['alpha', 'C', 'beta', 'aux', 'cbias', 'rbias']` and `result_keys=['D', 'F', 'F_row_max', 'E_col_max']` tell the launcher which values are supplied and which outputs must be validated. The surrounding control flow iterates over `self.get_problem_sizes(8)` to cover multiple shapes/configurations. The method ultimately routes execution through `EVTTestBed.verify`, which compares the generated CUTLASS path against a PyTorch-based reference.
**CN:** `test_mixed_dag_stage2(self)` 是一个位于 `TestEVTMixed` 内部的方法。 它定义了内部 visitor `evt_mixed_dag`；被 trace 的关键步骤是 `F = alpha * accum + (beta * C + aux); F_row_max = max(F, dim=[0, 1]); E = relu(F + 1) + cbias + rbias; E_col_max = max(E, dim=[0, 2])`。 `example_inputs` 为 `accum`、`alpha`、`C`、`beta`、`aux`、`cbias`、`rbias`、`D` 等 准备运行时占位符或实际数值。 `input_keys=['alpha', 'C', 'beta', 'aux', 'cbias', 'rbias']` 与 `result_keys=['D', 'F', 'F_row_max', 'E_col_max']` 告诉启动器哪些值由外部提供、哪些输出需要验证。 外层控制流会遍历 `self.get_problem_sizes(8)`，以覆盖多种形状或配置。 该方法最终会通过 `EVTTestBed.verify` 执行，它会把生成的 CUTLASS 路径与基于 PyTorch 的参考实现进行比较。

### Lines 192-225
```python
    @unittest.skipIf(device_cc() not in [80, 89], "This unittest is for cc 80 and 89 only")
    def test_mixed_dag_partition_k(self):
        def evt_mixed_dag(accum, alpha, C, beta, aux, cbias, rbias):
            F = alpha * accum + (beta * C + aux)
            F_row_max = max(F, dim=[0, 1])
            E = relu(F + 1) + cbias + rbias
            E_col_max = max(E, dim=[0, 2])
            D = E + F
            return D, F, F_row_max, E_col_max

        for m, n, k, l in self.get_problem_sizes(8):
            example_inputs = {
                "accum": self.fake_tensor(self.element, (l, m, n)),
                "alpha": 1.0,
                "C": self.fake_tensor(self.element, (l, m, n)),
                "beta": 1.0,
                "aux": self.fake_tensor(self.element, (l, m, n)),
                "cbias": self.fake_tensor(self.element, (m, 1)),
                "rbias": self.fake_tensor(self.element, (n,)),
                "D": self.fake_tensor(self.element, (l, m, n)),
                "F": self.fake_tensor(self.element, (l, m, n)),
                "F_row_max": self.fake_tensor(DataType.f32, (n,)),
                "E_col_max": self.fake_tensor(DataType.f32, (m, 1))
            }

            tile_description = {
                "threadblock_shape": [128, 128, 64],
                "warp_count": [2, 2, 2]
            }

            launcher = EVTTestBed(self.element, evt_mixed_dag, example_inputs, tile_description=tile_description, epilogue_stages=2)
            input_keys = ["alpha", "C", "beta", "aux", "cbias", "rbias"]
            result_keys = ["D", "F", "F_row_max", "E_col_max"]
            launcher.verify((m, n, k), input_keys, result_keys, l)
```
**EN:** `test_mixed_dag_partition_k(self)` is a method inside `TestEVTMixed`. It defines the inner visitor `evt_mixed_dag`; the key traced steps are `F = alpha * accum + (beta * C + aux); F_row_max = max(F, dim=[0, 1]); E = relu(F + 1) + cbias + rbias; E_col_max = max(E, dim=[0, 2])`. `example_inputs` prepares runtime placeholders/values for `accum`, `alpha`, `C`, `beta`, `aux`, `cbias`, `rbias`, `D`.... `input_keys=['alpha', 'C', 'beta', 'aux', 'cbias', 'rbias']` and `result_keys=['D', 'F', 'F_row_max', 'E_col_max']` tell the launcher which values are supplied and which outputs must be validated. The surrounding control flow iterates over `self.get_problem_sizes(8)` to cover multiple shapes/configurations. The method ultimately routes execution through `EVTTestBed.verify`, which compares the generated CUTLASS path against a PyTorch-based reference.
**CN:** `test_mixed_dag_partition_k(self)` 是一个位于 `TestEVTMixed` 内部的方法。 它定义了内部 visitor `evt_mixed_dag`；被 trace 的关键步骤是 `F = alpha * accum + (beta * C + aux); F_row_max = max(F, dim=[0, 1]); E = relu(F + 1) + cbias + rbias; E_col_max = max(E, dim=[0, 2])`。 `example_inputs` 为 `accum`、`alpha`、`C`、`beta`、`aux`、`cbias`、`rbias`、`D` 等 准备运行时占位符或实际数值。 `input_keys=['alpha', 'C', 'beta', 'aux', 'cbias', 'rbias']` 与 `result_keys=['D', 'F', 'F_row_max', 'E_col_max']` 告诉启动器哪些值由外部提供、哪些输出需要验证。 外层控制流会遍历 `self.get_problem_sizes(8)`，以覆盖多种形状或配置。 该方法最终会通过 `EVTTestBed.verify` 执行，它会把生成的 CUTLASS 路径与基于 PyTorch 的参考实现进行比较。

### Lines 227-287
```python
    @unittest.skipIf(device_cc() not in [80, 89], "This unittest is for cc 80 and 89 only")
    def test_mixed_dag_stream_k(self):
        def evt_mixed_dag(accum, alpha, C, beta, aux, cbias, rbias):
            F = alpha * accum + (beta * C + aux)
            F_row_max = max(F, dim=[0, 1])
            E = relu(F + 1) + cbias + rbias
            E_col_max = max(E, dim=[0, 2])
            D = E + F
            return D, F, F_row_max, E_col_max

        # High per-sm occupancy tile_description
        tile_description = {
            "threadblock_shape": [128, 128, 32],
            "warp_count": [2, 2, 1],
            "stages": 3
        }
        tds = [None, tile_description]
        for td in tds:
            for m, n, k, l in self.get_problem_sizes(8, k=960, batch_count=[1, 3]):
                if l == 1:
                    example_inputs = {
                        "accum": self.fake_tensor(self.element, (m, n)),
                        "alpha": 1.0,
                        "C": self.fake_tensor(self.element, (m, n)),
                        "beta": 1.0,
                        "aux": self.fake_tensor(self.element, (m, n)),
                        "cbias": self.fake_tensor(self.element, (m, 1)),
                        "rbias": self.fake_tensor(self.element, (n,)),
                        "D": self.fake_tensor(self.element, (m, n)),
                        "F": self.fake_tensor(self.element, (m, n)),
                        "F_row_max": self.fake_tensor(DataType.f32, (n,)),
                        "E_col_max": self.fake_tensor(DataType.f32, (m, 1))
                    }
                else:
                    example_inputs = {
                        "accum": self.fake_tensor(self.element, (l, m, n)),
                        "alpha": 1.0,
                        "C": self.fake_tensor(self.element, (l, m, n)),
                        "beta": 1.0,
                        "aux": self.fake_tensor(self.element, (l, m, n)),
                        "cbias": self.fake_tensor(self.element, (m, 1)),
                        "rbias": self.fake_tensor(self.element, (n,)),
                        "D": self.fake_tensor(self.element, (l, m, n)),
                        "F": self.fake_tensor(self.element, (l, m, n)),
                        "F_row_max": self.fake_tensor(DataType.f32, (n,)),
                        "E_col_max": self.fake_tensor(DataType.f32, (m, 1))
                    }

                if td is not None:
                    launcher = EVTTestBed(
                        self.element, evt_mixed_dag, example_inputs,
                        tile_description=td,
                        swizzling_functor=ThreadblockSwizzleStreamK, backend="torch")
                else:
                    launcher = EVTTestBed(
                        self.element, evt_mixed_dag, example_inputs,
                        swizzling_functor=ThreadblockSwizzleStreamK, backend="torch")

                input_keys = ["alpha", "C", "beta", "aux", "cbias", "rbias"]
                result_keys = ["D", "F", "F_row_max", "E_col_max"]
                launcher.verify((m, n, k), input_keys, result_keys, l)
```
**EN:** `test_mixed_dag_stream_k(self)` is a method inside `TestEVTMixed`. It defines the inner visitor `evt_mixed_dag`; the key traced steps are `F = alpha * accum + (beta * C + aux); F_row_max = max(F, dim=[0, 1]); E = relu(F + 1) + cbias + rbias; E_col_max = max(E, dim=[0, 2])`. `example_inputs` prepares runtime placeholders/values for `accum`, `alpha`, `C`, `beta`, `aux`, `cbias`, `rbias`, `D`.... `input_keys=['alpha', 'C', 'beta', 'aux', 'cbias', 'rbias']` and `result_keys=['D', 'F', 'F_row_max', 'E_col_max']` tell the launcher which values are supplied and which outputs must be validated. The surrounding control flow iterates over `tds` to cover multiple shapes/configurations. The method ultimately routes execution through `EVTTestBed.verify`, which compares the generated CUTLASS path against a PyTorch-based reference.
**CN:** `test_mixed_dag_stream_k(self)` 是一个位于 `TestEVTMixed` 内部的方法。 它定义了内部 visitor `evt_mixed_dag`；被 trace 的关键步骤是 `F = alpha * accum + (beta * C + aux); F_row_max = max(F, dim=[0, 1]); E = relu(F + 1) + cbias + rbias; E_col_max = max(E, dim=[0, 2])`。 `example_inputs` 为 `accum`、`alpha`、`C`、`beta`、`aux`、`cbias`、`rbias`、`D` 等 准备运行时占位符或实际数值。 `input_keys=['alpha', 'C', 'beta', 'aux', 'cbias', 'rbias']` 与 `result_keys=['D', 'F', 'F_row_max', 'E_col_max']` 告诉启动器哪些值由外部提供、哪些输出需要验证。 外层控制流会遍历 `tds`，以覆盖多种形状或配置。 该方法最终会通过 `EVTTestBed.verify` 执行，它会把生成的 CUTLASS 路径与基于 PyTorch 的参考实现进行比较。

### Lines 289-316
```python
    def test_mixed_dag_no_batch(self):
        def evt_mixed_dag_no_batch(accum, alpha, C, beta, aux, cbias, rbias):
            F = alpha * accum + (beta * C + aux)
            F_row_max = max(F, dim=[0, 1])
            E = relu(F + 1) + cbias + rbias
            E_col_max = max(E, dim=[0, 2])
            D = E + F
            return D, F, F_row_max, E_col_max

        for m, n, k, _ in self.get_problem_sizes(8):
            example_inputs = {
                "accum": self.fake_tensor(self.element, (m, n)),
                "alpha": 1.0,
                "C": self.fake_tensor(self.element, (m, n)),
                "beta": 1.0,
                "aux": self.fake_tensor(self.element, (m, n)),
                "cbias": self.fake_tensor(self.element, (m, 1)),
                "rbias": self.fake_tensor(self.element, (n,)),
                "D": self.fake_tensor(self.element, (m, n)),
                "F": self.fake_tensor(self.element, (m, n)),
                "F_row_max": self.fake_tensor(DataType.f32, (n,)),
                "E_col_max": self.fake_tensor(DataType.f32, (m, 1))
            }

            launcher = EVTTestBed(self.element, evt_mixed_dag_no_batch, example_inputs)
            input_keys = ["alpha", "C", "beta", "aux", "cbias", "rbias"]
            result_keys = ["D", "F", "F_row_max", "E_col_max"]
            launcher.verify((m, n, k), input_keys, result_keys, 1)
```
**EN:** `test_mixed_dag_no_batch(self)` is a method inside `TestEVTMixed`. It defines the inner visitor `evt_mixed_dag_no_batch`; the key traced steps are `F = alpha * accum + (beta * C + aux); F_row_max = max(F, dim=[0, 1]); E = relu(F + 1) + cbias + rbias; E_col_max = max(E, dim=[0, 2])`. `example_inputs` prepares runtime placeholders/values for `accum`, `alpha`, `C`, `beta`, `aux`, `cbias`, `rbias`, `D`.... `input_keys=['alpha', 'C', 'beta', 'aux', 'cbias', 'rbias']` and `result_keys=['D', 'F', 'F_row_max', 'E_col_max']` tell the launcher which values are supplied and which outputs must be validated. The surrounding control flow iterates over `self.get_problem_sizes(8)` to cover multiple shapes/configurations. The method ultimately routes execution through `EVTTestBed.verify`, which compares the generated CUTLASS path against a PyTorch-based reference.
**CN:** `test_mixed_dag_no_batch(self)` 是一个位于 `TestEVTMixed` 内部的方法。 它定义了内部 visitor `evt_mixed_dag_no_batch`；被 trace 的关键步骤是 `F = alpha * accum + (beta * C + aux); F_row_max = max(F, dim=[0, 1]); E = relu(F + 1) + cbias + rbias; E_col_max = max(E, dim=[0, 2])`。 `example_inputs` 为 `accum`、`alpha`、`C`、`beta`、`aux`、`cbias`、`rbias`、`D` 等 准备运行时占位符或实际数值。 `input_keys=['alpha', 'C', 'beta', 'aux', 'cbias', 'rbias']` 与 `result_keys=['D', 'F', 'F_row_max', 'E_col_max']` 告诉启动器哪些值由外部提供、哪些输出需要验证。 外层控制流会遍历 `self.get_problem_sizes(8)`，以覆盖多种形状或配置。 该方法最终会通过 `EVTTestBed.verify` 执行，它会把生成的 CUTLASS 路径与基于 PyTorch 的参考实现进行比较。

### Lines 318-319
```python
if __name__ == '__main__':
    unittest.main()
```
**EN:** This conditional block is guarded by `__name__ == '__main__'`. It acts as the executable entry point and runs `unittest.main()` when the file is invoked as a script.
**CN:** 该条件块由 `__name__ == '__main__'` 控制。 当文件以脚本方式运行时，它就是可执行入口，并会执行 `unittest.main()`。

## Key Concepts / 关键概念
- **EN:** DAGs with reused intermediates, auxiliary outputs, and reductions.
  **CN:** 包含中间值复用、辅助输出和归约的 DAG。
- **EN:** Architecture-specific alignment and backend restrictions.
  **CN:** 架构相关的对齐要求与后端限制。
- **EN:** Variants for Stream-K swizzling, partition-K style tiles, and multi-stage epilogues.
  **CN:** 覆盖 Stream-K swizzle、partition-K 风格 tile 以及多 stage epilogue 的变体。
- **EN:** Batch and non-batch execution shapes.
  **CN:** 批量与非批量执行形状。

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
- `from cutlass_cppgen.swizzle import ThreadblockSwizzleStreamK`
  - **EN:** Provides the main CUTLASS Python API used to build plans, trace visitors, or emit code.
  - **CN:** 提供主要的 CUTLASS Python API，用于构建 plan、trace visitor 或发射代码。
- `from utils.evt_testbed import EVTTestBed, EVTTestCaseBase`
  - **EN:** Imports the shared EVT launcher and testcase base used by the test modules.
  - **CN:** 导入测试模块复用的 EVT 启动器与 testcase 基类。
- `device_cc()` runtime check
  - **EN:** Controls architecture-specific skips so unsupported devices do not execute invalid cases.
  - **CN:** 控制架构相关跳过逻辑，避免不支持的设备执行无效用例。
- `utils.evt_testbed.EVTTestBed`
  - **EN:** Shared harness that compiles the CUTLASS plan, runs the kernel, and checks the reference output.
  - **CN:** 共享测试支架：负责编译 CUTLASS plan、运行内核并检查参考输出。
