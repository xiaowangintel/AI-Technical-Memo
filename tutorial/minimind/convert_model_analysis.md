# convert_model.py 代码分析

## 1. 文件概述 (File Overview)
`convert_model.py` 是 MiniMind 的模型格式转换中枢。它负责把训练产出的 **MiniMind 原生 `.pth` checkpoint** 转成两类部署友好格式：
1. **MiniMind 自定义 Transformers format**；
2. **Qwen3 / Qwen3Moe-compatible Transformers format**。
此外，它还支持 **LoRA merge** 和 `chat_template` 的 **jinja ↔ json** 转换。因此它处在“训练完成 → 产物适配 → 部署分发”的关键节点。

## 2. 依赖说明 (Dependencies)
- `os`, `sys`：修正导入路径、拼接文件路径。
- `json`：修改 `config.json`、`tokenizer_config.json`、模板转换。
- `torch`：读写 checkpoint、张量堆叠、dtype 转换。
- `transformers`：读取版本号，并保存 Transformers 模型目录。
- `AutoTokenizer`, `AutoModelForCausalLM`：通用 tokenizer/model 读写。
- `Qwen3Config`, `Qwen3ForCausalLM`, `Qwen3MoeConfig`, `Qwen3MoeForCausalLM`：构造兼容 Qwen3 生态的模型结构。
- `MiniMindConfig`, `MiniMindForCausalLM`：MiniMind 原生结构。
- `apply_lora`, `merge_lora`：LoRA 注入与合并。

## 3. 逐行代码分析 (Line-by-Line Analysis)
- **第 1-14 行**：导入模块，设置 `__package__ = "scripts"` 并追加上级目录到 `sys.path`，保证脚本可直接运行；随后关闭 `UserWarning`。

### 3.1 `convert_torch2transformers_minimind()`
- **第 16 行**：定义“原生 `.pth` → MiniMind Transformers format”转换函数。
- **第 17-18 行**：把 `MiniMindConfig` 与 `MiniMindForCausalLM` 注册到 AutoClass，让保存后的目录可被 Hugging Face 自动识别。
- **第 19 行**：基于全局 `lm_config` 实例化空模型。
- **第 20-22 行**：选择 device、加载 `state_dict`、并以 `strict=False` 写入模型。
- **第 23 行**：把模型参数转换为指定 `dtype`，默认 `float16`。
- **第 24-25 行**：统计参数量并打印，便于核查规模。
- **第 26 行**：调用 `save_pretrained(transformers_path, safe_serialization=False)` 保存模型。
- **第 27-28 行**：从 `../model/` 复制 tokenizer 到输出目录。
- **第 29-35 行**：针对 `transformers>=5` 做兼容修补：
  - 给 `tokenizer_config.json` 补 `tokenizer_class` 与 `extra_special_tokens`；
  - 给 `config.json` 补 `rope_theta`、清理 `rope_scaling`，并删除 `rope_parameters`。
- **第 36 行**：打印保存成功信息。

### 3.2 `convert_torch2transformers()`
- **第 39 行**：注释说明此函数面向 Qwen/Llama 兼容生态，而不是 MiniMind 自定义类。
- **第 40-42 行**：定义函数并读取原生 `state_dict`。
- **第 43-55 行**：构建 `common_config`，把 MiniMind 的核心超参数映射为 Qwen3 所需配置字段。
- **第 56-63 行**：如果不是 MoE，构造 `Qwen3Config` 与 `Qwen3ForCausalLM`，并关闭 sliding window。
- **第 64-71 行**：如果是 MoE，改用 `Qwen3MoeConfig` 与 `Qwen3MoeForCausalLM`，同时注入 `num_experts`、`num_experts_per_tok`、`moe_intermediate_size`、`norm_topk_prob`。
- **第 72-79 行**：对 `transformers>=5` 的 MoE 权重做结构重排：
  - 过滤不需要直接搬运的 expert 子权重；
  - 遍历每一层，把各 expert 的 `gate_proj` 与 `up_proj` 堆叠后拼成 `gate_up_proj`；
  - 把各 expert 的 `down_proj` 堆叠成统一 tensor；
  - 最终生成符合 `Qwen3MoeForCausalLM` 预期的 `state_dict`。
- **第 81 行**：`strict=True` 加载映射结果，保证结构严格匹配。
- **第 82-83 行**：转 dtype 并保存为标准 Transformers 目录。
- **第 84-85 行**：再次打印参数量。
- **第 86-87 行**：保存 tokenizer。
- **第 89-95 行**：再次对 `transformers>=5` 应用 tokenizer/config 兼容补丁。
- **第 96 行**：打印完成信息。

### 3.3 反向转换与辅助工具
- **第 99-102 行**：`convert_transformers2torch()` 负责把 Transformers 目录重新打包成原生 `.pth`，其中所有权重先 `cpu().half()`，便于本地原生加载。
- **第 105-112 行**：`convert_merge_base_lora()`：加载 base MiniMind 模型、注入 LoRA、调用 `merge_lora(...)`，把 adapter 真正合并进基座权重。
- **第 115-118 行**：`convert_jinja_to_json()`：读取 `.jinja` 模板，用 `json.dumps` 做转义，然后以 JSON 字段文本形式打印出来。
- **第 121-125 行**：`convert_json_to_jinja()`：从 JSON 文件中读取 `chat_template` 字段，并写回 `.jinja` 文件。

### 3.4 脚本入口
- **第 128-129 行**：初始化全局 `lm_config`，默认 `hidden_size=768`、`num_hidden_layers=8`、`max_seq_len=8192`、`use_moe=False`。
- **第 131-134 行**：默认示例是把 `../out/full_sft_768.pth` 转成 `../minimind-3` Transformers 目录，并直接调用 `convert_torch2transformers(...)`。
- **第 136-140 行**：注释掉的 LoRA merge 示例。
- **第 142-144 行**：注释掉的反向转换与模板转换示例。
- **第 145 行**：文件结束。

## 4. 关键算法解析 (Key Algorithm Deep-Dive)
### 4.1 Torch ↔ Transformers conversion
核心思想是：
- **native MiniMind**：保留自定义类，直接 `save_pretrained()`；
- **compatible Transformers**：不保留自定义类，而是把配置字段和权重键尽量映射到主流生态结构。
这样既能保持原生可用性，也能提升部署兼容性。

### 4.2 Qwen3 / Qwen3Moe 兼容映射
`convert_torch2transformers()` 通过 `common_config` 把 MiniMind 超参数映射到 Qwen3 所需字段；在 MoE 情况下，还需要把离散 expert 权重重排成 `gate_up_proj/down_proj` 形式，这是最关键的结构适配步骤。

### 4.3 LoRA merge
`apply_lora()` 先让模型具备 LoRA 分支结构，`merge_lora()` 再把低秩增量权重折叠进原始权重矩阵。合并后的 checkpoint 推理时不再依赖单独 adapter 文件，更适合部署。

### 4.4 chat_template：jinja ↔ json
- `jinja -> json`：便于把模板嵌入 `tokenizer_config.json`；
- `json -> jinja`：便于人工编辑与版本管理。
这是 tokenizer 部署中很常见的模板双向转换。

## 5. 调用关系 (Call Graph)
```text
__main__
 ├─ lm_config = MiniMindConfig(...)
 ├─ convert_torch2transformers(torch_path, transformers_path)
 │   ├─ torch.load()
 │   ├─ Qwen3Config / Qwen3MoeConfig
 │   ├─ Qwen3ForCausalLM / Qwen3MoeForCausalLM
 │   ├─ qwen_model.load_state_dict()
 │   ├─ qwen_model.save_pretrained()
 │   └─ AutoTokenizer.from_pretrained().save_pretrained()
 ├─ convert_torch2transformers_minimind(...)      [可选]
 ├─ convert_transformers2torch(...)               [可选]
 ├─ convert_merge_base_lora(...)                  [可选]
 ├─ convert_jinja_to_json(...)                    [可选]
 └─ convert_json_to_jinja(...)                    [可选]
```
这里模型与 tokenizer 的连接点主要是 `save_pretrained()`、`from_pretrained()` 和 `chat_template` 文件；与部署最相关的是输出目录中的 `config.json`、`tokenizer_config.json` 与模型权重文件。
