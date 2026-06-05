# bitsandbytes.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/layers/quantization/bitsandbytes.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module defines configuration objects and method-selection logic for bitsandbytes quantization in SGLang SRT. / 该模块定义了用于 SGLang SRT 的 bitsandbytes 量化配置对象与方法选择逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-20: module imports and setup
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project
# Adapted from: https://github.com/vllm-project/vllm/blob/d4d2751732c3ccae162a5a0160c7d4fe05d2779a/vllm/model_executor/layers/quantization/bitsandbytes.py
from __future__ import annotations

from typing import TYPE_CHECKING, Any, Optional

import torch
from packaging import version

from sglang.srt.layers.linear import LinearBase
from sglang.srt.layers.quantization.base_config import (
    FusedMoEMethodBase,
    LinearMethodBase,
    QuantizationConfig,
    QuantizeMethodBase,
)
from sglang.srt.layers.quantization.unquant import UnquantizedLinearMethod
from sglang.srt.utils import set_weight_attrs
from sglang.srt.utils.custom_op import register_custom_op
```
**EN:** This block imports __future__, bitsandbytes, bitsandbytes.functional, bitsandbytes.nn, sglang.srt.layers.linear, sglang.srt.layers.moe.fused_moe_triton.layer, sglang.srt.layers.moe.moe_runner.triton_utils.fused_moe, sglang.srt.layers.moe.token_dispatcher and prepares the module namespace for the quantization implementation.
**CN:** 该代码块导入 __future__, bitsandbytes, bitsandbytes.functional, bitsandbytes.nn, sglang.srt.layers.linear, sglang.srt.layers.moe.fused_moe_triton.layer, sglang.srt.layers.moe.moe_runner.triton_utils.fused_moe, sglang.srt.layers.moe.token_dispatcher 等依赖，并为当前量化实现准备模块命名空间。

### Lines 22-26: typing-only imports
```python
if TYPE_CHECKING:
    from sglang.srt.layers.moe.token_dispatcher import (
        CombineInput,
        StandardDispatchOutput,
    )
```
**EN:** This conditional block imports symbols that are only needed for static typing and do not affect runtime behavior.
**CN:** 该条件代码块只导入静态类型检查需要的符号，不影响运行时行为。

### Lines 29-34: class BitsAndBytesConfig: definition
```python
class BitsAndBytesConfig(QuantizationConfig):
    """Config class for BitsAndBytes Quantization.

    Reference: https://arxiv.org/abs/2305.14314
    """
```
**EN:** This block declares `BitsAndBytesConfig`, a configuration class for the quantization stack. It organizes behaviors such as __init__, __repr__, get_name, get_scaled_act_names.
**CN:** 该代码块声明 `BitsAndBytesConfig`，它是量化栈中的配置类，组织了 __init__, __repr__, get_name, get_scaled_act_names 等行为。

### Lines 35-63: BitsAndBytesConfig.__init__()
```python
    def __init__(
        self,
        load_in_8bit: bool = False,
        load_in_4bit: bool = True,
        bnb_4bit_compute_dtype: str = "float32",
        bnb_4bit_quant_storage: str = "uint8",
        bnb_4bit_quant_type: str = "fp4",
        bnb_4bit_use_double_quant: bool = False,
        llm_int8_enable_fp32_cpu_offload: bool = False,
        llm_int8_has_fp16_weight: bool = False,
        llm_int8_skip_modules: list[str] | None = None,
        llm_int8_threshold: float = 6.0,
    ) -> None:
        super().__init__()
        self.load_in_8bit = load_in_8bit
        self.load_in_4bit = load_in_4bit
        self.bnb_4bit_compute_dtype = bnb_4bit_compute_dtype
        self.bnb_4bit_quant_storage = bnb_4bit_quant_storage
        self.bnb_4bit_quant_type = bnb_4bit_quant_type
        self.bnb_4bit_use_double_quant = bnb_4bit_use_double_quant
        self.llm_int8_enable_fp32_cpu_offload = llm_int8_enable_fp32_cpu_offload
        self.llm_int8_has_fp16_weight = llm_int8_has_fp16_weight
        self.llm_int8_skip_modules = llm_int8_skip_modules or []
        self.llm_int8_threshold = llm_int8_threshold

        if self.bnb_4bit_quant_storage not in ["uint8"]:
            raise ValueError(
                f"Unsupported bnb_4bit_quant_storage: {self.bnb_4bit_quant_storage}"
            )
```
**EN:** This block defines `BitsAndBytesConfig.__init__()`, which initializes object state and validates constructor inputs.
**CN:** 该代码块定义了 `BitsAndBytesConfig.__init__()`，用于初始化对象状态并校验构造参数。

### Lines 65-73: BitsAndBytesConfig.__repr__()
```python
    def __repr__(self) -> str:
        return (
            f"BitsAndBytesConfig(load_in_8bit={self.load_in_8bit}, "
            f"load_in_4bit={self.load_in_4bit}, "
            f"bnb_4bit_compute_dtype={self.bnb_4bit_compute_dtype}, "
            f"bnb_4bit_quant_storage={self.bnb_4bit_quant_storage}, "
            f"bnb_4bit_quant_type={self.bnb_4bit_quant_type}, "
            f"llm_int8_skip_modules={self.llm_int8_skip_modules})"
        )
```
**EN:** This block defines `BitsAndBytesConfig.__repr__()`, which builds a readable debug representation.
**CN:** 该代码块定义了 `BitsAndBytesConfig.__repr__()`，用于构造可读的调试表示。

### Lines 75-76: BitsAndBytesConfig.get_name()
```python
    def get_name(self) -> str:
        return "bitsandbytes"
```
**EN:** This block defines `BitsAndBytesConfig.get_name()`, which retrieves or computes a derived value for callers.
**CN:** 该代码块定义了 `BitsAndBytesConfig.get_name()`，用于为调用方获取或计算派生值。

### Lines 78-79: BitsAndBytesConfig.get_scaled_act_names()
```python
    def get_scaled_act_names(self) -> list[str]:
        return []
```
**EN:** This block defines `BitsAndBytesConfig.get_scaled_act_names()`, which retrieves or computes a derived value for callers.
**CN:** 该代码块定义了 `BitsAndBytesConfig.get_scaled_act_names()`，用于为调用方获取或计算派生值。

### Lines 81-82: BitsAndBytesConfig.get_supported_act_dtypes()
```python
    def get_supported_act_dtypes(self) -> list[torch.dtype]:
        return [torch.float32, torch.float16, torch.bfloat16]
```
**EN:** This block defines `BitsAndBytesConfig.get_supported_act_dtypes()`, which retrieves or computes a derived value for callers.
**CN:** 该代码块定义了 `BitsAndBytesConfig.get_supported_act_dtypes()`，用于为调用方获取或计算派生值。

### Lines 84-86: BitsAndBytesConfig.get_min_capability()
```python
    @classmethod
    def get_min_capability(cls) -> int:
        return 70
```
**EN:** This block defines `BitsAndBytesConfig.get_min_capability()`, which retrieves or computes a derived value for callers.
**CN:** 该代码块定义了 `BitsAndBytesConfig.get_min_capability()`，用于为调用方获取或计算派生值。

### Lines 88-90: BitsAndBytesConfig.get_config_filenames()
```python
    @staticmethod
    def get_config_filenames() -> list[str]:
        return []
```
**EN:** This block defines `BitsAndBytesConfig.get_config_filenames()`, which retrieves or computes a derived value for callers.
**CN:** 该代码块定义了 `BitsAndBytesConfig.get_config_filenames()`，用于为调用方获取或计算派生值。

### Lines 92-139: BitsAndBytesConfig.from_config()
```python
    @classmethod
    def from_config(cls, config: dict[str, Any]) -> "BitsAndBytesConfig":
        def get_safe_value(config, keys, default_value=None):
            try:
                value = QuantizationConfig.get_from_keys(config, keys)
                return value if value is not None else default_value
            except ValueError:
                return default_value

        load_in_8bit = get_safe_value(config, ["load_in_8bit"], default_value=False)
        load_in_4bit = get_safe_value(config, ["load_in_4bit"], default_value=True)
        bnb_4bit_compute_dtype = get_safe_value(
            config, ["bnb_4bit_compute_dtype"], default_value="float32"
        )
        bnb_4bit_quant_storage = get_safe_value(
            config, ["bnb_4bit_quant_storage"], default_value="uint8"
        )
        bnb_4bit_quant_type = get_safe_value(
            config, ["bnb_4bit_quant_type"], default_value="fp4"
        )
        bnb_4bit_use_double_quant = get_safe_value(
            config, ["bnb_4bit_use_double_quant"], default_value=False
        )
        llm_int8_enable_fp32_cpu_offload = get_safe_value(
            config, ["llm_int8_enable_fp32_cpu_offload"], default_value=False
        )
        llm_int8_has_fp16_weight = get_safe_value(
            config, ["llm_int8_has_fp16_weight"], default_value=False
        )
        llm_int8_skip_modules = get_safe_value(
            config, ["llm_int8_skip_modules"], default_value=[]
        )
        llm_int8_threshold = get_safe_value(
            config, ["llm_int8_threshold"], default_value=6.0
        )

        return cls(
            load_in_8bit=load_in_8bit,
            load_in_4bit=load_in_4bit,
            bnb_4bit_compute_dtype=bnb_4bit_compute_dtype,
            bnb_4bit_quant_storage=bnb_4bit_quant_storage,
            bnb_4bit_quant_type=bnb_4bit_quant_type,
            bnb_4bit_use_double_quant=bnb_4bit_use_double_quant,
            llm_int8_enable_fp32_cpu_offload=llm_int8_enable_fp32_cpu_offload,
            llm_int8_has_fp16_weight=llm_int8_has_fp16_weight,
            llm_int8_skip_modules=llm_int8_skip_modules,
            llm_int8_threshold=llm_int8_threshold,
        )
```
**EN:** This block defines `BitsAndBytesConfig.from_config()`, which constructs an object from serialized configuration data.
**CN:** 该代码块定义了 `BitsAndBytesConfig.from_config()`，用于从序列化配置数据构造对象。

### Lines 141-152: BitsAndBytesConfig.get_quant_method()
```python
    def get_quant_method(
        self, layer: torch.nn.Module, prefix: str
    ) -> Optional[QuantizeMethodBase]:
        from sglang.srt.layers.moe.fused_moe_triton.layer import FusedMoE

        if isinstance(layer, LinearBase):
            if is_layer_skipped_bnb(prefix, self.llm_int8_skip_modules):
                return UnquantizedLinearMethod()
            return BitsAndBytesLinearMethod(self)
        elif isinstance(layer, FusedMoE):
            return BitsAndBytesMoEMethod(self)
        return None
```
**EN:** This block defines `BitsAndBytesConfig.get_quant_method()`, which retrieves or computes a derived value for callers.
**CN:** 该代码块定义了 `BitsAndBytesConfig.get_quant_method()`，用于为调用方获取或计算派生值。

### Lines 155-169: is_layer_skipped_bnb()
```python
def is_layer_skipped_bnb(prefix: str, llm_int8_skip_modules: list[str]):
    # Split the prefix into its dot-separated components
    components = prefix.split(".")

    # Check if any of the skip modules exactly matches any component
    substr_check = any(
        module_name in components for module_name in llm_int8_skip_modules
    )

    # Allow certain layers to not be quantized
    set_components = set(".".join(components[: i + 1]) for i in range(len(components)))
    set_llm_int8_skip_modules = set(llm_int8_skip_modules)
    prefix_check = len(set_llm_int8_skip_modules & set_components) != 0

    return substr_check or prefix_check
```
**EN:** This block defines `is_layer_skipped_bnb()`, which checks a condition used by higher-level control flow.
**CN:** 该代码块定义了 `is_layer_skipped_bnb()`，用于检查供上层控制流使用的条件。

### Lines 172-176: calculate_quant_ratio()
```python
def calculate_quant_ratio(dtype):
    if dtype.is_floating_point:
        return torch.finfo(dtype).bits // torch.iinfo(torch.uint8).bits
    else:
        return torch.iinfo(dtype).bits // torch.iinfo(torch.uint8).bits
```
**EN:** This block defines `calculate_quant_ratio()`, which handles tensor or weight quantization work.
**CN:** 该代码块定义了 `calculate_quant_ratio()`，用于处理张量或权重量化逻辑。

### Lines 179-185: class BitsAndBytesLinearMethod: definition
```python
class BitsAndBytesLinearMethod(LinearMethodBase):
    """Linear method for BitsAndBytes.

    Args:
       quant_config: The BitsAndBytes quantization config.
    """
```
**EN:** This block declares `BitsAndBytesLinearMethod`, a runtime method class for the quantization stack. It organizes behaviors such as __init__, create_weights, apply, _apply_8bit_weight.
**CN:** 该代码块声明 `BitsAndBytesLinearMethod`，它是量化栈中的运行方法类，组织了 __init__, create_weights, apply, _apply_8bit_weight 等行为。

### Lines 186-202: BitsAndBytesLinearMethod.__init__()
```python
    def __init__(self, quant_config: BitsAndBytesConfig):
        try:
            import bitsandbytes

            if version.parse(bitsandbytes.__version__) < version.parse("0.46.1"):
                raise ImportError(
                    "bitsandbytes version is wrong. Please "
                    "install bitsandbytes>=0.46.1."
                )
        except ImportError as err:
            raise ImportError(
                "Please install bitsandbytes>=0.46.1 via "
                "`pip install bitsandbytes>=0.46.1` to use "
                "bitsandbytes quantizer."
            ) from err

        self.quant_config = quant_config
```
**EN:** This block defines `BitsAndBytesLinearMethod.__init__()`, which initializes object state and validates constructor inputs.
**CN:** 该代码块定义了 `BitsAndBytesLinearMethod.__init__()`，用于初始化对象状态并校验构造参数。

### Lines 204-269: BitsAndBytesLinearMethod.create_weights()
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
    ):
        from bitsandbytes.nn import Int8Params

        def create_qweight_for_8bit():
            qweight = Int8Params(
                data=torch.empty(
                    sum(output_partition_sizes),
                    input_size_per_partition,
                    dtype=torch.int8,
                ),
                has_fp16_weights=self.quant_config.llm_int8_has_fp16_weight,
                requires_grad=False,
            )
            set_weight_attrs(
                qweight,
                {
                    "input_dim": 0,
                    "output_dim": 0,
                    "pack_factor": 1,
                    "use_bitsandbytes_8bit": True,
                    "generation": 0,
                },
            )
            return qweight

        def create_qweight_for_4bit():
            quant_ratio = calculate_quant_ratio(params_dtype)

            total_size = input_size_per_partition * sum(output_partition_sizes)
            if total_size % quant_ratio != 0:
                raise ValueError(
                    "The input size is not aligned with the quantized weight shape."
                )

            qweight = torch.nn.Parameter(
                torch.empty(total_size // quant_ratio, 1, dtype=torch.uint8),
                requires_grad=False,
            )
            set_weight_attrs(
                qweight,
                {
                    "input_dim": 0,
                    "output_dim": 0,
                    "pack_factor": quant_ratio,
                    "use_bitsandbytes_4bit": True,
                },
            )
            return qweight

        if self.quant_config.load_in_8bit:
            qweight = create_qweight_for_8bit()
        else:
            qweight = create_qweight_for_4bit()
        # Enable parameters to have the same name as in the BNB
        # checkpoint format.
        layer.register_parameter("weight", qweight)
        set_weight_attrs(qweight, extra_weight_attrs)
```
**EN:** This block defines `BitsAndBytesLinearMethod.create_weights()`, which creates parameters, buffers, or helper objects on demand.
**CN:** 该代码块定义了 `BitsAndBytesLinearMethod.create_weights()`，用于按需创建参数、缓冲区或辅助对象。

### Lines 271-280: BitsAndBytesLinearMethod.apply()
```python
    def apply(
        self,
        layer: torch.nn.Module,
        x: torch.Tensor,
        bias: torch.Tensor | None = None,
    ) -> torch.Tensor:
        if self.quant_config.load_in_8bit:
            return self._apply_8bit_weight(layer, x, bias)
        else:
            return self._apply_4bit_weight(layer, x, bias)
```
**EN:** This block defines `BitsAndBytesLinearMethod.apply()`, which applies quantized computation to runtime inputs.
**CN:** 该代码块定义了 `BitsAndBytesLinearMethod.apply()`，用于将量化计算应用到运行时输入上。

### Lines 282-326: BitsAndBytesLinearMethod._apply_8bit_weight() (part 1/2)
```python
    def _apply_8bit_weight(
        self,
        layer: torch.nn.Module,
        x: torch.Tensor,
        bias: torch.Tensor | None = None,
    ) -> torch.Tensor:
        # only load the bitsandbytes module when needed
        from bitsandbytes import MatmulLtState, matmul

        original_type = x.dtype
        original_shape = x.shape
        reshape_after_matmul = False
        if x.ndim > 2:
            x = x.reshape(-1, x.size(-1))
            reshape_after_matmul = True
        bf_x = x.to(torch.bfloat16)

        qweight = layer.weight
        offsets = qweight.bnb_shard_offsets
        quant_states = qweight.bnb_quant_state
        matmul_states = qweight.matmul_state
        generation = qweight.generation

        out_dim_0 = x.shape[0]
        out_dim_1 = sum(
            [quant_state[1].shape[0] for quant_state in quant_states.items()]
        )
        out = torch.empty(out_dim_0, out_dim_1, dtype=torch.float16, device=x.device)

        current_index = 0
        for i in range(len(quant_states)):
            output_size = quant_states[i].shape[0]

            # in profile_run or the first generation of inference,
            # create new matmul_states
            if generation == 0 or generation == 1:
                matmul_states[i] = MatmulLtState()
                matmul_states[i].CB = qweight[offsets[i] : offsets[i + 1]]
                matmul_states[i].SCB = quant_states[i].to(x.device)
                matmul_states[i].threshold = self.quant_config.llm_int8_threshold
                matmul_states[i].has_fp16_weights = (
                    self.quant_config.llm_int8_has_fp16_weight
                )
                matmul_states[i].is_training = False
                if (
```
**EN:** This segment of `BitsAndBytesLinearMethod._apply_8bit_weight()` implements reusable module logic for the quantization stack. It covers one portion of the full implementation.
**CN:** `BitsAndBytesLinearMethod._apply_8bit_weight()` 的这一段代码用于实现量化栈中的可复用模块逻辑，展示了完整实现中的一部分。

### Lines 327-360: BitsAndBytesLinearMethod._apply_8bit_weight() (part 2/2)
```python
                    matmul_states[i].threshold > 0.0
                    and not matmul_states[i].has_fp16_weights
                ):
                    matmul_states[i].use_pool = True

            new_x = bf_x.unsqueeze(0)

            out[:, current_index : current_index + output_size] = matmul(
                new_x, qweight[offsets[i] : offsets[i + 1]], state=matmul_states[i]
            )

            current_index += output_size

            # only update the matmul_states if it is not profile_run
            if (
                generation > 0
                and not self.quant_config.llm_int8_has_fp16_weight
                and matmul_states[i].CB is not None
                and matmul_states[i].CxB is not None
            ):
                del matmul_states[i].CB
                qweight[offsets[i] : offsets[i + 1]] = matmul_states[i].CxB

        out = out.to(original_type)

        if reshape_after_matmul:
            out = out.view(*original_shape[:-1], out.size(-1))

        if bias is not None:
            out += bias

        qweight.generation += 1

        return out
```
**EN:** This segment of `BitsAndBytesLinearMethod._apply_8bit_weight()` implements reusable module logic for the quantization stack. It covers one portion of the full implementation.
**CN:** `BitsAndBytesLinearMethod._apply_8bit_weight()` 的这一段代码用于实现量化栈中的可复用模块逻辑，展示了完整实现中的一部分。

### Lines 362-394: BitsAndBytesLinearMethod._apply_4bit_weight()
```python
    def _apply_4bit_weight(
        self,
        layer: torch.nn.Module,
        x: torch.Tensor,
        bias: torch.Tensor | None = None,
    ) -> torch.Tensor:
        original_type = x.dtype
        original_shape = x.shape
        reshape_after_matmul = False
        if x.ndim > 2:
            x = x.reshape(-1, x.size(-1))
            reshape_after_matmul = True
        bf_x = x.to(torch.bfloat16)

        qweight = layer.weight
        quant_states = qweight.bnb_quant_state
        offsets = qweight.bnb_shard_offsets

        out_dim_0 = x.shape[0]
        out_dim_1 = sum(
            [quant_state[1].shape[0] for quant_state in quant_states.items()]
        )
        out = torch.empty(out_dim_0, out_dim_1, dtype=torch.bfloat16, device=x.device)
        apply_bnb_4bit(bf_x, qweight, offsets, out)
        out = out.to(original_type)

        if reshape_after_matmul:
            out = out.view(*original_shape[:-1], out.size(-1))

        if bias is not None:
            out += bias

        return out
```
**EN:** This block defines `BitsAndBytesLinearMethod._apply_4bit_weight()`, which implements reusable module logic for the quantization stack.
**CN:** 该代码块定义了 `BitsAndBytesLinearMethod._apply_4bit_weight()`，用于实现量化栈中的可复用模块逻辑。

### Lines 397-418: apply_bnb_4bit()
```python
@register_custom_op(mutates_args=["out"])
def apply_bnb_4bit(
    x: torch.Tensor,
    weight: torch.Tensor,
    offsets: torch.Tensor,
    out: torch.Tensor,
) -> None:
    # only load the bitsandbytes module when needed
    from bitsandbytes import matmul_4bit

    quant_states = weight.bnb_quant_state
    current_index = 0
    for i in range(len(quant_states)):
        output_size = quant_states[i].shape[0]
        # It is more efficient to use out kwarg like
        # matmul_4bit(..., out = ...).  Infeasible now due to the bug
        # https://github.com/TimDettmers/bitsandbytes/issues/1235.
        # Need to change  after the bug is fixed.
        out[:, current_index : current_index + output_size] = matmul_4bit(
            x, weight[offsets[i] : offsets[i + 1]].t(), quant_states[i]
        )
        current_index += output_size
```
**EN:** This block defines `apply_bnb_4bit()`, which applies quantized computation to runtime inputs.
**CN:** 该代码块定义了 `apply_bnb_4bit()`，用于将量化计算应用到运行时输入上。

### Lines 421-427: class BitsAndBytesMoEMethod: definition
```python
class BitsAndBytesMoEMethod(FusedMoEMethodBase):
    """MoE method for BitsAndBytes.

    Args:
       quant_config: The BitsAndBytes quantization config.
    """
```
**EN:** This block declares `BitsAndBytesMoEMethod`, a runtime method class for the quantization stack. It organizes behaviors such as __init__, create_weights, create_moe_runner, apply.
**CN:** 该代码块声明 `BitsAndBytesMoEMethod`，它是量化栈中的运行方法类，组织了 __init__, create_weights, create_moe_runner, apply 等行为。

### Lines 428-447: BitsAndBytesMoEMethod.__init__()
```python
    def __init__(
        self,
        quant_config: BitsAndBytesConfig,
    ):
        super().__init__()
        try:
            import bitsandbytes

            if version.parse(bitsandbytes.__version__) < version.parse("0.46.1"):
                raise ImportError(
                    "bitsandbytes version is wrong. Please "
                    "install bitsandbytes>=0.46.1."
                )
        except ImportError as err:
            raise ImportError(
                "Please install bitsandbytes>=0.46.1 via "
                "`pip install bitsandbytes>=0.46.1` to use "
                "bitsandbytes quantizer."
            ) from err
        self.quant_config = quant_config
```
**EN:** This block defines `BitsAndBytesMoEMethod.__init__()`, which initializes object state and validates constructor inputs.
**CN:** 该代码块定义了 `BitsAndBytesMoEMethod.__init__()`，用于初始化对象状态并校验构造参数。

### Lines 449-469: BitsAndBytesMoEMethod.create_weights()
```python
    def create_weights(
        self,
        layer: torch.nn.Module,
        num_experts: int,
        hidden_size: int,
        intermediate_size_per_partition: int,
        params_dtype: torch.dtype,
        **extra_weight_attrs,
    ):
        if self.quant_config.load_in_8bit:
            call_fun = self._create_weights_8bit
        else:
            call_fun = self._create_weights_4bit
        call_fun(
            layer,
            num_experts,
            hidden_size,
            intermediate_size_per_partition,
            params_dtype,
            **extra_weight_attrs,
        )
```
**EN:** This block defines `BitsAndBytesMoEMethod.create_weights()`, which creates parameters, buffers, or helper objects on demand.
**CN:** 该代码块定义了 `BitsAndBytesMoEMethod.create_weights()`，用于按需创建参数、缓冲区或辅助对象。

### Lines 471-472: BitsAndBytesMoEMethod.create_moe_runner()
```python
    def create_moe_runner(self, layer: torch.nn.Module, moe_runner_config):
        self.moe_runner_config = moe_runner_config
```
**EN:** This block defines `BitsAndBytesMoEMethod.create_moe_runner()`, which creates parameters, buffers, or helper objects on demand.
**CN:** 该代码块定义了 `BitsAndBytesMoEMethod.create_moe_runner()`，用于按需创建参数、缓冲区或辅助对象。

### Lines 474-499: BitsAndBytesMoEMethod.apply()
```python
    def apply(
        self,
        layer: torch.nn.Module,
        dispatch_output: StandardDispatchOutput,
    ) -> CombineInput:
        from sglang.srt.layers.moe.moe_runner.triton_utils.fused_moe import fused_moe
        from sglang.srt.layers.moe.token_dispatcher import StandardCombineInput

        x = dispatch_output.hidden_states
        topk_output = dispatch_output.topk_output

        # TODO(bnell): Do these need to be called on the hot path?
        if self.quant_config.load_in_8bit:
            w13, w2 = self._apply_8bit_dequant(layer)
        else:
            w13, w2 = self._apply_4bit_dequant(layer)

        moe_runner_config = self.moe_runner_config
        output = fused_moe(
            hidden_states=x,
            w1=w13,
            w2=w2,
            topk_output=topk_output,
            moe_runner_config=moe_runner_config,
        )
        return StandardCombineInput(hidden_states=output)
```
**EN:** This block defines `BitsAndBytesMoEMethod.apply()`, which applies quantized computation to runtime inputs.
**CN:** 该代码块定义了 `BitsAndBytesMoEMethod.apply()`，用于将量化计算应用到运行时输入上。

### Lines 501-568: BitsAndBytesMoEMethod._create_weights_4bit()
```python
    def _create_weights_4bit(
        self,
        layer: torch.nn.Module,
        num_experts: int,
        hidden_size: int,
        intermediate_size_per_partition: int,
        params_dtype: torch.dtype,
        **extra_weight_attrs,
    ):
        quant_ratio = calculate_quant_ratio(params_dtype)
        # Fused gate_up_proj (column parallel)
        w13_total_size = (
            hidden_size * 2 * intermediate_size_per_partition
        ) // quant_ratio
        w13_qweight = torch.nn.Parameter(
            torch.empty(
                num_experts,
                w13_total_size,
                1,
                dtype=torch.uint8,
            ),
            requires_grad=False,
        )
        layer.register_parameter("w13_weight", w13_qweight)
        set_weight_attrs(w13_qweight, extra_weight_attrs)
        set_weight_attrs(
            w13_qweight,
            {
                "num_experts": num_experts,
                "input_dim": hidden_size,
                "output_dim": 2 * intermediate_size_per_partition,
                "experts_shape": (
                    num_experts,
                    intermediate_size_per_partition * 2,
                    hidden_size,
                ),
                "pack_factor": quant_ratio,
                "use_bitsandbytes_4bit": True,
            },
        )
        # down_proj (row parallel)
        w2_total_size = (hidden_size * intermediate_size_per_partition) // quant_ratio
        w2_qweight = torch.nn.Parameter(
            torch.empty(
                num_experts,
                w2_total_size,
                1,
                dtype=torch.uint8,
            ),
            requires_grad=False,
        )
        set_weight_attrs(
            w2_qweight,
            {
                "num_experts": num_experts,
                "input_dim": intermediate_size_per_partition,
                "output_dim": hidden_size,
                "experts_shape": (
                    num_experts,
                    hidden_size,
                    intermediate_size_per_partition,
                ),
                "pack_factor": quant_ratio,
                "use_bitsandbytes_4bit": True,
            },
        )
        layer.register_parameter("w2_weight", w2_qweight)
        set_weight_attrs(w2_qweight, extra_weight_attrs)
```
**EN:** This block defines `BitsAndBytesMoEMethod._create_weights_4bit()`, which implements reusable module logic for the quantization stack.
**CN:** 该代码块定义了 `BitsAndBytesMoEMethod._create_weights_4bit()`，用于实现量化栈中的可复用模块逻辑。

### Lines 570-579: BitsAndBytesMoEMethod._create_weights_8bit()
```python
    def _create_weights_8bit(
        self,
        layer: torch.nn.Module,
        num_experts: int,
        hidden_size: int,
        intermediate_size_per_partition: int,
        params_dtype: torch.dtype,
        **extra_weight_attrs,
    ):
        raise NotImplementedError
```
**EN:** This block defines `BitsAndBytesMoEMethod._create_weights_8bit()`, which implements reusable module logic for the quantization stack.
**CN:** 该代码块定义了 `BitsAndBytesMoEMethod._create_weights_8bit()`，用于实现量化栈中的可复用模块逻辑。

### Lines 581-596: BitsAndBytesMoEMethod._apply_4bit_dequant()
```python
    def _apply_4bit_dequant(
        self, layer: torch.nn.Module
    ) -> tuple[torch.Tensor, torch.Tensor]:
        from bitsandbytes.functional import dequantize_4bit

        w13 = dequantize_4bit(
            layer.w13_weight.reshape(-1, 1),
            layer.w13_weight.bnb_quant_state,
        )
        w2 = dequantize_4bit(
            layer.w2_weight.reshape(-1, 1),
            layer.w2_weight.bnb_quant_state,
        )
        w13 = w13.reshape(layer.w13_weight.experts_shape)
        w2 = w2.reshape(layer.w2_weight.experts_shape)
        return w13, w2
```
**EN:** This block defines `BitsAndBytesMoEMethod._apply_4bit_dequant()`, which restores low-precision data for downstream compute.
**CN:** 该代码块定义了 `BitsAndBytesMoEMethod._apply_4bit_dequant()`，用于为后续计算恢复低精度数据。

### Lines 598-601: BitsAndBytesMoEMethod._apply_8bit_dequant()
```python
    def _apply_8bit_dequant(
        self, layer: torch.nn.Module
    ) -> tuple[torch.Tensor, torch.Tensor]:
        raise NotImplementedError
```
**EN:** This block defines `BitsAndBytesMoEMethod._apply_8bit_dequant()`, which restores low-precision data for downstream compute.
**CN:** 该代码块定义了 `BitsAndBytesMoEMethod._apply_8bit_dequant()`，用于为后续计算恢复低精度数据。

## Key Concepts / 关键概念
- `BitsAndBytesConfig`: A configuration class that structures file-level quantization behavior. / `BitsAndBytesConfig` 是一个配置类，用于组织该文件中的量化行为。
- `is_layer_skipped_bnb()` : A public function that checks a condition used by higher-level control flow. / `is_layer_skipped_bnb()`：一个公开函数，用于检查供上层控制流使用的条件。
- `calculate_quant_ratio()` : A public function that handles tensor or weight quantization work. / `calculate_quant_ratio()`：一个公开函数，用于处理张量或权重量化逻辑。
- `BitsAndBytesLinearMethod`: A runtime method class that structures file-level quantization behavior. / `BitsAndBytesLinearMethod` 是一个运行方法类，用于组织该文件中的量化行为。
- `apply_bnb_4bit()` : A public function that applies quantized computation to runtime inputs. / `apply_bnb_4bit()`：一个公开函数，用于将量化计算应用到运行时输入上。
- `BitsAndBytesMoEMethod`: A runtime method class that structures file-level quantization behavior. / `BitsAndBytesMoEMethod` 是一个运行方法类，用于组织该文件中的量化行为。

## Dependencies / 依赖关系
- **External / 外部**: `__future__`, `bitsandbytes`, `bitsandbytes.functional`, `bitsandbytes.nn`, `packaging`, `torch`, `typing`
- **Internal / 内部**: `sglang.srt.layers.linear`, `sglang.srt.layers.moe.fused_moe_triton.layer`, `sglang.srt.layers.moe.moe_runner.triton_utils.fused_moe`, `sglang.srt.layers.moe.token_dispatcher`, `sglang.srt.layers.quantization.base_config`, `sglang.srt.layers.quantization.unquant`, `sglang.srt.utils`, `sglang.srt.utils.custom_op`
