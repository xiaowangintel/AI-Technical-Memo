# evt_store_xe_sm80_90.py — Code Analysis / 代码分析

## Source / 来源
- Path / 路径: `test/python/cutlass/evt/evt_store_xe_sm80_90.py`
- **EN:** Checks EVT store semantics, including invalid output reuse, auxiliary stores, and reduction outputs of several shapes.
- **CN:** 检查 EVT 的 store 语义，包括非法输出复用、辅助输出存储以及多种形状的归约输出。

## Line-by-Line Analysis / 逐行分析

### Lines 33-46
```python
"""
Unit test for store nodes in SM90
"""

import logging
import unittest

import cutlass_cppgen
from cutlass_cppgen.backend import *
from cutlass_cppgen.epilogue import *

from utils.evt_testbed import EVTTestBed, EVTTestCaseBase

cutlass_cppgen.set_log_level(logging.WARNING)
```
**EN:** This opening block establishes the module context. The module docstring states the purpose: Unit test for store nodes in SM90 It imports the libraries needed for code generation, testing, and tensor/runtime support. `cutlass_cppgen.set_log_level(logging.WARNING)` reduces generator-side logging noise during test runs.
**CN:** 这一开头代码块建立了模块的上下文。 模块文档字符串直接说明了用途：Unit test for store nodes in SM90 它导入了代码生成、测试以及张量/运行时支持所需的库。 `cutlass_cppgen.set_log_level(logging.WARNING)` 会降低测试运行期间生成器侧的日志噪声。

### Lines 49-51
```python
@unittest.skipIf(device_cc() not in [12, 20, 80, 86, 89, 90], "This unittest is only supported on CC [12, 20, 80, 86, 89, 90]")
class TestEVTStore(EVTTestCaseBase):
```
**EN:** `TestEVTStore` is the main class introduced in this block. It inherits from `EVTTestCaseBase`, so the class reuses shared helpers or unittest behavior. The decorator(s) `unittest.skipIf(device_cc() not in [12, 20, 80, 86, 89, 90], 'This unittest is only supported on CC [12, 20, 80, 86, 89, 90]')` gate execution to supported environments. That base class supplies common element types, fake tensor descriptors, and standard problem-size sweeps for all contained tests.
**CN:** `TestEVTStore` 是此处引入的主要类。 它继承自 `EVTTestCaseBase`，因此会复用共享辅助逻辑或 unittest 行为。 装饰器 `unittest.skipIf(device_cc() not in [12, 20, 80, 86, 89, 90], 'This unittest is only supported on CC [12, 20, 80, 86, 89, 90]')` 用来把执行限制在受支持的环境中。 该基类为内部所有测试提供统一的元素类型、伪张量描述符以及标准问题尺寸扫描逻辑。

### Lines 52-75
```python
    @unittest.skipIf(device_cc() not in [12, 20, 90], "This test is only supported on CC [12, 20, 90]")
    def test_invalid_store(self):
        """
        Test invalid store
        """
        def evt_invalid_store(accum):
            D = accum
            F = D + 1 # D has users, which is not allowed on SM90 or higher
            return D, F
        
        for m, n, k, l in self.get_problem_sizes(8):
            example_inputs = {
                "accum": self.fake_tensor(self.element, (l, m, n)),
                "D": self.fake_tensor(self.element, (l, m, n)),
                "F": self.fake_tensor(self.element, (l, m, n))
            }
            with self.assertRaisesRegex(
                    RuntimeError, 
                    r"On SM90 or higher, D is expected to be a output node with 0 users " 
                    r"to enable smem reuse between C and D, but got 1"
                ):
                launcher = EVTTestBed(self.element, evt_invalid_store, example_inputs)
            
            break  # Only need to test once
```
**EN:** `test_invalid_store(self)` is a method inside `TestEVTStore`. The docstring describes it as: Test invalid store It defines the inner visitor `evt_invalid_store`; the key traced steps are `D = accum; F = D + 1; return (D, F)`. `example_inputs` prepares runtime placeholders/values for `accum`, `D`, `F`. The surrounding control flow iterates over `self.get_problem_sizes(8)` to cover multiple shapes/configurations.
**CN:** `test_invalid_store(self)` 是一个位于 `TestEVTStore` 内部的方法。 文档字符串将其描述为：Test invalid store 它定义了内部 visitor `evt_invalid_store`；被 trace 的关键步骤是 `D = accum; F = D + 1; return (D, F)`。 `example_inputs` 为 `accum`、`D`、`F` 准备运行时占位符或实际数值。 外层控制流会遍历 `self.get_problem_sizes(8)`，以覆盖多种形状或配置。

### Lines 77-98
```python
    def test_aux_store(self):
        """
        Returning a tensor with shape [m, n]
        """
        def evt_aux_store(accum, alpha, C):
            F = alpha * accum
            D = F + C
            return D, F

        for m, n, k, l in self.get_problem_sizes(8):
            example_inputs = {
                "accum": self.fake_tensor(self.element, (l, m, n)),
                "alpha": 0.5,
                "C": self.fake_tensor(self.element, (l, m, n)),
                "F": self.fake_tensor(self.element, (l, m, n)),
                "D": self.fake_tensor(self.element, (l, m, n)),
            }

            launcher = EVTTestBed(self.element, evt_aux_store, example_inputs)
            input_keys = ["C", "alpha"]
            result_keys = ["D", "F"]
            launcher.verify((m, n, k), input_keys, result_keys, l)
```
**EN:** `test_aux_store(self)` is a method inside `TestEVTStore`. The docstring describes it as: Returning a tensor with shape [m, n] It defines the inner visitor `evt_aux_store`; the key traced steps are `F = alpha * accum; D = F + C; return (D, F)`. `example_inputs` prepares runtime placeholders/values for `accum`, `alpha`, `C`, `F`, `D`. `input_keys=['C', 'alpha']` and `result_keys=['D', 'F']` tell the launcher which values are supplied and which outputs must be validated. The surrounding control flow iterates over `self.get_problem_sizes(8)` to cover multiple shapes/configurations. The method ultimately routes execution through `EVTTestBed.verify`, which compares the generated CUTLASS path against a PyTorch-based reference.
**CN:** `test_aux_store(self)` 是一个位于 `TestEVTStore` 内部的方法。 文档字符串将其描述为：Returning a tensor with shape [m, n] 它定义了内部 visitor `evt_aux_store`；被 trace 的关键步骤是 `F = alpha * accum; D = F + C; return (D, F)`。 `example_inputs` 为 `accum`、`alpha`、`C`、`F`、`D` 准备运行时占位符或实际数值。 `input_keys=['C', 'alpha']` 与 `result_keys=['D', 'F']` 告诉启动器哪些值由外部提供、哪些输出需要验证。 外层控制流会遍历 `self.get_problem_sizes(8)`，以覆盖多种形状或配置。 该方法最终会通过 `EVTTestBed.verify` 执行，它会把生成的 CUTLASS 路径与基于 PyTorch 的参考实现进行比较。

### Lines 100-124
```python
    def test_col_reduce(self):
        """
        Reduction [m, n] -> [m, 1]
        """
        def evt_row_reduce(accum, alpha, C):
            acc_row_max = max(accum, dim=[2,])
            F = alpha * accum
            F_row_max = max(F, dim=[0, 2])
            D = F + C
            return D, F_row_max, acc_row_max

        for m, n, k, l in self.get_problem_sizes(8):
            example_inputs = {
                "accum": self.fake_tensor(self.element, (l, m, n)),
                "alpha": 2.0,
                "C": self.fake_tensor(self.element, (l, m, n)),
                "F_row_max": self.fake_tensor(np.float32, (m, 1)),
                "acc_row_max": self.fake_tensor(np.float32, (l, m, 1)),
                "D": self.fake_tensor(self.element, (l, m, n)),
            }

            launcher = EVTTestBed(self.element, evt_row_reduce, example_inputs)
            input_keys = ["C", "alpha"]
            result_keys = ["D", "F_row_max", "acc_row_max"]
            launcher.verify((m, n, k), input_keys, result_keys, l)
```
**EN:** `test_col_reduce(self)` is a method inside `TestEVTStore`. The docstring describes it as: Reduction [m, n] -> [m, 1] It defines the inner visitor `evt_row_reduce`; the key traced steps are `acc_row_max = max(accum, dim=[2]); F = alpha * accum; F_row_max = max(F, dim=[0, 2]); D = F + C`. `example_inputs` prepares runtime placeholders/values for `accum`, `alpha`, `C`, `F_row_max`, `acc_row_max`, `D`. `input_keys=['C', 'alpha']` and `result_keys=['D', 'F_row_max', 'acc_row_max']` tell the launcher which values are supplied and which outputs must be validated. The surrounding control flow iterates over `self.get_problem_sizes(8)` to cover multiple shapes/configurations. The method ultimately routes execution through `EVTTestBed.verify`, which compares the generated CUTLASS path against a PyTorch-based reference.
**CN:** `test_col_reduce(self)` 是一个位于 `TestEVTStore` 内部的方法。 文档字符串将其描述为：Reduction [m, n] -> [m, 1] 它定义了内部 visitor `evt_row_reduce`；被 trace 的关键步骤是 `acc_row_max = max(accum, dim=[2]); F = alpha * accum; F_row_max = max(F, dim=[0, 2]); D = F + C`。 `example_inputs` 为 `accum`、`alpha`、`C`、`F_row_max`、`acc_row_max`、`D` 准备运行时占位符或实际数值。 `input_keys=['C', 'alpha']` 与 `result_keys=['D', 'F_row_max', 'acc_row_max']` 告诉启动器哪些值由外部提供、哪些输出需要验证。 外层控制流会遍历 `self.get_problem_sizes(8)`，以覆盖多种形状或配置。 该方法最终会通过 `EVTTestBed.verify` 执行，它会把生成的 CUTLASS 路径与基于 PyTorch 的参考实现进行比较。

### Lines 126-150
```python
    def test_row_reduce(self):
        """
        Reduction [m, n] -> [n]
        """
        def evt_col_reduce(accum, alpha, C):
            acc_col_max = max(accum, dim=[1,])
            F = alpha * accum
            F_col_max = max(F, dim=[0, 1])
            D = F + C
            return D, F_col_max, acc_col_max

        for m, n, k, l in self.get_problem_sizes(8):
            example_inputs = {
                "accum": self.fake_tensor(self.element, (l, m, n)),
                "alpha": 2.0,
                "C": self.fake_tensor(self.element, (l, m, n)),
                "F_col_max": self.fake_tensor(np.float32, (n,)),
                "acc_col_max": self.fake_tensor(np.float32, (l, 1, n)),
                "D": self.fake_tensor(self.element, (l, m, n)),
            }

            launcher = EVTTestBed(self.element, evt_col_reduce, example_inputs)
            input_keys = ["C", "alpha"]
            result_keys = ["D", "F_col_max", "acc_col_max"]
            launcher.verify((m, n, k), input_keys, result_keys, l)
```
**EN:** `test_row_reduce(self)` is a method inside `TestEVTStore`. The docstring describes it as: Reduction [m, n] -> [n] It defines the inner visitor `evt_col_reduce`; the key traced steps are `acc_col_max = max(accum, dim=[1]); F = alpha * accum; F_col_max = max(F, dim=[0, 1]); D = F + C`. `example_inputs` prepares runtime placeholders/values for `accum`, `alpha`, `C`, `F_col_max`, `acc_col_max`, `D`. `input_keys=['C', 'alpha']` and `result_keys=['D', 'F_col_max', 'acc_col_max']` tell the launcher which values are supplied and which outputs must be validated. The surrounding control flow iterates over `self.get_problem_sizes(8)` to cover multiple shapes/configurations. The method ultimately routes execution through `EVTTestBed.verify`, which compares the generated CUTLASS path against a PyTorch-based reference.
**CN:** `test_row_reduce(self)` 是一个位于 `TestEVTStore` 内部的方法。 文档字符串将其描述为：Reduction [m, n] -> [n] 它定义了内部 visitor `evt_col_reduce`；被 trace 的关键步骤是 `acc_col_max = max(accum, dim=[1]); F = alpha * accum; F_col_max = max(F, dim=[0, 1]); D = F + C`。 `example_inputs` 为 `accum`、`alpha`、`C`、`F_col_max`、`acc_col_max`、`D` 准备运行时占位符或实际数值。 `input_keys=['C', 'alpha']` 与 `result_keys=['D', 'F_col_max', 'acc_col_max']` 告诉启动器哪些值由外部提供、哪些输出需要验证。 外层控制流会遍历 `self.get_problem_sizes(8)`，以覆盖多种形状或配置。 该方法最终会通过 `EVTTestBed.verify` 执行，它会把生成的 CUTLASS 路径与基于 PyTorch 的参考实现进行比较。

### Lines 152-176
```python
    def test_scalar_reduce(self):
        """
        Reduction [m, n] -> [1,]
        """
        def evt_scalar_reduce(accum, alpha, C):
            acc_max = max(accum, dim=[1, 2])
            F = alpha * accum
            F_max = max(F, dim=[0, 1, 2])
            D = F + C
            return D, F_max, acc_max

        for m, n, k, l in self.get_problem_sizes(8):
            example_inputs = {
                "accum": self.fake_tensor(self.element, (l, m, n)),
                "alpha": 2.0,
                "C": self.fake_tensor(self.element, (l, m, n)),
                "acc_max": self.fake_tensor(np.float32, (l, 1, 1)),
                "F_max": self.fake_tensor(np.float32, (1,)),
                "D": self.fake_tensor(self.element, (l, m, n)),
            }

            launcher = EVTTestBed(self.element, evt_scalar_reduce, example_inputs)
            input_keys = ["C", "alpha"]
            result_keys = ["D", "F_max", "acc_max"]
            launcher.verify((m, n, k), input_keys, result_keys, l)
```
**EN:** `test_scalar_reduce(self)` is a method inside `TestEVTStore`. The docstring describes it as: Reduction [m, n] -> [1,] It defines the inner visitor `evt_scalar_reduce`; the key traced steps are `acc_max = max(accum, dim=[1, 2]); F = alpha * accum; F_max = max(F, dim=[0, 1, 2]); D = F + C`. `example_inputs` prepares runtime placeholders/values for `accum`, `alpha`, `C`, `acc_max`, `F_max`, `D`. `input_keys=['C', 'alpha']` and `result_keys=['D', 'F_max', 'acc_max']` tell the launcher which values are supplied and which outputs must be validated. The surrounding control flow iterates over `self.get_problem_sizes(8)` to cover multiple shapes/configurations. The method ultimately routes execution through `EVTTestBed.verify`, which compares the generated CUTLASS path against a PyTorch-based reference.
**CN:** `test_scalar_reduce(self)` 是一个位于 `TestEVTStore` 内部的方法。 文档字符串将其描述为：Reduction [m, n] -> [1,] 它定义了内部 visitor `evt_scalar_reduce`；被 trace 的关键步骤是 `acc_max = max(accum, dim=[1, 2]); F = alpha * accum; F_max = max(F, dim=[0, 1, 2]); D = F + C`。 `example_inputs` 为 `accum`、`alpha`、`C`、`acc_max`、`F_max`、`D` 准备运行时占位符或实际数值。 `input_keys=['C', 'alpha']` 与 `result_keys=['D', 'F_max', 'acc_max']` 告诉启动器哪些值由外部提供、哪些输出需要验证。 外层控制流会遍历 `self.get_problem_sizes(8)`，以覆盖多种形状或配置。 该方法最终会通过 `EVTTestBed.verify` 执行，它会把生成的 CUTLASS 路径与基于 PyTorch 的参考实现进行比较。

### Lines 178-208
```python
    def test_store_with_multiple_reductions(self):
        """
        Test storing main output with multiple types of reductions
        """
        def evt_store_multi_reduce(accum, alpha, beta, C):
            F = alpha * accum + beta * C
            
            # Multiple reduction types
            row_max = max(F, dim=[2,])      # [l, m, 1]
            col_max = max(F, dim=[1,])      # [l, 1, n] 
            scalar_max = max(F, dim=[1, 2]) # [l, 1, 1]
            
            D = F + C
            return D, row_max, col_max, scalar_max

        for m, n, k, l in self.get_problem_sizes(8):
            example_inputs = {
                "accum": self.fake_tensor(self.element, (l, m, n)),
                "alpha": 2.0,
                "beta": 0.5,
                "C": self.fake_tensor(self.element, (l, m, n)),
                "D": self.fake_tensor(self.element, (l, m, n)),
                "row_max": self.fake_tensor(np.float32, (l, m, 1)),
                "col_max": self.fake_tensor(np.float32, (l, 1, n)),
                "scalar_max": self.fake_tensor(np.float32, (l, 1, 1)),
            }

            launcher = EVTTestBed(self.element, evt_store_multi_reduce, example_inputs)
            input_keys = ["C", "alpha", "beta"]
            result_keys = ["D", "row_max", "col_max", "scalar_max"]
            launcher.verify((m, n, k), input_keys, result_keys, l)
```
**EN:** `test_store_with_multiple_reductions(self)` is a method inside `TestEVTStore`. The docstring describes it as: Test storing main output with multiple types of reductions It defines the inner visitor `evt_store_multi_reduce`; the key traced steps are `F = alpha * accum + beta * C; row_max = max(F, dim=[2]); col_max = max(F, dim=[1]); scalar_max = max(F, dim=[1, 2])`. `example_inputs` prepares runtime placeholders/values for `accum`, `alpha`, `beta`, `C`, `D`, `row_max`, `col_max`, `scalar_max`. `input_keys=['C', 'alpha', 'beta']` and `result_keys=['D', 'row_max', 'col_max', 'scalar_max']` tell the launcher which values are supplied and which outputs must be validated. The surrounding control flow iterates over `self.get_problem_sizes(8)` to cover multiple shapes/configurations. The method ultimately routes execution through `EVTTestBed.verify`, which compares the generated CUTLASS path against a PyTorch-based reference.
**CN:** `test_store_with_multiple_reductions(self)` 是一个位于 `TestEVTStore` 内部的方法。 文档字符串将其描述为：Test storing main output with multiple types of reductions 它定义了内部 visitor `evt_store_multi_reduce`；被 trace 的关键步骤是 `F = alpha * accum + beta * C; row_max = max(F, dim=[2]); col_max = max(F, dim=[1]); scalar_max = max(F, dim=[1, 2])`。 `example_inputs` 为 `accum`、`alpha`、`beta`、`C`、`D`、`row_max`、`col_max`、`scalar_max` 准备运行时占位符或实际数值。 `input_keys=['C', 'alpha', 'beta']` 与 `result_keys=['D', 'row_max', 'col_max', 'scalar_max']` 告诉启动器哪些值由外部提供、哪些输出需要验证。 外层控制流会遍历 `self.get_problem_sizes(8)`，以覆盖多种形状或配置。 该方法最终会通过 `EVTTestBed.verify` 执行，它会把生成的 CUTLASS 路径与基于 PyTorch 的参考实现进行比较。

### Lines 212-213
```python
if __name__ == '__main__':
    unittest.main()
```
**EN:** This conditional block is guarded by `__name__ == '__main__'`. It acts as the executable entry point and runs `unittest.main()` when the file is invoked as a script.
**CN:** 该条件块由 `__name__ == '__main__'` 控制。 当文件以脚本方式运行时，它就是可执行入口，并会执行 `unittest.main()`。

## Key Concepts / 关键概念
- **EN:** Main-output store rules, especially SM90 restrictions on reusing `D`.
  **CN:** 主输出存储规则，尤其是 SM90 对复用 `D` 的限制。
- **EN:** Auxiliary stores that materialize intermediate tensors.
  **CN:** 将中间张量物化出来的辅助存储。
- **EN:** Row, column, and scalar reductions derived from epilogue tensors.
  **CN:** 从 epilogue 张量导出的行、列和标量归约。
- **EN:** Multi-output verification with the shared EVT testbed.
  **CN:** 使用共享 EVT testbed 验证多输出结果。

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
