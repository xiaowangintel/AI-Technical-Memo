# `train_grpo.py` 代码解析（Code Analysis）

## 1. 文件概述 (File Overview)

`minimind/trainer/train_grpo.py` 实现的是 **GRPO (Group Relative Policy Optimization)**，并支持一个变体 **CISPO**。

- 和 PPO 一样，它属于 **online RLHF / RLAIF**：需要先对 prompt 进行 rollout，再根据 reward 更新策略。
- 和 PPO 不同，它**不训练 critic**，而是对同一个 prompt 下的多个候选回答做组内归一化，直接构造 advantage。
- 本文件的两个重点：
  - **group-relative advantages**（组内均值/方差标准化）
  - **CISPO upper-bounded ratio**（上界裁剪的重要性比率）

---

## 2. 依赖说明 (Dependencies)

| 模块 | 作用 |
|---|---|
| `os`, `sys` | 修正包路径。 |
| `argparse`, `math`, `re`, `gc`, `warnings` | 参数、数学、正则、垃圾回收、warning 控制。 |
| `torch`, `F`, `dist` | 张量计算、log-prob、分布式训练。 |
| `AutoTokenizer`, `AutoModel` | tokenizer / 模型接口；本文件主要显式用 tokenizer。 |
| `nullcontext` | CPU 环境下的空 autocast。 |
| `optim`, `DistributedDataParallel`, `DataLoader`, `DistributedSampler`, `CosineAnnealingLR` | 优化、并行、数据加载、学习率调度。 |
| `MiniMindConfig`, `MiniMindForCausalLM` | MiniMind 模型定义。 |
| `RLAIFDataset` | RLAIF 训练样本集。 |
| `trainer_utils.*` | logger、checkpoint、随机种子、模型初始化、Reward model。 |
| `create_rollout_engine` | 可插拔 rollout backend。 |

---

## 3. 逐行代码分析 (Line-by-Line Analysis)

### 第 1-5 行：包路径设置

- 和其他 trainer 脚本相同，为直接脚本执行提供 import 路径支持。

### 第 7-25 行：导入依赖

- 这一段导入 GRPO 所需的数值计算、分布式、优化器、Reward model、rollout engine 等组件。
- `gc` 在当前文件中没有实际使用，属于预留/遗留导入。
- `AutoModel` 也没有在本文件主体中使用，可能是早期版本遗留。

### 第 27 行：关闭 warning

- 与其他训练脚本保持一致。

### 第 30-33 行：`rep_penalty`

- 与 PPO 脚本相同，计算文本重复惩罚。
- 通过 n-gram 重复率估计生成文本是否啰嗦或循环。

### 第 36-67 行：`calculate_rewards`

这个函数是 GRPO 的奖励入口。与 PPO 版本类似，但这里要处理 **每个 prompt 的多个 generations**。

- 第 37 行：初始化 `[B*num_gen]` 的 reward 向量。
- 第 39-40 行：关闭梯度并准备收集 reward model 分数。
- 第 41 行：`batch_size = len(prompts)`。
- 第 43-45 行：双层循环，外层遍历 prompt，内层遍历该 prompt 下的多个生成结果。
- 第 46-47 行：通过 `response_idx = i * num_generations + j` 找到扁平化后的回答。
- 第 49-51 行：把 prompt 解析成 chat messages。
- 第 53 行：长度在 `[20, 800]` 范围内给长度奖励，否则惩罚。
- 第 54-58 行：若包含 `</think>`，就对 thinking 片段长度和结构完整性进行奖励/惩罚。
- 第 59 行：对最终 answer 扣除重复惩罚。
- 第 61-62 行：用 reward model 对 answer 打分。
- 第 64-65 行：把 reward model 分数叠加到 shaping reward 上。
- 第 67 行：返回总奖励。

### 第 70-77 行：`grpo_train_epoch` 开始与 prompt 编码

- 第 71 行：按 step 遍历训练 batch。
- 第 72 行：从数据集中取出 prompt 文本列表。
- 第 73-74 行：用 tokenizer 编码，并使用 left padding。
- 第 75-77 行：若 prompt 太长，则只保留最后 `max_seq_len` 个 token，避免上下文过长影响 rollout。

### 第 79-90 行：一次 prompt 生成多个回答

- 第 79-85 行：调用 `rollout_engine.rollout(...)`，其中 `num_generations=args.num_generations` 是 GRPO 的关键。
- 第 86-90 行：从 rollout 结果中提取完整输出、completion ids、文本回答、旧策略 log-prob 和 prompt 长度。

与 PPO 不同，GRPO 会对同一个 prompt 一次性拿到多条候选回答，再做组内比较。

### 第 91-94 行：mask、位置索引与奖励

- 第 91 行：`full_mask` 标记完整序列中的非 pad token。
- 第 92 行：`logp_pos` 定位 response token 在整条序列中的位置。
- 第 94 行：调用 `calculate_rewards(...)` 计算 `[B*num_gen]` 的总奖励。

### 第 96-103 行：当前策略与参考策略的逐 token log-prob

- 第 96 行：取出未包装的当前策略模型。
- 第 97-100 行：在 autocast 下前向当前 policy，计算 response token 的 `per_token_logps`。
- 第 99 行：若使用 MoE，则取 `aux_loss`，否则置 0。
- 第 102-103 行：在 `torch.no_grad()` 下计算 `ref_per_token_logps`。

### 第 105-119 行：调试输出

- 当 debug 模式打开时，打印每个 prompt 及其多条 generation 的文本和 reward。
- 这对检查组内 reward 分布是否合理特别有帮助。

### 第 120-123 行：组相对优势 (group-relative advantages)

这是 GRPO 的核心。

- 第 120 行：把扁平化奖励 reshape 成 `[B, num_gen]`。
- 第 121 行：计算每组（同一个 prompt）的平均奖励，并 repeat 回 `[B*num_gen]`。
- 第 122 行：计算组内标准差，并 repeat 回去。
- 第 123 行：`advantages = (rewards - mean_r) / (std_r + 1e-4)`。

这表示：某条回答的 advantage 不是看“绝对奖励”，而是看“它在同组里比其它回答好多少”。

### 第 125-129 行：completion 有效范围与 EOS 截断

- 第 125 行：把 rollout 返回的 completion mask 转成 bool。
- 第 126 行：定位 response 中的 EOS。
- 第 127-128 行：若存在 EOS，则取第一个 EOS 的位置；否则默认最后一个有效 token。
- 第 129 行：构造 `completion_mask`，保证只对 EOS 前的有效 token 计算损失。

### 第 131-143 行：KL、ratio、GRPO/CISPO 损失

这里是 GRPO 训练目标的核心。

- 第 131 行：`kl_div = ref_logp - policy_logp`。
- 第 132 行：`per_token_kl = exp(kl_div) - kl_div - 1`，作为 reference KL penalty。
- 第 133 行：`ratio = exp(new_logp - old_logp)`，importance ratio。
- 第 134-137 行：如果 `loss_type == "cispo"`：
  - 第 135 行：`torch.clamp(ratio, max=epsilon_high).detach()`，只保留上界并把 ratio 从梯度图中分离。
  - 第 136 行：损失写成 `-(clamped_ratio * A * logp - beta * KL)`。
  这体现了 CISPO 的“上界控制、偏保守”风格。
- 第 138-141 行：否则走标准 GRPO/PPO 风格的 clipped ratio objective。
- 第 142 行：按 token mask 求序列平均，再对 batch 求均值，得到 `policy_loss`。
- 第 143 行：加上 `aux_loss` 并按 `accumulation_steps` 缩放。

### 第 144-152 行：反向传播与参数更新

- 第 144 行：直接 `loss.backward()`。
- 第 146-151 行：达到梯度累积步数时进行梯度裁剪、`optimizer.step()`、`scheduler.step()` 和 `zero_grad()`。

### 第 153-177 行：训练日志

- 第 154-161 行：整理 `policy_loss`、`aux_loss`、平均奖励、平均长度、KL、advantage 统计和学习率。
- 第 163-166 行：打印日志。
- 第 168-177 行：如启用 wandb，则记录这些指标。

值得注意的是：
- `advantages_mean` 理论上应接近 0
- `advantages_std` 理论上接近 1
这是 group normalization 正常工作的一个信号。

### 第 179-192 行：保存模型并同步 rollout policy

- 第 179-190 行：主进程保存策略权重和 checkpoint。
- 第 192 行：无论是否保存 checkpoint，只要达到间隔，就调用 `rollout_engine.update_policy(model)`，让 rollout 使用最新策略。

### 第 194-202 行：显存清理与尾部更新

- 第 194-195 行：删除当前 step 的大张量。
- 第 197-202 行：处理 epoch 末尾尚未触发 step 的梯度累积残留。

### 第 205-243 行：命令行参数

本段定义 GRPO/CISPO 训练参数：

- 常规参数：保存路径、epoch、batch size、LR、dtype、workers 等。
- 生成参数：`max_seq_len`、`max_gen_len`、`num_generations`。
- GRPO 参数：`beta`、`loss_type`、`epsilon`。
- CISPO 参数：`epsilon_high`。
- Reward / 数据相关：`reward_model_path`、`thinking_ratio`、`data_path`。
- rollout backend：`rollout_engine`、SGLang 相关 URL/路径。

### 第 245-259 行：环境与混合精度初始化

- 初始化分布式、随机种子、保存目录、模型配置与 checkpoint。
- 这里 `max_seq_len` 被设置为 `prompt_len + gen_len` 的总长度，便于完整序列前向。
- 配置 autocast 精度上下文。

### 第 261-268 行：wandb 初始化

- 仅主进程开启实验记录。
- 若 checkpoint 中带有旧的 `wandb_id`，则继续接续记录。

### 第 270-297 行：初始化 policy、reference、reward、rollout、数据与优化器

- 第 273 行：初始化 policy model。
- 第 275-276 行：初始化并冻结 reference model。
- 第 278 行：初始化 reward model。
- 第 280-289 行：通过工厂函数创建 rollout engine。
- 第 291 行：构建 `RLAIFDataset`，其中 `thinking_ratio` 会影响 prompt/response 模板中的 thinking 风格分布。
- 第 293 行：创建 `AdamW`。
- 第 294-297 行：计算总迭代数，并设置 cosine scheduler。

### 第 299-315 行：恢复训练、compile、DDP 与 rollout 同步

- 恢复模型、优化器、scheduler、epoch 与 step。
- 若开启 `torch.compile`，需要立即把 compiled model 同步给 rollout engine。
- 若使用 DDP，则包装 model。
- 第 315 行再次调用 `rollout_engine.update_policy(model)`，确保 rollout 端始终拿到最新 policy。

### 第 317-331 行：epoch 训练与清理进程

- 每个 epoch 重置 sampler 和随机种子。
- 如需续训则跳过已处理 batch。
- 调用 `grpo_train_epoch(...)` 执行主逻辑。
- 结束后销毁分布式进程组。

---

## 4. 关键算法解析 (Key Algorithm Deep-Dive)

### 4.1 Group-relative advantages

对每个 prompt，GRPO 会生成 `G = num_generations` 条回答，并得到奖励 `r_1, ..., r_G`。

然后用组内标准化构造 advantage：

\
A_i = \frac{r_i - \mu_g}{\sigma_g + \epsilon}
\

其中：

- `μ_g`：同一 prompt 对应这组回答的平均奖励
- `σ_g`：同组标准差

直观上：

- 如果一条回答比同组其它回答好，它 advantage 为正
- 如果它只是“绝对分数高”，但组内都很高，那么优势未必大

这让训练更关注 **相对排序 (relative ranking)**，而不是 reward 的绝对标尺。

### 4.2 为什么 GRPO 不需要 Critic

PPO 需要 critic 来估计 `V(s)`；GRPO 则直接利用“同一 prompt 下多样本比较”构造 advantage。

所以它省去了：

- value head
- value loss
- GAE 递推

代价是：

- 需要一次生成多个回答，增加 rollout 开销
- 优势估计更依赖组内样本质量

### 4.3 CISPO：upper-bounded ratio

本文件支持 `loss_type="cispo"`。核心做法是：

- 先计算 `ratio = exp(new_logp - old_logp)`
- 再只做**上界裁剪**：`clamp(ratio, max=epsilon_high)`
- 并 `detach()`，让 ratio 作为常量权重，而不是标准 PPO 那样直接参与梯度

这意味着：

- 当新策略相对旧策略的概率放大过多时，更新被强行限制
- 训练倾向更保守，减少某些 token 因 ratio 过大导致的不稳定放大

### 4.4 Reference KL penalty

第 131-142 行中的 KL 项：

\
\text{KL-penalty} = e^{(\log p_{ref} - \log p_\theta)} - (\log p_{ref} - \log p_\theta) - 1
\

它是一个始终非负的惩罚项，用于抑制策略远离参考模型过快，从而减少 reward hacking 和语言质量退化。

### 4.5 Reward shaping：长度奖励、thinking structure、重复惩罚、Reward model score

本文件的 reward 由四部分叠加：

```text
reward = length bonus/penalty
       + thinking structure reward/penalty
       - repetition penalty
       + reward model score
```

各项作用：

- **length bonus**：让输出长度落在经验合理区间。
- **thinking structure reward**：鼓励结构化推理输出，例如 `</think>` 边界正确、thinking 部分不过短也不过长。
- **repetition penalty**：避免答案循环、堆砌、复读。
- **reward model score**：提供更语义化的好坏判断。

---

## 5. 调用关系 (Call Graph)

### 5.1 本文件内部调用链

```text
main
├─ init_model -> policy model
├─ init_model -> reference model
├─ LMForRewardModel(...)
├─ create_rollout_engine(...)
└─ grpo_train_epoch
   ├─ rollout_engine.rollout(..., num_generations=G)
   ├─ calculate_rewards(prompts, completions, reward_model)
   ├─ policy model(outputs) -> per_token_logps
   ├─ ref_model(outputs) -> ref_per_token_logps
   ├─ grouped reward normalization -> advantages
   ├─ GRPO/CISPO loss + KL penalty
   └─ rollout_engine.update_policy(model)
```

### 5.2 与 `rollout_engine.py` 的关系

- `train_grpo.py` 通过 `create_rollout_engine(...)` 获取采样后端。
- rollout engine 只负责：
  - 给定 prompt 生成 completion
  - 返回旧策略 log-prob
  - 在策略更新后同步最新权重
- 真正的 reward 计算、group normalization、loss 构造都在 `train_grpo.py` 中完成。

### 5.3 与 `train_ppo.py` 的关系

```text
rollout_engine.py
├─ train_ppo.py 使用：单样本 rollout + critic + GAE
└─ train_grpo.py 使用：多样本 rollout + group-relative advantages
```

两者共同点：

- 都有 reference model KL regularization
- 都有 reward shaping
- 都要在训练后 `update_policy`

差异点：

- PPO：`rollout -> reward -> critic -> GAE -> PPO loss`
- GRPO：`rollout -> reward -> group normalize -> GRPO/CISPO loss`
