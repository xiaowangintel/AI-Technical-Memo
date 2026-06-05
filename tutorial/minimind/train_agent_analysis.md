# train_agent.py 代码分析

## 1. 文件概述 (File Overview)
- `minimind/trainer/train_agent.py` 是项目里的 **Agent RL (agent reinforcement learning)** 训练脚本。
- 它的目标不是普通 next-token supervised learning，而是让模型在 **multi-turn rollout** 中学会：先思考（`<think>`）、再按需发起 **tool call**、读取 **tool response**、最后给出答案。
- 在训练流水线（training pipeline）里，它位于 `SFT / Distillation` 之后：先加载已有语言模型权重（默认 `full_sft`），再通过 **GRPO/CISPO-style policy optimization** 做策略微调。
- 它同时接入三类外部能力：
  1. `AgentRLDataset` 提供 agent 任务样本；
  2. `RolloutEngine` 负责采样生成；
  3. `LMForRewardModel` + GT 校验（ground-truth validation）负责给 reward。

## 2. 依赖说明 (Dependencies)
- `os`, `sys`：修正包路径、处理保存目录。
- `re`, `json`, `math`, `random`, `signal`：分别用于正则解析、JSON 工具调用解析、数学工具、随机 thinking 开关、执行超时控制。
- `gc`：虽然本文件里没有显式调用，但常见于大模型训练场景的显存/对象管理预留。
- `argparse`：定义 CLI 参数（command-line arguments）。
- `warnings`：关闭 warning 输出。
- `torch`, `torch.nn.functional as F`：核心训练框架、logprob、KL、loss 计算。
- `torch.distributed as dist`：DDP（DistributedDataParallel）分布式训练。
- `nullcontext`：CPU 时替代 autocast context。
- `optim`、`CosineAnnealingLR`：优化器与学习率调度器（scheduler）。
- `DataLoader`, `DistributedSampler`：数据读取与分布式采样。
- `AutoTokenizer`：虽然本文件最终通过 `init_model` 间接拿 tokenizer，但 agent prompt/rendering 依赖 Hugging Face tokenizer 的 `apply_chat_template`。
- `MiniMindConfig`, `MiniMindForCausalLM`：MiniMind 模型配置与因果语言模型定义。
- `AgentRLDataset`：把 `agent_rl.jsonl` 转成 `messages/tools/gt` 三元组。
- `trainer_utils` 中的：
  - `Logger`：统一日志；
  - `is_main_process`：限制主进程保存/记录；
  - `lm_checkpoint`：保存/恢复 checkpoint；
  - `init_distributed_mode`：初始化 DDP；
  - `setup_seed`：固定随机种子；
  - `SkipBatchSampler`：断点续训时跳过已训练 batch；
  - `init_model`：加载 tokenizer 与 MiniMind 权重；
  - `LMForRewardModel`：加载 reward model。
- `create_rollout_engine`, `compute_per_token_logps`：前者创建 PyTorch 或 SGLang rollout backend，后者计算逐 token log probability。

## 3. 逐行代码分析 (Line-by-Line Analysis)

### 1-28 行：路径修正与训练基础设施导入
- 1-5 行通过 `__package__ = "trainer"` 与 `sys.path.append(...)` 保证脚本直接运行时，仍能找到 `model/`、`dataset/`、`trainer/` 下的模块。
- 7-28 行导入 RL 训练、分布式训练、rollout、checkpoint 所需的全部依赖。
- 24-27 行的几个项目内模块，构成了这个脚本的主调用骨架：`MiniMindConfig` / `AgentRLDataset` / `trainer_utils` / `rollout_engine`。

### 29-31 行：warning 控制与功能分区
- 29 行 `warnings.filterwarnings('ignore')` 让训练日志更干净。
- 31 行开始进入“工具与 reward”部分，说明此脚本把 agent 行为与 RL loss 紧密绑定，而不是把 tool use 放到推理时再单独拼接。

### 33-36 行：`rep_penalty` 重复惩罚 (repetition penalty)
- 把文本按 `word + punctuation` 形式切分为 token-like 单位。
- 构造 `n-gram`（默认 3-gram）。
- 若重复 3-gram 越多，惩罚越大，但上限由 `cap=0.5` 截断。
- 这是一个轻量级 **anti-repetition heuristic**，避免 agent 靠刷重复文本拿格式分。

### 38-46 行：`TOOLS` 工具定义 (tool schema)
- 这里手写了一个与 OpenAI function calling 类似的 **tool schema list**。
- 每个工具都包含：`name`、`description`、`parameters`、`required`。
- 定义的工具有：
  - `calculate_math`
  - `unit_converter`
  - `get_current_weather`
  - `get_current_time`
  - `get_exchange_rate`
  - `translate_text`
- 这些 schema 会被 `tokenizer.apply_chat_template(..., tools=tools)` 注入 prompt，模型据此学习 `<tool_call>{...}</tool_call>` 的输出格式。

### 48-54 行：模拟环境数据 (mock environment data)
- 这几行是工具环境的 **lookup table**：天气、时间、汇率、翻译、单位换算。
- 目的不是构建真实 API，而是提供一个可控、可重复（deterministic）的 agent 训练环境。
- RL 中，环境越稳定，reward variance 通常越小。

### 55-63 行：`MOCK_RESULTS` 模拟执行系统 (mock execution system)
- 每个工具名映射到一个 lambda，用于返回 JSON 风格结果。
- `calculate_math` 使用 `eval`，但限制了 `__builtins__`，只暴露 `math`，属于简化版安全沙箱（lightweight sandbox）。
- `unit_converter` 通过 `UNIT_DATA` 的倍率直接计算。
- 天气/时间/汇率/翻译工具都是查表返回。
- 该设计让 agent rollout 可以在训练时“真的调用工具”，而不是只做静态格式模仿。

### 65-73 行：`CHECK_ARGS` 参数校验 (argument validation)
- 这里为每个工具定义最小参数合法性检查。
- 作用不是执行工具，而是 reward 时判断 tool call 是否“有效”。
- 例如 `unit_converter` 要同时有 `value/from_unit/to_unit`；`get_current_time` 则永远合法。
- 这让 reward 不只看最终答案，也看 action format 是否正确。

### 75-81 行：`parse_tool_calls` 工具调用解析
- 用正则提取所有 `<tool_call>...</tool_call>` 片段。
- 对每个片段尝试 `json.loads`。
- 解析失败直接跳过，不抛异常。
- 这是一层 **fault-tolerant parser**：模型输出可能不严格合法，训练端不应因此崩溃。

### 83-95 行：`execute_tool` 工具执行与超时保护
- 84-85 行先根据名称拿到模拟函数；不存在则返回 `None`。
- 87-88 行通过 `signal.alarm(1)` 给每次工具执行加 1 秒超时。
- 89 行真正执行 mock tool。
- 90-91 行任何异常都吞掉并返回 `None`，防止 rollout 被坏 action 卡死。
- 92-94 行在 `finally` 中清除 alarm，属于标准超时清理逻辑。

### 96-105 行：`rollout_single` 初始化单样本多轮 rollout
- `rollout_single(...)` 是本脚本最核心的函数之一。
- 它维护：
  - `all_outputs`：每一轮 assistant 原始输出；
  - `prompt_ids`：首轮 prompt token；
  - `response_ids`：后续所有被训练/观察到的 token；
  - `response_mask`：哪些 token 应参与 policy training；
  - `response_old_logps`：采样时旧策略的 logprob；
  - `unfinished`：达到 `max_turns` 但仍在调用工具时记为未完成。
- 105 行 `open_thinking = random.random() < thinking_ratio`：随机决定这一条样本是否开启 **open thinking mode**。

### 106-118 行：构造上下文并执行一轮生成
- 107 行用 `apply_chat_template` 把 `messages + tools + open_thinking` 渲染成完整 prompt。
- 108-110 行把文本 prompt 编码成 tensor，并在首轮缓存 `prompt_ids`。
- 112-118 行调用 `rollout_engine.rollout(...)`：
  - 输入当前上下文；
  - 只采样 1 条；
  - 限制 `max_new_tokens`；
  - 用 `temperature=0.8` 做随机探索（exploration）。

### 119-130 行：提取新生成 token 与旧策略 logprob
- 119-120 行拿到本轮 completion ids 和对应 `per_token_logps`。
- 121 行若 token 数与 logprob 数不一致会打印日志，但不中断训练。
- 122-124 行过滤 `pad/eos`，保证后续训练只看有效 completion token。
- 125-130 行把本轮结果追加到：
  - `all_outputs`
  - `response_ids`
  - `response_mask`（全部记 1）
  - `response_old_logps`
- 这里的 1 表示：这些 token 是 **model-generated tokens**，应该进入 policy loss。

### 131-143 行：解析 tool call 并执行 mock tool
- 131 行对当前输出 `new_text` 做 tool call 解析。
- 132-133 行若没有 tool call，说明 agent 选择直接回答，多轮 rollout 结束。
- 134 行若当前已经是最后一轮仍有 tool call，则标记 `unfinished=True`。
- 135 行把 assistant 输出写回 `messages`，形成下一轮对话历史。
- 136-143 行逐个工具调用：
  - 解析 `name/arguments`；
  - 若 `arguments` 是字符串，再次反序列化；
  - `execute_tool` 得到结果；
  - 裁剪到 2048 字符，防止 tool response 过长撑爆 tokenizer；
  - 作为 `role="tool"` 消息追加回对话。

### 145-153 行：把工具观察结果拼回序列，但不训练这些 token
- `observe_context` 是“assistant 调了工具之后，模型下一轮将看到的完整上下文”。
- 146-148 行通过重新渲染聊天模板，计算新出现的 observation token（`obs_delta`）。
- 149-151 行把 observation token 也追加到 `response_ids`，但 `response_mask` 全部记为 0，`response_old_logps` 记 0。
- 这一步非常关键：**tool response 会进入 context，但不会进入 loss**。
- 这正是 agent RL 中常见的 **response_mask design**：只训练模型自己的 action tokens，不训练环境返回 tokens。

### 154-156 行：返回单样本 rollout 结果
- 返回最终输出、最终上下文、prompt ids、response ids、mask、旧 logprob、多轮输出列表、unfinished 标记。
- 这些对象后面会被 `rl_train_epoch` 打包成 batch 训练样本。

### 158-179 行：`rollout_batch` 批量 rollout
- 对 batch 中每个样本，循环生成 `num_gen` 个 completion。
- 169 行对 `messages` 做浅拷贝，避免多次生成互相污染上下文。
- 170 行调用 `rollout_single`。
- 171-178 行把所有中间结果收集到多个 list 中。
- 返回值是“并行 batch 版”的 rollout cache，后续用于 reward、advantage、policy loss。

### 181-185 行：`validate_gt_in_text` GT 匹配函数
- 这个函数负责从最终文本里检查 **ground truth (GT)** 是否出现。
- 183 行先构造原文本与去掉逗号的数字文本版本。
- 184 行用正则抽取所有数值。
- 185 行返回命中的 GT 集合，支持两种匹配：
  1. 字符串包含（case-insensitive substring）；
  2. 数值匹配（numeric match），容差 `1e-6`。
- 这让 reward 既能验证文字答案，也能验证数值答案。

### 187-217 行：`calculate_rewards` 的“无工具调用”分支
- 188 行先初始化 reward tensor。
- 189-199 行对每个 completion：
  - 找到所属样本 `sample_idx`；
  - 取出该样本可用工具、各轮输出、unfinished 状态；
  - 去掉 `<think>` 前缀，得到每轮 answer 文本；
  - 汇总所有 tool call。
- 200 行先对 `<tool_call>` 标签不配对做扣分。
- 201-217 行若完全没有 tool call：
  - 203 行按总长度给格式分；
  - 204-208 行若包含 `</think>`，则检查 thinking 长度与闭合次数；
  - 209-215 行若配置了 `reward_model`，则把历史对话和最终 answer 送给 RM 打分；
  - 216 行扣掉重复惩罚；
  - 217 行把最终 reward 截断到 `[-3, 3]`。
- 这个分支适合“无需工具即可直接回答”的问题。

### 218-238 行：`calculate_rewards` 的“有工具调用”分支
- 220 行取出该样本 GT。
- 221-229 行统计 `valid_call_count`：只要工具名合法、参数校验通过，就算有效调用。
- 229-230 行用 `tool_gap` 衡量“调用数量/有效性”和 GT 长度之间的差异，并转换成对齐分（alignment reward）。
- 232 行如果 rollout 因轮数上限结束，则 `final_text=""`，否则取最后一次 `</tool_call>` 后的文本作为最终答案。
- 233-234 行用 `validate_gt_in_text` 对最终答案做 GT 校验，命中比例越高，奖励越高。
- 235 行对 unfinished rollout 扣分。
- 236 行继续加入重复惩罚。
- 237 行仍然把总分裁剪到 `[-3, 3]`。
- 与无工具分支不同，这里核心 reward 主要来自 **tool correctness + GT verification**，而不是 reward model。

### 241-250 行：`rl_train_epoch` 入口与 rollout 触发
- 241 行定义一个 epoch 的 RL 训练函数。
- 243-247 行从 `DataLoader` 取出 `messages/tools/gt`。
- 249-250 行在 `torch.no_grad()` 下先做 rollout：
  - 这是采样阶段（sampling phase），不是反向传播阶段；
  - 会返回 completion、context、mask、旧 logprob 等完整缓存。

### 252-270 行：把 rollout 结果打包为统一张量
- 252 行重新渲染 prompt 文本，供 reward model 使用。
- 253-263 行把每个样本整理为 `(ids, mask, prompt_len, old_logps)`：
  - `ids = prompt + response`；
  - `mask = [0]*prompt + response_mask`；
  - `old_logps` 与 token 对齐；
  - 超过 `max_total_len` 时从左侧截断。
- 262 行用第一个 `mask==1` 的位置推断 `prompt_len`。
- 264-270 行把变长样本 pad 成 batch tensor：`input_ids`、`prompt_lens`、`full_response_masks`、`old_per_token_logps`、`full_mask`。

### 272-282 行：计算 reward，并得到当前/参考策略 logprob
- 272 行先调用 `calculate_rewards(...)` 得到标量 reward。
- 274-279 行前向当前 policy model：
  - 若是 DDP，就先取 `model.module`；
  - 若启用 MoE，再加上 `aux_loss`；
  - `per_token_logps` 通过 `log_softmax + gather` 拿到每个真实输出 token 的当前策略概率。
- 281-282 行对 `ref_model` 调 `compute_per_token_logps`，得到 reference policy 的逐 token logprob。
- `ref_model` 是一个冻结的旧策略，用于 KL regularization。

### 284-292 行：构造 completion mask 与有效样本过滤
- 284 行把 `full_response_masks[:, 1:]` 作为实际对齐到 logprob 的 mask。
- 285-290 行如果序列中提前出现 EOS，只保留 EOS 之前的 completion token。
- 291 行统计每条 completion 的有效 token 数。
- 292 行只保留至少有 1 个有效 token 的样本，避免除以 0。

### 294-312 行：debug 模式打印 rollout 细节
- 仅在 `--debug_mode` 且达到 `debug_interval` 时打印。
- 会输出：GT、完整上下文、prompt/completion 边界、completion 文本、reward。
- 这对于排查 agent 是否真的学会调用工具、是否把 `<think>` 和 `<tool_call>` 格式写对，非常有帮助。

### 313-317 行：组内标准化 advantage
- 313 行把 reward reshape 为 `[batch, num_generations]`。
- 314-315 行计算同一 prompt 下多条生成的均值和标准差。
- 316 行得到 `advantages = (r - mean) / (std + 1e-4)`。
- 这是典型的 **group-relative advantage normalization**，与 GRPO 思路一致：不依赖 value model，只靠组内对比。

### 318-333 行：KL、ratio，以及 GRPO/CISPO policy loss
- 318 行 `kl_div = ref_logp - current_logp`。
- 319 行 `per_token_kl = exp(Δ) - Δ - 1`，这是常见的 token-level 近似 KL 形式。
- 320 行 `ratio = exp(logπθ - logπold)`，即新旧策略概率比。
- 321-324 行若 `loss_type == "cispo"`：
  - 对 `ratio` 只做上界 clamp；
  - 用 `detach()` 的 `clamped_ratio` 稳定训练；
  - 目标项是 `clamped_ratio * A * logπθ`，再减 `beta * KL`。
- 324-328 行若是 `grpo`：
  - 构造 PPO 风格 `clipped_ratio`；
  - 用 `min(ratio*A, clipped_ratio*A)` 做裁剪目标。
- 329-331 行只对 `completion_mask==1` 的 token 求平均，再加 `aux_loss`，并按 `accumulation_steps` 缩放。
- 332 行开始反向传播。

### 334-336 行：优化器更新
- 到达梯度累积边界时：
  - 可选梯度裁剪；
  - `optimizer.step()`；
  - `scheduler.step()`；
  - `optimizer.zero_grad()`。
- RL 这里没有 `GradScaler`，而是直接 backward。

### 338-349 行：训练日志与可视化记录
- 339-345 行汇总当前 loss、平均 reward、平均长度、KL、advantage 统计量、学习率。
- 346 行打印一条较完整的 RL 训练日志。
- 347-348 行若启用 `wandb/swanlab`，则上报相同指标。
- 这些指标能帮助判断：
  - reward 是否上涨；
  - KL 是否失控；
  - rollout 是否变短/变长；
  - advantage 是否塌缩。

### 350-364 行：保存模型与同步 rollout engine
- 350-360 行在主进程保存当前模型权重与 resume checkpoint。
- 352-353 行根据 `use_moe` 拼接文件名后缀。
- 354-356 行兼容 DDP 和 `torch.compile` 包裹后的真实模型对象。
- 357 行保存半精度权重到 `args.save_dir`。
- 358-359 行用 `lm_checkpoint` 额外保存 optimizer/scheduler/epoch/step。
- 363 行每次保存后执行 `rollout_engine.update_policy(model)`，让 rollout 阶段使用最新策略。
- 对 `SGLangRolloutEngine` 来说，这一步会把新权重同步到外部推理服务。

### 365-370 行：epoch 末尾补一次优化步
- 如果最后一个 batch 没正好落在 `accumulation_steps` 边界，就在 epoch 结束时补做一次 `step()`。
- 这是标准的梯度累积收尾逻辑。

### 373-412 行：CLI 参数定义
- 374 行定义脚本描述 `MiniMind Agent RL`。
- 375-386 行：保存路径、epoch、batch size、学习率、dtype、日志/保存频率等通用训练参数。
- 387-393 行：模型大小、序列长度、数据路径。
- 394-399 行：RL 核心超参，包括 `num_generations`、`beta`、`loss_type`、`epsilon`、`epsilon_high`。
- 399-403 行：加载权重、断点续训、`use_compile`。
- 404-406 行：debug 与 thinking 比例。
- 407-412 行：reward model 路径与 rollout engine（`torch` / `sglang`）配置。
- 这一段决定了脚本既能本地 rollout，也能把采样卸载给 SGLang 服务。

### 414-425 行：分布式初始化与 autocast 设置
- 414-416 行初始化 DDP，并按 rank 设置随机种子。
- 418-421 行创建输出目录、构建 `MiniMindConfig`、根据 `--from_resume` 尝试加载 checkpoint。
- 423-425 行根据设备与 dtype 构造 `autocast_ctx`，供 rollout/model forward 使用。

### 427-432 行：实验追踪 (experiment tracking)
- 若开启 `--use_wandb` 且当前是主进程，就初始化 `swanlab as wandb`。
- 如果存在 resume checkpoint，则复用 `wandb_id`，实现断点续跑。

### 434-451 行：初始化 policy/ref/reward/rollout 子系统
- 434 行通过 `init_model` 加载 policy model 和 tokenizer。
- 436-437 行再加载一个同配置同权重的 `ref_model`，并冻结为 reference policy。
- 439-440 行加载 reward model。
- 442-451 行调用 `create_rollout_engine(...)`：
  - `torch` 模式直接用当前模型生成；
  - `sglang` 模式通过 HTTP 调远程服务生成。
- 这几步一起构成完整 **RLHF-like / agent-RL training loop** 的“采样-评分-更新”基础设施。

### 452-460 行：数据集、优化器、scheduler
- 452 行创建 `AgentRLDataset`，样本格式是 `messages + tools + gt`。
- 453 行若是 DDP，则创建 `DistributedSampler`。
- 454 行使用 `AdamW`。
- 455 行自定义 `collate_fn`，因为 agent 样本是嵌套结构，不能直接默认堆叠。
- 456-459 行先构建一个只用于计数的 DataLoader，再计算总 optimizer step 数，最后创建 `CosineAnnealingLR`。

### 461-475 行：恢复训练、编译与 DDP 包装
- 461-467 行若有 checkpoint，则恢复模型、优化器、scheduler、epoch、step。
- 469-472 行可选 `torch.compile`，并在编译后立刻把新 policy 同步给 rollout engine。
- 473-474 行若启用 DDP，对训练模型做 `DistributedDataParallel` 包装。
- 475 行再次 `update_policy(model)`，确保 rollout 使用的是最终包装后的模型引用。

### 477-489 行：主训练循环与清理
- 477 行开始 epoch 循环。
- 478-482 行设置 sampler epoch、重新打乱索引、构造带 skip 能力的 `SkipBatchSampler` 与正式 DataLoader。
- 483-487 行若是断点续训，则跳过前 `start_step` 个 step；否则正常调用 `rl_train_epoch`。
- 489 行在分布式环境结束时销毁 process group。

## 4. 关键算法解析 (Key Algorithm Deep-Dive)

### 4.1 Agentic RL：multi-turn rollout + tool execution
- 这个脚本不是“一次 prompt，一次 answer”的单轮 RL，而是 **agentic multi-turn rollout**。
- 单条样本的执行逻辑是：
  1. 根据当前 `messages + tools` 渲染 prompt；
  2. 模型生成一段 assistant 输出；
  3. 若输出中含 `<tool_call>`，训练器解析 JSON；
  4. 执行 mock tool，得到 `<tool_response>`；
  5. 把 tool response 作为新观察（observation）拼回上下文；
  6. 继续下一轮生成，直到没有 tool call 或达到 `max_turns`。
- 这相当于在文本环境中实现了一个简化版 **interaction loop**。

### 4.2 `response_mask`：只训练模型自己的 token
- `response_ids` 里既包含：
  - 模型自己生成的 assistant token；
  - 工具返回后补进上下文的 observation token。
- 但 `response_mask` 做了严格区分：
  - 模型生成 token -> `1`
  - tool response token -> `0`
- 后续 `policy_loss` 只在 `completion_mask == 1` 的位置求和，因此不会把环境返回文本当成“模型应该预测的目标”。
- 这是 agent training 非常重要的一点，否则模型会被错误地训练去“复述工具输出”。

### 4.3 GT-based reward：结合动作合法性与答案正确性
- 对工具任务，reward 主要来自两层：
  1. **action-level**：tool 名是否在允许集合里、参数是否齐全、调用次数是否和 GT 对齐；
  2. **answer-level**：最终回答里是否出现 GT 字符串/数值。
- `validate_gt_in_text` 同时支持 substring match 与 numeric match。
- 因为很多工具题最终答案是数字，纯字符串比较会过脆弱，所以这里额外做了数字抽取与容差比较。

### 4.4 GRPO / CISPO policy loss
- 组内 advantage：
  \[
  A_i = \frac{r_i - \mu_{group}}{\sigma_{group} + 10^{-4}}
  \]
- 新旧策略比率：
  \[
  ratio = \exp(\log \pi_\theta - \log \pi_{old})
  \]
- 参考策略 KL 项：
  \[
  \Delta = \log \pi_{ref} - \log \pi_\theta, \quad KL_{token} \approx e^{\Delta} - \Delta - 1
  \]
- **GRPO branch** 本质上是 PPO-style clipped objective：
  \[
  -\min(ratio \cdot A, clip(ratio) \cdot A) + \beta KL
  \]
- **CISPO branch** 则对 ratio 做上界截断，并把截断后的系数从梯度里 `detach()`，更强调稳定性（stability）。
- 两种 loss 最后都只在模型行为 token 上累计，再加上 MoE 的 `aux_loss`。

### 4.5 Thinking 与 tool calling 的模板协同
- 脚本中 `open_thinking` 只是一个随机开关，但它依赖 tokenizer 的 `chat_template` 支持：
  - 生成提示里预留 `<think>`；
  - assistant 历史里可含 `<think>...</think>`；
  - tool 调用使用 `<tool_call>` 包裹 JSON。
- 所以 agent RL 与 tokenizer/template 是强耦合（tightly coupled）的。

## 5. 调用关系 (Call Graph)
- `train_agent.py::__main__`
  -> `init_distributed_mode()` / `setup_seed()`
  -> `MiniMindConfig(...)`
  -> `init_model(...)`
  -> `AutoTokenizer.from_pretrained(...)` + `MiniMindForCausalLM(...)`
- 数据流：
  - `AgentRLDataset` -> `DataLoader(collate_fn)` -> `rl_train_epoch()`
- rollout 流：
  - `rl_train_epoch()` -> `rollout_batch()` -> `rollout_single()`
  - `rollout_single()` -> `tokenizer.apply_chat_template()`
  - `rollout_single()` -> `create_rollout_engine(...).rollout()`
  - `rollout_single()` -> `parse_tool_calls()` -> `execute_tool()`
- reward 流：
  - `rl_train_epoch()` -> `calculate_rewards()`
  - `calculate_rewards()` -> `validate_gt_in_text()` / `rep_penalty()` / `LMForRewardModel.get_score()`
- optimization 流：
  - `rl_train_epoch()` -> `compute_per_token_logps(ref_model, ...)`
  - `rl_train_epoch()` -> GRPO/CISPO loss -> `optimizer.step()` -> `scheduler.step()`
- checkpoint 流：
  - `rl_train_epoch()` -> `lm_checkpoint(...)`
  - `rollout_engine.update_policy(model)` 保证采样策略与训练策略同步。
- 与项目其余脚本的关系：
  - 默认通过 `--from_weight full_sft` 加载 SFT 权重，因此通常接在 `train_full_sft.py` 或 `train_distillation.py` 之后；
  - 其 `tokenizer.apply_chat_template()` 依赖 tokenizer 配置中的 `chat_template`，而这正是 `train_tokenizer.py` 会生成/演示的关键配置之一。
