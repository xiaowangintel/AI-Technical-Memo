# ParallelGraphExecutor.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `torch/nativert/executor/ParallelGraphExecutor.cpp`
- Repository: `pytorch/pytorch`
- Purpose (EN): Implements Native Runtime executor behavior for ParallelGraphExecutor, including execution, delegation, or low-level utilities.
- 用途 (CN): 实现 Native Runtime 中 executor 子模块里与 ParallelGraphExecutor 相关的行为，包括执行、delegate 协调或底层工具逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-12
```cpp
#include <moodycamel/concurrentqueue.h>
#include <torch/nativert/executor/ExecutorConfig.h>
#include <torch/nativert/executor/ParallelGraphExecutor.h>

namespace {

#define WITH_LOCK(m, block)               \
  {                                       \
    std::unique_lock<decltype(m)> lk_(m); \
    block                                 \
  }

```
- EN: This block manipulates graph-like program structures; coordinates runtime execution state; protects shared state or ordering guarantees. Key symbols: `lk_`.
- CN: 该代码块操作图状程序结构；协调运行时执行状态；保护共享状态或执行顺序保证。关键符号：`lk_`。

### Lines 13-19
```cpp
} // namespace

namespace torch::nativert {

ThreadPoolExecutor::ThreadPoolExecutor()
    : work_(std::make_unique<moodycamel::ConcurrentQueue<Work>>()) {}

```
- EN: This block coordinates runtime execution state; protects shared state or ordering guarantees. Key symbols: `ThreadPoolExecutor`, `work_`.
- CN: 该代码块协调运行时执行状态；保护共享状态或执行顺序保证。关键符号：`ThreadPoolExecutor`, `work_`。

### Lines 20-28
```cpp
ThreadPoolExecutor::~ThreadPoolExecutor() {
  stop();
}

C10_ALWAYS_INLINE moodycamel::ProducerToken& ThreadPoolExecutor::ptok() {
  // NOLINTNEXTLINE(misc-use-internal-linkage)
  thread_local moodycamel::ProducerToken* pTokPtr = nullptr;
  thread_local moodycamel::ConcurrentQueue<Work>* associatedQueue = nullptr;

```
- EN: This block coordinates runtime execution state; protects shared state or ordering guarantees. Key symbols: `~ThreadPoolExecutor`, `stop`, `ptok`.
- CN: 该代码块协调运行时执行状态；保护共享状态或执行顺序保证。关键符号：`~ThreadPoolExecutor`, `stop`, `ptok`。

### Lines 29-35
```cpp
  // Check if token exists AND is for the current queue
  if (pTokPtr == nullptr || associatedQueue != work_.get()) {
    delete pTokPtr;
    pTokPtr = new moodycamel::ProducerToken(*work_);
    associatedQueue = work_.get();
  }
  return *pTokPtr;
```
- EN: This block checks invariants or expected outcomes; handles conditional control flow; iterates over collections or execution units; returns results to callers or downstream stages; protects shared state or ordering guarantees. Key symbols: `get`, `ProducerToken`.
- CN: 该代码块检查不变量或预期结果；处理条件控制流；遍历集合或执行单元；向调用方或后续阶段返回结果；保护共享状态或执行顺序保证。关键符号：`get`, `ProducerToken`。

### Lines 36-43
```cpp
}

C10_ALWAYS_INLINE moodycamel::ConsumerToken& ThreadPoolExecutor::ctok() {
  // NOLINTNEXTLINE(misc-use-internal-linkage)
  thread_local moodycamel::ConsumerToken ctok(*work_);
  return ctok;
}

```
- EN: This block coordinates runtime execution state; returns results to callers or downstream stages; protects shared state or ordering guarantees. Key symbols: `ctok`.
- CN: 该代码块协调运行时执行状态；向调用方或后续阶段返回结果；保护共享状态或执行顺序保证。关键符号：`ctok`。

### Lines 44-51
```cpp
void ThreadPoolExecutor::execute_inline(SessionState* session, WorkUnit* unit) {
  session->addWork();
  unit->run(this, session);
}

void ThreadPoolExecutor::start(int32_t numThreads) {
  stopped_ = false;
  for (int32_t i = 0; i < numThreads; ++i) {
```
- EN: This block coordinates runtime execution state; iterates over collections or execution units; protects shared state or ordering guarantees. Key symbols: `execute_inline`, `addWork`, `run`, `start`.
- CN: 该代码块协调运行时执行状态；遍历集合或执行单元；保护共享状态或执行顺序保证。关键符号：`execute_inline`, `addWork`, `run`, `start`。

### Lines 52-58
```cpp
    threads_.emplace_back(&ThreadPoolExecutor::loop, this);
  }
}

void ThreadPoolExecutor::loop() {
  // Track profiler state for this thread to synchronize with main thread
  bool profilerEnabledInThisThread = false;
```
- EN: This block coordinates runtime execution state; iterates over collections or execution units; protects shared state or ordering guarantees. Key symbols: `emplace_back`, `loop`.
- CN: 该代码块协调运行时执行状态；遍历集合或执行单元；保护共享状态或执行顺序保证。关键符号：`emplace_back`, `loop`。

### Lines 59-65
```cpp

  while (true) {
    Work unit;

    sem_->acquire();

    if (stopped_) {
```
- EN: This block handles conditional control flow; iterates over collections or execution units. Key symbols: `acquire`.
- CN: 该代码块处理条件控制流；遍历集合或执行单元。关键符号：`acquire`。

### Lines 66-72
```cpp
      // Clean up profiler state before thread exits
      if (profilerEnabledInThisThread) {
        torch::autograd::profiler::disableProfilerInChildThread();
      }
      return;
    }

```
- EN: This block handles conditional control flow; protects shared state or ordering guarantees. Key symbols: `disableProfilerInChildThread`.
- CN: 该代码块处理条件控制流；保护共享状态或执行顺序保证。关键符号：`disableProfilerInChildThread`。

### Lines 73-82
```cpp
    // Synchronize profiler state with main thread
    syncProfilerStateFromMainThread(profilerEnabledInThisThread);

    while (!work_->try_dequeue(ctok(), unit)) {
    };

    unit();
  }
}

```
- EN: This block iterates over collections or execution units; protects shared state or ordering guarantees. Key symbols: `syncProfilerStateFromMainThread`, `try_dequeue`, `ctok`, `unit`.
- CN: 该代码块遍历集合或执行单元；保护共享状态或执行顺序保证。关键符号：`syncProfilerStateFromMainThread`, `try_dequeue`, `ctok`, `unit`。

### Lines 83-89
```cpp
void ThreadPoolExecutor::add(SessionState* session, WorkUnit* unit) {
  session->addWork();
  work_->enqueue(ptok(), [unit, this, session] { unit->run(this, session); });
  sem_->release();
}

void ThreadPoolExecutor::add(
```
- EN: This block coordinates runtime execution state; protects shared state or ordering guarantees. Key symbols: `add`, `addWork`, `enqueue`, `ptok`, `run`, `release`.
- CN: 该代码块协调运行时执行状态；保护共享状态或执行顺序保证。关键符号：`add`, `addWork`, `enqueue`, `ptok`, `run`, `release`。

### Lines 90-101
```cpp
    SessionState* session,
    std::vector<WorkUnit*>::const_iterator begin,
    const std::vector<WorkUnit*>::const_iterator& end) {
  const auto count = end - begin;

  // NOLINTNEXTLINE(bugprone-switch-missing-default-case)
  switch (count) {
    case 0: {
      return;
    }
    case 1: {
      return add(session, *begin);
```
- EN: This block returns results to callers or downstream stages. Key symbols: `add`.
- CN: 该代码块向调用方或后续阶段返回结果。关键符号：`add`。

### Lines 102-109
```cpp
    }
  }

  session->addWork(count);

  std::vector<Work> runnables;
  runnables.reserve(count);
  for (; begin != end; ++begin) {
```
- EN: This block iterates over collections or execution units. Key symbols: `addWork`, `reserve`.
- CN: 该代码块遍历集合或执行单元。关键符号：`addWork`, `reserve`。

### Lines 110-117
```cpp
    runnables.emplace_back(
        [capture0 = *begin, this, session] { capture0->run(this, session); });
  }

  work_->enqueue_bulk(ptok(), runnables.begin(), count);
  sem_->release(static_cast<int32_t>(count));
}

```
- EN: This block protects shared state or ordering guarantees. Key symbols: `emplace_back`, `run`, `enqueue_bulk`, `ptok`, `begin`, `release`.
- CN: 该代码块保护共享状态或执行顺序保证。关键符号：`emplace_back`, `run`, `enqueue_bulk`, `ptok`, `begin`, `release`。

### Lines 118-124
```cpp
void ThreadPoolExecutor::stop() {
  stopped_ = true;
  sem_->release(static_cast<int32_t>(threads_.size()));

  std::for_each(threads_.begin(), threads_.end(), [](auto& t) { t.join(); });
  threads_.clear();

```
- EN: This block coordinates runtime execution state; protects shared state or ordering guarantees. Key symbols: `stop`, `release`, `size`, `for_each`, `begin`, `end`, `...`.
- CN: 该代码块协调运行时执行状态；保护共享状态或执行顺序保证。关键符号：`stop`, `release`, `size`, `for_each`, `begin`, `end`, `...`。

### Lines 125-133
```cpp
  {
    // reset sem
    auto tmp = std::make_unique<c10::Semaphore>();
    sem_.swap(tmp);
  }

  {
    // flush queue
    auto tmp = moodycamel::ConcurrentQueue<Work>();
```
- EN: This block protects shared state or ordering guarantees. Key symbols: `swap`.
- CN: 该代码块保护共享状态或执行顺序保证。关键符号：`swap`。

### Lines 134-141
```cpp
    work_->swap(tmp);
  }
}

size_t ThreadPoolExecutor::queueSizeApprox() const {
  return work_->size_approx();
}

```
- EN: This block coordinates runtime execution state; returns results to callers or downstream stages; protects shared state or ordering guarantees. Key symbols: `swap`, `queueSizeApprox`, `size_approx`.
- CN: 该代码块协调运行时执行状态；向调用方或后续阶段返回结果；保护共享状态或执行顺序保证。关键符号：`swap`, `queueSizeApprox`, `size_approx`。

### Lines 142-150
```cpp
void ThreadPoolExecutor::run(
    SessionState& session,
    const std::vector<WorkUnit*>& roots) {
  // case where thread ptok exists but work_ was swapped
  if (auto& tok = ptok(); C10_UNLIKELY(!tok.valid())) {
    moodycamel::ProducerToken tmp(*work_);
    tok.swap(tmp);
  }

```
- EN: This block coordinates runtime execution state; handles conditional control flow; protects shared state or ordering guarantees. Key symbols: `run`, `ptok`, `valid`, `tmp`, `swap`.
- CN: 该代码块协调运行时执行状态；处理条件控制流；保护共享状态或执行顺序保证。关键符号：`run`, `ptok`, `valid`, `tmp`, `swap`。

### Lines 151-158
```cpp
  const auto rootCount = roots.size();

  if (C10_UNLIKELY(rootCount == 0)) {
    return;
  } else if (C10_LIKELY(rootCount > 1)) {
    add(&session, roots.begin() + 1, roots.end());
  }

```
- EN: This block handles conditional control flow. Key symbols: `size`, `add`, `begin`, `end`.
- CN: 该代码块处理条件控制流。关键符号：`size`, `add`, `begin`, `end`。

### Lines 159-167
```cpp
  execute_inline(&session, roots[0]);

  session.wait();
}

void WorkUnit::run(ThreadPoolExecutor* executor, SessionState* session) {
  /* thread_local */ std::vector<WorkUnit*> newWorkUnits;
  /* thread_local */ c10::InferenceMode mode;

```
- EN: This block coordinates runtime execution state; protects shared state or ordering guarantees. Key symbols: `execute_inline`, `wait`, `run`.
- CN: 该代码块协调运行时执行状态；保护共享状态或执行顺序保证。关键符号：`execute_inline`, `wait`, `run`。

### Lines 168-174
```cpp
  /* thread_local */ WorkUnit* unit = this;

  while (true) {
    unit->kernel->compute(session->frame());

    for (auto* user : unit->users) {
      if (session->decrementProducers(user->node)) {
```
- EN: This block manipulates graph-like program structures; coordinates runtime execution state; handles conditional control flow; iterates over collections or execution units; protects shared state or ordering guarantees. Key symbols: `compute`, `frame`, `decrementProducers`.
- CN: 该代码块操作图状程序结构；协调运行时执行状态；处理条件控制流；遍历集合或执行单元；保护共享状态或执行顺序保证。关键符号：`compute`, `frame`, `decrementProducers`。

### Lines 175-181
```cpp
        newWorkUnits.push_back(user);
      }
    }

    switch (newWorkUnits.size()) {
      case 0: {
        return session->removeWork();
```
- EN: This block returns results to callers or downstream stages. Key symbols: `push_back`, `size`, `removeWork`.
- CN: 该代码块向调用方或后续阶段返回结果。关键符号：`push_back`, `size`, `removeWork`。

### Lines 182-195
```cpp
      }
      case 1: {
        break;
      }
      case 2: {
        executor->add(session, newWorkUnits[1]);
        break;
      }
      default: {
        executor->add(session, newWorkUnits.begin() + 1, newWorkUnits.end());
        break;
      }
    }

```
- EN: This block coordinates runtime execution state. Key symbols: `add`, `begin`, `end`.
- CN: 该代码块协调运行时执行状态。关键符号：`add`, `begin`, `end`。

### Lines 196-209
```cpp
    unit = newWorkUnits[0];
    newWorkUnits.clear();
  }
}

ParallelGraphExecutor::ParallelGraphExecutor(
    const Graph& graph,
    std::vector<std::unique_ptr<OpKernel>> nodeKernels,
    const ExecutorConfig& executorConfig)
    : GraphExecutorBase(graph, std::move(nodeKernels), executorConfig),
      workUnits_(
          graph.nodes().size() - 2 /* no need for prim.Input or Prim.Output */),
      graph_(graph) {
  auto& nodes = graph_.nodes();
```
- EN: This block manipulates graph-like program structures; coordinates runtime execution state; iterates over collections or execution units. Key symbols: `clear`, `ParallelGraphExecutor`, `GraphExecutorBase`, `move`, `workUnits_`, `nodes`, `...`.
- CN: 该代码块操作图状程序结构；协调运行时执行状态；遍历集合或执行单元。关键符号：`clear`, `ParallelGraphExecutor`, `GraphExecutorBase`, `move`, `workUnits_`, `nodes`, `...`。

### Lines 210-220
```cpp

  auto input = &*nodes.begin();
  auto output = &*nodes.rbegin();

  {
    // get rid of prim.Input and prim.Output kernels
    // since we won't be needing them
    nodeKernels_.erase(nodeKernels_.begin());
    nodeKernels_.pop_back();
  }

```
- EN: This block manipulates graph-like program structures. Key symbols: `begin`, `rbegin`, `erase`, `pop_back`.
- CN: 该代码块操作图状程序结构。关键符号：`begin`, `rbegin`, `erase`, `pop_back`。

### Lines 221-232
```cpp
  size_t idx = 0;
  for (const auto& node : nodes) {
    if (&node == input || &node == output) {
      continue;
    }
    auto& workUnit =
        nodeToWorkUnit_.insert_or_assign(&node, &workUnits_[idx]).first->second;
    workUnit->node = &node;
    workUnit->kernel = nodeKernels_[idx++].get();
    producers_.insert({&node, 0});
  }

```
- EN: This block manipulates graph-like program structures; handles conditional control flow; iterates over collections or execution units. Key symbols: `insert_or_assign`, `get`, `insert`.
- CN: 该代码块操作图状程序结构；处理条件控制流；遍历集合或执行单元。关键符号：`insert_or_assign`, `get`, `insert`。

### Lines 233-241
```cpp
  for (auto& unit : workUnits_) {
    for (const auto* dep : unit.node->users()) {
      if (dep != output) {
        unit.users.push_back(nodeToWorkUnit_[dep]);
        producers_[dep] += 1;
      }
    }
  }

```
- EN: This block manipulates graph-like program structures; handles conditional control flow; iterates over collections or execution units. Key symbols: `users`, `push_back`.
- CN: 该代码块操作图状程序结构；处理条件控制流；遍历集合或执行单元。关键符号：`users`, `push_back`。

### Lines 242-250
```cpp
  for (auto& [node, p] : producers_) {
    if (p == 0) {
      inputWorkUnits_.push_back(nodeToWorkUnit_[node]);
    }
  }

  executor_.start(static_cast<int32_t>(executorConfig.maxParallelOps));
}

```
- EN: This block manipulates graph-like program structures; coordinates runtime execution state; handles conditional control flow; iterates over collections or execution units. Key symbols: `push_back`, `start`.
- CN: 该代码块操作图状程序结构；协调运行时执行状态；处理条件控制流；遍历集合或执行单元。关键符号：`push_back`, `start`。

### Lines 251-257
```cpp
std::vector<c10::IValue> ParallelGraphExecutor::execute(
    ExecutionFrame& executionFrame,
    std::vector<c10::IValue> inputs) {
  fillUserInputs(executionFrame, std::move(inputs));
  return executeWithPrefilledFrame(executionFrame);
}

```
- EN: This block manipulates graph-like program structures; coordinates runtime execution state; returns results to callers or downstream stages. Key symbols: `execute`, `fillUserInputs`, `move`, `executeWithPrefilledFrame`.
- CN: 该代码块操作图状程序结构；协调运行时执行状态；向调用方或后续阶段返回结果。关键符号：`execute`, `fillUserInputs`, `move`, `executeWithPrefilledFrame`。

### Lines 258-265
```cpp
std::vector<c10::IValue> ParallelGraphExecutor::executeWithPrefilledFrame(
    ExecutionFrame& executionFrame) {
  auto session = SessionState(executionFrame, producers_);
  executor_.run(session, inputWorkUnits_);

  return executionFrame.tryMoveUserOutputs();
}

```
- EN: This block manipulates graph-like program structures; coordinates runtime execution state; returns results to callers or downstream stages. Key symbols: `executeWithPrefilledFrame`, `SessionState`, `run`, `tryMoveUserOutputs`.
- CN: 该代码块操作图状程序结构；协调运行时执行状态；向调用方或后续阶段返回结果。关键符号：`executeWithPrefilledFrame`, `SessionState`, `run`, `tryMoveUserOutputs`。

### Lines 266-266
```cpp
} // namespace torch::nativert
```
- EN: This block implements local helper logic for ParallelGraphExecutor. Key symbols: no dominant local symbols.
- CN: 该代码块实现与 ParallelGraphExecutor 相关的局部辅助逻辑。关键符号：无明显局部符号。


## Key Concepts / 关键概念
- EN: Graph capture and transformation — the implementation works with program graphs and rewrite passes.
  CN: 图捕获与变换——实现会处理程序图及其改写 pass。
- EN: Runtime execution — the file schedules or executes native runtime programs and delegates.
  CN: 运行时执行——该文件会调度或执行原生运行时程序及其 delegate。
- EN: Concurrency support — the implementation manages coordination, ownership, or thread safety.
  CN: 并发支持——实现会管理协作、所有权或线程安全。
- EN: Runtime composition — the file participates in assembling native runtime data flow, execution state, or backend handoff.
  CN: 运行时组合——该文件参与组织原生运行时的数据流、执行状态或后端交接。
- EN: Header-level integration — the translation unit depends on nearby runtime and framework declarations.
  CN: 头文件级集成——该翻译单元依赖周边运行时与框架声明。

## Dependencies / 依赖关系
- Internal includes / 内部头文件: `torch/nativert/executor/ExecutorConfig.h`, `torch/nativert/executor/ParallelGraphExecutor.h`
- External includes / 外部头文件: `moodycamel/concurrentqueue.h`
- Namespaces / 命名空间: `torch::nativert`
- Representative symbols / 代表性符号: `lk_`, `ThreadPoolExecutor`, `work_`, `~ThreadPoolExecutor`, `stop`, `ptok`, `get`, `ProducerToken`, `ctok`, `execute_inline`, `...`
