# test_slice_kernel.py — Code Analysis / 代码分析

## Source / 来源
- **Path / 路径:** `python/test/unit/tools/test_slice_kernel.py`
- **EN:** Pytest module covering slice kernel behavior in Triton's Python tests. It contains 23 top-level definition(s) and 13 imported module reference(s).
- **CN:** 这是一个 pytest 模块，用于覆盖 Triton Python 测试中的 slice kernel 行为。 该文件包含 23 个顶层定义，以及 13 个导入模块引用。

## Line-by-Line Analysis / 逐行分析

### Lines 1-13

```python
import ast
import importlib
import sys
import sysconfig
import textwrap
import uuid
from pathlib import Path
from typing import Callable

import pytest

from triton.tools.triton_to_gluon_translator.slice_kernel import RewriteSpec, get_reference, slice_kernel
from triton.tools.triton_to_gluon_translator.target import TranslatorTarget
```
- **EN:** Imports the modules used in this scope: `ast`, `importlib`, `sys`, `sysconfig`, `textwrap`, `uuid`, `pathlib`, `typing`, `pytest`, `triton.tools.triton_to_gluon_translator.slice_kernel`, and 1 more.
- **CN:** 导入此作用域使用的模块：`ast`、`importlib`、`sys`、`sysconfig`、`textwrap`、`uuid`、`pathlib`、`typing`、`pytest`、`triton.tools.triton_to_gluon_translator.slice_kernel` 等另外 1 项。

### Lines 14-17

```python


@pytest.fixture(autouse=True)
def clean_import_state(monkeypatch):
```
- **EN:** Defines the helper function `clean_import_state`. Decorators: `pytest.fixture(autouse=True)`. Parameters: `monkeypatch`. Key calls include `pytest.fixture`, `monkeypatch.setattr`, `sys.modules.copy`, `sys.modules.clear`, `sys.modules.update`. This scope touches shared fixtures.
- **CN:** 定义辅助函数 `clean_import_state`。 装饰器：`pytest.fixture(autouse=True)`。 参数：`monkeypatch`。 关键调用包括 `pytest.fixture`、`monkeypatch.setattr`、`sys.modules.copy`、`sys.modules.clear`、`sys.modules.update`。 该作用域涉及共享 fixture。

#### Lines 18-19

```python
    monkeypatch.setattr(sys, "path", list(sys.path))
    original = sys.modules.copy()
```
- **EN:** Prepares or updates state through `original`. Invokes `monkeypatch.setattr`, `sys.modules.copy` to execute the test logic.
- **CN:** 通过 `original` 准备或更新状态。 调用 `monkeypatch.setattr`、`sys.modules.copy` 执行测试逻辑。

#### Lines 20-24

```python
    try:
        yield
    finally:
        sys.modules.clear()
        sys.modules.update(original)
```
- **EN:** Invokes `sys.modules.clear`, `sys.modules.update` to execute the test logic. Wraps operations in exception-handling logic.
- **CN:** 调用 `sys.modules.clear`、`sys.modules.update` 执行测试逻辑。 使用异常处理逻辑包裹相关操作。

### Lines 25-27

```python


def _make_package(tmp_path: Path, files: dict[str, str]) -> tuple[str, Callable[[str], str]]:
```
- **EN:** Defines the helper function `_make_package`. Parameters: `tmp_path`, `files`. Key calls include `pkg_dir.mkdir`, `files.items`, `sys.path.insert`, `importlib.invalidate_caches`, `uuid.uuid4`, `textwrap.dedent`. This scope touches cache management behavior.
- **CN:** 定义辅助函数 `_make_package`。 参数：`tmp_path`、`files`。 关键调用包括 `pkg_dir.mkdir`、`files.items`、`sys.path.insert`、`importlib.invalidate_caches`、`uuid.uuid4`、`textwrap.dedent`。 该作用域涉及缓存管理行为。

#### Lines 28-31

```python
    pkg = f"slice_kernel_test_{uuid.uuid4().hex}"
    pkg_dir = tmp_path / pkg
    pkg_dir.mkdir()
    (pkg_dir / "__init__.py").write_text("")
```
- **EN:** Prepares or updates state through `pkg`, `pkg_dir`. Invokes `uuid.uuid4`, `pkg_dir.mkdir` to execute the test logic.
- **CN:** 通过 `pkg`、`pkg_dir` 准备或更新状态。 调用 `uuid.uuid4`、`pkg_dir.mkdir` 执行测试逻辑。

#### Lines 32-33

```python
    for name, source in files.items():
        (pkg_dir / name).write_text(textwrap.dedent(source).strip().replace("{pkg}", pkg) + "\n")
```
- **EN:** Invokes `files.items`, `textwrap.dedent` to execute the test logic. Iterates across cases or data tiles.
- **CN:** 调用 `files.items`、`textwrap.dedent` 执行测试逻辑。 通过循环覆盖多个用例或数据分块。

#### Lines 34-36

```python
    sys.path.insert(0, str(tmp_path))
    importlib.invalidate_caches()
    return pkg, lambda mod: f"{pkg}.{mod}"
```
- **EN:** Invokes `sys.path.insert`, `importlib.invalidate_caches` to execute the test logic. Relevant themes: cache management behavior.
- **CN:** 调用 `sys.path.insert`、`importlib.invalidate_caches` 执行测试逻辑。 相关主题：缓存管理行为。

### Lines 37-39

```python


def assert_code_equal(actual: str, expected: str) -> None:
```
- **EN:** Defines the helper function `assert_code_equal`. Parameters: `actual`, `expected`. Key calls include `ast.dump`, `ast.parse`.
- **CN:** 定义辅助函数 `assert_code_equal`。 参数：`actual`、`expected`。 关键调用包括 `ast.dump`、`ast.parse`。

#### Lines 40-41

```python
    lhs = ast.dump(ast.parse(actual))
    rhs = ast.dump(ast.parse(expected))
```
- **EN:** Prepares or updates state through `lhs`, `rhs`. Invokes `ast.dump`, `ast.parse` to execute the test logic.
- **CN:** 通过 `lhs`、`rhs` 准备或更新状态。 调用 `ast.dump`、`ast.parse` 执行测试逻辑。

#### Lines 42-42

```python
    assert lhs == rhs
```
- **EN:** Validates behavior with 1 assertion(s).
- **CN:** 通过 1 个断言验证行为。

### Lines 43-45

```python


def test_slice_kernel_basic_module_slicing(tmp_path):
```
- **EN:** Defines the test function `test_slice_kernel_basic_module_slicing`. Parameters: `tmp_path`. Key calls include `_make_package`, `slice_kernel`, `assert_code_equal`, `mod`.
- **CN:** 定义测试函数 `test_slice_kernel_basic_module_slicing`。 参数：`tmp_path`。 关键调用包括 `_make_package`、`slice_kernel`、`assert_code_equal`、`mod`。

#### Lines 46-118

```python
    pkg, mod = _make_package(
        tmp_path,
        {
            "lib_foo.py":
            """
                import math

                def some_util() -> int:
                    return 42

                def common_util() -> int:
                    return math.prod([11, 33])
            """,
            "lib_bar.py":
            """
                def prod(values) -> None:
                    return

                def some_util() -> int:
                    return 55
            """,
            "kernel_mod.py":
            """
                from .lib_bar import prod as util_bar
                from .lib_bar import some_util
                from .lib_foo import common_util as util_foo
                from .lib_foo import some_util as util

                def prod() -> None:
                    return

                def kernel() -> None:
                    prod()
                    util_foo()
                    util_bar([42, 22])
                    some_util()
                    util()
            """,
        },
    )

    output = slice_kernel([f"{mod('kernel_mod')}:kernel"], ["triton", "torch"], target=TranslatorTarget.GENERIC)
    expected = R"""
import math

def lib_foo_some_util() -> int:
    return 42


def some_util() -> int:
    return 55


def lib_bar_prod(values) -> None:
    return


def common_util() -> int:
    return math.prod([11, 33])


def prod() -> None:
    return


def kernel() -> None:
    prod()
    common_util()
    lib_bar_prod([42, 22])
    some_util()
    lib_foo_some_util()
    """
    assert_code_equal(output, expected)
```
- **EN:** Prepares or updates state through `pkg`, `mod`, `output`, `expected`. Invokes `_make_package`, `slice_kernel`, `mod`, `assert_code_equal` to execute the test logic.
- **CN:** 通过 `pkg`、`mod`、`output`、`expected` 准备或更新状态。 调用 `_make_package`、`slice_kernel`、`mod`、`assert_code_equal` 执行测试逻辑。

### Lines 119-121

```python


def test_slice_kernel_does_not_treat_site_packages_as_stdlib(tmp_path, monkeypatch):
```
- **EN:** Defines the test function `test_slice_kernel_does_not_treat_site_packages_as_stdlib`. Parameters: `tmp_path`, `monkeypatch`. Nested definitions in this scope: `fake_get_paths`. Key calls include `fake_site_packages.mkdir`, `_make_package`, `monkeypatch.setattr`, `slice_kernel`, `assert_code_equal`, `original_get_paths`, and 1 more.
- **CN:** 定义测试函数 `test_slice_kernel_does_not_treat_site_packages_as_stdlib`。 参数：`tmp_path`、`monkeypatch`。 该作用域中的嵌套定义：`fake_get_paths`。 关键调用包括 `fake_site_packages.mkdir`、`_make_package`、`monkeypatch.setattr`、`slice_kernel`、`assert_code_equal`、`original_get_paths` 等另外 1 项。

#### Lines 122-143

```python
    fake_stdlib = tmp_path / "venv" / "lib" / "python3.12"
    fake_site_packages = fake_stdlib / "site-packages"
    fake_site_packages.mkdir(parents=True)
    pkg, mod = _make_package(
        fake_site_packages,
        {
            "helpers.py":
            """
                def helper() -> int:
                    return 7
            """,
            "kernel_mod.py":
            """
                from .helpers import helper

                def kernel() -> int:
                    return helper()
            """,
        },
    )

    original_get_paths = sysconfig.get_paths
```
- **EN:** Prepares or updates state through `fake_stdlib`, `fake_site_packages`, `pkg`, `mod`, `original_get_paths`. Invokes `fake_site_packages.mkdir`, `_make_package` to execute the test logic.
- **CN:** 通过 `fake_stdlib`、`fake_site_packages`、`pkg`、`mod`、`original_get_paths` 准备或更新状态。 调用 `fake_site_packages.mkdir`、`_make_package` 执行测试逻辑。

#### Lines 144-145

```python

    def fake_get_paths():
```
- **EN:** Defines the helper function `fake_get_paths`. Key calls include `original_get_paths`.
- **CN:** 定义辅助函数 `fake_get_paths`。 关键调用包括 `original_get_paths`。

##### Lines 146-151

```python
        paths = original_get_paths().copy()
        paths["stdlib"] = str(fake_stdlib)
        paths["platstdlib"] = str(fake_stdlib)
        paths["purelib"] = str(fake_site_packages)
        paths["platlib"] = str(fake_site_packages)
        return paths
```
- **EN:** Prepares or updates state through `paths`. Invokes `original_get_paths` to execute the test logic.
- **CN:** 通过 `paths` 准备或更新状态。 调用 `original_get_paths` 执行测试逻辑。

#### Lines 152-164

```python

    monkeypatch.setattr(sysconfig, "get_paths", fake_get_paths)

    output = slice_kernel([f"{mod('kernel_mod')}:kernel"], ["triton", "torch"], target=TranslatorTarget.GENERIC)
    expected = R"""
def helper() -> int:
    return 7


def kernel() -> int:
    return helper()
    """
    assert_code_equal(output, expected)
```
- **EN:** Prepares or updates state through `output`, `expected`. Invokes `monkeypatch.setattr`, `slice_kernel`, `mod`, `assert_code_equal` to execute the test logic.
- **CN:** 通过 `output`、`expected` 准备或更新状态。 调用 `monkeypatch.setattr`、`slice_kernel`、`mod`、`assert_code_equal` 执行测试逻辑。

### Lines 165-167

```python


def test_slice_kernel_supports_injected_decorator_matchers(tmp_path):
```
- **EN:** Defines the test function `test_slice_kernel_supports_injected_decorator_matchers`. Parameters: `tmp_path`. Nested definitions in this scope: `matcher`. Key calls include `_make_package`, `importlib.import_module`, `slice_kernel`, `assert_code_equal`, `mod`, `get_reference`, and 2 more.
- **CN:** 定义测试函数 `test_slice_kernel_supports_injected_decorator_matchers`。 参数：`tmp_path`。 该作用域中的嵌套定义：`matcher`。 关键调用包括 `_make_package`、`importlib.import_module`、`slice_kernel`、`assert_code_equal`、`mod`、`get_reference` 等另外 2 项。

#### Lines 168-208

```python
    pkg, mod = _make_package(
        tmp_path,
        {
            "kernel_mod.py":
            """
                def mock_kernel(fn=None, *, idle_sms=None):
                    def deco(inner):
                        return inner

                    if fn is None:
                        return deco
                    return deco(fn)

                def keep():
                    def deco(inner):
                        return inner

                    return deco

                def foo() -> None:
                    pass

                @mock_kernel
                @keep()
                def kernel_top() -> None:
                    foo()

                def nested_dep() -> int:
                    return 0

                def get_idle_sms() -> int:
                    return nested_dep()

                @keep()
                @mock_kernel(idle_sms=get_idle_sms())
                def kernel_bottom() -> None:
                    foo()
            """,
        },
    )
    module = importlib.import_module(mod("kernel_mod"))
```
- **EN:** Prepares or updates state through `pkg`, `mod`, `module`. Invokes `_make_package`, `importlib.import_module`, `mod` to execute the test logic.
- **CN:** 通过 `pkg`、`mod`、`module` 准备或更新状态。 调用 `_make_package`、`importlib.import_module`、`mod` 执行测试逻辑。

#### Lines 209-210

```python

    def matcher(context, cur_module, decorator):
```
- **EN:** Defines the helper function `matcher`. Parameters: `context`, `cur_module`, `decorator`. Key calls include `get_reference`, `isinstance`.
- **CN:** 定义辅助函数 `matcher`。 参数：`context`、`cur_module`、`decorator`。 关键调用包括 `get_reference`、`isinstance`。

##### Lines 211-213

```python
        func = decorator.func if isinstance(decorator, ast.Call) else decorator
        ref = get_reference(context, cur_module, func)
        return ref is not None and ref[0] is module.mock_kernel
```
- **EN:** Prepares or updates state through `func`, `ref`. Invokes `isinstance`, `get_reference` to execute the test logic.
- **CN:** 通过 `func`、`ref` 准备或更新状态。 调用 `isinstance`、`get_reference` 执行测试逻辑。

#### Lines 214-260

```python

    top = slice_kernel(
        [f"{mod('kernel_mod')}:kernel_top"],
        ["triton", "torch"],
        rewrite_spec=RewriteSpec(ignored_decorator_matchers=[matcher]),
        target=TranslatorTarget.GENERIC,
    )
    expected_top = R"""
def keep():

    def deco(inner):
        return inner
    return deco


def foo() -> None:
    pass


@keep()
def kernel_top() -> None:
    foo()
    """
    assert_code_equal(top, expected_top)

    bottom = slice_kernel(
        [f"{mod('kernel_mod')}:kernel_bottom"],
        ["triton", "torch"],
        rewrite_spec=RewriteSpec(ignored_decorator_matchers=[matcher]),
        target=TranslatorTarget.GENERIC,
    )
    expected_bottom = R"""
def keep():

    def deco(inner):
        return inner
    return deco


def foo() -> None:
    pass


def kernel_bottom() -> None:
    foo()
    """
    assert_code_equal(bottom, expected_bottom)
```
- **EN:** Prepares or updates state through `top`, `expected_top`, `bottom`, `expected_bottom`. Invokes `slice_kernel`, `RewriteSpec`, `mod`, `assert_code_equal` to execute the test logic.
- **CN:** 通过 `top`、`expected_top`、`bottom`、`expected_bottom` 准备或更新状态。 调用 `slice_kernel`、`RewriteSpec`、`mod`、`assert_code_equal` 执行测试逻辑。

### Lines 261-263

```python


def test_slice_kernel_translate_to_gluon_keeps_tensor_method_rewrites(tmp_path):
```
- **EN:** Defines the test function `test_slice_kernel_translate_to_gluon_keeps_tensor_method_rewrites`. Parameters: `tmp_path`. Key calls include `_make_package`, `slice_kernel`, `assert_code_equal`, `mod`. This scope touches Triton compilation or JIT kernels, Triton language operations, layout transformation reasoning.
- **CN:** 定义测试函数 `test_slice_kernel_translate_to_gluon_keeps_tensor_method_rewrites`。 参数：`tmp_path`。 关键调用包括 `_make_package`、`slice_kernel`、`assert_code_equal`、`mod`。 该作用域涉及Triton 编译或 JIT kernel、Triton language 操作、布局变换推理。

#### Lines 264-296

```python
    pkg, mod = _make_package(
        tmp_path,
        {
            "kernel_mod.py":
            """
                import triton
                import triton.language as tl

                @triton.jit(repr=lambda _: "custom_kernel_name")
                def kernel(x):
                    tl.squeeze(x, 0)
            """,
        },
    )

    output = slice_kernel([f"{mod('kernel_mod')}:kernel"], ["triton", "torch"], translate_to_gluon=True,
                          target=TranslatorTarget.GENERIC)
    expected = R"""
import triton.experimental.gluon.language as gl
import triton.tools.triton_to_gluon_translator.common_helpers
import triton.experimental.gluon as gluon

@gluon.jit
def squeeze(x, dim: gl.constexpr):
    gl.static_assert(x.shape[dim] == 1)
    return triton.tools.triton_to_gluon_translator.common_helpers.reset_to_default_layout(x.reshape(x.shape[:dim] + x.shape[dim + 1:]))


@gluon.jit(repr=lambda _: 'custom_kernel_name')
def kernel(x):
    squeeze(x, 0)
    """
    assert_code_equal(output, expected)
```
- **EN:** Prepares or updates state through `pkg`, `mod`, `output`, `expected`. Invokes `_make_package`, `slice_kernel`, `mod`, `assert_code_equal` to execute the test logic. Relevant themes: Triton compilation or JIT kernels, Triton language operations, layout transformation reasoning.
- **CN:** 通过 `pkg`、`mod`、`output`、`expected` 准备或更新状态。 调用 `_make_package`、`slice_kernel`、`mod`、`assert_code_equal` 执行测试逻辑。 相关主题：Triton 编译或 JIT kernel、Triton language 操作、布局变换推理。

### Lines 297-299

```python


def test_slice_kernel_translate_to_gluon_inlines_descriptor_adapter(tmp_path):
```
- **EN:** Defines the test function `test_slice_kernel_translate_to_gluon_inlines_descriptor_adapter`. Parameters: `tmp_path`. Key calls include `_make_package`, `slice_kernel`, `assert_code_equal`, `mod`. This scope touches PyTorch tensor setup and checks, tensor/descriptor metadata, layout transformation reasoning.
- **CN:** 定义测试函数 `test_slice_kernel_translate_to_gluon_inlines_descriptor_adapter`。 参数：`tmp_path`。 关键调用包括 `_make_package`、`slice_kernel`、`assert_code_equal`、`mod`。 该作用域涉及PyTorch 张量准备与校验、张量/描述符元数据、布局变换推理。

#### Lines 300-343

```python
    pkg, mod = _make_package(
        tmp_path,
        {
            "kernel_mod.py":
            """
                from triton.tools.ragged_tma import create_ragged_descriptor

                def kernel(t):
                    return create_ragged_descriptor(t, [16, 16])
            """,
        },
    )

    output = slice_kernel([f"{mod('kernel_mod')}:kernel"], ["triton", "torch"], translate_to_gluon=True,
                          target=TranslatorTarget.GENERIC)
    expected = R"""
import triton.tools.ragged_tma

def kernel(t):
    return convert_host_descriptor(triton.tools.ragged_tma.create_ragged_descriptor(t, [16, 16]))


def _torch_dtype_to_triton(dtype):
    import torch

    if dtype == torch.float8_e5m2:
        return gl.float8e5
    if dtype == torch.float8_e4m3fn:
        return gl.float8e4nv
    return getattr(gl, str(dtype).split(".")[1])

def convert_host_descriptor(desc):
    from triton.tools.tensor_descriptor import TensorDescriptor

    assert isinstance(desc, TensorDescriptor)
    block_shape = desc.block_shape
    dtype = desc.base.dtype
    tensor = desc.base
    layout = gl.NVMMASharedLayout.get_default_for(block_shape, _torch_dtype_to_triton(dtype))
    return gluon.nvidia.hopper.TensorDescriptor(
        tensor, desc.shape, desc.strides, block_shape, layout
    )
    """
    assert_code_equal(output, expected)
```
- **EN:** Prepares or updates state through `pkg`, `mod`, `output`, `expected`. Invokes `_make_package`, `slice_kernel`, `mod`, `assert_code_equal` to execute the test logic. Relevant themes: PyTorch tensor setup and checks, tensor/descriptor metadata, layout transformation reasoning.
- **CN:** 通过 `pkg`、`mod`、`output`、`expected` 准备或更新状态。 调用 `_make_package`、`slice_kernel`、`mod`、`assert_code_equal` 执行测试逻辑。 相关主题：PyTorch 张量准备与校验、张量/描述符元数据、布局变换推理。

### Lines 344-346

```python


def test_slice_kernel_binds_local_imports(tmp_path):
```
- **EN:** Defines the test function `test_slice_kernel_binds_local_imports`. Parameters: `tmp_path`. Key calls include `_make_package`, `slice_kernel`, `assert_code_equal`, `mod`.
- **CN:** 定义测试函数 `test_slice_kernel_binds_local_imports`。 参数：`tmp_path`。 关键调用包括 `_make_package`、`slice_kernel`、`assert_code_equal`、`mod`。

#### Lines 347-373

```python
    pkg, mod = _make_package(
        tmp_path,
        {
            "helpers.py":
            """
                def local_helper() -> int:
                    return 3
            """,
            "kernel_mod.py":
            """
                def kernel() -> int:
                    from .helpers import local_helper
                    return local_helper()
            """,
        },
    )

    output = slice_kernel([f"{mod('kernel_mod')}:kernel"], ["triton", "torch"], target=TranslatorTarget.GENERIC)
    expected = R"""
def local_helper() -> int:
    return 3


def kernel() -> int:
    return local_helper()
    """
    assert_code_equal(output, expected)
```
- **EN:** Prepares or updates state through `pkg`, `mod`, `output`, `expected`. Invokes `_make_package`, `slice_kernel`, `mod`, `assert_code_equal` to execute the test logic.
- **CN:** 通过 `pkg`、`mod`、`output`、`expected` 准备或更新状态。 调用 `_make_package`、`slice_kernel`、`mod`、`assert_code_equal` 执行测试逻辑。

### Lines 374-376

```python


def test_slice_kernel_function_import(tmp_path):
```
- **EN:** Defines the test function `test_slice_kernel_function_import`. Parameters: `tmp_path`. Key calls include `_make_package`, `slice_kernel`, `assert_code_equal`, `mod`.
- **CN:** 定义测试函数 `test_slice_kernel_function_import`。 参数：`tmp_path`。 关键调用包括 `_make_package`、`slice_kernel`、`assert_code_equal`、`mod`。

#### Lines 377-404

```python
    pkg, mod = _make_package(
        tmp_path,
        {
            "kernel_mod.py":
            """
                def helper() -> None:
                    import math

                    math.prod([11, 33])

                def kernel() -> None:
                    helper()
            """,
        },
    )

    output = slice_kernel([f"{mod('kernel_mod')}:kernel"], ["triton", "torch"], target=TranslatorTarget.GENERIC)
    expected = R"""
import math

def helper() -> None:
    math.prod([11, 33])


def kernel() -> None:
    helper()
    """
    assert_code_equal(output, expected)
```
- **EN:** Prepares or updates state through `pkg`, `mod`, `output`, `expected`. Invokes `_make_package`, `slice_kernel`, `mod`, `assert_code_equal` to execute the test logic.
- **CN:** 通过 `pkg`、`mod`、`output`、`expected` 准备或更新状态。 调用 `_make_package`、`slice_kernel`、`mod`、`assert_code_equal` 执行测试逻辑。

### Lines 405-411

```python


@pytest.mark.xfail(
    strict=True,
    reason="TODO: handle local import aliases that are used as module values",
)
def test_slice_kernel_function_import_module_value(tmp_path):
```
- **EN:** Defines the test function `test_slice_kernel_function_import_module_value`. Decorators: `pytest.mark.xfail(strict=True, reason='TODO: handle local import aliases that are used as module values')`. Parameters: `tmp_path`. Key calls include `pytest.mark.xfail`, `_make_package`, `slice_kernel`, `assert_code_equal`, `mod`.
- **CN:** 定义测试函数 `test_slice_kernel_function_import_module_value`。 装饰器：`pytest.mark.xfail(strict=True, reason='TODO: handle local import aliases that are used as module values')`。 参数：`tmp_path`。 关键调用包括 `pytest.mark.xfail`、`_make_package`、`slice_kernel`、`assert_code_equal`、`mod`。

#### Lines 412-439

```python
    pkg, mod = _make_package(
        tmp_path,
        {
            "kernel_mod.py":
            """
                def helper():
                    import math

                    return math

                def kernel():
                    return helper()
            """,
        },
    )

    output = slice_kernel([f"{mod('kernel_mod')}:kernel"], ["triton", "torch"], target=TranslatorTarget.GENERIC)
    expected = R"""
import math

def helper():
    return math


def kernel():
    return helper()
    """
    assert_code_equal(output, expected)
```
- **EN:** Prepares or updates state through `pkg`, `mod`, `output`, `expected`. Invokes `_make_package`, `slice_kernel`, `mod`, `assert_code_equal` to execute the test logic.
- **CN:** 通过 `pkg`、`mod`、`output`、`expected` 准备或更新状态。 调用 `_make_package`、`slice_kernel`、`mod`、`assert_code_equal` 执行测试逻辑。

### Lines 440-442

```python


def test_slice_kernel_function_relative_import(tmp_path):
```
- **EN:** Defines the test function `test_slice_kernel_function_relative_import`. Parameters: `tmp_path`. Key calls include `_make_package`, `slice_kernel`, `assert_code_equal`, `mod`.
- **CN:** 定义测试函数 `test_slice_kernel_function_relative_import`。 参数：`tmp_path`。 关键调用包括 `_make_package`、`slice_kernel`、`assert_code_equal`、`mod`。

#### Lines 443-481

```python
    pkg, mod = _make_package(
        tmp_path,
        {
            "lib_foo.py":
            """
                import math

                def common_util() -> int:
                    return math.prod([11, 33])
            """,
            "kernel_mod.py":
            """
                def helper() -> None:
                    from .lib_foo import common_util as util_foo

                    util_foo()

                def kernel() -> None:
                    helper()
            """,
        },
    )

    output = slice_kernel([f"{mod('kernel_mod')}:kernel"], ["triton", "torch"], target=TranslatorTarget.GENERIC)
    expected = R"""
import math

def common_util() -> int:
    return math.prod([11, 33])


def helper() -> None:
    common_util()


def kernel() -> None:
    helper()
    """
    assert_code_equal(output, expected)
```
- **EN:** Prepares or updates state through `pkg`, `mod`, `output`, `expected`. Invokes `_make_package`, `slice_kernel`, `mod`, `assert_code_equal` to execute the test logic.
- **CN:** 通过 `pkg`、`mod`、`output`、`expected` 准备或更新状态。 调用 `_make_package`、`slice_kernel`、`mod`、`assert_code_equal` 执行测试逻辑。

### Lines 482-488

```python


@pytest.mark.xfail(
    strict=True,
    reason="TODO: preserve origin metadata for local from-import values",
)
def test_slice_kernel_function_from_import_value(tmp_path):
```
- **EN:** Defines the test function `test_slice_kernel_function_from_import_value`. Decorators: `pytest.mark.xfail(strict=True, reason='TODO: preserve origin metadata for local from-import values')`. Parameters: `tmp_path`. Key calls include `pytest.mark.xfail`, `_make_package`, `slice_kernel`, `assert_code_equal`, `mod`.
- **CN:** 定义测试函数 `test_slice_kernel_function_from_import_value`。 装饰器：`pytest.mark.xfail(strict=True, reason='TODO: preserve origin metadata for local from-import values')`。 参数：`tmp_path`。 关键调用包括 `pytest.mark.xfail`、`_make_package`、`slice_kernel`、`assert_code_equal`、`mod`。

#### Lines 489-516

```python
    pkg, mod = _make_package(
        tmp_path,
        {
            "kernel_mod.py":
            """
                def helper():
                    from math import pi

                    return pi

                def kernel():
                    return helper()
            """,
        },
    )

    output = slice_kernel([f"{mod('kernel_mod')}:kernel"], ["triton", "torch"], target=TranslatorTarget.GENERIC)
    expected = R"""
import math

def helper():
    return math.pi


def kernel():
    return helper()
    """
    assert_code_equal(output, expected)
```
- **EN:** Prepares or updates state through `pkg`, `mod`, `output`, `expected`. Invokes `_make_package`, `slice_kernel`, `mod`, `assert_code_equal` to execute the test logic.
- **CN:** 通过 `pkg`、`mod`、`output`、`expected` 准备或更新状态。 调用 `_make_package`、`slice_kernel`、`mod`、`assert_code_equal` 执行测试逻辑。

### Lines 517-519

```python


def test_slice_kernel_function_absolute_import(tmp_path):
```
- **EN:** Defines the test function `test_slice_kernel_function_absolute_import`. Parameters: `tmp_path`. Key calls include `_make_package`, `slice_kernel`, `assert_code_equal`, `mod`.
- **CN:** 定义测试函数 `test_slice_kernel_function_absolute_import`。 参数：`tmp_path`。 关键调用包括 `_make_package`、`slice_kernel`、`assert_code_equal`、`mod`。

#### Lines 520-562

```python
    pkg, mod = _make_package(
        tmp_path,
        {
            "lib_foo.py":
            """
                import math

                def common_util() -> int:
                    return math.prod([11, 33])
            """,
            "kernel_mod.py":
            """
                import {pkg}.lib_foo

                _PRELOADED_LIB_FOO = {pkg}.lib_foo

                def helper() -> None:
                    from {pkg}.lib_foo import common_util as util_foo

                    util_foo()

                def kernel() -> None:
                    helper()
            """,
        },
    )

    output = slice_kernel([f"{mod('kernel_mod')}:kernel"], ["triton", "torch"], target=TranslatorTarget.GENERIC)
    expected = R"""
import math

def common_util() -> int:
    return math.prod([11, 33])


def helper() -> None:
    common_util()


def kernel() -> None:
    helper()
    """
    assert_code_equal(output, expected)
```
- **EN:** Prepares or updates state through `pkg`, `mod`, `output`, `expected`. Invokes `_make_package`, `slice_kernel`, `mod`, `assert_code_equal` to execute the test logic.
- **CN:** 通过 `pkg`、`mod`、`output`、`expected` 准备或更新状态。 调用 `_make_package`、`slice_kernel`、`mod`、`assert_code_equal` 执行测试逻辑。

### Lines 563-565

```python


def test_slice_kernel_function_module_relative_import(tmp_path):
```
- **EN:** Defines the test function `test_slice_kernel_function_module_relative_import`. Parameters: `tmp_path`. Key calls include `_make_package`, `slice_kernel`, `assert_code_equal`, `mod`.
- **CN:** 定义测试函数 `test_slice_kernel_function_module_relative_import`。 参数：`tmp_path`。 关键调用包括 `_make_package`、`slice_kernel`、`assert_code_equal`、`mod`。

#### Lines 566-604

```python
    pkg, mod = _make_package(
        tmp_path,
        {
            "lib_foo.py":
            """
                import math

                def common_util() -> int:
                    return math.prod([11, 33])
            """,
            "kernel_mod.py":
            """
                def helper() -> None:
                    from . import lib_foo

                    lib_foo.common_util()

                def kernel() -> None:
                    helper()
            """,
        },
    )

    output = slice_kernel([f"{mod('kernel_mod')}:kernel"], ["triton", "torch"], target=TranslatorTarget.GENERIC)
    expected = R"""
import math

def common_util() -> int:
    return math.prod([11, 33])


def helper() -> None:
    common_util()


def kernel() -> None:
    helper()
    """
    assert_code_equal(output, expected)
```
- **EN:** Prepares or updates state through `pkg`, `mod`, `output`, `expected`. Invokes `_make_package`, `slice_kernel`, `mod`, `assert_code_equal` to execute the test logic.
- **CN:** 通过 `pkg`、`mod`、`output`、`expected` 准备或更新状态。 调用 `_make_package`、`slice_kernel`、`mod`、`assert_code_equal` 执行测试逻辑。

### Lines 605-607

```python


def test_slice_kernel_function_module_relative_import_leaf(tmp_path):
```
- **EN:** Defines the test function `test_slice_kernel_function_module_relative_import_leaf`. Parameters: `tmp_path`. Key calls include `_make_package`, `slice_kernel`, `assert_code_equal`, `mod`.
- **CN:** 定义测试函数 `test_slice_kernel_function_module_relative_import_leaf`。 参数：`tmp_path`。 关键调用包括 `_make_package`、`slice_kernel`、`assert_code_equal`、`mod`。

#### Lines 608-643

```python
    pkg, mod = _make_package(
        tmp_path,
        {
            "lib_foo.py":
            """
                import math

                def common_util() -> int:
                    return math.prod([11, 33])
            """,
            "kernel_mod.py":
            """
                def helper() -> None:
                    from . import lib_foo

                    lib_foo.common_util()

                def kernel() -> None:
                    helper()
            """,
        },
    )

    output = slice_kernel([f"{mod('kernel_mod')}:kernel"], ["triton", "torch", mod("lib_foo")],
                          target=TranslatorTarget.GENERIC)
    expected = f"""
import {pkg}.lib_foo

def helper() -> None:
    {pkg}.lib_foo.common_util()


def kernel() -> None:
    helper()
    """
    assert_code_equal(output, expected)
```
- **EN:** Prepares or updates state through `pkg`, `mod`, `output`, `expected`. Invokes `_make_package`, `slice_kernel`, `mod`, `assert_code_equal` to execute the test logic.
- **CN:** 通过 `pkg`、`mod`、`output`、`expected` 准备或更新状态。 调用 `_make_package`、`slice_kernel`、`mod`、`assert_code_equal` 执行测试逻辑。

### Lines 644-646

```python


def test_slice_kernel_treats_assign_targets_as_locals(tmp_path):
```
- **EN:** Defines the test function `test_slice_kernel_treats_assign_targets_as_locals`. Parameters: `tmp_path`. Key calls include `_make_package`, `slice_kernel`, `assert_code_equal`, `mod`.
- **CN:** 定义测试函数 `test_slice_kernel_treats_assign_targets_as_locals`。 参数：`tmp_path`。 关键调用包括 `_make_package`、`slice_kernel`、`assert_code_equal`、`mod`。

#### Lines 647-668

```python
    pkg, mod = _make_package(
        tmp_path,
        {
            "kernel_mod.py":
            """
                def helper() -> int:
                    return 1

                def kernel() -> int:
                    helper = lambda: 2
                    return helper()
            """,
        },
    )

    output = slice_kernel([f"{mod('kernel_mod')}:kernel"], ["triton", "torch"], target=TranslatorTarget.GENERIC)
    expected = R"""
def kernel() -> int:
    helper = lambda: 2
    return helper()
    """
    assert_code_equal(output, expected)
```
- **EN:** Prepares or updates state through `pkg`, `mod`, `output`, `expected`. Invokes `_make_package`, `slice_kernel`, `mod`, `assert_code_equal` to execute the test logic.
- **CN:** 通过 `pkg`、`mod`、`output`、`expected` 准备或更新状态。 调用 `_make_package`、`slice_kernel`、`mod`、`assert_code_equal` 执行测试逻辑。

### Lines 669-671

```python


def test_slice_kernel_treats_annassign_targets_as_locals(tmp_path):
```
- **EN:** Defines the test function `test_slice_kernel_treats_annassign_targets_as_locals`. Parameters: `tmp_path`. Key calls include `_make_package`, `slice_kernel`, `assert_code_equal`, `mod`.
- **CN:** 定义测试函数 `test_slice_kernel_treats_annassign_targets_as_locals`。 参数：`tmp_path`。 关键调用包括 `_make_package`、`slice_kernel`、`assert_code_equal`、`mod`。

#### Lines 672-692

```python
    pkg, mod = _make_package(
        tmp_path,
        {
            "kernel_mod.py":
            """
                value = 7

                def kernel() -> int:
                    value: int = 3
                    return value
            """,
        },
    )

    output = slice_kernel([f"{mod('kernel_mod')}:kernel"], ["triton", "torch"], target=TranslatorTarget.GENERIC)
    expected = R"""
def kernel() -> int:
    value: int = 3
    return value
    """
    assert_code_equal(output, expected)
```
- **EN:** Prepares or updates state through `pkg`, `mod`, `output`, `expected`. Invokes `_make_package`, `slice_kernel`, `mod`, `assert_code_equal` to execute the test logic.
- **CN:** 通过 `pkg`、`mod`、`output`、`expected` 准备或更新状态。 调用 `_make_package`、`slice_kernel`、`mod`、`assert_code_equal` 执行测试逻辑。

### Lines 693-695

```python


def test_slice_kernel_treats_assign_and_annassign_targets_as_locals(tmp_path):
```
- **EN:** Defines the test function `test_slice_kernel_treats_assign_and_annassign_targets_as_locals`. Parameters: `tmp_path`. Key calls include `_make_package`, `slice_kernel`, `assert_code_equal`, `mod`. This scope touches Triton compilation or JIT kernels.
- **CN:** 定义测试函数 `test_slice_kernel_treats_assign_and_annassign_targets_as_locals`。 参数：`tmp_path`。 关键调用包括 `_make_package`、`slice_kernel`、`assert_code_equal`、`mod`。 该作用域涉及Triton 编译或 JIT kernel。

#### Lines 696-730

```python
    pkg, mod = _make_package(
        tmp_path,
        {
            "kernel_mod.py":
            """
                import triton

                def global_assign() -> int:
                    return 1

                def global_annassign() -> int:
                    return 2

                @triton.jit
                def kernel() -> tuple[int, int]:
                    global_assign = 3
                    copied = global_assign
                    global_annassign: int = copied + 1
                    return copied, global_annassign
            """,
        },
    )

    output = slice_kernel([f"{mod('kernel_mod')}:kernel"], ["triton", "torch"], target=TranslatorTarget.GENERIC)
    expected = R"""
import triton

@triton.jit
def kernel() -> tuple[int, int]:
    global_assign = 3
    copied = global_assign
    global_annassign: int = copied + 1
    return (copied, global_annassign)
    """
    assert_code_equal(output, expected)
```
- **EN:** Prepares or updates state through `pkg`, `mod`, `output`, `expected`. Invokes `_make_package`, `slice_kernel`, `mod`, `assert_code_equal` to execute the test logic. Relevant themes: Triton compilation or JIT kernels.
- **CN:** 通过 `pkg`、`mod`、`output`、`expected` 准备或更新状态。 调用 `_make_package`、`slice_kernel`、`mod`、`assert_code_equal` 执行测试逻辑。 相关主题：Triton 编译或 JIT kernel。

### Lines 731-733

```python


def test_slice_kernel_translate_to_gluon_avoids_double_descriptor_wrap(tmp_path):
```
- **EN:** Defines the test function `test_slice_kernel_translate_to_gluon_avoids_double_descriptor_wrap`. Parameters: `tmp_path`. Key calls include `_make_package`, `slice_kernel`, `assert_code_equal`, `mod`. This scope touches tensor/descriptor metadata.
- **CN:** 定义测试函数 `test_slice_kernel_translate_to_gluon_avoids_double_descriptor_wrap`。 参数：`tmp_path`。 关键调用包括 `_make_package`、`slice_kernel`、`assert_code_equal`、`mod`。 该作用域涉及张量/描述符元数据。

#### Lines 734-762

```python
    pkg, mod = _make_package(
        tmp_path,
        {
            "kernel_mod.py":
            """
                from triton.tools.tensor_descriptor import TensorDescriptor

                def convert_host_descriptor(desc):
                    return desc

                def kernel(t):
                    return convert_host_descriptor(TensorDescriptor.from_tensor(t, [16, 16]))
            """,
        },
    )

    output = slice_kernel([f"{mod('kernel_mod')}:kernel"], ["triton", "torch"], translate_to_gluon=True,
                          target=TranslatorTarget.GENERIC)
    expected = R"""
from triton.tools.tensor_descriptor import TensorDescriptor

def convert_host_descriptor(desc):
    return desc


def kernel(t):
    return convert_host_descriptor(TensorDescriptor.from_tensor(t, [16, 16]))
    """
    assert_code_equal(output, expected)
```
- **EN:** Prepares or updates state through `pkg`, `mod`, `output`, `expected`. Invokes `_make_package`, `slice_kernel`, `mod`, `assert_code_equal` to execute the test logic. Relevant themes: tensor/descriptor metadata.
- **CN:** 通过 `pkg`、`mod`、`output`、`expected` 准备或更新状态。 调用 `_make_package`、`slice_kernel`、`mod`、`assert_code_equal` 执行测试逻辑。 相关主题：张量/描述符元数据。

### Lines 763-765

```python


def test_translate_to_gluon_explicit_expand_dims_rewrites_layout(tmp_path):
```
- **EN:** Defines the test function `test_translate_to_gluon_explicit_expand_dims_rewrites_layout`. Parameters: `tmp_path`. Key calls include `_make_package`, `slice_kernel`, `assert_code_equal`, `mod`. This scope touches Triton compilation or JIT kernels, Triton language operations, layout transformation reasoning.
- **CN:** 定义测试函数 `test_translate_to_gluon_explicit_expand_dims_rewrites_layout`。 参数：`tmp_path`。 关键调用包括 `_make_package`、`slice_kernel`、`assert_code_equal`、`mod`。 该作用域涉及Triton 编译或 JIT kernel、Triton language 操作、布局变换推理。

#### Lines 766-796

```python
    pkg, mod = _make_package(
        tmp_path,
        {
            "kernel_mod.py":
            """
                import triton
                import triton.language as tl

                @triton.jit
                def kernel(out_ptr, BLOCK: tl.constexpr):
                    offsets = tl.arange(0, BLOCK)
                    expanded = tl.expand_dims(offsets, 0)
                    tl.store(out_ptr + expanded, expanded)
            """,
        },
    )

    output = slice_kernel([f"{mod('kernel_mod')}:kernel"], ["triton"], translate_to_gluon=True,
                          target=TranslatorTarget.GENERIC)
    expected = R"""
import triton.experimental.gluon.language as gl
import triton.tools.triton_to_gluon_translator.common_helpers
import triton.experimental.gluon as gluon

@gluon.jit
def kernel(out_ptr, BLOCK: gl.constexpr):
    offsets = triton.tools.triton_to_gluon_translator.common_helpers.tl_arange(0, BLOCK)
    expanded = gl.expand_dims(triton.tools.triton_to_gluon_translator.common_helpers.convert_to_expand_dims_layout(offsets, [0]), 0)
    gl.store(out_ptr + expanded, expanded)
    """
    assert_code_equal(output, expected)
```
- **EN:** Prepares or updates state through `pkg`, `mod`, `output`, `expected`. Invokes `_make_package`, `slice_kernel`, `mod`, `assert_code_equal` to execute the test logic. Relevant themes: Triton compilation or JIT kernels, Triton language operations, layout transformation reasoning.
- **CN:** 通过 `pkg`、`mod`、`output`、`expected` 准备或更新状态。 调用 `_make_package`、`slice_kernel`、`mod`、`assert_code_equal` 执行测试逻辑。 相关主题：Triton 编译或 JIT kernel、Triton language 操作、布局变换推理。

### Lines 797-799

```python


def test_translate_to_gluon_member_fn_expand_dims_rewrites_layout(tmp_path):
```
- **EN:** Defines the test function `test_translate_to_gluon_member_fn_expand_dims_rewrites_layout`. Parameters: `tmp_path`. Key calls include `_make_package`, `slice_kernel`, `assert_code_equal`, `mod`. This scope touches Triton compilation or JIT kernels, Triton language operations, layout transformation reasoning.
- **CN:** 定义测试函数 `test_translate_to_gluon_member_fn_expand_dims_rewrites_layout`。 参数：`tmp_path`。 关键调用包括 `_make_package`、`slice_kernel`、`assert_code_equal`、`mod`。 该作用域涉及Triton 编译或 JIT kernel、Triton language 操作、布局变换推理。

#### Lines 800-830

```python
    pkg, mod = _make_package(
        tmp_path,
        {
            "kernel_mod.py":
            """
                import triton
                import triton.language as tl

                @triton.jit
                def kernel(out_ptr, BLOCK: tl.constexpr):
                    offsets = tl.arange(0, BLOCK)
                    expanded = offsets.expand_dims(0)
                    tl.store(out_ptr + expanded, expanded)
            """,
        },
    )

    output = slice_kernel([f"{mod('kernel_mod')}:kernel"], ["triton"], translate_to_gluon=True,
                          target=TranslatorTarget.GENERIC)
    expected = R"""
import triton.experimental.gluon.language as gl
import triton.tools.triton_to_gluon_translator.common_helpers
import triton.experimental.gluon as gluon

@gluon.jit
def kernel(out_ptr, BLOCK: gl.constexpr):
    offsets = triton.tools.triton_to_gluon_translator.common_helpers.tl_arange(0, BLOCK)
    expanded = triton.tools.triton_to_gluon_translator.common_helpers.convert_to_expand_dims_layout(offsets, [0]).expand_dims(0)
    gl.store(out_ptr + expanded, expanded)
    """
    assert_code_equal(output, expected)
```
- **EN:** Prepares or updates state through `pkg`, `mod`, `output`, `expected`. Invokes `_make_package`, `slice_kernel`, `mod`, `assert_code_equal` to execute the test logic. Relevant themes: Triton compilation or JIT kernels, Triton language operations, layout transformation reasoning.
- **CN:** 通过 `pkg`、`mod`、`output`、`expected` 准备或更新状态。 调用 `_make_package`、`slice_kernel`、`mod`、`assert_code_equal` 执行测试逻辑。 相关主题：Triton 编译或 JIT kernel、Triton language 操作、布局变换推理。

### Lines 831-833

```python


def test_slice_kernel_public_imports():
```
- **EN:** Defines the test function `test_slice_kernel_public_imports`. Key calls include `callable`. This scope touches tensor/descriptor metadata.
- **CN:** 定义测试函数 `test_slice_kernel_public_imports`。 关键调用包括 `callable`。 该作用域涉及张量/描述符元数据。

#### Lines 834-837

```python
    from triton.tools.triton_to_gluon_translator.slice_kernel import slice_kernel as new_slice_kernel
    from triton.tools.triton_to_gluon_translator.translator import translate_paths
    from triton.tools.triton_to_gluon_translator.translator import convert_triton_to_gluon
    from triton.tools.triton_to_gluon_translator.nvidia_helpers import convert_host_descriptor
```
- **EN:** Imports the modules used in this scope: `triton.tools.triton_to_gluon_translator.slice_kernel`, `triton.tools.triton_to_gluon_translator.translator`, `triton.tools.triton_to_gluon_translator.nvidia_helpers`. Relevant themes: tensor/descriptor metadata.
- **CN:** 导入此作用域使用的模块：`triton.tools.triton_to_gluon_translator.slice_kernel`、`triton.tools.triton_to_gluon_translator.translator`、`triton.tools.triton_to_gluon_translator.nvidia_helpers`。 相关主题：张量/描述符元数据。

#### Lines 838-842

```python

    assert callable(new_slice_kernel)
    assert callable(translate_paths)
    assert callable(convert_triton_to_gluon)
    assert callable(convert_host_descriptor)
```
- **EN:** Invokes `callable` to execute the test logic. Validates behavior with 4 assertion(s). Relevant themes: tensor/descriptor metadata.
- **CN:** 调用 `callable` 执行测试逻辑。 通过 4 个断言验证行为。 相关主题：张量/描述符元数据。

## Key Concepts / 关键概念

- **EN:** Top-level scopes such as `clean_import_state`, `_make_package`, `assert_code_equal`, `test_slice_kernel_basic_module_slicing`, `test_slice_kernel_does_not_treat_site_packages_as_stdlib`, `test_slice_kernel_supports_injected_decorator_matchers`, `test_slice_kernel_translate_to_gluon_keeps_tensor_method_rewrites`, `test_slice_kernel_translate_to_gluon_inlines_descriptor_adapter`
  **CN:** 顶层作用域，例如 `clean_import_state`、`_make_package`、`assert_code_equal`、`test_slice_kernel_basic_module_slicing`、`test_slice_kernel_does_not_treat_site_packages_as_stdlib`、`test_slice_kernel_supports_injected_decorator_matchers`、`test_slice_kernel_translate_to_gluon_keeps_tensor_method_rewrites`、`test_slice_kernel_translate_to_gluon_inlines_descriptor_adapter`
- **EN:** shared fixtures
  **CN:** 共享 fixture
- **EN:** Triton compilation or JIT kernels
  **CN:** Triton 编译或 JIT kernel
- **EN:** Triton language operations
  **CN:** Triton language 操作
- **EN:** PyTorch tensor setup and checks
  **CN:** PyTorch 张量准备与校验
- **EN:** cache management behavior
  **CN:** 缓存管理行为
- **EN:** tensor/descriptor metadata
  **CN:** 张量/描述符元数据
- **EN:** layout transformation reasoning
  **CN:** 布局变换推理

## Dependencies / 依赖关系

- **EN:** External or absolute imports include `ast`, `importlib`, `sys`, `sysconfig`, `textwrap`, `uuid`, `pathlib`, `typing`, `pytest`, `triton.tools.triton_to_gluon_translator.slice_kernel`, `triton.tools.triton_to_gluon_translator.target`, `triton.tools.triton_to_gluon_translator.translator`, and 1 more.
  **CN:** 外部或绝对导入包括 `ast`、`importlib`、`sys`、`sysconfig`、`textwrap`、`uuid`、`pathlib`、`typing`、`pytest`、`triton.tools.triton_to_gluon_translator.slice_kernel`、`triton.tools.triton_to_gluon_translator.target`、`triton.tools.triton_to_gluon_translator.translator` 等另外 1 项。
- **EN:** Execution centers on top-level definitions such as `clean_import_state`, `_make_package`, `assert_code_equal`, `test_slice_kernel_basic_module_slicing`, `test_slice_kernel_does_not_treat_site_packages_as_stdlib`, `test_slice_kernel_supports_injected_decorator_matchers`, `test_slice_kernel_translate_to_gluon_keeps_tensor_method_rewrites`, `test_slice_kernel_translate_to_gluon_inlines_descriptor_adapter`, `test_slice_kernel_binds_local_imports`, `test_slice_kernel_function_import`.
  **CN:** 执行逻辑主要围绕顶层定义展开，例如 `clean_import_state`、`_make_package`、`assert_code_equal`、`test_slice_kernel_basic_module_slicing`、`test_slice_kernel_does_not_treat_site_packages_as_stdlib`、`test_slice_kernel_supports_injected_decorator_matchers`、`test_slice_kernel_translate_to_gluon_keeps_tensor_method_rewrites`、`test_slice_kernel_translate_to_gluon_inlines_descriptor_adapter`、`test_slice_kernel_binds_local_imports`、`test_slice_kernel_function_import`。
- **EN:** Runtime behavior also depends on pytest collection, Python execution semantics, and the imported Triton/PyTorch utilities visible above.
  **CN:** 运行时行为还依赖 pytest 的收集机制、Python 执行语义，以及上文可见的 Triton/PyTorch 工具。
