# CommandObjectTraceStartIntelPT.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `lldb/source/Plugins/Trace/intel-pt/CommandObjectTraceStartIntelPT.cpp`
- **Repository / 仓库**: `/root/xw/llvm-project/`
- **Purpose (EN)**: Implements LLDB logic for instruction tracing, trace decoding, packet interpretation, and trace-session plumbing related to `CommandObjectTraceStartIntelPT` in the `Trace` subsystem.
- **Purpose (CN)**: 该文件在 LLDB 的 `Trace` 子系统中实现与 `CommandObjectTraceStartIntelPT` 相关的逻辑，重点覆盖指令追踪、追踪解码、报文解释与追踪会话管线。对应英文说明：Implements LLDB logic for instruction tracing, trace decoding, packet interpretation, and trace-session plumbing related to `CommandObjectTraceStartIntelPT` in the `Trace` subsystem。

## Line-by-Line Analysis / 逐行分析

### Lines 1-20 / 第 1-20 行

````cpp
//===-- CommandObjectTraceStartIntelPT.cpp --------------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#include "CommandObjectTraceStartIntelPT.h"
#include "TraceIntelPT.h"
#include "TraceIntelPTConstants.h"
#include "lldb/Host/OptionParser.h"
#include "lldb/Interpreter/CommandOptionArgumentTable.h"
#include "lldb/Target/Process.h"
#include "lldb/Target/Trace.h"
#include <optional>

using namespace lldb;
using namespace lldb_private;
using namespace lldb_private::trace_intel_pt;
````
- **L1 EN**: Banner comment marks a file or section boundary.
  **L1 CN**: 横幅注释用于标记文件或章节边界。
- **L2 EN**: Separator comment visually groups nearby code.
  **L2 CN**: 分隔注释用于在视觉上分组附近代码。
- **L3 EN**: Comment explains surrounding design intent or invariants: `Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.`.
  **L3 CN**: 注释说明周边设计意图或不变式：`Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.`。
- **L4 EN**: Comment explains surrounding design intent or invariants: `See https://llvm.org/LICENSE.txt for license information.`.
  **L4 CN**: 注释说明周边设计意图或不变式：`See https://llvm.org/LICENSE.txt for license information.`。
- **L5 EN**: Comment explains surrounding design intent or invariants: `SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception`.
  **L5 CN**: 注释说明周边设计意图或不变式：`SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception`。
- **L6 EN**: Separator comment visually groups nearby code.
  **L6 CN**: 分隔注释用于在视觉上分组附近代码。
- **L7 EN**: Banner comment marks a file or section boundary.
  **L7 CN**: 横幅注释用于标记文件或章节边界。
- **L8 EN**: Blank line separates nearby declarations or logic blocks.
  **L8 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L9 EN**: Includes `CommandObjectTraceStartIntelPT.h` so this header can use supporting declarations from another header.
  **L9 CN**: 引入 `CommandObjectTraceStartIntelPT.h`，使该头文件能够使用来自其他头文件的辅助声明。
- **L10 EN**: Includes `TraceIntelPT.h` so this header can use supporting declarations from another header.
  **L10 CN**: 引入 `TraceIntelPT.h`，使该头文件能够使用来自其他头文件的辅助声明。
- **L11 EN**: Includes `TraceIntelPTConstants.h` so this header can use supporting declarations from another header.
  **L11 CN**: 引入 `TraceIntelPTConstants.h`，使该头文件能够使用来自其他头文件的辅助声明。
- **L12 EN**: Includes `lldb/Host/OptionParser.h` so this header can use host-facing services such as files, processes, threads, and sockets.
  **L12 CN**: 引入 `lldb/Host/OptionParser.h`，使该头文件能够使用面向宿主机的文件、进程、线程与套接字服务。
- **L13 EN**: Includes `lldb/Interpreter/CommandOptionArgumentTable.h` so this header can use command interpreter and option handling support.
  **L13 CN**: 引入 `lldb/Interpreter/CommandOptionArgumentTable.h`，使该头文件能够使用命令解释器与选项处理支持。
- **L14 EN**: Includes `lldb/Target/Process.h` so this header can use target/process/thread execution-control facilities.
  **L14 CN**: 引入 `lldb/Target/Process.h`，使该头文件能够使用目标/进程/线程执行控制设施。
- **L15 EN**: Includes `lldb/Target/Trace.h` so this header can use target/process/thread execution-control facilities.
  **L15 CN**: 引入 `lldb/Target/Trace.h`，使该头文件能够使用目标/进程/线程执行控制设施。
- **L16 EN**: Includes `optional` so this header can use standard-library or system facilities.
  **L16 CN**: 引入 `optional`，使该头文件能够使用标准库或系统设施。
- **L17 EN**: Blank line separates nearby declarations or logic blocks.
  **L17 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L18 EN**: Imports namespace `lldb` into the current scope.
  **L18 CN**: 将命名空间 `lldb` 导入当前作用域。
- **L19 EN**: Imports namespace `lldb_private` into the current scope.
  **L19 CN**: 将命名空间 `lldb_private` 导入当前作用域。
- **L20 EN**: Imports namespace `lldb_private::trace_intel_pt` into the current scope.
  **L20 CN**: 将命名空间 `lldb_private::trace_intel_pt` 导入当前作用域。

### Lines 21-40 / 第 21-40 行

````cpp
using namespace llvm;

// CommandObjectThreadTraceStartIntelPT

#define LLDB_OPTIONS_thread_trace_start_intel_pt
#include "TraceIntelPTCommandOptions.inc"

Status CommandObjectThreadTraceStartIntelPT::CommandOptions::SetOptionValue(
    uint32_t option_idx, llvm::StringRef option_arg,
    ExecutionContext *execution_context) {
  Status error;
  const int short_option = m_getopt_table[option_idx].val;

  switch (short_option) {
  case 's': {
    if (std::optional<uint64_t> bytes =
            ParsingUtils::ParseUserFriendlySizeExpression(option_arg))
      m_ipt_trace_size = *bytes;
    else
      error = Status::FromErrorStringWithFormat(
````
- **L21 EN**: Imports namespace `llvm` into the current scope.
  **L21 CN**: 将命名空间 `llvm` 导入当前作用域。
- **L22 EN**: Blank line separates nearby declarations or logic blocks.
  **L22 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L23 EN**: Comment explains surrounding design intent or invariants: `CommandObjectThreadTraceStartIntelPT`.
  **L23 CN**: 注释说明周边设计意图或不变式：`CommandObjectThreadTraceStartIntelPT`。
- **L24 EN**: Blank line separates nearby declarations or logic blocks.
  **L24 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L25 EN**: Defines macro `LLDB_OPTIONS_thread_trace_start_intel_pt` for include-guarding, feature control, or helper reuse.
  **L25 CN**: 定义宏 `LLDB_OPTIONS_thread_trace_start_intel_pt`，用于头文件保护、特性控制或辅助复用。
- **L26 EN**: Includes `TraceIntelPTCommandOptions.inc` so this header can use standard-library or system facilities.
  **L26 CN**: 引入 `TraceIntelPTCommandOptions.inc`，使该头文件能够使用标准库或系统设施。
- **L27 EN**: Blank line separates nearby declarations or logic blocks.
  **L27 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L28 EN**: Continues logic associated with callable symbol `SetOptionValue`.
  **L28 CN**: 继续与可调用符号 `SetOptionValue` 相关的逻辑。
- **L29 EN**: Continues a multi-line list, initializer, or aggregate entry: `uint32_t option_idx, llvm::StringRef option_arg,`.
  **L29 CN**: 继续一个多行列表、初始化器或聚合项：`uint32_t option_idx, llvm::StringRef option_arg,`。
- **L30 EN**: Continues the surrounding declaration or expression: `ExecutionContext *execution_context) {`.
  **L30 CN**: 继续构造周围的声明或表达式：`ExecutionContext *execution_context) {`。
- **L31 EN**: Completes a standalone declaration or statement: `Status error;`.
  **L31 CN**: 完成一条独立声明或语句：`Status error;`。
- **L32 EN**: Initializes or assigns variable `short_option` from the right-hand expression.
  **L32 CN**: 使用右侧表达式初始化或赋值变量 `short_option`。
- **L33 EN**: Blank line separates nearby declarations or logic blocks.
  **L33 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L34 EN**: Begins a `switch` control-flow statement.
  **L34 CN**: 开始一个 `switch` 控制流语句。
- **L35 EN**: Introduces a `switch` dispatch label: `case 's': {`.
  **L35 CN**: 引入一个 `switch` 分发标签：`case 's': {`。
- **L36 EN**: Begins a `if` control-flow statement.
  **L36 CN**: 开始一个 `if` 控制流语句。
- **L37 EN**: Continues logic associated with callable symbol `ParseUserFriendlySizeExpression`.
  **L37 CN**: 继续与可调用符号 `ParseUserFriendlySizeExpression` 相关的逻辑。
- **L38 EN**: Completes a standalone declaration or statement: `m_ipt_trace_size = *bytes;`.
  **L38 CN**: 完成一条独立声明或语句：`m_ipt_trace_size = *bytes;`。
- **L39 EN**: Begins the fallback branch of the preceding conditional.
  **L39 CN**: 开始前述条件语句的后备分支。
- **L40 EN**: Continues logic associated with callable symbol `FromErrorStringWithFormat`.
  **L40 CN**: 继续与可调用符号 `FromErrorStringWithFormat` 相关的逻辑。

### Lines 41-60 / 第 41-60 行

````cpp
          "invalid bytes expression for '%s'", option_arg.str().c_str());
    break;
  }
  case 't': {
    m_enable_tsc = true;
    break;
  }
  case 'p': {
    int64_t psb_period;
    if (option_arg.empty() || option_arg.getAsInteger(0, psb_period) ||
        psb_period < 0)
      error = Status::FromErrorStringWithFormat(
          "invalid integer value for option '%s'", option_arg.str().c_str());
    else
      m_psb_period = psb_period;
    break;
  }
  default:
    llvm_unreachable("Unimplemented option");
  }
````
- **L41 EN**: Declares or invokes callable logic centered on `option_arg.str`.
  **L41 CN**: 声明或调用以 `option_arg.str` 为核心的可调用逻辑。
- **L42 EN**: Exits the nearest loop or switch statement.
  **L42 CN**: 退出最近的循环或 switch 语句。
- **L43 EN**: Closes the current lexical scope or body.
  **L43 CN**: 关闭当前词法作用域或代码体。
- **L44 EN**: Introduces a `switch` dispatch label: `case 't': {`.
  **L44 CN**: 引入一个 `switch` 分发标签：`case 't': {`。
- **L45 EN**: Completes a standalone declaration or statement: `m_enable_tsc = true;`.
  **L45 CN**: 完成一条独立声明或语句：`m_enable_tsc = true;`。
- **L46 EN**: Exits the nearest loop or switch statement.
  **L46 CN**: 退出最近的循环或 switch 语句。
- **L47 EN**: Closes the current lexical scope or body.
  **L47 CN**: 关闭当前词法作用域或代码体。
- **L48 EN**: Introduces a `switch` dispatch label: `case 'p': {`.
  **L48 CN**: 引入一个 `switch` 分发标签：`case 'p': {`。
- **L49 EN**: Completes a standalone declaration or statement: `int64_t psb_period;`.
  **L49 CN**: 完成一条独立声明或语句：`int64_t psb_period;`。
- **L50 EN**: Begins a `if` control-flow statement.
  **L50 CN**: 开始一个 `if` 控制流语句。
- **L51 EN**: Continues the surrounding declaration or expression: `psb_period < 0)`.
  **L51 CN**: 继续构造周围的声明或表达式：`psb_period < 0)`。
- **L52 EN**: Continues logic associated with callable symbol `FromErrorStringWithFormat`.
  **L52 CN**: 继续与可调用符号 `FromErrorStringWithFormat` 相关的逻辑。
- **L53 EN**: Declares or invokes callable logic centered on `option_arg.str`.
  **L53 CN**: 声明或调用以 `option_arg.str` 为核心的可调用逻辑。
- **L54 EN**: Begins the fallback branch of the preceding conditional.
  **L54 CN**: 开始前述条件语句的后备分支。
- **L55 EN**: Completes a standalone declaration or statement: `m_psb_period = psb_period;`.
  **L55 CN**: 完成一条独立声明或语句：`m_psb_period = psb_period;`。
- **L56 EN**: Exits the nearest loop or switch statement.
  **L56 CN**: 退出最近的循环或 switch 语句。
- **L57 EN**: Closes the current lexical scope or body.
  **L57 CN**: 关闭当前词法作用域或代码体。
- **L58 EN**: Introduces a `switch` dispatch label: `default:`.
  **L58 CN**: 引入一个 `switch` 分发标签：`default:`。
- **L59 EN**: Marks the current control path as unreachable.
  **L59 CN**: 将当前控制路径标记为不可达。
- **L60 EN**: Closes the current lexical scope or body.
  **L60 CN**: 关闭当前词法作用域或代码体。

### Lines 61-80 / 第 61-80 行

````cpp
  return error;
}

void CommandObjectThreadTraceStartIntelPT::CommandOptions::
    OptionParsingStarting(ExecutionContext *execution_context) {
  m_ipt_trace_size = kDefaultIptTraceSize;
  m_enable_tsc = kDefaultEnableTscValue;
  m_psb_period = kDefaultPsbPeriod;
}

llvm::ArrayRef<OptionDefinition>
CommandObjectThreadTraceStartIntelPT::CommandOptions::GetDefinitions() {
  return llvm::ArrayRef(g_thread_trace_start_intel_pt_options);
}

bool CommandObjectThreadTraceStartIntelPT::DoExecuteOnThreads(
    Args &command, CommandReturnObject &result,
    llvm::ArrayRef<lldb::tid_t> tids) {
  if (Error err = m_trace.Start(tids, m_options.m_ipt_trace_size,
                                m_options.m_enable_tsc, m_options.m_psb_period))
````
- **L61 EN**: Returns from the current function with `error`.
  **L61 CN**: 以 `error` 从当前函数返回。
- **L62 EN**: Closes the current lexical scope or body.
  **L62 CN**: 关闭当前词法作用域或代码体。
- **L63 EN**: Blank line separates nearby declarations or logic blocks.
  **L63 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L64 EN**: Continues the surrounding declaration or expression: `void CommandObjectThreadTraceStartIntelPT::CommandOptions::`.
  **L64 CN**: 继续构造周围的声明或表达式：`void CommandObjectThreadTraceStartIntelPT::CommandOptions::`。
- **L65 EN**: Starts a function, method, lambda, or structured scope: `OptionParsingStarting(ExecutionContext *execution_context) {`.
  **L65 CN**: 开始一个函数、方法、lambda 或结构化作用域：`OptionParsingStarting(ExecutionContext *execution_context) {`。
- **L66 EN**: Completes a standalone declaration or statement: `m_ipt_trace_size = kDefaultIptTraceSize;`.
  **L66 CN**: 完成一条独立声明或语句：`m_ipt_trace_size = kDefaultIptTraceSize;`。
- **L67 EN**: Completes a standalone declaration or statement: `m_enable_tsc = kDefaultEnableTscValue;`.
  **L67 CN**: 完成一条独立声明或语句：`m_enable_tsc = kDefaultEnableTscValue;`。
- **L68 EN**: Completes a standalone declaration or statement: `m_psb_period = kDefaultPsbPeriod;`.
  **L68 CN**: 完成一条独立声明或语句：`m_psb_period = kDefaultPsbPeriod;`。
- **L69 EN**: Closes the current lexical scope or body.
  **L69 CN**: 关闭当前词法作用域或代码体。
- **L70 EN**: Blank line separates nearby declarations or logic blocks.
  **L70 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L71 EN**: Continues the surrounding declaration or expression: `llvm::ArrayRef<OptionDefinition>`.
  **L71 CN**: 继续构造周围的声明或表达式：`llvm::ArrayRef<OptionDefinition>`。
- **L72 EN**: Starts a function, method, lambda, or structured scope: `CommandObjectThreadTraceStartIntelPT::CommandOptions::GetDefinitions() {`.
  **L72 CN**: 开始一个函数、方法、lambda 或结构化作用域：`CommandObjectThreadTraceStartIntelPT::CommandOptions::GetDefinitions() {`。
- **L73 EN**: Returns from the current function with `llvm::ArrayRef(g_thread_trace_start_intel_pt_options)`.
  **L73 CN**: 以 `llvm::ArrayRef(g_thread_trace_start_intel_pt_options)` 从当前函数返回。
- **L74 EN**: Closes the current lexical scope or body.
  **L74 CN**: 关闭当前词法作用域或代码体。
- **L75 EN**: Blank line separates nearby declarations or logic blocks.
  **L75 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L76 EN**: Continues logic associated with callable symbol `DoExecuteOnThreads`.
  **L76 CN**: 继续与可调用符号 `DoExecuteOnThreads` 相关的逻辑。
- **L77 EN**: Continues a multi-line list, initializer, or aggregate entry: `Args &command, CommandReturnObject &result,`.
  **L77 CN**: 继续一个多行列表、初始化器或聚合项：`Args &command, CommandReturnObject &result,`。
- **L78 EN**: Continues the surrounding declaration or expression: `llvm::ArrayRef<lldb::tid_t> tids) {`.
  **L78 CN**: 继续构造周围的声明或表达式：`llvm::ArrayRef<lldb::tid_t> tids) {`。
- **L79 EN**: Begins a `if` control-flow statement.
  **L79 CN**: 开始一个 `if` 控制流语句。
- **L80 EN**: Continues the surrounding declaration or expression: `m_options.m_enable_tsc, m_options.m_psb_period))`.
  **L80 CN**: 继续构造周围的声明或表达式：`m_options.m_enable_tsc, m_options.m_psb_period))`。

### Lines 81-100 / 第 81-100 行

````cpp
    result.SetError(std::move(err));
  else
    result.SetStatus(eReturnStatusSuccessFinishResult);

  return result.Succeeded();
}

/// CommandObjectProcessTraceStartIntelPT

#define LLDB_OPTIONS_process_trace_start_intel_pt
#include "TraceIntelPTCommandOptions.inc"

Status CommandObjectProcessTraceStartIntelPT::CommandOptions::SetOptionValue(
    uint32_t option_idx, llvm::StringRef option_arg,
    ExecutionContext *execution_context) {
  Status error;
  const int short_option = m_getopt_table[option_idx].val;

  switch (short_option) {
  case 's': {
````
- **L81 EN**: Declares or invokes callable logic centered on `result.SetError`.
  **L81 CN**: 声明或调用以 `result.SetError` 为核心的可调用逻辑。
- **L82 EN**: Begins the fallback branch of the preceding conditional.
  **L82 CN**: 开始前述条件语句的后备分支。
- **L83 EN**: Declares or invokes callable logic centered on `result.SetStatus`.
  **L83 CN**: 声明或调用以 `result.SetStatus` 为核心的可调用逻辑。
- **L84 EN**: Blank line separates nearby declarations or logic blocks.
  **L84 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L85 EN**: Returns from the current function with `result.Succeeded()`.
  **L85 CN**: 以 `result.Succeeded()` 从当前函数返回。
- **L86 EN**: Closes the current lexical scope or body.
  **L86 CN**: 关闭当前词法作用域或代码体。
- **L87 EN**: Blank line separates nearby declarations or logic blocks.
  **L87 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L88 EN**: Doxygen comment documents API intent or semantics: `CommandObjectProcessTraceStartIntelPT`.
  **L88 CN**: Doxygen 注释记录 API 意图或语义：`CommandObjectProcessTraceStartIntelPT`。
- **L89 EN**: Blank line separates nearby declarations or logic blocks.
  **L89 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L90 EN**: Defines macro `LLDB_OPTIONS_process_trace_start_intel_pt` for include-guarding, feature control, or helper reuse.
  **L90 CN**: 定义宏 `LLDB_OPTIONS_process_trace_start_intel_pt`，用于头文件保护、特性控制或辅助复用。
- **L91 EN**: Includes `TraceIntelPTCommandOptions.inc` so this header can use standard-library or system facilities.
  **L91 CN**: 引入 `TraceIntelPTCommandOptions.inc`，使该头文件能够使用标准库或系统设施。
- **L92 EN**: Blank line separates nearby declarations or logic blocks.
  **L92 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L93 EN**: Continues logic associated with callable symbol `SetOptionValue`.
  **L93 CN**: 继续与可调用符号 `SetOptionValue` 相关的逻辑。
- **L94 EN**: Continues a multi-line list, initializer, or aggregate entry: `uint32_t option_idx, llvm::StringRef option_arg,`.
  **L94 CN**: 继续一个多行列表、初始化器或聚合项：`uint32_t option_idx, llvm::StringRef option_arg,`。
- **L95 EN**: Continues the surrounding declaration or expression: `ExecutionContext *execution_context) {`.
  **L95 CN**: 继续构造周围的声明或表达式：`ExecutionContext *execution_context) {`。
- **L96 EN**: Completes a standalone declaration or statement: `Status error;`.
  **L96 CN**: 完成一条独立声明或语句：`Status error;`。
- **L97 EN**: Initializes or assigns variable `short_option` from the right-hand expression.
  **L97 CN**: 使用右侧表达式初始化或赋值变量 `short_option`。
- **L98 EN**: Blank line separates nearby declarations or logic blocks.
  **L98 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L99 EN**: Begins a `switch` control-flow statement.
  **L99 CN**: 开始一个 `switch` 控制流语句。
- **L100 EN**: Introduces a `switch` dispatch label: `case 's': {`.
  **L100 CN**: 引入一个 `switch` 分发标签：`case 's': {`。

### Lines 101-120 / 第 101-120 行

````cpp
    if (std::optional<uint64_t> bytes =
            ParsingUtils::ParseUserFriendlySizeExpression(option_arg))
      m_ipt_trace_size = *bytes;
    else
      error = Status::FromErrorStringWithFormat(
          "invalid bytes expression for '%s'", option_arg.str().c_str());
    break;
  }
  case 'l': {
    if (std::optional<uint64_t> bytes =
            ParsingUtils::ParseUserFriendlySizeExpression(option_arg))
      m_process_buffer_size_limit = *bytes;
    else
      error = Status::FromErrorStringWithFormat(
          "invalid bytes expression for '%s'", option_arg.str().c_str());
    break;
  }
  case 't': {
    m_enable_tsc = true;
    break;
````
- **L101 EN**: Begins a `if` control-flow statement.
  **L101 CN**: 开始一个 `if` 控制流语句。
- **L102 EN**: Continues logic associated with callable symbol `ParseUserFriendlySizeExpression`.
  **L102 CN**: 继续与可调用符号 `ParseUserFriendlySizeExpression` 相关的逻辑。
- **L103 EN**: Completes a standalone declaration or statement: `m_ipt_trace_size = *bytes;`.
  **L103 CN**: 完成一条独立声明或语句：`m_ipt_trace_size = *bytes;`。
- **L104 EN**: Begins the fallback branch of the preceding conditional.
  **L104 CN**: 开始前述条件语句的后备分支。
- **L105 EN**: Continues logic associated with callable symbol `FromErrorStringWithFormat`.
  **L105 CN**: 继续与可调用符号 `FromErrorStringWithFormat` 相关的逻辑。
- **L106 EN**: Declares or invokes callable logic centered on `option_arg.str`.
  **L106 CN**: 声明或调用以 `option_arg.str` 为核心的可调用逻辑。
- **L107 EN**: Exits the nearest loop or switch statement.
  **L107 CN**: 退出最近的循环或 switch 语句。
- **L108 EN**: Closes the current lexical scope or body.
  **L108 CN**: 关闭当前词法作用域或代码体。
- **L109 EN**: Introduces a `switch` dispatch label: `case 'l': {`.
  **L109 CN**: 引入一个 `switch` 分发标签：`case 'l': {`。
- **L110 EN**: Begins a `if` control-flow statement.
  **L110 CN**: 开始一个 `if` 控制流语句。
- **L111 EN**: Continues logic associated with callable symbol `ParseUserFriendlySizeExpression`.
  **L111 CN**: 继续与可调用符号 `ParseUserFriendlySizeExpression` 相关的逻辑。
- **L112 EN**: Completes a standalone declaration or statement: `m_process_buffer_size_limit = *bytes;`.
  **L112 CN**: 完成一条独立声明或语句：`m_process_buffer_size_limit = *bytes;`。
- **L113 EN**: Begins the fallback branch of the preceding conditional.
  **L113 CN**: 开始前述条件语句的后备分支。
- **L114 EN**: Continues logic associated with callable symbol `FromErrorStringWithFormat`.
  **L114 CN**: 继续与可调用符号 `FromErrorStringWithFormat` 相关的逻辑。
- **L115 EN**: Declares or invokes callable logic centered on `option_arg.str`.
  **L115 CN**: 声明或调用以 `option_arg.str` 为核心的可调用逻辑。
- **L116 EN**: Exits the nearest loop or switch statement.
  **L116 CN**: 退出最近的循环或 switch 语句。
- **L117 EN**: Closes the current lexical scope or body.
  **L117 CN**: 关闭当前词法作用域或代码体。
- **L118 EN**: Introduces a `switch` dispatch label: `case 't': {`.
  **L118 CN**: 引入一个 `switch` 分发标签：`case 't': {`。
- **L119 EN**: Completes a standalone declaration or statement: `m_enable_tsc = true;`.
  **L119 CN**: 完成一条独立声明或语句：`m_enable_tsc = true;`。
- **L120 EN**: Exits the nearest loop or switch statement.
  **L120 CN**: 退出最近的循环或 switch 语句。

### Lines 121-140 / 第 121-140 行

````cpp
  }
  case 'c': {
    m_per_cpu_tracing = true;
    break;
  }
  case 'd': {
    m_disable_cgroup_filtering = true;
    break;
  }
  case 'p': {
    int64_t psb_period;
    if (option_arg.empty() || option_arg.getAsInteger(0, psb_period) ||
        psb_period < 0)
      error = Status::FromErrorStringWithFormat(
          "invalid integer value for option '%s'", option_arg.str().c_str());
    else
      m_psb_period = psb_period;
    break;
  }
  default:
````
- **L121 EN**: Closes the current lexical scope or body.
  **L121 CN**: 关闭当前词法作用域或代码体。
- **L122 EN**: Introduces a `switch` dispatch label: `case 'c': {`.
  **L122 CN**: 引入一个 `switch` 分发标签：`case 'c': {`。
- **L123 EN**: Completes a standalone declaration or statement: `m_per_cpu_tracing = true;`.
  **L123 CN**: 完成一条独立声明或语句：`m_per_cpu_tracing = true;`。
- **L124 EN**: Exits the nearest loop or switch statement.
  **L124 CN**: 退出最近的循环或 switch 语句。
- **L125 EN**: Closes the current lexical scope or body.
  **L125 CN**: 关闭当前词法作用域或代码体。
- **L126 EN**: Introduces a `switch` dispatch label: `case 'd': {`.
  **L126 CN**: 引入一个 `switch` 分发标签：`case 'd': {`。
- **L127 EN**: Completes a standalone declaration or statement: `m_disable_cgroup_filtering = true;`.
  **L127 CN**: 完成一条独立声明或语句：`m_disable_cgroup_filtering = true;`。
- **L128 EN**: Exits the nearest loop or switch statement.
  **L128 CN**: 退出最近的循环或 switch 语句。
- **L129 EN**: Closes the current lexical scope or body.
  **L129 CN**: 关闭当前词法作用域或代码体。
- **L130 EN**: Introduces a `switch` dispatch label: `case 'p': {`.
  **L130 CN**: 引入一个 `switch` 分发标签：`case 'p': {`。
- **L131 EN**: Completes a standalone declaration or statement: `int64_t psb_period;`.
  **L131 CN**: 完成一条独立声明或语句：`int64_t psb_period;`。
- **L132 EN**: Begins a `if` control-flow statement.
  **L132 CN**: 开始一个 `if` 控制流语句。
- **L133 EN**: Continues the surrounding declaration or expression: `psb_period < 0)`.
  **L133 CN**: 继续构造周围的声明或表达式：`psb_period < 0)`。
- **L134 EN**: Continues logic associated with callable symbol `FromErrorStringWithFormat`.
  **L134 CN**: 继续与可调用符号 `FromErrorStringWithFormat` 相关的逻辑。
- **L135 EN**: Declares or invokes callable logic centered on `option_arg.str`.
  **L135 CN**: 声明或调用以 `option_arg.str` 为核心的可调用逻辑。
- **L136 EN**: Begins the fallback branch of the preceding conditional.
  **L136 CN**: 开始前述条件语句的后备分支。
- **L137 EN**: Completes a standalone declaration or statement: `m_psb_period = psb_period;`.
  **L137 CN**: 完成一条独立声明或语句：`m_psb_period = psb_period;`。
- **L138 EN**: Exits the nearest loop or switch statement.
  **L138 CN**: 退出最近的循环或 switch 语句。
- **L139 EN**: Closes the current lexical scope or body.
  **L139 CN**: 关闭当前词法作用域或代码体。
- **L140 EN**: Introduces a `switch` dispatch label: `default:`.
  **L140 CN**: 引入一个 `switch` 分发标签：`default:`。

### Lines 141-160 / 第 141-160 行

````cpp
    llvm_unreachable("Unimplemented option");
  }
  return error;
}

void CommandObjectProcessTraceStartIntelPT::CommandOptions::
    OptionParsingStarting(ExecutionContext *execution_context) {
  m_ipt_trace_size = kDefaultIptTraceSize;
  m_process_buffer_size_limit = kDefaultProcessBufferSizeLimit;
  m_enable_tsc = kDefaultEnableTscValue;
  m_psb_period = kDefaultPsbPeriod;
  m_per_cpu_tracing = kDefaultPerCpuTracing;
  m_disable_cgroup_filtering = kDefaultDisableCgroupFiltering;
}

llvm::ArrayRef<OptionDefinition>
CommandObjectProcessTraceStartIntelPT::CommandOptions::GetDefinitions() {
  return llvm::ArrayRef(g_process_trace_start_intel_pt_options);
}

````
- **L141 EN**: Marks the current control path as unreachable.
  **L141 CN**: 将当前控制路径标记为不可达。
- **L142 EN**: Closes the current lexical scope or body.
  **L142 CN**: 关闭当前词法作用域或代码体。
- **L143 EN**: Returns from the current function with `error`.
  **L143 CN**: 以 `error` 从当前函数返回。
- **L144 EN**: Closes the current lexical scope or body.
  **L144 CN**: 关闭当前词法作用域或代码体。
- **L145 EN**: Blank line separates nearby declarations or logic blocks.
  **L145 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L146 EN**: Continues the surrounding declaration or expression: `void CommandObjectProcessTraceStartIntelPT::CommandOptions::`.
  **L146 CN**: 继续构造周围的声明或表达式：`void CommandObjectProcessTraceStartIntelPT::CommandOptions::`。
- **L147 EN**: Starts a function, method, lambda, or structured scope: `OptionParsingStarting(ExecutionContext *execution_context) {`.
  **L147 CN**: 开始一个函数、方法、lambda 或结构化作用域：`OptionParsingStarting(ExecutionContext *execution_context) {`。
- **L148 EN**: Completes a standalone declaration or statement: `m_ipt_trace_size = kDefaultIptTraceSize;`.
  **L148 CN**: 完成一条独立声明或语句：`m_ipt_trace_size = kDefaultIptTraceSize;`。
- **L149 EN**: Completes a standalone declaration or statement: `m_process_buffer_size_limit = kDefaultProcessBufferSizeLimit;`.
  **L149 CN**: 完成一条独立声明或语句：`m_process_buffer_size_limit = kDefaultProcessBufferSizeLimit;`。
- **L150 EN**: Completes a standalone declaration or statement: `m_enable_tsc = kDefaultEnableTscValue;`.
  **L150 CN**: 完成一条独立声明或语句：`m_enable_tsc = kDefaultEnableTscValue;`。
- **L151 EN**: Completes a standalone declaration or statement: `m_psb_period = kDefaultPsbPeriod;`.
  **L151 CN**: 完成一条独立声明或语句：`m_psb_period = kDefaultPsbPeriod;`。
- **L152 EN**: Completes a standalone declaration or statement: `m_per_cpu_tracing = kDefaultPerCpuTracing;`.
  **L152 CN**: 完成一条独立声明或语句：`m_per_cpu_tracing = kDefaultPerCpuTracing;`。
- **L153 EN**: Completes a standalone declaration or statement: `m_disable_cgroup_filtering = kDefaultDisableCgroupFiltering;`.
  **L153 CN**: 完成一条独立声明或语句：`m_disable_cgroup_filtering = kDefaultDisableCgroupFiltering;`。
- **L154 EN**: Closes the current lexical scope or body.
  **L154 CN**: 关闭当前词法作用域或代码体。
- **L155 EN**: Blank line separates nearby declarations or logic blocks.
  **L155 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L156 EN**: Continues the surrounding declaration or expression: `llvm::ArrayRef<OptionDefinition>`.
  **L156 CN**: 继续构造周围的声明或表达式：`llvm::ArrayRef<OptionDefinition>`。
- **L157 EN**: Starts a function, method, lambda, or structured scope: `CommandObjectProcessTraceStartIntelPT::CommandOptions::GetDefinitions() {`.
  **L157 CN**: 开始一个函数、方法、lambda 或结构化作用域：`CommandObjectProcessTraceStartIntelPT::CommandOptions::GetDefinitions() {`。
- **L158 EN**: Returns from the current function with `llvm::ArrayRef(g_process_trace_start_intel_pt_options)`.
  **L158 CN**: 以 `llvm::ArrayRef(g_process_trace_start_intel_pt_options)` 从当前函数返回。
- **L159 EN**: Closes the current lexical scope or body.
  **L159 CN**: 关闭当前词法作用域或代码体。
- **L160 EN**: Blank line separates nearby declarations or logic blocks.
  **L160 CN**: 空行用于分隔相邻的声明或逻辑块。

### Lines 161-180 / 第 161-180 行

````cpp
void CommandObjectProcessTraceStartIntelPT::DoExecute(
    Args &command, CommandReturnObject &result) {
  if (Error err = m_trace.Start(
          m_options.m_ipt_trace_size, m_options.m_process_buffer_size_limit,
          m_options.m_enable_tsc, m_options.m_psb_period,
          m_options.m_per_cpu_tracing, m_options.m_disable_cgroup_filtering))
    result.SetError(std::move(err));
  else
    result.SetStatus(eReturnStatusSuccessFinishResult);
}

std::optional<uint64_t>
ParsingUtils::ParseUserFriendlySizeExpression(llvm::StringRef size_expression) {
  if (size_expression.empty()) {
    return std::nullopt;
  }
  const uint64_t kBytesMultiplier = 1;
  const uint64_t kKibiBytesMultiplier = 1024;
  const uint64_t kMebiBytesMultiplier = 1024 * 1024;

````
- **L161 EN**: Continues logic associated with callable symbol `DoExecute`.
  **L161 CN**: 继续与可调用符号 `DoExecute` 相关的逻辑。
- **L162 EN**: Continues the surrounding declaration or expression: `Args &command, CommandReturnObject &result) {`.
  **L162 CN**: 继续构造周围的声明或表达式：`Args &command, CommandReturnObject &result) {`。
- **L163 EN**: Begins a `if` control-flow statement.
  **L163 CN**: 开始一个 `if` 控制流语句。
- **L164 EN**: Continues a multi-line list, initializer, or aggregate entry: `m_options.m_ipt_trace_size, m_options.m_process_buffer_size_limit,`.
  **L164 CN**: 继续一个多行列表、初始化器或聚合项：`m_options.m_ipt_trace_size, m_options.m_process_buffer_size_limit,`。
- **L165 EN**: Continues a multi-line list, initializer, or aggregate entry: `m_options.m_enable_tsc, m_options.m_psb_period,`.
  **L165 CN**: 继续一个多行列表、初始化器或聚合项：`m_options.m_enable_tsc, m_options.m_psb_period,`。
- **L166 EN**: Continues the surrounding declaration or expression: `m_options.m_per_cpu_tracing, m_options.m_disable_cgroup_filtering))`.
  **L166 CN**: 继续构造周围的声明或表达式：`m_options.m_per_cpu_tracing, m_options.m_disable_cgroup_filtering))`。
- **L167 EN**: Declares or invokes callable logic centered on `result.SetError`.
  **L167 CN**: 声明或调用以 `result.SetError` 为核心的可调用逻辑。
- **L168 EN**: Begins the fallback branch of the preceding conditional.
  **L168 CN**: 开始前述条件语句的后备分支。
- **L169 EN**: Declares or invokes callable logic centered on `result.SetStatus`.
  **L169 CN**: 声明或调用以 `result.SetStatus` 为核心的可调用逻辑。
- **L170 EN**: Closes the current lexical scope or body.
  **L170 CN**: 关闭当前词法作用域或代码体。
- **L171 EN**: Blank line separates nearby declarations or logic blocks.
  **L171 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L172 EN**: Continues the surrounding declaration or expression: `std::optional<uint64_t>`.
  **L172 CN**: 继续构造周围的声明或表达式：`std::optional<uint64_t>`。
- **L173 EN**: Starts a function, method, lambda, or structured scope: `ParsingUtils::ParseUserFriendlySizeExpression(llvm::StringRef size_expression) {`.
  **L173 CN**: 开始一个函数、方法、lambda 或结构化作用域：`ParsingUtils::ParseUserFriendlySizeExpression(llvm::StringRef size_expression) {`。
- **L174 EN**: Begins a `if` control-flow statement.
  **L174 CN**: 开始一个 `if` 控制流语句。
- **L175 EN**: Returns from the current function with `std::nullopt`.
  **L175 CN**: 以 `std::nullopt` 从当前函数返回。
- **L176 EN**: Closes the current lexical scope or body.
  **L176 CN**: 关闭当前词法作用域或代码体。
- **L177 EN**: Initializes or assigns variable `kBytesMultiplier` from the right-hand expression.
  **L177 CN**: 使用右侧表达式初始化或赋值变量 `kBytesMultiplier`。
- **L178 EN**: Initializes or assigns variable `kKibiBytesMultiplier` from the right-hand expression.
  **L178 CN**: 使用右侧表达式初始化或赋值变量 `kKibiBytesMultiplier`。
- **L179 EN**: Initializes or assigns variable `kMebiBytesMultiplier` from the right-hand expression.
  **L179 CN**: 使用右侧表达式初始化或赋值变量 `kMebiBytesMultiplier`。
- **L180 EN**: Blank line separates nearby declarations or logic blocks.
  **L180 CN**: 空行用于分隔相邻的声明或逻辑块。

### Lines 181-200 / 第 181-200 行

````cpp
  DenseMap<StringRef, uint64_t> multipliers = {
      {"mib", kMebiBytesMultiplier}, {"mb", kMebiBytesMultiplier},
      {"m", kMebiBytesMultiplier},   {"kib", kKibiBytesMultiplier},
      {"kb", kKibiBytesMultiplier},  {"k", kKibiBytesMultiplier},
      {"b", kBytesMultiplier},       {"", kBytesMultiplier}};

  const auto non_digit_index = size_expression.find_first_not_of("0123456789");
  if (non_digit_index == 0) { // expression starts from from non-digit char.
    return std::nullopt;
  }

  const llvm::StringRef number_part =
      non_digit_index == llvm::StringRef::npos
          ? size_expression
          : size_expression.substr(0, non_digit_index);
  uint64_t parsed_number;
  if (number_part.getAsInteger(10, parsed_number)) {
    return std::nullopt;
  }

````
- **L181 EN**: Continues the surrounding declaration or expression: `DenseMap<StringRef, uint64_t> multipliers = {`.
  **L181 CN**: 继续构造周围的声明或表达式：`DenseMap<StringRef, uint64_t> multipliers = {`。
- **L182 EN**: Continues a multi-line list, initializer, or aggregate entry: `{"mib", kMebiBytesMultiplier}, {"mb", kMebiBytesMultiplier},`.
  **L182 CN**: 继续一个多行列表、初始化器或聚合项：`{"mib", kMebiBytesMultiplier}, {"mb", kMebiBytesMultiplier},`。
- **L183 EN**: Continues a multi-line list, initializer, or aggregate entry: `{"m", kMebiBytesMultiplier},   {"kib", kKibiBytesMultiplier},`.
  **L183 CN**: 继续一个多行列表、初始化器或聚合项：`{"m", kMebiBytesMultiplier},   {"kib", kKibiBytesMultiplier},`。
- **L184 EN**: Continues a multi-line list, initializer, or aggregate entry: `{"kb", kKibiBytesMultiplier},  {"k", kKibiBytesMultiplier},`.
  **L184 CN**: 继续一个多行列表、初始化器或聚合项：`{"kb", kKibiBytesMultiplier},  {"k", kKibiBytesMultiplier},`。
- **L185 EN**: Completes a standalone declaration or statement: `{"b", kBytesMultiplier},       {"", kBytesMultiplier}};`.
  **L185 CN**: 完成一条独立声明或语句：`{"b", kBytesMultiplier},       {"", kBytesMultiplier}};`。
- **L186 EN**: Blank line separates nearby declarations or logic blocks.
  **L186 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L187 EN**: Initializes or assigns variable `non_digit_index` from the right-hand expression.
  **L187 CN**: 使用右侧表达式初始化或赋值变量 `non_digit_index`。
- **L188 EN**: Begins a `if` control-flow statement.
  **L188 CN**: 开始一个 `if` 控制流语句。
- **L189 EN**: Returns from the current function with `std::nullopt`.
  **L189 CN**: 以 `std::nullopt` 从当前函数返回。
- **L190 EN**: Closes the current lexical scope or body.
  **L190 CN**: 关闭当前词法作用域或代码体。
- **L191 EN**: Blank line separates nearby declarations or logic blocks.
  **L191 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L192 EN**: Continues the surrounding declaration or expression: `const llvm::StringRef number_part =`.
  **L192 CN**: 继续构造周围的声明或表达式：`const llvm::StringRef number_part =`。
- **L193 EN**: Continues the surrounding declaration or expression: `non_digit_index == llvm::StringRef::npos`.
  **L193 CN**: 继续构造周围的声明或表达式：`non_digit_index == llvm::StringRef::npos`。
- **L194 EN**: Continues the surrounding declaration or expression: `? size_expression`.
  **L194 CN**: 继续构造周围的声明或表达式：`? size_expression`。
- **L195 EN**: Declares or invokes callable logic centered on `size_expression.substr`.
  **L195 CN**: 声明或调用以 `size_expression.substr` 为核心的可调用逻辑。
- **L196 EN**: Completes a standalone declaration or statement: `uint64_t parsed_number;`.
  **L196 CN**: 完成一条独立声明或语句：`uint64_t parsed_number;`。
- **L197 EN**: Begins a `if` control-flow statement.
  **L197 CN**: 开始一个 `if` 控制流语句。
- **L198 EN**: Returns from the current function with `std::nullopt`.
  **L198 CN**: 以 `std::nullopt` 从当前函数返回。
- **L199 EN**: Closes the current lexical scope or body.
  **L199 CN**: 关闭当前词法作用域或代码体。
- **L200 EN**: Blank line separates nearby declarations or logic blocks.
  **L200 CN**: 空行用于分隔相邻的声明或逻辑块。

### Lines 201-212 / 第 201-212 行

````cpp
  if (non_digit_index != llvm::StringRef::npos) { // if expression has units.
    const auto multiplier = size_expression.substr(non_digit_index).lower();

    auto it = multipliers.find(multiplier);
    if (it == multipliers.end())
      return std::nullopt;

    return parsed_number * it->second;
  } else {
    return parsed_number;
  }
}
````
- **L201 EN**: Begins a `if` control-flow statement.
  **L201 CN**: 开始一个 `if` 控制流语句。
- **L202 EN**: Initializes or assigns variable `multiplier` from the right-hand expression.
  **L202 CN**: 使用右侧表达式初始化或赋值变量 `multiplier`。
- **L203 EN**: Blank line separates nearby declarations or logic blocks.
  **L203 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L204 EN**: Initializes or assigns variable `it` from the right-hand expression.
  **L204 CN**: 使用右侧表达式初始化或赋值变量 `it`。
- **L205 EN**: Begins a `if` control-flow statement.
  **L205 CN**: 开始一个 `if` 控制流语句。
- **L206 EN**: Returns from the current function with `std::nullopt`.
  **L206 CN**: 以 `std::nullopt` 从当前函数返回。
- **L207 EN**: Blank line separates nearby declarations or logic blocks.
  **L207 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L208 EN**: Returns from the current function with `parsed_number * it->second`.
  **L208 CN**: 以 `parsed_number * it->second` 从当前函数返回。
- **L209 EN**: Continues the surrounding declaration or expression: `} else {`.
  **L209 CN**: 继续构造周围的声明或表达式：`} else {`。
- **L210 EN**: Returns from the current function with `parsed_number`.
  **L210 CN**: 以 `parsed_number` 从当前函数返回。
- **L211 EN**: Closes the current lexical scope or body.
  **L211 CN**: 关闭当前词法作用域或代码体。
- **L212 EN**: Closes the current lexical scope or body.
  **L212 CN**: 关闭当前词法作用域或代码体。

## Key Concepts / 关键概念

- **Role / 角色**: This file is a implementation unit inside LLDB's **Trace** area. / 该文件是 LLDB **Trace** 范围内的实现文件。
- **Scale / 规模**: 212 lines with 9 direct includes. / 共 212 行，直接包含 9 个头文件。
- **Subsystem focus / 子系统关注点**: trace packet decoding, instruction history reconstruction, trace-session management. / 追踪报文解码、指令历史重建、追踪会话管理。
- **Visible entry points / 关键入口**: `str`, `llvm_unreachable`, `OptionParsingStarting`, `CommandObjectThreadTraceStartIntelPT::CommandOptions::GetDefinitions`, `llvm::ArrayRef`, `SetError`, `SetStatus`, `Succeeded`, `CommandObjectProcessTraceStartIntelPT::CommandOptions::GetDefinitions`, `ParsingUtils::ParseUserFriendlySizeExpression`. / 可见的关键入口包括 `str`, `llvm_unreachable`, `OptionParsingStarting`, `CommandObjectThreadTraceStartIntelPT::CommandOptions::GetDefinitions`, `llvm::ArrayRef`, `SetError`, `SetStatus`, `Succeeded`, `CommandObjectProcessTraceStartIntelPT::CommandOptions::GetDefinitions`, `ParsingUtils::ParseUserFriendlySizeExpression`。
- **Macros / 宏**: `LLDB_OPTIONS_thread_trace_start_intel_pt`, `LLDB_OPTIONS_process_trace_start_intel_pt`. / 关键宏包括 `LLDB_OPTIONS_thread_trace_start_intel_pt`, `LLDB_OPTIONS_process_trace_start_intel_pt`。
- **Concept / 概念**: Argument vector handling. / 参数向量处理。
- **Concept / 概念**: Command object dispatch. / 命令对象分发。
- **Concept / 概念**: LLVM error propagation. / LLVM 错误传播。
- **Concept / 概念**: Typed option value storage. / 强类型选项值存储。

## Dependencies / 依赖关系

- **LLDB headers / LLDB 头文件**: `lldb/Host/OptionParser.h`, `lldb/Interpreter/CommandOptionArgumentTable.h`, `lldb/Target/Process.h`, `lldb/Target/Trace.h`.
- **System/other headers / 系统或其他头文件**: `CommandObjectTraceStartIntelPT.h`, `TraceIntelPT.h`, `TraceIntelPTConstants.h`, `optional`, `TraceIntelPTCommandOptions.inc`.
- **Callable interfaces / 可调用接口**: `str`, `llvm_unreachable`, `OptionParsingStarting`, `CommandObjectThreadTraceStartIntelPT::CommandOptions::GetDefinitions`, `llvm::ArrayRef`, `SetError`, `SetStatus`, `Succeeded`, `CommandObjectProcessTraceStartIntelPT::CommandOptions::GetDefinitions`, `ParsingUtils::ParseUserFriendlySizeExpression`.
