# `trainer_utils.py` 代码分析

## 1. 文件概述 (File Overview)
- 这是 MiniMind 训练脚本共用的工具模块 (training utilities module)。
- 它把“训练外围基础设施”抽离出来：模型参数统计、主进程日志、余弦学习率、DDP 初始化、随机种子、checkpoint 保存/恢复、模型初始化、断点跳批采样器、奖励模型包装器。
- `train_pretrain.py`、`train_full_sft.py`、`train_lora.py` 都直接依赖这里的函数，因此它相当于这些脚本的共享 backbone utility layer。

## 2. 依赖说明 (Dependencies)

| 依赖 | 作用 |
| --- | --- |
| `os`, `sys` | 文件路径处理与脚本导入路径修正。 |
| `random`, `numpy`, `torch` | 随机种子与张量操作。 |
| `math` | 计算 cosine learning rate。 |
| `torch.distributed as dist` | 判断 rank、初始化/读取 world size。 |
| `DistributedDataParallel` | checkpoint 保存时对 DDP 模型“脱壳”。 |
| `Sampler` | 自定义 `SkipBatchSampler` 继承基类。 |
| `AutoTokenizer`, `AutoModel`, `AutoModelForSequenceClassification` | tokenizer/外部 reward model 初始化。这里 `AutoModelForSequenceClassification` 当前未使用，可能是预留导入。 |
| `MiniMindForCausalLM` | 训练脚本真正初始化的语言模型类。 |

## 3. 逐行代码分析 (Line-by-Line Analysis)

> 为了保持可读性，以下按函数/类的连续块解释，**覆盖整份源码全部行号范围**。

| 行号 | 代码/逻辑 | 分析 |
| --- | --- | --- |
| 1-3 | 模块注释 | 说明这是“训练工具函数集合”。 |
| 4-17 | 导入与路径修正 | 与训练脚本同样通过 `sys.path.append(...)` 处理包路径。之后导入随机库、PyTorch、transformers 与 `MiniMindForCausalLM`。 |
| 18-29 | `get_model_params(model, config)` | 用于打印模型参数规模。它会先计算总参数量 `total`，再读取 MoE 相关配置：总 expert 数、每 token 激活 expert 数、共享 expert 数。通过统计 `mlp.experts.0.` 和 `mlp.shared_experts.0.` 的参数规模，推导出“总参数量”与“单 token 实际激活参数量 (active params)”的区别。若 `active < total`，日志会打印 `Model Params: total-Aactive`。 |
| 31-32 | `is_main_process()` | 若分布式未初始化，默认当前就是主进程；否则只有 `rank == 0` 返回真。所有日志/保存都依赖这个判断。 |
| 35-37 | `Logger(content)` | 一个非常薄的 logging wrapper，只在主进程打印，避免多卡重复输出同样内容。 |
| 40-41 | `get_lr(current_step, total_steps, lr)` | 这里实现了 cosine learning rate schedule：`lr * (0.1 + 0.45 * (1 + cos(pi * current_step / total_steps)))`。训练脚本会在每个 step 调用它。 |
| 44-51 | `init_distributed_mode()` | 先检查环境变量 `RANK`。如果不存在，则返回 0，表示非 DDP。若存在，则调用 `dist.init_process_group(backend='nccl')` 初始化分布式通信，再根据 `LOCAL_RANK` 设置当前 CUDA 设备。 |
| 54-61 | `setup_seed(seed)` | 同时固定 Python `random`、NumPy、PyTorch CPU/CUDA 的随机种子，并关闭 `cudnn.benchmark`、开启 `cudnn.deterministic`，尽量提高复现性。 |
| 63-68 | `lm_checkpoint(...)` 开头：路径构造 | 根据 `weight`、`hidden_size`、`use_moe` 生成两个文件路径：1) `ckp_path` 存“可直接加载的权重”；2) `resume_path` 存“断点续训状态”。 |
| 69-76 | 保存模式：模型脱壳与原子写入 | 如果传入 `model`，说明当前是保存。先把 DDP / compile 包裹去掉，取出原始 `state_dict()`；再把所有参数转成 `half().cpu()`。随后先写到 `*.tmp`，再用 `os.replace` 原子替换，降低中途写坏文件的风险。 |
| 77-84 | 保存模式：提取 wandb/swanlab run id | 为了支持日志断点续跑，函数会尝试从 `wandb` 对象中拿到 `run.id`。这里兼容了 `wandb.get_run()` 和 `wandb.id` 两种接口风格。 |
| 85-100 | 保存模式：组装 `resume_data` | 除模型权重外，还保存 optimizer state、当前 epoch、step、world size、wandb_id。对额外 `kwargs`（如 `scaler`）如果对象有 `state_dict()`，也会自动序列化进去。这个设计让 `lm_checkpoint()` 很通用。 |
| 102-106 | 保存模式：原子保存 resume checkpoint 并清理缓存 | resume 数据也用 `*.tmp + os.replace` 保存。最后删除大对象并 `torch.cuda.empty_cache()`。 |
| 107-116 | 加载模式：返回 resume checkpoint，并适配 GPU 数变化 | 如果调用时 `model is None`，说明是加载。若 `resume_path` 存在，则用 CPU 读入 checkpoint。若保存时 `world_size` 与当前 `world_size` 不同，会执行 `step = step * saved_ws // current_ws` 的换算，并打印提示。这样可以在 GPU 数变化时尽量从正确的训练位置继续。 |
| 119-131 | `init_model(...)` | 先加载 tokenizer，再实例化 `MiniMindForCausalLM(lm_config)`。若 `from_weight != 'none'`，就从 `../out/{from_weight}_{hidden_size}{moe_suffix}.pth` 加载已有权重。最后打印模型总参数、可训练参数，并把模型搬到目标设备。这个函数是所有基础训练脚本的模型入口。 |
| 134-139 | `SkipBatchSampler.__init__` | 接收原始 sampler、batch size 和要跳过的 batch 数。这个类专门服务断点续训。 |
| 140-153 | `SkipBatchSampler.__iter__` | 迭代原始样本索引，按 `batch_size` 组 batch。如果当前 batch 还处在“应跳过区间”，则增加 `skipped` 计数并丢弃；超过后再正常 `yield batch`。这样 DataLoader 从外观上就像“天然从中间开始”。 |
| 155-157 | `SkipBatchSampler.__len__` | 返回总 batch 数减去需要跳过的 batch 数，保证 `len(loader)` 与 resume 后的有效长度一致。 |
| 160-166 | `LMForRewardModel.__init__` | 一个奖励模型包装器 (reward model wrapper)。加载 tokenizer 和 `AutoModel`，移动到目标设备并切到 `eval()`。这个类主要服务偏好优化/RL，不是前三个基础训练脚本的直接主角。 |
| 167-177 | `get_score(messages, response)` | 根据历史对话 `messages` 和候选 `response` 构造评估消息，再调用底层模型的 `get_score(...)`。最终把分数裁剪到 `[-3, 3]`。这种 clamp 可以避免异常高分/低分破坏后续奖励尺度。 |

## 4. 关键算法解析 (Key Algorithm Deep-Dive)

### 4.1 余弦学习率公式 (Cosine LR Formula)
这里最关键的公式在第 41 行：

\[
lr_t = lr_0 \times (0.1 + 0.45 \times (1 + \cos(\pi t / T)))
\]

含义：
- 一开始 `t=0`，`cos(0)=1`，所以 `lr_t = lr_0`
- 训练结束附近 `cos(\pi)=-1`，所以 `lr_t = 0.1 \times lr_0`

它是一个“有底座”的 cosine decay，不会降到 0。

### 4.2 断点续训与 world size 变化
`lm_checkpoint()` 的一个实用细节是：如果保存 checkpoint 时用的是 `saved_ws` 张卡，而恢复时用的是 `current_ws` 张卡，就自动调整 step：

\[
step_{new} = step_{old} \times saved\_ws / current\_ws
\]

这是一个近似换算，目的是让“已经消费过的数据量”在 GPU 数变化时保持大致一致。

### 4.3 `SkipBatchSampler` 与梯度累积的关系
这个文件本身不直接做 backward，但它通过 `SkipBatchSampler` 保证恢复训练时**从正确的 batch 边界继续**。这对 gradient accumulation 非常关键：
- 如果 step 恢复错位，累积周期也会错位
- 学习率、日志、保存频率都会一起偏移

因此 `SkipBatchSampler` 是 resume 正确性的关键组件。

### 4.4 LoRA 参数冻结策略在本文件中的位置
`trainer_utils.py` 本身**不直接实现** LoRA freezing，但它为 LoRA 训练提供了通用基础设施：
- `init_model()` 负责加载 base model
- `lm_checkpoint()` 能序列化 LoRA 训练时的 optimizer/scaler/step
- `Logger`、`get_lr`、`SkipBatchSampler` 对 LoRA 和 Full SFT 一视同仁

真正的 `requires_grad=False/True` 逻辑在 `train_lora.py` 中完成。

## 5. 调用关系 (Call Graph)

```text
train_pretrain.py / train_full_sft.py / train_lora.py
  -> init_distributed_mode()
  -> setup_seed()
  -> init_model()
       -> AutoTokenizer
       -> MiniMindForCausalLM
  -> get_lr()
  -> Logger()
  -> lm_checkpoint()
  -> SkipBatchSampler()

RL / reward related scripts
  -> LMForRewardModel
       -> AutoModel.get_score()
```

- 对基础训练脚本而言：`trainer_utils.py` 是“公共训练中枢 (shared training utilities hub)”。
- 对模型侧：它直接连接 `MiniMindForCausalLM`。
- 对训练循环侧：它提供 LR、resume、DDP、日志、skip-batch 等关键支撑逻辑。
