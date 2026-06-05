# test_core.py — Code Analysis / 代码分析

## Source / 来源
- **Path / 路径:** `python/test/gluon/test_core.py`
- **EN:** Pytest module covering core behavior in Triton's Python tests. It contains 109 top-level definition(s) and 21 imported module reference(s).
- **CN:** 这是一个 pytest 模块，用于覆盖 Triton Python 测试中的 core 行为。 该文件包含 109 个顶层定义，以及 21 个导入模块引用。

## Line-by-Line Analysis / 逐行分析

### Lines 1-46

```python
import torch
import math
import pytest
import re
from itertools import product

import triton
import triton.language as tl

from triton._internal_testing import (
    is_ampere_or_newer,
    is_blackwell,
    is_blackwell_ultra,
    is_hip_rdna,
    is_hip_rdna3,
    is_hip_rdna4,
    is_hip_cdna,
    is_hip_cdna3,
    is_hip_cdna4,
    is_hopper_or_newer,
    is_hopper,
)
from triton.compiler import max_shared_mem
from triton.tools.mxfp import MXFP4Tensor, MXScaleTensor
from triton.experimental import gluon
from triton.experimental.gluon import language as ttgl
from triton.experimental.gluon.language.nvidia.ampere import async_copy, mma_v2
from triton.experimental.gluon.language.nvidia.hopper import tma, mbarrier, fence_async_shared
from triton.experimental.gluon.language.nvidia import hopper
from triton.experimental.gluon.language.nvidia.blackwell import tma as blackwell_tma
from triton.experimental.gluon.language.amd.cdna4 import async_copy as cdna4_async_copy
from triton.experimental.gluon.language.extra import libdevice
from triton.experimental.gluon.language.nvidia.blackwell import (
    TensorMemoryLayout,
    TensorMemoryScalesLayout,
    allocate_tensor_memory,
    tcgen05_mma_barrier_count,
    tcgen05_mma,
    tcgen05_mma_scaled,
    tcgen05_commit,
    tcgen05_copy,
    float2,
    clc,
)
from triton.experimental.gluon.nvidia.hopper import TensorDescriptor
from triton._C.libtriton.gluon_ir import make_cga_layout
```
- **EN:** Imports the modules used in this scope: `torch`, `math`, `pytest`, `re`, `itertools`, `triton`, `triton.language`, `triton._internal_testing`, `triton.compiler`, `triton.tools.mxfp`, and 10 more. Relevant themes: Triton compilation or JIT kernels, Triton language operations, tensor/descriptor metadata, layout transformation reasoning.
- **CN:** 导入此作用域使用的模块：`torch`、`math`、`pytest`、`re`、`itertools`、`triton`、`triton.language`、`triton._internal_testing`、`triton.compiler`、`triton.tools.mxfp` 等另外 10 项。 相关主题：Triton 编译或 JIT kernel、Triton language 操作、张量/描述符元数据、布局变换推理。

### Lines 47-48

```python

THREADS_PER_WARP = triton.runtime.driver.active.get_current_target().warp_size
```
- **EN:** Prepares or updates state through `THREADS_PER_WARP`. Invokes `triton.runtime.driver.active.get_current_target` to execute the test logic. Relevant themes: runtime driver interaction.
- **CN:** 通过 `THREADS_PER_WARP` 准备或更新状态。 调用 `triton.runtime.driver.active.get_current_target` 执行测试逻辑。 相关主题：运行时驱动交互。

### Lines 49-52

```python


@gluon.jit
def copy_kernel(Out, In, numel, XBLOCK: ttgl.constexpr, layout: ttgl.constexpr):
```
- **EN:** Defines the helper function `copy_kernel`. Decorators: `gluon.jit`. Parameters: `Out`, `In`, `numel`, `XBLOCK`, `layout`. Key calls include `ttgl.load`, `ttgl.store`, `ttgl.program_id`, `ttgl.arange`. This scope touches layout transformation reasoning.
- **CN:** 定义辅助函数 `copy_kernel`。 装饰器：`gluon.jit`。 参数：`Out`、`In`、`numel`、`XBLOCK`、`layout`。 关键调用包括 `ttgl.load`、`ttgl.store`、`ttgl.program_id`、`ttgl.arange`。 该作用域涉及布局变换推理。

#### Lines 53-57

```python
    xbase = ttgl.program_id(0) * XBLOCK
    xoffset = xbase + ttgl.arange(0, XBLOCK, layout=layout)
    xmask = xoffset < numel
    data = ttgl.load(In + xoffset, xmask)
    ttgl.store(Out + xoffset, data, xmask)
```
- **EN:** Prepares or updates state through `xbase`, `xoffset`, `xmask`, `data`. Invokes `ttgl.program_id`, `ttgl.arange`, `ttgl.load`, `ttgl.store` to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `xbase`、`xoffset`、`xmask`、`data` 准备或更新状态。 调用 `ttgl.program_id`、`ttgl.arange`、`ttgl.load`、`ttgl.store` 执行测试逻辑。 相关主题：布局变换推理。

### Lines 58-71

```python


@pytest.mark.parametrize("layout", [
    ttgl.BlockedLayout(size_per_thread=[1], threads_per_warp=[THREADS_PER_WARP], warps_per_cta=[4], order=[0]),
    ttgl.BlockedLayout(size_per_thread=[2], threads_per_warp=[THREADS_PER_WARP], warps_per_cta=[4], order=[0]),
    ttgl.BlockedLayout(size_per_thread=[4], threads_per_warp=[THREADS_PER_WARP], warps_per_cta=[4], order=[0]),
    ttgl.BlockedLayout(size_per_thread=[8], threads_per_warp=[THREADS_PER_WARP], warps_per_cta=[4], order=[0]),
    ttgl.BlockedLayout(size_per_thread=[1], threads_per_warp=[THREADS_PER_WARP], warps_per_cta=[8], order=[0]),
    ttgl.BlockedLayout(size_per_thread=[2], threads_per_warp=[THREADS_PER_WARP], warps_per_cta=[8], order=[0]),
    ttgl.BlockedLayout(size_per_thread=[4], threads_per_warp=[THREADS_PER_WARP], warps_per_cta=[8], order=[0]),
    ttgl.BlockedLayout(size_per_thread=[8], threads_per_warp=[THREADS_PER_WARP], warps_per_cta=[8], order=[0]),
])
@pytest.mark.parametrize("XBLOCK", [128, 256, 512, 1024, 2048])
def test_copy_kernel(layout, XBLOCK):
```
- **EN:** Defines the test function `test_copy_kernel`. Decorators: `pytest.mark.parametrize('layout', [ttgl.BlockedLayout(size_per_thread=[1], threads_per_warp=[THREADS_PER_WARP], warps_per_cta=[4], order=[0]), ttgl.BlockedLayout(size_per_thread=[2], threads_per_warp=[THREADS_PER_WARP], warps_per_cta=[4], order=[0]), ttgl.BlockedLayout(size_per_thread=[4], threads_per_warp=[THREADS_PER_WARP], warps_per_cta=[4], order=[0]), ttgl.BlockedLayout(size_per_thread=[8], threads_per_warp=[THREADS_PER_WARP], warps_per_cta=[4], order=[0]), ttgl.BlockedLayout(size_per_thread=[1], threads_per_warp=[THREADS_PER_WARP], warps_per_cta=[8], order=[0]), ttgl.BlockedLayout(size_per_thread=[2], threads_per_warp=[THREADS_PER_WARP], warps_per_cta=[8], order=[0]), ttgl.BlockedLayout(size_per_thread=[4], threads_per_warp=[THREADS_PER_WARP], warps_per_cta=[8], order=[0]), ttgl.BlockedLayout(size_per_thread=[8], threads_per_warp=[THREADS_PER_WARP], warps_per_cta=[8], order=[0])])`, `pytest.mark.parametrize('XBLOCK', [128, 256, 512, 1024, 2048])`. Parameters: `layout`, `XBLOCK`. Key calls include `pytest.mark.parametrize`, `torch.randn`, `torch.empty_like`, `torch.testing.assert_close`, `inp.numel`, `ttgl.BlockedLayout`. This scope touches pytest parametrization, PyTorch tensor setup and checks, layout transformation reasoning, random-data generation.
- **CN:** 定义测试函数 `test_copy_kernel`。 装饰器：`pytest.mark.parametrize('layout', [ttgl.BlockedLayout(size_per_thread=[1], threads_per_warp=[THREADS_PER_WARP], warps_per_cta=[4], order=[0]), ttgl.BlockedLayout(size_per_thread=[2], threads_per_warp=[THREADS_PER_WARP], warps_per_cta=[4], order=[0]), ttgl.BlockedLayout(size_per_thread=[4], threads_per_warp=[THREADS_PER_WARP], warps_per_cta=[4], order=[0]), ttgl.BlockedLayout(size_per_thread=[8], threads_per_warp=[THREADS_PER_WARP], warps_per_cta=[4], order=[0]), ttgl.BlockedLayout(size_per_thread=[1], threads_per_warp=[THREADS_PER_WARP], warps_per_cta=[8], order=[0]), ttgl.BlockedLayout(size_per_thread=[2], threads_per_warp=[THREADS_PER_WARP], warps_per_cta=[8], order=[0]), ttgl.BlockedLayout(size_per_thread=[4], threads_per_warp=[THREADS_PER_WARP], warps_per_cta=[8], order=[0]), ttgl.BlockedLayout(size_per_thread=[8], threads_per_warp=[THREADS_PER_WARP], warps_per_cta=[8], order=[0])])`、`pytest.mark.parametrize('XBLOCK', [128, 256, 512, 1024, 2048])`。 参数：`layout`、`XBLOCK`。 关键调用包括 `pytest.mark.parametrize`、`torch.randn`、`torch.empty_like`、`torch.testing.assert_close`、`inp.numel`、`ttgl.BlockedLayout`。 该作用域涉及pytest 参数化、PyTorch 张量准备与校验、布局变换推理、随机数据生成。

#### Lines 72-76

```python
    inp = torch.randn(XBLOCK * 4 - 7, device="cuda")
    out = torch.empty_like(inp)

    copy_kernel[(4, )](out, inp, inp.numel(), XBLOCK, layout, num_warps=layout.warps_per_cta[0])
    torch.testing.assert_close(out, inp)
```
- **EN:** Prepares or updates state through `inp`, `out`. Invokes `torch.randn`, `torch.empty_like`, `inp.numel`, `torch.testing.assert_close` to execute the test logic. Relevant themes: PyTorch tensor setup and checks, layout transformation reasoning, random-data generation.
- **CN:** 通过 `inp`、`out` 准备或更新状态。 调用 `torch.randn`、`torch.empty_like`、`inp.numel`、`torch.testing.assert_close` 执行测试逻辑。 相关主题：PyTorch 张量准备与校验、布局变换推理、随机数据生成。

### Lines 77-80

```python


@pytest.mark.skipif(not is_hopper_or_newer(), reason="Requires Hopper")
def test_copy_kernel_multi_cta():
```
- **EN:** Defines the test function `test_copy_kernel_multi_cta`. Decorators: `pytest.mark.skipif(not is_hopper_or_newer(), reason='Requires Hopper')`. Key calls include `pytest.mark.skipif`, `ttgl.BlockedLayout`, `torch.randn`, `torch.empty_like`, `torch.testing.assert_close`, `inp.numel`, and 1 more. This scope touches PyTorch tensor setup and checks, layout transformation reasoning, random-data generation.
- **CN:** 定义测试函数 `test_copy_kernel_multi_cta`。 装饰器：`pytest.mark.skipif(not is_hopper_or_newer(), reason='Requires Hopper')`。 关键调用包括 `pytest.mark.skipif`、`ttgl.BlockedLayout`、`torch.randn`、`torch.empty_like`、`torch.testing.assert_close`、`inp.numel` 等另外 1 项。 该作用域涉及PyTorch 张量准备与校验、布局变换推理、随机数据生成。

#### Lines 81-88

```python
    XBLOCK = 2048
    layout = ttgl.BlockedLayout(size_per_thread=[8], threads_per_warp=[THREADS_PER_WARP], warps_per_cta=[8], order=[0],
                                cga_layout=[[1]])

    inp = torch.randn(XBLOCK * 4 - 7, device="cuda")
    out = torch.empty_like(inp)
    copy_kernel[(4, )](out, inp, inp.numel(), XBLOCK, layout, num_warps=layout.warps_per_cta[0], num_ctas=2)
    torch.testing.assert_close(out, inp)
```
- **EN:** Prepares or updates state through `XBLOCK`, `layout`, `inp`, `out`. Invokes `ttgl.BlockedLayout`, `torch.randn`, `torch.empty_like`, `inp.numel`, `torch.testing.assert_close` to execute the test logic. Relevant themes: PyTorch tensor setup and checks, layout transformation reasoning, random-data generation.
- **CN:** 通过 `XBLOCK`、`layout`、`inp`、`out` 准备或更新状态。 调用 `ttgl.BlockedLayout`、`torch.randn`、`torch.empty_like`、`inp.numel`、`torch.testing.assert_close` 执行测试逻辑。 相关主题：PyTorch 张量准备与校验、布局变换推理、随机数据生成。

### Lines 89-92

```python


@gluon.jit
def local_store_transposed_cga_kernel(inp, out, BLOCK_M: ttgl.constexpr, BLOCK_N: ttgl.constexpr):
```
- **EN:** Defines the helper function `local_store_transposed_cga_kernel`. Decorators: `gluon.jit`. Parameters: `inp`, `out`, `BLOCK_M`, `BLOCK_N`. Key calls include `ttgl.BlockedLayout`, `ttgl.SwizzledSharedLayout`, `ttgl.load`, `value.trans`, `ttgl.allocate_shared_memory`, `smem.store`, and 6 more. This scope touches layout transformation reasoning.
- **CN:** 定义辅助函数 `local_store_transposed_cga_kernel`。 装饰器：`gluon.jit`。 参数：`inp`、`out`、`BLOCK_M`、`BLOCK_N`。 关键调用包括 `ttgl.BlockedLayout`、`ttgl.SwizzledSharedLayout`、`ttgl.load`、`value.trans`、`ttgl.allocate_shared_memory`、`smem.store` 等另外 6 项。 该作用域涉及布局变换推理。

#### Lines 93-129

```python
    src_layout: ttgl.constexpr = ttgl.BlockedLayout(
        size_per_thread=[1, BLOCK_N],
        threads_per_warp=[32, 1],
        warps_per_cta=[4, 1],
        order=[0, 1],
        cga_layout=((1, 0), ),
    )
    dst_layout: ttgl.constexpr = ttgl.BlockedLayout(
        size_per_thread=[1, BLOCK_M // 4],
        threads_per_warp=[8, 4],
        warps_per_cta=[4, 1],
        order=[1, 0],
        cga_layout=((0, 1), ),
    )
    smem_layout: ttgl.constexpr = ttgl.SwizzledSharedLayout(
        vec=1,
        per_phase=1,
        max_phase=1,
        order=[1, 0],
        cga_layout=((1, 0), ),
    )

    src_m = ttgl.arange(0, BLOCK_M, layout=ttgl.SliceLayout(1, src_layout))[:, None]
    src_n = ttgl.arange(0, BLOCK_N, layout=ttgl.SliceLayout(0, src_layout))[None, :]
    src_ptrs = inp + src_m * BLOCK_N + src_n
    value = ttgl.load(src_ptrs)
    transposed = value.trans()

    smem = ttgl.allocate_shared_memory(ttgl.float32, [BLOCK_N, BLOCK_M], smem_layout)
    smem.store(transposed)
    ttgl.barrier(cluster=True)

    dst_n = ttgl.arange(0, BLOCK_N)[:, None]
    dst_m = ttgl.arange(0, BLOCK_M)[None, :]
    result = smem.load(dst_layout)
    dst_ptrs = out + dst_n * BLOCK_M + dst_m
    ttgl.store(ttgl.set_auto_layout(dst_ptrs, dst_layout), result)
```
- **EN:** Prepares or updates state through `src_layout`, `dst_layout`, `smem_layout`, `src_m`, `src_n`, `src_ptrs`, `value`, `transposed`, and 5 more. Invokes `ttgl.BlockedLayout`, `ttgl.SwizzledSharedLayout`, `ttgl.arange`, `ttgl.SliceLayout`, `ttgl.load`, `value.trans`, and 6 more to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `src_layout`、`dst_layout`、`smem_layout`、`src_m`、`src_n`、`src_ptrs`、`value`、`transposed` 等另外 5 项 准备或更新状态。 调用 `ttgl.BlockedLayout`、`ttgl.SwizzledSharedLayout`、`ttgl.arange`、`ttgl.SliceLayout`、`ttgl.load`、`value.trans` 等另外 6 项 执行测试逻辑。 相关主题：布局变换推理。

### Lines 130-133

```python


@pytest.mark.skipif(not is_hopper_or_newer(), reason="Requires Hopper")
def test_local_store_transposed_cga_to_non_transposed_alloc():
```
- **EN:** Defines the test function `test_local_store_transposed_cga_to_non_transposed_alloc`. Decorators: `pytest.mark.skipif(not is_hopper_or_newer(), reason='Requires Hopper')`. Key calls include `pytest.mark.skipif`, `torch.empty`, `torch.testing.assert_close`, `is_hopper_or_newer`, `torch.arange`. This scope touches PyTorch tensor setup and checks.
- **CN:** 定义测试函数 `test_local_store_transposed_cga_to_non_transposed_alloc`。 装饰器：`pytest.mark.skipif(not is_hopper_or_newer(), reason='Requires Hopper')`。 关键调用包括 `pytest.mark.skipif`、`torch.empty`、`torch.testing.assert_close`、`is_hopper_or_newer`、`torch.arange`。 该作用域涉及PyTorch 张量准备与校验。

#### Lines 134-140

```python
    block_m = 256
    block_n = 64
    inp = torch.arange(block_m * block_n, device="cuda", dtype=torch.float32).reshape(block_m, block_n)
    out = torch.empty((block_n, block_m), device="cuda", dtype=torch.float32)

    compiled = local_store_transposed_cga_kernel[(1, )](inp, out, block_m, block_n, num_warps=4, num_ctas=2)
    ptx = compiled.asm["ptx"]
```
- **EN:** Prepares or updates state through `block_m`, `block_n`, `inp`, `out`, `compiled`, `ptx`. Invokes `torch.arange`, `torch.empty` to execute the test logic. Relevant themes: PyTorch tensor setup and checks.
- **CN:** 通过 `block_m`、`block_n`、`inp`、`out`、`compiled`、`ptx` 准备或更新状态。 调用 `torch.arange`、`torch.empty` 执行测试逻辑。 相关主题：PyTorch 张量准备与校验。

#### Lines 141-142

```python
    assert "st.shared::cluster" in ptx
    assert "ld.shared::cluster" in ptx
```
- **EN:** Validates behavior with 2 assertion(s).
- **CN:** 通过 2 个断言验证行为。

#### Lines 143-143

```python
    torch.testing.assert_close(out, inp.T, atol=0, rtol=0)
```
- **EN:** Invokes `torch.testing.assert_close` to execute the test logic. Relevant themes: PyTorch tensor setup and checks.
- **CN:** 调用 `torch.testing.assert_close` 执行测试逻辑。 相关主题：PyTorch 张量准备与校验。

### Lines 144-147

```python


@gluon.jit
def tma_kernel(desc):
```
- **EN:** Defines the helper function `tma_kernel`. Decorators: `gluon.jit`. Parameters: `desc`. Key calls include `ttgl.BlockedLayout`, `ttgl.full`, `ttgl.allocate_shared_memory`, `tma.async_copy_shared_to_global`, `tma.store_wait`, `alloc._keep_alive`. This scope touches layout transformation reasoning.
- **CN:** 定义辅助函数 `tma_kernel`。 装饰器：`gluon.jit`。 参数：`desc`。 关键调用包括 `ttgl.BlockedLayout`、`ttgl.full`、`ttgl.allocate_shared_memory`、`tma.async_copy_shared_to_global`、`tma.store_wait`、`alloc._keep_alive`。 该作用域涉及布局变换推理。

#### Lines 148-153

```python
    layout: ttgl.constexpr = ttgl.BlockedLayout([1, 2], [4, 8], [4, 1], [1, 0])
    value = ttgl.full(desc.block_shape, 0, desc.dtype, layout)
    alloc = ttgl.allocate_shared_memory(desc.dtype, desc.block_shape, desc.layout, value)
    tma.async_copy_shared_to_global(desc, [0, 0], alloc)
    tma.store_wait(0)
    alloc._keep_alive()
```
- **EN:** Prepares or updates state through `layout`, `value`, `alloc`. Invokes `ttgl.BlockedLayout`, `ttgl.full`, `ttgl.allocate_shared_memory`, `tma.async_copy_shared_to_global`, `tma.store_wait`, `alloc._keep_alive` to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `layout`、`value`、`alloc` 准备或更新状态。 调用 `ttgl.BlockedLayout`、`ttgl.full`、`ttgl.allocate_shared_memory`、`tma.async_copy_shared_to_global`、`tma.store_wait`、`alloc._keep_alive` 执行测试逻辑。 相关主题：布局变换推理。

### Lines 154-157

```python


@pytest.mark.skipif(not is_hopper_or_newer(), reason="Requires Hopper")
def test_tma():
```
- **EN:** Defines the test function `test_tma`. Decorators: `pytest.mark.skipif(not is_hopper_or_newer(), reason='Requires Hopper')`. Key calls include `pytest.mark.skipif`, `torch.ones`, `ttgl.NVMMASharedLayout`, `gluon.nvidia.hopper.TensorDescriptor.from_tensor`, `torch.testing.assert_close`, `torch.zeros_like`, and 1 more. This scope touches PyTorch tensor setup and checks, tensor/descriptor metadata, layout transformation reasoning.
- **CN:** 定义测试函数 `test_tma`。 装饰器：`pytest.mark.skipif(not is_hopper_or_newer(), reason='Requires Hopper')`。 关键调用包括 `pytest.mark.skipif`、`torch.ones`、`ttgl.NVMMASharedLayout`、`gluon.nvidia.hopper.TensorDescriptor.from_tensor`、`torch.testing.assert_close`、`torch.zeros_like` 等另外 1 项。 该作用域涉及PyTorch 张量准备与校验、张量/描述符元数据、布局变换推理。

#### Lines 158-169

```python
    out = torch.ones((16, 16), dtype=torch.float16, device="cuda")
    layout = ttgl.NVMMASharedLayout(
        swizzle_byte_width=32,
        element_bitwidth=16,
        rank=2,
        transposed=False,
        fp4_padded=False,
    )

    desc = gluon.nvidia.hopper.TensorDescriptor.from_tensor(out, [16, 16], layout)
    tma_kernel[(1, )](desc)
    torch.testing.assert_close(out, torch.zeros_like(out))
```
- **EN:** Prepares or updates state through `out`, `layout`, `desc`. Invokes `torch.ones`, `ttgl.NVMMASharedLayout`, `gluon.nvidia.hopper.TensorDescriptor.from_tensor`, `torch.testing.assert_close`, `torch.zeros_like` to execute the test logic. Relevant themes: PyTorch tensor setup and checks, tensor/descriptor metadata, layout transformation reasoning.
- **CN:** 通过 `out`、`layout`、`desc` 准备或更新状态。 调用 `torch.ones`、`ttgl.NVMMASharedLayout`、`gluon.nvidia.hopper.TensorDescriptor.from_tensor`、`torch.testing.assert_close`、`torch.zeros_like` 执行测试逻辑。 相关主题：PyTorch 张量准备与校验、张量/描述符元数据、布局变换推理。

### Lines 170-173

```python


@gluon.jit
def tma_im2col_kernel(in_desc, out_desc):
```
- **EN:** Defines the helper function `tma_im2col_kernel`. Decorators: `gluon.jit`. Parameters: `in_desc`, `out_desc`. Key calls include `ttgl.allocate_shared_memory`, `mbarrier.allocate_mbarrier`, `mbarrier.init`, `mbarrier.expect`, `tma.async_load_im2col`, `mbarrier.wait`, and 3 more. This scope touches layout transformation reasoning.
- **CN:** 定义辅助函数 `tma_im2col_kernel`。 装饰器：`gluon.jit`。 参数：`in_desc`、`out_desc`。 关键调用包括 `ttgl.allocate_shared_memory`、`mbarrier.allocate_mbarrier`、`mbarrier.init`、`mbarrier.expect`、`tma.async_load_im2col`、`mbarrier.wait` 等另外 3 项。 该作用域涉及布局变换推理。

#### Lines 174-182

```python
    smem = ttgl.allocate_shared_memory(in_desc.dtype, in_desc.block_shape, in_desc.layout)
    bar = mbarrier.allocate_mbarrier()
    mbarrier.init(bar, count=1)
    mbarrier.expect(bar, in_desc.block_type.nbytes)
    tma.async_load_im2col(in_desc, [0, 0, 0, 0], [0, 0], bar, smem)
    mbarrier.wait(bar, phase=0)
    mbarrier.invalidate(bar)
    tma.async_copy_shared_to_global(out_desc, [0, 0], smem)
    tma.store_wait(pendings=0)
```
- **EN:** Prepares or updates state through `smem`, `bar`. Invokes `ttgl.allocate_shared_memory`, `mbarrier.allocate_mbarrier`, `mbarrier.init`, `mbarrier.expect`, `tma.async_load_im2col`, `mbarrier.wait`, and 3 more to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `smem`、`bar` 准备或更新状态。 调用 `ttgl.allocate_shared_memory`、`mbarrier.allocate_mbarrier`、`mbarrier.init`、`mbarrier.expect`、`tma.async_load_im2col`、`mbarrier.wait` 等另外 3 项 执行测试逻辑。 相关主题：布局变换推理。

### Lines 183-189

```python


@pytest.mark.skipif(not is_hopper_or_newer(), reason="Requires Hopper")
@pytest.mark.parametrize("pixels_per_column", [32, 256, 512, 1024])
@pytest.mark.parametrize("channels_per_pixel", [32])
@pytest.mark.parametrize("swizzle_byte_width", [32])
def test_tma_im2col(pixels_per_column, channels_per_pixel, swizzle_byte_width):
```
- **EN:** Defines the test function `test_tma_im2col`. Decorators: `pytest.mark.skipif(not is_hopper_or_newer(), reason='Requires Hopper')`, `pytest.mark.parametrize('pixels_per_column', [32, 256, 512, 1024])`, `pytest.mark.parametrize('channels_per_pixel', [32])`, `pytest.mark.parametrize('swizzle_byte_width', [32])`. Parameters: `pixels_per_column`, `channels_per_pixel`, `swizzle_byte_width`. Key calls include `pytest.mark.skipif`, `pytest.mark.parametrize`, `torch.arange`, `inp.reshape`, `torch.zeros`, `ttgl.NVMMASharedLayout`, and 6 more. This scope touches pytest parametrization, PyTorch tensor setup and checks, tensor/descriptor metadata, layout transformation reasoning.
- **CN:** 定义测试函数 `test_tma_im2col`。 装饰器：`pytest.mark.skipif(not is_hopper_or_newer(), reason='Requires Hopper')`、`pytest.mark.parametrize('pixels_per_column', [32, 256, 512, 1024])`、`pytest.mark.parametrize('channels_per_pixel', [32])`、`pytest.mark.parametrize('swizzle_byte_width', [32])`。 参数：`pixels_per_column`、`channels_per_pixel`、`swizzle_byte_width`。 关键调用包括 `pytest.mark.skipif`、`pytest.mark.parametrize`、`torch.arange`、`inp.reshape`、`torch.zeros`、`ttgl.NVMMASharedLayout` 等另外 6 项。 该作用域涉及pytest 参数化、PyTorch 张量准备与校验、张量/描述符元数据、布局变换推理。

#### Lines 190-190

```python
    smem_bytes = pixels_per_column * channels_per_pixel * 4 + 8192  # block + mbarrier overhead
```
- **EN:** Prepares or updates state through `smem_bytes`.
- **CN:** 通过 `smem_bytes` 准备或更新状态。

#### Lines 191-192

```python
    if smem_bytes > 200000:
        pytest.skip(f"Skipping: shared memory {smem_bytes} exceeds limit")
```
- **EN:** Invokes `pytest.skip` to execute the test logic. Branches on runtime or test conditions.
- **CN:** 调用 `pytest.skip` 执行测试逻辑。 根据运行时或测试条件进行分支。

#### Lines 193-220

```python

    inp = torch.arange(pixels_per_column * channels_per_pixel, device="cuda", dtype=torch.float32)
    inp = inp.reshape(1, 1, pixels_per_column, channels_per_pixel)
    out = torch.zeros(pixels_per_column, channels_per_pixel, device="cuda", dtype=torch.float32)

    block_shape = [pixels_per_column, channels_per_pixel]
    layout = ttgl.NVMMASharedLayout(
        swizzle_byte_width=swizzle_byte_width,
        element_bitwidth=32,
        rank=2,
        transposed=False,
        fp4_padded=False,
    )

    in_desc = gluon.nvidia.hopper.TensorDescriptorIm2Col(
        base=inp,
        shape=list(inp.shape),
        strides=list(inp.stride()),
        block_shape=block_shape,
        layout=layout,
        padding="zero",
        element_strides=[1, 1, 1, 1],
        pixel_box_lower_corner=[0, 0],
        pixel_box_upper_corner=[0, 0],
    )
    out_desc = gluon.nvidia.hopper.TensorDescriptor.from_tensor(out, block_shape, layout)
    tma_im2col_kernel[(1, )](in_desc, out_desc, num_warps=1)
    torch.testing.assert_close(out, inp.reshape(pixels_per_column, channels_per_pixel), atol=0, rtol=0)
```
- **EN:** Prepares or updates state through `inp`, `out`, `block_shape`, `layout`, `in_desc`, `out_desc`. Invokes `torch.arange`, `inp.reshape`, `torch.zeros`, `ttgl.NVMMASharedLayout`, `gluon.nvidia.hopper.TensorDescriptorIm2Col`, `inp.stride`, and 2 more to execute the test logic. Relevant themes: PyTorch tensor setup and checks, tensor/descriptor metadata, layout transformation reasoning.
- **CN:** 通过 `inp`、`out`、`block_shape`、`layout`、`in_desc`、`out_desc` 准备或更新状态。 调用 `torch.arange`、`inp.reshape`、`torch.zeros`、`ttgl.NVMMASharedLayout`、`gluon.nvidia.hopper.TensorDescriptorIm2Col`、`inp.stride` 等另外 2 项 执行测试逻辑。 相关主题：PyTorch 张量准备与校验、张量/描述符元数据、布局变换推理。

### Lines 221-224

```python


@gluon.jit
def tma_round_f32_to_tf32_kernel(in_desc, out_desc):
```
- **EN:** Defines the helper function `tma_round_f32_to_tf32_kernel`. Decorators: `gluon.jit`. Parameters: `in_desc`, `out_desc`. Key calls include `ttgl.allocate_shared_memory`, `mbarrier.allocate_mbarrier`, `mbarrier.init`, `mbarrier.expect`, `tma.async_load`, `mbarrier.wait`, and 4 more. This scope touches layout transformation reasoning.
- **CN:** 定义辅助函数 `tma_round_f32_to_tf32_kernel`。 装饰器：`gluon.jit`。 参数：`in_desc`、`out_desc`。 关键调用包括 `ttgl.allocate_shared_memory`、`mbarrier.allocate_mbarrier`、`mbarrier.init`、`mbarrier.expect`、`tma.async_load`、`mbarrier.wait` 等另外 4 项。 该作用域涉及布局变换推理。

#### Lines 225-234

```python
    smem = ttgl.allocate_shared_memory(in_desc.dtype, in_desc.block_shape, in_desc.layout)
    bar = mbarrier.allocate_mbarrier()
    mbarrier.init(bar, count=1)
    mbarrier.expect(bar, in_desc.nbytes_per_cta)
    tma.async_load(in_desc, [0, 0], bar, smem)
    mbarrier.wait(bar, phase=0, deps=[smem])
    mbarrier.invalidate(bar)
    tma.async_copy_shared_to_global(out_desc, [0, 0], smem)
    tma.store_wait(0)
    smem._keep_alive()
```
- **EN:** Prepares or updates state through `smem`, `bar`. Invokes `ttgl.allocate_shared_memory`, `mbarrier.allocate_mbarrier`, `mbarrier.init`, `mbarrier.expect`, `tma.async_load`, `mbarrier.wait`, and 4 more to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `smem`、`bar` 准备或更新状态。 调用 `ttgl.allocate_shared_memory`、`mbarrier.allocate_mbarrier`、`mbarrier.init`、`mbarrier.expect`、`tma.async_load`、`mbarrier.wait` 等另外 4 项 执行测试逻辑。 相关主题：布局变换推理。

### Lines 235-239

```python


@pytest.mark.skipif(not is_hopper_or_newer(), reason="Requires Hopper")
def test_gluon_tma_round_f32_to_tf32():
```
- **EN:** Defines the test function `test_gluon_tma_round_f32_to_tf32`. Decorators: `pytest.mark.skipif(not is_hopper_or_newer(), reason='Requires Hopper')`. Nested definitions in this scope: `round_to_tf32`. Key calls include `pytest.mark.skipif`, `torch.manual_seed`, `torch.randn`, `torch.empty_like`, `ttgl.NVMMASharedLayout`, `gluon.nvidia.hopper.TensorDescriptor.from_tensor`, and 6 more. This scope touches PyTorch tensor setup and checks, tensor/descriptor metadata, layout transformation reasoning, random-data generation.
- **CN:** 定义测试函数 `test_gluon_tma_round_f32_to_tf32`。 装饰器：`pytest.mark.skipif(not is_hopper_or_newer(), reason='Requires Hopper')`。 该作用域中的嵌套定义：`round_to_tf32`。 关键调用包括 `pytest.mark.skipif`、`torch.manual_seed`、`torch.randn`、`torch.empty_like`、`ttgl.NVMMASharedLayout`、`gluon.nvidia.hopper.TensorDescriptor.from_tensor` 等另外 6 项。 该作用域涉及PyTorch 张量准备与校验、张量/描述符元数据、布局变换推理、随机数据生成。

#### Lines 240-240

```python
    def round_to_tf32(x: torch.Tensor) -> torch.Tensor:
```
- **EN:** Defines the helper function `round_to_tf32`. Parameters: `x`. Key calls include `x.view`, `torch.where`, `bits.to`. This scope touches PyTorch tensor setup and checks.
- **CN:** 定义辅助函数 `round_to_tf32`。 参数：`x`。 关键调用包括 `x.view`、`torch.where`、`bits.to`。 该作用域涉及PyTorch 张量准备与校验。

##### Lines 241-248

```python
        bits = x.view(torch.int32)
        bits_i64 = bits.to(torch.int64) & 0xFFFFFFFF
        exp_mask = 0x7F800000
        is_special = (bits_i64 & exp_mask) == exp_mask
        round_bias = ((bits_i64 >> 13) & 1) + 0x00000FFF
        rounded = (bits_i64 + round_bias) & 0xFFFFE000
        out_bits = torch.where(is_special, bits_i64, rounded)
        return (out_bits & 0xFFFFFFFF).to(torch.int32).view(torch.float32)
```
- **EN:** Prepares or updates state through `bits`, `bits_i64`, `exp_mask`, `is_special`, `round_bias`, `rounded`, `out_bits`. Invokes `x.view`, `bits.to`, `torch.where` to execute the test logic. Relevant themes: PyTorch tensor setup and checks.
- **CN:** 通过 `bits`、`bits_i64`、`exp_mask`、`is_special`、`round_bias`、`rounded`、`out_bits` 准备或更新状态。 调用 `x.view`、`bits.to`、`torch.where` 执行测试逻辑。 相关主题：PyTorch 张量准备与校验。

#### Lines 249-267

```python

    torch.manual_seed(17)
    inp = torch.randn((16, 16), device="cuda", dtype=torch.float32)
    out = torch.empty_like(inp)

    layout = ttgl.NVMMASharedLayout(
        swizzle_byte_width=32,
        element_bitwidth=32,
        rank=2,
        transposed=False,
        fp4_padded=False,
    )
    in_desc = gluon.nvidia.hopper.TensorDescriptor.from_tensor(inp, [16, 16], layout, round_f32_to_tf32=True)
    out_desc = gluon.nvidia.hopper.TensorDescriptor.from_tensor(out, [16, 16], layout)

    tma_round_f32_to_tf32_kernel[(1, )](in_desc, out_desc)

    expected = round_to_tf32(inp)
    torch.testing.assert_close(out, expected, rtol=0, atol=0)
```
- **EN:** Prepares or updates state through `inp`, `out`, `layout`, `in_desc`, `out_desc`, `expected`. Invokes `torch.manual_seed`, `torch.randn`, `torch.empty_like`, `ttgl.NVMMASharedLayout`, `gluon.nvidia.hopper.TensorDescriptor.from_tensor`, `round_to_tf32`, and 1 more to execute the test logic. Relevant themes: PyTorch tensor setup and checks, tensor/descriptor metadata, layout transformation reasoning, random-data generation.
- **CN:** 通过 `inp`、`out`、`layout`、`in_desc`、`out_desc`、`expected` 准备或更新状态。 调用 `torch.manual_seed`、`torch.randn`、`torch.empty_like`、`ttgl.NVMMASharedLayout`、`gluon.nvidia.hopper.TensorDescriptor.from_tensor`、`round_to_tf32` 等另外 1 项 执行测试逻辑。 相关主题：PyTorch 张量准备与校验、张量/描述符元数据、布局变换推理、随机数据生成。

### Lines 268-271

```python


@gluon.jit
def tma_multicast_copy_kernel(in_desc, out_desc):
```
- **EN:** Defines the helper function `tma_multicast_copy_kernel`. Decorators: `gluon.jit`. Parameters: `in_desc`, `out_desc`. Key calls include `ttgl.allocate_shared_memory`, `mbarrier.allocate_mbarrier`, `mbarrier.init`, `mbarrier.expect`, `tma.async_load`, `mbarrier.wait`, and 4 more. This scope touches layout transformation reasoning.
- **CN:** 定义辅助函数 `tma_multicast_copy_kernel`。 装饰器：`gluon.jit`。 参数：`in_desc`、`out_desc`。 关键调用包括 `ttgl.allocate_shared_memory`、`mbarrier.allocate_mbarrier`、`mbarrier.init`、`mbarrier.expect`、`tma.async_load`、`mbarrier.wait` 等另外 4 项。 该作用域涉及布局变换推理。

#### Lines 272-285

```python
    smem = ttgl.allocate_shared_memory(in_desc.dtype, in_desc.block_shape, in_desc.layout)

    bar = mbarrier.allocate_mbarrier()
    mbarrier.init(bar, count=1)

    mbarrier.expect(bar, in_desc.nbytes_per_cta)
    tma.async_load(in_desc, [0, 0], bar, smem, multicast=True)
    mbarrier.wait(bar, phase=0, deps=[smem])

    tma.async_copy_shared_to_global(out_desc, [0, 0], smem)
    tma.store_wait(0)

    mbarrier.invalidate(bar)
    smem._keep_alive()
```
- **EN:** Prepares or updates state through `smem`, `bar`. Invokes `ttgl.allocate_shared_memory`, `mbarrier.allocate_mbarrier`, `mbarrier.init`, `mbarrier.expect`, `tma.async_load`, `mbarrier.wait`, and 4 more to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `smem`、`bar` 准备或更新状态。 调用 `ttgl.allocate_shared_memory`、`mbarrier.allocate_mbarrier`、`mbarrier.init`、`mbarrier.expect`、`tma.async_load`、`mbarrier.wait` 等另外 4 项 执行测试逻辑。 相关主题：布局变换推理。

### Lines 286-290

```python


@pytest.mark.skipif(not is_hopper_or_newer(), reason="Requires Hopper")
@pytest.mark.parametrize("ctas_per_cga", [[2, 1], [1, 4], [4, 4]])
def test_tma_multicast_copy(ctas_per_cga):
```
- **EN:** Defines the test function `test_tma_multicast_copy`. Decorators: `pytest.mark.skipif(not is_hopper_or_newer(), reason='Requires Hopper')`, `pytest.mark.parametrize('ctas_per_cga', [[2, 1], [1, 4], [4, 4]])`. Parameters: `ctas_per_cga`. Key calls include `pytest.mark.skipif`, `pytest.mark.parametrize`, `make_cga_layout`, `torch.randn`, `torch.empty_like`, `ttgl.NVMMASharedLayout.get_default_for`, and 5 more. This scope touches pytest parametrization, PyTorch tensor setup and checks, tensor/descriptor metadata, layout transformation reasoning.
- **CN:** 定义测试函数 `test_tma_multicast_copy`。 装饰器：`pytest.mark.skipif(not is_hopper_or_newer(), reason='Requires Hopper')`、`pytest.mark.parametrize('ctas_per_cga', [[2, 1], [1, 4], [4, 4]])`。 参数：`ctas_per_cga`。 关键调用包括 `pytest.mark.skipif`、`pytest.mark.parametrize`、`make_cga_layout`、`torch.randn`、`torch.empty_like`、`ttgl.NVMMASharedLayout.get_default_for` 等另外 5 项。 该作用域涉及pytest 参数化、PyTorch 张量准备与校验、张量/描述符元数据、布局变换推理。

#### Lines 291-316

```python
    cga_split_num = [min(ctas_per_cga[0], 2), min(ctas_per_cga[1], 2)]
    cga_layout = make_cga_layout(ctas_per_cga, cga_split_num, [1, 0])

    BLOCK_M, BLOCK_N = 16, 16
    BLOCK_M *= cga_split_num[0]
    BLOCK_N *= cga_split_num[1]

    inp = torch.randn((BLOCK_M, BLOCK_N), dtype=torch.float16, device="cuda")
    out = torch.empty_like(inp)

    layout = ttgl.NVMMASharedLayout.get_default_for(
        [BLOCK_M, BLOCK_N],
        ttgl.float16,
        cga_layout=cga_layout,
    )

    in_desc = gluon.nvidia.hopper.TensorDescriptor.from_tensor(inp, [BLOCK_M, BLOCK_N], layout)
    out_desc = gluon.nvidia.hopper.TensorDescriptor.from_tensor(out, [BLOCK_M, BLOCK_N], layout)
    num_ctas = ctas_per_cga[0] * ctas_per_cga[1]
    compiled = tma_multicast_copy_kernel[(1, )](
        in_desc,
        out_desc,
        num_warps=4,
        num_ctas=num_ctas,
    )
    expect_multicast = any(ctas_per_cga[i] > cga_split_num[i] for i in range(len(ctas_per_cga)))
```
- **EN:** Prepares or updates state through `cga_split_num`, `cga_layout`, `BLOCK_M`, `BLOCK_N`, `inp`, `out`, `layout`, `in_desc`, and 4 more. Invokes `min`, `make_cga_layout`, `torch.randn`, `torch.empty_like`, `ttgl.NVMMASharedLayout.get_default_for`, `gluon.nvidia.hopper.TensorDescriptor.from_tensor`, and 1 more to execute the test logic. Relevant themes: PyTorch tensor setup and checks, tensor/descriptor metadata, layout transformation reasoning, random-data generation.
- **CN:** 通过 `cga_split_num`、`cga_layout`、`BLOCK_M`、`BLOCK_N`、`inp`、`out`、`layout`、`in_desc` 等另外 4 项 准备或更新状态。 调用 `min`、`make_cga_layout`、`torch.randn`、`torch.empty_like`、`ttgl.NVMMASharedLayout.get_default_for`、`gluon.nvidia.hopper.TensorDescriptor.from_tensor` 等另外 1 项 执行测试逻辑。 相关主题：PyTorch 张量准备与校验、张量/描述符元数据、布局变换推理、随机数据生成。

#### Lines 317-317

```python
    assert (".multicast::cluster" in compiled.asm["ptx"]) == expect_multicast
```
- **EN:** Validates behavior with 1 assertion(s).
- **CN:** 通过 1 个断言验证行为。

#### Lines 318-318

```python
    torch.testing.assert_close(out, inp, atol=0, rtol=0)
```
- **EN:** Invokes `torch.testing.assert_close` to execute the test logic. Relevant themes: PyTorch tensor setup and checks.
- **CN:** 调用 `torch.testing.assert_close` 执行测试逻辑。 相关主题：PyTorch 张量准备与校验。

### Lines 319-323

```python


@gluon.jit
def tma_gather_scatter_kernel(in_desc, gather_out_desc, scatter_out_desc, gather_idx_ptr, scatter_idx_ptr,
                              BLOCK_M: ttgl.constexpr, x_offsets_layout: ttgl.constexpr):
```
- **EN:** Defines the helper function `tma_gather_scatter_kernel`. Decorators: `gluon.jit`. Parameters: `in_desc`, `gather_out_desc`, `scatter_out_desc`, `gather_idx_ptr`, `scatter_idx_ptr`, `BLOCK_M`, `x_offsets_layout`. Key calls include `ttgl.allocate_shared_memory`, `mbarrier.allocate_mbarrier`, `mbarrier.init`, `ttgl.load`, `mbarrier.expect`, `blackwell_tma.async_gather`, and 7 more. This scope touches layout transformation reasoning.
- **CN:** 定义辅助函数 `tma_gather_scatter_kernel`。 装饰器：`gluon.jit`。 参数：`in_desc`、`gather_out_desc`、`scatter_out_desc`、`gather_idx_ptr`、`scatter_idx_ptr`、`BLOCK_M`、`x_offsets_layout`。 关键调用包括 `ttgl.allocate_shared_memory`、`mbarrier.allocate_mbarrier`、`mbarrier.init`、`ttgl.load`、`mbarrier.expect`、`blackwell_tma.async_gather` 等另外 7 项。 该作用域涉及布局变换推理。

#### Lines 324-341

```python
    smem = ttgl.allocate_shared_memory(in_desc.dtype, [BLOCK_M, gather_out_desc.block_shape[1]], gather_out_desc.layout)

    bar = mbarrier.allocate_mbarrier()
    mbarrier.init(bar, count=1)

    gather_offsets = ttgl.load(gather_idx_ptr + ttgl.arange(0, BLOCK_M, layout=x_offsets_layout))
    mbarrier.expect(bar, smem.nbytes_per_cta)
    blackwell_tma.async_gather(in_desc, gather_offsets, 0, bar, smem, multicast=True)
    mbarrier.wait(bar, phase=0, deps=[smem])

    mbarrier.invalidate(bar)

    scatter_offsets = ttgl.load(scatter_idx_ptr + ttgl.arange(0, BLOCK_M, layout=x_offsets_layout))
    tma.async_copy_shared_to_global(gather_out_desc, [0, 0], smem)
    blackwell_tma.async_scatter(scatter_out_desc, scatter_offsets, 0, smem)
    tma.store_wait(0)

    smem._keep_alive()
```
- **EN:** Prepares or updates state through `smem`, `bar`, `gather_offsets`, `scatter_offsets`. Invokes `ttgl.allocate_shared_memory`, `mbarrier.allocate_mbarrier`, `mbarrier.init`, `ttgl.load`, `ttgl.arange`, `mbarrier.expect`, and 7 more to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `smem`、`bar`、`gather_offsets`、`scatter_offsets` 准备或更新状态。 调用 `ttgl.allocate_shared_memory`、`mbarrier.allocate_mbarrier`、`mbarrier.init`、`ttgl.load`、`ttgl.arange`、`mbarrier.expect` 等另外 7 项 执行测试逻辑。 相关主题：布局变换推理。

### Lines 342-344

```python


def get_split_dim(cga_layout, dim):
```
- **EN:** Defines the helper function `get_split_dim`. Parameters: `cga_layout`, `dim`. Key calls include `sum`. This scope touches layout transformation reasoning.
- **CN:** 定义辅助函数 `get_split_dim`。 参数：`cga_layout`、`dim`。 关键调用包括 `sum`。 该作用域涉及布局变换推理。

#### Lines 345-345

```python
    return 1 << sum(basis[dim] != 0 for basis in cga_layout)
```
- **EN:** Invokes `sum` to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 调用 `sum` 执行测试逻辑。 相关主题：布局变换推理。

### Lines 346-355

```python


@pytest.mark.skipif(not is_blackwell(), reason="Requires Blackwell")
@pytest.mark.parametrize("cga_layout", [
    [[1, 0]],
    [[0, 0], [1, 0]],
    [[1, 0], [0, 0]],
    [[1, 0], [2, 0]],
])
def test_tma_gather_scatter_multi_cta(cga_layout):
```
- **EN:** Defines the test function `test_tma_gather_scatter_multi_cta`. Decorators: `pytest.mark.skipif(not is_blackwell(), reason='Requires Blackwell')`, `pytest.mark.parametrize('cga_layout', [[[1, 0]], [[0, 0], [1, 0]], [[1, 0], [0, 0]], [[1, 0], [2, 0]]])`. Parameters: `cga_layout`. Key calls include `pytest.mark.skipif`, `pytest.mark.parametrize`, `torch.arange`, `torch.empty_like`, `torch.zeros_like`, `ttgl.NVMMASharedLayout.get_default_for`, and 10 more. This scope touches pytest parametrization, PyTorch tensor setup and checks, tensor/descriptor metadata, layout transformation reasoning.
- **CN:** 定义测试函数 `test_tma_gather_scatter_multi_cta`。 装饰器：`pytest.mark.skipif(not is_blackwell(), reason='Requires Blackwell')`、`pytest.mark.parametrize('cga_layout', [[[1, 0]], [[0, 0], [1, 0]], [[1, 0], [0, 0]], [[1, 0], [2, 0]]])`。 参数：`cga_layout`。 关键调用包括 `pytest.mark.skipif`、`pytest.mark.parametrize`、`torch.arange`、`torch.empty_like`、`torch.zeros_like`、`ttgl.NVMMASharedLayout.get_default_for` 等另外 10 项。 该作用域涉及pytest 参数化、PyTorch 张量准备与校验、张量/描述符元数据、布局变换推理。

#### Lines 356-396

```python
    cga_split_num = [get_split_dim(cga_layout, dim) for dim in range(2)]

    BLOCK_M = 32 * cga_split_num[0]
    BLOCK_N = 128 * cga_split_num[1]

    inp = torch.arange(BLOCK_M * BLOCK_N, dtype=torch.float16, device="cuda").reshape(BLOCK_M, BLOCK_N)
    gather_idx = torch.arange(BLOCK_M - 1, -1, -1, dtype=torch.int32, device="cuda")
    scatter_idx = (torch.arange(0, BLOCK_M, dtype=torch.int32, device="cuda") + 1) % BLOCK_M
    gather_out = torch.empty_like(inp)
    scatter_out = torch.zeros_like(inp)

    layout = ttgl.NVMMASharedLayout.get_default_for(
        [BLOCK_M, BLOCK_N],
        ttgl.float16,
        cga_layout=cga_layout,
    )
    in_desc = gluon.nvidia.hopper.TensorDescriptor.from_tensor(inp, [1, BLOCK_N // cga_split_num[1]], layout)
    gather_out_desc = gluon.nvidia.hopper.TensorDescriptor.from_tensor(gather_out, [BLOCK_M, BLOCK_N], layout)
    scatter_out_desc = gluon.nvidia.hopper.TensorDescriptor.from_tensor(scatter_out, [1, BLOCK_N // cga_split_num[1]],
                                                                        layout)

    offset_layout = ttgl.BlockedLayout([4, 1], [1, 32], [4, 1], [0, 1], cga_layout=cga_layout)
    x_offsets_layout = ttgl.SliceLayout(1, offset_layout)

    num_ctas = 1 << len(cga_layout)
    compiled = tma_gather_scatter_kernel[(1, )](
        in_desc,
        gather_out_desc,
        scatter_out_desc,
        gather_idx,
        scatter_idx,
        BLOCK_M,
        x_offsets_layout,
        num_warps=4,
        num_ctas=num_ctas,
    )

    expected_gather = inp[gather_idx.to(torch.int64)]
    expected_scatter = torch.zeros_like(inp)
    expected_scatter[scatter_idx.to(torch.int64)] = expected_gather
    expect_multicast = any(all(coord == 0 for coord in basis) for basis in cga_layout)
```
- **EN:** Prepares or updates state through `cga_split_num`, `BLOCK_M`, `BLOCK_N`, `inp`, `gather_idx`, `scatter_idx`, `gather_out`, `scatter_out`, and 12 more. Invokes `get_split_dim`, `torch.arange`, `torch.empty_like`, `torch.zeros_like`, `ttgl.NVMMASharedLayout.get_default_for`, `gluon.nvidia.hopper.TensorDescriptor.from_tensor`, and 6 more to execute the test logic. Relevant themes: PyTorch tensor setup and checks, tensor/descriptor metadata, layout transformation reasoning.
- **CN:** 通过 `cga_split_num`、`BLOCK_M`、`BLOCK_N`、`inp`、`gather_idx`、`scatter_idx`、`gather_out`、`scatter_out` 等另外 12 项 准备或更新状态。 调用 `get_split_dim`、`torch.arange`、`torch.empty_like`、`torch.zeros_like`、`ttgl.NVMMASharedLayout.get_default_for`、`gluon.nvidia.hopper.TensorDescriptor.from_tensor` 等另外 6 项 执行测试逻辑。 相关主题：PyTorch 张量准备与校验、张量/描述符元数据、布局变换推理。

#### Lines 397-397

```python
    assert (".multicast::cluster" in compiled.asm["ptx"]) == expect_multicast
```
- **EN:** Validates behavior with 1 assertion(s).
- **CN:** 通过 1 个断言验证行为。

#### Lines 398-399

```python
    torch.testing.assert_close(gather_out, expected_gather, atol=0, rtol=0)
    torch.testing.assert_close(scatter_out, expected_scatter, atol=0, rtol=0)
```
- **EN:** Invokes `torch.testing.assert_close` to execute the test logic. Relevant themes: PyTorch tensor setup and checks.
- **CN:** 调用 `torch.testing.assert_close` 执行测试逻辑。 相关主题：PyTorch 张量准备与校验。

### Lines 400-404

```python


@gluon.jit
def tcgen05_mma_multicast_commit_kernel(a_desc, b_desc, out_ptrs, BLOCK_M: ttgl.constexpr, BLOCK_N: ttgl.constexpr,
                                        acc_tmem_layout: ttgl.constexpr, blocked_c: ttgl.constexpr):
```
- **EN:** Defines the helper function `tcgen05_mma_multicast_commit_kernel`. Decorators: `gluon.jit`. Parameters: `a_desc`, `b_desc`, `out_ptrs`, `BLOCK_M`, `BLOCK_N`, `acc_tmem_layout`, `blocked_c`. Key calls include `ttgl.allocate_shared_memory`, `allocate_tensor_memory`, `mbarrier.allocate_mbarrier`, `mbarrier.init`, `mbarrier.expect`, `tma.async_load`, and 8 more. This scope touches layout transformation reasoning.
- **CN:** 定义辅助函数 `tcgen05_mma_multicast_commit_kernel`。 装饰器：`gluon.jit`。 参数：`a_desc`、`b_desc`、`out_ptrs`、`BLOCK_M`、`BLOCK_N`、`acc_tmem_layout`、`blocked_c`。 关键调用包括 `ttgl.allocate_shared_memory`、`allocate_tensor_memory`、`mbarrier.allocate_mbarrier`、`mbarrier.init`、`mbarrier.expect`、`tma.async_load` 等另外 8 项。 该作用域涉及布局变换推理。

#### Lines 405-431

```python
    smem_a = ttgl.allocate_shared_memory(a_desc.dtype, a_desc.block_shape, a_desc.layout)
    smem_b = ttgl.allocate_shared_memory(b_desc.dtype, b_desc.block_shape, b_desc.layout)
    acc_tmem = allocate_tensor_memory(ttgl.float32, [BLOCK_M, BLOCK_N], acc_tmem_layout)

    tma_bar = mbarrier.allocate_mbarrier(two_ctas=acc_tmem_layout.two_ctas)
    mbarrier.init(tma_bar, count=1)
    mma_bar = mbarrier.allocate_mbarrier()
    mbarrier.init(mma_bar, count=tcgen05_mma_barrier_count([smem_a, smem_b], True, acc_tmem.type.layout.two_ctas))

    mbarrier.expect(tma_bar, a_desc.nbytes_per_cta + b_desc.nbytes_per_cta)
    tma.async_load(a_desc, [0, 0], tma_bar, smem_a, multicast=True)
    tma.async_load(b_desc, [0, 0], tma_bar, smem_b, multicast=True)
    mbarrier.wait(tma_bar, phase=0, deps=[smem_a, smem_b])
    mbarrier.invalidate(tma_bar)

    # If it's not in a loop we don't striclty need multicast=True, but we add it to exercise the path in the test
    tcgen05_mma(smem_a, smem_b, acc_tmem, use_acc=False, multicast=True, mbarriers=[mma_bar])
    mbarrier.wait(mma_bar, phase=0, deps=[smem_a, smem_b])
    mbarrier.invalidate(mma_bar)

    out = acc_tmem.load()
    out = ttgl.convert_layout(out, blocked_c)

    out_offs_m = ttgl.arange(0, BLOCK_M)[:, None]
    out_offs_n = ttgl.arange(0, BLOCK_N)[None, :]
    out_ptrs = out_ptrs + out_offs_m * BLOCK_N + out_offs_n
    ttgl.store(out_ptrs, out)
```
- **EN:** Prepares or updates state through `smem_a`, `smem_b`, `acc_tmem`, `tma_bar`, `mma_bar`, `out`, `out_offs_m`, `out_offs_n`, and 1 more. Invokes `ttgl.allocate_shared_memory`, `allocate_tensor_memory`, `mbarrier.allocate_mbarrier`, `mbarrier.init`, `tcgen05_mma_barrier_count`, `mbarrier.expect`, and 8 more to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `smem_a`、`smem_b`、`acc_tmem`、`tma_bar`、`mma_bar`、`out`、`out_offs_m`、`out_offs_n` 等另外 1 项 准备或更新状态。 调用 `ttgl.allocate_shared_memory`、`allocate_tensor_memory`、`mbarrier.allocate_mbarrier`、`mbarrier.init`、`tcgen05_mma_barrier_count`、`mbarrier.expect` 等另外 8 项 执行测试逻辑。 相关主题：布局变换推理。

### Lines 432-436

```python


@gluon.jit
def tma_wrong_barrier_cga_layout_kernel(a_desc, b_desc, BLOCK_M: ttgl.constexpr, BLOCK_N: ttgl.constexpr,
                                        acc_tmem_layout: ttgl.constexpr):
```
- **EN:** Defines the helper function `tma_wrong_barrier_cga_layout_kernel`. Decorators: `gluon.jit`. Parameters: `a_desc`, `b_desc`, `BLOCK_M`, `BLOCK_N`, `acc_tmem_layout`. Key calls include `ttgl.allocate_shared_memory`, `allocate_tensor_memory`, `mbarrier.MBarrierLayout`, `mbarrier.init`, `mbarrier.expect`, `tma.async_load`, and 1 more. This scope touches layout transformation reasoning.
- **CN:** 定义辅助函数 `tma_wrong_barrier_cga_layout_kernel`。 装饰器：`gluon.jit`。 参数：`a_desc`、`b_desc`、`BLOCK_M`、`BLOCK_N`、`acc_tmem_layout`。 关键调用包括 `ttgl.allocate_shared_memory`、`allocate_tensor_memory`、`mbarrier.MBarrierLayout`、`mbarrier.init`、`mbarrier.expect`、`tma.async_load` 等另外 1 项。 该作用域涉及布局变换推理。

#### Lines 437-447

```python
    smem_a = ttgl.allocate_shared_memory(a_desc.dtype, a_desc.block_shape, a_desc.layout)
    smem_b = ttgl.allocate_shared_memory(b_desc.dtype, b_desc.block_shape, b_desc.layout)
    acc_tmem = allocate_tensor_memory(ttgl.float32, [BLOCK_M, BLOCK_N], acc_tmem_layout)

    bad_barrier_layout: ttgl.constexpr = mbarrier.MBarrierLayout(cga_layout=((0, ), (0, )))
    bad_tma_bar = ttgl.allocate_shared_memory(ttgl.int64, [1], bad_barrier_layout)
    mbarrier.init(bad_tma_bar, count=1)
    mbarrier.expect(bad_tma_bar, a_desc.nbytes_per_cta)
    tma.async_load(a_desc, [0, 0], bad_tma_bar, smem_a)

    tcgen05_mma(smem_a, smem_b, acc_tmem, use_acc=False)
```
- **EN:** Prepares or updates state through `smem_a`, `smem_b`, `acc_tmem`, `bad_barrier_layout`, `bad_tma_bar`. Invokes `ttgl.allocate_shared_memory`, `allocate_tensor_memory`, `mbarrier.MBarrierLayout`, `mbarrier.init`, `mbarrier.expect`, `tma.async_load`, and 1 more to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `smem_a`、`smem_b`、`acc_tmem`、`bad_barrier_layout`、`bad_tma_bar` 准备或更新状态。 调用 `ttgl.allocate_shared_memory`、`allocate_tensor_memory`、`mbarrier.MBarrierLayout`、`mbarrier.init`、`mbarrier.expect`、`tma.async_load` 等另外 1 项 执行测试逻辑。 相关主题：布局变换推理。

### Lines 448-450

```python


def make_2cta_cga_layout(ctas_per_cga, cta_split, cta_order, two_cta_dim):
```
- **EN:** Defines the helper function `make_2cta_cga_layout`. Parameters: `ctas_per_cga`, `cta_split`, `cta_order`, `two_cta_dim`. Key calls include `make_cga_layout`. This scope touches layout transformation reasoning.
- **CN:** 定义辅助函数 `make_2cta_cga_layout`。 参数：`ctas_per_cga`、`cta_split`、`cta_order`、`two_cta_dim`。 关键调用包括 `make_cga_layout`。 该作用域涉及布局变换推理。

#### Lines 451-452

```python
    ctas_per_cga = list(ctas_per_cga)
    cta_split = list(cta_split)
```
- **EN:** Prepares or updates state through `ctas_per_cga`, `cta_split`.
- **CN:** 通过 `ctas_per_cga`、`cta_split` 准备或更新状态。

#### Lines 453-453

```python
    assert cta_split[two_cta_dim] > 1
```
- **EN:** Validates behavior with 1 assertion(s).
- **CN:** 通过 1 个断言验证行为。

#### Lines 454-456

```python
    cta_split[two_cta_dim] //= 2
    ctas_per_cga[two_cta_dim] //= 2
    aux_cga_layout = make_cga_layout(ctas_per_cga, cta_split, cta_order)
```
- **EN:** Prepares or updates state through `cta_split`, `two_cta_dim`, `ctas_per_cga`, `aux_cga_layout`. Invokes `make_cga_layout` to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `cta_split`、`two_cta_dim`、`ctas_per_cga`、`aux_cga_layout` 准备或更新状态。 调用 `make_cga_layout` 执行测试逻辑。 相关主题：布局变换推理。

#### Lines 457-457

```python
    assert two_cta_dim in (0, 1)
```
- **EN:** Validates behavior with 1 assertion(s).
- **CN:** 通过 1 个断言验证行为。

#### Lines 458-459

```python
    basis = [0, 0]
    basis[two_cta_dim] = 1
```
- **EN:** Prepares or updates state through `basis`, `two_cta_dim`.
- **CN:** 通过 `basis`、`two_cta_dim` 准备或更新状态。

#### Lines 460-461

```python
    for b in aux_cga_layout:
        b[two_cta_dim] *= 2
```
- **EN:** Iterates across cases or data tiles. Relevant themes: layout transformation reasoning.
- **CN:** 通过循环覆盖多个用例或数据分块。 相关主题：布局变换推理。

#### Lines 462-463

```python
    cga_layout = [basis] + aux_cga_layout
    return cga_layout
```
- **EN:** Prepares or updates state through `cga_layout`. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `cga_layout` 准备或更新状态。 相关主题：布局变换推理。

### Lines 464-470

```python


@pytest.mark.skipif(not is_blackwell(), reason="Requires Blackwell")
@pytest.mark.parametrize("ctas_per_cga", [[2, 1], [2, 4], [4, 4]])
@pytest.mark.parametrize("two_ctas", [True, False] if is_blackwell() else [False])
def test_tcgen05_mma_multicast_commit(ctas_per_cga, two_ctas):
```
- **EN:** Defines the test function `test_tcgen05_mma_multicast_commit`. Decorators: `pytest.mark.skipif(not is_blackwell(), reason='Requires Blackwell')`, `pytest.mark.parametrize('ctas_per_cga', [[2, 1], [2, 4], [4, 4]])`, `pytest.mark.parametrize('two_ctas', [True, False] if is_blackwell() else [False])`. Parameters: `ctas_per_cga`, `two_ctas`. Key calls include `pytest.mark.skipif`, `pytest.mark.parametrize`, `ttgl.NVMMASharedLayout.get_default_for`, `torch.randn`, `torch.empty`, `gluon.nvidia.hopper.TensorDescriptor.from_tensor`, and 9 more. This scope touches pytest parametrization, PyTorch tensor setup and checks, tensor/descriptor metadata, matrix multiplication workflows.
- **CN:** 定义测试函数 `test_tcgen05_mma_multicast_commit`。 装饰器：`pytest.mark.skipif(not is_blackwell(), reason='Requires Blackwell')`、`pytest.mark.parametrize('ctas_per_cga', [[2, 1], [2, 4], [4, 4]])`、`pytest.mark.parametrize('two_ctas', [True, False] if is_blackwell() else [False])`。 参数：`ctas_per_cga`、`two_ctas`。 关键调用包括 `pytest.mark.skipif`、`pytest.mark.parametrize`、`ttgl.NVMMASharedLayout.get_default_for`、`torch.randn`、`torch.empty`、`gluon.nvidia.hopper.TensorDescriptor.from_tensor` 等另外 9 项。 该作用域涉及pytest 参数化、PyTorch 张量准备与校验、张量/描述符元数据、矩阵乘法工作流。

#### Lines 471-474

```python
    if two_ctas:
        ctas_per_cga_b = [ctas_per_cga[0] // 2, 2 * ctas_per_cga[1]]
    else:
        ctas_per_cga_b = ctas_per_cga
```
- **EN:** Branches on runtime or test conditions.
- **CN:** 根据运行时或测试条件进行分支。

#### Lines 475-482

```python
    BLOCK_M = 128 * ctas_per_cga[0]
    BLOCK_N = 64 * ctas_per_cga_b[1]
    BLOCK_K = 32

    # multicast into tcgen05_mma
    cta_split_a = [ctas_per_cga[0], 1]
    cta_split_b = [1, ctas_per_cga_b[1]]
    cta_order = [1, 0]
```
- **EN:** Prepares or updates state through `BLOCK_M`, `BLOCK_N`, `BLOCK_K`, `cta_split_a`, `cta_split_b`, `cta_order`.
- **CN:** 通过 `BLOCK_M`、`BLOCK_N`、`BLOCK_K`、`cta_split_a`、`cta_split_b`、`cta_order` 准备或更新状态。

#### Lines 483-491

```python

    if two_ctas:
        cga_layout_a = make_2cta_cga_layout(ctas_per_cga, cta_split_a, cta_order, 0)
        cga_layout_b = make_2cta_cga_layout(ctas_per_cga_b, cta_split_b, cta_order, 1)
        cga_layout_c = make_2cta_cga_layout(ctas_per_cga, ctas_per_cga, cta_order, 0)
    else:
        cga_layout_a = make_cga_layout(ctas_per_cga, cta_split_a, cta_order)
        cga_layout_b = make_cga_layout(ctas_per_cga_b, cta_split_b, cta_order)
        cga_layout_c = make_cga_layout(ctas_per_cga, ctas_per_cga, cta_order)
```
- **EN:** Invokes `make_2cta_cga_layout`, `make_cga_layout` to execute the test logic. Branches on runtime or test conditions. Relevant themes: layout transformation reasoning.
- **CN:** 调用 `make_2cta_cga_layout`、`make_cga_layout` 执行测试逻辑。 根据运行时或测试条件进行分支。 相关主题：布局变换推理。

#### Lines 492-523

```python

    shared_layout_a = ttgl.NVMMASharedLayout.get_default_for([BLOCK_M, BLOCK_K], ttgl.float16, cga_layout=cga_layout_a)
    shared_layout_b = ttgl.NVMMASharedLayout.get_default_for([BLOCK_K, BLOCK_N], ttgl.float16, cga_layout=cga_layout_b)

    a = torch.randn((BLOCK_M, BLOCK_K), dtype=torch.float16, device="cuda")
    b = torch.randn((BLOCK_K, BLOCK_N), dtype=torch.float16, device="cuda")
    out = torch.empty((BLOCK_M, BLOCK_N), dtype=torch.float32, device="cuda")

    a_desc = gluon.nvidia.hopper.TensorDescriptor.from_tensor(a, [BLOCK_M, BLOCK_K], shared_layout_a)
    b_desc = gluon.nvidia.hopper.TensorDescriptor.from_tensor(b, [BLOCK_K, BLOCK_N], shared_layout_b)

    tmem_shape = (128, BLOCK_N // ctas_per_cga[1])
    acc_tmem_layout = TensorMemoryLayout(
        block=tmem_shape,
        col_stride=1,
        two_ctas=two_ctas,
        cga_layout=cga_layout_c,
    )
    blocked_c = ttgl.BlockedLayout([1, 2], [ctas_per_cga[1], 32 // ctas_per_cga[1]], [4, 1], [1, 0],
                                   cga_layout=cga_layout_c)

    compiled = tcgen05_mma_multicast_commit_kernel[(1, )](
        a_desc,
        b_desc,
        out,
        BLOCK_M,
        BLOCK_N,
        acc_tmem_layout,
        blocked_c,
        num_warps=4,
        num_ctas=ctas_per_cga[0] * ctas_per_cga[1],
    )
```
- **EN:** Prepares or updates state through `shared_layout_a`, `shared_layout_b`, `a`, `b`, `out`, `a_desc`, `b_desc`, `tmem_shape`, and 3 more. Invokes `ttgl.NVMMASharedLayout.get_default_for`, `torch.randn`, `torch.empty`, `gluon.nvidia.hopper.TensorDescriptor.from_tensor`, `TensorMemoryLayout`, `ttgl.BlockedLayout` to execute the test logic. Relevant themes: PyTorch tensor setup and checks, tensor/descriptor metadata, layout transformation reasoning, random-data generation.
- **CN:** 通过 `shared_layout_a`、`shared_layout_b`、`a`、`b`、`out`、`a_desc`、`b_desc`、`tmem_shape` 等另外 3 项 准备或更新状态。 调用 `ttgl.NVMMASharedLayout.get_default_for`、`torch.randn`、`torch.empty`、`gluon.nvidia.hopper.TensorDescriptor.from_tensor`、`TensorMemoryLayout`、`ttgl.BlockedLayout` 执行测试逻辑。 相关主题：PyTorch 张量准备与校验、张量/描述符元数据、布局变换推理、随机数据生成。

#### Lines 524-525

```python

    assert "tcgen05.commit.cta_group::" + ("2" if two_ctas else "1") in compiled.asm["ptx"]
```
- **EN:** Validates behavior with 1 assertion(s).
- **CN:** 通过 1 个断言验证行为。

#### Lines 526-527

```python
    if two_ctas:
        assert "fence.mbarrier_init.release.cluster" in compiled.asm["ptx"]
```
- **EN:** Validates behavior with 1 assertion(s). Branches on runtime or test conditions.
- **CN:** 通过 1 个断言验证行为。 根据运行时或测试条件进行分支。

#### Lines 528-530

```python
    # For [2, 1] and two_ctas we don't multicast as there are not enough tiles
    # but we do a commit.multicast::cluster so let's grep that one instead
    assert ("multicast::cluster" in compiled.asm["ptx"])
```
- **EN:** Validates behavior with 1 assertion(s).
- **CN:** 通过 1 个断言验证行为。

#### Lines 531-531

```python
    torch.testing.assert_close(out, torch.matmul(a.to(out.dtype), b.to(out.dtype)))
```
- **EN:** Invokes `torch.testing.assert_close`, `torch.matmul`, `a.to`, `b.to` to execute the test logic. Relevant themes: PyTorch tensor setup and checks, matrix multiplication workflows.
- **CN:** 调用 `torch.testing.assert_close`、`torch.matmul`、`a.to`、`b.to` 执行测试逻辑。 相关主题：PyTorch 张量准备与校验、矩阵乘法工作流。

### Lines 532-535

```python


@pytest.mark.skipif(not is_blackwell(), reason="Requires Blackwell")
def test_tma_rejects_wrong_barrier_cga_layout_in_two_cta_kernel(capfd):
```
- **EN:** Defines the test function `test_tma_rejects_wrong_barrier_cga_layout_in_two_cta_kernel`. Decorators: `pytest.mark.skipif(not is_blackwell(), reason='Requires Blackwell')`. Parameters: `capfd`. Key calls include `pytest.mark.skipif`, `make_2cta_cga_layout`, `torch.empty`, `ttgl.NVMMASharedLayout.get_default_for`, `TensorDescriptor.from_tensor`, `TensorMemoryLayout`, and 4 more. This scope touches PyTorch tensor setup and checks, tensor/descriptor metadata, layout transformation reasoning.
- **CN:** 定义测试函数 `test_tma_rejects_wrong_barrier_cga_layout_in_two_cta_kernel`。 装饰器：`pytest.mark.skipif(not is_blackwell(), reason='Requires Blackwell')`。 参数：`capfd`。 关键调用包括 `pytest.mark.skipif`、`make_2cta_cga_layout`、`torch.empty`、`ttgl.NVMMASharedLayout.get_default_for`、`TensorDescriptor.from_tensor`、`TensorMemoryLayout` 等另外 4 项。 该作用域涉及PyTorch 张量准备与校验、张量/描述符元数据、布局变换推理。

#### Lines 536-556

```python
    ctas_per_cga = [2, 2]
    ctas_per_cga_b = [1, 4]
    block_m = 128 * ctas_per_cga[0]
    block_n = 64 * ctas_per_cga_b[1]
    block_k = 32
    cga_layout_a = make_2cta_cga_layout(ctas_per_cga, [ctas_per_cga[0], 1], [1, 0], 0)
    cga_layout_b = make_2cta_cga_layout(ctas_per_cga_b, [1, ctas_per_cga_b[1]], [1, 0], 1)
    cga_layout_c = make_2cta_cga_layout(ctas_per_cga, ctas_per_cga, [1, 0], 0)

    a = torch.empty((block_m, block_k), dtype=torch.float16, device="cuda")
    b = torch.empty((block_k, block_n), dtype=torch.float16, device="cuda")
    a_layout = ttgl.NVMMASharedLayout.get_default_for([block_m, block_k], ttgl.float16, cga_layout=cga_layout_a)
    b_layout = ttgl.NVMMASharedLayout.get_default_for([block_k, block_n], ttgl.float16, cga_layout=cga_layout_b)
    a_desc = TensorDescriptor.from_tensor(a, [block_m, block_k], a_layout)
    b_desc = TensorDescriptor.from_tensor(b, [block_k, block_n], b_layout)
    acc_tmem_layout = TensorMemoryLayout(
        block=(128, block_n // ctas_per_cga[1]),
        col_stride=1,
        two_ctas=True,
        cga_layout=cga_layout_c,
    )
```
- **EN:** Prepares or updates state through `ctas_per_cga`, `ctas_per_cga_b`, `block_m`, `block_n`, `block_k`, `cga_layout_a`, `cga_layout_b`, `cga_layout_c`, and 7 more. Invokes `make_2cta_cga_layout`, `torch.empty`, `ttgl.NVMMASharedLayout.get_default_for`, `TensorDescriptor.from_tensor`, `TensorMemoryLayout` to execute the test logic. Relevant themes: PyTorch tensor setup and checks, tensor/descriptor metadata, layout transformation reasoning.
- **CN:** 通过 `ctas_per_cga`、`ctas_per_cga_b`、`block_m`、`block_n`、`block_k`、`cga_layout_a`、`cga_layout_b`、`cga_layout_c` 等另外 7 项 准备或更新状态。 调用 `make_2cta_cga_layout`、`torch.empty`、`ttgl.NVMMASharedLayout.get_default_for`、`TensorDescriptor.from_tensor`、`TensorMemoryLayout` 执行测试逻辑。 相关主题：PyTorch 张量准备与校验、张量/描述符元数据、布局变换推理。

#### Lines 557-568

```python

    with pytest.raises(RuntimeError, match="PassManager::run failed"):
        tma_wrong_barrier_cga_layout_kernel.warmup(
            a_desc,
            b_desc,
            block_m,
            block_n,
            acc_tmem_layout,
            grid=(1, ),
            num_warps=4,
            num_ctas=4,
        )
```
- **EN:** Invokes `pytest.raises`, `tma_wrong_barrier_cga_layout_kernel.warmup` to execute the test logic. Uses `pytest.raises` to confirm expected failure paths. Uses context managers to control resources or expectations. Relevant themes: layout transformation reasoning.
- **CN:** 调用 `pytest.raises`、`tma_wrong_barrier_cga_layout_kernel.warmup` 执行测试逻辑。 使用 `pytest.raises` 确认预期的失败路径。 使用上下文管理器控制资源或预期行为。 相关主题：布局变换推理。

#### Lines 569-569

```python
    captured = capfd.readouterr()
```
- **EN:** Prepares or updates state through `captured`. Invokes `capfd.readouterr` to execute the test logic.
- **CN:** 通过 `captured` 准备或更新状态。 调用 `capfd.readouterr` 执行测试逻辑。

#### Lines 570-570

```python
    assert "TMA barrier cga_layout must be [[1], [2]] or [[0], [1]], got [[0], [0]]" in (captured.out + captured.err)
```
- **EN:** Validates behavior with 1 assertion(s). Relevant themes: layout transformation reasoning.
- **CN:** 通过 1 个断言验证行为。 相关主题：布局变换推理。

### Lines 571-578

```python


@gluon.jit
def tcgen05_mma_scaled_direct_multicast_kernel(a_desc, b_desc, out_ptr, BLOCK_M: ttgl.constexpr,
                                               BLOCK_N: ttgl.constexpr, BLOCK_K: ttgl.constexpr,
                                               NUM_K_TILES: ttgl.constexpr, cga_layout_a: ttgl.constexpr,
                                               cga_layout_b: ttgl.constexpr, cga_layout_c: ttgl.constexpr,
                                               blocked_c: ttgl.constexpr):
```
- **EN:** Defines the helper function `tcgen05_mma_scaled_direct_multicast_kernel`. Decorators: `gluon.jit`. Parameters: `a_desc`, `b_desc`, `out_ptr`, `BLOCK_M`, `BLOCK_N`, `BLOCK_K`, `NUM_K_TILES`, `cga_layout_a`, and 3 more. Key calls include `ttgl.allocate_shared_memory`, `TensorMemoryScalesLayout`, `allocate_tensor_memory`, `a_scale.store`, `b_scale.store`, `TensorMemoryLayout`, and 15 more. This scope touches layout transformation reasoning.
- **CN:** 定义辅助函数 `tcgen05_mma_scaled_direct_multicast_kernel`。 装饰器：`gluon.jit`。 参数：`a_desc`、`b_desc`、`out_ptr`、`BLOCK_M`、`BLOCK_N`、`BLOCK_K`、`NUM_K_TILES`、`cga_layout_a` 等另外 3 项。 关键调用包括 `ttgl.allocate_shared_memory`、`TensorMemoryScalesLayout`、`allocate_tensor_memory`、`a_scale.store`、`b_scale.store`、`TensorMemoryLayout` 等另外 15 项。 该作用域涉及布局变换推理。

#### Lines 579-598

```python
    smem_a = ttgl.allocate_shared_memory(a_desc.dtype, a_desc.block_shape, a_desc.layout)
    smem_b = ttgl.allocate_shared_memory(b_desc.dtype, b_desc.block_shape, b_desc.layout)

    a_scale_layout: ttgl.constexpr = TensorMemoryScalesLayout(cga_layout=cga_layout_a)
    b_scale_layout: ttgl.constexpr = TensorMemoryScalesLayout(cga_layout=cga_layout_b)
    a_scale = allocate_tensor_memory(ttgl.uint8, [BLOCK_M, BLOCK_K // 32], a_scale_layout)
    b_scale = allocate_tensor_memory(ttgl.uint8, [BLOCK_N, BLOCK_K // 32], b_scale_layout)
    a_scale.store(ttgl.full([BLOCK_M, BLOCK_K // 32], 127, ttgl.uint8, a_scale.get_reg_layout()))
    b_scale.store(ttgl.full([BLOCK_N, BLOCK_K // 32], 127, ttgl.uint8, b_scale.get_reg_layout()))

    acc_layout: ttgl.constexpr = TensorMemoryLayout([128, 128], col_stride=1, cga_layout=cga_layout_c, two_ctas=True)
    acc = allocate_tensor_memory(ttgl.float32, [BLOCK_M, BLOCK_N], acc_layout)

    tma_bar = mbarrier.allocate_mbarrier(two_ctas=True)
    mbarrier.init(tma_bar, count=1)
    mma_bar = mbarrier.allocate_mbarrier()
    mbarrier.init(mma_bar, count=tcgen05_mma_barrier_count([smem_a, smem_b], True, acc.type.layout.two_ctas))

    phase_tma = 0
    phase_mma = 0
```
- **EN:** Prepares or updates state through `smem_a`, `smem_b`, `a_scale_layout`, `b_scale_layout`, `a_scale`, `b_scale`, `acc_layout`, `acc`, and 4 more. Invokes `ttgl.allocate_shared_memory`, `TensorMemoryScalesLayout`, `allocate_tensor_memory`, `a_scale.store`, `ttgl.full`, `a_scale.get_reg_layout`, and 6 more to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `smem_a`、`smem_b`、`a_scale_layout`、`b_scale_layout`、`a_scale`、`b_scale`、`acc_layout`、`acc` 等另外 4 项 准备或更新状态。 调用 `ttgl.allocate_shared_memory`、`TensorMemoryScalesLayout`、`allocate_tensor_memory`、`a_scale.store`、`ttgl.full`、`a_scale.get_reg_layout` 等另外 6 项 执行测试逻辑。 相关主题：布局变换推理。

#### Lines 599-609

```python
    for k in range(NUM_K_TILES):
        offs_k = k * BLOCK_K
        mbarrier.expect(tma_bar, a_desc.nbytes_per_cta + b_desc.nbytes_per_cta)
        tma.async_load(a_desc, [0, offs_k], tma_bar, smem_a, multicast=True)
        tma.async_load(b_desc, [offs_k, 0], tma_bar, smem_b, multicast=True)
        mbarrier.wait(tma_bar, phase_tma, deps=[smem_a, smem_b])
        tcgen05_mma_scaled(smem_a, smem_b, acc, a_scale, b_scale, "e5m2", "e5m2", use_acc=k != 0, multicast=True,
                           mbarriers=[mma_bar])
        mbarrier.wait(mma_bar, phase_mma, deps=[smem_a, smem_b])
        phase_tma ^= 1
        phase_mma ^= 1
```
- **EN:** Invokes `mbarrier.expect`, `tma.async_load`, `mbarrier.wait`, `tcgen05_mma_scaled` to execute the test logic. Iterates across cases or data tiles.
- **CN:** 调用 `mbarrier.expect`、`tma.async_load`、`mbarrier.wait`、`tcgen05_mma_scaled` 执行测试逻辑。 通过循环覆盖多个用例或数据分块。

#### Lines 610-617

```python

    mbarrier.invalidate(tma_bar)
    mbarrier.invalidate(mma_bar)

    out = ttgl.convert_layout(acc.load(), blocked_c)
    out_offs_m = ttgl.arange(0, BLOCK_M)[:, None]
    out_offs_n = ttgl.arange(0, BLOCK_N)[None, :]
    ttgl.store(out_ptr + out_offs_m * BLOCK_N + out_offs_n, out)
```
- **EN:** Prepares or updates state through `out`, `out_offs_m`, `out_offs_n`. Invokes `mbarrier.invalidate`, `ttgl.convert_layout`, `acc.load`, `ttgl.arange`, `ttgl.store` to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `out`、`out_offs_m`、`out_offs_n` 准备或更新状态。 调用 `mbarrier.invalidate`、`ttgl.convert_layout`、`acc.load`、`ttgl.arange`、`ttgl.store` 执行测试逻辑。 相关主题：布局变换推理。

### Lines 618-621

```python


@pytest.mark.skipif(not is_blackwell(), reason="Requires Blackwell")
def test_tcgen05_mma_scaled_direct_multicast_barrier():
```
- **EN:** Defines the test function `test_tcgen05_mma_scaled_direct_multicast_barrier`. Decorators: `pytest.mark.skipif(not is_blackwell(), reason='Requires Blackwell')`. Key calls include `pytest.mark.skipif`, `ttgl.NVMMASharedLayout.get_default_for`, `ttgl.BlockedLayout`, `torch.empty`, `TensorDescriptor.from_tensor`, `torch.testing.assert_close`, and 5 more. This scope touches PyTorch tensor setup and checks, tensor/descriptor metadata, matrix multiplication workflows, layout transformation reasoning.
- **CN:** 定义测试函数 `test_tcgen05_mma_scaled_direct_multicast_barrier`。 装饰器：`pytest.mark.skipif(not is_blackwell(), reason='Requires Blackwell')`。 关键调用包括 `pytest.mark.skipif`、`ttgl.NVMMASharedLayout.get_default_for`、`ttgl.BlockedLayout`、`torch.empty`、`TensorDescriptor.from_tensor`、`torch.testing.assert_close` 等另外 5 项。 该作用域涉及PyTorch 张量准备与校验、张量/描述符元数据、矩阵乘法工作流、布局变换推理。

#### Lines 622-645

```python
    num_ctas = 4
    BLOCK_M = 256
    BLOCK_N = 256
    BLOCK_K = 128
    NUM_K_TILES = 4
    cga_layout_a = ((1, 0), (0, 0))
    cga_layout_b = ((0, 1), (0, 2))
    cga_layout_c = ((1, 0), (0, 1))
    shared_layout_a = ttgl.NVMMASharedLayout.get_default_for([BLOCK_M, BLOCK_K], ttgl.float8e5, cga_layout=cga_layout_a)
    shared_layout_b = ttgl.NVMMASharedLayout.get_default_for([BLOCK_K, BLOCK_N], ttgl.float8e5, cga_layout=cga_layout_b)
    blocked_c = ttgl.BlockedLayout([1, 2], [2, 16], [4, 1], [1, 0], cga_layout=cga_layout_c)

    a = torch.randint(20, 40, (BLOCK_M, NUM_K_TILES * BLOCK_K), device="cuda",
                      dtype=torch.uint8).view(torch.float8_e5m2)
    b = torch.randint(20, 40, (NUM_K_TILES * BLOCK_K, BLOCK_N), device="cuda",
                      dtype=torch.uint8).view(torch.float8_e5m2)
    out = torch.empty((BLOCK_M, BLOCK_N), device="cuda", dtype=torch.float32)
    a_desc = TensorDescriptor.from_tensor(a, [BLOCK_M, BLOCK_K], shared_layout_a)
    b_desc = TensorDescriptor.from_tensor(b, [BLOCK_K, BLOCK_N], shared_layout_b)

    tcgen05_mma_scaled_direct_multicast_kernel[(1, )](a_desc, b_desc, out, BLOCK_M, BLOCK_N, BLOCK_K, NUM_K_TILES,
                                                      cga_layout_a, cga_layout_b, cga_layout_c, blocked_c, num_warps=4,
                                                      num_ctas=num_ctas)
    torch.testing.assert_close(out, torch.matmul(a.to(torch.float32), b.to(torch.float32)), atol=5e-2, rtol=1e-3)
```
- **EN:** Prepares or updates state through `num_ctas`, `BLOCK_M`, `BLOCK_N`, `BLOCK_K`, `NUM_K_TILES`, `cga_layout_a`, `cga_layout_b`, `cga_layout_c`, and 8 more. Invokes `ttgl.NVMMASharedLayout.get_default_for`, `ttgl.BlockedLayout`, `torch.randint`, `torch.empty`, `TensorDescriptor.from_tensor`, `torch.testing.assert_close`, and 3 more to execute the test logic. Relevant themes: PyTorch tensor setup and checks, tensor/descriptor metadata, matrix multiplication workflows, layout transformation reasoning.
- **CN:** 通过 `num_ctas`、`BLOCK_M`、`BLOCK_N`、`BLOCK_K`、`NUM_K_TILES`、`cga_layout_a`、`cga_layout_b`、`cga_layout_c` 等另外 8 项 准备或更新状态。 调用 `ttgl.NVMMASharedLayout.get_default_for`、`ttgl.BlockedLayout`、`torch.randint`、`torch.empty`、`TensorDescriptor.from_tensor`、`torch.testing.assert_close` 等另外 3 项 执行测试逻辑。 相关主题：PyTorch 张量准备与校验、张量/描述符元数据、矩阵乘法工作流、布局变换推理。

### Lines 646-649

```python


@gluon.jit
def async_copy_mbarrier_kernel(out, inp, xnumel, XBLOCK: ttgl.constexpr, YBLOCK: ttgl.constexpr):
```
- **EN:** Defines the helper function `async_copy_mbarrier_kernel`. Decorators: `gluon.jit`. Parameters: `out`, `inp`, `xnumel`, `XBLOCK`, `YBLOCK`. Key calls include `ttgl.allocate_shared_memory`, `ttgl.BlockedLayout`, `async_copy.async_copy_global_to_shared`, `mbarrier.init`, `async_copy.mbarrier_arrive`, `mbarrier.arrive`, and 7 more. This scope touches layout transformation reasoning.
- **CN:** 定义辅助函数 `async_copy_mbarrier_kernel`。 装饰器：`gluon.jit`。 参数：`out`、`inp`、`xnumel`、`XBLOCK`、`YBLOCK`。 关键调用包括 `ttgl.allocate_shared_memory`、`ttgl.BlockedLayout`、`async_copy.async_copy_global_to_shared`、`mbarrier.init`、`async_copy.mbarrier_arrive`、`mbarrier.arrive` 等另外 7 项。 该作用域涉及布局变换推理。

#### Lines 650-668

```python
    smem = ttgl.allocate_shared_memory(inp.dtype.element_ty, [XBLOCK, YBLOCK],
                                       ttgl.SwizzledSharedLayout(1, 1, 1, order=[1, 0]))
    block_layout: ttgl.constexpr = ttgl.BlockedLayout([1, 4], [1, 32], [4, 1], [1, 0])
    xindex = ttgl.arange(0, XBLOCK, ttgl.SliceLayout(1, block_layout))[:, None]
    yindex = ttgl.arange(0, YBLOCK, ttgl.SliceLayout(0, block_layout))[None, :]
    mask = xindex < xnumel
    async_copy.async_copy_global_to_shared(
        smem,
        inp + xindex * YBLOCK + yindex,
        mask,
    )
    mbar = ttgl.allocate_shared_memory(ttgl.int64, [1], mbarrier.MBarrierLayout())
    mbarrier.init(mbar, count=1)
    async_copy.mbarrier_arrive(mbar)
    mbarrier.arrive(mbar)
    mbarrier.wait(mbar, 0)

    val = smem.load(block_layout)
    ttgl.store(out + xindex * YBLOCK + yindex, val)
```
- **EN:** Prepares or updates state through `smem`, `block_layout`, `xindex`, `yindex`, `mask`, `mbar`, `val`. Invokes `ttgl.allocate_shared_memory`, `ttgl.SwizzledSharedLayout`, `ttgl.BlockedLayout`, `ttgl.arange`, `ttgl.SliceLayout`, `async_copy.async_copy_global_to_shared`, and 7 more to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `smem`、`block_layout`、`xindex`、`yindex`、`mask`、`mbar`、`val` 准备或更新状态。 调用 `ttgl.allocate_shared_memory`、`ttgl.SwizzledSharedLayout`、`ttgl.BlockedLayout`、`ttgl.arange`、`ttgl.SliceLayout`、`async_copy.async_copy_global_to_shared` 等另外 7 项 执行测试逻辑。 相关主题：布局变换推理。

### Lines 669-672

```python


@pytest.mark.skipif(not is_ampere_or_newer(), reason="Requires Ampere")
def test_async_copy_mbarrier():
```
- **EN:** Defines the test function `test_async_copy_mbarrier`. Decorators: `pytest.mark.skipif(not is_ampere_or_newer(), reason='Requires Ampere')`. Key calls include `pytest.mark.skipif`, `torch.empty`, `torch.randn`, `torch.testing.assert_close`, `torch.zeros`, `is_ampere_or_newer`. This scope touches PyTorch tensor setup and checks, random-data generation.
- **CN:** 定义测试函数 `test_async_copy_mbarrier`。 装饰器：`pytest.mark.skipif(not is_ampere_or_newer(), reason='Requires Ampere')`。 关键调用包括 `pytest.mark.skipif`、`torch.empty`、`torch.randn`、`torch.testing.assert_close`、`torch.zeros`、`is_ampere_or_newer`。 该作用域涉及PyTorch 张量准备与校验、随机数据生成。

#### Lines 673-678

```python
    tensor_opts = dict(dtype=torch.float, device="cuda")
    out = torch.empty((32, 32), **tensor_opts)
    inp = torch.randn((20, 32), **tensor_opts)
    async_copy_mbarrier_kernel[(1, )](out, inp, inp.shape[0], XBLOCK=32, YBLOCK=32)
    torch.testing.assert_close(out[:20], inp)
    torch.testing.assert_close(out[20:], torch.zeros((12, 32), **tensor_opts))
```
- **EN:** Prepares or updates state through `tensor_opts`, `out`, `inp`. Invokes `torch.empty`, `torch.randn`, `torch.testing.assert_close`, `torch.zeros` to execute the test logic. Relevant themes: PyTorch tensor setup and checks, random-data generation.
- **CN:** 通过 `tensor_opts`、`out`、`inp` 准备或更新状态。 调用 `torch.empty`、`torch.randn`、`torch.testing.assert_close`、`torch.zeros` 执行测试逻辑。 相关主题：PyTorch 张量准备与校验、随机数据生成。

### Lines 679-684

```python


@pytest.mark.skipif(not is_hopper_or_newer(), reason="Requires Hopper")
def test_device_tma_load():

    @gluon.jit
```
- **EN:** Defines the test function `test_device_tma_load`. Decorators: `pytest.mark.skipif(not is_hopper_or_newer(), reason='Requires Hopper')`. Nested definitions in this scope: `tma_device_load_kernel`, `alloc_fn`. Key calls include `pytest.mark.skipif`, `torch.zeros`, `torch.ones_like`, `ttgl.NVMMASharedLayout`, `triton.set_allocator`, `torch.testing.assert_close`, and 15 more. This scope touches PyTorch tensor setup and checks, tensor/descriptor metadata, layout transformation reasoning.
- **CN:** 定义测试函数 `test_device_tma_load`。 装饰器：`pytest.mark.skipif(not is_hopper_or_newer(), reason='Requires Hopper')`。 该作用域中的嵌套定义：`tma_device_load_kernel`、`alloc_fn`。 关键调用包括 `pytest.mark.skipif`、`torch.zeros`、`torch.ones_like`、`ttgl.NVMMASharedLayout`、`triton.set_allocator`、`torch.testing.assert_close` 等另外 15 项。 该作用域涉及PyTorch 张量准备与校验、张量/描述符元数据、布局变换推理。

#### Lines 684-685

```python
    @gluon.jit
    def tma_device_load_kernel(input_ptr, output_ptr, XBLOCK: ttgl.constexpr, smem_layout: ttgl.constexpr):
```
- **EN:** Defines the helper function `tma_device_load_kernel`. Decorators: `gluon.jit`. Parameters: `input_ptr`, `output_ptr`, `XBLOCK`, `smem_layout`. Key calls include `tma.make_tensor_descriptor`, `ttgl.allocate_shared_memory`, `mbarrier.init`, `mbarrier.expect`, `tma.async_load`, `mbarrier.wait`, and 7 more. This scope touches tensor/descriptor metadata, layout transformation reasoning.
- **CN:** 定义辅助函数 `tma_device_load_kernel`。 装饰器：`gluon.jit`。 参数：`input_ptr`、`output_ptr`、`XBLOCK`、`smem_layout`。 关键调用包括 `tma.make_tensor_descriptor`、`ttgl.allocate_shared_memory`、`mbarrier.init`、`mbarrier.expect`、`tma.async_load`、`mbarrier.wait` 等另外 7 项。 该作用域涉及张量/描述符元数据、布局变换推理。

##### Lines 686-707

```python
        input_desc = tma.make_tensor_descriptor(
            input_ptr,
            shape=[XBLOCK, XBLOCK],
            strides=[XBLOCK, 1],
            block_shape=[XBLOCK, XBLOCK],
            layout=smem_layout,
        )

        smem = ttgl.allocate_shared_memory(ttgl.float16, [XBLOCK, XBLOCK], smem_layout)
        bar = ttgl.allocate_shared_memory(ttgl.int64, [1], mbarrier.MBarrierLayout())
        mbarrier.init(bar, count=1)

        mbarrier.expect(bar, input_desc.nbytes_per_cta)
        tma.async_load(input_desc, [0, 0], bar, smem)
        mbarrier.wait(bar, 0)
        mbarrier.invalidate(bar)

        block_layout: ttgl.constexpr = ttgl.BlockedLayout([1, 2], [4, 8], [4, 1], [1, 0])
        xindex = ttgl.arange(0, XBLOCK, ttgl.SliceLayout(1, block_layout))[:, None]
        yindex = ttgl.arange(0, XBLOCK, ttgl.SliceLayout(0, block_layout))[None, :]
        val = smem.load(block_layout)
        ttgl.store(output_ptr + yindex + xindex * XBLOCK, val)
```
- **EN:** Prepares or updates state through `input_desc`, `smem`, `bar`, `block_layout`, `xindex`, `yindex`, `val`. Invokes `tma.make_tensor_descriptor`, `ttgl.allocate_shared_memory`, `mbarrier.MBarrierLayout`, `mbarrier.init`, `mbarrier.expect`, `tma.async_load`, and 7 more to execute the test logic. Relevant themes: tensor/descriptor metadata, layout transformation reasoning.
- **CN:** 通过 `input_desc`、`smem`、`bar`、`block_layout`、`xindex`、`yindex`、`val` 准备或更新状态。 调用 `tma.make_tensor_descriptor`、`ttgl.allocate_shared_memory`、`mbarrier.MBarrierLayout`、`mbarrier.init`、`mbarrier.expect`、`tma.async_load` 等另外 7 项 执行测试逻辑。 相关主题：张量/描述符元数据、布局变换推理。

#### Lines 708-718

```python

    XBLOCK = 16
    input = torch.zeros((XBLOCK, XBLOCK), device="cuda", dtype=torch.float16)
    output = torch.ones_like(input)
    smem_layout = ttgl.NVMMASharedLayout(
        swizzle_byte_width=32,
        element_bitwidth=16,
        rank=2,
        transposed=False,
        fp4_padded=False,
    )
```
- **EN:** Prepares or updates state through `XBLOCK`, `input`, `output`, `smem_layout`. Invokes `torch.zeros`, `torch.ones_like`, `ttgl.NVMMASharedLayout` to execute the test logic. Relevant themes: PyTorch tensor setup and checks, layout transformation reasoning.
- **CN:** 通过 `XBLOCK`、`input`、`output`、`smem_layout` 准备或更新状态。 调用 `torch.zeros`、`torch.ones_like`、`ttgl.NVMMASharedLayout` 执行测试逻辑。 相关主题：PyTorch 张量准备与校验、布局变换推理。

#### Lines 719-720

```python

    def alloc_fn(size: int, alignment: int, stream: int):
```
- **EN:** Defines the helper function `alloc_fn`. Parameters: `size`, `alignment`, `stream`. Key calls include `torch.empty`. This scope touches PyTorch tensor setup and checks.
- **CN:** 定义辅助函数 `alloc_fn`。 参数：`size`、`alignment`、`stream`。 关键调用包括 `torch.empty`。 该作用域涉及PyTorch 张量准备与校验。

##### Lines 721-721

```python
        return torch.empty(size, device="cuda", dtype=torch.int8)
```
- **EN:** Invokes `torch.empty` to execute the test logic. Relevant themes: PyTorch tensor setup and checks.
- **CN:** 调用 `torch.empty` 执行测试逻辑。 相关主题：PyTorch 张量准备与校验。

#### Lines 722-726

```python

    triton.set_allocator(alloc_fn)

    tma_device_load_kernel[(1, )](input, output, XBLOCK, smem_layout)
    torch.testing.assert_close(input, output)
```
- **EN:** Invokes `triton.set_allocator`, `torch.testing.assert_close` to execute the test logic. Relevant themes: PyTorch tensor setup and checks, layout transformation reasoning.
- **CN:** 调用 `triton.set_allocator`、`torch.testing.assert_close` 执行测试逻辑。 相关主题：PyTorch 张量准备与校验、布局变换推理。

### Lines 727-732

```python


@pytest.mark.skipif(not is_hopper_or_newer(), reason="Requires Hopper")
def test_device_tma_store():

    @gluon.jit
```
- **EN:** Defines the test function `test_device_tma_store`. Decorators: `pytest.mark.skipif(not is_hopper_or_newer(), reason='Requires Hopper')`. Nested definitions in this scope: `tma_device_store_kernel`, `alloc_fn`. Key calls include `pytest.mark.skipif`, `torch.ones`, `ttgl.NVMMASharedLayout`, `triton.set_allocator`, `torch.testing.assert_close`, `ttgl.BlockedLayout`, and 9 more. This scope touches PyTorch tensor setup and checks, tensor/descriptor metadata, layout transformation reasoning.
- **CN:** 定义测试函数 `test_device_tma_store`。 装饰器：`pytest.mark.skipif(not is_hopper_or_newer(), reason='Requires Hopper')`。 该作用域中的嵌套定义：`tma_device_store_kernel`、`alloc_fn`。 关键调用包括 `pytest.mark.skipif`、`torch.ones`、`ttgl.NVMMASharedLayout`、`triton.set_allocator`、`torch.testing.assert_close`、`ttgl.BlockedLayout` 等另外 9 项。 该作用域涉及PyTorch 张量准备与校验、张量/描述符元数据、布局变换推理。

#### Lines 732-733

```python
    @gluon.jit
    def tma_device_store_kernel(out_ptr, XBLOCK: ttgl.constexpr, smem_layout: ttgl.constexpr):
```
- **EN:** Defines the helper function `tma_device_store_kernel`. Decorators: `gluon.jit`. Parameters: `out_ptr`, `XBLOCK`, `smem_layout`. Key calls include `ttgl.BlockedLayout`, `ttgl.full`, `ttgl.allocate_shared_memory`, `tma.make_tensor_descriptor`, `tma.async_copy_shared_to_global`, `tma.store_wait`, and 1 more. This scope touches tensor/descriptor metadata, layout transformation reasoning.
- **CN:** 定义辅助函数 `tma_device_store_kernel`。 装饰器：`gluon.jit`。 参数：`out_ptr`、`XBLOCK`、`smem_layout`。 关键调用包括 `ttgl.BlockedLayout`、`ttgl.full`、`ttgl.allocate_shared_memory`、`tma.make_tensor_descriptor`、`tma.async_copy_shared_to_global`、`tma.store_wait` 等另外 1 项。 该作用域涉及张量/描述符元数据、布局变换推理。

##### Lines 734-746

```python
        layout: ttgl.constexpr = ttgl.BlockedLayout([1, 2], [4, 8], [4, 1], [1, 0])
        value = ttgl.full([XBLOCK, XBLOCK], 0, ttgl.float16, layout)
        alloc = ttgl.allocate_shared_memory(ttgl.float16, [XBLOCK, XBLOCK], smem_layout, value)
        out_desc = tma.make_tensor_descriptor(
            out_ptr,
            shape=[XBLOCK, XBLOCK],
            strides=[XBLOCK, 1],
            block_shape=[XBLOCK, XBLOCK],
            layout=smem_layout,
        )
        tma.async_copy_shared_to_global(out_desc, [0, 0], alloc)
        tma.store_wait(0)
        alloc._keep_alive()
```
- **EN:** Prepares or updates state through `layout`, `value`, `alloc`, `out_desc`. Invokes `ttgl.BlockedLayout`, `ttgl.full`, `ttgl.allocate_shared_memory`, `tma.make_tensor_descriptor`, `tma.async_copy_shared_to_global`, `tma.store_wait`, and 1 more to execute the test logic. Relevant themes: tensor/descriptor metadata, layout transformation reasoning.
- **CN:** 通过 `layout`、`value`、`alloc`、`out_desc` 准备或更新状态。 调用 `ttgl.BlockedLayout`、`ttgl.full`、`ttgl.allocate_shared_memory`、`tma.make_tensor_descriptor`、`tma.async_copy_shared_to_global`、`tma.store_wait` 等另外 1 项 执行测试逻辑。 相关主题：张量/描述符元数据、布局变换推理。

#### Lines 747-756

```python

    XBLOCK = 16
    out = torch.ones((XBLOCK, XBLOCK), dtype=torch.float16, device="cuda")
    smem_layout = ttgl.NVMMASharedLayout(
        swizzle_byte_width=32,
        element_bitwidth=16,
        rank=2,
        transposed=False,
        fp4_padded=False,
    )
```
- **EN:** Prepares or updates state through `XBLOCK`, `out`, `smem_layout`. Invokes `torch.ones`, `ttgl.NVMMASharedLayout` to execute the test logic. Relevant themes: PyTorch tensor setup and checks, layout transformation reasoning.
- **CN:** 通过 `XBLOCK`、`out`、`smem_layout` 准备或更新状态。 调用 `torch.ones`、`ttgl.NVMMASharedLayout` 执行测试逻辑。 相关主题：PyTorch 张量准备与校验、布局变换推理。

#### Lines 757-758

```python

    def alloc_fn(size: int, alignment: int, stream: int):
```
- **EN:** Defines the helper function `alloc_fn`. Parameters: `size`, `alignment`, `stream`. Key calls include `torch.empty`. This scope touches PyTorch tensor setup and checks.
- **CN:** 定义辅助函数 `alloc_fn`。 参数：`size`、`alignment`、`stream`。 关键调用包括 `torch.empty`。 该作用域涉及PyTorch 张量准备与校验。

##### Lines 759-759

```python
        return torch.empty(size, device="cuda", dtype=torch.int8)
```
- **EN:** Invokes `torch.empty` to execute the test logic. Relevant themes: PyTorch tensor setup and checks.
- **CN:** 调用 `torch.empty` 执行测试逻辑。 相关主题：PyTorch 张量准备与校验。

#### Lines 760-764

```python

    triton.set_allocator(alloc_fn)

    tma_device_store_kernel[(1, )](out, XBLOCK, smem_layout)
    torch.testing.assert_close(out, torch.zeros_like(out))
```
- **EN:** Invokes `triton.set_allocator`, `torch.testing.assert_close`, `torch.zeros_like` to execute the test logic. Relevant themes: PyTorch tensor setup and checks, layout transformation reasoning.
- **CN:** 调用 `triton.set_allocator`、`torch.testing.assert_close`、`torch.zeros_like` 执行测试逻辑。 相关主题：PyTorch 张量准备与校验、布局变换推理。

### Lines 765-771

```python


@gluon.jit
def mma_kernel(a, b, out, M: ttgl.constexpr, N: ttgl.constexpr, K: ttgl.constexpr, block_layout_a: ttgl.constexpr,
               block_layout_b: ttgl.constexpr, cga_layout_c: ttgl.constexpr, acc_layout: ttgl.constexpr,
               shared_layout_a: ttgl.constexpr, shared_layout_b: ttgl.constexpr, acc_dtype: ttgl.constexpr,
               ASYNC: ttgl.constexpr, USE_TCGEN05: ttgl.constexpr):
```
- **EN:** Defines the helper function `mma_kernel`. Decorators: `gluon.jit`. Parameters: `a`, `b`, `out`, `M`, `N`, `K`, `block_layout_a`, `block_layout_b`, and 7 more. Key calls include `ttgl.load`, `ttgl.allocate_shared_memory`, `ttgl.store`, `ttgl.arange`, `ttgl.set_auto_layout`, `fence_async_shared`, and 10 more. This scope touches layout transformation reasoning, random-data generation.
- **CN:** 定义辅助函数 `mma_kernel`。 装饰器：`gluon.jit`。 参数：`a`、`b`、`out`、`M`、`N`、`K`、`block_layout_a`、`block_layout_b` 等另外 7 项。 关键调用包括 `ttgl.load`、`ttgl.allocate_shared_memory`、`ttgl.store`、`ttgl.arange`、`ttgl.set_auto_layout`、`fence_async_shared` 等另外 10 项。 该作用域涉及布局变换推理、随机数据生成。

#### Lines 772-784

```python
    a_offs_m = ttgl.arange(0, M)[:, None]
    a_offs_k = ttgl.arange(0, K)[None, :]
    b_offs_k = ttgl.arange(0, K)[:, None]
    b_offs_n = ttgl.arange(0, N)[None, :]

    operand_dtype = a.dtype.element_ty
    a_ptrs = a + a_offs_m * K + a_offs_k
    b_ptrs = b + b_offs_k * N + b_offs_n
    a_tile = ttgl.load(ttgl.set_auto_layout(a_ptrs, block_layout_a))
    b_tile = ttgl.load(ttgl.set_auto_layout(b_ptrs, block_layout_b))

    smem_a = ttgl.allocate_shared_memory(operand_dtype, [M, K], shared_layout_a, a_tile)
    smem_b = ttgl.allocate_shared_memory(operand_dtype, [K, N], shared_layout_b, b_tile)
```
- **EN:** Prepares or updates state through `a_offs_m`, `a_offs_k`, `b_offs_k`, `b_offs_n`, `operand_dtype`, `a_ptrs`, `b_ptrs`, `a_tile`, and 3 more. Invokes `ttgl.arange`, `ttgl.load`, `ttgl.set_auto_layout`, `ttgl.allocate_shared_memory` to execute the test logic. Relevant themes: layout transformation reasoning, random-data generation.
- **CN:** 通过 `a_offs_m`、`a_offs_k`、`b_offs_k`、`b_offs_n`、`operand_dtype`、`a_ptrs`、`b_ptrs`、`a_tile` 等另外 3 项 准备或更新状态。 调用 `ttgl.arange`、`ttgl.load`、`ttgl.set_auto_layout`、`ttgl.allocate_shared_memory` 执行测试逻辑。 相关主题：布局变换推理、随机数据生成。

#### Lines 785-804

```python

    if USE_TCGEN05:
        two_ctas: ttgl.constexpr = acc_layout.two_ctas
        fence_async_shared(cluster=two_ctas)
        mma_barrier = mbarrier.allocate_mbarrier()
        mbarrier.init(mma_barrier, count=1)

        acc_tmem = allocate_tensor_memory(acc_dtype, [M, N], acc_layout)

        tcgen05_mma(smem_a, smem_b, acc_tmem, use_acc=False, mbarriers=[mma_barrier])
        mbarrier.wait(mma_barrier, phase=0, deps=[smem_a, smem_b])
        mbarrier.invalidate(mma_barrier)

        acc = acc_tmem.load()
    else:
        acc = ttgl.zeros([M, N], dtype=acc_dtype, layout=acc_layout)
        acc = hopper.warpgroup_mma(smem_a, smem_b, acc, is_async=ASYNC)

        if ASYNC:
            acc = hopper.warpgroup_mma_wait(num_outstanding=0, deps=[acc])
```
- **EN:** Invokes `fence_async_shared`, `mbarrier.allocate_mbarrier`, `mbarrier.init`, `allocate_tensor_memory`, `tcgen05_mma`, `mbarrier.wait`, and 5 more to execute the test logic. Branches on runtime or test conditions. Relevant themes: layout transformation reasoning.
- **CN:** 调用 `fence_async_shared`、`mbarrier.allocate_mbarrier`、`mbarrier.init`、`allocate_tensor_memory`、`tcgen05_mma`、`mbarrier.wait` 等另外 5 项 执行测试逻辑。 根据运行时或测试条件进行分支。 相关主题：布局变换推理。

#### Lines 805-809

```python

    out_offs_m = ttgl.arange(0, M)[:, None]
    out_offs_n = ttgl.arange(0, N)[None, :]
    out_ptrs = out + out_offs_m * N + out_offs_n
    ttgl.store(out_ptrs, acc)
```
- **EN:** Prepares or updates state through `out_offs_m`, `out_offs_n`, `out_ptrs`. Invokes `ttgl.arange`, `ttgl.store` to execute the test logic.
- **CN:** 通过 `out_offs_m`、`out_offs_n`、`out_ptrs` 准备或更新状态。 调用 `ttgl.arange`、`ttgl.store` 执行测试逻辑。

### Lines 810-814

```python


@pytest.mark.skipif(not is_hopper(), reason="Requires Hopper")
@pytest.mark.parametrize("ASYNC", [True, False])
def test_warpgroup_mma(ASYNC):
```
- **EN:** Defines the test function `test_warpgroup_mma`. Decorators: `pytest.mark.skipif(not is_hopper(), reason='Requires Hopper')`, `pytest.mark.parametrize('ASYNC', [True, False])`. Parameters: `ASYNC`. Key calls include `pytest.mark.skipif`, `pytest.mark.parametrize`, `torch.manual_seed`, `ttgl.BlockedLayout`, `ttgl.NVMMADistributedLayout`, `ttgl.NVMMASharedLayout.get_default_for`, and 5 more. This scope touches pytest parametrization, PyTorch tensor setup and checks, matrix multiplication workflows, layout transformation reasoning.
- **CN:** 定义测试函数 `test_warpgroup_mma`。 装饰器：`pytest.mark.skipif(not is_hopper(), reason='Requires Hopper')`、`pytest.mark.parametrize('ASYNC', [True, False])`。 参数：`ASYNC`。 关键调用包括 `pytest.mark.skipif`、`pytest.mark.parametrize`、`torch.manual_seed`、`ttgl.BlockedLayout`、`ttgl.NVMMADistributedLayout`、`ttgl.NVMMASharedLayout.get_default_for` 等另外 5 项。 该作用域涉及pytest 参数化、PyTorch 张量准备与校验、矩阵乘法工作流、布局变换推理。

#### Lines 815-846

```python
    torch.manual_seed(0)
    M, N, K = 64, 32, 32
    warps = [4, 1]
    block_layout = ttgl.BlockedLayout([1, 1], [1, THREADS_PER_WARP], warps_per_cta=warps, order=[1, 0])
    acc_layout = ttgl.NVMMADistributedLayout(version=[3, 0], warps_per_cta=warps, instr_shape=[16, 32, 16])
    shared_layout_a = ttgl.NVMMASharedLayout.get_default_for([M, K], ttgl.float16)
    shared_layout_b = ttgl.NVMMASharedLayout.get_default_for([K, N], ttgl.float16)
    a = torch.randn((M, K), device="cuda", dtype=torch.float16)
    b = torch.randn((K, N), device="cuda", dtype=torch.float16)
    out = torch.zeros((M, N), device="cuda", dtype=torch.float16)
    mma_kernel[(1, )](
        a,
        b,
        out,
        M,
        N,
        K,
        block_layout,
        block_layout,
        block_layout,
        acc_layout,
        shared_layout_a,
        shared_layout_b,
        ttgl.float16,
        ASYNC,
        False,
        num_warps=warps[0] * warps[1],
    )

    ref = torch.matmul(a, b)

    torch.testing.assert_close(out, ref, atol=1e-3, rtol=1e-1)
```
- **EN:** Prepares or updates state through `M`, `N`, `K`, `warps`, `block_layout`, `acc_layout`, `shared_layout_a`, `shared_layout_b`, and 4 more. Invokes `torch.manual_seed`, `ttgl.BlockedLayout`, `ttgl.NVMMADistributedLayout`, `ttgl.NVMMASharedLayout.get_default_for`, `torch.randn`, `torch.zeros`, and 2 more to execute the test logic. Relevant themes: PyTorch tensor setup and checks, matrix multiplication workflows, layout transformation reasoning, random-data generation.
- **CN:** 通过 `M`、`N`、`K`、`warps`、`block_layout`、`acc_layout`、`shared_layout_a`、`shared_layout_b` 等另外 4 项 准备或更新状态。 调用 `torch.manual_seed`、`ttgl.BlockedLayout`、`ttgl.NVMMADistributedLayout`、`ttgl.NVMMASharedLayout.get_default_for`、`torch.randn`、`torch.zeros` 等另外 2 项 执行测试逻辑。 相关主题：PyTorch 张量准备与校验、矩阵乘法工作流、布局变换推理、随机数据生成。

### Lines 847-855

```python


@gluon.jit
def tma_mma_shared_inputs_kernel(a_desc, b_desc, out_ptr, out_desc, gather_idx_ptr, scatter_idx_ptr,
                                 BLOCK_M: ttgl.constexpr, BLOCK_N: ttgl.constexpr, BLOCK_K: ttgl.constexpr,
                                 NUM_K_TILES: ttgl.constexpr, block_layout_c: ttgl.constexpr,
                                 acc_layout: ttgl.constexpr, acc_tmem_layout: ttgl.constexpr,
                                 use_tcgen05: ttgl.constexpr, multicast: ttgl.constexpr,
                                 use_gather_scatter: ttgl.constexpr):
```
- **EN:** Defines the helper function `tma_mma_shared_inputs_kernel`. Decorators: `gluon.jit`. Parameters: `a_desc`, `b_desc`, `out_ptr`, `out_desc`, `gather_idx_ptr`, `scatter_idx_ptr`, `BLOCK_M`, `BLOCK_N`, and 8 more. Key calls include `ttgl.allocate_shared_memory`, `mbarrier.allocate_mbarrier`, `mbarrier.init`, `mbarrier.invalidate`, `ttgl.convert_layout`, `isinstance`, and 20 more. This scope touches layout transformation reasoning.
- **CN:** 定义辅助函数 `tma_mma_shared_inputs_kernel`。 装饰器：`gluon.jit`。 参数：`a_desc`、`b_desc`、`out_ptr`、`out_desc`、`gather_idx_ptr`、`scatter_idx_ptr`、`BLOCK_M`、`BLOCK_N` 等另外 8 项。 关键调用包括 `ttgl.allocate_shared_memory`、`mbarrier.allocate_mbarrier`、`mbarrier.init`、`mbarrier.invalidate`、`ttgl.convert_layout`、`isinstance` 等另外 20 项。 该作用域涉及布局变换推理。

#### Lines 856-863

```python
    smem_a = ttgl.allocate_shared_memory(a_desc.dtype, [BLOCK_M, BLOCK_K], a_desc.layout)
    smem_b = ttgl.allocate_shared_memory(b_desc.dtype, b_desc.block_shape, b_desc.layout)

    two_ctas: ttgl.constexpr = isinstance(acc_tmem_layout, TensorMemoryLayout) and acc_tmem_layout.two_ctas

    tma_bar = mbarrier.allocate_mbarrier(two_ctas=two_ctas)
    mbarrier.init(tma_bar, count=1)
    phase_tma = 0
```
- **EN:** Prepares or updates state through `smem_a`, `smem_b`, `two_ctas`, `tma_bar`, `phase_tma`. Invokes `ttgl.allocate_shared_memory`, `isinstance`, `mbarrier.allocate_mbarrier`, `mbarrier.init` to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `smem_a`、`smem_b`、`two_ctas`、`tma_bar`、`phase_tma` 准备或更新状态。 调用 `ttgl.allocate_shared_memory`、`isinstance`、`mbarrier.allocate_mbarrier`、`mbarrier.init` 执行测试逻辑。 相关主题：布局变换推理。

#### Lines 864-876

```python

    if use_tcgen05:
        acc_tmem = allocate_tensor_memory(
            element_ty=ttgl.float32,
            shape=[BLOCK_M, BLOCK_N],
            layout=acc_tmem_layout,
        )
        mma_bar = mbarrier.allocate_mbarrier()
        phase_mma = 0
        mbarrier.init(mma_bar, count=tcgen05_mma_barrier_count([smem_a, smem_b], multicast,
                                                               acc_tmem.type.layout.two_ctas))
    else:
        acc = ttgl.zeros([BLOCK_M, BLOCK_N], dtype=ttgl.float32, layout=acc_layout)
```
- **EN:** Invokes `allocate_tensor_memory`, `mbarrier.allocate_mbarrier`, `mbarrier.init`, `ttgl.zeros`, `tcgen05_mma_barrier_count` to execute the test logic. Branches on runtime or test conditions. Relevant themes: layout transformation reasoning.
- **CN:** 调用 `allocate_tensor_memory`、`mbarrier.allocate_mbarrier`、`mbarrier.init`、`ttgl.zeros`、`tcgen05_mma_barrier_count` 执行测试逻辑。 根据运行时或测试条件进行分支。 相关主题：布局变换推理。

#### Lines 877-881

```python

    if use_gather_scatter:
        gather_offsets_layout: ttgl.constexpr = ttgl.SliceLayout(
            1, ttgl.BlockedLayout([4, 1], [1, 32], [ttgl.num_warps(), 1], [0, 1], cga_layout=a_desc.layout.cga_layout))
        gather_offsets = ttgl.load(gather_idx_ptr + ttgl.arange(0, BLOCK_M, layout=gather_offsets_layout))
```
- **EN:** Invokes `ttgl.SliceLayout`, `ttgl.load`, `ttgl.BlockedLayout`, `ttgl.arange`, `ttgl.num_warps` to execute the test logic. Branches on runtime or test conditions. Relevant themes: layout transformation reasoning.
- **CN:** 调用 `ttgl.SliceLayout`、`ttgl.load`、`ttgl.BlockedLayout`、`ttgl.arange`、`ttgl.num_warps` 执行测试逻辑。 根据运行时或测试条件进行分支。 相关主题：布局变换推理。

#### Lines 882-902

```python

    for k in range(NUM_K_TILES):
        mbarrier.expect(tma_bar, smem_a.nbytes_per_cta + smem_b.nbytes_per_cta)
        if use_gather_scatter:
            blackwell_tma.async_gather(a_desc, gather_offsets, k * BLOCK_K, tma_bar, smem_a, multicast=multicast)
        else:
            tma.async_load(a_desc, [0, k * BLOCK_K], tma_bar, smem_a, multicast=multicast)
        tma.async_load(b_desc, [k * BLOCK_K, 0], tma_bar, smem_b, multicast=multicast)
        mbarrier.wait(tma_bar, phase=phase_tma, deps=[smem_a, smem_b])
        phase_tma ^= 1

        if use_tcgen05:
            tcgen05_mma(smem_a, smem_b, acc_tmem, use_acc=(k != 0), multicast=multicast, mbarriers=[mma_bar])
            mbarrier.wait(mma_bar, phase=phase_mma, deps=[smem_a, smem_b])
            phase_mma ^= 1
        else:
            acc = hopper.warpgroup_mma(smem_a, smem_b, acc, is_async=False)
            if multicast:
                # multicast into wgmma doesn't make much sense as you need to synchronise all
                # CTAs after the wgmma, as it doesn't provide a finer synchronization mechanism.
                ttgl.barrier(cluster=True)
```
- **EN:** Invokes `mbarrier.expect`, `tma.async_load`, `mbarrier.wait`, `blackwell_tma.async_gather`, `tcgen05_mma`, `hopper.warpgroup_mma`, and 1 more to execute the test logic. Iterates across cases or data tiles.
- **CN:** 调用 `mbarrier.expect`、`tma.async_load`、`mbarrier.wait`、`blackwell_tma.async_gather`、`tcgen05_mma`、`hopper.warpgroup_mma` 等另外 1 项 执行测试逻辑。 通过循环覆盖多个用例或数据分块。

#### Lines 903-904

```python

    mbarrier.invalidate(tma_bar)
```
- **EN:** Invokes `mbarrier.invalidate` to execute the test logic.
- **CN:** 调用 `mbarrier.invalidate` 执行测试逻辑。

#### Lines 905-908

```python

    if use_tcgen05:
        mbarrier.invalidate(mma_bar)
        acc = acc_tmem.load()
```
- **EN:** Invokes `mbarrier.invalidate`, `acc_tmem.load` to execute the test logic. Branches on runtime or test conditions.
- **CN:** 调用 `mbarrier.invalidate`、`acc_tmem.load` 执行测试逻辑。 根据运行时或测试条件进行分支。

#### Lines 909-910

```python

    acc = ttgl.convert_layout(acc, block_layout_c)
```
- **EN:** Prepares or updates state through `acc`. Invokes `ttgl.convert_layout` to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `acc` 准备或更新状态。 调用 `ttgl.convert_layout` 执行测试逻辑。 相关主题：布局变换推理。

#### Lines 911-923

```python
    if use_gather_scatter:
        scatter_offsets_layout: ttgl.constexpr = ttgl.SliceLayout(
            1, ttgl.BlockedLayout([4, 1], [1, 32], [ttgl.num_warps(), 1], [0, 1],
                                  cga_layout=out_desc.layout.cga_layout))
        scatter_offsets = ttgl.load(scatter_idx_ptr + ttgl.arange(0, BLOCK_M, layout=scatter_offsets_layout))
        acc_smem = ttgl.allocate_shared_memory(out_desc.dtype, [BLOCK_M, BLOCK_N], out_desc.layout, acc)
        blackwell_tma.async_scatter(out_desc, scatter_offsets, 0, acc_smem)
        tma.store_wait(0)
        acc_smem._keep_alive()
    else:
        offs_m = ttgl.arange(0, BLOCK_M)[:, None]
        offs_n = ttgl.arange(0, BLOCK_N)[None, :]
        ttgl.store(out_ptr + offs_m * BLOCK_N + offs_n, acc)
```
- **EN:** Invokes `ttgl.SliceLayout`, `ttgl.load`, `ttgl.allocate_shared_memory`, `blackwell_tma.async_scatter`, `tma.store_wait`, `acc_smem._keep_alive`, and 4 more to execute the test logic. Branches on runtime or test conditions. Relevant themes: layout transformation reasoning.
- **CN:** 调用 `ttgl.SliceLayout`、`ttgl.load`、`ttgl.allocate_shared_memory`、`blackwell_tma.async_scatter`、`tma.store_wait`、`acc_smem._keep_alive` 等另外 4 项 执行测试逻辑。 根据运行时或测试条件进行分支。 相关主题：布局变换推理。

### Lines 924-933

```python


@pytest.mark.skipif(not (is_hopper() or is_blackwell()), reason="Requires Hopper or Blackwell")
@pytest.mark.parametrize("warps", ([8, 1], [4, 2], [4, 1]))
@pytest.mark.parametrize("reps", ([1, 1, 1], [2, 2, 2], [1, 4, 2]))
@pytest.mark.parametrize("ctas_per_cga", [[1, 1], [2, 1], [4, 4]])
@pytest.mark.parametrize("two_ctas", [False, True] if is_blackwell() else [False])
@pytest.mark.parametrize("multicast", [False, True])
@pytest.mark.parametrize("use_gather_scatter", [False, True] if is_blackwell() else [False])
def test_tma_mma_shared_inputs(warps, reps, ctas_per_cga, two_ctas, multicast, use_gather_scatter):
```
- **EN:** Defines the test function `test_tma_mma_shared_inputs`. Decorators: `pytest.mark.skipif(not (is_hopper() or is_blackwell()), reason='Requires Hopper or Blackwell')`, `pytest.mark.parametrize('warps', ([8, 1], [4, 2], [4, 1]))`, `pytest.mark.parametrize('reps', ([1, 1, 1], [2, 2, 2], [1, 4, 2]))`, `pytest.mark.parametrize('ctas_per_cga', [[1, 1], [2, 1], [4, 4]])`, `pytest.mark.parametrize('two_ctas', [False, True] if is_blackwell() else [False])`, `pytest.mark.parametrize('multicast', [False, True])`, and 1 more. Parameters: `warps`, `reps`, `ctas_per_cga`, `two_ctas`, `multicast`, `use_gather_scatter`. Nested definitions in this scope: `cast`. Key calls include `pytest.mark.skipif`, `pytest.mark.parametrize`, `ttgl.BlockedLayout`, `ttgl.NVMMADistributedLayout`, `TensorMemoryLayout`, `triton.runtime.driver.active.get_current_device`, and 21 more. This scope touches pytest parametrization, PyTorch tensor setup and checks, tensor/descriptor metadata, matrix multiplication workflows.
- **CN:** 定义测试函数 `test_tma_mma_shared_inputs`。 装饰器：`pytest.mark.skipif(not (is_hopper() or is_blackwell()), reason='Requires Hopper or Blackwell')`、`pytest.mark.parametrize('warps', ([8, 1], [4, 2], [4, 1]))`、`pytest.mark.parametrize('reps', ([1, 1, 1], [2, 2, 2], [1, 4, 2]))`、`pytest.mark.parametrize('ctas_per_cga', [[1, 1], [2, 1], [4, 4]])`、`pytest.mark.parametrize('two_ctas', [False, True] if is_blackwell() else [False])`、`pytest.mark.parametrize('multicast', [False, True])` 等另外 1 项。 参数：`warps`、`reps`、`ctas_per_cga`、`two_ctas`、`multicast`、`use_gather_scatter`。 该作用域中的嵌套定义：`cast`。 关键调用包括 `pytest.mark.skipif`、`pytest.mark.parametrize`、`ttgl.BlockedLayout`、`ttgl.NVMMADistributedLayout`、`TensorMemoryLayout`、`triton.runtime.driver.active.get_current_device` 等另外 21 项。 该作用域涉及pytest 参数化、PyTorch 张量准备与校验、张量/描述符元数据、矩阵乘法工作流。

#### Lines 934-935

```python
    bitwidth = 16
    acc_dtype = torch.float32
```
- **EN:** Prepares or updates state through `bitwidth`, `acc_dtype`. Relevant themes: PyTorch tensor setup and checks.
- **CN:** 通过 `bitwidth`、`acc_dtype` 准备或更新状态。 相关主题：PyTorch 张量准备与校验。

#### Lines 936-938

```python

    if ctas_per_cga[0] == 1 and two_ctas:
        pytest.skip("Need at least 2 CTAs along M for 2CTA mode")
```
- **EN:** Invokes `pytest.skip` to execute the test logic. Branches on runtime or test conditions.
- **CN:** 调用 `pytest.skip` 执行测试逻辑。 根据运行时或测试条件进行分支。

#### Lines 939-940

```python

    cta_order = [1, 0]
```
- **EN:** Prepares or updates state through `cta_order`.
- **CN:** 通过 `cta_order` 准备或更新状态。

#### Lines 941-946

```python

    if two_ctas:
        assert ctas_per_cga[0] >= 2, "Need at least 2 CTAs along M for 2CTA mode"
        ctas_per_cga_b = [ctas_per_cga[0] // 2, 2 * ctas_per_cga[1]]
    else:
        ctas_per_cga_b = ctas_per_cga
```
- **EN:** Validates behavior with 1 assertion(s). Branches on runtime or test conditions.
- **CN:** 通过 1 个断言验证行为。 根据运行时或测试条件进行分支。

#### Lines 947-954

```python
    cta_split_a = [ctas_per_cga[0], 1]
    cta_split_b = [1, ctas_per_cga_b[1]]

    # M = 128 for blackkwell
    instr_shape = [32 if is_blackwell() else 16, 32, 256 // bitwidth]
    NUM_K_TILES = 4
    BLOCK_M = instr_shape[0] * warps[0] * ctas_per_cga[0] * reps[0]
    BLOCK_N = instr_shape[1] * warps[1] * ctas_per_cga_b[1] * reps[1]
```
- **EN:** Prepares or updates state through `cta_split_a`, `cta_split_b`, `instr_shape`, `NUM_K_TILES`, `BLOCK_M`, `BLOCK_N`. Invokes `is_blackwell` to execute the test logic.
- **CN:** 通过 `cta_split_a`、`cta_split_b`、`instr_shape`、`NUM_K_TILES`、`BLOCK_M`、`BLOCK_N` 准备或更新状态。 调用 `is_blackwell` 执行测试逻辑。

#### Lines 955-957

```python
    if is_blackwell() and BLOCK_N >= 256 * ctas_per_cga[1]:
        # tcgen05 doesn't support reps along N
        BLOCK_N = 256 * ctas_per_cga[1]
```
- **EN:** Invokes `is_blackwell` to execute the test logic. Branches on runtime or test conditions.
- **CN:** 调用 `is_blackwell` 执行测试逻辑。 根据运行时或测试条件进行分支。

#### Lines 958-959

```python
    BLOCK_K = instr_shape[2] * reps[2]
    K = (256 // bitwidth) * NUM_K_TILES
```
- **EN:** Prepares or updates state through `BLOCK_K`, `K`.
- **CN:** 通过 `BLOCK_K`、`K` 准备或更新状态。

#### Lines 960-968

```python

    if two_ctas:
        cga_layout_a = make_2cta_cga_layout(ctas_per_cga, cta_split_a, cta_order, 0)
        cga_layout_b = make_2cta_cga_layout(ctas_per_cga_b, cta_split_b, cta_order, 1)
        cga_layout_c = make_2cta_cga_layout(ctas_per_cga, ctas_per_cga, cta_order, 0)
    else:
        cga_layout_a = make_cga_layout(ctas_per_cga, cta_split_a, cta_order)
        cga_layout_b = make_cga_layout(ctas_per_cga_b, cta_split_b, cta_order)
        cga_layout_c = make_cga_layout(ctas_per_cga, ctas_per_cga, cta_order)
```
- **EN:** Invokes `make_2cta_cga_layout`, `make_cga_layout` to execute the test logic. Branches on runtime or test conditions. Relevant themes: layout transformation reasoning.
- **CN:** 调用 `make_2cta_cga_layout`、`make_cga_layout` 执行测试逻辑。 根据运行时或测试条件进行分支。 相关主题：布局变换推理。

#### Lines 969-982

```python

    block_layout_c = ttgl.BlockedLayout([1, 8], [1, THREADS_PER_WARP], warps_per_cta=warps, order=[1, 0],
                                        cga_layout=cga_layout_c)

    acc_layout = ttgl.NVMMADistributedLayout(version=[3, 0], warps_per_cta=warps, instr_shape=instr_shape,
                                             cga_layout=cga_layout_c)

    tmem_shape = (min(BLOCK_M // ctas_per_cga[0], 128), BLOCK_N // ctas_per_cga[1])
    acc_tmem_layout = TensorMemoryLayout(
        block=tmem_shape,
        col_stride=1,
        cga_layout=cga_layout_c,
        two_ctas=two_ctas,
    )
```
- **EN:** Prepares or updates state through `block_layout_c`, `acc_layout`, `tmem_shape`, `acc_tmem_layout`. Invokes `ttgl.BlockedLayout`, `ttgl.NVMMADistributedLayout`, `min`, `TensorMemoryLayout` to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `block_layout_c`、`acc_layout`、`tmem_shape`、`acc_tmem_layout` 准备或更新状态。 调用 `ttgl.BlockedLayout`、`ttgl.NVMMADistributedLayout`、`min`、`TensorMemoryLayout` 执行测试逻辑。 相关主题：布局变换推理。

#### Lines 983-984

```python

    def cast(x, dtype):
```
- **EN:** Defines the helper function `cast`. Parameters: `x`, `dtype`. Key calls include `x.view`, `x.to`. This scope touches PyTorch tensor setup and checks.
- **CN:** 定义辅助函数 `cast`。 参数：`x`、`dtype`。 关键调用包括 `x.view`、`x.to`。 该作用域涉及PyTorch 张量准备与校验。

##### Lines 985-986

```python
        if dtype != torch.float32:
            return x.to(dtype)
```
- **EN:** Invokes `x.to` to execute the test logic. Branches on runtime or test conditions. Relevant themes: PyTorch tensor setup and checks.
- **CN:** 调用 `x.to` 执行测试逻辑。 根据运行时或测试条件进行分支。 相关主题：PyTorch 张量准备与校验。

##### Lines 987-993

```python
        # For b16 and fp32 (in both hopper and blackwell it seems)
        # Element-wise multiplication of matrix A and B is performed with specified precision.
        # wgmma.mma_async operation involving type .tf32 will truncate lower 13 bits of the 32-bit
        # input data before multiplication is issued
        x = x.view(torch.int32)
        x = x & ~((1 << 13) - 1)
        return x.view(dtype)
```
- **EN:** Prepares or updates state through `x`. Invokes `x.view` to execute the test logic. Relevant themes: PyTorch tensor setup and checks.
- **CN:** 通过 `x` 准备或更新状态。 调用 `x.view` 执行测试逻辑。 相关主题：PyTorch 张量准备与校验。

#### Lines 994-1005

```python

    torch_dtype = torch.float16
    device = triton.runtime.driver.active.get_current_device()
    a = cast(torch.randn((BLOCK_M, K), device=device, dtype=torch.float32), torch_dtype)
    # We transpose b in the kernel
    b = cast(torch.randn((K, BLOCK_N), device=device, dtype=torch.float32), torch_dtype)
    out = torch.empty((BLOCK_M, BLOCK_N), device=device, dtype=acc_dtype)

    gluon_dtype = ttgl.float16
    shared_layout_a = ttgl.NVMMASharedLayout.get_default_for([BLOCK_M, BLOCK_K], gluon_dtype, cga_layout=cga_layout_a)
    shared_layout_b = ttgl.NVMMASharedLayout.get_default_for([BLOCK_K, BLOCK_N], gluon_dtype, cga_layout=cga_layout_b)
    shared_layout_c = ttgl.NVMMASharedLayout.get_default_for([BLOCK_M, BLOCK_N], ttgl.float32, cga_layout=cga_layout_c)
```
- **EN:** Prepares or updates state through `torch_dtype`, `device`, `a`, `b`, `out`, `gluon_dtype`, `shared_layout_a`, `shared_layout_b`, and 1 more. Invokes `triton.runtime.driver.active.get_current_device`, `cast`, `torch.randn`, `torch.empty`, `ttgl.NVMMASharedLayout.get_default_for` to execute the test logic. Relevant themes: PyTorch tensor setup and checks, runtime driver interaction, layout transformation reasoning, random-data generation.
- **CN:** 通过 `torch_dtype`、`device`、`a`、`b`、`out`、`gluon_dtype`、`shared_layout_a`、`shared_layout_b` 等另外 1 项 准备或更新状态。 调用 `triton.runtime.driver.active.get_current_device`、`cast`、`torch.randn`、`torch.empty`、`ttgl.NVMMASharedLayout.get_default_for` 执行测试逻辑。 相关主题：PyTorch 张量准备与校验、运行时驱动交互、布局变换推理、随机数据生成。

#### Lines 1006-1007

```python
    assert shared_layout_a.swizzle_byte_width != 0
    assert shared_layout_b.swizzle_byte_width != 0
```
- **EN:** Validates behavior with 2 assertion(s). Relevant themes: layout transformation reasoning.
- **CN:** 通过 2 个断言验证行为。 相关主题：布局变换推理。

#### Lines 1008-1015

```python
    a_desc = gluon.nvidia.hopper.TensorDescriptor.from_tensor(a, [1 if use_gather_scatter else BLOCK_M, BLOCK_K],
                                                              shared_layout_a)
    b_desc = gluon.nvidia.hopper.TensorDescriptor.from_tensor(b, [BLOCK_K, BLOCK_N], shared_layout_b)
    num_warps = warps[0] * warps[1]
    num_ctas = ctas_per_cga[0] * ctas_per_cga[1]
    out_desc = gluon.nvidia.hopper.TensorDescriptor.from_tensor(out, [1, BLOCK_N], shared_layout_c)
    gather_idx = torch.arange(BLOCK_M - 1, -1, -1, dtype=torch.int32, device=device)
    scatter_idx = (torch.arange(0, BLOCK_M, dtype=torch.int32, device=device) + 1) % BLOCK_M
```
- **EN:** Prepares or updates state through `a_desc`, `b_desc`, `num_warps`, `num_ctas`, `out_desc`, `gather_idx`, `scatter_idx`. Invokes `gluon.nvidia.hopper.TensorDescriptor.from_tensor`, `torch.arange` to execute the test logic. Relevant themes: PyTorch tensor setup and checks, tensor/descriptor metadata, layout transformation reasoning.
- **CN:** 通过 `a_desc`、`b_desc`、`num_warps`、`num_ctas`、`out_desc`、`gather_idx`、`scatter_idx` 准备或更新状态。 调用 `gluon.nvidia.hopper.TensorDescriptor.from_tensor`、`torch.arange` 执行测试逻辑。 相关主题：PyTorch 张量准备与校验、张量/描述符元数据、布局变换推理。

#### Lines 1016-1039

```python

    try:
        tma_mma_shared_inputs_kernel[(1, )](
            a_desc,
            b_desc,
            out,
            out_desc,
            gather_idx,
            scatter_idx,
            BLOCK_M,
            BLOCK_N,
            BLOCK_K,
            NUM_K_TILES,
            block_layout_c,
            acc_layout,
            acc_tmem_layout,
            is_blackwell(),
            multicast=multicast,
            use_gather_scatter=use_gather_scatter,
            num_warps=num_warps,
            num_ctas=num_ctas,
        )
    except triton.runtime.errors.OutOfResources:
        pytest.skip("Too much shared memory required")
```
- **EN:** Invokes `is_blackwell`, `pytest.skip` to execute the test logic. Wraps operations in exception-handling logic. Relevant themes: layout transformation reasoning.
- **CN:** 调用 `is_blackwell`、`pytest.skip` 执行测试逻辑。 使用异常处理逻辑包裹相关操作。 相关主题：布局变换推理。

#### Lines 1040-1052

```python

    try:
        allow_tf32 = torch.backends.cuda.matmul.allow_tf32
        torch.backends.cuda.matmul.allow_tf32 = True
        if use_gather_scatter:
            matmul = torch.matmul(a[gather_idx.to(torch.int64)].to(torch.float32), b.to(torch.float32))
            ref = torch.empty_like(matmul)
            # Correct as scatter_idx is a permutation!
            ref[scatter_idx.to(torch.int64)] = matmul
        else:
            ref = torch.matmul(a.to(torch.float32), b.to(torch.float32))
    finally:
        torch.backends.cuda.matmul.allow_tf32 = allow_tf32
```
- **EN:** Invokes `torch.matmul`, `torch.empty_like`, `b.to`, `scatter_idx.to`, `a.to`, `gather_idx.to` to execute the test logic. Wraps operations in exception-handling logic. Relevant themes: PyTorch tensor setup and checks, matrix multiplication workflows.
- **CN:** 调用 `torch.matmul`、`torch.empty_like`、`b.to`、`scatter_idx.to`、`a.to`、`gather_idx.to` 执行测试逻辑。 使用异常处理逻辑包裹相关操作。 相关主题：PyTorch 张量准备与校验、矩阵乘法工作流。

#### Lines 1053-1059

```python

    if bitwidth == 8:
        atol, rtol = 8e-2, 8e-1
    elif bitwidth == 16:
        atol, rtol = 5e-2, 5e-1
    else:
        atol, rtol = 8e-4, 8e-3
```
- **EN:** Branches on runtime or test conditions.
- **CN:** 根据运行时或测试条件进行分支。

#### Lines 1060-1060

```python
    torch.testing.assert_close(out, ref, atol=atol, rtol=rtol)
```
- **EN:** Invokes `torch.testing.assert_close` to execute the test logic. Relevant themes: PyTorch tensor setup and checks.
- **CN:** 调用 `torch.testing.assert_close` 执行测试逻辑。 相关主题：PyTorch 张量准备与校验。

### Lines 1061-1079

```python


@pytest.mark.skipif(not (is_hopper() or is_blackwell()), reason="Requires Hopper or Blackwell")
@pytest.mark.parametrize("bitwidth, transpose_a, transpose_b, acc_dtype",
                         [(bitwidth, transpose_a, transpose_b, acc_dtype)
                          for bitwidth in [8, 16, 32]
                          for (transpose_a, transpose_b) in product([False, True], repeat=2)
                          for acc_dtype in [torch.float16, torch.float32]
                          if bitwidth == 16 or (acc_dtype == torch.float32 and not transpose_a and transpose_b)])
@pytest.mark.parametrize("warps", ([8, 1], [4, 2], [4, 1]))
@pytest.mark.parametrize("swizzling_a, swizzling_b", product([0, 32, 64, 128], repeat=2))
@pytest.mark.parametrize("instr_m", [64, 128] if is_blackwell() else [64])
@pytest.mark.parametrize("shape_m, shape_n, shape_k", [(1, 1, 1), (2, 4, 1), (2, 2, 4)])
@pytest.mark.parametrize("ctas_per_cga", [[1, 1], [2, 1], [4, 4]])
@pytest.mark.parametrize("two_ctas", [False, True] if is_blackwell() else [False])
def test_mma_shared_inputs(bitwidth, transpose_a, transpose_b, acc_dtype, warps, swizzling_a, swizzling_b, instr_m,
                           shape_m, shape_n, shape_k, ctas_per_cga, two_ctas):
    # FIXME: Workaround for a bug in PTXAS when the shared layout is transposed and the swizzling is 0
    # This is fixed in PTXAS 13.0.88. Remove once we upgrade
```
- **EN:** Defines the test function `test_mma_shared_inputs`. Decorators: `pytest.mark.skipif(not (is_hopper() or is_blackwell()), reason='Requires Hopper or Blackwell')`, `pytest.mark.parametrize('bitwidth, transpose_a, transpose_b, acc_dtype', [(bitwidth, transpose_a, transpose_b, acc_dtype) for bitwidth in [8, 16, 32] for transpose_a, transpose_b in product([False, True], repeat=2) for acc_dtype in [torch.float16, torch.float32] if bitwidth == 16 or (acc_dtype == torch.float32 and (not transpose_a) and transpose_b)])`, `pytest.mark.parametrize('warps', ([8, 1], [4, 2], [4, 1]))`, `pytest.mark.parametrize('swizzling_a, swizzling_b', product([0, 32, 64, 128], repeat=2))`, `pytest.mark.parametrize('instr_m', [64, 128] if is_blackwell() else [64])`, `pytest.mark.parametrize('shape_m, shape_n, shape_k', [(1, 1, 1), (2, 4, 1), (2, 2, 4)])`, and 2 more. Parameters: `bitwidth`, `transpose_a`, `transpose_b`, `acc_dtype`, `warps`, `swizzling_a`, `swizzling_b`, `instr_m`, and 5 more. Nested definitions in this scope: `min_shape`, `log2_int`, `get_shared_swizzling_zero`, `cast`. Key calls include `pytest.mark.skipif`, `pytest.mark.parametrize`, `is_blackwell`, `min_shape`, `triton.runtime.driver.active.get_current_device`, `max_shared_mem`, and 29 more. This scope touches pytest parametrization, PyTorch tensor setup and checks, matrix multiplication workflows, runtime driver interaction.
- **CN:** 定义测试函数 `test_mma_shared_inputs`。 装饰器：`pytest.mark.skipif(not (is_hopper() or is_blackwell()), reason='Requires Hopper or Blackwell')`、`pytest.mark.parametrize('bitwidth, transpose_a, transpose_b, acc_dtype', [(bitwidth, transpose_a, transpose_b, acc_dtype) for bitwidth in [8, 16, 32] for transpose_a, transpose_b in product([False, True], repeat=2) for acc_dtype in [torch.float16, torch.float32] if bitwidth == 16 or (acc_dtype == torch.float32 and (not transpose_a) and transpose_b)])`、`pytest.mark.parametrize('warps', ([8, 1], [4, 2], [4, 1]))`、`pytest.mark.parametrize('swizzling_a, swizzling_b', product([0, 32, 64, 128], repeat=2))`、`pytest.mark.parametrize('instr_m', [64, 128] if is_blackwell() else [64])`、`pytest.mark.parametrize('shape_m, shape_n, shape_k', [(1, 1, 1), (2, 4, 1), (2, 2, 4)])` 等另外 2 项。 参数：`bitwidth`、`transpose_a`、`transpose_b`、`acc_dtype`、`warps`、`swizzling_a`、`swizzling_b`、`instr_m` 等另外 5 项。 该作用域中的嵌套定义：`min_shape`、`log2_int`、`get_shared_swizzling_zero`、`cast`。 关键调用包括 `pytest.mark.skipif`、`pytest.mark.parametrize`、`is_blackwell`、`min_shape`、`triton.runtime.driver.active.get_current_device`、`max_shared_mem` 等另外 29 项。 该作用域涉及pytest 参数化、PyTorch 张量准备与校验、矩阵乘法工作流、运行时驱动交互。

#### Lines 1080-1082

```python
    if bitwidth == 16 and ((transpose_a and swizzling_a == 0 and shape_m > 1) or
                           (not transpose_b and swizzling_b == 0 and shape_n > 1)):
        pytest.skip("Skipped due to a bug in PTXAS when the shared layout is transposed and the swizzling is 0")
```
- **EN:** Invokes `pytest.skip` to execute the test logic. Branches on runtime or test conditions. Relevant themes: layout transformation reasoning.
- **CN:** 调用 `pytest.skip` 执行测试逻辑。 根据运行时或测试条件进行分支。 相关主题：布局变换推理。

#### Lines 1083-1084

```python
    if ctas_per_cga[0] == 1 and two_ctas:
        pytest.skip("Need at least 2 CTAs for 2CTA mode")
```
- **EN:** Invokes `pytest.skip` to execute the test logic. Branches on runtime or test conditions.
- **CN:** 调用 `pytest.skip` 执行测试逻辑。 根据运行时或测试条件进行分支。

#### Lines 1085-1103

```python

    use_tcgen05 = is_blackwell()

    torch_dtype_map = {
        8: torch.float8_e4m3fn,
        16: torch.float16,
        32: torch.float32,
    }
    acc_dtype_map = {
        torch.float16: ttgl.float16,
        torch.float32: ttgl.float32,
    }

    # We'll choose a larger instr shape along N, but sure
    # instr_m is the instruction per warp group so we divide by 4
    instr_shape = [instr_m // 4, 32, 256 // bitwidth]
    M = instr_shape[0] * warps[0]
    N = instr_shape[1] * warps[1]
    K = instr_shape[2]
```
- **EN:** Prepares or updates state through `use_tcgen05`, `torch_dtype_map`, `acc_dtype_map`, `instr_shape`, `M`, `N`, `K`. Invokes `is_blackwell` to execute the test logic. Relevant themes: PyTorch tensor setup and checks.
- **CN:** 通过 `use_tcgen05`、`torch_dtype_map`、`acc_dtype_map`、`instr_shape`、`M`、`N`、`K` 准备或更新状态。 调用 `is_blackwell` 执行测试逻辑。 相关主题：PyTorch 张量准备与校验。

#### Lines 1104-1109

```python

    if two_ctas:
        assert ctas_per_cga[0] >= 2, "Need at least 2 CTAs along M for 2CTA mode"
        ctas_per_cga_b = [ctas_per_cga[0] // 2, 2 * ctas_per_cga[1]]
    else:
        ctas_per_cga_b = ctas_per_cga
```
- **EN:** Validates behavior with 1 assertion(s). Branches on runtime or test conditions.
- **CN:** 通过 1 个断言验证行为。 根据运行时或测试条件进行分支。

#### Lines 1110-1111

```python
    cta_split_a = [ctas_per_cga[0], 1]
    cta_split_b = [1, ctas_per_cga_b[1]]
```
- **EN:** Prepares or updates state through `cta_split_a`, `cta_split_b`.
- **CN:** 通过 `cta_split_a`、`cta_split_b` 准备或更新状态。

#### Lines 1112-1113

```python

    def min_shape(swizzling, dim0, dim1, trans):
```
- **EN:** Defines the helper function `min_shape`. Parameters: `swizzling`, `dim0`, `dim1`, `trans`. Key calls include `max`.
- **CN:** 定义辅助函数 `min_shape`。 参数：`swizzling`、`dim0`、`dim1`、`trans`。 关键调用包括 `max`。

##### Lines 1114-1115

```python
        tile_cols = (8 * max(16, swizzling)) // bitwidth
        outer_dim, contig_dim = (dim0, dim1)
```
- **EN:** Prepares or updates state through `tile_cols`, `outer_dim`, `contig_dim`. Invokes `max` to execute the test logic.
- **CN:** 通过 `tile_cols`、`outer_dim`、`contig_dim` 准备或更新状态。 调用 `max` 执行测试逻辑。

##### Lines 1116-1117

```python
        if trans:
            outer_dim, contig_dim = contig_dim, outer_dim
```
- **EN:** Branches on runtime or test conditions.
- **CN:** 根据运行时或测试条件进行分支。

##### Lines 1118-1119

```python
        contig_dim = max(contig_dim, tile_cols)
        outer_dim = max(outer_dim, 8)
```
- **EN:** Prepares or updates state through `contig_dim`, `outer_dim`. Invokes `max` to execute the test logic.
- **CN:** 通过 `contig_dim`、`outer_dim` 准备或更新状态。 调用 `max` 执行测试逻辑。

##### Lines 1120-1121

```python
        if trans:
            outer_dim, contig_dim = contig_dim, outer_dim
```
- **EN:** Branches on runtime or test conditions.
- **CN:** 根据运行时或测试条件进行分支。

##### Lines 1122-1122

```python
        return outer_dim, contig_dim
```
- **EN:** Carries supporting logic for the surrounding test or helper scope.
- **CN:** 为周围的测试或辅助作用域提供支撑逻辑。

#### Lines 1123-1133

```python

    # Get the minimum shape for the given swizzling / transpose
    M, K = min_shape(swizzling_a, M, K, transpose_a)
    K, N = min_shape(swizzling_b, K, N, transpose_b)
    M *= shape_m * ctas_per_cga[0]
    N *= shape_n * ctas_per_cga_b[1]
    K *= shape_k
    instr_shape[1] *= shape_n

    num_warps = warps[0] * warps[1]
    num_ctas = ctas_per_cga[0] * ctas_per_cga[1]
```
- **EN:** Prepares or updates state through `M`, `K`, `N`, `instr_shape`, `num_warps`, `num_ctas`. Invokes `min_shape` to execute the test logic.
- **CN:** 通过 `M`、`K`、`N`、`instr_shape`、`num_warps`、`num_ctas` 准备或更新状态。 调用 `min_shape` 执行测试逻辑。

#### Lines 1134-1139

```python

    if is_blackwell():
        # Avoid too many rows in TMEM
        MAX_ROWS = 512
        if M * N // 128 // num_ctas > MAX_ROWS:
            N //= (M * N // 128 // num_ctas // MAX_ROWS)
```
- **EN:** Invokes `is_blackwell` to execute the test logic. Branches on runtime or test conditions.
- **CN:** 调用 `is_blackwell` 执行测试逻辑。 根据运行时或测试条件进行分支。

#### Lines 1140-1143

```python

    total_shmem = (M + N) * K * bitwidth // 8
    device = triton.runtime.driver.active.get_current_device()
    MAX_SHMEM = max_shared_mem(device)
```
- **EN:** Prepares or updates state through `total_shmem`, `device`, `MAX_SHMEM`. Invokes `triton.runtime.driver.active.get_current_device`, `max_shared_mem` to execute the test logic. Relevant themes: runtime driver interaction.
- **CN:** 通过 `total_shmem`、`device`、`MAX_SHMEM` 准备或更新状态。 调用 `triton.runtime.driver.active.get_current_device`、`max_shared_mem` 执行测试逻辑。 相关主题：运行时驱动交互。

#### Lines 1144-1145

```python
    if total_shmem > MAX_SHMEM:
        pytest.skip(f"Total shared memory {total_shmem} bytes exceeds the maximum allowed {MAX_SHMEM} bytes")
```
- **EN:** Invokes `pytest.skip` to execute the test logic. Branches on runtime or test conditions.
- **CN:** 调用 `pytest.skip` 执行测试逻辑。 根据运行时或测试条件进行分支。

#### Lines 1146-1149

```python

    if two_ctas and N // ctas_per_cga[1] == 512:
        # grep for [Note: numRepN > 1 and two_ctas]
        pytest.skip("grep for [Note: numRepN > 1 and two_ctas]")
```
- **EN:** Invokes `pytest.skip` to execute the test logic. Branches on runtime or test conditions.
- **CN:** 调用 `pytest.skip` 执行测试逻辑。 根据运行时或测试条件进行分支。

#### Lines 1150-1151

```python

    assert M >= 64, "M must be at least 64 for mmav3 and mmav5"
```
- **EN:** Validates behavior with 1 assertion(s).
- **CN:** 通过 1 个断言验证行为。

#### Lines 1152-1153

```python

    def log2_int(x):
```
- **EN:** Defines the helper function `log2_int`. Parameters: `x`. Key calls include `x.bit_length`.
- **CN:** 定义辅助函数 `log2_int`。 参数：`x`。 关键调用包括 `x.bit_length`。

##### Lines 1154-1154

```python
        return x.bit_length() - 1
```
- **EN:** Invokes `x.bit_length` to execute the test logic.
- **CN:** 调用 `x.bit_length` 执行测试逻辑。

#### Lines 1155-1156

```python

    def get_shared_swizzling_zero(M, K, transpose, cga_layout):
```
- **EN:** Defines the helper function `get_shared_swizzling_zero`. Parameters: `M`, `K`, `transpose`, `cga_layout`. Key calls include `ttgl.SharedLinearLayout`, `get_shared_swizzling_zero`, `log2_int`, `bases.append`, `enumerate`, `math.log2`. This scope touches layout transformation reasoning.
- **CN:** 定义辅助函数 `get_shared_swizzling_zero`。 参数：`M`、`K`、`transpose`、`cga_layout`。 关键调用包括 `ttgl.SharedLinearLayout`、`get_shared_swizzling_zero`、`log2_int`、`bases.append`、`enumerate`、`math.log2`。 该作用域涉及布局变换推理。

##### Lines 1157-1169

```python
        if cga_layout:
            dim_cga = [1, 1]
            for b in cga_layout:
                for i, bi in enumerate(b):
                    if bi != 0:
                        dim_cga[i] *= 2
            cta_shape = (M // dim_cga[0], K // dim_cga[1])
            cta_layout = get_shared_swizzling_zero(cta_shape[0], cta_shape[1], transpose, None)
            cga_bases = list(cga_layout)
            for b in cga_bases:
                for i in range(len(b)):
                    b[i] *= cta_shape[i]
            return ttgl.SharedLinearLayout(cta_layout.offset_bases, cga_bases)
```
- **EN:** Invokes `get_shared_swizzling_zero`, `ttgl.SharedLinearLayout`, `enumerate` to execute the test logic. Branches on runtime or test conditions. Relevant themes: layout transformation reasoning.
- **CN:** 调用 `get_shared_swizzling_zero`、`ttgl.SharedLinearLayout`、`enumerate` 执行测试逻辑。 根据运行时或测试条件进行分支。 相关主题：布局变换推理。

##### Lines 1170-1177

```python
        if transpose:
            assert not cga_layout
            shared = get_shared_swizzling_zero(K, M, False, cga_layout)
            # Transpose the bases
            bases = list(shared.offset_bases)
            for i in range(len(bases)):
                bases[i] = [bases[i][1], bases[i][0]]
            return ttgl.SharedLinearLayout(bases)
```
- **EN:** Invokes `get_shared_swizzling_zero`, `ttgl.SharedLinearLayout` to execute the test logic. Validates behavior with 1 assertion(s). Branches on runtime or test conditions. Relevant themes: layout transformation reasoning.
- **CN:** 调用 `get_shared_swizzling_zero`、`ttgl.SharedLinearLayout` 执行测试逻辑。 通过 1 个断言验证行为。 根据运行时或测试条件进行分支。 相关主题：布局变换推理。

##### Lines 1178-1178

```python
        bases = []
```
- **EN:** Prepares or updates state through `bases`.
- **CN:** 通过 `bases` 准备或更新状态。

##### Lines 1179-1180

```python
        for i in range(log2_int(128 // bitwidth)):
            bases.append([0, 1 << i])
```
- **EN:** Invokes `log2_int`, `bases.append` to execute the test logic. Iterates across cases or data tiles.
- **CN:** 调用 `log2_int`、`bases.append` 执行测试逻辑。 通过循环覆盖多个用例或数据分块。

##### Lines 1181-1182

```python
        for i in range(log2_int(M)):
            bases.append([1 << i, 0])
```
- **EN:** Invokes `log2_int`, `bases.append` to execute the test logic. Iterates across cases or data tiles.
- **CN:** 调用 `log2_int`、`bases.append` 执行测试逻辑。 通过循环覆盖多个用例或数据分块。

##### Lines 1183-1185

```python
        for i in range(log2_int(K // (128 // bitwidth))):
            offset = int(math.log2(128 // bitwidth)) + i
            bases.append([0, 1 << offset])
```
- **EN:** Invokes `log2_int`, `bases.append`, `math.log2` to execute the test logic. Iterates across cases or data tiles.
- **CN:** 调用 `log2_int`、`bases.append`、`math.log2` 执行测试逻辑。 通过循环覆盖多个用例或数据分块。

##### Lines 1186-1186

```python
        return ttgl.SharedLinearLayout(bases)
```
- **EN:** Invokes `ttgl.SharedLinearLayout` to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 调用 `ttgl.SharedLinearLayout` 执行测试逻辑。 相关主题：布局变换推理。

#### Lines 1187-1191

```python

    torch_dtype = torch_dtype_map[bitwidth]
    gl_acc_dtype = acc_dtype_map[acc_dtype]
    out_dtype = torch.float32
    cta_order = [1, 0]
```
- **EN:** Prepares or updates state through `torch_dtype`, `gl_acc_dtype`, `out_dtype`, `cta_order`. Relevant themes: PyTorch tensor setup and checks.
- **CN:** 通过 `torch_dtype`、`gl_acc_dtype`、`out_dtype`、`cta_order` 准备或更新状态。 相关主题：PyTorch 张量准备与校验。

#### Lines 1192-1201

```python

    if two_ctas:
        cga_layout_a = make_2cta_cga_layout(ctas_per_cga, cta_split_a, cta_order, 0)
        cga_layout_b = make_2cta_cga_layout(ctas_per_cga_b, cta_split_b, cta_order, 1)
        # The TMEM layout for instr_m == 128 splits along M, the one for instr_m == 64 splits along N
        cga_layout_c = make_2cta_cga_layout(ctas_per_cga, ctas_per_cga, cta_order, 0)
    else:
        cga_layout_a = make_cga_layout(ctas_per_cga, cta_split_a, cta_order)
        cga_layout_b = make_cga_layout(ctas_per_cga_b, cta_split_b, cta_order)
        cga_layout_c = make_cga_layout(ctas_per_cga, ctas_per_cga, cta_order)
```
- **EN:** Invokes `make_2cta_cga_layout`, `make_cga_layout` to execute the test logic. Branches on runtime or test conditions. Relevant themes: layout transformation reasoning.
- **CN:** 调用 `make_2cta_cga_layout`、`make_cga_layout` 执行测试逻辑。 根据运行时或测试条件进行分支。 相关主题：布局变换推理。

#### Lines 1202-1207

```python
    cga_layout_c = tuple(tuple(basis) for basis in cga_layout_c)

    block_layout_a = ttgl.BlockedLayout([1, 8], [1, THREADS_PER_WARP], warps_per_cta=warps, order=[0, 1],
                                        cga_layout=cga_layout_a)
    block_layout_b = ttgl.BlockedLayout([1, 8], [1, THREADS_PER_WARP], warps_per_cta=warps, order=[1, 0],
                                        cga_layout=cga_layout_b)
```
- **EN:** Prepares or updates state through `cga_layout_c`, `block_layout_a`, `block_layout_b`. Invokes `ttgl.BlockedLayout` to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `cga_layout_c`、`block_layout_a`、`block_layout_b` 准备或更新状态。 调用 `ttgl.BlockedLayout` 执行测试逻辑。 相关主题：布局变换推理。

#### Lines 1208-1212

```python
    if swizzling_a == 0:
        shared_layout_a = get_shared_swizzling_zero(M, K, transpose_a, cga_layout_a)
    else:
        shared_layout_a = ttgl.NVMMASharedLayout(swizzle_byte_width=swizzling_a, element_bitwidth=bitwidth, rank=2,
                                                 transposed=transpose_a, cga_layout=cga_layout_a)
```
- **EN:** Invokes `get_shared_swizzling_zero`, `ttgl.NVMMASharedLayout` to execute the test logic. Branches on runtime or test conditions. Relevant themes: layout transformation reasoning.
- **CN:** 调用 `get_shared_swizzling_zero`、`ttgl.NVMMASharedLayout` 执行测试逻辑。 根据运行时或测试条件进行分支。 相关主题：布局变换推理。

#### Lines 1213-1217

```python
    if swizzling_b == 0:
        shared_layout_b = get_shared_swizzling_zero(K, N, transpose_b, cga_layout_b)
    else:
        shared_layout_b = ttgl.NVMMASharedLayout(swizzle_byte_width=swizzling_b, element_bitwidth=bitwidth, rank=2,
                                                 transposed=transpose_b, cga_layout=cga_layout_b)
```
- **EN:** Invokes `get_shared_swizzling_zero`, `ttgl.NVMMASharedLayout` to execute the test logic. Branches on runtime or test conditions. Relevant themes: layout transformation reasoning.
- **CN:** 调用 `get_shared_swizzling_zero`、`ttgl.NVMMASharedLayout` 执行测试逻辑。 根据运行时或测试条件进行分支。 相关主题：布局变换推理。

#### Lines 1218-1224

```python
    if use_tcgen05:
        tmem_shape = (instr_m, min(N // ctas_per_cga[1], 256))
        acc_layout = TensorMemoryLayout(tmem_shape, col_stride=32 // torch.finfo(acc_dtype).bits,
                                        cga_layout=cga_layout_c, two_ctas=two_ctas)
    else:
        acc_layout = ttgl.NVMMADistributedLayout(version=[3, 0], warps_per_cta=warps, instr_shape=instr_shape,
                                                 cga_layout=cga_layout_c)
```
- **EN:** Invokes `TensorMemoryLayout`, `ttgl.NVMMADistributedLayout`, `min`, `torch.finfo` to execute the test logic. Branches on runtime or test conditions. Relevant themes: PyTorch tensor setup and checks, layout transformation reasoning.
- **CN:** 调用 `TensorMemoryLayout`、`ttgl.NVMMADistributedLayout`、`min`、`torch.finfo` 执行测试逻辑。 根据运行时或测试条件进行分支。 相关主题：PyTorch 张量准备与校验、布局变换推理。

#### Lines 1225-1226

```python

    torch.manual_seed(0)
```
- **EN:** Invokes `torch.manual_seed` to execute the test logic. Relevant themes: PyTorch tensor setup and checks, random-data generation.
- **CN:** 调用 `torch.manual_seed` 执行测试逻辑。 相关主题：PyTorch 张量准备与校验、随机数据生成。

#### Lines 1227-1228

```python

    def cast(x, dtype):
```
- **EN:** Defines the helper function `cast`. Parameters: `x`, `dtype`. Key calls include `x.view`, `x.to`. This scope touches PyTorch tensor setup and checks.
- **CN:** 定义辅助函数 `cast`。 参数：`x`、`dtype`。 关键调用包括 `x.view`、`x.to`。 该作用域涉及PyTorch 张量准备与校验。

##### Lines 1229-1230

```python
        if dtype != torch.float32:
            return x.to(dtype)
```
- **EN:** Invokes `x.to` to execute the test logic. Branches on runtime or test conditions. Relevant themes: PyTorch tensor setup and checks.
- **CN:** 调用 `x.to` 执行测试逻辑。 根据运行时或测试条件进行分支。 相关主题：PyTorch 张量准备与校验。

##### Lines 1231-1237

```python
        # For b16 and fp32 (in both hopper and blackwell it seems)
        # Element-wise multiplication of matrix A and B is performed with specified precision.
        # wgmma.mma_async operation involving type .tf32 will truncate lower 13 bits of the 32-bit
        # input data before multiplication is issued
        x = x.view(torch.int32)
        x = x & ~((1 << 13) - 1)
        return x.view(dtype)
```
- **EN:** Prepares or updates state through `x`. Invokes `x.view` to execute the test logic. Relevant themes: PyTorch tensor setup and checks.
- **CN:** 通过 `x` 准备或更新状态。 调用 `x.view` 执行测试逻辑。 相关主题：PyTorch 张量准备与校验。

#### Lines 1238-1262

```python

    # Sample bf16 as tf32 does not use the full range
    a = cast(torch.randn((M, K), device=device, dtype=torch.float32), torch_dtype)
    b = cast(torch.randn((K, N), device=device, dtype=torch.float32), torch_dtype)
    out = torch.zeros((M, N), device=device, dtype=out_dtype)

    compiled = mma_kernel[(1, )](
        a,
        b,
        out,
        M,
        N,
        K,
        block_layout_a,
        block_layout_b,
        cga_layout_c,
        acc_layout,
        shared_layout_a,
        shared_layout_b,
        gl_acc_dtype,
        False,
        use_tcgen05,
        num_warps=num_warps,
        num_ctas=num_ctas,
    )
```
- **EN:** Prepares or updates state through `a`, `b`, `out`, `compiled`. Invokes `cast`, `torch.randn`, `torch.zeros` to execute the test logic. Relevant themes: PyTorch tensor setup and checks, layout transformation reasoning, random-data generation.
- **CN:** 通过 `a`、`b`、`out`、`compiled` 准备或更新状态。 调用 `cast`、`torch.randn`、`torch.zeros` 执行测试逻辑。 相关主题：PyTorch 张量准备与校验、布局变换推理、随机数据生成。

#### Lines 1263-1264

```python

    assert two_ctas == ("two_ctas" in compiled.asm["ttgir"])
```
- **EN:** Validates behavior with 1 assertion(s).
- **CN:** 通过 1 个断言验证行为。

#### Lines 1265-1266

```python
    if two_ctas:
        assert "fence.mbarrier_init.release.cluster" in compiled.asm["ptx"]
```
- **EN:** Validates behavior with 1 assertion(s). Branches on runtime or test conditions.
- **CN:** 通过 1 个断言验证行为。 根据运行时或测试条件进行分支。

#### Lines 1267-1276

```python

    try:
        allow_tf32 = torch.backends.cuda.matmul.allow_tf32
        torch.backends.cuda.matmul.allow_tf32 = True
        allow_fp16_red = torch.backends.cuda.matmul.allow_fp16_reduced_precision_reduction
        torch.backends.cuda.matmul.allow_fp16_reduced_precision_reduction = acc_dtype == torch.float16
        ref = torch.matmul(a.to(acc_dtype), b.to(acc_dtype)).to(out_dtype)
    finally:
        torch.backends.cuda.matmul.allow_tf32 = allow_tf32
        torch.backends.cuda.matmul.allow_fp16_reduced_precision_reduction = allow_fp16_red
```
- **EN:** Invokes `torch.matmul`, `a.to`, `b.to` to execute the test logic. Wraps operations in exception-handling logic. Relevant themes: PyTorch tensor setup and checks, matrix multiplication workflows.
- **CN:** 调用 `torch.matmul`、`a.to`、`b.to` 执行测试逻辑。 使用异常处理逻辑包裹相关操作。 相关主题：PyTorch 张量准备与校验、矩阵乘法工作流。

#### Lines 1277-1283

```python

    if bitwidth == 8:
        atol, rtol = 5e-2, 5e-1
    elif bitwidth == 16:
        atol, rtol = 5e-2, 5e-1
    else:
        atol, rtol = 5e-4, 5e-3
```
- **EN:** Branches on runtime or test conditions.
- **CN:** 根据运行时或测试条件进行分支。

#### Lines 1284-1284

```python
    torch.testing.assert_close(out, ref, atol=atol, rtol=rtol)
```
- **EN:** Invokes `torch.testing.assert_close` to execute the test logic. Relevant themes: PyTorch tensor setup and checks.
- **CN:** 调用 `torch.testing.assert_close` 执行测试逻辑。 相关主题：PyTorch 张量准备与校验。

### Lines 1285-1291

```python


@pytest.mark.skipif(not is_hip_cdna4(), reason="Requires CDNA4")
@pytest.mark.parametrize("use_buffer_load", [True, False])
def test_amd_direct_load_to_shared(use_buffer_load):

    @gluon.jit
```
- **EN:** Defines the test function `test_amd_direct_load_to_shared`. Decorators: `pytest.mark.skipif(not is_hip_cdna4(), reason='Requires CDNA4')`, `pytest.mark.parametrize('use_buffer_load', [True, False])`. Parameters: `use_buffer_load`. Nested definitions in this scope: `kernel`. Key calls include `pytest.mark.skipif`, `pytest.mark.parametrize`, `torch.manual_seed`, `torch.randn`, `torch.empty_like`, `torch.testing.assert_close`, and 13 more. This scope touches pytest parametrization, PyTorch tensor setup and checks, layout transformation reasoning, random-data generation.
- **CN:** 定义测试函数 `test_amd_direct_load_to_shared`。 装饰器：`pytest.mark.skipif(not is_hip_cdna4(), reason='Requires CDNA4')`、`pytest.mark.parametrize('use_buffer_load', [True, False])`。 参数：`use_buffer_load`。 该作用域中的嵌套定义：`kernel`。 关键调用包括 `pytest.mark.skipif`、`pytest.mark.parametrize`、`torch.manual_seed`、`torch.randn`、`torch.empty_like`、`torch.testing.assert_close` 等另外 13 项。 该作用域涉及pytest 参数化、PyTorch 张量准备与校验、布局变换推理、随机数据生成。

#### Lines 1291-1292

```python
    @gluon.jit
    def kernel(a_ptr, b_ptr, use_buffer_load: ttgl.constexpr):
```
- **EN:** Defines the helper function `kernel`. Decorators: `gluon.jit`. Parameters: `a_ptr`, `b_ptr`, `use_buffer_load`. Key calls include `ttgl.BlockedLayout`, `ttgl.SwizzledSharedLayout`, `ttgl.allocate_shared_memory`, `cdna4_async_copy.commit_group`, `cdna4_async_copy.wait_group`, `cdna4_async_copy.load_shared_relaxed`, and 5 more. This scope touches layout transformation reasoning.
- **CN:** 定义辅助函数 `kernel`。 装饰器：`gluon.jit`。 参数：`a_ptr`、`b_ptr`、`use_buffer_load`。 关键调用包括 `ttgl.BlockedLayout`、`ttgl.SwizzledSharedLayout`、`ttgl.allocate_shared_memory`、`cdna4_async_copy.commit_group`、`cdna4_async_copy.wait_group`、`cdna4_async_copy.load_shared_relaxed` 等另外 5 项。 该作用域涉及布局变换推理。

##### Lines 1293-1298

```python
        blocked: ttgl.constexpr = ttgl.BlockedLayout([1, 8], [32, 2], [4, 1], [1, 0])
        shared: ttgl.constexpr = ttgl.SwizzledSharedLayout(1, 1, 1, order=[1, 0])

        smem = ttgl.allocate_shared_memory(a_ptr.dtype.element_ty, [128, 16], shared)
        offsets = ttgl.arange(0, 128, layout=ttgl.SliceLayout(1, blocked))[:, None] * 16 + \
                  ttgl.arange(0, 16, layout=ttgl.SliceLayout(0, blocked))[None, :]
```
- **EN:** Prepares or updates state through `blocked`, `shared`, `smem`, `offsets`. Invokes `ttgl.BlockedLayout`, `ttgl.SwizzledSharedLayout`, `ttgl.allocate_shared_memory`, `ttgl.arange`, `ttgl.SliceLayout` to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `blocked`、`shared`、`smem`、`offsets` 准备或更新状态。 调用 `ttgl.BlockedLayout`、`ttgl.SwizzledSharedLayout`、`ttgl.allocate_shared_memory`、`ttgl.arange`、`ttgl.SliceLayout` 执行测试逻辑。 相关主题：布局变换推理。

##### Lines 1299-1302

```python
        if use_buffer_load:
            cdna4_async_copy.buffer_load_to_shared(smem, a_ptr, offsets)
        else:
            cdna4_async_copy.global_load_to_shared(smem, a_ptr + offsets)
```
- **EN:** Invokes `cdna4_async_copy.buffer_load_to_shared`, `cdna4_async_copy.global_load_to_shared` to execute the test logic. Branches on runtime or test conditions.
- **CN:** 调用 `cdna4_async_copy.buffer_load_to_shared`、`cdna4_async_copy.global_load_to_shared` 执行测试逻辑。 根据运行时或测试条件进行分支。

##### Lines 1303-1308

```python
        cdna4_async_copy.commit_group()

        cdna4_async_copy.wait_group(0)
        a = cdna4_async_copy.load_shared_relaxed(smem, blocked)

        ttgl.store(b_ptr + offsets, a)
```
- **EN:** Prepares or updates state through `a`. Invokes `cdna4_async_copy.commit_group`, `cdna4_async_copy.wait_group`, `cdna4_async_copy.load_shared_relaxed`, `ttgl.store` to execute the test logic.
- **CN:** 通过 `a` 准备或更新状态。 调用 `cdna4_async_copy.commit_group`、`cdna4_async_copy.wait_group`、`cdna4_async_copy.load_shared_relaxed`、`ttgl.store` 执行测试逻辑。

#### Lines 1309-1315

```python

    torch.manual_seed(0)
    a = torch.randn((128, 16), dtype=torch.float16, device='cuda')
    b = torch.empty_like(a)
    pgm = kernel[(1, )](a, b, use_buffer_load)

    torch.testing.assert_close(a, b)
```
- **EN:** Prepares or updates state through `a`, `b`, `pgm`. Invokes `torch.manual_seed`, `torch.randn`, `torch.empty_like`, `torch.testing.assert_close` to execute the test logic. Relevant themes: PyTorch tensor setup and checks, random-data generation.
- **CN:** 通过 `a`、`b`、`pgm` 准备或更新状态。 调用 `torch.manual_seed`、`torch.randn`、`torch.empty_like`、`torch.testing.assert_close` 执行测试逻辑。 相关主题：PyTorch 张量准备与校验、随机数据生成。

#### Lines 1316-1316

```python
    assert re.search(r'ttg\.local_load .* \{ttg\.amdg\.syncedViaAsyncWait = true\}', pgm.asm['ttgir'], re.MULTILINE)
```
- **EN:** Invokes `re.search` to execute the test logic. Validates behavior with 1 assertion(s).
- **CN:** 调用 `re.search` 执行测试逻辑。 通过 1 个断言验证行为。

#### Lines 1317-1320

```python
    if use_buffer_load:
        assert re.search(r"buffer_load.*lds$", pgm.asm['amdgcn'], re.MULTILINE)
    else:
        assert re.search(r"global_load_lds", pgm.asm['amdgcn'], re.MULTILINE)
```
- **EN:** Invokes `re.search` to execute the test logic. Validates behavior with 2 assertion(s). Branches on runtime or test conditions.
- **CN:** 调用 `re.search` 执行测试逻辑。 通过 2 个断言验证行为。 根据运行时或测试条件进行分支。

#### Lines 1321-1321

```python
    assert 'vmcnt(0)' in pgm.asm['amdgcn']
```
- **EN:** Validates behavior with 1 assertion(s).
- **CN:** 通过 1 个断言验证行为。

### Lines 1322-1329

```python


@pytest.mark.skipif(not (is_hip_rdna3() or is_hip_rdna4()), reason="Requires RDNA3 or RDNA4")
@pytest.mark.parametrize("M, N, K", [(64, 64, 64)])
@pytest.mark.parametrize("in_dtype", ['float16', 'bfloat16'])
def test_amd_wmma(M, N, K, in_dtype):

    @gluon.jit
```
- **EN:** Defines the test function `test_amd_wmma`. Decorators: `pytest.mark.skipif(not (is_hip_rdna3() or is_hip_rdna4()), reason='Requires RDNA3 or RDNA4')`, `pytest.mark.parametrize('M, N, K', [(64, 64, 64)])`, `pytest.mark.parametrize('in_dtype', ['float16', 'bfloat16'])`. Parameters: `M`, `N`, `K`, `in_dtype`. Nested definitions in this scope: `kernel`. Key calls include `pytest.mark.skipif`, `pytest.mark.parametrize`, `torch.randn`, `torch.empty`, `ttgl.BlockedLayout`, `ttgl.amd.AMDWMMALayout`, and 18 more. This scope touches pytest parametrization, PyTorch tensor setup and checks, matrix multiplication workflows, layout transformation reasoning.
- **CN:** 定义测试函数 `test_amd_wmma`。 装饰器：`pytest.mark.skipif(not (is_hip_rdna3() or is_hip_rdna4()), reason='Requires RDNA3 or RDNA4')`、`pytest.mark.parametrize('M, N, K', [(64, 64, 64)])`、`pytest.mark.parametrize('in_dtype', ['float16', 'bfloat16'])`。 参数：`M`、`N`、`K`、`in_dtype`。 该作用域中的嵌套定义：`kernel`。 关键调用包括 `pytest.mark.skipif`、`pytest.mark.parametrize`、`torch.randn`、`torch.empty`、`ttgl.BlockedLayout`、`ttgl.amd.AMDWMMALayout` 等另外 18 项。 该作用域涉及pytest 参数化、PyTorch 张量准备与校验、矩阵乘法工作流、布局变换推理。

#### Lines 1329-1339

```python
    @gluon.jit
    def kernel(a_ptr, b_ptr, c_ptr,  #
               stride_am, stride_ak,  #
               stride_bk, stride_bn,  #
               stride_cm, stride_cn,  #
               BLOCK_SIZE_M: ttgl.constexpr,  #
               BLOCK_SIZE_N: ttgl.constexpr,  #
               BLOCK_SIZE_K: ttgl.constexpr,  #
               BLOCKED_LAYOUT: ttgl.constexpr,  #
               WMMA_LAYOUT: ttgl.constexpr,  #
               K_WIDTH: ttgl.constexpr):
```
- **EN:** Defines the helper function `kernel`. Decorators: `gluon.jit`. Parameters: `a_ptr`, `b_ptr`, `c_ptr`, `stride_am`, `stride_ak`, `stride_bk`, `stride_bn`, `stride_cm`, and 7 more. Key calls include `ttgl.arange`, `ttgl.load`, `ttgl.convert_layout`, `ttgl.zeros`, `c.to`, `ttgl.store`, and 5 more. This scope touches layout transformation reasoning, random-data generation.
- **CN:** 定义辅助函数 `kernel`。 装饰器：`gluon.jit`。 参数：`a_ptr`、`b_ptr`、`c_ptr`、`stride_am`、`stride_ak`、`stride_bk`、`stride_bn`、`stride_cm` 等另外 7 项。 关键调用包括 `ttgl.arange`、`ttgl.load`、`ttgl.convert_layout`、`ttgl.zeros`、`c.to`、`ttgl.store` 等另外 5 项。 该作用域涉及布局变换推理、随机数据生成。

##### Lines 1340-1355

```python
        offs_am = ttgl.arange(0, BLOCK_SIZE_M, layout=ttgl.SliceLayout(1, BLOCKED_LAYOUT))
        offs_bn = ttgl.arange(0, BLOCK_SIZE_N, layout=ttgl.SliceLayout(0, BLOCKED_LAYOUT))

        offs_ak = ttgl.arange(0, BLOCK_SIZE_K, layout=ttgl.SliceLayout(0, BLOCKED_LAYOUT))
        offs_bk = ttgl.arange(0, BLOCK_SIZE_K, layout=ttgl.SliceLayout(1, BLOCKED_LAYOUT))

        offs_a = offs_am[:, None] * stride_am + offs_ak[None, :] * stride_ak
        offs_b = offs_bk[:, None] * stride_bk + offs_bn[None, :] * stride_bn

        a = ttgl.load(a_ptr + offs_a)
        b = ttgl.load(b_ptr + offs_b)

        a = ttgl.convert_layout(a, layout=ttgl.DotOperandLayout(0, WMMA_LAYOUT, K_WIDTH))
        b = ttgl.convert_layout(b, layout=ttgl.DotOperandLayout(1, WMMA_LAYOUT, K_WIDTH))

        acc = ttgl.zeros([BLOCK_SIZE_M, BLOCK_SIZE_N], ttgl.float32, WMMA_LAYOUT)
```
- **EN:** Prepares or updates state through `offs_am`, `offs_bn`, `offs_ak`, `offs_bk`, `offs_a`, `offs_b`, `a`, `b`, and 1 more. Invokes `ttgl.arange`, `ttgl.SliceLayout`, `ttgl.load`, `ttgl.convert_layout`, `ttgl.DotOperandLayout`, `ttgl.zeros` to execute the test logic. Relevant themes: layout transformation reasoning, random-data generation.
- **CN:** 通过 `offs_am`、`offs_bn`、`offs_ak`、`offs_bk`、`offs_a`、`offs_b`、`a`、`b` 等另外 1 项 准备或更新状态。 调用 `ttgl.arange`、`ttgl.SliceLayout`、`ttgl.load`、`ttgl.convert_layout`、`ttgl.DotOperandLayout`、`ttgl.zeros` 执行测试逻辑。 相关主题：布局变换推理、随机数据生成。

##### Lines 1356-1360

```python
        if WMMA_LAYOUT.version == 1:
            c = ttgl.amd.rdna3.wmma(a, b, acc)
        else:
            ttgl.static_assert(WMMA_LAYOUT.version == 2, "WMMA_LAYOUT.version must be 1 or 2")
            c = ttgl.amd.rdna4.wmma(a, b, acc)
```
- **EN:** Invokes `ttgl.amd.rdna3.wmma`, `ttgl.static_assert`, `ttgl.amd.rdna4.wmma` to execute the test logic. Branches on runtime or test conditions. Relevant themes: layout transformation reasoning.
- **CN:** 调用 `ttgl.amd.rdna3.wmma`、`ttgl.static_assert`、`ttgl.amd.rdna4.wmma` 执行测试逻辑。 根据运行时或测试条件进行分支。 相关主题：布局变换推理。

##### Lines 1361-1366

```python
        c = c.to(a_ptr.dtype.element_ty)

        offs_cm = ttgl.arange(0, BLOCK_SIZE_M, layout=ttgl.SliceLayout(1, WMMA_LAYOUT))
        offs_cn = ttgl.arange(0, BLOCK_SIZE_N, layout=ttgl.SliceLayout(0, WMMA_LAYOUT))
        offs_c = offs_cm[:, None] * stride_cm + offs_cn[None, :] * stride_cn
        ttgl.store(c_ptr + offs_c, c)
```
- **EN:** Prepares or updates state through `c`, `offs_cm`, `offs_cn`, `offs_c`. Invokes `c.to`, `ttgl.arange`, `ttgl.SliceLayout`, `ttgl.store` to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `c`、`offs_cm`、`offs_cn`、`offs_c` 准备或更新状态。 调用 `c.to`、`ttgl.arange`、`ttgl.SliceLayout`、`ttgl.store` 执行测试逻辑。 相关主题：布局变换推理。

#### Lines 1367-1382

```python

    elem_type = torch.float16 if in_dtype == 'float16' else torch.bfloat16
    a = torch.randn((M, K), device='cuda', dtype=elem_type)
    b = torch.randn((K, N), device='cuda', dtype=elem_type)
    c = torch.empty((M, N), device=a.device, dtype=elem_type)

    blocked = ttgl.BlockedLayout([1, 8], [4, 8], [4, 1], [1, 0])
    wmma_version = 1 if is_hip_rdna3() else 2
    k_width = 16 if is_hip_rdna3() else 8
    wmma = ttgl.amd.AMDWMMALayout(wmma_version, True, [[0, 1], [1, 0]])
    kernel[1, 1](a, b, c, a.stride(0), a.stride(1), b.stride(0), b.stride(1), c.stride(0), c.stride(1), BLOCK_SIZE_M=M,
                 BLOCK_SIZE_N=N, BLOCK_SIZE_K=K, BLOCKED_LAYOUT=blocked, WMMA_LAYOUT=wmma, K_WIDTH=k_width, num_warps=4)

    ref = torch.matmul(a, b)
    triton_output = c
    torch.testing.assert_close(ref, triton_output)
```
- **EN:** Prepares or updates state through `elem_type`, `a`, `b`, `c`, `blocked`, `wmma_version`, `k_width`, `wmma`, and 2 more. Invokes `torch.randn`, `torch.empty`, `ttgl.BlockedLayout`, `is_hip_rdna3`, `ttgl.amd.AMDWMMALayout`, `a.stride`, and 4 more to execute the test logic. Relevant themes: PyTorch tensor setup and checks, matrix multiplication workflows, layout transformation reasoning, random-data generation.
- **CN:** 通过 `elem_type`、`a`、`b`、`c`、`blocked`、`wmma_version`、`k_width`、`wmma` 等另外 2 项 准备或更新状态。 调用 `torch.randn`、`torch.empty`、`ttgl.BlockedLayout`、`is_hip_rdna3`、`ttgl.amd.AMDWMMALayout`、`a.stride` 等另外 4 项 执行测试逻辑。 相关主题：PyTorch 张量准备与校验、矩阵乘法工作流、布局变换推理、随机数据生成。

### Lines 1383-1390

```python


@pytest.mark.skipif(not (is_hip_cdna3() or is_hip_cdna4()), reason="Requires CDNA3 or CDNA4")
@pytest.mark.parametrize("M, N, K", [(32, 32, 16), (16, 16, 32)])
@pytest.mark.parametrize("in_dtype", ['float16', 'bfloat16'])
@pytest.mark.parametrize("num_warps", [4, 8])
@pytest.mark.parametrize("cdna_version", [3, 4])
def test_amd_mfma(M, N, K, in_dtype, num_warps, cdna_version):
```
- **EN:** Defines the test function `test_amd_mfma`. Decorators: `pytest.mark.skipif(not (is_hip_cdna3() or is_hip_cdna4()), reason='Requires CDNA3 or CDNA4')`, `pytest.mark.parametrize('M, N, K', [(32, 32, 16), (16, 16, 32)])`, `pytest.mark.parametrize('in_dtype', ['float16', 'bfloat16'])`, `pytest.mark.parametrize('num_warps', [4, 8])`, `pytest.mark.parametrize('cdna_version', [3, 4])`. Parameters: `M`, `N`, `K`, `in_dtype`, `num_warps`, `cdna_version`. Nested definitions in this scope: `kernel`. Key calls include `pytest.mark.skipif`, `pytest.mark.parametrize`, `torch.empty`, `ttgl.BlockedLayout`, `ttgl.amd.AMDMFMALayout`, `torch.matmul`, and 17 more. This scope touches pytest parametrization, PyTorch tensor setup and checks, matrix multiplication workflows, layout transformation reasoning.
- **CN:** 定义测试函数 `test_amd_mfma`。 装饰器：`pytest.mark.skipif(not (is_hip_cdna3() or is_hip_cdna4()), reason='Requires CDNA3 or CDNA4')`、`pytest.mark.parametrize('M, N, K', [(32, 32, 16), (16, 16, 32)])`、`pytest.mark.parametrize('in_dtype', ['float16', 'bfloat16'])`、`pytest.mark.parametrize('num_warps', [4, 8])`、`pytest.mark.parametrize('cdna_version', [3, 4])`。 参数：`M`、`N`、`K`、`in_dtype`、`num_warps`、`cdna_version`。 该作用域中的嵌套定义：`kernel`。 关键调用包括 `pytest.mark.skipif`、`pytest.mark.parametrize`、`torch.empty`、`ttgl.BlockedLayout`、`ttgl.amd.AMDMFMALayout`、`torch.matmul` 等另外 17 项。 该作用域涉及pytest 参数化、PyTorch 张量准备与校验、矩阵乘法工作流、布局变换推理。

#### Lines 1391-1392

```python
    if is_hip_cdna3() and cdna_version != 3:
        pytest.skip("On CDNA3 target, skip if mfma version is not 3")
```
- **EN:** Invokes `is_hip_cdna3`, `pytest.skip` to execute the test logic. Branches on runtime or test conditions.
- **CN:** 调用 `is_hip_cdna3`、`pytest.skip` 执行测试逻辑。 根据运行时或测试条件进行分支。

#### Lines 1393-1395

```python

    if is_hip_cdna4() and cdna_version != 4:
        pytest.skip("On CDNA4 target, skip if mfma version is not 4")
```
- **EN:** Invokes `is_hip_cdna4`, `pytest.skip` to execute the test logic. Branches on runtime or test conditions.
- **CN:** 调用 `is_hip_cdna4`、`pytest.skip` 执行测试逻辑。 根据运行时或测试条件进行分支。

#### Lines 1396-1403

```python

    @gluon.jit
    def kernel(a_ptr, b_ptr, c_ptr,  #
               stride_am, stride_ak,  #
               stride_bk, stride_bn,  #
               stride_cm, stride_cn,  #
               BLOCK_SIZE_M: ttgl.constexpr, BLOCK_SIZE_N: ttgl.constexpr, BLOCK_SIZE_K: ttgl.constexpr,
               blocked: ttgl.constexpr, k_width: ttgl.constexpr, mfma_layout: ttgl.constexpr):
```
- **EN:** Defines the helper function `kernel`. Decorators: `gluon.jit`. Parameters: `a_ptr`, `b_ptr`, `c_ptr`, `stride_am`, `stride_ak`, `stride_bk`, `stride_bn`, `stride_cm`, and 7 more. Key calls include `ttgl.DotOperandLayout`, `ttgl.arange`, `ttgl.amd.cdna3.buffer_load`, `ttgl.convert_layout`, `ttgl.zeros`, `ttgl.amd.cdna3.mfma`, and 3 more. This scope touches layout transformation reasoning, random-data generation.
- **CN:** 定义辅助函数 `kernel`。 装饰器：`gluon.jit`。 参数：`a_ptr`、`b_ptr`、`c_ptr`、`stride_am`、`stride_ak`、`stride_bk`、`stride_bn`、`stride_cm` 等另外 7 项。 关键调用包括 `ttgl.DotOperandLayout`、`ttgl.arange`、`ttgl.amd.cdna3.buffer_load`、`ttgl.convert_layout`、`ttgl.zeros`、`ttgl.amd.cdna3.mfma` 等另外 3 项。 该作用域涉及布局变换推理、随机数据生成。

##### Lines 1404-1427

```python
        dot_a_layout: ttgl.constexpr = ttgl.DotOperandLayout(operand_index=0, parent=mfma_layout, k_width=k_width)
        dot_b_layout: ttgl.constexpr = ttgl.DotOperandLayout(operand_index=1, parent=mfma_layout, k_width=k_width)

        offs_am = ttgl.arange(0, BLOCK_SIZE_M, layout=ttgl.SliceLayout(1, blocked))
        offs_bn = ttgl.arange(0, BLOCK_SIZE_N, layout=ttgl.SliceLayout(0, blocked))

        offs_ak = ttgl.arange(0, BLOCK_SIZE_K, layout=ttgl.SliceLayout(0, blocked))
        offs_bk = ttgl.arange(0, BLOCK_SIZE_K, layout=ttgl.SliceLayout(1, blocked))
        offs_a = offs_am[:, None] * stride_am + offs_ak[None, :] * stride_ak
        offs_b = offs_bk[:, None] * stride_bk + offs_bn[None, :] * stride_bn

        a = ttgl.amd.cdna3.buffer_load(ptr=a_ptr, offsets=offs_a)
        b = ttgl.amd.cdna3.buffer_load(ptr=b_ptr, offsets=offs_b)
        a1 = ttgl.convert_layout(a, layout=dot_a_layout)
        b1 = ttgl.convert_layout(b, layout=dot_b_layout)
        acc = ttgl.zeros([BLOCK_SIZE_M, BLOCK_SIZE_N], ttgl.float32, mfma_layout)
        c = ttgl.amd.cdna3.mfma(a1, b1, acc)
        c = ttgl.convert_layout(c, layout=blocked)
        c = c.to(a_ptr.dtype.element_ty)

        offs_cm = ttgl.arange(0, BLOCK_SIZE_M)
        offs_cn = ttgl.arange(0, BLOCK_SIZE_N)
        offs_c = offs_cm[:, None] * stride_cm + offs_cn[None, :] * stride_cn
        ttgl.amd.cdna3.buffer_store(stored_value=c, ptr=c_ptr, offsets=offs_c)
```
- **EN:** Prepares or updates state through `dot_a_layout`, `dot_b_layout`, `offs_am`, `offs_bn`, `offs_ak`, `offs_bk`, `offs_a`, `offs_b`, and 9 more. Invokes `ttgl.DotOperandLayout`, `ttgl.arange`, `ttgl.SliceLayout`, `ttgl.amd.cdna3.buffer_load`, `ttgl.convert_layout`, `ttgl.zeros`, and 3 more to execute the test logic. Relevant themes: layout transformation reasoning, random-data generation.
- **CN:** 通过 `dot_a_layout`、`dot_b_layout`、`offs_am`、`offs_bn`、`offs_ak`、`offs_bk`、`offs_a`、`offs_b` 等另外 9 项 准备或更新状态。 调用 `ttgl.DotOperandLayout`、`ttgl.arange`、`ttgl.SliceLayout`、`ttgl.amd.cdna3.buffer_load`、`ttgl.convert_layout`、`ttgl.zeros` 等另外 3 项 执行测试逻辑。 相关主题：布局变换推理、随机数据生成。

#### Lines 1428-1452

```python

    elem_type = torch.float16 if in_dtype == 'float16' else torch.bfloat16
    a = torch.randn((M, K), device='cuda', dtype=elem_type) - 0.5
    b = torch.randn((K, N), device='cuda', dtype=elem_type) - 0.5
    c = torch.empty((M, N), device=a.device, dtype=elem_type)
    nonkdim: ttgl.constexpr = 32
    kdim: ttgl.constexpr = 8 if cdna_version == 3 else 16
    k_width: ttgl.constexpr = 4 if cdna_version == 3 else 8
    blocked: ttgl.constexpr = ttgl.BlockedLayout(size_per_thread=[4, 4], threads_per_warp=[4, 16],
                                                 warps_per_cta=[num_warps, 1], order=[1, 0])
    mfma_layout: ttgl.constexpr = ttgl.amd.AMDMFMALayout(version=cdna_version, instr_shape=[nonkdim, nonkdim, kdim],
                                                         transposed=True, warps_per_cta=[num_warps, 1])

    kernel[1, 1](
        a, b, c,  #
        a.stride(0), a.stride(1),  #
        b.stride(0), b.stride(1),  #
        c.stride(0), c.stride(1),  #
        BLOCK_SIZE_M=M, BLOCK_SIZE_N=N, BLOCK_SIZE_K=K,  #
        blocked=blocked, k_width=k_width, mfma_layout=mfma_layout,  #
        num_warps=num_warps)

    ref = torch.matmul(a, b)
    triton_output = c
    torch.testing.assert_close(ref, triton_output)
```
- **EN:** Prepares or updates state through `elem_type`, `a`, `b`, `c`, `nonkdim`, `kdim`, `k_width`, `blocked`, and 3 more. Invokes `torch.randn`, `torch.empty`, `ttgl.BlockedLayout`, `ttgl.amd.AMDMFMALayout`, `a.stride`, `b.stride`, and 3 more to execute the test logic. Relevant themes: PyTorch tensor setup and checks, matrix multiplication workflows, layout transformation reasoning, random-data generation.
- **CN:** 通过 `elem_type`、`a`、`b`、`c`、`nonkdim`、`kdim`、`k_width`、`blocked` 等另外 3 项 准备或更新状态。 调用 `torch.randn`、`torch.empty`、`ttgl.BlockedLayout`、`ttgl.amd.AMDMFMALayout`、`a.stride`、`b.stride` 等另外 3 项 执行测试逻辑。 相关主题：PyTorch 张量准备与校验、矩阵乘法工作流、布局变换推理、随机数据生成。

### Lines 1453-1463

```python


@pytest.mark.skipif(not is_hip_cdna4(), reason="Requires CDNA4")
@pytest.mark.parametrize("M, N, K", [(32, 32, 128)])
@pytest.mark.parametrize("a_type, b_type", [(a_type, b_type)
                                            for a_type in ["e2m1", "e4m3", "e5m2"]
                                            for b_type in ["e2m1", "e4m3", "e5m2"]])
@pytest.mark.parametrize("has_scale", [True, False])
def test_amd_mfma_scaled(M, N, K, a_type, b_type, has_scale, device='cuda'):

    @gluon.jit
```
- **EN:** Defines the test function `test_amd_mfma_scaled`. Decorators: `pytest.mark.skipif(not is_hip_cdna4(), reason='Requires CDNA4')`, `pytest.mark.parametrize('M, N, K', [(32, 32, 128)])`, `pytest.mark.parametrize('a_type, b_type', [(a_type, b_type) for a_type in ['e2m1', 'e4m3', 'e5m2'] for b_type in ['e2m1', 'e4m3', 'e5m2']])`, `pytest.mark.parametrize('has_scale', [True, False])`. Parameters: `M`, `N`, `K`, `a_type`, `b_type`, `has_scale`, `device`. Nested definitions in this scope: `kernel`, `_create_mxfp_operand`, `_create_mxfp_scale`. Key calls include `pytest.mark.skipif`, `pytest.mark.parametrize`, `torch.manual_seed`, `_create_mxfp_operand`, `ttgl.amd.AMDMFMALayout`, `ttgl.BlockedLayout`, and 27 more. This scope touches pytest parametrization, Triton language operations, PyTorch tensor setup and checks, matrix multiplication workflows.
- **CN:** 定义测试函数 `test_amd_mfma_scaled`。 装饰器：`pytest.mark.skipif(not is_hip_cdna4(), reason='Requires CDNA4')`、`pytest.mark.parametrize('M, N, K', [(32, 32, 128)])`、`pytest.mark.parametrize('a_type, b_type', [(a_type, b_type) for a_type in ['e2m1', 'e4m3', 'e5m2'] for b_type in ['e2m1', 'e4m3', 'e5m2']])`、`pytest.mark.parametrize('has_scale', [True, False])`。 参数：`M`、`N`、`K`、`a_type`、`b_type`、`has_scale`、`device`。 该作用域中的嵌套定义：`kernel`、`_create_mxfp_operand`、`_create_mxfp_scale`。 关键调用包括 `pytest.mark.skipif`、`pytest.mark.parametrize`、`torch.manual_seed`、`_create_mxfp_operand`、`ttgl.amd.AMDMFMALayout`、`ttgl.BlockedLayout` 等另外 27 项。 该作用域涉及pytest 参数化、Triton language 操作、PyTorch 张量准备与校验、矩阵乘法工作流。

#### Lines 1463-1466

```python
    @gluon.jit
    def kernel(out_ptr, a_ptr, b_ptr, a_scale_ptr, b_scale_ptr,  #
               M: ttgl.constexpr, N: ttgl.constexpr, K: ttgl.constexpr,  #
               a_type: tl.constexpr, b_type: tl.constexpr):
```
- **EN:** Defines the helper function `kernel`. Decorators: `gluon.jit`. Parameters: `out_ptr`, `a_ptr`, `b_ptr`, `a_scale_ptr`, `b_scale_ptr`, `M`, `N`, `K`, and 2 more. Key calls include `ttgl.amd.AMDMFMALayout`, `ttgl.BlockedLayout`, `ttgl.DotOperandLayout`, `ttgl.amd.cdna4.get_mfma_scale_layout`, `ttgl.amd.cdna4.buffer_load`, `ttgl.convert_layout`, and 6 more. This scope touches Triton language operations, layout transformation reasoning, random-data generation.
- **CN:** 定义辅助函数 `kernel`。 装饰器：`gluon.jit`。 参数：`out_ptr`、`a_ptr`、`b_ptr`、`a_scale_ptr`、`b_scale_ptr`、`M`、`N`、`K` 等另外 2 项。 关键调用包括 `ttgl.amd.AMDMFMALayout`、`ttgl.BlockedLayout`、`ttgl.DotOperandLayout`、`ttgl.amd.cdna4.get_mfma_scale_layout`、`ttgl.amd.cdna4.buffer_load`、`ttgl.convert_layout` 等另外 6 项。 该作用域涉及Triton language 操作、布局变换推理、随机数据生成。

##### Lines 1467-1497

```python
        DIV_FACTOR_A: tl.constexpr = 2 if a_type == "e2m1" else 1
        DIV_FACTOR_B: tl.constexpr = 2 if b_type == "e2m1" else 1
        K_A: tl.constexpr = K // DIV_FACTOR_A
        K_B: tl.constexpr = K // DIV_FACTOR_B

        mfma_layout: ttgl.constexpr = ttgl.amd.AMDMFMALayout(version=4, instr_shape=[16, 16, 128], transposed=True,
                                                             warps_per_cta=[2, 2])

        a_unpacked_layout: ttgl.constexpr = ttgl.BlockedLayout([1, 16], [8, 8], [4, 1], [1, 0])
        a_packed_layout: ttgl.constexpr = ttgl.BlockedLayout([1, 8], [8, 8], [4, 1], [1, 0])
        a_load_layout: ttgl.constexpr = a_packed_layout if a_type == "e2m1" else a_unpacked_layout
        a_layout: ttgl.constexpr = ttgl.DotOperandLayout(operand_index=0, parent=mfma_layout, k_width=16)
        a_scale_layout: ttgl.constexpr = ttgl.amd.cdna4.get_mfma_scale_layout(a_layout, [M, K // 32])

        b_unpacked_layout: ttgl.constexpr = ttgl.BlockedLayout([1, 16], [32, 2], [4, 1], [1, 0])
        b_packed_layout: ttgl.constexpr = ttgl.BlockedLayout([1, 8], [16, 4], [4, 1], [1, 0])
        b_load_layout: ttgl.constexpr = b_packed_layout if b_type == "e2m1" else b_unpacked_layout
        b_layout: ttgl.constexpr = ttgl.DotOperandLayout(operand_index=1, parent=mfma_layout, k_width=16)
        b_scale_layout: ttgl.constexpr = ttgl.amd.cdna4.get_mfma_scale_layout(b_layout, [N, K // 32])

        a_offs_m = ttgl.arange(0, M, layout=ttgl.SliceLayout(1, a_load_layout))[:, None]
        a_offs_k = ttgl.arange(0, K_A, layout=ttgl.SliceLayout(0, a_load_layout))[None, :]
        a = ttgl.amd.cdna4.buffer_load(a_ptr, a_offs_m * K_A + a_offs_k)
        a = ttgl.convert_layout(a, a_layout)

        b_offs_k = ttgl.arange(0, K_B, layout=ttgl.SliceLayout(1, b_load_layout))[:, None]
        b_offs_n = ttgl.arange(0, N, layout=ttgl.SliceLayout(0, b_load_layout))[None, :]
        b = ttgl.amd.cdna4.buffer_load(b_ptr, b_offs_k * N + b_offs_n)
        b = ttgl.convert_layout(b, b_layout)

        a_scale = None
```
- **EN:** Prepares or updates state through `DIV_FACTOR_A`, `DIV_FACTOR_B`, `K_A`, `K_B`, `mfma_layout`, `a_unpacked_layout`, `a_packed_layout`, `a_load_layout`, and 14 more. Invokes `ttgl.amd.AMDMFMALayout`, `ttgl.BlockedLayout`, `ttgl.DotOperandLayout`, `ttgl.amd.cdna4.get_mfma_scale_layout`, `ttgl.arange`, `ttgl.SliceLayout`, and 2 more to execute the test logic. Relevant themes: Triton language operations, layout transformation reasoning, random-data generation.
- **CN:** 通过 `DIV_FACTOR_A`、`DIV_FACTOR_B`、`K_A`、`K_B`、`mfma_layout`、`a_unpacked_layout`、`a_packed_layout`、`a_load_layout` 等另外 14 项 准备或更新状态。 调用 `ttgl.amd.AMDMFMALayout`、`ttgl.BlockedLayout`、`ttgl.DotOperandLayout`、`ttgl.amd.cdna4.get_mfma_scale_layout`、`ttgl.arange`、`ttgl.SliceLayout` 等另外 2 项 执行测试逻辑。 相关主题：Triton language 操作、布局变换推理、随机数据生成。

##### Lines 1498-1501

```python
        if a_scale_ptr is not None:
            a_scale_offs_m = ttgl.arange(0, M, layout=ttgl.SliceLayout(1, a_scale_layout))[:, None]
            a_scale_offs_k = ttgl.arange(0, K // 32, layout=ttgl.SliceLayout(0, a_scale_layout))[None, :]
            a_scale = ttgl.amd.cdna4.buffer_load(a_scale_ptr, a_scale_offs_m * (K // 32) + a_scale_offs_k)
```
- **EN:** Invokes `ttgl.amd.cdna4.buffer_load`, `ttgl.arange`, `ttgl.SliceLayout` to execute the test logic. Branches on runtime or test conditions. Relevant themes: layout transformation reasoning.
- **CN:** 调用 `ttgl.amd.cdna4.buffer_load`、`ttgl.arange`、`ttgl.SliceLayout` 执行测试逻辑。 根据运行时或测试条件进行分支。 相关主题：布局变换推理。

##### Lines 1502-1503

```python

        b_scale = None
```
- **EN:** Prepares or updates state through `b_scale`.
- **CN:** 通过 `b_scale` 准备或更新状态。

##### Lines 1504-1507

```python
        if b_scale_ptr is not None:
            b_scale_offs_n = ttgl.arange(0, N, layout=ttgl.SliceLayout(1, b_scale_layout))[:, None]
            b_scale_offs_k = ttgl.arange(0, K // 32, layout=ttgl.SliceLayout(0, b_scale_layout))[None, :]
            b_scale = ttgl.amd.cdna4.buffer_load(b_scale_ptr, b_scale_offs_n * (K // 32) + b_scale_offs_k)
```
- **EN:** Invokes `ttgl.amd.cdna4.buffer_load`, `ttgl.arange`, `ttgl.SliceLayout` to execute the test logic. Branches on runtime or test conditions. Relevant themes: layout transformation reasoning.
- **CN:** 调用 `ttgl.amd.cdna4.buffer_load`、`ttgl.arange`、`ttgl.SliceLayout` 执行测试逻辑。 根据运行时或测试条件进行分支。 相关主题：布局变换推理。

##### Lines 1508-1515

```python

        zero = ttgl.zeros([M, N], dtype=ttgl.float32, layout=mfma_layout)
        c = ttgl.amd.cdna4.mfma_scaled(a, a_scale, a_type, b, b_scale, b_type, zero)
        c = c.to(out_ptr.dtype.element_ty)

        out_offs_m = ttgl.arange(0, M)[:, None]
        out_offs_n = ttgl.arange(0, N)[None, :]
        ttgl.amd.cdna4.buffer_store(c, out_ptr, out_offs_m * N + out_offs_n)
```
- **EN:** Prepares or updates state through `zero`, `c`, `out_offs_m`, `out_offs_n`. Invokes `ttgl.zeros`, `ttgl.amd.cdna4.mfma_scaled`, `c.to`, `ttgl.arange`, `ttgl.amd.cdna4.buffer_store` to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `zero`、`c`、`out_offs_m`、`out_offs_n` 准备或更新状态。 调用 `ttgl.zeros`、`ttgl.amd.cdna4.mfma_scaled`、`c.to`、`ttgl.arange`、`ttgl.amd.cdna4.buffer_store` 执行测试逻辑。 相关主题：布局变换推理。

#### Lines 1516-1517

```python

    def _create_mxfp_operand(operand: int, m: int, n: int, dtype: str):
```
- **EN:** Defines the helper function `_create_mxfp_operand`. Parameters: `operand`, `m`, `n`, `dtype`. Key calls include `torch.randint`, `v.to`, `v_ref.to`, `v_mxfp4.to_packed_tensor`, `v_mxfp4.to`, `v.view`, and 1 more. This scope touches PyTorch tensor setup and checks, random-data generation.
- **CN:** 定义辅助函数 `_create_mxfp_operand`。 参数：`operand`、`m`、`n`、`dtype`。 关键调用包括 `torch.randint`、`v.to`、`v_ref.to`、`v_mxfp4.to_packed_tensor`、`v_mxfp4.to`、`v.view` 等另外 1 项。 该作用域涉及PyTorch 张量准备与校验、随机数据生成。

##### Lines 1518-1518

```python
        size = (m, n)
```
- **EN:** Prepares or updates state through `size`.
- **CN:** 通过 `size` 准备或更新状态。

##### Lines 1519-1530

```python
        if dtype == 'e4m3':
            v = torch.randint(20, 40, size, dtype=torch.uint8)
            v_ref = v.view(torch.float8_e4m3fn).to(torch.float32)
        elif dtype == 'e5m2':
            v = torch.randint(20, 40, size, dtype=torch.uint8)
            v_ref = v.view(torch.float8_e5m2).to(torch.float32)
        else:
            assert dtype == 'e2m1'
            pack_dim = 1 if operand == 0 else 0
            v_mxfp4 = MXFP4Tensor(size=size).random()
            v = v_mxfp4.to_packed_tensor(pack_dim)
            v_ref = v_mxfp4.to(torch.float32)
```
- **EN:** Invokes `torch.randint`, `v_mxfp4.to_packed_tensor`, `v_mxfp4.to`, `v.view`, `MXFP4Tensor` to execute the test logic. Validates behavior with 1 assertion(s). Branches on runtime or test conditions. Relevant themes: PyTorch tensor setup and checks, random-data generation.
- **CN:** 调用 `torch.randint`、`v_mxfp4.to_packed_tensor`、`v_mxfp4.to`、`v.view`、`MXFP4Tensor` 执行测试逻辑。 通过 1 个断言验证行为。 根据运行时或测试条件进行分支。 相关主题：PyTorch 张量准备与校验、随机数据生成。

##### Lines 1531-1531

```python
        return v.to(device), v_ref.to(device)
```
- **EN:** Invokes `v.to`, `v_ref.to` to execute the test logic.
- **CN:** 调用 `v.to`、`v_ref.to` 执行测试逻辑。

#### Lines 1532-1533

```python

    def _create_mxfp_scale(operand: int, m: int, n: int):
```
- **EN:** Defines the helper function `_create_mxfp_scale`. Parameters: `operand`, `m`, `n`. Key calls include `scale_ref.T.contiguous`, `scale.data.to`, `scale_ref.to`, `MXScaleTensor`, `scale.to`. This scope touches PyTorch tensor setup and checks, random-data generation.
- **CN:** 定义辅助函数 `_create_mxfp_scale`。 参数：`operand`、`m`、`n`。 关键调用包括 `scale_ref.T.contiguous`、`scale.data.to`、`scale_ref.to`、`MXScaleTensor`、`scale.to`。 该作用域涉及PyTorch 张量准备与校验、随机数据生成。

##### Lines 1534-1538

```python
        size = (m, n // 32)
        scale = MXScaleTensor(size=tuple(size)).random(1 / 32, 32)
        scale_ref = scale.to(torch.float32).repeat_interleave(32, dim=1)
        scale_ref = scale_ref.T.contiguous() if operand == 1 else scale_ref
        return scale.data.to(device), scale_ref.to(device)
```
- **EN:** Prepares or updates state through `size`, `scale`, `scale_ref`. Invokes `MXScaleTensor`, `scale.to`, `scale_ref.T.contiguous`, `scale.data.to`, `scale_ref.to` to execute the test logic. Relevant themes: PyTorch tensor setup and checks, random-data generation.
- **CN:** 通过 `size`、`scale`、`scale_ref` 准备或更新状态。 调用 `MXScaleTensor`、`scale.to`、`scale_ref.T.contiguous`、`scale.data.to`、`scale_ref.to` 执行测试逻辑。 相关主题：PyTorch 张量准备与校验、随机数据生成。

#### Lines 1539-1542

```python

    torch.manual_seed(0)
    a, a_ref = _create_mxfp_operand(0, M, K, a_type)
    b, b_ref = _create_mxfp_operand(1, K, N, b_type)
```
- **EN:** Prepares or updates state through `a`, `a_ref`, `b`, `b_ref`. Invokes `torch.manual_seed`, `_create_mxfp_operand` to execute the test logic. Relevant themes: PyTorch tensor setup and checks, random-data generation.
- **CN:** 通过 `a`、`a_ref`、`b`、`b_ref` 准备或更新状态。 调用 `torch.manual_seed`、`_create_mxfp_operand` 执行测试逻辑。 相关主题：PyTorch 张量准备与校验、随机数据生成。

#### Lines 1543-1555

```python

    if has_scale:
        a_scale, a_scale_ref = _create_mxfp_scale(0, M, K)
        b_scale, b_scale_ref = _create_mxfp_scale(1, N, K)
        out = torch.empty((M, N), dtype=torch.float32, device=device)
        compiled = kernel[(1, )](out, a, b, a_scale, b_scale, M, N, K, a_type, b_type, num_warps=4)
        out_ref = torch.matmul(a_ref * a_scale_ref, b_ref * b_scale_ref)
        torch.testing.assert_close(out, out_ref)
    else:
        out = torch.empty((M, N), dtype=torch.float32, device=device)
        compiled = kernel[(1, )](out, a, b, None, None, M, N, K, a_type, b_type, num_warps=4)
        out_ref = torch.matmul(a_ref, b_ref)
        torch.testing.assert_close(out, out_ref)
```
- **EN:** Invokes `_create_mxfp_scale`, `torch.empty`, `torch.matmul`, `torch.testing.assert_close` to execute the test logic. Branches on runtime or test conditions. Relevant themes: PyTorch tensor setup and checks, matrix multiplication workflows.
- **CN:** 调用 `_create_mxfp_scale`、`torch.empty`、`torch.matmul`、`torch.testing.assert_close` 执行测试逻辑。 根据运行时或测试条件进行分支。 相关主题：PyTorch 张量准备与校验、矩阵乘法工作流。

#### Lines 1556-1557

```python

    assert 'v_mfma_scale_f32_16x16x128_f8f6f4' in compiled.asm['amdgcn']
```
- **EN:** Validates behavior with 1 assertion(s).
- **CN:** 通过 1 个断言验证行为。

### Lines 1558-1562

```python


def test_math_fast_expf():

    @gluon.jit
```
- **EN:** Defines the test function `test_math_fast_expf`. Nested definitions in this scope: `fast_expf_kernel`. Key calls include `torch.manual_seed`, `torch.randn`, `torch.empty_like`, `torch.testing.assert_close`, `ttgl.BlockedLayout`, `ttgl.arange`, and 4 more. This scope touches PyTorch tensor setup and checks, layout transformation reasoning, random-data generation, libdevice coverage.
- **CN:** 定义测试函数 `test_math_fast_expf`。 该作用域中的嵌套定义：`fast_expf_kernel`。 关键调用包括 `torch.manual_seed`、`torch.randn`、`torch.empty_like`、`torch.testing.assert_close`、`ttgl.BlockedLayout`、`ttgl.arange` 等另外 4 项。 该作用域涉及PyTorch 张量准备与校验、布局变换推理、随机数据生成、libdevice 覆盖。

#### Lines 1562-1563

```python
    @gluon.jit
    def fast_expf_kernel(x_ptr, y_ptr, warp_size: ttgl.constexpr, num_warps: ttgl.constexpr):
```
- **EN:** Defines the helper function `fast_expf_kernel`. Decorators: `gluon.jit`. Parameters: `x_ptr`, `y_ptr`, `warp_size`, `num_warps`. Key calls include `ttgl.BlockedLayout`, `ttgl.arange`, `ttgl.load`, `libdevice.fast_expf`, `ttgl.store`. This scope touches layout transformation reasoning, libdevice coverage.
- **CN:** 定义辅助函数 `fast_expf_kernel`。 装饰器：`gluon.jit`。 参数：`x_ptr`、`y_ptr`、`warp_size`、`num_warps`。 关键调用包括 `ttgl.BlockedLayout`、`ttgl.arange`、`ttgl.load`、`libdevice.fast_expf`、`ttgl.store`。 该作用域涉及布局变换推理、libdevice 覆盖。

##### Lines 1564-1569

```python
        blocked: ttgl.constexpr = ttgl.BlockedLayout([1], [warp_size], [num_warps], [0])

        offs = ttgl.arange(0, warp_size * num_warps, layout=blocked)
        x = ttgl.load(x_ptr + offs)
        y = libdevice.fast_expf(x)
        ttgl.store(y_ptr + offs, y)
```
- **EN:** Prepares or updates state through `blocked`, `offs`, `x`, `y`. Invokes `ttgl.BlockedLayout`, `ttgl.arange`, `ttgl.load`, `libdevice.fast_expf`, `ttgl.store` to execute the test logic. Relevant themes: layout transformation reasoning, libdevice coverage.
- **CN:** 通过 `blocked`、`offs`、`x`、`y` 准备或更新状态。 调用 `ttgl.BlockedLayout`、`ttgl.arange`、`ttgl.load`、`libdevice.fast_expf`、`ttgl.store` 执行测试逻辑。 相关主题：布局变换推理、libdevice 覆盖。

#### Lines 1570-1577

```python

    num_warps = 4

    torch.manual_seed(0)
    x = torch.randn(THREADS_PER_WARP * num_warps, device="cuda", dtype=torch.float32)
    y = torch.empty_like(x)
    fast_expf_kernel[(1, )](x, y, THREADS_PER_WARP, num_warps)
    torch.testing.assert_close(y, torch.exp(x), atol=1e-5, rtol=1e-4)
```
- **EN:** Prepares or updates state through `num_warps`, `x`, `y`. Invokes `torch.manual_seed`, `torch.randn`, `torch.empty_like`, `torch.testing.assert_close`, `torch.exp` to execute the test logic. Relevant themes: PyTorch tensor setup and checks, random-data generation.
- **CN:** 通过 `num_warps`、`x`、`y` 准备或更新状态。 调用 `torch.manual_seed`、`torch.randn`、`torch.empty_like`、`torch.testing.assert_close`、`torch.exp` 执行测试逻辑。 相关主题：PyTorch 张量准备与校验、随机数据生成。

### Lines 1578-1582

```python


def test_math_fast_dividef():

    @gluon.jit
```
- **EN:** Defines the test function `test_math_fast_dividef`. Nested definitions in this scope: `fast_dividef_kernel`. Key calls include `torch.manual_seed`, `torch.randn`, `torch.randn_like`, `torch.empty_like`, `torch.testing.assert_close`, `ttgl.BlockedLayout`, and 5 more. This scope touches PyTorch tensor setup and checks, layout transformation reasoning, random-data generation, libdevice coverage.
- **CN:** 定义测试函数 `test_math_fast_dividef`。 该作用域中的嵌套定义：`fast_dividef_kernel`。 关键调用包括 `torch.manual_seed`、`torch.randn`、`torch.randn_like`、`torch.empty_like`、`torch.testing.assert_close`、`ttgl.BlockedLayout` 等另外 5 项。 该作用域涉及PyTorch 张量准备与校验、布局变换推理、随机数据生成、libdevice 覆盖。

#### Lines 1582-1583

```python
    @gluon.jit
    def fast_dividef_kernel(x_ptr, y_ptr, z_ptr, warp_size: ttgl.constexpr, num_warps: ttgl.constexpr):
```
- **EN:** Defines the helper function `fast_dividef_kernel`. Decorators: `gluon.jit`. Parameters: `x_ptr`, `y_ptr`, `z_ptr`, `warp_size`, `num_warps`. Key calls include `ttgl.BlockedLayout`, `ttgl.arange`, `ttgl.load`, `libdevice.fast_dividef`, `ttgl.store`. This scope touches layout transformation reasoning, libdevice coverage.
- **CN:** 定义辅助函数 `fast_dividef_kernel`。 装饰器：`gluon.jit`。 参数：`x_ptr`、`y_ptr`、`z_ptr`、`warp_size`、`num_warps`。 关键调用包括 `ttgl.BlockedLayout`、`ttgl.arange`、`ttgl.load`、`libdevice.fast_dividef`、`ttgl.store`。 该作用域涉及布局变换推理、libdevice 覆盖。

##### Lines 1584-1590

```python
        blocked: ttgl.constexpr = ttgl.BlockedLayout([1], [warp_size], [num_warps], [0])

        offs = ttgl.arange(0, warp_size * num_warps, layout=blocked)
        x = ttgl.load(x_ptr + offs)
        y = ttgl.load(y_ptr + offs)
        z = libdevice.fast_dividef(x, y)
        ttgl.store(z_ptr + offs, z)
```
- **EN:** Prepares or updates state through `blocked`, `offs`, `x`, `y`, `z`. Invokes `ttgl.BlockedLayout`, `ttgl.arange`, `ttgl.load`, `libdevice.fast_dividef`, `ttgl.store` to execute the test logic. Relevant themes: layout transformation reasoning, libdevice coverage.
- **CN:** 通过 `blocked`、`offs`、`x`、`y`、`z` 准备或更新状态。 调用 `ttgl.BlockedLayout`、`ttgl.arange`、`ttgl.load`、`libdevice.fast_dividef`、`ttgl.store` 执行测试逻辑。 相关主题：布局变换推理、libdevice 覆盖。

#### Lines 1591-1600

```python

    num_warps = 4

    torch.manual_seed(0)
    x = torch.randn(THREADS_PER_WARP * num_warps, device="cuda", dtype=torch.float32)
    y = torch.randn_like(x)
    z = torch.empty_like(x)
    y[y == 0] = 1.0
    fast_dividef_kernel[(1, )](x, y, z, THREADS_PER_WARP, num_warps)
    torch.testing.assert_close(z, torch.div(x, y), atol=1e-5, rtol=1e-4)
```
- **EN:** Prepares or updates state through `num_warps`, `x`, `y`, `z`. Invokes `torch.manual_seed`, `torch.randn`, `torch.randn_like`, `torch.empty_like`, `torch.testing.assert_close`, `torch.div` to execute the test logic. Relevant themes: PyTorch tensor setup and checks, random-data generation.
- **CN:** 通过 `num_warps`、`x`、`y`、`z` 准备或更新状态。 调用 `torch.manual_seed`、`torch.randn`、`torch.randn_like`、`torch.empty_like`、`torch.testing.assert_close`、`torch.div` 执行测试逻辑。 相关主题：PyTorch 张量准备与校验、随机数据生成。

### Lines 1601-1604

```python


@pytest.mark.skipif(not is_blackwell(), reason="Requires Blackwell")
def test_tmem_copy_2d():
```
- **EN:** Defines the test function `test_tmem_copy_2d`. Decorators: `pytest.mark.skipif(not is_blackwell(), reason='Requires Blackwell')`. Nested definitions in this scope: `kernel`. Key calls include `pytest.mark.skipif`, `torch.manual_seed`, `x.reshape`, `x_res.permute`, `x_res.reshape`, `torch.chunk`, and 23 more. This scope touches PyTorch tensor setup and checks, layout transformation reasoning, random-data generation.
- **CN:** 定义测试函数 `test_tmem_copy_2d`。 装饰器：`pytest.mark.skipif(not is_blackwell(), reason='Requires Blackwell')`。 该作用域中的嵌套定义：`kernel`。 关键调用包括 `pytest.mark.skipif`、`torch.manual_seed`、`x.reshape`、`x_res.permute`、`x_res.reshape`、`torch.chunk` 等另外 23 项。 该作用域涉及PyTorch 张量准备与校验、布局变换推理、随机数据生成。

#### Lines 1605-1610

```python
    device = "cuda"

    smem_h = 64
    smem_w = 16
    num_rows = 128
    num_cols = smem_h * smem_w // 32
```
- **EN:** Prepares or updates state through `device`, `smem_h`, `smem_w`, `num_rows`, `num_cols`.
- **CN:** 通过 `device`、`smem_h`、`smem_w`、`num_rows`、`num_cols` 准备或更新状态。

#### Lines 1611-1614

```python

    @gluon.jit
    def kernel(in_ptr, out_ptr, smem_h: ttgl.constexpr, smem_w: ttgl.constexpr, num_rows: ttgl.constexpr,
               num_cols: ttgl.constexpr):
```
- **EN:** Defines the helper function `kernel`. Decorators: `gluon.jit`. Parameters: `in_ptr`, `out_ptr`, `smem_h`, `smem_w`, `num_rows`, `num_cols`. Key calls include `ttgl.BlockedLayout`, `ttgl.load`, `ttgl.SharedLinearLayout`, `TensorMemoryScalesLayout`, `ttgl.allocate_shared_memory`, `allocate_tensor_memory`, and 13 more. This scope touches layout transformation reasoning.
- **CN:** 定义辅助函数 `kernel`。 装饰器：`gluon.jit`。 参数：`in_ptr`、`out_ptr`、`smem_h`、`smem_w`、`num_rows`、`num_cols`。 关键调用包括 `ttgl.BlockedLayout`、`ttgl.load`、`ttgl.SharedLinearLayout`、`TensorMemoryScalesLayout`、`ttgl.allocate_shared_memory`、`allocate_tensor_memory` 等另外 13 项。 该作用域涉及布局变换推理。

##### Lines 1615-1637

```python
        in_ptrs = in_ptr + ttgl.arange(0, smem_h)[:, None] * smem_w + ttgl.arange(0, smem_w)[None, :]
        out_ptrs = out_ptr + ttgl.arange(0, num_rows)[:, None] * num_cols + ttgl.arange(0, num_cols)[None, :]

        blocked: ttgl.constexpr = ttgl.BlockedLayout([1, 4], [32, 1], [4, 1], [1, 0])
        value = ttgl.load(ttgl.set_auto_layout(in_ptrs, blocked))

        smem_layout: ttgl.constexpr = ttgl.SharedLinearLayout(
            offset_bases=[[0, 1], [0, 2], [32, 0], [0, 4], [1, 0], [2, 0], [4, 0], [8, 0], [16, 0], [0, 8]])
        tmem_layout: ttgl.constexpr = TensorMemoryScalesLayout()
        smem = ttgl.allocate_shared_memory(ttgl.int8, (smem_h, smem_w), layout=smem_layout)
        tmem = allocate_tensor_memory(ttgl.int8, (smem_h, smem_w), layout=tmem_layout)

        barrier = ttgl.allocate_shared_memory(ttgl.int64, [1], ttgl.constexpr(mbarrier.MBarrierLayout()))
        mbarrier.init(barrier, count=1)

        smem.store(value)
        tcgen05_copy(smem, tmem)
        tcgen05_commit(barrier)
        mbarrier.wait(barrier, phase=0)
        tmem_alias: ttgl.constexpr = TensorMemoryLayout((num_rows, num_cols), col_stride=1)
        tmem = tmem._reinterpret(shape=(num_rows, num_cols), layout=tmem_alias)
        value = tmem.load(blocked)
        ttgl.store(ttgl.set_auto_layout(out_ptrs, blocked), value)
```
- **EN:** Prepares or updates state through `in_ptrs`, `out_ptrs`, `blocked`, `value`, `smem_layout`, `tmem_layout`, `smem`, `tmem`, and 2 more. Invokes `ttgl.arange`, `ttgl.BlockedLayout`, `ttgl.load`, `ttgl.set_auto_layout`, `ttgl.SharedLinearLayout`, `TensorMemoryScalesLayout`, and 13 more to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `in_ptrs`、`out_ptrs`、`blocked`、`value`、`smem_layout`、`tmem_layout`、`smem`、`tmem` 等另外 2 项 准备或更新状态。 调用 `ttgl.arange`、`ttgl.BlockedLayout`、`ttgl.load`、`ttgl.set_auto_layout`、`ttgl.SharedLinearLayout`、`TensorMemoryScalesLayout` 等另外 13 项 执行测试逻辑。 相关主题：布局变换推理。

#### Lines 1638-1653

```python

    torch.manual_seed(0)
    x = torch.randint(size=(smem_h, smem_w), low=-100, high=100, dtype=torch.int8).to(device)
    #x = torch.arange(smem_h * smem_w, dtype=torch.int8, device=device).reshape(smem_h, smem_w)
    z_tri = torch.zeros(size=(num_rows, num_cols), dtype=torch.int8).to(device)
    kernel[(1, )](x, z_tri, smem_h, smem_w, num_rows, num_cols)

    # offset_bases=[[0, 1], [0, 2], [32, 0], [0, 4], [1, 0], [2, 0], [4, 0], [8, 0], [16, 0], [0, 8]],
    # Split into contiguous shmem chunks
    x_res = x.reshape(2, 32, 2, 2, 4)
    # Put tmem cols first then rows
    x_res = x_res.permute(1, 2, 3, 0, 4)
    # Reshape as 32xnum_cols
    x_res = x_res.reshape(num_rows // 4, num_cols)

    warps = torch.chunk(z_tri, chunks=4, dim=0)
```
- **EN:** Prepares or updates state through `x`, `z_tri`, `x_res`, `warps`. Invokes `torch.manual_seed`, `torch.randint`, `torch.zeros`, `x.reshape`, `x_res.permute`, `x_res.reshape`, and 1 more to execute the test logic. Relevant themes: PyTorch tensor setup and checks, random-data generation.
- **CN:** 通过 `x`、`z_tri`、`x_res`、`warps` 准备或更新状态。 调用 `torch.manual_seed`、`torch.randint`、`torch.zeros`、`x.reshape`、`x_res.permute`、`x_res.reshape` 等另外 1 项 执行测试逻辑。 相关主题：PyTorch 张量准备与校验、随机数据生成。

#### Lines 1654-1655

```python
    for warp in warps:
        torch.testing.assert_close(x_res, warp)
```
- **EN:** Invokes `torch.testing.assert_close` to execute the test logic. Iterates across cases or data tiles. Relevant themes: PyTorch tensor setup and checks.
- **CN:** 调用 `torch.testing.assert_close` 执行测试逻辑。 通过循环覆盖多个用例或数据分块。 相关主题：PyTorch 张量准备与校验。

### Lines 1656-1661

```python


@pytest.mark.skipif(not is_blackwell(), reason="Requires Blackwell")
def test_tmem_subslice_block_m_64():

    @gluon.jit
```
- **EN:** Defines the test function `test_tmem_subslice_block_m_64`. Decorators: `pytest.mark.skipif(not is_blackwell(), reason='Requires Blackwell')`. Nested definitions in this scope: `kernel`. Key calls include `pytest.mark.skipif`, `torch.manual_seed`, `torch.randn`, `torch.empty_like`, `llir.count`, `torch.testing.assert_close`, and 21 more. This scope touches PyTorch tensor setup and checks, layout transformation reasoning, random-data generation.
- **CN:** 定义测试函数 `test_tmem_subslice_block_m_64`。 装饰器：`pytest.mark.skipif(not is_blackwell(), reason='Requires Blackwell')`。 该作用域中的嵌套定义：`kernel`。 关键调用包括 `pytest.mark.skipif`、`torch.manual_seed`、`torch.randn`、`torch.empty_like`、`llir.count`、`torch.testing.assert_close` 等另外 21 项。 该作用域涉及PyTorch 张量准备与校验、布局变换推理、随机数据生成。

#### Lines 1661-1662

```python
    @gluon.jit
    def kernel(s_ptr, out_ptr):
```
- **EN:** Defines the helper function `kernel`. Decorators: `gluon.jit`. Parameters: `s_ptr`, `out_ptr`. Key calls include `TensorMemoryLayout`, `allocate_tensor_memory`, `s_tmem.get_reg_layout`, `ttgl.set_auto_layout`, `ttgl.load`, `s_tmem.store`, and 13 more. This scope touches layout transformation reasoning.
- **CN:** 定义辅助函数 `kernel`。 装饰器：`gluon.jit`。 参数：`s_ptr`、`out_ptr`。 关键调用包括 `TensorMemoryLayout`、`allocate_tensor_memory`、`s_tmem.get_reg_layout`、`ttgl.set_auto_layout`、`ttgl.load`、`s_tmem.store` 等另外 13 项。 该作用域涉及布局变换推理。

##### Lines 1663-1697

```python
        BLOCK_M: ttgl.constexpr = 64
        N: ttgl.constexpr = 128
        BLOCK_N: ttgl.constexpr = 64

        tmem_layout: ttgl.constexpr = TensorMemoryLayout((BLOCK_M, BLOCK_N), col_stride=1)
        s_tmem = allocate_tensor_memory(ttgl.float32, (BLOCK_M, N), layout=tmem_layout)
        o_tmem = allocate_tensor_memory(ttgl.float32, (BLOCK_M, N), layout=tmem_layout)

        layout: ttgl.constexpr = s_tmem.get_reg_layout()

        offsets = ttgl.arange(0, BLOCK_M)[:, None] * N + ttgl.arange(0, N)[None, :]
        offsets = ttgl.set_auto_layout(offsets, layout)
        s = ttgl.load(s_ptr + offsets)

        s_tmem.store(s)
        o_tmem.store(s)

        p_tmem_parent = s_tmem._reinterpret(ttgl.float16, [BLOCK_M, 2 * N], tmem_layout)
        p_tmem = p_tmem_parent.slice(0, N)
        p_tmem.store(ttgl.full((BLOCK_M, N), 0.0, dtype=ttgl.float16, layout=layout))

        d1_tmem_layout: ttgl.constexpr = TensorMemoryLayout((BLOCK_M, 2), col_stride=1)

        d1_tmem_parent = s_tmem._reinterpret(layout=d1_tmem_layout)
        m_tmem = d1_tmem_parent.slice(N // 2, 2)
        d1_layout: ttgl.constexpr = m_tmem.get_reg_layout()
        m_tmem.store(ttgl.full((BLOCK_M, 2), 2.0, dtype=ttgl.float32, layout=d1_layout))
        l_tmem = d1_tmem_parent.slice(N // 2 + 4, 2)
        l_tmem.store(ttgl.full((BLOCK_M, 2), 3.0, dtype=ttgl.float32, layout=d1_layout))
        a_tmem = d1_tmem_parent.slice(N // 2 + 8, 2)
        a_tmem.store(ttgl.full((BLOCK_M, 2), 4.0, dtype=ttgl.float32, layout=d1_layout))

        s = s_tmem.load()

        ttgl.store(out_ptr + offsets, s)
```
- **EN:** Prepares or updates state through `BLOCK_M`, `N`, `BLOCK_N`, `tmem_layout`, `s_tmem`, `o_tmem`, `layout`, `offsets`, and 9 more. Invokes `TensorMemoryLayout`, `allocate_tensor_memory`, `s_tmem.get_reg_layout`, `ttgl.arange`, `ttgl.set_auto_layout`, `ttgl.load`, and 13 more to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `BLOCK_M`、`N`、`BLOCK_N`、`tmem_layout`、`s_tmem`、`o_tmem`、`layout`、`offsets` 等另外 9 项 准备或更新状态。 调用 `TensorMemoryLayout`、`allocate_tensor_memory`、`s_tmem.get_reg_layout`、`ttgl.arange`、`ttgl.set_auto_layout`、`ttgl.load` 等另外 13 项 执行测试逻辑。 相关主题：布局变换推理。

#### Lines 1698-1705

```python

    torch.manual_seed(0)
    s = torch.randn((64, 128), dtype=torch.float32, device="cuda")

    out_tri = torch.empty_like(s)
    compiled = kernel[(1, )](s, out_tri)

    ttgir = compiled.asm["ttgir"]
```
- **EN:** Prepares or updates state through `s`, `out_tri`, `compiled`, `ttgir`. Invokes `torch.manual_seed`, `torch.randn`, `torch.empty_like` to execute the test logic. Relevant themes: PyTorch tensor setup and checks, random-data generation.
- **CN:** 通过 `s`、`out_tri`、`compiled`、`ttgir` 准备或更新状态。 调用 `torch.manual_seed`、`torch.randn`、`torch.empty_like` 执行测试逻辑。 相关主题：PyTorch 张量准备与校验、随机数据生成。

#### Lines 1706-1708

```python
    # Check that we have two 64x128xf32 allocations.
    assert ttgir.count("ttng.tmem_alloc") == 2
    assert ttgir.count("ttng.tmem_alloc : () -> !ttg.memdesc<64x128xf32") == 2
```
- **EN:** Invokes `ttgir.count` to execute the test logic. Validates behavior with 2 assertion(s).
- **CN:** 调用 `ttgir.count` 执行测试逻辑。 通过 2 个断言验证行为。

#### Lines 1709-1711

```python

    # Check that we allocated only 128 columns of TMEM.
    llir = compiled.asm["llir"]
```
- **EN:** Prepares or updates state through `llir`.
- **CN:** 通过 `llir` 准备或更新状态。

#### Lines 1712-1712

```python
    assert llir.count("tcgen05.alloc.cta_group::1.sync.aligned.shared::cta.b32 [$1], 128")
```
- **EN:** Invokes `llir.count` to execute the test logic. Validates behavior with 1 assertion(s).
- **CN:** 调用 `llir.count` 执行测试逻辑。 通过 1 个断言验证行为。

#### Lines 1713-1736

```python

    # Given TMEM[0:32] is the slice of TMEM for warpgroup 0, the expected layout
    # of S is
    #
    #   TMEM[0:16]  = S[0:16, 0:64]
    #   TMEM[16:32] = S[0:16, 64:128]
    #
    # When slicing S to obtain P, we expect it to overlap with the left half,
    # i.e. S[0:16, 0:32] and S[0:16, 64:96].
    out_ref = s
    out_ref[:, 0:32] = 0.0
    out_ref[:, 64:96] = 0.0

    # Given S = [s0, s1, s2, s3], they are arranged like
    #
    #   TMEM[0:16]  = [s0, s1]
    #   TMEM[16:32] = [s2, s3]
    #
    # Thus the narrow parent view is sliced at offsets that map back to s1.
    out_ref[:, 32:34] = 2.0
    out_ref[:, 34:36] = 3.0
    out_ref[:, 36:38] = 4.0

    torch.testing.assert_close(out_ref, out_tri, atol=0, rtol=0)
```
- **EN:** Prepares or updates state through `out_ref`. Invokes `torch.testing.assert_close` to execute the test logic. Relevant themes: PyTorch tensor setup and checks, layout transformation reasoning.
- **CN:** 通过 `out_ref` 准备或更新状态。 调用 `torch.testing.assert_close` 执行测试逻辑。 相关主题：PyTorch 张量准备与校验、布局变换推理。

### Lines 1737-1742

```python


@pytest.mark.skipif(not is_blackwell(), reason="Requires Blackwell")
def test_block_m_64_mma():

    @gluon.jit
```
- **EN:** Defines the test function `test_block_m_64_mma`. Decorators: `pytest.mark.skipif(not is_blackwell(), reason='Requires Blackwell')`. Nested definitions in this scope: `kernel`. Key calls include `pytest.mark.skipif`, `torch.manual_seed`, `torch.randn`, `torch.empty_like`, `llir.count`, `torch.testing.assert_close`, and 31 more. This scope touches PyTorch tensor setup and checks, layout transformation reasoning, random-data generation.
- **CN:** 定义测试函数 `test_block_m_64_mma`。 装饰器：`pytest.mark.skipif(not is_blackwell(), reason='Requires Blackwell')`。 该作用域中的嵌套定义：`kernel`。 关键调用包括 `pytest.mark.skipif`、`torch.manual_seed`、`torch.randn`、`torch.empty_like`、`llir.count`、`torch.testing.assert_close` 等另外 31 项。 该作用域涉及PyTorch 张量准备与校验、布局变换推理、随机数据生成。

#### Lines 1742-1743

```python
    @gluon.jit
    def kernel(a_ptr, b_ptr, c_ptr, d_ptr):
```
- **EN:** Defines the helper function `kernel`. Decorators: `gluon.jit`. Parameters: `a_ptr`, `b_ptr`, `c_ptr`, `d_ptr`. Key calls include `TensorMemoryLayout`, `allocate_tensor_memory`, `al_tmem.get_reg_layout`, `ttgl.BlockedLayout`, `ttgl.set_auto_layout`, `ttgl.load`, and 23 more. This scope touches layout transformation reasoning.
- **CN:** 定义辅助函数 `kernel`。 装饰器：`gluon.jit`。 参数：`a_ptr`、`b_ptr`、`c_ptr`、`d_ptr`。 关键调用包括 `TensorMemoryLayout`、`allocate_tensor_memory`、`al_tmem.get_reg_layout`、`ttgl.BlockedLayout`、`ttgl.set_auto_layout`、`ttgl.load` 等另外 23 项。 该作用域涉及布局变换推理。

##### Lines 1744-1807

```python
        BLOCK_M: ttgl.constexpr = 64
        N: ttgl.constexpr = 128
        BLOCK_N: ttgl.constexpr = 64

        a_offsets = ttgl.arange(0, BLOCK_M)[:, None] * N + ttgl.arange(0, N)[None, :]
        b_offsets = ttgl.arange(0, N)[:, None] * N + ttgl.arange(0, N)[None, :]

        a_tmem_layout: ttgl.constexpr = TensorMemoryLayout((BLOCK_M, BLOCK_N), col_stride=1)
        acc_tmem_layout: ttgl.constexpr = TensorMemoryLayout((BLOCK_M, BLOCK_N), col_stride=1)
        al_tmem = allocate_tensor_memory(ttgl.float16, (BLOCK_M, N), layout=a_tmem_layout)
        ar_tmem = allocate_tensor_memory(ttgl.float16, (BLOCK_M, N), layout=a_tmem_layout)
        acc_tmem = allocate_tensor_memory(ttgl.float32, (BLOCK_M, N), layout=acc_tmem_layout)
        a_layout: ttgl.constexpr = al_tmem.get_reg_layout(instr_variant="32x32b_splitn")
        b_layout: ttgl.constexpr = ttgl.BlockedLayout([1, 1], [1, 32], [4, 1], [1, 0])
        a_offsets = ttgl.set_auto_layout(a_offsets, a_layout)
        b_offsets = ttgl.set_auto_layout(b_offsets, b_layout)

        a = ttgl.load(a_ptr + a_offsets)
        b = ttgl.load(b_ptr + b_offsets)
        c = ttgl.load(c_ptr + a_offsets)

        a0, a1 = a.reshape((BLOCK_M, 2, N // 2)).permute(0, 2, 1).split()

        al = ttgl.join(a0, a1).permute(0, 2, 1).reshape((BLOCK_M, N))
        ar = ttgl.join(a1, a0).permute(0, 2, 1).reshape((BLOCK_M, N))

        al_tmem.store(ttgl.convert_layout(al, a_layout, assert_trivial=True))
        ar_tmem.store(ttgl.convert_layout(ar, a_layout, assert_trivial=True))

        b_shared_layout: ttgl.constexpr = ttgl.NVMMASharedLayout(swizzle_byte_width=32, element_bitwidth=16, rank=2)
        b_shared = ttgl.allocate_shared_memory(ttgl.float16, [N, N], layout=b_shared_layout)
        b_shared.store(b)

        acc_tmem.store(c)

        bar = ttgl.allocate_shared_memory(ttgl.int64, [1], ttgl.constexpr(mbarrier.MBarrierLayout()))
        mbarrier.init(bar, count=1)

        # This is a manually tiled MMA where LHS is in TMEM with blockM=64,
        # where we circumvent the limitation that LHS and accumulator need to
        # share the same TMEM rows by storing the LHS twice.
        #
        # TMEM      al   ar   c
        # [0, 16)   a0   a1   c0
        # [16, 32)  a1   a0   c1
        #
        # d0 = a0 @ b00 + a1 @ b10 + c0
        # d1 = a0 @ b10 + a1 @ b11 + c1

        N2: ttgl.constexpr = N // 2
        c0 = acc_tmem.slice(0, N2)
        c1 = acc_tmem.slice(N2, N2)

        tcgen05_mma(al_tmem.slice(0, N2), b_shared.slice(0, N2, dim=0).slice(0, N2, dim=1), c0)
        tcgen05_mma(ar_tmem.slice(0, N2), b_shared.slice(N2, N2, dim=0).slice(0, N2, dim=1), c0)
        tcgen05_mma(ar_tmem.slice(N2, N2), b_shared.slice(0, N2, dim=0).slice(N2, N2, dim=1), c1)
        tcgen05_mma(al_tmem.slice(N2, N2), b_shared.slice(N2, N2, dim=0).slice(N2, N2, dim=1), c1)

        tcgen05_commit(bar)
        mbarrier.wait(bar, 0)
        mbarrier.invalidate(bar)

        d = acc_tmem.load(a_layout)
        ttgl.store(d_ptr + a_offsets, d)
```
- **EN:** Prepares or updates state through `BLOCK_M`, `N`, `BLOCK_N`, `a_offsets`, `b_offsets`, `a_tmem_layout`, `acc_tmem_layout`, `al_tmem`, and 18 more. Invokes `ttgl.arange`, `TensorMemoryLayout`, `allocate_tensor_memory`, `al_tmem.get_reg_layout`, `ttgl.BlockedLayout`, `ttgl.set_auto_layout`, and 23 more to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `BLOCK_M`、`N`、`BLOCK_N`、`a_offsets`、`b_offsets`、`a_tmem_layout`、`acc_tmem_layout`、`al_tmem` 等另外 18 项 准备或更新状态。 调用 `ttgl.arange`、`TensorMemoryLayout`、`allocate_tensor_memory`、`al_tmem.get_reg_layout`、`ttgl.BlockedLayout`、`ttgl.set_auto_layout` 等另外 23 项 执行测试逻辑。 相关主题：布局变换推理。

#### Lines 1808-1817

```python

    torch.manual_seed(0)
    a = torch.randn((64, 128), dtype=torch.float16, device="cuda")
    b = torch.randn((128, 128), dtype=torch.float16, device="cuda")
    c = torch.randn((64, 128), dtype=torch.float32, device="cuda")

    d_tri = torch.empty_like(c)
    compiled = kernel[(1, )](a, b, c, d_tri)

    ttgir = compiled.asm["ttgir"]
```
- **EN:** Prepares or updates state through `a`, `b`, `c`, `d_tri`, `compiled`, `ttgir`. Invokes `torch.manual_seed`, `torch.randn`, `torch.empty_like` to execute the test logic. Relevant themes: PyTorch tensor setup and checks, random-data generation.
- **CN:** 通过 `a`、`b`、`c`、`d_tri`、`compiled`、`ttgir` 准备或更新状态。 调用 `torch.manual_seed`、`torch.randn`、`torch.empty_like` 执行测试逻辑。 相关主题：PyTorch 张量准备与校验、随机数据生成。

#### Lines 1818-1820

```python
    assert ttgir.count("ttng.tmem_alloc") == 3
    assert ttgir.count("ttng.tmem_alloc : () -> !ttg.memdesc<64x128xf32") == 1
    assert ttgir.count("ttng.tmem_alloc : () -> !ttg.memdesc<64x128xf16") == 2
```
- **EN:** Invokes `ttgir.count` to execute the test logic. Validates behavior with 3 assertion(s).
- **CN:** 调用 `ttgir.count` 执行测试逻辑。 通过 3 个断言验证行为。

#### Lines 1821-1822

```python

    llir = compiled.asm["llir"]
```
- **EN:** Prepares or updates state through `llir`.
- **CN:** 通过 `llir` 准备或更新状态。

#### Lines 1823-1823

```python
    assert llir.count("tcgen05.alloc.cta_group::1.sync.aligned.shared::cta.b32 [$1], 128")
```
- **EN:** Invokes `llir.count` to execute the test logic. Validates behavior with 1 assertion(s).
- **CN:** 调用 `llir.count` 执行测试逻辑。 通过 1 个断言验证行为。

#### Lines 1824-1826

```python

    d_ref = a @ b + c
    torch.testing.assert_close(d_ref, d_tri, rtol=0.08, atol=0)
```
- **EN:** Prepares or updates state through `d_ref`. Invokes `torch.testing.assert_close` to execute the test logic. Relevant themes: PyTorch tensor setup and checks.
- **CN:** 通过 `d_ref` 准备或更新状态。 调用 `torch.testing.assert_close` 执行测试逻辑。 相关主题：PyTorch 张量准备与校验。

### Lines 1827-1829

```python


def test_slice_reinterpret():
```
- **EN:** Defines the test function `test_slice_reinterpret`. Nested definitions in this scope: `kernel`. Key calls include `ttgl.constexpr`, `torch.randint`, `torch.empty_like`, `torch.testing.assert_close`, `ttgl.SwizzledSharedLayout`, `ttgl.allocate_shared_memory`, and 12 more. This scope touches PyTorch tensor setup and checks, layout transformation reasoning, random-data generation.
- **CN:** 定义测试函数 `test_slice_reinterpret`。 该作用域中的嵌套定义：`kernel`。 关键调用包括 `ttgl.constexpr`、`torch.randint`、`torch.empty_like`、`torch.testing.assert_close`、`ttgl.SwizzledSharedLayout`、`ttgl.allocate_shared_memory` 等另外 12 项。 该作用域涉及PyTorch 张量准备与校验、布局变换推理、随机数据生成。

#### Lines 1830-1834

```python
    BLOCK = ttgl.constexpr(2048)
    SPLIT_BLOCK = ttgl.constexpr(BLOCK // 2)
    XBLOCK = ttgl.constexpr(32)
    YBLOCK = ttgl.constexpr(SPLIT_BLOCK // 4 // XBLOCK)
    NUM_THREADS = ttgl.constexpr(THREADS_PER_WARP)
```
- **EN:** Prepares or updates state through `BLOCK`, `SPLIT_BLOCK`, `XBLOCK`, `YBLOCK`, `NUM_THREADS`. Invokes `ttgl.constexpr` to execute the test logic.
- **CN:** 通过 `BLOCK`、`SPLIT_BLOCK`、`XBLOCK`、`YBLOCK`、`NUM_THREADS` 准备或更新状态。 调用 `ttgl.constexpr` 执行测试逻辑。

#### Lines 1835-1837

```python

    @gluon.jit
    def kernel(in_ptr, out_ptr):
```
- **EN:** Defines the helper function `kernel`. Decorators: `gluon.jit`. Parameters: `in_ptr`, `out_ptr`. Key calls include `ttgl.SwizzledSharedLayout`, `ttgl.allocate_shared_memory`, `smem.slice`, `smem._reinterpret`, `smem_i32.slice`, `ttgl.BlockedLayout`, and 8 more. This scope touches layout transformation reasoning.
- **CN:** 定义辅助函数 `kernel`。 装饰器：`gluon.jit`。 参数：`in_ptr`、`out_ptr`。 关键调用包括 `ttgl.SwizzledSharedLayout`、`ttgl.allocate_shared_memory`、`smem.slice`、`smem._reinterpret`、`smem_i32.slice`、`ttgl.BlockedLayout` 等另外 8 项。 该作用域涉及布局变换推理。

##### Lines 1838-1853

```python
        smem_layout_1d: ttgl.constexpr = ttgl.SwizzledSharedLayout(vec=1, per_phase=1, max_phase=1, order=[0])
        smem_layout_2d: ttgl.constexpr = ttgl.SwizzledSharedLayout(vec=1, per_phase=1, max_phase=1, order=[1, 0])
        smem = ttgl.allocate_shared_memory(ttgl.int8, [BLOCK], smem_layout_1d)
        smem_slice0 = smem.slice(0, SPLIT_BLOCK)
        smem_i32 = smem._reinterpret(ttgl.int32, [2 * XBLOCK, YBLOCK], smem_layout_2d)
        smem_slice1 = smem_i32.slice(XBLOCK, XBLOCK, dim=0)

        offs = ttgl.arange(0, XBLOCK)[:, None] * YBLOCK + ttgl.arange(0, YBLOCK)[None, :]
        blocked: ttgl.constexpr = ttgl.BlockedLayout([1, 1], [1, NUM_THREADS], [1, 4], [1, 0])
        value = ttgl.load(ttgl.set_auto_layout(in_ptr + offs, blocked))

        blocked_1d: ttgl.constexpr = ttgl.BlockedLayout([1], [NUM_THREADS], [4], [0])
        smem_slice1.store(value)
        smem_slice0.store(ttgl.zeros((SPLIT_BLOCK, ), dtype=ttgl.int8, layout=blocked_1d))
        value = smem_slice1.load(blocked)
        ttgl.store(ttgl.set_auto_layout(out_ptr + offs, blocked), value)
```
- **EN:** Prepares or updates state through `smem_layout_1d`, `smem_layout_2d`, `smem`, `smem_slice0`, `smem_i32`, `smem_slice1`, `offs`, `blocked`, and 2 more. Invokes `ttgl.SwizzledSharedLayout`, `ttgl.allocate_shared_memory`, `smem.slice`, `smem._reinterpret`, `smem_i32.slice`, `ttgl.arange`, and 8 more to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `smem_layout_1d`、`smem_layout_2d`、`smem`、`smem_slice0`、`smem_i32`、`smem_slice1`、`offs`、`blocked` 等另外 2 项 准备或更新状态。 调用 `ttgl.SwizzledSharedLayout`、`ttgl.allocate_shared_memory`、`smem.slice`、`smem._reinterpret`、`smem_i32.slice`、`ttgl.arange` 等另外 8 项 执行测试逻辑。 相关主题：布局变换推理。

#### Lines 1854-1858

```python

    input = torch.randint(0, 100, (XBLOCK, YBLOCK), dtype=torch.int32, device="cuda")
    output = torch.empty_like(input)
    kernel[(1, )](input, output)
    torch.testing.assert_close(input, output, atol=0, rtol=0)
```
- **EN:** Prepares or updates state through `input`, `output`. Invokes `torch.randint`, `torch.empty_like`, `torch.testing.assert_close` to execute the test logic. Relevant themes: PyTorch tensor setup and checks, random-data generation.
- **CN:** 通过 `input`、`output` 准备或更新状态。 调用 `torch.randint`、`torch.empty_like`、`torch.testing.assert_close` 执行测试逻辑。 相关主题：PyTorch 张量准备与校验、随机数据生成。

### Lines 1859-1862

```python


@pytest.mark.skipif(not is_hopper_or_newer(), reason="Requires Hopper")
def test_tma_slice():
```
- **EN:** Defines the test function `test_tma_slice`. Decorators: `pytest.mark.skipif(not is_hopper_or_newer(), reason='Requires Hopper')`. Nested definitions in this scope: `kernel`. Key calls include `pytest.mark.skipif`, `ttgl.constexpr`, `torch.rand`, `torch.empty_like`, `ttgl.NVMMASharedLayout.get_default_for`, `gluon.nvidia.hopper.TensorDescriptor.from_tensor`, and 14 more. This scope touches PyTorch tensor setup and checks, tensor/descriptor metadata, layout transformation reasoning, random-data generation.
- **CN:** 定义测试函数 `test_tma_slice`。 装饰器：`pytest.mark.skipif(not is_hopper_or_newer(), reason='Requires Hopper')`。 该作用域中的嵌套定义：`kernel`。 关键调用包括 `pytest.mark.skipif`、`ttgl.constexpr`、`torch.rand`、`torch.empty_like`、`ttgl.NVMMASharedLayout.get_default_for`、`gluon.nvidia.hopper.TensorDescriptor.from_tensor` 等另外 14 项。 该作用域涉及PyTorch 张量准备与校验、张量/描述符元数据、布局变换推理、随机数据生成。

#### Lines 1863-1863

```python
    XBLOCK = YBLOCK = ttgl.constexpr(128)
```
- **EN:** Prepares or updates state through `XBLOCK`, `YBLOCK`. Invokes `ttgl.constexpr` to execute the test logic.
- **CN:** 通过 `XBLOCK`、`YBLOCK` 准备或更新状态。 调用 `ttgl.constexpr` 执行测试逻辑。

#### Lines 1864-1866

```python

    @gluon.jit
    def kernel(in_desc, out_desc):
```
- **EN:** Defines the helper function `kernel`. Decorators: `gluon.jit`. Parameters: `in_desc`, `out_desc`. Key calls include `ttgl.allocate_shared_memory`, `smem.slice`, `mbarrier.init`, `mbarrier.expect`, `tma.async_load`, `mbarrier.wait`, and 7 more. This scope touches layout transformation reasoning.
- **CN:** 定义辅助函数 `kernel`。 装饰器：`gluon.jit`。 参数：`in_desc`、`out_desc`。 关键调用包括 `ttgl.allocate_shared_memory`、`smem.slice`、`mbarrier.init`、`mbarrier.expect`、`tma.async_load`、`mbarrier.wait` 等另外 7 项。 该作用域涉及布局变换推理。

##### Lines 1867-1882

```python
        smem = ttgl.allocate_shared_memory(in_desc.dtype, [2 * XBLOCK, YBLOCK], in_desc.layout)
        smem_slice0 = smem.slice(0, XBLOCK)
        smem_slice1 = smem.slice(XBLOCK, XBLOCK)

        bar = ttgl.allocate_shared_memory(ttgl.int64, [1], ttgl.constexpr(mbarrier.MBarrierLayout()))
        mbarrier.init(bar, count=1)

        mbarrier.expect(bar, in_desc.nbytes_per_cta)
        tma.async_load(in_desc, [0, 0], bar, smem_slice1)
        mbarrier.wait(bar, phase=0)

        blocked: ttgl.constexpr = ttgl.BlockedLayout([1, 1], [1, 32], [1, 4], [1, 0])
        smem_slice0.store(ttgl.zeros((XBLOCK, YBLOCK), dtype=ttgl.float32, layout=blocked))

        tma.async_copy_shared_to_global(out_desc, [0, 0], smem_slice1)
        tma.store_wait(0)
```
- **EN:** Prepares or updates state through `smem`, `smem_slice0`, `smem_slice1`, `bar`, `blocked`. Invokes `ttgl.allocate_shared_memory`, `smem.slice`, `ttgl.constexpr`, `mbarrier.MBarrierLayout`, `mbarrier.init`, `mbarrier.expect`, and 7 more to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `smem`、`smem_slice0`、`smem_slice1`、`bar`、`blocked` 准备或更新状态。 调用 `ttgl.allocate_shared_memory`、`smem.slice`、`ttgl.constexpr`、`mbarrier.MBarrierLayout`、`mbarrier.init`、`mbarrier.expect` 等另外 7 项 执行测试逻辑。 相关主题：布局变换推理。

#### Lines 1883-1892

```python

    input = torch.rand((XBLOCK, YBLOCK), dtype=torch.float32, device="cuda")
    output = torch.empty_like(input)

    block_shape = [XBLOCK.value, YBLOCK.value]
    layout = ttgl.NVMMASharedLayout.get_default_for(block_shape, ttgl.float32)
    in_desc = gluon.nvidia.hopper.TensorDescriptor.from_tensor(input, block_shape, layout)
    out_desc = gluon.nvidia.hopper.TensorDescriptor.from_tensor(output, block_shape, layout)
    kernel[(1, )](in_desc, out_desc)
    torch.testing.assert_close(input, output, atol=0, rtol=0)
```
- **EN:** Prepares or updates state through `input`, `output`, `block_shape`, `layout`, `in_desc`, `out_desc`. Invokes `torch.rand`, `torch.empty_like`, `ttgl.NVMMASharedLayout.get_default_for`, `gluon.nvidia.hopper.TensorDescriptor.from_tensor`, `torch.testing.assert_close` to execute the test logic. Relevant themes: PyTorch tensor setup and checks, tensor/descriptor metadata, layout transformation reasoning, random-data generation.
- **CN:** 通过 `input`、`output`、`block_shape`、`layout`、`in_desc`、`out_desc` 准备或更新状态。 调用 `torch.rand`、`torch.empty_like`、`ttgl.NVMMASharedLayout.get_default_for`、`gluon.nvidia.hopper.TensorDescriptor.from_tensor`、`torch.testing.assert_close` 执行测试逻辑。 相关主题：PyTorch 张量准备与校验、张量/描述符元数据、布局变换推理、随机数据生成。

### Lines 1893-1901

```python


@pytest.mark.parametrize("swizzle", [32, 64, 128])
@pytest.mark.parametrize("num_warps", [4, 8])
@pytest.mark.parametrize("M, N, BLOCK_N", [(128, 128, 128), (256, 128, 64), (128, 128, 16)])
@pytest.mark.skipif(not is_blackwell(), reason="Requires Blackwell")
def test_tmem_copy_no_scales(M, N, BLOCK_N, num_warps, swizzle):

    @gluon.jit
```
- **EN:** Defines the test function `test_tmem_copy_no_scales`. Decorators: `pytest.mark.parametrize('swizzle', [32, 64, 128])`, `pytest.mark.parametrize('num_warps', [4, 8])`, `pytest.mark.parametrize('M, N, BLOCK_N', [(128, 128, 128), (256, 128, 64), (128, 128, 16)])`, `pytest.mark.skipif(not is_blackwell(), reason='Requires Blackwell')`. Parameters: `M`, `N`, `BLOCK_N`, `num_warps`, `swizzle`. Nested definitions in this scope: `tmem_copy_no_scales`. Key calls include `pytest.mark.parametrize`, `pytest.mark.skipif`, `torch.empty_like`, `TensorMemoryLayout`, `allocate_tensor_memory`, `tmem.get_reg_layout`, and 15 more. This scope touches pytest parametrization, PyTorch tensor setup and checks, layout transformation reasoning.
- **CN:** 定义测试函数 `test_tmem_copy_no_scales`。 装饰器：`pytest.mark.parametrize('swizzle', [32, 64, 128])`、`pytest.mark.parametrize('num_warps', [4, 8])`、`pytest.mark.parametrize('M, N, BLOCK_N', [(128, 128, 128), (256, 128, 64), (128, 128, 16)])`、`pytest.mark.skipif(not is_blackwell(), reason='Requires Blackwell')`。 参数：`M`、`N`、`BLOCK_N`、`num_warps`、`swizzle`。 该作用域中的嵌套定义：`tmem_copy_no_scales`。 关键调用包括 `pytest.mark.parametrize`、`pytest.mark.skipif`、`torch.empty_like`、`TensorMemoryLayout`、`allocate_tensor_memory`、`tmem.get_reg_layout` 等另外 15 项。 该作用域涉及pytest 参数化、PyTorch 张量准备与校验、布局变换推理。

#### Lines 1901-1903

```python
    @gluon.jit
    def tmem_copy_no_scales(in_ptr, out_ptr, M: ttgl.constexpr, N: ttgl.constexpr, BLOCK_N: ttgl.constexpr,
                            swizzle: ttgl.constexpr, num_warps: ttgl.constexpr):
```
- **EN:** Defines the helper function `tmem_copy_no_scales`. Decorators: `gluon.jit`. Parameters: `in_ptr`, `out_ptr`, `M`, `N`, `BLOCK_N`, `swizzle`, `num_warps`. Key calls include `TensorMemoryLayout`, `allocate_tensor_memory`, `tmem.get_reg_layout`, `ttgl.arange`, `ttgl.load`, `ttgl.NVMMASharedLayout`, and 10 more. This scope touches layout transformation reasoning.
- **CN:** 定义辅助函数 `tmem_copy_no_scales`。 装饰器：`gluon.jit`。 参数：`in_ptr`、`out_ptr`、`M`、`N`、`BLOCK_N`、`swizzle`、`num_warps`。 关键调用包括 `TensorMemoryLayout`、`allocate_tensor_memory`、`tmem.get_reg_layout`、`ttgl.arange`、`ttgl.load`、`ttgl.NVMMASharedLayout` 等另外 10 项。 该作用域涉及布局变换推理。

##### Lines 1904-1930

```python
        tmem_layout: ttgl.constexpr = TensorMemoryLayout(
            block=(128, BLOCK_N),
            col_stride=32 // in_ptr.dtype.element_ty.primitive_bitwidth,
        )
        tmem = allocate_tensor_memory(
            element_ty=in_ptr.dtype.element_ty,
            shape=[M, N],
            layout=tmem_layout,
        )
        tmem_reg_layout: ttgl.constexpr = tmem.get_reg_layout()
        offs_m = ttgl.arange(0, M, ttgl.SliceLayout(1, tmem_reg_layout))
        offs_n = ttgl.arange(0, N, ttgl.SliceLayout(0, tmem_reg_layout))
        offs = offs_m[:, None] * N + offs_n[None, :]

        input = ttgl.load(in_ptr + offs)

        smem_layout: ttgl.constexpr = ttgl.NVMMASharedLayout(swizzle_byte_width=swizzle, element_bitwidth=32, rank=2)
        smem = ttgl.allocate_shared_memory(in_ptr.dtype.element_ty, [M, N], layout=smem_layout)

        smem.store(input)
        bar = ttgl.allocate_shared_memory(ttgl.int64, [1], mbarrier.MBarrierLayout())
        mbarrier.init(bar, count=1)
        tcgen05_copy(smem, tmem)
        tcgen05_commit(bar)
        mbarrier.wait(bar, phase=0)
        output = tmem.load()
        ttgl.store(out_ptr + offs, output)
```
- **EN:** Prepares or updates state through `tmem_layout`, `tmem`, `tmem_reg_layout`, `offs_m`, `offs_n`, `offs`, `input`, `smem_layout`, and 3 more. Invokes `TensorMemoryLayout`, `allocate_tensor_memory`, `tmem.get_reg_layout`, `ttgl.arange`, `ttgl.SliceLayout`, `ttgl.load`, and 10 more to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `tmem_layout`、`tmem`、`tmem_reg_layout`、`offs_m`、`offs_n`、`offs`、`input`、`smem_layout` 等另外 3 项 准备或更新状态。 调用 `TensorMemoryLayout`、`allocate_tensor_memory`、`tmem.get_reg_layout`、`ttgl.arange`、`ttgl.SliceLayout`、`ttgl.load` 等另外 10 项 执行测试逻辑。 相关主题：布局变换推理。

#### Lines 1931-1935

```python

    input = torch.arange(M * N, device="cuda").reshape(M, N).to(torch.int32)
    output = torch.empty_like(input)

    tmem_copy_no_scales[(1, )](input, output, M, N, BLOCK_N, swizzle, num_warps=num_warps)
```
- **EN:** Prepares or updates state through `input`, `output`. Invokes `torch.arange`, `torch.empty_like` to execute the test logic. Relevant themes: PyTorch tensor setup and checks.
- **CN:** 通过 `input`、`output` 准备或更新状态。 调用 `torch.arange`、`torch.empty_like` 执行测试逻辑。 相关主题：PyTorch 张量准备与校验。

#### Lines 1936-1936

```python
    assert (output == input).all()
```
- **EN:** Validates behavior with 1 assertion(s).
- **CN:** 通过 1 个断言验证行为。

### Lines 1937-1940

```python


@gluon.jit
def early_return_kernel(x):
```
- **EN:** Defines the helper function `early_return_kernel`. Decorators: `gluon.jit`. Parameters: `x`. Key calls include `x.sum`.
- **CN:** 定义辅助函数 `early_return_kernel`。 装饰器：`gluon.jit`。 参数：`x`。 关键调用包括 `x.sum`。

#### Lines 1941-1942

```python
    if x.sum(0).sum(0):
        return x
```
- **EN:** Invokes `x.sum` to execute the test logic. Branches on runtime or test conditions.
- **CN:** 调用 `x.sum` 执行测试逻辑。 根据运行时或测试条件进行分支。

#### Lines 1943-1944

```python
    x = x + x
    return x
```
- **EN:** Prepares or updates state through `x`.
- **CN:** 通过 `x` 准备或更新状态。

### Lines 1945-1947

```python


def test_2d_tensor_early_return():
```
- **EN:** Defines the test function `test_2d_tensor_early_return`. Nested definitions in this scope: `kernel`. Key calls include `ttgl.constexpr`, `torch.empty`, `kernel.warmup`, `ttgl.BlockedLayout`, `ttgl.arange`, `ttgl.store`, and 3 more. This scope touches PyTorch tensor setup and checks, layout transformation reasoning.
- **CN:** 定义测试函数 `test_2d_tensor_early_return`。 该作用域中的嵌套定义：`kernel`。 关键调用包括 `ttgl.constexpr`、`torch.empty`、`kernel.warmup`、`ttgl.BlockedLayout`、`ttgl.arange`、`ttgl.store` 等另外 3 项。 该作用域涉及PyTorch 张量准备与校验、布局变换推理。

#### Lines 1948-1948

```python
    warp_size = ttgl.constexpr(THREADS_PER_WARP)
```
- **EN:** Prepares or updates state through `warp_size`. Invokes `ttgl.constexpr` to execute the test logic.
- **CN:** 通过 `warp_size` 准备或更新状态。 调用 `ttgl.constexpr` 执行测试逻辑。

#### Lines 1949-1951

```python

    @gluon.jit
    def kernel(N, out):
```
- **EN:** Defines the helper function `kernel`. Decorators: `gluon.jit`. Parameters: `N`, `out`. Key calls include `ttgl.BlockedLayout`, `ttgl.arange`, `ttgl.store`, `early_return_kernel`, `ttgl.SliceLayout`, `x.sum`. This scope touches layout transformation reasoning.
- **CN:** 定义辅助函数 `kernel`。 装饰器：`gluon.jit`。 参数：`N`、`out`。 关键调用包括 `ttgl.BlockedLayout`、`ttgl.arange`、`ttgl.store`、`early_return_kernel`、`ttgl.SliceLayout`、`x.sum`。 该作用域涉及布局变换推理。

##### Lines 1952-1957

```python
        layout: ttgl.constexpr = ttgl.BlockedLayout([1, 1], [1, warp_size], [1, 4], [1, 0])
        BLOCK: ttgl.constexpr = 32

        x0 = ttgl.arange(0, BLOCK, layout=ttgl.SliceLayout(1, layout))
        x1 = ttgl.arange(0, BLOCK, layout=ttgl.SliceLayout(0, layout))
        x = x0[:, None] * x1[None, :]
```
- **EN:** Prepares or updates state through `layout`, `BLOCK`, `x0`, `x1`, `x`. Invokes `ttgl.BlockedLayout`, `ttgl.arange`, `ttgl.SliceLayout` to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `layout`、`BLOCK`、`x0`、`x1`、`x` 准备或更新状态。 调用 `ttgl.BlockedLayout`、`ttgl.arange`、`ttgl.SliceLayout` 执行测试逻辑。 相关主题：布局变换推理。

##### Lines 1958-1959

```python
        for i in range(N):
            x += early_return_kernel(x)
```
- **EN:** Invokes `early_return_kernel` to execute the test logic. Iterates across cases or data tiles.
- **CN:** 调用 `early_return_kernel` 执行测试逻辑。 通过循环覆盖多个用例或数据分块。

##### Lines 1960-1960

```python
        ttgl.store(out, x.sum(0).sum(0))
```
- **EN:** Invokes `ttgl.store`, `x.sum` to execute the test logic.
- **CN:** 调用 `ttgl.store`、`x.sum` 执行测试逻辑。

#### Lines 1961-1963

```python

    out = torch.empty(1, dtype=torch.int32, device="cuda")
    compiled_kernel = kernel.warmup(N=100, out=out, grid=(1, ))
```
- **EN:** Prepares or updates state through `out`, `compiled_kernel`. Invokes `torch.empty`, `kernel.warmup` to execute the test logic. Relevant themes: PyTorch tensor setup and checks.
- **CN:** 通过 `out`、`compiled_kernel` 准备或更新状态。 调用 `torch.empty`、`kernel.warmup` 执行测试逻辑。 相关主题：PyTorch 张量准备与校验。

#### Lines 1964-1964

```python
    assert compiled_kernel.asm["llir"].count("define") == 1
```
- **EN:** Validates behavior with 1 assertion(s).
- **CN:** 通过 1 个断言验证行为。

### Lines 1965-1970

```python


@pytest.mark.skipif(not is_hip_cdna3() and not is_hip_cdna4(), reason="Requires CDNA3 or CDNA4")
def test_inline_with_amdgpu_dialect():

    @gluon.jit
```
- **EN:** Defines the test function `test_inline_with_amdgpu_dialect`. Decorators: `pytest.mark.skipif(not is_hip_cdna3() and (not is_hip_cdna4()), reason='Requires CDNA3 or CDNA4')`. Nested definitions in this scope: `buffer_load`, `kernel`. Key calls include `pytest.mark.skipif`, `torch.empty_like`, `kernel.warmup`, `ttgl.amd.cdna3.buffer_load`, `ttgl.BlockedLayout`, `ttgl.arange`, and 5 more. This scope touches PyTorch tensor setup and checks, plugin or compiler extension points, layout transformation reasoning.
- **CN:** 定义测试函数 `test_inline_with_amdgpu_dialect`。 装饰器：`pytest.mark.skipif(not is_hip_cdna3() and (not is_hip_cdna4()), reason='Requires CDNA3 or CDNA4')`。 该作用域中的嵌套定义：`buffer_load`、`kernel`。 关键调用包括 `pytest.mark.skipif`、`torch.empty_like`、`kernel.warmup`、`ttgl.amd.cdna3.buffer_load`、`ttgl.BlockedLayout`、`ttgl.arange` 等另外 5 项。 该作用域涉及PyTorch 张量准备与校验、插件或编译器扩展点、布局变换推理。

#### Lines 1970-1971

```python
    @gluon.jit
    def buffer_load(x, offsets):
```
- **EN:** Defines the helper function `buffer_load`. Decorators: `gluon.jit`. Parameters: `x`, `offsets`. Key calls include `ttgl.amd.cdna3.buffer_load`.
- **CN:** 定义辅助函数 `buffer_load`。 装饰器：`gluon.jit`。 参数：`x`、`offsets`。 关键调用包括 `ttgl.amd.cdna3.buffer_load`。

##### Lines 1972-1972

```python
        return ttgl.amd.cdna3.buffer_load(ptr=x, offsets=offsets)
```
- **EN:** Invokes `ttgl.amd.cdna3.buffer_load` to execute the test logic.
- **CN:** 调用 `ttgl.amd.cdna3.buffer_load` 执行测试逻辑。

#### Lines 1973-1975

```python

    @gluon.jit
    def kernel(x, y):
```
- **EN:** Defines the helper function `kernel`. Decorators: `gluon.jit`. Parameters: `x`, `y`. Key calls include `ttgl.BlockedLayout`, `ttgl.arange`, `buffer_load`, `ttgl.amd.cdna3.buffer_store`. This scope touches layout transformation reasoning.
- **CN:** 定义辅助函数 `kernel`。 装饰器：`gluon.jit`。 参数：`x`、`y`。 关键调用包括 `ttgl.BlockedLayout`、`ttgl.arange`、`buffer_load`、`ttgl.amd.cdna3.buffer_store`。 该作用域涉及布局变换推理。

##### Lines 1976-1981

```python
        layout: ttgl.constexpr = ttgl.BlockedLayout(size_per_thread=[1], threads_per_warp=[64], warps_per_cta=[4],
                                                    order=[0])
        offsets = ttgl.arange(0, 64, layout=layout)

        a = buffer_load(x, offsets)
        ttgl.amd.cdna3.buffer_store(stored_value=a, ptr=y, offsets=offsets)
```
- **EN:** Prepares or updates state through `layout`, `offsets`, `a`. Invokes `ttgl.BlockedLayout`, `ttgl.arange`, `buffer_load`, `ttgl.amd.cdna3.buffer_store` to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `layout`、`offsets`、`a` 准备或更新状态。 调用 `ttgl.BlockedLayout`、`ttgl.arange`、`buffer_load`、`ttgl.amd.cdna3.buffer_store` 执行测试逻辑。 相关主题：布局变换推理。

#### Lines 1982-1986

```python

    input = torch.arange(64, device="cuda").to(torch.int32)
    output = torch.empty_like(input)

    compiled_kernel = kernel.warmup(input, output, grid=(1, ))
```
- **EN:** Prepares or updates state through `input`, `output`, `compiled_kernel`. Invokes `torch.arange`, `torch.empty_like`, `kernel.warmup` to execute the test logic. Relevant themes: PyTorch tensor setup and checks.
- **CN:** 通过 `input`、`output`、`compiled_kernel` 准备或更新状态。 调用 `torch.arange`、`torch.empty_like`、`kernel.warmup` 执行测试逻辑。 相关主题：PyTorch 张量准备与校验。

#### Lines 1987-1987

```python
    assert compiled_kernel.asm["ttgir"].count("tt.func private") == 0
```
- **EN:** Validates behavior with 1 assertion(s).
- **CN:** 通过 1 个断言验证行为。

### Lines 1988-1997

```python


@pytest.mark.parametrize("interval_pairs", [[[32, 4]], [[16, 4]], [[16, 4], [64, 8]]])
@pytest.mark.parametrize(
    "shared_layout",
    [{"order": [0, 1]}, {"order": [1, 0]},
     {"offsets": [[0, 1], [0, 2], [0, 8], [0, 4], [0, 16], [0, 32], [2, 0], [1, 0], [4, 0], [8, 0], [16, 0], [32, 0]]}])
@pytest.mark.parametrize("slice_m_offset, slice_n_offset, slice_m, slice_n", [(48, 16, 16, 16), (32, 48, 32, 16),
                                                                              (48, 32, 16, 32)])
def test_padded_shared_layout_subslice(interval_pairs, shared_layout, slice_m_offset, slice_n_offset, slice_m, slice_n):
```
- **EN:** Defines the test function `test_padded_shared_layout_subslice`. Decorators: `pytest.mark.parametrize('interval_pairs', [[[32, 4]], [[16, 4]], [[16, 4], [64, 8]]])`, `pytest.mark.parametrize('shared_layout', [{'order': [0, 1]}, {'order': [1, 0]}, {'offsets': [[0, 1], [0, 2], [0, 8], [0, 4], [0, 16], [0, 32], [2, 0], [1, 0], [4, 0], [8, 0], [16, 0], [32, 0]]}])`, `pytest.mark.parametrize('slice_m_offset, slice_n_offset, slice_m, slice_n', [(48, 16, 16, 16), (32, 48, 32, 16), (48, 32, 16, 32)])`. Parameters: `interval_pairs`, `shared_layout`, `slice_m_offset`, `slice_n_offset`, `slice_m`, `slice_n`. Nested definitions in this scope: `kernel`. Key calls include `pytest.mark.parametrize`, `ttgl.constexpr`, `torch.zeros`, `ttgl.BlockedLayout`, `ttgl.arange`, `ttgl.load`, and 10 more. This scope touches pytest parametrization, PyTorch tensor setup and checks, layout transformation reasoning.
- **CN:** 定义测试函数 `test_padded_shared_layout_subslice`。 装饰器：`pytest.mark.parametrize('interval_pairs', [[[32, 4]], [[16, 4]], [[16, 4], [64, 8]]])`、`pytest.mark.parametrize('shared_layout', [{'order': [0, 1]}, {'order': [1, 0]}, {'offsets': [[0, 1], [0, 2], [0, 8], [0, 4], [0, 16], [0, 32], [2, 0], [1, 0], [4, 0], [8, 0], [16, 0], [32, 0]]}])`、`pytest.mark.parametrize('slice_m_offset, slice_n_offset, slice_m, slice_n', [(48, 16, 16, 16), (32, 48, 32, 16), (48, 32, 16, 32)])`。 参数：`interval_pairs`、`shared_layout`、`slice_m_offset`、`slice_n_offset`、`slice_m`、`slice_n`。 该作用域中的嵌套定义：`kernel`。 关键调用包括 `pytest.mark.parametrize`、`ttgl.constexpr`、`torch.zeros`、`ttgl.BlockedLayout`、`ttgl.arange`、`ttgl.load` 等另外 10 项。 该作用域涉及pytest 参数化、PyTorch 张量准备与校验、布局变换推理。

#### Lines 1998-2004

```python
    m = 64
    n = 64
    num_warps = 1
    num_warps_cst = ttgl.constexpr(num_warps)
    warp_size_cst = ttgl.constexpr(THREADS_PER_WARP)

    shape = [m, n]
```
- **EN:** Prepares or updates state through `m`, `n`, `num_warps`, `num_warps_cst`, `warp_size_cst`, `shape`. Invokes `ttgl.constexpr` to execute the test logic.
- **CN:** 通过 `m`、`n`、`num_warps`、`num_warps_cst`、`warp_size_cst`、`shape` 准备或更新状态。 调用 `ttgl.constexpr` 执行测试逻辑。

#### Lines 2005-2011

```python
    if "order" in shared_layout:
        order = shared_layout["order"]
        smem_layout = ttgl.constexpr(ttgl.PaddedSharedLayout.with_identity_for(interval_pairs, shape, order))
    elif "offsets" in shared_layout:
        offsets = shared_layout["offsets"]
        blocks = []
        smem_layout = ttgl.constexpr(ttgl.PaddedSharedLayout(interval_pairs, offsets, blocks, shape))
```
- **EN:** Invokes `ttgl.constexpr`, `ttgl.PaddedSharedLayout.with_identity_for`, `ttgl.PaddedSharedLayout` to execute the test logic. Branches on runtime or test conditions. Relevant themes: layout transformation reasoning.
- **CN:** 调用 `ttgl.constexpr`、`ttgl.PaddedSharedLayout.with_identity_for`、`ttgl.PaddedSharedLayout` 执行测试逻辑。 根据运行时或测试条件进行分支。 相关主题：布局变换推理。

#### Lines 2012-2015

```python

    @gluon.jit
    def kernel(in_ptr, out_ptr, M: ttgl.constexpr, N: ttgl.constexpr, SLICE_M_OFFSET: ttgl.constexpr,
               SLICE_N_OFFSET: ttgl.constexpr, SLICE_M: ttgl.constexpr, SLICE_N: ttgl.constexpr):
```
- **EN:** Defines the helper function `kernel`. Decorators: `gluon.jit`. Parameters: `in_ptr`, `out_ptr`, `M`, `N`, `SLICE_M_OFFSET`, `SLICE_N_OFFSET`, `SLICE_M`, `SLICE_N`. Key calls include `ttgl.BlockedLayout`, `ttgl.arange`, `ttgl.load`, `ttgl.allocate_shared_memory`, `smem.slice`, `smem_slice0.slice`, and 4 more. This scope touches layout transformation reasoning.
- **CN:** 定义辅助函数 `kernel`。 装饰器：`gluon.jit`。 参数：`in_ptr`、`out_ptr`、`M`、`N`、`SLICE_M_OFFSET`、`SLICE_N_OFFSET`、`SLICE_M`、`SLICE_N`。 关键调用包括 `ttgl.BlockedLayout`、`ttgl.arange`、`ttgl.load`、`ttgl.allocate_shared_memory`、`smem.slice`、`smem_slice0.slice` 等另外 4 项。 该作用域涉及布局变换推理。

##### Lines 2016-2034

```python
        blocked: ttgl.constexpr = ttgl.BlockedLayout([1, 1], [warp_size_cst, 1], [1, num_warps_cst], [1, 0])
        offs_m_load = ttgl.arange(0, M, ttgl.SliceLayout(1, blocked))
        offs_n_load = ttgl.arange(0, N, ttgl.SliceLayout(0, blocked))
        in_offs = offs_m_load[:, None] * N + offs_n_load[None, :]

        in_data = ttgl.load(in_ptr + in_offs)

        smem = ttgl.allocate_shared_memory(ttgl.int32, [M, N], smem_layout)
        smem_slice0 = smem.slice(SLICE_M_OFFSET, SLICE_M, dim=0)
        smem_slice1 = smem_slice0.slice(SLICE_N_OFFSET, SLICE_N, dim=1)

        smem.store(in_data)

        out_data = smem_slice1.load(blocked)

        offs_m_store = ttgl.arange(0, SLICE_M, ttgl.SliceLayout(1, blocked))
        offs_n_store = ttgl.arange(0, SLICE_N, ttgl.SliceLayout(0, blocked))
        out_offs = offs_m_store[:, None] * SLICE_N + offs_n_store[None, :]
        ttgl.store(out_ptr + out_offs, out_data)
```
- **EN:** Prepares or updates state through `blocked`, `offs_m_load`, `offs_n_load`, `in_offs`, `in_data`, `smem`, `smem_slice0`, `smem_slice1`, and 4 more. Invokes `ttgl.BlockedLayout`, `ttgl.arange`, `ttgl.SliceLayout`, `ttgl.load`, `ttgl.allocate_shared_memory`, `smem.slice`, and 4 more to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `blocked`、`offs_m_load`、`offs_n_load`、`in_offs`、`in_data`、`smem`、`smem_slice0`、`smem_slice1` 等另外 4 项 准备或更新状态。 调用 `ttgl.BlockedLayout`、`ttgl.arange`、`ttgl.SliceLayout`、`ttgl.load`、`ttgl.allocate_shared_memory`、`smem.slice` 等另外 4 项 执行测试逻辑。 相关主题：布局变换推理。

#### Lines 2035-2040

```python

    input = torch.arange(m * n, device="cuda").reshape(m, n).to(torch.int32)
    output = torch.zeros((slice_m, slice_n), dtype=torch.int32, device="cuda")
    ref_output = input[slice_m_offset:slice_m_offset + slice_m, slice_n_offset:slice_n_offset + slice_n]

    kernel[(1, )](input, output, m, n, slice_m_offset, slice_n_offset, slice_m, slice_n, num_warps=num_warps)
```
- **EN:** Prepares or updates state through `input`, `output`, `ref_output`. Invokes `torch.arange`, `torch.zeros` to execute the test logic. Relevant themes: PyTorch tensor setup and checks.
- **CN:** 通过 `input`、`output`、`ref_output` 准备或更新状态。 调用 `torch.arange`、`torch.zeros` 执行测试逻辑。 相关主题：PyTorch 张量准备与校验。

#### Lines 2041-2042

```python

    assert (output == ref_output).all()
```
- **EN:** Validates behavior with 1 assertion(s).
- **CN:** 通过 1 个断言验证行为。

### Lines 2043-2047

```python


@pytest.mark.skipif(not is_blackwell(), reason="Requires Blackwell")
@pytest.mark.parametrize("op, tol", [("add", 0), ("sub", 0), ("mul", 0), ("fma", 1e-6)])
def test_float2(op, tol):
```
- **EN:** Defines the test function `test_float2`. Decorators: `pytest.mark.skipif(not is_blackwell(), reason='Requires Blackwell')`, `pytest.mark.parametrize('op, tol', [('add', 0), ('sub', 0), ('mul', 0), ('fma', 1e-06)])`. Parameters: `op`, `tol`. Nested definitions in this scope: `kernel`. Key calls include `pytest.mark.skipif`, `pytest.mark.parametrize`, `ttgl.constexpr`, `torch.manual_seed`, `torch.rand`, `torch.empty`, and 11 more. This scope touches pytest parametrization, PyTorch tensor setup and checks, layout transformation reasoning, random-data generation.
- **CN:** 定义测试函数 `test_float2`。 装饰器：`pytest.mark.skipif(not is_blackwell(), reason='Requires Blackwell')`、`pytest.mark.parametrize('op, tol', [('add', 0), ('sub', 0), ('mul', 0), ('fma', 1e-06)])`。 参数：`op`、`tol`。 该作用域中的嵌套定义：`kernel`。 关键调用包括 `pytest.mark.skipif`、`pytest.mark.parametrize`、`ttgl.constexpr`、`torch.manual_seed`、`torch.rand`、`torch.empty` 等另外 11 项。 该作用域涉及pytest 参数化、PyTorch 张量准备与校验、布局变换推理、随机数据生成。

#### Lines 2048-2051

```python
    BLOCK_M = ttgl.constexpr(128)
    BLOCK_N = ttgl.constexpr(128)
    threads_per_warp = ttgl.constexpr(THREADS_PER_WARP)
    op = ttgl.constexpr(op)
```
- **EN:** Prepares or updates state through `BLOCK_M`, `BLOCK_N`, `threads_per_warp`, `op`. Invokes `ttgl.constexpr` to execute the test logic.
- **CN:** 通过 `BLOCK_M`、`BLOCK_N`、`threads_per_warp`、`op` 准备或更新状态。 调用 `ttgl.constexpr` 执行测试逻辑。

#### Lines 2052-2054

```python

    @gluon.jit
    def kernel(a_ptr, b_ptr, c_ptr, out_ptr):
```
- **EN:** Defines the helper function `kernel`. Decorators: `gluon.jit`. Parameters: `a_ptr`, `b_ptr`, `c_ptr`, `out_ptr`. Key calls include `ttgl.BlockedLayout`, `ttgl.load`, `float2.pack`, `float2.unpack`, `ttgl.store`, `ttgl.arange`, and 3 more. This scope touches layout transformation reasoning.
- **CN:** 定义辅助函数 `kernel`。 装饰器：`gluon.jit`。 参数：`a_ptr`、`b_ptr`、`c_ptr`、`out_ptr`。 关键调用包括 `ttgl.BlockedLayout`、`ttgl.load`、`float2.pack`、`float2.unpack`、`ttgl.store`、`ttgl.arange` 等另外 3 项。 该作用域涉及布局变换推理。

##### Lines 2055-2068

```python
        layout: ttgl.constexpr = ttgl.BlockedLayout(
            size_per_thread=[1, BLOCK_N],
            threads_per_warp=[threads_per_warp, 1],
            warps_per_cta=[ttgl.num_warps(), 1],
            order=[0, 1],
        )
        offs_m = ttgl.arange(0, BLOCK_M, layout=ttgl.SliceLayout(1, layout))[:, None]
        offs_n = ttgl.arange(0, BLOCK_N, layout=ttgl.SliceLayout(0, layout))[None, :]
        a = ttgl.load(a_ptr + offs_m * BLOCK_N + offs_n)
        b = ttgl.load(b_ptr + offs_m * BLOCK_N + offs_n)
        c = ttgl.load(c_ptr + offs_m * BLOCK_N + offs_n)
        a = float2.pack(a, axis=1)
        b = float2.pack(b, axis=1)
        c = float2.pack(c, axis=1)
```
- **EN:** Prepares or updates state through `layout`, `offs_m`, `offs_n`, `a`, `b`, `c`. Invokes `ttgl.BlockedLayout`, `ttgl.num_warps`, `ttgl.arange`, `ttgl.SliceLayout`, `ttgl.load`, `float2.pack` to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `layout`、`offs_m`、`offs_n`、`a`、`b`、`c` 准备或更新状态。 调用 `ttgl.BlockedLayout`、`ttgl.num_warps`、`ttgl.arange`、`ttgl.SliceLayout`、`ttgl.load`、`float2.pack` 执行测试逻辑。 相关主题：布局变换推理。

##### Lines 2069-2077

```python

        if op == "add":
            out = a + b
        elif op == "sub":
            out = a - b
        elif op == "mul":
            out = a * b
        elif op == "fma":
            out = float2.fma(a, b, c)
```
- **EN:** Invokes `float2.fma` to execute the test logic. Branches on runtime or test conditions.
- **CN:** 调用 `float2.fma` 执行测试逻辑。 根据运行时或测试条件进行分支。

##### Lines 2078-2080

```python

        out = float2.unpack(out, axis=1)
        ttgl.store(out_ptr + offs_m * BLOCK_N + offs_n, out)
```
- **EN:** Prepares or updates state through `out`. Invokes `float2.unpack`, `ttgl.store` to execute the test logic.
- **CN:** 通过 `out` 准备或更新状态。 调用 `float2.unpack`、`ttgl.store` 执行测试逻辑。

#### Lines 2081-2089

```python

    torch.manual_seed(0)
    shape = [BLOCK_M.value, BLOCK_N.value]
    a = torch.rand(shape, dtype=torch.float32, device="cuda")
    b = torch.rand(shape, dtype=torch.float32, device="cuda")
    c = torch.rand(shape, dtype=torch.float32, device="cuda")
    out = torch.empty(shape, dtype=torch.float32, device="cuda")

    kernel[(1, )](a, b, c, out)
```
- **EN:** Prepares or updates state through `shape`, `a`, `b`, `c`, `out`. Invokes `torch.manual_seed`, `torch.rand`, `torch.empty` to execute the test logic. Relevant themes: PyTorch tensor setup and checks, random-data generation.
- **CN:** 通过 `shape`、`a`、`b`、`c`、`out` 准备或更新状态。 调用 `torch.manual_seed`、`torch.rand`、`torch.empty` 执行测试逻辑。 相关主题：PyTorch 张量准备与校验、随机数据生成。

#### Lines 2090-2097

```python
    if op == "add":
        ref = a + b
    elif op == "sub":
        ref = a - b
    elif op == "mul":
        ref = a * b
    elif op == "fma":
        ref = a * b + c
```
- **EN:** Branches on runtime or test conditions.
- **CN:** 根据运行时或测试条件进行分支。

#### Lines 2098-2098

```python
    torch.testing.assert_close(ref, out, atol=tol, rtol=tol)
```
- **EN:** Invokes `torch.testing.assert_close` to execute the test logic. Relevant themes: PyTorch tensor setup and checks.
- **CN:** 调用 `torch.testing.assert_close` 执行测试逻辑。 相关主题：PyTorch 张量准备与校验。

### Lines 2099-2102

```python


@pytest.mark.skipif(not is_hip_cdna4(), reason="Requires CDNA4")
def test_buffer_atomic_rmw_add_bf16():
```
- **EN:** Defines the test function `test_buffer_atomic_rmw_add_bf16`. Decorators: `pytest.mark.skipif(not is_hip_cdna4(), reason='Requires CDNA4')`. Nested definitions in this scope: `kernel`. Key calls include `pytest.mark.skipif`, `torch.randn`, `a.clone`, `torch.testing.assert_close`, `ttgl.BlockedLayout`, `ttgl.arange`, and 6 more. This scope touches PyTorch tensor setup and checks, layout transformation reasoning, random-data generation.
- **CN:** 定义测试函数 `test_buffer_atomic_rmw_add_bf16`。 装饰器：`pytest.mark.skipif(not is_hip_cdna4(), reason='Requires CDNA4')`。 该作用域中的嵌套定义：`kernel`。 关键调用包括 `pytest.mark.skipif`、`torch.randn`、`a.clone`、`torch.testing.assert_close`、`ttgl.BlockedLayout`、`ttgl.arange` 等另外 6 项。 该作用域涉及PyTorch 张量准备与校验、布局变换推理、随机数据生成。

#### Lines 2103-2105

```python
    BLOCK = 128
    elem_type = torch.bfloat16
    SIZE_PER_THREAD = 8
```
- **EN:** Prepares or updates state through `BLOCK`, `elem_type`, `SIZE_PER_THREAD`. Relevant themes: PyTorch tensor setup and checks.
- **CN:** 通过 `BLOCK`、`elem_type`、`SIZE_PER_THREAD` 准备或更新状态。 相关主题：PyTorch 张量准备与校验。

#### Lines 2106-2108

```python

    @gluon.jit
    def kernel(a, BLOCK: ttgl.constexpr, SIZE_PER_THREAD: ttgl.constexpr):
```
- **EN:** Defines the helper function `kernel`. Decorators: `gluon.jit`. Parameters: `a`, `BLOCK`, `SIZE_PER_THREAD`. Key calls include `ttgl.BlockedLayout`, `ttgl.arange`, `ttgl.full`, `ttgl.amd.cdna4.buffer_atomic_add`. This scope touches layout transformation reasoning.
- **CN:** 定义辅助函数 `kernel`。 装饰器：`gluon.jit`。 参数：`a`、`BLOCK`、`SIZE_PER_THREAD`。 关键调用包括 `ttgl.BlockedLayout`、`ttgl.arange`、`ttgl.full`、`ttgl.amd.cdna4.buffer_atomic_add`。 该作用域涉及布局变换推理。

##### Lines 2109-2112

```python
        blocked: ttgl.constexpr = ttgl.BlockedLayout([SIZE_PER_THREAD], [64], [4], [0])
        offsets = ttgl.arange(0, BLOCK, layout=blocked)
        val = ttgl.full([BLOCK], 1.0, ttgl.bfloat16, layout=blocked)
        ttgl.amd.cdna4.buffer_atomic_add(a, offsets, val, mask=1, scope="cta", sem="relaxed")
```
- **EN:** Prepares or updates state through `blocked`, `offsets`, `val`. Invokes `ttgl.BlockedLayout`, `ttgl.arange`, `ttgl.full`, `ttgl.amd.cdna4.buffer_atomic_add` to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `blocked`、`offsets`、`val` 准备或更新状态。 调用 `ttgl.BlockedLayout`、`ttgl.arange`、`ttgl.full`、`ttgl.amd.cdna4.buffer_atomic_add` 执行测试逻辑。 相关主题：布局变换推理。

#### Lines 2113-2121

```python

    a = torch.randn((BLOCK), dtype=elem_type, device="cuda")
    origin_a = a.clone()
    compiled = kernel[(1, )](a, BLOCK, SIZE_PER_THREAD)

    torch_ref = origin_a + torch.ones((BLOCK, ), device='cuda', dtype=torch.bfloat16)
    torch.testing.assert_close(a, torch_ref)

    ttgir = compiled.asm["ttgir"]
```
- **EN:** Prepares or updates state through `a`, `origin_a`, `compiled`, `torch_ref`, `ttgir`. Invokes `torch.randn`, `a.clone`, `torch.ones`, `torch.testing.assert_close` to execute the test logic. Relevant themes: PyTorch tensor setup and checks, random-data generation.
- **CN:** 通过 `a`、`origin_a`、`compiled`、`torch_ref`、`ttgir` 准备或更新状态。 调用 `torch.randn`、`a.clone`、`torch.ones`、`torch.testing.assert_close` 执行测试逻辑。 相关主题：PyTorch 张量准备与校验、随机数据生成。

#### Lines 2122-2122

```python
    assert ttgir.count("amdg.buffer_atomic_rmw fadd, relaxed, cta") == 1
```
- **EN:** Invokes `ttgir.count` to execute the test logic. Validates behavior with 1 assertion(s).
- **CN:** 调用 `ttgir.count` 执行测试逻辑。 通过 1 个断言验证行为。

#### Lines 2123-2124

```python

    llir = compiled.asm["llir"]
```
- **EN:** Prepares or updates state through `llir`.
- **CN:** 通过 `llir` 准备或更新状态。

#### Lines 2125-2125

```python
    assert llir.count("tail call <2 x bfloat> @llvm.amdgcn.raw.ptr.buffer.atomic.fadd.v2bf16") == SIZE_PER_THREAD // 2
```
- **EN:** Invokes `llir.count` to execute the test logic. Validates behavior with 1 assertion(s).
- **CN:** 调用 `llir.count` 执行测试逻辑。 通过 1 个断言验证行为。

### Lines 2126-2130

```python


@pytest.mark.skipif(not is_ampere_or_newer(), reason="Requires Ampere or newer")
@pytest.mark.parametrize("dtype", [torch.float16, torch.bfloat16, torch.float32])
def test_mma_v2(dtype):
```
- **EN:** Defines the test function `test_mma_v2`. Decorators: `pytest.mark.skipif(not is_ampere_or_newer(), reason='Requires Ampere or newer')`, `pytest.mark.parametrize('dtype', [torch.float16, torch.bfloat16, torch.float32])`. Parameters: `dtype`. Nested definitions in this scope: `kernel`. Key calls include `pytest.mark.skipif`, `pytest.mark.parametrize`, `torch.manual_seed`, `ttgl.constexpr`, `torch.randn`, `torch.empty`, and 14 more. This scope touches pytest parametrization, PyTorch tensor setup and checks, layout transformation reasoning, random-data generation.
- **CN:** 定义测试函数 `test_mma_v2`。 装饰器：`pytest.mark.skipif(not is_ampere_or_newer(), reason='Requires Ampere or newer')`、`pytest.mark.parametrize('dtype', [torch.float16, torch.bfloat16, torch.float32])`。 参数：`dtype`。 该作用域中的嵌套定义：`kernel`。 关键调用包括 `pytest.mark.skipif`、`pytest.mark.parametrize`、`torch.manual_seed`、`ttgl.constexpr`、`torch.randn`、`torch.empty` 等另外 14 项。 该作用域涉及pytest 参数化、PyTorch 张量准备与校验、布局变换推理、随机数据生成。

#### Lines 2131-2133

```python
    torch.manual_seed(42)
    B = ttgl.constexpr(128)
    threads_per_warp = ttgl.constexpr(THREADS_PER_WARP)
```
- **EN:** Prepares or updates state through `B`, `threads_per_warp`. Invokes `torch.manual_seed`, `ttgl.constexpr` to execute the test logic. Relevant themes: PyTorch tensor setup and checks, random-data generation.
- **CN:** 通过 `B`、`threads_per_warp` 准备或更新状态。 调用 `torch.manual_seed`、`ttgl.constexpr` 执行测试逻辑。 相关主题：PyTorch 张量准备与校验、随机数据生成。

#### Lines 2134-2136

```python

    @gluon.jit
    def kernel(a_ptr, b_ptr, c_ptr, out_ptr):
```
- **EN:** Defines the helper function `kernel`. Decorators: `gluon.jit`. Parameters: `a_ptr`, `b_ptr`, `c_ptr`, `out_ptr`. Key calls include `ttgl.BlockedLayout`, `ttgl.NVMMADistributedLayout`, `ttgl.DotOperandLayout`, `ttgl.convert_layout`, `ttgl.store`, `ttgl.arange`, and 5 more. This scope touches layout transformation reasoning, random-data generation.
- **CN:** 定义辅助函数 `kernel`。 装饰器：`gluon.jit`。 参数：`a_ptr`、`b_ptr`、`c_ptr`、`out_ptr`。 关键调用包括 `ttgl.BlockedLayout`、`ttgl.NVMMADistributedLayout`、`ttgl.DotOperandLayout`、`ttgl.convert_layout`、`ttgl.store`、`ttgl.arange` 等另外 5 项。 该作用域涉及布局变换推理、随机数据生成。

##### Lines 2137-2148

```python
        layout: ttgl.constexpr = ttgl.BlockedLayout([1, 1], [threads_per_warp, 1], [ttgl.num_warps(), 1], [1, 0])
        acc_layout: ttgl.constexpr = ttgl.NVMMADistributedLayout(version=[2, 0], warps_per_cta=[ttgl.num_warps(), 1],
                                                                 instr_shape=[16, 8])
        lhs_layout: ttgl.constexpr = ttgl.DotOperandLayout(parent=acc_layout, operand_index=0, k_width=8)
        rhs_layout: ttgl.constexpr = ttgl.DotOperandLayout(parent=acc_layout, operand_index=1, k_width=8)

        offs_m = ttgl.arange(0, B, layout=ttgl.SliceLayout(1, layout))[:, None]
        offs_n = ttgl.arange(0, B, layout=ttgl.SliceLayout(0, layout))[None, :]
        offs = offs_m * B + offs_n
        a = ttgl.convert_layout(ttgl.load(a_ptr + offs), lhs_layout)
        b = ttgl.convert_layout(ttgl.load(b_ptr + offs), rhs_layout)
        c = ttgl.convert_layout(ttgl.load(c_ptr + offs), acc_layout)
```
- **EN:** Prepares or updates state through `layout`, `acc_layout`, `lhs_layout`, `rhs_layout`, `offs_m`, `offs_n`, `offs`, `a`, and 2 more. Invokes `ttgl.BlockedLayout`, `ttgl.num_warps`, `ttgl.NVMMADistributedLayout`, `ttgl.DotOperandLayout`, `ttgl.arange`, `ttgl.SliceLayout`, and 2 more to execute the test logic. Relevant themes: layout transformation reasoning, random-data generation.
- **CN:** 通过 `layout`、`acc_layout`、`lhs_layout`、`rhs_layout`、`offs_m`、`offs_n`、`offs`、`a` 等另外 2 项 准备或更新状态。 调用 `ttgl.BlockedLayout`、`ttgl.num_warps`、`ttgl.NVMMADistributedLayout`、`ttgl.DotOperandLayout`、`ttgl.arange`、`ttgl.SliceLayout` 等另外 2 项 执行测试逻辑。 相关主题：布局变换推理、随机数据生成。

##### Lines 2149-2152

```python
        if c.dtype == ttgl.bfloat16:
            out = mma_v2(a, b, c.to(ttgl.float32), input_precision="tf32").to(ttgl.bfloat16)
        else:
            out = mma_v2(a, b, c, input_precision="tf32")
```
- **EN:** Invokes `mma_v2`, `c.to` to execute the test logic. Branches on runtime or test conditions.
- **CN:** 调用 `mma_v2`、`c.to` 执行测试逻辑。 根据运行时或测试条件进行分支。

##### Lines 2153-2153

```python
        ttgl.store(out_ptr + offs, ttgl.convert_layout(out, layout))
```
- **EN:** Invokes `ttgl.store`, `ttgl.convert_layout` to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 调用 `ttgl.store`、`ttgl.convert_layout` 执行测试逻辑。 相关主题：布局变换推理。

#### Lines 2154-2160

```python

    a = torch.randn((B, B), dtype=dtype, device="cuda")
    b = torch.randn((B, B), dtype=dtype, device="cuda")
    c = torch.randn((B, B), dtype=dtype, device="cuda")
    out = torch.empty((B, B), dtype=dtype, device="cuda")
    kernel[(1, )](a, b, c, out)
    torch.testing.assert_close(out, torch.addmm(c, a, b), atol=0.05, rtol=1e-2)
```
- **EN:** Prepares or updates state through `a`, `b`, `c`, `out`. Invokes `torch.randn`, `torch.empty`, `torch.testing.assert_close`, `torch.addmm` to execute the test logic. Relevant themes: PyTorch tensor setup and checks, random-data generation.
- **CN:** 通过 `a`、`b`、`c`、`out` 准备或更新状态。 调用 `torch.randn`、`torch.empty`、`torch.testing.assert_close`、`torch.addmm` 执行测试逻辑。 相关主题：PyTorch 张量准备与校验、随机数据生成。

### Lines 2161-2163

```python


def test_dot_fma():
```
- **EN:** Defines the test function `test_dot_fma`. Nested definitions in this scope: `kernel`. Key calls include `torch.manual_seed`, `ttgl.constexpr`, `torch.rand`, `torch.empty`, `torch.testing.assert_close`, `ttgl.BlockedLayout`, and 9 more. This scope touches PyTorch tensor setup and checks, layout transformation reasoning, random-data generation.
- **CN:** 定义测试函数 `test_dot_fma`。 该作用域中的嵌套定义：`kernel`。 关键调用包括 `torch.manual_seed`、`ttgl.constexpr`、`torch.rand`、`torch.empty`、`torch.testing.assert_close`、`ttgl.BlockedLayout` 等另外 9 项。 该作用域涉及PyTorch 张量准备与校验、布局变换推理、随机数据生成。

#### Lines 2164-2166

```python
    torch.manual_seed(42)
    B = ttgl.constexpr(32)
    threads_per_warp = ttgl.constexpr(THREADS_PER_WARP)
```
- **EN:** Prepares or updates state through `B`, `threads_per_warp`. Invokes `torch.manual_seed`, `ttgl.constexpr` to execute the test logic. Relevant themes: PyTorch tensor setup and checks, random-data generation.
- **CN:** 通过 `B`、`threads_per_warp` 准备或更新状态。 调用 `torch.manual_seed`、`ttgl.constexpr` 执行测试逻辑。 相关主题：PyTorch 张量准备与校验、随机数据生成。

#### Lines 2167-2169

```python

    @gluon.jit
    def kernel(a_ptr, b_ptr, c_ptr, out_ptr):
```
- **EN:** Defines the helper function `kernel`. Decorators: `gluon.jit`. Parameters: `a_ptr`, `b_ptr`, `c_ptr`, `out_ptr`. Key calls include `ttgl.BlockedLayout`, `ttgl.DotOperandLayout`, `ttgl.convert_layout`, `ttgl.load`, `ttgl.dot_fma`, `ttgl.store`, and 3 more. This scope touches layout transformation reasoning, random-data generation.
- **CN:** 定义辅助函数 `kernel`。 装饰器：`gluon.jit`。 参数：`a_ptr`、`b_ptr`、`c_ptr`、`out_ptr`。 关键调用包括 `ttgl.BlockedLayout`、`ttgl.DotOperandLayout`、`ttgl.convert_layout`、`ttgl.load`、`ttgl.dot_fma`、`ttgl.store` 等另外 3 项。 该作用域涉及布局变换推理、随机数据生成。

##### Lines 2170-2181

```python
        layout: ttgl.constexpr = ttgl.BlockedLayout([1, 1], [threads_per_warp, 1], [ttgl.num_warps(), 1], [1, 0])
        lhs_layout: ttgl.constexpr = ttgl.DotOperandLayout(parent=layout, operand_index=0, k_width=0)
        rhs_layout: ttgl.constexpr = ttgl.DotOperandLayout(parent=layout, operand_index=1, k_width=0)

        offs_m = ttgl.arange(0, B, layout=ttgl.SliceLayout(1, layout))[:, None]
        offs_n = ttgl.arange(0, B, layout=ttgl.SliceLayout(0, layout))[None, :]
        offs = offs_m * B + offs_n
        a = ttgl.convert_layout(ttgl.load(a_ptr + offs), lhs_layout)
        b = ttgl.convert_layout(ttgl.load(b_ptr + offs), rhs_layout)
        c = ttgl.load(c_ptr + offs)
        out = ttgl.dot_fma(a, b, c)
        ttgl.store(out_ptr + offs, out)
```
- **EN:** Prepares or updates state through `layout`, `lhs_layout`, `rhs_layout`, `offs_m`, `offs_n`, `offs`, `a`, `b`, and 2 more. Invokes `ttgl.BlockedLayout`, `ttgl.num_warps`, `ttgl.DotOperandLayout`, `ttgl.arange`, `ttgl.SliceLayout`, `ttgl.convert_layout`, and 3 more to execute the test logic. Relevant themes: layout transformation reasoning, random-data generation.
- **CN:** 通过 `layout`、`lhs_layout`、`rhs_layout`、`offs_m`、`offs_n`、`offs`、`a`、`b` 等另外 2 项 准备或更新状态。 调用 `ttgl.BlockedLayout`、`ttgl.num_warps`、`ttgl.DotOperandLayout`、`ttgl.arange`、`ttgl.SliceLayout`、`ttgl.convert_layout` 等另外 3 项 执行测试逻辑。 相关主题：布局变换推理、随机数据生成。

#### Lines 2182-2188

```python

    a = torch.rand((B, B), dtype=torch.float32, device="cuda")
    b = torch.rand((B, B), dtype=torch.float32, device="cuda")
    c = torch.rand((B, B), dtype=torch.float32, device="cuda")
    out = torch.empty((B, B), dtype=torch.float32, device="cuda")
    kernel[(1, )](a, b, c, out)
    torch.testing.assert_close(out, torch.addmm(c, a, b), atol=1e-2, rtol=1e-2)
```
- **EN:** Prepares or updates state through `a`, `b`, `c`, `out`. Invokes `torch.rand`, `torch.empty`, `torch.testing.assert_close`, `torch.addmm` to execute the test logic. Relevant themes: PyTorch tensor setup and checks, random-data generation.
- **CN:** 通过 `a`、`b`、`c`、`out` 准备或更新状态。 调用 `torch.rand`、`torch.empty`、`torch.testing.assert_close`、`torch.addmm` 执行测试逻辑。 相关主题：PyTorch 张量准备与校验、随机数据生成。

### Lines 2189-2191

```python


def test_dot3d_fma():
```
- **EN:** Defines the test function `test_dot3d_fma`. Nested definitions in this scope: `kernel`. Key calls include `torch.manual_seed`, `ttgl.constexpr`, `torch.rand`, `torch.empty`, `torch.testing.assert_close`, `ttgl.BlockedLayout`, and 9 more. This scope touches PyTorch tensor setup and checks, matrix multiplication workflows, layout transformation reasoning, random-data generation.
- **CN:** 定义测试函数 `test_dot3d_fma`。 该作用域中的嵌套定义：`kernel`。 关键调用包括 `torch.manual_seed`、`ttgl.constexpr`、`torch.rand`、`torch.empty`、`torch.testing.assert_close`、`ttgl.BlockedLayout` 等另外 9 项。 该作用域涉及PyTorch 张量准备与校验、矩阵乘法工作流、布局变换推理、随机数据生成。

#### Lines 2192-2195

```python
    torch.manual_seed(42)
    B = ttgl.constexpr(32)
    BATCH = ttgl.constexpr(8)
    threads_per_warp = ttgl.constexpr(THREADS_PER_WARP)
```
- **EN:** Prepares or updates state through `B`, `BATCH`, `threads_per_warp`. Invokes `torch.manual_seed`, `ttgl.constexpr` to execute the test logic. Relevant themes: PyTorch tensor setup and checks, random-data generation.
- **CN:** 通过 `B`、`BATCH`、`threads_per_warp` 准备或更新状态。 调用 `torch.manual_seed`、`ttgl.constexpr` 执行测试逻辑。 相关主题：PyTorch 张量准备与校验、随机数据生成。

#### Lines 2196-2198

```python

    @gluon.jit
    def kernel(a_ptr, b_ptr, c_ptr, out_ptr):
```
- **EN:** Defines the helper function `kernel`. Decorators: `gluon.jit`. Parameters: `a_ptr`, `b_ptr`, `c_ptr`, `out_ptr`. Key calls include `ttgl.BlockedLayout`, `ttgl.DotOperandLayout`, `ttgl.convert_layout`, `ttgl.load`, `ttgl.dot_fma`, `ttgl.store`, and 3 more. This scope touches layout transformation reasoning, random-data generation.
- **CN:** 定义辅助函数 `kernel`。 装饰器：`gluon.jit`。 参数：`a_ptr`、`b_ptr`、`c_ptr`、`out_ptr`。 关键调用包括 `ttgl.BlockedLayout`、`ttgl.DotOperandLayout`、`ttgl.convert_layout`、`ttgl.load`、`ttgl.dot_fma`、`ttgl.store` 等另外 3 项。 该作用域涉及布局变换推理、随机数据生成。

##### Lines 2199-2212

```python
        layout: ttgl.constexpr = ttgl.BlockedLayout([1, 1, 1], [1, threads_per_warp, 1], [ttgl.num_warps(), 1, 1],
                                                    [2, 1, 0])
        lhs_layout: ttgl.constexpr = ttgl.DotOperandLayout(parent=layout, operand_index=0, k_width=0)
        rhs_layout: ttgl.constexpr = ttgl.DotOperandLayout(parent=layout, operand_index=1, k_width=0)

        offs_b = ttgl.arange(0, BATCH, layout=ttgl.SliceLayout(1, ttgl.SliceLayout(2, layout)))[:, None, None]
        offs_m = ttgl.arange(0, B, layout=ttgl.SliceLayout(0, ttgl.SliceLayout(2, layout)))[None, :, None]
        offs_n = ttgl.arange(0, B, layout=ttgl.SliceLayout(0, ttgl.SliceLayout(1, layout)))[None, None, :]
        offs = offs_b * B * B + offs_m * B + offs_n
        a = ttgl.convert_layout(ttgl.load(a_ptr + offs), lhs_layout)
        b = ttgl.convert_layout(ttgl.load(b_ptr + offs), rhs_layout)
        c = ttgl.load(c_ptr + offs)
        out = ttgl.dot_fma(a, b, c)
        ttgl.store(out_ptr + offs, out)
```
- **EN:** Prepares or updates state through `layout`, `lhs_layout`, `rhs_layout`, `offs_b`, `offs_m`, `offs_n`, `offs`, `a`, and 3 more. Invokes `ttgl.BlockedLayout`, `ttgl.num_warps`, `ttgl.DotOperandLayout`, `ttgl.arange`, `ttgl.SliceLayout`, `ttgl.convert_layout`, and 3 more to execute the test logic. Relevant themes: layout transformation reasoning, random-data generation.
- **CN:** 通过 `layout`、`lhs_layout`、`rhs_layout`、`offs_b`、`offs_m`、`offs_n`、`offs`、`a` 等另外 3 项 准备或更新状态。 调用 `ttgl.BlockedLayout`、`ttgl.num_warps`、`ttgl.DotOperandLayout`、`ttgl.arange`、`ttgl.SliceLayout`、`ttgl.convert_layout` 等另外 3 项 执行测试逻辑。 相关主题：布局变换推理、随机数据生成。

#### Lines 2213-2219

```python

    a = torch.rand((BATCH, B, B), dtype=torch.float32, device="cuda")
    b = torch.rand((BATCH, B, B), dtype=torch.float32, device="cuda")
    c = torch.rand((BATCH, B, B), dtype=torch.float32, device="cuda")
    out = torch.empty((BATCH, B, B), dtype=torch.float32, device="cuda")
    kernel[(1, )](a, b, c, out)
    torch.testing.assert_close(out, torch.matmul(a, b) + c, atol=1e-2, rtol=1e-2)
```
- **EN:** Prepares or updates state through `a`, `b`, `c`, `out`. Invokes `torch.rand`, `torch.empty`, `torch.testing.assert_close`, `torch.matmul` to execute the test logic. Relevant themes: PyTorch tensor setup and checks, matrix multiplication workflows, random-data generation.
- **CN:** 通过 `a`、`b`、`c`、`out` 准备或更新状态。 调用 `torch.rand`、`torch.empty`、`torch.testing.assert_close`、`torch.matmul` 执行测试逻辑。 相关主题：PyTorch 张量准备与校验、矩阵乘法工作流、随机数据生成。

### Lines 2220-2223

```python


@gluon.jit
def kernel_auto_layout_constant(threads_per_warp: ttgl.constexpr):
```
- **EN:** Defines the helper function `kernel_auto_layout_constant`. Decorators: `gluon.jit`. Parameters: `threads_per_warp`. Key calls include `ttgl.full`, `ttgl.BlockedLayout`, `ttgl.arange`, `ttgl.AutoLayout`. This scope touches layout transformation reasoning.
- **CN:** 定义辅助函数 `kernel_auto_layout_constant`。 装饰器：`gluon.jit`。 参数：`threads_per_warp`。 关键调用包括 `ttgl.full`、`ttgl.BlockedLayout`、`ttgl.arange`、`ttgl.AutoLayout`。 该作用域涉及布局变换推理。

#### Lines 2224-2240

```python
    BLOCK: ttgl.constexpr = 16
    SIZE: ttgl.constexpr = 10

    mask = ttgl.full(
        (BLOCK, BLOCK),
        True,
        ttgl.int1,
        ttgl.BlockedLayout(
            size_per_thread=[1, 1],
            threads_per_warp=[1, threads_per_warp],
            warps_per_cta=[1, 4],
            order=[1, 0],
        ),
    )

    mask &= (ttgl.arange(0, BLOCK, ttgl.AutoLayout()) < SIZE).expand_dims(0)
    mask &= (ttgl.arange(0, BLOCK, ttgl.AutoLayout()) < SIZE).expand_dims(1)
```
- **EN:** Prepares or updates state through `BLOCK`, `SIZE`, `mask`. Invokes `ttgl.full`, `ttgl.BlockedLayout`, `ttgl.arange`, `ttgl.AutoLayout` to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `BLOCK`、`SIZE`、`mask` 准备或更新状态。 调用 `ttgl.full`、`ttgl.BlockedLayout`、`ttgl.arange`、`ttgl.AutoLayout` 执行测试逻辑。 相关主题：布局变换推理。

### Lines 2241-2243

```python


def test_auto_layout_constant():
```
- **EN:** Defines the test function `test_auto_layout_constant`. Key calls include `kernel_auto_layout_constant.warmup`. This scope touches layout transformation reasoning.
- **CN:** 定义测试函数 `test_auto_layout_constant`。 关键调用包括 `kernel_auto_layout_constant.warmup`。 该作用域涉及布局变换推理。

#### Lines 2244-2244

```python
    kernel_auto_layout_constant.warmup(THREADS_PER_WARP, grid=(1, ))
```
- **EN:** Invokes `kernel_auto_layout_constant.warmup` to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 调用 `kernel_auto_layout_constant.warmup` 执行测试逻辑。 相关主题：布局变换推理。

### Lines 2245-2247

```python


def test_split_auto_layout_execution():
```
- **EN:** Defines the test function `test_split_auto_layout_execution`. Nested definitions in this scope: `kernel`. Key calls include `ttgl.constexpr`, `torch.randint`, `torch.empty`, `torch.testing.assert_close`, `ttgl.arange`, `x.split`, and 5 more. This scope touches PyTorch tensor setup and checks, layout transformation reasoning, random-data generation.
- **CN:** 定义测试函数 `test_split_auto_layout_execution`。 该作用域中的嵌套定义：`kernel`。 关键调用包括 `ttgl.constexpr`、`torch.randint`、`torch.empty`、`torch.testing.assert_close`、`ttgl.arange`、`x.split` 等另外 5 项。 该作用域涉及PyTorch 张量准备与校验、布局变换推理、随机数据生成。

#### Lines 2248-2248

```python
    threads_per_warp = ttgl.constexpr(THREADS_PER_WARP)
```
- **EN:** Prepares or updates state through `threads_per_warp`. Invokes `ttgl.constexpr` to execute the test logic.
- **CN:** 通过 `threads_per_warp` 准备或更新状态。 调用 `ttgl.constexpr` 执行测试逻辑。

#### Lines 2249-2251

```python

    @gluon.jit
    def kernel(in_ptr, out_ptr, XBLOCK: ttgl.constexpr):
```
- **EN:** Defines the helper function `kernel`. Decorators: `gluon.jit`. Parameters: `in_ptr`, `out_ptr`, `XBLOCK`. Key calls include `ttgl.arange`, `x.split`, `ttgl.BlockedLayout`, `ttgl.set_auto_layout`, `ttgl.store`, `ttgl.load`. This scope touches layout transformation reasoning.
- **CN:** 定义辅助函数 `kernel`。 装饰器：`gluon.jit`。 参数：`in_ptr`、`out_ptr`、`XBLOCK`。 关键调用包括 `ttgl.arange`、`x.split`、`ttgl.BlockedLayout`、`ttgl.set_auto_layout`、`ttgl.store`、`ttgl.load`。 该作用域涉及布局变换推理。

##### Lines 2252-2259

```python
        in_offsets = ttgl.arange(0, 2 * XBLOCK)
        x = ttgl.load(in_ptr + in_offsets).reshape((XBLOCK, 2))
        lhs, rhs = x.split()

        out_layout: ttgl.constexpr = ttgl.BlockedLayout([1], [threads_per_warp], [4], [0])
        diff = ttgl.set_auto_layout(rhs - lhs, out_layout)
        out_offsets = ttgl.arange(0, XBLOCK)
        ttgl.store(out_ptr + out_offsets, diff)
```
- **EN:** Prepares or updates state through `in_offsets`, `x`, `lhs`, `rhs`, `out_layout`, `diff`, `out_offsets`. Invokes `ttgl.arange`, `ttgl.load`, `x.split`, `ttgl.BlockedLayout`, `ttgl.set_auto_layout`, `ttgl.store` to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `in_offsets`、`x`、`lhs`、`rhs`、`out_layout`、`diff`、`out_offsets` 准备或更新状态。 调用 `ttgl.arange`、`ttgl.load`、`x.split`、`ttgl.BlockedLayout`、`ttgl.set_auto_layout`、`ttgl.store` 执行测试逻辑。 相关主题：布局变换推理。

#### Lines 2260-2267

```python

    XBLOCK = 128
    input = torch.randint(-100, 100, (2 * XBLOCK, ), device="cuda", dtype=torch.int32)
    output = torch.empty(XBLOCK, device="cuda", dtype=torch.int32)
    ref = input.view(XBLOCK, 2)[:, 1] - input.view(XBLOCK, 2)[:, 0]

    kernel[(1, )](input, output, XBLOCK, num_warps=4)
    torch.testing.assert_close(output, ref)
```
- **EN:** Prepares or updates state through `XBLOCK`, `input`, `output`, `ref`. Invokes `torch.randint`, `torch.empty`, `input.view`, `torch.testing.assert_close` to execute the test logic. Relevant themes: PyTorch tensor setup and checks, random-data generation.
- **CN:** 通过 `XBLOCK`、`input`、`output`、`ref` 准备或更新状态。 调用 `torch.randint`、`torch.empty`、`input.view`、`torch.testing.assert_close` 执行测试逻辑。 相关主题：PyTorch 张量准备与校验、随机数据生成。

### Lines 2268-2270

```python


def fp8e8m0_to_float32(scale):
```
- **EN:** Defines the helper function `fp8e8m0_to_float32`. Parameters: `scale`. Key calls include `scale.view`, `scale.to`. This scope touches PyTorch tensor setup and checks.
- **CN:** 定义辅助函数 `fp8e8m0_to_float32`。 参数：`scale`。 关键调用包括 `scale.view`、`scale.to`。 该作用域涉及PyTorch 张量准备与校验。

#### Lines 2271-2275

```python
    scale = scale.view(torch.uint8)
    scale = scale.to(torch.int32)
    scale = scale << 23
    scale = scale.view(torch.float32)
    return scale
```
- **EN:** Prepares or updates state through `scale`. Invokes `scale.view`, `scale.to` to execute the test logic. Relevant themes: PyTorch tensor setup and checks.
- **CN:** 通过 `scale` 准备或更新状态。 调用 `scale.view`、`scale.to` 执行测试逻辑。 相关主题：PyTorch 张量准备与校验。

### Lines 2276-2279

```python


@pytest.mark.skipif(not is_blackwell(), reason="Requires Blackwell")
def test_tcgen05_mma_scaled_minimal():
```
- **EN:** Defines the test function `test_tcgen05_mma_scaled_minimal`. Decorators: `pytest.mark.skipif(not is_blackwell(), reason='Requires Blackwell')`. Nested definitions in this scope: `kernel`. Key calls include `pytest.mark.skipif`, `ttgl.constexpr`, `torch.empty`, `torch.randint`, `a.to`, `b.to`, and 31 more. This scope touches PyTorch tensor setup and checks, matrix multiplication workflows, layout transformation reasoning, random-data generation.
- **CN:** 定义测试函数 `test_tcgen05_mma_scaled_minimal`。 装饰器：`pytest.mark.skipif(not is_blackwell(), reason='Requires Blackwell')`。 该作用域中的嵌套定义：`kernel`。 关键调用包括 `pytest.mark.skipif`、`ttgl.constexpr`、`torch.empty`、`torch.randint`、`a.to`、`b.to` 等另外 31 项。 该作用域涉及PyTorch 张量准备与校验、矩阵乘法工作流、布局变换推理、随机数据生成。

#### Lines 2280-2283

```python
    M = 128
    N = 128
    K = 128
    threads_per_warp = ttgl.constexpr(THREADS_PER_WARP)
```
- **EN:** Prepares or updates state through `M`, `N`, `K`, `threads_per_warp`. Invokes `ttgl.constexpr` to execute the test logic.
- **CN:** 通过 `M`、`N`、`K`、`threads_per_warp` 准备或更新状态。 调用 `ttgl.constexpr` 执行测试逻辑。

#### Lines 2284-2287

```python

    @gluon.jit
    def kernel(out_ptr, M: ttgl.constexpr, N: ttgl.constexpr, K: ttgl.constexpr, a, b, a_scale, b_scale):
        # Simple register layout for creating constants and storing results
```
- **EN:** Defines the helper function `kernel`. Decorators: `gluon.jit`. Parameters: `out_ptr`, `M`, `N`, `K`, `a`, `b`, `a_scale`, `b_scale`. Key calls include `ttgl.BlockedLayout`, `ttgl.NVMMASharedLayout`, `ttgl.load`, `ttgl.allocate_shared_memory`, `TensorMemoryLayout`, `allocate_tensor_memory`, and 18 more. This scope touches layout transformation reasoning.
- **CN:** 定义辅助函数 `kernel`。 装饰器：`gluon.jit`。 参数：`out_ptr`、`M`、`N`、`K`、`a`、`b`、`a_scale`、`b_scale`。 关键调用包括 `ttgl.BlockedLayout`、`ttgl.NVMMASharedLayout`、`ttgl.load`、`ttgl.allocate_shared_memory`、`TensorMemoryLayout`、`allocate_tensor_memory` 等另外 18 项。 该作用域涉及布局变换推理。

##### Lines 2288-2340

```python
        reg_layout: ttgl.constexpr = ttgl.BlockedLayout([1, 1], [threads_per_warp, 1], [ttgl.num_warps(), 1], [1, 0])

        # Shared-memory layouts for MMA operands
        nvmma_layout: ttgl.constexpr = ttgl.NVMMASharedLayout(swizzle_byte_width=128, transposed=False,
                                                              element_bitwidth=8, rank=2)
        # Allocate zero operands in shared memory (values don't matter since scales are zero)
        block_layout: ttgl.constexpr = ttgl.BlockedLayout([1, 1], [1, 32], warps_per_cta=[ttgl.num_warps(), 1],
                                                          order=[1, 0])
        a_offs_m = ttgl.arange(0, M, layout=ttgl.SliceLayout(1, block_layout))[:, None]
        a_offs_k = ttgl.arange(0, K, layout=ttgl.SliceLayout(0, block_layout))[None, :]
        b_offs_k = ttgl.arange(0, K, layout=ttgl.SliceLayout(1, block_layout))[:, None]
        b_offs_n = ttgl.arange(0, N, layout=ttgl.SliceLayout(0, block_layout))[None, :]

        a_tile = ttgl.load(a + a_offs_m * K + a_offs_k)
        b_tile = ttgl.load(b + b_offs_k * N + b_offs_n)
        a_smem = ttgl.allocate_shared_memory(ttgl.float8e5, [M, K], nvmma_layout, a_tile)
        b_smem = ttgl.allocate_shared_memory(ttgl.float8e5, [K, N], nvmma_layout, b_tile)

        # Accumulator in TMEM initialized to ones
        acc_tmem_layout: ttgl.constexpr = TensorMemoryLayout([M, N], col_stride=1)
        acc_tmem = allocate_tensor_memory(ttgl.float32, [M, N], acc_tmem_layout)
        tmem_reg_layout: ttgl.constexpr = acc_tmem.get_reg_layout()
        acc_init = ttgl.zeros([M, N], ttgl.float32, layout=tmem_reg_layout)
        acc_tmem.store(acc_init)

        # Zero scales in TMEM
        scale_layout: ttgl.constexpr = TensorMemoryScalesLayout()
        a_scale_tmem = allocate_tensor_memory(a_scale.dtype.element_ty, [M, K // 32], scale_layout)
        b_scale_tmem = allocate_tensor_memory(b_scale.dtype.element_ty, [N, K // 32], scale_layout)
        scale_reg_layout_m: ttgl.constexpr = a_scale_tmem.get_reg_layout()
        scale_reg_layout_n: ttgl.constexpr = b_scale_tmem.get_reg_layout()
        scale_offs_k = ttgl.arange(0, (K // 32), layout=ttgl.SliceLayout(0, scale_reg_layout_m))[None, :]
        scale_offs_m = ttgl.arange(0, M, layout=ttgl.SliceLayout(1, scale_reg_layout_m))[:, None]
        scale_offs_n = ttgl.arange(0, N, layout=ttgl.SliceLayout(1, scale_reg_layout_n))[:, None]
        a_scale_init = ttgl.load(a_scale + scale_offs_m * (K // 32) + scale_offs_k)
        b_scale_init = ttgl.load(b_scale + scale_offs_n * (K // 32) + scale_offs_k)
        a_scale_tmem.store(a_scale_init)
        b_scale_tmem.store(b_scale_init)

        # Issue a single scaled MMA with a completion barrier.
        bar = ttgl.allocate_shared_memory(ttgl.int64, [1], mbarrier.MBarrierLayout())
        mbarrier.init(bar, count=1)
        tcgen05_mma_scaled(a_smem, b_smem, acc_tmem, a_scale_tmem, b_scale_tmem, "e5m2", "e5m2", use_acc=True,
                           mbarriers=[bar])
        mbarrier.wait(bar, phase=0)

        # Load result from TMEM and store to global
        out_reg = acc_tmem.load()
        store_layout: ttgl.constexpr = reg_layout
        offs_m = ttgl.arange(0, M, layout=ttgl.SliceLayout(1, store_layout))[:, None]
        offs_n = ttgl.arange(0, N, layout=ttgl.SliceLayout(0, store_layout))[None, :]
        offs = offs_m * N + offs_n
        ttgl.store(out_ptr + offs, ttgl.convert_layout(out_reg, store_layout))
```
- **EN:** Prepares or updates state through `reg_layout`, `nvmma_layout`, `block_layout`, `a_offs_m`, `a_offs_k`, `b_offs_k`, `b_offs_n`, `a_tile`, and 23 more. Invokes `ttgl.BlockedLayout`, `ttgl.num_warps`, `ttgl.NVMMASharedLayout`, `ttgl.arange`, `ttgl.SliceLayout`, `ttgl.load`, and 18 more to execute the test logic. Relevant themes: layout transformation reasoning, random-data generation.
- **CN:** 通过 `reg_layout`、`nvmma_layout`、`block_layout`、`a_offs_m`、`a_offs_k`、`b_offs_k`、`b_offs_n`、`a_tile` 等另外 23 项 准备或更新状态。 调用 `ttgl.BlockedLayout`、`ttgl.num_warps`、`ttgl.NVMMASharedLayout`、`ttgl.arange`、`ttgl.SliceLayout`、`ttgl.load` 等另外 18 项 执行测试逻辑。 相关主题：布局变换推理、随机数据生成。

#### Lines 2341-2359

```python

    out = torch.empty((M, N), dtype=torch.float32, device="cuda")
    a = torch.randint(20, 40, (M, K), dtype=torch.uint8, device="cuda").view(torch.float8_e5m2)
    b = torch.randint(20, 40, (K, N), dtype=torch.uint8, device="cuda").view(torch.float8_e5m2)
    a_scale = torch.randint(64, 130, (M, K // 32), dtype=torch.uint8, device="cuda")
    b_scale = torch.randint(64, 130, (N, K // 32), dtype=torch.uint8, device="cuda")
    compiled = kernel[(1, )](out, M, N, K, a, b, a_scale, b_scale)
    A = a.to(torch.float32)
    B = b.to(torch.float32)
    a_scale_f32 = fp8e8m0_to_float32(a_scale)
    b_scale_f32 = fp8e8m0_to_float32(b_scale)
    a_scale_f32 = a_scale_f32.repeat_interleave(32, dim=1)
    b_scale_f32 = b_scale_f32.repeat_interleave(32, dim=1)
    b_scale_f32 = b_scale_f32.T.contiguous()
    A = A * a_scale_f32
    B = B * b_scale_f32
    ref = torch.matmul(A, B)
    torch.testing.assert_close(out, ref, atol=1e-6, rtol=1e-6)
    ttgir = compiled.asm["ttgir"]
```
- **EN:** Prepares or updates state through `out`, `a`, `b`, `a_scale`, `b_scale`, `compiled`, `A`, `B`, and 4 more. Invokes `torch.empty`, `torch.randint`, `a.to`, `b.to`, `fp8e8m0_to_float32`, `a_scale_f32.repeat_interleave`, and 4 more to execute the test logic. Relevant themes: PyTorch tensor setup and checks, matrix multiplication workflows, random-data generation.
- **CN:** 通过 `out`、`a`、`b`、`a_scale`、`b_scale`、`compiled`、`A`、`B` 等另外 4 项 准备或更新状态。 调用 `torch.empty`、`torch.randint`、`a.to`、`b.to`、`fp8e8m0_to_float32`、`a_scale_f32.repeat_interleave` 等另外 4 项 执行测试逻辑。 相关主题：PyTorch 张量准备与校验、矩阵乘法工作流、随机数据生成。

#### Lines 2360-2360

```python
    assert "ttng.tc_gen5_mma_scaled" in ttgir
```
- **EN:** Validates behavior with 1 assertion(s).
- **CN:** 通过 1 个断言验证行为。

### Lines 2361-2366

```python


@pytest.mark.skipif(not is_ampere_or_newer(), reason="Requires Ampere or newer")
def test_coalesced_layout():

    @gluon.jit
```
- **EN:** Defines the test function `test_coalesced_layout`. Decorators: `pytest.mark.skipif(not is_ampere_or_newer(), reason='Requires Ampere or newer')`. Nested definitions in this scope: `kernel`. Key calls include `pytest.mark.skipif`, `torch.randn`, `torch.zeros_like`, `torch.maximum`, `torch.testing.assert_close`, `ttgl.program_id`, and 12 more. This scope touches PyTorch tensor setup and checks, layout transformation reasoning, random-data generation.
- **CN:** 定义测试函数 `test_coalesced_layout`。 装饰器：`pytest.mark.skipif(not is_ampere_or_newer(), reason='Requires Ampere or newer')`。 该作用域中的嵌套定义：`kernel`。 关键调用包括 `pytest.mark.skipif`、`torch.randn`、`torch.zeros_like`、`torch.maximum`、`torch.testing.assert_close`、`ttgl.program_id` 等另外 12 项。 该作用域涉及PyTorch 张量准备与校验、布局变换推理、随机数据生成。

#### Lines 2366-2369

```python
    @gluon.jit
    def kernel(in_ptr, out_ptr,  #
               xnumel, ynumel, xstride_in, ystride_in, xstride_out, ystride_out,  #
               XBLOCK: ttgl.constexpr, YBLOCK: ttgl.constexpr):
```
- **EN:** Defines the helper function `kernel`. Decorators: `gluon.jit`. Parameters: `in_ptr`, `out_ptr`, `xnumel`, `ynumel`, `xstride_in`, `ystride_in`, `xstride_out`, `ystride_out`, and 2 more. Key calls include `ttgl.program_id`, `ttgl.load`, `ttgl.sin`, `ttgl.maximum`, `ttgl.store`, `ttgl.arange`, and 1 more. This scope touches layout transformation reasoning.
- **CN:** 定义辅助函数 `kernel`。 装饰器：`gluon.jit`。 参数：`in_ptr`、`out_ptr`、`xnumel`、`ynumel`、`xstride_in`、`ystride_in`、`xstride_out`、`ystride_out` 等另外 2 项。 关键调用包括 `ttgl.program_id`、`ttgl.load`、`ttgl.sin`、`ttgl.maximum`、`ttgl.store`、`ttgl.arange` 等另外 1 项。 该作用域涉及布局变换推理。

##### Lines 2370-2389

```python
        pid_x = ttgl.program_id(0)
        pid_y = ttgl.program_id(1)
        indices_x = pid_x * XBLOCK + ttgl.arange(0, XBLOCK, ttgl.CoalescedLayout())
        indices_y = pid_y * YBLOCK + ttgl.arange(0, YBLOCK, ttgl.CoalescedLayout())

        in_offsets = xstride_in * indices_x[:, None] + ystride_in * indices_y[None, :]
        out_offsets = xstride_out * indices_x[:, None] + ystride_out * indices_y[None, :]

        # MASK
        mask = (indices_x[:, None] < xnumel) & (indices_y[None, :] < ynumel)

        # IN PTR
        in_ptrs = in_ptr + in_offsets
        value = ttgl.load(in_ptrs, mask=mask)
        value = ttgl.sin(value)
        value = ttgl.maximum(value, 0.0)

        # OUT PTR
        out_ptrs = out_ptr + out_offsets
        ttgl.store(out_ptrs, value, mask=mask)
```
- **EN:** Prepares or updates state through `pid_x`, `pid_y`, `indices_x`, `indices_y`, `in_offsets`, `out_offsets`, `mask`, `in_ptrs`, and 2 more. Invokes `ttgl.program_id`, `ttgl.arange`, `ttgl.CoalescedLayout`, `ttgl.load`, `ttgl.sin`, `ttgl.maximum`, and 1 more to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `pid_x`、`pid_y`、`indices_x`、`indices_y`、`in_offsets`、`out_offsets`、`mask`、`in_ptrs` 等另外 2 项 准备或更新状态。 调用 `ttgl.program_id`、`ttgl.arange`、`ttgl.CoalescedLayout`、`ttgl.load`、`ttgl.sin`、`ttgl.maximum` 等另外 1 项 执行测试逻辑。 相关主题：布局变换推理。

#### Lines 2390-2405

```python

    XBLOCK = 128
    YBLOCK = 256
    xnumel = 1000
    ynumel = 2000
    input = torch.randn((xnumel, ynumel), device="cuda")
    output = torch.zeros_like(input)
    ref = torch.maximum(torch.sin(input), torch.tensor(0.0, device="cuda"))

    grid = (triton.cdiv(xnumel, XBLOCK), triton.cdiv(ynumel, YBLOCK))
    kernel[grid](  #
        input, output, xnumel, ynumel,  #
        *input.stride(), *output.stride(),  #
        XBLOCK, YBLOCK, num_warps=4)

    torch.testing.assert_close(output, ref)
```
- **EN:** Prepares or updates state through `XBLOCK`, `YBLOCK`, `xnumel`, `ynumel`, `input`, `output`, `ref`, `grid`. Invokes `torch.randn`, `torch.zeros_like`, `torch.maximum`, `torch.sin`, `torch.tensor`, `triton.cdiv`, and 3 more to execute the test logic. Relevant themes: PyTorch tensor setup and checks, random-data generation.
- **CN:** 通过 `XBLOCK`、`YBLOCK`、`xnumel`、`ynumel`、`input`、`output`、`ref`、`grid` 准备或更新状态。 调用 `torch.randn`、`torch.zeros_like`、`torch.maximum`、`torch.sin`、`torch.tensor`、`triton.cdiv` 等另外 3 项 执行测试逻辑。 相关主题：PyTorch 张量准备与校验、随机数据生成。

### Lines 2406-2411

```python


@pytest.mark.skipif(not is_ampere_or_newer(), reason="Requires Ampere or newer")
def test_convert_auto_layout_to_coalesced_layout():

    @gluon.jit
```
- **EN:** Defines the test function `test_convert_auto_layout_to_coalesced_layout`. Decorators: `pytest.mark.skipif(not is_ampere_or_newer(), reason='Requires Ampere or newer')`. Nested definitions in this scope: `kernel`. Key calls include `pytest.mark.skipif`, `torch.ones`, `torch.zeros_like`, `torch.ones_like`, `torch.testing.assert_close`, `ttgl.program_id`, and 10 more. This scope touches PyTorch tensor setup and checks, layout transformation reasoning.
- **CN:** 定义测试函数 `test_convert_auto_layout_to_coalesced_layout`。 装饰器：`pytest.mark.skipif(not is_ampere_or_newer(), reason='Requires Ampere or newer')`。 该作用域中的嵌套定义：`kernel`。 关键调用包括 `pytest.mark.skipif`、`torch.ones`、`torch.zeros_like`、`torch.ones_like`、`torch.testing.assert_close`、`ttgl.program_id` 等另外 10 项。 该作用域涉及PyTorch 张量准备与校验、布局变换推理。

#### Lines 2411-2414

```python
    @gluon.jit
    def kernel(in_ptr, out_ptr,  #
               xnumel, ynumel, xstride_in, ystride_in, xstride_out, ystride_out,  #
               XBLOCK: ttgl.constexpr, YBLOCK: ttgl.constexpr):
```
- **EN:** Defines the helper function `kernel`. Decorators: `gluon.jit`. Parameters: `in_ptr`, `out_ptr`, `xnumel`, `ynumel`, `xstride_in`, `ystride_in`, `xstride_out`, `ystride_out`, and 2 more. Key calls include `ttgl.program_id`, `ttgl.set_auto_layout`, `ttgl.load`, `ttgl.store`, `ttgl.arange`, `ttgl.CoalescedLayout`, and 1 more. This scope touches layout transformation reasoning.
- **CN:** 定义辅助函数 `kernel`。 装饰器：`gluon.jit`。 参数：`in_ptr`、`out_ptr`、`xnumel`、`ynumel`、`xstride_in`、`ystride_in`、`xstride_out`、`ystride_out` 等另外 2 项。 关键调用包括 `ttgl.program_id`、`ttgl.set_auto_layout`、`ttgl.load`、`ttgl.store`、`ttgl.arange`、`ttgl.CoalescedLayout` 等另外 1 项。 该作用域涉及布局变换推理。

##### Lines 2415-2433

```python
        pid_x = ttgl.program_id(0)
        pid_y = ttgl.program_id(1)
        indices_x = pid_x * XBLOCK + ttgl.arange(0, XBLOCK, ttgl.AutoLayout())
        indices_y = pid_y * YBLOCK + ttgl.arange(0, YBLOCK, ttgl.AutoLayout())

        in_offsets = xstride_in * indices_x[:, None] + ystride_in * indices_y[None, :]
        out_offsets = xstride_out * indices_x[:, None] + ystride_out * indices_y[None, :]

        # MASK
        mask = (indices_x[:, None] < xnumel) & (indices_y[None, :] < ynumel)  # auto layout

        # IN PTR
        in_ptrs = ttgl.set_auto_layout(in_ptr + in_offsets, ttgl.CoalescedLayout())
        value = ttgl.load(in_ptrs, mask=mask)

        # OUT PTR
        out_ptrs = ttgl.set_auto_layout(out_ptr + out_offsets, ttgl.CoalescedLayout())
        out_mask_layouted = ttgl.set_auto_layout(mask, ttgl.CoalescedLayout())
        ttgl.store(out_ptrs, value, mask=out_mask_layouted)
```
- **EN:** Prepares or updates state through `pid_x`, `pid_y`, `indices_x`, `indices_y`, `in_offsets`, `out_offsets`, `mask`, `in_ptrs`, and 3 more. Invokes `ttgl.program_id`, `ttgl.arange`, `ttgl.AutoLayout`, `ttgl.set_auto_layout`, `ttgl.CoalescedLayout`, `ttgl.load`, and 1 more to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `pid_x`、`pid_y`、`indices_x`、`indices_y`、`in_offsets`、`out_offsets`、`mask`、`in_ptrs` 等另外 3 项 准备或更新状态。 调用 `ttgl.program_id`、`ttgl.arange`、`ttgl.AutoLayout`、`ttgl.set_auto_layout`、`ttgl.CoalescedLayout`、`ttgl.load` 等另外 1 项 执行测试逻辑。 相关主题：布局变换推理。

#### Lines 2434-2449

```python

    XBLOCK = 128
    YBLOCK = 256
    xnumel = 1000
    ynumel = 2000
    input = torch.ones((xnumel, ynumel), device="cuda")
    output = torch.zeros_like(input)
    ref = torch.ones_like(input)

    grid = (triton.cdiv(xnumel, XBLOCK), triton.cdiv(ynumel, YBLOCK))
    kernel[grid](  #
        input, output, xnumel, ynumel,  #
        *input.stride(), *output.stride(),  #
        XBLOCK, YBLOCK, num_warps=4)

    torch.testing.assert_close(output, ref)
```
- **EN:** Prepares or updates state through `XBLOCK`, `YBLOCK`, `xnumel`, `ynumel`, `input`, `output`, `ref`, `grid`. Invokes `torch.ones`, `torch.zeros_like`, `torch.ones_like`, `triton.cdiv`, `input.stride`, `output.stride`, and 1 more to execute the test logic. Relevant themes: PyTorch tensor setup and checks.
- **CN:** 通过 `XBLOCK`、`YBLOCK`、`xnumel`、`ynumel`、`input`、`output`、`ref`、`grid` 准备或更新状态。 调用 `torch.ones`、`torch.zeros_like`、`torch.ones_like`、`triton.cdiv`、`input.stride`、`output.stride` 等另外 1 项 执行测试逻辑。 相关主题：PyTorch 张量准备与校验。

### Lines 2450-2455

```python


@gluon.jit
def in_thread_transpose_roundtrip_kernel(input, output, M: ttgl.constexpr, N: ttgl.constexpr,
                                         first_layout: ttgl.constexpr, second_layout: ttgl.constexpr,
                                         shared_layout: ttgl.constexpr):
```
- **EN:** Defines the helper function `in_thread_transpose_roundtrip_kernel`. Decorators: `gluon.jit`. Parameters: `input`, `output`, `M`, `N`, `first_layout`, `second_layout`, `shared_layout`. Key calls include `ttgl.load`, `ttgl.convert_layout`, `ttgl.allocate_shared_memory`, `smem.load`, `ttgl.store`, `ttgl.arange`, and 1 more. This scope touches layout transformation reasoning.
- **CN:** 定义辅助函数 `in_thread_transpose_roundtrip_kernel`。 装饰器：`gluon.jit`。 参数：`input`、`output`、`M`、`N`、`first_layout`、`second_layout`、`shared_layout`。 关键调用包括 `ttgl.load`、`ttgl.convert_layout`、`ttgl.allocate_shared_memory`、`smem.load`、`ttgl.store`、`ttgl.arange` 等另外 1 项。 该作用域涉及布局变换推理。

#### Lines 2456-2463

```python
    offs_m = ttgl.arange(0, M, layout=ttgl.SliceLayout(1, first_layout))[:, None]
    offs_n = ttgl.arange(0, N, layout=ttgl.SliceLayout(0, first_layout))[None, :]

    load_data = ttgl.load(input + offs_m * N + offs_n)
    converted_data = ttgl.convert_layout(load_data, second_layout)
    smem = ttgl.allocate_shared_memory(input.dtype.element_ty, [M, N], shared_layout, converted_data)
    out_data = smem.load(first_layout)
    ttgl.store(output + offs_m * N + offs_n, out_data)
```
- **EN:** Prepares or updates state through `offs_m`, `offs_n`, `load_data`, `converted_data`, `smem`, `out_data`. Invokes `ttgl.arange`, `ttgl.SliceLayout`, `ttgl.load`, `ttgl.convert_layout`, `ttgl.allocate_shared_memory`, `smem.load`, and 1 more to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `offs_m`、`offs_n`、`load_data`、`converted_data`、`smem`、`out_data` 准备或更新状态。 调用 `ttgl.arange`、`ttgl.SliceLayout`、`ttgl.load`、`ttgl.convert_layout`、`ttgl.allocate_shared_memory`、`smem.load` 等另外 1 项 执行测试逻辑。 相关主题：布局变换推理。

### Lines 2464-2479

```python


@pytest.mark.skipif(not (is_hip_cdna() or is_hip_rdna()),
                    reason="Correctness tests for special cases on AMD architectures")
@pytest.mark.parametrize("src_reg_bases, dst_reg_bases", [
    ([[0, 1], [0, 2], [0, 4], [1, 0], [2, 0], [4, 0]], [[1, 0], [2, 0], [4, 0], [0, 1], [0, 2], [0, 4]]),
    ([[0, 1], [0, 2], [0, 4], [1, 0], [2, 0], [4, 0]], [[0, 1], [0, 4], [0, 2], [1, 0], [2, 0], [4, 0]]),
    ([[0, 1], [0, 2], [0, 4], [1, 0], [2, 0], [4, 0]], [[0, 2], [0, 1], [0, 4], [1, 0], [4, 0], [2, 0]]),
    ([[0, 0], [0, 1], [0, 2], [0, 4], [1, 0], [2, 0], [4, 0]], [[0, 2], [0, 1], [0, 4], [1, 0], [4, 0], [2, 0]]),
    ([[0, 1], [0, 2], [0, 4], [1, 0], [2, 0], [4, 0]], [[0, 2], [0, 0], [0, 1], [0, 4], [1, 0], [4, 0], [2, 0]]),
    ([[0, 1], [0, 0], [0, 2], [0, 4], [1, 0], [2, 0], [4, 0]], [[0, 0], [0, 2], [0, 1], [0, 4], [1, 0], [4, 0], [2, 0]
                                                                ]),
    ([[0, 1], [0, 2], [0, 4], [0, 0], [1, 0], [2, 0], [4, 0]], [[0, 2], [0, 1], [0, 0], [0, 4], [1, 0], [4, 0], [2, 0]
                                                                ]),
])
def test_in_thread_convert_layout_8bit(src_reg_bases, dst_reg_bases):
```
- **EN:** Defines the test function `test_in_thread_convert_layout_8bit`. Decorators: `pytest.mark.skipif(not (is_hip_cdna() or is_hip_rdna()), reason='Correctness tests for special cases on AMD architectures')`, `pytest.mark.parametrize('src_reg_bases, dst_reg_bases', [([[0, 1], [0, 2], [0, 4], [1, 0], [2, 0], [4, 0]], [[1, 0], [2, 0], [4, 0], [0, 1], [0, 2], [0, 4]]), ([[0, 1], [0, 2], [0, 4], [1, 0], [2, 0], [4, 0]], [[0, 1], [0, 4], [0, 2], [1, 0], [2, 0], [4, 0]]), ([[0, 1], [0, 2], [0, 4], [1, 0], [2, 0], [4, 0]], [[0, 2], [0, 1], [0, 4], [1, 0], [4, 0], [2, 0]]), ([[0, 0], [0, 1], [0, 2], [0, 4], [1, 0], [2, 0], [4, 0]], [[0, 2], [0, 1], [0, 4], [1, 0], [4, 0], [2, 0]]), ([[0, 1], [0, 2], [0, 4], [1, 0], [2, 0], [4, 0]], [[0, 2], [0, 0], [0, 1], [0, 4], [1, 0], [4, 0], [2, 0]]), ([[0, 1], [0, 0], [0, 2], [0, 4], [1, 0], [2, 0], [4, 0]], [[0, 0], [0, 2], [0, 1], [0, 4], [1, 0], [4, 0], [2, 0]]), ([[0, 1], [0, 2], [0, 4], [0, 0], [1, 0], [2, 0], [4, 0]], [[0, 2], [0, 1], [0, 0], [0, 4], [1, 0], [4, 0], [2, 0]])])`. Parameters: `src_reg_bases`, `dst_reg_bases`. Key calls include `pytest.mark.skipif`, `pytest.mark.parametrize`, `torch.manual_seed`, `ttgl.DistributedLinearLayout`, `ttgl.SwizzledSharedLayout`, `torch.zeros`, and 6 more. This scope touches pytest parametrization, PyTorch tensor setup and checks, layout transformation reasoning, random-data generation.
- **CN:** 定义测试函数 `test_in_thread_convert_layout_8bit`。 装饰器：`pytest.mark.skipif(not (is_hip_cdna() or is_hip_rdna()), reason='Correctness tests for special cases on AMD architectures')`、`pytest.mark.parametrize('src_reg_bases, dst_reg_bases', [([[0, 1], [0, 2], [0, 4], [1, 0], [2, 0], [4, 0]], [[1, 0], [2, 0], [4, 0], [0, 1], [0, 2], [0, 4]]), ([[0, 1], [0, 2], [0, 4], [1, 0], [2, 0], [4, 0]], [[0, 1], [0, 4], [0, 2], [1, 0], [2, 0], [4, 0]]), ([[0, 1], [0, 2], [0, 4], [1, 0], [2, 0], [4, 0]], [[0, 2], [0, 1], [0, 4], [1, 0], [4, 0], [2, 0]]), ([[0, 0], [0, 1], [0, 2], [0, 4], [1, 0], [2, 0], [4, 0]], [[0, 2], [0, 1], [0, 4], [1, 0], [4, 0], [2, 0]]), ([[0, 1], [0, 2], [0, 4], [1, 0], [2, 0], [4, 0]], [[0, 2], [0, 0], [0, 1], [0, 4], [1, 0], [4, 0], [2, 0]]), ([[0, 1], [0, 0], [0, 2], [0, 4], [1, 0], [2, 0], [4, 0]], [[0, 0], [0, 2], [0, 1], [0, 4], [1, 0], [4, 0], [2, 0]]), ([[0, 1], [0, 2], [0, 4], [0, 0], [1, 0], [2, 0], [4, 0]], [[0, 2], [0, 1], [0, 0], [0, 4], [1, 0], [4, 0], [2, 0]])])`。 参数：`src_reg_bases`、`dst_reg_bases`。 关键调用包括 `pytest.mark.skipif`、`pytest.mark.parametrize`、`torch.manual_seed`、`ttgl.DistributedLinearLayout`、`ttgl.SwizzledSharedLayout`、`torch.zeros` 等另外 6 项。 该作用域涉及pytest 参数化、PyTorch 张量准备与校验、布局变换推理、随机数据生成。

#### Lines 2480-2498

```python
    torch.manual_seed(0)
    dtype = torch.int8
    M = 8
    N = 8 * THREADS_PER_WARP

    numLaneBases = int(math.log2(THREADS_PER_WARP))
    lane_bases = [[0, 8 * (2**baseNo)] for baseNo in range(numLaneBases)]
    warp_bases = []
    first_layout = ttgl.DistributedLinearLayout(reg_bases=src_reg_bases, lane_bases=lane_bases, warp_bases=warp_bases,
                                                block_bases=[], shape=[M, N])

    second_layout = ttgl.DistributedLinearLayout(reg_bases=dst_reg_bases, lane_bases=lane_bases, warp_bases=warp_bases,
                                                 block_bases=[], shape=[M, N])

    shared_layout = ttgl.SwizzledSharedLayout(1, 1, 1, order=[0, 1])
    input_buffer = (torch.randn((M, N), device="cuda") * 100).to(dtype)
    output_buffer = torch.zeros((M, N), device="cuda", dtype=dtype)
    pgm = in_thread_transpose_roundtrip_kernel[(1, )](input_buffer, output_buffer, M, N, first_layout, second_layout,
                                                      shared_layout, num_warps=1)
```
- **EN:** Prepares or updates state through `dtype`, `M`, `N`, `numLaneBases`, `lane_bases`, `warp_bases`, `first_layout`, `second_layout`, and 4 more. Invokes `torch.manual_seed`, `math.log2`, `ttgl.DistributedLinearLayout`, `ttgl.SwizzledSharedLayout`, `torch.randn`, `torch.zeros` to execute the test logic. Relevant themes: PyTorch tensor setup and checks, layout transformation reasoning, random-data generation.
- **CN:** 通过 `dtype`、`M`、`N`、`numLaneBases`、`lane_bases`、`warp_bases`、`first_layout`、`second_layout` 等另外 4 项 准备或更新状态。 调用 `torch.manual_seed`、`math.log2`、`ttgl.DistributedLinearLayout`、`ttgl.SwizzledSharedLayout`、`torch.randn`、`torch.zeros` 执行测试逻辑。 相关主题：PyTorch 张量准备与校验、布局变换推理、随机数据生成。

#### Lines 2499-2500

```python

    assert re.search(r"v_perm", pgm.asm['amdgcn'], re.MULTILINE)
```
- **EN:** Invokes `re.search` to execute the test logic. Validates behavior with 1 assertion(s).
- **CN:** 调用 `re.search` 执行测试逻辑。 通过 1 个断言验证行为。

#### Lines 2501-2501

```python
    torch.testing.assert_close(input_buffer, output_buffer, atol=1e-3, rtol=1e-3)
```
- **EN:** Invokes `torch.testing.assert_close` to execute the test logic. Relevant themes: PyTorch tensor setup and checks.
- **CN:** 调用 `torch.testing.assert_close` 执行测试逻辑。 相关主题：PyTorch 张量准备与校验。

### Lines 2502-2505

```python


@gluon.jit
def descriptor_shape_kernel(desc, expect_shape):
```
- **EN:** Defines the helper function `descriptor_shape_kernel`. Decorators: `gluon.jit`. Parameters: `desc`, `expect_shape`. Key calls include `ttgl.static_range`, `ttgl.device_assert`. This scope touches tensor/descriptor metadata.
- **CN:** 定义辅助函数 `descriptor_shape_kernel`。 装饰器：`gluon.jit`。 参数：`desc`、`expect_shape`。 关键调用包括 `ttgl.static_range`、`ttgl.device_assert`。 该作用域涉及张量/描述符元数据。

#### Lines 2506-2507

```python
    for i in ttgl.static_range(len(expect_shape)):
        ttgl.device_assert(desc.shape[i] == expect_shape[i])
```
- **EN:** Invokes `ttgl.static_range`, `ttgl.device_assert` to execute the test logic. Iterates across cases or data tiles.
- **CN:** 调用 `ttgl.static_range`、`ttgl.device_assert` 执行测试逻辑。 通过循环覆盖多个用例或数据分块。

### Lines 2508-2511

```python


@pytest.mark.skipif(not is_blackwell(), reason="Requires Blackwell")
def test_descriptor_shape():
```
- **EN:** Defines the test function `test_descriptor_shape`. Decorators: `pytest.mark.skipif(not is_blackwell(), reason='Requires Blackwell')`. Key calls include `pytest.mark.skipif`, `torch.randint`, `ttgl.NVMMASharedLayout.get_default_for`, `TensorDescriptor.from_tensor`, `torch.cuda.synchronize`, `is_blackwell`. This scope touches PyTorch tensor setup and checks, tensor/descriptor metadata, debugging and inspection paths, layout transformation reasoning.
- **CN:** 定义测试函数 `test_descriptor_shape`。 装饰器：`pytest.mark.skipif(not is_blackwell(), reason='Requires Blackwell')`。 关键调用包括 `pytest.mark.skipif`、`torch.randint`、`ttgl.NVMMASharedLayout.get_default_for`、`TensorDescriptor.from_tensor`、`torch.cuda.synchronize`、`is_blackwell`。 该作用域涉及PyTorch 张量准备与校验、张量/描述符元数据、调试与检查路径、布局变换推理。

#### Lines 2512-2512

```python
    t = torch.randint(0, 256, (512, 512), dtype=torch.uint8)
```
- **EN:** Prepares or updates state through `t`. Invokes `torch.randint` to execute the test logic. Relevant themes: PyTorch tensor setup and checks, random-data generation.
- **CN:** 通过 `t` 准备或更新状态。 调用 `torch.randint` 执行测试逻辑。 相关主题：PyTorch 张量准备与校验、随机数据生成。

#### Lines 2513-2518

```python

    for fp4_padded in [True]:
        layout = ttgl.NVMMASharedLayout.get_default_for([128, 64], ttgl.uint8, fp4_padded=fp4_padded)
        desc = TensorDescriptor.from_tensor(t, [128, 64], layout)
        descriptor_shape_kernel[(1, )](desc, t.shape, num_warps=1, debug=True)
        torch.cuda.synchronize()
```
- **EN:** Invokes `ttgl.NVMMASharedLayout.get_default_for`, `TensorDescriptor.from_tensor`, `torch.cuda.synchronize` to execute the test logic. Iterates across cases or data tiles. Relevant themes: PyTorch tensor setup and checks, tensor/descriptor metadata, debugging and inspection paths, layout transformation reasoning.
- **CN:** 调用 `ttgl.NVMMASharedLayout.get_default_for`、`TensorDescriptor.from_tensor`、`torch.cuda.synchronize` 执行测试逻辑。 通过循环覆盖多个用例或数据分块。 相关主题：PyTorch 张量准备与校验、张量/描述符元数据、调试与检查路径、布局变换推理。

### Lines 2519-2531

```python


@gluon.jit
def shared_gather_kernel(
    matrix_ptr,
    indices_ptr,
    output_ptr,
    N: ttgl.constexpr,
    M: ttgl.constexpr,
    layout_2d: ttgl.constexpr,
    layout_1d: ttgl.constexpr,
    shared_layout: ttgl.constexpr,
):
```
- **EN:** Defines the helper function `shared_gather_kernel`. Decorators: `gluon.jit`. Parameters: `matrix_ptr`, `indices_ptr`, `output_ptr`, `N`, `M`, `layout_2d`, `layout_1d`, `shared_layout`. Key calls include `ttgl.arange`, `ttgl.load`, `ttgl.allocate_shared_memory`, `smem_2d.store`, `smem_2d.reshape`, `smem_1d.gather`, and 2 more. This scope touches layout transformation reasoning.
- **CN:** 定义辅助函数 `shared_gather_kernel`。 装饰器：`gluon.jit`。 参数：`matrix_ptr`、`indices_ptr`、`output_ptr`、`N`、`M`、`layout_2d`、`layout_1d`、`shared_layout`。 关键调用包括 `ttgl.arange`、`ttgl.load`、`ttgl.allocate_shared_memory`、`smem_2d.store`、`smem_2d.reshape`、`smem_1d.gather` 等另外 2 项。 该作用域涉及布局变换推理。

#### Lines 2532-2554

```python
    """Test shared memory gather using smem.gather() with axis-based API."""
    # Load the matrix from global memory into registers
    indices_x = ttgl.arange(0, N, layout=ttgl.SliceLayout(dim=1, parent=layout_2d))
    indices_y = ttgl.arange(0, M, layout=ttgl.SliceLayout(dim=0, parent=layout_2d))
    offsets_2d = indices_x[:, None] * M + indices_y[None, :]
    matrix_data = ttgl.load(matrix_ptr + offsets_2d)

    # Allocate 2D shared memory and store the matrix
    smem_2d = ttgl.allocate_shared_memory(ttgl.float32, [N, M], layout=shared_layout)
    smem_2d.store(matrix_data)

    # Reshape to 1D to test gather along axis 0
    smem_1d = smem_2d.reshape([N * M])

    # Load the gather indices (diagonal elements: 0, M+1, 2*(M+1), ...)
    offsets_1d = ttgl.arange(0, N, layout=layout_1d)
    indices = ttgl.load(indices_ptr + offsets_1d)

    # Gather using axis-based API: result[i] = smem_1d[indices[i]]
    gathered = smem_1d.gather(indices, axis=0)

    # Store result to global memory
    ttgl.store(output_ptr + offsets_1d, gathered)
```
- **EN:** Prepares or updates state through `indices_x`, `indices_y`, `offsets_2d`, `matrix_data`, `smem_2d`, `smem_1d`, `offsets_1d`, `indices`, and 1 more. Invokes `ttgl.arange`, `ttgl.SliceLayout`, `ttgl.load`, `ttgl.allocate_shared_memory`, `smem_2d.store`, `smem_2d.reshape`, and 2 more to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `indices_x`、`indices_y`、`offsets_2d`、`matrix_data`、`smem_2d`、`smem_1d`、`offsets_1d`、`indices` 等另外 1 项 准备或更新状态。 调用 `ttgl.arange`、`ttgl.SliceLayout`、`ttgl.load`、`ttgl.allocate_shared_memory`、`smem_2d.store`、`smem_2d.reshape` 等另外 2 项 执行测试逻辑。 相关主题：布局变换推理。

### Lines 2555-2558

```python


@pytest.mark.parametrize("N,M", [(32, 32), (64, 64), (128, 128)])
def test_shared_gather(N, M):
```
- **EN:** Defines the test function `test_shared_gather`. Decorators: `pytest.mark.parametrize('N,M', [(32, 32), (64, 64), (128, 128)])`. Parameters: `N`, `M`. Key calls include `pytest.mark.parametrize`, `torch.device`, `torch.zeros`, `ttgl.BlockedLayout`, `ttgl.SwizzledSharedLayout`, `torch.testing.assert_close`, and 2 more. This scope touches pytest parametrization, PyTorch tensor setup and checks, layout transformation reasoning.
- **CN:** 定义测试函数 `test_shared_gather`。 装饰器：`pytest.mark.parametrize('N,M', [(32, 32), (64, 64), (128, 128)])`。 参数：`N`、`M`。 关键调用包括 `pytest.mark.parametrize`、`torch.device`、`torch.zeros`、`ttgl.BlockedLayout`、`ttgl.SwizzledSharedLayout`、`torch.testing.assert_close` 等另外 2 项。 该作用域涉及pytest 参数化、PyTorch 张量准备与校验、布局变换推理。

#### Lines 2559-2593

```python
    """Test gathering from 1D reshaped shared memory (diagonal of 2D matrix)."""
    device = torch.device("cuda")

    # Create a test matrix with known values
    matrix = torch.arange(N * M, dtype=torch.float32, device=device).reshape(N, M)

    # Create gather indices for diagonal elements: 0, M+1, 2*(M+1), ...
    indices = torch.arange(N, dtype=torch.int32, device=device) * (M + 1)

    output = torch.zeros(N, dtype=torch.float32, device=device)

    # Compute expected result: diagonal elements
    expected = matrix.flatten()[indices]

    # Create layouts dynamically based on THREADS_PER_WARP
    layout_2d = ttgl.BlockedLayout(size_per_thread=[1, 1], threads_per_warp=[THREADS_PER_WARP // 4, 4],
                                   warps_per_cta=[1, 1], order=[1, 0])
    layout_1d = ttgl.BlockedLayout(size_per_thread=[1], threads_per_warp=[THREADS_PER_WARP], warps_per_cta=[1],
                                   order=[0])
    shared_layout = ttgl.SwizzledSharedLayout(vec=1, per_phase=1, max_phase=1, order=[1, 0])

    # Launch kernel
    shared_gather_kernel[(1, )](
        matrix,
        indices,
        output,
        N=N,
        M=M,
        layout_2d=layout_2d,
        layout_1d=layout_1d,
        shared_layout=shared_layout,
        num_warps=1,
    )

    torch.testing.assert_close(output, expected)
```
- **EN:** Prepares or updates state through `device`, `matrix`, `indices`, `output`, `expected`, `layout_2d`, `layout_1d`, `shared_layout`. Invokes `torch.device`, `torch.arange`, `torch.zeros`, `matrix.flatten`, `ttgl.BlockedLayout`, `ttgl.SwizzledSharedLayout`, and 1 more to execute the test logic. Relevant themes: PyTorch tensor setup and checks, kernel launch orchestration, layout transformation reasoning.
- **CN:** 通过 `device`、`matrix`、`indices`、`output`、`expected`、`layout_2d`、`layout_1d`、`shared_layout` 准备或更新状态。 调用 `torch.device`、`torch.arange`、`torch.zeros`、`matrix.flatten`、`ttgl.BlockedLayout`、`ttgl.SwizzledSharedLayout` 等另外 1 项 执行测试逻辑。 相关主题：PyTorch 张量准备与校验、kernel 启动编排、布局变换推理。

### Lines 2594-2607

```python


@gluon.jit
def shared_scatter_kernel(
    indices_ptr,
    values_ptr,
    output_ptr,
    N: ttgl.constexpr,
    M: ttgl.constexpr,
    layout_2d: ttgl.constexpr,
    layout_1d: ttgl.constexpr,
    shared_layout: ttgl.constexpr,
    use_broadcast: ttgl.constexpr,
):
```
- **EN:** Defines the helper function `shared_scatter_kernel`. Decorators: `gluon.jit`. Parameters: `indices_ptr`, `values_ptr`, `output_ptr`, `N`, `M`, `layout_2d`, `layout_1d`, `shared_layout`, and 1 more. Key calls include `ttgl.allocate_shared_memory`, `ttgl.arange`, `ttgl.zeros`, `smem.store`, `smem.load`, `ttgl.store`, and 5 more. This scope touches layout transformation reasoning.
- **CN:** 定义辅助函数 `shared_scatter_kernel`。 装饰器：`gluon.jit`。 参数：`indices_ptr`、`values_ptr`、`output_ptr`、`N`、`M`、`layout_2d`、`layout_1d`、`shared_layout` 等另外 1 项。 关键调用包括 `ttgl.allocate_shared_memory`、`ttgl.arange`、`ttgl.zeros`、`smem.store`、`smem.load`、`ttgl.store` 等另外 5 项。 该作用域涉及布局变换推理。

#### Lines 2608-2617

```python
    """Test shared memory scatter using smem.scatter() with axis-based API."""
    # Allocate 2D shared memory initialized to zero
    smem = ttgl.allocate_shared_memory(ttgl.float32, [N, M], layout=shared_layout)

    # Initialize shared memory to zero
    indices_x = ttgl.arange(0, N, layout=ttgl.SliceLayout(dim=1, parent=layout_2d))
    indices_y = ttgl.arange(0, M, layout=ttgl.SliceLayout(dim=0, parent=layout_2d))
    offsets_2d = indices_x[:, None] * M + indices_y[None, :]
    zeros = ttgl.zeros([N, M], ttgl.float32, layout=layout_2d)
    smem.store(zeros)
```
- **EN:** Prepares or updates state through `smem`, `indices_x`, `indices_y`, `offsets_2d`, `zeros`. Invokes `ttgl.allocate_shared_memory`, `ttgl.arange`, `ttgl.SliceLayout`, `ttgl.zeros`, `smem.store` to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `smem`、`indices_x`、`indices_y`、`offsets_2d`、`zeros` 准备或更新状态。 调用 `ttgl.allocate_shared_memory`、`ttgl.arange`、`ttgl.SliceLayout`、`ttgl.zeros`、`smem.store` 执行测试逻辑。 相关主题：布局变换推理。

#### Lines 2618-2636

```python

    if use_broadcast:
        # values has shape [N, 1] and indices has shape [1, M]. scatter broadcasts
        # both to [N, M], so each row's scalar value is written across columns:
        # smem[i, indices[0, j]] = values[i, 0].
        values = ttgl.load(values_ptr + indices_x)[:, None]
        indices = indices_y[None, :]
        smem.scatter(values, indices, axis=1)
    else:
        # Reshape to 1D to test scatter along axis 0
        smem_1d = smem.reshape([N * M])

        # Load the scatter indices and values (diagonal elements: 0, M+1, 2*(M+1), ...)
        offsets_1d = ttgl.arange(0, N, layout=layout_1d)
        indices = ttgl.load(indices_ptr + offsets_1d)
        values = ttgl.load(values_ptr + offsets_1d)

        # Scatter using axis-based API: smem_1d[indices[i]] = values[i]
        smem_1d.scatter(values, indices, axis=0)
```
- **EN:** Invokes `smem.scatter`, `smem.reshape`, `ttgl.arange`, `ttgl.load`, `smem_1d.scatter` to execute the test logic. Branches on runtime or test conditions. Relevant themes: layout transformation reasoning.
- **CN:** 调用 `smem.scatter`、`smem.reshape`、`ttgl.arange`、`ttgl.load`、`smem_1d.scatter` 执行测试逻辑。 根据运行时或测试条件进行分支。 相关主题：布局变换推理。

#### Lines 2637-2642

```python

    # Read back the full matrix from shared memory
    matrix_data = smem.load(layout=layout_2d)

    # Store result to global memory
    ttgl.store(output_ptr + offsets_2d, matrix_data)
```
- **EN:** Prepares or updates state through `matrix_data`. Invokes `smem.load`, `ttgl.store` to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `matrix_data` 准备或更新状态。 调用 `smem.load`、`ttgl.store` 执行测试逻辑。 相关主题：布局变换推理。

### Lines 2643-2646

```python


@pytest.mark.parametrize("N,M", [(32, 32), (64, 64), (128, 128)])
def test_shared_scatter(N, M):
```
- **EN:** Defines the test function `test_shared_scatter`. Decorators: `pytest.mark.parametrize('N,M', [(32, 32), (64, 64), (128, 128)])`. Parameters: `N`, `M`. Key calls include `pytest.mark.parametrize`, `torch.device`, `torch.zeros`, `ttgl.BlockedLayout`, `ttgl.SwizzledSharedLayout`, `torch.testing.assert_close`, and 2 more. This scope touches pytest parametrization, PyTorch tensor setup and checks, layout transformation reasoning.
- **CN:** 定义测试函数 `test_shared_scatter`。 装饰器：`pytest.mark.parametrize('N,M', [(32, 32), (64, 64), (128, 128)])`。 参数：`N`、`M`。 关键调用包括 `pytest.mark.parametrize`、`torch.device`、`torch.zeros`、`ttgl.BlockedLayout`、`ttgl.SwizzledSharedLayout`、`torch.testing.assert_close` 等另外 2 项。 该作用域涉及pytest 参数化、PyTorch 张量准备与校验、布局变换推理。

#### Lines 2647-2659

```python
    """Test scattering to 1D reshaped shared memory (diagonal of 2D matrix)."""
    device = torch.device("cuda")

    # Create scatter indices for diagonal elements: 0, M+1, 2*(M+1), ...
    indices = torch.arange(N, dtype=torch.int32, device=device) * (M + 1)

    # Create values to scatter
    values = torch.arange(N, dtype=torch.float32, device=device) + 100.0

    output = torch.zeros((N, M), dtype=torch.float32, device=device)

    # Compute expected result: matrix starts at zero, then diagonal gets values
    expected = torch.zeros((N, M), dtype=torch.float32, device=device)
```
- **EN:** Prepares or updates state through `device`, `indices`, `values`, `output`, `expected`. Invokes `torch.device`, `torch.arange`, `torch.zeros` to execute the test logic. Relevant themes: PyTorch tensor setup and checks.
- **CN:** 通过 `device`、`indices`、`values`、`output`、`expected` 准备或更新状态。 调用 `torch.device`、`torch.arange`、`torch.zeros` 执行测试逻辑。 相关主题：PyTorch 张量准备与校验。

#### Lines 2660-2661

```python
    for i in range(N):
        expected[i, i] = values[i]
```
- **EN:** Iterates across cases or data tiles.
- **CN:** 通过循环覆盖多个用例或数据分块。

#### Lines 2662-2700

```python

    # Create layouts dynamically based on THREADS_PER_WARP
    layout_2d = ttgl.BlockedLayout(size_per_thread=[1, 1], threads_per_warp=[THREADS_PER_WARP // 4, 4],
                                   warps_per_cta=[1, 1], order=[1, 0])
    layout_1d = ttgl.BlockedLayout(size_per_thread=[1], threads_per_warp=[THREADS_PER_WARP], warps_per_cta=[1],
                                   order=[0])
    shared_layout = ttgl.SwizzledSharedLayout(vec=1, per_phase=1, max_phase=1, order=[1, 0])

    # Launch kernel
    shared_scatter_kernel[(1, )](
        indices,
        values,
        output,
        N=N,
        M=M,
        layout_2d=layout_2d,
        layout_1d=layout_1d,
        shared_layout=shared_layout,
        use_broadcast=False,
        num_warps=1,
    )

    torch.testing.assert_close(output, expected)

    broadcast_output = torch.empty((N, M), dtype=torch.float32, device=device)
    shared_scatter_kernel[(1, )](
        indices,
        values,
        broadcast_output,
        N=N,
        M=M,
        layout_2d=layout_2d,
        layout_1d=layout_1d,
        shared_layout=shared_layout,
        use_broadcast=True,
        num_warps=1,
    )

    torch.testing.assert_close(broadcast_output, values[:, None].expand(N, M).contiguous())
```
- **EN:** Prepares or updates state through `layout_2d`, `layout_1d`, `shared_layout`, `broadcast_output`. Invokes `ttgl.BlockedLayout`, `ttgl.SwizzledSharedLayout`, `torch.testing.assert_close`, `torch.empty` to execute the test logic. Relevant themes: PyTorch tensor setup and checks, kernel launch orchestration, layout transformation reasoning.
- **CN:** 通过 `layout_2d`、`layout_1d`、`shared_layout`、`broadcast_output` 准备或更新状态。 调用 `ttgl.BlockedLayout`、`ttgl.SwizzledSharedLayout`、`torch.testing.assert_close`、`torch.empty` 执行测试逻辑。 相关主题：PyTorch 张量准备与校验、kernel 启动编排、布局变换推理。

### Lines 2701-2723

```python


@gluon.jit
def shared_atomic_scatter_rmw_kernel(
    values_ptr,
    indices_ptr,
    mask_ptr,
    old_ptr,
    final_ptr,
    N: ttgl.constexpr,
    M: ttgl.constexpr,
    axis: ttgl.constexpr,
    op: ttgl.constexpr,
    dtype: ttgl.constexpr,
    use_mask: ttgl.constexpr,
    use_constant_values: ttgl.constexpr,
    init_value: ttgl.constexpr,
    RHS_N: ttgl.constexpr,
    RHS_M: ttgl.constexpr,
    layout_2d: ttgl.constexpr,
    layout_rhs: ttgl.constexpr,
    shared_layout: ttgl.constexpr,
):
```
- **EN:** Defines the helper function `shared_atomic_scatter_rmw_kernel`. Decorators: `gluon.jit`. Parameters: `values_ptr`, `indices_ptr`, `mask_ptr`, `old_ptr`, `final_ptr`, `N`, `M`, `axis`, and 10 more. Key calls include `ttgl.arange`, `ttgl.allocate_shared_memory`, `smem.store`, `ttgl.load`, `ttgl.store`, `smem.load`, and 9 more. This scope touches layout transformation reasoning.
- **CN:** 定义辅助函数 `shared_atomic_scatter_rmw_kernel`。 装饰器：`gluon.jit`。 参数：`values_ptr`、`indices_ptr`、`mask_ptr`、`old_ptr`、`final_ptr`、`N`、`M`、`axis` 等另外 10 项。 关键调用包括 `ttgl.arange`、`ttgl.allocate_shared_memory`、`smem.store`、`ttgl.load`、`ttgl.store`、`smem.load` 等另外 9 项。 该作用域涉及布局变换推理。

#### Lines 2724-2734

```python
    indices_x = ttgl.arange(0, N, layout=ttgl.SliceLayout(dim=1, parent=layout_2d))
    indices_y = ttgl.arange(0, M, layout=ttgl.SliceLayout(dim=0, parent=layout_2d))
    offsets_2d = indices_x[:, None] * M + indices_y[None, :]
    rhs_x = ttgl.arange(0, RHS_N, layout=ttgl.SliceLayout(dim=1, parent=layout_rhs))
    rhs_y = ttgl.arange(0, RHS_M, layout=ttgl.SliceLayout(dim=0, parent=layout_rhs))
    offsets_rhs = rhs_x[:, None] * RHS_M + rhs_y[None, :]

    smem = ttgl.allocate_shared_memory(dtype, [N, M], layout=shared_layout)
    smem.store(ttgl.full([N, M], init_value, dtype, layout=layout_2d))

    indices = ttgl.load(indices_ptr + offsets_rhs)
```
- **EN:** Prepares or updates state through `indices_x`, `indices_y`, `offsets_2d`, `rhs_x`, `rhs_y`, `offsets_rhs`, `smem`, `indices`. Invokes `ttgl.arange`, `ttgl.SliceLayout`, `ttgl.allocate_shared_memory`, `smem.store`, `ttgl.full`, `ttgl.load` to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `indices_x`、`indices_y`、`offsets_2d`、`rhs_x`、`rhs_y`、`offsets_rhs`、`smem`、`indices` 准备或更新状态。 调用 `ttgl.arange`、`ttgl.SliceLayout`、`ttgl.allocate_shared_memory`、`smem.store`、`ttgl.full`、`ttgl.load` 执行测试逻辑。 相关主题：布局变换推理。

#### Lines 2735-2739

```python
    if use_constant_values:
        # Cover the all-ones RHS path; int32 add can lower to atom.inc on NVIDIA.
        values = ttgl.full([RHS_N, RHS_M], 1, dtype, layout_rhs)
    else:
        values = ttgl.load(values_ptr + offsets_rhs)
```
- **EN:** Invokes `ttgl.full`, `ttgl.load` to execute the test logic. Branches on runtime or test conditions. Relevant themes: layout transformation reasoning.
- **CN:** 调用 `ttgl.full`、`ttgl.load` 执行测试逻辑。 根据运行时或测试条件进行分支。 相关主题：布局变换推理。

#### Lines 2740-2771

```python

    if use_mask:
        mask = ttgl.load(mask_ptr + offsets_rhs)
        if op == "add":
            old = smem.atomic_scatter_add(values, indices, axis=axis, mask=mask)
        elif op == "max":
            old = smem.atomic_scatter_max(values, indices, axis=axis, mask=mask)
        elif op == "min":
            old = smem.atomic_scatter_min(values, indices, axis=axis, mask=mask)
        elif op == "and":
            old = smem.atomic_scatter_and(values, indices, axis=axis, mask=mask)
        elif op == "or":
            old = smem.atomic_scatter_or(values, indices, axis=axis, mask=mask)
        elif op == "xor":
            old = smem.atomic_scatter_xor(values, indices, axis=axis, mask=mask)
        else:
            old = smem.atomic_scatter_xchg(values, indices, axis=axis, mask=mask)
    else:
        if op == "add":
            old = smem.atomic_scatter_add(values, indices, axis=axis)
        elif op == "max":
            old = smem.atomic_scatter_max(values, indices, axis=axis)
        elif op == "min":
            old = smem.atomic_scatter_min(values, indices, axis=axis)
        elif op == "and":
            old = smem.atomic_scatter_and(values, indices, axis=axis)
        elif op == "or":
            old = smem.atomic_scatter_or(values, indices, axis=axis)
        elif op == "xor":
            old = smem.atomic_scatter_xor(values, indices, axis=axis)
        else:
            old = smem.atomic_scatter_xchg(values, indices, axis=axis)
```
- **EN:** Invokes `ttgl.load`, `smem.atomic_scatter_add`, `smem.atomic_scatter_max`, `smem.atomic_scatter_min`, `smem.atomic_scatter_and`, `smem.atomic_scatter_or`, and 2 more to execute the test logic. Branches on runtime or test conditions.
- **CN:** 调用 `ttgl.load`、`smem.atomic_scatter_add`、`smem.atomic_scatter_max`、`smem.atomic_scatter_min`、`smem.atomic_scatter_and`、`smem.atomic_scatter_or` 等另外 2 项 执行测试逻辑。 根据运行时或测试条件进行分支。

#### Lines 2772-2775

```python
    ttgl.store(old_ptr + offsets_rhs, old)

    final = smem.load(layout=layout_2d)
    ttgl.store(final_ptr + offsets_2d, final)
```
- **EN:** Prepares or updates state through `final`. Invokes `ttgl.store`, `smem.load` to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `final` 准备或更新状态。 调用 `ttgl.store`、`smem.load` 执行测试逻辑。 相关主题：布局变换推理。

### Lines 2776-2778

```python


def _expected_shared_atomic_scatter_rmw(op, init_value, indices, values, mask, axis, shape):
```
- **EN:** Defines the helper function `_expected_shared_atomic_scatter_rmw`. Parameters: `op`, `init_value`, `indices`, `values`, `mask`, `axis`, `shape`. Nested definitions in this scope: `combine`. Key calls include `torch.full`, `torch.arange`, `torch.maximum`, `torch.minimum`, `combine`. This scope touches PyTorch tensor setup and checks.
- **CN:** 定义辅助函数 `_expected_shared_atomic_scatter_rmw`。 参数：`op`、`init_value`、`indices`、`values`、`mask`、`axis`、`shape`。 该作用域中的嵌套定义：`combine`。 关键调用包括 `torch.full`、`torch.arange`、`torch.maximum`、`torch.minimum`、`combine`。 该作用域涉及PyTorch 张量准备与校验。

#### Lines 2779-2781

```python
    device = values.device
    expected = torch.full(shape, init_value, dtype=values.dtype, device=device)
    rhs_n, rhs_m = values.shape
```
- **EN:** Prepares or updates state through `device`, `expected`, `rhs_n`, `rhs_m`. Invokes `torch.full` to execute the test logic. Relevant themes: PyTorch tensor setup and checks.
- **CN:** 通过 `device`、`expected`、`rhs_n`、`rhs_m` 准备或更新状态。 调用 `torch.full` 执行测试逻辑。 相关主题：PyTorch 张量准备与校验。

#### Lines 2782-2783

```python

    def combine(old, val):
```
- **EN:** Defines the helper function `combine`. Parameters: `old`, `val`. Key calls include `torch.maximum`, `torch.minimum`. This scope touches PyTorch tensor setup and checks.
- **CN:** 定义辅助函数 `combine`。 参数：`old`、`val`。 关键调用包括 `torch.maximum`、`torch.minimum`。 该作用域涉及PyTorch 张量准备与校验。

##### Lines 2784-2785

```python
        if op == "add":
            return old + val
```
- **EN:** Branches on runtime or test conditions.
- **CN:** 根据运行时或测试条件进行分支。

##### Lines 2786-2787

```python
        if op == "max":
            return torch.maximum(old, val)
```
- **EN:** Invokes `torch.maximum` to execute the test logic. Branches on runtime or test conditions. Relevant themes: PyTorch tensor setup and checks.
- **CN:** 调用 `torch.maximum` 执行测试逻辑。 根据运行时或测试条件进行分支。 相关主题：PyTorch 张量准备与校验。

##### Lines 2788-2789

```python
        if op == "min":
            return torch.minimum(old, val)
```
- **EN:** Invokes `torch.minimum` to execute the test logic. Branches on runtime or test conditions. Relevant themes: PyTorch tensor setup and checks.
- **CN:** 调用 `torch.minimum` 执行测试逻辑。 根据运行时或测试条件进行分支。 相关主题：PyTorch 张量准备与校验。

##### Lines 2790-2791

```python
        if op == "and":
            return old & val
```
- **EN:** Branches on runtime or test conditions.
- **CN:** 根据运行时或测试条件进行分支。

##### Lines 2792-2793

```python
        if op == "or":
            return old | val
```
- **EN:** Branches on runtime or test conditions.
- **CN:** 根据运行时或测试条件进行分支。

##### Lines 2794-2795

```python
        if op == "xor":
            return old ^ val
```
- **EN:** Branches on runtime or test conditions.
- **CN:** 根据运行时或测试条件进行分支。

##### Lines 2796-2797

```python
        # xchg
        return val
```
- **EN:** Carries supporting logic for the surrounding test or helper scope.
- **CN:** 为周围的测试或辅助作用域提供支撑逻辑。

#### Lines 2798-2812

```python

    if axis == 1:
        row_ids = torch.arange(rhs_n, device=device)
        for j in range(rhs_m):
            active = mask[:, j]
            rows = row_ids[active]
            cols = indices[active, j].long()
            expected[rows, cols] = combine(expected[rows, cols], values[active, j])
    else:
        col_ids = torch.arange(rhs_m, device=device)
        for i in range(rhs_n):
            active = mask[i, :]
            rows = indices[i, active].long()
            cols = col_ids[active]
            expected[rows, cols] = combine(expected[rows, cols], values[i, active])
```
- **EN:** Invokes `torch.arange`, `combine` to execute the test logic. Branches on runtime or test conditions. Relevant themes: PyTorch tensor setup and checks.
- **CN:** 调用 `torch.arange`、`combine` 执行测试逻辑。 根据运行时或测试条件进行分支。 相关主题：PyTorch 张量准备与校验。

#### Lines 2813-2813

```python
    return expected
```
- **EN:** Carries supporting logic for the surrounding test or helper scope.
- **CN:** 为周围的测试或辅助作用域提供支撑逻辑。

### Lines 2814-2816

```python


def _shared_atomic_scatter_rmw_cases():
```
- **EN:** Defines the helper function `_shared_atomic_scatter_rmw_cases`. Key calls include `cases.append`, `id_parts.append`, `pytest.param`. This scope touches PyTorch tensor setup and checks.
- **CN:** 定义辅助函数 `_shared_atomic_scatter_rmw_cases`。 关键调用包括 `cases.append`、`id_parts.append`、`pytest.param`。 该作用域涉及PyTorch 张量准备与校验。

#### Lines 2817-2850

```python
    int32_dtype_cases = [
        ("unmasked_int32", False, torch.int32, ttgl.int32),
        ("masked_int32", True, torch.int32, ttgl.int32),
    ]
    add_dtype_cases = [
        *int32_dtype_cases,
        ("unmasked_float16", False, torch.float16, ttgl.float16),
        ("unmasked_float32", False, torch.float32, ttgl.float32),
        ("masked_float32", True, torch.float32, ttgl.float32),
    ]
    xchg_dtype_cases = [
        *int32_dtype_cases,
        ("unmasked_float32", False, torch.float32, ttgl.float32),
        ("masked_float32", True, torch.float32, ttgl.float32),
    ]
    value_cases = [
        ("loaded_values", False),
        ("constant_values", True),
    ]
    shape_cases = [
        ("axis1_rhs_cols", 16, 32, 1, (16, 2)),
        ("axis0_rhs_rows", 32, 16, 0, (2, 16)),
    ]
    op_cases = [
        ("add", 0, add_dtype_cases),
        ("max", 0, int32_dtype_cases),
        ("min", 99, int32_dtype_cases),
        ("and", 15, int32_dtype_cases),
        ("or", 0, int32_dtype_cases),
        ("xor", 0, int32_dtype_cases),
        ("xchg", 0, xchg_dtype_cases),
    ]

    cases = []
```
- **EN:** Prepares or updates state through `int32_dtype_cases`, `add_dtype_cases`, `xchg_dtype_cases`, `value_cases`, `shape_cases`, `op_cases`, `cases`. Relevant themes: PyTorch tensor setup and checks.
- **CN:** 通过 `int32_dtype_cases`、`add_dtype_cases`、`xchg_dtype_cases`、`value_cases`、`shape_cases`、`op_cases`、`cases` 准备或更新状态。 相关主题：PyTorch 张量准备与校验。

#### Lines 2851-2874

```python
    for op, init_value, dtype_cases in op_cases:
        for dtype_id, use_mask, torch_dtype, gluon_dtype in dtype_cases:
            for values_id, use_constant_values in value_cases:
                for shape_id, N, M, axis, rhs_shape in shape_cases:
                    id_parts = [op, dtype_id]
                    if values_id:
                        id_parts.append(values_id)
                    if shape_id:
                        id_parts.append(shape_id)
                    case_id = "_".join(id_parts)
                    cases.append(
                        pytest.param(
                            op,
                            init_value,
                            use_mask,
                            torch_dtype,
                            gluon_dtype,
                            use_constant_values,
                            N,
                            M,
                            axis,
                            rhs_shape,
                            id=case_id,
                        ))
```
- **EN:** Invokes `cases.append`, `id_parts.append`, `pytest.param` to execute the test logic. Iterates across cases or data tiles.
- **CN:** 调用 `cases.append`、`id_parts.append`、`pytest.param` 执行测试逻辑。 通过循环覆盖多个用例或数据分块。

#### Lines 2875-2876

```python

    return cases
```
- **EN:** Carries supporting logic for the surrounding test or helper scope.
- **CN:** 为周围的测试或辅助作用域提供支撑逻辑。

### Lines 2877-2884

```python


@pytest.mark.parametrize(
    "op,init_value,use_mask,torch_dtype,gluon_dtype,use_constant_values,N,M,axis,rhs_shape",
    _shared_atomic_scatter_rmw_cases(),
)
def test_shared_atomic_scatter_rmw(op, init_value, use_mask, torch_dtype, gluon_dtype, use_constant_values, N, M, axis,
                                   rhs_shape):
```
- **EN:** Defines the test function `test_shared_atomic_scatter_rmw`. Decorators: `pytest.mark.parametrize('op,init_value,use_mask,torch_dtype,gluon_dtype,use_constant_values,N,M,axis,rhs_shape', _shared_atomic_scatter_rmw_cases())`. Parameters: `op`, `init_value`, `use_mask`, `torch_dtype`, `gluon_dtype`, `use_constant_values`, `N`, `M`, and 2 more. Key calls include `pytest.mark.parametrize`, `torch.device`, `torch.empty`, `_expected_shared_atomic_scatter_rmw`, `ttgl.BlockedLayout`, `ttgl.SwizzledSharedLayout`, and 10 more. This scope touches pytest parametrization, PyTorch tensor setup and checks, layout transformation reasoning, random-data generation.
- **CN:** 定义测试函数 `test_shared_atomic_scatter_rmw`。 装饰器：`pytest.mark.parametrize('op,init_value,use_mask,torch_dtype,gluon_dtype,use_constant_values,N,M,axis,rhs_shape', _shared_atomic_scatter_rmw_cases())`。 参数：`op`、`init_value`、`use_mask`、`torch_dtype`、`gluon_dtype`、`use_constant_values`、`N`、`M` 等另外 2 项。 关键调用包括 `pytest.mark.parametrize`、`torch.device`、`torch.empty`、`_expected_shared_atomic_scatter_rmw`、`ttgl.BlockedLayout`、`ttgl.SwizzledSharedLayout` 等另外 10 项。 该作用域涉及pytest 参数化、PyTorch 张量准备与校验、布局变换推理、随机数据生成。

#### Lines 2885-2886

```python
    if is_hip_cdna() or is_hip_rdna():
        pytest.skip("Shared atomic_scatter_rmw is not supported on AMD")
```
- **EN:** Invokes `is_hip_cdna`, `is_hip_rdna`, `pytest.skip` to execute the test logic. Branches on runtime or test conditions.
- **CN:** 调用 `is_hip_cdna`、`is_hip_rdna`、`pytest.skip` 执行测试逻辑。 根据运行时或测试条件进行分支。

#### Lines 2887-2889

```python

    device = torch.device("cuda")
    rhs_n, rhs_m = rhs_shape
```
- **EN:** Prepares or updates state through `device`, `rhs_n`, `rhs_m`. Invokes `torch.device` to execute the test logic. Relevant themes: PyTorch tensor setup and checks.
- **CN:** 通过 `device`、`rhs_n`、`rhs_m` 准备或更新状态。 调用 `torch.device` 执行测试逻辑。 相关主题：PyTorch 张量准备与校验。

#### Lines 2890-2895

```python

    if use_constant_values:
        values = torch.ones(rhs_shape, dtype=torch_dtype, device=device)
    else:
        values = torch.arange(rhs_n * rhs_m, dtype=torch.int32, device=device).reshape(rhs_shape)
        values = (values % 7 + 1).to(torch_dtype)
```
- **EN:** Invokes `torch.ones`, `torch.arange` to execute the test logic. Branches on runtime or test conditions. Relevant themes: PyTorch tensor setup and checks.
- **CN:** 调用 `torch.ones`、`torch.arange` 执行测试逻辑。 根据运行时或测试条件进行分支。 相关主题：PyTorch 张量准备与校验。

#### Lines 2896-2906

```python
    if op == "xchg":
        # make xchg deterministic with broadcasting
        if axis == 1:
            base_indices = torch.arange(0, rhs_m, dtype=torch.int32, device=device)[None, :]
        else:
            base_indices = torch.arange(0, rhs_n, dtype=torch.int32, device=device)[:, None]
        indices = base_indices.expand(rhs_n, rhs_m).contiguous()
    else:
        torch.manual_seed(23 if use_mask else 17)
        upper = M if axis == 1 else N
        indices = torch.randint(0, upper, rhs_shape, dtype=torch.int32, device=device)
```
- **EN:** Invokes `torch.manual_seed`, `torch.randint`, `torch.arange`, `base_indices.expand` to execute the test logic. Branches on runtime or test conditions. Relevant themes: PyTorch tensor setup and checks, random-data generation.
- **CN:** 调用 `torch.manual_seed`、`torch.randint`、`torch.arange`、`base_indices.expand` 执行测试逻辑。 根据运行时或测试条件进行分支。 相关主题：PyTorch 张量准备与校验、随机数据生成。

#### Lines 2907-2911

```python

    if use_mask:
        mask = (torch.arange(rhs_n * rhs_m, device=device).reshape(rhs_n, rhs_m) % 3) != 0
    else:
        mask = torch.ones((rhs_n, rhs_m), dtype=torch.bool, device=device)
```
- **EN:** Invokes `torch.ones`, `torch.arange` to execute the test logic. Branches on runtime or test conditions. Relevant themes: PyTorch tensor setup and checks.
- **CN:** 调用 `torch.ones`、`torch.arange` 执行测试逻辑。 根据运行时或测试条件进行分支。 相关主题：PyTorch 张量准备与校验。

#### Lines 2912-2945

```python

    old = torch.empty((rhs_n, rhs_m), dtype=torch_dtype, device=device)
    final = torch.empty((N, M), dtype=torch_dtype, device=device)
    expected = _expected_shared_atomic_scatter_rmw(op, init_value, indices, values, mask, axis, (N, M))

    layout_2d = ttgl.BlockedLayout(size_per_thread=[1, 1], threads_per_warp=[THREADS_PER_WARP // 4, 4],
                                   warps_per_cta=[4, 1], order=[1, 0])
    layout_rhs = ttgl.BlockedLayout(size_per_thread=[1, 1], threads_per_warp=[THREADS_PER_WARP // 4, 4],
                                    warps_per_cta=[4, 1], order=[1, 0])
    shared_layout = ttgl.SwizzledSharedLayout(vec=1, per_phase=1, max_phase=1, order=[1, 0])

    shared_atomic_scatter_rmw_kernel[(1, )](
        values,
        indices,
        mask,
        old,
        final,
        N=N,
        M=M,
        axis=axis,
        op=op,
        dtype=gluon_dtype,
        use_mask=use_mask,
        use_constant_values=use_constant_values,
        init_value=init_value,
        RHS_N=rhs_n,
        RHS_M=rhs_m,
        layout_2d=layout_2d,
        layout_rhs=layout_rhs,
        shared_layout=shared_layout,
        num_warps=4,
    )

    torch.testing.assert_close(final, expected, atol=0, rtol=0)
```
- **EN:** Prepares or updates state through `old`, `final`, `expected`, `layout_2d`, `layout_rhs`, `shared_layout`. Invokes `torch.empty`, `_expected_shared_atomic_scatter_rmw`, `ttgl.BlockedLayout`, `ttgl.SwizzledSharedLayout`, `torch.testing.assert_close` to execute the test logic. Relevant themes: PyTorch tensor setup and checks, layout transformation reasoning.
- **CN:** 通过 `old`、`final`、`expected`、`layout_2d`、`layout_rhs`、`shared_layout` 准备或更新状态。 调用 `torch.empty`、`_expected_shared_atomic_scatter_rmw`、`ttgl.BlockedLayout`、`ttgl.SwizzledSharedLayout`、`torch.testing.assert_close` 执行测试逻辑。 相关主题：PyTorch 张量准备与校验、布局变换推理。

### Lines 2946-2957

```python


@gluon.jit
def shared_atomic_scatter_rmw_broadcast_kernel(
    values_ptr,
    old_ptr,
    final_ptr,
    N: ttgl.constexpr,
    M: ttgl.constexpr,
    layout_2d: ttgl.constexpr,
    shared_layout: ttgl.constexpr,
):
```
- **EN:** Defines the helper function `shared_atomic_scatter_rmw_broadcast_kernel`. Decorators: `gluon.jit`. Parameters: `values_ptr`, `old_ptr`, `final_ptr`, `N`, `M`, `layout_2d`, `shared_layout`. Key calls include `ttgl.arange`, `ttgl.allocate_shared_memory`, `smem.store`, `smem.atomic_scatter_add`, `ttgl.store`, `smem.load`, and 3 more. This scope touches layout transformation reasoning.
- **CN:** 定义辅助函数 `shared_atomic_scatter_rmw_broadcast_kernel`。 装饰器：`gluon.jit`。 参数：`values_ptr`、`old_ptr`、`final_ptr`、`N`、`M`、`layout_2d`、`shared_layout`。 关键调用包括 `ttgl.arange`、`ttgl.allocate_shared_memory`、`smem.store`、`smem.atomic_scatter_add`、`ttgl.store`、`smem.load` 等另外 3 项。 该作用域涉及布局变换推理。

#### Lines 2958-2974

```python
    indices_x = ttgl.arange(0, N, layout=ttgl.SliceLayout(dim=1, parent=layout_2d))
    indices_y = ttgl.arange(0, M, layout=ttgl.SliceLayout(dim=0, parent=layout_2d))
    offsets_2d = indices_x[:, None] * M + indices_y[None, :]

    smem = ttgl.allocate_shared_memory(ttgl.float32, [N, M], layout=shared_layout)
    smem.store(ttgl.zeros([N, M], ttgl.float32, layout=layout_2d))

    # values has shape [N, 1], indices has shape [1, M], and mask has shape
    # [N, 1]. atomic_scatter_add broadcasts all three operands to [N, M].
    values = ttgl.load(values_ptr + indices_x)[:, None]
    indices = indices_y[None, :]
    mask = (indices_x >= 0)[:, None]
    old = smem.atomic_scatter_add(values, indices, axis=1, mask=mask)
    ttgl.store(old_ptr + offsets_2d, old)

    final = smem.load(layout=layout_2d)
    ttgl.store(final_ptr + offsets_2d, final)
```
- **EN:** Prepares or updates state through `indices_x`, `indices_y`, `offsets_2d`, `smem`, `values`, `indices`, `mask`, `old`, and 1 more. Invokes `ttgl.arange`, `ttgl.SliceLayout`, `ttgl.allocate_shared_memory`, `smem.store`, `ttgl.zeros`, `ttgl.load`, and 3 more to execute the test logic. Relevant themes: layout transformation reasoning, random-data generation.
- **CN:** 通过 `indices_x`、`indices_y`、`offsets_2d`、`smem`、`values`、`indices`、`mask`、`old` 等另外 1 项 准备或更新状态。 调用 `ttgl.arange`、`ttgl.SliceLayout`、`ttgl.allocate_shared_memory`、`smem.store`、`ttgl.zeros`、`ttgl.load` 等另外 3 项 执行测试逻辑。 相关主题：布局变换推理、随机数据生成。

### Lines 2975-2977

```python


def test_shared_atomic_scatter_rmw_broadcast():
```
- **EN:** Defines the test function `test_shared_atomic_scatter_rmw_broadcast`. Key calls include `torch.device`, `torch.empty`, `ttgl.BlockedLayout`, `ttgl.SwizzledSharedLayout`, `torch.testing.assert_close`, `is_hip_cdna`, and 4 more. This scope touches PyTorch tensor setup and checks, layout transformation reasoning.
- **CN:** 定义测试函数 `test_shared_atomic_scatter_rmw_broadcast`。 关键调用包括 `torch.device`、`torch.empty`、`ttgl.BlockedLayout`、`ttgl.SwizzledSharedLayout`、`torch.testing.assert_close`、`is_hip_cdna` 等另外 4 项。 该作用域涉及PyTorch 张量准备与校验、布局变换推理。

#### Lines 2978-2979

```python
    if is_hip_cdna() or is_hip_rdna():
        pytest.skip("Shared atomic_scatter_rmw is not supported on AMD")
```
- **EN:** Invokes `is_hip_cdna`, `is_hip_rdna`, `pytest.skip` to execute the test logic. Branches on runtime or test conditions.
- **CN:** 调用 `is_hip_cdna`、`is_hip_rdna`、`pytest.skip` 执行测试逻辑。 根据运行时或测试条件进行分支。

#### Lines 2980-3004

```python

    device = torch.device("cuda")
    N, M = 16, 32
    values = torch.arange(N, dtype=torch.float32, device=device) + 1.0
    old = torch.empty((N, M), dtype=torch.float32, device=device)
    final = torch.empty((N, M), dtype=torch.float32, device=device)

    layout_2d = ttgl.BlockedLayout(size_per_thread=[1, 1], threads_per_warp=[THREADS_PER_WARP // 4, 4],
                                   warps_per_cta=[4, 1], order=[1, 0])
    shared_layout = ttgl.SwizzledSharedLayout(vec=1, per_phase=1, max_phase=1, order=[1, 0])

    shared_atomic_scatter_rmw_broadcast_kernel[(1, )](
        values,
        old,
        final,
        N=N,
        M=M,
        layout_2d=layout_2d,
        shared_layout=shared_layout,
        num_warps=4,
    )

    expected = values[:, None].expand(N, M).contiguous()
    torch.testing.assert_close(final, expected, atol=0, rtol=0)
    torch.testing.assert_close(old, torch.zeros_like(old), atol=0, rtol=0)
```
- **EN:** Prepares or updates state through `device`, `N`, `M`, `values`, `old`, `final`, `layout_2d`, `shared_layout`, and 1 more. Invokes `torch.device`, `torch.arange`, `torch.empty`, `ttgl.BlockedLayout`, `ttgl.SwizzledSharedLayout`, `torch.testing.assert_close`, and 1 more to execute the test logic. Relevant themes: PyTorch tensor setup and checks, layout transformation reasoning.
- **CN:** 通过 `device`、`N`、`M`、`values`、`old`、`final`、`layout_2d`、`shared_layout` 等另外 1 项 准备或更新状态。 调用 `torch.device`、`torch.arange`、`torch.empty`、`ttgl.BlockedLayout`、`ttgl.SwizzledSharedLayout`、`torch.testing.assert_close` 等另外 1 项 执行测试逻辑。 相关主题：PyTorch 张量准备与校验、布局变换推理。

### Lines 3005-3013

```python


# ============================================================================
# Multi-warp Tests
# ============================================================================


@pytest.mark.parametrize("N,M,num_warps", [(64, 64, 2), (128, 128, 4)])
def test_scatter_gather_multiwarp(N, M, num_warps):
```
- **EN:** Defines the test function `test_scatter_gather_multiwarp`. Decorators: `pytest.mark.parametrize('N,M,num_warps', [(64, 64, 2), (128, 128, 4)])`. Parameters: `N`, `M`, `num_warps`. Key calls include `pytest.mark.parametrize`, `torch.device`, `ttgl.BlockedLayout`, `ttgl.SwizzledSharedLayout`, `torch.zeros`, `torch.testing.assert_close`, and 2 more. This scope touches pytest parametrization, PyTorch tensor setup and checks, layout transformation reasoning.
- **CN:** 定义测试函数 `test_scatter_gather_multiwarp`。 装饰器：`pytest.mark.parametrize('N,M,num_warps', [(64, 64, 2), (128, 128, 4)])`。 参数：`N`、`M`、`num_warps`。 关键调用包括 `pytest.mark.parametrize`、`torch.device`、`ttgl.BlockedLayout`、`ttgl.SwizzledSharedLayout`、`torch.zeros`、`torch.testing.assert_close` 等另外 2 项。 该作用域涉及pytest 参数化、PyTorch 张量准备与校验、布局变换推理。

#### Lines 3014-3048

```python
    """Test scatter and gather with multiple warps."""
    device = torch.device("cuda")

    # Create layouts with multiple warps (shared across both tests)
    layout_2d = ttgl.BlockedLayout(size_per_thread=[1, 1], threads_per_warp=[THREADS_PER_WARP // 4, 4],
                                   warps_per_cta=[num_warps, 1], order=[1, 0])
    layout_1d = ttgl.BlockedLayout(size_per_thread=[1], threads_per_warp=[THREADS_PER_WARP], warps_per_cta=[num_warps],
                                   order=[0])
    shared_layout = ttgl.SwizzledSharedLayout(vec=1, per_phase=1, max_phase=1, order=[1, 0])

    # Test gather
    matrix = torch.arange(N * M, dtype=torch.float32, device=device).reshape(N, M)
    gather_indices = torch.arange(N, dtype=torch.int32, device=device) * (M + 1)
    gather_output = torch.zeros(N, dtype=torch.float32, device=device)
    gather_expected = matrix.flatten()[gather_indices]

    shared_gather_kernel[(1, )](
        matrix,
        gather_indices,
        gather_output,
        N=N,
        M=M,
        layout_2d=layout_2d,
        layout_1d=layout_1d,
        shared_layout=shared_layout,
        num_warps=num_warps,
    )

    torch.testing.assert_close(gather_output, gather_expected)

    # Test scatter
    scatter_indices = torch.arange(N, dtype=torch.int32, device=device) * (M + 1)
    scatter_values = torch.arange(N, dtype=torch.float32, device=device) + 100.0
    scatter_output = torch.zeros((N, M), dtype=torch.float32, device=device)
    scatter_expected = torch.zeros((N, M), dtype=torch.float32, device=device)
```
- **EN:** Prepares or updates state through `device`, `layout_2d`, `layout_1d`, `shared_layout`, `matrix`, `gather_indices`, `gather_output`, `gather_expected`, and 4 more. Invokes `torch.device`, `ttgl.BlockedLayout`, `ttgl.SwizzledSharedLayout`, `torch.arange`, `torch.zeros`, `matrix.flatten`, and 1 more to execute the test logic. Relevant themes: PyTorch tensor setup and checks, layout transformation reasoning.
- **CN:** 通过 `device`、`layout_2d`、`layout_1d`、`shared_layout`、`matrix`、`gather_indices`、`gather_output`、`gather_expected` 等另外 4 项 准备或更新状态。 调用 `torch.device`、`ttgl.BlockedLayout`、`ttgl.SwizzledSharedLayout`、`torch.arange`、`torch.zeros`、`matrix.flatten` 等另外 1 项 执行测试逻辑。 相关主题：PyTorch 张量准备与校验、布局变换推理。

#### Lines 3049-3050

```python
    for i in range(N):
        scatter_expected[i, i] = scatter_values[i]
```
- **EN:** Iterates across cases or data tiles.
- **CN:** 通过循环覆盖多个用例或数据分块。

#### Lines 3051-3065

```python

    shared_scatter_kernel[(1, )](
        scatter_indices,
        scatter_values,
        scatter_output,
        N=N,
        M=M,
        layout_2d=layout_2d,
        layout_1d=layout_1d,
        shared_layout=shared_layout,
        use_broadcast=False,
        num_warps=num_warps,
    )

    torch.testing.assert_close(scatter_output, scatter_expected)
```
- **EN:** Invokes `torch.testing.assert_close` to execute the test logic. Relevant themes: PyTorch tensor setup and checks, layout transformation reasoning.
- **CN:** 调用 `torch.testing.assert_close` 执行测试逻辑。 相关主题：PyTorch 张量准备与校验、布局变换推理。

### Lines 3066-3083

```python


# ============================================================================
# 2D Native Gather/Scatter Tests
# ============================================================================


@gluon.jit
def gather_2d_kernel(
    matrix_ptr,
    indices_ptr,
    output_ptr,
    N: ttgl.constexpr,
    M: ttgl.constexpr,
    axis: ttgl.constexpr,
    layout_2d: ttgl.constexpr,
    shared_layout: ttgl.constexpr,
):
```
- **EN:** Defines the helper function `gather_2d_kernel`. Decorators: `gluon.jit`. Parameters: `matrix_ptr`, `indices_ptr`, `output_ptr`, `N`, `M`, `axis`, `layout_2d`, `shared_layout`. Key calls include `ttgl.arange`, `ttgl.load`, `ttgl.allocate_shared_memory`, `smem.store`, `smem.gather`, `ttgl.store`, and 1 more. This scope touches layout transformation reasoning.
- **CN:** 定义辅助函数 `gather_2d_kernel`。 装饰器：`gluon.jit`。 参数：`matrix_ptr`、`indices_ptr`、`output_ptr`、`N`、`M`、`axis`、`layout_2d`、`shared_layout`。 关键调用包括 `ttgl.arange`、`ttgl.load`、`ttgl.allocate_shared_memory`、`smem.store`、`smem.gather`、`ttgl.store` 等另外 1 项。 该作用域涉及布局变换推理。

#### Lines 3084-3102

```python
    """Test 2D gather along specified axis."""
    # Load the matrix from global memory [N, M]
    indices_x = ttgl.arange(0, N, layout=ttgl.SliceLayout(dim=1, parent=layout_2d))
    indices_y = ttgl.arange(0, M, layout=ttgl.SliceLayout(dim=0, parent=layout_2d))
    offsets_2d = indices_x[:, None] * M + indices_y[None, :]
    matrix_data = ttgl.load(matrix_ptr + offsets_2d)

    # Store in shared memory
    smem = ttgl.allocate_shared_memory(ttgl.float32, [N, M], layout=shared_layout)
    smem.store(matrix_data)

    # Load indices [N, M] - same rank as source
    indices = ttgl.load(indices_ptr + offsets_2d)

    # Gather along specified axis
    gathered = smem.gather(indices, axis=axis)

    # Store result
    ttgl.store(output_ptr + offsets_2d, gathered)
```
- **EN:** Prepares or updates state through `indices_x`, `indices_y`, `offsets_2d`, `matrix_data`, `smem`, `indices`, `gathered`. Invokes `ttgl.arange`, `ttgl.SliceLayout`, `ttgl.load`, `ttgl.allocate_shared_memory`, `smem.store`, `smem.gather`, and 1 more to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `indices_x`、`indices_y`、`offsets_2d`、`matrix_data`、`smem`、`indices`、`gathered` 准备或更新状态。 调用 `ttgl.arange`、`ttgl.SliceLayout`、`ttgl.load`、`ttgl.allocate_shared_memory`、`smem.store`、`smem.gather` 等另外 1 项 执行测试逻辑。 相关主题：布局变换推理。

### Lines 3103-3106

```python


@pytest.mark.parametrize("N,M,axis", [(32, 32, 0), (32, 32, 1), (64, 64, 0), (64, 64, 1)])
def test_gather_2d_native(N, M, axis):
```
- **EN:** Defines the test function `test_gather_2d_native`. Decorators: `pytest.mark.parametrize('N,M,axis', [(32, 32, 0), (32, 32, 1), (64, 64, 0), (64, 64, 1)])`. Parameters: `N`, `M`, `axis`. Key calls include `pytest.mark.parametrize`, `torch.device`, `torch.zeros`, `ttgl.BlockedLayout`, `ttgl.SwizzledSharedLayout`, `torch.testing.assert_close`, and 3 more. This scope touches pytest parametrization, PyTorch tensor setup and checks, layout transformation reasoning.
- **CN:** 定义测试函数 `test_gather_2d_native`。 装饰器：`pytest.mark.parametrize('N,M,axis', [(32, 32, 0), (32, 32, 1), (64, 64, 0), (64, 64, 1)])`。 参数：`N`、`M`、`axis`。 关键调用包括 `pytest.mark.parametrize`、`torch.device`、`torch.zeros`、`ttgl.BlockedLayout`、`ttgl.SwizzledSharedLayout`、`torch.testing.assert_close` 等另外 3 项。 该作用域涉及pytest 参数化、PyTorch 张量准备与校验、布局变换推理。

#### Lines 3107-3111

```python
    """Test 2D gather along different axes."""
    device = torch.device("cuda")

    # Create a test matrix [N, M]
    matrix = torch.arange(N * M, dtype=torch.float32, device=device).reshape(N, M)
```
- **EN:** Prepares or updates state through `device`, `matrix`. Invokes `torch.device`, `torch.arange` to execute the test logic. Relevant themes: PyTorch tensor setup and checks.
- **CN:** 通过 `device`、`matrix` 准备或更新状态。 调用 `torch.device`、`torch.arange` 执行测试逻辑。 相关主题：PyTorch 张量准备与校验。

#### Lines 3112-3125

```python

    # Create indices [N, M] - each position specifies where to gather from along the axis
    if axis == 0:
        # Each column gathers from a shifted row pattern
        indices = torch.arange(M, dtype=torch.int32, device=device)[None, :].expand(N, M)
        indices = (indices + torch.arange(N, dtype=torch.int32, device=device)[:, None]) % N
        # Expected: result[i, j] = matrix[indices[i, j], j]
        expected = torch.gather(matrix, 0, indices.long())
    else:  # axis == 1
        # Each row gathers from a shifted column pattern
        indices = torch.arange(N, dtype=torch.int32, device=device)[:, None].expand(N, M)
        indices = (indices + torch.arange(M, dtype=torch.int32, device=device)[None, :]) % M
        # Expected: result[i, j] = matrix[i, indices[i, j]]
        expected = torch.gather(matrix, 1, indices.long())
```
- **EN:** Invokes `torch.gather`, `indices.long`, `torch.arange` to execute the test logic. Branches on runtime or test conditions. Relevant themes: PyTorch tensor setup and checks.
- **CN:** 调用 `torch.gather`、`indices.long`、`torch.arange` 执行测试逻辑。 根据运行时或测试条件进行分支。 相关主题：PyTorch 张量准备与校验。

#### Lines 3126-3146

```python

    output = torch.zeros((N, M), dtype=torch.float32, device=device)

    # Create layouts dynamically based on THREADS_PER_WARP
    layout_2d = ttgl.BlockedLayout(size_per_thread=[1, 1], threads_per_warp=[THREADS_PER_WARP // 4, 4],
                                   warps_per_cta=[1, 1], order=[1, 0])
    shared_layout = ttgl.SwizzledSharedLayout(vec=1, per_phase=1, max_phase=1, order=[1, 0])

    gather_2d_kernel[(1, )](
        matrix,
        indices,
        output,
        N=N,
        M=M,
        axis=axis,
        layout_2d=layout_2d,
        shared_layout=shared_layout,
        num_warps=1,
    )

    torch.testing.assert_close(output, expected)
```
- **EN:** Prepares or updates state through `output`, `layout_2d`, `shared_layout`. Invokes `torch.zeros`, `ttgl.BlockedLayout`, `ttgl.SwizzledSharedLayout`, `torch.testing.assert_close` to execute the test logic. Relevant themes: PyTorch tensor setup and checks, layout transformation reasoning.
- **CN:** 通过 `output`、`layout_2d`、`shared_layout` 准备或更新状态。 调用 `torch.zeros`、`ttgl.BlockedLayout`、`ttgl.SwizzledSharedLayout`、`torch.testing.assert_close` 执行测试逻辑。 相关主题：PyTorch 张量准备与校验、布局变换推理。

### Lines 3147-3159

```python


@gluon.jit
def scatter_2d_kernel(
    indices_ptr,
    values_ptr,
    output_ptr,
    N: ttgl.constexpr,
    M: ttgl.constexpr,
    axis: ttgl.constexpr,
    layout_2d: ttgl.constexpr,
    shared_layout: ttgl.constexpr,
):
```
- **EN:** Defines the helper function `scatter_2d_kernel`. Decorators: `gluon.jit`. Parameters: `indices_ptr`, `values_ptr`, `output_ptr`, `N`, `M`, `axis`, `layout_2d`, `shared_layout`. Key calls include `ttgl.allocate_shared_memory`, `ttgl.arange`, `ttgl.zeros`, `smem.store`, `ttgl.load`, `smem.scatter`, and 3 more. This scope touches layout transformation reasoning.
- **CN:** 定义辅助函数 `scatter_2d_kernel`。 装饰器：`gluon.jit`。 参数：`indices_ptr`、`values_ptr`、`output_ptr`、`N`、`M`、`axis`、`layout_2d`、`shared_layout`。 关键调用包括 `ttgl.allocate_shared_memory`、`ttgl.arange`、`ttgl.zeros`、`smem.store`、`ttgl.load`、`smem.scatter` 等另外 3 项。 该作用域涉及布局变换推理。

#### Lines 3160-3179

```python
    """Test 2D scatter along specified axis."""
    # Initialize shared memory to zero
    smem = ttgl.allocate_shared_memory(ttgl.float32, [N, M], layout=shared_layout)

    indices_x = ttgl.arange(0, N, layout=ttgl.SliceLayout(dim=1, parent=layout_2d))
    indices_y = ttgl.arange(0, M, layout=ttgl.SliceLayout(dim=0, parent=layout_2d))
    offsets_2d = indices_x[:, None] * M + indices_y[None, :]
    zeros = ttgl.zeros([N, M], ttgl.float32, layout=layout_2d)
    smem.store(zeros)

    # Load indices [N, M] and values [N, M]
    indices = ttgl.load(indices_ptr + offsets_2d)
    values = ttgl.load(values_ptr + offsets_2d)

    # Scatter along specified axis
    smem.scatter(values, indices, axis=axis)

    # Read back the result
    result = smem.load(layout=layout_2d)
    ttgl.store(output_ptr + offsets_2d, result)
```
- **EN:** Prepares or updates state through `smem`, `indices_x`, `indices_y`, `offsets_2d`, `zeros`, `indices`, `values`, `result`. Invokes `ttgl.allocate_shared_memory`, `ttgl.arange`, `ttgl.SliceLayout`, `ttgl.zeros`, `smem.store`, `ttgl.load`, and 3 more to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `smem`、`indices_x`、`indices_y`、`offsets_2d`、`zeros`、`indices`、`values`、`result` 准备或更新状态。 调用 `ttgl.allocate_shared_memory`、`ttgl.arange`、`ttgl.SliceLayout`、`ttgl.zeros`、`smem.store`、`ttgl.load` 等另外 3 项 执行测试逻辑。 相关主题：布局变换推理。

### Lines 3180-3183

```python


@pytest.mark.parametrize("N,M,axis", [(32, 32, 0), (32, 32, 1)])
def test_scatter_2d_native(N, M, axis):
```
- **EN:** Defines the test function `test_scatter_2d_native`. Decorators: `pytest.mark.parametrize('N,M,axis', [(32, 32, 0), (32, 32, 1)])`. Parameters: `N`, `M`, `axis`. Key calls include `pytest.mark.parametrize`, `torch.device`, `torch.zeros`, `expected.scatter_`, `ttgl.BlockedLayout`, `ttgl.SwizzledSharedLayout`, and 3 more. This scope touches pytest parametrization, PyTorch tensor setup and checks, layout transformation reasoning.
- **CN:** 定义测试函数 `test_scatter_2d_native`。 装饰器：`pytest.mark.parametrize('N,M,axis', [(32, 32, 0), (32, 32, 1)])`。 参数：`N`、`M`、`axis`。 关键调用包括 `pytest.mark.parametrize`、`torch.device`、`torch.zeros`、`expected.scatter_`、`ttgl.BlockedLayout`、`ttgl.SwizzledSharedLayout` 等另外 3 项。 该作用域涉及pytest 参数化、PyTorch 张量准备与校验、布局变换推理。

#### Lines 3184-3185

```python
    """Test 2D scatter along different axes."""
    device = torch.device("cuda")
```
- **EN:** Prepares or updates state through `device`. Invokes `torch.device` to execute the test logic. Relevant themes: PyTorch tensor setup and checks.
- **CN:** 通过 `device` 准备或更新状态。 调用 `torch.device` 执行测试逻辑。 相关主题：PyTorch 张量准备与校验。

#### Lines 3186-3193

```python

    # Create indices [N, M] - reverse pattern for scatter
    if axis == 0:
        indices = torch.arange(M, dtype=torch.int32, device=device)[None, :].expand(N, M)
        indices = (N - 1 - indices - torch.arange(N, dtype=torch.int32, device=device)[:, None]) % N
    else:  # axis == 1
        indices = torch.arange(N, dtype=torch.int32, device=device)[:, None].expand(N, M)
        indices = (M - 1 - indices - torch.arange(M, dtype=torch.int32, device=device)[None, :]) % M
```
- **EN:** Invokes `torch.arange` to execute the test logic. Branches on runtime or test conditions. Relevant themes: PyTorch tensor setup and checks.
- **CN:** 调用 `torch.arange` 执行测试逻辑。 根据运行时或测试条件进行分支。 相关主题：PyTorch 张量准备与校验。

#### Lines 3194-3221

```python

    # Create values to scatter
    values = torch.arange(N * M, dtype=torch.float32, device=device).reshape(N, M) + 100.0

    output = torch.zeros((N, M), dtype=torch.float32, device=device)

    # Expected: scatter values according to indices
    expected = torch.zeros((N, M), dtype=torch.float32, device=device)
    expected.scatter_(axis, indices.long(), values)

    # Create layouts dynamically based on THREADS_PER_WARP
    layout_2d = ttgl.BlockedLayout(size_per_thread=[1, 1], threads_per_warp=[THREADS_PER_WARP // 4, 4],
                                   warps_per_cta=[1, 1], order=[1, 0])
    shared_layout = ttgl.SwizzledSharedLayout(vec=1, per_phase=1, max_phase=1, order=[1, 0])

    scatter_2d_kernel[(1, )](
        indices,
        values,
        output,
        N=N,
        M=M,
        axis=axis,
        layout_2d=layout_2d,
        shared_layout=shared_layout,
        num_warps=1,
    )

    torch.testing.assert_close(output, expected)
```
- **EN:** Prepares or updates state through `values`, `output`, `expected`, `layout_2d`, `shared_layout`. Invokes `torch.arange`, `torch.zeros`, `expected.scatter_`, `indices.long`, `ttgl.BlockedLayout`, `ttgl.SwizzledSharedLayout`, and 1 more to execute the test logic. Relevant themes: PyTorch tensor setup and checks, layout transformation reasoning.
- **CN:** 通过 `values`、`output`、`expected`、`layout_2d`、`shared_layout` 准备或更新状态。 调用 `torch.arange`、`torch.zeros`、`expected.scatter_`、`indices.long`、`ttgl.BlockedLayout`、`ttgl.SwizzledSharedLayout` 等另外 1 项 执行测试逻辑。 相关主题：PyTorch 张量准备与校验、布局变换推理。

### Lines 3222-3240

```python


# ============================================================================
# 3D Gather/Scatter Tests
# ============================================================================


@gluon.jit
def gather_3d_kernel(
    tensor_ptr,
    indices_ptr,
    output_ptr,
    N: ttgl.constexpr,
    M: ttgl.constexpr,
    P: ttgl.constexpr,
    axis: ttgl.constexpr,
    layout_3d: ttgl.constexpr,
    shared_layout: ttgl.constexpr,
):
```
- **EN:** Defines the helper function `gather_3d_kernel`. Decorators: `gluon.jit`. Parameters: `tensor_ptr`, `indices_ptr`, `output_ptr`, `N`, `M`, `P`, `axis`, `layout_3d`, and 1 more. Key calls include `ttgl.set_auto_layout`, `ttgl.load`, `ttgl.allocate_shared_memory`, `smem.store`, `smem.gather`, `ttgl.store`, and 1 more. This scope touches layout transformation reasoning.
- **CN:** 定义辅助函数 `gather_3d_kernel`。 装饰器：`gluon.jit`。 参数：`tensor_ptr`、`indices_ptr`、`output_ptr`、`N`、`M`、`P`、`axis`、`layout_3d` 等另外 1 项。 关键调用包括 `ttgl.set_auto_layout`、`ttgl.load`、`ttgl.allocate_shared_memory`、`smem.store`、`smem.gather`、`ttgl.store` 等另外 1 项。 该作用域涉及布局变换推理。

#### Lines 3241-3263

```python
    """Test 3D gather along specified axis."""
    # Load the tensor from global memory [N, M, P]
    idx_n = ttgl.arange(0, N)[:, None, None]
    idx_m = ttgl.arange(0, M)[None, :, None]
    idx_p = ttgl.arange(0, P)[None, None, :]

    offsets_3d = idx_n * (M * P) + idx_m * P + idx_p
    offsets_3d = ttgl.set_auto_layout(offsets_3d, layout_3d)

    tensor_data = ttgl.load(tensor_ptr + offsets_3d)

    # Store in shared memory
    smem = ttgl.allocate_shared_memory(ttgl.float32, [N, M, P], layout=shared_layout)
    smem.store(tensor_data)

    # Load indices [N, M, P] - same rank as source
    indices_data = ttgl.load(indices_ptr + offsets_3d)

    # Gather along specified axis
    gathered = smem.gather(indices_data, axis=axis)

    # Store result
    ttgl.store(output_ptr + offsets_3d, gathered)
```
- **EN:** Prepares or updates state through `idx_n`, `idx_m`, `idx_p`, `offsets_3d`, `tensor_data`, `smem`, `indices_data`, `gathered`. Invokes `ttgl.arange`, `ttgl.set_auto_layout`, `ttgl.load`, `ttgl.allocate_shared_memory`, `smem.store`, `smem.gather`, and 1 more to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `idx_n`、`idx_m`、`idx_p`、`offsets_3d`、`tensor_data`、`smem`、`indices_data`、`gathered` 准备或更新状态。 调用 `ttgl.arange`、`ttgl.set_auto_layout`、`ttgl.load`、`ttgl.allocate_shared_memory`、`smem.store`、`smem.gather` 等另外 1 项 执行测试逻辑。 相关主题：布局变换推理。

### Lines 3264-3267

```python


@pytest.mark.parametrize("N,M,P,axis", [(16, 8, 4, 0), (16, 8, 4, 1), (16, 8, 4, 2)])
def test_gather_3d_native(N, M, P, axis):
```
- **EN:** Defines the test function `test_gather_3d_native`. Decorators: `pytest.mark.parametrize('N,M,P,axis', [(16, 8, 4, 0), (16, 8, 4, 1), (16, 8, 4, 2)])`. Parameters: `N`, `M`, `P`, `axis`. Key calls include `pytest.mark.parametrize`, `torch.device`, `indices.contiguous`, `torch.gather`, `torch.zeros`, `ttgl.BlockedLayout`, and 4 more. This scope touches pytest parametrization, PyTorch tensor setup and checks, layout transformation reasoning.
- **CN:** 定义测试函数 `test_gather_3d_native`。 装饰器：`pytest.mark.parametrize('N,M,P,axis', [(16, 8, 4, 0), (16, 8, 4, 1), (16, 8, 4, 2)])`。 参数：`N`、`M`、`P`、`axis`。 关键调用包括 `pytest.mark.parametrize`、`torch.device`、`indices.contiguous`、`torch.gather`、`torch.zeros`、`ttgl.BlockedLayout` 等另外 4 项。 该作用域涉及pytest 参数化、PyTorch 张量准备与校验、布局变换推理。

#### Lines 3268-3272

```python
    """Test 3D gather along different axes."""
    device = torch.device("cuda")

    # Create a test tensor [N, M, P]
    tensor = torch.arange(N * M * P, dtype=torch.float32, device=device).reshape(N, M, P)
```
- **EN:** Prepares or updates state through `device`, `tensor`. Invokes `torch.device`, `torch.arange` to execute the test logic. Relevant themes: PyTorch tensor setup and checks.
- **CN:** 通过 `device`、`tensor` 准备或更新状态。 调用 `torch.device`、`torch.arange` 执行测试逻辑。 相关主题：PyTorch 张量准备与校验。

#### Lines 3273-3288

```python

    # Create indices [N, M, P] - each position specifies where to gather from along the axis
    if axis == 0:
        # Pattern for gathering along first dimension
        base = torch.arange(M * P, dtype=torch.int32, device=device).reshape(1, M, P)
        offset = torch.arange(N, dtype=torch.int32, device=device).reshape(N, 1, 1)
        indices = (base + offset) % N
    elif axis == 1:
        # Pattern for gathering along second dimension
        base = torch.arange(N, dtype=torch.int32, device=device).reshape(N, 1, 1)
        offset = torch.arange(P, dtype=torch.int32, device=device).reshape(1, 1, P)
        indices = ((base + offset) % M).expand(N, M, P).contiguous()
    else:  # axis == 2
        # Pattern for gathering along third dimension
        base = torch.arange(N * M, dtype=torch.int32, device=device).reshape(N, M, 1)
        indices = (base % P).expand(N, M, P).contiguous()
```
- **EN:** Invokes `torch.arange` to execute the test logic. Branches on runtime or test conditions. Relevant themes: PyTorch tensor setup and checks.
- **CN:** 调用 `torch.arange` 执行测试逻辑。 根据运行时或测试条件进行分支。 相关主题：PyTorch 张量准备与校验。

#### Lines 3289-3316

```python

    # Ensure indices is contiguous in C-style layout
    indices = indices.contiguous()

    # Compute expected result using torch.gather
    expected = torch.gather(tensor, axis, indices.long())

    output = torch.zeros((N, M, P), dtype=torch.float32, device=device)

    # Create layouts dynamically based on THREADS_PER_WARP
    layout_3d = ttgl.BlockedLayout(size_per_thread=[1, 1, 1], threads_per_warp=[4, 4, THREADS_PER_WARP // 16],
                                   warps_per_cta=[1, 1, 1], order=[2, 1, 0])
    shared_layout = ttgl.SwizzledSharedLayout(vec=1, per_phase=1, max_phase=1, order=[2, 1, 0])

    gather_3d_kernel[(1, )](
        tensor,
        indices,
        output,
        N=N,
        M=M,
        P=P,
        axis=axis,
        layout_3d=layout_3d,
        shared_layout=shared_layout,
        num_warps=1,
    )

    torch.testing.assert_close(output, expected)
```
- **EN:** Prepares or updates state through `indices`, `expected`, `output`, `layout_3d`, `shared_layout`. Invokes `indices.contiguous`, `torch.gather`, `indices.long`, `torch.zeros`, `ttgl.BlockedLayout`, `ttgl.SwizzledSharedLayout`, and 1 more to execute the test logic. Relevant themes: PyTorch tensor setup and checks, layout transformation reasoning.
- **CN:** 通过 `indices`、`expected`、`output`、`layout_3d`、`shared_layout` 准备或更新状态。 调用 `indices.contiguous`、`torch.gather`、`indices.long`、`torch.zeros`、`ttgl.BlockedLayout`、`ttgl.SwizzledSharedLayout` 等另外 1 项 执行测试逻辑。 相关主题：PyTorch 张量准备与校验、布局变换推理。

### Lines 3317-3330

```python


@gluon.jit
def scatter_3d_kernel(
    indices_ptr,
    values_ptr,
    output_ptr,
    N: ttgl.constexpr,
    M: ttgl.constexpr,
    P: ttgl.constexpr,
    axis: ttgl.constexpr,
    layout_3d: ttgl.constexpr,
    shared_layout: ttgl.constexpr,
):
```
- **EN:** Defines the helper function `scatter_3d_kernel`. Decorators: `gluon.jit`. Parameters: `indices_ptr`, `values_ptr`, `output_ptr`, `N`, `M`, `P`, `axis`, `layout_3d`, and 1 more. Key calls include `ttgl.set_auto_layout`, `ttgl.allocate_shared_memory`, `ttgl.full`, `smem.store`, `ttgl.load`, `smem.scatter`, and 3 more. This scope touches layout transformation reasoning.
- **CN:** 定义辅助函数 `scatter_3d_kernel`。 装饰器：`gluon.jit`。 参数：`indices_ptr`、`values_ptr`、`output_ptr`、`N`、`M`、`P`、`axis`、`layout_3d` 等另外 1 项。 关键调用包括 `ttgl.set_auto_layout`、`ttgl.allocate_shared_memory`、`ttgl.full`、`smem.store`、`ttgl.load`、`smem.scatter` 等另外 3 项。 该作用域涉及布局变换推理。

#### Lines 3331-3353

```python
    """Test 3D scatter along specified axis."""
    idx_n = ttgl.arange(0, N)[:, None, None]
    idx_m = ttgl.arange(0, M)[None, :, None]
    idx_p = ttgl.arange(0, P)[None, None, :]

    offsets_3d = idx_n * (M * P) + idx_m * P + idx_p
    offsets_3d = ttgl.set_auto_layout(offsets_3d, layout_3d)

    # Initialize shared memory to zero
    smem = ttgl.allocate_shared_memory(ttgl.float32, [N, M, P], layout=shared_layout)
    zeros = ttgl.full([N, M, P], 0.0, ttgl.float32, layout=layout_3d)
    smem.store(zeros)

    # Load indices [N, M, P] and values [N, M, P]
    indices_data = ttgl.load(indices_ptr + offsets_3d)
    values_data = ttgl.load(values_ptr + offsets_3d)

    # Scatter along specified axis
    smem.scatter(values_data, indices_data, axis=axis)

    # Read back the result
    result = smem.load(layout=layout_3d)
    ttgl.store(output_ptr + offsets_3d, result)
```
- **EN:** Prepares or updates state through `idx_n`, `idx_m`, `idx_p`, `offsets_3d`, `smem`, `zeros`, `indices_data`, `values_data`, and 1 more. Invokes `ttgl.arange`, `ttgl.set_auto_layout`, `ttgl.allocate_shared_memory`, `ttgl.full`, `smem.store`, `ttgl.load`, and 3 more to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `idx_n`、`idx_m`、`idx_p`、`offsets_3d`、`smem`、`zeros`、`indices_data`、`values_data` 等另外 1 项 准备或更新状态。 调用 `ttgl.arange`、`ttgl.set_auto_layout`、`ttgl.allocate_shared_memory`、`ttgl.full`、`smem.store`、`ttgl.load` 等另外 3 项 执行测试逻辑。 相关主题：布局变换推理。

### Lines 3354-3357

```python


@pytest.mark.parametrize("N,M,P,axis", [(16, 8, 4, 0), (16, 8, 4, 1), (16, 8, 4, 2)])
def test_scatter_3d_native(N, M, P, axis):
```
- **EN:** Defines the test function `test_scatter_3d_native`. Decorators: `pytest.mark.parametrize('N,M,P,axis', [(16, 8, 4, 0), (16, 8, 4, 1), (16, 8, 4, 2)])`. Parameters: `N`, `M`, `P`, `axis`. Key calls include `pytest.mark.parametrize`, `torch.device`, `indices.contiguous`, `torch.zeros`, `expected.scatter_`, `ttgl.BlockedLayout`, and 4 more. This scope touches pytest parametrization, PyTorch tensor setup and checks, layout transformation reasoning.
- **CN:** 定义测试函数 `test_scatter_3d_native`。 装饰器：`pytest.mark.parametrize('N,M,P,axis', [(16, 8, 4, 0), (16, 8, 4, 1), (16, 8, 4, 2)])`。 参数：`N`、`M`、`P`、`axis`。 关键调用包括 `pytest.mark.parametrize`、`torch.device`、`indices.contiguous`、`torch.zeros`、`expected.scatter_`、`ttgl.BlockedLayout` 等另外 4 项。 该作用域涉及pytest 参数化、PyTorch 张量准备与校验、布局变换推理。

#### Lines 3358-3359

```python
    """Test 3D scatter along different axes."""
    device = torch.device("cuda")
```
- **EN:** Prepares or updates state through `device`. Invokes `torch.device` to execute the test logic. Relevant themes: PyTorch tensor setup and checks.
- **CN:** 通过 `device` 准备或更新状态。 调用 `torch.device` 执行测试逻辑。 相关主题：PyTorch 张量准备与校验。

#### Lines 3360-3379

```python

    # Create indices [N, M, P] that form a permutation along the scatter axis
    if axis == 0:
        # For axis 0: permute N dimension, keeping (M, P) coordinates fixed
        # Each (j, k) position has a unique permutation of N indices
        base = torch.arange(M * P, dtype=torch.int32, device=device).reshape(1, M, P)
        offset = torch.arange(N, dtype=torch.int32, device=device).reshape(N, 1, 1)
        indices = ((N - 1 - base - offset) % N).contiguous()
    elif axis == 1:
        # For axis 1: permute M dimension, keeping (N, P) coordinates fixed
        # Each (i, k) position has a unique permutation of M indices
        base = torch.arange(N * P, dtype=torch.int32, device=device).reshape(N, 1, P)
        offset = torch.arange(M, dtype=torch.int32, device=device).reshape(1, M, 1)
        indices = ((M - 1 - base - offset) % M).contiguous()
    else:  # axis == 2
        # For axis 2: permute P dimension, keeping (N, M) coordinates fixed
        # Each (i, j) position has a unique permutation of P indices
        base = torch.arange(N * M, dtype=torch.int32, device=device).reshape(N, M, 1)
        offset = torch.arange(P, dtype=torch.int32, device=device).reshape(1, 1, P)
        indices = ((P - 1 - base - offset) % P).contiguous()
```
- **EN:** Invokes `torch.arange` to execute the test logic. Branches on runtime or test conditions. Relevant themes: PyTorch tensor setup and checks.
- **CN:** 调用 `torch.arange` 执行测试逻辑。 根据运行时或测试条件进行分支。 相关主题：PyTorch 张量准备与校验。

#### Lines 3380-3411

```python

    # Ensure indices is contiguous
    indices = indices.contiguous()

    # Create values to scatter
    values = (torch.arange(N * M * P, dtype=torch.float32, device=device).reshape(N, M, P) + 200.0).contiguous()

    output = torch.zeros((N, M, P), dtype=torch.float32, device=device)

    # Expected: scatter values according to indices
    expected = torch.zeros((N, M, P), dtype=torch.float32, device=device)
    expected.scatter_(axis, indices.long(), values)

    # Create layouts dynamically based on THREADS_PER_WARP
    layout_3d = ttgl.BlockedLayout(size_per_thread=[1, 1, 1], threads_per_warp=[4, 4, THREADS_PER_WARP // 16],
                                   warps_per_cta=[1, 1, 1], order=[2, 1, 0])
    shared_layout = ttgl.SwizzledSharedLayout(vec=1, per_phase=1, max_phase=1, order=[2, 1, 0])

    scatter_3d_kernel[(1, )](
        indices,
        values,
        output,
        N=N,
        M=M,
        P=P,
        axis=axis,
        layout_3d=layout_3d,
        shared_layout=shared_layout,
        num_warps=1,
    )

    torch.testing.assert_close(output, expected)
```
- **EN:** Prepares or updates state through `indices`, `values`, `output`, `expected`, `layout_3d`, `shared_layout`. Invokes `indices.contiguous`, `torch.arange`, `torch.zeros`, `expected.scatter_`, `indices.long`, `ttgl.BlockedLayout`, and 2 more to execute the test logic. Relevant themes: PyTorch tensor setup and checks, layout transformation reasoning.
- **CN:** 通过 `indices`、`values`、`output`、`expected`、`layout_3d`、`shared_layout` 准备或更新状态。 调用 `indices.contiguous`、`torch.arange`、`torch.zeros`、`expected.scatter_`、`indices.long`、`ttgl.BlockedLayout` 等另外 2 项 执行测试逻辑。 相关主题：PyTorch 张量准备与校验、布局变换推理。

### Lines 3412-3433

```python


# =============================================================================
# Subslice Tests (2D slicing along individual dimensions)
# =============================================================================


@gluon.jit
def gather_subslice_2d_kernel(
    matrix_ptr,
    indices_ptr,
    output_ptr,
    M: ttgl.constexpr,
    N: ttgl.constexpr,
    SLICE_M_OFFSET: ttgl.constexpr,
    SLICE_N_OFFSET: ttgl.constexpr,
    SLICE_M: ttgl.constexpr,
    SLICE_N: ttgl.constexpr,
    layout_full: ttgl.constexpr,
    layout_slice: ttgl.constexpr,
    shared_layout: ttgl.constexpr,
):
```
- **EN:** Defines the helper function `gather_subslice_2d_kernel`. Decorators: `gluon.jit`. Parameters: `matrix_ptr`, `indices_ptr`, `output_ptr`, `M`, `N`, `SLICE_M_OFFSET`, `SLICE_N_OFFSET`, `SLICE_M`, and 4 more. Key calls include `ttgl.load`, `ttgl.allocate_shared_memory`, `smem.store`, `smem_slice.gather`, `ttgl.store`, `ttgl.arange`, and 2 more. This scope touches tensor/descriptor metadata, layout transformation reasoning.
- **CN:** 定义辅助函数 `gather_subslice_2d_kernel`。 装饰器：`gluon.jit`。 参数：`matrix_ptr`、`indices_ptr`、`output_ptr`、`M`、`N`、`SLICE_M_OFFSET`、`SLICE_N_OFFSET`、`SLICE_M` 等另外 4 项。 关键调用包括 `ttgl.load`、`ttgl.allocate_shared_memory`、`smem.store`、`smem_slice.gather`、`ttgl.store`、`ttgl.arange` 等另外 2 项。 该作用域涉及张量/描述符元数据、布局变换推理。

#### Lines 3434-3457

```python
    """Gather from a 2D subsliced shared memory descriptor."""
    # Load full matrix into shared memory
    offs_m = ttgl.arange(0, M, layout=ttgl.SliceLayout(1, layout_full))[:, None]
    offs_n = ttgl.arange(0, N, layout=ttgl.SliceLayout(0, layout_full))[None, :]
    in_offs = offs_m * N + offs_n
    in_data = ttgl.load(matrix_ptr + in_offs)

    smem = ttgl.allocate_shared_memory(ttgl.float32, [M, N], layout=shared_layout)
    smem.store(in_data)

    # Create 2D subslice
    smem_slice = smem.slice(SLICE_M_OFFSET, SLICE_M, dim=0).slice(SLICE_N_OFFSET, SLICE_N, dim=1)

    # Load indices for gathering within the slice
    slice_offs_m = ttgl.arange(0, SLICE_M, layout=ttgl.SliceLayout(1, layout_slice))[:, None]
    slice_offs_n = ttgl.arange(0, SLICE_N, layout=ttgl.SliceLayout(0, layout_slice))[None, :]
    idx_offs = slice_offs_m * SLICE_N + slice_offs_n
    indices = ttgl.load(indices_ptr + idx_offs)

    # Gather along axis 0: result[i, j] = smem_slice[indices[i, j], j]
    gathered = smem_slice.gather(indices, axis=0)

    # Store result
    ttgl.store(output_ptr + idx_offs, gathered)
```
- **EN:** Prepares or updates state through `offs_m`, `offs_n`, `in_offs`, `in_data`, `smem`, `smem_slice`, `slice_offs_m`, `slice_offs_n`, and 3 more. Invokes `ttgl.arange`, `ttgl.SliceLayout`, `ttgl.load`, `ttgl.allocate_shared_memory`, `smem.store`, `smem.slice`, and 2 more to execute the test logic. Relevant themes: tensor/descriptor metadata, layout transformation reasoning.
- **CN:** 通过 `offs_m`、`offs_n`、`in_offs`、`in_data`、`smem`、`smem_slice`、`slice_offs_m`、`slice_offs_n` 等另外 3 项 准备或更新状态。 调用 `ttgl.arange`、`ttgl.SliceLayout`、`ttgl.load`、`ttgl.allocate_shared_memory`、`smem.store`、`smem.slice` 等另外 2 项 执行测试逻辑。 相关主题：张量/描述符元数据、布局变换推理。

### Lines 3458-3466

```python


@pytest.mark.parametrize("M,N,slice_m_offset,slice_n_offset,slice_m,slice_n", [
    # Offset must be a multiple of tile (slice) size for each dimension
    (64, 64, 48, 16, 16, 16),  # offset 48 % 16 == 0, offset 16 % 16 == 0
    (64, 64, 32, 48, 32, 16),  # offset 32 % 32 == 0, offset 48 % 16 == 0
    (64, 64, 48, 32, 16, 32),  # offset 48 % 16 == 0, offset 32 % 32 == 0
])
def test_gather_subslice_2d(M, N, slice_m_offset, slice_n_offset, slice_m, slice_n):
```
- **EN:** Defines the test function `test_gather_subslice_2d`. Decorators: `pytest.mark.parametrize('M,N,slice_m_offset,slice_n_offset,slice_m,slice_n', [(64, 64, 48, 16, 16, 16), (64, 64, 32, 48, 32, 16), (64, 64, 48, 32, 16, 32)])`. Parameters: `M`, `N`, `slice_m_offset`, `slice_n_offset`, `slice_m`, `slice_n`. Key calls include `pytest.mark.parametrize`, `torch.device`, `torch.zeros`, `torch.gather`, `ttgl.BlockedLayout`, `ttgl.SwizzledSharedLayout`, and 3 more. This scope touches pytest parametrization, PyTorch tensor setup and checks, tensor/descriptor metadata, layout transformation reasoning.
- **CN:** 定义测试函数 `test_gather_subslice_2d`。 装饰器：`pytest.mark.parametrize('M,N,slice_m_offset,slice_n_offset,slice_m,slice_n', [(64, 64, 48, 16, 16, 16), (64, 64, 32, 48, 32, 16), (64, 64, 48, 32, 16, 32)])`。 参数：`M`、`N`、`slice_m_offset`、`slice_n_offset`、`slice_m`、`slice_n`。 关键调用包括 `pytest.mark.parametrize`、`torch.device`、`torch.zeros`、`torch.gather`、`ttgl.BlockedLayout`、`ttgl.SwizzledSharedLayout` 等另外 3 项。 该作用域涉及pytest 参数化、PyTorch 张量准备与校验、张量/描述符元数据、布局变换推理。

#### Lines 3467-3516

```python
    """Test gathering from a 2D subsliced shared memory descriptor."""
    device = torch.device("cuda")

    # Create input matrix
    matrix = torch.arange(M * N, dtype=torch.float32, device=device).reshape(M, N)

    # Create indices for gather (within the slice dimensions)
    # Each position gathers from a shifted row
    indices = torch.arange(slice_n, dtype=torch.int32, device=device)[None, :].expand(slice_m, slice_n)
    indices = (indices + torch.arange(slice_m, dtype=torch.int32, device=device)[:, None]) % slice_m

    output = torch.zeros((slice_m, slice_n), dtype=torch.float32, device=device)

    # Expected: gather from the subslice
    subslice = matrix[slice_m_offset:slice_m_offset + slice_m, slice_n_offset:slice_n_offset + slice_n]
    expected = torch.gather(subslice, 0, indices.long())

    # Layouts
    layout_full = ttgl.BlockedLayout(
        size_per_thread=[1, 1],
        threads_per_warp=[THREADS_PER_WARP // 4, 4],
        warps_per_cta=[1, 1],
        order=[1, 0],
    )
    layout_slice = ttgl.BlockedLayout(
        size_per_thread=[1, 1],
        threads_per_warp=[THREADS_PER_WARP // 4, 4],
        warps_per_cta=[1, 1],
        order=[1, 0],
    )
    # Use non-swizzled layout for subslicing
    shared_layout = ttgl.SwizzledSharedLayout(vec=1, per_phase=1, max_phase=1, order=[1, 0])

    gather_subslice_2d_kernel[(1, )](
        matrix,
        indices,
        output,
        M=M,
        N=N,
        SLICE_M_OFFSET=slice_m_offset,
        SLICE_N_OFFSET=slice_n_offset,
        SLICE_M=slice_m,
        SLICE_N=slice_n,
        layout_full=layout_full,
        layout_slice=layout_slice,
        shared_layout=shared_layout,
        num_warps=1,
    )

    torch.testing.assert_close(output, expected)
```
- **EN:** Prepares or updates state through `device`, `matrix`, `indices`, `output`, `subslice`, `expected`, `layout_full`, `layout_slice`, and 1 more. Invokes `torch.device`, `torch.arange`, `torch.zeros`, `torch.gather`, `indices.long`, `ttgl.BlockedLayout`, and 2 more to execute the test logic. Relevant themes: PyTorch tensor setup and checks, tensor/descriptor metadata, layout transformation reasoning.
- **CN:** 通过 `device`、`matrix`、`indices`、`output`、`subslice`、`expected`、`layout_full`、`layout_slice` 等另外 1 项 准备或更新状态。 调用 `torch.device`、`torch.arange`、`torch.zeros`、`torch.gather`、`indices.long`、`ttgl.BlockedLayout` 等另外 2 项 执行测试逻辑。 相关主题：PyTorch 张量准备与校验、张量/描述符元数据、布局变换推理。

### Lines 3517-3533

```python


@gluon.jit
def scatter_subslice_2d_kernel(
    indices_ptr,
    values_ptr,
    output_ptr,
    M: ttgl.constexpr,
    N: ttgl.constexpr,
    SLICE_M_OFFSET: ttgl.constexpr,
    SLICE_N_OFFSET: ttgl.constexpr,
    SLICE_M: ttgl.constexpr,
    SLICE_N: ttgl.constexpr,
    layout_full: ttgl.constexpr,
    layout_slice: ttgl.constexpr,
    shared_layout: ttgl.constexpr,
):
```
- **EN:** Defines the helper function `scatter_subslice_2d_kernel`. Decorators: `gluon.jit`. Parameters: `indices_ptr`, `values_ptr`, `output_ptr`, `M`, `N`, `SLICE_M_OFFSET`, `SLICE_N_OFFSET`, `SLICE_M`, and 4 more. Key calls include `ttgl.full`, `ttgl.allocate_shared_memory`, `smem.store`, `ttgl.load`, `smem_slice.scatter`, `smem.load`, and 4 more. This scope touches tensor/descriptor metadata, layout transformation reasoning.
- **CN:** 定义辅助函数 `scatter_subslice_2d_kernel`。 装饰器：`gluon.jit`。 参数：`indices_ptr`、`values_ptr`、`output_ptr`、`M`、`N`、`SLICE_M_OFFSET`、`SLICE_N_OFFSET`、`SLICE_M` 等另外 4 项。 关键调用包括 `ttgl.full`、`ttgl.allocate_shared_memory`、`smem.store`、`ttgl.load`、`smem_slice.scatter`、`smem.load` 等另外 4 项。 该作用域涉及张量/描述符元数据、布局变换推理。

#### Lines 3534-3559

```python
    """Scatter to a 2D subsliced shared memory descriptor."""
    # Initialize shared memory with -1
    offs_m = ttgl.arange(0, M, layout=ttgl.SliceLayout(1, layout_full))[:, None]
    offs_n = ttgl.arange(0, N, layout=ttgl.SliceLayout(0, layout_full))[None, :]
    full_offs = offs_m * N + offs_n
    init_data = ttgl.full([M, N], -1.0, dtype=ttgl.float32, layout=layout_full)

    smem = ttgl.allocate_shared_memory(ttgl.float32, [M, N], layout=shared_layout)
    smem.store(init_data)

    # Create 2D subslice
    smem_slice = smem.slice(SLICE_M_OFFSET, SLICE_M, dim=0).slice(SLICE_N_OFFSET, SLICE_N, dim=1)

    # Load indices and values for scattering within the slice
    slice_offs_m = ttgl.arange(0, SLICE_M, layout=ttgl.SliceLayout(1, layout_slice))[:, None]
    slice_offs_n = ttgl.arange(0, SLICE_N, layout=ttgl.SliceLayout(0, layout_slice))[None, :]
    idx_offs = slice_offs_m * SLICE_N + slice_offs_n
    indices = ttgl.load(indices_ptr + idx_offs)
    values = ttgl.load(values_ptr + idx_offs)

    # Scatter along axis 0: smem_slice[indices[i, j], j] = values[i, j]
    smem_slice.scatter(values, indices, axis=0)

    # Load back full matrix
    result = smem.load(layout=layout_full)
    ttgl.store(output_ptr + full_offs, result)
```
- **EN:** Prepares or updates state through `offs_m`, `offs_n`, `full_offs`, `init_data`, `smem`, `smem_slice`, `slice_offs_m`, `slice_offs_n`, and 4 more. Invokes `ttgl.arange`, `ttgl.SliceLayout`, `ttgl.full`, `ttgl.allocate_shared_memory`, `smem.store`, `smem.slice`, and 4 more to execute the test logic. Relevant themes: tensor/descriptor metadata, layout transformation reasoning.
- **CN:** 通过 `offs_m`、`offs_n`、`full_offs`、`init_data`、`smem`、`smem_slice`、`slice_offs_m`、`slice_offs_n` 等另外 4 项 准备或更新状态。 调用 `ttgl.arange`、`ttgl.SliceLayout`、`ttgl.full`、`ttgl.allocate_shared_memory`、`smem.store`、`smem.slice` 等另外 4 项 执行测试逻辑。 相关主题：张量/描述符元数据、布局变换推理。

### Lines 3560-3567

```python


@pytest.mark.parametrize("M,N,slice_m_offset,slice_n_offset,slice_m,slice_n", [
    # Offset must be a multiple of tile (slice) size for each dimension
    (64, 64, 48, 16, 16, 16),  # offset 48 % 16 == 0, offset 16 % 16 == 0
    (64, 64, 32, 48, 32, 16),  # offset 32 % 32 == 0, offset 48 % 16 == 0
])
def test_scatter_subslice_2d(M, N, slice_m_offset, slice_n_offset, slice_m, slice_n):
```
- **EN:** Defines the test function `test_scatter_subslice_2d`. Decorators: `pytest.mark.parametrize('M,N,slice_m_offset,slice_n_offset,slice_m,slice_n', [(64, 64, 48, 16, 16, 16), (64, 64, 32, 48, 32, 16)])`. Parameters: `M`, `N`, `slice_m_offset`, `slice_n_offset`, `slice_m`, `slice_n`. Key calls include `pytest.mark.parametrize`, `torch.device`, `torch.zeros`, `torch.full`, `subslice_expected.scatter_`, `ttgl.BlockedLayout`, and 4 more. This scope touches pytest parametrization, PyTorch tensor setup and checks, tensor/descriptor metadata, layout transformation reasoning.
- **CN:** 定义测试函数 `test_scatter_subslice_2d`。 装饰器：`pytest.mark.parametrize('M,N,slice_m_offset,slice_n_offset,slice_m,slice_n', [(64, 64, 48, 16, 16, 16), (64, 64, 32, 48, 32, 16)])`。 参数：`M`、`N`、`slice_m_offset`、`slice_n_offset`、`slice_m`、`slice_n`。 关键调用包括 `pytest.mark.parametrize`、`torch.device`、`torch.zeros`、`torch.full`、`subslice_expected.scatter_`、`ttgl.BlockedLayout` 等另外 4 项。 该作用域涉及pytest 参数化、PyTorch 张量准备与校验、张量/描述符元数据、布局变换推理。

#### Lines 3568-3617

```python
    """Test scattering to a 2D subsliced shared memory descriptor."""
    device = torch.device("cuda")

    # Create indices (reverse pattern for scatter)
    indices = torch.arange(slice_n, dtype=torch.int32, device=device)[None, :].expand(slice_m, slice_n)
    indices = (slice_m - 1 - indices - torch.arange(slice_m, dtype=torch.int32, device=device)[:, None]) % slice_m

    # Create values to scatter
    values = torch.arange(slice_m * slice_n, dtype=torch.float32, device=device).reshape(slice_m, slice_n) + 100.0

    output = torch.zeros((M, N), dtype=torch.float32, device=device)

    # Expected: -1 everywhere, then scatter into the subslice region
    expected = torch.full((M, N), -1.0, dtype=torch.float32, device=device)
    subslice_expected = torch.zeros((slice_m, slice_n), dtype=torch.float32, device=device)
    subslice_expected.scatter_(0, indices.long(), values)
    expected[slice_m_offset:slice_m_offset + slice_m, slice_n_offset:slice_n_offset + slice_n] = subslice_expected

    # Layouts
    layout_full = ttgl.BlockedLayout(
        size_per_thread=[1, 1],
        threads_per_warp=[THREADS_PER_WARP // 4, 4],
        warps_per_cta=[1, 1],
        order=[1, 0],
    )
    layout_slice = ttgl.BlockedLayout(
        size_per_thread=[1, 1],
        threads_per_warp=[THREADS_PER_WARP // 4, 4],
        warps_per_cta=[1, 1],
        order=[1, 0],
    )
    shared_layout = ttgl.SwizzledSharedLayout(vec=1, per_phase=1, max_phase=1, order=[1, 0])

    scatter_subslice_2d_kernel[(1, )](
        indices,
        values,
        output,
        M=M,
        N=N,
        SLICE_M_OFFSET=slice_m_offset,
        SLICE_N_OFFSET=slice_n_offset,
        SLICE_M=slice_m,
        SLICE_N=slice_n,
        layout_full=layout_full,
        layout_slice=layout_slice,
        shared_layout=shared_layout,
        num_warps=1,
    )

    torch.testing.assert_close(output, expected)
```
- **EN:** Prepares or updates state through `device`, `indices`, `values`, `output`, `expected`, `subslice_expected`, `slice_m_offset`, `slice_n_offset`, and 5 more. Invokes `torch.device`, `torch.arange`, `torch.zeros`, `torch.full`, `subslice_expected.scatter_`, `indices.long`, and 3 more to execute the test logic. Relevant themes: PyTorch tensor setup and checks, tensor/descriptor metadata, layout transformation reasoning.
- **CN:** 通过 `device`、`indices`、`values`、`output`、`expected`、`subslice_expected`、`slice_m_offset`、`slice_n_offset` 等另外 5 项 准备或更新状态。 调用 `torch.device`、`torch.arange`、`torch.zeros`、`torch.full`、`subslice_expected.scatter_`、`indices.long` 等另外 3 项 执行测试逻辑。 相关主题：PyTorch 张量准备与校验、张量/描述符元数据、布局变换推理。

### Lines 3618-3634

```python


# =============================================================================
# Padded Layout Tests
# =============================================================================


@gluon.jit
def gather_padded_kernel(
    matrix_ptr,
    indices_ptr,
    output_ptr,
    M: ttgl.constexpr,
    N: ttgl.constexpr,
    layout_2d: ttgl.constexpr,
    padded_layout: ttgl.constexpr,
):
```
- **EN:** Defines the helper function `gather_padded_kernel`. Decorators: `gluon.jit`. Parameters: `matrix_ptr`, `indices_ptr`, `output_ptr`, `M`, `N`, `layout_2d`, `padded_layout`. Key calls include `ttgl.load`, `ttgl.allocate_shared_memory`, `smem.store`, `smem.gather`, `ttgl.store`, `ttgl.arange`, and 1 more. This scope touches layout transformation reasoning.
- **CN:** 定义辅助函数 `gather_padded_kernel`。 装饰器：`gluon.jit`。 参数：`matrix_ptr`、`indices_ptr`、`output_ptr`、`M`、`N`、`layout_2d`、`padded_layout`。 关键调用包括 `ttgl.load`、`ttgl.allocate_shared_memory`、`smem.store`、`smem.gather`、`ttgl.store`、`ttgl.arange` 等另外 1 项。 该作用域涉及布局变换推理。

#### Lines 3635-3651

```python
    """Gather from shared memory with a padded layout."""
    # Load matrix into padded shared memory
    offs_m = ttgl.arange(0, M, layout=ttgl.SliceLayout(1, layout_2d))[:, None]
    offs_n = ttgl.arange(0, N, layout=ttgl.SliceLayout(0, layout_2d))[None, :]
    in_offs = offs_m * N + offs_n
    in_data = ttgl.load(matrix_ptr + in_offs)

    smem = ttgl.allocate_shared_memory(ttgl.float32, [M, N], layout=padded_layout)
    smem.store(in_data)

    # Load indices
    indices = ttgl.load(indices_ptr + in_offs)

    # Gather along axis 0
    gathered = smem.gather(indices, axis=0)

    ttgl.store(output_ptr + in_offs, gathered)
```
- **EN:** Prepares or updates state through `offs_m`, `offs_n`, `in_offs`, `in_data`, `smem`, `indices`, `gathered`. Invokes `ttgl.arange`, `ttgl.SliceLayout`, `ttgl.load`, `ttgl.allocate_shared_memory`, `smem.store`, `smem.gather`, and 1 more to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `offs_m`、`offs_n`、`in_offs`、`in_data`、`smem`、`indices`、`gathered` 准备或更新状态。 调用 `ttgl.arange`、`ttgl.SliceLayout`、`ttgl.load`、`ttgl.allocate_shared_memory`、`smem.store`、`smem.gather` 等另外 1 项 执行测试逻辑。 相关主题：布局变换推理。

### Lines 3652-3657

```python


@pytest.mark.parametrize("M,N", [(64, 64)])
@pytest.mark.parametrize("interval_pairs", [[[32, 4]], [[16, 4]], [[16, 4], [64, 8]]])
@pytest.mark.parametrize("order", [[0, 1], [1, 0]])
def test_gather_padded(M, N, interval_pairs, order):
```
- **EN:** Defines the test function `test_gather_padded`. Decorators: `pytest.mark.parametrize('M,N', [(64, 64)])`, `pytest.mark.parametrize('interval_pairs', [[[32, 4]], [[16, 4]], [[16, 4], [64, 8]]])`, `pytest.mark.parametrize('order', [[0, 1], [1, 0]])`. Parameters: `M`, `N`, `interval_pairs`, `order`. Key calls include `pytest.mark.parametrize`, `torch.device`, `torch.zeros`, `torch.gather`, `ttgl.BlockedLayout`, `ttgl.PaddedSharedLayout.with_identity_for`, and 3 more. This scope touches pytest parametrization, PyTorch tensor setup and checks, layout transformation reasoning.
- **CN:** 定义测试函数 `test_gather_padded`。 装饰器：`pytest.mark.parametrize('M,N', [(64, 64)])`、`pytest.mark.parametrize('interval_pairs', [[[32, 4]], [[16, 4]], [[16, 4], [64, 8]]])`、`pytest.mark.parametrize('order', [[0, 1], [1, 0]])`。 参数：`M`、`N`、`interval_pairs`、`order`。 关键调用包括 `pytest.mark.parametrize`、`torch.device`、`torch.zeros`、`torch.gather`、`ttgl.BlockedLayout`、`ttgl.PaddedSharedLayout.with_identity_for` 等另外 3 项。 该作用域涉及pytest 参数化、PyTorch 张量准备与校验、布局变换推理。

#### Lines 3658-3693

```python
    """Test gathering from shared memory with a padded layout."""
    device = torch.device("cuda")

    # Create input matrix
    matrix = torch.arange(M * N, dtype=torch.float32, device=device).reshape(M, N)

    # Create indices for gather along axis 0
    indices = torch.arange(N, dtype=torch.int32, device=device)[None, :].expand(M, N)
    indices = (indices + torch.arange(M, dtype=torch.int32, device=device)[:, None]) % M

    output = torch.zeros((M, N), dtype=torch.float32, device=device)

    # Expected: gather along axis 0
    expected = torch.gather(matrix, 0, indices.long())

    # Layouts
    layout_2d = ttgl.BlockedLayout(
        size_per_thread=[1, 1],
        threads_per_warp=[THREADS_PER_WARP // 4, 4],
        warps_per_cta=[1, 1],
        order=[1, 0],
    )
    padded_layout = ttgl.PaddedSharedLayout.with_identity_for(interval_pairs, [M, N], order)

    gather_padded_kernel[(1, )](
        matrix,
        indices,
        output,
        M=M,
        N=N,
        layout_2d=layout_2d,
        padded_layout=padded_layout,
        num_warps=1,
    )

    torch.testing.assert_close(output, expected)
```
- **EN:** Prepares or updates state through `device`, `matrix`, `indices`, `output`, `expected`, `layout_2d`, `padded_layout`. Invokes `torch.device`, `torch.arange`, `torch.zeros`, `torch.gather`, `indices.long`, `ttgl.BlockedLayout`, and 2 more to execute the test logic. Relevant themes: PyTorch tensor setup and checks, layout transformation reasoning.
- **CN:** 通过 `device`、`matrix`、`indices`、`output`、`expected`、`layout_2d`、`padded_layout` 准备或更新状态。 调用 `torch.device`、`torch.arange`、`torch.zeros`、`torch.gather`、`indices.long`、`ttgl.BlockedLayout` 等另外 2 项 执行测试逻辑。 相关主题：PyTorch 张量准备与校验、布局变换推理。

### Lines 3694-3705

```python


@gluon.jit
def scatter_padded_kernel(
    indices_ptr,
    values_ptr,
    output_ptr,
    M: ttgl.constexpr,
    N: ttgl.constexpr,
    layout_2d: ttgl.constexpr,
    padded_layout: ttgl.constexpr,
):
```
- **EN:** Defines the helper function `scatter_padded_kernel`. Decorators: `gluon.jit`. Parameters: `indices_ptr`, `values_ptr`, `output_ptr`, `M`, `N`, `layout_2d`, `padded_layout`. Key calls include `ttgl.zeros`, `ttgl.allocate_shared_memory`, `smem.store`, `ttgl.load`, `smem.scatter`, `smem.load`, and 3 more. This scope touches layout transformation reasoning.
- **CN:** 定义辅助函数 `scatter_padded_kernel`。 装饰器：`gluon.jit`。 参数：`indices_ptr`、`values_ptr`、`output_ptr`、`M`、`N`、`layout_2d`、`padded_layout`。 关键调用包括 `ttgl.zeros`、`ttgl.allocate_shared_memory`、`smem.store`、`ttgl.load`、`smem.scatter`、`smem.load` 等另外 3 项。 该作用域涉及布局变换推理。

#### Lines 3706-3725

```python
    """Scatter to shared memory with a padded layout."""
    # Initialize padded shared memory with zeros
    offs_m = ttgl.arange(0, M, layout=ttgl.SliceLayout(1, layout_2d))[:, None]
    offs_n = ttgl.arange(0, N, layout=ttgl.SliceLayout(0, layout_2d))[None, :]
    full_offs = offs_m * N + offs_n
    zeros = ttgl.zeros([M, N], ttgl.float32, layout=layout_2d)

    smem = ttgl.allocate_shared_memory(ttgl.float32, [M, N], layout=padded_layout)
    smem.store(zeros)

    # Load indices and values
    indices = ttgl.load(indices_ptr + full_offs)
    values = ttgl.load(values_ptr + full_offs)

    # Scatter along axis 0
    smem.scatter(values, indices, axis=0)

    # Load back
    result = smem.load(layout=layout_2d)
    ttgl.store(output_ptr + full_offs, result)
```
- **EN:** Prepares or updates state through `offs_m`, `offs_n`, `full_offs`, `zeros`, `smem`, `indices`, `values`, `result`. Invokes `ttgl.arange`, `ttgl.SliceLayout`, `ttgl.zeros`, `ttgl.allocate_shared_memory`, `smem.store`, `ttgl.load`, and 3 more to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `offs_m`、`offs_n`、`full_offs`、`zeros`、`smem`、`indices`、`values`、`result` 准备或更新状态。 调用 `ttgl.arange`、`ttgl.SliceLayout`、`ttgl.zeros`、`ttgl.allocate_shared_memory`、`smem.store`、`ttgl.load` 等另外 3 项 执行测试逻辑。 相关主题：布局变换推理。

### Lines 3726-3731

```python


@pytest.mark.parametrize("M,N", [(64, 64)])
@pytest.mark.parametrize("interval_pairs", [[[32, 4]], [[16, 4]]])
@pytest.mark.parametrize("order", [[0, 1], [1, 0]])
def test_scatter_padded(M, N, interval_pairs, order):
```
- **EN:** Defines the test function `test_scatter_padded`. Decorators: `pytest.mark.parametrize('M,N', [(64, 64)])`, `pytest.mark.parametrize('interval_pairs', [[[32, 4]], [[16, 4]]])`, `pytest.mark.parametrize('order', [[0, 1], [1, 0]])`. Parameters: `M`, `N`, `interval_pairs`, `order`. Key calls include `pytest.mark.parametrize`, `torch.device`, `torch.zeros`, `expected.scatter_`, `ttgl.BlockedLayout`, `ttgl.PaddedSharedLayout.with_identity_for`, and 3 more. This scope touches pytest parametrization, PyTorch tensor setup and checks, layout transformation reasoning.
- **CN:** 定义测试函数 `test_scatter_padded`。 装饰器：`pytest.mark.parametrize('M,N', [(64, 64)])`、`pytest.mark.parametrize('interval_pairs', [[[32, 4]], [[16, 4]]])`、`pytest.mark.parametrize('order', [[0, 1], [1, 0]])`。 参数：`M`、`N`、`interval_pairs`、`order`。 关键调用包括 `pytest.mark.parametrize`、`torch.device`、`torch.zeros`、`expected.scatter_`、`ttgl.BlockedLayout`、`ttgl.PaddedSharedLayout.with_identity_for` 等另外 3 项。 该作用域涉及pytest 参数化、PyTorch 张量准备与校验、布局变换推理。

#### Lines 3732-3768

```python
    """Test scattering to shared memory with a padded layout."""
    device = torch.device("cuda")

    # Create indices (reverse pattern)
    indices = torch.arange(N, dtype=torch.int32, device=device)[None, :].expand(M, N)
    indices = (M - 1 - indices - torch.arange(M, dtype=torch.int32, device=device)[:, None]) % M

    # Create values
    values = torch.arange(M * N, dtype=torch.float32, device=device).reshape(M, N) + 100.0

    output = torch.zeros((M, N), dtype=torch.float32, device=device)

    # Expected: scatter along axis 0
    expected = torch.zeros((M, N), dtype=torch.float32, device=device)
    expected.scatter_(0, indices.long(), values)

    # Layouts
    layout_2d = ttgl.BlockedLayout(
        size_per_thread=[1, 1],
        threads_per_warp=[THREADS_PER_WARP // 4, 4],
        warps_per_cta=[1, 1],
        order=[1, 0],
    )
    padded_layout = ttgl.PaddedSharedLayout.with_identity_for(interval_pairs, [M, N], order)

    scatter_padded_kernel[(1, )](
        indices,
        values,
        output,
        M=M,
        N=N,
        layout_2d=layout_2d,
        padded_layout=padded_layout,
        num_warps=1,
    )

    torch.testing.assert_close(output, expected)
```
- **EN:** Prepares or updates state through `device`, `indices`, `values`, `output`, `expected`, `layout_2d`, `padded_layout`. Invokes `torch.device`, `torch.arange`, `torch.zeros`, `expected.scatter_`, `indices.long`, `ttgl.BlockedLayout`, and 2 more to execute the test logic. Relevant themes: PyTorch tensor setup and checks, layout transformation reasoning.
- **CN:** 通过 `device`、`indices`、`values`、`output`、`expected`、`layout_2d`、`padded_layout` 准备或更新状态。 调用 `torch.device`、`torch.arange`、`torch.zeros`、`expected.scatter_`、`indices.long`、`ttgl.BlockedLayout` 等另外 2 项 执行测试逻辑。 相关主题：PyTorch 张量准备与校验、布局变换推理。

### Lines 3769-3790

```python


# =============================================================================
# Padded Layout with Subslice Tests
# =============================================================================


@gluon.jit
def gather_padded_subslice_kernel(
    matrix_ptr,
    indices_ptr,
    output_ptr,
    M: ttgl.constexpr,
    N: ttgl.constexpr,
    SLICE_M_OFFSET: ttgl.constexpr,
    SLICE_N_OFFSET: ttgl.constexpr,
    SLICE_M: ttgl.constexpr,
    SLICE_N: ttgl.constexpr,
    layout_full: ttgl.constexpr,
    layout_slice: ttgl.constexpr,
    padded_layout: ttgl.constexpr,
):
```
- **EN:** Defines the helper function `gather_padded_subslice_kernel`. Decorators: `gluon.jit`. Parameters: `matrix_ptr`, `indices_ptr`, `output_ptr`, `M`, `N`, `SLICE_M_OFFSET`, `SLICE_N_OFFSET`, `SLICE_M`, and 4 more. Key calls include `ttgl.load`, `ttgl.allocate_shared_memory`, `smem.store`, `smem_slice.gather`, `ttgl.store`, `ttgl.arange`, and 2 more. This scope touches tensor/descriptor metadata, layout transformation reasoning.
- **CN:** 定义辅助函数 `gather_padded_subslice_kernel`。 装饰器：`gluon.jit`。 参数：`matrix_ptr`、`indices_ptr`、`output_ptr`、`M`、`N`、`SLICE_M_OFFSET`、`SLICE_N_OFFSET`、`SLICE_M` 等另外 4 项。 关键调用包括 `ttgl.load`、`ttgl.allocate_shared_memory`、`smem.store`、`smem_slice.gather`、`ttgl.store`、`ttgl.arange` 等另外 2 项。 该作用域涉及张量/描述符元数据、布局变换推理。

#### Lines 3791-3813

```python
    """Gather from a subsliced padded shared memory descriptor."""
    # Load full matrix into padded shared memory
    offs_m = ttgl.arange(0, M, layout=ttgl.SliceLayout(1, layout_full))[:, None]
    offs_n = ttgl.arange(0, N, layout=ttgl.SliceLayout(0, layout_full))[None, :]
    in_offs = offs_m * N + offs_n
    in_data = ttgl.load(matrix_ptr + in_offs)

    smem = ttgl.allocate_shared_memory(ttgl.float32, [M, N], layout=padded_layout)
    smem.store(in_data)

    # Create 2D subslice
    smem_slice = smem.slice(SLICE_M_OFFSET, SLICE_M, dim=0).slice(SLICE_N_OFFSET, SLICE_N, dim=1)

    # Load indices for gathering within the slice
    slice_offs_m = ttgl.arange(0, SLICE_M, layout=ttgl.SliceLayout(1, layout_slice))[:, None]
    slice_offs_n = ttgl.arange(0, SLICE_N, layout=ttgl.SliceLayout(0, layout_slice))[None, :]
    idx_offs = slice_offs_m * SLICE_N + slice_offs_n
    indices = ttgl.load(indices_ptr + idx_offs)

    # Gather along axis 0
    gathered = smem_slice.gather(indices, axis=0)

    ttgl.store(output_ptr + idx_offs, gathered)
```
- **EN:** Prepares or updates state through `offs_m`, `offs_n`, `in_offs`, `in_data`, `smem`, `smem_slice`, `slice_offs_m`, `slice_offs_n`, and 3 more. Invokes `ttgl.arange`, `ttgl.SliceLayout`, `ttgl.load`, `ttgl.allocate_shared_memory`, `smem.store`, `smem.slice`, and 2 more to execute the test logic. Relevant themes: tensor/descriptor metadata, layout transformation reasoning.
- **CN:** 通过 `offs_m`、`offs_n`、`in_offs`、`in_data`、`smem`、`smem_slice`、`slice_offs_m`、`slice_offs_n` 等另外 3 项 准备或更新状态。 调用 `ttgl.arange`、`ttgl.SliceLayout`、`ttgl.load`、`ttgl.allocate_shared_memory`、`smem.store`、`smem.slice` 等另外 2 项 执行测试逻辑。 相关主题：张量/描述符元数据、布局变换推理。

### Lines 3814-3823

```python


@pytest.mark.parametrize("interval_pairs", [[[32, 4]], [[16, 4]]])
@pytest.mark.parametrize("order", [[0, 1], [1, 0]])
@pytest.mark.parametrize("slice_m_offset,slice_n_offset,slice_m,slice_n", [
    (48, 16, 16, 16),
    (32, 48, 32, 16),
    (48, 32, 16, 32),
])
def test_gather_padded_subslice(interval_pairs, order, slice_m_offset, slice_n_offset, slice_m, slice_n):
```
- **EN:** Defines the test function `test_gather_padded_subslice`. Decorators: `pytest.mark.parametrize('interval_pairs', [[[32, 4]], [[16, 4]]])`, `pytest.mark.parametrize('order', [[0, 1], [1, 0]])`, `pytest.mark.parametrize('slice_m_offset,slice_n_offset,slice_m,slice_n', [(48, 16, 16, 16), (32, 48, 32, 16), (48, 32, 16, 32)])`. Parameters: `interval_pairs`, `order`, `slice_m_offset`, `slice_n_offset`, `slice_m`, `slice_n`. Key calls include `pytest.mark.parametrize`, `torch.device`, `torch.zeros`, `torch.gather`, `ttgl.BlockedLayout`, `ttgl.PaddedSharedLayout.with_identity_for`, and 3 more. This scope touches pytest parametrization, PyTorch tensor setup and checks, tensor/descriptor metadata, layout transformation reasoning.
- **CN:** 定义测试函数 `test_gather_padded_subslice`。 装饰器：`pytest.mark.parametrize('interval_pairs', [[[32, 4]], [[16, 4]]])`、`pytest.mark.parametrize('order', [[0, 1], [1, 0]])`、`pytest.mark.parametrize('slice_m_offset,slice_n_offset,slice_m,slice_n', [(48, 16, 16, 16), (32, 48, 32, 16), (48, 32, 16, 32)])`。 参数：`interval_pairs`、`order`、`slice_m_offset`、`slice_n_offset`、`slice_m`、`slice_n`。 关键调用包括 `pytest.mark.parametrize`、`torch.device`、`torch.zeros`、`torch.gather`、`ttgl.BlockedLayout`、`ttgl.PaddedSharedLayout.with_identity_for` 等另外 3 项。 该作用域涉及pytest 参数化、PyTorch 张量准备与校验、张量/描述符元数据、布局变换推理。

#### Lines 3824-3872

```python
    """Test gathering from a subsliced padded shared memory descriptor."""
    M, N = 64, 64
    device = torch.device("cuda")

    # Create input matrix
    matrix = torch.arange(M * N, dtype=torch.float32, device=device).reshape(M, N)

    # Create indices for gather within the slice
    indices = torch.arange(slice_n, dtype=torch.int32, device=device)[None, :].expand(slice_m, slice_n)
    indices = (indices + torch.arange(slice_m, dtype=torch.int32, device=device)[:, None]) % slice_m

    output = torch.zeros((slice_m, slice_n), dtype=torch.float32, device=device)

    # Expected: gather from the subslice
    subslice = matrix[slice_m_offset:slice_m_offset + slice_m, slice_n_offset:slice_n_offset + slice_n]
    expected = torch.gather(subslice, 0, indices.long())

    # Layouts
    layout_full = ttgl.BlockedLayout(
        size_per_thread=[1, 1],
        threads_per_warp=[THREADS_PER_WARP // 4, 4],
        warps_per_cta=[1, 1],
        order=[1, 0],
    )
    layout_slice = ttgl.BlockedLayout(
        size_per_thread=[1, 1],
        threads_per_warp=[THREADS_PER_WARP // 4, 4],
        warps_per_cta=[1, 1],
        order=[1, 0],
    )
    padded_layout = ttgl.PaddedSharedLayout.with_identity_for(interval_pairs, [M, N], order)

    gather_padded_subslice_kernel[(1, )](
        matrix,
        indices,
        output,
        M=M,
        N=N,
        SLICE_M_OFFSET=slice_m_offset,
        SLICE_N_OFFSET=slice_n_offset,
        SLICE_M=slice_m,
        SLICE_N=slice_n,
        layout_full=layout_full,
        layout_slice=layout_slice,
        padded_layout=padded_layout,
        num_warps=1,
    )

    torch.testing.assert_close(output, expected)
```
- **EN:** Prepares or updates state through `M`, `N`, `device`, `matrix`, `indices`, `output`, `subslice`, `expected`, and 3 more. Invokes `torch.device`, `torch.arange`, `torch.zeros`, `torch.gather`, `indices.long`, `ttgl.BlockedLayout`, and 2 more to execute the test logic. Relevant themes: PyTorch tensor setup and checks, tensor/descriptor metadata, layout transformation reasoning.
- **CN:** 通过 `M`、`N`、`device`、`matrix`、`indices`、`output`、`subslice`、`expected` 等另外 3 项 准备或更新状态。 调用 `torch.device`、`torch.arange`、`torch.zeros`、`torch.gather`、`indices.long`、`ttgl.BlockedLayout` 等另外 2 项 执行测试逻辑。 相关主题：PyTorch 张量准备与校验、张量/描述符元数据、布局变换推理。

### Lines 3873-3889

```python


@gluon.jit
def scatter_padded_subslice_kernel(
    indices_ptr,
    values_ptr,
    output_ptr,
    M: ttgl.constexpr,
    N: ttgl.constexpr,
    SLICE_M_OFFSET: ttgl.constexpr,
    SLICE_N_OFFSET: ttgl.constexpr,
    SLICE_M: ttgl.constexpr,
    SLICE_N: ttgl.constexpr,
    layout_full: ttgl.constexpr,
    layout_slice: ttgl.constexpr,
    padded_layout: ttgl.constexpr,
):
```
- **EN:** Defines the helper function `scatter_padded_subslice_kernel`. Decorators: `gluon.jit`. Parameters: `indices_ptr`, `values_ptr`, `output_ptr`, `M`, `N`, `SLICE_M_OFFSET`, `SLICE_N_OFFSET`, `SLICE_M`, and 4 more. Key calls include `ttgl.full`, `ttgl.allocate_shared_memory`, `smem.store`, `ttgl.load`, `smem_slice.scatter`, `smem.load`, and 4 more. This scope touches tensor/descriptor metadata, layout transformation reasoning.
- **CN:** 定义辅助函数 `scatter_padded_subslice_kernel`。 装饰器：`gluon.jit`。 参数：`indices_ptr`、`values_ptr`、`output_ptr`、`M`、`N`、`SLICE_M_OFFSET`、`SLICE_N_OFFSET`、`SLICE_M` 等另外 4 项。 关键调用包括 `ttgl.full`、`ttgl.allocate_shared_memory`、`smem.store`、`ttgl.load`、`smem_slice.scatter`、`smem.load` 等另外 4 项。 该作用域涉及张量/描述符元数据、布局变换推理。

#### Lines 3890-3915

```python
    """Scatter to a subsliced padded shared memory descriptor."""
    # Initialize padded shared memory with -1
    offs_m = ttgl.arange(0, M, layout=ttgl.SliceLayout(1, layout_full))[:, None]
    offs_n = ttgl.arange(0, N, layout=ttgl.SliceLayout(0, layout_full))[None, :]
    full_offs = offs_m * N + offs_n
    init_data = ttgl.full([M, N], -1.0, dtype=ttgl.float32, layout=layout_full)

    smem = ttgl.allocate_shared_memory(ttgl.float32, [M, N], layout=padded_layout)
    smem.store(init_data)

    # Create 2D subslice
    smem_slice = smem.slice(SLICE_M_OFFSET, SLICE_M, dim=0).slice(SLICE_N_OFFSET, SLICE_N, dim=1)

    # Load indices and values for scattering within the slice
    slice_offs_m = ttgl.arange(0, SLICE_M, layout=ttgl.SliceLayout(1, layout_slice))[:, None]
    slice_offs_n = ttgl.arange(0, SLICE_N, layout=ttgl.SliceLayout(0, layout_slice))[None, :]
    idx_offs = slice_offs_m * SLICE_N + slice_offs_n
    indices = ttgl.load(indices_ptr + idx_offs)
    values = ttgl.load(values_ptr + idx_offs)

    # Scatter along axis 0
    smem_slice.scatter(values, indices, axis=0)

    # Load back full matrix
    result = smem.load(layout=layout_full)
    ttgl.store(output_ptr + full_offs, result)
```
- **EN:** Prepares or updates state through `offs_m`, `offs_n`, `full_offs`, `init_data`, `smem`, `smem_slice`, `slice_offs_m`, `slice_offs_n`, and 4 more. Invokes `ttgl.arange`, `ttgl.SliceLayout`, `ttgl.full`, `ttgl.allocate_shared_memory`, `smem.store`, `smem.slice`, and 4 more to execute the test logic. Relevant themes: tensor/descriptor metadata, layout transformation reasoning.
- **CN:** 通过 `offs_m`、`offs_n`、`full_offs`、`init_data`、`smem`、`smem_slice`、`slice_offs_m`、`slice_offs_n` 等另外 4 项 准备或更新状态。 调用 `ttgl.arange`、`ttgl.SliceLayout`、`ttgl.full`、`ttgl.allocate_shared_memory`、`smem.store`、`smem.slice` 等另外 4 项 执行测试逻辑。 相关主题：张量/描述符元数据、布局变换推理。

### Lines 3916-3924

```python


@pytest.mark.parametrize("interval_pairs", [[[32, 4]], [[16, 4]]])
@pytest.mark.parametrize("order", [[0, 1], [1, 0]])
@pytest.mark.parametrize("slice_m_offset,slice_n_offset,slice_m,slice_n", [
    (48, 16, 16, 16),
    (32, 48, 32, 16),
])
def test_scatter_padded_subslice(interval_pairs, order, slice_m_offset, slice_n_offset, slice_m, slice_n):
```
- **EN:** Defines the test function `test_scatter_padded_subslice`. Decorators: `pytest.mark.parametrize('interval_pairs', [[[32, 4]], [[16, 4]]])`, `pytest.mark.parametrize('order', [[0, 1], [1, 0]])`, `pytest.mark.parametrize('slice_m_offset,slice_n_offset,slice_m,slice_n', [(48, 16, 16, 16), (32, 48, 32, 16)])`. Parameters: `interval_pairs`, `order`, `slice_m_offset`, `slice_n_offset`, `slice_m`, `slice_n`. Key calls include `pytest.mark.parametrize`, `torch.device`, `torch.zeros`, `torch.full`, `subslice_expected.scatter_`, `ttgl.BlockedLayout`, and 4 more. This scope touches pytest parametrization, PyTorch tensor setup and checks, tensor/descriptor metadata, layout transformation reasoning.
- **CN:** 定义测试函数 `test_scatter_padded_subslice`。 装饰器：`pytest.mark.parametrize('interval_pairs', [[[32, 4]], [[16, 4]]])`、`pytest.mark.parametrize('order', [[0, 1], [1, 0]])`、`pytest.mark.parametrize('slice_m_offset,slice_n_offset,slice_m,slice_n', [(48, 16, 16, 16), (32, 48, 32, 16)])`。 参数：`interval_pairs`、`order`、`slice_m_offset`、`slice_n_offset`、`slice_m`、`slice_n`。 关键调用包括 `pytest.mark.parametrize`、`torch.device`、`torch.zeros`、`torch.full`、`subslice_expected.scatter_`、`ttgl.BlockedLayout` 等另外 4 项。 该作用域涉及pytest 参数化、PyTorch 张量准备与校验、张量/描述符元数据、布局变换推理。

#### Lines 3925-3975

```python
    """Test scattering to a subsliced padded shared memory descriptor."""
    M, N = 64, 64
    device = torch.device("cuda")

    # Create indices (reverse pattern)
    indices = torch.arange(slice_n, dtype=torch.int32, device=device)[None, :].expand(slice_m, slice_n)
    indices = (slice_m - 1 - indices - torch.arange(slice_m, dtype=torch.int32, device=device)[:, None]) % slice_m

    # Create values
    values = torch.arange(slice_m * slice_n, dtype=torch.float32, device=device).reshape(slice_m, slice_n) + 100.0

    output = torch.zeros((M, N), dtype=torch.float32, device=device)

    # Expected: -1 everywhere, then scatter into the subslice region
    expected = torch.full((M, N), -1.0, dtype=torch.float32, device=device)
    subslice_expected = torch.zeros((slice_m, slice_n), dtype=torch.float32, device=device)
    subslice_expected.scatter_(0, indices.long(), values)
    expected[slice_m_offset:slice_m_offset + slice_m, slice_n_offset:slice_n_offset + slice_n] = subslice_expected

    # Layouts
    layout_full = ttgl.BlockedLayout(
        size_per_thread=[1, 1],
        threads_per_warp=[THREADS_PER_WARP // 4, 4],
        warps_per_cta=[1, 1],
        order=[1, 0],
    )
    layout_slice = ttgl.BlockedLayout(
        size_per_thread=[1, 1],
        threads_per_warp=[THREADS_PER_WARP // 4, 4],
        warps_per_cta=[1, 1],
        order=[1, 0],
    )
    padded_layout = ttgl.PaddedSharedLayout.with_identity_for(interval_pairs, [M, N], order)

    scatter_padded_subslice_kernel[(1, )](
        indices,
        values,
        output,
        M=M,
        N=N,
        SLICE_M_OFFSET=slice_m_offset,
        SLICE_N_OFFSET=slice_n_offset,
        SLICE_M=slice_m,
        SLICE_N=slice_n,
        layout_full=layout_full,
        layout_slice=layout_slice,
        padded_layout=padded_layout,
        num_warps=1,
    )

    torch.testing.assert_close(output, expected)
```
- **EN:** Prepares or updates state through `M`, `N`, `device`, `indices`, `values`, `output`, `expected`, `subslice_expected`, and 7 more. Invokes `torch.device`, `torch.arange`, `torch.zeros`, `torch.full`, `subslice_expected.scatter_`, `indices.long`, and 3 more to execute the test logic. Relevant themes: PyTorch tensor setup and checks, tensor/descriptor metadata, layout transformation reasoning.
- **CN:** 通过 `M`、`N`、`device`、`indices`、`values`、`output`、`expected`、`subslice_expected` 等另外 7 项 准备或更新状态。 调用 `torch.device`、`torch.arange`、`torch.zeros`、`torch.full`、`subslice_expected.scatter_`、`indices.long` 等另外 3 项 执行测试逻辑。 相关主题：PyTorch 张量准备与校验、张量/描述符元数据、布局变换推理。

### Lines 3976-3992

```python


# --- TMEM Load with Reduction Tests ---


@gluon.jit
def tmem_reduction_kernel(
    in_ptr,
    out_ptr,
    red_ptr,
    M: ttgl.constexpr,
    N: ttgl.constexpr,
    RED_OP: ttgl.constexpr,
    USE_ABS: ttgl.constexpr,
    PROPAGATE_NAN: ttgl.constexpr,
    num_warps: ttgl.constexpr,
):
```
- **EN:** Defines the helper function `tmem_reduction_kernel`. Decorators: `gluon.jit`. Parameters: `in_ptr`, `out_ptr`, `red_ptr`, `M`, `N`, `RED_OP`, `USE_ABS`, `PROPAGATE_NAN`, and 1 more. Key calls include `ttgl.BlockedLayout`, `ttgl.arange`, `ttgl.load`, `TensorMemoryLayout`, `allocate_tensor_memory`, `tmem.get_reg_layout`, and 6 more. This scope touches layout transformation reasoning.
- **CN:** 定义辅助函数 `tmem_reduction_kernel`。 装饰器：`gluon.jit`。 参数：`in_ptr`、`out_ptr`、`red_ptr`、`M`、`N`、`RED_OP`、`USE_ABS`、`PROPAGATE_NAN` 等另外 1 项。 关键调用包括 `ttgl.BlockedLayout`、`ttgl.arange`、`ttgl.load`、`TensorMemoryLayout`、`allocate_tensor_memory`、`tmem.get_reg_layout` 等另外 6 项。 该作用域涉及布局变换推理。

#### Lines 3993-4021

```python
    """Kernel to test TMEM load with hardware reduction."""
    global_memory_layout: ttgl.constexpr = ttgl.BlockedLayout([1, 1], [1, 32], [1, num_warps], [1, 0])
    global_memory_layout_1d: ttgl.constexpr = ttgl.BlockedLayout([1], [32], [num_warps], [0])

    # Offsets for 2D tensor
    offs_m = ttgl.arange(0, M, ttgl.SliceLayout(1, global_memory_layout))
    offs_n = ttgl.arange(0, N, ttgl.SliceLayout(0, global_memory_layout))
    offs_2d = offs_m[:, None] * N + offs_n[None, :]

    # Load input from global memory
    input_data = ttgl.load(in_ptr + offs_2d)

    # Setup TMEM layout - blockN must match N for single reduction value per row
    tmem_layout: ttgl.constexpr = TensorMemoryLayout(block=(128, N), col_stride=1,  # packed for f32
                                                     )

    # Allocate TMEM
    tmem = allocate_tensor_memory(
        element_ty=in_ptr.dtype.element_ty,
        shape=[M, N],
        layout=tmem_layout,
    )

    # Get register layout for TMEM access
    tmem_reg_layout: ttgl.constexpr = tmem.get_reg_layout()

    # Store input to TMEM
    input_data = ttgl.convert_layout(input_data, tmem_reg_layout)
    tmem.store(input_data)
```
- **EN:** Prepares or updates state through `global_memory_layout`, `global_memory_layout_1d`, `offs_m`, `offs_n`, `offs_2d`, `input_data`, `tmem_layout`, `tmem`, and 1 more. Invokes `ttgl.BlockedLayout`, `ttgl.arange`, `ttgl.SliceLayout`, `ttgl.load`, `TensorMemoryLayout`, `allocate_tensor_memory`, and 3 more to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `global_memory_layout`、`global_memory_layout_1d`、`offs_m`、`offs_n`、`offs_2d`、`input_data`、`tmem_layout`、`tmem` 等另外 1 项 准备或更新状态。 调用 `ttgl.BlockedLayout`、`ttgl.arange`、`ttgl.SliceLayout`、`ttgl.load`、`TensorMemoryLayout`、`allocate_tensor_memory` 等另外 3 项 执行测试逻辑。 相关主题：布局变换推理。

#### Lines 4022-4027

```python

    # Load from TMEM with reduction
    if RED_OP == "min":
        output, reduced = tmem.load_min(abs=USE_ABS, propagate_nan=PROPAGATE_NAN)
    elif RED_OP == "max":
        output, reduced = tmem.load_max(abs=USE_ABS, propagate_nan=PROPAGATE_NAN)
```
- **EN:** Invokes `tmem.load_min`, `tmem.load_max` to execute the test logic. Branches on runtime or test conditions.
- **CN:** 调用 `tmem.load_min`、`tmem.load_max` 执行测试逻辑。 根据运行时或测试条件进行分支。

#### Lines 4028-4036

```python

    # Store full output
    output = ttgl.convert_layout(output, global_memory_layout)
    ttgl.store(out_ptr + offs_2d, output)

    # Store reduced output (1D tensor of shape [M])
    offs_1d = ttgl.arange(0, M, global_memory_layout_1d)
    reduced = ttgl.convert_layout(reduced, global_memory_layout_1d)
    ttgl.store(red_ptr + offs_1d, reduced)
```
- **EN:** Prepares or updates state through `output`, `offs_1d`, `reduced`. Invokes `ttgl.convert_layout`, `ttgl.store`, `ttgl.arange` to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `output`、`offs_1d`、`reduced` 准备或更新状态。 调用 `ttgl.convert_layout`、`ttgl.store`、`ttgl.arange` 执行测试逻辑。 相关主题：布局变换推理。

### Lines 4037-4047

```python


@pytest.mark.skipif(not is_blackwell_ultra(), reason="Requires Blackwell Ultra")
@pytest.mark.parametrize("red_op", ["min", "max"])
@pytest.mark.parametrize("use_abs", [False, True])
@pytest.mark.parametrize("propagate_nan", [tl.PropagateNan.NONE, tl.PropagateNan.ALL])
@pytest.mark.parametrize(
    "M, N, num_warps",
    [(128, 32, 4), (128, 64, 4), (128, 128, 4), (128, 256, 4), (256, 128, 8)],
)
def test_tmem_reduction(red_op, use_abs, propagate_nan, M, N, num_warps):
```
- **EN:** Defines the test function `test_tmem_reduction`. Decorators: `pytest.mark.skipif(not is_blackwell_ultra(), reason='Requires Blackwell Ultra')`, `pytest.mark.parametrize('red_op', ['min', 'max'])`, `pytest.mark.parametrize('use_abs', [False, True])`, `pytest.mark.parametrize('propagate_nan', [tl.PropagateNan.NONE, tl.PropagateNan.ALL])`, `pytest.mark.parametrize('M, N, num_warps', [(128, 32, 4), (128, 64, 4), (128, 128, 4), (128, 256, 4), (256, 128, 8)])`. Parameters: `red_op`, `use_abs`, `propagate_nan`, `M`, `N`, `num_warps`. Key calls include `pytest.mark.skipif`, `pytest.mark.parametrize`, `torch.randn`, `torch.empty_like`, `torch.empty`, `torch.testing.assert_close`, and 3 more. This scope touches pytest parametrization, Triton language operations, PyTorch tensor setup and checks, random-data generation.
- **CN:** 定义测试函数 `test_tmem_reduction`。 装饰器：`pytest.mark.skipif(not is_blackwell_ultra(), reason='Requires Blackwell Ultra')`、`pytest.mark.parametrize('red_op', ['min', 'max'])`、`pytest.mark.parametrize('use_abs', [False, True])`、`pytest.mark.parametrize('propagate_nan', [tl.PropagateNan.NONE, tl.PropagateNan.ALL])`、`pytest.mark.parametrize('M, N, num_warps', [(128, 32, 4), (128, 64, 4), (128, 128, 4), (128, 256, 4), (256, 128, 8)])`。 参数：`red_op`、`use_abs`、`propagate_nan`、`M`、`N`、`num_warps`。 关键调用包括 `pytest.mark.skipif`、`pytest.mark.parametrize`、`torch.randn`、`torch.empty_like`、`torch.empty`、`torch.testing.assert_close` 等另外 3 项。 该作用域涉及pytest 参数化、Triton language 操作、PyTorch 张量准备与校验、随机数据生成。

#### Lines 4048-4060

```python
    """Test TMEM load with hardware reduction on MxN tile

    Note: With M=128, only 4 warps can be used (warpsPerCTA=[4,1]) since all
    warps must fit in the M dimension for reduction. 8 warps would require
    M=256 (8*32=256). The N=256 case tests partial reduction combining where
    4 hardware reductions are combined via llvm.minnum/maxnum.
    """

    # Create test input with some negative values
    input_tensor = torch.randn(M, N, dtype=torch.float32, device="cuda")

    # Inject NaN for testing if needed
    use_nan = False if propagate_nan == tl.PropagateNan.NONE else True
```
- **EN:** Prepares or updates state through `input_tensor`, `use_nan`. Invokes `torch.randn` to execute the test logic. Relevant themes: Triton language operations, PyTorch tensor setup and checks, random-data generation.
- **CN:** 通过 `input_tensor`、`use_nan` 准备或更新状态。 调用 `torch.randn` 执行测试逻辑。 相关主题：Triton language 操作、PyTorch 张量准备与校验、随机数据生成。

#### Lines 4061-4063

```python
    if use_nan:
        input_tensor[10, 5] = float("nan")
        input_tensor[50, 15] = float("nan")
```
- **EN:** Branches on runtime or test conditions.
- **CN:** 根据运行时或测试条件进行分支。

#### Lines 4064-4093

```python

    # Output tensors
    output = torch.empty_like(input_tensor)
    red_output = torch.empty(M, dtype=torch.float32, device="cuda")

    # Run kernel
    tmem_reduction_kernel[(1, )](
        input_tensor,
        output,
        red_output,
        M,
        N,
        red_op,
        use_abs,
        propagate_nan,
        num_warps=num_warps,
    )

    # Verify full output matches input (tmem store/load roundtrip)
    # Use equal_nan=True when we have NaN values in the input
    torch.testing.assert_close(input_tensor, output, atol=0, rtol=0, equal_nan=use_nan)

    # Compute expected reduction
    ref_input = torch.abs(input_tensor) if use_abs else input_tensor
    torch_red = torch.min if red_op == "min" else torch.max
    expected_red = torch_red(ref_input, dim=1).values

    # Verify reduction output
    # Use equal_nan=True when testing NaN propagation
    torch.testing.assert_close(expected_red, red_output, atol=1e-5, rtol=1e-5, equal_nan=use_nan)
```
- **EN:** Prepares or updates state through `output`, `red_output`, `ref_input`, `torch_red`, `expected_red`. Invokes `torch.empty_like`, `torch.empty`, `torch.testing.assert_close`, `torch.abs`, `torch_red` to execute the test logic. Relevant themes: PyTorch tensor setup and checks.
- **CN:** 通过 `output`、`red_output`、`ref_input`、`torch_red`、`expected_red` 准备或更新状态。 调用 `torch.empty_like`、`torch.empty`、`torch.testing.assert_close`、`torch.abs`、`torch_red` 执行测试逻辑。 相关主题：PyTorch 张量准备与校验。

### Lines 4094-4104

```python


@pytest.mark.parametrize("num_ctas", [1, 2])
@pytest.mark.skipif(not is_blackwell(), reason="Requires Blackwell")
def test_clc_basic(num_ctas):
    # Launch a grid with 2x the number of CTAs as the number of SMs
    # And ask to allocate a big chunk of smem per block (almost all
    # the smem minus 32 i64 elements to make room for the barriers),
    # so that we force 1 block per SM

    @gluon.jit
```
- **EN:** Defines the test function `test_clc_basic`. Decorators: `pytest.mark.parametrize('num_ctas', [1, 2])`, `pytest.mark.skipif(not is_blackwell(), reason='Requires Blackwell')`. Parameters: `num_ctas`. Nested definitions in this scope: `clc_kernel`. Key calls include `pytest.mark.parametrize`, `pytest.mark.skipif`, `torch.cuda.get_device_properties`, `torch.zeros`, `mbarrier.allocate_mbarrier`, `ttgl.SwizzledSharedLayout`, and 14 more. This scope touches pytest parametrization, PyTorch tensor setup and checks, kernel launch orchestration, layout transformation reasoning.
- **CN:** 定义测试函数 `test_clc_basic`。 装饰器：`pytest.mark.parametrize('num_ctas', [1, 2])`、`pytest.mark.skipif(not is_blackwell(), reason='Requires Blackwell')`。 参数：`num_ctas`。 该作用域中的嵌套定义：`clc_kernel`。 关键调用包括 `pytest.mark.parametrize`、`pytest.mark.skipif`、`torch.cuda.get_device_properties`、`torch.zeros`、`mbarrier.allocate_mbarrier`、`ttgl.SwizzledSharedLayout` 等另外 14 项。 该作用域涉及pytest 参数化、PyTorch 张量准备与校验、kernel 启动编排、布局变换推理。

#### Lines 4104-4107

```python
    @gluon.jit
    def clc_kernel(WasLaunched, IsCancelled, ProgramId, smem_size: ttgl.constexpr):
        # Allocate clc_mbar before clc_result to make sure that we are indeed aligning
        # clc_result correctly after a i64 element.
```
- **EN:** Defines the helper function `clc_kernel`. Decorators: `gluon.jit`. Parameters: `WasLaunched`, `IsCancelled`, `ProgramId`, `smem_size`. Key calls include `mbarrier.allocate_mbarrier`, `ttgl.SwizzledSharedLayout`, `ttgl.allocate_shared_memory`, `mbarrier.init`, `clc.try_cancel`, `mbarrier.expect`, and 8 more. This scope touches kernel launch orchestration, layout transformation reasoning.
- **CN:** 定义辅助函数 `clc_kernel`。 装饰器：`gluon.jit`。 参数：`WasLaunched`、`IsCancelled`、`ProgramId`、`smem_size`。 关键调用包括 `mbarrier.allocate_mbarrier`、`ttgl.SwizzledSharedLayout`、`ttgl.allocate_shared_memory`、`mbarrier.init`、`clc.try_cancel`、`mbarrier.expect` 等另外 8 项。 该作用域涉及kernel 启动编排、布局变换推理。

##### Lines 4108-4126

```python
        clc_mbar = mbarrier.allocate_mbarrier()
        cga_layout: ttgl.constexpr = [[0]] * (ttgl.num_ctas().bit_length() - 1)
        layout: ttgl.constexpr = ttgl.SwizzledSharedLayout(1, 1, 1, order=[0], cga_layout=cga_layout)
        clc_result = ttgl.allocate_shared_memory(ttgl.int64, [2], layout)
        mbarrier.init(clc_mbar, count=1)

        # Large shared memory allocation to force 1 block per SM
        dummy = ttgl.allocate_shared_memory(ttgl.int64, [smem_size // 8 - 32], clc_mbar.layout)

        clc.try_cancel(clc_result, clc_mbar)
        mbarrier.expect(clc_mbar, 16)
        mbarrier.wait(clc_mbar, 0)

        response = clc.load_result(clc_result)
        pid = ttgl.program_id(0)
        ttgl.store(WasLaunched + pid, True)
        ttgl.store(IsCancelled + pid, response.is_canceled())
        ttgl.store(ProgramId + pid, response.program_id(0))
        dummy._keep_alive()
```
- **EN:** Prepares or updates state through `clc_mbar`, `cga_layout`, `layout`, `clc_result`, `dummy`, `response`, `pid`. Invokes `mbarrier.allocate_mbarrier`, `ttgl.num_ctas`, `ttgl.SwizzledSharedLayout`, `ttgl.allocate_shared_memory`, `mbarrier.init`, `clc.try_cancel`, and 8 more to execute the test logic. Relevant themes: kernel launch orchestration, layout transformation reasoning.
- **CN:** 通过 `clc_mbar`、`cga_layout`、`layout`、`clc_result`、`dummy`、`response`、`pid` 准备或更新状态。 调用 `mbarrier.allocate_mbarrier`、`ttgl.num_ctas`、`ttgl.SwizzledSharedLayout`、`ttgl.allocate_shared_memory`、`mbarrier.init`、`clc.try_cancel` 等另外 8 项 执行测试逻辑。 相关主题：kernel 启动编排、布局变换推理。

#### Lines 4127-4138

```python

    dev_props = torch.cuda.get_device_properties("cuda")
    num_sms = dev_props.multi_processor_count
    smem_size = dev_props.shared_memory_per_block_optin
    grid = 2 * (num_sms // num_ctas)

    was_launched = torch.zeros([grid], dtype=torch.bool, device="cuda")
    is_cancelled = torch.zeros([grid], dtype=torch.bool, device="cuda")
    program_ids = torch.zeros([grid], dtype=torch.int32, device="cuda")
    clc_kernel[(grid, )](was_launched, is_cancelled, program_ids, smem_size, num_ctas=num_ctas)

    num_launched = torch.sum(was_launched).item()
```
- **EN:** Prepares or updates state through `dev_props`, `num_sms`, `smem_size`, `grid`, `was_launched`, `is_cancelled`, `program_ids`, `num_launched`. Invokes `torch.cuda.get_device_properties`, `torch.zeros`, `torch.sum` to execute the test logic. Relevant themes: PyTorch tensor setup and checks, kernel launch orchestration.
- **CN:** 通过 `dev_props`、`num_sms`、`smem_size`、`grid`、`was_launched`、`is_cancelled`、`program_ids`、`num_launched` 准备或更新状态。 调用 `torch.cuda.get_device_properties`、`torch.zeros`、`torch.sum` 执行测试逻辑。 相关主题：PyTorch 张量准备与校验、kernel 启动编排。

#### Lines 4139-4139

```python
    assert num_launched < grid
```
- **EN:** Validates behavior with 1 assertion(s). Relevant themes: kernel launch orchestration.
- **CN:** 通过 1 个断言验证行为。 相关主题：kernel 启动编排。

#### Lines 4140-4141

```python

    num_cancelled = torch.sum(is_cancelled).item()
```
- **EN:** Prepares or updates state through `num_cancelled`. Invokes `torch.sum` to execute the test logic. Relevant themes: PyTorch tensor setup and checks.
- **CN:** 通过 `num_cancelled` 准备或更新状态。 调用 `torch.sum` 执行测试逻辑。 相关主题：PyTorch 张量准备与校验。

#### Lines 4142-4142

```python
    assert num_launched + num_cancelled == grid
```
- **EN:** Validates behavior with 1 assertion(s). Relevant themes: kernel launch orchestration.
- **CN:** 通过 1 个断言验证行为。 相关主题：kernel 启动编排。

#### Lines 4143-4146

```python

    for pid in range(grid):
        if is_cancelled[pid]:
            assert not was_launched[program_ids[pid]]
```
- **EN:** Validates behavior with 1 assertion(s). Iterates across cases or data tiles. Relevant themes: kernel launch orchestration.
- **CN:** 通过 1 个断言验证行为。 通过循环覆盖多个用例或数据分块。 相关主题：kernel 启动编排。

### Lines 4147-4149

```python


def align_to(a, b):
```
- **EN:** Defines the helper function `align_to`. Parameters: `a`, `b`. Key calls include `triton.cdiv`.
- **CN:** 定义辅助函数 `align_to`。 参数：`a`、`b`。 关键调用包括 `triton.cdiv`。

#### Lines 4150-4150

```python
    return triton.cdiv(a, b) * b
```
- **EN:** Invokes `triton.cdiv` to execute the test logic.
- **CN:** 调用 `triton.cdiv` 执行测试逻辑。

### Lines 4151-4153

```python


def make_operand_descriptor(value, BLOCK_MN, BLOCK_K, MIXED_PREC, cga_layout=None):
```
- **EN:** Defines the helper function `make_operand_descriptor`. Parameters: `value`, `BLOCK_MN`, `BLOCK_K`, `MIXED_PREC`, `cga_layout`. Key calls include `ttgl.NVMMASharedLayout.get_default_for`, `TensorDescriptor.from_tensor`. This scope touches PyTorch tensor setup and checks, tensor/descriptor metadata, layout transformation reasoning, random-data generation.
- **CN:** 定义辅助函数 `make_operand_descriptor`。 参数：`value`、`BLOCK_MN`、`BLOCK_K`、`MIXED_PREC`、`cga_layout`。 关键调用包括 `ttgl.NVMMASharedLayout.get_default_for`、`TensorDescriptor.from_tensor`。 该作用域涉及PyTorch 张量准备与校验、张量/描述符元数据、布局变换推理、随机数据生成。

#### Lines 4154-4163

```python
    IS_FP4 = value.dtype == torch.uint8
    ELEM_PER_BYTE = 2 if IS_FP4 else 1
    IS_MIXED_PREC_FP4 = MIXED_PREC and IS_FP4
    layout = ttgl.NVMMASharedLayout.get_default_for(
        [BLOCK_MN, BLOCK_K // ELEM_PER_BYTE],
        ttgl.uint8 if IS_FP4 else ttgl.float8e4nv,
        fp4_padded=IS_MIXED_PREC_FP4,
        cga_layout=cga_layout,
    )
    return TensorDescriptor.from_tensor(value, [BLOCK_MN, BLOCK_K // ELEM_PER_BYTE], layout)
```
- **EN:** Prepares or updates state through `IS_FP4`, `ELEM_PER_BYTE`, `IS_MIXED_PREC_FP4`, `layout`. Invokes `ttgl.NVMMASharedLayout.get_default_for`, `TensorDescriptor.from_tensor` to execute the test logic. Relevant themes: PyTorch tensor setup and checks, tensor/descriptor metadata, layout transformation reasoning.
- **CN:** 通过 `IS_FP4`、`ELEM_PER_BYTE`、`IS_MIXED_PREC_FP4`、`layout` 准备或更新状态。 调用 `ttgl.NVMMASharedLayout.get_default_for`、`TensorDescriptor.from_tensor` 执行测试逻辑。 相关主题：PyTorch 张量准备与校验、张量/描述符元数据、布局变换推理。

### Lines 4164-4166

```python


def make_output_descriptor(M, N, dtype, BLOCK_M, BLOCK_N, cga_layout=None):
```
- **EN:** Defines the helper function `make_output_descriptor`. Parameters: `M`, `N`, `dtype`, `BLOCK_M`, `BLOCK_N`, `cga_layout`. Key calls include `torch.empty`, `getattr`, `ttgl.NVMMASharedLayout.get_default_for`, `TensorDescriptor.from_tensor`. This scope touches PyTorch tensor setup and checks, tensor/descriptor metadata, layout transformation reasoning.
- **CN:** 定义辅助函数 `make_output_descriptor`。 参数：`M`、`N`、`dtype`、`BLOCK_M`、`BLOCK_N`、`cga_layout`。 关键调用包括 `torch.empty`、`getattr`、`ttgl.NVMMASharedLayout.get_default_for`、`TensorDescriptor.from_tensor`。 该作用域涉及PyTorch 张量准备与校验、张量/描述符元数据、布局变换推理。

#### Lines 4167-4170

```python
    C = torch.empty(M, N, device="cuda", dtype=dtype)
    C_dtype = getattr(ttgl, str(dtype).split('.')[1])
    C_desc_layout = ttgl.NVMMASharedLayout.get_default_for([BLOCK_M, BLOCK_N], C_dtype, cga_layout=cga_layout)
    return TensorDescriptor.from_tensor(C, [BLOCK_M, BLOCK_N], C_desc_layout)
```
- **EN:** Prepares or updates state through `C`, `C_dtype`, `C_desc_layout`. Invokes `torch.empty`, `getattr`, `ttgl.NVMMASharedLayout.get_default_for`, `TensorDescriptor.from_tensor` to execute the test logic. Relevant themes: PyTorch tensor setup and checks, tensor/descriptor metadata, layout transformation reasoning.
- **CN:** 通过 `C`、`C_dtype`、`C_desc_layout` 准备或更新状态。 调用 `torch.empty`、`getattr`、`ttgl.NVMMASharedLayout.get_default_for`、`TensorDescriptor.from_tensor` 执行测试逻辑。 相关主题：PyTorch 张量准备与校验、张量/描述符元数据、布局变换推理。

### Lines 4171-4173

```python


def random_quantized_tensor(MN, K, format):
```
- **EN:** Defines the helper function `random_quantized_tensor`. Parameters: `MN`, `K`, `format`. Key calls include `base.to`, `scale.to`, `scale_ref.repeat_interleave`, `MXFP4Tensor`, `MXScaleTensor`, `ref.to`, and 2 more. This scope touches PyTorch tensor setup and checks, random-data generation.
- **CN:** 定义辅助函数 `random_quantized_tensor`。 参数：`MN`、`K`、`format`。 关键调用包括 `base.to`、`scale.to`、`scale_ref.repeat_interleave`、`MXFP4Tensor`、`MXScaleTensor`、`ref.to` 等另外 2 项。 该作用域涉及PyTorch 张量准备与校验、随机数据生成。

#### Lines 4174-4174

```python
    assert format in ["mxfp4", "mxfp8", "nvfp4"]
```
- **EN:** Validates behavior with 1 assertion(s).
- **CN:** 通过 1 个断言验证行为。

#### Lines 4175-4180

```python
    VEC_SIZE = 16 if format == "nvfp4" else 32
    base = MXFP4Tensor(size=(MN, K), device="cuda").random()
    scale = MXScaleTensor(size=(MN, K // VEC_SIZE), device="cuda").random(low=1 / 128, high=2.0)
    ref = base.to(torch.float32)
    scale_ref = scale.to(torch.float32)
    value = ref * scale_ref.repeat_interleave(VEC_SIZE, dim=1)
```
- **EN:** Prepares or updates state through `VEC_SIZE`, `base`, `scale`, `ref`, `scale_ref`, `value`. Invokes `MXFP4Tensor`, `MXScaleTensor`, `base.to`, `scale.to`, `scale_ref.repeat_interleave` to execute the test logic. Relevant themes: PyTorch tensor setup and checks, random-data generation.
- **CN:** 通过 `VEC_SIZE`、`base`、`scale`、`ref`、`scale_ref`、`value` 准备或更新状态。 调用 `MXFP4Tensor`、`MXScaleTensor`、`base.to`、`scale.to`、`scale_ref.repeat_interleave` 执行测试逻辑。 相关主题：PyTorch 张量准备与校验、随机数据生成。

#### Lines 4181-4186

```python
    if format == "mxfp8":
        return ref.to(torch.float8_e4m3fn), scale.data, value
    elif format == "mxfp4":
        return base.to_packed_tensor(dim=1), scale.data, value
    else:
        return base.to_packed_tensor(dim=1), scale_ref.to(torch.float8_e4m3fn), value
```
- **EN:** Invokes `ref.to`, `base.to_packed_tensor`, `scale_ref.to` to execute the test logic. Branches on runtime or test conditions. Relevant themes: PyTorch tensor setup and checks.
- **CN:** 调用 `ref.to`、`base.to_packed_tensor`、`scale_ref.to` 执行测试逻辑。 根据运行时或测试条件进行分支。 相关主题：PyTorch 张量准备与校验。

### Lines 4187-4189

```python


def swizzle_scales_packed_block(scales, VEC_SIZE):
```
- **EN:** Defines the helper function `swizzle_scales_packed_block`. Parameters: `scales`, `VEC_SIZE`. Key calls include `torch.nn.functional.pad`, `scales.reshape`, `scales.permute`, `scales.contiguous`, `align_to`. This scope touches PyTorch tensor setup and checks.
- **CN:** 定义辅助函数 `swizzle_scales_packed_block`。 参数：`scales`、`VEC_SIZE`。 关键调用包括 `torch.nn.functional.pad`、`scales.reshape`、`scales.permute`、`scales.contiguous`、`align_to`。 该作用域涉及PyTorch 张量准备与校验。

#### Lines 4190-4198

```python
    PAD_MN = align_to(scales.shape[0], 128) - scales.shape[0]
    PAD_K = align_to(scales.shape[1], 4) - scales.shape[1]
    scales = torch.nn.functional.pad(scales, (0, PAD_K, 0, PAD_MN))
    MN, SCALE_K = scales.shape[0], scales.shape[1]
    REP_MN = MN // 128
    REP_K = SCALE_K // 4
    scales = scales.reshape(REP_MN, 4, 32, REP_K, 4)
    scales = scales.permute(0, 3, 2, 1, 4)
    return scales.contiguous()
```
- **EN:** Prepares or updates state through `PAD_MN`, `PAD_K`, `scales`, `MN`, `SCALE_K`, `REP_MN`, `REP_K`. Invokes `align_to`, `torch.nn.functional.pad`, `scales.reshape`, `scales.permute`, `scales.contiguous` to execute the test logic. Relevant themes: PyTorch tensor setup and checks.
- **CN:** 通过 `PAD_MN`、`PAD_K`、`scales`、`MN`、`SCALE_K`、`REP_MN`、`REP_K` 准备或更新状态。 调用 `align_to`、`torch.nn.functional.pad`、`scales.reshape`、`scales.permute`、`scales.contiguous` 执行测试逻辑。 相关主题：PyTorch 张量准备与校验。

### Lines 4199-4201

```python


def make_scales_descriptor(scales, BLOCK_MN, BLOCK_K, VEC_SIZE, cga_layout=None):
```
- **EN:** Defines the helper function `make_scales_descriptor`. Parameters: `scales`, `BLOCK_MN`, `BLOCK_K`, `VEC_SIZE`, `cga_layout`. Key calls include `scales.reshape`, `ttgl.NVMMASharedLayout.get_default_for`, `TensorDescriptor.from_tensor`. This scope touches PyTorch tensor setup and checks, tensor/descriptor metadata, layout transformation reasoning.
- **CN:** 定义辅助函数 `make_scales_descriptor`。 参数：`scales`、`BLOCK_MN`、`BLOCK_K`、`VEC_SIZE`、`cga_layout`。 关键调用包括 `scales.reshape`、`ttgl.NVMMASharedLayout.get_default_for`、`TensorDescriptor.from_tensor`。 该作用域涉及PyTorch 张量准备与校验、张量/描述符元数据、布局变换推理。

#### Lines 4202-4209

```python
    REP_MN = BLOCK_MN // 128
    REP_K = BLOCK_K // (VEC_SIZE * 4)
    block_shape = [1, REP_MN, REP_K, 2, 256]
    scales = scales.reshape(1, scales.shape[0], scales.shape[1], 2, 256)
    IS_NVFP4 = scales.dtype == torch.float8_e4m3fn
    layout = ttgl.NVMMASharedLayout.get_default_for(block_shape, ttgl.float8e4nv if IS_NVFP4 else ttgl.uint8,
                                                    cga_layout=cga_layout)
    return TensorDescriptor.from_tensor(scales, block_shape, layout)
```
- **EN:** Prepares or updates state through `REP_MN`, `REP_K`, `block_shape`, `scales`, `IS_NVFP4`, `layout`. Invokes `scales.reshape`, `ttgl.NVMMASharedLayout.get_default_for`, `TensorDescriptor.from_tensor` to execute the test logic. Relevant themes: PyTorch tensor setup and checks, tensor/descriptor metadata, layout transformation reasoning.
- **CN:** 通过 `REP_MN`、`REP_K`、`block_shape`、`scales`、`IS_NVFP4`、`layout` 准备或更新状态。 调用 `scales.reshape`、`ttgl.NVMMASharedLayout.get_default_for`、`TensorDescriptor.from_tensor` 执行测试逻辑。 相关主题：PyTorch 张量准备与校验、张量/描述符元数据、布局变换推理。

### Lines 4210-4212

```python


def get_cga_layout(layout, op_idx):
```
- **EN:** Defines the helper function `get_cga_layout`. Parameters: `layout`, `op_idx`. Nested definitions in this scope: `broadcast`. Key calls include `map`. This scope touches matrix multiplication workflows, layout transformation reasoning.
- **CN:** 定义辅助函数 `get_cga_layout`。 参数：`layout`、`op_idx`。 该作用域中的嵌套定义：`broadcast`。 关键调用包括 `map`。 该作用域涉及矩阵乘法工作流、布局变换推理。

#### Lines 4213-4214

```python
    """Based on get_cga_layout from 03-matmul-multicta.py.
    Difference: B is N×K (transposed), so offsets are in dim 0 not dim 1."""
```
- **EN:** Relevant themes: matrix multiplication workflows, layout transformation reasoning.
- **CN:** 相关主题：矩阵乘法工作流、布局变换推理。

#### Lines 4215-4215

```python
    assert op_idx in (0, 1)
```
- **EN:** Validates behavior with 1 assertion(s).
- **CN:** 通过 1 个断言验证行为。

#### Lines 4216-4217

```python
    if not layout:
        return layout
```
- **EN:** Branches on runtime or test conditions. Relevant themes: layout transformation reasoning.
- **CN:** 根据运行时或测试条件进行分支。 相关主题：布局变换推理。

#### Lines 4218-4220

```python

    # 2CTA performs an outer product so bases are [1, 0] and [0, 1]
    assert list(layout[0]) == [1, 0]
```
- **EN:** Validates behavior with 1 assertion(s). Relevant themes: layout transformation reasoning.
- **CN:** 通过 1 个断言验证行为。 相关主题：布局变换推理。

#### Lines 4221-4221

```python
    first = (1, 0)
```
- **EN:** Prepares or updates state through `first`.
- **CN:** 通过 `first` 准备或更新状态。

#### Lines 4222-4225

```python

    # Broadcast along K (the reduction dimension)
    # We multiply by 2 for op_idx == 1, as we have added the (1, 0) basis.
    def broadcast(b):
```
- **EN:** Defines the helper function `broadcast`. Parameters: `b`.
- **CN:** 定义辅助函数 `broadcast`。 参数：`b`。

##### Lines 4226-4226

```python
        return (b[0], 0) if op_idx == 0 else (2 * b[1], 0)
```
- **EN:** Carries supporting logic for the surrounding test or helper scope.
- **CN:** 为周围的测试或辅助作用域提供支撑逻辑。

#### Lines 4227-4228

```python

    return (first, *map(broadcast, layout[1:]))
```
- **EN:** Invokes `map` to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 调用 `map` 执行测试逻辑。 相关主题：布局变换推理。

### Lines 4229-4232

```python


@gluon.jit
def unswizzle_scales_shared_memory(smem, BLOCK_MN: ttgl.constexpr, BLOCK_K: ttgl.constexpr, VEC_SIZE: ttgl.constexpr):
```
- **EN:** Defines the helper function `unswizzle_scales_shared_memory`. Decorators: `gluon.jit`. Parameters: `smem`, `BLOCK_MN`, `BLOCK_K`, `VEC_SIZE`. Key calls include `smem.reshape`, `smem.permute`.
- **CN:** 定义辅助函数 `unswizzle_scales_shared_memory`。 装饰器：`gluon.jit`。 参数：`smem`、`BLOCK_MN`、`BLOCK_K`、`VEC_SIZE`。 关键调用包括 `smem.reshape`、`smem.permute`。

#### Lines 4233-4235

```python
    smem = smem.reshape((smem.shape[1], smem.shape[2], 32, 4, 4))
    smem = smem.permute((0, 3, 2, 1, 4))
    return smem.reshape((BLOCK_MN, BLOCK_K // VEC_SIZE))
```
- **EN:** Prepares or updates state through `smem`. Invokes `smem.reshape`, `smem.permute` to execute the test logic.
- **CN:** 通过 `smem` 准备或更新状态。 调用 `smem.reshape`、`smem.permute` 执行测试逻辑。

### Lines 4236-4242

```python


@gluon.jit
def mma_scaled_tcgen05_copy_kernel(a_desc, b_desc, c_desc, a_scale_desc, b_scale_desc, VEC_SIZE: ttgl.constexpr,
                                   block_layout_c: ttgl.constexpr, a_scale_layout_tmem: ttgl.constexpr,
                                   b_scale_layout_tmem: ttgl.constexpr, ctas_per_cga: ttgl.constexpr,
                                   multicast: ttgl.constexpr):
```
- **EN:** Defines the helper function `mma_scaled_tcgen05_copy_kernel`. Decorators: `gluon.jit`. Parameters: `a_desc`, `b_desc`, `c_desc`, `a_scale_desc`, `b_scale_desc`, `VEC_SIZE`, `block_layout_c`, `a_scale_layout_tmem`, and 3 more. Key calls include `ttgl.allocate_shared_memory`, `ttgl.num_ctas`, `TensorMemoryScalesLayout`, `allocate_tensor_memory`, `TensorMemoryLayout`, `mbarrier.allocate_mbarrier`, and 17 more. This scope touches layout transformation reasoning.
- **CN:** 定义辅助函数 `mma_scaled_tcgen05_copy_kernel`。 装饰器：`gluon.jit`。 参数：`a_desc`、`b_desc`、`c_desc`、`a_scale_desc`、`b_scale_desc`、`VEC_SIZE`、`block_layout_c`、`a_scale_layout_tmem` 等另外 3 项。 关键调用包括 `ttgl.allocate_shared_memory`、`ttgl.num_ctas`、`TensorMemoryScalesLayout`、`allocate_tensor_memory`、`TensorMemoryLayout`、`mbarrier.allocate_mbarrier` 等另外 17 项。 该作用域涉及布局变换推理。

#### Lines 4243-4291

```python
    A_IS_FP4: ttgl.constexpr = a_desc.dtype == ttgl.uint8
    B_IS_FP4: ttgl.constexpr = b_desc.dtype == ttgl.uint8
    A_ELEM_PER_BYTE: ttgl.constexpr = 2 if A_IS_FP4 else 1
    B_ELEM_PER_BYTE: ttgl.constexpr = 2 if B_IS_FP4 else 1
    BLOCK_M: ttgl.constexpr = c_desc.block_type.shape[0]
    BLOCK_N: ttgl.constexpr = c_desc.block_type.shape[1]
    BLOCK_K: ttgl.constexpr = a_desc.block_type.shape[1] * A_ELEM_PER_BYTE
    K = a_desc.shape[1] * A_ELEM_PER_BYTE

    a_smem = ttgl.allocate_shared_memory(a_desc.dtype, a_desc.block_type.shape, a_desc.layout)
    b_smem = ttgl.allocate_shared_memory(b_desc.dtype, b_desc.block_type.shape, b_desc.layout)

    num_ctas: ttgl.constexpr = ttgl.num_ctas()
    multi_cta: ttgl.constexpr = num_ctas > 1
    scale_layout_a: ttgl.constexpr = TensorMemoryScalesLayout(cga_layout=a_scale_layout_tmem)
    scale_layout_b: ttgl.constexpr = TensorMemoryScalesLayout(cga_layout=b_scale_layout_tmem)
    a_scale_tmem = allocate_tensor_memory(a_scale_desc.dtype, [BLOCK_M, BLOCK_K // VEC_SIZE], scale_layout_a)
    b_scale_tmem = allocate_tensor_memory(b_scale_desc.dtype, [BLOCK_N, BLOCK_K // VEC_SIZE], scale_layout_b)
    tmem_layout: ttgl.constexpr = TensorMemoryLayout(
        [BLOCK_M // ctas_per_cga[0], BLOCK_N // ctas_per_cga[1]],
        col_stride=1,
        cga_layout=c_desc.layout.cga_layout,
        two_ctas=multi_cta,
    )
    acc_tmem = allocate_tensor_memory(ttgl.float32, [BLOCK_M, BLOCK_N], tmem_layout)

    tma_bar = mbarrier.allocate_mbarrier(two_ctas=multi_cta)
    mbarrier.init(tma_bar, count=1)

    mma_bar = mbarrier.allocate_mbarrier()
    mma_bar_count: ttgl.constexpr = tcgen05_mma_barrier_count([a_smem, b_smem], multicast=multicast,
                                                              two_ctas=acc_tmem.type.layout.two_ctas)
    mbarrier.init(mma_bar, count=mma_bar_count)

    phase_tma = 0
    phase_mma = 0
    pid_m = ttgl.program_id(0)
    pid_n = ttgl.program_id(1)
    off_m = pid_m * BLOCK_M
    off_n = pid_n * BLOCK_N

    a_scale_smem = ttgl.allocate_shared_memory(a_scale_desc.dtype, a_scale_desc.block_type.shape, a_scale_desc.layout)
    b_scale_smem = ttgl.allocate_shared_memory(b_scale_desc.dtype, b_scale_desc.block_type.shape, b_scale_desc.layout)
    REP_M: ttgl.constexpr = a_scale_desc.block_type.shape[1]
    REP_N: ttgl.constexpr = b_scale_desc.block_type.shape[1]
    A_REP_K: ttgl.constexpr = a_scale_desc.block_type.shape[2]
    B_REP_K: ttgl.constexpr = b_scale_desc.block_type.shape[2]
    off_m_a_scale = pid_m * REP_M
    off_n_b_scale = pid_n * REP_N
```
- **EN:** Prepares or updates state through `A_IS_FP4`, `B_IS_FP4`, `A_ELEM_PER_BYTE`, `B_ELEM_PER_BYTE`, `BLOCK_M`, `BLOCK_N`, `BLOCK_K`, `K`, and 27 more. Invokes `ttgl.allocate_shared_memory`, `ttgl.num_ctas`, `TensorMemoryScalesLayout`, `allocate_tensor_memory`, `TensorMemoryLayout`, `mbarrier.allocate_mbarrier`, and 3 more to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `A_IS_FP4`、`B_IS_FP4`、`A_ELEM_PER_BYTE`、`B_ELEM_PER_BYTE`、`BLOCK_M`、`BLOCK_N`、`BLOCK_K`、`K` 等另外 27 项 准备或更新状态。 调用 `ttgl.allocate_shared_memory`、`ttgl.num_ctas`、`TensorMemoryScalesLayout`、`allocate_tensor_memory`、`TensorMemoryLayout`、`mbarrier.allocate_mbarrier` 等另外 3 项 执行测试逻辑。 相关主题：布局变换推理。

#### Lines 4292-4321

```python

    for k in range(0, K, BLOCK_K):
        off_k_a = k // A_ELEM_PER_BYTE
        off_k_b = k // B_ELEM_PER_BYTE
        off_k_a_scale = (k // BLOCK_K) * A_REP_K
        off_k_b_scale = (k // BLOCK_K) * B_REP_K

        EXPECTED_BYTES: ttgl.constexpr = (a_desc.nbytes_per_cta + b_desc.nbytes_per_cta + a_scale_desc.nbytes_per_cta +
                                          b_scale_desc.nbytes_per_cta)
        mbarrier.expect(tma_bar, EXPECTED_BYTES)
        tma.async_load(a_desc, [off_m, off_k_a], tma_bar, a_smem, multicast=multicast)
        tma.async_load(b_desc, [off_n, off_k_b], tma_bar, b_smem, multicast=multicast)
        tma.async_load(a_scale_desc, [0, off_m_a_scale, off_k_a_scale, 0, 0], tma_bar, a_scale_smem,
                       multicast=multicast)
        tma.async_load(b_scale_desc, [0, off_n_b_scale, off_k_b_scale, 0, 0], tma_bar, b_scale_smem,
                       multicast=multicast)
        mbarrier.wait(tma_bar, phase_tma, deps=[a_smem, b_smem, a_scale_smem, b_scale_smem])
        phase_tma ^= 1

        a_scale = unswizzle_scales_shared_memory(a_scale_smem, BLOCK_M, BLOCK_K, VEC_SIZE)
        b_scale = unswizzle_scales_shared_memory(b_scale_smem, BLOCK_N, BLOCK_K, VEC_SIZE)
        tcgen05_copy(a_scale, a_scale_tmem)
        tcgen05_copy(b_scale, b_scale_tmem)

        a_format: ttgl.constexpr = "e2m1" if A_IS_FP4 else "e4m3"
        b_format: ttgl.constexpr = "e2m1" if B_IS_FP4 else "e4m3"
        tcgen05_mma_scaled(a_smem, b_smem.permute((1, 0)), acc_tmem, a_scale_tmem, b_scale_tmem, a_format, b_format,
                           use_acc=(k != 0), multicast=multicast, mbarriers=[mma_bar])
        mbarrier.wait(mma_bar, phase_mma)
        phase_mma ^= 1
```
- **EN:** Invokes `mbarrier.expect`, `tma.async_load`, `mbarrier.wait`, `unswizzle_scales_shared_memory`, `tcgen05_copy`, `tcgen05_mma_scaled`, and 1 more to execute the test logic. Iterates across cases or data tiles.
- **CN:** 调用 `mbarrier.expect`、`tma.async_load`、`mbarrier.wait`、`unswizzle_scales_shared_memory`、`tcgen05_copy`、`tcgen05_mma_scaled` 等另外 1 项 执行测试逻辑。 通过循环覆盖多个用例或数据分块。

#### Lines 4322-4325

```python

    mbarrier.invalidate(tma_bar)
    mbarrier.invalidate(mma_bar)
    acc = acc_tmem.load()
```
- **EN:** Prepares or updates state through `acc`. Invokes `mbarrier.invalidate`, `acc_tmem.load` to execute the test logic.
- **CN:** 通过 `acc` 准备或更新状态。 调用 `mbarrier.invalidate`、`acc_tmem.load` 执行测试逻辑。

#### Lines 4326-4327

```python
    if multi_cta:
        acc = ttgl.convert_layout(acc, block_layout_c)
```
- **EN:** Invokes `ttgl.convert_layout` to execute the test logic. Branches on runtime or test conditions. Relevant themes: layout transformation reasoning.
- **CN:** 调用 `ttgl.convert_layout` 执行测试逻辑。 根据运行时或测试条件进行分支。 相关主题：布局变换推理。

#### Lines 4328-4332

```python
    acc = acc.to(c_desc.dtype)
    acc_smem = ttgl.allocate_shared_memory(c_desc.dtype, c_desc.block_type.shape, c_desc.layout)
    acc_smem.store(acc)
    tma.async_copy_shared_to_global(c_desc, [off_m, off_n], acc_smem)
    tma.store_wait(0)
```
- **EN:** Prepares or updates state through `acc`, `acc_smem`. Invokes `acc.to`, `ttgl.allocate_shared_memory`, `acc_smem.store`, `tma.async_copy_shared_to_global`, `tma.store_wait` to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `acc`、`acc_smem` 准备或更新状态。 调用 `acc.to`、`ttgl.allocate_shared_memory`、`acc_smem.store`、`tma.async_copy_shared_to_global`、`tma.store_wait` 执行测试逻辑。 相关主题：布局变换推理。

### Lines 4333-4336

```python


def mma_scaled_tcgen05_copy(A, B, A_scale, B_scale, VEC_SIZE, BLOCK_M, BLOCK_N, BLOCK_K, ctas_per_cga, multicast,
                            out_dtype=torch.float16):
```
- **EN:** Defines the helper function `mma_scaled_tcgen05_copy`. Parameters: `A`, `B`, `A_scale`, `B_scale`, `VEC_SIZE`, `BLOCK_M`, `BLOCK_N`, `BLOCK_K`, and 3 more. Key calls include `get_cga_layout`, `make_operand_descriptor`, `make_output_descriptor`, `make_scales_descriptor`, `ttgl.NVMMASharedLayout`, `replace`, and 3 more. This scope touches PyTorch tensor setup and checks, tensor/descriptor metadata, layout transformation reasoning, random-data generation.
- **CN:** 定义辅助函数 `mma_scaled_tcgen05_copy`。 参数：`A`、`B`、`A_scale`、`B_scale`、`VEC_SIZE`、`BLOCK_M`、`BLOCK_N`、`BLOCK_K` 等另外 3 项。 关键调用包括 `get_cga_layout`、`make_operand_descriptor`、`make_output_descriptor`、`make_scales_descriptor`、`ttgl.NVMMASharedLayout`、`replace` 等另外 3 项。 该作用域涉及PyTorch 张量准备与校验、张量/描述符元数据、布局变换推理、随机数据生成。

#### Lines 4337-4337

```python
    from dataclasses import replace
```
- **EN:** Imports the modules used in this scope: `dataclasses`.
- **CN:** 导入此作用域使用的模块：`dataclasses`。

#### Lines 4338-4379

```python
    M, N = A.shape[0], B.shape[0]
    MIXED_PREC = A.dtype != B.dtype
    num_ctas = ctas_per_cga[0] * ctas_per_cga[1]
    multi_cta = num_ctas > 1
    warps = [4, 1]
    num_warps = warps[0] * warps[1]

    cta_order = (1, 0) if multi_cta else None
    cga_layout_c = make_2cta_cga_layout(ctas_per_cga, ctas_per_cga, cta_order, 0) if multi_cta else None
    cga_layout_a = get_cga_layout(cga_layout_c, 0)
    cga_layout_b = get_cga_layout(cga_layout_c, 1)

    # Scale CGA layouts derived from C layout:
    # A scales project M (c[0]), B scales project N (c[1])
    # CGA layouts are 5D (matching scale SMEM shape [1, REP_MN, REP_K, 2, 256])
    # TMEM layouts are 2D (matching scale TMEM shape [BLOCK_MN, BLOCK_K // VEC_SIZE])
    cga_layout_a_scale = [[0, c[0], 0, 0, 0] for c in cga_layout_c] if multi_cta else None
    cga_layout_b_scale = [[0, c[1], 0, 0, 0] for c in cga_layout_c] if multi_cta else None
    a_scale_layout_tmem = tuple((c[0], 0) for c in cga_layout_c) if multi_cta else ()
    b_scale_layout_tmem = tuple((c[1], 0) for c in cga_layout_c) if multi_cta else ()

    A_desc = make_operand_descriptor(A, BLOCK_M, BLOCK_K, MIXED_PREC, cga_layout=cga_layout_a)
    B_desc = make_operand_descriptor(B, BLOCK_N, BLOCK_K, MIXED_PREC, cga_layout=cga_layout_b)
    C_desc = make_output_descriptor(M, N, out_dtype, BLOCK_M, BLOCK_N, cga_layout=cga_layout_c)
    A_scale_desc = make_scales_descriptor(A_scale, BLOCK_M, BLOCK_K, VEC_SIZE, cga_layout=cga_layout_a_scale)
    B_scale_desc = make_scales_descriptor(B_scale, BLOCK_N, BLOCK_K, VEC_SIZE, cga_layout=cga_layout_b_scale)

    a_scale_layout = ttgl.NVMMASharedLayout(swizzle_byte_width=0, element_bitwidth=8, rank=5,
                                            cga_layout=cga_layout_a_scale)
    b_scale_layout = ttgl.NVMMASharedLayout(swizzle_byte_width=0, element_bitwidth=8, rank=5,
                                            cga_layout=cga_layout_b_scale)
    A_scale_desc = replace(A_scale_desc, layout=a_scale_layout)
    B_scale_desc = replace(B_scale_desc, layout=b_scale_layout)

    block_layout_c = ttgl.BlockedLayout([1, 8], [1, 32], warps_per_cta=warps, order=[1, 0],
                                        cga_layout=cga_layout_c) if multi_cta else None

    grid = (triton.cdiv(M, BLOCK_M), triton.cdiv(N, BLOCK_N))
    mma_scaled_tcgen05_copy_kernel[grid](A_desc, B_desc, C_desc, A_scale_desc, B_scale_desc, VEC_SIZE, block_layout_c,
                                         a_scale_layout_tmem, b_scale_layout_tmem, ctas_per_cga, num_warps=num_warps,
                                         num_ctas=num_ctas, multicast=multicast)
    return C_desc.base
```
- **EN:** Prepares or updates state through `M`, `N`, `MIXED_PREC`, `num_ctas`, `multi_cta`, `warps`, `num_warps`, `cta_order`, and 16 more. Invokes `make_2cta_cga_layout`, `get_cga_layout`, `make_operand_descriptor`, `make_output_descriptor`, `make_scales_descriptor`, `ttgl.NVMMASharedLayout`, and 3 more to execute the test logic. Relevant themes: tensor/descriptor metadata, layout transformation reasoning, random-data generation.
- **CN:** 通过 `M`、`N`、`MIXED_PREC`、`num_ctas`、`multi_cta`、`warps`、`num_warps`、`cta_order` 等另外 16 项 准备或更新状态。 调用 `make_2cta_cga_layout`、`get_cga_layout`、`make_operand_descriptor`、`make_output_descriptor`、`make_scales_descriptor`、`ttgl.NVMMASharedLayout` 等另外 3 项 执行测试逻辑。 相关主题：张量/描述符元数据、布局变换推理、随机数据生成。

### Lines 4380-4392

```python


@pytest.mark.parametrize("M, N, K", [(2048, 2048, 4096)])
@pytest.mark.parametrize("BLOCK_K", [128, 256])
@pytest.mark.parametrize("a_format, b_format", [
    ("mxfp8", "mxfp8"),
    ("nvfp4", "nvfp4"),
    ("mxfp8", "mxfp4"),
])
@pytest.mark.parametrize("ctas_per_cga", [(1, 1), (2, 1), (4, 1), (2, 2), (4, 2), (4, 4)])
@pytest.mark.parametrize("multicast", [True, False])
@pytest.mark.skipif(not is_blackwell(), reason="Requires Blackwell")
def test_mma_scaled_tcgen05_copy(M, N, K, BLOCK_K, a_format, b_format, ctas_per_cga, multicast):
```
- **EN:** Defines the test function `test_mma_scaled_tcgen05_copy`. Decorators: `pytest.mark.parametrize('M, N, K', [(2048, 2048, 4096)])`, `pytest.mark.parametrize('BLOCK_K', [128, 256])`, `pytest.mark.parametrize('a_format, b_format', [('mxfp8', 'mxfp8'), ('nvfp4', 'nvfp4'), ('mxfp8', 'mxfp4')])`, `pytest.mark.parametrize('ctas_per_cga', [(1, 1), (2, 1), (4, 1), (2, 2), (4, 2), (4, 4)])`, `pytest.mark.parametrize('multicast', [True, False])`, `pytest.mark.skipif(not is_blackwell(), reason='Requires Blackwell')`. Parameters: `M`, `N`, `K`, `BLOCK_K`, `a_format`, `b_format`, `ctas_per_cga`, `multicast`. Key calls include `pytest.mark.parametrize`, `pytest.mark.skipif`, `torch.manual_seed`, `random_quantized_tensor`, `swizzle_scales_packed_block`, `mma_scaled_tcgen05_copy`, and 3 more. This scope touches pytest parametrization, PyTorch tensor setup and checks, random-data generation.
- **CN:** 定义测试函数 `test_mma_scaled_tcgen05_copy`。 装饰器：`pytest.mark.parametrize('M, N, K', [(2048, 2048, 4096)])`、`pytest.mark.parametrize('BLOCK_K', [128, 256])`、`pytest.mark.parametrize('a_format, b_format', [('mxfp8', 'mxfp8'), ('nvfp4', 'nvfp4'), ('mxfp8', 'mxfp4')])`、`pytest.mark.parametrize('ctas_per_cga', [(1, 1), (2, 1), (4, 1), (2, 2), (4, 2), (4, 4)])`、`pytest.mark.parametrize('multicast', [True, False])`、`pytest.mark.skipif(not is_blackwell(), reason='Requires Blackwell')`。 参数：`M`、`N`、`K`、`BLOCK_K`、`a_format`、`b_format`、`ctas_per_cga`、`multicast`。 关键调用包括 `pytest.mark.parametrize`、`pytest.mark.skipif`、`torch.manual_seed`、`random_quantized_tensor`、`swizzle_scales_packed_block`、`mma_scaled_tcgen05_copy` 等另外 3 项。 该作用域涉及pytest 参数化、PyTorch 张量准备与校验、随机数据生成。

#### Lines 4393-4403

```python
    BLOCK_M = 128 * ctas_per_cga[0]
    BLOCK_N = 128 * ctas_per_cga[1]
    torch.manual_seed(0)
    A, A_scale, A_ref = random_quantized_tensor(M, K, a_format)
    B, B_scale, B_ref = random_quantized_tensor(N, K, b_format)
    VEC_SIZE = 16 if a_format == "nvfp4" else 32
    A_scale = swizzle_scales_packed_block(A_scale, VEC_SIZE)
    B_scale = swizzle_scales_packed_block(B_scale, VEC_SIZE)
    C_ref = A_ref @ B_ref.T
    C = mma_scaled_tcgen05_copy(A, B, A_scale, B_scale, VEC_SIZE, BLOCK_M, BLOCK_N, BLOCK_K, ctas_per_cga, multicast)
    torch.testing.assert_close(C_ref, C.to(torch.float32), atol=1e-3, rtol=1e-3)
```
- **EN:** Prepares or updates state through `BLOCK_M`, `BLOCK_N`, `A`, `A_scale`, `A_ref`, `B`, `B_scale`, `B_ref`, and 3 more. Invokes `torch.manual_seed`, `random_quantized_tensor`, `swizzle_scales_packed_block`, `mma_scaled_tcgen05_copy`, `torch.testing.assert_close`, `C.to` to execute the test logic. Relevant themes: PyTorch tensor setup and checks, random-data generation.
- **CN:** 通过 `BLOCK_M`、`BLOCK_N`、`A`、`A_scale`、`A_ref`、`B`、`B_scale`、`B_ref` 等另外 3 项 准备或更新状态。 调用 `torch.manual_seed`、`random_quantized_tensor`、`swizzle_scales_packed_block`、`mma_scaled_tcgen05_copy`、`torch.testing.assert_close`、`C.to` 执行测试逻辑。 相关主题：PyTorch 张量准备与校验、随机数据生成。

## Key Concepts / 关键概念

- **EN:** Top-level scopes such as `copy_kernel`, `test_copy_kernel`, `test_copy_kernel_multi_cta`, `local_store_transposed_cga_kernel`, `test_local_store_transposed_cga_to_non_transposed_alloc`, `tma_kernel`, `test_tma`, `tma_im2col_kernel`
  **CN:** 顶层作用域，例如 `copy_kernel`、`test_copy_kernel`、`test_copy_kernel_multi_cta`、`local_store_transposed_cga_kernel`、`test_local_store_transposed_cga_to_non_transposed_alloc`、`tma_kernel`、`test_tma`、`tma_im2col_kernel`
- **EN:** pytest parametrization
  **CN:** pytest 参数化
- **EN:** Triton compilation or JIT kernels
  **CN:** Triton 编译或 JIT kernel
- **EN:** Triton language operations
  **CN:** Triton language 操作
- **EN:** PyTorch tensor setup and checks
  **CN:** PyTorch 张量准备与校验
- **EN:** tensor/descriptor metadata
  **CN:** 张量/描述符元数据
- **EN:** matrix multiplication workflows
  **CN:** 矩阵乘法工作流
- **EN:** plugin or compiler extension points
  **CN:** 插件或编译器扩展点

## Dependencies / 依赖关系

- **EN:** External or absolute imports include `torch`, `math`, `pytest`, `re`, `itertools`, `triton`, `triton.language`, `triton._internal_testing`, `triton.compiler`, `triton.tools.mxfp`, `triton.experimental`, `triton.experimental.gluon`, and 9 more.
  **CN:** 外部或绝对导入包括 `torch`、`math`、`pytest`、`re`、`itertools`、`triton`、`triton.language`、`triton._internal_testing`、`triton.compiler`、`triton.tools.mxfp`、`triton.experimental`、`triton.experimental.gluon` 等另外 9 项。
- **EN:** Execution centers on top-level definitions such as `copy_kernel`, `test_copy_kernel`, `test_copy_kernel_multi_cta`, `local_store_transposed_cga_kernel`, `test_local_store_transposed_cga_to_non_transposed_alloc`, `tma_kernel`, `test_tma`, `tma_im2col_kernel`, `test_tma_im2col`, `tma_round_f32_to_tf32_kernel`.
  **CN:** 执行逻辑主要围绕顶层定义展开，例如 `copy_kernel`、`test_copy_kernel`、`test_copy_kernel_multi_cta`、`local_store_transposed_cga_kernel`、`test_local_store_transposed_cga_to_non_transposed_alloc`、`tma_kernel`、`test_tma`、`tma_im2col_kernel`、`test_tma_im2col`、`tma_round_f32_to_tf32_kernel`。
- **EN:** Runtime behavior also depends on pytest collection, Python execution semantics, and the imported Triton/PyTorch utilities visible above.
  **CN:** 运行时行为还依赖 pytest 的收集机制、Python 执行语义，以及上文可见的 Triton/PyTorch 工具。
