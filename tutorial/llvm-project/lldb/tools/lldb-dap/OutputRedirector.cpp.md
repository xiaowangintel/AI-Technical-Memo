# OutputRedirector.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `lldb/tools/lldb-dap/OutputRedirector.cpp`
- **Repository / 仓库**: `llvm-project` (`/root/xw/llvm-project`)
- **Purpose / 目的**:
  - **EN**: Implements the LLDB support logic associated with `OutputRedirector`.
  - **CN**: 实现与 `OutputRedirector` 相关的 LLDB 支持逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-12 / 第 1-12 行

```cpp
 1 | //===-- OutputRedirector.cpp -----------------------------------*- C++ -*-===//
 2 | //
 3 | // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
 4 | // See https://llvm.org/LICENSE.txt for license information.
 5 | // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
 6 | //
 7 | //===----------------------------------------------------------------------===/
 8 | 
 9 | #include "OutputRedirector.h"
10 | #include "DAP.h"
11 | #include "llvm/ADT/StringRef.h"
12 | #include "llvm/Support/Error.h"
```

- **L1**: Banner comment marking a file or section boundary. / 横幅注释，用于标记文件或章节边界。
- **L2**: Separator comment used for visual grouping. / 用于视觉分组的分隔注释。
- **L3**: Comment explains nearby logic, invariants, or intent: `Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.`. / 注释说明了附近代码的逻辑、不变式或设计意图：`Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.`。
- **L4**: Comment explains nearby logic, invariants, or intent: `See https://llvm.org/LICENSE.txt for license information.`. / 注释说明了附近代码的逻辑、不变式或设计意图：`See https://llvm.org/LICENSE.txt for license information.`。
- **L5**: Comment explains nearby logic, invariants, or intent: `SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception`. / 注释说明了附近代码的逻辑、不变式或设计意图：`SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception`。
- **L6**: Separator comment used for visual grouping. / 用于视觉分组的分隔注释。
- **L7**: Banner comment marking a file or section boundary. / 横幅注释，用于标记文件或章节边界。
- **L8**: Blank line separating nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L9**: Includes "OutputRedirector.h" to access local declarations used by this file. / 引入 "OutputRedirector.h" 以使用本文件使用的本地声明。
- **L10**: Includes "DAP.h" to access local declarations used by this file. / 引入 "DAP.h" 以使用本文件使用的本地声明。
- **L11**: Includes "llvm/ADT/StringRef.h" to access LLVM ADT containers and utility types. / 引入 "llvm/ADT/StringRef.h" 以使用LLVM ADT 容器与工具类型。
- **L12**: Includes "llvm/Support/Error.h" to access LLVM support-library facilities. / 引入 "llvm/Support/Error.h" 以使用LLVM Support 库设施。

### Lines 13-24 / 第 13-24 行

```cpp
13 | #include <cstring>
14 | #include <system_error>
15 | #if defined(_WIN32)
16 | #include <fcntl.h>
17 | #include <io.h>
18 | #else
19 | #include <unistd.h>
20 | #endif
21 | 
22 | using namespace llvm;
23 | 
24 | static constexpr auto kCloseSentinel = StringLiteral::withInnerNUL("\0");
```

- **L13**: Includes <cstring> to access supporting declarations used by the current translation unit. / 引入 <cstring> 以使用当前编译单元使用的辅助声明。
- **L14**: Includes <system_error> to access supporting declarations used by the current translation unit. / 引入 <system_error> 以使用当前编译单元使用的辅助声明。
- **L15**: Starts a preprocessor conditional block: `#if defined(_WIN32)`. / 开始一个预处理条件块：`#if defined(_WIN32)`。
- **L16**: Includes <fcntl.h> to access local declarations used by this file. / 引入 <fcntl.h> 以使用本文件使用的本地声明。
- **L17**: Includes <io.h> to access local declarations used by this file. / 引入 <io.h> 以使用本文件使用的本地声明。
- **L18**: Continues the active preprocessor branch selection. / 继续当前的预处理分支选择。
- **L19**: Includes <unistd.h> to access local declarations used by this file. / 引入 <unistd.h> 以使用本文件使用的本地声明。
- **L20**: Closes the current preprocessor conditional block. / 结束当前预处理条件块。
- **L21**: Blank line separating nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L22**: Brings namespace `llvm` into the local scope. / 将命名空间 `llvm` 引入当前作用域。
- **L23**: Blank line separating nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L24**: Initializes variable `kCloseSentinel` from the right-hand expression. / 使用右侧表达式初始化变量 `kCloseSentinel`。

### Lines 25-36 / 第 25-36 行

```cpp
25 | 
26 | namespace lldb_dap {
27 | 
28 | int OutputRedirector::kInvalidDescriptor = -1;
29 | 
30 | OutputRedirector::OutputRedirector()
31 |     : m_fd(kInvalidDescriptor), m_original_fd(kInvalidDescriptor),
32 |       m_restore_fd(kInvalidDescriptor) {}
33 | 
34 | Expected<int> OutputRedirector::GetWriteFileDescriptor() {
35 |   if (m_fd == kInvalidDescriptor)
36 |     return createStringError(std::errc::bad_file_descriptor,
```

- **L25**: Blank line separating nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L26**: Opens namespace scope `lldb_dap`. / 打开命名空间作用域 `lldb_dap`。
- **L27**: Blank line separating nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L28**: Executes a standalone statement or declaration: `int OutputRedirector::kInvalidDescriptor = -1;`. / 执行一条独立语句或声明：`int OutputRedirector::kInvalidDescriptor = -1;`。
- **L29**: Blank line separating nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L30**: Continues logic associated with callable symbol `OutputRedirector`. / 继续与可调用符号 `OutputRedirector` 相关的逻辑。
- **L31**: Continues a multi-line argument list, initializer, or aggregate entry: `: m_fd(kInvalidDescriptor), m_original_fd(kInvalidDescriptor),`. / 继续一个多行参数列表、初始化器或聚合项：`: m_fd(kInvalidDescriptor), m_original_fd(kInvalidDescriptor),`。
- **L32**: Continues logic associated with callable symbol `m_restore_fd`. / 继续与可调用符号 `m_restore_fd` 相关的逻辑。
- **L33**: Blank line separating nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L34**: Starts a function, method, lambda, or structured scope: `Expected<int> OutputRedirector::GetWriteFileDescriptor() {`. / 开始一个函数、方法、lambda 或结构化作用域：`Expected<int> OutputRedirector::GetWriteFileDescriptor() {`。
- **L35**: Begins a `if` control-flow statement and evaluates its condition. / 开始 `if` 控制流语句并计算其条件。
- **L36**: Returns from the current function with `createStringError(std::errc::bad_file_descriptor,`. / 以 `createStringError(std::errc::bad_file_descriptor,` 从当前函数返回。

### Lines 37-48 / 第 37-48 行

```cpp
37 |                              "write handle is not open for writing");
38 |   return m_fd;
39 | }
40 | 
41 | Error OutputRedirector::RedirectTo(std::FILE *file_override,
42 |                                    std::function<void(StringRef)> callback) {
43 |   assert(m_fd == kInvalidDescriptor && "Output readirector already started.");
44 |   int new_fd[2];
45 | 
46 | #if defined(_WIN32)
47 |   if (::_pipe(new_fd, OutputBufferSize, O_TEXT) == -1) {
48 | #else
```

- **L37**: Executes a standalone statement or declaration: `"write handle is not open for writing");`. / 执行一条独立语句或声明：`"write handle is not open for writing");`。
- **L38**: Returns from the current function with `m_fd`. / 以 `m_fd` 从当前函数返回。
- **L39**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L40**: Blank line separating nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L41**: Continues a multi-line argument list, initializer, or aggregate entry: `Error OutputRedirector::RedirectTo(std::FILE *file_override,`. / 继续一个多行参数列表、初始化器或聚合项：`Error OutputRedirector::RedirectTo(std::FILE *file_override,`。
- **L42**: Starts a function, method, lambda, or structured scope: `std::function<void(StringRef)> callback) {`. / 开始一个函数、方法、lambda 或结构化作用域：`std::function<void(StringRef)> callback) {`。
- **L43**: Checks an internal invariant in debug builds. / 在调试构建中检查内部不变式。
- **L44**: Executes a standalone statement or declaration: `int new_fd[2];`. / 执行一条独立语句或声明：`int new_fd[2];`。
- **L45**: Blank line separating nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L46**: Starts a preprocessor conditional block: `#if defined(_WIN32)`. / 开始一个预处理条件块：`#if defined(_WIN32)`。
- **L47**: Begins a `if` control-flow statement and evaluates its condition. / 开始 `if` 控制流语句并计算其条件。
- **L48**: Continues the active preprocessor branch selection. / 继续当前的预处理分支选择。

### Lines 49-60 / 第 49-60 行

```cpp
49 |   if (::pipe(new_fd) == -1) {
50 | #endif
51 |     int error = errno;
52 |     return createStringError(inconvertibleErrorCode(),
53 |                              "Couldn't create new pipe %s", strerror(error));
54 |   }
55 | 
56 |   int read_fd = new_fd[0];
57 |   m_fd = new_fd[1];
58 | 
59 |   if (file_override) {
60 |     int override_fd = fileno(file_override);
```

- **L49**: Begins a `if` control-flow statement and evaluates its condition. / 开始 `if` 控制流语句并计算其条件。
- **L50**: Closes the current preprocessor conditional block. / 结束当前预处理条件块。
- **L51**: Initializes variable `error` from the right-hand expression. / 使用右侧表达式初始化变量 `error`。
- **L52**: Returns from the current function with `createStringError(inconvertibleErrorCode(),`. / 以 `createStringError(inconvertibleErrorCode(),` 从当前函数返回。
- **L53**: Executes a call or declaration centered on `strerror`. / 执行以 `strerror` 为核心的调用或声明。
- **L54**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L55**: Blank line separating nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L56**: Initializes variable `read_fd` from the right-hand expression. / 使用右侧表达式初始化变量 `read_fd`。
- **L57**: Executes a standalone statement or declaration: `m_fd = new_fd[1];`. / 执行一条独立语句或声明：`m_fd = new_fd[1];`。
- **L58**: Blank line separating nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L59**: Begins a `if` control-flow statement and evaluates its condition. / 开始 `if` 控制流语句并计算其条件。
- **L60**: Initializes variable `override_fd` from the right-hand expression. / 使用右侧表达式初始化变量 `override_fd`。

### Lines 61-72 / 第 61-72 行

```cpp
61 | 
62 |     // Backup the FD to restore once redirection is complete.
63 |     m_original_fd = override_fd;
64 |     m_restore_fd = dup(override_fd);
65 | 
66 |     // Override the existing fd the new write end of the pipe.
67 |     if (::dup2(m_fd, override_fd) == -1)
68 |       return llvm::errorCodeToError(llvm::errnoAsErrorCode());
69 |   }
70 | 
71 |   m_forwarder = std::thread([this, callback, read_fd]() {
72 |     char buffer[OutputBufferSize];
```

- **L61**: Blank line separating nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L62**: Comment explains nearby logic, invariants, or intent: `Backup the FD to restore once redirection is complete.`. / 注释说明了附近代码的逻辑、不变式或设计意图：`Backup the FD to restore once redirection is complete.`。
- **L63**: Executes a standalone statement or declaration: `m_original_fd = override_fd;`. / 执行一条独立语句或声明：`m_original_fd = override_fd;`。
- **L64**: Executes a call or declaration centered on `dup`. / 执行以 `dup` 为核心的调用或声明。
- **L65**: Blank line separating nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L66**: Comment explains nearby logic, invariants, or intent: `Override the existing fd the new write end of the pipe.`. / 注释说明了附近代码的逻辑、不变式或设计意图：`Override the existing fd the new write end of the pipe.`。
- **L67**: Begins a `if` control-flow statement and evaluates its condition. / 开始 `if` 控制流语句并计算其条件。
- **L68**: Returns from the current function with `llvm::errorCodeToError(llvm::errnoAsErrorCode())`. / 以 `llvm::errorCodeToError(llvm::errnoAsErrorCode())` 从当前函数返回。
- **L69**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L70**: Blank line separating nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L71**: Starts a function, method, lambda, or structured scope: `m_forwarder = std::thread([this, callback, read_fd]() {`. / 开始一个函数、方法、lambda 或结构化作用域：`m_forwarder = std::thread([this, callback, read_fd]() {`。
- **L72**: Executes a standalone statement or declaration: `char buffer[OutputBufferSize];`. / 执行一条独立语句或声明：`char buffer[OutputBufferSize];`。

### Lines 73-84 / 第 73-84 行

```cpp
73 |     while (!m_stopped) {
74 |       ssize_t bytes_count = ::read(read_fd, &buffer, sizeof(buffer));
75 |       if (bytes_count == -1) {
76 |         // Skip non-fatal errors.
77 |         if (errno == EAGAIN || errno == EINTR || errno == EWOULDBLOCK)
78 |           continue;
79 |         break;
80 |       }
81 |       // Skip the null byte used to trigger a Stop.
82 |       if (bytes_count == 1 && buffer[0] == '\0')
83 |         continue;
84 | 
```

- **L73**: Begins a `while` control-flow statement and evaluates its condition. / 开始 `while` 控制流语句并计算其条件。
- **L74**: Initializes variable `bytes_count` from the right-hand expression. / 使用右侧表达式初始化变量 `bytes_count`。
- **L75**: Begins a `if` control-flow statement and evaluates its condition. / 开始 `if` 控制流语句并计算其条件。
- **L76**: Comment explains nearby logic, invariants, or intent: `Skip non-fatal errors.`. / 注释说明了附近代码的逻辑、不变式或设计意图：`Skip non-fatal errors.`。
- **L77**: Begins a `if` control-flow statement and evaluates its condition. / 开始 `if` 控制流语句并计算其条件。
- **L78**: Skips to the next loop iteration. / 跳到下一次循环迭代。
- **L79**: Exits the nearest loop or switch statement. / 退出最近的循环或 switch 语句。
- **L80**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L81**: Comment explains nearby logic, invariants, or intent: `Skip the null byte used to trigger a Stop.`. / 注释说明了附近代码的逻辑、不变式或设计意图：`Skip the null byte used to trigger a Stop.`。
- **L82**: Begins a `if` control-flow statement and evaluates its condition. / 开始 `if` 控制流语句并计算其条件。
- **L83**: Skips to the next loop iteration. / 跳到下一次循环迭代。
- **L84**: Blank line separating nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。

### Lines 85-96 / 第 85-96 行

```cpp
85 |       StringRef data(buffer, bytes_count);
86 |       if (m_stopped)
87 |         data.consume_back(kCloseSentinel);
88 |       if (data.empty())
89 |         break;
90 | 
91 |       callback(data);
92 |     }
93 |     ::close(read_fd);
94 |   });
95 | 
96 |   return Error::success();
```

- **L85**: Executes a call or declaration centered on `data`. / 执行以 `data` 为核心的调用或声明。
- **L86**: Begins a `if` control-flow statement and evaluates its condition. / 开始 `if` 控制流语句并计算其条件。
- **L87**: Executes a call or declaration centered on `data.consume_back`. / 执行以 `data.consume_back` 为核心的调用或声明。
- **L88**: Begins a `if` control-flow statement and evaluates its condition. / 开始 `if` 控制流语句并计算其条件。
- **L89**: Exits the nearest loop or switch statement. / 退出最近的循环或 switch 语句。
- **L90**: Blank line separating nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L91**: Executes a call or declaration centered on `callback`. / 执行以 `callback` 为核心的调用或声明。
- **L92**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L93**: Executes a call or declaration centered on `::close`. / 执行以 `::close` 为核心的调用或声明。
- **L94**: Executes a standalone statement or declaration: `});`. / 执行一条独立语句或声明：`});`。
- **L95**: Blank line separating nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L96**: Returns from the current function with `Error::success()`. / 以 `Error::success()` 从当前函数返回。

### Lines 97-108 / 第 97-108 行

```cpp
 97 | }
 98 | 
 99 | void OutputRedirector::Stop() {
100 |   m_stopped = true;
101 | 
102 |   if (m_fd != kInvalidDescriptor) {
103 |     int fd = m_fd;
104 |     m_fd = kInvalidDescriptor;
105 |     // Closing the pipe may not be sufficient to wake up the thread in case the
106 |     // write descriptor is duplicated (to stdout/err or to another process).
107 |     // Write a null byte to ensure the read call returns.
108 |     (void)::write(fd, kCloseSentinel.data(), kCloseSentinel.size());
```

- **L97**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L98**: Blank line separating nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L99**: Starts a function, method, lambda, or structured scope: `void OutputRedirector::Stop() {`. / 开始一个函数、方法、lambda 或结构化作用域：`void OutputRedirector::Stop() {`。
- **L100**: Executes a standalone statement or declaration: `m_stopped = true;`. / 执行一条独立语句或声明：`m_stopped = true;`。
- **L101**: Blank line separating nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L102**: Begins a `if` control-flow statement and evaluates its condition. / 开始 `if` 控制流语句并计算其条件。
- **L103**: Initializes variable `fd` from the right-hand expression. / 使用右侧表达式初始化变量 `fd`。
- **L104**: Executes a standalone statement or declaration: `m_fd = kInvalidDescriptor;`. / 执行一条独立语句或声明：`m_fd = kInvalidDescriptor;`。
- **L105**: Comment explains nearby logic, invariants, or intent: `Closing the pipe may not be sufficient to wake up the thread in case the`. / 注释说明了附近代码的逻辑、不变式或设计意图：`Closing the pipe may not be sufficient to wake up the thread in case the`。
- **L106**: Comment explains nearby logic, invariants, or intent: `write descriptor is duplicated (to stdout/err or to another process).`. / 注释说明了附近代码的逻辑、不变式或设计意图：`write descriptor is duplicated (to stdout/err or to another process).`。
- **L107**: Comment explains nearby logic, invariants, or intent: `Write a null byte to ensure the read call returns.`. / 注释说明了附近代码的逻辑、不变式或设计意图：`Write a null byte to ensure the read call returns.`。
- **L108**: Executes a call or declaration centered on `statement`. / 执行以 `statement` 为核心的调用或声明。

### Lines 109-120 / 第 109-120 行

```cpp
109 |     ::close(fd);
110 |     m_forwarder.join();
111 | 
112 |     // Restore the fd back to its original state since we stopped the
113 |     // redirection.
114 |     if (m_restore_fd != kInvalidDescriptor &&
115 |         m_original_fd != kInvalidDescriptor) {
116 |       int restore_fd = m_restore_fd;
117 |       m_restore_fd = kInvalidDescriptor;
118 |       int original_fd = m_original_fd;
119 |       m_original_fd = kInvalidDescriptor;
120 |       ::dup2(restore_fd, original_fd);
```

- **L109**: Executes a call or declaration centered on `::close`. / 执行以 `::close` 为核心的调用或声明。
- **L110**: Executes a call or declaration centered on `m_forwarder.join`. / 执行以 `m_forwarder.join` 为核心的调用或声明。
- **L111**: Blank line separating nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L112**: Comment explains nearby logic, invariants, or intent: `Restore the fd back to its original state since we stopped the`. / 注释说明了附近代码的逻辑、不变式或设计意图：`Restore the fd back to its original state since we stopped the`。
- **L113**: Comment explains nearby logic, invariants, or intent: `redirection.`. / 注释说明了附近代码的逻辑、不变式或设计意图：`redirection.`。
- **L114**: Begins a `if` control-flow statement and evaluates its condition. / 开始 `if` 控制流语句并计算其条件。
- **L115**: Continues the surrounding expression or declaration: `m_original_fd != kInvalidDescriptor) {`. / 继续构造周围的表达式或声明：`m_original_fd != kInvalidDescriptor) {`。
- **L116**: Initializes variable `restore_fd` from the right-hand expression. / 使用右侧表达式初始化变量 `restore_fd`。
- **L117**: Executes a standalone statement or declaration: `m_restore_fd = kInvalidDescriptor;`. / 执行一条独立语句或声明：`m_restore_fd = kInvalidDescriptor;`。
- **L118**: Initializes variable `original_fd` from the right-hand expression. / 使用右侧表达式初始化变量 `original_fd`。
- **L119**: Executes a standalone statement or declaration: `m_original_fd = kInvalidDescriptor;`. / 执行一条独立语句或声明：`m_original_fd = kInvalidDescriptor;`。
- **L120**: Executes a call or declaration centered on `::dup2`. / 执行以 `::dup2` 为核心的调用或声明。

### Lines 121-125 / 第 121-125 行

```cpp
121 |     }
122 |   }
123 | }
124 | 
125 | } // namespace lldb_dap
```

- **L121**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L122**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L123**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L124**: Blank line separating nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L125**: Closes a namespace scope while preserving the trailing comment: `} // namespace lldb_dap`. / 结束一个命名空间作用域，并保留尾部注释：`} // namespace lldb_dap`。

## Key Concepts / 关键概念

- **Structured error handling / 结构化错误处理**:
  - **EN**: Uses LLVM error utilities to make failures explicit and composable.
  - **CN**: 使用 LLVM 错误处理工具显式且可组合地表示失败。

## Dependencies / 依赖关系

- `OutputRedirector.h`: Provides local declarations used by this file. / 提供本文件使用的本地声明。
- `DAP.h`: Provides local declarations used by this file. / 提供本文件使用的本地声明。
- `llvm/ADT/StringRef.h`: Provides LLVM ADT containers and utility types. / 提供LLVM ADT 容器与工具类型。
- `llvm/Support/Error.h`: Provides LLVM support-library facilities. / 提供LLVM Support 库设施。
- `cstring`: Provides supporting declarations used by the current translation unit. / 提供当前编译单元使用的辅助声明。
- `system_error`: Provides supporting declarations used by the current translation unit. / 提供当前编译单元使用的辅助声明。
- `fcntl.h`: Provides local declarations used by this file. / 提供本文件使用的本地声明。
- `io.h`: Provides local declarations used by this file. / 提供本文件使用的本地声明。
- `unistd.h`: Provides local declarations used by this file. / 提供本文件使用的本地声明。
