# test_consan.py — Code Analysis / 代码分析

## Source / 来源
- **Path / 路径:** `python/test/gluon/test_consan.py`
- **EN:** Pytest module covering consan behavior in Triton's Python tests. It contains 71 top-level definition(s) and 10 imported module reference(s).
- **CN:** 这是一个 pytest 模块，用于覆盖 Triton Python 测试中的 consan 行为。 该文件包含 71 个顶层定义，以及 10 个导入模块引用。

## Line-by-Line Analysis / 逐行分析

### Lines 1-12

```python
import os
import tempfile
import torch
import pytest
from triton import knobs
from triton.experimental import gluon
from triton.experimental.gluon import language as ttgl
from triton.experimental.gluon.language.nvidia import blackwell
from triton.experimental.gluon.language.nvidia import hopper
from triton.experimental.gluon.language.nvidia import ampere
from triton.experimental.gluon.language.nvidia.blackwell import allocate_tensor_memory, clc, mbarrier, tma
from triton._internal_testing import is_cuda, run_in_process
```
- **EN:** Imports the modules used in this scope: `os`, `tempfile`, `torch`, `pytest`, `triton`, `triton.experimental`, `triton.experimental.gluon`, `triton.experimental.gluon.language.nvidia`, `triton.experimental.gluon.language.nvidia.blackwell`, `triton._internal_testing`.
- **CN:** 导入此作用域使用的模块：`os`、`tempfile`、`torch`、`pytest`、`triton`、`triton.experimental`、`triton.experimental.gluon`、`triton.experimental.gluon.language.nvidia`、`triton.experimental.gluon.language.nvidia.blackwell`、`triton._internal_testing`。

### Lines 13-18

```python


@pytest.fixture
def run_wrapper():
    # Use DISABLE_SUBPROCESS to run the tests in the main process
    # (useful for debugging but assert in any test will make all the tests fail)
```
- **EN:** Defines the helper function `run_wrapper`. Decorators: `pytest.fixture`. Key calls include `os.environ.get`. This scope touches shared fixtures, subprocess-driven validation.
- **CN:** 定义辅助函数 `run_wrapper`。 装饰器：`pytest.fixture`。 关键调用包括 `os.environ.get`。 该作用域涉及共享 fixture、基于子进程的验证。

#### Lines 19-19

```python
    return not os.environ.get("DISABLE_SUBPROCESS")
```
- **EN:** Invokes `os.environ.get` to execute the test logic. Relevant themes: subprocess-driven validation.
- **CN:** 调用 `os.environ.get` 执行测试逻辑。 相关主题：基于子进程的验证。

### Lines 20-23

```python


@pytest.fixture(params=[1, 2, 4], ids=lambda num_ctas: f"{num_ctas}ctas")
def num_ctas(request):
```
- **EN:** Defines the helper function `num_ctas`. Decorators: `pytest.fixture(params=[1, 2, 4], ids=lambda num_ctas: f'{num_ctas}ctas')`. Parameters: `request`. Key calls include `pytest.fixture`. This scope touches shared fixtures.
- **CN:** 定义辅助函数 `num_ctas`。 装饰器：`pytest.fixture(params=[1, 2, 4], ids=lambda num_ctas: f'{num_ctas}ctas')`。 参数：`request`。 关键调用包括 `pytest.fixture`。 该作用域涉及共享 fixture。

#### Lines 24-24

```python
    return request.param
```
- **EN:** Carries supporting logic for the surrounding test or helper scope.
- **CN:** 为周围的测试或辅助作用域提供支撑逻辑。

### Lines 25-27

```python


def assert_expected_cuda_failure(exc):
```
- **EN:** Defines the helper function `assert_expected_cuda_failure`. Parameters: `exc`. Key calls include `any`. This scope touches kernel launch orchestration.
- **CN:** 定义辅助函数 `assert_expected_cuda_failure`。 参数：`exc`。 关键调用包括 `any`。 该作用域涉及kernel 启动编排。

#### Lines 28-29

```python
    assert exc is not None
    assert any(msg in str(exc) for msg in ["device-side assert", "unspecified launch failure"]), str(exc)
```
- **EN:** Invokes `any` to execute the test logic. Validates behavior with 2 assertion(s). Relevant themes: kernel launch orchestration.
- **CN:** 调用 `any` 执行测试逻辑。 通过 2 个断言验证行为。 相关主题：kernel 启动编排。

### Lines 30-33

```python


@gluon.constexpr_function
def mma_cga_layout(num_ctas, op_idx, two_cta=False):
```
- **EN:** Defines the helper function `mma_cga_layout`. Decorators: `gluon.constexpr_function`. Parameters: `num_ctas`, `op_idx`, `two_cta`. Key calls include `getattr`, `num_ctas.bit_length`, `result.append`. This scope touches layout transformation reasoning.
- **CN:** 定义辅助函数 `mma_cga_layout`。 装饰器：`gluon.constexpr_function`。 参数：`num_ctas`、`op_idx`、`two_cta`。 关键调用包括 `getattr`、`num_ctas.bit_length`、`result.append`。 该作用域涉及布局变换推理。

#### Lines 34-36

```python
    num_ctas = getattr(num_ctas, "value", num_ctas)
    op_idx = getattr(op_idx, "value", op_idx)
    two_cta = getattr(two_cta, "value", two_cta)
```
- **EN:** Prepares or updates state through `num_ctas`, `op_idx`, `two_cta`. Invokes `getattr` to execute the test logic.
- **CN:** 通过 `num_ctas`、`op_idx`、`two_cta` 准备或更新状态。 调用 `getattr` 执行测试逻辑。

#### Lines 37-39

```python
    assert op_idx in (0, 1, 2)
    # For now, but the code above is generic really
    assert num_ctas <= 4
```
- **EN:** Validates behavior with 2 assertion(s).
- **CN:** 通过 2 个断言验证行为。

#### Lines 40-41

```python
    log2_num_ctas = num_ctas.bit_length() - 1
    cga_layout = [[1, 0], [0, 1]][:log2_num_ctas]
```
- **EN:** Prepares or updates state through `log2_num_ctas`, `cga_layout`. Invokes `num_ctas.bit_length` to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `log2_num_ctas`、`cga_layout` 准备或更新状态。 调用 `num_ctas.bit_length` 执行测试逻辑。 相关主题：布局变换推理。

#### Lines 42-43

```python
    if op_idx == 2 or not cga_layout:
        return tuple(tuple(b) for b in cga_layout)
```
- **EN:** Branches on runtime or test conditions. Relevant themes: layout transformation reasoning.
- **CN:** 根据运行时或测试条件进行分支。 相关主题：布局变换推理。

#### Lines 44-46

```python

    # 2CTA performs an outer product so bases are [1, 0] and [0, 1].
    assert cga_layout[0] == [1, 0]
```
- **EN:** Validates behavior with 1 assertion(s). Relevant themes: layout transformation reasoning.
- **CN:** 通过 1 个断言验证行为。 相关主题：布局变换推理。

#### Lines 47-48

```python
    first = (1, 0) if op_idx == 0 else ((0, 1) if two_cta else (0, 0))
    result = [first]
```
- **EN:** Prepares or updates state through `first`, `result`.
- **CN:** 通过 `first`、`result` 准备或更新状态。

#### Lines 49-56

```python
    # Broadcast along K (the reduction dimension). We multiply by 2 for
    # op_idx == 1, as we have added the (0, 1) basis.
    for b in cga_layout[1:]:
        if op_idx == 0:
            result.append((b[0], 0))
        else:
            mul = 2 if two_cta else 1
            result.append((0, mul * b[1]))
```
- **EN:** Invokes `result.append` to execute the test logic. Iterates across cases or data tiles. Relevant themes: layout transformation reasoning.
- **CN:** 调用 `result.append` 执行测试逻辑。 通过循环覆盖多个用例或数据分块。 相关主题：布局变换推理。

#### Lines 57-57

```python
    return tuple(result)
```
- **EN:** Carries supporting logic for the surrounding test or helper scope.
- **CN:** 为周围的测试或辅助作用域提供支撑逻辑。

### Lines 58-61

```python


@gluon.constexpr_function
def mma_block_m(num_ctas):
```
- **EN:** Defines the helper function `mma_block_m`. Decorators: `gluon.constexpr_function`. Parameters: `num_ctas`. Key calls include `getattr`.
- **CN:** 定义辅助函数 `mma_block_m`。 装饰器：`gluon.constexpr_function`。 参数：`num_ctas`。 关键调用包括 `getattr`。

#### Lines 62-63

```python
    num_ctas = getattr(num_ctas, "value", num_ctas)
    return 256 if num_ctas > 1 else 128
```
- **EN:** Prepares or updates state through `num_ctas`. Invokes `getattr` to execute the test logic.
- **CN:** 通过 `num_ctas` 准备或更新状态。 调用 `getattr` 执行测试逻辑。

### Lines 64-67

```python


@gluon.constexpr_function
def mma_block_n(num_ctas):
```
- **EN:** Defines the helper function `mma_block_n`. Decorators: `gluon.constexpr_function`. Parameters: `num_ctas`. Key calls include `getattr`.
- **CN:** 定义辅助函数 `mma_block_n`。 装饰器：`gluon.constexpr_function`。 参数：`num_ctas`。 关键调用包括 `getattr`。

#### Lines 68-69

```python
    num_ctas = getattr(num_ctas, "value", num_ctas)
    return 256 if num_ctas == 4 else 128
```
- **EN:** Prepares or updates state through `num_ctas`. Invokes `getattr` to execute the test logic.
- **CN:** 通过 `num_ctas` 准备或更新状态。 调用 `getattr` 执行测试逻辑。

### Lines 70-73

```python


@gluon.constexpr_function
def default_cga_layout(num_ctas, rank, dim=0):
```
- **EN:** Defines the helper function `default_cga_layout`. Decorators: `gluon.constexpr_function`. Parameters: `num_ctas`, `rank`, `dim`. Key calls include `getattr`, `num_ctas.bit_length`. This scope touches layout transformation reasoning.
- **CN:** 定义辅助函数 `default_cga_layout`。 装饰器：`gluon.constexpr_function`。 参数：`num_ctas`、`rank`、`dim`。 关键调用包括 `getattr`、`num_ctas.bit_length`。 该作用域涉及布局变换推理。

#### Lines 74-74

```python
    num_ctas = getattr(num_ctas, "value", num_ctas)
```
- **EN:** Prepares or updates state through `num_ctas`. Invokes `getattr` to execute the test logic.
- **CN:** 通过 `num_ctas` 准备或更新状态。 调用 `getattr` 执行测试逻辑。

#### Lines 75-76

```python
    if num_ctas == 1:
        return []
```
- **EN:** Branches on runtime or test conditions.
- **CN:** 根据运行时或测试条件进行分支。

#### Lines 77-77

```python
    assert 0 <= dim < rank
```
- **EN:** Validates behavior with 1 assertion(s).
- **CN:** 通过 1 个断言验证行为。

#### Lines 78-78

```python
    return [[0] * dim + [1 << i] + [0] * (rank - dim - 1) for i in range(num_ctas.bit_length() - 1)]
```
- **EN:** Invokes `num_ctas.bit_length` to execute the test logic.
- **CN:** 调用 `num_ctas.bit_length` 执行测试逻辑。

### Lines 79-82

```python


@gluon.constexpr_function
def multicast_cga_layout(num_ctas, rank):
```
- **EN:** Defines the helper function `multicast_cga_layout`. Decorators: `gluon.constexpr_function`. Parameters: `num_ctas`, `rank`. Key calls include `getattr`, `num_ctas.bit_length`. This scope touches layout transformation reasoning.
- **CN:** 定义辅助函数 `multicast_cga_layout`。 装饰器：`gluon.constexpr_function`。 参数：`num_ctas`、`rank`。 关键调用包括 `getattr`、`num_ctas.bit_length`。 该作用域涉及布局变换推理。

#### Lines 83-83

```python
    num_ctas = getattr(num_ctas, "value", num_ctas)
```
- **EN:** Prepares or updates state through `num_ctas`. Invokes `getattr` to execute the test logic.
- **CN:** 通过 `num_ctas` 准备或更新状态。 调用 `getattr` 执行测试逻辑。

#### Lines 84-85

```python
    if num_ctas == 1:
        return []
```
- **EN:** Branches on runtime or test conditions.
- **CN:** 根据运行时或测试条件进行分支。

#### Lines 86-86

```python
    return [[0] * rank for _ in range(num_ctas.bit_length() - 1)]
```
- **EN:** Invokes `num_ctas.bit_length` to execute the test logic.
- **CN:** 调用 `num_ctas.bit_length` 执行测试逻辑。

### Lines 87-90

```python


# Use the same block size for all tests
XBLOCK = ttgl.constexpr(128)
```
- **EN:** Prepares or updates state through `XBLOCK`. Invokes `ttgl.constexpr` to execute the test logic.
- **CN:** 通过 `XBLOCK` 准备或更新状态。 调用 `ttgl.constexpr` 执行测试逻辑。

### Lines 91-94

```python


@gluon.jit
def failing_kernel(input):
```
- **EN:** Defines the helper function `failing_kernel`. Decorators: `gluon.jit`. Parameters: `input`. Key calls include `default_cga_layout`, `ttgl.NVMMASharedLayout`, `ttgl.allocate_shared_memory`, `ttgl.BlockedLayout`, `ampere.async_copy.async_copy_global_to_shared`, `ampere.async_copy.commit_group`, and 4 more. This scope touches layout transformation reasoning.
- **CN:** 定义辅助函数 `failing_kernel`。 装饰器：`gluon.jit`。 参数：`input`。 关键调用包括 `default_cga_layout`、`ttgl.NVMMASharedLayout`、`ttgl.allocate_shared_memory`、`ttgl.BlockedLayout`、`ampere.async_copy.async_copy_global_to_shared`、`ampere.async_copy.commit_group` 等另外 4 项。 该作用域涉及布局变换推理。

#### Lines 95-109

```python
    cga_layout: ttgl.constexpr = default_cga_layout(ttgl.num_ctas(), 2)
    smem_layout: ttgl.constexpr = ttgl.NVMMASharedLayout(swizzle_byte_width=128, element_bitwidth=16, rank=2,
                                                         cga_layout=cga_layout)
    smem = ttgl.allocate_shared_memory(ttgl.float16, [XBLOCK, XBLOCK], smem_layout)
    blocked_layout: ttgl.constexpr = ttgl.BlockedLayout(size_per_thread=[1, XBLOCK], threads_per_warp=[32, 1],
                                                        warps_per_cta=[4, 1], order=[0, 1], cga_layout=cga_layout)
    offs_m = ttgl.arange(0, XBLOCK, layout=ttgl.SliceLayout(dim=1, parent=blocked_layout))[:, None]
    offs_n = ttgl.arange(0, XBLOCK, layout=ttgl.SliceLayout(dim=0, parent=blocked_layout))[None, :]
    offs = offs_m * XBLOCK + offs_n
    ampere.async_copy.async_copy_global_to_shared(smem, input + offs)
    ampere.async_copy.commit_group()

    ampere.async_copy.async_copy_global_to_shared(smem, input + offs)
    ampere.async_copy.commit_group()
    ampere.async_copy.wait_group(0)
```
- **EN:** Prepares or updates state through `cga_layout`, `smem_layout`, `smem`, `blocked_layout`, `offs_m`, `offs_n`, `offs`. Invokes `default_cga_layout`, `ttgl.num_ctas`, `ttgl.NVMMASharedLayout`, `ttgl.allocate_shared_memory`, `ttgl.BlockedLayout`, `ttgl.arange`, and 4 more to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `cga_layout`、`smem_layout`、`smem`、`blocked_layout`、`offs_m`、`offs_n`、`offs` 准备或更新状态。 调用 `default_cga_layout`、`ttgl.num_ctas`、`ttgl.NVMMASharedLayout`、`ttgl.allocate_shared_memory`、`ttgl.BlockedLayout`、`ttgl.arange` 等另外 4 项 执行测试逻辑。 相关主题：布局变换推理。

### Lines 110-112

```python


def run_failing_kernel(device, enable_consan, mode, num_ctas):
```
- **EN:** Defines the helper function `run_failing_kernel`. Parameters: `device`, `enable_consan`, `mode`, `num_ctas`. Key calls include `torch.randn`, `knobs.refresh_knobs`. This scope touches PyTorch tensor setup and checks, random-data generation.
- **CN:** 定义辅助函数 `run_failing_kernel`。 参数：`device`、`enable_consan`、`mode`、`num_ctas`。 关键调用包括 `torch.randn`、`knobs.refresh_knobs`。 该作用域涉及PyTorch 张量准备与校验、随机数据生成。

#### Lines 113-118

```python
    if enable_consan:
        if mode == "env":
            os.environ["TRITON_INSTRUMENTATION_MODE"] = "consan"
            knobs.refresh_knobs()
        elif mode == "knob":
            knobs.compilation.instrumentation_mode = "consan"
```
- **EN:** Invokes `knobs.refresh_knobs` to execute the test logic. Branches on runtime or test conditions.
- **CN:** 调用 `knobs.refresh_knobs` 执行测试逻辑。 根据运行时或测试条件进行分支。

#### Lines 119-121

```python

    input = torch.randn((XBLOCK, XBLOCK), device=device, dtype=torch.float16)
    failing_kernel[(1, )](input, num_ctas=num_ctas)
```
- **EN:** Prepares or updates state through `input`. Invokes `torch.randn` to execute the test logic. Relevant themes: PyTorch tensor setup and checks, random-data generation.
- **CN:** 通过 `input` 准备或更新状态。 调用 `torch.randn` 执行测试逻辑。 相关主题：PyTorch 张量准备与校验、随机数据生成。

### Lines 122-126

```python


@pytest.mark.skipif(not is_cuda() or torch.cuda.get_device_capability()[0] < 9, reason="Requires hopper")
def test_cache_miss_knob(device, monkeypatch, num_ctas):
    # First run without consan
```
- **EN:** Defines the test function `test_cache_miss_knob`. Decorators: `pytest.mark.skipif(not is_cuda() or torch.cuda.get_device_capability()[0] < 9, reason='Requires hopper')`. Parameters: `device`, `monkeypatch`, `num_ctas`. Key calls include `pytest.mark.skipif`, `run_in_process`, `monkeypatch.setenv`, `any`, `is_cuda`, `torch.cuda.get_device_capability`. This scope touches PyTorch tensor setup and checks, cache management behavior, kernel launch orchestration.
- **CN:** 定义测试函数 `test_cache_miss_knob`。 装饰器：`pytest.mark.skipif(not is_cuda() or torch.cuda.get_device_capability()[0] < 9, reason='Requires hopper')`。 参数：`device`、`monkeypatch`、`num_ctas`。 关键调用包括 `pytest.mark.skipif`、`run_in_process`、`monkeypatch.setenv`、`any`、`is_cuda`、`torch.cuda.get_device_capability`。 该作用域涉及PyTorch 张量准备与校验、缓存管理行为、kernel 启动编排。

#### Lines 127-131

```python
    run_in_process(run_failing_kernel, (device, False, "knob", num_ctas))

    # Then run with consan and assert that if fails
    monkeypatch.setenv("CUDA_LAUNCH_BLOCKING", "1")
    result = run_in_process(run_failing_kernel, (device, True, "knob", num_ctas))
```
- **EN:** Prepares or updates state through `result`. Invokes `run_in_process`, `monkeypatch.setenv` to execute the test logic. Relevant themes: kernel launch orchestration.
- **CN:** 通过 `result` 准备或更新状态。 调用 `run_in_process`、`monkeypatch.setenv` 执行测试逻辑。 相关主题：kernel 启动编排。

#### Lines 132-133

```python
    assert result.exc is not None
    assert any(msg in str(result.exc) for msg in ["device-side assert", "unspecified launch failure"])
```
- **EN:** Invokes `any` to execute the test logic. Validates behavior with 2 assertion(s). Relevant themes: kernel launch orchestration.
- **CN:** 调用 `any` 执行测试逻辑。 通过 2 个断言验证行为。 相关主题：kernel 启动编排。

### Lines 134-138

```python


@pytest.mark.skipif(not is_cuda() or torch.cuda.get_device_capability()[0] < 9, reason="Requires hopper")
def test_cache_miss_env(device, monkeypatch, num_ctas):
    # First run without consan
```
- **EN:** Defines the test function `test_cache_miss_env`. Decorators: `pytest.mark.skipif(not is_cuda() or torch.cuda.get_device_capability()[0] < 9, reason='Requires hopper')`. Parameters: `device`, `monkeypatch`, `num_ctas`. Key calls include `pytest.mark.skipif`, `run_in_process`, `monkeypatch.setenv`, `any`, `is_cuda`, `torch.cuda.get_device_capability`. This scope touches PyTorch tensor setup and checks, cache management behavior, kernel launch orchestration.
- **CN:** 定义测试函数 `test_cache_miss_env`。 装饰器：`pytest.mark.skipif(not is_cuda() or torch.cuda.get_device_capability()[0] < 9, reason='Requires hopper')`。 参数：`device`、`monkeypatch`、`num_ctas`。 关键调用包括 `pytest.mark.skipif`、`run_in_process`、`monkeypatch.setenv`、`any`、`is_cuda`、`torch.cuda.get_device_capability`。 该作用域涉及PyTorch 张量准备与校验、缓存管理行为、kernel 启动编排。

#### Lines 139-143

```python
    run_in_process(run_failing_kernel, (device, False, "env", num_ctas))

    # Then run with consan and assert that if fails
    monkeypatch.setenv("CUDA_LAUNCH_BLOCKING", "1")
    result = run_in_process(run_failing_kernel, (device, True, "env", num_ctas))
```
- **EN:** Prepares or updates state through `result`. Invokes `run_in_process`, `monkeypatch.setenv` to execute the test logic. Relevant themes: kernel launch orchestration.
- **CN:** 通过 `result` 准备或更新状态。 调用 `run_in_process`、`monkeypatch.setenv` 执行测试逻辑。 相关主题：kernel 启动编排。

#### Lines 144-145

```python
    assert result.exc is not None
    assert any(msg in str(result.exc) for msg in ["device-side assert", "unspecified launch failure"])
```
- **EN:** Invokes `any` to execute the test logic. Validates behavior with 2 assertion(s). Relevant themes: kernel launch orchestration.
- **CN:** 调用 `any` 执行测试逻辑。 通过 2 个断言验证行为。 相关主题：kernel 启动编排。

### Lines 146-149

```python


@pytest.mark.skipif(not is_cuda() or torch.cuda.get_device_capability()[0] < 9, reason="Requires hopper")
def test_consan_uses_profile_scratch(device, fresh_knobs, num_ctas):
```
- **EN:** Defines the test function `test_consan_uses_profile_scratch`. Decorators: `pytest.mark.skipif(not is_cuda() or torch.cuda.get_device_capability()[0] < 9, reason='Requires hopper')`. Parameters: `device`, `fresh_knobs`, `num_ctas`. Key calls include `pytest.mark.skipif`, `knobs.cache.scope`, `knobs.runtime.scope`, `tempfile.mkdtemp`, `torch.randn`, `failing_kernel.warmup`, and 2 more. This scope touches PyTorch tensor setup and checks, cache management behavior, random-data generation.
- **CN:** 定义测试函数 `test_consan_uses_profile_scratch`。 装饰器：`pytest.mark.skipif(not is_cuda() or torch.cuda.get_device_capability()[0] < 9, reason='Requires hopper')`。 参数：`device`、`fresh_knobs`、`num_ctas`。 关键调用包括 `pytest.mark.skipif`、`knobs.cache.scope`、`knobs.runtime.scope`、`tempfile.mkdtemp`、`torch.randn`、`failing_kernel.warmup` 等另外 2 项。 该作用域涉及PyTorch 张量准备与校验、缓存管理行为、随机数据生成。

#### Lines 150-156

```python
    with knobs.cache.scope(), knobs.runtime.scope():
        knobs.cache.dir = tempfile.mkdtemp(prefix="triton-cache-")
        fresh_knobs.compilation.instrumentation_mode = "consan"
        input = torch.randn((XBLOCK, XBLOCK), device=device, dtype=torch.float16)
        compiled = failing_kernel.warmup(input, grid=(1, ), num_ctas=num_ctas)
        assert compiled.metadata.profile_scratch_size > 0
        assert compiled.metadata.global_scratch_size == 0
```
- **EN:** Invokes `knobs.cache.scope`, `knobs.runtime.scope`, `tempfile.mkdtemp`, `torch.randn`, `failing_kernel.warmup` to execute the test logic. Validates behavior with 2 assertion(s). Uses context managers to control resources or expectations. Relevant themes: PyTorch tensor setup and checks, cache management behavior, random-data generation.
- **CN:** 调用 `knobs.cache.scope`、`knobs.runtime.scope`、`tempfile.mkdtemp`、`torch.randn`、`failing_kernel.warmup` 执行测试逻辑。 通过 2 个断言验证行为。 使用上下文管理器控制资源或预期行为。 相关主题：PyTorch 张量准备与校验、缓存管理行为、随机数据生成。

### Lines 157-161

```python


@pytest.mark.skipif(not is_cuda() or torch.cuda.get_device_capability()[0] < 10, reason="Requires blackwell or newer")
@pytest.mark.parametrize("MEMORY_KIND", ["shared", "tensor"])
def test_consan_initializes_allocations_with_nan(MEMORY_KIND, device, fresh_knobs, num_ctas):
```
- **EN:** Defines the test function `test_consan_initializes_allocations_with_nan`. Decorators: `pytest.mark.skipif(not is_cuda() or torch.cuda.get_device_capability()[0] < 10, reason='Requires blackwell or newer')`, `pytest.mark.parametrize('MEMORY_KIND', ['shared', 'tensor'])`. Parameters: `MEMORY_KIND`, `device`, `fresh_knobs`, `num_ctas`. Nested definitions in this scope: `kernel`. Key calls include `pytest.mark.skipif`, `pytest.mark.parametrize`, `torch.empty`, `default_cga_layout`, `ttgl.BlockedLayout`, `ttgl.store`, and 11 more. This scope touches pytest parametrization, PyTorch tensor setup and checks, layout transformation reasoning.
- **CN:** 定义测试函数 `test_consan_initializes_allocations_with_nan`。 装饰器：`pytest.mark.skipif(not is_cuda() or torch.cuda.get_device_capability()[0] < 10, reason='Requires blackwell or newer')`、`pytest.mark.parametrize('MEMORY_KIND', ['shared', 'tensor'])`。 参数：`MEMORY_KIND`、`device`、`fresh_knobs`、`num_ctas`。 该作用域中的嵌套定义：`kernel`。 关键调用包括 `pytest.mark.skipif`、`pytest.mark.parametrize`、`torch.empty`、`default_cga_layout`、`ttgl.BlockedLayout`、`ttgl.store` 等另外 11 项。 该作用域涉及pytest 参数化、PyTorch 张量准备与校验、布局变换推理。

#### Lines 162-162

```python
    fresh_knobs.compilation.instrumentation_mode = "consan"
```
- **EN:** Prepares or updates state through `fresh_knobs`.
- **CN:** 通过 `fresh_knobs` 准备或更新状态。

#### Lines 163-165

```python

    @gluon.jit
    def kernel(output, MEMORY_KIND: ttgl.constexpr):
```
- **EN:** Defines the helper function `kernel`. Decorators: `gluon.jit`. Parameters: `output`, `MEMORY_KIND`. Key calls include `default_cga_layout`, `ttgl.BlockedLayout`, `ttgl.store`, `ttgl.num_ctas`, `ttgl.arange`, `ttgl.NVMMASharedLayout`, and 5 more. This scope touches layout transformation reasoning.
- **CN:** 定义辅助函数 `kernel`。 装饰器：`gluon.jit`。 参数：`output`、`MEMORY_KIND`。 关键调用包括 `default_cga_layout`、`ttgl.BlockedLayout`、`ttgl.store`、`ttgl.num_ctas`、`ttgl.arange`、`ttgl.NVMMASharedLayout` 等另外 5 项。 该作用域涉及布局变换推理。

##### Lines 166-172

```python
        block_m: ttgl.constexpr = XBLOCK * ttgl.num_ctas()
        cga_layout: ttgl.constexpr = default_cga_layout(ttgl.num_ctas(), 2)
        reg_layout: ttgl.constexpr = ttgl.BlockedLayout(size_per_thread=[1, XBLOCK], threads_per_warp=[32, 1],
                                                        warps_per_cta=[4, 1], order=[0, 1], cga_layout=cga_layout)
        offs_m = ttgl.arange(0, block_m, ttgl.SliceLayout(1, reg_layout))[:, None]
        offs_n = ttgl.arange(0, XBLOCK, ttgl.SliceLayout(0, reg_layout))[None, :]
        offs = offs_m * XBLOCK + offs_n
```
- **EN:** Prepares or updates state through `block_m`, `cga_layout`, `reg_layout`, `offs_m`, `offs_n`, `offs`. Invokes `ttgl.num_ctas`, `default_cga_layout`, `ttgl.BlockedLayout`, `ttgl.arange`, `ttgl.SliceLayout` to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `block_m`、`cga_layout`、`reg_layout`、`offs_m`、`offs_n`、`offs` 准备或更新状态。 调用 `ttgl.num_ctas`、`default_cga_layout`、`ttgl.BlockedLayout`、`ttgl.arange`、`ttgl.SliceLayout` 执行测试逻辑。 相关主题：布局变换推理。

##### Lines 173-182

```python
        if MEMORY_KIND == "shared":
            memory_layout: ttgl.constexpr = ttgl.NVMMASharedLayout(swizzle_byte_width=128, element_bitwidth=32, rank=2,
                                                                   cga_layout=cga_layout)
            alloc = ttgl.allocate_shared_memory(ttgl.float32, [block_m, XBLOCK], memory_layout)
            value = alloc.load(reg_layout)
        else:
            memory_layout: ttgl.constexpr = blackwell.TensorMemoryLayout((XBLOCK, XBLOCK), col_stride=1,
                                                                         cga_layout=cga_layout)
            alloc = blackwell.allocate_tensor_memory(ttgl.float32, [block_m, XBLOCK], memory_layout)
            value = alloc.load(reg_layout)
```
- **EN:** Invokes `ttgl.NVMMASharedLayout`, `ttgl.allocate_shared_memory`, `alloc.load`, `blackwell.TensorMemoryLayout`, `blackwell.allocate_tensor_memory` to execute the test logic. Branches on runtime or test conditions. Relevant themes: layout transformation reasoning.
- **CN:** 调用 `ttgl.NVMMASharedLayout`、`ttgl.allocate_shared_memory`、`alloc.load`、`blackwell.TensorMemoryLayout`、`blackwell.allocate_tensor_memory` 执行测试逻辑。 根据运行时或测试条件进行分支。 相关主题：布局变换推理。

##### Lines 183-183

```python
        ttgl.store(output + offs, value)
```
- **EN:** Invokes `ttgl.store` to execute the test logic.
- **CN:** 调用 `ttgl.store` 执行测试逻辑。

#### Lines 184-186

```python

    output = torch.empty((XBLOCK.value * num_ctas, XBLOCK.value), device=device, dtype=torch.float32)
    kernel[(1, )](output, MEMORY_KIND=MEMORY_KIND, num_warps=4, num_ctas=num_ctas)
```
- **EN:** Prepares or updates state through `output`. Invokes `torch.empty` to execute the test logic. Relevant themes: PyTorch tensor setup and checks.
- **CN:** 通过 `output` 准备或更新状态。 调用 `torch.empty` 执行测试逻辑。 相关主题：PyTorch 张量准备与校验。

#### Lines 187-187

```python
    assert torch.isnan(output).all()
```
- **EN:** Invokes `torch.isnan` to execute the test logic. Validates behavior with 1 assertion(s). Relevant themes: PyTorch tensor setup and checks.
- **CN:** 调用 `torch.isnan` 执行测试逻辑。 通过 1 个断言验证行为。 相关主题：PyTorch 张量准备与校验。

### Lines 188-192

```python


@pytest.mark.skipif(not is_cuda() or torch.cuda.get_device_capability()[0] < 9, reason="Requires hopper or newer")
@pytest.mark.parametrize("FAILURE", [True, False])
def test_async_tma_kernel(FAILURE, device, run_wrapper, monkeypatch, num_ctas):
```
- **EN:** Defines the test function `test_async_tma_kernel`. Decorators: `pytest.mark.skipif(not is_cuda() or torch.cuda.get_device_capability()[0] < 9, reason='Requires hopper or newer')`, `pytest.mark.parametrize('FAILURE', [True, False])`. Parameters: `FAILURE`, `device`, `run_wrapper`, `monkeypatch`, `num_ctas`. Nested definitions in this scope: `kernel`. Key calls include `pytest.mark.skipif`, `pytest.mark.parametrize`, `monkeypatch.setenv`, `knobs.refresh_knobs`, `torch.randn`, `torch.empty`, and 20 more. This scope touches pytest parametrization, PyTorch tensor setup and checks, tensor/descriptor metadata, runtime driver interaction.
- **CN:** 定义测试函数 `test_async_tma_kernel`。 装饰器：`pytest.mark.skipif(not is_cuda() or torch.cuda.get_device_capability()[0] < 9, reason='Requires hopper or newer')`、`pytest.mark.parametrize('FAILURE', [True, False])`。 参数：`FAILURE`、`device`、`run_wrapper`、`monkeypatch`、`num_ctas`。 该作用域中的嵌套定义：`kernel`。 关键调用包括 `pytest.mark.skipif`、`pytest.mark.parametrize`、`monkeypatch.setenv`、`knobs.refresh_knobs`、`torch.randn`、`torch.empty` 等另外 20 项。 该作用域涉及pytest 参数化、PyTorch 张量准备与校验、张量/描述符元数据、运行时驱动交互。

#### Lines 193-201

```python
    if run_wrapper:
        result = run_in_process(test_async_tma_kernel, (FAILURE, device, False, monkeypatch, num_ctas))
        if FAILURE:
            assert_expected_cuda_failure(result.exc)
            assert "Buffer being accessed has outstanding writes" in result.driver_stderr_output
        else:
            assert result.exc is None
            assert result.driver_stderr_output == ""
        return
```
- **EN:** Invokes `run_in_process`, `assert_expected_cuda_failure` to execute the test logic. Validates behavior with 3 assertion(s). Branches on runtime or test conditions. Relevant themes: runtime driver interaction.
- **CN:** 调用 `run_in_process`、`assert_expected_cuda_failure` 执行测试逻辑。 通过 3 个断言验证行为。 根据运行时或测试条件进行分支。 相关主题：运行时驱动交互。

#### Lines 202-205

```python

    monkeypatch.setenv("TRITON_INSTRUMENTATION_MODE", "consan")
    monkeypatch.setenv("CUDA_LAUNCH_BLOCKING", "1")
    knobs.refresh_knobs()
```
- **EN:** Invokes `monkeypatch.setenv`, `knobs.refresh_knobs` to execute the test logic. Relevant themes: kernel launch orchestration.
- **CN:** 调用 `monkeypatch.setenv`、`knobs.refresh_knobs` 执行测试逻辑。 相关主题：kernel 启动编排。

#### Lines 206-208

```python

    @gluon.jit
    def kernel(input_desc, out, FAILURE: ttgl.constexpr):
```
- **EN:** Defines the helper function `kernel`. Decorators: `gluon.jit`. Parameters: `input_desc`, `out`, `FAILURE`. Key calls include `default_cga_layout`, `ttgl.BlockedLayout`, `ttgl.allocate_shared_memory`, `mbarrier.allocate_mbarrier`, `mbarrier.init`, `mbarrier.expect`, and 8 more. This scope touches layout transformation reasoning.
- **CN:** 定义辅助函数 `kernel`。 装饰器：`gluon.jit`。 参数：`input_desc`、`out`、`FAILURE`。 关键调用包括 `default_cga_layout`、`ttgl.BlockedLayout`、`ttgl.allocate_shared_memory`、`mbarrier.allocate_mbarrier`、`mbarrier.init`、`mbarrier.expect` 等另外 8 项。 该作用域涉及布局变换推理。

##### Lines 209-226

```python
        block_m: ttgl.constexpr = XBLOCK * ttgl.num_ctas()
        cga_layout: ttgl.constexpr = default_cga_layout(ttgl.num_ctas(), 2)
        blocked_layout: ttgl.constexpr = ttgl.BlockedLayout(size_per_thread=[1, 1], threads_per_warp=[32, 1],
                                                            warps_per_cta=[4, 1], order=[0, 1], cga_layout=cga_layout)
        smem = ttgl.allocate_shared_memory(ttgl.float16, [block_m, XBLOCK], input_desc.layout)
        bar = mbarrier.allocate_mbarrier()
        mbarrier.init(bar, count=1)
        mbarrier.expect(bar, input_desc.nbytes_per_cta)
        tma.async_load(input_desc, [0, 0], bar, smem)
        mbarrier.wait(bar, 0, pred=(not FAILURE), deps=[smem])
        val = smem.load(blocked_layout)
        mbarrier.wait(bar, 0, pred=FAILURE, deps=[smem])
        mbarrier.invalidate(bar)

        out_m = ttgl.arange(0, block_m, ttgl.SliceLayout(1, blocked_layout))[:, None]
        out_n = ttgl.arange(0, XBLOCK, ttgl.SliceLayout(0, blocked_layout))[None, :]
        out_ptr = out + out_m * XBLOCK + out_n
        ttgl.store(out_ptr, val)
```
- **EN:** Prepares or updates state through `block_m`, `cga_layout`, `blocked_layout`, `smem`, `bar`, `val`, `out_m`, `out_n`, and 1 more. Invokes `ttgl.num_ctas`, `default_cga_layout`, `ttgl.BlockedLayout`, `ttgl.allocate_shared_memory`, `mbarrier.allocate_mbarrier`, `mbarrier.init`, and 8 more to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `block_m`、`cga_layout`、`blocked_layout`、`smem`、`bar`、`val`、`out_m`、`out_n` 等另外 1 项 准备或更新状态。 调用 `ttgl.num_ctas`、`default_cga_layout`、`ttgl.BlockedLayout`、`ttgl.allocate_shared_memory`、`mbarrier.allocate_mbarrier`、`mbarrier.init` 等另外 8 项 执行测试逻辑。 相关主题：布局变换推理。

#### Lines 227-234

```python

    block_m = XBLOCK.value * num_ctas
    input = torch.randn((block_m, XBLOCK.value), device=device, dtype=torch.float16)
    output = torch.empty((block_m, XBLOCK.value), device=device, dtype=torch.float16)
    shared_layout = ttgl.NVMMASharedLayout(swizzle_byte_width=128, element_bitwidth=16, rank=2,
                                           cga_layout=default_cga_layout(num_ctas, 2))
    input_desc = gluon.nvidia.hopper.TensorDescriptor.from_tensor(input, [block_m, XBLOCK.value], shared_layout)
    kernel[(1, )](input_desc, output, FAILURE=FAILURE, num_warps=4, num_ctas=num_ctas)
```
- **EN:** Prepares or updates state through `block_m`, `input`, `output`, `shared_layout`, `input_desc`. Invokes `torch.randn`, `torch.empty`, `ttgl.NVMMASharedLayout`, `default_cga_layout`, `gluon.nvidia.hopper.TensorDescriptor.from_tensor` to execute the test logic. Relevant themes: PyTorch tensor setup and checks, tensor/descriptor metadata, layout transformation reasoning, random-data generation.
- **CN:** 通过 `block_m`、`input`、`output`、`shared_layout`、`input_desc` 准备或更新状态。 调用 `torch.randn`、`torch.empty`、`ttgl.NVMMASharedLayout`、`default_cga_layout`、`gluon.nvidia.hopper.TensorDescriptor.from_tensor` 执行测试逻辑。 相关主题：PyTorch 张量准备与校验、张量/描述符元数据、布局变换推理、随机数据生成。

### Lines 235-239

```python


@pytest.mark.skipif(not is_cuda() or torch.cuda.get_device_capability()[0] < 9, reason="Requires hopper or newer")
@pytest.mark.parametrize("FAILURE", [True, False])
def test_async_tma_multicast_kernel(FAILURE, device, run_wrapper, monkeypatch, num_ctas):
```
- **EN:** Defines the test function `test_async_tma_multicast_kernel`. Decorators: `pytest.mark.skipif(not is_cuda() or torch.cuda.get_device_capability()[0] < 9, reason='Requires hopper or newer')`, `pytest.mark.parametrize('FAILURE', [True, False])`. Parameters: `FAILURE`, `device`, `run_wrapper`, `monkeypatch`, `num_ctas`. Nested definitions in this scope: `kernel`. Key calls include `pytest.mark.skipif`, `pytest.mark.parametrize`, `monkeypatch.setenv`, `knobs.refresh_knobs`, `torch.randn`, `torch.empty`, and 21 more. This scope touches pytest parametrization, PyTorch tensor setup and checks, tensor/descriptor metadata, runtime driver interaction.
- **CN:** 定义测试函数 `test_async_tma_multicast_kernel`。 装饰器：`pytest.mark.skipif(not is_cuda() or torch.cuda.get_device_capability()[0] < 9, reason='Requires hopper or newer')`、`pytest.mark.parametrize('FAILURE', [True, False])`。 参数：`FAILURE`、`device`、`run_wrapper`、`monkeypatch`、`num_ctas`。 该作用域中的嵌套定义：`kernel`。 关键调用包括 `pytest.mark.skipif`、`pytest.mark.parametrize`、`monkeypatch.setenv`、`knobs.refresh_knobs`、`torch.randn`、`torch.empty` 等另外 21 项。 该作用域涉及pytest 参数化、PyTorch 张量准备与校验、张量/描述符元数据、运行时驱动交互。

#### Lines 240-241

```python
    if num_ctas == 1:
        pytest.skip("Need at least 2 CTAs for multicast in this test")
```
- **EN:** Invokes `pytest.skip` to execute the test logic. Branches on runtime or test conditions.
- **CN:** 调用 `pytest.skip` 执行测试逻辑。 根据运行时或测试条件进行分支。

#### Lines 242-243

```python
    if FAILURE and num_ctas == 4:
        pytest.skip("Temporarily disabled: flaky with 4 CTAs when FAILURE=True")
```
- **EN:** Invokes `pytest.skip` to execute the test logic. Branches on runtime or test conditions.
- **CN:** 调用 `pytest.skip` 执行测试逻辑。 根据运行时或测试条件进行分支。

#### Lines 244-252

```python
    if run_wrapper:
        result = run_in_process(test_async_tma_multicast_kernel, (FAILURE, device, False, monkeypatch, num_ctas))
        if FAILURE:
            assert_expected_cuda_failure(result.exc)
            assert "Buffer being accessed has outstanding writes" in result.driver_stderr_output
        else:
            assert result.exc is None
            assert result.driver_stderr_output == ""
        return
```
- **EN:** Invokes `run_in_process`, `assert_expected_cuda_failure` to execute the test logic. Validates behavior with 3 assertion(s). Branches on runtime or test conditions. Relevant themes: runtime driver interaction.
- **CN:** 调用 `run_in_process`、`assert_expected_cuda_failure` 执行测试逻辑。 通过 3 个断言验证行为。 根据运行时或测试条件进行分支。 相关主题：运行时驱动交互。

#### Lines 253-256

```python

    monkeypatch.setenv("TRITON_INSTRUMENTATION_MODE", "consan")
    monkeypatch.setenv("CUDA_LAUNCH_BLOCKING", "1")
    knobs.refresh_knobs()
```
- **EN:** Invokes `monkeypatch.setenv`, `knobs.refresh_knobs` to execute the test logic. Relevant themes: kernel launch orchestration.
- **CN:** 调用 `monkeypatch.setenv`、`knobs.refresh_knobs` 执行测试逻辑。 相关主题：kernel 启动编排。

#### Lines 257-259

```python

    @gluon.jit
    def kernel(input_desc, out, FAILURE: ttgl.constexpr):
```
- **EN:** Defines the helper function `kernel`. Decorators: `gluon.jit`. Parameters: `input_desc`, `out`, `FAILURE`. Key calls include `multicast_cga_layout`, `ttgl.BlockedLayout`, `ttgl.allocate_shared_memory`, `mbarrier.allocate_mbarrier`, `mbarrier.init`, `mbarrier.expect`, and 8 more. This scope touches layout transformation reasoning.
- **CN:** 定义辅助函数 `kernel`。 装饰器：`gluon.jit`。 参数：`input_desc`、`out`、`FAILURE`。 关键调用包括 `multicast_cga_layout`、`ttgl.BlockedLayout`、`ttgl.allocate_shared_memory`、`mbarrier.allocate_mbarrier`、`mbarrier.init`、`mbarrier.expect` 等另外 8 项。 该作用域涉及布局变换推理。

##### Lines 260-276

```python
        cga_layout: ttgl.constexpr = multicast_cga_layout(ttgl.num_ctas(), 2)
        blocked_layout: ttgl.constexpr = ttgl.BlockedLayout(size_per_thread=[1, 1], threads_per_warp=[32, 1],
                                                            warps_per_cta=[4, 1], order=[0, 1], cga_layout=cga_layout)
        smem = ttgl.allocate_shared_memory(ttgl.float16, [XBLOCK, XBLOCK], input_desc.layout)
        bar = mbarrier.allocate_mbarrier()
        mbarrier.init(bar, count=1)
        mbarrier.expect(bar, input_desc.nbytes_per_cta)
        tma.async_load(input_desc, [0, 0], bar, smem, multicast=True)
        mbarrier.wait(bar, 0, pred=(not FAILURE), deps=[smem])
        val = smem.load(blocked_layout)
        mbarrier.wait(bar, 0, pred=FAILURE, deps=[smem])
        mbarrier.invalidate(bar)

        out_m = ttgl.arange(0, XBLOCK, ttgl.SliceLayout(1, blocked_layout))[:, None]
        out_n = ttgl.arange(0, XBLOCK, ttgl.SliceLayout(0, blocked_layout))[None, :]
        out_ptr = out + out_m * XBLOCK + out_n
        ttgl.store(out_ptr, val)
```
- **EN:** Prepares or updates state through `cga_layout`, `blocked_layout`, `smem`, `bar`, `val`, `out_m`, `out_n`, `out_ptr`. Invokes `multicast_cga_layout`, `ttgl.num_ctas`, `ttgl.BlockedLayout`, `ttgl.allocate_shared_memory`, `mbarrier.allocate_mbarrier`, `mbarrier.init`, and 8 more to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `cga_layout`、`blocked_layout`、`smem`、`bar`、`val`、`out_m`、`out_n`、`out_ptr` 准备或更新状态。 调用 `multicast_cga_layout`、`ttgl.num_ctas`、`ttgl.BlockedLayout`、`ttgl.allocate_shared_memory`、`mbarrier.allocate_mbarrier`、`mbarrier.init` 等另外 8 项 执行测试逻辑。 相关主题：布局变换推理。

#### Lines 277-283

```python

    input = torch.randn((XBLOCK.value, XBLOCK.value), device=device, dtype=torch.float16)
    output = torch.empty((XBLOCK.value, XBLOCK.value), device=device, dtype=torch.float16)
    shared_layout = ttgl.NVMMASharedLayout(swizzle_byte_width=128, element_bitwidth=16, rank=2,
                                           cga_layout=multicast_cga_layout(num_ctas, 2))
    input_desc = gluon.nvidia.hopper.TensorDescriptor.from_tensor(input, [XBLOCK.value, XBLOCK.value], shared_layout)
    kernel[(1, )](input_desc, output, FAILURE=FAILURE, num_warps=4, num_ctas=num_ctas)
```
- **EN:** Prepares or updates state through `input`, `output`, `shared_layout`, `input_desc`. Invokes `torch.randn`, `torch.empty`, `ttgl.NVMMASharedLayout`, `multicast_cga_layout`, `gluon.nvidia.hopper.TensorDescriptor.from_tensor` to execute the test logic. Relevant themes: PyTorch tensor setup and checks, tensor/descriptor metadata, layout transformation reasoning, random-data generation.
- **CN:** 通过 `input`、`output`、`shared_layout`、`input_desc` 准备或更新状态。 调用 `torch.randn`、`torch.empty`、`ttgl.NVMMASharedLayout`、`multicast_cga_layout`、`gluon.nvidia.hopper.TensorDescriptor.from_tensor` 执行测试逻辑。 相关主题：PyTorch 张量准备与校验、张量/描述符元数据、布局变换推理、随机数据生成。

### Lines 284-287

```python


@pytest.mark.skipif(not is_cuda() or torch.cuda.get_device_capability()[0] < 10, reason="Requires blackwell")
def test_collapsed_wait_does_not_publish_peer_cta(device, run_wrapper, monkeypatch):
```
- **EN:** Defines the test function `test_collapsed_wait_does_not_publish_peer_cta`. Decorators: `pytest.mark.skipif(not is_cuda() or torch.cuda.get_device_capability()[0] < 10, reason='Requires blackwell')`. Parameters: `device`, `run_wrapper`, `monkeypatch`. Nested definitions in this scope: `kernel`. Key calls include `pytest.mark.skipif`, `monkeypatch.setenv`, `knobs.refresh_knobs`, `torch.randn`, `ttgl.NVMMASharedLayout.get_default_for`, `gluon.nvidia.hopper.TensorDescriptor.from_tensor`, and 16 more. This scope touches PyTorch tensor setup and checks, tensor/descriptor metadata, runtime driver interaction, kernel launch orchestration.
- **CN:** 定义测试函数 `test_collapsed_wait_does_not_publish_peer_cta`。 装饰器：`pytest.mark.skipif(not is_cuda() or torch.cuda.get_device_capability()[0] < 10, reason='Requires blackwell')`。 参数：`device`、`run_wrapper`、`monkeypatch`。 该作用域中的嵌套定义：`kernel`。 关键调用包括 `pytest.mark.skipif`、`monkeypatch.setenv`、`knobs.refresh_knobs`、`torch.randn`、`ttgl.NVMMASharedLayout.get_default_for`、`gluon.nvidia.hopper.TensorDescriptor.from_tensor` 等另外 16 项。 该作用域涉及PyTorch 张量准备与校验、张量/描述符元数据、运行时驱动交互、kernel 启动编排。

#### Lines 288-292

```python
    if run_wrapper:
        result = run_in_process(test_collapsed_wait_does_not_publish_peer_cta, (device, False, monkeypatch))
        assert_expected_cuda_failure(result.exc)
        assert "Buffer being accessed has outstanding writes" in result.driver_stderr_output
        return
```
- **EN:** Invokes `run_in_process`, `assert_expected_cuda_failure` to execute the test logic. Validates behavior with 1 assertion(s). Branches on runtime or test conditions. Relevant themes: runtime driver interaction.
- **CN:** 调用 `run_in_process`、`assert_expected_cuda_failure` 执行测试逻辑。 通过 1 个断言验证行为。 根据运行时或测试条件进行分支。 相关主题：运行时驱动交互。

#### Lines 293-296

```python

    monkeypatch.setenv("TRITON_INSTRUMENTATION_MODE", "consan")
    monkeypatch.setenv("CUDA_LAUNCH_BLOCKING", "1")
    knobs.refresh_knobs()
```
- **EN:** Invokes `monkeypatch.setenv`, `knobs.refresh_knobs` to execute the test logic. Relevant themes: kernel launch orchestration.
- **CN:** 调用 `monkeypatch.setenv`、`knobs.refresh_knobs` 执行测试逻辑。 相关主题：kernel 启动编排。

#### Lines 297-299

```python

    @gluon.jit
    def kernel(a_desc, b_desc):
```
- **EN:** Defines the helper function `kernel`. Decorators: `gluon.jit`. Parameters: `a_desc`, `b_desc`. Key calls include `ttgl.BlockedLayout`, `ttgl.allocate_shared_memory`, `mbarrier.allocate_mbarrier`, `mbarrier.init`, `mbarrier.expect`, `tma.async_load`, and 7 more. This scope touches layout transformation reasoning.
- **CN:** 定义辅助函数 `kernel`。 装饰器：`gluon.jit`。 参数：`a_desc`、`b_desc`。 关键调用包括 `ttgl.BlockedLayout`、`ttgl.allocate_shared_memory`、`mbarrier.allocate_mbarrier`、`mbarrier.init`、`mbarrier.expect`、`tma.async_load` 等另外 7 项。 该作用域涉及布局变换推理。

##### Lines 300-317

```python
        blocked_a: ttgl.constexpr = ttgl.BlockedLayout([1, 1], [32, 1], [4, 1], [0, 1], ((1, 0), ))
        smem_a = ttgl.allocate_shared_memory(ttgl.float16, [256, 128], a_desc.layout)
        smem_b = ttgl.allocate_shared_memory(
            ttgl.float16, [128, 128],
            ttgl.NVMMASharedLayout.get_default_for([128, 128], ttgl.float16, cga_layout=((0, 1), )))
        tma_bar = mbarrier.allocate_mbarrier(two_ctas=True)
        mbarrier.init(tma_bar, count=1)
        mbarrier.expect(tma_bar, a_desc.nbytes_per_cta + b_desc.nbytes_per_cta)
        tma.async_load(a_desc, [0, 0], tma_bar, smem_a)
        tma.async_load(b_desc, [0, 0], tma_bar, smem_b)
        mbarrier.wait(tma_bar, 0, deps=[smem_a, smem_b])
        val = smem_a.load(blocked_a)
        smem_a.store(val)

        acc_layout: ttgl.constexpr = blackwell.TensorMemoryLayout([128, 128], col_stride=1, cga_layout=((1, 0), ),
                                                                  two_ctas=True)
        acc = blackwell.allocate_tensor_memory(ttgl.float32, [256, 128], acc_layout)
        blackwell.tcgen05_mma(smem_a, smem_b, acc, use_acc=False)
```
- **EN:** Prepares or updates state through `blocked_a`, `smem_a`, `smem_b`, `tma_bar`, `val`, `acc_layout`, `acc`. Invokes `ttgl.BlockedLayout`, `ttgl.allocate_shared_memory`, `ttgl.NVMMASharedLayout.get_default_for`, `mbarrier.allocate_mbarrier`, `mbarrier.init`, `mbarrier.expect`, and 7 more to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `blocked_a`、`smem_a`、`smem_b`、`tma_bar`、`val`、`acc_layout`、`acc` 准备或更新状态。 调用 `ttgl.BlockedLayout`、`ttgl.allocate_shared_memory`、`ttgl.NVMMASharedLayout.get_default_for`、`mbarrier.allocate_mbarrier`、`mbarrier.init`、`mbarrier.expect` 等另外 7 项 执行测试逻辑。 相关主题：布局变换推理。

#### Lines 318-325

```python

    a = torch.randn((256, 128), device=device, dtype=torch.float16)
    b = torch.randn((128, 128), device=device, dtype=torch.float16)
    a_layout = ttgl.NVMMASharedLayout.get_default_for([256, 128], ttgl.float16, cga_layout=((1, 0), ))
    b_layout = ttgl.NVMMASharedLayout.get_default_for([128, 128], ttgl.float16, cga_layout=((0, 1), ))
    a_desc = gluon.nvidia.hopper.TensorDescriptor.from_tensor(a, [256, 128], a_layout)
    b_desc = gluon.nvidia.hopper.TensorDescriptor.from_tensor(b, [128, 128], b_layout)
    kernel[(1, )](a_desc, b_desc, num_warps=4, num_ctas=2)
```
- **EN:** Prepares or updates state through `a`, `b`, `a_layout`, `b_layout`, `a_desc`, `b_desc`. Invokes `torch.randn`, `ttgl.NVMMASharedLayout.get_default_for`, `gluon.nvidia.hopper.TensorDescriptor.from_tensor` to execute the test logic. Relevant themes: PyTorch tensor setup and checks, tensor/descriptor metadata, layout transformation reasoning, random-data generation.
- **CN:** 通过 `a`、`b`、`a_layout`、`b_layout`、`a_desc`、`b_desc` 准备或更新状态。 调用 `torch.randn`、`ttgl.NVMMASharedLayout.get_default_for`、`gluon.nvidia.hopper.TensorDescriptor.from_tensor` 执行测试逻辑。 相关主题：PyTorch 张量准备与校验、张量/描述符元数据、布局变换推理、随机数据生成。

### Lines 326-330

```python


@pytest.mark.skipif(not is_cuda() or torch.cuda.get_device_capability()[0] < 10, reason="Requires blackwell")
@pytest.mark.parametrize("FAILURE", [True, False])
def test_clc_result_visibility(FAILURE, device, run_wrapper, monkeypatch, num_ctas):
```
- **EN:** Defines the test function `test_clc_result_visibility`. Decorators: `pytest.mark.skipif(not is_cuda() or torch.cuda.get_device_capability()[0] < 10, reason='Requires blackwell')`, `pytest.mark.parametrize('FAILURE', [True, False])`. Parameters: `FAILURE`, `device`, `run_wrapper`, `monkeypatch`, `num_ctas`. Nested definitions in this scope: `kernel`. Key calls include `pytest.mark.skipif`, `pytest.mark.parametrize`, `monkeypatch.setenv`, `knobs.refresh_knobs`, `torch.empty`, `run_in_process`, and 17 more. This scope touches pytest parametrization, PyTorch tensor setup and checks, runtime driver interaction, kernel launch orchestration.
- **CN:** 定义测试函数 `test_clc_result_visibility`。 装饰器：`pytest.mark.skipif(not is_cuda() or torch.cuda.get_device_capability()[0] < 10, reason='Requires blackwell')`、`pytest.mark.parametrize('FAILURE', [True, False])`。 参数：`FAILURE`、`device`、`run_wrapper`、`monkeypatch`、`num_ctas`。 该作用域中的嵌套定义：`kernel`。 关键调用包括 `pytest.mark.skipif`、`pytest.mark.parametrize`、`monkeypatch.setenv`、`knobs.refresh_knobs`、`torch.empty`、`run_in_process` 等另外 17 项。 该作用域涉及pytest 参数化、PyTorch 张量准备与校验、运行时驱动交互、kernel 启动编排。

#### Lines 331-339

```python
    if run_wrapper:
        result = run_in_process(test_clc_result_visibility, (FAILURE, device, False, monkeypatch, num_ctas))
        if FAILURE:
            assert_expected_cuda_failure(result.exc)
            assert "Buffer being accessed has outstanding writes" in result.driver_stderr_output
        else:
            assert result.exc is None
            assert result.driver_stderr_output == ""
        return
```
- **EN:** Invokes `run_in_process`, `assert_expected_cuda_failure` to execute the test logic. Validates behavior with 3 assertion(s). Branches on runtime or test conditions. Relevant themes: runtime driver interaction.
- **CN:** 调用 `run_in_process`、`assert_expected_cuda_failure` 执行测试逻辑。 通过 3 个断言验证行为。 根据运行时或测试条件进行分支。 相关主题：运行时驱动交互。

#### Lines 340-343

```python

    monkeypatch.setenv("TRITON_INSTRUMENTATION_MODE", "consan")
    monkeypatch.setenv("CUDA_LAUNCH_BLOCKING", "1")
    knobs.refresh_knobs()
```
- **EN:** Invokes `monkeypatch.setenv`, `knobs.refresh_knobs` to execute the test logic. Relevant themes: kernel launch orchestration.
- **CN:** 调用 `monkeypatch.setenv`、`knobs.refresh_knobs` 执行测试逻辑。 相关主题：kernel 启动编排。

#### Lines 344-346

```python

    @gluon.jit
    def kernel(out, FAILURE: ttgl.constexpr):
```
- **EN:** Defines the helper function `kernel`. Decorators: `gluon.jit`. Parameters: `out`, `FAILURE`. Key calls include `multicast_cga_layout`, `ttgl.SwizzledSharedLayout`, `ttgl.allocate_shared_memory`, `mbarrier.allocate_mbarrier`, `mbarrier.init`, `clc.try_cancel`, and 8 more. This scope touches layout transformation reasoning.
- **CN:** 定义辅助函数 `kernel`。 装饰器：`gluon.jit`。 参数：`out`、`FAILURE`。 关键调用包括 `multicast_cga_layout`、`ttgl.SwizzledSharedLayout`、`ttgl.allocate_shared_memory`、`mbarrier.allocate_mbarrier`、`mbarrier.init`、`clc.try_cancel` 等另外 8 项。 该作用域涉及布局变换推理。

##### Lines 347-360

```python
        cga_layout: ttgl.constexpr = multicast_cga_layout(ttgl.num_ctas(), 1)
        layout: ttgl.constexpr = ttgl.SwizzledSharedLayout(1, 1, 1, order=[0], cga_layout=cga_layout)
        clc_result = ttgl.allocate_shared_memory(ttgl.int64, [2], layout)
        clc_bar = mbarrier.allocate_mbarrier()
        mbarrier.init(clc_bar, count=1)

        clc.try_cancel(clc_result, clc_bar)
        mbarrier.expect(clc_bar, 16)
        mbarrier.wait(clc_bar, 0, pred=(not FAILURE))
        response = clc.load_result(clc_result)
        mbarrier.wait(clc_bar, 0, pred=FAILURE)
        mbarrier.invalidate(clc_bar)

        ttgl.store(out + ttgl.program_id(0), response.is_canceled())
```
- **EN:** Prepares or updates state through `cga_layout`, `layout`, `clc_result`, `clc_bar`, `response`. Invokes `multicast_cga_layout`, `ttgl.num_ctas`, `ttgl.SwizzledSharedLayout`, `ttgl.allocate_shared_memory`, `mbarrier.allocate_mbarrier`, `mbarrier.init`, and 8 more to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `cga_layout`、`layout`、`clc_result`、`clc_bar`、`response` 准备或更新状态。 调用 `multicast_cga_layout`、`ttgl.num_ctas`、`ttgl.SwizzledSharedLayout`、`ttgl.allocate_shared_memory`、`mbarrier.allocate_mbarrier`、`mbarrier.init` 等另外 8 项 执行测试逻辑。 相关主题：布局变换推理。

#### Lines 361-363

```python

    output = torch.empty((1, ), device=device, dtype=torch.bool)
    kernel[(1, )](output, FAILURE=FAILURE, num_warps=4, num_ctas=num_ctas)
```
- **EN:** Prepares or updates state through `output`. Invokes `torch.empty` to execute the test logic. Relevant themes: PyTorch tensor setup and checks.
- **CN:** 通过 `output` 准备或更新状态。 调用 `torch.empty` 执行测试逻辑。 相关主题：PyTorch 张量准备与校验。

### Lines 364-367

```python


@pytest.mark.skipif(not is_cuda() or torch.cuda.get_device_capability()[0] < 10, reason="Requires blackwell")
def test_clc_double_try_cancel_result_overwrite(device, run_wrapper, monkeypatch):
```
- **EN:** Defines the test function `test_clc_double_try_cancel_result_overwrite`. Decorators: `pytest.mark.skipif(not is_cuda() or torch.cuda.get_device_capability()[0] < 10, reason='Requires blackwell')`. Parameters: `device`, `run_wrapper`, `monkeypatch`. Nested definitions in this scope: `kernel`. Key calls include `pytest.mark.skipif`, `monkeypatch.setenv`, `knobs.refresh_knobs`, `run_in_process`, `multicast_cga_layout`, `ttgl.SwizzledSharedLayout`, and 10 more. This scope touches PyTorch tensor setup and checks, runtime driver interaction, kernel launch orchestration, layout transformation reasoning.
- **CN:** 定义测试函数 `test_clc_double_try_cancel_result_overwrite`。 装饰器：`pytest.mark.skipif(not is_cuda() or torch.cuda.get_device_capability()[0] < 10, reason='Requires blackwell')`。 参数：`device`、`run_wrapper`、`monkeypatch`。 该作用域中的嵌套定义：`kernel`。 关键调用包括 `pytest.mark.skipif`、`monkeypatch.setenv`、`knobs.refresh_knobs`、`run_in_process`、`multicast_cga_layout`、`ttgl.SwizzledSharedLayout` 等另外 10 项。 该作用域涉及PyTorch 张量准备与校验、运行时驱动交互、kernel 启动编排、布局变换推理。

#### Lines 368-372

```python
    if run_wrapper:
        result = run_in_process(test_clc_double_try_cancel_result_overwrite, (device, False, monkeypatch))
        assert result.exc is None
        assert result.driver_stderr_output == ""
        return
```
- **EN:** Invokes `run_in_process` to execute the test logic. Validates behavior with 2 assertion(s). Branches on runtime or test conditions. Relevant themes: runtime driver interaction.
- **CN:** 调用 `run_in_process` 执行测试逻辑。 通过 2 个断言验证行为。 根据运行时或测试条件进行分支。 相关主题：运行时驱动交互。

#### Lines 373-376

```python

    monkeypatch.setenv("TRITON_INSTRUMENTATION_MODE", "consan")
    monkeypatch.setenv("CUDA_LAUNCH_BLOCKING", "1")
    knobs.refresh_knobs()
```
- **EN:** Invokes `monkeypatch.setenv`, `knobs.refresh_knobs` to execute the test logic. Relevant themes: kernel launch orchestration.
- **CN:** 调用 `monkeypatch.setenv`、`knobs.refresh_knobs` 执行测试逻辑。 相关主题：kernel 启动编排。

#### Lines 377-379

```python

    @gluon.jit
    def kernel():
```
- **EN:** Defines the helper function `kernel`. Decorators: `gluon.jit`. Key calls include `multicast_cga_layout`, `ttgl.SwizzledSharedLayout`, `ttgl.allocate_shared_memory`, `mbarrier.allocate_mbarrier`, `mbarrier.init`, `mbarrier.expect`, and 4 more. This scope touches layout transformation reasoning.
- **CN:** 定义辅助函数 `kernel`。 装饰器：`gluon.jit`。 关键调用包括 `multicast_cga_layout`、`ttgl.SwizzledSharedLayout`、`ttgl.allocate_shared_memory`、`mbarrier.allocate_mbarrier`、`mbarrier.init`、`mbarrier.expect` 等另外 4 项。 该作用域涉及布局变换推理。

##### Lines 380-393

```python
        cga_layout: ttgl.constexpr = multicast_cga_layout(ttgl.num_ctas(), 1)
        layout: ttgl.constexpr = ttgl.SwizzledSharedLayout(1, 1, 1, order=[0], cga_layout=cga_layout)
        result = ttgl.allocate_shared_memory(ttgl.int64, [2], layout)
        bars = mbarrier.allocate_mbarrier(batch=2)
        mbarrier.init(bars.index(0), count=1)
        mbarrier.init(bars.index(1), count=1)

        mbarrier.expect(bars.index(0), 16)
        clc.try_cancel(result, bars.index(0))
        mbarrier.expect(bars.index(1), 16)
        clc.try_cancel(result, bars.index(1))

        mbarrier.wait(bars.index(0), 0)
        mbarrier.wait(bars.index(1), 0)
```
- **EN:** Prepares or updates state through `cga_layout`, `layout`, `result`, `bars`. Invokes `multicast_cga_layout`, `ttgl.num_ctas`, `ttgl.SwizzledSharedLayout`, `ttgl.allocate_shared_memory`, `mbarrier.allocate_mbarrier`, `mbarrier.init`, and 4 more to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `cga_layout`、`layout`、`result`、`bars` 准备或更新状态。 调用 `multicast_cga_layout`、`ttgl.num_ctas`、`ttgl.SwizzledSharedLayout`、`ttgl.allocate_shared_memory`、`mbarrier.allocate_mbarrier`、`mbarrier.init` 等另外 4 项 执行测试逻辑。 相关主题：布局变换推理。

#### Lines 394-395

```python

    kernel[(1, )](num_warps=4, num_ctas=2)
```
- **EN:** Carries supporting logic for the surrounding test or helper scope.
- **CN:** 为周围的测试或辅助作用域提供支撑逻辑。

### Lines 396-399

```python


@pytest.mark.skipif(not is_cuda() or torch.cuda.get_device_capability()[0] < 10, reason="Requires blackwell")
def test_clc_result_reuse_after_cluster_barrier(device, run_wrapper, monkeypatch):
```
- **EN:** Defines the test function `test_clc_result_reuse_after_cluster_barrier`. Decorators: `pytest.mark.skipif(not is_cuda() or torch.cuda.get_device_capability()[0] < 10, reason='Requires blackwell')`. Parameters: `device`, `run_wrapper`, `monkeypatch`. Nested definitions in this scope: `kernel`. Key calls include `pytest.mark.skipif`, `monkeypatch.setenv`, `knobs.refresh_knobs`, `torch.empty`, `run_in_process`, `ttgl.SwizzledSharedLayout`, and 14 more. This scope touches PyTorch tensor setup and checks, runtime driver interaction, kernel launch orchestration, layout transformation reasoning.
- **CN:** 定义测试函数 `test_clc_result_reuse_after_cluster_barrier`。 装饰器：`pytest.mark.skipif(not is_cuda() or torch.cuda.get_device_capability()[0] < 10, reason='Requires blackwell')`。 参数：`device`、`run_wrapper`、`monkeypatch`。 该作用域中的嵌套定义：`kernel`。 关键调用包括 `pytest.mark.skipif`、`monkeypatch.setenv`、`knobs.refresh_knobs`、`torch.empty`、`run_in_process`、`ttgl.SwizzledSharedLayout` 等另外 14 项。 该作用域涉及PyTorch 张量准备与校验、运行时驱动交互、kernel 启动编排、布局变换推理。

#### Lines 400-404

```python
    if run_wrapper:
        result = run_in_process(test_clc_result_reuse_after_cluster_barrier, (device, False, monkeypatch))
        assert result.exc is None
        assert result.driver_stderr_output == ""
        return
```
- **EN:** Invokes `run_in_process` to execute the test logic. Validates behavior with 2 assertion(s). Branches on runtime or test conditions. Relevant themes: runtime driver interaction.
- **CN:** 调用 `run_in_process` 执行测试逻辑。 通过 2 个断言验证行为。 根据运行时或测试条件进行分支。 相关主题：运行时驱动交互。

#### Lines 405-408

```python

    monkeypatch.setenv("TRITON_INSTRUMENTATION_MODE", "consan")
    monkeypatch.setenv("CUDA_LAUNCH_BLOCKING", "1")
    knobs.refresh_knobs()
```
- **EN:** Invokes `monkeypatch.setenv`, `knobs.refresh_knobs` to execute the test logic. Relevant themes: kernel launch orchestration.
- **CN:** 调用 `monkeypatch.setenv`、`knobs.refresh_knobs` 执行测试逻辑。 相关主题：kernel 启动编排。

#### Lines 409-411

```python

    @gluon.jit
    def kernel(out):
```
- **EN:** Defines the helper function `kernel`. Decorators: `gluon.jit`. Parameters: `out`. Key calls include `ttgl.SwizzledSharedLayout`, `ttgl.allocate_shared_memory`, `mbarrier.allocate_mbarrier`, `mbarrier.init`, `mbarrier.expect`, `clc.try_cancel`, and 7 more. This scope touches layout transformation reasoning.
- **CN:** 定义辅助函数 `kernel`。 装饰器：`gluon.jit`。 参数：`out`。 关键调用包括 `ttgl.SwizzledSharedLayout`、`ttgl.allocate_shared_memory`、`mbarrier.allocate_mbarrier`、`mbarrier.init`、`mbarrier.expect`、`clc.try_cancel` 等另外 7 项。 该作用域涉及布局变换推理。

##### Lines 412-428

```python
        cga_layout: ttgl.constexpr = [[0]]
        layout: ttgl.constexpr = ttgl.SwizzledSharedLayout(1, 1, 1, order=[0], cga_layout=cga_layout)
        clc_result = ttgl.allocate_shared_memory(ttgl.int64, [2], layout)
        clc_bar = mbarrier.allocate_mbarrier()
        mbarrier.init(clc_bar, count=1)

        mbarrier.expect(clc_bar, 16)
        clc.try_cancel(clc_result, clc_bar)
        mbarrier.wait(clc_bar, 0)
        first = clc.load_result(clc_result)
        ttgl.barrier(cluster=True)

        mbarrier.expect(clc_bar, 16)
        clc.try_cancel(clc_result, clc_bar)
        mbarrier.wait(clc_bar, 1)
        second = clc.load_result(clc_result)
        ttgl.store(out + ttgl.program_id(0), first.is_canceled() | second.is_canceled())
```
- **EN:** Prepares or updates state through `cga_layout`, `layout`, `clc_result`, `clc_bar`, `first`, `second`. Invokes `ttgl.SwizzledSharedLayout`, `ttgl.allocate_shared_memory`, `mbarrier.allocate_mbarrier`, `mbarrier.init`, `mbarrier.expect`, `clc.try_cancel`, and 7 more to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `cga_layout`、`layout`、`clc_result`、`clc_bar`、`first`、`second` 准备或更新状态。 调用 `ttgl.SwizzledSharedLayout`、`ttgl.allocate_shared_memory`、`mbarrier.allocate_mbarrier`、`mbarrier.init`、`mbarrier.expect`、`clc.try_cancel` 等另外 7 项 执行测试逻辑。 相关主题：布局变换推理。

#### Lines 429-431

```python

    output = torch.empty((1, ), device=device, dtype=torch.bool)
    kernel[(1, )](output, num_warps=4, num_ctas=2)
```
- **EN:** Prepares or updates state through `output`. Invokes `torch.empty` to execute the test logic. Relevant themes: PyTorch tensor setup and checks.
- **CN:** 通过 `output` 准备或更新状态。 调用 `torch.empty` 执行测试逻辑。 相关主题：PyTorch 张量准备与校验。

### Lines 432-435

```python


@pytest.mark.skipif(not is_cuda() or torch.cuda.get_device_capability()[0] < 9, reason="Requires hopper or newer")
def test_async_tma_multicast_kernel_reuse(device, run_wrapper, monkeypatch, num_ctas):
```
- **EN:** Defines the test function `test_async_tma_multicast_kernel_reuse`. Decorators: `pytest.mark.skipif(not is_cuda() or torch.cuda.get_device_capability()[0] < 9, reason='Requires hopper or newer')`. Parameters: `device`, `run_wrapper`, `monkeypatch`, `num_ctas`. Nested definitions in this scope: `kernel`. Key calls include `pytest.mark.skipif`, `monkeypatch.setenv`, `knobs.refresh_knobs`, `torch.randn`, `torch.empty`, `ttgl.NVMMASharedLayout`, and 21 more. This scope touches PyTorch tensor setup and checks, tensor/descriptor metadata, runtime driver interaction, kernel launch orchestration.
- **CN:** 定义测试函数 `test_async_tma_multicast_kernel_reuse`。 装饰器：`pytest.mark.skipif(not is_cuda() or torch.cuda.get_device_capability()[0] < 9, reason='Requires hopper or newer')`。 参数：`device`、`run_wrapper`、`monkeypatch`、`num_ctas`。 该作用域中的嵌套定义：`kernel`。 关键调用包括 `pytest.mark.skipif`、`monkeypatch.setenv`、`knobs.refresh_knobs`、`torch.randn`、`torch.empty`、`ttgl.NVMMASharedLayout` 等另外 21 项。 该作用域涉及PyTorch 张量准备与校验、张量/描述符元数据、运行时驱动交互、kernel 启动编排。

#### Lines 436-437

```python
    if num_ctas == 1:
        pytest.skip("Need at least 2 CTAs for multicast in this test")
```
- **EN:** Invokes `pytest.skip` to execute the test logic. Branches on runtime or test conditions.
- **CN:** 调用 `pytest.skip` 执行测试逻辑。 根据运行时或测试条件进行分支。

#### Lines 438-442

```python
    if run_wrapper:
        result = run_in_process(test_async_tma_multicast_kernel_reuse, (device, False, monkeypatch, num_ctas))
        assert result.exc is None
        assert result.driver_stderr_output == ""
        return
```
- **EN:** Invokes `run_in_process` to execute the test logic. Validates behavior with 2 assertion(s). Branches on runtime or test conditions. Relevant themes: runtime driver interaction.
- **CN:** 调用 `run_in_process` 执行测试逻辑。 通过 2 个断言验证行为。 根据运行时或测试条件进行分支。 相关主题：运行时驱动交互。

#### Lines 443-446

```python

    monkeypatch.setenv("TRITON_INSTRUMENTATION_MODE", "consan")
    monkeypatch.setenv("CUDA_LAUNCH_BLOCKING", "1")
    knobs.refresh_knobs()
```
- **EN:** Invokes `monkeypatch.setenv`, `knobs.refresh_knobs` to execute the test logic. Relevant themes: kernel launch orchestration.
- **CN:** 调用 `monkeypatch.setenv`、`knobs.refresh_knobs` 执行测试逻辑。 相关主题：kernel 启动编排。

#### Lines 447-449

```python

    @gluon.jit
    def kernel(input_desc, out):
```
- **EN:** Defines the helper function `kernel`. Decorators: `gluon.jit`. Parameters: `input_desc`, `out`. Key calls include `multicast_cga_layout`, `ttgl.BlockedLayout`, `ttgl.allocate_shared_memory`, `mbarrier.allocate_mbarrier`, `mbarrier.init`, `ttgl.static_range`, and 10 more. This scope touches layout transformation reasoning.
- **CN:** 定义辅助函数 `kernel`。 装饰器：`gluon.jit`。 参数：`input_desc`、`out`。 关键调用包括 `multicast_cga_layout`、`ttgl.BlockedLayout`、`ttgl.allocate_shared_memory`、`mbarrier.allocate_mbarrier`、`mbarrier.init`、`ttgl.static_range` 等另外 10 项。 该作用域涉及布局变换推理。

##### Lines 450-455

```python
        cga_layout: ttgl.constexpr = multicast_cga_layout(ttgl.num_ctas(), 2)
        blocked_layout: ttgl.constexpr = ttgl.BlockedLayout(size_per_thread=[1, 1], threads_per_warp=[32, 1],
                                                            warps_per_cta=[4, 1], order=[0, 1], cga_layout=cga_layout)
        smem = ttgl.allocate_shared_memory(ttgl.float16, [XBLOCK, XBLOCK], input_desc.layout)
        bar = mbarrier.allocate_mbarrier()
        mbarrier.init(bar, count=1)
```
- **EN:** Prepares or updates state through `cga_layout`, `blocked_layout`, `smem`, `bar`. Invokes `multicast_cga_layout`, `ttgl.num_ctas`, `ttgl.BlockedLayout`, `ttgl.allocate_shared_memory`, `mbarrier.allocate_mbarrier`, `mbarrier.init` to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `cga_layout`、`blocked_layout`、`smem`、`bar` 准备或更新状态。 调用 `multicast_cga_layout`、`ttgl.num_ctas`、`ttgl.BlockedLayout`、`ttgl.allocate_shared_memory`、`mbarrier.allocate_mbarrier`、`mbarrier.init` 执行测试逻辑。 相关主题：布局变换推理。

##### Lines 456-461

```python
        for phase in ttgl.static_range(2):
            mbarrier.expect(bar, input_desc.nbytes_per_cta)
            ttgl.barrier(cluster=True)
            tma.async_load(input_desc, [0, 0], bar, smem, multicast=True)
            mbarrier.wait(bar, phase % 2, deps=[smem])
            ttgl.barrier(cluster=True)
```
- **EN:** Invokes `ttgl.static_range`, `mbarrier.expect`, `ttgl.barrier`, `tma.async_load`, `mbarrier.wait` to execute the test logic. Iterates across cases or data tiles.
- **CN:** 调用 `ttgl.static_range`、`mbarrier.expect`、`ttgl.barrier`、`tma.async_load`、`mbarrier.wait` 执行测试逻辑。 通过循环覆盖多个用例或数据分块。

##### Lines 462-468

```python
        val = smem.load(blocked_layout)
        mbarrier.invalidate(bar)

        out_m = ttgl.arange(0, XBLOCK, ttgl.SliceLayout(1, blocked_layout))[:, None]
        out_n = ttgl.arange(0, XBLOCK, ttgl.SliceLayout(0, blocked_layout))[None, :]
        out_ptr = out + out_m * XBLOCK + out_n
        ttgl.store(out_ptr, val)
```
- **EN:** Prepares or updates state through `val`, `out_m`, `out_n`, `out_ptr`. Invokes `smem.load`, `mbarrier.invalidate`, `ttgl.arange`, `ttgl.SliceLayout`, `ttgl.store` to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `val`、`out_m`、`out_n`、`out_ptr` 准备或更新状态。 调用 `smem.load`、`mbarrier.invalidate`、`ttgl.arange`、`ttgl.SliceLayout`、`ttgl.store` 执行测试逻辑。 相关主题：布局变换推理。

#### Lines 469-475

```python

    input = torch.randn((XBLOCK.value, XBLOCK.value), device=device, dtype=torch.float16)
    output = torch.empty((XBLOCK.value, XBLOCK.value), device=device, dtype=torch.float16)
    shared_layout = ttgl.NVMMASharedLayout(swizzle_byte_width=128, element_bitwidth=16, rank=2,
                                           cga_layout=multicast_cga_layout(num_ctas, 2))
    input_desc = gluon.nvidia.hopper.TensorDescriptor.from_tensor(input, [XBLOCK.value, XBLOCK.value], shared_layout)
    kernel[(1, )](input_desc, output, num_warps=4, num_ctas=num_ctas)
```
- **EN:** Prepares or updates state through `input`, `output`, `shared_layout`, `input_desc`. Invokes `torch.randn`, `torch.empty`, `ttgl.NVMMASharedLayout`, `multicast_cga_layout`, `gluon.nvidia.hopper.TensorDescriptor.from_tensor` to execute the test logic. Relevant themes: PyTorch tensor setup and checks, tensor/descriptor metadata, layout transformation reasoning, random-data generation.
- **CN:** 通过 `input`、`output`、`shared_layout`、`input_desc` 准备或更新状态。 调用 `torch.randn`、`torch.empty`、`ttgl.NVMMASharedLayout`、`multicast_cga_layout`、`gluon.nvidia.hopper.TensorDescriptor.from_tensor` 执行测试逻辑。 相关主题：PyTorch 张量准备与校验、张量/描述符元数据、布局变换推理、随机数据生成。

### Lines 476-479

```python


@pytest.mark.skipif(not is_cuda() or torch.cuda.get_device_capability()[0] < 9, reason="Requires hopper or newer")
def test_async_tma_multicast_kernel_local_store_race(device, run_wrapper, monkeypatch, num_ctas):
```
- **EN:** Defines the test function `test_async_tma_multicast_kernel_local_store_race`. Decorators: `pytest.mark.skipif(not is_cuda() or torch.cuda.get_device_capability()[0] < 9, reason='Requires hopper or newer')`. Parameters: `device`, `run_wrapper`, `monkeypatch`, `num_ctas`. Nested definitions in this scope: `kernel`. Key calls include `pytest.mark.skipif`, `monkeypatch.setenv`, `knobs.refresh_knobs`, `torch.randn`, `torch.empty`, `ttgl.NVMMASharedLayout`, and 23 more. This scope touches PyTorch tensor setup and checks, tensor/descriptor metadata, runtime driver interaction, kernel launch orchestration.
- **CN:** 定义测试函数 `test_async_tma_multicast_kernel_local_store_race`。 装饰器：`pytest.mark.skipif(not is_cuda() or torch.cuda.get_device_capability()[0] < 9, reason='Requires hopper or newer')`。 参数：`device`、`run_wrapper`、`monkeypatch`、`num_ctas`。 该作用域中的嵌套定义：`kernel`。 关键调用包括 `pytest.mark.skipif`、`monkeypatch.setenv`、`knobs.refresh_knobs`、`torch.randn`、`torch.empty`、`ttgl.NVMMASharedLayout` 等另外 23 项。 该作用域涉及PyTorch 张量准备与校验、张量/描述符元数据、运行时驱动交互、kernel 启动编排。

#### Lines 480-481

```python
    if num_ctas == 1:
        pytest.skip("Need at least 2 CTAs for multicast in this test")
```
- **EN:** Invokes `pytest.skip` to execute the test logic. Branches on runtime or test conditions.
- **CN:** 调用 `pytest.skip` 执行测试逻辑。 根据运行时或测试条件进行分支。

#### Lines 482-487

```python
    if run_wrapper:
        result = run_in_process(test_async_tma_multicast_kernel_local_store_race,
                                (device, False, monkeypatch, num_ctas))
        assert_expected_cuda_failure(result.exc)
        assert "Buffer being accessed has outstanding writes" in result.driver_stderr_output
        return
```
- **EN:** Invokes `run_in_process`, `assert_expected_cuda_failure` to execute the test logic. Validates behavior with 1 assertion(s). Branches on runtime or test conditions. Relevant themes: runtime driver interaction.
- **CN:** 调用 `run_in_process`、`assert_expected_cuda_failure` 执行测试逻辑。 通过 1 个断言验证行为。 根据运行时或测试条件进行分支。 相关主题：运行时驱动交互。

#### Lines 488-491

```python

    monkeypatch.setenv("TRITON_INSTRUMENTATION_MODE", "consan")
    monkeypatch.setenv("CUDA_LAUNCH_BLOCKING", "1")
    knobs.refresh_knobs()
```
- **EN:** Invokes `monkeypatch.setenv`, `knobs.refresh_knobs` to execute the test logic. Relevant themes: kernel launch orchestration.
- **CN:** 调用 `monkeypatch.setenv`、`knobs.refresh_knobs` 执行测试逻辑。 相关主题：kernel 启动编排。

#### Lines 492-494

```python

    @gluon.jit
    def kernel(input_desc, out):
```
- **EN:** Defines the helper function `kernel`. Decorators: `gluon.jit`. Parameters: `input_desc`, `out`. Key calls include `multicast_cga_layout`, `ttgl.BlockedLayout`, `ttgl.allocate_shared_memory`, `mbarrier.allocate_mbarrier`, `mbarrier.init`, `mbarrier.expect`, and 11 more. This scope touches layout transformation reasoning.
- **CN:** 定义辅助函数 `kernel`。 装饰器：`gluon.jit`。 参数：`input_desc`、`out`。 关键调用包括 `multicast_cga_layout`、`ttgl.BlockedLayout`、`ttgl.allocate_shared_memory`、`mbarrier.allocate_mbarrier`、`mbarrier.init`、`mbarrier.expect` 等另外 11 项。 该作用域涉及布局变换推理。

##### Lines 495-513

```python
        cga_layout: ttgl.constexpr = multicast_cga_layout(ttgl.num_ctas(), 2)
        blocked_layout: ttgl.constexpr = ttgl.BlockedLayout(size_per_thread=[1, 1], threads_per_warp=[32, 1],
                                                            warps_per_cta=[4, 1], order=[0, 1], cga_layout=cga_layout)
        smem = ttgl.allocate_shared_memory(ttgl.float16, [XBLOCK, XBLOCK], input_desc.layout)

        bar = mbarrier.allocate_mbarrier()
        mbarrier.init(bar, count=1)
        mbarrier.expect(bar, input_desc.nbytes_per_cta)
        tma.async_load(input_desc, [0, 0], bar, smem, multicast=True)
        ttgl.barrier(cluster=True)
        smem.store(ttgl.full([XBLOCK, XBLOCK], 1, ttgl.float16, blocked_layout))
        mbarrier.wait(bar, 0, deps=[smem])
        val = smem.load(blocked_layout)
        mbarrier.invalidate(bar)

        out_m = ttgl.arange(0, XBLOCK, ttgl.SliceLayout(1, blocked_layout))[:, None]
        out_n = ttgl.arange(0, XBLOCK, ttgl.SliceLayout(0, blocked_layout))[None, :]
        out_ptr = out + out_m * XBLOCK + out_n
        ttgl.store(out_ptr, val)
```
- **EN:** Prepares or updates state through `cga_layout`, `blocked_layout`, `smem`, `bar`, `val`, `out_m`, `out_n`, `out_ptr`. Invokes `multicast_cga_layout`, `ttgl.num_ctas`, `ttgl.BlockedLayout`, `ttgl.allocate_shared_memory`, `mbarrier.allocate_mbarrier`, `mbarrier.init`, and 11 more to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `cga_layout`、`blocked_layout`、`smem`、`bar`、`val`、`out_m`、`out_n`、`out_ptr` 准备或更新状态。 调用 `multicast_cga_layout`、`ttgl.num_ctas`、`ttgl.BlockedLayout`、`ttgl.allocate_shared_memory`、`mbarrier.allocate_mbarrier`、`mbarrier.init` 等另外 11 项 执行测试逻辑。 相关主题：布局变换推理。

#### Lines 514-520

```python

    input = torch.randn((XBLOCK.value, XBLOCK.value), device=device, dtype=torch.float16)
    output = torch.empty((XBLOCK.value, XBLOCK.value), device=device, dtype=torch.float16)
    shared_layout = ttgl.NVMMASharedLayout(swizzle_byte_width=128, element_bitwidth=16, rank=2,
                                           cga_layout=multicast_cga_layout(num_ctas, 2))
    input_desc = gluon.nvidia.hopper.TensorDescriptor.from_tensor(input, [XBLOCK.value, XBLOCK.value], shared_layout)
    kernel[(1, )](input_desc, output, num_warps=4, num_ctas=num_ctas)
```
- **EN:** Prepares or updates state through `input`, `output`, `shared_layout`, `input_desc`. Invokes `torch.randn`, `torch.empty`, `ttgl.NVMMASharedLayout`, `multicast_cga_layout`, `gluon.nvidia.hopper.TensorDescriptor.from_tensor` to execute the test logic. Relevant themes: PyTorch tensor setup and checks, tensor/descriptor metadata, layout transformation reasoning, random-data generation.
- **CN:** 通过 `input`、`output`、`shared_layout`、`input_desc` 准备或更新状态。 调用 `torch.randn`、`torch.empty`、`ttgl.NVMMASharedLayout`、`multicast_cga_layout`、`gluon.nvidia.hopper.TensorDescriptor.from_tensor` 执行测试逻辑。 相关主题：PyTorch 张量准备与校验、张量/描述符元数据、布局变换推理、随机数据生成。

### Lines 521-524

```python


@pytest.mark.skipif(not is_cuda() or torch.cuda.get_device_capability()[0] < 9, reason="Requires hopper or newer")
def test_cluster_barrier_does_not_publish_later_read(device, run_wrapper, monkeypatch):
```
- **EN:** Defines the test function `test_cluster_barrier_does_not_publish_later_read`. Decorators: `pytest.mark.skipif(not is_cuda() or torch.cuda.get_device_capability()[0] < 9, reason='Requires hopper or newer')`. Parameters: `device`, `run_wrapper`, `monkeypatch`. Nested definitions in this scope: `kernel`. Key calls include `pytest.mark.skipif`, `monkeypatch.setenv`, `knobs.refresh_knobs`, `torch.randn`, `ttgl.NVMMASharedLayout`, `gluon.nvidia.hopper.TensorDescriptor.from_tensor`, and 17 more. This scope touches PyTorch tensor setup and checks, tensor/descriptor metadata, runtime driver interaction, kernel launch orchestration.
- **CN:** 定义测试函数 `test_cluster_barrier_does_not_publish_later_read`。 装饰器：`pytest.mark.skipif(not is_cuda() or torch.cuda.get_device_capability()[0] < 9, reason='Requires hopper or newer')`。 参数：`device`、`run_wrapper`、`monkeypatch`。 该作用域中的嵌套定义：`kernel`。 关键调用包括 `pytest.mark.skipif`、`monkeypatch.setenv`、`knobs.refresh_knobs`、`torch.randn`、`ttgl.NVMMASharedLayout`、`gluon.nvidia.hopper.TensorDescriptor.from_tensor` 等另外 17 项。 该作用域涉及PyTorch 张量准备与校验、张量/描述符元数据、运行时驱动交互、kernel 启动编排。

#### Lines 525-529

```python
    if run_wrapper:
        result = run_in_process(test_cluster_barrier_does_not_publish_later_read, (device, False, monkeypatch))
        assert_expected_cuda_failure(result.exc)
        assert "Buffer being accessed has outstanding reads" in result.driver_stderr_output
        return
```
- **EN:** Invokes `run_in_process`, `assert_expected_cuda_failure` to execute the test logic. Validates behavior with 1 assertion(s). Branches on runtime or test conditions. Relevant themes: runtime driver interaction.
- **CN:** 调用 `run_in_process`、`assert_expected_cuda_failure` 执行测试逻辑。 通过 1 个断言验证行为。 根据运行时或测试条件进行分支。 相关主题：运行时驱动交互。

#### Lines 530-533

```python

    monkeypatch.setenv("TRITON_INSTRUMENTATION_MODE", "consan")
    monkeypatch.setenv("CUDA_LAUNCH_BLOCKING", "1")
    knobs.refresh_knobs()
```
- **EN:** Invokes `monkeypatch.setenv`, `knobs.refresh_knobs` to execute the test logic. Relevant themes: kernel launch orchestration.
- **CN:** 调用 `monkeypatch.setenv`、`knobs.refresh_knobs` 执行测试逻辑。 相关主题：kernel 启动编排。

#### Lines 534-536

```python

    @gluon.jit
    def kernel(input_desc):
```
- **EN:** Defines the helper function `kernel`. Decorators: `gluon.jit`. Parameters: `input_desc`. Key calls include `multicast_cga_layout`, `ttgl.BlockedLayout`, `ttgl.allocate_shared_memory`, `mbarrier.allocate_mbarrier`, `mbarrier.init`, `ttgl.barrier`, and 7 more. This scope touches layout transformation reasoning.
- **CN:** 定义辅助函数 `kernel`。 装饰器：`gluon.jit`。 参数：`input_desc`。 关键调用包括 `multicast_cga_layout`、`ttgl.BlockedLayout`、`ttgl.allocate_shared_memory`、`mbarrier.allocate_mbarrier`、`mbarrier.init`、`ttgl.barrier` 等另外 7 项。 该作用域涉及布局变换推理。

##### Lines 537-546

```python
        cga_layout: ttgl.constexpr = multicast_cga_layout(ttgl.num_ctas(), 2)
        blocked_layout: ttgl.constexpr = ttgl.BlockedLayout(size_per_thread=[1, 1], threads_per_warp=[32, 1],
                                                            warps_per_cta=[4, 1], order=[0, 1], cga_layout=cga_layout)
        smem = ttgl.allocate_shared_memory(ttgl.float16, [XBLOCK, XBLOCK], input_desc.layout)
        sink = ttgl.allocate_shared_memory(ttgl.float16, [2, XBLOCK, XBLOCK], input_desc.layout)
        bar = mbarrier.allocate_mbarrier()
        mbarrier.init(bar, count=1)

        ttgl.barrier(cluster=True)
        val = smem.load(blocked_layout)
```
- **EN:** Prepares or updates state through `cga_layout`, `blocked_layout`, `smem`, `sink`, `bar`, `val`. Invokes `multicast_cga_layout`, `ttgl.num_ctas`, `ttgl.BlockedLayout`, `ttgl.allocate_shared_memory`, `mbarrier.allocate_mbarrier`, `mbarrier.init`, and 2 more to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `cga_layout`、`blocked_layout`、`smem`、`sink`、`bar`、`val` 准备或更新状态。 调用 `multicast_cga_layout`、`ttgl.num_ctas`、`ttgl.BlockedLayout`、`ttgl.allocate_shared_memory`、`mbarrier.allocate_mbarrier`、`mbarrier.init` 等另外 2 项 执行测试逻辑。 相关主题：布局变换推理。

##### Lines 547-549

```python
        # Keep the post-barrier read live long enough to exercise delayed publication from another CTA.
        for i in ttgl.static_range(2):
            sink.index(i).store(val)
```
- **EN:** Invokes `ttgl.static_range`, `sink.index` to execute the test logic. Iterates across cases or data tiles.
- **CN:** 调用 `ttgl.static_range`、`sink.index` 执行测试逻辑。 通过循环覆盖多个用例或数据分块。

##### Lines 550-553

```python

        mbarrier.expect(bar, input_desc.nbytes_per_cta)
        tma.async_load(input_desc, [0, 0], bar, smem, multicast=True)
        mbarrier.wait(bar, 0, deps=[smem])
```
- **EN:** Invokes `mbarrier.expect`, `tma.async_load`, `mbarrier.wait` to execute the test logic.
- **CN:** 调用 `mbarrier.expect`、`tma.async_load`、`mbarrier.wait` 执行测试逻辑。

#### Lines 554-559

```python

    input = torch.randn((XBLOCK.value, XBLOCK.value), device=device, dtype=torch.float16)
    shared_layout = ttgl.NVMMASharedLayout(swizzle_byte_width=128, element_bitwidth=16, rank=2,
                                           cga_layout=multicast_cga_layout(4, 2))
    input_desc = gluon.nvidia.hopper.TensorDescriptor.from_tensor(input, [XBLOCK.value, XBLOCK.value], shared_layout)
    kernel[(1, )](input_desc, num_warps=4, num_ctas=4)
```
- **EN:** Prepares or updates state through `input`, `shared_layout`, `input_desc`. Invokes `torch.randn`, `ttgl.NVMMASharedLayout`, `multicast_cga_layout`, `gluon.nvidia.hopper.TensorDescriptor.from_tensor` to execute the test logic. Relevant themes: PyTorch tensor setup and checks, tensor/descriptor metadata, layout transformation reasoning, random-data generation.
- **CN:** 通过 `input`、`shared_layout`、`input_desc` 准备或更新状态。 调用 `torch.randn`、`ttgl.NVMMASharedLayout`、`multicast_cga_layout`、`gluon.nvidia.hopper.TensorDescriptor.from_tensor` 执行测试逻辑。 相关主题：PyTorch 张量准备与校验、张量/描述符元数据、布局变换推理、随机数据生成。

### Lines 560-564

```python


@pytest.mark.skipif(not is_cuda() or torch.cuda.get_device_capability()[0] < 9, reason="Requires hopper or newer")
@pytest.mark.parametrize("FAILURE", [True, False])
def test_async_tma_kernel_2bufs_1bar(FAILURE, device, run_wrapper, monkeypatch, num_ctas):
```
- **EN:** Defines the test function `test_async_tma_kernel_2bufs_1bar`. Decorators: `pytest.mark.skipif(not is_cuda() or torch.cuda.get_device_capability()[0] < 9, reason='Requires hopper or newer')`, `pytest.mark.parametrize('FAILURE', [True, False])`. Parameters: `FAILURE`, `device`, `run_wrapper`, `monkeypatch`, `num_ctas`. Nested definitions in this scope: `kernel`. Key calls include `pytest.mark.skipif`, `pytest.mark.parametrize`, `monkeypatch.setenv`, `knobs.refresh_knobs`, `torch.randn`, `torch.empty`, and 21 more. This scope touches pytest parametrization, PyTorch tensor setup and checks, tensor/descriptor metadata, runtime driver interaction.
- **CN:** 定义测试函数 `test_async_tma_kernel_2bufs_1bar`。 装饰器：`pytest.mark.skipif(not is_cuda() or torch.cuda.get_device_capability()[0] < 9, reason='Requires hopper or newer')`、`pytest.mark.parametrize('FAILURE', [True, False])`。 参数：`FAILURE`、`device`、`run_wrapper`、`monkeypatch`、`num_ctas`。 该作用域中的嵌套定义：`kernel`。 关键调用包括 `pytest.mark.skipif`、`pytest.mark.parametrize`、`monkeypatch.setenv`、`knobs.refresh_knobs`、`torch.randn`、`torch.empty` 等另外 21 项。 该作用域涉及pytest 参数化、PyTorch 张量准备与校验、张量/描述符元数据、运行时驱动交互。

#### Lines 565-573

```python
    if run_wrapper:
        result = run_in_process(test_async_tma_kernel_2bufs_1bar, (FAILURE, device, False, monkeypatch, num_ctas))
        if FAILURE:
            assert_expected_cuda_failure(result.exc)
            assert "Buffer being accessed has outstanding writes" in result.driver_stderr_output
        else:
            assert result.exc is None
            assert result.driver_stderr_output == ""
        return
```
- **EN:** Invokes `run_in_process`, `assert_expected_cuda_failure` to execute the test logic. Validates behavior with 3 assertion(s). Branches on runtime or test conditions. Relevant themes: runtime driver interaction.
- **CN:** 调用 `run_in_process`、`assert_expected_cuda_failure` 执行测试逻辑。 通过 3 个断言验证行为。 根据运行时或测试条件进行分支。 相关主题：运行时驱动交互。

#### Lines 574-577

```python

    monkeypatch.setenv("TRITON_INSTRUMENTATION_MODE", "consan")
    monkeypatch.setenv("CUDA_LAUNCH_BLOCKING", "1")
    knobs.refresh_knobs()
```
- **EN:** Invokes `monkeypatch.setenv`, `knobs.refresh_knobs` to execute the test logic. Relevant themes: kernel launch orchestration.
- **CN:** 调用 `monkeypatch.setenv`、`knobs.refresh_knobs` 执行测试逻辑。 相关主题：kernel 启动编排。

#### Lines 578-580

```python

    @gluon.jit
    def kernel(a_desc, b_desc, out, FAILURE: ttgl.constexpr):
```
- **EN:** Defines the helper function `kernel`. Decorators: `gluon.jit`. Parameters: `a_desc`, `b_desc`, `out`, `FAILURE`. Key calls include `default_cga_layout`, `ttgl.BlockedLayout`, `ttgl.allocate_shared_memory`, `mbarrier.allocate_mbarrier`, `mbarrier.init`, `mbarrier.expect`, and 9 more. This scope touches layout transformation reasoning.
- **CN:** 定义辅助函数 `kernel`。 装饰器：`gluon.jit`。 参数：`a_desc`、`b_desc`、`out`、`FAILURE`。 关键调用包括 `default_cga_layout`、`ttgl.BlockedLayout`、`ttgl.allocate_shared_memory`、`mbarrier.allocate_mbarrier`、`mbarrier.init`、`mbarrier.expect` 等另外 9 项。 该作用域涉及布局变换推理。

##### Lines 581-601

```python
        block_m: ttgl.constexpr = XBLOCK * ttgl.num_ctas()
        cga_layout: ttgl.constexpr = default_cga_layout(ttgl.num_ctas(), 2)
        blocked_layout: ttgl.constexpr = ttgl.BlockedLayout(size_per_thread=[1, 1], threads_per_warp=[32, 1],
                                                            warps_per_cta=[4, 1], order=[0, 1], cga_layout=cga_layout)
        a_smem = ttgl.allocate_shared_memory(ttgl.float16, [block_m, XBLOCK], a_desc.layout)
        b_smem = ttgl.allocate_shared_memory(ttgl.float16, [block_m, XBLOCK], b_desc.layout)
        bar = mbarrier.allocate_mbarrier()
        mbarrier.init(bar, count=1)
        mbarrier.expect(bar, a_desc.nbytes_per_cta + b_desc.nbytes_per_cta)
        tma.async_load(a_desc, [0, 0], bar, a_smem)
        tma.async_load(b_desc, [0, 0], bar, b_smem)
        mbarrier.wait(bar, 0, pred=(not FAILURE), deps=[a_smem, b_smem])
        val = a_smem.load(blocked_layout)
        val = val + b_smem.load(blocked_layout)
        mbarrier.wait(bar, 0, pred=FAILURE, deps=[a_smem, b_smem])
        mbarrier.invalidate(bar)

        out_m = ttgl.arange(0, block_m, ttgl.SliceLayout(1, blocked_layout))[:, None]
        out_n = ttgl.arange(0, XBLOCK, ttgl.SliceLayout(0, blocked_layout))[None, :]
        out_ptr = out + out_m * XBLOCK + out_n
        ttgl.store(out_ptr, val)
```
- **EN:** Prepares or updates state through `block_m`, `cga_layout`, `blocked_layout`, `a_smem`, `b_smem`, `bar`, `val`, `out_m`, and 2 more. Invokes `ttgl.num_ctas`, `default_cga_layout`, `ttgl.BlockedLayout`, `ttgl.allocate_shared_memory`, `mbarrier.allocate_mbarrier`, `mbarrier.init`, and 9 more to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `block_m`、`cga_layout`、`blocked_layout`、`a_smem`、`b_smem`、`bar`、`val`、`out_m` 等另外 2 项 准备或更新状态。 调用 `ttgl.num_ctas`、`default_cga_layout`、`ttgl.BlockedLayout`、`ttgl.allocate_shared_memory`、`mbarrier.allocate_mbarrier`、`mbarrier.init` 等另外 9 项 执行测试逻辑。 相关主题：布局变换推理。

#### Lines 602-611

```python

    block_m = XBLOCK.value * num_ctas
    a = torch.randn((block_m, XBLOCK.value), device=device, dtype=torch.float16)
    b = torch.randn((block_m, XBLOCK.value), device=device, dtype=torch.float16)
    output = torch.empty((block_m, XBLOCK.value), device=device, dtype=torch.float16)
    shared_layout = ttgl.NVMMASharedLayout(swizzle_byte_width=128, element_bitwidth=16, rank=2,
                                           cga_layout=default_cga_layout(num_ctas, 2))
    a_desc = gluon.nvidia.hopper.TensorDescriptor.from_tensor(a, [block_m, XBLOCK.value], shared_layout)
    b_desc = gluon.nvidia.hopper.TensorDescriptor.from_tensor(b, [block_m, XBLOCK.value], shared_layout)
    kernel[(1, )](a_desc, b_desc, output, FAILURE=FAILURE, num_warps=4, num_ctas=num_ctas)
```
- **EN:** Prepares or updates state through `block_m`, `a`, `b`, `output`, `shared_layout`, `a_desc`, `b_desc`. Invokes `torch.randn`, `torch.empty`, `ttgl.NVMMASharedLayout`, `default_cga_layout`, `gluon.nvidia.hopper.TensorDescriptor.from_tensor` to execute the test logic. Relevant themes: PyTorch tensor setup and checks, tensor/descriptor metadata, layout transformation reasoning, random-data generation.
- **CN:** 通过 `block_m`、`a`、`b`、`output`、`shared_layout`、`a_desc`、`b_desc` 准备或更新状态。 调用 `torch.randn`、`torch.empty`、`ttgl.NVMMASharedLayout`、`default_cga_layout`、`gluon.nvidia.hopper.TensorDescriptor.from_tensor` 执行测试逻辑。 相关主题：PyTorch 张量准备与校验、张量/描述符元数据、布局变换推理、随机数据生成。

### Lines 612-616

```python


@pytest.mark.skipif(not is_cuda() or torch.cuda.get_device_capability()[0] < 9, reason="Requires hopper or newer")
@pytest.mark.parametrize("EXPECT_DELTA", [-16, 16], ids=["under", "over"])
def test_async_tma_expect_bytes_mismatch(EXPECT_DELTA, device, run_wrapper, monkeypatch, num_ctas):
```
- **EN:** Defines the test function `test_async_tma_expect_bytes_mismatch`. Decorators: `pytest.mark.skipif(not is_cuda() or torch.cuda.get_device_capability()[0] < 9, reason='Requires hopper or newer')`, `pytest.mark.parametrize('EXPECT_DELTA', [-16, 16], ids=['under', 'over'])`. Parameters: `EXPECT_DELTA`, `device`, `run_wrapper`, `monkeypatch`, `num_ctas`. Nested definitions in this scope: `kernel`. Key calls include `pytest.mark.skipif`, `pytest.mark.parametrize`, `monkeypatch.setenv`, `knobs.refresh_knobs`, `torch.randn`, `torch.empty`, and 20 more. This scope touches pytest parametrization, PyTorch tensor setup and checks, tensor/descriptor metadata, runtime driver interaction.
- **CN:** 定义测试函数 `test_async_tma_expect_bytes_mismatch`。 装饰器：`pytest.mark.skipif(not is_cuda() or torch.cuda.get_device_capability()[0] < 9, reason='Requires hopper or newer')`、`pytest.mark.parametrize('EXPECT_DELTA', [-16, 16], ids=['under', 'over'])`。 参数：`EXPECT_DELTA`、`device`、`run_wrapper`、`monkeypatch`、`num_ctas`。 该作用域中的嵌套定义：`kernel`。 关键调用包括 `pytest.mark.skipif`、`pytest.mark.parametrize`、`monkeypatch.setenv`、`knobs.refresh_knobs`、`torch.randn`、`torch.empty` 等另外 20 项。 该作用域涉及pytest 参数化、PyTorch 张量准备与校验、张量/描述符元数据、运行时驱动交互。

#### Lines 617-622

```python
    if run_wrapper:
        result = run_in_process(test_async_tma_expect_bytes_mismatch,
                                (EXPECT_DELTA, device, False, monkeypatch, num_ctas))
        assert_expected_cuda_failure(result.exc)
        assert "Deadlock detected" in result.driver_stderr_output
        return
```
- **EN:** Invokes `run_in_process`, `assert_expected_cuda_failure` to execute the test logic. Validates behavior with 1 assertion(s). Branches on runtime or test conditions. Relevant themes: runtime driver interaction.
- **CN:** 调用 `run_in_process`、`assert_expected_cuda_failure` 执行测试逻辑。 通过 1 个断言验证行为。 根据运行时或测试条件进行分支。 相关主题：运行时驱动交互。

#### Lines 623-626

```python

    monkeypatch.setenv("TRITON_INSTRUMENTATION_MODE", "consan")
    monkeypatch.setenv("CUDA_LAUNCH_BLOCKING", "1")
    knobs.refresh_knobs()
```
- **EN:** Invokes `monkeypatch.setenv`, `knobs.refresh_knobs` to execute the test logic. Relevant themes: kernel launch orchestration.
- **CN:** 调用 `monkeypatch.setenv`、`knobs.refresh_knobs` 执行测试逻辑。 相关主题：kernel 启动编排。

#### Lines 627-629

```python

    @gluon.jit
    def kernel(input_desc, out, EXPECT_DELTA: ttgl.constexpr):
```
- **EN:** Defines the helper function `kernel`. Decorators: `gluon.jit`. Parameters: `input_desc`, `out`, `EXPECT_DELTA`. Key calls include `default_cga_layout`, `ttgl.BlockedLayout`, `ttgl.allocate_shared_memory`, `mbarrier.allocate_mbarrier`, `mbarrier.init`, `mbarrier.expect`, and 8 more. This scope touches layout transformation reasoning.
- **CN:** 定义辅助函数 `kernel`。 装饰器：`gluon.jit`。 参数：`input_desc`、`out`、`EXPECT_DELTA`。 关键调用包括 `default_cga_layout`、`ttgl.BlockedLayout`、`ttgl.allocate_shared_memory`、`mbarrier.allocate_mbarrier`、`mbarrier.init`、`mbarrier.expect` 等另外 8 项。 该作用域涉及布局变换推理。

##### Lines 630-646

```python
        block_m: ttgl.constexpr = XBLOCK * ttgl.num_ctas()
        cga_layout: ttgl.constexpr = default_cga_layout(ttgl.num_ctas(), 2)
        blocked_layout: ttgl.constexpr = ttgl.BlockedLayout(size_per_thread=[1, 1], threads_per_warp=[32, 1],
                                                            warps_per_cta=[4, 1], order=[0, 1], cga_layout=cga_layout)
        smem = ttgl.allocate_shared_memory(ttgl.float16, [block_m, XBLOCK], input_desc.layout)
        bar = mbarrier.allocate_mbarrier()
        mbarrier.init(bar, count=1)
        mbarrier.expect(bar, input_desc.nbytes_per_cta + EXPECT_DELTA)
        tma.async_load(input_desc, [0, 0], bar, smem)
        mbarrier.wait(bar, 0, deps=[smem])
        val = smem.load(blocked_layout)
        mbarrier.invalidate(bar)

        out_m = ttgl.arange(0, block_m, ttgl.SliceLayout(1, blocked_layout))[:, None]
        out_n = ttgl.arange(0, XBLOCK, ttgl.SliceLayout(0, blocked_layout))[None, :]
        out_ptr = out + out_m * XBLOCK + out_n
        ttgl.store(out_ptr, val)
```
- **EN:** Prepares or updates state through `block_m`, `cga_layout`, `blocked_layout`, `smem`, `bar`, `val`, `out_m`, `out_n`, and 1 more. Invokes `ttgl.num_ctas`, `default_cga_layout`, `ttgl.BlockedLayout`, `ttgl.allocate_shared_memory`, `mbarrier.allocate_mbarrier`, `mbarrier.init`, and 8 more to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `block_m`、`cga_layout`、`blocked_layout`、`smem`、`bar`、`val`、`out_m`、`out_n` 等另外 1 项 准备或更新状态。 调用 `ttgl.num_ctas`、`default_cga_layout`、`ttgl.BlockedLayout`、`ttgl.allocate_shared_memory`、`mbarrier.allocate_mbarrier`、`mbarrier.init` 等另外 8 项 执行测试逻辑。 相关主题：布局变换推理。

#### Lines 647-654

```python

    block_m = XBLOCK.value * num_ctas
    input = torch.randn((block_m, XBLOCK.value), device=device, dtype=torch.float16)
    output = torch.empty((block_m, XBLOCK.value), device=device, dtype=torch.float16)
    shared_layout = ttgl.NVMMASharedLayout(swizzle_byte_width=128, element_bitwidth=16, rank=2,
                                           cga_layout=default_cga_layout(num_ctas, 2))
    input_desc = gluon.nvidia.hopper.TensorDescriptor.from_tensor(input, [block_m, XBLOCK.value], shared_layout)
    kernel[(1, )](input_desc, output, EXPECT_DELTA=EXPECT_DELTA, num_warps=4, num_ctas=num_ctas)
```
- **EN:** Prepares or updates state through `block_m`, `input`, `output`, `shared_layout`, `input_desc`. Invokes `torch.randn`, `torch.empty`, `ttgl.NVMMASharedLayout`, `default_cga_layout`, `gluon.nvidia.hopper.TensorDescriptor.from_tensor` to execute the test logic. Relevant themes: PyTorch tensor setup and checks, tensor/descriptor metadata, layout transformation reasoning, random-data generation.
- **CN:** 通过 `block_m`、`input`、`output`、`shared_layout`、`input_desc` 准备或更新状态。 调用 `torch.randn`、`torch.empty`、`ttgl.NVMMASharedLayout`、`default_cga_layout`、`gluon.nvidia.hopper.TensorDescriptor.from_tensor` 执行测试逻辑。 相关主题：PyTorch 张量准备与校验、张量/描述符元数据、布局变换推理、随机数据生成。

### Lines 655-659

```python


@pytest.mark.skipif(not is_cuda() or torch.cuda.get_device_capability()[0] < 9, reason="Requires hopper or newer")
@pytest.mark.parametrize("FAILURE", [True, False])
def test_tma_interleave_kernel(FAILURE, device, run_wrapper, monkeypatch, num_ctas):
```
- **EN:** Defines the test function `test_tma_interleave_kernel`. Decorators: `pytest.mark.skipif(not is_cuda() or torch.cuda.get_device_capability()[0] < 9, reason='Requires hopper or newer')`, `pytest.mark.parametrize('FAILURE', [True, False])`. Parameters: `FAILURE`, `device`, `run_wrapper`, `monkeypatch`, `num_ctas`. Nested definitions in this scope: `kernel`. Key calls include `pytest.mark.skipif`, `pytest.mark.parametrize`, `monkeypatch.setenv`, `knobs.refresh_knobs`, `torch.randn`, `torch.empty`, and 23 more. This scope touches pytest parametrization, PyTorch tensor setup and checks, tensor/descriptor metadata, runtime driver interaction.
- **CN:** 定义测试函数 `test_tma_interleave_kernel`。 装饰器：`pytest.mark.skipif(not is_cuda() or torch.cuda.get_device_capability()[0] < 9, reason='Requires hopper or newer')`、`pytest.mark.parametrize('FAILURE', [True, False])`。 参数：`FAILURE`、`device`、`run_wrapper`、`monkeypatch`、`num_ctas`。 该作用域中的嵌套定义：`kernel`。 关键调用包括 `pytest.mark.skipif`、`pytest.mark.parametrize`、`monkeypatch.setenv`、`knobs.refresh_knobs`、`torch.randn`、`torch.empty` 等另外 23 项。 该作用域涉及pytest 参数化、PyTorch 张量准备与校验、张量/描述符元数据、运行时驱动交互。

#### Lines 660-668

```python
    if run_wrapper:
        result = run_in_process(test_tma_interleave_kernel, (FAILURE, device, False, monkeypatch, num_ctas))
        if FAILURE:
            assert_expected_cuda_failure(result.exc)
            assert "Buffer being accessed has outstanding writes" in result.driver_stderr_output
        else:
            assert result.exc is None
            assert result.driver_stderr_output == ""
        return
```
- **EN:** Invokes `run_in_process`, `assert_expected_cuda_failure` to execute the test logic. Validates behavior with 3 assertion(s). Branches on runtime or test conditions. Relevant themes: runtime driver interaction.
- **CN:** 调用 `run_in_process`、`assert_expected_cuda_failure` 执行测试逻辑。 通过 3 个断言验证行为。 根据运行时或测试条件进行分支。 相关主题：运行时驱动交互。

#### Lines 669-672

```python

    monkeypatch.setenv("TRITON_INSTRUMENTATION_MODE", "consan")
    monkeypatch.setenv("CUDA_LAUNCH_BLOCKING", "1")
    knobs.refresh_knobs()
```
- **EN:** Invokes `monkeypatch.setenv`, `knobs.refresh_knobs` to execute the test logic. Relevant themes: kernel launch orchestration.
- **CN:** 调用 `monkeypatch.setenv`、`knobs.refresh_knobs` 执行测试逻辑。 相关主题：kernel 启动编排。

#### Lines 673-675

```python

    @gluon.jit
    def kernel(input_desc, out, FAILURE: ttgl.constexpr):
```
- **EN:** Defines the helper function `kernel`. Decorators: `gluon.jit`. Parameters: `input_desc`, `out`, `FAILURE`. Key calls include `default_cga_layout`, `ttgl.allocate_shared_memory`, `mbarrier.allocate_mbarrier`, `mbarrier.init`, `mbarrier.expect`, `tma.async_load`, and 11 more. This scope touches layout transformation reasoning.
- **CN:** 定义辅助函数 `kernel`。 装饰器：`gluon.jit`。 参数：`input_desc`、`out`、`FAILURE`。 关键调用包括 `default_cga_layout`、`ttgl.allocate_shared_memory`、`mbarrier.allocate_mbarrier`、`mbarrier.init`、`mbarrier.expect`、`tma.async_load` 等另外 11 项。 该作用域涉及布局变换推理。

##### Lines 676-687

```python
        block_m: ttgl.constexpr = XBLOCK * ttgl.num_ctas()
        cga_layout: ttgl.constexpr = default_cga_layout(ttgl.num_ctas(), 2)
        smem = ttgl.allocate_shared_memory(ttgl.float16, [2, block_m, XBLOCK], input_desc.layout)
        bar = mbarrier.allocate_mbarrier(batch=2)
        mbarrier.init(bar.index(0), count=1)
        mbarrier.init(bar.index(1), count=1)
        mbarrier.expect(bar.index(0), input_desc.nbytes_per_cta)
        mbarrier.expect(bar.index(1), input_desc.nbytes_per_cta)
        tma.async_load(input_desc, [0, 0], bar.index(0), smem.index(0))
        tma.async_load(input_desc, [0, 0], bar.index(1), smem.index(1))

        mbarrier.wait(bar.index(0), 0, deps=[smem.index(0)])
```
- **EN:** Prepares or updates state through `block_m`, `cga_layout`, `smem`, `bar`. Invokes `ttgl.num_ctas`, `default_cga_layout`, `ttgl.allocate_shared_memory`, `mbarrier.allocate_mbarrier`, `mbarrier.init`, `bar.index`, and 4 more to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `block_m`、`cga_layout`、`smem`、`bar` 准备或更新状态。 调用 `ttgl.num_ctas`、`default_cga_layout`、`ttgl.allocate_shared_memory`、`mbarrier.allocate_mbarrier`、`mbarrier.init`、`bar.index` 等另外 4 项 执行测试逻辑。 相关主题：布局变换推理。

##### Lines 688-689

```python
        if not FAILURE:
            mbarrier.wait(bar.index(1), 0, deps=[smem.index(1)])
```
- **EN:** Invokes `mbarrier.wait`, `bar.index`, `smem.index` to execute the test logic. Branches on runtime or test conditions.
- **CN:** 调用 `mbarrier.wait`、`bar.index`、`smem.index` 执行测试逻辑。 根据运行时或测试条件进行分支。

##### Lines 690-703

```python

        blocked_layout: ttgl.constexpr = ttgl.BlockedLayout(size_per_thread=[1, 1], threads_per_warp=[32, 1],
                                                            warps_per_cta=[4, 1], order=[0, 1], cga_layout=cga_layout)
        out_m = ttgl.arange(0, block_m, ttgl.SliceLayout(1, blocked_layout))[:, None]
        out_n = ttgl.arange(0, XBLOCK, ttgl.SliceLayout(0, blocked_layout))[None, :]
        out_ptr = out + out_m * XBLOCK + out_n
        ttgl.store(out_ptr, smem.index(0).load(blocked_layout))
        ttgl.store(out_ptr, smem.index(1).load(blocked_layout))

        mbarrier.invalidate(bar.index(0))
        mbarrier.invalidate(bar.index(1))

        tma.async_copy_shared_to_global(input_desc, [0, 0], smem.index(0))
        tma.store_wait(0)
```
- **EN:** Prepares or updates state through `blocked_layout`, `out_m`, `out_n`, `out_ptr`. Invokes `ttgl.BlockedLayout`, `ttgl.arange`, `ttgl.SliceLayout`, `ttgl.store`, `smem.index`, `mbarrier.invalidate`, and 3 more to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `blocked_layout`、`out_m`、`out_n`、`out_ptr` 准备或更新状态。 调用 `ttgl.BlockedLayout`、`ttgl.arange`、`ttgl.SliceLayout`、`ttgl.store`、`smem.index`、`mbarrier.invalidate` 等另外 3 项 执行测试逻辑。 相关主题：布局变换推理。

#### Lines 704-711

```python

    block_m = XBLOCK.value * num_ctas
    input = torch.randn((block_m, XBLOCK.value), device=device, dtype=torch.float16)
    output = torch.empty((block_m, XBLOCK.value), device=device, dtype=torch.float16)
    shared_layout = ttgl.NVMMASharedLayout(swizzle_byte_width=128, element_bitwidth=16, rank=2,
                                           cga_layout=default_cga_layout(num_ctas, 2))
    input_desc = gluon.nvidia.hopper.TensorDescriptor.from_tensor(input, [block_m, XBLOCK.value], shared_layout)
    kernel[(1, )](input_desc, output, FAILURE=FAILURE, num_warps=4, num_ctas=num_ctas)
```
- **EN:** Prepares or updates state through `block_m`, `input`, `output`, `shared_layout`, `input_desc`. Invokes `torch.randn`, `torch.empty`, `ttgl.NVMMASharedLayout`, `default_cga_layout`, `gluon.nvidia.hopper.TensorDescriptor.from_tensor` to execute the test logic. Relevant themes: PyTorch tensor setup and checks, tensor/descriptor metadata, layout transformation reasoning, random-data generation.
- **CN:** 通过 `block_m`、`input`、`output`、`shared_layout`、`input_desc` 准备或更新状态。 调用 `torch.randn`、`torch.empty`、`ttgl.NVMMASharedLayout`、`default_cga_layout`、`gluon.nvidia.hopper.TensorDescriptor.from_tensor` 执行测试逻辑。 相关主题：PyTorch 张量准备与校验、张量/描述符元数据、布局变换推理、随机数据生成。

### Lines 712-716

```python


@pytest.mark.skipif(not is_cuda() or torch.cuda.get_device_capability()[0] < 9, reason="Requires hopper or newer")
@pytest.mark.parametrize("FAILURE", [True, False])
def test_tma_wait_tracks_only_waited_barrier(FAILURE, device, run_wrapper, monkeypatch, num_ctas):
```
- **EN:** Defines the test function `test_tma_wait_tracks_only_waited_barrier`. Decorators: `pytest.mark.skipif(not is_cuda() or torch.cuda.get_device_capability()[0] < 9, reason='Requires hopper or newer')`, `pytest.mark.parametrize('FAILURE', [True, False])`. Parameters: `FAILURE`, `device`, `run_wrapper`, `monkeypatch`, `num_ctas`. Nested definitions in this scope: `kernel`. Key calls include `pytest.mark.skipif`, `pytest.mark.parametrize`, `monkeypatch.setenv`, `knobs.refresh_knobs`, `torch.randn`, `torch.empty`, and 21 more. This scope touches pytest parametrization, PyTorch tensor setup and checks, tensor/descriptor metadata, runtime driver interaction.
- **CN:** 定义测试函数 `test_tma_wait_tracks_only_waited_barrier`。 装饰器：`pytest.mark.skipif(not is_cuda() or torch.cuda.get_device_capability()[0] < 9, reason='Requires hopper or newer')`、`pytest.mark.parametrize('FAILURE', [True, False])`。 参数：`FAILURE`、`device`、`run_wrapper`、`monkeypatch`、`num_ctas`。 该作用域中的嵌套定义：`kernel`。 关键调用包括 `pytest.mark.skipif`、`pytest.mark.parametrize`、`monkeypatch.setenv`、`knobs.refresh_knobs`、`torch.randn`、`torch.empty` 等另外 21 项。 该作用域涉及pytest 参数化、PyTorch 张量准备与校验、张量/描述符元数据、运行时驱动交互。

#### Lines 717-726

```python
    if run_wrapper:
        result = run_in_process(test_tma_wait_tracks_only_waited_barrier,
                                (FAILURE, device, False, monkeypatch, num_ctas))
        if FAILURE:
            assert_expected_cuda_failure(result.exc)
            assert "Buffer being accessed has outstanding writes" in result.driver_stderr_output
        else:
            assert result.exc is None
            assert result.driver_stderr_output == ""
        return
```
- **EN:** Invokes `run_in_process`, `assert_expected_cuda_failure` to execute the test logic. Validates behavior with 3 assertion(s). Branches on runtime or test conditions. Relevant themes: runtime driver interaction.
- **CN:** 调用 `run_in_process`、`assert_expected_cuda_failure` 执行测试逻辑。 通过 3 个断言验证行为。 根据运行时或测试条件进行分支。 相关主题：运行时驱动交互。

#### Lines 727-730

```python

    monkeypatch.setenv("TRITON_INSTRUMENTATION_MODE", "consan")
    monkeypatch.setenv("CUDA_LAUNCH_BLOCKING", "1")
    knobs.refresh_knobs()
```
- **EN:** Invokes `monkeypatch.setenv`, `knobs.refresh_knobs` to execute the test logic. Relevant themes: kernel launch orchestration.
- **CN:** 调用 `monkeypatch.setenv`、`knobs.refresh_knobs` 执行测试逻辑。 相关主题：kernel 启动编排。

#### Lines 731-733

```python

    @gluon.jit
    def kernel(input_desc, out, FAILURE: ttgl.constexpr):
```
- **EN:** Defines the helper function `kernel`. Decorators: `gluon.jit`. Parameters: `input_desc`, `out`, `FAILURE`. Key calls include `default_cga_layout`, `ttgl.allocate_shared_memory`, `mbarrier.allocate_mbarrier`, `mbarrier.init`, `mbarrier.expect`, `tma.async_load`, and 9 more. This scope touches layout transformation reasoning.
- **CN:** 定义辅助函数 `kernel`。 装饰器：`gluon.jit`。 参数：`input_desc`、`out`、`FAILURE`。 关键调用包括 `default_cga_layout`、`ttgl.allocate_shared_memory`、`mbarrier.allocate_mbarrier`、`mbarrier.init`、`mbarrier.expect`、`tma.async_load` 等另外 9 项。 该作用域涉及布局变换推理。

##### Lines 734-749

```python
        block_m: ttgl.constexpr = XBLOCK * ttgl.num_ctas()
        cga_layout: ttgl.constexpr = default_cga_layout(ttgl.num_ctas(), 2)
        smem = ttgl.allocate_shared_memory(ttgl.float16, [2, block_m, XBLOCK], input_desc.layout)
        bar = mbarrier.allocate_mbarrier(batch=2)
        mbarrier.init(bar.index(0), count=1)
        mbarrier.init(bar.index(1), count=1)

        mbarrier.expect(bar.index(0), input_desc.nbytes_per_cta)
        tma.async_load(input_desc, [0, 0], bar.index(0), smem.index(0))
        mbarrier.expect(bar.index(1), input_desc.nbytes_per_cta)
        tma.async_load(input_desc, [0, 0], bar.index(1), smem.index(1))

        mbarrier.wait(bar.index(1), 0)

        blocked_layout: ttgl.constexpr = ttgl.BlockedLayout(size_per_thread=[1, 1], threads_per_warp=[32, 1],
                                                            warps_per_cta=[4, 1], order=[0, 1], cga_layout=cga_layout)
```
- **EN:** Prepares or updates state through `block_m`, `cga_layout`, `smem`, `bar`, `blocked_layout`. Invokes `ttgl.num_ctas`, `default_cga_layout`, `ttgl.allocate_shared_memory`, `mbarrier.allocate_mbarrier`, `mbarrier.init`, `bar.index`, and 5 more to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `block_m`、`cga_layout`、`smem`、`bar`、`blocked_layout` 准备或更新状态。 调用 `ttgl.num_ctas`、`default_cga_layout`、`ttgl.allocate_shared_memory`、`mbarrier.allocate_mbarrier`、`mbarrier.init`、`bar.index` 等另外 5 项 执行测试逻辑。 相关主题：布局变换推理。

##### Lines 750-753

```python
        if FAILURE:
            val = smem.index(0).load(blocked_layout)
        else:
            val = smem.index(1).load(blocked_layout)
```
- **EN:** Invokes `smem.index` to execute the test logic. Branches on runtime or test conditions. Relevant themes: layout transformation reasoning.
- **CN:** 调用 `smem.index` 执行测试逻辑。 根据运行时或测试条件进行分支。 相关主题：布局变换推理。

##### Lines 754-757

```python
        out_m = ttgl.arange(0, block_m, ttgl.SliceLayout(1, blocked_layout))[:, None]
        out_n = ttgl.arange(0, XBLOCK, ttgl.SliceLayout(0, blocked_layout))[None, :]
        out_ptr = out + out_m * XBLOCK + out_n
        ttgl.store(out_ptr, val)
```
- **EN:** Prepares or updates state through `out_m`, `out_n`, `out_ptr`. Invokes `ttgl.arange`, `ttgl.SliceLayout`, `ttgl.store` to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `out_m`、`out_n`、`out_ptr` 准备或更新状态。 调用 `ttgl.arange`、`ttgl.SliceLayout`、`ttgl.store` 执行测试逻辑。 相关主题：布局变换推理。

##### Lines 758-760

```python

        if not FAILURE:
            mbarrier.wait(bar.index(0), 0)
```
- **EN:** Invokes `mbarrier.wait`, `bar.index` to execute the test logic. Branches on runtime or test conditions.
- **CN:** 调用 `mbarrier.wait`、`bar.index` 执行测试逻辑。 根据运行时或测试条件进行分支。

##### Lines 761-763

```python

        mbarrier.invalidate(bar.index(0))
        mbarrier.invalidate(bar.index(1))
```
- **EN:** Invokes `mbarrier.invalidate`, `bar.index` to execute the test logic.
- **CN:** 调用 `mbarrier.invalidate`、`bar.index` 执行测试逻辑。

#### Lines 764-771

```python

    block_m = XBLOCK.value * num_ctas
    input = torch.randn((block_m, XBLOCK.value), device=device, dtype=torch.float16)
    output = torch.empty((block_m, XBLOCK.value), device=device, dtype=torch.float16)
    shared_layout = ttgl.NVMMASharedLayout(swizzle_byte_width=128, element_bitwidth=16, rank=2,
                                           cga_layout=default_cga_layout(num_ctas, 2))
    input_desc = gluon.nvidia.hopper.TensorDescriptor.from_tensor(input, [block_m, XBLOCK.value], shared_layout)
    kernel[(1, )](input_desc, output, FAILURE=FAILURE, num_warps=4, num_ctas=num_ctas)
```
- **EN:** Prepares or updates state through `block_m`, `input`, `output`, `shared_layout`, `input_desc`. Invokes `torch.randn`, `torch.empty`, `ttgl.NVMMASharedLayout`, `default_cga_layout`, `gluon.nvidia.hopper.TensorDescriptor.from_tensor` to execute the test logic. Relevant themes: PyTorch tensor setup and checks, tensor/descriptor metadata, layout transformation reasoning, random-data generation.
- **CN:** 通过 `block_m`、`input`、`output`、`shared_layout`、`input_desc` 准备或更新状态。 调用 `torch.randn`、`torch.empty`、`ttgl.NVMMASharedLayout`、`default_cga_layout`、`gluon.nvidia.hopper.TensorDescriptor.from_tensor` 执行测试逻辑。 相关主题：PyTorch 张量准备与校验、张量/描述符元数据、布局变换推理、随机数据生成。

### Lines 772-776

```python


@pytest.mark.skipif(not is_cuda() or torch.cuda.get_device_capability()[0] < 9, reason="Requires hopper or newer")
@pytest.mark.parametrize("WAIT_LATEST", [True, False])
def test_tma_wait_does_not_publish_overwritten_row(WAIT_LATEST, device, run_wrapper, monkeypatch, num_ctas):
```
- **EN:** Defines the test function `test_tma_wait_does_not_publish_overwritten_row`. Decorators: `pytest.mark.skipif(not is_cuda() or torch.cuda.get_device_capability()[0] < 9, reason='Requires hopper or newer')`, `pytest.mark.parametrize('WAIT_LATEST', [True, False])`. Parameters: `WAIT_LATEST`, `device`, `run_wrapper`, `monkeypatch`, `num_ctas`. Nested definitions in this scope: `kernel`. Key calls include `pytest.mark.skipif`, `pytest.mark.parametrize`, `monkeypatch.setenv`, `knobs.refresh_knobs`, `torch.randn`, `torch.empty_like`, and 21 more. This scope touches pytest parametrization, PyTorch tensor setup and checks, tensor/descriptor metadata, runtime driver interaction.
- **CN:** 定义测试函数 `test_tma_wait_does_not_publish_overwritten_row`。 装饰器：`pytest.mark.skipif(not is_cuda() or torch.cuda.get_device_capability()[0] < 9, reason='Requires hopper or newer')`、`pytest.mark.parametrize('WAIT_LATEST', [True, False])`。 参数：`WAIT_LATEST`、`device`、`run_wrapper`、`monkeypatch`、`num_ctas`。 该作用域中的嵌套定义：`kernel`。 关键调用包括 `pytest.mark.skipif`、`pytest.mark.parametrize`、`monkeypatch.setenv`、`knobs.refresh_knobs`、`torch.randn`、`torch.empty_like` 等另外 21 项。 该作用域涉及pytest 参数化、PyTorch 张量准备与校验、张量/描述符元数据、运行时驱动交互。

#### Lines 777-786

```python
    if run_wrapper:
        result = run_in_process(test_tma_wait_does_not_publish_overwritten_row,
                                (WAIT_LATEST, device, False, monkeypatch, num_ctas))
        if WAIT_LATEST:
            assert result.exc is None
            assert result.driver_stderr_output == ""
        else:
            assert_expected_cuda_failure(result.exc)
            assert "Buffer being accessed has outstanding writes" in result.driver_stderr_output
        return
```
- **EN:** Invokes `run_in_process`, `assert_expected_cuda_failure` to execute the test logic. Validates behavior with 3 assertion(s). Branches on runtime or test conditions. Relevant themes: runtime driver interaction.
- **CN:** 调用 `run_in_process`、`assert_expected_cuda_failure` 执行测试逻辑。 通过 3 个断言验证行为。 根据运行时或测试条件进行分支。 相关主题：运行时驱动交互。

#### Lines 787-790

```python

    monkeypatch.setenv("TRITON_INSTRUMENTATION_MODE", "consan")
    monkeypatch.setenv("CUDA_LAUNCH_BLOCKING", "1")
    knobs.refresh_knobs()
```
- **EN:** Invokes `monkeypatch.setenv`, `knobs.refresh_knobs` to execute the test logic. Relevant themes: kernel launch orchestration.
- **CN:** 调用 `monkeypatch.setenv`、`knobs.refresh_knobs` 执行测试逻辑。 相关主题：kernel 启动编排。

#### Lines 791-793

```python

    @gluon.jit
    def kernel(input_desc, out, WAIT_LATEST: ttgl.constexpr):
```
- **EN:** Defines the helper function `kernel`. Decorators: `gluon.jit`. Parameters: `input_desc`, `out`, `WAIT_LATEST`. Key calls include `default_cga_layout`, `ttgl.allocate_shared_memory`, `mbarrier.allocate_mbarrier`, `mbarrier.init`, `mbarrier.expect`, `tma.async_load`, and 9 more. This scope touches layout transformation reasoning.
- **CN:** 定义辅助函数 `kernel`。 装饰器：`gluon.jit`。 参数：`input_desc`、`out`、`WAIT_LATEST`。 关键调用包括 `default_cga_layout`、`ttgl.allocate_shared_memory`、`mbarrier.allocate_mbarrier`、`mbarrier.init`、`mbarrier.expect`、`tma.async_load` 等另外 9 项。 该作用域涉及布局变换推理。

##### Lines 794-804

```python
        block_m: ttgl.constexpr = XBLOCK * ttgl.num_ctas()
        cga_layout: ttgl.constexpr = default_cga_layout(ttgl.num_ctas(), 2)
        smem = ttgl.allocate_shared_memory(ttgl.float16, [block_m, XBLOCK], input_desc.layout)
        bar = mbarrier.allocate_mbarrier(batch=2)
        mbarrier.init(bar.index(0), count=1)
        mbarrier.init(bar.index(1), count=1)

        mbarrier.expect(bar.index(0), input_desc.nbytes_per_cta)
        tma.async_load(input_desc, [0, 0], bar.index(0), smem)
        mbarrier.expect(bar.index(1), input_desc.nbytes_per_cta)
        tma.async_load(input_desc, [0, 0], bar.index(1), smem)
```
- **EN:** Prepares or updates state through `block_m`, `cga_layout`, `smem`, `bar`. Invokes `ttgl.num_ctas`, `default_cga_layout`, `ttgl.allocate_shared_memory`, `mbarrier.allocate_mbarrier`, `mbarrier.init`, `bar.index`, and 2 more to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `block_m`、`cga_layout`、`smem`、`bar` 准备或更新状态。 调用 `ttgl.num_ctas`、`default_cga_layout`、`ttgl.allocate_shared_memory`、`mbarrier.allocate_mbarrier`、`mbarrier.init`、`bar.index` 等另外 2 项 执行测试逻辑。 相关主题：布局变换推理。

##### Lines 805-809

```python

        if WAIT_LATEST:
            mbarrier.wait(bar.index(1), 0)
        else:
            mbarrier.wait(bar.index(0), 0)
```
- **EN:** Invokes `mbarrier.wait`, `bar.index` to execute the test logic. Branches on runtime or test conditions.
- **CN:** 调用 `mbarrier.wait`、`bar.index` 执行测试逻辑。 根据运行时或测试条件进行分支。

##### Lines 810-816

```python

        blocked_layout: ttgl.constexpr = ttgl.BlockedLayout(size_per_thread=[1, 1], threads_per_warp=[32, 1],
                                                            warps_per_cta=[4, 1], order=[0, 1], cga_layout=cga_layout)
        val = smem.load(blocked_layout)
        out_m = ttgl.arange(0, block_m, ttgl.SliceLayout(1, blocked_layout))[:, None]
        out_n = ttgl.arange(0, XBLOCK, ttgl.SliceLayout(0, blocked_layout))[None, :]
        ttgl.store(out + out_m * XBLOCK + out_n, val)
```
- **EN:** Prepares or updates state through `blocked_layout`, `val`, `out_m`, `out_n`. Invokes `ttgl.BlockedLayout`, `smem.load`, `ttgl.arange`, `ttgl.SliceLayout`, `ttgl.store` to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `blocked_layout`、`val`、`out_m`、`out_n` 准备或更新状态。 调用 `ttgl.BlockedLayout`、`smem.load`、`ttgl.arange`、`ttgl.SliceLayout`、`ttgl.store` 执行测试逻辑。 相关主题：布局变换推理。

##### Lines 817-821

```python

        if WAIT_LATEST:
            mbarrier.wait(bar.index(0), 0)
        else:
            mbarrier.wait(bar.index(1), 0)
```
- **EN:** Invokes `mbarrier.wait`, `bar.index` to execute the test logic. Branches on runtime or test conditions.
- **CN:** 调用 `mbarrier.wait`、`bar.index` 执行测试逻辑。 根据运行时或测试条件进行分支。

##### Lines 822-824

```python

        mbarrier.invalidate(bar.index(0))
        mbarrier.invalidate(bar.index(1))
```
- **EN:** Invokes `mbarrier.invalidate`, `bar.index` to execute the test logic.
- **CN:** 调用 `mbarrier.invalidate`、`bar.index` 执行测试逻辑。

#### Lines 825-832

```python

    block_m = XBLOCK.value * num_ctas
    input = torch.randn((block_m, XBLOCK.value), device=device, dtype=torch.float16)
    output = torch.empty_like(input)
    shared_layout = ttgl.NVMMASharedLayout(swizzle_byte_width=128, element_bitwidth=16, rank=2,
                                           cga_layout=default_cga_layout(num_ctas, 2))
    input_desc = gluon.nvidia.hopper.TensorDescriptor.from_tensor(input, [block_m, XBLOCK.value], shared_layout)
    kernel[(1, )](input_desc, output, WAIT_LATEST=WAIT_LATEST, num_warps=4, num_ctas=num_ctas)
```
- **EN:** Prepares or updates state through `block_m`, `input`, `output`, `shared_layout`, `input_desc`. Invokes `torch.randn`, `torch.empty_like`, `ttgl.NVMMASharedLayout`, `default_cga_layout`, `gluon.nvidia.hopper.TensorDescriptor.from_tensor` to execute the test logic. Relevant themes: PyTorch tensor setup and checks, tensor/descriptor metadata, layout transformation reasoning, random-data generation.
- **CN:** 通过 `block_m`、`input`、`output`、`shared_layout`、`input_desc` 准备或更新状态。 调用 `torch.randn`、`torch.empty_like`、`ttgl.NVMMASharedLayout`、`default_cga_layout`、`gluon.nvidia.hopper.TensorDescriptor.from_tensor` 执行测试逻辑。 相关主题：PyTorch 张量准备与校验、张量/描述符元数据、布局变换推理、随机数据生成。

### Lines 833-837

```python


@pytest.mark.skipif(not is_cuda() or torch.cuda.get_device_capability()[0] < 9, reason="Requires ampere or newer")
@pytest.mark.parametrize("FAILURE", [True, False])
def test_async_copy(FAILURE, device, run_wrapper, monkeypatch, num_ctas):
```
- **EN:** Defines the test function `test_async_copy`. Decorators: `pytest.mark.skipif(not is_cuda() or torch.cuda.get_device_capability()[0] < 9, reason='Requires ampere or newer')`, `pytest.mark.parametrize('FAILURE', [True, False])`. Parameters: `FAILURE`, `device`, `run_wrapper`, `monkeypatch`, `num_ctas`. Nested definitions in this scope: `kernel`. Key calls include `pytest.mark.skipif`, `pytest.mark.parametrize`, `monkeypatch.setenv`, `knobs.refresh_knobs`, `torch.randn`, `run_in_process`, and 14 more. This scope touches pytest parametrization, PyTorch tensor setup and checks, runtime driver interaction, kernel launch orchestration.
- **CN:** 定义测试函数 `test_async_copy`。 装饰器：`pytest.mark.skipif(not is_cuda() or torch.cuda.get_device_capability()[0] < 9, reason='Requires ampere or newer')`、`pytest.mark.parametrize('FAILURE', [True, False])`。 参数：`FAILURE`、`device`、`run_wrapper`、`monkeypatch`、`num_ctas`。 该作用域中的嵌套定义：`kernel`。 关键调用包括 `pytest.mark.skipif`、`pytest.mark.parametrize`、`monkeypatch.setenv`、`knobs.refresh_knobs`、`torch.randn`、`run_in_process` 等另外 14 项。 该作用域涉及pytest 参数化、PyTorch 张量准备与校验、运行时驱动交互、kernel 启动编排。

#### Lines 838-846

```python
    if run_wrapper:
        result = run_in_process(test_async_copy, (FAILURE, device, False, monkeypatch, num_ctas))
        if FAILURE:
            assert_expected_cuda_failure(result.exc)
            assert "Accessing buffer with pending access. Pending access type: async_copy_global_to_shared" in result.driver_stderr_output
        else:
            assert result.exc is None
            assert result.driver_stderr_output == ""
        return
```
- **EN:** Invokes `run_in_process`, `assert_expected_cuda_failure` to execute the test logic. Validates behavior with 3 assertion(s). Branches on runtime or test conditions. Relevant themes: runtime driver interaction.
- **CN:** 调用 `run_in_process`、`assert_expected_cuda_failure` 执行测试逻辑。 通过 3 个断言验证行为。 根据运行时或测试条件进行分支。 相关主题：运行时驱动交互。

#### Lines 847-850

```python

    monkeypatch.setenv("TRITON_INSTRUMENTATION_MODE", "consan")
    monkeypatch.setenv("CUDA_LAUNCH_BLOCKING", "1")
    knobs.refresh_knobs()
```
- **EN:** Invokes `monkeypatch.setenv`, `knobs.refresh_knobs` to execute the test logic. Relevant themes: kernel launch orchestration.
- **CN:** 调用 `monkeypatch.setenv`、`knobs.refresh_knobs` 执行测试逻辑。 相关主题：kernel 启动编排。

#### Lines 851-853

```python

    @gluon.jit
    def kernel(input, FAILURE: ttgl.constexpr):
```
- **EN:** Defines the helper function `kernel`. Decorators: `gluon.jit`. Parameters: `input`, `FAILURE`. Key calls include `default_cga_layout`, `ttgl.NVMMASharedLayout`, `ttgl.allocate_shared_memory`, `ttgl.BlockedLayout`, `ampere.async_copy.async_copy_global_to_shared`, `ampere.async_copy.commit_group`, and 5 more. This scope touches layout transformation reasoning.
- **CN:** 定义辅助函数 `kernel`。 装饰器：`gluon.jit`。 参数：`input`、`FAILURE`。 关键调用包括 `default_cga_layout`、`ttgl.NVMMASharedLayout`、`ttgl.allocate_shared_memory`、`ttgl.BlockedLayout`、`ampere.async_copy.async_copy_global_to_shared`、`ampere.async_copy.commit_group` 等另外 5 项。 该作用域涉及布局变换推理。

##### Lines 854-873

```python
        block_m: ttgl.constexpr = XBLOCK * ttgl.num_ctas()
        cga_layout: ttgl.constexpr = default_cga_layout(ttgl.num_ctas(), 2)
        smem_layout: ttgl.constexpr = ttgl.NVMMASharedLayout(swizzle_byte_width=128, element_bitwidth=16, rank=2,
                                                             cga_layout=cga_layout)
        smem = ttgl.allocate_shared_memory(ttgl.float16, [2, block_m, XBLOCK], smem_layout)
        blocked_layout: ttgl.constexpr = ttgl.BlockedLayout(size_per_thread=[1, XBLOCK], threads_per_warp=[32, 1],
                                                            warps_per_cta=[4, 1], order=[0, 1], cga_layout=cga_layout)
        offs_m = ttgl.arange(0, block_m, layout=ttgl.SliceLayout(dim=1, parent=blocked_layout))[:, None]
        offs_n = ttgl.arange(0, XBLOCK, layout=ttgl.SliceLayout(dim=0, parent=blocked_layout))[None, :]
        offs = offs_m * XBLOCK + offs_n
        ampere.async_copy.async_copy_global_to_shared(smem.index(0), input + offs)
        ampere.async_copy.commit_group()

        ampere.async_copy.async_copy_global_to_shared(smem.index(1), input + offs)
        ampere.async_copy.commit_group()
        ampere.async_copy.wait_group(2 if FAILURE else 1)

        ampere.async_copy.async_copy_global_to_shared(smem.index(0), input + offs)
        ampere.async_copy.commit_group()
        ampere.async_copy.wait_group(0)
```
- **EN:** Prepares or updates state through `block_m`, `cga_layout`, `smem_layout`, `smem`, `blocked_layout`, `offs_m`, `offs_n`, `offs`. Invokes `ttgl.num_ctas`, `default_cga_layout`, `ttgl.NVMMASharedLayout`, `ttgl.allocate_shared_memory`, `ttgl.BlockedLayout`, `ttgl.arange`, and 5 more to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `block_m`、`cga_layout`、`smem_layout`、`smem`、`blocked_layout`、`offs_m`、`offs_n`、`offs` 准备或更新状态。 调用 `ttgl.num_ctas`、`default_cga_layout`、`ttgl.NVMMASharedLayout`、`ttgl.allocate_shared_memory`、`ttgl.BlockedLayout`、`ttgl.arange` 等另外 5 项 执行测试逻辑。 相关主题：布局变换推理。

#### Lines 874-876

```python

    input = torch.randn((XBLOCK.value * num_ctas, XBLOCK.value), device=device, dtype=torch.float16)
    kernel[(1, )](input, FAILURE=FAILURE, num_warps=4, num_ctas=num_ctas)
```
- **EN:** Prepares or updates state through `input`. Invokes `torch.randn` to execute the test logic. Relevant themes: PyTorch tensor setup and checks, random-data generation.
- **CN:** 通过 `input` 准备或更新状态。 调用 `torch.randn` 执行测试逻辑。 相关主题：PyTorch 张量准备与校验、随机数据生成。

### Lines 877-881

```python


@pytest.mark.skipif(not is_cuda() or torch.cuda.get_device_capability()[0] < 9, reason="Requires ampere or newer")
@pytest.mark.parametrize("FAILURE", [True, False])
def test_tma_store(FAILURE, device, run_wrapper, monkeypatch, num_ctas):
```
- **EN:** Defines the test function `test_tma_store`. Decorators: `pytest.mark.skipif(not is_cuda() or torch.cuda.get_device_capability()[0] < 9, reason='Requires ampere or newer')`, `pytest.mark.parametrize('FAILURE', [True, False])`. Parameters: `FAILURE`, `device`, `run_wrapper`, `monkeypatch`, `num_ctas`. Nested definitions in this scope: `kernel`. Key calls include `pytest.mark.skipif`, `pytest.mark.parametrize`, `monkeypatch.setenv`, `knobs.refresh_knobs`, `torch.empty`, `ttgl.NVMMASharedLayout`, and 13 more. This scope touches pytest parametrization, PyTorch tensor setup and checks, tensor/descriptor metadata, runtime driver interaction.
- **CN:** 定义测试函数 `test_tma_store`。 装饰器：`pytest.mark.skipif(not is_cuda() or torch.cuda.get_device_capability()[0] < 9, reason='Requires ampere or newer')`、`pytest.mark.parametrize('FAILURE', [True, False])`。 参数：`FAILURE`、`device`、`run_wrapper`、`monkeypatch`、`num_ctas`。 该作用域中的嵌套定义：`kernel`。 关键调用包括 `pytest.mark.skipif`、`pytest.mark.parametrize`、`monkeypatch.setenv`、`knobs.refresh_knobs`、`torch.empty`、`ttgl.NVMMASharedLayout` 等另外 13 项。 该作用域涉及pytest 参数化、PyTorch 张量准备与校验、张量/描述符元数据、运行时驱动交互。

#### Lines 882-890

```python
    if run_wrapper:
        result = run_in_process(test_tma_store, (FAILURE, device, False, monkeypatch, num_ctas))
        if FAILURE:
            assert_expected_cuda_failure(result.exc)
            assert "Accessing buffer with pending access. Pending access type: async_copy_shared_to_global" in result.driver_stderr_output
        else:
            assert result.exc is None
            assert result.driver_stderr_output == ""
        return
```
- **EN:** Invokes `run_in_process`, `assert_expected_cuda_failure` to execute the test logic. Validates behavior with 3 assertion(s). Branches on runtime or test conditions. Relevant themes: runtime driver interaction.
- **CN:** 调用 `run_in_process`、`assert_expected_cuda_failure` 执行测试逻辑。 通过 3 个断言验证行为。 根据运行时或测试条件进行分支。 相关主题：运行时驱动交互。

#### Lines 891-894

```python

    monkeypatch.setenv("TRITON_INSTRUMENTATION_MODE", "consan")
    monkeypatch.setenv("CUDA_LAUNCH_BLOCKING", "1")
    knobs.refresh_knobs()
```
- **EN:** Invokes `monkeypatch.setenv`, `knobs.refresh_knobs` to execute the test logic. Relevant themes: kernel launch orchestration.
- **CN:** 调用 `monkeypatch.setenv`、`knobs.refresh_knobs` 执行测试逻辑。 相关主题：kernel 启动编排。

#### Lines 895-897

```python

    @gluon.jit
    def kernel(output_desc, FAILURE: ttgl.constexpr):
```
- **EN:** Defines the helper function `kernel`. Decorators: `gluon.jit`. Parameters: `output_desc`, `FAILURE`. Key calls include `default_cga_layout`, `ttgl.NVMMASharedLayout`, `ttgl.allocate_shared_memory`, `ttgl.BlockedLayout`, `ttgl.full`, `tma.async_copy_shared_to_global`, and 3 more. This scope touches layout transformation reasoning.
- **CN:** 定义辅助函数 `kernel`。 装饰器：`gluon.jit`。 参数：`output_desc`、`FAILURE`。 关键调用包括 `default_cga_layout`、`ttgl.NVMMASharedLayout`、`ttgl.allocate_shared_memory`、`ttgl.BlockedLayout`、`ttgl.full`、`tma.async_copy_shared_to_global` 等另外 3 项。 该作用域涉及布局变换推理。

##### Lines 898-909

```python
        block_m: ttgl.constexpr = XBLOCK * ttgl.num_ctas()
        cga_layout: ttgl.constexpr = default_cga_layout(ttgl.num_ctas(), 2)
        smem_layout: ttgl.constexpr = ttgl.NVMMASharedLayout(swizzle_byte_width=128, element_bitwidth=16, rank=2,
                                                             cga_layout=cga_layout)
        smem = ttgl.allocate_shared_memory(ttgl.float16, [2, block_m, XBLOCK], smem_layout)
        blocked_layout: ttgl.constexpr = ttgl.BlockedLayout(size_per_thread=[1, XBLOCK], threads_per_warp=[32, 1],
                                                            warps_per_cta=[4, 1], order=[0, 1], cga_layout=cga_layout)
        val = ttgl.full([block_m, XBLOCK], 42, ttgl.float16, blocked_layout)
        tma.async_copy_shared_to_global(output_desc, [0, 0], smem.index(0))
        tma.async_copy_shared_to_global(output_desc, [0, 0], smem.index(1))
        tma.store_wait(pendings=1)
        smem.index(0).store(val)
```
- **EN:** Prepares or updates state through `block_m`, `cga_layout`, `smem_layout`, `smem`, `blocked_layout`, `val`. Invokes `ttgl.num_ctas`, `default_cga_layout`, `ttgl.NVMMASharedLayout`, `ttgl.allocate_shared_memory`, `ttgl.BlockedLayout`, `ttgl.full`, and 3 more to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `block_m`、`cga_layout`、`smem_layout`、`smem`、`blocked_layout`、`val` 准备或更新状态。 调用 `ttgl.num_ctas`、`default_cga_layout`、`ttgl.NVMMASharedLayout`、`ttgl.allocate_shared_memory`、`ttgl.BlockedLayout`、`ttgl.full` 等另外 3 项 执行测试逻辑。 相关主题：布局变换推理。

##### Lines 910-911

```python
        if not FAILURE:
            tma.store_wait(pendings=0)
```
- **EN:** Invokes `tma.store_wait` to execute the test logic. Branches on runtime or test conditions.
- **CN:** 调用 `tma.store_wait` 执行测试逻辑。 根据运行时或测试条件进行分支。

##### Lines 912-912

```python
        smem.index(1).store(val)
```
- **EN:** Invokes `smem.index` to execute the test logic.
- **CN:** 调用 `smem.index` 执行测试逻辑。

#### Lines 913-919

```python

    block_m = XBLOCK.value * num_ctas
    output = torch.empty((block_m, XBLOCK.value), device=device, dtype=torch.float16)
    shared_layout = ttgl.NVMMASharedLayout(swizzle_byte_width=128, element_bitwidth=16, rank=2,
                                           cga_layout=default_cga_layout(num_ctas, 2))
    output_desc = gluon.nvidia.hopper.TensorDescriptor.from_tensor(output, [block_m, XBLOCK.value], shared_layout)
    kernel[(1, )](output_desc, FAILURE=FAILURE, num_warps=4, num_ctas=num_ctas)
```
- **EN:** Prepares or updates state through `block_m`, `output`, `shared_layout`, `output_desc`. Invokes `torch.empty`, `ttgl.NVMMASharedLayout`, `default_cga_layout`, `gluon.nvidia.hopper.TensorDescriptor.from_tensor` to execute the test logic. Relevant themes: PyTorch tensor setup and checks, tensor/descriptor metadata, layout transformation reasoning.
- **CN:** 通过 `block_m`、`output`、`shared_layout`、`output_desc` 准备或更新状态。 调用 `torch.empty`、`ttgl.NVMMASharedLayout`、`default_cga_layout`、`gluon.nvidia.hopper.TensorDescriptor.from_tensor` 执行测试逻辑。 相关主题：PyTorch 张量准备与校验、张量/描述符元数据、布局变换推理。

### Lines 920-926

```python


@pytest.mark.skipif(not is_cuda() or torch.cuda.get_device_capability()[0] < 10, reason="Requires blackwell or newer")
@pytest.mark.parametrize("FAILURE", [True, False])
@pytest.mark.parametrize("MEM_ACCESS_KIND", ["tma_cp", "local_store", "tmem_load", "tmem_store"])
@pytest.mark.parametrize("TWO_CTAS", [False, True])
def test_tcgen5_mma(FAILURE, MEM_ACCESS_KIND, TWO_CTAS, device, run_wrapper, monkeypatch, num_ctas):
```
- **EN:** Defines the test function `test_tcgen5_mma`. Decorators: `pytest.mark.skipif(not is_cuda() or torch.cuda.get_device_capability()[0] < 10, reason='Requires blackwell or newer')`, `pytest.mark.parametrize('FAILURE', [True, False])`, `pytest.mark.parametrize('MEM_ACCESS_KIND', ['tma_cp', 'local_store', 'tmem_load', 'tmem_store'])`, `pytest.mark.parametrize('TWO_CTAS', [False, True])`. Parameters: `FAILURE`, `MEM_ACCESS_KIND`, `TWO_CTAS`, `device`, `run_wrapper`, `monkeypatch`, `num_ctas`. Nested definitions in this scope: `kernel`. Key calls include `pytest.mark.skipif`, `pytest.mark.parametrize`, `monkeypatch.setenv`, `knobs.refresh_knobs`, `mma_block_m`, `mma_block_n`, and 30 more. This scope touches pytest parametrization, PyTorch tensor setup and checks, tensor/descriptor metadata, runtime driver interaction.
- **CN:** 定义测试函数 `test_tcgen5_mma`。 装饰器：`pytest.mark.skipif(not is_cuda() or torch.cuda.get_device_capability()[0] < 10, reason='Requires blackwell or newer')`、`pytest.mark.parametrize('FAILURE', [True, False])`、`pytest.mark.parametrize('MEM_ACCESS_KIND', ['tma_cp', 'local_store', 'tmem_load', 'tmem_store'])`、`pytest.mark.parametrize('TWO_CTAS', [False, True])`。 参数：`FAILURE`、`MEM_ACCESS_KIND`、`TWO_CTAS`、`device`、`run_wrapper`、`monkeypatch`、`num_ctas`。 该作用域中的嵌套定义：`kernel`。 关键调用包括 `pytest.mark.skipif`、`pytest.mark.parametrize`、`monkeypatch.setenv`、`knobs.refresh_knobs`、`mma_block_m`、`mma_block_n` 等另外 30 项。 该作用域涉及pytest 参数化、PyTorch 张量准备与校验、张量/描述符元数据、运行时驱动交互。

#### Lines 927-928

```python
    if TWO_CTAS and num_ctas == 1:
        pytest.skip("Need at least 2 CTAs for 2CTA mode in this test")
```
- **EN:** Invokes `pytest.skip` to execute the test logic. Branches on runtime or test conditions.
- **CN:** 调用 `pytest.skip` 执行测试逻辑。 根据运行时或测试条件进行分支。

#### Lines 929-943

```python
    if run_wrapper:
        result = run_in_process(test_tcgen5_mma,
                                (FAILURE, MEM_ACCESS_KIND, TWO_CTAS, device, False, monkeypatch, num_ctas))
        if FAILURE:
            assert_expected_cuda_failure(result.exc)
            if MEM_ACCESS_KIND == "tma_cp":
                # shmem operands are being read by the tcgen05_mma
                assert "Buffer being accessed has outstanding reads" in result.driver_stderr_output
            elif MEM_ACCESS_KIND in ["tmem_load", "tmem_store"]:
                # tmem is being written by the tcgen05_mma
                assert "Buffer being accessed has outstanding writes" in result.driver_stderr_output
        else:
            assert result.exc is None
            assert result.driver_stderr_output == ""
        return
```
- **EN:** Invokes `run_in_process`, `assert_expected_cuda_failure` to execute the test logic. Validates behavior with 4 assertion(s). Branches on runtime or test conditions. Relevant themes: runtime driver interaction, random-data generation.
- **CN:** 调用 `run_in_process`、`assert_expected_cuda_failure` 执行测试逻辑。 通过 4 个断言验证行为。 根据运行时或测试条件进行分支。 相关主题：运行时驱动交互、随机数据生成。

#### Lines 944-947

```python

    monkeypatch.setenv("TRITON_INSTRUMENTATION_MODE", "consan")
    monkeypatch.setenv("CUDA_LAUNCH_BLOCKING", "1")
    knobs.refresh_knobs()
```
- **EN:** Invokes `monkeypatch.setenv`, `knobs.refresh_knobs` to execute the test logic. Relevant themes: kernel launch orchestration.
- **CN:** 调用 `monkeypatch.setenv`、`knobs.refresh_knobs` 执行测试逻辑。 相关主题：kernel 启动编排。

#### Lines 948-951

```python

    @gluon.jit
    def kernel(input_desc, output_desc, FAILURE: ttgl.constexpr, MEM_ACCESS_KIND: ttgl.constexpr,
               TWO_CTAS: ttgl.constexpr):
```
- **EN:** Defines the helper function `kernel`. Decorators: `gluon.jit`. Parameters: `input_desc`, `output_desc`, `FAILURE`, `MEM_ACCESS_KIND`, `TWO_CTAS`. Key calls include `mma_block_m`, `mma_block_n`, `blackwell.TensorMemoryLayout`, `ttgl.BlockedLayout`, `ttgl.allocate_shared_memory`, `mbarrier.allocate_mbarrier`, and 18 more. This scope touches layout transformation reasoning.
- **CN:** 定义辅助函数 `kernel`。 装饰器：`gluon.jit`。 参数：`input_desc`、`output_desc`、`FAILURE`、`MEM_ACCESS_KIND`、`TWO_CTAS`。 关键调用包括 `mma_block_m`、`mma_block_n`、`blackwell.TensorMemoryLayout`、`ttgl.BlockedLayout`、`ttgl.allocate_shared_memory`、`mbarrier.allocate_mbarrier` 等另外 18 项。 该作用域涉及布局变换推理。

##### Lines 952-975

```python
        block_m: ttgl.constexpr = mma_block_m(ttgl.num_ctas())
        block_n: ttgl.constexpr = mma_block_n(ttgl.num_ctas())
        acc_layout: ttgl.constexpr = blackwell.TensorMemoryLayout(
            [XBLOCK, XBLOCK],
            col_stride=1,
            cga_layout=mma_cga_layout(ttgl.num_ctas(), 2, TWO_CTAS),
            two_ctas=TWO_CTAS,
        )
        smem_a_blocked_layout: ttgl.constexpr = ttgl.BlockedLayout(
            size_per_thread=[1, XBLOCK], threads_per_warp=[32, 1], warps_per_cta=[4, 1], order=[0, 1],
            cga_layout=mma_cga_layout(ttgl.num_ctas(), 0, TWO_CTAS))
        acc_blocked_layout: ttgl.constexpr = ttgl.BlockedLayout(size_per_thread=[1, XBLOCK], threads_per_warp=[32, 1],
                                                                warps_per_cta=[4, 1], order=[0, 1],
                                                                cga_layout=acc_layout.cga_layout)
        smemA = ttgl.allocate_shared_memory(ttgl.float16, [block_m, XBLOCK], input_desc.layout)
        smemB = ttgl.allocate_shared_memory(
            ttgl.float16,
            [XBLOCK, block_n],
            ttgl.NVMMASharedLayout.get_default_for([XBLOCK, block_n], ttgl.float16,
                                                   cga_layout=mma_cga_layout(ttgl.num_ctas(), 1, TWO_CTAS)),
        )
        mma_bar = mbarrier.allocate_mbarrier()
        acc = blackwell.allocate_tensor_memory(ttgl.float32, [block_m, block_n], acc_layout)
        mbarrier.init(mma_bar, count=1)
```
- **EN:** Prepares or updates state through `block_m`, `block_n`, `acc_layout`, `smem_a_blocked_layout`, `acc_blocked_layout`, `smemA`, `smemB`, `mma_bar`, and 1 more. Invokes `mma_block_m`, `ttgl.num_ctas`, `mma_block_n`, `blackwell.TensorMemoryLayout`, `mma_cga_layout`, `ttgl.BlockedLayout`, and 5 more to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `block_m`、`block_n`、`acc_layout`、`smem_a_blocked_layout`、`acc_blocked_layout`、`smemA`、`smemB`、`mma_bar` 等另外 1 项 准备或更新状态。 调用 `mma_block_m`、`ttgl.num_ctas`、`mma_block_n`、`blackwell.TensorMemoryLayout`、`mma_cga_layout`、`ttgl.BlockedLayout` 等另外 5 项 执行测试逻辑。 相关主题：布局变换推理。

##### Lines 976-978

```python
        if MEM_ACCESS_KIND == "tma_cp":
            tma_bar = mbarrier.allocate_mbarrier(two_ctas=TWO_CTAS)
            mbarrier.init(tma_bar, count=1)
```
- **EN:** Invokes `mbarrier.allocate_mbarrier`, `mbarrier.init` to execute the test logic. Branches on runtime or test conditions.
- **CN:** 调用 `mbarrier.allocate_mbarrier`、`mbarrier.init` 执行测试逻辑。 根据运行时或测试条件进行分支。

##### Lines 979-981

```python

        blackwell.tcgen05_mma(smemA, smemB, acc)
        blackwell.tcgen05_commit(mma_bar)
```
- **EN:** Invokes `blackwell.tcgen05_mma`, `blackwell.tcgen05_commit` to execute the test logic.
- **CN:** 调用 `blackwell.tcgen05_mma`、`blackwell.tcgen05_commit` 执行测试逻辑。

##### Lines 982-984

```python

        if not FAILURE:
            mbarrier.wait(mma_bar, 0)
```
- **EN:** Invokes `mbarrier.wait` to execute the test logic. Branches on runtime or test conditions.
- **CN:** 调用 `mbarrier.wait` 执行测试逻辑。 根据运行时或测试条件进行分支。

##### Lines 985-1002

```python

        if MEM_ACCESS_KIND == "tma_cp":
            mbarrier.expect(tma_bar, input_desc.nbytes_per_cta)
            tma.async_load(input_desc, [0, 0], tma_bar, smemA)
            mbarrier.wait(tma_bar, 0)
            mbarrier.invalidate(tma_bar)
        elif MEM_ACCESS_KIND == "local_store":
            smemA.store(ttgl.full([block_m, XBLOCK], 42, ttgl.float16, smem_a_blocked_layout))
        elif MEM_ACCESS_KIND == "tmem_load":
            res = acc.load(acc_blocked_layout)
            smemAcc = ttgl.allocate_shared_memory(
                input_desc.dtype, [block_m, block_n],
                ttgl.NVMMASharedLayout.get_default_for([block_m, block_n], input_desc.dtype,
                                                       cga_layout=acc_layout.cga_layout), res.to(input_desc.dtype))
            tma.async_copy_shared_to_global(output_desc, [0, 0], smemAcc)
            tma.store_wait(0)
        elif MEM_ACCESS_KIND == "tmem_store":
            acc.store(ttgl.full([block_m, block_n], 42, ttgl.float32, acc_blocked_layout))
```
- **EN:** Invokes `mbarrier.expect`, `tma.async_load`, `mbarrier.wait`, `mbarrier.invalidate`, `smemA.store`, `ttgl.full`, and 7 more to execute the test logic. Branches on runtime or test conditions. Relevant themes: layout transformation reasoning.
- **CN:** 调用 `mbarrier.expect`、`tma.async_load`、`mbarrier.wait`、`mbarrier.invalidate`、`smemA.store`、`ttgl.full` 等另外 7 项 执行测试逻辑。 根据运行时或测试条件进行分支。 相关主题：布局变换推理。

##### Lines 1003-1004

```python

        mbarrier.invalidate(mma_bar)
```
- **EN:** Invokes `mbarrier.invalidate` to execute the test logic.
- **CN:** 调用 `mbarrier.invalidate` 执行测试逻辑。

#### Lines 1005-1017

```python

    block_m = mma_block_m(num_ctas)
    block_n = mma_block_n(num_ctas)
    input = torch.randn((block_m, XBLOCK.value), device=device, dtype=torch.float16)
    shared_layout = ttgl.NVMMASharedLayout.get_default_for([block_m, XBLOCK.value], ttgl.float16,
                                                           cga_layout=mma_cga_layout(num_ctas, 0, TWO_CTAS))
    input_desc = gluon.nvidia.hopper.TensorDescriptor.from_tensor(input, [block_m, XBLOCK.value], shared_layout)
    output = torch.empty((block_m, block_n), device=device, dtype=torch.float16)
    output_layout = ttgl.NVMMASharedLayout.get_default_for([block_m, block_n], ttgl.float16,
                                                           cga_layout=mma_cga_layout(num_ctas, 2, TWO_CTAS))
    output_desc = gluon.nvidia.hopper.TensorDescriptor.from_tensor(output, [block_m, block_n], output_layout)
    kernel[(1, )](input_desc, output_desc, FAILURE=FAILURE, MEM_ACCESS_KIND=MEM_ACCESS_KIND, TWO_CTAS=TWO_CTAS,
                  num_warps=4, num_ctas=num_ctas)
```
- **EN:** Prepares or updates state through `block_m`, `block_n`, `input`, `shared_layout`, `input_desc`, `output`, `output_layout`, `output_desc`. Invokes `mma_block_m`, `mma_block_n`, `torch.randn`, `ttgl.NVMMASharedLayout.get_default_for`, `mma_cga_layout`, `gluon.nvidia.hopper.TensorDescriptor.from_tensor`, and 1 more to execute the test logic. Relevant themes: PyTorch tensor setup and checks, tensor/descriptor metadata, layout transformation reasoning, random-data generation.
- **CN:** 通过 `block_m`、`block_n`、`input`、`shared_layout`、`input_desc`、`output`、`output_layout`、`output_desc` 准备或更新状态。 调用 `mma_block_m`、`mma_block_n`、`torch.randn`、`ttgl.NVMMASharedLayout.get_default_for`、`mma_cga_layout`、`gluon.nvidia.hopper.TensorDescriptor.from_tensor` 等另外 1 项 执行测试逻辑。 相关主题：PyTorch 张量准备与校验、张量/描述符元数据、布局变换推理、随机数据生成。

### Lines 1018-1023

```python


@pytest.mark.skipif(not is_cuda() or torch.cuda.get_device_capability()[0] < 10, reason="Requires blackwell or newer")
@pytest.mark.parametrize("FAILURE", [True, False])
@pytest.mark.parametrize("MEM_ACCESS_KIND", ["local_store", "tmem_load"])
def test_tcgen5_copy(FAILURE, MEM_ACCESS_KIND, device, run_wrapper, monkeypatch, num_ctas):
```
- **EN:** Defines the test function `test_tcgen5_copy`. Decorators: `pytest.mark.skipif(not is_cuda() or torch.cuda.get_device_capability()[0] < 10, reason='Requires blackwell or newer')`, `pytest.mark.parametrize('FAILURE', [True, False])`, `pytest.mark.parametrize('MEM_ACCESS_KIND', ['local_store', 'tmem_load'])`. Parameters: `FAILURE`, `MEM_ACCESS_KIND`, `device`, `run_wrapper`, `monkeypatch`, `num_ctas`. Nested definitions in this scope: `kernel`. Key calls include `pytest.mark.skipif`, `pytest.mark.parametrize`, `monkeypatch.setenv`, `knobs.refresh_knobs`, `torch.empty_like`, `run_in_process`, and 24 more. This scope touches pytest parametrization, PyTorch tensor setup and checks, runtime driver interaction, kernel launch orchestration.
- **CN:** 定义测试函数 `test_tcgen5_copy`。 装饰器：`pytest.mark.skipif(not is_cuda() or torch.cuda.get_device_capability()[0] < 10, reason='Requires blackwell or newer')`、`pytest.mark.parametrize('FAILURE', [True, False])`、`pytest.mark.parametrize('MEM_ACCESS_KIND', ['local_store', 'tmem_load'])`。 参数：`FAILURE`、`MEM_ACCESS_KIND`、`device`、`run_wrapper`、`monkeypatch`、`num_ctas`。 该作用域中的嵌套定义：`kernel`。 关键调用包括 `pytest.mark.skipif`、`pytest.mark.parametrize`、`monkeypatch.setenv`、`knobs.refresh_knobs`、`torch.empty_like`、`run_in_process` 等另外 24 项。 该作用域涉及pytest 参数化、PyTorch 张量准备与校验、运行时驱动交互、kernel 启动编排。

#### Lines 1024-1035

```python
    if run_wrapper:
        result = run_in_process(test_tcgen5_copy, (FAILURE, MEM_ACCESS_KIND, device, False, monkeypatch, num_ctas))
        if FAILURE:
            assert_expected_cuda_failure(result.exc)
            if MEM_ACCESS_KIND == "local_store":
                assert "Buffer being accessed has outstanding reads" in result.driver_stderr_output
            else:
                assert "Buffer being accessed has outstanding writes" in result.driver_stderr_output
        else:
            assert result.exc is None
            assert result.driver_stderr_output == ""
        return
```
- **EN:** Invokes `run_in_process`, `assert_expected_cuda_failure` to execute the test logic. Validates behavior with 4 assertion(s). Branches on runtime or test conditions. Relevant themes: runtime driver interaction.
- **CN:** 调用 `run_in_process`、`assert_expected_cuda_failure` 执行测试逻辑。 通过 4 个断言验证行为。 根据运行时或测试条件进行分支。 相关主题：运行时驱动交互。

#### Lines 1036-1039

```python

    monkeypatch.setenv("TRITON_INSTRUMENTATION_MODE", "consan")
    monkeypatch.setenv("CUDA_LAUNCH_BLOCKING", "1")
    knobs.refresh_knobs()
```
- **EN:** Invokes `monkeypatch.setenv`, `knobs.refresh_knobs` to execute the test logic. Relevant themes: kernel launch orchestration.
- **CN:** 调用 `monkeypatch.setenv`、`knobs.refresh_knobs` 执行测试逻辑。 相关主题：kernel 启动编排。

#### Lines 1040-1042

```python

    @gluon.jit
    def kernel(input, output, FAILURE: ttgl.constexpr, MEM_ACCESS_KIND: ttgl.constexpr):
```
- **EN:** Defines the helper function `kernel`. Decorators: `gluon.jit`. Parameters: `input`, `output`, `FAILURE`, `MEM_ACCESS_KIND`. Key calls include `default_cga_layout`, `blackwell.TensorMemoryLayout`, `blackwell.allocate_tensor_memory`, `tmem.get_reg_layout`, `ttgl.load`, `ttgl.NVMMASharedLayout`, and 14 more. This scope touches layout transformation reasoning.
- **CN:** 定义辅助函数 `kernel`。 装饰器：`gluon.jit`。 参数：`input`、`output`、`FAILURE`、`MEM_ACCESS_KIND`。 关键调用包括 `default_cga_layout`、`blackwell.TensorMemoryLayout`、`blackwell.allocate_tensor_memory`、`tmem.get_reg_layout`、`ttgl.load`、`ttgl.NVMMASharedLayout` 等另外 14 项。 该作用域涉及布局变换推理。

##### Lines 1043-1059

```python
        block_m: ttgl.constexpr = XBLOCK * ttgl.num_ctas()
        cga_layout: ttgl.constexpr = default_cga_layout(ttgl.num_ctas(), 2)
        tmem_layout: ttgl.constexpr = blackwell.TensorMemoryLayout((128, XBLOCK), col_stride=1, cga_layout=cga_layout)
        tmem = blackwell.allocate_tensor_memory(ttgl.int32, [block_m, XBLOCK], tmem_layout)
        reg_layout: ttgl.constexpr = tmem.get_reg_layout()
        offs_m = ttgl.arange(0, block_m, ttgl.SliceLayout(1, reg_layout))[:, None]
        offs_n = ttgl.arange(0, XBLOCK, ttgl.SliceLayout(0, reg_layout))[None, :]
        offs = offs_m * XBLOCK + offs_n
        val = ttgl.load(input + offs)
        smem_layout: ttgl.constexpr = ttgl.NVMMASharedLayout(swizzle_byte_width=128, element_bitwidth=32, rank=2,
                                                             cga_layout=cga_layout)
        smem = ttgl.allocate_shared_memory(ttgl.int32, [block_m, XBLOCK], smem_layout)
        smem.store(val)
        bar = mbarrier.allocate_mbarrier()
        mbarrier.init(bar, count=1)
        blackwell.tcgen05_copy(smem, tmem)
        blackwell.tcgen05_commit(bar)
```
- **EN:** Prepares or updates state through `block_m`, `cga_layout`, `tmem_layout`, `tmem`, `reg_layout`, `offs_m`, `offs_n`, `offs`, and 4 more. Invokes `ttgl.num_ctas`, `default_cga_layout`, `blackwell.TensorMemoryLayout`, `blackwell.allocate_tensor_memory`, `tmem.get_reg_layout`, `ttgl.arange`, and 9 more to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `block_m`、`cga_layout`、`tmem_layout`、`tmem`、`reg_layout`、`offs_m`、`offs_n`、`offs` 等另外 4 项 准备或更新状态。 调用 `ttgl.num_ctas`、`default_cga_layout`、`blackwell.TensorMemoryLayout`、`blackwell.allocate_tensor_memory`、`tmem.get_reg_layout`、`ttgl.arange` 等另外 9 项 执行测试逻辑。 相关主题：布局变换推理。

##### Lines 1060-1061

```python
        if not FAILURE:
            mbarrier.wait(bar, 0)
```
- **EN:** Invokes `mbarrier.wait` to execute the test logic. Branches on runtime or test conditions.
- **CN:** 调用 `mbarrier.wait` 执行测试逻辑。 根据运行时或测试条件进行分支。

##### Lines 1062-1066

```python
        if MEM_ACCESS_KIND == "local_store":
            smem.store(ttgl.zeros([block_m, XBLOCK], ttgl.int32, reg_layout))
        else:
            val = tmem.load(reg_layout)
            ttgl.store(output + offs, val)
```
- **EN:** Invokes `smem.store`, `tmem.load`, `ttgl.store`, `ttgl.zeros` to execute the test logic. Branches on runtime or test conditions. Relevant themes: layout transformation reasoning.
- **CN:** 调用 `smem.store`、`tmem.load`、`ttgl.store`、`ttgl.zeros` 执行测试逻辑。 根据运行时或测试条件进行分支。 相关主题：布局变换推理。

##### Lines 1067-1068

```python
        if FAILURE:
            mbarrier.wait(bar, 0)
```
- **EN:** Invokes `mbarrier.wait` to execute the test logic. Branches on runtime or test conditions.
- **CN:** 调用 `mbarrier.wait` 执行测试逻辑。 根据运行时或测试条件进行分支。

##### Lines 1069-1069

```python
        mbarrier.invalidate(bar)
```
- **EN:** Invokes `mbarrier.invalidate` to execute the test logic.
- **CN:** 调用 `mbarrier.invalidate` 执行测试逻辑。

#### Lines 1070-1074

```python

    input = torch.arange(XBLOCK.value * XBLOCK.value * num_ctas, device=device,
                         dtype=torch.int32).reshape(XBLOCK.value * num_ctas, XBLOCK.value)
    output = torch.empty_like(input)
    kernel[(1, )](input, output, FAILURE=FAILURE, MEM_ACCESS_KIND=MEM_ACCESS_KIND, num_warps=4, num_ctas=num_ctas)
```
- **EN:** Prepares or updates state through `input`, `output`. Invokes `torch.arange`, `torch.empty_like` to execute the test logic. Relevant themes: PyTorch tensor setup and checks.
- **CN:** 通过 `input`、`output` 准备或更新状态。 调用 `torch.arange`、`torch.empty_like` 执行测试逻辑。 相关主题：PyTorch 张量准备与校验。

### Lines 1075-1079

```python


@pytest.mark.skipif(not is_cuda() or torch.cuda.get_device_capability()[0] != 9, reason="Requires hopper")
@pytest.mark.parametrize("FAILURE", [True, False])
def test_warpgroup_mma(FAILURE, device, run_wrapper, monkeypatch, num_ctas):
```
- **EN:** Defines the test function `test_warpgroup_mma`. Decorators: `pytest.mark.skipif(not is_cuda() or torch.cuda.get_device_capability()[0] != 9, reason='Requires hopper')`, `pytest.mark.parametrize('FAILURE', [True, False])`. Parameters: `FAILURE`, `device`, `run_wrapper`, `monkeypatch`, `num_ctas`. Nested definitions in this scope: `kernel`. Key calls include `pytest.mark.skipif`, `pytest.mark.parametrize`, `monkeypatch.setenv`, `knobs.refresh_knobs`, `torch.randn`, `run_in_process`, and 16 more. This scope touches pytest parametrization, PyTorch tensor setup and checks, runtime driver interaction, kernel launch orchestration.
- **CN:** 定义测试函数 `test_warpgroup_mma`。 装饰器：`pytest.mark.skipif(not is_cuda() or torch.cuda.get_device_capability()[0] != 9, reason='Requires hopper')`、`pytest.mark.parametrize('FAILURE', [True, False])`。 参数：`FAILURE`、`device`、`run_wrapper`、`monkeypatch`、`num_ctas`。 该作用域中的嵌套定义：`kernel`。 关键调用包括 `pytest.mark.skipif`、`pytest.mark.parametrize`、`monkeypatch.setenv`、`knobs.refresh_knobs`、`torch.randn`、`run_in_process` 等另外 16 项。 该作用域涉及pytest 参数化、PyTorch 张量准备与校验、运行时驱动交互、kernel 启动编排。

#### Lines 1080-1088

```python
    if run_wrapper:
        result = run_in_process(test_warpgroup_mma, (FAILURE, device, False, monkeypatch, num_ctas))
        if FAILURE:
            assert_expected_cuda_failure(result.exc)
            assert "Accessing buffer with pending access. Pending access type: warpgroup_mma operand read" in result.driver_stderr_output
        else:
            assert result.exc is None
            assert result.driver_stderr_output == ""
        return
```
- **EN:** Invokes `run_in_process`, `assert_expected_cuda_failure` to execute the test logic. Validates behavior with 3 assertion(s). Branches on runtime or test conditions. Relevant themes: runtime driver interaction, random-data generation.
- **CN:** 调用 `run_in_process`、`assert_expected_cuda_failure` 执行测试逻辑。 通过 3 个断言验证行为。 根据运行时或测试条件进行分支。 相关主题：运行时驱动交互、随机数据生成。

#### Lines 1089-1092

```python

    monkeypatch.setenv("TRITON_INSTRUMENTATION_MODE", "consan")
    monkeypatch.setenv("CUDA_LAUNCH_BLOCKING", "1")
    knobs.refresh_knobs()
```
- **EN:** Invokes `monkeypatch.setenv`, `knobs.refresh_knobs` to execute the test logic. Relevant themes: kernel launch orchestration.
- **CN:** 调用 `monkeypatch.setenv`、`knobs.refresh_knobs` 执行测试逻辑。 相关主题：kernel 启动编排。

#### Lines 1093-1095

```python

    @gluon.jit
    def kernel(input, FAILURE: ttgl.constexpr):
```
- **EN:** Defines the helper function `kernel`. Decorators: `gluon.jit`. Parameters: `input`, `FAILURE`. Key calls include `mma_block_m`, `mma_block_n`, `mma_cga_layout`, `ttgl.NVMMASharedLayout`, `ttgl.allocate_shared_memory`, `ttgl.BlockedLayout`, and 7 more. This scope touches layout transformation reasoning.
- **CN:** 定义辅助函数 `kernel`。 装饰器：`gluon.jit`。 参数：`input`、`FAILURE`。 关键调用包括 `mma_block_m`、`mma_block_n`、`mma_cga_layout`、`ttgl.NVMMASharedLayout`、`ttgl.allocate_shared_memory`、`ttgl.BlockedLayout` 等另外 7 项。 该作用域涉及布局变换推理。

##### Lines 1096-1114

```python
        block_m: ttgl.constexpr = mma_block_m(ttgl.num_ctas())
        block_n: ttgl.constexpr = mma_block_n(ttgl.num_ctas())
        cga_layout_a: ttgl.constexpr = mma_cga_layout(ttgl.num_ctas(), 0)
        cga_layout_b: ttgl.constexpr = mma_cga_layout(ttgl.num_ctas(), 1)
        cga_layout_c: ttgl.constexpr = mma_cga_layout(ttgl.num_ctas(), 2)
        smem_layout_a: ttgl.constexpr = ttgl.NVMMASharedLayout(swizzle_byte_width=128, element_bitwidth=16, rank=2,
                                                               cga_layout=cga_layout_a)
        smem_layout_b: ttgl.constexpr = ttgl.NVMMASharedLayout(swizzle_byte_width=128, element_bitwidth=16, rank=2,
                                                               cga_layout=cga_layout_b)
        smemA = ttgl.allocate_shared_memory(ttgl.float16, [block_m, XBLOCK], smem_layout_a)
        smemB = ttgl.allocate_shared_memory(ttgl.float16, [XBLOCK, block_n], smem_layout_b)

        blocked_layout: ttgl.constexpr = ttgl.BlockedLayout(size_per_thread=[1, XBLOCK], threads_per_warp=[32, 1],
                                                            warps_per_cta=[4, 1], order=[0, 1], cga_layout=cga_layout_a)

        acc_layout: ttgl.constexpr = ttgl.NVMMADistributedLayout(version=[3, 0], warps_per_cta=[4, 1],
                                                                 instr_shape=[16, 32, 16], cga_layout=cga_layout_c)
        acc = ttgl.zeros([block_m, block_n], ttgl.float16, acc_layout)
        acc = hopper.warpgroup_mma(smemA, smemB, acc, is_async=True)
```
- **EN:** Prepares or updates state through `block_m`, `block_n`, `cga_layout_a`, `cga_layout_b`, `cga_layout_c`, `smem_layout_a`, `smem_layout_b`, `smemA`, and 4 more. Invokes `mma_block_m`, `ttgl.num_ctas`, `mma_block_n`, `mma_cga_layout`, `ttgl.NVMMASharedLayout`, `ttgl.allocate_shared_memory`, and 4 more to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `block_m`、`block_n`、`cga_layout_a`、`cga_layout_b`、`cga_layout_c`、`smem_layout_a`、`smem_layout_b`、`smemA` 等另外 4 项 准备或更新状态。 调用 `mma_block_m`、`ttgl.num_ctas`、`mma_block_n`、`mma_cga_layout`、`ttgl.NVMMASharedLayout`、`ttgl.allocate_shared_memory` 等另外 4 项 执行测试逻辑。 相关主题：布局变换推理。

##### Lines 1115-1116

```python
        if FAILURE:
            smemA.store(ttgl.full([block_m, XBLOCK], 42, ttgl.float16, blocked_layout))
```
- **EN:** Invokes `smemA.store`, `ttgl.full` to execute the test logic. Branches on runtime or test conditions. Relevant themes: layout transformation reasoning.
- **CN:** 调用 `smemA.store`、`ttgl.full` 执行测试逻辑。 根据运行时或测试条件进行分支。 相关主题：布局变换推理。

##### Lines 1117-1118

```python
        hopper.warpgroup_mma_wait(num_outstanding=0, deps=[acc])
        smemA.store(ttgl.full([block_m, XBLOCK], 42, ttgl.float16, blocked_layout))
```
- **EN:** Invokes `hopper.warpgroup_mma_wait`, `smemA.store`, `ttgl.full` to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 调用 `hopper.warpgroup_mma_wait`、`smemA.store`、`ttgl.full` 执行测试逻辑。 相关主题：布局变换推理。

#### Lines 1119-1121

```python

    input = torch.randn((XBLOCK, XBLOCK), device=device, dtype=torch.float16)
    kernel[(1, )](input, FAILURE=FAILURE, num_ctas=num_ctas)
```
- **EN:** Prepares or updates state through `input`. Invokes `torch.randn` to execute the test logic. Relevant themes: PyTorch tensor setup and checks, random-data generation.
- **CN:** 通过 `input` 准备或更新状态。 调用 `torch.randn` 执行测试逻辑。 相关主题：PyTorch 张量准备与校验、随机数据生成。

### Lines 1122-1126

```python


@pytest.mark.skipif(not is_cuda() or torch.cuda.get_device_capability()[0] != 9, reason="Requires hopper")
@pytest.mark.parametrize("FAILURE", [True, False])
def test_warpgroup_mma2(FAILURE, device, run_wrapper, monkeypatch, num_ctas):
```
- **EN:** Defines the test function `test_warpgroup_mma2`. Decorators: `pytest.mark.skipif(not is_cuda() or torch.cuda.get_device_capability()[0] != 9, reason='Requires hopper')`, `pytest.mark.parametrize('FAILURE', [True, False])`. Parameters: `FAILURE`, `device`, `run_wrapper`, `monkeypatch`, `num_ctas`. Nested definitions in this scope: `kernel`. Key calls include `pytest.mark.skipif`, `pytest.mark.parametrize`, `monkeypatch.setenv`, `knobs.refresh_knobs`, `torch.randn`, `run_in_process`, and 16 more. This scope touches pytest parametrization, PyTorch tensor setup and checks, runtime driver interaction, kernel launch orchestration.
- **CN:** 定义测试函数 `test_warpgroup_mma2`。 装饰器：`pytest.mark.skipif(not is_cuda() or torch.cuda.get_device_capability()[0] != 9, reason='Requires hopper')`、`pytest.mark.parametrize('FAILURE', [True, False])`。 参数：`FAILURE`、`device`、`run_wrapper`、`monkeypatch`、`num_ctas`。 该作用域中的嵌套定义：`kernel`。 关键调用包括 `pytest.mark.skipif`、`pytest.mark.parametrize`、`monkeypatch.setenv`、`knobs.refresh_knobs`、`torch.randn`、`run_in_process` 等另外 16 项。 该作用域涉及pytest 参数化、PyTorch 张量准备与校验、运行时驱动交互、kernel 启动编排。

#### Lines 1127-1135

```python
    if run_wrapper:
        result = run_in_process(test_warpgroup_mma2, (FAILURE, device, False, monkeypatch, num_ctas))
        if FAILURE:
            assert_expected_cuda_failure(result.exc)
            assert "Accessing buffer with pending access. Pending access type: warpgroup_mma operand read" in result.driver_stderr_output
        else:
            assert result.exc is None
            assert result.driver_stderr_output == ""
        return
```
- **EN:** Invokes `run_in_process`, `assert_expected_cuda_failure` to execute the test logic. Validates behavior with 3 assertion(s). Branches on runtime or test conditions. Relevant themes: runtime driver interaction, random-data generation.
- **CN:** 调用 `run_in_process`、`assert_expected_cuda_failure` 执行测试逻辑。 通过 3 个断言验证行为。 根据运行时或测试条件进行分支。 相关主题：运行时驱动交互、随机数据生成。

#### Lines 1136-1139

```python

    monkeypatch.setenv("TRITON_INSTRUMENTATION_MODE", "consan")
    monkeypatch.setenv("CUDA_LAUNCH_BLOCKING", "1")
    knobs.refresh_knobs()
```
- **EN:** Invokes `monkeypatch.setenv`, `knobs.refresh_knobs` to execute the test logic. Relevant themes: kernel launch orchestration.
- **CN:** 调用 `monkeypatch.setenv`、`knobs.refresh_knobs` 执行测试逻辑。 相关主题：kernel 启动编排。

#### Lines 1140-1142

```python

    @gluon.jit
    def kernel(input, FAILURE: ttgl.constexpr):
```
- **EN:** Defines the helper function `kernel`. Decorators: `gluon.jit`. Parameters: `input`, `FAILURE`. Key calls include `mma_block_m`, `mma_block_n`, `mma_cga_layout`, `ttgl.NVMMASharedLayout`, `ttgl.allocate_shared_memory`, `ttgl.BlockedLayout`, and 7 more. This scope touches layout transformation reasoning.
- **CN:** 定义辅助函数 `kernel`。 装饰器：`gluon.jit`。 参数：`input`、`FAILURE`。 关键调用包括 `mma_block_m`、`mma_block_n`、`mma_cga_layout`、`ttgl.NVMMASharedLayout`、`ttgl.allocate_shared_memory`、`ttgl.BlockedLayout` 等另外 7 项。 该作用域涉及布局变换推理。

##### Lines 1143-1163

```python
        block_m: ttgl.constexpr = mma_block_m(ttgl.num_ctas())
        block_n: ttgl.constexpr = mma_block_n(ttgl.num_ctas())
        cga_layout_a: ttgl.constexpr = mma_cga_layout(ttgl.num_ctas(), 0)
        cga_layout_b: ttgl.constexpr = mma_cga_layout(ttgl.num_ctas(), 1)
        cga_layout_c: ttgl.constexpr = mma_cga_layout(ttgl.num_ctas(), 2)
        smem_layout_a: ttgl.constexpr = ttgl.NVMMASharedLayout(swizzle_byte_width=128, element_bitwidth=16, rank=2,
                                                               cga_layout=cga_layout_a)
        smem_layout_b: ttgl.constexpr = ttgl.NVMMASharedLayout(swizzle_byte_width=128, element_bitwidth=16, rank=2,
                                                               cga_layout=cga_layout_b)
        smemA = ttgl.allocate_shared_memory(ttgl.float16, [block_m, XBLOCK], smem_layout_a)
        smemB = ttgl.allocate_shared_memory(ttgl.float16, [XBLOCK, block_n], smem_layout_b)

        blocked_layout: ttgl.constexpr = ttgl.BlockedLayout(size_per_thread=[1, XBLOCK], threads_per_warp=[32, 1],
                                                            warps_per_cta=[4, 1], order=[0, 1], cga_layout=cga_layout_a)

        acc_layout: ttgl.constexpr = ttgl.NVMMADistributedLayout(version=[3, 0], warps_per_cta=[4, 1],
                                                                 instr_shape=[16, 32, 16], cga_layout=cga_layout_c)
        acc = ttgl.zeros([block_m, block_n], ttgl.float16, acc_layout)
        acc = hopper.warpgroup_mma(smemA, smemB, acc, is_async=True)
        acc = hopper.warpgroup_mma(smemA, smemB, acc, is_async=True)
        hopper.warpgroup_mma_wait(num_outstanding=1, deps=[acc])
```
- **EN:** Prepares or updates state through `block_m`, `block_n`, `cga_layout_a`, `cga_layout_b`, `cga_layout_c`, `smem_layout_a`, `smem_layout_b`, `smemA`, and 4 more. Invokes `mma_block_m`, `ttgl.num_ctas`, `mma_block_n`, `mma_cga_layout`, `ttgl.NVMMASharedLayout`, `ttgl.allocate_shared_memory`, and 5 more to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `block_m`、`block_n`、`cga_layout_a`、`cga_layout_b`、`cga_layout_c`、`smem_layout_a`、`smem_layout_b`、`smemA` 等另外 4 项 准备或更新状态。 调用 `mma_block_m`、`ttgl.num_ctas`、`mma_block_n`、`mma_cga_layout`、`ttgl.NVMMASharedLayout`、`ttgl.allocate_shared_memory` 等另外 5 项 执行测试逻辑。 相关主题：布局变换推理。

##### Lines 1164-1165

```python
        if FAILURE:
            smemA.store(ttgl.full([block_m, XBLOCK], 42, ttgl.float16, blocked_layout))
```
- **EN:** Invokes `smemA.store`, `ttgl.full` to execute the test logic. Branches on runtime or test conditions. Relevant themes: layout transformation reasoning.
- **CN:** 调用 `smemA.store`、`ttgl.full` 执行测试逻辑。 根据运行时或测试条件进行分支。 相关主题：布局变换推理。

##### Lines 1166-1167

```python
        hopper.warpgroup_mma_wait(num_outstanding=0, deps=[acc])
        smemA.store(ttgl.full([block_m, XBLOCK], 42, ttgl.float16, blocked_layout))
```
- **EN:** Invokes `hopper.warpgroup_mma_wait`, `smemA.store`, `ttgl.full` to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 调用 `hopper.warpgroup_mma_wait`、`smemA.store`、`ttgl.full` 执行测试逻辑。 相关主题：布局变换推理。

#### Lines 1168-1170

```python

    input = torch.randn((XBLOCK, XBLOCK), device=device, dtype=torch.float16)
    kernel[(1, )](input, FAILURE=FAILURE, num_ctas=num_ctas)
```
- **EN:** Prepares or updates state through `input`. Invokes `torch.randn` to execute the test logic. Relevant themes: PyTorch tensor setup and checks, random-data generation.
- **CN:** 通过 `input` 准备或更新状态。 调用 `torch.randn` 执行测试逻辑。 相关主题：PyTorch 张量准备与校验、随机数据生成。

### Lines 1171-1176

```python


@pytest.mark.skipif(not is_cuda() or torch.cuda.get_device_capability()[0] < 10, reason="Requires blackwell or newer")
@pytest.mark.parametrize("BUF_IDX", [0, 1])
@pytest.mark.parametrize("BAR_IDX", [0, 1, 2, 3])
def test_tcgen5_mma_multibar(BUF_IDX, BAR_IDX, device, run_wrapper, monkeypatch, num_ctas):
```
- **EN:** Defines the test function `test_tcgen5_mma_multibar`. Decorators: `pytest.mark.skipif(not is_cuda() or torch.cuda.get_device_capability()[0] < 10, reason='Requires blackwell or newer')`, `pytest.mark.parametrize('BUF_IDX', [0, 1])`, `pytest.mark.parametrize('BAR_IDX', [0, 1, 2, 3])`. Parameters: `BUF_IDX`, `BAR_IDX`, `device`, `run_wrapper`, `monkeypatch`, `num_ctas`. Nested definitions in this scope: `kernel`. Key calls include `pytest.mark.skipif`, `pytest.mark.parametrize`, `monkeypatch.setenv`, `knobs.refresh_knobs`, `mma_block_m`, `torch.randn`, and 23 more. This scope touches pytest parametrization, PyTorch tensor setup and checks, tensor/descriptor metadata, runtime driver interaction.
- **CN:** 定义测试函数 `test_tcgen5_mma_multibar`。 装饰器：`pytest.mark.skipif(not is_cuda() or torch.cuda.get_device_capability()[0] < 10, reason='Requires blackwell or newer')`、`pytest.mark.parametrize('BUF_IDX', [0, 1])`、`pytest.mark.parametrize('BAR_IDX', [0, 1, 2, 3])`。 参数：`BUF_IDX`、`BAR_IDX`、`device`、`run_wrapper`、`monkeypatch`、`num_ctas`。 该作用域中的嵌套定义：`kernel`。 关键调用包括 `pytest.mark.skipif`、`pytest.mark.parametrize`、`monkeypatch.setenv`、`knobs.refresh_knobs`、`mma_block_m`、`torch.randn` 等另外 23 项。 该作用域涉及pytest 参数化、PyTorch 张量准备与校验、张量/描述符元数据、运行时驱动交互。

#### Lines 1177-1178

```python
    if BAR_IDX == 0:
        pytest.skip("Skipping due to wait on false-predicated barrier - not supported yet")
```
- **EN:** Invokes `pytest.skip` to execute the test logic. Branches on runtime or test conditions.
- **CN:** 调用 `pytest.skip` 执行测试逻辑。 根据运行时或测试条件进行分支。

#### Lines 1179-1187

```python
    if run_wrapper:
        result = run_in_process(test_tcgen5_mma_multibar, (BUF_IDX, BAR_IDX, device, False, monkeypatch, num_ctas))
        if BAR_IDX // 2 < BUF_IDX:
            assert_expected_cuda_failure(result.exc)
            assert "Buffer being accessed has outstanding writes" in result.driver_stderr_output
        else:
            assert result.exc is None
            assert result.driver_stderr_output == ""
        return
```
- **EN:** Invokes `run_in_process`, `assert_expected_cuda_failure` to execute the test logic. Validates behavior with 3 assertion(s). Branches on runtime or test conditions. Relevant themes: runtime driver interaction.
- **CN:** 调用 `run_in_process`、`assert_expected_cuda_failure` 执行测试逻辑。 通过 3 个断言验证行为。 根据运行时或测试条件进行分支。 相关主题：运行时驱动交互。

#### Lines 1188-1190

```python
    monkeypatch.setenv("TRITON_INSTRUMENTATION_MODE", "consan")
    monkeypatch.setenv("CUDA_LAUNCH_BLOCKING", "1")
    knobs.refresh_knobs()
```
- **EN:** Invokes `monkeypatch.setenv`, `knobs.refresh_knobs` to execute the test logic. Relevant themes: kernel launch orchestration.
- **CN:** 调用 `monkeypatch.setenv`、`knobs.refresh_knobs` 执行测试逻辑。 相关主题：kernel 启动编排。

#### Lines 1191-1193

```python

    @gluon.jit
    def kernel(input_desc, BUF_IDX: ttgl.constexpr, BAR_IDX: ttgl.constexpr):
```
- **EN:** Defines the helper function `kernel`. Decorators: `gluon.jit`. Parameters: `input_desc`, `BUF_IDX`, `BAR_IDX`. Key calls include `mma_block_m`, `mma_block_n`, `blackwell.TensorMemoryLayout`, `ttgl.BlockedLayout`, `ttgl.allocate_shared_memory`, `mbarrier.allocate_mbarrier`, and 12 more. This scope touches layout transformation reasoning.
- **CN:** 定义辅助函数 `kernel`。 装饰器：`gluon.jit`。 参数：`input_desc`、`BUF_IDX`、`BAR_IDX`。 关键调用包括 `mma_block_m`、`mma_block_n`、`blackwell.TensorMemoryLayout`、`ttgl.BlockedLayout`、`ttgl.allocate_shared_memory`、`mbarrier.allocate_mbarrier` 等另外 12 项。 该作用域涉及布局变换推理。

##### Lines 1194-1212

```python
        block_m: ttgl.constexpr = mma_block_m(ttgl.num_ctas())
        block_n: ttgl.constexpr = mma_block_n(ttgl.num_ctas())
        acc_layout: ttgl.constexpr = blackwell.TensorMemoryLayout(
            [XBLOCK, XBLOCK],
            col_stride=1,
            cga_layout=mma_cga_layout(ttgl.num_ctas(), 2),
        )
        acc_blocked_layout: ttgl.constexpr = ttgl.BlockedLayout(size_per_thread=[1, XBLOCK], threads_per_warp=[32, 1],
                                                                warps_per_cta=[4, 1], order=[0, 1],
                                                                cga_layout=acc_layout.cga_layout)
        smemA = ttgl.allocate_shared_memory(ttgl.float16, [block_m, XBLOCK], input_desc.layout)
        smemB = ttgl.allocate_shared_memory(
            ttgl.float16,
            [XBLOCK, block_n],
            ttgl.NVMMASharedLayout.get_default_for([XBLOCK, block_n], ttgl.float16,
                                                   cga_layout=mma_cga_layout(ttgl.num_ctas(), 1)),
        )
        bar = mbarrier.allocate_mbarrier(batch=4)
        acc = blackwell.allocate_tensor_memory(ttgl.float32, [2, block_m, block_n], acc_layout)
```
- **EN:** Prepares or updates state through `block_m`, `block_n`, `acc_layout`, `acc_blocked_layout`, `smemA`, `smemB`, `bar`, `acc`. Invokes `mma_block_m`, `ttgl.num_ctas`, `mma_block_n`, `blackwell.TensorMemoryLayout`, `mma_cga_layout`, `ttgl.BlockedLayout`, and 4 more to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `block_m`、`block_n`、`acc_layout`、`acc_blocked_layout`、`smemA`、`smemB`、`bar`、`acc` 准备或更新状态。 调用 `mma_block_m`、`ttgl.num_ctas`、`mma_block_n`、`blackwell.TensorMemoryLayout`、`mma_cga_layout`、`ttgl.BlockedLayout` 等另外 4 项 执行测试逻辑。 相关主题：布局变换推理。

##### Lines 1213-1214

```python
        for i in range(4):
            mbarrier.init(bar.index(i), count=1)
```
- **EN:** Invokes `mbarrier.init`, `bar.index` to execute the test logic. Iterates across cases or data tiles.
- **CN:** 调用 `mbarrier.init`、`bar.index` 执行测试逻辑。 通过循环覆盖多个用例或数据分块。

##### Lines 1215-1224

```python

        blackwell.tcgen05_mma(smemA, smemB, acc.index(0), mbarriers=[bar.index(0), bar.index(1)],
                              mbarrier_preds=[False, True])
        blackwell.tcgen05_mma(smemA, smemB, acc.index(1), mbarriers=[bar.index(2)])
        blackwell.tcgen05_commit(bar.index(3))

        mbarrier.wait(bar.index(BAR_IDX), 0)

        store_shape: ttgl.constexpr = [block_m, block_n]
        acc.index(BUF_IDX).store(ttgl.full(store_shape, 42, ttgl.float32, acc_blocked_layout))
```
- **EN:** Prepares or updates state through `store_shape`. Invokes `blackwell.tcgen05_mma`, `acc.index`, `bar.index`, `blackwell.tcgen05_commit`, `mbarrier.wait`, `ttgl.full` to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `store_shape` 准备或更新状态。 调用 `blackwell.tcgen05_mma`、`acc.index`、`bar.index`、`blackwell.tcgen05_commit`、`mbarrier.wait`、`ttgl.full` 执行测试逻辑。 相关主题：布局变换推理。

##### Lines 1225-1227

```python

        for i in range(4):
            mbarrier.invalidate(bar.index(i))
```
- **EN:** Invokes `mbarrier.invalidate`, `bar.index` to execute the test logic. Iterates across cases or data tiles.
- **CN:** 调用 `mbarrier.invalidate`、`bar.index` 执行测试逻辑。 通过循环覆盖多个用例或数据分块。

#### Lines 1228-1234

```python

    block_m = mma_block_m(num_ctas)
    input = torch.randn((block_m, XBLOCK.value), device=device, dtype=torch.float16)
    shared_layout = ttgl.NVMMASharedLayout.get_default_for([block_m, XBLOCK.value], ttgl.float16,
                                                           cga_layout=mma_cga_layout(num_ctas, 0))
    input_desc = gluon.nvidia.hopper.TensorDescriptor.from_tensor(input, [block_m, XBLOCK.value], shared_layout)
    kernel[(1, )](input_desc, BUF_IDX, BAR_IDX, num_warps=4, num_ctas=num_ctas)
```
- **EN:** Prepares or updates state through `block_m`, `input`, `shared_layout`, `input_desc`. Invokes `mma_block_m`, `torch.randn`, `ttgl.NVMMASharedLayout.get_default_for`, `mma_cga_layout`, `gluon.nvidia.hopper.TensorDescriptor.from_tensor` to execute the test logic. Relevant themes: PyTorch tensor setup and checks, tensor/descriptor metadata, layout transformation reasoning, random-data generation.
- **CN:** 通过 `block_m`、`input`、`shared_layout`、`input_desc` 准备或更新状态。 调用 `mma_block_m`、`torch.randn`、`ttgl.NVMMASharedLayout.get_default_for`、`mma_cga_layout`、`gluon.nvidia.hopper.TensorDescriptor.from_tensor` 执行测试逻辑。 相关主题：PyTorch 张量准备与校验、张量/描述符元数据、布局变换推理、随机数据生成。

### Lines 1235-1238

```python


@gluon.jit
def inc_mod(x, mod):
```
- **EN:** Defines the helper function `inc_mod`. Decorators: `gluon.jit`. Parameters: `x`, `mod`.
- **CN:** 定义辅助函数 `inc_mod`。 装饰器：`gluon.jit`。 参数：`x`、`mod`。

#### Lines 1239-1239

```python
    return (x + 1) % mod
```
- **EN:** Carries supporting logic for the surrounding test or helper scope.
- **CN:** 为周围的测试或辅助作用域提供支撑逻辑。

### Lines 1240-1244

```python


@pytest.mark.skipif(not is_cuda() or torch.cuda.get_device_capability()[0] < 10, reason="Requires blackwell or newer")
@pytest.mark.parametrize("FAILURE", [True, False])
def test_multibuffered_loop(FAILURE, device, run_wrapper, monkeypatch, num_ctas):
```
- **EN:** Defines the test function `test_multibuffered_loop`. Decorators: `pytest.mark.skipif(not is_cuda() or torch.cuda.get_device_capability()[0] < 10, reason='Requires blackwell or newer')`, `pytest.mark.parametrize('FAILURE', [True, False])`. Parameters: `FAILURE`, `device`, `run_wrapper`, `monkeypatch`, `num_ctas`. Nested definitions in this scope: `kernel`. Key calls include `pytest.mark.skipif`, `pytest.mark.parametrize`, `monkeypatch.setenv`, `knobs.refresh_knobs`, `mma_block_m`, `mma_block_n`, and 27 more. This scope touches pytest parametrization, PyTorch tensor setup and checks, tensor/descriptor metadata, plugin or compiler extension points.
- **CN:** 定义测试函数 `test_multibuffered_loop`。 装饰器：`pytest.mark.skipif(not is_cuda() or torch.cuda.get_device_capability()[0] < 10, reason='Requires blackwell or newer')`、`pytest.mark.parametrize('FAILURE', [True, False])`。 参数：`FAILURE`、`device`、`run_wrapper`、`monkeypatch`、`num_ctas`。 该作用域中的嵌套定义：`kernel`。 关键调用包括 `pytest.mark.skipif`、`pytest.mark.parametrize`、`monkeypatch.setenv`、`knobs.refresh_knobs`、`mma_block_m`、`mma_block_n` 等另外 27 项。 该作用域涉及pytest 参数化、PyTorch 张量准备与校验、张量/描述符元数据、插件或编译器扩展点。

#### Lines 1245-1253

```python
    if run_wrapper:
        result = run_in_process(test_multibuffered_loop, (FAILURE, device, False, monkeypatch, num_ctas))
        if FAILURE:
            assert_expected_cuda_failure(result.exc)
            assert "Buffer being accessed has outstanding reads" in result.driver_stderr_output
        else:
            assert result.exc is None
            assert result.driver_stderr_output == ""
        return
```
- **EN:** Invokes `run_in_process`, `assert_expected_cuda_failure` to execute the test logic. Validates behavior with 3 assertion(s). Branches on runtime or test conditions. Relevant themes: runtime driver interaction.
- **CN:** 调用 `run_in_process`、`assert_expected_cuda_failure` 执行测试逻辑。 通过 3 个断言验证行为。 根据运行时或测试条件进行分支。 相关主题：运行时驱动交互。

#### Lines 1254-1257

```python

    monkeypatch.setenv("TRITON_INSTRUMENTATION_MODE", "consan")
    monkeypatch.setenv("CUDA_LAUNCH_BLOCKING", "1")
    knobs.refresh_knobs()
```
- **EN:** Invokes `monkeypatch.setenv`, `knobs.refresh_knobs` to execute the test logic. Relevant themes: kernel launch orchestration.
- **CN:** 调用 `monkeypatch.setenv`、`knobs.refresh_knobs` 执行测试逻辑。 相关主题：kernel 启动编排。

#### Lines 1258-1260

```python

    @gluon.jit
    def kernel(a_desc, b_desc, FAILURE: ttgl.constexpr):
```
- **EN:** Defines the helper function `kernel`. Decorators: `gluon.jit`. Parameters: `a_desc`, `b_desc`, `FAILURE`. Key calls include `mma_block_m`, `mma_block_n`, `blackwell.TensorMemoryLayout`, `ttgl.BlockedLayout`, `ttgl.zeros`, `ttgl.NVMMASharedLayout.get_default_for`, and 17 more. This scope touches plugin or compiler extension points, layout transformation reasoning.
- **CN:** 定义辅助函数 `kernel`。 装饰器：`gluon.jit`。 参数：`a_desc`、`b_desc`、`FAILURE`。 关键调用包括 `mma_block_m`、`mma_block_n`、`blackwell.TensorMemoryLayout`、`ttgl.BlockedLayout`、`ttgl.zeros`、`ttgl.NVMMASharedLayout.get_default_for` 等另外 17 项。 该作用域涉及插件或编译器扩展点、布局变换推理。

##### Lines 1261-1281

```python
        num_buffers: ttgl.constexpr = 2 if FAILURE else 3
        num_mma_stages: ttgl.constexpr = 2
        block_m: ttgl.constexpr = mma_block_m(ttgl.num_ctas())
        block_n: ttgl.constexpr = mma_block_n(ttgl.num_ctas())

        acc_layout: ttgl.constexpr = blackwell.TensorMemoryLayout([XBLOCK, XBLOCK], col_stride=1,
                                                                  cga_layout=mma_cga_layout(ttgl.num_ctas(), 2))
        zero_layout: ttgl.constexpr = ttgl.BlockedLayout(size_per_thread=[1, XBLOCK], threads_per_warp=[32, 1],
                                                         warps_per_cta=[4, 1], order=[0, 1],
                                                         cga_layout=mma_cga_layout(ttgl.num_ctas(), 2))
        zero = ttgl.zeros([block_m, block_n], ttgl.float32, zero_layout)
        b_smem_layout: ttgl.constexpr = ttgl.NVMMASharedLayout.get_default_for([XBLOCK, block_n], ttgl.float16,
                                                                               cga_layout=mma_cga_layout(
                                                                                   ttgl.num_ctas(), 1))

        smemA = ttgl.allocate_shared_memory(ttgl.float16, [num_buffers, block_m, XBLOCK], a_desc.layout)
        smemB = ttgl.allocate_shared_memory(ttgl.float16, [num_buffers, XBLOCK, block_n], b_smem_layout)
        barLoadA = mbarrier.allocate_mbarrier(batch=num_buffers)
        barLoadB = mbarrier.allocate_mbarrier(batch=num_buffers)
        barMMA = mbarrier.allocate_mbarrier(batch=num_mma_stages)
        acc = blackwell.allocate_tensor_memory(ttgl.float32, [block_m, block_n], acc_layout, zero)
```
- **EN:** Prepares or updates state through `num_buffers`, `num_mma_stages`, `block_m`, `block_n`, `acc_layout`, `zero_layout`, `zero`, `b_smem_layout`, and 6 more. Invokes `mma_block_m`, `ttgl.num_ctas`, `mma_block_n`, `blackwell.TensorMemoryLayout`, `mma_cga_layout`, `ttgl.BlockedLayout`, and 5 more to execute the test logic. Relevant themes: plugin or compiler extension points, layout transformation reasoning.
- **CN:** 通过 `num_buffers`、`num_mma_stages`、`block_m`、`block_n`、`acc_layout`、`zero_layout`、`zero`、`b_smem_layout` 等另外 6 项 准备或更新状态。 调用 `mma_block_m`、`ttgl.num_ctas`、`mma_block_n`、`blackwell.TensorMemoryLayout`、`mma_cga_layout`、`ttgl.BlockedLayout` 等另外 5 项 执行测试逻辑。 相关主题：插件或编译器扩展点、布局变换推理。

##### Lines 1282-1284

```python
        for i in range(num_buffers):
            mbarrier.init(barLoadA.index(i), count=1)
            mbarrier.init(barLoadB.index(i), count=1)
```
- **EN:** Invokes `mbarrier.init`, `barLoadA.index`, `barLoadB.index` to execute the test logic. Iterates across cases or data tiles.
- **CN:** 调用 `mbarrier.init`、`barLoadA.index`、`barLoadB.index` 执行测试逻辑。 通过循环覆盖多个用例或数据分块。

##### Lines 1285-1287

```python

        for i in range(num_mma_stages):
            mbarrier.init(barMMA.index(i), count=1)
```
- **EN:** Invokes `mbarrier.init`, `barMMA.index` to execute the test logic. Iterates across cases or data tiles. Relevant themes: plugin or compiler extension points.
- **CN:** 调用 `mbarrier.init`、`barMMA.index` 执行测试逻辑。 通过循环覆盖多个用例或数据分块。 相关主题：插件或编译器扩展点。

##### Lines 1288-1320

```python

        phase = 0
        mma_phase = 0
        ins_id = 0
        ext_id = 0
        mma_id = 0
        wait_id = 0

        # ins_id = 0
        mbarrier.expect(barLoadA.index(ins_id), a_desc.nbytes_per_cta)
        tma.async_load(a_desc, [0, 0], barLoadA.index(ins_id), smemA.index(ins_id))

        mbarrier.expect(barLoadB.index(ins_id), b_desc.nbytes_per_cta)
        tma.async_load(b_desc, [0, 0], barLoadB.index(ins_id), smemB.index(ins_id))
        ins_id = inc_mod(ins_id, num_buffers)

        # ins_id = 1
        mbarrier.expect(barLoadA.index(ins_id), a_desc.nbytes_per_cta)
        tma.async_load(a_desc, [0, 0], barLoadA.index(ins_id), smemA.index(ins_id))

        mbarrier.expect(barLoadB.index(ins_id), b_desc.nbytes_per_cta)
        tma.async_load(b_desc, [0, 0], barLoadB.index(ins_id), smemB.index(ins_id))
        ins_id = inc_mod(ins_id, num_buffers)

        mbarrier.wait(barLoadA.index(ext_id), phase)
        mbarrier.wait(barLoadB.index(ext_id), phase)

        blackwell.tcgen05_mma(smemA.index(ext_id), smemB.index(ext_id), acc, mbarriers=[barMMA.index(mma_id)])
        ext_id = inc_mod(ext_id, num_buffers)
        mma_id = inc_mod(mma_id, num_mma_stages)

        # ins_id = 2
        ub = 10
```
- **EN:** Prepares or updates state through `phase`, `mma_phase`, `ins_id`, `ext_id`, `mma_id`, `wait_id`, `ub`. Invokes `mbarrier.expect`, `barLoadA.index`, `tma.async_load`, `smemA.index`, `barLoadB.index`, `smemB.index`, and 4 more to execute the test logic. Relevant themes: plugin or compiler extension points.
- **CN:** 通过 `phase`、`mma_phase`、`ins_id`、`ext_id`、`mma_id`、`wait_id`、`ub` 准备或更新状态。 调用 `mbarrier.expect`、`barLoadA.index`、`tma.async_load`、`smemA.index`、`barLoadB.index`、`smemB.index` 等另外 4 项 执行测试逻辑。 相关主题：插件或编译器扩展点。

##### Lines 1321-1343

```python
        for i in range(ub):
            if i < ub - 2:
                mbarrier.expect(barLoadA.index(ins_id), a_desc.nbytes_per_cta)
                tma.async_load(a_desc, [0, 0], barLoadA.index(ins_id), smemA.index(ins_id))

                mbarrier.expect(barLoadB.index(ins_id), b_desc.nbytes_per_cta)
                tma.async_load(b_desc, [0, 0], barLoadB.index(ins_id), smemB.index(ins_id))
                ins_id = inc_mod(ins_id, num_buffers)

            if i < ub - 1:
                mbarrier.wait(barLoadA.index(ext_id), phase)
                mbarrier.wait(barLoadB.index(ext_id), phase)

                blackwell.tcgen05_mma(smemA.index(ext_id), smemB.index(ext_id), acc, mbarriers=[barMMA.index(mma_id)])
                mma_id = inc_mod(mma_id, num_mma_stages)

            mbarrier.wait(barMMA.index(wait_id), mma_phase)
            wait_id = inc_mod(wait_id, num_mma_stages)
            if wait_id == 0:
                mma_phase = (mma_phase + 1) % 2
            ext_id = inc_mod(ext_id, num_buffers)
            if ext_id == 0:
                phase = (phase + 1) % 2
```
- **EN:** Invokes `mbarrier.wait`, `inc_mod`, `mbarrier.expect`, `tma.async_load`, `blackwell.tcgen05_mma`, `barMMA.index`, and 4 more to execute the test logic. Iterates across cases or data tiles. Relevant themes: plugin or compiler extension points.
- **CN:** 调用 `mbarrier.wait`、`inc_mod`、`mbarrier.expect`、`tma.async_load`、`blackwell.tcgen05_mma`、`barMMA.index` 等另外 4 项 执行测试逻辑。 通过循环覆盖多个用例或数据分块。 相关主题：插件或编译器扩展点。

##### Lines 1344-1347

```python

        for i in range(num_buffers):
            mbarrier.invalidate(barLoadA.index(i))
            mbarrier.invalidate(barLoadB.index(i))
```
- **EN:** Invokes `mbarrier.invalidate`, `barLoadA.index`, `barLoadB.index` to execute the test logic. Iterates across cases or data tiles.
- **CN:** 调用 `mbarrier.invalidate`、`barLoadA.index`、`barLoadB.index` 执行测试逻辑。 通过循环覆盖多个用例或数据分块。

##### Lines 1348-1350

```python

        for i in range(num_mma_stages):
            mbarrier.invalidate(barMMA.index(i))
```
- **EN:** Invokes `mbarrier.invalidate`, `barMMA.index` to execute the test logic. Iterates across cases or data tiles. Relevant themes: plugin or compiler extension points.
- **CN:** 调用 `mbarrier.invalidate`、`barMMA.index` 执行测试逻辑。 通过循环覆盖多个用例或数据分块。 相关主题：插件或编译器扩展点。

#### Lines 1351-1363

```python

    block_m = mma_block_m(num_ctas)
    block_n = mma_block_n(num_ctas)
    input = torch.randn((block_m, XBLOCK.value), device=device, dtype=torch.float16)
    a_desc = gluon.nvidia.hopper.TensorDescriptor.from_tensor(
        input, [block_m, XBLOCK.value],
        ttgl.NVMMASharedLayout.get_default_for([block_m, XBLOCK.value], ttgl.float16,
                                               cga_layout=mma_cga_layout(num_ctas, 0)))
    b_desc = gluon.nvidia.hopper.TensorDescriptor.from_tensor(
        input, [XBLOCK.value, block_n],
        ttgl.NVMMASharedLayout.get_default_for([XBLOCK.value, block_n], ttgl.float16,
                                               cga_layout=mma_cga_layout(num_ctas, 1)))
    kernel[(1, )](a_desc, b_desc, FAILURE=FAILURE, num_warps=4, num_ctas=num_ctas)
```
- **EN:** Prepares or updates state through `block_m`, `block_n`, `input`, `a_desc`, `b_desc`. Invokes `mma_block_m`, `mma_block_n`, `torch.randn`, `gluon.nvidia.hopper.TensorDescriptor.from_tensor`, `ttgl.NVMMASharedLayout.get_default_for`, `mma_cga_layout` to execute the test logic. Relevant themes: PyTorch tensor setup and checks, tensor/descriptor metadata, layout transformation reasoning, random-data generation.
- **CN:** 通过 `block_m`、`block_n`、`input`、`a_desc`、`b_desc` 准备或更新状态。 调用 `mma_block_m`、`mma_block_n`、`torch.randn`、`gluon.nvidia.hopper.TensorDescriptor.from_tensor`、`ttgl.NVMMASharedLayout.get_default_for`、`mma_cga_layout` 执行测试逻辑。 相关主题：PyTorch 张量准备与校验、张量/描述符元数据、布局变换推理、随机数据生成。

### Lines 1364-1368

```python


@pytest.mark.skipif(not is_cuda() or torch.cuda.get_device_capability()[0] < 10, reason="Requires blackwell or newer")
@pytest.mark.parametrize("FAILURE", [True, False])
def test_tma_tcgen05_mma_multicast_loop(FAILURE, device, run_wrapper, monkeypatch, num_ctas):
```
- **EN:** Defines the test function `test_tma_tcgen05_mma_multicast_loop`. Decorators: `pytest.mark.skipif(not is_cuda() or torch.cuda.get_device_capability()[0] < 10, reason='Requires blackwell or newer')`, `pytest.mark.parametrize('FAILURE', [True, False])`. Parameters: `FAILURE`, `device`, `run_wrapper`, `monkeypatch`, `num_ctas`. Nested definitions in this scope: `kernel`. Key calls include `pytest.mark.skipif`, `pytest.mark.parametrize`, `monkeypatch.setenv`, `knobs.refresh_knobs`, `mma_block_m`, `mma_block_n`, and 21 more. This scope touches pytest parametrization, PyTorch tensor setup and checks, tensor/descriptor metadata, runtime driver interaction.
- **CN:** 定义测试函数 `test_tma_tcgen05_mma_multicast_loop`。 装饰器：`pytest.mark.skipif(not is_cuda() or torch.cuda.get_device_capability()[0] < 10, reason='Requires blackwell or newer')`、`pytest.mark.parametrize('FAILURE', [True, False])`。 参数：`FAILURE`、`device`、`run_wrapper`、`monkeypatch`、`num_ctas`。 该作用域中的嵌套定义：`kernel`。 关键调用包括 `pytest.mark.skipif`、`pytest.mark.parametrize`、`monkeypatch.setenv`、`knobs.refresh_knobs`、`mma_block_m`、`mma_block_n` 等另外 21 项。 该作用域涉及pytest 参数化、PyTorch 张量准备与校验、张量/描述符元数据、运行时驱动交互。

#### Lines 1369-1370

```python
    if num_ctas == 1:
        pytest.skip("Need at least 2 CTAs for 2CTA mode in this test")
```
- **EN:** Invokes `pytest.skip` to execute the test logic. Branches on runtime or test conditions.
- **CN:** 调用 `pytest.skip` 执行测试逻辑。 根据运行时或测试条件进行分支。

#### Lines 1371-1379

```python
    if run_wrapper:
        result = run_in_process(test_tma_tcgen05_mma_multicast_loop, (FAILURE, device, False, monkeypatch, num_ctas))
        if FAILURE:
            assert_expected_cuda_failure(result.exc)
            assert "Buffer being accessed has outstanding" in result.driver_stderr_output
        else:
            assert result.exc is None
            assert result.driver_stderr_output == ""
        return
```
- **EN:** Invokes `run_in_process`, `assert_expected_cuda_failure` to execute the test logic. Validates behavior with 3 assertion(s). Branches on runtime or test conditions. Relevant themes: runtime driver interaction.
- **CN:** 调用 `run_in_process`、`assert_expected_cuda_failure` 执行测试逻辑。 通过 3 个断言验证行为。 根据运行时或测试条件进行分支。 相关主题：运行时驱动交互。

#### Lines 1380-1383

```python

    monkeypatch.setenv("TRITON_INSTRUMENTATION_MODE", "consan")
    monkeypatch.setenv("CUDA_LAUNCH_BLOCKING", "1")
    knobs.refresh_knobs()
```
- **EN:** Invokes `monkeypatch.setenv`, `knobs.refresh_knobs` to execute the test logic. Relevant themes: kernel launch orchestration.
- **CN:** 调用 `monkeypatch.setenv`、`knobs.refresh_knobs` 执行测试逻辑。 相关主题：kernel 启动编排。

#### Lines 1384-1386

```python

    @gluon.jit
    def kernel(a_desc, b_desc, FAILURE: ttgl.constexpr):
```
- **EN:** Defines the helper function `kernel`. Decorators: `gluon.jit`. Parameters: `a_desc`, `b_desc`, `FAILURE`. Key calls include `mma_block_m`, `mma_block_n`, `blackwell.TensorMemoryLayout`, `ttgl.allocate_shared_memory`, `blackwell.allocate_tensor_memory`, `mbarrier.allocate_mbarrier`, and 10 more. This scope touches layout transformation reasoning.
- **CN:** 定义辅助函数 `kernel`。 装饰器：`gluon.jit`。 参数：`a_desc`、`b_desc`、`FAILURE`。 关键调用包括 `mma_block_m`、`mma_block_n`、`blackwell.TensorMemoryLayout`、`ttgl.allocate_shared_memory`、`blackwell.allocate_tensor_memory`、`mbarrier.allocate_mbarrier` 等另外 10 项。 该作用域涉及布局变换推理。

##### Lines 1387-1412

```python
        num_k_tiles: ttgl.constexpr = 1 if FAILURE else 4
        block_m: ttgl.constexpr = mma_block_m(ttgl.num_ctas())
        block_n: ttgl.constexpr = mma_block_n(ttgl.num_ctas())
        acc_layout: ttgl.constexpr = blackwell.TensorMemoryLayout(
            [XBLOCK, XBLOCK],
            col_stride=1,
            cga_layout=mma_cga_layout(ttgl.num_ctas(), 2, True),
            two_ctas=True,
        )
        smemA = ttgl.allocate_shared_memory(ttgl.float16, [block_m, XBLOCK], a_desc.layout)
        smemB = ttgl.allocate_shared_memory(
            ttgl.float16,
            [XBLOCK, block_n],
            ttgl.NVMMASharedLayout.get_default_for([XBLOCK, block_n], ttgl.float16,
                                                   cga_layout=mma_cga_layout(ttgl.num_ctas(), 1, True)),
        )
        acc = blackwell.allocate_tensor_memory(ttgl.float32, [block_m, block_n], acc_layout)
        tma_bar = mbarrier.allocate_mbarrier(two_ctas=True)
        mbarrier.init(tma_bar, count=1)
        mma_bar = mbarrier.allocate_mbarrier()
        mma_bar_count: ttgl.constexpr = blackwell.tcgen05_mma_barrier_count([smemA, smemB], True,
                                                                            acc.type.layout.two_ctas)
        mbarrier.init(mma_bar, count=mma_bar_count)

        phase_tma = 0
        phase_mma = 0
```
- **EN:** Prepares or updates state through `num_k_tiles`, `block_m`, `block_n`, `acc_layout`, `smemA`, `smemB`, `acc`, `tma_bar`, and 4 more. Invokes `mma_block_m`, `ttgl.num_ctas`, `mma_block_n`, `blackwell.TensorMemoryLayout`, `mma_cga_layout`, `ttgl.allocate_shared_memory`, and 5 more to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `num_k_tiles`、`block_m`、`block_n`、`acc_layout`、`smemA`、`smemB`、`acc`、`tma_bar` 等另外 4 项 准备或更新状态。 调用 `mma_block_m`、`ttgl.num_ctas`、`mma_block_n`、`blackwell.TensorMemoryLayout`、`mma_cga_layout`、`ttgl.allocate_shared_memory` 等另外 5 项 执行测试逻辑。 相关主题：布局变换推理。

##### Lines 1413-1423

```python
        for k in range(num_k_tiles):
            offs_k = k * XBLOCK
            mbarrier.expect(tma_bar, a_desc.nbytes_per_cta + b_desc.nbytes_per_cta)
            tma.async_load(a_desc, [0, offs_k], tma_bar, smemA, multicast=True)
            tma.async_load(b_desc, [offs_k, 0], tma_bar, smemB, multicast=True)
            if not FAILURE:
                mbarrier.wait(tma_bar, phase_tma, deps=[smemA, smemB])
            blackwell.tcgen05_mma(smemA, smemB, acc, use_acc=k != 0, multicast=True, mbarriers=[mma_bar])
            mbarrier.wait(mma_bar, phase_mma, deps=[smemA, smemB])
            phase_tma = (phase_tma + 1) % 2
            phase_mma = (phase_mma + 1) % 2
```
- **EN:** Invokes `mbarrier.expect`, `tma.async_load`, `blackwell.tcgen05_mma`, `mbarrier.wait` to execute the test logic. Iterates across cases or data tiles.
- **CN:** 调用 `mbarrier.expect`、`tma.async_load`、`blackwell.tcgen05_mma`、`mbarrier.wait` 执行测试逻辑。 通过循环覆盖多个用例或数据分块。

##### Lines 1424-1426

```python

        mbarrier.invalidate(tma_bar)
        mbarrier.invalidate(mma_bar)
```
- **EN:** Invokes `mbarrier.invalidate` to execute the test logic.
- **CN:** 调用 `mbarrier.invalidate` 执行测试逻辑。

#### Lines 1427-1441

```python

    block_m = mma_block_m(num_ctas)
    block_n = mma_block_n(num_ctas)
    num_k_tiles = 1 if FAILURE else 4
    a = torch.randn((block_m, XBLOCK.value * num_k_tiles), device=device, dtype=torch.float16)
    b = torch.randn((XBLOCK.value * num_k_tiles, block_n), device=device, dtype=torch.float16)
    a_desc = gluon.nvidia.hopper.TensorDescriptor.from_tensor(
        a, [block_m, XBLOCK.value],
        ttgl.NVMMASharedLayout.get_default_for([block_m, XBLOCK.value], ttgl.float16,
                                               cga_layout=mma_cga_layout(num_ctas, 0, True)))
    b_desc = gluon.nvidia.hopper.TensorDescriptor.from_tensor(
        b, [XBLOCK.value, block_n],
        ttgl.NVMMASharedLayout.get_default_for([XBLOCK.value, block_n], ttgl.float16,
                                               cga_layout=mma_cga_layout(num_ctas, 1, True)))
    kernel[(1, )](a_desc, b_desc, FAILURE=FAILURE, num_warps=4, num_ctas=num_ctas)
```
- **EN:** Prepares or updates state through `block_m`, `block_n`, `num_k_tiles`, `a`, `b`, `a_desc`, `b_desc`. Invokes `mma_block_m`, `mma_block_n`, `torch.randn`, `gluon.nvidia.hopper.TensorDescriptor.from_tensor`, `ttgl.NVMMASharedLayout.get_default_for`, `mma_cga_layout` to execute the test logic. Relevant themes: PyTorch tensor setup and checks, tensor/descriptor metadata, layout transformation reasoning, random-data generation.
- **CN:** 通过 `block_m`、`block_n`、`num_k_tiles`、`a`、`b`、`a_desc`、`b_desc` 准备或更新状态。 调用 `mma_block_m`、`mma_block_n`、`torch.randn`、`gluon.nvidia.hopper.TensorDescriptor.from_tensor`、`ttgl.NVMMASharedLayout.get_default_for`、`mma_cga_layout` 执行测试逻辑。 相关主题：PyTorch 张量准备与校验、张量/描述符元数据、布局变换推理、随机数据生成。

### Lines 1442-1445

```python


@pytest.mark.skipif(not is_cuda() or torch.cuda.get_device_capability()[0] < 10, reason="Requires blackwell or newer")
def test_tma_tcgen05_mma_missing_multicast(device, run_wrapper, monkeypatch, num_ctas):
```
- **EN:** Defines the test function `test_tma_tcgen05_mma_missing_multicast`. Decorators: `pytest.mark.skipif(not is_cuda() or torch.cuda.get_device_capability()[0] < 10, reason='Requires blackwell or newer')`. Parameters: `device`, `run_wrapper`, `monkeypatch`, `num_ctas`. Nested definitions in this scope: `kernel`. Key calls include `pytest.mark.skipif`, `monkeypatch.setenv`, `knobs.refresh_knobs`, `mma_block_m`, `mma_block_n`, `torch.randn`, and 20 more. This scope touches PyTorch tensor setup and checks, tensor/descriptor metadata, runtime driver interaction, kernel launch orchestration.
- **CN:** 定义测试函数 `test_tma_tcgen05_mma_missing_multicast`。 装饰器：`pytest.mark.skipif(not is_cuda() or torch.cuda.get_device_capability()[0] < 10, reason='Requires blackwell or newer')`。 参数：`device`、`run_wrapper`、`monkeypatch`、`num_ctas`。 该作用域中的嵌套定义：`kernel`。 关键调用包括 `pytest.mark.skipif`、`monkeypatch.setenv`、`knobs.refresh_knobs`、`mma_block_m`、`mma_block_n`、`torch.randn` 等另外 20 项。 该作用域涉及PyTorch 张量准备与校验、张量/描述符元数据、运行时驱动交互、kernel 启动编排。

#### Lines 1446-1447

```python
    if num_ctas != 4:
        pytest.skip("Need 4 CTAs to exercise the missing tcgen05_mma multicast race")
```
- **EN:** Invokes `pytest.skip` to execute the test logic. Branches on runtime or test conditions.
- **CN:** 调用 `pytest.skip` 执行测试逻辑。 根据运行时或测试条件进行分支。

#### Lines 1448-1452

```python
    if run_wrapper:
        result = run_in_process(test_tma_tcgen05_mma_missing_multicast, (device, False, monkeypatch, num_ctas))
        assert_expected_cuda_failure(result.exc)
        assert "Buffer being accessed has outstanding" in result.driver_stderr_output
        return
```
- **EN:** Invokes `run_in_process`, `assert_expected_cuda_failure` to execute the test logic. Validates behavior with 1 assertion(s). Branches on runtime or test conditions. Relevant themes: runtime driver interaction.
- **CN:** 调用 `run_in_process`、`assert_expected_cuda_failure` 执行测试逻辑。 通过 1 个断言验证行为。 根据运行时或测试条件进行分支。 相关主题：运行时驱动交互。

#### Lines 1453-1456

```python

    monkeypatch.setenv("TRITON_INSTRUMENTATION_MODE", "consan")
    monkeypatch.setenv("CUDA_LAUNCH_BLOCKING", "1")
    knobs.refresh_knobs()
```
- **EN:** Invokes `monkeypatch.setenv`, `knobs.refresh_knobs` to execute the test logic. Relevant themes: kernel launch orchestration.
- **CN:** 调用 `monkeypatch.setenv`、`knobs.refresh_knobs` 执行测试逻辑。 相关主题：kernel 启动编排。

#### Lines 1457-1459

```python

    @gluon.jit
    def kernel(a_desc, b_desc):
```
- **EN:** Defines the helper function `kernel`. Decorators: `gluon.jit`. Parameters: `a_desc`, `b_desc`. Key calls include `mma_block_m`, `mma_block_n`, `blackwell.TensorMemoryLayout`, `ttgl.allocate_shared_memory`, `blackwell.allocate_tensor_memory`, `mbarrier.allocate_mbarrier`, and 10 more. This scope touches layout transformation reasoning.
- **CN:** 定义辅助函数 `kernel`。 装饰器：`gluon.jit`。 参数：`a_desc`、`b_desc`。 关键调用包括 `mma_block_m`、`mma_block_n`、`blackwell.TensorMemoryLayout`、`ttgl.allocate_shared_memory`、`blackwell.allocate_tensor_memory`、`mbarrier.allocate_mbarrier` 等另外 10 项。 该作用域涉及布局变换推理。

##### Lines 1460-1484

```python
        num_k_tiles: ttgl.constexpr = 4
        block_m: ttgl.constexpr = mma_block_m(ttgl.num_ctas())
        block_n: ttgl.constexpr = mma_block_n(ttgl.num_ctas())
        acc_layout: ttgl.constexpr = blackwell.TensorMemoryLayout(
            [XBLOCK, XBLOCK],
            col_stride=1,
            cga_layout=mma_cga_layout(ttgl.num_ctas(), 2, True),
            two_ctas=True,
        )
        smemA = ttgl.allocate_shared_memory(ttgl.float16, [block_m, XBLOCK], a_desc.layout)
        smemB = ttgl.allocate_shared_memory(
            ttgl.float16,
            [XBLOCK, block_n],
            ttgl.NVMMASharedLayout.get_default_for([XBLOCK, block_n], ttgl.float16,
                                                   cga_layout=mma_cga_layout(ttgl.num_ctas(), 1, True)),
        )
        acc = blackwell.allocate_tensor_memory(ttgl.float32, [block_m, block_n], acc_layout)
        tma_bar = mbarrier.allocate_mbarrier(two_ctas=True)
        mbarrier.init(tma_bar, count=1)
        mma_bar = mbarrier.allocate_mbarrier()
        mbarrier.init(mma_bar, count=blackwell.tcgen05_mma_barrier_count([smemA, smemB], False,
                                                                         acc.type.layout.two_ctas))

        phase_tma = 0
        phase_mma = 0
```
- **EN:** Prepares or updates state through `num_k_tiles`, `block_m`, `block_n`, `acc_layout`, `smemA`, `smemB`, `acc`, `tma_bar`, and 3 more. Invokes `mma_block_m`, `ttgl.num_ctas`, `mma_block_n`, `blackwell.TensorMemoryLayout`, `mma_cga_layout`, `ttgl.allocate_shared_memory`, and 5 more to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `num_k_tiles`、`block_m`、`block_n`、`acc_layout`、`smemA`、`smemB`、`acc`、`tma_bar` 等另外 3 项 准备或更新状态。 调用 `mma_block_m`、`ttgl.num_ctas`、`mma_block_n`、`blackwell.TensorMemoryLayout`、`mma_cga_layout`、`ttgl.allocate_shared_memory` 等另外 5 项 执行测试逻辑。 相关主题：布局变换推理。

##### Lines 1485-1497

```python
        for k in range(num_k_tiles):
            offs_k = k * XBLOCK
            mbarrier.expect(tma_bar, a_desc.nbytes_per_cta + b_desc.nbytes_per_cta)
            tma.async_load(a_desc, [0, offs_k], tma_bar, smemA, multicast=True)
            tma.async_load(b_desc, [offs_k, 0], tma_bar, smemB, multicast=True)
            mbarrier.wait(tma_bar, phase_tma, deps=[smemA, smemB])

            # Missing multicast=True is the bug under test. The next iteration
            # reuses smemA/smemB after a local completion wait.
            blackwell.tcgen05_mma(smemA, smemB, acc, use_acc=k != 0, mbarriers=[mma_bar])
            mbarrier.wait(mma_bar, phase_mma, deps=[smemA, smemB])
            phase_tma = (phase_tma + 1) % 2
            phase_mma = (phase_mma + 1) % 2
```
- **EN:** Invokes `mbarrier.expect`, `tma.async_load`, `mbarrier.wait`, `blackwell.tcgen05_mma` to execute the test logic. Iterates across cases or data tiles.
- **CN:** 调用 `mbarrier.expect`、`tma.async_load`、`mbarrier.wait`、`blackwell.tcgen05_mma` 执行测试逻辑。 通过循环覆盖多个用例或数据分块。

##### Lines 1498-1500

```python

        mbarrier.invalidate(tma_bar)
        mbarrier.invalidate(mma_bar)
```
- **EN:** Invokes `mbarrier.invalidate` to execute the test logic.
- **CN:** 调用 `mbarrier.invalidate` 执行测试逻辑。

#### Lines 1501-1515

```python

    block_m = mma_block_m(num_ctas)
    block_n = mma_block_n(num_ctas)
    num_k_tiles = 4
    a = torch.randn((block_m, XBLOCK.value * num_k_tiles), device=device, dtype=torch.float16)
    b = torch.randn((XBLOCK.value * num_k_tiles, block_n), device=device, dtype=torch.float16)
    a_desc = gluon.nvidia.hopper.TensorDescriptor.from_tensor(
        a, [block_m, XBLOCK.value],
        ttgl.NVMMASharedLayout.get_default_for([block_m, XBLOCK.value], ttgl.float16,
                                               cga_layout=mma_cga_layout(num_ctas, 0, True)))
    b_desc = gluon.nvidia.hopper.TensorDescriptor.from_tensor(
        b, [XBLOCK.value, block_n],
        ttgl.NVMMASharedLayout.get_default_for([XBLOCK.value, block_n], ttgl.float16,
                                               cga_layout=mma_cga_layout(num_ctas, 1, True)))
    kernel[(1, )](a_desc, b_desc, num_warps=4, num_ctas=num_ctas)
```
- **EN:** Prepares or updates state through `block_m`, `block_n`, `num_k_tiles`, `a`, `b`, `a_desc`, `b_desc`. Invokes `mma_block_m`, `mma_block_n`, `torch.randn`, `gluon.nvidia.hopper.TensorDescriptor.from_tensor`, `ttgl.NVMMASharedLayout.get_default_for`, `mma_cga_layout` to execute the test logic. Relevant themes: PyTorch tensor setup and checks, tensor/descriptor metadata, layout transformation reasoning, random-data generation.
- **CN:** 通过 `block_m`、`block_n`、`num_k_tiles`、`a`、`b`、`a_desc`、`b_desc` 准备或更新状态。 调用 `mma_block_m`、`mma_block_n`、`torch.randn`、`gluon.nvidia.hopper.TensorDescriptor.from_tensor`、`ttgl.NVMMASharedLayout.get_default_for`、`mma_cga_layout` 执行测试逻辑。 相关主题：PyTorch 张量准备与校验、张量/描述符元数据、布局变换推理、随机数据生成。

### Lines 1516-1520

```python


@pytest.mark.skipif(not is_cuda() or torch.cuda.get_device_capability()[0] < 10, reason="Requires blackwell or newer")
@pytest.mark.parametrize("OVERCOUNTED", [False, True])
def test_tcgen5_commit_multicast_barrier_count(OVERCOUNTED, device, run_wrapper, monkeypatch):
```
- **EN:** Defines the test function `test_tcgen5_commit_multicast_barrier_count`. Decorators: `pytest.mark.skipif(not is_cuda() or torch.cuda.get_device_capability()[0] < 10, reason='Requires blackwell or newer')`, `pytest.mark.parametrize('OVERCOUNTED', [False, True])`. Parameters: `OVERCOUNTED`, `device`, `run_wrapper`, `monkeypatch`. Nested definitions in this scope: `kernel`. Key calls include `pytest.mark.skipif`, `pytest.mark.parametrize`, `monkeypatch.setenv`, `knobs.refresh_knobs`, `torch.randn`, `gluon.nvidia.hopper.TensorDescriptor.from_tensor`, and 17 more. This scope touches pytest parametrization, PyTorch tensor setup and checks, tensor/descriptor metadata, runtime driver interaction.
- **CN:** 定义测试函数 `test_tcgen5_commit_multicast_barrier_count`。 装饰器：`pytest.mark.skipif(not is_cuda() or torch.cuda.get_device_capability()[0] < 10, reason='Requires blackwell or newer')`、`pytest.mark.parametrize('OVERCOUNTED', [False, True])`。 参数：`OVERCOUNTED`、`device`、`run_wrapper`、`monkeypatch`。 该作用域中的嵌套定义：`kernel`。 关键调用包括 `pytest.mark.skipif`、`pytest.mark.parametrize`、`monkeypatch.setenv`、`knobs.refresh_knobs`、`torch.randn`、`gluon.nvidia.hopper.TensorDescriptor.from_tensor` 等另外 17 项。 该作用域涉及pytest 参数化、PyTorch 张量准备与校验、张量/描述符元数据、运行时驱动交互。

#### Lines 1521-1529

```python
    if run_wrapper:
        result = run_in_process(test_tcgen5_commit_multicast_barrier_count, (OVERCOUNTED, device, False, monkeypatch))
        if OVERCOUNTED:
            assert_expected_cuda_failure(result.exc)
            assert "Deadlock detected" in result.driver_stderr_output
        else:
            assert result.exc is None
            assert result.driver_stderr_output == ""
        return
```
- **EN:** Invokes `run_in_process`, `assert_expected_cuda_failure` to execute the test logic. Validates behavior with 3 assertion(s). Branches on runtime or test conditions. Relevant themes: runtime driver interaction.
- **CN:** 调用 `run_in_process`、`assert_expected_cuda_failure` 执行测试逻辑。 通过 3 个断言验证行为。 根据运行时或测试条件进行分支。 相关主题：运行时驱动交互。

#### Lines 1530-1533

```python

    monkeypatch.setenv("TRITON_INSTRUMENTATION_MODE", "consan")
    monkeypatch.setenv("CUDA_LAUNCH_BLOCKING", "1")
    knobs.refresh_knobs()
```
- **EN:** Invokes `monkeypatch.setenv`, `knobs.refresh_knobs` to execute the test logic. Relevant themes: kernel launch orchestration.
- **CN:** 调用 `monkeypatch.setenv`、`knobs.refresh_knobs` 执行测试逻辑。 相关主题：kernel 启动编排。

#### Lines 1534-1536

```python

    @gluon.jit
    def kernel(a_desc, b_desc, OVERCOUNTED: ttgl.constexpr):
```
- **EN:** Defines the helper function `kernel`. Decorators: `gluon.jit`. Parameters: `a_desc`, `b_desc`, `OVERCOUNTED`. Key calls include `ttgl.allocate_shared_memory`, `blackwell.TensorMemoryLayout`, `allocate_tensor_memory`, `mbarrier.allocate_mbarrier`, `blackwell.tcgen05_mma_barrier_count`, `mbarrier.init`, and 6 more. This scope touches layout transformation reasoning.
- **CN:** 定义辅助函数 `kernel`。 装饰器：`gluon.jit`。 参数：`a_desc`、`b_desc`、`OVERCOUNTED`。 关键调用包括 `ttgl.allocate_shared_memory`、`blackwell.TensorMemoryLayout`、`allocate_tensor_memory`、`mbarrier.allocate_mbarrier`、`blackwell.tcgen05_mma_barrier_count`、`mbarrier.init` 等另外 6 项。 该作用域涉及布局变换推理。

##### Lines 1537-1558

```python
        block_m: ttgl.constexpr = 256
        block_n: ttgl.constexpr = 128
        smem_a = ttgl.allocate_shared_memory(ttgl.float16, [block_m, XBLOCK], a_desc.layout)
        smem_b = ttgl.allocate_shared_memory(ttgl.float16, [XBLOCK, block_n], b_desc.layout)
        acc_layout: ttgl.constexpr = blackwell.TensorMemoryLayout([XBLOCK, XBLOCK], col_stride=1, cga_layout=((1, 0), ),
                                                                  two_ctas=True)
        acc = allocate_tensor_memory(ttgl.float32, [block_m, block_n], acc_layout)

        tma_bar = mbarrier.allocate_mbarrier(two_ctas=True)
        commit_bar = mbarrier.allocate_mbarrier()
        count: ttgl.constexpr = blackwell.tcgen05_mma_barrier_count([smem_a, smem_b], True, acc.type.layout.two_ctas)
        mbarrier.init(tma_bar, count=1)
        mbarrier.init(commit_bar, count=count + OVERCOUNTED)
        mbarrier.expect(tma_bar, a_desc.nbytes_per_cta + b_desc.nbytes_per_cta)
        tma.async_load(a_desc, [0, 0], tma_bar, smem_a, multicast=True)
        tma.async_load(b_desc, [0, 0], tma_bar, smem_b, multicast=True)
        mbarrier.wait(tma_bar, 0, deps=[smem_a, smem_b])
        mbarrier.invalidate(tma_bar)

        blackwell.tcgen05_mma(smem_a, smem_b, acc, use_acc=False, multicast=True)
        blackwell.tcgen05_commit(commit_bar, descs=[smem_a, smem_b])
        mbarrier.wait(commit_bar, 0)
```
- **EN:** Prepares or updates state through `block_m`, `block_n`, `smem_a`, `smem_b`, `acc_layout`, `acc`, `tma_bar`, `commit_bar`, and 1 more. Invokes `ttgl.allocate_shared_memory`, `blackwell.TensorMemoryLayout`, `allocate_tensor_memory`, `mbarrier.allocate_mbarrier`, `blackwell.tcgen05_mma_barrier_count`, `mbarrier.init`, and 6 more to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `block_m`、`block_n`、`smem_a`、`smem_b`、`acc_layout`、`acc`、`tma_bar`、`commit_bar` 等另外 1 项 准备或更新状态。 调用 `ttgl.allocate_shared_memory`、`blackwell.TensorMemoryLayout`、`allocate_tensor_memory`、`mbarrier.allocate_mbarrier`、`blackwell.tcgen05_mma_barrier_count`、`mbarrier.init` 等另外 6 项 执行测试逻辑。 相关主题：布局变换推理。

#### Lines 1559-1568

```python

    a = torch.randn((256, XBLOCK.value), device=device, dtype=torch.float16)
    b = torch.randn((XBLOCK.value, 128), device=device, dtype=torch.float16)
    a_desc = gluon.nvidia.hopper.TensorDescriptor.from_tensor(
        a, [256, XBLOCK.value],
        ttgl.NVMMASharedLayout.get_default_for([256, XBLOCK.value], ttgl.float16, cga_layout=((1, 0), )))
    b_desc = gluon.nvidia.hopper.TensorDescriptor.from_tensor(
        b, [XBLOCK.value, 128],
        ttgl.NVMMASharedLayout.get_default_for([XBLOCK.value, 128], ttgl.float16, cga_layout=((0, 1), )))
    kernel[(1, )](a_desc, b_desc, OVERCOUNTED=OVERCOUNTED, num_warps=4, num_ctas=2)
```
- **EN:** Prepares or updates state through `a`, `b`, `a_desc`, `b_desc`. Invokes `torch.randn`, `gluon.nvidia.hopper.TensorDescriptor.from_tensor`, `ttgl.NVMMASharedLayout.get_default_for` to execute the test logic. Relevant themes: PyTorch tensor setup and checks, tensor/descriptor metadata, layout transformation reasoning, random-data generation.
- **CN:** 通过 `a`、`b`、`a_desc`、`b_desc` 准备或更新状态。 调用 `torch.randn`、`gluon.nvidia.hopper.TensorDescriptor.from_tensor`、`ttgl.NVMMASharedLayout.get_default_for` 执行测试逻辑。 相关主题：PyTorch 张量准备与校验、张量/描述符元数据、布局变换推理、随机数据生成。

### Lines 1569-1573

```python


@pytest.mark.skipif(not is_cuda() or torch.cuda.get_device_capability()[0] != 9, reason="Requires hopper")
@pytest.mark.parametrize("FAILURE", [True, False])
def test_multibuffered_wgmma_loop(FAILURE, device, run_wrapper, monkeypatch, num_ctas):
```
- **EN:** Defines the test function `test_multibuffered_wgmma_loop`. Decorators: `pytest.mark.skipif(not is_cuda() or torch.cuda.get_device_capability()[0] != 9, reason='Requires hopper')`, `pytest.mark.parametrize('FAILURE', [True, False])`. Parameters: `FAILURE`, `device`, `run_wrapper`, `monkeypatch`, `num_ctas`. Nested definitions in this scope: `kernel`. Key calls include `pytest.mark.skipif`, `pytest.mark.parametrize`, `monkeypatch.setenv`, `knobs.refresh_knobs`, `mma_block_m`, `mma_block_n`, and 26 more. This scope touches pytest parametrization, PyTorch tensor setup and checks, tensor/descriptor metadata, runtime driver interaction.
- **CN:** 定义测试函数 `test_multibuffered_wgmma_loop`。 装饰器：`pytest.mark.skipif(not is_cuda() or torch.cuda.get_device_capability()[0] != 9, reason='Requires hopper')`、`pytest.mark.parametrize('FAILURE', [True, False])`。 参数：`FAILURE`、`device`、`run_wrapper`、`monkeypatch`、`num_ctas`。 该作用域中的嵌套定义：`kernel`。 关键调用包括 `pytest.mark.skipif`、`pytest.mark.parametrize`、`monkeypatch.setenv`、`knobs.refresh_knobs`、`mma_block_m`、`mma_block_n` 等另外 26 项。 该作用域涉及pytest 参数化、PyTorch 张量准备与校验、张量/描述符元数据、运行时驱动交互。

#### Lines 1574-1582

```python
    if run_wrapper:
        result = run_in_process(test_multibuffered_wgmma_loop, (FAILURE, device, False, monkeypatch, num_ctas))
        if FAILURE:
            assert_expected_cuda_failure(result.exc)
            assert "Accessing buffer with pending access. Pending access type: warpgroup_mma operand read" in result.driver_stderr_output
        else:
            assert result.exc is None
            assert result.driver_stderr_output == ""
        return
```
- **EN:** Invokes `run_in_process`, `assert_expected_cuda_failure` to execute the test logic. Validates behavior with 3 assertion(s). Branches on runtime or test conditions. Relevant themes: runtime driver interaction, random-data generation.
- **CN:** 调用 `run_in_process`、`assert_expected_cuda_failure` 执行测试逻辑。 通过 3 个断言验证行为。 根据运行时或测试条件进行分支。 相关主题：运行时驱动交互、随机数据生成。

#### Lines 1583-1586

```python

    monkeypatch.setenv("TRITON_INSTRUMENTATION_MODE", "consan")
    monkeypatch.setenv("CUDA_LAUNCH_BLOCKING", "1")
    knobs.refresh_knobs()
```
- **EN:** Invokes `monkeypatch.setenv`, `knobs.refresh_knobs` to execute the test logic. Relevant themes: kernel launch orchestration.
- **CN:** 调用 `monkeypatch.setenv`、`knobs.refresh_knobs` 执行测试逻辑。 相关主题：kernel 启动编排。

#### Lines 1587-1589

```python

    @gluon.jit
    def kernel(a_desc, b_desc, FAILURE: ttgl.constexpr):
```
- **EN:** Defines the helper function `kernel`. Decorators: `gluon.jit`. Parameters: `a_desc`, `b_desc`, `FAILURE`. Key calls include `mma_block_m`, `mma_block_n`, `mma_cga_layout`, `ttgl.NVMMADistributedLayout`, `hopper.warpgroup_mma_init`, `ttgl.allocate_shared_memory`, and 15 more. This scope touches layout transformation reasoning.
- **CN:** 定义辅助函数 `kernel`。 装饰器：`gluon.jit`。 参数：`a_desc`、`b_desc`、`FAILURE`。 关键调用包括 `mma_block_m`、`mma_block_n`、`mma_cga_layout`、`ttgl.NVMMADistributedLayout`、`hopper.warpgroup_mma_init`、`ttgl.allocate_shared_memory` 等另外 15 项。 该作用域涉及布局变换推理。

##### Lines 1590-1602

```python
        num_buffers: ttgl.constexpr = 2 if FAILURE else 3
        block_m: ttgl.constexpr = mma_block_m(ttgl.num_ctas())
        block_n: ttgl.constexpr = mma_block_n(ttgl.num_ctas())

        cga_layout_c: ttgl.constexpr = mma_cga_layout(ttgl.num_ctas(), 2)
        mma_layout: ttgl.constexpr = ttgl.NVMMADistributedLayout(version=[3, 0], warps_per_cta=[4, 1],
                                                                 instr_shape=[16, 32, 16], cga_layout=cga_layout_c)
        acc = hopper.warpgroup_mma_init(ttgl.zeros([block_m, block_n], ttgl.float32, mma_layout))

        smemA = ttgl.allocate_shared_memory(ttgl.float16, [num_buffers, block_m, XBLOCK], a_desc.layout)
        smemB = ttgl.allocate_shared_memory(ttgl.float16, [num_buffers, XBLOCK, block_n], b_desc.layout)
        barLoadA = mbarrier.allocate_mbarrier(batch=num_buffers)
        barLoadB = mbarrier.allocate_mbarrier(batch=num_buffers)
```
- **EN:** Prepares or updates state through `num_buffers`, `block_m`, `block_n`, `cga_layout_c`, `mma_layout`, `acc`, `smemA`, `smemB`, and 2 more. Invokes `mma_block_m`, `ttgl.num_ctas`, `mma_block_n`, `mma_cga_layout`, `ttgl.NVMMADistributedLayout`, `hopper.warpgroup_mma_init`, and 3 more to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `num_buffers`、`block_m`、`block_n`、`cga_layout_c`、`mma_layout`、`acc`、`smemA`、`smemB` 等另外 2 项 准备或更新状态。 调用 `mma_block_m`、`ttgl.num_ctas`、`mma_block_n`、`mma_cga_layout`、`ttgl.NVMMADistributedLayout`、`hopper.warpgroup_mma_init` 等另外 3 项 执行测试逻辑。 相关主题：布局变换推理。

##### Lines 1603-1605

```python
        for i in range(num_buffers):
            mbarrier.init(barLoadA.index(i), count=1)
            mbarrier.init(barLoadB.index(i), count=1)
```
- **EN:** Invokes `mbarrier.init`, `barLoadA.index`, `barLoadB.index` to execute the test logic. Iterates across cases or data tiles.
- **CN:** 调用 `mbarrier.init`、`barLoadA.index`、`barLoadB.index` 执行测试逻辑。 通过循环覆盖多个用例或数据分块。

##### Lines 1606-1620

```python

        phase = 0
        ins_id = 0
        ext_id = 0

        # ins_id = 0
        mbarrier.expect(barLoadA.index(ins_id), a_desc.nbytes_per_cta)
        tma.async_load(a_desc, [0, 0], barLoadA.index(ins_id), smemA.index(ins_id))

        mbarrier.expect(barLoadB.index(ins_id), b_desc.nbytes_per_cta)
        tma.async_load(b_desc, [0, 0], barLoadB.index(ins_id), smemB.index(ins_id))
        ins_id = inc_mod(ins_id, num_buffers)

        # ins_id = 1
        ub = 10
```
- **EN:** Prepares or updates state through `phase`, `ins_id`, `ext_id`, `ub`. Invokes `mbarrier.expect`, `barLoadA.index`, `tma.async_load`, `smemA.index`, `barLoadB.index`, `smemB.index`, and 1 more to execute the test logic.
- **CN:** 通过 `phase`、`ins_id`、`ext_id`、`ub` 准备或更新状态。 调用 `mbarrier.expect`、`barLoadA.index`、`tma.async_load`、`smemA.index`、`barLoadB.index`、`smemB.index` 等另外 1 项 执行测试逻辑。

##### Lines 1621-1637

```python
        for i in range(ub):
            if i < ub - 1:
                mbarrier.expect(barLoadA.index(ins_id), a_desc.nbytes_per_cta)
                tma.async_load(a_desc, [0, 0], barLoadA.index(ins_id), smemA.index(ins_id))

                mbarrier.expect(barLoadB.index(ins_id), b_desc.nbytes_per_cta)
                tma.async_load(b_desc, [0, 0], barLoadB.index(ins_id), smemB.index(ins_id))
                ins_id = inc_mod(ins_id, num_buffers)

            mbarrier.wait(barLoadA.index(ext_id), phase)
            mbarrier.wait(barLoadB.index(ext_id), phase)

            acc = hopper.warpgroup_mma(smemA.index(ext_id), smemB.index(ext_id), acc, is_async=True)
            hopper.warpgroup_mma_wait(num_outstanding=1, deps=[acc])
            ext_id = inc_mod(ext_id, num_buffers)
            if ext_id == 0:
                phase = (phase + 1) % 2
```
- **EN:** Invokes `mbarrier.wait`, `hopper.warpgroup_mma`, `hopper.warpgroup_mma_wait`, `inc_mod`, `mbarrier.expect`, `tma.async_load`, and 4 more to execute the test logic. Iterates across cases or data tiles.
- **CN:** 调用 `mbarrier.wait`、`hopper.warpgroup_mma`、`hopper.warpgroup_mma_wait`、`inc_mod`、`mbarrier.expect`、`tma.async_load` 等另外 4 项 执行测试逻辑。 通过循环覆盖多个用例或数据分块。

##### Lines 1638-1638

```python
        hopper.warpgroup_mma_wait(num_outstanding=0, deps=[acc])
```
- **EN:** Invokes `hopper.warpgroup_mma_wait` to execute the test logic.
- **CN:** 调用 `hopper.warpgroup_mma_wait` 执行测试逻辑。

##### Lines 1639-1642

```python

        for i in range(num_buffers):
            mbarrier.invalidate(barLoadA.index(i))
            mbarrier.invalidate(barLoadB.index(i))
```
- **EN:** Invokes `mbarrier.invalidate`, `barLoadA.index`, `barLoadB.index` to execute the test logic. Iterates across cases or data tiles.
- **CN:** 调用 `mbarrier.invalidate`、`barLoadA.index`、`barLoadB.index` 执行测试逻辑。 通过循环覆盖多个用例或数据分块。

#### Lines 1643-1654

```python

    block_m = mma_block_m(num_ctas)
    block_n = mma_block_n(num_ctas)
    input_a = torch.randn((block_m, XBLOCK.value), device=device, dtype=torch.float16)
    input_b = torch.randn((XBLOCK.value, block_n), device=device, dtype=torch.float16)
    shared_layout_a = ttgl.NVMMASharedLayout(swizzle_byte_width=128, element_bitwidth=16, rank=2,
                                             cga_layout=mma_cga_layout(num_ctas, 0))
    shared_layout_b = ttgl.NVMMASharedLayout(swizzle_byte_width=128, element_bitwidth=16, rank=2,
                                             cga_layout=mma_cga_layout(num_ctas, 1))
    a_desc = gluon.nvidia.hopper.TensorDescriptor.from_tensor(input_a, [block_m, XBLOCK.value], shared_layout_a)
    b_desc = gluon.nvidia.hopper.TensorDescriptor.from_tensor(input_b, [XBLOCK.value, block_n], shared_layout_b)
    kernel[(1, )](a_desc, b_desc, FAILURE=FAILURE, num_warps=4, num_ctas=num_ctas)
```
- **EN:** Prepares or updates state through `block_m`, `block_n`, `input_a`, `input_b`, `shared_layout_a`, `shared_layout_b`, `a_desc`, `b_desc`. Invokes `mma_block_m`, `mma_block_n`, `torch.randn`, `ttgl.NVMMASharedLayout`, `mma_cga_layout`, `gluon.nvidia.hopper.TensorDescriptor.from_tensor` to execute the test logic. Relevant themes: PyTorch tensor setup and checks, tensor/descriptor metadata, layout transformation reasoning, random-data generation.
- **CN:** 通过 `block_m`、`block_n`、`input_a`、`input_b`、`shared_layout_a`、`shared_layout_b`、`a_desc`、`b_desc` 准备或更新状态。 调用 `mma_block_m`、`mma_block_n`、`torch.randn`、`ttgl.NVMMASharedLayout`、`mma_cga_layout`、`gluon.nvidia.hopper.TensorDescriptor.from_tensor` 执行测试逻辑。 相关主题：PyTorch 张量准备与校验、张量/描述符元数据、布局变换推理、随机数据生成。

### Lines 1655-1659

```python


@pytest.mark.skipif(not is_cuda() or torch.cuda.get_device_capability()[0] < 9, reason="Requires hopper")
@pytest.mark.parametrize("FAILURE", [True, False])
def test_ws_store_wait_load(FAILURE, device, run_wrapper, monkeypatch, num_ctas):
```
- **EN:** Defines the test function `test_ws_store_wait_load`. Decorators: `pytest.mark.skipif(not is_cuda() or torch.cuda.get_device_capability()[0] < 9, reason='Requires hopper')`, `pytest.mark.parametrize('FAILURE', [True, False])`. Parameters: `FAILURE`, `device`, `run_wrapper`, `monkeypatch`, `num_ctas`. Nested definitions in this scope: `ws_default`, `ws_1`, `ws_kernel`. Key calls include `pytest.mark.skipif`, `pytest.mark.parametrize`, `monkeypatch.setenv`, `knobs.refresh_knobs`, `torch.empty`, `run_in_process`, and 17 more. This scope touches pytest parametrization, PyTorch tensor setup and checks, runtime driver interaction, kernel launch orchestration.
- **CN:** 定义测试函数 `test_ws_store_wait_load`。 装饰器：`pytest.mark.skipif(not is_cuda() or torch.cuda.get_device_capability()[0] < 9, reason='Requires hopper')`、`pytest.mark.parametrize('FAILURE', [True, False])`。 参数：`FAILURE`、`device`、`run_wrapper`、`monkeypatch`、`num_ctas`。 该作用域中的嵌套定义：`ws_default`、`ws_1`、`ws_kernel`。 关键调用包括 `pytest.mark.skipif`、`pytest.mark.parametrize`、`monkeypatch.setenv`、`knobs.refresh_knobs`、`torch.empty`、`run_in_process` 等另外 17 项。 该作用域涉及pytest 参数化、PyTorch 张量准备与校验、运行时驱动交互、kernel 启动编排。

#### Lines 1660-1668

```python
    if run_wrapper:
        result = run_in_process(test_ws_store_wait_load, (FAILURE, device, False, monkeypatch, num_ctas))
        if FAILURE:
            assert_expected_cuda_failure(result.exc)
            assert "Buffer being accessed has outstanding" in result.driver_stderr_output
        else:
            assert result.exc is None
            assert result.driver_stderr_output == ""
        return
```
- **EN:** Invokes `run_in_process`, `assert_expected_cuda_failure` to execute the test logic. Validates behavior with 3 assertion(s). Branches on runtime or test conditions. Relevant themes: runtime driver interaction.
- **CN:** 调用 `run_in_process`、`assert_expected_cuda_failure` 执行测试逻辑。 通过 3 个断言验证行为。 根据运行时或测试条件进行分支。 相关主题：运行时驱动交互。

#### Lines 1669-1671

```python
    monkeypatch.setenv("TRITON_INSTRUMENTATION_MODE", "consan")
    monkeypatch.setenv("CUDA_LAUNCH_BLOCKING", "1")
    knobs.refresh_knobs()
```
- **EN:** Invokes `monkeypatch.setenv`, `knobs.refresh_knobs` to execute the test logic. Relevant themes: kernel launch orchestration.
- **CN:** 调用 `monkeypatch.setenv`、`knobs.refresh_knobs` 执行测试逻辑。 相关主题：kernel 启动编排。

#### Lines 1672-1674

```python

    @gluon.jit
    def ws_default(smem, bar, FAILURE: ttgl.constexpr, layout: ttgl.constexpr):
```
- **EN:** Defines the helper function `ws_default`. Decorators: `gluon.jit`. Parameters: `smem`, `bar`, `FAILURE`, `layout`. Key calls include `mbarrier.wait`, `mbarrier.arrive`, `bar.index`, `smem.index`. This scope touches layout transformation reasoning.
- **CN:** 定义辅助函数 `ws_default`。 装饰器：`gluon.jit`。 参数：`smem`、`bar`、`FAILURE`、`layout`。 关键调用包括 `mbarrier.wait`、`mbarrier.arrive`、`bar.index`、`smem.index`。 该作用域涉及布局变换推理。

##### Lines 1675-1678

```python
        mbarrier.wait(bar.index(0), phase=0, pred=(not FAILURE))
        val = smem.index(0).load(layout)
        smem.index(1).store(val)
        mbarrier.arrive(bar.index(1), count=1)
```
- **EN:** Prepares or updates state through `val`. Invokes `mbarrier.wait`, `bar.index`, `smem.index`, `mbarrier.arrive` to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `val` 准备或更新状态。 调用 `mbarrier.wait`、`bar.index`、`smem.index`、`mbarrier.arrive` 执行测试逻辑。 相关主题：布局变换推理。

#### Lines 1679-1681

```python

    @gluon.jit
    def ws_1(smem, bar, FAILURE: ttgl.constexpr, layout: ttgl.constexpr):
```
- **EN:** Defines the helper function `ws_1`. Decorators: `gluon.jit`. Parameters: `smem`, `bar`, `FAILURE`, `layout`. Key calls include `mbarrier.arrive`, `ttgl.num_ctas`, `bar.index`, `smem.index`, `ttgl.arange`. This scope touches layout transformation reasoning.
- **CN:** 定义辅助函数 `ws_1`。 装饰器：`gluon.jit`。 参数：`smem`、`bar`、`FAILURE`、`layout`。 关键调用包括 `mbarrier.arrive`、`ttgl.num_ctas`、`bar.index`、`smem.index`、`ttgl.arange`。 该作用域涉及布局变换推理。

##### Lines 1682-1684

```python
        block_x: ttgl.constexpr = XBLOCK * ttgl.num_ctas()
        smem.index(0).store(ttgl.arange(0, block_x, layout).to(ttgl.float16))
        mbarrier.arrive(bar.index(0), count=1)
```
- **EN:** Prepares or updates state through `block_x`. Invokes `ttgl.num_ctas`, `smem.index`, `ttgl.arange`, `mbarrier.arrive`, `bar.index` to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `block_x` 准备或更新状态。 调用 `ttgl.num_ctas`、`smem.index`、`ttgl.arange`、`mbarrier.arrive`、`bar.index` 执行测试逻辑。 相关主题：布局变换推理。

#### Lines 1685-1687

```python

    @gluon.jit
    def ws_kernel(output, FAILURE: ttgl.constexpr):
```
- **EN:** Defines the helper function `ws_kernel`. Decorators: `gluon.jit`. Parameters: `output`, `FAILURE`. Key calls include `default_cga_layout`, `ttgl.SwizzledSharedLayout`, `ttgl.BlockedLayout`, `ttgl.allocate_shared_memory`, `mbarrier.allocate_mbarrier`, `ttgl.warp_specialize`, and 7 more. This scope touches layout transformation reasoning.
- **CN:** 定义辅助函数 `ws_kernel`。 装饰器：`gluon.jit`。 参数：`output`、`FAILURE`。 关键调用包括 `default_cga_layout`、`ttgl.SwizzledSharedLayout`、`ttgl.BlockedLayout`、`ttgl.allocate_shared_memory`、`mbarrier.allocate_mbarrier`、`ttgl.warp_specialize` 等另外 7 项。 该作用域涉及布局变换推理。

##### Lines 1688-1695

```python
        block_x: ttgl.constexpr = XBLOCK * ttgl.num_ctas()
        cga_layout: ttgl.constexpr = default_cga_layout(ttgl.num_ctas(), 1)
        smem_layout: ttgl.constexpr = ttgl.SwizzledSharedLayout(vec=1, per_phase=1, max_phase=1, order=[0],
                                                                cga_layout=cga_layout)
        blocked_layout: ttgl.constexpr = ttgl.BlockedLayout(size_per_thread=[1], threads_per_warp=[32],
                                                            warps_per_cta=[4], order=[0], cga_layout=cga_layout)
        smem = ttgl.allocate_shared_memory(ttgl.float16, [2, block_x], smem_layout)
        bar = mbarrier.allocate_mbarrier(batch=2)
```
- **EN:** Prepares or updates state through `block_x`, `cga_layout`, `smem_layout`, `blocked_layout`, `smem`, `bar`. Invokes `ttgl.num_ctas`, `default_cga_layout`, `ttgl.SwizzledSharedLayout`, `ttgl.BlockedLayout`, `ttgl.allocate_shared_memory`, `mbarrier.allocate_mbarrier` to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `block_x`、`cga_layout`、`smem_layout`、`blocked_layout`、`smem`、`bar` 准备或更新状态。 调用 `ttgl.num_ctas`、`default_cga_layout`、`ttgl.SwizzledSharedLayout`、`ttgl.BlockedLayout`、`ttgl.allocate_shared_memory`、`mbarrier.allocate_mbarrier` 执行测试逻辑。 相关主题：布局变换推理。

##### Lines 1696-1697

```python
        for i in range(2):
            mbarrier.init(bar.index(i), count=1)
```
- **EN:** Invokes `mbarrier.init`, `bar.index` to execute the test logic. Iterates across cases or data tiles.
- **CN:** 调用 `mbarrier.init`、`bar.index` 执行测试逻辑。 通过循环覆盖多个用例或数据分块。

##### Lines 1698-1705

```python
        ttgl.warp_specialize([
            (ws_default, (smem, bar, FAILURE, blocked_layout)),
            (ws_1, (smem, bar, FAILURE, blocked_layout)),
        ], [4], [32])
        mbarrier.wait(bar.index(1), phase=0)
        val = smem.index(0).load(blocked_layout)
        output_ptrs = output + ttgl.arange(0, block_x, blocked_layout)
        ttgl.store(output_ptrs, val)
```
- **EN:** Prepares or updates state through `val`, `output_ptrs`. Invokes `ttgl.warp_specialize`, `mbarrier.wait`, `bar.index`, `smem.index`, `ttgl.arange`, `ttgl.store` to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `val`、`output_ptrs` 准备或更新状态。 调用 `ttgl.warp_specialize`、`mbarrier.wait`、`bar.index`、`smem.index`、`ttgl.arange`、`ttgl.store` 执行测试逻辑。 相关主题：布局变换推理。

#### Lines 1706-1708

```python

    output = torch.empty((XBLOCK.value * num_ctas, ), device=device, dtype=torch.float16)
    ws_kernel[(1, )](output, FAILURE=FAILURE, num_warps=4, num_ctas=num_ctas)
```
- **EN:** Prepares or updates state through `output`. Invokes `torch.empty` to execute the test logic. Relevant themes: PyTorch tensor setup and checks.
- **CN:** 通过 `output` 准备或更新状态。 调用 `torch.empty` 执行测试逻辑。 相关主题：PyTorch 张量准备与校验。

### Lines 1709-1713

```python


@pytest.mark.skipif(not is_cuda() or torch.cuda.get_device_capability()[0] < 9, reason="Requires hopper")
@pytest.mark.parametrize("FAILURE", [True, False])
def test_ws_load_wait_store(FAILURE, device, run_wrapper, monkeypatch, num_ctas):
```
- **EN:** Defines the test function `test_ws_load_wait_store`. Decorators: `pytest.mark.skipif(not is_cuda() or torch.cuda.get_device_capability()[0] < 9, reason='Requires hopper')`, `pytest.mark.parametrize('FAILURE', [True, False])`. Parameters: `FAILURE`, `device`, `run_wrapper`, `monkeypatch`, `num_ctas`. Nested definitions in this scope: `ws_default`, `ws_1`, `ws_kernel`. Key calls include `pytest.mark.skipif`, `pytest.mark.parametrize`, `monkeypatch.setenv`, `knobs.refresh_knobs`, `torch.empty`, `run_in_process`, and 17 more. This scope touches pytest parametrization, PyTorch tensor setup and checks, runtime driver interaction, kernel launch orchestration.
- **CN:** 定义测试函数 `test_ws_load_wait_store`。 装饰器：`pytest.mark.skipif(not is_cuda() or torch.cuda.get_device_capability()[0] < 9, reason='Requires hopper')`、`pytest.mark.parametrize('FAILURE', [True, False])`。 参数：`FAILURE`、`device`、`run_wrapper`、`monkeypatch`、`num_ctas`。 该作用域中的嵌套定义：`ws_default`、`ws_1`、`ws_kernel`。 关键调用包括 `pytest.mark.skipif`、`pytest.mark.parametrize`、`monkeypatch.setenv`、`knobs.refresh_knobs`、`torch.empty`、`run_in_process` 等另外 17 项。 该作用域涉及pytest 参数化、PyTorch 张量准备与校验、运行时驱动交互、kernel 启动编排。

#### Lines 1714-1722

```python
    if run_wrapper:
        result = run_in_process(test_ws_load_wait_store, (FAILURE, device, False, monkeypatch, num_ctas))
        if FAILURE:
            assert_expected_cuda_failure(result.exc)
            assert "Buffer being accessed has outstanding" in result.driver_stderr_output
        else:
            assert result.exc is None
            assert result.driver_stderr_output == ""
        return
```
- **EN:** Invokes `run_in_process`, `assert_expected_cuda_failure` to execute the test logic. Validates behavior with 3 assertion(s). Branches on runtime or test conditions. Relevant themes: runtime driver interaction.
- **CN:** 调用 `run_in_process`、`assert_expected_cuda_failure` 执行测试逻辑。 通过 3 个断言验证行为。 根据运行时或测试条件进行分支。 相关主题：运行时驱动交互。

#### Lines 1723-1725

```python
    monkeypatch.setenv("TRITON_INSTRUMENTATION_MODE", "consan")
    monkeypatch.setenv("CUDA_LAUNCH_BLOCKING", "1")
    knobs.refresh_knobs()
```
- **EN:** Invokes `monkeypatch.setenv`, `knobs.refresh_knobs` to execute the test logic. Relevant themes: kernel launch orchestration.
- **CN:** 调用 `monkeypatch.setenv`、`knobs.refresh_knobs` 执行测试逻辑。 相关主题：kernel 启动编排。

#### Lines 1726-1728

```python

    @gluon.jit
    def ws_default(smem, bar, FAILURE: ttgl.constexpr, layout: ttgl.constexpr):
```
- **EN:** Defines the helper function `ws_default`. Decorators: `gluon.jit`. Parameters: `smem`, `bar`, `FAILURE`, `layout`. Key calls include `mbarrier.wait`, `mbarrier.arrive`, `bar.index`, `ttgl.num_ctas`, `smem.index`, `ttgl.arange`. This scope touches layout transformation reasoning.
- **CN:** 定义辅助函数 `ws_default`。 装饰器：`gluon.jit`。 参数：`smem`、`bar`、`FAILURE`、`layout`。 关键调用包括 `mbarrier.wait`、`mbarrier.arrive`、`bar.index`、`ttgl.num_ctas`、`smem.index`、`ttgl.arange`。 该作用域涉及布局变换推理。

##### Lines 1729-1732

```python
        mbarrier.wait(bar.index(0), phase=0, pred=(not FAILURE))
        block_x: ttgl.constexpr = XBLOCK * ttgl.num_ctas()
        smem.index(0).store(ttgl.arange(0, block_x, layout).to(ttgl.float16))
        mbarrier.arrive(bar.index(1), count=1)
```
- **EN:** Prepares or updates state through `block_x`. Invokes `mbarrier.wait`, `bar.index`, `ttgl.num_ctas`, `smem.index`, `ttgl.arange`, `mbarrier.arrive` to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `block_x` 准备或更新状态。 调用 `mbarrier.wait`、`bar.index`、`ttgl.num_ctas`、`smem.index`、`ttgl.arange`、`mbarrier.arrive` 执行测试逻辑。 相关主题：布局变换推理。

#### Lines 1733-1735

```python

    @gluon.jit
    def ws_1(smem, bar, FAILURE: ttgl.constexpr, layout: ttgl.constexpr):
```
- **EN:** Defines the helper function `ws_1`. Decorators: `gluon.jit`. Parameters: `smem`, `bar`, `FAILURE`, `layout`. Key calls include `mbarrier.arrive`, `bar.index`, `smem.index`. This scope touches layout transformation reasoning.
- **CN:** 定义辅助函数 `ws_1`。 装饰器：`gluon.jit`。 参数：`smem`、`bar`、`FAILURE`、`layout`。 关键调用包括 `mbarrier.arrive`、`bar.index`、`smem.index`。 该作用域涉及布局变换推理。

##### Lines 1736-1738

```python
        val = smem.index(0).load(layout)
        mbarrier.arrive(bar.index(0), count=1)
        smem.index(1).store(val)  # dummy store to make sure the load is executed
```
- **EN:** Prepares or updates state through `val`. Invokes `smem.index`, `mbarrier.arrive`, `bar.index` to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `val` 准备或更新状态。 调用 `smem.index`、`mbarrier.arrive`、`bar.index` 执行测试逻辑。 相关主题：布局变换推理。

#### Lines 1739-1741

```python

    @gluon.jit
    def ws_kernel(output, FAILURE: ttgl.constexpr):
```
- **EN:** Defines the helper function `ws_kernel`. Decorators: `gluon.jit`. Parameters: `output`, `FAILURE`. Key calls include `default_cga_layout`, `ttgl.SwizzledSharedLayout`, `ttgl.BlockedLayout`, `ttgl.allocate_shared_memory`, `mbarrier.allocate_mbarrier`, `ttgl.warp_specialize`, and 7 more. This scope touches layout transformation reasoning.
- **CN:** 定义辅助函数 `ws_kernel`。 装饰器：`gluon.jit`。 参数：`output`、`FAILURE`。 关键调用包括 `default_cga_layout`、`ttgl.SwizzledSharedLayout`、`ttgl.BlockedLayout`、`ttgl.allocate_shared_memory`、`mbarrier.allocate_mbarrier`、`ttgl.warp_specialize` 等另外 7 项。 该作用域涉及布局变换推理。

##### Lines 1742-1749

```python
        block_x: ttgl.constexpr = XBLOCK * ttgl.num_ctas()
        cga_layout: ttgl.constexpr = default_cga_layout(ttgl.num_ctas(), 1)
        smem_layout: ttgl.constexpr = ttgl.SwizzledSharedLayout(vec=1, per_phase=1, max_phase=1, order=[0],
                                                                cga_layout=cga_layout)
        blocked_layout: ttgl.constexpr = ttgl.BlockedLayout(size_per_thread=[1], threads_per_warp=[32],
                                                            warps_per_cta=[4], order=[0], cga_layout=cga_layout)
        smem = ttgl.allocate_shared_memory(ttgl.float16, [2, block_x], smem_layout)
        bar = mbarrier.allocate_mbarrier(batch=2)
```
- **EN:** Prepares or updates state through `block_x`, `cga_layout`, `smem_layout`, `blocked_layout`, `smem`, `bar`. Invokes `ttgl.num_ctas`, `default_cga_layout`, `ttgl.SwizzledSharedLayout`, `ttgl.BlockedLayout`, `ttgl.allocate_shared_memory`, `mbarrier.allocate_mbarrier` to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `block_x`、`cga_layout`、`smem_layout`、`blocked_layout`、`smem`、`bar` 准备或更新状态。 调用 `ttgl.num_ctas`、`default_cga_layout`、`ttgl.SwizzledSharedLayout`、`ttgl.BlockedLayout`、`ttgl.allocate_shared_memory`、`mbarrier.allocate_mbarrier` 执行测试逻辑。 相关主题：布局变换推理。

##### Lines 1750-1751

```python
        for i in range(2):
            mbarrier.init(bar.index(i), count=1)
```
- **EN:** Invokes `mbarrier.init`, `bar.index` to execute the test logic. Iterates across cases or data tiles.
- **CN:** 调用 `mbarrier.init`、`bar.index` 执行测试逻辑。 通过循环覆盖多个用例或数据分块。

##### Lines 1752-1759

```python
        ttgl.warp_specialize([
            (ws_default, (smem, bar, FAILURE, blocked_layout)),
            (ws_1, (smem, bar, FAILURE, blocked_layout)),
        ], [4], [32])
        mbarrier.wait(bar.index(1), phase=0)
        val = smem.index(0).load(blocked_layout)
        output_ptrs = output + ttgl.arange(0, block_x, blocked_layout)
        ttgl.store(output_ptrs, val)
```
- **EN:** Prepares or updates state through `val`, `output_ptrs`. Invokes `ttgl.warp_specialize`, `mbarrier.wait`, `bar.index`, `smem.index`, `ttgl.arange`, `ttgl.store` to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `val`、`output_ptrs` 准备或更新状态。 调用 `ttgl.warp_specialize`、`mbarrier.wait`、`bar.index`、`smem.index`、`ttgl.arange`、`ttgl.store` 执行测试逻辑。 相关主题：布局变换推理。

#### Lines 1760-1762

```python

    output = torch.empty((XBLOCK.value * num_ctas, ), device=device, dtype=torch.float16)
    ws_kernel[(1, )](output, FAILURE=FAILURE, num_warps=4, num_ctas=num_ctas)
```
- **EN:** Prepares or updates state through `output`. Invokes `torch.empty` to execute the test logic. Relevant themes: PyTorch tensor setup and checks.
- **CN:** 通过 `output` 准备或更新状态。 调用 `torch.empty` 执行测试逻辑。 相关主题：PyTorch 张量准备与校验。

### Lines 1763-1767

```python


@pytest.mark.skipif(not is_cuda() or torch.cuda.get_device_capability()[0] < 9, reason="Requires hopper")
@pytest.mark.parametrize("MISSING_BAR", ["none", "1", "2"])
def test_ws_two_loads_two_bars(MISSING_BAR, device, run_wrapper, monkeypatch, num_ctas):
```
- **EN:** Defines the test function `test_ws_two_loads_two_bars`. Decorators: `pytest.mark.skipif(not is_cuda() or torch.cuda.get_device_capability()[0] < 9, reason='Requires hopper')`, `pytest.mark.parametrize('MISSING_BAR', ['none', '1', '2'])`. Parameters: `MISSING_BAR`, `device`, `run_wrapper`, `monkeypatch`, `num_ctas`. Nested definitions in this scope: `ws_default`, `ws_1`, `ws_2`, `kernel`. Key calls include `pytest.mark.skipif`, `pytest.mark.parametrize`, `monkeypatch.setenv`, `knobs.refresh_knobs`, `torch.empty`, `run_in_process`, and 17 more. This scope touches pytest parametrization, PyTorch tensor setup and checks, runtime driver interaction, kernel launch orchestration.
- **CN:** 定义测试函数 `test_ws_two_loads_two_bars`。 装饰器：`pytest.mark.skipif(not is_cuda() or torch.cuda.get_device_capability()[0] < 9, reason='Requires hopper')`、`pytest.mark.parametrize('MISSING_BAR', ['none', '1', '2'])`。 参数：`MISSING_BAR`、`device`、`run_wrapper`、`monkeypatch`、`num_ctas`。 该作用域中的嵌套定义：`ws_default`、`ws_1`、`ws_2`、`kernel`。 关键调用包括 `pytest.mark.skipif`、`pytest.mark.parametrize`、`monkeypatch.setenv`、`knobs.refresh_knobs`、`torch.empty`、`run_in_process` 等另外 17 项。 该作用域涉及pytest 参数化、PyTorch 张量准备与校验、运行时驱动交互、kernel 启动编排。

#### Lines 1768-1776

```python
    if run_wrapper:
        result = run_in_process(test_ws_two_loads_two_bars, (MISSING_BAR, device, False, monkeypatch, num_ctas))
        if MISSING_BAR != "none":
            assert_expected_cuda_failure(result.exc)
            assert "Buffer being accessed has outstanding" in result.driver_stderr_output
        else:
            assert result.exc is None
            assert result.driver_stderr_output == ""
        return
```
- **EN:** Invokes `run_in_process`, `assert_expected_cuda_failure` to execute the test logic. Validates behavior with 3 assertion(s). Branches on runtime or test conditions. Relevant themes: runtime driver interaction.
- **CN:** 调用 `run_in_process`、`assert_expected_cuda_failure` 执行测试逻辑。 通过 3 个断言验证行为。 根据运行时或测试条件进行分支。 相关主题：运行时驱动交互。

#### Lines 1777-1779

```python
    monkeypatch.setenv("TRITON_INSTRUMENTATION_MODE", "consan")
    monkeypatch.setenv("CUDA_LAUNCH_BLOCKING", "1")
    knobs.refresh_knobs()
```
- **EN:** Invokes `monkeypatch.setenv`, `knobs.refresh_knobs` to execute the test logic. Relevant themes: kernel launch orchestration.
- **CN:** 调用 `monkeypatch.setenv`、`knobs.refresh_knobs` 执行测试逻辑。 相关主题：kernel 启动编排。

#### Lines 1780-1782

```python

    @gluon.jit
    def ws_default(smem, bar, MISSING_BAR: ttgl.constexpr, layout: ttgl.constexpr):
```
- **EN:** Defines the helper function `ws_default`. Decorators: `gluon.jit`. Parameters: `smem`, `bar`, `MISSING_BAR`, `layout`. Key calls include `mbarrier.arrive`, `bar.index`, `smem.index`. This scope touches layout transformation reasoning.
- **CN:** 定义辅助函数 `ws_default`。 装饰器：`gluon.jit`。 参数：`smem`、`bar`、`MISSING_BAR`、`layout`。 关键调用包括 `mbarrier.arrive`、`bar.index`、`smem.index`。 该作用域涉及布局变换推理。

##### Lines 1783-1785

```python
        val = smem.index(0).load(layout)
        mbarrier.arrive(bar.index(0), count=1)
        smem.index(1).store(val)  # dummy store to make sure the load is executed
```
- **EN:** Prepares or updates state through `val`. Invokes `smem.index`, `mbarrier.arrive`, `bar.index` to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `val` 准备或更新状态。 调用 `smem.index`、`mbarrier.arrive`、`bar.index` 执行测试逻辑。 相关主题：布局变换推理。

#### Lines 1786-1788

```python

    @gluon.jit
    def ws_1(smem, bar, MISSING_BAR: ttgl.constexpr, layout: ttgl.constexpr):
```
- **EN:** Defines the helper function `ws_1`. Decorators: `gluon.jit`. Parameters: `smem`, `bar`, `MISSING_BAR`, `layout`. Key calls include `mbarrier.arrive`, `bar.index`, `smem.index`. This scope touches layout transformation reasoning.
- **CN:** 定义辅助函数 `ws_1`。 装饰器：`gluon.jit`。 参数：`smem`、`bar`、`MISSING_BAR`、`layout`。 关键调用包括 `mbarrier.arrive`、`bar.index`、`smem.index`。 该作用域涉及布局变换推理。

##### Lines 1789-1791

```python
        val = smem.index(0).load(layout)
        mbarrier.arrive(bar.index(1), count=1)
        smem.index(2).store(val)  # dummy store to make sure the load is executed
```
- **EN:** Prepares or updates state through `val`. Invokes `smem.index`, `mbarrier.arrive`, `bar.index` to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `val` 准备或更新状态。 调用 `smem.index`、`mbarrier.arrive`、`bar.index` 执行测试逻辑。 相关主题：布局变换推理。

#### Lines 1792-1794

```python

    @gluon.jit
    def ws_2(smem, bar, MISSING_BAR: ttgl.constexpr, layout: ttgl.constexpr):
```
- **EN:** Defines the helper function `ws_2`. Decorators: `gluon.jit`. Parameters: `smem`, `bar`, `MISSING_BAR`, `layout`. Key calls include `mbarrier.arrive`, `mbarrier.wait`, `ttgl.num_ctas`, `bar.index`, `smem.index`, `ttgl.arange`. This scope touches layout transformation reasoning.
- **CN:** 定义辅助函数 `ws_2`。 装饰器：`gluon.jit`。 参数：`smem`、`bar`、`MISSING_BAR`、`layout`。 关键调用包括 `mbarrier.arrive`、`mbarrier.wait`、`ttgl.num_ctas`、`bar.index`、`smem.index`、`ttgl.arange`。 该作用域涉及布局变换推理。

##### Lines 1795-1796

```python
        if MISSING_BAR != "1":
            mbarrier.wait(bar.index(0), phase=0)
```
- **EN:** Invokes `mbarrier.wait`, `bar.index` to execute the test logic. Branches on runtime or test conditions.
- **CN:** 调用 `mbarrier.wait`、`bar.index` 执行测试逻辑。 根据运行时或测试条件进行分支。

##### Lines 1797-1798

```python
        if MISSING_BAR != "2":
            mbarrier.wait(bar.index(1), phase=0)
```
- **EN:** Invokes `mbarrier.wait`, `bar.index` to execute the test logic. Branches on runtime or test conditions.
- **CN:** 调用 `mbarrier.wait`、`bar.index` 执行测试逻辑。 根据运行时或测试条件进行分支。

##### Lines 1799-1801

```python
        block_x: ttgl.constexpr = XBLOCK * ttgl.num_ctas()
        smem.index(0).store(ttgl.arange(0, block_x, layout).to(ttgl.float16))
        mbarrier.arrive(bar.index(2), count=1)
```
- **EN:** Prepares or updates state through `block_x`. Invokes `ttgl.num_ctas`, `smem.index`, `ttgl.arange`, `mbarrier.arrive`, `bar.index` to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `block_x` 准备或更新状态。 调用 `ttgl.num_ctas`、`smem.index`、`ttgl.arange`、`mbarrier.arrive`、`bar.index` 执行测试逻辑。 相关主题：布局变换推理。

#### Lines 1802-1804

```python

    @gluon.jit
    def kernel(output, MISSING_BAR: ttgl.constexpr):
```
- **EN:** Defines the helper function `kernel`. Decorators: `gluon.jit`. Parameters: `output`, `MISSING_BAR`. Key calls include `default_cga_layout`, `ttgl.SwizzledSharedLayout`, `ttgl.BlockedLayout`, `ttgl.allocate_shared_memory`, `mbarrier.allocate_mbarrier`, `ttgl.warp_specialize`, and 7 more. This scope touches layout transformation reasoning.
- **CN:** 定义辅助函数 `kernel`。 装饰器：`gluon.jit`。 参数：`output`、`MISSING_BAR`。 关键调用包括 `default_cga_layout`、`ttgl.SwizzledSharedLayout`、`ttgl.BlockedLayout`、`ttgl.allocate_shared_memory`、`mbarrier.allocate_mbarrier`、`ttgl.warp_specialize` 等另外 7 项。 该作用域涉及布局变换推理。

##### Lines 1805-1812

```python
        block_x: ttgl.constexpr = XBLOCK * ttgl.num_ctas()
        cga_layout: ttgl.constexpr = default_cga_layout(ttgl.num_ctas(), 1)
        smem_layout: ttgl.constexpr = ttgl.SwizzledSharedLayout(vec=1, per_phase=1, max_phase=1, order=[0],
                                                                cga_layout=cga_layout)
        blocked_layout: ttgl.constexpr = ttgl.BlockedLayout(size_per_thread=[1], threads_per_warp=[32],
                                                            warps_per_cta=[4], order=[0], cga_layout=cga_layout)
        smem = ttgl.allocate_shared_memory(ttgl.float16, [3, block_x], smem_layout)
        bar = mbarrier.allocate_mbarrier(batch=3)
```
- **EN:** Prepares or updates state through `block_x`, `cga_layout`, `smem_layout`, `blocked_layout`, `smem`, `bar`. Invokes `ttgl.num_ctas`, `default_cga_layout`, `ttgl.SwizzledSharedLayout`, `ttgl.BlockedLayout`, `ttgl.allocate_shared_memory`, `mbarrier.allocate_mbarrier` to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `block_x`、`cga_layout`、`smem_layout`、`blocked_layout`、`smem`、`bar` 准备或更新状态。 调用 `ttgl.num_ctas`、`default_cga_layout`、`ttgl.SwizzledSharedLayout`、`ttgl.BlockedLayout`、`ttgl.allocate_shared_memory`、`mbarrier.allocate_mbarrier` 执行测试逻辑。 相关主题：布局变换推理。

##### Lines 1813-1814

```python
        for i in range(3):
            mbarrier.init(bar.index(i), count=1)
```
- **EN:** Invokes `mbarrier.init`, `bar.index` to execute the test logic. Iterates across cases or data tiles.
- **CN:** 调用 `mbarrier.init`、`bar.index` 执行测试逻辑。 通过循环覆盖多个用例或数据分块。

##### Lines 1815-1823

```python
        ttgl.warp_specialize([
            (ws_default, (smem, bar, MISSING_BAR, blocked_layout)),
            (ws_1, (smem, bar, MISSING_BAR, blocked_layout)),
            (ws_2, (smem, bar, MISSING_BAR, blocked_layout)),
        ], [4, 4], [32, 32])
        mbarrier.wait(bar.index(2), phase=0)
        val = smem.index(0).load(blocked_layout)
        output_ptrs = output + ttgl.arange(0, block_x, blocked_layout)
        ttgl.store(output_ptrs, val)
```
- **EN:** Prepares or updates state through `val`, `output_ptrs`. Invokes `ttgl.warp_specialize`, `mbarrier.wait`, `bar.index`, `smem.index`, `ttgl.arange`, `ttgl.store` to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `val`、`output_ptrs` 准备或更新状态。 调用 `ttgl.warp_specialize`、`mbarrier.wait`、`bar.index`、`smem.index`、`ttgl.arange`、`ttgl.store` 执行测试逻辑。 相关主题：布局变换推理。

#### Lines 1824-1826

```python

    output = torch.empty((XBLOCK.value * num_ctas, ), device=device, dtype=torch.float16)
    kernel[(1, )](output, MISSING_BAR=MISSING_BAR, num_warps=4, num_ctas=num_ctas)
```
- **EN:** Prepares or updates state through `output`. Invokes `torch.empty` to execute the test logic. Relevant themes: PyTorch tensor setup and checks.
- **CN:** 通过 `output` 准备或更新状态。 调用 `torch.empty` 执行测试逻辑。 相关主题：PyTorch 张量准备与校验。

### Lines 1827-1831

```python


@pytest.mark.skipif(not is_cuda() or torch.cuda.get_device_capability()[0] < 9, reason="Requires hopper")
@pytest.mark.parametrize("FAILURE", [True, False])
def test_ws_two_loads_one_bar(FAILURE, device, run_wrapper, monkeypatch, num_ctas):
```
- **EN:** Defines the test function `test_ws_two_loads_one_bar`. Decorators: `pytest.mark.skipif(not is_cuda() or torch.cuda.get_device_capability()[0] < 9, reason='Requires hopper')`, `pytest.mark.parametrize('FAILURE', [True, False])`. Parameters: `FAILURE`, `device`, `run_wrapper`, `monkeypatch`, `num_ctas`. Nested definitions in this scope: `ws_default`, `ws_1`, `ws_2`, `kernel`. Key calls include `pytest.mark.skipif`, `pytest.mark.parametrize`, `monkeypatch.setenv`, `knobs.refresh_knobs`, `torch.empty`, `run_in_process`, and 17 more. This scope touches pytest parametrization, PyTorch tensor setup and checks, runtime driver interaction, kernel launch orchestration.
- **CN:** 定义测试函数 `test_ws_two_loads_one_bar`。 装饰器：`pytest.mark.skipif(not is_cuda() or torch.cuda.get_device_capability()[0] < 9, reason='Requires hopper')`、`pytest.mark.parametrize('FAILURE', [True, False])`。 参数：`FAILURE`、`device`、`run_wrapper`、`monkeypatch`、`num_ctas`。 该作用域中的嵌套定义：`ws_default`、`ws_1`、`ws_2`、`kernel`。 关键调用包括 `pytest.mark.skipif`、`pytest.mark.parametrize`、`monkeypatch.setenv`、`knobs.refresh_knobs`、`torch.empty`、`run_in_process` 等另外 17 项。 该作用域涉及pytest 参数化、PyTorch 张量准备与校验、运行时驱动交互、kernel 启动编排。

#### Lines 1832-1840

```python
    if run_wrapper:
        result = run_in_process(test_ws_two_loads_one_bar, (FAILURE, device, False, monkeypatch, num_ctas))
        if FAILURE:
            assert_expected_cuda_failure(result.exc)
            assert "Buffer being accessed has outstanding" in result.driver_stderr_output
        else:
            assert result.exc is None
            assert result.driver_stderr_output == ""
        return
```
- **EN:** Invokes `run_in_process`, `assert_expected_cuda_failure` to execute the test logic. Validates behavior with 3 assertion(s). Branches on runtime or test conditions. Relevant themes: runtime driver interaction.
- **CN:** 调用 `run_in_process`、`assert_expected_cuda_failure` 执行测试逻辑。 通过 3 个断言验证行为。 根据运行时或测试条件进行分支。 相关主题：运行时驱动交互。

#### Lines 1841-1843

```python
    monkeypatch.setenv("TRITON_INSTRUMENTATION_MODE", "consan")
    monkeypatch.setenv("CUDA_LAUNCH_BLOCKING", "1")
    knobs.refresh_knobs()
```
- **EN:** Invokes `monkeypatch.setenv`, `knobs.refresh_knobs` to execute the test logic. Relevant themes: kernel launch orchestration.
- **CN:** 调用 `monkeypatch.setenv`、`knobs.refresh_knobs` 执行测试逻辑。 相关主题：kernel 启动编排。

#### Lines 1844-1846

```python

    @gluon.jit
    def ws_default(smem, bar, FAILURE: ttgl.constexpr, layout: ttgl.constexpr):
```
- **EN:** Defines the helper function `ws_default`. Decorators: `gluon.jit`. Parameters: `smem`, `bar`, `FAILURE`, `layout`. Key calls include `mbarrier.arrive`, `bar.index`, `smem.index`. This scope touches layout transformation reasoning.
- **CN:** 定义辅助函数 `ws_default`。 装饰器：`gluon.jit`。 参数：`smem`、`bar`、`FAILURE`、`layout`。 关键调用包括 `mbarrier.arrive`、`bar.index`、`smem.index`。 该作用域涉及布局变换推理。

##### Lines 1847-1849

```python
        val = smem.index(0).load(layout)
        mbarrier.arrive(bar.index(0), count=1)
        smem.index(1).store(val)  # dummy store to make sure the load is executed
```
- **EN:** Prepares or updates state through `val`. Invokes `smem.index`, `mbarrier.arrive`, `bar.index` to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `val` 准备或更新状态。 调用 `smem.index`、`mbarrier.arrive`、`bar.index` 执行测试逻辑。 相关主题：布局变换推理。

#### Lines 1850-1852

```python

    @gluon.jit
    def ws_1(smem, bar, FAILURE: ttgl.constexpr, layout: ttgl.constexpr):
```
- **EN:** Defines the helper function `ws_1`. Decorators: `gluon.jit`. Parameters: `smem`, `bar`, `FAILURE`, `layout`. Key calls include `mbarrier.arrive`, `bar.index`, `smem.index`. This scope touches layout transformation reasoning.
- **CN:** 定义辅助函数 `ws_1`。 装饰器：`gluon.jit`。 参数：`smem`、`bar`、`FAILURE`、`layout`。 关键调用包括 `mbarrier.arrive`、`bar.index`、`smem.index`。 该作用域涉及布局变换推理。

##### Lines 1853-1855

```python
        val = smem.index(0).load(layout)
        mbarrier.arrive(bar.index(0), count=1)
        smem.index(2).store(val)  # dummy store to make sure the load is executed
```
- **EN:** Prepares or updates state through `val`. Invokes `smem.index`, `mbarrier.arrive`, `bar.index` to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `val` 准备或更新状态。 调用 `smem.index`、`mbarrier.arrive`、`bar.index` 执行测试逻辑。 相关主题：布局变换推理。

#### Lines 1856-1858

```python

    @gluon.jit
    def ws_2(smem, bar, FAILURE: ttgl.constexpr, layout: ttgl.constexpr):
```
- **EN:** Defines the helper function `ws_2`. Decorators: `gluon.jit`. Parameters: `smem`, `bar`, `FAILURE`, `layout`. Key calls include `mbarrier.wait`, `mbarrier.arrive`, `bar.index`, `ttgl.num_ctas`, `smem.index`, `ttgl.arange`. This scope touches layout transformation reasoning.
- **CN:** 定义辅助函数 `ws_2`。 装饰器：`gluon.jit`。 参数：`smem`、`bar`、`FAILURE`、`layout`。 关键调用包括 `mbarrier.wait`、`mbarrier.arrive`、`bar.index`、`ttgl.num_ctas`、`smem.index`、`ttgl.arange`。 该作用域涉及布局变换推理。

##### Lines 1859-1862

```python
        mbarrier.wait(bar.index(0), phase=0, pred=(not FAILURE), deps=[smem.index(0)])
        block_x: ttgl.constexpr = XBLOCK * ttgl.num_ctas()
        smem.index(0).store(ttgl.arange(0, block_x, layout).to(ttgl.float16))
        mbarrier.arrive(bar.index(1), count=1)
```
- **EN:** Prepares or updates state through `block_x`. Invokes `mbarrier.wait`, `bar.index`, `smem.index`, `ttgl.num_ctas`, `ttgl.arange`, `mbarrier.arrive` to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `block_x` 准备或更新状态。 调用 `mbarrier.wait`、`bar.index`、`smem.index`、`ttgl.num_ctas`、`ttgl.arange`、`mbarrier.arrive` 执行测试逻辑。 相关主题：布局变换推理。

#### Lines 1863-1865

```python

    @gluon.jit
    def kernel(output, FAILURE: ttgl.constexpr):
```
- **EN:** Defines the helper function `kernel`. Decorators: `gluon.jit`. Parameters: `output`, `FAILURE`. Key calls include `default_cga_layout`, `ttgl.SwizzledSharedLayout`, `ttgl.BlockedLayout`, `ttgl.allocate_shared_memory`, `mbarrier.allocate_mbarrier`, `mbarrier.init`, and 7 more. This scope touches layout transformation reasoning.
- **CN:** 定义辅助函数 `kernel`。 装饰器：`gluon.jit`。 参数：`output`、`FAILURE`。 关键调用包括 `default_cga_layout`、`ttgl.SwizzledSharedLayout`、`ttgl.BlockedLayout`、`ttgl.allocate_shared_memory`、`mbarrier.allocate_mbarrier`、`mbarrier.init` 等另外 7 项。 该作用域涉及布局变换推理。

##### Lines 1866-1884

```python
        block_x: ttgl.constexpr = XBLOCK * ttgl.num_ctas()
        cga_layout: ttgl.constexpr = default_cga_layout(ttgl.num_ctas(), 1)
        smem_layout: ttgl.constexpr = ttgl.SwizzledSharedLayout(vec=1, per_phase=1, max_phase=1, order=[0],
                                                                cga_layout=cga_layout)
        blocked_layout: ttgl.constexpr = ttgl.BlockedLayout(size_per_thread=[1], threads_per_warp=[32],
                                                            warps_per_cta=[4], order=[0], cga_layout=cga_layout)
        smem = ttgl.allocate_shared_memory(ttgl.float16, [3, block_x], smem_layout)
        bar = mbarrier.allocate_mbarrier(batch=2)
        mbarrier.init(bar.index(0), count=2)
        mbarrier.init(bar.index(1), count=1)
        ttgl.warp_specialize([
            (ws_default, (smem, bar, FAILURE, blocked_layout)),
            (ws_1, (smem, bar, FAILURE, blocked_layout)),
            (ws_2, (smem, bar, FAILURE, blocked_layout)),
        ], [4, 4], [32, 32])
        mbarrier.wait(bar.index(1), phase=0, deps=[smem.index(0)])
        val = smem.index(0).load(blocked_layout)
        output_ptrs = output + ttgl.arange(0, block_x, blocked_layout)
        ttgl.store(output_ptrs, val)
```
- **EN:** Prepares or updates state through `block_x`, `cga_layout`, `smem_layout`, `blocked_layout`, `smem`, `bar`, `val`, `output_ptrs`. Invokes `ttgl.num_ctas`, `default_cga_layout`, `ttgl.SwizzledSharedLayout`, `ttgl.BlockedLayout`, `ttgl.allocate_shared_memory`, `mbarrier.allocate_mbarrier`, and 7 more to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `block_x`、`cga_layout`、`smem_layout`、`blocked_layout`、`smem`、`bar`、`val`、`output_ptrs` 准备或更新状态。 调用 `ttgl.num_ctas`、`default_cga_layout`、`ttgl.SwizzledSharedLayout`、`ttgl.BlockedLayout`、`ttgl.allocate_shared_memory`、`mbarrier.allocate_mbarrier` 等另外 7 项 执行测试逻辑。 相关主题：布局变换推理。

#### Lines 1885-1887

```python

    output = torch.empty((XBLOCK.value * num_ctas, ), device=device, dtype=torch.float16)
    kernel[(1, )](output, FAILURE=FAILURE, num_warps=4, num_ctas=num_ctas)
```
- **EN:** Prepares or updates state through `output`. Invokes `torch.empty` to execute the test logic. Relevant themes: PyTorch tensor setup and checks.
- **CN:** 通过 `output` 准备或更新状态。 调用 `torch.empty` 执行测试逻辑。 相关主题：PyTorch 张量准备与校验。

### Lines 1888-1892

```python


@pytest.mark.skipif(not is_cuda() or torch.cuda.get_device_capability()[0] < 9, reason="Requires hopper")
@pytest.mark.parametrize("MISSING_BAR", ["none", "0", "1", "2", "3"])
def test_ws_two_loads_two_bars_loop(MISSING_BAR, device, run_wrapper, monkeypatch, num_ctas):
```
- **EN:** Defines the test function `test_ws_two_loads_two_bars_loop`. Decorators: `pytest.mark.skipif(not is_cuda() or torch.cuda.get_device_capability()[0] < 9, reason='Requires hopper')`, `pytest.mark.parametrize('MISSING_BAR', ['none', '0', '1', '2', '3'])`. Parameters: `MISSING_BAR`, `device`, `run_wrapper`, `monkeypatch`, `num_ctas`. Nested definitions in this scope: `ws_default`, `ws_1`, `ws_2`, `kernel`. Key calls include `pytest.mark.skipif`, `pytest.mark.parametrize`, `monkeypatch.setenv`, `knobs.refresh_knobs`, `torch.empty`, `run_in_process`, and 17 more. This scope touches pytest parametrization, PyTorch tensor setup and checks, runtime driver interaction, kernel launch orchestration.
- **CN:** 定义测试函数 `test_ws_two_loads_two_bars_loop`。 装饰器：`pytest.mark.skipif(not is_cuda() or torch.cuda.get_device_capability()[0] < 9, reason='Requires hopper')`、`pytest.mark.parametrize('MISSING_BAR', ['none', '0', '1', '2', '3'])`。 参数：`MISSING_BAR`、`device`、`run_wrapper`、`monkeypatch`、`num_ctas`。 该作用域中的嵌套定义：`ws_default`、`ws_1`、`ws_2`、`kernel`。 关键调用包括 `pytest.mark.skipif`、`pytest.mark.parametrize`、`monkeypatch.setenv`、`knobs.refresh_knobs`、`torch.empty`、`run_in_process` 等另外 17 项。 该作用域涉及pytest 参数化、PyTorch 张量准备与校验、运行时驱动交互、kernel 启动编排。

#### Lines 1893-1901

```python
    if run_wrapper:
        result = run_in_process(test_ws_two_loads_two_bars_loop, (MISSING_BAR, device, False, monkeypatch, num_ctas))
        if MISSING_BAR != "none":
            assert_expected_cuda_failure(result.exc)
            assert "Buffer being accessed has outstanding" in result.driver_stderr_output
        else:
            assert result.exc is None
            assert result.driver_stderr_output == ""
        return
```
- **EN:** Invokes `run_in_process`, `assert_expected_cuda_failure` to execute the test logic. Validates behavior with 3 assertion(s). Branches on runtime or test conditions. Relevant themes: runtime driver interaction.
- **CN:** 调用 `run_in_process`、`assert_expected_cuda_failure` 执行测试逻辑。 通过 3 个断言验证行为。 根据运行时或测试条件进行分支。 相关主题：运行时驱动交互。

#### Lines 1902-1904

```python
    monkeypatch.setenv("TRITON_INSTRUMENTATION_MODE", "consan")
    monkeypatch.setenv("CUDA_LAUNCH_BLOCKING", "1")
    knobs.refresh_knobs()
```
- **EN:** Invokes `monkeypatch.setenv`, `knobs.refresh_knobs` to execute the test logic. Relevant themes: kernel launch orchestration.
- **CN:** 调用 `monkeypatch.setenv`、`knobs.refresh_knobs` 执行测试逻辑。 相关主题：kernel 启动编排。

#### Lines 1905-1907

```python

    @gluon.jit
    def ws_default(smem, bar, MISSING_BAR: ttgl.constexpr, layout: ttgl.constexpr):
```
- **EN:** Defines the helper function `ws_default`. Decorators: `gluon.jit`. Parameters: `smem`, `bar`, `MISSING_BAR`, `layout`. Key calls include `ttgl.zeros`, `ttgl.num_ctas`, `mbarrier.arrive`, `mbarrier.wait`, `bar.index`, `smem.index`. This scope touches layout transformation reasoning.
- **CN:** 定义辅助函数 `ws_default`。 装饰器：`gluon.jit`。 参数：`smem`、`bar`、`MISSING_BAR`、`layout`。 关键调用包括 `ttgl.zeros`、`ttgl.num_ctas`、`mbarrier.arrive`、`mbarrier.wait`、`bar.index`、`smem.index`。 该作用域涉及布局变换推理。

##### Lines 1908-1910

```python
        block_x: ttgl.constexpr = XBLOCK * ttgl.num_ctas()
        acc = ttgl.zeros([block_x], ttgl.float16, layout)
        phase = 0
```
- **EN:** Prepares or updates state through `block_x`, `acc`, `phase`. Invokes `ttgl.num_ctas`, `ttgl.zeros` to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `block_x`、`acc`、`phase` 准备或更新状态。 调用 `ttgl.num_ctas`、`ttgl.zeros` 执行测试逻辑。 相关主题：布局变换推理。

##### Lines 1911-1917

```python
        for _ in range(10):
            if MISSING_BAR != "2":
                mbarrier.wait(bar.index(2), phase=phase)
            phase = (phase + 1) % 2
            val = smem.index(0).load(layout)
            mbarrier.arrive(bar.index(0), count=1)
            acc = acc + val
```
- **EN:** Invokes `mbarrier.arrive`, `mbarrier.wait`, `bar.index`, `smem.index` to execute the test logic. Iterates across cases or data tiles. Relevant themes: layout transformation reasoning.
- **CN:** 调用 `mbarrier.arrive`、`mbarrier.wait`、`bar.index`、`smem.index` 执行测试逻辑。 通过循环覆盖多个用例或数据分块。 相关主题：布局变换推理。

##### Lines 1918-1918

```python
        smem.index(1).store(acc)  # dummy store to make sure the load is executed
```
- **EN:** Invokes `smem.index` to execute the test logic.
- **CN:** 调用 `smem.index` 执行测试逻辑。

#### Lines 1919-1921

```python

    @gluon.jit
    def ws_1(smem, bar, MISSING_BAR: ttgl.constexpr, layout: ttgl.constexpr):
```
- **EN:** Defines the helper function `ws_1`. Decorators: `gluon.jit`. Parameters: `smem`, `bar`, `MISSING_BAR`, `layout`. Key calls include `ttgl.zeros`, `ttgl.num_ctas`, `mbarrier.arrive`, `mbarrier.wait`, `bar.index`, `smem.index`. This scope touches layout transformation reasoning.
- **CN:** 定义辅助函数 `ws_1`。 装饰器：`gluon.jit`。 参数：`smem`、`bar`、`MISSING_BAR`、`layout`。 关键调用包括 `ttgl.zeros`、`ttgl.num_ctas`、`mbarrier.arrive`、`mbarrier.wait`、`bar.index`、`smem.index`。 该作用域涉及布局变换推理。

##### Lines 1922-1924

```python
        block_x: ttgl.constexpr = XBLOCK * ttgl.num_ctas()
        acc = ttgl.zeros([block_x], ttgl.float16, layout)
        phase = 0
```
- **EN:** Prepares or updates state through `block_x`, `acc`, `phase`. Invokes `ttgl.num_ctas`, `ttgl.zeros` to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `block_x`、`acc`、`phase` 准备或更新状态。 调用 `ttgl.num_ctas`、`ttgl.zeros` 执行测试逻辑。 相关主题：布局变换推理。

##### Lines 1925-1931

```python
        for _ in range(10):
            if MISSING_BAR != "3":
                mbarrier.wait(bar.index(3), phase=phase)
            phase = (phase + 1) % 2
            val = smem.index(0).load(layout)
            mbarrier.arrive(bar.index(1), count=1)
            acc = acc + val
```
- **EN:** Invokes `mbarrier.arrive`, `mbarrier.wait`, `bar.index`, `smem.index` to execute the test logic. Iterates across cases or data tiles. Relevant themes: layout transformation reasoning.
- **CN:** 调用 `mbarrier.arrive`、`mbarrier.wait`、`bar.index`、`smem.index` 执行测试逻辑。 通过循环覆盖多个用例或数据分块。 相关主题：布局变换推理。

##### Lines 1932-1932

```python
        smem.index(2).store(acc)  # dummy store to make sure the load is executed
```
- **EN:** Invokes `smem.index` to execute the test logic.
- **CN:** 调用 `smem.index` 执行测试逻辑。

#### Lines 1933-1935

```python

    @gluon.jit
    def ws_2(smem, bar, MISSING_BAR: ttgl.constexpr, layout: ttgl.constexpr):
```
- **EN:** Defines the helper function `ws_2`. Decorators: `gluon.jit`. Parameters: `smem`, `bar`, `MISSING_BAR`, `layout`. Key calls include `ttgl.num_ctas`, `mbarrier.arrive`, `mbarrier.wait`, `bar.index`, `smem.index`, `ttgl.arange`. This scope touches layout transformation reasoning.
- **CN:** 定义辅助函数 `ws_2`。 装饰器：`gluon.jit`。 参数：`smem`、`bar`、`MISSING_BAR`、`layout`。 关键调用包括 `ttgl.num_ctas`、`mbarrier.arrive`、`mbarrier.wait`、`bar.index`、`smem.index`、`ttgl.arange`。 该作用域涉及布局变换推理。

##### Lines 1936-1937

```python
        block_x: ttgl.constexpr = XBLOCK * ttgl.num_ctas()
        phase = 0
```
- **EN:** Prepares or updates state through `block_x`, `phase`. Invokes `ttgl.num_ctas` to execute the test logic.
- **CN:** 通过 `block_x`、`phase` 准备或更新状态。 调用 `ttgl.num_ctas` 执行测试逻辑。

##### Lines 1938-1946

```python
        for _ in range(10):
            if MISSING_BAR != "0":
                mbarrier.wait(bar.index(0), phase=phase)
            if MISSING_BAR != "1":
                mbarrier.wait(bar.index(1), phase=phase)
            phase = (phase + 1) % 2
            smem.index(0).store(ttgl.arange(0, block_x, layout).to(ttgl.float16))
            mbarrier.arrive(bar.index(2), count=1)
            mbarrier.arrive(bar.index(3), count=1)
```
- **EN:** Invokes `mbarrier.arrive`, `mbarrier.wait`, `bar.index`, `smem.index`, `ttgl.arange` to execute the test logic. Iterates across cases or data tiles. Relevant themes: layout transformation reasoning.
- **CN:** 调用 `mbarrier.arrive`、`mbarrier.wait`、`bar.index`、`smem.index`、`ttgl.arange` 执行测试逻辑。 通过循环覆盖多个用例或数据分块。 相关主题：布局变换推理。

#### Lines 1947-1949

```python

    @gluon.jit
    def kernel(output, MISSING_BAR: ttgl.constexpr):
```
- **EN:** Defines the helper function `kernel`. Decorators: `gluon.jit`. Parameters: `output`, `MISSING_BAR`. Key calls include `default_cga_layout`, `ttgl.SwizzledSharedLayout`, `ttgl.BlockedLayout`, `ttgl.allocate_shared_memory`, `mbarrier.allocate_mbarrier`, `mbarrier.arrive`, and 4 more. This scope touches layout transformation reasoning.
- **CN:** 定义辅助函数 `kernel`。 装饰器：`gluon.jit`。 参数：`output`、`MISSING_BAR`。 关键调用包括 `default_cga_layout`、`ttgl.SwizzledSharedLayout`、`ttgl.BlockedLayout`、`ttgl.allocate_shared_memory`、`mbarrier.allocate_mbarrier`、`mbarrier.arrive` 等另外 4 项。 该作用域涉及布局变换推理。

##### Lines 1950-1957

```python
        block_x: ttgl.constexpr = XBLOCK * ttgl.num_ctas()
        cga_layout: ttgl.constexpr = default_cga_layout(ttgl.num_ctas(), 1)
        smem_layout: ttgl.constexpr = ttgl.SwizzledSharedLayout(vec=1, per_phase=1, max_phase=1, order=[0],
                                                                cga_layout=cga_layout)
        blocked_layout: ttgl.constexpr = ttgl.BlockedLayout(size_per_thread=[1], threads_per_warp=[32],
                                                            warps_per_cta=[4], order=[0], cga_layout=cga_layout)
        smem = ttgl.allocate_shared_memory(ttgl.float16, [3, block_x], smem_layout)
        bar = mbarrier.allocate_mbarrier(batch=4)
```
- **EN:** Prepares or updates state through `block_x`, `cga_layout`, `smem_layout`, `blocked_layout`, `smem`, `bar`. Invokes `ttgl.num_ctas`, `default_cga_layout`, `ttgl.SwizzledSharedLayout`, `ttgl.BlockedLayout`, `ttgl.allocate_shared_memory`, `mbarrier.allocate_mbarrier` to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `block_x`、`cga_layout`、`smem_layout`、`blocked_layout`、`smem`、`bar` 准备或更新状态。 调用 `ttgl.num_ctas`、`default_cga_layout`、`ttgl.SwizzledSharedLayout`、`ttgl.BlockedLayout`、`ttgl.allocate_shared_memory`、`mbarrier.allocate_mbarrier` 执行测试逻辑。 相关主题：布局变换推理。

##### Lines 1958-1959

```python
        for i in range(4):
            mbarrier.init(bar.index(i), count=1)
```
- **EN:** Invokes `mbarrier.init`, `bar.index` to execute the test logic. Iterates across cases or data tiles.
- **CN:** 调用 `mbarrier.init`、`bar.index` 执行测试逻辑。 通过循环覆盖多个用例或数据分块。

##### Lines 1960-1968

```python

        mbarrier.arrive(bar.index(2), count=1)
        mbarrier.arrive(bar.index(3), count=1)

        ttgl.warp_specialize([
            (ws_default, (smem, bar, MISSING_BAR, blocked_layout)),
            (ws_1, (smem, bar, MISSING_BAR, blocked_layout)),
            (ws_2, (smem, bar, MISSING_BAR, blocked_layout)),
        ], [4, 4], [32, 32])
```
- **EN:** Invokes `mbarrier.arrive`, `bar.index`, `ttgl.warp_specialize` to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 调用 `mbarrier.arrive`、`bar.index`、`ttgl.warp_specialize` 执行测试逻辑。 相关主题：布局变换推理。

#### Lines 1969-1971

```python

    output = torch.empty((XBLOCK.value * num_ctas, ), device=device, dtype=torch.float16)
    kernel[(1, )](output, MISSING_BAR=MISSING_BAR, num_warps=4, num_ctas=num_ctas)
```
- **EN:** Prepares or updates state through `output`. Invokes `torch.empty` to execute the test logic. Relevant themes: PyTorch tensor setup and checks.
- **CN:** 通过 `output` 准备或更新状态。 调用 `torch.empty` 执行测试逻辑。 相关主题：PyTorch 张量准备与校验。

### Lines 1972-1976

```python


@pytest.mark.skipif(not is_cuda() or torch.cuda.get_device_capability()[0] < 9, reason="Requires hopper")
@pytest.mark.parametrize("FAILURE", [True, False])
def test_ws_load_ordering(FAILURE, device, run_wrapper, monkeypatch, num_ctas):
```
- **EN:** Defines the test function `test_ws_load_ordering`. Decorators: `pytest.mark.skipif(not is_cuda() or torch.cuda.get_device_capability()[0] < 9, reason='Requires hopper')`, `pytest.mark.parametrize('FAILURE', [True, False])`. Parameters: `FAILURE`, `device`, `run_wrapper`, `monkeypatch`, `num_ctas`. Nested definitions in this scope: `ws_default`, `ws_1`, `kernel`. Key calls include `pytest.mark.skipif`, `pytest.mark.parametrize`, `monkeypatch.setenv`, `knobs.refresh_knobs`, `torch.empty`, `run_in_process`, and 17 more. This scope touches pytest parametrization, PyTorch tensor setup and checks, runtime driver interaction, kernel launch orchestration.
- **CN:** 定义测试函数 `test_ws_load_ordering`。 装饰器：`pytest.mark.skipif(not is_cuda() or torch.cuda.get_device_capability()[0] < 9, reason='Requires hopper')`、`pytest.mark.parametrize('FAILURE', [True, False])`。 参数：`FAILURE`、`device`、`run_wrapper`、`monkeypatch`、`num_ctas`。 该作用域中的嵌套定义：`ws_default`、`ws_1`、`kernel`。 关键调用包括 `pytest.mark.skipif`、`pytest.mark.parametrize`、`monkeypatch.setenv`、`knobs.refresh_knobs`、`torch.empty`、`run_in_process` 等另外 17 项。 该作用域涉及pytest 参数化、PyTorch 张量准备与校验、运行时驱动交互、kernel 启动编排。

#### Lines 1977-1985

```python
    if run_wrapper:
        result = run_in_process(test_ws_load_ordering, (FAILURE, device, False, monkeypatch, num_ctas))
        if FAILURE:
            assert_expected_cuda_failure(result.exc)
            assert "Buffer being accessed has outstanding" in result.driver_stderr_output
        else:
            assert result.exc is None
            assert result.driver_stderr_output == ""
        return
```
- **EN:** Invokes `run_in_process`, `assert_expected_cuda_failure` to execute the test logic. Validates behavior with 3 assertion(s). Branches on runtime or test conditions. Relevant themes: runtime driver interaction.
- **CN:** 调用 `run_in_process`、`assert_expected_cuda_failure` 执行测试逻辑。 通过 3 个断言验证行为。 根据运行时或测试条件进行分支。 相关主题：运行时驱动交互。

#### Lines 1986-1988

```python
    monkeypatch.setenv("TRITON_INSTRUMENTATION_MODE", "consan")
    monkeypatch.setenv("CUDA_LAUNCH_BLOCKING", "1")
    knobs.refresh_knobs()
```
- **EN:** Invokes `monkeypatch.setenv`, `knobs.refresh_knobs` to execute the test logic. Relevant themes: kernel launch orchestration.
- **CN:** 调用 `monkeypatch.setenv`、`knobs.refresh_knobs` 执行测试逻辑。 相关主题：kernel 启动编排。

#### Lines 1989-1991

```python

    @gluon.jit
    def ws_default(smem, bar, FAILURE: ttgl.constexpr, layout: ttgl.constexpr):
```
- **EN:** Defines the helper function `ws_default`. Decorators: `gluon.jit`. Parameters: `smem`, `bar`, `FAILURE`, `layout`. Key calls include `ttgl.num_ctas`, `mbarrier.wait`, `mbarrier.arrive`, `bar.index`, `smem.index`, `ttgl.arange`. This scope touches layout transformation reasoning.
- **CN:** 定义辅助函数 `ws_default`。 装饰器：`gluon.jit`。 参数：`smem`、`bar`、`FAILURE`、`layout`。 关键调用包括 `ttgl.num_ctas`、`mbarrier.wait`、`mbarrier.arrive`、`bar.index`、`smem.index`、`ttgl.arange`。 该作用域涉及布局变换推理。

##### Lines 1992-1993

```python
        block_x: ttgl.constexpr = XBLOCK * ttgl.num_ctas()
        phase = 0
```
- **EN:** Prepares or updates state through `block_x`, `phase`. Invokes `ttgl.num_ctas` to execute the test logic.
- **CN:** 通过 `block_x`、`phase` 准备或更新状态。 调用 `ttgl.num_ctas` 执行测试逻辑。

##### Lines 1994-2000

```python
        for _ in range(10):
            mbarrier.wait(bar.index(2), phase=phase)
            phase = (phase + 1) % 2
            smem.index(0).store(ttgl.arange(0, block_x, layout).to(ttgl.float16))
            mbarrier.arrive(bar.index(0), count=1)
            smem.index(1).store(ttgl.arange(0, block_x, layout).to(ttgl.float16))
            mbarrier.arrive(bar.index(1), count=1)
```
- **EN:** Invokes `mbarrier.wait`, `mbarrier.arrive`, `bar.index`, `smem.index`, `ttgl.arange` to execute the test logic. Iterates across cases or data tiles. Relevant themes: layout transformation reasoning.
- **CN:** 调用 `mbarrier.wait`、`mbarrier.arrive`、`bar.index`、`smem.index`、`ttgl.arange` 执行测试逻辑。 通过循环覆盖多个用例或数据分块。 相关主题：布局变换推理。

#### Lines 2001-2003

```python

    @gluon.jit
    def ws_1(smem, bar, FAILURE: ttgl.constexpr, layout: ttgl.constexpr):
```
- **EN:** Defines the helper function `ws_1`. Decorators: `gluon.jit`. Parameters: `smem`, `bar`, `FAILURE`, `layout`. Key calls include `ttgl.zeros`, `ttgl.num_ctas`, `mbarrier.wait`, `mbarrier.arrive`, `bar.index`, `smem.index`. This scope touches layout transformation reasoning.
- **CN:** 定义辅助函数 `ws_1`。 装饰器：`gluon.jit`。 参数：`smem`、`bar`、`FAILURE`、`layout`。 关键调用包括 `ttgl.zeros`、`ttgl.num_ctas`、`mbarrier.wait`、`mbarrier.arrive`、`bar.index`、`smem.index`。 该作用域涉及布局变换推理。

##### Lines 2004-2006

```python
        block_x: ttgl.constexpr = XBLOCK * ttgl.num_ctas()
        acc = ttgl.zeros([block_x], ttgl.float16, layout)
        phase = 0
```
- **EN:** Prepares or updates state through `block_x`, `acc`, `phase`. Invokes `ttgl.num_ctas`, `ttgl.zeros` to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `block_x`、`acc`、`phase` 准备或更新状态。 调用 `ttgl.num_ctas`、`ttgl.zeros` 执行测试逻辑。 相关主题：布局变换推理。

##### Lines 2007-2013

```python
        for _ in range(10):
            mbarrier.wait(bar.index(0), phase=phase)
            val = smem.index(1 if FAILURE else 0).load(layout)
            mbarrier.wait(bar.index(1), phase=phase)
            phase = (phase + 1) % 2
            mbarrier.arrive(bar.index(2), count=1)
            acc = acc + val
```
- **EN:** Invokes `mbarrier.wait`, `mbarrier.arrive`, `bar.index`, `smem.index` to execute the test logic. Iterates across cases or data tiles. Relevant themes: layout transformation reasoning.
- **CN:** 调用 `mbarrier.wait`、`mbarrier.arrive`、`bar.index`、`smem.index` 执行测试逻辑。 通过循环覆盖多个用例或数据分块。 相关主题：布局变换推理。

##### Lines 2014-2014

```python
        smem.index(2).store(acc)  # dummy store to make sure the load is executed
```
- **EN:** Invokes `smem.index` to execute the test logic.
- **CN:** 调用 `smem.index` 执行测试逻辑。

#### Lines 2015-2017

```python

    @gluon.jit
    def kernel(output, FAILURE: ttgl.constexpr):
```
- **EN:** Defines the helper function `kernel`. Decorators: `gluon.jit`. Parameters: `output`, `FAILURE`. Key calls include `default_cga_layout`, `ttgl.SwizzledSharedLayout`, `ttgl.BlockedLayout`, `ttgl.allocate_shared_memory`, `mbarrier.allocate_mbarrier`, `mbarrier.arrive`, and 4 more. This scope touches layout transformation reasoning.
- **CN:** 定义辅助函数 `kernel`。 装饰器：`gluon.jit`。 参数：`output`、`FAILURE`。 关键调用包括 `default_cga_layout`、`ttgl.SwizzledSharedLayout`、`ttgl.BlockedLayout`、`ttgl.allocate_shared_memory`、`mbarrier.allocate_mbarrier`、`mbarrier.arrive` 等另外 4 项。 该作用域涉及布局变换推理。

##### Lines 2018-2025

```python
        block_x: ttgl.constexpr = XBLOCK * ttgl.num_ctas()
        cga_layout: ttgl.constexpr = default_cga_layout(ttgl.num_ctas(), 1)
        smem_layout: ttgl.constexpr = ttgl.SwizzledSharedLayout(vec=1, per_phase=1, max_phase=1, order=[0],
                                                                cga_layout=cga_layout)
        blocked_layout: ttgl.constexpr = ttgl.BlockedLayout(size_per_thread=[1], threads_per_warp=[32],
                                                            warps_per_cta=[4], order=[0], cga_layout=cga_layout)
        smem = ttgl.allocate_shared_memory(ttgl.float16, [3, block_x], smem_layout)
        bar = mbarrier.allocate_mbarrier(batch=3)
```
- **EN:** Prepares or updates state through `block_x`, `cga_layout`, `smem_layout`, `blocked_layout`, `smem`, `bar`. Invokes `ttgl.num_ctas`, `default_cga_layout`, `ttgl.SwizzledSharedLayout`, `ttgl.BlockedLayout`, `ttgl.allocate_shared_memory`, `mbarrier.allocate_mbarrier` to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `block_x`、`cga_layout`、`smem_layout`、`blocked_layout`、`smem`、`bar` 准备或更新状态。 调用 `ttgl.num_ctas`、`default_cga_layout`、`ttgl.SwizzledSharedLayout`、`ttgl.BlockedLayout`、`ttgl.allocate_shared_memory`、`mbarrier.allocate_mbarrier` 执行测试逻辑。 相关主题：布局变换推理。

##### Lines 2026-2027

```python
        for i in range(3):
            mbarrier.init(bar.index(i), count=1)
```
- **EN:** Invokes `mbarrier.init`, `bar.index` to execute the test logic. Iterates across cases or data tiles.
- **CN:** 调用 `mbarrier.init`、`bar.index` 执行测试逻辑。 通过循环覆盖多个用例或数据分块。

##### Lines 2028-2034

```python

        mbarrier.arrive(bar.index(2), count=1)

        ttgl.warp_specialize([
            (ws_default, (smem, bar, FAILURE, blocked_layout)),
            (ws_1, (smem, bar, FAILURE, blocked_layout)),
        ], [4], [32])
```
- **EN:** Invokes `mbarrier.arrive`, `bar.index`, `ttgl.warp_specialize` to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 调用 `mbarrier.arrive`、`bar.index`、`ttgl.warp_specialize` 执行测试逻辑。 相关主题：布局变换推理。

#### Lines 2035-2037

```python

    output = torch.empty((XBLOCK.value * num_ctas, ), device=device, dtype=torch.float16)
    kernel[(1, )](output, FAILURE=FAILURE, num_warps=4, num_ctas=num_ctas)
```
- **EN:** Prepares or updates state through `output`. Invokes `torch.empty` to execute the test logic. Relevant themes: PyTorch tensor setup and checks.
- **CN:** 通过 `output` 准备或更新状态。 调用 `torch.empty` 执行测试逻辑。 相关主题：PyTorch 张量准备与校验。

### Lines 2038-2042

```python


@pytest.mark.skipif(not is_cuda() or torch.cuda.get_device_capability()[0] < 9, reason="Requires hopper")
@pytest.mark.parametrize("MISSING_BAR", ["none", "T2", "T3"])
def test_ws_two_producers_two_consumers(MISSING_BAR, device, run_wrapper, monkeypatch, num_ctas):
```
- **EN:** Defines the test function `test_ws_two_producers_two_consumers`. Decorators: `pytest.mark.skipif(not is_cuda() or torch.cuda.get_device_capability()[0] < 9, reason='Requires hopper')`, `pytest.mark.parametrize('MISSING_BAR', ['none', 'T2', 'T3'])`. Parameters: `MISSING_BAR`, `device`, `run_wrapper`, `monkeypatch`, `num_ctas`. Nested definitions in this scope: `ws_default`, `ws_1`, `ws_2`, `ws_3`, `kernel`. Key calls include `pytest.mark.skipif`, `pytest.mark.parametrize`, `monkeypatch.setenv`, `knobs.refresh_knobs`, `torch.empty`, `run_in_process`, and 17 more. This scope touches pytest parametrization, PyTorch tensor setup and checks, runtime driver interaction, kernel launch orchestration.
- **CN:** 定义测试函数 `test_ws_two_producers_two_consumers`。 装饰器：`pytest.mark.skipif(not is_cuda() or torch.cuda.get_device_capability()[0] < 9, reason='Requires hopper')`、`pytest.mark.parametrize('MISSING_BAR', ['none', 'T2', 'T3'])`。 参数：`MISSING_BAR`、`device`、`run_wrapper`、`monkeypatch`、`num_ctas`。 该作用域中的嵌套定义：`ws_default`、`ws_1`、`ws_2`、`ws_3`、`kernel`。 关键调用包括 `pytest.mark.skipif`、`pytest.mark.parametrize`、`monkeypatch.setenv`、`knobs.refresh_knobs`、`torch.empty`、`run_in_process` 等另外 17 项。 该作用域涉及pytest 参数化、PyTorch 张量准备与校验、运行时驱动交互、kernel 启动编排。

#### Lines 2043-2052

```python
    if run_wrapper:
        result = run_in_process(test_ws_two_producers_two_consumers,
                                (MISSING_BAR, device, False, monkeypatch, num_ctas))
        if MISSING_BAR != "none":
            assert_expected_cuda_failure(result.exc)
            assert "Buffer being accessed has outstanding" in result.driver_stderr_output
        else:
            assert result.exc is None
            assert result.driver_stderr_output == ""
        return
```
- **EN:** Invokes `run_in_process`, `assert_expected_cuda_failure` to execute the test logic. Validates behavior with 3 assertion(s). Branches on runtime or test conditions. Relevant themes: runtime driver interaction.
- **CN:** 调用 `run_in_process`、`assert_expected_cuda_failure` 执行测试逻辑。 通过 3 个断言验证行为。 根据运行时或测试条件进行分支。 相关主题：运行时驱动交互。

#### Lines 2053-2055

```python
    monkeypatch.setenv("TRITON_INSTRUMENTATION_MODE", "consan")
    monkeypatch.setenv("CUDA_LAUNCH_BLOCKING", "1")
    knobs.refresh_knobs()
```
- **EN:** Invokes `monkeypatch.setenv`, `knobs.refresh_knobs` to execute the test logic. Relevant themes: kernel launch orchestration.
- **CN:** 调用 `monkeypatch.setenv`、`knobs.refresh_knobs` 执行测试逻辑。 相关主题：kernel 启动编排。

#### Lines 2056-2058

```python

    @gluon.jit
    def ws_default(smem, bar, MISSING_BAR: ttgl.constexpr, layout: ttgl.constexpr):
```
- **EN:** Defines the helper function `ws_default`. Decorators: `gluon.jit`. Parameters: `smem`, `bar`, `MISSING_BAR`, `layout`. Key calls include `ttgl.num_ctas`, `mbarrier.wait`, `mbarrier.arrive`, `bar.index`, `smem.index`, `ttgl.arange`. This scope touches layout transformation reasoning.
- **CN:** 定义辅助函数 `ws_default`。 装饰器：`gluon.jit`。 参数：`smem`、`bar`、`MISSING_BAR`、`layout`。 关键调用包括 `ttgl.num_ctas`、`mbarrier.wait`、`mbarrier.arrive`、`bar.index`、`smem.index`、`ttgl.arange`。 该作用域涉及布局变换推理。

##### Lines 2059-2060

```python
        block_x: ttgl.constexpr = XBLOCK * ttgl.num_ctas()
        phase = 0
```
- **EN:** Prepares or updates state through `block_x`, `phase`. Invokes `ttgl.num_ctas` to execute the test logic.
- **CN:** 通过 `block_x`、`phase` 准备或更新状态。 调用 `ttgl.num_ctas` 执行测试逻辑。

##### Lines 2061-2065

```python
        for _ in range(10):
            mbarrier.wait(bar.index(2), phase=phase)
            phase = (phase + 1) % 2
            smem.index(0).store(ttgl.arange(0, block_x, layout).to(ttgl.float16))
            mbarrier.arrive(bar.index(0), count=1)
```
- **EN:** Invokes `mbarrier.wait`, `mbarrier.arrive`, `bar.index`, `smem.index`, `ttgl.arange` to execute the test logic. Iterates across cases or data tiles. Relevant themes: layout transformation reasoning.
- **CN:** 调用 `mbarrier.wait`、`mbarrier.arrive`、`bar.index`、`smem.index`、`ttgl.arange` 执行测试逻辑。 通过循环覆盖多个用例或数据分块。 相关主题：布局变换推理。

#### Lines 2066-2068

```python

    @gluon.jit
    def ws_1(smem, bar, MISSING_BAR: ttgl.constexpr, layout: ttgl.constexpr):
```
- **EN:** Defines the helper function `ws_1`. Decorators: `gluon.jit`. Parameters: `smem`, `bar`, `MISSING_BAR`, `layout`. Key calls include `ttgl.num_ctas`, `mbarrier.wait`, `mbarrier.arrive`, `bar.index`, `smem.index`, `ttgl.arange`. This scope touches layout transformation reasoning.
- **CN:** 定义辅助函数 `ws_1`。 装饰器：`gluon.jit`。 参数：`smem`、`bar`、`MISSING_BAR`、`layout`。 关键调用包括 `ttgl.num_ctas`、`mbarrier.wait`、`mbarrier.arrive`、`bar.index`、`smem.index`、`ttgl.arange`。 该作用域涉及布局变换推理。

##### Lines 2069-2070

```python
        block_x: ttgl.constexpr = XBLOCK * ttgl.num_ctas()
        phase = 0
```
- **EN:** Prepares or updates state through `block_x`, `phase`. Invokes `ttgl.num_ctas` to execute the test logic.
- **CN:** 通过 `block_x`、`phase` 准备或更新状态。 调用 `ttgl.num_ctas` 执行测试逻辑。

##### Lines 2071-2075

```python
        for _ in range(10):
            mbarrier.wait(bar.index(3), phase=phase)
            phase = (phase + 1) % 2
            smem.index(1).store(ttgl.arange(0, block_x, layout).to(ttgl.float16))
            mbarrier.arrive(bar.index(0), count=1)
```
- **EN:** Invokes `mbarrier.wait`, `mbarrier.arrive`, `bar.index`, `smem.index`, `ttgl.arange` to execute the test logic. Iterates across cases or data tiles. Relevant themes: layout transformation reasoning.
- **CN:** 调用 `mbarrier.wait`、`mbarrier.arrive`、`bar.index`、`smem.index`、`ttgl.arange` 执行测试逻辑。 通过循环覆盖多个用例或数据分块。 相关主题：布局变换推理。

#### Lines 2076-2078

```python

    @gluon.jit
    def ws_2(smem, bar, MISSING_BAR: ttgl.constexpr, layout: ttgl.constexpr):
```
- **EN:** Defines the helper function `ws_2`. Decorators: `gluon.jit`. Parameters: `smem`, `bar`, `MISSING_BAR`, `layout`. Key calls include `ttgl.zeros`, `ttgl.num_ctas`, `mbarrier.arrive`, `mbarrier.wait`, `bar.index`, `smem.index`. This scope touches layout transformation reasoning.
- **CN:** 定义辅助函数 `ws_2`。 装饰器：`gluon.jit`。 参数：`smem`、`bar`、`MISSING_BAR`、`layout`。 关键调用包括 `ttgl.zeros`、`ttgl.num_ctas`、`mbarrier.arrive`、`mbarrier.wait`、`bar.index`、`smem.index`。 该作用域涉及布局变换推理。

##### Lines 2079-2081

```python
        block_x: ttgl.constexpr = XBLOCK * ttgl.num_ctas()
        acc = ttgl.zeros([block_x], ttgl.float16, layout)
        phase = 0
```
- **EN:** Prepares or updates state through `block_x`, `acc`, `phase`. Invokes `ttgl.num_ctas`, `ttgl.zeros` to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `block_x`、`acc`、`phase` 准备或更新状态。 调用 `ttgl.num_ctas`、`ttgl.zeros` 执行测试逻辑。 相关主题：布局变换推理。

##### Lines 2082-2089

```python
        for _ in range(10):
            if MISSING_BAR != "T2":
                mbarrier.wait(bar.index(0), phase=phase)
            phase = (phase + 1) % 2
            val = smem.index(0).load(layout)
            mbarrier.arrive(bar.index(2), count=1)
            mbarrier.arrive(bar.index(3), count=1)
            acc = acc + val
```
- **EN:** Invokes `mbarrier.arrive`, `mbarrier.wait`, `bar.index`, `smem.index` to execute the test logic. Iterates across cases or data tiles. Relevant themes: layout transformation reasoning.
- **CN:** 调用 `mbarrier.arrive`、`mbarrier.wait`、`bar.index`、`smem.index` 执行测试逻辑。 通过循环覆盖多个用例或数据分块。 相关主题：布局变换推理。

##### Lines 2090-2090

```python
        smem.index(2).store(acc)  # dummy store to make sure the load is executed
```
- **EN:** Invokes `smem.index` to execute the test logic.
- **CN:** 调用 `smem.index` 执行测试逻辑。

#### Lines 2091-2093

```python

    @gluon.jit
    def ws_3(smem, bar, MISSING_BAR: ttgl.constexpr, layout: ttgl.constexpr):
```
- **EN:** Defines the helper function `ws_3`. Decorators: `gluon.jit`. Parameters: `smem`, `bar`, `MISSING_BAR`, `layout`. Key calls include `ttgl.zeros`, `ttgl.num_ctas`, `mbarrier.arrive`, `mbarrier.wait`, `bar.index`, `smem.index`. This scope touches layout transformation reasoning.
- **CN:** 定义辅助函数 `ws_3`。 装饰器：`gluon.jit`。 参数：`smem`、`bar`、`MISSING_BAR`、`layout`。 关键调用包括 `ttgl.zeros`、`ttgl.num_ctas`、`mbarrier.arrive`、`mbarrier.wait`、`bar.index`、`smem.index`。 该作用域涉及布局变换推理。

##### Lines 2094-2096

```python
        block_x: ttgl.constexpr = XBLOCK * ttgl.num_ctas()
        acc = ttgl.zeros([block_x], ttgl.float16, layout)
        phase = 0
```
- **EN:** Prepares or updates state through `block_x`, `acc`, `phase`. Invokes `ttgl.num_ctas`, `ttgl.zeros` to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `block_x`、`acc`、`phase` 准备或更新状态。 调用 `ttgl.num_ctas`、`ttgl.zeros` 执行测试逻辑。 相关主题：布局变换推理。

##### Lines 2097-2104

```python
        for _ in range(10):
            if MISSING_BAR != "T3":
                mbarrier.wait(bar.index(0), phase=phase)
            phase = (phase + 1) % 2
            val = smem.index(1).load(layout)
            mbarrier.arrive(bar.index(2), count=1)
            mbarrier.arrive(bar.index(3), count=1)
            acc = acc + val
```
- **EN:** Invokes `mbarrier.arrive`, `mbarrier.wait`, `bar.index`, `smem.index` to execute the test logic. Iterates across cases or data tiles. Relevant themes: layout transformation reasoning.
- **CN:** 调用 `mbarrier.arrive`、`mbarrier.wait`、`bar.index`、`smem.index` 执行测试逻辑。 通过循环覆盖多个用例或数据分块。 相关主题：布局变换推理。

##### Lines 2105-2105

```python
        smem.index(3).store(acc)  # dummy store to make sure the load is executed
```
- **EN:** Invokes `smem.index` to execute the test logic.
- **CN:** 调用 `smem.index` 执行测试逻辑。

#### Lines 2106-2108

```python

    @gluon.jit
    def kernel(output, MISSING_BAR: ttgl.constexpr):
```
- **EN:** Defines the helper function `kernel`. Decorators: `gluon.jit`. Parameters: `output`, `MISSING_BAR`. Key calls include `default_cga_layout`, `ttgl.SwizzledSharedLayout`, `ttgl.BlockedLayout`, `ttgl.allocate_shared_memory`, `mbarrier.allocate_mbarrier`, `mbarrier.arrive`, and 4 more. This scope touches layout transformation reasoning.
- **CN:** 定义辅助函数 `kernel`。 装饰器：`gluon.jit`。 参数：`output`、`MISSING_BAR`。 关键调用包括 `default_cga_layout`、`ttgl.SwizzledSharedLayout`、`ttgl.BlockedLayout`、`ttgl.allocate_shared_memory`、`mbarrier.allocate_mbarrier`、`mbarrier.arrive` 等另外 4 项。 该作用域涉及布局变换推理。

##### Lines 2109-2116

```python
        block_x: ttgl.constexpr = XBLOCK * ttgl.num_ctas()
        cga_layout: ttgl.constexpr = default_cga_layout(ttgl.num_ctas(), 1)
        smem_layout: ttgl.constexpr = ttgl.SwizzledSharedLayout(vec=1, per_phase=1, max_phase=1, order=[0],
                                                                cga_layout=cga_layout)
        blocked_layout: ttgl.constexpr = ttgl.BlockedLayout(size_per_thread=[1], threads_per_warp=[32],
                                                            warps_per_cta=[4], order=[0], cga_layout=cga_layout)
        smem = ttgl.allocate_shared_memory(ttgl.float16, [4, block_x], smem_layout)
        bar = mbarrier.allocate_mbarrier(batch=4)
```
- **EN:** Prepares or updates state through `block_x`, `cga_layout`, `smem_layout`, `blocked_layout`, `smem`, `bar`. Invokes `ttgl.num_ctas`, `default_cga_layout`, `ttgl.SwizzledSharedLayout`, `ttgl.BlockedLayout`, `ttgl.allocate_shared_memory`, `mbarrier.allocate_mbarrier` to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `block_x`、`cga_layout`、`smem_layout`、`blocked_layout`、`smem`、`bar` 准备或更新状态。 调用 `ttgl.num_ctas`、`default_cga_layout`、`ttgl.SwizzledSharedLayout`、`ttgl.BlockedLayout`、`ttgl.allocate_shared_memory`、`mbarrier.allocate_mbarrier` 执行测试逻辑。 相关主题：布局变换推理。

##### Lines 2117-2118

```python
        for i in range(4):
            mbarrier.init(bar.index(i), count=2)
```
- **EN:** Invokes `mbarrier.init`, `bar.index` to execute the test logic. Iterates across cases or data tiles.
- **CN:** 调用 `mbarrier.init`、`bar.index` 执行测试逻辑。 通过循环覆盖多个用例或数据分块。

##### Lines 2119-2128

```python

        mbarrier.arrive(bar.index(2), count=2)
        mbarrier.arrive(bar.index(3), count=2)

        ttgl.warp_specialize([
            (ws_default, (smem, bar, MISSING_BAR, blocked_layout)),
            (ws_1, (smem, bar, MISSING_BAR, blocked_layout)),
            (ws_2, (smem, bar, MISSING_BAR, blocked_layout)),
            (ws_3, (smem, bar, MISSING_BAR, blocked_layout)),
        ], [4, 4, 4], [32, 32, 32])
```
- **EN:** Invokes `mbarrier.arrive`, `bar.index`, `ttgl.warp_specialize` to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 调用 `mbarrier.arrive`、`bar.index`、`ttgl.warp_specialize` 执行测试逻辑。 相关主题：布局变换推理。

#### Lines 2129-2131

```python

    output = torch.empty((XBLOCK.value * num_ctas, ), device=device, dtype=torch.float16)
    kernel[(1, )](output, MISSING_BAR=MISSING_BAR, num_warps=4, num_ctas=num_ctas)
```
- **EN:** Prepares or updates state through `output`. Invokes `torch.empty` to execute the test logic. Relevant themes: PyTorch tensor setup and checks.
- **CN:** 通过 `output` 准备或更新状态。 调用 `torch.empty` 执行测试逻辑。 相关主题：PyTorch 张量准备与校验。

### Lines 2132-2136

```python


@pytest.mark.skipif(not is_cuda() or torch.cuda.get_device_capability()[0] < 9, reason="Requires hopper")
@pytest.mark.parametrize("MISSING_BAR", ["none", "1", "2"])
def test_ws_different_warp_sizes(MISSING_BAR, device, run_wrapper, monkeypatch, num_ctas):
```
- **EN:** Defines the test function `test_ws_different_warp_sizes`. Decorators: `pytest.mark.skipif(not is_cuda() or torch.cuda.get_device_capability()[0] < 9, reason='Requires hopper')`, `pytest.mark.parametrize('MISSING_BAR', ['none', '1', '2'])`. Parameters: `MISSING_BAR`, `device`, `run_wrapper`, `monkeypatch`, `num_ctas`. Nested definitions in this scope: `ws_default`, `ws_1`, `ws_2`, `kernel`. Key calls include `pytest.mark.skipif`, `pytest.mark.parametrize`, `monkeypatch.setenv`, `knobs.refresh_knobs`, `torch.empty`, `run_in_process`, and 17 more. This scope touches pytest parametrization, PyTorch tensor setup and checks, runtime driver interaction, kernel launch orchestration.
- **CN:** 定义测试函数 `test_ws_different_warp_sizes`。 装饰器：`pytest.mark.skipif(not is_cuda() or torch.cuda.get_device_capability()[0] < 9, reason='Requires hopper')`、`pytest.mark.parametrize('MISSING_BAR', ['none', '1', '2'])`。 参数：`MISSING_BAR`、`device`、`run_wrapper`、`monkeypatch`、`num_ctas`。 该作用域中的嵌套定义：`ws_default`、`ws_1`、`ws_2`、`kernel`。 关键调用包括 `pytest.mark.skipif`、`pytest.mark.parametrize`、`monkeypatch.setenv`、`knobs.refresh_knobs`、`torch.empty`、`run_in_process` 等另外 17 项。 该作用域涉及pytest 参数化、PyTorch 张量准备与校验、运行时驱动交互、kernel 启动编排。

#### Lines 2137-2145

```python
    if run_wrapper:
        result = run_in_process(test_ws_different_warp_sizes, (MISSING_BAR, device, False, monkeypatch, num_ctas))
        if MISSING_BAR != "none":
            assert_expected_cuda_failure(result.exc)
            assert "Buffer being accessed has outstanding" in result.driver_stderr_output
        else:
            assert result.exc is None
            assert result.driver_stderr_output == ""
        return
```
- **EN:** Invokes `run_in_process`, `assert_expected_cuda_failure` to execute the test logic. Validates behavior with 3 assertion(s). Branches on runtime or test conditions. Relevant themes: runtime driver interaction.
- **CN:** 调用 `run_in_process`、`assert_expected_cuda_failure` 执行测试逻辑。 通过 3 个断言验证行为。 根据运行时或测试条件进行分支。 相关主题：运行时驱动交互。

#### Lines 2146-2148

```python
    monkeypatch.setenv("TRITON_INSTRUMENTATION_MODE", "consan")
    monkeypatch.setenv("CUDA_LAUNCH_BLOCKING", "1")
    knobs.refresh_knobs()
```
- **EN:** Invokes `monkeypatch.setenv`, `knobs.refresh_knobs` to execute the test logic. Relevant themes: kernel launch orchestration.
- **CN:** 调用 `monkeypatch.setenv`、`knobs.refresh_knobs` 执行测试逻辑。 相关主题：kernel 启动编排。

#### Lines 2149-2151

```python

    @gluon.jit
    def ws_default(smem, bar, MISSING_BAR: ttgl.constexpr):
```
- **EN:** Defines the helper function `ws_default`. Decorators: `gluon.jit`. Parameters: `smem`, `bar`, `MISSING_BAR`. Key calls include `default_cga_layout`, `ttgl.BlockedLayout`, `mbarrier.arrive`, `ttgl.num_ctas`, `bar.index`, `smem.index`. This scope touches layout transformation reasoning.
- **CN:** 定义辅助函数 `ws_default`。 装饰器：`gluon.jit`。 参数：`smem`、`bar`、`MISSING_BAR`。 关键调用包括 `default_cga_layout`、`ttgl.BlockedLayout`、`mbarrier.arrive`、`ttgl.num_ctas`、`bar.index`、`smem.index`。 该作用域涉及布局变换推理。

##### Lines 2152-2157

```python
        cga_layout: ttgl.constexpr = default_cga_layout(ttgl.num_ctas(), 1)
        layout: ttgl.constexpr = ttgl.BlockedLayout(size_per_thread=[1], threads_per_warp=[32], warps_per_cta=[4],
                                                    order=[0], cga_layout=cga_layout)
        val = smem.index(0).load(layout)
        mbarrier.arrive(bar.index(0), count=1)
        smem.index(1).store(val)  # dummy store to make sure the load is executed
```
- **EN:** Prepares or updates state through `cga_layout`, `layout`, `val`. Invokes `default_cga_layout`, `ttgl.num_ctas`, `ttgl.BlockedLayout`, `smem.index`, `mbarrier.arrive`, `bar.index` to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `cga_layout`、`layout`、`val` 准备或更新状态。 调用 `default_cga_layout`、`ttgl.num_ctas`、`ttgl.BlockedLayout`、`smem.index`、`mbarrier.arrive`、`bar.index` 执行测试逻辑。 相关主题：布局变换推理。

#### Lines 2158-2160

```python

    @gluon.jit
    def ws_1(smem, bar, MISSING_BAR: ttgl.constexpr):
```
- **EN:** Defines the helper function `ws_1`. Decorators: `gluon.jit`. Parameters: `smem`, `bar`, `MISSING_BAR`. Key calls include `default_cga_layout`, `ttgl.BlockedLayout`, `mbarrier.arrive`, `ttgl.num_ctas`, `bar.index`, `smem.index`. This scope touches layout transformation reasoning.
- **CN:** 定义辅助函数 `ws_1`。 装饰器：`gluon.jit`。 参数：`smem`、`bar`、`MISSING_BAR`。 关键调用包括 `default_cga_layout`、`ttgl.BlockedLayout`、`mbarrier.arrive`、`ttgl.num_ctas`、`bar.index`、`smem.index`。 该作用域涉及布局变换推理。

##### Lines 2161-2166

```python
        cga_layout: ttgl.constexpr = default_cga_layout(ttgl.num_ctas(), 1)
        layout: ttgl.constexpr = ttgl.BlockedLayout(size_per_thread=[1], threads_per_warp=[32], warps_per_cta=[2],
                                                    order=[0], cga_layout=cga_layout)
        val = smem.index(0).load(layout)
        mbarrier.arrive(bar.index(1), count=1)
        smem.index(2).store(val)  # dummy store to make sure the load is executed
```
- **EN:** Prepares or updates state through `cga_layout`, `layout`, `val`. Invokes `default_cga_layout`, `ttgl.num_ctas`, `ttgl.BlockedLayout`, `smem.index`, `mbarrier.arrive`, `bar.index` to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `cga_layout`、`layout`、`val` 准备或更新状态。 调用 `default_cga_layout`、`ttgl.num_ctas`、`ttgl.BlockedLayout`、`smem.index`、`mbarrier.arrive`、`bar.index` 执行测试逻辑。 相关主题：布局变换推理。

#### Lines 2167-2169

```python

    @gluon.jit
    def ws_2(smem, bar, MISSING_BAR: ttgl.constexpr):
```
- **EN:** Defines the helper function `ws_2`. Decorators: `gluon.jit`. Parameters: `smem`, `bar`, `MISSING_BAR`. Key calls include `default_cga_layout`, `ttgl.BlockedLayout`, `mbarrier.arrive`, `ttgl.num_ctas`, `mbarrier.wait`, `bar.index`, and 2 more. This scope touches layout transformation reasoning.
- **CN:** 定义辅助函数 `ws_2`。 装饰器：`gluon.jit`。 参数：`smem`、`bar`、`MISSING_BAR`。 关键调用包括 `default_cga_layout`、`ttgl.BlockedLayout`、`mbarrier.arrive`、`ttgl.num_ctas`、`mbarrier.wait`、`bar.index` 等另外 2 项。 该作用域涉及布局变换推理。

##### Lines 2170-2173

```python
        block_x: ttgl.constexpr = XBLOCK * ttgl.num_ctas()
        cga_layout: ttgl.constexpr = default_cga_layout(ttgl.num_ctas(), 1)
        layout: ttgl.constexpr = ttgl.BlockedLayout(size_per_thread=[1], threads_per_warp=[32], warps_per_cta=[8],
                                                    order=[0], cga_layout=cga_layout)
```
- **EN:** Prepares or updates state through `block_x`, `cga_layout`, `layout`. Invokes `ttgl.num_ctas`, `default_cga_layout`, `ttgl.BlockedLayout` to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `block_x`、`cga_layout`、`layout` 准备或更新状态。 调用 `ttgl.num_ctas`、`default_cga_layout`、`ttgl.BlockedLayout` 执行测试逻辑。 相关主题：布局变换推理。

##### Lines 2174-2175

```python
        if MISSING_BAR != "1":
            mbarrier.wait(bar.index(0), phase=0)
```
- **EN:** Invokes `mbarrier.wait`, `bar.index` to execute the test logic. Branches on runtime or test conditions.
- **CN:** 调用 `mbarrier.wait`、`bar.index` 执行测试逻辑。 根据运行时或测试条件进行分支。

##### Lines 2176-2177

```python
        if MISSING_BAR != "2":
            mbarrier.wait(bar.index(1), phase=0)
```
- **EN:** Invokes `mbarrier.wait`, `bar.index` to execute the test logic. Branches on runtime or test conditions.
- **CN:** 调用 `mbarrier.wait`、`bar.index` 执行测试逻辑。 根据运行时或测试条件进行分支。

##### Lines 2178-2179

```python
        smem.index(0).store(ttgl.arange(0, block_x, layout).to(ttgl.float16))
        mbarrier.arrive(bar.index(2), count=1)
```
- **EN:** Invokes `smem.index`, `ttgl.arange`, `mbarrier.arrive`, `bar.index` to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 调用 `smem.index`、`ttgl.arange`、`mbarrier.arrive`、`bar.index` 执行测试逻辑。 相关主题：布局变换推理。

#### Lines 2180-2182

```python

    @gluon.jit
    def kernel(output, MISSING_BAR: ttgl.constexpr):
```
- **EN:** Defines the helper function `kernel`. Decorators: `gluon.jit`. Parameters: `output`, `MISSING_BAR`. Key calls include `default_cga_layout`, `ttgl.SwizzledSharedLayout`, `ttgl.BlockedLayout`, `ttgl.allocate_shared_memory`, `mbarrier.allocate_mbarrier`, `ttgl.warp_specialize`, and 7 more. This scope touches layout transformation reasoning.
- **CN:** 定义辅助函数 `kernel`。 装饰器：`gluon.jit`。 参数：`output`、`MISSING_BAR`。 关键调用包括 `default_cga_layout`、`ttgl.SwizzledSharedLayout`、`ttgl.BlockedLayout`、`ttgl.allocate_shared_memory`、`mbarrier.allocate_mbarrier`、`ttgl.warp_specialize` 等另外 7 项。 该作用域涉及布局变换推理。

##### Lines 2183-2190

```python
        block_x: ttgl.constexpr = XBLOCK * ttgl.num_ctas()
        cga_layout: ttgl.constexpr = default_cga_layout(ttgl.num_ctas(), 1)
        smem_layout: ttgl.constexpr = ttgl.SwizzledSharedLayout(vec=1, per_phase=1, max_phase=1, order=[0],
                                                                cga_layout=cga_layout)
        blocked_layout: ttgl.constexpr = ttgl.BlockedLayout(size_per_thread=[1], threads_per_warp=[32],
                                                            warps_per_cta=[4], order=[0], cga_layout=cga_layout)
        smem = ttgl.allocate_shared_memory(ttgl.float16, [3, block_x], smem_layout)
        bar = mbarrier.allocate_mbarrier(batch=3)
```
- **EN:** Prepares or updates state through `block_x`, `cga_layout`, `smem_layout`, `blocked_layout`, `smem`, `bar`. Invokes `ttgl.num_ctas`, `default_cga_layout`, `ttgl.SwizzledSharedLayout`, `ttgl.BlockedLayout`, `ttgl.allocate_shared_memory`, `mbarrier.allocate_mbarrier` to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `block_x`、`cga_layout`、`smem_layout`、`blocked_layout`、`smem`、`bar` 准备或更新状态。 调用 `ttgl.num_ctas`、`default_cga_layout`、`ttgl.SwizzledSharedLayout`、`ttgl.BlockedLayout`、`ttgl.allocate_shared_memory`、`mbarrier.allocate_mbarrier` 执行测试逻辑。 相关主题：布局变换推理。

##### Lines 2191-2192

```python
        for i in range(3):
            mbarrier.init(bar.index(i), count=1)
```
- **EN:** Invokes `mbarrier.init`, `bar.index` to execute the test logic. Iterates across cases or data tiles.
- **CN:** 调用 `mbarrier.init`、`bar.index` 执行测试逻辑。 通过循环覆盖多个用例或数据分块。

##### Lines 2193-2201

```python
        ttgl.warp_specialize([
            (ws_default, (smem, bar, MISSING_BAR)),
            (ws_1, (smem, bar, MISSING_BAR)),
            (ws_2, (smem, bar, MISSING_BAR)),
        ], [2, 8], [32, 32])
        mbarrier.wait(bar.index(2), phase=0)
        val = smem.index(0).load(blocked_layout)
        output_ptrs = output + ttgl.arange(0, block_x, blocked_layout)
        ttgl.store(output_ptrs, val)
```
- **EN:** Prepares or updates state through `val`, `output_ptrs`. Invokes `ttgl.warp_specialize`, `mbarrier.wait`, `bar.index`, `smem.index`, `ttgl.arange`, `ttgl.store` to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `val`、`output_ptrs` 准备或更新状态。 调用 `ttgl.warp_specialize`、`mbarrier.wait`、`bar.index`、`smem.index`、`ttgl.arange`、`ttgl.store` 执行测试逻辑。 相关主题：布局变换推理。

#### Lines 2202-2204

```python

    output = torch.empty((XBLOCK.value * num_ctas, ), device=device, dtype=torch.float16)
    kernel[(1, )](output, MISSING_BAR=MISSING_BAR, num_warps=4, num_ctas=num_ctas)
```
- **EN:** Prepares or updates state through `output`. Invokes `torch.empty` to execute the test logic. Relevant themes: PyTorch tensor setup and checks.
- **CN:** 通过 `output` 准备或更新状态。 调用 `torch.empty` 执行测试逻辑。 相关主题：PyTorch 张量准备与校验。

### Lines 2205-2209

```python


@pytest.mark.skipif(not is_cuda() or torch.cuda.get_device_capability()[0] < 9, reason="Requires hopper or newer")
@pytest.mark.parametrize("FAILURE", [True, False])
def test_ws_async_copy_commits(FAILURE, device, run_wrapper, monkeypatch, num_ctas):
```
- **EN:** Defines the test function `test_ws_async_copy_commits`. Decorators: `pytest.mark.skipif(not is_cuda() or torch.cuda.get_device_capability()[0] < 9, reason='Requires hopper or newer')`, `pytest.mark.parametrize('FAILURE', [True, False])`. Parameters: `FAILURE`, `device`, `run_wrapper`, `monkeypatch`, `num_ctas`. Nested definitions in this scope: `ws_prog`, `kernel`. Key calls include `pytest.mark.skipif`, `pytest.mark.parametrize`, `monkeypatch.setenv`, `knobs.refresh_knobs`, `torch.randn`, `run_in_process`, and 16 more. This scope touches pytest parametrization, PyTorch tensor setup and checks, runtime driver interaction, kernel launch orchestration.
- **CN:** 定义测试函数 `test_ws_async_copy_commits`。 装饰器：`pytest.mark.skipif(not is_cuda() or torch.cuda.get_device_capability()[0] < 9, reason='Requires hopper or newer')`、`pytest.mark.parametrize('FAILURE', [True, False])`。 参数：`FAILURE`、`device`、`run_wrapper`、`monkeypatch`、`num_ctas`。 该作用域中的嵌套定义：`ws_prog`、`kernel`。 关键调用包括 `pytest.mark.skipif`、`pytest.mark.parametrize`、`monkeypatch.setenv`、`knobs.refresh_knobs`、`torch.randn`、`run_in_process` 等另外 16 项。 该作用域涉及pytest 参数化、PyTorch 张量准备与校验、运行时驱动交互、kernel 启动编排。

#### Lines 2210-2222

```python
    if run_wrapper:
        result = run_in_process(test_ws_async_copy_commits, (FAILURE, device, False, monkeypatch, num_ctas))
        if FAILURE:
            assert_expected_cuda_failure(result.exc)
            assert any(msg in result.driver_stderr_output for msg in [
                "Buffer being accessed has outstanding writes",
                "Buffer being accessed has outstanding reads",
                "Accessing buffer with pending access. Pending access type: async_copy_global_to_shared",
            ])
        else:
            assert result.exc is None
            assert result.driver_stderr_output == ""
        return
```
- **EN:** Invokes `run_in_process`, `assert_expected_cuda_failure`, `any` to execute the test logic. Validates behavior with 3 assertion(s). Branches on runtime or test conditions. Relevant themes: runtime driver interaction.
- **CN:** 调用 `run_in_process`、`assert_expected_cuda_failure`、`any` 执行测试逻辑。 通过 3 个断言验证行为。 根据运行时或测试条件进行分支。 相关主题：运行时驱动交互。

#### Lines 2223-2226

```python

    monkeypatch.setenv("TRITON_INSTRUMENTATION_MODE", "consan")
    monkeypatch.setenv("CUDA_LAUNCH_BLOCKING", "1")
    knobs.refresh_knobs()
```
- **EN:** Invokes `monkeypatch.setenv`, `knobs.refresh_knobs` to execute the test logic. Relevant themes: kernel launch orchestration.
- **CN:** 调用 `monkeypatch.setenv`、`knobs.refresh_knobs` 执行测试逻辑。 相关主题：kernel 启动编排。

#### Lines 2227-2230

```python

    @gluon.jit
    def ws_prog(input, smem, FAILURE: ttgl.constexpr, blocked_layout: ttgl.constexpr, BASE: ttgl.constexpr):
        # Two-buffer ping-pong within a partition: buffers BASE and BASE+1
```
- **EN:** Defines the helper function `ws_prog`. Decorators: `gluon.jit`. Parameters: `input`, `smem`, `FAILURE`, `blocked_layout`, `BASE`. Key calls include `ttgl.arange`, `ttgl.zeros`, `ampere.async_copy.async_copy_global_to_shared`, `ampere.async_copy.commit_group`, `ttgl.num_ctas`, `smem.index`, and 1 more. This scope touches layout transformation reasoning.
- **CN:** 定义辅助函数 `ws_prog`。 装饰器：`gluon.jit`。 参数：`input`、`smem`、`FAILURE`、`blocked_layout`、`BASE`。 关键调用包括 `ttgl.arange`、`ttgl.zeros`、`ampere.async_copy.async_copy_global_to_shared`、`ampere.async_copy.commit_group`、`ttgl.num_ctas`、`smem.index` 等另外 1 项。 该作用域涉及布局变换推理。

##### Lines 2231-2238

```python
        block_x: ttgl.constexpr = XBLOCK * ttgl.num_ctas()
        offs = ttgl.arange(0, block_x, layout=blocked_layout)

        acc = ttgl.zeros([block_x], ttgl.float16, blocked_layout)

        # Prime pipeline
        ampere.async_copy.async_copy_global_to_shared(smem.index(BASE + 0), input + offs)
        ampere.async_copy.commit_group()
```
- **EN:** Prepares or updates state through `block_x`, `offs`, `acc`. Invokes `ttgl.num_ctas`, `ttgl.arange`, `ttgl.zeros`, `ampere.async_copy.async_copy_global_to_shared`, `smem.index`, `ampere.async_copy.commit_group` to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `block_x`、`offs`、`acc` 准备或更新状态。 调用 `ttgl.num_ctas`、`ttgl.arange`、`ttgl.zeros`、`ampere.async_copy.async_copy_global_to_shared`、`smem.index`、`ampere.async_copy.commit_group` 执行测试逻辑。 相关主题：布局变换推理。

##### Lines 2239-2252

```python

        for i in range(1, 10):
            dst = (i % 2)
            src = ((i - 1) % 2)
            if i < 9:
                ampere.async_copy.async_copy_global_to_shared(smem.index(BASE + dst), input + offs)
                ampere.async_copy.commit_group()
                ampere.async_copy.wait_group(1)
            else:
                ampere.async_copy.wait_group(0)

            # Load from last completed buffer. In failure mode for BASE==2 (ws_1), read other partition's buffers (0/1)
            load_base = 0 if (FAILURE and BASE == 2) else BASE
            acc = acc + smem.index(load_base + src).load(blocked_layout)
```
- **EN:** Invokes `ampere.async_copy.async_copy_global_to_shared`, `ampere.async_copy.commit_group`, `ampere.async_copy.wait_group`, `smem.index` to execute the test logic. Iterates across cases or data tiles. Relevant themes: layout transformation reasoning.
- **CN:** 调用 `ampere.async_copy.async_copy_global_to_shared`、`ampere.async_copy.commit_group`、`ampere.async_copy.wait_group`、`smem.index` 执行测试逻辑。 通过循环覆盖多个用例或数据分块。 相关主题：布局变换推理。

##### Lines 2253-2253

```python
        smem.index(BASE).store(acc)
```
- **EN:** Invokes `smem.index` to execute the test logic.
- **CN:** 调用 `smem.index` 执行测试逻辑。

#### Lines 2254-2256

```python

    @gluon.jit
    def kernel(input, FAILURE: ttgl.constexpr):
```
- **EN:** Defines the helper function `kernel`. Decorators: `gluon.jit`. Parameters: `input`, `FAILURE`. Key calls include `default_cga_layout`, `ttgl.SwizzledSharedLayout`, `ttgl.allocate_shared_memory`, `ttgl.BlockedLayout`, `ttgl.warp_specialize`, `ttgl.num_ctas`. This scope touches layout transformation reasoning.
- **CN:** 定义辅助函数 `kernel`。 装饰器：`gluon.jit`。 参数：`input`、`FAILURE`。 关键调用包括 `default_cga_layout`、`ttgl.SwizzledSharedLayout`、`ttgl.allocate_shared_memory`、`ttgl.BlockedLayout`、`ttgl.warp_specialize`、`ttgl.num_ctas`。 该作用域涉及布局变换推理。

##### Lines 2257-2268

```python
        block_x: ttgl.constexpr = XBLOCK * ttgl.num_ctas()
        cga_layout: ttgl.constexpr = default_cga_layout(ttgl.num_ctas(), 1)
        smem_layout: ttgl.constexpr = ttgl.SwizzledSharedLayout(vec=1, per_phase=1, max_phase=1, order=[0],
                                                                cga_layout=cga_layout)
        # 4 buffers total: ws_default uses 0/1; ws_1 uses 2/3
        smem = ttgl.allocate_shared_memory(ttgl.float16, [4, block_x], smem_layout)
        blocked_layout: ttgl.constexpr = ttgl.BlockedLayout(size_per_thread=[block_x], threads_per_warp=[32],
                                                            warps_per_cta=[4], order=[0], cga_layout=cga_layout)
        ttgl.warp_specialize([
            (ws_prog, (input, smem, FAILURE, blocked_layout, 0)),
            (ws_prog, (input, smem, FAILURE, blocked_layout, 2)),
        ], [4], [32])
```
- **EN:** Prepares or updates state through `block_x`, `cga_layout`, `smem_layout`, `smem`, `blocked_layout`. Invokes `ttgl.num_ctas`, `default_cga_layout`, `ttgl.SwizzledSharedLayout`, `ttgl.allocate_shared_memory`, `ttgl.BlockedLayout`, `ttgl.warp_specialize` to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `block_x`、`cga_layout`、`smem_layout`、`smem`、`blocked_layout` 准备或更新状态。 调用 `ttgl.num_ctas`、`default_cga_layout`、`ttgl.SwizzledSharedLayout`、`ttgl.allocate_shared_memory`、`ttgl.BlockedLayout`、`ttgl.warp_specialize` 执行测试逻辑。 相关主题：布局变换推理。

#### Lines 2269-2271

```python

    input = torch.randn((XBLOCK.value * num_ctas, ), device=device, dtype=torch.float16)
    kernel[(1, )](input, FAILURE=FAILURE, num_warps=4, num_ctas=num_ctas)
```
- **EN:** Prepares or updates state through `input`. Invokes `torch.randn` to execute the test logic. Relevant themes: PyTorch tensor setup and checks, random-data generation.
- **CN:** 通过 `input` 准备或更新状态。 调用 `torch.randn` 执行测试逻辑。 相关主题：PyTorch 张量准备与校验、随机数据生成。

### Lines 2272-2276

```python


@pytest.mark.skipif(not is_cuda() or torch.cuda.get_device_capability()[0] < 9, reason="Requires hopper or newer")
@pytest.mark.parametrize("FAILURE", [True, False])
def test_ws_async_copy_wait_visibility(FAILURE, device, run_wrapper, monkeypatch, num_ctas):
```
- **EN:** Defines the test function `test_ws_async_copy_wait_visibility`. Decorators: `pytest.mark.skipif(not is_cuda() or torch.cuda.get_device_capability()[0] < 9, reason='Requires hopper or newer')`, `pytest.mark.parametrize('FAILURE', [True, False])`. Parameters: `FAILURE`, `device`, `run_wrapper`, `monkeypatch`, `num_ctas`. Nested definitions in this scope: `ws_default`, `ws_1`, `kernel`. Key calls include `pytest.mark.skipif`, `pytest.mark.parametrize`, `monkeypatch.setenv`, `knobs.refresh_knobs`, `torch.randn`, `run_in_process`, and 19 more. This scope touches pytest parametrization, PyTorch tensor setup and checks, runtime driver interaction, kernel launch orchestration.
- **CN:** 定义测试函数 `test_ws_async_copy_wait_visibility`。 装饰器：`pytest.mark.skipif(not is_cuda() or torch.cuda.get_device_capability()[0] < 9, reason='Requires hopper or newer')`、`pytest.mark.parametrize('FAILURE', [True, False])`。 参数：`FAILURE`、`device`、`run_wrapper`、`monkeypatch`、`num_ctas`。 该作用域中的嵌套定义：`ws_default`、`ws_1`、`kernel`。 关键调用包括 `pytest.mark.skipif`、`pytest.mark.parametrize`、`monkeypatch.setenv`、`knobs.refresh_knobs`、`torch.randn`、`run_in_process` 等另外 19 项。 该作用域涉及pytest 参数化、PyTorch 张量准备与校验、运行时驱动交互、kernel 启动编排。

#### Lines 2277-2287

```python
    if run_wrapper:
        result = run_in_process(test_ws_async_copy_wait_visibility, (FAILURE, device, False, monkeypatch, num_ctas))
        if FAILURE:
            assert_expected_cuda_failure(result.exc)
            assert (("Buffer being accessed has outstanding writes" in result.driver_stderr_output)
                    or ("Accessing buffer with pending access. Pending access type: async_copy_global_to_shared"
                        in result.driver_stderr_output))
        else:
            assert result.exc is None
            assert result.driver_stderr_output == ""
        return
```
- **EN:** Invokes `run_in_process`, `assert_expected_cuda_failure` to execute the test logic. Validates behavior with 3 assertion(s). Branches on runtime or test conditions. Relevant themes: runtime driver interaction.
- **CN:** 调用 `run_in_process`、`assert_expected_cuda_failure` 执行测试逻辑。 通过 3 个断言验证行为。 根据运行时或测试条件进行分支。 相关主题：运行时驱动交互。

#### Lines 2288-2291

```python

    monkeypatch.setenv("TRITON_INSTRUMENTATION_MODE", "consan")
    monkeypatch.setenv("CUDA_LAUNCH_BLOCKING", "1")
    knobs.refresh_knobs()
```
- **EN:** Invokes `monkeypatch.setenv`, `knobs.refresh_knobs` to execute the test logic. Relevant themes: kernel launch orchestration.
- **CN:** 调用 `monkeypatch.setenv`、`knobs.refresh_knobs` 执行测试逻辑。 相关主题：kernel 启动编排。

#### Lines 2292-2294

```python

    @gluon.jit
    def ws_default(input, smem, bar, FAILURE: ttgl.constexpr, layout: ttgl.constexpr):
```
- **EN:** Defines the helper function `ws_default`. Decorators: `gluon.jit`. Parameters: `input`, `smem`, `bar`, `FAILURE`, `layout`. Key calls include `ttgl.arange`, `ampere.async_copy.async_copy_global_to_shared`, `ampere.async_copy.commit_group`, `ampere.async_copy.wait_group`, `mbarrier.arrive`, `ttgl.num_ctas`, and 2 more. This scope touches layout transformation reasoning.
- **CN:** 定义辅助函数 `ws_default`。 装饰器：`gluon.jit`。 参数：`input`、`smem`、`bar`、`FAILURE`、`layout`。 关键调用包括 `ttgl.arange`、`ampere.async_copy.async_copy_global_to_shared`、`ampere.async_copy.commit_group`、`ampere.async_copy.wait_group`、`mbarrier.arrive`、`ttgl.num_ctas` 等另外 2 项。 该作用域涉及布局变换推理。

##### Lines 2295-2302

```python
        block_x: ttgl.constexpr = XBLOCK * ttgl.num_ctas()
        offs = ttgl.arange(0, block_x, layout)
        ampere.async_copy.async_copy_global_to_shared(smem.index(0), input + offs)
        ampere.async_copy.commit_group()
        ampere.async_copy.async_copy_global_to_shared(smem.index(1), input + offs)
        ampere.async_copy.commit_group()
        ampere.async_copy.wait_group(1)
        mbarrier.arrive(bar.index(0), count=1)
```
- **EN:** Prepares or updates state through `block_x`, `offs`. Invokes `ttgl.num_ctas`, `ttgl.arange`, `ampere.async_copy.async_copy_global_to_shared`, `smem.index`, `ampere.async_copy.commit_group`, `ampere.async_copy.wait_group`, and 2 more to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `block_x`、`offs` 准备或更新状态。 调用 `ttgl.num_ctas`、`ttgl.arange`、`ampere.async_copy.async_copy_global_to_shared`、`smem.index`、`ampere.async_copy.commit_group`、`ampere.async_copy.wait_group` 等另外 2 项 执行测试逻辑。 相关主题：布局变换推理。

#### Lines 2303-2305

```python

    @gluon.jit
    def ws_1(input, smem, bar, FAILURE: ttgl.constexpr, layout: ttgl.constexpr):
```
- **EN:** Defines the helper function `ws_1`. Decorators: `gluon.jit`. Parameters: `input`, `smem`, `bar`, `FAILURE`, `layout`. Key calls include `mbarrier.wait`, `bar.index`, `smem.index`. This scope touches layout transformation reasoning.
- **CN:** 定义辅助函数 `ws_1`。 装饰器：`gluon.jit`。 参数：`input`、`smem`、`bar`、`FAILURE`、`layout`。 关键调用包括 `mbarrier.wait`、`bar.index`、`smem.index`。 该作用域涉及布局变换推理。

##### Lines 2306-2308

```python
        mbarrier.wait(bar.index(0), phase=0)
        val = smem.index(1 if FAILURE else 0).load(layout)
        smem.index(0).store(val)  # keep load
```
- **EN:** Prepares or updates state through `val`. Invokes `mbarrier.wait`, `bar.index`, `smem.index` to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `val` 准备或更新状态。 调用 `mbarrier.wait`、`bar.index`、`smem.index` 执行测试逻辑。 相关主题：布局变换推理。

#### Lines 2309-2311

```python

    @gluon.jit
    def kernel(input, FAILURE: ttgl.constexpr):
```
- **EN:** Defines the helper function `kernel`. Decorators: `gluon.jit`. Parameters: `input`, `FAILURE`. Key calls include `default_cga_layout`, `ttgl.SwizzledSharedLayout`, `ttgl.BlockedLayout`, `ttgl.allocate_shared_memory`, `mbarrier.allocate_mbarrier`, `mbarrier.init`, and 3 more. This scope touches layout transformation reasoning.
- **CN:** 定义辅助函数 `kernel`。 装饰器：`gluon.jit`。 参数：`input`、`FAILURE`。 关键调用包括 `default_cga_layout`、`ttgl.SwizzledSharedLayout`、`ttgl.BlockedLayout`、`ttgl.allocate_shared_memory`、`mbarrier.allocate_mbarrier`、`mbarrier.init` 等另外 3 项。 该作用域涉及布局变换推理。

##### Lines 2312-2324

```python
        block_x: ttgl.constexpr = XBLOCK * ttgl.num_ctas()
        cga_layout: ttgl.constexpr = default_cga_layout(ttgl.num_ctas(), 1)
        smem_layout: ttgl.constexpr = ttgl.SwizzledSharedLayout(vec=1, per_phase=1, max_phase=1, order=[0],
                                                                cga_layout=cga_layout)
        blocked_layout: ttgl.constexpr = ttgl.BlockedLayout(size_per_thread=[block_x], threads_per_warp=[32],
                                                            warps_per_cta=[4], order=[0], cga_layout=cga_layout)
        smem = ttgl.allocate_shared_memory(ttgl.float16, [2, block_x], smem_layout)
        bar = mbarrier.allocate_mbarrier(batch=1)
        mbarrier.init(bar.index(0), count=1)
        ttgl.warp_specialize([
            (ws_default, (input, smem, bar, FAILURE, blocked_layout)),
            (ws_1, (input, smem, bar, FAILURE, blocked_layout)),
        ], [4], [32])
```
- **EN:** Prepares or updates state through `block_x`, `cga_layout`, `smem_layout`, `blocked_layout`, `smem`, `bar`. Invokes `ttgl.num_ctas`, `default_cga_layout`, `ttgl.SwizzledSharedLayout`, `ttgl.BlockedLayout`, `ttgl.allocate_shared_memory`, `mbarrier.allocate_mbarrier`, and 3 more to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `block_x`、`cga_layout`、`smem_layout`、`blocked_layout`、`smem`、`bar` 准备或更新状态。 调用 `ttgl.num_ctas`、`default_cga_layout`、`ttgl.SwizzledSharedLayout`、`ttgl.BlockedLayout`、`ttgl.allocate_shared_memory`、`mbarrier.allocate_mbarrier` 等另外 3 项 执行测试逻辑。 相关主题：布局变换推理。

#### Lines 2325-2327

```python

    input = torch.randn((XBLOCK.value * num_ctas, ), device=device, dtype=torch.float16)
    kernel[(1, )](input, FAILURE=FAILURE, num_warps=4, num_ctas=num_ctas)
```
- **EN:** Prepares or updates state through `input`. Invokes `torch.randn` to execute the test logic. Relevant themes: PyTorch tensor setup and checks, random-data generation.
- **CN:** 通过 `input` 准备或更新状态。 调用 `torch.randn` 执行测试逻辑。 相关主题：PyTorch 张量准备与校验、随机数据生成。

### Lines 2328-2332

```python


@pytest.mark.skipif(not is_cuda() or torch.cuda.get_device_capability()[0] != 9, reason="Requires hopper")
@pytest.mark.parametrize("FAILURE", [True, False])
def test_ws_wgmma_wait_visibility(FAILURE, device, run_wrapper, monkeypatch, num_ctas):
```
- **EN:** Defines the test function `test_ws_wgmma_wait_visibility`. Decorators: `pytest.mark.skipif(not is_cuda() or torch.cuda.get_device_capability()[0] != 9, reason='Requires hopper')`, `pytest.mark.parametrize('FAILURE', [True, False])`. Parameters: `FAILURE`, `device`, `run_wrapper`, `monkeypatch`, `num_ctas`. Nested definitions in this scope: `ws_default`, `ws_1`, `kernel`. Key calls include `pytest.mark.skipif`, `pytest.mark.parametrize`, `monkeypatch.setenv`, `knobs.refresh_knobs`, `run_in_process`, `mma_block_m`, and 22 more. This scope touches pytest parametrization, PyTorch tensor setup and checks, runtime driver interaction, kernel launch orchestration.
- **CN:** 定义测试函数 `test_ws_wgmma_wait_visibility`。 装饰器：`pytest.mark.skipif(not is_cuda() or torch.cuda.get_device_capability()[0] != 9, reason='Requires hopper')`、`pytest.mark.parametrize('FAILURE', [True, False])`。 参数：`FAILURE`、`device`、`run_wrapper`、`monkeypatch`、`num_ctas`。 该作用域中的嵌套定义：`ws_default`、`ws_1`、`kernel`。 关键调用包括 `pytest.mark.skipif`、`pytest.mark.parametrize`、`monkeypatch.setenv`、`knobs.refresh_knobs`、`run_in_process`、`mma_block_m` 等另外 22 项。 该作用域涉及pytest 参数化、PyTorch 张量准备与校验、运行时驱动交互、kernel 启动编排。

#### Lines 2333-2341

```python
    if run_wrapper:
        result = run_in_process(test_ws_wgmma_wait_visibility, (FAILURE, device, False, monkeypatch, num_ctas))
        if FAILURE:
            assert_expected_cuda_failure(result.exc)
            assert "Accessing buffer with pending access. Pending access type: warpgroup_mma operand read" in result.driver_stderr_output
        else:
            assert result.exc is None
            assert result.driver_stderr_output == ""
        return
```
- **EN:** Invokes `run_in_process`, `assert_expected_cuda_failure` to execute the test logic. Validates behavior with 3 assertion(s). Branches on runtime or test conditions. Relevant themes: runtime driver interaction, random-data generation.
- **CN:** 调用 `run_in_process`、`assert_expected_cuda_failure` 执行测试逻辑。 通过 3 个断言验证行为。 根据运行时或测试条件进行分支。 相关主题：运行时驱动交互、随机数据生成。

#### Lines 2342-2345

```python

    monkeypatch.setenv("TRITON_INSTRUMENTATION_MODE", "consan")
    monkeypatch.setenv("CUDA_LAUNCH_BLOCKING", "1")
    knobs.refresh_knobs()
```
- **EN:** Invokes `monkeypatch.setenv`, `knobs.refresh_knobs` to execute the test logic. Relevant themes: kernel launch orchestration.
- **CN:** 调用 `monkeypatch.setenv`、`knobs.refresh_knobs` 执行测试逻辑。 相关主题：kernel 启动编排。

#### Lines 2346-2349

```python

    @gluon.jit
    def ws_default(smemA, smemB, bar, FAILURE: ttgl.constexpr, blocked_layout: ttgl.constexpr,
                   mma_layout: ttgl.constexpr):
```
- **EN:** Defines the helper function `ws_default`. Decorators: `gluon.jit`. Parameters: `smemA`, `smemB`, `bar`, `FAILURE`, `blocked_layout`, `mma_layout`. Key calls include `mma_block_m`, `mma_block_n`, `ttgl.zeros`, `hopper.warpgroup_mma`, `hopper.warpgroup_mma_wait`, `mbarrier.arrive`, and 4 more. This scope touches layout transformation reasoning.
- **CN:** 定义辅助函数 `ws_default`。 装饰器：`gluon.jit`。 参数：`smemA`、`smemB`、`bar`、`FAILURE`、`blocked_layout`、`mma_layout`。 关键调用包括 `mma_block_m`、`mma_block_n`、`ttgl.zeros`、`hopper.warpgroup_mma`、`hopper.warpgroup_mma_wait`、`mbarrier.arrive` 等另外 4 项。 该作用域涉及布局变换推理。

##### Lines 2350-2359

```python
        block_m: ttgl.constexpr = mma_block_m(ttgl.num_ctas())
        block_n: ttgl.constexpr = mma_block_n(ttgl.num_ctas())
        acc = ttgl.zeros([block_m, block_n], ttgl.float16, mma_layout)
        # Issue two async MMAs on two different buffers
        acc = hopper.warpgroup_mma(smemA.index(0), smemB.index(0), acc, is_async=True)
        acc = hopper.warpgroup_mma(smemA.index(1), smemB.index(1), acc, is_async=True)
        # Wait until only 1 outstanding remains
        hopper.warpgroup_mma_wait(num_outstanding=1, deps=[acc])
        # Signal to consumer
        mbarrier.arrive(bar.index(0), count=1)
```
- **EN:** Prepares or updates state through `block_m`, `block_n`, `acc`. Invokes `mma_block_m`, `ttgl.num_ctas`, `mma_block_n`, `ttgl.zeros`, `hopper.warpgroup_mma`, `smemA.index`, and 4 more to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `block_m`、`block_n`、`acc` 准备或更新状态。 调用 `mma_block_m`、`ttgl.num_ctas`、`mma_block_n`、`ttgl.zeros`、`hopper.warpgroup_mma`、`smemA.index` 等另外 4 项 执行测试逻辑。 相关主题：布局变换推理。

#### Lines 2360-2362

```python

    @gluon.jit
    def ws_1(smemA, smemB, bar, FAILURE: ttgl.constexpr, blocked_layout: ttgl.constexpr):
```
- **EN:** Defines the helper function `ws_1`. Decorators: `gluon.jit`. Parameters: `smemA`, `smemB`, `bar`, `FAILURE`, `blocked_layout`. Key calls include `mma_block_m`, `mbarrier.wait`, `ttgl.full`, `ttgl.num_ctas`, `bar.index`, `smemA.index`. This scope touches layout transformation reasoning.
- **CN:** 定义辅助函数 `ws_1`。 装饰器：`gluon.jit`。 参数：`smemA`、`smemB`、`bar`、`FAILURE`、`blocked_layout`。 关键调用包括 `mma_block_m`、`mbarrier.wait`、`ttgl.full`、`ttgl.num_ctas`、`bar.index`、`smemA.index`。 该作用域涉及布局变换推理。

##### Lines 2363-2366

```python
        block_m: ttgl.constexpr = mma_block_m(ttgl.num_ctas())
        mbarrier.wait(bar.index(0), phase=0)
        val = ttgl.full([block_m, XBLOCK], 42, ttgl.float16, blocked_layout)
        smemA.index(1 if FAILURE else 0).store(val)
```
- **EN:** Prepares or updates state through `block_m`, `val`. Invokes `mma_block_m`, `ttgl.num_ctas`, `mbarrier.wait`, `bar.index`, `ttgl.full`, `smemA.index` to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `block_m`、`val` 准备或更新状态。 调用 `mma_block_m`、`ttgl.num_ctas`、`mbarrier.wait`、`bar.index`、`ttgl.full`、`smemA.index` 执行测试逻辑。 相关主题：布局变换推理。

#### Lines 2367-2369

```python

    @gluon.jit
    def kernel(FAILURE: ttgl.constexpr):
```
- **EN:** Defines the helper function `kernel`. Decorators: `gluon.jit`. Parameters: `FAILURE`. Key calls include `mma_block_m`, `mma_block_n`, `mma_cga_layout`, `ttgl.NVMMASharedLayout`, `ttgl.BlockedLayout`, `ttgl.NVMMADistributedLayout`, and 6 more. This scope touches layout transformation reasoning.
- **CN:** 定义辅助函数 `kernel`。 装饰器：`gluon.jit`。 参数：`FAILURE`。 关键调用包括 `mma_block_m`、`mma_block_n`、`mma_cga_layout`、`ttgl.NVMMASharedLayout`、`ttgl.BlockedLayout`、`ttgl.NVMMADistributedLayout` 等另外 6 项。 该作用域涉及布局变换推理。

##### Lines 2370-2390

```python
        block_m: ttgl.constexpr = mma_block_m(ttgl.num_ctas())
        block_n: ttgl.constexpr = mma_block_n(ttgl.num_ctas())
        cga_layout_a: ttgl.constexpr = mma_cga_layout(ttgl.num_ctas(), 0)
        cga_layout_b: ttgl.constexpr = mma_cga_layout(ttgl.num_ctas(), 1)
        cga_layout_c: ttgl.constexpr = mma_cga_layout(ttgl.num_ctas(), 2)
        smem_layout_a: ttgl.constexpr = ttgl.NVMMASharedLayout(swizzle_byte_width=128, element_bitwidth=16, rank=2,
                                                               cga_layout=cga_layout_a)
        smem_layout_b: ttgl.constexpr = ttgl.NVMMASharedLayout(swizzle_byte_width=128, element_bitwidth=16, rank=2,
                                                               cga_layout=cga_layout_b)
        blocked_layout: ttgl.constexpr = ttgl.BlockedLayout(size_per_thread=[1, XBLOCK], threads_per_warp=[32, 1],
                                                            warps_per_cta=[4, 1], order=[0, 1], cga_layout=cga_layout_a)
        mma_layout: ttgl.constexpr = ttgl.NVMMADistributedLayout(version=[3, 0], warps_per_cta=[4, 1],
                                                                 instr_shape=[16, 32, 16], cga_layout=cga_layout_c)
        smemA = ttgl.allocate_shared_memory(ttgl.float16, [2, block_m, XBLOCK], smem_layout_a)
        smemB = ttgl.allocate_shared_memory(ttgl.float16, [2, XBLOCK, block_n], smem_layout_b)
        bar = mbarrier.allocate_mbarrier(batch=1)
        mbarrier.init(bar.index(0), count=1)
        ttgl.warp_specialize([
            (ws_default, (smemA, smemB, bar, FAILURE, blocked_layout, mma_layout)),
            (ws_1, (smemA, smemB, bar, FAILURE, blocked_layout)),
        ], [4], [32])
```
- **EN:** Prepares or updates state through `block_m`, `block_n`, `cga_layout_a`, `cga_layout_b`, `cga_layout_c`, `smem_layout_a`, `smem_layout_b`, `blocked_layout`, and 4 more. Invokes `mma_block_m`, `ttgl.num_ctas`, `mma_block_n`, `mma_cga_layout`, `ttgl.NVMMASharedLayout`, `ttgl.BlockedLayout`, and 6 more to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `block_m`、`block_n`、`cga_layout_a`、`cga_layout_b`、`cga_layout_c`、`smem_layout_a`、`smem_layout_b`、`blocked_layout` 等另外 4 项 准备或更新状态。 调用 `mma_block_m`、`ttgl.num_ctas`、`mma_block_n`、`mma_cga_layout`、`ttgl.NVMMASharedLayout`、`ttgl.BlockedLayout` 等另外 6 项 执行测试逻辑。 相关主题：布局变换推理。

#### Lines 2391-2392

```python

    kernel[(1, )](FAILURE=FAILURE, num_warps=4, num_ctas=num_ctas)
```
- **EN:** Carries supporting logic for the surrounding test or helper scope.
- **CN:** 为周围的测试或辅助作用域提供支撑逻辑。

### Lines 2393-2396

```python


@pytest.mark.skipif(not is_cuda() or torch.cuda.get_device_capability()[0] < 9, reason="Requires hopper")
def test_deadlock_two_partitions(device, run_wrapper, monkeypatch, num_ctas):
```
- **EN:** Defines the test function `test_deadlock_two_partitions`. Decorators: `pytest.mark.skipif(not is_cuda() or torch.cuda.get_device_capability()[0] < 9, reason='Requires hopper')`. Parameters: `device`, `run_wrapper`, `monkeypatch`, `num_ctas`. Nested definitions in this scope: `ws_default`, `ws_1`, `kernel`. Key calls include `pytest.mark.skipif`, `monkeypatch.setenv`, `knobs.refresh_knobs`, `run_in_process`, `assert_expected_cuda_failure`, `mbarrier.wait`, and 6 more. This scope touches PyTorch tensor setup and checks, runtime driver interaction, kernel launch orchestration.
- **CN:** 定义测试函数 `test_deadlock_two_partitions`。 装饰器：`pytest.mark.skipif(not is_cuda() or torch.cuda.get_device_capability()[0] < 9, reason='Requires hopper')`。 参数：`device`、`run_wrapper`、`monkeypatch`、`num_ctas`。 该作用域中的嵌套定义：`ws_default`、`ws_1`、`kernel`。 关键调用包括 `pytest.mark.skipif`、`monkeypatch.setenv`、`knobs.refresh_knobs`、`run_in_process`、`assert_expected_cuda_failure`、`mbarrier.wait` 等另外 6 项。 该作用域涉及PyTorch 张量准备与校验、运行时驱动交互、kernel 启动编排。

#### Lines 2397-2401

```python
    if run_wrapper:
        result = run_in_process(test_deadlock_two_partitions, (device, False, monkeypatch, num_ctas))
        assert_expected_cuda_failure(result.exc)
        assert "Deadlock detected" in result.driver_stderr_output
        return
```
- **EN:** Invokes `run_in_process`, `assert_expected_cuda_failure` to execute the test logic. Validates behavior with 1 assertion(s). Branches on runtime or test conditions. Relevant themes: runtime driver interaction.
- **CN:** 调用 `run_in_process`、`assert_expected_cuda_failure` 执行测试逻辑。 通过 1 个断言验证行为。 根据运行时或测试条件进行分支。 相关主题：运行时驱动交互。

#### Lines 2402-2404

```python
    monkeypatch.setenv("TRITON_INSTRUMENTATION_MODE", "consan")
    monkeypatch.setenv("CUDA_LAUNCH_BLOCKING", "1")
    knobs.refresh_knobs()
```
- **EN:** Invokes `monkeypatch.setenv`, `knobs.refresh_knobs` to execute the test logic. Relevant themes: kernel launch orchestration.
- **CN:** 调用 `monkeypatch.setenv`、`knobs.refresh_knobs` 执行测试逻辑。 相关主题：kernel 启动编排。

#### Lines 2405-2407

```python

    @gluon.jit
    def ws_default(bar):
```
- **EN:** Defines the helper function `ws_default`. Decorators: `gluon.jit`. Parameters: `bar`. Key calls include `mbarrier.wait`, `bar.index`.
- **CN:** 定义辅助函数 `ws_default`。 装饰器：`gluon.jit`。 参数：`bar`。 关键调用包括 `mbarrier.wait`、`bar.index`。

##### Lines 2408-2408

```python
        mbarrier.wait(bar.index(0), phase=0)
```
- **EN:** Invokes `mbarrier.wait`, `bar.index` to execute the test logic.
- **CN:** 调用 `mbarrier.wait`、`bar.index` 执行测试逻辑。

#### Lines 2409-2411

```python

    @gluon.jit
    def ws_1(bar):
```
- **EN:** Defines the helper function `ws_1`. Decorators: `gluon.jit`. Parameters: `bar`. Key calls include `mbarrier.wait`, `bar.index`.
- **CN:** 定义辅助函数 `ws_1`。 装饰器：`gluon.jit`。 参数：`bar`。 关键调用包括 `mbarrier.wait`、`bar.index`。

##### Lines 2412-2412

```python
        mbarrier.wait(bar.index(1), phase=0)
```
- **EN:** Invokes `mbarrier.wait`, `bar.index` to execute the test logic.
- **CN:** 调用 `mbarrier.wait`、`bar.index` 执行测试逻辑。

#### Lines 2413-2415

```python

    @gluon.jit
    def kernel():
```
- **EN:** Defines the helper function `kernel`. Decorators: `gluon.jit`. Key calls include `mbarrier.allocate_mbarrier`, `mbarrier.init`, `ttgl.warp_specialize`, `bar.index`.
- **CN:** 定义辅助函数 `kernel`。 装饰器：`gluon.jit`。 关键调用包括 `mbarrier.allocate_mbarrier`、`mbarrier.init`、`ttgl.warp_specialize`、`bar.index`。

##### Lines 2416-2422

```python
        bar = mbarrier.allocate_mbarrier(batch=2)
        mbarrier.init(bar.index(0), count=1)
        mbarrier.init(bar.index(1), count=1)
        ttgl.warp_specialize([
            (ws_default, (bar, )),
            (ws_1, (bar, )),
        ], [4], [32])
```
- **EN:** Prepares or updates state through `bar`. Invokes `mbarrier.allocate_mbarrier`, `mbarrier.init`, `bar.index`, `ttgl.warp_specialize` to execute the test logic.
- **CN:** 通过 `bar` 准备或更新状态。 调用 `mbarrier.allocate_mbarrier`、`mbarrier.init`、`bar.index`、`ttgl.warp_specialize` 执行测试逻辑。

#### Lines 2423-2424

```python

    kernel[(1, )](num_warps=4, num_ctas=num_ctas)
```
- **EN:** Carries supporting logic for the surrounding test or helper scope.
- **CN:** 为周围的测试或辅助作用域提供支撑逻辑。

### Lines 2425-2428

```python


@pytest.mark.skipif(not is_cuda() or torch.cuda.get_device_capability()[0] < 9, reason="Requires hopper")
def test_deadlock_with_padded_warp_specialize_partition(device, run_wrapper, monkeypatch, num_ctas):
```
- **EN:** Defines the test function `test_deadlock_with_padded_warp_specialize_partition`. Decorators: `pytest.mark.skipif(not is_cuda() or torch.cuda.get_device_capability()[0] < 9, reason='Requires hopper')`. Parameters: `device`, `run_wrapper`, `monkeypatch`, `num_ctas`. Nested definitions in this scope: `wait_forever`, `done`, `kernel`. Key calls include `pytest.mark.skipif`, `monkeypatch.setenv`, `knobs.refresh_knobs`, `run_in_process`, `assert_expected_cuda_failure`, `mbarrier.wait`, and 5 more. This scope touches PyTorch tensor setup and checks, runtime driver interaction, kernel launch orchestration.
- **CN:** 定义测试函数 `test_deadlock_with_padded_warp_specialize_partition`。 装饰器：`pytest.mark.skipif(not is_cuda() or torch.cuda.get_device_capability()[0] < 9, reason='Requires hopper')`。 参数：`device`、`run_wrapper`、`monkeypatch`、`num_ctas`。 该作用域中的嵌套定义：`wait_forever`、`done`、`kernel`。 关键调用包括 `pytest.mark.skipif`、`monkeypatch.setenv`、`knobs.refresh_knobs`、`run_in_process`、`assert_expected_cuda_failure`、`mbarrier.wait` 等另外 5 项。 该作用域涉及PyTorch 张量准备与校验、运行时驱动交互、kernel 启动编排。

#### Lines 2429-2434

```python
    if run_wrapper:
        result = run_in_process(test_deadlock_with_padded_warp_specialize_partition,
                                (device, False, monkeypatch, num_ctas))
        assert_expected_cuda_failure(result.exc)
        assert "Deadlock detected" in result.driver_stderr_output
        return
```
- **EN:** Invokes `run_in_process`, `assert_expected_cuda_failure` to execute the test logic. Validates behavior with 1 assertion(s). Branches on runtime or test conditions. Relevant themes: runtime driver interaction.
- **CN:** 调用 `run_in_process`、`assert_expected_cuda_failure` 执行测试逻辑。 通过 1 个断言验证行为。 根据运行时或测试条件进行分支。 相关主题：运行时驱动交互。

#### Lines 2435-2437

```python
    monkeypatch.setenv("TRITON_INSTRUMENTATION_MODE", "consan")
    monkeypatch.setenv("CUDA_LAUNCH_BLOCKING", "1")
    knobs.refresh_knobs()
```
- **EN:** Invokes `monkeypatch.setenv`, `knobs.refresh_knobs` to execute the test logic. Relevant themes: kernel launch orchestration.
- **CN:** 调用 `monkeypatch.setenv`、`knobs.refresh_knobs` 执行测试逻辑。 相关主题：kernel 启动编排。

#### Lines 2438-2440

```python

    @gluon.jit
    def wait_forever(bar):
```
- **EN:** Defines the helper function `wait_forever`. Decorators: `gluon.jit`. Parameters: `bar`. Key calls include `mbarrier.wait`.
- **CN:** 定义辅助函数 `wait_forever`。 装饰器：`gluon.jit`。 参数：`bar`。 关键调用包括 `mbarrier.wait`。

##### Lines 2441-2441

```python
        mbarrier.wait(bar, phase=0)
```
- **EN:** Invokes `mbarrier.wait` to execute the test logic.
- **CN:** 调用 `mbarrier.wait` 执行测试逻辑。

#### Lines 2442-2444

```python

    @gluon.jit
    def done(bar):
```
- **EN:** Defines the helper function `done`. Decorators: `gluon.jit`. Parameters: `bar`.
- **CN:** 定义辅助函数 `done`。 装饰器：`gluon.jit`。 参数：`bar`。

##### Lines 2445-2445

```python
        pass
```
- **EN:** Carries supporting logic for the surrounding test or helper scope.
- **CN:** 为周围的测试或辅助作用域提供支撑逻辑。

#### Lines 2446-2448

```python

    @gluon.jit
    def kernel():
```
- **EN:** Defines the helper function `kernel`. Decorators: `gluon.jit`. Key calls include `mbarrier.allocate_mbarrier`, `mbarrier.init`, `ttgl.warp_specialize`.
- **CN:** 定义辅助函数 `kernel`。 装饰器：`gluon.jit`。 关键调用包括 `mbarrier.allocate_mbarrier`、`mbarrier.init`、`ttgl.warp_specialize`。

##### Lines 2449-2456

```python
        bar = mbarrier.allocate_mbarrier()
        mbarrier.init(bar, count=1)
        ttgl.warp_specialize([
            (done, (bar, )),
            (wait_forever, (bar, )),
            (wait_forever, (bar, )),
            (wait_forever, (bar, )),
        ], [1, 1, 1], [32, 32, 32])
```
- **EN:** Prepares or updates state through `bar`. Invokes `mbarrier.allocate_mbarrier`, `mbarrier.init`, `ttgl.warp_specialize` to execute the test logic.
- **CN:** 通过 `bar` 准备或更新状态。 调用 `mbarrier.allocate_mbarrier`、`mbarrier.init`、`ttgl.warp_specialize` 执行测试逻辑。

#### Lines 2457-2458

```python

    kernel[(1, )](num_warps=4, num_ctas=num_ctas)
```
- **EN:** Carries supporting logic for the surrounding test or helper scope.
- **CN:** 为周围的测试或辅助作用域提供支撑逻辑。

### Lines 2459-2463

```python


@pytest.mark.skipif(not is_cuda() or torch.cuda.get_device_capability()[0] < 9, reason="Requires hopper")
@pytest.mark.parametrize("TWO_CTAS", [False, True], ids=["single-cta-barrier", "two-cta-barrier"])
def test_deadlock_after_other_partition_returns(TWO_CTAS, device, run_wrapper, monkeypatch, num_ctas):
```
- **EN:** Defines the test function `test_deadlock_after_other_partition_returns`. Decorators: `pytest.mark.skipif(not is_cuda() or torch.cuda.get_device_capability()[0] < 9, reason='Requires hopper')`, `pytest.mark.parametrize('TWO_CTAS', [False, True], ids=['single-cta-barrier', 'two-cta-barrier'])`. Parameters: `TWO_CTAS`, `device`, `run_wrapper`, `monkeypatch`, `num_ctas`. Nested definitions in this scope: `done`, `wait_forever`, `complete_and_return`, `kernel`. Key calls include `pytest.mark.skipif`, `pytest.mark.parametrize`, `monkeypatch.setenv`, `knobs.refresh_knobs`, `pytest.skip`, `run_in_process`, and 9 more. This scope touches pytest parametrization, PyTorch tensor setup and checks, runtime driver interaction, kernel launch orchestration.
- **CN:** 定义测试函数 `test_deadlock_after_other_partition_returns`。 装饰器：`pytest.mark.skipif(not is_cuda() or torch.cuda.get_device_capability()[0] < 9, reason='Requires hopper')`、`pytest.mark.parametrize('TWO_CTAS', [False, True], ids=['single-cta-barrier', 'two-cta-barrier'])`。 参数：`TWO_CTAS`、`device`、`run_wrapper`、`monkeypatch`、`num_ctas`。 该作用域中的嵌套定义：`done`、`wait_forever`、`complete_and_return`、`kernel`。 关键调用包括 `pytest.mark.skipif`、`pytest.mark.parametrize`、`monkeypatch.setenv`、`knobs.refresh_knobs`、`pytest.skip`、`run_in_process` 等另外 9 项。 该作用域涉及pytest 参数化、PyTorch 张量准备与校验、运行时驱动交互、kernel 启动编排。

#### Lines 2464-2465

```python
    if TWO_CTAS and num_ctas == 1:
        pytest.skip("two-CTA barriers require at least two CTAs")
```
- **EN:** Invokes `pytest.skip` to execute the test logic. Branches on runtime or test conditions.
- **CN:** 调用 `pytest.skip` 执行测试逻辑。 根据运行时或测试条件进行分支。

#### Lines 2466-2471

```python
    if run_wrapper:
        result = run_in_process(test_deadlock_after_other_partition_returns,
                                (TWO_CTAS, device, False, monkeypatch, num_ctas))
        assert_expected_cuda_failure(result.exc)
        assert "Deadlock detected" in result.driver_stderr_output
        return
```
- **EN:** Invokes `run_in_process`, `assert_expected_cuda_failure` to execute the test logic. Validates behavior with 1 assertion(s). Branches on runtime or test conditions. Relevant themes: runtime driver interaction.
- **CN:** 调用 `run_in_process`、`assert_expected_cuda_failure` 执行测试逻辑。 通过 1 个断言验证行为。 根据运行时或测试条件进行分支。 相关主题：运行时驱动交互。

#### Lines 2472-2474

```python
    monkeypatch.setenv("TRITON_INSTRUMENTATION_MODE", "consan")
    monkeypatch.setenv("CUDA_LAUNCH_BLOCKING", "1")
    knobs.refresh_knobs()
```
- **EN:** Invokes `monkeypatch.setenv`, `knobs.refresh_knobs` to execute the test logic. Relevant themes: kernel launch orchestration.
- **CN:** 调用 `monkeypatch.setenv`、`knobs.refresh_knobs` 执行测试逻辑。 相关主题：kernel 启动编排。

#### Lines 2475-2477

```python

    @gluon.jit
    def done(bar):
```
- **EN:** Defines the helper function `done`. Decorators: `gluon.jit`. Parameters: `bar`.
- **CN:** 定义辅助函数 `done`。 装饰器：`gluon.jit`。 参数：`bar`。

##### Lines 2478-2478

```python
        pass
```
- **EN:** Carries supporting logic for the surrounding test or helper scope.
- **CN:** 为周围的测试或辅助作用域提供支撑逻辑。

#### Lines 2479-2481

```python

    @gluon.jit
    def wait_forever(bar):
```
- **EN:** Defines the helper function `wait_forever`. Decorators: `gluon.jit`. Parameters: `bar`. Key calls include `mbarrier.wait`, `bar.index`.
- **CN:** 定义辅助函数 `wait_forever`。 装饰器：`gluon.jit`。 参数：`bar`。 关键调用包括 `mbarrier.wait`、`bar.index`。

##### Lines 2482-2482

```python
        mbarrier.wait(bar.index(0), phase=0)
```
- **EN:** Invokes `mbarrier.wait`, `bar.index` to execute the test logic.
- **CN:** 调用 `mbarrier.wait`、`bar.index` 执行测试逻辑。

#### Lines 2483-2485

```python

    @gluon.jit
    def complete_and_return(bar):
```
- **EN:** Defines the helper function `complete_and_return`. Decorators: `gluon.jit`. Parameters: `bar`. Key calls include `mbarrier.arrive`, `bar.index`.
- **CN:** 定义辅助函数 `complete_and_return`。 装饰器：`gluon.jit`。 参数：`bar`。 关键调用包括 `mbarrier.arrive`、`bar.index`。

##### Lines 2486-2486

```python
        mbarrier.arrive(bar.index(1), count=1)
```
- **EN:** Invokes `mbarrier.arrive`, `bar.index` to execute the test logic.
- **CN:** 调用 `mbarrier.arrive`、`bar.index` 执行测试逻辑。

#### Lines 2487-2489

```python

    @gluon.jit
    def kernel(TWO_CTAS: ttgl.constexpr):
```
- **EN:** Defines the helper function `kernel`. Decorators: `gluon.jit`. Parameters: `TWO_CTAS`. Key calls include `mbarrier.allocate_mbarrier`, `mbarrier.init`, `ttgl.warp_specialize`, `bar.index`.
- **CN:** 定义辅助函数 `kernel`。 装饰器：`gluon.jit`。 参数：`TWO_CTAS`。 关键调用包括 `mbarrier.allocate_mbarrier`、`mbarrier.init`、`ttgl.warp_specialize`、`bar.index`。

##### Lines 2490-2497

```python
        bar = mbarrier.allocate_mbarrier(batch=2, two_ctas=TWO_CTAS)
        mbarrier.init(bar.index(0), count=1)
        mbarrier.init(bar.index(1), count=1)
        ttgl.warp_specialize([
            (done, (bar, )),
            (wait_forever, (bar, )),
            (complete_and_return, (bar, )),
        ], [4, 4], [32, 32])
```
- **EN:** Prepares or updates state through `bar`. Invokes `mbarrier.allocate_mbarrier`, `mbarrier.init`, `bar.index`, `ttgl.warp_specialize` to execute the test logic.
- **CN:** 通过 `bar` 准备或更新状态。 调用 `mbarrier.allocate_mbarrier`、`mbarrier.init`、`bar.index`、`ttgl.warp_specialize` 执行测试逻辑。

#### Lines 2498-2499

```python

    kernel[(1, )](TWO_CTAS, num_warps=4, num_ctas=num_ctas)
```
- **EN:** Carries supporting logic for the surrounding test or helper scope.
- **CN:** 为周围的测试或辅助作用域提供支撑逻辑。

### Lines 2500-2503

```python


@pytest.mark.skipif(not is_cuda() or torch.cuda.get_device_capability()[0] < 9, reason="Requires hopper")
def test_deadlock_overarrival(device, run_wrapper, monkeypatch, num_ctas):
```
- **EN:** Defines the test function `test_deadlock_overarrival`. Decorators: `pytest.mark.skipif(not is_cuda() or torch.cuda.get_device_capability()[0] < 9, reason='Requires hopper')`. Parameters: `device`, `run_wrapper`, `monkeypatch`, `num_ctas`. Nested definitions in this scope: `kernel`. Key calls include `pytest.mark.skipif`, `monkeypatch.setenv`, `knobs.refresh_knobs`, `run_in_process`, `assert_expected_cuda_failure`, `mbarrier.allocate_mbarrier`, and 6 more. This scope touches PyTorch tensor setup and checks, runtime driver interaction, kernel launch orchestration.
- **CN:** 定义测试函数 `test_deadlock_overarrival`。 装饰器：`pytest.mark.skipif(not is_cuda() or torch.cuda.get_device_capability()[0] < 9, reason='Requires hopper')`。 参数：`device`、`run_wrapper`、`monkeypatch`、`num_ctas`。 该作用域中的嵌套定义：`kernel`。 关键调用包括 `pytest.mark.skipif`、`monkeypatch.setenv`、`knobs.refresh_knobs`、`run_in_process`、`assert_expected_cuda_failure`、`mbarrier.allocate_mbarrier` 等另外 6 项。 该作用域涉及PyTorch 张量准备与校验、运行时驱动交互、kernel 启动编排。

#### Lines 2504-2508

```python
    if run_wrapper:
        result = run_in_process(test_deadlock_overarrival, (device, False, monkeypatch, num_ctas))
        assert_expected_cuda_failure(result.exc)
        assert "Deadlock detected" in result.driver_stderr_output
        return
```
- **EN:** Invokes `run_in_process`, `assert_expected_cuda_failure` to execute the test logic. Validates behavior with 1 assertion(s). Branches on runtime or test conditions. Relevant themes: runtime driver interaction.
- **CN:** 调用 `run_in_process`、`assert_expected_cuda_failure` 执行测试逻辑。 通过 1 个断言验证行为。 根据运行时或测试条件进行分支。 相关主题：运行时驱动交互。

#### Lines 2509-2511

```python
    monkeypatch.setenv("TRITON_INSTRUMENTATION_MODE", "consan")
    monkeypatch.setenv("CUDA_LAUNCH_BLOCKING", "1")
    knobs.refresh_knobs()
```
- **EN:** Invokes `monkeypatch.setenv`, `knobs.refresh_knobs` to execute the test logic. Relevant themes: kernel launch orchestration.
- **CN:** 调用 `monkeypatch.setenv`、`knobs.refresh_knobs` 执行测试逻辑。 相关主题：kernel 启动编排。

#### Lines 2512-2514

```python

    @gluon.jit
    def kernel():
```
- **EN:** Defines the helper function `kernel`. Decorators: `gluon.jit`. Key calls include `mbarrier.allocate_mbarrier`, `mbarrier.init`, `mbarrier.arrive`, `mbarrier.wait`, `bar.index`.
- **CN:** 定义辅助函数 `kernel`。 装饰器：`gluon.jit`。 关键调用包括 `mbarrier.allocate_mbarrier`、`mbarrier.init`、`mbarrier.arrive`、`mbarrier.wait`、`bar.index`。

##### Lines 2515-2521

```python
        bar = mbarrier.allocate_mbarrier(batch=2)
        mbarrier.init(bar.index(0), count=1)
        mbarrier.init(bar.index(1), count=1)

        mbarrier.arrive(bar.index(0), count=1)
        mbarrier.arrive(bar.index(0), count=1)
        mbarrier.wait(bar.index(0), phase=0)
```
- **EN:** Prepares or updates state through `bar`. Invokes `mbarrier.allocate_mbarrier`, `mbarrier.init`, `bar.index`, `mbarrier.arrive`, `mbarrier.wait` to execute the test logic.
- **CN:** 通过 `bar` 准备或更新状态。 调用 `mbarrier.allocate_mbarrier`、`mbarrier.init`、`bar.index`、`mbarrier.arrive`、`mbarrier.wait` 执行测试逻辑。

#### Lines 2522-2523

```python

    kernel[(1, )](num_warps=4, num_ctas=num_ctas)
```
- **EN:** Carries supporting logic for the surrounding test or helper scope.
- **CN:** 为周围的测试或辅助作用域提供支撑逻辑。

### Lines 2524-2527

```python


@pytest.mark.skipif(not is_cuda() or torch.cuda.get_device_capability()[0] < 9, reason="Requires hopper")
def test_deadlock_underarrival(device, run_wrapper, monkeypatch, num_ctas):
```
- **EN:** Defines the test function `test_deadlock_underarrival`. Decorators: `pytest.mark.skipif(not is_cuda() or torch.cuda.get_device_capability()[0] < 9, reason='Requires hopper')`. Parameters: `device`, `run_wrapper`, `monkeypatch`, `num_ctas`. Nested definitions in this scope: `ws_default`, `ws_1`, `kernel`. Key calls include `pytest.mark.skipif`, `monkeypatch.setenv`, `knobs.refresh_knobs`, `run_in_process`, `assert_expected_cuda_failure`, `mbarrier.arrive`, and 7 more. This scope touches PyTorch tensor setup and checks, runtime driver interaction, kernel launch orchestration.
- **CN:** 定义测试函数 `test_deadlock_underarrival`。 装饰器：`pytest.mark.skipif(not is_cuda() or torch.cuda.get_device_capability()[0] < 9, reason='Requires hopper')`。 参数：`device`、`run_wrapper`、`monkeypatch`、`num_ctas`。 该作用域中的嵌套定义：`ws_default`、`ws_1`、`kernel`。 关键调用包括 `pytest.mark.skipif`、`monkeypatch.setenv`、`knobs.refresh_knobs`、`run_in_process`、`assert_expected_cuda_failure`、`mbarrier.arrive` 等另外 7 项。 该作用域涉及PyTorch 张量准备与校验、运行时驱动交互、kernel 启动编排。

#### Lines 2528-2532

```python
    if run_wrapper:
        result = run_in_process(test_deadlock_underarrival, (device, False, monkeypatch, num_ctas))
        assert_expected_cuda_failure(result.exc)
        assert "Deadlock detected" in result.driver_stderr_output
        return
```
- **EN:** Invokes `run_in_process`, `assert_expected_cuda_failure` to execute the test logic. Validates behavior with 1 assertion(s). Branches on runtime or test conditions. Relevant themes: runtime driver interaction.
- **CN:** 调用 `run_in_process`、`assert_expected_cuda_failure` 执行测试逻辑。 通过 1 个断言验证行为。 根据运行时或测试条件进行分支。 相关主题：运行时驱动交互。

#### Lines 2533-2535

```python
    monkeypatch.setenv("TRITON_INSTRUMENTATION_MODE", "consan")
    monkeypatch.setenv("CUDA_LAUNCH_BLOCKING", "1")
    knobs.refresh_knobs()
```
- **EN:** Invokes `monkeypatch.setenv`, `knobs.refresh_knobs` to execute the test logic. Relevant themes: kernel launch orchestration.
- **CN:** 调用 `monkeypatch.setenv`、`knobs.refresh_knobs` 执行测试逻辑。 相关主题：kernel 启动编排。

#### Lines 2536-2538

```python

    @gluon.jit
    def ws_default(bar):
```
- **EN:** Defines the helper function `ws_default`. Decorators: `gluon.jit`. Parameters: `bar`. Key calls include `mbarrier.arrive`, `mbarrier.wait`, `bar.index`.
- **CN:** 定义辅助函数 `ws_default`。 装饰器：`gluon.jit`。 参数：`bar`。 关键调用包括 `mbarrier.arrive`、`mbarrier.wait`、`bar.index`。

##### Lines 2539-2540

```python
        mbarrier.arrive(bar.index(1), count=1)
        mbarrier.wait(bar.index(0), phase=0)
```
- **EN:** Invokes `mbarrier.arrive`, `bar.index`, `mbarrier.wait` to execute the test logic.
- **CN:** 调用 `mbarrier.arrive`、`bar.index`、`mbarrier.wait` 执行测试逻辑。

#### Lines 2541-2543

```python

    @gluon.jit
    def ws_1(bar):
```
- **EN:** Defines the helper function `ws_1`. Decorators: `gluon.jit`. Parameters: `bar`. Key calls include `mbarrier.arrive`, `mbarrier.wait`, `bar.index`.
- **CN:** 定义辅助函数 `ws_1`。 装饰器：`gluon.jit`。 参数：`bar`。 关键调用包括 `mbarrier.arrive`、`mbarrier.wait`、`bar.index`。

##### Lines 2544-2545

```python
        mbarrier.arrive(bar.index(0), count=1)
        mbarrier.wait(bar.index(1), phase=0)
```
- **EN:** Invokes `mbarrier.arrive`, `bar.index`, `mbarrier.wait` to execute the test logic.
- **CN:** 调用 `mbarrier.arrive`、`bar.index`、`mbarrier.wait` 执行测试逻辑。

#### Lines 2546-2548

```python

    @gluon.jit
    def kernel():
```
- **EN:** Defines the helper function `kernel`. Decorators: `gluon.jit`. Key calls include `mbarrier.allocate_mbarrier`, `mbarrier.init`, `ttgl.warp_specialize`, `bar.index`.
- **CN:** 定义辅助函数 `kernel`。 装饰器：`gluon.jit`。 关键调用包括 `mbarrier.allocate_mbarrier`、`mbarrier.init`、`ttgl.warp_specialize`、`bar.index`。

##### Lines 2549-2555

```python
        bar = mbarrier.allocate_mbarrier(batch=2)
        mbarrier.init(bar.index(0), count=2)
        mbarrier.init(bar.index(1), count=2)
        ttgl.warp_specialize([
            (ws_default, (bar, )),
            (ws_1, (bar, )),
        ], [4], [32])
```
- **EN:** Prepares or updates state through `bar`. Invokes `mbarrier.allocate_mbarrier`, `mbarrier.init`, `bar.index`, `ttgl.warp_specialize` to execute the test logic.
- **CN:** 通过 `bar` 准备或更新状态。 调用 `mbarrier.allocate_mbarrier`、`mbarrier.init`、`bar.index`、`ttgl.warp_specialize` 执行测试逻辑。

#### Lines 2556-2557

```python

    kernel[(1, )](num_warps=4, num_ctas=num_ctas)
```
- **EN:** Carries supporting logic for the surrounding test or helper scope.
- **CN:** 为周围的测试或辅助作用域提供支撑逻辑。

### Lines 2558-2561

```python


@pytest.mark.skipif(not is_cuda() or torch.cuda.get_device_capability()[0] < 9, reason="Requires hopper")
def test_deadlock_different_phases(device, run_wrapper, monkeypatch, num_ctas):
```
- **EN:** Defines the test function `test_deadlock_different_phases`. Decorators: `pytest.mark.skipif(not is_cuda() or torch.cuda.get_device_capability()[0] < 9, reason='Requires hopper')`. Parameters: `device`, `run_wrapper`, `monkeypatch`, `num_ctas`. Nested definitions in this scope: `ws_default`, `ws_1`, `kernel`. Key calls include `pytest.mark.skipif`, `monkeypatch.setenv`, `knobs.refresh_knobs`, `run_in_process`, `mbarrier.wait`, `mbarrier.arrive`, and 6 more. This scope touches PyTorch tensor setup and checks, runtime driver interaction, kernel launch orchestration.
- **CN:** 定义测试函数 `test_deadlock_different_phases`。 装饰器：`pytest.mark.skipif(not is_cuda() or torch.cuda.get_device_capability()[0] < 9, reason='Requires hopper')`。 参数：`device`、`run_wrapper`、`monkeypatch`、`num_ctas`。 该作用域中的嵌套定义：`ws_default`、`ws_1`、`kernel`。 关键调用包括 `pytest.mark.skipif`、`monkeypatch.setenv`、`knobs.refresh_knobs`、`run_in_process`、`mbarrier.wait`、`mbarrier.arrive` 等另外 6 项。 该作用域涉及PyTorch 张量准备与校验、运行时驱动交互、kernel 启动编排。

#### Lines 2562-2566

```python
    if run_wrapper:
        result = run_in_process(test_deadlock_different_phases, (device, False, monkeypatch, num_ctas))
        assert result.exc is None
        assert result.driver_stderr_output == ""
        return
```
- **EN:** Invokes `run_in_process` to execute the test logic. Validates behavior with 2 assertion(s). Branches on runtime or test conditions. Relevant themes: runtime driver interaction.
- **CN:** 调用 `run_in_process` 执行测试逻辑。 通过 2 个断言验证行为。 根据运行时或测试条件进行分支。 相关主题：运行时驱动交互。

#### Lines 2567-2569

```python
    monkeypatch.setenv("TRITON_INSTRUMENTATION_MODE", "consan")
    monkeypatch.setenv("CUDA_LAUNCH_BLOCKING", "1")
    knobs.refresh_knobs()
```
- **EN:** Invokes `monkeypatch.setenv`, `knobs.refresh_knobs` to execute the test logic. Relevant themes: kernel launch orchestration.
- **CN:** 调用 `monkeypatch.setenv`、`knobs.refresh_knobs` 执行测试逻辑。 相关主题：kernel 启动编排。

#### Lines 2570-2572

```python

    @gluon.jit
    def ws_default(bar):
```
- **EN:** Defines the helper function `ws_default`. Decorators: `gluon.jit`. Parameters: `bar`. Key calls include `mbarrier.wait`, `mbarrier.arrive`, `bar.index`.
- **CN:** 定义辅助函数 `ws_default`。 装饰器：`gluon.jit`。 参数：`bar`。 关键调用包括 `mbarrier.wait`、`mbarrier.arrive`、`bar.index`。

##### Lines 2573-2574

```python
        mbarrier.wait(bar.index(0), phase=0)
        mbarrier.arrive(bar.index(0), count=1)
```
- **EN:** Invokes `mbarrier.wait`, `bar.index`, `mbarrier.arrive` to execute the test logic.
- **CN:** 调用 `mbarrier.wait`、`bar.index`、`mbarrier.arrive` 执行测试逻辑。

#### Lines 2575-2577

```python

    @gluon.jit
    def ws_1(bar):
```
- **EN:** Defines the helper function `ws_1`. Decorators: `gluon.jit`. Parameters: `bar`. Key calls include `mbarrier.wait`, `bar.index`.
- **CN:** 定义辅助函数 `ws_1`。 装饰器：`gluon.jit`。 参数：`bar`。 关键调用包括 `mbarrier.wait`、`bar.index`。

##### Lines 2578-2578

```python
        mbarrier.wait(bar.index(0), phase=1)
```
- **EN:** Invokes `mbarrier.wait`, `bar.index` to execute the test logic.
- **CN:** 调用 `mbarrier.wait`、`bar.index` 执行测试逻辑。

#### Lines 2579-2581

```python

    @gluon.jit
    def kernel():
```
- **EN:** Defines the helper function `kernel`. Decorators: `gluon.jit`. Key calls include `mbarrier.allocate_mbarrier`, `mbarrier.init`, `mbarrier.arrive`, `ttgl.warp_specialize`, `bar.index`.
- **CN:** 定义辅助函数 `kernel`。 装饰器：`gluon.jit`。 关键调用包括 `mbarrier.allocate_mbarrier`、`mbarrier.init`、`mbarrier.arrive`、`ttgl.warp_specialize`、`bar.index`。

##### Lines 2582-2588

```python
        bar = mbarrier.allocate_mbarrier(batch=1)
        mbarrier.init(bar.index(0), count=1)
        mbarrier.arrive(bar.index(0), count=1)
        ttgl.warp_specialize([
            (ws_default, (bar, )),
            (ws_1, (bar, )),
        ], [4], [32])
```
- **EN:** Prepares or updates state through `bar`. Invokes `mbarrier.allocate_mbarrier`, `mbarrier.init`, `bar.index`, `mbarrier.arrive`, `ttgl.warp_specialize` to execute the test logic.
- **CN:** 通过 `bar` 准备或更新状态。 调用 `mbarrier.allocate_mbarrier`、`mbarrier.init`、`bar.index`、`mbarrier.arrive`、`ttgl.warp_specialize` 执行测试逻辑。

#### Lines 2589-2590

```python

    kernel[(1, )](num_warps=4, num_ctas=num_ctas)
```
- **EN:** Carries supporting logic for the surrounding test or helper scope.
- **CN:** 为周围的测试或辅助作用域提供支撑逻辑。

### Lines 2591-2594

```python


@pytest.mark.skipif(not is_cuda() or torch.cuda.get_device_capability()[0] < 9, reason="Requires hopper")
def test_deadlock_exempt_when_tma_signals(device, run_wrapper, monkeypatch, num_ctas):
```
- **EN:** Defines the test function `test_deadlock_exempt_when_tma_signals`. Decorators: `pytest.mark.skipif(not is_cuda() or torch.cuda.get_device_capability()[0] < 9, reason='Requires hopper')`. Parameters: `device`, `run_wrapper`, `monkeypatch`, `num_ctas`. Nested definitions in this scope: `ws_default`, `ws_1`, `kernel`. Key calls include `pytest.mark.skipif`, `monkeypatch.setenv`, `knobs.refresh_knobs`, `torch.randn`, `ttgl.NVMMASharedLayout`, `gluon.nvidia.hopper.TensorDescriptor.from_tensor`, and 14 more. This scope touches PyTorch tensor setup and checks, tensor/descriptor metadata, runtime driver interaction, kernel launch orchestration.
- **CN:** 定义测试函数 `test_deadlock_exempt_when_tma_signals`。 装饰器：`pytest.mark.skipif(not is_cuda() or torch.cuda.get_device_capability()[0] < 9, reason='Requires hopper')`。 参数：`device`、`run_wrapper`、`monkeypatch`、`num_ctas`。 该作用域中的嵌套定义：`ws_default`、`ws_1`、`kernel`。 关键调用包括 `pytest.mark.skipif`、`monkeypatch.setenv`、`knobs.refresh_knobs`、`torch.randn`、`ttgl.NVMMASharedLayout`、`gluon.nvidia.hopper.TensorDescriptor.from_tensor` 等另外 14 项。 该作用域涉及PyTorch 张量准备与校验、张量/描述符元数据、运行时驱动交互、kernel 启动编排。

#### Lines 2595-2599

```python
    if run_wrapper:
        result = run_in_process(test_deadlock_exempt_when_tma_signals, (device, False, monkeypatch, num_ctas))
        assert result.exc is None
        assert result.driver_stderr_output == ""
        return
```
- **EN:** Invokes `run_in_process` to execute the test logic. Validates behavior with 2 assertion(s). Branches on runtime or test conditions. Relevant themes: runtime driver interaction.
- **CN:** 调用 `run_in_process` 执行测试逻辑。 通过 2 个断言验证行为。 根据运行时或测试条件进行分支。 相关主题：运行时驱动交互。

#### Lines 2600-2602

```python
    monkeypatch.setenv("TRITON_INSTRUMENTATION_MODE", "consan")
    monkeypatch.setenv("CUDA_LAUNCH_BLOCKING", "1")
    knobs.refresh_knobs()
```
- **EN:** Invokes `monkeypatch.setenv`, `knobs.refresh_knobs` to execute the test logic. Relevant themes: kernel launch orchestration.
- **CN:** 调用 `monkeypatch.setenv`、`knobs.refresh_knobs` 执行测试逻辑。 相关主题：kernel 启动编排。

#### Lines 2603-2605

```python

    @gluon.jit
    def ws_default(input_desc, smem, bar):
```
- **EN:** Defines the helper function `ws_default`. Decorators: `gluon.jit`. Parameters: `input_desc`, `smem`, `bar`. Key calls include `mbarrier.expect`, `tma.async_load`, `mbarrier.wait`, `bar.index`, `smem.index`.
- **CN:** 定义辅助函数 `ws_default`。 装饰器：`gluon.jit`。 参数：`input_desc`、`smem`、`bar`。 关键调用包括 `mbarrier.expect`、`tma.async_load`、`mbarrier.wait`、`bar.index`、`smem.index`。

##### Lines 2606-2608

```python
        mbarrier.expect(bar.index(0), input_desc.nbytes_per_cta)
        tma.async_load(input_desc, [0, 0], bar.index(0), smem.index(0))
        mbarrier.wait(bar.index(0), phase=0)
```
- **EN:** Invokes `mbarrier.expect`, `bar.index`, `tma.async_load`, `smem.index`, `mbarrier.wait` to execute the test logic.
- **CN:** 调用 `mbarrier.expect`、`bar.index`、`tma.async_load`、`smem.index`、`mbarrier.wait` 执行测试逻辑。

#### Lines 2609-2611

```python

    @gluon.jit
    def ws_1(input_desc, smem, bar):
```
- **EN:** Defines the helper function `ws_1`. Decorators: `gluon.jit`. Parameters: `input_desc`, `smem`, `bar`. Key calls include `mbarrier.expect`, `tma.async_load`, `mbarrier.wait`, `bar.index`, `smem.index`.
- **CN:** 定义辅助函数 `ws_1`。 装饰器：`gluon.jit`。 参数：`input_desc`、`smem`、`bar`。 关键调用包括 `mbarrier.expect`、`tma.async_load`、`mbarrier.wait`、`bar.index`、`smem.index`。

##### Lines 2612-2614

```python
        mbarrier.expect(bar.index(1), input_desc.nbytes_per_cta)
        tma.async_load(input_desc, [0, 0], bar.index(1), smem.index(1))
        mbarrier.wait(bar.index(1), phase=0)
```
- **EN:** Invokes `mbarrier.expect`, `bar.index`, `tma.async_load`, `smem.index`, `mbarrier.wait` to execute the test logic.
- **CN:** 调用 `mbarrier.expect`、`bar.index`、`tma.async_load`、`smem.index`、`mbarrier.wait` 执行测试逻辑。

#### Lines 2615-2617

```python

    @gluon.jit
    def kernel(input_desc):
```
- **EN:** Defines the helper function `kernel`. Decorators: `gluon.jit`. Parameters: `input_desc`. Key calls include `ttgl.NVMMASharedLayout`, `ttgl.allocate_shared_memory`, `mbarrier.allocate_mbarrier`, `mbarrier.init`, `ttgl.warp_specialize`, `bar.index`, and 2 more. This scope touches layout transformation reasoning.
- **CN:** 定义辅助函数 `kernel`。 装饰器：`gluon.jit`。 参数：`input_desc`。 关键调用包括 `ttgl.NVMMASharedLayout`、`ttgl.allocate_shared_memory`、`mbarrier.allocate_mbarrier`、`mbarrier.init`、`ttgl.warp_specialize`、`bar.index` 等另外 2 项。 该作用域涉及布局变换推理。

##### Lines 2618-2627

```python
        shared_layout: ttgl.constexpr = ttgl.NVMMASharedLayout(swizzle_byte_width=128, element_bitwidth=16, rank=2,
                                                               cga_layout=default_cga_layout(ttgl.num_ctas(), 2))
        smem = ttgl.allocate_shared_memory(ttgl.float16, [2, XBLOCK, XBLOCK], shared_layout)
        bar = mbarrier.allocate_mbarrier(batch=2)
        mbarrier.init(bar.index(0), count=1)
        mbarrier.init(bar.index(1), count=1)
        ttgl.warp_specialize([
            (ws_default, (input_desc, smem, bar)),
            (ws_1, (input_desc, smem, bar)),
        ], [4], [32])
```
- **EN:** Prepares or updates state through `shared_layout`, `smem`, `bar`. Invokes `ttgl.NVMMASharedLayout`, `default_cga_layout`, `ttgl.num_ctas`, `ttgl.allocate_shared_memory`, `mbarrier.allocate_mbarrier`, `mbarrier.init`, and 2 more to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `shared_layout`、`smem`、`bar` 准备或更新状态。 调用 `ttgl.NVMMASharedLayout`、`default_cga_layout`、`ttgl.num_ctas`、`ttgl.allocate_shared_memory`、`mbarrier.allocate_mbarrier`、`mbarrier.init` 等另外 2 项 执行测试逻辑。 相关主题：布局变换推理。

#### Lines 2628-2633

```python

    input = torch.randn((XBLOCK, XBLOCK), device=device, dtype=torch.float16)
    shared_layout = ttgl.NVMMASharedLayout(swizzle_byte_width=128, element_bitwidth=16, rank=2,
                                           cga_layout=default_cga_layout(num_ctas, 2))
    input_desc = gluon.nvidia.hopper.TensorDescriptor.from_tensor(input, [XBLOCK.value, XBLOCK.value], shared_layout)
    kernel[(1, )](input_desc, num_warps=4, num_ctas=num_ctas)
```
- **EN:** Prepares or updates state through `input`, `shared_layout`, `input_desc`. Invokes `torch.randn`, `ttgl.NVMMASharedLayout`, `default_cga_layout`, `gluon.nvidia.hopper.TensorDescriptor.from_tensor` to execute the test logic. Relevant themes: PyTorch tensor setup and checks, tensor/descriptor metadata, layout transformation reasoning, random-data generation.
- **CN:** 通过 `input`、`shared_layout`、`input_desc` 准备或更新状态。 调用 `torch.randn`、`ttgl.NVMMASharedLayout`、`default_cga_layout`、`gluon.nvidia.hopper.TensorDescriptor.from_tensor` 执行测试逻辑。 相关主题：PyTorch 张量准备与校验、张量/描述符元数据、布局变换推理、随机数据生成。

### Lines 2634-2637

```python


@pytest.mark.skipif(not is_cuda() or torch.cuda.get_device_capability()[0] < 9, reason="Requires hopper")
def test_barrier_underflow(device, run_wrapper, monkeypatch, num_ctas):
```
- **EN:** Defines the test function `test_barrier_underflow`. Decorators: `pytest.mark.skipif(not is_cuda() or torch.cuda.get_device_capability()[0] < 9, reason='Requires hopper')`. Parameters: `device`, `run_wrapper`, `monkeypatch`, `num_ctas`. Nested definitions in this scope: `ws_default`, `ws_1`, `kernel`. Key calls include `pytest.mark.skipif`, `monkeypatch.setenv`, `knobs.refresh_knobs`, `run_in_process`, `assert_expected_cuda_failure`, `mbarrier.arrive`, and 7 more. This scope touches PyTorch tensor setup and checks, runtime driver interaction, kernel launch orchestration.
- **CN:** 定义测试函数 `test_barrier_underflow`。 装饰器：`pytest.mark.skipif(not is_cuda() or torch.cuda.get_device_capability()[0] < 9, reason='Requires hopper')`。 参数：`device`、`run_wrapper`、`monkeypatch`、`num_ctas`。 该作用域中的嵌套定义：`ws_default`、`ws_1`、`kernel`。 关键调用包括 `pytest.mark.skipif`、`monkeypatch.setenv`、`knobs.refresh_knobs`、`run_in_process`、`assert_expected_cuda_failure`、`mbarrier.arrive` 等另外 7 项。 该作用域涉及PyTorch 张量准备与校验、运行时驱动交互、kernel 启动编排。

#### Lines 2638-2642

```python
    if run_wrapper:
        result = run_in_process(test_barrier_underflow, (device, False, monkeypatch, num_ctas))
        assert_expected_cuda_failure(result.exc)
        assert "Barrier arrive underflow" in result.driver_stderr_output
        return
```
- **EN:** Invokes `run_in_process`, `assert_expected_cuda_failure` to execute the test logic. Validates behavior with 1 assertion(s). Branches on runtime or test conditions. Relevant themes: runtime driver interaction.
- **CN:** 调用 `run_in_process`、`assert_expected_cuda_failure` 执行测试逻辑。 通过 1 个断言验证行为。 根据运行时或测试条件进行分支。 相关主题：运行时驱动交互。

#### Lines 2643-2645

```python
    monkeypatch.setenv("TRITON_INSTRUMENTATION_MODE", "consan")
    monkeypatch.setenv("CUDA_LAUNCH_BLOCKING", "1")
    knobs.refresh_knobs()
```
- **EN:** Invokes `monkeypatch.setenv`, `knobs.refresh_knobs` to execute the test logic. Relevant themes: kernel launch orchestration.
- **CN:** 调用 `monkeypatch.setenv`、`knobs.refresh_knobs` 执行测试逻辑。 相关主题：kernel 启动编排。

#### Lines 2646-2648

```python

    @gluon.jit
    def ws_default(bar):
```
- **EN:** Defines the helper function `ws_default`. Decorators: `gluon.jit`. Parameters: `bar`. Key calls include `mbarrier.arrive`, `mbarrier.wait`, `bar.index`.
- **CN:** 定义辅助函数 `ws_default`。 装饰器：`gluon.jit`。 参数：`bar`。 关键调用包括 `mbarrier.arrive`、`mbarrier.wait`、`bar.index`。

##### Lines 2649-2650

```python
        mbarrier.arrive(bar.index(1), count=2)
        mbarrier.wait(bar.index(0), phase=0)
```
- **EN:** Invokes `mbarrier.arrive`, `bar.index`, `mbarrier.wait` to execute the test logic.
- **CN:** 调用 `mbarrier.arrive`、`bar.index`、`mbarrier.wait` 执行测试逻辑。

#### Lines 2651-2653

```python

    @gluon.jit
    def ws_1(bar):
```
- **EN:** Defines the helper function `ws_1`. Decorators: `gluon.jit`. Parameters: `bar`. Key calls include `mbarrier.wait`, `bar.index`.
- **CN:** 定义辅助函数 `ws_1`。 装饰器：`gluon.jit`。 参数：`bar`。 关键调用包括 `mbarrier.wait`、`bar.index`。

##### Lines 2654-2654

```python
        mbarrier.wait(bar.index(1), phase=0)
```
- **EN:** Invokes `mbarrier.wait`, `bar.index` to execute the test logic.
- **CN:** 调用 `mbarrier.wait`、`bar.index` 执行测试逻辑。

#### Lines 2655-2657

```python

    @gluon.jit
    def kernel():
```
- **EN:** Defines the helper function `kernel`. Decorators: `gluon.jit`. Key calls include `mbarrier.allocate_mbarrier`, `mbarrier.init`, `ttgl.warp_specialize`, `bar.index`.
- **CN:** 定义辅助函数 `kernel`。 装饰器：`gluon.jit`。 关键调用包括 `mbarrier.allocate_mbarrier`、`mbarrier.init`、`ttgl.warp_specialize`、`bar.index`。

##### Lines 2658-2664

```python
        bar = mbarrier.allocate_mbarrier(batch=2)
        mbarrier.init(bar.index(0), count=1)
        mbarrier.init(bar.index(1), count=1)
        ttgl.warp_specialize([
            (ws_default, (bar, )),
            (ws_1, (bar, )),
        ], [4], [32])
```
- **EN:** Prepares or updates state through `bar`. Invokes `mbarrier.allocate_mbarrier`, `mbarrier.init`, `bar.index`, `ttgl.warp_specialize` to execute the test logic.
- **CN:** 通过 `bar` 准备或更新状态。 调用 `mbarrier.allocate_mbarrier`、`mbarrier.init`、`bar.index`、`ttgl.warp_specialize` 执行测试逻辑。

#### Lines 2665-2666

```python

    kernel[(1, )](num_warps=4, num_ctas=num_ctas)
```
- **EN:** Carries supporting logic for the surrounding test or helper scope.
- **CN:** 为周围的测试或辅助作用域提供支撑逻辑。

### Lines 2667-2671

```python


@pytest.mark.skipif(not is_cuda() or torch.cuda.get_device_capability()[0] < 9, reason="Requires hopper or newer")
@pytest.mark.parametrize("WITH_INVALIDATE", [False, True])
def test_barrier_reinit_requires_invalidate(WITH_INVALIDATE, device, run_wrapper, monkeypatch, num_ctas):
```
- **EN:** Defines the test function `test_barrier_reinit_requires_invalidate`. Decorators: `pytest.mark.skipif(not is_cuda() or torch.cuda.get_device_capability()[0] < 9, reason='Requires hopper or newer')`, `pytest.mark.parametrize('WITH_INVALIDATE', [False, True])`. Parameters: `WITH_INVALIDATE`, `device`, `run_wrapper`, `monkeypatch`, `num_ctas`. Nested definitions in this scope: `kernel`. Key calls include `pytest.mark.skipif`, `pytest.mark.parametrize`, `monkeypatch.setenv`, `knobs.refresh_knobs`, `run_in_process`, `mbarrier.allocate_mbarrier`, and 6 more. This scope touches pytest parametrization, PyTorch tensor setup and checks, runtime driver interaction, kernel launch orchestration.
- **CN:** 定义测试函数 `test_barrier_reinit_requires_invalidate`。 装饰器：`pytest.mark.skipif(not is_cuda() or torch.cuda.get_device_capability()[0] < 9, reason='Requires hopper or newer')`、`pytest.mark.parametrize('WITH_INVALIDATE', [False, True])`。 参数：`WITH_INVALIDATE`、`device`、`run_wrapper`、`monkeypatch`、`num_ctas`。 该作用域中的嵌套定义：`kernel`。 关键调用包括 `pytest.mark.skipif`、`pytest.mark.parametrize`、`monkeypatch.setenv`、`knobs.refresh_knobs`、`run_in_process`、`mbarrier.allocate_mbarrier` 等另外 6 项。 该作用域涉及pytest 参数化、PyTorch 张量准备与校验、运行时驱动交互、kernel 启动编排。

#### Lines 2672-2681

```python
    if run_wrapper:
        result = run_in_process(test_barrier_reinit_requires_invalidate,
                                (WITH_INVALIDATE, device, False, monkeypatch, num_ctas))
        if WITH_INVALIDATE:
            assert result.exc is None
            assert result.driver_stderr_output == ""
        else:
            assert_expected_cuda_failure(result.exc)
            assert "Barrier re-initialized without prior invalidation" in result.driver_stderr_output
        return
```
- **EN:** Invokes `run_in_process`, `assert_expected_cuda_failure` to execute the test logic. Validates behavior with 3 assertion(s). Branches on runtime or test conditions. Relevant themes: runtime driver interaction.
- **CN:** 调用 `run_in_process`、`assert_expected_cuda_failure` 执行测试逻辑。 通过 3 个断言验证行为。 根据运行时或测试条件进行分支。 相关主题：运行时驱动交互。

#### Lines 2682-2684

```python
    monkeypatch.setenv("TRITON_INSTRUMENTATION_MODE", "consan")
    monkeypatch.setenv("CUDA_LAUNCH_BLOCKING", "1")
    knobs.refresh_knobs()
```
- **EN:** Invokes `monkeypatch.setenv`, `knobs.refresh_knobs` to execute the test logic. Relevant themes: kernel launch orchestration.
- **CN:** 调用 `monkeypatch.setenv`、`knobs.refresh_knobs` 执行测试逻辑。 相关主题：kernel 启动编排。

#### Lines 2685-2687

```python

    @gluon.jit
    def kernel(WITH_INVALIDATE: ttgl.constexpr):
```
- **EN:** Defines the helper function `kernel`. Decorators: `gluon.jit`. Parameters: `WITH_INVALIDATE`. Key calls include `mbarrier.allocate_mbarrier`, `mbarrier.init`, `mbarrier.invalidate`, `bar.index`.
- **CN:** 定义辅助函数 `kernel`。 装饰器：`gluon.jit`。 参数：`WITH_INVALIDATE`。 关键调用包括 `mbarrier.allocate_mbarrier`、`mbarrier.init`、`mbarrier.invalidate`、`bar.index`。

##### Lines 2688-2689

```python
        bar = mbarrier.allocate_mbarrier(batch=1)
        mbarrier.init(bar.index(0), count=1)
```
- **EN:** Prepares or updates state through `bar`. Invokes `mbarrier.allocate_mbarrier`, `mbarrier.init`, `bar.index` to execute the test logic.
- **CN:** 通过 `bar` 准备或更新状态。 调用 `mbarrier.allocate_mbarrier`、`mbarrier.init`、`bar.index` 执行测试逻辑。

##### Lines 2690-2691

```python
        if WITH_INVALIDATE:
            mbarrier.invalidate(bar.index(0))
```
- **EN:** Invokes `mbarrier.invalidate`, `bar.index` to execute the test logic. Branches on runtime or test conditions.
- **CN:** 调用 `mbarrier.invalidate`、`bar.index` 执行测试逻辑。 根据运行时或测试条件进行分支。

##### Lines 2692-2693

```python
        mbarrier.init(bar.index(0), count=1)
        mbarrier.invalidate(bar.index(0))
```
- **EN:** Invokes `mbarrier.init`, `bar.index`, `mbarrier.invalidate` to execute the test logic.
- **CN:** 调用 `mbarrier.init`、`bar.index`、`mbarrier.invalidate` 执行测试逻辑。

#### Lines 2694-2695

```python

    kernel[(1, )](WITH_INVALIDATE=WITH_INVALIDATE, num_warps=4, num_ctas=num_ctas)
```
- **EN:** Carries supporting logic for the surrounding test or helper scope.
- **CN:** 为周围的测试或辅助作用域提供支撑逻辑。

### Lines 2696-2700

```python


@pytest.mark.skipif(not is_cuda() or torch.cuda.get_device_capability()[0] < 9, reason="Requires hopper or newer")
@pytest.mark.parametrize("USE_KIND", ["wait", "arrive", "invalidate", "expect"])
def test_barrier_use_without_init(USE_KIND, device, run_wrapper, monkeypatch, num_ctas):
```
- **EN:** Defines the test function `test_barrier_use_without_init`. Decorators: `pytest.mark.skipif(not is_cuda() or torch.cuda.get_device_capability()[0] < 9, reason='Requires hopper or newer')`, `pytest.mark.parametrize('USE_KIND', ['wait', 'arrive', 'invalidate', 'expect'])`. Parameters: `USE_KIND`, `device`, `run_wrapper`, `monkeypatch`, `num_ctas`. Nested definitions in this scope: `kernel`. Key calls include `pytest.mark.skipif`, `pytest.mark.parametrize`, `monkeypatch.setenv`, `knobs.refresh_knobs`, `run_in_process`, `assert_expected_cuda_failure`, and 8 more. This scope touches pytest parametrization, PyTorch tensor setup and checks, runtime driver interaction, kernel launch orchestration.
- **CN:** 定义测试函数 `test_barrier_use_without_init`。 装饰器：`pytest.mark.skipif(not is_cuda() or torch.cuda.get_device_capability()[0] < 9, reason='Requires hopper or newer')`、`pytest.mark.parametrize('USE_KIND', ['wait', 'arrive', 'invalidate', 'expect'])`。 参数：`USE_KIND`、`device`、`run_wrapper`、`monkeypatch`、`num_ctas`。 该作用域中的嵌套定义：`kernel`。 关键调用包括 `pytest.mark.skipif`、`pytest.mark.parametrize`、`monkeypatch.setenv`、`knobs.refresh_knobs`、`run_in_process`、`assert_expected_cuda_failure` 等另外 8 项。 该作用域涉及pytest 参数化、PyTorch 张量准备与校验、运行时驱动交互、kernel 启动编排。

#### Lines 2701-2705

```python
    if run_wrapper:
        result = run_in_process(test_barrier_use_without_init, (USE_KIND, device, False, monkeypatch, num_ctas))
        assert_expected_cuda_failure(result.exc)
        assert "Barrier used before initialization or after invalidation" in result.driver_stderr_output
        return
```
- **EN:** Invokes `run_in_process`, `assert_expected_cuda_failure` to execute the test logic. Validates behavior with 1 assertion(s). Branches on runtime or test conditions. Relevant themes: runtime driver interaction.
- **CN:** 调用 `run_in_process`、`assert_expected_cuda_failure` 执行测试逻辑。 通过 1 个断言验证行为。 根据运行时或测试条件进行分支。 相关主题：运行时驱动交互。

#### Lines 2706-2708

```python
    monkeypatch.setenv("TRITON_INSTRUMENTATION_MODE", "consan")
    monkeypatch.setenv("CUDA_LAUNCH_BLOCKING", "1")
    knobs.refresh_knobs()
```
- **EN:** Invokes `monkeypatch.setenv`, `knobs.refresh_knobs` to execute the test logic. Relevant themes: kernel launch orchestration.
- **CN:** 调用 `monkeypatch.setenv`、`knobs.refresh_knobs` 执行测试逻辑。 相关主题：kernel 启动编排。

#### Lines 2709-2711

```python

    @gluon.jit
    def kernel(USE_KIND: ttgl.constexpr):
```
- **EN:** Defines the helper function `kernel`. Decorators: `gluon.jit`. Parameters: `USE_KIND`. Key calls include `mbarrier.allocate_mbarrier`, `mbarrier.wait`, `bar.index`, `mbarrier.arrive`, `mbarrier.invalidate`, `mbarrier.expect`.
- **CN:** 定义辅助函数 `kernel`。 装饰器：`gluon.jit`。 参数：`USE_KIND`。 关键调用包括 `mbarrier.allocate_mbarrier`、`mbarrier.wait`、`bar.index`、`mbarrier.arrive`、`mbarrier.invalidate`、`mbarrier.expect`。

##### Lines 2712-2712

```python
        bar = mbarrier.allocate_mbarrier(batch=1)
```
- **EN:** Prepares or updates state through `bar`. Invokes `mbarrier.allocate_mbarrier` to execute the test logic.
- **CN:** 通过 `bar` 准备或更新状态。 调用 `mbarrier.allocate_mbarrier` 执行测试逻辑。

##### Lines 2713-2720

```python
        if USE_KIND == "wait":
            mbarrier.wait(bar.index(0), phase=0)
        elif USE_KIND == "arrive":
            mbarrier.arrive(bar.index(0), count=1)
        elif USE_KIND == "invalidate":
            mbarrier.invalidate(bar.index(0))
        elif USE_KIND == "expect":
            mbarrier.expect(bar.index(0), XBLOCK * XBLOCK * ttgl.float16.primitive_bitwidth // 8)
```
- **EN:** Invokes `mbarrier.wait`, `bar.index`, `mbarrier.arrive`, `mbarrier.invalidate`, `mbarrier.expect` to execute the test logic. Branches on runtime or test conditions.
- **CN:** 调用 `mbarrier.wait`、`bar.index`、`mbarrier.arrive`、`mbarrier.invalidate`、`mbarrier.expect` 执行测试逻辑。 根据运行时或测试条件进行分支。

#### Lines 2721-2722

```python

    kernel[(1, )](USE_KIND=USE_KIND, num_warps=4, num_ctas=num_ctas)
```
- **EN:** Carries supporting logic for the surrounding test or helper scope.
- **CN:** 为周围的测试或辅助作用域提供支撑逻辑。

### Lines 2723-2727

```python


@pytest.mark.skipif(not is_cuda() or torch.cuda.get_device_capability()[0] < 9, reason="Requires hopper or newer")
@pytest.mark.parametrize("USE_KIND", ["wait", "arrive", "expect"])
def test_barrier_use_after_invalidate(USE_KIND, device, run_wrapper, monkeypatch, num_ctas):
```
- **EN:** Defines the test function `test_barrier_use_after_invalidate`. Decorators: `pytest.mark.skipif(not is_cuda() or torch.cuda.get_device_capability()[0] < 9, reason='Requires hopper or newer')`, `pytest.mark.parametrize('USE_KIND', ['wait', 'arrive', 'expect'])`. Parameters: `USE_KIND`, `device`, `run_wrapper`, `monkeypatch`, `num_ctas`. Nested definitions in this scope: `kernel`. Key calls include `pytest.mark.skipif`, `pytest.mark.parametrize`, `monkeypatch.setenv`, `knobs.refresh_knobs`, `run_in_process`, `assert_expected_cuda_failure`, and 9 more. This scope touches pytest parametrization, PyTorch tensor setup and checks, runtime driver interaction, kernel launch orchestration.
- **CN:** 定义测试函数 `test_barrier_use_after_invalidate`。 装饰器：`pytest.mark.skipif(not is_cuda() or torch.cuda.get_device_capability()[0] < 9, reason='Requires hopper or newer')`、`pytest.mark.parametrize('USE_KIND', ['wait', 'arrive', 'expect'])`。 参数：`USE_KIND`、`device`、`run_wrapper`、`monkeypatch`、`num_ctas`。 该作用域中的嵌套定义：`kernel`。 关键调用包括 `pytest.mark.skipif`、`pytest.mark.parametrize`、`monkeypatch.setenv`、`knobs.refresh_knobs`、`run_in_process`、`assert_expected_cuda_failure` 等另外 9 项。 该作用域涉及pytest 参数化、PyTorch 张量准备与校验、运行时驱动交互、kernel 启动编排。

#### Lines 2728-2732

```python
    if run_wrapper:
        result = run_in_process(test_barrier_use_after_invalidate, (USE_KIND, device, False, monkeypatch, num_ctas))
        assert_expected_cuda_failure(result.exc)
        assert "Barrier used before initialization or after invalidation" in result.driver_stderr_output
        return
```
- **EN:** Invokes `run_in_process`, `assert_expected_cuda_failure` to execute the test logic. Validates behavior with 1 assertion(s). Branches on runtime or test conditions. Relevant themes: runtime driver interaction.
- **CN:** 调用 `run_in_process`、`assert_expected_cuda_failure` 执行测试逻辑。 通过 1 个断言验证行为。 根据运行时或测试条件进行分支。 相关主题：运行时驱动交互。

#### Lines 2733-2735

```python
    monkeypatch.setenv("TRITON_INSTRUMENTATION_MODE", "consan")
    monkeypatch.setenv("CUDA_LAUNCH_BLOCKING", "1")
    knobs.refresh_knobs()
```
- **EN:** Invokes `monkeypatch.setenv`, `knobs.refresh_knobs` to execute the test logic. Relevant themes: kernel launch orchestration.
- **CN:** 调用 `monkeypatch.setenv`、`knobs.refresh_knobs` 执行测试逻辑。 相关主题：kernel 启动编排。

#### Lines 2736-2738

```python

    @gluon.jit
    def kernel(USE_KIND: ttgl.constexpr):
```
- **EN:** Defines the helper function `kernel`. Decorators: `gluon.jit`. Parameters: `USE_KIND`. Key calls include `mbarrier.allocate_mbarrier`, `mbarrier.init`, `mbarrier.invalidate`, `bar.index`, `mbarrier.wait`, `mbarrier.arrive`, and 1 more.
- **CN:** 定义辅助函数 `kernel`。 装饰器：`gluon.jit`。 参数：`USE_KIND`。 关键调用包括 `mbarrier.allocate_mbarrier`、`mbarrier.init`、`mbarrier.invalidate`、`bar.index`、`mbarrier.wait`、`mbarrier.arrive` 等另外 1 项。

##### Lines 2739-2741

```python
        bar = mbarrier.allocate_mbarrier(batch=1)
        mbarrier.init(bar.index(0), count=1)
        mbarrier.invalidate(bar.index(0))
```
- **EN:** Prepares or updates state through `bar`. Invokes `mbarrier.allocate_mbarrier`, `mbarrier.init`, `bar.index`, `mbarrier.invalidate` to execute the test logic.
- **CN:** 通过 `bar` 准备或更新状态。 调用 `mbarrier.allocate_mbarrier`、`mbarrier.init`、`bar.index`、`mbarrier.invalidate` 执行测试逻辑。

##### Lines 2742-2747

```python
        if USE_KIND == "wait":
            mbarrier.wait(bar.index(0), phase=0)
        elif USE_KIND == "arrive":
            mbarrier.arrive(bar.index(0), count=1)
        elif USE_KIND == "expect":
            mbarrier.expect(bar.index(0), XBLOCK * XBLOCK * ttgl.float16.primitive_bitwidth // 8)
```
- **EN:** Invokes `mbarrier.wait`, `bar.index`, `mbarrier.arrive`, `mbarrier.expect` to execute the test logic. Branches on runtime or test conditions.
- **CN:** 调用 `mbarrier.wait`、`bar.index`、`mbarrier.arrive`、`mbarrier.expect` 执行测试逻辑。 根据运行时或测试条件进行分支。

#### Lines 2748-2749

```python

    kernel[(1, )](USE_KIND=USE_KIND, num_warps=4, num_ctas=num_ctas)
```
- **EN:** Carries supporting logic for the surrounding test or helper scope.
- **CN:** 为周围的测试或辅助作用域提供支撑逻辑。

### Lines 2750-2755

```python


@pytest.mark.skipif(not is_cuda() or torch.cuda.get_device_capability()[0] < 9, reason="Requires hopper")
@pytest.mark.parametrize("MISSING_BAR", [True, False])
@pytest.mark.parametrize("OVERLAP", [True, False])
def test_aliasing_shared_visibility_outstanding_write(MISSING_BAR, OVERLAP, device, run_wrapper, monkeypatch, num_ctas):
```
- **EN:** Defines the test function `test_aliasing_shared_visibility_outstanding_write`. Decorators: `pytest.mark.skipif(not is_cuda() or torch.cuda.get_device_capability()[0] < 9, reason='Requires hopper')`, `pytest.mark.parametrize('MISSING_BAR', [True, False])`, `pytest.mark.parametrize('OVERLAP', [True, False])`. Parameters: `MISSING_BAR`, `OVERLAP`, `device`, `run_wrapper`, `monkeypatch`, `num_ctas`. Nested definitions in this scope: `writer`, `reader`, `kernel`. Key calls include `pytest.mark.skipif`, `pytest.mark.parametrize`, `monkeypatch.setenv`, `knobs.refresh_knobs`, `run_in_process`, `alias0.store`, and 18 more. This scope touches pytest parametrization, PyTorch tensor setup and checks, runtime driver interaction, kernel launch orchestration.
- **CN:** 定义测试函数 `test_aliasing_shared_visibility_outstanding_write`。 装饰器：`pytest.mark.skipif(not is_cuda() or torch.cuda.get_device_capability()[0] < 9, reason='Requires hopper')`、`pytest.mark.parametrize('MISSING_BAR', [True, False])`、`pytest.mark.parametrize('OVERLAP', [True, False])`。 参数：`MISSING_BAR`、`OVERLAP`、`device`、`run_wrapper`、`monkeypatch`、`num_ctas`。 该作用域中的嵌套定义：`writer`、`reader`、`kernel`。 关键调用包括 `pytest.mark.skipif`、`pytest.mark.parametrize`、`monkeypatch.setenv`、`knobs.refresh_knobs`、`run_in_process`、`alias0.store` 等另外 18 项。 该作用域涉及pytest 参数化、PyTorch 张量准备与校验、运行时驱动交互、kernel 启动编排。

#### Lines 2756-2767

```python
    if run_wrapper:
        result = run_in_process(test_aliasing_shared_visibility_outstanding_write,
                                (MISSING_BAR, OVERLAP, device, False, monkeypatch, num_ctas))
        if MISSING_BAR and OVERLAP:
            assert result.exc is not None
            assert_expected_cuda_failure(result.exc)
            # The race can be reported from either side depending on timing.
            assert "Buffer being accessed has outstanding" in result.driver_stderr_output
        else:
            assert result.exc is None
            assert result.driver_stderr_output == ""
        return
```
- **EN:** Invokes `run_in_process`, `assert_expected_cuda_failure` to execute the test logic. Validates behavior with 4 assertion(s). Branches on runtime or test conditions. Relevant themes: runtime driver interaction.
- **CN:** 调用 `run_in_process`、`assert_expected_cuda_failure` 执行测试逻辑。 通过 4 个断言验证行为。 根据运行时或测试条件进行分支。 相关主题：运行时驱动交互。

#### Lines 2768-2771

```python

    monkeypatch.setenv("TRITON_INSTRUMENTATION_MODE", "consan")
    monkeypatch.setenv("CUDA_LAUNCH_BLOCKING", "1")
    knobs.refresh_knobs()
```
- **EN:** Invokes `monkeypatch.setenv`, `knobs.refresh_knobs` to execute the test logic. Relevant themes: kernel launch orchestration.
- **CN:** 调用 `monkeypatch.setenv`、`knobs.refresh_knobs` 执行测试逻辑。 相关主题：kernel 启动编排。

#### Lines 2772-2775

```python

    @gluon.jit
    def writer(alias0: ttgl.constexpr, bar: ttgl.constexpr, OVERLAP: ttgl.constexpr, blocked_layout: ttgl.constexpr,
               blocked_layout_wide: ttgl.constexpr):
```
- **EN:** Defines the helper function `writer`. Decorators: `gluon.jit`. Parameters: `alias0`, `bar`, `OVERLAP`, `blocked_layout`, `blocked_layout_wide`. Key calls include `alias0.store`, `mbarrier.arrive`, `ttgl.num_ctas`, `ttgl.full`, `bar.index`. This scope touches layout transformation reasoning.
- **CN:** 定义辅助函数 `writer`。 装饰器：`gluon.jit`。 参数：`alias0`、`bar`、`OVERLAP`、`blocked_layout`、`blocked_layout_wide`。 关键调用包括 `alias0.store`、`mbarrier.arrive`、`ttgl.num_ctas`、`ttgl.full`、`bar.index`。 该作用域涉及布局变换推理。

##### Lines 2776-2776

```python
        block_m: ttgl.constexpr = XBLOCK * ttgl.num_ctas()
```
- **EN:** Prepares or updates state through `block_m`. Invokes `ttgl.num_ctas` to execute the test logic.
- **CN:** 通过 `block_m` 准备或更新状态。 调用 `ttgl.num_ctas` 执行测试逻辑。

##### Lines 2777-2780

```python
        if OVERLAP:
            vals = ttgl.full([block_m, XBLOCK * 2], 42.0, ttgl.float16, blocked_layout_wide)
        else:
            vals = ttgl.full([block_m, XBLOCK], 42.0, ttgl.float16, blocked_layout)
```
- **EN:** Invokes `ttgl.full` to execute the test logic. Branches on runtime or test conditions. Relevant themes: layout transformation reasoning.
- **CN:** 调用 `ttgl.full` 执行测试逻辑。 根据运行时或测试条件进行分支。 相关主题：布局变换推理。

##### Lines 2781-2782

```python
        alias0.store(vals)
        mbarrier.arrive(bar.index(0), count=1)
```
- **EN:** Invokes `alias0.store`, `mbarrier.arrive`, `bar.index` to execute the test logic.
- **CN:** 调用 `alias0.store`、`mbarrier.arrive`、`bar.index` 执行测试逻辑。

#### Lines 2783-2786

```python

    @gluon.jit
    def reader(alias1: ttgl.constexpr, dummy: ttgl.constexpr, bar: ttgl.constexpr, MISSING_BAR: ttgl.constexpr,
               blocked_layout: ttgl.constexpr):
```
- **EN:** Defines the helper function `reader`. Decorators: `gluon.jit`. Parameters: `alias1`, `dummy`, `bar`, `MISSING_BAR`, `blocked_layout`. Key calls include `alias1.load`, `dummy.store`, `mbarrier.wait`, `bar.index`. This scope touches layout transformation reasoning.
- **CN:** 定义辅助函数 `reader`。 装饰器：`gluon.jit`。 参数：`alias1`、`dummy`、`bar`、`MISSING_BAR`、`blocked_layout`。 关键调用包括 `alias1.load`、`dummy.store`、`mbarrier.wait`、`bar.index`。 该作用域涉及布局变换推理。

##### Lines 2787-2788

```python
        if not MISSING_BAR:
            mbarrier.wait(bar.index(0), phase=0)
```
- **EN:** Invokes `mbarrier.wait`, `bar.index` to execute the test logic. Branches on runtime or test conditions.
- **CN:** 调用 `mbarrier.wait`、`bar.index` 执行测试逻辑。 根据运行时或测试条件进行分支。

##### Lines 2789-2790

```python
        val = alias1.load(blocked_layout)
        dummy.store(val)  # keep the load alive
```
- **EN:** Prepares or updates state through `val`. Invokes `alias1.load`, `dummy.store` to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `val` 准备或更新状态。 调用 `alias1.load`、`dummy.store` 执行测试逻辑。 相关主题：布局变换推理。

#### Lines 2791-2793

```python

    @gluon.jit
    def kernel(MISSING_BAR: ttgl.constexpr, OVERLAP: ttgl.constexpr):
```
- **EN:** Defines the helper function `kernel`. Decorators: `gluon.jit`. Parameters: `MISSING_BAR`, `OVERLAP`. Key calls include `default_cga_layout`, `ttgl.SwizzledSharedLayout`, `ttgl.BlockedLayout`, `ttgl.allocate_shared_memory`, `mbarrier.allocate_mbarrier`, `mbarrier.init`, and 4 more. This scope touches layout transformation reasoning.
- **CN:** 定义辅助函数 `kernel`。 装饰器：`gluon.jit`。 参数：`MISSING_BAR`、`OVERLAP`。 关键调用包括 `default_cga_layout`、`ttgl.SwizzledSharedLayout`、`ttgl.BlockedLayout`、`ttgl.allocate_shared_memory`、`mbarrier.allocate_mbarrier`、`mbarrier.init` 等另外 4 项。 该作用域涉及布局变换推理。

##### Lines 2794-2811

```python
        block_m: ttgl.constexpr = XBLOCK * ttgl.num_ctas()
        cga_layout: ttgl.constexpr = default_cga_layout(ttgl.num_ctas(), 2)
        smem_layout: ttgl.constexpr = ttgl.SwizzledSharedLayout(vec=1, per_phase=1, max_phase=1, order=[0, 1],
                                                                cga_layout=cga_layout)
        blocked_layout: ttgl.constexpr = ttgl.BlockedLayout(size_per_thread=[1, XBLOCK], threads_per_warp=[32, 1],
                                                            warps_per_cta=[4, 1], order=[0, 1], cga_layout=cga_layout)
        blocked_layout_wide: ttgl.constexpr = ttgl.BlockedLayout(size_per_thread=[2, XBLOCK], threads_per_warp=[32, 1],
                                                                 warps_per_cta=[4, 1], order=[0,
                                                                                              1], cga_layout=cga_layout)
        smem = ttgl.allocate_shared_memory(ttgl.float16, [block_m, XBLOCK * 2], smem_layout)
        smem2 = ttgl.allocate_shared_memory(ttgl.float16, [block_m, XBLOCK], smem_layout)
        bar = mbarrier.allocate_mbarrier(batch=1)
        mbarrier.init(bar.index(0), count=1)
        alias0 = smem if OVERLAP else smem.slice(0, XBLOCK, dim=1)
        alias1 = smem.slice(XBLOCK, XBLOCK, dim=1)

        ttgl.warp_specialize([(writer, (alias0, bar, OVERLAP, blocked_layout, blocked_layout_wide)),
                              (reader, (alias1, smem2, bar, MISSING_BAR, blocked_layout))], [4], [32])
```
- **EN:** Prepares or updates state through `block_m`, `cga_layout`, `smem_layout`, `blocked_layout`, `blocked_layout_wide`, `smem`, `smem2`, `bar`, and 2 more. Invokes `ttgl.num_ctas`, `default_cga_layout`, `ttgl.SwizzledSharedLayout`, `ttgl.BlockedLayout`, `ttgl.allocate_shared_memory`, `mbarrier.allocate_mbarrier`, and 4 more to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `block_m`、`cga_layout`、`smem_layout`、`blocked_layout`、`blocked_layout_wide`、`smem`、`smem2`、`bar` 等另外 2 项 准备或更新状态。 调用 `ttgl.num_ctas`、`default_cga_layout`、`ttgl.SwizzledSharedLayout`、`ttgl.BlockedLayout`、`ttgl.allocate_shared_memory`、`mbarrier.allocate_mbarrier` 等另外 4 项 执行测试逻辑。 相关主题：布局变换推理。

#### Lines 2812-2813

```python

    kernel[(1, )](MISSING_BAR=MISSING_BAR, OVERLAP=OVERLAP, num_warps=4, num_ctas=num_ctas)
```
- **EN:** Carries supporting logic for the surrounding test or helper scope.
- **CN:** 为周围的测试或辅助作用域提供支撑逻辑。

### Lines 2814-2817

```python


@pytest.mark.skipif(not is_cuda() or torch.cuda.get_device_capability()[0] < 9, reason="Requires hopper")
def test_aliasing_tma_overwrite_clears_stale_write_visibility(device, run_wrapper, monkeypatch, num_ctas):
```
- **EN:** Defines the test function `test_aliasing_tma_overwrite_clears_stale_write_visibility`. Decorators: `pytest.mark.skipif(not is_cuda() or torch.cuda.get_device_capability()[0] < 9, reason='Requires hopper')`. Parameters: `device`, `run_wrapper`, `monkeypatch`, `num_ctas`. Nested definitions in this scope: `writer`, `reader`, `kernel`. Key calls include `pytest.mark.skipif`, `monkeypatch.setenv`, `knobs.refresh_knobs`, `torch.randn`, `ttgl.NVMMASharedLayout`, `gluon.nvidia.hopper.TensorDescriptor.from_tensor`, and 19 more. This scope touches PyTorch tensor setup and checks, tensor/descriptor metadata, runtime driver interaction, kernel launch orchestration.
- **CN:** 定义测试函数 `test_aliasing_tma_overwrite_clears_stale_write_visibility`。 装饰器：`pytest.mark.skipif(not is_cuda() or torch.cuda.get_device_capability()[0] < 9, reason='Requires hopper')`。 参数：`device`、`run_wrapper`、`monkeypatch`、`num_ctas`。 该作用域中的嵌套定义：`writer`、`reader`、`kernel`。 关键调用包括 `pytest.mark.skipif`、`monkeypatch.setenv`、`knobs.refresh_knobs`、`torch.randn`、`ttgl.NVMMASharedLayout`、`gluon.nvidia.hopper.TensorDescriptor.from_tensor` 等另外 19 项。 该作用域涉及PyTorch 张量准备与校验、张量/描述符元数据、运行时驱动交互、kernel 启动编排。

#### Lines 2818-2823

```python
    if run_wrapper:
        result = run_in_process(test_aliasing_tma_overwrite_clears_stale_write_visibility,
                                (device, False, monkeypatch, num_ctas))
        assert result.exc is None
        assert result.driver_stderr_output == ""
        return
```
- **EN:** Invokes `run_in_process` to execute the test logic. Validates behavior with 2 assertion(s). Branches on runtime or test conditions. Relevant themes: runtime driver interaction.
- **CN:** 调用 `run_in_process` 执行测试逻辑。 通过 2 个断言验证行为。 根据运行时或测试条件进行分支。 相关主题：运行时驱动交互。

#### Lines 2824-2827

```python

    monkeypatch.setenv("TRITON_INSTRUMENTATION_MODE", "consan")
    monkeypatch.setenv("CUDA_LAUNCH_BLOCKING", "1")
    knobs.refresh_knobs()
```
- **EN:** Invokes `monkeypatch.setenv`, `knobs.refresh_knobs` to execute the test logic. Relevant themes: kernel launch orchestration.
- **CN:** 调用 `monkeypatch.setenv`、`knobs.refresh_knobs` 执行测试逻辑。 相关主题：kernel 启动编排。

#### Lines 2828-2831

```python

    @gluon.jit
    def writer(full: ttgl.constexpr, tail: ttgl.constexpr, input_desc, bar: ttgl.constexpr,
               blocked_layout_wide: ttgl.constexpr):
```
- **EN:** Defines the helper function `writer`. Decorators: `gluon.jit`. Parameters: `full`, `tail`, `input_desc`, `bar`, `blocked_layout_wide`. Key calls include `ttgl.full`, `full.store`, `mbarrier.expect`, `tma.async_load`, `ttgl.num_ctas`, `bar.index`. This scope touches layout transformation reasoning.
- **CN:** 定义辅助函数 `writer`。 装饰器：`gluon.jit`。 参数：`full`、`tail`、`input_desc`、`bar`、`blocked_layout_wide`。 关键调用包括 `ttgl.full`、`full.store`、`mbarrier.expect`、`tma.async_load`、`ttgl.num_ctas`、`bar.index`。 该作用域涉及布局变换推理。

##### Lines 2832-2836

```python
        block_m: ttgl.constexpr = XBLOCK * ttgl.num_ctas()
        vals = ttgl.full([block_m, XBLOCK * 2], 42.0, ttgl.float16, blocked_layout_wide)
        full.store(vals)
        mbarrier.expect(bar.index(0), input_desc.nbytes_per_cta)
        tma.async_load(input_desc, [0, 0], bar.index(0), tail)
```
- **EN:** Prepares or updates state through `block_m`, `vals`. Invokes `ttgl.num_ctas`, `ttgl.full`, `full.store`, `mbarrier.expect`, `bar.index`, `tma.async_load` to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `block_m`、`vals` 准备或更新状态。 调用 `ttgl.num_ctas`、`ttgl.full`、`full.store`、`mbarrier.expect`、`bar.index`、`tma.async_load` 执行测试逻辑。 相关主题：布局变换推理。

#### Lines 2837-2839

```python

    @gluon.jit
    def reader(tail: ttgl.constexpr, dummy: ttgl.constexpr, bar: ttgl.constexpr, blocked_layout: ttgl.constexpr):
```
- **EN:** Defines the helper function `reader`. Decorators: `gluon.jit`. Parameters: `tail`, `dummy`, `bar`, `blocked_layout`. Key calls include `mbarrier.wait`, `tail.load`, `dummy.store`, `bar.index`. This scope touches layout transformation reasoning.
- **CN:** 定义辅助函数 `reader`。 装饰器：`gluon.jit`。 参数：`tail`、`dummy`、`bar`、`blocked_layout`。 关键调用包括 `mbarrier.wait`、`tail.load`、`dummy.store`、`bar.index`。 该作用域涉及布局变换推理。

##### Lines 2840-2842

```python
        mbarrier.wait(bar.index(0), phase=0)
        val = tail.load(blocked_layout)
        dummy.store(val)
```
- **EN:** Prepares or updates state through `val`. Invokes `mbarrier.wait`, `bar.index`, `tail.load`, `dummy.store` to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `val` 准备或更新状态。 调用 `mbarrier.wait`、`bar.index`、`tail.load`、`dummy.store` 执行测试逻辑。 相关主题：布局变换推理。

#### Lines 2843-2845

```python

    @gluon.jit
    def kernel(input_desc):
```
- **EN:** Defines the helper function `kernel`. Decorators: `gluon.jit`. Parameters: `input_desc`. Key calls include `default_cga_layout`, `ttgl.NVMMASharedLayout`, `ttgl.BlockedLayout`, `ttgl.allocate_shared_memory`, `full.slice`, `mbarrier.allocate_mbarrier`, and 4 more. This scope touches layout transformation reasoning.
- **CN:** 定义辅助函数 `kernel`。 装饰器：`gluon.jit`。 参数：`input_desc`。 关键调用包括 `default_cga_layout`、`ttgl.NVMMASharedLayout`、`ttgl.BlockedLayout`、`ttgl.allocate_shared_memory`、`full.slice`、`mbarrier.allocate_mbarrier` 等另外 4 项。 该作用域涉及布局变换推理。

##### Lines 2846-2861

```python
        block_m: ttgl.constexpr = XBLOCK * ttgl.num_ctas()
        cga_layout: ttgl.constexpr = default_cga_layout(ttgl.num_ctas(), 2)
        smem_layout: ttgl.constexpr = ttgl.NVMMASharedLayout(swizzle_byte_width=128, element_bitwidth=16, rank=2,
                                                             cga_layout=cga_layout)
        blocked_layout: ttgl.constexpr = ttgl.BlockedLayout(size_per_thread=[1, XBLOCK], threads_per_warp=[32, 1],
                                                            warps_per_cta=[4, 1], order=[0, 1], cga_layout=cga_layout)
        blocked_layout_wide: ttgl.constexpr = ttgl.BlockedLayout(size_per_thread=[2, XBLOCK], threads_per_warp=[32, 1],
                                                                 warps_per_cta=[4, 1], order=[0,
                                                                                              1], cga_layout=cga_layout)
        full = ttgl.allocate_shared_memory(ttgl.float16, [block_m, XBLOCK * 2], smem_layout)
        tail = full.slice(XBLOCK, XBLOCK, dim=1)
        dummy = ttgl.allocate_shared_memory(ttgl.float16, [block_m, XBLOCK], smem_layout)
        bar = mbarrier.allocate_mbarrier(batch=1)
        mbarrier.init(bar.index(0), count=1)
        ttgl.warp_specialize([(writer, (full, tail, input_desc, bar, blocked_layout_wide)),
                              (reader, (tail, dummy, bar, blocked_layout))], [4], [32])
```
- **EN:** Prepares or updates state through `block_m`, `cga_layout`, `smem_layout`, `blocked_layout`, `blocked_layout_wide`, `full`, `tail`, `dummy`, and 1 more. Invokes `ttgl.num_ctas`, `default_cga_layout`, `ttgl.NVMMASharedLayout`, `ttgl.BlockedLayout`, `ttgl.allocate_shared_memory`, `full.slice`, and 4 more to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `block_m`、`cga_layout`、`smem_layout`、`blocked_layout`、`blocked_layout_wide`、`full`、`tail`、`dummy` 等另外 1 项 准备或更新状态。 调用 `ttgl.num_ctas`、`default_cga_layout`、`ttgl.NVMMASharedLayout`、`ttgl.BlockedLayout`、`ttgl.allocate_shared_memory`、`full.slice` 等另外 4 项 执行测试逻辑。 相关主题：布局变换推理。

#### Lines 2862-2868

```python

    block_m = XBLOCK.value * num_ctas
    input = torch.randn((block_m, XBLOCK.value), device=device, dtype=torch.float16)
    shared_layout = ttgl.NVMMASharedLayout(swizzle_byte_width=128, element_bitwidth=16, rank=2,
                                           cga_layout=default_cga_layout(num_ctas, 2))
    input_desc = gluon.nvidia.hopper.TensorDescriptor.from_tensor(input, [block_m, XBLOCK.value], shared_layout)
    kernel[(1, )](input_desc, num_warps=4, num_ctas=num_ctas)
```
- **EN:** Prepares or updates state through `block_m`, `input`, `shared_layout`, `input_desc`. Invokes `torch.randn`, `ttgl.NVMMASharedLayout`, `default_cga_layout`, `gluon.nvidia.hopper.TensorDescriptor.from_tensor` to execute the test logic. Relevant themes: PyTorch tensor setup and checks, tensor/descriptor metadata, layout transformation reasoning, random-data generation.
- **CN:** 通过 `block_m`、`input`、`shared_layout`、`input_desc` 准备或更新状态。 调用 `torch.randn`、`ttgl.NVMMASharedLayout`、`default_cga_layout`、`gluon.nvidia.hopper.TensorDescriptor.from_tensor` 执行测试逻辑。 相关主题：PyTorch 张量准备与校验、张量/描述符元数据、布局变换推理、随机数据生成。

### Lines 2869-2873

```python


@pytest.mark.skipif(not is_cuda() or torch.cuda.get_device_capability()[0] < 10, reason="Requires blackwell or newer")
@pytest.mark.parametrize("FAILURE", [True, False])
def test_aliasing_tensor_visibility_outstanding_read(FAILURE, device, run_wrapper, monkeypatch, num_ctas):
```
- **EN:** Defines the test function `test_aliasing_tensor_visibility_outstanding_read`. Decorators: `pytest.mark.skipif(not is_cuda() or torch.cuda.get_device_capability()[0] < 10, reason='Requires blackwell or newer')`, `pytest.mark.parametrize('FAILURE', [True, False])`. Parameters: `FAILURE`, `device`, `run_wrapper`, `monkeypatch`, `num_ctas`. Nested definitions in this scope: `reader`, `writer`, `kernel`. Key calls include `pytest.mark.skipif`, `pytest.mark.parametrize`, `monkeypatch.setenv`, `knobs.refresh_knobs`, `run_in_process`, `alias0.load`, and 20 more. This scope touches pytest parametrization, PyTorch tensor setup and checks, runtime driver interaction, kernel launch orchestration.
- **CN:** 定义测试函数 `test_aliasing_tensor_visibility_outstanding_read`。 装饰器：`pytest.mark.skipif(not is_cuda() or torch.cuda.get_device_capability()[0] < 10, reason='Requires blackwell or newer')`、`pytest.mark.parametrize('FAILURE', [True, False])`。 参数：`FAILURE`、`device`、`run_wrapper`、`monkeypatch`、`num_ctas`。 该作用域中的嵌套定义：`reader`、`writer`、`kernel`。 关键调用包括 `pytest.mark.skipif`、`pytest.mark.parametrize`、`monkeypatch.setenv`、`knobs.refresh_knobs`、`run_in_process`、`alias0.load` 等另外 20 项。 该作用域涉及pytest 参数化、PyTorch 张量准备与校验、运行时驱动交互、kernel 启动编排。

#### Lines 2874-2885

```python
    if run_wrapper:
        result = run_in_process(test_aliasing_tensor_visibility_outstanding_read,
                                (FAILURE, device, False, monkeypatch, num_ctas))
        if FAILURE:
            assert result.exc is not None
            assert_expected_cuda_failure(result.exc)
            # outstanding reads or writes depends on the timing of the operations.
            assert "Buffer being accessed has outstanding" in result.driver_stderr_output
        else:
            assert result.exc is None
            assert result.driver_stderr_output == ""
        return
```
- **EN:** Invokes `run_in_process`, `assert_expected_cuda_failure` to execute the test logic. Validates behavior with 4 assertion(s). Branches on runtime or test conditions. Relevant themes: runtime driver interaction.
- **CN:** 调用 `run_in_process`、`assert_expected_cuda_failure` 执行测试逻辑。 通过 4 个断言验证行为。 根据运行时或测试条件进行分支。 相关主题：运行时驱动交互。

#### Lines 2886-2889

```python

    monkeypatch.setenv("TRITON_INSTRUMENTATION_MODE", "consan")
    monkeypatch.setenv("CUDA_LAUNCH_BLOCKING", "1")
    knobs.refresh_knobs()
```
- **EN:** Invokes `monkeypatch.setenv`, `knobs.refresh_knobs` to execute the test logic. Relevant themes: kernel launch orchestration.
- **CN:** 调用 `monkeypatch.setenv`、`knobs.refresh_knobs` 执行测试逻辑。 相关主题：kernel 启动编排。

#### Lines 2890-2892

```python

    @gluon.jit
    def reader(alias0: ttgl.constexpr, smem: ttgl.constexpr, blocked_layout_read: ttgl.constexpr, bar: ttgl.constexpr):
```
- **EN:** Defines the helper function `reader`. Decorators: `gluon.jit`. Parameters: `alias0`, `smem`, `blocked_layout_read`, `bar`. Key calls include `alias0.load`, `smem.store`, `mbarrier.arrive`, `bar.index`. This scope touches layout transformation reasoning.
- **CN:** 定义辅助函数 `reader`。 装饰器：`gluon.jit`。 参数：`alias0`、`smem`、`blocked_layout_read`、`bar`。 关键调用包括 `alias0.load`、`smem.store`、`mbarrier.arrive`、`bar.index`。 该作用域涉及布局变换推理。

##### Lines 2893-2895

```python
        val = alias0.load(blocked_layout_read)
        smem.store(val)  # keep the load alive
        mbarrier.arrive(bar.index(0), count=1)
```
- **EN:** Prepares or updates state through `val`. Invokes `alias0.load`, `smem.store`, `mbarrier.arrive`, `bar.index` to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `val` 准备或更新状态。 调用 `alias0.load`、`smem.store`、`mbarrier.arrive`、`bar.index` 执行测试逻辑。 相关主题：布局变换推理。

#### Lines 2896-2899

```python

    @gluon.jit
    def writer(alias1: ttgl.constexpr, bar: ttgl.constexpr, FAILURE: ttgl.constexpr,
               blocked_layout_write: ttgl.constexpr):
```
- **EN:** Defines the helper function `writer`. Decorators: `gluon.jit`. Parameters: `alias1`, `bar`, `FAILURE`, `blocked_layout_write`. Key calls include `alias1.store`, `ttgl.num_ctas`, `mbarrier.wait`, `ttgl.zeros`, `bar.index`. This scope touches layout transformation reasoning.
- **CN:** 定义辅助函数 `writer`。 装饰器：`gluon.jit`。 参数：`alias1`、`bar`、`FAILURE`、`blocked_layout_write`。 关键调用包括 `alias1.store`、`ttgl.num_ctas`、`mbarrier.wait`、`ttgl.zeros`、`bar.index`。 该作用域涉及布局变换推理。

##### Lines 2900-2900

```python
        block_m: ttgl.constexpr = XBLOCK * ttgl.num_ctas()
```
- **EN:** Prepares or updates state through `block_m`. Invokes `ttgl.num_ctas` to execute the test logic.
- **CN:** 通过 `block_m` 准备或更新状态。 调用 `ttgl.num_ctas` 执行测试逻辑。

##### Lines 2901-2902

```python
        if not FAILURE:
            mbarrier.wait(bar.index(0), phase=0)
```
- **EN:** Invokes `mbarrier.wait`, `bar.index` to execute the test logic. Branches on runtime or test conditions.
- **CN:** 调用 `mbarrier.wait`、`bar.index` 执行测试逻辑。 根据运行时或测试条件进行分支。

##### Lines 2903-2903

```python
        alias1.store(ttgl.zeros([block_m, XBLOCK // 2], ttgl.float32, blocked_layout_write))
```
- **EN:** Invokes `alias1.store`, `ttgl.zeros` to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 调用 `alias1.store`、`ttgl.zeros` 执行测试逻辑。 相关主题：布局变换推理。

#### Lines 2904-2906

```python

    @gluon.jit
    def kernel(FAILURE: ttgl.constexpr):
```
- **EN:** Defines the helper function `kernel`. Decorators: `gluon.jit`. Parameters: `FAILURE`. Key calls include `default_cga_layout`, `ttgl.SwizzledSharedLayout`, `ttgl.BlockedLayout`, `ttgl.allocate_shared_memory`, `blackwell.TensorMemoryLayout`, `blackwell.allocate_tensor_memory`, and 6 more. This scope touches layout transformation reasoning.
- **CN:** 定义辅助函数 `kernel`。 装饰器：`gluon.jit`。 参数：`FAILURE`。 关键调用包括 `default_cga_layout`、`ttgl.SwizzledSharedLayout`、`ttgl.BlockedLayout`、`ttgl.allocate_shared_memory`、`blackwell.TensorMemoryLayout`、`blackwell.allocate_tensor_memory` 等另外 6 项。 该作用域涉及布局变换推理。

##### Lines 2907-2928

```python
        block_m: ttgl.constexpr = XBLOCK * ttgl.num_ctas()
        cga_layout: ttgl.constexpr = default_cga_layout(ttgl.num_ctas(), 2)
        smem_layout: ttgl.constexpr = ttgl.SwizzledSharedLayout(vec=1, per_phase=1, max_phase=1, order=[0, 1],
                                                                cga_layout=cga_layout)
        blocked_layout_read: ttgl.constexpr = ttgl.BlockedLayout(size_per_thread=[1, XBLOCK], threads_per_warp=[32, 1],
                                                                 warps_per_cta=[4, 1], order=[0,
                                                                                              1], cga_layout=cga_layout)
        blocked_layout_write: ttgl.constexpr = ttgl.BlockedLayout(size_per_thread=[1, XBLOCK // 2],
                                                                  threads_per_warp=[32, 1], warps_per_cta=[4, 1],
                                                                  order=[0, 1], cga_layout=cga_layout)
        smem = ttgl.allocate_shared_memory(ttgl.float32, [block_m, XBLOCK], smem_layout)
        tmem_layout: ttgl.constexpr = blackwell.TensorMemoryLayout([XBLOCK, XBLOCK * 2], col_stride=1,
                                                                   cga_layout=cga_layout)
        tmem = blackwell.allocate_tensor_memory(ttgl.float32, [block_m, XBLOCK * 2], tmem_layout)
        bar = mbarrier.allocate_mbarrier(batch=1)
        mbarrier.init(bar.index(0), count=1)
        alias0 = tmem.slice(0, XBLOCK)
        # Second half of the tmem
        alias1 = tmem.slice(XBLOCK // 2, XBLOCK // 2)

        ttgl.warp_specialize([(reader, (alias0, smem, blocked_layout_read, bar)),
                              (writer, (alias1, bar, FAILURE, blocked_layout_write))], [4], [32])
```
- **EN:** Prepares or updates state through `block_m`, `cga_layout`, `smem_layout`, `blocked_layout_read`, `blocked_layout_write`, `smem`, `tmem_layout`, `tmem`, and 3 more. Invokes `ttgl.num_ctas`, `default_cga_layout`, `ttgl.SwizzledSharedLayout`, `ttgl.BlockedLayout`, `ttgl.allocate_shared_memory`, `blackwell.TensorMemoryLayout`, and 6 more to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `block_m`、`cga_layout`、`smem_layout`、`blocked_layout_read`、`blocked_layout_write`、`smem`、`tmem_layout`、`tmem` 等另外 3 项 准备或更新状态。 调用 `ttgl.num_ctas`、`default_cga_layout`、`ttgl.SwizzledSharedLayout`、`ttgl.BlockedLayout`、`ttgl.allocate_shared_memory`、`blackwell.TensorMemoryLayout` 等另外 6 项 执行测试逻辑。 相关主题：布局变换推理。

#### Lines 2929-2930

```python

    kernel[(1, )](FAILURE=FAILURE, num_warps=4, num_ctas=num_ctas)
```
- **EN:** Carries supporting logic for the surrounding test or helper scope.
- **CN:** 为周围的测试或辅助作用域提供支撑逻辑。

### Lines 2931-2936

```python


@pytest.mark.skipif(not is_cuda() or torch.cuda.get_device_capability()[0] < 9, reason="Requires hopper")
@pytest.mark.parametrize("MISSING_WAIT", [True, False])
@pytest.mark.parametrize("OVERLAP", [True, False])
def test_aliasing_commit_tracking(MISSING_WAIT, OVERLAP, device, run_wrapper, monkeypatch, num_ctas):
```
- **EN:** Defines the test function `test_aliasing_commit_tracking`. Decorators: `pytest.mark.skipif(not is_cuda() or torch.cuda.get_device_capability()[0] < 9, reason='Requires hopper')`, `pytest.mark.parametrize('MISSING_WAIT', [True, False])`, `pytest.mark.parametrize('OVERLAP', [True, False])`. Parameters: `MISSING_WAIT`, `OVERLAP`, `device`, `run_wrapper`, `monkeypatch`, `num_ctas`. Nested definitions in this scope: `producer`, `consumer`, `kernel`. Key calls include `pytest.mark.skipif`, `pytest.mark.parametrize`, `monkeypatch.setenv`, `knobs.refresh_knobs`, `torch.randn`, `run_in_process`, and 22 more. This scope touches pytest parametrization, PyTorch tensor setup and checks, runtime driver interaction, kernel launch orchestration.
- **CN:** 定义测试函数 `test_aliasing_commit_tracking`。 装饰器：`pytest.mark.skipif(not is_cuda() or torch.cuda.get_device_capability()[0] < 9, reason='Requires hopper')`、`pytest.mark.parametrize('MISSING_WAIT', [True, False])`、`pytest.mark.parametrize('OVERLAP', [True, False])`。 参数：`MISSING_WAIT`、`OVERLAP`、`device`、`run_wrapper`、`monkeypatch`、`num_ctas`。 该作用域中的嵌套定义：`producer`、`consumer`、`kernel`。 关键调用包括 `pytest.mark.skipif`、`pytest.mark.parametrize`、`monkeypatch.setenv`、`knobs.refresh_knobs`、`torch.randn`、`run_in_process` 等另外 22 项。 该作用域涉及pytest 参数化、PyTorch 张量准备与校验、运行时驱动交互、kernel 启动编排。

#### Lines 2937-2947

```python
    if run_wrapper:
        result = run_in_process(test_aliasing_commit_tracking,
                                (MISSING_WAIT, OVERLAP, device, False, monkeypatch, num_ctas))
        if MISSING_WAIT and OVERLAP:
            assert result.exc is not None
            assert_expected_cuda_failure(result.exc)
            assert "Accessing buffer with pending access. Pending access type: async_copy_global_to_shared" in result.driver_stderr_output
        else:
            assert result.exc is None
            assert result.driver_stderr_output == ""
        return
```
- **EN:** Invokes `run_in_process`, `assert_expected_cuda_failure` to execute the test logic. Validates behavior with 4 assertion(s). Branches on runtime or test conditions. Relevant themes: runtime driver interaction.
- **CN:** 调用 `run_in_process`、`assert_expected_cuda_failure` 执行测试逻辑。 通过 4 个断言验证行为。 根据运行时或测试条件进行分支。 相关主题：运行时驱动交互。

#### Lines 2948-2951

```python

    monkeypatch.setenv("TRITON_INSTRUMENTATION_MODE", "consan")
    monkeypatch.setenv("CUDA_LAUNCH_BLOCKING", "1")
    knobs.refresh_knobs()
```
- **EN:** Invokes `monkeypatch.setenv`, `knobs.refresh_knobs` to execute the test logic. Relevant themes: kernel launch orchestration.
- **CN:** 调用 `monkeypatch.setenv`、`knobs.refresh_knobs` 执行测试逻辑。 相关主题：kernel 启动编排。

#### Lines 2952-2955

```python

    @gluon.jit
    def producer(input, alias0, bar, MISSING_WAIT: ttgl.constexpr, OVERLAP: ttgl.constexpr,
                 blocked_layout: ttgl.constexpr, blocked_layout_wide: ttgl.constexpr):
```
- **EN:** Defines the helper function `producer`. Decorators: `gluon.jit`. Parameters: `input`, `alias0`, `bar`, `MISSING_WAIT`, `OVERLAP`, `blocked_layout`, `blocked_layout_wide`. Key calls include `ampere.async_copy.async_copy_global_to_shared`, `ampere.async_copy.commit_group`, `mbarrier.arrive`, `ttgl.num_ctas`, `ttgl.arange`, `ampere.async_copy.wait_group`, and 2 more. This scope touches layout transformation reasoning.
- **CN:** 定义辅助函数 `producer`。 装饰器：`gluon.jit`。 参数：`input`、`alias0`、`bar`、`MISSING_WAIT`、`OVERLAP`、`blocked_layout`、`blocked_layout_wide`。 关键调用包括 `ampere.async_copy.async_copy_global_to_shared`、`ampere.async_copy.commit_group`、`mbarrier.arrive`、`ttgl.num_ctas`、`ttgl.arange`、`ampere.async_copy.wait_group` 等另外 2 项。 该作用域涉及布局变换推理。

##### Lines 2956-2963

```python
        block_m: ttgl.constexpr = XBLOCK * ttgl.num_ctas()
        layout: ttgl.constexpr = blocked_layout_wide if OVERLAP else blocked_layout
        SIZE_N: ttgl.constexpr = XBLOCK * 2 if OVERLAP else XBLOCK
        offs_m = ttgl.arange(0, block_m, layout=ttgl.SliceLayout(dim=1, parent=layout))[:, None]
        offs_n = ttgl.arange(0, SIZE_N, layout=ttgl.SliceLayout(dim=0, parent=layout))[None, :]
        offs = offs_m * (XBLOCK * 2) + offs_n
        ampere.async_copy.async_copy_global_to_shared(alias0, input + offs)
        ampere.async_copy.commit_group()
```
- **EN:** Prepares or updates state through `block_m`, `layout`, `SIZE_N`, `offs_m`, `offs_n`, `offs`. Invokes `ttgl.num_ctas`, `ttgl.arange`, `ttgl.SliceLayout`, `ampere.async_copy.async_copy_global_to_shared`, `ampere.async_copy.commit_group` to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `block_m`、`layout`、`SIZE_N`、`offs_m`、`offs_n`、`offs` 准备或更新状态。 调用 `ttgl.num_ctas`、`ttgl.arange`、`ttgl.SliceLayout`、`ampere.async_copy.async_copy_global_to_shared`、`ampere.async_copy.commit_group` 执行测试逻辑。 相关主题：布局变换推理。

##### Lines 2964-2965

```python
        if not MISSING_WAIT:
            ampere.async_copy.wait_group(0)
```
- **EN:** Invokes `ampere.async_copy.wait_group` to execute the test logic. Branches on runtime or test conditions.
- **CN:** 调用 `ampere.async_copy.wait_group` 执行测试逻辑。 根据运行时或测试条件进行分支。

##### Lines 2966-2966

```python
        mbarrier.arrive(bar.index(0), count=1)
```
- **EN:** Invokes `mbarrier.arrive`, `bar.index` to execute the test logic.
- **CN:** 调用 `mbarrier.arrive`、`bar.index` 执行测试逻辑。

#### Lines 2967-2969

```python

    @gluon.jit
    def consumer(alias1, bar, blocked_layout: ttgl.constexpr):
```
- **EN:** Defines the helper function `consumer`. Decorators: `gluon.jit`. Parameters: `alias1`, `bar`, `blocked_layout`. Key calls include `mbarrier.wait`, `alias1.store`, `ttgl.num_ctas`, `bar.index`, `ttgl.zeros`. This scope touches layout transformation reasoning.
- **CN:** 定义辅助函数 `consumer`。 装饰器：`gluon.jit`。 参数：`alias1`、`bar`、`blocked_layout`。 关键调用包括 `mbarrier.wait`、`alias1.store`、`ttgl.num_ctas`、`bar.index`、`ttgl.zeros`。 该作用域涉及布局变换推理。

##### Lines 2970-2972

```python
        block_m: ttgl.constexpr = XBLOCK * ttgl.num_ctas()
        mbarrier.wait(bar.index(0), phase=0)
        alias1.store(ttgl.zeros([block_m, XBLOCK], ttgl.float32, blocked_layout))
```
- **EN:** Prepares or updates state through `block_m`. Invokes `ttgl.num_ctas`, `mbarrier.wait`, `bar.index`, `alias1.store`, `ttgl.zeros` to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `block_m` 准备或更新状态。 调用 `ttgl.num_ctas`、`mbarrier.wait`、`bar.index`、`alias1.store`、`ttgl.zeros` 执行测试逻辑。 相关主题：布局变换推理。

#### Lines 2973-2975

```python

    @gluon.jit
    def kernel(input, MISSING_WAIT: ttgl.constexpr, OVERLAP: ttgl.constexpr):
```
- **EN:** Defines the helper function `kernel`. Decorators: `gluon.jit`. Parameters: `input`, `MISSING_WAIT`, `OVERLAP`. Key calls include `default_cga_layout`, `ttgl.SwizzledSharedLayout`, `ttgl.BlockedLayout`, `ttgl.allocate_shared_memory`, `mbarrier.allocate_mbarrier`, `mbarrier.init`, and 4 more. This scope touches layout transformation reasoning.
- **CN:** 定义辅助函数 `kernel`。 装饰器：`gluon.jit`。 参数：`input`、`MISSING_WAIT`、`OVERLAP`。 关键调用包括 `default_cga_layout`、`ttgl.SwizzledSharedLayout`、`ttgl.BlockedLayout`、`ttgl.allocate_shared_memory`、`mbarrier.allocate_mbarrier`、`mbarrier.init` 等另外 4 项。 该作用域涉及布局变换推理。

##### Lines 2976-2994

```python
        block_m: ttgl.constexpr = XBLOCK * ttgl.num_ctas()
        cga_layout: ttgl.constexpr = default_cga_layout(ttgl.num_ctas(), 2)
        smem_layout: ttgl.constexpr = ttgl.SwizzledSharedLayout(vec=1, per_phase=1, max_phase=1, order=[0, 1],
                                                                cga_layout=cga_layout)
        blocked_layout: ttgl.constexpr = ttgl.BlockedLayout(size_per_thread=[1, XBLOCK], threads_per_warp=[32, 1],
                                                            warps_per_cta=[4, 1], order=[0, 1], cga_layout=cga_layout)
        blocked_layout_wide: ttgl.constexpr = ttgl.BlockedLayout(size_per_thread=[2, XBLOCK], threads_per_warp=[32, 1],
                                                                 warps_per_cta=[4, 1], order=[0,
                                                                                              1], cga_layout=cga_layout)
        smem = ttgl.allocate_shared_memory(ttgl.float32, [block_m, XBLOCK * 2], smem_layout)
        bar = mbarrier.allocate_mbarrier(batch=1)
        mbarrier.init(bar.index(0), count=1)

        alias0 = smem if OVERLAP else smem.slice(0, XBLOCK, dim=1)
        alias1 = smem.slice(XBLOCK, XBLOCK, dim=1)

        ttgl.warp_specialize([(producer,
                               (input, alias0, bar, MISSING_WAIT, OVERLAP, blocked_layout, blocked_layout_wide)),
                              (consumer, (alias1, bar, blocked_layout))], [4], [32])
```
- **EN:** Prepares or updates state through `block_m`, `cga_layout`, `smem_layout`, `blocked_layout`, `blocked_layout_wide`, `smem`, `bar`, `alias0`, and 1 more. Invokes `ttgl.num_ctas`, `default_cga_layout`, `ttgl.SwizzledSharedLayout`, `ttgl.BlockedLayout`, `ttgl.allocate_shared_memory`, `mbarrier.allocate_mbarrier`, and 4 more to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `block_m`、`cga_layout`、`smem_layout`、`blocked_layout`、`blocked_layout_wide`、`smem`、`bar`、`alias0` 等另外 1 项 准备或更新状态。 调用 `ttgl.num_ctas`、`default_cga_layout`、`ttgl.SwizzledSharedLayout`、`ttgl.BlockedLayout`、`ttgl.allocate_shared_memory`、`mbarrier.allocate_mbarrier` 等另外 4 项 执行测试逻辑。 相关主题：布局变换推理。

#### Lines 2995-2997

```python

    input = torch.randn((XBLOCK.value * num_ctas, XBLOCK.value * 2), device=device, dtype=torch.float32)
    kernel[(1, )](input, MISSING_WAIT=MISSING_WAIT, OVERLAP=OVERLAP, num_warps=4, num_ctas=num_ctas)
```
- **EN:** Prepares or updates state through `input`. Invokes `torch.randn` to execute the test logic. Relevant themes: PyTorch tensor setup and checks, random-data generation.
- **CN:** 通过 `input` 准备或更新状态。 调用 `torch.randn` 执行测试逻辑。 相关主题：PyTorch 张量准备与校验、随机数据生成。

### Lines 2998-3002

```python


@gluon.jit
def async_copy_mma_write_after_read_kernel(a_ptr, BLOCK_M: ttgl.constexpr, BLOCK_N: ttgl.constexpr,
                                           BLOCK_K: ttgl.constexpr):
```
- **EN:** Defines the helper function `async_copy_mma_write_after_read_kernel`. Decorators: `gluon.jit`. Parameters: `a_ptr`, `BLOCK_M`, `BLOCK_N`, `BLOCK_K`. Key calls include `ttgl.BlockedLayout`, `ttgl.allocate_shared_memory`, `mbarrier.allocate_mbarrier`, `blackwell.TensorMemoryLayout`, `allocate_tensor_memory`, `mbarrier.init`, and 8 more. This scope touches layout transformation reasoning.
- **CN:** 定义辅助函数 `async_copy_mma_write_after_read_kernel`。 装饰器：`gluon.jit`。 参数：`a_ptr`、`BLOCK_M`、`BLOCK_N`、`BLOCK_K`。 关键调用包括 `ttgl.BlockedLayout`、`ttgl.allocate_shared_memory`、`mbarrier.allocate_mbarrier`、`blackwell.TensorMemoryLayout`、`allocate_tensor_memory`、`mbarrier.init` 等另外 8 项。 该作用域涉及布局变换推理。

#### Lines 3003-3029

```python
    blocked_layout: ttgl.constexpr = ttgl.BlockedLayout(size_per_thread=[1, 4], threads_per_warp=[32, 1],
                                                        warps_per_cta=[ttgl.num_warps(), 1], order=[0, 1],
                                                        cga_layout=mma_cga_layout(ttgl.num_ctas(), 0))
    a_smem = ttgl.allocate_shared_memory(
        ttgl.float16,
        [BLOCK_M, BLOCK_K],
        ttgl.NVMMASharedLayout.get_default_for([BLOCK_M, BLOCK_K], ttgl.float16,
                                               cga_layout=mma_cga_layout(ttgl.num_ctas(), 0)),
    )
    b_smem = ttgl.allocate_shared_memory(
        ttgl.float16,
        [BLOCK_K, BLOCK_N],
        ttgl.NVMMASharedLayout.get_default_for([BLOCK_K, BLOCK_N], ttgl.float16,
                                               cga_layout=mma_cga_layout(ttgl.num_ctas(), 1)),
    )

    bar = mbarrier.allocate_mbarrier()
    tmem_layout: ttgl.constexpr = blackwell.TensorMemoryLayout([XBLOCK, XBLOCK], col_stride=1,
                                                               cga_layout=mma_cga_layout(ttgl.num_ctas(), 2))
    tmem = allocate_tensor_memory(ttgl.float32, [BLOCK_M, BLOCK_N], tmem_layout)

    mbarrier.init(bar, count=1)
    blackwell.tcgen05_mma(a_smem, b_smem, tmem, use_acc=False)
    offs_m = ttgl.arange(0, BLOCK_M, layout=ttgl.SliceLayout(1, blocked_layout))[:, None]
    offs_k = ttgl.arange(0, BLOCK_K, layout=ttgl.SliceLayout(0, blocked_layout))[None, :]
    offs = offs_m * BLOCK_K + offs_k
    ampere.async_copy.async_copy_global_to_shared(a_smem, a_ptr + offs)
```
- **EN:** Prepares or updates state through `blocked_layout`, `a_smem`, `b_smem`, `bar`, `tmem_layout`, `tmem`, `offs_m`, `offs_k`, and 1 more. Invokes `ttgl.BlockedLayout`, `mma_cga_layout`, `ttgl.num_warps`, `ttgl.num_ctas`, `ttgl.allocate_shared_memory`, `ttgl.NVMMASharedLayout.get_default_for`, and 8 more to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `blocked_layout`、`a_smem`、`b_smem`、`bar`、`tmem_layout`、`tmem`、`offs_m`、`offs_k` 等另外 1 项 准备或更新状态。 调用 `ttgl.BlockedLayout`、`mma_cga_layout`、`ttgl.num_warps`、`ttgl.num_ctas`、`ttgl.allocate_shared_memory`、`ttgl.NVMMASharedLayout.get_default_for` 等另外 8 项 执行测试逻辑。 相关主题：布局变换推理。

### Lines 3030-3033

```python


@pytest.mark.skipif(not is_cuda() or torch.cuda.get_device_capability()[0] < 10, reason="Requires blackwell or newer")
def test_mma_read_async_copy_write(run_wrapper, monkeypatch, num_ctas):
```
- **EN:** Defines the test function `test_mma_read_async_copy_write`. Decorators: `pytest.mark.skipif(not is_cuda() or torch.cuda.get_device_capability()[0] < 10, reason='Requires blackwell or newer')`. Parameters: `run_wrapper`, `monkeypatch`, `num_ctas`. Key calls include `pytest.mark.skipif`, `monkeypatch.setenv`, `knobs.refresh_knobs`, `mma_block_m`, `mma_block_n`, `torch.randn`, and 4 more. This scope touches PyTorch tensor setup and checks, runtime driver interaction, kernel launch orchestration, random-data generation.
- **CN:** 定义测试函数 `test_mma_read_async_copy_write`。 装饰器：`pytest.mark.skipif(not is_cuda() or torch.cuda.get_device_capability()[0] < 10, reason='Requires blackwell or newer')`。 参数：`run_wrapper`、`monkeypatch`、`num_ctas`。 关键调用包括 `pytest.mark.skipif`、`monkeypatch.setenv`、`knobs.refresh_knobs`、`mma_block_m`、`mma_block_n`、`torch.randn` 等另外 4 项。 该作用域涉及PyTorch 张量准备与校验、运行时驱动交互、kernel 启动编排、随机数据生成。

#### Lines 3034-3038

```python
    if run_wrapper:
        result = run_in_process(test_mma_read_async_copy_write, (False, monkeypatch, num_ctas))
        assert_expected_cuda_failure(result.exc)
        assert "Buffer being accessed has outstanding reads" in result.driver_stderr_output
        return
```
- **EN:** Invokes `run_in_process`, `assert_expected_cuda_failure` to execute the test logic. Validates behavior with 1 assertion(s). Branches on runtime or test conditions. Relevant themes: runtime driver interaction.
- **CN:** 调用 `run_in_process`、`assert_expected_cuda_failure` 执行测试逻辑。 通过 1 个断言验证行为。 根据运行时或测试条件进行分支。 相关主题：运行时驱动交互。

#### Lines 3039-3048

```python

    monkeypatch.setenv("TRITON_INSTRUMENTATION_MODE", "consan")
    monkeypatch.setenv("CUDA_LAUNCH_BLOCKING", "1")
    knobs.refresh_knobs()

    BLOCK_M = mma_block_m(num_ctas)
    BLOCK_N = mma_block_n(num_ctas)
    BLOCK_K = XBLOCK.value
    A = torch.randn((BLOCK_M, BLOCK_K), device="cuda", dtype=torch.float16)
    async_copy_mma_write_after_read_kernel[(1, )](A, BLOCK_M, BLOCK_N, BLOCK_K, num_ctas=num_ctas)
```
- **EN:** Prepares or updates state through `BLOCK_M`, `BLOCK_N`, `BLOCK_K`, `A`. Invokes `monkeypatch.setenv`, `knobs.refresh_knobs`, `mma_block_m`, `mma_block_n`, `torch.randn` to execute the test logic. Relevant themes: PyTorch tensor setup and checks, kernel launch orchestration, random-data generation.
- **CN:** 通过 `BLOCK_M`、`BLOCK_N`、`BLOCK_K`、`A` 准备或更新状态。 调用 `monkeypatch.setenv`、`knobs.refresh_knobs`、`mma_block_m`、`mma_block_n`、`torch.randn` 执行测试逻辑。 相关主题：PyTorch 张量准备与校验、kernel 启动编排、随机数据生成。

### Lines 3049-3053

```python


@gluon.jit
def load_local_alloc_mma_write_after_read_kernel(a_ptr, K, BLOCK_M: ttgl.constexpr, BLOCK_N: ttgl.constexpr,
                                                 BLOCK_K: ttgl.constexpr):
```
- **EN:** Defines the helper function `load_local_alloc_mma_write_after_read_kernel`. Decorators: `gluon.jit`. Parameters: `a_ptr`, `K`, `BLOCK_M`, `BLOCK_N`, `BLOCK_K`. Key calls include `ttgl.BlockedLayout`, `ttgl.NVMMASharedLayout.get_default_for`, `ttgl.allocate_shared_memory`, `mbarrier.allocate_mbarrier`, `blackwell.TensorMemoryLayout`, `allocate_tensor_memory`, and 11 more. This scope touches layout transformation reasoning.
- **CN:** 定义辅助函数 `load_local_alloc_mma_write_after_read_kernel`。 装饰器：`gluon.jit`。 参数：`a_ptr`、`K`、`BLOCK_M`、`BLOCK_N`、`BLOCK_K`。 关键调用包括 `ttgl.BlockedLayout`、`ttgl.NVMMASharedLayout.get_default_for`、`ttgl.allocate_shared_memory`、`mbarrier.allocate_mbarrier`、`blackwell.TensorMemoryLayout`、`allocate_tensor_memory` 等另外 11 项。 该作用域涉及布局变换推理。

#### Lines 3054-3077

```python
    blocked_layout: ttgl.constexpr = ttgl.BlockedLayout(size_per_thread=[1, 4], threads_per_warp=[32, 1],
                                                        warps_per_cta=[ttgl.num_warps(), 1], order=[0, 1],
                                                        cga_layout=mma_cga_layout(ttgl.num_ctas(), 0))
    a_smem_layout: ttgl.constexpr = ttgl.NVMMASharedLayout.get_default_for([BLOCK_M, BLOCK_K], ttgl.float16,
                                                                           cga_layout=mma_cga_layout(
                                                                               ttgl.num_ctas(), 0))
    b_smem = ttgl.allocate_shared_memory(
        ttgl.float16,
        [BLOCK_K, BLOCK_N],
        ttgl.NVMMASharedLayout.get_default_for([BLOCK_K, BLOCK_N], ttgl.float16,
                                               cga_layout=mma_cga_layout(ttgl.num_ctas(), 1)),
    )

    bar = mbarrier.allocate_mbarrier()
    tmem_layout: ttgl.constexpr = blackwell.TensorMemoryLayout([XBLOCK, XBLOCK], col_stride=1,
                                                               cga_layout=mma_cga_layout(ttgl.num_ctas(), 2))
    tmem = allocate_tensor_memory(ttgl.float32, [BLOCK_M, BLOCK_N], tmem_layout)

    mbarrier.init(bar, count=1)

    offs_m = ttgl.arange(0, BLOCK_M, layout=ttgl.SliceLayout(1, blocked_layout))[:, None]
    offs_k = ttgl.arange(0, BLOCK_K, layout=ttgl.SliceLayout(0, blocked_layout))[None, :]

    use_acc = False
```
- **EN:** Prepares or updates state through `blocked_layout`, `a_smem_layout`, `b_smem`, `bar`, `tmem_layout`, `tmem`, `offs_m`, `offs_k`, and 1 more. Invokes `ttgl.BlockedLayout`, `mma_cga_layout`, `ttgl.num_warps`, `ttgl.num_ctas`, `ttgl.NVMMASharedLayout.get_default_for`, `ttgl.allocate_shared_memory`, and 6 more to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `blocked_layout`、`a_smem_layout`、`b_smem`、`bar`、`tmem_layout`、`tmem`、`offs_m`、`offs_k` 等另外 1 项 准备或更新状态。 调用 `ttgl.BlockedLayout`、`mma_cga_layout`、`ttgl.num_warps`、`ttgl.num_ctas`、`ttgl.NVMMASharedLayout.get_default_for`、`ttgl.allocate_shared_memory` 等另外 6 项 执行测试逻辑。 相关主题：布局变换推理。

#### Lines 3078-3083

```python
    for k in range(0, K, BLOCK_K):
        a_value = ttgl.load(a_ptr + offs_m * K + offs_k + k)

        a_smem = ttgl.allocate_shared_memory(ttgl.float16, [BLOCK_M, BLOCK_K], a_smem_layout, a_value)
        blackwell.tcgen05_mma(a_smem, b_smem, tmem, use_acc=use_acc)
        use_acc = True
```
- **EN:** Invokes `ttgl.load`, `ttgl.allocate_shared_memory`, `blackwell.tcgen05_mma` to execute the test logic. Iterates across cases or data tiles. Relevant themes: layout transformation reasoning.
- **CN:** 调用 `ttgl.load`、`ttgl.allocate_shared_memory`、`blackwell.tcgen05_mma` 执行测试逻辑。 通过循环覆盖多个用例或数据分块。 相关主题：布局变换推理。

#### Lines 3084-3086

```python
    blackwell.tcgen05_commit(bar)
    mbarrier.wait(bar, phase=0)
    mbarrier.invalidate(bar)
```
- **EN:** Invokes `blackwell.tcgen05_commit`, `mbarrier.wait`, `mbarrier.invalidate` to execute the test logic.
- **CN:** 调用 `blackwell.tcgen05_commit`、`mbarrier.wait`、`mbarrier.invalidate` 执行测试逻辑。

### Lines 3087-3090

```python


@pytest.mark.skipif(not is_cuda() or torch.cuda.get_device_capability()[0] < 10, reason="Requires blackwell or newer")
def test_mma_read_local_alloc_write(run_wrapper, monkeypatch, num_ctas):
```
- **EN:** Defines the test function `test_mma_read_local_alloc_write`. Decorators: `pytest.mark.skipif(not is_cuda() or torch.cuda.get_device_capability()[0] < 10, reason='Requires blackwell or newer')`. Parameters: `run_wrapper`, `monkeypatch`, `num_ctas`. Key calls include `pytest.mark.skipif`, `monkeypatch.setenv`, `knobs.refresh_knobs`, `mma_block_m`, `mma_block_n`, `torch.randn`, and 4 more. This scope touches PyTorch tensor setup and checks, runtime driver interaction, kernel launch orchestration, random-data generation.
- **CN:** 定义测试函数 `test_mma_read_local_alloc_write`。 装饰器：`pytest.mark.skipif(not is_cuda() or torch.cuda.get_device_capability()[0] < 10, reason='Requires blackwell or newer')`。 参数：`run_wrapper`、`monkeypatch`、`num_ctas`。 关键调用包括 `pytest.mark.skipif`、`monkeypatch.setenv`、`knobs.refresh_knobs`、`mma_block_m`、`mma_block_n`、`torch.randn` 等另外 4 项。 该作用域涉及PyTorch 张量准备与校验、运行时驱动交互、kernel 启动编排、随机数据生成。

#### Lines 3091-3095

```python
    if run_wrapper:
        result = run_in_process(test_mma_read_local_alloc_write, (False, monkeypatch, num_ctas))
        assert_expected_cuda_failure(result.exc)
        assert "Buffer being accessed has outstanding reads" in result.driver_stderr_output
        return
```
- **EN:** Invokes `run_in_process`, `assert_expected_cuda_failure` to execute the test logic. Validates behavior with 1 assertion(s). Branches on runtime or test conditions. Relevant themes: runtime driver interaction.
- **CN:** 调用 `run_in_process`、`assert_expected_cuda_failure` 执行测试逻辑。 通过 1 个断言验证行为。 根据运行时或测试条件进行分支。 相关主题：运行时驱动交互。

#### Lines 3096-3106

```python

    monkeypatch.setenv("TRITON_INSTRUMENTATION_MODE", "consan")
    monkeypatch.setenv("CUDA_LAUNCH_BLOCKING", "1")
    knobs.refresh_knobs()

    K = 512
    BLOCK_M = mma_block_m(num_ctas)
    BLOCK_N = mma_block_n(num_ctas)
    BLOCK_K = 64
    A = torch.randn((BLOCK_M, K), device="cuda", dtype=torch.float16)
    load_local_alloc_mma_write_after_read_kernel[(1, )](A, K, BLOCK_M, BLOCK_N, BLOCK_K, num_ctas=num_ctas)
```
- **EN:** Prepares or updates state through `K`, `BLOCK_M`, `BLOCK_N`, `BLOCK_K`, `A`. Invokes `monkeypatch.setenv`, `knobs.refresh_knobs`, `mma_block_m`, `mma_block_n`, `torch.randn` to execute the test logic. Relevant themes: PyTorch tensor setup and checks, kernel launch orchestration, random-data generation.
- **CN:** 通过 `K`、`BLOCK_M`、`BLOCK_N`、`BLOCK_K`、`A` 准备或更新状态。 调用 `monkeypatch.setenv`、`knobs.refresh_knobs`、`mma_block_m`、`mma_block_n`、`torch.randn` 执行测试逻辑。 相关主题：PyTorch 张量准备与校验、kernel 启动编排、随机数据生成。

## Key Concepts / 关键概念

- **EN:** Top-level scopes such as `run_wrapper`, `num_ctas`, `assert_expected_cuda_failure`, `mma_cga_layout`, `mma_block_m`, `mma_block_n`, `default_cga_layout`, `multicast_cga_layout`
  **CN:** 顶层作用域，例如 `run_wrapper`、`num_ctas`、`assert_expected_cuda_failure`、`mma_cga_layout`、`mma_block_m`、`mma_block_n`、`default_cga_layout`、`multicast_cga_layout`
- **EN:** pytest parametrization
  **CN:** pytest 参数化
- **EN:** shared fixtures
  **CN:** 共享 fixture
- **EN:** PyTorch tensor setup and checks
  **CN:** PyTorch 张量准备与校验
- **EN:** subprocess-driven validation
  **CN:** 基于子进程的验证
- **EN:** cache management behavior
  **CN:** 缓存管理行为
- **EN:** tensor/descriptor metadata
  **CN:** 张量/描述符元数据
- **EN:** plugin or compiler extension points
  **CN:** 插件或编译器扩展点

## Dependencies / 依赖关系

- **EN:** External or absolute imports include `os`, `tempfile`, `torch`, `pytest`, `triton`, `triton.experimental`, `triton.experimental.gluon`, `triton.experimental.gluon.language.nvidia`, `triton.experimental.gluon.language.nvidia.blackwell`, `triton._internal_testing`.
  **CN:** 外部或绝对导入包括 `os`、`tempfile`、`torch`、`pytest`、`triton`、`triton.experimental`、`triton.experimental.gluon`、`triton.experimental.gluon.language.nvidia`、`triton.experimental.gluon.language.nvidia.blackwell`、`triton._internal_testing`。
- **EN:** Execution centers on top-level definitions such as `run_wrapper`, `num_ctas`, `assert_expected_cuda_failure`, `mma_cga_layout`, `mma_block_m`, `mma_block_n`, `default_cga_layout`, `multicast_cga_layout`, `failing_kernel`, `run_failing_kernel`.
  **CN:** 执行逻辑主要围绕顶层定义展开，例如 `run_wrapper`、`num_ctas`、`assert_expected_cuda_failure`、`mma_cga_layout`、`mma_block_m`、`mma_block_n`、`default_cga_layout`、`multicast_cga_layout`、`failing_kernel`、`run_failing_kernel`。
- **EN:** Runtime behavior also depends on pytest collection, Python execution semantics, and the imported Triton/PyTorch utilities visible above.
  **CN:** 运行时行为还依赖 pytest 的收集机制、Python 执行语义，以及上文可见的 Triton/PyTorch 工具。
