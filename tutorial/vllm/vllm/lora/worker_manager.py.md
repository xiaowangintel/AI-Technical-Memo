# worker_manager.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/lora/worker_manager.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Coordinates request-time LoRA loading, dummy warmup adapters, activation updates, and cache policy on each worker. / 在每个 worker 上协调请求时的 LoRA 加载、dummy warmup 适配器、激活更新以及缓存策略。

## Line-by-Line Analysis / 逐行分析
### Worker-side configuration bootstrap
```python
class WorkerLoRAManager:
    def __init__(self, vllm_config, device, embedding_modules, lora_model_cls=LoRAModel):
        self.embedding_modules = embedding_modules
        self.max_num_seqs = vllm_config.scheduler_config.max_num_seqs
        self.max_num_batched_tokens = vllm_config.scheduler_config.max_num_batched_tokens
        self.vocab_size = vllm_config.model_config.get_vocab_size()
        self.lora_config = vllm_config.lora_config
```
**EN:** The worker manager extracts the scheduling and model limits needed by the underlying `LoRAModelManager`. It also computes `max_position_embeddings` from the text config, with a special case for encoder-decoder models such as Whisper.
**CN:** Worker 管理器会提取底层 `LoRAModelManager` 所需的调度与模型限制参数。同时它还会从文本配置中计算 `max_position_embeddings`；对于 Whisper 这类 encoder-decoder 模型，则会走专门分支。

### Creating the model-side manager
```python
lora_manager = create_lora_manager(
    model,
    max_num_seqs=self.max_num_seqs,
    max_num_batched_tokens=self.max_num_batched_tokens,
    vocab_size=self.vocab_size,
    lora_config=self.lora_config,
    device=self.device,
    lora_manager_cls=self._manager_cls,
    vllm_config=vllm_config,
)
```
**EN:** This method is the bridge between the worker and the rewritten runtime model. It instantiates either the plain manager or the LRU variant, stores it, and returns the wrapped model object for execution.
**CN:** 该方法是 worker 与“已重写的运行时模型”之间的桥梁。它会实例化普通管理器或 LRU 变体，保存该实例，并返回包装后的模型对象供执行阶段使用。

### Loading and validating one adapter
```python
lora_path = get_adapter_absolute_path(lora_request.lora_path)

peft_helper = PEFTHelper.from_local_dir(
    lora_path,
    self.max_position_embeddings,
    lora_request.tensorizer_config_dict,
)
peft_helper.validate_legal(self.lora_config)

lora = self._lora_model_cls.from_local_checkpoint(
    lora_path,
    expected_lora_modules,
    peft_helper=peft_helper,
    lora_model_id=lora_request.lora_int_id,
)
```
**EN:** `_load_adapter` first builds the expected module-name set from the current model, including packed-module expansions and the MoE `experts` alias. It resolves the path, reads PEFT config, validates legality, loads the checkpoint on CPU, and stamps the request's 3D/2D MoE hint onto the resulting `LoRAModel`.
**CN:** `_load_adapter` 会先根据当前模型构造“期望模块名集合”，其中包括 packed 模块展开结果以及 MoE 的 `experts` 别名。随后它解析路径、读取 PEFT 配置、验证合法性、在 CPU 上加载检查点，并把请求里携带的 3D/2D MoE 提示写回生成的 `LoRAModel`。

### Dummy adapter reuse
```python
@contextmanager
def dummy_lora_cache(self):
    self._cached_dummy_lora = None
    yield
    self._cached_dummy_lora = False
```
**EN:** Warmup may request many temporary adapters with identical shapes. This context manager lets the worker create one dummy LoRA, cache it temporarily, and clone it for additional IDs instead of rebuilding zero tensors every time.
**CN:** Warmup 阶段可能会请求许多形状完全相同的临时适配器。该上下文管理器允许 worker 先创建一个 dummy LoRA，短暂缓存后再为其他 ID 克隆它，而不必反复重建全零张量。

### Applying request-time adapter sets
```python
existing_adapters = self.list_adapters()
models_map = {
    adapter_request.adapter_id: adapter_request
    for adapter_request in adapter_requests
    if adapter_request
}
requested_ids = set(models_map)
for adapter_id in existing_adapters - requested_ids:
    self.remove_adapter(adapter_id)
for adapter_id in requested_ids - existing_adapters:
    self.add_adapter(models_map[adapter_id])
```
**EN:** In the default worker manager, the active adapter set is treated as exact: adapters not requested anymore are removed, and missing ones are loaded. `set_active_adapters` then optionally pushes mapping metadata into the underlying model manager.
**CN:** 在默认 worker 管理器中，活动适配器集合被视为“精确集合”：不再被请求的适配器会被移除，缺失的适配器会被加载。`set_active_adapters` 随后还会在需要时把映射元数据下发给底层模型管理器。

### Add/remove operations
```python
def add_adapter(self, adapter_request: Any) -> bool:
    if adapter_request.adapter_id in self.list_adapters():
        return False
    loaded_adapter = self._load_adapter(adapter_request)
    loaded = self._adapter_manager.add_adapter(loaded_adapter)
    self._adapter_manager.activate_adapter(loaded_adapter.id)
    return loaded
```
**EN:** Adding an adapter is a two-step operation: load it into the manager's CPU-side registry, then activate it into GPU-visible runtime slots. Removal delegates to the underlying manager, which also handles slot cleanup.
**CN:** 添加适配器分两步：先把它加载到管理器的 CPU 侧注册表，再激活到 GPU 可见的运行时槽位。移除操作则委托给底层管理器，由后者负责清理对应槽位。

### LRU worker specialization
```python
if (
    lora_request.lora_int_id not in self.list_adapters()
    or lora_request.load_inplace
):
    lora = self._load_adapter(lora_request)
    self._adapter_manager.remove_adapter(lora.id)
    if len(self._adapter_manager) + 1 > self._adapter_manager.capacity:
        self._adapter_manager.remove_oldest_adapter()
    loaded = self._adapter_manager.add_adapter(lora)
```
**EN:** The LRU variant never tries to keep the requested set exact. Instead it incrementally loads requested adapters, optionally forces reload with `load_inplace`, evicts the oldest cached adapter when CPU capacity would overflow, and always activates the requested adapter at the end.
**CN:** LRU 变体不会强求“请求集合与缓存集合完全一致”。它采用增量加载策略：对被请求的适配器逐个加载，必要时通过 `load_inplace` 强制重载；如果 CPU 缓存将要超限，就驱逐最旧的适配器；最后始终激活当前请求的适配器。

## Key Concepts / 关键概念
- Worker-side adapter orchestration / worker 侧适配器编排
- PEFT config validation before load / 加载前进行 PEFT 配置校验
- Dummy adapter caching for warmup / 用于 warmup 的 dummy 适配器缓存
- Exact-set policy vs LRU policy / 精确集合策略与 LRU 策略
- CPU registry to GPU activation flow / 从 CPU 注册到 GPU 激活的流程

## Dependencies / 依赖关系
- `LoRAModelManager` performs actual registration and activation / `LoRAModelManager` 负责实际注册与激活
- `PEFTHelper` validates adapter configs / `PEFTHelper` 负责校验适配器配置
- `LoRARequest` carries adapter metadata from the request path / `LoRARequest` 携带请求路径上的适配器元数据
- `get_adapter_absolute_path` resolves local and remote adapter paths / `get_adapter_absolute_path` 负责解析本地和远程适配器路径
