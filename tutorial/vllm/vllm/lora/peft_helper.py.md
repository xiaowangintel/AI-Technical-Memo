# peft_helper.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/lora/peft_helper.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Loads PEFT LoRA configuration, derives scaling values, and enforces what vLLM's runtime can legally accept. / 加载 PEFT 的 LoRA 配置，推导缩放值，并约束 vLLM 运行时可合法接受的配置。

## Line-by-Line Analysis / 逐行分析
### Dataclass schema
```python
@dataclass
class PEFTHelper:
    r: int
    lora_alpha: int
    target_modules: list[str] | str
    bias: Literal["none"] = field(default="none")
    modules_to_save: list[str] | None = field(default=None)
    use_rslora: bool = field(default=False)
    use_dora: bool = field(default=False)
```
**EN:** `PEFTHelper` mirrors the important fields from `adapter_config.json` and adds vLLM-specific fields prefixed with `vllm_`. This keeps the runtime loosely compatible with PEFT while avoiding namespace clashes.
**CN:** `PEFTHelper` 映射了 `adapter_config.json` 中的重要字段，并增加了以 `vllm_` 开头的 vLLM 专用字段。这样既能保持与 PEFT 的兼容，又能避免命名冲突。

### Unsupported-feature checks and scaling
```python
def _validate_features(self) -> list[str]:
    if self.modules_to_save:
        error_msg.append("vLLM only supports modules_to_save being None.")
    if self.use_dora:
        error_msg.append("vLLM does not yet support DoRA.")

def __post_init__(self):
    if self.use_rslora:
        self.vllm_lora_scaling_factor = self.lora_alpha / math.sqrt(self.r)
    else:
        self.vllm_lora_scaling_factor = self.lora_alpha / self.r
```
**EN:** The helper rejects PEFT features that the runtime cannot implement, especially `modules_to_save` and DoRA. It also computes the effective scaling factor, switching to the rsLoRA formula `alpha / sqrt(r)` when required.
**CN:** 该辅助类会拒绝运行时尚无法实现的 PEFT 特性，尤其是 `modules_to_save` 和 DoRA。同时它会计算实际缩放因子；如果启用了 rsLoRA，则改用 `alpha / sqrt(r)` 公式。

### Parsing from dictionaries and local directories
```python
class_fields = {f.name: f for f in fields(cls)}
required_fields = {
    name
    for name, f in class_fields.items()
    if f.default is MISSING and f.default_factory is MISSING
}
missing_fields = required_fields - set(config_dict.keys())
```
**EN:** `from_dict` validates required fields and drops unknown keys instead of failing on every PEFT extension field. `from_local_dir` loads `adapter_config.json` either from a plain directory or through tensorizer stream I/O, then injects `vllm_max_position_embeddings` before constructing the helper.
**CN:** `from_dict` 会检查必需字段，并丢弃未知键，而不是对所有 PEFT 扩展字段都报错。`from_local_dir` 则支持从普通目录或 tensorizer 流式 I/O 中读取 `adapter_config.json`，并在构造对象前注入 `vllm_max_position_embeddings`。

### Runtime legality check
```python
if self.r > lora_config.max_lora_rank:
    error_msg.append(
        f"LoRA rank {self.r} is greater than max_lora_rank"
        f" {lora_config.max_lora_rank}."
    )
if self.bias != "none":
    error_msg.append("Adapter bias is not supported.")
if error_msg:
    raise ValueError(f"{' '.join(error_msg)}")
```
**EN:** This is the final gate before adapter weights are loaded. It combines PEFT-level incompatibilities with deployment-time limits such as `max_lora_rank`, ensuring invalid adapters fail fast.
**CN:** 这是加载适配器权重前的最后一道关卡。它把 PEFT 层面的不兼容项与部署时限制（如 `max_lora_rank`）合并检查，确保非法适配器能够尽早失败。

## Key Concepts / 关键概念
- PEFT config mirroring / 映射 PEFT 配置
- rsLoRA scaling support / 支持 rsLoRA 缩放
- Feature gating for unsupported options / 对未支持特性进行门控
- Tensorizer-aware config loading / 感知 tensorizer 的配置加载

## Dependencies / 依赖关系
- `LoRAConfig` provides runtime limits / `LoRAConfig` 提供运行时限制
- `TensorizerConfig` supports streamed config loading / `TensorizerConfig` 支持流式配置加载
- `json`, `dataclasses`, and `math` power parsing and scaling / `json`、`dataclasses` 与 `math` 负责解析和缩放计算
