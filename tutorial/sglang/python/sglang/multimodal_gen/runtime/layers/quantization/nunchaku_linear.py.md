# nunchaku_linear.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/runtime/layers/quantization/nunchaku_linear.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file belongs to the runtime layer implementation. It centers on `NunchakuSVDQLinearMethod`, and `NunchakuAWQLinearMethod`, organizing the main control flow, data structures, or helper routines for this module. / 该文件属于运行时算子层。它围绕 `NunchakuSVDQLinearMethod` 和 `NunchakuAWQLinearMethod` 展开，组织了本模块的主控制流程、数据结构或辅助逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 2-10: module setup and imports / 模块初始化与导入
```python
from typing import List, Optional

import torch
import torch.nn as nn
from torch.nn.parameter import Parameter

from sglang.multimodal_gen.runtime.layers.linear import LinearMethodBase
from sglang.multimodal_gen.runtime.models.utils import set_weight_attrs
from sglang.multimodal_gen.runtime.utils.logging_utils import init_logger
```
**EN:** This block establishes the module context and imports `typing`, `torch`, `torch.nn`, `torch.nn.parameter`, `sglang.multimodal_gen.runtime.layers.linear`, and `sglang.multimodal_gen.runtime.models.utils`. These dependencies provide the symbols needed by the rest of the file.
**CN:** 该代码块建立模块上下文，并导入 `typing`、`torch`、`torch.nn`、`torch.nn.parameter`、`sglang.multimodal_gen.runtime.layers.linear` 和 `sglang.multimodal_gen.runtime.models.utils`。这些依赖为后续实现提供所需符号。

### Lines 12-21: supporting statements / 辅助语句
```python
logger = init_logger(__name__)

try:
    from nunchaku.ops.gemm import svdq_gemm_w4a4_cuda
    from nunchaku.ops.gemv import awq_gemv_w4a16_cuda
    from nunchaku.ops.quantize import svdq_quantize_w4a4_act_fuse_lora_cuda
except ImportError:
    svdq_gemm_w4a4_cuda = None
    awq_gemv_w4a16_cuda = None
    svdq_quantize_w4a4_act_fuse_lora_cuda = None
```
**EN:** This block gathers supporting statements at module scope. It updates names such as `logger`. The code collaborates with `init_logger`.
**CN:** 该代码块汇集了位于模块作用域的辅助语句。 它会更新 `logger` 等名称。 代码会与 `init_logger` 协同工作。

### Lines 24-24: `NunchakuSVDQLinearMethod` class overview / `NunchakuSVDQLinearMethod` 类概览
```python
class NunchakuSVDQLinearMethod(LinearMethodBase):
```
**EN:** This block defines class `NunchakuSVDQLinearMethod`. It encapsulates nunchaku svdqlinear method behavior. It inherits from `LinearMethodBase`.
**CN:** 该代码块定义了类 `NunchakuSVDQLinearMethod`。 它用于封装 nunchaku svdqlinear method 相关行为。 它继承自 `LinearMethodBase`。

### Lines 25-38: `__init__` implementation / `__init__` 实现
```python
    def __init__(
        self,
        precision: str = "int4",
        rank: int = 32,
        act_unsigned: bool = False,
    ):
        self.precision = precision
        self.rank = rank
        self.act_unsigned = act_unsigned

        if precision == "nvfp4":
            self.group_size = 16
        else:
            self.group_size = 64
```
**EN:** This block defines method `__init__` on `NunchakuSVDQLinearMethod`. It initializes the instance state. The implementation branches on conditions. Parameters such as `precision`, `rank`, and `act_unsigned` drive the behavior in this section.
**CN:** 该代码块定义了 `NunchakuSVDQLinearMethod` 的方法 `__init__`。 它用于初始化实例状态。 实现中包含条件分支。 本段逻辑主要由 `precision`、`rank` 和 `act_unsigned` 等参数驱动。

### Lines 40-136: `create_weights` implementation / `create_weights` 实现
```python
    def create_weights(
        self,
        layer: torch.nn.Module,
        input_size_per_partition: int,
        output_partition_sizes: List[int],
        input_size: int,
        output_size: int,
        params_dtype: torch.dtype,
        **extra_weight_attrs,
    ) -> None:
        output_size_per_partition = sum(output_partition_sizes)

        qweight = Parameter(
            torch.empty(
                output_size_per_partition,
                input_size_per_partition // 2,
                dtype=torch.int8,
            ),
            requires_grad=False,
        )
        set_weight_attrs(qweight, {"input_dim": 1, "output_dim": 0})

        num_groups = input_size_per_partition // self.group_size
        if self.precision == "nvfp4":
            scale_dtype = torch.float8_e4m3fn
        else:
            scale_dtype = params_dtype
        wscales = Parameter(
            torch.empty(num_groups, output_size_per_partition, dtype=scale_dtype),
            requires_grad=False,
        )

        smooth_factor = Parameter(
            torch.empty(input_size_per_partition, dtype=params_dtype),
            requires_grad=False,
        )

        smooth_factor_orig = Parameter(
            torch.empty(input_size_per_partition, dtype=params_dtype),
            requires_grad=False,
        )

        proj_down = Parameter(
            torch.empty(input_size_per_partition, self.rank, dtype=params_dtype),
            requires_grad=False,
        )
        proj_up = Parameter(
            torch.empty(output_size_per_partition, self.rank, dtype=params_dtype),
            requires_grad=False,
        )

        if self.precision == "nvfp4":
            wcscales = Parameter(
                torch.empty(
                    output_size_per_partition,
                    dtype=params_dtype,
                ),
                requires_grad=False,
            )
            wtscale = Parameter(
                torch.empty(1, dtype=params_dtype),
                requires_grad=False,
            )
        else:
            wcscales = None
            wtscale = None

        layer.register_parameter("qweight", qweight)
        layer.register_parameter("wscales", wscales)
        layer.register_parameter("smooth_factor", smooth_factor)
        layer.register_parameter("smooth_factor_orig", smooth_factor_orig)
        layer.register_parameter("proj_down", proj_down)
        layer.register_parameter("proj_up", proj_up)
        if wcscales is not None:
            layer.register_parameter("wcscales", wcscales)
        if wtscale is not None:
            layer.register_parameter("wtscale", wtscale)

        layer.input_size_per_partition = input_size_per_partition
        layer.output_size_per_partition = output_size_per_partition
        layer.precision = self.precision
        layer.rank = self.rank
        layer.group_size = self.group_size
        layer.act_unsigned = self.act_unsigned

        weight_loader = extra_weight_attrs.get("weight_loader")
        if weight_loader is not None:
            set_weight_attrs(qweight, {"weight_loader": weight_loader})
            set_weight_attrs(wscales, {"weight_loader": weight_loader})
            set_weight_attrs(smooth_factor, {"weight_loader": weight_loader})
            set_weight_attrs(smooth_factor_orig, {"weight_loader": weight_loader})
            set_weight_attrs(proj_down, {"weight_loader": weight_loader})
            set_weight_attrs(proj_up, {"weight_loader": weight_loader})
            if wcscales is not None:
                set_weight_attrs(wcscales, {"weight_loader": weight_loader})
            if wtscale is not None:
                set_weight_attrs(wtscale, {"weight_loader": weight_loader})
```
**EN:** This block defines method `create_weights` on `NunchakuSVDQLinearMethod`. It creates weights. Key calls include `sum`, `Parameter`, `set_weight_attrs`, `layer.register_parameter`, and `extra_weight_attrs.get`. The implementation branches on conditions. Parameters such as `layer`, `input_size_per_partition`, `output_partition_sizes`, `input_size`, and `output_size` drive the behavior in this section.
**CN:** 该代码块定义了 `NunchakuSVDQLinearMethod` 的方法 `create_weights`。 它用于创建weights。 关键调用包括 `sum`、`Parameter`、`set_weight_attrs`、`layer.register_parameter` 和 `extra_weight_attrs.get`。 实现中包含条件分支。 本段逻辑主要由 `layer`、`input_size_per_partition`、`output_partition_sizes`、`input_size` 和 `output_size` 等参数驱动。

### Lines 138-161: `process_weights_after_loading` implementation / `process_weights_after_loading` 实现
```python
    def process_weights_after_loading(self, layer: nn.Module) -> None:
        layer.qweight = Parameter(layer.qweight.data, requires_grad=False)
        layer.wscales = Parameter(layer.wscales.data, requires_grad=False)
        layer.smooth_factor = Parameter(layer.smooth_factor.data, requires_grad=False)
        layer.smooth_factor_orig = Parameter(
            layer.smooth_factor_orig.data, requires_grad=False
        )
        layer.proj_down = Parameter(layer.proj_down.data, requires_grad=False)
        layer.proj_up = Parameter(layer.proj_up.data, requires_grad=False)
        if hasattr(layer, "wcscales") and layer.wcscales is not None:
            layer.wcscales = Parameter(layer.wcscales.data, requires_grad=False)
        if hasattr(layer, "wtscale") and layer.wtscale is not None:
            layer.wtscale = Parameter(layer.wtscale.data, requires_grad=False)

        alpha: float | None = None
        wtscale = getattr(layer, "wtscale", None)
        if wtscale is not None:
            if isinstance(wtscale, Parameter):
                wtscale = wtscale.data
            if isinstance(wtscale, torch.Tensor):
                alpha = float(wtscale.detach().cpu().item())
            else:
                alpha = float(wtscale)
        layer._nunchaku_alpha = alpha
```
**EN:** This block defines method `process_weights_after_loading` on `NunchakuSVDQLinearMethod`. It processes weights after loading. Key calls include `Parameter`, `getattr`, `hasattr`, `isinstance`, and `float`. The implementation branches on conditions. Parameters such as `layer` drive the behavior in this section.
**CN:** 该代码块定义了 `NunchakuSVDQLinearMethod` 的方法 `process_weights_after_loading`。 它用于处理weights after loading。 关键调用包括 `Parameter`、`getattr`、`hasattr`、`isinstance` 和 `float`。 实现中包含条件分支。 本段逻辑主要由 `layer` 等参数驱动。

### Lines 163-203: `apply` implementation / `apply` 实现
```python
    def apply(
        self,
        layer: torch.nn.Module,
        x: torch.Tensor,
        bias: Optional[torch.Tensor] = None,
    ) -> torch.Tensor:

        orig_shape = x.shape
        x_2d = x.reshape(-1, orig_shape[-1])
        quantized_x, ascales, lora_act_out = svdq_quantize_w4a4_act_fuse_lora_cuda(
            x_2d,
            lora_down=layer.proj_down,
            smooth=layer.smooth_factor,
            fp4=layer.precision == "nvfp4",
            pad_size=256,
        )
        out_2d = torch.empty(
            x_2d.shape[0],
            layer.output_size_per_partition,
            dtype=x_2d.dtype,
            device=x_2d.device,
        )
        alpha: float | None = getattr(layer, "_nunchaku_alpha", None)
        wcscales = getattr(layer, "wcscales", None)

        svdq_gemm_w4a4_cuda(
            act=quantized_x,
            wgt=layer.qweight,
            out=out_2d,
            ascales=ascales,
            wscales=layer.wscales,
            lora_act_in=lora_act_out,
            lora_up=layer.proj_up,
            bias=bias,
            fp4=layer.precision == "nvfp4",
            alpha=alpha,
            wcscales=wcscales,
            act_unsigned=getattr(layer, "act_unsigned", False),
        )
        out = out_2d.reshape(*orig_shape[:-1], layer.output_size_per_partition)
        return out
```
**EN:** This block defines method `apply` on `NunchakuSVDQLinearMethod`. It applies function. Key calls include `x.reshape`, `svdq_quantize_w4a4_act_fuse_lora_cuda`, `torch.empty`, `getattr`, and `svdq_gemm_w4a4_cuda`. Parameters such as `layer`, `x`, and `bias` drive the behavior in this section.
**CN:** 该代码块定义了 `NunchakuSVDQLinearMethod` 的方法 `apply`。 它用于应用函数。 关键调用包括 `x.reshape`、`svdq_quantize_w4a4_act_fuse_lora_cuda`、`torch.empty`、`getattr` 和 `svdq_gemm_w4a4_cuda`。 本段逻辑主要由 `layer`、`x` 和 `bias` 等参数驱动。

### Lines 206-206: `NunchakuAWQLinearMethod` class overview / `NunchakuAWQLinearMethod` 类概览
```python
class NunchakuAWQLinearMethod(LinearMethodBase):
```
**EN:** This block defines class `NunchakuAWQLinearMethod`. It encapsulates nunchaku awqlinear method behavior. It inherits from `LinearMethodBase`.
**CN:** 该代码块定义了类 `NunchakuAWQLinearMethod`。 它用于封装 nunchaku awqlinear method 相关行为。 它继承自 `LinearMethodBase`。

### Lines 207-209: `__init__` implementation / `__init__` 实现
```python
    def __init__(self, group_size: int = 64):
        self.group_size = group_size
        self.pack_factor = 8
```
**EN:** This block defines method `__init__` on `NunchakuAWQLinearMethod`. It initializes the instance state. Parameters such as `group_size` drive the behavior in this section.
**CN:** 该代码块定义了 `NunchakuAWQLinearMethod` 的方法 `__init__`。 它用于初始化实例状态。 本段逻辑主要由 `group_size` 等参数驱动。

### Lines 211-257: `create_weights` implementation / `create_weights` 实现
```python
    def create_weights(
        self,
        layer: torch.nn.Module,
        input_size_per_partition: int,
        output_partition_sizes: List[int],
        input_size: int,
        output_size: int,
        params_dtype: torch.dtype,
        **extra_weight_attrs,
    ) -> None:
        output_size_per_partition = sum(output_partition_sizes)

        qweight = Parameter(
            torch.empty(
                output_size_per_partition // 4,
                input_size_per_partition // 2,
                dtype=torch.int32,
            ),
            requires_grad=False,
        )
        set_weight_attrs(qweight, {"input_dim": 1, "output_dim": 0})

        num_groups = input_size_per_partition // self.group_size
        wscales = Parameter(
            torch.empty(num_groups, output_size_per_partition, dtype=params_dtype),
            requires_grad=False,
        )

        wzeros = Parameter(
            torch.empty(num_groups, output_size_per_partition, dtype=params_dtype),
            requires_grad=False,
        )

        layer.register_parameter("qweight", qweight)
        layer.register_parameter("wscales", wscales)
        layer.register_parameter("wzeros", wzeros)

        layer.input_size_per_partition = input_size_per_partition
        layer.output_size_per_partition = output_size_per_partition
        layer.group_size = self.group_size
        layer.pack_factor = self.pack_factor

        weight_loader = extra_weight_attrs.get("weight_loader")
        if weight_loader is not None:
            set_weight_attrs(qweight, {"weight_loader": weight_loader})
            set_weight_attrs(wscales, {"weight_loader": weight_loader})
            set_weight_attrs(wzeros, {"weight_loader": weight_loader})
```
**EN:** This block defines method `create_weights` on `NunchakuAWQLinearMethod`. It creates weights. Key calls include `sum`, `Parameter`, `set_weight_attrs`, `layer.register_parameter`, and `extra_weight_attrs.get`. The implementation branches on conditions. Parameters such as `layer`, `input_size_per_partition`, `output_partition_sizes`, `input_size`, and `output_size` drive the behavior in this section.
**CN:** 该代码块定义了 `NunchakuAWQLinearMethod` 的方法 `create_weights`。 它用于创建weights。 关键调用包括 `sum`、`Parameter`、`set_weight_attrs`、`layer.register_parameter` 和 `extra_weight_attrs.get`。 实现中包含条件分支。 本段逻辑主要由 `layer`、`input_size_per_partition`、`output_partition_sizes`、`input_size` 和 `output_size` 等参数驱动。

### Lines 259-262: `process_weights_after_loading` implementation / `process_weights_after_loading` 实现
```python
    def process_weights_after_loading(self, layer: nn.Module) -> None:
        layer.qweight = Parameter(layer.qweight.data, requires_grad=False)
        layer.wscales = Parameter(layer.wscales.data, requires_grad=False)
        layer.wzeros = Parameter(layer.wzeros.data, requires_grad=False)
```
**EN:** This block defines method `process_weights_after_loading` on `NunchakuAWQLinearMethod`. It processes weights after loading. Key calls include `Parameter`. Parameters such as `layer` drive the behavior in this section.
**CN:** 该代码块定义了 `NunchakuAWQLinearMethod` 的方法 `process_weights_after_loading`。 它用于处理weights after loading。 关键调用包括 `Parameter`。 本段逻辑主要由 `layer` 等参数驱动。

### Lines 264-291: `apply` implementation / `apply` 实现
```python
    def apply(
        self,
        layer: torch.nn.Module,
        x: torch.Tensor,
        bias: Optional[torch.Tensor] = None,
    ) -> torch.Tensor:

        orig_shape = x.shape
        x_2d = x.reshape(-1, orig_shape[-1])

        in_features = layer.input_size_per_partition
        out_features = layer.output_size_per_partition
        out_2d = awq_gemv_w4a16_cuda(
            in_feats=x_2d,
            kernel=layer.qweight,
            scaling_factors=layer.wscales,
            zeros=layer.wzeros,
            m=x_2d.shape[0],
            n=out_features,
            k=in_features,
            group_size=layer.group_size,
        )
        if bias is not None:
            view_shape = [1] * (out_2d.ndim - 1) + [-1]
            out_2d.add_(bias.view(view_shape))

        out = out_2d.reshape(*orig_shape[:-1], out_features)
        return out
```
**EN:** This block defines method `apply` on `NunchakuAWQLinearMethod`. It applies function. Key calls include `x.reshape`, `awq_gemv_w4a16_cuda`, `out_2d.reshape`, `out_2d.add_`, and `bias.view`. The implementation branches on conditions. Parameters such as `layer`, `x`, and `bias` drive the behavior in this section.
**CN:** 该代码块定义了 `NunchakuAWQLinearMethod` 的方法 `apply`。 它用于应用函数。 关键调用包括 `x.reshape`、`awq_gemv_w4a16_cuda`、`out_2d.reshape`、`out_2d.add_` 和 `bias.view`。 实现中包含条件分支。 本段逻辑主要由 `layer`、`x` 和 `bias` 等参数驱动。

## Key Concepts / 关键概念
- `NunchakuSVDQLinearMethod`: Primary class that encapsulates nunchaku svdqlinear method behavior. / 核心类，用于封装 nunchaku svdqlinear method 相关行为。
- `NunchakuAWQLinearMethod`: Primary class that encapsulates nunchaku awqlinear method behavior. / 核心类，用于封装 nunchaku awqlinear method 相关行为。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `typing`
- **Third-party / 第三方依赖**: `torch`, `torch.nn`, `torch.nn.parameter`, `nunchaku.ops.gemm`, `nunchaku.ops.gemv`, `nunchaku.ops.quantize`
- **Internal modules / 内部模块**: `sglang.multimodal_gen.runtime.layers.linear`, `sglang.multimodal_gen.runtime.models.utils`, `sglang.multimodal_gen.runtime.utils.logging_utils`

- **Total lines / 总行数**: 291
