# MiniMind 全面教程（Comprehensive Tutorial）

> 面向对象（Target Audience）：想系统理解 LLM internals、并亲手跑通一个超小语言模型训练闭环的学习者。  
> 项目位置（Project Path）：`/root/xw/ModelTrain/minimind/`

---

## 1. 项目概述（Project Overview）

MiniMind 是一个**教学导向（education-oriented）**的小型大语言模型（Large Language Model, LLM）项目。它的核心目标不是“再造一个更大的商用模型”，而是：

- 让你从 **0 到 1** 理解 LLM 的训练与推理；
- 用**尽可能低的成本（low cost）**复现完整流程；
- 用相对简洁、可读的 PyTorch 代码，把 Transformer、MoE、RL、Tool Use、Distillation 等关键概念串起来。

根据项目 README，`minimind-3` 的一条典型快速复现路线：

- `pretrain_t2t_mini.jsonl` + `sft_t2t_mini.jsonl`
- 单卡 `3090`
- 约 `2.31h`
- 成本约 **3 RMB**

这就是 README 中“**花约 3 块钱训练一个 64M LLM**”的来源。

### 为什么这个项目对学习很重要？

大多数人接触 LLM 时，往往直接使用：

- `transformers`
- `trl`
- `peft`
- 各种一键训练框架

这些工具非常强大，但也会把很多底层细节封装掉。MiniMind 的价值在于：

1. **模型结构足够现代（modern）**  
   它并不是过时的 toy model，而是对齐 `Qwen3 / Qwen3-MoE` 风格生态：
   - RMSNorm
   - RoPE
   - GQA
   - SwiGLU
   - MoE
   - weight tying

2. **训练链路完整（end-to-end）**  
   不只有 Pretrain，还包含：
   - SFT
   - LoRA
   - DPO
   - PPO
   - GRPO / CISPO
   - Agentic RL
   - Knowledge Distillation

3. **工程复杂度可控（manageable complexity）**  
   相比十亿、百亿参数模型，64M / 198M-A64M 规模更适合学习：
   - 显存门槛低
   - 实验迭代快
   - 出错后更容易 debug

### 一句话理解 MiniMind

> MiniMind = 一个“**小而全（small but complete）**”的 LLM 学习实验场。

---

## 2. 项目结构（Project Structure）

下面是一个适合学习时记忆的目录图：

```text
minimind/
├── model/                  # 模型定义、LoRA、tokenizer
│   ├── model_minimind.py   # Dense/MoE 主模型
│   ├── model_lora.py       # LoRA 注入、保存、合并
│   ├── tokenizer.json
│   └── tokenizer_config.json
├── trainer/                # 训练脚本与训练工具
│   ├── train_pretrain.py
│   ├── train_full_sft.py
│   ├── train_lora.py
│   ├── train_dpo.py
│   ├── train_ppo.py
│   ├── train_grpo.py
│   ├── train_agent.py
│   ├── train_distillation.py
│   ├── train_tokenizer.py
│   ├── trainer_utils.py
│   └── rollout_engine.py
├── scripts/                # 推理、部署、转换、Web/API 工具
│   ├── serve_openai_api.py
│   ├── web_demo.py
│   ├── convert_model.py
│   ├── chat_api.py
│   └── eval_toolcall.py
├── dataset/                # 数据集说明与 Dataset 类
│   ├── lm_dataset.py
│   ├── dataset.md
│   └── download_modelscope
├── eval_llm.py             # 命令行交互推理
├── requirements.txt        # 依赖
└── README.md               # 项目总说明
```

### 2.1 `model/`

这是**模型核心（model core）**。

- `model_minimind.py`：
  - `MiniMindConfig`
  - `RMSNorm`
  - `Attention`
  - `FeedForward`
  - `MOEFeedForward`
  - `MiniMindBlock`
  - `MiniMindModel`
  - `MiniMindForCausalLM`

- `model_lora.py`：
  - LoRA 低秩适配（Low-Rank Adaptation）
  - 仅对**方阵 Linear 层（square Linear layers）**插入 LoRA
  - 支持保存、加载、合并（merge）

- `tokenizer.json` / `tokenizer_config.json`：
  - BPE + ByteLevel tokenizer
  - 特殊 token（special tokens）
  - `chat_template`（Jinja2）

### 2.2 `trainer/`

这是**训练流水线（training pipeline）**的主战场。

- `train_pretrain.py`：预训练（next-token prediction）
- `train_full_sft.py`：全参数指令微调（full SFT）
- `train_lora.py`：参数高效微调（PEFT）
- `train_dpo.py`：偏好优化（preference optimization）
- `train_ppo.py`：Actor-Critic PPO
- `train_grpo.py`：GRPO / CISPO
- `train_agent.py`：多轮工具调用强化学习（Agentic RL）
- `train_distillation.py`：知识蒸馏（distillation）
- `train_tokenizer.py`：Tokenizer 训练示例
- `trainer_utils.py`：
  - 学习率（learning rate）
  - DDP 初始化
  - checkpoint 保存/恢复
  - model/tokenizer 初始化
- `rollout_engine.py`：
  - rollout 抽象层
  - 本地 PyTorch rollout
  - SGLang rollout

### 2.3 `scripts/`

这是**推理与部署（inference & deployment）**工具箱。

- `serve_openai_api.py`：OpenAI-compatible API server
- `web_demo.py`：Streamlit Web UI
- `convert_model.py`：
  - `torch -> transformers`
  - `transformers -> torch`
  - LoRA merge
- `chat_api.py`：API 调用示例
- `eval_toolcall.py`：Tool Call 能力测试

### 2.4 `dataset/`

这是**数据读取层（data layer）**。

- `lm_dataset.py`：定义多种 Dataset
  - `PretrainDataset`
  - `SFTDataset`
  - `DPODataset`
  - `RLAIFDataset`
  - `AgentRLDataset`
- `download_modelscope`：给出 ModelScope 下载方式
- `dataset.md`：数据放置说明

---

## 3. 模型架构（Model Architecture）

先看整体结构图：

```text
input text
   │
   ▼
Tokenizer (BPE + ByteLevel)
   │
   ▼
Embedding
   │
   ▼
[ MiniMindBlock × N ]
   │    ├─ RMSNorm
   │    ├─ GQA Attention + RoPE (+ YaRN)
   │    ├─ Residual
   │    ├─ RMSNorm
   │    └─ SwiGLU FFN / MoE FFN
   ▼
Final RMSNorm
   │
   ▼
lm_head
   │
   ▼
next-token logits
```

### 3.1 `MiniMindConfig`：超参数总表（Hyperparameter Table）

`MiniMindConfig` 是整个模型的配置中心。

| 参数 | 含义 | 默认值/说明 |
|---|---|---|
| `hidden_size` | 隐藏维度（model width） | `768` |
| `num_hidden_layers` | Transformer block 层数 | `8` |
| `use_moe` | 是否启用 MoE | `False` |
| `dropout` | dropout 概率 | `0.0` |
| `vocab_size` | 词表大小 | `6400` |
| `bos_token_id` | 起始 token | `1` |
| `eos_token_id` | 终止 token | `2` |
| `flash_attn` | 是否优先使用 Flash Attention / SDPA | `True` |
| `num_attention_heads` | Query heads 数量 | `8` |
| `num_key_value_heads` | Key/Value heads 数量 | `4` |
| `head_dim` | 每个头的维度 | `hidden_size / num_attention_heads` |
| `hidden_act` | FFN 激活函数 | `silu` |
| `intermediate_size` | FFN 中间层维度 | `ceil(hidden_size * π / 64) * 64` |
| `max_position_embeddings` | 最大位置长度 | `32768` |
| `rms_norm_eps` | RMSNorm 的 eps | `1e-6` |
| `rope_theta` | RoPE base | `1e6` |
| `tie_word_embeddings` | 是否共享 embedding 与 lm_head | `True` |
| `inference_rope_scaling` | 推理期是否启用 YaRN 外推 | `False` |
| `rope_scaling` | YaRN 配置字典 | 启用外推时自动生成 |
| `num_experts` | MoE 专家数 | `4` |
| `num_experts_per_tok` | 每个 token 激活几个专家 | `1` |
| `moe_intermediate_size` | 每个专家 FFN 的中间层维度 | 默认等于 `intermediate_size` |
| `norm_topk_prob` | 是否归一化 top-k 概率 | `True` |
| `router_aux_loss_coef` | 路由辅助损失系数 | `5e-4` |

### 3.2 RMSNorm（vs LayerNorm）

MiniMind 使用的是 **RMSNorm**，不是 **LayerNorm**。

RMSNorm 的核心思想：

- 只按均方根（root mean square）做缩放；
- 不显式减去均值（mean subtraction）；
- 参数更少、计算更简单。

代码逻辑近似是：

```python
x_norm = x * rsqrt(mean(x^2) + eps)
out = weight * x_norm
```

#### 为什么小模型也喜欢 RMSNorm？

相比 LayerNorm：

- **更轻量（lighter）**
- **更常见于现代 decoder-only LLM**
- 在大模型实践中已被广泛验证

对学习者来说，理解要点是：

> LayerNorm 更“完整”，RMSNorm 更“高效简洁”。

### 3.3 Rotary Position Embedding（RoPE）+ YaRN extension

MiniMind 使用 **RoPE（Rotary Position Embedding）** 来编码位置信息。

它不是把位置向量直接加到 token embedding 上，而是：

- 在 attention 的 `q` / `k` 上做旋转（rotation）
- 让相对位置信息自然体现在内积里

简化理解：

```text
q, k 先线性投影
   ↓
对 q, k 的偶/奇维做旋转
   ↓
再参与 attention score 计算
```

#### `precompute_freqs_cis`

模型会预先计算：

- `freqs_cos`
- `freqs_sin`

这样在 forward 时可以直接切片使用，提高效率。

#### YaRN（Yet another RoPE extensioN）

当 `inference_rope_scaling=True` 时，会自动构造 `rope_scaling`，启用 **YaRN** 外推。

它的作用是：

- 把训练时较短的 RoPE 长度，外推到更长上下文；
- 通过调节不同频段的缩放比例，减轻长上下文时 RoPE 失真。

README 中也强调：

- 这更像是**位置编码外推（position extrapolation）**；
- 它能缓解长度问题，但**不能凭空创造长上下文理解能力**。

### 3.4 Grouped Query Attention（GQA）+ Flash Attention support

MiniMind 的 attention 配置：

- `q_heads = 8`
- `kv_heads = 4`

这就是 **GQA（Grouped Query Attention）**：

- Query 头更多
- Key/Value 头更少
- 多个 Query head 共享一组 Key/Value head

图示如下：

```text
Q heads:   q1 q2 q3 q4 q5 q6 q7 q8
KV heads:  k1    k2    k3    k4
           v1    v2    v3    v4

映射关系（repeat_kv）：
q1 q2 -> k1 v1
q3 q4 -> k2 v2
q5 q6 -> k3 v3
q7 q8 -> k4 v4
```

#### 为什么这样设计？

GQA 的优点：

- 减少 KV cache 大小
- 降低推理显存开销
- 兼顾效果与效率

#### Attention 里的几个关键点

1. `q_proj / k_proj / v_proj / o_proj`
2. `q_norm / k_norm`：对 q 和 k 单独做 RMSNorm
3. `apply_rotary_pos_emb(q, k, cos, sin)`
4. `repeat_kv()`：把较少的 KV heads 扩展到 Query heads 数量
5. 因果掩码（causal mask）
6. past key/value cache（自回归生成）

#### Flash Attention support

代码里会优先检查：

- `torch.nn.functional.scaled_dot_product_attention` 是否存在
- `config.flash_attn` 是否开启
- 当前场景是否适合走 flash path

如果满足条件，就走更高效的 **SDPA / Flash-like path**；否则退回手写 attention。

### 3.5 SwiGLU FeedForward

MiniMind 的 FFN 不是传统 `Linear -> GELU -> Linear`，而是 **SwiGLU** 风格：

- `gate_proj`
- `up_proj`
- `down_proj`

公式可理解为：

```text
FFN(x) = down_proj( SiLU(gate_proj(x)) * up_proj(x) )
```

其中：

- `gate_proj(x)` 产生门控（gate）
- `up_proj(x)` 产生候选特征
- 两者逐元素相乘（element-wise multiply）
- `down_proj` 投回 hidden size

#### 为什么用 SwiGLU？

相比传统 FFN：

- 表达能力更强
- 现代 LLM 中很常见
- 在相近参数量下效果通常更好

### 3.6 Mixture of Experts（MoE）

当 `use_moe=True` 时，FFN 会替换为 `MOEFeedForward`。

其流程如下：

```text
token hidden state
   │
   ▼
router gate (Linear -> softmax)
   │
   ├─ top-k expert selection
   ├─ probability normalization
   └─ dispatch to experts
   ▼
expert outputs weighted sum
   ▼
merged output
```

#### 关键组件

- `gate = nn.Linear(hidden_size, num_experts)`：路由器（router）
- `experts = ModuleList([...])`：多个 FFN 专家
- `topk_weight, topk_idx = torch.topk(scores, k=num_experts_per_tok)`

默认主线配置：

- `4 experts`
- `top-1 routing`

也就是说：

- 每个 token 只激活 1 个专家
- 总容量更大，但单 token 激活参数量不变太多

#### Load balancing loss（负载均衡损失）

MoE 常见问题是：

- 所有 token 都挤向少数几个专家
- 导致专家利用不均衡（imbalance）

MiniMind 在训练中加入 `aux_loss`：

- 统计 top-k 路由的 one-hot 分布 `load`
- 与平均路由分数 `scores.mean(0)` 结合
- 再乘 `router_aux_loss_coef`

直觉上就是：

> 鼓励专家“不要闲着”，也不要“只有一个专家干活”。

### 3.7 `MiniMindBlock`：Pre-Norm Transformer block

`MiniMindBlock` 使用 **Pre-Norm（预归一化）** 结构。

一个 block 的计算顺序是：

```text
x
│
├─ RMSNorm
├─ Self-Attention
├─ Residual Add
│
├─ RMSNorm
├─ FFN / MoE
└─ Residual Add
```

对应代码逻辑：

```python
hidden_states = hidden_states + self_attn(norm(hidden_states))
hidden_states = hidden_states + mlp(norm(hidden_states))
```

#### 为什么很多现代 LLM 用 Pre-Norm？

因为它通常：

- 训练更稳定
- 更适合深层网络
- 梯度传播更顺畅

### 3.8 `MiniMindForCausalLM`

这是最终对外暴露的**因果语言模型（causal language model）**封装。

#### 1）Weight tying

```python
if tie_word_embeddings:
    self.model.embed_tokens.weight = self.lm_head.weight
```

即：

- 输入 embedding
- 输出 lm_head

共享同一套权重。

好处：

- 降低参数量
- 对小模型尤其重要
- 常见于 GPT 系列 / decoder-only LLM

#### 2）训练时的 loss

当提供 `labels` 时：

- logits 去掉最后一个位置
- labels 去掉第一个位置
- 做标准 next-token cross entropy

即自回归训练（autoregressive training）：

```text
给定 token_1 ... token_t
预测 token_{t+1}
```

MoE 模式下总 loss =

```text
CE loss + aux_loss
```

#### 3）生成（generation）

MiniMind 自己实现了 `generate()`，支持：

- `temperature`
- `top_k`
- `top_p`（nucleus sampling）
- `repetition_penalty`
- `use_cache`
- `streamer`
- `num_return_sequences`

生成循环核心逻辑：

```text
prompt
  ↓
forward 得到最后一个 token logits
  ↓
温度缩放 / repetition penalty
  ↓
top-k / top-p 过滤
  ↓
采样或贪心选择
  ↓
拼回输入，继续下一轮
```

#### repetition_penalty

如果某些 token 已经在历史里出现过很多次，就降低它们再次被采样的倾向，减少复读（repetition）。

---

## 4. Tokenizer

MiniMind 的 tokenizer 不是 SentencePiece，而是：

- **BPE（Byte Pair Encoding）**
- **ByteLevel pre-tokenizer / decoder**

### 4.1 为什么是 BPE + ByteLevel？

优点：

- 对中英混合文本鲁棒（robust）
- 不容易因为未知字符（unknown character）彻底崩掉
- 词表可以做得比较小

当前主线词表大小：

- `6400`

这对小模型很重要，因为：

- embedding 层参数 = `vocab_size × hidden_size`
- lm_head 参数也与词表大小强相关

词表小，模型就更“轻”。

### 4.2 特殊 tokens（Special Tokens）

MiniMind tokenizer 中比较重要的 token 包括：

#### 基础对话模板 tokens

- `<|endoftext|>`：pad / unk
- `<|im_start|>`：BOS 风格角色起始
- `<|im_end|>`：EOS 风格消息结束

#### Tool / Thinking tokens

- `<tool_call>` / `</tool_call>`
- `<tool_response>` / `</tool_response>`
- `<think>` / `</think>`

#### 预留 buffer tokens

- `<|buffer1|>` ... `<|buffer9|>`

这些 buffer token 的作用是：

- 给后续模板扩展预留位置
- 避免将来随意改 tokenizer 破坏兼容性

### 4.3 一个非常关键的设计：Tool/Think 标签作为“可见文本 token”

`<tool_call>`、`<tool_response>`、`<think>` 虽然是模板标记，但它们在实践中更像**结构化文本 token**，而不是纯粹被跳过的不可见 special token。

这意味着模型是在真正学习：

- 什么时候输出 `<tool_call>`
- 什么时候结束 `</think>`
- 什么时候把工具结果包进 `<tool_response>`

这对 Tool Use / Reasoning 很重要。

### 4.4 `chat_template`（Jinja2 template）

MiniMind 的 `tokenizer_config.json` 里自带了一个较完整的 **Jinja2 chat_template**。

它完成了几件事：

1. **角色模板化（role formatting）**
   - system
   - user
   - assistant
   - tool

2. **Tool schema 注入**
   - 把工具定义挂进 system 段
   - 包装进 `<tools>...</tools>`

3. **assistant 的 reasoning_content 处理**
   - 如果有 `reasoning_content`，就写入 `<think>...</think>`
   - 如果 `open_thinking=False`，则注入空的 `<think>\n\n</think>`

4. **tool message 包装**
   - tool 消息会被包成 `<tool_response>...</tool_response>`

#### 模板展开示意（simplified）

```text
<|im_start|>system
你是一个助手...
<|im_end|>
<|im_start|>user
你好
<|im_end|>
<|im_start|>assistant
<think>
...
</think>

最终回答
<|im_end|>
```

#### Tool Calling 模板示意

```text
<|im_start|>system
# Tools
<tools>
{...tool schema...}
</tools>
...
<|im_end|>
```

然后 assistant 可能输出：

```text
<tool_call>
{"name": "calculate_math", "arguments": {"expression": "256*37"}}
</tool_call>
```

### 4.5 `open_thinking` 的意义

`open_thinking` 是 MiniMind 很有特色的一个设计。

- `open_thinking=False`：模板里会预埋空 `<think></think>`，模型更容易直接回答
- `open_thinking=True`：模板只打开 `<think>` 起始，模型可继续写显式推理过程（explicit reasoning）

所以它不是“切换不同模型”，而是：

> 用**同一个模型（same model）**，通过**不同模板（different template prompt）**切换思考模式。

---

## 5. 训练流水线（Training Pipeline）

MiniMind 的训练链路可以画成：

```text
Stage 1  Pretrain
   ↓
Stage 2  Full SFT
   ├─ Stage 3 LoRA（平行分支，按需）
   ├─ Stage 4 DPO
   ├─ Stage 5 PPO
   ├─ Stage 6 GRPO / CISPO
   ├─ Stage 7 Agentic RL
   └─ Stage 8 Distillation
```

> 不是所有阶段都必须执行。最核心的主线通常是：`Pretrain -> Full SFT -> Eval`。

### 5.1 Stage 1：Pretrain

**目标（Goal）**：让模型学会基础语言统计规律与世界知识。

输入数据格式：

```json
{"text": "Transformer 通过自注意力机制建模上下文关系。"}
```

`PretrainDataset` 的做法：

1. 读取 `text`
2. tokenizer 编码
3. 前后加 BOS / EOS
4. pad 到固定长度
5. `labels = input_ids`，pad 位置置为 `-100`

所以本质上就是：

- **next-token prediction**
- 标准 causal LM 训练

### 5.2 Stage 2：Full SFT（Supervised Fine-Tuning）

**目标（Goal）**：让模型学会助手行为（assistant behavior）、多轮对话格式、Tool Use、Thinking 模板。

SFT 关键点在 `SFTDataset`：

#### 1）先把结构化 conversations 交给 chat_template

```python
prompt = tokenizer.apply_chat_template(messages, tokenize=False)
```

#### 2）只让 assistant 部分参与 loss

`generate_labels()` 会扫描：

- `<|im_start|>assistant\n`
- 到 `<|im_end|>`

只对 assistant span 填标签，其它位置都设为 `-100`。

这非常重要，因为它体现了 instruction tuning 的基本思路：

> 模型不是去“背整个 prompt”，而是去学习**如何回复（how to respond）**。

### 5.3 Stage 3：LoRA fine-tuning

**目标（Goal）**：低成本适配领域数据（domain adaptation）。

MiniMind 的 LoRA 实现很直接：

- 遍历所有 `nn.Linear`
- 只对**输出维度 = 输入维度**的方阵 Linear 层插 LoRA
- 给每个层附加一个 `module.lora`
- forward 变成：

```text
original_linear(x) + lora(x)
```

#### 为什么只改 square Linear layers？

因为这样：

- 实现更简单
- 对 attention / FFN 中一部分核心投影层很自然
- 参数量可控

训练脚本会：

- 冻结原模型参数
- 只优化名字里带 `lora` 的参数

所以它是一个典型的 **PEFT（Parameter-Efficient Fine-Tuning）**。

### 5.4 Stage 4：DPO（Direct Preference Optimization）

**目标（Goal）**：通过 chosen vs rejected 偏好对，让模型更偏向“人更喜欢”的回答。

MiniMind 的 DPO 训练流程：

1. 读入 `chosen` / `rejected`
2. 用 policy model 和 reference model 分别算 log-prob
3. 计算 DPO loss：

```text
loss = -log sigmoid(beta * ((π_chosen - π_rejected) - (ref_chosen - ref_rejected)))
```

#### 你可以这样理解 DPO

- policy model：当前要学习的模型
- reference model：冻结的旧模型，提供“不要偏移太远”的基线
- chosen / rejected：正反例

DPO 不需要单独训练 reward model 或 critic，因此比 PPO 更简单、显存更省。

### 5.5 Stage 5：PPO（Actor-Critic + GAE）

**目标（Goal）**：做在线强化学习（online RL），让模型通过 rollout + reward 继续优化。

MiniMind 的 PPO 有三类模型：

- **Actor**：策略模型，负责生成回答
- **Critic**：价值模型，输出每个位置的 value
- **Reference model**：KL 约束参考模型

此外还有：

- **Reward model**：给整段回答打分

#### PPO 训练闭环

```text
prompt
  ↓
Actor rollout response
  ↓
Reward model + heuristic reward
  ↓
Critic 估计 value
  ↓
GAE 计算 advantage
  ↓
clipped PPO objective 更新 Actor
  ↓
value loss 更新 Critic
```

#### Reward 组成（MiniMind 默认示例）

- 回答长度是否合适
- 是否正确闭合 `</think>`
- 思考长度是否合理
- 是否重复（repetition penalty）
- Reward model 分数

#### GAE（Generalized Advantage Estimation）

PPO 使用：

- `gamma`
- `lam`

从 token-level reward 与 value 序列构造 advantage，减少高方差问题。

### 5.6 Stage 6：GRPO / CISPO

MiniMind 把这类 **group-based policy optimization** 放在 `train_grpo.py`。

#### GRPO 核心思想

对同一个 prompt：

- 生成 `num_generations` 个回答
- 每个回答得到 reward
- 用组内均值和标准差归一化
- 构造相对优势（relative advantage）

即：

```text
A_i = (R_i - mean(group)) / std(group)
```

这意味着：

- 不需要 critic 网络
- 训练更简洁
- 但若同组回答都差不多，会出现 learning signal 很弱的问题

#### CISPO 核心思想

MiniMind 里 `loss_type` 可选：

- `grpo`
- `cispo`

`cispo` 的思路是：

- 保留 importance ratio 的作用
- 但避免传统 clip 后梯度被“硬截断”得太厉害
- 用“裁剪权重 × log-prob”式目标替代经典最小值写法

如果你第一次学 RL，可以把它理解成：

> CISPO = 对 GRPO/PPO 风格目标的一个更“平滑”的 loss 变体。

### 5.7 Stage 7：Agentic RL

这是 MiniMind 很有意思的一部分。

普通 RL 往往是一轮问答就打分；Agentic RL 则是：

- 模型先决定是否调用工具
- 工具执行后把 observation 放回上下文
- 再继续生成
- 多轮后再整体打 reward

#### 多轮轨迹（trajectory）示意

```text
user question
   ↓
assistant: <tool_call>...</tool_call>
   ↓
tool executes
   ↓
<tool_response>...</tool_response>
   ↓
assistant continues
   ↓
maybe another tool call
   ↓
final answer
   ↓
trajectory-level reward
```

`train_agent.py` 里有几个关键函数：

- `rollout_single()`：单样本多轮 rollout
- `rollout_batch()`：批量 rollout
- `parse_tool_calls()`：从文本里解析 `<tool_call>`
- `execute_tool()`：模拟工具执行
- `calculate_rewards()`：联合打分

#### Agent reward 由哪些部分组成？

大体包括：

- Tool 数量与合法性
- 参数是否有效
- `gt`（ground truth）是否命中
- 最终回答是否完成
- 格式闭合
- Reward model 分数
- unfinished penalty

这是一个典型的**延迟奖励（delayed reward）**场景。

### 5.8 Stage 8：Knowledge Distillation

MiniMind 的蒸馏脚本体现的是：

- **teacher-student distillation**
- **CE + KL** 混合目标

总损失形式：

```text
loss = alpha * CE + (1 - alpha) * KL(teacher || student)
```

其中：

- `alpha`：控制 hard label vs soft target 的权重
- `temperature`：蒸馏温度

代码里还支持：

- dense <- moe
- moe <- dense
- 同尺寸或不同尺寸 teacher/student

所以它既是一个训练脚本，也是理解 distillation 的教学样例。

---

## 6. 数据集说明（Dataset Description）

MiniMind 当前主线常用数据：

```text
./dataset/
├── pretrain_t2t_mini.jsonl
├── pretrain_t2t.jsonl
├── sft_t2t_mini.jsonl
├── sft_t2t.jsonl
├── dpo.jsonl
├── rlaif.jsonl
├── agent_rl.jsonl
└── agent_rl_math.jsonl
```

### 6.1 `pretrain_t2t.jsonl`

这是最简单的数据格式：

```jsonl
{"text": "如何才能摆脱拖延症？治愈拖延症并不容易，但以下建议可能有所帮助。"}
{"text": "Transformer 通过自注意力机制建模上下文关系，是现代大语言模型的重要基础结构。"}
```

特点：

- 单字段 `text`
- 用于 next-token prediction
- 无显式角色（no roles）

### 6.2 `sft_t2t.jsonl`

这是最核心的数据格式。每条样本一般是：

```jsonl
{
  "conversations": [
    {"role": "user", "content": "你好"},
    {"role": "assistant", "content": "你好！很高兴见到你。"}
  ]
}
```

#### 带 `reasoning_content` 的形式

MiniMind 的 structured message 支持把 reasoning 单独放在字段里：

```jsonl
{
  "conversations": [
    {"role": "user", "content": "请比较 Transformer 和 RNN"},
    {
      "role": "assistant",
      "reasoning_content": "先比较并行性、长距离依赖和训练效率，再给结论。",
      "content": "Transformer 更适合并行计算，长距离依赖建模也更强；RNN 结构更直观，但训练速度和长程依赖通常不如 Transformer。"
    }
  ]
}
```

在 `chat_template` 展开后，`reasoning_content` 会进入：

```text
<think>...</think>
```

#### 带 Tool Calls 的形式

```jsonl
{
  "conversations": [
    {
      "role": "system",
      "content": "# Tools ...",
      "tools": "[{\"type\":\"function\",\"function\":{\"name\":\"calculate_math\",\"parameters\":{...}}}]"
    },
    {"role": "user", "content": "帮我算一下 256 乘以 37"},
    {
      "role": "assistant",
      "content": "",
      "tool_calls": "[{\"name\":\"calculate_math\",\"arguments\":{\"expression\":\"256 * 37\"}}]"
    },
    {"role": "tool", "content": "{\"result\":\"9472\"}"},
    {"role": "assistant", "content": "256 乘以 37 等于 9472。"}
  ]
}
```

#### `SFTDataset` 的关键处理

- `tools` 通常挂在 `system` message 上
- `tool_calls` 挂在 `assistant` message 上
- 训练时会自动模板化成 `<tool_call>` / `<tool_response>`
- loss 只计算 assistant span

### 6.3 `dpo.jsonl`

偏好学习（preference learning）格式：

```jsonl
{
  "chosen": [
    {"role": "user", "content": "Q"},
    {"role": "assistant", "content": "good answer"}
  ],
  "rejected": [
    {"role": "user", "content": "Q"},
    {"role": "assistant", "content": "bad answer"}
  ]
}
```

特点：

- `chosen` = 更好答案
- `rejected` = 更差答案
- DPO 会分别计算两者 log-prob

### 6.4 `rlaif.jsonl`

RLAIF 数据通常长得像 SFT，但**最后一个 assistant 不需要真实答案**，因为训练时由当前 policy 在线生成。

```jsonl
{
  "conversations": [
    {"role": "user", "content": "请解释一下什么是光合作用？"},
    {"role": "assistant", "content": "无"}
  ]
}
```

更准确地说，`RLAIFDataset` 在训练中主要使用：

- prompt
- policy rollout response
- reward model / rule reward

而不是离线固定答案。

### 6.5 `agent_rl.jsonl`

Agent RL 数据比普通对话多一个关键字段：`gt`（ground truth）。

它表示：

- 工具调用链最终应该命中的结果
- 或者可验证任务的目标答案

示意格式如下：

```jsonl
{
  "conversations": [
    {
      "role": "system",
      "content": "# Tools ...",
      "tools": "[...]"
    },
    {"role": "user", "content": "帮我生成一个1到100的随机数，然后计算平方"},
    {"role": "assistant", "content": ""}
  ],
  "gt": ["平方结果", "最终结论"]
}
```

`AgentRLDataset` 实际读取时会：

- 取 `messages[:-1]` 作为 rollout 起点
- 解析 `tools`
- 保留 `gt`

所以你可以把它理解为：

> 这是一个“带工具环境与目标检查”的 RL 数据格式。

---

## 7. 推理与部署（Inference & Deployment）

### 7.1 `eval_llm.py`：交互式聊天（Interactive Chat）

这是最直接的入口。

支持两种加载方式：

1. **原生 PyTorch 权重**
2. **Transformers 格式模型**

基本用法：

```bash
python eval_llm.py --load_from ./model --weight full_sft
```

或者：

```bash
python eval_llm.py --load_from ./minimind-3
```

#### 常用参数

- `--open_thinking 1`：开启显式思考
- `--historys N`：保留历史轮次
- `--temperature`
- `--top_p`
- `--max_new_tokens`
- `--lora_weight xxx`：叠加 LoRA

### 7.2 `serve_openai_api.py`：OpenAI-compatible API server

这个脚本用 `FastAPI + StreamingResponse` 实现了一个轻量 API 服务。

支持：

- `/v1/chat/completions`
- `stream=true`
- `tools`
- `open_thinking`
- `reasoning_content`
- `tool_calls`

#### 启动方式

```bash
cd scripts
python serve_openai_api.py --load_from ../minimind-3
```

#### 请求示例

```bash
curl http://localhost:8998/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{
    "model": "minimind",
    "messages": [{"role": "user", "content": "请介绍一下 Transformer"}],
    "stream": true,
    "open_thinking": true
  }'
```

#### 它做了什么额外处理？

`parse_response()` 会把模型输出拆成：

- `content`
- `reasoning_content`
- `tool_calls`

这使得 MiniMind 更容易接入：

- Open-WebUI
- FastGPT
- Dify
- 自定义前端

### 7.3 `web_demo.py`：Streamlit Web UI

这个脚本提供一个教学友好的 Web 界面。

功能包括：

- 多轮对话（multi-turn chat）
- Thinking 展示
- Tool 选择（最多 4 个）
- 模型切换
- streaming 输出

启动：

```bash
cd scripts
streamlit run web_demo.py
```

注意：脚本会扫描 `scripts/` 目录下包含权重文件的子目录，所以通常需要把 Transformers 模型目录放到 `scripts/` 下或按 README 指示复制过去。

### 7.4 模型转换（torch ↔ transformers）

`convert_model.py` 提供了几个关键功能：

#### 1）`convert_torch2transformers_minimind`

- 把 MiniMind 原生 PyTorch 权重导出为保留自定义结构的 Transformers 形式

#### 2）`convert_torch2transformers`

- 把 MiniMind 权重映射到更生态友好的 `Qwen3 / Qwen3-MoE` 风格结构
- 方便接入第三方推理框架

#### 3）`convert_transformers2torch`

- 从 Transformers 格式转回原生 torch `.pth`

#### 4）`convert_merge_base_lora`

- 先加载 base model
- 应用 LoRA
- 把 LoRA 增量合并回原模型权重

这在部署时非常实用，因为很多推理引擎更喜欢“合并后的完整模型”。

### 7.5 llama.cpp / vLLM / Ollama compatibility

MiniMind README 专门说明了与主流推理生态的兼容路线。

#### vLLM

如果已经是 Transformers 格式：

```bash
vllm serve /path/to/model --model-impl transformers --served-model-name minimind --port 8998
```

#### llama.cpp

大体步骤：

1. 把 HF 模型转换成 GGUF
2. 必要时在 `convert_hf_to_gguf.py` 里给 tokenizer 加兼容分支
3. 可进一步量化（quantization）

#### Ollama

通常路径是：

```text
MiniMind HF model
   ↓
GGUF
   ↓
Modelfile
   ↓
ollama create / ollama run
```

README 还给出了支持 Tool Calling 的 `TEMPLATE` 示例，可把 MiniMind 的 chat_template 思路迁移到 Ollama。

### 7.6 额外说明：SGLang rollout / serve

虽然用户这里主要问部署，但 MiniMind 还有一个值得学习的点：

- 推理可以在训练外部由 `SGLang` 负责
- 训练脚本只关心 rollout result

这是“**训推分离（training-serving decoupling）**”的一个入门版工程设计。

---

## 8. 训练技巧（Training Tips）

### 8.1 Cosine LR schedule with warmup

先说一个**非常重要的准确性说明**：

- MiniMind 当前主线脚本里，`pretrain / sft / lora / dpo / distillation` 主要使用的是 **cosine-style decay**（`trainer_utils.get_lr()`）
- `ppo / grpo / agent` 使用 `CosineAnnealingLR`
- **默认并没有单独暴露“显式 warmup”参数**

也就是说，仓库当前实现更接近：

```text
cosine decay / cosine annealing
```

而不是标准的：

```text
linear warmup + cosine decay
```

#### 学习建议

如果你在自己扩展 MiniMind：

- 可以把前 `1% ~ 5%` steps 加上线性 warmup
- 然后再接 cosine decay

这通常对：

- 较大学习率
- 较深模型
- 刚开始不稳定的训练

会更友好。

### 8.2 Gradient accumulation & clipping

几乎所有训练脚本都用了：

- `accumulation_steps`
- `clip_grad_norm_`

意义：

#### Gradient accumulation

显存不够大时，可以：

- 小 batch forward/backward 多次
- 再统一 optimizer.step()

等价于模拟更大的 global batch。

#### Gradient clipping

```python
torch.nn.utils.clip_grad_norm_(..., args.grad_clip)
```

意义：

- 防止梯度爆炸（gradient explosion）
- 让训练更稳定

### 8.3 Mixed precision（bf16 / fp16）

MiniMind 训练脚本里常见：

- `torch.cuda.amp.autocast(dtype=...)`
- `GradScaler(enabled=(args.dtype == 'float16'))`

要点：

- `bf16`：数值范围更稳，通常更推荐
- `fp16`：更省显存，但更依赖 `GradScaler`

#### 实践建议

- 支持 `bf16` 的 GPU，优先用 `bf16`
- 否则用 `fp16`
- CPU 下则不会开启 CUDA autocast

### 8.4 DDP + DeepSpeed multi-GPU

#### DDP（仓库内实际主线）

MiniMind 当前代码直接实现了：

- `torch.distributed`
- `DistributedDataParallel`
- `torchrun --nproc_per_node N ...`

初始化逻辑在：

- `trainer_utils.init_distributed_mode()`

所以**当前仓库真正直接可读的多卡实现主线是 DDP**。

#### DeepSpeed（README 里的扩展方向）

README 提到支持 `DeepSpeed` 生态，但当前代码树里**没有专门的 deepspeed config 文件或专用训练脚本**。更准确地说：

- DDP 是当前仓库原生实现
- DeepSpeed 是 README 中承认的可扩展/兼容方向
- 对更大 MoE 或更大 batch 来说，DeepSpeed/Megatron/Triton-fused kernels 会更有优势

#### 多卡命令示例

```bash
cd trainer
torchrun --nproc_per_node 4 train_pretrain.py
```

### 8.5 Checkpoint resume（断点续训）

MiniMind 对 checkpoint 做得很实用。

核心函数：

- `lm_checkpoint()`

会同时保存：

- 模型参数
- optimizer 状态
- scaler 状态
- epoch
- step
- scheduler（若有）
- critic / extra modules（若有）
- wandb/swanlab run id

#### 开启方式

```bash
python train_pretrain.py --from_resume 1
```

#### 特点

- 自动恢复训练进度
- 支持**跨 GPU 数量恢复**（自动调整 step）
- 使用临时 `.tmp` 再 `os.replace()`，更安全

### 8.6 wandb / swanlab logging

训练脚本都带：

- `--use_wandb`
- `--wandb_project`

但有一个小技巧：

```python
import swanlab as wandb
```

也就是说，代码使用的是 **wandb 风格 API**，但默认可以接国内更友好的 `swanlab`。

这对学习者很方便：

- 不用改太多代码
- 保持 logging 接口统一

### 8.7 训练时要特别注意的几个坑

#### 1）`max_seq_len` 与 `max_position_embeddings` 不是同一个概念

- `max_seq_len`：训练脚本里常表示当前 batch 的截断长度（truncation length）
- `max_position_embeddings`：模型配置里的理论位置上限

#### 2）小模型特别怕奖励稀疏（reward sparsity）

做 RL 时如果任务太难：

- 奖励几乎全是 0
- advantage 也接近 0
- 学不到东西

#### 3）MoE 不一定更快

MoE 的“参数容量更大”不等于“训练更快”。在原生 PyTorch 下，dispatch 和 expert 调度会带来开销。

---

## 9. 快速开始（Quick Start Guide）

下面给出一条**最适合初学者（beginner-friendly）**的最小主线：

```text
Environment setup
   ↓
Download mini datasets
   ↓
Pretrain
   ↓
Full SFT
   ↓
Eval
```

### 9.1 环境准备（Environment Setup）

```bash
cd /root/xw/ModelTrain/minimind
pip install -r requirements.txt -i https://mirrors.aliyun.com/pypi/simple
```

建议环境：

- Python `>=3.10`
- CUDA 可用（若走 GPU）
- 先确认：

```bash
python - <<'PY'
import torch
print('cuda:', torch.cuda.is_available())
PY
```

### 9.2 下载数据（Download Data）

项目自带了 ModelScope 下载说明。最简单可以这样做：

```bash
cd /root/xw/ModelTrain/minimind
pip install modelscope
modelscope download --dataset gongjy/minimind_dataset pretrain_t2t_mini.jsonl --local_dir ./dataset
modelscope download --dataset gongjy/minimind_dataset sft_t2t_mini.jsonl --local_dir ./dataset
```

如果你之后想玩 RL，再下载：

```bash
modelscope download --dataset gongjy/minimind_dataset dpo.jsonl --local_dir ./dataset
modelscope download --dataset gongjy/minimind_dataset rlaif.jsonl --local_dir ./dataset
modelscope download --dataset gongjy/minimind_dataset agent_rl.jsonl --local_dir ./dataset
```

### 9.3 跑预训练（Run Pretrain）

```bash
cd /root/xw/ModelTrain/minimind/trainer
python train_pretrain.py \
  --data_path ../dataset/pretrain_t2t_mini.jsonl \
  --epochs 1 \
  --batch_size 32 \
  --accumulation_steps 8 \
  --hidden_size 768 \
  --num_hidden_layers 8
```

输出文件通常会保存在：

```text
../out/pretrain_768.pth
```

### 9.4 跑全参数 SFT（Run Full SFT）

```bash
cd /root/xw/ModelTrain/minimind/trainer
python train_full_sft.py \
  --data_path ../dataset/sft_t2t_mini.jsonl \
  --from_weight pretrain \
  --epochs 1 \
  --batch_size 16 \
  --hidden_size 768 \
  --num_hidden_layers 8
```

输出文件通常会是：

```text
../out/full_sft_768.pth
```

### 9.5 评估与对话（Evaluate）

回到项目根目录：

```bash
cd /root/xw/ModelTrain/minimind
python eval_llm.py --load_from model --weight full_sft
```

如果想看显式思考：

```bash
python eval_llm.py --load_from model --weight full_sft --open_thinking 1
```

如果想测试 Tool Call：

```bash
cd /root/xw/ModelTrain/minimind/scripts
python eval_toolcall.py --load_from ../model --weight full_sft
```

### 9.6 一个推荐的学习顺序（Recommended Learning Order）

如果你是第一次读这个项目，建议按下面顺序学：

```text
1. model/model_minimind.py
2. dataset/lm_dataset.py
3. trainer/train_pretrain.py
4. trainer/train_full_sft.py
5. eval_llm.py
6. trainer/train_dpo.py
7. trainer/train_ppo.py / train_grpo.py
8. trainer/train_agent.py
9. scripts/serve_openai_api.py
```

这样你会从：

- 模型结构（model structure）
- 到数据模板（data templating）
- 到监督训练（supervised training）
- 再到强化学习（reinforcement learning）
- 最后到部署（deployment）

逐步建立完整心智模型（mental model）。

---

## 结语（Conclusion）

MiniMind 最适合用来做两件事：

1. **学原理（learn principles）**：理解现代 LLM 的关键模块是怎么拼起来的；
2. **做实验（run experiments）**：在低成本下验证 tokenizer、SFT、RL、Tool Use、Distillation 等想法。

如果你能独立看懂并跑通下面这条链：

```text
Tokenizer -> Transformer -> Pretrain -> SFT -> RL -> API Serve
```

那你对“小模型如何工作”就已经建立了非常扎实的工程理解。

> 对很多人来说，MiniMind 不是终点（destination），而是进入 LLM world 的一张真正可落地的门票（a practical ticket into LLM internals）。
