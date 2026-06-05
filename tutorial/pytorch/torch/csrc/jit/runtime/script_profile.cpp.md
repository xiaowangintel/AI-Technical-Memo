# script_profile.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `torch/csrc/jit/runtime/script_profile.cpp`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Parses or loads source-based TorchScript definitions and resolves symbols during import.
- **Purpose (CN)**: 在导入过程中解析或加载基于源码的 TorchScript 定义，并解析符号。
## Line-by-Line Analysis / 逐行分析

### Lines 1-11
```cpp
#include <torch/csrc/jit/runtime/script_profile.h>

#include <atomic>
#include <chrono>
#include <mutex>
#include <unordered_set>

#include <c10/util/Exception.h>
#include <c10/util/intrusive_ptr.h>

namespace torch::jit {
```
- **EN**: This block assembles the compilation dependencies, pulling in local JIT headers such as torch/csrc/jit/runtime/script_profile.h; ATen/c10 facilities such as c10/util/Exception.h, c10/util/intrusive_ptr.h; standard-library headers such as atomic, chrono, mutex, and 1 more. The namespace declarations place the code inside torch::jit, matching the surrounding JIT subsystem.
- **CN**: 这一段组织编译依赖，引入了本地 JIT 头文件，如 torch/csrc/jit/runtime/script_profile.h；ATen/c10 基础设施，如 c10/util/Exception.h、c10/util/intrusive_ptr.h；标准库头文件，如 atomic、chrono、mutex 等共 4 项。 命名空间声明把代码放入 torch::jit 中，与周边 JIT 子系统保持一致。

### Lines 13-19
```cpp
namespace {

class ProfilesRegistry {
 public:
  bool empty() {
    return empty_.load(std::memory_order_relaxed);
  }
```
- **EN**: It introduces or extends ProfilesRegistry, which define the primary data structures or interfaces for this portion of the file. This chunk defines `empty`, which implements a focused step in the TorchScript execution pipeline. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 它引入或扩展了 ProfilesRegistry，这些类型定义了本段涉及的主要数据结构或接口。 这一段定义了 `empty`，其作用是实现 TorchScript 执行流水线中的一个关键步骤。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 21-32
```cpp
  void addProfile(ScriptProfile& p) {
    std::lock_guard<std::mutex> g(mutex_);
    enabledProfiles_.emplace(&p);
    empty_.store(false, std::memory_order_relaxed);
  }

  void removeProfile(ScriptProfile& p) {
    std::lock_guard<std::mutex> g(mutex_);
    enabledProfiles_.erase(&p);
    if (enabledProfiles_.empty()) {
      empty_.store(true, std::memory_order_relaxed);
    }
```
- **EN**: This chunk defines `removeProfile`, which implements a focused step in the TorchScript execution pipeline. Conditional branches guard special cases and preserve type, shape, or serialization invariants.
- **CN**: 这一段定义了 `removeProfile`，其作用是实现 TorchScript 执行流水线中的一个关键步骤。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。

### Lines 33-41
```cpp
  }

  void send(std::unique_ptr<profiling::Datapoint> datapoint) {
    auto shared = std::shared_ptr<profiling::Datapoint>(std::move(datapoint));
    std::lock_guard<std::mutex> g(mutex_);
    for (auto* p : enabledProfiles_) {
      p->addDatapoint(shared);
    }
  }
```
- **EN**: This chunk defines `g`, which implements a focused step in the TorchScript execution pipeline. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically.
- **CN**: 这一段定义了 `g`，其作用是实现 TorchScript 执行流水线中的一个关键步骤。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。

### Lines 43-52
```cpp
 private:
  std::atomic<bool> empty_{true};
  std::mutex mutex_;
  std::unordered_set<ScriptProfile*> enabledProfiles_;
};

ProfilesRegistry& getProfilesRegistry() {
  static auto registry = std::ref(*new ProfilesRegistry{});
  return registry;
}
```
- **EN**: This chunk defines `getProfilesRegistry`, which implements a focused step in the TorchScript execution pipeline. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `getProfilesRegistry`，其作用是实现 TorchScript 执行流水线中的一个关键步骤。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 54-63
```cpp
auto initBindings() {
  torch::class_<SourceRef>("profiling", "SourceRef")
      .def(
          "starting_lineno",
          [](const c10::intrusive_ptr<SourceRef>& self) {
            return static_cast<int64_t>((*self)->starting_line_no());
          })
      .def("text", [](const c10::intrusive_ptr<SourceRef>& self) {
        return (*self)->text_str().str();
      });
```
- **EN**: This chunk defines `initBindings`, which implements a focused step in the TorchScript execution pipeline. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `initBindings`，其作用是实现 TorchScript 执行流水线中的一个关键步骤。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 65-73
```cpp
  torch::class_<InstructionStats>("profiling", "InstructionStats")
      .def(
          "count",
          [](const c10::intrusive_ptr<InstructionStats>& self) {
            return self->count;
          })
      .def("duration_ns", [](const c10::intrusive_ptr<InstructionStats>& self) {
        return static_cast<int64_t>(self->duration.count());
      });
```
- **EN**: This chunk continues `initBindings` and expands its internal control flow or data movement. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段延续了 `initBindings`，进一步展开其内部控制流或数据流转。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 75-81
```cpp
  torch::class_<SourceStats>("profiling", "SourceStats")
      .def(
          "source",
          [](const c10::intrusive_ptr<SourceStats>& self) {
            return c10::make_intrusive<SourceRef>(self->getSourceRef());
          })
      .def("line_map", &SourceStats::getLineMap);
```
- **EN**: This chunk continues `initBindings` and expands its internal control flow or data movement. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段延续了 `initBindings`，进一步展开其内部控制流或数据流转。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 83-94
```cpp
  torch::class_<ScriptProfile>("profiling", "_ScriptProfile")
      .def(torch::init<>())
      .def("enable", &ScriptProfile::enable)
      .def("disable", &ScriptProfile::disable)
      .def("_dump_stats", [](const c10::intrusive_ptr<ScriptProfile>& self) {
        const auto& stats = self->dumpStats();
        c10::List<c10::intrusive_ptr<SourceStats>> ret;
        for (const auto& source : stats) {
          SourceStats::LineMap lineMap;
          for (const auto& line : source.second) {
            lineMap.insert(
                line.first, c10::make_intrusive<InstructionStats>(line.second));
```
- **EN**: This chunk continues `initBindings` and expands its internal control flow or data movement. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically.
- **CN**: 这一段延续了 `initBindings`，进一步展开其内部控制流或数据流转。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。

### Lines 95-104
```cpp
          }
          ret.push_back(c10::make_intrusive<SourceStats>(
              source.first, std::move(lineMap)));
        }
        return ret;
      });
  return nullptr;
}

[[maybe_unused]] const auto torchBindInitializer = initBindings();
```
- **EN**: This chunk continues `initBindings` and expands its internal control flow or data movement. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段延续了 `initBindings`，进一步展开其内部控制流或数据流转。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 106-112
```cpp
} // namespace

namespace profiling {

InstructionSpan::InstructionSpan(Node& node) {
  datapoint_ = std::make_unique<Datapoint>(node.sourceRange());
}
```
- **EN**: The namespace declarations place the code inside profiling, matching the surrounding JIT subsystem. This chunk continues `initBindings` and expands its internal control flow or data movement.
- **CN**: 命名空间声明把代码放入 profiling 中，与周边 JIT 子系统保持一致。 这一段延续了 `initBindings`，进一步展开其内部控制流或数据流转。

### Lines 114-123
```cpp
InstructionSpan::~InstructionSpan() {
  datapoint_->end = std::chrono::steady_clock::now();
  getProfilesRegistry().send(std::move(datapoint_));
}

bool isProfilingOngoing() {
  return !getProfilesRegistry().empty();
}

} // namespace profiling
```
- **EN**: This chunk defines `isProfilingOngoing`, which implements a focused step in the TorchScript execution pipeline. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `isProfilingOngoing`，其作用是实现 TorchScript 执行流水线中的一个关键步骤。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 125-135
```cpp
void ScriptProfile::enable() {
  if (!std::exchange(enabled_, true)) {
    getProfilesRegistry().addProfile(*this);
  }
}

void ScriptProfile::disable() {
  if (std::exchange(enabled_, false)) {
    getProfilesRegistry().removeProfile(*this);
  }
}
```
- **EN**: This chunk defines `disable`, which implements a focused step in the TorchScript execution pipeline. Conditional branches guard special cases and preserve type, shape, or serialization invariants.
- **CN**: 这一段定义了 `disable`，其作用是实现 TorchScript 执行流水线中的一个关键步骤。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。

### Lines 137-144
```cpp
void ScriptProfile::addDatapoint(
    std::shared_ptr<profiling::Datapoint> datapoint) {
  TORCH_CHECK(enabled_, "Cannot only add datapoint to disabled profilers.");
  datapoints_.push_back(std::move(datapoint));
}

const ScriptProfile::SourceMap& ScriptProfile::dumpStats() {
  TORCH_CHECK(!enabled_, "Only disabled profilers are allowed to dump stats.");
```
- **EN**: This chunk defines `dumpStats`, which implements a focused step in the TorchScript execution pipeline. Assertions and error paths make invalid states fail early instead of silently corrupting later stages.
- **CN**: 这一段定义了 `dumpStats`，其作用是实现 TorchScript 执行流水线中的一个关键步骤。 断言与错误路径让非法状态尽早失败，避免悄悄破坏后续阶段。

### Lines 146-157
```cpp
  for (const auto& datapoint : datapoints_) {
    if (const auto& source = datapoint->sourceRange.source()) {
      if (auto fileLineCol = datapoint->sourceRange.file_line_col()) {
        auto it = sourceMap_.find(*source);
        if (it == sourceMap_.end()) {
          it = sourceMap_.emplace(SourceRef{source}, LineMap{}).first;
        }
        auto& stats = it->second[std::get<1>(*fileLineCol)];
        stats.count++;
        stats.duration += datapoint->end - datapoint->start;
      }
    }
```
- **EN**: This chunk continues `dumpStats` and expands its internal control flow or data movement. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Conditional branches guard special cases and preserve type, shape, or serialization invariants.
- **CN**: 这一段延续了 `dumpStats`，进一步展开其内部控制流或数据流转。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。

### Lines 158-168
```cpp
  }
  datapoints_.clear();

  return sourceMap_;
}

ScriptProfile::~ScriptProfile() {
  if (enabled_) {
    getProfilesRegistry().removeProfile(*this);
  }
}
```
- **EN**: This chunk continues `dumpStats` and expands its internal control flow or data movement. Conditional branches guard special cases and preserve type, shape, or serialization invariants. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段延续了 `dumpStats`，进一步展开其内部控制流或数据流转。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 170-170
```cpp
} // namespace torch::jit
```
- **EN**: This chunk continues `dumpStats` and expands its internal control flow or data movement.
- **CN**: 这一段延续了 `dumpStats`，进一步展开其内部控制流或数据流转。

## Key Concepts / 关键概念

- **TorchScript runtime**
  - EN: Implements TorchScript runtime execution, specialization, operator dispatch, and static-runtime support.
  - CN: 实现 TorchScript 运行时执行、特化、算子分发以及静态运行时支持。
- **ProfilesRegistry**
  - EN: `ProfilesRegistry` is a central symbol declared or implemented in this file.
  - CN: `ProfilesRegistry` 是本文件声明或实现的核心符号。
- **empty**
  - EN: `empty` is a central symbol declared or implemented in this file.
  - CN: `empty` 是本文件声明或实现的核心符号。
- **Runtime profiling**
  - EN: Captures observed types or shapes so later passes can specialize execution.
  - CN: 捕获观测到的类型或形状，以便后续 pass 进行特化。
## Dependencies / 依赖关系

- **Internal PyTorch headers / 内部 PyTorch 头文件**: `torch/csrc/jit/runtime/script_profile.h`
- **ATen/c10 foundations / ATen/c10 基础设施**: `c10/util/Exception.h`, `c10/util/intrusive_ptr.h`
- **Standard library / 标准库**: `atomic`, `chrono`, `mutex`, `unordered_set`
- **Primary symbols in this file / 本文件核心符号**: `ProfilesRegistry`, `empty`, `addProfile`, `g`, `removeProfile`, `send`, `getProfilesRegistry`, `initBindings`
