# `rollout_engine.py` 代码解析（Code Analysis）

## 1. 文件概述 (File Overview)

`minimind/trainer/rollout_engine.py` 是 MiniMind 在线 RLHF 训练中的 **rollout abstraction layer**。

它并不直接实现 DPO/PPO/GRPO 的优化算法，而是负责统一“如何根据当前策略生成回答并返回 log-prob”的接口。

本文件包含四个核心组成：

1. **辅助函数**：`compute_per_token_logps`
2. **抽象基类 (abstract base class)**：`RolloutEngine`
3. **本地推理实现**：`TorchRolloutEngine`
4. **远程推理实现**：`SGLangRolloutEngine`
5. **工厂模式 (factory pattern)**：`create_rollout_engine`

在项目中的作用：

- `train_ppo.py` 依赖它做在线采样。
- `train_grpo.py` 依赖它做多样本在线采样。
- `train_dpo.py` 不依赖它，因为 DPO 直接吃离线 preference data。

---

## 2. 依赖说明 (Dependencies)

| 模块 | 作用 |
|---|---|
| `os`, `sys` | 处理路径与共享权重目录。 |
| `requests` | 与 SGLang 服务做 HTTP 通信。 |
| `torch`, `Tensor`, `dist` | 张量、设备与分布式广播。 |
| `ABC`, `abstractmethod` | 定义抽象基类接口。 |
| `nullcontext` | 当没有 autocast 时提供空上下文。 |
| `dataclass` | 定义结构化的 `RolloutResult`。 |
| `List`, `Optional`, `Tuple` | 类型注解。 |
| `DistributedDataParallel` | 兼容被 DDP 包装的模型。 |
| `AutoTokenizer` | SGLang backend 侧用于 decode/配置 tokenizer。 |

---

## 3. 逐行代码分析 (Line-by-Line Analysis)

### 第 1-4 行：文件头注释

- 这里给出使用 SGLang 时的启动命令示例。
- 说明：若选择 `sglang` 作为 rollout backend，需要先单独启动 HTTP 服务端。

### 第 5-20 行：导入依赖

- 包括本地推理、HTTP 通信、抽象类、数据类、类型注解、DDP 兼容和 tokenizer 支持。

### 第 23-36 行：`compute_per_token_logps`

这是一个重要的工具函数：给定完整 `input_ids`，计算末尾 `n_keep` 个 token 的逐 token log-prob。

- 第 24 行：函数签名，输入模型、token ids、保留长度、attention mask。
- 第 25-26 行：若 `n_keep <= 0`，直接返回空张量。
- 第 27 行：若模型被 DDP 包装，则先取 `.module`。
- 第 28 行：如果输入是 inference tensor，则 clone 一份，避免原地修改限制。
- 第 29 行：前向计算 logits，并通过 `logits_to_keep=n_keep + 1` 只保留最后相关部分，节省开销。
- 第 30 行：准备收集逐样本结果。
- 第 31-35 行：逐行遍历 batch：
  - 取当前样本对应的 logits 与最后 `n_keep` 个 token ids
  - 对 vocab 做 `log_softmax`
  - 再 `gather` 出真实 token 的 log-prob
- 第 36 行：拼成 `[B, n_keep]` 张量返回。

这个函数是 rollout 阶段返回 `old_logp` 的基础。

### 第 39-47 行：`RolloutResult` 数据类

- 用 `@dataclass` 封装 rollout 的统一返回结构：
  - `output_ids`：完整 prompt + completion
  - `completion_ids`：仅 completion 部分
  - `per_token_logps`：completion 区间逐 token log-prob
  - `completions`：文本形式 completion 列表
  - `prompt_lens`：每个样本 prompt 实际长度
  - `completion_mask`：completion 有效 token mask

统一结构让 PPO 和 GRPO 不需要关心 backend 是 torch 还是 SGLang。

### 第 50-60 行：`RolloutEngine` 抽象基类

- 第 51 行：定义抽象类，作为所有 rollout backend 的共同接口。
- 第 52 行：类属性 `tokenizer = None`。
- 第 54-56 行：抽象方法 `rollout(...)`，规定输入 prompt ids / mask / generation 参数，返回 `RolloutResult`。
- 第 58-60 行：抽象方法 `update_policy(...)`，规定策略更新后如何把新模型同步到 rollout 端。

这是典型的 **interface contract（接口契约）**。

### 第 63-70 行：`TorchRolloutEngine.__init__`

- 保存 policy model、tokenizer、device 和 autocast context。
- 这是“当前进程内直接调用模型 generate”的 backend。

### 第 71-92 行：`TorchRolloutEngine.rollout`

这是本地 PyTorch rollout 的主体。

- 第 72 行：若 policy model 被 DDP 包装，先取 `.module`。
- 第 73 行：若外部给了 autocast，就用它；否则用 `nullcontext()`。
- 第 74 行：在 `torch.no_grad()` 下执行生成。
- 第 75-84 行：调用 `model.generate(...)`：
  - prompt 与 attention mask 都按 `num_generations` 复制
  - 开启 `do_sample=True`
  - 设定温度 `temperature`
  - 指定 pad/eos token id
  - 最后 `.clone()`，避免某些后续视图问题
- 第 85 行：记录 prompt 长度。
- 第 86 行：切出 completion ids。
- 第 87 行：构造完整序列的 non-pad mask。
- 第 88 行：调用 `compute_per_token_logps(...)` 计算 completion 区间 old log-prob。
- 第 89 行：把 completion ids 解码成文本列表。
- 第 90-92 行：封装成 `RolloutResult` 返回。

值得注意：

- `completion_mask` 这里直接返回全 1，说明本地引擎默认 completion 全部有效；真正 EOS 截断通常由上层训练脚本再处理。

### 第 94-95 行：`TorchRolloutEngine.update_policy`

- 只做一件事：更新 `self.policy_model` 指针。
- 因为 torch backend 和训练进程共享同一 Python 进程，所以不需要额外序列化或远程同步。

### 第 98-106 行：`SGLangRolloutEngine.__init__`

- 初始化 SGLang 远程 backend。
- 第 101-103 行：保存 base URL、共享 checkpoint 路径与超时设置。
- 第 104 行：用 `AutoTokenizer.from_pretrained(model_path)` 加载 tokenizer。
- 第 105 行：把 `requests` 绑定到 `self.http`，便于后面 mock 或统一调用。

### 第 107-173 行：`SGLangRolloutEngine.rollout`

这是通过 HTTP 调用 SGLang 服务端生成文本的实现。

- 第 108-113 行：去掉左 padding，只保留有效 prompt token；然后按 `num_generations` 复制。
- 第 115-123 行：构造请求 payload：
  - `input_ids`
  - `sampling_params`（temperature、max_new_tokens、stop_token_ids）
  - `return_logprob=True`，要求服务端返回 token log-prob
- 第 125-126 行：POST 到 `/generate` 并检查状态码。
- 第 128-130 行：解析 JSON，兼容单样本或列表返回。
- 第 132-156 行：逐个结果提取：
  - `completion_ids`
  - `output_token_logprobs`
  - 处理 logprob 长度和 completion 长度不一致的情况
  - 拼出完整输出并 decode 成文本
- 第 158-160 行：确定 pad 后的最大 completion/output 长度。
- 第 162-163 行：定义辅助函数 `pad_to_tensor`。
- 第 165-173 行：把所有列表 pad 成同尺寸 tensor，封装为 `RolloutResult`。

和 torch backend 相比，SGLang backend 的重点是：

- **跨进程 / 跨服务通信**
- **把远程返回的不规则列表整理成规则张量**

### 第 175-194 行：`SGLangRolloutEngine.update_policy`

这是 SGLang 模式下最关键的同步逻辑。

- 第 176 行：默认同步状态 `ok=True`。
- 第 177 行：只有 rank 0 执行真实写盘和 HTTP 更新，避免多卡重复操作。
- 第 179-183 行：取出未包装/未 compile 包装的模型，把 state_dict 转成半精度 CPU 参数并保存到共享目录。
- 第 184 行：同步 tokenizer 到同一目录。
- 第 185 行：POST 到 `/update_weights_from_disk`，要求 SGLang 服务端热更新权重。
- 第 186-189 行：若失败则打印 warning，并将 `ok=False`。
- 第 190-192 行：如果是分布式训练，则把 rank 0 的更新结果广播给所有卡，并 `barrier()` 同步。
- 第 193 行：若任何一步失败，抛出异常。
- 第 194 行：返回更新是否成功。

这个设计让“训练进程”和“采样服务”解耦，但仍能在训练中途热切换到最新策略权重。

### 第 196-205 行：缓存与健康检查

- `flush_cache()`：请求远端服务清空缓存。
- `health()`：请求 `/health` 判断服务是否存活；异常时返回 `False`。

### 第 208-224 行：`create_rollout_engine` 工厂函数

- 第 209-218 行：定义工厂函数参数。
- 第 219-220 行：若 `engine_type == "torch"`，返回 `TorchRolloutEngine`。
- 第 221-222 行：若 `engine_type == "sglang"`，返回 `SGLangRolloutEngine`。
- 第 223-224 行：否则抛出 `ValueError`。

这就是标准 **factory pattern**：调用方只关心 `engine_type`，不关心具体实例化细节。

---

## 4. 关键算法解析 (Key Algorithm Deep-Dive)

### 4.1 抽象基类 (Abstract Base Class)

`RolloutEngine` 的核心价值是把“生成回答并返回 old log-prob”抽象成统一接口：

- `rollout(...)`
- `update_policy(...)`

于是上层训练代码（PPO / GRPO）可以只依赖接口，而不依赖具体 backend。

### 4.2 TorchRolloutEngine：native generation

`TorchRolloutEngine` 是最直接的实现：

- 直接调用当前 policy model 的 `generate`
- 再用 `compute_per_token_logps` 回算 completion token 的 old log-prob
- 不需要网络通信，适合单机或简洁实验环境

这类实现的优点是简单，缺点是 rollout 与训练争用同一进程/显存资源。

### 4.3 SGLangRolloutEngine：HTTP API backend

`SGLangRolloutEngine` 把 rollout 外包给独立的 SGLang 服务：

- 训练进程负责优化参数
- SGLang 进程负责高性能生成
- 通过 HTTP 交换 `input_ids`、生成结果和 log-probs

这适合：

- 训练/采样解耦
- 更灵活的部署
- potentially 更高吞吐的 rollout

### 4.4 Factory pattern

`create_rollout_engine(...)` 让上层脚本只写：

```python
rollout_engine = create_rollout_engine(engine_type=args.rollout_engine, ...)
```

这样：

- `train_ppo.py` 与 `train_grpo.py` 可以复用同一套接口
- 新增 backend 时，只需实现 `RolloutEngine` 子类并扩展工厂函数

### 4.5 Reward shaping 与本文件的边界

本文件**不负责 reward shaping**。

- 长度奖励 (length bonus)
- 思维结构奖励 (thinking structure reward)
- 重复惩罚 (repetition penalty)
- Reward model scoring

都是在 `train_ppo.py` 和 `train_grpo.py` 中完成的。

`rollout_engine.py` 只负责提供：

- completion token ids
- completion 文本
- old policy per-token log-prob
- prompt 长度与有效 mask

---

## 5. 调用关系 (Call Graph)

### 5.1 本文件被谁调用

```text
train_ppo.py
└─ create_rollout_engine(...) -> rollout_engine
   ├─ rollout_engine.rollout(...)       # 采样 response
   └─ rollout_engine.update_policy(...) # 同步最新 actor

train_grpo.py
└─ create_rollout_engine(...) -> rollout_engine
   ├─ rollout_engine.rollout(...)       # 每个 prompt 生成多条 response
   └─ rollout_engine.update_policy(...) # 同步最新 policy
```

### 5.2 与 `train_ppo.py` 的关系

- PPO 依赖 `RolloutResult.per_token_logps` 作为 `old_logp`。
- PPO 依赖 `output_ids`、`completion_ids`、`prompt_lens` 构造 response mask、GAE、policy/value loss。
- 每轮若 actor 更新后，PPO 会调用 `update_policy` 保证下次 rollout 用新策略。

### 5.3 与 `train_grpo.py` 的关系

- GRPO 依赖 `rollout(...)` 一次返回 `num_generations` 个 completion。
- 它使用 `per_token_logps` 计算 ratio，使用 `completion_mask` 计算 token-level loss。
- 同样会在保存点或间隔点调用 `update_policy`。

### 5.4 与 `train_dpo.py` 的关系

```text
train_dpo.py
└─ 不经过 rollout_engine
   因为 DPO 使用离线 chosen/rejected 样本，不做在线采样
```
