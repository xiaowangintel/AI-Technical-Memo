# eval_llm.py 代码分析

## 1. 文件概述 (File Overview)
`eval_llm.py` 是 MiniMind 的本地推理脚本，也是训练后最直接的验证入口。它支持加载 **MiniMind 原生 PyTorch 权重 (native checkpoint)** 与 **Transformers format** 模型，并提供自动测试、手动对话、流式输出、历史对话裁剪与速度统计。在部署链路里，它通常位于“训练完成 → 本地加载验证 → 对话体验检查”阶段。

## 2. 依赖说明 (Dependencies)
- `time`：统计 generation time 与 tokens/s。
- `argparse`：解析命令行参数。
- `random`：每轮对话随机设置 seed。
- `warnings`：关闭无关 warning。
- `torch`：权重加载、device 切换、half precision。
- `AutoTokenizer`：统一加载 tokenizer。
- `AutoModelForCausalLM`：加载 Transformers 模型。
- `TextStreamer`：边生成边打印 streaming text。
- `MiniMindConfig`, `MiniMindForCausalLM`：MiniMind 原生模型结构。
- `model_lora.*`：LoRA 注入与权重加载。
- `setup_seed`, `get_model_params`：随机种子与参数量统计。

## 3. 逐行代码分析 (Line-by-Line Analysis)
- **第 1-10 行**：导入标准库、Torch、Transformers、MiniMind 模型与 LoRA 工具，并通过 `warnings.filterwarnings('ignore')` 压制推理阶段的提示信息。
- **第 12 行**：定义 `init_model(args)`，专门负责模型与 tokenizer 初始化。
- **第 13 行**：`AutoTokenizer.from_pretrained(args.load_from)` 统一加载 tokenizer，不区分原生还是 Transformers 权重。
- **第 14 行**：`if 'model' in args.load_from` 用路径字符串判断是否走 MiniMind 原生权重分支，这是一个简单 heuristic。
- **第 15-20 行**：构造 `MiniMindConfig`，注入 `hidden_size`、`num_hidden_layers`、`use_moe`、`inference_rope_scaling` 等结构参数。
- **第 21 行**：根据 `use_moe` 决定是否拼接 `_moe` 后缀。
- **第 22 行**：生成原生 checkpoint 路径 `./{save_dir}/{weight}_{hidden_size}{moe_suffix}.pth`。
- **第 23 行**：用 `torch.load(..., map_location=args.device)` 读取权重，并 `strict=True` 加载到模型中。
- **第 24-26 行**：如果指定 `lora_weight`，先 `apply_lora(model)` 注入 LoRA 层，再 `load_lora(...)` 加载 adapter 权重。
- **第 27-28 行**：否则走 Transformers 分支，用 `AutoModelForCausalLM.from_pretrained(..., trust_remote_code=True)` 直接加载可部署模型。
- **第 29 行**：`get_model_params(model, model.config)` 输出参数量与配置摘要。
- **第 30 行**：返回 `model.half().eval().to(args.device), tokenizer`，即切到 `fp16 + eval mode + target device`。
- **第 32 行**：进入主函数 `main()`。
- **第 33-49 行**：定义 CLI 参数：
  - `load_from` 区分 native/Transformers。
  - `save_dir`、`weight`、`lora_weight` 控制权重来源。
  - `hidden_size`、`num_hidden_layers`、`use_moe` 对应原生结构。
  - `inference_rope_scaling` 用于 RoPE extrapolation。
  - `max_new_tokens`、`temperature`、`top_p` 控制生成行为。
  - `open_thinking` 控制 chat template 是否启用 thinking。
  - `historys` 控制保留历史轮数。
  - `show_speed` 与 `device` 控制展示和运行环境。
- **第 51-60 行**：内置 `prompts` 测试集，覆盖问答、解释、代码生成、常识与建议类场景。
- **第 62 行**：初始化 `conversation` 历史列表。
- **第 63 行**：调用 `init_model(args)` 加载模型与 tokenizer。
- **第 64 行**：提示用户选择自动测试或手动输入模式。
- **第 65 行**：创建 `TextStreamer`，设置 `skip_prompt=True` 与 `skip_special_tokens=True`，避免把 prompt 或特殊 token 直接打印出来。
- **第 67 行**：根据 `input_mode` 决定输入源：自动遍历 `prompts`，或持续从终端读取用户输入直到空字符串结束。
- **第 68 行**：开始主对话循环。
- **第 69 行**：每轮使用随机 seed 调 `setup_seed(...)`，既保持采样随机性，也能在单轮内稳定复现。
- **第 70 行**：自动测试模式下打印当前问题。
- **第 71 行**：如果启用历史，只保留最近 `args.historys` 条消息；否则清空历史，保证单轮推理。
- **第 72 行**：把当前用户消息 append 到 `conversation`。
- **第 73-76 行**：构造输入 prompt：
  - `pretrain` 权重：直接走 `bos_token + prompt`，更像纯 LM continuation；
  - 其他权重：走 `tokenizer.apply_chat_template(...)`，把 `conversation` 序列化成 chat prompt，并注入 `add_generation_prompt=True` 与 `open_thinking`。
- **第 78 行**：tokenize 成 `input_ids` 与 `attention_mask`，并转到 device。
- **第 80-81 行**：打印 AI 前缀并记录开始时间。
- **第 82-87 行**：调用 `model.generate(...)`：
  - `max_new_tokens` 控制新生成长度；
  - `do_sample=True` 开启 sampling；
  - `streamer=streamer` 实现终端实时输出；
  - `pad_token_id/eos_token_id` 决定 padding 与停止条件；
  - `top_p`、`temperature` 控制随机性；
  - `repetition_penalty=1` 实际上等于没有额外重复惩罚。
- **第 88 行**：只截取新生成部分进行 `decode()`，得到纯回答文本。
- **第 89 行**：把 assistant 回答加入 `conversation`，供下一轮继续使用。
- **第 90 行**：计算本轮新增 token 数量。
- **第 91 行**：如果 `show_speed` 为真，则输出 tokens/s；否则只输出换行。
- **第 93-94 行**：标准入口 `if __name__ == "__main__": main()`。

## 4. 关键算法解析 (Key Algorithm Deep-Dive)
### 4.1 模型初始化：native vs Transformers
- **Native path**：先构建 `MiniMindForCausalLM(MiniMindConfig)`，再手动 `load_state_dict()`；要求结构参数与 checkpoint 严格匹配。
- **Transformers path**：直接 `AutoModelForCausalLM.from_pretrained()`；更适合分发、部署与生态集成。
- **LoRA path**：通过 `apply_lora()` + `load_lora()` 叠加 adapter，属于 base model + delta inference。

### 4.2 Chat template 与 history management
`apply_chat_template()` 会把 `conversation` 转成模型能理解的聊天格式。`conversation[-args.historys:]` 是按消息条数做滑动窗口裁剪，而不是按 token 数裁剪，所以实现简单，但不能精确控制 context budget。

### 4.3 TextStreamer
`TextStreamer` 会在 `generate()` 进行时持续把 finalized text 打到终端，提供 streaming-like 交互感；而脚本在生成结束后仍额外 `decode()` 一次，以便把完整回答写回历史。

### 4.4 Generation parameters
- `temperature`：控制 logits 平滑，越高越发散。
- `top_p`：做 nucleus sampling。
- `top_k`：本脚本**未设置**，说明主要依靠 `top_p + temperature`。
- `repetition_penalty=1`：不额外惩罚重复。
- `max_new_tokens`：限制回答长度，而不是总上下文长度。

## 5. 调用关系 (Call Graph)
```text
main()
 ├─ argparse.parse_args()
 ├─ init_model(args)
 │   ├─ AutoTokenizer.from_pretrained()
 │   ├─ MiniMindForCausalLM(MiniMindConfig) + torch.load() + load_state_dict() [+ LoRA]
 │   └─ AutoModelForCausalLM.from_pretrained()
 └─ 对话循环
     ├─ setup_seed()
     ├─ tokenizer.apply_chat_template() / bos_token + prompt
     ├─ tokenizer(...)
     ├─ model.generate(..., streamer=TextStreamer)
     └─ tokenizer.decode()
```
这个脚本与模型的关键接口是 `generate()`，与 tokenizer 的关键接口是 `apply_chat_template()`、`tokenizer()` 和 `decode()`。
