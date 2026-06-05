# test_machete_mm.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/kernels/quantization/test_machete_mm.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Pytest module for kernel-level behavior in kernels / quantization / test_machete_mm, focusing on correctness, edge cases, and comparisons against trusted reference paths. / 面向 kernels / quantization / test_machete_mm 的内核级 pytest 模块，重点覆盖正确性、边界情况以及与可信参考路径的对照。

## Line-by-Line Analysis / 逐行分析
### Module docstring (lines 3-6)
```python
"""Tests for the machete kernel.

Run `pytest tests/kernels/quantization/test_machete_mm.py`.
"""
```
**EN:** This docstring states the file-level intent and helps readers understand the role of the module before they inspect the implementation.
**CN:** 这个文档字符串说明了文件级意图，让读者在查看实现前先理解模块职责。

### Imports and shared setup (lines 8-24)
```python
import math
from dataclasses import dataclass, fields

import pytest
import torch

from tests.kernels.utils import opcheck
from vllm import _custom_ops as ops
from vllm.model_executor.layers.quantization.utils.machete_utils import (
    query_machete_supported_group_sizes,
)
from vllm.model_executor.layers.quantization.utils.quant_utils import (
    pack_rows,
    quantize_weights,
)
from vllm.platforms import current_platform
from vllm.scalar_type import ScalarType, scalar_types
```
**EN:** This block imports the modules that the rest of the file relies on. It pulls in external libraries such as math, dataclasses, pytest, torch; shared test helpers from tests.kernels.utils; and vLLM components like vllm, vllm.model_executor.layers.quantization.utils.machete_utils, vllm.model_executor.layers.quantization.utils.quant_utils, vllm.platforms.
**CN:** 该代码块导入后续逻辑依赖的模块。 主要包括外部库，例如 math、dataclasses、pytest、torch；共享测试辅助模块，例如 tests.kernels.utils；vLLM 内部组件，例如 vllm、vllm.model_executor.layers.quantization.utils.machete_utils、vllm.model_executor.layers.quantization.utils.quant_utils、vllm.platforms。

### Top-level block starting at line 26 (lines 26-30)
```python
if current_platform.is_rocm():
    pytest.skip(
        "These tests require machete_prepack_B, not supported on ROCm.",
        allow_module_level=True,
    )
```
**EN:** This top-level `If` block performs supporting work that the surrounding tests depend on.
**CN:** 这个顶层 `If` 代码块执行周边测试所依赖的辅助逻辑。

### Constants and module state (lines 32-53)
```python
CUDA_DEVICES = [
    f"cuda:{i}" for i in range(1 if torch.accelerator.device_count() == 1 else 2)
]

# TODO: in future PR refactor this and `is_quant_method_supported` in the kernel
#  unit tests to a common utility function. Currently the use of
#  `is_quant_method_supported` conflates kernels with quantization methods
#  an assumption which is breaking down as quantizations methods can have
#  have kernels and some kernels support multiple quantization methods.
IS_SUPPORTED_BY_GPU = current_platform.get_device_capability()[0] >= 9

MNK_SHAPES = [
    (1, 128, 128),
    (1, 8192, 28672),
    (13, 8192, 4096),
    (26, 4096, 8192),
    (64, 4096, 4096),
    (64, 8192, 28672),
    (257, 128, 4096),
    (257, 4224, 4160),
    (1024, 8192, 4096),
]
```
**EN:** This block centralizes shared constants and parameter grids, including CUDA_DEVICES, IS_SUPPORTED_BY_GPU, MNK_SHAPES. Those values keep later pytest scenarios consistent and make the test matrix easier to audit.
**CN:** 该代码块集中定义共享常量与参数网格，例如 CUDA_DEVICES、IS_SUPPORTED_BY_GPU、MNK_SHAPES。这些值让后续 pytest 场景保持一致，也便于检查测试矩阵是否完整。

### Class `TypeConfig` (lines 56-64)
```python
@dataclass
class TypeConfig:
    act_type: torch.dtype
    weight_type: ScalarType
    output_type: torch.dtype | None
    group_scale_type: torch.dtype | None
    group_zero_type: torch.dtype | None
    channel_scale_type: torch.dtype | None
    token_scale_type: torch.dtype | None
```
**EN:** This dataclass packages the fields needed to describe TypeConfig. Keeping the inputs structured makes parameter sets explicit and easy to pass between helpers.
**CN:** 这个 dataclass 把描述 TypeConfig 所需的字段组织在一起，使参数集合更明确，也更容易在辅助函数之间传递。

### Class `Tensors` (lines 67-76)
```python
@dataclass
class Tensors:
    w_ref: torch.Tensor
    a_ref: torch.Tensor
    a: torch.Tensor
    w_q: torch.Tensor
    w_g_s: torch.Tensor | None
    w_g_zp: torch.Tensor | None
    w_ch_s: torch.Tensor | None
    w_tok_s: torch.Tensor | None
```
**EN:** This dataclass packages the fields needed to describe Tensors. Keeping the inputs structured makes parameter sets explicit and easy to pass between helpers.
**CN:** 这个 dataclass 把描述 Tensors 所需的字段组织在一起，使参数集合更明确，也更容易在辅助函数之间传递。

### Constants and module state (lines 83-139)
```python
TestTypeTuple = tuple[
    list[torch.dtype], ScalarType, torch.dtype | None, torch.dtype | None, bool
]
TEST_TYPES = [
    # GPTQ style
    *(
        TypeConfig(
            act_type=a_type,
            weight_type=w_type,
            output_type=None,
            group_scale_type=a_type,
            group_zero_type=None,
            channel_scale_type=None,
            token_scale_type=None,
        )
        for w_type in [scalar_types.uint4b8, scalar_types.uint8b128]
        for a_type in [torch.float16, torch.bfloat16]
    ),
    # AWQ style
    *(
        TypeConfig(
            act_type=a_type,
            weight_type=w_type,
            output_type=None,
            group_scale_type=a_type,
            group_zero_type=a_type,
            channel_scale_type=None,
            token_scale_type=None,
        )
        for w_type in [scalar_types.uint4, scalar_types.uint8]
        for a_type in [torch.float16, torch.bfloat16]
    ),
    # # QQQ style
    # *(TypeConfig(act_type=torch.int8,
    #              weight_type=scalar_types.uint4b8,
    #              output_type=torch.float16,
    #              group_scale_type=group_scale_type,
    #              group_zero_type=None,
    #              channel_scale_type=torch.float,
    #              token_scale_type=torch.float)
    #   for group_scale_type in [None, torch.float16]),
    # *(TypeConfig(act_type=torch.float8_e4m3fn,
    #              weight_type=scalar_types.uint4b8,
    #              output_type=torch.float16,
    #              group_scale_type=group_scale_type,
    #              group_zero_type=None,
    #              channel_scale_type=torch.float,
    #              token_scale_type=torch.float)
    #   for group_scale_type in [None, torch.float16]),
]

# TODO: in future PR refactor this and `is_quant_method_supported` in the kernel
#  unit tests to a common utility function. Currently the use of
#  `is_quant_method_supported` conflates kernels with quantization methods
#  an assumption which is breaking down as quantizations methods can have
#  have kernels and some kernels support multiple quantization methods.
IS_SUPPORTED_BY_GPU = current_platform.has_device_capability(90)
```
**EN:** This block centralizes shared constants and parameter grids, including TEST_TYPES, IS_SUPPORTED_BY_GPU. Those values keep later pytest scenarios consistent and make the test matrix easier to audit.
**CN:** 该代码块集中定义共享常量与参数网格，例如 TEST_TYPES、IS_SUPPORTED_BY_GPU。这些值让后续 pytest 场景保持一致，也便于检查测试矩阵是否完整。

### Function `rand_data` (lines 142-146)
```python
def rand_data(shape, dtype=torch.float16, scale=1, offset=0):
    if dtype.is_floating_point:
        return (scale * torch.rand(shape, device="cuda") - offset).to(dtype)
    else:
        return torch.randint(-8, 7, shape, dtype=dtype, device="cuda")
```
**EN:** This helper function implements the shared logic for rand data. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 该辅助函数实现了 rand data 所需的共享逻辑。 它把周边模块中的共用逻辑封装成可复用单元。

### Function `maybe_convert_zeropoints` (lines 149-150)
```python
def maybe_convert_zeropoints(zps: torch.Tensor | None, s: torch.Tensor):
    return zps if zps is None else -1 * s * (zps.to(s.dtype))
```
**EN:** This helper function implements the shared logic for maybe convert zeropoints. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 该辅助函数实现了 maybe convert zeropoints 所需的共享逻辑。 它把周边模块中的共用逻辑封装成可复用单元。

### Function `group_size_valid` (lines 153-154)
```python
def group_size_valid(shape: tuple[int, int, int], group_size: int | None) -> bool:
    return group_size is None or group_size == -1 or shape[2] % group_size == 0
```
**EN:** This helper function implements the shared logic for group size valid. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 该辅助函数实现了 group size valid 所需的共享逻辑。 它把周边模块中的共用逻辑封装成可复用单元。

### Function `machete_quantize_and_pack` (lines 157-182)
```python
def machete_quantize_and_pack(
    atype: torch.dtype,
    w: torch.Tensor,
    wtype: ScalarType,
    stype: torch.dtype | None,
    group_size: int | None,
    zero_points: bool = False,
):
    assert wtype.is_integer(), "TODO: support floating point weights"

    w_ref, w_q, w_s, w_zp = quantize_weights(
        w,
        wtype,
        group_size=group_size,
        zero_points=zero_points,
        # to match how the kernel applies zps
        ref_zero_points_after_scales=True,
    )

    w_q = pack_rows(w_q, wtype.size_bits, *w_q.shape)
    w_q = w_q.t().contiguous().t()  # convert to col major

    w_q_machete = ops.machete_prepack_B(w_q, atype, wtype, stype)
    opcheck(torch.ops._C.machete_prepack_B, (w_q, atype, wtype.id, stype))

    return w_ref, w_q_machete, w_s, w_zp
```
**EN:** This helper function implements the shared logic for machete quantize and pack. it also validates that the custom operator entry point is wired correctly. assertions at the end lock in the intended behavior or graph shape.
**CN:** 该辅助函数实现了 machete quantize and pack 所需的共享逻辑。 它还会校验自定义算子入口是否正确接线；结尾处的断言会固定预期行为或计算图形态。

### Function `create_test_tensors` (lines 185-246)
```python
def create_test_tensors(
    shape: tuple[int, int, int],
    types: TypeConfig,
    group_size: int | None,
    subset_stride_factor: int | None = None,
) -> Tensors:
    m, n, k = shape
    factor = subset_stride_factor or 1

    print(
        "create_test_tensors, shape:", shape, "types:", types, "group_size:", group_size
    )

    a = rand_data((m * factor, k * factor), types.act_type, scale=3, offset=2)
    w = rand_data((k * factor, n * factor), types.act_type, scale=3, offset=1)

    if factor > 1:
        a = a[0:m, 0:k]
        w = w[0:k, 0:n]

    if types.group_scale_type is not None:
        w = w.to(types.group_scale_type)
    if w.dtype.itemsize == 1:
        w = w.to(torch.float16)

    w_ref, w_q_packed, w_s, w_zp = machete_quantize_and_pack(
        a.dtype,
        w,
        types.weight_type,
        types.group_scale_type,
        group_size,
        types.group_zero_type is not None,
    )

    if not a.dtype.is_floating_point:
        aiinfo = torch.iinfo(a.dtype)
        w_ref = w_ref.round().clamp(aiinfo.min, aiinfo.max)

    a_ref = a.to(torch.float32)
    w_ref = w_ref.to(torch.float32)

    w_ch_s = (
        None
        if types.channel_scale_type is None
        else rand_data((n,), types.channel_scale_type)
    )
    w_tok_s = (
        None
        if types.token_scale_type is None
        else rand_data((m,), types.token_scale_type)
    )

    return Tensors(
        w_ref=w_ref,
        a_ref=a_ref,
        a=a,
        w_q=w_q_packed,
        w_g_s=w_s,
        w_g_zp=maybe_convert_zeropoints(w_zp, w_s),
        w_ch_s=w_ch_s,
        w_tok_s=w_tok_s,
    )
```
**EN:** This helper function implements the shared logic for test tensors. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 该辅助函数实现了 test tensors 所需的共享逻辑。 它把周边模块中的共用逻辑封装成可复用单元。

### Function `machete_mm_test_helper` (lines 250-295)
```python
def machete_mm_test_helper(
    types: TypeConfig,
    tensors: Tensors,
    group_size: int | None = None,
    schedule: str | None = None,
):
    output_ref = torch.matmul(tensors.a_ref, tensors.w_ref)
    output_ref_type = output_ref.dtype

    if tensors.w_ch_s is not None:
        output_ref = (
            output_ref.to(tensors.w_ch_s.dtype) * tensors.w_ch_s.unsqueeze(0)
        ).to(output_ref_type)
    if tensors.w_tok_s is not None:
        output_ref = (
            output_ref.to(tensors.w_tok_s.dtype) * tensors.w_tok_s.unsqueeze(1)
        ).to(output_ref_type)

    output = ops.machete_mm(
        a=tensors.a,
        b_q=tensors.w_q,
        b_type=types.weight_type,
        b_group_scales=tensors.w_g_s,
        b_group_zeros=tensors.w_g_zp,
        b_group_size=group_size,
        b_channel_scales=tensors.w_ch_s,
        a_token_scales=tensors.w_tok_s,
        out_type=types.output_type,
        schedule=schedule,
    )

    print(output)
    print(output_ref)

    # Relax atol as our reduction dim becomes larger (more rounding error)
    # Relax atol when we have zeropoints since the way machete applies
    #  zeropoints (after scales) causes noise around 0
    atol = (
        1
        if tensors.w_g_zp is not None
        else min(5e-2 * math.sqrt(tensors.a.shape[1]), 1)
    )
    rtol = 1e-1 if tensors.a.element_size() >= 2 else 2e-1
    torch.testing.assert_close(
        output, output_ref.to(output.dtype), rtol=rtol, atol=atol
    )
```
**EN:** This helper function implements the shared logic for machete mm test helper. numeric results are compared against a reference with explicit tolerances.
**CN:** 该辅助函数实现了 machete mm test helper 所需的共享逻辑。 数值结果会在显式容差下与参考结果进行比较。

### Function `test_machete_all_schedules` (lines 298-324)
```python
@pytest.mark.skipif(
    not IS_SUPPORTED_BY_GPU, reason="Machete is not supported on this GPU type."
)
@pytest.mark.parametrize("shape", MNK_SHAPES, ids=lambda x: "x".join(str(v) for v in x))
@pytest.mark.parametrize("types", TEST_TYPES)
def test_machete_all_schedules(shape, types: TypeConfig):
    group_sizes: list[int | None] = []
    if types.group_scale_type is None:
        group_sizes = [None]
    else:
        group_sizes = query_machete_supported_group_sizes(types.act_type)

    for group_size in group_sizes:
        if not group_size_valid(shape, group_size):
            continue

        tensors = create_test_tensors(shape, types, group_size)
        print(f"MNK = {shape}")
        for schedule in ops.machete_supported_schedules(
            types.act_type,
            types.weight_type,
            group_scales_type=types.group_scale_type,
            group_zeros_type=types.group_scale_type,
            out_type=types.output_type,
        ):
            print(f"Testing schedule {schedule}")
            machete_mm_test_helper(types, tensors, group_size, schedule)
```
**EN:** This pytest case verifies machete all schedules. It is parameterized across 2 input dimensions so one definition covers many scenarios. it consumes fixtures or inputs such as shape, types. unsupported hardware, backend, or configuration combinations are skipped early.
**CN:** 该 pytest 用例验证 machete all schedules 的行为。 它通过 2 组参数化输入覆盖多种场景；它会使用诸如 shape、types 等 fixture 或输入；不支持的硬件、后端或配置组合会被提前跳过。

### Function `test_machete_heuristic` (lines 327-344)
```python
@pytest.mark.skipif(
    not IS_SUPPORTED_BY_GPU, reason="Machete is not supported on this GPU type."
)
@pytest.mark.parametrize("shape", MNK_SHAPES, ids=lambda x: "x".join(str(v) for v in x))
@pytest.mark.parametrize("types", TEST_TYPES)
def test_machete_heuristic(shape, types: TypeConfig):
    group_sizes: list[int | None] = []
    if types.group_scale_type is None:
        group_sizes = [None]
    else:
        group_sizes = query_machete_supported_group_sizes(types.act_type)

    for group_size in group_sizes:
        if not group_size_valid(shape, group_size):
            continue

        tensors = create_test_tensors(shape, types, group_size)
        machete_mm_test_helper(types, tensors, group_size)
```
**EN:** This pytest case verifies machete heuristic. It is parameterized across 2 input dimensions so one definition covers many scenarios. it consumes fixtures or inputs such as shape, types. unsupported hardware, backend, or configuration combinations are skipped early.
**CN:** 该 pytest 用例验证 machete heuristic 的行为。 它通过 2 组参数化输入覆盖多种场景；它会使用诸如 shape、types 等 fixture 或输入；不支持的硬件、后端或配置组合会被提前跳过。

### Function `test_machete_devices` (lines 348-372)
```python
@pytest.mark.skipif(
    not IS_SUPPORTED_BY_GPU, reason="Machete is not supported on this GPU type."
)
@pytest.mark.parametrize("device", CUDA_DEVICES)
def test_machete_devices(device: str):
    group_size = 128

    type_config = TypeConfig(
        act_type=torch.float16,
        weight_type=scalar_types.uint4b8,
        output_type=None,
        group_scale_type=torch.float16,
        group_zero_type=None,
        channel_scale_type=None,
        token_scale_type=None,
    )

    tensors = create_test_tensors((512, 4096, 4096), type_config, group_size)

    for field in fields(Tensors):
        tensor = getattr(tensors, field.name)
        if isinstance(tensor, torch.Tensor):
            setattr(tensors, field.name, tensor.to(device))

    machete_mm_test_helper(type_config, tensors, group_size)
```
**EN:** This pytest case verifies machete devices. It is parameterized across 1 input dimension so one definition covers many scenarios. it consumes fixtures or inputs such as device. unsupported hardware, backend, or configuration combinations are skipped early.
**CN:** 该 pytest 用例验证 machete devices 的行为。 它通过 1 组参数化输入覆盖多种场景；它会使用诸如 device 等 fixture 或输入；不支持的硬件、后端或配置组合会被提前跳过。

### Function `test_machete_subset` (lines 376-395)
```python
@pytest.mark.skipif(
    not IS_SUPPORTED_BY_GPU, reason="Machete is not supported on this GPU type."
)
def test_machete_subset():
    group_size = 128

    type_config = TypeConfig(
        act_type=torch.float16,
        weight_type=scalar_types.uint4b8,
        output_type=None,
        group_scale_type=torch.float16,
        group_zero_type=None,
        channel_scale_type=None,
        token_scale_type=None,
    )

    tensors = create_test_tensors(
        (512, 4096, 4096), type_config, group_size, subset_stride_factor=2
    )
    machete_mm_test_helper(type_config, tensors, group_size)
```
**EN:** This pytest case verifies machete subset. unsupported hardware, backend, or configuration combinations are skipped early.
**CN:** 该 pytest 用例验证 machete subset 的行为。 不支持的硬件、后端或配置组合会被提前跳过。

### Class `MacheteLayer` (lines 399-399)
```python
class MacheteLayer(torch.nn.Module):
```
**EN:** This helper class groups the state and behavior needed for MacheteLayer. That structure makes repeated setup, graph inspection, or reusable assertions easier to manage.
**CN:** 这个辅助类把 MacheteLayer 所需的状态与行为收拢到一起，使重复初始化、计算图检查或可复用断言更容易管理。

### Method `MacheteLayer.__init__` (lines 400-402)
```python
    def __init__(self, **kwargs):
        super().__init__()
        self.kwargs = kwargs
```
**EN:** This method implements the initialization for `MacheteLayer`. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 这个方法实现了 `MacheteLayer` 的初始化逻辑。 它把周边模块中的共用逻辑封装成可复用单元。

### Method `MacheteLayer.forward` (lines 404-405)
```python
    def forward(self, a):
        return ops.machete_mm(a=a, **self.kwargs)
```
**EN:** This method on `MacheteLayer` implements forward. It keeps the surrounding module logic factored into a reusable unit.
**CN:** `MacheteLayer` 中的这个方法实现了 forward。 它把周边模块中的共用逻辑封装成可复用单元。

### Function `test_machete_cuda_graph` (lines 408-449)
```python
@pytest.mark.skipif(
    not IS_SUPPORTED_BY_GPU, reason="Machete is not supported on this GPU type."
)
def test_machete_cuda_graph():
    m, n, k = 512, 4096, 4096

    a = rand_data((m, k), torch.float16)
    b = rand_data((k, n), torch.float16)
    wtype = scalar_types.uint4b8
    stype = torch.float16
    group_size = 128
    zero_points = False

    w_ref, w_q_packed, w_s, w_zp = machete_quantize_and_pack(
        a.dtype, b, wtype, stype, group_size, zero_points
    )

    # Construct a trivial model with a single layer that calls a machete kernel
    model = MacheteLayer(
        b_q=w_q_packed,
        b_type=wtype,
        b_group_scales=w_s,
        b_group_zeros=maybe_convert_zeropoints(w_zp, w_s),
        b_group_size=group_size,
    )

    output_ref = torch.matmul(a, w_ref)

    # Run the model with a cuda graph
    stream = torch.cuda.Stream()
    with torch.cuda.stream(stream):
        g = torch.cuda.CUDAGraph()
        with torch.cuda.graph(g):
            output = model(a)
    output.zero_()
    g.replay()

    # Relax atol as our reduction dim becomes larger (more rounding error)
    # Relax atol when we have zeropoints since the way machete applies
    #  zeropoints (after scales) causes noise around 0
    atol = 1 if zero_points else min(5e-2 * math.sqrt(k), 1)
    torch.testing.assert_close(output, output_ref, rtol=1e-1, atol=atol)
```
**EN:** This pytest case verifies machete CUDA graph. unsupported hardware, backend, or configuration combinations are skipped early. numeric results are compared against a reference with explicit tolerances.
**CN:** 该 pytest 用例验证 machete CUDA graph 的行为。 不支持的硬件、后端或配置组合会被提前跳过；数值结果会在显式容差下与参考结果进行比较。

## Key Concepts / 关键概念
- **Pytest parameterization / pytest 参数化:** The module expands one definition into many concrete scenarios through parametrized inputs. / 该模块通过参数化输入把单个定义扩展成多组具体场景。
- **Behavioral assertions / 行为断言:** Assertions check tensor values, graph structure, or expected error conditions. / 断言用于检查张量结果、计算图结构或预期的报错路径。
- **Platform guards / 平台保护:** Hardware, backend, or version checks prevent unsupported combinations from running. / 硬件、后端或版本检查会阻止不受支持的组合继续执行。
- **Reference comparisons / 参考实现对照:** Optimized kernels are commonly checked against simpler reference implementations or fallback paths. / 优化后的内核通常会与更直接的参考实现或回退路径进行比对。

## Dependencies / 依赖关系
- `math`
- `dataclasses -> dataclass, fields`
- `pytest`
- `torch`
- `tests.kernels.utils -> opcheck`
- `vllm -> _custom_ops`
- `vllm.model_executor.layers.quantization.utils.machete_utils -> query_machete_supported_group_sizes`
- `vllm.model_executor.layers.quantization.utils.quant_utils -> pack_rows, quantize_weights`
- `vllm.platforms -> current_platform`
- `vllm.scalar_type -> ScalarType, scalar_types`
