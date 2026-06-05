# MachThreadList.h — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `lldb/tools/debugserver/source/MacOSX/MachThreadList.h`
- **Repository / 仓库**: `llvm-project` (`/root/xw/llvm-project`)
- **Purpose / 目的**:
  - **EN**: Created by Greg Clayton on 6/19/07.
  - **CN**: 声明与 `MachThreadList` 相关的 LLDB 接口、类型以及辅助 API。

## Line-by-Line Analysis / 逐行分析

### Lines 1-12 / 第 1-12 行

```cpp
 1 | //===-- MachThreadList.h ----------------------------------------*- C++ -*-===//
 2 | //
 3 | // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
 4 | // See https://llvm.org/LICENSE.txt for license information.
 5 | // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
 6 | //
 7 | //===----------------------------------------------------------------------===//
 8 | //
 9 | //  Created by Greg Clayton on 6/19/07.
10 | //
11 | //===----------------------------------------------------------------------===//
12 | 
```

- **L1**: Banner comment marking a file or section boundary. / 横幅注释，用于标记文件或章节边界。
- **L2**: Separator comment used for visual grouping. / 用于视觉分组的分隔注释。
- **L3**: Comment explains nearby logic, invariants, or intent: `Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.`. / 注释说明了附近代码的逻辑、不变式或设计意图：`Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.`。
- **L4**: Comment explains nearby logic, invariants, or intent: `See https://llvm.org/LICENSE.txt for license information.`. / 注释说明了附近代码的逻辑、不变式或设计意图：`See https://llvm.org/LICENSE.txt for license information.`。
- **L5**: Comment explains nearby logic, invariants, or intent: `SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception`. / 注释说明了附近代码的逻辑、不变式或设计意图：`SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception`。
- **L6**: Separator comment used for visual grouping. / 用于视觉分组的分隔注释。
- **L7**: Banner comment marking a file or section boundary. / 横幅注释，用于标记文件或章节边界。
- **L8**: Separator comment used for visual grouping. / 用于视觉分组的分隔注释。
- **L9**: Comment explains nearby logic, invariants, or intent: `Created by Greg Clayton on 6/19/07.`. / 注释说明了附近代码的逻辑、不变式或设计意图：`Created by Greg Clayton on 6/19/07.`。
- **L10**: Separator comment used for visual grouping. / 用于视觉分组的分隔注释。
- **L11**: Banner comment marking a file or section boundary. / 横幅注释，用于标记文件或章节边界。
- **L12**: Blank line separating nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。

### Lines 13-24 / 第 13-24 行

```cpp
13 | #ifndef LLDB_TOOLS_DEBUGSERVER_SOURCE_MACOSX_MACHTHREADLIST_H
14 | #define LLDB_TOOLS_DEBUGSERVER_SOURCE_MACOSX_MACHTHREADLIST_H
15 | 
16 | #include "MachThread.h"
17 | #include "ThreadInfo.h"
18 | 
19 | class DNBThreadResumeActions;
20 | 
21 | class MachThreadList {
22 | public:
23 |   MachThreadList();
24 |   ~MachThreadList();
```

- **L13**: Starts a preprocessor conditional block: `#ifndef LLDB_TOOLS_DEBUGSERVER_SOURCE_MACOSX_MACHTHREADLIST_H`. / 开始一个预处理条件块：`#ifndef LLDB_TOOLS_DEBUGSERVER_SOURCE_MACOSX_MACHTHREADLIST_H`。
- **L14**: Defines macro `LLDB_TOOLS_DEBUGSERVER_SOURCE_MACOSX_MACHTHREADLIST_H` for local shorthand, feature control, or decoding logic. / 定义宏 `LLDB_TOOLS_DEBUGSERVER_SOURCE_MACOSX_MACHTHREADLIST_H`，供本地简写、特性控制或解码逻辑使用。
- **L15**: Blank line separating nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L16**: Includes "MachThread.h" to access local declarations used by this file. / 引入 "MachThread.h" 以使用本文件使用的本地声明。
- **L17**: Includes "ThreadInfo.h" to access local declarations used by this file. / 引入 "ThreadInfo.h" 以使用本文件使用的本地声明。
- **L18**: Blank line separating nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L19**: Declares class `DNBThreadResumeActions;`. / 声明 class `DNBThreadResumeActions;`。
- **L20**: Blank line separating nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L21**: Declares class `MachThreadList`. / 声明 class `MachThreadList`。
- **L22**: Sets the following members to `public` access. / 将后续成员的访问级别设为 `public`。
- **L23**: Executes a call or declaration centered on `MachThreadList`. / 执行以 `MachThreadList` 为核心的调用或声明。
- **L24**: Executes a call or declaration centered on `~MachThreadList`. / 执行以 `~MachThreadList` 为核心的调用或声明。

### Lines 25-36 / 第 25-36 行

```cpp
25 | 
26 |   void Clear();
27 |   void Dump() const;
28 |   bool GetRegisterValue(nub_thread_t tid, uint32_t set, uint32_t reg,
29 |                         DNBRegisterValue *reg_value) const;
30 |   bool SetRegisterValue(nub_thread_t tid, uint32_t set, uint32_t reg,
31 |                         const DNBRegisterValue *reg_value) const;
32 |   nub_size_t GetRegisterContext(nub_thread_t tid, void *buf, size_t buf_len);
33 |   nub_size_t SetRegisterContext(nub_thread_t tid, const void *buf,
34 |                                 size_t buf_len);
35 |   uint32_t SaveRegisterState(nub_thread_t tid);
36 |   bool RestoreRegisterState(nub_thread_t tid, uint32_t save_id);
```

- **L25**: Blank line separating nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L26**: Executes a call or declaration centered on `Clear`. / 执行以 `Clear` 为核心的调用或声明。
- **L27**: Executes a call or declaration centered on `Dump`. / 执行以 `Dump` 为核心的调用或声明。
- **L28**: Continues a multi-line argument list, initializer, or aggregate entry: `bool GetRegisterValue(nub_thread_t tid, uint32_t set, uint32_t reg,`. / 继续一个多行参数列表、初始化器或聚合项：`bool GetRegisterValue(nub_thread_t tid, uint32_t set, uint32_t reg,`。
- **L29**: Executes a standalone statement or declaration: `DNBRegisterValue *reg_value) const;`. / 执行一条独立语句或声明：`DNBRegisterValue *reg_value) const;`。
- **L30**: Continues a multi-line argument list, initializer, or aggregate entry: `bool SetRegisterValue(nub_thread_t tid, uint32_t set, uint32_t reg,`. / 继续一个多行参数列表、初始化器或聚合项：`bool SetRegisterValue(nub_thread_t tid, uint32_t set, uint32_t reg,`。
- **L31**: Executes a standalone statement or declaration: `const DNBRegisterValue *reg_value) const;`. / 执行一条独立语句或声明：`const DNBRegisterValue *reg_value) const;`。
- **L32**: Executes a call or declaration centered on `GetRegisterContext`. / 执行以 `GetRegisterContext` 为核心的调用或声明。
- **L33**: Continues a multi-line argument list, initializer, or aggregate entry: `nub_size_t SetRegisterContext(nub_thread_t tid, const void *buf,`. / 继续一个多行参数列表、初始化器或聚合项：`nub_size_t SetRegisterContext(nub_thread_t tid, const void *buf,`。
- **L34**: Executes a standalone statement or declaration: `size_t buf_len);`. / 执行一条独立语句或声明：`size_t buf_len);`。
- **L35**: Executes a call or declaration centered on `SaveRegisterState`. / 执行以 `SaveRegisterState` 为核心的调用或声明。
- **L36**: Executes a call or declaration centered on `RestoreRegisterState`. / 执行以 `RestoreRegisterState` 为核心的调用或声明。

### Lines 37-48 / 第 37-48 行

```cpp
37 |   const char *GetThreadInfo(nub_thread_t tid) const;
38 |   void ProcessWillResume(MachProcess *process,
39 |                          const DNBThreadResumeActions &thread_actions);
40 |   uint32_t ProcessDidStop(MachProcess *process);
41 |   bool NotifyException(MachException::Data &exc);
42 |   bool ShouldStop(bool &step_more);
43 |   const char *GetName(nub_thread_t tid);
44 |   nub_state_t GetState(nub_thread_t tid);
45 |   nub_thread_t SetCurrentThread(nub_thread_t tid);
46 | 
47 |   ThreadInfo::QoS GetRequestedQoS(nub_thread_t tid, nub_addr_t tsd,
48 |                                   uint64_t dti_qos_class_index);
```

- **L37**: Executes a call or declaration centered on `*GetThreadInfo`. / 执行以 `*GetThreadInfo` 为核心的调用或声明。
- **L38**: Continues a multi-line argument list, initializer, or aggregate entry: `void ProcessWillResume(MachProcess *process,`. / 继续一个多行参数列表、初始化器或聚合项：`void ProcessWillResume(MachProcess *process,`。
- **L39**: Executes a standalone statement or declaration: `const DNBThreadResumeActions &thread_actions);`. / 执行一条独立语句或声明：`const DNBThreadResumeActions &thread_actions);`。
- **L40**: Executes a call or declaration centered on `ProcessDidStop`. / 执行以 `ProcessDidStop` 为核心的调用或声明。
- **L41**: Executes a call or declaration centered on `NotifyException`. / 执行以 `NotifyException` 为核心的调用或声明。
- **L42**: Executes a call or declaration centered on `ShouldStop`. / 执行以 `ShouldStop` 为核心的调用或声明。
- **L43**: Executes a call or declaration centered on `*GetName`. / 执行以 `*GetName` 为核心的调用或声明。
- **L44**: Executes a call or declaration centered on `GetState`. / 执行以 `GetState` 为核心的调用或声明。
- **L45**: Executes a call or declaration centered on `SetCurrentThread`. / 执行以 `SetCurrentThread` 为核心的调用或声明。
- **L46**: Blank line separating nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L47**: Continues a multi-line argument list, initializer, or aggregate entry: `ThreadInfo::QoS GetRequestedQoS(nub_thread_t tid, nub_addr_t tsd,`. / 继续一个多行参数列表、初始化器或聚合项：`ThreadInfo::QoS GetRequestedQoS(nub_thread_t tid, nub_addr_t tsd,`。
- **L48**: Executes a standalone statement or declaration: `uint64_t dti_qos_class_index);`. / 执行一条独立语句或声明：`uint64_t dti_qos_class_index);`。

### Lines 49-60 / 第 49-60 行

```cpp
49 |   nub_addr_t GetPThreadT(nub_thread_t tid);
50 |   nub_addr_t GetDispatchQueueT(nub_thread_t tid);
51 |   nub_addr_t
52 |   GetTSDAddressForThread(nub_thread_t tid,
53 |                          uint64_t plo_pthread_tsd_base_address_offset,
54 |                          uint64_t plo_pthread_tsd_base_offset,
55 |                          uint64_t plo_pthread_tsd_entry_size);
56 | 
57 |   bool GetThreadStoppedReason(nub_thread_t tid,
58 |                               struct DNBThreadStopInfo *stop_info) const;
59 |   void DumpThreadStoppedReason(nub_thread_t tid) const;
60 |   bool GetIdentifierInfo(nub_thread_t tid,
```

- **L49**: Executes a call or declaration centered on `GetPThreadT`. / 执行以 `GetPThreadT` 为核心的调用或声明。
- **L50**: Executes a call or declaration centered on `GetDispatchQueueT`. / 执行以 `GetDispatchQueueT` 为核心的调用或声明。
- **L51**: Continues the surrounding expression or declaration: `nub_addr_t`. / 继续构造周围的表达式或声明：`nub_addr_t`。
- **L52**: Continues a multi-line argument list, initializer, or aggregate entry: `GetTSDAddressForThread(nub_thread_t tid,`. / 继续一个多行参数列表、初始化器或聚合项：`GetTSDAddressForThread(nub_thread_t tid,`。
- **L53**: Continues a multi-line argument list, initializer, or aggregate entry: `uint64_t plo_pthread_tsd_base_address_offset,`. / 继续一个多行参数列表、初始化器或聚合项：`uint64_t plo_pthread_tsd_base_address_offset,`。
- **L54**: Continues a multi-line argument list, initializer, or aggregate entry: `uint64_t plo_pthread_tsd_base_offset,`. / 继续一个多行参数列表、初始化器或聚合项：`uint64_t plo_pthread_tsd_base_offset,`。
- **L55**: Executes a standalone statement or declaration: `uint64_t plo_pthread_tsd_entry_size);`. / 执行一条独立语句或声明：`uint64_t plo_pthread_tsd_entry_size);`。
- **L56**: Blank line separating nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L57**: Continues a multi-line argument list, initializer, or aggregate entry: `bool GetThreadStoppedReason(nub_thread_t tid,`. / 继续一个多行参数列表、初始化器或聚合项：`bool GetThreadStoppedReason(nub_thread_t tid,`。
- **L58**: Declares struct `DNBThreadStopInfo`. / 声明 struct `DNBThreadStopInfo`。
- **L59**: Executes a call or declaration centered on `DumpThreadStoppedReason`. / 执行以 `DumpThreadStoppedReason` 为核心的调用或声明。
- **L60**: Continues a multi-line argument list, initializer, or aggregate entry: `bool GetIdentifierInfo(nub_thread_t tid,`. / 继续一个多行参数列表、初始化器或聚合项：`bool GetIdentifierInfo(nub_thread_t tid,`。

### Lines 61-72 / 第 61-72 行

```cpp
61 |                          thread_identifier_info_data_t *ident_info);
62 |   nub_size_t NumThreads() const;
63 |   nub_thread_t ThreadIDAtIndex(nub_size_t idx) const;
64 |   nub_thread_t CurrentThreadID();
65 |   void CurrentThread(MachThreadSP &threadSP);
66 |   void NotifyBreakpointChanged(const DNBBreakpoint *bp);
67 |   uint32_t EnableHardwareBreakpoint(const DNBBreakpoint *bp) const;
68 |   bool DisableHardwareBreakpoint(const DNBBreakpoint *bp) const;
69 |   uint32_t EnableHardwareWatchpoint(const DNBBreakpoint *wp) const;
70 |   bool DisableHardwareWatchpoint(const DNBBreakpoint *wp) const;
71 |   uint32_t NumSupportedHardwareWatchpoints() const;
72 | 
```

- **L61**: Executes a standalone statement or declaration: `thread_identifier_info_data_t *ident_info);`. / 执行一条独立语句或声明：`thread_identifier_info_data_t *ident_info);`。
- **L62**: Executes a call or declaration centered on `NumThreads`. / 执行以 `NumThreads` 为核心的调用或声明。
- **L63**: Executes a call or declaration centered on `ThreadIDAtIndex`. / 执行以 `ThreadIDAtIndex` 为核心的调用或声明。
- **L64**: Executes a call or declaration centered on `CurrentThreadID`. / 执行以 `CurrentThreadID` 为核心的调用或声明。
- **L65**: Executes a call or declaration centered on `CurrentThread`. / 执行以 `CurrentThread` 为核心的调用或声明。
- **L66**: Executes a call or declaration centered on `NotifyBreakpointChanged`. / 执行以 `NotifyBreakpointChanged` 为核心的调用或声明。
- **L67**: Executes a call or declaration centered on `EnableHardwareBreakpoint`. / 执行以 `EnableHardwareBreakpoint` 为核心的调用或声明。
- **L68**: Executes a call or declaration centered on `DisableHardwareBreakpoint`. / 执行以 `DisableHardwareBreakpoint` 为核心的调用或声明。
- **L69**: Executes a call or declaration centered on `EnableHardwareWatchpoint`. / 执行以 `EnableHardwareWatchpoint` 为核心的调用或声明。
- **L70**: Executes a call or declaration centered on `DisableHardwareWatchpoint`. / 执行以 `DisableHardwareWatchpoint` 为核心的调用或声明。
- **L71**: Executes a call or declaration centered on `NumSupportedHardwareWatchpoints`. / 执行以 `NumSupportedHardwareWatchpoints` 为核心的调用或声明。
- **L72**: Blank line separating nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。

### Lines 73-84 / 第 73-84 行

```cpp
73 |   uint32_t GetThreadIndexForThreadStoppedWithSignal(const int signo) const;
74 | 
75 |   MachThreadSP GetThreadByID(nub_thread_t tid) const;
76 | 
77 |   MachThreadSP GetThreadByMachPortNumber(thread_t mach_port_number) const;
78 |   nub_thread_t GetThreadIDByMachPortNumber(thread_t mach_port_number) const;
79 |   thread_t GetMachPortNumberByThreadID(nub_thread_t globally_unique_id) const;
80 | 
81 | protected:
82 |   typedef std::vector<MachThreadSP> collection;
83 |   typedef collection::iterator iterator;
84 |   typedef collection::const_iterator const_iterator;
```

- **L73**: Executes a call or declaration centered on `GetThreadIndexForThreadStoppedWithSignal`. / 执行以 `GetThreadIndexForThreadStoppedWithSignal` 为核心的调用或声明。
- **L74**: Blank line separating nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L75**: Executes a call or declaration centered on `GetThreadByID`. / 执行以 `GetThreadByID` 为核心的调用或声明。
- **L76**: Blank line separating nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L77**: Executes a call or declaration centered on `GetThreadByMachPortNumber`. / 执行以 `GetThreadByMachPortNumber` 为核心的调用或声明。
- **L78**: Executes a call or declaration centered on `GetThreadIDByMachPortNumber`. / 执行以 `GetThreadIDByMachPortNumber` 为核心的调用或声明。
- **L79**: Executes a call or declaration centered on `GetMachPortNumberByThreadID`. / 执行以 `GetMachPortNumberByThreadID` 为核心的调用或声明。
- **L80**: Blank line separating nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L81**: Sets the following members to `protected` access. / 将后续成员的访问级别设为 `protected`。
- **L82**: Adds an auxiliary declaration: `typedef std::vector<MachThreadSP> collection;`. / 添加一条辅助声明：`typedef std::vector<MachThreadSP> collection;`。
- **L83**: Adds an auxiliary declaration: `typedef collection::iterator iterator;`. / 添加一条辅助声明：`typedef collection::iterator iterator;`。
- **L84**: Adds an auxiliary declaration: `typedef collection::const_iterator const_iterator;`. / 添加一条辅助声明：`typedef collection::const_iterator const_iterator;`。

### Lines 85-96 / 第 85-96 行

```cpp
85 | 
86 |   enum class HardwareBreakpointAction {
87 |     EnableWatchpoint,
88 |     DisableWatchpoint,
89 |     EnableBreakpoint,
90 |     DisableBreakpoint,
91 |   };
92 | 
93 |   uint32_t DoHardwareBreakpointAction(const DNBBreakpoint *bp,
94 |                                       HardwareBreakpointAction action) const;
95 | 
96 |   uint32_t UpdateThreadList(MachProcess *process, bool update,
```

- **L85**: Blank line separating nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L86**: Declares enum `class`. / 声明 enum `class`。
- **L87**: Continues a multi-line argument list, initializer, or aggregate entry: `EnableWatchpoint,`. / 继续一个多行参数列表、初始化器或聚合项：`EnableWatchpoint,`。
- **L88**: Continues a multi-line argument list, initializer, or aggregate entry: `DisableWatchpoint,`. / 继续一个多行参数列表、初始化器或聚合项：`DisableWatchpoint,`。
- **L89**: Continues a multi-line argument list, initializer, or aggregate entry: `EnableBreakpoint,`. / 继续一个多行参数列表、初始化器或聚合项：`EnableBreakpoint,`。
- **L90**: Continues a multi-line argument list, initializer, or aggregate entry: `DisableBreakpoint,`. / 继续一个多行参数列表、初始化器或聚合项：`DisableBreakpoint,`。
- **L91**: Closes the current declaration scope such as a class, struct, or enum. / 结束当前声明作用域，例如类、结构体或枚举。
- **L92**: Blank line separating nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L93**: Continues a multi-line argument list, initializer, or aggregate entry: `uint32_t DoHardwareBreakpointAction(const DNBBreakpoint *bp,`. / 继续一个多行参数列表、初始化器或聚合项：`uint32_t DoHardwareBreakpointAction(const DNBBreakpoint *bp,`。
- **L94**: Executes a standalone statement or declaration: `HardwareBreakpointAction action) const;`. / 执行一条独立语句或声明：`HardwareBreakpointAction action) const;`。
- **L95**: Blank line separating nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L96**: Continues a multi-line argument list, initializer, or aggregate entry: `uint32_t UpdateThreadList(MachProcess *process, bool update,`. / 继续一个多行参数列表、初始化器或聚合项：`uint32_t UpdateThreadList(MachProcess *process, bool update,`。

### Lines 97-106 / 第 97-106 行

```cpp
 97 |                             collection *num_threads = NULL);
 98 |   //  const_iterator  FindThreadByID (thread_t tid) const;
 99 | 
100 |   collection m_threads;
101 |   mutable std::recursive_mutex m_threads_mutex;
102 |   MachThreadSP m_current_thread;
103 |   bool m_is_64_bit;
104 | };
105 | 
106 | #endif // LLDB_TOOLS_DEBUGSERVER_SOURCE_MACOSX_MACHTHREADLIST_H
```

- **L97**: Executes a standalone statement or declaration: `collection *num_threads = NULL);`. / 执行一条独立语句或声明：`collection *num_threads = NULL);`。
- **L98**: Comment explains nearby logic, invariants, or intent: `const_iterator  FindThreadByID (thread_t tid) const;`. / 注释说明了附近代码的逻辑、不变式或设计意图：`const_iterator  FindThreadByID (thread_t tid) const;`。
- **L99**: Blank line separating nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L100**: Executes a standalone statement or declaration: `collection m_threads;`. / 执行一条独立语句或声明：`collection m_threads;`。
- **L101**: Executes a standalone statement or declaration: `mutable std::recursive_mutex m_threads_mutex;`. / 执行一条独立语句或声明：`mutable std::recursive_mutex m_threads_mutex;`。
- **L102**: Executes a standalone statement or declaration: `MachThreadSP m_current_thread;`. / 执行一条独立语句或声明：`MachThreadSP m_current_thread;`。
- **L103**: Executes a standalone statement or declaration: `bool m_is_64_bit;`. / 执行一条独立语句或声明：`bool m_is_64_bit;`。
- **L104**: Closes the current declaration scope such as a class, struct, or enum. / 结束当前声明作用域，例如类、结构体或枚举。
- **L105**: Blank line separating nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L106**: Closes the current preprocessor conditional block. / 结束当前预处理条件块。

## Key Concepts / 关键概念

- **Register modeling / 寄存器建模**:
  - **EN**: Represents register layouts, generic roles, or architecture-specific register behavior.
  - **CN**: 表示寄存器布局、通用角色或体系结构专用寄存器行为。
- **Breakpoint management / 断点管理**:
  - **EN**: Coordinates breakpoint placement, resolution, and stop-time behavior.
  - **CN**: 协调断点的设置、解析与停止时行为。
- **Thread and process state / 线程与进程状态**:
  - **EN**: Tracks execution context, threads, or process-side runtime state.
  - **CN**: 跟踪执行上下文、线程或进程侧运行时状态。

## Dependencies / 依赖关系

- `MachThread.h`: Provides local declarations used by this file. / 提供本文件使用的本地声明。
- `ThreadInfo.h`: Provides local declarations used by this file. / 提供本文件使用的本地声明。
