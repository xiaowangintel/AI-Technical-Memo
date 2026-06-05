# train_distillation.py 代码分析

## 1. 文件概述 (File Overview)
- `minimind/trainer/train_distillation.py` 是 MiniMind 的 **knowledge distillation** 训练脚本。
- 它的目标是让较小的 student model 学习 teacher model 的输出分布，而不只学习 ground-truth token。
- 在训练流水线（training pipeline）中，它通常位于：
  - `pretrain / SFT` 权重之后；
  - `agent RL / DPO / PPO` 之前。
- 脚本支持 **asymmetric configuration**：teacher 与 student 可以有不同的 `hidden_size`、`num_hidden_layers`、`use_moe`。

## 2. 依赖说明 (Dependencies)
- `os`, `sys`：处理路径与包导入。
- `argparse`, `time`, `warnings`：CLI 参数、计时、warning 控制。
- `torch`, `torch.nn.functional as F`：模型训练与 loss 计算。
- `torch.distributed as dist`：分布式训练。
- `nullcontext`：CPU 场景下替代 autocast。
- `optim`、`DistributedDataParallel`、`DataLoader`、`DistributedSampler`：优化、并行训练与数据加载。
- `MiniMindConfig`：创建 teacher/student 的模型配置。
- `SFTDataset`：把监督微调数据组织成 `(input_ids, labels)`。
- `trainer_utils` 中的：
  - `get_lr`：手写 learning rate schedule；
  - `Logger`：日志；
  - `is_main_process`：主进程判断；
  - `lm_checkpoint`：保存/恢复；
  - `init_distributed_mode`, `setup_seed`：环境初始化；
  - `init_model`：加载 tokenizer 和模型；
  - `SkipBatchSampler`：恢复训练时跳过 batch。

## 3. 逐行代码分析 (Line-by-Line Analysis)

### 1-20 行：基础导入与项目内模块接线
- 1-5 行和其他 trainer 脚本一样，先修正 `sys.path`，保证直接运行时也能导入上级包。
- 7-20 行导入 distillation 所需模块：PyTorch、DDP、优化器、SFTDataset、trainer_utils。
- 17-19 行最关键，因为它们把模型定义、数据集、训练工具真正接到一起。

### 21-22 行：关闭 warning
- 21 行用 `warnings.filterwarnings('ignore')` 清理控制台输出。
- 这对长时间训练尤其常见，避免日志被 warning 淹没。

### 24-35 行：`distillation_loss` 函数
- 24 行定义蒸馏损失 `distillation_loss(student_logits, teacher_logits, temperature=1.0, reduction='batchmean')`。
- 25-26 行：
  - 在 `no_grad` 下把 teacher logits 除以温度 `T` 后做 `softmax`；
  - 再 `detach()`，保证 teacher 不参与梯度。
- 28 行：student logits 也除以温度 `T` 后做 `log_softmax`。
- 30-34 行：计算 `F.kl_div(student_log_probs, teacher_probs)`。
- 35 行：返回 `(temperature ** 2) * kl`。
- 这个 `T²` 是经典 **Hinton distillation** 做法，用于抵消温度缩放对梯度尺度的影响。

### 38-45 行：`train_epoch` 初始化
- 38 行定义一个 epoch 的训练逻辑。
- 39-40 行记录 epoch 起始时间和最后 step。
- 42-44 行如果 teacher 存在，就设为 `eval()` 且 `requires_grad_(False)`。
- 这保证 teacher 只做 forward，不参与 backward，也不会受到 dropout 影响。

### 46-54 行：读 batch、搬运到设备、更新学习率
- 46 行从 loader 中取 `(input_ids, labels)`，并支持断点续训的 `start_step + 1`。
- 48-49 行把数据搬到 `args.device`。
- 50 行构造 `loss_mask = (labels[..., 1:] != -100).float()`，只保留有效 label 位置。
- 51-53 行用 `get_lr(...)` 计算当前步学习率，并写回每个 optimizer param group。
- 这说明本脚本没有用 PyTorch scheduler，而是用自定义 step-wise LR。

### 55-59 行：学生模型 forward
- 56 行在 `autocast_ctx` 里做混合精度 forward。
- 57 行执行 `model(input_ids)`。
- 58 行取 `res.logits[..., :-1, :]` 作为 student logits。
- 因为是自回归语言模型（causal LM），训练时需要预测下一个 token，所以 logits 要去掉最后一个位置。

### 61-66 行：教师模型 forward 与词表对齐
- 61-63 行 teacher 在 `no_grad()` 下前向，避免额外显存和梯度。
- 64 行取 student 的 vocab size。
- 65 行把 `teacher_logits` 裁到相同词表维度：`teacher_logits[..., :vocab_size_student]`。
- 这一步体现了 **asymmetric model config** 的容错性：即使 teacher/student 配置不同，只要 student 词表是 teacher 的前缀范围，就能做 KL。

### 67-80 行：Ground-Truth CE loss
- 68 行注释说明第一部分损失是 ground-truth cross entropy。
- 69 行 `shift_labels = labels[..., 1:]`，与 `student_logits[..., :-1, :]` 对齐。
- 70 行把 `loss_mask` 拉平，方便和 token-level loss 相乘。
- 71-76 行用 `F.cross_entropy(..., reduction='none')` 先算每个 token 的 CE。
- 77 行只对有效位置求平均，得到 `ce_loss_raw`。
- 78-79 行若 student 是 MoE，则把 `res.aux_loss` 加上去；否则 CE 就是最终监督损失。
- 这说明 MoE student 除了语言建模损失，还要承担路由正则项（router auxiliary loss）。

### 81-90 行：Distillation loss
- 82 行如果存在 teacher，就开始计算蒸馏损失。
- 83-87 行调用前面的 `distillation_loss(...)`。
- 84-85 行只选 `loss_mask_flat == 1` 的有效 token，避免把 padding 或非监督位置纳入 KL。
- 88-89 行如果没有 teacher，则 distillation loss 退化为 0。
- 这意味着该脚本也能“退化运行”为单纯 CE 训练，但主要设计目的仍是 distillation。

### 91-92 行：总损失加权融合
- 91 行注释写明：`总损失 = alpha * CE + (1-alpha) * Distill`。
- 92 行同时再除以 `args.accumulation_steps`，便于梯度累积。
- 注意：这里的 `alpha` 权重给的是 CE，而不是 KL；因此 `alpha` 越大，训练越偏向 ground truth，越小越偏向 teacher distribution。

### 94-101 行：反向传播与梯度更新
- 94 行用 `GradScaler` 做缩放 backward。
- 96 行到达梯度累积边界时才真正更新参数。
- 97 行先 `unscale_`，再做梯度裁剪。
- 98 行用 `clip_grad_norm_` 控制梯度爆炸。
- 99-101 行执行 `scaler.step()`、`scaler.update()`、`optimizer.zero_grad()`。
- 这是典型的 mixed precision training 流程。

### 103-121 行：日志与可视化记录
- 103 行在固定间隔或最后一步打印日志。
- 104-110 行统计耗时、loss、CE、MoE auxiliary loss、学习率、ETA。
- 111 行打印一条完整日志，包含 `loss/ce/aux_loss/distill/lr/epoch_time`。
- 113-121 行若启用 wandb/swanlab，则把这些指标同步写入实验平台。
- 这里专门把 `distill_loss` 独立记录，便于观察 teacher signal 是否真的在发挥作用。

### 123-133 行：保存权重与 checkpoint
- 123 行满足保存间隔且是主进程时开始保存。
- 124 行临时切到 `eval()`，保证保存状态稳定。
- 125-126 行拼接输出文件名，如是否带 `_moe` 后缀。
- 127-129 行解包 DDP / compile 包裹模型，拿到真实 state dict。
- 130 行把权重转半精度后保存到 `args.save_dir`。
- 131 行再调用 `lm_checkpoint(...)` 保存 resume 所需状态：模型、优化器、scaler、epoch、step、wandb id 等。
- 132-133 行恢复训练模式并释放 `state_dict`。

### 135 行：手动释放局部变量
- 显式删除 batch 相关 tensor，有助于长训练时减轻显存压力。

### 137-142 行：处理梯度累积尾批次
- 如果 epoch 最后剩余的 step 没达到累积边界，就补做一次 `step()`。
- 这是为了避免最后几步梯度被白白丢掉。

### 145-176 行：CLI 参数
- 146 行注释说明一个典型场景：用 MoE teacher 蒸馏 dense student，也可以用更大 hidden size 的 teacher 蒸馏更小 student。
- 147 行创建参数解析器。
- 148-161 行定义保存路径、epoch、batch size、learning rate、dtype、max sequence length、数据路径等基础参数。
- 162-169 行定义 student/teacher 的结构参数与起始权重来源。
- 170 行 `from_resume` 控制是否自动恢复。
- 171-172 行定义蒸馏最重要的两个超参：`alpha` 与 `temperature`。
- 173-176 行定义 wandb 和 `torch.compile` 开关。

### 178-181 行：环境初始化
- 179 行 `init_distributed_mode()` 初始化 DDP。
- 180 行若已初始化分布式，则把设备设为当前 local rank。
- 181 行根据 rank 设置不同随机种子，兼顾 reproducibility 与多卡差异。

### 183-187 行：目录、模型配置、checkpoint 探测
- 184 行确保保存目录存在。
- 185 行构建 student config。
- 186 行构建 teacher config。
- 187 行若设置 `from_resume==1`，则尝试通过 `lm_checkpoint(...)` 读取已有 resume 数据。
- 这一步只负责“探测/加载状态”，还没真正把状态写回模型。

### 189-193 行：mixed precision 上下文
- 190 行判断当前设备类型。
- 191 行根据 `args.dtype` 选择 `bfloat16` 或 `float16`。
- 192 行如果是 CPU 用 `nullcontext()`，否则用 `torch.cuda.amp.autocast(dtype=dtype)`。
- 这与前面 `train_epoch` 的 `with autocast_ctx:` 配套。

### 194-202 行：wandb/swanlab 初始化
- 195 行默认 `wandb = None`。
- 196-201 行若启用日志平台，就导入 `swanlab as wandb`。
- 198-199 行如果 checkpoint 内有 `wandb_id`，则恢复原 run。
- 200-201 行构造 run name 并执行 `wandb.init(...)`。

### 203-214 行：初始化 teacher/student、数据集、优化器
- 204 行通过 `init_model` 加载 student model 和 tokenizer。
- 205 行打印 student 参数量。
- 206-209 行加载 teacher，设为 eval 并冻结参数，同时打印参数量。
- 210 行创建 `SFTDataset`，说明 distillation 仍然建立在监督数据上，而不是无标签语料上。
- 211 行必要时使用 `DistributedSampler`。
- 212 行构建 `GradScaler`，只有 `float16` 时启用。
- 213 行使用 `AdamW` 优化 student 参数。

### 215-223 行：从 checkpoint 恢复训练状态
- 216 行初始化 `start_epoch, start_step`。
- 217-222 行若存在 checkpoint：
  - 恢复 student 权重；
  - 恢复 optimizer/scaler；
  - 取回 epoch 和 step。
- 这里不会恢复 teacher，因为 teacher 是固定的参考模型，通常重新加载即可。

### 224-230 行：编译与 DDP 包装
- 225-227 行若开启 `torch.compile`，对 student model 编译。
- 228-229 行若已初始化分布式，就用 `DistributedDataParallel` 包装 student。
- teacher 不进入 DDP，是合理的，因为 teacher 不参与更新。

### 231-243 行：训练主循环
- 232 行开始 epoch 循环。
- 233 行让 `DistributedSampler` 每轮重新设定 epoch。
- 234 行重新设定随机种子并生成新的随机索引。
- 235 行如果是恢复训练，则计算本轮需要跳过的 step。
- 236-237 行构建带 skip 能力的 batch sampler 和 DataLoader。
- 238-242 行根据是否断点恢复，调用 `train_epoch(...)`。
- `len(loader) + skip` 的写法保证日志中的总 step 数仍然与原训练过程一致。

### 244-245 行：分布式清理
- 若初始化了 DDP，最后销毁 process group。
- 这是标准收尾操作，避免僵尸通信上下文。

## 4. 关键算法解析 (Key Algorithm Deep-Dive)

### 4.1 Distillation loss：`KL(teacher || student) * T²`
- teacher logits 和 student logits 都先除以温度 `T`。
- teacher 经 `softmax` 变成软标签（soft targets），student 经 `log_softmax` 变成对数概率。
- 再计算：
  \[
  KL\big(p_{teacher}^{(T)} \;||\; p_{student}^{(T)}\big)
  \]
- 代码里通过 `F.kl_div(student_log_probs, teacher_probs)` 实现。
- 最后乘 `T^2`：
  \[
  L_{distill} = T^2 \cdot KL\big(p_t^{(T)} || p_s^{(T)}\big)
  \]
- 这样 teacher 的“暗知识（dark knowledge）”——尤其是非 top-1 token 的相对概率——能更稳定地传给 student。

### 4.2 Alpha blending：CE 与 KL 的加权融合
- 代码中的总损失是：
  \[
  L = \alpha \cdot L_{CE} + (1-\alpha) \cdot L_{KL}
  \]
- 含义：
  - `alpha` 大：更信 ground-truth labels；
  - `alpha` 小：更信 teacher distribution。
- 在实践中：
  - 数据质量高时，可适当提高 `alpha`；
  - teacher 很强而 student 很小时，可适当降低 `alpha`，让蒸馏信号更强。

### 4.3 Teacher-student forward 的非对称配置 (asymmetric config)
- teacher/student 可以不同宽度、深度、MoE 开关。
- 代码层面主要通过两个独立的 `MiniMindConfig` 初始化。
- 只要最终 logits 的 vocab 维可以对齐，就能做 distillation。
- 65 行的 `teacher_logits[..., :vocab_size_student]` 是一个务实的工程写法，用于处理输出维度对齐。

### 4.4 为什么还要保留 CE loss
- 纯 KL distillation 会让 student 过度模仿 teacher 的分布，但不一定严格贴合数据标签。
- 加入 CE 后，student 同时受到：
  - 数据监督（supervised signal）；
  - teacher 软分布监督（teacher signal）。
- 这通常比单独使用任一项更稳。

## 5. 调用关系 (Call Graph)
- `train_distillation.py::__main__`
  -> `init_distributed_mode()` / `setup_seed()`
  -> `MiniMindConfig(student)` + `MiniMindConfig(teacher)`
  -> `init_model(student)` / `init_model(teacher)`
- 数据流：
  - `SFTDataset(args.data_path, tokenizer, ...)`
  -> `DataLoader(...)`
  -> `train_epoch(...)`
- 模型流：
  - `train_epoch()`
  -> `model(input_ids)` 得到 `student_logits`
  -> `teacher_model(input_ids)` 得到 `teacher_logits`
  -> `distillation_loss(...)` + `F.cross_entropy(...)`
  -> `optimizer.step()`
- 保存流：
  - `train_epoch()` -> `lm_checkpoint(...)`
  - 同时向 `args.save_dir/*.pth` 写轻量权重文件。
- 与项目其余脚本的关系：
  - `init_model()` 默认从 `../model` 目录加载 tokenizer，因此它和 `train_tokenizer.py` 生成的 tokenizer 配置是可衔接的；
  - 产出的 student 权重可以继续被 `train_agent.py`、`train_dpo.py`、`train_ppo.py` 等脚本通过 `--from_weight` 作为初始化模型继续训练。
