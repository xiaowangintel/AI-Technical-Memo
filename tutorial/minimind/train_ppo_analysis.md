# `train_ppo.py` 代码解析（Code Analysis）

## 1. 文件概述 (File Overview)

`minimind/trainer/train_ppo.py` 是 MiniMind 的 **PPO (Proximal Policy Optimization)** 式 RLHF 训练脚本。

- 它使用 **Actor-Critic** 架构：
  - `actor_model` 负责生成文本并更新策略。
  - `critic_model` 负责估计 value function（价值函数）。
- 脚本依赖 `rollout_engine` 在线生成回答，再结合 `reward_model` 和若干 reward shaping 规则形成奖励。
- 它实现了 PPO 中的几个关键部分：
  - **GAE (Generalized Advantage Estimation)**
  - **clipped surrogate loss**
  - **value loss with clipping**
  - **KL penalty**
  - **early stopping by KL**

---

## 2. 依赖说明 (Dependencies)

| 模块 | 作用 |
|---|---|
| `os`, `sys` | 调整导入路径。 |
| `argparse`, `math`, `re`, `warnings` | 参数、数学计算、正则、warning 管理。 |
| `torch`, `torch.distributed`, `torch.nn.functional` | PPO 训练的张量、分布式与损失计算。 |
| `AutoTokenizer` | 分词器接口。 |
| `nullcontext` | CPU 情况下替代 autocast。 |
| `optim`, `nn` | 优化器和神经网络模块。 |
| `DistributedDataParallel` | 多卡训练。 |
| `DataLoader`, `DistributedSampler` | 数据读取与分发。 |
| `clip_grad_norm_` | Actor/Critic 梯度裁剪。 |
| `CosineAnnealingLR` | Actor/Critic 学习率调度。 |
| `MiniMindConfig`, `MiniMindForCausalLM` | MiniMind 模型定义；`CriticModel` 基于它改造。 |
| `RLAIFDataset` | RLHF/RLAIF 数据集。 |
| `trainer_utils.*` | 日志、checkpoint、分布式、初始化模型、Reward model 封装。 |
| `create_rollout_engine` | 创建 rollout backend，支持 `torch` 或 `sglang`。 |

---

## 3. 逐行代码分析 (Line-by-Line Analysis)

> 按连续代码块覆盖全文件，重点解释 PPO 训练流程。

### 第 1-5 行：包路径设置

- 与其他训练脚本一致，确保直接运行时能导入项目内部模块。

### 第 7-24 行：导入依赖

- 导入 PPO 训练所需的 PyTorch、分布式、优化器、LR scheduler、Tokenizer、Reward model、rollout engine 等组件。

### 第 26 行：关闭 warning

- 保持日志清爽。

### 第 29-32 行：`rep_penalty`

这是一个 **重复惩罚 (repetition penalty)** 函数：

- 用正则把文本拆成 token/标点片段。
- 构造 `n-gram`（默认 3-gram）。
- 比较总数与去重后的数量，重复越多，惩罚越大。
- `cap=0.5` 限制最大惩罚，避免 reward 被单项规则完全主导。

### 第 35-49 行：`CriticModel`

这里定义 PPO 中的 **Critic**。

- 第 36 行：继承 `MiniMindForCausalLM`，复用底层 Transformer。
- 第 39-40 行：新增 `value_head = nn.Linear(hidden_size, 1)`，把每个位置的 hidden state 映射成标量 value。
- 第 42-48 行：`forward` 时只走 backbone，拿到隐藏层后通过 `value_head` 输出每个 token 的 value estimate。

这意味着 Critic 是 **token-level value function**，不是整句一个分数。

### 第 51-75 行：`calculate_rewards`

该函数把最终奖励拆成两类：

1. **规则奖励 (heuristic shaping rewards)**
2. **Reward model score**

具体逻辑：

- 第 52 行：初始化 reward tensor。
- 第 54 行：关闭梯度。
- 第 56-59 行：把 prompt 中的 chat template 解析成 `messages` 列表，供 reward model 使用。
- 第 60 行：默认把整个 response 当作 answer。
- 第 61 行：若回答长度在 `[20, 800]` 内，加 `+0.5`，否则 `-0.5`。这是 **length bonus / length penalty**。
- 第 62-66 行：如果回答里有 `</think>`，就把前半段视为 thinking content，后半段视为最终 answer：
  - thinking 长度合理则 `+1.0`，否则 `-0.5`
  - 恰好只出现一个 `</think>` 再加 `+0.25`，否则减分
  这是 **thinking structure reward**。
- 第 67 行：对 answer 扣除重复惩罚。
- 第 69-70 行：调用 `reward_model.get_score(messages, answer)` 计算语义层面的偏好分数。
- 第 72-73 行：把 reward model score 加到规则奖励上。

### 第 78-87 行：`ppo_train_epoch` 的初始化与 prompt 编码

- 第 79-80 行：开启 actor 与 critic 的训练模式。
- 第 81 行：`grad_accum_step` 用于梯度累积计数。
- 第 83-86 行：读取 batch 内的 prompt，并用 tokenizer 编码成左侧 padding 的张量。
  - `padding_side="left"` 适合 decoder-only 模型在 batch 中对齐生成起点。
  - `max_length=args.max_seq_len` 限制 prompt 长度。

### 第 88-100 行：在线 rollout 与奖励计算

- 第 88-94 行：调用 `rollout_engine.rollout(...)` 在线生成 response。
- 第 95-99 行：从 `RolloutResult` 中取出：
  - `output_ids`：完整 prompt+response 序列
  - `completion_ids`：仅 response 部分
  - `prompt_lens`：每个样本 prompt 实际长度
  - `completions`：文本形式回答
  - `old_resp_logp`：rollout 时旧策略对 response 各 token 的 log-prob
- 第 100 行：调用 `calculate_rewards` 得到样本级 reward。

这里的 `old_resp_logp` 对应 PPO 里的 **old policy**。

### 第 102-114 行：调试打印

- 当 `debug_mode=True` 且达到设定间隔时，打印 prompt、response、长度和 reward，方便检查 rollout 质量与奖励合理性。

### 第 116-127 行：构造 response 区间 mask

这一段把“完整序列”切成 PPO 真正要优化的 response token 区间。

- 第 116 行：`full_mask` 表示整条序列中非 pad 的位置。
- 第 117 行：`labels = gen_out[:, 1:]`，供后面按 next-token 方式取 log-prob。
- 第 119-121 行：利用 `prompt_lens` 构造 response token 在整条序列中的位置索引 `logp_pos`。
- 第 122 行：`resp_pad_mask` 是 rollout 引擎返回的 response 有效位。
- 第 123-125 行：定位 EOS，得到真实 response 长度。
- 第 126-127 行：分别构造 policy/value 的有效 mask。

### 第 129-150 行：旧 value、参考 log-prob、token reward 与 GAE

这是 PPO 的核心一段。

- 第 129-132 行：用 critic 对完整生成序列估值，再取 response 区间的 value，得到 `old_resp_values`。
- 第 134 行：用 `ref_model` 计算 response token 的 reference log-prob。
- 第 135 行：初始化逐 token 奖励 `token_rewards`。
- 第 136-137 行：把外部奖励只加到 response 最后一个 token 上，即 sparse terminal reward。
- 第 139-145 行：反向循环实现 **GAE**：
  - `delta_t = r_t + gamma * V_{t+1} - V_t`
  - `A_t = delta_t + gamma * lambda * A_{t+1}`
- 第 146 行：`returns = advantages + old_resp_values`，这是 critic 监督目标。
- 第 148-150 行：按 mask 对 advantage 做均值方差归一化，提升训练稳定性。

### 第 152-160 行：mini-batch 更新前的统计量初始化

- 计算 `mb_size`，准备做 PPO 的多轮 mini-batch 更新。
- `stop_ppo` 用于 KL 过大时提前停止当前 batch 的 PPO 更新。
- 初始化一系列日志累计变量，如 `policy_loss_sum`、`value_loss_sum`、`kl_sum`、`clipfrac_sum` 等。

### 第 161-178 行：mini-batch 内重新计算 actor/critic 输出

- 第 161-162 行：拿到未封装版本 actor/critic，便于直接 forward。
- 第 163-167 行：外层循环 `ppo_update_iters` 表示对同一批 rollout 数据重复优化多次。
- 第 166-168 行：打乱 batch 索引，按 mini-batch 切分。
- 第 170-171 行：critic 重新计算当前 mini-batch 的 value。
- 第 173-176 行：actor 前向，顺带拿 MoE 的 `aux_loss`。
- 第 177 行：重新计算当前策略的 response log-prob。

这一步得到的是 PPO 中的 **new policy / new value**。

### 第 179-203 行：PPO 损失、value clipping、KL penalty、early stopping

- 第 179 行：`log_ratio = new_logp - old_logp`。
- 第 180 行：`approx_kl = 0.5 * log_ratio^2` 的 masked 平均，用于近似 KL。
- 第 182-185 行：若使用 DDP，对 `approx_kl` 在各卡取平均，避免某些卡提前退出、某些卡继续 backward 导致死锁。
- 第 187-188 行：若 `approx_kl > early_stop_kl`，触发 PPO early stopping。
- 第 190 行：`ratio = exp(log_ratio)`，即 importance sampling ratio。
- 第 191-192 行：统计 `clipfrac`，表示有多少 token 的 ratio 被裁剪了。
- 第 193-194 行：计算相对 reference model 的 KL 惩罚：
  \
  e^{(\log p_{ref} - \log p_\theta)} - (\log p_{ref} - \log p_\theta) - 1
  \
  这是一个常见的非负 KL-like penalty 形式。
- 第 195-198 行：`policy_loss` 是 **clipped surrogate objective** 的负号版本，再加上 `kl_coef * kl_ref_penalty`。
- 第 199-202 行：`value_loss` 使用 clipped value function：
  - 比较 `new_value` 与 `return`
  - 再比较裁剪后 `new_value` 与 `return`
  - 取二者较大值，防止 critic 一步更新过猛

### 第 204-243 行：反向传播、梯度累积与参数更新

- 第 204-205 行：保存 KL 指标供日志使用。
- 第 207-212 行：如果触发 `stop_ppo`，不立即 break，而是把 loss 乘以 0；这样仍能走完整的 forward-backward 通信闭环，避免 DDP 卡死。
- 第 213 行：`loss.backward()`。
- 第 215-221 行：累计日志统计。
- 第 223 行：累计梯度步数。
- 第 225-233 行：达到梯度累积步数后，同时更新 actor 和 critic，并推进各自 scheduler。
- 第 235-243 行：处理循环结束后剩余的未更新梯度。

### 第 245-293 行：同步 rollout 引擎、日志、保存 checkpoint、清理张量

- 第 245 行：定期调用 `rollout_engine.update_policy(actor_model)`，把最新 actor 同步给 rollout backend。
- 第 247-271 行：主进程打印并记录 reward、KL、clipfrac、critic loss、response 长度、学习率等指标。
- 第 273-288 行：保存 actor 权重和完整 checkpoint；checkpoint 中还包括 critic、各自 optimizer/scheduler。
- 第 290-292 行：删除当前 batch 大张量，释放显存。

### 第 295-338 行：命令行参数

这里定义 PPO 训练超参数，核心包括：

- 训练参数：`epochs`、`batch_size`、`learning_rate`、`critic_learning_rate`
- PPO 参数：`clip_epsilon`、`vf_coef`、`kl_coef`
- GAE 参数：`gamma`、`lam`
- Critic 裁剪参数：`cliprange_value`
- PPO 更新控制：`ppo_update_iters`、`early_stop_kl`、`mini_batch_size`
- 奖励相关：`reward_model_path`、`thinking_ratio`
- rollout 相关：`rollout_engine`、`sglang_base_url`、`sglang_model_path`、`sglang_shared_path`

### 第 340-398 行：训练前初始化

- 第 341-343 行：初始化分布式与随机种子。
- 第 346-348 行：创建保存目录、模型配置、尝试读取 checkpoint。
- 第 351-353 行：配置 autocast mixed precision。
- 第 356-362 行：初始化实验日志。
- 第 365-369 行：初始化 actor 与 frozen reference model。
- 第 370-375 行：从基础权重加载一个 `CriticModel`；由于 `strict=False`，语言模型权重能对齐加载，`value_head` 会保留随机初始化。
- 第 376 行：初始化 reward model。
- 第 378-387 行：通过工厂函数创建 rollout engine。
- 第 388-397 行：构建数据集、优化器、统计总 step 数，并为 actor/critic 分别配置 cosine scheduler。

### 第 399-418 行：恢复训练、可选 compile 与 DDP

- 第 400-408 行：如果发现 checkpoint，则恢复 actor/critic、优化器、scheduler 和进度。
- 第 411-414 行：若启用 `torch.compile`，先编译 actor，再把 compiled actor 同步给 rollout engine。
- 第 415-417 行：分布式下包装 actor 和 critic。
- 第 418 行：再次 `rollout_engine.update_policy(actor_model)`，确保 rollout 使用当前最新 actor。

### 第 420-434 行：epoch 循环与收尾

- 每个 epoch 都重置 sampler 与随机种子。
- 若从中间 step 恢复，则使用 `SkipBatchSampler` 跳过已完成 batch。
- 调用 `ppo_train_epoch(...)` 执行主训练逻辑。
- 最后一行销毁分布式进程组。

---

## 4. 关键算法解析 (Key Algorithm Deep-Dive)

### 4.1 CriticModel：为什么 PPO 需要 Critic

PPO 需要估计状态价值 `V(s_t)`，以便：

- 计算 advantage
- 降低策略梯度方差
- 训练更稳定

本文件中 `CriticModel` 直接复用语言模型 backbone，只把输出头改成 `value_head`，从而为每个 token 位置输出一个 scalar value。

### 4.2 GAE(γ, λ)

脚本实现的 GAE 为：

\
\delta_t = r_t + \gamma V(s_{t+1}) - V(s_t)
\
\
A_t = \delta_t + \gamma \lambda A_{t+1}
\

对应代码在第 139-145 行：

- 从后向前遍历 response token
- 用 `lastgaelam` 递推累积 advantage
- 再通过 `returns = advantages + old_resp_values` 构造 critic 监督目标

这里外部奖励只加在最后一个 token 上，因此 GAE 会把 terminal reward 向前传播到整段 response。

### 4.3 Clipped surrogate loss

PPO 的策略目标核心是：

\
r_t(\theta) = \frac{\pi_\theta(a_t|s_t)}{\pi_{old}(a_t|s_t)} = \exp(\log \pi_\theta - \log \pi_{old})
\

\
L^{clip} = \min(r_t A_t, \text{clip}(r_t, 1-\epsilon, 1+\epsilon) A_t)
\

脚本在第 195-198 行用 `torch.max(-A*ratio, -A*clipped_ratio)` 的形式实现负损失。其本质是同一个 PPO clipped surrogate objective。

### 4.4 Value function clipping

Critic 也可能更新过猛，因此第 199-202 行额外做了 **value clipping**：

- 一个分支用 `new_value` 直接回归 `returns`
- 另一个分支先把 `new_value` 限制在 `old_value ± cliprange_value`
- 最终取较大者，得到更保守的 critic 更新

这与 OpenAI/TRL 等 PPO 实现中的思路一致。

### 4.5 KL divergence penalty 与 early stopping

脚本同时用了两种 KL 控制：

1. **训练目标中的 KL penalty**：
   - 第 193-198 行把 `kl_ref_penalty` 加进 `policy_loss`
   - 作用是限制 actor 偏离 reference model 太远

2. **优化过程中的 early stopping**：
   - 第 180-188 行计算 `approx_kl`
   - 若超过 `early_stop_kl`，就停止该 batch 的后续 PPO 强更新

这种“双保险”在 RLHF 中很常见：一个软约束 (soft penalty)，一个硬阈值 (hard stop)。

### 4.6 Reward shaping：长度奖励、思维结构奖励、重复惩罚、Reward model 评分

本文件的最终奖励为：

```text
reward = length bonus/penalty
       + thinking structure reward/penalty
       - repetition penalty
       + reward model score
```

各项含义：

- **length bonus**：鼓励输出不要过短或过长。
- **thinking structure reward**：鼓励模型输出符合 `<think> ... </think>` 风格的可解释推理结构。
- **repetition penalty**：抑制答案中重复片段过多。
- **reward model score**：提供更高层语义偏好信号。

---

## 5. 调用关系 (Call Graph)

### 5.1 本文件内部调用链

```text
main
├─ init_model -> actor_model
├─ init_model -> ref_model
├─ CriticModel(...)
├─ LMForRewardModel(...)
├─ create_rollout_engine(...)
└─ ppo_train_epoch
   ├─ rollout_engine.rollout(...) -> 生成 response + old_logp
   ├─ calculate_rewards(...) -> shaping reward + reward model score
   ├─ critic_model(...) -> old/new values
   ├─ ref_model(...) -> ref log-prob
   ├─ actor_model(...) -> new policy log-prob
   ├─ GAE advantages / returns
   ├─ PPO policy loss + value loss + KL penalty
   └─ rollout_engine.update_policy(actor_model)
```

### 5.2 与 `rollout_engine.py` 的关系

```text
train_ppo.py
├─ import create_rollout_engine
├─ rollout_engine.rollout(): 负责在线采样
└─ rollout_engine.update_policy(): 负责把最新 actor 同步给采样端
```

若选择：

- `torch`：直接在当前进程内调用模型 `generate`
- `sglang`：通过 HTTP API 调用外部 SGLang 服务

### 5.3 与 `train_grpo.py` 的关系

- `train_ppo.py` 和 `train_grpo.py` 都依赖 `rollout_engine.py`。
- 二者都做 **在线生成 + reward shaping + reference KL regularization**。
- 区别在于：
  - PPO：有 `CriticModel`，使用 GAE 与 value loss。
  - GRPO：没有 critic，使用 group-relative normalized rewards 近似 advantage。
