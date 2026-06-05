# reinforcement_learning_rpc_test.py — Code Analysis / 代码分析

## Source / 来源
- File: `torch/testing/_internal/distributed/rpc/examples/reinforcement_learning_rpc_test.py`
- Repository: `pytorch/pytorch`
- Purpose (EN): Implements internal PyTorch testing infrastructure for reinforcement learning rpc test, including reusable fixtures, comparison helpers, and backend-specific test support.
- 用途 (CN): 实现与 reinforcement learning rpc test 相关的 PyTorch 内部测试基础设施，包括可复用夹具、比较辅助函数以及特定后端的测试支持。

## Line-by-Line Analysis / 逐行分析
### Lines 1-7
```python
# mypy: allow-untyped-defs

# If you need to modify this file to make this test pass, please also apply same edits accordingly to
# https://github.com/pytorch/examples/blob/master/distributed/rpc/rl/main.py
# and https://pytorch.org/tutorials/intermediate/rpc_tutorial.html

import numpy as np
```
- EN: This block handles conditional control flow. Key symbols: no dominant local symbols.
- CN: 该代码块处理条件控制流。关键符号：无明显局部符号。

### Lines 8-14
```python

import torch
import torch.distributed.rpc as rpc
import torch.nn as nn
import torch.nn.functional as F
import torch.optim as optim
from torch.distributed.rpc import remote, rpc_async, rpc_sync, RRef
```
- EN: Imports Python modules needed by this part of the subsystem. Internal imports: `torch`, `torch.distributed.rpc`, `torch.nn`, `torch.nn.functional`, `torch.optim`; external imports: none.
- CN: 引入该子系统这一部分所需的 Python 模块。内部导入：`torch`, `torch.distributed.rpc`, `torch.nn`, `torch.nn.functional`, `torch.optim`；外部导入：无。

### Lines 15-21
```python
from torch.distributions import Categorical
from torch.testing._internal.dist_utils import dist_init, worker_name
from torch.testing._internal.distributed.rpc.rpc_agent_test_fixture import (
    RpcAgentTestFixture,
)


```
- EN: Imports Python modules needed by this part of the subsystem. Internal imports: `torch.distributions`, `torch.testing._internal.dist_utils`, `torch.testing._internal.distributed.rpc.rpc_agent_test_fixture`; external imports: none.
- CN: 引入该子系统这一部分所需的 Python 模块。内部导入：`torch.distributions`, `torch.testing._internal.dist_utils`, `torch.testing._internal.distributed.rpc.rpc_agent_test_fixture`；外部导入：无。

### Lines 22-31
```python
TOTAL_EPISODE_STEP = 5000
GAMMA = 0.1
SEED = 543


def _call_method(method, rref, *args, **kwargs):
    r"""
    a helper function to call a method on the given RRef
    """
    return method(rref.local_value(), *args, **kwargs)
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `TOTAL_EPISODE_STEP`, `GAMMA`, `SEED`, `_call_method`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`TOTAL_EPISODE_STEP`, `GAMMA`, `SEED`, `_call_method`。

### Lines 32-40
```python


def _remote_method(method, rref, *args, **kwargs):
    r"""
    a helper function to run method on the owner of rref and fetch back the
    result using RPC
    """
    args = [method, rref] + list(args)
    return rpc_sync(rref.owner(), _call_method, args=args, kwargs=kwargs)
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `_remote_method`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`_remote_method`。

### Lines 41-49
```python


class Policy(nn.Module):
    r"""
    Borrowing the ``Policy`` class from the Reinforcement Learning example.
    Copying the code to make these two examples independent.
    See https://github.com/pytorch/examples/tree/master/reinforcement_learning
    """

```
- EN: Defines a class or structured abstraction used by the file. Representative symbols: `Policy`.
- CN: 定义该文件使用的类或结构化抽象。代表性符号：`Policy`。

### Lines 50-58
```python
    def __init__(self) -> None:
        super().__init__()
        self.affine1 = nn.Linear(4, 128)
        self.dropout = nn.Dropout(p=0.6)
        self.affine2 = nn.Linear(128, 2)

        self.saved_log_probs = []
        self.rewards = []

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `__init__`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`__init__`。

### Lines 59-65
```python
    def forward(self, x):
        x = self.affine1(x)
        x = self.dropout(x)
        x = F.relu(x)
        action_scores = self.affine2(x)
        return F.softmax(action_scores, dim=1)

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `forward`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`forward`。

### Lines 66-74
```python

class DummyEnv:
    r"""
    A dummy environment that implements the required subset of the OpenAI gym
    interface. It exists only to avoid a dependency on gym for running the
    tests in this file. It is designed to run for a set max number of iterations,
    returning random states and rewards at each step.
    """

```
- EN: Defines a class or structured abstraction used by the file. Representative symbols: `DummyEnv`.
- CN: 定义该文件使用的类或结构化抽象。代表性符号：`DummyEnv`。

### Lines 75-81
```python
    def __init__(self, state_dim=4, num_iters=10, reward_threshold=475.0):
        self.state_dim = state_dim
        self.num_iters = num_iters
        self.iter = 0
        self.reward_threshold = reward_threshold

    def seed(self, manual_seed):
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `__init__`, `seed`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`__init__`, `seed`。

### Lines 82-88
```python
        torch.manual_seed(manual_seed)

    def reset(self):
        self.iter = 0
        return torch.randn(self.state_dim)

    def step(self, action):
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `reset`, `step`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`reset`, `step`。

### Lines 89-95
```python
        self.iter += 1
        state = torch.randn(self.state_dim)
        reward = torch.rand(1).item() * self.reward_threshold
        done = self.iter >= self.num_iters
        info = {}
        return state, reward, done, info

```
- EN: This block returns results to callers or downstream stages. Key symbols: no dominant local symbols.
- CN: 该代码块向调用方或后续阶段返回结果。关键符号：无明显局部符号。

### Lines 96-104
```python

class Observer:
    r"""
    An observer has exclusive access to its own environment. Each observer
    captures the state from its environment, and send the state to the agent to
    select an action. Then, the observer applies the action to its environment
    and reports the reward to the agent.
    """

```
- EN: Defines a class or structured abstraction used by the file. Representative symbols: `Observer`.
- CN: 定义该文件使用的类或结构化抽象。代表性符号：`Observer`。

### Lines 105-118
```python
    def __init__(self) -> None:
        self.id = rpc.get_worker_info().id
        self.env = DummyEnv()
        self.env.seed(SEED)

    def run_episode(self, agent_rref, n_steps):
        r"""
        Run one episode of n_steps.
        Arguments:
            agent_rref (RRef): an RRef referencing the agent object.
            n_steps (int): number of steps in this episode
        """
        state, _ep_reward = self.env.reset(), 0
        for _ in range(n_steps):
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `__init__`, `run_episode`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`__init__`, `run_episode`。

### Lines 119-127
```python
            # send the state to the agent to get an action
            action = _remote_method(Agent.select_action, agent_rref, self.id, state)

            # apply the action to the environment, and get the reward
            state, reward, done, _ = self.env.step(action)

            # report the reward to the agent for training purpose
            _remote_method(Agent.report_reward, agent_rref, self.id, reward)

```
- EN: This block iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 128-141
```python
            if done:
                break


class Agent:
    def __init__(self, world_size):
        self.ob_rrefs = []
        self.agent_rref = RRef(self)
        self.rewards = {}
        self.saved_log_probs = {}
        self.policy = Policy()
        self.optimizer = optim.Adam(self.policy.parameters(), lr=1e-2)
        self.eps = np.finfo(np.float32).eps.item()
        self.running_reward = 0
```
- EN: Defines a class or structured abstraction used by the file. Representative symbols: `Agent`, `__init__`.
- CN: 定义该文件使用的类或结构化抽象。代表性符号：`Agent`, `__init__`。

### Lines 142-148
```python
        self.reward_threshold = DummyEnv().reward_threshold
        for ob_rank in range(1, world_size):
            ob_info = rpc.get_worker_info(worker_name(ob_rank))
            self.ob_rrefs.append(remote(ob_info, Observer))
            self.rewards[ob_info.id] = []
            self.saved_log_probs[ob_info.id] = []

```
- EN: This block iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 149-155
```python
    def select_action(self, ob_id, state):
        r"""
        This function is mostly borrowed from the Reinforcement Learning example.
        See https://github.com/pytorch/examples/tree/master/reinforcement_learning
        The main difference is that instead of keeping all probs in one list,
        the agent keeps probs in a dictionary, one key per observer.

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `select_action`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`select_action`。

### Lines 156-163
```python
        NB: no need to enforce thread-safety here as GIL will serialize
        executions.
        """
        probs = self.policy(state.unsqueeze(0))
        m = Categorical(probs)
        action = m.sample()
        self.saved_log_probs[ob_id].append(m.log_prob(action))
        return action.item()
```
- EN: This block returns results to callers or downstream stages; protects shared state or ordering guarantees. Key symbols: no dominant local symbols.
- CN: 该代码块向调用方或后续阶段返回结果；保护共享状态或执行顺序保证。关键符号：无明显局部符号。

### Lines 164-170
```python

    def report_reward(self, ob_id, reward):
        r"""
        Observers call this function to report rewards.
        """
        self.rewards[ob_id].append(reward)

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `report_reward`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`report_reward`。

### Lines 171-182
```python
    def run_episode(self, n_steps=0):
        r"""
        Run one episode. The agent will tell each observer to run n_steps.
        """
        # make async RPC to kick off an episode on all observers
        futs = [
            rpc_async(
                ob_rref.owner(),
                _call_method,
                args=(Observer.run_episode, ob_rref, self.agent_rref, n_steps),
            )
            for ob_rref in self.ob_rrefs
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `run_episode`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`run_episode`。

### Lines 183-189
```python
        ]

        # wait until all observers have finished this episode
        for fut in futs:
            fut.wait()

    def finish_episode(self):
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `finish_episode`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`finish_episode`。

### Lines 190-197
```python
        r"""
        This function is mostly borrowed from the Reinforcement Learning example.
        See https://github.com/pytorch/examples/tree/master/reinforcement_learning
        The main difference is that it joins all probs and rewards from
        different observers into one list, and uses the minimum observer rewards
        as the reward of the current episode.
        """

```
- EN: This block implements local helper logic for reinforcement learning rpc test. Key symbols: no dominant local symbols.
- CN: 该代码块实现与 reinforcement learning rpc test 相关的局部辅助逻辑。关键符号：无明显局部符号。

### Lines 198-207
```python
        # joins probs and rewards from different observers into lists
        R, probs, rewards = 0, [], []
        for ob_id in self.rewards:
            probs.extend(self.saved_log_probs[ob_id])
            rewards.extend(self.rewards[ob_id])

        # use the minimum observer reward to calculate the running reward
        min_reward = min(sum(self.rewards[ob_id]) for ob_id in self.rewards)
        self.running_reward = 0.05 * min_reward + (1 - 0.05) * self.running_reward

```
- EN: This block iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 208-214
```python
        # clear saved probs and rewards
        for ob_id in self.rewards:
            self.rewards[ob_id] = []
            self.saved_log_probs[ob_id] = []

        policy_loss, returns = [], []
        for r in rewards[::-1]:
```
- EN: This block iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 215-225
```python
            R = r + GAMMA * R
            returns.insert(0, R)
        returns = torch.tensor(returns)
        returns = (returns - returns.mean()) / (returns.std() + self.eps)
        for log_prob, R in zip(probs, returns, strict=True):
            policy_loss.append(-log_prob * R)
        self.optimizer.zero_grad()
        policy_loss = torch.cat(policy_loss).sum()
        policy_loss.backward()
        self.optimizer.step()
        return min_reward
```
- EN: This block handles tensor metadata or sample values; iterates over collections or execution units; returns results to callers or downstream stages. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值；遍历集合或执行单元；向调用方或后续阶段返回结果。关键符号：无明显局部符号。

### Lines 226-232
```python


def run_agent(agent, n_steps):
    while True:
        agent.run_episode(n_steps=n_steps)
        agent.finish_episode()

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `run_agent`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`run_agent`。

### Lines 233-239
```python
        if agent.running_reward > agent.reward_threshold:
            print(f"Solved! Running reward is now {agent.running_reward}!")
            break


class ReinforcementLearningRpcTest(RpcAgentTestFixture):
    @dist_init(setup_rpc=False)
```
- EN: Defines a class or structured abstraction used by the file. Representative symbols: `ReinforcementLearningRpcTest`.
- CN: 定义该文件使用的类或结构化抽象。代表性符号：`ReinforcementLearningRpcTest`。

### Lines 240-252
```python
    def test_rl_rpc(self):
        if self.rank == 0:
            # Rank 0 is the agent.
            rpc.init_rpc(
                name=worker_name(self.rank),
                backend=self.rpc_backend,
                rank=self.rank,
                world_size=self.world_size,
                rpc_backend_options=self.rpc_backend_options,
            )
            agent = Agent(self.world_size)
            run_agent(agent, n_steps=int(TOTAL_EPISODE_STEP / (self.world_size - 1)))

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `test_rl_rpc`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`test_rl_rpc`。

### Lines 253-265
```python
            # Ensure training was run. We don't really care about whether the task was learned,
            # since the purpose of the test is to check the API calls.
            self.assertGreater(agent.running_reward, 0.0)
        else:
            # Other ranks are observers that passively wait for instructions from the agent.
            rpc.init_rpc(
                name=worker_name(self.rank),
                backend=self.rpc_backend,
                rank=self.rank,
                world_size=self.world_size,
                rpc_backend_options=self.rpc_backend_options,
            )
        rpc.shutdown()
```
- EN: This block checks invariants or expected outcomes; bridges to backend-specific execution artifacts; handles conditional control flow; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；桥接到特定后端的执行产物；处理条件控制流；遍历集合或执行单元。关键符号：无明显局部符号。


## Key Concepts / 关键概念
- EN: Testing assertions — the file focuses on checking values, errors, or textual expectations.
  CN: 测试断言——该文件重点检查数值、异常或文本期望是否满足要求。
- EN: Tensor construction and metadata — the code manages sample tensors, dtypes, layouts, or device placement.
  CN: 张量构造与元数据——代码会管理示例张量、数据类型、布局或设备放置。
- EN: Reusable test infrastructure — the module provides shared helpers, fixtures, or data for many test suites.
  CN: 可复用测试基础设施——该模块为多个测试套件提供共享辅助函数、夹具或数据。
- EN: Distributed testing — the implementation coordinates multi-process or sharded training checks.
  CN: 分布式测试——该实现会协调多进程或分片训练相关的检查逻辑。
- EN: Runtime execution — the file schedules or executes native runtime programs and delegates.
  CN: 运行时执行——该文件会调度或执行原生运行时程序及其 delegate。
- EN: Backend lowering — the code bridges higher-level programs to executable backend artifacts.
  CN: 后端降级——代码把更高层程序桥接到可执行的后端产物。

## Dependencies / 依赖关系
- Internal imports / 内部导入: `torch`, `torch.distributed.rpc`, `torch.nn`, `torch.nn.functional`, `torch.optim`, `torch.distributions`, `torch.testing._internal.dist_utils`, `torch.testing._internal.distributed.rpc.rpc_agent_test_fixture`
- External imports / 外部导入: `numpy`
- Representative symbols / 代表性符号: `TOTAL_EPISODE_STEP`, `GAMMA`, `SEED`, `_call_method`, `_remote_method`, `Policy`, `DummyEnv`, `Observer`, `Agent`, `run_agent`, `...`
