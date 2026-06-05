# X86Counter.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `llvm/tools/llvm-exegesis/lib/X86/X86Counter.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**: This file belongs to `lib/X86` and implements benchmarking, target modeling, or analysis helpers for `X86Counter`. / 该文件位于 `lib/X86`，主要实现与 `X86Counter` 相关的基准分析、目标建模或辅助逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-18

```cpp
//===-- X86Counter.cpp ------------------------------------------*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#include "X86Counter.h"

#if defined(__linux__) && defined(HAVE_LIBPFM) &&                              \
    defined(LIBPFM_HAS_FIELD_CYCLES)

// FIXME: Use appropriate wrappers for poll.h and mman.h
// to support Windows and remove this linux-only guard.

#include "llvm/Support/Endian.h"
#include "llvm/Support/Errc.h"
```

- **L1**: Banner comment marking a file or section boundary. / 横幅注释，用于标记文件或章节边界。
- **L2**: Separator comment used to visually divide sections. / 分隔性注释，用于在视觉上划分小节。
- **L3**: Comment explains nearby logic or intent: `Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.`. / 注释说明了附近代码的逻辑或设计意图：`Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.`。
- **L4**: Comment explains nearby logic or intent: `See https://llvm.org/LICENSE.txt for license information.`. / 注释说明了附近代码的逻辑或设计意图：`See https://llvm.org/LICENSE.txt for license information.`。
- **L5**: Comment explains nearby logic or intent: `SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception`. / 注释说明了附近代码的逻辑或设计意图：`SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception`。
- **L6**: Separator comment used to visually divide sections. / 分隔性注释，用于在视觉上划分小节。
- **L7**: Banner comment marking a file or section boundary. / 横幅注释，用于标记文件或章节边界。
- **L8**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L9**: Includes `X86Counter.h` to access local declarations paired with this implementation file. / 引入 `X86Counter.h` 以使用与该实现文件配套的本地声明。
- **L10**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L11**: Preprocessor directive controls conditional compilation or build behavior: `#if defined(__linux__) && defined(HAVE_LIBPFM) && \`. / 预处理指令控制条件编译或构建行为：`#if defined(__linux__) && defined(HAVE_LIBPFM) && \`。
- **L12**: Continues the surrounding expression or declaration: `defined(LIBPFM_HAS_FIELD_CYCLES)`. / 继续构造周围的表达式或声明：`defined(LIBPFM_HAS_FIELD_CYCLES)`。
- **L13**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L14**: Comment records an implementation note or caution: `FIXME: Use appropriate wrappers for poll.h and mman.h`. / 注释记录了一条实现说明或注意事项：`FIXME: Use appropriate wrappers for poll.h and mman.h`。
- **L15**: Comment explains nearby logic or intent: `to support Windows and remove this linux-only guard.`. / 注释说明了附近代码的逻辑或设计意图：`to support Windows and remove this linux-only guard.`。
- **L16**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L17**: Includes `llvm/Support/Endian.h` to access LLVM support-library facilities. / 引入 `llvm/Support/Endian.h` 以使用LLVM 支持库设施。
- **L18**: Includes `llvm/Support/Errc.h` to access LLVM support-library facilities. / 引入 `llvm/Support/Errc.h` 以使用LLVM 支持库设施。

### Lines 19-36

```cpp

#include <perfmon/perf_event.h>
#include <perfmon/pfmlib.h>
#include <perfmon/pfmlib_perf_event.h>

#include <atomic>
#include <chrono>
#include <cstddef>
#include <cstdint>
#include <limits>
#include <memory>

#include <poll.h>
#include <sys/mman.h>
#include <unistd.h>

namespace llvm {
namespace exegesis {
```

- **L19**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L20**: Includes `perfmon/perf_event.h` to access local declarations paired with this implementation file. / 引入 `perfmon/perf_event.h` 以使用与该实现文件配套的本地声明。
- **L21**: Includes `perfmon/pfmlib.h` to access local declarations paired with this implementation file. / 引入 `perfmon/pfmlib.h` 以使用与该实现文件配套的本地声明。
- **L22**: Includes `perfmon/pfmlib_perf_event.h` to access local declarations paired with this implementation file. / 引入 `perfmon/pfmlib_perf_event.h` 以使用与该实现文件配套的本地声明。
- **L23**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L24**: Includes `atomic` to access supporting declarations required by this file. / 引入 `atomic` 以使用本文件所需的辅助声明。
- **L25**: Includes `chrono` to access supporting declarations required by this file. / 引入 `chrono` 以使用本文件所需的辅助声明。
- **L26**: Includes `cstddef` to access supporting declarations required by this file. / 引入 `cstddef` 以使用本文件所需的辅助声明。
- **L27**: Includes `cstdint` to access supporting declarations required by this file. / 引入 `cstdint` 以使用本文件所需的辅助声明。
- **L28**: Includes `limits` to access supporting declarations required by this file. / 引入 `limits` 以使用本文件所需的辅助声明。
- **L29**: Includes `memory` to access supporting declarations required by this file. / 引入 `memory` 以使用本文件所需的辅助声明。
- **L30**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L31**: Includes `poll.h` to access local declarations paired with this implementation file. / 引入 `poll.h` 以使用与该实现文件配套的本地声明。
- **L32**: Includes `sys/mman.h` to access local declarations paired with this implementation file. / 引入 `sys/mman.h` 以使用与该实现文件配套的本地声明。
- **L33**: Includes `unistd.h` to access local declarations paired with this implementation file. / 引入 `unistd.h` 以使用与该实现文件配套的本地声明。
- **L34**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L35**: Opens namespace scope `llvm`. / 打开命名空间作用域 `llvm`。
- **L36**: Opens namespace scope `exegesis`. / 打开命名空间作用域 `exegesis`。

### Lines 37-54

```cpp

// Number of entries in the LBR.
static constexpr int kLbrEntries = 16;
static constexpr size_t kBufferPages = 8;
static const size_t kDataBufferSize = kBufferPages * getpagesize();

// First page is reserved for perf_event_mmap_page. Data buffer starts on
// the next page, so we allocate one more page.
static const size_t kMappedBufferSize = (kBufferPages + 1) * getpagesize();

// Waits for the LBR perf events.
static int pollLbrPerfEvent(const int FileDescriptor) {
  struct pollfd PollFd;
  PollFd.fd = FileDescriptor;
  PollFd.events = POLLIN;
  PollFd.revents = 0;
  return poll(&PollFd, 1 /* num of fds */, 10000 /* timeout in ms */);
}
```

- **L37**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L38**: Comment explains nearby logic or intent: `Number of entries in the LBR.`. / 注释说明了附近代码的逻辑或设计意图：`Number of entries in the LBR.`。
- **L39**: Initializes or updates `static constexpr int kLbrEntries` from the right-hand expression. / 使用右侧表达式初始化或更新 `static constexpr int kLbrEntries`。
- **L40**: Initializes or updates `static constexpr size_t kBufferPages` from the right-hand expression. / 使用右侧表达式初始化或更新 `static constexpr size_t kBufferPages`。
- **L41**: Declares or invokes `getpagesize`. / 声明或调用 `getpagesize`。
- **L42**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L43**: Comment explains nearby logic or intent: `First page is reserved for perf_event_mmap_page. Data buffer starts on`. / 注释说明了附近代码的逻辑或设计意图：`First page is reserved for perf_event_mmap_page. Data buffer starts on`。
- **L44**: Comment explains nearby logic or intent: `the next page, so we allocate one more page.`. / 注释说明了附近代码的逻辑或设计意图：`the next page, so we allocate one more page.`。
- **L45**: Declares or invokes `=`. / 声明或调用 `=`。
- **L46**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L47**: Comment explains nearby logic or intent: `Waits for the LBR perf events.`. / 注释说明了附近代码的逻辑或设计意图：`Waits for the LBR perf events.`。
- **L48**: Starts the definition of function or method `pollLbrPerfEvent`. / 开始定义函数或方法 `pollLbrPerfEvent`。
- **L49**: Declares struct `PollFd;`. / 声明 struct `PollFd;`。
- **L50**: Initializes or updates `PollFd.fd` from the right-hand expression. / 使用右侧表达式初始化或更新 `PollFd.fd`。
- **L51**: Initializes or updates `PollFd.events` from the right-hand expression. / 使用右侧表达式初始化或更新 `PollFd.events`。
- **L52**: Initializes or updates `PollFd.revents` from the right-hand expression. / 使用右侧表达式初始化或更新 `PollFd.revents`。
- **L53**: Returns control, optionally with a value: `return poll(&PollFd, 1 /* num of fds */, 10000 /* timeout in ms */);`. / 返回控制流，并可附带返回值：`return poll(&PollFd, 1 /* num of fds */, 10000 /* timeout in ms */);`。
- **L54**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。

### Lines 55-72

```cpp

// Copies the data-buffer into Buf, given the pointer to MMapped.
static void copyDataBuffer(void *MMappedBuffer, char *Buf, uint64_t Tail,
                           size_t DataSize) {
  // First page is reserved for perf_event_mmap_page. Data buffer starts on
  // the next page.
  char *Start = reinterpret_cast<char *>(MMappedBuffer) + getpagesize();
  // The LBR buffer is a cyclic buffer, we copy data to another buffer.
  uint64_t Offset = Tail % kDataBufferSize;
  size_t CopySize = kDataBufferSize - Offset;
  memcpy(Buf, Start + Offset, CopySize);
  if (CopySize >= DataSize)
    return;

  memcpy(Buf + CopySize, Start, Offset);
  return;
}

```

- **L55**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L56**: Comment explains nearby logic or intent: `Copies the data-buffer into Buf, given the pointer to MMapped.`. / 注释说明了附近代码的逻辑或设计意图：`Copies the data-buffer into Buf, given the pointer to MMapped.`。
- **L57**: Continues a multi-line argument list or initializer: `static void copyDataBuffer(void *MMappedBuffer, char *Buf, uint64_t Tail,`. / 继续一个多行参数列表或初始化器：`static void copyDataBuffer(void *MMappedBuffer, char *Buf, uint64_t Tail,`。
- **L58**: Continues the surrounding expression or declaration: `size_t DataSize) {`. / 继续构造周围的表达式或声明：`size_t DataSize) {`。
- **L59**: Comment explains nearby logic or intent: `First page is reserved for perf_event_mmap_page. Data buffer starts on`. / 注释说明了附近代码的逻辑或设计意图：`First page is reserved for perf_event_mmap_page. Data buffer starts on`。
- **L60**: Comment explains nearby logic or intent: `the next page.`. / 注释说明了附近代码的逻辑或设计意图：`the next page.`。
- **L61**: Declares or invokes `>`. / 声明或调用 `>`。
- **L62**: Comment explains nearby logic or intent: `The LBR buffer is a cyclic buffer, we copy data to another buffer.`. / 注释说明了附近代码的逻辑或设计意图：`The LBR buffer is a cyclic buffer, we copy data to another buffer.`。
- **L63**: Initializes or updates `uint64_t Offset` from the right-hand expression. / 使用右侧表达式初始化或更新 `uint64_t Offset`。
- **L64**: Initializes or updates `size_t CopySize` from the right-hand expression. / 使用右侧表达式初始化或更新 `size_t CopySize`。
- **L65**: Declares or invokes `memcpy`. / 声明或调用 `memcpy`。
- **L66**: Introduces a conditional branch: `if (CopySize >= DataSize)`. / 引入条件分支：`if (CopySize >= DataSize)`。
- **L67**: Executes a standalone statement or declaration: `return;`. / 执行一条独立语句或声明：`return;`。
- **L68**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L69**: Declares or invokes `memcpy`. / 声明或调用 `memcpy`。
- **L70**: Executes a standalone statement or declaration: `return;`. / 执行一条独立语句或声明：`return;`。
- **L71**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L72**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。

### Lines 73-90

```cpp
// Parses the given data-buffer for stats and fill the CycleArray.
// If data has been extracted successfully, also modifies the code to jump
// out the benchmark loop.
static Error parseDataBuffer(const char *DataBuf, size_t DataSize,
                             const void *From, const void *To,
                             SmallVector<int64_t, 4> *CycleArray) {
  const char *DataPtr = DataBuf;
  while (DataPtr < DataBuf + DataSize) {
    struct perf_event_header Header;
    memcpy(&Header, DataPtr, sizeof(struct perf_event_header));
    if (Header.type != PERF_RECORD_SAMPLE) {
      // Ignores non-sample records.
      DataPtr += Header.size;
      continue;
    }
    DataPtr += sizeof(Header);
    uint64_t Count = support::endian::read64(DataPtr, endianness::native);
    DataPtr += sizeof(Count);
```

- **L73**: Comment explains nearby logic or intent: `Parses the given data-buffer for stats and fill the CycleArray.`. / 注释说明了附近代码的逻辑或设计意图：`Parses the given data-buffer for stats and fill the CycleArray.`。
- **L74**: Comment explains nearby logic or intent: `If data has been extracted successfully, also modifies the code to jump`. / 注释说明了附近代码的逻辑或设计意图：`If data has been extracted successfully, also modifies the code to jump`。
- **L75**: Comment explains nearby logic or intent: `out the benchmark loop.`. / 注释说明了附近代码的逻辑或设计意图：`out the benchmark loop.`。
- **L76**: Continues a multi-line argument list or initializer: `static Error parseDataBuffer(const char *DataBuf, size_t DataSize,`. / 继续一个多行参数列表或初始化器：`static Error parseDataBuffer(const char *DataBuf, size_t DataSize,`。
- **L77**: Continues a multi-line argument list or initializer: `const void *From, const void *To,`. / 继续一个多行参数列表或初始化器：`const void *From, const void *To,`。
- **L78**: Continues the surrounding expression or declaration: `SmallVector<int64_t, 4> *CycleArray) {`. / 继续构造周围的表达式或声明：`SmallVector<int64_t, 4> *CycleArray) {`。
- **L79**: Initializes or updates `const char *DataPtr` from the right-hand expression. / 使用右侧表达式初始化或更新 `const char *DataPtr`。
- **L80**: Starts a while-loop guarded by a runtime condition: `while (DataPtr < DataBuf + DataSize) {`. / 开始由运行时条件控制的 while 循环：`while (DataPtr < DataBuf + DataSize) {`。
- **L81**: Declares struct `Header;`. / 声明 struct `Header;`。
- **L82**: Declares or invokes `memcpy`. / 声明或调用 `memcpy`。
- **L83**: Introduces a conditional branch: `if (Header.type != PERF_RECORD_SAMPLE) {`. / 引入条件分支：`if (Header.type != PERF_RECORD_SAMPLE) {`。
- **L84**: Comment explains nearby logic or intent: `Ignores non-sample records.`. / 注释说明了附近代码的逻辑或设计意图：`Ignores non-sample records.`。
- **L85**: Initializes or updates `DataPtr +` from the right-hand expression. / 使用右侧表达式初始化或更新 `DataPtr +`。
- **L86**: Executes a standalone statement or declaration: `continue;`. / 执行一条独立语句或声明：`continue;`。
- **L87**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L88**: Declares or invokes `sizeof`. / 声明或调用 `sizeof`。
- **L89**: Declares or invokes `support::endian::read64`. / 声明或调用 `support::endian::read64`。
- **L90**: Declares or invokes `sizeof`. / 声明或调用 `sizeof`。

### Lines 91-108

```cpp

    struct perf_branch_entry Entry;
    memcpy(&Entry, DataPtr, sizeof(struct perf_branch_entry));

    // Read the perf_branch_entry array.
    for (uint64_t i = 0; i < Count; ++i) {
      const uint64_t BlockStart = From == nullptr
                                      ? std::numeric_limits<uint64_t>::min()
                                      : reinterpret_cast<uint64_t>(From);
      const uint64_t BlockEnd = To == nullptr
                                    ? std::numeric_limits<uint64_t>::max()
                                    : reinterpret_cast<uint64_t>(To);

      if (BlockStart <= Entry.from && BlockEnd >= Entry.to)
        CycleArray->push_back(Entry.cycles);

      if (i == Count - 1)
        // We've reached the last entry.
```

- **L91**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L92**: Declares struct `Entry;`. / 声明 struct `Entry;`。
- **L93**: Declares or invokes `memcpy`. / 声明或调用 `memcpy`。
- **L94**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L95**: Comment explains nearby logic or intent: `Read the perf_branch_entry array.`. / 注释说明了附近代码的逻辑或设计意图：`Read the perf_branch_entry array.`。
- **L96**: Starts a loop over a range or sequence: `for (uint64_t i = 0; i < Count; ++i) {`. / 开始遍历范围或序列的循环：`for (uint64_t i = 0; i < Count; ++i) {`。
- **L97**: Continues the surrounding expression or declaration: `const uint64_t BlockStart = From == nullptr`. / 继续构造周围的表达式或声明：`const uint64_t BlockStart = From == nullptr`。
- **L98**: Continues the surrounding expression or declaration: `? std::numeric_limits<uint64_t>::min()`. / 继续构造周围的表达式或声明：`? std::numeric_limits<uint64_t>::min()`。
- **L99**: Declares or invokes `reinterpret_cast<uint64_t>`. / 声明或调用 `reinterpret_cast<uint64_t>`。
- **L100**: Continues the surrounding expression or declaration: `const uint64_t BlockEnd = To == nullptr`. / 继续构造周围的表达式或声明：`const uint64_t BlockEnd = To == nullptr`。
- **L101**: Continues the surrounding expression or declaration: `? std::numeric_limits<uint64_t>::max()`. / 继续构造周围的表达式或声明：`? std::numeric_limits<uint64_t>::max()`。
- **L102**: Declares or invokes `reinterpret_cast<uint64_t>`. / 声明或调用 `reinterpret_cast<uint64_t>`。
- **L103**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L104**: Introduces a conditional branch: `if (BlockStart <= Entry.from && BlockEnd >= Entry.to)`. / 引入条件分支：`if (BlockStart <= Entry.from && BlockEnd >= Entry.to)`。
- **L105**: Declares or invokes `CycleArray->push_back`. / 声明或调用 `CycleArray->push_back`。
- **L106**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L107**: Introduces a conditional branch: `if (i == Count - 1)`. / 引入条件分支：`if (i == Count - 1)`。
- **L108**: Comment explains nearby logic or intent: `We've reached the last entry.`. / 注释说明了附近代码的逻辑或设计意图：`We've reached the last entry.`。

### Lines 109-126

```cpp
        return Error::success();

      // Advance to next entry
      DataPtr += sizeof(Entry);
      memcpy(&Entry, DataPtr, sizeof(struct perf_branch_entry));
    }
  }
  return make_error<StringError>("Unable to parse databuffer.", errc::io_error);
}

X86LbrPerfEvent::X86LbrPerfEvent(unsigned SamplingPeriod) {
  assert(SamplingPeriod > 0 && "SamplingPeriod must be positive");
  EventString = "BR_INST_RETIRED.NEAR_TAKEN";
  Attr = new perf_event_attr();
  Attr->size = sizeof(*Attr);
  Attr->type = PERF_TYPE_RAW;
  // FIXME This is SKL's encoding. Not sure if it'll change.
  Attr->config = 0x20c4; // BR_INST_RETIRED.NEAR_TAKEN
```

- **L109**: Returns control, optionally with a value: `return Error::success();`. / 返回控制流，并可附带返回值：`return Error::success();`。
- **L110**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L111**: Comment explains nearby logic or intent: `Advance to next entry`. / 注释说明了附近代码的逻辑或设计意图：`Advance to next entry`。
- **L112**: Declares or invokes `sizeof`. / 声明或调用 `sizeof`。
- **L113**: Declares or invokes `memcpy`. / 声明或调用 `memcpy`。
- **L114**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L115**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L116**: Returns control, optionally with a value: `return make_error<StringError>("Unable to parse databuffer.", errc::io_error);`. / 返回控制流，并可附带返回值：`return make_error<StringError>("Unable to parse databuffer.", errc::io_error);`。
- **L117**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L118**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L119**: Starts the definition of function or method `X86LbrPerfEvent::X86LbrPerfEvent`. / 开始定义函数或方法 `X86LbrPerfEvent::X86LbrPerfEvent`。
- **L120**: Checks an internal invariant with an assertion: `assert(SamplingPeriod > 0 && "SamplingPeriod must be positive");`. / 通过断言检查内部不变式：`assert(SamplingPeriod > 0 && "SamplingPeriod must be positive");`。
- **L121**: Initializes or updates `EventString` from the right-hand expression. / 使用右侧表达式初始化或更新 `EventString`。
- **L122**: Declares or invokes `perf_event_attr`. / 声明或调用 `perf_event_attr`。
- **L123**: Declares or invokes `sizeof`. / 声明或调用 `sizeof`。
- **L124**: Initializes or updates `Attr->type` from the right-hand expression. / 使用右侧表达式初始化或更新 `Attr->type`。
- **L125**: Comment records an implementation note or caution: `FIXME This is SKL's encoding. Not sure if it'll change.`. / 注释记录了一条实现说明或注意事项：`FIXME This is SKL's encoding. Not sure if it'll change.`。
- **L126**: Continues the surrounding expression or declaration: `Attr->config = 0x20c4; // BR_INST_RETIRED.NEAR_TAKEN`. / 继续构造周围的表达式或声明：`Attr->config = 0x20c4; // BR_INST_RETIRED.NEAR_TAKEN`。

### Lines 127-144

```cpp
  Attr->sample_type = PERF_SAMPLE_BRANCH_STACK;
  // Don't need to specify "USER" because we've already excluded HV and Kernel.
  Attr->branch_sample_type = PERF_SAMPLE_BRANCH_ANY;
  Attr->sample_period = SamplingPeriod;
  Attr->wakeup_events = 1; // We need this even when using ioctl REFRESH.
  Attr->disabled = 1;
  Attr->exclude_kernel = 1;
  Attr->exclude_hv = 1;
  Attr->read_format = PERF_FORMAT_GROUP;

  FullQualifiedEventString = EventString;
}

X86LbrCounter::X86LbrCounter(pfm::PerfEvent &&NewEvent)
    : CounterGroup(std::move(NewEvent), {}) {
  MMappedBuffer = mmap(nullptr, kMappedBufferSize, PROT_READ | PROT_WRITE,
                       MAP_SHARED, getFileDescriptor(), 0);
  if (MMappedBuffer == MAP_FAILED)
```

- **L127**: Initializes or updates `Attr->sample_type` from the right-hand expression. / 使用右侧表达式初始化或更新 `Attr->sample_type`。
- **L128**: Comment explains nearby logic or intent: `Don't need to specify "USER" because we've already excluded HV and Kernel.`. / 注释说明了附近代码的逻辑或设计意图：`Don't need to specify "USER" because we've already excluded HV and Kernel.`。
- **L129**: Initializes or updates `Attr->branch_sample_type` from the right-hand expression. / 使用右侧表达式初始化或更新 `Attr->branch_sample_type`。
- **L130**: Initializes or updates `Attr->sample_period` from the right-hand expression. / 使用右侧表达式初始化或更新 `Attr->sample_period`。
- **L131**: Continues the surrounding expression or declaration: `Attr->wakeup_events = 1; // We need this even when using ioctl REFRESH.`. / 继续构造周围的表达式或声明：`Attr->wakeup_events = 1; // We need this even when using ioctl REFRESH.`。
- **L132**: Initializes or updates `Attr->disabled` from the right-hand expression. / 使用右侧表达式初始化或更新 `Attr->disabled`。
- **L133**: Initializes or updates `Attr->exclude_kernel` from the right-hand expression. / 使用右侧表达式初始化或更新 `Attr->exclude_kernel`。
- **L134**: Initializes or updates `Attr->exclude_hv` from the right-hand expression. / 使用右侧表达式初始化或更新 `Attr->exclude_hv`。
- **L135**: Initializes or updates `Attr->read_format` from the right-hand expression. / 使用右侧表达式初始化或更新 `Attr->read_format`。
- **L136**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L137**: Initializes or updates `FullQualifiedEventString` from the right-hand expression. / 使用右侧表达式初始化或更新 `FullQualifiedEventString`。
- **L138**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L139**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L140**: Continues the surrounding expression or declaration: `X86LbrCounter::X86LbrCounter(pfm::PerfEvent &&NewEvent)`. / 继续构造周围的表达式或声明：`X86LbrCounter::X86LbrCounter(pfm::PerfEvent &&NewEvent)`。
- **L141**: Starts the definition of function or method `CounterGroup`. / 开始定义函数或方法 `CounterGroup`。
- **L142**: Continues a multi-line argument list or initializer: `MMappedBuffer = mmap(nullptr, kMappedBufferSize, PROT_READ | PROT_WRITE,`. / 继续一个多行参数列表或初始化器：`MMappedBuffer = mmap(nullptr, kMappedBufferSize, PROT_READ | PROT_WRITE,`。
- **L143**: Declares or invokes `getFileDescriptor`. / 声明或调用 `getFileDescriptor`。
- **L144**: Introduces a conditional branch: `if (MMappedBuffer == MAP_FAILED)`. / 引入条件分支：`if (MMappedBuffer == MAP_FAILED)`。

### Lines 145-162

```cpp
    errs() << "Failed to mmap buffer.";
}

X86LbrCounter::~X86LbrCounter() {
  if (0 != munmap(MMappedBuffer, kMappedBufferSize))
    errs() << "Failed to munmap buffer.";
}

void X86LbrCounter::start() {
  ioctl(getFileDescriptor(), PERF_EVENT_IOC_REFRESH, 1024 /* kMaxPollsPerFd */);
}

Error X86LbrCounter::checkLbrSupport() {
  // Do a sample read and check if the results contain non-zero values.

  X86LbrCounter counter(X86LbrPerfEvent(123));
  counter.start();

```

- **L145**: Declares or invokes `errs`. / 声明或调用 `errs`。
- **L146**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L147**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L148**: Starts the definition of function or method `X86LbrCounter::~X86LbrCounter`. / 开始定义函数或方法 `X86LbrCounter::~X86LbrCounter`。
- **L149**: Introduces a conditional branch: `if (0 != munmap(MMappedBuffer, kMappedBufferSize))`. / 引入条件分支：`if (0 != munmap(MMappedBuffer, kMappedBufferSize))`。
- **L150**: Declares or invokes `errs`. / 声明或调用 `errs`。
- **L151**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L152**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L153**: Starts the definition of function or method `X86LbrCounter::start`. / 开始定义函数或方法 `X86LbrCounter::start`。
- **L154**: Declares or invokes `ioctl`. / 声明或调用 `ioctl`。
- **L155**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L156**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L157**: Starts the definition of function or method `X86LbrCounter::checkLbrSupport`. / 开始定义函数或方法 `X86LbrCounter::checkLbrSupport`。
- **L158**: Comment explains nearby logic or intent: `Do a sample read and check if the results contain non-zero values.`. / 注释说明了附近代码的逻辑或设计意图：`Do a sample read and check if the results contain non-zero values.`。
- **L159**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L160**: Declares or invokes `counter`. / 声明或调用 `counter`。
- **L161**: Declares or invokes `counter.start`. / 声明或调用 `counter.start`。
- **L162**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。

### Lines 163-180

```cpp
  // Prevent the compiler from unrolling the loop and get rid of all the
  // branches. We need at least 16 iterations.
  int Sum = 0;
  int V = 1;

  volatile int *P = &V;
  auto TimeLimit =
      std::chrono::high_resolution_clock::now() + std::chrono::microseconds(5);

  for (int I = 0;
       I < kLbrEntries || std::chrono::high_resolution_clock::now() < TimeLimit;
       ++I) {
    Sum += *P;
  }

  counter.stop();
  (void)Sum;

```

- **L163**: Comment explains nearby logic or intent: `Prevent the compiler from unrolling the loop and get rid of all the`. / 注释说明了附近代码的逻辑或设计意图：`Prevent the compiler from unrolling the loop and get rid of all the`。
- **L164**: Comment explains nearby logic or intent: `branches. We need at least 16 iterations.`. / 注释说明了附近代码的逻辑或设计意图：`branches. We need at least 16 iterations.`。
- **L165**: Initializes or updates `int Sum` from the right-hand expression. / 使用右侧表达式初始化或更新 `int Sum`。
- **L166**: Initializes or updates `int V` from the right-hand expression. / 使用右侧表达式初始化或更新 `int V`。
- **L167**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L168**: Initializes or updates `volatile int *P` from the right-hand expression. / 使用右侧表达式初始化或更新 `volatile int *P`。
- **L169**: Continues the surrounding expression or declaration: `auto TimeLimit =`. / 继续构造周围的表达式或声明：`auto TimeLimit =`。
- **L170**: Declares or invokes `std::chrono::high_resolution_clock::now`. / 声明或调用 `std::chrono::high_resolution_clock::now`。
- **L171**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L172**: Starts a loop over a range or sequence: `for (int I = 0;`. / 开始遍历范围或序列的循环：`for (int I = 0;`。
- **L173**: Declares or invokes `std::chrono::high_resolution_clock::now`. / 声明或调用 `std::chrono::high_resolution_clock::now`。
- **L174**: Continues the surrounding expression or declaration: `++I) {`. / 继续构造周围的表达式或声明：`++I) {`。
- **L175**: Initializes or updates `Sum +` from the right-hand expression. / 使用右侧表达式初始化或更新 `Sum +`。
- **L176**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L177**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L178**: Declares or invokes `counter.stop`. / 声明或调用 `counter.stop`。
- **L179**: Executes a standalone statement or declaration: `(void)Sum;`. / 执行一条独立语句或声明：`(void)Sum;`。
- **L180**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。

### Lines 181-198

```cpp
  auto ResultOrError = counter.doReadCounter(nullptr, nullptr);
  if (ResultOrError)
    if (!ResultOrError.get().empty())
      // If there is at least one non-zero entry, then LBR is supported.
      for (const int64_t &Value : ResultOrError.get())
        if (Value != 0)
          return Error::success();

  return make_error<StringError>(
      "LBR format with cycles is not suppported on the host.",
      errc::not_supported);
}

Expected<SmallVector<int64_t, 4>>
X86LbrCounter::readOrError(StringRef FunctionBytes) const {
  // Disable the event before reading
  ioctl(getFileDescriptor(), PERF_EVENT_IOC_DISABLE, 0);

```

- **L181**: Declares or invokes `counter.doReadCounter`. / 声明或调用 `counter.doReadCounter`。
- **L182**: Introduces a conditional branch: `if (ResultOrError)`. / 引入条件分支：`if (ResultOrError)`。
- **L183**: Introduces a conditional branch: `if (!ResultOrError.get().empty())`. / 引入条件分支：`if (!ResultOrError.get().empty())`。
- **L184**: Comment explains nearby logic or intent: `If there is at least one non-zero entry, then LBR is supported.`. / 注释说明了附近代码的逻辑或设计意图：`If there is at least one non-zero entry, then LBR is supported.`。
- **L185**: Starts a loop over a range or sequence: `for (const int64_t &Value : ResultOrError.get())`. / 开始遍历范围或序列的循环：`for (const int64_t &Value : ResultOrError.get())`。
- **L186**: Introduces a conditional branch: `if (Value != 0)`. / 引入条件分支：`if (Value != 0)`。
- **L187**: Returns control, optionally with a value: `return Error::success();`. / 返回控制流，并可附带返回值：`return Error::success();`。
- **L188**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L189**: Returns control, optionally with a value: `return make_error<StringError>(`. / 返回控制流，并可附带返回值：`return make_error<StringError>(`。
- **L190**: Continues a multi-line argument list or initializer: `"LBR format with cycles is not suppported on the host.",`. / 继续一个多行参数列表或初始化器：`"LBR format with cycles is not suppported on the host.",`。
- **L191**: Executes a standalone statement or declaration: `errc::not_supported);`. / 执行一条独立语句或声明：`errc::not_supported);`。
- **L192**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L193**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L194**: Continues the surrounding expression or declaration: `Expected<SmallVector<int64_t, 4>>`. / 继续构造周围的表达式或声明：`Expected<SmallVector<int64_t, 4>>`。
- **L195**: Starts the definition of function or method `X86LbrCounter::readOrError`. / 开始定义函数或方法 `X86LbrCounter::readOrError`。
- **L196**: Comment explains nearby logic or intent: `Disable the event before reading`. / 注释说明了附近代码的逻辑或设计意图：`Disable the event before reading`。
- **L197**: Declares or invokes `ioctl`. / 声明或调用 `ioctl`。
- **L198**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。

### Lines 199-216

```cpp
  // Find the boundary of the function so that we could filter the LBRs
  // to keep only the relevant records.
  if (FunctionBytes.empty())
    return make_error<StringError>("Empty function bytes",
                                   errc::invalid_argument);
  const void *From = reinterpret_cast<const void *>(FunctionBytes.data());
  const void *To = reinterpret_cast<const void *>(FunctionBytes.data() +
                                                  FunctionBytes.size());
  return doReadCounter(From, To);
}

Expected<SmallVector<int64_t, 4>>
X86LbrCounter::doReadCounter(const void *From, const void *To) const {
  // The max number of time-outs/retries before we give up.
  static constexpr int kMaxTimeouts = 160;

  // Parses the LBR buffer and fills CycleArray with the sequence of cycle
  // counts from the buffer.
```

- **L199**: Comment explains nearby logic or intent: `Find the boundary of the function so that we could filter the LBRs`. / 注释说明了附近代码的逻辑或设计意图：`Find the boundary of the function so that we could filter the LBRs`。
- **L200**: Comment explains nearby logic or intent: `to keep only the relevant records.`. / 注释说明了附近代码的逻辑或设计意图：`to keep only the relevant records.`。
- **L201**: Introduces a conditional branch: `if (FunctionBytes.empty())`. / 引入条件分支：`if (FunctionBytes.empty())`。
- **L202**: Returns control, optionally with a value: `return make_error<StringError>("Empty function bytes",`. / 返回控制流，并可附带返回值：`return make_error<StringError>("Empty function bytes",`。
- **L203**: Executes a standalone statement or declaration: `errc::invalid_argument);`. / 执行一条独立语句或声明：`errc::invalid_argument);`。
- **L204**: Declares or invokes `>`. / 声明或调用 `>`。
- **L205**: Continues the surrounding expression or declaration: `const void *To = reinterpret_cast<const void *>(FunctionBytes.data() +`. / 继续构造周围的表达式或声明：`const void *To = reinterpret_cast<const void *>(FunctionBytes.data() +`。
- **L206**: Declares or invokes `FunctionBytes.size`. / 声明或调用 `FunctionBytes.size`。
- **L207**: Returns control, optionally with a value: `return doReadCounter(From, To);`. / 返回控制流，并可附带返回值：`return doReadCounter(From, To);`。
- **L208**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L209**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L210**: Continues the surrounding expression or declaration: `Expected<SmallVector<int64_t, 4>>`. / 继续构造周围的表达式或声明：`Expected<SmallVector<int64_t, 4>>`。
- **L211**: Starts the definition of function or method `X86LbrCounter::doReadCounter`. / 开始定义函数或方法 `X86LbrCounter::doReadCounter`。
- **L212**: Comment explains nearby logic or intent: `The max number of time-outs/retries before we give up.`. / 注释说明了附近代码的逻辑或设计意图：`The max number of time-outs/retries before we give up.`。
- **L213**: Initializes or updates `static constexpr int kMaxTimeouts` from the right-hand expression. / 使用右侧表达式初始化或更新 `static constexpr int kMaxTimeouts`。
- **L214**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L215**: Comment explains nearby logic or intent: `Parses the LBR buffer and fills CycleArray with the sequence of cycle`. / 注释说明了附近代码的逻辑或设计意图：`Parses the LBR buffer and fills CycleArray with the sequence of cycle`。
- **L216**: Comment explains nearby logic or intent: `counts from the buffer.`. / 注释说明了附近代码的逻辑或设计意图：`counts from the buffer.`。

### Lines 217-234

```cpp
  SmallVector<int64_t, 4> CycleArray;
  auto DataBuf = std::make_unique<char[]>(kDataBufferSize);
  int NumTimeouts = 0;
  int PollResult = 0;

  while (PollResult <= 0) {
    PollResult = pollLbrPerfEvent(getFileDescriptor());
    if (PollResult > 0)
      break;
    if (PollResult == -1)
      return make_error<StringError>("Cannot poll LBR perf event.",
                                     errc::io_error);
    if (NumTimeouts++ >= kMaxTimeouts)
      return make_error<StringError>(
          "LBR polling still timed out after max number of attempts.",
          errc::device_or_resource_busy);
  }

```

- **L217**: Executes a standalone statement or declaration: `SmallVector<int64_t, 4> CycleArray;`. / 执行一条独立语句或声明：`SmallVector<int64_t, 4> CycleArray;`。
- **L218**: Declares or invokes `std::make_unique<char[]>`. / 声明或调用 `std::make_unique<char[]>`。
- **L219**: Initializes or updates `int NumTimeouts` from the right-hand expression. / 使用右侧表达式初始化或更新 `int NumTimeouts`。
- **L220**: Initializes or updates `int PollResult` from the right-hand expression. / 使用右侧表达式初始化或更新 `int PollResult`。
- **L221**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L222**: Starts a while-loop guarded by a runtime condition: `while (PollResult <= 0) {`. / 开始由运行时条件控制的 while 循环：`while (PollResult <= 0) {`。
- **L223**: Declares or invokes `pollLbrPerfEvent`. / 声明或调用 `pollLbrPerfEvent`。
- **L224**: Introduces a conditional branch: `if (PollResult > 0)`. / 引入条件分支：`if (PollResult > 0)`。
- **L225**: Executes a standalone statement or declaration: `break;`. / 执行一条独立语句或声明：`break;`。
- **L226**: Introduces a conditional branch: `if (PollResult == -1)`. / 引入条件分支：`if (PollResult == -1)`。
- **L227**: Returns control, optionally with a value: `return make_error<StringError>("Cannot poll LBR perf event.",`. / 返回控制流，并可附带返回值：`return make_error<StringError>("Cannot poll LBR perf event.",`。
- **L228**: Executes a standalone statement or declaration: `errc::io_error);`. / 执行一条独立语句或声明：`errc::io_error);`。
- **L229**: Introduces a conditional branch: `if (NumTimeouts++ >= kMaxTimeouts)`. / 引入条件分支：`if (NumTimeouts++ >= kMaxTimeouts)`。
- **L230**: Returns control, optionally with a value: `return make_error<StringError>(`. / 返回控制流，并可附带返回值：`return make_error<StringError>(`。
- **L231**: Continues a multi-line argument list or initializer: `"LBR polling still timed out after max number of attempts.",`. / 继续一个多行参数列表或初始化器：`"LBR polling still timed out after max number of attempts.",`。
- **L232**: Executes a standalone statement or declaration: `errc::device_or_resource_busy);`. / 执行一条独立语句或声明：`errc::device_or_resource_busy);`。
- **L233**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L234**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。

### Lines 235-252

```cpp
  struct perf_event_mmap_page Page;
  memcpy(&Page, MMappedBuffer, sizeof(struct perf_event_mmap_page));

  const uint64_t DataTail = Page.data_tail;
  const uint64_t DataHead = Page.data_head;
  // We're supposed to use a barrier after reading data_head.
  std::atomic_thread_fence(std::memory_order_acq_rel);
  const size_t DataSize = DataHead - DataTail;
  if (DataSize > kDataBufferSize)
    return make_error<StringError>("DataSize larger than buffer size.",
                                   errc::invalid_argument);

  copyDataBuffer(MMappedBuffer, DataBuf.get(), DataTail, DataSize);
  Error error = parseDataBuffer(DataBuf.get(), DataSize, From, To, &CycleArray);
  if (!error)
    return CycleArray;
  return std::move(error);
}
```

- **L235**: Declares struct `Page;`. / 声明 struct `Page;`。
- **L236**: Declares or invokes `memcpy`. / 声明或调用 `memcpy`。
- **L237**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L238**: Initializes or updates `const uint64_t DataTail` from the right-hand expression. / 使用右侧表达式初始化或更新 `const uint64_t DataTail`。
- **L239**: Initializes or updates `const uint64_t DataHead` from the right-hand expression. / 使用右侧表达式初始化或更新 `const uint64_t DataHead`。
- **L240**: Comment explains nearby logic or intent: `We're supposed to use a barrier after reading data_head.`. / 注释说明了附近代码的逻辑或设计意图：`We're supposed to use a barrier after reading data_head.`。
- **L241**: Declares or invokes `std::atomic_thread_fence`. / 声明或调用 `std::atomic_thread_fence`。
- **L242**: Initializes or updates `const size_t DataSize` from the right-hand expression. / 使用右侧表达式初始化或更新 `const size_t DataSize`。
- **L243**: Introduces a conditional branch: `if (DataSize > kDataBufferSize)`. / 引入条件分支：`if (DataSize > kDataBufferSize)`。
- **L244**: Returns control, optionally with a value: `return make_error<StringError>("DataSize larger than buffer size.",`. / 返回控制流，并可附带返回值：`return make_error<StringError>("DataSize larger than buffer size.",`。
- **L245**: Executes a standalone statement or declaration: `errc::invalid_argument);`. / 执行一条独立语句或声明：`errc::invalid_argument);`。
- **L246**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L247**: Declares or invokes `copyDataBuffer`. / 声明或调用 `copyDataBuffer`。
- **L248**: Declares or invokes `parseDataBuffer`. / 声明或调用 `parseDataBuffer`。
- **L249**: Introduces a conditional branch: `if (!error)`. / 引入条件分支：`if (!error)`。
- **L250**: Returns control, optionally with a value: `return CycleArray;`. / 返回控制流，并可附带返回值：`return CycleArray;`。
- **L251**: Returns control, optionally with a value: `return std::move(error);`. / 返回控制流，并可附带返回值：`return std::move(error);`。
- **L252**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。

### Lines 253-258

```cpp

} // namespace exegesis
} // namespace llvm

#endif // defined(__linux__) && defined(HAVE_LIBPFM) &&
       // defined(LIBPFM_HAS_FIELD_CYCLES)
```

- **L253**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L254**: Closes a namespace scope with a trailing comment: `} // namespace exegesis`. / 结束一个带尾注释的命名空间作用域：`} // namespace exegesis`。
- **L255**: Closes a namespace scope with a trailing comment: `} // namespace llvm`. / 结束一个带尾注释的命名空间作用域：`} // namespace llvm`。
- **L256**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L257**: Preprocessor directive controls conditional compilation or build behavior: `#endif // defined(__linux__) && defined(HAVE_LIBPFM) &&`. / 预处理指令控制条件编译或构建行为：`#endif // defined(__linux__) && defined(HAVE_LIBPFM) &&`。
- **L258**: Comment explains nearby logic or intent: `defined(LIBPFM_HAS_FIELD_CYCLES)`. / 注释说明了附近代码的逻辑或设计意图：`defined(LIBPFM_HAS_FIELD_CYCLES)`。

## Key Concepts / 关键概念

- **Benchmarking and target-specific measurements / 基准测试与目标相关测量**
- **LLVM utility/container usage / LLVM 工具与容器使用**
- **Module/file-scoped coordination / 模块/文件级协调**
- **`X86Counter` focused implementation / 围绕 `X86Counter` 的实现逻辑**

## Dependencies / 依赖关系

- **Include / 包含** `X86Counter.h`: Provides local declarations paired with this implementation file. / 提供与该实现文件配套的本地声明。
- **Include / 包含** `llvm/Support/Endian.h`: Provides LLVM support-library facilities. / 提供LLVM 支持库设施。
- **Include / 包含** `llvm/Support/Errc.h`: Provides LLVM support-library facilities. / 提供LLVM 支持库设施。
- **Include / 包含** `perfmon/perf_event.h`: Provides local declarations paired with this implementation file. / 提供与该实现文件配套的本地声明。
- **Include / 包含** `perfmon/pfmlib.h`: Provides local declarations paired with this implementation file. / 提供与该实现文件配套的本地声明。
- **Include / 包含** `perfmon/pfmlib_perf_event.h`: Provides local declarations paired with this implementation file. / 提供与该实现文件配套的本地声明。
- **Include / 包含** `atomic`: Provides supporting declarations required by this file. / 提供本文件所需的辅助声明。
- **Include / 包含** `chrono`: Provides supporting declarations required by this file. / 提供本文件所需的辅助声明。
- **Include / 包含** `cstddef`: Provides supporting declarations required by this file. / 提供本文件所需的辅助声明。
- **Include / 包含** `cstdint`: Provides supporting declarations required by this file. / 提供本文件所需的辅助声明。
- **Include / 包含** `limits`: Provides supporting declarations required by this file. / 提供本文件所需的辅助声明。
- **Include / 包含** `memory`: Provides supporting declarations required by this file. / 提供本文件所需的辅助声明。
- **Include / 包含** `poll.h`: Provides local declarations paired with this implementation file. / 提供与该实现文件配套的本地声明。
- **Include / 包含** `sys/mman.h`: Provides local declarations paired with this implementation file. / 提供与该实现文件配套的本地声明。
- **Include / 包含** `unistd.h`: Provides local declarations paired with this implementation file. / 提供与该实现文件配套的本地声明。
