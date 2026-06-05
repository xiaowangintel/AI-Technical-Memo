# linear.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/runtime/layers/linear.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file belongs to the runtime layer implementation. It centers on `LinearMethodBase`, `UnquantizedLinearMethod`, and `LinearBase`, organizing the main control flow, data structures, or helper routines for this module. / 该文件属于运行时算子层。它围绕 `LinearMethodBase`、`UnquantizedLinearMethod` 和 `LinearBase` 展开，组织了本模块的主控制流程、数据结构或辅助逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 6-40: module setup and imports / 模块初始化与导入
```python
from abc import abstractmethod

import torch
import torch.distributed as dist
import torch.nn.functional as F
from torch.nn.parameter import Parameter

from sglang.kernel_api_logging import wrap_method_with_debug_kernel_once
from sglang.multimodal_gen.runtime.distributed import (
    divide,
    get_tp_group,
    split_tensor_along_last_dim,
    tensor_model_parallel_all_gather,
    tensor_model_parallel_all_reduce,
)
from sglang.multimodal_gen.runtime.layers.quantization.configs.base_config import (
    QuantizationConfig,
    QuantizeMethodBase,
)
from sglang.multimodal_gen.runtime.layers.utils import get_group_rank, get_group_size

# yapf: disable
from sglang.multimodal_gen.runtime.models.parameter import (
    BasevLLMParameter,
    BlockQuantScaleParameter,
    PackedColumnParameter,
    PackedvLLMParameter,
    PerTensorScaleParameter,
    RowvLLMParameter,
)

# yapf: enable
from sglang.multimodal_gen.runtime.models.utils import set_weight_attrs
from sglang.multimodal_gen.runtime.platforms import current_platform
from sglang.multimodal_gen.runtime.utils.logging_utils import init_logger
```
**EN:** This block establishes the module context and imports `abc`, `torch`, `torch.distributed`, `torch.nn.functional`, `torch.nn.parameter`, and `sglang.kernel_api_logging`. These dependencies provide the symbols needed by the rest of the file.
**CN:** 该代码块建立模块上下文，并导入 `abc`、`torch`、`torch.distributed`、`torch.nn.functional`、`torch.nn.parameter` 和 `sglang.kernel_api_logging`。这些依赖为后续实现提供所需符号。

### Lines 42-64: supporting statements / 辅助语句
```python
logger = init_logger(__name__)

IS_AMP_SUPPORTED = current_platform.is_amp_supported()
WEIGHT_LOADER_V2_SUPPORTED = [
    "CompressedTensorsLinearMethod",
    "AWQMarlinLinearMethod",
    "AWQLinearMethod",
    "GPTQMarlinLinearMethod",
    "Fp8LinearMethod",
    "MarlinLinearMethod",
    "QQQLinearMethod",
    "GPTQMarlin24LinearMethod",
    "TPUInt8LinearMethod",
    "GPTQLinearMethod",
    "FBGEMMFp8LinearMethod",
    "ModelOptFp8LinearMethod",
    "ModelOptFp4LinearMethod",
    "ComfyUIFp4LinearMethod",
    "IPEXAWQLinearMethod",
    "IPEXGPTQLinearMethod",
    "HQQMarlinMethod",
    "QuarkLinearMethod",
]
```
**EN:** This block gathers supporting statements at module scope. It updates names such as `logger`, `IS_AMP_SUPPORTED`, and `WEIGHT_LOADER_V2_SUPPORTED`. The code collaborates with `init_logger`, and `current_platform.is_amp_supported`.
**CN:** 该代码块汇集了位于模块作用域的辅助语句。 它会更新 `logger`、`IS_AMP_SUPPORTED` 和 `WEIGHT_LOADER_V2_SUPPORTED` 等名称。 代码会与 `init_logger` 和 `current_platform.is_amp_supported` 协同工作。

### Lines 67-89: `adjust_scalar_to_fused_array` implementation / `adjust_scalar_to_fused_array` 实现
```python
def adjust_scalar_to_fused_array(
    param: torch.Tensor, loaded_weight: torch.Tensor, shard_id: str | int
) -> tuple[torch.Tensor, torch.Tensor]:
    """For fused modules (QKV and MLP) we have an array of length
    N that holds 1 scale for each "logical" matrix. So the param
    is an array of length N. The loaded_weight corresponds to
    one of the shards on disk. Here, we slice the param based on
    the shard_id for loading.
    """
    qkv_idxs = {"q": 0, "k": 1, "v": 2}

    if isinstance(shard_id, str):
        shard_id = qkv_idxs[shard_id]
    elif not isinstance(shard_id, int):
        raise ValueError(f"Unknown Shard Id {shard_id}")

    # AutoFP8 scales do not have a shape
    # compressed-tensors scales do have a shape
    if len(loaded_weight.shape) != 0:
        assert loaded_weight.shape[0] == 1
        loaded_weight = loaded_weight[0]

    return param[shard_id], loaded_weight
```
**EN:** This block defines function `adjust_scalar_to_fused_array`. For fused modules (QKV and MLP) we have an array of length N that holds 1 scale for each "logical" matrix. So the param is an array of length N. Key calls include `isinstance`, `len`, and `ValueError`. The implementation branches on conditions. Parameters such as `param`, `loaded_weight`, and `shard_id` drive the behavior in this section.
**CN:** 该代码块定义了函数 `adjust_scalar_to_fused_array`。 它用于处理 adjust scalar to fused array 相关逻辑。 关键调用包括 `isinstance`、`len` 和 `ValueError`。 实现中包含条件分支。 本段逻辑主要由 `param`、`loaded_weight` 和 `shard_id` 等参数驱动。

### Lines 92-94: `LinearMethodBase` class overview / `LinearMethodBase` 类概览
```python
class LinearMethodBase(QuantizeMethodBase):
    """Base class for different (maybe quantized) linear methods."""
```
**EN:** This block defines class `LinearMethodBase`. Base class for different (maybe quantized) linear methods. It inherits from `QuantizeMethodBase`.
**CN:** 该代码块定义了类 `LinearMethodBase`。 它用于封装 linear method base 相关行为。 它继承自 `QuantizeMethodBase`。

### Lines 95-119: `create_weights` implementation / `create_weights` 实现
```python
    @abstractmethod
    def create_weights(
        self,
        layer: torch.nn.Module,
        input_size_per_partition: int,
        output_partition_sizes: list[int],
        input_size: int,
        output_size: int,
        params_dtype: torch.dtype,
        **extra_weight_attrs,
    ) -> None:
        """Create weights for a linear layer.
           The weights will be set as attributes of the layer.

        Args:
            layer: The layer that is using the LinearMethodBase factory.
            input_size_per_partition: Size of the weight input dim on rank X.
            output_partition_sizes: Sizes of the output dim of each logical
                weight on rank X. E.g., output_partition_sizes for QKVLinear
                is a list contains the width of Wq, Wk, Wv on rank X.
            input_size: Size of the input dim of the weight across all ranks.
            output_size: Size of the output dim of the weight across all ranks.
            params_dtype: Datatype of the parameters.
        """
        raise NotImplementedError
```
**EN:** This block defines method `create_weights` on `LinearMethodBase`. Create weights for a linear layer. The weights will be set as attributes of the layer. Parameters such as `layer`, `input_size_per_partition`, `output_partition_sizes`, `input_size`, and `output_size` drive the behavior in this section.
**CN:** 该代码块定义了 `LinearMethodBase` 的方法 `create_weights`。 它用于创建weights。 本段逻辑主要由 `layer`、`input_size_per_partition`、`output_partition_sizes`、`input_size` 和 `output_size` 等参数驱动。

### Lines 121-127: `apply` implementation / `apply` 实现
```python
    @abstractmethod
    def apply(
        self, layer: torch.nn.Module, x: torch.Tensor, bias: torch.Tensor | None = None
    ) -> torch.Tensor:
        """Apply the weights in layer to the input tensor.
        Expects create_weights to have been called before on the layer."""
        raise NotImplementedError
```
**EN:** This block defines method `apply` on `LinearMethodBase`. Apply the weights in layer to the input tensor. Expects create_weights to have been called before on the layer. Parameters such as `layer`, `x`, and `bias` drive the behavior in this section.
**CN:** 该代码块定义了 `LinearMethodBase` 的方法 `apply`。 它用于应用函数。 本段逻辑主要由 `layer`、`x` 和 `bias` 等参数驱动。

### Lines 130-132: `UnquantizedLinearMethod` class overview / `UnquantizedLinearMethod` 类概览
```python
class UnquantizedLinearMethod(LinearMethodBase):
    """Linear method without quantization."""
```
**EN:** This block defines class `UnquantizedLinearMethod`. Linear method without quantization. It inherits from `LinearMethodBase`.
**CN:** 该代码块定义了类 `UnquantizedLinearMethod`。 它用于封装 unquantized linear method 相关行为。 它继承自 `LinearMethodBase`。

### Lines 133-153: `create_weights` implementation / `create_weights` 实现
```python
    def create_weights(
        self,
        layer: torch.nn.Module,
        input_size_per_partition: int,
        output_partition_sizes: list[int],
        input_size: int,
        output_size: int,
        params_dtype: torch.dtype,
        **extra_weight_attrs,
    ) -> None:
        weight = Parameter(
            torch.empty(
                sum(output_partition_sizes),
                input_size_per_partition,
                dtype=params_dtype,
            ),
            requires_grad=False,
        )
        set_weight_attrs(weight, {"input_dim": 1, "output_dim": 0})
        layer.register_parameter("weight", weight)
        set_weight_attrs(weight, extra_weight_attrs)
```
**EN:** This block defines method `create_weights` on `UnquantizedLinearMethod`. It creates weights. Key calls include `Parameter`, `set_weight_attrs`, `layer.register_parameter`, `torch.empty`, and `sum`. Parameters such as `layer`, `input_size_per_partition`, `output_partition_sizes`, `input_size`, and `output_size` drive the behavior in this section.
**CN:** 该代码块定义了 `UnquantizedLinearMethod` 的方法 `create_weights`。 它用于创建weights。 关键调用包括 `Parameter`、`set_weight_attrs`、`layer.register_parameter`、`torch.empty` 和 `sum`。 本段逻辑主要由 `layer`、`input_size_per_partition`、`output_partition_sizes`、`input_size` 和 `output_size` 等参数驱动。

### Lines 155-163: `apply` implementation / `apply` 实现
```python
    def apply(
        self, layer: torch.nn.Module, x: torch.Tensor, bias: torch.Tensor | None = None
    ) -> torch.Tensor:
        output = (
            F.linear(x, layer.weight, bias)
            if IS_AMP_SUPPORTED or bias is None
            else F.linear(x, layer.weight, bias.to(x.dtype))
        )  # NOTE: explicit dtype cast for bias is needed on platforms where amp isn't supported
        return output
```
**EN:** This block defines method `apply` on `UnquantizedLinearMethod`. It applies function. Key calls include `F.linear`, and `bias.to`. Parameters such as `layer`, `x`, and `bias` drive the behavior in this section.
**CN:** 该代码块定义了 `UnquantizedLinearMethod` 的方法 `apply`。 它用于应用函数。 关键调用包括 `F.linear` 和 `bias.to`。 本段逻辑主要由 `layer`、`x` 和 `bias` 等参数驱动。

### Lines 166-176: `LinearBase` class overview / `LinearBase` 类概览
```python
class LinearBase(torch.nn.Module):
    """Base linear layer.

    Args:
        input_size: input dimension of the linear layer.
        output_size: output dimension of the linear layer.
        skip_bias_add: If true, skip adding bias but instead return it.
        params_dtype: Data type for the parameters.
        quant_config: Quantization configure.
    """
```
**EN:** This block defines class `LinearBase`. Base linear layer. Args: input_size: input dimension of the linear layer. It inherits from `torch.nn.Module`.
**CN:** 该代码块定义了类 `LinearBase`。 它用于封装 linear base 相关行为。 它继承自 `torch.nn.Module`。

### Lines 177-207: `__init__` implementation / `__init__` 实现
```python
    def __init__(
        self,
        input_size: int,
        output_size: int,
        skip_bias_add: bool = False,
        params_dtype: torch.dtype | None = None,
        quant_config: QuantizationConfig | None = None,
        prefix: str = "",
    ):
        super().__init__()

        # Keep input parameters
        self.input_size = input_size
        self.output_size = output_size
        self.skip_bias_add = skip_bias_add
        if params_dtype is None:
            params_dtype = torch.get_default_dtype()
        self.params_dtype = params_dtype
        self.quant_config = quant_config
        self.prefix = prefix
        if quant_config is None:
            self.quant_method: QuantizeMethodBase | None = UnquantizedLinearMethod()
        else:
            self.quant_method = quant_config.get_quant_method(self, prefix=prefix)

        if self.quant_method is not None:
            wrap_method_with_debug_kernel_once(
                self.quant_method,
                "apply",
                op_name=f"diffusion.quant_method.{self.quant_method.__class__.__name__}.apply",
            )
```
**EN:** This block defines method `__init__` on `LinearBase`. It initializes the instance state. Key calls include `super.__init__`, `torch.get_default_dtype`, `UnquantizedLinearMethod`, `quant_config.get_quant_method`, and `wrap_method_with_debug_kernel_once`. The implementation branches on conditions. Parameters such as `input_size`, `output_size`, `skip_bias_add`, `params_dtype`, and `quant_config` drive the behavior in this section.
**CN:** 该代码块定义了 `LinearBase` 的方法 `__init__`。 它用于初始化实例状态。 关键调用包括 `super.__init__`、`torch.get_default_dtype`、`UnquantizedLinearMethod`、`quant_config.get_quant_method` 和 `wrap_method_with_debug_kernel_once`。 实现中包含条件分支。 本段逻辑主要由 `input_size`、`output_size`、`skip_bias_add`、`params_dtype` 和 `quant_config` 等参数驱动。

### Lines 209-210: `forward` implementation / `forward` 实现
```python
    def forward(self, x: torch.Tensor) -> tuple[torch.Tensor, Parameter | None]:
        raise NotImplementedError
```
**EN:** This block defines method `forward` on `LinearBase`. It executes function. Parameters such as `x` drive the behavior in this section.
**CN:** 该代码块定义了 `LinearBase` 的方法 `forward`。 它用于执行前向计算函数。 本段逻辑主要由 `x` 等参数驱动。

### Lines 213-226: `ReplicatedLinear` class overview / `ReplicatedLinear` 类概览
```python
class ReplicatedLinear(LinearBase):
    """Replicated linear layer.

    Args:
        input_size: input dimension of the linear layer.
        output_size: output dimension of the linear layer.
        bias: If true, add bias.
        skip_bias_add: If true, skip adding bias but instead return it.
        params_dtype: Data type for the parameters.
        quant_config: Quantization configure.
        prefix: The name of the layer in the state dict, including all parents
                        (e.g. model.layers.0.qkv_proj)
    """
```
**EN:** This block defines class `ReplicatedLinear`. Replicated linear layer. Args: input_size: input dimension of the linear layer. It inherits from `LinearBase`.
**CN:** 该代码块定义了类 `ReplicatedLinear`。 它用于封装 replicated linear 相关行为。 它继承自 `LinearBase`。

### Lines 227-275: `__init__` implementation / `__init__` 实现
```python
    def __init__(
        self,
        input_size: int,
        output_size: int,
        bias: bool = True,
        skip_bias_add: bool = False,
        params_dtype: torch.dtype | None = None,
        quant_config: QuantizationConfig | None = None,
        output_sizes: list[int] | None = None,
        prefix: str = "",
    ):
        super().__init__(
            input_size,
            output_size,
            skip_bias_add,
            params_dtype,
            quant_config,
            prefix=prefix,
        )

        # All the linear layer supports quant method.
        assert self.quant_method is not None
        output_partition_sizes = output_sizes or [self.output_size]
        self.quant_method.create_weights(
            self,
            self.input_size,
            output_partition_sizes,
            self.input_size,
            self.output_size,
            self.params_dtype,
            weight_loader=self.weight_loader,
        )

        if bias:
            self.bias = Parameter(
                torch.empty(
                    self.output_size,
                    dtype=self.params_dtype,
                )
            )
            set_weight_attrs(
                self.bias,
                {
                    "output_dim": 0,
                    "weight_loader": self.weight_loader,
                },
            )
        else:
            self.register_parameter("bias", None)
```
**EN:** This block defines method `__init__` on `ReplicatedLinear`. It initializes the instance state. Key calls include `super.__init__`, `self.quant_method.create_weights`, `Parameter`, `set_weight_attrs`, and `self.register_parameter`. The implementation branches on conditions. Parameters such as `input_size`, `output_size`, `bias`, `skip_bias_add`, and `params_dtype` drive the behavior in this section.
**CN:** 该代码块定义了 `ReplicatedLinear` 的方法 `__init__`。 它用于初始化实例状态。 关键调用包括 `super.__init__`、`self.quant_method.create_weights`、`Parameter`、`set_weight_attrs` 和 `self.register_parameter`。 实现中包含条件分支。 本段逻辑主要由 `input_size`、`output_size`、`bias`、`skip_bias_add` 和 `params_dtype` 等参数驱动。

### Lines 277-287: `weight_loader` implementation / `weight_loader` 实现
```python
    def weight_loader(self, param: Parameter, loaded_weight: torch.Tensor) -> None:
        # If the weight on disk does not have a shape, give it one
        # (such scales for AutoFp8).
        if len(loaded_weight.shape) == 0:
            loaded_weight = loaded_weight.reshape(1)

        assert param.size() == loaded_weight.size(), (
            f"Tried to load weights of size {loaded_weight.size()}"
            f"to a parameter of size {param.size()}"
        )
        param.data.copy_(loaded_weight)
```
**EN:** This block defines method `weight_loader` on `ReplicatedLinear`. It handles weight loader logic. Key calls include `param.data.copy_`, `len`, `loaded_weight.reshape`, `param.size`, and `loaded_weight.size`. The implementation branches on conditions. Parameters such as `param`, and `loaded_weight` drive the behavior in this section.
**CN:** 该代码块定义了 `ReplicatedLinear` 的方法 `weight_loader`。 它用于处理 weight loader 相关逻辑。 关键调用包括 `param.data.copy_`、`len`、`loaded_weight.reshape`、`param.size` 和 `loaded_weight.size`。 实现中包含条件分支。 本段逻辑主要由 `param` 和 `loaded_weight` 等参数驱动。

### Lines 289-294: `forward` implementation / `forward` 实现
```python
    def forward(self, x: torch.Tensor) -> tuple[torch.Tensor, Parameter | None]:
        bias = self.bias if not self.skip_bias_add else None
        assert self.quant_method is not None
        output = self.quant_method.apply(self, x, bias)
        output_bias = self.bias if self.skip_bias_add else None
        return output, output_bias
```
**EN:** This block defines method `forward` on `ReplicatedLinear`. It executes function. Key calls include `self.quant_method.apply`. Parameters such as `x` drive the behavior in this section.
**CN:** 该代码块定义了 `ReplicatedLinear` 的方法 `forward`。 它用于执行前向计算函数。 关键调用包括 `self.quant_method.apply`。 本段逻辑主要由 `x` 等参数驱动。

### Lines 296-300: `extra_repr` implementation / `extra_repr` 实现
```python
    def extra_repr(self) -> str:
        s = f"in_features={self.input_size}"
        s += f", output_features={self.output_size}"
        s += f", bias={self.bias is not None}"
        return s
```
**EN:** This block defines method `extra_repr` on `ReplicatedLinear`. It handles extra repr logic.
**CN:** 该代码块定义了 `ReplicatedLinear` 的方法 `extra_repr`。 它用于处理 extra repr 相关逻辑。

### Lines 303-326: `ColumnParallelLinear` class overview / `ColumnParallelLinear` 类概览
```python
class ColumnParallelLinear(LinearBase):
    """Linear layer with column parallelism.

    The linear layer is defined as Y = XA + b. A is parallelized along
    its second dimension as A = [A_1, ..., A_p].

    Args:
        input_size: first dimension of matrix A.
        output_size: second dimension of matrix A.
        bias: If true, add bias.
        gather_output: If true, call all-gather on output and make Y available
                       to all GPUs, otherwise, every GPU will have its output
                       which is Y_i = XA_i
        skip_bias_add: This was added to enable performance optimizations where
                       bias can be fused with other element-wise operations. we
                       skip adding bias but instead return it.
        params_dtype: Data type for the parameters.
        quant_config: Quantization configure.
        output_sizes: list of output sizes packed into one output, like for QKV
                       the list would be size 3.
        prefix: The name of the layer in the state dict, including all parents
                        (e.g. model.layers.0.qkv_proj)
    """
```
**EN:** This block defines class `ColumnParallelLinear`. Linear layer with column parallelism. The linear layer is defined as Y = XA + b. It inherits from `LinearBase`.
**CN:** 该代码块定义了类 `ColumnParallelLinear`。 它用于封装 column parallel linear 相关行为。 它继承自 `LinearBase`。

### Lines 327-391: `__init__` implementation / `__init__` 实现
```python
    def __init__(
        self,
        input_size: int,
        output_size: int,
        bias: bool = True,
        gather_output: bool = False,
        skip_bias_add: bool = False,
        params_dtype: torch.dtype | None = None,
        quant_config: QuantizationConfig | None = None,
        output_sizes: list[int] | None = None,
        prefix: str = "",
        tp_group: dist.ProcessGroup = None,
    ):
        # Divide the weight matrix along the last dimension.
        self.tp_group = tp_group or get_tp_group()
        self.tp_size = get_group_size(self.tp_group)
        self.tp_rank = get_group_rank(self.tp_group)
        self.input_size_per_partition = input_size
        self.output_size_per_partition = divide(output_size, self.tp_size)
        self.output_partition_sizes = [self.output_size_per_partition]
        # If QKV or MergedColumn, use output size of each partition.
        if hasattr(self, "output_sizes"):
            self.output_partition_sizes = [
                divide(output_size, self.tp_size) for output_size in self.output_sizes
            ]

        super().__init__(
            input_size, output_size, skip_bias_add, params_dtype, quant_config, prefix
        )

        self.gather_output = gather_output

        if output_sizes is None:
            output_sizes = [output_size]

        assert self.quant_method is not None
        self.quant_method.create_weights(
            layer=self,
            input_size_per_partition=self.input_size_per_partition,
            output_partition_sizes=self.output_partition_sizes,
            input_size=self.input_size,
            output_size=self.output_size,
            params_dtype=self.params_dtype,
            weight_loader=(
                self.weight_loader_v2
                if self.quant_method.__class__.__name__ in WEIGHT_LOADER_V2_SUPPORTED
                else self.weight_loader
            ),
        )
        if bias:
            self.bias = Parameter(
                torch.empty(
                    self.output_size_per_partition,
                    dtype=params_dtype,
                )
            )
            set_weight_attrs(
                self.bias,
                {
                    "output_dim": 0,
                    "weight_loader": self.weight_loader,
                },
            )
        else:
            self.register_parameter("bias", None)
```
**EN:** This block defines method `__init__` on `ColumnParallelLinear`. It initializes the instance state. Key calls include `get_group_size`, `get_group_rank`, `divide`, `hasattr`, and `super.__init__`. The implementation branches on conditions. Parameters such as `input_size`, `output_size`, `bias`, `gather_output`, and `skip_bias_add` drive the behavior in this section.
**CN:** 该代码块定义了 `ColumnParallelLinear` 的方法 `__init__`。 它用于初始化实例状态。 关键调用包括 `get_group_size`、`get_group_rank`、`divide`、`hasattr` 和 `super.__init__`。 实现中包含条件分支。 本段逻辑主要由 `input_size`、`output_size`、`bias`、`gather_output` 和 `skip_bias_add` 等参数驱动。

### Lines 393-412: `weight_loader` implementation / `weight_loader` 实现
```python
    def weight_loader(self, param: Parameter, loaded_weight: torch.Tensor) -> None:
        tp_rank = self.tp_rank
        output_dim = getattr(param, "output_dim", None)

        is_sharded_weight = getattr(param, "is_sharded_weight", False)
        is_sharded_weight = is_sharded_weight

        param_data = param.data
        if output_dim is not None and not is_sharded_weight:
            shard_size = param_data.shape[output_dim]
            start_idx = tp_rank * shard_size
            loaded_weight = loaded_weight.narrow(output_dim, start_idx, shard_size)

        # Special case for loading scales off disk, which often do not
        # have a shape (such as in the case of AutoFP8).
        if len(loaded_weight.shape) == 0:
            loaded_weight = loaded_weight.reshape(1)

        assert param_data.shape == loaded_weight.shape
        param_data.copy_(loaded_weight)
```
**EN:** This block defines method `weight_loader` on `ColumnParallelLinear`. It handles weight loader logic. Key calls include `getattr`, `param_data.copy_`, `loaded_weight.narrow`, `len`, and `loaded_weight.reshape`. The implementation branches on conditions. Parameters such as `param`, and `loaded_weight` drive the behavior in this section.
**CN:** 该代码块定义了 `ColumnParallelLinear` 的方法 `weight_loader`。 它用于处理 weight loader 相关逻辑。 关键调用包括 `getattr`、`param_data.copy_`、`loaded_weight.narrow`、`len` 和 `loaded_weight.reshape`。 实现中包含条件分支。 本段逻辑主要由 `param` 和 `loaded_weight` 等参数驱动。

### Lines 414-420: `weight_loader_v2` implementation / `weight_loader_v2` 实现
```python
    def weight_loader_v2(self, param: Parameter, loaded_weight: torch.Tensor) -> None:
        # Special case for loading scales off disk, which often do not
        # have a shape (such as in the case of AutoFP8).
        if len(loaded_weight.shape) == 0:
            assert loaded_weight.numel() == 1
            loaded_weight = loaded_weight.reshape(1)
        param.load_column_parallel_weight(loaded_weight=loaded_weight)
```
**EN:** This block defines method `weight_loader_v2` on `ColumnParallelLinear`. It handles weight loader v2 logic. Key calls include `param.load_column_parallel_weight`, `len`, `loaded_weight.reshape`, and `loaded_weight.numel`. The implementation branches on conditions. Parameters such as `param`, and `loaded_weight` drive the behavior in this section.
**CN:** 该代码块定义了 `ColumnParallelLinear` 的方法 `weight_loader_v2`。 它用于处理 weight loader v2 相关逻辑。 关键调用包括 `param.load_column_parallel_weight`、`len`、`loaded_weight.reshape` 和 `loaded_weight.numel`。 实现中包含条件分支。 本段逻辑主要由 `param` 和 `loaded_weight` 等参数驱动。

### Lines 422-436: `forward` implementation / `forward` 实现
```python
    def forward(self, input_: torch.Tensor) -> tuple[torch.Tensor, Parameter | None]:
        bias = self.bias if not self.skip_bias_add else None

        # Matrix multiply.
        assert self.quant_method is not None
        output_parallel = self.quant_method.apply(self, input_, bias)
        if self.gather_output:
            # All-gather across the partitions.
            output = tensor_model_parallel_all_gather(
                output_parallel, tp_group=self.tp_group
            )
        else:
            output = output_parallel
        output_bias = self.bias if self.skip_bias_add else None
        return output, output_bias
```
**EN:** This block defines method `forward` on `ColumnParallelLinear`. It executes function. Key calls include `self.quant_method.apply`, and `tensor_model_parallel_all_gather`. The implementation branches on conditions. Parameters such as `input_` drive the behavior in this section.
**CN:** 该代码块定义了 `ColumnParallelLinear` 的方法 `forward`。 它用于执行前向计算函数。 关键调用包括 `self.quant_method.apply` 和 `tensor_model_parallel_all_gather`。 实现中包含条件分支。 本段逻辑主要由 `input_` 等参数驱动。

### Lines 438-444: `extra_repr` implementation / `extra_repr` 实现
```python
    def extra_repr(self) -> str:
        s = f"in_features={self.input_size}"
        s += f", output_features={self.output_size_per_partition}"
        s += f", bias={self.bias is not None}"
        s += f", tp_size={self.tp_size}"
        s += f", gather_output={self.gather_output}"
        return s
```
**EN:** This block defines method `extra_repr` on `ColumnParallelLinear`. It handles extra repr logic.
**CN:** 该代码块定义了 `ColumnParallelLinear` 的方法 `extra_repr`。 它用于处理 extra repr 相关逻辑。

### Lines 447-469: `MergedColumnParallelLinear` class overview / `MergedColumnParallelLinear` 类概览
```python
class MergedColumnParallelLinear(ColumnParallelLinear):
    """Packed linear layers with column parallelism.

    Similar to ColumnParallelLinear, but the weight matrix is concatenated
    along the output dimension. When the weight matrix is loaded, the
    different partitions are sharded separately.

    Args:
        input_size: input dimension of the linear layer.
        output_sizes: list of output dimensions of the linear layer.
        bias: If true, add bias.
        gather_output: If true, call all-gather on output and make the output
                       available to all GPUs, otherwise, every GPU will have
                       its own output.
        skip_bias_add: This was added to enable performance optimizations where
                       bias can be fused with other element-wise operations. we
                       skip adding bias but instead return it.
        params_dtype: Data type for the parameters.
        quant_config: Quantization configure.
        prefix: The name of the layer in the state dict, including all parents
                        (e.g. model.layers.0.qkv_proj)
    """
```
**EN:** This block defines class `MergedColumnParallelLinear`. Packed linear layers with column parallelism. Similar to ColumnParallelLinear, but the weight matrix is concatenated along the output dimension. It inherits from `ColumnParallelLinear`.
**CN:** 该代码块定义了类 `MergedColumnParallelLinear`。 它用于封装 merged column parallel linear 相关行为。 它继承自 `ColumnParallelLinear`。

### Lines 470-494: `__init__` implementation / `__init__` 实现
```python
    def __init__(
        self,
        input_size: int,
        output_sizes: list[int],
        bias: bool = True,
        gather_output: bool = False,
        skip_bias_add: bool = False,
        params_dtype: torch.dtype | None = None,
        quant_config: QuantizationConfig | None = None,
        prefix: str = "",
        tp_group: dist.ProcessGroup = None,
    ):
        super().__init__(
            input_size=input_size,
            output_size=sum(output_sizes),
            bias=bias,
            gather_output=gather_output,
            skip_bias_add=skip_bias_add,
            params_dtype=params_dtype,
            quant_config=quant_config,
            prefix=prefix,
            tp_group=tp_group,
        )
        self.output_sizes = output_sizes
        assert all(output_size % self.tp_size == 0 for output_size in output_sizes)
```
**EN:** This block defines method `__init__` on `MergedColumnParallelLinear`. It initializes the instance state. Key calls include `super.__init__`, `all`, `super`, and `sum`. Parameters such as `input_size`, `output_sizes`, `bias`, `gather_output`, and `skip_bias_add` drive the behavior in this section.
**CN:** 该代码块定义了 `MergedColumnParallelLinear` 的方法 `__init__`。 它用于初始化实例状态。 关键调用包括 `super.__init__`、`all`、`super` 和 `sum`。 本段逻辑主要由 `input_size`、`output_sizes`、`bias`、`gather_output` 和 `skip_bias_add` 等参数驱动。

### Lines 496-572: `weight_loader` implementation / `weight_loader` 实现
```python
    def weight_loader(
        self,
        param: Parameter,
        loaded_weight: torch.Tensor,
        loaded_shard_id: int | None = None,
    ) -> None:
        param_data = param.data
        output_dim = getattr(param, "output_dim", None)
        # Special case for AQLM codebooks.
        is_metadata = getattr(param, "is_metadata", False)
        # Special case for per-tensor scale to load scalar into fused array.
        needs_scalar_to_array = getattr(param, "needs_scalar_to_array", False)

        if loaded_shard_id is None:
            # Loaded weight is already fused on disk (mlp).
            # (e.g., Phi-3's gate_up_proj).
            if output_dim is None:
                if needs_scalar_to_array:
                    param_data, loaded_weight = adjust_scalar_to_fused_array(
                        param_data, loaded_weight, 0
                    )

                assert param_data.shape == loaded_weight.shape
                param_data.copy_(loaded_weight)
                return
            current_shard_offset = 0
            shard_offsets: list[tuple[int, int, int]] = []
            for i, output_size in enumerate(self.output_sizes):
                shard_offsets.append((i, current_shard_offset, output_size))
                current_shard_offset += output_size
            for shard_id, shard_offset, shard_size in shard_offsets:
                loaded_weight_shard = loaded_weight.narrow(
                    output_dim, shard_offset, shard_size
                )
                self.weight_loader(param, loaded_weight_shard, shard_id)
            return

        assert loaded_shard_id < len(self.output_sizes)
        tp_rank = self.tp_rank
        tp_size = self.tp_size
        if output_dim is not None:
            shard_offset = sum(self.output_sizes[:loaded_shard_id]) // tp_size
            shard_size = self.output_sizes[loaded_shard_id] // tp_size

            is_sharded_weight = getattr(param, "is_sharded_weight", False)
            # bitsandbytes loads the weights of the specific portion
            # no need to narrow
            is_sharded_weight = is_sharded_weight

            param_data = param_data.narrow(output_dim, shard_offset, shard_size)
            start_idx = tp_rank * shard_size
            if not is_sharded_weight:
                loaded_weight = loaded_weight.narrow(output_dim, start_idx, shard_size)
        # Special case for AQLM codebooks.
        elif is_metadata:
            # metadata indicates fixed size concatenated along dim 0
            shard_size = loaded_weight.shape[0]
            shard_offset = loaded_shard_id * shard_size
            param_data = param_data.narrow(0, shard_offset, shard_size)

        # Special case for per-tensor scales in fused case.
        elif needs_scalar_to_array:
            param_data, loaded_weight = adjust_scalar_to_fused_array(
                param_data, loaded_weight, loaded_shard_id
            )

        else:
            ignore_warning = getattr(param, "ignore_warning", False)
            if not ignore_warning:
                logger.warning(
                    "Loading a weight without `output_dim` attribute in "
                    "MergedColumnParallelLinear, assume the weight is "
                    "the same for all partitions."
                )

        assert param_data.shape == loaded_weight.shape
        param_data.copy_(loaded_weight)
```
**EN:** This block defines method `weight_loader` on `MergedColumnParallelLinear`. It handles weight loader logic. Key calls include `getattr`, `param_data.copy_`, `enumerate`, `len`, and `param_data.narrow`. The implementation branches on conditions, iterates over collections or steps. Parameters such as `param`, `loaded_weight`, and `loaded_shard_id` drive the behavior in this section.
**CN:** 该代码块定义了 `MergedColumnParallelLinear` 的方法 `weight_loader`。 它用于处理 weight loader 相关逻辑。 关键调用包括 `getattr`、`param_data.copy_`、`enumerate`、`len` 和 `param_data.narrow`。 实现中包含条件分支，会遍历集合或步骤。 本段逻辑主要由 `param`、`loaded_weight` 和 `loaded_shard_id` 等参数驱动。

### Lines 574-608: `_load_fused_module_from_checkpoint` implementation / `_load_fused_module_from_checkpoint` 实现
```python
    def _load_fused_module_from_checkpoint(
        self, param: BasevLLMParameter, loaded_weight: torch.Tensor
    ) -> None:
        """
        Handle special case for models where MLP layers are already
        fused on disk. In this case, we have no shard id. This function
        determmines the shard id by splitting these layers and then calls
        the weight loader using the shard id.

        An example of a model with these fused layers:
        https://huggingface.co/microsoft/Phi-3-mini-4k-instruct
        """

        current_shard_offset = 0
        shard_offsets: list[tuple[int, int, int]] = []
        for i, output_size in enumerate(self.output_sizes):
            shard_offsets.append((i, current_shard_offset, output_size))
            current_shard_offset += output_size

        for shard_id, shard_offset, shard_size in shard_offsets:
            # Special case for Quantization.
            # If quantized, we need to adjust the offset and size to account
            # for the packing.
            if (
                isinstance(param, PackedColumnParameter | PackedvLLMParameter)
                and param.packed_dim == param.output_dim
            ):
                shard_size, shard_offset = param.adjust_shard_indexes_for_packing(
                    shard_size=shard_size, shard_offset=shard_offset
                )

            loaded_weight_shard = loaded_weight.narrow(
                param.output_dim, shard_offset, shard_size
            )
            self.weight_loader_v2(param, loaded_weight_shard, shard_id)
```
**EN:** This block defines method `_load_fused_module_from_checkpoint` on `MergedColumnParallelLinear`. Handle special case for models where MLP layers are already fused on disk. In this case, we have no shard id. Key calls include `enumerate`, `shard_offsets.append`, `loaded_weight.narrow`, `self.weight_loader_v2`, and `isinstance`. The implementation branches on conditions, iterates over collections or steps. Parameters such as `param`, and `loaded_weight` drive the behavior in this section.
**CN:** 该代码块定义了 `MergedColumnParallelLinear` 的方法 `_load_fused_module_from_checkpoint`。 它用于加载fused module from checkpoint。 关键调用包括 `enumerate`、`shard_offsets.append`、`loaded_weight.narrow`、`self.weight_loader_v2` 和 `isinstance`。 实现中包含条件分支，会遍历集合或步骤。 本段逻辑主要由 `param` 和 `loaded_weight` 等参数驱动。

### Lines 610-645: `weight_loader_v2` implementation / `weight_loader_v2` 实现
```python
    def weight_loader_v2(
        self,
        param: BasevLLMParameter,
        loaded_weight: torch.Tensor,
        loaded_shard_id: int | None = None,
    ) -> None:
        if isinstance(param, BlockQuantScaleParameter):
            self._weight_loader_v2_block_quant_scale(
                param, loaded_weight, loaded_shard_id
            )
            return

        if loaded_shard_id is None:
            if isinstance(param, PerTensorScaleParameter):
                param.load_merged_column_weight(loaded_weight=loaded_weight, shard_id=0)
                return
            elif type(param) in (RowvLLMParameter, BasevLLMParameter):
                param.load_merged_column_weight(loaded_weight=loaded_weight)
                return
            # TODO: @dsikka - move to parameter.py
            self._load_fused_module_from_checkpoint(param, loaded_weight)
            return

        assert loaded_shard_id < len(self.output_sizes)

        tp_size = self.tp_size

        shard_offset = sum(self.output_sizes[:loaded_shard_id]) // tp_size
        shard_size = self.output_sizes[loaded_shard_id] // tp_size

        param.load_merged_column_weight(
            loaded_weight=loaded_weight,
            shard_id=loaded_shard_id,
            shard_offset=shard_offset,
            shard_size=shard_size,
        )
```
**EN:** This block defines method `weight_loader_v2` on `MergedColumnParallelLinear`. It handles weight loader v2 logic. Key calls include `isinstance`, `param.load_merged_column_weight`, `self._weight_loader_v2_block_quant_scale`, `self._load_fused_module_from_checkpoint`, and `len`. The implementation branches on conditions. Parameters such as `param`, `loaded_weight`, and `loaded_shard_id` drive the behavior in this section.
**CN:** 该代码块定义了 `MergedColumnParallelLinear` 的方法 `weight_loader_v2`。 它用于处理 weight loader v2 相关逻辑。 关键调用包括 `isinstance`、`param.load_merged_column_weight`、`self._weight_loader_v2_block_quant_scale`、`self._load_fused_module_from_checkpoint` 和 `len`。 实现中包含条件分支。 本段逻辑主要由 `param`、`loaded_weight` 和 `loaded_shard_id` 等参数驱动。

### Lines 647-692: `_weight_loader_v2_block_quant_scale` implementation / `_weight_loader_v2_block_quant_scale` 实现
```python
    def _weight_loader_v2_block_quant_scale(
        self,
        param: BlockQuantScaleParameter,
        loaded_weight: torch.Tensor,
        loaded_shard_id: int | None = None,
    ) -> None:
        assert self.quant_method is not None
        weight_block_size = getattr(
            self.quant_method.quant_config, "weight_block_size", None
        )
        if weight_block_size is None:
            raise ValueError(
                "MergedColumnParallelLinear block-scale loading requires "
                "quant_config.weight_block_size."
            )
        block_n, _ = weight_block_size
        output_dim = param.output_dim

        if loaded_shard_id is None:
            if param.data.shape == loaded_weight.shape:
                param.data.copy_(loaded_weight)
                return

            block_offset = 0
            for shard_id, output_size in enumerate(self.output_sizes):
                block_size = divide(output_size, block_n)
                loaded_weight_shard = loaded_weight.narrow(
                    output_dim, block_offset, block_size
                )
                self._weight_loader_v2_block_quant_scale(
                    param, loaded_weight_shard, shard_id
                )
                block_offset += block_size
            return

        assert loaded_shard_id < len(self.output_sizes)
        shard_offset = divide(sum(self.output_sizes[:loaded_shard_id]), self.tp_size)
        shard_size = divide(self.output_sizes[loaded_shard_id], self.tp_size)
        block_shard_offset = divide(shard_offset, block_n)
        block_shard_size = divide(shard_size, block_n)

        param_data = param.data.narrow(output_dim, block_shard_offset, block_shard_size)
        start_idx = self.tp_rank * block_shard_size
        loaded_weight = loaded_weight.narrow(output_dim, start_idx, block_shard_size)
        assert param_data.shape == loaded_weight.shape
        param_data.copy_(loaded_weight)
```
**EN:** This block defines method `_weight_loader_v2_block_quant_scale` on `MergedColumnParallelLinear`. It handles weight loader v2 block quant scale logic. Key calls include `getattr`, `divide`, `param.data.narrow`, `loaded_weight.narrow`, and `param_data.copy_`. The implementation branches on conditions, iterates over collections or steps. Parameters such as `param`, `loaded_weight`, and `loaded_shard_id` drive the behavior in this section.
**CN:** 该代码块定义了 `MergedColumnParallelLinear` 的方法 `_weight_loader_v2_block_quant_scale`。 它用于处理 weight loader v2 block quant scale 相关逻辑。 关键调用包括 `getattr`、`divide`、`param.data.narrow`、`loaded_weight.narrow` 和 `param_data.copy_`。 实现中包含条件分支，会遍历集合或步骤。 本段逻辑主要由 `param`、`loaded_weight` 和 `loaded_shard_id` 等参数驱动。

### Lines 695-720: `QKVParallelLinear` class overview / `QKVParallelLinear` 类概览
```python
class QKVParallelLinear(ColumnParallelLinear):
    """Linear layers for the attention's QKV transformation.

    Linear layers for the linear transformation of the query, key, and value
    vectors in the attention layer. The weight matrix is concatenated along
    the output dimension. The layer is parallelized along the head dimension.
    When the number of key/value heads is smaller than the number of query
    heads (e.g., multi-query/grouped-query attention), the key/value head may
    be replicated while the query heads are partitioned.

    Args:
        hidden_size: input hidden state size of the transformer.
        head_size: size of each attention head.
        total_num_heads: total number of attention query heads.
        total_num_kv_heads: total number of attention key/value heads. If
                            None, assume total_num_kv_heads = total_num_heads.
        bias: If true, add bias.
        skip_bias_add: This was added to enable performance optimizations where
                       bias can be fused with other element-wise operations. we
                       skip adding bias but instead return it.
        params_dtype: Data type for the parameters.
        quant_config: Quantization configure.
        prefix: The name of the layer in the state dict, including all parents
                        (e.g. model.layers.0.qkv_proj)
    """
```
**EN:** This block defines class `QKVParallelLinear`. Linear layers for the attention's QKV transformation. Linear layers for the linear transformation of the query, key, and value vectors in the attention layer. It inherits from `ColumnParallelLinear`.
**CN:** 该代码块定义了类 `QKVParallelLinear`。 它用于封装 qkvparallel linear 相关行为。 它继承自 `ColumnParallelLinear`。

### Lines 721-770: `__init__` implementation / `__init__` 实现
```python
    def __init__(
        self,
        hidden_size: int,
        head_size: int,
        total_num_heads: int,
        total_num_kv_heads: int | None = None,
        bias: bool = True,
        skip_bias_add: bool = False,
        params_dtype: torch.dtype | None = None,
        quant_config: QuantizationConfig | None = None,
        prefix: str = "",
        tp_group: dist.ProcessGroup = None,
    ):
        self.hidden_size = hidden_size
        self.head_size = head_size
        self.total_num_heads = total_num_heads
        if total_num_kv_heads is None:
            total_num_kv_heads = total_num_heads
        self.total_num_kv_heads = total_num_kv_heads
        # Divide the weight matrix along the last dimension.
        tp_group = tp_group or get_tp_group()
        tp_size = get_group_size(tp_group)
        self.num_heads = divide(self.total_num_heads, tp_size)
        if tp_size >= self.total_num_kv_heads:
            self.num_kv_heads = 1
            self.num_kv_head_replicas = divide(tp_size, self.total_num_kv_heads)
        else:
            self.num_kv_heads = divide(self.total_num_kv_heads, tp_size)
            self.num_kv_head_replicas = 1
        input_size = self.hidden_size
        output_size = (
            (self.num_heads + 2 * self.num_kv_heads) * tp_size * self.head_size
        )
        self.output_sizes = [
            self.num_heads * self.head_size * tp_size,  # q_proj
            self.num_kv_heads * self.head_size * tp_size,  # k_proj
            self.num_kv_heads * self.head_size * tp_size,  # v_proj
        ]

        super().__init__(
            input_size=input_size,
            output_size=output_size,
            bias=bias,
            gather_output=False,
            skip_bias_add=skip_bias_add,
            params_dtype=params_dtype,
            quant_config=quant_config,
            prefix=prefix,
            tp_group=tp_group,
        )
```
**EN:** This block defines method `__init__` on `QKVParallelLinear`. It initializes the instance state. Key calls include `get_group_size`, `divide`, `super.__init__`, `get_tp_group`, and `super`. The implementation branches on conditions. Parameters such as `hidden_size`, `head_size`, `total_num_heads`, `total_num_kv_heads`, and `bias` drive the behavior in this section.
**CN:** 该代码块定义了 `QKVParallelLinear` 的方法 `__init__`。 它用于初始化实例状态。 关键调用包括 `get_group_size`、`divide`、`super.__init__`、`get_tp_group` 和 `super`。 实现中包含条件分支。 本段逻辑主要由 `hidden_size`、`head_size`、`total_num_heads`、`total_num_kv_heads` 和 `bias` 等参数驱动。

### Lines 772-779: `_get_shard_offset_mapping` implementation / `_get_shard_offset_mapping` 实现
```python
    def _get_shard_offset_mapping(self, loaded_shard_id: str) -> int | None:
        shard_offset_mapping = {
            "q": 0,
            "k": self.num_heads * self.head_size,
            "v": (self.num_heads + self.num_kv_heads) * self.head_size,
            "total": (self.num_heads + 2 * self.num_kv_heads) * self.head_size,
        }
        return shard_offset_mapping.get(loaded_shard_id)
```
**EN:** This block defines method `_get_shard_offset_mapping` on `QKVParallelLinear`. It retrieves shard offset mapping. Key calls include `shard_offset_mapping.get`. Parameters such as `loaded_shard_id` drive the behavior in this section.
**CN:** 该代码块定义了 `QKVParallelLinear` 的方法 `_get_shard_offset_mapping`。 它用于获取shard offset mapping。 关键调用包括 `shard_offset_mapping.get`。 本段逻辑主要由 `loaded_shard_id` 等参数驱动。

### Lines 781-787: `_get_shard_size_mapping` implementation / `_get_shard_size_mapping` 实现
```python
    def _get_shard_size_mapping(self, loaded_shard_id: str) -> int | None:
        shard_size_mapping = {
            "q": self.num_heads * self.head_size,
            "k": self.num_kv_heads * self.head_size,
            "v": self.num_kv_heads * self.head_size,
        }
        return shard_size_mapping.get(loaded_shard_id)
```
**EN:** This block defines method `_get_shard_size_mapping` on `QKVParallelLinear`. It retrieves shard size mapping. Key calls include `shard_size_mapping.get`. Parameters such as `loaded_shard_id` drive the behavior in this section.
**CN:** 该代码块定义了 `QKVParallelLinear` 的方法 `_get_shard_size_mapping`。 它用于获取shard size mapping。 关键调用包括 `shard_size_mapping.get`。 本段逻辑主要由 `loaded_shard_id` 等参数驱动。

### Lines 789-831: `_load_fused_module_from_checkpoint` implementation / `_load_fused_module_from_checkpoint` 实现
```python
    def _load_fused_module_from_checkpoint(
        self, param: BasevLLMParameter, loaded_weight: torch.Tensor
    ):
        """
        Handle special case for models where QKV layers are already
        fused on disk. In this case, we have no shard id. This function
        determmines the shard id by splitting these layers and then calls
        the weight loader using the shard id.

        An example of a model with these fused layers:
        https://huggingface.co/microsoft/Phi-3-mini-4k-instruct
        """
        shard_offsets = [
            # (shard_id, shard_offset, shard_size)
            ("q", 0, self.total_num_heads * self.head_size),
            (
                "k",
                self.total_num_heads * self.head_size,
                self.total_num_kv_heads * self.head_size,
            ),
            (
                "v",
                (self.total_num_heads + self.total_num_kv_heads) * self.head_size,
                self.total_num_kv_heads * self.head_size,
            ),
        ]

        for shard_id, shard_offset, shard_size in shard_offsets:
            # Special case for Quantization.
            # If quantized, we need to adjust the offset and size to account
            # for the packing.
            if (
                isinstance(param, PackedColumnParameter | PackedvLLMParameter)
                and param.packed_dim == param.output_dim
            ):
                shard_size, shard_offset = param.adjust_shard_indexes_for_packing(
                    shard_size=shard_size, shard_offset=shard_offset
                )

            loaded_weight_shard = loaded_weight.narrow(
                param.output_dim, shard_offset, shard_size
            )
            self.weight_loader_v2(param, loaded_weight_shard, shard_id)
```
**EN:** This block defines method `_load_fused_module_from_checkpoint` on `QKVParallelLinear`. Handle special case for models where QKV layers are already fused on disk. In this case, we have no shard id. Key calls include `loaded_weight.narrow`, `self.weight_loader_v2`, `isinstance`, and `param.adjust_shard_indexes_for_packing`. The implementation branches on conditions, iterates over collections or steps. Parameters such as `param`, and `loaded_weight` drive the behavior in this section.
**CN:** 该代码块定义了 `QKVParallelLinear` 的方法 `_load_fused_module_from_checkpoint`。 它用于加载fused module from checkpoint。 关键调用包括 `loaded_weight.narrow`、`self.weight_loader_v2`、`isinstance` 和 `param.adjust_shard_indexes_for_packing`。 实现中包含条件分支，会遍历集合或步骤。 本段逻辑主要由 `param` 和 `loaded_weight` 等参数驱动。

### Lines 833-861: `weight_loader_v2` implementation / `weight_loader_v2` 实现
```python
    def weight_loader_v2(
        self,
        param: BasevLLMParameter,
        loaded_weight: torch.Tensor,
        loaded_shard_id: str | None = None,
    ):
        if loaded_shard_id is None:  # special case for certain models
            if isinstance(param, PerTensorScaleParameter):
                param.load_qkv_weight(loaded_weight=loaded_weight, shard_id=0)
                return
            elif type(param) in (RowvLLMParameter, BasevLLMParameter):
                param.load_qkv_weight(loaded_weight=loaded_weight)
                return
            # TODO: @dsikka - move to parameter.py
            self._load_fused_module_from_checkpoint(param, loaded_weight)
            return

        assert loaded_shard_id in ["q", "k", "v"]

        shard_offset = self._get_shard_offset_mapping(loaded_shard_id)
        shard_size = self._get_shard_size_mapping(loaded_shard_id)

        param.load_qkv_weight(
            loaded_weight=loaded_weight,
            num_heads=self.num_kv_head_replicas,
            shard_id=loaded_shard_id,
            shard_offset=shard_offset,
            shard_size=shard_size,
        )
```
**EN:** This block defines method `weight_loader_v2` on `QKVParallelLinear`. It handles weight loader v2 logic. Key calls include `self._get_shard_offset_mapping`, `self._get_shard_size_mapping`, `param.load_qkv_weight`, `isinstance`, and `self._load_fused_module_from_checkpoint`. The implementation branches on conditions. Parameters such as `param`, `loaded_weight`, and `loaded_shard_id` drive the behavior in this section.
**CN:** 该代码块定义了 `QKVParallelLinear` 的方法 `weight_loader_v2`。 它用于处理 weight loader v2 相关逻辑。 关键调用包括 `self._get_shard_offset_mapping`、`self._get_shard_size_mapping`、`param.load_qkv_weight`、`isinstance` 和 `self._load_fused_module_from_checkpoint`。 实现中包含条件分支。 本段逻辑主要由 `param`、`loaded_weight` 和 `loaded_shard_id` 等参数驱动。

### Lines 863-963: `weight_loader` implementation / `weight_loader` 实现
```python
    def weight_loader(
        self,
        param: Parameter,
        loaded_weight: torch.Tensor,
        loaded_shard_id: str | None = None,
    ):
        param_data = param.data
        output_dim = getattr(param, "output_dim", None)
        # Special case for AQLM codebooks.
        is_metadata = getattr(param, "is_metadata", False)

        # Special case for per-tensor scales in fused case.
        needs_scalar_to_array = getattr(param, "needs_scalar_to_array", False)

        if loaded_shard_id is None:
            # Loaded weight is already fused on disk (qkv).
            # (e.g., Phi-3's qkv_proj).
            if output_dim is None:
                if needs_scalar_to_array:
                    param_data, loaded_weight = adjust_scalar_to_fused_array(
                        param_data, loaded_weight, 0
                    )

                assert param_data.shape == loaded_weight.shape
                param_data.copy_(loaded_weight)
                return
            shard_offsets = [
                # (shard_id, shard_offset, shard_size)
                ("q", 0, self.total_num_heads * self.head_size),
                (
                    "k",
                    self.total_num_heads * self.head_size,
                    self.total_num_kv_heads * self.head_size,
                ),
                (
                    "v",
                    (self.total_num_heads + self.total_num_kv_heads) * self.head_size,
                    self.total_num_kv_heads * self.head_size,
                ),
            ]

            for shard_id, shard_offset, shard_size in shard_offsets:
                loaded_weight_shard = loaded_weight.narrow(
                    output_dim, shard_offset, shard_size
                )
                self.weight_loader(param, loaded_weight_shard, shard_id)
            return

        tp_rank = self.tp_rank
        assert loaded_shard_id in ["q", "k", "v"]

        # If output dim is defined, use the default loading process.
        if output_dim is not None:
            if loaded_shard_id == "q":
                shard_offset = 0
                shard_size = self.num_heads * self.head_size
            elif loaded_shard_id == "k":
                shard_offset = self.num_heads * self.head_size
                shard_size = self.num_kv_heads * self.head_size
            elif loaded_shard_id == "v":
                shard_offset = (self.num_heads + self.num_kv_heads) * self.head_size
                shard_size = self.num_kv_heads * self.head_size

            is_sharded_weight = getattr(param, "is_sharded_weight", False)
            # bitsandbytes loads the weights of the specific portion
            # no need to narrow
            is_sharded_weight = is_sharded_weight

            shard_idx = 0
            param_data = param_data.narrow(output_dim, shard_offset, shard_size)
            if loaded_shard_id == "q":
                shard_idx = tp_rank
            else:
                shard_idx = tp_rank // self.num_kv_head_replicas
            start_idx = shard_idx * shard_size

            if not is_sharded_weight:
                loaded_weight = loaded_weight.narrow(output_dim, start_idx, shard_size)

        # Special case for for AQLM codebooks.
        elif is_metadata:
            # metadata indicates fixed size concatenated along dim 0
            shard_size = loaded_weight.shape[0]
            shard_index = ["q", "k", "v"].index(loaded_shard_id)
            param_data = param_data.narrow(0, shard_index * shard_size, shard_size)
        # Special case for per-tensor scales in fused case.
        elif needs_scalar_to_array:
            param_data, loaded_weight = adjust_scalar_to_fused_array(
                param_data, loaded_weight, loaded_shard_id
            )
        else:
            ignore_warning = getattr(param, "ignore_warning", False)
            if not ignore_warning:
                logger.warning(
                    "Loading a weight without `output_dim` attribute in "
                    "QKVParallelLinear, assume the weight is the same "
                    "for all partitions."
                )

        assert param_data.shape == loaded_weight.shape
        param_data.copy_(loaded_weight)
```
**EN:** This block defines method `weight_loader` on `QKVParallelLinear`. It handles weight loader logic. Key calls include `getattr`, `param_data.copy_`, `param_data.narrow`, `loaded_weight.narrow`, and `self.weight_loader`. The implementation branches on conditions, iterates over collections or steps. Parameters such as `param`, `loaded_weight`, and `loaded_shard_id` drive the behavior in this section.
**CN:** 该代码块定义了 `QKVParallelLinear` 的方法 `weight_loader`。 它用于处理 weight loader 相关逻辑。 关键调用包括 `getattr`、`param_data.copy_`、`param_data.narrow`、`loaded_weight.narrow` 和 `self.weight_loader`。 实现中包含条件分支，会遍历集合或步骤。 本段逻辑主要由 `param`、`loaded_weight` 和 `loaded_shard_id` 等参数驱动。

### Lines 966-991: `RowParallelLinear` class overview / `RowParallelLinear` 类概览
```python
class RowParallelLinear(LinearBase):
    """Linear layer with row parallelism.

    The linear layer is defined as Y = XA + b. A is parallelized along
    its first dimension and X along its second dimension as:
               -   -
              | A_1 |
              | .   |
          A = | .   |        X = [X_1, ..., X_p]
              | .   |
              | A_p |
               -   -
    Arguments:
        input_size: first dimension of matrix A.
        output_size: second dimension of matrix A.
        bias: If true, add bias. Note that bias is not parallelized.
        input_is_parallel: If true, we assume that the input is already
                           split across the GPUs and we do not split
                           again.
        skip_bias_add: This was added to enable performance optimization where
                       bias can be fused with other element-wise operations.
                       We skip adding bias but instead return it.
        params_dtype: Data type for the parameters.
        quant_config: Quantization configure.
    """
```
**EN:** This block defines class `RowParallelLinear`. Linear layer with row parallelism. The linear layer is defined as Y = XA + b. It inherits from `LinearBase`.
**CN:** 该代码块定义了类 `RowParallelLinear`。 它用于封装 row parallel linear 相关行为。 它继承自 `LinearBase`。

### Lines 992-1050: `__init__` implementation / `__init__` 实现
```python
    def __init__(
        self,
        input_size: int,
        output_size: int,
        bias: bool = True,
        input_is_parallel: bool = True,
        skip_bias_add: bool = False,
        params_dtype: torch.dtype | None = None,
        reduce_results: bool = True,
        quant_config: QuantizationConfig | None = None,
        prefix: str = "",
        tp_group: dist.ProcessGroup = None,
    ):
        # Divide the weight matrix along the first dimension.
        self.tp_group = tp_group or get_tp_group()
        self.tp_rank = get_group_rank(self.tp_group)
        self.tp_size = get_group_size(self.tp_group)
        self.input_size_per_partition = divide(input_size, self.tp_size)
        self.output_size_per_partition = output_size
        self.output_partition_sizes = [output_size]

        super().__init__(
            input_size, output_size, skip_bias_add, params_dtype, quant_config, prefix
        )

        self.input_is_parallel = input_is_parallel
        self.reduce_results = reduce_results

        assert self.quant_method is not None
        self.quant_method.create_weights(
            layer=self,
            input_size_per_partition=self.input_size_per_partition,
            output_partition_sizes=self.output_partition_sizes,
            input_size=self.input_size,
            output_size=self.output_size,
            params_dtype=self.params_dtype,
            weight_loader=(
                self.weight_loader_v2
                if self.quant_method.__class__.__name__ in WEIGHT_LOADER_V2_SUPPORTED
                else self.weight_loader
            ),
        )
        if not reduce_results and (bias and not skip_bias_add):
            raise ValueError(
                "When not reduce the results, adding bias to the "
                "results can lead to incorrect results"
            )

        if bias:
            self.bias = Parameter(torch.empty(self.output_size, dtype=params_dtype))
            set_weight_attrs(
                self.bias,
                {
                    "output_dim": 0,
                    "weight_loader": self.weight_loader,
                },
            )
        else:
            self.register_parameter("bias", None)
```
**EN:** This block defines method `__init__` on `RowParallelLinear`. It initializes the instance state. Key calls include `get_group_rank`, `get_group_size`, `divide`, `super.__init__`, and `self.quant_method.create_weights`. The implementation branches on conditions. Parameters such as `input_size`, `output_size`, `bias`, `input_is_parallel`, and `skip_bias_add` drive the behavior in this section.
**CN:** 该代码块定义了 `RowParallelLinear` 的方法 `__init__`。 它用于初始化实例状态。 关键调用包括 `get_group_rank`、`get_group_size`、`divide`、`super.__init__` 和 `self.quant_method.create_weights`。 实现中包含条件分支。 本段逻辑主要由 `input_size`、`output_size`、`bias`、`input_is_parallel` 和 `skip_bias_add` 等参数驱动。

### Lines 1052-1072: `weight_loader` implementation / `weight_loader` 实现
```python
    def weight_loader(self, param: Parameter, loaded_weight: torch.Tensor):
        tp_rank = self.tp_rank
        input_dim = getattr(param, "input_dim", None)
        is_sharded_weight = getattr(param, "is_sharded_weight", False)
        # bitsandbytes loads the weights of the specific portion
        # no need to narrow
        is_sharded_weight = is_sharded_weight

        param_data = param.data
        if input_dim is not None and not is_sharded_weight:
            shard_size = param_data.shape[input_dim]
            start_idx = tp_rank * shard_size
            loaded_weight = loaded_weight.narrow(input_dim, start_idx, shard_size)

        # Special case for loading scales off disk, which often do not
        # have a shape (such as in the case of AutoFP8).
        if len(loaded_weight.shape) == 0:
            loaded_weight = loaded_weight.reshape(1)

        assert param_data.shape == loaded_weight.shape
        param_data.copy_(loaded_weight)
```
**EN:** This block defines method `weight_loader` on `RowParallelLinear`. It handles weight loader logic. Key calls include `getattr`, `param_data.copy_`, `loaded_weight.narrow`, `len`, and `loaded_weight.reshape`. The implementation branches on conditions. Parameters such as `param`, and `loaded_weight` drive the behavior in this section.
**CN:** 该代码块定义了 `RowParallelLinear` 的方法 `weight_loader`。 它用于处理 weight loader 相关逻辑。 关键调用包括 `getattr`、`param_data.copy_`、`loaded_weight.narrow`、`len` 和 `loaded_weight.reshape`。 实现中包含条件分支。 本段逻辑主要由 `param` 和 `loaded_weight` 等参数驱动。

### Lines 1074-1081: `weight_loader_v2` implementation / `weight_loader_v2` 实现
```python
    def weight_loader_v2(self, param: BasevLLMParameter, loaded_weight: torch.Tensor):
        # Special case for loading scales off disk, which often do not
        # have a shape (such as in the case of AutoFP8).
        if len(loaded_weight.shape) == 0:
            assert loaded_weight.numel() == 1
            loaded_weight = loaded_weight.reshape(1)

        param.load_row_parallel_weight(loaded_weight=loaded_weight)
```
**EN:** This block defines method `weight_loader_v2` on `RowParallelLinear`. It handles weight loader v2 logic. Key calls include `param.load_row_parallel_weight`, `len`, `loaded_weight.reshape`, and `loaded_weight.numel`. The implementation branches on conditions. Parameters such as `param`, and `loaded_weight` drive the behavior in this section.
**CN:** 该代码块定义了 `RowParallelLinear` 的方法 `weight_loader_v2`。 它用于处理 weight loader v2 相关逻辑。 关键调用包括 `param.load_row_parallel_weight`、`len`、`loaded_weight.reshape` 和 `loaded_weight.numel`。 实现中包含条件分支。 本段逻辑主要由 `param` 和 `loaded_weight` 等参数驱动。

### Lines 1083-1108: `forward` implementation / `forward` 实现
```python
    def forward(self, input_) -> tuple[torch.Tensor, Parameter | None]:
        if self.input_is_parallel:
            input_parallel = input_
        else:
            tp_rank = self.tp_rank
            splitted_input = split_tensor_along_last_dim(
                input_, num_partitions=self.tp_size
            )
            input_parallel = splitted_input[tp_rank].contiguous()

        # Matrix multiply.
        assert self.quant_method is not None
        # Only fuse bias add into GEMM for rank 0 (this ensures that
        # bias will not get added more than once in TP>1 case)
        bias_ = None if (self.tp_rank > 0 or self.skip_bias_add) else self.bias
        output_parallel = self.quant_method.apply(self, input_parallel, bias=bias_)
        if self.reduce_results and self.tp_size > 1:
            output = tensor_model_parallel_all_reduce(
                output_parallel, tp_group=self.tp_group
            )
        else:
            output = output_parallel

        output_bias = self.bias if self.skip_bias_add else None

        return output, output_bias
```
**EN:** This block defines method `forward` on `RowParallelLinear`. It executes function. Key calls include `self.quant_method.apply`, `split_tensor_along_last_dim`, `splitted_input.contiguous`, and `tensor_model_parallel_all_reduce`. The implementation branches on conditions. Parameters such as `input_` drive the behavior in this section.
**CN:** 该代码块定义了 `RowParallelLinear` 的方法 `forward`。 它用于执行前向计算函数。 关键调用包括 `self.quant_method.apply`、`split_tensor_along_last_dim`、`splitted_input.contiguous` 和 `tensor_model_parallel_all_reduce`。 实现中包含条件分支。 本段逻辑主要由 `input_` 等参数驱动。

### Lines 1110-1116: `extra_repr` implementation / `extra_repr` 实现
```python
    def extra_repr(self) -> str:
        s = f"input_features={self.input_size_per_partition}"
        s += f", output_features={self.output_size}"
        s += f", bias={self.bias is not None}"
        s += f", tp_size={self.tp_size}"
        s += f", reduce_results={self.reduce_results}"
        return s
```
**EN:** This block defines method `extra_repr` on `RowParallelLinear`. It handles extra repr logic.
**CN:** 该代码块定义了 `RowParallelLinear` 的方法 `extra_repr`。 它用于处理 extra repr 相关逻辑。

## Key Concepts / 关键概念
- `adjust_scalar_to_fused_array`: For fused modules (QKV and MLP) we have an array of length N that holds 1 scale for each "logical" matrix. / 顶层函数，用于处理 adjust scalar to fused array 相关逻辑。
- `LinearMethodBase`: Base class for different (maybe quantized) linear methods. / 核心类，用于封装 linear method base 相关行为。
- `UnquantizedLinearMethod`: Linear method without quantization. / 核心类，用于封装 unquantized linear method 相关行为。
- `LinearBase`: Base linear layer. / 核心类，用于封装 linear base 相关行为。
- `ReplicatedLinear`: Replicated linear layer. / 核心类，用于封装 replicated linear 相关行为。
- `ColumnParallelLinear`: Linear layer with column parallelism. / 核心类，用于封装 column parallel linear 相关行为。
- `MergedColumnParallelLinear`: Packed linear layers with column parallelism. / 核心类，用于封装 merged column parallel linear 相关行为。
- `QKVParallelLinear`: Linear layers for the attention's QKV transformation. / 核心类，用于封装 qkvparallel linear 相关行为。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `abc`
- **Third-party / 第三方依赖**: `torch`, `torch.distributed`, `torch.nn.functional`, `torch.nn.parameter`
- **Internal modules / 内部模块**: `sglang.kernel_api_logging`, `sglang.multimodal_gen.runtime.distributed`, `sglang.multimodal_gen.runtime.layers.quantization.configs.base_config`, `sglang.multimodal_gen.runtime.layers.utils`, `sglang.multimodal_gen.runtime.models.parameter`, `sglang.multimodal_gen.runtime.models.utils`, `sglang.multimodal_gen.runtime.platforms`, `sglang.multimodal_gen.runtime.utils.logging_utils`

- **Total lines / 总行数**: 1116
