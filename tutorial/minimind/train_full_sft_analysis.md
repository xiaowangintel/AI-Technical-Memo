# `train_full_sft.py` 代码分析

## 1. 文件概述 (File Overview)
- 这是 MiniMind 的全参数监督微调 (full-parameter supervised fine-tuning, Full SFT) 脚本。
- 它通常接在 `pretrain` 之后，用标注好的多轮对话数据，把基础语言模型继续对齐到 instruction-following / chat 场景。
- 整体流程与 `train_pretrain.py` 高度相似，但数据集换成 `SFTDataset`，默认起始权重 `from_weight='pretrain'`，保存前缀默认 `full_sft`。
- 该脚本依然是全参数训练：所有参数都进入 AdamW，而不是只训练 adapter。

## 2. 依赖说明 (Dependencies)

| 依赖 | 作用 |
| --- | --- |
| `os`, `sys` | 修正相对导入问题。 |
| `argparse` | 定义/解析命令行参数。 |
| `time` | 计算日志中的训练耗时。 |
| `warnings` | 屏蔽非关键 warning。 |
| `torch` | Tensor、AMP、保存权重等训练基础设施。 |
| `torch.distributed as dist` | DDP 分布式训练。 |
| `nullcontext` | CPU 模式下替代 autocast。 |
| `optim`, `nn` | `AdamW` 优化器等。 |
| `DistributedDataParallel` | 多卡并行训练包装器。 |
| `DataLoader`, `DistributedSampler` | 数据 batch 组织与 rank 间数据切分。 |
| `MiniMindConfig` | 描述模型结构超参数。 |
| `SFTDataset` | 监督微调数据集，会只对 assistant token 生成有效 labels。 |
| `trainer_utils.*` | 学习率调度、日志、resume checkpoint、随机种子、模型初始化、跳批采样。 |

## 3. 逐行代码分析 (Line-by-Line Analysis)

> 按连续代码块解释，**覆盖整份源码所有行号范围**。

| 行号 | 代码/逻辑 | 分析 |
| --- | --- | --- |
| 1-5 | 路径与包设置 | 保证脚本可直接通过 `python train_full_sft.py` 运行，并能找到 `model/`、`dataset/`、`trainer/` 模块。 |
| 7-18 | 训练相关导入 | 与预训练脚本几乎一致，但第 17 行换成了 `SFTDataset`。 |
| 20 | 关闭 warning | 保持终端输出简洁。 |
| 23-25 | `train_epoch` 函数定义 | 单个 epoch 的训练入口；`start_step` 支持 resume；`wandb` 变量实际使用 SwanLab 接口风格。 |
| 26 | `enumerate(..., start=start_step + 1)` | 恢复训练时继续沿用历史 step 编号，保证日志、LR、保存节奏连续。 |
| 27-29 | 数据上卡 | `input_ids` 和 `labels` 都搬到训练设备。 |
| 30-32 | 动态学习率写回 optimizer | 每个 step 重新计算 cosine learning rate，并同步到所有 param groups。 |
| 34-38 | autocast 下前向传播与损失 | `model(input_ids, labels=labels)` 返回 `loss` 与 `aux_loss`。对于普通 SFT，核心损失仍是 next-token cross-entropy；若启用 MoE，则会叠加 router auxiliary loss。除以 `accumulation_steps` 是为了正确做 gradient accumulation。 |
| 39 | `scaler.scale(loss).backward()` | 在 AMP 模式下执行反向传播 (backward pass)。 |
| 41-48 | 梯度累积、反缩放、裁剪、更新 | 达到累积步数后，先 `unscale_`，再做 `clip_grad_norm_`，再 `step/update/zero_grad`。这是 training loop 的参数更新核心。 |
| 50-58 | 日志与可视化打点 | 计算 `current_loss`、`current_aux_loss`、`current_logits_loss`、学习率与剩余时间。由于前面 loss 被除过，这里用 `* accumulation_steps` 恢复可解释的原始损失量级。 |
| 60-71 | 保存模型与 resume checkpoint | 只由主进程执行。先保存一个可直接加载的半精度模型权重，再调用 `lm_checkpoint(...)` 保存 resume 所需的 optimizer/scaler/epoch/step 等状态。`_orig_mod` 兼容 `torch.compile`。 |
| 73 | 删除临时张量 | 主动释放引用。 |
| 75-80 | 处理 epoch 末尾未满累积步数的情况 | 这一段很重要：若最后剩余 batch 数量不是 `accumulation_steps` 的整数倍，也会把已经积累的梯度更新出去，避免“最后几步白算”。 |
| 83-84 | 创建参数解析器 | 描述字符串明确写的是 `MiniMind Full SFT`。 |
| 85 | `--save_dir` | 保存目录。 |
| 86 | `--save_weight` | 保存前缀，默认 `full_sft`。 |
| 87 | `--epochs` | SFT 训练轮数。 |
| 88 | `--batch_size` | 每步 micro-batch 大小。 |
| 89 | `--learning_rate` | 初始学习率；SFT 默认 `1e-5`，明显低于预训练。 |
| 90 | `--device` | 训练设备。 |
| 91 | `--dtype` | mixed precision 类型。 |
| 92 | `--num_workers` | DataLoader worker 数。 |
| 93 | `--accumulation_steps` | 梯度累积步数，默认 1，意味着默认每个 batch 都更新。 |
| 94 | `--grad_clip` | 梯度裁剪阈值。 |
| 95 | `--log_interval` | 日志间隔。 |
| 96 | `--save_interval` | 模型保存间隔。 |
| 97 | `--hidden_size` | 隐层维度。 |
| 98 | `--num_hidden_layers` | Transformer 层数。 |
| 99 | `--max_seq_len` | 最大序列长度，SFT 默认 768。 |
| 100 | `--use_moe` | 是否启用 MoE。 |
| 101 | `--data_path` | SFT 数据路径，默认 `sft_t2t_mini.jsonl`。 |
| 102 | `--from_weight` | 载入哪个已有权重继续训练，默认 `pretrain`。这体现了典型 pipeline：`pretrain -> full_sft`。 |
| 103 | `--from_resume` | 是否自动从 checkpoint 续训。 |
| 104 | `--use_wandb` | 是否启用实验追踪。 |
| 105 | `--wandb_project` | 实验项目名。 |
| 106 | `--use_compile` | 是否启用 `torch.compile`。 |
| 107 | `parse_args()` | 完成参数读取。 |
| 109-112 | 初始化 DDP 与 seed | 如果在 DDP 环境下，把设备切换到当前 `local_rank`，并根据 `rank` 偏移随机种子。 |
| 114-117 | 创建保存目录、构造配置、尝试 resume | `MiniMindConfig` 根据 CLI 参数生成；resume 检查使用 `save_weight=full_sft` 对应的 checkpoint。 |
| 119-122 | mixed precision 配置 | CUDA 下启用 `torch.cuda.amp.autocast(dtype=dtype)`；CPU 下不启用 autocast。 |
| 124-131 | SwanLab/W&B 初始化 | 仅主进程初始化；如有 `wandb_id` 则复用原实验记录，实现日志级别的断点续训。 |
| 133-138 | 模型、数据集、采样器、scaler、optimizer 初始化 | `init_model(lm_config, args.from_weight, ...)` 会从 `../out/pretrain_*.pth` 之类的权重开始。`SFTDataset` 会把对话模板化 (chat template)，并只在 assistant 段上生成监督 labels。AdamW 仍然绑定 `model.parameters()`，因此这是 full SFT。 |
| 140-147 | 恢复模型/优化器/AMP 状态 | 若 resume checkpoint 存在，就把模型参数、optimizer 动量、GradScaler、epoch、step 一并恢复。 |
| 149-154 | compile 与 DDP 包装 | 与预训练脚本一致：先 compile，后 DDP。 |
| 156-167 | 训练主循环 | 遍历 epoch，必要时调用 `DistributedSampler.set_epoch(epoch)`；重新设定随机种子；在 resume 场景下根据 `start_step` 计算 `skip`。 |
| 158-161 | 生成索引与 `SkipBatchSampler` | 非 DDP 时使用 `torch.randperm` 生成随机顺序；resume 时通过 `SkipBatchSampler` 跳过已训练批次。 |
| 162 | 创建 DataLoader | 使用 `batch_sampler` 而非 `batch_size`，让“跳过前 N 个 batch”成为 DataLoader 级行为。 |
| 163-167 | 分两种模式进入 `train_epoch` | resume 模式会打印提示，并把 `iters` 修正为“原始 epoch 总 step 数”；非 resume 模式则直接按 `len(loader)` 训练。 |
| 169-170 | 销毁 DDP 进程组 | 训练收尾。 |

## 4. 关键算法解析 (Key Algorithm Deep-Dive)

### 4.1 余弦学习率 (Cosine LR)
`get_lr()` 使用：

\[
lr_t = lr_0 \times (0.1 + 0.45 \times (1 + \cos(\pi t / T)))
\]

这里学习率不会降到 0，而是最低保留到 `0.1 * lr_0`。对 SFT 来说，这样能避免后期学习率过低导致训练几乎停滞。

### 4.2 梯度累积与有效 batch (Effective Batch)
当 `accumulation_steps = k` 时，每个 micro-batch 的 loss 会先除以 `k`，累计 `k` 次 backward 后再执行一次 optimizer step。有效 batch 大小约为：

\[
batch\_size \times accumulation\_steps \times world\_size
\]

这让显存不足时仍能模拟更大 batch。

### 4.3 SFT 监督信号来自哪里？
本脚本的 `forward + loss` 逻辑本身和 pretrain 很像，但监督来源不同：
- `SFTDataset` 会基于 chat template 构造完整对话
- `generate_labels()` 只把 assistant 回复区间对应的 token 设为有效 label
- 其余位置用 `-100` 屏蔽

所以虽然模型仍做 next-token prediction，真正被优化的是“assistant 输出段”。

### 4.4 Mixed Precision / Gradient Clipping / DDP
- `autocast`：降低算子计算精度，提高吞吐。
- `GradScaler`：为 FP16 提供稳定的 loss scaling。
- `clip_grad_norm_`：限制梯度范数，避免偶发梯度爆炸 (gradient explosion)。
- `DistributedSampler + DDP`：保证多卡训练时每张卡看到不同数据，并在 backward 后自动同步梯度。

### 4.5 LoRA 策略在本脚本中的位置
本文件**不做** LoRA parameter freezing。`optimizer = AdamW(model.parameters(), ...)` 说明所有参数均可训练，因此它是“Full SFT”而非“PEFT/LoRA SFT”。

## 5. 调用关系 (Call Graph)

```text
train_full_sft.py
  -> init_distributed_mode() / setup_seed()
  -> MiniMindConfig
  -> init_model(from_weight='pretrain')
       -> AutoTokenizer
       -> MiniMindForCausalLM
  -> SFTDataset(data_path, tokenizer)
       -> create_chat_prompt()
       -> generate_labels()
  -> DataLoader + SkipBatchSampler
  -> train_epoch()
       -> model(input_ids, labels=labels)
       -> loss + aux_loss
       -> scaler.backward()
       -> clip_grad_norm_()
       -> optimizer.step()
       -> lm_checkpoint()
       -> SwanLab/W&B log
```

- 与模型的调用链：`train_full_sft.py -> init_model() -> MiniMindForCausalLM.forward()`。
- 与数据集的调用链：`train_full_sft.py -> SFTDataset.__getitem__()`，由数据集负责构造 assistant-only labels。
- 与工具模块的调用链：学习率、日志、checkpoint、resume、batch 跳过逻辑都来自 `trainer_utils.py`。
