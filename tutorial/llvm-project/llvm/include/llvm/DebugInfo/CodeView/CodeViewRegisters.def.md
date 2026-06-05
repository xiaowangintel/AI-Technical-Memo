# CodeViewRegisters.def — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `llvm/include/llvm/DebugInfo/CodeView/CodeViewRegisters.def`
- **Repository / 仓库**: `llvm-project`
- **Purpose (EN)**: See CV_HREG_e in cvconst.h. This should match the constants there.
- **Purpose (CN)**: 该文件位于 `llvm/include/llvm/DebugInfo`，主要声明 `CodeViewRegisters` 相关的调试信息数据结构、读取流程或辅助逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-24

````cpp
//===-- CodeViewRegisters.def - CodeView registers --------------*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// See CV_HREG_e in cvconst.h. This should match the constants there.
//
//===----------------------------------------------------------------------===//

#ifndef CV_REGISTER
#define CV_REGISTER(name, value)
#endif

#if !defined(CV_REGISTERS_ALL) && !defined(CV_REGISTERS_X86) &&                \
    !defined(CV_REGISTERS_ARM) &&                                              \
    !defined(CV_REGISTERS_ARM64) &&                                            \
    !defined(CV_REGISTERS_MIPS)
#error Need include at least one register set.
#endif

// This currently only contains the "register subset shared by all processor
````
- **L1 EN**: Banner comment marking a file or section boundary.
  **L1 CN**: 横幅注释，用于标记文件或章节边界。
- **L2 EN**: Separator comment used for visual grouping.
  **L2 CN**: 用于视觉分组的分隔注释。
- **L3 EN**: Comment explains nearby logic, invariants, or intent: `Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.`.
  **L3 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.`。
- **L4 EN**: Comment explains nearby logic, invariants, or intent: `See https://llvm.org/LICENSE.txt for license information.`.
  **L4 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`See https://llvm.org/LICENSE.txt for license information.`。
- **L5 EN**: Comment explains nearby logic, invariants, or intent: `SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception`.
  **L5 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception`。
- **L6 EN**: Separator comment used for visual grouping.
  **L6 CN**: 用于视觉分组的分隔注释。
- **L7 EN**: Banner comment marking a file or section boundary.
  **L7 CN**: 横幅注释，用于标记文件或章节边界。
- **L8 EN**: Separator comment used for visual grouping.
  **L8 CN**: 用于视觉分组的分隔注释。
- **L9 EN**: Comment explains nearby logic, invariants, or intent: `See CV_HREG_e in cvconst.h. This should match the constants there.`.
  **L9 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`See CV_HREG_e in cvconst.h. This should match the constants there.`。
- **L10 EN**: Separator comment used for visual grouping.
  **L10 CN**: 用于视觉分组的分隔注释。
- **L11 EN**: Banner comment marking a file or section boundary.
  **L11 CN**: 横幅注释，用于标记文件或章节边界。
- **L12 EN**: Blank line separating nearby declarations or logic blocks.
  **L12 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L13 EN**: Starts a preprocessor conditional block: `#ifndef CV_REGISTER`.
  **L13 CN**: 开始一个预处理条件块：`#ifndef CV_REGISTER`。
- **L14 EN**: Defines macro `CV_REGISTER(name,` for conditional compilation, local shorthand, or diagnostics.
  **L14 CN**: 定义宏 `CV_REGISTER(name,`，供条件编译、本地简写或诊断使用。
- **L15 EN**: Closes the current preprocessor conditional block.
  **L15 CN**: 结束当前预处理条件块。
- **L16 EN**: Blank line separating nearby declarations or logic blocks.
  **L16 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L17 EN**: Starts a preprocessor conditional block: `#if !defined(CV_REGISTERS_ALL) && !defined(CV_REGISTERS_X86) &&                \`.
  **L17 CN**: 开始一个预处理条件块：`#if !defined(CV_REGISTERS_ALL) && !defined(CV_REGISTERS_X86) &&                \`。
- **L18 EN**: Continues logic associated with callable symbol `defined`.
  **L18 CN**: 继续与可调用符号 `defined` 相关的逻辑。
- **L19 EN**: Continues logic associated with callable symbol `defined`.
  **L19 CN**: 继续与可调用符号 `defined` 相关的逻辑。
- **L20 EN**: Continues logic associated with callable symbol `defined`.
  **L20 CN**: 继续与可调用符号 `defined` 相关的逻辑。
- **L21 EN**: Continues the surrounding expression or declaration: `#error Need include at least one register set.`.
  **L21 CN**: 继续构造周围的表达式或声明：`#error Need include at least one register set.`。
- **L22 EN**: Closes the current preprocessor conditional block.
  **L22 CN**: 结束当前预处理条件块。
- **L23 EN**: Blank line separating nearby declarations or logic blocks.
  **L23 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L24 EN**: Comment explains nearby logic, invariants, or intent: `This currently only contains the "register subset shared by all processor`.
  **L24 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`This currently only contains the "register subset shared by all processor`。

### Lines 25-48

````cpp
// types" (ERR etc.) and the x86/arm64 registers.

#if defined(CV_REGISTERS_ALL) || defined(CV_REGISTERS_X86)

// Some system headers define macros that conflict with our enums. Every
// compiler supported by LLVM has the push_macro and pop_macro pragmas, so use
// them to avoid the conflict.
#pragma push_macro("CR0")
#pragma push_macro("CR1")
#pragma push_macro("CR2")
#pragma push_macro("CR3")
#pragma push_macro("CR4")

CV_REGISTER(ERR, 30000)
CV_REGISTER(TEB, 30001)
CV_REGISTER(TIMER, 30002)
CV_REGISTER(EFAD1, 30003)
CV_REGISTER(EFAD2, 30004)
CV_REGISTER(EFAD3, 30005)
CV_REGISTER(VFRAME, 30006)
CV_REGISTER(HANDLE, 30007)
CV_REGISTER(PARAMS, 30008)
CV_REGISTER(LOCALS, 30009)
CV_REGISTER(TID, 30010)
````
- **L25 EN**: Comment explains nearby logic, invariants, or intent: `types" (ERR etc.) and the x86/arm64 registers.`.
  **L25 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`types" (ERR etc.) and the x86/arm64 registers.`。
- **L26 EN**: Blank line separating nearby declarations or logic blocks.
  **L26 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L27 EN**: Starts a preprocessor conditional block: `#if defined(CV_REGISTERS_ALL) || defined(CV_REGISTERS_X86)`.
  **L27 CN**: 开始一个预处理条件块：`#if defined(CV_REGISTERS_ALL) || defined(CV_REGISTERS_X86)`。
- **L28 EN**: Blank line separating nearby declarations or logic blocks.
  **L28 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L29 EN**: Comment explains nearby logic, invariants, or intent: `Some system headers define macros that conflict with our enums. Every`.
  **L29 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Some system headers define macros that conflict with our enums. Every`。
- **L30 EN**: Comment explains nearby logic, invariants, or intent: `compiler supported by LLVM has the push_macro and pop_macro pragmas, so use`.
  **L30 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`compiler supported by LLVM has the push_macro and pop_macro pragmas, so use`。
- **L31 EN**: Comment explains nearby logic, invariants, or intent: `them to avoid the conflict.`.
  **L31 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`them to avoid the conflict.`。
- **L32 EN**: Continues logic associated with callable symbol `push_macro`.
  **L32 CN**: 继续与可调用符号 `push_macro` 相关的逻辑。
- **L33 EN**: Continues logic associated with callable symbol `push_macro`.
  **L33 CN**: 继续与可调用符号 `push_macro` 相关的逻辑。
- **L34 EN**: Continues logic associated with callable symbol `push_macro`.
  **L34 CN**: 继续与可调用符号 `push_macro` 相关的逻辑。
- **L35 EN**: Continues logic associated with callable symbol `push_macro`.
  **L35 CN**: 继续与可调用符号 `push_macro` 相关的逻辑。
- **L36 EN**: Continues logic associated with callable symbol `push_macro`.
  **L36 CN**: 继续与可调用符号 `push_macro` 相关的逻辑。
- **L37 EN**: Blank line separating nearby declarations or logic blocks.
  **L37 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L38 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L38 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L39 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L39 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L40 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L40 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L41 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L41 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L42 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L42 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L43 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L43 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L44 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L44 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L45 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L45 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L46 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L46 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L47 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L47 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L48 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L48 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。

### Lines 49-72

````cpp
CV_REGISTER(ENV, 30011)
CV_REGISTER(CMDLN, 30012)

CV_REGISTER(NONE, 0)
CV_REGISTER(AL, 1)
CV_REGISTER(CL, 2)
CV_REGISTER(DL, 3)
CV_REGISTER(BL, 4)
CV_REGISTER(AH, 5)
CV_REGISTER(CH, 6)
CV_REGISTER(DH, 7)
CV_REGISTER(BH, 8)
CV_REGISTER(AX, 9)
CV_REGISTER(CX, 10)
CV_REGISTER(DX, 11)
CV_REGISTER(BX, 12)
CV_REGISTER(SP, 13)
CV_REGISTER(BP, 14)
CV_REGISTER(SI, 15)
CV_REGISTER(DI, 16)
CV_REGISTER(EAX, 17)
CV_REGISTER(ECX, 18)
CV_REGISTER(EDX, 19)
CV_REGISTER(EBX, 20)
````
- **L49 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L49 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L50 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L50 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L51 EN**: Blank line separating nearby declarations or logic blocks.
  **L51 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L52 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L52 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L53 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L53 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L54 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L54 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L55 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L55 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L56 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L56 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L57 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L57 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L58 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L58 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L59 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L59 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L60 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L60 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L61 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L61 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L62 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L62 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L63 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L63 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L64 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L64 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L65 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L65 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L66 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L66 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L67 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L67 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L68 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L68 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L69 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L69 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L70 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L70 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L71 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L71 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L72 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L72 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。

### Lines 73-96

````cpp
CV_REGISTER(ESP, 21)
CV_REGISTER(EBP, 22)
CV_REGISTER(ESI, 23)
CV_REGISTER(EDI, 24)
CV_REGISTER(ES, 25)
CV_REGISTER(CS, 26)
CV_REGISTER(SS, 27)
CV_REGISTER(DS, 28)
CV_REGISTER(FS, 29)
CV_REGISTER(GS, 30)
CV_REGISTER(IP, 31)
CV_REGISTER(FLAGS, 32)
CV_REGISTER(EIP, 33)
CV_REGISTER(EFLAGS, 34)
CV_REGISTER(TEMP, 40)
CV_REGISTER(TEMPH, 41)
CV_REGISTER(QUOTE, 42)
CV_REGISTER(PCDR3, 43)
CV_REGISTER(PCDR4, 44)
CV_REGISTER(PCDR5, 45)
CV_REGISTER(PCDR6, 46)
CV_REGISTER(PCDR7, 47)
CV_REGISTER(CR0, 80)
CV_REGISTER(CR1, 81)
````
- **L73 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L73 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L74 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L74 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L75 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L75 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L76 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L76 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L77 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L77 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L78 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L78 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L79 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L79 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L80 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L80 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L81 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L81 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L82 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L82 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L83 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L83 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L84 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L84 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L85 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L85 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L86 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L86 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L87 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L87 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L88 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L88 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L89 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L89 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L90 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L90 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L91 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L91 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L92 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L92 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L93 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L93 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L94 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L94 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L95 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L95 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L96 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L96 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。

### Lines 97-120

````cpp
CV_REGISTER(CR2, 82)
CV_REGISTER(CR3, 83)
CV_REGISTER(CR4, 84)
CV_REGISTER(DR0, 90)
CV_REGISTER(DR1, 91)
CV_REGISTER(DR2, 92)
CV_REGISTER(DR3, 93)
CV_REGISTER(DR4, 94)
CV_REGISTER(DR5, 95)
CV_REGISTER(DR6, 96)
CV_REGISTER(DR7, 97)
CV_REGISTER(GDTR, 110)
CV_REGISTER(GDTL, 111)
CV_REGISTER(IDTR, 112)
CV_REGISTER(IDTL, 113)
CV_REGISTER(LDTR, 114)
CV_REGISTER(TR, 115)

CV_REGISTER(PSEUDO1, 116)
CV_REGISTER(PSEUDO2, 117)
CV_REGISTER(PSEUDO3, 118)
CV_REGISTER(PSEUDO4, 119)
CV_REGISTER(PSEUDO5, 120)
CV_REGISTER(PSEUDO6, 121)
````
- **L97 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L97 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L98 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L98 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L99 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L99 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L100 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L100 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L101 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L101 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L102 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L102 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L103 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L103 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L104 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L104 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L105 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L105 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L106 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L106 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L107 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L107 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L108 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L108 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L109 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L109 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L110 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L110 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L111 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L111 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L112 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L112 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L113 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L113 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L114 EN**: Blank line separating nearby declarations or logic blocks.
  **L114 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L115 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L115 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L116 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L116 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L117 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L117 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L118 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L118 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L119 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L119 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L120 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L120 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。

### Lines 121-144

````cpp
CV_REGISTER(PSEUDO7, 122)
CV_REGISTER(PSEUDO8, 123)
CV_REGISTER(PSEUDO9, 124)

CV_REGISTER(ST0, 128)
CV_REGISTER(ST1, 129)
CV_REGISTER(ST2, 130)
CV_REGISTER(ST3, 131)
CV_REGISTER(ST4, 132)
CV_REGISTER(ST5, 133)
CV_REGISTER(ST6, 134)
CV_REGISTER(ST7, 135)
CV_REGISTER(CTRL, 136)
CV_REGISTER(STAT, 137)
CV_REGISTER(TAG, 138)
CV_REGISTER(FPIP, 139)
CV_REGISTER(FPCS, 140)
CV_REGISTER(FPDO, 141)
CV_REGISTER(FPDS, 142)
CV_REGISTER(ISEM, 143)
CV_REGISTER(FPEIP, 144)
CV_REGISTER(FPEDO, 145)

CV_REGISTER(MM0, 146)
````
- **L121 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L121 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L122 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L122 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L123 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L123 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L124 EN**: Blank line separating nearby declarations or logic blocks.
  **L124 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L125 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L125 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L126 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L126 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L127 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L127 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L128 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L128 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L129 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L129 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L130 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L130 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L131 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L131 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L132 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L132 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L133 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L133 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L134 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L134 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L135 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L135 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L136 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L136 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L137 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L137 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L138 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L138 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L139 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L139 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L140 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L140 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L141 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L141 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L142 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L142 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L143 EN**: Blank line separating nearby declarations or logic blocks.
  **L143 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L144 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L144 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。

### Lines 145-168

````cpp
CV_REGISTER(MM1, 147)
CV_REGISTER(MM2, 148)
CV_REGISTER(MM3, 149)
CV_REGISTER(MM4, 150)
CV_REGISTER(MM5, 151)
CV_REGISTER(MM6, 152)
CV_REGISTER(MM7, 153)

CV_REGISTER(XMM0, 154)
CV_REGISTER(XMM1, 155)
CV_REGISTER(XMM2, 156)
CV_REGISTER(XMM3, 157)
CV_REGISTER(XMM4, 158)
CV_REGISTER(XMM5, 159)
CV_REGISTER(XMM6, 160)
CV_REGISTER(XMM7, 161)

CV_REGISTER(MXCSR, 211)

CV_REGISTER(EDXEAX, 212)

CV_REGISTER(EMM0L, 220)
CV_REGISTER(EMM1L, 221)
CV_REGISTER(EMM2L, 222)
````
- **L145 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L145 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L146 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L146 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L147 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L147 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L148 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L148 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L149 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L149 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L150 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L150 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L151 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L151 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L152 EN**: Blank line separating nearby declarations or logic blocks.
  **L152 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L153 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L153 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L154 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L154 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L155 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L155 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L156 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L156 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L157 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L157 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L158 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L158 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L159 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L159 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L160 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L160 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L161 EN**: Blank line separating nearby declarations or logic blocks.
  **L161 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L162 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L162 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L163 EN**: Blank line separating nearby declarations or logic blocks.
  **L163 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L164 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L164 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L165 EN**: Blank line separating nearby declarations or logic blocks.
  **L165 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L166 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L166 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L167 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L167 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L168 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L168 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。

### Lines 169-192

````cpp
CV_REGISTER(EMM3L, 223)
CV_REGISTER(EMM4L, 224)
CV_REGISTER(EMM5L, 225)
CV_REGISTER(EMM6L, 226)
CV_REGISTER(EMM7L, 227)

CV_REGISTER(EMM0H, 228)
CV_REGISTER(EMM1H, 229)
CV_REGISTER(EMM2H, 230)
CV_REGISTER(EMM3H, 231)
CV_REGISTER(EMM4H, 232)
CV_REGISTER(EMM5H, 233)
CV_REGISTER(EMM6H, 234)
CV_REGISTER(EMM7H, 235)

CV_REGISTER(MM00, 236)
CV_REGISTER(MM01, 237)
CV_REGISTER(MM10, 238)
CV_REGISTER(MM11, 239)
CV_REGISTER(MM20, 240)
CV_REGISTER(MM21, 241)
CV_REGISTER(MM30, 242)
CV_REGISTER(MM31, 243)
CV_REGISTER(MM40, 244)
````
- **L169 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L169 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L170 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L170 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L171 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L171 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L172 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L172 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L173 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L173 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L174 EN**: Blank line separating nearby declarations or logic blocks.
  **L174 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L175 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L175 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L176 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L176 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L177 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L177 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L178 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L178 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L179 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L179 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L180 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L180 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L181 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L181 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L182 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L182 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L183 EN**: Blank line separating nearby declarations or logic blocks.
  **L183 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L184 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L184 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L185 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L185 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L186 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L186 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L187 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L187 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L188 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L188 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L189 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L189 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L190 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L190 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L191 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L191 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L192 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L192 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。

### Lines 193-216

````cpp
CV_REGISTER(MM41, 245)
CV_REGISTER(MM50, 246)
CV_REGISTER(MM51, 247)
CV_REGISTER(MM60, 248)
CV_REGISTER(MM61, 249)
CV_REGISTER(MM70, 250)
CV_REGISTER(MM71, 251)

CV_REGISTER(BND0, 396)
CV_REGISTER(BND1, 397)
CV_REGISTER(BND2, 398)


CV_REGISTER(XMM8, 252)
CV_REGISTER(XMM9, 253)
CV_REGISTER(XMM10, 254)
CV_REGISTER(XMM11, 255)
CV_REGISTER(XMM12, 256)
CV_REGISTER(XMM13, 257)
CV_REGISTER(XMM14, 258)
CV_REGISTER(XMM15, 259)


CV_REGISTER(SIL, 324)
````
- **L193 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L193 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L194 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L194 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L195 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L195 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L196 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L196 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L197 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L197 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L198 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L198 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L199 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L199 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L200 EN**: Blank line separating nearby declarations or logic blocks.
  **L200 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L201 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L201 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L202 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L202 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L203 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L203 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L204 EN**: Blank line separating nearby declarations or logic blocks.
  **L204 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L205 EN**: Blank line separating nearby declarations or logic blocks.
  **L205 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L206 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L206 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L207 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L207 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L208 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L208 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L209 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L209 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L210 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L210 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L211 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L211 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L212 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L212 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L213 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L213 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L214 EN**: Blank line separating nearby declarations or logic blocks.
  **L214 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L215 EN**: Blank line separating nearby declarations or logic blocks.
  **L215 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L216 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L216 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。

### Lines 217-240

````cpp
CV_REGISTER(DIL, 325)
CV_REGISTER(BPL, 326)
CV_REGISTER(SPL, 327)

CV_REGISTER(RAX, 328)
CV_REGISTER(RBX, 329)
CV_REGISTER(RCX, 330)
CV_REGISTER(RDX, 331)
CV_REGISTER(RSI, 332)
CV_REGISTER(RDI, 333)
CV_REGISTER(RBP, 334)
CV_REGISTER(RSP, 335)

CV_REGISTER(R8, 336)
CV_REGISTER(R9, 337)
CV_REGISTER(R10, 338)
CV_REGISTER(R11, 339)
CV_REGISTER(R12, 340)
CV_REGISTER(R13, 341)
CV_REGISTER(R14, 342)
CV_REGISTER(R15, 343)

CV_REGISTER(R8B, 344)
CV_REGISTER(R9B, 345)
````
- **L217 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L217 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L218 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L218 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L219 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L219 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L220 EN**: Blank line separating nearby declarations or logic blocks.
  **L220 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L221 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L221 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L222 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L222 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L223 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L223 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L224 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L224 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L225 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L225 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L226 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L226 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L227 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L227 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L228 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L228 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L229 EN**: Blank line separating nearby declarations or logic blocks.
  **L229 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L230 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L230 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L231 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L231 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L232 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L232 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L233 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L233 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L234 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L234 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L235 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L235 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L236 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L236 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L237 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L237 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L238 EN**: Blank line separating nearby declarations or logic blocks.
  **L238 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L239 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L239 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L240 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L240 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。

### Lines 241-264

````cpp
CV_REGISTER(R10B, 346)
CV_REGISTER(R11B, 347)
CV_REGISTER(R12B, 348)
CV_REGISTER(R13B, 349)
CV_REGISTER(R14B, 350)
CV_REGISTER(R15B, 351)

CV_REGISTER(R8W, 352)
CV_REGISTER(R9W, 353)
CV_REGISTER(R10W, 354)
CV_REGISTER(R11W, 355)
CV_REGISTER(R12W, 356)
CV_REGISTER(R13W, 357)
CV_REGISTER(R14W, 358)
CV_REGISTER(R15W, 359)

CV_REGISTER(R8D, 360)
CV_REGISTER(R9D, 361)
CV_REGISTER(R10D, 362)
CV_REGISTER(R11D, 363)
CV_REGISTER(R12D, 364)
CV_REGISTER(R13D, 365)
CV_REGISTER(R14D, 366)
CV_REGISTER(R15D, 367)
````
- **L241 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L241 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L242 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L242 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L243 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L243 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L244 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L244 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L245 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L245 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L246 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L246 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L247 EN**: Blank line separating nearby declarations or logic blocks.
  **L247 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L248 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L248 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L249 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L249 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L250 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L250 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L251 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L251 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L252 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L252 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L253 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L253 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L254 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L254 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L255 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L255 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L256 EN**: Blank line separating nearby declarations or logic blocks.
  **L256 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L257 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L257 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L258 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L258 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L259 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L259 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L260 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L260 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L261 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L261 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L262 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L262 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L263 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L263 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L264 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L264 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。

### Lines 265-288

````cpp


// cvconst.h defines both CV_REG_YMM0 (252) and CV_AMD64_YMM0 (368). Keep the
// original prefix to distinguish them.

CV_REGISTER(AMD64_YMM0, 368)
CV_REGISTER(AMD64_YMM1, 369)
CV_REGISTER(AMD64_YMM2, 370)
CV_REGISTER(AMD64_YMM3, 371)
CV_REGISTER(AMD64_YMM4, 372)
CV_REGISTER(AMD64_YMM5, 373)
CV_REGISTER(AMD64_YMM6, 374)
CV_REGISTER(AMD64_YMM7, 375)
CV_REGISTER(AMD64_YMM8, 376)
CV_REGISTER(AMD64_YMM9, 377)
CV_REGISTER(AMD64_YMM10, 378)
CV_REGISTER(AMD64_YMM11, 379)
CV_REGISTER(AMD64_YMM12, 380)
CV_REGISTER(AMD64_YMM13, 381)
CV_REGISTER(AMD64_YMM14, 382)
CV_REGISTER(AMD64_YMM15, 383)

CV_REGISTER(AMD64_XMM16, 694)
CV_REGISTER(AMD64_XMM17, 695)
````
- **L265 EN**: Blank line separating nearby declarations or logic blocks.
  **L265 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L266 EN**: Blank line separating nearby declarations or logic blocks.
  **L266 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L267 EN**: Comment explains nearby logic, invariants, or intent: `cvconst.h defines both CV_REG_YMM0 (252) and CV_AMD64_YMM0 (368). Keep the`.
  **L267 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`cvconst.h defines both CV_REG_YMM0 (252) and CV_AMD64_YMM0 (368). Keep the`。
- **L268 EN**: Comment explains nearby logic, invariants, or intent: `original prefix to distinguish them.`.
  **L268 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`original prefix to distinguish them.`。
- **L269 EN**: Blank line separating nearby declarations or logic blocks.
  **L269 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L270 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L270 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L271 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L271 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L272 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L272 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L273 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L273 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L274 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L274 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L275 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L275 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L276 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L276 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L277 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L277 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L278 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L278 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L279 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L279 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L280 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L280 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L281 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L281 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L282 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L282 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L283 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L283 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L284 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L284 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L285 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L285 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L286 EN**: Blank line separating nearby declarations or logic blocks.
  **L286 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L287 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L287 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L288 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L288 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。

### Lines 289-312

````cpp
CV_REGISTER(AMD64_XMM18, 696)
CV_REGISTER(AMD64_XMM19, 697)
CV_REGISTER(AMD64_XMM20, 698)
CV_REGISTER(AMD64_XMM21, 699)
CV_REGISTER(AMD64_XMM22, 700)
CV_REGISTER(AMD64_XMM23, 701)
CV_REGISTER(AMD64_XMM24, 702)
CV_REGISTER(AMD64_XMM25, 703)
CV_REGISTER(AMD64_XMM26, 704)
CV_REGISTER(AMD64_XMM27, 705)
CV_REGISTER(AMD64_XMM28, 706)
CV_REGISTER(AMD64_XMM29, 707)
CV_REGISTER(AMD64_XMM30, 708)
CV_REGISTER(AMD64_XMM31, 709)

CV_REGISTER(AMD64_YMM16, 710)
CV_REGISTER(AMD64_YMM17, 711)
CV_REGISTER(AMD64_YMM18, 712)
CV_REGISTER(AMD64_YMM19, 713)
CV_REGISTER(AMD64_YMM20, 714)
CV_REGISTER(AMD64_YMM21, 715)
CV_REGISTER(AMD64_YMM22, 716)
CV_REGISTER(AMD64_YMM23, 717)
CV_REGISTER(AMD64_YMM24, 718)
````
- **L289 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L289 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L290 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L290 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L291 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L291 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L292 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L292 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L293 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L293 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L294 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L294 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L295 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L295 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L296 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L296 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L297 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L297 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L298 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L298 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L299 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L299 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L300 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L300 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L301 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L301 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L302 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L302 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L303 EN**: Blank line separating nearby declarations or logic blocks.
  **L303 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L304 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L304 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L305 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L305 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L306 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L306 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L307 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L307 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L308 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L308 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L309 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L309 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L310 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L310 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L311 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L311 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L312 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L312 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。

### Lines 313-336

````cpp
CV_REGISTER(AMD64_YMM25, 719)
CV_REGISTER(AMD64_YMM26, 720)
CV_REGISTER(AMD64_YMM27, 721)
CV_REGISTER(AMD64_YMM28, 722)
CV_REGISTER(AMD64_YMM29, 723)
CV_REGISTER(AMD64_YMM30, 724)
CV_REGISTER(AMD64_YMM31, 725)

CV_REGISTER(AMD64_ZMM0, 726)
CV_REGISTER(AMD64_ZMM1, 727)
CV_REGISTER(AMD64_ZMM2, 728)
CV_REGISTER(AMD64_ZMM3, 729)
CV_REGISTER(AMD64_ZMM4, 730)
CV_REGISTER(AMD64_ZMM5, 731)
CV_REGISTER(AMD64_ZMM6, 732)
CV_REGISTER(AMD64_ZMM7, 733)
CV_REGISTER(AMD64_ZMM8, 734)
CV_REGISTER(AMD64_ZMM9, 735)
CV_REGISTER(AMD64_ZMM10, 736)
CV_REGISTER(AMD64_ZMM11, 737)
CV_REGISTER(AMD64_ZMM12, 738)
CV_REGISTER(AMD64_ZMM13, 739)
CV_REGISTER(AMD64_ZMM14, 740)
CV_REGISTER(AMD64_ZMM15, 741)
````
- **L313 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L313 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L314 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L314 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L315 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L315 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L316 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L316 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L317 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L317 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L318 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L318 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L319 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L319 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L320 EN**: Blank line separating nearby declarations or logic blocks.
  **L320 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L321 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L321 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L322 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L322 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L323 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L323 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L324 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L324 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L325 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L325 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L326 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L326 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L327 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L327 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L328 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L328 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L329 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L329 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L330 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L330 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L331 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L331 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L332 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L332 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L333 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L333 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L334 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L334 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L335 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L335 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L336 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L336 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。

### Lines 337-360

````cpp
CV_REGISTER(AMD64_ZMM16, 742)
CV_REGISTER(AMD64_ZMM17, 743)
CV_REGISTER(AMD64_ZMM18, 744)
CV_REGISTER(AMD64_ZMM19, 745)
CV_REGISTER(AMD64_ZMM20, 746)
CV_REGISTER(AMD64_ZMM21, 747)
CV_REGISTER(AMD64_ZMM22, 748)
CV_REGISTER(AMD64_ZMM23, 749)
CV_REGISTER(AMD64_ZMM24, 750)
CV_REGISTER(AMD64_ZMM25, 751)
CV_REGISTER(AMD64_ZMM26, 752)
CV_REGISTER(AMD64_ZMM27, 753)
CV_REGISTER(AMD64_ZMM28, 754)
CV_REGISTER(AMD64_ZMM29, 755)
CV_REGISTER(AMD64_ZMM30, 756)
CV_REGISTER(AMD64_ZMM31, 757)

CV_REGISTER(AMD64_K0, 758)
CV_REGISTER(AMD64_K1, 759)
CV_REGISTER(AMD64_K2, 760)
CV_REGISTER(AMD64_K3, 761)
CV_REGISTER(AMD64_K4, 762)
CV_REGISTER(AMD64_K5, 763)
CV_REGISTER(AMD64_K6, 764)
````
- **L337 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L337 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L338 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L338 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L339 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L339 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L340 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L340 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L341 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L341 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L342 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L342 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L343 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L343 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L344 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L344 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L345 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L345 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L346 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L346 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L347 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L347 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L348 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L348 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L349 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L349 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L350 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L350 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L351 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L351 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L352 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L352 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L353 EN**: Blank line separating nearby declarations or logic blocks.
  **L353 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L354 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L354 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L355 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L355 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L356 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L356 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L357 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L357 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L358 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L358 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L359 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L359 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L360 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L360 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。

### Lines 361-384

````cpp
CV_REGISTER(AMD64_K7, 765)

#pragma pop_macro("CR0")
#pragma pop_macro("CR1")
#pragma pop_macro("CR2")
#pragma pop_macro("CR3")
#pragma pop_macro("CR4")

#endif // defined(CV_REGISTERS_ALL) || defined(CV_REGISTERS_X86)

#if defined(CV_REGISTERS_ALL) || defined(CV_REGISTERS_ARM)

// ARM registers

CV_REGISTER(ARM_NOREG, 0)

// General purpose 32-bit integer registers

CV_REGISTER(ARM_R0, 10)
CV_REGISTER(ARM_R1, 11)
CV_REGISTER(ARM_R2, 12)
CV_REGISTER(ARM_R3, 13)
CV_REGISTER(ARM_R4, 14)
CV_REGISTER(ARM_R5, 15)
````
- **L361 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L361 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L362 EN**: Blank line separating nearby declarations or logic blocks.
  **L362 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L363 EN**: Continues logic associated with callable symbol `pop_macro`.
  **L363 CN**: 继续与可调用符号 `pop_macro` 相关的逻辑。
- **L364 EN**: Continues logic associated with callable symbol `pop_macro`.
  **L364 CN**: 继续与可调用符号 `pop_macro` 相关的逻辑。
- **L365 EN**: Continues logic associated with callable symbol `pop_macro`.
  **L365 CN**: 继续与可调用符号 `pop_macro` 相关的逻辑。
- **L366 EN**: Continues logic associated with callable symbol `pop_macro`.
  **L366 CN**: 继续与可调用符号 `pop_macro` 相关的逻辑。
- **L367 EN**: Continues logic associated with callable symbol `pop_macro`.
  **L367 CN**: 继续与可调用符号 `pop_macro` 相关的逻辑。
- **L368 EN**: Blank line separating nearby declarations or logic blocks.
  **L368 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L369 EN**: Closes the current preprocessor conditional block.
  **L369 CN**: 结束当前预处理条件块。
- **L370 EN**: Blank line separating nearby declarations or logic blocks.
  **L370 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L371 EN**: Starts a preprocessor conditional block: `#if defined(CV_REGISTERS_ALL) || defined(CV_REGISTERS_ARM)`.
  **L371 CN**: 开始一个预处理条件块：`#if defined(CV_REGISTERS_ALL) || defined(CV_REGISTERS_ARM)`。
- **L372 EN**: Blank line separating nearby declarations or logic blocks.
  **L372 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L373 EN**: Comment explains nearby logic, invariants, or intent: `ARM registers`.
  **L373 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`ARM registers`。
- **L374 EN**: Blank line separating nearby declarations or logic blocks.
  **L374 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L375 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L375 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L376 EN**: Blank line separating nearby declarations or logic blocks.
  **L376 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L377 EN**: Comment explains nearby logic, invariants, or intent: `General purpose 32-bit integer registers`.
  **L377 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`General purpose 32-bit integer registers`。
- **L378 EN**: Blank line separating nearby declarations or logic blocks.
  **L378 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L379 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L379 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L380 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L380 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L381 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L381 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L382 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L382 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L383 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L383 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L384 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L384 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。

### Lines 385-408

````cpp
CV_REGISTER(ARM_R6, 16)
CV_REGISTER(ARM_R7, 17)
CV_REGISTER(ARM_R8, 18)
CV_REGISTER(ARM_R9, 19)
CV_REGISTER(ARM_R10, 20)
CV_REGISTER(ARM_R11, 21)
CV_REGISTER(ARM_R12, 22)
CV_REGISTER(ARM_SP, 23)
CV_REGISTER(ARM_LR, 24)
CV_REGISTER(ARM_PC, 25)

// Status register

CV_REGISTER(ARM_CPSR, 26)

// ARM VFPv1 registers

CV_REGISTER(ARM_FPSCR, 40)
CV_REGISTER(ARM_FPEXC, 41)

CV_REGISTER(ARM_FS0, 50)
CV_REGISTER(ARM_FS1, 51)
CV_REGISTER(ARM_FS2, 52)
CV_REGISTER(ARM_FS3, 53)
````
- **L385 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L385 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L386 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L386 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L387 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L387 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L388 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L388 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L389 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L389 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L390 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L390 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L391 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L391 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L392 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L392 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L393 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L393 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L394 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L394 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L395 EN**: Blank line separating nearby declarations or logic blocks.
  **L395 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L396 EN**: Comment explains nearby logic, invariants, or intent: `Status register`.
  **L396 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Status register`。
- **L397 EN**: Blank line separating nearby declarations or logic blocks.
  **L397 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L398 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L398 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L399 EN**: Blank line separating nearby declarations or logic blocks.
  **L399 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L400 EN**: Comment explains nearby logic, invariants, or intent: `ARM VFPv1 registers`.
  **L400 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`ARM VFPv1 registers`。
- **L401 EN**: Blank line separating nearby declarations or logic blocks.
  **L401 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L402 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L402 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L403 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L403 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L404 EN**: Blank line separating nearby declarations or logic blocks.
  **L404 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L405 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L405 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L406 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L406 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L407 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L407 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L408 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L408 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。

### Lines 409-432

````cpp
CV_REGISTER(ARM_FS4, 54)
CV_REGISTER(ARM_FS5, 55)
CV_REGISTER(ARM_FS6, 56)
CV_REGISTER(ARM_FS7, 57)
CV_REGISTER(ARM_FS8, 58)
CV_REGISTER(ARM_FS9, 59)
CV_REGISTER(ARM_FS10, 60)
CV_REGISTER(ARM_FS11, 61)
CV_REGISTER(ARM_FS12, 62)
CV_REGISTER(ARM_FS13, 63)
CV_REGISTER(ARM_FS14, 64)
CV_REGISTER(ARM_FS15, 65)
CV_REGISTER(ARM_FS16, 66)
CV_REGISTER(ARM_FS17, 67)
CV_REGISTER(ARM_FS18, 68)
CV_REGISTER(ARM_FS19, 69)
CV_REGISTER(ARM_FS20, 70)
CV_REGISTER(ARM_FS21, 71)
CV_REGISTER(ARM_FS22, 72)
CV_REGISTER(ARM_FS23, 73)
CV_REGISTER(ARM_FS24, 74)
CV_REGISTER(ARM_FS25, 75)
CV_REGISTER(ARM_FS26, 76)
CV_REGISTER(ARM_FS27, 77)
````
- **L409 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L409 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L410 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L410 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L411 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L411 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L412 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L412 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L413 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L413 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L414 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L414 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L415 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L415 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L416 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L416 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L417 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L417 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L418 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L418 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L419 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L419 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L420 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L420 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L421 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L421 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L422 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L422 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L423 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L423 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L424 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L424 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L425 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L425 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L426 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L426 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L427 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L427 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L428 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L428 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L429 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L429 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L430 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L430 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L431 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L431 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L432 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L432 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。

### Lines 433-456

````cpp
CV_REGISTER(ARM_FS28, 78)
CV_REGISTER(ARM_FS29, 79)
CV_REGISTER(ARM_FS30, 80)
CV_REGISTER(ARM_FS31, 81)

// ARM VFPv3/NEON registers

CV_REGISTER(ARM_FS32, 200)
CV_REGISTER(ARM_FS33, 201)
CV_REGISTER(ARM_FS34, 202)
CV_REGISTER(ARM_FS35, 203)
CV_REGISTER(ARM_FS36, 204)
CV_REGISTER(ARM_FS37, 205)
CV_REGISTER(ARM_FS38, 206)
CV_REGISTER(ARM_FS39, 207)
CV_REGISTER(ARM_FS40, 208)
CV_REGISTER(ARM_FS41, 209)
CV_REGISTER(ARM_FS42, 210)
CV_REGISTER(ARM_FS43, 211)
CV_REGISTER(ARM_FS44, 212)
CV_REGISTER(ARM_FS45, 213)
CV_REGISTER(ARM_FS46, 214)
CV_REGISTER(ARM_FS47, 215)
CV_REGISTER(ARM_FS48, 216)
````
- **L433 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L433 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L434 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L434 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L435 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L435 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L436 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L436 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L437 EN**: Blank line separating nearby declarations or logic blocks.
  **L437 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L438 EN**: Comment explains nearby logic, invariants, or intent: `ARM VFPv3/NEON registers`.
  **L438 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`ARM VFPv3/NEON registers`。
- **L439 EN**: Blank line separating nearby declarations or logic blocks.
  **L439 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L440 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L440 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L441 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L441 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L442 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L442 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L443 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L443 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L444 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L444 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L445 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L445 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L446 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L446 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L447 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L447 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L448 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L448 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L449 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L449 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L450 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L450 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L451 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L451 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L452 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L452 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L453 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L453 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L454 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L454 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L455 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L455 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L456 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L456 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。

### Lines 457-480

````cpp
CV_REGISTER(ARM_FS49, 217)
CV_REGISTER(ARM_FS50, 218)
CV_REGISTER(ARM_FS51, 219)
CV_REGISTER(ARM_FS52, 220)
CV_REGISTER(ARM_FS53, 221)
CV_REGISTER(ARM_FS54, 222)
CV_REGISTER(ARM_FS55, 223)
CV_REGISTER(ARM_FS56, 224)
CV_REGISTER(ARM_FS57, 225)
CV_REGISTER(ARM_FS58, 226)
CV_REGISTER(ARM_FS59, 227)
CV_REGISTER(ARM_FS60, 228)
CV_REGISTER(ARM_FS61, 229)
CV_REGISTER(ARM_FS62, 230)
CV_REGISTER(ARM_FS63, 231)

CV_REGISTER(ARM_ND0, 300)
CV_REGISTER(ARM_ND1, 301)
CV_REGISTER(ARM_ND2, 302)
CV_REGISTER(ARM_ND3, 303)
CV_REGISTER(ARM_ND4, 304)
CV_REGISTER(ARM_ND5, 305)
CV_REGISTER(ARM_ND6, 306)
CV_REGISTER(ARM_ND7, 307)
````
- **L457 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L457 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L458 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L458 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L459 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L459 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L460 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L460 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L461 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L461 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L462 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L462 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L463 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L463 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L464 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L464 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L465 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L465 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L466 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L466 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L467 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L467 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L468 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L468 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L469 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L469 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L470 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L470 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L471 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L471 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L472 EN**: Blank line separating nearby declarations or logic blocks.
  **L472 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L473 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L473 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L474 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L474 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L475 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L475 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L476 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L476 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L477 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L477 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L478 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L478 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L479 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L479 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L480 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L480 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。

### Lines 481-504

````cpp
CV_REGISTER(ARM_ND8, 308)
CV_REGISTER(ARM_ND9, 309)
CV_REGISTER(ARM_ND10, 310)
CV_REGISTER(ARM_ND11, 311)
CV_REGISTER(ARM_ND12, 312)
CV_REGISTER(ARM_ND13, 313)
CV_REGISTER(ARM_ND14, 314)
CV_REGISTER(ARM_ND15, 315)
CV_REGISTER(ARM_ND16, 316)
CV_REGISTER(ARM_ND17, 317)
CV_REGISTER(ARM_ND18, 318)
CV_REGISTER(ARM_ND19, 319)
CV_REGISTER(ARM_ND20, 320)
CV_REGISTER(ARM_ND21, 321)
CV_REGISTER(ARM_ND22, 322)
CV_REGISTER(ARM_ND23, 323)
CV_REGISTER(ARM_ND24, 324)
CV_REGISTER(ARM_ND25, 325)
CV_REGISTER(ARM_ND26, 326)
CV_REGISTER(ARM_ND27, 327)
CV_REGISTER(ARM_ND28, 328)
CV_REGISTER(ARM_ND29, 329)
CV_REGISTER(ARM_ND30, 330)
CV_REGISTER(ARM_ND31, 331)
````
- **L481 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L481 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L482 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L482 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L483 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L483 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L484 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L484 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L485 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L485 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L486 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L486 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L487 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L487 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L488 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L488 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L489 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L489 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L490 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L490 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L491 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L491 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L492 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L492 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L493 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L493 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L494 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L494 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L495 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L495 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L496 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L496 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L497 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L497 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L498 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L498 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L499 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L499 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L500 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L500 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L501 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L501 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L502 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L502 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L503 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L503 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L504 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L504 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。

### Lines 505-528

````cpp

CV_REGISTER(ARM_NQ0, 400)
CV_REGISTER(ARM_NQ1, 401)
CV_REGISTER(ARM_NQ2, 402)
CV_REGISTER(ARM_NQ3, 403)
CV_REGISTER(ARM_NQ4, 404)
CV_REGISTER(ARM_NQ5, 405)
CV_REGISTER(ARM_NQ6, 406)
CV_REGISTER(ARM_NQ7, 407)
CV_REGISTER(ARM_NQ8, 408)
CV_REGISTER(ARM_NQ9, 409)
CV_REGISTER(ARM_NQ10, 410)
CV_REGISTER(ARM_NQ11, 411)
CV_REGISTER(ARM_NQ12, 412)
CV_REGISTER(ARM_NQ13, 413)
CV_REGISTER(ARM_NQ14, 414)
CV_REGISTER(ARM_NQ15, 415)

#endif // defined(CV_REGISTERS_ALL) || defined(CV_REGISTERS_ARM)

#if defined(CV_REGISTERS_ALL) || defined(CV_REGISTERS_ARM64)

// arm64intr.h from MSVC defines ARM64_FPSR and ARM64_FPCR, which conflicts with
// these declarations.
````
- **L505 EN**: Blank line separating nearby declarations or logic blocks.
  **L505 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L506 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L506 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L507 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L507 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L508 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L508 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L509 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L509 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L510 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L510 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L511 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L511 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L512 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L512 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L513 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L513 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L514 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L514 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L515 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L515 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L516 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L516 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L517 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L517 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L518 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L518 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L519 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L519 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L520 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L520 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L521 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L521 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L522 EN**: Blank line separating nearby declarations or logic blocks.
  **L522 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L523 EN**: Closes the current preprocessor conditional block.
  **L523 CN**: 结束当前预处理条件块。
- **L524 EN**: Blank line separating nearby declarations or logic blocks.
  **L524 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L525 EN**: Starts a preprocessor conditional block: `#if defined(CV_REGISTERS_ALL) || defined(CV_REGISTERS_ARM64)`.
  **L525 CN**: 开始一个预处理条件块：`#if defined(CV_REGISTERS_ALL) || defined(CV_REGISTERS_ARM64)`。
- **L526 EN**: Blank line separating nearby declarations or logic blocks.
  **L526 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L527 EN**: Comment explains nearby logic, invariants, or intent: `arm64intr.h from MSVC defines ARM64_FPSR and ARM64_FPCR, which conflicts with`.
  **L527 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`arm64intr.h from MSVC defines ARM64_FPSR and ARM64_FPCR, which conflicts with`。
- **L528 EN**: Comment explains nearby logic, invariants, or intent: `these declarations.`.
  **L528 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`these declarations.`。

### Lines 529-552

````cpp
#pragma push_macro("ARM64_FPSR")
#pragma push_macro("ARM64_FPCR")
#undef ARM64_FPSR
#undef ARM64_FPCR

// ARM64 registers

CV_REGISTER(ARM64_NOREG, 0)

// General purpose 32-bit integer registers

CV_REGISTER(ARM64_W0, 10)
CV_REGISTER(ARM64_W1, 11)
CV_REGISTER(ARM64_W2, 12)
CV_REGISTER(ARM64_W3, 13)
CV_REGISTER(ARM64_W4, 14)
CV_REGISTER(ARM64_W5, 15)
CV_REGISTER(ARM64_W6, 16)
CV_REGISTER(ARM64_W7, 17)
CV_REGISTER(ARM64_W8, 18)
CV_REGISTER(ARM64_W9, 19)
CV_REGISTER(ARM64_W10, 20)
CV_REGISTER(ARM64_W11, 21)
CV_REGISTER(ARM64_W12, 22)
````
- **L529 EN**: Continues logic associated with callable symbol `push_macro`.
  **L529 CN**: 继续与可调用符号 `push_macro` 相关的逻辑。
- **L530 EN**: Continues logic associated with callable symbol `push_macro`.
  **L530 CN**: 继续与可调用符号 `push_macro` 相关的逻辑。
- **L531 EN**: Undefines a macro to limit its scope: `#undef ARM64_FPSR`.
  **L531 CN**: 取消宏定义以限制其作用域：`#undef ARM64_FPSR`。
- **L532 EN**: Undefines a macro to limit its scope: `#undef ARM64_FPCR`.
  **L532 CN**: 取消宏定义以限制其作用域：`#undef ARM64_FPCR`。
- **L533 EN**: Blank line separating nearby declarations or logic blocks.
  **L533 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L534 EN**: Comment explains nearby logic, invariants, or intent: `ARM64 registers`.
  **L534 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`ARM64 registers`。
- **L535 EN**: Blank line separating nearby declarations or logic blocks.
  **L535 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L536 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L536 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L537 EN**: Blank line separating nearby declarations or logic blocks.
  **L537 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L538 EN**: Comment explains nearby logic, invariants, or intent: `General purpose 32-bit integer registers`.
  **L538 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`General purpose 32-bit integer registers`。
- **L539 EN**: Blank line separating nearby declarations or logic blocks.
  **L539 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L540 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L540 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L541 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L541 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L542 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L542 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L543 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L543 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L544 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L544 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L545 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L545 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L546 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L546 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L547 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L547 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L548 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L548 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L549 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L549 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L550 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L550 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L551 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L551 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L552 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L552 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。

### Lines 553-576

````cpp
CV_REGISTER(ARM64_W13, 23)
CV_REGISTER(ARM64_W14, 24)
CV_REGISTER(ARM64_W15, 25)
CV_REGISTER(ARM64_W16, 26)
CV_REGISTER(ARM64_W17, 27)
CV_REGISTER(ARM64_W18, 28)
CV_REGISTER(ARM64_W19, 29)
CV_REGISTER(ARM64_W20, 30)
CV_REGISTER(ARM64_W21, 31)
CV_REGISTER(ARM64_W22, 32)
CV_REGISTER(ARM64_W23, 33)
CV_REGISTER(ARM64_W24, 34)
CV_REGISTER(ARM64_W25, 35)
CV_REGISTER(ARM64_W26, 36)
CV_REGISTER(ARM64_W27, 37)
CV_REGISTER(ARM64_W28, 38)
CV_REGISTER(ARM64_W29, 39)
CV_REGISTER(ARM64_W30, 40)
CV_REGISTER(ARM64_WZR, 41)

// General purpose 64-bit integer registers

CV_REGISTER(ARM64_X0, 50)
CV_REGISTER(ARM64_X1, 51)
````
- **L553 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L553 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L554 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L554 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L555 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L555 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L556 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L556 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L557 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L557 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L558 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L558 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L559 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L559 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L560 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L560 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L561 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L561 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L562 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L562 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L563 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L563 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L564 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L564 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L565 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L565 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L566 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L566 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L567 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L567 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L568 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L568 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L569 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L569 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L570 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L570 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L571 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L571 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L572 EN**: Blank line separating nearby declarations or logic blocks.
  **L572 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L573 EN**: Comment explains nearby logic, invariants, or intent: `General purpose 64-bit integer registers`.
  **L573 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`General purpose 64-bit integer registers`。
- **L574 EN**: Blank line separating nearby declarations or logic blocks.
  **L574 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L575 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L575 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L576 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L576 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。

### Lines 577-600

````cpp
CV_REGISTER(ARM64_X2, 52)
CV_REGISTER(ARM64_X3, 53)
CV_REGISTER(ARM64_X4, 54)
CV_REGISTER(ARM64_X5, 55)
CV_REGISTER(ARM64_X6, 56)
CV_REGISTER(ARM64_X7, 57)
CV_REGISTER(ARM64_X8, 58)
CV_REGISTER(ARM64_X9, 59)
CV_REGISTER(ARM64_X10, 60)
CV_REGISTER(ARM64_X11, 61)
CV_REGISTER(ARM64_X12, 62)
CV_REGISTER(ARM64_X13, 63)
CV_REGISTER(ARM64_X14, 64)
CV_REGISTER(ARM64_X15, 65)
CV_REGISTER(ARM64_X16, 66)
CV_REGISTER(ARM64_X17, 67)
CV_REGISTER(ARM64_X18, 68)
CV_REGISTER(ARM64_X19, 69)
CV_REGISTER(ARM64_X20, 70)
CV_REGISTER(ARM64_X21, 71)
CV_REGISTER(ARM64_X22, 72)
CV_REGISTER(ARM64_X23, 73)
CV_REGISTER(ARM64_X24, 74)
CV_REGISTER(ARM64_X25, 75)
````
- **L577 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L577 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L578 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L578 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L579 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L579 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L580 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L580 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L581 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L581 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L582 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L582 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L583 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L583 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L584 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L584 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L585 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L585 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L586 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L586 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L587 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L587 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L588 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L588 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L589 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L589 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L590 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L590 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L591 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L591 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L592 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L592 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L593 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L593 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L594 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L594 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L595 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L595 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L596 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L596 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L597 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L597 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L598 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L598 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L599 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L599 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L600 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L600 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。

### Lines 601-624

````cpp
CV_REGISTER(ARM64_X26, 76)
CV_REGISTER(ARM64_X27, 77)
CV_REGISTER(ARM64_X28, 78)
CV_REGISTER(ARM64_FP, 79)
CV_REGISTER(ARM64_LR, 80)
CV_REGISTER(ARM64_SP, 81)
CV_REGISTER(ARM64_ZR, 82)

// status register

CV_REGISTER(ARM64_NZCV, 90)

// 32-bit floating point registers

CV_REGISTER(ARM64_S0, 100)
CV_REGISTER(ARM64_S1, 101)
CV_REGISTER(ARM64_S2, 102)
CV_REGISTER(ARM64_S3, 103)
CV_REGISTER(ARM64_S4, 104)
CV_REGISTER(ARM64_S5, 105)
CV_REGISTER(ARM64_S6, 106)
CV_REGISTER(ARM64_S7, 107)
CV_REGISTER(ARM64_S8, 108)
CV_REGISTER(ARM64_S9, 109)
````
- **L601 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L601 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L602 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L602 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L603 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L603 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L604 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L604 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L605 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L605 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L606 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L606 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L607 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L607 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L608 EN**: Blank line separating nearby declarations or logic blocks.
  **L608 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L609 EN**: Comment explains nearby logic, invariants, or intent: `status register`.
  **L609 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`status register`。
- **L610 EN**: Blank line separating nearby declarations or logic blocks.
  **L610 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L611 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L611 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L612 EN**: Blank line separating nearby declarations or logic blocks.
  **L612 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L613 EN**: Comment explains nearby logic, invariants, or intent: `32-bit floating point registers`.
  **L613 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`32-bit floating point registers`。
- **L614 EN**: Blank line separating nearby declarations or logic blocks.
  **L614 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L615 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L615 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L616 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L616 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L617 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L617 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L618 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L618 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L619 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L619 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L620 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L620 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L621 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L621 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L622 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L622 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L623 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L623 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L624 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L624 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。

### Lines 625-648

````cpp
CV_REGISTER(ARM64_S10, 110)
CV_REGISTER(ARM64_S11, 111)
CV_REGISTER(ARM64_S12, 112)
CV_REGISTER(ARM64_S13, 113)
CV_REGISTER(ARM64_S14, 114)
CV_REGISTER(ARM64_S15, 115)
CV_REGISTER(ARM64_S16, 116)
CV_REGISTER(ARM64_S17, 117)
CV_REGISTER(ARM64_S18, 118)
CV_REGISTER(ARM64_S19, 119)
CV_REGISTER(ARM64_S20, 120)
CV_REGISTER(ARM64_S21, 121)
CV_REGISTER(ARM64_S22, 122)
CV_REGISTER(ARM64_S23, 123)
CV_REGISTER(ARM64_S24, 124)
CV_REGISTER(ARM64_S25, 125)
CV_REGISTER(ARM64_S26, 126)
CV_REGISTER(ARM64_S27, 127)
CV_REGISTER(ARM64_S28, 128)
CV_REGISTER(ARM64_S29, 129)
CV_REGISTER(ARM64_S30, 130)
CV_REGISTER(ARM64_S31, 131)

// 64-bit floating point registers
````
- **L625 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L625 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L626 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L626 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L627 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L627 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L628 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L628 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L629 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L629 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L630 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L630 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L631 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L631 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L632 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L632 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L633 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L633 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L634 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L634 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L635 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L635 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L636 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L636 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L637 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L637 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L638 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L638 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L639 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L639 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L640 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L640 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L641 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L641 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L642 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L642 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L643 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L643 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L644 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L644 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L645 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L645 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L646 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L646 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L647 EN**: Blank line separating nearby declarations or logic blocks.
  **L647 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L648 EN**: Comment explains nearby logic, invariants, or intent: `64-bit floating point registers`.
  **L648 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`64-bit floating point registers`。

### Lines 649-672

````cpp

CV_REGISTER(ARM64_D0, 140)
CV_REGISTER(ARM64_D1, 141)
CV_REGISTER(ARM64_D2, 142)
CV_REGISTER(ARM64_D3, 143)
CV_REGISTER(ARM64_D4, 144)
CV_REGISTER(ARM64_D5, 145)
CV_REGISTER(ARM64_D6, 146)
CV_REGISTER(ARM64_D7, 147)
CV_REGISTER(ARM64_D8, 148)
CV_REGISTER(ARM64_D9, 149)
CV_REGISTER(ARM64_D10, 150)
CV_REGISTER(ARM64_D11, 151)
CV_REGISTER(ARM64_D12, 152)
CV_REGISTER(ARM64_D13, 153)
CV_REGISTER(ARM64_D14, 154)
CV_REGISTER(ARM64_D15, 155)
CV_REGISTER(ARM64_D16, 156)
CV_REGISTER(ARM64_D17, 157)
CV_REGISTER(ARM64_D18, 158)
CV_REGISTER(ARM64_D19, 159)
CV_REGISTER(ARM64_D20, 160)
CV_REGISTER(ARM64_D21, 161)
CV_REGISTER(ARM64_D22, 162)
````
- **L649 EN**: Blank line separating nearby declarations or logic blocks.
  **L649 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L650 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L650 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L651 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L651 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L652 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L652 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L653 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L653 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L654 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L654 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L655 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L655 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L656 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L656 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L657 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L657 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L658 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L658 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L659 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L659 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L660 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L660 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L661 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L661 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L662 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L662 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L663 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L663 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L664 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L664 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L665 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L665 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L666 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L666 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L667 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L667 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L668 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L668 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L669 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L669 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L670 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L670 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L671 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L671 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L672 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L672 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。

### Lines 673-696

````cpp
CV_REGISTER(ARM64_D23, 163)
CV_REGISTER(ARM64_D24, 164)
CV_REGISTER(ARM64_D25, 165)
CV_REGISTER(ARM64_D26, 166)
CV_REGISTER(ARM64_D27, 167)
CV_REGISTER(ARM64_D28, 168)
CV_REGISTER(ARM64_D29, 169)
CV_REGISTER(ARM64_D30, 170)
CV_REGISTER(ARM64_D31, 171)

// 128-bit SIMD registers

CV_REGISTER(ARM64_Q0, 180)
CV_REGISTER(ARM64_Q1, 181)
CV_REGISTER(ARM64_Q2, 182)
CV_REGISTER(ARM64_Q3, 183)
CV_REGISTER(ARM64_Q4, 184)
CV_REGISTER(ARM64_Q5, 185)
CV_REGISTER(ARM64_Q6, 186)
CV_REGISTER(ARM64_Q7, 187)
CV_REGISTER(ARM64_Q8, 188)
CV_REGISTER(ARM64_Q9, 189)
CV_REGISTER(ARM64_Q10, 190)
CV_REGISTER(ARM64_Q11, 191)
````
- **L673 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L673 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L674 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L674 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L675 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L675 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L676 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L676 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L677 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L677 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L678 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L678 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L679 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L679 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L680 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L680 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L681 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L681 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L682 EN**: Blank line separating nearby declarations or logic blocks.
  **L682 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L683 EN**: Comment explains nearby logic, invariants, or intent: `128-bit SIMD registers`.
  **L683 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`128-bit SIMD registers`。
- **L684 EN**: Blank line separating nearby declarations or logic blocks.
  **L684 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L685 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L685 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L686 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L686 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L687 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L687 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L688 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L688 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L689 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L689 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L690 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L690 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L691 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L691 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L692 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L692 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L693 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L693 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L694 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L694 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L695 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L695 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L696 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L696 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。

### Lines 697-720

````cpp
CV_REGISTER(ARM64_Q12, 192)
CV_REGISTER(ARM64_Q13, 193)
CV_REGISTER(ARM64_Q14, 194)
CV_REGISTER(ARM64_Q15, 195)
CV_REGISTER(ARM64_Q16, 196)
CV_REGISTER(ARM64_Q17, 197)
CV_REGISTER(ARM64_Q18, 198)
CV_REGISTER(ARM64_Q19, 199)
CV_REGISTER(ARM64_Q20, 200)
CV_REGISTER(ARM64_Q21, 201)
CV_REGISTER(ARM64_Q22, 202)
CV_REGISTER(ARM64_Q23, 203)
CV_REGISTER(ARM64_Q24, 204)
CV_REGISTER(ARM64_Q25, 205)
CV_REGISTER(ARM64_Q26, 206)
CV_REGISTER(ARM64_Q27, 207)
CV_REGISTER(ARM64_Q28, 208)
CV_REGISTER(ARM64_Q29, 209)
CV_REGISTER(ARM64_Q30, 210)
CV_REGISTER(ARM64_Q31, 211)

// Floating point status register

CV_REGISTER(ARM64_FPSR, 220)
````
- **L697 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L697 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L698 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L698 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L699 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L699 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L700 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L700 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L701 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L701 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L702 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L702 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L703 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L703 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L704 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L704 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L705 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L705 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L706 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L706 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L707 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L707 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L708 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L708 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L709 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L709 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L710 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L710 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L711 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L711 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L712 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L712 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L713 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L713 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L714 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L714 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L715 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L715 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L716 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L716 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L717 EN**: Blank line separating nearby declarations or logic blocks.
  **L717 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L718 EN**: Comment explains nearby logic, invariants, or intent: `Floating point status register`.
  **L718 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Floating point status register`。
- **L719 EN**: Blank line separating nearby declarations or logic blocks.
  **L719 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L720 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L720 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。

### Lines 721-744

````cpp
CV_REGISTER(ARM64_FPCR, 221)

// 8 bit floating point registers

CV_REGISTER(ARM64_B0, 230)
CV_REGISTER(ARM64_B1, 231)
CV_REGISTER(ARM64_B2, 232)
CV_REGISTER(ARM64_B3, 233)
CV_REGISTER(ARM64_B4, 234)
CV_REGISTER(ARM64_B5, 235)
CV_REGISTER(ARM64_B6, 236)
CV_REGISTER(ARM64_B7, 237)
CV_REGISTER(ARM64_B8, 238)
CV_REGISTER(ARM64_B9, 239)
CV_REGISTER(ARM64_B10, 240)
CV_REGISTER(ARM64_B11, 241)
CV_REGISTER(ARM64_B12, 242)
CV_REGISTER(ARM64_B13, 243)
CV_REGISTER(ARM64_B14, 244)
CV_REGISTER(ARM64_B15, 245)
CV_REGISTER(ARM64_B16, 246)
CV_REGISTER(ARM64_B17, 247)
CV_REGISTER(ARM64_B18, 248)
CV_REGISTER(ARM64_B19, 249)
````
- **L721 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L721 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L722 EN**: Blank line separating nearby declarations or logic blocks.
  **L722 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L723 EN**: Comment explains nearby logic, invariants, or intent: `8 bit floating point registers`.
  **L723 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`8 bit floating point registers`。
- **L724 EN**: Blank line separating nearby declarations or logic blocks.
  **L724 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L725 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L725 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L726 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L726 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L727 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L727 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L728 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L728 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L729 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L729 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L730 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L730 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L731 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L731 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L732 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L732 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L733 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L733 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L734 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L734 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L735 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L735 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L736 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L736 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L737 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L737 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L738 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L738 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L739 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L739 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L740 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L740 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L741 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L741 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L742 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L742 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L743 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L743 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L744 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L744 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。

### Lines 745-768

````cpp
CV_REGISTER(ARM64_B20, 250)
CV_REGISTER(ARM64_B21, 251)
CV_REGISTER(ARM64_B22, 252)
CV_REGISTER(ARM64_B23, 253)
CV_REGISTER(ARM64_B24, 254)
CV_REGISTER(ARM64_B25, 255)
CV_REGISTER(ARM64_B26, 256)
CV_REGISTER(ARM64_B27, 257)
CV_REGISTER(ARM64_B28, 258)
CV_REGISTER(ARM64_B29, 259)
CV_REGISTER(ARM64_B30, 260)
CV_REGISTER(ARM64_B31, 261)

// 16 bit floating point registers

CV_REGISTER(ARM64_H0, 270)
CV_REGISTER(ARM64_H1, 271)
CV_REGISTER(ARM64_H2, 272)
CV_REGISTER(ARM64_H3, 273)
CV_REGISTER(ARM64_H4, 274)
CV_REGISTER(ARM64_H5, 275)
CV_REGISTER(ARM64_H6, 276)
CV_REGISTER(ARM64_H7, 277)
CV_REGISTER(ARM64_H8, 278)
````
- **L745 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L745 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L746 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L746 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L747 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L747 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L748 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L748 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L749 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L749 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L750 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L750 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L751 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L751 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L752 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L752 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L753 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L753 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L754 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L754 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L755 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L755 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L756 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L756 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L757 EN**: Blank line separating nearby declarations or logic blocks.
  **L757 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L758 EN**: Comment explains nearby logic, invariants, or intent: `16 bit floating point registers`.
  **L758 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`16 bit floating point registers`。
- **L759 EN**: Blank line separating nearby declarations or logic blocks.
  **L759 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L760 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L760 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L761 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L761 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L762 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L762 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L763 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L763 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L764 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L764 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L765 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L765 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L766 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L766 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L767 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L767 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L768 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L768 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。

### Lines 769-792

````cpp
CV_REGISTER(ARM64_H9, 279)
CV_REGISTER(ARM64_H10, 280)
CV_REGISTER(ARM64_H11, 281)
CV_REGISTER(ARM64_H12, 282)
CV_REGISTER(ARM64_H13, 283)
CV_REGISTER(ARM64_H14, 284)
CV_REGISTER(ARM64_H15, 285)
CV_REGISTER(ARM64_H16, 286)
CV_REGISTER(ARM64_H17, 287)
CV_REGISTER(ARM64_H18, 288)
CV_REGISTER(ARM64_H19, 289)
CV_REGISTER(ARM64_H20, 290)
CV_REGISTER(ARM64_H21, 291)
CV_REGISTER(ARM64_H22, 292)
CV_REGISTER(ARM64_H23, 293)
CV_REGISTER(ARM64_H24, 294)
CV_REGISTER(ARM64_H25, 295)
CV_REGISTER(ARM64_H26, 296)
CV_REGISTER(ARM64_H27, 297)
CV_REGISTER(ARM64_H28, 298)
CV_REGISTER(ARM64_H29, 299)
CV_REGISTER(ARM64_H30, 300)
CV_REGISTER(ARM64_H31, 301)

````
- **L769 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L769 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L770 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L770 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L771 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L771 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L772 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L772 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L773 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L773 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L774 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L774 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L775 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L775 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L776 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L776 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L777 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L777 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L778 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L778 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L779 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L779 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L780 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L780 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L781 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L781 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L782 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L782 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L783 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L783 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L784 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L784 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L785 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L785 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L786 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L786 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L787 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L787 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L788 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L788 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L789 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L789 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L790 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L790 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L791 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L791 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L792 EN**: Blank line separating nearby declarations or logic blocks.
  **L792 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 793-816

````cpp
#pragma pop_macro("ARM64_FPSR")
#pragma pop_macro("ARM64_FPCR")

#endif // defined(CV_REGISTERS_ALL) || defined(CV_REGISTERS_ARM64)

#if defined(CV_REGISTERS_ALL) || defined(CV_REGISTERS_MIPS)

// MIPS registers
CV_REGISTER(MIPS_NOREG, 0)

// General purpose integer registers

CV_REGISTER(MIPS_ZERO, 10)
CV_REGISTER(MIPS_AT, 11)
CV_REGISTER(MIPS_V0, 12)
CV_REGISTER(MIPS_V1, 13)
CV_REGISTER(MIPS_A0, 14)
CV_REGISTER(MIPS_A1, 15)
CV_REGISTER(MIPS_A2, 16)
CV_REGISTER(MIPS_A3, 17)
CV_REGISTER(MIPS_T0, 18)
CV_REGISTER(MIPS_T1, 19)
CV_REGISTER(MIPS_T2, 20)
CV_REGISTER(MIPS_T3, 21)
````
- **L793 EN**: Continues logic associated with callable symbol `pop_macro`.
  **L793 CN**: 继续与可调用符号 `pop_macro` 相关的逻辑。
- **L794 EN**: Continues logic associated with callable symbol `pop_macro`.
  **L794 CN**: 继续与可调用符号 `pop_macro` 相关的逻辑。
- **L795 EN**: Blank line separating nearby declarations or logic blocks.
  **L795 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L796 EN**: Closes the current preprocessor conditional block.
  **L796 CN**: 结束当前预处理条件块。
- **L797 EN**: Blank line separating nearby declarations or logic blocks.
  **L797 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L798 EN**: Starts a preprocessor conditional block: `#if defined(CV_REGISTERS_ALL) || defined(CV_REGISTERS_MIPS)`.
  **L798 CN**: 开始一个预处理条件块：`#if defined(CV_REGISTERS_ALL) || defined(CV_REGISTERS_MIPS)`。
- **L799 EN**: Blank line separating nearby declarations or logic blocks.
  **L799 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L800 EN**: Comment explains nearby logic, invariants, or intent: `MIPS registers`.
  **L800 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`MIPS registers`。
- **L801 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L801 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L802 EN**: Blank line separating nearby declarations or logic blocks.
  **L802 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L803 EN**: Comment explains nearby logic, invariants, or intent: `General purpose integer registers`.
  **L803 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`General purpose integer registers`。
- **L804 EN**: Blank line separating nearby declarations or logic blocks.
  **L804 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L805 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L805 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L806 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L806 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L807 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L807 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L808 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L808 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L809 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L809 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L810 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L810 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L811 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L811 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L812 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L812 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L813 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L813 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L814 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L814 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L815 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L815 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L816 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L816 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。

### Lines 817-840

````cpp
CV_REGISTER(MIPS_T4, 22)
CV_REGISTER(MIPS_T5, 23)
CV_REGISTER(MIPS_T6, 24)
CV_REGISTER(MIPS_T7, 25)
CV_REGISTER(MIPS_S0, 26)
CV_REGISTER(MIPS_S1, 27)
CV_REGISTER(MIPS_S2, 28)
CV_REGISTER(MIPS_S3, 29)
CV_REGISTER(MIPS_S4, 30)
CV_REGISTER(MIPS_S5, 31)
CV_REGISTER(MIPS_S6, 32)
CV_REGISTER(MIPS_S7, 33)
CV_REGISTER(MIPS_T8, 34)
CV_REGISTER(MIPS_T9, 35)
CV_REGISTER(MIPS_K0, 36)
CV_REGISTER(MIPS_K1, 37)
CV_REGISTER(MIPS_GP, 38)
CV_REGISTER(MIPS_SP, 39)
CV_REGISTER(MIPS_S8, 40)
CV_REGISTER(MIPS_RA, 41)
CV_REGISTER(MIPS_LO, 42)
CV_REGISTER(MIPS_HI, 43)

// Status registers
````
- **L817 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L817 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L818 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L818 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L819 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L819 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L820 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L820 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L821 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L821 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L822 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L822 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L823 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L823 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L824 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L824 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L825 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L825 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L826 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L826 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L827 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L827 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L828 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L828 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L829 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L829 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L830 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L830 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L831 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L831 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L832 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L832 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L833 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L833 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L834 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L834 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L835 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L835 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L836 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L836 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L837 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L837 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L838 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L838 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L839 EN**: Blank line separating nearby declarations or logic blocks.
  **L839 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L840 EN**: Comment explains nearby logic, invariants, or intent: `Status registers`.
  **L840 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Status registers`。

### Lines 841-864

````cpp

CV_REGISTER(MIPS_Fir, 50)
CV_REGISTER(MIPS_Psr, 51)

// Floating-point registers

CV_REGISTER(MIPS_F0, 60)
CV_REGISTER(MIPS_F1, 61)
CV_REGISTER(MIPS_F2, 62)
CV_REGISTER(MIPS_F3, 63)
CV_REGISTER(MIPS_F4, 64)
CV_REGISTER(MIPS_F5, 65)
CV_REGISTER(MIPS_F6, 66)
CV_REGISTER(MIPS_F7, 67)
CV_REGISTER(MIPS_F8, 68)
CV_REGISTER(MIPS_F9, 69)
CV_REGISTER(MIPS_F10, 70)
CV_REGISTER(MIPS_F11, 71)
CV_REGISTER(MIPS_F12, 72)
CV_REGISTER(MIPS_F13, 73)
CV_REGISTER(MIPS_F14, 74)
CV_REGISTER(MIPS_F15, 75)
CV_REGISTER(MIPS_F16, 76)
CV_REGISTER(MIPS_F17, 77)
````
- **L841 EN**: Blank line separating nearby declarations or logic blocks.
  **L841 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L842 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L842 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L843 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L843 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L844 EN**: Blank line separating nearby declarations or logic blocks.
  **L844 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L845 EN**: Comment explains nearby logic, invariants, or intent: `Floating-point registers`.
  **L845 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Floating-point registers`。
- **L846 EN**: Blank line separating nearby declarations or logic blocks.
  **L846 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L847 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L847 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L848 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L848 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L849 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L849 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L850 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L850 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L851 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L851 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L852 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L852 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L853 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L853 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L854 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L854 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L855 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L855 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L856 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L856 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L857 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L857 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L858 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L858 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L859 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L859 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L860 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L860 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L861 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L861 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L862 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L862 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L863 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L863 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L864 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L864 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。

### Lines 865-881

````cpp
CV_REGISTER(MIPS_F18, 78)
CV_REGISTER(MIPS_F19, 79)
CV_REGISTER(MIPS_F20, 80)
CV_REGISTER(MIPS_F21, 81)
CV_REGISTER(MIPS_F22, 82)
CV_REGISTER(MIPS_F23, 83)
CV_REGISTER(MIPS_F24, 84)
CV_REGISTER(MIPS_F25, 85)
CV_REGISTER(MIPS_F26, 86)
CV_REGISTER(MIPS_F27, 87)
CV_REGISTER(MIPS_F28, 88)
CV_REGISTER(MIPS_F29, 89)
CV_REGISTER(MIPS_F30, 90)
CV_REGISTER(MIPS_F31, 91)
CV_REGISTER(MIPS_Fsr, 92)

#endif // defined(CV_REGISTERS_ALL) || defined(CV_REGISTERS_MIPS)
````
- **L865 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L865 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L866 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L866 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L867 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L867 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L868 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L868 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L869 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L869 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L870 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L870 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L871 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L871 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L872 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L872 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L873 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L873 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L874 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L874 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L875 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L875 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L876 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L876 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L877 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L877 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L878 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L878 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L879 EN**: Continues logic associated with callable symbol `CV_REGISTER`.
  **L879 CN**: 继续与可调用符号 `CV_REGISTER` 相关的逻辑。
- **L880 EN**: Blank line separating nearby declarations or logic blocks.
  **L880 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L881 EN**: Closes the current preprocessor conditional block.
  **L881 CN**: 结束当前预处理条件块。

## Key Concepts / 关键概念

- **LLVM header interfaces / LLVM 头文件接口**
- **Debug information models / 调试信息模型**
- **Register allocation and tracking / 寄存器分配与跟踪**

## Dependencies / 依赖关系

- No direct `#include` dependencies appear in this file. / 该文件中没有直接出现 `#include` 依赖。
