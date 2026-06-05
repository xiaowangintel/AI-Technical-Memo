# test_wrapper.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/compile/test_wrapper.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Pytest module for compilation behavior in compile / test_wrapper, covering compiler settings, graph transformations, and end-to-end correctness checks. / 面向 compile / test_wrapper 的编译测试模块，覆盖编译器配置、计算图变换以及端到端正确性校验。

## Line-by-Line Analysis / 逐行分析
### Imports and shared setup (lines 5-16)
```python
import os

import pytest
import torch

from vllm.compilation.wrapper import TorchCompileWithNoGuardsWrapper
from vllm.config import (
    CompilationConfig,
    CompilationMode,
    VllmConfig,
    set_current_vllm_config,
)
```
**EN:** This block imports the modules that the rest of the file relies on. It pulls in external libraries such as os, pytest, torch; and vLLM components like vllm.compilation.wrapper, vllm.config.
**CN:** 该代码块导入后续逻辑依赖的模块。 主要包括外部库，例如 os、pytest、torch；vLLM 内部组件，例如 vllm.compilation.wrapper、vllm.config。

### Class `MyMod` (lines 19-19)
```python
class MyMod(torch.nn.Module):
```
**EN:** This helper class groups the state and behavior needed for MyMod. That structure makes repeated setup, graph inspection, or reusable assertions easier to manage.
**CN:** 这个辅助类把 MyMod 所需的状态与行为收拢到一起，使重复初始化、计算图检查或可复用断言更容易管理。

### Method `MyMod.forward` (lines 20-24)
```python
    def forward(self, x: torch.Tensor, cache: torch.Tensor | None = None):
        if x.size()[0] >= 4:
            return x * 2
        else:
            return x * 100
```
**EN:** This method on `MyMod` implements forward. It keeps the surrounding module logic factored into a reusable unit.
**CN:** `MyMod` 中的这个方法实现了 forward。 它把周边模块中的共用逻辑封装成可复用单元。

### Class `MyWrapper` (lines 27-27)
```python
class MyWrapper(TorchCompileWithNoGuardsWrapper):
```
**EN:** This helper class groups the state and behavior needed for MyWrapper. That structure makes repeated setup, graph inspection, or reusable assertions easier to manage.
**CN:** 这个辅助类把 MyWrapper 所需的状态与行为收拢到一起，使重复初始化、计算图检查或可复用断言更容易管理。

### Method `MyWrapper.__init__` (lines 28-30)
```python
    def __init__(self, model):
        self.model = model
        super().__init__()
```
**EN:** This method implements the initialization for `MyWrapper`. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 这个方法实现了 `MyWrapper` 的初始化逻辑。 它把周边模块中的共用逻辑封装成可复用单元。

### Method `MyWrapper.forward` (lines 32-34)
```python
    def forward(self, x: torch.Tensor):  # type: ignore[override]
        # this is the function to be compiled
        return self.model(x)
```
**EN:** This method on `MyWrapper` implements forward. It keeps the surrounding module logic factored into a reusable unit.
**CN:** `MyWrapper` 中的这个方法实现了 forward。 它把周边模块中的共用逻辑封装成可复用单元。

### Function `test_torch_compile_wrapper` (lines 37-117)
```python
@pytest.mark.parametrize("use_bytecode_hook", [True, False])
def test_torch_compile_wrapper(use_bytecode_hook, monkeypatch):
    """Test basic functionality of TorchCompileWithNoGuardsWrapper."""
    # Set the environment variable for this test
    monkeypatch.setenv("VLLM_USE_BYTECODE_HOOK", "1" if use_bytecode_hook else "0")

    # Create a proper vLLM config instead of mocking
    vllm_config = VllmConfig()
    vllm_config.compilation_config = CompilationConfig()
    vllm_config.compilation_config.mode = CompilationMode.DYNAMO_TRACE_ONCE
    vllm_config.compilation_config.backend = "inductor"

    # Test DYNAMO_TRACE_ONCE
    with set_current_vllm_config(vllm_config):
        torch._dynamo.reset()
        mod = MyMod()
        wrapper = MyWrapper(mod)

        # First call should trigger compilation
        x = torch.tensor([1, 2, 3, 4])
        torch._dynamo.mark_dynamic(x, 0)

        result1 = wrapper(x)
        expected1 = torch.tensor([2, 4, 6, 8])
        assert torch.allclose(result1, expected1), (
            f"Expected {expected1}, got {result1}"
        )

        # Second call should use compiled code
        x2 = torch.tensor([1, 2, 3])
        result2 = wrapper(x2)
        expected2 = torch.tensor([2, 4, 6])
        assert torch.allclose(result2, expected2), (
            f"Expected {expected2}, got {result2}"
        )

        # without the wrapper result would be different.
        result3 = mod(x2)
        expected3 = torch.tensor([100, 200, 300])

# ... excerpt ...
        )

    # with STOCK_TORCH_COMPILE we do not remove guards.
    vllm_config.compilation_config.mode = CompilationMode.STOCK_TORCH_COMPILE
    torch._dynamo.reset()
    with set_current_vllm_config(vllm_config):
        mod = MyMod()
        wrapper = MyWrapper(mod)

        # First call should trigger compilation
        x = torch.tensor([1, 2, 3, 4])
        torch._dynamo.mark_dynamic(x, 0)

        result1 = wrapper(x)
        expected1 = torch.tensor([2, 4, 6, 8])
        assert torch.allclose(result1, expected1), (
            f"Expected {expected1}, got {result1}"
        )

        # Second call should trigger another compilation
        x2 = torch.tensor([1, 2, 3])
        result2 = wrapper(x2)
        expected2 = torch.tensor([100, 200, 300])
        assert torch.allclose(result2, expected2), (
            f"Expected {expected2}, got {result2}"
        )

    # NO_COMPILATION level not supported.
    vllm_config.compilation_config.mode = None
    torch._dynamo.reset()
    with set_current_vllm_config(vllm_config):
        torch._dynamo.reset()
        mod = MyMod()

        try:
            wrapper = MyWrapper(mod)
        except Exception:
            return
        raise AssertionError("expected an exception to be raised")
```
**EN:** This pytest case verifies torch compile wrapper. It is parameterized across 1 input dimension so one definition covers many scenarios. it consumes fixtures or inputs such as use_bytecode_hook, monkeypatch. assertions at the end lock in the intended behavior or graph shape. Only the key portions are shown here because the block is large.
**CN:** 该 pytest 用例验证 torch compile wrapper 的行为。 它通过 1 组参数化输入覆盖多种场景；它会使用诸如 use_bytecode_hook、monkeypatch 等 fixture 或输入；结尾处的断言会固定预期行为或计算图形态。 由于该代码块较大，这里只展示关键片段。

### Top-level block starting at line 120 (lines 120-135)
```python
if __name__ == "__main__":
    # Run with both parameter values

    class MockMonkeypatch:
        def setenv(self, name, value):
            os.environ[name] = value

    mp = MockMonkeypatch()

    print("Testing with VLLM_USE_BYTECODE_HOOK=False")
    test_torch_compile_wrapper(False, mp)

    print("Testing with VLLM_USE_BYTECODE_HOOK=True")
    test_torch_compile_wrapper(True, mp)

    print("All tests passed!")
```
**EN:** This top-level `If` block performs supporting work that the surrounding tests depend on.
**CN:** 这个顶层 `If` 代码块执行周边测试所依赖的辅助逻辑。

## Key Concepts / 关键概念
- **Pytest parameterization / pytest 参数化:** The module expands one definition into many concrete scenarios through parametrized inputs. / 该模块通过参数化输入把单个定义扩展成多组具体场景。
- **Behavioral assertions / 行为断言:** Assertions check tensor values, graph structure, or expected error conditions. / 断言用于检查张量结果、计算图结构或预期的报错路径。
- **Compilation coverage / 编译路径覆盖:** The file exercises compile modes, graph passes, backend choices, or output stability across configurations. / 该文件覆盖编译模式、图优化 pass、后端选择以及不同配置下的输出稳定性。

## Dependencies / 依赖关系
- `os`
- `pytest`
- `torch`
- `vllm.compilation.wrapper -> TorchCompileWithNoGuardsWrapper`
- `vllm.config -> CompilationConfig, CompilationMode, VllmConfig, set_current_vllm_config`
