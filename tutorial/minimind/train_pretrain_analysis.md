# `train_pretrain.py` 代码分析

## 1. 文件概述 (File Overview)
- 这是 MiniMind 的预训练 (pretraining) 入口脚本，用于做标准的 Causal Language Modeling。
- 它负责串起完整训练流水线 (training pipeline)：参数解析、DDP 初始化、模型构建、数据集加载、mixed precision、optimizer/GradScaler、训练循环、日志记录、checkpoint 保存与断点续训。
- 该脚本是“全参数训练 (full-parameter training)”版本：`optimizer = AdamW(model.parameters(), ...)`，不会像 LoRA 那样冻结主干参数。
- 数据侧使用 `PretrainDataset`，输入通常是纯文本 `text` 字段；标签 `labels` 基本等于 `input_ids`，仅把 pad 位置设成 `-100`。

## 2. 依赖说明 (Dependencies)

| 依赖 | 作用 |
| --- | --- |
| `os`, `sys` | 调整模块搜索路径 (module search path)，确保脚本可直接运行。 |
| `argparse` | 解析命令行参数 (CLI arguments)。 |
| `time` | 统计 epoch 内耗时，估算 ETA。 |
| `warnings` | 关闭非关键 warning，减少训练日志噪声。 |
| `torch` | 张量计算、autocast、GradScaler、保存权重等核心能力。 |
| `torch.distributed as dist` | 初始化分布式训练 (DDP distributed training) 所需的进程组。 |
| `nullcontext` | CPU 场景下替代 autocast 的空上下文 (no-op context)。 |
| `optim`, `nn` | `optim.AdamW` 用于优化；`nn` 在本文件未直接使用，但与 PyTorch 训练语义相关。 |
| `DistributedDataParallel` | 把模型包装成 DDP 模式。 |
| `DataLoader`, `DistributedSampler` | 负责 batch 组装和分布式采样。 |
| `MiniMindConfig` | 定义 MiniMind 模型结构参数，如 `hidden_size`、`num_hidden_layers`、`use_moe`。 |
| `PretrainDataset` | 预训练数据集，输出 `(input_ids, labels)`。 |
| `get_lr` | 余弦学习率调度 (cosine LR schedule)。 |
| `Logger` | 只在主进程 (main process) 打印日志。 |
| `is_main_process` | 判断当前 rank 是否主进程。 |
| `lm_checkpoint` | 保存/加载断点续训 checkpoint。 |
| `init_distributed_mode` | 根据环境变量初始化 DDP。 |
| `setup_seed` | 固定随机种子，增强可复现性 (reproducibility)。 |
| `init_model` | 创建 tokenizer 与 `MiniMindForCausalLM`，并按需加载已有权重。 |
| `SkipBatchSampler` | 断点续训时跳过已训练 batch，恢复到正确 step。 |

## 3. 逐行代码分析 (Line-by-Line Analysis)

> 为了可读性，下面按连续语义块 (contiguous blocks) 解释，但**覆盖整份源码的全部行号范围**。

| 行号 | 代码/逻辑 | 分析 |
| --- | --- | --- |
| 1-5 | `os/sys` 导入、设置 `__package__`、追加 `sys.path` | 这几行解决“脚本直接运行时的相对导入”问题。训练脚本位于 `minimind/trainer/`，通过把上级目录加入搜索路径，让 `model.*`、`dataset.*`、`trainer.*` 可以被正确导入。 |
| 7-18 | 导入训练所需模块 | 包括参数解析、时间统计、PyTorch、DDP、数据加载、模型配置与训练工具函数。这里已经把训练主链路依赖全部拉齐。 |
| 20 | `warnings.filterwarnings('ignore')` | 关闭 warning 输出，避免训练日志被 tokenizers / amp / transformers 的提示刷屏。 |
| 23-25 | `train_epoch` 函数头 | 定义单个 epoch 的训练过程；`start_step` 用于断点续训；`wandb` 变量实际绑定的是 `swanlab as wandb`。 |
| 26 | `enumerate(loader, start=start_step + 1)` | 恢复训练时从 `start_step + 1` 开始编号，而不是从 0 开始，保证日志、学习率步数、保存步数与真实训练历史一致。 |
| 27-29 | `input_ids/labels` 搬到设备、更新 `last_step` | 数据被送到 `args.device`；`last_step` 记录最后一个实际执行的 step，便于 epoch 结束时处理“未整除 accumulation”的残余梯度。 |
| 30-32 | `get_lr(...)` 并写回 `optimizer.param_groups` | 这里按**每个 micro-step**更新学习率，而非每个 epoch。`epoch * iters + step` 表示当前全局步数 (global step)，`args.epochs * iters` 表示总步数。调度公式在 `trainer_utils.get_lr` 中实现。 |
| 34-38 | `with autocast_ctx:` 前向传播 (forward) 与 loss 计算 | `model(input_ids, labels=labels)` 会进入 `MiniMindForCausalLM.forward()`，返回 `res.loss` 和 `res.aux_loss`。普通 dense 模型时 `aux_loss` 基本为 0；若启用 MoE，则它是 router auxiliary loss。两者相加后再除以 `accumulation_steps`，确保梯度累积后的总梯度与“大 batch 直接训练”近似等价。 |
| 39 | `scaler.scale(loss).backward()` | 这是混合精度训练 (mixed precision training) 的标准写法。若 `dtype=float16`，`GradScaler` 会放大 loss，避免 FP16 梯度下溢；若 `dtype=bfloat16`，`GradScaler` 实际会被禁用。 |
| 41-48 | 梯度累积 + 梯度裁剪 + 参数更新 | 当 `step % accumulation_steps == 0` 时，说明已经累计了足够多的 micro-batches：1) `unscale_` 把梯度还原回真实尺度；2) `clip_grad_norm_` 做 gradient clipping；3) `scaler.step(optimizer)` 执行 optimizer step；4) `scaler.update()` 更新缩放因子；5) `zero_grad(set_to_none=True)` 清空梯度。这里正是完整的 `backward -> optimizer step` 核心路径。 |
| 50-58 | 日志打印与 SwanLab/W&B 记录 | 每到 `log_interval` 或 epoch 最后一步，就计算当前 loss、`aux_loss`、`logits_loss`、学习率与 ETA。`current_loss` 要乘回 `accumulation_steps`，因为前面为了梯度累积做了除法。`current_logits_loss = total_loss - aux_loss`，可帮助区分主任务损失与 MoE 辅助损失。 |
| 60-71 | 保存权重与断点续训 checkpoint | 仅主进程保存：先 `model.eval()`，再构造输出文件名。`raw_model = model.module if DDP else model` 用于“脱壳” (unwrap)；`getattr(..., '_orig_mod', ...)` 用于兼容 `torch.compile` 包裹。第 67 行把 state_dict 转成 `half().cpu()` 后落盘，减小体积。第 68-69 行额外调用 `lm_checkpoint(...)` 保存 resume 信息：模型、optimizer、scaler、epoch、step、wandb_id。 |
| 73 | 显式 `del` 临时变量 | 及时释放显存/内存引用，尤其对大 batch 或长序列训练更友好。 |
| 75-80 | 处理 epoch 尾部残余梯度 | 如果本 epoch 最后若干 batch 没有凑满 `accumulation_steps`，这里会额外执行一次 `unscale -> clip -> step -> update -> zero_grad`，避免最后一段梯度被丢弃。 |
| 83-84 | 创建 `ArgumentParser` | 描述字符串是 `MiniMind Pretraining`，说明脚本定位就是预训练入口。 |
| 85 | `--save_dir` | 最终模型权重输出目录。 |
| 86 | `--save_weight` | 保存权重前缀，默认 `pretrain`。最终文件名会再拼接 `hidden_size` 和可选 `_moe`。 |
| 87 | `--epochs` | 训练轮数 (number of epochs)。 |
| 88 | `--batch_size` | 单卡/单进程 DataLoader 的 batch size。真正的有效 batch size 还要乘 `accumulation_steps` 和 `world_size`。 |
| 89 | `--learning_rate` | 基础学习率 (base LR)，会被余弦调度动态缩放。 |
| 90 | `--device` | 默认优先 CUDA，否则 CPU。DDP 初始化后会被覆盖成 `cuda:{local_rank}`。 |
| 91 | `--dtype` | 混合精度类型，可选思路是 `bfloat16` 或 `float16`。 |
| 92 | `--num_workers` | DataLoader 子进程数。 |
| 93 | `--accumulation_steps` | 梯度累积步数 (gradient accumulation steps)。默认 8，说明预训练场景倾向于用更大 effective batch。 |
| 94 | `--grad_clip` | 梯度裁剪阈值。 |
| 95 | `--log_interval` | 日志打印间隔。 |
| 96 | `--save_interval` | checkpoint 保存间隔。 |
| 97 | `--hidden_size` | 模型隐藏维度。 |
| 98 | `--num_hidden_layers` | Transformer block 层数。 |
| 99 | `--max_seq_len` | 截断长度；预训练默认 340。 |
| 100 | `--use_moe` | 是否启用 Mixture-of-Experts。 |
| 101 | `--data_path` | 预训练数据 JSONL 路径。 |
| 102 | `--from_weight` | 初始权重来源；默认 `none`，即从头训练 (from scratch)。 |
| 103 | `--from_resume` | 是否尝试自动恢复最近的 resume checkpoint。 |
| 104 | `--use_wandb` | 是否启用实验跟踪；实际导入的是 SwanLab。 |
| 105 | `--wandb_project` | 实验项目名。 |
| 106 | `--use_compile` | 是否启用 `torch.compile`。 |
| 107 | `args = parser.parse_args()` | 正式读取 CLI 参数，后续全部训练行为都由 `args` 决定。 |
| 108-112 | 初始化 DDP 与随机种子 | `init_distributed_mode()` 检查 `RANK/LOCAL_RANK` 环境变量。若在 DDP 中，则把设备改成当前卡。种子使用 `42 + rank`，避免不同 rank 的某些随机流完全重合。 |
| 114-117 | 输出目录、模型配置、resume 检查 | 创建保存目录；构造 `MiniMindConfig`；若 `from_resume==1`，则去 `../checkpoints` 查找 resume checkpoint。注意这里恢复的是 `save_weight` 对应训练任务，而不是 `from_weight`。 |
| 119-122 | mixed precision 上下文设置 | CPU 场景使用 `nullcontext()`；CUDA 场景使用 `torch.cuda.amp.autocast(dtype=dtype)`。这是 `autocast + GradScaler` 方案中的 `autocast` 部分。 |
| 124-131 | 初始化 SwanLab/W&B | 只有主进程启用日志系统，避免多卡重复打点。若 checkpoint 中保存过 `wandb_id`，则以 `resume='must'` 方式续跑同一个 experiment。 |
| 133-138 | 初始化模型、数据集、采样器、GradScaler、优化器 | `init_model(...)` 会创建 tokenizer 和 `MiniMindForCausalLM`，并按 `from_weight` 加载已有权重。`PretrainDataset` 返回预训练样本。若 DDP 开启，则使用 `DistributedSampler`。`GradScaler(enabled=(args.dtype == 'float16'))` 表示仅 FP16 开启 loss scaling。`optimizer = AdamW(model.parameters(), ...)` 说明这是 full-parameter training。 |
| 140-147 | 从 resume checkpoint 恢复训练状态 | 恢复模型权重、optimizer 动量、GradScaler 状态、起始 epoch 和起始 step。这样不仅能恢复参数，还能恢复优化器历史与 AMP 状态，减少训练分布漂移。 |
| 149-154 | `torch.compile` 与 DDP 包装 | 若启用 compile，先做图编译；之后若分布式已初始化，再包装为 `DistributedDataParallel`。这种顺序能让 DDP 持有 compile 后模型。 |
| 156-166 | epoch 主循环 | 从 `start_epoch` 继续训练。`DistributedSampler.set_epoch(epoch)` 保证每轮 shuffle 不同。`torch.randperm` 生成本地随机索引，供非 DDP 模式使用。`skip` 表示断点续训时本轮要跳过多少个 step。 |
| 160-161 | `SkipBatchSampler(...)` | 这是断点续训的重要拼图：不重新处理已完成的 batch，而是直接跳过前 `skip` 个 batch，和日志编号、学习率步数保持一致。 |
| 162 | 构建 `DataLoader` | 这里把 `batch_sampler` 直接交给 DataLoader，因此 DataLoader 不再需要单独 `batch_size` 或 `sampler` 参数。 |
| 163-166 | 处理 resume 时的提示与 `train_epoch` 调用 | 若 `skip > 0`，日志会提示“跳过前 N 个 step”；并把 `iters` 设成 `len(loader) + skip`，使当前 epoch 的全长仍然对应“原始 epoch 总步数”。否则按正常模式训练。 |
| 168-169 | 进程组清理 | 训练结束后销毁 DDP process group，避免多进程资源泄漏。 |

## 4. 关键算法解析 (Key Algorithm Deep-Dive)

### 4.1 余弦学习率调度 (Cosine LR Schedule)
`get_lr()` 的公式是：

\[
lr_t = lr_0 \times \left(0.1 + 0.45 \times (1 + \cos(\pi \cdot t / T))\right)
\]

- `lr_0`：命令行传入的 `--learning_rate`
- `t`：当前 global step
- `T`：总训练步数 `args.epochs * iters`
- 因为 `cos` 从 `1 -> -1`，所以缩放系数从 `1.0 -> 0.1`
- 这意味着学习率从 base LR 平滑衰减到 `0.1 * base LR`

这种写法的好处是：前期学习率高、后期更稳，适合 language model 的长程训练。

### 4.2 梯度累积 (Gradient Accumulation) 数学
设：
- 单次前向的 micro-batch loss 为 `L_i`
- 累积步数为 `k = accumulation_steps`

脚本实际反传的是：

\[
\frac{L_i}{k}
\]

连续 `k` 次 backward 后，参数梯度近似等于：

\[
\nabla \theta \approx \frac{1}{k}\sum_{i=1}^{k}\nabla L_i
\]

即“等价于”对更大 batch 的平均梯度。有效 batch size (effective batch size) 约为：

\[
\text{batch\_size} \times \text{accumulation\_steps} \times \text{world\_size}
\]

### 4.3 Mixed Precision：`autocast + GradScaler`
- `autocast`：让 matmul / attention / linear 等算子自动用低精度执行，降低显存与提升吞吐。
- `GradScaler`：只在 `float16` 模式启用，防止小梯度 underflow。
- `bfloat16` 一般数值范围更大，通常不需要 loss scaling，所以这里 `enabled=(args.dtype == 'float16')`。

### 4.4 DDP 与断点续训 (Resume)
- `init_distributed_mode()` 读取环境变量完成多卡初始化。
- `DistributedSampler` 让不同 rank 看到不同数据切片。
- `SkipBatchSampler` 让恢复训练时精确跳过已完成 batch。
- `lm_checkpoint()` 会保存 `epoch/step/optimizer/scaler/world_size`；若 GPU 数量变化，`trainer_utils` 会自动换算 step。

### 4.5 LoRA 参数冻结策略在本脚本中的状态
本脚本**没有**使用 LoRA parameter freezing strategy，而是把 `model.parameters()` 全部交给 AdamW，因此属于全参数训练。与 `train_lora.py` 相比，这里所有参数都参与 `forward/backward/optimizer step`。

## 5. 调用关系 (Call Graph)

```text
CLI arguments
  -> MiniMindConfig
  -> init_distributed_mode() / setup_seed()
  -> init_model()
       -> AutoTokenizer.from_pretrained()
       -> MiniMindForCausalLM(lm_config)
       -> (optional) load pretrained/full_sft weights
  -> PretrainDataset(data_path, tokenizer)
       -> __getitem__() returns input_ids, labels
  -> DataLoader + (DistributedSampler or randperm indices)
  -> train_epoch()
       -> model(input_ids, labels=labels)
            -> MiniMindForCausalLM.forward()
            -> MiniMindModel.forward()
            -> returns loss + aux_loss
       -> scaler.scale(loss).backward()
       -> clip_grad_norm_()
       -> optimizer.step()
       -> lm_checkpoint()
       -> SwanLab/W&B log
```

- 与模型的连接：通过 `MiniMindConfig + init_model + MiniMindForCausalLM`。
- 与数据集的连接：通过 `PretrainDataset`，它把原始文本转成 token 序列与语言模型标签。
- 与工具模块的连接：`get_lr`、`Logger`、`lm_checkpoint`、`SkipBatchSampler` 全都来自 `trainer_utils.py`。
