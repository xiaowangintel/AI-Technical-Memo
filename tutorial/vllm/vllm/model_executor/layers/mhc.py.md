# mhc.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/layers/mhc.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements specialized model-executor layers centered on MHC attention/state mixing. / 实现围绕 MHC 注意力/状态混合的专用模型执行层。

## Line-by-Line Analysis / 逐行分析
### Imports and module setup (lines 3-7)
```python
import torch

# this import will also register the custom ops
import vllm.model_executor.kernels.mhc as mhc_kernels
from vllm.model_executor.custom_op import CustomOp
```
**EN:** This opening block pulls in external dependencies such as `torch` and internal modules such as `vllm.model_executor.kernels.mhc`, `vllm.model_executor.custom_op`. That import mix shows the file is part of the model-executor layers and helpers stack and whether it relies on PyTorch-only code or backend-specific helpers.
**CN:** 这个起始代码块引入了外部依赖（如 `torch`）以及内部模块（如 `vllm.model_executor.kernels.mhc`, `vllm.model_executor.custom_op`）。这些导入关系表明该文件属于模型执行层与辅助组件栈，并体现它更偏向纯 PyTorch 逻辑还是特定后端辅助实现。

### Class `MHCPreOp` overview (lines 12-95)
```python
class MHCPreOp(CustomOp):
    """MHC pre block.

    Computes mix logits from RMS-normalized HC residual streams, then
    returns post_mix, comb_mix, and
    layer_input = sum_i pre_mix_i * residual_i.
    """

    # --8<-- [end:mhc_pre]
    @classmethod
    def enabled(cls) -> bool:
        return True

    def forward_cuda(
        self,
        residual: torch.Tensor,
        fn: torch.Tensor,
        hc_scale: torch.Tensor,
        hc_base: torch.Tensor,
        rms_eps: float,
        hc_pre_eps: float,
        hc_sinkhorn_eps: float,
        hc_post_mult_value: float,
        sinkhorn_repeat: int,
        n_splits: int = 1,
```
**EN:** Defines class `MHCPreOp` with base classes `CustomOp` and decorators `CustomOp.register('mhc_pre')`. It acts as a reusable module building block and exposes 4 direct methods, with notable entries `enabled`, `forward_cuda`, `forward_hip`, `forward_native`. Its docstring says: MHC pre block.
**CN:** 定义类 `MHCPreOp`，其基类为 `CustomOp`，装饰器为 `CustomOp.register('mhc_pre')`。它在整体实现中充当可复用的模块构件，并直接暴露 4 个方法，较重要的包括 `enabled`, `forward_cuda`, `forward_hip`, `forward_native`。 文档字符串进一步说明了该类的定位。

### Method `MHCPreOp.enabled` (lines 22-23)
```python
    def enabled(cls) -> bool:
        return True
```
**EN:** Defines function `MHCPreOp.enabled` with signature `enabled(cls) -> bool`. It mainly works with object context only; implements one step of the module control flow. The body uses mostly straightforward data movement and object wiring. Key calls include no notable helper calls.
**CN:** 定义函数 `MHCPreOp.enabled`，其签名为 `enabled(cls) -> bool`。它主要围绕 仅依赖对象上下文 展开；实现模块控制流中的一个步骤。函数体包含以直接的数据组织和对象拼装为主。关键调用包括 无显著辅助调用。

### Method `MHCPreOp.forward_cuda` (lines 25-49)
```python
    def forward_cuda(
        self,
        residual: torch.Tensor,
        fn: torch.Tensor,
        hc_scale: torch.Tensor,
        hc_base: torch.Tensor,
        rms_eps: float,
        hc_pre_eps: float,
        hc_sinkhorn_eps: float,
        hc_post_mult_value: float,
        sinkhorn_repeat: int,
        n_splits: int = 1,
    ) -> tuple[torch.Tensor, torch.Tensor, torch.Tensor]:
        return torch.ops.vllm.mhc_pre_tilelang(
            residual,
            fn,
            hc_scale,
            hc_base,
            rms_eps,
            hc_pre_eps,
            hc_sinkhorn_eps,
            hc_post_mult_value,
            sinkhorn_repeat,
            n_splits,
        )
```
**EN:** Defines function `MHCPreOp.forward_cuda` with signature `forward_cuda(self, residual: torch.Tensor, fn: torch.Tensor, hc_scale: torch.Tensor, hc_base: torch.Tensor, rms_eps: float, hc_pre_eps: float, hc_sinkhorn_eps: float, hc_post_mult_value: float, sinkhorn_repeat: int, n_splits: int=1) -> tuple[torch.Tensor, torch.Tensor, torch.Tensor]`. It mainly works with `residual`, `fn`, `hc_scale`, `hc_base`, `rms_eps`, `hc_pre_eps`, `hc_sinkhorn_eps`, `hc_post_mult_value`; implements one step of the module control flow. The body uses tensor/kernel operations. Key calls include `torch.ops.vllm.mhc_pre_tilelang`.
**CN:** 定义函数 `MHCPreOp.forward_cuda`，其签名为 `forward_cuda(self, residual: torch.Tensor, fn: torch.Tensor, hc_scale: torch.Tensor, hc_base: torch.Tensor, rms_eps: float, hc_pre_eps: float, hc_sinkhorn_eps: float, hc_post_mult_value: float, sinkhorn_repeat: int, n_splits: int=1) -> tuple[torch.Tensor, torch.Tensor, torch.Tensor]`。它主要围绕 `residual`, `fn`, `hc_scale`, `hc_base`, `rms_eps`, `hc_pre_eps`, `hc_sinkhorn_eps`, `hc_post_mult_value` 展开；实现模块控制流中的一个步骤。函数体包含张量或内核操作。关键调用包括 `torch.ops.vllm.mhc_pre_tilelang`。

### Method `MHCPreOp.forward_hip` (lines 51-92)
```python
    def forward_hip(
        self,
        residual: torch.Tensor,
        fn: torch.Tensor,
        hc_scale: torch.Tensor,
        hc_base: torch.Tensor,
        rms_eps: float,
        hc_pre_eps: float,
        hc_sinkhorn_eps: float,
        hc_post_mult_value: float,
        sinkhorn_repeat: int,
        n_splits: int = 1,
    ) -> tuple[torch.Tensor, torch.Tensor, torch.Tensor]:
        # TODO: Reenable aiter after we are at the aiter
        # version that has this bugfix
        # https://github.com/ROCm/aiter/commit/b639cb63bcac4672dce33a731fad042a65cb3649
        # It has accuracy problem at large number of tokens.
        # hidden_size = residual.shape[-1]
        # if hidden_size % 256 == 0:
        #     return torch.ops.vllm.mhc_pre_aiter(
        #         residual,
        #         fn,
        #         hc_scale,
        #         hc_base,
        #         rms_eps,
        #         hc_pre_eps,
        #         hc_sinkhorn_eps,
        #         hc_post_mult_value,
        #         sinkhorn_repeat,
        #     )
        # else:
        return mhc_kernels.mhc_pre_torch(
            residual,
            fn,
            hc_scale,
            hc_base,
            rms_eps,
            hc_pre_eps,
            hc_sinkhorn_eps,
            hc_post_mult_value,
            sinkhorn_repeat,
        )
```
**EN:** Defines function `MHCPreOp.forward_hip` with signature `forward_hip(self, residual: torch.Tensor, fn: torch.Tensor, hc_scale: torch.Tensor, hc_base: torch.Tensor, rms_eps: float, hc_pre_eps: float, hc_sinkhorn_eps: float, hc_post_mult_value: float, sinkhorn_repeat: int, n_splits: int=1) -> tuple[torch.Tensor, torch.Tensor, torch.Tensor]`. It mainly works with `residual`, `fn`, `hc_scale`, `hc_base`, `rms_eps`, `hc_pre_eps`, `hc_sinkhorn_eps`, `hc_post_mult_value`; implements one step of the module control flow. The body uses mostly straightforward data movement and object wiring. Key calls include `mhc_kernels.mhc_pre_torch`.
**CN:** 定义函数 `MHCPreOp.forward_hip`，其签名为 `forward_hip(self, residual: torch.Tensor, fn: torch.Tensor, hc_scale: torch.Tensor, hc_base: torch.Tensor, rms_eps: float, hc_pre_eps: float, hc_sinkhorn_eps: float, hc_post_mult_value: float, sinkhorn_repeat: int, n_splits: int=1) -> tuple[torch.Tensor, torch.Tensor, torch.Tensor]`。它主要围绕 `residual`, `fn`, `hc_scale`, `hc_base`, `rms_eps`, `hc_pre_eps`, `hc_sinkhorn_eps`, `hc_post_mult_value` 展开；实现模块控制流中的一个步骤。函数体包含以直接的数据组织和对象拼装为主。关键调用包括 `mhc_kernels.mhc_pre_torch`。

### Method `MHCPreOp.forward_native` (lines 94-95)
```python
    def forward_native(self, *args, **kwargs):
        raise NotImplementedError("Native implementation of mhc_pre is not available")
```
**EN:** Defines function `MHCPreOp.forward_native` with signature `forward_native(self, *args, **kwargs)`. It mainly works with `*args`, `**kwargs`; implements one step of the module control flow. The body uses validation/error handling. Key calls include `NotImplementedError`.
**CN:** 定义函数 `MHCPreOp.forward_native`，其签名为 `forward_native(self, *args, **kwargs)`。它主要围绕 `*args`, `**kwargs` 展开；实现模块控制流中的一个步骤。函数体包含校验或报错逻辑。关键调用包括 `NotImplementedError`。

### Class `MHCPostOp` overview (lines 100-152)
```python
class MHCPostOp(CustomOp):
    """MHC post block.

    Combines the layer output with the HC residual streams:
    out_j = post_layer_mix_j * x + sum_i comb_res_mix_ij * residual_i.
    """

    # --8<-- [end:mhc_post]

    @classmethod
    def enabled(cls) -> bool:
        return True

    def forward_cuda(
        self,
        x: torch.Tensor,
        residual: torch.Tensor,
        post_layer_mix: torch.Tensor,
        comb_res_mix: torch.Tensor,
    ) -> torch.Tensor:
        return torch.ops.vllm.mhc_post_tilelang(
            x, residual, post_layer_mix, comb_res_mix
        )

    def forward_hip(
```
**EN:** Defines class `MHCPostOp` with base classes `CustomOp` and decorators `CustomOp.register('mhc_post')`. It acts as a reusable module building block and exposes 4 direct methods, with notable entries `enabled`, `forward_cuda`, `forward_hip`, `forward_native`. Its docstring says: MHC post block.
**CN:** 定义类 `MHCPostOp`，其基类为 `CustomOp`，装饰器为 `CustomOp.register('mhc_post')`。它在整体实现中充当可复用的模块构件，并直接暴露 4 个方法，较重要的包括 `enabled`, `forward_cuda`, `forward_hip`, `forward_native`。 文档字符串进一步说明了该类的定位。

### Method `MHCPostOp.enabled` (lines 110-111)
```python
    def enabled(cls) -> bool:
        return True
```
**EN:** Defines function `MHCPostOp.enabled` with signature `enabled(cls) -> bool`. It mainly works with object context only; implements one step of the module control flow. The body uses mostly straightforward data movement and object wiring. Key calls include no notable helper calls.
**CN:** 定义函数 `MHCPostOp.enabled`，其签名为 `enabled(cls) -> bool`。它主要围绕 仅依赖对象上下文 展开；实现模块控制流中的一个步骤。函数体包含以直接的数据组织和对象拼装为主。关键调用包括 无显著辅助调用。

### Method `MHCPostOp.forward_cuda` (lines 113-122)
```python
    def forward_cuda(
        self,
        x: torch.Tensor,
        residual: torch.Tensor,
        post_layer_mix: torch.Tensor,
        comb_res_mix: torch.Tensor,
    ) -> torch.Tensor:
        return torch.ops.vllm.mhc_post_tilelang(
            x, residual, post_layer_mix, comb_res_mix
        )
```
**EN:** Defines function `MHCPostOp.forward_cuda` with signature `forward_cuda(self, x: torch.Tensor, residual: torch.Tensor, post_layer_mix: torch.Tensor, comb_res_mix: torch.Tensor) -> torch.Tensor`. It mainly works with `x`, `residual`, `post_layer_mix`, `comb_res_mix`; implements one step of the module control flow. The body uses tensor/kernel operations. Key calls include `torch.ops.vllm.mhc_post_tilelang`.
**CN:** 定义函数 `MHCPostOp.forward_cuda`，其签名为 `forward_cuda(self, x: torch.Tensor, residual: torch.Tensor, post_layer_mix: torch.Tensor, comb_res_mix: torch.Tensor) -> torch.Tensor`。它主要围绕 `x`, `residual`, `post_layer_mix`, `comb_res_mix` 展开；实现模块控制流中的一个步骤。函数体包含张量或内核操作。关键调用包括 `torch.ops.vllm.mhc_post_tilelang`。

### Method `MHCPostOp.forward_hip` (lines 124-149)
```python
    def forward_hip(
        self,
        x: torch.Tensor,
        residual: torch.Tensor,
        post_layer_mix: torch.Tensor,
        comb_res_mix: torch.Tensor,
    ) -> torch.Tensor:
        # TODO: Reenable aiter after we are at the aiter
        # version that has this bugfix
        # https://github.com/ROCm/aiter/commit/b639cb63bcac4672dce33a731fad042a65cb3649
        # It has accuracy problem at large number of tokens.
        # hidden_size = residual.shape[-1]
        # if hidden_size % 256 == 0:
        #     return torch.ops.vllm.mhc_post_aiter(
        #         x,
        #         residual,
        #         post_layer_mix,
        #         comb_res_mix,
        #     )
        # else:
        return mhc_kernels.mhc_post_torch(
            x,
            residual,
            post_layer_mix,
            comb_res_mix,
        )
```
**EN:** Defines function `MHCPostOp.forward_hip` with signature `forward_hip(self, x: torch.Tensor, residual: torch.Tensor, post_layer_mix: torch.Tensor, comb_res_mix: torch.Tensor) -> torch.Tensor`. It mainly works with `x`, `residual`, `post_layer_mix`, `comb_res_mix`; implements one step of the module control flow. The body uses mostly straightforward data movement and object wiring. Key calls include `mhc_kernels.mhc_post_torch`.
**CN:** 定义函数 `MHCPostOp.forward_hip`，其签名为 `forward_hip(self, x: torch.Tensor, residual: torch.Tensor, post_layer_mix: torch.Tensor, comb_res_mix: torch.Tensor) -> torch.Tensor`。它主要围绕 `x`, `residual`, `post_layer_mix`, `comb_res_mix` 展开；实现模块控制流中的一个步骤。函数体包含以直接的数据组织和对象拼装为主。关键调用包括 `mhc_kernels.mhc_post_torch`。

### Method `MHCPostOp.forward_native` (lines 151-152)
```python
    def forward_native(self, *args, **kwargs):
        raise NotImplementedError("Native implementation of mhc_post is not available")
```
**EN:** Defines function `MHCPostOp.forward_native` with signature `forward_native(self, *args, **kwargs)`. It mainly works with `*args`, `**kwargs`; implements one step of the module control flow. The body uses validation/error handling. Key calls include `NotImplementedError`.
**CN:** 定义函数 `MHCPostOp.forward_native`，其签名为 `forward_native(self, *args, **kwargs)`。它主要围绕 `*args`, `**kwargs` 展开；实现模块控制流中的一个步骤。函数体包含校验或报错逻辑。关键调用包括 `NotImplementedError`。

### Class `HCHeadOp` overview (lines 157-231)
```python
class HCHeadOp(CustomOp):
    """HC head reduction for DeepSeek V4.

    Computes gates from the RMS-normalized flattened HC residual and
    returns out = sum_i gate_i * residual_i, collapsing hc_mult streams
    to one.
    """

    # --8<-- [end:hc_head]
    @classmethod
    def enabled(cls) -> bool:
        return True

    def forward_cuda(
        self,
        hidden_states: torch.Tensor,
        hc_fn: torch.Tensor,
        hc_scale: torch.Tensor,
        hc_base: torch.Tensor,
        rms_norm_eps: float,
        hc_eps: float,
    ) -> torch.Tensor:
        hc_mult, hidden_size = hidden_states.shape[-2:]
        outer_shape = hidden_states.shape[:-2]
        hs_flat = hidden_states.view(-1, hc_mult, hidden_size)
```
**EN:** Defines class `HCHeadOp` with base classes `CustomOp` and decorators `CustomOp.register('hc_head')`. It acts as a reusable module building block and exposes 4 direct methods, with notable entries `enabled`, `forward_cuda`, `forward_hip`, `forward_native`. Its docstring says: HC head reduction for DeepSeek V4.
**CN:** 定义类 `HCHeadOp`，其基类为 `CustomOp`，装饰器为 `CustomOp.register('hc_head')`。它在整体实现中充当可复用的模块构件，并直接暴露 4 个方法，较重要的包括 `enabled`, `forward_cuda`, `forward_hip`, `forward_native`。 文档字符串进一步说明了该类的定位。

### Method `HCHeadOp.enabled` (lines 167-168)
```python
    def enabled(cls) -> bool:
        return True
```
**EN:** Defines function `HCHeadOp.enabled` with signature `enabled(cls) -> bool`. It mainly works with object context only; implements one step of the module control flow. The body uses mostly straightforward data movement and object wiring. Key calls include no notable helper calls.
**CN:** 定义函数 `HCHeadOp.enabled`，其签名为 `enabled(cls) -> bool`。它主要围绕 仅依赖对象上下文 展开；实现模块控制流中的一个步骤。函数体包含以直接的数据组织和对象拼装为主。关键调用包括 无显著辅助调用。

### Method `HCHeadOp.forward_cuda` (lines 170-198)
```python
    def forward_cuda(
        self,
        hidden_states: torch.Tensor,
        hc_fn: torch.Tensor,
        hc_scale: torch.Tensor,
        hc_base: torch.Tensor,
        rms_norm_eps: float,
        hc_eps: float,
    ) -> torch.Tensor:
        hc_mult, hidden_size = hidden_states.shape[-2:]
        outer_shape = hidden_states.shape[:-2]
        hs_flat = hidden_states.view(-1, hc_mult, hidden_size)
        num_tokens = hs_flat.shape[0]

        out = torch.empty(
            num_tokens, hidden_size, dtype=torch.bfloat16, device=hidden_states.device
        )
        torch.ops.vllm.hc_head_fused_kernel_tilelang(
            hs_flat,
            hc_fn,
            hc_scale,
            hc_base,
            out,
            hidden_size,
            rms_norm_eps,
            hc_eps,
            hc_mult,
        )
        return out.view(*outer_shape, hidden_size)
```
**EN:** Defines function `HCHeadOp.forward_cuda` with signature `forward_cuda(self, hidden_states: torch.Tensor, hc_fn: torch.Tensor, hc_scale: torch.Tensor, hc_base: torch.Tensor, rms_norm_eps: float, hc_eps: float) -> torch.Tensor`. It mainly works with `hidden_states`, `hc_fn`, `hc_scale`, `hc_base`, `rms_norm_eps`, `hc_eps`; implements one step of the module control flow. The body uses tensor/kernel operations. Key calls include `hidden_states.view`, `torch.empty`, `torch.ops.vllm.hc_head_fused_kernel_tilelang`, `out.view`.
**CN:** 定义函数 `HCHeadOp.forward_cuda`，其签名为 `forward_cuda(self, hidden_states: torch.Tensor, hc_fn: torch.Tensor, hc_scale: torch.Tensor, hc_base: torch.Tensor, rms_norm_eps: float, hc_eps: float) -> torch.Tensor`。它主要围绕 `hidden_states`, `hc_fn`, `hc_scale`, `hc_base`, `rms_norm_eps`, `hc_eps` 展开；实现模块控制流中的一个步骤。函数体包含张量或内核操作。关键调用包括 `hidden_states.view`, `torch.empty`, `torch.ops.vllm.hc_head_fused_kernel_tilelang`, `out.view`。

### Method `HCHeadOp.forward_hip` (lines 200-228)
```python
    def forward_hip(
        self,
        hidden_states: torch.Tensor,
        hc_fn: torch.Tensor,
        hc_scale: torch.Tensor,
        hc_base: torch.Tensor,
        rms_norm_eps: float,
        hc_eps: float,
    ) -> torch.Tensor:
        hc_mult, hidden_size = hidden_states.shape[-2:]
        outer_shape = hidden_states.shape[:-2]
        hs_flat = hidden_states.view(-1, hc_mult, hidden_size)
        num_tokens = hs_flat.shape[0]

        out = torch.empty(
            num_tokens, hidden_size, dtype=torch.bfloat16, device=hidden_states.device
        )
        torch.ops.vllm.hc_head_triton(
            hs_flat,
            hc_fn,
            hc_scale,
            hc_base,
            out,
            hidden_size,
            rms_norm_eps,
            hc_eps,
            hc_mult,
        )
        return out.view(*outer_shape, hidden_size)
```
**EN:** Defines function `HCHeadOp.forward_hip` with signature `forward_hip(self, hidden_states: torch.Tensor, hc_fn: torch.Tensor, hc_scale: torch.Tensor, hc_base: torch.Tensor, rms_norm_eps: float, hc_eps: float) -> torch.Tensor`. It mainly works with `hidden_states`, `hc_fn`, `hc_scale`, `hc_base`, `rms_norm_eps`, `hc_eps`; implements one step of the module control flow. The body uses tensor/kernel operations. Key calls include `hidden_states.view`, `torch.empty`, `torch.ops.vllm.hc_head_triton`, `out.view`.
**CN:** 定义函数 `HCHeadOp.forward_hip`，其签名为 `forward_hip(self, hidden_states: torch.Tensor, hc_fn: torch.Tensor, hc_scale: torch.Tensor, hc_base: torch.Tensor, rms_norm_eps: float, hc_eps: float) -> torch.Tensor`。它主要围绕 `hidden_states`, `hc_fn`, `hc_scale`, `hc_base`, `rms_norm_eps`, `hc_eps` 展开；实现模块控制流中的一个步骤。函数体包含张量或内核操作。关键调用包括 `hidden_states.view`, `torch.empty`, `torch.ops.vllm.hc_head_triton`, `out.view`。

### Method `HCHeadOp.forward_native` (lines 230-231)
```python
    def forward_native(self, *args, **kwargs):
        raise NotImplementedError("Native implementation of hc_head is not available")
```
**EN:** Defines function `HCHeadOp.forward_native` with signature `forward_native(self, *args, **kwargs)`. It mainly works with `*args`, `**kwargs`; implements one step of the module control flow. The body uses validation/error handling. Key calls include `NotImplementedError`.
**CN:** 定义函数 `HCHeadOp.forward_native`，其签名为 `forward_native(self, *args, **kwargs)`。它主要围绕 `*args`, `**kwargs` 展开；实现模块控制流中的一个步骤。函数体包含校验或报错逻辑。关键调用包括 `NotImplementedError`。

### Class `MHCFusedPostPreOp` overview (lines 236-291)
```python
class MHCFusedPostPreOp(CustomOp):
    """Fused MHC post block followed by the next MHC pre block.

    Equivalent to applying MHCPostOp and then MHCPreOp to the updated
    residual streams, returning residual_cur, post_mix_cur, comb_mix_cur,
    and layer_input_cur.
    """

    # --8<-- [end:mhc_fused_post_pre]
    @classmethod
    def enabled(cls) -> bool:
        return True

    def forward_cuda(
        self,
        x: torch.Tensor,
        residual: torch.Tensor,
        post_layer_mix: torch.Tensor,
        comb_res_mix: torch.Tensor,
        fn: torch.Tensor,
        hc_scale: torch.Tensor,
        hc_base: torch.Tensor,
        rms_eps: float,
        hc_pre_eps: float,
        hc_sinkhorn_eps: float,
```
**EN:** Defines class `MHCFusedPostPreOp` with base classes `CustomOp` and decorators `CustomOp.register('mhc_fused_post_pre')`. It acts as a reusable module building block and exposes 4 direct methods, with notable entries `enabled`, `forward_cuda`, `forward_hip`, `forward_native`. Its docstring says: Fused MHC post block followed by the next MHC pre block.
**CN:** 定义类 `MHCFusedPostPreOp`，其基类为 `CustomOp`，装饰器为 `CustomOp.register('mhc_fused_post_pre')`。它在整体实现中充当可复用的模块构件，并直接暴露 4 个方法，较重要的包括 `enabled`, `forward_cuda`, `forward_hip`, `forward_native`。 文档字符串进一步说明了该类的定位。

### Method `MHCFusedPostPreOp.enabled` (lines 246-247)
```python
    def enabled(cls) -> bool:
        return True
```
**EN:** Defines function `MHCFusedPostPreOp.enabled` with signature `enabled(cls) -> bool`. It mainly works with object context only; implements one step of the module control flow. The body uses mostly straightforward data movement and object wiring. Key calls include no notable helper calls.
**CN:** 定义函数 `MHCFusedPostPreOp.enabled`，其签名为 `enabled(cls) -> bool`。它主要围绕 仅依赖对象上下文 展开；实现模块控制流中的一个步骤。函数体包含以直接的数据组织和对象拼装为主。关键调用包括 无显著辅助调用。

### Method `MHCFusedPostPreOp.forward_cuda` (lines 249-281)
```python
    def forward_cuda(
        self,
        x: torch.Tensor,
        residual: torch.Tensor,
        post_layer_mix: torch.Tensor,
        comb_res_mix: torch.Tensor,
        fn: torch.Tensor,
        hc_scale: torch.Tensor,
        hc_base: torch.Tensor,
        rms_eps: float,
        hc_pre_eps: float,
        hc_sinkhorn_eps: float,
        hc_post_mult_value: float,
        sinkhorn_repeat: int,
        n_splits: int = 1,
        tile_n: int = 1,
    ) -> tuple[torch.Tensor, torch.Tensor, torch.Tensor, torch.Tensor]:
        return torch.ops.vllm.mhc_fused_post_pre_tilelang(
            x,
            residual,
            post_layer_mix,
            comb_res_mix,
            fn,
            hc_scale,
            hc_base,
            rms_eps,
            hc_pre_eps,
            hc_sinkhorn_eps,
            hc_post_mult_value,
            sinkhorn_repeat,
            n_splits,
            tile_n,
        )
```
**EN:** Defines function `MHCFusedPostPreOp.forward_cuda` with signature `forward_cuda(self, x: torch.Tensor, residual: torch.Tensor, post_layer_mix: torch.Tensor, comb_res_mix: torch.Tensor, fn: torch.Tensor, hc_scale: torch.Tensor, hc_base: torch.Tensor, rms_eps: float, hc_pre_eps: float, hc_sinkhorn_eps: float, hc_post_mult_value: float, sinkhorn_repeat: int, n_splits: int=1, tile_n: int=1) -> tuple[torch.Tensor, torch.Tensor, torch.Tensor, torch.Tensor]`. It mainly works with `x`, `residual`, `post_layer_mix`, `comb_res_mix`, `fn`, `hc_scale`, `hc_base`, `rms_eps`; implements one step of the module control flow. The body uses tensor/kernel operations. Key calls include `torch.ops.vllm.mhc_fused_post_pre_tilelang`.
**CN:** 定义函数 `MHCFusedPostPreOp.forward_cuda`，其签名为 `forward_cuda(self, x: torch.Tensor, residual: torch.Tensor, post_layer_mix: torch.Tensor, comb_res_mix: torch.Tensor, fn: torch.Tensor, hc_scale: torch.Tensor, hc_base: torch.Tensor, rms_eps: float, hc_pre_eps: float, hc_sinkhorn_eps: float, hc_post_mult_value: float, sinkhorn_repeat: int, n_splits: int=1, tile_n: int=1) -> tuple[torch.Tensor, torch.Tensor, torch.Tensor, torch.Tensor]`。它主要围绕 `x`, `residual`, `post_layer_mix`, `comb_res_mix`, `fn`, `hc_scale`, `hc_base`, `rms_eps` 展开；实现模块控制流中的一个步骤。函数体包含张量或内核操作。关键调用包括 `torch.ops.vllm.mhc_fused_post_pre_tilelang`。

### Method `MHCFusedPostPreOp.forward_hip` (lines 283-286)
```python
    def forward_hip(self, *args, **kwargs):
        raise NotImplementedError(
            "Hip implementation of mhc_fused_post_pre is not available"
        )
```
**EN:** Defines function `MHCFusedPostPreOp.forward_hip` with signature `forward_hip(self, *args, **kwargs)`. It mainly works with `*args`, `**kwargs`; implements one step of the module control flow. The body uses validation/error handling. Key calls include `NotImplementedError`.
**CN:** 定义函数 `MHCFusedPostPreOp.forward_hip`，其签名为 `forward_hip(self, *args, **kwargs)`。它主要围绕 `*args`, `**kwargs` 展开；实现模块控制流中的一个步骤。函数体包含校验或报错逻辑。关键调用包括 `NotImplementedError`。

### Method `MHCFusedPostPreOp.forward_native` (lines 288-291)
```python
    def forward_native(self, *args, **kwargs):
        raise NotImplementedError(
            "Native implementation of mhc_fused_post_pre is not available"
        )
```
**EN:** Defines function `MHCFusedPostPreOp.forward_native` with signature `forward_native(self, *args, **kwargs)`. It mainly works with `*args`, `**kwargs`; implements one step of the module control flow. The body uses validation/error handling. Key calls include `NotImplementedError`.
**CN:** 定义函数 `MHCFusedPostPreOp.forward_native`，其签名为 `forward_native(self, *args, **kwargs)`。它主要围绕 `*args`, `**kwargs` 展开；实现模块控制流中的一个步骤。函数体包含校验或报错逻辑。关键调用包括 `NotImplementedError`。

## Key Concepts / 关键概念
- **EN:** The file provides a reusable layer/helper inside the model-executor subsystem.
  **CN:** 该文件在模型执行子系统中提供可复用的层或辅助逻辑。
- **EN:** Top-level classes include `MHCPreOp`, `MHCPostOp`, `HCHeadOp`, `MHCFusedPostPreOp`, which organize the main reusable abstractions.
  **CN:** 顶层类包括 `MHCPreOp`, `MHCPostOp`, `HCHeadOp`, `MHCFusedPostPreOp`，它们组织了主要的可复用抽象。
- **EN:** PyTorch is the main execution substrate, so most logic is expressed as tensor transformations and module wrappers.
  **CN:** PyTorch 是主要执行基础，因此大部分逻辑会表现为张量变换和模块封装。
- **EN:** Internal vLLM imports show that this module is tightly integrated with the broader runtime, parallelism, or custom-layer stack.
  **CN:** vLLM 内部导入说明该模块与更大的运行时、并行机制或自定义层栈紧密集成。

## Dependencies / 依赖关系
- **External / 外部**: `torch`
- **Internal / 内部**: `vllm.model_executor.kernels.mhc`, `vllm.model_executor.custom_op`
