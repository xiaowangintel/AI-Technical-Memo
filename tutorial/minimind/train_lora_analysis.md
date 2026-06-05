# `train_lora.py` 代码分析

## 1. 文件概述 (File Overview)
- 这是 MiniMind 的 LoRA 微调 (LoRA fine-tuning / PEFT) 脚本。
- 它与 `train_full_sft.py` 的主要区别不是数据流，而是**参数更新策略**：先对线性层注入 LoRA 模块，再冻结原始 backbone，只训练少量 `lora` 参数。
- 训练过程仍包含 DDP、mixed precision、gradient accumulation、checkpoint 保存/恢复、SwanLab/W&B 记录等完整基础设施。
- 该脚本会同时保存两种东西：1) `save_lora()` 导出的 adapter-only 权重；2) `lm_checkpoint()` 导出的 resume checkpoint。

## 2. 依赖说明 (Dependencies)

| 依赖 | 作用 |
| --- | --- |
| `os`, `sys` | 修复脚本直跑时的导入路径。 |
| `argparse`, `time`, `warnings` | CLI、时间统计、警告控制。 |
| `torch`, `torch.distributed` | 核心训练框架与 DDP。 |
| `nullcontext` | CPU 模式下禁用 autocast 的空上下文。 |
| `optim`, `nn` | AdamW 与神经网络层定义支持。 |
| `DistributedDataParallel` | 多卡训练包装器。 |
| `DataLoader`, `DistributedSampler` | 数据加载与分布式采样。 |
| `MiniMindConfig` | 创建基座模型配置。 |
| `SFTDataset` | LoRA 微调同样使用监督微调数据。 |
| `save_lora`, `apply_lora` | 注入 LoRA 结构、保存 LoRA adapter 权重。 |
| `trainer_utils.*` | 学习率、日志、checkpoint、DDP 初始化、seed、模型初始化、resume 跳批。 |

## 3. 逐行代码分析 (Line-by-Line Analysis)

> 为保证完整性，下面按连续块解释，**覆盖整个文件的全部行号范围**。

| 行号 | 代码/逻辑 | 分析 |
| --- | --- | --- |
| 1-5 | 包路径修正 | 与其他训练脚本一致，为直接运行脚本准备模块搜索路径。 |
| 7-19 | 导入依赖 | 与 Full SFT 基本一致，但新增 `model.model_lora` 的 `save_lora/apply_lora`。 |
| 21 | 关闭 warning | 保持日志整洁。 |
| 24-26 | `train_epoch` 定义 | 多了 `lora_params` 参数，因为梯度裁剪和 optimizer 都只针对 LoRA 参数。 |
| 27-30 | 取 batch、搬到设备、更新 `last_step` | 数据张量进入 GPU/CPU；记录当前 step。 |
| 31-33 | 计算并设置学习率 | 仍采用 cosine LR schedule，并逐个 param group 更新。这里只有 LoRA 参数被 optimizer 管理，但接口写法与全参数训练一致。 |
| 35-38 | autocast 下前向传播与 loss | `model(input_ids, labels=labels)` 仍返回 `res.loss + res.aux_loss`。LoRA 并不改变 loss 形式，只改变可训练参数集合。损失同样要除以 `accumulation_steps`。 |
| 40 | `scaler.scale(loss).backward()` | 反向传播会把梯度传播到整个图，但因为非 LoRA 参数 `requires_grad=False`，真正保留梯度的只有 adapter 参数。 |
| 42-47 | 梯度累积、LoRA 梯度裁剪、参数更新 | 达到累积步数后：1) 反缩放梯度；2) 对 `lora_params` 做 `clip_grad_norm_`；3) 执行 optimizer step；4) 更新 scaler；5) 清梯度。这里明确体现“只更新 LoRA”的策略。 |
| 49-57 | 日志打印与实验记录 | 与 Full SFT 类似，记录 loss、logits loss、aux loss、lr、ETA。 |
| 59-66 | 保存 LoRA 权重与 resume checkpoint | 第 62 行生成 adapter 文件名；第 64 行 `save_lora(model, path)` 只保存 `.lora.*` 权重，不保存整模型。第 65 行 `lm_checkpoint(...)` 额外保存 resume 所需信息，便于中断后继续训练。 |
| 68 | 删除临时变量 | 释放引用。 |
| 70-75 | 处理尾部残余梯度 | 如果 epoch 末尾不足一个完整 accumulation 周期，也会对已有的 LoRA 梯度执行一次更新。 |
| 77-78 | 创建参数解析器 | 描述字符串为 `MiniMind LoRA Fine-tuning`。 |
| 79 | `--save_dir` | 输出目录。 |
| 80 | `--lora_name` | LoRA 权重名，也是 checkpoint 名的核心前缀。 |
| 81 | `--epochs` | 微调轮数，默认 10。 |
| 82 | `--batch_size` | micro-batch 大小。 |
| 83 | `--learning_rate` | LoRA 基础学习率，默认 `1e-4`，通常高于 Full SFT。 |
| 84 | `--device` | 训练设备。 |
| 85 | `--dtype` | mixed precision 类型。 |
| 86 | `--num_workers` | DataLoader worker 数。 |
| 87 | `--accumulation_steps` | 梯度累积步数。 |
| 88 | `--grad_clip` | LoRA 梯度裁剪阈值。 |
| 89 | `--log_interval` | 日志输出间隔。 |
| 90 | `--save_interval` | 保存间隔。 |
| 91 | `--hidden_size` | 模型隐藏维度。 |
| 92 | `--num_hidden_layers` | 模型层数。 |
| 93 | `--max_seq_len` | 最大序列长度。 |
| 94 | `--use_moe` | 是否启用 MoE。 |
| 95 | `--data_path` | LoRA 微调数据路径。 |
| 96 | `--from_weight` | 基座权重来源，默认 `full_sft`。典型 pipeline 是 `pretrain -> full_sft -> lora`。 |
| 97 | `--from_resume` | 是否自动续训。 |
| 98 | `--use_wandb` | 是否启用实验追踪。 |
| 99 | `--wandb_project` | 项目名。 |
| 100 | `--use_compile` | 是否启用 `torch.compile`。 |
| 101 | `parse_args()` | 读取命令行参数。 |
| 103-106 | 初始化 DDP 与 seed | 与其他脚本一致。 |
| 108-111 | 创建目录、模型配置、resume 检查 | 注意 resume 使用的是 `args.lora_name` 对应 checkpoint，而不是 `from_weight`。 |
| 113-116 | mixed precision autocast 设置 | 仍采用 `autocast + GradScaler` 的标准模式。 |
| 118-125 | SwanLab/W&B 初始化 | 使用 `lora_name` 生成更具体的 run name；如存在 `wandb_id` 则恢复到同一实验记录。 |
| 127-129 | 初始化基座模型并注入 LoRA | `init_model(...)` 先加载基础大模型；`apply_lora(model)` 再遍历线性层，为满足条件的 `nn.Linear` 注入 `lora` 子模块，并 monkey-patch 原 `forward`。 |
| 131-136 | 统计总参数与 LoRA 参数占比 | 通过 `named_parameters()` 数 `lora` 参数量，让用户知道 PEFT 的参数效率。 |
| 138-146 | 冻结非 LoRA 参数，收集 `lora_params` | 这是本脚本最关键的部分：如果参数名包含 `'lora'`，则 `requires_grad=True` 并加入 `lora_params`；否则全部冻结。这一步决定了 backward 后真正可更新的只有 adapter。 |
| 147-151 | 初始化数据集、采样器、scaler、optimizer | 数据集仍是 `SFTDataset`。`optimizer = AdamW(lora_params, ...)` 再次明确：只有 LoRA 参数会参与 optimizer state 和参数更新。 |
| 153-160 | 恢复 checkpoint 状态 | `model.load_state_dict(..., strict=False)` 是一个容错设计：LoRA 场景下 state dict 可能存在 base 权重与 adapter 权重的组合，`strict=False` 可以避免键不完全匹配时报错。随后恢复 optimizer/scaler/epoch/step。 |
| 162-165 | 自动关闭 compile | 因为 `apply_lora()` 通过 monkey-patch 改写了线性层 `forward`，与 `torch.compile` 兼容性不好，所以脚本直接关闭 compile 并打印提示。 |
| 166-167 | DDP 包装 | 若启用分布式，则对已经注入 LoRA 的模型做 `DistributedDataParallel`。 |
| 169-180 | epoch 训练主循环 | 逻辑与其他脚本一致：设置 sampler epoch、准备索引、计算 `skip`、构建 `SkipBatchSampler`、创建 DataLoader、进入 `train_epoch`。 |
| 176-180 | resume/非 resume 两条路径 | 若 `skip > 0`，就从中断位置后继续；否则正常从头训练本 epoch。 |
| 182-183 | 销毁进程组 | DDP 训练结束收尾。 |

## 4. 关键算法解析 (Key Algorithm Deep-Dive)

### 4.1 LoRA 参数冻结策略 (LoRA Parameter Freezing Strategy)
LoRA 的核心思想是：

\[
W'x = Wx + BAx
\]

其中：
- `W`：原始大矩阵 (frozen weight)
- `A \in R^{r \times d}`、`B \in R^{d' \times r}`：低秩增量矩阵 (low-rank adapters)
- `r`：rank，远小于原维度

在本脚本里，这个策略落地为两步：
1. `apply_lora(model)` 给线性层添加 `module.lora`
2. 只让名字中包含 `'lora'` 的参数 `requires_grad=True`

因此：
- backbone 权重被冻结，不更新
- optimizer 只维护 `lora_params`
- gradient clipping 也只裁剪 `lora_params`

这就是参数高效微调 (parameter-efficient fine-tuning, PEFT) 的关键。

### 4.2 梯度累积 (Gradient Accumulation)
与其他训练脚本完全一致：
- 每个 micro-batch loss 先除以 `accumulation_steps`
- 多次 backward 后再执行一次 optimizer step

但这里更新的对象不是全参数，而只是 adapter 参数，所以“有效 batch 扩大”与“参数量缩小”同时发生。

### 4.3 Mixed Precision：`autocast + GradScaler`
- `autocast` 负责低精度算子执行
- `GradScaler` 负责在 FP16 场景稳定训练
- 因为 LoRA 训练常用于较大模型，mixed precision 对显存节省尤其有价值

### 4.4 Checkpoint 双保存策略
本脚本保存两类文件：
- **adapter-only**：`save_lora()` 输出，只含 LoRA 权重，便于分发与部署
- **resume checkpoint**：`lm_checkpoint()` 输出，含 optimizer/scaler/epoch/step，便于断点续训

两者用途不同：前者面向“使用结果”，后者面向“继续训练”。

### 4.5 余弦学习率 (Cosine LR)
仍使用：

\[
lr_t = lr_0 \times (0.1 + 0.45 \times (1 + \cos(\pi t / T)))
\]

LoRA 常允许稍大的基础学习率，因为训练参数更少、更新更聚焦。脚本默认 `1e-4` 就体现了这一点。

## 5. 调用关系 (Call Graph)

```text
train_lora.py
  -> init_distributed_mode() / setup_seed()
  -> MiniMindConfig
  -> init_model(from_weight='full_sft')
       -> MiniMindForCausalLM
  -> apply_lora(model)
       -> inject LoRA modules into Linear layers
  -> freeze non-LoRA params
  -> SFTDataset(data_path, tokenizer)
  -> DataLoader + SkipBatchSampler
  -> train_epoch(..., lora_params)
       -> model(input_ids, labels=labels)
       -> backward on LoRA graph
       -> clip_grad_norm_(lora_params)
       -> optimizer.step()  # only LoRA params
       -> save_lora()
       -> lm_checkpoint()
       -> SwanLab/W&B log
```

- 与模型的关系：先加载 `MiniMindForCausalLM`，再由 `apply_lora()` 对线性层做 adapter 注入。
- 与数据集的关系：依旧使用 `SFTDataset`，因为 LoRA 微调常针对 instruction/chat 数据。
- 与工具模块的关系：学习率、resume、日志、DDP 初始化、skip sampler 都复用 `trainer_utils.py`。
