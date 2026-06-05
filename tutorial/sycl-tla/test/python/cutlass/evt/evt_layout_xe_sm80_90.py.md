# evt_layout_xe_sm80_90.py — Code Analysis / 代码分析

## Source / 来源
- Path / 路径: `test/python/cutlass/evt/evt_layout_xe_sm80_90.py`
- **EN:** Validates EVT layout-transform nodes, especially permute and reshape flows that change tensor views inside the epilogue graph.
- **CN:** 验证 EVT 中的布局变换节点，重点是会在 epilogue 图中改变张量视图的 permute 与 reshape 流程。

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
class TestEVTLayout(EVTTestCaseBase):
```
**EN:** `TestEVTLayout` is the main class introduced in this block. It inherits from `EVTTestCaseBase`, so the class reuses shared helpers or unittest behavior. The decorator(s) `unittest.skipIf(device_cc() not in [12, 20, 80, 86, 89, 90], 'This unittest is only supported on CC [12, 20, 80, 86, 89, 90]')` gate execution to supported environments. That base class supplies common element types, fake tensor descriptors, and standard problem-size sweeps for all contained tests.
**CN:** `TestEVTLayout` 是此处引入的主要类。 它继承自 `EVTTestCaseBase`，因此会复用共享辅助逻辑或 unittest 行为。 装饰器 `unittest.skipIf(device_cc() not in [12, 20, 80, 86, 89, 90], 'This unittest is only supported on CC [12, 20, 80, 86, 89, 90]')` 用来把执行限制在受支持的环境中。 该基类为内部所有测试提供统一的元素类型、伪张量描述符以及标准问题尺寸扫描逻辑。

### Lines 52-75
```python
    def test_permute_1(self):
        """
        Returning a tensor with shape [m, n]
        """
        def evt_permute(accum, alpha, C):
            F = alpha * accum
            F_permute = permute(F, indices=(0, 2, 1))
            D_permute = F_permute + permute(C, indices=(0, 2, 1))
            D = permute(D_permute, indices=(0, 2, 1))
            return D, F

        for m, n, k, l in self.get_problem_sizes(8):
            example_inputs = {
                "accum": self.fake_tensor(self.element, (l, m, n)),
                "alpha": 0.5,
                "C": self.fake_tensor(self.element, (l, m, n)),
                "F": self.fake_tensor(self.element, (l, m, n)),
                "D": self.fake_tensor(self.element, (l, m, n)),
            }

            launcher = EVTTestBed(self.element, evt_permute, example_inputs)
            input_keys = ["C", "alpha"]
            result_keys = ["D", "F"]
            launcher.verify((m, n, k), input_keys, result_keys, l)
```
**EN:** `test_permute_1(self)` is a method inside `TestEVTLayout`. The docstring describes it as: Returning a tensor with shape [m, n] It defines the inner visitor `evt_permute`; the key traced steps are `F = alpha * accum; F_permute = permute(F, indices=(0, 2, 1)); D_permute = F_permute + permute(C, indices=(0, 2, 1)); D = permute(D_permute, indices=(0, 2, 1))`. `example_inputs` prepares runtime placeholders/values for `accum`, `alpha`, `C`, `F`, `D`. `input_keys=['C', 'alpha']` and `result_keys=['D', 'F']` tell the launcher which values are supplied and which outputs must be validated. The surrounding control flow iterates over `self.get_problem_sizes(8)` to cover multiple shapes/configurations. The method ultimately routes execution through `EVTTestBed.verify`, which compares the generated CUTLASS path against a PyTorch-based reference.
**CN:** `test_permute_1(self)` 是一个位于 `TestEVTLayout` 内部的方法。 文档字符串将其描述为：Returning a tensor with shape [m, n] 它定义了内部 visitor `evt_permute`；被 trace 的关键步骤是 `F = alpha * accum; F_permute = permute(F, indices=(0, 2, 1)); D_permute = F_permute + permute(C, indices=(0, 2, 1)); D = permute(D_permute, indices=(0, 2, 1))`。 `example_inputs` 为 `accum`、`alpha`、`C`、`F`、`D` 准备运行时占位符或实际数值。 `input_keys=['C', 'alpha']` 与 `result_keys=['D', 'F']` 告诉启动器哪些值由外部提供、哪些输出需要验证。 外层控制流会遍历 `self.get_problem_sizes(8)`，以覆盖多种形状或配置。 该方法最终会通过 `EVTTestBed.verify` 执行，它会把生成的 CUTLASS 路径与基于 PyTorch 的参考实现进行比较。

### Lines 77-100
```python
    @unittest.skipIf(device_cc() != 90, "This unittest is for cc = Sm90 only")
    def test_permute_2(self):
        """
        Returning a tensor with shape [m, n]
        """
        def evt_permute(accum, alpha, C):
            F = alpha * accum
            F_permute = permute(F, indices=(0, 2, 1))
            D = F_permute + C
            return D, F

        for m, n, k, l in self.get_problem_sizes(8):
            example_inputs = {
                "accum": self.fake_tensor(self.element, (l, m, n)),
                "alpha": 0.5,
                "C": self.fake_tensor(self.element, (l, n, m)),
                "F": self.fake_tensor(self.element, (l, m, n)),
                "D": self.fake_tensor(self.element, (l, n, m)),
            }

            launcher = EVTTestBed(self.element, evt_permute, example_inputs)
            input_keys = ["C", "alpha"]
            result_keys = ["D", "F"]
            launcher.verify((m, n, k), input_keys, result_keys, l)
```
**EN:** `test_permute_2(self)` is a method inside `TestEVTLayout`. The docstring describes it as: Returning a tensor with shape [m, n] It defines the inner visitor `evt_permute`; the key traced steps are `F = alpha * accum; F_permute = permute(F, indices=(0, 2, 1)); D = F_permute + C; return (D, F)`. `example_inputs` prepares runtime placeholders/values for `accum`, `alpha`, `C`, `F`, `D`. `input_keys=['C', 'alpha']` and `result_keys=['D', 'F']` tell the launcher which values are supplied and which outputs must be validated. The surrounding control flow iterates over `self.get_problem_sizes(8)` to cover multiple shapes/configurations. The method ultimately routes execution through `EVTTestBed.verify`, which compares the generated CUTLASS path against a PyTorch-based reference.
**CN:** `test_permute_2(self)` 是一个位于 `TestEVTLayout` 内部的方法。 文档字符串将其描述为：Returning a tensor with shape [m, n] 它定义了内部 visitor `evt_permute`；被 trace 的关键步骤是 `F = alpha * accum; F_permute = permute(F, indices=(0, 2, 1)); D = F_permute + C; return (D, F)`。 `example_inputs` 为 `accum`、`alpha`、`C`、`F`、`D` 准备运行时占位符或实际数值。 `input_keys=['C', 'alpha']` 与 `result_keys=['D', 'F']` 告诉启动器哪些值由外部提供、哪些输出需要验证。 外层控制流会遍历 `self.get_problem_sizes(8)`，以覆盖多种形状或配置。 该方法最终会通过 `EVTTestBed.verify` 执行，它会把生成的 CUTLASS 路径与基于 PyTorch 的参考实现进行比较。

### Lines 102-125
```python
    @unittest.skipIf(device_cc() != 90, "This unittest is for cc = Sm90 only")
    def test_permute_3(self):
        """
        Returning a tensor with shape [m, n]
        """
        def evt_permute(accum, alpha, C):
            F = alpha * accum
            F_permute = permute(F, indices=(1, 0, 2))
            D = F_permute + C
            return D, F

        for m, n, k, l in self.get_problem_sizes(8):
            example_inputs = {
                "accum": self.fake_tensor(self.element, (l, m, n)),
                "alpha": 0.5,
                "C": self.fake_tensor(self.element, (m, l, n)),
                "F": self.fake_tensor(self.element, (l, m, n)),
                "D": self.fake_tensor(self.element, (m, l, n)),
            }

            launcher = EVTTestBed(self.element, evt_permute, example_inputs)
            input_keys = ["C", "alpha"]
            result_keys = ["D", "F"]
            launcher.verify((m, n, k), input_keys, result_keys, l)
```
**EN:** `test_permute_3(self)` is a method inside `TestEVTLayout`. The docstring describes it as: Returning a tensor with shape [m, n] It defines the inner visitor `evt_permute`; the key traced steps are `F = alpha * accum; F_permute = permute(F, indices=(1, 0, 2)); D = F_permute + C; return (D, F)`. `example_inputs` prepares runtime placeholders/values for `accum`, `alpha`, `C`, `F`, `D`. `input_keys=['C', 'alpha']` and `result_keys=['D', 'F']` tell the launcher which values are supplied and which outputs must be validated. The surrounding control flow iterates over `self.get_problem_sizes(8)` to cover multiple shapes/configurations. The method ultimately routes execution through `EVTTestBed.verify`, which compares the generated CUTLASS path against a PyTorch-based reference.
**CN:** `test_permute_3(self)` 是一个位于 `TestEVTLayout` 内部的方法。 文档字符串将其描述为：Returning a tensor with shape [m, n] 它定义了内部 visitor `evt_permute`；被 trace 的关键步骤是 `F = alpha * accum; F_permute = permute(F, indices=(1, 0, 2)); D = F_permute + C; return (D, F)`。 `example_inputs` 为 `accum`、`alpha`、`C`、`F`、`D` 准备运行时占位符或实际数值。 `input_keys=['C', 'alpha']` 与 `result_keys=['D', 'F']` 告诉启动器哪些值由外部提供、哪些输出需要验证。 外层控制流会遍历 `self.get_problem_sizes(8)`，以覆盖多种形状或配置。 该方法最终会通过 `EVTTestBed.verify` 执行，它会把生成的 CUTLASS 路径与基于 PyTorch 的参考实现进行比较。

### Lines 127-147
```python
    def test_reshape(self):
        """
        Test reshape
        """
        def evt_reshape(accum, alpha, TensorE):
            F = alpha * accum
            E_reshape = reshape(TensorE, new_shape=(512, 1))
            D = F + E_reshape
            return D

        example_inputs = {
            "accum": self.fake_tensor(self.element, (self.l, self.m, self.n)),
            "alpha": 0.5,
            "TensorE": self.fake_tensor(self.element, (16, 32)),
            "D": self.fake_tensor(self.element, (self.l, self.m, self.n)),
        }

        launcher = EVTTestBed(self.element, evt_reshape, example_inputs)
        input_keys = ["alpha", "TensorE"]
        result_keys = ["D"]
        launcher.verify(self.problem_size, input_keys, result_keys, self.l)
```
**EN:** `test_reshape(self)` is a method inside `TestEVTLayout`. The docstring describes it as: Test reshape It defines the inner visitor `evt_reshape`; the key traced steps are `F = alpha * accum; E_reshape = reshape(TensorE, new_shape=(512, 1)); D = F + E_reshape; return D`. `example_inputs` prepares runtime placeholders/values for `accum`, `alpha`, `TensorE`, `D`. `input_keys=['alpha', 'TensorE']` and `result_keys=['D']` tell the launcher which values are supplied and which outputs must be validated. The method ultimately routes execution through `EVTTestBed.verify`, which compares the generated CUTLASS path against a PyTorch-based reference.
**CN:** `test_reshape(self)` 是一个位于 `TestEVTLayout` 内部的方法。 文档字符串将其描述为：Test reshape 它定义了内部 visitor `evt_reshape`；被 trace 的关键步骤是 `F = alpha * accum; E_reshape = reshape(TensorE, new_shape=(512, 1)); D = F + E_reshape; return D`。 `example_inputs` 为 `accum`、`alpha`、`TensorE`、`D` 准备运行时占位符或实际数值。 `input_keys=['alpha', 'TensorE']` 与 `result_keys=['D']` 告诉启动器哪些值由外部提供、哪些输出需要验证。 该方法最终会通过 `EVTTestBed.verify` 执行，它会把生成的 CUTLASS 路径与基于 PyTorch 的参考实现进行比较。

### Lines 149-169
```python
    def test_reshape2(self):
        """
        Test reshape
        """
        def evt_reshape(accum, alpha, TensorE):
            F = alpha * accum
            F_reshape = reshape(F, new_shape=(2, 3, 512, 256))
            D = F_reshape + TensorE
            return D

        example_inputs = {
            "accum": self.fake_tensor(self.element, (self.l, self.m, self.n)),
            "alpha": 0.5,
            "TensorE": self.fake_tensor(self.element, (2, 3, 1, self.n)),
            "D": self.fake_tensor(self.element, (2, 3, self.m, self.n)),
        }

        launcher = EVTTestBed(self.element, evt_reshape, example_inputs)
        input_keys = ["alpha", "TensorE"]
        result_keys = ["D"]
        launcher.verify(self.problem_size, input_keys, result_keys, self.l)
```
**EN:** `test_reshape2(self)` is a method inside `TestEVTLayout`. The docstring describes it as: Test reshape It defines the inner visitor `evt_reshape`; the key traced steps are `F = alpha * accum; F_reshape = reshape(F, new_shape=(2, 3, 512, 256)); D = F_reshape + TensorE; return D`. `example_inputs` prepares runtime placeholders/values for `accum`, `alpha`, `TensorE`, `D`. `input_keys=['alpha', 'TensorE']` and `result_keys=['D']` tell the launcher which values are supplied and which outputs must be validated. The method ultimately routes execution through `EVTTestBed.verify`, which compares the generated CUTLASS path against a PyTorch-based reference.
**CN:** `test_reshape2(self)` 是一个位于 `TestEVTLayout` 内部的方法。 文档字符串将其描述为：Test reshape 它定义了内部 visitor `evt_reshape`；被 trace 的关键步骤是 `F = alpha * accum; F_reshape = reshape(F, new_shape=(2, 3, 512, 256)); D = F_reshape + TensorE; return D`。 `example_inputs` 为 `accum`、`alpha`、`TensorE`、`D` 准备运行时占位符或实际数值。 `input_keys=['alpha', 'TensorE']` 与 `result_keys=['D']` 告诉启动器哪些值由外部提供、哪些输出需要验证。 该方法最终会通过 `EVTTestBed.verify` 执行，它会把生成的 CUTLASS 路径与基于 PyTorch 的参考实现进行比较。

### Lines 172-173
```python
if __name__ == '__main__':
    unittest.main()
```
**EN:** This conditional block is guarded by `__name__ == '__main__'`. It acts as the executable entry point and runs `unittest.main()` when the file is invoked as a script.
**CN:** 该条件块由 `__name__ == '__main__'` 控制。 当文件以脚本方式运行时，它就是可执行入口，并会执行 `unittest.main()`。

## Key Concepts / 关键概念
- **EN:** Layout-changing nodes: `permute` and `reshape`.
  **CN:** 改变布局的节点：`permute` 与 `reshape`。
- **EN:** Testing outputs whose physical shape differs from the original accumulator.
  **CN:** 测试物理形状与原始累加器不同的输出。
- **EN:** SM90-only permutations with alternative tensor shapes.
  **CN:** 仅在 SM90 上验证的替代张量形状置换路径。
- **EN:** Reuse of the generic EVT testbed to compare transformed outputs.
  **CN:** 复用通用 EVT testbed 对变换后的输出进行比较。

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
