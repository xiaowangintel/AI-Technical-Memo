# `train_dpo.py` 代码解析（Code Analysis）

## 1. 文件概述 (File Overview)

`minimind/trainer/train_dpo.py` 是 MiniMind 的 **DPO (Direct Preference Optimization)** 训练脚本。

- 它基于 **偏好数据 (preference data)** 训练策略模型 (policy model)，不需要在线采样 (online rollout) 或显式奖励模型 (reward model)。
- 核心思想是：让策略模型在同一个 prompt 上，对 **chosen response** 的相对偏好高于 **rejected response**，同时用 **reference model** 约束更新幅度。
- 该脚本实现的是经典 DPO 目标：
  \
  L_{DPO} = - \log \sigma\left(\beta \left[(\log \pi_\theta(y_w|x)-\log \pi_\theta(y_l|x))-(\log \pi_{ref}(y_w|x)-\log \pi_{ref}(y_l|x))\right]\right)
  \

---

## 2. 依赖说明 (Dependencies)

| 模块 | 作用 |
|---|---|
| `os`, `sys` | 处理路径，保证脚本可直接运行并找到上级包。 |
| `argparse` | 解析命令行参数 (CLI arguments)。 |
| `time` | 统计训练耗时与 ETA。 |
| `warnings` | 关闭部分 warning 输出。 |
| `torch` | PyTorch 主框架。 |
| `torch.nn.functional as F` | 提供 `log_softmax`、`logsigmoid` 等函数。 |
| `torch.distributed as dist` | 分布式训练 (distributed training)。 |
| `nullcontext` | CPU 环境下替代 autocast context。 |
| `optim` | 优化器，这里主要是 `AdamW`。 |
| `DistributedDataParallel` | DDP 并行包装模型。 |
| `DataLoader`, `DistributedSampler` | 构建训练数据迭代器。 |
| `MiniMindConfig` | 配置 MiniMind 模型结构。 |
| `DPODataset` | 读取 DPO 偏好数据，提供 chosen/rejected 样本。 |
| `trainer_utils.*` | 学习率调度、日志、checkpoint、随机种子、模型初始化等工具函数。 |

---

## 3. 逐行代码分析 (Line-by-Line Analysis)

> 说明：这里按 **连续代码块 (line blocks)** 解析，覆盖整份脚本的所有行。

### 第 1-5 行：包路径修正

- `import os`, `import sys`：导入系统与路径工具。
- `__package__ = "trainer"`：显式声明当前脚本所在包。
- `sys.path.append(...)`：把上一级目录加入 Python 搜索路径，便于直接运行脚本时正常导入 `model/`、`dataset/`、`trainer/` 下模块。

### 第 7-20 行：导入训练所需依赖

- 导入参数解析、计时、warning 控制。
- 导入 PyTorch 主体、函数式 API、分布式 API。
- 导入混合精度上下文 `nullcontext`、优化器、DDP、DataLoader。
- 导入 `MiniMindConfig`、`DPODataset` 与一组训练辅助函数。

### 第 21 行：关闭 warning

- `warnings.filterwarnings('ignore')`：减少终端噪声，方便观察训练日志。

### 第 24-30 行：`logits_to_log_probs`

这个函数把模型输出的 **logits** 转成标签 token 对应的 **log probabilities**。

- 第 24 行：定义函数，输入是 `logits` 与 `labels`。
- 第 25-27 行：注释说明 shape。
- 第 28 行：`F.log_softmax(logits, dim=2)` 把 vocab 维度转成对数概率分布。
- 第 29 行：`torch.gather(...)` 取出每个位置真实标签 token 的 log-prob。
- 第 30 行：返回 `[batch_size, seq_len]` 形状的逐 token log-prob。

这一步很关键，因为 DPO 不直接比较 raw logits，而是比较 **chosen/rejected 序列的对数似然 (sequence log-likelihood)**。

### 第 33-49 行：`dpo_loss`

这是 DPO 的核心实现。

- 第 33 行：定义损失函数，输入参考模型与策略模型的逐 token log-prob、mask 和 `beta`。
- 第 34 行：注释说明输入 shape。
- 第 35 行：`(ref_log_probs * mask).sum(dim=1)`，把逐 token log-prob 根据有效 mask 聚合成整条序列的 log-prob。
- 第 36 行：对 policy 做同样的聚合。
- 第 38-43 行：把 batch 前半部分当作 `chosen`，后半部分当作 `rejected`。这要求数据拼接顺序固定为 chosen 在前、rejected 在后。
- 第 45 行：`pi_logratios = chosen_policy - reject_policy`，表示当前策略对优选答案的相对偏好。
- 第 46 行：`ref_logratios = chosen_ref - reject_ref`，表示参考模型的原始偏好。
- 第 47 行：`logits = pi_logratios - ref_logratios`，即 **策略相对参考模型的偏好提升值**。
- 第 48 行：`-F.logsigmoid(beta * logits)`，实现 DPO 的 logistic loss。
- 第 49 行：对 batch 求均值。

数学上，它对应：
\
-\log \sigma\left(\beta[(\log \pi_\theta(y_w|x)-\log \pi_\theta(y_l|x))-(\log \pi_{ref}(y_w|x)-\log \pi_{ref}(y_l|x))]\right)
\

### 第 52-54 行：`train_epoch` 初始化

- 定义单个 epoch 的训练函数。
- `start_time` 用于日志耗时统计。
- `last_step` 用于处理 epoch 末尾未整除的梯度累积。

### 第 56-66 行：读取并拼接 chosen/rejected batch

- 第 56 行：从 `start_step + 1` 开始枚举，支持断点续训。
- 第 58-63 行：把 chosen/rejected 的输入、标签、mask 都迁移到训练设备。
- 第 64-66 行：分别沿 batch 维拼接 chosen 与 rejected。

这样后续一次 forward 就能同时得到 chosen/rejected 的对数概率，减少两次单独推理的开销。

### 第 68-70 行：动态学习率

- 调用 `get_lr(...)` 根据当前全局步数计算学习率。
- 遍历 `optimizer.param_groups` 更新实际学习率。

### 第 72-85 行：参考模型 + 策略模型前向计算

- 第 72 行：进入 autocast mixed precision 上下文。
- 第 73-76 行：`torch.no_grad()` 下运行参考模型 `ref_model`，得到 `ref_logits`，避免反向传播。
- 第 76 行：把参考模型输出转成逐 token log-prob。
- 第 78-80 行：运行当前策略模型 `model(x)`，得到 `policy_log_probs`。
- 第 82 行：调用 `dpo_loss(...)` 计算 DPO 主损失。
- 第 83 行：把 `outputs.aux_loss` 加进去。若模型是 MoE (Mixture-of-Experts)，这里通常是负载均衡辅助损失。
- 第 84 行：按 `accumulation_steps` 缩放，服务于梯度累积。

### 第 86-93 行：反向传播与优化器更新

- 第 86 行：`scaler.scale(loss).backward()` 使用 GradScaler 进行混合精度反传。
- 第 88 行：达到梯度累积步数时再执行参数更新。
- 第 89 行：`scaler.unscale_(optimizer)` 把梯度还原，便于做梯度裁剪。
- 第 90 行：`clip_grad_norm_` 防止梯度爆炸。
- 第 91-92 行：`scaler.step/update` 完成 optimizer step 与 scaler 状态更新。
- 第 93 行：清空梯度。

### 第 95-105 行：训练日志

- 按 `log_interval` 打印一次训练状态。
- `current_loss`、`current_dpo_loss`、`current_aux_loss` 分别表示总损失、DPO 主损失、辅助损失。
- `eta_min` 用简单平均速度估算剩余时间。
- 若启用 `wandb`（这里实际用的是 `swanlab as wandb`），同步记录指标。

### 第 107-117 行：保存权重与 checkpoint

- 仅主进程保存，防止多卡重复写盘。
- 先切到 `eval()`，减少不必要状态变化。
- 根据是否使用 MoE 决定保存文件名后缀。
- 若模型被 DDP 或 `torch.compile` 包装，需要先解包到原始模型。
- `torch.save({k: v.half().cpu() ...})`：把参数转成 half 并搬到 CPU 再存盘，节省显存与磁盘。
- `lm_checkpoint(...)`：额外保存优化器、scaler、epoch、step 等可恢复状态。
- 保存后切回 `train()`。

### 第 119-127 行：显存释放与尾部梯度处理

- 第 119-120 行：手动 `del` 掉大张量，帮助显存回收。
- 第 122-127 行：如果 epoch 结束时还有尚未触发 step 的累积梯度，则补一次 optimizer 更新。

### 第 130-155 行：命令行参数定义

这一段定义训练超参数：

- 保存路径、权重前缀、epoch、batch size。
- `learning_rate`：默认非常小，注释说明是为了避免 preference tuning 时的 catastrophic forgetting（灾难性遗忘）。
- `dtype`、`num_workers`、`accumulation_steps`、`grad_clip` 等训练控制参数。
- 模型规模参数：`hidden_size`、`num_hidden_layers`、`max_seq_len`、`use_moe`。
- 数据与权重来源：`data_path`、`from_weight`、`from_resume`。
- DPO 专有参数：`beta`。
- 实验工程参数：`use_wandb`、`use_compile`。

### 第 157-170 行：环境初始化

- 第 158 行：初始化分布式模式，返回 `local_rank`。
- 第 159 行：若 DDP 已启用，则把设备设置到对应 GPU。
- 第 160 行：按 rank 偏移随机种子，保证不同进程的随机性可控。
- 第 163 行：创建保存目录。
- 第 164 行：构建 `MiniMindConfig`。
- 第 165 行：若允许断点恢复，则读取已有 checkpoint 元数据。
- 第 168-170 行：根据设备与 `dtype` 选择 autocast 上下文。

### 第 173-179 行：实验日志系统

- 只有主进程初始化 `wandb`。
- 从 checkpoint 里读取 `wandb_id`，支持 resume。
- 运行名包含 epoch、batch size、LR，方便区分实验。

### 第 181-193 行：初始化策略模型、参考模型与数据

- 第 182 行：加载待训练的 policy model 与 tokenizer。
- 第 183 行：记录策略模型参数量。
- 第 185-187 行：再加载一份相同初始权重作为 `ref_model`，并设为 `eval()+requires_grad_(False)`，保证参考分布固定不更新。
- 第 190 行：构建 `DPODataset`。
- 第 191 行：若分布式训练，则使用 `DistributedSampler`。
- 第 192 行：构建 `GradScaler`。只有 `float16` 时启用。
- 第 193 行：创建 `AdamW` 优化器。

### 第 195-223 行：恢复训练、编译、DDP 包装与 epoch 循环

- 第 196-202 行：若读取到 checkpoint，则恢复模型、优化器、scaler、起始 epoch/step。
- 第 205-207 行：可选 `torch.compile` 提升运行效率。
- 第 208-209 行：若分布式可用，则包装为 `DistributedDataParallel`。
- 第 212-222 行：开始 epoch 训练循环。
- 第 213 行：`train_sampler.set_epoch(epoch)` 保证多卡 shuffle 一致。
- 第 214 行：重新设置种子并生成随机索引。
- 第 215-216 行：若从中途恢复，则借助 `SkipBatchSampler` 跳过已完成 step。
- 第 217 行：构建 DataLoader。
- 第 218-222 行：根据是否需要跳步，调用 `train_epoch(...)`。

### 第 224-225 行：清理分布式进程

- 若启用了分布式训练，最后销毁 process group。

---

## 4. 关键算法解析 (Key Algorithm Deep-Dive)

### 4.1 DPO 的核心比较对象：`π_θ(y_w|x) - π_θ(y_l|x)` vs `π_ref`

DPO 不像 PPO 那样显式训练 value function，也不像传统 RLHF 那样先训 reward model 再 rollout。
它直接利用成对偏好数据 `(x, y_w, y_l)`：

- `y_w` = preferred / chosen response
- `y_l` = dispreferred / rejected response

当前策略的相对偏好为：
\
\Delta_\theta = \log \pi_\theta(y_w|x) - \log \pi_\theta(y_l|x)
\

参考模型的相对偏好为：
\
\Delta_{ref} = \log \pi_{ref}(y_w|x) - \log \pi_{ref}(y_l|x)
\

DPO 实际优化的是 `Δθ - Δref`，也就是“相对参考模型，当前策略对 chosen 的偏好提升了多少”。

### 4.2 Bradley-Terry preference model

DPO 背后的概率建模来自 **Bradley-Terry preference model**。它假设：

\
P(y_w \succ y_l | x) = \sigma\left(\beta[(\log \pi_\theta(y_w|x)-\log \pi_\theta(y_l|x))-(\log \pi_{ref}(y_w|x)-\log \pi_{ref}(y_l|x))]\right)
\

- `σ` 是 sigmoid。
- `β` 控制偏好信号强度 (temperature / sharpness)。
- 当策略模型比参考模型更偏向 chosen 时，这个概率会上升。

### 4.3 `beta` 的作用

- `beta` 大：更激进地放大 preference margin，训练更强，但也更容易偏离参考模型。
- `beta` 小：更保守，更像在参考模型附近做微调。

### 4.4 Log probability 计算为什么要用 token-level 再 sum

语言模型天然输出的是逐 token 条件分布：
\
\log \pi(y|x) = \sum_t \log \pi(y_t | x, y_{<t})
\

所以脚本先用 `logits_to_log_probs` 取出每个 token 的 log-prob，再用 `mask` 聚合成序列级别的对数似然。这是 DPO 序列比较的基础。

### 4.5 本文件与 Reward Shaping 的关系

`train_dpo.py` **不做 reward shaping**，也不依赖 `rollout_engine`。

- DPO 使用的是离线偏好样本，而不是在线生成样本。
- 长度奖励、思维链结构奖励、重复惩罚等 shaping 逻辑，出现在 `train_ppo.py` / `train_grpo.py` 中，而不在本文件中。

---

## 5. 调用关系 (Call Graph)

### 5.1 本文件内部调用链

```text
main
├─ init_distributed_mode / setup_seed / lm_checkpoint
├─ init_model -> policy model
├─ init_model -> reference model
├─ DPODataset + DataLoader
└─ train_epoch
   ├─ ref_model(x) -> ref_logits
   ├─ model(x) -> policy logits
   ├─ logits_to_log_probs(ref_logits, y)
   ├─ logits_to_log_probs(policy_logits, y)
   └─ dpo_loss(ref_log_probs, policy_log_probs, mask, beta)
```

### 5.2 与 `rollout_engine`, `train_grpo`, `train_ppo` 的关系

```text
rollout_engine.py
├─ 被 train_ppo.py 调用（在线生成 response）
└─ 被 train_grpo.py 调用（一次 prompt 生成多个 response）

train_dpo.py
└─ 不调用 rollout_engine.py
   因为 DPO 直接使用离线 chosen/rejected 数据
```

### 5.3 训练范式对比

- `train_dpo.py`：**offline preference optimization**，无 rollout、无 critic、无 reward model。
- `train_ppo.py`：**online actor-critic RLHF**，依赖 rollout engine、reward model、critic。
- `train_grpo.py`：**online group-relative RLHF**，依赖 rollout engine、reward model，但不训练 critic。
