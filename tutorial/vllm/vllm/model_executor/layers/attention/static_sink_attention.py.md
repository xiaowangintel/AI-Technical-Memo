# static_sink_attention.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/layers/attention/static_sink_attention.py`
- **Repository**: vllm-project/vllm
- **Purpose**: attention execution, backend dispatch, and KV-cache handling / 注意力执行、后端分派与 KV 缓存处理

## Line-by-Line Analysis / 逐行分析
### Lines 3-37 — imports and setup
```python
import functools

import torch

from vllm.config import CacheConfig, VllmConfig
from vllm.forward_context import ForwardContext, get_forward_context
from vllm.logger import init_logger
from vllm.model_executor.custom_op import CustomOp
from vllm.model_executor.layers.attention import Attention
from vllm.utils.math_utils import cdiv
from vllm.utils.torch_utils import (
    LayerNameType,
    _encode_layer_name,
    _resolve_layer_name,
    direct_register_custom_op,
)
from vllm.v1.attention.backend import (
    AttentionBackend,
    AttentionMetadata,
    AttentionType,
    CommonAttentionMetadata,
    subclass_attention_backend,
)
from vllm.v1.attention.ops.triton_reshape_and_cache_flash import (
    triton_reshape_and_cache_flash_diffkv,
)
from vllm.v1.attention.selector import get_attn_backend
from vllm.v1.kv_cache_interface import (
    AttentionSpec,
    KVCacheSpec,
    SinkFullAttentionSpec,
    get_kv_quant_mode,
)

logger = init_logger(__name__)
```
**EN:** This block gathers the imports and module-level setup used by the rest of the file. Key calls include `init_logger`. It writes or updates `logger`.
**CN:** 该代码块汇总了后续实现依赖的导入项与模块级初始化。 关键调用包括 `init_logger`。 它会写入或更新 `logger`。

### Lines 40-112 — function `create_static_sink_attention_backend`
```python
@functools.lru_cache
def create_static_sink_attention_backend(
    underlying_attn_backend: type[AttentionBackend],
    sink_len: int = 0,
) -> type[AttentionBackend]:
    prefix = "StaticSink_"
    underlying_builder = underlying_attn_backend.get_builder_cls()

    class StaticSinkAttentionBuilder(underlying_builder):  # type: ignore
        def __init__(
            self,
            kv_cache_spec: AttentionSpec,
            layer_names: list[str],
            vllm_config: VllmConfig,
            device: torch.device,
        ):
            super().__init__(kv_cache_spec, layer_names, vllm_config, device)
            model_config = vllm_config.model_config
            scheduler_config = vllm_config.scheduler_config
            self.sink_len = sink_len
            self.block_size = vllm_config.cache_config.block_size
            self.num_sink_blocks = self.sink_len // vllm_config.cache_config.block_size
            self.max_num_blocks = cdiv(
                model_config.max_model_len, vllm_config.cache_config.block_size
            )
            self.block_table_with_sink = torch.zeros(
                (
                    scheduler_config.max_num_seqs,
                    self.max_num_blocks + self.num_sink_blocks,
                ),
                device=device,
                dtype=torch.int32,
            )
            self.block_table_with_sink[:, : self.num_sink_blocks] = torch.arange(
                1,
                self.num_sink_blocks + 1,
                device=device,
                dtype=torch.int32,
            )

        def build(
            self,
            common_prefix_len: int,
            common_attn_metadata: CommonAttentionMetadata,
# ... omitted for brevity ...

    return attn_backend
```
**EN:** This function defines `create_static_sink_attention_backend`. It handles attention-specific state, metadata, or tensor transforms. The main inputs are `underlying_attn_backend`, `sink_len`. Key calls include `underlying_attn_backend.get_builder_cls`, `subclass_attention_backend`, `super.__init__`, `cdiv`, `torch.zeros`, `torch.arange`. It writes or updates `prefix`, `underlying_builder`, `attn_backend`, `model_config`, `scheduler_config`, `sink_len`.
**CN:** 该函数定义 `create_static_sink_attention_backend`。 它处理注意力相关的状态、元数据或张量变换。 其主要输入参数包括 `underlying_attn_backend`, `sink_len`。 关键调用包括 `underlying_attn_backend.get_builder_cls`, `subclass_attention_backend`, `super.__init__`, `cdiv`, `torch.zeros`, `torch.arange`。 它会写入或更新 `prefix`, `underlying_builder`, `attn_backend`, `model_config`, `scheduler_config`, `sink_len`。

### Lines 115-229 — class `StaticSinkAttention`
```python
@CustomOp.register("static_sink_attention")
class StaticSinkAttention(Attention, CustomOp):
    """
    Attention with static sink tokens
    """

    def __init__(
        self,
        num_heads: int,
        head_size: int,
        scale: float,
        sink_len: int,
        attn_backend: type[AttentionBackend] | None = None,
        cache_config: CacheConfig | None = None,
# ... omitted for brevity ...
            kv_quant_mode=get_kv_quant_mode(self.kv_cache_dtype),
        )
```
**EN:** This class defines `StaticSinkAttention`. It inherits from `Attention`, `CustomOp`. Attention with static sink tokens Important methods include `__init__`, `forward`, `forward_native`, `forward_cuda`, `update_sink_kv`, `populate_sink_kv`. Key calls include `CustomOp.register`, `torch.get_default_dtype`, `create_static_sink_attention_backend`, `Attention.__init__`, `CustomOp.__init__`, `super.forward`. It writes or updates `dtype`, `attn_backend`, `sink_len`, `sink_populated`, `sink_key`, `sink_value`. The body uses conditional branches to cover different runtime cases.
**CN:** 该类定义了 `StaticSinkAttention`。 它继承自 `Attention`, `CustomOp`。 该类的文档字符串说明了它在当前模块中的职责。 重要方法包括 `__init__`, `forward`, `forward_native`, `forward_cuda`, `update_sink_kv`, `populate_sink_kv`。 关键调用包括 `CustomOp.register`, `torch.get_default_dtype`, `create_static_sink_attention_backend`, `Attention.__init__`, `CustomOp.__init__`, `super.forward`。 它会写入或更新 `dtype`, `attn_backend`, `sink_len`, `sink_populated`, `sink_key`, `sink_value`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 121-160 — method `StaticSinkAttention.__init__`
```python
    def __init__(
        self,
        num_heads: int,
        head_size: int,
        scale: float,
        sink_len: int,
        attn_backend: type[AttentionBackend] | None = None,
        cache_config: CacheConfig | None = None,
        **kwargs,
    ):
        dtype = torch.get_default_dtype()

        if cache_config is not None:
            kv_cache_dtype = cache_config.cache_dtype
        else:
            kv_cache_dtype = "auto"

        if attn_backend is not None:
            underlying_attn_backend = attn_backend
        else:
            underlying_attn_backend = get_attn_backend(head_size, dtype, kv_cache_dtype)
        attn_backend = create_static_sink_attention_backend(
            underlying_attn_backend,  # type: ignore[arg-type]
            sink_len=sink_len,
        )
        Attention.__init__(
            self=self,
            num_heads=num_heads,
            head_size=head_size,
            scale=scale,
            cache_config=cache_config,
            attn_backend=attn_backend,
            **kwargs,
        )
        CustomOp.__init__(self)

        self.sink_len = sink_len
        self.sink_populated = False
        self.sink_key = None
        self.sink_value = None
```
**EN:** This method defines `__init__`. It stores configuration and binds the backend-specific implementation details. The main inputs are `num_heads`, `head_size`, `scale`, `sink_len`, `attn_backend`, `cache_config`. Key calls include `torch.get_default_dtype`, `create_static_sink_attention_backend`, `Attention.__init__`, `CustomOp.__init__`, `get_attn_backend`. It writes or updates `dtype`, `attn_backend`, `sink_len`, `sink_populated`, `sink_key`, `sink_value`. The body uses conditional branches to cover different runtime cases.
**CN:** 该方法定义 `__init__`。 它保存配置，并绑定具体后端实现所需的细节。 其主要输入参数包括 `num_heads`, `head_size`, `scale`, `sink_len`, `attn_backend`, `cache_config`。 关键调用包括 `torch.get_default_dtype`, `create_static_sink_attention_backend`, `Attention.__init__`, `CustomOp.__init__`, `get_attn_backend`。 它会写入或更新 `dtype`, `attn_backend`, `sink_len`, `sink_populated`, `sink_key`, `sink_value`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 193-194 — method `StaticSinkAttention.forward`
```python
    def forward(self, *args, **kwargs):
        return self._forward_method(*args, **kwargs)
```
**EN:** This method defines `forward`. It executes the main forward/runtime path for this component. The main inputs are `*args`, `**kwargs`. Key calls include `self._forward_method`.
**CN:** 该方法定义 `forward`。 它执行该组件的主前向/运行时路径。 其主要输入参数包括 `*args`, `**kwargs`。 关键调用包括 `self._forward_method`。

### Lines 166-182 — method `StaticSinkAttention.forward_native`
```python
    def forward_native(
        self,
        query: torch.Tensor,
        key: torch.Tensor,
        value: torch.Tensor,
        output_shape: torch.Size | None = None,
    ) -> torch.Tensor:
        assert self.sink_key is not None and self.sink_value is not None, (
            "sink_key and sink_value have not been prepared"
        )
        if not self.sink_populated:
            self_kv_cache = self.kv_cache
            torch.ops.vllm.maybe_populate_sink(
                self_kv_cache, _encode_layer_name(self.layer_name)
            )

        return super().forward(query, key, value, output_shape)
```
**EN:** This method defines `forward_native`. It executes the main forward/runtime path for this component. The main inputs are `query`, `key`, `value`, `output_shape`. Key calls include `super.forward`, `torch.ops.vllm.maybe_populate_sink`, `_encode_layer_name`, `super`. It writes or updates `self_kv_cache`. The body uses conditional branches to cover different runtime cases.
**CN:** 该方法定义 `forward_native`。 它执行该组件的主前向/运行时路径。 其主要输入参数包括 `query`, `key`, `value`, `output_shape`。 关键调用包括 `super.forward`, `torch.ops.vllm.maybe_populate_sink`, `_encode_layer_name`, `super`。 它会写入或更新 `self_kv_cache`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 184-191 — method `StaticSinkAttention.forward_cuda`
```python
    def forward_cuda(
        self,
        query: torch.Tensor,
        key: torch.Tensor,
        value: torch.Tensor,
        output_shape: torch.Size | None = None,
    ) -> torch.Tensor:
        return self.forward_native(query, key, value, output_shape)
```
**EN:** This method defines `forward_cuda`. It executes the main forward/runtime path for this component. The main inputs are `query`, `key`, `value`, `output_shape`. Key calls include `self.forward_native`.
**CN:** 该方法定义 `forward_cuda`。 它执行该组件的主前向/运行时路径。 其主要输入参数包括 `query`, `key`, `value`, `output_shape`。 关键调用包括 `self.forward_native`。

### Lines 162-164 — method `StaticSinkAttention.update_sink_kv`
```python
    def update_sink_kv(self, sink_key, sink_value) -> None:
        self.sink_key = sink_key
        self.sink_value = sink_value
```
**EN:** This method defines `update_sink_kv`. It handles attention-specific state, metadata, or tensor transforms. The main inputs are `sink_key`, `sink_value`. It writes or updates `sink_key`, `sink_value`.
**CN:** 该方法定义 `update_sink_kv`。 它处理注意力相关的状态、元数据或张量变换。 其主要输入参数包括 `sink_key`, `sink_value`。 它会写入或更新 `sink_key`, `sink_value`。

### Lines 196-213 — method `StaticSinkAttention.populate_sink_kv`
```python
    def populate_sink_kv(self, self_kv_cache):
        sink_kv_slot_mapping = torch.arange(
            self.block_size,
            self.sink_len + self.block_size,
            device=torch.accelerator.current_device_index(),
            dtype=torch.long,
        )
        triton_reshape_and_cache_flash_diffkv(
            self.sink_key,
            self.sink_value,
            self_kv_cache,
            sink_kv_slot_mapping,
            self.kv_cache_dtype,
            self._k_scale,
            self._v_scale,
        )
        # We only populate the sink_key and sink_value once
        self.sink_populated = True
```
**EN:** This method defines `populate_sink_kv`. It handles attention-specific state, metadata, or tensor transforms. The main inputs are `self_kv_cache`. Key calls include `torch.arange`, `triton_reshape_and_cache_flash_diffkv`, `torch.accelerator.current_device_index`. It writes or updates `sink_kv_slot_mapping`, `sink_populated`.
**CN:** 该方法定义 `populate_sink_kv`。 它处理注意力相关的状态、元数据或张量变换。 其主要输入参数包括 `self_kv_cache`。 关键调用包括 `torch.arange`, `triton_reshape_and_cache_flash_diffkv`, `torch.accelerator.current_device_index`。 它会写入或更新 `sink_kv_slot_mapping`, `sink_populated`。

### Lines 232-241 — function `maybe_populate_sink`
```python
def maybe_populate_sink(
    self_kv_cache: torch.Tensor,
    layer_name: LayerNameType,
) -> None:
    layer_name = _resolve_layer_name(layer_name)
    forward_context: ForwardContext = get_forward_context()
    self = forward_context.no_compile_layers[layer_name]
    if self.sink_populated or self_kv_cache.numel() == 0:
        return
    self.populate_sink_kv(self_kv_cache)
```
**EN:** This function defines `maybe_populate_sink`. It handles attention-specific state, metadata, or tensor transforms. The main inputs are `self_kv_cache`, `layer_name`. Key calls include `_resolve_layer_name`, `get_forward_context`, `self.populate_sink_kv`, `self_kv_cache.numel`. It writes or updates `layer_name`, `forward_context`, `self`. The body uses conditional branches to cover different runtime cases.
**CN:** 该函数定义 `maybe_populate_sink`。 它处理注意力相关的状态、元数据或张量变换。 其主要输入参数包括 `self_kv_cache`, `layer_name`。 关键调用包括 `_resolve_layer_name`, `get_forward_context`, `self.populate_sink_kv`, `self_kv_cache.numel`。 它会写入或更新 `layer_name`, `forward_context`, `self`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 244-248 — function `maybe_populate_sink_fake`
```python
def maybe_populate_sink_fake(
    self_kv_cache: torch.Tensor,
    layer_name: LayerNameType,
) -> None:
    return
```
**EN:** This function defines `maybe_populate_sink_fake`. It handles attention-specific state, metadata, or tensor transforms. The main inputs are `self_kv_cache`, `layer_name`.
**CN:** 该函数定义 `maybe_populate_sink_fake`。 它处理注意力相关的状态、元数据或张量变换。 其主要输入参数包括 `self_kv_cache`, `layer_name`。

## Key Concepts / 关键概念
- [EN] Attention execution, backend dispatch, and kv-cache handling / [CN] 注意力执行、后端分派与 KV 缓存处理
- [EN] PyTorch tensor computation / [CN] PyTorch 张量计算
- [EN] Triton kernel specialization / [CN] Triton 内核特化
- [EN] Core symbols: `create_static_sink_attention_backend`, `StaticSinkAttention`, `maybe_populate_sink`, `maybe_populate_sink_fake` / [CN] 核心符号：`create_static_sink_attention_backend`, `StaticSinkAttention`, `maybe_populate_sink`, `maybe_populate_sink_fake`

## Dependencies / 依赖关系
- **External**: `functools`, `torch` / **外部依赖**: `functools`, `torch`
- **Internal**: `vllm.config`, `vllm.forward_context`, `vllm.logger`, `vllm.model_executor.custom_op`, `vllm.model_executor.layers.attention`, `vllm.utils.math_utils`, `vllm.utils.torch_utils`, `vllm.v1.attention.backend`, `vllm.v1.attention.ops.triton_reshape_and_cache_flash`, `vllm.v1.attention.selector` / **内部依赖**: `vllm.config`, `vllm.forward_context`, `vllm.logger`, `vllm.model_executor.custom_op`, `vllm.model_executor.layers.attention`, `vllm.utils.math_utils`, `vllm.utils.torch_utils`, `vllm.v1.attention.backend`, `vllm.v1.attention.ops.triton_reshape_and_cache_flash`, `vllm.v1.attention.selector`
- **Runtime traits**: Triton kernels / **运行时特征**: Triton kernels
