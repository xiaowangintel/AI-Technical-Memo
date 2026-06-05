# common.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/kernels/moe/modular_kernel_tools/common.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Support module for kernel tests in kernels / moe / modular_kernel_tools / common; it provides reference logic, fixtures, or utilities reused by multiple scenarios. / kernels / moe / modular_kernel_tools / common 对应的内核测试支撑模块；它提供会被多个场景复用的参考逻辑、fixture 或工具函数。

## Line-by-Line Analysis / 逐行分析
### Imports and shared setup (lines 3-57)
```python
from dataclasses import dataclass
from typing import Any

import torch

import vllm._custom_ops as ops
import vllm.model_executor.layers.fused_moe.modular_kernel as mk
from tests.kernels.moe.utils import make_test_weights, per_token_cast_to_fp8
from tests.kernels.quantization.nvfp4_utils import (
    FLOAT4_E2M1_MAX,
    FLOAT8_E4M3_MAX,
    dequantize_nvfp4_to_dtype,
)
from tests.kernels.utils import torch_experts
from vllm.config import VllmConfig
from vllm.distributed import (
    get_dp_group,
    get_pcp_group,
    get_tensor_model_parallel_world_size,
)
from vllm.forward_context import set_forward_context
from vllm.model_executor.layers.fused_moe import fused_topk
from vllm.model_executor.layers.fused_moe.activation import MoEActivation
from vllm.model_executor.layers.fused_moe.all2all_utils import (
    maybe_make_prepare_finalize,
)
from vllm.model_executor.layers.fused_moe.config import (
    FusedMoEConfig,
    FusedMoEParallelConfig,
    FusedMoEQuantConfig,
    RoutingMethodType,
)
from vllm.model_executor.layers.quantization.utils.quant_utils import (
    kFp8Dynamic128Sym,
    kFp8DynamicTensorSym,
    kFp8DynamicTokenSym,
    kFp8Static128BlockSym,
    kFp8StaticChannelSym,
    kFp8StaticTensorSym,
)
from vllm.utils.import_utils import (
    has_aiter,
    has_deep_ep,
    has_deep_gemm,
    has_mori,
)
from vllm.utils.math_utils import next_power_of_2

from .mk_objects import (
    TestMoEQuantConfig,
    expert_info,
    make_fused_experts,
    prepare_finalize_info,
)
from .parallel_utils import ProcessGroupInfo
```
**EN:** This block imports the modules that the rest of the file relies on. It pulls in external libraries such as dataclasses, typing, torch; shared test helpers from tests.kernels.moe.utils, tests.kernels.quantization.nvfp4_utils, tests.kernels.utils, .mk_objects; and vLLM components like vllm._custom_ops, vllm.model_executor.layers.fused_moe.modular_kernel, vllm.config, vllm.distributed.
**CN:** 该代码块导入后续逻辑依赖的模块。 主要包括外部库，例如 dataclasses、typing、torch；共享测试辅助模块，例如 tests.kernels.moe.utils、tests.kernels.quantization.nvfp4_utils、tests.kernels.utils、.mk_objects；vLLM 内部组件，例如 vllm._custom_ops、vllm.model_executor.layers.fused_moe.modular_kernel、vllm.config、vllm.distributed。

### Function `_describe_tensor` (lines 60-64)
```python
def _describe_tensor(t: torch.Tensor | None, name: str) -> str:
    if t is None:
        return f"{name} : None"
    else:
        return f"{name} : {t.shape} {t.dtype} {t.device}"
```
**EN:** This helper function implements the shared logic for describe tensor. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 该辅助函数实现了 describe tensor 所需的共享逻辑。 它把周边模块中的共用逻辑封装成可复用单元。

### Class `Config` (lines 67-83)
```python
@dataclass
class Config:
    Ms: list[int] | int
    K: int
    N: int
    E: int
    topks: list[int] | int
    dtype: torch.dtype
    quant_config: TestMoEQuantConfig | None

    prepare_finalize_type: mk.FusedMoEPrepareAndFinalize
    fused_experts_type: mk.FusedMoEExperts

    world_size: int

    torch_trace_dir_path: str | None = None
```
**EN:** This dataclass packages the fields needed to describe Config. Keeping the inputs structured makes parameter sets explicit and easy to pass between helpers.
**CN:** 这个 dataclass 把描述 Config 所需的字段组织在一起，使参数集合更明确，也更容易在辅助函数之间传递。

### Method `Config.__post_init__` (lines 84-86)
```python
    def __post_init__(self):
        if self.quant_config is None:
            self.quant_config = TestMoEQuantConfig(None, False, False, None)
```
**EN:** This method on `Config` implements post init. It keeps the surrounding module logic factored into a reusable unit.
**CN:** `Config` 中的这个方法实现了 post init。 它把周边模块中的共用逻辑封装成可复用单元。

### Method `Config.describe` (lines 88-108)
```python
    def describe(self) -> str:
        s = ""
        s += "== Config:\n"
        s += f" world_size={self.world_size}\n"
        s += f" PF={self.prepare_finalize_type.__name__}\n"
        s += f" FE={self.fused_experts_type.__name__}\n"
        s += f" E={self.E}\n"
        s += f" Ms={self.Ms}\n"
        s += f" N={self.N}\n"
        s += f" K={self.K}\n"
        s += f" topk={self.topks}\n"
        s += f" dtype={self.dtype}\n"
        s += " Quant:\n"
        if self.quant_config is not None:
            s += f"     q_dtype={self.quant_dtype}\n"
            s += f"     q_block_shape={self.quant_block_shape}\n"
            s += f"     q_per_out_ch_quant={self.is_per_out_ch_quant}\n"
            s += f"     q_per_act_token={self.is_per_act_token_quant}\n"
        else:
            s += "     quant=None\n"
        return s
```
**EN:** This method on `Config` implements describe. It keeps the surrounding module logic factored into a reusable unit.
**CN:** `Config` 中的这个方法实现了 describe。 它把周边模块中的共用逻辑封装成可复用单元。

### Method `Config.M` (lines 110-113)
```python
    @property
    def M(self) -> int:
        assert isinstance(self.Ms, int)
        return self.Ms
```
**EN:** This method on `Config` implements M. assertions at the end lock in the intended behavior or graph shape.
**CN:** `Config` 中的这个方法实现了 M。 结尾处的断言会固定预期行为或计算图形态。

### Method `Config.quant_dtype` (lines 115-118)
```python
    @property
    def quant_dtype(self) -> torch.dtype | str | None:
        assert self.quant_config is not None
        return self.quant_config.quant_dtype
```
**EN:** This method on `Config` implements quant dtype. assertions at the end lock in the intended behavior or graph shape.
**CN:** `Config` 中的这个方法实现了 quant dtype。 结尾处的断言会固定预期行为或计算图形态。

### Method `Config.is_per_act_token_quant` (lines 120-123)
```python
    @property
    def is_per_act_token_quant(self) -> bool:
        assert self.quant_config is not None
        return self.quant_config.per_act_token_quant
```
**EN:** This method on `Config` implements is per act token quant. assertions at the end lock in the intended behavior or graph shape.
**CN:** `Config` 中的这个方法实现了 is per act token quant。 结尾处的断言会固定预期行为或计算图形态。

### Method `Config.is_per_tensor_act_quant` (lines 125-127)
```python
    @property
    def is_per_tensor_act_quant(self) -> bool:
        return not self.is_per_act_token_quant and self.quant_block_shape is None
```
**EN:** This method on `Config` implements is per tensor act quant. It keeps the surrounding module logic factored into a reusable unit.
**CN:** `Config` 中的这个方法实现了 is per tensor act quant。 它把周边模块中的共用逻辑封装成可复用单元。

### Method `Config.is_per_out_ch_quant` (lines 129-132)
```python
    @property
    def is_per_out_ch_quant(self) -> bool:
        assert self.quant_config is not None
        return self.quant_config.per_out_ch_quant
```
**EN:** This method on `Config` implements is per out ch quant. assertions at the end lock in the intended behavior or graph shape.
**CN:** `Config` 中的这个方法实现了 is per out ch quant。 结尾处的断言会固定预期行为或计算图形态。

### Method `Config.quant_block_shape` (lines 134-137)
```python
    @property
    def quant_block_shape(self) -> list[int] | None:
        assert self.quant_config is not None
        return self.quant_config.block_shape
```
**EN:** This method on `Config` implements quant block shape. assertions at the end lock in the intended behavior or graph shape.
**CN:** `Config` 中的这个方法实现了 quant block shape。 结尾处的断言会固定预期行为或计算图形态。

### Method `Config.topk` (lines 139-142)
```python
    @property
    def topk(self) -> int:
        assert isinstance(self.topks, int)
        return self.topks
```
**EN:** This method on `Config` implements topk. assertions at the end lock in the intended behavior or graph shape.
**CN:** `Config` 中的这个方法实现了 topk。 结尾处的断言会固定预期行为或计算图形态。

### Method `Config.num_local_experts` (lines 144-146)
```python
    @property
    def num_local_experts(self) -> int:
        return self.E // self.world_size
```
**EN:** This method on `Config` implements num local experts. It keeps the surrounding module logic factored into a reusable unit.
**CN:** `Config` 中的这个方法实现了 num local experts。 它把周边模块中的共用逻辑封装成可复用单元。

### Method `Config.make_env_data` (lines 148-162)
```python
    def make_env_data(self) -> tuple[VllmConfig, dict[Any, Any]]:
        """
        make env data for vllm launch.
        """
        vllm_config = VllmConfig()
        vllm_config.parallel_config.data_parallel_size = self.world_size
        vllm_config.parallel_config.enable_expert_parallel = True

        env_dict = {
            "VLLM_USE_DEEP_GEMM": str(int(self.needs_deep_gemm())),
        }

        vllm_config.parallel_config.all2all_backend = self.all2all_backend()

        return vllm_config, env_dict
```
**EN:** This method on `Config` implements env data. It keeps the surrounding module logic factored into a reusable unit.
**CN:** `Config` 中的这个方法实现了 env data。 它把周边模块中的共用逻辑封装成可复用单元。

### Method `Config.fe_supports_quant_scheme` (lines 164-195)
```python
    def fe_supports_quant_scheme(self) -> bool:
        """Check if the fused experts class supports this quant config.
        See https://github.com/ROCm/aiter/issues/2419 for AITER gaps."""
        if self.quant_config is None or self.quant_dtype is None:
            return True
        if self.quant_dtype != torch.float8_e4m3fn:
            return True
        # Derive QuantKeys from test config
        if self.quant_block_shape is not None:
            w_key = kFp8Static128BlockSym
            a_key = kFp8Dynamic128Sym
        elif self.is_per_out_ch_quant:
            w_key = kFp8StaticChannelSym
            a_key = (
                kFp8DynamicTokenSym
                if self.is_per_act_token_quant
                else kFp8StaticTensorSym
            )
        else:
            w_key = kFp8StaticTensorSym
            a_key = (
                kFp8DynamicTensorSym
                if self.is_per_act_token_quant
                else kFp8StaticTensorSym
            )
        fe_cls = self.fused_experts_type
        if hasattr(fe_cls, "_supports_quant_scheme"):
            try:
                return fe_cls._supports_quant_scheme(w_key, a_key)
            except NotImplementedError:
                pass
        return True
```
**EN:** This method on `Config` implements fe supports quant scheme. It keeps the surrounding module logic factored into a reusable unit.
**CN:** `Config` 中的这个方法实现了 fe supports quant scheme。 它把周边模块中的共用逻辑封装成可复用单元。

### Method `Config.is_fp8_block_quantized` (lines 197-201)
```python
    def is_fp8_block_quantized(self):
        return (
            self.quant_dtype == torch.float8_e4m3fn
            and self.quant_block_shape is not None
        )
```
**EN:** This method on `Config` implements is FP8 block quantized. It keeps the surrounding module logic factored into a reusable unit.
**CN:** `Config` 中的这个方法实现了 is FP8 block quantized。 它把周边模块中的共用逻辑封装成可复用单元。

### Method `Config.is_batched_prepare_finalize` (lines 203-205)
```python
    def is_batched_prepare_finalize(self):
        info = prepare_finalize_info(self.prepare_finalize_type)
        return mk.FusedMoEActivationFormat.BatchedExperts == info.activation_format
```
**EN:** This method on `Config` implements is batched prepare finalize. It keeps the surrounding module logic factored into a reusable unit.
**CN:** `Config` 中的这个方法实现了 is batched prepare finalize。 它把周边模块中的共用逻辑封装成可复用单元。

### Method `Config.is_batched_fused_experts` (lines 207-209)
```python
    def is_batched_fused_experts(self):
        info = expert_info(self.fused_experts_type)
        return mk.FusedMoEActivationFormat.BatchedExperts == info.activation_format
```
**EN:** This method on `Config` implements is batched fused experts. It keeps the surrounding module logic factored into a reusable unit.
**CN:** `Config` 中的这个方法实现了 is batched fused experts。 它把周边模块中的共用逻辑封装成可复用单元。

### Method `Config.is_standard_fused_experts` (lines 211-213)
```python
    def is_standard_fused_experts(self):
        info = expert_info(self.fused_experts_type)
        return mk.FusedMoEActivationFormat.Standard == info.activation_format
```
**EN:** This method on `Config` implements is standard fused experts. It keeps the surrounding module logic factored into a reusable unit.
**CN:** `Config` 中的这个方法实现了 is standard fused experts。 它把周边模块中的共用逻辑封装成可复用单元。

### Method `Config.fe_supported_types` (lines 215-217)
```python
    def fe_supported_types(self):
        info = expert_info(self.fused_experts_type)
        return info.supported_dtypes
```
**EN:** This method on `Config` implements fe supported types. It keeps the surrounding module logic factored into a reusable unit.
**CN:** `Config` 中的这个方法实现了 fe supported types。 它把周边模块中的共用逻辑封装成可复用单元。

### Method `Config.pf_supported_types` (lines 219-221)
```python
    def pf_supported_types(self):
        info = prepare_finalize_info(self.prepare_finalize_type)
        return info.supported_dtypes
```
**EN:** This method on `Config` implements pf supported types. It keeps the surrounding module logic factored into a reusable unit.
**CN:** `Config` 中的这个方法实现了 pf supported types。 它把周边模块中的共用逻辑封装成可复用单元。

### Method `Config.is_block_quant_supported` (lines 223-225)
```python
    def is_block_quant_supported(self):
        info = expert_info(self.fused_experts_type)
        return info.blocked_quantization_support
```
**EN:** This method on `Config` implements is block quant supported. It keeps the surrounding module logic factored into a reusable unit.
**CN:** `Config` 中的这个方法实现了 is block quant supported。 它把周边模块中的共用逻辑封装成可复用单元。

### Method `Config.supports_expert_map` (lines 227-229)
```python
    def supports_expert_map(self):
        info = expert_info(self.fused_experts_type)
        return info.supports_expert_map
```
**EN:** This method on `Config` implements supports expert map. It keeps the surrounding module logic factored into a reusable unit.
**CN:** `Config` 中的这个方法实现了 supports expert map。 它把周边模块中的共用逻辑封装成可复用单元。

### Method `Config.supports_apply_weight_on_input` (lines 231-233)
```python
    def supports_apply_weight_on_input(self):
        info = prepare_finalize_info(self.prepare_finalize_type)
        return info.supports_apply_weight_on_input
```
**EN:** This method on `Config` implements supports apply weight on input. It keeps the surrounding module logic factored into a reusable unit.
**CN:** `Config` 中的这个方法实现了 supports apply weight on input。 它把周边模块中的共用逻辑封装成可复用单元。

### Method `Config.needs_deep_gemm` (lines 235-237)
```python
    def needs_deep_gemm(self):
        info = expert_info(self.fused_experts_type)
        return info.needs_deep_gemm
```
**EN:** This method on `Config` implements needs deep gemm. It keeps the surrounding module logic factored into a reusable unit.
**CN:** `Config` 中的这个方法实现了 needs deep gemm。 它把周边模块中的共用逻辑封装成可复用单元。

### Method `Config.needs_deep_ep` (lines 239-244)
```python
    def needs_deep_ep(self):
        info = prepare_finalize_info(self.prepare_finalize_type)
        return (
            info.backend == "deepep_high_throughput"
            or info.backend == "deepep_low_latency"
        )
```
**EN:** This method on `Config` implements needs deep ep. It keeps the surrounding module logic factored into a reusable unit.
**CN:** `Config` 中的这个方法实现了 needs deep ep。 它把周边模块中的共用逻辑封装成可复用单元。

### Method `Config.needs_aiter` (lines 246-248)
```python
    def needs_aiter(self):
        info = expert_info(self.fused_experts_type)
        return info.needs_aiter
```
**EN:** This method on `Config` implements needs aiter. It keeps the surrounding module logic factored into a reusable unit.
**CN:** `Config` 中的这个方法实现了 needs aiter。 它把周边模块中的共用逻辑封装成可复用单元。

### Method `Config.needs_mori` (lines 250-252)
```python
    def needs_mori(self):
        info = prepare_finalize_info(self.prepare_finalize_type)
        return info.backend == "mori"
```
**EN:** This method on `Config` implements needs mori. It keeps the surrounding module logic factored into a reusable unit.
**CN:** `Config` 中的这个方法实现了 needs mori。 它把周边模块中的共用逻辑封装成可复用单元。

### Method `Config.all2all_backend` (lines 254-256)
```python
    def all2all_backend(self):
        info = prepare_finalize_info(self.prepare_finalize_type)
        return info.backend
```
**EN:** This method on `Config` implements all2all backend. It keeps the surrounding module logic factored into a reusable unit.
**CN:** `Config` 中的这个方法实现了 all2all backend。 它把周边模块中的共用逻辑封装成可复用单元。

### Method `Config.is_valid` (lines 258-329)
```python
    def is_valid(self) -> tuple[bool, str | None]:
        # Check prepare-finalize and fused-experts compatibility
        if self.is_batched_prepare_finalize():
            if not self.is_batched_fused_experts():
                return False, "Mismatched format."
        else:
            if not self.is_standard_fused_experts():
                return False, "Mismatched format."

        # Check quantization sanity
        if (
            int(self.is_per_act_token_quant)
            + int(self.is_per_tensor_act_quant)
            + int(self.quant_block_shape is not None)
        ) > 1:
            # invalid quant config
            return False, f"Bad quant_config {self.quant_config}."

        # check type support
        if self.quant_dtype is None:
            if (
                self.dtype not in self.pf_supported_types()
                or self.dtype not in self.fe_supported_types()
            ):
                return False, (
                    f"Unsupported type {self.dtype} not in "
                    f"{self.pf_supported_types()} and "
                    f"{self.fe_supported_types()}."
                )
        else:
            if (
                self.quant_dtype not in self.pf_supported_types()
                or self.quant_dtype not in self.fe_supported_types()
            ):
                return False, (
                    f"Unsupported quant type {self.quant_dtype} "
                    f"not in {self.pf_supported_types()} and "
                    f"{self.fe_supported_types()}."
                )

        # Check quant scheme compatibility with fused experts class
        if not self.fe_supports_quant_scheme():
            return False, (
                f"FE {self.fused_experts_type.__name__} does not support "
                f"quant scheme (per_out_ch={self.is_per_out_ch_quant}, "
                f"per_act_token={self.is_per_act_token_quant}, "
                f"block={self.quant_block_shape})"
            )

        # Check block quantization support
        is_block_quantized = self.quant_block_shape is not None
        if is_block_quantized and self.quant_dtype is None:
            return False, "No block quantization support."

        if is_block_quantized and not self.is_block_quant_supported():
            return False, "Mismatched block quantization support."

        # deep_gemm only works with block-quantized
        if self.needs_deep_gemm() and not is_block_quantized:
            return False, "Needs DeepGEMM but not block quantized."

        # Check dependencies (turn into asserts?)
        if self.needs_deep_ep() and not has_deep_ep():
            return False, "Needs DeepEP, but DeepEP not available."
        if self.needs_deep_gemm() and not has_deep_gemm():
            return False, "Needs DeepGEMM, but DeepGEMM not available."
        if self.needs_aiter() and not has_aiter():  # noqa: SIM103
            return False, "Needs Aiter, but Aiter not available."
        if self.needs_mori() and not has_mori():  # noqa: SIM103
            return False, "Needs MoRI, but MoRI not available."

        return True, None
```
**EN:** This method on `Config` implements is valid. It keeps the surrounding module logic factored into a reusable unit.
**CN:** `Config` 中的这个方法实现了 is valid。 它把周边模块中的共用逻辑封装成可复用单元。

### Class `WeightTensors` (lines 332-340)
```python
@dataclass
class WeightTensors:
    w1: torch.Tensor
    w2: torch.Tensor
    w1_scale: torch.Tensor | None
    w2_scale: torch.Tensor | None
    w1_gs: torch.Tensor | None = None
    w2_gs: torch.Tensor | None = None
```
**EN:** This dataclass packages the fields needed to describe WeightTensors. Keeping the inputs structured makes parameter sets explicit and easy to pass between helpers.
**CN:** 这个 dataclass 把描述 WeightTensors 所需的字段组织在一起，使参数集合更明确，也更容易在辅助函数之间传递。

### Method `WeightTensors.describe` (lines 341-350)
```python
    def describe(self):
        s = ""
        s += "== Weight Tensors: \n"
        s += f" - {_describe_tensor(self.w1, 'w1')} \n"
        s += f" - {_describe_tensor(self.w2, 'w2')} \n"
        s += f" - {_describe_tensor(self.w1_scale, 'w1_scale')} \n"
        s += f" - {_describe_tensor(self.w2_scale, 'w2_scale')} \n"
        s += f" - {_describe_tensor(self.w1_gs, 'w1_gs')} \n"
        s += f" - {_describe_tensor(self.w2_gs, 'w2_gs')} \n"
        return s
```
**EN:** This method on `WeightTensors` implements describe. It keeps the surrounding module logic factored into a reusable unit.
**CN:** `WeightTensors` 中的这个方法实现了 describe。 它把周边模块中的共用逻辑封装成可复用单元。

### Method `WeightTensors.is_quantized` (lines 352-358)
```python
    def is_quantized(self) -> bool:
        # or w1_scale is not None?
        return (
            self.w1.dtype == torch.float8_e4m3fn
            or self.w1.dtype == torch.uint8
            or self.w1.dtype == torch.int8
        )
```
**EN:** This method on `WeightTensors` implements is quantized. It keeps the surrounding module logic factored into a reusable unit.
**CN:** `WeightTensors` 中的这个方法实现了 is quantized。 它把周边模块中的共用逻辑封装成可复用单元。

### Method `WeightTensors.to_current_device` (lines 360-373)
```python
    def to_current_device(self):
        device = torch.accelerator.current_device_index()
        self.w1 = self.w1.to(device=device)
        self.w2 = self.w2.to(device=device)

        if self.w1_scale is not None:
            self.w1_scale = self.w1_scale.to(device=device)
        if self.w2_scale is not None:
            self.w2_scale = self.w2_scale.to(device=device)

        if self.w1_gs is not None:
            self.w1_gs = self.w1_gs.to(device=device)
        if self.w2_gs is not None:
            self.w2_gs = self.w2_gs.to(device=device)
```
**EN:** This method on `WeightTensors` implements to current device. It keeps the surrounding module logic factored into a reusable unit.
**CN:** `WeightTensors` 中的这个方法实现了 to current device。 它把周边模块中的共用逻辑封装成可复用单元。

### Method `WeightTensors.slice_weights` (lines 375-385)
```python
    def slice_weights(self, rank: int, num_local_experts: int) -> "WeightTensors":
        s = rank * num_local_experts
        e = s + num_local_experts
        w1 = self.w1[s:e, :, :]
        w2 = self.w2[s:e, :, :]
        w1_scale = self.w1_scale[s:e, :, :] if self.w1_scale is not None else None
        w2_scale = self.w2_scale[s:e, :, :] if self.w2_scale is not None else None
        w1_gs = self.w1_gs[s:e] if self.w1_gs is not None else None
        w2_gs = self.w2_gs[s:e] if self.w2_gs is not None else None

        return WeightTensors(w1, w2, w1_scale, w2_scale, w1_gs, w2_gs)
```
**EN:** This method on `WeightTensors` implements slice weights. It keeps the surrounding module logic factored into a reusable unit.
**CN:** `WeightTensors` 中的这个方法实现了 slice weights。 它把周边模块中的共用逻辑封装成可复用单元。

### Method `WeightTensors.make` (lines 387-401)
```python
    @staticmethod
    def make(config: Config) -> "WeightTensors":
        (_, w1, w1_scale, w1_gs), (_, w2, w2_scale, w2_gs) = make_test_weights(
            e=config.E,
            n=config.N,
            k=config.K,
            in_dtype=config.dtype,
            quant_dtype=config.quant_dtype,
            block_shape=config.quant_block_shape,
            # or config.is_per_out_ch_quant
            per_out_ch_quant=config.is_per_act_token_quant,
        )
        return WeightTensors(
            w1=w1, w2=w2, w1_scale=w1_scale, w2_scale=w2_scale, w1_gs=w1_gs, w2_gs=w2_gs
        )
```
**EN:** This method on `WeightTensors` implements make. It keeps the surrounding module logic factored into a reusable unit.
**CN:** `WeightTensors` 中的这个方法实现了 make。 它把周边模块中的共用逻辑封装成可复用单元。

### Class `RankTensors` (lines 404-412)
```python
@dataclass
class RankTensors:
    hidden_states: torch.Tensor
    hidden_states_scale: torch.Tensor | None

    topk_weights: torch.Tensor
    topk_ids: torch.Tensor
    expert_map: torch.Tensor | None
```
**EN:** This dataclass packages the fields needed to describe RankTensors. Keeping the inputs structured makes parameter sets explicit and easy to pass between helpers.
**CN:** 这个 dataclass 把描述 RankTensors 所需的字段组织在一起，使参数集合更明确，也更容易在辅助函数之间传递。

### Method `RankTensors.describe` (lines 413-421)
```python
    def describe(self):
        s = ""
        s += "== Rank Tensors: \n"
        s += f" - {_describe_tensor(self.hidden_states, 'HS')} \n"
        s += f" - {_describe_tensor(self.hidden_states_scale, 'HS_scale')} \n"
        s += f" - {_describe_tensor(self.topk_weights, 'topk_weights')} \n"
        s += f" - {_describe_tensor(self.topk_ids, 'topk_ids')} \n"
        s += f" - {_describe_tensor(self.expert_map, 'expert_map')} \n"
        return s
```
**EN:** This method on `RankTensors` implements describe. It keeps the surrounding module logic factored into a reusable unit.
**CN:** `RankTensors` 中的这个方法实现了 describe。 它把周边模块中的共用逻辑封装成可复用单元。

### Method `RankTensors.make_hidden_states` (lines 423-455)
```python
    @staticmethod
    def make_hidden_states(
        config: Config,
    ) -> tuple[torch.Tensor, torch.Tensor | None]:
        """
        Return hidden_states
        """
        m, k, dtype = (config.M, config.K, config.dtype)
        device = torch.accelerator.current_device_index()
        a = torch.randn((m, k), device=device, dtype=dtype) / 15.0

        if config.quant_dtype is None:
            return a, None

        # We dequant and use that as hidden_states so the tests are stable.
        # quantizing and dequantizing yield slightly different results
        # depending on the hardware. Here we, quantize and dequantize
        # first - so further quantize and dequantize will yield the same
        # values.
        if config.is_per_tensor_act_quant:
            a_q, a_scales = ops.scaled_fp8_quant(a, use_per_token_if_dynamic=False)
            return a_q.float().mul(a_scales).to(dtype), a_scales

        if config.is_per_act_token_quant:
            a_q, a_scales = ops.scaled_fp8_quant(a, use_per_token_if_dynamic=True)
            return a_q.float().mul(a_scales).to(dtype), None

        assert config.quant_block_shape is not None
        block_k = config.quant_block_shape[1]
        a_q, a_scales = per_token_cast_to_fp8(a, block_size=block_k)
        return a_q.float().view((-1, block_k)).mul(a_scales.view(-1, 1)).view(m, k).to(
            dtype
        ), None
```
**EN:** This method on `RankTensors` implements hidden states. assertions at the end lock in the intended behavior or graph shape.
**CN:** `RankTensors` 中的这个方法实现了 hidden states。 结尾处的断言会固定预期行为或计算图形态。

### Method `RankTensors.make` (lines 457-489)
```python
    @staticmethod
    def make(config: Config, pgi: ProcessGroupInfo):
        dtype = config.dtype
        topk, m, _ = (config.topk, config.M, config.K)
        hidden_states, hidden_states_scale = RankTensors.make_hidden_states(config)

        num_local_experts, global_num_experts = (config.num_local_experts, config.E)
        score = torch.randn((m, global_num_experts), device="cuda", dtype=dtype)
        topk_weights, topk_ids, _ = fused_topk(hidden_states, score, topk, False)

        # distribute topk_ids evenly
        device = torch.accelerator.current_device_index()
        for mi in range(m):
            topk_ids[mi] = torch.randperm(config.E)[:topk]
        topk_ids = topk_ids.to(device=device)

        expert_map = None
        if config.world_size > 1 and config.supports_expert_map():
            expert_map = torch.full(
                (global_num_experts,), fill_value=-1, dtype=torch.int32
            )
            s = pgi.rank * num_local_experts
            e = s + num_local_experts
            expert_map[s:e] = torch.tensor(list(range(num_local_experts)))
            expert_map = expert_map.to(device=device, dtype=torch.int32)

        return RankTensors(
            hidden_states=hidden_states,
            hidden_states_scale=hidden_states_scale,
            topk_weights=topk_weights,
            topk_ids=topk_ids,
            expert_map=expert_map,
        )
```
**EN:** This method on `RankTensors` implements make. It keeps the surrounding module logic factored into a reusable unit.
**CN:** `RankTensors` 中的这个方法实现了 make。 它把周边模块中的共用逻辑封装成可复用单元。

### Function `reference_moe_impl` (lines 492-592)
```python
def reference_moe_impl(
    config: Config, weights: WeightTensors, rank_tensors: RankTensors
) -> torch.Tensor:
    if config.quant_dtype == "nvfp4":
        quant_blocksize = 16
        dtype = config.dtype

        w1_q = weights.w1
        w1_blockscale = weights.w1_scale
        w1_gs = weights.w1_gs

        w2_q = weights.w2
        w2_blockscale = weights.w2_scale
        w2_gs = weights.w2_gs

        a_global_scale = (
            (FLOAT8_E4M3_MAX * FLOAT4_E2M1_MAX)
            / torch.amax(rank_tensors.hidden_states.flatten(), dim=-1)
        ).to(torch.float32)

        assert w1_gs is not None
        assert w2_gs is not None
        assert w1_blockscale is not None
        assert w2_blockscale is not None

        assert w1_blockscale.shape[1] % 128 == 0
        assert w1_blockscale.shape[2] % 4 == 0
        assert w2_blockscale.shape[1] % 128 == 0
        assert w2_blockscale.shape[2] % 4 == 0

        a_fp4, a_scale_interleaved = ops.scaled_fp4_quant(
            rank_tensors.hidden_states, a_global_scale
        )

        a = dequantize_nvfp4_to_dtype(
            a_fp4,
            a_scale_interleaved,
            a_global_scale,
            dtype=dtype,
            device=a_fp4.device,
# ... excerpt ...
                w2_gs[idx],
                dtype=dtype,
                device=w2_q.device,
                block_size=quant_blocksize,
            )
        a_scale = None
        w1_scale = None
        w2_scale = None
        quant_dtype = None
        per_act_token_quant = False
        block_shape = None
    else:
        a = rank_tensors.hidden_states
        a_scale = rank_tensors.hidden_states_scale
        w1 = weights.w1
        w1_scale = weights.w1_scale
        w2 = weights.w2
        w2_scale = weights.w2_scale
        quant_dtype = config.quant_dtype
        per_act_token_quant = config.is_per_act_token_quant
        block_shape = config.quant_block_shape

    return torch_experts(
        a=a,
        w1=w1,
        w2=w2,
        topk_weight=rank_tensors.topk_weights,
        topk_ids=rank_tensors.topk_ids,
        global_num_experts=config.E,
        expert_map=None,
        w1_scale=w1_scale,
        w2_scale=w2_scale,
        a1_scale=a_scale,
        quant_dtype=quant_dtype,
        per_act_token_quant=per_act_token_quant,
        block_shape=block_shape,
        apply_router_weights_on_input=config.topk == 1
        and config.supports_apply_weight_on_input(),
    )
```
**EN:** This helper function implements the shared logic for reference MoE impl. assertions at the end lock in the intended behavior or graph shape. Only the key portions are shown here because the block is large.
**CN:** 该辅助函数实现了 reference MoE impl 所需的共享逻辑。 结尾处的断言会固定预期行为或计算图形态。 由于该代码块较大，这里只展示关键片段。

### Function `_make_gscale` (lines 595-600)
```python
def _make_gscale(num_experts: int) -> torch.Tensor:
    return torch.ones(
        (num_experts,),
        device=torch.accelerator.current_device_index(),
        dtype=torch.float32,
    )
```
**EN:** This helper function implements the shared logic for make gscale. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 该辅助函数实现了 make gscale 所需的共享逻辑。 它把周边模块中的共用逻辑封装成可复用单元。

### Function `make_modular_kernel` (lines 603-653)
```python
def make_modular_kernel(
    config: Config,
    vllm_config: VllmConfig,
    quant_config: FusedMoEQuantConfig,
) -> mk.FusedMoEKernel:
    # make moe config
    moe_parallel_config: FusedMoEParallelConfig = FusedMoEParallelConfig.make(
        tp_size_=get_tensor_model_parallel_world_size(),
        pcp_size_=get_pcp_group().world_size,
        dp_size_=get_dp_group().world_size,
        sp_size_=1,
        vllm_parallel_config=vllm_config.parallel_config,
    )

    moe = FusedMoEConfig(
        num_experts=config.E,
        experts_per_token=config.topk,
        hidden_dim=config.K,
        intermediate_size_per_partition=config.N,
        num_local_experts=config.num_local_experts,
        num_logical_experts=config.E,
        moe_parallel_config=moe_parallel_config,
        in_dtype=config.dtype,
        max_num_tokens=next_power_of_2(config.M),
        activation=MoEActivation.SILU,
        device=vllm_config.device_config.device,
        routing_method=RoutingMethodType.DeepSeekV3,
    )

    prepare_finalize = maybe_make_prepare_finalize(
        moe=moe,
        quant_config=quant_config,
        allow_new_interface=True,
    )
    assert prepare_finalize is not None

    fused_experts = make_fused_experts(
        config.fused_experts_type,
        moe,
        quant_config,
        prepare_finalize.num_dispatchers(),
        config.N,
    )

    modular_kernel = mk.FusedMoEKernel(
        prepare_finalize=prepare_finalize,
        fused_experts=fused_experts,
        inplace=False,
    )

    return modular_kernel
```
**EN:** This helper function implements the shared logic for modular kernel. assertions at the end lock in the intended behavior or graph shape.
**CN:** 该辅助函数实现了 modular kernel 所需的共享逻辑。 结尾处的断言会固定预期行为或计算图形态。

### Function `run_modular_kernel` (lines 656-721)
```python
def run_modular_kernel(
    pgi: ProcessGroupInfo,
    vllm_config: VllmConfig,
    config: Config,
    weights: WeightTensors,
    rank_tensors: RankTensors,
) -> torch.Tensor:
    assert isinstance(config.Ms, int)
    assert isinstance(config.topks, int)

    # weights for rank
    rank_weights = weights.slice_weights(pgi.rank, config.num_local_experts)

    if config.quant_dtype == "nvfp4":
        gscale = _make_gscale(config.num_local_experts)
    else:
        gscale = None

    quant_config = FusedMoEQuantConfig.make(
        config.quant_dtype,
        w1_scale=rank_weights.w1_scale,
        w2_scale=rank_weights.w2_scale,
        a1_scale=rank_tensors.hidden_states_scale,
        g1_alphas=(1 / rank_weights.w1_gs) if rank_weights.w1_gs is not None else None,
        g2_alphas=(1 / rank_weights.w2_gs) if rank_weights.w2_gs is not None else None,
        a1_gscale=gscale,
        a2_gscale=gscale,
        block_shape=config.quant_block_shape,
        per_act_token_quant=config.is_per_act_token_quant,
        per_out_ch_quant=config.is_per_out_ch_quant,
    )

    mk = make_modular_kernel(config, vllm_config, quant_config)

    # impls might update the tensor in place
    hidden_states = rank_tensors.hidden_states.clone()

    topk_ids = rank_tensors.topk_ids.to(mk.prepare_finalize.topk_indices_dtype())

    mk_kwargs = {
        "hidden_states": hidden_states,
        "w1": rank_weights.w1,
        "w2": rank_weights.w2,
        "topk_weights": rank_tensors.topk_weights,
        "topk_ids": topk_ids,
        "activation": MoEActivation.SILU,
        "expert_map": rank_tensors.expert_map,
        "global_num_experts": config.E,
        "apply_router_weight_on_input": config.topk == 1
        and config.supports_apply_weight_on_input(),
    }

    num_tokens = rank_tensors.hidden_states.shape[0]
    num_tokens_across_dp = torch.tensor(
        [num_tokens] * config.world_size, device="cuda", dtype=torch.int
    )

    with set_forward_context(
        None,
        vllm_config,
        num_tokens=num_tokens,
        num_tokens_across_dp=num_tokens_across_dp,
    ):
        out = mk.apply(**mk_kwargs)

    return out
```
**EN:** This helper function implements the shared logic for run modular kernel. assertions at the end lock in the intended behavior or graph shape.
**CN:** 该辅助函数实现了 run modular kernel 所需的共享逻辑。 结尾处的断言会固定预期行为或计算图形态。

## Key Concepts / 关键概念
- **Behavioral assertions / 行为断言:** Assertions check tensor values, graph structure, or expected error conditions. / 断言用于检查张量结果、计算图结构或预期的报错路径。
- **Reference comparisons / 参考实现对照:** Optimized kernels are commonly checked against simpler reference implementations or fallback paths. / 优化后的内核通常会与更直接的参考实现或回退路径进行比对。
- **Shared test utilities / 共享测试工具:** The module factors repeated setup or reference logic out of the individual test files. / 该模块把重复的初始化或参考逻辑从单个测试文件中抽离出来复用。

## Dependencies / 依赖关系
- `dataclasses -> dataclass`
- `typing -> Any`
- `torch`
- `vllm._custom_ops`
- `vllm.model_executor.layers.fused_moe.modular_kernel`
- `tests.kernels.moe.utils -> make_test_weights, per_token_cast_to_fp8`
- `tests.kernels.quantization.nvfp4_utils -> FLOAT4_E2M1_MAX, FLOAT8_E4M3_MAX, dequantize_nvfp4_to_dtype`
- `tests.kernels.utils -> torch_experts`
- `vllm.config -> VllmConfig`
- `vllm.distributed -> get_dp_group, get_pcp_group, get_tensor_model_parallel_world_size`
- `vllm.forward_context -> set_forward_context`
- `vllm.model_executor.layers.fused_moe -> fused_topk`
- `vllm.model_executor.layers.fused_moe.activation -> MoEActivation`
- `vllm.model_executor.layers.fused_moe.all2all_utils -> maybe_make_prepare_finalize`
- `vllm.model_executor.layers.fused_moe.config -> FusedMoEConfig, FusedMoEParallelConfig, FusedMoEQuantConfig, RoutingMethodType`
- `vllm.model_executor.layers.quantization.utils.quant_utils -> kFp8Dynamic128Sym, kFp8DynamicTensorSym, kFp8DynamicTokenSym, kFp8Static128BlockSym, kFp8StaticChannelSym, kFp8StaticTensorSym`
