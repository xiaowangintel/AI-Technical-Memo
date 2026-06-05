# test_frontend.py — Code Analysis / 代码分析

## Source / 来源
- **Path / 路径:** `python/test/gluon/test_frontend.py`
- **EN:** Pytest module covering frontend behavior in Triton's Python tests. It contains 193 top-level definition(s) and 18 imported module reference(s).
- **CN:** 这是一个 pytest 模块，用于覆盖 Triton Python 测试中的 frontend 行为。 该文件包含 193 个顶层定义，以及 18 个导入模块引用。

## Line-by-Line Analysis / 逐行分析

### Lines 1-27

```python
import expecttest
import pytest
import re

from triton.backends.compiler import GPUTarget
from triton.experimental import gluon
from triton.experimental.gluon import language as ttgl
from triton.experimental.gluon.language.nvidia import blackwell
from triton.experimental.gluon.language.nvidia import hopper
from triton.experimental.gluon.language.nvidia.hopper import cluster
from triton.experimental.gluon.language.nvidia.blackwell import mbarrier, tma, TensorMemoryLayout, TensorMemoryScalesLayout, async_copy
from triton.experimental.gluon.nvidia.hopper import TensorDescriptor
from triton.experimental.gluon.language.amd import _layouts as amd_layouts
from triton.experimental.gluon.language.amd.cdna4 import async_copy as cdna4_async_copy
from triton.experimental.gluon.language.amd.gfx1250 import async_copy as gfx1250_async_copy
from triton.experimental.gluon.language.amd.gfx1250 import mbarrier as gfx1250_mbarrier
from triton.experimental.gluon.language.amd.gfx1250 import cluster as gfx1250_cluster
from triton.experimental.gluon.language.amd.gfx1250 import (
    PartitionedSharedLayout,
    make_partitioned_dot_layouts,
)
from triton.experimental.gluon.language.extra import libdevice

from triton._filecheck import filecheck_test, run_parser
from triton.runtime.jit import MockTensor
import triton.language as tl
from triton.compiler.errors import CompilationError, CompileTimeAssertionFailure
```
- **EN:** Imports the modules used in this scope: `expecttest`, `pytest`, `re`, `triton.backends.compiler`, `triton.experimental`, `triton.experimental.gluon`, `triton.experimental.gluon.language.nvidia`, `triton.experimental.gluon.language.nvidia.hopper`, `triton.experimental.gluon.language.nvidia.blackwell`, `triton.experimental.gluon.nvidia.hopper`, and 8 more. Relevant themes: Triton compilation or JIT kernels, Triton language operations, tensor/descriptor metadata, layout transformation reasoning.
- **CN:** 导入此作用域使用的模块：`expecttest`、`pytest`、`re`、`triton.backends.compiler`、`triton.experimental`、`triton.experimental.gluon`、`triton.experimental.gluon.language.nvidia`、`triton.experimental.gluon.language.nvidia.hopper`、`triton.experimental.gluon.language.nvidia.blackwell`、`triton.experimental.gluon.nvidia.hopper` 等另外 8 项。 相关主题：Triton 编译或 JIT kernel、Triton language 操作、张量/描述符元数据、布局变换推理。

### Lines 28-43

```python

TARGET_PAT = re.compile('ttg.target = "[^"]*"')
# HIP backend can add this attribute to function parameters
PTRRANGE_PAT = re.compile('(, )?tt.pointer_range = 32 : i32')
LIBDEVICE_PAT = re.compile('{libname = "", libpath = "", pure = true, symbol = "__.*"}')

BLACKWELL_TARGET = GPUTarget("cuda", 100, 32)
HOPPER_TARGET = GPUTarget("cuda", 90, 32)
AMPERE_TARGET = GPUTarget("cuda", 80, 32)
HIP_TARGET_RDNA3 = GPUTarget("hip", "gfx1100", 32)
HIP_TARGET_RDNA4 = GPUTarget("hip", "gfx1200", 32)
HIP_TARGET_CDNA3 = GPUTarget("hip", "gfx942", 64)
HIP_TARGET_CDNA4 = GPUTarget("hip", "gfx950", 64)
HIP_TARGET_GFX1250 = GPUTarget("hip", "gfx1250", 32)

ALL_TARGETS = [AMPERE_TARGET, HOPPER_TARGET, BLACKWELL_TARGET, HIP_TARGET_RDNA4]
```
- **EN:** Prepares or updates state through `TARGET_PAT`, `PTRRANGE_PAT`, `LIBDEVICE_PAT`, `BLACKWELL_TARGET`, `HOPPER_TARGET`, `AMPERE_TARGET`, `HIP_TARGET_RDNA3`, `HIP_TARGET_RDNA4`, and 4 more. Invokes `re.compile`, `GPUTarget` to execute the test logic. Relevant themes: libdevice coverage.
- **CN:** 通过 `TARGET_PAT`、`PTRRANGE_PAT`、`LIBDEVICE_PAT`、`BLACKWELL_TARGET`、`HOPPER_TARGET`、`AMPERE_TARGET`、`HIP_TARGET_RDNA3`、`HIP_TARGET_RDNA4` 等另外 4 项 准备或更新状态。 调用 `re.compile`、`GPUTarget` 执行测试逻辑。 相关主题：libdevice 覆盖。

### Lines 44-46

```python


def anonymize_ir(ir):
```
- **EN:** Defines the helper function `anonymize_ir`. Parameters: `ir`. Key calls include `TARGET_PAT.sub`, `PTRRANGE_PAT.sub`, `LIBDEVICE_PAT.sub`. This scope touches libdevice coverage.
- **CN:** 定义辅助函数 `anonymize_ir`。 参数：`ir`。 关键调用包括 `TARGET_PAT.sub`、`PTRRANGE_PAT.sub`、`LIBDEVICE_PAT.sub`。 该作用域涉及libdevice 覆盖。

#### Lines 47-50

```python
    ir = TARGET_PAT.sub('ttg.target = "..."', ir)
    ir = PTRRANGE_PAT.sub('', ir)
    ir = LIBDEVICE_PAT.sub('{libname = "", libpath = "", pure = true, symbol = "..."}', ir)
    return ir
```
- **EN:** Prepares or updates state through `ir`. Invokes `TARGET_PAT.sub`, `PTRRANGE_PAT.sub`, `LIBDEVICE_PAT.sub` to execute the test logic. Relevant themes: libdevice coverage.
- **CN:** 通过 `ir` 准备或更新状态。 调用 `TARGET_PAT.sub`、`PTRRANGE_PAT.sub`、`LIBDEVICE_PAT.sub` 执行测试逻辑。 相关主题：libdevice 覆盖。

### Lines 51-53

```python


def make_args(*args, **kwargs):
```
- **EN:** Defines the helper function `make_args`. Parameters: `*args`, `**kwargs`.
- **CN:** 定义辅助函数 `make_args`。 参数：`*args`、`**kwargs`。

#### Lines 54-54

```python
    return args, kwargs
```
- **EN:** Carries supporting logic for the surrounding test or helper scope.
- **CN:** 为周围的测试或辅助作用域提供支撑逻辑。

### Lines 55-58

```python


@gluon.jit
def convert_layout_kernel(XBLOCK: ttgl.constexpr, layout_a: ttgl.constexpr, layout_b: ttgl.constexpr):
```
- **EN:** Defines the helper function `convert_layout_kernel`. Decorators: `gluon.jit`. Parameters: `XBLOCK`, `layout_a`, `layout_b`. Key calls include `ttgl.arange`, `ttgl.convert_layout`. This scope touches layout transformation reasoning.
- **CN:** 定义辅助函数 `convert_layout_kernel`。 装饰器：`gluon.jit`。 参数：`XBLOCK`、`layout_a`、`layout_b`。 关键调用包括 `ttgl.arange`、`ttgl.convert_layout`。 该作用域涉及布局变换推理。

#### Lines 59-60

```python
    x = ttgl.arange(0, XBLOCK, layout=layout_a)
    res = ttgl.convert_layout(x, layout_b)  # noqa: F841
```
- **EN:** Prepares or updates state through `x`, `res`. Invokes `ttgl.arange`, `ttgl.convert_layout` to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `x`、`res` 准备或更新状态。 调用 `ttgl.arange`、`ttgl.convert_layout` 执行测试逻辑。 相关主题：布局变换推理。

### Lines 61-64

```python


@pytest.mark.parametrize("target", ALL_TARGETS)
def test_convert_layout(target):
```
- **EN:** Defines the test function `test_convert_layout`. Decorators: `pytest.mark.parametrize('target', ALL_TARGETS)`. Parameters: `target`. Key calls include `pytest.mark.parametrize`, `ttgl.BlockedLayout`, `ttgl.SliceLayout`, `run_parser`, `expecttest.assert_expected_inline`, `anonymize_ir`, and 2 more. This scope touches pytest parametrization, debugging and inspection paths, layout transformation reasoning.
- **CN:** 定义测试函数 `test_convert_layout`。 装饰器：`pytest.mark.parametrize('target', ALL_TARGETS)`。 参数：`target`。 关键调用包括 `pytest.mark.parametrize`、`ttgl.BlockedLayout`、`ttgl.SliceLayout`、`run_parser`、`expecttest.assert_expected_inline`、`anonymize_ir` 等另外 2 项。 该作用域涉及pytest 参数化、调试与检查路径、布局变换推理。

#### Lines 65-84

```python
    layout_a = ttgl.BlockedLayout(size_per_thread=[1], threads_per_warp=[32], warps_per_cta=[4], order=[0])
    layout_b = ttgl.SliceLayout(
        1, ttgl.BlockedLayout(size_per_thread=[1, 1], threads_per_warp=[1, 32], warps_per_cta=[1, 4], order=[1, 0]))
    mod = run_parser(
        convert_layout_kernel,
        *make_args(128, layout_a, layout_b, num_warps=layout_a.warps_per_cta[0]),
        target=target,
    )
    expecttest.assert_expected_inline(
        anonymize_ir(mod.str_nodebug()), """\
#blocked = #ttg.blocked<{sizePerThread = [1], threadsPerWarp = [32], warpsPerCTA = [4], order = [0]}>
#blocked1 = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [1, 32], warpsPerCTA = [1, 4], order = [1, 0]}>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "...", "ttg.threads-per-warp" = 32 : i32} {
  tt.func public @convert_layout_kernel() attributes {noinline = false} {
    %0 = tt.make_range {end = 128 : i32, start = 0 : i32} : tensor<128xi32, #blocked>
    %1 = ttg.convert_layout %0 : tensor<128xi32, #blocked> -> tensor<128xi32, #ttg.slice<{dim = 1, parent = #blocked1}>>
    tt.return
  }
}
""")
```
- **EN:** Prepares or updates state through `layout_a`, `layout_b`, `mod`. Invokes `ttgl.BlockedLayout`, `ttgl.SliceLayout`, `run_parser`, `make_args`, `expecttest.assert_expected_inline`, `anonymize_ir`, and 1 more to execute the test logic. Relevant themes: debugging and inspection paths, layout transformation reasoning.
- **CN:** 通过 `layout_a`、`layout_b`、`mod` 准备或更新状态。 调用 `ttgl.BlockedLayout`、`ttgl.SliceLayout`、`run_parser`、`make_args`、`expecttest.assert_expected_inline`、`anonymize_ir` 等另外 1 项 执行测试逻辑。 相关主题：调试与检查路径、布局变换推理。

### Lines 85-88

```python


@gluon.jit
def simple_ops_kernel(arg: tl.int32):
```
- **EN:** Defines the helper function `simple_ops_kernel`. Decorators: `gluon.jit`. Parameters: `arg`. Key calls include `ttgl.assume`, `ttgl.device_print`. This scope touches Triton language operations, debugging and inspection paths.
- **CN:** 定义辅助函数 `simple_ops_kernel`。 装饰器：`gluon.jit`。 参数：`arg`。 关键调用包括 `ttgl.assume`、`ttgl.device_print`。 该作用域涉及Triton language 操作、调试与检查路径。

#### Lines 89-90

```python
    ttgl.assume(arg > 1)
    ttgl.device_print("arg: ", arg)
```
- **EN:** Invokes `ttgl.assume`, `ttgl.device_print` to execute the test logic. Relevant themes: debugging and inspection paths.
- **CN:** 调用 `ttgl.assume`、`ttgl.device_print` 执行测试逻辑。 相关主题：调试与检查路径。

### Lines 91-94

```python


@pytest.mark.parametrize("target", ALL_TARGETS)
def test_simple_ops(target):
```
- **EN:** Defines the test function `test_simple_ops`. Decorators: `pytest.mark.parametrize('target', ALL_TARGETS)`. Parameters: `target`. Key calls include `pytest.mark.parametrize`, `run_parser`, `expecttest.assert_expected_inline`, `anonymize_ir`, `make_args`, `mod.str_nodebug`. This scope touches pytest parametrization, debugging and inspection paths.
- **CN:** 定义测试函数 `test_simple_ops`。 装饰器：`pytest.mark.parametrize('target', ALL_TARGETS)`。 参数：`target`。 关键调用包括 `pytest.mark.parametrize`、`run_parser`、`expecttest.assert_expected_inline`、`anonymize_ir`、`make_args`、`mod.str_nodebug`。 该作用域涉及pytest 参数化、调试与检查路径。

#### Lines 95-112

```python
    arg = 100
    mod = run_parser(
        simple_ops_kernel,
        *make_args(arg),
        target=target,
    )
    expecttest.assert_expected_inline(
        anonymize_ir(mod.str_nodebug()), """\
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "...", "ttg.threads-per-warp" = 32 : i32} {
  tt.func public @simple_ops_kernel(%arg0: i32) attributes {noinline = false} {
    %c1_i32 = arith.constant 1 : i32
    %0 = arith.cmpi sgt, %arg0, %c1_i32 : i32
    llvm.intr.assume %0 : i1
    tt.print " arg: " {hex = false, isSigned = array<i32: 1>} : %arg0 : i32
    tt.return
  }
}
""")
```
- **EN:** Prepares or updates state through `arg`, `mod`. Invokes `run_parser`, `make_args`, `expecttest.assert_expected_inline`, `anonymize_ir`, `mod.str_nodebug` to execute the test logic. Relevant themes: debugging and inspection paths.
- **CN:** 通过 `arg`、`mod` 准备或更新状态。 调用 `run_parser`、`make_args`、`expecttest.assert_expected_inline`、`anonymize_ir`、`mod.str_nodebug` 执行测试逻辑。 相关主题：调试与检查路径。

### Lines 113-119

```python


@filecheck_test
@gluon.jit
def test_histogram_frontend():
    # CHECK: #blocked = #ttg.blocked
    # CHECK-LABEL: test_histogram_frontend
```
- **EN:** Defines the test function `test_histogram_frontend`. Decorators: `filecheck_test`, `gluon.jit`. Key calls include `ttgl.BlockedLayout`, `ttgl.arange`, `ttgl.histogram`. This scope touches layout transformation reasoning.
- **CN:** 定义测试函数 `test_histogram_frontend`。 装饰器：`filecheck_test`、`gluon.jit`。 关键调用包括 `ttgl.BlockedLayout`、`ttgl.arange`、`ttgl.histogram`。 该作用域涉及布局变换推理。

#### Lines 120-124

```python
    layout: ttgl.constexpr = ttgl.BlockedLayout([1], [32], [4], [0])
    x = ttgl.arange(0, 256, layout=layout)
    m = x < 128
    # CHECK: tt.histogram %{{.*}}, %{{.*}} : tensor<256xi32, #blocked> -> tensor<512xi32, #blocked>
    _ = ttgl.histogram(x, 512, mask=m, layout=layout)
```
- **EN:** Prepares or updates state through `layout`, `x`, `m`, `_`. Invokes `ttgl.BlockedLayout`, `ttgl.arange`, `ttgl.histogram` to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `layout`、`x`、`m`、`_` 准备或更新状态。 调用 `ttgl.BlockedLayout`、`ttgl.arange`、`ttgl.histogram` 执行测试逻辑。 相关主题：布局变换推理。

### Lines 125-130

```python


@filecheck_test
@gluon.jit
def test_convert_layout_assert_trivial():
    # CHECK: test_convert_layout_assert_trivial
```
- **EN:** Defines the test function `test_convert_layout_assert_trivial`. Decorators: `filecheck_test`, `gluon.jit`. Key calls include `ttgl.BlockedLayout`, `ttgl.SliceLayout`, `ttgl.arange`, `ttgl.convert_layout`. This scope touches layout transformation reasoning.
- **CN:** 定义测试函数 `test_convert_layout_assert_trivial`。 装饰器：`filecheck_test`、`gluon.jit`。 关键调用包括 `ttgl.BlockedLayout`、`ttgl.SliceLayout`、`ttgl.arange`、`ttgl.convert_layout`。 该作用域涉及布局变换推理。

#### Lines 131-137

```python
    parent_layout: ttgl.constexpr = ttgl.BlockedLayout([1, 128], [32, 1], [4, 1], [0, 1])
    slice_layout: ttgl.constexpr = ttgl.SliceLayout(1, parent_layout)
    equiv_layout: ttgl.constexpr = ttgl.BlockedLayout([1], [32], [4], [0])

    value = ttgl.arange(0, 128, layout=slice_layout)
    # CHECK: ttg.convert_layout
    ttgl.convert_layout(value, equiv_layout, assert_trivial=True)
```
- **EN:** Prepares or updates state through `parent_layout`, `slice_layout`, `equiv_layout`, `value`. Invokes `ttgl.BlockedLayout`, `ttgl.SliceLayout`, `ttgl.arange`, `ttgl.convert_layout` to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `parent_layout`、`slice_layout`、`equiv_layout`、`value` 准备或更新状态。 调用 `ttgl.BlockedLayout`、`ttgl.SliceLayout`、`ttgl.arange`、`ttgl.convert_layout` 执行测试逻辑。 相关主题：布局变换推理。

### Lines 138-143

```python


@pytest.mark.parametrize("target", ALL_TARGETS)
def test_convert_layout_not_trivial(target):

    @gluon.jit
```
- **EN:** Defines the test function `test_convert_layout_not_trivial`. Decorators: `pytest.mark.parametrize('target', ALL_TARGETS)`. Parameters: `target`. Nested definitions in this scope: `kernel`. Key calls include `pytest.mark.parametrize`, `ttgl.arange`, `ttgl.convert_layout`, `pytest.raises`, `ttgl.BlockedLayout`, `run_parser`, and 2 more. This scope touches pytest parametrization, layout transformation reasoning.
- **CN:** 定义测试函数 `test_convert_layout_not_trivial`。 装饰器：`pytest.mark.parametrize('target', ALL_TARGETS)`。 参数：`target`。 该作用域中的嵌套定义：`kernel`。 关键调用包括 `pytest.mark.parametrize`、`ttgl.arange`、`ttgl.convert_layout`、`pytest.raises`、`ttgl.BlockedLayout`、`run_parser` 等另外 2 项。 该作用域涉及pytest 参数化、布局变换推理。

#### Lines 143-144

```python
    @gluon.jit
    def kernel(src_layout: ttgl.constexpr, dst_layout: ttgl.constexpr):
```
- **EN:** Defines the helper function `kernel`. Decorators: `gluon.jit`. Parameters: `src_layout`, `dst_layout`. Key calls include `ttgl.arange`, `ttgl.convert_layout`. This scope touches layout transformation reasoning.
- **CN:** 定义辅助函数 `kernel`。 装饰器：`gluon.jit`。 参数：`src_layout`、`dst_layout`。 关键调用包括 `ttgl.arange`、`ttgl.convert_layout`。 该作用域涉及布局变换推理。

##### Lines 145-146

```python
        value = ttgl.arange(0, 128, layout=src_layout)
        ttgl.convert_layout(value, dst_layout, assert_trivial=True)
```
- **EN:** Prepares or updates state through `value`. Invokes `ttgl.arange`, `ttgl.convert_layout` to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `value` 准备或更新状态。 调用 `ttgl.arange`、`ttgl.convert_layout` 执行测试逻辑。 相关主题：布局变换推理。

#### Lines 147-151

```python

    with pytest.raises(CompilationError) as e:
        src_layout = ttgl.BlockedLayout([2], [32], [4], [0])
        dst_layout = ttgl.BlockedLayout([1], [32], [4], [0])
        run_parser(kernel, *make_args(src_layout, dst_layout), target=target)
```
- **EN:** Invokes `pytest.raises`, `ttgl.BlockedLayout`, `run_parser`, `make_args` to execute the test logic. Uses `pytest.raises` to confirm expected failure paths. Uses context managers to control resources or expectations. Relevant themes: layout transformation reasoning.
- **CN:** 调用 `pytest.raises`、`ttgl.BlockedLayout`、`run_parser`、`make_args` 执行测试逻辑。 使用 `pytest.raises` 确认预期的失败路径。 使用上下文管理器控制资源或预期行为。 相关主题：布局变换推理。

#### Lines 152-155

```python

    assert "layout conversion from BlockedLayout(size_per_thread=[2]" in str(e.value.__cause__)
    assert "to BlockedLayout(size_per_thread=[1]" in str(e.value.__cause__)
    assert "is not trivial" in str(e.value.__cause__)
```
- **EN:** Validates behavior with 3 assertion(s). Relevant themes: layout transformation reasoning.
- **CN:** 通过 3 个断言验证行为。 相关主题：布局变换推理。

#### Lines 156-160

```python

    with pytest.raises(CompilationError) as e:
        src_layout = ttgl.BlockedLayout([2], [32], [4], [0])
        dst_layout = ttgl.AutoLayout()
        run_parser(kernel, *make_args(src_layout, dst_layout), target=target)
```
- **EN:** Invokes `pytest.raises`, `ttgl.BlockedLayout`, `ttgl.AutoLayout`, `run_parser`, `make_args` to execute the test logic. Uses `pytest.raises` to confirm expected failure paths. Uses context managers to control resources or expectations. Relevant themes: layout transformation reasoning.
- **CN:** 调用 `pytest.raises`、`ttgl.BlockedLayout`、`ttgl.AutoLayout`、`run_parser`、`make_args` 执行测试逻辑。 使用 `pytest.raises` 确认预期的失败路径。 使用上下文管理器控制资源或预期行为。 相关主题：布局变换推理。

#### Lines 161-163

```python

    assert "layout conversion from BlockedLayout(size_per_thread=[2]" in str(e.value.__cause__)
    assert "to AutoLayout() is not trivial" in str(e.value.__cause__)
```
- **EN:** Validates behavior with 2 assertion(s). Relevant themes: layout transformation reasoning.
- **CN:** 通过 2 个断言验证行为。 相关主题：布局变换推理。

#### Lines 164-168

```python

    with pytest.raises(CompilationError) as e:
        src_layout: ttgl.constexpr = ttgl.AutoLayout()
        dst_layout: ttgl.constexpr = ttgl.BlockedLayout([2], [32], [4], [0])
        run_parser(kernel, *make_args(src_layout, dst_layout), target=target)
```
- **EN:** Invokes `pytest.raises`, `ttgl.AutoLayout`, `ttgl.BlockedLayout`, `run_parser`, `make_args` to execute the test logic. Uses `pytest.raises` to confirm expected failure paths. Uses context managers to control resources or expectations. Relevant themes: layout transformation reasoning.
- **CN:** 调用 `pytest.raises`、`ttgl.AutoLayout`、`ttgl.BlockedLayout`、`run_parser`、`make_args` 执行测试逻辑。 使用 `pytest.raises` 确认预期的失败路径。 使用上下文管理器控制资源或预期行为。 相关主题：布局变换推理。

#### Lines 169-172

```python

    assert "layout conversion from AutoLayout()" in str(e.value.__cause__)
    assert "to BlockedLayout(size_per_thread=[2]" in str(e.value.__cause__)
    assert "is not trivial" in str(e.value.__cause__)
```
- **EN:** Validates behavior with 3 assertion(s). Relevant themes: layout transformation reasoning.
- **CN:** 通过 3 个断言验证行为。 相关主题：布局变换推理。

### Lines 173-177

```python


@gluon.jit
def shared_memory_kernel(XBLOCK: ttgl.constexpr, YBLOCK: ttgl.constexpr, layout_a: ttgl.constexpr,
                         layout_b: ttgl.constexpr, smem_layout: ttgl.constexpr):
```
- **EN:** Defines the helper function `shared_memory_kernel`. Decorators: `gluon.jit`. Parameters: `XBLOCK`, `YBLOCK`, `layout_a`, `layout_b`, `smem_layout`. Key calls include `ttgl.allocate_shared_memory`, `ttgl.full`, `ttgl.static_assert`, `mem.load`, `mem.store`, `unused._keep_alive`. This scope touches layout transformation reasoning.
- **CN:** 定义辅助函数 `shared_memory_kernel`。 装饰器：`gluon.jit`。 参数：`XBLOCK`、`YBLOCK`、`layout_a`、`layout_b`、`smem_layout`。 关键调用包括 `ttgl.allocate_shared_memory`、`ttgl.full`、`ttgl.static_assert`、`mem.load`、`mem.store`、`unused._keep_alive`。 该作用域涉及布局变换推理。

#### Lines 178-185

```python
    unused = ttgl.allocate_shared_memory(ttgl.int32, [XBLOCK, YBLOCK], smem_layout)
    a = ttgl.full([XBLOCK, YBLOCK], 0, ttgl.int32, layout_a)
    ttgl.static_assert(a.numel == unused.numel)
    ttgl.static_assert(unused.numel == XBLOCK * YBLOCK)
    mem = ttgl.allocate_shared_memory(ttgl.int32, a.shape, smem_layout, a)
    b = mem.load(layout_b)  # noqa: F841
    mem.store(a)
    unused._keep_alive()
```
- **EN:** Prepares or updates state through `unused`, `a`, `mem`, `b`. Invokes `ttgl.allocate_shared_memory`, `ttgl.full`, `ttgl.static_assert`, `mem.load`, `mem.store`, `unused._keep_alive` to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `unused`、`a`、`mem`、`b` 准备或更新状态。 调用 `ttgl.allocate_shared_memory`、`ttgl.full`、`ttgl.static_assert`、`mem.load`、`mem.store`、`unused._keep_alive` 执行测试逻辑。 相关主题：布局变换推理。

### Lines 186-189

```python


@pytest.mark.parametrize("target", ALL_TARGETS)
def test_shared_memory(target):
```
- **EN:** Defines the test function `test_shared_memory`. Decorators: `pytest.mark.parametrize('target', ALL_TARGETS)`. Parameters: `target`. Key calls include `pytest.mark.parametrize`, `ttgl.BlockedLayout`, `ttgl.NVMMASharedLayout`, `run_parser`, `expecttest.assert_expected_inline`, `anonymize_ir`, and 2 more. This scope touches pytest parametrization, debugging and inspection paths, layout transformation reasoning.
- **CN:** 定义测试函数 `test_shared_memory`。 装饰器：`pytest.mark.parametrize('target', ALL_TARGETS)`。 参数：`target`。 关键调用包括 `pytest.mark.parametrize`、`ttgl.BlockedLayout`、`ttgl.NVMMASharedLayout`、`run_parser`、`expecttest.assert_expected_inline`、`anonymize_ir` 等另外 2 项。 该作用域涉及pytest 参数化、调试与检查路径、布局变换推理。

#### Lines 190-216

```python
    layout_a = ttgl.BlockedLayout(size_per_thread=[1, 1], threads_per_warp=[1, 32], warps_per_cta=[4, 1], order=[1, 0])
    layout_b = ttgl.BlockedLayout(size_per_thread=[1, 4], threads_per_warp=[1, 32], warps_per_cta=[4, 1], order=[1, 0])
    smem_layout = ttgl.NVMMASharedLayout(swizzle_byte_width=128, element_bitwidth=32, rank=2)
    mod = run_parser(
        shared_memory_kernel,
        *make_args(8, 32, layout_a, layout_b, smem_layout, num_warps=layout_a.warps_per_cta[0]),
        target=target,
    )
    expecttest.assert_expected_inline(
        anonymize_ir(mod.str_nodebug()), """\
#blocked = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [1, 32], warpsPerCTA = [4, 1], order = [1, 0]}>
#blocked1 = #ttg.blocked<{sizePerThread = [1, 4], threadsPerWarp = [1, 32], warpsPerCTA = [4, 1], order = [1, 0]}>
#shared = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 32}>
#smem = #ttg.shared_memory
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "...", "ttg.threads-per-warp" = 32 : i32} {
  tt.func public @shared_memory_kernel() attributes {noinline = false} {
    %0 = ttg.local_alloc : () -> !ttg.memdesc<8x32xi32, #shared, #smem, mutable>
    %c0_i32 = arith.constant 0 : i32
    %cst = arith.constant dense<0> : tensor<8x32xi32, #blocked>
    %1 = ttg.local_alloc %cst : (tensor<8x32xi32, #blocked>) -> !ttg.memdesc<8x32xi32, #shared, #smem, mutable>
    %2 = ttg.local_load %1 : !ttg.memdesc<8x32xi32, #shared, #smem, mutable> -> tensor<8x32xi32, #blocked1>
    ttg.local_store %cst, %1 : tensor<8x32xi32, #blocked> -> !ttg.memdesc<8x32xi32, #shared, #smem, mutable>
    ttg.local_dealloc %0 : !ttg.memdesc<8x32xi32, #shared, #smem, mutable>
    tt.return
  }
}
""")
```
- **EN:** Prepares or updates state through `layout_a`, `layout_b`, `smem_layout`, `mod`. Invokes `ttgl.BlockedLayout`, `ttgl.NVMMASharedLayout`, `run_parser`, `make_args`, `expecttest.assert_expected_inline`, `anonymize_ir`, and 1 more to execute the test logic. Relevant themes: debugging and inspection paths, layout transformation reasoning.
- **CN:** 通过 `layout_a`、`layout_b`、`smem_layout`、`mod` 准备或更新状态。 调用 `ttgl.BlockedLayout`、`ttgl.NVMMASharedLayout`、`run_parser`、`make_args`、`expecttest.assert_expected_inline`、`anonymize_ir` 等另外 1 项 执行测试逻辑。 相关主题：调试与检查路径、布局变换推理。

### Lines 217-235

```python


@filecheck_test
@gluon.jit
def test_shared_atomic_scatter_rmw():
    # CHECK: [[SMEM:%.*]] = ttg.local_alloc : () -> !ttg.memdesc<8x32xi32
    # CHECK: [[INDICES:%.*]] = arith.addi
    # CHECK: [[VALUES:%.*]] = arith.constant dense<1> : tensor<8x32xi32, #blocked>
    # CHECK: [[MASK:%.*]] = arith.constant dense<true> : tensor<8x32xi1, #blocked>
    # CHECK: ttg.local_atomic_scatter_rmw add, [[SMEM]][[[INDICES]]], [[VALUES]], [[MASK]]
    # CHECK: ttg.local_atomic_scatter_rmw max, [[SMEM]][[[INDICES]]], [[VALUES]], [[MASK]]
    # CHECK: ttg.local_atomic_scatter_rmw min, [[SMEM]][[[INDICES]]], [[VALUES]], [[MASK]]
    # CHECK: ttg.local_atomic_scatter_rmw and, [[SMEM]][[[INDICES]]], [[VALUES]], [[MASK]]
    # CHECK: ttg.local_atomic_scatter_rmw or, [[SMEM]][[[INDICES]]], [[VALUES]], [[MASK]]
    # CHECK: ttg.local_atomic_scatter_rmw xor, [[SMEM]][[[INDICES]]], [[VALUES]], [[MASK]]
    # CHECK: ttg.local_atomic_scatter_rmw exch, [[SMEM]][[[INDICES]]], [[VALUES]], [[MASK]]
    # CHECK: ttg.local_atomic_scatter_rmw umax
    # CHECK: ttg.local_atomic_scatter_rmw umin
    # CHECK: ttg.local_atomic_scatter_rmw fadd
```
- **EN:** Defines the test function `test_shared_atomic_scatter_rmw`. Decorators: `filecheck_test`, `gluon.jit`. Key calls include `ttgl.BlockedLayout`, `ttgl.NVMMASharedLayout`, `ttgl.allocate_shared_memory`, `smem.store`, `ttgl.full`, `smem.atomic_scatter_add`, and 14 more. This scope touches layout transformation reasoning.
- **CN:** 定义测试函数 `test_shared_atomic_scatter_rmw`。 装饰器：`filecheck_test`、`gluon.jit`。 关键调用包括 `ttgl.BlockedLayout`、`ttgl.NVMMASharedLayout`、`ttgl.allocate_shared_memory`、`smem.store`、`ttgl.full`、`smem.atomic_scatter_add` 等另外 14 项。 该作用域涉及布局变换推理。

#### Lines 236-264

```python
    shape: ttgl.constexpr = [8, 32]
    layout: ttgl.constexpr = ttgl.BlockedLayout([1, 1], [1, 32], [4, 1], [1, 0])
    smem_layout: ttgl.constexpr = ttgl.NVMMASharedLayout(swizzle_byte_width=128, element_bitwidth=32, rank=2)

    smem = ttgl.allocate_shared_memory(ttgl.int32, shape, smem_layout)
    smem.store(ttgl.zeros(shape, ttgl.int32, layout))

    cols = ttgl.arange(0, shape[1], layout=ttgl.SliceLayout(0, layout))[None, :]
    indices = cols + ttgl.zeros(shape, ttgl.int32, layout)
    values = ttgl.full(shape, 1, ttgl.int32, layout)
    mask = ttgl.full(shape, True, ttgl.int1, layout)
    old = smem.atomic_scatter_add(values, indices, axis=1, mask=mask)  # noqa: F841
    old = smem.atomic_scatter_max(values, indices, axis=1, mask=mask)  # noqa: F841
    old = smem.atomic_scatter_min(values, indices, axis=1, mask=mask)  # noqa: F841
    old = smem.atomic_scatter_and(values, indices, axis=1, mask=mask)  # noqa: F841
    old = smem.atomic_scatter_or(values, indices, axis=1, mask=mask)  # noqa: F841
    old = smem.atomic_scatter_xor(values, indices, axis=1, mask=mask)  # noqa: F841
    old = smem.atomic_scatter_xchg(values, indices, axis=1, mask=mask)  # noqa: F841

    smem_u = ttgl.allocate_shared_memory(ttgl.uint32, shape, smem_layout)
    smem_u.store(ttgl.zeros(shape, ttgl.uint32, layout))
    values_u = ttgl.full(shape, 1, ttgl.uint32, layout)
    old = smem_u.atomic_scatter_max(values_u, indices, axis=1, mask=mask)  # noqa: F841
    old = smem_u.atomic_scatter_min(values_u, indices, axis=1, mask=mask)  # noqa: F841

    smem_f = ttgl.allocate_shared_memory(ttgl.float32, shape, smem_layout)
    smem_f.store(ttgl.zeros(shape, ttgl.float32, layout))
    values_f = ttgl.full(shape, 1.0, ttgl.float32, layout)
    old = smem_f.atomic_scatter_add(values_f, indices, axis=1, mask=mask)  # noqa: F841
```
- **EN:** Prepares or updates state through `shape`, `layout`, `smem_layout`, `smem`, `cols`, `indices`, `values`, `mask`, and 5 more. Invokes `ttgl.BlockedLayout`, `ttgl.NVMMASharedLayout`, `ttgl.allocate_shared_memory`, `smem.store`, `ttgl.zeros`, `ttgl.arange`, and 14 more to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `shape`、`layout`、`smem_layout`、`smem`、`cols`、`indices`、`values`、`mask` 等另外 5 项 准备或更新状态。 调用 `ttgl.BlockedLayout`、`ttgl.NVMMASharedLayout`、`ttgl.allocate_shared_memory`、`smem.store`、`ttgl.zeros`、`ttgl.arange` 等另外 14 项 执行测试逻辑。 相关主题：布局变换推理。

### Lines 265-268

```python


@gluon.jit
def tensor_memory_kernel(layout: ttgl.constexpr, tmem_layout: ttgl.constexpr):
```
- **EN:** Defines the helper function `tensor_memory_kernel`. Decorators: `gluon.jit`. Parameters: `layout`, `tmem_layout`. Key calls include `ttgl.full`, `ttgl.nvidia.blackwell.allocate_tensor_memory`, `mem.load`, `mem.store`, `mem.slice`, `buffers.index`. This scope touches layout transformation reasoning.
- **CN:** 定义辅助函数 `tensor_memory_kernel`。 装饰器：`gluon.jit`。 参数：`layout`、`tmem_layout`。 关键调用包括 `ttgl.full`、`ttgl.nvidia.blackwell.allocate_tensor_memory`、`mem.load`、`mem.store`、`mem.slice`、`buffers.index`。 该作用域涉及布局变换推理。

#### Lines 269-279

```python
    XBLOCK: ttgl.constexpr = tmem_layout.block[0]
    YBLOCK: ttgl.constexpr = tmem_layout.block[1]
    a = ttgl.full([XBLOCK, YBLOCK], 0, ttgl.int32, layout)
    _ = ttgl.nvidia.blackwell.allocate_tensor_memory(ttgl.int32, a.shape, tmem_layout)
    mem = ttgl.nvidia.blackwell.allocate_tensor_memory(ttgl.int32, a.shape, tmem_layout, a)
    b = mem.load(layout)  # noqa: F841
    mem.store(a)
    slice1 = mem.slice(0, YBLOCK // 2)  # noqa: F841
    slice2 = mem.slice(YBLOCK // 2, YBLOCK // 2)  # noqa: F841

    buffers = ttgl.nvidia.blackwell.allocate_tensor_memory(ttgl.float32, [2, XBLOCK, YBLOCK], tmem_layout)
```
- **EN:** Prepares or updates state through `XBLOCK`, `YBLOCK`, `a`, `_`, `mem`, `b`, `slice1`, `slice2`, and 1 more. Invokes `ttgl.full`, `ttgl.nvidia.blackwell.allocate_tensor_memory`, `mem.load`, `mem.store`, `mem.slice` to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `XBLOCK`、`YBLOCK`、`a`、`_`、`mem`、`b`、`slice1`、`slice2` 等另外 1 项 准备或更新状态。 调用 `ttgl.full`、`ttgl.nvidia.blackwell.allocate_tensor_memory`、`mem.load`、`mem.store`、`mem.slice` 执行测试逻辑。 相关主题：布局变换推理。

#### Lines 280-281

```python
    for ivar in range(2):
        buffers.index(ivar).load(layout)
```
- **EN:** Invokes `buffers.index` to execute the test logic. Iterates across cases or data tiles. Relevant themes: layout transformation reasoning.
- **CN:** 调用 `buffers.index` 执行测试逻辑。 通过循环覆盖多个用例或数据分块。 相关主题：布局变换推理。

### Lines 282-284

```python


def test_tensor_memory():
```
- **EN:** Defines the test function `test_tensor_memory`. Key calls include `ttgl.BlockedLayout`, `TensorMemoryLayout`, `run_parser`, `expecttest.assert_expected_inline`, `anonymize_ir`, `make_args`, and 1 more. This scope touches debugging and inspection paths, layout transformation reasoning.
- **CN:** 定义测试函数 `test_tensor_memory`。 关键调用包括 `ttgl.BlockedLayout`、`TensorMemoryLayout`、`run_parser`、`expecttest.assert_expected_inline`、`anonymize_ir`、`make_args` 等另外 1 项。 该作用域涉及调试与检查路径、布局变换推理。

#### Lines 285-322

```python
    layout = ttgl.BlockedLayout(size_per_thread=[1, 64], threads_per_warp=[32, 1], warps_per_cta=[4, 1], order=[0, 1])
    tmem_layout = TensorMemoryLayout(block=[128, 128], col_stride=1)
    mod = run_parser(
        tensor_memory_kernel,
        *make_args(layout, tmem_layout, num_warps=4),
        target=BLACKWELL_TARGET,
    )
    expecttest.assert_expected_inline(
        anonymize_ir(mod.str_nodebug()), """\
#blocked = #ttg.blocked<{sizePerThread = [1, 64], threadsPerWarp = [32, 1], warpsPerCTA = [4, 1], order = [0, 1]}>
#tmem = #ttng.tensor_memory_encoding<blockM = 128, blockN = 128, colStride = 1>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "...", "ttg.threads-per-warp" = 32 : i32} {
  tt.func public @tensor_memory_kernel() attributes {noinline = false} {
    %c0_i32 = arith.constant 0 : i32
    %cst = arith.constant dense<0> : tensor<128x128xi32, #blocked>
    %result = ttng.tmem_alloc : () -> !ttg.memdesc<128x128xi32, #tmem, #ttng.tensor_memory, mutable>
    %result_0 = ttng.tmem_alloc %cst : (tensor<128x128xi32, #blocked>) -> !ttg.memdesc<128x128xi32, #tmem, #ttng.tensor_memory, mutable>
    %result_1 = ttng.tmem_load %result_0 : !ttg.memdesc<128x128xi32, #tmem, #ttng.tensor_memory, mutable> -> tensor<128x128xi32, #blocked>
    %true = arith.constant true
    ttng.tmem_store %cst, %result_0, %true : tensor<128x128xi32, #blocked> -> !ttg.memdesc<128x128xi32, #tmem, #ttng.tensor_memory, mutable>
    %0 = ttng.tmem_subslice %result_0 {N = 0 : i32} : !ttg.memdesc<128x128xi32, #tmem, #ttng.tensor_memory, mutable> -> !ttg.memdesc<128x64xi32, #tmem, #ttng.tensor_memory, mutable, 128x128>
    %1 = ttng.tmem_subslice %result_0 {N = 64 : i32} : !ttg.memdesc<128x128xi32, #tmem, #ttng.tensor_memory, mutable> -> !ttg.memdesc<128x64xi32, #tmem, #ttng.tensor_memory, mutable, 128x128>
    %result_2 = ttng.tmem_alloc : () -> !ttg.memdesc<2x128x128xf32, #tmem, #ttng.tensor_memory, mutable>
    %c0_i32_3 = arith.constant 0 : i32
    %c2_i32 = arith.constant 2 : i32
    %c1_i32 = arith.constant 1 : i32
    %2 = arith.bitcast %c0_i32_3 : i32 to i32
    %3 = arith.bitcast %c2_i32 : i32 to i32
    %4 = arith.bitcast %c1_i32 : i32 to i32
    %5 = ub.poison : i32
    scf.for %arg0 = %2 to %3 step %4  : i32 {
      %6 = ttg.memdesc_index %result_2[%arg0] : !ttg.memdesc<2x128x128xf32, #tmem, #ttng.tensor_memory, mutable> -> !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
      %result_4 = ttng.tmem_load %6 : !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable> -> tensor<128x128xf32, #blocked>
    }
    tt.return
  }
}
""")
```
- **EN:** Prepares or updates state through `layout`, `tmem_layout`, `mod`. Invokes `ttgl.BlockedLayout`, `TensorMemoryLayout`, `run_parser`, `make_args`, `expecttest.assert_expected_inline`, `anonymize_ir`, and 1 more to execute the test logic. Relevant themes: debugging and inspection paths, layout transformation reasoning.
- **CN:** 通过 `layout`、`tmem_layout`、`mod` 准备或更新状态。 调用 `ttgl.BlockedLayout`、`TensorMemoryLayout`、`run_parser`、`make_args`、`expecttest.assert_expected_inline`、`anonymize_ir` 等另外 1 项 执行测试逻辑。 相关主题：调试与检查路径、布局变换推理。

### Lines 323-326

```python


@gluon.jit
def shared_memory_subview_kernel(XBLOCK: ttgl.constexpr, layout: ttgl.constexpr, smem_layout: ttgl.constexpr):
```
- **EN:** Defines the helper function `shared_memory_subview_kernel`. Decorators: `gluon.jit`. Parameters: `XBLOCK`, `layout`, `smem_layout`. Key calls include `ttgl.allocate_shared_memory`, `smem.slice`, `view.load`, `view.store`, `value.trans`. This scope touches layout transformation reasoning.
- **CN:** 定义辅助函数 `shared_memory_subview_kernel`。 装饰器：`gluon.jit`。 参数：`XBLOCK`、`layout`、`smem_layout`。 关键调用包括 `ttgl.allocate_shared_memory`、`smem.slice`、`view.load`、`view.store`、`value.trans`。 该作用域涉及布局变换推理。

#### Lines 327-332

```python
    XHALF: ttgl.constexpr = XBLOCK // 2
    smem = ttgl.allocate_shared_memory(ttgl.int32, [XBLOCK, XBLOCK], smem_layout)
    view = smem.slice(XHALF, XHALF, dim=1)
    value = view.load(layout)
    view = smem.slice(XHALF, XHALF, dim=0)
    view.store(value.trans())
```
- **EN:** Prepares or updates state through `XHALF`, `smem`, `view`, `value`. Invokes `ttgl.allocate_shared_memory`, `smem.slice`, `view.load`, `view.store`, `value.trans` to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `XHALF`、`smem`、`view`、`value` 准备或更新状态。 调用 `ttgl.allocate_shared_memory`、`smem.slice`、`view.load`、`view.store`、`value.trans` 执行测试逻辑。 相关主题：布局变换推理。

### Lines 333-336

```python


@pytest.mark.parametrize("target", ALL_TARGETS)
def test_shared_memory_subview(target):
```
- **EN:** Defines the test function `test_shared_memory_subview`. Decorators: `pytest.mark.parametrize('target', ALL_TARGETS)`. Parameters: `target`. Key calls include `pytest.mark.parametrize`, `ttgl.BlockedLayout`, `ttgl.SwizzledSharedLayout`, `run_parser`, `expecttest.assert_expected_inline`, `anonymize_ir`, and 2 more. This scope touches pytest parametrization, debugging and inspection paths, layout transformation reasoning.
- **CN:** 定义测试函数 `test_shared_memory_subview`。 装饰器：`pytest.mark.parametrize('target', ALL_TARGETS)`。 参数：`target`。 关键调用包括 `pytest.mark.parametrize`、`ttgl.BlockedLayout`、`ttgl.SwizzledSharedLayout`、`run_parser`、`expecttest.assert_expected_inline`、`anonymize_ir` 等另外 2 项。 该作用域涉及pytest 参数化、调试与检查路径、布局变换推理。

#### Lines 337-361

```python
    layout = ttgl.BlockedLayout(size_per_thread=[1, 1], threads_per_warp=[1, 32], warps_per_cta=[4, 1], order=[1, 0])
    smem_layout = ttgl.SwizzledSharedLayout(1, 1, 1, [1, 0])
    mod = run_parser(
        shared_memory_subview_kernel,
        *make_args(256, layout, smem_layout, num_warps=4),
        target=target,
    )
    expecttest.assert_expected_inline(
        anonymize_ir(mod.str_nodebug()), """\
#blocked = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [1, 32], warpsPerCTA = [4, 1], order = [1, 0]}>
#blocked1 = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [32, 1], warpsPerCTA = [1, 4], order = [0, 1]}>
#shared = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [1, 0]}>
#smem = #ttg.shared_memory
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "...", "ttg.threads-per-warp" = 32 : i32} {
  tt.func public @shared_memory_subview_kernel() attributes {noinline = false} {
    %0 = ttg.local_alloc : () -> !ttg.memdesc<256x256xi32, #shared, #smem, mutable>
    %1 = ttg.memdesc_subslice %0[0, 128] : !ttg.memdesc<256x256xi32, #shared, #smem, mutable> -> !ttg.memdesc<256x128xi32, #shared, #smem, mutable, 256x256>
    %2 = ttg.local_load %1 : !ttg.memdesc<256x128xi32, #shared, #smem, mutable, 256x256> -> tensor<256x128xi32, #blocked>
    %3 = ttg.memdesc_subslice %0[128, 0] : !ttg.memdesc<256x256xi32, #shared, #smem, mutable> -> !ttg.memdesc<128x256xi32, #shared, #smem, mutable, 256x256>
    %4 = tt.trans %2 {order = array<i32: 1, 0>} : tensor<256x128xi32, #blocked> -> tensor<128x256xi32, #blocked1>
    ttg.local_store %4, %3 : tensor<128x256xi32, #blocked1> -> !ttg.memdesc<128x256xi32, #shared, #smem, mutable, 256x256>
    tt.return
  }
}
""")
```
- **EN:** Prepares or updates state through `layout`, `smem_layout`, `mod`. Invokes `ttgl.BlockedLayout`, `ttgl.SwizzledSharedLayout`, `run_parser`, `make_args`, `expecttest.assert_expected_inline`, `anonymize_ir`, and 1 more to execute the test logic. Relevant themes: debugging and inspection paths, layout transformation reasoning.
- **CN:** 通过 `layout`、`smem_layout`、`mod` 准备或更新状态。 调用 `ttgl.BlockedLayout`、`ttgl.SwizzledSharedLayout`、`run_parser`、`make_args`、`expecttest.assert_expected_inline`、`anonymize_ir` 等另外 1 项 执行测试逻辑。 相关主题：调试与检查路径、布局变换推理。

### Lines 362-365

```python


@gluon.jit
def shared_memory_index_kernel(XBLOCK: ttgl.constexpr, layout: ttgl.constexpr, smem_layout: ttgl.constexpr):
```
- **EN:** Defines the helper function `shared_memory_index_kernel`. Decorators: `gluon.jit`. Parameters: `XBLOCK`, `layout`, `smem_layout`. Key calls include `ttgl.allocate_shared_memory`, `smem.index`. This scope touches layout transformation reasoning.
- **CN:** 定义辅助函数 `shared_memory_index_kernel`。 装饰器：`gluon.jit`。 参数：`XBLOCK`、`layout`、`smem_layout`。 关键调用包括 `ttgl.allocate_shared_memory`、`smem.index`。 该作用域涉及布局变换推理。

#### Lines 366-366

```python
    smem = ttgl.allocate_shared_memory(ttgl.int32, [4, XBLOCK], smem_layout)
```
- **EN:** Prepares or updates state through `smem`. Invokes `ttgl.allocate_shared_memory` to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `smem` 准备或更新状态。 调用 `ttgl.allocate_shared_memory` 执行测试逻辑。 相关主题：布局变换推理。

#### Lines 367-368

```python
    for ivar in range(4):
        smem.index(ivar).load(layout)
```
- **EN:** Invokes `smem.index` to execute the test logic. Iterates across cases or data tiles. Relevant themes: layout transformation reasoning.
- **CN:** 调用 `smem.index` 执行测试逻辑。 通过循环覆盖多个用例或数据分块。 相关主题：布局变换推理。

### Lines 369-372

```python


@pytest.mark.parametrize("target", ALL_TARGETS)
def test_shared_memory_index(target):
```
- **EN:** Defines the test function `test_shared_memory_index`. Decorators: `pytest.mark.parametrize('target', ALL_TARGETS)`. Parameters: `target`. Key calls include `pytest.mark.parametrize`, `ttgl.BlockedLayout`, `ttgl.SwizzledSharedLayout`, `run_parser`, `expecttest.assert_expected_inline`, `anonymize_ir`, and 2 more. This scope touches pytest parametrization, debugging and inspection paths, layout transformation reasoning.
- **CN:** 定义测试函数 `test_shared_memory_index`。 装饰器：`pytest.mark.parametrize('target', ALL_TARGETS)`。 参数：`target`。 关键调用包括 `pytest.mark.parametrize`、`ttgl.BlockedLayout`、`ttgl.SwizzledSharedLayout`、`run_parser`、`expecttest.assert_expected_inline`、`anonymize_ir` 等另外 2 项。 该作用域涉及pytest 参数化、调试与检查路径、布局变换推理。

#### Lines 373-402

```python
    layout = ttgl.BlockedLayout(size_per_thread=[1], threads_per_warp=[32], warps_per_cta=[4], order=[0])
    smem_layout = ttgl.SwizzledSharedLayout(vec=1, per_phase=1, max_phase=1, order=[0])
    mod = run_parser(
        shared_memory_index_kernel,
        *make_args(256, layout, smem_layout, num_warps=4),
        target=target,
    )
    expecttest.assert_expected_inline(
        anonymize_ir(mod.str_nodebug()), """\
#blocked = #ttg.blocked<{sizePerThread = [1], threadsPerWarp = [32], warpsPerCTA = [4], order = [0]}>
#shared = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0]}>
#smem = #ttg.shared_memory
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "...", "ttg.threads-per-warp" = 32 : i32} {
  tt.func public @shared_memory_index_kernel() attributes {noinline = false} {
    %0 = ttg.local_alloc : () -> !ttg.memdesc<4x256xi32, #shared, #smem, mutable>
    %c0_i32 = arith.constant 0 : i32
    %c4_i32 = arith.constant 4 : i32
    %c1_i32 = arith.constant 1 : i32
    %1 = arith.bitcast %c0_i32 : i32 to i32
    %2 = arith.bitcast %c4_i32 : i32 to i32
    %3 = arith.bitcast %c1_i32 : i32 to i32
    %4 = ub.poison : i32
    scf.for %arg0 = %1 to %2 step %3  : i32 {
      %5 = ttg.memdesc_index %0[%arg0] : !ttg.memdesc<4x256xi32, #shared, #smem, mutable> -> !ttg.memdesc<256xi32, #shared, #smem, mutable>
      %6 = ttg.local_load %5 : !ttg.memdesc<256xi32, #shared, #smem, mutable> -> tensor<256xi32, #blocked>
    }
    tt.return
  }
}
""")
```
- **EN:** Prepares or updates state through `layout`, `smem_layout`, `mod`. Invokes `ttgl.BlockedLayout`, `ttgl.SwizzledSharedLayout`, `run_parser`, `make_args`, `expecttest.assert_expected_inline`, `anonymize_ir`, and 1 more to execute the test logic. Relevant themes: debugging and inspection paths, layout transformation reasoning.
- **CN:** 通过 `layout`、`smem_layout`、`mod` 准备或更新状态。 调用 `ttgl.BlockedLayout`、`ttgl.SwizzledSharedLayout`、`run_parser`、`make_args`、`expecttest.assert_expected_inline`、`anonymize_ir` 等另外 1 项 执行测试逻辑。 相关主题：调试与检查路径、布局变换推理。

### Lines 403-406

```python


@gluon.jit
def shared_memory_permute_kernel():
```
- **EN:** Defines the helper function `shared_memory_permute_kernel`. Decorators: `gluon.jit`. Key calls include `ttgl.SwizzledSharedLayout`, `ttgl.allocate_shared_memory`, `smem.permute`, `ttgl.static_assert`. This scope touches layout transformation reasoning.
- **CN:** 定义辅助函数 `shared_memory_permute_kernel`。 装饰器：`gluon.jit`。 关键调用包括 `ttgl.SwizzledSharedLayout`、`ttgl.allocate_shared_memory`、`smem.permute`、`ttgl.static_assert`。 该作用域涉及布局变换推理。

#### Lines 407-410

```python
    layout: ttgl.constexpr = ttgl.SwizzledSharedLayout(1, 1, 1, [1, 0])
    smem = ttgl.allocate_shared_memory(ttgl.float16, [4, 128], layout)
    perm = smem.permute((1, 0))
    ttgl.static_assert(perm.layout == ttgl.SwizzledSharedLayout(1, 1, 1, [0, 1]))
```
- **EN:** Prepares or updates state through `layout`, `smem`, `perm`. Invokes `ttgl.SwizzledSharedLayout`, `ttgl.allocate_shared_memory`, `smem.permute`, `ttgl.static_assert` to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `layout`、`smem`、`perm` 准备或更新状态。 调用 `ttgl.SwizzledSharedLayout`、`ttgl.allocate_shared_memory`、`smem.permute`、`ttgl.static_assert` 执行测试逻辑。 相关主题：布局变换推理。

### Lines 411-414

```python


@pytest.mark.parametrize("target", ALL_TARGETS)
def test_shared_memory_permute(target):
```
- **EN:** Defines the test function `test_shared_memory_permute`. Decorators: `pytest.mark.parametrize('target', ALL_TARGETS)`. Parameters: `target`. Key calls include `pytest.mark.parametrize`, `run_parser`, `expecttest.assert_expected_inline`, `anonymize_ir`, `mod.str_nodebug`. This scope touches pytest parametrization, debugging and inspection paths.
- **CN:** 定义测试函数 `test_shared_memory_permute`。 装饰器：`pytest.mark.parametrize('target', ALL_TARGETS)`。 参数：`target`。 关键调用包括 `pytest.mark.parametrize`、`run_parser`、`expecttest.assert_expected_inline`、`anonymize_ir`、`mod.str_nodebug`。 该作用域涉及pytest 参数化、调试与检查路径。

#### Lines 415-428

```python
    mod = run_parser(shared_memory_permute_kernel, target=target)
    expecttest.assert_expected_inline(
        anonymize_ir(mod.str_nodebug()), """\
#shared = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [1, 0]}>
#shared1 = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0, 1]}>
#smem = #ttg.shared_memory
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "...", "ttg.threads-per-warp" = 32 : i32} {
  tt.func public @shared_memory_permute_kernel() attributes {noinline = false} {
    %0 = ttg.local_alloc : () -> !ttg.memdesc<4x128xf16, #shared, #smem, mutable>
    %1 = ttg.memdesc_trans %0 {order = array<i32: 1, 0>} : !ttg.memdesc<4x128xf16, #shared, #smem, mutable> -> !ttg.memdesc<128x4xf16, #shared1, #smem, mutable>
    tt.return
  }
}
""")
```
- **EN:** Prepares or updates state through `mod`. Invokes `run_parser`, `expecttest.assert_expected_inline`, `anonymize_ir`, `mod.str_nodebug` to execute the test logic. Relevant themes: debugging and inspection paths.
- **CN:** 通过 `mod` 准备或更新状态。 调用 `run_parser`、`expecttest.assert_expected_inline`、`anonymize_ir`、`mod.str_nodebug` 执行测试逻辑。 相关主题：调试与检查路径。

### Lines 429-432

```python


@gluon.jit
def shared_memory_cast_kernel():
```
- **EN:** Defines the helper function `shared_memory_cast_kernel`. Decorators: `gluon.jit`. Key calls include `ttgl.NVMMASharedLayout`, `ttgl.allocate_shared_memory`, `ttgl.static_assert`, `anchor_noinline`, `smem.reshape`, `smem._reinterpret`, and 2 more. This scope touches layout transformation reasoning.
- **CN:** 定义辅助函数 `shared_memory_cast_kernel`。 装饰器：`gluon.jit`。 关键调用包括 `ttgl.NVMMASharedLayout`、`ttgl.allocate_shared_memory`、`ttgl.static_assert`、`anchor_noinline`、`smem.reshape`、`smem._reinterpret` 等另外 2 项。 该作用域涉及布局变换推理。

#### Lines 433-448

```python
    layout_a: ttgl.constexpr = ttgl.NVMMASharedLayout(swizzle_byte_width=64, transposed=False, element_bitwidth=8,
                                                      rank=2)
    layout_T: ttgl.constexpr = ttgl.NVMMASharedLayout(swizzle_byte_width=64, transposed=True, element_bitwidth=8,
                                                      rank=2)
    smem = ttgl.allocate_shared_memory(ttgl.int8, [2, 256, 128], layout_a)
    perm = smem.index(0).permute((1, 0))
    ttgl.static_assert(perm.type.layout == layout_T)
    # Check that the MLIR type and Gluon types match by emitting a call.
    anchor_noinline(perm)

    layout_b: ttgl.constexpr = ttgl.NVMMASharedLayout(swizzle_byte_width=64, transposed=False, element_bitwidth=16,
                                                      rank=4)
    smem = ttgl.allocate_shared_memory(ttgl.float16, [32, 1, 4, 64], layout_b)
    smem.reshape((128, 64))

    smem._reinterpret(ttgl.int8, [16384], ttgl.SwizzledSharedLayout(1, 1, 1, [0]))
```
- **EN:** Prepares or updates state through `layout_a`, `layout_T`, `smem`, `perm`, `layout_b`. Invokes `ttgl.NVMMASharedLayout`, `ttgl.allocate_shared_memory`, `smem.index`, `ttgl.static_assert`, `anchor_noinline`, `smem.reshape`, and 2 more to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `layout_a`、`layout_T`、`smem`、`perm`、`layout_b` 准备或更新状态。 调用 `ttgl.NVMMASharedLayout`、`ttgl.allocate_shared_memory`、`smem.index`、`ttgl.static_assert`、`anchor_noinline`、`smem.reshape` 等另外 2 项 执行测试逻辑。 相关主题：布局变换推理。

### Lines 449-452

```python


@pytest.mark.parametrize("target", ALL_TARGETS)
def test_shared_memory_cast(target):
```
- **EN:** Defines the test function `test_shared_memory_cast`. Decorators: `pytest.mark.parametrize('target', ALL_TARGETS)`. Parameters: `target`. Key calls include `pytest.mark.parametrize`, `run_parser`, `expecttest.assert_expected_inline`, `anonymize_ir`, `mod.str_nodebug`. This scope touches pytest parametrization, debugging and inspection paths.
- **CN:** 定义测试函数 `test_shared_memory_cast`。 装饰器：`pytest.mark.parametrize('target', ALL_TARGETS)`。 参数：`target`。 关键调用包括 `pytest.mark.parametrize`、`run_parser`、`expecttest.assert_expected_inline`、`anonymize_ir`、`mod.str_nodebug`。 该作用域涉及pytest 参数化、调试与检查路径。

#### Lines 453-478

```python
    mod = run_parser(shared_memory_cast_kernel, target=target)
    expecttest.assert_expected_inline(
        anonymize_ir(mod.str_nodebug()), """\
#shared = #ttg.nvmma_shared<{swizzlingByteWidth = 64, transposed = false, elementBitWidth = 8}>
#shared1 = #ttg.nvmma_shared<{swizzlingByteWidth = 64, transposed = true, elementBitWidth = 8}>
#shared2 = #ttg.nvmma_shared<{swizzlingByteWidth = 64, transposed = false, elementBitWidth = 16, rank = 4}>
#shared3 = #ttg.nvmma_shared<{swizzlingByteWidth = 64, transposed = false, elementBitWidth = 16}>
#shared4 = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0]}>
#smem = #ttg.shared_memory
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "...", "ttg.threads-per-warp" = 32 : i32} {
  tt.func public @shared_memory_cast_kernel() attributes {noinline = false} {
    %0 = ttg.local_alloc : () -> !ttg.memdesc<2x256x128xi8, #shared, #smem, mutable>
    %c0_i32 = arith.constant 0 : i32
    %1 = ttg.memdesc_index %0[%c0_i32] : !ttg.memdesc<2x256x128xi8, #shared, #smem, mutable> -> !ttg.memdesc<256x128xi8, #shared, #smem, mutable>
    %2 = ttg.memdesc_trans %1 {order = array<i32: 1, 0>} : !ttg.memdesc<256x128xi8, #shared, #smem, mutable> -> !ttg.memdesc<128x256xi8, #shared1, #smem, mutable>
    tt.call @test_frontend.anchor_noinline__MDi8S128_256SLNVMMA_64_8_True_False__NVMMALAS128_256ASMD(%2) : (!ttg.memdesc<128x256xi8, #shared1, #smem, mutable>) -> ()
    %3 = ttg.local_alloc : () -> !ttg.memdesc<32x1x4x64xf16, #shared2, #smem, mutable>
    %4 = ttg.memdesc_reshape %3 : !ttg.memdesc<32x1x4x64xf16, #shared2, #smem, mutable> -> !ttg.memdesc<128x64xf16, #shared3, #smem, mutable>
    %5 = ttg.memdesc_reinterpret %3 : !ttg.memdesc<32x1x4x64xf16, #shared2, #smem, mutable> -> !ttg.memdesc<16384xi8, #shared4, #smem, mutable>
    tt.return
  }
  tt.func private @test_frontend.anchor_noinline__MDi8S128_256SLNVMMA_64_8_True_False__NVMMALAS128_256ASMD(%arg0: !ttg.memdesc<128x256xi8, #shared1, #smem, mutable>) attributes {noinline = true} {
    tt.return
  }
}
""")
```
- **EN:** Prepares or updates state through `mod`. Invokes `run_parser`, `expecttest.assert_expected_inline`, `anonymize_ir`, `mod.str_nodebug` to execute the test logic. Relevant themes: debugging and inspection paths.
- **CN:** 通过 `mod` 准备或更新状态。 调用 `run_parser`、`expecttest.assert_expected_inline`、`anonymize_ir`、`mod.str_nodebug` 执行测试逻辑。 相关主题：调试与检查路径。

### Lines 479-482

```python


@gluon.jit
def warp_specialize_default(a, b, e: ttgl.constexpr):
```
- **EN:** Defines the helper function `warp_specialize_default`. Decorators: `gluon.jit`. Parameters: `a`, `b`, `e`.
- **CN:** 定义辅助函数 `warp_specialize_default`。 装饰器：`gluon.jit`。 参数：`a`、`b`、`e`。

#### Lines 483-483

```python
    return b, a
```
- **EN:** Carries supporting logic for the surrounding test or helper scope.
- **CN:** 为周围的测试或辅助作用域提供支撑逻辑。

### Lines 484-487

```python


@gluon.jit
def warp_specialize_worker0(a, b, e: ttgl.constexpr):
```
- **EN:** Defines the helper function `warp_specialize_worker0`. Decorators: `gluon.jit`. Parameters: `a`, `b`, `e`.
- **CN:** 定义辅助函数 `warp_specialize_worker0`。 装饰器：`gluon.jit`。 参数：`a`、`b`、`e`。

#### Lines 488-488

```python
    pass
```
- **EN:** Carries supporting logic for the surrounding test or helper scope.
- **CN:** 为周围的测试或辅助作用域提供支撑逻辑。

### Lines 489-492

```python


@gluon.jit
def warp_specialize_worker1(a, b, e: ttgl.constexpr):
```
- **EN:** Defines the helper function `warp_specialize_worker1`. Decorators: `gluon.jit`. Parameters: `a`, `b`, `e`.
- **CN:** 定义辅助函数 `warp_specialize_worker1`。 装饰器：`gluon.jit`。 参数：`a`、`b`、`e`。

#### Lines 493-493

```python
    pass
```
- **EN:** Carries supporting logic for the surrounding test or helper scope.
- **CN:** 为周围的测试或辅助作用域提供支撑逻辑。

### Lines 494-497

```python


@gluon.aggregate
class Pair:
```
- **EN:** Defines class `Pair`. Decorators: `gluon.aggregate`.
- **CN:** 定义类 `Pair`。 装饰器：`gluon.aggregate`。

#### Lines 498-499

```python
    first: tl.tensor
    second: tl.tensor
```
- **EN:** Prepares or updates state through `first`, `second`. Relevant themes: Triton language operations.
- **CN:** 通过 `first`、`second` 准备或更新状态。 相关主题：Triton language 操作。

### Lines 500-503

```python


@gluon.jit
def anchor(x):
```
- **EN:** Defines the helper function `anchor`. Decorators: `gluon.jit`. Parameters: `x`.
- **CN:** 定义辅助函数 `anchor`。 装饰器：`gluon.jit`。 参数：`x`。

#### Lines 504-504

```python
    pass
```
- **EN:** Carries supporting logic for the surrounding test or helper scope.
- **CN:** 为周围的测试或辅助作用域提供支撑逻辑。

### Lines 505-508

```python


@gluon.jit(noinline=True)
def anchor_noinline(x):
```
- **EN:** Defines the helper function `anchor_noinline`. Decorators: `gluon.jit(noinline=True)`. Parameters: `x`. Key calls include `gluon.jit`.
- **CN:** 定义辅助函数 `anchor_noinline`。 装饰器：`gluon.jit(noinline=True)`。 参数：`x`。 关键调用包括 `gluon.jit`。

#### Lines 509-509

```python
    pass
```
- **EN:** Carries supporting logic for the surrounding test or helper scope.
- **CN:** 为周围的测试或辅助作用域提供支撑逻辑。

### Lines 510-534

```python


@filecheck_test
@gluon.jit
def test_warp_specialize():
    # CHECK:       [[BLOCKED:#.*]] = #ttg.blocked<{sizePerThread = [1], threadsPerWarp = [32], warpsPerCTA = [4], order = [0]}>
    # CHECK-LABEL: test_warp_specialize
    # CHECK-NEXT:    [[A:%.*]] = tt.make_range {end = 1 : i32, start = 0 : i32}
    # CHECK-NEXT:    [[B:%.*]] = tt.make_range {end = 2 : i32, start = 0 : i32}
    # CHECK-NEXT:    [[C:%.*]] = tt.make_range {end = 4 : i32, start = 0 : i32}
    # CHECK-NEXT:    [[OUTS:%.*]]:3 = ttg.warp_specialize([[A]], [[B]], [[C]], [[A]], [[B]], [[C]]) {{.*}}requestedRegisters = array<i32: 24, 48>
    # CHECK-NEXT:    default {
    # CHECK-NEXT:      [[RESULTS:%.*]]:3 = tt.call @{{.*}}warp_specialize_default{{.*}}c42{{.*}}([[A]], [[B]], [[C]])
    # CHECK-NEXT:      warp_yield [[RESULTS]]#0, [[RESULTS]]#1, [[RESULTS]]#2
    # CHECK-NEXT:    }
    # CHECK-NEXT:    partition0(%arg0: tensor<1xi32, [[BLOCKED]]>, %arg1: tensor<2xi32, [[BLOCKED]]>, %arg2: tensor<4xi32, [[BLOCKED]]>, %arg3: tensor<1xi32, [[BLOCKED]]>, %arg4: tensor<2xi32, [[BLOCKED]]>, %arg5: tensor<4xi32, [[BLOCKED]]>) num_warps(4) {
    # CHECK-NEXT:      call @{{.*}}warp_specialize_worker0{{.*}}c42{{.*}}(%arg0, %arg1, %arg2)
    # CHECK-NEXT:      warp_return
    # CHECK-NEXT:    }
    # CHECK-NEXT:    partition1(%arg0: tensor<1xi32, [[BLOCKED]]>, %arg1: tensor<2xi32, [[BLOCKED]]>, %arg2: tensor<4xi32, [[BLOCKED]]>, %arg3: tensor<1xi32, [[BLOCKED]]>, %arg4: tensor<2xi32, [[BLOCKED]]>, %arg5: tensor<4xi32, [[BLOCKED]]>) num_warps(4) {
    # CHECK-NEXT:      call @{{.*}}warp_specialize_worker1{{.*}}c42{{.*}}(%arg3, %arg4, %arg5)
    # CHECK-NEXT:      warp_return
    # CHECK-NEXT:    }
    # CHECK-NEXT:    call @{{.*}}anchor{{.*}}([[OUTS]]#0)
    # CHECK-NEXT:    call @{{.*}}anchor{{.*}}([[OUTS]]#1, [[OUTS]]#2)
```
- **EN:** Defines the test function `test_warp_specialize`. Decorators: `filecheck_test`, `gluon.jit`. Key calls include `ttgl.BlockedLayout`, `ttgl.arange`, `Pair`, `ttgl.warp_specialize`, `anchor`. This scope touches layout transformation reasoning.
- **CN:** 定义测试函数 `test_warp_specialize`。 装饰器：`filecheck_test`、`gluon.jit`。 关键调用包括 `ttgl.BlockedLayout`、`ttgl.arange`、`Pair`、`ttgl.warp_specialize`、`anchor`。 该作用域涉及布局变换推理。

#### Lines 535-554

```python
    layout: ttgl.constexpr = ttgl.BlockedLayout([1], [32], [4], [0])
    a = ttgl.arange(0, 1, layout=layout)
    b = ttgl.arange(0, 2, layout=layout)
    c = ttgl.arange(0, 4, layout=layout)
    pair = Pair(a, b)
    e: ttgl.constexpr = 42
    a, b = ttgl.warp_specialize([
        (warp_specialize_default, (pair, c, e)),
        (warp_specialize_worker0, (pair, c, e)),
        (warp_specialize_worker1, (pair, c, e)),
    ], [4, 4], [24, 48])
    anchor(a)
    anchor(b)

    # CHECK: ttg.warp_specialize([[A]], [[B]], [[C]])
    # CHECK: (tensor<1xi32, [[BLOCKED]]>, tensor<2xi32, [[BLOCKED]]>, tensor<4xi32, [[BLOCKED]]>) -> ()
    ttgl.warp_specialize([
        (warp_specialize_worker0, (pair, c, e)),
        (warp_specialize_worker1, (pair, c, e)),
    ], [4], [48])
```
- **EN:** Prepares or updates state through `layout`, `a`, `b`, `c`, `pair`, `e`. Invokes `ttgl.BlockedLayout`, `ttgl.arange`, `Pair`, `ttgl.warp_specialize`, `anchor` to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `layout`、`a`、`b`、`c`、`pair`、`e` 准备或更新状态。 调用 `ttgl.BlockedLayout`、`ttgl.arange`、`Pair`、`ttgl.warp_specialize`、`anchor` 执行测试逻辑。 相关主题：布局变换推理。

### Lines 555-558

```python


@gluon.jit
def ws_body(num_warps: ttgl.constexpr):
```
- **EN:** Defines the helper function `ws_body`. Decorators: `gluon.jit`. Parameters: `num_warps`. Key calls include `anchor`, `ttgl.arange`, `ttgl.BlockedLayout`. This scope touches layout transformation reasoning.
- **CN:** 定义辅助函数 `ws_body`。 装饰器：`gluon.jit`。 参数：`num_warps`。 关键调用包括 `anchor`、`ttgl.arange`、`ttgl.BlockedLayout`。 该作用域涉及布局变换推理。

#### Lines 559-559

```python
    anchor(ttgl.arange(0, 128, layout=ttgl.BlockedLayout([1], [32], [num_warps], [0])))
```
- **EN:** Invokes `anchor`, `ttgl.arange`, `ttgl.BlockedLayout` to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 调用 `anchor`、`ttgl.arange`、`ttgl.BlockedLayout` 执行测试逻辑。 相关主题：布局变换推理。

### Lines 560-563

```python


@gluon.jit
def ws_test_default():
```
- **EN:** Defines the helper function `ws_test_default`. Decorators: `gluon.jit`. Key calls include `ws_body`.
- **CN:** 定义辅助函数 `ws_test_default`。 装饰器：`gluon.jit`。 关键调用包括 `ws_body`。

#### Lines 564-564

```python
    ws_body(4)
```
- **EN:** Invokes `ws_body` to execute the test logic.
- **CN:** 调用 `ws_body` 执行测试逻辑。

### Lines 565-568

```python


@gluon.jit
def ws_test_worker0():
```
- **EN:** Defines the helper function `ws_test_worker0`. Decorators: `gluon.jit`. Key calls include `ws_body`.
- **CN:** 定义辅助函数 `ws_test_worker0`。 装饰器：`gluon.jit`。 关键调用包括 `ws_body`。

#### Lines 569-569

```python
    ws_body(2)
```
- **EN:** Invokes `ws_body` to execute the test logic.
- **CN:** 调用 `ws_body` 执行测试逻辑。

### Lines 570-573

```python


@gluon.jit
def ws_test_worker1():
```
- **EN:** Defines the helper function `ws_test_worker1`. Decorators: `gluon.jit`. Key calls include `ws_body`.
- **CN:** 定义辅助函数 `ws_test_worker1`。 装饰器：`gluon.jit`。 关键调用包括 `ws_body`。

#### Lines 574-574

```python
    ws_body(1)
```
- **EN:** Invokes `ws_body` to execute the test logic.
- **CN:** 调用 `ws_body` 执行测试逻辑。

### Lines 575-594

```python


@filecheck_test
@gluon.jit
def test_num_warps_caller_context():
    # CHECK-DAG: [[BLOCKED_NW4:#.*]] = #ttg.blocked<{sizePerThread = [1], threadsPerWarp = [32], warpsPerCTA = [4], order = [0]}>
    # CHECK-DAG: [[BLOCKED_NW2:#.*]] = #ttg.blocked<{sizePerThread = [1], threadsPerWarp = [32], warpsPerCTA = [2], order = [0]}>
    # CHECK-DAG: [[BLOCKED_NW1:#.*]] = #ttg.blocked<{sizePerThread = [1], threadsPerWarp = [32], warpsPerCTA = [1], order = [0]}>

    # CHECK: func private @{{.*}}ws_test_default{{.*}}() attributes {noinline = false}
    # CHECK: func private @{{.*}}ws_body{{.*}}() attributes {noinline = false}
    # CHECK: func private @{{.*}}anchor{{.*}}(%arg0: tensor<128xi32, [[BLOCKED_NW4]]>) attributes {noinline = false}

    # CHECK: func private @{{.*}}ws_test_worker0{{.*}}_NW2() attributes {noinline = false, "ttg.num-warps" = 2 : i32}
    # CHECK: func private @{{.*}}ws_body{{.*}}_NW2() attributes {noinline = false, "ttg.num-warps" = 2 : i32}
    # CHECK: func private @{{.*}}anchor{{.*}}_NW2(%arg0: tensor<128xi32, [[BLOCKED_NW2]]>) attributes {noinline = false, "ttg.num-warps" = 2 : i32}

    # CHECK: func private @{{.*}}ws_test_worker1{{.*}}_NW1() attributes {noinline = false, "ttg.num-warps" = 1 : i32}
    # CHECK: func private @{{.*}}ws_body{{.*}}_NW1() attributes {noinline = false, "ttg.num-warps" = 1 : i32}
    # CHECK: func private @{{.*}}anchor{{.*}}_NW1(%arg0: tensor<128xi32, [[BLOCKED_NW1]]>) attributes {noinline = false, "ttg.num-warps" = 1 : i32}
```
- **EN:** Defines the test function `test_num_warps_caller_context`. Decorators: `filecheck_test`, `gluon.jit`. Key calls include `ttgl.warp_specialize`.
- **CN:** 定义测试函数 `test_num_warps_caller_context`。 装饰器：`filecheck_test`、`gluon.jit`。 关键调用包括 `ttgl.warp_specialize`。

#### Lines 595-599

```python
    ttgl.warp_specialize([
        (ws_test_default, ()),
        (ws_test_worker0, ()),
        (ws_test_worker1, ()),
    ], [2, 1], [80, 80])
```
- **EN:** Invokes `ttgl.warp_specialize` to execute the test logic.
- **CN:** 调用 `ttgl.warp_specialize` 执行测试逻辑。

### Lines 600-603

```python


@gluon.jit
def mbarrier_kernel():
```
- **EN:** Defines the helper function `mbarrier_kernel`. Decorators: `gluon.jit`. Key calls include `ttgl.allocate_shared_memory`, `mbarrier.init`, `mbarrier.expect`, `mbarrier.arrive`, `mbarrier.wait`, `mbarrier.invalidate`, and 1 more. This scope touches layout transformation reasoning.
- **CN:** 定义辅助函数 `mbarrier_kernel`。 装饰器：`gluon.jit`。 关键调用包括 `ttgl.allocate_shared_memory`、`mbarrier.init`、`mbarrier.expect`、`mbarrier.arrive`、`mbarrier.wait`、`mbarrier.invalidate` 等另外 1 项。 该作用域涉及布局变换推理。

#### Lines 604-610

```python
    bar = ttgl.allocate_shared_memory(ttgl.int64, [1], mbarrier.MBarrierLayout())
    mbarrier.init(bar, count=1)
    mbarrier.expect(bar, 4)
    mbarrier.arrive(bar, count=1)
    phase = 0
    mbarrier.wait(bar, phase, deps=[bar])
    mbarrier.invalidate(bar)
```
- **EN:** Prepares or updates state through `bar`, `phase`. Invokes `ttgl.allocate_shared_memory`, `mbarrier.MBarrierLayout`, `mbarrier.init`, `mbarrier.expect`, `mbarrier.arrive`, `mbarrier.wait`, and 1 more to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `bar`、`phase` 准备或更新状态。 调用 `ttgl.allocate_shared_memory`、`mbarrier.MBarrierLayout`、`mbarrier.init`、`mbarrier.expect`、`mbarrier.arrive`、`mbarrier.wait` 等另外 1 项 执行测试逻辑。 相关主题：布局变换推理。

### Lines 611-614

```python


@pytest.mark.parametrize("target", [HOPPER_TARGET, BLACKWELL_TARGET])
def test_mbarrier(target):
```
- **EN:** Defines the test function `test_mbarrier`. Decorators: `pytest.mark.parametrize('target', [HOPPER_TARGET, BLACKWELL_TARGET])`. Parameters: `target`. Key calls include `pytest.mark.parametrize`, `run_parser`, `expecttest.assert_expected_inline`, `anonymize_ir`, `mod.str_nodebug`. This scope touches pytest parametrization, debugging and inspection paths.
- **CN:** 定义测试函数 `test_mbarrier`。 装饰器：`pytest.mark.parametrize('target', [HOPPER_TARGET, BLACKWELL_TARGET])`。 参数：`target`。 关键调用包括 `pytest.mark.parametrize`、`run_parser`、`expecttest.assert_expected_inline`、`anonymize_ir`、`mod.str_nodebug`。 该作用域涉及pytest 参数化、调试与检查路径。

#### Lines 615-635

```python
    mod = run_parser(mbarrier_kernel, target=target)
    expecttest.assert_expected_inline(
        anonymize_ir(mod.str_nodebug()), """\
#shared = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0]}>
#smem = #ttg.shared_memory
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "...", "ttg.threads-per-warp" = 32 : i32} {
  tt.func public @mbarrier_kernel() attributes {noinline = false} {
    %0 = ttg.local_alloc : () -> !ttg.memdesc<1xi64, #shared, #smem, mutable>
    ttng.init_barrier %0, 1 : !ttg.memdesc<1xi64, #shared, #smem, mutable>
    %true = arith.constant true
    ttng.barrier_expect %0, 4, %true : !ttg.memdesc<1xi64, #shared, #smem, mutable>
    %true_0 = arith.constant true
    ttng.arrive_barrier %0, 1, %true_0 : !ttg.memdesc<1xi64, #shared, #smem, mutable>
    %c0_i32 = arith.constant 0 : i32
    %true_1 = arith.constant true
    ttng.wait_barrier %0, %c0_i32, %true_1 deps %0 : !ttg.memdesc<1xi64, #shared, #smem, mutable>, !ttg.memdesc<1xi64, #shared, #smem, mutable>
    ttng.inval_barrier %0 : !ttg.memdesc<1xi64, #shared, #smem, mutable>
    tt.return
  }
}
""")
```
- **EN:** Prepares or updates state through `mod`. Invokes `run_parser`, `expecttest.assert_expected_inline`, `anonymize_ir`, `mod.str_nodebug` to execute the test logic. Relevant themes: debugging and inspection paths.
- **CN:** 通过 `mod` 准备或更新状态。 调用 `run_parser`、`expecttest.assert_expected_inline`、`anonymize_ir`、`mod.str_nodebug` 执行测试逻辑。 相关主题：调试与检查路径。

### Lines 636-639

```python


@gluon.jit
def tcgen05_mma_kernel(nvmma_layout: ttgl.constexpr, acc_layout: ttgl.constexpr):
```
- **EN:** Defines the helper function `tcgen05_mma_kernel`. Decorators: `gluon.jit`. Parameters: `nvmma_layout`, `acc_layout`. Key calls include `ttgl.allocate_shared_memory`, `blackwell.allocate_tensor_memory`, `blackwell.tcgen05_mma`. This scope touches layout transformation reasoning.
- **CN:** 定义辅助函数 `tcgen05_mma_kernel`。 装饰器：`gluon.jit`。 参数：`nvmma_layout`、`acc_layout`。 关键调用包括 `ttgl.allocate_shared_memory`、`blackwell.allocate_tensor_memory`、`blackwell.tcgen05_mma`。 该作用域涉及布局变换推理。

#### Lines 640-643

```python
    a = ttgl.allocate_shared_memory(ttgl.float16, [128, 128], nvmma_layout)
    b = ttgl.allocate_shared_memory(ttgl.float16, [128, 128], nvmma_layout)
    acc = blackwell.allocate_tensor_memory(ttgl.float16, [128, 128], acc_layout)
    blackwell.tcgen05_mma(a, b, acc)
```
- **EN:** Prepares or updates state through `a`, `b`, `acc`. Invokes `ttgl.allocate_shared_memory`, `blackwell.allocate_tensor_memory`, `blackwell.tcgen05_mma` to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `a`、`b`、`acc` 准备或更新状态。 调用 `ttgl.allocate_shared_memory`、`blackwell.allocate_tensor_memory`、`blackwell.tcgen05_mma` 执行测试逻辑。 相关主题：布局变换推理。

### Lines 644-646

```python


def test_tcgen05_mma():
```
- **EN:** Defines the test function `test_tcgen05_mma`. Key calls include `ttgl.NVMMASharedLayout`, `TensorMemoryLayout`, `run_parser`, `expecttest.assert_expected_inline`, `anonymize_ir`, `make_args`, and 1 more. This scope touches debugging and inspection paths, layout transformation reasoning.
- **CN:** 定义测试函数 `test_tcgen05_mma`。 关键调用包括 `ttgl.NVMMASharedLayout`、`TensorMemoryLayout`、`run_parser`、`expecttest.assert_expected_inline`、`anonymize_ir`、`make_args` 等另外 1 项。 该作用域涉及调试与检查路径、布局变换推理。

#### Lines 647-667

```python
    nvmma_layout = ttgl.NVMMASharedLayout(swizzle_byte_width=128, element_bitwidth=16, rank=2)
    acc_layout = TensorMemoryLayout([128, 128], col_stride=2)

    mod = run_parser(tcgen05_mma_kernel, *make_args(nvmma_layout, acc_layout), target=BLACKWELL_TARGET)
    expecttest.assert_expected_inline(
        anonymize_ir(mod.str_nodebug()), """\
#shared = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 16}>
#smem = #ttg.shared_memory
#tmem = #ttng.tensor_memory_encoding<blockM = 128, blockN = 128, colStride = 2>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "...", "ttg.threads-per-warp" = 32 : i32} {
  tt.func public @tcgen05_mma_kernel() attributes {noinline = false} {
    %0 = ttg.local_alloc : () -> !ttg.memdesc<128x128xf16, #shared, #smem, mutable>
    %1 = ttg.local_alloc : () -> !ttg.memdesc<128x128xf16, #shared, #smem, mutable>
    %result = ttng.tmem_alloc : () -> !ttg.memdesc<128x128xf16, #tmem, #ttng.tensor_memory, mutable>
    %true = arith.constant true
    %true_0 = arith.constant true
    %2 = ttng.tc_gen5_mma %0, %1, %result[], %true, %true_0 : !ttg.memdesc<128x128xf16, #shared, #smem, mutable>, !ttg.memdesc<128x128xf16, #shared, #smem, mutable>, !ttg.memdesc<128x128xf16, #tmem, #ttng.tensor_memory, mutable>
    tt.return
  }
}
""")
```
- **EN:** Prepares or updates state through `nvmma_layout`, `acc_layout`, `mod`. Invokes `ttgl.NVMMASharedLayout`, `TensorMemoryLayout`, `run_parser`, `make_args`, `expecttest.assert_expected_inline`, `anonymize_ir`, and 1 more to execute the test logic. Relevant themes: debugging and inspection paths, layout transformation reasoning.
- **CN:** 通过 `nvmma_layout`、`acc_layout`、`mod` 准备或更新状态。 调用 `ttgl.NVMMASharedLayout`、`TensorMemoryLayout`、`run_parser`、`make_args`、`expecttest.assert_expected_inline`、`anonymize_ir` 等另外 1 项 执行测试逻辑。 相关主题：调试与检查路径、布局变换推理。

### Lines 668-671

```python


@gluon.jit
def tcgen05_mma_scaled_kernel(nvmma_layout: ttgl.constexpr, acc_layout: ttgl.constexpr, scale_layout: ttgl.constexpr):
```
- **EN:** Defines the helper function `tcgen05_mma_scaled_kernel`. Decorators: `gluon.jit`. Parameters: `nvmma_layout`, `acc_layout`, `scale_layout`. Key calls include `ttgl.allocate_shared_memory`, `blackwell.allocate_tensor_memory`, `blackwell.tcgen05_mma_scaled`. This scope touches layout transformation reasoning.
- **CN:** 定义辅助函数 `tcgen05_mma_scaled_kernel`。 装饰器：`gluon.jit`。 参数：`nvmma_layout`、`acc_layout`、`scale_layout`。 关键调用包括 `ttgl.allocate_shared_memory`、`blackwell.allocate_tensor_memory`、`blackwell.tcgen05_mma_scaled`。 该作用域涉及布局变换推理。

#### Lines 672-677

```python
    a = ttgl.allocate_shared_memory(ttgl.float8e5, [128, 128], nvmma_layout)
    b = ttgl.allocate_shared_memory(ttgl.float8e5, [128, 128], nvmma_layout)
    scale_a = blackwell.allocate_tensor_memory(ttgl.int8, [128, 32], scale_layout)
    scale_b = blackwell.allocate_tensor_memory(ttgl.int8, [128, 32], scale_layout)
    acc = blackwell.allocate_tensor_memory(ttgl.float16, [128, 128], acc_layout)
    blackwell.tcgen05_mma_scaled(a, b, acc, scale_a, scale_b, "e5m2", "e5m2")
```
- **EN:** Prepares or updates state through `a`, `b`, `scale_a`, `scale_b`, `acc`. Invokes `ttgl.allocate_shared_memory`, `blackwell.allocate_tensor_memory`, `blackwell.tcgen05_mma_scaled` to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `a`、`b`、`scale_a`、`scale_b`、`acc` 准备或更新状态。 调用 `ttgl.allocate_shared_memory`、`blackwell.allocate_tensor_memory`、`blackwell.tcgen05_mma_scaled` 执行测试逻辑。 相关主题：布局变换推理。

### Lines 678-680

```python


def test_tcgen05_mma_scaled():
```
- **EN:** Defines the test function `test_tcgen05_mma_scaled`. Key calls include `ttgl.NVMMASharedLayout`, `TensorMemoryScalesLayout`, `TensorMemoryLayout`, `run_parser`, `expecttest.assert_expected_inline`, `anonymize_ir`, and 2 more. This scope touches debugging and inspection paths, layout transformation reasoning.
- **CN:** 定义测试函数 `test_tcgen05_mma_scaled`。 关键调用包括 `ttgl.NVMMASharedLayout`、`TensorMemoryScalesLayout`、`TensorMemoryLayout`、`run_parser`、`expecttest.assert_expected_inline`、`anonymize_ir` 等另外 2 项。 该作用域涉及调试与检查路径、布局变换推理。

#### Lines 681-706

```python
    nvmma_layout = ttgl.NVMMASharedLayout(swizzle_byte_width=128, element_bitwidth=16, rank=2)
    scale_layout = TensorMemoryScalesLayout()
    acc_layout = TensorMemoryLayout([128, 128], col_stride=2)

    mod = run_parser(tcgen05_mma_scaled_kernel, *make_args(nvmma_layout, acc_layout, scale_layout),
                     target=BLACKWELL_TARGET)
    expecttest.assert_expected_inline(
        anonymize_ir(mod.str_nodebug()), """\
#shared = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 16}>
#smem = #ttg.shared_memory
#tmem = #ttng.tensor_memory_encoding<blockM = 128, blockN = 128, colStride = 2>
#tmem_scales = #ttng.tensor_memory_scales_encoding<>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "...", "ttg.threads-per-warp" = 32 : i32} {
  tt.func public @tcgen05_mma_scaled_kernel() attributes {noinline = false} {
    %0 = ttg.local_alloc : () -> !ttg.memdesc<128x128xf8E5M2, #shared, #smem, mutable>
    %1 = ttg.local_alloc : () -> !ttg.memdesc<128x128xf8E5M2, #shared, #smem, mutable>
    %result = ttng.tmem_alloc : () -> !ttg.memdesc<128x32xi8, #tmem_scales, #ttng.tensor_memory, mutable>
    %result_0 = ttng.tmem_alloc : () -> !ttg.memdesc<128x32xi8, #tmem_scales, #ttng.tensor_memory, mutable>
    %result_1 = ttng.tmem_alloc : () -> !ttg.memdesc<128x128xf16, #tmem, #ttng.tensor_memory, mutable>
    %true = arith.constant true
    %true_2 = arith.constant true
    %2 = ttng.tc_gen5_mma_scaled %0, %1, %result_1[], %result, %result_0, %true, %true_2 lhs = e5m2 rhs = e5m2 : !ttg.memdesc<128x128xf8E5M2, #shared, #smem, mutable>, !ttg.memdesc<128x128xf8E5M2, #shared, #smem, mutable>, !ttg.memdesc<128x128xf16, #tmem, #ttng.tensor_memory, mutable>, !ttg.memdesc<128x32xi8, #tmem_scales, #ttng.tensor_memory, mutable>, !ttg.memdesc<128x32xi8, #tmem_scales, #ttng.tensor_memory, mutable>
    tt.return
  }
}
""")
```
- **EN:** Prepares or updates state through `nvmma_layout`, `scale_layout`, `acc_layout`, `mod`. Invokes `ttgl.NVMMASharedLayout`, `TensorMemoryScalesLayout`, `TensorMemoryLayout`, `run_parser`, `make_args`, `expecttest.assert_expected_inline`, and 2 more to execute the test logic. Relevant themes: debugging and inspection paths, layout transformation reasoning.
- **CN:** 通过 `nvmma_layout`、`scale_layout`、`acc_layout`、`mod` 准备或更新状态。 调用 `ttgl.NVMMASharedLayout`、`TensorMemoryScalesLayout`、`TensorMemoryLayout`、`run_parser`、`make_args`、`expecttest.assert_expected_inline` 等另外 2 项 执行测试逻辑。 相关主题：调试与检查路径、布局变换推理。

### Lines 707-710

```python


@gluon.jit
def tcgen05_mma_mbar_kernel(nvmma_layout: ttgl.constexpr, acc_layout: ttgl.constexpr):
```
- **EN:** Defines the helper function `tcgen05_mma_mbar_kernel`. Decorators: `gluon.jit`. Parameters: `nvmma_layout`, `acc_layout`. Key calls include `ttgl.allocate_shared_memory`, `blackwell.allocate_tensor_memory`, `blackwell.tcgen05_mma`, `mbarrier.MBarrierLayout`. This scope touches layout transformation reasoning.
- **CN:** 定义辅助函数 `tcgen05_mma_mbar_kernel`。 装饰器：`gluon.jit`。 参数：`nvmma_layout`、`acc_layout`。 关键调用包括 `ttgl.allocate_shared_memory`、`blackwell.allocate_tensor_memory`、`blackwell.tcgen05_mma`、`mbarrier.MBarrierLayout`。 该作用域涉及布局变换推理。

#### Lines 711-715

```python
    a = ttgl.allocate_shared_memory(ttgl.float16, [128, 128], nvmma_layout)
    b = ttgl.allocate_shared_memory(ttgl.float16, [128, 128], nvmma_layout)
    bar = ttgl.allocate_shared_memory(ttgl.int64, [1], mbarrier.MBarrierLayout())
    acc = blackwell.allocate_tensor_memory(ttgl.float16, [128, 128], acc_layout)
    blackwell.tcgen05_mma(a, b, acc, mbarriers=[bar])
```
- **EN:** Prepares or updates state through `a`, `b`, `bar`, `acc`. Invokes `ttgl.allocate_shared_memory`, `mbarrier.MBarrierLayout`, `blackwell.allocate_tensor_memory`, `blackwell.tcgen05_mma` to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `a`、`b`、`bar`、`acc` 准备或更新状态。 调用 `ttgl.allocate_shared_memory`、`mbarrier.MBarrierLayout`、`blackwell.allocate_tensor_memory`、`blackwell.tcgen05_mma` 执行测试逻辑。 相关主题：布局变换推理。

### Lines 716-718

```python


def test_tcgen05_mma_mbar():
```
- **EN:** Defines the test function `test_tcgen05_mma_mbar`. Key calls include `ttgl.NVMMASharedLayout`, `TensorMemoryLayout`, `run_parser`, `expecttest.assert_expected_inline`, `anonymize_ir`, `make_args`, and 1 more. This scope touches debugging and inspection paths, layout transformation reasoning.
- **CN:** 定义测试函数 `test_tcgen05_mma_mbar`。 关键调用包括 `ttgl.NVMMASharedLayout`、`TensorMemoryLayout`、`run_parser`、`expecttest.assert_expected_inline`、`anonymize_ir`、`make_args` 等另外 1 项。 该作用域涉及调试与检查路径、布局变换推理。

#### Lines 719-742

```python
    nvmma_layout = ttgl.NVMMASharedLayout(swizzle_byte_width=128, element_bitwidth=16, rank=2)
    acc_layout = TensorMemoryLayout([128, 128], col_stride=2)

    mod = run_parser(tcgen05_mma_mbar_kernel, *make_args(nvmma_layout, acc_layout), target=BLACKWELL_TARGET)
    expecttest.assert_expected_inline(
        anonymize_ir(mod.str_nodebug()), """\
#shared = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 16}>
#shared1 = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0]}>
#smem = #ttg.shared_memory
#tmem = #ttng.tensor_memory_encoding<blockM = 128, blockN = 128, colStride = 2>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "...", "ttg.threads-per-warp" = 32 : i32} {
  tt.func public @tcgen05_mma_mbar_kernel() attributes {noinline = false} {
    %0 = ttg.local_alloc : () -> !ttg.memdesc<128x128xf16, #shared, #smem, mutable>
    %1 = ttg.local_alloc : () -> !ttg.memdesc<128x128xf16, #shared, #smem, mutable>
    %2 = ttg.local_alloc : () -> !ttg.memdesc<1xi64, #shared1, #smem, mutable>
    %result = ttng.tmem_alloc : () -> !ttg.memdesc<128x128xf16, #tmem, #ttng.tensor_memory, mutable>
    %true = arith.constant true
    %true_0 = arith.constant true
    %true_1 = arith.constant true
    %3 = ttng.tc_gen5_mma %0, %1, %result[], %true, %true_0, %2[%true_1] {is_async} : !ttg.memdesc<128x128xf16, #shared, #smem, mutable>, !ttg.memdesc<128x128xf16, #shared, #smem, mutable>, !ttg.memdesc<128x128xf16, #tmem, #ttng.tensor_memory, mutable>, !ttg.memdesc<1xi64, #shared1, #smem, mutable>
    tt.return
  }
}
""")
```
- **EN:** Prepares or updates state through `nvmma_layout`, `acc_layout`, `mod`. Invokes `ttgl.NVMMASharedLayout`, `TensorMemoryLayout`, `run_parser`, `make_args`, `expecttest.assert_expected_inline`, `anonymize_ir`, and 1 more to execute the test logic. Relevant themes: debugging and inspection paths, layout transformation reasoning.
- **CN:** 通过 `nvmma_layout`、`acc_layout`、`mod` 准备或更新状态。 调用 `ttgl.NVMMASharedLayout`、`TensorMemoryLayout`、`run_parser`、`make_args`、`expecttest.assert_expected_inline`、`anonymize_ir` 等另外 1 项 执行测试逻辑。 相关主题：调试与检查路径、布局变换推理。

### Lines 743-748

```python


@filecheck_test
@gluon.jit
def test_tcgen05_commit():
    # CHECK-LABEL: test_tcgen05_commit
```
- **EN:** Defines the test function `test_tcgen05_commit`. Decorators: `filecheck_test`, `gluon.jit`. Key calls include `ttgl.allocate_shared_memory`, `blackwell.tcgen05_commit`, `mbarrier.MBarrierLayout`. This scope touches layout transformation reasoning.
- **CN:** 定义测试函数 `test_tcgen05_commit`。 装饰器：`filecheck_test`、`gluon.jit`。 关键调用包括 `ttgl.allocate_shared_memory`、`blackwell.tcgen05_commit`、`mbarrier.MBarrierLayout`。 该作用域涉及布局变换推理。

#### Lines 749-752

```python
    barrier = ttgl.allocate_shared_memory(ttgl.int64, [1], mbarrier.MBarrierLayout())
    # CHECK: [[BARRIER:%.*]] = ttg.local_alloc
    # CHECK: ttng.tc_gen5_commit [[BARRIER]]
    blackwell.tcgen05_commit(barrier)
```
- **EN:** Prepares or updates state through `barrier`. Invokes `ttgl.allocate_shared_memory`, `mbarrier.MBarrierLayout`, `blackwell.tcgen05_commit` to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `barrier` 准备或更新状态。 调用 `ttgl.allocate_shared_memory`、`mbarrier.MBarrierLayout`、`blackwell.tcgen05_commit` 执行测试逻辑。 相关主题：布局变换推理。

### Lines 753-756

```python


@gluon.jit
def tcgen05_commit_multicast_two_ctas_kernel():
```
- **EN:** Defines the helper function `tcgen05_commit_multicast_two_ctas_kernel`. Decorators: `gluon.jit`. Key calls include `mbarrier.allocate_mbarrier`, `mbarrier.expect`.
- **CN:** 定义辅助函数 `tcgen05_commit_multicast_two_ctas_kernel`。 装饰器：`gluon.jit`。 关键调用包括 `mbarrier.allocate_mbarrier`、`mbarrier.expect`。

#### Lines 757-758

```python
    barrier = mbarrier.allocate_mbarrier(two_ctas=True)
    mbarrier.expect(barrier, 4)
```
- **EN:** Prepares or updates state through `barrier`. Invokes `mbarrier.allocate_mbarrier`, `mbarrier.expect` to execute the test logic.
- **CN:** 通过 `barrier` 准备或更新状态。 调用 `mbarrier.allocate_mbarrier`、`mbarrier.expect` 执行测试逻辑。

### Lines 759-761

```python


def test_tcgen05_commit_multicast_two_ctas():
```
- **EN:** Defines the test function `test_tcgen05_commit_multicast_two_ctas`. Key calls include `run_parser`, `expecttest.assert_expected_inline`, `anonymize_ir`, `make_args`, `mod.str_nodebug`. This scope touches debugging and inspection paths, layout transformation reasoning.
- **CN:** 定义测试函数 `test_tcgen05_commit_multicast_two_ctas`。 关键调用包括 `run_parser`、`expecttest.assert_expected_inline`、`anonymize_ir`、`make_args`、`mod.str_nodebug`。 该作用域涉及调试与检查路径、布局变换推理。

#### Lines 762-782

```python
    mod = run_parser(tcgen05_commit_multicast_two_ctas_kernel, *make_args(num_ctas=2), target=BLACKWELL_TARGET)
    expecttest.assert_expected_inline(
        anonymize_ir(mod.str_nodebug()), """\
#shared = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0], CGALayout = [[0]]}>
#smem = #ttg.shared_memory
module attributes {"ttg.num-ctas" = 2 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "...", "ttg.threads-per-warp" = 32 : i32} {
  tt.func public @tcgen05_commit_multicast_two_ctas_kernel() attributes {noinline = false} {
    %0 = tt.call @triton.experimental.gluon.language.nvidia.ampere.mbarrier.allocate_mbarrier__cNone_cTrue() : () -> !ttg.memdesc<1xi64, #shared, #smem, mutable>
    %true = arith.constant true
    ttng.barrier_expect %0, 4, %true : !ttg.memdesc<1xi64, #shared, #smem, mutable>
    tt.return
  }
  tt.func private @triton.experimental.gluon.language.nvidia.ampere.mbarrier.allocate_mbarrier__cNone_cTrue() -> !ttg.memdesc<1xi64, #shared, #smem, mutable> attributes {noinline = false} {
    %0 = ttg.local_alloc : () -> !ttg.memdesc<1xi64, #shared, #smem, mutable>
    tt.return %0 : !ttg.memdesc<1xi64, #shared, #smem, mutable>
  ^bb1:  // no predecessors
    %1 = ub.poison : !ttg.memdesc<1xi64, #shared, #smem, mutable>
    tt.return %1 : !ttg.memdesc<1xi64, #shared, #smem, mutable>
  }
}
""")
```
- **EN:** Prepares or updates state through `mod`. Invokes `run_parser`, `make_args`, `expecttest.assert_expected_inline`, `anonymize_ir`, `mod.str_nodebug` to execute the test logic. Relevant themes: debugging and inspection paths, layout transformation reasoning.
- **CN:** 通过 `mod` 准备或更新状态。 调用 `run_parser`、`make_args`、`expecttest.assert_expected_inline`、`anonymize_ir`、`mod.str_nodebug` 执行测试逻辑。 相关主题：调试与检查路径、布局变换推理。

### Lines 783-786

```python


@gluon.jit
def warpgroup_mma_kernel(nvmma_layout: ttgl.constexpr, acc_layout: ttgl.constexpr):
```
- **EN:** Defines the helper function `warpgroup_mma_kernel`. Decorators: `gluon.jit`. Parameters: `nvmma_layout`, `acc_layout`. Key calls include `ttgl.allocate_shared_memory`, `ttgl.full`, `hopper.warpgroup_mma`, `ttgl.static_assert`, `isinstance`. This scope touches layout transformation reasoning.
- **CN:** 定义辅助函数 `warpgroup_mma_kernel`。 装饰器：`gluon.jit`。 参数：`nvmma_layout`、`acc_layout`。 关键调用包括 `ttgl.allocate_shared_memory`、`ttgl.full`、`hopper.warpgroup_mma`、`ttgl.static_assert`、`isinstance`。 该作用域涉及布局变换推理。

#### Lines 787-794

```python
    a = ttgl.allocate_shared_memory(ttgl.float16, [128, 128], nvmma_layout)
    b = ttgl.allocate_shared_memory(ttgl.float16, [128, 128], nvmma_layout)
    acc = ttgl.full([128, 128], 0, dtype=ttgl.float16, layout=acc_layout)
    acc = hopper.warpgroup_mma(a, b, acc)
    ttgl.static_assert(isinstance(acc, ttgl.tensor))

    acc = hopper.warpgroup_mma(a, b, acc, is_async=True)
    ttgl.static_assert(isinstance(acc, hopper.warpgroup_mma_accumulator))
```
- **EN:** Prepares or updates state through `a`, `b`, `acc`. Invokes `ttgl.allocate_shared_memory`, `ttgl.full`, `hopper.warpgroup_mma`, `ttgl.static_assert`, `isinstance` to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `a`、`b`、`acc` 准备或更新状态。 调用 `ttgl.allocate_shared_memory`、`ttgl.full`、`hopper.warpgroup_mma`、`ttgl.static_assert`、`isinstance` 执行测试逻辑。 相关主题：布局变换推理。

### Lines 795-797

```python


def test_warpgroup_mma():
```
- **EN:** Defines the test function `test_warpgroup_mma`. Key calls include `ttgl.NVMMASharedLayout`, `ttgl.NVMMADistributedLayout`, `run_parser`, `expecttest.assert_expected_inline`, `anonymize_ir`, `make_args`, and 1 more. This scope touches debugging and inspection paths, layout transformation reasoning.
- **CN:** 定义测试函数 `test_warpgroup_mma`。 关键调用包括 `ttgl.NVMMASharedLayout`、`ttgl.NVMMADistributedLayout`、`run_parser`、`expecttest.assert_expected_inline`、`anonymize_ir`、`make_args` 等另外 1 项。 该作用域涉及调试与检查路径、布局变换推理。

#### Lines 798-823

```python
    nvmma_layout = ttgl.NVMMASharedLayout(swizzle_byte_width=128, element_bitwidth=16, rank=2)
    mma_layout = ttgl.NVMMADistributedLayout(version=[3, 0], warps_per_cta=[4, 1], instr_shape=[16, 32, 16])
    mod = run_parser(
        warpgroup_mma_kernel,
        *make_args(nvmma_layout, mma_layout),
        target=HOPPER_TARGET,
    )
    expecttest.assert_expected_inline(
        anonymize_ir(mod.str_nodebug()), """\
#mma = #ttg.nvidia_mma<{versionMajor = 3, versionMinor = 0, warpsPerCTA = [4, 1], instrShape = [16, 32, 16]}>
#shared = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 16}>
#smem = #ttg.shared_memory
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "...", "ttg.threads-per-warp" = 32 : i32} {
  tt.func public @warpgroup_mma_kernel() attributes {noinline = false} {
    %0 = ttg.local_alloc : () -> !ttg.memdesc<128x128xf16, #shared, #smem, mutable>
    %1 = ttg.local_alloc : () -> !ttg.memdesc<128x128xf16, #shared, #smem, mutable>
    %cst = arith.constant 0.000000e+00 : f16
    %cst_0 = arith.constant dense<0.000000e+00> : tensor<128x128xf16, #mma>
    %true = arith.constant true
    %2 = ttng.warp_group_dot %0, %1, %cst_0, %true {inputPrecision = 0 : i32} : !ttg.memdesc<128x128xf16, #shared, #smem, mutable> * !ttg.memdesc<128x128xf16, #shared, #smem, mutable> -> tensor<128x128xf16, #mma>
    %true_1 = arith.constant true
    %3 = ttng.warp_group_dot %0, %1, %2, %true_1 {inputPrecision = 0 : i32, isAsync = true} : !ttg.memdesc<128x128xf16, #shared, #smem, mutable> * !ttg.memdesc<128x128xf16, #shared, #smem, mutable> -> tensor<128x128xf16, #mma>
    tt.return
  }
}
""")
```
- **EN:** Prepares or updates state through `nvmma_layout`, `mma_layout`, `mod`. Invokes `ttgl.NVMMASharedLayout`, `ttgl.NVMMADistributedLayout`, `run_parser`, `make_args`, `expecttest.assert_expected_inline`, `anonymize_ir`, and 1 more to execute the test logic. Relevant themes: debugging and inspection paths, layout transformation reasoning.
- **CN:** 通过 `nvmma_layout`、`mma_layout`、`mod` 准备或更新状态。 调用 `ttgl.NVMMASharedLayout`、`ttgl.NVMMADistributedLayout`、`run_parser`、`make_args`、`expecttest.assert_expected_inline`、`anonymize_ir` 等另外 1 项 执行测试逻辑。 相关主题：调试与检查路径、布局变换推理。

### Lines 824-827

```python


@gluon.jit
def warpgroup_mma_wait_kernel():
```
- **EN:** Defines the helper function `warpgroup_mma_wait_kernel`. Decorators: `gluon.jit`. Key calls include `ttgl.NVMMADistributedLayout`, `hopper.warpgroup_mma_init`, `hopper.warpgroup_mma_wait`, `ttgl.full`. This scope touches layout transformation reasoning.
- **CN:** 定义辅助函数 `warpgroup_mma_wait_kernel`。 装饰器：`gluon.jit`。 关键调用包括 `ttgl.NVMMADistributedLayout`、`hopper.warpgroup_mma_init`、`hopper.warpgroup_mma_wait`、`ttgl.full`。 该作用域涉及布局变换推理。

#### Lines 828-831

```python
    layout: ttgl.constexpr = ttgl.NVMMADistributedLayout(version=[3, 0], warps_per_cta=[4, 1], instr_shape=[16, 32, 16])
    acc = hopper.warpgroup_mma_init(ttgl.full([128, 128], 0, dtype=ttgl.float16, layout=layout))
    acc = hopper.warpgroup_mma_wait(num_outstanding=1, deps=[acc])
    _ = acc + acc
```
- **EN:** Prepares or updates state through `layout`, `acc`, `_`. Invokes `ttgl.NVMMADistributedLayout`, `hopper.warpgroup_mma_init`, `ttgl.full`, `hopper.warpgroup_mma_wait` to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `layout`、`acc`、`_` 准备或更新状态。 调用 `ttgl.NVMMADistributedLayout`、`hopper.warpgroup_mma_init`、`ttgl.full`、`hopper.warpgroup_mma_wait` 执行测试逻辑。 相关主题：布局变换推理。

### Lines 832-834

```python


def test_warpgroup_mma_wait():
```
- **EN:** Defines the test function `test_warpgroup_mma_wait`. Key calls include `run_parser`, `expecttest.assert_expected_inline`, `anonymize_ir`, `mod.str_nodebug`. This scope touches debugging and inspection paths.
- **CN:** 定义测试函数 `test_warpgroup_mma_wait`。 关键调用包括 `run_parser`、`expecttest.assert_expected_inline`、`anonymize_ir`、`mod.str_nodebug`。 该作用域涉及调试与检查路径。

#### Lines 835-848

```python
    mod = run_parser(warpgroup_mma_wait_kernel, target=HOPPER_TARGET)
    expecttest.assert_expected_inline(
        anonymize_ir(mod.str_nodebug()), """\
#mma = #ttg.nvidia_mma<{versionMajor = 3, versionMinor = 0, warpsPerCTA = [4, 1], instrShape = [16, 32, 16]}>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "...", "ttg.threads-per-warp" = 32 : i32} {
  tt.func public @warpgroup_mma_wait_kernel() attributes {noinline = false} {
    %cst = arith.constant 0.000000e+00 : f16
    %cst_0 = arith.constant dense<0.000000e+00> : tensor<128x128xf16, #mma>
    %0 = ttng.warp_group_dot_wait %cst_0 {pendings = 1 : i32} : tensor<128x128xf16, #mma>
    %1 = arith.addf %0, %0 : tensor<128x128xf16, #mma>
    tt.return
  }
}
""")
```
- **EN:** Prepares or updates state through `mod`. Invokes `run_parser`, `expecttest.assert_expected_inline`, `anonymize_ir`, `mod.str_nodebug` to execute the test logic. Relevant themes: debugging and inspection paths.
- **CN:** 通过 `mod` 准备或更新状态。 调用 `run_parser`、`expecttest.assert_expected_inline`、`anonymize_ir`、`mod.str_nodebug` 执行测试逻辑。 相关主题：调试与检查路径。

### Lines 849-852

```python


@gluon.jit
def async_tma_kernel(input_desc, XBLOCK: ttgl.constexpr):
```
- **EN:** Defines the helper function `async_tma_kernel`. Decorators: `gluon.jit`. Parameters: `input_desc`, `XBLOCK`. Key calls include `ttgl.allocate_shared_memory`, `mbarrier.init`, `tma.async_load`, `ttgl.static_assert`, `mbarrier.expect`, `mbarrier.wait`, and 4 more. This scope touches layout transformation reasoning.
- **CN:** 定义辅助函数 `async_tma_kernel`。 装饰器：`gluon.jit`。 参数：`input_desc`、`XBLOCK`。 关键调用包括 `ttgl.allocate_shared_memory`、`mbarrier.init`、`tma.async_load`、`ttgl.static_assert`、`mbarrier.expect`、`mbarrier.wait` 等另外 4 项。 该作用域涉及布局变换推理。

#### Lines 853-865

```python
    smem = ttgl.allocate_shared_memory(ttgl.float16, [XBLOCK, XBLOCK], input_desc.layout)
    bar = ttgl.allocate_shared_memory(ttgl.int64, [1], mbarrier.MBarrierLayout())
    mbarrier.init(bar, count=1)

    tma.async_load(input_desc, [0, 0], bar, smem)
    ttgl.static_assert(input_desc.block_type.nbytes == XBLOCK * XBLOCK * 2)
    mbarrier.expect(bar, input_desc.block_type.nbytes)
    mbarrier.wait(bar, 0)

    mbarrier.invalidate(bar)

    tma.async_copy_shared_to_global(input_desc, [0, 0], smem)
    tma.store_wait(0)
```
- **EN:** Prepares or updates state through `smem`, `bar`. Invokes `ttgl.allocate_shared_memory`, `mbarrier.MBarrierLayout`, `mbarrier.init`, `tma.async_load`, `ttgl.static_assert`, `mbarrier.expect`, and 4 more to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `smem`、`bar` 准备或更新状态。 调用 `ttgl.allocate_shared_memory`、`mbarrier.MBarrierLayout`、`mbarrier.init`、`tma.async_load`、`ttgl.static_assert`、`mbarrier.expect` 等另外 4 项 执行测试逻辑。 相关主题：布局变换推理。

### Lines 866-869

```python


@pytest.mark.parametrize("target", [HOPPER_TARGET, BLACKWELL_TARGET])
def test_async_tma(target):
```
- **EN:** Defines the test function `test_async_tma`. Decorators: `pytest.mark.parametrize('target', [HOPPER_TARGET, BLACKWELL_TARGET])`. Parameters: `target`. Key calls include `pytest.mark.parametrize`, `MockTensor`, `ttgl.NVMMASharedLayout`, `TensorDescriptor.from_tensor`, `run_parser`, `expecttest.assert_expected_inline`, and 3 more. This scope touches pytest parametrization, tensor/descriptor metadata, debugging and inspection paths, layout transformation reasoning.
- **CN:** 定义测试函数 `test_async_tma`。 装饰器：`pytest.mark.parametrize('target', [HOPPER_TARGET, BLACKWELL_TARGET])`。 参数：`target`。 关键调用包括 `pytest.mark.parametrize`、`MockTensor`、`ttgl.NVMMASharedLayout`、`TensorDescriptor.from_tensor`、`run_parser`、`expecttest.assert_expected_inline` 等另外 3 项。 该作用域涉及pytest 参数化、张量/描述符元数据、调试与检查路径、布局变换推理。

#### Lines 870-907

```python
    input = MockTensor(ttgl.float16, (1024, 1024))
    XBLOCK = 128
    shared_layout = ttgl.NVMMASharedLayout(swizzle_byte_width=128, element_bitwidth=16, rank=2)
    input_desc = TensorDescriptor.from_tensor(input, [XBLOCK, XBLOCK], shared_layout)

    mod = run_parser(
        async_tma_kernel,
        *make_args(input_desc, XBLOCK, num_warps=4),
        target=target,
    )
    expecttest.assert_expected_inline(
        anonymize_ir(mod.str_nodebug()), """\
#shared = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 16}>
#shared1 = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0]}>
#smem = #ttg.shared_memory
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "...", "ttg.threads-per-warp" = 32 : i32} {
  tt.func public @async_tma_kernel(%arg0: !tt.tensordesc<128x128xf16, #shared>, %arg1: i32, %arg2: i32, %arg3: i64, %arg4: i64) attributes {noinline = false} {
    %0 = ttg.local_alloc : () -> !ttg.memdesc<128x128xf16, #shared, #smem, mutable>
    %1 = ttg.local_alloc : () -> !ttg.memdesc<1xi64, #shared1, #smem, mutable>
    ttng.init_barrier %1, 1 : !ttg.memdesc<1xi64, #shared1, #smem, mutable>
    %c0_i32 = arith.constant 0 : i32
    %c0_i32_0 = arith.constant 0 : i32
    %true = arith.constant true
    ttng.async_tma_copy_global_to_local %arg0[%c0_i32, %c0_i32_0] %0, %1, %true : !tt.tensordesc<128x128xf16, #shared>, !ttg.memdesc<1xi64, #shared1, #smem, mutable> -> !ttg.memdesc<128x128xf16, #shared, #smem, mutable>
    %true_1 = arith.constant true
    ttng.barrier_expect %1, 32768, %true_1 : !ttg.memdesc<1xi64, #shared1, #smem, mutable>
    %c0_i32_2 = arith.constant 0 : i32
    %true_3 = arith.constant true
    ttng.wait_barrier %1, %c0_i32_2, %true_3 : !ttg.memdesc<1xi64, #shared1, #smem, mutable>
    ttng.inval_barrier %1 : !ttg.memdesc<1xi64, #shared1, #smem, mutable>
    %c0_i32_4 = arith.constant 0 : i32
    %c0_i32_5 = arith.constant 0 : i32
    ttng.async_tma_copy_local_to_global %arg0[%c0_i32_4, %c0_i32_5] %0 : !tt.tensordesc<128x128xf16, #shared>, !ttg.memdesc<128x128xf16, #shared, #smem, mutable>
    ttng.async_tma_store_wait {pendings = 0 : i32}
    tt.return
  }
}
""")
```
- **EN:** Prepares or updates state through `input`, `XBLOCK`, `shared_layout`, `input_desc`, `mod`. Invokes `MockTensor`, `ttgl.NVMMASharedLayout`, `TensorDescriptor.from_tensor`, `run_parser`, `make_args`, `expecttest.assert_expected_inline`, and 2 more to execute the test logic. Relevant themes: tensor/descriptor metadata, debugging and inspection paths, layout transformation reasoning.
- **CN:** 通过 `input`、`XBLOCK`、`shared_layout`、`input_desc`、`mod` 准备或更新状态。 调用 `MockTensor`、`ttgl.NVMMASharedLayout`、`TensorDescriptor.from_tensor`、`run_parser`、`make_args`、`expecttest.assert_expected_inline` 等另外 2 项 执行测试逻辑。 相关主题：张量/描述符元数据、调试与检查路径、布局变换推理。

### Lines 908-911

```python


@gluon.jit
def async_tma_blackwell_kernel(input_desc, XBLOCK: ttgl.constexpr):
```
- **EN:** Defines the helper function `async_tma_blackwell_kernel`. Decorators: `gluon.jit`. Parameters: `input_desc`, `XBLOCK`. Key calls include `ttgl.allocate_shared_memory`, `mbarrier.init`, `ttgl.BlockedLayout`, `ttgl.arange`, `tma.async_gather`, `mbarrier.expect`, and 6 more. This scope touches layout transformation reasoning.
- **CN:** 定义辅助函数 `async_tma_blackwell_kernel`。 装饰器：`gluon.jit`。 参数：`input_desc`、`XBLOCK`。 关键调用包括 `ttgl.allocate_shared_memory`、`mbarrier.init`、`ttgl.BlockedLayout`、`ttgl.arange`、`tma.async_gather`、`mbarrier.expect` 等另外 6 项。 该作用域涉及布局变换推理。

#### Lines 912-925

```python
    smem = ttgl.allocate_shared_memory(ttgl.float16, [XBLOCK, XBLOCK], input_desc.layout)
    bar = ttgl.allocate_shared_memory(ttgl.int64, [1], mbarrier.MBarrierLayout())
    mbarrier.init(bar, count=1)

    offset_layout: ttgl.constexpr = ttgl.BlockedLayout([1, 4], [32, 1], [1, 4], [1, 0])
    x_offsets = ttgl.arange(0, XBLOCK, layout=ttgl.SliceLayout(0, offset_layout))
    tma.async_gather(input_desc, x_offsets, 0, bar, smem)
    mbarrier.expect(bar, XBLOCK * XBLOCK * ttgl.float16.primitive_bitwidth // 8)
    mbarrier.wait(bar, 0)

    mbarrier.invalidate(bar)

    tma.async_scatter(input_desc, x_offsets, 0, smem)
    tma.store_wait(0)
```
- **EN:** Prepares or updates state through `smem`, `bar`, `offset_layout`, `x_offsets`. Invokes `ttgl.allocate_shared_memory`, `mbarrier.MBarrierLayout`, `mbarrier.init`, `ttgl.BlockedLayout`, `ttgl.arange`, `ttgl.SliceLayout`, and 6 more to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `smem`、`bar`、`offset_layout`、`x_offsets` 准备或更新状态。 调用 `ttgl.allocate_shared_memory`、`mbarrier.MBarrierLayout`、`mbarrier.init`、`ttgl.BlockedLayout`、`ttgl.arange`、`ttgl.SliceLayout` 等另外 6 项 执行测试逻辑。 相关主题：布局变换推理。

### Lines 926-928

```python


def test_async_tma_blackwell():
```
- **EN:** Defines the test function `test_async_tma_blackwell`. Key calls include `MockTensor`, `ttgl.NVMMASharedLayout`, `TensorDescriptor.from_tensor`, `run_parser`, `expecttest.assert_expected_inline`, `anonymize_ir`, and 2 more. This scope touches tensor/descriptor metadata, debugging and inspection paths, layout transformation reasoning.
- **CN:** 定义测试函数 `test_async_tma_blackwell`。 关键调用包括 `MockTensor`、`ttgl.NVMMASharedLayout`、`TensorDescriptor.from_tensor`、`run_parser`、`expecttest.assert_expected_inline`、`anonymize_ir` 等另外 2 项。 该作用域涉及张量/描述符元数据、调试与检查路径、布局变换推理。

#### Lines 929-966

```python
    input = MockTensor(ttgl.float16, (1024, 1024))
    XBLOCK = 128
    shared_layout = ttgl.NVMMASharedLayout(swizzle_byte_width=128, element_bitwidth=16, rank=2)
    input_desc = TensorDescriptor.from_tensor(input, [1, XBLOCK], shared_layout)

    mod = run_parser(
        async_tma_blackwell_kernel,
        *make_args(input_desc, XBLOCK, num_warps=4),
        target=BLACKWELL_TARGET,
    )
    expecttest.assert_expected_inline(
        anonymize_ir(mod.str_nodebug()), """\
#blocked = #ttg.blocked<{sizePerThread = [1, 4], threadsPerWarp = [32, 1], warpsPerCTA = [1, 4], order = [1, 0]}>
#shared = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 16}>
#shared1 = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0]}>
#smem = #ttg.shared_memory
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "...", "ttg.threads-per-warp" = 32 : i32} {
  tt.func public @async_tma_blackwell_kernel(%arg0: !tt.tensordesc<1x128xf16, #shared>, %arg1: i32, %arg2: i32, %arg3: i64, %arg4: i64) attributes {noinline = false} {
    %0 = ttg.local_alloc : () -> !ttg.memdesc<128x128xf16, #shared, #smem, mutable>
    %1 = ttg.local_alloc : () -> !ttg.memdesc<1xi64, #shared1, #smem, mutable>
    ttng.init_barrier %1, 1 : !ttg.memdesc<1xi64, #shared1, #smem, mutable>
    %2 = tt.make_range {end = 128 : i32, start = 0 : i32} : tensor<128xi32, #ttg.slice<{dim = 0, parent = #blocked}>>
    %true = arith.constant true
    %c0_i32 = arith.constant 0 : i32
    ttng.async_tma_gather %arg0[%2, %c0_i32] %0, %1, %true : !tt.tensordesc<1x128xf16, #shared>, tensor<128xi32, #ttg.slice<{dim = 0, parent = #blocked}>>, i32, !ttg.memdesc<1xi64, #shared1, #smem, mutable>, !ttg.memdesc<128x128xf16, #shared, #smem, mutable>, i1
    %true_0 = arith.constant true
    ttng.barrier_expect %1, 32768, %true_0 : !ttg.memdesc<1xi64, #shared1, #smem, mutable>
    %c0_i32_1 = arith.constant 0 : i32
    %true_2 = arith.constant true
    ttng.wait_barrier %1, %c0_i32_1, %true_2 : !ttg.memdesc<1xi64, #shared1, #smem, mutable>
    ttng.inval_barrier %1 : !ttg.memdesc<1xi64, #shared1, #smem, mutable>
    %c0_i32_3 = arith.constant 0 : i32
    ttng.async_tma_scatter %arg0[%2, %c0_i32_3] %0 : !tt.tensordesc<1x128xf16, #shared>, tensor<128xi32, #ttg.slice<{dim = 0, parent = #blocked}>>, i32, !ttg.memdesc<128x128xf16, #shared, #smem, mutable>
    ttng.async_tma_store_wait {pendings = 0 : i32}
    tt.return
  }
}
""")
```
- **EN:** Prepares or updates state through `input`, `XBLOCK`, `shared_layout`, `input_desc`, `mod`. Invokes `MockTensor`, `ttgl.NVMMASharedLayout`, `TensorDescriptor.from_tensor`, `run_parser`, `make_args`, `expecttest.assert_expected_inline`, and 2 more to execute the test logic. Relevant themes: tensor/descriptor metadata, debugging and inspection paths, layout transformation reasoning.
- **CN:** 通过 `input`、`XBLOCK`、`shared_layout`、`input_desc`、`mod` 准备或更新状态。 调用 `MockTensor`、`ttgl.NVMMASharedLayout`、`TensorDescriptor.from_tensor`、`run_parser`、`make_args`、`expecttest.assert_expected_inline` 等另外 2 项 执行测试逻辑。 相关主题：张量/描述符元数据、调试与检查路径、布局变换推理。

### Lines 967-971

```python


def test_mlir_attr_error():

    @gluon.jit
```
- **EN:** Defines the test function `test_mlir_attr_error`. Nested definitions in this scope: `kernel`. Key calls include `ttgl.arange`, `pytest.raises`, `run_parser`, `ttgl.BlockedLayout`. This scope touches layout transformation reasoning.
- **CN:** 定义测试函数 `test_mlir_attr_error`。 该作用域中的嵌套定义：`kernel`。 关键调用包括 `ttgl.arange`、`pytest.raises`、`run_parser`、`ttgl.BlockedLayout`。 该作用域涉及布局变换推理。

#### Lines 971-972

```python
    @gluon.jit
    def kernel():
```
- **EN:** Defines the helper function `kernel`. Decorators: `gluon.jit`. Key calls include `ttgl.arange`, `ttgl.BlockedLayout`. This scope touches layout transformation reasoning.
- **CN:** 定义辅助函数 `kernel`。 装饰器：`gluon.jit`。 关键调用包括 `ttgl.arange`、`ttgl.BlockedLayout`。 该作用域涉及布局变换推理。

##### Lines 973-973

```python
        ttgl.arange(0, 1, layout=ttgl.BlockedLayout([1], [32], [4], [1]))
```
- **EN:** Invokes `ttgl.arange`, `ttgl.BlockedLayout` to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 调用 `ttgl.arange`、`ttgl.BlockedLayout` 执行测试逻辑。 相关主题：布局变换推理。

#### Lines 974-976

```python

    with pytest.raises(CompilationError) as e:
        run_parser(kernel)
```
- **EN:** Invokes `pytest.raises`, `run_parser` to execute the test logic. Uses `pytest.raises` to confirm expected failure paths. Uses context managers to control resources or expectations.
- **CN:** 调用 `pytest.raises`、`run_parser` 执行测试逻辑。 使用 `pytest.raises` 确认预期的失败路径。 使用上下文管理器控制资源或预期行为。

#### Lines 977-978

```python

    assert "order must be a permutation of 0..(rank-1), but was [1]" in str(e.value.__cause__)
```
- **EN:** Validates behavior with 1 assertion(s).
- **CN:** 通过 1 个断言验证行为。

### Lines 979-983

```python


def test_tensor_layout_type_changed():

    @gluon.jit
```
- **EN:** Defines the test function `test_tensor_layout_type_changed`. Nested definitions in this scope: `kernel`. Key calls include `ttgl.BlockedLayout`, `ttgl.zeros`, `ttgl.to_tensor`, `pytest.raises`, `run_parser`, `y.sum`. This scope touches layout transformation reasoning.
- **CN:** 定义测试函数 `test_tensor_layout_type_changed`。 该作用域中的嵌套定义：`kernel`。 关键调用包括 `ttgl.BlockedLayout`、`ttgl.zeros`、`ttgl.to_tensor`、`pytest.raises`、`run_parser`、`y.sum`。 该作用域涉及布局变换推理。

#### Lines 983-984

```python
    @gluon.jit
    def kernel():
```
- **EN:** Defines the helper function `kernel`. Decorators: `gluon.jit`. Key calls include `ttgl.BlockedLayout`, `ttgl.zeros`, `ttgl.to_tensor`, `y.sum`. This scope touches layout transformation reasoning.
- **CN:** 定义辅助函数 `kernel`。 装饰器：`gluon.jit`。 关键调用包括 `ttgl.BlockedLayout`、`ttgl.zeros`、`ttgl.to_tensor`、`y.sum`。 该作用域涉及布局变换推理。

##### Lines 985-989

```python
        layout: ttgl.constexpr = ttgl.BlockedLayout(size_per_thread=[1, 1], threads_per_warp=[1, 32],
                                                    warps_per_cta=[1, 4], order=[1, 0])
        x = ttgl.zeros([128], ttgl.float32)
        y = ttgl.zeros([128, 128], ttgl.float32, layout=layout)
        c = ttgl.to_tensor(True)
```
- **EN:** Prepares or updates state through `layout`, `x`, `y`, `c`. Invokes `ttgl.BlockedLayout`, `ttgl.zeros`, `ttgl.to_tensor` to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `layout`、`x`、`y`、`c` 准备或更新状态。 调用 `ttgl.BlockedLayout`、`ttgl.zeros`、`ttgl.to_tensor` 执行测试逻辑。 相关主题：布局变换推理。

##### Lines 990-991

```python
        while c:
            x = x + y.sum(axis=0)
```
- **EN:** Invokes `y.sum` to execute the test logic. Iterates across cases or data tiles.
- **CN:** 调用 `y.sum` 执行测试逻辑。 通过循环覆盖多个用例或数据分块。

#### Lines 992-994

```python

    with pytest.raises(CompilationError) as e:
        run_parser(kernel)
```
- **EN:** Invokes `pytest.raises`, `run_parser` to execute the test logic. Uses `pytest.raises` to confirm expected failure paths. Uses context managers to control resources or expectations.
- **CN:** 调用 `pytest.raises`、`run_parser` 执行测试逻辑。 使用 `pytest.raises` 确认预期的失败路径。 使用上下文管理器控制资源或预期行为。

#### Lines 995-996

```python

    assert "Loop-carried variable x has initial type" in str(e.value)
```
- **EN:** Validates behavior with 1 assertion(s).
- **CN:** 通过 1 个断言验证行为。

### Lines 997-1000

```python


@gluon.jit
def tmem_index_kernel():
```
- **EN:** Defines the helper function `tmem_index_kernel`. Decorators: `gluon.jit`. Key calls include `TensorMemoryLayout`, `ttgl.nvidia.blackwell.allocate_tensor_memory`, `tmem.index`. This scope touches layout transformation reasoning.
- **CN:** 定义辅助函数 `tmem_index_kernel`。 装饰器：`gluon.jit`。 关键调用包括 `TensorMemoryLayout`、`ttgl.nvidia.blackwell.allocate_tensor_memory`、`tmem.index`。 该作用域涉及布局变换推理。

#### Lines 1001-1003

```python
    layout: ttgl.constexpr = TensorMemoryLayout(block=[128, 128], col_stride=1)
    tmem = ttgl.nvidia.blackwell.allocate_tensor_memory(ttgl.int32, [2, 256, 256], layout)
    tmem.index(0)
```
- **EN:** Prepares or updates state through `layout`, `tmem`. Invokes `TensorMemoryLayout`, `ttgl.nvidia.blackwell.allocate_tensor_memory`, `tmem.index` to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `layout`、`tmem` 准备或更新状态。 调用 `TensorMemoryLayout`、`ttgl.nvidia.blackwell.allocate_tensor_memory`、`tmem.index` 执行测试逻辑。 相关主题：布局变换推理。

### Lines 1004-1006

```python


def test_tmem_index_constexpr():
```
- **EN:** Defines the test function `test_tmem_index_constexpr`. Key calls include `expecttest.assert_expected_inline`, `anonymize_ir`, `run_parser`. This scope touches debugging and inspection paths.
- **CN:** 定义测试函数 `test_tmem_index_constexpr`。 关键调用包括 `expecttest.assert_expected_inline`、`anonymize_ir`、`run_parser`。 该作用域涉及调试与检查路径。

#### Lines 1007-1018

```python
    expecttest.assert_expected_inline(
        anonymize_ir(run_parser(tmem_index_kernel).str_nodebug()), """\
#tmem = #ttng.tensor_memory_encoding<blockM = 128, blockN = 128, colStride = 1>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "...", "ttg.threads-per-warp" = 32 : i32} {
  tt.func public @tmem_index_kernel() attributes {noinline = false} {
    %result = ttng.tmem_alloc : () -> !ttg.memdesc<2x256x256xi32, #tmem, #ttng.tensor_memory, mutable>
    %c0_i32 = arith.constant 0 : i32
    %0 = ttg.memdesc_index %result[%c0_i32] : !ttg.memdesc<2x256x256xi32, #tmem, #ttng.tensor_memory, mutable> -> !ttg.memdesc<256x256xi32, #tmem, #ttng.tensor_memory, mutable>
    tt.return
  }
}
""")
```
- **EN:** Invokes `expecttest.assert_expected_inline`, `anonymize_ir`, `run_parser` to execute the test logic. Relevant themes: debugging and inspection paths.
- **CN:** 调用 `expecttest.assert_expected_inline`、`anonymize_ir`、`run_parser` 执行测试逻辑。 相关主题：调试与检查路径。

### Lines 1019-1022

```python


@gluon.jit
def tmem_subslice_reg_layout_kernel():
```
- **EN:** Defines the helper function `tmem_subslice_reg_layout_kernel`. Decorators: `gluon.jit`. Key calls include `TensorMemoryLayout`, `ttgl.nvidia.blackwell.allocate_tensor_memory`, `sub.load`, `tmem.index`. This scope touches layout transformation reasoning.
- **CN:** 定义辅助函数 `tmem_subslice_reg_layout_kernel`。 装饰器：`gluon.jit`。 关键调用包括 `TensorMemoryLayout`、`ttgl.nvidia.blackwell.allocate_tensor_memory`、`sub.load`、`tmem.index`。 该作用域涉及布局变换推理。

#### Lines 1023-1026

```python
    layout: ttgl.constexpr = TensorMemoryLayout(block=[128, 256], col_stride=1, cga_layout=((1, 0), (2, 0)))
    tmem = ttgl.nvidia.blackwell.allocate_tensor_memory(ttgl.float32, [2, 512, 256], layout)
    sub = tmem.index(0).slice(0, 32)
    _ = sub.load()
```
- **EN:** Prepares or updates state through `layout`, `tmem`, `sub`, `_`. Invokes `TensorMemoryLayout`, `ttgl.nvidia.blackwell.allocate_tensor_memory`, `tmem.index`, `sub.load` to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `layout`、`tmem`、`sub`、`_` 准备或更新状态。 调用 `TensorMemoryLayout`、`ttgl.nvidia.blackwell.allocate_tensor_memory`、`tmem.index`、`sub.load` 执行测试逻辑。 相关主题：布局变换推理。

### Lines 1027-1029

```python


def test_tmem_subslice_reg_layout_constexpr():
```
- **EN:** Defines the test function `test_tmem_subslice_reg_layout_constexpr`. Key calls include `expecttest.assert_expected_inline`, `anonymize_ir`, `run_parser`, `make_args`. This scope touches debugging and inspection paths, layout transformation reasoning.
- **CN:** 定义测试函数 `test_tmem_subslice_reg_layout_constexpr`。 关键调用包括 `expecttest.assert_expected_inline`、`anonymize_ir`、`run_parser`、`make_args`。 该作用域涉及调试与检查路径、布局变换推理。

#### Lines 1030-1049

```python
    expecttest.assert_expected_inline(
        anonymize_ir(
            run_parser(
                tmem_subslice_reg_layout_kernel,
                *make_args(num_warps=4, num_ctas=4),
                target=BLACKWELL_TARGET,
            ).str_nodebug()), """\
#linear = #ttg.linear<{register = [[0, 1], [0, 2], [0, 4], [0, 8], [0, 16]], lane = [[1, 0], [2, 0], [4, 0], [8, 0], [16, 0]], warp = [[32, 0], [64, 0]], block = [[128, 0], [256, 0]]}>
#tmem = #ttng.tensor_memory_encoding<blockM = 128, blockN = 256, colStride = 1, CGALayout = [[1, 0], [2, 0]]>
module attributes {"ttg.num-ctas" = 4 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "...", "ttg.threads-per-warp" = 32 : i32} {
  tt.func public @tmem_subslice_reg_layout_kernel() attributes {noinline = false} {
    %result = ttng.tmem_alloc : () -> !ttg.memdesc<2x512x256xf32, #tmem, #ttng.tensor_memory, mutable>
    %c0_i32 = arith.constant 0 : i32
    %0 = ttg.memdesc_index %result[%c0_i32] : !ttg.memdesc<2x512x256xf32, #tmem, #ttng.tensor_memory, mutable> -> !ttg.memdesc<512x256xf32, #tmem, #ttng.tensor_memory, mutable>
    %1 = ttng.tmem_subslice %0 {N = 0 : i32} : !ttg.memdesc<512x256xf32, #tmem, #ttng.tensor_memory, mutable> -> !ttg.memdesc<512x32xf32, #tmem, #ttng.tensor_memory, mutable, 512x256>
    %result_0 = ttng.tmem_load %1 : !ttg.memdesc<512x32xf32, #tmem, #ttng.tensor_memory, mutable, 512x256> -> tensor<512x32xf32, #linear>
    tt.return
  }
}
""")
```
- **EN:** Invokes `expecttest.assert_expected_inline`, `anonymize_ir`, `run_parser`, `make_args` to execute the test logic. Relevant themes: debugging and inspection paths, layout transformation reasoning.
- **CN:** 调用 `expecttest.assert_expected_inline`、`anonymize_ir`、`run_parser`、`make_args` 执行测试逻辑。 相关主题：调试与检查路径、布局变换推理。

### Lines 1050-1055

```python


@filecheck_test
@gluon.jit
def test_tmem_reduction_default_layout_constexpr():
    # CHECK-LABEL: @test_tmem_reduction_default_layout_constexpr
```
- **EN:** Defines the test function `test_tmem_reduction_default_layout_constexpr`. Decorators: `filecheck_test`, `gluon.jit`. Key calls include `TensorMemoryLayout`, `ttgl.nvidia.blackwell.allocate_tensor_memory`, `tmem.load_min`, `tmem.load_max`. This scope touches Triton language operations, layout transformation reasoning.
- **CN:** 定义测试函数 `test_tmem_reduction_default_layout_constexpr`。 装饰器：`filecheck_test`、`gluon.jit`。 关键调用包括 `TensorMemoryLayout`、`ttgl.nvidia.blackwell.allocate_tensor_memory`、`tmem.load_min`、`tmem.load_max`。 该作用域涉及Triton language 操作、布局变换推理。

#### Lines 1056-1063

```python
    layout: ttgl.constexpr = TensorMemoryLayout(block=[128, 128], col_stride=1)
    tmem = ttgl.nvidia.blackwell.allocate_tensor_memory(ttgl.float32, [128, 128], layout)
    # CHECK: ttng.tmem_load {{.*}} {abs = true, redOp = #ttng.redOp<min>}
    _ = tmem.load_min(abs=True)
    # CHECK: ttng.tmem_load {{.*}} {NaN = true, redOp = #ttng.redOp<max>}
    # CHECK-NOT: ttng.tmem_load
    # CHECK: tt.return
    _ = tmem.load_max(propagate_nan=tl.PropagateNan.ALL)
```
- **EN:** Prepares or updates state through `layout`, `tmem`, `_`. Invokes `TensorMemoryLayout`, `ttgl.nvidia.blackwell.allocate_tensor_memory`, `tmem.load_min`, `tmem.load_max` to execute the test logic. Relevant themes: Triton language operations, layout transformation reasoning.
- **CN:** 通过 `layout`、`tmem`、`_` 准备或更新状态。 调用 `TensorMemoryLayout`、`ttgl.nvidia.blackwell.allocate_tensor_memory`、`tmem.load_min`、`tmem.load_max` 执行测试逻辑。 相关主题：Triton language 操作、布局变换推理。

### Lines 1064-1067

```python


@gluon.jit
def smem_and_layout_user(smem, a: ttgl.constexpr):
```
- **EN:** Defines the helper function `smem_and_layout_user`. Decorators: `gluon.jit`. Parameters: `smem`, `a`. This scope touches layout transformation reasoning.
- **CN:** 定义辅助函数 `smem_and_layout_user`。 装饰器：`gluon.jit`。 参数：`smem`、`a`。 该作用域涉及布局变换推理。

#### Lines 1068-1068

```python
    pass
```
- **EN:** Carries supporting logic for the surrounding test or helper scope.
- **CN:** 为周围的测试或辅助作用域提供支撑逻辑。

### Lines 1069-1073

```python


def test_layout_mangling():

    @gluon.jit
```
- **EN:** Defines the test function `test_layout_mangling`. Nested definitions in this scope: `kernel`. Key calls include `expecttest.assert_expected_inline`, `ttgl.SwizzledSharedLayout`, `ttgl.allocate_shared_memory`, `smem_and_layout_user`, `anonymize_ir`, `run_parser`. This scope touches debugging and inspection paths, layout transformation reasoning.
- **CN:** 定义测试函数 `test_layout_mangling`。 该作用域中的嵌套定义：`kernel`。 关键调用包括 `expecttest.assert_expected_inline`、`ttgl.SwizzledSharedLayout`、`ttgl.allocate_shared_memory`、`smem_and_layout_user`、`anonymize_ir`、`run_parser`。 该作用域涉及调试与检查路径、布局变换推理。

#### Lines 1073-1074

```python
    @gluon.jit
    def kernel():
```
- **EN:** Defines the helper function `kernel`. Decorators: `gluon.jit`. Key calls include `ttgl.SwizzledSharedLayout`, `ttgl.allocate_shared_memory`, `smem_and_layout_user`. This scope touches layout transformation reasoning.
- **CN:** 定义辅助函数 `kernel`。 装饰器：`gluon.jit`。 关键调用包括 `ttgl.SwizzledSharedLayout`、`ttgl.allocate_shared_memory`、`smem_and_layout_user`。 该作用域涉及布局变换推理。

##### Lines 1075-1077

```python
        a: ttgl.constexpr = ttgl.SwizzledSharedLayout(1, 1, 1, [1, 0])
        smem = ttgl.allocate_shared_memory(ttgl.int32, [32, 32], a)
        smem_and_layout_user(smem, a)
```
- **EN:** Prepares or updates state through `a`, `smem`. Invokes `ttgl.SwizzledSharedLayout`, `ttgl.allocate_shared_memory`, `smem_and_layout_user` to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `a`、`smem` 准备或更新状态。 调用 `ttgl.SwizzledSharedLayout`、`ttgl.allocate_shared_memory`、`smem_and_layout_user` 执行测试逻辑。 相关主题：布局变换推理。

#### Lines 1078-1093

```python

    expecttest.assert_expected_inline(
        anonymize_ir(run_parser(kernel).str_nodebug()), """\
#shared = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [1, 0]}>
#smem = #ttg.shared_memory
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "...", "ttg.threads-per-warp" = 32 : i32} {
  tt.func public @kernel() attributes {noinline = false} {
    %0 = ttg.local_alloc : () -> !ttg.memdesc<32x32xi32, #shared, #smem, mutable>
    tt.call @test_frontend.smem_and_layout_user__MDi32S32_32SLSSS_1_1_1_1_0__SSSLAS32_32ASMD_cSSS_1_1_1_1_0__SSS(%0) : (!ttg.memdesc<32x32xi32, #shared, #smem, mutable>) -> ()
    tt.return
  }
  tt.func private @test_frontend.smem_and_layout_user__MDi32S32_32SLSSS_1_1_1_1_0__SSSLAS32_32ASMD_cSSS_1_1_1_1_0__SSS(%arg0: !ttg.memdesc<32x32xi32, #shared, #smem, mutable>) attributes {noinline = false} {
    tt.return
  }
}
""")
```
- **EN:** Invokes `expecttest.assert_expected_inline`, `anonymize_ir`, `run_parser` to execute the test logic. Relevant themes: debugging and inspection paths, layout transformation reasoning.
- **CN:** 调用 `expecttest.assert_expected_inline`、`anonymize_ir`、`run_parser` 执行测试逻辑。 相关主题：调试与检查路径、布局变换推理。

### Lines 1094-1097

```python


@gluon.jit
def broadcast_kernel():
```
- **EN:** Defines the helper function `broadcast_kernel`. Decorators: `gluon.jit`. Key calls include `ttgl.BlockedLayout`, `ttgl.arange`, `ttgl.SliceLayout`. This scope touches layout transformation reasoning.
- **CN:** 定义辅助函数 `broadcast_kernel`。 装饰器：`gluon.jit`。 关键调用包括 `ttgl.BlockedLayout`、`ttgl.arange`、`ttgl.SliceLayout`。 该作用域涉及布局变换推理。

#### Lines 1098-1101

```python
    layout: ttgl.constexpr = ttgl.BlockedLayout([1, 1], [2, 16], [4, 1], [1, 0])
    a = ttgl.arange(0, 16, layout=ttgl.SliceLayout(0, layout))[None, :]
    b = ttgl.arange(0, 16, layout=ttgl.SliceLayout(1, layout))[:, None]
    0 + a + b
```
- **EN:** Prepares or updates state through `layout`, `a`, `b`. Invokes `ttgl.BlockedLayout`, `ttgl.arange`, `ttgl.SliceLayout` to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `layout`、`a`、`b` 准备或更新状态。 调用 `ttgl.BlockedLayout`、`ttgl.arange`、`ttgl.SliceLayout` 执行测试逻辑。 相关主题：布局变换推理。

### Lines 1102-1105

```python


@pytest.mark.parametrize("target", ALL_TARGETS)
def test_broadcast(target):
```
- **EN:** Defines the test function `test_broadcast`. Decorators: `pytest.mark.parametrize('target', ALL_TARGETS)`. Parameters: `target`. Key calls include `pytest.mark.parametrize`, `run_parser`, `expecttest.assert_expected_inline`, `anonymize_ir`, `mod.str_nodebug`. This scope touches pytest parametrization, debugging and inspection paths.
- **CN:** 定义测试函数 `test_broadcast`。 装饰器：`pytest.mark.parametrize('target', ALL_TARGETS)`。 参数：`target`。 关键调用包括 `pytest.mark.parametrize`、`run_parser`、`expecttest.assert_expected_inline`、`anonymize_ir`、`mod.str_nodebug`。 该作用域涉及pytest 参数化、调试与检查路径。

#### Lines 1106-1126

```python
    mod = run_parser(broadcast_kernel, target=target)
    expecttest.assert_expected_inline(
        anonymize_ir(mod.str_nodebug()), """\
#blocked = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [2, 16], warpsPerCTA = [4, 1], order = [1, 0]}>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "...", "ttg.threads-per-warp" = 32 : i32} {
  tt.func public @broadcast_kernel() attributes {noinline = false} {
    %0 = tt.make_range {end = 16 : i32, start = 0 : i32} : tensor<16xi32, #ttg.slice<{dim = 0, parent = #blocked}>>
    %1 = tt.expand_dims %0 {axis = 0 : i32} : tensor<16xi32, #ttg.slice<{dim = 0, parent = #blocked}>> -> tensor<1x16xi32, #blocked>
    %2 = tt.make_range {end = 16 : i32, start = 0 : i32} : tensor<16xi32, #ttg.slice<{dim = 1, parent = #blocked}>>
    %3 = tt.expand_dims %2 {axis = 1 : i32} : tensor<16xi32, #ttg.slice<{dim = 1, parent = #blocked}>> -> tensor<16x1xi32, #blocked>
    %c0_i32 = arith.constant 0 : i32
    %c0_i32_0 = arith.constant 0 : i32
    %cst = arith.constant dense<0> : tensor<1x16xi32, #blocked>
    %4 = arith.addi %cst, %1 : tensor<1x16xi32, #blocked>
    %5 = tt.broadcast %4 : tensor<1x16xi32, #blocked> -> tensor<16x16xi32, #blocked>
    %6 = tt.broadcast %3 : tensor<16x1xi32, #blocked> -> tensor<16x16xi32, #blocked>
    %7 = arith.addi %5, %6 : tensor<16x16xi32, #blocked>
    tt.return
  }
}
""")
```
- **EN:** Prepares or updates state through `mod`. Invokes `run_parser`, `expecttest.assert_expected_inline`, `anonymize_ir`, `mod.str_nodebug` to execute the test logic. Relevant themes: debugging and inspection paths.
- **CN:** 通过 `mod` 准备或更新状态。 调用 `run_parser`、`expecttest.assert_expected_inline`、`anonymize_ir`、`mod.str_nodebug` 执行测试逻辑。 相关主题：调试与检查路径。

### Lines 1127-1130

```python


@gluon.jit
def math_kernel():
```
- **EN:** Defines the helper function `math_kernel`. Decorators: `gluon.jit`. Key calls include `ttgl.BlockedLayout`, `ttgl.full`, `ttgl.umulhi`, `ttgl.exp`, `ttgl.exp2`, `ttgl.log`, and 13 more. This scope touches layout transformation reasoning.
- **CN:** 定义辅助函数 `math_kernel`。 装饰器：`gluon.jit`。 关键调用包括 `ttgl.BlockedLayout`、`ttgl.full`、`ttgl.umulhi`、`ttgl.exp`、`ttgl.exp2`、`ttgl.log` 等另外 13 项。 该作用域涉及布局变换推理。

#### Lines 1131-1153

```python
    layout: ttgl.constexpr = ttgl.BlockedLayout([1, 1], [1, 32], [4, 1], [1, 0])
    a = ttgl.full([16, 16], 1, ttgl.float32, layout)
    b = ttgl.full([16, 16], 2, ttgl.float32, layout)
    c = ttgl.full([16, 16], 4, ttgl.float32, layout)
    d = ttgl.full([16, 16], 1, ttgl.int32, layout)
    e = ttgl.full([16, 16], 1, ttgl.int32, layout)
    ttgl.umulhi(d, e)
    ttgl.exp(a)
    ttgl.exp2(a)
    ttgl.log(a)
    ttgl.log2(a)
    ttgl.cos(a)
    ttgl.sin(a)
    ttgl.sqrt(a)
    ttgl.sqrt_rn(a)
    ttgl.rsqrt(a)
    ttgl.abs(a)
    ttgl.fdiv(a, b)
    ttgl.div_rn(a, b)
    ttgl.erf(a)
    ttgl.floor(a)
    ttgl.ceil(a)
    ttgl.fma(a, b, c)
```
- **EN:** Prepares or updates state through `layout`, `a`, `b`, `c`, `d`, `e`. Invokes `ttgl.BlockedLayout`, `ttgl.full`, `ttgl.umulhi`, `ttgl.exp`, `ttgl.exp2`, `ttgl.log`, and 13 more to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `layout`、`a`、`b`、`c`、`d`、`e` 准备或更新状态。 调用 `ttgl.BlockedLayout`、`ttgl.full`、`ttgl.umulhi`、`ttgl.exp`、`ttgl.exp2`、`ttgl.log` 等另外 13 项 执行测试逻辑。 相关主题：布局变换推理。

### Lines 1154-1157

```python


@pytest.mark.parametrize("target", ALL_TARGETS)
def test_math(target):
```
- **EN:** Defines the test function `test_math`. Decorators: `pytest.mark.parametrize('target', ALL_TARGETS)`. Parameters: `target`. Key calls include `pytest.mark.parametrize`, `run_parser`, `expecttest.assert_expected_inline`, `anonymize_ir`, `mod.str_nodebug`. This scope touches pytest parametrization, debugging and inspection paths.
- **CN:** 定义测试函数 `test_math`。 装饰器：`pytest.mark.parametrize('target', ALL_TARGETS)`。 参数：`target`。 关键调用包括 `pytest.mark.parametrize`、`run_parser`、`expecttest.assert_expected_inline`、`anonymize_ir`、`mod.str_nodebug`。 该作用域涉及pytest 参数化、调试与检查路径。

#### Lines 1158-1194

```python
    mod = run_parser(math_kernel, target=target)
    expecttest.assert_expected_inline(
        anonymize_ir(mod.str_nodebug()), """\
#blocked = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [1, 32], warpsPerCTA = [4, 1], order = [1, 0]}>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "...", "ttg.threads-per-warp" = 32 : i32} {
  tt.func public @math_kernel() attributes {noinline = false} {
    %cst = arith.constant 1.000000e+00 : f32
    %cst_0 = arith.constant dense<1.000000e+00> : tensor<16x16xf32, #blocked>
    %cst_1 = arith.constant 2.000000e+00 : f32
    %cst_2 = arith.constant dense<2.000000e+00> : tensor<16x16xf32, #blocked>
    %cst_3 = arith.constant 4.000000e+00 : f32
    %cst_4 = arith.constant dense<4.000000e+00> : tensor<16x16xf32, #blocked>
    %c1_i32 = arith.constant 1 : i32
    %cst_5 = arith.constant dense<1> : tensor<16x16xi32, #blocked>
    %c1_i32_6 = arith.constant 1 : i32
    %cst_7 = arith.constant dense<1> : tensor<16x16xi32, #blocked>
    %0 = tt.mulhiui %cst_5, %cst_7 : tensor<16x16xi32, #blocked>
    %1 = math.exp %cst_0 : tensor<16x16xf32, #blocked>
    %2 = math.exp2 %cst_0 : tensor<16x16xf32, #blocked>
    %3 = math.log %cst_0 : tensor<16x16xf32, #blocked>
    %4 = math.log2 %cst_0 : tensor<16x16xf32, #blocked>
    %5 = math.cos %cst_0 : tensor<16x16xf32, #blocked>
    %6 = math.sin %cst_0 : tensor<16x16xf32, #blocked>
    %7 = math.sqrt %cst_0 : tensor<16x16xf32, #blocked>
    %8 = tt.precise_sqrt %cst_0 : tensor<16x16xf32, #blocked>
    %9 = math.rsqrt %cst_0 : tensor<16x16xf32, #blocked>
    %10 = math.absf %cst_0 : tensor<16x16xf32, #blocked>
    %11 = arith.divf %cst_0, %cst_2 : tensor<16x16xf32, #blocked>
    %12 = tt.precise_divf %cst_0, %cst_2 : tensor<16x16xf32, #blocked>
    %13 = math.erf %cst_0 : tensor<16x16xf32, #blocked>
    %14 = math.floor %cst_0 : tensor<16x16xf32, #blocked>
    %15 = math.ceil %cst_0 : tensor<16x16xf32, #blocked>
    %16 = math.fma %cst_0, %cst_2, %cst_4 : tensor<16x16xf32, #blocked>
    tt.return
  }
}
""")
```
- **EN:** Prepares or updates state through `mod`. Invokes `run_parser`, `expecttest.assert_expected_inline`, `anonymize_ir`, `mod.str_nodebug` to execute the test logic. Relevant themes: debugging and inspection paths.
- **CN:** 通过 `mod` 准备或更新状态。 调用 `run_parser`、`expecttest.assert_expected_inline`、`anonymize_ir`、`mod.str_nodebug` 执行测试逻辑。 相关主题：调试与检查路径。

### Lines 1195-1198

```python


@gluon.jit
def libdevice_kernel():
```
- **EN:** Defines the helper function `libdevice_kernel`. Decorators: `gluon.jit`. Key calls include `ttgl.BlockedLayout`, `ttgl.full`, `libdevice.abs`, `libdevice.fast_dividef`, `libdevice.fma`, `libdevice.isnan`, and 1 more. This scope touches layout transformation reasoning, libdevice coverage.
- **CN:** 定义辅助函数 `libdevice_kernel`。 装饰器：`gluon.jit`。 关键调用包括 `ttgl.BlockedLayout`、`ttgl.full`、`libdevice.abs`、`libdevice.fast_dividef`、`libdevice.fma`、`libdevice.isnan` 等另外 1 项。 该作用域涉及布局变换推理、libdevice 覆盖。

#### Lines 1199-1209

```python
    layout: ttgl.constexpr = ttgl.BlockedLayout([1, 1], [1, 32], [4, 1], [1, 0])
    a = ttgl.full([4, 32], 1, ttgl.float32, layout)
    b = ttgl.full([4, 32], 2, ttgl.float32, layout)
    c = ttgl.full([4, 32], 4, ttgl.float32, layout)

    libdevice.abs(a)
    libdevice.fast_dividef(a, b)
    libdevice.fma(a, b, c)

    libdevice.isnan(a)
    libdevice.isinf(a)
```
- **EN:** Prepares or updates state through `layout`, `a`, `b`, `c`. Invokes `ttgl.BlockedLayout`, `ttgl.full`, `libdevice.abs`, `libdevice.fast_dividef`, `libdevice.fma`, `libdevice.isnan`, and 1 more to execute the test logic. Relevant themes: layout transformation reasoning, libdevice coverage.
- **CN:** 通过 `layout`、`a`、`b`、`c` 准备或更新状态。 调用 `ttgl.BlockedLayout`、`ttgl.full`、`libdevice.abs`、`libdevice.fast_dividef`、`libdevice.fma`、`libdevice.isnan` 等另外 1 项 执行测试逻辑。 相关主题：布局变换推理、libdevice 覆盖。

### Lines 1210-1213

```python


@pytest.mark.parametrize("target", ALL_TARGETS)
def test_libdevice(target):
```
- **EN:** Defines the test function `test_libdevice`. Decorators: `pytest.mark.parametrize('target', ALL_TARGETS)`. Parameters: `target`. Key calls include `pytest.mark.parametrize`, `run_parser`, `expecttest.assert_expected_inline`, `anonymize_ir`, `mod.str_nodebug`. This scope touches pytest parametrization, debugging and inspection paths, libdevice coverage.
- **CN:** 定义测试函数 `test_libdevice`。 装饰器：`pytest.mark.parametrize('target', ALL_TARGETS)`。 参数：`target`。 关键调用包括 `pytest.mark.parametrize`、`run_parser`、`expecttest.assert_expected_inline`、`anonymize_ir`、`mod.str_nodebug`。 该作用域涉及pytest 参数化、调试与检查路径、libdevice 覆盖。

#### Lines 1214-1240

```python
    mod = run_parser(libdevice_kernel, target=target)
    expecttest.assert_expected_inline(
        anonymize_ir(mod.str_nodebug()), """\
#blocked = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [1, 32], warpsPerCTA = [4, 1], order = [1, 0]}>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "...", "ttg.threads-per-warp" = 32 : i32} {
  tt.func public @libdevice_kernel() attributes {noinline = false} {
    %cst = arith.constant 1.000000e+00 : f32
    %cst_0 = arith.constant dense<1.000000e+00> : tensor<4x32xf32, #blocked>
    %cst_1 = arith.constant 2.000000e+00 : f32
    %cst_2 = arith.constant dense<2.000000e+00> : tensor<4x32xf32, #blocked>
    %cst_3 = arith.constant 4.000000e+00 : f32
    %cst_4 = arith.constant dense<4.000000e+00> : tensor<4x32xf32, #blocked>
    %0 = tt.extern_elementwise %cst_0 {libname = "", libpath = "", pure = true, symbol = "..."} : (tensor<4x32xf32, #blocked>) -> tensor<4x32xf32, #blocked>
    %1 = tt.extern_elementwise %cst_0, %cst_2 {libname = "", libpath = "", pure = true, symbol = "..."} : (tensor<4x32xf32, #blocked>, tensor<4x32xf32, #blocked>) -> tensor<4x32xf32, #blocked>
    %2 = tt.extern_elementwise %cst_0, %cst_2, %cst_4 {libname = "", libpath = "", pure = true, symbol = "..."} : (tensor<4x32xf32, #blocked>, tensor<4x32xf32, #blocked>, tensor<4x32xf32, #blocked>) -> tensor<4x32xf32, #blocked>
    %3 = tt.extern_elementwise %cst_0 {libname = "", libpath = "", pure = true, symbol = "..."} : (tensor<4x32xf32, #blocked>) -> tensor<4x32xi32, #blocked>
    %c0_i32 = arith.constant 0 : i32
    %cst_5 = arith.constant dense<0> : tensor<4x32xi32, #blocked>
    %4 = arith.cmpi ne, %3, %cst_5 : tensor<4x32xi32, #blocked>
    %5 = tt.extern_elementwise %cst_0 {libname = "", libpath = "", pure = true, symbol = "..."} : (tensor<4x32xf32, #blocked>) -> tensor<4x32xi32, #blocked>
    %c0_i32_6 = arith.constant 0 : i32
    %cst_7 = arith.constant dense<0> : tensor<4x32xi32, #blocked>
    %6 = arith.cmpi ne, %5, %cst_7 : tensor<4x32xi32, #blocked>
    tt.return
  }
}
""")
```
- **EN:** Prepares or updates state through `mod`. Invokes `run_parser`, `expecttest.assert_expected_inline`, `anonymize_ir`, `mod.str_nodebug` to execute the test logic. Relevant themes: debugging and inspection paths, libdevice coverage.
- **CN:** 通过 `mod` 准备或更新状态。 调用 `run_parser`、`expecttest.assert_expected_inline`、`anonymize_ir`、`mod.str_nodebug` 执行测试逻辑。 相关主题：调试与检查路径、libdevice 覆盖。

### Lines 1241-1244

```python


@gluon.jit
def libdevice_implicit_broadcast_kernel():
```
- **EN:** Defines the helper function `libdevice_implicit_broadcast_kernel`. Decorators: `gluon.jit`. Key calls include `ttgl.BlockedLayout`, `ttgl.full`, `libdevice.abs`, `libdevice.fast_dividef`, `libdevice.fma`, `ttgl.SliceLayout`. This scope touches layout transformation reasoning, libdevice coverage.
- **CN:** 定义辅助函数 `libdevice_implicit_broadcast_kernel`。 装饰器：`gluon.jit`。 关键调用包括 `ttgl.BlockedLayout`、`ttgl.full`、`libdevice.abs`、`libdevice.fast_dividef`、`libdevice.fma`、`ttgl.SliceLayout`。 该作用域涉及布局变换推理、libdevice 覆盖。

#### Lines 1245-1251

```python
    layout: ttgl.constexpr = ttgl.BlockedLayout([1, 1], [1, 32], [4, 1], [1, 0])
    a = ttgl.full([4, 32], 1, ttgl.float32, layout)
    b = ttgl.full([32], 2, ttgl.float32, ttgl.SliceLayout(0, layout))[None, :]
    c = ttgl.full([4], 4, ttgl.float32, ttgl.SliceLayout(1, layout))[:, None]
    libdevice.abs(a)
    libdevice.fast_dividef(a, b)
    libdevice.fma(a, b, c)
```
- **EN:** Prepares or updates state through `layout`, `a`, `b`, `c`. Invokes `ttgl.BlockedLayout`, `ttgl.full`, `ttgl.SliceLayout`, `libdevice.abs`, `libdevice.fast_dividef`, `libdevice.fma` to execute the test logic. Relevant themes: layout transformation reasoning, libdevice coverage.
- **CN:** 通过 `layout`、`a`、`b`、`c` 准备或更新状态。 调用 `ttgl.BlockedLayout`、`ttgl.full`、`ttgl.SliceLayout`、`libdevice.abs`、`libdevice.fast_dividef`、`libdevice.fma` 执行测试逻辑。 相关主题：布局变换推理、libdevice 覆盖。

### Lines 1252-1255

```python


@pytest.mark.parametrize("target", ALL_TARGETS)
def test_libdevice_implicit_broadcast(target):
```
- **EN:** Defines the test function `test_libdevice_implicit_broadcast`. Decorators: `pytest.mark.parametrize('target', ALL_TARGETS)`. Parameters: `target`. Key calls include `pytest.mark.parametrize`, `run_parser`, `expecttest.assert_expected_inline`, `anonymize_ir`, `mod.str_nodebug`. This scope touches pytest parametrization, debugging and inspection paths, libdevice coverage.
- **CN:** 定义测试函数 `test_libdevice_implicit_broadcast`。 装饰器：`pytest.mark.parametrize('target', ALL_TARGETS)`。 参数：`target`。 关键调用包括 `pytest.mark.parametrize`、`run_parser`、`expecttest.assert_expected_inline`、`anonymize_ir`、`mod.str_nodebug`。 该作用域涉及pytest 参数化、调试与检查路径、libdevice 覆盖。

#### Lines 1256-1282

```python
    mod = run_parser(libdevice_implicit_broadcast_kernel, target=target)
    expecttest.assert_expected_inline(
        anonymize_ir(mod.str_nodebug()), """\
#blocked = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [1, 32], warpsPerCTA = [4, 1], order = [1, 0]}>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "...", "ttg.threads-per-warp" = 32 : i32} {
  tt.func public @libdevice_implicit_broadcast_kernel() attributes {noinline = false} {
    %cst = arith.constant 1.000000e+00 : f32
    %cst_0 = arith.constant dense<1.000000e+00> : tensor<4x32xf32, #blocked>
    %cst_1 = arith.constant 2.000000e+00 : f32
    %cst_2 = arith.constant dense<2.000000e+00> : tensor<32xf32, #ttg.slice<{dim = 0, parent = #blocked}>>
    %0 = tt.expand_dims %cst_2 {axis = 0 : i32} : tensor<32xf32, #ttg.slice<{dim = 0, parent = #blocked}>> -> tensor<1x32xf32, #blocked>
    %cst_3 = arith.constant 4.000000e+00 : f32
    %cst_4 = arith.constant dense<4.000000e+00> : tensor<4xf32, #ttg.slice<{dim = 1, parent = #blocked}>>
    %1 = tt.expand_dims %cst_4 {axis = 1 : i32} : tensor<4xf32, #ttg.slice<{dim = 1, parent = #blocked}>> -> tensor<4x1xf32, #blocked>
    %2 = tt.extern_elementwise %cst_0 {libname = "", libpath = "", pure = true, symbol = "..."} : (tensor<4x32xf32, #blocked>) -> tensor<4x32xf32, #blocked>
    %3 = tt.broadcast %0 : tensor<1x32xf32, #blocked> -> tensor<4x32xf32, #blocked>
    %4 = tt.broadcast %0 : tensor<1x32xf32, #blocked> -> tensor<4x32xf32, #blocked>
    %5 = tt.extern_elementwise %cst_0, %4 {libname = "", libpath = "", pure = true, symbol = "..."} : (tensor<4x32xf32, #blocked>, tensor<4x32xf32, #blocked>) -> tensor<4x32xf32, #blocked>
    %6 = tt.broadcast %0 : tensor<1x32xf32, #blocked> -> tensor<4x32xf32, #blocked>
    %7 = tt.broadcast %1 : tensor<4x1xf32, #blocked> -> tensor<4x32xf32, #blocked>
    %8 = tt.broadcast %0 : tensor<1x32xf32, #blocked> -> tensor<4x32xf32, #blocked>
    %9 = tt.broadcast %1 : tensor<4x1xf32, #blocked> -> tensor<4x32xf32, #blocked>
    %10 = tt.extern_elementwise %cst_0, %8, %9 {libname = "", libpath = "", pure = true, symbol = "..."} : (tensor<4x32xf32, #blocked>, tensor<4x32xf32, #blocked>, tensor<4x32xf32, #blocked>) -> tensor<4x32xf32, #blocked>
    tt.return
  }
}
""")
```
- **EN:** Prepares or updates state through `mod`. Invokes `run_parser`, `expecttest.assert_expected_inline`, `anonymize_ir`, `mod.str_nodebug` to execute the test logic. Relevant themes: debugging and inspection paths, libdevice coverage.
- **CN:** 通过 `mod` 准备或更新状态。 调用 `run_parser`、`expecttest.assert_expected_inline`、`anonymize_ir`、`mod.str_nodebug` 执行测试逻辑。 相关主题：调试与检查路径、libdevice 覆盖。

### Lines 1283-1286

```python


@gluon.jit
def pair_add(a0, a1, b0, b1):
```
- **EN:** Defines the helper function `pair_add`. Decorators: `gluon.jit`. Parameters: `a0`, `a1`, `b0`, `b1`.
- **CN:** 定义辅助函数 `pair_add`。 装饰器：`gluon.jit`。 参数：`a0`、`a1`、`b0`、`b1`。

#### Lines 1287-1287

```python
    return a0 + b0, a1 + b1
```
- **EN:** Carries supporting logic for the surrounding test or helper scope.
- **CN:** 为周围的测试或辅助作用域提供支撑逻辑。

### Lines 1288-1291

```python


@gluon.jit
def reduce_kernel(out):
```
- **EN:** Defines the helper function `reduce_kernel`. Decorators: `gluon.jit`. Parameters: `out`. Key calls include `ttgl.BlockedLayout`, `ttgl.full`, `a.sum`, `ttgl.static_assert`, `ttgl.sum`, `ttgl.max`, and 5 more. This scope touches layout transformation reasoning.
- **CN:** 定义辅助函数 `reduce_kernel`。 装饰器：`gluon.jit`。 参数：`out`。 关键调用包括 `ttgl.BlockedLayout`、`ttgl.full`、`a.sum`、`ttgl.static_assert`、`ttgl.sum`、`ttgl.max` 等另外 5 项。 该作用域涉及布局变换推理。

#### Lines 1292-1311

```python
    layout: ttgl.constexpr = ttgl.BlockedLayout([1, 1], [1, 32], [4, 1], [1, 0])
    a = ttgl.full([16, 16], 1, ttgl.float32, layout)
    b = ttgl.full([16, 16], 2, ttgl.float32, layout)
    s0 = a.sum(0)
    ttgl.static_assert(s0.type.layout == ttgl.SliceLayout(0, layout))
    s1 = ttgl.sum(a, 1)
    ttgl.static_assert(s1.type.layout == ttgl.SliceLayout(1, layout))
    s2 = ttgl.sum(a)
    ttgl.static_assert(s2.shape == [])

    scalar = ttgl.max(s0, 0)
    ttgl.static_assert(scalar.type == ttgl.float32)

    s1 = ttgl.convert_layout(s1, s0.type.layout)

    pairs = ttgl.reduce((a, b), 0, pair_add)
    ttgl.static_assert(pairs[0].type.layout == ttgl.SliceLayout(0, layout))
    ttgl.static_assert(pairs[1].type.layout == ttgl.SliceLayout(0, layout))
    result = scalar + s1 + pairs[0] + pairs[1]
    ttgl.store(out + ttgl.arange(0, 16, s0.type.layout), result)
```
- **EN:** Prepares or updates state through `layout`, `a`, `b`, `s0`, `s1`, `s2`, `scalar`, `pairs`, and 1 more. Invokes `ttgl.BlockedLayout`, `ttgl.full`, `a.sum`, `ttgl.static_assert`, `ttgl.SliceLayout`, `ttgl.sum`, and 5 more to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `layout`、`a`、`b`、`s0`、`s1`、`s2`、`scalar`、`pairs` 等另外 1 项 准备或更新状态。 调用 `ttgl.BlockedLayout`、`ttgl.full`、`a.sum`、`ttgl.static_assert`、`ttgl.SliceLayout`、`ttgl.sum` 等另外 5 项 执行测试逻辑。 相关主题：布局变换推理。

### Lines 1312-1315

```python


@pytest.mark.parametrize("target", ALL_TARGETS)
def test_reduce(target):
```
- **EN:** Defines the test function `test_reduce`. Decorators: `pytest.mark.parametrize('target', ALL_TARGETS)`. Parameters: `target`. Key calls include `pytest.mark.parametrize`, `run_parser`, `expecttest.assert_expected_inline`, `anonymize_ir`, `make_args`, `mod.str_nodebug`, and 1 more. This scope touches pytest parametrization, Triton language operations, debugging and inspection paths, layout transformation reasoning.
- **CN:** 定义测试函数 `test_reduce`。 装饰器：`pytest.mark.parametrize('target', ALL_TARGETS)`。 参数：`target`。 关键调用包括 `pytest.mark.parametrize`、`run_parser`、`expecttest.assert_expected_inline`、`anonymize_ir`、`make_args`、`mod.str_nodebug` 等另外 1 项。 该作用域涉及pytest 参数化、Triton language 操作、调试与检查路径、布局变换推理。

#### Lines 1316-1416

```python
    mod = run_parser(reduce_kernel, *make_args(MockTensor(ttgl.float32)), target=target)
    expecttest.assert_expected_inline(
        anonymize_ir(mod.str_nodebug()), """\
#blocked = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [1, 32], warpsPerCTA = [4, 1], order = [1, 0]}>
#linear = #ttg.linear<{register = [[64], [128]], lane = [[1], [2], [4], [8], [0]], warp = [[16], [32]], block = []}>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "...", "ttg.threads-per-warp" = 32 : i32} {
  tt.func public @reduce_kernel(%arg0: !tt.ptr<f32> {tt.divisibility = 16 : i32}) attributes {noinline = false} {
    %cst = arith.constant 1.000000e+00 : f32
    %cst_0 = arith.constant dense<1.000000e+00> : tensor<16x16xf32, #blocked>
    %cst_1 = arith.constant 2.000000e+00 : f32
    %cst_2 = arith.constant dense<2.000000e+00> : tensor<16x16xf32, #blocked>
    %0 = tt.call @triton.language.standard.sum__fp32S16_16SLB1_1_1_32_4_1_1_0_BL_c0_cFalse_cNone(%cst_0) : (tensor<16x16xf32, #blocked>) -> tensor<16xf32, #ttg.slice<{dim = 0, parent = #blocked}>>
    %1 = tt.call @triton.language.standard.sum__fp32S16_16SLB1_1_1_32_4_1_1_0_BL_c1_cFalse_cNone(%cst_0) : (tensor<16x16xf32, #blocked>) -> tensor<16xf32, #ttg.slice<{dim = 1, parent = #blocked}>>
    %2 = tt.call @triton.language.standard.sum__fp32S16_16SLB1_1_1_32_4_1_1_0_BL_cNone_cFalse_cNone(%cst_0) : (tensor<16x16xf32, #blocked>) -> f32
    %3 = tt.call @triton.language.standard.max__fp32S16SLSL0_B1_1_1_32_4_1_1_0_BSLL_c0_cFalse_cTrue_cFalse(%0) : (tensor<16xf32, #ttg.slice<{dim = 0, parent = #blocked}>>) -> f32
    %4 = ttg.convert_layout %1 : tensor<16xf32, #ttg.slice<{dim = 1, parent = #blocked}>> -> tensor<16xf32, #ttg.slice<{dim = 0, parent = #blocked}>>
    %5:2 = "tt.reduce"(%cst_0, %cst_2) <{axis = 0 : i32}> ({
    ^bb0(%arg1: f32, %arg2: f32, %arg3: f32, %arg4: f32):
      %13:2 = tt.call @test_frontend.pair_add__fp32_fp32_fp32_fp32(%arg1, %arg2, %arg3, %arg4) : (f32, f32, f32, f32) -> (f32, f32)
      tt.reduce.return %13#0, %13#1 : f32, f32
    }) : (tensor<16x16xf32, #blocked>, tensor<16x16xf32, #blocked>) -> (tensor<16xf32, #ttg.slice<{dim = 0, parent = #blocked}>>, tensor<16xf32, #ttg.slice<{dim = 0, parent = #blocked}>>)
    %6 = tt.splat %3 : f32 -> tensor<16xf32, #ttg.slice<{dim = 0, parent = #blocked}>>
    %7 = arith.addf %6, %4 : tensor<16xf32, #ttg.slice<{dim = 0, parent = #blocked}>>
    %8 = arith.addf %7, %5#0 : tensor<16xf32, #ttg.slice<{dim = 0, parent = #blocked}>>
    %9 = arith.addf %8, %5#1 : tensor<16xf32, #ttg.slice<{dim = 0, parent = #blocked}>>
    %10 = tt.make_range {end = 16 : i32, start = 0 : i32} : tensor<16xi32, #ttg.slice<{dim = 0, parent = #blocked}>>
    %11 = tt.splat %arg0 : !tt.ptr<f32> -> tensor<16x!tt.ptr<f32>, #ttg.slice<{dim = 0, parent = #blocked}>>
    %12 = tt.addptr %11, %10 : tensor<16x!tt.ptr<f32>, #ttg.slice<{dim = 0, parent = #blocked}>>, tensor<16xi32, #ttg.slice<{dim = 0, parent = #blocked}>>
    tt.store %12, %9 : tensor<16x!tt.ptr<f32>, #ttg.slice<{dim = 0, parent = #blocked}>>
    tt.return
  }
  tt.func private @triton.language.standard.sum__fp32S16_16SLB1_1_1_32_4_1_1_0_BL_c0_cFalse_cNone(%arg0: tensor<16x16xf32, #blocked>) -> tensor<16xf32, #ttg.slice<{dim = 0, parent = #blocked}>> attributes {noinline = false} {
    %0 = "tt.reduce"(%arg0) <{axis = 0 : i32}> ({
    ^bb0(%arg1: f32, %arg2: f32):
      %2 = tt.call @triton.language.standard._sum_combine__fp32_fp32(%arg1, %arg2) : (f32, f32) -> f32
      tt.reduce.return %2 : f32
    }) : (tensor<16x16xf32, #blocked>) -> tensor<16xf32, #ttg.slice<{dim = 0, parent = #blocked}>>
    tt.return %0 : tensor<16xf32, #ttg.slice<{dim = 0, parent = #blocked}>>
  ^bb1:  // no predecessors
    %1 = ub.poison : tensor<16xf32, #ttg.slice<{dim = 0, parent = #blocked}>>
    tt.return %1 : tensor<16xf32, #ttg.slice<{dim = 0, parent = #blocked}>>
  }
  tt.func private @triton.language.standard._sum_combine__fp32_fp32(%arg0: f32, %arg1: f32) -> f32 attributes {noinline = false} {
    %0 = arith.addf %arg0, %arg1 : f32
    tt.return %0 : f32
  ^bb1:  // no predecessors
    %1 = ub.poison : f32
    tt.return %1 : f32
  }
  tt.func private @triton.language.standard.sum__fp32S16_16SLB1_1_1_32_4_1_1_0_BL_c1_cFalse_cNone(%arg0: tensor<16x16xf32, #blocked>) -> tensor<16xf32, #ttg.slice<{dim = 1, parent = #blocked}>> attributes {noinline = false} {
    %0 = "tt.reduce"(%arg0) <{axis = 1 : i32}> ({
    ^bb0(%arg1: f32, %arg2: f32):
      %2 = tt.call @triton.language.standard._sum_combine__fp32_fp32(%arg1, %arg2) : (f32, f32) -> f32
      tt.reduce.return %2 : f32
    }) : (tensor<16x16xf32, #blocked>) -> tensor<16xf32, #ttg.slice<{dim = 1, parent = #blocked}>>
    tt.return %0 : tensor<16xf32, #ttg.slice<{dim = 1, parent = #blocked}>>
  ^bb1:  // no predecessors
    %1 = ub.poison : tensor<16xf32, #ttg.slice<{dim = 1, parent = #blocked}>>
    tt.return %1 : tensor<16xf32, #ttg.slice<{dim = 1, parent = #blocked}>>
  }
  tt.func private @triton.language.standard.sum__fp32S16_16SLB1_1_1_32_4_1_1_0_BL_cNone_cFalse_cNone(%arg0: tensor<16x16xf32, #blocked>) -> f32 attributes {noinline = false} {
    %0 = tt.reshape %arg0 : tensor<16x16xf32, #blocked> -> tensor<256xf32, #linear>
    %1 = "tt.reduce"(%0) <{axis = 0 : i32}> ({
    ^bb0(%arg1: f32, %arg2: f32):
      %3 = tt.call @triton.language.standard._sum_combine__fp32_fp32(%arg1, %arg2) : (f32, f32) -> f32
      tt.reduce.return %3 : f32
    }) : (tensor<256xf32, #linear>) -> f32
    tt.return %1 : f32
  ^bb1:  // no predecessors
    %2 = ub.poison : f32
    tt.return %2 : f32
  }
  tt.func private @triton.language.standard.max__fp32S16SLSL0_B1_1_1_32_4_1_1_0_BSLL_c0_cFalse_cTrue_cFalse(%arg0: tensor<16xf32, #ttg.slice<{dim = 0, parent = #blocked}>>) -> f32 attributes {noinline = false} {
    %0 = "tt.reduce"(%arg0) <{axis = 0 : i32}> ({
    ^bb0(%arg1: f32, %arg2: f32):
      %2 = tt.call @triton.language.standard._elementwise_max__fp32_fp32(%arg1, %arg2) : (f32, f32) -> f32
      tt.reduce.return %2 : f32
    }) : (tensor<16xf32, #ttg.slice<{dim = 0, parent = #blocked}>>) -> f32
    tt.return %0 : f32
  ^bb1:  // no predecessors
    %1 = ub.poison : f32
    tt.return %1 : f32
  }
  tt.func private @triton.language.standard._elementwise_max__fp32_fp32(%arg0: f32, %arg1: f32) -> f32 attributes {noinline = false} {
    %0 = arith.maxnumf %arg0, %arg1 : f32
    tt.return %0 : f32
  ^bb1:  // no predecessors
    %1 = ub.poison : f32
    tt.return %1 : f32
  }
  tt.func private @test_frontend.pair_add__fp32_fp32_fp32_fp32(%arg0: f32, %arg1: f32, %arg2: f32, %arg3: f32) -> (f32, f32) attributes {noinline = false} {
    %0 = arith.addf %arg0, %arg2 : f32
    %1 = arith.addf %arg1, %arg3 : f32
    tt.return %0, %1 : f32, f32
  ^bb1:  // no predecessors
    %2 = ub.poison : f32
    %3 = ub.poison : f32
    tt.return %2, %3 : f32, f32
  }
}
""")
```
- **EN:** Prepares or updates state through `mod`. Invokes `run_parser`, `make_args`, `MockTensor`, `expecttest.assert_expected_inline`, `anonymize_ir`, `mod.str_nodebug` to execute the test logic. Relevant themes: Triton language operations, debugging and inspection paths, layout transformation reasoning.
- **CN:** 通过 `mod` 准备或更新状态。 调用 `run_parser`、`make_args`、`MockTensor`、`expecttest.assert_expected_inline`、`anonymize_ir`、`mod.str_nodebug` 执行测试逻辑。 相关主题：Triton language 操作、调试与检查路径、布局变换推理。

### Lines 1417-1423

```python


@filecheck_test
@gluon.jit
def test_elementwise_core():
    # CHECK: [[BLOCKED:#.*]] = #ttg.blocked<{sizePerThread = [1], threadsPerWarp = [32], warpsPerCTA = [4], order = [0]}>
    # CHECK: @test_elementwise_core
```
- **EN:** Defines the test function `test_elementwise_core`. Decorators: `filecheck_test`, `gluon.jit`. Key calls include `ttgl.BlockedLayout`, `ttgl.arange`, `ttgl.where`, `ttgl.maximum`, `ttgl.minimum`, `ttgl.static_assert`. This scope touches layout transformation reasoning.
- **CN:** 定义测试函数 `test_elementwise_core`。 装饰器：`filecheck_test`、`gluon.jit`。 关键调用包括 `ttgl.BlockedLayout`、`ttgl.arange`、`ttgl.where`、`ttgl.maximum`、`ttgl.minimum`、`ttgl.static_assert`。 该作用域涉及布局变换推理。

#### Lines 1424-1436

```python
    layout: ttgl.constexpr = ttgl.BlockedLayout([1], [32], [4], [0])
    x = ttgl.arange(0, 16, layout)
    y = ttgl.arange(16, 32, layout)

    # CHECK: arith.select {{.*}} : tensor<16xi1, [[BLOCKED]]>, tensor<16xi32, [[BLOCKED]]>
    a = ttgl.where(x > 8, x, y)
    # CHECK: arith.maxsi {{.*}} : tensor<16xi32, [[BLOCKED]]>
    b = ttgl.maximum(x, y)
    # CHECK: arith.minsi {{.*}} : tensor<16xi32, [[BLOCKED]]>
    c = ttgl.minimum(x, y)
    ttgl.static_assert(a.type == x.type)
    ttgl.static_assert(b.type == x.type)
    ttgl.static_assert(c.type == x.type)
```
- **EN:** Prepares or updates state through `layout`, `x`, `y`, `a`, `b`, `c`. Invokes `ttgl.BlockedLayout`, `ttgl.arange`, `ttgl.where`, `ttgl.maximum`, `ttgl.minimum`, `ttgl.static_assert` to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `layout`、`x`、`y`、`a`、`b`、`c` 准备或更新状态。 调用 `ttgl.BlockedLayout`、`ttgl.arange`、`ttgl.where`、`ttgl.maximum`、`ttgl.minimum`、`ttgl.static_assert` 执行测试逻辑。 相关主题：布局变换推理。

### Lines 1437-1440

```python


@gluon.jit
def linear_layout_kernel():
```
- **EN:** Defines the helper function `linear_layout_kernel`. Decorators: `gluon.jit`. Key calls include `ttgl.DistributedLinearLayout`, `ttgl.arange`. This scope touches layout transformation reasoning.
- **CN:** 定义辅助函数 `linear_layout_kernel`。 装饰器：`gluon.jit`。 关键调用包括 `ttgl.DistributedLinearLayout`、`ttgl.arange`。 该作用域涉及布局变换推理。

#### Lines 1441-1443

```python
    ll: ttgl.constexpr = ttgl.DistributedLinearLayout(reg_bases=[[1]], lane_bases=[[2], [4], [8], [16], [32]],
                                                      warp_bases=[[64], [128]], block_bases=[], shape=[256])
    ttgl.arange(0, 256, layout=ll)
```
- **EN:** Prepares or updates state through `ll`. Invokes `ttgl.DistributedLinearLayout`, `ttgl.arange` to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `ll` 准备或更新状态。 调用 `ttgl.DistributedLinearLayout`、`ttgl.arange` 执行测试逻辑。 相关主题：布局变换推理。

### Lines 1444-1447

```python


@pytest.mark.parametrize("target", ALL_TARGETS)
def test_linear_layout(target):
```
- **EN:** Defines the test function `test_linear_layout`. Decorators: `pytest.mark.parametrize('target', ALL_TARGETS)`. Parameters: `target`. Key calls include `pytest.mark.parametrize`, `run_parser`, `expecttest.assert_expected_inline`, `anonymize_ir`, `mod.str_nodebug`. This scope touches pytest parametrization, debugging and inspection paths, layout transformation reasoning.
- **CN:** 定义测试函数 `test_linear_layout`。 装饰器：`pytest.mark.parametrize('target', ALL_TARGETS)`。 参数：`target`。 关键调用包括 `pytest.mark.parametrize`、`run_parser`、`expecttest.assert_expected_inline`、`anonymize_ir`、`mod.str_nodebug`。 该作用域涉及pytest 参数化、调试与检查路径、布局变换推理。

#### Lines 1448-1458

```python
    mod = run_parser(linear_layout_kernel, target=target)
    expecttest.assert_expected_inline(
        anonymize_ir(mod.str_nodebug()), """\
#linear = #ttg.linear<{register = [[1]], lane = [[2], [4], [8], [16], [32]], warp = [[64], [128]], block = []}>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "...", "ttg.threads-per-warp" = 32 : i32} {
  tt.func public @linear_layout_kernel() attributes {noinline = false} {
    %0 = tt.make_range {end = 256 : i32, start = 0 : i32} : tensor<256xi32, #linear>
    tt.return
  }
}
""")
```
- **EN:** Prepares or updates state through `mod`. Invokes `run_parser`, `expecttest.assert_expected_inline`, `anonymize_ir`, `mod.str_nodebug` to execute the test logic. Relevant themes: debugging and inspection paths, layout transformation reasoning.
- **CN:** 通过 `mod` 准备或更新状态。 调用 `run_parser`、`expecttest.assert_expected_inline`、`anonymize_ir`、`mod.str_nodebug` 执行测试逻辑。 相关主题：调试与检查路径、布局变换推理。

### Lines 1459-1465

```python


@filecheck_test
@gluon.jit
def test_dot_operand_layout():
    # CHECK: [[NVMMA:#.*]] = #ttg.nvidia_mma
    # CHECK: test_dot_operand_layout
```
- **EN:** Defines the test function `test_dot_operand_layout`. Decorators: `filecheck_test`, `gluon.jit`. Key calls include `ttgl.NVMMADistributedLayout`, `ttgl.DotOperandLayout`, `ttgl.full`, `x.sum`, `ttgl.static_assert`. This scope touches layout transformation reasoning, random-data generation.
- **CN:** 定义测试函数 `test_dot_operand_layout`。 装饰器：`filecheck_test`、`gluon.jit`。 关键调用包括 `ttgl.NVMMADistributedLayout`、`ttgl.DotOperandLayout`、`ttgl.full`、`x.sum`、`ttgl.static_assert`。 该作用域涉及布局变换推理、随机数据生成。

#### Lines 1466-1472

```python
    mma_layout: ttgl.constexpr = ttgl.NVMMADistributedLayout(version=[3, 0], warps_per_cta=[4, 1],
                                                             instr_shape=[16, 32, 16])
    layout: ttgl.constexpr = ttgl.DotOperandLayout(operand_index=0, parent=mma_layout, k_width=2)
    # CHECK: arith.constant {{.*}} tensor<256x128xf16, #ttg.dot_op<{opIdx = 0, parent = [[NVMMA]], kWidth = 2}>>
    x = ttgl.full([256, 128], 0.0, ttgl.float16, layout)
    y = x.sum(axis=1)
    ttgl.static_assert(y.type.layout.parent == layout)
```
- **EN:** Prepares or updates state through `mma_layout`, `layout`, `x`, `y`. Invokes `ttgl.NVMMADistributedLayout`, `ttgl.DotOperandLayout`, `ttgl.full`, `x.sum`, `ttgl.static_assert` to execute the test logic. Relevant themes: layout transformation reasoning, random-data generation.
- **CN:** 通过 `mma_layout`、`layout`、`x`、`y` 准备或更新状态。 调用 `ttgl.NVMMADistributedLayout`、`ttgl.DotOperandLayout`、`ttgl.full`、`x.sum`、`ttgl.static_assert` 执行测试逻辑。 相关主题：布局变换推理、随机数据生成。

### Lines 1473-1479

```python


@filecheck_test
@gluon.jit
def test_tensor_permute():
    # CHECK-DAG: [[BLOCKED:#.*]] = #ttg.blocked<{sizePerThread = [1, 2], threadsPerWarp = [4, 8], warpsPerCTA = [4, 1], order = [1, 0]}>
    # CHECK-DAG: [[BLOCKED1:#.*]] = #ttg.blocked<{sizePerThread = [2, 1], threadsPerWarp = [8, 4], warpsPerCTA = [1, 4], order = [0, 1]}>
```
- **EN:** Defines the test function `test_tensor_permute`. Decorators: `filecheck_test`, `gluon.jit`. Key calls include `ttgl.BlockedLayout`, `ttgl.full`, `ttgl.permute`, `ttgl.static_assert`. This scope touches layout transformation reasoning.
- **CN:** 定义测试函数 `test_tensor_permute`。 装饰器：`filecheck_test`、`gluon.jit`。 关键调用包括 `ttgl.BlockedLayout`、`ttgl.full`、`ttgl.permute`、`ttgl.static_assert`。 该作用域涉及布局变换推理。

#### Lines 1480-1485

```python
    layout: ttgl.constexpr = ttgl.BlockedLayout([1, 2], [4, 8], [4, 1], [1, 0])
    a = ttgl.full([32, 16], 0, ttgl.int32, layout=layout)
    # CHECK: tt.trans{{.*}} : tensor<32x16xi32, [[BLOCKED]]> -> tensor<16x32xi32, [[BLOCKED1]]>
    res = ttgl.permute(a, [1, 0])
    permuted_layout: ttgl.constexpr = ttgl.BlockedLayout([2, 1], [8, 4], [1, 4], [0, 1])
    ttgl.static_assert(permuted_layout == res.type.layout)
```
- **EN:** Prepares or updates state through `layout`, `a`, `res`, `permuted_layout`. Invokes `ttgl.BlockedLayout`, `ttgl.full`, `ttgl.permute`, `ttgl.static_assert` to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `layout`、`a`、`res`、`permuted_layout` 准备或更新状态。 调用 `ttgl.BlockedLayout`、`ttgl.full`、`ttgl.permute`、`ttgl.static_assert` 执行测试逻辑。 相关主题：布局变换推理。

### Lines 1486-1492

```python


@filecheck_test
@gluon.jit
def test_split_join():
    # CHECK: [[BLOCKED:#.*]] = #ttg.blocked<{sizePerThread = [2], threadsPerWarp = [32], warpsPerCTA = [4], order = [0]}>
    # CHECK: [[BLOCKED1:#.*]] = #ttg.blocked<{sizePerThread = [2, 2], threadsPerWarp = [32, 1], warpsPerCTA = [4, 1], order = [1, 0]}>
```
- **EN:** Defines the test function `test_split_join`. Decorators: `filecheck_test`, `gluon.jit`. Key calls include `ttgl.BlockedLayout`, `ttgl.full`, `ttgl.join`, `ttgl.static_assert`, `ttgl.split`, `ttgl.SliceLayout`. This scope touches layout transformation reasoning.
- **CN:** 定义测试函数 `test_split_join`。 装饰器：`filecheck_test`、`gluon.jit`。 关键调用包括 `ttgl.BlockedLayout`、`ttgl.full`、`ttgl.join`、`ttgl.static_assert`、`ttgl.split`、`ttgl.SliceLayout`。 该作用域涉及布局变换推理。

#### Lines 1493-1504

```python
    layout: ttgl.constexpr = ttgl.BlockedLayout([2], [32], [4], [0])
    a = ttgl.full([128], 1, ttgl.int32, layout)
    b = ttgl.full([128], 2, ttgl.int32, layout)
    # CHECK: tt.join {{.*}} : tensor<128xi32, [[BLOCKED]]> -> tensor<128x2xi32, [[BLOCKED1]]>
    res = ttgl.join(a, b)
    expect_layout: ttgl.constexpr = ttgl.BlockedLayout([2, 2], [32, 1], [4, 1], [1, 0])
    ttgl.static_assert(res.type.layout == expect_layout)

    # CHECK: tt.split {{.*}} : tensor<128x2xi32, [[BLOCKED1]]> -> tensor<128xi32, #ttg.slice<{dim = 1, parent = [[BLOCKED1]]}>>
    c, d = ttgl.split(res)
    ttgl.static_assert(c.type.layout == ttgl.SliceLayout(1, expect_layout))
    ttgl.static_assert(d.type.layout == ttgl.SliceLayout(1, expect_layout))
```
- **EN:** Prepares or updates state through `layout`, `a`, `b`, `res`, `expect_layout`, `c`, `d`. Invokes `ttgl.BlockedLayout`, `ttgl.full`, `ttgl.join`, `ttgl.static_assert`, `ttgl.split`, `ttgl.SliceLayout` to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `layout`、`a`、`b`、`res`、`expect_layout`、`c`、`d` 准备或更新状态。 调用 `ttgl.BlockedLayout`、`ttgl.full`、`ttgl.join`、`ttgl.static_assert`、`ttgl.split`、`ttgl.SliceLayout` 执行测试逻辑。 相关主题：布局变换推理。

### Lines 1505-1512

```python


@filecheck_test
@gluon.jit
def test_split_auto_layout():
    # CHECK-LABEL: test_split_auto_layout
    # CHECK: %[[X:.+]] = arith.constant dense<1> : tensor<128x2xi32, #gluon.auto_encoding>
    # CHECK: %[[LHS:.+]], %[[RHS:.+]] = tt.split %[[X]] : tensor<128x2xi32, #gluon.auto_encoding> -> tensor<128xi32, #gluon.auto_encoding>
```
- **EN:** Defines the test function `test_split_auto_layout`. Decorators: `filecheck_test`, `gluon.jit`. Key calls include `ttgl.full`, `ttgl.split`, `ttgl.static_assert`, `ttgl.AutoLayout`. This scope touches layout transformation reasoning.
- **CN:** 定义测试函数 `test_split_auto_layout`。 装饰器：`filecheck_test`、`gluon.jit`。 关键调用包括 `ttgl.full`、`ttgl.split`、`ttgl.static_assert`、`ttgl.AutoLayout`。 该作用域涉及布局变换推理。

#### Lines 1513-1516

```python
    x = ttgl.full([128, 2], 1, ttgl.int32, layout=ttgl.AutoLayout())
    lhs, rhs = ttgl.split(x)
    ttgl.static_assert(lhs.type.layout == ttgl.AutoLayout())
    ttgl.static_assert(rhs.type.layout == ttgl.AutoLayout())
```
- **EN:** Prepares or updates state through `x`, `lhs`, `rhs`. Invokes `ttgl.full`, `ttgl.AutoLayout`, `ttgl.split`, `ttgl.static_assert` to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `x`、`lhs`、`rhs` 准备或更新状态。 调用 `ttgl.full`、`ttgl.AutoLayout`、`ttgl.split`、`ttgl.static_assert` 执行测试逻辑。 相关主题：布局变换推理。

### Lines 1517-1523

```python


@filecheck_test
@gluon.jit
def test_reshape_linear_layout():
    # CHECK: [[BLOCKED:#.*]] = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [32, 1], warpsPerCTA = [4, 1], order = [0, 1]}>
    # CHECK: [[LINEAR:#.*]] = #ttg.linear
```
- **EN:** Defines the test function `test_reshape_linear_layout`. Decorators: `filecheck_test`, `gluon.jit`. Key calls include `ttgl.BlockedLayout`, `ttgl.full`, `x.reshape`. This scope touches layout transformation reasoning.
- **CN:** 定义测试函数 `test_reshape_linear_layout`。 装饰器：`filecheck_test`、`gluon.jit`。 关键调用包括 `ttgl.BlockedLayout`、`ttgl.full`、`x.reshape`。 该作用域涉及布局变换推理。

#### Lines 1524-1527

```python
    layout: ttgl.constexpr = ttgl.BlockedLayout([1, 1], [32, 1], [4, 1], [0, 1])
    x = ttgl.full([128, 1], 1, ttgl.int32, layout=layout)
    # CHECK: tt.reshape %{{.*}} : tensor<128x1xi32, [[BLOCKED]]> -> tensor<128xi32, [[LINEAR]]>
    x.reshape([128])
```
- **EN:** Prepares or updates state through `layout`, `x`. Invokes `ttgl.BlockedLayout`, `ttgl.full`, `x.reshape` to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `layout`、`x` 准备或更新状态。 调用 `ttgl.BlockedLayout`、`ttgl.full`、`x.reshape` 执行测试逻辑。 相关主题：布局变换推理。

### Lines 1528-1534

```python


@filecheck_test
@gluon.jit
def test_tensor_reshape():
    # CHECK: [[BLOCKED:#.*]] = #ttg.blocked<{sizePerThread = [2], threadsPerWarp = [32], warpsPerCTA = [4], order = [0]}>
    # CHECK: [[BLOCKED1:#.*]] = #ttg.blocked<{sizePerThread = [1, 1, 2], threadsPerWarp = [2, 4, 4], warpsPerCTA = [4, 1, 1], order = [2, 1, 0]}>
```
- **EN:** Defines the test function `test_tensor_reshape`. Decorators: `filecheck_test`, `gluon.jit`. Key calls include `ttgl.BlockedLayout`, `ttgl.full`, `a.reshape`, `ttgl.static_assert`. This scope touches layout transformation reasoning.
- **CN:** 定义测试函数 `test_tensor_reshape`。 装饰器：`filecheck_test`、`gluon.jit`。 关键调用包括 `ttgl.BlockedLayout`、`ttgl.full`、`a.reshape`、`ttgl.static_assert`。 该作用域涉及布局变换推理。

#### Lines 1535-1540

```python
    layout: ttgl.constexpr = ttgl.BlockedLayout([2], [32], [4], [0])
    a = ttgl.full([256], 1, ttgl.int32, layout)
    # CHECK: tt.reshape {{.*}} : tensor<256xi32, [[BLOCKED]]> -> tensor<8x4x8xi32, [[BLOCKED1]]>
    v = a.reshape([8, 4, 8])
    expect_layout: ttgl.constexpr = ttgl.BlockedLayout([1, 1, 2], [2, 4, 4], [4, 1, 1], [2, 1, 0])
    ttgl.static_assert(v.type.layout == expect_layout)
```
- **EN:** Prepares or updates state through `layout`, `a`, `v`, `expect_layout`. Invokes `ttgl.BlockedLayout`, `ttgl.full`, `a.reshape`, `ttgl.static_assert` to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `layout`、`a`、`v`、`expect_layout` 准备或更新状态。 调用 `ttgl.BlockedLayout`、`ttgl.full`、`a.reshape`、`ttgl.static_assert` 执行测试逻辑。 相关主题：布局变换推理。

### Lines 1541-1544

```python


@gluon.jit
def static_assert_kernel():
```
- **EN:** Defines the helper function `static_assert_kernel`. Decorators: `gluon.jit`. Key calls include `ttgl.static_assert`.
- **CN:** 定义辅助函数 `static_assert_kernel`。 装饰器：`gluon.jit`。 关键调用包括 `ttgl.static_assert`。

#### Lines 1545-1545

```python
    ttgl.static_assert(False)
```
- **EN:** Invokes `ttgl.static_assert` to execute the test logic.
- **CN:** 调用 `ttgl.static_assert` 执行测试逻辑。

### Lines 1546-1548

```python


def test_static_assert():
```
- **EN:** Defines the test function `test_static_assert`. Key calls include `pytest.raises`, `run_parser`.
- **CN:** 定义测试函数 `test_static_assert`。 关键调用包括 `pytest.raises`、`run_parser`。

#### Lines 1549-1550

```python
    with pytest.raises(CompileTimeAssertionFailure):
        run_parser(static_assert_kernel)
```
- **EN:** Invokes `pytest.raises`, `run_parser` to execute the test logic. Uses `pytest.raises` to confirm expected failure paths. Uses context managers to control resources or expectations.
- **CN:** 调用 `pytest.raises`、`run_parser` 执行测试逻辑。 使用 `pytest.raises` 确认预期的失败路径。 使用上下文管理器控制资源或预期行为。

### Lines 1551-1585

```python


@pytest.mark.parametrize("reg_layout, shared_layout, shape, bitwidth, ref_conflicts", [
    (ttgl.BlockedLayout([1], [32], [4], [0]), ttgl.SwizzledSharedLayout(1, 1, 1, order=[0]), [32], 32, 0),
    (ttgl.BlockedLayout([1], [32], [4], [0]), ttgl.SwizzledSharedLayout(1, 1, 1, order=[0]), [32], 16, 0),
    # MMAv3 accumulator tile lowered with the 128B swizzle (WGMMA default path).
    (ttgl.NVMMADistributedLayout(version=[3, 0], warps_per_cta=[4, 1], instr_shape=[16, 32, 16]),
     ttgl.NVMMASharedLayout(swizzle_byte_width=128, element_bitwidth=16, rank=2), [128, 128], 16, 0),
    # Small-M tiles disable swizzling entirely.
    # MMAv2 rhs operand emitted with the 64B swizzle.
    (ttgl.DotOperandLayout(
        operand_index=1, parent=ttgl.NVMMADistributedLayout(version=[2, 0], warps_per_cta=[1, 4], instr_shape=[16, 8]),
        k_width=2), ttgl.NVMMASharedLayout(swizzle_byte_width=64, element_bitwidth=16, rank=2), [64, 32], 16, 0),
    # MMAv2 lhs operand uses the transposed 64B swizzle flavour.
    (ttgl.DotOperandLayout(
        operand_index=0, parent=ttgl.NVMMADistributedLayout(version=[2, 0], warps_per_cta=[1, 4], instr_shape=[16, 8]),
        k_width=2), ttgl.NVMMASharedLayout(swizzle_byte_width=64, element_bitwidth=16, rank=2,
                                           transposed=True), [32, 64], 16, 0),
    # int8 tensor-core tiles follow the 32B swizzle path.
    (ttgl.DotOperandLayout(
        operand_index=1, parent=ttgl.NVMMADistributedLayout(version=[2, 0], warps_per_cta=[1, 4], instr_shape=[16, 8]),
        k_width=1), ttgl.NVMMASharedLayout(swizzle_byte_width=32, element_bitwidth=8, rank=2), [8, 32], 8, 0),
    # Small-M tiles disable swizzling entirely.
    (ttgl.NVMMADistributedLayout(version=[2, 0], warps_per_cta=[4, 1], instr_shape=[16, 8]),
     ttgl.NVMMASharedLayout(swizzle_byte_width=64, element_bitwidth=16, rank=2, transposed=True), [64, 64], 16, 0),
    (ttgl.NVMMADistributedLayout(version=[3, 0], warps_per_cta=[2, 2], instr_shape=[16, 32, 16]),
     ttgl.NVMMASharedLayout(swizzle_byte_width=64, element_bitwidth=16, rank=2), [64, 32], 16, 0),
    (ttgl.NVMMADistributedLayout(version=[2, 0], warps_per_cta=[4, 1], instr_shape=[16, 8]),
     ttgl.NVMMASharedLayout(swizzle_byte_width=32, element_bitwidth=8, rank=2), [32, 32], 8, 0),
    (ttgl.NVMMADistributedLayout(version=[2, 0], warps_per_cta=[2, 4], instr_shape=[16, 8]),
     ttgl.NVMMASharedLayout(swizzle_byte_width=0, element_bitwidth=16, rank=2), [4, 64], 16, 3),
    (ttgl.NVMMADistributedLayout(version=[3, 0], warps_per_cta=[4, 1], instr_shape=[16, 32, 16]),
     ttgl.NVMMASharedLayout(swizzle_byte_width=128, element_bitwidth=32, rank=2), [128, 64], 32, 1),
])
def test_bank_conflicts(reg_layout, shared_layout, shape, bitwidth, ref_conflicts):
```
- **EN:** Defines the test function `test_bank_conflicts`. Decorators: `pytest.mark.parametrize('reg_layout, shared_layout, shape, bitwidth, ref_conflicts', [(ttgl.BlockedLayout([1], [32], [4], [0]), ttgl.SwizzledSharedLayout(1, 1, 1, order=[0]), [32], 32, 0), (ttgl.BlockedLayout([1], [32], [4], [0]), ttgl.SwizzledSharedLayout(1, 1, 1, order=[0]), [32], 16, 0), (ttgl.NVMMADistributedLayout(version=[3, 0], warps_per_cta=[4, 1], instr_shape=[16, 32, 16]), ttgl.NVMMASharedLayout(swizzle_byte_width=128, element_bitwidth=16, rank=2), [128, 128], 16, 0), (ttgl.DotOperandLayout(operand_index=1, parent=ttgl.NVMMADistributedLayout(version=[2, 0], warps_per_cta=[1, 4], instr_shape=[16, 8]), k_width=2), ttgl.NVMMASharedLayout(swizzle_byte_width=64, element_bitwidth=16, rank=2), [64, 32], 16, 0), (ttgl.DotOperandLayout(operand_index=0, parent=ttgl.NVMMADistributedLayout(version=[2, 0], warps_per_cta=[1, 4], instr_shape=[16, 8]), k_width=2), ttgl.NVMMASharedLayout(swizzle_byte_width=64, element_bitwidth=16, rank=2, transposed=True), [32, 64], 16, 0), (ttgl.DotOperandLayout(operand_index=1, parent=ttgl.NVMMADistributedLayout(version=[2, 0], warps_per_cta=[1, 4], instr_shape=[16, 8]), k_width=1), ttgl.NVMMASharedLayout(swizzle_byte_width=32, element_bitwidth=8, rank=2), [8, 32], 8, 0), (ttgl.NVMMADistributedLayout(version=[2, 0], warps_per_cta=[4, 1], instr_shape=[16, 8]), ttgl.NVMMASharedLayout(swizzle_byte_width=64, element_bitwidth=16, rank=2, transposed=True), [64, 64], 16, 0), (ttgl.NVMMADistributedLayout(version=[3, 0], warps_per_cta=[2, 2], instr_shape=[16, 32, 16]), ttgl.NVMMASharedLayout(swizzle_byte_width=64, element_bitwidth=16, rank=2), [64, 32], 16, 0), (ttgl.NVMMADistributedLayout(version=[2, 0], warps_per_cta=[4, 1], instr_shape=[16, 8]), ttgl.NVMMASharedLayout(swizzle_byte_width=32, element_bitwidth=8, rank=2), [32, 32], 8, 0), (ttgl.NVMMADistributedLayout(version=[2, 0], warps_per_cta=[2, 4], instr_shape=[16, 8]), ttgl.NVMMASharedLayout(swizzle_byte_width=0, element_bitwidth=16, rank=2), [4, 64], 16, 3), (ttgl.NVMMADistributedLayout(version=[3, 0], warps_per_cta=[4, 1], instr_shape=[16, 32, 16]), ttgl.NVMMASharedLayout(swizzle_byte_width=128, element_bitwidth=32, rank=2), [128, 64], 32, 1)])`. Parameters: `reg_layout`, `shared_layout`, `shape`, `bitwidth`, `ref_conflicts`. Nested definitions in this scope: `kernel`. Key calls include `pytest.mark.parametrize`, `run_parser`, `ttgl.distributed_type`, `ttgl.shared_memory_descriptor_type`, `ttgl.bank_conflicts`, `ttgl.static_assert`, and 5 more. This scope touches pytest parametrization, tensor/descriptor metadata, layout transformation reasoning, random-data generation.
- **CN:** 定义测试函数 `test_bank_conflicts`。 装饰器：`pytest.mark.parametrize('reg_layout, shared_layout, shape, bitwidth, ref_conflicts', [(ttgl.BlockedLayout([1], [32], [4], [0]), ttgl.SwizzledSharedLayout(1, 1, 1, order=[0]), [32], 32, 0), (ttgl.BlockedLayout([1], [32], [4], [0]), ttgl.SwizzledSharedLayout(1, 1, 1, order=[0]), [32], 16, 0), (ttgl.NVMMADistributedLayout(version=[3, 0], warps_per_cta=[4, 1], instr_shape=[16, 32, 16]), ttgl.NVMMASharedLayout(swizzle_byte_width=128, element_bitwidth=16, rank=2), [128, 128], 16, 0), (ttgl.DotOperandLayout(operand_index=1, parent=ttgl.NVMMADistributedLayout(version=[2, 0], warps_per_cta=[1, 4], instr_shape=[16, 8]), k_width=2), ttgl.NVMMASharedLayout(swizzle_byte_width=64, element_bitwidth=16, rank=2), [64, 32], 16, 0), (ttgl.DotOperandLayout(operand_index=0, parent=ttgl.NVMMADistributedLayout(version=[2, 0], warps_per_cta=[1, 4], instr_shape=[16, 8]), k_width=2), ttgl.NVMMASharedLayout(swizzle_byte_width=64, element_bitwidth=16, rank=2, transposed=True), [32, 64], 16, 0), (ttgl.DotOperandLayout(operand_index=1, parent=ttgl.NVMMADistributedLayout(version=[2, 0], warps_per_cta=[1, 4], instr_shape=[16, 8]), k_width=1), ttgl.NVMMASharedLayout(swizzle_byte_width=32, element_bitwidth=8, rank=2), [8, 32], 8, 0), (ttgl.NVMMADistributedLayout(version=[2, 0], warps_per_cta=[4, 1], instr_shape=[16, 8]), ttgl.NVMMASharedLayout(swizzle_byte_width=64, element_bitwidth=16, rank=2, transposed=True), [64, 64], 16, 0), (ttgl.NVMMADistributedLayout(version=[3, 0], warps_per_cta=[2, 2], instr_shape=[16, 32, 16]), ttgl.NVMMASharedLayout(swizzle_byte_width=64, element_bitwidth=16, rank=2), [64, 32], 16, 0), (ttgl.NVMMADistributedLayout(version=[2, 0], warps_per_cta=[4, 1], instr_shape=[16, 8]), ttgl.NVMMASharedLayout(swizzle_byte_width=32, element_bitwidth=8, rank=2), [32, 32], 8, 0), (ttgl.NVMMADistributedLayout(version=[2, 0], warps_per_cta=[2, 4], instr_shape=[16, 8]), ttgl.NVMMASharedLayout(swizzle_byte_width=0, element_bitwidth=16, rank=2), [4, 64], 16, 3), (ttgl.NVMMADistributedLayout(version=[3, 0], warps_per_cta=[4, 1], instr_shape=[16, 32, 16]), ttgl.NVMMASharedLayout(swizzle_byte_width=128, element_bitwidth=32, rank=2), [128, 64], 32, 1)])`。 参数：`reg_layout`、`shared_layout`、`shape`、`bitwidth`、`ref_conflicts`。 该作用域中的嵌套定义：`kernel`。 关键调用包括 `pytest.mark.parametrize`、`run_parser`、`ttgl.distributed_type`、`ttgl.shared_memory_descriptor_type`、`ttgl.bank_conflicts`、`ttgl.static_assert` 等另外 5 项。 该作用域涉及pytest 参数化、张量/描述符元数据、布局变换推理、随机数据生成。

#### Lines 1586-1589

```python
    dtype = {8: ttgl.int8, 16: ttgl.float16, 32: ttgl.float32}[bitwidth]
    args = (ttgl.distributed_type(dtype, shape,
                                  reg_layout), ttgl.shared_memory_descriptor_type(dtype, shape, shared_layout,
                                                                                  shape), ref_conflicts)
```
- **EN:** Prepares or updates state through `dtype`, `args`. Invokes `ttgl.distributed_type`, `ttgl.shared_memory_descriptor_type` to execute the test logic. Relevant themes: tensor/descriptor metadata, layout transformation reasoning.
- **CN:** 通过 `dtype`、`args` 准备或更新状态。 调用 `ttgl.distributed_type`、`ttgl.shared_memory_descriptor_type` 执行测试逻辑。 相关主题：张量/描述符元数据、布局变换推理。

#### Lines 1590-1592

```python

    @gluon.jit
    def kernel(reg_type: ttgl.constexpr, shared_type: ttgl.constexpr, ref_conflicts: ttgl.constexpr):
```
- **EN:** Defines the helper function `kernel`. Decorators: `gluon.jit`. Parameters: `reg_type`, `shared_type`, `ref_conflicts`. Key calls include `ttgl.bank_conflicts`, `ttgl.static_assert`.
- **CN:** 定义辅助函数 `kernel`。 装饰器：`gluon.jit`。 参数：`reg_type`、`shared_type`、`ref_conflicts`。 关键调用包括 `ttgl.bank_conflicts`、`ttgl.static_assert`。

##### Lines 1593-1594

```python
        conflicts: ttgl.constexpr = ttgl.bank_conflicts(reg_type, shared_type)
        ttgl.static_assert(conflicts == ref_conflicts)
```
- **EN:** Prepares or updates state through `conflicts`. Invokes `ttgl.bank_conflicts`, `ttgl.static_assert` to execute the test logic.
- **CN:** 通过 `conflicts` 准备或更新状态。 调用 `ttgl.bank_conflicts`、`ttgl.static_assert` 执行测试逻辑。

#### Lines 1595-1596

```python

    run_parser(kernel, args=args, target=AMPERE_TARGET)
```
- **EN:** Invokes `run_parser` to execute the test logic.
- **CN:** 调用 `run_parser` 执行测试逻辑。

### Lines 1597-1611

```python


@pytest.mark.parametrize(
    "layout, shape",
    [
        (ttgl.BlockedLayout([1], [4], [4], [0]), [16]),
        (ttgl.BlockedLayout([1], [4], [4], [0], [[1], [0]]), [32]),
        (ttgl.BlockedLayout([8, 1], [8, 4], [1, 4], [0, 1], [[0, 1]]), [64, 128]),
        (ttgl.NVMMASharedLayout(swizzle_byte_width=128, element_bitwidth=16, rank=2), [64, 64]),
        (TensorMemoryLayout((64, 64), col_stride=2), [64, 64]),
    ],
)
def test_to_linear_layout(layout, shape, capsys):

    @gluon.jit
```
- **EN:** Defines the test function `test_to_linear_layout`. Decorators: `pytest.mark.parametrize('layout, shape', [(ttgl.BlockedLayout([1], [4], [4], [0]), [16]), (ttgl.BlockedLayout([1], [4], [4], [0], [[1], [0]]), [32]), (ttgl.BlockedLayout([8, 1], [8, 4], [1, 4], [0, 1], [[0, 1]]), [64, 128]), (ttgl.NVMMASharedLayout(swizzle_byte_width=128, element_bitwidth=16, rank=2), [64, 64]), (TensorMemoryLayout((64, 64), col_stride=2), [64, 64])])`. Parameters: `layout`, `shape`, `capsys`. Nested definitions in this scope: `kernel`. Key calls include `pytest.mark.parametrize`, `run_parser`, `isinstance`, `ttgl.to_linear_layout`, `ttgl.static_print`, `capsys.readouterr`, and 3 more. This scope touches pytest parametrization, debugging and inspection paths, layout transformation reasoning.
- **CN:** 定义测试函数 `test_to_linear_layout`。 装饰器：`pytest.mark.parametrize('layout, shape', [(ttgl.BlockedLayout([1], [4], [4], [0]), [16]), (ttgl.BlockedLayout([1], [4], [4], [0], [[1], [0]]), [32]), (ttgl.BlockedLayout([8, 1], [8, 4], [1, 4], [0, 1], [[0, 1]]), [64, 128]), (ttgl.NVMMASharedLayout(swizzle_byte_width=128, element_bitwidth=16, rank=2), [64, 64]), (TensorMemoryLayout((64, 64), col_stride=2), [64, 64])])`。 参数：`layout`、`shape`、`capsys`。 该作用域中的嵌套定义：`kernel`。 关键调用包括 `pytest.mark.parametrize`、`run_parser`、`isinstance`、`ttgl.to_linear_layout`、`ttgl.static_print`、`capsys.readouterr` 等另外 3 项。 该作用域涉及pytest 参数化、调试与检查路径、布局变换推理。

#### Lines 1611-1612

```python
    @gluon.jit
    def kernel(layout: ttgl.constexpr, shape: ttgl.constexpr):
```
- **EN:** Defines the helper function `kernel`. Decorators: `gluon.jit`. Parameters: `layout`, `shape`. Key calls include `ttgl.to_linear_layout`, `ttgl.static_print`. This scope touches debugging and inspection paths, layout transformation reasoning.
- **CN:** 定义辅助函数 `kernel`。 装饰器：`gluon.jit`。 参数：`layout`、`shape`。 关键调用包括 `ttgl.to_linear_layout`、`ttgl.static_print`。 该作用域涉及调试与检查路径、布局变换推理。

##### Lines 1613-1614

```python
        computed: ttgl.constexpr = ttgl.to_linear_layout(layout, shape)
        ttgl.static_print(computed)
```
- **EN:** Prepares or updates state through `computed`. Invokes `ttgl.to_linear_layout`, `ttgl.static_print` to execute the test logic. Relevant themes: debugging and inspection paths, layout transformation reasoning.
- **CN:** 通过 `computed` 准备或更新状态。 调用 `ttgl.to_linear_layout`、`ttgl.static_print` 执行测试逻辑。 相关主题：调试与检查路径、布局变换推理。

#### Lines 1615-1617

```python

    run_parser(kernel, args=(layout, tuple(shape)), target=AMPERE_TARGET)
    out = capsys.readouterr().out
```
- **EN:** Prepares or updates state through `out`. Invokes `run_parser`, `capsys.readouterr` to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `out` 准备或更新状态。 调用 `run_parser`、`capsys.readouterr` 执行测试逻辑。 相关主题：布局变换推理。

#### Lines 1618-1622

```python
    if isinstance(layout, TensorMemoryLayout):
        assert "rows=" in out
        assert "cols=" in out
    else:
        assert "DistributedLinearLayout" in out or "SharedLinearLayout" in out
```
- **EN:** Invokes `isinstance` to execute the test logic. Validates behavior with 3 assertion(s). Branches on runtime or test conditions. Relevant themes: layout transformation reasoning.
- **CN:** 调用 `isinstance` 执行测试逻辑。 通过 3 个断言验证行为。 根据运行时或测试条件进行分支。 相关主题：布局变换推理。

### Lines 1623-1629

```python


@filecheck_test
@gluon.jit
def test_zeros():
    # CHECK: [[BLOCKED:#.*]] = #ttg.blocked<{sizePerThread = [2]
    # CHECK: [[BLOCKED2D:#.*]] = #ttg.blocked<{sizePerThread = [1, 2]
```
- **EN:** Defines the test function `test_zeros`. Decorators: `filecheck_test`, `gluon.jit`. Key calls include `ttgl.BlockedLayout`, `ttgl.zeros`, `ttgl.full_like`, `ttgl.zeros_like`. This scope touches layout transformation reasoning.
- **CN:** 定义测试函数 `test_zeros`。 装饰器：`filecheck_test`、`gluon.jit`。 关键调用包括 `ttgl.BlockedLayout`、`ttgl.zeros`、`ttgl.full_like`、`ttgl.zeros_like`。 该作用域涉及布局变换推理。

#### Lines 1630-1652

```python
    layout: ttgl.constexpr = ttgl.BlockedLayout([2], [32], [4], [0])
    layout_2d: ttgl.constexpr = ttgl.BlockedLayout([1, 2], [4, 8], [4, 1], [1, 0])

    # CHECK: arith.constant dense<0.000000e+00> : tensor<32xf32, [[BLOCKED]]>
    a = ttgl.zeros([32], ttgl.float32, layout)

    # CHECK: arith.constant dense<7.000000e+00> : tensor<32xf32, [[BLOCKED]]>
    ttgl.full_like(a, 7)

    # CHECK: arith.constant dense<0.000000e+00> : tensor<32xf32, [[BLOCKED]]>
    ttgl.zeros_like(a)

    # CHECK: arith.constant dense<0.000000e+00> : tensor<64xf32, [[BLOCKED]]>
    ttgl.zeros_like(a, shape=[64])

    # CHECK: arith.constant dense<0> : tensor<16x16xi8, [[BLOCKED2D]]>
    ttgl.zeros_like(a, shape=[16, 16], dtype=ttgl.int8, layout=layout_2d)

    # CHECK: arith.constant dense<7> : tensor<8x8xi16, [[BLOCKED2D]]>
    ttgl.full_like(a, 7, shape=[8, 8], dtype=ttgl.int16, layout=layout_2d)

    # CHECK: arith.constant 0.000000e+00 : f32
    ttgl.zeros((), ttgl.float32, layout)
```
- **EN:** Prepares or updates state through `layout`, `layout_2d`, `a`. Invokes `ttgl.BlockedLayout`, `ttgl.zeros`, `ttgl.full_like`, `ttgl.zeros_like` to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `layout`、`layout_2d`、`a` 准备或更新状态。 调用 `ttgl.BlockedLayout`、`ttgl.zeros`、`ttgl.full_like`、`ttgl.zeros_like` 执行测试逻辑。 相关主题：布局变换推理。

### Lines 1653-1658

```python


@filecheck_test
@gluon.jit
def test_barrier():
    # CHECK: ttg.barrier
```
- **EN:** Defines the test function `test_barrier`. Decorators: `filecheck_test`, `gluon.jit`. Key calls include `ttgl.barrier`.
- **CN:** 定义测试函数 `test_barrier`。 装饰器：`filecheck_test`、`gluon.jit`。 关键调用包括 `ttgl.barrier`。

#### Lines 1659-1659

```python
    ttgl.barrier()
```
- **EN:** Invokes `ttgl.barrier` to execute the test logic.
- **CN:** 调用 `ttgl.barrier` 执行测试逻辑。

### Lines 1660-1665

```python


@filecheck_test
@gluon.jit
def test_fence_async_shared():
    # CHECK: ttng.fence_async_shared {bCluster = false}
```
- **EN:** Defines the test function `test_fence_async_shared`. Decorators: `filecheck_test`, `gluon.jit`. Key calls include `blackwell.fence_async_shared`.
- **CN:** 定义测试函数 `test_fence_async_shared`。 装饰器：`filecheck_test`、`gluon.jit`。 关键调用包括 `blackwell.fence_async_shared`。

#### Lines 1666-1669

```python
    blackwell.fence_async_shared()

    # CHECK-NEXT: ttng.fence_async_shared {bCluster = true}
    blackwell.fence_async_shared(cluster=True)
```
- **EN:** Invokes `blackwell.fence_async_shared` to execute the test logic.
- **CN:** 调用 `blackwell.fence_async_shared` 执行测试逻辑。

### Lines 1670-1675

```python


@filecheck_test
@gluon.jit
def test_barrier_cluster_single_cta():
    # CHECK: ttg.barrier
```
- **EN:** Defines the test function `test_barrier_cluster_single_cta`. Decorators: `filecheck_test`, `gluon.jit`. Key calls include `ttgl.barrier`.
- **CN:** 定义测试函数 `test_barrier_cluster_single_cta`。 装饰器：`filecheck_test`、`gluon.jit`。 关键调用包括 `ttgl.barrier`。

#### Lines 1676-1676

```python
    ttgl.barrier(cluster=True)
```
- **EN:** Invokes `ttgl.barrier` to execute the test logic.
- **CN:** 调用 `ttgl.barrier` 执行测试逻辑。

### Lines 1677-1680

```python


@gluon.jit
def cluster_barrier_multi_cta_kernel():
```
- **EN:** Defines the helper function `cluster_barrier_multi_cta_kernel`. Decorators: `gluon.jit`. Key calls include `ttgl.barrier`.
- **CN:** 定义辅助函数 `cluster_barrier_multi_cta_kernel`。 装饰器：`gluon.jit`。 关键调用包括 `ttgl.barrier`。

#### Lines 1681-1681

```python
    ttgl.barrier(cluster=True)
```
- **EN:** Invokes `ttgl.barrier` to execute the test logic.
- **CN:** 调用 `ttgl.barrier` 执行测试逻辑。

### Lines 1682-1684

```python


def test_cluster_barrier_multi_cta():
```
- **EN:** Defines the test function `test_cluster_barrier_multi_cta`. Key calls include `run_parser`, `expecttest.assert_expected_inline`, `anonymize_ir`, `make_args`, `mod.str_nodebug`. This scope touches debugging and inspection paths.
- **CN:** 定义测试函数 `test_cluster_barrier_multi_cta`。 关键调用包括 `run_parser`、`expecttest.assert_expected_inline`、`anonymize_ir`、`make_args`、`mod.str_nodebug`。 该作用域涉及调试与检查路径。

#### Lines 1685-1694

```python
    mod = run_parser(cluster_barrier_multi_cta_kernel, *make_args(num_ctas=2), target=BLACKWELL_TARGET)
    expecttest.assert_expected_inline(
        anonymize_ir(mod.str_nodebug()), """\
module attributes {"ttg.num-ctas" = 2 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "...", "ttg.threads-per-warp" = 32 : i32} {
  tt.func public @cluster_barrier_multi_cta_kernel() attributes {noinline = false} {
    ttng.cluster_barrier
    tt.return
  }
}
""")
```
- **EN:** Prepares or updates state through `mod`. Invokes `run_parser`, `make_args`, `expecttest.assert_expected_inline`, `anonymize_ir`, `mod.str_nodebug` to execute the test logic. Relevant themes: debugging and inspection paths.
- **CN:** 通过 `mod` 准备或更新状态。 调用 `run_parser`、`make_args`、`expecttest.assert_expected_inline`、`anonymize_ir`、`mod.str_nodebug` 执行测试逻辑。 相关主题：调试与检查路径。

### Lines 1695-1698

```python


@gluon.jit
def cluster_arrive_wait_ops_kernel():
```
- **EN:** Defines the helper function `cluster_arrive_wait_ops_kernel`. Decorators: `gluon.jit`. Key calls include `cluster.arrive`, `cluster.wait`.
- **CN:** 定义辅助函数 `cluster_arrive_wait_ops_kernel`。 装饰器：`gluon.jit`。 关键调用包括 `cluster.arrive`、`cluster.wait`。

#### Lines 1699-1700

```python
    cluster.arrive()
    cluster.wait()
```
- **EN:** Invokes `cluster.arrive`, `cluster.wait` to execute the test logic.
- **CN:** 调用 `cluster.arrive`、`cluster.wait` 执行测试逻辑。

### Lines 1701-1703

```python


def test_cluster_arrive_wait_ops():
```
- **EN:** Defines the test function `test_cluster_arrive_wait_ops`. Key calls include `run_parser`, `expecttest.assert_expected_inline`, `anonymize_ir`, `make_args`, `mod.str_nodebug`. This scope touches debugging and inspection paths.
- **CN:** 定义测试函数 `test_cluster_arrive_wait_ops`。 关键调用包括 `run_parser`、`expecttest.assert_expected_inline`、`anonymize_ir`、`make_args`、`mod.str_nodebug`。 该作用域涉及调试与检查路径。

#### Lines 1704-1714

```python
    mod = run_parser(cluster_arrive_wait_ops_kernel, *make_args(num_ctas=2), target=HOPPER_TARGET)
    expecttest.assert_expected_inline(
        anonymize_ir(mod.str_nodebug()), """\
module attributes {"ttg.num-ctas" = 2 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "...", "ttg.threads-per-warp" = 32 : i32} {
  tt.func public @cluster_arrive_wait_ops_kernel() attributes {noinline = false} {
    ttng.cluster_arrive
    ttng.cluster_wait
    tt.return
  }
}
""")
```
- **EN:** Prepares or updates state through `mod`. Invokes `run_parser`, `make_args`, `expecttest.assert_expected_inline`, `anonymize_ir`, `mod.str_nodebug` to execute the test logic. Relevant themes: debugging and inspection paths.
- **CN:** 通过 `mod` 准备或更新状态。 调用 `run_parser`、`make_args`、`expecttest.assert_expected_inline`、`anonymize_ir`、`mod.str_nodebug` 执行测试逻辑。 相关主题：调试与检查路径。

### Lines 1715-1718

```python


@gluon.jit
def cluster_barrier_relaxed_kernel():
```
- **EN:** Defines the helper function `cluster_barrier_relaxed_kernel`. Decorators: `gluon.jit`. Key calls include `cluster.barrier`.
- **CN:** 定义辅助函数 `cluster_barrier_relaxed_kernel`。 装饰器：`gluon.jit`。 关键调用包括 `cluster.barrier`。

#### Lines 1719-1719

```python
    cluster.barrier(relaxed=True)
```
- **EN:** Invokes `cluster.barrier` to execute the test logic.
- **CN:** 调用 `cluster.barrier` 执行测试逻辑。

### Lines 1720-1722

```python


def test_cluster_barrier_relaxed():
```
- **EN:** Defines the test function `test_cluster_barrier_relaxed`. Key calls include `run_parser`, `expecttest.assert_expected_inline`, `anonymize_ir`, `make_args`, `mod.str_nodebug`. This scope touches debugging and inspection paths.
- **CN:** 定义测试函数 `test_cluster_barrier_relaxed`。 关键调用包括 `run_parser`、`expecttest.assert_expected_inline`、`anonymize_ir`、`make_args`、`mod.str_nodebug`。 该作用域涉及调试与检查路径。

#### Lines 1723-1732

```python
    mod = run_parser(cluster_barrier_relaxed_kernel, *make_args(num_ctas=2), target=HOPPER_TARGET)
    expecttest.assert_expected_inline(
        anonymize_ir(mod.str_nodebug()), """\
module attributes {"ttg.num-ctas" = 2 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "...", "ttg.threads-per-warp" = 32 : i32} {
  tt.func public @cluster_barrier_relaxed_kernel() attributes {noinline = false} {
    ttng.cluster_barrier {relaxed = true}
    tt.return
  }
}
""")
```
- **EN:** Prepares or updates state through `mod`. Invokes `run_parser`, `make_args`, `expecttest.assert_expected_inline`, `anonymize_ir`, `mod.str_nodebug` to execute the test logic. Relevant themes: debugging and inspection paths.
- **CN:** 通过 `mod` 准备或更新状态。 调用 `run_parser`、`make_args`、`expecttest.assert_expected_inline`、`anonymize_ir`、`mod.str_nodebug` 执行测试逻辑。 相关主题：调试与检查路径。

### Lines 1733-1737

```python


@filecheck_test
@gluon.jit
def test_inline_asm_elementwise():
```
- **EN:** Defines the test function `test_inline_asm_elementwise`. Decorators: `filecheck_test`, `gluon.jit`. Key calls include `ttgl.BlockedLayout`, `ttgl.arange`, `ttgl.inline_asm_elementwise`. This scope touches layout transformation reasoning.
- **CN:** 定义测试函数 `test_inline_asm_elementwise`。 装饰器：`filecheck_test`、`gluon.jit`。 关键调用包括 `ttgl.BlockedLayout`、`ttgl.arange`、`ttgl.inline_asm_elementwise`。 该作用域涉及布局变换推理。

#### Lines 1738-1741

```python
    layout: ttgl.constexpr = ttgl.BlockedLayout([1], [32], [4], [0])
    x = ttgl.arange(0, 16, layout)
    # CHECK: elementwise_inline_asm {{.*}} : tensor<16xi32, [[BLOCKED:#.*]]> -> tensor<16xi32, [[BLOCKED]]>
    ttgl.inline_asm_elementwise("mov $0, $0;", "=r,r", [x], dtype=x.dtype, is_pure=True, pack=1)
```
- **EN:** Prepares or updates state through `layout`, `x`. Invokes `ttgl.BlockedLayout`, `ttgl.arange`, `ttgl.inline_asm_elementwise` to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `layout`、`x` 准备或更新状态。 调用 `ttgl.BlockedLayout`、`ttgl.arange`、`ttgl.inline_asm_elementwise` 执行测试逻辑。 相关主题：布局变换推理。

### Lines 1742-1745

```python


@gluon.jit
def load_kernel(inp, xnumel):
```
- **EN:** Defines the helper function `load_kernel`. Decorators: `gluon.jit`. Parameters: `inp`, `xnumel`. Key calls include `ttgl.BlockedLayout`, `ttgl.arange`, `ttgl.load`. This scope touches layout transformation reasoning.
- **CN:** 定义辅助函数 `load_kernel`。 装饰器：`gluon.jit`。 参数：`inp`、`xnumel`。 关键调用包括 `ttgl.BlockedLayout`、`ttgl.arange`、`ttgl.load`。 该作用域涉及布局变换推理。

#### Lines 1746-1749

```python
    block_layout: ttgl.constexpr = ttgl.BlockedLayout([1], [32], [4], [0])
    xindex = ttgl.arange(0, 128, block_layout)
    mask = xindex < xnumel
    ttgl.load(inp + xindex, mask=mask, other=0.0)
```
- **EN:** Prepares or updates state through `block_layout`, `xindex`, `mask`. Invokes `ttgl.BlockedLayout`, `ttgl.arange`, `ttgl.load` to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `block_layout`、`xindex`、`mask` 准备或更新状态。 调用 `ttgl.BlockedLayout`、`ttgl.arange`、`ttgl.load` 执行测试逻辑。 相关主题：布局变换推理。

### Lines 1750-1753

```python


@pytest.mark.parametrize("target", ALL_TARGETS)
def test_load(target):
```
- **EN:** Defines the test function `test_load`. Decorators: `pytest.mark.parametrize('target', ALL_TARGETS)`. Parameters: `target`. Key calls include `pytest.mark.parametrize`, `run_parser`, `expecttest.assert_expected_inline`, `anonymize_ir`, `make_args`, `mod.str_nodebug`, and 1 more. This scope touches pytest parametrization, debugging and inspection paths.
- **CN:** 定义测试函数 `test_load`。 装饰器：`pytest.mark.parametrize('target', ALL_TARGETS)`。 参数：`target`。 关键调用包括 `pytest.mark.parametrize`、`run_parser`、`expecttest.assert_expected_inline`、`anonymize_ir`、`make_args`、`mod.str_nodebug` 等另外 1 项。 该作用域涉及pytest 参数化、调试与检查路径。

#### Lines 1754-1771

```python
    mod = run_parser(load_kernel, *make_args(MockTensor(ttgl.float32), xnumel=100), target=target)
    expecttest.assert_expected_inline(
        anonymize_ir(mod.str_nodebug()), """\
#blocked = #ttg.blocked<{sizePerThread = [1], threadsPerWarp = [32], warpsPerCTA = [4], order = [0]}>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "...", "ttg.threads-per-warp" = 32 : i32} {
  tt.func public @load_kernel(%arg0: !tt.ptr<f32> {tt.divisibility = 16 : i32}, %arg1: i32) attributes {noinline = false} {
    %0 = tt.make_range {end = 128 : i32, start = 0 : i32} : tensor<128xi32, #blocked>
    %1 = tt.splat %arg1 : i32 -> tensor<128xi32, #blocked>
    %2 = arith.cmpi slt, %0, %1 : tensor<128xi32, #blocked>
    %3 = tt.splat %arg0 : !tt.ptr<f32> -> tensor<128x!tt.ptr<f32>, #blocked>
    %4 = tt.addptr %3, %0 : tensor<128x!tt.ptr<f32>, #blocked>, tensor<128xi32, #blocked>
    %cst = arith.constant 0.000000e+00 : f32
    %cst_0 = arith.constant dense<0.000000e+00> : tensor<128xf32, #blocked>
    %5 = tt.load %4, %2, %cst_0 : tensor<128x!tt.ptr<f32>, #blocked>
    tt.return
  }
}
""")
```
- **EN:** Prepares or updates state through `mod`. Invokes `run_parser`, `make_args`, `MockTensor`, `expecttest.assert_expected_inline`, `anonymize_ir`, `mod.str_nodebug` to execute the test logic. Relevant themes: debugging and inspection paths.
- **CN:** 通过 `mod` 准备或更新状态。 调用 `run_parser`、`make_args`、`MockTensor`、`expecttest.assert_expected_inline`、`anonymize_ir`、`mod.str_nodebug` 执行测试逻辑。 相关主题：调试与检查路径。

### Lines 1772-1775

```python


@gluon.jit
def async_copy_kernel(inp, xnumel, XBLOCK: ttgl.constexpr):
```
- **EN:** Defines the helper function `async_copy_kernel`. Decorators: `gluon.jit`. Parameters: `inp`, `xnumel`, `XBLOCK`. Key calls include `ttgl.allocate_shared_memory`, `ttgl.BlockedLayout`, `ttgl.arange`, `ttgl.max_constancy`, `async_copy.async_copy_global_to_shared`, `async_copy.mbarrier_arrive`, and 4 more. This scope touches cache management behavior, layout transformation reasoning.
- **CN:** 定义辅助函数 `async_copy_kernel`。 装饰器：`gluon.jit`。 参数：`inp`、`xnumel`、`XBLOCK`。 关键调用包括 `ttgl.allocate_shared_memory`、`ttgl.BlockedLayout`、`ttgl.arange`、`ttgl.max_constancy`、`async_copy.async_copy_global_to_shared`、`async_copy.mbarrier_arrive` 等另外 4 项。 该作用域涉及缓存管理行为、布局变换推理。

#### Lines 1776-1789

```python
    smem = ttgl.allocate_shared_memory(inp.dtype.element_ty, [XBLOCK], ttgl.SwizzledSharedLayout(1, 1, 1, order=[0]))
    block_layout: ttgl.constexpr = ttgl.BlockedLayout([2], [32], [4], [0])
    xindex = ttgl.arange(0, XBLOCK, block_layout)
    mask = ttgl.max_constancy(xindex < xnumel, 2)

    async_copy.async_copy_global_to_shared(smem, inp + xindex)
    async_copy.async_copy_global_to_shared(smem, inp + xindex, mask, cache_modifier=".ca", eviction_policy="evict_last",
                                           volatile=True)

    mbar = ttgl.allocate_shared_memory(ttgl.int64, [1], mbarrier.MBarrierLayout())
    async_copy.mbarrier_arrive(mbar)
    async_copy.mbarrier_arrive(mbar, increment_count=False)
    async_copy.commit_group()
    async_copy.wait_group(0)
```
- **EN:** Prepares or updates state through `smem`, `block_layout`, `xindex`, `mask`, `mbar`. Invokes `ttgl.allocate_shared_memory`, `ttgl.SwizzledSharedLayout`, `ttgl.BlockedLayout`, `ttgl.arange`, `ttgl.max_constancy`, `async_copy.async_copy_global_to_shared`, and 4 more to execute the test logic. Relevant themes: cache management behavior, layout transformation reasoning.
- **CN:** 通过 `smem`、`block_layout`、`xindex`、`mask`、`mbar` 准备或更新状态。 调用 `ttgl.allocate_shared_memory`、`ttgl.SwizzledSharedLayout`、`ttgl.BlockedLayout`、`ttgl.arange`、`ttgl.max_constancy`、`async_copy.async_copy_global_to_shared` 等另外 4 项 执行测试逻辑。 相关主题：缓存管理行为、布局变换推理。

### Lines 1790-1793

```python


@pytest.mark.parametrize("target", [AMPERE_TARGET, HOPPER_TARGET, BLACKWELL_TARGET])
def test_async_copy(target):
```
- **EN:** Defines the test function `test_async_copy`. Decorators: `pytest.mark.parametrize('target', [AMPERE_TARGET, HOPPER_TARGET, BLACKWELL_TARGET])`. Parameters: `target`. Key calls include `pytest.mark.parametrize`, `run_parser`, `expecttest.assert_expected_inline`, `anonymize_ir`, `make_args`, `mod.str_nodebug`, and 1 more. This scope touches pytest parametrization, cache management behavior, debugging and inspection paths.
- **CN:** 定义测试函数 `test_async_copy`。 装饰器：`pytest.mark.parametrize('target', [AMPERE_TARGET, HOPPER_TARGET, BLACKWELL_TARGET])`。 参数：`target`。 关键调用包括 `pytest.mark.parametrize`、`run_parser`、`expecttest.assert_expected_inline`、`anonymize_ir`、`make_args`、`mod.str_nodebug` 等另外 1 项。 该作用域涉及pytest 参数化、缓存管理行为、调试与检查路径。

#### Lines 1794-1824

```python
    mod = run_parser(
        async_copy_kernel,
        *make_args(MockTensor(ttgl.float16), xnumel=100, XBLOCK=128),
        target=target,
    )
    expecttest.assert_expected_inline(
        anonymize_ir(mod.str_nodebug()), """\
#blocked = #ttg.blocked<{sizePerThread = [2], threadsPerWarp = [32], warpsPerCTA = [4], order = [0]}>
#shared = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0]}>
#smem = #ttg.shared_memory
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "...", "ttg.threads-per-warp" = 32 : i32} {
  tt.func public @async_copy_kernel(%arg0: !tt.ptr<f16> {tt.divisibility = 16 : i32}, %arg1: i32) attributes {noinline = false} {
    %0 = ttg.local_alloc : () -> !ttg.memdesc<128xf16, #shared, #smem, mutable>
    %1 = tt.make_range {end = 128 : i32, start = 0 : i32} : tensor<128xi32, #blocked>
    %2 = tt.splat %arg1 : i32 -> tensor<128xi32, #blocked>
    %3 = arith.cmpi slt, %1, %2 {tt.constancy = dense<2> : tensor<1xi32>} : tensor<128xi32, #blocked>
    %4 = tt.splat %arg0 : !tt.ptr<f16> -> tensor<128x!tt.ptr<f16>, #blocked>
    %5 = tt.addptr %4, %1 : tensor<128x!tt.ptr<f16>, #blocked>, tensor<128xi32, #blocked>
    %6 = ttg.async_copy_global_to_local %5, %0 : tensor<128x!tt.ptr<f16>, #blocked> -> <128xf16, #shared, #smem, mutable>
    %7 = tt.splat %arg0 : !tt.ptr<f16> -> tensor<128x!tt.ptr<f16>, #blocked>
    %8 = tt.addptr %7, %1 : tensor<128x!tt.ptr<f16>, #blocked>, tensor<128xi32, #blocked>
    %9 = ttg.async_copy_global_to_local %8, %0 mask %3 cacheModifier = ca evictionPolicy = evict_last {isVolatile = true} : tensor<128x!tt.ptr<f16>, #blocked> -> <128xf16, #shared, #smem, mutable>
    %10 = ttg.local_alloc : () -> !ttg.memdesc<1xi64, #shared, #smem, mutable>
    ttng.async_copy_mbarrier_arrive %10 : !ttg.memdesc<1xi64, #shared, #smem, mutable>
    ttng.async_copy_mbarrier_arrive %10 {noIncrement} : !ttg.memdesc<1xi64, #shared, #smem, mutable>
    %11 = ttg.async_commit_group
    %12 = ttg.async_wait {num = 0 : i32}
    tt.return
  }
}
""")
```
- **EN:** Prepares or updates state through `mod`. Invokes `run_parser`, `make_args`, `MockTensor`, `expecttest.assert_expected_inline`, `anonymize_ir`, `mod.str_nodebug` to execute the test logic. Relevant themes: cache management behavior, debugging and inspection paths.
- **CN:** 通过 `mod` 准备或更新状态。 调用 `run_parser`、`make_args`、`MockTensor`、`expecttest.assert_expected_inline`、`anonymize_ir`、`mod.str_nodebug` 执行测试逻辑。 相关主题：缓存管理行为、调试与检查路径。

### Lines 1825-1830

```python


@pytest.mark.parametrize("target", ALL_TARGETS)
def test_split_join_subtile(target):

    @gluon.jit
```
- **EN:** Defines the test function `test_split_join_subtile`. Decorators: `pytest.mark.parametrize('target', ALL_TARGETS)`. Parameters: `target`. Nested definitions in this scope: `kernel`. Key calls include `pytest.mark.parametrize`, `run_parser`, `expecttest.assert_expected_inline`, `ttgl.BlockedLayout`, `ttgl.full`, `anonymize_ir`, and 3 more. This scope touches pytest parametrization, debugging and inspection paths, layout transformation reasoning.
- **CN:** 定义测试函数 `test_split_join_subtile`。 装饰器：`pytest.mark.parametrize('target', ALL_TARGETS)`。 参数：`target`。 该作用域中的嵌套定义：`kernel`。 关键调用包括 `pytest.mark.parametrize`、`run_parser`、`expecttest.assert_expected_inline`、`ttgl.BlockedLayout`、`ttgl.full`、`anonymize_ir` 等另外 3 项。 该作用域涉及pytest 参数化、调试与检查路径、布局变换推理。

#### Lines 1830-1831

```python
    @gluon.jit
    def kernel():
```
- **EN:** Defines the helper function `kernel`. Decorators: `gluon.jit`. Key calls include `ttgl.BlockedLayout`, `ttgl.full`, `x.reshape`, `ttgl.join`. This scope touches layout transformation reasoning.
- **CN:** 定义辅助函数 `kernel`。 装饰器：`gluon.jit`。 关键调用包括 `ttgl.BlockedLayout`、`ttgl.full`、`x.reshape`、`ttgl.join`。 该作用域涉及布局变换推理。

##### Lines 1832-1837

```python
        layout: ttgl.constexpr = ttgl.BlockedLayout([1, 128], [32, 1], [4, 1], [0, 1])
        x = ttgl.full([128, 128], 1, ttgl.int32, layout=layout)

        a, b = x.reshape([128, 2, 64]).permute([0, 2, 1]).split()
        y = ttgl.join(a, b).permute([0, 2, 1]).reshape([128, 128])
        _ = x + y
```
- **EN:** Prepares or updates state through `layout`, `x`, `a`, `b`, `y`, `_`. Invokes `ttgl.BlockedLayout`, `ttgl.full`, `x.reshape`, `ttgl.join` to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `layout`、`x`、`a`、`b`、`y`、`_` 准备或更新状态。 调用 `ttgl.BlockedLayout`、`ttgl.full`、`x.reshape`、`ttgl.join` 执行测试逻辑。 相关主题：布局变换推理。

#### Lines 1838-1859

```python

    mod = run_parser(kernel, target=target)
    expecttest.assert_expected_inline(
        anonymize_ir(mod.str_nodebug()), """\
#blocked = #ttg.blocked<{sizePerThread = [1, 128], threadsPerWarp = [32, 1], warpsPerCTA = [4, 1], order = [0, 1]}>
#blocked1 = #ttg.blocked<{sizePerThread = [1, 2, 64], threadsPerWarp = [32, 1, 1], warpsPerCTA = [4, 1, 1], order = [0, 2, 1]}>
#blocked2 = #ttg.blocked<{sizePerThread = [1, 64, 2], threadsPerWarp = [32, 1, 1], warpsPerCTA = [4, 1, 1], order = [0, 1, 2]}>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "...", "ttg.threads-per-warp" = 32 : i32} {
  tt.func public @kernel() attributes {noinline = false} {
    %c1_i32 = arith.constant 1 : i32
    %cst = arith.constant dense<1> : tensor<128x128xi32, #blocked>
    %0 = tt.reshape %cst : tensor<128x128xi32, #blocked> -> tensor<128x2x64xi32, #blocked1>
    %1 = tt.trans %0 {order = array<i32: 0, 2, 1>} : tensor<128x2x64xi32, #blocked1> -> tensor<128x64x2xi32, #blocked2>
    %outLHS, %outRHS = tt.split %1 : tensor<128x64x2xi32, #blocked2> -> tensor<128x64xi32, #ttg.slice<{dim = 2, parent = #blocked2}>>
    %2 = tt.join %outLHS, %outRHS : tensor<128x64xi32, #ttg.slice<{dim = 2, parent = #blocked2}>> -> tensor<128x64x2xi32, #blocked2>
    %3 = tt.trans %2 {order = array<i32: 0, 2, 1>} : tensor<128x64x2xi32, #blocked2> -> tensor<128x2x64xi32, #blocked1>
    %4 = tt.reshape %3 : tensor<128x2x64xi32, #blocked1> -> tensor<128x128xi32, #blocked>
    %5 = arith.addi %cst, %4 : tensor<128x128xi32, #blocked>
    tt.return
  }
}
""")
```
- **EN:** Prepares or updates state through `mod`. Invokes `run_parser`, `expecttest.assert_expected_inline`, `anonymize_ir`, `mod.str_nodebug` to execute the test logic. Relevant themes: debugging and inspection paths.
- **CN:** 通过 `mod` 准备或更新状态。 调用 `run_parser`、`expecttest.assert_expected_inline`、`anonymize_ir`、`mod.str_nodebug` 执行测试逻辑。 相关主题：调试与检查路径。

### Lines 1860-1867

```python


@filecheck_test
@gluon.jit
def test_auto_layout():
    # CHECK-DAG: [[BLOCKED:#.*]] = #ttg.blocked<{sizePerThread = [1], threadsPerWarp = [32], warpsPerCTA = [4], order = [0]}>
    # CHECK: [[X_1D:%.*]] = arith.constant dense<7> : tensor<16xi32, #gluon.auto_encoding>
    # CHECK: [[Y_1D:%.*]] = arith.constant dense<2> : tensor<8xi32, #gluon.auto_encoding>
```
- **EN:** Defines the test function `test_auto_layout`. Decorators: `filecheck_test`, `gluon.jit`. Key calls include `ttgl.sum`, `ttgl.arange`, `ttgl.set_auto_layout`, `ttgl.full`, `ttgl.BlockedLayout`, `ttgl.AutoLayout`. This scope touches layout transformation reasoning.
- **CN:** 定义测试函数 `test_auto_layout`。 装饰器：`filecheck_test`、`gluon.jit`。 关键调用包括 `ttgl.sum`、`ttgl.arange`、`ttgl.set_auto_layout`、`ttgl.full`、`ttgl.BlockedLayout`、`ttgl.AutoLayout`。 该作用域涉及布局变换推理。

#### Lines 1868-1879

```python
    x = ttgl.full([16], 7, ttgl.int32, layout=ttgl.AutoLayout())[:, None]
    y = ttgl.full([8], 2, ttgl.int32, layout=ttgl.AutoLayout())[None, :]
    # CHECK: arith.addi {{.*}} : tensor<16x8xi32, #gluon.auto_encoding>
    z = x + y
    # CHECK: (tensor<16x8xi32, #gluon.auto_encoding>) -> tensor<16xi32, #gluon.auto_encoding
    ttgl.sum(z, axis=1)

    # CHECK: [[I:%.*]] = tt.make_range {end = 32 : i32, start = 0 : i32} : tensor<32xi32, #gluon.auto_encoding>
    i = ttgl.arange(0, 32)

    # CHECK: gluon.set_auto_layout [[I]] : tensor<32xi32, #gluon.auto_encoding> -> tensor<32xi32, [[BLOCKED]]
    ttgl.set_auto_layout(i, ttgl.BlockedLayout([1], [32], [4], [0]))
```
- **EN:** Prepares or updates state through `x`, `y`, `z`, `i`. Invokes `ttgl.full`, `ttgl.AutoLayout`, `ttgl.sum`, `ttgl.arange`, `ttgl.set_auto_layout`, `ttgl.BlockedLayout` to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `x`、`y`、`z`、`i` 准备或更新状态。 调用 `ttgl.full`、`ttgl.AutoLayout`、`ttgl.sum`、`ttgl.arange`、`ttgl.set_auto_layout`、`ttgl.BlockedLayout` 执行测试逻辑。 相关主题：布局变换推理。

### Lines 1880-1887

```python


@filecheck_test
@gluon.jit
def test_auto_layout_broadcast():
    # CHECK: [[BLOCKED:#.*]] = #ttg.blocked
    # CHECK: [[X:%.*]] = arith.constant dense<1> : tensor<16x1xi32, #gluon.auto_encoding>
    # CHECK: [[Y:%.*]] = arith.constant dense<2> : tensor<1x16xi32, [[BLOCKED]]>
```
- **EN:** Defines the test function `test_auto_layout_broadcast`. Decorators: `filecheck_test`, `gluon.jit`. Key calls include `ttgl.full`, `ttgl.AutoLayout`, `ttgl.BlockedLayout`. This scope touches layout transformation reasoning.
- **CN:** 定义测试函数 `test_auto_layout_broadcast`。 装饰器：`filecheck_test`、`gluon.jit`。 关键调用包括 `ttgl.full`、`ttgl.AutoLayout`、`ttgl.BlockedLayout`。 该作用域涉及布局变换推理。

#### Lines 1888-1901

```python
    x = ttgl.full([16, 1], 1, ttgl.int32, layout=ttgl.AutoLayout())
    y = ttgl.full([1, 16], 2, ttgl.int32, layout=ttgl.BlockedLayout([1, 1], [1, 32], [4, 1], [1, 0]))

    # CHECK: [[XCVT:%.*]] = gluon.set_auto_layout [[X]] : tensor<16x1xi32, #gluon.auto_encoding> -> tensor<16x1xi32, [[BLOCKED]]>
    # CHECK: [[XBCAST:%.*]] = tt.broadcast [[XCVT]]
    # CHECK: [[YBCAST:%.*]] = tt.broadcast [[Y]]
    # CHECK: arith.addi [[XBCAST]], [[YBCAST]] : tensor<16x16xi32, [[BLOCKED]]>
    _ = x + y

    # CHECK: [[XCVT2:%.*]] = gluon.set_auto_layout [[X]] : tensor<16x1xi32, #gluon.auto_encoding> -> tensor<16x1xi32, [[BLOCKED]]>
    # CHECK: [[YBCAST2:%.*]] = tt.broadcast [[Y]]
    # CHECK: [[XBCAST2:%.*]] = tt.broadcast [[XCVT2]]
    # CHECK: arith.muli [[YBCAST2]], [[XBCAST2]] : tensor<16x16xi32, [[BLOCKED]]>
    _ = y * x
```
- **EN:** Prepares or updates state through `x`, `y`, `_`. Invokes `ttgl.full`, `ttgl.AutoLayout`, `ttgl.BlockedLayout` to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `x`、`y`、`_` 准备或更新状态。 调用 `ttgl.full`、`ttgl.AutoLayout`、`ttgl.BlockedLayout` 执行测试逻辑。 相关主题：布局变换推理。

### Lines 1902-1906

```python


@filecheck_test
@gluon.jit
def test_atomic_rmw():
```
- **EN:** Defines the test function `test_atomic_rmw`. Decorators: `filecheck_test`, `gluon.jit`. Key calls include `ttgl.full`, `ttgl.atomic_xchg`, `x.cast`, `ttgl.arange`, `ttgl.atomic_min`, `ttgl.atomic_max`, and 7 more. This scope touches layout transformation reasoning.
- **CN:** 定义测试函数 `test_atomic_rmw`。 装饰器：`filecheck_test`、`gluon.jit`。 关键调用包括 `ttgl.full`、`ttgl.atomic_xchg`、`x.cast`、`ttgl.arange`、`ttgl.atomic_min`、`ttgl.atomic_max` 等另外 7 项。 该作用域涉及布局变换推理。

#### Lines 1907-1938

```python
    x0 = ttgl.full([1], 1, ttgl.int64, layout=ttgl.AutoLayout())
    ptr0 = x0.cast(ttgl.pointer_type(ttgl.int32), bitcast=True).item()
    # CHECK: [[c1:%.*]] = arith.constant 1 : i32
    # CHECK: {{.*}} = tt.atomic_rmw exch, acq_rel, gpu, %{{.*}}, [[c1]], %true : (!tt.ptr<i32>, i32, i1) -> i32
    ttgl.atomic_xchg(ptr0, 1)

    BLOCK: ttgl.constexpr = 128
    x = ttgl.full([BLOCK], 0, ttgl.int64, layout=ttgl.AutoLayout())
    ptr = x.cast(ttgl.pointer_type(ttgl.int32), bitcast=True)
    val = ttgl.full([BLOCK], 1, ttgl.int32, layout=ttgl.AutoLayout())
    mask = ttgl.full([BLOCK], True, ttgl.int1, layout=ttgl.AutoLayout())
    scalar_mask = True
    offset = ttgl.arange(0, BLOCK, layout=ttgl.AutoLayout())
    # CHECK: [[val:%.*]] = arith.constant dense<1> : tensor<128xi32, #gluon.auto_encoding>
    # CHECK: {{.*}} = tt.atomic_rmw min, acq_rel, gpu, %{{.*}}, [[val]], %{{.*}} : (tensor<128x!tt.ptr<i32>, #gluon.auto_encoding>, tensor<128xi32, #gluon.auto_encoding>, tensor<128xi1, #gluon.auto_encoding>) -> tensor<128xi32, #gluon.auto_encoding>
    # CHECK: {{.*}} = tt.atomic_rmw max, acq_rel, gpu, %{{.*}}, [[val]], %{{.*}} : (tensor<128x!tt.ptr<i32>, #gluon.auto_encoding>, tensor<128xi32, #gluon.auto_encoding>, tensor<128xi1, #gluon.auto_encoding>) -> tensor<128xi32, #gluon.auto_encoding>
    # CHECK: {{.*}} = tt.atomic_rmw add, acq_rel, gpu, %{{.*}}, [[val]], %{{.*}} : (tensor<128x!tt.ptr<i32>, #gluon.auto_encoding>, tensor<128xi32, #gluon.auto_encoding>, tensor<128xi1, #gluon.auto_encoding>) -> tensor<128xi32, #gluon.auto_encoding>
    # CHECK: {{.*}} = tt.atomic_rmw and, acq_rel, gpu, %{{.*}}, [[val]], %{{.*}} : (tensor<128x!tt.ptr<i32>, #gluon.auto_encoding>, tensor<128xi32, #gluon.auto_encoding>, tensor<128xi1, #gluon.auto_encoding>) -> tensor<128xi32, #gluon.auto_encoding>
    # CHECK: {{.*}} = tt.atomic_rmw or, acq_rel, gpu, %{{.*}}, [[val]], %{{.*}} : (tensor<128x!tt.ptr<i32>, #gluon.auto_encoding>, tensor<128xi32, #gluon.auto_encoding>, tensor<128xi1, #gluon.auto_encoding>) -> tensor<128xi32, #gluon.auto_encoding>
    # CHECK: {{.*}} = tt.atomic_rmw xor, acq_rel, gpu, %{{.*}}, [[val]], %{{.*}} : (tensor<128x!tt.ptr<i32>, #gluon.auto_encoding>, tensor<128xi32, #gluon.auto_encoding>, tensor<128xi1, #gluon.auto_encoding>) -> tensor<128xi32, #gluon.auto_encoding>
    # CHECK: {{.*}} = tt.atomic_rmw max, acq_rel, gpu, %{{.*}}, [[val]], %{{.*}} : (tensor<128x!tt.ptr<i32>, #gluon.auto_encoding>, tensor<128xi32, #gluon.auto_encoding>, tensor<128xi1, #gluon.auto_encoding>) -> tensor<128xi32, #gluon.auto_encoding>
    # CHECK: {{.*}} = tt.atomic_rmw add, relaxed, gpu, %{{.*}}, [[val]], %{{.*}} : (tensor<128x!tt.ptr<i32>, #gluon.auto_encoding>, tensor<128xi32, #gluon.auto_encoding>, tensor<128xi1, #gluon.auto_encoding>) -> tensor<128xi32, #gluon.auto_encoding>
    # CHECK: {{.*}} = tt.atomic_rmw add, acquire, cta, %{{.*}}, [[val]], %{{.*}} : (tensor<128x!tt.ptr<i32>, #gluon.auto_encoding>, tensor<128xi32, #gluon.auto_encoding>, tensor<128xi1, #gluon.auto_encoding>) -> tensor<128xi32, #gluon.auto_encoding>
    ttgl.atomic_min(offset + ptr, val)
    ttgl.atomic_max(offset + ptr, val)
    ttgl.atomic_add(offset + ptr, val)
    ttgl.atomic_and(offset + ptr, val)
    ttgl.atomic_or(offset + ptr, val)
    ttgl.atomic_xor(offset + ptr, val)
    ttgl.atomic_max(offset + ptr, val, mask=mask)
    ttgl.atomic_add(offset + ptr, val, mask=mask, sem="relaxed")
    ttgl.atomic_add(offset + ptr, val, mask=scalar_mask, sem="acquire", scope="cta")
```
- **EN:** Prepares or updates state through `x0`, `ptr0`, `BLOCK`, `x`, `ptr`, `val`, `mask`, `scalar_mask`, and 1 more. Invokes `ttgl.full`, `ttgl.AutoLayout`, `x0.cast`, `ttgl.pointer_type`, `ttgl.atomic_xchg`, `x.cast`, and 7 more to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `x0`、`ptr0`、`BLOCK`、`x`、`ptr`、`val`、`mask`、`scalar_mask` 等另外 1 项 准备或更新状态。 调用 `ttgl.full`、`ttgl.AutoLayout`、`x0.cast`、`ttgl.pointer_type`、`ttgl.atomic_xchg`、`x.cast` 等另外 7 项 执行测试逻辑。 相关主题：布局变换推理。

### Lines 1939-1944

```python


@filecheck_test
@gluon.jit
def test_atomic_rmw_scalar_masks():
    # CHECK-LABEL: test_atomic_rmw_scalar_masks
```
- **EN:** Defines the test function `test_atomic_rmw_scalar_masks`. Decorators: `filecheck_test`, `gluon.jit`. Key calls include `ttgl.full`, `x.cast`, `ttgl.arange`, `ttgl.atomic_add`, `ttgl.atomic_xchg`, `ttgl.atomic_max`, and 5 more.
- **CN:** 定义测试函数 `test_atomic_rmw_scalar_masks`。 装饰器：`filecheck_test`、`gluon.jit`。 关键调用包括 `ttgl.full`、`x.cast`、`ttgl.arange`、`ttgl.atomic_add`、`ttgl.atomic_xchg`、`ttgl.atomic_max` 等另外 5 项。

#### Lines 1945-1976

```python
    BLOCK: ttgl.constexpr = 128
    x = ttgl.full([BLOCK], 0, ttgl.int64)
    ptr = x.cast(ttgl.pointer_type(ttgl.int32), bitcast=True)
    offs = ttgl.arange(0, BLOCK)
    ptrs = ptr + offs
    val = ttgl.full([BLOCK], 1, ttgl.int32)
    mask = offs >= 0
    scalar_mask = True
    constexpr_value: ttgl.constexpr = 1
    constexpr_mask: ttgl.constexpr = True

    # CHECK: {{.*}} = tt.atomic_rmw add, acq_rel, gpu
    ttgl.atomic_add(ptrs, val, mask=mask)
    # CHECK: {{.*}} = tt.atomic_rmw add, acq_rel, gpu
    ttgl.atomic_add(ptrs, 1, mask=True)
    # CHECK: {{.*}} = tt.atomic_rmw add, acq_rel, gpu
    ttgl.atomic_add(ptrs, constexpr_value, mask=constexpr_mask)
    # CHECK: {{.*}} = tt.atomic_rmw add, acq_rel, gpu
    ttgl.atomic_add(ptrs, val, mask=scalar_mask)

    # CHECK: {{.*}} = tt.atomic_rmw exch, acq_rel, gpu
    ttgl.atomic_xchg(ptrs, 1, mask=True)
    # CHECK: {{.*}} = tt.atomic_rmw max, acq_rel, gpu
    ttgl.atomic_max(ptrs, 1, mask=True)
    # CHECK: {{.*}} = tt.atomic_rmw min, acq_rel, gpu
    ttgl.atomic_min(ptrs, 1, mask=True)
    # CHECK: {{.*}} = tt.atomic_rmw and, acq_rel, gpu
    ttgl.atomic_and(ptrs, 1, mask=True)
    # CHECK: {{.*}} = tt.atomic_rmw or, acq_rel, gpu
    ttgl.atomic_or(ptrs, 1, mask=True)
    # CHECK: {{.*}} = tt.atomic_rmw xor, acq_rel, gpu
    ttgl.atomic_xor(ptrs, 1, mask=True)
```
- **EN:** Prepares or updates state through `BLOCK`, `x`, `ptr`, `offs`, `ptrs`, `val`, `mask`, `scalar_mask`, and 2 more. Invokes `ttgl.full`, `x.cast`, `ttgl.pointer_type`, `ttgl.arange`, `ttgl.atomic_add`, `ttgl.atomic_xchg`, and 5 more to execute the test logic.
- **CN:** 通过 `BLOCK`、`x`、`ptr`、`offs`、`ptrs`、`val`、`mask`、`scalar_mask` 等另外 2 项 准备或更新状态。 调用 `ttgl.full`、`x.cast`、`ttgl.pointer_type`、`ttgl.arange`、`ttgl.atomic_add`、`ttgl.atomic_xchg` 等另外 5 项 执行测试逻辑。

### Lines 1977-1982

```python


@filecheck_test
@gluon.jit
def test_atomic_cas():
    # CHECK: {{.*}} = arith.constant dense<1> : tensor<1xi64, #gluon.auto_encoding>
```
- **EN:** Defines the test function `test_atomic_cas`. Decorators: `filecheck_test`, `gluon.jit`. Key calls include `ttgl.full`, `ttgl.atomic_cas`, `x.cast`, `ttgl.arange`, `ttgl.pointer_type`, `ttgl.AutoLayout`, and 1 more. This scope touches layout transformation reasoning.
- **CN:** 定义测试函数 `test_atomic_cas`。 装饰器：`filecheck_test`、`gluon.jit`。 关键调用包括 `ttgl.full`、`ttgl.atomic_cas`、`x.cast`、`ttgl.arange`、`ttgl.pointer_type`、`ttgl.AutoLayout` 等另外 1 项。 该作用域涉及布局变换推理。

#### Lines 1983-2002

```python
    x0 = ttgl.full([1], 1, ttgl.int64, layout=ttgl.AutoLayout())
    ptr0 = x0.cast(ttgl.pointer_type(ttgl.int32), bitcast=True).item()
    # CHECK: [[c0:%.*]] = arith.constant 0 : i32
    # CHECK: [[c1:%.*]] = arith.constant 1 : i32
    # CHECK: {{.*}} = tt.atomic_cas acq_rel, gpu, %{{.*}}, [[c0]], [[c1]] : (!tt.ptr<i32>, i32, i32) -> i32
    ttgl.atomic_cas(ptr0, 0, 1)

    BLOCK: ttgl.constexpr = 128
    x = ttgl.full([BLOCK], 0, ttgl.int64, layout=ttgl.AutoLayout())
    ptr = x.cast(ttgl.pointer_type(ttgl.int32), bitcast=True)
    # CHECK: {{.*}} = arith.constant dense<0> : tensor<128xi64, #gluon.auto_encoding>
    offset = ttgl.arange(0, BLOCK, layout=ttgl.AutoLayout())
    old = ttgl.full([BLOCK], 0, ttgl.int32, layout=ttgl.AutoLayout())
    new = ttgl.full([BLOCK], 1, ttgl.int32, layout=ttgl.AutoLayout())
    # CHECK: [[old:%.*]] = arith.constant dense<0> : tensor<128xi32, #gluon.auto_encoding>
    # CHECK: [[new:%.*]] = arith.constant dense<1> : tensor<128xi32, #gluon.auto_encoding>
    # CHECK: {{.*}} = tt.atomic_cas relaxed, gpu, %{{.*}}, [[old]], [[new]] : (tensor<128x!tt.ptr<i32>, #gluon.auto_encoding>, tensor<128xi32, #gluon.auto_encoding>, tensor<128xi32, #gluon.auto_encoding>) -> tensor<128xi32, #gluon.auto_encoding>
    # CHECK: {{.*}} = tt.atomic_cas acq_rel, gpu, %{{.*}}, [[old]], [[new]] : (tensor<128x!tt.ptr<i32>, #gluon.auto_encoding>, tensor<128xi32, #gluon.auto_encoding>, tensor<128xi32, #gluon.auto_encoding>) -> tensor<128xi32, #gluon.auto_encoding>
    ttgl.atomic_cas(offset + ptr, old, new, sem="relaxed")
    ttgl.atomic_cas(offset + ptr, old, new)
```
- **EN:** Prepares or updates state through `x0`, `ptr0`, `BLOCK`, `x`, `ptr`, `offset`, `old`, `new`. Invokes `ttgl.full`, `ttgl.AutoLayout`, `x0.cast`, `ttgl.pointer_type`, `ttgl.atomic_cas`, `x.cast`, and 1 more to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `x0`、`ptr0`、`BLOCK`、`x`、`ptr`、`offset`、`old`、`new` 准备或更新状态。 调用 `ttgl.full`、`ttgl.AutoLayout`、`x0.cast`、`ttgl.pointer_type`、`ttgl.atomic_cas`、`x.cast` 等另外 1 项 执行测试逻辑。 相关主题：布局变换推理。

### Lines 2003-2006

```python


@gluon.jit
def amd_mfma_layout_kernel():
```
- **EN:** Defines the helper function `amd_mfma_layout_kernel`. Decorators: `gluon.jit`. Key calls include `ttgl.full`, `amd_layouts.AMDMFMALayout`. This scope touches layout transformation reasoning.
- **CN:** 定义辅助函数 `amd_mfma_layout_kernel`。 装饰器：`gluon.jit`。 关键调用包括 `ttgl.full`、`amd_layouts.AMDMFMALayout`。 该作用域涉及布局变换推理。

#### Lines 2007-2024

```python
    ttgl.full([128, 32], 0, ttgl.float32, layout=amd_layouts.AMDMFMALayout(version=3, instr_shape=[32, 32, 8],
                                                                           transposed=True, warps_per_cta=[4, 1]))

    ttgl.full([128, 32], 0, ttgl.float32, layout=amd_layouts.AMDMFMALayout(version=3, instr_shape=[32, 32,
                                                                                                   8], transposed=True,
                                                                           warps_per_cta=[4, 1], tiles_per_warp=[2, 2]))

    ttgl.full([128, 32], 0, ttgl.float32, layout=amd_layouts.AMDMFMALayout(version=3, instr_shape=[32, 32,
                                                                                                   8], transposed=True,
                                                                           warps_per_cta=[4, 1], tiles_per_warp=[1, 1]))

    ttgl.full([128, 32], 0, ttgl.float64,
              layout=amd_layouts.AMDMFMALayout(version=3, instr_shape=[16, 16, 16], transposed=True,  #
                                               warps_per_cta=[4, 1], element_bitwidth=64, tiles_per_warp=[1, 1]))

    ttgl.full([128, 32], 0, ttgl.int32,
              layout=amd_layouts.AMDMFMALayout(version=3, instr_shape=[16, 16, 16], transposed=True,  #
                                               warps_per_cta=[4, 1], element_bitwidth=32, tiles_per_warp=[1, 1]))
```
- **EN:** Invokes `ttgl.full`, `amd_layouts.AMDMFMALayout` to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 调用 `ttgl.full`、`amd_layouts.AMDMFMALayout` 执行测试逻辑。 相关主题：布局变换推理。

### Lines 2025-2029

```python


@pytest.mark.parametrize("target", [HIP_TARGET_CDNA3, HIP_TARGET_CDNA4])
def test_amd_mfma_layout(target):
```
- **EN:** Defines the test function `test_amd_mfma_layout`. Decorators: `pytest.mark.parametrize('target', [HIP_TARGET_CDNA3, HIP_TARGET_CDNA4])`. Parameters: `target`. Key calls include `pytest.mark.parametrize`, `run_parser`, `expecttest.assert_expected_inline`, `anonymize_ir`, `module.str_nodebug`. This scope touches pytest parametrization, debugging and inspection paths, layout transformation reasoning.
- **CN:** 定义测试函数 `test_amd_mfma_layout`。 装饰器：`pytest.mark.parametrize('target', [HIP_TARGET_CDNA3, HIP_TARGET_CDNA4])`。 参数：`target`。 关键调用包括 `pytest.mark.parametrize`、`run_parser`、`expecttest.assert_expected_inline`、`anonymize_ir`、`module.str_nodebug`。 该作用域涉及pytest 参数化、调试与检查路径、布局变换推理。

#### Lines 2030-2052

```python
    module = run_parser(amd_mfma_layout_kernel, target=target)
    expecttest.assert_expected_inline(
        anonymize_ir(module.str_nodebug()), """\
#mma = #ttg.amd_mfma<{version = 3, warpsPerCTA = [4, 1], instrShape = [32, 32, 8], isTransposed = true}>
#mma1 = #ttg.amd_mfma<{version = 3, warpsPerCTA = [4, 1], instrShape = [32, 32, 8], isTransposed = true, tilesPerWarp = [2, 2]}>
#mma2 = #ttg.amd_mfma<{version = 3, warpsPerCTA = [4, 1], instrShape = [16, 16, 16], isTransposed = true, elementBitWidth = 64}>
#mma3 = #ttg.amd_mfma<{version = 3, warpsPerCTA = [4, 1], instrShape = [16, 16, 16], isTransposed = true}>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "...", "ttg.threads-per-warp" = 64 : i32} {
  tt.func public @amd_mfma_layout_kernel() attributes {noinline = false} {
    %cst = arith.constant 0.000000e+00 : f32
    %cst_0 = arith.constant dense<0.000000e+00> : tensor<128x32xf32, #mma>
    %cst_1 = arith.constant 0.000000e+00 : f32
    %cst_2 = arith.constant dense<0.000000e+00> : tensor<128x32xf32, #mma1>
    %cst_3 = arith.constant 0.000000e+00 : f32
    %cst_4 = arith.constant dense<0.000000e+00> : tensor<128x32xf32, #mma>
    %cst_5 = arith.constant 0.000000e+00 : f64
    %cst_6 = arith.constant dense<0.000000e+00> : tensor<128x32xf64, #mma2>
    %c0_i32 = arith.constant 0 : i32
    %cst_7 = arith.constant dense<0> : tensor<128x32xi32, #mma3>
    tt.return
  }
}
""")
```
- **EN:** Prepares or updates state through `module`. Invokes `run_parser`, `expecttest.assert_expected_inline`, `anonymize_ir`, `module.str_nodebug` to execute the test logic. Relevant themes: debugging and inspection paths, layout transformation reasoning.
- **CN:** 通过 `module` 准备或更新状态。 调用 `run_parser`、`expecttest.assert_expected_inline`、`anonymize_ir`、`module.str_nodebug` 执行测试逻辑。 相关主题：调试与检查路径、布局变换推理。

### Lines 2053-2056

```python


@gluon.jit
def add_int(a, b):
```
- **EN:** Defines the helper function `add_int`. Decorators: `gluon.jit`. Parameters: `a`, `b`.
- **CN:** 定义辅助函数 `add_int`。 装饰器：`gluon.jit`。 参数：`a`、`b`。

#### Lines 2057-2057

```python
    return a + b
```
- **EN:** Carries supporting logic for the surrounding test or helper scope.
- **CN:** 为周围的测试或辅助作用域提供支撑逻辑。

### Lines 2058-2061

```python


@gluon.jit
def infer_layout_for_amd_mfma_kernel():
```
- **EN:** Defines the helper function `infer_layout_for_amd_mfma_kernel`. Decorators: `gluon.jit`. Key calls include `amd_layouts.AMDMFMALayout`, `ttgl.full`, `ttgl.reduce`, `ttgl.static_assert`, `ttgl.SliceLayout`. This scope touches layout transformation reasoning.
- **CN:** 定义辅助函数 `infer_layout_for_amd_mfma_kernel`。 装饰器：`gluon.jit`。 关键调用包括 `amd_layouts.AMDMFMALayout`、`ttgl.full`、`ttgl.reduce`、`ttgl.static_assert`、`ttgl.SliceLayout`。 该作用域涉及布局变换推理。

#### Lines 2062-2066

```python
    layout: ttgl.constexpr = amd_layouts.AMDMFMALayout(version=3, instr_shape=[32, 32, 8], transposed=True,
                                                       warps_per_cta=[4, 1])
    a = ttgl.full([128, 32], 1, ttgl.int32, layout)
    b = ttgl.reduce(a, 1, add_int)
    ttgl.static_assert(b.type.layout == ttgl.SliceLayout(1, layout))
```
- **EN:** Prepares or updates state through `layout`, `a`, `b`. Invokes `amd_layouts.AMDMFMALayout`, `ttgl.full`, `ttgl.reduce`, `ttgl.static_assert`, `ttgl.SliceLayout` to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `layout`、`a`、`b` 准备或更新状态。 调用 `amd_layouts.AMDMFMALayout`、`ttgl.full`、`ttgl.reduce`、`ttgl.static_assert`、`ttgl.SliceLayout` 执行测试逻辑。 相关主题：布局变换推理。

### Lines 2067-2070

```python


@pytest.mark.parametrize("target", [HIP_TARGET_CDNA3, HIP_TARGET_CDNA4])
def test_infer_layout_for_amd_mfma(target):
```
- **EN:** Defines the test function `test_infer_layout_for_amd_mfma`. Decorators: `pytest.mark.parametrize('target', [HIP_TARGET_CDNA3, HIP_TARGET_CDNA4])`. Parameters: `target`. Key calls include `pytest.mark.parametrize`, `run_parser`, `expecttest.assert_expected_inline`, `anonymize_ir`, `module.str_nodebug`. This scope touches pytest parametrization, debugging and inspection paths, layout transformation reasoning.
- **CN:** 定义测试函数 `test_infer_layout_for_amd_mfma`。 装饰器：`pytest.mark.parametrize('target', [HIP_TARGET_CDNA3, HIP_TARGET_CDNA4])`。 参数：`target`。 关键调用包括 `pytest.mark.parametrize`、`run_parser`、`expecttest.assert_expected_inline`、`anonymize_ir`、`module.str_nodebug`。 该作用域涉及pytest 参数化、调试与检查路径、布局变换推理。

#### Lines 2071-2095

```python
    module = run_parser(infer_layout_for_amd_mfma_kernel, target=target)

    expecttest.assert_expected_inline(
        anonymize_ir(module.str_nodebug()), """\
#mma = #ttg.amd_mfma<{version = 3, warpsPerCTA = [4, 1], instrShape = [32, 32, 8], isTransposed = true}>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "...", "ttg.threads-per-warp" = 64 : i32} {
  tt.func public @infer_layout_for_amd_mfma_kernel() attributes {noinline = false} {
    %c1_i32 = arith.constant 1 : i32
    %cst = arith.constant dense<1> : tensor<128x32xi32, #mma>
    %0 = "tt.reduce"(%cst) <{axis = 1 : i32}> ({
    ^bb0(%arg0: i32, %arg1: i32):
      %1 = tt.call @test_frontend.add_int__i32_i32(%arg0, %arg1) : (i32, i32) -> i32
      tt.reduce.return %1 : i32
    }) : (tensor<128x32xi32, #mma>) -> tensor<128xi32, #ttg.slice<{dim = 1, parent = #mma}>>
    tt.return
  }
  tt.func private @test_frontend.add_int__i32_i32(%arg0: i32, %arg1: i32) -> i32 attributes {noinline = false} {
    %0 = arith.addi %arg0, %arg1 : i32
    tt.return %0 : i32
  ^bb1:  // no predecessors
    %1 = ub.poison : i32
    tt.return %1 : i32
  }
}
""")
```
- **EN:** Prepares or updates state through `module`. Invokes `run_parser`, `expecttest.assert_expected_inline`, `anonymize_ir`, `module.str_nodebug` to execute the test logic. Relevant themes: debugging and inspection paths, layout transformation reasoning.
- **CN:** 通过 `module` 准备或更新状态。 调用 `run_parser`、`expecttest.assert_expected_inline`、`anonymize_ir`、`module.str_nodebug` 执行测试逻辑。 相关主题：调试与检查路径、布局变换推理。

### Lines 2096-2099

```python


@gluon.jit
def amd_wmma_layout_kernel():
```
- **EN:** Defines the helper function `amd_wmma_layout_kernel`. Decorators: `gluon.jit`. Key calls include `ttgl.full`, `amd_layouts.AMDWMMALayout`. This scope touches layout transformation reasoning.
- **CN:** 定义辅助函数 `amd_wmma_layout_kernel`。 装饰器：`gluon.jit`。 关键调用包括 `ttgl.full`、`amd_layouts.AMDWMMALayout`。 该作用域涉及布局变换推理。

#### Lines 2100-2107

```python
    ttgl.full([64, 64], 0, ttgl.float16, layout=amd_layouts.AMDWMMALayout(version=2, warp_bases=[[0, 1], [0, 2]],
                                                                          transposed=True))
    ttgl.full([64, 64], 0, ttgl.float16, layout=amd_layouts.AMDWMMALayout(version=2, transposed=True,
                                                                          warp_bases=[[0, 1], [1, 0]]))
    ttgl.full([64, 64], 0, ttgl.float16, layout=amd_layouts.AMDWMMALayout(version=2, transposed=False,
                                                                          warp_bases=[[0, 1], [0, 2]]))
    ttgl.full([64, 64], 0, ttgl.float16, layout=amd_layouts.AMDWMMALayout(version=2, transposed=False,
                                                                          warp_bases=[[0, 1], [1, 0]]))
```
- **EN:** Invokes `ttgl.full`, `amd_layouts.AMDWMMALayout` to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 调用 `ttgl.full`、`amd_layouts.AMDWMMALayout` 执行测试逻辑。 相关主题：布局变换推理。

### Lines 2108-2111

```python


@pytest.mark.parametrize("target", [HIP_TARGET_RDNA4])
def test_amd_wmma_layout(target):
```
- **EN:** Defines the test function `test_amd_wmma_layout`. Decorators: `pytest.mark.parametrize('target', [HIP_TARGET_RDNA4])`. Parameters: `target`. Key calls include `pytest.mark.parametrize`, `run_parser`, `expecttest.assert_expected_inline`, `anonymize_ir`, `module.str_nodebug`. This scope touches pytest parametrization, debugging and inspection paths, layout transformation reasoning.
- **CN:** 定义测试函数 `test_amd_wmma_layout`。 装饰器：`pytest.mark.parametrize('target', [HIP_TARGET_RDNA4])`。 参数：`target`。 关键调用包括 `pytest.mark.parametrize`、`run_parser`、`expecttest.assert_expected_inline`、`anonymize_ir`、`module.str_nodebug`。 该作用域涉及pytest 参数化、调试与检查路径、布局变换推理。

#### Lines 2112-2132

```python
    module = run_parser(amd_wmma_layout_kernel, target=target)
    expecttest.assert_expected_inline(
        anonymize_ir(module.str_nodebug()), """\
#mma = #ttg.amd_wmma<{version = 2, isTranspose = true, ctaLayout = {warp = [[0, 1], [0, 2]]}}>
#mma1 = #ttg.amd_wmma<{version = 2, isTranspose = true, ctaLayout = {warp = [[0, 1], [1, 0]]}}>
#mma2 = #ttg.amd_wmma<{version = 2, isTranspose = false, ctaLayout = {warp = [[0, 1], [0, 2]]}}>
#mma3 = #ttg.amd_wmma<{version = 2, isTranspose = false, ctaLayout = {warp = [[0, 1], [1, 0]]}}>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "...", "ttg.threads-per-warp" = 32 : i32} {
  tt.func public @amd_wmma_layout_kernel() attributes {noinline = false} {
    %cst = arith.constant 0.000000e+00 : f16
    %cst_0 = arith.constant dense<0.000000e+00> : tensor<64x64xf16, #mma>
    %cst_1 = arith.constant 0.000000e+00 : f16
    %cst_2 = arith.constant dense<0.000000e+00> : tensor<64x64xf16, #mma1>
    %cst_3 = arith.constant 0.000000e+00 : f16
    %cst_4 = arith.constant dense<0.000000e+00> : tensor<64x64xf16, #mma2>
    %cst_5 = arith.constant 0.000000e+00 : f16
    %cst_6 = arith.constant dense<0.000000e+00> : tensor<64x64xf16, #mma3>
    tt.return
  }
}
""")
```
- **EN:** Prepares or updates state through `module`. Invokes `run_parser`, `expecttest.assert_expected_inline`, `anonymize_ir`, `module.str_nodebug` to execute the test logic. Relevant themes: debugging and inspection paths, layout transformation reasoning.
- **CN:** 通过 `module` 准备或更新状态。 调用 `run_parser`、`expecttest.assert_expected_inline`、`anonymize_ir`、`module.str_nodebug` 执行测试逻辑。 相关主题：调试与检查路径、布局变换推理。

### Lines 2133-2136

```python


@gluon.jit
def infer_layout_for_amd_wmma_kernel():
```
- **EN:** Defines the helper function `infer_layout_for_amd_wmma_kernel`. Decorators: `gluon.jit`. Key calls include `amd_layouts.AMDWMMALayout`, `ttgl.full`, `ttgl.reduce`, `ttgl.static_assert`, `ttgl.SliceLayout`. This scope touches layout transformation reasoning.
- **CN:** 定义辅助函数 `infer_layout_for_amd_wmma_kernel`。 装饰器：`gluon.jit`。 关键调用包括 `amd_layouts.AMDWMMALayout`、`ttgl.full`、`ttgl.reduce`、`ttgl.static_assert`、`ttgl.SliceLayout`。 该作用域涉及布局变换推理。

#### Lines 2137-2140

```python
    layout: ttgl.constexpr = amd_layouts.AMDWMMALayout(version=2, transposed=True, warp_bases=[[1, 0], [2, 0]])
    a = ttgl.full([128, 32], 1, ttgl.float16, layout)
    b = ttgl.reduce(a, 1, add_int)
    ttgl.static_assert(b.type.layout == ttgl.SliceLayout(1, layout))
```
- **EN:** Prepares or updates state through `layout`, `a`, `b`. Invokes `amd_layouts.AMDWMMALayout`, `ttgl.full`, `ttgl.reduce`, `ttgl.static_assert`, `ttgl.SliceLayout` to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `layout`、`a`、`b` 准备或更新状态。 调用 `amd_layouts.AMDWMMALayout`、`ttgl.full`、`ttgl.reduce`、`ttgl.static_assert`、`ttgl.SliceLayout` 执行测试逻辑。 相关主题：布局变换推理。

### Lines 2141-2144

```python


@pytest.mark.parametrize("target", [HIP_TARGET_RDNA4])
def test_infer_layout_for_amd_wmma(target):
```
- **EN:** Defines the test function `test_infer_layout_for_amd_wmma`. Decorators: `pytest.mark.parametrize('target', [HIP_TARGET_RDNA4])`. Parameters: `target`. Key calls include `pytest.mark.parametrize`, `run_parser`, `expecttest.assert_expected_inline`, `anonymize_ir`, `module.str_nodebug`. This scope touches pytest parametrization, debugging and inspection paths, layout transformation reasoning.
- **CN:** 定义测试函数 `test_infer_layout_for_amd_wmma`。 装饰器：`pytest.mark.parametrize('target', [HIP_TARGET_RDNA4])`。 参数：`target`。 关键调用包括 `pytest.mark.parametrize`、`run_parser`、`expecttest.assert_expected_inline`、`anonymize_ir`、`module.str_nodebug`。 该作用域涉及pytest 参数化、调试与检查路径、布局变换推理。

#### Lines 2145-2168

```python
    module = run_parser(infer_layout_for_amd_wmma_kernel, target=target)
    expecttest.assert_expected_inline(
        anonymize_ir(module.str_nodebug()), """\
#mma = #ttg.amd_wmma<{version = 2, isTranspose = true, ctaLayout = {warp = [[1, 0], [2, 0]]}}>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "...", "ttg.threads-per-warp" = 32 : i32} {
  tt.func public @infer_layout_for_amd_wmma_kernel() attributes {noinline = false} {
    %cst = arith.constant 1.000000e+00 : f16
    %cst_0 = arith.constant dense<1.000000e+00> : tensor<128x32xf16, #mma>
    %0 = "tt.reduce"(%cst_0) <{axis = 1 : i32}> ({
    ^bb0(%arg0: f16, %arg1: f16):
      %1 = tt.call @test_frontend.add_int__fp16_fp16(%arg0, %arg1) : (f16, f16) -> f16
      tt.reduce.return %1 : f16
    }) : (tensor<128x32xf16, #mma>) -> tensor<128xf16, #ttg.slice<{dim = 1, parent = #mma}>>
    tt.return
  }
  tt.func private @test_frontend.add_int__fp16_fp16(%arg0: f16, %arg1: f16) -> f16 attributes {noinline = false} {
    %0 = arith.addf %arg0, %arg1 : f16
    tt.return %0 : f16
  ^bb1:  // no predecessors
    %1 = ub.poison : f16
    tt.return %1 : f16
  }
}
""")
```
- **EN:** Prepares or updates state through `module`. Invokes `run_parser`, `expecttest.assert_expected_inline`, `anonymize_ir`, `module.str_nodebug` to execute the test logic. Relevant themes: debugging and inspection paths, layout transformation reasoning.
- **CN:** 通过 `module` 准备或更新状态。 调用 `run_parser`、`expecttest.assert_expected_inline`、`anonymize_ir`、`module.str_nodebug` 执行测试逻辑。 相关主题：调试与检查路径、布局变换推理。

### Lines 2169-2175

```python


@pytest.mark.parametrize("num_warps", [4, 8])
@pytest.mark.parametrize("block_m,block_n", [(64, 64), (64, 128), (128, 128)])
@pytest.mark.parametrize("a_transposed", [False, True])
@pytest.mark.parametrize("b_transposed", [False, True])
def test_make_partitioned_dot_layouts(num_warps, block_m, block_n, a_transposed, b_transposed):
```
- **EN:** Defines the test function `test_make_partitioned_dot_layouts`. Decorators: `pytest.mark.parametrize('num_warps', [4, 8])`, `pytest.mark.parametrize('block_m,block_n', [(64, 64), (64, 128), (128, 128)])`, `pytest.mark.parametrize('a_transposed', [False, True])`, `pytest.mark.parametrize('b_transposed', [False, True])`. Parameters: `num_warps`, `block_m`, `block_n`, `a_transposed`, `b_transposed`. Key calls include `pytest.mark.parametrize`, `ttgl.PaddedSharedLayout.with_identity_for`, `make_partitioned_dot_layouts`, `isinstance`. This scope touches pytest parametrization, layout transformation reasoning.
- **CN:** 定义测试函数 `test_make_partitioned_dot_layouts`。 装饰器：`pytest.mark.parametrize('num_warps', [4, 8])`、`pytest.mark.parametrize('block_m,block_n', [(64, 64), (64, 128), (128, 128)])`、`pytest.mark.parametrize('a_transposed', [False, True])`、`pytest.mark.parametrize('b_transposed', [False, True])`。 参数：`num_warps`、`block_m`、`block_n`、`a_transposed`、`b_transposed`。 关键调用包括 `pytest.mark.parametrize`、`ttgl.PaddedSharedLayout.with_identity_for`、`make_partitioned_dot_layouts`、`isinstance`。 该作用域涉及pytest 参数化、布局变换推理。

#### Lines 2176-2198

```python
    block_k = 32
    INSTR_M, INSTR_N, INSTR_K = 16, 16, 32
    # WARP_TILES_M=4 and WARP_TILES_N=2 for both 4- and 8-warp cases (different
    # warp/reg base layouts but same per-warp tile extents).
    warp_coverage_m = 4 * INSTR_M
    warp_coverage_n = 2 * INSTR_N

    a_shape = [block_k, block_m] if a_transposed else [block_m, block_k]
    b_shape = [block_n, block_k] if b_transposed else [block_k, block_n]
    pa = ttgl.PaddedSharedLayout.with_identity_for([[a_shape[1], 8]], a_shape, [1, 0])
    pb = ttgl.PaddedSharedLayout.with_identity_for([[b_shape[1], 8]], b_shape, [1, 0])
    sla, slb, wmma = make_partitioned_dot_layouts(block_m, block_n, pa, pb, num_warps=num_warps,
                                                  instr_shape=[INSTR_M, INSTR_N, INSTR_K], a_transposed=a_transposed,
                                                  b_transposed=b_transposed)

    a_num_groups = block_m // warp_coverage_m
    b_num_groups = block_n // warp_coverage_n
    a_partition_dim = 1 if a_transposed else 0
    b_partition_dim = 0 if b_transposed else 1
    # The non-partitioned axis of each piece is block_k, the partitioned axis is
    # divided down to (block / num_partitions / num_groups).
    a_inner = [block_m / 2 / a_num_groups, block_k]
    b_inner = [block_n / 2 / b_num_groups, block_k] if b_transposed else [block_k, block_n / 2 / b_num_groups]
```
- **EN:** Prepares or updates state through `block_k`, `INSTR_M`, `INSTR_N`, `INSTR_K`, `warp_coverage_m`, `warp_coverage_n`, `a_shape`, `b_shape`, and 11 more. Invokes `ttgl.PaddedSharedLayout.with_identity_for`, `make_partitioned_dot_layouts` to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `block_k`、`INSTR_M`、`INSTR_N`、`INSTR_K`、`warp_coverage_m`、`warp_coverage_n`、`a_shape`、`b_shape` 等另外 11 项 准备或更新状态。 调用 `ttgl.PaddedSharedLayout.with_identity_for`、`make_partitioned_dot_layouts` 执行测试逻辑。 相关主题：布局变换推理。

#### Lines 2199-2207

```python
    for layout, num_groups, partition_dim, inner_shape in [
        (sla, a_num_groups, a_partition_dim, a_inner),
        (slb, b_num_groups, b_partition_dim, b_inner),
    ]:
        assert isinstance(layout, PartitionedSharedLayout)
        assert layout.num_partitions == 2
        assert layout.num_groups == num_groups
        assert layout.partition_dim == partition_dim
        assert layout.partition_layout.shape == inner_shape
```
- **EN:** Invokes `isinstance` to execute the test logic. Validates behavior with 5 assertion(s). Iterates across cases or data tiles. Relevant themes: layout transformation reasoning.
- **CN:** 调用 `isinstance` 执行测试逻辑。 通过 5 个断言验证行为。 通过循环覆盖多个用例或数据分块。 相关主题：布局变换推理。

#### Lines 2208-2212

```python

    if num_warps == 4:
        expected_warp_bases, expected_reg_bases = [[2, 1], [1, 0]], [[2, 0]]
    else:
        expected_warp_bases, expected_reg_bases = [[2, 1], [1, 0], [2, 0]], []
```
- **EN:** Branches on runtime or test conditions.
- **CN:** 根据运行时或测试条件进行分支。

#### Lines 2213-2216

```python
    assert isinstance(wmma, amd_layouts.AMDWMMALayout)
    assert wmma.warp_bases == expected_warp_bases
    assert wmma.reg_bases == expected_reg_bases
    assert wmma.instr_shape == [INSTR_M, INSTR_N, INSTR_K]
```
- **EN:** Invokes `isinstance` to execute the test logic. Validates behavior with 4 assertion(s). Relevant themes: layout transformation reasoning.
- **CN:** 调用 `isinstance` 执行测试逻辑。 通过 4 个断言验证行为。 相关主题：布局变换推理。

### Lines 2217-2220

```python


@gluon.jit
def amd_async_copy_global_to_shared(ptr):
```
- **EN:** Defines the helper function `amd_async_copy_global_to_shared`. Decorators: `gluon.jit`. Parameters: `ptr`. Key calls include `ttgl.BlockedLayout`, `ttgl.SwizzledSharedLayout`, `ttgl.allocate_shared_memory`, `ttgl.arange`, `gfx1250_async_copy.global_to_shared`, `ttgl.full`, and 2 more. This scope touches layout transformation reasoning.
- **CN:** 定义辅助函数 `amd_async_copy_global_to_shared`。 装饰器：`gluon.jit`。 参数：`ptr`。 关键调用包括 `ttgl.BlockedLayout`、`ttgl.SwizzledSharedLayout`、`ttgl.allocate_shared_memory`、`ttgl.arange`、`gfx1250_async_copy.global_to_shared`、`ttgl.full` 等另外 2 项。 该作用域涉及布局变换推理。

#### Lines 2221-2243

```python
    blocked: ttgl.constexpr = ttgl.BlockedLayout([1, 8], [32, 1], [4, 1], [1, 0])
    shared: ttgl.constexpr = ttgl.SwizzledSharedLayout(1, 1, 1, order=[1, 0])

    smem = ttgl.allocate_shared_memory(ptr.dtype.element_ty, [128, 16], shared)
    y_offset = ttgl.arange(0, 128, layout=ttgl.SliceLayout(1, blocked))
    x_offset = ttgl.arange(0, 16, layout=ttgl.SliceLayout(0, blocked))
    offsets = y_offset[:, None] * 16 + x_offset[None, :]

    # test default parameters
    gfx1250_async_copy.global_to_shared(smem, ptr + offsets)

    # test mask
    mask = (y_offset < 64)[:, None]
    gfx1250_async_copy.global_to_shared(smem, ptr + offsets, mask)

    # Test other with scalar
    gfx1250_async_copy.global_to_shared(smem, ptr + offsets, mask, other=0.0)

    # Test other with tensor
    other = ttgl.full([128, 16], 0.0, ptr.dtype.element_ty, layout=blocked)
    gfx1250_async_copy.global_to_shared(smem, ptr + offsets, mask, other)

    gfx1250_async_copy.commit_group()
```
- **EN:** Prepares or updates state through `blocked`, `shared`, `smem`, `y_offset`, `x_offset`, `offsets`, `mask`, `other`. Invokes `ttgl.BlockedLayout`, `ttgl.SwizzledSharedLayout`, `ttgl.allocate_shared_memory`, `ttgl.arange`, `ttgl.SliceLayout`, `gfx1250_async_copy.global_to_shared`, and 2 more to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `blocked`、`shared`、`smem`、`y_offset`、`x_offset`、`offsets`、`mask`、`other` 准备或更新状态。 调用 `ttgl.BlockedLayout`、`ttgl.SwizzledSharedLayout`、`ttgl.allocate_shared_memory`、`ttgl.arange`、`ttgl.SliceLayout`、`gfx1250_async_copy.global_to_shared` 等另外 2 项 执行测试逻辑。 相关主题：布局变换推理。

### Lines 2244-2247

```python


@pytest.mark.parametrize("target", [HIP_TARGET_GFX1250])
def test_amd_async_copy_global_to_shared(target):
```
- **EN:** Defines the test function `test_amd_async_copy_global_to_shared`. Decorators: `pytest.mark.parametrize('target', [HIP_TARGET_GFX1250])`. Parameters: `target`. Key calls include `pytest.mark.parametrize`, `MockTensor`, `run_parser`, `expecttest.assert_expected_inline`, `anonymize_ir`, `make_args`, and 1 more. This scope touches pytest parametrization, debugging and inspection paths.
- **CN:** 定义测试函数 `test_amd_async_copy_global_to_shared`。 装饰器：`pytest.mark.parametrize('target', [HIP_TARGET_GFX1250])`。 参数：`target`。 关键调用包括 `pytest.mark.parametrize`、`MockTensor`、`run_parser`、`expecttest.assert_expected_inline`、`anonymize_ir`、`make_args` 等另外 1 项。 该作用域涉及pytest 参数化、调试与检查路径。

#### Lines 2248-2297

```python
    ptr = MockTensor(ttgl.float16)
    mod = run_parser(amd_async_copy_global_to_shared, *make_args(ptr), target=target)
    expecttest.assert_expected_inline(
        anonymize_ir(mod.str_nodebug()), """\
#blocked = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [32, 1], warpsPerCTA = [4, 1], order = [1, 0]}>
#shared = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [1, 0]}>
#smem = #ttg.shared_memory
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "...", "ttg.threads-per-warp" = 32 : i32} {
  tt.func public @amd_async_copy_global_to_shared(%arg0: !tt.ptr<f16> {tt.divisibility = 16 : i32}) attributes {noinline = false} {
    %0 = ttg.local_alloc : () -> !ttg.memdesc<128x16xf16, #shared, #smem, mutable>
    %1 = tt.make_range {end = 128 : i32, start = 0 : i32} : tensor<128xi32, #ttg.slice<{dim = 1, parent = #blocked}>>
    %2 = tt.make_range {end = 16 : i32, start = 0 : i32} : tensor<16xi32, #ttg.slice<{dim = 0, parent = #blocked}>>
    %3 = tt.expand_dims %1 {axis = 1 : i32} : tensor<128xi32, #ttg.slice<{dim = 1, parent = #blocked}>> -> tensor<128x1xi32, #blocked>
    %c16_i32 = arith.constant 16 : i32
    %c16_i32_0 = arith.constant 16 : i32
    %cst = arith.constant dense<16> : tensor<128x1xi32, #blocked>
    %4 = arith.muli %3, %cst : tensor<128x1xi32, #blocked>
    %5 = tt.expand_dims %2 {axis = 0 : i32} : tensor<16xi32, #ttg.slice<{dim = 0, parent = #blocked}>> -> tensor<1x16xi32, #blocked>
    %6 = tt.broadcast %4 : tensor<128x1xi32, #blocked> -> tensor<128x16xi32, #blocked>
    %7 = tt.broadcast %5 : tensor<1x16xi32, #blocked> -> tensor<128x16xi32, #blocked>
    %8 = arith.addi %6, %7 : tensor<128x16xi32, #blocked>
    %9 = tt.splat %arg0 : !tt.ptr<f16> -> tensor<128x16x!tt.ptr<f16>, #blocked>
    %10 = tt.addptr %9, %8 : tensor<128x16x!tt.ptr<f16>, #blocked>, tensor<128x16xi32, #blocked>
    %11 = ttg.async_copy_global_to_local %10, %0 : tensor<128x16x!tt.ptr<f16>, #blocked> -> <128x16xf16, #shared, #smem, mutable>
    %c64_i32 = arith.constant 64 : i32
    %cst_1 = arith.constant dense<64> : tensor<128xi32, #ttg.slice<{dim = 1, parent = #blocked}>>
    %12 = arith.cmpi slt, %1, %cst_1 : tensor<128xi32, #ttg.slice<{dim = 1, parent = #blocked}>>
    %13 = tt.expand_dims %12 {axis = 1 : i32} : tensor<128xi1, #ttg.slice<{dim = 1, parent = #blocked}>> -> tensor<128x1xi1, #blocked>
    %14 = tt.splat %arg0 : !tt.ptr<f16> -> tensor<128x16x!tt.ptr<f16>, #blocked>
    %15 = tt.addptr %14, %8 : tensor<128x16x!tt.ptr<f16>, #blocked>, tensor<128x16xi32, #blocked>
    %16 = tt.broadcast %13 : tensor<128x1xi1, #blocked> -> tensor<128x16xi1, #blocked>
    %17 = ttg.async_copy_global_to_local %15, %0 mask %16 : tensor<128x16x!tt.ptr<f16>, #blocked> -> <128x16xf16, #shared, #smem, mutable>
    %18 = tt.splat %arg0 : !tt.ptr<f16> -> tensor<128x16x!tt.ptr<f16>, #blocked>
    %19 = tt.addptr %18, %8 : tensor<128x16x!tt.ptr<f16>, #blocked>, tensor<128x16xi32, #blocked>
    %20 = tt.broadcast %13 : tensor<128x1xi1, #blocked> -> tensor<128x16xi1, #blocked>
    %cst_2 = arith.constant 0.000000e+00 : f32
    %21 = arith.truncf %cst_2 : f32 to f16
    %22 = tt.splat %21 : f16 -> tensor<128x16xf16, #blocked>
    %23 = ttg.async_copy_global_to_local %19, %0 mask %20 other %22 : tensor<128x16x!tt.ptr<f16>, #blocked> -> <128x16xf16, #shared, #smem, mutable>
    %cst_3 = arith.constant 0.000000e+00 : f16
    %cst_4 = arith.constant dense<0.000000e+00> : tensor<128x16xf16, #blocked>
    %24 = tt.splat %arg0 : !tt.ptr<f16> -> tensor<128x16x!tt.ptr<f16>, #blocked>
    %25 = tt.addptr %24, %8 : tensor<128x16x!tt.ptr<f16>, #blocked>, tensor<128x16xi32, #blocked>
    %26 = tt.broadcast %13 : tensor<128x1xi1, #blocked> -> tensor<128x16xi1, #blocked>
    %27 = ttg.async_copy_global_to_local %25, %0 mask %26 other %cst_4 : tensor<128x16x!tt.ptr<f16>, #blocked> -> <128x16xf16, #shared, #smem, mutable>
    %28 = ttg.async_commit_group
    tt.return
  }
}
""")
```
- **EN:** Prepares or updates state through `ptr`, `mod`. Invokes `MockTensor`, `run_parser`, `make_args`, `expecttest.assert_expected_inline`, `anonymize_ir`, `mod.str_nodebug` to execute the test logic. Relevant themes: debugging and inspection paths.
- **CN:** 通过 `ptr`、`mod` 准备或更新状态。 调用 `MockTensor`、`run_parser`、`make_args`、`expecttest.assert_expected_inline`、`anonymize_ir`、`mod.str_nodebug` 执行测试逻辑。 相关主题：调试与检查路径。

### Lines 2298-2301

```python


@gluon.jit
def amd_async_copy_shared_to_global(ptr):
```
- **EN:** Defines the helper function `amd_async_copy_shared_to_global`. Decorators: `gluon.jit`. Parameters: `ptr`. Key calls include `ttgl.BlockedLayout`, `ttgl.SwizzledSharedLayout`, `ttgl.allocate_shared_memory`, `ttgl.arange`, `gfx1250_async_copy.shared_to_global`, `gfx1250_async_copy.commit_group`, and 1 more. This scope touches layout transformation reasoning.
- **CN:** 定义辅助函数 `amd_async_copy_shared_to_global`。 装饰器：`gluon.jit`。 参数：`ptr`。 关键调用包括 `ttgl.BlockedLayout`、`ttgl.SwizzledSharedLayout`、`ttgl.allocate_shared_memory`、`ttgl.arange`、`gfx1250_async_copy.shared_to_global`、`gfx1250_async_copy.commit_group` 等另外 1 项。 该作用域涉及布局变换推理。

#### Lines 2302-2317

```python
    blocked: ttgl.constexpr = ttgl.BlockedLayout([1, 8], [32, 1], [4, 1], [1, 0])
    shared: ttgl.constexpr = ttgl.SwizzledSharedLayout(1, 1, 1, order=[1, 0])

    smem = ttgl.allocate_shared_memory(ptr.dtype.element_ty, [128, 16], shared)
    y_offset = ttgl.arange(0, 128, layout=ttgl.SliceLayout(1, blocked))
    x_offset = ttgl.arange(0, 16, layout=ttgl.SliceLayout(0, blocked))
    offsets = y_offset[:, None] * 16 + x_offset[None, :]

    # test default parameters
    gfx1250_async_copy.shared_to_global(ptr + offsets, smem)

    # test mask
    mask = (y_offset < 64)[:, None]
    gfx1250_async_copy.shared_to_global(ptr + offsets, smem, mask)

    gfx1250_async_copy.commit_group()
```
- **EN:** Prepares or updates state through `blocked`, `shared`, `smem`, `y_offset`, `x_offset`, `offsets`, `mask`. Invokes `ttgl.BlockedLayout`, `ttgl.SwizzledSharedLayout`, `ttgl.allocate_shared_memory`, `ttgl.arange`, `ttgl.SliceLayout`, `gfx1250_async_copy.shared_to_global`, and 1 more to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `blocked`、`shared`、`smem`、`y_offset`、`x_offset`、`offsets`、`mask` 准备或更新状态。 调用 `ttgl.BlockedLayout`、`ttgl.SwizzledSharedLayout`、`ttgl.allocate_shared_memory`、`ttgl.arange`、`ttgl.SliceLayout`、`gfx1250_async_copy.shared_to_global` 等另外 1 项 执行测试逻辑。 相关主题：布局变换推理。

### Lines 2318-2321

```python


@pytest.mark.parametrize("target", [HIP_TARGET_GFX1250])
def test_amd_async_copy_shared_to_global(target):
```
- **EN:** Defines the test function `test_amd_async_copy_shared_to_global`. Decorators: `pytest.mark.parametrize('target', [HIP_TARGET_GFX1250])`. Parameters: `target`. Key calls include `pytest.mark.parametrize`, `MockTensor`, `run_parser`, `expecttest.assert_expected_inline`, `anonymize_ir`, `make_args`, and 1 more. This scope touches pytest parametrization, debugging and inspection paths.
- **CN:** 定义测试函数 `test_amd_async_copy_shared_to_global`。 装饰器：`pytest.mark.parametrize('target', [HIP_TARGET_GFX1250])`。 参数：`target`。 关键调用包括 `pytest.mark.parametrize`、`MockTensor`、`run_parser`、`expecttest.assert_expected_inline`、`anonymize_ir`、`make_args` 等另外 1 项。 该作用域涉及pytest 参数化、调试与检查路径。

#### Lines 2322-2358

```python
    ptr = MockTensor(ttgl.float16)
    mod = run_parser(amd_async_copy_shared_to_global, *make_args(ptr), target=target)
    expecttest.assert_expected_inline(
        anonymize_ir(mod.str_nodebug()), """\
#blocked = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [32, 1], warpsPerCTA = [4, 1], order = [1, 0]}>
#shared = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [1, 0]}>
#smem = #ttg.shared_memory
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "...", "ttg.threads-per-warp" = 32 : i32} {
  tt.func public @amd_async_copy_shared_to_global(%arg0: !tt.ptr<f16> {tt.divisibility = 16 : i32}) attributes {noinline = false} {
    %0 = ttg.local_alloc : () -> !ttg.memdesc<128x16xf16, #shared, #smem, mutable>
    %1 = tt.make_range {end = 128 : i32, start = 0 : i32} : tensor<128xi32, #ttg.slice<{dim = 1, parent = #blocked}>>
    %2 = tt.make_range {end = 16 : i32, start = 0 : i32} : tensor<16xi32, #ttg.slice<{dim = 0, parent = #blocked}>>
    %3 = tt.expand_dims %1 {axis = 1 : i32} : tensor<128xi32, #ttg.slice<{dim = 1, parent = #blocked}>> -> tensor<128x1xi32, #blocked>
    %c16_i32 = arith.constant 16 : i32
    %c16_i32_0 = arith.constant 16 : i32
    %cst = arith.constant dense<16> : tensor<128x1xi32, #blocked>
    %4 = arith.muli %3, %cst : tensor<128x1xi32, #blocked>
    %5 = tt.expand_dims %2 {axis = 0 : i32} : tensor<16xi32, #ttg.slice<{dim = 0, parent = #blocked}>> -> tensor<1x16xi32, #blocked>
    %6 = tt.broadcast %4 : tensor<128x1xi32, #blocked> -> tensor<128x16xi32, #blocked>
    %7 = tt.broadcast %5 : tensor<1x16xi32, #blocked> -> tensor<128x16xi32, #blocked>
    %8 = arith.addi %6, %7 : tensor<128x16xi32, #blocked>
    %9 = tt.splat %arg0 : !tt.ptr<f16> -> tensor<128x16x!tt.ptr<f16>, #blocked>
    %10 = tt.addptr %9, %8 : tensor<128x16x!tt.ptr<f16>, #blocked>, tensor<128x16xi32, #blocked>
    %11 = amdg.async_copy_local_to_global %0, %10 : !ttg.memdesc<128x16xf16, #shared, #smem, mutable> -> tensor<128x16x!tt.ptr<f16>, #blocked>
    %c64_i32 = arith.constant 64 : i32
    %cst_1 = arith.constant dense<64> : tensor<128xi32, #ttg.slice<{dim = 1, parent = #blocked}>>
    %12 = arith.cmpi slt, %1, %cst_1 : tensor<128xi32, #ttg.slice<{dim = 1, parent = #blocked}>>
    %13 = tt.expand_dims %12 {axis = 1 : i32} : tensor<128xi1, #ttg.slice<{dim = 1, parent = #blocked}>> -> tensor<128x1xi1, #blocked>
    %14 = tt.splat %arg0 : !tt.ptr<f16> -> tensor<128x16x!tt.ptr<f16>, #blocked>
    %15 = tt.addptr %14, %8 : tensor<128x16x!tt.ptr<f16>, #blocked>, tensor<128x16xi32, #blocked>
    %16 = tt.broadcast %13 : tensor<128x1xi1, #blocked> -> tensor<128x16xi1, #blocked>
    %17 = amdg.async_copy_local_to_global %0, %15 mask %16 : !ttg.memdesc<128x16xf16, #shared, #smem, mutable> -> tensor<128x16x!tt.ptr<f16>, #blocked>
    %18 = ttg.async_commit_group
    tt.return
  }
}
""")
```
- **EN:** Prepares or updates state through `ptr`, `mod`. Invokes `MockTensor`, `run_parser`, `make_args`, `expecttest.assert_expected_inline`, `anonymize_ir`, `mod.str_nodebug` to execute the test logic. Relevant themes: debugging and inspection paths.
- **CN:** 通过 `ptr`、`mod` 准备或更新状态。 调用 `MockTensor`、`run_parser`、`make_args`、`expecttest.assert_expected_inline`、`anonymize_ir`、`mod.str_nodebug` 执行测试逻辑。 相关主题：调试与检查路径。

### Lines 2359-2362

```python


@gluon.jit
def amd_commit_group():
```
- **EN:** Defines the helper function `amd_commit_group`. Decorators: `gluon.jit`. Key calls include `cdna4_async_copy.commit_group`.
- **CN:** 定义辅助函数 `amd_commit_group`。 装饰器：`gluon.jit`。 关键调用包括 `cdna4_async_copy.commit_group`。

#### Lines 2363-2363

```python
    cdna4_async_copy.commit_group()
```
- **EN:** Invokes `cdna4_async_copy.commit_group` to execute the test logic.
- **CN:** 调用 `cdna4_async_copy.commit_group` 执行测试逻辑。

### Lines 2364-2367

```python


@pytest.mark.parametrize("target", [HIP_TARGET_CDNA4])
def test_amd_commit_group(target):
```
- **EN:** Defines the test function `test_amd_commit_group`. Decorators: `pytest.mark.parametrize('target', [HIP_TARGET_CDNA4])`. Parameters: `target`. Key calls include `pytest.mark.parametrize`, `run_parser`, `expecttest.assert_expected_inline`, `anonymize_ir`, `mod.str_nodebug`. This scope touches pytest parametrization, debugging and inspection paths.
- **CN:** 定义测试函数 `test_amd_commit_group`。 装饰器：`pytest.mark.parametrize('target', [HIP_TARGET_CDNA4])`。 参数：`target`。 关键调用包括 `pytest.mark.parametrize`、`run_parser`、`expecttest.assert_expected_inline`、`anonymize_ir`、`mod.str_nodebug`。 该作用域涉及pytest 参数化、调试与检查路径。

#### Lines 2368-2377

```python
    mod = run_parser(amd_wait_group, target=target)
    expecttest.assert_expected_inline(
        anonymize_ir(mod.str_nodebug()), """\
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "...", "ttg.threads-per-warp" = 64 : i32} {
  tt.func public @amd_wait_group() attributes {noinline = false} {
    %0 = ttg.async_wait {num = 0 : i32}
    tt.return
  }
}
""")
```
- **EN:** Prepares or updates state through `mod`. Invokes `run_parser`, `expecttest.assert_expected_inline`, `anonymize_ir`, `mod.str_nodebug` to execute the test logic. Relevant themes: debugging and inspection paths.
- **CN:** 通过 `mod` 准备或更新状态。 调用 `run_parser`、`expecttest.assert_expected_inline`、`anonymize_ir`、`mod.str_nodebug` 执行测试逻辑。 相关主题：调试与检查路径。

### Lines 2378-2381

```python


@gluon.jit
def amd_wait_group():
```
- **EN:** Defines the helper function `amd_wait_group`. Decorators: `gluon.jit`. Key calls include `cdna4_async_copy.wait_group`.
- **CN:** 定义辅助函数 `amd_wait_group`。 装饰器：`gluon.jit`。 关键调用包括 `cdna4_async_copy.wait_group`。

#### Lines 2382-2382

```python
    cdna4_async_copy.wait_group(0)
```
- **EN:** Invokes `cdna4_async_copy.wait_group` to execute the test logic.
- **CN:** 调用 `cdna4_async_copy.wait_group` 执行测试逻辑。

### Lines 2383-2386

```python


@pytest.mark.parametrize("target", [HIP_TARGET_CDNA4])
def test_amd_async_wait(target):
```
- **EN:** Defines the test function `test_amd_async_wait`. Decorators: `pytest.mark.parametrize('target', [HIP_TARGET_CDNA4])`. Parameters: `target`. Key calls include `pytest.mark.parametrize`, `run_parser`, `expecttest.assert_expected_inline`, `anonymize_ir`, `mod.str_nodebug`. This scope touches pytest parametrization, debugging and inspection paths.
- **CN:** 定义测试函数 `test_amd_async_wait`。 装饰器：`pytest.mark.parametrize('target', [HIP_TARGET_CDNA4])`。 参数：`target`。 关键调用包括 `pytest.mark.parametrize`、`run_parser`、`expecttest.assert_expected_inline`、`anonymize_ir`、`mod.str_nodebug`。 该作用域涉及pytest 参数化、调试与检查路径。

#### Lines 2387-2396

```python
    mod = run_parser(amd_wait_group, target=target)
    expecttest.assert_expected_inline(
        anonymize_ir(mod.str_nodebug()), """\
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "...", "ttg.threads-per-warp" = 64 : i32} {
  tt.func public @amd_wait_group() attributes {noinline = false} {
    %0 = ttg.async_wait {num = 0 : i32}
    tt.return
  }
}
""")
```
- **EN:** Prepares or updates state through `mod`. Invokes `run_parser`, `expecttest.assert_expected_inline`, `anonymize_ir`, `mod.str_nodebug` to execute the test logic. Relevant themes: debugging and inspection paths.
- **CN:** 通过 `mod` 准备或更新状态。 调用 `run_parser`、`expecttest.assert_expected_inline`、`anonymize_ir`、`mod.str_nodebug` 执行测试逻辑。 相关主题：调试与检查路径。

### Lines 2397-2402

```python


@pytest.mark.parametrize("target", [HIP_TARGET_CDNA4])
def test_amd_load_shared_relaxed(target):

    @gluon.jit
```
- **EN:** Defines the test function `test_amd_load_shared_relaxed`. Decorators: `pytest.mark.parametrize('target', [HIP_TARGET_CDNA4])`. Parameters: `target`. Nested definitions in this scope: `kernel`. Key calls include `pytest.mark.parametrize`, `run_parser`, `expecttest.assert_expected_inline`, `ttgl.BlockedLayout`, `ttgl.SwizzledSharedLayout`, `ttgl.allocate_shared_memory`, and 3 more. This scope touches pytest parametrization, debugging and inspection paths, layout transformation reasoning.
- **CN:** 定义测试函数 `test_amd_load_shared_relaxed`。 装饰器：`pytest.mark.parametrize('target', [HIP_TARGET_CDNA4])`。 参数：`target`。 该作用域中的嵌套定义：`kernel`。 关键调用包括 `pytest.mark.parametrize`、`run_parser`、`expecttest.assert_expected_inline`、`ttgl.BlockedLayout`、`ttgl.SwizzledSharedLayout`、`ttgl.allocate_shared_memory` 等另外 3 项。 该作用域涉及pytest 参数化、调试与检查路径、布局变换推理。

#### Lines 2402-2403

```python
    @gluon.jit
    def kernel():
```
- **EN:** Defines the helper function `kernel`. Decorators: `gluon.jit`. Key calls include `ttgl.BlockedLayout`, `ttgl.SwizzledSharedLayout`, `ttgl.allocate_shared_memory`, `cdna4_async_copy.load_shared_relaxed`. This scope touches layout transformation reasoning.
- **CN:** 定义辅助函数 `kernel`。 装饰器：`gluon.jit`。 关键调用包括 `ttgl.BlockedLayout`、`ttgl.SwizzledSharedLayout`、`ttgl.allocate_shared_memory`、`cdna4_async_copy.load_shared_relaxed`。 该作用域涉及布局变换推理。

##### Lines 2404-2408

```python
        blocked: ttgl.constexpr = ttgl.BlockedLayout([1, 8], [32, 2], [4, 1], [1, 0])
        shared: ttgl.constexpr = ttgl.SwizzledSharedLayout(1, 1, 1, order=[1, 0])

        smem = ttgl.allocate_shared_memory(ttgl.float16, [128, 16], shared)
        cdna4_async_copy.load_shared_relaxed(smem, blocked)
```
- **EN:** Prepares or updates state through `blocked`, `shared`, `smem`. Invokes `ttgl.BlockedLayout`, `ttgl.SwizzledSharedLayout`, `ttgl.allocate_shared_memory`, `cdna4_async_copy.load_shared_relaxed` to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `blocked`、`shared`、`smem` 准备或更新状态。 调用 `ttgl.BlockedLayout`、`ttgl.SwizzledSharedLayout`、`ttgl.allocate_shared_memory`、`cdna4_async_copy.load_shared_relaxed` 执行测试逻辑。 相关主题：布局变换推理。

#### Lines 2409-2423

```python

    mod = run_parser(kernel, target=target)
    expecttest.assert_expected_inline(
        anonymize_ir(mod.str_nodebug()), """\
#blocked = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [32, 2], warpsPerCTA = [4, 1], order = [1, 0]}>
#shared = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [1, 0]}>
#smem = #ttg.shared_memory
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "...", "ttg.threads-per-warp" = 64 : i32} {
  tt.func public @kernel() attributes {noinline = false} {
    %0 = ttg.local_alloc : () -> !ttg.memdesc<128x16xf16, #shared, #smem, mutable>
    %1 = ttg.local_load %0 {ttg.amdg.syncedViaAsyncWait = true} : !ttg.memdesc<128x16xf16, #shared, #smem, mutable> -> tensor<128x16xf16, #blocked>
    tt.return
  }
}
""")
```
- **EN:** Prepares or updates state through `mod`. Invokes `run_parser`, `expecttest.assert_expected_inline`, `anonymize_ir`, `mod.str_nodebug` to execute the test logic. Relevant themes: debugging and inspection paths.
- **CN:** 通过 `mod` 准备或更新状态。 调用 `run_parser`、`expecttest.assert_expected_inline`、`anonymize_ir`、`mod.str_nodebug` 执行测试逻辑。 相关主题：调试与检查路径。

### Lines 2424-2429

```python


@pytest.mark.parametrize("target", [HIP_TARGET_CDNA4])
def test_amd_load_shared_relaxed_in_loop(target):

    @gluon.jit
```
- **EN:** Defines the test function `test_amd_load_shared_relaxed_in_loop`. Decorators: `pytest.mark.parametrize('target', [HIP_TARGET_CDNA4])`. Parameters: `target`. Nested definitions in this scope: `kernel`. Key calls include `pytest.mark.parametrize`, `run_parser`, `expecttest.assert_expected_inline`, `ttgl.BlockedLayout`, `ttgl.SwizzledSharedLayout`, `ttgl.allocate_shared_memory`, and 3 more. This scope touches pytest parametrization, debugging and inspection paths, layout transformation reasoning.
- **CN:** 定义测试函数 `test_amd_load_shared_relaxed_in_loop`。 装饰器：`pytest.mark.parametrize('target', [HIP_TARGET_CDNA4])`。 参数：`target`。 该作用域中的嵌套定义：`kernel`。 关键调用包括 `pytest.mark.parametrize`、`run_parser`、`expecttest.assert_expected_inline`、`ttgl.BlockedLayout`、`ttgl.SwizzledSharedLayout`、`ttgl.allocate_shared_memory` 等另外 3 项。 该作用域涉及pytest 参数化、调试与检查路径、布局变换推理。

#### Lines 2429-2430

```python
    @gluon.jit
    def kernel():
```
- **EN:** Defines the helper function `kernel`. Decorators: `gluon.jit`. Key calls include `ttgl.BlockedLayout`, `ttgl.SwizzledSharedLayout`, `ttgl.allocate_shared_memory`, `cdna4_async_copy.load_shared_relaxed`. This scope touches layout transformation reasoning.
- **CN:** 定义辅助函数 `kernel`。 装饰器：`gluon.jit`。 关键调用包括 `ttgl.BlockedLayout`、`ttgl.SwizzledSharedLayout`、`ttgl.allocate_shared_memory`、`cdna4_async_copy.load_shared_relaxed`。 该作用域涉及布局变换推理。

##### Lines 2431-2434

```python
        blocked: ttgl.constexpr = ttgl.BlockedLayout([1, 8], [32, 2], [4, 1], [1, 0])
        shared: ttgl.constexpr = ttgl.SwizzledSharedLayout(1, 1, 1, order=[1, 0])

        smem = ttgl.allocate_shared_memory(ttgl.float16, [128, 16], shared)
```
- **EN:** Prepares or updates state through `blocked`, `shared`, `smem`. Invokes `ttgl.BlockedLayout`, `ttgl.SwizzledSharedLayout`, `ttgl.allocate_shared_memory` to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `blocked`、`shared`、`smem` 准备或更新状态。 调用 `ttgl.BlockedLayout`、`ttgl.SwizzledSharedLayout`、`ttgl.allocate_shared_memory` 执行测试逻辑。 相关主题：布局变换推理。

##### Lines 2435-2436

```python
        for i in range(10):
            cdna4_async_copy.load_shared_relaxed(smem, blocked)
```
- **EN:** Invokes `cdna4_async_copy.load_shared_relaxed` to execute the test logic. Iterates across cases or data tiles.
- **CN:** 调用 `cdna4_async_copy.load_shared_relaxed` 执行测试逻辑。 通过循环覆盖多个用例或数据分块。

#### Lines 2437-2460

```python

    mod = run_parser(kernel, target=target)
    expecttest.assert_expected_inline(
        anonymize_ir(mod.str_nodebug()), """\
#blocked = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [32, 2], warpsPerCTA = [4, 1], order = [1, 0]}>
#shared = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [1, 0]}>
#smem = #ttg.shared_memory
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "...", "ttg.threads-per-warp" = 64 : i32} {
  tt.func public @kernel() attributes {noinline = false} {
    %0 = ttg.local_alloc : () -> !ttg.memdesc<128x16xf16, #shared, #smem, mutable>
    %c0_i32 = arith.constant 0 : i32
    %c10_i32 = arith.constant 10 : i32
    %c1_i32 = arith.constant 1 : i32
    %1 = arith.bitcast %c0_i32 : i32 to i32
    %2 = arith.bitcast %c10_i32 : i32 to i32
    %3 = arith.bitcast %c1_i32 : i32 to i32
    %4 = ub.poison : i32
    scf.for %arg0 = %1 to %2 step %3  : i32 {
      %5 = ttg.local_load %0 {ttg.amdg.syncedViaAsyncWait = true} : !ttg.memdesc<128x16xf16, #shared, #smem, mutable> -> tensor<128x16xf16, #blocked>
    }
    tt.return
  }
}
""")
```
- **EN:** Prepares or updates state through `mod`. Invokes `run_parser`, `expecttest.assert_expected_inline`, `anonymize_ir`, `mod.str_nodebug` to execute the test logic. Relevant themes: debugging and inspection paths.
- **CN:** 通过 `mod` 准备或更新状态。 调用 `run_parser`、`expecttest.assert_expected_inline`、`anonymize_ir`、`mod.str_nodebug` 执行测试逻辑。 相关主题：调试与检查路径。

### Lines 2461-2464

```python


@gluon.jit
def amd_global_load_to_shared(ptr):
```
- **EN:** Defines the helper function `amd_global_load_to_shared`. Decorators: `gluon.jit`. Parameters: `ptr`. Key calls include `ttgl.BlockedLayout`, `ttgl.SwizzledSharedLayout`, `ttgl.allocate_shared_memory`, `ttgl.arange`, `cdna4_async_copy.global_load_to_shared`, `ttgl.full`, and 1 more. This scope touches layout transformation reasoning.
- **CN:** 定义辅助函数 `amd_global_load_to_shared`。 装饰器：`gluon.jit`。 参数：`ptr`。 关键调用包括 `ttgl.BlockedLayout`、`ttgl.SwizzledSharedLayout`、`ttgl.allocate_shared_memory`、`ttgl.arange`、`cdna4_async_copy.global_load_to_shared`、`ttgl.full` 等另外 1 项。 该作用域涉及布局变换推理。

#### Lines 2465-2484

```python
    blocked: ttgl.constexpr = ttgl.BlockedLayout([1, 8], [32, 2], [4, 1], [1, 0])
    shared: ttgl.constexpr = ttgl.SwizzledSharedLayout(1, 1, 1, order=[1, 0])

    smem = ttgl.allocate_shared_memory(ptr.dtype.element_ty, [128, 16], shared)
    y_offset = ttgl.arange(0, 128, layout=ttgl.SliceLayout(1, blocked))
    x_offset = ttgl.arange(0, 16, layout=ttgl.SliceLayout(0, blocked))
    offsets = y_offset[:, None] * 16 + x_offset[None, :]

    cdna4_async_copy.global_load_to_shared(smem, ptr + offsets)

    # test mask and other
    mask = (y_offset < 64)[:, None]

    other = ttgl.full([128, 16], 0.0, ptr.dtype.element_ty, layout=blocked)
    cdna4_async_copy.global_load_to_shared(smem, ptr + offsets, mask, other=other)

    other = ttgl.full([128, 1], 0.0, ptr.dtype.element_ty, layout=blocked)
    cdna4_async_copy.global_load_to_shared(smem, ptr + offsets, mask, other=other)

    cdna4_async_copy.global_load_to_shared(smem, ptr + offsets, mask, other=0.0)
```
- **EN:** Prepares or updates state through `blocked`, `shared`, `smem`, `y_offset`, `x_offset`, `offsets`, `mask`, `other`. Invokes `ttgl.BlockedLayout`, `ttgl.SwizzledSharedLayout`, `ttgl.allocate_shared_memory`, `ttgl.arange`, `ttgl.SliceLayout`, `cdna4_async_copy.global_load_to_shared`, and 1 more to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `blocked`、`shared`、`smem`、`y_offset`、`x_offset`、`offsets`、`mask`、`other` 准备或更新状态。 调用 `ttgl.BlockedLayout`、`ttgl.SwizzledSharedLayout`、`ttgl.allocate_shared_memory`、`ttgl.arange`、`ttgl.SliceLayout`、`cdna4_async_copy.global_load_to_shared` 等另外 1 项 执行测试逻辑。 相关主题：布局变换推理。

### Lines 2485-2488

```python


@pytest.mark.parametrize("target", [HIP_TARGET_CDNA4])
def test_amd_global_load_to_shared(target):
```
- **EN:** Defines the test function `test_amd_global_load_to_shared`. Decorators: `pytest.mark.parametrize('target', [HIP_TARGET_CDNA4])`. Parameters: `target`. Key calls include `pytest.mark.parametrize`, `MockTensor`, `run_parser`, `expecttest.assert_expected_inline`, `anonymize_ir`, `make_args`, and 1 more. This scope touches pytest parametrization, debugging and inspection paths.
- **CN:** 定义测试函数 `test_amd_global_load_to_shared`。 装饰器：`pytest.mark.parametrize('target', [HIP_TARGET_CDNA4])`。 参数：`target`。 关键调用包括 `pytest.mark.parametrize`、`MockTensor`、`run_parser`、`expecttest.assert_expected_inline`、`anonymize_ir`、`make_args` 等另外 1 项。 该作用域涉及pytest 参数化、调试与检查路径。

#### Lines 2489-2540

```python
    ptr = MockTensor(ttgl.float16)
    mod = run_parser(amd_global_load_to_shared, *make_args(ptr), target=target)
    expecttest.assert_expected_inline(
        anonymize_ir(mod.str_nodebug()), """\
#blocked = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [32, 2], warpsPerCTA = [4, 1], order = [1, 0]}>
#shared = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [1, 0]}>
#smem = #ttg.shared_memory
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "...", "ttg.threads-per-warp" = 64 : i32} {
  tt.func public @amd_global_load_to_shared(%arg0: !tt.ptr<f16> {tt.divisibility = 16 : i32}) attributes {noinline = false} {
    %0 = ttg.local_alloc : () -> !ttg.memdesc<128x16xf16, #shared, #smem, mutable>
    %1 = tt.make_range {end = 128 : i32, start = 0 : i32} : tensor<128xi32, #ttg.slice<{dim = 1, parent = #blocked}>>
    %2 = tt.make_range {end = 16 : i32, start = 0 : i32} : tensor<16xi32, #ttg.slice<{dim = 0, parent = #blocked}>>
    %3 = tt.expand_dims %1 {axis = 1 : i32} : tensor<128xi32, #ttg.slice<{dim = 1, parent = #blocked}>> -> tensor<128x1xi32, #blocked>
    %c16_i32 = arith.constant 16 : i32
    %c16_i32_0 = arith.constant 16 : i32
    %cst = arith.constant dense<16> : tensor<128x1xi32, #blocked>
    %4 = arith.muli %3, %cst : tensor<128x1xi32, #blocked>
    %5 = tt.expand_dims %2 {axis = 0 : i32} : tensor<16xi32, #ttg.slice<{dim = 0, parent = #blocked}>> -> tensor<1x16xi32, #blocked>
    %6 = tt.broadcast %4 : tensor<128x1xi32, #blocked> -> tensor<128x16xi32, #blocked>
    %7 = tt.broadcast %5 : tensor<1x16xi32, #blocked> -> tensor<128x16xi32, #blocked>
    %8 = arith.addi %6, %7 : tensor<128x16xi32, #blocked>
    %9 = tt.splat %arg0 : !tt.ptr<f16> -> tensor<128x16x!tt.ptr<f16>, #blocked>
    %10 = tt.addptr %9, %8 : tensor<128x16x!tt.ptr<f16>, #blocked>, tensor<128x16xi32, #blocked>
    %11 = ttg.async_copy_global_to_local %10, %0 : tensor<128x16x!tt.ptr<f16>, #blocked> -> <128x16xf16, #shared, #smem, mutable>
    %c64_i32 = arith.constant 64 : i32
    %cst_1 = arith.constant dense<64> : tensor<128xi32, #ttg.slice<{dim = 1, parent = #blocked}>>
    %12 = arith.cmpi slt, %1, %cst_1 : tensor<128xi32, #ttg.slice<{dim = 1, parent = #blocked}>>
    %13 = tt.expand_dims %12 {axis = 1 : i32} : tensor<128xi1, #ttg.slice<{dim = 1, parent = #blocked}>> -> tensor<128x1xi1, #blocked>
    %cst_2 = arith.constant 0.000000e+00 : f16
    %cst_3 = arith.constant dense<0.000000e+00> : tensor<128x16xf16, #blocked>
    %14 = tt.splat %arg0 : !tt.ptr<f16> -> tensor<128x16x!tt.ptr<f16>, #blocked>
    %15 = tt.addptr %14, %8 : tensor<128x16x!tt.ptr<f16>, #blocked>, tensor<128x16xi32, #blocked>
    %16 = tt.broadcast %13 : tensor<128x1xi1, #blocked> -> tensor<128x16xi1, #blocked>
    %17 = ttg.async_copy_global_to_local %15, %0 mask %16 other %cst_3 : tensor<128x16x!tt.ptr<f16>, #blocked> -> <128x16xf16, #shared, #smem, mutable>
    %cst_4 = arith.constant 0.000000e+00 : f16
    %cst_5 = arith.constant dense<0.000000e+00> : tensor<128x1xf16, #blocked>
    %18 = tt.splat %arg0 : !tt.ptr<f16> -> tensor<128x16x!tt.ptr<f16>, #blocked>
    %19 = tt.addptr %18, %8 : tensor<128x16x!tt.ptr<f16>, #blocked>, tensor<128x16xi32, #blocked>
    %20 = tt.broadcast %13 : tensor<128x1xi1, #blocked> -> tensor<128x16xi1, #blocked>
    %21 = tt.broadcast %cst_5 : tensor<128x1xf16, #blocked> -> tensor<128x16xf16, #blocked>
    %22 = ttg.async_copy_global_to_local %19, %0 mask %20 other %21 : tensor<128x16x!tt.ptr<f16>, #blocked> -> <128x16xf16, #shared, #smem, mutable>
    %23 = tt.splat %arg0 : !tt.ptr<f16> -> tensor<128x16x!tt.ptr<f16>, #blocked>
    %24 = tt.addptr %23, %8 : tensor<128x16x!tt.ptr<f16>, #blocked>, tensor<128x16xi32, #blocked>
    %25 = tt.broadcast %13 : tensor<128x1xi1, #blocked> -> tensor<128x16xi1, #blocked>
    %cst_6 = arith.constant 0.000000e+00 : f32
    %26 = arith.truncf %cst_6 : f32 to f16
    %27 = tt.splat %26 : f16 -> tensor<128x16xf16, #blocked>
    %28 = ttg.async_copy_global_to_local %24, %0 mask %25 other %27 : tensor<128x16x!tt.ptr<f16>, #blocked> -> <128x16xf16, #shared, #smem, mutable>
    tt.return
  }
}
""")
```
- **EN:** Prepares or updates state through `ptr`, `mod`. Invokes `MockTensor`, `run_parser`, `make_args`, `expecttest.assert_expected_inline`, `anonymize_ir`, `mod.str_nodebug` to execute the test logic. Relevant themes: debugging and inspection paths.
- **CN:** 通过 `ptr`、`mod` 准备或更新状态。 调用 `MockTensor`、`run_parser`、`make_args`、`expecttest.assert_expected_inline`、`anonymize_ir`、`mod.str_nodebug` 执行测试逻辑。 相关主题：调试与检查路径。

### Lines 2541-2544

```python


@gluon.jit
def buffer_load_to_shared_kernel(ptr):
```
- **EN:** Defines the helper function `buffer_load_to_shared_kernel`. Decorators: `gluon.jit`. Parameters: `ptr`. Key calls include `ttgl.BlockedLayout`, `ttgl.SwizzledSharedLayout`, `ttgl.allocate_shared_memory`, `ttgl.arange`, `cdna4_async_copy.buffer_load_to_shared`, `ttgl.full`, and 1 more. This scope touches cache management behavior, layout transformation reasoning.
- **CN:** 定义辅助函数 `buffer_load_to_shared_kernel`。 装饰器：`gluon.jit`。 参数：`ptr`。 关键调用包括 `ttgl.BlockedLayout`、`ttgl.SwizzledSharedLayout`、`ttgl.allocate_shared_memory`、`ttgl.arange`、`cdna4_async_copy.buffer_load_to_shared`、`ttgl.full` 等另外 1 项。 该作用域涉及缓存管理行为、布局变换推理。

#### Lines 2545-2569

```python
    blocked: ttgl.constexpr = ttgl.BlockedLayout([1, 8], [32, 2], [4, 1], [1, 0])
    shared: ttgl.constexpr = ttgl.SwizzledSharedLayout(1, 1, 1, order=[1, 0])

    smem = ttgl.allocate_shared_memory(ptr.dtype.element_ty, [128, 16], shared)
    y_offset = ttgl.arange(0, 128, layout=ttgl.SliceLayout(1, blocked))
    x_offset = ttgl.arange(0, 16, layout=ttgl.SliceLayout(0, blocked))
    offsets = y_offset[:, None] * 16 + x_offset[None, :]

    cdna4_async_copy.buffer_load_to_shared(smem, ptr, offsets)

    # test cache modifiers
    cdna4_async_copy.buffer_load_to_shared(smem, ptr, offsets, cache_modifier=".ca")
    cdna4_async_copy.buffer_load_to_shared(smem, ptr, offsets, cache_modifier=".cg")
    cdna4_async_copy.buffer_load_to_shared(smem, ptr, offsets, cache_modifier=".cv")

    # test mask and other
    mask = (y_offset < 64)[:, None]

    other = ttgl.full([128, 16], 0.0, ptr.dtype.element_ty, layout=blocked)
    cdna4_async_copy.buffer_load_to_shared(smem, ptr, offsets, mask, other=other)

    other = ttgl.full([128, 1], 0.0, ptr.dtype.element_ty, layout=blocked)
    cdna4_async_copy.buffer_load_to_shared(smem, ptr, offsets, mask, other=other)

    cdna4_async_copy.buffer_load_to_shared(smem, ptr, offsets, mask, other=0.0)
```
- **EN:** Prepares or updates state through `blocked`, `shared`, `smem`, `y_offset`, `x_offset`, `offsets`, `mask`, `other`. Invokes `ttgl.BlockedLayout`, `ttgl.SwizzledSharedLayout`, `ttgl.allocate_shared_memory`, `ttgl.arange`, `ttgl.SliceLayout`, `cdna4_async_copy.buffer_load_to_shared`, and 1 more to execute the test logic. Relevant themes: cache management behavior, layout transformation reasoning.
- **CN:** 通过 `blocked`、`shared`、`smem`、`y_offset`、`x_offset`、`offsets`、`mask`、`other` 准备或更新状态。 调用 `ttgl.BlockedLayout`、`ttgl.SwizzledSharedLayout`、`ttgl.allocate_shared_memory`、`ttgl.arange`、`ttgl.SliceLayout`、`cdna4_async_copy.buffer_load_to_shared` 等另外 1 项 执行测试逻辑。 相关主题：缓存管理行为、布局变换推理。

### Lines 2570-2573

```python


@pytest.mark.parametrize("target", [HIP_TARGET_CDNA4])
def test_buffer_load_to_shared(target):
```
- **EN:** Defines the test function `test_buffer_load_to_shared`. Decorators: `pytest.mark.parametrize('target', [HIP_TARGET_CDNA4])`. Parameters: `target`. Key calls include `pytest.mark.parametrize`, `MockTensor`, `run_parser`, `expecttest.assert_expected_inline`, `anonymize_ir`, `make_args`, and 1 more. This scope touches pytest parametrization, cache management behavior, debugging and inspection paths.
- **CN:** 定义测试函数 `test_buffer_load_to_shared`。 装饰器：`pytest.mark.parametrize('target', [HIP_TARGET_CDNA4])`。 参数：`target`。 关键调用包括 `pytest.mark.parametrize`、`MockTensor`、`run_parser`、`expecttest.assert_expected_inline`、`anonymize_ir`、`make_args` 等另外 1 项。 该作用域涉及pytest 参数化、缓存管理行为、调试与检查路径。

#### Lines 2574-2620

```python
    ptr = MockTensor(ttgl.float16)
    mod = run_parser(buffer_load_to_shared_kernel, *make_args(ptr), target=target)
    expecttest.assert_expected_inline(
        anonymize_ir(mod.str_nodebug()), """\
#blocked = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [32, 2], warpsPerCTA = [4, 1], order = [1, 0]}>
#shared = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [1, 0]}>
#smem = #ttg.shared_memory
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "...", "ttg.threads-per-warp" = 64 : i32} {
  tt.func public @buffer_load_to_shared_kernel(%arg0: !tt.ptr<f16> {tt.divisibility = 16 : i32}) attributes {noinline = false} {
    %0 = ttg.local_alloc : () -> !ttg.memdesc<128x16xf16, #shared, #smem, mutable>
    %1 = tt.make_range {end = 128 : i32, start = 0 : i32} : tensor<128xi32, #ttg.slice<{dim = 1, parent = #blocked}>>
    %2 = tt.make_range {end = 16 : i32, start = 0 : i32} : tensor<16xi32, #ttg.slice<{dim = 0, parent = #blocked}>>
    %3 = tt.expand_dims %1 {axis = 1 : i32} : tensor<128xi32, #ttg.slice<{dim = 1, parent = #blocked}>> -> tensor<128x1xi32, #blocked>
    %c16_i32 = arith.constant 16 : i32
    %c16_i32_0 = arith.constant 16 : i32
    %cst = arith.constant dense<16> : tensor<128x1xi32, #blocked>
    %4 = arith.muli %3, %cst : tensor<128x1xi32, #blocked>
    %5 = tt.expand_dims %2 {axis = 0 : i32} : tensor<16xi32, #ttg.slice<{dim = 0, parent = #blocked}>> -> tensor<1x16xi32, #blocked>
    %6 = tt.broadcast %4 : tensor<128x1xi32, #blocked> -> tensor<128x16xi32, #blocked>
    %7 = tt.broadcast %5 : tensor<1x16xi32, #blocked> -> tensor<128x16xi32, #blocked>
    %8 = arith.addi %6, %7 : tensor<128x16xi32, #blocked>
    %9 = amdg.buffer_load_to_local %arg0[%8] into %0 : <f16>[tensor<128x16xi32, #blocked>]  -> <128x16xf16, #shared, #smem, mutable>
    %10 = amdg.buffer_load_to_local %arg0[%8] cacheModifier = ca into %0 : <f16>[tensor<128x16xi32, #blocked>]  -> <128x16xf16, #shared, #smem, mutable>
    %11 = amdg.buffer_load_to_local %arg0[%8] cacheModifier = cg into %0 : <f16>[tensor<128x16xi32, #blocked>]  -> <128x16xf16, #shared, #smem, mutable>
    %12 = amdg.buffer_load_to_local %arg0[%8] cacheModifier = cv into %0 : <f16>[tensor<128x16xi32, #blocked>]  -> <128x16xf16, #shared, #smem, mutable>
    %c64_i32 = arith.constant 64 : i32
    %cst_1 = arith.constant dense<64> : tensor<128xi32, #ttg.slice<{dim = 1, parent = #blocked}>>
    %13 = arith.cmpi slt, %1, %cst_1 : tensor<128xi32, #ttg.slice<{dim = 1, parent = #blocked}>>
    %14 = tt.expand_dims %13 {axis = 1 : i32} : tensor<128xi1, #ttg.slice<{dim = 1, parent = #blocked}>> -> tensor<128x1xi1, #blocked>
    %cst_2 = arith.constant 0.000000e+00 : f16
    %cst_3 = arith.constant dense<0.000000e+00> : tensor<128x16xf16, #blocked>
    %15 = tt.broadcast %14 : tensor<128x1xi1, #blocked> -> tensor<128x16xi1, #blocked>
    %16 = amdg.buffer_load_to_local %arg0[%8] mask = %15 other = %cst_3 into %0 : <f16>[tensor<128x16xi32, #blocked>] tensor<128x16xf16, #blocked> -> <128x16xf16, #shared, #smem, mutable>
    %cst_4 = arith.constant 0.000000e+00 : f16
    %cst_5 = arith.constant dense<0.000000e+00> : tensor<128x1xf16, #blocked>
    %17 = tt.broadcast %14 : tensor<128x1xi1, #blocked> -> tensor<128x16xi1, #blocked>
    %18 = tt.broadcast %cst_5 : tensor<128x1xf16, #blocked> -> tensor<128x16xf16, #blocked>
    %19 = amdg.buffer_load_to_local %arg0[%8] mask = %17 other = %18 into %0 : <f16>[tensor<128x16xi32, #blocked>] tensor<128x16xf16, #blocked> -> <128x16xf16, #shared, #smem, mutable>
    %20 = tt.broadcast %14 : tensor<128x1xi1, #blocked> -> tensor<128x16xi1, #blocked>
    %cst_6 = arith.constant 0.000000e+00 : f32
    %21 = arith.truncf %cst_6 : f32 to f16
    %22 = tt.splat %21 : f16 -> tensor<128x16xf16, #blocked>
    %23 = amdg.buffer_load_to_local %arg0[%8] mask = %20 other = %22 into %0 : <f16>[tensor<128x16xi32, #blocked>] tensor<128x16xf16, #blocked> -> <128x16xf16, #shared, #smem, mutable>
    tt.return
  }
}
""")
```
- **EN:** Prepares or updates state through `ptr`, `mod`. Invokes `MockTensor`, `run_parser`, `make_args`, `expecttest.assert_expected_inline`, `anonymize_ir`, `mod.str_nodebug` to execute the test logic. Relevant themes: cache management behavior, debugging and inspection paths.
- **CN:** 通过 `ptr`、`mod` 准备或更新状态。 调用 `MockTensor`、`run_parser`、`make_args`、`expecttest.assert_expected_inline`、`anonymize_ir`、`mod.str_nodebug` 执行测试逻辑。 相关主题：缓存管理行为、调试与检查路径。

### Lines 2621-2624

```python


@gluon.jit
def buffer_load_store_kernel(x, y):
```
- **EN:** Defines the helper function `buffer_load_store_kernel`. Decorators: `gluon.jit`. Parameters: `x`, `y`. Key calls include `ttgl.BlockedLayout`, `ttgl.convert_layout`, `ttgl.full`, `ttgl.amd.cdna3.buffer_load`, `ttgl.amd.cdna3.buffer_store`, `ttgl.amd.cdna4.buffer_load`, and 2 more. This scope touches Triton language operations, cache management behavior, layout transformation reasoning.
- **CN:** 定义辅助函数 `buffer_load_store_kernel`。 装饰器：`gluon.jit`。 参数：`x`、`y`。 关键调用包括 `ttgl.BlockedLayout`、`ttgl.convert_layout`、`ttgl.full`、`ttgl.amd.cdna3.buffer_load`、`ttgl.amd.cdna3.buffer_store`、`ttgl.amd.cdna4.buffer_load` 等另外 2 项。 该作用域涉及Triton language 操作、缓存管理行为、布局变换推理。

#### Lines 2625-2642

```python
    layout: ttgl.constexpr = ttgl.BlockedLayout(size_per_thread=[1, 1], threads_per_warp=[1, 64], warps_per_cta=[4, 1],
                                                order=[1, 0])

    auto_layout_offsets = ttgl.arange(0, 64 * 64).reshape(64, 64)
    offsets = ttgl.convert_layout(auto_layout_offsets, layout=layout)
    mask = ttgl.full((64, 64), 1, tl.int1, layout=layout)
    other = ttgl.full((64, 64), 1.0, tl.float32, layout=layout)
    a = ttgl.amd.cdna3.buffer_load(ptr=x, offsets=offsets, mask=mask, other=other, cache='.ca')
    ttgl.amd.cdna3.buffer_store(stored_value=a, ptr=y, offsets=offsets, mask=mask, cache='.cs')

    a = ttgl.amd.cdna4.buffer_load(ptr=x, offsets=offsets, mask=mask, other=other, cache='.ca')
    ttgl.amd.cdna4.buffer_store(stored_value=a, ptr=y, offsets=offsets, mask=mask, cache='.cs')

    # Test auto layout support
    auto_layout_mask = ttgl.full((64, 64), 1, tl.int1)
    auto_layout_other = ttgl.full((64, 64), 1.0, tl.float32)
    a = ttgl.amd.cdna4.buffer_load(ptr=x, offsets=offsets, mask=auto_layout_mask, other=auto_layout_other)
    ttgl.amd.cdna4.buffer_store(stored_value=a, ptr=y, offsets=auto_layout_offsets, mask=auto_layout_mask)
```
- **EN:** Prepares or updates state through `layout`, `auto_layout_offsets`, `offsets`, `mask`, `other`, `a`, `auto_layout_mask`, `auto_layout_other`. Invokes `ttgl.BlockedLayout`, `ttgl.arange`, `ttgl.convert_layout`, `ttgl.full`, `ttgl.amd.cdna3.buffer_load`, `ttgl.amd.cdna3.buffer_store`, and 2 more to execute the test logic. Relevant themes: Triton language operations, cache management behavior, layout transformation reasoning.
- **CN:** 通过 `layout`、`auto_layout_offsets`、`offsets`、`mask`、`other`、`a`、`auto_layout_mask`、`auto_layout_other` 准备或更新状态。 调用 `ttgl.BlockedLayout`、`ttgl.arange`、`ttgl.convert_layout`、`ttgl.full`、`ttgl.amd.cdna3.buffer_load`、`ttgl.amd.cdna3.buffer_store` 等另外 2 项 执行测试逻辑。 相关主题：Triton language 操作、缓存管理行为、布局变换推理。

### Lines 2643-2645

```python


def test_buffer_load_store():
```
- **EN:** Defines the test function `test_buffer_load_store`. Key calls include `MockTensor`, `run_parser`, `expecttest.assert_expected_inline`, `anonymize_ir`, `make_args`, `module.str_nodebug`. This scope touches cache management behavior, debugging and inspection paths, layout transformation reasoning.
- **CN:** 定义测试函数 `test_buffer_load_store`。 关键调用包括 `MockTensor`、`run_parser`、`expecttest.assert_expected_inline`、`anonymize_ir`、`make_args`、`module.str_nodebug`。 该作用域涉及缓存管理行为、调试与检查路径、布局变换推理。

#### Lines 2646-2679

```python
    x = MockTensor(ttgl.float32)
    y = MockTensor(ttgl.float32)
    module = run_parser(buffer_load_store_kernel, *make_args(x, y), target=HIP_TARGET_CDNA3)

    expecttest.assert_expected_inline(
        anonymize_ir(module.str_nodebug()), """\
#blocked = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [1, 64], warpsPerCTA = [4, 1], order = [1, 0]}>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "...", "ttg.threads-per-warp" = 64 : i32} {
  tt.func public @buffer_load_store_kernel(%arg0: !tt.ptr<f32> {tt.divisibility = 16 : i32}, %arg1: !tt.ptr<f32> {tt.divisibility = 16 : i32}) attributes {noinline = false} {
    %0 = tt.make_range {end = 4096 : i32, start = 0 : i32} : tensor<4096xi32, #gluon.auto_encoding>
    %1 = tt.reshape %0 : tensor<4096xi32, #gluon.auto_encoding> -> tensor<64x64xi32, #gluon.auto_encoding>
    %2 = ttg.convert_layout %1 : tensor<64x64xi32, #gluon.auto_encoding> -> tensor<64x64xi32, #blocked>
    %true = arith.constant true
    %cst = arith.constant dense<true> : tensor<64x64xi1, #blocked>
    %cst_0 = arith.constant 1.000000e+00 : f32
    %cst_1 = arith.constant dense<1.000000e+00> : tensor<64x64xf32, #blocked>
    %3 = amdg.buffer_load %arg0[%2], %cst, %cst_1 cacheModifier = ca : tensor<64x64xf32, #blocked>
    amdg.buffer_store %3, %arg1[%2], %cst cacheModifier = cs : tensor<64x64xf32, #blocked>
    %4 = amdg.buffer_load %arg0[%2], %cst, %cst_1 cacheModifier = ca : tensor<64x64xf32, #blocked>
    amdg.buffer_store %4, %arg1[%2], %cst cacheModifier = cs : tensor<64x64xf32, #blocked>
    %true_2 = arith.constant true
    %cst_3 = arith.constant dense<true> : tensor<64x64xi1, #gluon.auto_encoding>
    %cst_4 = arith.constant 1.000000e+00 : f32
    %cst_5 = arith.constant dense<1.000000e+00> : tensor<64x64xf32, #gluon.auto_encoding>
    %5 = gluon.set_auto_layout %cst_3 : tensor<64x64xi1, #gluon.auto_encoding> -> tensor<64x64xi1, #blocked>
    %6 = gluon.set_auto_layout %cst_5 : tensor<64x64xf32, #gluon.auto_encoding> -> tensor<64x64xf32, #blocked>
    %7 = amdg.buffer_load %arg0[%2], %5, %6 : tensor<64x64xf32, #blocked>
    %8 = gluon.set_auto_layout %1 : tensor<64x64xi32, #gluon.auto_encoding> -> tensor<64x64xi32, #blocked>
    %9 = gluon.set_auto_layout %cst_3 : tensor<64x64xi1, #gluon.auto_encoding> -> tensor<64x64xi1, #blocked>
    amdg.buffer_store %7, %arg1[%8], %9 : tensor<64x64xf32, #blocked>
    tt.return
  }
}
""")
```
- **EN:** Prepares or updates state through `x`, `y`, `module`. Invokes `MockTensor`, `run_parser`, `make_args`, `expecttest.assert_expected_inline`, `anonymize_ir`, `module.str_nodebug` to execute the test logic. Relevant themes: cache management behavior, debugging and inspection paths, layout transformation reasoning.
- **CN:** 通过 `x`、`y`、`module` 准备或更新状态。 调用 `MockTensor`、`run_parser`、`make_args`、`expecttest.assert_expected_inline`、`anonymize_ir`、`module.str_nodebug` 执行测试逻辑。 相关主题：缓存管理行为、调试与检查路径、布局变换推理。

### Lines 2680-2683

```python


@gluon.jit
def buffer_load_store_with_broadcast_kernel(x, y):
```
- **EN:** Defines the helper function `buffer_load_store_with_broadcast_kernel`. Decorators: `gluon.jit`. Parameters: `x`, `y`. Key calls include `ttgl.BlockedLayout`, `ttgl.convert_layout`, `ttgl.full`, `ttgl.amd.cdna3.buffer_load`, `ttgl.amd.cdna3.buffer_store`, `ttgl.arange`. This scope touches Triton language operations, cache management behavior, layout transformation reasoning.
- **CN:** 定义辅助函数 `buffer_load_store_with_broadcast_kernel`。 装饰器：`gluon.jit`。 参数：`x`、`y`。 关键调用包括 `ttgl.BlockedLayout`、`ttgl.convert_layout`、`ttgl.full`、`ttgl.amd.cdna3.buffer_load`、`ttgl.amd.cdna3.buffer_store`、`ttgl.arange`。 该作用域涉及Triton language 操作、缓存管理行为、布局变换推理。

#### Lines 2684-2700

```python
    layout: ttgl.constexpr = ttgl.BlockedLayout(size_per_thread=[1, 1], threads_per_warp=[1, 64], warps_per_cta=[4, 1],
                                                order=[1, 0])

    offsets = ttgl.arange(0, 64 * 64).reshape(64, 64)
    offsets = ttgl.convert_layout(offsets, layout=layout)
    other = ttgl.full((64, 64), 1.0, tl.float32, layout=layout)

    mask = ttgl.full((64, 1), 1, tl.int1, layout=layout)
    a = ttgl.amd.cdna3.buffer_load(ptr=x, offsets=offsets, mask=mask, other=other, cache='.ca')
    ttgl.amd.cdna3.buffer_store(stored_value=a, ptr=y, offsets=offsets, mask=mask, cache='.cs')

    mask = ttgl.full((1, 64), 1, tl.int1, layout=layout)
    a = ttgl.amd.cdna3.buffer_load(ptr=x, offsets=offsets, mask=mask, other=other, cache='.ca')
    ttgl.amd.cdna3.buffer_store(stored_value=a, ptr=y, offsets=offsets, mask=mask, cache='.cs')

    a = ttgl.amd.cdna3.buffer_load(ptr=x, offsets=offsets, mask=mask, other=1.0, cache='.ca')
    ttgl.amd.cdna3.buffer_store(stored_value=a, ptr=y, offsets=offsets, mask=mask, cache='.cs')
```
- **EN:** Prepares or updates state through `layout`, `offsets`, `other`, `mask`, `a`. Invokes `ttgl.BlockedLayout`, `ttgl.arange`, `ttgl.convert_layout`, `ttgl.full`, `ttgl.amd.cdna3.buffer_load`, `ttgl.amd.cdna3.buffer_store` to execute the test logic. Relevant themes: Triton language operations, cache management behavior, layout transformation reasoning.
- **CN:** 通过 `layout`、`offsets`、`other`、`mask`、`a` 准备或更新状态。 调用 `ttgl.BlockedLayout`、`ttgl.arange`、`ttgl.convert_layout`、`ttgl.full`、`ttgl.amd.cdna3.buffer_load`、`ttgl.amd.cdna3.buffer_store` 执行测试逻辑。 相关主题：Triton language 操作、缓存管理行为、布局变换推理。

### Lines 2701-2703

```python


def test_buffer_load_store_with_broadcast():
```
- **EN:** Defines the test function `test_buffer_load_store_with_broadcast`. Key calls include `MockTensor`, `run_parser`, `expecttest.assert_expected_inline`, `anonymize_ir`, `make_args`, `module.str_nodebug`. This scope touches cache management behavior, debugging and inspection paths, layout transformation reasoning.
- **CN:** 定义测试函数 `test_buffer_load_store_with_broadcast`。 关键调用包括 `MockTensor`、`run_parser`、`expecttest.assert_expected_inline`、`anonymize_ir`、`make_args`、`module.str_nodebug`。 该作用域涉及缓存管理行为、调试与检查路径、布局变换推理。

#### Lines 2704-2742

```python
    x = MockTensor(ttgl.float16)
    y = MockTensor(ttgl.float16)
    module = run_parser(buffer_load_store_with_broadcast_kernel, *make_args(x, y), target=HIP_TARGET_CDNA3)

    expecttest.assert_expected_inline(
        anonymize_ir(module.str_nodebug()), """\
#blocked = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [1, 64], warpsPerCTA = [4, 1], order = [1, 0]}>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "...", "ttg.threads-per-warp" = 64 : i32} {
  tt.func public @buffer_load_store_with_broadcast_kernel(%arg0: !tt.ptr<f16> {tt.divisibility = 16 : i32}, %arg1: !tt.ptr<f16> {tt.divisibility = 16 : i32}) attributes {noinline = false} {
    %0 = tt.make_range {end = 4096 : i32, start = 0 : i32} : tensor<4096xi32, #gluon.auto_encoding>
    %1 = tt.reshape %0 : tensor<4096xi32, #gluon.auto_encoding> -> tensor<64x64xi32, #gluon.auto_encoding>
    %2 = ttg.convert_layout %1 : tensor<64x64xi32, #gluon.auto_encoding> -> tensor<64x64xi32, #blocked>
    %cst = arith.constant 1.000000e+00 : f32
    %cst_0 = arith.constant dense<1.000000e+00> : tensor<64x64xf32, #blocked>
    %true = arith.constant true
    %cst_1 = arith.constant dense<true> : tensor<64x1xi1, #blocked>
    %3 = tt.broadcast %cst_1 : tensor<64x1xi1, #blocked> -> tensor<64x64xi1, #blocked>
    %4 = arith.truncf %cst_0 : tensor<64x64xf32, #blocked> to tensor<64x64xf16, #blocked>
    %5 = amdg.buffer_load %arg0[%2], %3, %4 cacheModifier = ca : tensor<64x64xf16, #blocked>
    %6 = tt.broadcast %cst_1 : tensor<64x1xi1, #blocked> -> tensor<64x64xi1, #blocked>
    amdg.buffer_store %5, %arg1[%2], %6 cacheModifier = cs : tensor<64x64xf16, #blocked>
    %true_2 = arith.constant true
    %cst_3 = arith.constant dense<true> : tensor<1x64xi1, #blocked>
    %7 = tt.broadcast %cst_3 : tensor<1x64xi1, #blocked> -> tensor<64x64xi1, #blocked>
    %8 = arith.truncf %cst_0 : tensor<64x64xf32, #blocked> to tensor<64x64xf16, #blocked>
    %9 = amdg.buffer_load %arg0[%2], %7, %8 cacheModifier = ca : tensor<64x64xf16, #blocked>
    %10 = tt.broadcast %cst_3 : tensor<1x64xi1, #blocked> -> tensor<64x64xi1, #blocked>
    amdg.buffer_store %9, %arg1[%2], %10 cacheModifier = cs : tensor<64x64xf16, #blocked>
    %11 = tt.broadcast %cst_3 : tensor<1x64xi1, #blocked> -> tensor<64x64xi1, #blocked>
    %cst_4 = arith.constant 1.000000e+00 : f32
    %12 = arith.truncf %cst_4 : f32 to f16
    %13 = tt.splat %12 : f16 -> tensor<64x64xf16, #blocked>
    %14 = amdg.buffer_load %arg0[%2], %11, %13 cacheModifier = ca : tensor<64x64xf16, #blocked>
    %15 = tt.broadcast %cst_3 : tensor<1x64xi1, #blocked> -> tensor<64x64xi1, #blocked>
    amdg.buffer_store %14, %arg1[%2], %15 cacheModifier = cs : tensor<64x64xf16, #blocked>
    tt.return
  }
}
""")
```
- **EN:** Prepares or updates state through `x`, `y`, `module`. Invokes `MockTensor`, `run_parser`, `make_args`, `expecttest.assert_expected_inline`, `anonymize_ir`, `module.str_nodebug` to execute the test logic. Relevant themes: cache management behavior, debugging and inspection paths, layout transformation reasoning.
- **CN:** 通过 `x`、`y`、`module` 准备或更新状态。 调用 `MockTensor`、`run_parser`、`make_args`、`expecttest.assert_expected_inline`、`anonymize_ir`、`module.str_nodebug` 执行测试逻辑。 相关主题：缓存管理行为、调试与检查路径、布局变换推理。

### Lines 2743-2748

```python


@pytest.mark.parametrize("target", [HIP_TARGET_RDNA3])
def test_amd_rdna3_wmma(target):

    @gluon.jit
```
- **EN:** Defines the test function `test_amd_rdna3_wmma`. Decorators: `pytest.mark.parametrize('target', [HIP_TARGET_RDNA3])`. Parameters: `target`. Nested definitions in this scope: `kernel`. Key calls include `pytest.mark.parametrize`, `run_parser`, `expecttest.assert_expected_inline`, `ttgl.amd.AMDWMMALayout`, `ttgl.full`, `ttgl.amd.rdna3.wmma`, and 5 more. This scope touches pytest parametrization, debugging and inspection paths, layout transformation reasoning, random-data generation.
- **CN:** 定义测试函数 `test_amd_rdna3_wmma`。 装饰器：`pytest.mark.parametrize('target', [HIP_TARGET_RDNA3])`。 参数：`target`。 该作用域中的嵌套定义：`kernel`。 关键调用包括 `pytest.mark.parametrize`、`run_parser`、`expecttest.assert_expected_inline`、`ttgl.amd.AMDWMMALayout`、`ttgl.full`、`ttgl.amd.rdna3.wmma` 等另外 5 项。 该作用域涉及pytest 参数化、调试与检查路径、布局变换推理、随机数据生成。

#### Lines 2748-2749

```python
    @gluon.jit
    def kernel():
```
- **EN:** Defines the helper function `kernel`. Decorators: `gluon.jit`. Key calls include `ttgl.amd.AMDWMMALayout`, `ttgl.full`, `ttgl.amd.rdna3.wmma`, `ttgl.static_assert`, `isinstance`, `ttgl.DotOperandLayout`. This scope touches layout transformation reasoning, random-data generation.
- **CN:** 定义辅助函数 `kernel`。 装饰器：`gluon.jit`。 关键调用包括 `ttgl.amd.AMDWMMALayout`、`ttgl.full`、`ttgl.amd.rdna3.wmma`、`ttgl.static_assert`、`isinstance`、`ttgl.DotOperandLayout`。 该作用域涉及布局变换推理、随机数据生成。

##### Lines 2750-2759

```python
        wmma_layout: ttgl.constexpr = ttgl.amd.AMDWMMALayout(version=1, transposed=True, warp_bases=[[1, 0], [2, 0]])

        a = ttgl.full([64, 64], 1.0, ttgl.float16, layout=ttgl.DotOperandLayout(0, wmma_layout, 16))
        b = ttgl.full([64, 64], 2.0, ttgl.float16, layout=ttgl.DotOperandLayout(1, wmma_layout, 16))

        acc = ttgl.full([64, 64], 0.0, ttgl.float32, layout=wmma_layout)
        acc = ttgl.amd.rdna3.wmma(a, b, acc)

        ttgl.static_assert(isinstance(acc, ttgl.tensor))
        ttgl.static_assert(acc.type.layout == wmma_layout)
```
- **EN:** Prepares or updates state through `wmma_layout`, `a`, `b`, `acc`. Invokes `ttgl.amd.AMDWMMALayout`, `ttgl.full`, `ttgl.DotOperandLayout`, `ttgl.amd.rdna3.wmma`, `ttgl.static_assert`, `isinstance` to execute the test logic. Relevant themes: layout transformation reasoning, random-data generation.
- **CN:** 通过 `wmma_layout`、`a`、`b`、`acc` 准备或更新状态。 调用 `ttgl.amd.AMDWMMALayout`、`ttgl.full`、`ttgl.DotOperandLayout`、`ttgl.amd.rdna3.wmma`、`ttgl.static_assert`、`isinstance` 执行测试逻辑。 相关主题：布局变换推理、随机数据生成。

#### Lines 2760-2778

```python

    module = run_parser(kernel, target=target)
    expecttest.assert_expected_inline(
        anonymize_ir(module.str_nodebug()), """\
#mma = #ttg.amd_wmma<{version = 1, isTranspose = true, ctaLayout = {warp = [[1, 0], [2, 0]]}}>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "...", "ttg.threads-per-warp" = 32 : i32} {
  tt.func public @kernel() attributes {noinline = false} {
    %cst = arith.constant 1.000000e+00 : f16
    %cst_0 = arith.constant dense<1.000000e+00> : tensor<64x64xf16, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 16}>>
    %cst_1 = arith.constant 2.000000e+00 : f16
    %cst_2 = arith.constant dense<2.000000e+00> : tensor<64x64xf16, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 16}>>
    %cst_3 = arith.constant 0.000000e+00 : f32
    %cst_4 = arith.constant dense<0.000000e+00> : tensor<64x64xf32, #mma>
    %cst_5 = arith.constant 0.000000e+00 : f32
    %0 = tt.dot %cst_0, %cst_2, %cst_4 : tensor<64x64xf16, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 16}>> * tensor<64x64xf16, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 16}>> -> tensor<64x64xf32, #mma>
    tt.return
  }
}
""")
```
- **EN:** Prepares or updates state through `module`. Invokes `run_parser`, `expecttest.assert_expected_inline`, `anonymize_ir`, `module.str_nodebug` to execute the test logic. Relevant themes: debugging and inspection paths, layout transformation reasoning.
- **CN:** 通过 `module` 准备或更新状态。 调用 `run_parser`、`expecttest.assert_expected_inline`、`anonymize_ir`、`module.str_nodebug` 执行测试逻辑。 相关主题：调试与检查路径、布局变换推理。

### Lines 2779-2784

```python


@pytest.mark.parametrize("target", [HIP_TARGET_RDNA4])
def test_amd_rdna4_wmma(target):

    @gluon.jit
```
- **EN:** Defines the test function `test_amd_rdna4_wmma`. Decorators: `pytest.mark.parametrize('target', [HIP_TARGET_RDNA4])`. Parameters: `target`. Nested definitions in this scope: `kernel`. Key calls include `pytest.mark.parametrize`, `run_parser`, `expecttest.assert_expected_inline`, `ttgl.amd.AMDWMMALayout`, `ttgl.full`, `ttgl.amd.rdna4.wmma`, and 5 more. This scope touches pytest parametrization, debugging and inspection paths, layout transformation reasoning, random-data generation.
- **CN:** 定义测试函数 `test_amd_rdna4_wmma`。 装饰器：`pytest.mark.parametrize('target', [HIP_TARGET_RDNA4])`。 参数：`target`。 该作用域中的嵌套定义：`kernel`。 关键调用包括 `pytest.mark.parametrize`、`run_parser`、`expecttest.assert_expected_inline`、`ttgl.amd.AMDWMMALayout`、`ttgl.full`、`ttgl.amd.rdna4.wmma` 等另外 5 项。 该作用域涉及pytest 参数化、调试与检查路径、布局变换推理、随机数据生成。

#### Lines 2784-2785

```python
    @gluon.jit
    def kernel():
```
- **EN:** Defines the helper function `kernel`. Decorators: `gluon.jit`. Key calls include `ttgl.amd.AMDWMMALayout`, `ttgl.full`, `ttgl.amd.rdna4.wmma`, `ttgl.static_assert`, `isinstance`, `ttgl.DotOperandLayout`. This scope touches layout transformation reasoning, random-data generation.
- **CN:** 定义辅助函数 `kernel`。 装饰器：`gluon.jit`。 关键调用包括 `ttgl.amd.AMDWMMALayout`、`ttgl.full`、`ttgl.amd.rdna4.wmma`、`ttgl.static_assert`、`isinstance`、`ttgl.DotOperandLayout`。 该作用域涉及布局变换推理、随机数据生成。

##### Lines 2786-2795

```python
        wmma_layout: ttgl.constexpr = ttgl.amd.AMDWMMALayout(version=2, transposed=True, warp_bases=[[1, 0], [2, 0]])

        a = ttgl.full([64, 64], 1.0, ttgl.float16, layout=ttgl.DotOperandLayout(0, wmma_layout, 8))
        b = ttgl.full([64, 64], 2.0, ttgl.float16, layout=ttgl.DotOperandLayout(1, wmma_layout, 8))

        acc = ttgl.full([64, 64], 0.0, ttgl.float32, layout=wmma_layout)
        acc = ttgl.amd.rdna4.wmma(a, b, acc)

        ttgl.static_assert(isinstance(acc, ttgl.tensor))
        ttgl.static_assert(acc.type.layout == wmma_layout)
```
- **EN:** Prepares or updates state through `wmma_layout`, `a`, `b`, `acc`. Invokes `ttgl.amd.AMDWMMALayout`, `ttgl.full`, `ttgl.DotOperandLayout`, `ttgl.amd.rdna4.wmma`, `ttgl.static_assert`, `isinstance` to execute the test logic. Relevant themes: layout transformation reasoning, random-data generation.
- **CN:** 通过 `wmma_layout`、`a`、`b`、`acc` 准备或更新状态。 调用 `ttgl.amd.AMDWMMALayout`、`ttgl.full`、`ttgl.DotOperandLayout`、`ttgl.amd.rdna4.wmma`、`ttgl.static_assert`、`isinstance` 执行测试逻辑。 相关主题：布局变换推理、随机数据生成。

#### Lines 2796-2814

```python

    module = run_parser(kernel, target=target)
    expecttest.assert_expected_inline(
        anonymize_ir(module.str_nodebug()), """\
#mma = #ttg.amd_wmma<{version = 2, isTranspose = true, ctaLayout = {warp = [[1, 0], [2, 0]]}}>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "...", "ttg.threads-per-warp" = 32 : i32} {
  tt.func public @kernel() attributes {noinline = false} {
    %cst = arith.constant 1.000000e+00 : f16
    %cst_0 = arith.constant dense<1.000000e+00> : tensor<64x64xf16, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 8}>>
    %cst_1 = arith.constant 2.000000e+00 : f16
    %cst_2 = arith.constant dense<2.000000e+00> : tensor<64x64xf16, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 8}>>
    %cst_3 = arith.constant 0.000000e+00 : f32
    %cst_4 = arith.constant dense<0.000000e+00> : tensor<64x64xf32, #mma>
    %cst_5 = arith.constant 0.000000e+00 : f32
    %0 = tt.dot %cst_0, %cst_2, %cst_4 : tensor<64x64xf16, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 8}>> * tensor<64x64xf16, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 8}>> -> tensor<64x64xf32, #mma>
    tt.return
  }
}
""")
```
- **EN:** Prepares or updates state through `module`. Invokes `run_parser`, `expecttest.assert_expected_inline`, `anonymize_ir`, `module.str_nodebug` to execute the test logic. Relevant themes: debugging and inspection paths, layout transformation reasoning.
- **CN:** 通过 `module` 准备或更新状态。 调用 `run_parser`、`expecttest.assert_expected_inline`、`anonymize_ir`、`module.str_nodebug` 执行测试逻辑。 相关主题：调试与检查路径、布局变换推理。

### Lines 2815-2820

```python


@pytest.mark.parametrize("target", [HIP_TARGET_CDNA3, HIP_TARGET_CDNA4])
def test_amd_mfma(target):

    @gluon.jit
```
- **EN:** Defines the test function `test_amd_mfma`. Decorators: `pytest.mark.parametrize('target', [HIP_TARGET_CDNA3, HIP_TARGET_CDNA4])`. Parameters: `target`. Nested definitions in this scope: `kernel`. Key calls include `pytest.mark.parametrize`, `run_parser`, `expecttest.assert_expected_inline`, `ttgl.amd.AMDMFMALayout`, `ttgl.full`, `ttgl.amd.cdna3.mfma`, and 5 more. This scope touches pytest parametrization, debugging and inspection paths, layout transformation reasoning, random-data generation.
- **CN:** 定义测试函数 `test_amd_mfma`。 装饰器：`pytest.mark.parametrize('target', [HIP_TARGET_CDNA3, HIP_TARGET_CDNA4])`。 参数：`target`。 该作用域中的嵌套定义：`kernel`。 关键调用包括 `pytest.mark.parametrize`、`run_parser`、`expecttest.assert_expected_inline`、`ttgl.amd.AMDMFMALayout`、`ttgl.full`、`ttgl.amd.cdna3.mfma` 等另外 5 项。 该作用域涉及pytest 参数化、调试与检查路径、布局变换推理、随机数据生成。

#### Lines 2820-2821

```python
    @gluon.jit
    def kernel():
```
- **EN:** Defines the helper function `kernel`. Decorators: `gluon.jit`. Key calls include `ttgl.amd.AMDMFMALayout`, `ttgl.full`, `ttgl.amd.cdna3.mfma`, `ttgl.static_assert`, `isinstance`, `ttgl.DotOperandLayout`. This scope touches layout transformation reasoning, random-data generation.
- **CN:** 定义辅助函数 `kernel`。 装饰器：`gluon.jit`。 关键调用包括 `ttgl.amd.AMDMFMALayout`、`ttgl.full`、`ttgl.amd.cdna3.mfma`、`ttgl.static_assert`、`isinstance`、`ttgl.DotOperandLayout`。 该作用域涉及布局变换推理、随机数据生成。

##### Lines 2822-2833

```python
        mfma_layout: ttgl.constexpr = ttgl.amd.AMDMFMALayout(version=3, warps_per_cta=[4, 1], instr_shape=[32, 32, 8],
                                                             transposed=True)

        a = ttgl.full([64, 32], 1.0, ttgl.float32, layout=ttgl.DotOperandLayout(operand_index=0, parent=mfma_layout,
                                                                                k_width=8))
        b = ttgl.full([32, 64], 2.0, ttgl.float32, layout=ttgl.DotOperandLayout(operand_index=1, parent=mfma_layout,
                                                                                k_width=8))

        acc = ttgl.full([64, 64], 0.0, ttgl.float32, layout=mfma_layout)
        acc = ttgl.amd.cdna3.mfma(a, b, acc)
        ttgl.static_assert(isinstance(acc, ttgl.tensor))
        ttgl.static_assert(acc.type.layout == mfma_layout)
```
- **EN:** Prepares or updates state through `mfma_layout`, `a`, `b`, `acc`. Invokes `ttgl.amd.AMDMFMALayout`, `ttgl.full`, `ttgl.DotOperandLayout`, `ttgl.amd.cdna3.mfma`, `ttgl.static_assert`, `isinstance` to execute the test logic. Relevant themes: layout transformation reasoning, random-data generation.
- **CN:** 通过 `mfma_layout`、`a`、`b`、`acc` 准备或更新状态。 调用 `ttgl.amd.AMDMFMALayout`、`ttgl.full`、`ttgl.DotOperandLayout`、`ttgl.amd.cdna3.mfma`、`ttgl.static_assert`、`isinstance` 执行测试逻辑。 相关主题：布局变换推理、随机数据生成。

#### Lines 2834-2852

```python

    module = run_parser(kernel, target=target)
    expecttest.assert_expected_inline(
        anonymize_ir(module.str_nodebug()), """\
#mma = #ttg.amd_mfma<{version = 3, warpsPerCTA = [4, 1], instrShape = [32, 32, 8], isTransposed = true}>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "...", "ttg.threads-per-warp" = 64 : i32} {
  tt.func public @kernel() attributes {noinline = false} {
    %cst = arith.constant 1.000000e+00 : f32
    %cst_0 = arith.constant dense<1.000000e+00> : tensor<64x32xf32, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 8}>>
    %cst_1 = arith.constant 2.000000e+00 : f32
    %cst_2 = arith.constant dense<2.000000e+00> : tensor<32x64xf32, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 8}>>
    %cst_3 = arith.constant 0.000000e+00 : f32
    %cst_4 = arith.constant dense<0.000000e+00> : tensor<64x64xf32, #mma>
    %cst_5 = arith.constant 0.000000e+00 : f32
    %0 = tt.dot %cst_0, %cst_2, %cst_4 : tensor<64x32xf32, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 8}>> * tensor<32x64xf32, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 8}>> -> tensor<64x64xf32, #mma>
    tt.return
  }
}
""")
```
- **EN:** Prepares or updates state through `module`. Invokes `run_parser`, `expecttest.assert_expected_inline`, `anonymize_ir`, `module.str_nodebug` to execute the test logic. Relevant themes: debugging and inspection paths.
- **CN:** 通过 `module` 准备或更新状态。 调用 `run_parser`、`expecttest.assert_expected_inline`、`anonymize_ir`、`module.str_nodebug` 执行测试逻辑。 相关主题：调试与检查路径。

### Lines 2853-2858

```python


@pytest.mark.parametrize("target", [HIP_TARGET_CDNA4])
def test_amd_mfma_scaled(target):

    @gluon.jit
```
- **EN:** Defines the test function `test_amd_mfma_scaled`. Decorators: `pytest.mark.parametrize('target', [HIP_TARGET_CDNA4])`. Parameters: `target`. Nested definitions in this scope: `kernel`. Key calls include `pytest.mark.parametrize`, `run_parser`, `expecttest.assert_expected_inline`, `ttgl.amd.AMDMFMALayout`, `ttgl.DotOperandLayout`, `ttgl.amd.cdna4.get_mfma_scale_layout`, and 5 more. This scope touches pytest parametrization, debugging and inspection paths, layout transformation reasoning, random-data generation.
- **CN:** 定义测试函数 `test_amd_mfma_scaled`。 装饰器：`pytest.mark.parametrize('target', [HIP_TARGET_CDNA4])`。 参数：`target`。 该作用域中的嵌套定义：`kernel`。 关键调用包括 `pytest.mark.parametrize`、`run_parser`、`expecttest.assert_expected_inline`、`ttgl.amd.AMDMFMALayout`、`ttgl.DotOperandLayout`、`ttgl.amd.cdna4.get_mfma_scale_layout` 等另外 5 项。 该作用域涉及pytest 参数化、调试与检查路径、布局变换推理、随机数据生成。

#### Lines 2858-2859

```python
    @gluon.jit
    def kernel():
```
- **EN:** Defines the helper function `kernel`. Decorators: `gluon.jit`. Key calls include `ttgl.amd.AMDMFMALayout`, `ttgl.DotOperandLayout`, `ttgl.amd.cdna4.get_mfma_scale_layout`, `ttgl.full`, `ttgl.amd.cdna4.mfma_scaled`. This scope touches layout transformation reasoning, random-data generation.
- **CN:** 定义辅助函数 `kernel`。 装饰器：`gluon.jit`。 关键调用包括 `ttgl.amd.AMDMFMALayout`、`ttgl.DotOperandLayout`、`ttgl.amd.cdna4.get_mfma_scale_layout`、`ttgl.full`、`ttgl.amd.cdna4.mfma_scaled`。 该作用域涉及布局变换推理、随机数据生成。

##### Lines 2860-2872

```python
        mfma_layout: ttgl.constexpr = ttgl.amd.AMDMFMALayout(version=4, instr_shape=[16, 16, 128], transposed=True,
                                                             warps_per_cta=[1, 1])
        a_layout: ttgl.constexpr = ttgl.DotOperandLayout(operand_index=0, parent=mfma_layout, k_width=16)
        b_layout: ttgl.constexpr = ttgl.DotOperandLayout(operand_index=1, parent=mfma_layout, k_width=16)
        a_scale_layout: ttgl.constexpr = ttgl.amd.cdna4.get_mfma_scale_layout(a_layout, [16, 4])
        b_scale_layout: ttgl.constexpr = ttgl.amd.cdna4.get_mfma_scale_layout(b_layout, [16, 4])

        a = ttgl.full([16, 64], 0x11, ttgl.uint8, a_layout)
        b = ttgl.full([64, 16], 0x22, ttgl.uint8, b_layout)
        a_scale = ttgl.full([16, 4], 0x02, ttgl.uint8, a_scale_layout)
        b_scale = ttgl.full([16, 4], 0x01, ttgl.uint8, b_scale_layout)
        acc = ttgl.full([16, 16], 0, ttgl.float32, mfma_layout)
        ttgl.amd.cdna4.mfma_scaled(a, a_scale, 'e2m1', b, b_scale, 'e2m1', acc)
```
- **EN:** Prepares or updates state through `mfma_layout`, `a_layout`, `b_layout`, `a_scale_layout`, `b_scale_layout`, `a`, `b`, `a_scale`, and 2 more. Invokes `ttgl.amd.AMDMFMALayout`, `ttgl.DotOperandLayout`, `ttgl.amd.cdna4.get_mfma_scale_layout`, `ttgl.full`, `ttgl.amd.cdna4.mfma_scaled` to execute the test logic. Relevant themes: layout transformation reasoning, random-data generation.
- **CN:** 通过 `mfma_layout`、`a_layout`、`b_layout`、`a_scale_layout`、`b_scale_layout`、`a`、`b`、`a_scale` 等另外 2 项 准备或更新状态。 调用 `ttgl.amd.AMDMFMALayout`、`ttgl.DotOperandLayout`、`ttgl.amd.cdna4.get_mfma_scale_layout`、`ttgl.full`、`ttgl.amd.cdna4.mfma_scaled` 执行测试逻辑。 相关主题：布局变换推理、随机数据生成。

#### Lines 2873-2896

```python

    module = run_parser(kernel, *make_args(num_warps=1), target=target)
    expecttest.assert_expected_inline(
        anonymize_ir(module.str_nodebug()), """\
#linear = #ttg.linear<{register = [], lane = [[1, 0], [2, 0], [4, 0], [8, 0], [0, 1], [0, 2]], warp = [], block = []}>
#mma = #ttg.amd_mfma<{version = 4, warpsPerCTA = [1, 1], instrShape = [16, 16, 128], isTransposed = true}>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 1 : i32, ttg.target = "...", "ttg.threads-per-warp" = 64 : i32} {
  tt.func public @kernel() attributes {noinline = false} {
    %c17_i8 = arith.constant 17 : i8
    %cst = arith.constant dense<17> : tensor<16x64xi8, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 16}>>
    %c34_i8 = arith.constant 34 : i8
    %cst_0 = arith.constant dense<34> : tensor<64x16xi8, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 16}>>
    %c2_i8 = arith.constant 2 : i8
    %cst_1 = arith.constant dense<2> : tensor<16x4xi8, #linear>
    %c1_i8 = arith.constant 1 : i8
    %cst_2 = arith.constant dense<1> : tensor<16x4xi8, #linear>
    %cst_3 = arith.constant 0.000000e+00 : f32
    %cst_4 = arith.constant dense<0.000000e+00> : tensor<16x16xf32, #mma>
    %cst_5 = arith.constant 0.000000e+00 : f32
    %0 = tt.dot_scaled %cst scale %cst_1, %cst_0 scale %cst_2, %cst_4 lhs = e2m1 rhs = e2m1 {fastMath = false} : tensor<16x64xi8, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 16}>>, tensor<16x4xi8, #linear> * tensor<64x16xi8, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 16}>>, tensor<16x4xi8, #linear> -> tensor<16x16xf32, #mma>
    tt.return
  }
}
""")
```
- **EN:** Prepares or updates state through `module`. Invokes `run_parser`, `make_args`, `expecttest.assert_expected_inline`, `anonymize_ir`, `module.str_nodebug` to execute the test logic. Relevant themes: debugging and inspection paths.
- **CN:** 通过 `module` 准备或更新状态。 调用 `run_parser`、`make_args`、`expecttest.assert_expected_inline`、`anonymize_ir`、`module.str_nodebug` 执行测试逻辑。 相关主题：调试与检查路径。

### Lines 2897-2902

```python


@pytest.mark.parametrize("target", [HIP_TARGET_CDNA4])
def test_amd_mfma_scaled_none(target):

    @gluon.jit
```
- **EN:** Defines the test function `test_amd_mfma_scaled_none`. Decorators: `pytest.mark.parametrize('target', [HIP_TARGET_CDNA4])`. Parameters: `target`. Nested definitions in this scope: `kernel`. Key calls include `pytest.mark.parametrize`, `run_parser`, `expecttest.assert_expected_inline`, `ttgl.amd.AMDMFMALayout`, `ttgl.full`, `ttgl.amd.cdna4.mfma_scaled`, and 4 more. This scope touches pytest parametrization, debugging and inspection paths, layout transformation reasoning, random-data generation.
- **CN:** 定义测试函数 `test_amd_mfma_scaled_none`。 装饰器：`pytest.mark.parametrize('target', [HIP_TARGET_CDNA4])`。 参数：`target`。 该作用域中的嵌套定义：`kernel`。 关键调用包括 `pytest.mark.parametrize`、`run_parser`、`expecttest.assert_expected_inline`、`ttgl.amd.AMDMFMALayout`、`ttgl.full`、`ttgl.amd.cdna4.mfma_scaled` 等另外 4 项。 该作用域涉及pytest 参数化、调试与检查路径、布局变换推理、随机数据生成。

#### Lines 2902-2903

```python
    @gluon.jit
    def kernel():
```
- **EN:** Defines the helper function `kernel`. Decorators: `gluon.jit`. Key calls include `ttgl.amd.AMDMFMALayout`, `ttgl.full`, `ttgl.amd.cdna4.mfma_scaled`, `ttgl.DotOperandLayout`. This scope touches layout transformation reasoning, random-data generation.
- **CN:** 定义辅助函数 `kernel`。 装饰器：`gluon.jit`。 关键调用包括 `ttgl.amd.AMDMFMALayout`、`ttgl.full`、`ttgl.amd.cdna4.mfma_scaled`、`ttgl.DotOperandLayout`。 该作用域涉及布局变换推理、随机数据生成。

##### Lines 2904-2908

```python
        mfma_layout: ttgl.constexpr = ttgl.amd.AMDMFMALayout(4, [16, 16, 128], True, [1, 1])
        a = ttgl.full([16, 64], 0x11, ttgl.uint8, ttgl.DotOperandLayout(0, mfma_layout, 16))
        b = ttgl.full([64, 16], 0x22, ttgl.uint8, ttgl.DotOperandLayout(1, mfma_layout, 16))
        acc = ttgl.full([16, 16], 0, ttgl.float32, mfma_layout)
        ttgl.amd.cdna4.mfma_scaled(a, None, 'e2m1', b, None, 'e2m1', acc)
```
- **EN:** Prepares or updates state through `mfma_layout`, `a`, `b`, `acc`. Invokes `ttgl.amd.AMDMFMALayout`, `ttgl.full`, `ttgl.DotOperandLayout`, `ttgl.amd.cdna4.mfma_scaled` to execute the test logic. Relevant themes: layout transformation reasoning, random-data generation.
- **CN:** 通过 `mfma_layout`、`a`、`b`、`acc` 准备或更新状态。 调用 `ttgl.amd.AMDMFMALayout`、`ttgl.full`、`ttgl.DotOperandLayout`、`ttgl.amd.cdna4.mfma_scaled` 执行测试逻辑。 相关主题：布局变换推理、随机数据生成。

#### Lines 2909-2932

```python

    module = run_parser(kernel, *make_args(num_warps=1), target=target)
    expecttest.assert_expected_inline(
        anonymize_ir(module.str_nodebug()), """\
#linear = #ttg.linear<{register = [], lane = [[1, 0], [2, 0], [4, 0], [8, 0], [0, 1], [0, 2]], warp = [], block = []}>
#mma = #ttg.amd_mfma<{version = 4, warpsPerCTA = [1, 1], instrShape = [16, 16, 128], isTransposed = true}>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 1 : i32, ttg.target = "...", "ttg.threads-per-warp" = 64 : i32} {
  tt.func public @kernel() attributes {noinline = false} {
    %c17_i8 = arith.constant 17 : i8
    %cst = arith.constant dense<17> : tensor<16x64xi8, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 16}>>
    %c34_i8 = arith.constant 34 : i8
    %cst_0 = arith.constant dense<34> : tensor<64x16xi8, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 16}>>
    %cst_1 = arith.constant 0.000000e+00 : f32
    %cst_2 = arith.constant dense<0.000000e+00> : tensor<16x16xf32, #mma>
    %c127_i8 = arith.constant 127 : i8
    %cst_3 = arith.constant dense<127> : tensor<16x4xi8, #linear>
    %c127_i8_4 = arith.constant 127 : i8
    %cst_5 = arith.constant dense<127> : tensor<16x4xi8, #linear>
    %cst_6 = arith.constant 0.000000e+00 : f32
    %0 = tt.dot_scaled %cst scale %cst_3, %cst_0 scale %cst_5, %cst_2 lhs = e2m1 rhs = e2m1 {fastMath = false} : tensor<16x64xi8, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 16}>>, tensor<16x4xi8, #linear> * tensor<64x16xi8, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 16}>>, tensor<16x4xi8, #linear> -> tensor<16x16xf32, #mma>
    tt.return
  }
}
""")
```
- **EN:** Prepares or updates state through `module`. Invokes `run_parser`, `make_args`, `expecttest.assert_expected_inline`, `anonymize_ir`, `module.str_nodebug` to execute the test logic. Relevant themes: debugging and inspection paths.
- **CN:** 通过 `module` 准备或更新状态。 调用 `run_parser`、`make_args`、`expecttest.assert_expected_inline`、`anonymize_ir`、`module.str_nodebug` 执行测试逻辑。 相关主题：调试与检查路径。

### Lines 2933-2938

```python


@pytest.mark.parametrize("target", [HIP_TARGET_CDNA4])
def test_amd_mfma_scaled_scalar(target):

    @gluon.jit
```
- **EN:** Defines the test function `test_amd_mfma_scaled_scalar`. Decorators: `pytest.mark.parametrize('target', [HIP_TARGET_CDNA4])`. Parameters: `target`. Nested definitions in this scope: `kernel`. Key calls include `pytest.mark.parametrize`, `run_parser`, `expecttest.assert_expected_inline`, `ttgl.amd.AMDMFMALayout`, `ttgl.full`, `ttgl.amd.cdna4.mfma_scaled`, and 4 more. This scope touches pytest parametrization, debugging and inspection paths, layout transformation reasoning, random-data generation.
- **CN:** 定义测试函数 `test_amd_mfma_scaled_scalar`。 装饰器：`pytest.mark.parametrize('target', [HIP_TARGET_CDNA4])`。 参数：`target`。 该作用域中的嵌套定义：`kernel`。 关键调用包括 `pytest.mark.parametrize`、`run_parser`、`expecttest.assert_expected_inline`、`ttgl.amd.AMDMFMALayout`、`ttgl.full`、`ttgl.amd.cdna4.mfma_scaled` 等另外 4 项。 该作用域涉及pytest 参数化、调试与检查路径、布局变换推理、随机数据生成。

#### Lines 2938-2939

```python
    @gluon.jit
    def kernel():
```
- **EN:** Defines the helper function `kernel`. Decorators: `gluon.jit`. Key calls include `ttgl.amd.AMDMFMALayout`, `ttgl.full`, `ttgl.amd.cdna4.mfma_scaled`, `ttgl.DotOperandLayout`. This scope touches layout transformation reasoning, random-data generation.
- **CN:** 定义辅助函数 `kernel`。 装饰器：`gluon.jit`。 关键调用包括 `ttgl.amd.AMDMFMALayout`、`ttgl.full`、`ttgl.amd.cdna4.mfma_scaled`、`ttgl.DotOperandLayout`。 该作用域涉及布局变换推理、随机数据生成。

##### Lines 2940-2944

```python
        mfma_layout: ttgl.constexpr = ttgl.amd.AMDMFMALayout(4, [16, 16, 128], True, [1, 1])
        a = ttgl.full([16, 64], 0x11, ttgl.uint8, ttgl.DotOperandLayout(0, mfma_layout, 16))
        b = ttgl.full([64, 16], 0x22, ttgl.uint8, ttgl.DotOperandLayout(1, mfma_layout, 16))
        acc = ttgl.full([16, 16], 0, ttgl.float32, mfma_layout)
        ttgl.amd.cdna4.mfma_scaled(a, 0x02, 'e2m1', b, 0x01, 'e2m1', acc)
```
- **EN:** Prepares or updates state through `mfma_layout`, `a`, `b`, `acc`. Invokes `ttgl.amd.AMDMFMALayout`, `ttgl.full`, `ttgl.DotOperandLayout`, `ttgl.amd.cdna4.mfma_scaled` to execute the test logic. Relevant themes: layout transformation reasoning, random-data generation.
- **CN:** 通过 `mfma_layout`、`a`、`b`、`acc` 准备或更新状态。 调用 `ttgl.amd.AMDMFMALayout`、`ttgl.full`、`ttgl.DotOperandLayout`、`ttgl.amd.cdna4.mfma_scaled` 执行测试逻辑。 相关主题：布局变换推理、随机数据生成。

#### Lines 2945-2968

```python

    module = run_parser(kernel, *make_args(num_warps=1), target=target)
    expecttest.assert_expected_inline(
        anonymize_ir(module.str_nodebug()), """\
#linear = #ttg.linear<{register = [], lane = [[1, 0], [2, 0], [4, 0], [8, 0], [0, 1], [0, 2]], warp = [], block = []}>
#mma = #ttg.amd_mfma<{version = 4, warpsPerCTA = [1, 1], instrShape = [16, 16, 128], isTransposed = true}>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 1 : i32, ttg.target = "...", "ttg.threads-per-warp" = 64 : i32} {
  tt.func public @kernel() attributes {noinline = false} {
    %c17_i8 = arith.constant 17 : i8
    %cst = arith.constant dense<17> : tensor<16x64xi8, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 16}>>
    %c34_i8 = arith.constant 34 : i8
    %cst_0 = arith.constant dense<34> : tensor<64x16xi8, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 16}>>
    %cst_1 = arith.constant 0.000000e+00 : f32
    %cst_2 = arith.constant dense<0.000000e+00> : tensor<16x16xf32, #mma>
    %c2_i8 = arith.constant 2 : i8
    %cst_3 = arith.constant dense<2> : tensor<16x4xi8, #linear>
    %c1_i8 = arith.constant 1 : i8
    %cst_4 = arith.constant dense<1> : tensor<16x4xi8, #linear>
    %cst_5 = arith.constant 0.000000e+00 : f32
    %0 = tt.dot_scaled %cst scale %cst_3, %cst_0 scale %cst_4, %cst_2 lhs = e2m1 rhs = e2m1 {fastMath = false} : tensor<16x64xi8, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 16}>>, tensor<16x4xi8, #linear> * tensor<64x16xi8, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 16}>>, tensor<16x4xi8, #linear> -> tensor<16x16xf32, #mma>
    tt.return
  }
}
""")
```
- **EN:** Prepares or updates state through `module`. Invokes `run_parser`, `make_args`, `expecttest.assert_expected_inline`, `anonymize_ir`, `module.str_nodebug` to execute the test logic. Relevant themes: debugging and inspection paths.
- **CN:** 通过 `module` 准备或更新状态。 调用 `run_parser`、`make_args`、`expecttest.assert_expected_inline`、`anonymize_ir`、`module.str_nodebug` 执行测试逻辑。 相关主题：调试与检查路径。

### Lines 2969-2972

```python


@pytest.mark.parametrize("target", [HIP_TARGET_CDNA3, HIP_TARGET_CDNA4], ids=["cdna3", "cdna4"])
def test_amd_scaled_upcast_fp4_cdna(target):
```
- **EN:** Defines the test function `test_amd_scaled_upcast_fp4_cdna`. Decorators: `pytest.mark.parametrize('target', [HIP_TARGET_CDNA3, HIP_TARGET_CDNA4], ids=['cdna3', 'cdna4'])`. Parameters: `target`. Nested definitions in this scope: `kernel`. Key calls include `pytest.mark.parametrize`, `_get_amd_scaled_upcast`, `run_parser`, `expecttest.assert_expected_inline`, `ttgl.BlockedLayout`, `ttgl.full`, and 4 more. This scope touches pytest parametrization, debugging and inspection paths, layout transformation reasoning.
- **CN:** 定义测试函数 `test_amd_scaled_upcast_fp4_cdna`。 装饰器：`pytest.mark.parametrize('target', [HIP_TARGET_CDNA3, HIP_TARGET_CDNA4], ids=['cdna3', 'cdna4'])`。 参数：`target`。 该作用域中的嵌套定义：`kernel`。 关键调用包括 `pytest.mark.parametrize`、`_get_amd_scaled_upcast`、`run_parser`、`expecttest.assert_expected_inline`、`ttgl.BlockedLayout`、`ttgl.full` 等另外 4 项。 该作用域涉及pytest 参数化、调试与检查路径、布局变换推理。

#### Lines 2973-2973

```python
    scaled_upcast = _get_amd_scaled_upcast(target)
```
- **EN:** Prepares or updates state through `scaled_upcast`. Invokes `_get_amd_scaled_upcast` to execute the test logic.
- **CN:** 通过 `scaled_upcast` 准备或更新状态。 调用 `_get_amd_scaled_upcast` 执行测试逻辑。

#### Lines 2974-2976

```python

    @gluon.jit
    def kernel():
```
- **EN:** Defines the helper function `kernel`. Decorators: `gluon.jit`. Key calls include `ttgl.BlockedLayout`, `ttgl.full`, `scaled_upcast`. This scope touches layout transformation reasoning.
- **CN:** 定义辅助函数 `kernel`。 装饰器：`gluon.jit`。 关键调用包括 `ttgl.BlockedLayout`、`ttgl.full`、`scaled_upcast`。 该作用域涉及布局变换推理。

##### Lines 2977-2981

```python
        packed_layout: ttgl.constexpr = ttgl.BlockedLayout([1, 4], [8, 8], [1, 1], [1, 0])
        unpacked_layout: ttgl.constexpr = ttgl.BlockedLayout([1, 8], [8, 8], [1, 1], [1, 0])
        src = ttgl.full([16, 32], 0x11, ttgl.uint8, packed_layout)
        scale = ttgl.full([16, 64], 0x02, ttgl.uint8, unpacked_layout)
        scaled_upcast(src, scale, ttgl.bfloat16, axis=1)
```
- **EN:** Prepares or updates state through `packed_layout`, `unpacked_layout`, `src`, `scale`. Invokes `ttgl.BlockedLayout`, `ttgl.full`, `scaled_upcast` to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `packed_layout`、`unpacked_layout`、`src`、`scale` 准备或更新状态。 调用 `ttgl.BlockedLayout`、`ttgl.full`、`scaled_upcast` 执行测试逻辑。 相关主题：布局变换推理。

#### Lines 2982-3004

```python

    module = run_parser(kernel, *make_args(num_warps=1), target=target)
    expecttest.assert_expected_inline(
        anonymize_ir(module.str_nodebug()), """\
#blocked = #ttg.blocked<{sizePerThread = [1, 4], threadsPerWarp = [8, 8], warpsPerCTA = [1, 1], order = [1, 0]}>
#blocked1 = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [8, 8], warpsPerCTA = [1, 1], order = [1, 0]}>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 1 : i32, ttg.target = "...", "ttg.threads-per-warp" = 64 : i32} {
  tt.func public @kernel() attributes {noinline = false} {
    %c17_i8 = arith.constant 17 : i8
    %cst = arith.constant dense<17> : tensor<16x32xi8, #blocked>
    %c2_i8 = arith.constant 2 : i8
    %cst_0 = arith.constant dense<2> : tensor<16x64xi8, #blocked1>
    %0 = arith.extui %cst_0 : tensor<16x64xi8, #blocked1> to tensor<16x64xi16, #blocked1>
    %c7_i32 = arith.constant 7 : i32
    %1 = arith.trunci %c7_i32 : i32 to i16
    %2 = tt.splat %1 : i16 -> tensor<16x64xi16, #blocked1>
    %3 = arith.shli %0, %2 : tensor<16x64xi16, #blocked1>
    %4 = tt.bitcast %3 : tensor<16x64xi16, #blocked1> -> tensor<16x64xbf16, #blocked1>
    %5 = amdg.scaled_upcast_fp4 %cst scale %4 {axis = 1 : i32} : tensor<16x32xi8, #blocked>, tensor<16x64xbf16, #blocked1> -> tensor<16x64xbf16, #blocked1>
    tt.return
  }
}
""")
```
- **EN:** Prepares or updates state through `module`. Invokes `run_parser`, `make_args`, `expecttest.assert_expected_inline`, `anonymize_ir`, `module.str_nodebug` to execute the test logic. Relevant themes: debugging and inspection paths.
- **CN:** 通过 `module` 准备或更新状态。 调用 `run_parser`、`make_args`、`expecttest.assert_expected_inline`、`anonymize_ir`、`module.str_nodebug` 执行测试逻辑。 相关主题：调试与检查路径。

### Lines 3005-3008

```python


@pytest.mark.parametrize("target", [HIP_TARGET_CDNA3, HIP_TARGET_CDNA4], ids=["cdna3", "cdna4"])
def test_amd_scaled_upcast_fp8_cdna(target):
```
- **EN:** Defines the test function `test_amd_scaled_upcast_fp8_cdna`. Decorators: `pytest.mark.parametrize('target', [HIP_TARGET_CDNA3, HIP_TARGET_CDNA4], ids=['cdna3', 'cdna4'])`. Parameters: `target`. Nested definitions in this scope: `kernel`. Key calls include `pytest.mark.parametrize`, `_get_amd_scaled_upcast`, `run_parser`, `expecttest.assert_expected_inline`, `ttgl.BlockedLayout`, `ttgl.full`, and 4 more. This scope touches pytest parametrization, debugging and inspection paths, layout transformation reasoning.
- **CN:** 定义测试函数 `test_amd_scaled_upcast_fp8_cdna`。 装饰器：`pytest.mark.parametrize('target', [HIP_TARGET_CDNA3, HIP_TARGET_CDNA4], ids=['cdna3', 'cdna4'])`。 参数：`target`。 该作用域中的嵌套定义：`kernel`。 关键调用包括 `pytest.mark.parametrize`、`_get_amd_scaled_upcast`、`run_parser`、`expecttest.assert_expected_inline`、`ttgl.BlockedLayout`、`ttgl.full` 等另外 4 项。 该作用域涉及pytest 参数化、调试与检查路径、布局变换推理。

#### Lines 3009-3009

```python
    scaled_upcast = _get_amd_scaled_upcast(target)
```
- **EN:** Prepares or updates state through `scaled_upcast`. Invokes `_get_amd_scaled_upcast` to execute the test logic.
- **CN:** 通过 `scaled_upcast` 准备或更新状态。 调用 `_get_amd_scaled_upcast` 执行测试逻辑。

#### Lines 3010-3012

```python

    @gluon.jit
    def kernel():
```
- **EN:** Defines the helper function `kernel`. Decorators: `gluon.jit`. Key calls include `ttgl.BlockedLayout`, `ttgl.full`, `scaled_upcast`. This scope touches layout transformation reasoning.
- **CN:** 定义辅助函数 `kernel`。 装饰器：`gluon.jit`。 关键调用包括 `ttgl.BlockedLayout`、`ttgl.full`、`scaled_upcast`。 该作用域涉及布局变换推理。

##### Lines 3013-3016

```python
        layout: ttgl.constexpr = ttgl.BlockedLayout([1, 8], [8, 8], [1, 1], [1, 0])
        src = ttgl.full([16, 64], 1.0, ttgl.float8e4nv, layout)
        scale = ttgl.full([16, 64], 0x02, ttgl.uint8, layout)
        scaled_upcast(src, scale, ttgl.bfloat16)
```
- **EN:** Prepares or updates state through `layout`, `src`, `scale`. Invokes `ttgl.BlockedLayout`, `ttgl.full`, `scaled_upcast` to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `layout`、`src`、`scale` 准备或更新状态。 调用 `ttgl.BlockedLayout`、`ttgl.full`、`scaled_upcast` 执行测试逻辑。 相关主题：布局变换推理。

#### Lines 3017-3039

```python

    module = run_parser(kernel, *make_args(num_warps=1), target=target)
    expecttest.assert_expected_inline(
        anonymize_ir(module.str_nodebug()), """\
#blocked = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [8, 8], warpsPerCTA = [1, 1], order = [1, 0]}>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 1 : i32, ttg.target = "...", "ttg.threads-per-warp" = 64 : i32} {
  tt.func public @kernel() attributes {noinline = false} {
    %cst = arith.constant 1.000000e+00 : f32
    %0 = arith.truncf %cst : f32 to f8E4M3FN
    %1 = tt.splat %0 : f8E4M3FN -> tensor<16x64xf8E4M3FN, #blocked>
    %c2_i8 = arith.constant 2 : i8
    %cst_0 = arith.constant dense<2> : tensor<16x64xi8, #blocked>
    %2 = arith.extui %cst_0 : tensor<16x64xi8, #blocked> to tensor<16x64xi16, #blocked>
    %c7_i32 = arith.constant 7 : i32
    %3 = arith.trunci %c7_i32 : i32 to i16
    %4 = tt.splat %3 : i16 -> tensor<16x64xi16, #blocked>
    %5 = arith.shli %2, %4 : tensor<16x64xi16, #blocked>
    %6 = tt.bitcast %5 : tensor<16x64xi16, #blocked> -> tensor<16x64xbf16, #blocked>
    %7 = amdg.scaled_upcast_fp8 %1 scale %6 : tensor<16x64xf8E4M3FN, #blocked>, tensor<16x64xbf16, #blocked> -> tensor<16x64xbf16, #blocked>
    tt.return
  }
}
""")
```
- **EN:** Prepares or updates state through `module`. Invokes `run_parser`, `make_args`, `expecttest.assert_expected_inline`, `anonymize_ir`, `module.str_nodebug` to execute the test logic. Relevant themes: debugging and inspection paths.
- **CN:** 通过 `module` 准备或更新状态。 调用 `run_parser`、`make_args`、`expecttest.assert_expected_inline`、`anonymize_ir`、`module.str_nodebug` 执行测试逻辑。 相关主题：调试与检查路径。

### Lines 3040-3042

```python


def _get_amd_scaled_upcast(target):
```
- **EN:** Defines the helper function `_get_amd_scaled_upcast`. Parameters: `target`.
- **CN:** 定义辅助函数 `_get_amd_scaled_upcast`。 参数：`target`。

#### Lines 3043-3044

```python
    if target == HIP_TARGET_CDNA3:
        return ttgl.amd.cdna3.scaled_upcast
```
- **EN:** Branches on runtime or test conditions.
- **CN:** 根据运行时或测试条件进行分支。

#### Lines 3045-3046

```python
    if target == HIP_TARGET_CDNA4:
        return ttgl.amd.cdna4.scaled_upcast
```
- **EN:** Branches on runtime or test conditions.
- **CN:** 根据运行时或测试条件进行分支。

#### Lines 3047-3047

```python
    return ttgl.amd.gfx1250.scaled_upcast
```
- **EN:** Carries supporting logic for the surrounding test or helper scope.
- **CN:** 为周围的测试或辅助作用域提供支撑逻辑。

### Lines 3048-3051

```python


@pytest.mark.parametrize("target", [HIP_TARGET_CDNA3, HIP_TARGET_CDNA4, HIP_TARGET_GFX1250])
def test_amd_scaled_upcast_requires_e8m0_scale(target):
```
- **EN:** Defines the test function `test_amd_scaled_upcast_requires_e8m0_scale`. Decorators: `pytest.mark.parametrize('target', [HIP_TARGET_CDNA3, HIP_TARGET_CDNA4, HIP_TARGET_GFX1250])`. Parameters: `target`. Nested definitions in this scope: `kernel`. Key calls include `pytest.mark.parametrize`, `_get_amd_scaled_upcast`, `ttgl.BlockedLayout`, `ttgl.full`, `scaled_upcast`, `pytest.raises`, and 1 more. This scope touches pytest parametrization, layout transformation reasoning.
- **CN:** 定义测试函数 `test_amd_scaled_upcast_requires_e8m0_scale`。 装饰器：`pytest.mark.parametrize('target', [HIP_TARGET_CDNA3, HIP_TARGET_CDNA4, HIP_TARGET_GFX1250])`。 参数：`target`。 该作用域中的嵌套定义：`kernel`。 关键调用包括 `pytest.mark.parametrize`、`_get_amd_scaled_upcast`、`ttgl.BlockedLayout`、`ttgl.full`、`scaled_upcast`、`pytest.raises` 等另外 1 项。 该作用域涉及pytest 参数化、布局变换推理。

#### Lines 3052-3052

```python
    scaled_upcast = _get_amd_scaled_upcast(target)
```
- **EN:** Prepares or updates state through `scaled_upcast`. Invokes `_get_amd_scaled_upcast` to execute the test logic.
- **CN:** 通过 `scaled_upcast` 准备或更新状态。 调用 `_get_amd_scaled_upcast` 执行测试逻辑。

#### Lines 3053-3055

```python

    @gluon.jit
    def kernel():
```
- **EN:** Defines the helper function `kernel`. Decorators: `gluon.jit`. Key calls include `ttgl.BlockedLayout`, `ttgl.full`, `scaled_upcast`. This scope touches layout transformation reasoning.
- **CN:** 定义辅助函数 `kernel`。 装饰器：`gluon.jit`。 关键调用包括 `ttgl.BlockedLayout`、`ttgl.full`、`scaled_upcast`。 该作用域涉及布局变换推理。

##### Lines 3056-3059

```python
        layout: ttgl.constexpr = ttgl.BlockedLayout([1, 8], [8, 4], [4, 1], [1, 0])
        src = ttgl.full([16, 64], 1.0, ttgl.float8e4nv, layout)
        scale = ttgl.full([16, 64], 1.0, ttgl.float8e4nv, layout)
        scaled_upcast(src, scale, ttgl.bfloat16)
```
- **EN:** Prepares or updates state through `layout`, `src`, `scale`. Invokes `ttgl.BlockedLayout`, `ttgl.full`, `scaled_upcast` to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `layout`、`src`、`scale` 准备或更新状态。 调用 `ttgl.BlockedLayout`、`ttgl.full`、`scaled_upcast` 执行测试逻辑。 相关主题：布局变换推理。

#### Lines 3060-3062

```python

    with pytest.raises(CompilationError) as e:
        run_parser(kernel, target=target)
```
- **EN:** Invokes `pytest.raises`, `run_parser` to execute the test logic. Uses `pytest.raises` to confirm expected failure paths. Uses context managers to control resources or expectations.
- **CN:** 调用 `pytest.raises`、`run_parser` 执行测试逻辑。 使用 `pytest.raises` 确认预期的失败路径。 使用上下文管理器控制资源或预期行为。

#### Lines 3063-3064

```python

    err = str(e.value.__cause__ or e.value)
```
- **EN:** Prepares or updates state through `err`.
- **CN:** 通过 `err` 准备或更新状态。

#### Lines 3065-3065

```python
    assert "Expected scale to use raw E8M0 payload in int8/uint8" in err
```
- **EN:** Validates behavior with 1 assertion(s).
- **CN:** 通过 1 个断言验证行为。

### Lines 3066-3069

```python


@pytest.mark.parametrize("target", [HIP_TARGET_CDNA3, HIP_TARGET_CDNA4, HIP_TARGET_GFX1250])
def test_amd_scaled_upcast_requires_fp16_or_bf16_result_type(target):
```
- **EN:** Defines the test function `test_amd_scaled_upcast_requires_fp16_or_bf16_result_type`. Decorators: `pytest.mark.parametrize('target', [HIP_TARGET_CDNA3, HIP_TARGET_CDNA4, HIP_TARGET_GFX1250])`. Parameters: `target`. Nested definitions in this scope: `kernel`. Key calls include `pytest.mark.parametrize`, `_get_amd_scaled_upcast`, `ttgl.BlockedLayout`, `ttgl.full`, `scaled_upcast`, `pytest.raises`, and 1 more. This scope touches pytest parametrization, layout transformation reasoning.
- **CN:** 定义测试函数 `test_amd_scaled_upcast_requires_fp16_or_bf16_result_type`。 装饰器：`pytest.mark.parametrize('target', [HIP_TARGET_CDNA3, HIP_TARGET_CDNA4, HIP_TARGET_GFX1250])`。 参数：`target`。 该作用域中的嵌套定义：`kernel`。 关键调用包括 `pytest.mark.parametrize`、`_get_amd_scaled_upcast`、`ttgl.BlockedLayout`、`ttgl.full`、`scaled_upcast`、`pytest.raises` 等另外 1 项。 该作用域涉及pytest 参数化、布局变换推理。

#### Lines 3070-3070

```python
    scaled_upcast = _get_amd_scaled_upcast(target)
```
- **EN:** Prepares or updates state through `scaled_upcast`. Invokes `_get_amd_scaled_upcast` to execute the test logic.
- **CN:** 通过 `scaled_upcast` 准备或更新状态。 调用 `_get_amd_scaled_upcast` 执行测试逻辑。

#### Lines 3071-3073

```python

    @gluon.jit
    def kernel():
```
- **EN:** Defines the helper function `kernel`. Decorators: `gluon.jit`. Key calls include `ttgl.BlockedLayout`, `ttgl.full`, `scaled_upcast`. This scope touches layout transformation reasoning.
- **CN:** 定义辅助函数 `kernel`。 装饰器：`gluon.jit`。 关键调用包括 `ttgl.BlockedLayout`、`ttgl.full`、`scaled_upcast`。 该作用域涉及布局变换推理。

##### Lines 3074-3077

```python
        layout: ttgl.constexpr = ttgl.BlockedLayout([1, 8], [8, 4], [4, 1], [1, 0])
        src = ttgl.full([16, 64], 1.0, ttgl.float8e4nv, layout)
        scale = ttgl.full([16, 64], 0x02, ttgl.uint8, layout)
        scaled_upcast(src, scale, ttgl.float32)
```
- **EN:** Prepares or updates state through `layout`, `src`, `scale`. Invokes `ttgl.BlockedLayout`, `ttgl.full`, `scaled_upcast` to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `layout`、`src`、`scale` 准备或更新状态。 调用 `ttgl.BlockedLayout`、`ttgl.full`、`scaled_upcast` 执行测试逻辑。 相关主题：布局变换推理。

#### Lines 3078-3080

```python

    with pytest.raises(CompilationError) as e:
        run_parser(kernel, target=target)
```
- **EN:** Invokes `pytest.raises`, `run_parser` to execute the test logic. Uses `pytest.raises` to confirm expected failure paths. Uses context managers to control resources or expectations.
- **CN:** 调用 `pytest.raises`、`run_parser` 执行测试逻辑。 使用 `pytest.raises` 确认预期的失败路径。 使用上下文管理器控制资源或预期行为。

#### Lines 3081-3082

```python

    err = str(e.value.__cause__ or e.value)
```
- **EN:** Prepares or updates state through `err`.
- **CN:** 通过 `err` 准备或更新状态。

#### Lines 3083-3083

```python
    assert "Expected elem_type to be fp16 or bf16" in err
```
- **EN:** Validates behavior with 1 assertion(s).
- **CN:** 通过 1 个断言验证行为。

### Lines 3084-3087

```python


@pytest.mark.parametrize("target", [HIP_TARGET_CDNA3, HIP_TARGET_CDNA4, HIP_TARGET_GFX1250])
def test_amd_scaled_upcast_fp8_requires_matching_layout(target):
```
- **EN:** Defines the test function `test_amd_scaled_upcast_fp8_requires_matching_layout`. Decorators: `pytest.mark.parametrize('target', [HIP_TARGET_CDNA3, HIP_TARGET_CDNA4, HIP_TARGET_GFX1250])`. Parameters: `target`. Nested definitions in this scope: `kernel`. Key calls include `pytest.mark.parametrize`, `_get_amd_scaled_upcast`, `ttgl.BlockedLayout`, `ttgl.full`, `scaled_upcast`, `pytest.raises`, and 1 more. This scope touches pytest parametrization, layout transformation reasoning.
- **CN:** 定义测试函数 `test_amd_scaled_upcast_fp8_requires_matching_layout`。 装饰器：`pytest.mark.parametrize('target', [HIP_TARGET_CDNA3, HIP_TARGET_CDNA4, HIP_TARGET_GFX1250])`。 参数：`target`。 该作用域中的嵌套定义：`kernel`。 关键调用包括 `pytest.mark.parametrize`、`_get_amd_scaled_upcast`、`ttgl.BlockedLayout`、`ttgl.full`、`scaled_upcast`、`pytest.raises` 等另外 1 项。 该作用域涉及pytest 参数化、布局变换推理。

#### Lines 3088-3088

```python
    scaled_upcast = _get_amd_scaled_upcast(target)
```
- **EN:** Prepares or updates state through `scaled_upcast`. Invokes `_get_amd_scaled_upcast` to execute the test logic.
- **CN:** 通过 `scaled_upcast` 准备或更新状态。 调用 `_get_amd_scaled_upcast` 执行测试逻辑。

#### Lines 3089-3094

```python
    if target in (HIP_TARGET_CDNA3, HIP_TARGET_CDNA4):
        src_layout = ttgl.BlockedLayout([1, 8], [8, 8], [1, 1], [1, 0])
        scale_layout = ttgl.BlockedLayout([1, 8], [4, 16], [1, 1], [1, 0])
    else:
        src_layout = ttgl.BlockedLayout([1, 8], [8, 4], [4, 1], [1, 0])
        scale_layout = ttgl.BlockedLayout([1, 8], [4, 8], [4, 1], [1, 0])
```
- **EN:** Invokes `ttgl.BlockedLayout` to execute the test logic. Branches on runtime or test conditions. Relevant themes: layout transformation reasoning.
- **CN:** 调用 `ttgl.BlockedLayout` 执行测试逻辑。 根据运行时或测试条件进行分支。 相关主题：布局变换推理。

#### Lines 3095-3097

```python

    @gluon.jit
    def kernel():
```
- **EN:** Defines the helper function `kernel`. Decorators: `gluon.jit`. Key calls include `ttgl.full`, `scaled_upcast`. This scope touches layout transformation reasoning.
- **CN:** 定义辅助函数 `kernel`。 装饰器：`gluon.jit`。 关键调用包括 `ttgl.full`、`scaled_upcast`。 该作用域涉及布局变换推理。

##### Lines 3098-3100

```python
        src = ttgl.full([16, 64], 1.0, ttgl.float8e4nv, src_layout)
        scale = ttgl.full([16, 64], 0x02, ttgl.uint8, scale_layout)
        scaled_upcast(src, scale, ttgl.bfloat16)
```
- **EN:** Prepares or updates state through `src`, `scale`. Invokes `ttgl.full`, `scaled_upcast` to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `src`、`scale` 准备或更新状态。 调用 `ttgl.full`、`scaled_upcast` 执行测试逻辑。 相关主题：布局变换推理。

#### Lines 3101-3103

```python

    with pytest.raises(CompilationError) as e:
        run_parser(kernel, target=target)
```
- **EN:** Invokes `pytest.raises`, `run_parser` to execute the test logic. Uses `pytest.raises` to confirm expected failure paths. Uses context managers to control resources or expectations.
- **CN:** 调用 `pytest.raises`、`run_parser` 执行测试逻辑。 使用 `pytest.raises` 确认预期的失败路径。 使用上下文管理器控制资源或预期行为。

#### Lines 3104-3105

```python

    err = str(e.value.__cause__ or e.value)
```
- **EN:** Prepares or updates state through `err`.
- **CN:** 通过 `err` 准备或更新状态。

#### Lines 3106-3106

```python
    assert "Expected scale layout for fp8 scaled_upcast" in err
```
- **EN:** Validates behavior with 1 assertion(s). Relevant themes: layout transformation reasoning.
- **CN:** 通过 1 个断言验证行为。 相关主题：布局变换推理。

### Lines 3107-3112

```python


@pytest.mark.parametrize("target", [HIP_TARGET_GFX1250])
def test_amd_wmma_scaled(target):

    @gluon.jit
```
- **EN:** Defines the test function `test_amd_wmma_scaled`. Decorators: `pytest.mark.parametrize('target', [HIP_TARGET_GFX1250])`. Parameters: `target`. Nested definitions in this scope: `kernel`. Key calls include `pytest.mark.parametrize`, `run_parser`, `expecttest.assert_expected_inline`, `ttgl.amd.AMDWMMALayout`, `ttgl.DotOperandLayout`, `ttgl.amd.gfx1250.get_wmma_scale_layout`, and 5 more. This scope touches pytest parametrization, debugging and inspection paths, layout transformation reasoning, random-data generation.
- **CN:** 定义测试函数 `test_amd_wmma_scaled`。 装饰器：`pytest.mark.parametrize('target', [HIP_TARGET_GFX1250])`。 参数：`target`。 该作用域中的嵌套定义：`kernel`。 关键调用包括 `pytest.mark.parametrize`、`run_parser`、`expecttest.assert_expected_inline`、`ttgl.amd.AMDWMMALayout`、`ttgl.DotOperandLayout`、`ttgl.amd.gfx1250.get_wmma_scale_layout` 等另外 5 项。 该作用域涉及pytest 参数化、调试与检查路径、布局变换推理、随机数据生成。

#### Lines 3112-3113

```python
    @gluon.jit
    def kernel():
```
- **EN:** Defines the helper function `kernel`. Decorators: `gluon.jit`. Key calls include `ttgl.amd.AMDWMMALayout`, `ttgl.DotOperandLayout`, `ttgl.amd.gfx1250.get_wmma_scale_layout`, `ttgl.full`, `ttgl.amd.gfx1250.wmma_scaled`. This scope touches layout transformation reasoning, random-data generation.
- **CN:** 定义辅助函数 `kernel`。 装饰器：`gluon.jit`。 关键调用包括 `ttgl.amd.AMDWMMALayout`、`ttgl.DotOperandLayout`、`ttgl.amd.gfx1250.get_wmma_scale_layout`、`ttgl.full`、`ttgl.amd.gfx1250.wmma_scaled`。 该作用域涉及布局变换推理、随机数据生成。

##### Lines 3114-3129

```python
        wmma_layout: ttgl.constexpr = ttgl.amd.AMDWMMALayout(version=3, transposed=True, warp_bases=[[0, 1], [1, 0]],
                                                             instr_shape=[16, 16, 128])
        wmma_layout_packed: ttgl.constexpr = ttgl.amd.AMDWMMALayout(version=3, transposed=True, warp_bases=[[0, 1],
                                                                                                            [1, 0]],
                                                                    instr_shape=[16, 16, 64])
        a_layout: ttgl.constexpr = ttgl.DotOperandLayout(operand_index=0, parent=wmma_layout_packed, k_width=16)
        b_layout: ttgl.constexpr = ttgl.DotOperandLayout(operand_index=1, parent=wmma_layout_packed, k_width=16)
        a_scale_layout: ttgl.constexpr = ttgl.amd.gfx1250.get_wmma_scale_layout(a_layout, [32, 4])
        b_scale_layout: ttgl.constexpr = ttgl.amd.gfx1250.get_wmma_scale_layout(b_layout, [32, 4])

        a = ttgl.full([32, 64], 0x11, ttgl.uint8, a_layout)
        b = ttgl.full([64, 32], 0x22, ttgl.uint8, b_layout)
        a_scale = ttgl.full([32, 4], 0x02, ttgl.uint8, a_scale_layout)
        b_scale = ttgl.full([32, 4], 0x01, ttgl.uint8, b_scale_layout)
        acc = ttgl.full([32, 32], 0, ttgl.float32, wmma_layout)
        ttgl.amd.gfx1250.wmma_scaled(a, a_scale, 'e2m1', b, b_scale, 'e2m1', acc)
```
- **EN:** Prepares or updates state through `wmma_layout`, `wmma_layout_packed`, `a_layout`, `b_layout`, `a_scale_layout`, `b_scale_layout`, `a`, `b`, and 3 more. Invokes `ttgl.amd.AMDWMMALayout`, `ttgl.DotOperandLayout`, `ttgl.amd.gfx1250.get_wmma_scale_layout`, `ttgl.full`, `ttgl.amd.gfx1250.wmma_scaled` to execute the test logic. Relevant themes: layout transformation reasoning, random-data generation.
- **CN:** 通过 `wmma_layout`、`wmma_layout_packed`、`a_layout`、`b_layout`、`a_scale_layout`、`b_scale_layout`、`a`、`b` 等另外 3 项 准备或更新状态。 调用 `ttgl.amd.AMDWMMALayout`、`ttgl.DotOperandLayout`、`ttgl.amd.gfx1250.get_wmma_scale_layout`、`ttgl.full`、`ttgl.amd.gfx1250.wmma_scaled` 执行测试逻辑。 相关主题：布局变换推理、随机数据生成。

#### Lines 3130-3155

```python

    module = run_parser(kernel, *make_args(num_warps=4), target=target)
    expecttest.assert_expected_inline(
        anonymize_ir(module.str_nodebug()), """\
#linear = #ttg.linear<{register = [[0, 1], [0, 2]], lane = [[1, 0], [2, 0], [4, 0], [8, 0], [0, 0]], warp = [[0, 0], [16, 0]], block = []}>
#linear1 = #ttg.linear<{register = [[0, 1], [0, 2]], lane = [[1, 0], [2, 0], [4, 0], [8, 0], [0, 0]], warp = [[16, 0], [0, 0]], block = []}>
#mma = #ttg.amd_wmma<{version = 3, isTranspose = true, ctaLayout = {warp = [[0, 1], [1, 0]]}, instrShape = [16, 16, 64]}>
#mma1 = #ttg.amd_wmma<{version = 3, isTranspose = true, ctaLayout = {warp = [[0, 1], [1, 0]]}, instrShape = [16, 16, 128]}>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "...", "ttg.threads-per-warp" = 32 : i32} {
  tt.func public @kernel() attributes {noinline = false} {
    %c17_i8 = arith.constant 17 : i8
    %cst = arith.constant dense<17> : tensor<32x64xi8, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 16}>>
    %c34_i8 = arith.constant 34 : i8
    %cst_0 = arith.constant dense<34> : tensor<64x32xi8, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 16}>>
    %c2_i8 = arith.constant 2 : i8
    %cst_1 = arith.constant dense<2> : tensor<32x4xi8, #linear>
    %c1_i8 = arith.constant 1 : i8
    %cst_2 = arith.constant dense<1> : tensor<32x4xi8, #linear1>
    %cst_3 = arith.constant 0.000000e+00 : f32
    %cst_4 = arith.constant dense<0.000000e+00> : tensor<32x32xf32, #mma1>
    %cst_5 = arith.constant 0.000000e+00 : f32
    %0 = tt.dot_scaled %cst scale %cst_1, %cst_0 scale %cst_2, %cst_4 lhs = e2m1 rhs = e2m1 {fastMath = false} : tensor<32x64xi8, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 16}>>, tensor<32x4xi8, #linear> * tensor<64x32xi8, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 16}>>, tensor<32x4xi8, #linear1> -> tensor<32x32xf32, #mma1>
    tt.return
  }
}
""")
```
- **EN:** Prepares or updates state through `module`. Invokes `run_parser`, `make_args`, `expecttest.assert_expected_inline`, `anonymize_ir`, `module.str_nodebug` to execute the test logic. Relevant themes: debugging and inspection paths, layout transformation reasoning.
- **CN:** 通过 `module` 准备或更新状态。 调用 `run_parser`、`make_args`、`expecttest.assert_expected_inline`、`anonymize_ir`、`module.str_nodebug` 执行测试逻辑。 相关主题：调试与检查路径、布局变换推理。

### Lines 3156-3161

```python


@pytest.mark.parametrize("target", [HIP_TARGET_GFX1250])
def test_amd_wmma_scaled_none(target):

    @gluon.jit
```
- **EN:** Defines the test function `test_amd_wmma_scaled_none`. Decorators: `pytest.mark.parametrize('target', [HIP_TARGET_GFX1250])`. Parameters: `target`. Nested definitions in this scope: `kernel`. Key calls include `pytest.mark.parametrize`, `run_parser`, `expecttest.assert_expected_inline`, `ttgl.amd.AMDWMMALayout`, `ttgl.DotOperandLayout`, `ttgl.full`, and 4 more. This scope touches pytest parametrization, debugging and inspection paths, layout transformation reasoning, random-data generation.
- **CN:** 定义测试函数 `test_amd_wmma_scaled_none`。 装饰器：`pytest.mark.parametrize('target', [HIP_TARGET_GFX1250])`。 参数：`target`。 该作用域中的嵌套定义：`kernel`。 关键调用包括 `pytest.mark.parametrize`、`run_parser`、`expecttest.assert_expected_inline`、`ttgl.amd.AMDWMMALayout`、`ttgl.DotOperandLayout`、`ttgl.full` 等另外 4 项。 该作用域涉及pytest 参数化、调试与检查路径、布局变换推理、随机数据生成。

#### Lines 3161-3162

```python
    @gluon.jit
    def kernel():
```
- **EN:** Defines the helper function `kernel`. Decorators: `gluon.jit`. Key calls include `ttgl.amd.AMDWMMALayout`, `ttgl.DotOperandLayout`, `ttgl.full`, `ttgl.amd.gfx1250.wmma_scaled`. This scope touches layout transformation reasoning, random-data generation.
- **CN:** 定义辅助函数 `kernel`。 装饰器：`gluon.jit`。 关键调用包括 `ttgl.amd.AMDWMMALayout`、`ttgl.DotOperandLayout`、`ttgl.full`、`ttgl.amd.gfx1250.wmma_scaled`。 该作用域涉及布局变换推理、随机数据生成。

##### Lines 3163-3172

```python
        wmma_layout: ttgl.constexpr = ttgl.amd.AMDWMMALayout(3, True, [], [], [16, 16, 128])
        wmma_layout_packed: ttgl.constexpr = ttgl.amd.AMDWMMALayout(3, True, [], [], [16, 16, 64])
        a_layout: ttgl.constexpr = ttgl.DotOperandLayout(0, wmma_layout_packed, 16)
        b_layout: ttgl.constexpr = ttgl.DotOperandLayout(1, wmma_layout_packed, 16)

        a = ttgl.full([16, 64], 0x11, ttgl.uint8, a_layout)
        b = ttgl.full([64, 16], 0x22, ttgl.uint8, b_layout)
        acc = ttgl.full([16, 16], 0, ttgl.float32, wmma_layout)

        ttgl.amd.gfx1250.wmma_scaled(a, None, 'e2m1', b, None, 'e2m1', acc)
```
- **EN:** Prepares or updates state through `wmma_layout`, `wmma_layout_packed`, `a_layout`, `b_layout`, `a`, `b`, `acc`. Invokes `ttgl.amd.AMDWMMALayout`, `ttgl.DotOperandLayout`, `ttgl.full`, `ttgl.amd.gfx1250.wmma_scaled` to execute the test logic. Relevant themes: layout transformation reasoning, random-data generation.
- **CN:** 通过 `wmma_layout`、`wmma_layout_packed`、`a_layout`、`b_layout`、`a`、`b`、`acc` 准备或更新状态。 调用 `ttgl.amd.AMDWMMALayout`、`ttgl.DotOperandLayout`、`ttgl.full`、`ttgl.amd.gfx1250.wmma_scaled` 执行测试逻辑。 相关主题：布局变换推理、随机数据生成。

#### Lines 3173-3197

```python

    module = run_parser(kernel, *make_args(num_warps=1), target=target)
    expecttest.assert_expected_inline(
        anonymize_ir(module.str_nodebug()), """\
#linear = #ttg.linear<{register = [[0, 1], [0, 2]], lane = [[1, 0], [2, 0], [4, 0], [8, 0], [0, 0]], warp = [], block = []}>
#mma = #ttg.amd_wmma<{version = 3, isTranspose = true, ctaLayout = {}, instrShape = [16, 16, 64]}>
#mma1 = #ttg.amd_wmma<{version = 3, isTranspose = true, ctaLayout = {}, instrShape = [16, 16, 128]}>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 1 : i32, ttg.target = "...", "ttg.threads-per-warp" = 32 : i32} {
  tt.func public @kernel() attributes {noinline = false} {
    %c17_i8 = arith.constant 17 : i8
    %cst = arith.constant dense<17> : tensor<16x64xi8, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 16}>>
    %c34_i8 = arith.constant 34 : i8
    %cst_0 = arith.constant dense<34> : tensor<64x16xi8, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 16}>>
    %cst_1 = arith.constant 0.000000e+00 : f32
    %cst_2 = arith.constant dense<0.000000e+00> : tensor<16x16xf32, #mma1>
    %c127_i8 = arith.constant 127 : i8
    %cst_3 = arith.constant dense<127> : tensor<16x4xi8, #linear>
    %c127_i8_4 = arith.constant 127 : i8
    %cst_5 = arith.constant dense<127> : tensor<16x4xi8, #linear>
    %cst_6 = arith.constant 0.000000e+00 : f32
    %0 = tt.dot_scaled %cst scale %cst_3, %cst_0 scale %cst_5, %cst_2 lhs = e2m1 rhs = e2m1 {fastMath = false} : tensor<16x64xi8, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 16}>>, tensor<16x4xi8, #linear> * tensor<64x16xi8, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 16}>>, tensor<16x4xi8, #linear> -> tensor<16x16xf32, #mma1>
    tt.return
  }
}
""")
```
- **EN:** Prepares or updates state through `module`. Invokes `run_parser`, `make_args`, `expecttest.assert_expected_inline`, `anonymize_ir`, `module.str_nodebug` to execute the test logic. Relevant themes: debugging and inspection paths, layout transformation reasoning.
- **CN:** 通过 `module` 准备或更新状态。 调用 `run_parser`、`make_args`、`expecttest.assert_expected_inline`、`anonymize_ir`、`module.str_nodebug` 执行测试逻辑。 相关主题：调试与检查路径、布局变换推理。

### Lines 3198-3203

```python


@pytest.mark.parametrize("target", [HIP_TARGET_GFX1250])
def test_amd_wmma_scaled_scalar(target):

    @gluon.jit
```
- **EN:** Defines the test function `test_amd_wmma_scaled_scalar`. Decorators: `pytest.mark.parametrize('target', [HIP_TARGET_GFX1250])`. Parameters: `target`. Nested definitions in this scope: `kernel`. Key calls include `pytest.mark.parametrize`, `run_parser`, `expecttest.assert_expected_inline`, `ttgl.amd.AMDWMMALayout`, `ttgl.DotOperandLayout`, `ttgl.full`, and 6 more. This scope touches pytest parametrization, debugging and inspection paths, layout transformation reasoning, random-data generation.
- **CN:** 定义测试函数 `test_amd_wmma_scaled_scalar`。 装饰器：`pytest.mark.parametrize('target', [HIP_TARGET_GFX1250])`。 参数：`target`。 该作用域中的嵌套定义：`kernel`。 关键调用包括 `pytest.mark.parametrize`、`run_parser`、`expecttest.assert_expected_inline`、`ttgl.amd.AMDWMMALayout`、`ttgl.DotOperandLayout`、`ttgl.full` 等另外 6 项。 该作用域涉及pytest 参数化、调试与检查路径、布局变换推理、随机数据生成。

#### Lines 3203-3204

```python
    @gluon.jit
    def kernel():
```
- **EN:** Defines the helper function `kernel`. Decorators: `gluon.jit`. Key calls include `ttgl.amd.AMDWMMALayout`, `ttgl.DotOperandLayout`, `ttgl.full`, `ttgl.amd.gfx1250.wmma_scaled`, `a_scale.to`, `b_scale.to`. This scope touches layout transformation reasoning, random-data generation.
- **CN:** 定义辅助函数 `kernel`。 装饰器：`gluon.jit`。 关键调用包括 `ttgl.amd.AMDWMMALayout`、`ttgl.DotOperandLayout`、`ttgl.full`、`ttgl.amd.gfx1250.wmma_scaled`、`a_scale.to`、`b_scale.to`。 该作用域涉及布局变换推理、随机数据生成。

##### Lines 3205-3222

```python
        wmma_layout: ttgl.constexpr = ttgl.amd.AMDWMMALayout(3, True, [], [], [16, 16, 128])
        wmma_layout_packed: ttgl.constexpr = ttgl.amd.AMDWMMALayout(3, True, [], [], [16, 16, 64])
        a_layout: ttgl.constexpr = ttgl.DotOperandLayout(0, wmma_layout_packed, 16)
        b_layout: ttgl.constexpr = ttgl.DotOperandLayout(1, wmma_layout_packed, 16)

        a = ttgl.full([16, 64], 0x11, ttgl.uint8, a_layout)
        b = ttgl.full([64, 16], 0x22, ttgl.uint8, b_layout)
        acc = ttgl.full([16, 16], 0, ttgl.float32, wmma_layout)

        # test constexpr
        ttgl.amd.gfx1250.wmma_scaled(a, 0x02, 'e2m1', b, 0x01, 'e2m1', acc)

        # test scalar value
        a_scale = 0x03
        a_scale = a_scale.to(ttgl.uint8)
        b_scale = 0x04
        b_scale = b_scale.to(ttgl.uint8)
        ttgl.amd.gfx1250.wmma_scaled(a, a_scale, 'e2m1', b, b_scale, 'e2m1', acc)
```
- **EN:** Prepares or updates state through `wmma_layout`, `wmma_layout_packed`, `a_layout`, `b_layout`, `a`, `b`, `acc`, `a_scale`, and 1 more. Invokes `ttgl.amd.AMDWMMALayout`, `ttgl.DotOperandLayout`, `ttgl.full`, `ttgl.amd.gfx1250.wmma_scaled`, `a_scale.to`, `b_scale.to` to execute the test logic. Relevant themes: layout transformation reasoning, random-data generation.
- **CN:** 通过 `wmma_layout`、`wmma_layout_packed`、`a_layout`、`b_layout`、`a`、`b`、`acc`、`a_scale` 等另外 1 项 准备或更新状态。 调用 `ttgl.amd.AMDWMMALayout`、`ttgl.DotOperandLayout`、`ttgl.full`、`ttgl.amd.gfx1250.wmma_scaled`、`a_scale.to`、`b_scale.to` 执行测试逻辑。 相关主题：布局变换推理、随机数据生成。

#### Lines 3223-3255

```python

    module = run_parser(kernel, *make_args(num_warps=1), target=target)
    expecttest.assert_expected_inline(
        anonymize_ir(module.str_nodebug()), """\
#linear = #ttg.linear<{register = [[0, 1], [0, 2]], lane = [[1, 0], [2, 0], [4, 0], [8, 0], [0, 0]], warp = [], block = []}>
#mma = #ttg.amd_wmma<{version = 3, isTranspose = true, ctaLayout = {}, instrShape = [16, 16, 64]}>
#mma1 = #ttg.amd_wmma<{version = 3, isTranspose = true, ctaLayout = {}, instrShape = [16, 16, 128]}>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 1 : i32, ttg.target = "...", "ttg.threads-per-warp" = 32 : i32} {
  tt.func public @kernel() attributes {noinline = false} {
    %c17_i8 = arith.constant 17 : i8
    %cst = arith.constant dense<17> : tensor<16x64xi8, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 16}>>
    %c34_i8 = arith.constant 34 : i8
    %cst_0 = arith.constant dense<34> : tensor<64x16xi8, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 16}>>
    %cst_1 = arith.constant 0.000000e+00 : f32
    %cst_2 = arith.constant dense<0.000000e+00> : tensor<16x16xf32, #mma1>
    %c2_i8 = arith.constant 2 : i8
    %cst_3 = arith.constant dense<2> : tensor<16x4xi8, #linear>
    %c1_i8 = arith.constant 1 : i8
    %cst_4 = arith.constant dense<1> : tensor<16x4xi8, #linear>
    %cst_5 = arith.constant 0.000000e+00 : f32
    %0 = tt.dot_scaled %cst scale %cst_3, %cst_0 scale %cst_4, %cst_2 lhs = e2m1 rhs = e2m1 {fastMath = false} : tensor<16x64xi8, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 16}>>, tensor<16x4xi8, #linear> * tensor<64x16xi8, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 16}>>, tensor<16x4xi8, #linear> -> tensor<16x16xf32, #mma1>
    %c3_i32 = arith.constant 3 : i32
    %1 = arith.trunci %c3_i32 : i32 to i8
    %c4_i32 = arith.constant 4 : i32
    %2 = arith.trunci %c4_i32 : i32 to i8
    %3 = tt.splat %1 : i8 -> tensor<16x4xi8, #linear>
    %4 = tt.splat %2 : i8 -> tensor<16x4xi8, #linear>
    %cst_6 = arith.constant 0.000000e+00 : f32
    %5 = tt.dot_scaled %cst scale %3, %cst_0 scale %4, %cst_2 lhs = e2m1 rhs = e2m1 {fastMath = false} : tensor<16x64xi8, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 16}>>, tensor<16x4xi8, #linear> * tensor<64x16xi8, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 16}>>, tensor<16x4xi8, #linear> -> tensor<16x16xf32, #mma1>
    tt.return
  }
}
""")
```
- **EN:** Prepares or updates state through `module`. Invokes `run_parser`, `make_args`, `expecttest.assert_expected_inline`, `anonymize_ir`, `module.str_nodebug` to execute the test logic. Relevant themes: debugging and inspection paths, layout transformation reasoning.
- **CN:** 通过 `module` 准备或更新状态。 调用 `run_parser`、`make_args`、`expecttest.assert_expected_inline`、`anonymize_ir`、`module.str_nodebug` 执行测试逻辑。 相关主题：调试与检查路径、布局变换推理。

### Lines 3256-3261

```python


@pytest.mark.parametrize("target", [HIP_TARGET_GFX1250])
def test_amd_wmma_scale_layout_for_multicta(target):

    @gluon.jit
```
- **EN:** Defines the test function `test_amd_wmma_scale_layout_for_multicta`. Decorators: `pytest.mark.parametrize('target', [HIP_TARGET_GFX1250])`. Parameters: `target`. Nested definitions in this scope: `kernel`. Key calls include `pytest.mark.parametrize`, `run_parser`, `expecttest.assert_expected_inline`, `ttgl.DotOperandLayout`, `ttgl.amd.gfx1250.get_wmma_scale_layout`, `ttgl.full`, and 4 more. This scope touches pytest parametrization, debugging and inspection paths, layout transformation reasoning, random-data generation.
- **CN:** 定义测试函数 `test_amd_wmma_scale_layout_for_multicta`。 装饰器：`pytest.mark.parametrize('target', [HIP_TARGET_GFX1250])`。 参数：`target`。 该作用域中的嵌套定义：`kernel`。 关键调用包括 `pytest.mark.parametrize`、`run_parser`、`expecttest.assert_expected_inline`、`ttgl.DotOperandLayout`、`ttgl.amd.gfx1250.get_wmma_scale_layout`、`ttgl.full` 等另外 4 项。 该作用域涉及pytest 参数化、调试与检查路径、布局变换推理、随机数据生成。

#### Lines 3261-3262

```python
    @gluon.jit
    def kernel():
```
- **EN:** Defines the helper function `kernel`. Decorators: `gluon.jit`. Key calls include `ttgl.DotOperandLayout`, `ttgl.amd.gfx1250.get_wmma_scale_layout`, `ttgl.full`, `ttgl.amd.AMDWMMALayout`. This scope touches layout transformation reasoning, random-data generation.
- **CN:** 定义辅助函数 `kernel`。 装饰器：`gluon.jit`。 关键调用包括 `ttgl.DotOperandLayout`、`ttgl.amd.gfx1250.get_wmma_scale_layout`、`ttgl.full`、`ttgl.amd.AMDWMMALayout`。 该作用域涉及布局变换推理、随机数据生成。

##### Lines 3263-3278

```python
        a_layout: ttgl.constexpr = ttgl.DotOperandLayout(
            operand_index=0,  #
            parent=ttgl.amd.AMDWMMALayout(version=3, transposed=True, warp_bases=[[0, 1], [1, 0]],
                                          instr_shape=[16, 16, 64], cga_layout=[[0, 0], [1, 0]]),  #
            k_width=16)
        a_scale_layout: ttgl.constexpr = ttgl.amd.gfx1250.get_wmma_scale_layout(a_layout, [64, 4])
        ttgl.full([64, 4], 0x02, ttgl.uint8, a_scale_layout)

        b_layout: ttgl.constexpr = ttgl.DotOperandLayout(
            operand_index=1,  #
            parent=ttgl.amd.AMDWMMALayout(version=3, transposed=True, warp_bases=[[0, 1], [1, 0]],
                                          instr_shape=[16, 16, 64], cga_layout=[[1, 0], [0, 0]]),  #
            k_width=16,
        )
        b_scale_layout: ttgl.constexpr = ttgl.amd.gfx1250.get_wmma_scale_layout(b_layout, [64, 4])
        ttgl.full([64, 4], 0x01, ttgl.uint8, b_scale_layout)
```
- **EN:** Prepares or updates state through `a_layout`, `a_scale_layout`, `b_layout`, `b_scale_layout`. Invokes `ttgl.DotOperandLayout`, `ttgl.amd.AMDWMMALayout`, `ttgl.amd.gfx1250.get_wmma_scale_layout`, `ttgl.full` to execute the test logic. Relevant themes: layout transformation reasoning, random-data generation.
- **CN:** 通过 `a_layout`、`a_scale_layout`、`b_layout`、`b_scale_layout` 准备或更新状态。 调用 `ttgl.DotOperandLayout`、`ttgl.amd.AMDWMMALayout`、`ttgl.amd.gfx1250.get_wmma_scale_layout`、`ttgl.full` 执行测试逻辑。 相关主题：布局变换推理、随机数据生成。

#### Lines 3279-3294

```python

    module = run_parser(kernel, *make_args(num_warps=4, num_ctas=4), target=target)
    expecttest.assert_expected_inline(
        anonymize_ir(module.str_nodebug()), """\
#linear = #ttg.linear<{register = [[0, 1], [0, 2]], lane = [[1, 0], [2, 0], [4, 0], [8, 0], [0, 0]], warp = [[0, 0], [16, 0]], block = [[0, 0], [32, 0]]}>
#linear1 = #ttg.linear<{register = [[0, 1], [0, 2]], lane = [[1, 0], [2, 0], [4, 0], [8, 0], [0, 0]], warp = [[16, 0], [0, 0]], block = [[32, 0], [0, 0]]}>
module attributes {"ttg.num-ctas" = 4 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "...", "ttg.threads-per-warp" = 32 : i32} {
  tt.func public @kernel() attributes {noinline = false} {
    %c2_i8 = arith.constant 2 : i8
    %cst = arith.constant dense<2> : tensor<64x4xi8, #linear>
    %c1_i8 = arith.constant 1 : i8
    %cst_0 = arith.constant dense<1> : tensor<64x4xi8, #linear1>
    tt.return
  }
}
""")
```
- **EN:** Prepares or updates state through `module`. Invokes `run_parser`, `make_args`, `expecttest.assert_expected_inline`, `anonymize_ir`, `module.str_nodebug` to execute the test logic. Relevant themes: debugging and inspection paths.
- **CN:** 通过 `module` 准备或更新状态。 调用 `run_parser`、`make_args`、`expecttest.assert_expected_inline`、`anonymize_ir`、`module.str_nodebug` 执行测试逻辑。 相关主题：调试与检查路径。

### Lines 3295-3298

```python


@gluon.jit
def padded_shared_layout_kernel():
```
- **EN:** Defines the helper function `padded_shared_layout_kernel`. Decorators: `gluon.jit`. Key calls include `ttgl.PaddedSharedLayout.with_identity_for`, `ttgl.allocate_shared_memory`. This scope touches layout transformation reasoning.
- **CN:** 定义辅助函数 `padded_shared_layout_kernel`。 装饰器：`gluon.jit`。 关键调用包括 `ttgl.PaddedSharedLayout.with_identity_for`、`ttgl.allocate_shared_memory`。 该作用域涉及布局变换推理。

#### Lines 3299-3302

```python
    shape: ttgl.constexpr = [64, 64]
    padded_shared_layout: ttgl.constexpr = ttgl.PaddedSharedLayout.with_identity_for(
        interval_padding_pairs=[[2, 1], [4, 2], [8, 4]], shape=shape, order=[1, 0])
    ttgl.allocate_shared_memory(ttgl.int32, shape, padded_shared_layout)
```
- **EN:** Prepares or updates state through `shape`, `padded_shared_layout`. Invokes `ttgl.PaddedSharedLayout.with_identity_for`, `ttgl.allocate_shared_memory` to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `shape`、`padded_shared_layout` 准备或更新状态。 调用 `ttgl.PaddedSharedLayout.with_identity_for`、`ttgl.allocate_shared_memory` 执行测试逻辑。 相关主题：布局变换推理。

### Lines 3303-3307

```python


@pytest.mark.parametrize("target", [HIP_TARGET_CDNA3, HIP_TARGET_CDNA4])
def test_padded_shared_layout(target):
    # This test is used to test the construction of PaddedSharedEncodingAttr in the gluon.
```
- **EN:** Defines the test function `test_padded_shared_layout`. Decorators: `pytest.mark.parametrize('target', [HIP_TARGET_CDNA3, HIP_TARGET_CDNA4])`. Parameters: `target`. Key calls include `pytest.mark.parametrize`, `run_parser`, `expecttest.assert_expected_inline`, `anonymize_ir`, `module.str_nodebug`. This scope touches pytest parametrization, debugging and inspection paths, layout transformation reasoning.
- **CN:** 定义测试函数 `test_padded_shared_layout`。 装饰器：`pytest.mark.parametrize('target', [HIP_TARGET_CDNA3, HIP_TARGET_CDNA4])`。 参数：`target`。 关键调用包括 `pytest.mark.parametrize`、`run_parser`、`expecttest.assert_expected_inline`、`anonymize_ir`、`module.str_nodebug`。 该作用域涉及pytest 参数化、调试与检查路径、布局变换推理。

#### Lines 3308-3319

```python
    module = run_parser(padded_shared_layout_kernel, target=target)
    expecttest.assert_expected_inline(
        anonymize_ir(module.str_nodebug()), """\
#shared = #ttg.padded_shared<[2:+1, 4:+2, 8:+4] {order = [1, 0], shape = [64, 64]}>
#smem = #ttg.shared_memory
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "...", "ttg.threads-per-warp" = 64 : i32} {
  tt.func public @padded_shared_layout_kernel() attributes {noinline = false} {
    %0 = ttg.local_alloc : () -> !ttg.memdesc<64x64xi32, #shared, #smem, mutable>
    tt.return
  }
}
""")
```
- **EN:** Prepares or updates state through `module`. Invokes `run_parser`, `expecttest.assert_expected_inline`, `anonymize_ir`, `module.str_nodebug` to execute the test logic. Relevant themes: debugging and inspection paths, layout transformation reasoning.
- **CN:** 通过 `module` 准备或更新状态。 调用 `run_parser`、`expecttest.assert_expected_inline`、`anonymize_ir`、`module.str_nodebug` 执行测试逻辑。 相关主题：调试与检查路径、布局变换推理。

### Lines 3320-3323

```python


@gluon.jit
def infer_layout_for_padded_shared_kernel():
```
- **EN:** Defines the helper function `infer_layout_for_padded_shared_kernel`. Decorators: `gluon.jit`. Key calls include `ttgl.PaddedSharedLayout.with_identity_for`, `ttgl.allocate_shared_memory`, `smem.permute`, `ttgl.static_assert`. This scope touches layout transformation reasoning.
- **CN:** 定义辅助函数 `infer_layout_for_padded_shared_kernel`。 装饰器：`gluon.jit`。 关键调用包括 `ttgl.PaddedSharedLayout.with_identity_for`、`ttgl.allocate_shared_memory`、`smem.permute`、`ttgl.static_assert`。 该作用域涉及布局变换推理。

#### Lines 3324-3350

```python
    shape: ttgl.constexpr = [32, 4, 32]
    initial_order: ttgl.constexpr = [2, 0, 1]
    layout: ttgl.constexpr = ttgl.PaddedSharedLayout.with_identity_for(interval_padding_pairs=[[2, 1], [4, 2], [8, 4]],
                                                                       shape=shape, order=initial_order)
    smem = ttgl.allocate_shared_memory(ttgl.int32, shape, layout)

    reshaped = smem.permute((1, 0, 2))
    """
    permute is [1 0 2], which means
    old 1 to new 0
    old 0 to new 1
    old 2 to new 2
    so inverseMapping[0] = 1, inverseMapping[1] = 0, inverseMapping[2] = 2

    order in srcEnc is [2, 0, 1]
    thus the order in dstEnc are:
    newOrder[0] = inverseMapping[srcEncOrder[0]] = 2
    newOrder[1] = inverseMapping[srcEncOrder[1]] = 1
    newOrder[2] = inverseMapping[srcEncOrder[2]] = 0

    which results in the new shape of [4, 32, 32]
    """
    perm_shape: ttgl.constexpr = [4, 32, 32]
    perm_order: ttgl.constexpr = [2, 1, 0]
    ref_layout: ttgl.constexpr = ttgl.PaddedSharedLayout.with_identity_for(
        interval_padding_pairs=[[2, 1], [4, 2], [8, 4]], shape=perm_shape, order=perm_order)
    ttgl.static_assert(reshaped.type.layout == ref_layout)
```
- **EN:** Prepares or updates state through `shape`, `initial_order`, `layout`, `smem`, `reshaped`, `perm_shape`, `perm_order`, `ref_layout`. Invokes `ttgl.PaddedSharedLayout.with_identity_for`, `ttgl.allocate_shared_memory`, `smem.permute`, `ttgl.static_assert` to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `shape`、`initial_order`、`layout`、`smem`、`reshaped`、`perm_shape`、`perm_order`、`ref_layout` 准备或更新状态。 调用 `ttgl.PaddedSharedLayout.with_identity_for`、`ttgl.allocate_shared_memory`、`smem.permute`、`ttgl.static_assert` 执行测试逻辑。 相关主题：布局变换推理。

### Lines 3351-3356

```python


@pytest.mark.parametrize("target", ALL_TARGETS)
def test_infer_layout_for_padded_shared(target):
    # This test is used to test the conversion to gluon object PaddedSharedLayout from PaddedSharedEncodingAttr.
    # This conversion is in layoutToGluon and ttgl.permute will finally use it.
```
- **EN:** Defines the test function `test_infer_layout_for_padded_shared`. Decorators: `pytest.mark.parametrize('target', ALL_TARGETS)`. Parameters: `target`. Key calls include `pytest.mark.parametrize`, `run_parser`, `expecttest.assert_expected_inline`, `anonymize_ir`, `module.str_nodebug`. This scope touches pytest parametrization, debugging and inspection paths, layout transformation reasoning.
- **CN:** 定义测试函数 `test_infer_layout_for_padded_shared`。 装饰器：`pytest.mark.parametrize('target', ALL_TARGETS)`。 参数：`target`。 关键调用包括 `pytest.mark.parametrize`、`run_parser`、`expecttest.assert_expected_inline`、`anonymize_ir`、`module.str_nodebug`。 该作用域涉及pytest 参数化、调试与检查路径、布局变换推理。

#### Lines 3357-3370

```python
    module = run_parser(infer_layout_for_padded_shared_kernel, target=target)
    expecttest.assert_expected_inline(
        anonymize_ir(module.str_nodebug()), """\
#shared = #ttg.padded_shared<[2:+1, 4:+2, 8:+4] {order = [2, 0, 1], shape = [32, 4, 32]}>
#shared1 = #ttg.padded_shared<[2:+1, 4:+2, 8:+4] {order = [2, 1, 0], shape = [4, 32, 32]}>
#smem = #ttg.shared_memory
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "...", "ttg.threads-per-warp" = 32 : i32} {
  tt.func public @infer_layout_for_padded_shared_kernel() attributes {noinline = false} {
    %0 = ttg.local_alloc : () -> !ttg.memdesc<32x4x32xi32, #shared, #smem, mutable>
    %1 = ttg.memdesc_trans %0 {order = array<i32: 1, 0, 2>} : !ttg.memdesc<32x4x32xi32, #shared, #smem, mutable> -> !ttg.memdesc<4x32x32xi32, #shared1, #smem, mutable>
    tt.return
  }
}
""")
```
- **EN:** Prepares or updates state through `module`. Invokes `run_parser`, `expecttest.assert_expected_inline`, `anonymize_ir`, `module.str_nodebug` to execute the test logic. Relevant themes: debugging and inspection paths, layout transformation reasoning.
- **CN:** 通过 `module` 准备或更新状态。 调用 `run_parser`、`expecttest.assert_expected_inline`、`anonymize_ir`、`module.str_nodebug` 执行测试逻辑。 相关主题：调试与检查路径、布局变换推理。

### Lines 3371-3377

```python


@filecheck_test
@gluon.jit
def test_layout_zeros():
    # CHECK: #blocked = #ttg.blocked
    # CHECK: arith.constant dense<0.000000e+00> : tensor<128xf32, #blocked>
```
- **EN:** Defines the test function `test_layout_zeros`. Decorators: `filecheck_test`, `gluon.jit`. Key calls include `ttgl.zeros`, `ttgl.BlockedLayout`. This scope touches layout transformation reasoning.
- **CN:** 定义测试函数 `test_layout_zeros`。 装饰器：`filecheck_test`、`gluon.jit`。 关键调用包括 `ttgl.zeros`、`ttgl.BlockedLayout`。 该作用域涉及布局变换推理。

#### Lines 3378-3378

```python
    ttgl.zeros([128], ttgl.float32, layout=ttgl.BlockedLayout([1], [32], [4], [0]))
```
- **EN:** Invokes `ttgl.zeros`, `ttgl.BlockedLayout` to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 调用 `ttgl.zeros`、`ttgl.BlockedLayout` 执行测试逻辑。 相关主题：布局变换推理。

### Lines 3379-3384

```python


@pytest.mark.parametrize("target", [HIP_TARGET_CDNA3, HIP_TARGET_CDNA4])
def test_buffer_atomic_rmw(target):

    @gluon.jit
```
- **EN:** Defines the test function `test_buffer_atomic_rmw`. Decorators: `pytest.mark.parametrize('target', [HIP_TARGET_CDNA3, HIP_TARGET_CDNA4])`. Parameters: `target`. Nested definitions in this scope: `kernel`. Key calls include `pytest.mark.parametrize`, `MockTensor`, `run_parser`, `expecttest.assert_expected_inline`, `ttgl.arange`, `ttgl.full`, and 13 more. This scope touches pytest parametrization, debugging and inspection paths, layout transformation reasoning.
- **CN:** 定义测试函数 `test_buffer_atomic_rmw`。 装饰器：`pytest.mark.parametrize('target', [HIP_TARGET_CDNA3, HIP_TARGET_CDNA4])`。 参数：`target`。 该作用域中的嵌套定义：`kernel`。 关键调用包括 `pytest.mark.parametrize`、`MockTensor`、`run_parser`、`expecttest.assert_expected_inline`、`ttgl.arange`、`ttgl.full` 等另外 13 项。 该作用域涉及pytest 参数化、调试与检查路径、布局变换推理。

#### Lines 3384-3385

```python
    @gluon.jit
    def kernel(int32_ptr, uint32_ptr, int64_ptr, fp16_ptr, fp32_ptr):
```
- **EN:** Defines the helper function `kernel`. Decorators: `gluon.jit`. Parameters: `int32_ptr`, `uint32_ptr`, `int64_ptr`, `fp16_ptr`, `fp32_ptr`. Key calls include `ttgl.arange`, `ttgl.full`, `ttgl.amd.cdna3.buffer_atomic_max`, `ttgl.amd.cdna3.buffer_atomic_min`, `ttgl.amd.cdna3.buffer_atomic_and`, `ttgl.amd.cdna3.buffer_atomic_or`, and 6 more. This scope touches layout transformation reasoning.
- **CN:** 定义辅助函数 `kernel`。 装饰器：`gluon.jit`。 参数：`int32_ptr`、`uint32_ptr`、`int64_ptr`、`fp16_ptr`、`fp32_ptr`。 关键调用包括 `ttgl.arange`、`ttgl.full`、`ttgl.amd.cdna3.buffer_atomic_max`、`ttgl.amd.cdna3.buffer_atomic_min`、`ttgl.amd.cdna3.buffer_atomic_and`、`ttgl.amd.cdna3.buffer_atomic_or` 等另外 6 项。 该作用域涉及布局变换推理。

##### Lines 3386-3416

```python
        BLOCK: ttgl.constexpr = 1
        offsets = ttgl.arange(0, BLOCK, layout=ttgl.AutoLayout())

        val = ttgl.full([BLOCK], 1, ttgl.int32, layout=ttgl.AutoLayout())
        ttgl.amd.cdna3.buffer_atomic_max(int32_ptr, offsets, val)
        ttgl.amd.cdna3.buffer_atomic_min(int32_ptr, offsets, val)
        ttgl.amd.cdna3.buffer_atomic_and(int32_ptr, offsets, val)
        ttgl.amd.cdna3.buffer_atomic_or(int32_ptr, offsets, val)
        #value broadcast
        ttgl.amd.cdna3.buffer_atomic_xor(int32_ptr, offsets, value=1)

        # operands should be unsigned
        val = ttgl.full([BLOCK], 1, ttgl.uint32, layout=ttgl.AutoLayout())
        ttgl.amd.cdna3.buffer_atomic_max(uint32_ptr, offsets, val)
        ttgl.amd.cdna3.buffer_atomic_min(uint32_ptr, offsets, val)
        ttgl.amd.cdna3.buffer_atomic_add(uint32_ptr, offsets, val)

        val = val.cast(ttgl.int64)
        #mask broadcast
        ttgl.amd.cdna3.buffer_atomic_xchg(int64_ptr, offsets, val, mask=0)

        mask = ttgl.full([BLOCK], True, ttgl.int32, layout=ttgl.AutoLayout())
        val = ttgl.zeros([BLOCK], ttgl.float16, layout=ttgl.AutoLayout())
        ttgl.amd.cdna3.buffer_atomic_add(fp16_ptr, offsets, val, mask=mask)
        ttgl.amd.cdna3.buffer_atomic_add(fp16_ptr, offsets, val, mask=mask, scope="sys")
        ttgl.amd.cdna3.buffer_atomic_add(fp16_ptr, offsets, val, mask=mask, scope="cta", sem="relaxed")

        val = val.cast(ttgl.float32)
        ttgl.amd.cdna3.buffer_atomic_add(fp32_ptr, offsets, val, mask=mask)
        ttgl.amd.cdna3.buffer_atomic_add(fp32_ptr, offsets, val, mask=mask, scope="sys")
        ttgl.amd.cdna3.buffer_atomic_add(fp32_ptr, offsets, val, mask=mask, scope="cta", sem="relaxed")
```
- **EN:** Prepares or updates state through `BLOCK`, `offsets`, `val`, `mask`. Invokes `ttgl.arange`, `ttgl.AutoLayout`, `ttgl.full`, `ttgl.amd.cdna3.buffer_atomic_max`, `ttgl.amd.cdna3.buffer_atomic_min`, `ttgl.amd.cdna3.buffer_atomic_and`, and 6 more to execute the test logic. Relevant themes: layout transformation reasoning, random-data generation.
- **CN:** 通过 `BLOCK`、`offsets`、`val`、`mask` 准备或更新状态。 调用 `ttgl.arange`、`ttgl.AutoLayout`、`ttgl.full`、`ttgl.amd.cdna3.buffer_atomic_max`、`ttgl.amd.cdna3.buffer_atomic_min`、`ttgl.amd.cdna3.buffer_atomic_and` 等另外 6 项 执行测试逻辑。 相关主题：布局变换推理、随机数据生成。

#### Lines 3417-3488

```python

    fp16_ptr = MockTensor(ttgl.float16)
    fp32_ptr = MockTensor(ttgl.float32)
    int_ptr = MockTensor(ttgl.int32)
    uint_ptr = MockTensor(ttgl.uint32)
    int64_ptr = MockTensor(ttgl.int64)
    module = run_parser(kernel, *make_args(int_ptr, uint_ptr, int64_ptr, fp16_ptr, fp32_ptr), target=target)
    expecttest.assert_expected_inline(
        anonymize_ir(module.str_nodebug()), """\
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "...", "ttg.threads-per-warp" = 64 : i32} {
  tt.func public @kernel(%arg0: !tt.ptr<i32> {tt.divisibility = 16 : i32}, %arg1: !tt.ptr<i32> {tt.divisibility = 16 : i32}, %arg2: !tt.ptr<i64> {tt.divisibility = 16 : i32}, %arg3: !tt.ptr<f16> {tt.divisibility = 16 : i32}, %arg4: !tt.ptr<f32> {tt.divisibility = 16 : i32}) attributes {noinline = false} {
    %0 = tt.make_range {end = 1 : i32, start = 0 : i32} : tensor<1xi32, #gluon.auto_encoding>
    %c1_i32 = arith.constant 1 : i32
    %cst = arith.constant dense<1> : tensor<1xi32, #gluon.auto_encoding>
    %1 = amdg.buffer_atomic_rmw max, acq_rel, gpu, %cst, %arg0[%0] : tensor<1xi32, #gluon.auto_encoding>
    %2 = amdg.buffer_atomic_rmw min, acq_rel, gpu, %cst, %arg0[%0] : tensor<1xi32, #gluon.auto_encoding>
    %3 = amdg.buffer_atomic_rmw and, acq_rel, gpu, %cst, %arg0[%0] : tensor<1xi32, #gluon.auto_encoding>
    %4 = amdg.buffer_atomic_rmw or, acq_rel, gpu, %cst, %arg0[%0] : tensor<1xi32, #gluon.auto_encoding>
    %c1_i32_0 = arith.constant 1 : i32
    %cst_1 = arith.constant dense<1> : tensor<1xi32, #gluon.auto_encoding>
    %5 = amdg.buffer_atomic_rmw xor, acq_rel, gpu, %cst_1, %arg0[%0] : tensor<1xi32, #gluon.auto_encoding>
    %c1_i32_2 = arith.constant 1 : i32
    %cst_3 = arith.constant dense<1> : tensor<1xi32, #gluon.auto_encoding>
    %6 = amdg.buffer_atomic_rmw umax, acq_rel, gpu, %cst_3, %arg1[%0] : tensor<1xi32, #gluon.auto_encoding>
    %7 = amdg.buffer_atomic_rmw umin, acq_rel, gpu, %cst_3, %arg1[%0] : tensor<1xi32, #gluon.auto_encoding>
    %8 = amdg.buffer_atomic_rmw add, acq_rel, gpu, %cst_3, %arg1[%0] : tensor<1xi32, #gluon.auto_encoding>
    %9 = arith.extui %cst_3 : tensor<1xi32, #gluon.auto_encoding> to tensor<1xi64, #gluon.auto_encoding>
    %c0_i32 = arith.constant 0 : i32
    %c0_i32_4 = arith.constant 0 : i32
    %10 = arith.cmpi ne, %c0_i32, %c0_i32_4 : i32
    %11 = tt.splat %10 : i1 -> tensor<1xi1, #gluon.auto_encoding>
    %12 = amdg.buffer_atomic_rmw exch, acq_rel, gpu, %9, %arg2[%0], %11 : tensor<1xi64, #gluon.auto_encoding>
    %c1_i32_5 = arith.constant 1 : i32
    %cst_6 = arith.constant dense<1> : tensor<1xi32, #gluon.auto_encoding>
    %13 = tt.call @triton.experimental.gluon.language._standard.zeros__Tc1T_cfp16_cAL() : () -> tensor<1xf16, #gluon.auto_encoding>
    %c0_i32_7 = arith.constant 0 : i32
    %cst_8 = arith.constant dense<0> : tensor<1xi32, #gluon.auto_encoding>
    %14 = arith.cmpi ne, %cst_6, %cst_8 : tensor<1xi32, #gluon.auto_encoding>
    %15 = amdg.buffer_atomic_rmw fadd, acq_rel, gpu, %13, %arg3[%0], %14 : tensor<1xf16, #gluon.auto_encoding>
    %c0_i32_9 = arith.constant 0 : i32
    %cst_10 = arith.constant dense<0> : tensor<1xi32, #gluon.auto_encoding>
    %16 = arith.cmpi ne, %cst_6, %cst_10 : tensor<1xi32, #gluon.auto_encoding>
    %17 = amdg.buffer_atomic_rmw fadd, acq_rel, sys, %13, %arg3[%0], %16 : tensor<1xf16, #gluon.auto_encoding>
    %c0_i32_11 = arith.constant 0 : i32
    %cst_12 = arith.constant dense<0> : tensor<1xi32, #gluon.auto_encoding>
    %18 = arith.cmpi ne, %cst_6, %cst_12 : tensor<1xi32, #gluon.auto_encoding>
    %19 = amdg.buffer_atomic_rmw fadd, relaxed, cta, %13, %arg3[%0], %18 : tensor<1xf16, #gluon.auto_encoding>
    %20 = arith.extf %13 : tensor<1xf16, #gluon.auto_encoding> to tensor<1xf32, #gluon.auto_encoding>
    %c0_i32_13 = arith.constant 0 : i32
    %cst_14 = arith.constant dense<0> : tensor<1xi32, #gluon.auto_encoding>
    %21 = arith.cmpi ne, %cst_6, %cst_14 : tensor<1xi32, #gluon.auto_encoding>
    %22 = amdg.buffer_atomic_rmw fadd, acq_rel, gpu, %20, %arg4[%0], %21 : tensor<1xf32, #gluon.auto_encoding>
    %c0_i32_15 = arith.constant 0 : i32
    %cst_16 = arith.constant dense<0> : tensor<1xi32, #gluon.auto_encoding>
    %23 = arith.cmpi ne, %cst_6, %cst_16 : tensor<1xi32, #gluon.auto_encoding>
    %24 = amdg.buffer_atomic_rmw fadd, acq_rel, sys, %20, %arg4[%0], %23 : tensor<1xf32, #gluon.auto_encoding>
    %c0_i32_17 = arith.constant 0 : i32
    %cst_18 = arith.constant dense<0> : tensor<1xi32, #gluon.auto_encoding>
    %25 = arith.cmpi ne, %cst_6, %cst_18 : tensor<1xi32, #gluon.auto_encoding>
    %26 = amdg.buffer_atomic_rmw fadd, relaxed, cta, %20, %arg4[%0], %25 : tensor<1xf32, #gluon.auto_encoding>
    tt.return
  }
  tt.func private @triton.experimental.gluon.language._standard.zeros__Tc1T_cfp16_cAL() -> tensor<1xf16, #gluon.auto_encoding> attributes {noinline = false} {
    %cst = arith.constant 0.000000e+00 : f16
    %cst_0 = arith.constant dense<0.000000e+00> : tensor<1xf16, #gluon.auto_encoding>
    tt.return %cst_0 : tensor<1xf16, #gluon.auto_encoding>
  ^bb1:  // no predecessors
    %0 = ub.poison : tensor<1xf16, #gluon.auto_encoding>
    tt.return %0 : tensor<1xf16, #gluon.auto_encoding>
  }
}
""")
```
- **EN:** Prepares or updates state through `fp16_ptr`, `fp32_ptr`, `int_ptr`, `uint_ptr`, `int64_ptr`, `module`. Invokes `MockTensor`, `run_parser`, `make_args`, `expecttest.assert_expected_inline`, `anonymize_ir`, `module.str_nodebug` to execute the test logic. Relevant themes: debugging and inspection paths.
- **CN:** 通过 `fp16_ptr`、`fp32_ptr`、`int_ptr`、`uint_ptr`、`int64_ptr`、`module` 准备或更新状态。 调用 `MockTensor`、`run_parser`、`make_args`、`expecttest.assert_expected_inline`、`anonymize_ir`、`module.str_nodebug` 执行测试逻辑。 相关主题：调试与检查路径。

### Lines 3489-3494

```python


@pytest.mark.parametrize("target", [HIP_TARGET_CDNA4])
def test_buffer_atomic_rmw_bf16(target):

    @gluon.jit
```
- **EN:** Defines the test function `test_buffer_atomic_rmw_bf16`. Decorators: `pytest.mark.parametrize('target', [HIP_TARGET_CDNA4])`. Parameters: `target`. Nested definitions in this scope: `kernel`. Key calls include `pytest.mark.parametrize`, `MockTensor`, `run_parser`, `expecttest.assert_expected_inline`, `ttgl.arange`, `ttgl.zeros`, and 6 more. This scope touches pytest parametrization, debugging and inspection paths, layout transformation reasoning.
- **CN:** 定义测试函数 `test_buffer_atomic_rmw_bf16`。 装饰器：`pytest.mark.parametrize('target', [HIP_TARGET_CDNA4])`。 参数：`target`。 该作用域中的嵌套定义：`kernel`。 关键调用包括 `pytest.mark.parametrize`、`MockTensor`、`run_parser`、`expecttest.assert_expected_inline`、`ttgl.arange`、`ttgl.zeros` 等另外 6 项。 该作用域涉及pytest 参数化、调试与检查路径、布局变换推理。

#### Lines 3494-3495

```python
    @gluon.jit
    def kernel(bf16_ptr):
```
- **EN:** Defines the helper function `kernel`. Decorators: `gluon.jit`. Parameters: `bf16_ptr`. Key calls include `ttgl.arange`, `ttgl.zeros`, `ttgl.amd.cdna4.buffer_atomic_add`, `ttgl.full`, `ttgl.AutoLayout`. This scope touches layout transformation reasoning.
- **CN:** 定义辅助函数 `kernel`。 装饰器：`gluon.jit`。 参数：`bf16_ptr`。 关键调用包括 `ttgl.arange`、`ttgl.zeros`、`ttgl.amd.cdna4.buffer_atomic_add`、`ttgl.full`、`ttgl.AutoLayout`。 该作用域涉及布局变换推理。

##### Lines 3496-3501

```python
        offsets = ttgl.arange(0, 1, layout=ttgl.AutoLayout())
        val = ttgl.zeros([1], ttgl.bfloat16, layout=ttgl.AutoLayout())
        ttgl.amd.cdna4.buffer_atomic_add(bf16_ptr, offsets, val, mask=0)
        mask = ttgl.full([1], True, ttgl.int32, layout=ttgl.AutoLayout())
        ttgl.amd.cdna4.buffer_atomic_add(bf16_ptr, offsets, val, mask=mask, scope="sys")
        ttgl.amd.cdna4.buffer_atomic_add(bf16_ptr, offsets, val, mask=mask, scope="cta", sem="relaxed")
```
- **EN:** Prepares or updates state through `offsets`, `val`, `mask`. Invokes `ttgl.arange`, `ttgl.AutoLayout`, `ttgl.zeros`, `ttgl.amd.cdna4.buffer_atomic_add`, `ttgl.full` to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `offsets`、`val`、`mask` 准备或更新状态。 调用 `ttgl.arange`、`ttgl.AutoLayout`、`ttgl.zeros`、`ttgl.amd.cdna4.buffer_atomic_add`、`ttgl.full` 执行测试逻辑。 相关主题：布局变换推理。

#### Lines 3502-3537

```python

    bf16_ptr = MockTensor(ttgl.bfloat16)
    module = run_parser(kernel, *make_args(bf16_ptr), target=target)
    expecttest.assert_expected_inline(
        anonymize_ir(module.str_nodebug()), """\
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "...", "ttg.threads-per-warp" = 64 : i32} {
  tt.func public @kernel(%arg0: !tt.ptr<bf16> {tt.divisibility = 16 : i32}) attributes {noinline = false} {
    %0 = tt.make_range {end = 1 : i32, start = 0 : i32} : tensor<1xi32, #gluon.auto_encoding>
    %1 = tt.call @triton.experimental.gluon.language._standard.zeros__Tc1T_cbf16_cAL() : () -> tensor<1xbf16, #gluon.auto_encoding>
    %c0_i32 = arith.constant 0 : i32
    %c0_i32_0 = arith.constant 0 : i32
    %2 = arith.cmpi ne, %c0_i32, %c0_i32_0 : i32
    %3 = tt.splat %2 : i1 -> tensor<1xi1, #gluon.auto_encoding>
    %4 = amdg.buffer_atomic_rmw fadd, acq_rel, gpu, %1, %arg0[%0], %3 : tensor<1xbf16, #gluon.auto_encoding>
    %c1_i32 = arith.constant 1 : i32
    %cst = arith.constant dense<1> : tensor<1xi32, #gluon.auto_encoding>
    %c0_i32_1 = arith.constant 0 : i32
    %cst_2 = arith.constant dense<0> : tensor<1xi32, #gluon.auto_encoding>
    %5 = arith.cmpi ne, %cst, %cst_2 : tensor<1xi32, #gluon.auto_encoding>
    %6 = amdg.buffer_atomic_rmw fadd, acq_rel, sys, %1, %arg0[%0], %5 : tensor<1xbf16, #gluon.auto_encoding>
    %c0_i32_3 = arith.constant 0 : i32
    %cst_4 = arith.constant dense<0> : tensor<1xi32, #gluon.auto_encoding>
    %7 = arith.cmpi ne, %cst, %cst_4 : tensor<1xi32, #gluon.auto_encoding>
    %8 = amdg.buffer_atomic_rmw fadd, relaxed, cta, %1, %arg0[%0], %7 : tensor<1xbf16, #gluon.auto_encoding>
    tt.return
  }
  tt.func private @triton.experimental.gluon.language._standard.zeros__Tc1T_cbf16_cAL() -> tensor<1xbf16, #gluon.auto_encoding> attributes {noinline = false} {
    %cst = arith.constant 0.000000e+00 : bf16
    %cst_0 = arith.constant dense<0.000000e+00> : tensor<1xbf16, #gluon.auto_encoding>
    tt.return %cst_0 : tensor<1xbf16, #gluon.auto_encoding>
  ^bb1:  // no predecessors
    %0 = ub.poison : tensor<1xbf16, #gluon.auto_encoding>
    tt.return %0 : tensor<1xbf16, #gluon.auto_encoding>
  }
}
""")
```
- **EN:** Prepares or updates state through `bf16_ptr`, `module`. Invokes `MockTensor`, `run_parser`, `make_args`, `expecttest.assert_expected_inline`, `anonymize_ir`, `module.str_nodebug` to execute the test logic. Relevant themes: debugging and inspection paths.
- **CN:** 通过 `bf16_ptr`、`module` 准备或更新状态。 调用 `MockTensor`、`run_parser`、`make_args`、`expecttest.assert_expected_inline`、`anonymize_ir`、`module.str_nodebug` 执行测试逻辑。 相关主题：调试与检查路径。

### Lines 3538-3543

```python


@pytest.mark.parametrize("target", [HIP_TARGET_CDNA3, HIP_TARGET_CDNA4, HIP_TARGET_GFX1250])
def test_amd_warp_pipeline(target):

    @gluon.jit
```
- **EN:** Defines the test function `test_amd_warp_pipeline`. Decorators: `pytest.mark.parametrize('target', [HIP_TARGET_CDNA3, HIP_TARGET_CDNA4, HIP_TARGET_GFX1250])`. Parameters: `target`. Nested definitions in this scope: `kernel`. Key calls include `pytest.mark.parametrize`, `run_parser`, `anonymize_ir`, `re.sub`, `expecttest.assert_expected_inline`, `module.str_nodebug`, and 2 more. This scope touches pytest parametrization, plugin or compiler extension points, debugging and inspection paths.
- **CN:** 定义测试函数 `test_amd_warp_pipeline`。 装饰器：`pytest.mark.parametrize('target', [HIP_TARGET_CDNA3, HIP_TARGET_CDNA4, HIP_TARGET_GFX1250])`。 参数：`target`。 该作用域中的嵌套定义：`kernel`。 关键调用包括 `pytest.mark.parametrize`、`run_parser`、`anonymize_ir`、`re.sub`、`expecttest.assert_expected_inline`、`module.str_nodebug` 等另外 2 项。 该作用域涉及pytest 参数化、插件或编译器扩展点、调试与检查路径。

#### Lines 3543-3544

```python
    @gluon.jit
    def kernel():
```
- **EN:** Defines the helper function `kernel`. Decorators: `gluon.jit`. Key calls include `ttgl.amd.warp_pipeline_stage`. This scope touches plugin or compiler extension points.
- **CN:** 定义辅助函数 `kernel`。 装饰器：`gluon.jit`。 关键调用包括 `ttgl.amd.warp_pipeline_stage`。 该作用域涉及插件或编译器扩展点。

##### Lines 3545-3546

```python
        c0: ttgl.constexpr = 0
        one: ttgl.constexpr = 1
```
- **EN:** Prepares or updates state through `c0`, `one`.
- **CN:** 通过 `c0`、`one` 准备或更新状态。

##### Lines 3547-3554

```python

        # Simple loop with an explicit split point
        for i in range(c0, 10, one):
            with ttgl.amd.warp_pipeline_stage("stage0"):
                x = i + one
            with ttgl.amd.warp_pipeline_stage("stage1"):
                y = x * one
                x = y + one
```
- **EN:** Invokes `ttgl.amd.warp_pipeline_stage` to execute the test logic. Iterates across cases or data tiles. Relevant themes: plugin or compiler extension points.
- **CN:** 调用 `ttgl.amd.warp_pipeline_stage` 执行测试逻辑。 通过循环覆盖多个用例或数据分块。 相关主题：插件或编译器扩展点。

#### Lines 3555-3586

```python

    module = run_parser(kernel, *make_args(num_warps=8), target=target)
    ir_str = anonymize_ir(module.str_nodebug())
    ir_str = re.sub(r'("ttg\.threads-per-warp"\s*=\s*)\d{2}', r'\1...', ir_str)
    expecttest.assert_expected_inline(
        ir_str, """\
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 8 : i32, ttg.target = "...", "ttg.threads-per-warp" = ... : i32} {
  tt.func public @kernel() attributes {noinline = false} {
    %c0_i32 = arith.constant 0 : i32
    %c10_i32 = arith.constant 10 : i32
    %c1_i32 = arith.constant 1 : i32
    %0 = arith.bitcast %c0_i32 : i32 to i32
    %1 = arith.bitcast %c10_i32 : i32 to i32
    %2 = arith.bitcast %c1_i32 : i32 to i32
    %3 = ub.poison : i32
    scf.for %arg0 = %0 to %1 step %2  : i32 {
      %c1_i32_0 = arith.constant 1 : i32
      %c1_i32_1 = arith.constant 1 : i32
      %4 = arith.addi %arg0, %c1_i32_1 : i32
      rocdl.sched.barrier 0 {triton.warp_pipeline.border = "stage0"}
      %c1_i32_2 = arith.constant 1 : i32
      %c1_i32_3 = arith.constant 1 : i32
      %5 = arith.muli %4, %c1_i32_3 : i32
      %c1_i32_4 = arith.constant 1 : i32
      %c1_i32_5 = arith.constant 1 : i32
      %6 = arith.addi %5, %c1_i32_5 : i32
      rocdl.sched.barrier 0 {triton.warp_pipeline.border = "stage1"}
    }
    tt.return
  }
}
""")
```
- **EN:** Prepares or updates state through `module`, `ir_str`. Invokes `run_parser`, `make_args`, `anonymize_ir`, `module.str_nodebug`, `re.sub`, `expecttest.assert_expected_inline` to execute the test logic. Relevant themes: plugin or compiler extension points, debugging and inspection paths.
- **CN:** 通过 `module`、`ir_str` 准备或更新状态。 调用 `run_parser`、`make_args`、`anonymize_ir`、`module.str_nodebug`、`re.sub`、`expecttest.assert_expected_inline` 执行测试逻辑。 相关主题：插件或编译器扩展点、调试与检查路径。

### Lines 3587-3590

```python


@gluon.jit
def print_num_warps():
```
- **EN:** Defines the helper function `print_num_warps`. Decorators: `gluon.jit`. Key calls include `ttgl.num_warps`. This scope touches debugging and inspection paths.
- **CN:** 定义辅助函数 `print_num_warps`。 装饰器：`gluon.jit`。 关键调用包括 `ttgl.num_warps`。 该作用域涉及调试与检查路径。

#### Lines 3591-3592

```python
    num_warps: ttgl.constexpr = ttgl.num_warps()
    print("num_warps", num_warps)
```
- **EN:** Prepares or updates state through `num_warps`. Invokes `ttgl.num_warps` to execute the test logic. Relevant themes: debugging and inspection paths.
- **CN:** 通过 `num_warps` 准备或更新状态。 调用 `ttgl.num_warps` 执行测试逻辑。 相关主题：调试与检查路径。

### Lines 3593-3596

```python


@gluon.jit
def print_num_ctas():
```
- **EN:** Defines the helper function `print_num_ctas`. Decorators: `gluon.jit`. Key calls include `ttgl.num_ctas`. This scope touches debugging and inspection paths.
- **CN:** 定义辅助函数 `print_num_ctas`。 装饰器：`gluon.jit`。 关键调用包括 `ttgl.num_ctas`。 该作用域涉及调试与检查路径。

#### Lines 3597-3598

```python
    num_ctas: ttgl.constexpr = ttgl.num_ctas()
    print("num_ctas", num_ctas)
```
- **EN:** Prepares or updates state through `num_ctas`. Invokes `ttgl.num_ctas` to execute the test logic. Relevant themes: debugging and inspection paths.
- **CN:** 通过 `num_ctas` 准备或更新状态。 调用 `ttgl.num_ctas` 执行测试逻辑。 相关主题：调试与检查路径。

### Lines 3599-3615

```python


@filecheck_test
@gluon.jit
def test_get_num_warps():
    # CHECK-LABEL: test_get_num_warps
    # CHECK: tt.func private @{{.*}}print_num_warps
    # CHECK-NEXT arith.constant 4 : i32

    # CHECK: tt.func private @{{.*}}print_num_warps{{.*}}NW1
    # CHECK-NEXT arith.constant 1 : i32

    # CHECK: tt.func private @{{.*}}print_num_warps{{.*}}NW2
    # CHECK-NEXT arith.constant 2 : i32

    # CHECK: tt.func private @{{.*}}print_num_warps{{.*}}NW8
    # CHECK-NEXT arith.constant 8 : i32
```
- **EN:** Defines the test function `test_get_num_warps`. Decorators: `filecheck_test`, `gluon.jit`. Key calls include `print_num_warps`, `ttgl.warp_specialize`.
- **CN:** 定义测试函数 `test_get_num_warps`。 装饰器：`filecheck_test`、`gluon.jit`。 关键调用包括 `print_num_warps`、`ttgl.warp_specialize`。

#### Lines 3616-3622

```python
    print_num_warps()
    ttgl.warp_specialize([
        (print_num_warps, ()),
        (print_num_warps, ()),
        (print_num_warps, ()),
        (print_num_warps, ()),
    ], [1, 2, 8], [24, 24, 24])
```
- **EN:** Invokes `print_num_warps`, `ttgl.warp_specialize` to execute the test logic.
- **CN:** 调用 `print_num_warps`、`ttgl.warp_specialize` 执行测试逻辑。

### Lines 3623-3630

```python


@filecheck_test
@gluon.jit
def test_num_ctas():
    # CHECK-LABEL: test_num_ctas
    # CHECK: tt.func private @{{.*}}print_num_ctas
    # CHECK-NEXT: arith.constant 1 : i32
```
- **EN:** Defines the test function `test_num_ctas`. Decorators: `filecheck_test`, `gluon.jit`. Key calls include `print_num_ctas`.
- **CN:** 定义测试函数 `test_num_ctas`。 装饰器：`filecheck_test`、`gluon.jit`。 关键调用包括 `print_num_ctas`。

#### Lines 3631-3631

```python
    print_num_ctas()
```
- **EN:** Invokes `print_num_ctas` to execute the test logic.
- **CN:** 调用 `print_num_ctas` 执行测试逻辑。

### Lines 3632-3636

```python


def test_mismatch_shape_and_layout_rank():

    @gluon.jit
```
- **EN:** Defines the test function `test_mismatch_shape_and_layout_rank`. Nested definitions in this scope: `kernel`. Key calls include `ttgl.BlockedLayout`, `ttgl.full`, `pytest.raises`, `run_parser`. This scope touches layout transformation reasoning.
- **CN:** 定义测试函数 `test_mismatch_shape_and_layout_rank`。 该作用域中的嵌套定义：`kernel`。 关键调用包括 `ttgl.BlockedLayout`、`ttgl.full`、`pytest.raises`、`run_parser`。 该作用域涉及布局变换推理。

#### Lines 3636-3637

```python
    @gluon.jit
    def kernel():
```
- **EN:** Defines the helper function `kernel`. Decorators: `gluon.jit`. Key calls include `ttgl.BlockedLayout`, `ttgl.full`. This scope touches layout transformation reasoning.
- **CN:** 定义辅助函数 `kernel`。 装饰器：`gluon.jit`。 关键调用包括 `ttgl.BlockedLayout`、`ttgl.full`。 该作用域涉及布局变换推理。

##### Lines 3638-3639

```python
        layout: ttgl.constexpr = ttgl.BlockedLayout([1, 1], [1, 32], [1, 4], [1, 0])
        _ = ttgl.full([1, 16, 16, 1, 16], 0, ttgl.float16, layout=layout)
```
- **EN:** Prepares or updates state through `layout`, `_`. Invokes `ttgl.BlockedLayout`, `ttgl.full` to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `layout`、`_` 准备或更新状态。 调用 `ttgl.BlockedLayout`、`ttgl.full` 执行测试逻辑。 相关主题：布局变换推理。

#### Lines 3640-3642

```python

    with pytest.raises(CompilationError) as e:
        run_parser(kernel)
```
- **EN:** Invokes `pytest.raises`, `run_parser` to execute the test logic. Uses `pytest.raises` to confirm expected failure paths. Uses context managers to control resources or expectations.
- **CN:** 调用 `pytest.raises`、`run_parser` 执行测试逻辑。 使用 `pytest.raises` 确认预期的失败路径。 使用上下文管理器控制资源或预期行为。

#### Lines 3643-3644

```python

    assert "tensor shape and layout rank mismatch" in str(e.value.__cause__)
```
- **EN:** Validates behavior with 1 assertion(s). Relevant themes: layout transformation reasoning.
- **CN:** 通过 1 个断言验证行为。 相关主题：布局变换推理。

### Lines 3645-3649

```python


def test_non_scalar_loop_bounds():

    @gluon.jit
```
- **EN:** Defines the test function `test_non_scalar_loop_bounds`. Nested definitions in this scope: `kernel`, `kernel`, `kernel`. Key calls include `ttgl.full`, `pytest.raises`, `run_parser`, `ttgl.BlockedLayout`. This scope touches layout transformation reasoning.
- **CN:** 定义测试函数 `test_non_scalar_loop_bounds`。 该作用域中的嵌套定义：`kernel`、`kernel`、`kernel`。 关键调用包括 `ttgl.full`、`pytest.raises`、`run_parser`、`ttgl.BlockedLayout`。 该作用域涉及布局变换推理。

#### Lines 3649-3650

```python
    @gluon.jit
    def kernel():
```
- **EN:** Defines the helper function `kernel`. Decorators: `gluon.jit`. Key calls include `ttgl.full`, `ttgl.BlockedLayout`. This scope touches layout transformation reasoning.
- **CN:** 定义辅助函数 `kernel`。 装饰器：`gluon.jit`。 关键调用包括 `ttgl.full`、`ttgl.BlockedLayout`。 该作用域涉及布局变换推理。

##### Lines 3651-3651

```python
        x = ttgl.full([32], 0, ttgl.int32, layout=ttgl.BlockedLayout([1], [32], [1], [0]))
```
- **EN:** Prepares or updates state through `x`. Invokes `ttgl.full`, `ttgl.BlockedLayout` to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `x` 准备或更新状态。 调用 `ttgl.full`、`ttgl.BlockedLayout` 执行测试逻辑。 相关主题：布局变换推理。

##### Lines 3652-3653

```python
        for _ in range(x, 10, 1):
            pass
```
- **EN:** Iterates across cases or data tiles.
- **CN:** 通过循环覆盖多个用例或数据分块。

#### Lines 3654-3656

```python

    with pytest.raises(CompilationError) as e:
        run_parser(kernel)
```
- **EN:** Invokes `pytest.raises`, `run_parser` to execute the test logic. Uses `pytest.raises` to confirm expected failure paths. Uses context managers to control resources or expectations.
- **CN:** 调用 `pytest.raises`、`run_parser` 执行测试逻辑。 使用 `pytest.raises` 确认预期的失败路径。 使用上下文管理器控制资源或预期行为。

#### Lines 3657-3658

```python

    assert "For lower bound must be a scalar, got" in str(e.value)
```
- **EN:** Validates behavior with 1 assertion(s).
- **CN:** 通过 1 个断言验证行为。

#### Lines 3659-3661

```python

    @gluon.jit
    def kernel():
```
- **EN:** Defines the helper function `kernel`. Decorators: `gluon.jit`. Key calls include `ttgl.full`, `ttgl.BlockedLayout`. This scope touches layout transformation reasoning.
- **CN:** 定义辅助函数 `kernel`。 装饰器：`gluon.jit`。 关键调用包括 `ttgl.full`、`ttgl.BlockedLayout`。 该作用域涉及布局变换推理。

##### Lines 3662-3662

```python
        x = ttgl.full([32], 0, ttgl.int32, layout=ttgl.BlockedLayout([1], [32], [1], [0]))
```
- **EN:** Prepares or updates state through `x`. Invokes `ttgl.full`, `ttgl.BlockedLayout` to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `x` 准备或更新状态。 调用 `ttgl.full`、`ttgl.BlockedLayout` 执行测试逻辑。 相关主题：布局变换推理。

##### Lines 3663-3664

```python
        for _ in range(1, x, 1):
            pass
```
- **EN:** Iterates across cases or data tiles.
- **CN:** 通过循环覆盖多个用例或数据分块。

#### Lines 3665-3667

```python

    with pytest.raises(CompilationError) as e:
        run_parser(kernel)
```
- **EN:** Invokes `pytest.raises`, `run_parser` to execute the test logic. Uses `pytest.raises` to confirm expected failure paths. Uses context managers to control resources or expectations.
- **CN:** 调用 `pytest.raises`、`run_parser` 执行测试逻辑。 使用 `pytest.raises` 确认预期的失败路径。 使用上下文管理器控制资源或预期行为。

#### Lines 3668-3669

```python

    assert "For upper bound must be a scalar, got" in str(e.value)
```
- **EN:** Validates behavior with 1 assertion(s).
- **CN:** 通过 1 个断言验证行为。

#### Lines 3670-3672

```python

    @gluon.jit
    def kernel():
```
- **EN:** Defines the helper function `kernel`. Decorators: `gluon.jit`. Key calls include `ttgl.full`, `ttgl.BlockedLayout`. This scope touches layout transformation reasoning.
- **CN:** 定义辅助函数 `kernel`。 装饰器：`gluon.jit`。 关键调用包括 `ttgl.full`、`ttgl.BlockedLayout`。 该作用域涉及布局变换推理。

##### Lines 3673-3673

```python
        x = ttgl.full([32], 0, ttgl.int32, layout=ttgl.BlockedLayout([1], [32], [1], [0]))
```
- **EN:** Prepares or updates state through `x`. Invokes `ttgl.full`, `ttgl.BlockedLayout` to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `x` 准备或更新状态。 调用 `ttgl.full`、`ttgl.BlockedLayout` 执行测试逻辑。 相关主题：布局变换推理。

##### Lines 3674-3675

```python
        for _ in range(1, 10, x):
            pass
```
- **EN:** Iterates across cases or data tiles.
- **CN:** 通过循环覆盖多个用例或数据分块。

#### Lines 3676-3678

```python

    with pytest.raises(CompilationError) as e:
        run_parser(kernel)
```
- **EN:** Invokes `pytest.raises`, `run_parser` to execute the test logic. Uses `pytest.raises` to confirm expected failure paths. Uses context managers to control resources or expectations.
- **CN:** 调用 `pytest.raises`、`run_parser` 执行测试逻辑。 使用 `pytest.raises` 确认预期的失败路径。 使用上下文管理器控制资源或预期行为。

#### Lines 3679-3680

```python

    assert "For step must be a scalar, got" in str(e.value)
```
- **EN:** Validates behavior with 1 assertion(s).
- **CN:** 通过 1 个断言验证行为。

### Lines 3681-3684

```python


@gluon.jit
def amd_tdm_load_kernel(ptr):
```
- **EN:** Defines the helper function `amd_tdm_load_kernel`. Decorators: `gluon.jit`. Parameters: `ptr`. Key calls include `ttgl.PaddedSharedLayout.with_identity_for`, `ttgl.BlockedLayout`, `ttgl.amd.gfx1250.tdm.make_tensor_descriptor`, `ttgl.allocate_shared_memory`, `ttgl.amd.gfx1250.tdm.async_load`, `ttgl.amd.gfx1250.tdm.async_wait`, and 1 more. This scope touches tensor/descriptor metadata, layout transformation reasoning.
- **CN:** 定义辅助函数 `amd_tdm_load_kernel`。 装饰器：`gluon.jit`。 参数：`ptr`。 关键调用包括 `ttgl.PaddedSharedLayout.with_identity_for`、`ttgl.BlockedLayout`、`ttgl.amd.gfx1250.tdm.make_tensor_descriptor`、`ttgl.allocate_shared_memory`、`ttgl.amd.gfx1250.tdm.async_load`、`ttgl.amd.gfx1250.tdm.async_wait` 等另外 1 项。 该作用域涉及张量/描述符元数据、布局变换推理。

#### Lines 3685-3695

```python
    SHARED_LAYOUT: ttgl.constexpr = ttgl.PaddedSharedLayout.with_identity_for([[32, 4]], [16, 64], [1, 0])
    BLOCKED_LAYOUT: ttgl.constexpr = ttgl.BlockedLayout([1, 8], [4, 8], [4, 1], [1, 0])

    desc = ttgl.amd.gfx1250.tdm.make_tensor_descriptor(base=ptr, shape=(32, 128), strides=(128, 1),
                                                       block_shape=(16, 64), layout=SHARED_LAYOUT)

    buffer = ttgl.allocate_shared_memory(desc.dtype, shape=desc.block_shape, layout=desc.layout)
    ttgl.amd.gfx1250.tdm.async_load(desc, offsets=[0, 2], dest=buffer)

    ttgl.amd.gfx1250.tdm.async_wait(0)
    buffer.load(layout=BLOCKED_LAYOUT)
```
- **EN:** Prepares or updates state through `SHARED_LAYOUT`, `BLOCKED_LAYOUT`, `desc`, `buffer`. Invokes `ttgl.PaddedSharedLayout.with_identity_for`, `ttgl.BlockedLayout`, `ttgl.amd.gfx1250.tdm.make_tensor_descriptor`, `ttgl.allocate_shared_memory`, `ttgl.amd.gfx1250.tdm.async_load`, `ttgl.amd.gfx1250.tdm.async_wait`, and 1 more to execute the test logic. Relevant themes: tensor/descriptor metadata, layout transformation reasoning.
- **CN:** 通过 `SHARED_LAYOUT`、`BLOCKED_LAYOUT`、`desc`、`buffer` 准备或更新状态。 调用 `ttgl.PaddedSharedLayout.with_identity_for`、`ttgl.BlockedLayout`、`ttgl.amd.gfx1250.tdm.make_tensor_descriptor`、`ttgl.allocate_shared_memory`、`ttgl.amd.gfx1250.tdm.async_load`、`ttgl.amd.gfx1250.tdm.async_wait` 等另外 1 项 执行测试逻辑。 相关主题：张量/描述符元数据、布局变换推理。

### Lines 3696-3700

```python


@pytest.mark.parametrize("target", [HIP_TARGET_GFX1250])
def test_amd_tdm_load(target):
```
- **EN:** Defines the test function `test_amd_tdm_load`. Decorators: `pytest.mark.parametrize('target', [HIP_TARGET_GFX1250])`. Parameters: `target`. Key calls include `pytest.mark.parametrize`, `MockTensor`, `run_parser`, `expecttest.assert_expected_inline`, `anonymize_ir`, `make_args`, and 1 more. This scope touches pytest parametrization, tensor/descriptor metadata, debugging and inspection paths.
- **CN:** 定义测试函数 `test_amd_tdm_load`。 装饰器：`pytest.mark.parametrize('target', [HIP_TARGET_GFX1250])`。 参数：`target`。 关键调用包括 `pytest.mark.parametrize`、`MockTensor`、`run_parser`、`expecttest.assert_expected_inline`、`anonymize_ir`、`make_args` 等另外 1 项。 该作用域涉及pytest 参数化、张量/描述符元数据、调试与检查路径。

#### Lines 3701-3725

```python
    ptr = MockTensor(ttgl.float16)
    module = run_parser(amd_tdm_load_kernel, *make_args(ptr), target)
    expecttest.assert_expected_inline(
        anonymize_ir(module.str_nodebug()), """\
#blocked = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [4, 8], warpsPerCTA = [4, 1], order = [1, 0]}>
#shared = #ttg.padded_shared<[32:+4] {order = [1, 0], shape = [16, 64]}>
#smem = #ttg.shared_memory
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "...", "ttg.threads-per-warp" = 32 : i32} {
  tt.func public @amd_tdm_load_kernel(%arg0: !tt.ptr<f16> {tt.divisibility = 16 : i32}) attributes {noinline = false} {
    %c32_i32 = arith.constant 32 : i32
    %c128_i32 = arith.constant 128 : i32
    %c128_i64 = arith.constant 128 : i64
    %c1_i64 = arith.constant 1 : i64
    %0 = tt.make_tensor_descriptor %arg0, [%c32_i32, %c128_i32], [%c128_i64, %c1_i64] : <f16>, <16x64xf16, #shared>
    %1 = ttg.local_alloc : () -> !ttg.memdesc<16x64xf16, #shared, #smem, mutable>
    %c0_i32 = arith.constant 0 : i32
    %c2_i32 = arith.constant 2 : i32
    %c1_i32 = arith.constant 1 : i32
    %2 = amdg.async_tdm_copy_global_to_local %0[%c0_i32, %c2_i32] into %1, pred = %c1_i32 : !tt.tensordesc<16x64xf16, #shared> -> !ttg.memdesc<16x64xf16, #shared, #smem, mutable>
    %3 = amdg.async_tdm_wait  {num = 0 : i32}
    %4 = ttg.local_load %1 : !ttg.memdesc<16x64xf16, #shared, #smem, mutable> -> tensor<16x64xf16, #blocked>
    tt.return
  }
}
""")
```
- **EN:** Prepares or updates state through `ptr`, `module`. Invokes `MockTensor`, `run_parser`, `make_args`, `expecttest.assert_expected_inline`, `anonymize_ir`, `module.str_nodebug` to execute the test logic. Relevant themes: tensor/descriptor metadata, debugging and inspection paths.
- **CN:** 通过 `ptr`、`module` 准备或更新状态。 调用 `MockTensor`、`run_parser`、`make_args`、`expecttest.assert_expected_inline`、`anonymize_ir`、`module.str_nodebug` 执行测试逻辑。 相关主题：张量/描述符元数据、调试与检查路径。

### Lines 3726-3729

```python


@gluon.jit
def amd_host_tdm_load_kernel(desc):
```
- **EN:** Defines the helper function `amd_host_tdm_load_kernel`. Decorators: `gluon.jit`. Parameters: `desc`. Key calls include `ttgl.allocate_shared_memory`, `ttgl.amd.gfx1250.tdm.async_load`, `ttgl.amd.gfx1250.tdm.async_wait`, `buffer.load`, `ttgl.BlockedLayout`. This scope touches layout transformation reasoning.
- **CN:** 定义辅助函数 `amd_host_tdm_load_kernel`。 装饰器：`gluon.jit`。 参数：`desc`。 关键调用包括 `ttgl.allocate_shared_memory`、`ttgl.amd.gfx1250.tdm.async_load`、`ttgl.amd.gfx1250.tdm.async_wait`、`buffer.load`、`ttgl.BlockedLayout`。 该作用域涉及布局变换推理。

#### Lines 3730-3734

```python
    buffer = ttgl.allocate_shared_memory(desc.dtype, shape=desc.block_shape, layout=desc.layout)
    ttgl.amd.gfx1250.tdm.async_load(desc, offsets=[0, 2], dest=buffer)

    ttgl.amd.gfx1250.tdm.async_wait(0)
    buffer.load(layout=ttgl.BlockedLayout([1, 8], [4, 8], [4, 1], [1, 0]))
```
- **EN:** Prepares or updates state through `buffer`. Invokes `ttgl.allocate_shared_memory`, `ttgl.amd.gfx1250.tdm.async_load`, `ttgl.amd.gfx1250.tdm.async_wait`, `buffer.load`, `ttgl.BlockedLayout` to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `buffer` 准备或更新状态。 调用 `ttgl.allocate_shared_memory`、`ttgl.amd.gfx1250.tdm.async_load`、`ttgl.amd.gfx1250.tdm.async_wait`、`buffer.load`、`ttgl.BlockedLayout` 执行测试逻辑。 相关主题：布局变换推理。

### Lines 3735-3739

```python


@pytest.mark.parametrize("target", [HIP_TARGET_GFX1250])
def test_amd_host_tdm_load(target):
```
- **EN:** Defines the test function `test_amd_host_tdm_load`. Decorators: `pytest.mark.parametrize('target', [HIP_TARGET_GFX1250])`. Parameters: `target`. Key calls include `pytest.mark.parametrize`, `MockTensor`, `ttgl.PaddedSharedLayout.with_identity_for`, `gluon.amd.gfx1250.TensorDescriptor.from_tensor`, `run_parser`, `expecttest.assert_expected_inline`, and 3 more. This scope touches pytest parametrization, tensor/descriptor metadata, debugging and inspection paths, layout transformation reasoning.
- **CN:** 定义测试函数 `test_amd_host_tdm_load`。 装饰器：`pytest.mark.parametrize('target', [HIP_TARGET_GFX1250])`。 参数：`target`。 关键调用包括 `pytest.mark.parametrize`、`MockTensor`、`ttgl.PaddedSharedLayout.with_identity_for`、`gluon.amd.gfx1250.TensorDescriptor.from_tensor`、`run_parser`、`expecttest.assert_expected_inline` 等另外 3 项。 该作用域涉及pytest 参数化、张量/描述符元数据、调试与检查路径、布局变换推理。

#### Lines 3740-3761

```python
    ptr = MockTensor(ttgl.float16, shape=(32, 128))
    layout = ttgl.PaddedSharedLayout.with_identity_for([[32, 4]], [16, 64], [1, 0])
    desc = gluon.amd.gfx1250.TensorDescriptor.from_tensor(ptr, block_shape=(16, 64), layout=layout)
    module = run_parser(amd_host_tdm_load_kernel, *make_args(desc), target)
    expecttest.assert_expected_inline(
        anonymize_ir(module.str_nodebug()), """\
#blocked = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [4, 8], warpsPerCTA = [4, 1], order = [1, 0]}>
#shared = #ttg.padded_shared<[32:+4] {order = [1, 0], shape = [16, 64]}>
#smem = #ttg.shared_memory
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "...", "ttg.threads-per-warp" = 32 : i32} {
  tt.func public @amd_host_tdm_load_kernel(%arg0: !tt.tensordesc<16x64xf16, #shared>, %arg1: i32, %arg2: i32, %arg3: i64, %arg4: i64) attributes {noinline = false} {
    %0 = ttg.local_alloc : () -> !ttg.memdesc<16x64xf16, #shared, #smem, mutable>
    %c0_i32 = arith.constant 0 : i32
    %c2_i32 = arith.constant 2 : i32
    %c1_i32 = arith.constant 1 : i32
    %1 = amdg.async_tdm_copy_global_to_local %arg0[%c0_i32, %c2_i32] into %0, pred = %c1_i32 : !tt.tensordesc<16x64xf16, #shared> -> !ttg.memdesc<16x64xf16, #shared, #smem, mutable>
    %2 = amdg.async_tdm_wait  {num = 0 : i32}
    %3 = ttg.local_load %0 : !ttg.memdesc<16x64xf16, #shared, #smem, mutable> -> tensor<16x64xf16, #blocked>
    tt.return
  }
}
""")
```
- **EN:** Prepares or updates state through `ptr`, `layout`, `desc`, `module`. Invokes `MockTensor`, `ttgl.PaddedSharedLayout.with_identity_for`, `gluon.amd.gfx1250.TensorDescriptor.from_tensor`, `run_parser`, `make_args`, `expecttest.assert_expected_inline`, and 2 more to execute the test logic. Relevant themes: tensor/descriptor metadata, debugging and inspection paths, layout transformation reasoning.
- **CN:** 通过 `ptr`、`layout`、`desc`、`module` 准备或更新状态。 调用 `MockTensor`、`ttgl.PaddedSharedLayout.with_identity_for`、`gluon.amd.gfx1250.TensorDescriptor.from_tensor`、`run_parser`、`make_args`、`expecttest.assert_expected_inline` 等另外 2 项 执行测试逻辑。 相关主题：张量/描述符元数据、调试与检查路径、布局变换推理。

### Lines 3762-3765

```python


@gluon.jit
def amd_tdm_store_kernel(ptr):
```
- **EN:** Defines the helper function `amd_tdm_store_kernel`. Decorators: `gluon.jit`. Parameters: `ptr`. Key calls include `ttgl.SwizzledSharedLayout`, `ttgl.BlockedLayout`, `ttgl.amd.gfx1250.tdm.make_tensor_descriptor`, `ttgl.full`, `ttgl.allocate_shared_memory`, `ttgl.amd.gfx1250.tdm.async_store`, and 1 more. This scope touches tensor/descriptor metadata, layout transformation reasoning.
- **CN:** 定义辅助函数 `amd_tdm_store_kernel`。 装饰器：`gluon.jit`。 参数：`ptr`。 关键调用包括 `ttgl.SwizzledSharedLayout`、`ttgl.BlockedLayout`、`ttgl.amd.gfx1250.tdm.make_tensor_descriptor`、`ttgl.full`、`ttgl.allocate_shared_memory`、`ttgl.amd.gfx1250.tdm.async_store` 等另外 1 项。 该作用域涉及张量/描述符元数据、布局变换推理。

#### Lines 3766-3776

```python
    SHARED_LAYOUT: ttgl.constexpr = ttgl.SwizzledSharedLayout(1, 1, 1, [1, 0])
    BLOCKED_LAYOUT: ttgl.constexpr = ttgl.BlockedLayout([1, 8], [4, 8], [4, 1], [1, 0])

    desc = ttgl.amd.gfx1250.tdm.make_tensor_descriptor(base=ptr, shape=(32, 128), strides=(128, 1),
                                                       block_shape=(16, 64), layout=SHARED_LAYOUT)

    value = ttgl.full([16, 64], 1.0, ttgl.float16, layout=BLOCKED_LAYOUT)
    buffer = ttgl.allocate_shared_memory(desc.dtype, desc.block_shape, desc.layout, value)

    ttgl.amd.gfx1250.tdm.async_store(desc, offsets=[0, 2], src=buffer)
    ttgl.amd.gfx1250.tdm.async_wait(0)
```
- **EN:** Prepares or updates state through `SHARED_LAYOUT`, `BLOCKED_LAYOUT`, `desc`, `value`, `buffer`. Invokes `ttgl.SwizzledSharedLayout`, `ttgl.BlockedLayout`, `ttgl.amd.gfx1250.tdm.make_tensor_descriptor`, `ttgl.full`, `ttgl.allocate_shared_memory`, `ttgl.amd.gfx1250.tdm.async_store`, and 1 more to execute the test logic. Relevant themes: tensor/descriptor metadata, layout transformation reasoning.
- **CN:** 通过 `SHARED_LAYOUT`、`BLOCKED_LAYOUT`、`desc`、`value`、`buffer` 准备或更新状态。 调用 `ttgl.SwizzledSharedLayout`、`ttgl.BlockedLayout`、`ttgl.amd.gfx1250.tdm.make_tensor_descriptor`、`ttgl.full`、`ttgl.allocate_shared_memory`、`ttgl.amd.gfx1250.tdm.async_store` 等另外 1 项 执行测试逻辑。 相关主题：张量/描述符元数据、布局变换推理。

### Lines 3777-3781

```python


@pytest.mark.parametrize("target", [HIP_TARGET_GFX1250])
def test_amd_tdm_store(target):
```
- **EN:** Defines the test function `test_amd_tdm_store`. Decorators: `pytest.mark.parametrize('target', [HIP_TARGET_GFX1250])`. Parameters: `target`. Key calls include `pytest.mark.parametrize`, `MockTensor`, `run_parser`, `expecttest.assert_expected_inline`, `anonymize_ir`, `make_args`, and 1 more. This scope touches pytest parametrization, tensor/descriptor metadata, debugging and inspection paths.
- **CN:** 定义测试函数 `test_amd_tdm_store`。 装饰器：`pytest.mark.parametrize('target', [HIP_TARGET_GFX1250])`。 参数：`target`。 关键调用包括 `pytest.mark.parametrize`、`MockTensor`、`run_parser`、`expecttest.assert_expected_inline`、`anonymize_ir`、`make_args` 等另外 1 项。 该作用域涉及pytest 参数化、张量/描述符元数据、调试与检查路径。

#### Lines 3782-3806

```python
    ptr = MockTensor(ttgl.float16)
    module = run_parser(amd_tdm_store_kernel, *make_args(ptr), target)
    expecttest.assert_expected_inline(
        anonymize_ir(module.str_nodebug()), """\
#blocked = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [4, 8], warpsPerCTA = [4, 1], order = [1, 0]}>
#shared = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [1, 0]}>
#smem = #ttg.shared_memory
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "...", "ttg.threads-per-warp" = 32 : i32} {
  tt.func public @amd_tdm_store_kernel(%arg0: !tt.ptr<f16> {tt.divisibility = 16 : i32}) attributes {noinline = false} {
    %c32_i32 = arith.constant 32 : i32
    %c128_i32 = arith.constant 128 : i32
    %c128_i64 = arith.constant 128 : i64
    %c1_i64 = arith.constant 1 : i64
    %0 = tt.make_tensor_descriptor %arg0, [%c32_i32, %c128_i32], [%c128_i64, %c1_i64] : <f16>, <16x64xf16, #shared>
    %cst = arith.constant 1.000000e+00 : f16
    %cst_0 = arith.constant dense<1.000000e+00> : tensor<16x64xf16, #blocked>
    %1 = ttg.local_alloc %cst_0 : (tensor<16x64xf16, #blocked>) -> !ttg.memdesc<16x64xf16, #shared, #smem, mutable>
    %c0_i32 = arith.constant 0 : i32
    %c2_i32 = arith.constant 2 : i32
    amdg.async_tdm_copy_local_to_global %0[%c0_i32, %c2_i32] from %1 : !ttg.memdesc<16x64xf16, #shared, #smem, mutable> -> !tt.tensordesc<16x64xf16, #shared>
    %2 = amdg.async_tdm_wait  {num = 0 : i32}
    tt.return
  }
}
""")
```
- **EN:** Prepares or updates state through `ptr`, `module`. Invokes `MockTensor`, `run_parser`, `make_args`, `expecttest.assert_expected_inline`, `anonymize_ir`, `module.str_nodebug` to execute the test logic. Relevant themes: tensor/descriptor metadata, debugging and inspection paths.
- **CN:** 通过 `ptr`、`module` 准备或更新状态。 调用 `MockTensor`、`run_parser`、`make_args`、`expecttest.assert_expected_inline`、`anonymize_ir`、`module.str_nodebug` 执行测试逻辑。 相关主题：张量/描述符元数据、调试与检查路径。

### Lines 3807-3810

```python


@gluon.jit
def amd_tdm_gather_kernel(ptr):
```
- **EN:** Defines the helper function `amd_tdm_gather_kernel`. Decorators: `gluon.jit`. Parameters: `ptr`. Key calls include `ttgl.num_warps`, `ttgl.PaddedSharedLayout.with_identity_for`, `ttgl.BlockedLayout`, `ttgl.SliceLayout`, `ttgl.amd.gfx1250.tdm.make_tensor_descriptor`, `ttgl.arange`, and 4 more. This scope touches tensor/descriptor metadata, layout transformation reasoning.
- **CN:** 定义辅助函数 `amd_tdm_gather_kernel`。 装饰器：`gluon.jit`。 参数：`ptr`。 关键调用包括 `ttgl.num_warps`、`ttgl.PaddedSharedLayout.with_identity_for`、`ttgl.BlockedLayout`、`ttgl.SliceLayout`、`ttgl.amd.gfx1250.tdm.make_tensor_descriptor`、`ttgl.arange` 等另外 4 项。 该作用域涉及张量/描述符元数据、布局变换推理。

#### Lines 3811-3825

```python
    num_warps: ttgl.constexpr = ttgl.num_warps()
    NUM_INDICES: ttgl.constexpr = 16
    SHARED_LAYOUT: ttgl.constexpr = ttgl.PaddedSharedLayout.with_identity_for([[32, 4]], [16, 64], [1, 0])
    BLOCKED_LAYOUT: ttgl.constexpr = ttgl.BlockedLayout([NUM_INDICES, 1], [1, 32], [1, num_warps], [1, 0])
    ROW_IDX_LAYOUT: ttgl.constexpr = ttgl.SliceLayout(1, BLOCKED_LAYOUT)

    desc = ttgl.amd.gfx1250.tdm.make_tensor_descriptor(base=ptr, shape=(32, 128), strides=(128, 1),
                                                       block_shape=(16, 64), layout=SHARED_LAYOUT)

    row_indices = ttgl.arange(0, NUM_INDICES, layout=ROW_IDX_LAYOUT)
    buffer = ttgl.allocate_shared_memory(desc.dtype, shape=desc.block_shape, layout=desc.layout)
    ttgl.amd.gfx1250.tdm.async_gather(desc, src_row_indices=row_indices, src_col_offset=0, dst=buffer)

    ttgl.amd.gfx1250.tdm.async_wait(0)
    buffer.load(layout=BLOCKED_LAYOUT)
```
- **EN:** Prepares or updates state through `num_warps`, `NUM_INDICES`, `SHARED_LAYOUT`, `BLOCKED_LAYOUT`, `ROW_IDX_LAYOUT`, `desc`, `row_indices`, `buffer`. Invokes `ttgl.num_warps`, `ttgl.PaddedSharedLayout.with_identity_for`, `ttgl.BlockedLayout`, `ttgl.SliceLayout`, `ttgl.amd.gfx1250.tdm.make_tensor_descriptor`, `ttgl.arange`, and 4 more to execute the test logic. Relevant themes: tensor/descriptor metadata, layout transformation reasoning.
- **CN:** 通过 `num_warps`、`NUM_INDICES`、`SHARED_LAYOUT`、`BLOCKED_LAYOUT`、`ROW_IDX_LAYOUT`、`desc`、`row_indices`、`buffer` 准备或更新状态。 调用 `ttgl.num_warps`、`ttgl.PaddedSharedLayout.with_identity_for`、`ttgl.BlockedLayout`、`ttgl.SliceLayout`、`ttgl.amd.gfx1250.tdm.make_tensor_descriptor`、`ttgl.arange` 等另外 4 项 执行测试逻辑。 相关主题：张量/描述符元数据、布局变换推理。

### Lines 3826-3830

```python


@pytest.mark.parametrize("target", [HIP_TARGET_GFX1250])
def test_amd_tdm_gather(target):
```
- **EN:** Defines the test function `test_amd_tdm_gather`. Decorators: `pytest.mark.parametrize('target', [HIP_TARGET_GFX1250])`. Parameters: `target`. Key calls include `pytest.mark.parametrize`, `MockTensor`, `run_parser`, `expecttest.assert_expected_inline`, `anonymize_ir`, `make_args`, and 1 more. This scope touches pytest parametrization, tensor/descriptor metadata, debugging and inspection paths.
- **CN:** 定义测试函数 `test_amd_tdm_gather`。 装饰器：`pytest.mark.parametrize('target', [HIP_TARGET_GFX1250])`。 参数：`target`。 关键调用包括 `pytest.mark.parametrize`、`MockTensor`、`run_parser`、`expecttest.assert_expected_inline`、`anonymize_ir`、`make_args` 等另外 1 项。 该作用域涉及pytest 参数化、张量/描述符元数据、调试与检查路径。

#### Lines 3831-3855

```python
    ptr = MockTensor(ttgl.float16)
    module = run_parser(amd_tdm_gather_kernel, *make_args(ptr), target)
    expecttest.assert_expected_inline(
        anonymize_ir(module.str_nodebug()), """\
#blocked = #ttg.blocked<{sizePerThread = [16, 1], threadsPerWarp = [1, 32], warpsPerCTA = [1, 4], order = [1, 0]}>
#shared = #ttg.padded_shared<[32:+4] {order = [1, 0], shape = [16, 64]}>
#smem = #ttg.shared_memory
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "...", "ttg.threads-per-warp" = 32 : i32} {
  tt.func public @amd_tdm_gather_kernel(%arg0: !tt.ptr<f16> {tt.divisibility = 16 : i32}) attributes {noinline = false} {
    %c32_i32 = arith.constant 32 : i32
    %c128_i32 = arith.constant 128 : i32
    %c128_i64 = arith.constant 128 : i64
    %c1_i64 = arith.constant 1 : i64
    %0 = tt.make_tensor_descriptor %arg0, [%c32_i32, %c128_i32], [%c128_i64, %c1_i64] : <f16>, <16x64xf16, #shared>
    %1 = tt.make_range {end = 16 : i32, start = 0 : i32} : tensor<16xi32, #ttg.slice<{dim = 1, parent = #blocked}>>
    %2 = ttg.local_alloc : () -> !ttg.memdesc<16x64xf16, #shared, #smem, mutable>
    %c0_i32 = arith.constant 0 : i32
    %c1_i32 = arith.constant 1 : i32
    %3 = amdg.async_tdm_gather %0[%1, %c0_i32] to %2, pred = %c1_i32 : tensor<16xi32, #ttg.slice<{dim = 1, parent = #blocked}>>, !ttg.memdesc<16x64xf16, #shared, #smem, mutable> -> !tt.tensordesc<16x64xf16, #shared>
    %4 = amdg.async_tdm_wait  {num = 0 : i32}
    %5 = ttg.local_load %2 : !ttg.memdesc<16x64xf16, #shared, #smem, mutable> -> tensor<16x64xf16, #blocked>
    tt.return
  }
}
""")
```
- **EN:** Prepares or updates state through `ptr`, `module`. Invokes `MockTensor`, `run_parser`, `make_args`, `expecttest.assert_expected_inline`, `anonymize_ir`, `module.str_nodebug` to execute the test logic. Relevant themes: tensor/descriptor metadata, debugging and inspection paths.
- **CN:** 通过 `ptr`、`module` 准备或更新状态。 调用 `MockTensor`、`run_parser`、`make_args`、`expecttest.assert_expected_inline`、`anonymize_ir`、`module.str_nodebug` 执行测试逻辑。 相关主题：张量/描述符元数据、调试与检查路径。

### Lines 3856-3859

```python


@gluon.jit
def amd_tdm_scatter_kernel(ptr):
```
- **EN:** Defines the helper function `amd_tdm_scatter_kernel`. Decorators: `gluon.jit`. Parameters: `ptr`. Key calls include `ttgl.num_warps`, `ttgl.SwizzledSharedLayout`, `ttgl.BlockedLayout`, `ttgl.SliceLayout`, `ttgl.amd.gfx1250.tdm.make_tensor_descriptor`, `ttgl.full`, and 4 more. This scope touches tensor/descriptor metadata, layout transformation reasoning.
- **CN:** 定义辅助函数 `amd_tdm_scatter_kernel`。 装饰器：`gluon.jit`。 参数：`ptr`。 关键调用包括 `ttgl.num_warps`、`ttgl.SwizzledSharedLayout`、`ttgl.BlockedLayout`、`ttgl.SliceLayout`、`ttgl.amd.gfx1250.tdm.make_tensor_descriptor`、`ttgl.full` 等另外 4 项。 该作用域涉及张量/描述符元数据、布局变换推理。

#### Lines 3860-3874

```python
    num_warps: ttgl.constexpr = ttgl.num_warps()
    NUM_INDICES: ttgl.constexpr = 16
    SHARED_LAYOUT: ttgl.constexpr = ttgl.SwizzledSharedLayout(vec=1, per_phase=1, max_phase=1, order=[1, 0])
    BLOCKED_LAYOUT: ttgl.constexpr = ttgl.BlockedLayout([NUM_INDICES, 1], [1, 32], [1, num_warps], [1, 0])
    ROW_IDX_LAYOUT: ttgl.constexpr = ttgl.SliceLayout(1, BLOCKED_LAYOUT)

    desc = ttgl.amd.gfx1250.tdm.make_tensor_descriptor(base=ptr, shape=(32, 128), strides=(128, 1),
                                                       block_shape=(16, 64), layout=SHARED_LAYOUT)

    value = ttgl.full([16, 64], 1.0, ttgl.float16, layout=BLOCKED_LAYOUT)
    buffer = ttgl.allocate_shared_memory(desc.dtype, desc.block_shape, desc.layout, value)

    row_indices = ttgl.arange(0, NUM_INDICES, layout=ROW_IDX_LAYOUT)
    ttgl.amd.gfx1250.tdm.async_scatter(desc, dst_row_indices=row_indices, dst_col_offset=0, src=buffer)
    ttgl.amd.gfx1250.tdm.async_wait(0)
```
- **EN:** Prepares or updates state through `num_warps`, `NUM_INDICES`, `SHARED_LAYOUT`, `BLOCKED_LAYOUT`, `ROW_IDX_LAYOUT`, `desc`, `value`, `buffer`, and 1 more. Invokes `ttgl.num_warps`, `ttgl.SwizzledSharedLayout`, `ttgl.BlockedLayout`, `ttgl.SliceLayout`, `ttgl.amd.gfx1250.tdm.make_tensor_descriptor`, `ttgl.full`, and 4 more to execute the test logic. Relevant themes: tensor/descriptor metadata, layout transformation reasoning.
- **CN:** 通过 `num_warps`、`NUM_INDICES`、`SHARED_LAYOUT`、`BLOCKED_LAYOUT`、`ROW_IDX_LAYOUT`、`desc`、`value`、`buffer` 等另外 1 项 准备或更新状态。 调用 `ttgl.num_warps`、`ttgl.SwizzledSharedLayout`、`ttgl.BlockedLayout`、`ttgl.SliceLayout`、`ttgl.amd.gfx1250.tdm.make_tensor_descriptor`、`ttgl.full` 等另外 4 项 执行测试逻辑。 相关主题：张量/描述符元数据、布局变换推理。

### Lines 3875-3879

```python


@pytest.mark.parametrize("target", [HIP_TARGET_GFX1250])
def test_amd_tdm_scatter(target):
```
- **EN:** Defines the test function `test_amd_tdm_scatter`. Decorators: `pytest.mark.parametrize('target', [HIP_TARGET_GFX1250])`. Parameters: `target`. Key calls include `pytest.mark.parametrize`, `MockTensor`, `run_parser`, `expecttest.assert_expected_inline`, `anonymize_ir`, `make_args`, and 1 more. This scope touches pytest parametrization, tensor/descriptor metadata, debugging and inspection paths.
- **CN:** 定义测试函数 `test_amd_tdm_scatter`。 装饰器：`pytest.mark.parametrize('target', [HIP_TARGET_GFX1250])`。 参数：`target`。 关键调用包括 `pytest.mark.parametrize`、`MockTensor`、`run_parser`、`expecttest.assert_expected_inline`、`anonymize_ir`、`make_args` 等另外 1 项。 该作用域涉及pytest 参数化、张量/描述符元数据、调试与检查路径。

#### Lines 3880-3904

```python
    ptr = MockTensor(ttgl.float16)
    module = run_parser(amd_tdm_scatter_kernel, *make_args(ptr), target)
    expecttest.assert_expected_inline(
        anonymize_ir(module.str_nodebug()), """\
#blocked = #ttg.blocked<{sizePerThread = [16, 1], threadsPerWarp = [1, 32], warpsPerCTA = [1, 4], order = [1, 0]}>
#shared = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [1, 0]}>
#smem = #ttg.shared_memory
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "...", "ttg.threads-per-warp" = 32 : i32} {
  tt.func public @amd_tdm_scatter_kernel(%arg0: !tt.ptr<f16> {tt.divisibility = 16 : i32}) attributes {noinline = false} {
    %c32_i32 = arith.constant 32 : i32
    %c128_i32 = arith.constant 128 : i32
    %c128_i64 = arith.constant 128 : i64
    %c1_i64 = arith.constant 1 : i64
    %0 = tt.make_tensor_descriptor %arg0, [%c32_i32, %c128_i32], [%c128_i64, %c1_i64] : <f16>, <16x64xf16, #shared>
    %cst = arith.constant 1.000000e+00 : f16
    %cst_0 = arith.constant dense<1.000000e+00> : tensor<16x64xf16, #blocked>
    %1 = ttg.local_alloc %cst_0 : (tensor<16x64xf16, #blocked>) -> !ttg.memdesc<16x64xf16, #shared, #smem, mutable>
    %2 = tt.make_range {end = 16 : i32, start = 0 : i32} : tensor<16xi32, #ttg.slice<{dim = 1, parent = #blocked}>>
    %c0_i32 = arith.constant 0 : i32
    %3 = amdg.async_tdm_scatter %0[%2, %c0_i32] from %1 : tensor<16xi32, #ttg.slice<{dim = 1, parent = #blocked}>>, !ttg.memdesc<16x64xf16, #shared, #smem, mutable> -> !tt.tensordesc<16x64xf16, #shared>
    %4 = amdg.async_tdm_wait  {num = 0 : i32}
    tt.return
  }
}
""")
```
- **EN:** Prepares or updates state through `ptr`, `module`. Invokes `MockTensor`, `run_parser`, `make_args`, `expecttest.assert_expected_inline`, `anonymize_ir`, `module.str_nodebug` to execute the test logic. Relevant themes: tensor/descriptor metadata, debugging and inspection paths.
- **CN:** 通过 `ptr`、`module` 准备或更新状态。 调用 `MockTensor`、`run_parser`、`make_args`、`expecttest.assert_expected_inline`、`anonymize_ir`、`module.str_nodebug` 执行测试逻辑。 相关主题：张量/描述符元数据、调试与检查路径。

### Lines 3905-3908

```python


@gluon.jit
def amd_tdm_load_pred_kernel(ptr, n):
```
- **EN:** Defines the helper function `amd_tdm_load_pred_kernel`. Decorators: `gluon.jit`. Parameters: `ptr`, `n`. Key calls include `ttgl.PaddedSharedLayout.with_identity_for`, `ttgl.amd.gfx1250.tdm.make_tensor_descriptor`, `ttgl.allocate_shared_memory`, `ttgl.amd.gfx1250.tdm.async_load`. This scope touches tensor/descriptor metadata, layout transformation reasoning.
- **CN:** 定义辅助函数 `amd_tdm_load_pred_kernel`。 装饰器：`gluon.jit`。 参数：`ptr`、`n`。 关键调用包括 `ttgl.PaddedSharedLayout.with_identity_for`、`ttgl.amd.gfx1250.tdm.make_tensor_descriptor`、`ttgl.allocate_shared_memory`、`ttgl.amd.gfx1250.tdm.async_load`。 该作用域涉及张量/描述符元数据、布局变换推理。

#### Lines 3909-3916

```python
    layout: ttgl.constexpr = ttgl.PaddedSharedLayout.with_identity_for([[32, 4]], [64, 64], [1, 0])
    desc = ttgl.amd.gfx1250.tdm.make_tensor_descriptor(base=ptr, shape=(64, 64), strides=(64, 1), block_shape=(64, 64),
                                                       layout=layout)
    buffer = ttgl.allocate_shared_memory(desc.dtype, shape=desc.block_shape, layout=desc.layout)
    ttgl.amd.gfx1250.tdm.async_load(desc, offsets=[0, 2], dest=buffer, pred=False)
    ttgl.amd.gfx1250.tdm.async_load(desc, offsets=[0, 2], dest=buffer, pred=True)
    ttgl.amd.gfx1250.tdm.async_load(desc, offsets=[0, 2], dest=buffer, pred=n < 64)
    ttgl.amd.gfx1250.tdm.async_load(desc, offsets=[0, 2], dest=buffer, pred=n & 1)
```
- **EN:** Prepares or updates state through `layout`, `desc`, `buffer`. Invokes `ttgl.PaddedSharedLayout.with_identity_for`, `ttgl.amd.gfx1250.tdm.make_tensor_descriptor`, `ttgl.allocate_shared_memory`, `ttgl.amd.gfx1250.tdm.async_load` to execute the test logic. Relevant themes: tensor/descriptor metadata, layout transformation reasoning.
- **CN:** 通过 `layout`、`desc`、`buffer` 准备或更新状态。 调用 `ttgl.PaddedSharedLayout.with_identity_for`、`ttgl.amd.gfx1250.tdm.make_tensor_descriptor`、`ttgl.allocate_shared_memory`、`ttgl.amd.gfx1250.tdm.async_load` 执行测试逻辑。 相关主题：张量/描述符元数据、布局变换推理。

### Lines 3917-3921

```python


@pytest.mark.parametrize("target", [HIP_TARGET_GFX1250])
def test_amd_tdm_load_pred(target):
```
- **EN:** Defines the test function `test_amd_tdm_load_pred`. Decorators: `pytest.mark.parametrize('target', [HIP_TARGET_GFX1250])`. Parameters: `target`. Key calls include `pytest.mark.parametrize`, `MockTensor`, `run_parser`, `expecttest.assert_expected_inline`, `anonymize_ir`, `make_args`, and 1 more. This scope touches pytest parametrization, tensor/descriptor metadata, debugging and inspection paths.
- **CN:** 定义测试函数 `test_amd_tdm_load_pred`。 装饰器：`pytest.mark.parametrize('target', [HIP_TARGET_GFX1250])`。 参数：`target`。 关键调用包括 `pytest.mark.parametrize`、`MockTensor`、`run_parser`、`expecttest.assert_expected_inline`、`anonymize_ir`、`make_args` 等另外 1 项。 该作用域涉及pytest 参数化、张量/描述符元数据、调试与检查路径。

#### Lines 3922-3959

```python
    ptr = MockTensor(ttgl.float16)
    module = run_parser(amd_tdm_load_pred_kernel, *make_args(ptr, 32), target)
    expecttest.assert_expected_inline(
        anonymize_ir(module.str_nodebug()), """\
#shared = #ttg.padded_shared<[32:+4] {order = [1, 0], shape = [64, 64]}>
#smem = #ttg.shared_memory
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "...", "ttg.threads-per-warp" = 32 : i32} {
  tt.func public @amd_tdm_load_pred_kernel(%arg0: !tt.ptr<f16> {tt.divisibility = 16 : i32}, %arg1: i32 {tt.divisibility = 16 : i32}) attributes {noinline = false} {
    %c64_i32 = arith.constant 64 : i32
    %c64_i32_0 = arith.constant 64 : i32
    %c64_i64 = arith.constant 64 : i64
    %c1_i64 = arith.constant 1 : i64
    %0 = tt.make_tensor_descriptor %arg0, [%c64_i32, %c64_i32_0], [%c64_i64, %c1_i64] : <f16>, <64x64xf16, #shared>
    %1 = ttg.local_alloc : () -> !ttg.memdesc<64x64xf16, #shared, #smem, mutable>
    %c0_i32 = arith.constant 0 : i32
    %c2_i32 = arith.constant 2 : i32
    %c0_i32_1 = arith.constant 0 : i32
    %2 = amdg.async_tdm_copy_global_to_local %0[%c0_i32, %c2_i32] into %1, pred = %c0_i32_1 : !tt.tensordesc<64x64xf16, #shared> -> !ttg.memdesc<64x64xf16, #shared, #smem, mutable>
    %c0_i32_2 = arith.constant 0 : i32
    %c2_i32_3 = arith.constant 2 : i32
    %c1_i32 = arith.constant 1 : i32
    %3 = amdg.async_tdm_copy_global_to_local %0[%c0_i32_2, %c2_i32_3] into %1, pred = %c1_i32 : !tt.tensordesc<64x64xf16, #shared> -> !ttg.memdesc<64x64xf16, #shared, #smem, mutable>
    %c64_i32_4 = arith.constant 64 : i32
    %4 = arith.cmpi slt, %arg1, %c64_i32_4 : i32
    %c0_i32_5 = arith.constant 0 : i32
    %c2_i32_6 = arith.constant 2 : i32
    %5 = arith.extui %4 : i1 to i32
    %6 = amdg.async_tdm_copy_global_to_local %0[%c0_i32_5, %c2_i32_6] into %1, pred = %5 : !tt.tensordesc<64x64xf16, #shared> -> !ttg.memdesc<64x64xf16, #shared, #smem, mutable>
    %c1_i32_7 = arith.constant 1 : i32
    %c1_i32_8 = arith.constant 1 : i32
    %7 = arith.andi %arg1, %c1_i32_8 : i32
    %c0_i32_9 = arith.constant 0 : i32
    %c2_i32_10 = arith.constant 2 : i32
    %8 = amdg.async_tdm_copy_global_to_local %0[%c0_i32_9, %c2_i32_10] into %1, pred = %7 : !tt.tensordesc<64x64xf16, #shared> -> !ttg.memdesc<64x64xf16, #shared, #smem, mutable>
    tt.return
  }
}
""")
```
- **EN:** Prepares or updates state through `ptr`, `module`. Invokes `MockTensor`, `run_parser`, `make_args`, `expecttest.assert_expected_inline`, `anonymize_ir`, `module.str_nodebug` to execute the test logic. Relevant themes: tensor/descriptor metadata, debugging and inspection paths.
- **CN:** 通过 `ptr`、`module` 准备或更新状态。 调用 `MockTensor`、`run_parser`、`make_args`、`expecttest.assert_expected_inline`、`anonymize_ir`、`module.str_nodebug` 执行测试逻辑。 相关主题：张量/描述符元数据、调试与检查路径。

### Lines 3960-3963

```python


@gluon.jit
def amd_mbarrier_kernel():
```
- **EN:** Defines the helper function `amd_mbarrier_kernel`. Decorators: `gluon.jit`. Key calls include `ttgl.allocate_shared_memory`, `gfx1250_mbarrier.init`, `gfx1250_mbarrier.arrive`, `gfx1250_mbarrier.wait`, `gfx1250_mbarrier.MBarrierLayout`. This scope touches layout transformation reasoning.
- **CN:** 定义辅助函数 `amd_mbarrier_kernel`。 装饰器：`gluon.jit`。 关键调用包括 `ttgl.allocate_shared_memory`、`gfx1250_mbarrier.init`、`gfx1250_mbarrier.arrive`、`gfx1250_mbarrier.wait`、`gfx1250_mbarrier.MBarrierLayout`。 该作用域涉及布局变换推理。

#### Lines 3964-3967

```python
    bar = ttgl.allocate_shared_memory(ttgl.int64, [1], gfx1250_mbarrier.MBarrierLayout())
    gfx1250_mbarrier.init(bar, count=2)
    prior_phase = gfx1250_mbarrier.arrive(bar)
    gfx1250_mbarrier.wait(bar, prior_phase)
```
- **EN:** Prepares or updates state through `bar`, `prior_phase`. Invokes `ttgl.allocate_shared_memory`, `gfx1250_mbarrier.MBarrierLayout`, `gfx1250_mbarrier.init`, `gfx1250_mbarrier.arrive`, `gfx1250_mbarrier.wait` to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `bar`、`prior_phase` 准备或更新状态。 调用 `ttgl.allocate_shared_memory`、`gfx1250_mbarrier.MBarrierLayout`、`gfx1250_mbarrier.init`、`gfx1250_mbarrier.arrive`、`gfx1250_mbarrier.wait` 执行测试逻辑。 相关主题：布局变换推理。

### Lines 3968-3971

```python


@pytest.mark.parametrize("target", [HIP_TARGET_GFX1250])
def test_amd_mbarrier(target):
```
- **EN:** Defines the test function `test_amd_mbarrier`. Decorators: `pytest.mark.parametrize('target', [HIP_TARGET_GFX1250])`. Parameters: `target`. Key calls include `pytest.mark.parametrize`, `run_parser`, `expecttest.assert_expected_inline`, `anonymize_ir`, `mod.str_nodebug`. This scope touches pytest parametrization, debugging and inspection paths.
- **CN:** 定义测试函数 `test_amd_mbarrier`。 装饰器：`pytest.mark.parametrize('target', [HIP_TARGET_GFX1250])`。 参数：`target`。 关键调用包括 `pytest.mark.parametrize`、`run_parser`、`expecttest.assert_expected_inline`、`anonymize_ir`、`mod.str_nodebug`。 该作用域涉及pytest 参数化、调试与检查路径。

#### Lines 3972-3986

```python
    mod = run_parser(amd_mbarrier_kernel, target=target)
    expecttest.assert_expected_inline(
        anonymize_ir(mod.str_nodebug()), """\
#shared = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0]}>
#smem = #ttg.shared_memory
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "...", "ttg.threads-per-warp" = 32 : i32} {
  tt.func public @amd_mbarrier_kernel() attributes {noinline = false} {
    %0 = ttg.local_alloc : () -> !ttg.memdesc<1xi64, #shared, #smem, mutable>
    amdg.init_barrier %0, 2 : !ttg.memdesc<1xi64, #shared, #smem, mutable>
    %1 = amdg.arrive_barrier %0, 1 : !ttg.memdesc<1xi64, #shared, #smem, mutable> -> i32
    amdg.wait_barrier %0, %1 : !ttg.memdesc<1xi64, #shared, #smem, mutable>
    tt.return
  }
}
""")
```
- **EN:** Prepares or updates state through `mod`. Invokes `run_parser`, `expecttest.assert_expected_inline`, `anonymize_ir`, `mod.str_nodebug` to execute the test logic. Relevant themes: debugging and inspection paths.
- **CN:** 通过 `mod` 准备或更新状态。 调用 `run_parser`、`expecttest.assert_expected_inline`、`anonymize_ir`、`mod.str_nodebug` 执行测试逻辑。 相关主题：调试与检查路径。

### Lines 3987-3990

```python


@gluon.jit
def amd_async_copy_mbarrier_kernel(ptr):
```
- **EN:** Defines the helper function `amd_async_copy_mbarrier_kernel`. Decorators: `gluon.jit`. Parameters: `ptr`. Key calls include `ttgl.allocate_shared_memory`, `gfx1250_async_copy.mbarrier_arrive`, `gfx1250_mbarrier.MBarrierLayout`. This scope touches layout transformation reasoning.
- **CN:** 定义辅助函数 `amd_async_copy_mbarrier_kernel`。 装饰器：`gluon.jit`。 参数：`ptr`。 关键调用包括 `ttgl.allocate_shared_memory`、`gfx1250_async_copy.mbarrier_arrive`、`gfx1250_mbarrier.MBarrierLayout`。 该作用域涉及布局变换推理。

#### Lines 3991-3992

```python
    bar = ttgl.allocate_shared_memory(ttgl.int64, [1], gfx1250_mbarrier.MBarrierLayout())
    gfx1250_async_copy.mbarrier_arrive(bar)
```
- **EN:** Prepares or updates state through `bar`. Invokes `ttgl.allocate_shared_memory`, `gfx1250_mbarrier.MBarrierLayout`, `gfx1250_async_copy.mbarrier_arrive` to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `bar` 准备或更新状态。 调用 `ttgl.allocate_shared_memory`、`gfx1250_mbarrier.MBarrierLayout`、`gfx1250_async_copy.mbarrier_arrive` 执行测试逻辑。 相关主题：布局变换推理。

### Lines 3993-3996

```python


@pytest.mark.parametrize("target", [HIP_TARGET_GFX1250])
def test_amd_async_copy_mbarrier(target):
```
- **EN:** Defines the test function `test_amd_async_copy_mbarrier`. Decorators: `pytest.mark.parametrize('target', [HIP_TARGET_GFX1250])`. Parameters: `target`. Key calls include `pytest.mark.parametrize`, `MockTensor`, `run_parser`, `expecttest.assert_expected_inline`, `anonymize_ir`, `make_args`, and 1 more. This scope touches pytest parametrization, debugging and inspection paths.
- **CN:** 定义测试函数 `test_amd_async_copy_mbarrier`。 装饰器：`pytest.mark.parametrize('target', [HIP_TARGET_GFX1250])`。 参数：`target`。 关键调用包括 `pytest.mark.parametrize`、`MockTensor`、`run_parser`、`expecttest.assert_expected_inline`、`anonymize_ir`、`make_args` 等另外 1 项。 该作用域涉及pytest 参数化、调试与检查路径。

#### Lines 3997-4010

```python
    ptr = MockTensor(ttgl.float16)
    mod = run_parser(amd_async_copy_mbarrier_kernel, *make_args(ptr), target=target)
    expecttest.assert_expected_inline(
        anonymize_ir(mod.str_nodebug()), """\
#shared = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0]}>
#smem = #ttg.shared_memory
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "...", "ttg.threads-per-warp" = 32 : i32} {
  tt.func public @amd_async_copy_mbarrier_kernel(%arg0: !tt.ptr<f16> {tt.divisibility = 16 : i32}) attributes {noinline = false} {
    %0 = ttg.local_alloc : () -> !ttg.memdesc<1xi64, #shared, #smem, mutable>
    amdg.async_copy_mbarrier_arrive %0 : !ttg.memdesc<1xi64, #shared, #smem, mutable>
    tt.return
  }
}
""")
```
- **EN:** Prepares or updates state through `ptr`, `mod`. Invokes `MockTensor`, `run_parser`, `make_args`, `expecttest.assert_expected_inline`, `anonymize_ir`, `mod.str_nodebug` to execute the test logic. Relevant themes: debugging and inspection paths.
- **CN:** 通过 `ptr`、`mod` 准备或更新状态。 调用 `MockTensor`、`run_parser`、`make_args`、`expecttest.assert_expected_inline`、`anonymize_ir`、`mod.str_nodebug` 执行测试逻辑。 相关主题：调试与检查路径。

### Lines 4011-4014

```python


@gluon.jit
def amd_tdm_load_mbarrier_kernel(ptr):
```
- **EN:** Defines the helper function `amd_tdm_load_mbarrier_kernel`. Decorators: `gluon.jit`. Parameters: `ptr`. Key calls include `ttgl.PaddedSharedLayout.with_identity_for`, `ttgl.BlockedLayout`, `ttgl.amd.gfx1250.tdm.make_tensor_descriptor`, `ttgl.allocate_shared_memory`, `gfx1250_mbarrier.init`, `ttgl.amd.gfx1250.tdm.async_load`, and 2 more. This scope touches tensor/descriptor metadata, layout transformation reasoning.
- **CN:** 定义辅助函数 `amd_tdm_load_mbarrier_kernel`。 装饰器：`gluon.jit`。 参数：`ptr`。 关键调用包括 `ttgl.PaddedSharedLayout.with_identity_for`、`ttgl.BlockedLayout`、`ttgl.amd.gfx1250.tdm.make_tensor_descriptor`、`ttgl.allocate_shared_memory`、`gfx1250_mbarrier.init`、`ttgl.amd.gfx1250.tdm.async_load` 等另外 2 项。 该作用域涉及张量/描述符元数据、布局变换推理。

#### Lines 4015-4025

```python
    SHARED_LAYOUT: ttgl.constexpr = ttgl.PaddedSharedLayout.with_identity_for([[32, 4]], [16, 64], [1, 0])
    BLOCKED_LAYOUT: ttgl.constexpr = ttgl.BlockedLayout([1, 8], [4, 8], [4, 1], [1, 0])

    desc = ttgl.amd.gfx1250.tdm.make_tensor_descriptor(base=ptr, shape=(32, 128), strides=(128, 1),
                                                       block_shape=(16, 64), layout=SHARED_LAYOUT)

    bar = ttgl.allocate_shared_memory(ttgl.int64, [1], gfx1250_mbarrier.MBarrierLayout())
    buffer = ttgl.allocate_shared_memory(desc.dtype, shape=desc.block_shape, layout=desc.layout)
    gfx1250_mbarrier.init(bar, count=1)
    ttgl.amd.gfx1250.tdm.async_load(desc, offsets=[0, 2], dest=buffer, mbarrier=bar)
    buffer.load(layout=BLOCKED_LAYOUT)
```
- **EN:** Prepares or updates state through `SHARED_LAYOUT`, `BLOCKED_LAYOUT`, `desc`, `bar`, `buffer`. Invokes `ttgl.PaddedSharedLayout.with_identity_for`, `ttgl.BlockedLayout`, `ttgl.amd.gfx1250.tdm.make_tensor_descriptor`, `ttgl.allocate_shared_memory`, `gfx1250_mbarrier.MBarrierLayout`, `gfx1250_mbarrier.init`, and 2 more to execute the test logic. Relevant themes: tensor/descriptor metadata, layout transformation reasoning.
- **CN:** 通过 `SHARED_LAYOUT`、`BLOCKED_LAYOUT`、`desc`、`bar`、`buffer` 准备或更新状态。 调用 `ttgl.PaddedSharedLayout.with_identity_for`、`ttgl.BlockedLayout`、`ttgl.amd.gfx1250.tdm.make_tensor_descriptor`、`ttgl.allocate_shared_memory`、`gfx1250_mbarrier.MBarrierLayout`、`gfx1250_mbarrier.init` 等另外 2 项 执行测试逻辑。 相关主题：张量/描述符元数据、布局变换推理。

### Lines 4026-4029

```python


@gluon.jit
def amd_cluster_barrier_arrive_kernel():
```
- **EN:** Defines the helper function `amd_cluster_barrier_arrive_kernel`. Decorators: `gluon.jit`. Key calls include `gfx1250_cluster.arrive`.
- **CN:** 定义辅助函数 `amd_cluster_barrier_arrive_kernel`。 装饰器：`gluon.jit`。 关键调用包括 `gfx1250_cluster.arrive`。

#### Lines 4030-4030

```python
    gfx1250_cluster.arrive()
```
- **EN:** Invokes `gfx1250_cluster.arrive` to execute the test logic.
- **CN:** 调用 `gfx1250_cluster.arrive` 执行测试逻辑。

### Lines 4031-4034

```python


@pytest.mark.parametrize("target", [HIP_TARGET_GFX1250])
def test_amd_cluster_barrier_arrive(target):
```
- **EN:** Defines the test function `test_amd_cluster_barrier_arrive`. Decorators: `pytest.mark.parametrize('target', [HIP_TARGET_GFX1250])`. Parameters: `target`. Key calls include `pytest.mark.parametrize`, `run_parser`, `expecttest.assert_expected_inline`, `anonymize_ir`, `make_args`, `mod.str_nodebug`. This scope touches pytest parametrization, debugging and inspection paths.
- **CN:** 定义测试函数 `test_amd_cluster_barrier_arrive`。 装饰器：`pytest.mark.parametrize('target', [HIP_TARGET_GFX1250])`。 参数：`target`。 关键调用包括 `pytest.mark.parametrize`、`run_parser`、`expecttest.assert_expected_inline`、`anonymize_ir`、`make_args`、`mod.str_nodebug`。 该作用域涉及pytest 参数化、调试与检查路径。

#### Lines 4035-4044

```python
    mod = run_parser(amd_cluster_barrier_arrive_kernel, *make_args(num_ctas=2), target=target)
    expecttest.assert_expected_inline(
        anonymize_ir(mod.str_nodebug()), """\
module attributes {"ttg.num-ctas" = 2 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "...", "ttg.threads-per-warp" = 32 : i32} {
  tt.func public @amd_cluster_barrier_arrive_kernel() attributes {noinline = false} {
    amdg.cluster_barrier_arrive
    tt.return
  }
}
""")
```
- **EN:** Prepares or updates state through `mod`. Invokes `run_parser`, `make_args`, `expecttest.assert_expected_inline`, `anonymize_ir`, `mod.str_nodebug` to execute the test logic. Relevant themes: debugging and inspection paths.
- **CN:** 通过 `mod` 准备或更新状态。 调用 `run_parser`、`make_args`、`expecttest.assert_expected_inline`、`anonymize_ir`、`mod.str_nodebug` 执行测试逻辑。 相关主题：调试与检查路径。

### Lines 4045-4048

```python


@gluon.jit
def amd_cluster_barrier_wait_kernel():
```
- **EN:** Defines the helper function `amd_cluster_barrier_wait_kernel`. Decorators: `gluon.jit`. Key calls include `gfx1250_cluster.wait`.
- **CN:** 定义辅助函数 `amd_cluster_barrier_wait_kernel`。 装饰器：`gluon.jit`。 关键调用包括 `gfx1250_cluster.wait`。

#### Lines 4049-4049

```python
    gfx1250_cluster.wait()
```
- **EN:** Invokes `gfx1250_cluster.wait` to execute the test logic.
- **CN:** 调用 `gfx1250_cluster.wait` 执行测试逻辑。

### Lines 4050-4053

```python


@pytest.mark.parametrize("target", [HIP_TARGET_GFX1250])
def test_amd_cluster_barrier_wait(target):
```
- **EN:** Defines the test function `test_amd_cluster_barrier_wait`. Decorators: `pytest.mark.parametrize('target', [HIP_TARGET_GFX1250])`. Parameters: `target`. Key calls include `pytest.mark.parametrize`, `run_parser`, `expecttest.assert_expected_inline`, `anonymize_ir`, `make_args`, `mod.str_nodebug`. This scope touches pytest parametrization, debugging and inspection paths.
- **CN:** 定义测试函数 `test_amd_cluster_barrier_wait`。 装饰器：`pytest.mark.parametrize('target', [HIP_TARGET_GFX1250])`。 参数：`target`。 关键调用包括 `pytest.mark.parametrize`、`run_parser`、`expecttest.assert_expected_inline`、`anonymize_ir`、`make_args`、`mod.str_nodebug`。 该作用域涉及pytest 参数化、调试与检查路径。

#### Lines 4054-4063

```python
    mod = run_parser(amd_cluster_barrier_wait_kernel, *make_args(num_ctas=2), target=target)
    expecttest.assert_expected_inline(
        anonymize_ir(mod.str_nodebug()), """\
module attributes {"ttg.num-ctas" = 2 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "...", "ttg.threads-per-warp" = 32 : i32} {
  tt.func public @amd_cluster_barrier_wait_kernel() attributes {noinline = false} {
    amdg.cluster_barrier_wait
    tt.return
  }
}
""")
```
- **EN:** Prepares or updates state through `mod`. Invokes `run_parser`, `make_args`, `expecttest.assert_expected_inline`, `anonymize_ir`, `mod.str_nodebug` to execute the test logic. Relevant themes: debugging and inspection paths.
- **CN:** 通过 `mod` 准备或更新状态。 调用 `run_parser`、`make_args`、`expecttest.assert_expected_inline`、`anonymize_ir`、`mod.str_nodebug` 执行测试逻辑。 相关主题：调试与检查路径。

### Lines 4064-4068

```python


@pytest.mark.parametrize("target", [HIP_TARGET_GFX1250])
def test_amd_tdm_load_mbarrier(target):
```
- **EN:** Defines the test function `test_amd_tdm_load_mbarrier`. Decorators: `pytest.mark.parametrize('target', [HIP_TARGET_GFX1250])`. Parameters: `target`. Key calls include `pytest.mark.parametrize`, `MockTensor`, `run_parser`, `expecttest.assert_expected_inline`, `anonymize_ir`, `make_args`, and 1 more. This scope touches pytest parametrization, tensor/descriptor metadata, debugging and inspection paths.
- **CN:** 定义测试函数 `test_amd_tdm_load_mbarrier`。 装饰器：`pytest.mark.parametrize('target', [HIP_TARGET_GFX1250])`。 参数：`target`。 关键调用包括 `pytest.mark.parametrize`、`MockTensor`、`run_parser`、`expecttest.assert_expected_inline`、`anonymize_ir`、`make_args` 等另外 1 项。 该作用域涉及pytest 参数化、张量/描述符元数据、调试与检查路径。

#### Lines 4069-4095

```python
    ptr = MockTensor(ttgl.float16)
    module = run_parser(amd_tdm_load_mbarrier_kernel, *make_args(ptr), target)
    expecttest.assert_expected_inline(
        anonymize_ir(module.str_nodebug()), """\
#blocked = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [4, 8], warpsPerCTA = [4, 1], order = [1, 0]}>
#shared = #ttg.padded_shared<[32:+4] {order = [1, 0], shape = [16, 64]}>
#shared1 = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0]}>
#smem = #ttg.shared_memory
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "...", "ttg.threads-per-warp" = 32 : i32} {
  tt.func public @amd_tdm_load_mbarrier_kernel(%arg0: !tt.ptr<f16> {tt.divisibility = 16 : i32}) attributes {noinline = false} {
    %c32_i32 = arith.constant 32 : i32
    %c128_i32 = arith.constant 128 : i32
    %c128_i64 = arith.constant 128 : i64
    %c1_i64 = arith.constant 1 : i64
    %0 = tt.make_tensor_descriptor %arg0, [%c32_i32, %c128_i32], [%c128_i64, %c1_i64] : <f16>, <16x64xf16, #shared>
    %1 = ttg.local_alloc : () -> !ttg.memdesc<1xi64, #shared1, #smem, mutable>
    %2 = ttg.local_alloc : () -> !ttg.memdesc<16x64xf16, #shared, #smem, mutable>
    amdg.init_barrier %1, 1 : !ttg.memdesc<1xi64, #shared1, #smem, mutable>
    %c0_i32 = arith.constant 0 : i32
    %c2_i32 = arith.constant 2 : i32
    %c1_i32 = arith.constant 1 : i32
    %3 = amdg.async_tdm_copy_global_to_local %0[%c0_i32, %c2_i32] into %2, pred = %c1_i32, barrier = %1 : !tt.tensordesc<16x64xf16, #shared>, !ttg.memdesc<1xi64, #shared1, #smem, mutable> -> !ttg.memdesc<16x64xf16, #shared, #smem, mutable>
    %4 = ttg.local_load %2 : !ttg.memdesc<16x64xf16, #shared, #smem, mutable> -> tensor<16x64xf16, #blocked>
    tt.return
  }
}
""")
```
- **EN:** Prepares or updates state through `ptr`, `module`. Invokes `MockTensor`, `run_parser`, `make_args`, `expecttest.assert_expected_inline`, `anonymize_ir`, `module.str_nodebug` to execute the test logic. Relevant themes: tensor/descriptor metadata, debugging and inspection paths.
- **CN:** 通过 `ptr`、`module` 准备或更新状态。 调用 `MockTensor`、`run_parser`、`make_args`、`expecttest.assert_expected_inline`、`anonymize_ir`、`module.str_nodebug` 执行测试逻辑。 相关主题：张量/描述符元数据、调试与检查路径。

### Lines 4096-4101

```python


@pytest.mark.parametrize("target", [BLACKWELL_TARGET, HOPPER_TARGET])
def test_nv_tma_descriptor_load_kernel(target):

    @gluon.jit
```
- **EN:** Defines the test function `test_nv_tma_descriptor_load_kernel`. Decorators: `pytest.mark.parametrize('target', [BLACKWELL_TARGET, HOPPER_TARGET])`. Parameters: `target`. Nested definitions in this scope: `nv_tma_descriptor_load_kernel`. Key calls include `pytest.mark.parametrize`, `MockTensor`, `run_parser`, `expecttest.assert_expected_inline`, `ttgl.NVMMASharedLayout`, `tma.make_tensor_descriptor`, and 8 more. This scope touches pytest parametrization, tensor/descriptor metadata, debugging and inspection paths, layout transformation reasoning.
- **CN:** 定义测试函数 `test_nv_tma_descriptor_load_kernel`。 装饰器：`pytest.mark.parametrize('target', [BLACKWELL_TARGET, HOPPER_TARGET])`。 参数：`target`。 该作用域中的嵌套定义：`nv_tma_descriptor_load_kernel`。 关键调用包括 `pytest.mark.parametrize`、`MockTensor`、`run_parser`、`expecttest.assert_expected_inline`、`ttgl.NVMMASharedLayout`、`tma.make_tensor_descriptor` 等另外 8 项。 该作用域涉及pytest 参数化、张量/描述符元数据、调试与检查路径、布局变换推理。

#### Lines 4101-4102

```python
    @gluon.jit
    def nv_tma_descriptor_load_kernel(input_ptr):
```
- **EN:** Defines the helper function `nv_tma_descriptor_load_kernel`. Decorators: `gluon.jit`. Parameters: `input_ptr`. Key calls include `ttgl.NVMMASharedLayout`, `tma.make_tensor_descriptor`, `ttgl.allocate_shared_memory`, `mbarrier.init`, `mbarrier.expect`, `tma.async_load`, and 1 more. This scope touches tensor/descriptor metadata, layout transformation reasoning.
- **CN:** 定义辅助函数 `nv_tma_descriptor_load_kernel`。 装饰器：`gluon.jit`。 参数：`input_ptr`。 关键调用包括 `ttgl.NVMMASharedLayout`、`tma.make_tensor_descriptor`、`ttgl.allocate_shared_memory`、`mbarrier.init`、`mbarrier.expect`、`tma.async_load` 等另外 1 项。 该作用域涉及张量/描述符元数据、布局变换推理。

##### Lines 4103-4116

```python
        XBLOCK: ttgl.constexpr = 128
        smem_layout: ttgl.constexpr = ttgl.NVMMASharedLayout(swizzle_byte_width=128, element_bitwidth=32, rank=2)
        input_desc = tma.make_tensor_descriptor(
            input_ptr,
            shape=[XBLOCK, XBLOCK],
            strides=[XBLOCK, 1],
            block_shape=[XBLOCK, XBLOCK],
            layout=smem_layout,
        )
        smem = ttgl.allocate_shared_memory(ttgl.float32, [XBLOCK, XBLOCK], smem_layout)
        bar = ttgl.allocate_shared_memory(ttgl.int64, [1], mbarrier.MBarrierLayout())
        mbarrier.init(bar, count=1)
        mbarrier.expect(bar, XBLOCK * XBLOCK * ttgl.float32.primitive_bitwidth // 8)
        tma.async_load(input_desc, [0, 0], bar, smem)
```
- **EN:** Prepares or updates state through `XBLOCK`, `smem_layout`, `input_desc`, `smem`, `bar`. Invokes `ttgl.NVMMASharedLayout`, `tma.make_tensor_descriptor`, `ttgl.allocate_shared_memory`, `mbarrier.MBarrierLayout`, `mbarrier.init`, `mbarrier.expect`, and 1 more to execute the test logic. Relevant themes: tensor/descriptor metadata, layout transformation reasoning.
- **CN:** 通过 `XBLOCK`、`smem_layout`、`input_desc`、`smem`、`bar` 准备或更新状态。 调用 `ttgl.NVMMASharedLayout`、`tma.make_tensor_descriptor`、`ttgl.allocate_shared_memory`、`mbarrier.MBarrierLayout`、`mbarrier.init`、`mbarrier.expect` 等另外 1 项 执行测试逻辑。 相关主题：张量/描述符元数据、布局变换推理。

#### Lines 4117-4144

```python

    ptr = MockTensor(ttgl.float32)
    module = run_parser(nv_tma_descriptor_load_kernel, *make_args(ptr), target)
    expecttest.assert_expected_inline(
        anonymize_ir(module.str_nodebug()), """\
#shared = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 32}>
#shared1 = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0]}>
#smem = #ttg.shared_memory
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "...", "ttg.threads-per-warp" = 32 : i32} {
  tt.func public @nv_tma_descriptor_load_kernel(%arg0: !tt.ptr<f32> {tt.divisibility = 16 : i32}) attributes {noinline = false} {
    %c128_i32 = arith.constant 128 : i32
    %c128_i32_0 = arith.constant 128 : i32
    %c128_i64 = arith.constant 128 : i64
    %c1_i64 = arith.constant 1 : i64
    %0 = tt.make_tensor_descriptor %arg0, [%c128_i32, %c128_i32_0], [%c128_i64, %c1_i64] : <f32>, <128x128xf32, #shared>
    %1 = ttg.local_alloc : () -> !ttg.memdesc<128x128xf32, #shared, #smem, mutable>
    %2 = ttg.local_alloc : () -> !ttg.memdesc<1xi64, #shared1, #smem, mutable>
    ttng.init_barrier %2, 1 : !ttg.memdesc<1xi64, #shared1, #smem, mutable>
    %true = arith.constant true
    ttng.barrier_expect %2, 65536, %true : !ttg.memdesc<1xi64, #shared1, #smem, mutable>
    %c0_i32 = arith.constant 0 : i32
    %c0_i32_1 = arith.constant 0 : i32
    %true_2 = arith.constant true
    ttng.async_tma_copy_global_to_local %0[%c0_i32, %c0_i32_1] %1, %2, %true_2 : !tt.tensordesc<128x128xf32, #shared>, !ttg.memdesc<1xi64, #shared1, #smem, mutable> -> !ttg.memdesc<128x128xf32, #shared, #smem, mutable>
    tt.return
  }
}
""")
```
- **EN:** Prepares or updates state through `ptr`, `module`. Invokes `MockTensor`, `run_parser`, `make_args`, `expecttest.assert_expected_inline`, `anonymize_ir`, `module.str_nodebug` to execute the test logic. Relevant themes: tensor/descriptor metadata, debugging and inspection paths.
- **CN:** 通过 `ptr`、`module` 准备或更新状态。 调用 `MockTensor`、`run_parser`、`make_args`、`expecttest.assert_expected_inline`、`anonymize_ir`、`module.str_nodebug` 执行测试逻辑。 相关主题：张量/描述符元数据、调试与检查路径。

### Lines 4145-4150

```python


@pytest.mark.parametrize("target", [BLACKWELL_TARGET, HOPPER_TARGET])
def test_nv_tma_descriptor_store_kernel(target):

    @gluon.jit
```
- **EN:** Defines the test function `test_nv_tma_descriptor_store_kernel`. Decorators: `pytest.mark.parametrize('target', [BLACKWELL_TARGET, HOPPER_TARGET])`. Parameters: `target`. Nested definitions in this scope: `nv_tma_descriptor_store_kernel`. Key calls include `pytest.mark.parametrize`, `MockTensor`, `run_parser`, `expecttest.assert_expected_inline`, `ttgl.NVMMASharedLayout`, `tma.make_tensor_descriptor`, and 6 more. This scope touches pytest parametrization, tensor/descriptor metadata, debugging and inspection paths, layout transformation reasoning.
- **CN:** 定义测试函数 `test_nv_tma_descriptor_store_kernel`。 装饰器：`pytest.mark.parametrize('target', [BLACKWELL_TARGET, HOPPER_TARGET])`。 参数：`target`。 该作用域中的嵌套定义：`nv_tma_descriptor_store_kernel`。 关键调用包括 `pytest.mark.parametrize`、`MockTensor`、`run_parser`、`expecttest.assert_expected_inline`、`ttgl.NVMMASharedLayout`、`tma.make_tensor_descriptor` 等另外 6 项。 该作用域涉及pytest 参数化、张量/描述符元数据、调试与检查路径、布局变换推理。

#### Lines 4150-4151

```python
    @gluon.jit
    def nv_tma_descriptor_store_kernel(input_ptr):
```
- **EN:** Defines the helper function `nv_tma_descriptor_store_kernel`. Decorators: `gluon.jit`. Parameters: `input_ptr`. Key calls include `ttgl.NVMMASharedLayout`, `tma.make_tensor_descriptor`, `ttgl.allocate_shared_memory`, `tma.async_copy_shared_to_global`, `tma.store_wait`. This scope touches tensor/descriptor metadata, layout transformation reasoning.
- **CN:** 定义辅助函数 `nv_tma_descriptor_store_kernel`。 装饰器：`gluon.jit`。 参数：`input_ptr`。 关键调用包括 `ttgl.NVMMASharedLayout`、`tma.make_tensor_descriptor`、`ttgl.allocate_shared_memory`、`tma.async_copy_shared_to_global`、`tma.store_wait`。 该作用域涉及张量/描述符元数据、布局变换推理。

##### Lines 4152-4163

```python
        XBLOCK: ttgl.constexpr = 128
        smem_layout: ttgl.constexpr = ttgl.NVMMASharedLayout(swizzle_byte_width=128, element_bitwidth=32, rank=2)
        input_desc = tma.make_tensor_descriptor(
            input_ptr,
            shape=[XBLOCK, XBLOCK],
            strides=[XBLOCK, 1],
            block_shape=[XBLOCK, XBLOCK],
            layout=smem_layout,
        )
        smem = ttgl.allocate_shared_memory(ttgl.float32, [XBLOCK, XBLOCK], smem_layout)
        tma.async_copy_shared_to_global(input_desc, [0, 0], smem)
        tma.store_wait(0)
```
- **EN:** Prepares or updates state through `XBLOCK`, `smem_layout`, `input_desc`, `smem`. Invokes `ttgl.NVMMASharedLayout`, `tma.make_tensor_descriptor`, `ttgl.allocate_shared_memory`, `tma.async_copy_shared_to_global`, `tma.store_wait` to execute the test logic. Relevant themes: tensor/descriptor metadata, layout transformation reasoning.
- **CN:** 通过 `XBLOCK`、`smem_layout`、`input_desc`、`smem` 准备或更新状态。 调用 `ttgl.NVMMASharedLayout`、`tma.make_tensor_descriptor`、`ttgl.allocate_shared_memory`、`tma.async_copy_shared_to_global`、`tma.store_wait` 执行测试逻辑。 相关主题：张量/描述符元数据、布局变换推理。

#### Lines 4164-4186

```python

    ptr = MockTensor(ttgl.float32)
    module = run_parser(nv_tma_descriptor_store_kernel, *make_args(ptr), target)
    expecttest.assert_expected_inline(
        anonymize_ir(module.str_nodebug()), """\
#shared = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 32}>
#smem = #ttg.shared_memory
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "...", "ttg.threads-per-warp" = 32 : i32} {
  tt.func public @nv_tma_descriptor_store_kernel(%arg0: !tt.ptr<f32> {tt.divisibility = 16 : i32}) attributes {noinline = false} {
    %c128_i32 = arith.constant 128 : i32
    %c128_i32_0 = arith.constant 128 : i32
    %c128_i64 = arith.constant 128 : i64
    %c1_i64 = arith.constant 1 : i64
    %0 = tt.make_tensor_descriptor %arg0, [%c128_i32, %c128_i32_0], [%c128_i64, %c1_i64] : <f32>, <128x128xf32, #shared>
    %1 = ttg.local_alloc : () -> !ttg.memdesc<128x128xf32, #shared, #smem, mutable>
    %c0_i32 = arith.constant 0 : i32
    %c0_i32_1 = arith.constant 0 : i32
    ttng.async_tma_copy_local_to_global %0[%c0_i32, %c0_i32_1] %1 : !tt.tensordesc<128x128xf32, #shared>, !ttg.memdesc<128x128xf32, #shared, #smem, mutable>
    ttng.async_tma_store_wait {pendings = 0 : i32}
    tt.return
  }
}
""")
```
- **EN:** Prepares or updates state through `ptr`, `module`. Invokes `MockTensor`, `run_parser`, `make_args`, `expecttest.assert_expected_inline`, `anonymize_ir`, `module.str_nodebug` to execute the test logic. Relevant themes: tensor/descriptor metadata, debugging and inspection paths.
- **CN:** 通过 `ptr`、`module` 准备或更新状态。 调用 `MockTensor`、`run_parser`、`make_args`、`expecttest.assert_expected_inline`、`anonymize_ir`、`module.str_nodebug` 执行测试逻辑。 相关主题：张量/描述符元数据、调试与检查路径。

### Lines 4187-4200

```python


@pytest.mark.parametrize("target", [BLACKWELL_TARGET, HOPPER_TARGET])
@pytest.mark.parametrize(("op_name", "kind"), [
    ("async_atomic_add", "add"),
    ("async_atomic_min", "min"),
    ("async_atomic_max", "max"),
    ("async_atomic_and", "and"),
    ("async_atomic_or", "or"),
    ("async_atomic_xor", "xor"),
])
def test_nv_tma_descriptor_reduce_kernel(target, op_name, kind):

    @gluon.jit
```
- **EN:** Defines the test function `test_nv_tma_descriptor_reduce_kernel`. Decorators: `pytest.mark.parametrize('target', [BLACKWELL_TARGET, HOPPER_TARGET])`, `pytest.mark.parametrize(('op_name', 'kind'), [('async_atomic_add', 'add'), ('async_atomic_min', 'min'), ('async_atomic_max', 'max'), ('async_atomic_and', 'and'), ('async_atomic_or', 'or'), ('async_atomic_xor', 'xor')])`. Parameters: `target`, `op_name`, `kind`. Nested definitions in this scope: `nv_tma_descriptor_reduce_kernel`. Key calls include `pytest.mark.parametrize`, `MockTensor`, `run_parser`, `anonymize_ir`, `ttgl.NVMMASharedLayout`, `tma.make_tensor_descriptor`, and 10 more. This scope touches pytest parametrization, tensor/descriptor metadata, debugging and inspection paths, layout transformation reasoning.
- **CN:** 定义测试函数 `test_nv_tma_descriptor_reduce_kernel`。 装饰器：`pytest.mark.parametrize('target', [BLACKWELL_TARGET, HOPPER_TARGET])`、`pytest.mark.parametrize(('op_name', 'kind'), [('async_atomic_add', 'add'), ('async_atomic_min', 'min'), ('async_atomic_max', 'max'), ('async_atomic_and', 'and'), ('async_atomic_or', 'or'), ('async_atomic_xor', 'xor')])`。 参数：`target`、`op_name`、`kind`。 该作用域中的嵌套定义：`nv_tma_descriptor_reduce_kernel`。 关键调用包括 `pytest.mark.parametrize`、`MockTensor`、`run_parser`、`anonymize_ir`、`ttgl.NVMMASharedLayout`、`tma.make_tensor_descriptor` 等另外 10 项。 该作用域涉及pytest 参数化、张量/描述符元数据、调试与检查路径、布局变换推理。

#### Lines 4200-4201

```python
    @gluon.jit
    def nv_tma_descriptor_reduce_kernel(input_ptr, OP_NAME: ttgl.constexpr):
```
- **EN:** Defines the helper function `nv_tma_descriptor_reduce_kernel`. Decorators: `gluon.jit`. Parameters: `input_ptr`, `OP_NAME`. Key calls include `ttgl.NVMMASharedLayout`, `tma.make_tensor_descriptor`, `ttgl.allocate_shared_memory`, `tma.store_wait`, `tma.async_atomic_add`, `tma.async_atomic_min`, and 4 more. This scope touches tensor/descriptor metadata, layout transformation reasoning.
- **CN:** 定义辅助函数 `nv_tma_descriptor_reduce_kernel`。 装饰器：`gluon.jit`。 参数：`input_ptr`、`OP_NAME`。 关键调用包括 `ttgl.NVMMASharedLayout`、`tma.make_tensor_descriptor`、`ttgl.allocate_shared_memory`、`tma.store_wait`、`tma.async_atomic_add`、`tma.async_atomic_min` 等另外 4 项。 该作用域涉及张量/描述符元数据、布局变换推理。

##### Lines 4202-4211

```python
        XBLOCK: ttgl.constexpr = 128
        smem_layout: ttgl.constexpr = ttgl.NVMMASharedLayout(swizzle_byte_width=128, element_bitwidth=32, rank=2)
        input_desc = tma.make_tensor_descriptor(
            input_ptr,
            shape=[XBLOCK, XBLOCK],
            strides=[XBLOCK, 1],
            block_shape=[XBLOCK, XBLOCK],
            layout=smem_layout,
        )
        smem = ttgl.allocate_shared_memory(ttgl.int32, [XBLOCK, XBLOCK], smem_layout)
```
- **EN:** Prepares or updates state through `XBLOCK`, `smem_layout`, `input_desc`, `smem`. Invokes `ttgl.NVMMASharedLayout`, `tma.make_tensor_descriptor`, `ttgl.allocate_shared_memory` to execute the test logic. Relevant themes: tensor/descriptor metadata, layout transformation reasoning.
- **CN:** 通过 `XBLOCK`、`smem_layout`、`input_desc`、`smem` 准备或更新状态。 调用 `ttgl.NVMMASharedLayout`、`tma.make_tensor_descriptor`、`ttgl.allocate_shared_memory` 执行测试逻辑。 相关主题：张量/描述符元数据、布局变换推理。

##### Lines 4212-4223

```python
        if OP_NAME == "async_atomic_add":
            tma.async_atomic_add(input_desc, [0, 0], smem)
        elif OP_NAME == "async_atomic_min":
            tma.async_atomic_min(input_desc, [0, 0], smem)
        elif OP_NAME == "async_atomic_max":
            tma.async_atomic_max(input_desc, [0, 0], smem)
        elif OP_NAME == "async_atomic_and":
            tma.async_atomic_and(input_desc, [0, 0], smem)
        elif OP_NAME == "async_atomic_or":
            tma.async_atomic_or(input_desc, [0, 0], smem)
        elif OP_NAME == "async_atomic_xor":
            tma.async_atomic_xor(input_desc, [0, 0], smem)
```
- **EN:** Invokes `tma.async_atomic_add`, `tma.async_atomic_min`, `tma.async_atomic_max`, `tma.async_atomic_and`, `tma.async_atomic_or`, `tma.async_atomic_xor` to execute the test logic. Branches on runtime or test conditions.
- **CN:** 调用 `tma.async_atomic_add`、`tma.async_atomic_min`、`tma.async_atomic_max`、`tma.async_atomic_and`、`tma.async_atomic_or`、`tma.async_atomic_xor` 执行测试逻辑。 根据运行时或测试条件进行分支。

##### Lines 4224-4224

```python
        tma.store_wait(0)
```
- **EN:** Invokes `tma.store_wait` to execute the test logic.
- **CN:** 调用 `tma.store_wait` 执行测试逻辑。

#### Lines 4225-4228

```python

    ptr = MockTensor(ttgl.int32)
    module = run_parser(nv_tma_descriptor_reduce_kernel, *make_args(ptr, op_name), target)
    ttgir = anonymize_ir(module.str_nodebug())
```
- **EN:** Prepares or updates state through `ptr`, `module`, `ttgir`. Invokes `MockTensor`, `run_parser`, `make_args`, `anonymize_ir`, `module.str_nodebug` to execute the test logic. Relevant themes: tensor/descriptor metadata, debugging and inspection paths.
- **CN:** 通过 `ptr`、`module`、`ttgir` 准备或更新状态。 调用 `MockTensor`、`run_parser`、`make_args`、`anonymize_ir`、`module.str_nodebug` 执行测试逻辑。 相关主题：张量/描述符元数据、调试与检查路径。

#### Lines 4229-4230

```python
    assert f"ttng.async_tma_reduce {kind}," in ttgir
    assert "ttng.async_tma_copy_local_to_global" not in ttgir
```
- **EN:** Validates behavior with 2 assertion(s).
- **CN:** 通过 2 个断言验证行为。

### Lines 4231-4234

```python


@filecheck_test
def tmem_constexpr():
```
- **EN:** Defines the helper function `tmem_constexpr`. Decorators: `filecheck_test`. Key calls include `TensorMemoryLayout`, `anchor_noinline`. This scope touches layout transformation reasoning.
- **CN:** 定义辅助函数 `tmem_constexpr`。 装饰器：`filecheck_test`。 关键调用包括 `TensorMemoryLayout`、`anchor_noinline`。 该作用域涉及布局变换推理。

#### Lines 4235-4240

```python
    tmem_shape: ttgl.constexpr = (64, 64)
    bitwidth: ttgl.constexpr = 32
    tmem_layout: ttgl.constexpr = TensorMemoryLayout(tmem_shape, col_stride=32 // bitwidth)

    # CHECK-NOT: constexpr
    anchor_noinline(tmem_layout)
```
- **EN:** Prepares or updates state through `tmem_shape`, `bitwidth`, `tmem_layout`. Invokes `TensorMemoryLayout`, `anchor_noinline` to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `tmem_shape`、`bitwidth`、`tmem_layout` 准备或更新状态。 调用 `TensorMemoryLayout`、`anchor_noinline` 执行测试逻辑。 相关主题：布局变换推理。

### Lines 4241-4245

```python


def test_auto_layout_convert_store_val():

    @gluon.jit
```
- **EN:** Defines the test function `test_auto_layout_convert_store_val`. Nested definitions in this scope: `kernel`. Key calls include `MockTensor`, `run_parser`, `expecttest.assert_expected_inline`, `ttgl.BlockedLayout`, `ttgl.arange`, `ttgl.set_auto_layout`, and 6 more. This scope touches debugging and inspection paths, layout transformation reasoning.
- **CN:** 定义测试函数 `test_auto_layout_convert_store_val`。 该作用域中的嵌套定义：`kernel`。 关键调用包括 `MockTensor`、`run_parser`、`expecttest.assert_expected_inline`、`ttgl.BlockedLayout`、`ttgl.arange`、`ttgl.set_auto_layout` 等另外 6 项。 该作用域涉及调试与检查路径、布局变换推理。

#### Lines 4245-4247

```python
    @gluon.jit
    def kernel(out_ptr,  #
               XBLOCK: ttgl.constexpr, YBLOCK: ttgl.constexpr):
```
- **EN:** Defines the helper function `kernel`. Decorators: `gluon.jit`. Parameters: `out_ptr`, `XBLOCK`, `YBLOCK`. Key calls include `ttgl.BlockedLayout`, `ttgl.arange`, `ttgl.set_auto_layout`, `ttgl.full`, `ttgl.store`, `ttgl.AutoLayout`. This scope touches layout transformation reasoning.
- **CN:** 定义辅助函数 `kernel`。 装饰器：`gluon.jit`。 参数：`out_ptr`、`XBLOCK`、`YBLOCK`。 关键调用包括 `ttgl.BlockedLayout`、`ttgl.arange`、`ttgl.set_auto_layout`、`ttgl.full`、`ttgl.store`、`ttgl.AutoLayout`。 该作用域涉及布局变换推理。

##### Lines 4248-4255

```python
        blocked: ttgl.constexpr = ttgl.BlockedLayout([1, 4], [32, 1], [2, 2], [1, 0])
        indices_x = ttgl.arange(0, XBLOCK)
        indices_y = ttgl.arange(0, YBLOCK)
        out_offsets = indices_x[:, None] + indices_y[None, :]
        mask = (indices_x[:, None] < 100) & (indices_y[None, :] < 200)
        out_ptrs = ttgl.set_auto_layout(out_ptr + out_offsets, blocked)
        value = ttgl.full([XBLOCK, YBLOCK], 0, dtype=ttgl.float32, layout=ttgl.AutoLayout())
        ttgl.store(out_ptrs, value, mask=mask)
```
- **EN:** Prepares or updates state through `blocked`, `indices_x`, `indices_y`, `out_offsets`, `mask`, `out_ptrs`, `value`. Invokes `ttgl.BlockedLayout`, `ttgl.arange`, `ttgl.set_auto_layout`, `ttgl.full`, `ttgl.AutoLayout`, `ttgl.store` to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `blocked`、`indices_x`、`indices_y`、`out_offsets`、`mask`、`out_ptrs`、`value` 准备或更新状态。 调用 `ttgl.BlockedLayout`、`ttgl.arange`、`ttgl.set_auto_layout`、`ttgl.full`、`ttgl.AutoLayout`、`ttgl.store` 执行测试逻辑。 相关主题：布局变换推理。

#### Lines 4256-4295

```python

    XBLOCK = 128
    YBLOCK = 256
    output = MockTensor(ttgl.float32)
    module = run_parser(kernel, *make_args(output, XBLOCK, YBLOCK))
    expecttest.assert_expected_inline(
        anonymize_ir(module.str_nodebug()), """\
#blocked = #ttg.blocked<{sizePerThread = [1, 4], threadsPerWarp = [32, 1], warpsPerCTA = [2, 2], order = [1, 0]}>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "...", "ttg.threads-per-warp" = 32 : i32} {
  tt.func public @kernel(%arg0: !tt.ptr<f32> {tt.divisibility = 16 : i32}) attributes {noinline = false} {
    %0 = tt.make_range {end = 128 : i32, start = 0 : i32} : tensor<128xi32, #gluon.auto_encoding>
    %1 = tt.make_range {end = 256 : i32, start = 0 : i32} : tensor<256xi32, #gluon.auto_encoding>
    %2 = tt.expand_dims %0 {axis = 1 : i32} : tensor<128xi32, #gluon.auto_encoding> -> tensor<128x1xi32, #gluon.auto_encoding>
    %3 = tt.expand_dims %1 {axis = 0 : i32} : tensor<256xi32, #gluon.auto_encoding> -> tensor<1x256xi32, #gluon.auto_encoding>
    %4 = tt.broadcast %2 : tensor<128x1xi32, #gluon.auto_encoding> -> tensor<128x256xi32, #gluon.auto_encoding>
    %5 = tt.broadcast %3 : tensor<1x256xi32, #gluon.auto_encoding> -> tensor<128x256xi32, #gluon.auto_encoding>
    %6 = arith.addi %4, %5 : tensor<128x256xi32, #gluon.auto_encoding>
    %7 = tt.expand_dims %0 {axis = 1 : i32} : tensor<128xi32, #gluon.auto_encoding> -> tensor<128x1xi32, #gluon.auto_encoding>
    %c100_i32 = arith.constant 100 : i32
    %cst = arith.constant dense<100> : tensor<128x1xi32, #gluon.auto_encoding>
    %8 = arith.cmpi slt, %7, %cst : tensor<128x1xi32, #gluon.auto_encoding>
    %9 = tt.expand_dims %1 {axis = 0 : i32} : tensor<256xi32, #gluon.auto_encoding> -> tensor<1x256xi32, #gluon.auto_encoding>
    %c200_i32 = arith.constant 200 : i32
    %cst_0 = arith.constant dense<200> : tensor<1x256xi32, #gluon.auto_encoding>
    %10 = arith.cmpi slt, %9, %cst_0 : tensor<1x256xi32, #gluon.auto_encoding>
    %11 = tt.broadcast %8 : tensor<128x1xi1, #gluon.auto_encoding> -> tensor<128x256xi1, #gluon.auto_encoding>
    %12 = tt.broadcast %10 : tensor<1x256xi1, #gluon.auto_encoding> -> tensor<128x256xi1, #gluon.auto_encoding>
    %13 = arith.andi %11, %12 : tensor<128x256xi1, #gluon.auto_encoding>
    %14 = tt.splat %arg0 : !tt.ptr<f32> -> tensor<128x256x!tt.ptr<f32>, #gluon.auto_encoding>
    %15 = tt.addptr %14, %6 : tensor<128x256x!tt.ptr<f32>, #gluon.auto_encoding>, tensor<128x256xi32, #gluon.auto_encoding>
    %16 = gluon.set_auto_layout %15 : tensor<128x256x!tt.ptr<f32>, #gluon.auto_encoding> -> tensor<128x256x!tt.ptr<f32>, #blocked>
    %cst_1 = arith.constant 0.000000e+00 : f32
    %cst_2 = arith.constant dense<0.000000e+00> : tensor<128x256xf32, #gluon.auto_encoding>
    %17 = gluon.set_auto_layout %cst_2 : tensor<128x256xf32, #gluon.auto_encoding> -> tensor<128x256xf32, #blocked>
    %18 = gluon.set_auto_layout %13 : tensor<128x256xi1, #gluon.auto_encoding> -> tensor<128x256xi1, #blocked>
    tt.store %16, %17, %18 : tensor<128x256x!tt.ptr<f32>, #blocked>
    tt.return
  }
}
""")
```
- **EN:** Prepares or updates state through `XBLOCK`, `YBLOCK`, `output`, `module`. Invokes `MockTensor`, `run_parser`, `make_args`, `expecttest.assert_expected_inline`, `anonymize_ir`, `module.str_nodebug` to execute the test logic. Relevant themes: debugging and inspection paths, layout transformation reasoning.
- **CN:** 通过 `XBLOCK`、`YBLOCK`、`output`、`module` 准备或更新状态。 调用 `MockTensor`、`run_parser`、`make_args`、`expecttest.assert_expected_inline`、`anonymize_ir`、`module.str_nodebug` 执行测试逻辑。 相关主题：调试与检查路径、布局变换推理。

### Lines 4296-4300

```python


def test_auto_layout_convert_store_ptr():

    @gluon.jit
```
- **EN:** Defines the test function `test_auto_layout_convert_store_ptr`. Nested definitions in this scope: `kernel`. Key calls include `MockTensor`, `run_parser`, `expecttest.assert_expected_inline`, `ttgl.BlockedLayout`, `ttgl.arange`, `ttgl.full`, and 4 more. This scope touches debugging and inspection paths, layout transformation reasoning.
- **CN:** 定义测试函数 `test_auto_layout_convert_store_ptr`。 该作用域中的嵌套定义：`kernel`。 关键调用包括 `MockTensor`、`run_parser`、`expecttest.assert_expected_inline`、`ttgl.BlockedLayout`、`ttgl.arange`、`ttgl.full` 等另外 4 项。 该作用域涉及调试与检查路径、布局变换推理。

#### Lines 4300-4302

```python
    @gluon.jit
    def kernel(out_ptr,  #
               XBLOCK: ttgl.constexpr, YBLOCK: ttgl.constexpr):
```
- **EN:** Defines the helper function `kernel`. Decorators: `gluon.jit`. Parameters: `out_ptr`, `XBLOCK`, `YBLOCK`. Key calls include `ttgl.BlockedLayout`, `ttgl.arange`, `ttgl.full`, `ttgl.store`. This scope touches layout transformation reasoning.
- **CN:** 定义辅助函数 `kernel`。 装饰器：`gluon.jit`。 参数：`out_ptr`、`XBLOCK`、`YBLOCK`。 关键调用包括 `ttgl.BlockedLayout`、`ttgl.arange`、`ttgl.full`、`ttgl.store`。 该作用域涉及布局变换推理。

##### Lines 4303-4309

```python
        blocked: ttgl.constexpr = ttgl.BlockedLayout([1, 4], [32, 1], [2, 2], [1, 0])
        indices_x = ttgl.arange(0, XBLOCK)
        indices_y = ttgl.arange(0, YBLOCK)
        out_offsets = indices_x[:, None] + indices_y[None, :]
        mask = (indices_x[:, None] < 100) & (indices_y[None, :] < 200)
        value = ttgl.full([XBLOCK, YBLOCK], 0, dtype=ttgl.float32, layout=blocked)
        ttgl.store(out_ptr + out_offsets, value, mask=mask)
```
- **EN:** Prepares or updates state through `blocked`, `indices_x`, `indices_y`, `out_offsets`, `mask`, `value`. Invokes `ttgl.BlockedLayout`, `ttgl.arange`, `ttgl.full`, `ttgl.store` to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `blocked`、`indices_x`、`indices_y`、`out_offsets`、`mask`、`value` 准备或更新状态。 调用 `ttgl.BlockedLayout`、`ttgl.arange`、`ttgl.full`、`ttgl.store` 执行测试逻辑。 相关主题：布局变换推理。

#### Lines 4310-4348

```python

    XBLOCK = 128
    YBLOCK = 256
    output = MockTensor(ttgl.float32)
    module = run_parser(kernel, *make_args(output, XBLOCK, YBLOCK))
    expecttest.assert_expected_inline(
        anonymize_ir(module.str_nodebug()), """\
#blocked = #ttg.blocked<{sizePerThread = [1, 4], threadsPerWarp = [32, 1], warpsPerCTA = [2, 2], order = [1, 0]}>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "...", "ttg.threads-per-warp" = 32 : i32} {
  tt.func public @kernel(%arg0: !tt.ptr<f32> {tt.divisibility = 16 : i32}) attributes {noinline = false} {
    %0 = tt.make_range {end = 128 : i32, start = 0 : i32} : tensor<128xi32, #gluon.auto_encoding>
    %1 = tt.make_range {end = 256 : i32, start = 0 : i32} : tensor<256xi32, #gluon.auto_encoding>
    %2 = tt.expand_dims %0 {axis = 1 : i32} : tensor<128xi32, #gluon.auto_encoding> -> tensor<128x1xi32, #gluon.auto_encoding>
    %3 = tt.expand_dims %1 {axis = 0 : i32} : tensor<256xi32, #gluon.auto_encoding> -> tensor<1x256xi32, #gluon.auto_encoding>
    %4 = tt.broadcast %2 : tensor<128x1xi32, #gluon.auto_encoding> -> tensor<128x256xi32, #gluon.auto_encoding>
    %5 = tt.broadcast %3 : tensor<1x256xi32, #gluon.auto_encoding> -> tensor<128x256xi32, #gluon.auto_encoding>
    %6 = arith.addi %4, %5 : tensor<128x256xi32, #gluon.auto_encoding>
    %7 = tt.expand_dims %0 {axis = 1 : i32} : tensor<128xi32, #gluon.auto_encoding> -> tensor<128x1xi32, #gluon.auto_encoding>
    %c100_i32 = arith.constant 100 : i32
    %cst = arith.constant dense<100> : tensor<128x1xi32, #gluon.auto_encoding>
    %8 = arith.cmpi slt, %7, %cst : tensor<128x1xi32, #gluon.auto_encoding>
    %9 = tt.expand_dims %1 {axis = 0 : i32} : tensor<256xi32, #gluon.auto_encoding> -> tensor<1x256xi32, #gluon.auto_encoding>
    %c200_i32 = arith.constant 200 : i32
    %cst_0 = arith.constant dense<200> : tensor<1x256xi32, #gluon.auto_encoding>
    %10 = arith.cmpi slt, %9, %cst_0 : tensor<1x256xi32, #gluon.auto_encoding>
    %11 = tt.broadcast %8 : tensor<128x1xi1, #gluon.auto_encoding> -> tensor<128x256xi1, #gluon.auto_encoding>
    %12 = tt.broadcast %10 : tensor<1x256xi1, #gluon.auto_encoding> -> tensor<128x256xi1, #gluon.auto_encoding>
    %13 = arith.andi %11, %12 : tensor<128x256xi1, #gluon.auto_encoding>
    %cst_1 = arith.constant 0.000000e+00 : f32
    %cst_2 = arith.constant dense<0.000000e+00> : tensor<128x256xf32, #blocked>
    %14 = tt.splat %arg0 : !tt.ptr<f32> -> tensor<128x256x!tt.ptr<f32>, #gluon.auto_encoding>
    %15 = tt.addptr %14, %6 : tensor<128x256x!tt.ptr<f32>, #gluon.auto_encoding>, tensor<128x256xi32, #gluon.auto_encoding>
    %16 = gluon.set_auto_layout %15 : tensor<128x256x!tt.ptr<f32>, #gluon.auto_encoding> -> tensor<128x256x!tt.ptr<f32>, #blocked>
    %17 = gluon.set_auto_layout %13 : tensor<128x256xi1, #gluon.auto_encoding> -> tensor<128x256xi1, #blocked>
    tt.store %16, %cst_2, %17 : tensor<128x256x!tt.ptr<f32>, #blocked>
    tt.return
  }
}
""")
```
- **EN:** Prepares or updates state through `XBLOCK`, `YBLOCK`, `output`, `module`. Invokes `MockTensor`, `run_parser`, `make_args`, `expecttest.assert_expected_inline`, `anonymize_ir`, `module.str_nodebug` to execute the test logic. Relevant themes: debugging and inspection paths, layout transformation reasoning.
- **CN:** 通过 `XBLOCK`、`YBLOCK`、`output`、`module` 准备或更新状态。 调用 `MockTensor`、`run_parser`、`make_args`、`expecttest.assert_expected_inline`、`anonymize_ir`、`module.str_nodebug` 执行测试逻辑。 相关主题：调试与检查路径、布局变换推理。

## Key Concepts / 关键概念

- **EN:** Top-level scopes such as `anonymize_ir`, `make_args`, `convert_layout_kernel`, `test_convert_layout`, `simple_ops_kernel`, `test_simple_ops`, `test_histogram_frontend`, `test_convert_layout_assert_trivial`
  **CN:** 顶层作用域，例如 `anonymize_ir`、`make_args`、`convert_layout_kernel`、`test_convert_layout`、`simple_ops_kernel`、`test_simple_ops`、`test_histogram_frontend`、`test_convert_layout_assert_trivial`
- **EN:** pytest parametrization
  **CN:** pytest 参数化
- **EN:** Triton compilation or JIT kernels
  **CN:** Triton 编译或 JIT kernel
- **EN:** Triton language operations
  **CN:** Triton language 操作
- **EN:** cache management behavior
  **CN:** 缓存管理行为
- **EN:** tensor/descriptor metadata
  **CN:** 张量/描述符元数据
- **EN:** plugin or compiler extension points
  **CN:** 插件或编译器扩展点
- **EN:** debugging and inspection paths
  **CN:** 调试与检查路径

## Dependencies / 依赖关系

- **EN:** External or absolute imports include `expecttest`, `pytest`, `re`, `triton.backends.compiler`, `triton.experimental`, `triton.experimental.gluon`, `triton.experimental.gluon.language.nvidia`, `triton.experimental.gluon.language.nvidia.hopper`, `triton.experimental.gluon.language.nvidia.blackwell`, `triton.experimental.gluon.nvidia.hopper`, `triton.experimental.gluon.language.amd`, `triton.experimental.gluon.language.amd.cdna4`, and 6 more.
  **CN:** 外部或绝对导入包括 `expecttest`、`pytest`、`re`、`triton.backends.compiler`、`triton.experimental`、`triton.experimental.gluon`、`triton.experimental.gluon.language.nvidia`、`triton.experimental.gluon.language.nvidia.hopper`、`triton.experimental.gluon.language.nvidia.blackwell`、`triton.experimental.gluon.nvidia.hopper`、`triton.experimental.gluon.language.amd`、`triton.experimental.gluon.language.amd.cdna4` 等另外 6 项。
- **EN:** Execution centers on top-level definitions such as `anonymize_ir`, `make_args`, `convert_layout_kernel`, `test_convert_layout`, `simple_ops_kernel`, `test_simple_ops`, `test_histogram_frontend`, `test_convert_layout_assert_trivial`, `test_convert_layout_not_trivial`, `shared_memory_kernel`.
  **CN:** 执行逻辑主要围绕顶层定义展开，例如 `anonymize_ir`、`make_args`、`convert_layout_kernel`、`test_convert_layout`、`simple_ops_kernel`、`test_simple_ops`、`test_histogram_frontend`、`test_convert_layout_assert_trivial`、`test_convert_layout_not_trivial`、`shared_memory_kernel`。
- **EN:** Runtime behavior also depends on pytest collection, Python execution semantics, and the imported Triton/PyTorch utilities visible above.
  **CN:** 运行时行为还依赖 pytest 的收集机制、Python 执行语义，以及上文可见的 Triton/PyTorch 工具。
