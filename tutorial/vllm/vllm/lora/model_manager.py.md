# model_manager.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/lora/model_manager.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Manages LoRA wrapper creation, adapter registration, activation slots, MoE format conversion, and optional LRU caching for a live model instance. / 管理运行中模型实例的 LoRA wrapper 创建、适配器注册、激活槽位、MoE 格式转换以及可选的 LRU 缓存。

## Line-by-Line Analysis / 逐行分析
### Cache helper and manager bootstrap
```python
self.supported_lora_modules = get_supported_lora_modules(self.model)
assert self.supported_lora_modules, (
    f"No supported LoRA modules found in {self.model.__class__.__name__}."
)
self._registered_adapters = {}
self._active_adapters = {}
self.packed_modules_mapping = process_packed_modules_mapping(
    self.model, force_2d_moe=self._enable_mixed_moe_lora_format
)
self._init_punica_wrapper(max_num_batched_tokens, vllm_config)
self._create_lora_modules()
```
**EN:** The file starts by discovering supported target modules, initializing CPU/GPU-side adapter bookkeeping, computing packed-module mappings, and then rewriting the model with LoRA-aware wrappers. The nearby `AdapterLRUCache` class extends generic LRU eviction with adapter deactivation hooks.
**CN:** 文件一开始会发现模型支持的目标模块、初始化 CPU/GPU 两侧的适配器管理结构、计算 packed 模块映射，然后用支持 LoRA 的 wrapper 重写模型。邻近定义的 `AdapterLRUCache` 则在通用 LRU 的基础上增加了“驱逐时自动停用适配器”的钩子。

### Punica wrapper setup for text and multimodal models
```python
self.supports_mm = (
    supports_multimodal(self.model)
    and hasattr(self.model, "get_mm_mapping")
)
if self.supports_mm:
    self._maybe_init_mm(vllm_config, max_num_batched_tokens)
else:
    llm_punica_wrapper = get_punica_wrapper(
        max_num_batched_tokens,
        max_batches=self.max_num_seqs,
        device=self.device,
        lora_config=self.lora_config,
    )
```
**EN:** Every wrapped LoRA layer needs a Punica wrapper that owns runtime metadata and GPU buffers. For ordinary language models there is a single wrapper; multimodal models may get distinct wrappers for the language model, image tower, and connector, depending on config flags and token-budget helpers.
**CN:** 每个带 LoRA 的包装层都需要一个 Punica wrapper 来维护运行时元数据和 GPU 缓冲区。普通语言模型只需要一个 wrapper；多模态模型则可能根据配置和 token 预算，为语言模型、视觉塔以及连接器分别创建不同的 wrapper。

### Tower/connector LoRA policy
```python
if not self.lora_config.enable_tower_connector_lora:
    self.supports_tower_connector_lora = False
    return

if (
    vllm_config.model_config.multimodal_config
    and vllm_config.model_config.multimodal_config.language_model_only
):
    self.supports_tower_connector_lora = False
    return

mm_budget = MultiModalBudget(vllm_config, mm_registry)
num_encoder_tokens = self.model.get_num_mm_encoder_tokens(
    mm_budget.get_encoder_budget()
)
```
**EN:** `_maybe_init_mm` carefully gates experimental tower/connector LoRA support. It checks whether the feature is enabled, whether the model exposes the needed multimodal hooks, and whether the engine is running in language-model-only mode before allocating additional wrappers.
**CN:** `_maybe_init_mm` 会非常谨慎地控制实验性的 tower/connector LoRA 支持。它会检查功能是否开启、模型是否暴露所需多模态接口，以及引擎是否运行在仅语言模型模式下，然后才会分配额外 wrapper。

### Activating adapters and updating mapping metadata
```python
first_free_slot = next(
    ((i, lora_id) for i, lora_id in enumerate(self.lora_index_to_id) if lora_id is None),
    None,
)
self._active_adapters[lora_id] = None
self.lora_index_to_id[index] = lora_model.id
for module_name, module in self.modules.items():
    module_lora = self._get_lora_layer_weights(lora_model, module_name)
    if not module_lora:
        module.reset_lora(index)
        continue
    module.set_lora(index, module_lora.lora_a, module_lora.lora_b)
```
**EN:** Activation means copying one registered adapter into an available runtime slot across every wrapped module. If a module has no weights for that adapter, the slot is reset for that module. `_set_adapter_mapping` then pushes sequence-to-slot metadata into the correct Punica wrapper, choosing language, tower, or connector paths according to mapping type.
**CN:** 激活的含义是：把一个已注册适配器复制到所有已包装模块中的某个空闲运行时槽位。如果某个模块没有该适配器的权重，就对该模块的对应槽位执行 reset。之后 `_set_adapter_mapping` 会把序列到槽位的映射元数据推送进正确的 Punica wrapper，并根据映射类型选择语言、tower 或 connector 路径。

### Rewriting the model with LoRA-capable modules
```python
for module_name, module in self.model.named_modules(remove_duplicate=False):
    if isinstance(module, PPMissingLayer):
        continue
    if not self._match_target_modules(module_name):
        continue
    punica_wrapper = self._get_punica_wrapper(module_name)
    new_module = replace_submodule(
        self.model,
        module_name,
        from_layer(
            module,
            self.lora_slots,
            self.lora_config,
            packed_moduled_lst,
            self.model.config,
        ),
    )
```
**EN:** `_create_lora_modules` scans the model tree, skips unsupported or missing pipeline-parallel layers, and wraps only target modules. It also handles aliasing so the same underlying object is not wrapped twice, treats MoE layers specially to choose 2D vs 3D wrappers, and adds a matching LoRA-aware `logits_processor` when `lm_head` is wrapped.
**CN:** `_create_lora_modules` 会扫描整个模型树，跳过不支持或在流水线并行中缺失的层，只包装目标模块。它还会处理模块别名，避免同一个底层对象被重复包装；对 MoE 层会特别选择 2D 或 3D wrapper；当 `lm_head` 被包装时，也会同步替换对应的 `logits_processor`。

### Dummy adapters and warmup rank adjustment
```python
model = LoRAModel(lora_id, rank, {})
for i, r in enumerate(replacements):
    lora = LoRALayerWeights.create_dummy_lora_weights(
        module_name + "." + r,
        module.lora_a_stacked[i].shape[-1],
        module.lora_b_stacked[i].shape[-2],
        rank,
        module.lora_a_stacked[i].dtype,
        "cpu",
    )
    subloras.append(lora)
```
**EN:** Warmup does not need real adapter weights, so `create_dummy_lora` synthesizes zero-filled LoRAs that match the wrapped module shapes, including packed layers and MoE experts. `get_dummy_lora_warmup_rank` additionally raises the dummy rank to a tensor-parallel-compatible multiple when fully sharded MoE wrappers require it.
**CN:** Warmup 不需要真实的适配器权重，因此 `create_dummy_lora` 会按已包装模块的形状构造全零 LoRA，包括 packed 层和 MoE experts。`get_dummy_lora_warmup_rank` 还会在 fully-sharded 的 MoE wrapper 有要求时，把 dummy rank 调整到兼容 tensor parallel 的倍数。

### Target matching and packed-LoRA merging
```python
for module_name, new_module_names in self.packed_modules.items():
    replacement_loras = []
    for r in new_module_names:
        lora = self._get_lora_layer_weights(lora_model, r)
        replacement_loras.append(lora)
    if module_name.endswith(".experts"):
        lora_model.loras[module_name] = PackedLoRALayerWeights.pack_moe(
            replacement_loras,
            module_name,
            is_non_gated_moe=self._is_non_gated_moe,
        )
```
**EN:** Runtime wrappers often expect one packed weight object even if the checkpoint stored multiple child tensors. This method merges those child entries into packed LoRAs, removes the now-redundant originals, optimizes scaling, and finally pins memory if the adapter lives on CPU.
**CN:** 运行时 wrapper 往往期望得到一个打包后的权重对象，而检查点可能存的是多个子张量。该方法会把这些子条目合并成 packed LoRA，删除冗余的原始条目，执行缩放优化，并在适配器位于 CPU 时最终进行 pin memory。

### MoE slicing and 3D-to-2D conversion
```python
if self._enable_mixed_moe_lora_format and getattr(
    lora_model, "is_3d_lora_weight", False
):
    self._convert_3d_to_2d_moe_lora(lora_model, module, module_name)
else:
    self._slice_moe_lora_ep(lora_model, module, module_name)
```
**EN:** The most specialized part of the manager is its MoE handling. `_stack_moe_lora_weights` reshapes fused 3D checkpoints into per-expert tensors, `_convert_3d_to_2d_moe_lora` adapts 3D files for the universal 2D wrapper used in mixed mode, and `_slice_moe_lora_ep` trims cached tensors to the current expert-parallel shard.
**CN:** 管理器中最专门的逻辑是对 MoE 的处理。`_stack_moe_lora_weights` 会把融合的 3D 检查点重塑成逐 expert 张量；`_convert_3d_to_2d_moe_lora` 则在混合模式下把 3D 文件转换给通用 2D wrapper 使用；`_slice_moe_lora_ep` 会把缓存张量裁剪到当前 expert-parallel 分片。

### LRU manager variant and factory
```python
if lora.id not in self._registered_adapters:
    self._add_adapter(lora)
    was_added = True
else:
    self._registered_adapters.touch(lora.id)
    was_added = False
```
**EN:** The LRU subclass swaps plain dictionaries for eviction-aware caches on both CPU and GPU sides. It can evict the oldest registered adapter, auto-evict active slots when full, and pin adapters in both caches. `create_lora_manager` is the public factory that validates `SupportsLoRA` and instantiates the chosen manager class.
**CN:** LRU 子类会把普通字典替换为带驱逐能力的缓存，同时覆盖 CPU 与 GPU 两侧的管理逻辑。它可以驱逐最老的已注册适配器、在活动槽位满时自动淘汰旧项，并支持把适配器固定在两级缓存中。`create_lora_manager` 则是公开工厂函数，先验证模型是否实现 `SupportsLoRA`，再实例化指定的管理器类。

## Key Concepts / 关键概念
- Adapter registration vs activation slots / 适配器注册与激活槽位分离
- Model rewriting with LoRA wrappers / 通过 LoRA wrapper 改写模型
- Punica metadata routing / Punica 元数据路由
- Packed-layer and MoE adapter merging / Packed 层与 MoE 适配器合并
- Multimodal tower/connector support gating / 多模态 tower/connector 支持门控
- LRU eviction on CPU and GPU caches / CPU 与 GPU 缓存的 LRU 驱逐

## Dependencies / 依赖关系
- `LoRAModel`, `LoRALayerWeights`, and `PackedLoRALayerWeights` hold adapter state / `LoRAModel`、`LoRALayerWeights` 与 `PackedLoRALayerWeights` 保存适配器状态
- `BaseLayerWithLoRA` and concrete wrappers perform module replacement / `BaseLayerWithLoRA` 及具体 wrapper 负责模块替换
- Punica wrappers hold runtime buffer metadata / Punica wrapper 维护运行时缓冲区元数据
- `process_packed_modules_mapping` and related utils define packed/MoE naming / `process_packed_modules_mapping` 等辅助函数定义 packed/MoE 命名映射
- `LRUCache` powers eviction-aware manager variants / `LRUCache` 为带驱逐能力的管理器变体提供基础设施
