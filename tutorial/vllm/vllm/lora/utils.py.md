# utils.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/lora/utils.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Collects helper functions for LoRA wrapper selection, module filtering, name parsing, adapter path resolution, and MoE packed-module mapping. / 汇总 LoRA wrapper 选择、模块过滤、名称解析、适配器路径解析以及 MoE 打包模块映射等辅助函数。

## Line-by-Line Analysis / 逐行分析
### Capture counts and global adapter IDs
```python
def get_captured_lora_counts(max_loras: int, specialize: bool) -> list[int]:
    if not specialize:
        return [max_loras + 1]
    return [
        n for n in range(1, max_loras + 2)
        if (n & (n - 1)) == 0 or n == max_loras + 1
    ]
```
**EN:** This helper defines which active-LoRA counts should get separate cudagraph captures. The same file also owns `_GLOBAL_LORA_ID` and `get_lora_id()`, making it the central source of monotonically increasing adapter IDs.
**CN:** 该函数定义了哪些“活动 LoRA 数量”需要单独做 cudagraph capture。同一文件还维护 `_GLOBAL_LORA_ID` 与 `get_lora_id()`，因此这里也是全局递增适配器 ID 的统一来源。

### Choosing LoRA wrapper classes
```python
for lora_cls in _all_lora_classes:
    if lora_cls.can_replace_layer(
        source_layer=layer,
        lora_config=lora_config,
        packed_modules_list=packed_modules_list,
        model_config=model_config,
    ):
        instance_layer = lora_cls(layer)
        instance_layer.create_lora_weights(max_loras, lora_config, model_config)
        return instance_layer
```
**EN:** `_all_lora_classes` is ordered deliberately so specialized wrappers win before generic ones. `from_layer` and `from_layer_logits_processor` wrap compatible modules and allocate their LoRA buffers, while `replace_submodule` swaps them back into the model tree.
**CN:** `_all_lora_classes` 的顺序是刻意设计的：更专用的 wrapper 要先于通用 wrapper 命中。`from_layer` 与 `from_layer_logits_processor` 会为兼容模块创建 LoRA 包装层并分配缓冲区，而 `replace_submodule` 则把它们替换回模型树中。

### Parsing PEFT tensor names
```python
if name.startswith("base_model.model."):
    name = name.replace("base_model.model.", "")
    name = weights_mapper._map_name(name) if weights_mapper else name
    name = "base_model.model." + name

parts = name.split(".")
if parts[-1] == "weight" and (parts[-2] == "lora_A" or parts[-2] == "lora_B"):
    new_name = ".".join(parts[start_index:-2])
    return new_name, parts[-2] == "lora_A"
```
**EN:** PEFT checkpoints use naming conventions such as `base_model.model.*.lora_A.weight`. This parser strips or remaps prefixes, supports embedding-specific suffixes, and returns both the normalized runtime module name and whether the tensor is the A or B matrix.
**CN:** PEFT 检查点通常使用 `base_model.model.*.lora_A.weight` 这类命名。该解析器会剥离或重映射前缀，兼容 embedding 专用后缀，并同时返回规范化后的运行时模块名以及当前张量是 A 还是 B 矩阵。

### Discovering and filtering target modules
```python
for name, module in model.named_modules():
    embedding_modules = getattr(module, "embedding_modules", None)
    if embedding_modules is not None:
        for name in embedding_modules:
            supported_lora_modules.add(name)
    if isinstance(module, (LinearBase,)):
        supported_lora_modules.add(name.split(".")[-1])
    if isinstance(module, (FusedMoE,)):
        supported_lora_modules.add(name.split(".")[-1])
```
**EN:** These helpers separate two concerns: what the model implementation can support, and what deployment configuration wants to enable. `is_supported_lora_module` does suffix-based matching, while `is_in_target_modules` also handles packed-parent/packed-child equivalence such as `gate_up_proj` versus `gate_proj` and `up_proj`.
**CN:** 这些辅助函数把两个问题拆开：模型实现本身支持哪些模块，以及部署配置想启用哪些模块。`is_supported_lora_module` 负责基于后缀匹配；`is_in_target_modules` 还会处理 packed 父子模块的等价关系，例如 `gate_up_proj` 与 `gate_proj`、`up_proj` 之间的映射。

### Resolving adapter paths
```python
if os.path.isabs(lora_path):
    return lora_path
if lora_path.startswith("~"):
    return os.path.expanduser(lora_path)
if os.path.exists(lora_path):
    return os.path.abspath(lora_path)
if envs.VLLM_USE_MODELSCOPE:
    download_fn = lambda: snapshot_download(model_id=lora_path)
else:
    download_fn = lambda: huggingface_hub.snapshot_download(repo_id=lora_path)
```
**EN:** A LoRA path may already be local, may need `~` expansion, or may actually be a remote model identifier. This function resolves all three cases and falls back gracefully by returning the original string if remote download fails.
**CN:** LoRA 路径可能已经是本地绝对路径，也可能需要展开 `~`，或者实际上是远程模型标识符。该函数统一处理这三种情况；若远程下载失败，则优雅回退，直接返回原始字符串。

### Building packed-module mappings for MoE
```python
if is_moe_model(model):
    if moe_packed_mapping := get_moe_expert_mapping(model):
        packed_modules_mapping = get_packed_modules_mapping(model)
        if (not model.is_3d_moe_weight) or force_2d_moe:
            packed_modules_mapping["experts"] = [
                weight_name.rstrip(".")
                for _, weight_name, _, _ in moe_packed_mapping
                if ".." not in weight_name
            ]
        return packed_modules_mapping
```
**EN:** MoE models need extra mapping logic because runtime modules may be fused while checkpoints expose per-expert names. This function merges static packed-module mappings with dynamic expert mappings, and it can force the 2D view even for 3D-capable models when mixed-format LoRA loading is enabled.
**CN:** MoE 模型需要额外的映射逻辑，因为运行时模块往往是融合的，而检查点却暴露逐 expert 的名字。该函数会把静态 packed 模块映射与动态 expert 映射合并；当启用混合格式 LoRA 加载时，即使模型本身支持 3D，也可以强制构造 2D 视图。

## Key Concepts / 关键概念
- Centralized wrapper selection / 集中式 wrapper 选择
- PEFT-to-runtime name normalization / PEFT 到运行时的名称规范化
- Support filtering vs deployment filtering / 模型支持过滤与部署过滤分离
- Local-or-remote adapter path resolution / 本地或远程适配器路径解析
- MoE packed-module translation / MoE 打包模块映射转换

## Dependencies / 依赖关系
- LoRA wrapper classes from `vllm.lora.layers` / 依赖 `vllm.lora.layers` 中的各类 LoRA wrapper
- `LinearBase` and `FusedMoE` identify eligible modules / `LinearBase` 与 `FusedMoE` 用于识别可适配模块
- `huggingface_hub` and optional ModelScope resolve remote adapters / `huggingface_hub` 与可选的 ModelScope 用于解析远程适配器
- `get_moe_expert_mapping` and `get_packed_modules_mapping` drive packed-name expansion / `get_moe_expert_mapping` 与 `get_packed_modules_mapping` 驱动打包名称展开
