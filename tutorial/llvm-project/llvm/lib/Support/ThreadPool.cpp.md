# ThreadPool.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `llvm/lib/Support/ThreadPool.cpp`
- Repository: `llvm-project`
- Purpose (EN): ==-- llvm/Support/ThreadPool.
- Purpose (CN): 该文件位于 LLVM 的 `Support` 目录中，主要实现与 `ThreadPool` 相关的接口、数据结构和辅助逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-40

```cpp
//==-- llvm/Support/ThreadPool.cpp - A ThreadPool implementation -*- C++ -*-==//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
//
// This file implements a crude C++11 based thread pool.
//
//===----------------------------------------------------------------------===//

#include "llvm/Support/ThreadPool.h"

#include "llvm/Config/llvm-config.h"

#include "llvm/ADT/ScopeExit.h"
#include "llvm/Support/ExponentialBackoff.h"
#include "llvm/Support/FormatVariadic.h"
#include "llvm/Support/Threading.h"
#include "llvm/Support/raw_ostream.h"

using namespace llvm;

ThreadPoolInterface::~ThreadPoolInterface() = default;

// A note on thread groups: Tasks are by default in no group (represented
// by nullptr ThreadPoolTaskGroup pointer in the Tasks queue) and functionality
// here normally works on all tasks regardless of their group (functions
// in that case receive nullptr ThreadPoolTaskGroup pointer as argument).
// A task in a group has a pointer to that ThreadPoolTaskGroup in the Tasks
// queue, and functions called to work only on tasks from one group take that
// pointer.

#if LLVM_ENABLE_THREADS

StdThreadPool::StdThreadPool(ThreadPoolStrategy S)
    : Strategy(S), MaxThreadCount(S.compute_thread_count()) {
  if (Strategy.UseJobserver)
```
- EN: Introduces the file banner, comments, and the initial setup for this translation unit.
  CN: 这一段给出文件横幅、注释说明以及该编译单元的初始设置。
- EN: Brings in 7 direct dependencies, including `llvm/Support/ThreadPool.h`, `llvm/Config/llvm-config.h`, `llvm/ADT/ScopeExit.h`, `llvm/Support/ExponentialBackoff.h`.
  CN: 引入了 7 个直接依赖，其中包括 `llvm/Support/ThreadPool.h`, `llvm/Config/llvm-config.h`, `llvm/ADT/ScopeExit.h`, `llvm/Support/ExponentialBackoff.h`。
- EN: This section centers on `StdThreadPool` and parses input and converts raw data into structured form.
  CN: 这一段主要围绕 `StdThreadPool` 等符号展开，负责解析输入并把原始数据转换成结构化形式。

### Lines 41-80

```cpp
    TheJobserver = JobserverClient::getInstance();
}

void StdThreadPool::grow(int requested) {
  llvm::sys::ScopedWriter LockGuard(ThreadsLock);
  if (Threads.size() >= MaxThreadCount)
    return; // Already hit the max thread pool size.
  int newThreadCount = std::min<int>(requested, MaxThreadCount);
  while (static_cast<int>(Threads.size()) < newThreadCount) {
    int ThreadID = Threads.size();
    Threads.emplace_back([this, ThreadID] {
      set_thread_name(formatv("llvm-worker-{0}", ThreadID));
      Strategy.apply_thread_strategy(ThreadID);
      // Note on jobserver deadlock avoidance:
      // GNU Make grants each invoked process one implicit job slot.
      // JobserverClient::tryAcquire() returns that implicit slot on the first
      // successful call in a process, ensuring forward progress without a
      // dedicated "always-on" thread.
      if (TheJobserver)
        processTasksWithJobserver();
      else
        processTasks(nullptr);
    });
  }
}

#ifndef NDEBUG
// The group of the tasks run by the current thread.
static LLVM_THREAD_LOCAL std::vector<ThreadPoolTaskGroup *>
    *CurrentThreadTaskGroups = nullptr;
#endif

// WaitingForGroup == nullptr means all tasks regardless of their group.
void StdThreadPool::processTasks(ThreadPoolTaskGroup *WaitingForGroup) {
  while (true) {
    llvm::unique_function<void()> Task;
    ThreadPoolTaskGroup *GroupOfTask;
    {
      std::unique_lock<std::mutex> LockGuard(QueueLock);
      bool workCompletedForGroup = false; // Result of workCompletedUnlocked()
```
- EN: This section centers on `grow`, `LockGuard`, `processTasks` and implements the local control flow and bookkeeping.
  CN: 这一段主要围绕 `grow`, `LockGuard`, `processTasks` 等符号展开，负责实现局部控制流程与状态维护。
- EN: In this range, the code iterates over collections, ranges, or records and checks conditions and handles edge cases.
  CN: 在这一段中，代码遍历集合、区间或记录，并检查条件并处理边界情况。

### Lines 81-120

```cpp
      // Wait for tasks to be pushed in the queue
      QueueCondition.wait(LockGuard, [&] {
        return !EnableFlag || !Tasks.empty() ||
               (WaitingForGroup != nullptr &&
                (workCompletedForGroup =
                     workCompletedUnlocked(WaitingForGroup)));
      });
      // Exit condition
      if (!EnableFlag && Tasks.empty())
        return;
      if (WaitingForGroup != nullptr && workCompletedForGroup)
        return;
      // Yeah, we have a task, grab it and release the lock on the queue

      // We first need to signal that we are active before popping the queue
      // in order for wait() to properly detect that even if the queue is
      // empty, there is still a task in flight.
      ++ActiveThreads;
      Task = std::move(Tasks.front().first);
      GroupOfTask = Tasks.front().second;
      // Need to count active threads in each group separately, ActiveThreads
      // would never be 0 if waiting for another group inside a wait.
      if (GroupOfTask != nullptr)
        ++ActiveGroups[GroupOfTask]; // Increment or set to 1 if new item
      Tasks.pop_front();
    }
#ifndef NDEBUG
    if (CurrentThreadTaskGroups == nullptr)
      CurrentThreadTaskGroups = new std::vector<ThreadPoolTaskGroup *>;
    CurrentThreadTaskGroups->push_back(GroupOfTask);
#endif

    // Run the task we just grabbed. This also destroys the task once run to
    // release any resources held by it through RAII captured objects.
    //
    // It is particularly important to do this here so that we're not holding
    // any lock and any further operations on the thread or `ThreadPool` take
    // place here, at the same point as the task itself is executed.
    std::exchange(Task, {})();

```
- EN: This section centers on `workCompletedUnlocked` and implements the local control flow and bookkeeping.
  CN: 这一段主要围绕 `workCompletedUnlocked` 等符号展开，负责实现局部控制流程与状态维护。
- EN: In this range, the code checks conditions and handles edge cases and updates helper containers and temporary state.
  CN: 在这一段中，代码检查条件并处理边界情况，并更新辅助容器和临时状态。

### Lines 121-160

```cpp
#ifndef NDEBUG
    CurrentThreadTaskGroups->pop_back();
    if (CurrentThreadTaskGroups->empty()) {
      delete CurrentThreadTaskGroups;
      CurrentThreadTaskGroups = nullptr;
    }
#endif

    bool Notify;
    bool NotifyGroup;
    {
      // Adjust `ActiveThreads`, in case someone waits on StdThreadPool::wait()
      std::lock_guard<std::mutex> LockGuard(QueueLock);
      --ActiveThreads;
      if (GroupOfTask != nullptr) {
        auto A = ActiveGroups.find(GroupOfTask);
        if (--(A->second) == 0)
          ActiveGroups.erase(A);
      }
      Notify = workCompletedUnlocked(GroupOfTask);
      NotifyGroup = GroupOfTask != nullptr && Notify;
    }
    // Notify task completion if this is the last active thread, in case
    // someone waits on StdThreadPool::wait().
    if (Notify)
      CompletionCondition.notify_all();
    // If this was a task in a group, notify also threads waiting for tasks
    // in this function on QueueCondition, to make a recursive wait() return
    // after the group it's been waiting for has finished.
    if (NotifyGroup)
      QueueCondition.notify_all();
  }
}

/// Main loop for worker threads when using a jobserver.
/// This function uses a two-level queue; it first acquires a job slot from the
/// external jobserver, then retrieves a task from the internal queue.
/// This allows the thread pool to cooperate with build systems like `make -j`.
void StdThreadPool::processTasksWithJobserver() {
  while (true) {
```
- EN: This section centers on `LockGuard`, `processTasksWithJobserver` and implements the local control flow and bookkeeping.
  CN: 这一段主要围绕 `LockGuard`, `processTasksWithJobserver` 等符号展开，负责实现局部控制流程与状态维护。
- EN: In this range, the code iterates over collections, ranges, or records and checks conditions and handles edge cases.
  CN: 在这一段中，代码遍历集合、区间或记录，并检查条件并处理边界情况。

### Lines 161-200

```cpp
    // Acquire a job slot from the external jobserver.
    // This polls for a slot and yields the thread to avoid a high-CPU wait.
    JobSlot Slot;
    // The timeout for the backoff can be very long, as the shutdown
    // is checked on each iteration. The sleep duration is capped by MaxWait
    // in ExponentialBackoff, so shutdown latency is not a problem.
    ExponentialBackoff Backoff(std::chrono::hours(24));
    bool AcquiredToken = false;
    do {
      // Return if the thread pool is shutting down.
      {
        std::unique_lock<std::mutex> LockGuard(QueueLock);
        if (!EnableFlag)
          return;
      }

      Slot = TheJobserver->tryAcquire();
      if (Slot.isValid()) {
        AcquiredToken = true;
        break;
      }
    } while (Backoff.waitForNextAttempt());

    if (!AcquiredToken) {
      // This is practically unreachable with a 24h timeout and indicates a
      // deeper problem if hit.
      report_fatal_error("Timed out waiting for jobserver token.");
    }

    // `llvm::scope_exit` guarantees the job slot is released, even if the
    // task throws or we exit early. This prevents deadlocking the build.
    llvm::scope_exit SlotReleaser(
        [&] { TheJobserver->release(std::move(Slot)); });

    // While we hold a job slot, process tasks from the internal queue.
    while (true) {
      llvm::unique_function<void()> Task;
      ThreadPoolTaskGroup *GroupOfTask = nullptr;

      {
```
- EN: This section centers on `Backoff`, `LockGuard`, `report_fatal_error` and parses input and converts raw data into structured form.
  CN: 这一段主要围绕 `Backoff`, `LockGuard`, `report_fatal_error` 等符号展开，负责解析输入并把原始数据转换成结构化形式。
- EN: In this range, the code iterates over collections, ranges, or records and checks conditions and handles edge cases.
  CN: 在这一段中，代码遍历集合、区间或记录，并检查条件并处理边界情况。

### Lines 201-240

```cpp
        std::unique_lock<std::mutex> LockGuard(QueueLock);

        // Wait until a task is available or the pool is shutting down.
        QueueCondition.wait(LockGuard,
                            [&] { return !EnableFlag || !Tasks.empty(); });

        // If shutting down and the queue is empty, the thread can terminate.
        if (!EnableFlag && Tasks.empty())
          return;

        // If the queue is empty, we're done processing tasks for now.
        // Break the inner loop to release the job slot.
        if (Tasks.empty())
          break;

        // A task is available. Mark it as active before releasing the lock
        // to prevent race conditions with `wait()`.
        ++ActiveThreads;
        Task = std::move(Tasks.front().first);
        GroupOfTask = Tasks.front().second;
        if (GroupOfTask != nullptr)
          ++ActiveGroups[GroupOfTask];
        Tasks.pop_front();
      } // The queue lock is released.

      // Run the task. The job slot remains acquired during execution.
      Task();

      // The task has finished. Update the active count and notify any waiters.
      {
        std::lock_guard<std::mutex> LockGuard(QueueLock);
        --ActiveThreads;
        if (GroupOfTask != nullptr) {
          auto A = ActiveGroups.find(GroupOfTask);
          if (--(A->second) == 0)
            ActiveGroups.erase(A);
        }
        // If all tasks are complete, notify any waiting threads.
        if (workCompletedUnlocked(nullptr))
          CompletionCondition.notify_all();
```
- EN: This section centers on `LockGuard`, `Task` and implements the local control flow and bookkeeping.
  CN: 这一段主要围绕 `LockGuard`, `Task` 等符号展开，负责实现局部控制流程与状态维护。
- EN: In this range, the code checks conditions and handles edge cases and returns the resulting value to its callers.
  CN: 在这一段中，代码检查条件并处理边界情况，并将结果返回给调用方。

### Lines 241-280

```cpp
      }
    }
  }
}
bool StdThreadPool::workCompletedUnlocked(ThreadPoolTaskGroup *Group) const {
  if (Group == nullptr)
    return !ActiveThreads && Tasks.empty();
  return ActiveGroups.count(Group) == 0 &&
         !llvm::is_contained(llvm::make_second_range(Tasks), Group);
}

void StdThreadPool::wait() {
  assert(!isWorkerThread()); // Would deadlock waiting for itself.
  // Wait for all threads to complete and the queue to be empty
  std::unique_lock<std::mutex> LockGuard(QueueLock);
  CompletionCondition.wait(LockGuard,
                           [&] { return workCompletedUnlocked(nullptr); });
}

void StdThreadPool::wait(ThreadPoolTaskGroup &Group) {
  // Wait for all threads in the group to complete.
  if (!isWorkerThread()) {
    std::unique_lock<std::mutex> LockGuard(QueueLock);
    CompletionCondition.wait(LockGuard,
                             [&] { return workCompletedUnlocked(&Group); });
    return;
  }
  // Make sure to not deadlock waiting for oneself.
  assert(CurrentThreadTaskGroups == nullptr ||
         !llvm::is_contained(*CurrentThreadTaskGroups, &Group));
  // Handle the case of recursive call from another task in a different group,
  // in which case process tasks while waiting to keep the thread busy and avoid
  // possible deadlock.
  processTasks(&Group);
}

bool StdThreadPool::isWorkerThread() const {
  llvm::sys::ScopedReader LockGuard(ThreadsLock);
  llvm::thread::id CurrentThreadId = llvm::this_thread::get_id();
  for (const llvm::thread &Thread : Threads)
```
- EN: This section centers on `workCompletedUnlocked`, `wait`, `assert` and parses input and converts raw data into structured form.
  CN: 这一段主要围绕 `workCompletedUnlocked`, `wait`, `assert` 等符号展开，负责解析输入并把原始数据转换成结构化形式。
- EN: In this range, the code iterates over collections, ranges, or records and checks conditions and handles edge cases.
  CN: 在这一段中，代码遍历集合、区间或记录，并检查条件并处理边界情况。

### Lines 281-320

```cpp
    if (CurrentThreadId == Thread.get_id())
      return true;
  return false;
}

// The destructor joins all threads, waiting for completion.
StdThreadPool::~StdThreadPool() {
  {
    std::unique_lock<std::mutex> LockGuard(QueueLock);
    EnableFlag = false;
  }
  QueueCondition.notify_all();
  llvm::sys::ScopedReader LockGuard(ThreadsLock);
  for (auto &Worker : Threads)
    Worker.join();
}

#endif // LLVM_ENABLE_THREADS Disabled

// No threads are launched, issue a warning if ThreadCount is not 0
SingleThreadExecutor::SingleThreadExecutor(ThreadPoolStrategy S) {
  int ThreadCount = S.compute_thread_count();
  if (ThreadCount != 1) {
    errs() << "Warning: request a ThreadPool with " << ThreadCount
           << " threads, but LLVM_ENABLE_THREADS has been turned off\n";
  }
}

void SingleThreadExecutor::wait() {
  // Sequential implementation running the tasks
  while (!Tasks.empty()) {
    auto Task = std::move(Tasks.front().first);
    Tasks.pop_front();
    Task();
  }
}

void SingleThreadExecutor::wait(ThreadPoolTaskGroup &) {
  // Simply wait for all, this works even if recursive (the running task
  // is already removed from the queue).
```
- EN: This section centers on `LockGuard`, `SingleThreadExecutor`, `wait` and parses input and converts raw data into structured form.
  CN: 这一段主要围绕 `LockGuard`, `SingleThreadExecutor`, `wait` 等符号展开，负责解析输入并把原始数据转换成结构化形式。
- EN: In this range, the code iterates over collections, ranges, or records and checks conditions and handles edge cases.
  CN: 在这一段中，代码遍历集合、区间或记录，并检查条件并处理边界情况。

### Lines 321-328

```cpp
  wait();
}

bool SingleThreadExecutor::isWorkerThread() const {
  report_fatal_error("LLVM compiled without multithreading");
}

SingleThreadExecutor::~SingleThreadExecutor() { wait(); }
```
- EN: This section centers on `wait`, `isWorkerThread`, `report_fatal_error` and parses input and converts raw data into structured form.
  CN: 这一段主要围绕 `wait`, `isWorkerThread`, `report_fatal_error` 等符号展开，负责解析输入并把原始数据转换成结构化形式。

## Key Concepts / 关键概念
- Domain / 领域: LLVM support utilities / LLVM 支撑工具
- Core symbols / 核心符号: `StdThreadPool`, `grow`, `LockGuard`, `processTasks` / 该文件围绕这些类型或函数组织主要逻辑。
- Data flow / 数据流: Helper routines coordinate local state, control flow, and result construction. / 通过辅助例程协调局部状态、控制流与结果构造。
- Error model / 错误模型: LLVM-style `Error`/`Expected` handling and invariant checks. / 使用 LLVM 风格的 `Error`/`Expected` 处理与不变量检查。
- Data structures / 数据结构: Relies on LLVM or STL containers for compact state management. / 依赖 LLVM 或 STL 容器管理紧凑状态。

## Dependencies / 依赖关系
- LLVM headers / LLVM 头文件: `llvm/Support/ThreadPool.h`, `llvm/Config/llvm-config.h`, `llvm/ADT/ScopeExit.h`, `llvm/Support/ExponentialBackoff.h`, `llvm/Support/FormatVariadic.h`, `llvm/Support/Threading.h`, `llvm/Support/raw_ostream.h`
- Standard library / 标准库: None / 无
- Other/system headers / 其他或系统头文件: None / 无
- Related symbols / 相关符号: `StdThreadPool`, `grow`, `LockGuard`, `processTasks`, `workCompletedUnlocked`
