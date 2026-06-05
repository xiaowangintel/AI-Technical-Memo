# DebugCounter.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `llvm/lib/Support/DebugCounter.cpp`
- Repository: `llvm-project`
- Purpose (EN): This class overrides the default list implementation of printing so we can pretty print the list of debug counter options.
- Purpose (CN): 该文件位于 LLVM 的 `Support` 目录中，主要实现与 `DebugCounter` 相关的接口、数据结构和辅助逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-40

```cpp
#include "llvm/Support/DebugCounter.h"

#include "DebugOptions.h"

#include "llvm/Support/CommandLine.h"
#include "llvm/Support/Format.h"
#include "llvm/Support/ManagedStatic.h"

using namespace llvm;

namespace llvm {

void DebugCounter::printChunks(raw_ostream &OS,
                               ArrayRef<IntegerInclusiveInterval> Chunks) {
  IntegerInclusiveIntervalUtils::printIntervals(OS, Chunks, ':');
}

} // namespace llvm

namespace {
// This class overrides the default list implementation of printing so we
// can pretty print the list of debug counter options.  This type of
// dynamic option is pretty rare (basically this and pass lists).
class DebugCounterList : public cl::list<std::string, DebugCounter> {
private:
  using Base = cl::list<std::string, DebugCounter>;

public:
  template <class... Mods>
  explicit DebugCounterList(Mods &&... Ms) : Base(std::forward<Mods>(Ms)...) {}

private:
  void printOptionInfo(size_t GlobalWidth) const override {
    // This is a variant of from generic_parser_base::printOptionInfo.  Sadly,
    // it's not easy to make it more usable.  We could get it to print these as
    // options if we were a cl::opt and registered them, but lists don't have
    // options, nor does the parser for std::string.  The other mechanisms for
    // options are global and would pollute the global namespace with our
    // counters.  Rather than go that route, we have just overridden the
    // printing, which only a few things call anyway.
```
- EN: Introduces the file banner, comments, and the initial setup for this translation unit.
  CN: 这一段给出文件横幅、注释说明以及该编译单元的初始设置。
- EN: Brings in 5 direct dependencies, including `llvm/Support/DebugCounter.h`, `DebugOptions.h`, `llvm/Support/CommandLine.h`, `llvm/Support/Format.h`.
  CN: 引入了 5 个直接依赖，其中包括 `llvm/Support/DebugCounter.h`, `DebugOptions.h`, `llvm/Support/CommandLine.h`, `llvm/Support/Format.h`。
- EN: This section centers on `printChunks`, `printIntervals`, `DebugCounterList` and parses input and converts raw data into structured form.
  CN: 这一段主要围绕 `printChunks`, `printIntervals`, `DebugCounterList` 等符号展开，负责解析输入并把原始数据转换成结构化形式。

### Lines 41-80

```cpp
    outs() << "  -" << ArgStr;
    // All of the other options in CommandLine.cpp use ArgStr.size() + 6 for
    // width, so we do the same.
    Option::printHelpStr(HelpStr, GlobalWidth, ArgStr.size() + 6);
    const auto &CounterInstance = DebugCounter::instance();
    for (const auto &Entry : CounterInstance) {
      const auto &[Name, Desc] = CounterInstance.getCounterDesc(Entry.second);
      size_t NumSpaces = GlobalWidth - Name.size() - 8;
      outs() << "    =" << Name;
      outs().indent(NumSpaces) << " -   " << Desc << '\n';
    }
  }
};

// All global objects associated to the DebugCounter, including the DebugCounter
// itself, are owned by a single global instance of the DebugCounterOwner
// struct. This makes it easier to control the order in which constructors and
// destructors are run.
struct DebugCounterOwner : DebugCounter {
  DebugCounterList DebugCounterOption{
      "debug-counter", cl::Hidden,
      cl::desc("Comma separated list of debug counter skip and count"),
      cl::CommaSeparated, cl::location<DebugCounter>(*this)};
  cl::opt<bool, true> PrintDebugCounter{
      "print-debug-counter",
      cl::Hidden,
      cl::Optional,
      cl::location(this->ShouldPrintCounter),
      cl::init(false),
      cl::desc("Print out debug counter info after all counters accumulated"),
      cl::callback([&](const bool &Value) {
        if (Value)
          activateAllCounters();
      })};
  cl::opt<bool, true> PrintDebugCounterQueries{
      "print-debug-counter-queries",
      cl::Hidden,
      cl::Optional,
      cl::location(this->ShouldPrintCounterQueries),
      cl::init(false),
```
- EN: This section centers on `printHelpStr`, `location` and writes, formats, or serializes results for downstream consumers.
  CN: 这一段主要围绕 `printHelpStr`, `location` 等符号展开，负责为下游使用方写出、格式化或序列化结果。
- EN: In this range, the code iterates over collections, ranges, or records and checks conditions and handles edge cases.
  CN: 在这一段中，代码遍历集合、区间或记录，并检查条件并处理边界情况。

### Lines 81-120

```cpp
      cl::desc("Print out each query of an enabled debug counter")};
  cl::opt<bool, true> BreakOnLastCount{
      "debug-counter-break-on-last",
      cl::Hidden,
      cl::Optional,
      cl::location(this->BreakOnLast),
      cl::init(false),
      cl::desc("Insert a break point on the last enabled count of a "
               "chunks list")};

  DebugCounterOwner() {
    // Our destructor uses the debug stream. By referencing it here, we
    // ensure that its destructor runs after our destructor.
    (void)dbgs();
  }

  // Print information when destroyed, iff command line option is specified.
  ~DebugCounterOwner() {
    if (ShouldPrintCounter)
      print(dbgs());
  }
};

} // anonymous namespace

// Use ManagedStatic instead of function-local static variable to ensure
// the destructor (which accesses counters and streams) runs during
// llvm_shutdown() rather than at some unspecified point.
static ManagedStatic<DebugCounterOwner> Owner;

void llvm::initDebugCounterOptions() { (void)DebugCounter::instance(); }

DebugCounter &DebugCounter::instance() { return *Owner; }

// This is called by the command line parser when it sees a value for the
// debug-counter option defined above.
void DebugCounter::push_back(const std::string &Val) {
  if (Val.empty())
    return;

```
- EN: This section centers on `DebugCounterOwner`, `~DebugCounterOwner`, `initDebugCounterOptions` and parses input and converts raw data into structured form.
  CN: 这一段主要围绕 `DebugCounterOwner`, `~DebugCounterOwner`, `initDebugCounterOptions` 等符号展开，负责解析输入并把原始数据转换成结构化形式。
- EN: In this range, the code checks conditions and handles edge cases and returns the resulting value to its callers.
  CN: 在这一段中，代码检查条件并处理边界情况，并将结果返回给调用方。

### Lines 121-160

```cpp
  // The strings should come in as counter=chunk_list
  auto CounterPair = StringRef(Val).split('=');
  if (CounterPair.second.empty()) {
    errs() << "DebugCounter Error: " << Val << " does not have an = in it\n";
    exit(1);
  }
  StringRef CounterName = CounterPair.first;

  CounterInfo *Counter = getCounterInfo(CounterName);
  if (!Counter) {
    errs() << "DebugCounter Error: " << CounterName
           << " is not a registered counter\n";
    return;
  }

  auto ExpectedChunks =
      IntegerInclusiveIntervalUtils::parseIntervals(CounterPair.second, ':');
  if (!ExpectedChunks) {
    handleAllErrors(ExpectedChunks.takeError(), [&](const StringError &E) {
      errs() << "DebugCounter Error: " << E.getMessage() << "\n";
    });
    exit(1);
  }
  Counter->Chunks = std::move(*ExpectedChunks);
  Counter->Active = Counter->IsSet = true;
}

void DebugCounter::print(raw_ostream &OS) const {
  SmallVector<StringRef, 16> CounterNames(Counters.keys());
  sort(CounterNames);

  OS << "Counters and values:\n";
  for (StringRef CounterName : CounterNames) {
    const CounterInfo *C = getCounterInfo(CounterName);
    OS << left_justify(C->Name, 32) << ": {" << C->Count << ",";
    printChunks(OS, C->Chunks);
    OS << "}\n";
  }
}

```
- EN: This section centers on `exit`, `parseIntervals`, `handleAllErrors` and parses input and converts raw data into structured form.
  CN: 这一段主要围绕 `exit`, `parseIntervals`, `handleAllErrors` 等符号展开，负责解析输入并把原始数据转换成结构化形式。
- EN: In this range, the code iterates over collections, ranges, or records and checks conditions and handles edge cases.
  CN: 在这一段中，代码遍历集合、区间或记录，并检查条件并处理边界情况。

### Lines 161-200

```cpp
bool DebugCounter::handleCounterIncrement(CounterInfo &Info) {
  int64_t CurrCount = Info.Count++;
  uint64_t CurrIdx = Info.CurrChunkIdx;

  if (Info.Chunks.empty())
    return true;
  if (CurrIdx >= Info.Chunks.size())
    return false;

  bool Res = Info.Chunks[CurrIdx].contains(CurrCount);
  if (BreakOnLast && CurrIdx == (Info.Chunks.size() - 1) &&
      CurrCount == Info.Chunks[CurrIdx].getEnd()) {
    LLVM_BUILTIN_DEBUGTRAP;
  }
  if (CurrCount > Info.Chunks[CurrIdx].getEnd()) {
    Info.CurrChunkIdx++;

    /// Handle consecutive blocks.
    if (Info.CurrChunkIdx < Info.Chunks.size() &&
        CurrCount == Info.Chunks[Info.CurrChunkIdx].getBegin())
      return true;
  }
  return Res;
}

bool DebugCounter::shouldExecuteImpl(CounterInfo &Counter) {
  auto &Us = instance();
  bool Res = Us.handleCounterIncrement(Counter);
  if (Us.ShouldPrintCounterQueries && Counter.IsSet) {
    dbgs() << "DebugCounter " << Counter.Name << "=" << (Counter.Count - 1)
           << (Res ? " execute" : " skip") << "\n";
  }
  return Res;
}

#if !defined(NDEBUG) || defined(LLVM_ENABLE_DUMP)
LLVM_DUMP_METHOD void DebugCounter::dump() const {
  print(dbgs());
}
#endif
```
- EN: This section centers on `handleCounterIncrement`, `shouldExecuteImpl`, `dump` and writes, formats, or serializes results for downstream consumers.
  CN: 这一段主要围绕 `handleCounterIncrement`, `shouldExecuteImpl`, `dump` 等符号展开，负责为下游使用方写出、格式化或序列化结果。
- EN: In this range, the code checks conditions and handles edge cases and returns the resulting value to its callers.
  CN: 在这一段中，代码检查条件并处理边界情况，并将结果返回给调用方。

## Key Concepts / 关键概念
- Domain / 领域: LLVM support utilities / LLVM 支撑工具
- Core symbols / 核心符号: `overrides`, `DebugCounterList`, `DebugCounterOwner`, `printChunks`, `printIntervals`, `printHelpStr` / 该文件围绕这些类型或函数组织主要逻辑。
- Data flow / 数据流: Parsing, decoding, and structural validation. / 重点关注解析、解码与结构校验。
- Error model / 错误模型: LLVM-style `Error`/`Expected` handling and invariant checks. / 使用 LLVM 风格的 `Error`/`Expected` 处理与不变量检查。
- Data structures / 数据结构: Relies on LLVM or STL containers for compact state management. / 依赖 LLVM 或 STL 容器管理紧凑状态。

## Dependencies / 依赖关系
- LLVM headers / LLVM 头文件: `llvm/Support/DebugCounter.h`, `llvm/Support/CommandLine.h`, `llvm/Support/Format.h`, `llvm/Support/ManagedStatic.h`
- Standard library / 标准库: None / 无
- Other/system headers / 其他或系统头文件: `DebugOptions.h`
- Related symbols / 相关符号: `overrides`, `DebugCounterList`, `DebugCounterOwner`, `printChunks`, `printIntervals`, `printHelpStr`, `location`
