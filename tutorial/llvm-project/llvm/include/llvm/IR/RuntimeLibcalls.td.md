# RuntimeLibcalls.td — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `llvm/include/llvm/IR/RuntimeLibcalls.td`
- **Repository / 仓库**: `llvm-project`
- **Purpose (EN)**: Defines TableGen records, generated metadata, and declarative rules for `RuntimeLibcalls`.
- **Purpose (CN)**: 该 TableGen 文件位于 `llvm/include/llvm/IR`，主要定义与 `RuntimeLibcalls` 相关的记录、生成元数据和声明式规则。

## Line-by-Line Analysis / 逐行分析

### Lines 1-32

````tablegen
//===-- llvm/RuntimeLibcallList.td - Describe libcalls --------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

include "llvm/IR/RuntimeLibcallsImpl.td"

//--------------------------------------------------------------------
// Utility classes
//--------------------------------------------------------------------

class DuplicateLibcallImplWithPrefix<RuntimeLibcallImpl Impl, string prefix>
    : RuntimeLibcallImpl<Impl.Provides, prefix#Impl.LibCallFuncName>;

/// Libcall Predicates
def isOSDarwin : RuntimeLibcallPredicate<"TT.isOSDarwin()">;
def isOSOpenBSD : RuntimeLibcallPredicate<"TT.isOSOpenBSD()">;
def isNotOSOpenBSD : RuntimeLibcallPredicate<"!TT.isOSOpenBSD()">;
def isOSWindows : RuntimeLibcallPredicate<"TT.isOSWindows()">;
def isNotOSWindows : RuntimeLibcallPredicate<"!TT.isOSWindows()">;
def isNotOSLinux : RuntimeLibcallPredicate<[{!TT.isOSLinux()}]>;
def isNotOSMSVCRT : RuntimeLibcallPredicate<"!TT.isOSMSVCRT()">;
def isPS : RuntimeLibcallPredicate<"TT.isPS()">;
def isMacOSX : RuntimeLibcallPredicate<[{TT.isMacOSX()}]>;
def isNotOSWindowsOrIsCygwinMinGW
  : RuntimeLibcallPredicate<"!TT.isOSWindows() || TT.isOSCygMing()">;

def isWindowsMSVCEnvironment : RuntimeLibcallPredicate<
  [{TT.isWindowsMSVCEnvironment()}]>;
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
- **L8 EN**: Blank line separating nearby declarations or logic blocks.
  **L8 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L9 EN**: Imports TableGen file "llvm/IR/RuntimeLibcallsImpl.td" to reuse LLVM IR core types, instructions, and ownership utilities.
  **L9 CN**: 导入 TableGen 文件 "llvm/IR/RuntimeLibcallsImpl.td" 以复用LLVM IR 核心类型、指令与所有权辅助组件。
- **L10 EN**: Blank line separating nearby declarations or logic blocks.
  **L10 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L11 EN**: Comment explains nearby logic, invariants, or intent: `--------------------------------------------------------------------`.
  **L11 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`--------------------------------------------------------------------`。
- **L12 EN**: Comment explains nearby logic, invariants, or intent: `Utility classes`.
  **L12 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Utility classes`。
- **L13 EN**: Comment explains nearby logic, invariants, or intent: `--------------------------------------------------------------------`.
  **L13 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`--------------------------------------------------------------------`。
- **L14 EN**: Blank line separating nearby declarations or logic blocks.
  **L14 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L15 EN**: Declares class `DuplicateLibcallImplWithPrefix<RuntimeLibcallImpl`.
  **L15 CN**: 声明 class `DuplicateLibcallImplWithPrefix<RuntimeLibcallImpl`。
- **L16 EN**: Executes a standalone statement or declaration: `: RuntimeLibcallImpl<Impl.Provides, prefix#Impl.LibCallFuncName>;`.
  **L16 CN**: 执行一条独立语句或声明：`: RuntimeLibcallImpl<Impl.Provides, prefix#Impl.LibCallFuncName>;`。
- **L17 EN**: Blank line separating nearby declarations or logic blocks.
  **L17 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L18 EN**: Comment explains nearby logic, invariants, or intent: `Libcall Predicates`.
  **L18 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Libcall Predicates`。
- **L19 EN**: Declares TableGen def `isOSDarwin`.
  **L19 CN**: 声明 TableGen def `isOSDarwin`。
- **L20 EN**: Declares TableGen def `isOSOpenBSD`.
  **L20 CN**: 声明 TableGen def `isOSOpenBSD`。
- **L21 EN**: Declares TableGen def `isNotOSOpenBSD`.
  **L21 CN**: 声明 TableGen def `isNotOSOpenBSD`。
- **L22 EN**: Declares TableGen def `isOSWindows`.
  **L22 CN**: 声明 TableGen def `isOSWindows`。
- **L23 EN**: Declares TableGen def `isNotOSWindows`.
  **L23 CN**: 声明 TableGen def `isNotOSWindows`。
- **L24 EN**: Declares TableGen def `isNotOSLinux`.
  **L24 CN**: 声明 TableGen def `isNotOSLinux`。
- **L25 EN**: Declares TableGen def `isNotOSMSVCRT`.
  **L25 CN**: 声明 TableGen def `isNotOSMSVCRT`。
- **L26 EN**: Declares TableGen def `isPS`.
  **L26 CN**: 声明 TableGen def `isPS`。
- **L27 EN**: Declares TableGen def `isMacOSX`.
  **L27 CN**: 声明 TableGen def `isMacOSX`。
- **L28 EN**: Declares TableGen def `isNotOSWindowsOrIsCygwinMinGW`.
  **L28 CN**: 声明 TableGen def `isNotOSWindowsOrIsCygwinMinGW`。
- **L29 EN**: Executes a call or declaration centered on `RuntimeLibcallPredicate<"!TT.isOSWindows`.
  **L29 CN**: 执行以 `RuntimeLibcallPredicate<"!TT.isOSWindows` 为核心的调用或声明。
- **L30 EN**: Blank line separating nearby declarations or logic blocks.
  **L30 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L31 EN**: Declares TableGen def `isWindowsMSVCEnvironment`.
  **L31 CN**: 声明 TableGen def `isWindowsMSVCEnvironment`。
- **L32 EN**: Executes a call or declaration centered on `[{TT.isWindowsMSVCEnvironment`.
  **L32 CN**: 执行以 `[{TT.isWindowsMSVCEnvironment` 为核心的调用或声明。

### Lines 33-64

````tablegen

def isNotOSLinuxAndNotOSOpenBSD : RuntimeLibcallPredicate<
  [{!TT.isOSLinux() && !TT.isOSOpenBSD()}]>;

def isNotOSAIXAndNotOSOpenBSD : RuntimeLibcallPredicate<
  [{!TT.isOSAIX() && !TT.isOSOpenBSD()}]>;

def isNotPS : RuntimeLibcallPredicate<
  [{!TT.isPS()}]>;

// OpenBSD uses __guard_local. AIX uses __ssp_canary_word, MSVC/Windows
// Itanium uses __security_cookie
def hasStackChkFail : RuntimeLibcallPredicate<
  [{ !TT.isOSOpenBSD() && !TT.isWindowsMSVCEnvironment() &&
     !TT.isWindowsItaniumEnvironment()}]>;

def isWindowsMSVCOrItaniumEnvironment : RuntimeLibcallPredicate<
  [{TT.isWindowsMSVCEnvironment() || TT.isWindowsItaniumEnvironment()}]>;

def isAndroid : RuntimeLibcallPredicate<"TT.isAndroid()">;

def isGNUEnvironment : RuntimeLibcallPredicate<"TT.isGNUEnvironment()">;
def darwinHasSinCosStret : RuntimeLibcallPredicate<"darwinHasSinCosStret(TT)">;
def darwinHasExp10 : RuntimeLibcallPredicate<"darwinHasExp10(TT)">;
def darwinHasMemsetPattern : RuntimeLibcallPredicate<[{darwinHasMemsetPattern(TT)}]>;

def hasExp10 : RuntimeLibcallPredicate<[{!TT.isOSDarwin()}]>;

def hasSinCos : RuntimeLibcallPredicate<"hasSinCos(TT)">;

// FIXME: Way to combine predicates
def hasSinCos_f32_f64 : RuntimeLibcallPredicate<"hasSinCos_f32_f64(TT)">;
````
- **L33 EN**: Blank line separating nearby declarations or logic blocks.
  **L33 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L34 EN**: Declares TableGen def `isNotOSLinuxAndNotOSOpenBSD`.
  **L34 CN**: 声明 TableGen def `isNotOSLinuxAndNotOSOpenBSD`。
- **L35 EN**: Executes a call or declaration centered on `[{!TT.isOSLinux`.
  **L35 CN**: 执行以 `[{!TT.isOSLinux` 为核心的调用或声明。
- **L36 EN**: Blank line separating nearby declarations or logic blocks.
  **L36 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L37 EN**: Declares TableGen def `isNotOSAIXAndNotOSOpenBSD`.
  **L37 CN**: 声明 TableGen def `isNotOSAIXAndNotOSOpenBSD`。
- **L38 EN**: Executes a call or declaration centered on `[{!TT.isOSAIX`.
  **L38 CN**: 执行以 `[{!TT.isOSAIX` 为核心的调用或声明。
- **L39 EN**: Blank line separating nearby declarations or logic blocks.
  **L39 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L40 EN**: Declares TableGen def `isNotPS`.
  **L40 CN**: 声明 TableGen def `isNotPS`。
- **L41 EN**: Executes a call or declaration centered on `[{!TT.isPS`.
  **L41 CN**: 执行以 `[{!TT.isPS` 为核心的调用或声明。
- **L42 EN**: Blank line separating nearby declarations or logic blocks.
  **L42 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L43 EN**: Comment explains nearby logic, invariants, or intent: `OpenBSD uses __guard_local. AIX uses __ssp_canary_word, MSVC/Windows`.
  **L43 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`OpenBSD uses __guard_local. AIX uses __ssp_canary_word, MSVC/Windows`。
- **L44 EN**: Comment explains nearby logic, invariants, or intent: `Itanium uses __security_cookie`.
  **L44 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Itanium uses __security_cookie`。
- **L45 EN**: Declares TableGen def `hasStackChkFail`.
  **L45 CN**: 声明 TableGen def `hasStackChkFail`。
- **L46 EN**: Continues logic associated with callable symbol `isOSOpenBSD`.
  **L46 CN**: 继续与可调用符号 `isOSOpenBSD` 相关的逻辑。
- **L47 EN**: Executes a call or declaration centered on `!TT.isWindowsItaniumEnvironment`.
  **L47 CN**: 执行以 `!TT.isWindowsItaniumEnvironment` 为核心的调用或声明。
- **L48 EN**: Blank line separating nearby declarations or logic blocks.
  **L48 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L49 EN**: Declares TableGen def `isWindowsMSVCOrItaniumEnvironment`.
  **L49 CN**: 声明 TableGen def `isWindowsMSVCOrItaniumEnvironment`。
- **L50 EN**: Executes a call or declaration centered on `[{TT.isWindowsMSVCEnvironment`.
  **L50 CN**: 执行以 `[{TT.isWindowsMSVCEnvironment` 为核心的调用或声明。
- **L51 EN**: Blank line separating nearby declarations or logic blocks.
  **L51 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L52 EN**: Declares TableGen def `isAndroid`.
  **L52 CN**: 声明 TableGen def `isAndroid`。
- **L53 EN**: Blank line separating nearby declarations or logic blocks.
  **L53 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L54 EN**: Declares TableGen def `isGNUEnvironment`.
  **L54 CN**: 声明 TableGen def `isGNUEnvironment`。
- **L55 EN**: Declares TableGen def `darwinHasSinCosStret`.
  **L55 CN**: 声明 TableGen def `darwinHasSinCosStret`。
- **L56 EN**: Declares TableGen def `darwinHasExp10`.
  **L56 CN**: 声明 TableGen def `darwinHasExp10`。
- **L57 EN**: Declares TableGen def `darwinHasMemsetPattern`.
  **L57 CN**: 声明 TableGen def `darwinHasMemsetPattern`。
- **L58 EN**: Blank line separating nearby declarations or logic blocks.
  **L58 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L59 EN**: Declares TableGen def `hasExp10`.
  **L59 CN**: 声明 TableGen def `hasExp10`。
- **L60 EN**: Blank line separating nearby declarations or logic blocks.
  **L60 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L61 EN**: Declares TableGen def `hasSinCos`.
  **L61 CN**: 声明 TableGen def `hasSinCos`。
- **L62 EN**: Blank line separating nearby declarations or logic blocks.
  **L62 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L63 EN**: Comment records a pending task or caution: `FIXME: Way to combine predicates`.
  **L63 CN**: 注释记录了待办事项或注意点：`FIXME: Way to combine predicates`。
- **L64 EN**: Declares TableGen def `hasSinCos_f32_f64`.
  **L64 CN**: 声明 TableGen def `hasSinCos_f32_f64`。

### Lines 65-96

````tablegen

def ExceptionModelIsNotNone : RuntimeLibcallPredicate<
  [{ExceptionModel != ExceptionHandling::None}]
>;

def ExceptionModelHasUnwindResume : RuntimeLibcallPredicate<
  [{ExceptionModel != ExceptionHandling::None &&
    ExceptionModel != ExceptionHandling::SjLj}]
>;

def ExceptionModelIsSjLj : RuntimeLibcallPredicate<
  [{ExceptionModel == ExceptionHandling::SjLj}]
>;

//--------------------------------------------------------------------
// Declare all kinds of used libcalls
//--------------------------------------------------------------------

// Integer
foreach IntTy = ["I16", "I32", "I64", "I128"] in {
  def SHL_#IntTy : RuntimeLibcall;
  def SRL_#IntTy : RuntimeLibcall;
  def SRA_#IntTy : RuntimeLibcall;
}

foreach IntTy = ["I8", "I16", "I32", "I64", "I128"] in {
  def MUL_#IntTy : RuntimeLibcall;
}

foreach IntTy = ["I32", "I64", "I128" ] in {
  def MULO_#IntTy : RuntimeLibcall;
}
````
- **L65 EN**: Blank line separating nearby declarations or logic blocks.
  **L65 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L66 EN**: Declares TableGen def `ExceptionModelIsNotNone`.
  **L66 CN**: 声明 TableGen def `ExceptionModelIsNotNone`。
- **L67 EN**: Continues the surrounding expression or declaration: `[{ExceptionModel != ExceptionHandling::None}]`.
  **L67 CN**: 继续构造周围的表达式或声明：`[{ExceptionModel != ExceptionHandling::None}]`。
- **L68 EN**: Executes a standalone statement or declaration: `>;`.
  **L68 CN**: 执行一条独立语句或声明：`>;`。
- **L69 EN**: Blank line separating nearby declarations or logic blocks.
  **L69 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L70 EN**: Declares TableGen def `ExceptionModelHasUnwindResume`.
  **L70 CN**: 声明 TableGen def `ExceptionModelHasUnwindResume`。
- **L71 EN**: Continues the surrounding expression or declaration: `[{ExceptionModel != ExceptionHandling::None &&`.
  **L71 CN**: 继续构造周围的表达式或声明：`[{ExceptionModel != ExceptionHandling::None &&`。
- **L72 EN**: Continues the surrounding expression or declaration: `ExceptionModel != ExceptionHandling::SjLj}]`.
  **L72 CN**: 继续构造周围的表达式或声明：`ExceptionModel != ExceptionHandling::SjLj}]`。
- **L73 EN**: Executes a standalone statement or declaration: `>;`.
  **L73 CN**: 执行一条独立语句或声明：`>;`。
- **L74 EN**: Blank line separating nearby declarations or logic blocks.
  **L74 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L75 EN**: Declares TableGen def `ExceptionModelIsSjLj`.
  **L75 CN**: 声明 TableGen def `ExceptionModelIsSjLj`。
- **L76 EN**: Continues the surrounding expression or declaration: `[{ExceptionModel == ExceptionHandling::SjLj}]`.
  **L76 CN**: 继续构造周围的表达式或声明：`[{ExceptionModel == ExceptionHandling::SjLj}]`。
- **L77 EN**: Executes a standalone statement or declaration: `>;`.
  **L77 CN**: 执行一条独立语句或声明：`>;`。
- **L78 EN**: Blank line separating nearby declarations or logic blocks.
  **L78 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L79 EN**: Comment explains nearby logic, invariants, or intent: `--------------------------------------------------------------------`.
  **L79 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`--------------------------------------------------------------------`。
- **L80 EN**: Comment explains nearby logic, invariants, or intent: `Declare all kinds of used libcalls`.
  **L80 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Declare all kinds of used libcalls`。
- **L81 EN**: Comment explains nearby logic, invariants, or intent: `--------------------------------------------------------------------`.
  **L81 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`--------------------------------------------------------------------`。
- **L82 EN**: Blank line separating nearby declarations or logic blocks.
  **L82 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L83 EN**: Comment explains nearby logic, invariants, or intent: `Integer`.
  **L83 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Integer`。
- **L84 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L84 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L85 EN**: Declares TableGen def `SHL_#IntTy`.
  **L85 CN**: 声明 TableGen def `SHL_#IntTy`。
- **L86 EN**: Declares TableGen def `SRL_#IntTy`.
  **L86 CN**: 声明 TableGen def `SRL_#IntTy`。
- **L87 EN**: Declares TableGen def `SRA_#IntTy`.
  **L87 CN**: 声明 TableGen def `SRA_#IntTy`。
- **L88 EN**: Closes the current lexical scope or compound statement.
  **L88 CN**: 结束当前词法作用域或复合语句块。
- **L89 EN**: Blank line separating nearby declarations or logic blocks.
  **L89 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L90 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L90 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L91 EN**: Declares TableGen def `MUL_#IntTy`.
  **L91 CN**: 声明 TableGen def `MUL_#IntTy`。
- **L92 EN**: Closes the current lexical scope or compound statement.
  **L92 CN**: 结束当前词法作用域或复合语句块。
- **L93 EN**: Blank line separating nearby declarations or logic blocks.
  **L93 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L94 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L94 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L95 EN**: Declares TableGen def `MULO_#IntTy`.
  **L95 CN**: 声明 TableGen def `MULO_#IntTy`。
- **L96 EN**: Closes the current lexical scope or compound statement.
  **L96 CN**: 结束当前词法作用域或复合语句块。

### Lines 97-128

````tablegen

foreach IntTy = ["I8", "I16", "I32", "I64", "I128"] in {
  def SDIV_#IntTy : RuntimeLibcall;
  def UDIV_#IntTy : RuntimeLibcall;
  def SREM_#IntTy : RuntimeLibcall;
  def UREM_#IntTy : RuntimeLibcall;
  def SDIVREM_#IntTy : RuntimeLibcall;
  def UDIVREM_#IntTy : RuntimeLibcall;
}

foreach IntTy = ["I32", "I64" ] in {
  def NEG_#IntTy : RuntimeLibcall;
}

foreach IntTy = ["I32", "I64", "I128"] in {
  def CTLZ_#IntTy : RuntimeLibcall;
  def CTPOP_#IntTy : RuntimeLibcall;
}

foreach FPTy = ["F32", "F64", "F80", "F128", "PPCF128"] in {
  def ADD_#FPTy : RuntimeLibcall;
  def FAST_ADD_#FPTy : RuntimeLibcall;

  def SUB_#FPTy : RuntimeLibcall;
  def FAST_SUB_#FPTy : RuntimeLibcall;

  def MUL_#FPTy : RuntimeLibcall;
  def FAST_MUL_#FPTy : RuntimeLibcall;

  def DIV_#FPTy : RuntimeLibcall;
  def FAST_DIV_#FPTy : RuntimeLibcall;

````
- **L97 EN**: Blank line separating nearby declarations or logic blocks.
  **L97 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L98 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L98 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L99 EN**: Declares TableGen def `SDIV_#IntTy`.
  **L99 CN**: 声明 TableGen def `SDIV_#IntTy`。
- **L100 EN**: Declares TableGen def `UDIV_#IntTy`.
  **L100 CN**: 声明 TableGen def `UDIV_#IntTy`。
- **L101 EN**: Declares TableGen def `SREM_#IntTy`.
  **L101 CN**: 声明 TableGen def `SREM_#IntTy`。
- **L102 EN**: Declares TableGen def `UREM_#IntTy`.
  **L102 CN**: 声明 TableGen def `UREM_#IntTy`。
- **L103 EN**: Declares TableGen def `SDIVREM_#IntTy`.
  **L103 CN**: 声明 TableGen def `SDIVREM_#IntTy`。
- **L104 EN**: Declares TableGen def `UDIVREM_#IntTy`.
  **L104 CN**: 声明 TableGen def `UDIVREM_#IntTy`。
- **L105 EN**: Closes the current lexical scope or compound statement.
  **L105 CN**: 结束当前词法作用域或复合语句块。
- **L106 EN**: Blank line separating nearby declarations or logic blocks.
  **L106 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L107 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L107 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L108 EN**: Declares TableGen def `NEG_#IntTy`.
  **L108 CN**: 声明 TableGen def `NEG_#IntTy`。
- **L109 EN**: Closes the current lexical scope or compound statement.
  **L109 CN**: 结束当前词法作用域或复合语句块。
- **L110 EN**: Blank line separating nearby declarations or logic blocks.
  **L110 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L111 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L111 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L112 EN**: Declares TableGen def `CTLZ_#IntTy`.
  **L112 CN**: 声明 TableGen def `CTLZ_#IntTy`。
- **L113 EN**: Declares TableGen def `CTPOP_#IntTy`.
  **L113 CN**: 声明 TableGen def `CTPOP_#IntTy`。
- **L114 EN**: Closes the current lexical scope or compound statement.
  **L114 CN**: 结束当前词法作用域或复合语句块。
- **L115 EN**: Blank line separating nearby declarations or logic blocks.
  **L115 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L116 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L116 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L117 EN**: Declares TableGen def `ADD_#FPTy`.
  **L117 CN**: 声明 TableGen def `ADD_#FPTy`。
- **L118 EN**: Declares TableGen def `FAST_ADD_#FPTy`.
  **L118 CN**: 声明 TableGen def `FAST_ADD_#FPTy`。
- **L119 EN**: Blank line separating nearby declarations or logic blocks.
  **L119 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L120 EN**: Declares TableGen def `SUB_#FPTy`.
  **L120 CN**: 声明 TableGen def `SUB_#FPTy`。
- **L121 EN**: Declares TableGen def `FAST_SUB_#FPTy`.
  **L121 CN**: 声明 TableGen def `FAST_SUB_#FPTy`。
- **L122 EN**: Blank line separating nearby declarations or logic blocks.
  **L122 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L123 EN**: Declares TableGen def `MUL_#FPTy`.
  **L123 CN**: 声明 TableGen def `MUL_#FPTy`。
- **L124 EN**: Declares TableGen def `FAST_MUL_#FPTy`.
  **L124 CN**: 声明 TableGen def `FAST_MUL_#FPTy`。
- **L125 EN**: Blank line separating nearby declarations or logic blocks.
  **L125 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L126 EN**: Declares TableGen def `DIV_#FPTy`.
  **L126 CN**: 声明 TableGen def `DIV_#FPTy`。
- **L127 EN**: Declares TableGen def `FAST_DIV_#FPTy`.
  **L127 CN**: 声明 TableGen def `FAST_DIV_#FPTy`。
- **L128 EN**: Blank line separating nearby declarations or logic blocks.
  **L128 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 129-160

````tablegen
  def REM_#FPTy : RuntimeLibcall;
  def FMA_#FPTy : RuntimeLibcall;
  def POWI_#FPTy : RuntimeLibcall;

  def SQRT_#FPTy : RuntimeLibcall;
  def FAST_SQRT_#FPTy : RuntimeLibcall;

  def CBRT_#FPTy : RuntimeLibcall;
  def LOG_#FPTy : RuntimeLibcall;
  def LOG_FINITE_#FPTy : RuntimeLibcall;
  def LOG2_#FPTy : RuntimeLibcall;
  def LOG2_FINITE_#FPTy : RuntimeLibcall;
  def LOG10_#FPTy : RuntimeLibcall;
  def LOG10_FINITE_#FPTy : RuntimeLibcall;
  def EXP_#FPTy : RuntimeLibcall;
  def EXP_FINITE_#FPTy : RuntimeLibcall;
  def EXP2_#FPTy : RuntimeLibcall;
  def EXP2_FINITE_#FPTy : RuntimeLibcall;
  def EXP10_#FPTy : RuntimeLibcall;
  def EXP10_FINITE_#FPTy : RuntimeLibcall;
  def SIN_#FPTy : RuntimeLibcall;
  def COS_#FPTy : RuntimeLibcall;
  def TAN_#FPTy : RuntimeLibcall;
  def SINH_#FPTy : RuntimeLibcall;
  def COSH_#FPTy : RuntimeLibcall;
  def TANH_#FPTy : RuntimeLibcall;
  def ASIN_#FPTy : RuntimeLibcall;
  def ACOS_#FPTy : RuntimeLibcall;
  def ATAN_#FPTy : RuntimeLibcall;
  def ATAN2_#FPTy : RuntimeLibcall;
  def SINCOS_#FPTy : RuntimeLibcall;
  def REMQUO_#FPTy : RuntimeLibcall;
````
- **L129 EN**: Declares TableGen def `REM_#FPTy`.
  **L129 CN**: 声明 TableGen def `REM_#FPTy`。
- **L130 EN**: Declares TableGen def `FMA_#FPTy`.
  **L130 CN**: 声明 TableGen def `FMA_#FPTy`。
- **L131 EN**: Declares TableGen def `POWI_#FPTy`.
  **L131 CN**: 声明 TableGen def `POWI_#FPTy`。
- **L132 EN**: Blank line separating nearby declarations or logic blocks.
  **L132 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L133 EN**: Declares TableGen def `SQRT_#FPTy`.
  **L133 CN**: 声明 TableGen def `SQRT_#FPTy`。
- **L134 EN**: Declares TableGen def `FAST_SQRT_#FPTy`.
  **L134 CN**: 声明 TableGen def `FAST_SQRT_#FPTy`。
- **L135 EN**: Blank line separating nearby declarations or logic blocks.
  **L135 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L136 EN**: Declares TableGen def `CBRT_#FPTy`.
  **L136 CN**: 声明 TableGen def `CBRT_#FPTy`。
- **L137 EN**: Declares TableGen def `LOG_#FPTy`.
  **L137 CN**: 声明 TableGen def `LOG_#FPTy`。
- **L138 EN**: Declares TableGen def `LOG_FINITE_#FPTy`.
  **L138 CN**: 声明 TableGen def `LOG_FINITE_#FPTy`。
- **L139 EN**: Declares TableGen def `LOG2_#FPTy`.
  **L139 CN**: 声明 TableGen def `LOG2_#FPTy`。
- **L140 EN**: Declares TableGen def `LOG2_FINITE_#FPTy`.
  **L140 CN**: 声明 TableGen def `LOG2_FINITE_#FPTy`。
- **L141 EN**: Declares TableGen def `LOG10_#FPTy`.
  **L141 CN**: 声明 TableGen def `LOG10_#FPTy`。
- **L142 EN**: Declares TableGen def `LOG10_FINITE_#FPTy`.
  **L142 CN**: 声明 TableGen def `LOG10_FINITE_#FPTy`。
- **L143 EN**: Declares TableGen def `EXP_#FPTy`.
  **L143 CN**: 声明 TableGen def `EXP_#FPTy`。
- **L144 EN**: Declares TableGen def `EXP_FINITE_#FPTy`.
  **L144 CN**: 声明 TableGen def `EXP_FINITE_#FPTy`。
- **L145 EN**: Declares TableGen def `EXP2_#FPTy`.
  **L145 CN**: 声明 TableGen def `EXP2_#FPTy`。
- **L146 EN**: Declares TableGen def `EXP2_FINITE_#FPTy`.
  **L146 CN**: 声明 TableGen def `EXP2_FINITE_#FPTy`。
- **L147 EN**: Declares TableGen def `EXP10_#FPTy`.
  **L147 CN**: 声明 TableGen def `EXP10_#FPTy`。
- **L148 EN**: Declares TableGen def `EXP10_FINITE_#FPTy`.
  **L148 CN**: 声明 TableGen def `EXP10_FINITE_#FPTy`。
- **L149 EN**: Declares TableGen def `SIN_#FPTy`.
  **L149 CN**: 声明 TableGen def `SIN_#FPTy`。
- **L150 EN**: Declares TableGen def `COS_#FPTy`.
  **L150 CN**: 声明 TableGen def `COS_#FPTy`。
- **L151 EN**: Declares TableGen def `TAN_#FPTy`.
  **L151 CN**: 声明 TableGen def `TAN_#FPTy`。
- **L152 EN**: Declares TableGen def `SINH_#FPTy`.
  **L152 CN**: 声明 TableGen def `SINH_#FPTy`。
- **L153 EN**: Declares TableGen def `COSH_#FPTy`.
  **L153 CN**: 声明 TableGen def `COSH_#FPTy`。
- **L154 EN**: Declares TableGen def `TANH_#FPTy`.
  **L154 CN**: 声明 TableGen def `TANH_#FPTy`。
- **L155 EN**: Declares TableGen def `ASIN_#FPTy`.
  **L155 CN**: 声明 TableGen def `ASIN_#FPTy`。
- **L156 EN**: Declares TableGen def `ACOS_#FPTy`.
  **L156 CN**: 声明 TableGen def `ACOS_#FPTy`。
- **L157 EN**: Declares TableGen def `ATAN_#FPTy`.
  **L157 CN**: 声明 TableGen def `ATAN_#FPTy`。
- **L158 EN**: Declares TableGen def `ATAN2_#FPTy`.
  **L158 CN**: 声明 TableGen def `ATAN2_#FPTy`。
- **L159 EN**: Declares TableGen def `SINCOS_#FPTy`.
  **L159 CN**: 声明 TableGen def `SINCOS_#FPTy`。
- **L160 EN**: Declares TableGen def `REMQUO_#FPTy`.
  **L160 CN**: 声明 TableGen def `REMQUO_#FPTy`。

### Lines 161-192

````tablegen
  def FDIM_#FPTy : RuntimeLibcall;

  def CABS_#FPTy : RuntimeLibcall;
}

foreach FPTy = [ "F32", "F64" ] in {
  def SINCOS_STRET_#FPTy : RuntimeLibcall;
}

foreach FPTy = ["F32", "F64", "F80", "F128", "PPCF128"] in {
  def POW_#FPTy : RuntimeLibcall;
  def POW_FINITE_#FPTy : RuntimeLibcall;
  def CEIL_#FPTy : RuntimeLibcall;
  def TRUNC_#FPTy : RuntimeLibcall;
  def RINT_#FPTy : RuntimeLibcall;
  def NEARBYINT_#FPTy : RuntimeLibcall;
  def ROUND_#FPTy : RuntimeLibcall;
  def ROUNDEVEN_#FPTy : RuntimeLibcall;
  def FLOOR_#FPTy : RuntimeLibcall;
  def COPYSIGN_#FPTy : RuntimeLibcall;
  def FMIN_#FPTy : RuntimeLibcall;
  def FMAX_#FPTy : RuntimeLibcall;
  def FMINIMUM_#FPTy : RuntimeLibcall;
  def FMAXIMUM_#FPTy : RuntimeLibcall;
  def FMINIMUM_NUM_#FPTy : RuntimeLibcall;
  def FMAXIMUM_NUM_#FPTy : RuntimeLibcall;
  def LROUND_#FPTy : RuntimeLibcall;
  def LLROUND_#FPTy : RuntimeLibcall;
  def LRINT_#FPTy : RuntimeLibcall;
  def LLRINT_#FPTy : RuntimeLibcall;
  def LDEXP_#FPTy : RuntimeLibcall;
  def FREXP_#FPTy : RuntimeLibcall;
````
- **L161 EN**: Declares TableGen def `FDIM_#FPTy`.
  **L161 CN**: 声明 TableGen def `FDIM_#FPTy`。
- **L162 EN**: Blank line separating nearby declarations or logic blocks.
  **L162 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L163 EN**: Declares TableGen def `CABS_#FPTy`.
  **L163 CN**: 声明 TableGen def `CABS_#FPTy`。
- **L164 EN**: Closes the current lexical scope or compound statement.
  **L164 CN**: 结束当前词法作用域或复合语句块。
- **L165 EN**: Blank line separating nearby declarations or logic blocks.
  **L165 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L166 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L166 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L167 EN**: Declares TableGen def `SINCOS_STRET_#FPTy`.
  **L167 CN**: 声明 TableGen def `SINCOS_STRET_#FPTy`。
- **L168 EN**: Closes the current lexical scope or compound statement.
  **L168 CN**: 结束当前词法作用域或复合语句块。
- **L169 EN**: Blank line separating nearby declarations or logic blocks.
  **L169 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L170 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L170 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L171 EN**: Declares TableGen def `POW_#FPTy`.
  **L171 CN**: 声明 TableGen def `POW_#FPTy`。
- **L172 EN**: Declares TableGen def `POW_FINITE_#FPTy`.
  **L172 CN**: 声明 TableGen def `POW_FINITE_#FPTy`。
- **L173 EN**: Declares TableGen def `CEIL_#FPTy`.
  **L173 CN**: 声明 TableGen def `CEIL_#FPTy`。
- **L174 EN**: Declares TableGen def `TRUNC_#FPTy`.
  **L174 CN**: 声明 TableGen def `TRUNC_#FPTy`。
- **L175 EN**: Declares TableGen def `RINT_#FPTy`.
  **L175 CN**: 声明 TableGen def `RINT_#FPTy`。
- **L176 EN**: Declares TableGen def `NEARBYINT_#FPTy`.
  **L176 CN**: 声明 TableGen def `NEARBYINT_#FPTy`。
- **L177 EN**: Declares TableGen def `ROUND_#FPTy`.
  **L177 CN**: 声明 TableGen def `ROUND_#FPTy`。
- **L178 EN**: Declares TableGen def `ROUNDEVEN_#FPTy`.
  **L178 CN**: 声明 TableGen def `ROUNDEVEN_#FPTy`。
- **L179 EN**: Declares TableGen def `FLOOR_#FPTy`.
  **L179 CN**: 声明 TableGen def `FLOOR_#FPTy`。
- **L180 EN**: Declares TableGen def `COPYSIGN_#FPTy`.
  **L180 CN**: 声明 TableGen def `COPYSIGN_#FPTy`。
- **L181 EN**: Declares TableGen def `FMIN_#FPTy`.
  **L181 CN**: 声明 TableGen def `FMIN_#FPTy`。
- **L182 EN**: Declares TableGen def `FMAX_#FPTy`.
  **L182 CN**: 声明 TableGen def `FMAX_#FPTy`。
- **L183 EN**: Declares TableGen def `FMINIMUM_#FPTy`.
  **L183 CN**: 声明 TableGen def `FMINIMUM_#FPTy`。
- **L184 EN**: Declares TableGen def `FMAXIMUM_#FPTy`.
  **L184 CN**: 声明 TableGen def `FMAXIMUM_#FPTy`。
- **L185 EN**: Declares TableGen def `FMINIMUM_NUM_#FPTy`.
  **L185 CN**: 声明 TableGen def `FMINIMUM_NUM_#FPTy`。
- **L186 EN**: Declares TableGen def `FMAXIMUM_NUM_#FPTy`.
  **L186 CN**: 声明 TableGen def `FMAXIMUM_NUM_#FPTy`。
- **L187 EN**: Declares TableGen def `LROUND_#FPTy`.
  **L187 CN**: 声明 TableGen def `LROUND_#FPTy`。
- **L188 EN**: Declares TableGen def `LLROUND_#FPTy`.
  **L188 CN**: 声明 TableGen def `LLROUND_#FPTy`。
- **L189 EN**: Declares TableGen def `LRINT_#FPTy`.
  **L189 CN**: 声明 TableGen def `LRINT_#FPTy`。
- **L190 EN**: Declares TableGen def `LLRINT_#FPTy`.
  **L190 CN**: 声明 TableGen def `LLRINT_#FPTy`。
- **L191 EN**: Declares TableGen def `LDEXP_#FPTy`.
  **L191 CN**: 声明 TableGen def `LDEXP_#FPTy`。
- **L192 EN**: Declares TableGen def `FREXP_#FPTy`.
  **L192 CN**: 声明 TableGen def `FREXP_#FPTy`。

### Lines 193-224

````tablegen
  def SINCOSPI_#FPTy : RuntimeLibcall;
  def MODF_#FPTy : RuntimeLibcall;
  def NAN_#FPTy : RuntimeLibcall;
  def NEXTTOWARD_#FPTy : RuntimeLibcall;
  def REMAINDER_#FPTy : RuntimeLibcall;
  def SCALBLN_#FPTy : RuntimeLibcall;
  def SCALBN_#FPTy : RuntimeLibcall;
  def TGAMMA_#FPTy : RuntimeLibcall;
}

defvar F32VectorSuffixes = ["V2F32", "V4F32", "V8F32", "V16F32", "NXV4F32"];
defvar F64VectorSuffixes = ["V2F64", "V4F64", "V8F64", "NXV2F64"];

foreach S = !listconcat(F32VectorSuffixes, F64VectorSuffixes) in {
  def ACOS_#S : RuntimeLibcall;
  def ACOSH_#S : RuntimeLibcall;
  def ASIN_#S : RuntimeLibcall;
  def ASINH_#S : RuntimeLibcall;
  def ATAN_#S : RuntimeLibcall;
  def ATAN2_#S : RuntimeLibcall;
  def ATANH_#S : RuntimeLibcall;
  def CBRT_#S : RuntimeLibcall;
  def CEIL_#S : RuntimeLibcall;
  def COPYSIGN_#S : RuntimeLibcall;
  def COS_#S : RuntimeLibcall;
  def COSH_#S : RuntimeLibcall;
  def COSPI_#S : RuntimeLibcall;
  def ERFC_#S : RuntimeLibcall;
  def ERFCINV_#S : RuntimeLibcall;
  def ERF_#S : RuntimeLibcall;
  def ERFINV_#S : RuntimeLibcall;
  def EXP_#S : RuntimeLibcall;
````
- **L193 EN**: Declares TableGen def `SINCOSPI_#FPTy`.
  **L193 CN**: 声明 TableGen def `SINCOSPI_#FPTy`。
- **L194 EN**: Declares TableGen def `MODF_#FPTy`.
  **L194 CN**: 声明 TableGen def `MODF_#FPTy`。
- **L195 EN**: Declares TableGen def `NAN_#FPTy`.
  **L195 CN**: 声明 TableGen def `NAN_#FPTy`。
- **L196 EN**: Declares TableGen def `NEXTTOWARD_#FPTy`.
  **L196 CN**: 声明 TableGen def `NEXTTOWARD_#FPTy`。
- **L197 EN**: Declares TableGen def `REMAINDER_#FPTy`.
  **L197 CN**: 声明 TableGen def `REMAINDER_#FPTy`。
- **L198 EN**: Declares TableGen def `SCALBLN_#FPTy`.
  **L198 CN**: 声明 TableGen def `SCALBLN_#FPTy`。
- **L199 EN**: Declares TableGen def `SCALBN_#FPTy`.
  **L199 CN**: 声明 TableGen def `SCALBN_#FPTy`。
- **L200 EN**: Declares TableGen def `TGAMMA_#FPTy`.
  **L200 CN**: 声明 TableGen def `TGAMMA_#FPTy`。
- **L201 EN**: Closes the current lexical scope or compound statement.
  **L201 CN**: 结束当前词法作用域或复合语句块。
- **L202 EN**: Blank line separating nearby declarations or logic blocks.
  **L202 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L203 EN**: Initializes variable `F32VectorSuffixes` from the right-hand expression.
  **L203 CN**: 使用右侧表达式初始化变量 `F32VectorSuffixes`。
- **L204 EN**: Initializes variable `F64VectorSuffixes` from the right-hand expression.
  **L204 CN**: 使用右侧表达式初始化变量 `F64VectorSuffixes`。
- **L205 EN**: Blank line separating nearby declarations or logic blocks.
  **L205 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L206 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L206 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L207 EN**: Declares TableGen def `ACOS_#S`.
  **L207 CN**: 声明 TableGen def `ACOS_#S`。
- **L208 EN**: Declares TableGen def `ACOSH_#S`.
  **L208 CN**: 声明 TableGen def `ACOSH_#S`。
- **L209 EN**: Declares TableGen def `ASIN_#S`.
  **L209 CN**: 声明 TableGen def `ASIN_#S`。
- **L210 EN**: Declares TableGen def `ASINH_#S`.
  **L210 CN**: 声明 TableGen def `ASINH_#S`。
- **L211 EN**: Declares TableGen def `ATAN_#S`.
  **L211 CN**: 声明 TableGen def `ATAN_#S`。
- **L212 EN**: Declares TableGen def `ATAN2_#S`.
  **L212 CN**: 声明 TableGen def `ATAN2_#S`。
- **L213 EN**: Declares TableGen def `ATANH_#S`.
  **L213 CN**: 声明 TableGen def `ATANH_#S`。
- **L214 EN**: Declares TableGen def `CBRT_#S`.
  **L214 CN**: 声明 TableGen def `CBRT_#S`。
- **L215 EN**: Declares TableGen def `CEIL_#S`.
  **L215 CN**: 声明 TableGen def `CEIL_#S`。
- **L216 EN**: Declares TableGen def `COPYSIGN_#S`.
  **L216 CN**: 声明 TableGen def `COPYSIGN_#S`。
- **L217 EN**: Declares TableGen def `COS_#S`.
  **L217 CN**: 声明 TableGen def `COS_#S`。
- **L218 EN**: Declares TableGen def `COSH_#S`.
  **L218 CN**: 声明 TableGen def `COSH_#S`。
- **L219 EN**: Declares TableGen def `COSPI_#S`.
  **L219 CN**: 声明 TableGen def `COSPI_#S`。
- **L220 EN**: Declares TableGen def `ERFC_#S`.
  **L220 CN**: 声明 TableGen def `ERFC_#S`。
- **L221 EN**: Declares TableGen def `ERFCINV_#S`.
  **L221 CN**: 声明 TableGen def `ERFCINV_#S`。
- **L222 EN**: Declares TableGen def `ERF_#S`.
  **L222 CN**: 声明 TableGen def `ERF_#S`。
- **L223 EN**: Declares TableGen def `ERFINV_#S`.
  **L223 CN**: 声明 TableGen def `ERFINV_#S`。
- **L224 EN**: Declares TableGen def `EXP_#S`.
  **L224 CN**: 声明 TableGen def `EXP_#S`。

### Lines 225-256

````tablegen
  def EXP_FINITE_#S : RuntimeLibcall;
  def EXP10_#S : RuntimeLibcall;
  def EXP2_#S : RuntimeLibcall;
  def EXPM1_#S : RuntimeLibcall;
  def FABS_#S : RuntimeLibcall;
  def FDIM_#S : RuntimeLibcall;
  def FLOOR_#S : RuntimeLibcall;
  def FMA_#S : RuntimeLibcall;
  def FMAX_#S : RuntimeLibcall;
  def FMIN_#S : RuntimeLibcall;
  def REM_#S : RuntimeLibcall; // "fmod"
  def HYPOT_#S : RuntimeLibcall;
  def ILOGB_#S : RuntimeLibcall;
  def LDEXP_#S : RuntimeLibcall;
  def LGAMMA_#S : RuntimeLibcall;
  def LOG_#S : RuntimeLibcall;
  def LOG10_#S : RuntimeLibcall;
  def LOG1P_#S : RuntimeLibcall;
  def LOG2_#S : RuntimeLibcall;
  def LOGB_#S : RuntimeLibcall;
  def MODF_#S : RuntimeLibcall;
  def NEXTAFTER_#S : RuntimeLibcall;
  def POW_#S : RuntimeLibcall;
  def SINCOS_#S : RuntimeLibcall;
  def SINCOSPI_#S : RuntimeLibcall;
  def SIN_#S : RuntimeLibcall;
  def SINH_#S : RuntimeLibcall;
  def SINPI_#S : RuntimeLibcall;
  def SQRT_#S : RuntimeLibcall;
  def TAN_#S : RuntimeLibcall;
  def TANH_#S : RuntimeLibcall;
  def TGAMMA_#S : RuntimeLibcall;
````
- **L225 EN**: Declares TableGen def `EXP_FINITE_#S`.
  **L225 CN**: 声明 TableGen def `EXP_FINITE_#S`。
- **L226 EN**: Declares TableGen def `EXP10_#S`.
  **L226 CN**: 声明 TableGen def `EXP10_#S`。
- **L227 EN**: Declares TableGen def `EXP2_#S`.
  **L227 CN**: 声明 TableGen def `EXP2_#S`。
- **L228 EN**: Declares TableGen def `EXPM1_#S`.
  **L228 CN**: 声明 TableGen def `EXPM1_#S`。
- **L229 EN**: Declares TableGen def `FABS_#S`.
  **L229 CN**: 声明 TableGen def `FABS_#S`。
- **L230 EN**: Declares TableGen def `FDIM_#S`.
  **L230 CN**: 声明 TableGen def `FDIM_#S`。
- **L231 EN**: Declares TableGen def `FLOOR_#S`.
  **L231 CN**: 声明 TableGen def `FLOOR_#S`。
- **L232 EN**: Declares TableGen def `FMA_#S`.
  **L232 CN**: 声明 TableGen def `FMA_#S`。
- **L233 EN**: Declares TableGen def `FMAX_#S`.
  **L233 CN**: 声明 TableGen def `FMAX_#S`。
- **L234 EN**: Declares TableGen def `FMIN_#S`.
  **L234 CN**: 声明 TableGen def `FMIN_#S`。
- **L235 EN**: Declares TableGen def `REM_#S`.
  **L235 CN**: 声明 TableGen def `REM_#S`。
- **L236 EN**: Declares TableGen def `HYPOT_#S`.
  **L236 CN**: 声明 TableGen def `HYPOT_#S`。
- **L237 EN**: Declares TableGen def `ILOGB_#S`.
  **L237 CN**: 声明 TableGen def `ILOGB_#S`。
- **L238 EN**: Declares TableGen def `LDEXP_#S`.
  **L238 CN**: 声明 TableGen def `LDEXP_#S`。
- **L239 EN**: Declares TableGen def `LGAMMA_#S`.
  **L239 CN**: 声明 TableGen def `LGAMMA_#S`。
- **L240 EN**: Declares TableGen def `LOG_#S`.
  **L240 CN**: 声明 TableGen def `LOG_#S`。
- **L241 EN**: Declares TableGen def `LOG10_#S`.
  **L241 CN**: 声明 TableGen def `LOG10_#S`。
- **L242 EN**: Declares TableGen def `LOG1P_#S`.
  **L242 CN**: 声明 TableGen def `LOG1P_#S`。
- **L243 EN**: Declares TableGen def `LOG2_#S`.
  **L243 CN**: 声明 TableGen def `LOG2_#S`。
- **L244 EN**: Declares TableGen def `LOGB_#S`.
  **L244 CN**: 声明 TableGen def `LOGB_#S`。
- **L245 EN**: Declares TableGen def `MODF_#S`.
  **L245 CN**: 声明 TableGen def `MODF_#S`。
- **L246 EN**: Declares TableGen def `NEXTAFTER_#S`.
  **L246 CN**: 声明 TableGen def `NEXTAFTER_#S`。
- **L247 EN**: Declares TableGen def `POW_#S`.
  **L247 CN**: 声明 TableGen def `POW_#S`。
- **L248 EN**: Declares TableGen def `SINCOS_#S`.
  **L248 CN**: 声明 TableGen def `SINCOS_#S`。
- **L249 EN**: Declares TableGen def `SINCOSPI_#S`.
  **L249 CN**: 声明 TableGen def `SINCOSPI_#S`。
- **L250 EN**: Declares TableGen def `SIN_#S`.
  **L250 CN**: 声明 TableGen def `SIN_#S`。
- **L251 EN**: Declares TableGen def `SINH_#S`.
  **L251 CN**: 声明 TableGen def `SINH_#S`。
- **L252 EN**: Declares TableGen def `SINPI_#S`.
  **L252 CN**: 声明 TableGen def `SINPI_#S`。
- **L253 EN**: Declares TableGen def `SQRT_#S`.
  **L253 CN**: 声明 TableGen def `SQRT_#S`。
- **L254 EN**: Declares TableGen def `TAN_#S`.
  **L254 CN**: 声明 TableGen def `TAN_#S`。
- **L255 EN**: Declares TableGen def `TANH_#S`.
  **L255 CN**: 声明 TableGen def `TANH_#S`。
- **L256 EN**: Declares TableGen def `TGAMMA_#S`.
  **L256 CN**: 声明 TableGen def `TGAMMA_#S`。

### Lines 257-288

````tablegen
  def CDFNORM_#S : RuntimeLibcall;
  def CDFNORMINV_#S : RuntimeLibcall;
  def ROUND_#S : RuntimeLibcall;
}

foreach S = F64VectorSuffixes in {
  def LOG_FINITE_#S : RuntimeLibcall;
  def POW_FINITE_#S : RuntimeLibcall;
}

def FEGETENV : RuntimeLibcall;
def FESETENV : RuntimeLibcall;

def FEGETMODE : RuntimeLibcall;
def FESETMODE : RuntimeLibcall;

def FPEXT_BF16_F32 : RuntimeLibcall;
def FPEXT_F32_PPCF128 : RuntimeLibcall;
def FPEXT_F64_PPCF128 : RuntimeLibcall;
def FPEXT_F80_F128 : RuntimeLibcall;
def FPEXT_F64_F128 : RuntimeLibcall;
def FPEXT_F32_F128 : RuntimeLibcall;
def FPEXT_F16_F128 : RuntimeLibcall;
def FPEXT_F16_F80 : RuntimeLibcall;
def FPEXT_F32_F64 : RuntimeLibcall;
def FPEXT_F16_F64 : RuntimeLibcall;
def FPEXT_F16_F32 : RuntimeLibcall;
def FPROUND_F32_F16 : RuntimeLibcall;
def FPROUND_F64_F16 : RuntimeLibcall;
def FPROUND_F80_F16 : RuntimeLibcall;
def FPROUND_F128_F16 : RuntimeLibcall;
def FPROUND_PPCF128_F16 : RuntimeLibcall;
````
- **L257 EN**: Declares TableGen def `CDFNORM_#S`.
  **L257 CN**: 声明 TableGen def `CDFNORM_#S`。
- **L258 EN**: Declares TableGen def `CDFNORMINV_#S`.
  **L258 CN**: 声明 TableGen def `CDFNORMINV_#S`。
- **L259 EN**: Declares TableGen def `ROUND_#S`.
  **L259 CN**: 声明 TableGen def `ROUND_#S`。
- **L260 EN**: Closes the current lexical scope or compound statement.
  **L260 CN**: 结束当前词法作用域或复合语句块。
- **L261 EN**: Blank line separating nearby declarations or logic blocks.
  **L261 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L262 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L262 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L263 EN**: Declares TableGen def `LOG_FINITE_#S`.
  **L263 CN**: 声明 TableGen def `LOG_FINITE_#S`。
- **L264 EN**: Declares TableGen def `POW_FINITE_#S`.
  **L264 CN**: 声明 TableGen def `POW_FINITE_#S`。
- **L265 EN**: Closes the current lexical scope or compound statement.
  **L265 CN**: 结束当前词法作用域或复合语句块。
- **L266 EN**: Blank line separating nearby declarations or logic blocks.
  **L266 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L267 EN**: Declares TableGen def `FEGETENV`.
  **L267 CN**: 声明 TableGen def `FEGETENV`。
- **L268 EN**: Declares TableGen def `FESETENV`.
  **L268 CN**: 声明 TableGen def `FESETENV`。
- **L269 EN**: Blank line separating nearby declarations or logic blocks.
  **L269 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L270 EN**: Declares TableGen def `FEGETMODE`.
  **L270 CN**: 声明 TableGen def `FEGETMODE`。
- **L271 EN**: Declares TableGen def `FESETMODE`.
  **L271 CN**: 声明 TableGen def `FESETMODE`。
- **L272 EN**: Blank line separating nearby declarations or logic blocks.
  **L272 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L273 EN**: Declares TableGen def `FPEXT_BF16_F32`.
  **L273 CN**: 声明 TableGen def `FPEXT_BF16_F32`。
- **L274 EN**: Declares TableGen def `FPEXT_F32_PPCF128`.
  **L274 CN**: 声明 TableGen def `FPEXT_F32_PPCF128`。
- **L275 EN**: Declares TableGen def `FPEXT_F64_PPCF128`.
  **L275 CN**: 声明 TableGen def `FPEXT_F64_PPCF128`。
- **L276 EN**: Declares TableGen def `FPEXT_F80_F128`.
  **L276 CN**: 声明 TableGen def `FPEXT_F80_F128`。
- **L277 EN**: Declares TableGen def `FPEXT_F64_F128`.
  **L277 CN**: 声明 TableGen def `FPEXT_F64_F128`。
- **L278 EN**: Declares TableGen def `FPEXT_F32_F128`.
  **L278 CN**: 声明 TableGen def `FPEXT_F32_F128`。
- **L279 EN**: Declares TableGen def `FPEXT_F16_F128`.
  **L279 CN**: 声明 TableGen def `FPEXT_F16_F128`。
- **L280 EN**: Declares TableGen def `FPEXT_F16_F80`.
  **L280 CN**: 声明 TableGen def `FPEXT_F16_F80`。
- **L281 EN**: Declares TableGen def `FPEXT_F32_F64`.
  **L281 CN**: 声明 TableGen def `FPEXT_F32_F64`。
- **L282 EN**: Declares TableGen def `FPEXT_F16_F64`.
  **L282 CN**: 声明 TableGen def `FPEXT_F16_F64`。
- **L283 EN**: Declares TableGen def `FPEXT_F16_F32`.
  **L283 CN**: 声明 TableGen def `FPEXT_F16_F32`。
- **L284 EN**: Declares TableGen def `FPROUND_F32_F16`.
  **L284 CN**: 声明 TableGen def `FPROUND_F32_F16`。
- **L285 EN**: Declares TableGen def `FPROUND_F64_F16`.
  **L285 CN**: 声明 TableGen def `FPROUND_F64_F16`。
- **L286 EN**: Declares TableGen def `FPROUND_F80_F16`.
  **L286 CN**: 声明 TableGen def `FPROUND_F80_F16`。
- **L287 EN**: Declares TableGen def `FPROUND_F128_F16`.
  **L287 CN**: 声明 TableGen def `FPROUND_F128_F16`。
- **L288 EN**: Declares TableGen def `FPROUND_PPCF128_F16`.
  **L288 CN**: 声明 TableGen def `FPROUND_PPCF128_F16`。

### Lines 289-320

````tablegen
def FPROUND_F32_BF16 : RuntimeLibcall;
def FPROUND_F64_BF16 : RuntimeLibcall;
def FPROUND_F80_BF16 : RuntimeLibcall;
def FPROUND_F128_BF16 : RuntimeLibcall;
def FPROUND_F64_F32 : RuntimeLibcall;
def FPROUND_F80_F32 : RuntimeLibcall;
def FPROUND_F128_F32 : RuntimeLibcall;
def FPROUND_PPCF128_F32 : RuntimeLibcall;
def FPROUND_F80_F64 : RuntimeLibcall;
def FPROUND_F128_F64 : RuntimeLibcall;
def FPROUND_PPCF128_F64 : RuntimeLibcall;
def FPROUND_F128_F80 : RuntimeLibcall;
def FPTOSINT_F16_I32 : RuntimeLibcall;
def FPTOSINT_F16_I64 : RuntimeLibcall;
def FPTOSINT_F16_I128 : RuntimeLibcall;
def FPTOSINT_F32_I32 : RuntimeLibcall;
def FPTOSINT_F32_I64 : RuntimeLibcall;
def FPTOSINT_F32_I128 : RuntimeLibcall;
def FPTOSINT_F64_I32 : RuntimeLibcall;
def FPTOSINT_F64_I64 : RuntimeLibcall;
def FPTOSINT_F64_I128 : RuntimeLibcall;
def FPTOSINT_F80_I32 : RuntimeLibcall;
def FPTOSINT_F80_I64 : RuntimeLibcall;
def FPTOSINT_F80_I128 : RuntimeLibcall;
def FPTOSINT_F128_I32 : RuntimeLibcall;
def FPTOSINT_F128_I64 : RuntimeLibcall;
def FPTOSINT_F128_I128 : RuntimeLibcall;
def FPTOSINT_PPCF128_I32 : RuntimeLibcall;
def FPTOSINT_PPCF128_I64 : RuntimeLibcall;
def FPTOSINT_PPCF128_I128 : RuntimeLibcall;
def FPTOUINT_F16_I32 : RuntimeLibcall;
def FPTOUINT_F16_I64 : RuntimeLibcall;
````
- **L289 EN**: Declares TableGen def `FPROUND_F32_BF16`.
  **L289 CN**: 声明 TableGen def `FPROUND_F32_BF16`。
- **L290 EN**: Declares TableGen def `FPROUND_F64_BF16`.
  **L290 CN**: 声明 TableGen def `FPROUND_F64_BF16`。
- **L291 EN**: Declares TableGen def `FPROUND_F80_BF16`.
  **L291 CN**: 声明 TableGen def `FPROUND_F80_BF16`。
- **L292 EN**: Declares TableGen def `FPROUND_F128_BF16`.
  **L292 CN**: 声明 TableGen def `FPROUND_F128_BF16`。
- **L293 EN**: Declares TableGen def `FPROUND_F64_F32`.
  **L293 CN**: 声明 TableGen def `FPROUND_F64_F32`。
- **L294 EN**: Declares TableGen def `FPROUND_F80_F32`.
  **L294 CN**: 声明 TableGen def `FPROUND_F80_F32`。
- **L295 EN**: Declares TableGen def `FPROUND_F128_F32`.
  **L295 CN**: 声明 TableGen def `FPROUND_F128_F32`。
- **L296 EN**: Declares TableGen def `FPROUND_PPCF128_F32`.
  **L296 CN**: 声明 TableGen def `FPROUND_PPCF128_F32`。
- **L297 EN**: Declares TableGen def `FPROUND_F80_F64`.
  **L297 CN**: 声明 TableGen def `FPROUND_F80_F64`。
- **L298 EN**: Declares TableGen def `FPROUND_F128_F64`.
  **L298 CN**: 声明 TableGen def `FPROUND_F128_F64`。
- **L299 EN**: Declares TableGen def `FPROUND_PPCF128_F64`.
  **L299 CN**: 声明 TableGen def `FPROUND_PPCF128_F64`。
- **L300 EN**: Declares TableGen def `FPROUND_F128_F80`.
  **L300 CN**: 声明 TableGen def `FPROUND_F128_F80`。
- **L301 EN**: Declares TableGen def `FPTOSINT_F16_I32`.
  **L301 CN**: 声明 TableGen def `FPTOSINT_F16_I32`。
- **L302 EN**: Declares TableGen def `FPTOSINT_F16_I64`.
  **L302 CN**: 声明 TableGen def `FPTOSINT_F16_I64`。
- **L303 EN**: Declares TableGen def `FPTOSINT_F16_I128`.
  **L303 CN**: 声明 TableGen def `FPTOSINT_F16_I128`。
- **L304 EN**: Declares TableGen def `FPTOSINT_F32_I32`.
  **L304 CN**: 声明 TableGen def `FPTOSINT_F32_I32`。
- **L305 EN**: Declares TableGen def `FPTOSINT_F32_I64`.
  **L305 CN**: 声明 TableGen def `FPTOSINT_F32_I64`。
- **L306 EN**: Declares TableGen def `FPTOSINT_F32_I128`.
  **L306 CN**: 声明 TableGen def `FPTOSINT_F32_I128`。
- **L307 EN**: Declares TableGen def `FPTOSINT_F64_I32`.
  **L307 CN**: 声明 TableGen def `FPTOSINT_F64_I32`。
- **L308 EN**: Declares TableGen def `FPTOSINT_F64_I64`.
  **L308 CN**: 声明 TableGen def `FPTOSINT_F64_I64`。
- **L309 EN**: Declares TableGen def `FPTOSINT_F64_I128`.
  **L309 CN**: 声明 TableGen def `FPTOSINT_F64_I128`。
- **L310 EN**: Declares TableGen def `FPTOSINT_F80_I32`.
  **L310 CN**: 声明 TableGen def `FPTOSINT_F80_I32`。
- **L311 EN**: Declares TableGen def `FPTOSINT_F80_I64`.
  **L311 CN**: 声明 TableGen def `FPTOSINT_F80_I64`。
- **L312 EN**: Declares TableGen def `FPTOSINT_F80_I128`.
  **L312 CN**: 声明 TableGen def `FPTOSINT_F80_I128`。
- **L313 EN**: Declares TableGen def `FPTOSINT_F128_I32`.
  **L313 CN**: 声明 TableGen def `FPTOSINT_F128_I32`。
- **L314 EN**: Declares TableGen def `FPTOSINT_F128_I64`.
  **L314 CN**: 声明 TableGen def `FPTOSINT_F128_I64`。
- **L315 EN**: Declares TableGen def `FPTOSINT_F128_I128`.
  **L315 CN**: 声明 TableGen def `FPTOSINT_F128_I128`。
- **L316 EN**: Declares TableGen def `FPTOSINT_PPCF128_I32`.
  **L316 CN**: 声明 TableGen def `FPTOSINT_PPCF128_I32`。
- **L317 EN**: Declares TableGen def `FPTOSINT_PPCF128_I64`.
  **L317 CN**: 声明 TableGen def `FPTOSINT_PPCF128_I64`。
- **L318 EN**: Declares TableGen def `FPTOSINT_PPCF128_I128`.
  **L318 CN**: 声明 TableGen def `FPTOSINT_PPCF128_I128`。
- **L319 EN**: Declares TableGen def `FPTOUINT_F16_I32`.
  **L319 CN**: 声明 TableGen def `FPTOUINT_F16_I32`。
- **L320 EN**: Declares TableGen def `FPTOUINT_F16_I64`.
  **L320 CN**: 声明 TableGen def `FPTOUINT_F16_I64`。

### Lines 321-352

````tablegen
def FPTOUINT_F16_I128 : RuntimeLibcall;
def FPTOUINT_F32_I32 : RuntimeLibcall;
def FPTOUINT_F32_I64 : RuntimeLibcall;
def FPTOUINT_F32_I128 : RuntimeLibcall;
def FPTOUINT_F64_I32 : RuntimeLibcall;
def FPTOUINT_F64_I64 : RuntimeLibcall;
def FPTOUINT_F64_I128 : RuntimeLibcall;
def FPTOUINT_F80_I32 : RuntimeLibcall;
def FPTOUINT_F80_I64 : RuntimeLibcall;
def FPTOUINT_F80_I128 : RuntimeLibcall;
def FPTOUINT_F128_I32 : RuntimeLibcall;
def FPTOUINT_F128_I64 : RuntimeLibcall;
def FPTOUINT_F128_I128 : RuntimeLibcall;
def FPTOUINT_PPCF128_I32 : RuntimeLibcall;
def FPTOUINT_PPCF128_I64 : RuntimeLibcall;
def FPTOUINT_PPCF128_I128 : RuntimeLibcall;
def SINTTOFP_I32_F16 : RuntimeLibcall;
def SINTTOFP_I32_F32 : RuntimeLibcall;
def SINTTOFP_I32_F64 : RuntimeLibcall;
def SINTTOFP_I32_F80 : RuntimeLibcall;
def SINTTOFP_I32_F128 : RuntimeLibcall;
def SINTTOFP_I32_PPCF128 : RuntimeLibcall;
def SINTTOFP_I64_BF16 : RuntimeLibcall;
def SINTTOFP_I64_F16 : RuntimeLibcall;
def SINTTOFP_I64_F32 : RuntimeLibcall;
def SINTTOFP_I64_F64 : RuntimeLibcall;
def SINTTOFP_I64_F80 : RuntimeLibcall;
def SINTTOFP_I64_F128 : RuntimeLibcall;
def SINTTOFP_I64_PPCF128 : RuntimeLibcall;
def SINTTOFP_I128_F16 : RuntimeLibcall;
def SINTTOFP_I128_F32 : RuntimeLibcall;
def SINTTOFP_I128_F64 : RuntimeLibcall;
````
- **L321 EN**: Declares TableGen def `FPTOUINT_F16_I128`.
  **L321 CN**: 声明 TableGen def `FPTOUINT_F16_I128`。
- **L322 EN**: Declares TableGen def `FPTOUINT_F32_I32`.
  **L322 CN**: 声明 TableGen def `FPTOUINT_F32_I32`。
- **L323 EN**: Declares TableGen def `FPTOUINT_F32_I64`.
  **L323 CN**: 声明 TableGen def `FPTOUINT_F32_I64`。
- **L324 EN**: Declares TableGen def `FPTOUINT_F32_I128`.
  **L324 CN**: 声明 TableGen def `FPTOUINT_F32_I128`。
- **L325 EN**: Declares TableGen def `FPTOUINT_F64_I32`.
  **L325 CN**: 声明 TableGen def `FPTOUINT_F64_I32`。
- **L326 EN**: Declares TableGen def `FPTOUINT_F64_I64`.
  **L326 CN**: 声明 TableGen def `FPTOUINT_F64_I64`。
- **L327 EN**: Declares TableGen def `FPTOUINT_F64_I128`.
  **L327 CN**: 声明 TableGen def `FPTOUINT_F64_I128`。
- **L328 EN**: Declares TableGen def `FPTOUINT_F80_I32`.
  **L328 CN**: 声明 TableGen def `FPTOUINT_F80_I32`。
- **L329 EN**: Declares TableGen def `FPTOUINT_F80_I64`.
  **L329 CN**: 声明 TableGen def `FPTOUINT_F80_I64`。
- **L330 EN**: Declares TableGen def `FPTOUINT_F80_I128`.
  **L330 CN**: 声明 TableGen def `FPTOUINT_F80_I128`。
- **L331 EN**: Declares TableGen def `FPTOUINT_F128_I32`.
  **L331 CN**: 声明 TableGen def `FPTOUINT_F128_I32`。
- **L332 EN**: Declares TableGen def `FPTOUINT_F128_I64`.
  **L332 CN**: 声明 TableGen def `FPTOUINT_F128_I64`。
- **L333 EN**: Declares TableGen def `FPTOUINT_F128_I128`.
  **L333 CN**: 声明 TableGen def `FPTOUINT_F128_I128`。
- **L334 EN**: Declares TableGen def `FPTOUINT_PPCF128_I32`.
  **L334 CN**: 声明 TableGen def `FPTOUINT_PPCF128_I32`。
- **L335 EN**: Declares TableGen def `FPTOUINT_PPCF128_I64`.
  **L335 CN**: 声明 TableGen def `FPTOUINT_PPCF128_I64`。
- **L336 EN**: Declares TableGen def `FPTOUINT_PPCF128_I128`.
  **L336 CN**: 声明 TableGen def `FPTOUINT_PPCF128_I128`。
- **L337 EN**: Declares TableGen def `SINTTOFP_I32_F16`.
  **L337 CN**: 声明 TableGen def `SINTTOFP_I32_F16`。
- **L338 EN**: Declares TableGen def `SINTTOFP_I32_F32`.
  **L338 CN**: 声明 TableGen def `SINTTOFP_I32_F32`。
- **L339 EN**: Declares TableGen def `SINTTOFP_I32_F64`.
  **L339 CN**: 声明 TableGen def `SINTTOFP_I32_F64`。
- **L340 EN**: Declares TableGen def `SINTTOFP_I32_F80`.
  **L340 CN**: 声明 TableGen def `SINTTOFP_I32_F80`。
- **L341 EN**: Declares TableGen def `SINTTOFP_I32_F128`.
  **L341 CN**: 声明 TableGen def `SINTTOFP_I32_F128`。
- **L342 EN**: Declares TableGen def `SINTTOFP_I32_PPCF128`.
  **L342 CN**: 声明 TableGen def `SINTTOFP_I32_PPCF128`。
- **L343 EN**: Declares TableGen def `SINTTOFP_I64_BF16`.
  **L343 CN**: 声明 TableGen def `SINTTOFP_I64_BF16`。
- **L344 EN**: Declares TableGen def `SINTTOFP_I64_F16`.
  **L344 CN**: 声明 TableGen def `SINTTOFP_I64_F16`。
- **L345 EN**: Declares TableGen def `SINTTOFP_I64_F32`.
  **L345 CN**: 声明 TableGen def `SINTTOFP_I64_F32`。
- **L346 EN**: Declares TableGen def `SINTTOFP_I64_F64`.
  **L346 CN**: 声明 TableGen def `SINTTOFP_I64_F64`。
- **L347 EN**: Declares TableGen def `SINTTOFP_I64_F80`.
  **L347 CN**: 声明 TableGen def `SINTTOFP_I64_F80`。
- **L348 EN**: Declares TableGen def `SINTTOFP_I64_F128`.
  **L348 CN**: 声明 TableGen def `SINTTOFP_I64_F128`。
- **L349 EN**: Declares TableGen def `SINTTOFP_I64_PPCF128`.
  **L349 CN**: 声明 TableGen def `SINTTOFP_I64_PPCF128`。
- **L350 EN**: Declares TableGen def `SINTTOFP_I128_F16`.
  **L350 CN**: 声明 TableGen def `SINTTOFP_I128_F16`。
- **L351 EN**: Declares TableGen def `SINTTOFP_I128_F32`.
  **L351 CN**: 声明 TableGen def `SINTTOFP_I128_F32`。
- **L352 EN**: Declares TableGen def `SINTTOFP_I128_F64`.
  **L352 CN**: 声明 TableGen def `SINTTOFP_I128_F64`。

### Lines 353-384

````tablegen
def SINTTOFP_I128_F80 : RuntimeLibcall;
def SINTTOFP_I128_F128 : RuntimeLibcall;
def SINTTOFP_I128_PPCF128 : RuntimeLibcall;
def UINTTOFP_I32_F16 : RuntimeLibcall;
def UINTTOFP_I32_F32 : RuntimeLibcall;
def UINTTOFP_I32_F64 : RuntimeLibcall;
def UINTTOFP_I32_F80 : RuntimeLibcall;
def UINTTOFP_I32_F128 : RuntimeLibcall;
def UINTTOFP_I32_PPCF128 : RuntimeLibcall;
def UINTTOFP_I64_BF16 : RuntimeLibcall;
def UINTTOFP_I64_F16 : RuntimeLibcall;
def UINTTOFP_I64_F32 : RuntimeLibcall;
def UINTTOFP_I64_F64 : RuntimeLibcall;
def UINTTOFP_I64_F80 : RuntimeLibcall;
def UINTTOFP_I64_F128 : RuntimeLibcall;
def UINTTOFP_I64_PPCF128 : RuntimeLibcall;
def UINTTOFP_I128_F16 : RuntimeLibcall;
def UINTTOFP_I128_F32 : RuntimeLibcall;
def UINTTOFP_I128_F64 : RuntimeLibcall;
def UINTTOFP_I128_F80 : RuntimeLibcall;
def UINTTOFP_I128_F128 : RuntimeLibcall;
def UINTTOFP_I128_PPCF128 : RuntimeLibcall;
def CONVERT_F128_PPCF128 : RuntimeLibcall;
def CONVERT_PPCF128_F128 : RuntimeLibcall;

// Comparisons
foreach FPTy = ["F32", "F64", "F128", "PPCF128"] in {
  def OEQ_#FPTy : RuntimeLibcall;
  def UNE_#FPTy : RuntimeLibcall;
  def OGE_#FPTy : RuntimeLibcall;
  def OLT_#FPTy : RuntimeLibcall;
  def OLE_#FPTy : RuntimeLibcall;
````
- **L353 EN**: Declares TableGen def `SINTTOFP_I128_F80`.
  **L353 CN**: 声明 TableGen def `SINTTOFP_I128_F80`。
- **L354 EN**: Declares TableGen def `SINTTOFP_I128_F128`.
  **L354 CN**: 声明 TableGen def `SINTTOFP_I128_F128`。
- **L355 EN**: Declares TableGen def `SINTTOFP_I128_PPCF128`.
  **L355 CN**: 声明 TableGen def `SINTTOFP_I128_PPCF128`。
- **L356 EN**: Declares TableGen def `UINTTOFP_I32_F16`.
  **L356 CN**: 声明 TableGen def `UINTTOFP_I32_F16`。
- **L357 EN**: Declares TableGen def `UINTTOFP_I32_F32`.
  **L357 CN**: 声明 TableGen def `UINTTOFP_I32_F32`。
- **L358 EN**: Declares TableGen def `UINTTOFP_I32_F64`.
  **L358 CN**: 声明 TableGen def `UINTTOFP_I32_F64`。
- **L359 EN**: Declares TableGen def `UINTTOFP_I32_F80`.
  **L359 CN**: 声明 TableGen def `UINTTOFP_I32_F80`。
- **L360 EN**: Declares TableGen def `UINTTOFP_I32_F128`.
  **L360 CN**: 声明 TableGen def `UINTTOFP_I32_F128`。
- **L361 EN**: Declares TableGen def `UINTTOFP_I32_PPCF128`.
  **L361 CN**: 声明 TableGen def `UINTTOFP_I32_PPCF128`。
- **L362 EN**: Declares TableGen def `UINTTOFP_I64_BF16`.
  **L362 CN**: 声明 TableGen def `UINTTOFP_I64_BF16`。
- **L363 EN**: Declares TableGen def `UINTTOFP_I64_F16`.
  **L363 CN**: 声明 TableGen def `UINTTOFP_I64_F16`。
- **L364 EN**: Declares TableGen def `UINTTOFP_I64_F32`.
  **L364 CN**: 声明 TableGen def `UINTTOFP_I64_F32`。
- **L365 EN**: Declares TableGen def `UINTTOFP_I64_F64`.
  **L365 CN**: 声明 TableGen def `UINTTOFP_I64_F64`。
- **L366 EN**: Declares TableGen def `UINTTOFP_I64_F80`.
  **L366 CN**: 声明 TableGen def `UINTTOFP_I64_F80`。
- **L367 EN**: Declares TableGen def `UINTTOFP_I64_F128`.
  **L367 CN**: 声明 TableGen def `UINTTOFP_I64_F128`。
- **L368 EN**: Declares TableGen def `UINTTOFP_I64_PPCF128`.
  **L368 CN**: 声明 TableGen def `UINTTOFP_I64_PPCF128`。
- **L369 EN**: Declares TableGen def `UINTTOFP_I128_F16`.
  **L369 CN**: 声明 TableGen def `UINTTOFP_I128_F16`。
- **L370 EN**: Declares TableGen def `UINTTOFP_I128_F32`.
  **L370 CN**: 声明 TableGen def `UINTTOFP_I128_F32`。
- **L371 EN**: Declares TableGen def `UINTTOFP_I128_F64`.
  **L371 CN**: 声明 TableGen def `UINTTOFP_I128_F64`。
- **L372 EN**: Declares TableGen def `UINTTOFP_I128_F80`.
  **L372 CN**: 声明 TableGen def `UINTTOFP_I128_F80`。
- **L373 EN**: Declares TableGen def `UINTTOFP_I128_F128`.
  **L373 CN**: 声明 TableGen def `UINTTOFP_I128_F128`。
- **L374 EN**: Declares TableGen def `UINTTOFP_I128_PPCF128`.
  **L374 CN**: 声明 TableGen def `UINTTOFP_I128_PPCF128`。
- **L375 EN**: Declares TableGen def `CONVERT_F128_PPCF128`.
  **L375 CN**: 声明 TableGen def `CONVERT_F128_PPCF128`。
- **L376 EN**: Declares TableGen def `CONVERT_PPCF128_F128`.
  **L376 CN**: 声明 TableGen def `CONVERT_PPCF128_F128`。
- **L377 EN**: Blank line separating nearby declarations or logic blocks.
  **L377 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L378 EN**: Comment explains nearby logic, invariants, or intent: `Comparisons`.
  **L378 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Comparisons`。
- **L379 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L379 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L380 EN**: Declares TableGen def `OEQ_#FPTy`.
  **L380 CN**: 声明 TableGen def `OEQ_#FPTy`。
- **L381 EN**: Declares TableGen def `UNE_#FPTy`.
  **L381 CN**: 声明 TableGen def `UNE_#FPTy`。
- **L382 EN**: Declares TableGen def `OGE_#FPTy`.
  **L382 CN**: 声明 TableGen def `OGE_#FPTy`。
- **L383 EN**: Declares TableGen def `OLT_#FPTy`.
  **L383 CN**: 声明 TableGen def `OLT_#FPTy`。
- **L384 EN**: Declares TableGen def `OLE_#FPTy`.
  **L384 CN**: 声明 TableGen def `OLE_#FPTy`。

### Lines 385-416

````tablegen
  def OGT_#FPTy : RuntimeLibcall;
  def UO_#FPTy : RuntimeLibcall;
}

// Memory
def MEMCMP : RuntimeLibcall;
def MEMCPY : RuntimeLibcall;
def MEMCPY_CHK : RuntimeLibcall;
def MEMMOVE : RuntimeLibcall;
def MEMMOVE_CHK : RuntimeLibcall;
def MEMSET : RuntimeLibcall;
def MEMSET_CHK : RuntimeLibcall;
def MALLOC : RuntimeLibcall;
def CALLOC : RuntimeLibcall;
def FREE : RuntimeLibcall;
def BZERO : RuntimeLibcall;
def STRLEN : RuntimeLibcall;

// Element-wise unordered-atomic memory of different sizes
foreach MemSize = [1, 2, 4, 8, 16] in {
  def MEMCPY_ELEMENT_UNORDERED_ATOMIC_#MemSize : RuntimeLibcall;
  def MEMMOVE_ELEMENT_UNORDERED_ATOMIC_#MemSize : RuntimeLibcall;
  def MEMSET_ELEMENT_UNORDERED_ATOMIC_#MemSize : RuntimeLibcall;
}

// Exception handling
def UNWIND_RESUME : RuntimeLibcall;
def UNWIND_REGISTER : RuntimeLibcall;
def UNWIND_UNREGISTER : RuntimeLibcall;
def UNWIND_CALL_PERSONALITY : RuntimeLibcall;
def CXA_END_CLEANUP : RuntimeLibcall;

````
- **L385 EN**: Declares TableGen def `OGT_#FPTy`.
  **L385 CN**: 声明 TableGen def `OGT_#FPTy`。
- **L386 EN**: Declares TableGen def `UO_#FPTy`.
  **L386 CN**: 声明 TableGen def `UO_#FPTy`。
- **L387 EN**: Closes the current lexical scope or compound statement.
  **L387 CN**: 结束当前词法作用域或复合语句块。
- **L388 EN**: Blank line separating nearby declarations or logic blocks.
  **L388 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L389 EN**: Comment explains nearby logic, invariants, or intent: `Memory`.
  **L389 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Memory`。
- **L390 EN**: Declares TableGen def `MEMCMP`.
  **L390 CN**: 声明 TableGen def `MEMCMP`。
- **L391 EN**: Declares TableGen def `MEMCPY`.
  **L391 CN**: 声明 TableGen def `MEMCPY`。
- **L392 EN**: Declares TableGen def `MEMCPY_CHK`.
  **L392 CN**: 声明 TableGen def `MEMCPY_CHK`。
- **L393 EN**: Declares TableGen def `MEMMOVE`.
  **L393 CN**: 声明 TableGen def `MEMMOVE`。
- **L394 EN**: Declares TableGen def `MEMMOVE_CHK`.
  **L394 CN**: 声明 TableGen def `MEMMOVE_CHK`。
- **L395 EN**: Declares TableGen def `MEMSET`.
  **L395 CN**: 声明 TableGen def `MEMSET`。
- **L396 EN**: Declares TableGen def `MEMSET_CHK`.
  **L396 CN**: 声明 TableGen def `MEMSET_CHK`。
- **L397 EN**: Declares TableGen def `MALLOC`.
  **L397 CN**: 声明 TableGen def `MALLOC`。
- **L398 EN**: Declares TableGen def `CALLOC`.
  **L398 CN**: 声明 TableGen def `CALLOC`。
- **L399 EN**: Declares TableGen def `FREE`.
  **L399 CN**: 声明 TableGen def `FREE`。
- **L400 EN**: Declares TableGen def `BZERO`.
  **L400 CN**: 声明 TableGen def `BZERO`。
- **L401 EN**: Declares TableGen def `STRLEN`.
  **L401 CN**: 声明 TableGen def `STRLEN`。
- **L402 EN**: Blank line separating nearby declarations or logic blocks.
  **L402 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L403 EN**: Comment explains nearby logic, invariants, or intent: `Element-wise unordered-atomic memory of different sizes`.
  **L403 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Element-wise unordered-atomic memory of different sizes`。
- **L404 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L404 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L405 EN**: Declares TableGen def `MEMCPY_ELEMENT_UNORDERED_ATOMIC_#MemSize`.
  **L405 CN**: 声明 TableGen def `MEMCPY_ELEMENT_UNORDERED_ATOMIC_#MemSize`。
- **L406 EN**: Declares TableGen def `MEMMOVE_ELEMENT_UNORDERED_ATOMIC_#MemSize`.
  **L406 CN**: 声明 TableGen def `MEMMOVE_ELEMENT_UNORDERED_ATOMIC_#MemSize`。
- **L407 EN**: Declares TableGen def `MEMSET_ELEMENT_UNORDERED_ATOMIC_#MemSize`.
  **L407 CN**: 声明 TableGen def `MEMSET_ELEMENT_UNORDERED_ATOMIC_#MemSize`。
- **L408 EN**: Closes the current lexical scope or compound statement.
  **L408 CN**: 结束当前词法作用域或复合语句块。
- **L409 EN**: Blank line separating nearby declarations or logic blocks.
  **L409 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L410 EN**: Comment explains nearby logic, invariants, or intent: `Exception handling`.
  **L410 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Exception handling`。
- **L411 EN**: Declares TableGen def `UNWIND_RESUME`.
  **L411 CN**: 声明 TableGen def `UNWIND_RESUME`。
- **L412 EN**: Declares TableGen def `UNWIND_REGISTER`.
  **L412 CN**: 声明 TableGen def `UNWIND_REGISTER`。
- **L413 EN**: Declares TableGen def `UNWIND_UNREGISTER`.
  **L413 CN**: 声明 TableGen def `UNWIND_UNREGISTER`。
- **L414 EN**: Declares TableGen def `UNWIND_CALL_PERSONALITY`.
  **L414 CN**: 声明 TableGen def `UNWIND_CALL_PERSONALITY`。
- **L415 EN**: Declares TableGen def `CXA_END_CLEANUP`.
  **L415 CN**: 声明 TableGen def `CXA_END_CLEANUP`。
- **L416 EN**: Blank line separating nearby declarations or logic blocks.
  **L416 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 417-448

````tablegen
// Note: there are two sets of atomics libcalls; see
// <https://llvm.org/docs/Atomics.html> for more info on the
// difference between them.

// Atomic '__sync_*' libcalls.
defset list<RuntimeLibcall> LibCalls__sync = {
  foreach MemSize = [1, 2, 4, 8, 16] in {
    def SYNC_VAL_COMPARE_AND_SWAP_#MemSize : RuntimeLibcall;
    def SYNC_LOCK_TEST_AND_SET_#MemSize : RuntimeLibcall;
    def SYNC_FETCH_AND_ADD_#MemSize : RuntimeLibcall;
    def SYNC_FETCH_AND_SUB_#MemSize : RuntimeLibcall;
    def SYNC_FETCH_AND_AND_#MemSize : RuntimeLibcall;
    def SYNC_FETCH_AND_OR_#MemSize : RuntimeLibcall;
    def SYNC_FETCH_AND_XOR_#MemSize : RuntimeLibcall;
    def SYNC_FETCH_AND_NAND_#MemSize : RuntimeLibcall;
    def SYNC_FETCH_AND_MAX_#MemSize : RuntimeLibcall;
    def SYNC_FETCH_AND_UMAX_#MemSize : RuntimeLibcall;
    def SYNC_FETCH_AND_MIN_#MemSize : RuntimeLibcall;
    def SYNC_FETCH_AND_UMIN_#MemSize : RuntimeLibcall;
  }
}

defset list<RuntimeLibcall> LibCalls__atomic = {
  // Atomic `__atomic_*' libcalls.
  foreach MemSize = ["", "_1", "_2", "_4", "_8", "_16"] in {
    def ATOMIC_LOAD#MemSize : RuntimeLibcall;
    def ATOMIC_STORE#MemSize : RuntimeLibcall;
    def ATOMIC_EXCHANGE#MemSize : RuntimeLibcall;
    def ATOMIC_COMPARE_EXCHANGE#MemSize : RuntimeLibcall;
  }

  foreach MemSize = [1, 2, 4, 8, 16] in {
````
- **L417 EN**: Comment explains nearby logic, invariants, or intent: `Note: there are two sets of atomics libcalls; see`.
  **L417 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Note: there are two sets of atomics libcalls; see`。
- **L418 EN**: Comment explains nearby logic, invariants, or intent: `<https://llvm.org/docs/Atomics.html> for more info on the`.
  **L418 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`<https://llvm.org/docs/Atomics.html> for more info on the`。
- **L419 EN**: Comment explains nearby logic, invariants, or intent: `difference between them.`.
  **L419 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`difference between them.`。
- **L420 EN**: Blank line separating nearby declarations or logic blocks.
  **L420 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L421 EN**: Comment explains nearby logic, invariants, or intent: `Atomic '__sync_*' libcalls.`.
  **L421 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Atomic '__sync_*' libcalls.`。
- **L422 EN**: Continues the surrounding expression or declaration: `defset list<RuntimeLibcall> LibCalls__sync = {`.
  **L422 CN**: 继续构造周围的表达式或声明：`defset list<RuntimeLibcall> LibCalls__sync = {`。
- **L423 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L423 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L424 EN**: Declares TableGen def `SYNC_VAL_COMPARE_AND_SWAP_#MemSize`.
  **L424 CN**: 声明 TableGen def `SYNC_VAL_COMPARE_AND_SWAP_#MemSize`。
- **L425 EN**: Declares TableGen def `SYNC_LOCK_TEST_AND_SET_#MemSize`.
  **L425 CN**: 声明 TableGen def `SYNC_LOCK_TEST_AND_SET_#MemSize`。
- **L426 EN**: Declares TableGen def `SYNC_FETCH_AND_ADD_#MemSize`.
  **L426 CN**: 声明 TableGen def `SYNC_FETCH_AND_ADD_#MemSize`。
- **L427 EN**: Declares TableGen def `SYNC_FETCH_AND_SUB_#MemSize`.
  **L427 CN**: 声明 TableGen def `SYNC_FETCH_AND_SUB_#MemSize`。
- **L428 EN**: Declares TableGen def `SYNC_FETCH_AND_AND_#MemSize`.
  **L428 CN**: 声明 TableGen def `SYNC_FETCH_AND_AND_#MemSize`。
- **L429 EN**: Declares TableGen def `SYNC_FETCH_AND_OR_#MemSize`.
  **L429 CN**: 声明 TableGen def `SYNC_FETCH_AND_OR_#MemSize`。
- **L430 EN**: Declares TableGen def `SYNC_FETCH_AND_XOR_#MemSize`.
  **L430 CN**: 声明 TableGen def `SYNC_FETCH_AND_XOR_#MemSize`。
- **L431 EN**: Declares TableGen def `SYNC_FETCH_AND_NAND_#MemSize`.
  **L431 CN**: 声明 TableGen def `SYNC_FETCH_AND_NAND_#MemSize`。
- **L432 EN**: Declares TableGen def `SYNC_FETCH_AND_MAX_#MemSize`.
  **L432 CN**: 声明 TableGen def `SYNC_FETCH_AND_MAX_#MemSize`。
- **L433 EN**: Declares TableGen def `SYNC_FETCH_AND_UMAX_#MemSize`.
  **L433 CN**: 声明 TableGen def `SYNC_FETCH_AND_UMAX_#MemSize`。
- **L434 EN**: Declares TableGen def `SYNC_FETCH_AND_MIN_#MemSize`.
  **L434 CN**: 声明 TableGen def `SYNC_FETCH_AND_MIN_#MemSize`。
- **L435 EN**: Declares TableGen def `SYNC_FETCH_AND_UMIN_#MemSize`.
  **L435 CN**: 声明 TableGen def `SYNC_FETCH_AND_UMIN_#MemSize`。
- **L436 EN**: Closes the current lexical scope or compound statement.
  **L436 CN**: 结束当前词法作用域或复合语句块。
- **L437 EN**: Closes the current lexical scope or compound statement.
  **L437 CN**: 结束当前词法作用域或复合语句块。
- **L438 EN**: Blank line separating nearby declarations or logic blocks.
  **L438 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L439 EN**: Continues the surrounding expression or declaration: `defset list<RuntimeLibcall> LibCalls__atomic = {`.
  **L439 CN**: 继续构造周围的表达式或声明：`defset list<RuntimeLibcall> LibCalls__atomic = {`。
- **L440 EN**: Comment explains nearby logic, invariants, or intent: `Atomic `__atomic_*' libcalls.`.
  **L440 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Atomic `__atomic_*' libcalls.`。
- **L441 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L441 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L442 EN**: Declares TableGen def `ATOMIC_LOAD#MemSize`.
  **L442 CN**: 声明 TableGen def `ATOMIC_LOAD#MemSize`。
- **L443 EN**: Declares TableGen def `ATOMIC_STORE#MemSize`.
  **L443 CN**: 声明 TableGen def `ATOMIC_STORE#MemSize`。
- **L444 EN**: Declares TableGen def `ATOMIC_EXCHANGE#MemSize`.
  **L444 CN**: 声明 TableGen def `ATOMIC_EXCHANGE#MemSize`。
- **L445 EN**: Declares TableGen def `ATOMIC_COMPARE_EXCHANGE#MemSize`.
  **L445 CN**: 声明 TableGen def `ATOMIC_COMPARE_EXCHANGE#MemSize`。
- **L446 EN**: Closes the current lexical scope or compound statement.
  **L446 CN**: 结束当前词法作用域或复合语句块。
- **L447 EN**: Blank line separating nearby declarations or logic blocks.
  **L447 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L448 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L448 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。

### Lines 449-480

````tablegen
    def ATOMIC_FETCH_ADD_#MemSize : RuntimeLibcall;
    def ATOMIC_FETCH_SUB_#MemSize : RuntimeLibcall;
    def ATOMIC_FETCH_AND_#MemSize : RuntimeLibcall;
    def ATOMIC_FETCH_OR_#MemSize : RuntimeLibcall;
    def ATOMIC_FETCH_XOR_#MemSize : RuntimeLibcall;
    def ATOMIC_FETCH_NAND_#MemSize : RuntimeLibcall;
  }
}

multiclass AtomicOrderSizeLibcall {
  def _RELAX : RuntimeLibcall;
  def _ACQ : RuntimeLibcall;
  def _REL : RuntimeLibcall;
  def _ACQ_REL : RuntimeLibcall;
}

multiclass AtomicOrderSizeLibcallImpl<string ProvidesBase> {
  def _relax : RuntimeLibcallImpl<!cast<RuntimeLibcall>(ProvidesBase#"_RELAX")>;
  def _acq : RuntimeLibcallImpl<!cast<RuntimeLibcall>(ProvidesBase#"_ACQ")>;
  def _rel : RuntimeLibcallImpl<!cast<RuntimeLibcall>(ProvidesBase#"_REL")>;
  def _acq_rel : RuntimeLibcallImpl<!cast<RuntimeLibcall>(ProvidesBase#"_ACQ_REL")>;
}

// Out-of-line atomics libcalls
defset list<RuntimeLibcall> LibCalls__OutOfLineAtomic = {
  foreach MemSize = [1, 2, 4, 8, 16] in {
    defm OUTLINE_ATOMIC_CAS#MemSize : AtomicOrderSizeLibcall;
    defm OUTLINE_ATOMIC_SWP#MemSize : AtomicOrderSizeLibcall;
    defm OUTLINE_ATOMIC_LDADD#MemSize : AtomicOrderSizeLibcall;
    defm OUTLINE_ATOMIC_LDSET#MemSize : AtomicOrderSizeLibcall;
    defm OUTLINE_ATOMIC_LDCLR#MemSize : AtomicOrderSizeLibcall;
    defm OUTLINE_ATOMIC_LDEOR#MemSize : AtomicOrderSizeLibcall;
````
- **L449 EN**: Declares TableGen def `ATOMIC_FETCH_ADD_#MemSize`.
  **L449 CN**: 声明 TableGen def `ATOMIC_FETCH_ADD_#MemSize`。
- **L450 EN**: Declares TableGen def `ATOMIC_FETCH_SUB_#MemSize`.
  **L450 CN**: 声明 TableGen def `ATOMIC_FETCH_SUB_#MemSize`。
- **L451 EN**: Declares TableGen def `ATOMIC_FETCH_AND_#MemSize`.
  **L451 CN**: 声明 TableGen def `ATOMIC_FETCH_AND_#MemSize`。
- **L452 EN**: Declares TableGen def `ATOMIC_FETCH_OR_#MemSize`.
  **L452 CN**: 声明 TableGen def `ATOMIC_FETCH_OR_#MemSize`。
- **L453 EN**: Declares TableGen def `ATOMIC_FETCH_XOR_#MemSize`.
  **L453 CN**: 声明 TableGen def `ATOMIC_FETCH_XOR_#MemSize`。
- **L454 EN**: Declares TableGen def `ATOMIC_FETCH_NAND_#MemSize`.
  **L454 CN**: 声明 TableGen def `ATOMIC_FETCH_NAND_#MemSize`。
- **L455 EN**: Closes the current lexical scope or compound statement.
  **L455 CN**: 结束当前词法作用域或复合语句块。
- **L456 EN**: Closes the current lexical scope or compound statement.
  **L456 CN**: 结束当前词法作用域或复合语句块。
- **L457 EN**: Blank line separating nearby declarations or logic blocks.
  **L457 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L458 EN**: Declares TableGen multiclass `AtomicOrderSizeLibcall`.
  **L458 CN**: 声明 TableGen multiclass `AtomicOrderSizeLibcall`。
- **L459 EN**: Declares TableGen def `_RELAX`.
  **L459 CN**: 声明 TableGen def `_RELAX`。
- **L460 EN**: Declares TableGen def `_ACQ`.
  **L460 CN**: 声明 TableGen def `_ACQ`。
- **L461 EN**: Declares TableGen def `_REL`.
  **L461 CN**: 声明 TableGen def `_REL`。
- **L462 EN**: Declares TableGen def `_ACQ_REL`.
  **L462 CN**: 声明 TableGen def `_ACQ_REL`。
- **L463 EN**: Closes the current lexical scope or compound statement.
  **L463 CN**: 结束当前词法作用域或复合语句块。
- **L464 EN**: Blank line separating nearby declarations or logic blocks.
  **L464 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L465 EN**: Declares TableGen multiclass `AtomicOrderSizeLibcallImpl`.
  **L465 CN**: 声明 TableGen multiclass `AtomicOrderSizeLibcallImpl`。
- **L466 EN**: Declares TableGen def `_relax`.
  **L466 CN**: 声明 TableGen def `_relax`。
- **L467 EN**: Declares TableGen def `_acq`.
  **L467 CN**: 声明 TableGen def `_acq`。
- **L468 EN**: Declares TableGen def `_rel`.
  **L468 CN**: 声明 TableGen def `_rel`。
- **L469 EN**: Declares TableGen def `_acq_rel`.
  **L469 CN**: 声明 TableGen def `_acq_rel`。
- **L470 EN**: Closes the current lexical scope or compound statement.
  **L470 CN**: 结束当前词法作用域或复合语句块。
- **L471 EN**: Blank line separating nearby declarations or logic blocks.
  **L471 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L472 EN**: Comment explains nearby logic, invariants, or intent: `Out-of-line atomics libcalls`.
  **L472 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Out-of-line atomics libcalls`。
- **L473 EN**: Continues the surrounding expression or declaration: `defset list<RuntimeLibcall> LibCalls__OutOfLineAtomic = {`.
  **L473 CN**: 继续构造周围的表达式或声明：`defset list<RuntimeLibcall> LibCalls__OutOfLineAtomic = {`。
- **L474 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L474 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L475 EN**: Declares TableGen defm `OUTLINE_ATOMIC_CAS#MemSize`.
  **L475 CN**: 声明 TableGen defm `OUTLINE_ATOMIC_CAS#MemSize`。
- **L476 EN**: Declares TableGen defm `OUTLINE_ATOMIC_SWP#MemSize`.
  **L476 CN**: 声明 TableGen defm `OUTLINE_ATOMIC_SWP#MemSize`。
- **L477 EN**: Declares TableGen defm `OUTLINE_ATOMIC_LDADD#MemSize`.
  **L477 CN**: 声明 TableGen defm `OUTLINE_ATOMIC_LDADD#MemSize`。
- **L478 EN**: Declares TableGen defm `OUTLINE_ATOMIC_LDSET#MemSize`.
  **L478 CN**: 声明 TableGen defm `OUTLINE_ATOMIC_LDSET#MemSize`。
- **L479 EN**: Declares TableGen defm `OUTLINE_ATOMIC_LDCLR#MemSize`.
  **L479 CN**: 声明 TableGen defm `OUTLINE_ATOMIC_LDCLR#MemSize`。
- **L480 EN**: Declares TableGen defm `OUTLINE_ATOMIC_LDEOR#MemSize`.
  **L480 CN**: 声明 TableGen defm `OUTLINE_ATOMIC_LDEOR#MemSize`。

### Lines 481-512

````tablegen
  }
}

// Stack Protector Fail
def STACKPROTECTOR_CHECK_FAIL : RuntimeLibcall;
def STACK_SMASH_HANDLER : RuntimeLibcall;

// Safe stack
def SAFESTACK_POINTER_ADDRESS : RuntimeLibcall;

// Really a global variable, not a function
def SAFESTACK_UNSAFE_STACK_PTR : RuntimeLibcall;

def STACK_PROBE : RuntimeLibcall;
def SECURITY_CHECK_COOKIE : RuntimeLibcall;

// Deoptimization
def DEOPTIMIZE : RuntimeLibcall;

// Return address
def RETURN_ADDRESS : RuntimeLibcall;

// Clear cache
def CLEAR_CACHE : RuntimeLibcall;
def RISCV_FLUSH_ICACHE : RuntimeLibcall;

// Mips16 calls
def MIPS16_RET_DC : RuntimeLibcall;
def MIPS16_RET_DF : RuntimeLibcall;
def MIPS16_RET_SC : RuntimeLibcall;
def MIPS16_RET_SF : RuntimeLibcall;

````
- **L481 EN**: Closes the current lexical scope or compound statement.
  **L481 CN**: 结束当前词法作用域或复合语句块。
- **L482 EN**: Closes the current lexical scope or compound statement.
  **L482 CN**: 结束当前词法作用域或复合语句块。
- **L483 EN**: Blank line separating nearby declarations or logic blocks.
  **L483 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L484 EN**: Comment explains nearby logic, invariants, or intent: `Stack Protector Fail`.
  **L484 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Stack Protector Fail`。
- **L485 EN**: Declares TableGen def `STACKPROTECTOR_CHECK_FAIL`.
  **L485 CN**: 声明 TableGen def `STACKPROTECTOR_CHECK_FAIL`。
- **L486 EN**: Declares TableGen def `STACK_SMASH_HANDLER`.
  **L486 CN**: 声明 TableGen def `STACK_SMASH_HANDLER`。
- **L487 EN**: Blank line separating nearby declarations or logic blocks.
  **L487 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L488 EN**: Comment explains nearby logic, invariants, or intent: `Safe stack`.
  **L488 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Safe stack`。
- **L489 EN**: Declares TableGen def `SAFESTACK_POINTER_ADDRESS`.
  **L489 CN**: 声明 TableGen def `SAFESTACK_POINTER_ADDRESS`。
- **L490 EN**: Blank line separating nearby declarations or logic blocks.
  **L490 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L491 EN**: Comment explains nearby logic, invariants, or intent: `Really a global variable, not a function`.
  **L491 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Really a global variable, not a function`。
- **L492 EN**: Declares TableGen def `SAFESTACK_UNSAFE_STACK_PTR`.
  **L492 CN**: 声明 TableGen def `SAFESTACK_UNSAFE_STACK_PTR`。
- **L493 EN**: Blank line separating nearby declarations or logic blocks.
  **L493 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L494 EN**: Declares TableGen def `STACK_PROBE`.
  **L494 CN**: 声明 TableGen def `STACK_PROBE`。
- **L495 EN**: Declares TableGen def `SECURITY_CHECK_COOKIE`.
  **L495 CN**: 声明 TableGen def `SECURITY_CHECK_COOKIE`。
- **L496 EN**: Blank line separating nearby declarations or logic blocks.
  **L496 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L497 EN**: Comment explains nearby logic, invariants, or intent: `Deoptimization`.
  **L497 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Deoptimization`。
- **L498 EN**: Declares TableGen def `DEOPTIMIZE`.
  **L498 CN**: 声明 TableGen def `DEOPTIMIZE`。
- **L499 EN**: Blank line separating nearby declarations or logic blocks.
  **L499 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L500 EN**: Comment explains nearby logic, invariants, or intent: `Return address`.
  **L500 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Return address`。
- **L501 EN**: Declares TableGen def `RETURN_ADDRESS`.
  **L501 CN**: 声明 TableGen def `RETURN_ADDRESS`。
- **L502 EN**: Blank line separating nearby declarations or logic blocks.
  **L502 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L503 EN**: Comment explains nearby logic, invariants, or intent: `Clear cache`.
  **L503 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Clear cache`。
- **L504 EN**: Declares TableGen def `CLEAR_CACHE`.
  **L504 CN**: 声明 TableGen def `CLEAR_CACHE`。
- **L505 EN**: Declares TableGen def `RISCV_FLUSH_ICACHE`.
  **L505 CN**: 声明 TableGen def `RISCV_FLUSH_ICACHE`。
- **L506 EN**: Blank line separating nearby declarations or logic blocks.
  **L506 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L507 EN**: Comment explains nearby logic, invariants, or intent: `Mips16 calls`.
  **L507 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Mips16 calls`。
- **L508 EN**: Declares TableGen def `MIPS16_RET_DC`.
  **L508 CN**: 声明 TableGen def `MIPS16_RET_DC`。
- **L509 EN**: Declares TableGen def `MIPS16_RET_DF`.
  **L509 CN**: 声明 TableGen def `MIPS16_RET_DF`。
- **L510 EN**: Declares TableGen def `MIPS16_RET_SC`.
  **L510 CN**: 声明 TableGen def `MIPS16_RET_SC`。
- **L511 EN**: Declares TableGen def `MIPS16_RET_SF`.
  **L511 CN**: 声明 TableGen def `MIPS16_RET_SF`。
- **L512 EN**: Blank line separating nearby declarations or logic blocks.
  **L512 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 513-544

````tablegen
multiclass LibmLongDoubleLibCall<string libcall_basename = !toupper(!substr(NAME, 0, !sub(!size(NAME), 1))),
                                 string rtname = NAME> {


  def NAME#"_f128"
      : RuntimeLibcallImpl<!cast<RuntimeLibcall>(libcall_basename#"_F128"),
                           rtname>;
  def NAME#"_ppcf128"
      : RuntimeLibcallImpl<!cast<RuntimeLibcall>(libcall_basename#"_PPCF128"),
                           rtname>;
  def NAME#"_f80"
      : RuntimeLibcallImpl<!cast<RuntimeLibcall>(libcall_basename#"_F80"),
                           rtname>;
}

// AArch64 calls
def SC_MEMCPY : RuntimeLibcall;
def SC_MEMMOVE : RuntimeLibcall;
def SC_MEMSET : RuntimeLibcall;
def SC_MEMCHR: RuntimeLibcall;

// AArch64 SME ABI calls
def SMEABI_SME_STATE : RuntimeLibcall;
def SMEABI_TPIDR2_SAVE : RuntimeLibcall;
def SMEABI_ZA_DISABLE : RuntimeLibcall;
def SMEABI_TPIDR2_RESTORE : RuntimeLibcall;
def SMEABI_GET_CURRENT_VG : RuntimeLibcall;
def SMEABI_SME_STATE_SIZE : RuntimeLibcall;
def SMEABI_SME_SAVE : RuntimeLibcall;
def SMEABI_SME_RESTORE : RuntimeLibcall;

// ARM EABI calls
````
- **L513 EN**: Declares TableGen multiclass `LibmLongDoubleLibCall`.
  **L513 CN**: 声明 TableGen multiclass `LibmLongDoubleLibCall`。
- **L514 EN**: Continues the surrounding expression or declaration: `string rtname = NAME> {`.
  **L514 CN**: 继续构造周围的表达式或声明：`string rtname = NAME> {`。
- **L515 EN**: Blank line separating nearby declarations or logic blocks.
  **L515 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L516 EN**: Blank line separating nearby declarations or logic blocks.
  **L516 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L517 EN**: Declares TableGen def `NAME#"_f128"`.
  **L517 CN**: 声明 TableGen def `NAME#"_f128"`。
- **L518 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: RuntimeLibcallImpl<!cast<RuntimeLibcall>(libcall_basename#"_F128"),`.
  **L518 CN**: 继续一个多行参数列表、初始化器或聚合项：`: RuntimeLibcallImpl<!cast<RuntimeLibcall>(libcall_basename#"_F128"),`。
- **L519 EN**: Executes a standalone statement or declaration: `rtname>;`.
  **L519 CN**: 执行一条独立语句或声明：`rtname>;`。
- **L520 EN**: Declares TableGen def `NAME#"_ppcf128"`.
  **L520 CN**: 声明 TableGen def `NAME#"_ppcf128"`。
- **L521 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: RuntimeLibcallImpl<!cast<RuntimeLibcall>(libcall_basename#"_PPCF128"),`.
  **L521 CN**: 继续一个多行参数列表、初始化器或聚合项：`: RuntimeLibcallImpl<!cast<RuntimeLibcall>(libcall_basename#"_PPCF128"),`。
- **L522 EN**: Executes a standalone statement or declaration: `rtname>;`.
  **L522 CN**: 执行一条独立语句或声明：`rtname>;`。
- **L523 EN**: Declares TableGen def `NAME#"_f80"`.
  **L523 CN**: 声明 TableGen def `NAME#"_f80"`。
- **L524 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: RuntimeLibcallImpl<!cast<RuntimeLibcall>(libcall_basename#"_F80"),`.
  **L524 CN**: 继续一个多行参数列表、初始化器或聚合项：`: RuntimeLibcallImpl<!cast<RuntimeLibcall>(libcall_basename#"_F80"),`。
- **L525 EN**: Executes a standalone statement or declaration: `rtname>;`.
  **L525 CN**: 执行一条独立语句或声明：`rtname>;`。
- **L526 EN**: Closes the current lexical scope or compound statement.
  **L526 CN**: 结束当前词法作用域或复合语句块。
- **L527 EN**: Blank line separating nearby declarations or logic blocks.
  **L527 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L528 EN**: Comment explains nearby logic, invariants, or intent: `AArch64 calls`.
  **L528 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`AArch64 calls`。
- **L529 EN**: Declares TableGen def `SC_MEMCPY`.
  **L529 CN**: 声明 TableGen def `SC_MEMCPY`。
- **L530 EN**: Declares TableGen def `SC_MEMMOVE`.
  **L530 CN**: 声明 TableGen def `SC_MEMMOVE`。
- **L531 EN**: Declares TableGen def `SC_MEMSET`.
  **L531 CN**: 声明 TableGen def `SC_MEMSET`。
- **L532 EN**: Declares TableGen def `SC_MEMCHR`.
  **L532 CN**: 声明 TableGen def `SC_MEMCHR`。
- **L533 EN**: Blank line separating nearby declarations or logic blocks.
  **L533 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L534 EN**: Comment explains nearby logic, invariants, or intent: `AArch64 SME ABI calls`.
  **L534 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`AArch64 SME ABI calls`。
- **L535 EN**: Declares TableGen def `SMEABI_SME_STATE`.
  **L535 CN**: 声明 TableGen def `SMEABI_SME_STATE`。
- **L536 EN**: Declares TableGen def `SMEABI_TPIDR2_SAVE`.
  **L536 CN**: 声明 TableGen def `SMEABI_TPIDR2_SAVE`。
- **L537 EN**: Declares TableGen def `SMEABI_ZA_DISABLE`.
  **L537 CN**: 声明 TableGen def `SMEABI_ZA_DISABLE`。
- **L538 EN**: Declares TableGen def `SMEABI_TPIDR2_RESTORE`.
  **L538 CN**: 声明 TableGen def `SMEABI_TPIDR2_RESTORE`。
- **L539 EN**: Declares TableGen def `SMEABI_GET_CURRENT_VG`.
  **L539 CN**: 声明 TableGen def `SMEABI_GET_CURRENT_VG`。
- **L540 EN**: Declares TableGen def `SMEABI_SME_STATE_SIZE`.
  **L540 CN**: 声明 TableGen def `SMEABI_SME_STATE_SIZE`。
- **L541 EN**: Declares TableGen def `SMEABI_SME_SAVE`.
  **L541 CN**: 声明 TableGen def `SMEABI_SME_SAVE`。
- **L542 EN**: Declares TableGen def `SMEABI_SME_RESTORE`.
  **L542 CN**: 声明 TableGen def `SMEABI_SME_RESTORE`。
- **L543 EN**: Blank line separating nearby declarations or logic blocks.
  **L543 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L544 EN**: Comment explains nearby logic, invariants, or intent: `ARM EABI calls`.
  **L544 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`ARM EABI calls`。

### Lines 545-576

````tablegen
def AEABI_MEMCPY4 : RuntimeLibcall; // Align 4
def AEABI_MEMCPY8 : RuntimeLibcall; // Align 8
def AEABI_MEMMOVE4 : RuntimeLibcall;
def AEABI_MEMMOVE8 : RuntimeLibcall;
def AEABI_MEMSET4 : RuntimeLibcall;
def AEABI_MEMSET8 : RuntimeLibcall;
def AEABI_MEMCLR : RuntimeLibcall;
def AEABI_MEMCLR4 : RuntimeLibcall;
def AEABI_MEMCLR8 : RuntimeLibcall;
def AEABI_UREAD4 : RuntimeLibcall;
def AEABI_UREAD8 : RuntimeLibcall;
def AEABI_UWRITE4 : RuntimeLibcall;
def AEABI_UWRITE8 : RuntimeLibcall;

// Hexagon calls
def HEXAGON_MEMCPY_LIKELY_ALIGNED_MIN32BYTES_MULT8BYTES : RuntimeLibcall;
def HEXAGON_VOLATILE_MEMCPY : RuntimeLibcall;

// XCore calls
def MEMCPY_ALIGN_4 : RuntimeLibcall;

// Objective-C calls
def OBJC_AUTORELEASE : RuntimeLibcall;
def OBJC_AUTORELEASEPOOLPOP : RuntimeLibcall;
def OBJC_AUTORELEASEPOOLPUSH : RuntimeLibcall;
def OBJC_AUTORELEASERETURNVALUE : RuntimeLibcall;
def OBJC_COPYWEAK : RuntimeLibcall;
def OBJC_DESTROYWEAK : RuntimeLibcall;
def OBJC_INITWEAK : RuntimeLibcall;
def OBJC_LOADWEAK : RuntimeLibcall;
def OBJC_LOADWEAKRETAINED : RuntimeLibcall;
def OBJC_MOVEWEAK : RuntimeLibcall;
````
- **L545 EN**: Declares TableGen def `AEABI_MEMCPY4`.
  **L545 CN**: 声明 TableGen def `AEABI_MEMCPY4`。
- **L546 EN**: Declares TableGen def `AEABI_MEMCPY8`.
  **L546 CN**: 声明 TableGen def `AEABI_MEMCPY8`。
- **L547 EN**: Declares TableGen def `AEABI_MEMMOVE4`.
  **L547 CN**: 声明 TableGen def `AEABI_MEMMOVE4`。
- **L548 EN**: Declares TableGen def `AEABI_MEMMOVE8`.
  **L548 CN**: 声明 TableGen def `AEABI_MEMMOVE8`。
- **L549 EN**: Declares TableGen def `AEABI_MEMSET4`.
  **L549 CN**: 声明 TableGen def `AEABI_MEMSET4`。
- **L550 EN**: Declares TableGen def `AEABI_MEMSET8`.
  **L550 CN**: 声明 TableGen def `AEABI_MEMSET8`。
- **L551 EN**: Declares TableGen def `AEABI_MEMCLR`.
  **L551 CN**: 声明 TableGen def `AEABI_MEMCLR`。
- **L552 EN**: Declares TableGen def `AEABI_MEMCLR4`.
  **L552 CN**: 声明 TableGen def `AEABI_MEMCLR4`。
- **L553 EN**: Declares TableGen def `AEABI_MEMCLR8`.
  **L553 CN**: 声明 TableGen def `AEABI_MEMCLR8`。
- **L554 EN**: Declares TableGen def `AEABI_UREAD4`.
  **L554 CN**: 声明 TableGen def `AEABI_UREAD4`。
- **L555 EN**: Declares TableGen def `AEABI_UREAD8`.
  **L555 CN**: 声明 TableGen def `AEABI_UREAD8`。
- **L556 EN**: Declares TableGen def `AEABI_UWRITE4`.
  **L556 CN**: 声明 TableGen def `AEABI_UWRITE4`。
- **L557 EN**: Declares TableGen def `AEABI_UWRITE8`.
  **L557 CN**: 声明 TableGen def `AEABI_UWRITE8`。
- **L558 EN**: Blank line separating nearby declarations or logic blocks.
  **L558 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L559 EN**: Comment explains nearby logic, invariants, or intent: `Hexagon calls`.
  **L559 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Hexagon calls`。
- **L560 EN**: Declares TableGen def `HEXAGON_MEMCPY_LIKELY_ALIGNED_MIN32BYTES_MULT8BYTES`.
  **L560 CN**: 声明 TableGen def `HEXAGON_MEMCPY_LIKELY_ALIGNED_MIN32BYTES_MULT8BYTES`。
- **L561 EN**: Declares TableGen def `HEXAGON_VOLATILE_MEMCPY`.
  **L561 CN**: 声明 TableGen def `HEXAGON_VOLATILE_MEMCPY`。
- **L562 EN**: Blank line separating nearby declarations or logic blocks.
  **L562 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L563 EN**: Comment explains nearby logic, invariants, or intent: `XCore calls`.
  **L563 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`XCore calls`。
- **L564 EN**: Declares TableGen def `MEMCPY_ALIGN_4`.
  **L564 CN**: 声明 TableGen def `MEMCPY_ALIGN_4`。
- **L565 EN**: Blank line separating nearby declarations or logic blocks.
  **L565 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L566 EN**: Comment explains nearby logic, invariants, or intent: `Objective-C calls`.
  **L566 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Objective-C calls`。
- **L567 EN**: Declares TableGen def `OBJC_AUTORELEASE`.
  **L567 CN**: 声明 TableGen def `OBJC_AUTORELEASE`。
- **L568 EN**: Declares TableGen def `OBJC_AUTORELEASEPOOLPOP`.
  **L568 CN**: 声明 TableGen def `OBJC_AUTORELEASEPOOLPOP`。
- **L569 EN**: Declares TableGen def `OBJC_AUTORELEASEPOOLPUSH`.
  **L569 CN**: 声明 TableGen def `OBJC_AUTORELEASEPOOLPUSH`。
- **L570 EN**: Declares TableGen def `OBJC_AUTORELEASERETURNVALUE`.
  **L570 CN**: 声明 TableGen def `OBJC_AUTORELEASERETURNVALUE`。
- **L571 EN**: Declares TableGen def `OBJC_COPYWEAK`.
  **L571 CN**: 声明 TableGen def `OBJC_COPYWEAK`。
- **L572 EN**: Declares TableGen def `OBJC_DESTROYWEAK`.
  **L572 CN**: 声明 TableGen def `OBJC_DESTROYWEAK`。
- **L573 EN**: Declares TableGen def `OBJC_INITWEAK`.
  **L573 CN**: 声明 TableGen def `OBJC_INITWEAK`。
- **L574 EN**: Declares TableGen def `OBJC_LOADWEAK`.
  **L574 CN**: 声明 TableGen def `OBJC_LOADWEAK`。
- **L575 EN**: Declares TableGen def `OBJC_LOADWEAKRETAINED`.
  **L575 CN**: 声明 TableGen def `OBJC_LOADWEAKRETAINED`。
- **L576 EN**: Declares TableGen def `OBJC_MOVEWEAK`.
  **L576 CN**: 声明 TableGen def `OBJC_MOVEWEAK`。

### Lines 577-608

````tablegen
def OBJC_RELEASE : RuntimeLibcall;
def OBJC_RETAIN : RuntimeLibcall;
def OBJC_RETAINAUTORELEASE : RuntimeLibcall;
def OBJC_RETAINAUTORELEASERETURNVALUE : RuntimeLibcall;
def OBJC_RETAINAUTORELEASEDRETURNVALUE : RuntimeLibcall;
def OBJC_CLAIMAUTORELEASEDRETURNVALUE : RuntimeLibcall;
def OBJC_RETAINBLOCK : RuntimeLibcall;
def OBJC_STORESTRONG : RuntimeLibcall;
def OBJC_STOREWEAK : RuntimeLibcall;
def OBJC_UNSAFECLAIMAUTORELEASEDRETURNVALUE : RuntimeLibcall;
def OBJC_RETAINEDOBJECT : RuntimeLibcall;
def OBJC_UNRETAINEDOBJECT : RuntimeLibcall;
def OBJC_UNRETAINEDPOINTER : RuntimeLibcall;
def OBJC_RETAIN_AUTORELEASE : RuntimeLibcall;
def OBJC_SYNC_ENTER : RuntimeLibcall;
def OBJC_SYNC_EXIT : RuntimeLibcall;

def ABORT : RuntimeLibcall;
def ABS : RuntimeLibcall;
def ACCESS : RuntimeLibcall;
def ALIGNED_ALLOC : RuntimeLibcall;
def ATEXIT : RuntimeLibcall;
def ATOF : RuntimeLibcall;
def ATOI : RuntimeLibcall;
def ATOL : RuntimeLibcall;
def ATOLL : RuntimeLibcall;
def BCMP : RuntimeLibcall;
def BCOPY : RuntimeLibcall;
def CHMOD : RuntimeLibcall;
def CHOWN : RuntimeLibcall;
def CLEARERR : RuntimeLibcall;
def CLOSEDIR : RuntimeLibcall;
````
- **L577 EN**: Declares TableGen def `OBJC_RELEASE`.
  **L577 CN**: 声明 TableGen def `OBJC_RELEASE`。
- **L578 EN**: Declares TableGen def `OBJC_RETAIN`.
  **L578 CN**: 声明 TableGen def `OBJC_RETAIN`。
- **L579 EN**: Declares TableGen def `OBJC_RETAINAUTORELEASE`.
  **L579 CN**: 声明 TableGen def `OBJC_RETAINAUTORELEASE`。
- **L580 EN**: Declares TableGen def `OBJC_RETAINAUTORELEASERETURNVALUE`.
  **L580 CN**: 声明 TableGen def `OBJC_RETAINAUTORELEASERETURNVALUE`。
- **L581 EN**: Declares TableGen def `OBJC_RETAINAUTORELEASEDRETURNVALUE`.
  **L581 CN**: 声明 TableGen def `OBJC_RETAINAUTORELEASEDRETURNVALUE`。
- **L582 EN**: Declares TableGen def `OBJC_CLAIMAUTORELEASEDRETURNVALUE`.
  **L582 CN**: 声明 TableGen def `OBJC_CLAIMAUTORELEASEDRETURNVALUE`。
- **L583 EN**: Declares TableGen def `OBJC_RETAINBLOCK`.
  **L583 CN**: 声明 TableGen def `OBJC_RETAINBLOCK`。
- **L584 EN**: Declares TableGen def `OBJC_STORESTRONG`.
  **L584 CN**: 声明 TableGen def `OBJC_STORESTRONG`。
- **L585 EN**: Declares TableGen def `OBJC_STOREWEAK`.
  **L585 CN**: 声明 TableGen def `OBJC_STOREWEAK`。
- **L586 EN**: Declares TableGen def `OBJC_UNSAFECLAIMAUTORELEASEDRETURNVALUE`.
  **L586 CN**: 声明 TableGen def `OBJC_UNSAFECLAIMAUTORELEASEDRETURNVALUE`。
- **L587 EN**: Declares TableGen def `OBJC_RETAINEDOBJECT`.
  **L587 CN**: 声明 TableGen def `OBJC_RETAINEDOBJECT`。
- **L588 EN**: Declares TableGen def `OBJC_UNRETAINEDOBJECT`.
  **L588 CN**: 声明 TableGen def `OBJC_UNRETAINEDOBJECT`。
- **L589 EN**: Declares TableGen def `OBJC_UNRETAINEDPOINTER`.
  **L589 CN**: 声明 TableGen def `OBJC_UNRETAINEDPOINTER`。
- **L590 EN**: Declares TableGen def `OBJC_RETAIN_AUTORELEASE`.
  **L590 CN**: 声明 TableGen def `OBJC_RETAIN_AUTORELEASE`。
- **L591 EN**: Declares TableGen def `OBJC_SYNC_ENTER`.
  **L591 CN**: 声明 TableGen def `OBJC_SYNC_ENTER`。
- **L592 EN**: Declares TableGen def `OBJC_SYNC_EXIT`.
  **L592 CN**: 声明 TableGen def `OBJC_SYNC_EXIT`。
- **L593 EN**: Blank line separating nearby declarations or logic blocks.
  **L593 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L594 EN**: Declares TableGen def `ABORT`.
  **L594 CN**: 声明 TableGen def `ABORT`。
- **L595 EN**: Declares TableGen def `ABS`.
  **L595 CN**: 声明 TableGen def `ABS`。
- **L596 EN**: Declares TableGen def `ACCESS`.
  **L596 CN**: 声明 TableGen def `ACCESS`。
- **L597 EN**: Declares TableGen def `ALIGNED_ALLOC`.
  **L597 CN**: 声明 TableGen def `ALIGNED_ALLOC`。
- **L598 EN**: Declares TableGen def `ATEXIT`.
  **L598 CN**: 声明 TableGen def `ATEXIT`。
- **L599 EN**: Declares TableGen def `ATOF`.
  **L599 CN**: 声明 TableGen def `ATOF`。
- **L600 EN**: Declares TableGen def `ATOI`.
  **L600 CN**: 声明 TableGen def `ATOI`。
- **L601 EN**: Declares TableGen def `ATOL`.
  **L601 CN**: 声明 TableGen def `ATOL`。
- **L602 EN**: Declares TableGen def `ATOLL`.
  **L602 CN**: 声明 TableGen def `ATOLL`。
- **L603 EN**: Declares TableGen def `BCMP`.
  **L603 CN**: 声明 TableGen def `BCMP`。
- **L604 EN**: Declares TableGen def `BCOPY`.
  **L604 CN**: 声明 TableGen def `BCOPY`。
- **L605 EN**: Declares TableGen def `CHMOD`.
  **L605 CN**: 声明 TableGen def `CHMOD`。
- **L606 EN**: Declares TableGen def `CHOWN`.
  **L606 CN**: 声明 TableGen def `CHOWN`。
- **L607 EN**: Declares TableGen def `CLEARERR`.
  **L607 CN**: 声明 TableGen def `CLEARERR`。
- **L608 EN**: Declares TableGen def `CLOSEDIR`.
  **L608 CN**: 声明 TableGen def `CLOSEDIR`。

### Lines 609-640

````tablegen
def CTERMID : RuntimeLibcall;
def CXA_ATEXIT : RuntimeLibcall;
def CXA_GUARD_ABORT : RuntimeLibcall;
def CXA_GUARD_ACQUIRE : RuntimeLibcall;
def CXA_GUARD_RELEASE : RuntimeLibcall;
def CXA_THROW : RuntimeLibcall;
def DUNDER_ISOC99_SCANF : RuntimeLibcall;
def DUNDER_ISOC99_SSCANF : RuntimeLibcall;
def DUNDER_STRDUP : RuntimeLibcall;
def DUNDER_STRNDUP : RuntimeLibcall;
def DUNDER_STRTOK_R : RuntimeLibcall;
def ENUM_VARIANT : RuntimeLibcall;
def EXECL : RuntimeLibcall;
def EXECLE : RuntimeLibcall;
def EXECLP : RuntimeLibcall;
def EXECV : RuntimeLibcall;
def EXECVE : RuntimeLibcall;
def EXECVP : RuntimeLibcall;
def EXECVPE : RuntimeLibcall;
def EXIT : RuntimeLibcall;
def FCLOSE : RuntimeLibcall;
def FDOPEN : RuntimeLibcall;
def FEOF : RuntimeLibcall;
def FERROR : RuntimeLibcall;
def FFLUSH : RuntimeLibcall;
def FFS : RuntimeLibcall;
def FFSL : RuntimeLibcall;
def FFSLL : RuntimeLibcall;
def FGETC : RuntimeLibcall;
def FGETC_UNLOCKED : RuntimeLibcall;
def FGETPOS : RuntimeLibcall;
def FGETS : RuntimeLibcall;
````
- **L609 EN**: Declares TableGen def `CTERMID`.
  **L609 CN**: 声明 TableGen def `CTERMID`。
- **L610 EN**: Declares TableGen def `CXA_ATEXIT`.
  **L610 CN**: 声明 TableGen def `CXA_ATEXIT`。
- **L611 EN**: Declares TableGen def `CXA_GUARD_ABORT`.
  **L611 CN**: 声明 TableGen def `CXA_GUARD_ABORT`。
- **L612 EN**: Declares TableGen def `CXA_GUARD_ACQUIRE`.
  **L612 CN**: 声明 TableGen def `CXA_GUARD_ACQUIRE`。
- **L613 EN**: Declares TableGen def `CXA_GUARD_RELEASE`.
  **L613 CN**: 声明 TableGen def `CXA_GUARD_RELEASE`。
- **L614 EN**: Declares TableGen def `CXA_THROW`.
  **L614 CN**: 声明 TableGen def `CXA_THROW`。
- **L615 EN**: Declares TableGen def `DUNDER_ISOC99_SCANF`.
  **L615 CN**: 声明 TableGen def `DUNDER_ISOC99_SCANF`。
- **L616 EN**: Declares TableGen def `DUNDER_ISOC99_SSCANF`.
  **L616 CN**: 声明 TableGen def `DUNDER_ISOC99_SSCANF`。
- **L617 EN**: Declares TableGen def `DUNDER_STRDUP`.
  **L617 CN**: 声明 TableGen def `DUNDER_STRDUP`。
- **L618 EN**: Declares TableGen def `DUNDER_STRNDUP`.
  **L618 CN**: 声明 TableGen def `DUNDER_STRNDUP`。
- **L619 EN**: Declares TableGen def `DUNDER_STRTOK_R`.
  **L619 CN**: 声明 TableGen def `DUNDER_STRTOK_R`。
- **L620 EN**: Declares TableGen def `ENUM_VARIANT`.
  **L620 CN**: 声明 TableGen def `ENUM_VARIANT`。
- **L621 EN**: Declares TableGen def `EXECL`.
  **L621 CN**: 声明 TableGen def `EXECL`。
- **L622 EN**: Declares TableGen def `EXECLE`.
  **L622 CN**: 声明 TableGen def `EXECLE`。
- **L623 EN**: Declares TableGen def `EXECLP`.
  **L623 CN**: 声明 TableGen def `EXECLP`。
- **L624 EN**: Declares TableGen def `EXECV`.
  **L624 CN**: 声明 TableGen def `EXECV`。
- **L625 EN**: Declares TableGen def `EXECVE`.
  **L625 CN**: 声明 TableGen def `EXECVE`。
- **L626 EN**: Declares TableGen def `EXECVP`.
  **L626 CN**: 声明 TableGen def `EXECVP`。
- **L627 EN**: Declares TableGen def `EXECVPE`.
  **L627 CN**: 声明 TableGen def `EXECVPE`。
- **L628 EN**: Declares TableGen def `EXIT`.
  **L628 CN**: 声明 TableGen def `EXIT`。
- **L629 EN**: Declares TableGen def `FCLOSE`.
  **L629 CN**: 声明 TableGen def `FCLOSE`。
- **L630 EN**: Declares TableGen def `FDOPEN`.
  **L630 CN**: 声明 TableGen def `FDOPEN`。
- **L631 EN**: Declares TableGen def `FEOF`.
  **L631 CN**: 声明 TableGen def `FEOF`。
- **L632 EN**: Declares TableGen def `FERROR`.
  **L632 CN**: 声明 TableGen def `FERROR`。
- **L633 EN**: Declares TableGen def `FFLUSH`.
  **L633 CN**: 声明 TableGen def `FFLUSH`。
- **L634 EN**: Declares TableGen def `FFS`.
  **L634 CN**: 声明 TableGen def `FFS`。
- **L635 EN**: Declares TableGen def `FFSL`.
  **L635 CN**: 声明 TableGen def `FFSL`。
- **L636 EN**: Declares TableGen def `FFSLL`.
  **L636 CN**: 声明 TableGen def `FFSLL`。
- **L637 EN**: Declares TableGen def `FGETC`.
  **L637 CN**: 声明 TableGen def `FGETC`。
- **L638 EN**: Declares TableGen def `FGETC_UNLOCKED`.
  **L638 CN**: 声明 TableGen def `FGETC_UNLOCKED`。
- **L639 EN**: Declares TableGen def `FGETPOS`.
  **L639 CN**: 声明 TableGen def `FGETPOS`。
- **L640 EN**: Declares TableGen def `FGETS`.
  **L640 CN**: 声明 TableGen def `FGETS`。

### Lines 641-672

````tablegen
def FGETS_UNLOCKED : RuntimeLibcall;
def FILENO : RuntimeLibcall;
def FIPRINTF : RuntimeLibcall;
def FLOCKFILE : RuntimeLibcall;
def FLS : RuntimeLibcall;
def FLSL : RuntimeLibcall;
def FLSLL : RuntimeLibcall;
def FOPEN : RuntimeLibcall;
def FOPEN64 : RuntimeLibcall;
def FORK : RuntimeLibcall;
def FPRINTF : RuntimeLibcall;
def FPUTC : RuntimeLibcall;
def FPUTC_UNLOCKED : RuntimeLibcall;
def FPUTS : RuntimeLibcall;
def FPUTS_UNLOCKED : RuntimeLibcall;
def FREAD : RuntimeLibcall;
def FREAD_UNLOCKED : RuntimeLibcall;
def FSCANF : RuntimeLibcall;
def FSEEK : RuntimeLibcall;
def FSEEKO : RuntimeLibcall;
def FSEEKO64 : RuntimeLibcall;
def FSETPOS : RuntimeLibcall;
def FSTAT : RuntimeLibcall;
def FSTAT64 : RuntimeLibcall;
def FSTATVFS : RuntimeLibcall;
def FSTATVFS64 : RuntimeLibcall;
def FTELL : RuntimeLibcall;
def FTELLO : RuntimeLibcall;
def FTELLO64 : RuntimeLibcall;
def FTRYLOCKFILE : RuntimeLibcall;
def FUNLOCKFILE : RuntimeLibcall;
def FWRITE : RuntimeLibcall;
````
- **L641 EN**: Declares TableGen def `FGETS_UNLOCKED`.
  **L641 CN**: 声明 TableGen def `FGETS_UNLOCKED`。
- **L642 EN**: Declares TableGen def `FILENO`.
  **L642 CN**: 声明 TableGen def `FILENO`。
- **L643 EN**: Declares TableGen def `FIPRINTF`.
  **L643 CN**: 声明 TableGen def `FIPRINTF`。
- **L644 EN**: Declares TableGen def `FLOCKFILE`.
  **L644 CN**: 声明 TableGen def `FLOCKFILE`。
- **L645 EN**: Declares TableGen def `FLS`.
  **L645 CN**: 声明 TableGen def `FLS`。
- **L646 EN**: Declares TableGen def `FLSL`.
  **L646 CN**: 声明 TableGen def `FLSL`。
- **L647 EN**: Declares TableGen def `FLSLL`.
  **L647 CN**: 声明 TableGen def `FLSLL`。
- **L648 EN**: Declares TableGen def `FOPEN`.
  **L648 CN**: 声明 TableGen def `FOPEN`。
- **L649 EN**: Declares TableGen def `FOPEN64`.
  **L649 CN**: 声明 TableGen def `FOPEN64`。
- **L650 EN**: Declares TableGen def `FORK`.
  **L650 CN**: 声明 TableGen def `FORK`。
- **L651 EN**: Declares TableGen def `FPRINTF`.
  **L651 CN**: 声明 TableGen def `FPRINTF`。
- **L652 EN**: Declares TableGen def `FPUTC`.
  **L652 CN**: 声明 TableGen def `FPUTC`。
- **L653 EN**: Declares TableGen def `FPUTC_UNLOCKED`.
  **L653 CN**: 声明 TableGen def `FPUTC_UNLOCKED`。
- **L654 EN**: Declares TableGen def `FPUTS`.
  **L654 CN**: 声明 TableGen def `FPUTS`。
- **L655 EN**: Declares TableGen def `FPUTS_UNLOCKED`.
  **L655 CN**: 声明 TableGen def `FPUTS_UNLOCKED`。
- **L656 EN**: Declares TableGen def `FREAD`.
  **L656 CN**: 声明 TableGen def `FREAD`。
- **L657 EN**: Declares TableGen def `FREAD_UNLOCKED`.
  **L657 CN**: 声明 TableGen def `FREAD_UNLOCKED`。
- **L658 EN**: Declares TableGen def `FSCANF`.
  **L658 CN**: 声明 TableGen def `FSCANF`。
- **L659 EN**: Declares TableGen def `FSEEK`.
  **L659 CN**: 声明 TableGen def `FSEEK`。
- **L660 EN**: Declares TableGen def `FSEEKO`.
  **L660 CN**: 声明 TableGen def `FSEEKO`。
- **L661 EN**: Declares TableGen def `FSEEKO64`.
  **L661 CN**: 声明 TableGen def `FSEEKO64`。
- **L662 EN**: Declares TableGen def `FSETPOS`.
  **L662 CN**: 声明 TableGen def `FSETPOS`。
- **L663 EN**: Declares TableGen def `FSTAT`.
  **L663 CN**: 声明 TableGen def `FSTAT`。
- **L664 EN**: Declares TableGen def `FSTAT64`.
  **L664 CN**: 声明 TableGen def `FSTAT64`。
- **L665 EN**: Declares TableGen def `FSTATVFS`.
  **L665 CN**: 声明 TableGen def `FSTATVFS`。
- **L666 EN**: Declares TableGen def `FSTATVFS64`.
  **L666 CN**: 声明 TableGen def `FSTATVFS64`。
- **L667 EN**: Declares TableGen def `FTELL`.
  **L667 CN**: 声明 TableGen def `FTELL`。
- **L668 EN**: Declares TableGen def `FTELLO`.
  **L668 CN**: 声明 TableGen def `FTELLO`。
- **L669 EN**: Declares TableGen def `FTELLO64`.
  **L669 CN**: 声明 TableGen def `FTELLO64`。
- **L670 EN**: Declares TableGen def `FTRYLOCKFILE`.
  **L670 CN**: 声明 TableGen def `FTRYLOCKFILE`。
- **L671 EN**: Declares TableGen def `FUNLOCKFILE`.
  **L671 CN**: 声明 TableGen def `FUNLOCKFILE`。
- **L672 EN**: Declares TableGen def `FWRITE`.
  **L672 CN**: 声明 TableGen def `FWRITE`。

### Lines 673-704

````tablegen
def FWRITE_UNLOCKED : RuntimeLibcall;
def GETC : RuntimeLibcall;
def GETCHAR : RuntimeLibcall;
def GETCHAR_UNLOCKED : RuntimeLibcall;
def GETC_UNLOCKED : RuntimeLibcall;
def GETENV : RuntimeLibcall;
def GETITIMER : RuntimeLibcall;
def GETLOGIN_R : RuntimeLibcall;
def GETPWNAM : RuntimeLibcall;
def GETS : RuntimeLibcall;
def GETTIMEOFDAY : RuntimeLibcall;
def HTONL : RuntimeLibcall;
def HTONS : RuntimeLibcall;
def IPRINTF : RuntimeLibcall;
def ISASCII : RuntimeLibcall;
def ISDIGIT : RuntimeLibcall;
def LABS : RuntimeLibcall;
def LCHOWN : RuntimeLibcall;
def LLABS : RuntimeLibcall;
def LSTAT : RuntimeLibcall;
def LSTAT64 : RuntimeLibcall;
def MEMALIGN : RuntimeLibcall;
def MEMCCPY : RuntimeLibcall;
def MEMCCPY_CHK : RuntimeLibcall;
def MEMCHR : RuntimeLibcall;
def MEMPCPY : RuntimeLibcall;
def MEMPCPY_CHK : RuntimeLibcall;
def MEMRCHR : RuntimeLibcall;
def MEMSET_PATTERN16 : RuntimeLibcall;
def MEMSET_PATTERN4 : RuntimeLibcall;
def MEMSET_PATTERN8 : RuntimeLibcall;
def MKDIR : RuntimeLibcall;
````
- **L673 EN**: Declares TableGen def `FWRITE_UNLOCKED`.
  **L673 CN**: 声明 TableGen def `FWRITE_UNLOCKED`。
- **L674 EN**: Declares TableGen def `GETC`.
  **L674 CN**: 声明 TableGen def `GETC`。
- **L675 EN**: Declares TableGen def `GETCHAR`.
  **L675 CN**: 声明 TableGen def `GETCHAR`。
- **L676 EN**: Declares TableGen def `GETCHAR_UNLOCKED`.
  **L676 CN**: 声明 TableGen def `GETCHAR_UNLOCKED`。
- **L677 EN**: Declares TableGen def `GETC_UNLOCKED`.
  **L677 CN**: 声明 TableGen def `GETC_UNLOCKED`。
- **L678 EN**: Declares TableGen def `GETENV`.
  **L678 CN**: 声明 TableGen def `GETENV`。
- **L679 EN**: Declares TableGen def `GETITIMER`.
  **L679 CN**: 声明 TableGen def `GETITIMER`。
- **L680 EN**: Declares TableGen def `GETLOGIN_R`.
  **L680 CN**: 声明 TableGen def `GETLOGIN_R`。
- **L681 EN**: Declares TableGen def `GETPWNAM`.
  **L681 CN**: 声明 TableGen def `GETPWNAM`。
- **L682 EN**: Declares TableGen def `GETS`.
  **L682 CN**: 声明 TableGen def `GETS`。
- **L683 EN**: Declares TableGen def `GETTIMEOFDAY`.
  **L683 CN**: 声明 TableGen def `GETTIMEOFDAY`。
- **L684 EN**: Declares TableGen def `HTONL`.
  **L684 CN**: 声明 TableGen def `HTONL`。
- **L685 EN**: Declares TableGen def `HTONS`.
  **L685 CN**: 声明 TableGen def `HTONS`。
- **L686 EN**: Declares TableGen def `IPRINTF`.
  **L686 CN**: 声明 TableGen def `IPRINTF`。
- **L687 EN**: Declares TableGen def `ISASCII`.
  **L687 CN**: 声明 TableGen def `ISASCII`。
- **L688 EN**: Declares TableGen def `ISDIGIT`.
  **L688 CN**: 声明 TableGen def `ISDIGIT`。
- **L689 EN**: Declares TableGen def `LABS`.
  **L689 CN**: 声明 TableGen def `LABS`。
- **L690 EN**: Declares TableGen def `LCHOWN`.
  **L690 CN**: 声明 TableGen def `LCHOWN`。
- **L691 EN**: Declares TableGen def `LLABS`.
  **L691 CN**: 声明 TableGen def `LLABS`。
- **L692 EN**: Declares TableGen def `LSTAT`.
  **L692 CN**: 声明 TableGen def `LSTAT`。
- **L693 EN**: Declares TableGen def `LSTAT64`.
  **L693 CN**: 声明 TableGen def `LSTAT64`。
- **L694 EN**: Declares TableGen def `MEMALIGN`.
  **L694 CN**: 声明 TableGen def `MEMALIGN`。
- **L695 EN**: Declares TableGen def `MEMCCPY`.
  **L695 CN**: 声明 TableGen def `MEMCCPY`。
- **L696 EN**: Declares TableGen def `MEMCCPY_CHK`.
  **L696 CN**: 声明 TableGen def `MEMCCPY_CHK`。
- **L697 EN**: Declares TableGen def `MEMCHR`.
  **L697 CN**: 声明 TableGen def `MEMCHR`。
- **L698 EN**: Declares TableGen def `MEMPCPY`.
  **L698 CN**: 声明 TableGen def `MEMPCPY`。
- **L699 EN**: Declares TableGen def `MEMPCPY_CHK`.
  **L699 CN**: 声明 TableGen def `MEMPCPY_CHK`。
- **L700 EN**: Declares TableGen def `MEMRCHR`.
  **L700 CN**: 声明 TableGen def `MEMRCHR`。
- **L701 EN**: Declares TableGen def `MEMSET_PATTERN16`.
  **L701 CN**: 声明 TableGen def `MEMSET_PATTERN16`。
- **L702 EN**: Declares TableGen def `MEMSET_PATTERN4`.
  **L702 CN**: 声明 TableGen def `MEMSET_PATTERN4`。
- **L703 EN**: Declares TableGen def `MEMSET_PATTERN8`.
  **L703 CN**: 声明 TableGen def `MEMSET_PATTERN8`。
- **L704 EN**: Declares TableGen def `MKDIR`.
  **L704 CN**: 声明 TableGen def `MKDIR`。

### Lines 705-736

````tablegen
def MKTIME : RuntimeLibcall;
def MSVC_DELETE_ARRAY_PTR32 : RuntimeLibcall;
def MSVC_DELETE_ARRAY_PTR32_INT : RuntimeLibcall;
def MSVC_DELETE_ARRAY_PTR32_NOTHROW : RuntimeLibcall;
def MSVC_DELETE_ARRAY_PTR64 : RuntimeLibcall;
def MSVC_DELETE_ARRAY_PTR64_LONGLONG : RuntimeLibcall;
def MSVC_DELETE_ARRAY_PTR64_NOTHROW : RuntimeLibcall;
def MSVC_DELETE_PTR32 : RuntimeLibcall;
def MSVC_DELETE_PTR32_INT : RuntimeLibcall;
def MSVC_DELETE_PTR32_NOTHROW : RuntimeLibcall;
def MSVC_DELETE_PTR64 : RuntimeLibcall;
def MSVC_DELETE_PTR64_LONGLONG : RuntimeLibcall;
def MSVC_DELETE_PTR64_NOTHROW : RuntimeLibcall;
def MSVC_NEW_ARRAY_INT : RuntimeLibcall;
def MSVC_NEW_ARRAY_INT_NOTHROW : RuntimeLibcall;
def MSVC_NEW_ARRAY_LONGLONG : RuntimeLibcall;
def MSVC_NEW_ARRAY_LONGLONG_NOTHROW : RuntimeLibcall;
def MSVC_NEW_INT : RuntimeLibcall;
def MSVC_NEW_INT_NOTHROW : RuntimeLibcall;
def MSVC_NEW_LONGLONG : RuntimeLibcall;
def MSVC_NEW_LONGLONG_NOTHROW : RuntimeLibcall;
def NTOHL : RuntimeLibcall;
def NTOHS : RuntimeLibcall;
def OPEN : RuntimeLibcall;
def OPEN64 : RuntimeLibcall;
def OPENDIR : RuntimeLibcall;
def PCLOSE : RuntimeLibcall;
def PERROR : RuntimeLibcall;
def POPEN : RuntimeLibcall;
def POSIX_MEMALIGN : RuntimeLibcall;
def PREAD : RuntimeLibcall;
def PRINTF : RuntimeLibcall;
````
- **L705 EN**: Declares TableGen def `MKTIME`.
  **L705 CN**: 声明 TableGen def `MKTIME`。
- **L706 EN**: Declares TableGen def `MSVC_DELETE_ARRAY_PTR32`.
  **L706 CN**: 声明 TableGen def `MSVC_DELETE_ARRAY_PTR32`。
- **L707 EN**: Declares TableGen def `MSVC_DELETE_ARRAY_PTR32_INT`.
  **L707 CN**: 声明 TableGen def `MSVC_DELETE_ARRAY_PTR32_INT`。
- **L708 EN**: Declares TableGen def `MSVC_DELETE_ARRAY_PTR32_NOTHROW`.
  **L708 CN**: 声明 TableGen def `MSVC_DELETE_ARRAY_PTR32_NOTHROW`。
- **L709 EN**: Declares TableGen def `MSVC_DELETE_ARRAY_PTR64`.
  **L709 CN**: 声明 TableGen def `MSVC_DELETE_ARRAY_PTR64`。
- **L710 EN**: Declares TableGen def `MSVC_DELETE_ARRAY_PTR64_LONGLONG`.
  **L710 CN**: 声明 TableGen def `MSVC_DELETE_ARRAY_PTR64_LONGLONG`。
- **L711 EN**: Declares TableGen def `MSVC_DELETE_ARRAY_PTR64_NOTHROW`.
  **L711 CN**: 声明 TableGen def `MSVC_DELETE_ARRAY_PTR64_NOTHROW`。
- **L712 EN**: Declares TableGen def `MSVC_DELETE_PTR32`.
  **L712 CN**: 声明 TableGen def `MSVC_DELETE_PTR32`。
- **L713 EN**: Declares TableGen def `MSVC_DELETE_PTR32_INT`.
  **L713 CN**: 声明 TableGen def `MSVC_DELETE_PTR32_INT`。
- **L714 EN**: Declares TableGen def `MSVC_DELETE_PTR32_NOTHROW`.
  **L714 CN**: 声明 TableGen def `MSVC_DELETE_PTR32_NOTHROW`。
- **L715 EN**: Declares TableGen def `MSVC_DELETE_PTR64`.
  **L715 CN**: 声明 TableGen def `MSVC_DELETE_PTR64`。
- **L716 EN**: Declares TableGen def `MSVC_DELETE_PTR64_LONGLONG`.
  **L716 CN**: 声明 TableGen def `MSVC_DELETE_PTR64_LONGLONG`。
- **L717 EN**: Declares TableGen def `MSVC_DELETE_PTR64_NOTHROW`.
  **L717 CN**: 声明 TableGen def `MSVC_DELETE_PTR64_NOTHROW`。
- **L718 EN**: Declares TableGen def `MSVC_NEW_ARRAY_INT`.
  **L718 CN**: 声明 TableGen def `MSVC_NEW_ARRAY_INT`。
- **L719 EN**: Declares TableGen def `MSVC_NEW_ARRAY_INT_NOTHROW`.
  **L719 CN**: 声明 TableGen def `MSVC_NEW_ARRAY_INT_NOTHROW`。
- **L720 EN**: Declares TableGen def `MSVC_NEW_ARRAY_LONGLONG`.
  **L720 CN**: 声明 TableGen def `MSVC_NEW_ARRAY_LONGLONG`。
- **L721 EN**: Declares TableGen def `MSVC_NEW_ARRAY_LONGLONG_NOTHROW`.
  **L721 CN**: 声明 TableGen def `MSVC_NEW_ARRAY_LONGLONG_NOTHROW`。
- **L722 EN**: Declares TableGen def `MSVC_NEW_INT`.
  **L722 CN**: 声明 TableGen def `MSVC_NEW_INT`。
- **L723 EN**: Declares TableGen def `MSVC_NEW_INT_NOTHROW`.
  **L723 CN**: 声明 TableGen def `MSVC_NEW_INT_NOTHROW`。
- **L724 EN**: Declares TableGen def `MSVC_NEW_LONGLONG`.
  **L724 CN**: 声明 TableGen def `MSVC_NEW_LONGLONG`。
- **L725 EN**: Declares TableGen def `MSVC_NEW_LONGLONG_NOTHROW`.
  **L725 CN**: 声明 TableGen def `MSVC_NEW_LONGLONG_NOTHROW`。
- **L726 EN**: Declares TableGen def `NTOHL`.
  **L726 CN**: 声明 TableGen def `NTOHL`。
- **L727 EN**: Declares TableGen def `NTOHS`.
  **L727 CN**: 声明 TableGen def `NTOHS`。
- **L728 EN**: Declares TableGen def `OPEN`.
  **L728 CN**: 声明 TableGen def `OPEN`。
- **L729 EN**: Declares TableGen def `OPEN64`.
  **L729 CN**: 声明 TableGen def `OPEN64`。
- **L730 EN**: Declares TableGen def `OPENDIR`.
  **L730 CN**: 声明 TableGen def `OPENDIR`。
- **L731 EN**: Declares TableGen def `PCLOSE`.
  **L731 CN**: 声明 TableGen def `PCLOSE`。
- **L732 EN**: Declares TableGen def `PERROR`.
  **L732 CN**: 声明 TableGen def `PERROR`。
- **L733 EN**: Declares TableGen def `POPEN`.
  **L733 CN**: 声明 TableGen def `POPEN`。
- **L734 EN**: Declares TableGen def `POSIX_MEMALIGN`.
  **L734 CN**: 声明 TableGen def `POSIX_MEMALIGN`。
- **L735 EN**: Declares TableGen def `PREAD`.
  **L735 CN**: 声明 TableGen def `PREAD`。
- **L736 EN**: Declares TableGen def `PRINTF`.
  **L736 CN**: 声明 TableGen def `PRINTF`。

### Lines 737-768

````tablegen
def PUTC : RuntimeLibcall;
def PUTCHAR : RuntimeLibcall;
def PUTCHAR_UNLOCKED : RuntimeLibcall;
def PUTC_UNLOCKED : RuntimeLibcall;
def PUTS : RuntimeLibcall;
def PVALLOC : RuntimeLibcall;
def PWRITE : RuntimeLibcall;
def QSORT : RuntimeLibcall;
def READ : RuntimeLibcall;
def READLINK : RuntimeLibcall;
def REALLOC : RuntimeLibcall;
def REALLOCARRAY : RuntimeLibcall;
def REALLOCF : RuntimeLibcall;
def REALPATH : RuntimeLibcall;
def REMOVE : RuntimeLibcall;
def RENAME : RuntimeLibcall;
def REWIND : RuntimeLibcall;
def RMDIR : RuntimeLibcall;
def SCANF : RuntimeLibcall;
def SETBUF : RuntimeLibcall;
def SETITIMER : RuntimeLibcall;
def SETVBUF : RuntimeLibcall;
def SIPRINTF : RuntimeLibcall;
def SIZE_RETURNING_NEW : RuntimeLibcall;
def SIZE_RETURNING_NEW_ALIGNED : RuntimeLibcall;
def SIZE_RETURNING_NEW_ALIGNED_HOT_COLD : RuntimeLibcall;
def SIZE_RETURNING_NEW_HOT_COLD : RuntimeLibcall;
def SMALL_FPRINTF : RuntimeLibcall;
def SMALL_PRINTF : RuntimeLibcall;
def SMALL_SPRINTF : RuntimeLibcall;
def SNPRINTF : RuntimeLibcall;
def SNPRINTF_CHK : RuntimeLibcall;
````
- **L737 EN**: Declares TableGen def `PUTC`.
  **L737 CN**: 声明 TableGen def `PUTC`。
- **L738 EN**: Declares TableGen def `PUTCHAR`.
  **L738 CN**: 声明 TableGen def `PUTCHAR`。
- **L739 EN**: Declares TableGen def `PUTCHAR_UNLOCKED`.
  **L739 CN**: 声明 TableGen def `PUTCHAR_UNLOCKED`。
- **L740 EN**: Declares TableGen def `PUTC_UNLOCKED`.
  **L740 CN**: 声明 TableGen def `PUTC_UNLOCKED`。
- **L741 EN**: Declares TableGen def `PUTS`.
  **L741 CN**: 声明 TableGen def `PUTS`。
- **L742 EN**: Declares TableGen def `PVALLOC`.
  **L742 CN**: 声明 TableGen def `PVALLOC`。
- **L743 EN**: Declares TableGen def `PWRITE`.
  **L743 CN**: 声明 TableGen def `PWRITE`。
- **L744 EN**: Declares TableGen def `QSORT`.
  **L744 CN**: 声明 TableGen def `QSORT`。
- **L745 EN**: Declares TableGen def `READ`.
  **L745 CN**: 声明 TableGen def `READ`。
- **L746 EN**: Declares TableGen def `READLINK`.
  **L746 CN**: 声明 TableGen def `READLINK`。
- **L747 EN**: Declares TableGen def `REALLOC`.
  **L747 CN**: 声明 TableGen def `REALLOC`。
- **L748 EN**: Declares TableGen def `REALLOCARRAY`.
  **L748 CN**: 声明 TableGen def `REALLOCARRAY`。
- **L749 EN**: Declares TableGen def `REALLOCF`.
  **L749 CN**: 声明 TableGen def `REALLOCF`。
- **L750 EN**: Declares TableGen def `REALPATH`.
  **L750 CN**: 声明 TableGen def `REALPATH`。
- **L751 EN**: Declares TableGen def `REMOVE`.
  **L751 CN**: 声明 TableGen def `REMOVE`。
- **L752 EN**: Declares TableGen def `RENAME`.
  **L752 CN**: 声明 TableGen def `RENAME`。
- **L753 EN**: Declares TableGen def `REWIND`.
  **L753 CN**: 声明 TableGen def `REWIND`。
- **L754 EN**: Declares TableGen def `RMDIR`.
  **L754 CN**: 声明 TableGen def `RMDIR`。
- **L755 EN**: Declares TableGen def `SCANF`.
  **L755 CN**: 声明 TableGen def `SCANF`。
- **L756 EN**: Declares TableGen def `SETBUF`.
  **L756 CN**: 声明 TableGen def `SETBUF`。
- **L757 EN**: Declares TableGen def `SETITIMER`.
  **L757 CN**: 声明 TableGen def `SETITIMER`。
- **L758 EN**: Declares TableGen def `SETVBUF`.
  **L758 CN**: 声明 TableGen def `SETVBUF`。
- **L759 EN**: Declares TableGen def `SIPRINTF`.
  **L759 CN**: 声明 TableGen def `SIPRINTF`。
- **L760 EN**: Declares TableGen def `SIZE_RETURNING_NEW`.
  **L760 CN**: 声明 TableGen def `SIZE_RETURNING_NEW`。
- **L761 EN**: Declares TableGen def `SIZE_RETURNING_NEW_ALIGNED`.
  **L761 CN**: 声明 TableGen def `SIZE_RETURNING_NEW_ALIGNED`。
- **L762 EN**: Declares TableGen def `SIZE_RETURNING_NEW_ALIGNED_HOT_COLD`.
  **L762 CN**: 声明 TableGen def `SIZE_RETURNING_NEW_ALIGNED_HOT_COLD`。
- **L763 EN**: Declares TableGen def `SIZE_RETURNING_NEW_HOT_COLD`.
  **L763 CN**: 声明 TableGen def `SIZE_RETURNING_NEW_HOT_COLD`。
- **L764 EN**: Declares TableGen def `SMALL_FPRINTF`.
  **L764 CN**: 声明 TableGen def `SMALL_FPRINTF`。
- **L765 EN**: Declares TableGen def `SMALL_PRINTF`.
  **L765 CN**: 声明 TableGen def `SMALL_PRINTF`。
- **L766 EN**: Declares TableGen def `SMALL_SPRINTF`.
  **L766 CN**: 声明 TableGen def `SMALL_SPRINTF`。
- **L767 EN**: Declares TableGen def `SNPRINTF`.
  **L767 CN**: 声明 TableGen def `SNPRINTF`。
- **L768 EN**: Declares TableGen def `SNPRINTF_CHK`.
  **L768 CN**: 声明 TableGen def `SNPRINTF_CHK`。

### Lines 769-800

````tablegen
def SPRINTF : RuntimeLibcall;
def SPRINTF_CHK : RuntimeLibcall;
def SSCANF : RuntimeLibcall;
def STAT : RuntimeLibcall;
def STAT64 : RuntimeLibcall;
def STATVFS : RuntimeLibcall;
def STATVFS64 : RuntimeLibcall;
def STPCPY : RuntimeLibcall;
def STPCPY_CHK : RuntimeLibcall;
def STPNCPY : RuntimeLibcall;
def STPNCPY_CHK : RuntimeLibcall;
def STRCASECMP : RuntimeLibcall;
def STRCAT : RuntimeLibcall;
def STRCAT_CHK : RuntimeLibcall;
def STRCHR : RuntimeLibcall;
def STRCMP : RuntimeLibcall;
def STRCOLL : RuntimeLibcall;
def STRCPY : RuntimeLibcall;
def STRCPY_CHK : RuntimeLibcall;
def STRCSPN : RuntimeLibcall;
def STRDUP : RuntimeLibcall;
def STRLCAT : RuntimeLibcall;
def STRLCAT_CHK : RuntimeLibcall;
def STRLCPY : RuntimeLibcall;
def STRLCPY_CHK : RuntimeLibcall;
def STRLEN_CHK : RuntimeLibcall;
def STRNCASECMP : RuntimeLibcall;
def STRNCAT : RuntimeLibcall;
def STRNCAT_CHK : RuntimeLibcall;
def STRNCMP : RuntimeLibcall;
def STRNCPY : RuntimeLibcall;
def STRNCPY_CHK : RuntimeLibcall;
````
- **L769 EN**: Declares TableGen def `SPRINTF`.
  **L769 CN**: 声明 TableGen def `SPRINTF`。
- **L770 EN**: Declares TableGen def `SPRINTF_CHK`.
  **L770 CN**: 声明 TableGen def `SPRINTF_CHK`。
- **L771 EN**: Declares TableGen def `SSCANF`.
  **L771 CN**: 声明 TableGen def `SSCANF`。
- **L772 EN**: Declares TableGen def `STAT`.
  **L772 CN**: 声明 TableGen def `STAT`。
- **L773 EN**: Declares TableGen def `STAT64`.
  **L773 CN**: 声明 TableGen def `STAT64`。
- **L774 EN**: Declares TableGen def `STATVFS`.
  **L774 CN**: 声明 TableGen def `STATVFS`。
- **L775 EN**: Declares TableGen def `STATVFS64`.
  **L775 CN**: 声明 TableGen def `STATVFS64`。
- **L776 EN**: Declares TableGen def `STPCPY`.
  **L776 CN**: 声明 TableGen def `STPCPY`。
- **L777 EN**: Declares TableGen def `STPCPY_CHK`.
  **L777 CN**: 声明 TableGen def `STPCPY_CHK`。
- **L778 EN**: Declares TableGen def `STPNCPY`.
  **L778 CN**: 声明 TableGen def `STPNCPY`。
- **L779 EN**: Declares TableGen def `STPNCPY_CHK`.
  **L779 CN**: 声明 TableGen def `STPNCPY_CHK`。
- **L780 EN**: Declares TableGen def `STRCASECMP`.
  **L780 CN**: 声明 TableGen def `STRCASECMP`。
- **L781 EN**: Declares TableGen def `STRCAT`.
  **L781 CN**: 声明 TableGen def `STRCAT`。
- **L782 EN**: Declares TableGen def `STRCAT_CHK`.
  **L782 CN**: 声明 TableGen def `STRCAT_CHK`。
- **L783 EN**: Declares TableGen def `STRCHR`.
  **L783 CN**: 声明 TableGen def `STRCHR`。
- **L784 EN**: Declares TableGen def `STRCMP`.
  **L784 CN**: 声明 TableGen def `STRCMP`。
- **L785 EN**: Declares TableGen def `STRCOLL`.
  **L785 CN**: 声明 TableGen def `STRCOLL`。
- **L786 EN**: Declares TableGen def `STRCPY`.
  **L786 CN**: 声明 TableGen def `STRCPY`。
- **L787 EN**: Declares TableGen def `STRCPY_CHK`.
  **L787 CN**: 声明 TableGen def `STRCPY_CHK`。
- **L788 EN**: Declares TableGen def `STRCSPN`.
  **L788 CN**: 声明 TableGen def `STRCSPN`。
- **L789 EN**: Declares TableGen def `STRDUP`.
  **L789 CN**: 声明 TableGen def `STRDUP`。
- **L790 EN**: Declares TableGen def `STRLCAT`.
  **L790 CN**: 声明 TableGen def `STRLCAT`。
- **L791 EN**: Declares TableGen def `STRLCAT_CHK`.
  **L791 CN**: 声明 TableGen def `STRLCAT_CHK`。
- **L792 EN**: Declares TableGen def `STRLCPY`.
  **L792 CN**: 声明 TableGen def `STRLCPY`。
- **L793 EN**: Declares TableGen def `STRLCPY_CHK`.
  **L793 CN**: 声明 TableGen def `STRLCPY_CHK`。
- **L794 EN**: Declares TableGen def `STRLEN_CHK`.
  **L794 CN**: 声明 TableGen def `STRLEN_CHK`。
- **L795 EN**: Declares TableGen def `STRNCASECMP`.
  **L795 CN**: 声明 TableGen def `STRNCASECMP`。
- **L796 EN**: Declares TableGen def `STRNCAT`.
  **L796 CN**: 声明 TableGen def `STRNCAT`。
- **L797 EN**: Declares TableGen def `STRNCAT_CHK`.
  **L797 CN**: 声明 TableGen def `STRNCAT_CHK`。
- **L798 EN**: Declares TableGen def `STRNCMP`.
  **L798 CN**: 声明 TableGen def `STRNCMP`。
- **L799 EN**: Declares TableGen def `STRNCPY`.
  **L799 CN**: 声明 TableGen def `STRNCPY`。
- **L800 EN**: Declares TableGen def `STRNCPY_CHK`.
  **L800 CN**: 声明 TableGen def `STRNCPY_CHK`。

### Lines 801-832

````tablegen
def STRNDUP : RuntimeLibcall;
def STRNLEN : RuntimeLibcall;
def STRPBRK : RuntimeLibcall;
def STRRCHR : RuntimeLibcall;
def STRSPN : RuntimeLibcall;
def STRSTR : RuntimeLibcall;
def STRTOD : RuntimeLibcall;
def STRTOF : RuntimeLibcall;
def STRTOK : RuntimeLibcall;
def STRTOK_R : RuntimeLibcall;
def STRTOL : RuntimeLibcall;
def STRTOLD : RuntimeLibcall;
def STRTOLL : RuntimeLibcall;
def STRTOUL : RuntimeLibcall;
def STRTOULL : RuntimeLibcall;
def STRXFRM : RuntimeLibcall;
def SYSTEM : RuntimeLibcall;
def TERMINATE : RuntimeLibcall;
def TIMES : RuntimeLibcall;
def TMPFILE : RuntimeLibcall;
def TMPFILE64 : RuntimeLibcall;
def TOASCII : RuntimeLibcall;
def UNAME : RuntimeLibcall;
def UNDER_IO_GETC : RuntimeLibcall;
def UNDER_IO_PUTC : RuntimeLibcall;
def UNGETC : RuntimeLibcall;
def UNLINK : RuntimeLibcall;
def UNSETENV : RuntimeLibcall;
def UTIME : RuntimeLibcall;
def UTIMES : RuntimeLibcall;
def VALLOC : RuntimeLibcall;
def VEC_CALLOC : RuntimeLibcall;
````
- **L801 EN**: Declares TableGen def `STRNDUP`.
  **L801 CN**: 声明 TableGen def `STRNDUP`。
- **L802 EN**: Declares TableGen def `STRNLEN`.
  **L802 CN**: 声明 TableGen def `STRNLEN`。
- **L803 EN**: Declares TableGen def `STRPBRK`.
  **L803 CN**: 声明 TableGen def `STRPBRK`。
- **L804 EN**: Declares TableGen def `STRRCHR`.
  **L804 CN**: 声明 TableGen def `STRRCHR`。
- **L805 EN**: Declares TableGen def `STRSPN`.
  **L805 CN**: 声明 TableGen def `STRSPN`。
- **L806 EN**: Declares TableGen def `STRSTR`.
  **L806 CN**: 声明 TableGen def `STRSTR`。
- **L807 EN**: Declares TableGen def `STRTOD`.
  **L807 CN**: 声明 TableGen def `STRTOD`。
- **L808 EN**: Declares TableGen def `STRTOF`.
  **L808 CN**: 声明 TableGen def `STRTOF`。
- **L809 EN**: Declares TableGen def `STRTOK`.
  **L809 CN**: 声明 TableGen def `STRTOK`。
- **L810 EN**: Declares TableGen def `STRTOK_R`.
  **L810 CN**: 声明 TableGen def `STRTOK_R`。
- **L811 EN**: Declares TableGen def `STRTOL`.
  **L811 CN**: 声明 TableGen def `STRTOL`。
- **L812 EN**: Declares TableGen def `STRTOLD`.
  **L812 CN**: 声明 TableGen def `STRTOLD`。
- **L813 EN**: Declares TableGen def `STRTOLL`.
  **L813 CN**: 声明 TableGen def `STRTOLL`。
- **L814 EN**: Declares TableGen def `STRTOUL`.
  **L814 CN**: 声明 TableGen def `STRTOUL`。
- **L815 EN**: Declares TableGen def `STRTOULL`.
  **L815 CN**: 声明 TableGen def `STRTOULL`。
- **L816 EN**: Declares TableGen def `STRXFRM`.
  **L816 CN**: 声明 TableGen def `STRXFRM`。
- **L817 EN**: Declares TableGen def `SYSTEM`.
  **L817 CN**: 声明 TableGen def `SYSTEM`。
- **L818 EN**: Declares TableGen def `TERMINATE`.
  **L818 CN**: 声明 TableGen def `TERMINATE`。
- **L819 EN**: Declares TableGen def `TIMES`.
  **L819 CN**: 声明 TableGen def `TIMES`。
- **L820 EN**: Declares TableGen def `TMPFILE`.
  **L820 CN**: 声明 TableGen def `TMPFILE`。
- **L821 EN**: Declares TableGen def `TMPFILE64`.
  **L821 CN**: 声明 TableGen def `TMPFILE64`。
- **L822 EN**: Declares TableGen def `TOASCII`.
  **L822 CN**: 声明 TableGen def `TOASCII`。
- **L823 EN**: Declares TableGen def `UNAME`.
  **L823 CN**: 声明 TableGen def `UNAME`。
- **L824 EN**: Declares TableGen def `UNDER_IO_GETC`.
  **L824 CN**: 声明 TableGen def `UNDER_IO_GETC`。
- **L825 EN**: Declares TableGen def `UNDER_IO_PUTC`.
  **L825 CN**: 声明 TableGen def `UNDER_IO_PUTC`。
- **L826 EN**: Declares TableGen def `UNGETC`.
  **L826 CN**: 声明 TableGen def `UNGETC`。
- **L827 EN**: Declares TableGen def `UNLINK`.
  **L827 CN**: 声明 TableGen def `UNLINK`。
- **L828 EN**: Declares TableGen def `UNSETENV`.
  **L828 CN**: 声明 TableGen def `UNSETENV`。
- **L829 EN**: Declares TableGen def `UTIME`.
  **L829 CN**: 声明 TableGen def `UTIME`。
- **L830 EN**: Declares TableGen def `UTIMES`.
  **L830 CN**: 声明 TableGen def `UTIMES`。
- **L831 EN**: Declares TableGen def `VALLOC`.
  **L831 CN**: 声明 TableGen def `VALLOC`。
- **L832 EN**: Declares TableGen def `VEC_CALLOC`.
  **L832 CN**: 声明 TableGen def `VEC_CALLOC`。

### Lines 833-864

````tablegen
def VEC_FREE : RuntimeLibcall;
def VEC_MALLOC : RuntimeLibcall;
def VEC_REALLOC : RuntimeLibcall;
def VFPRINTF : RuntimeLibcall;
def VFSCANF : RuntimeLibcall;
def VPRINTF : RuntimeLibcall;
def VSCANF : RuntimeLibcall;
def VSNPRINTF : RuntimeLibcall;
def VSNPRINTF_CHK : RuntimeLibcall;
def VSPRINTF : RuntimeLibcall;
def VSPRINTF_CHK : RuntimeLibcall;
def VSSCANF : RuntimeLibcall;
def WCSLEN : RuntimeLibcall;
def WRITE : RuntimeLibcall;
def ZDAPV : RuntimeLibcall;
def ZDAPVJ : RuntimeLibcall;
def ZDAPVJST11ALIGN_VAL_T : RuntimeLibcall;
def ZDAPVM : RuntimeLibcall;
def ZDAPVMST11ALIGN_VAL_T : RuntimeLibcall;
def ZDAPVRKST9NOTHROW_T : RuntimeLibcall;
def ZDAPVST11ALIGN_VAL_T : RuntimeLibcall;
def ZDAPVST11ALIGN_VAL_TRKST9NOTHROW_T : RuntimeLibcall;
def ZDLPV : RuntimeLibcall;
def ZDLPVJ : RuntimeLibcall;
def ZDLPVJST11ALIGN_VAL_T : RuntimeLibcall;
def ZDLPVM : RuntimeLibcall;
def ZDLPVMST11ALIGN_VAL_T : RuntimeLibcall;
def ZDLPVRKST9NOTHROW_T : RuntimeLibcall;
def ZDLPVST11ALIGN_VAL_T : RuntimeLibcall;
def ZDLPVST11ALIGN_VAL_TRKST9NOTHROW_T : RuntimeLibcall;
def ZNAJ : RuntimeLibcall;
def ZNAJRKST9NOTHROW_T : RuntimeLibcall;
````
- **L833 EN**: Declares TableGen def `VEC_FREE`.
  **L833 CN**: 声明 TableGen def `VEC_FREE`。
- **L834 EN**: Declares TableGen def `VEC_MALLOC`.
  **L834 CN**: 声明 TableGen def `VEC_MALLOC`。
- **L835 EN**: Declares TableGen def `VEC_REALLOC`.
  **L835 CN**: 声明 TableGen def `VEC_REALLOC`。
- **L836 EN**: Declares TableGen def `VFPRINTF`.
  **L836 CN**: 声明 TableGen def `VFPRINTF`。
- **L837 EN**: Declares TableGen def `VFSCANF`.
  **L837 CN**: 声明 TableGen def `VFSCANF`。
- **L838 EN**: Declares TableGen def `VPRINTF`.
  **L838 CN**: 声明 TableGen def `VPRINTF`。
- **L839 EN**: Declares TableGen def `VSCANF`.
  **L839 CN**: 声明 TableGen def `VSCANF`。
- **L840 EN**: Declares TableGen def `VSNPRINTF`.
  **L840 CN**: 声明 TableGen def `VSNPRINTF`。
- **L841 EN**: Declares TableGen def `VSNPRINTF_CHK`.
  **L841 CN**: 声明 TableGen def `VSNPRINTF_CHK`。
- **L842 EN**: Declares TableGen def `VSPRINTF`.
  **L842 CN**: 声明 TableGen def `VSPRINTF`。
- **L843 EN**: Declares TableGen def `VSPRINTF_CHK`.
  **L843 CN**: 声明 TableGen def `VSPRINTF_CHK`。
- **L844 EN**: Declares TableGen def `VSSCANF`.
  **L844 CN**: 声明 TableGen def `VSSCANF`。
- **L845 EN**: Declares TableGen def `WCSLEN`.
  **L845 CN**: 声明 TableGen def `WCSLEN`。
- **L846 EN**: Declares TableGen def `WRITE`.
  **L846 CN**: 声明 TableGen def `WRITE`。
- **L847 EN**: Declares TableGen def `ZDAPV`.
  **L847 CN**: 声明 TableGen def `ZDAPV`。
- **L848 EN**: Declares TableGen def `ZDAPVJ`.
  **L848 CN**: 声明 TableGen def `ZDAPVJ`。
- **L849 EN**: Declares TableGen def `ZDAPVJST11ALIGN_VAL_T`.
  **L849 CN**: 声明 TableGen def `ZDAPVJST11ALIGN_VAL_T`。
- **L850 EN**: Declares TableGen def `ZDAPVM`.
  **L850 CN**: 声明 TableGen def `ZDAPVM`。
- **L851 EN**: Declares TableGen def `ZDAPVMST11ALIGN_VAL_T`.
  **L851 CN**: 声明 TableGen def `ZDAPVMST11ALIGN_VAL_T`。
- **L852 EN**: Declares TableGen def `ZDAPVRKST9NOTHROW_T`.
  **L852 CN**: 声明 TableGen def `ZDAPVRKST9NOTHROW_T`。
- **L853 EN**: Declares TableGen def `ZDAPVST11ALIGN_VAL_T`.
  **L853 CN**: 声明 TableGen def `ZDAPVST11ALIGN_VAL_T`。
- **L854 EN**: Declares TableGen def `ZDAPVST11ALIGN_VAL_TRKST9NOTHROW_T`.
  **L854 CN**: 声明 TableGen def `ZDAPVST11ALIGN_VAL_TRKST9NOTHROW_T`。
- **L855 EN**: Declares TableGen def `ZDLPV`.
  **L855 CN**: 声明 TableGen def `ZDLPV`。
- **L856 EN**: Declares TableGen def `ZDLPVJ`.
  **L856 CN**: 声明 TableGen def `ZDLPVJ`。
- **L857 EN**: Declares TableGen def `ZDLPVJST11ALIGN_VAL_T`.
  **L857 CN**: 声明 TableGen def `ZDLPVJST11ALIGN_VAL_T`。
- **L858 EN**: Declares TableGen def `ZDLPVM`.
  **L858 CN**: 声明 TableGen def `ZDLPVM`。
- **L859 EN**: Declares TableGen def `ZDLPVMST11ALIGN_VAL_T`.
  **L859 CN**: 声明 TableGen def `ZDLPVMST11ALIGN_VAL_T`。
- **L860 EN**: Declares TableGen def `ZDLPVRKST9NOTHROW_T`.
  **L860 CN**: 声明 TableGen def `ZDLPVRKST9NOTHROW_T`。
- **L861 EN**: Declares TableGen def `ZDLPVST11ALIGN_VAL_T`.
  **L861 CN**: 声明 TableGen def `ZDLPVST11ALIGN_VAL_T`。
- **L862 EN**: Declares TableGen def `ZDLPVST11ALIGN_VAL_TRKST9NOTHROW_T`.
  **L862 CN**: 声明 TableGen def `ZDLPVST11ALIGN_VAL_TRKST9NOTHROW_T`。
- **L863 EN**: Declares TableGen def `ZNAJ`.
  **L863 CN**: 声明 TableGen def `ZNAJ`。
- **L864 EN**: Declares TableGen def `ZNAJRKST9NOTHROW_T`.
  **L864 CN**: 声明 TableGen def `ZNAJRKST9NOTHROW_T`。

### Lines 865-896

````tablegen
def ZNAJST11ALIGN_VAL_T : RuntimeLibcall;
def ZNAJST11ALIGN_VAL_TRKST9NOTHROW_T : RuntimeLibcall;
def ZNAM : RuntimeLibcall;
def ZNAM12__HOT_COLD_T : RuntimeLibcall;
def ZNAMRKST9NOTHROW_T : RuntimeLibcall;
def ZNAMRKST9NOTHROW_T12__HOT_COLD_T : RuntimeLibcall;
def ZNAMST11ALIGN_VAL_T : RuntimeLibcall;
def ZNAMST11ALIGN_VAL_T12__HOT_COLD_T : RuntimeLibcall;
def ZNAMST11ALIGN_VAL_TRKST9NOTHROW_T : RuntimeLibcall;
def ZNAMST11ALIGN_VAL_TRKST9NOTHROW_T12__HOT_COLD_T : RuntimeLibcall;
def ZNWJ : RuntimeLibcall;
def ZNWJRKST9NOTHROW_T : RuntimeLibcall;
def ZNWJST11ALIGN_VAL_T : RuntimeLibcall;
def ZNWJST11ALIGN_VAL_TRKST9NOTHROW_T : RuntimeLibcall;
def ZNWM : RuntimeLibcall;
def ZNWM12__HOT_COLD_T : RuntimeLibcall;
def ZNWMRKST9NOTHROW_T : RuntimeLibcall;
def ZNWMRKST9NOTHROW_T12__HOT_COLD_T : RuntimeLibcall;
def ZNWMST11ALIGN_VAL_T : RuntimeLibcall;
def ZNWMST11ALIGN_VAL_T12__HOT_COLD_T : RuntimeLibcall;
def ZNWMST11ALIGN_VAL_TRKST9NOTHROW_T : RuntimeLibcall;
def ZNWMST11ALIGN_VAL_TRKST9NOTHROW_T12__HOT_COLD_T : RuntimeLibcall;
def KMPC_ALLOC_SHARED : RuntimeLibcall;
def KMPC_FREE_SHARED : RuntimeLibcall;

//--------------------------------------------------------------------
// Global variable references
//--------------------------------------------------------------------
//
// TODO: These are not libcalls and probably should be distinguished
// in some way from callable functions.
// --------------------------------------------------------------------
````
- **L865 EN**: Declares TableGen def `ZNAJST11ALIGN_VAL_T`.
  **L865 CN**: 声明 TableGen def `ZNAJST11ALIGN_VAL_T`。
- **L866 EN**: Declares TableGen def `ZNAJST11ALIGN_VAL_TRKST9NOTHROW_T`.
  **L866 CN**: 声明 TableGen def `ZNAJST11ALIGN_VAL_TRKST9NOTHROW_T`。
- **L867 EN**: Declares TableGen def `ZNAM`.
  **L867 CN**: 声明 TableGen def `ZNAM`。
- **L868 EN**: Declares TableGen def `ZNAM12__HOT_COLD_T`.
  **L868 CN**: 声明 TableGen def `ZNAM12__HOT_COLD_T`。
- **L869 EN**: Declares TableGen def `ZNAMRKST9NOTHROW_T`.
  **L869 CN**: 声明 TableGen def `ZNAMRKST9NOTHROW_T`。
- **L870 EN**: Declares TableGen def `ZNAMRKST9NOTHROW_T12__HOT_COLD_T`.
  **L870 CN**: 声明 TableGen def `ZNAMRKST9NOTHROW_T12__HOT_COLD_T`。
- **L871 EN**: Declares TableGen def `ZNAMST11ALIGN_VAL_T`.
  **L871 CN**: 声明 TableGen def `ZNAMST11ALIGN_VAL_T`。
- **L872 EN**: Declares TableGen def `ZNAMST11ALIGN_VAL_T12__HOT_COLD_T`.
  **L872 CN**: 声明 TableGen def `ZNAMST11ALIGN_VAL_T12__HOT_COLD_T`。
- **L873 EN**: Declares TableGen def `ZNAMST11ALIGN_VAL_TRKST9NOTHROW_T`.
  **L873 CN**: 声明 TableGen def `ZNAMST11ALIGN_VAL_TRKST9NOTHROW_T`。
- **L874 EN**: Declares TableGen def `ZNAMST11ALIGN_VAL_TRKST9NOTHROW_T12__HOT_COLD_T`.
  **L874 CN**: 声明 TableGen def `ZNAMST11ALIGN_VAL_TRKST9NOTHROW_T12__HOT_COLD_T`。
- **L875 EN**: Declares TableGen def `ZNWJ`.
  **L875 CN**: 声明 TableGen def `ZNWJ`。
- **L876 EN**: Declares TableGen def `ZNWJRKST9NOTHROW_T`.
  **L876 CN**: 声明 TableGen def `ZNWJRKST9NOTHROW_T`。
- **L877 EN**: Declares TableGen def `ZNWJST11ALIGN_VAL_T`.
  **L877 CN**: 声明 TableGen def `ZNWJST11ALIGN_VAL_T`。
- **L878 EN**: Declares TableGen def `ZNWJST11ALIGN_VAL_TRKST9NOTHROW_T`.
  **L878 CN**: 声明 TableGen def `ZNWJST11ALIGN_VAL_TRKST9NOTHROW_T`。
- **L879 EN**: Declares TableGen def `ZNWM`.
  **L879 CN**: 声明 TableGen def `ZNWM`。
- **L880 EN**: Declares TableGen def `ZNWM12__HOT_COLD_T`.
  **L880 CN**: 声明 TableGen def `ZNWM12__HOT_COLD_T`。
- **L881 EN**: Declares TableGen def `ZNWMRKST9NOTHROW_T`.
  **L881 CN**: 声明 TableGen def `ZNWMRKST9NOTHROW_T`。
- **L882 EN**: Declares TableGen def `ZNWMRKST9NOTHROW_T12__HOT_COLD_T`.
  **L882 CN**: 声明 TableGen def `ZNWMRKST9NOTHROW_T12__HOT_COLD_T`。
- **L883 EN**: Declares TableGen def `ZNWMST11ALIGN_VAL_T`.
  **L883 CN**: 声明 TableGen def `ZNWMST11ALIGN_VAL_T`。
- **L884 EN**: Declares TableGen def `ZNWMST11ALIGN_VAL_T12__HOT_COLD_T`.
  **L884 CN**: 声明 TableGen def `ZNWMST11ALIGN_VAL_T12__HOT_COLD_T`。
- **L885 EN**: Declares TableGen def `ZNWMST11ALIGN_VAL_TRKST9NOTHROW_T`.
  **L885 CN**: 声明 TableGen def `ZNWMST11ALIGN_VAL_TRKST9NOTHROW_T`。
- **L886 EN**: Declares TableGen def `ZNWMST11ALIGN_VAL_TRKST9NOTHROW_T12__HOT_COLD_T`.
  **L886 CN**: 声明 TableGen def `ZNWMST11ALIGN_VAL_TRKST9NOTHROW_T12__HOT_COLD_T`。
- **L887 EN**: Declares TableGen def `KMPC_ALLOC_SHARED`.
  **L887 CN**: 声明 TableGen def `KMPC_ALLOC_SHARED`。
- **L888 EN**: Declares TableGen def `KMPC_FREE_SHARED`.
  **L888 CN**: 声明 TableGen def `KMPC_FREE_SHARED`。
- **L889 EN**: Blank line separating nearby declarations or logic blocks.
  **L889 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L890 EN**: Comment explains nearby logic, invariants, or intent: `--------------------------------------------------------------------`.
  **L890 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`--------------------------------------------------------------------`。
- **L891 EN**: Comment explains nearby logic, invariants, or intent: `Global variable references`.
  **L891 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Global variable references`。
- **L892 EN**: Comment explains nearby logic, invariants, or intent: `--------------------------------------------------------------------`.
  **L892 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`--------------------------------------------------------------------`。
- **L893 EN**: Separator comment used for visual grouping.
  **L893 CN**: 用于视觉分组的分隔注释。
- **L894 EN**: Comment records a pending task or caution: `TODO: These are not libcalls and probably should be distinguished`.
  **L894 CN**: 注释记录了待办事项或注意点：`TODO: These are not libcalls and probably should be distinguished`。
- **L895 EN**: Comment explains nearby logic, invariants, or intent: `in some way from callable functions.`.
  **L895 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`in some way from callable functions.`。
- **L896 EN**: Comment explains nearby logic, invariants, or intent: `--------------------------------------------------------------------`.
  **L896 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`--------------------------------------------------------------------`。

### Lines 897-928

````tablegen

def STACK_CHECK_GUARD : RuntimeLibcall;

//--------------------------------------------------------------------
// Define implementation default libcalls
//--------------------------------------------------------------------

let IsDefault = true in {

//--------------------------------------------------------------------
// compiler-rt / libgcc
//--------------------------------------------------------------------

def __ashlhi3 : RuntimeLibcallImpl<SHL_I16>;
def __ashlsi3 : RuntimeLibcallImpl<SHL_I32>;
def __ashldi3 : RuntimeLibcallImpl<SHL_I64>;

def __lshrhi3 : RuntimeLibcallImpl<SRL_I16>;
def __lshrsi3 : RuntimeLibcallImpl<SRL_I32>;
def __lshrdi3 : RuntimeLibcallImpl<SRL_I64>;

def __ashrhi3 : RuntimeLibcallImpl<SRA_I16>;
def __ashrsi3 : RuntimeLibcallImpl<SRA_I32>;
def __ashrdi3 : RuntimeLibcallImpl<SRA_I64>;

def __mulqi3 : RuntimeLibcallImpl<MUL_I8>;
def __mulhi3 : RuntimeLibcallImpl<MUL_I16>;
def __mulsi3 : RuntimeLibcallImpl<MUL_I32>;
def __muldi3 : RuntimeLibcallImpl<MUL_I64>;

def __mulosi4 : RuntimeLibcallImpl<MULO_I32>;

````
- **L897 EN**: Blank line separating nearby declarations or logic blocks.
  **L897 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L898 EN**: Declares TableGen def `STACK_CHECK_GUARD`.
  **L898 CN**: 声明 TableGen def `STACK_CHECK_GUARD`。
- **L899 EN**: Blank line separating nearby declarations or logic blocks.
  **L899 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L900 EN**: Comment explains nearby logic, invariants, or intent: `--------------------------------------------------------------------`.
  **L900 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`--------------------------------------------------------------------`。
- **L901 EN**: Comment explains nearby logic, invariants, or intent: `Define implementation default libcalls`.
  **L901 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Define implementation default libcalls`。
- **L902 EN**: Comment explains nearby logic, invariants, or intent: `--------------------------------------------------------------------`.
  **L902 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`--------------------------------------------------------------------`。
- **L903 EN**: Blank line separating nearby declarations or logic blocks.
  **L903 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L904 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L904 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L905 EN**: Blank line separating nearby declarations or logic blocks.
  **L905 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L906 EN**: Comment explains nearby logic, invariants, or intent: `--------------------------------------------------------------------`.
  **L906 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`--------------------------------------------------------------------`。
- **L907 EN**: Comment explains nearby logic, invariants, or intent: `compiler-rt / libgcc`.
  **L907 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`compiler-rt / libgcc`。
- **L908 EN**: Comment explains nearby logic, invariants, or intent: `--------------------------------------------------------------------`.
  **L908 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`--------------------------------------------------------------------`。
- **L909 EN**: Blank line separating nearby declarations or logic blocks.
  **L909 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L910 EN**: Declares TableGen def `__ashlhi3`.
  **L910 CN**: 声明 TableGen def `__ashlhi3`。
- **L911 EN**: Declares TableGen def `__ashlsi3`.
  **L911 CN**: 声明 TableGen def `__ashlsi3`。
- **L912 EN**: Declares TableGen def `__ashldi3`.
  **L912 CN**: 声明 TableGen def `__ashldi3`。
- **L913 EN**: Blank line separating nearby declarations or logic blocks.
  **L913 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L914 EN**: Declares TableGen def `__lshrhi3`.
  **L914 CN**: 声明 TableGen def `__lshrhi3`。
- **L915 EN**: Declares TableGen def `__lshrsi3`.
  **L915 CN**: 声明 TableGen def `__lshrsi3`。
- **L916 EN**: Declares TableGen def `__lshrdi3`.
  **L916 CN**: 声明 TableGen def `__lshrdi3`。
- **L917 EN**: Blank line separating nearby declarations or logic blocks.
  **L917 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L918 EN**: Declares TableGen def `__ashrhi3`.
  **L918 CN**: 声明 TableGen def `__ashrhi3`。
- **L919 EN**: Declares TableGen def `__ashrsi3`.
  **L919 CN**: 声明 TableGen def `__ashrsi3`。
- **L920 EN**: Declares TableGen def `__ashrdi3`.
  **L920 CN**: 声明 TableGen def `__ashrdi3`。
- **L921 EN**: Blank line separating nearby declarations or logic blocks.
  **L921 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L922 EN**: Declares TableGen def `__mulqi3`.
  **L922 CN**: 声明 TableGen def `__mulqi3`。
- **L923 EN**: Declares TableGen def `__mulhi3`.
  **L923 CN**: 声明 TableGen def `__mulhi3`。
- **L924 EN**: Declares TableGen def `__mulsi3`.
  **L924 CN**: 声明 TableGen def `__mulsi3`。
- **L925 EN**: Declares TableGen def `__muldi3`.
  **L925 CN**: 声明 TableGen def `__muldi3`。
- **L926 EN**: Blank line separating nearby declarations or logic blocks.
  **L926 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L927 EN**: Declares TableGen def `__mulosi4`.
  **L927 CN**: 声明 TableGen def `__mulosi4`。
- **L928 EN**: Blank line separating nearby declarations or logic blocks.
  **L928 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 929-960

````tablegen
def __divqi3 : RuntimeLibcallImpl<SDIV_I8>;
def __divhi3 : RuntimeLibcallImpl<SDIV_I16>;
def __divsi3 : RuntimeLibcallImpl<SDIV_I32>;
def __divdi3 : RuntimeLibcallImpl<SDIV_I64>;
def __divti3 : RuntimeLibcallImpl<SDIV_I128>;

def __udivqi3 : RuntimeLibcallImpl<UDIV_I8>;
def __udivhi3 : RuntimeLibcallImpl<UDIV_I16>;
def __udivsi3 : RuntimeLibcallImpl<UDIV_I32>;
def __udivdi3 : RuntimeLibcallImpl<UDIV_I64>;
def __udivti3 : RuntimeLibcallImpl<UDIV_I128>;

def __modqi3 : RuntimeLibcallImpl<SREM_I8>;
def __modhi3 : RuntimeLibcallImpl<SREM_I16>;
def __modsi3 : RuntimeLibcallImpl<SREM_I32>;
def __moddi3 : RuntimeLibcallImpl<SREM_I64>;
def __modti3 : RuntimeLibcallImpl<SREM_I128>;

def __umodqi3 : RuntimeLibcallImpl<UREM_I8>;
def __umodhi3 : RuntimeLibcallImpl<UREM_I16>;
def __umodsi3 : RuntimeLibcallImpl<UREM_I32>;
def __umoddi3 : RuntimeLibcallImpl<UREM_I64>;
def __umodti3 : RuntimeLibcallImpl<UREM_I128>;

def __negsi2 : RuntimeLibcallImpl<NEG_I32>;
def __negdi2 : RuntimeLibcallImpl<NEG_I64>;

def __clzsi2 : RuntimeLibcallImpl<CTLZ_I32>;
def __clzdi2 : RuntimeLibcallImpl<CTLZ_I64>;
def __clzti2 : RuntimeLibcallImpl<CTLZ_I128>;

def __popcountsi2 : RuntimeLibcallImpl<CTPOP_I32>;
````
- **L929 EN**: Declares TableGen def `__divqi3`.
  **L929 CN**: 声明 TableGen def `__divqi3`。
- **L930 EN**: Declares TableGen def `__divhi3`.
  **L930 CN**: 声明 TableGen def `__divhi3`。
- **L931 EN**: Declares TableGen def `__divsi3`.
  **L931 CN**: 声明 TableGen def `__divsi3`。
- **L932 EN**: Declares TableGen def `__divdi3`.
  **L932 CN**: 声明 TableGen def `__divdi3`。
- **L933 EN**: Declares TableGen def `__divti3`.
  **L933 CN**: 声明 TableGen def `__divti3`。
- **L934 EN**: Blank line separating nearby declarations or logic blocks.
  **L934 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L935 EN**: Declares TableGen def `__udivqi3`.
  **L935 CN**: 声明 TableGen def `__udivqi3`。
- **L936 EN**: Declares TableGen def `__udivhi3`.
  **L936 CN**: 声明 TableGen def `__udivhi3`。
- **L937 EN**: Declares TableGen def `__udivsi3`.
  **L937 CN**: 声明 TableGen def `__udivsi3`。
- **L938 EN**: Declares TableGen def `__udivdi3`.
  **L938 CN**: 声明 TableGen def `__udivdi3`。
- **L939 EN**: Declares TableGen def `__udivti3`.
  **L939 CN**: 声明 TableGen def `__udivti3`。
- **L940 EN**: Blank line separating nearby declarations or logic blocks.
  **L940 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L941 EN**: Declares TableGen def `__modqi3`.
  **L941 CN**: 声明 TableGen def `__modqi3`。
- **L942 EN**: Declares TableGen def `__modhi3`.
  **L942 CN**: 声明 TableGen def `__modhi3`。
- **L943 EN**: Declares TableGen def `__modsi3`.
  **L943 CN**: 声明 TableGen def `__modsi3`。
- **L944 EN**: Declares TableGen def `__moddi3`.
  **L944 CN**: 声明 TableGen def `__moddi3`。
- **L945 EN**: Declares TableGen def `__modti3`.
  **L945 CN**: 声明 TableGen def `__modti3`。
- **L946 EN**: Blank line separating nearby declarations or logic blocks.
  **L946 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L947 EN**: Declares TableGen def `__umodqi3`.
  **L947 CN**: 声明 TableGen def `__umodqi3`。
- **L948 EN**: Declares TableGen def `__umodhi3`.
  **L948 CN**: 声明 TableGen def `__umodhi3`。
- **L949 EN**: Declares TableGen def `__umodsi3`.
  **L949 CN**: 声明 TableGen def `__umodsi3`。
- **L950 EN**: Declares TableGen def `__umoddi3`.
  **L950 CN**: 声明 TableGen def `__umoddi3`。
- **L951 EN**: Declares TableGen def `__umodti3`.
  **L951 CN**: 声明 TableGen def `__umodti3`。
- **L952 EN**: Blank line separating nearby declarations or logic blocks.
  **L952 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L953 EN**: Declares TableGen def `__negsi2`.
  **L953 CN**: 声明 TableGen def `__negsi2`。
- **L954 EN**: Declares TableGen def `__negdi2`.
  **L954 CN**: 声明 TableGen def `__negdi2`。
- **L955 EN**: Blank line separating nearby declarations or logic blocks.
  **L955 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L956 EN**: Declares TableGen def `__clzsi2`.
  **L956 CN**: 声明 TableGen def `__clzsi2`。
- **L957 EN**: Declares TableGen def `__clzdi2`.
  **L957 CN**: 声明 TableGen def `__clzdi2`。
- **L958 EN**: Declares TableGen def `__clzti2`.
  **L958 CN**: 声明 TableGen def `__clzti2`。
- **L959 EN**: Blank line separating nearby declarations or logic blocks.
  **L959 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L960 EN**: Declares TableGen def `__popcountsi2`.
  **L960 CN**: 声明 TableGen def `__popcountsi2`。

### Lines 961-992

````tablegen
def __popcountdi2 : RuntimeLibcallImpl<CTPOP_I64>;
def __popcountti2 : RuntimeLibcallImpl<CTPOP_I128>;

def __addsf3 : RuntimeLibcallImpl<ADD_F32>;
def __adddf3 : RuntimeLibcallImpl<ADD_F64>;
def __addxf3 : RuntimeLibcallImpl<ADD_F80>;
def __addtf3 : RuntimeLibcallImpl<ADD_F128>;
def __gcc_qadd : RuntimeLibcallImpl<ADD_PPCF128>;

def __subsf3 : RuntimeLibcallImpl<SUB_F32>;
def __subdf3 : RuntimeLibcallImpl<SUB_F64>;
def __subxf3 : RuntimeLibcallImpl<SUB_F80>;
def __subtf3 : RuntimeLibcallImpl<SUB_F128>;
def __gcc_qsub : RuntimeLibcallImpl<SUB_PPCF128>;

def __mulsf3 : RuntimeLibcallImpl<MUL_F32>;
def __muldf3 : RuntimeLibcallImpl<MUL_F64>;
def __mulxf3 : RuntimeLibcallImpl<MUL_F80>;
def __multf3 : RuntimeLibcallImpl<MUL_F128>;
def __gcc_qmul : RuntimeLibcallImpl<MUL_PPCF128>;

def __divsf3 : RuntimeLibcallImpl<DIV_F32>;
def __divdf3 : RuntimeLibcallImpl<DIV_F64>;
def __divxf3 : RuntimeLibcallImpl<DIV_F80>;
def __divtf3 : RuntimeLibcallImpl<DIV_F128>;
def __gcc_qdiv : RuntimeLibcallImpl<DIV_PPCF128>;

defset list<RuntimeLibcallImpl> PowiLibcallImpls = {
def __powisf2 : RuntimeLibcallImpl<POWI_F32>;
def __powidf2 : RuntimeLibcallImpl<POWI_F64>;
def __powixf2 : RuntimeLibcallImpl<POWI_F80>;
def __powitf2_f128 : RuntimeLibcallImpl<POWI_F128, "__powitf2">;
````
- **L961 EN**: Declares TableGen def `__popcountdi2`.
  **L961 CN**: 声明 TableGen def `__popcountdi2`。
- **L962 EN**: Declares TableGen def `__popcountti2`.
  **L962 CN**: 声明 TableGen def `__popcountti2`。
- **L963 EN**: Blank line separating nearby declarations or logic blocks.
  **L963 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L964 EN**: Declares TableGen def `__addsf3`.
  **L964 CN**: 声明 TableGen def `__addsf3`。
- **L965 EN**: Declares TableGen def `__adddf3`.
  **L965 CN**: 声明 TableGen def `__adddf3`。
- **L966 EN**: Declares TableGen def `__addxf3`.
  **L966 CN**: 声明 TableGen def `__addxf3`。
- **L967 EN**: Declares TableGen def `__addtf3`.
  **L967 CN**: 声明 TableGen def `__addtf3`。
- **L968 EN**: Declares TableGen def `__gcc_qadd`.
  **L968 CN**: 声明 TableGen def `__gcc_qadd`。
- **L969 EN**: Blank line separating nearby declarations or logic blocks.
  **L969 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L970 EN**: Declares TableGen def `__subsf3`.
  **L970 CN**: 声明 TableGen def `__subsf3`。
- **L971 EN**: Declares TableGen def `__subdf3`.
  **L971 CN**: 声明 TableGen def `__subdf3`。
- **L972 EN**: Declares TableGen def `__subxf3`.
  **L972 CN**: 声明 TableGen def `__subxf3`。
- **L973 EN**: Declares TableGen def `__subtf3`.
  **L973 CN**: 声明 TableGen def `__subtf3`。
- **L974 EN**: Declares TableGen def `__gcc_qsub`.
  **L974 CN**: 声明 TableGen def `__gcc_qsub`。
- **L975 EN**: Blank line separating nearby declarations or logic blocks.
  **L975 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L976 EN**: Declares TableGen def `__mulsf3`.
  **L976 CN**: 声明 TableGen def `__mulsf3`。
- **L977 EN**: Declares TableGen def `__muldf3`.
  **L977 CN**: 声明 TableGen def `__muldf3`。
- **L978 EN**: Declares TableGen def `__mulxf3`.
  **L978 CN**: 声明 TableGen def `__mulxf3`。
- **L979 EN**: Declares TableGen def `__multf3`.
  **L979 CN**: 声明 TableGen def `__multf3`。
- **L980 EN**: Declares TableGen def `__gcc_qmul`.
  **L980 CN**: 声明 TableGen def `__gcc_qmul`。
- **L981 EN**: Blank line separating nearby declarations or logic blocks.
  **L981 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L982 EN**: Declares TableGen def `__divsf3`.
  **L982 CN**: 声明 TableGen def `__divsf3`。
- **L983 EN**: Declares TableGen def `__divdf3`.
  **L983 CN**: 声明 TableGen def `__divdf3`。
- **L984 EN**: Declares TableGen def `__divxf3`.
  **L984 CN**: 声明 TableGen def `__divxf3`。
- **L985 EN**: Declares TableGen def `__divtf3`.
  **L985 CN**: 声明 TableGen def `__divtf3`。
- **L986 EN**: Declares TableGen def `__gcc_qdiv`.
  **L986 CN**: 声明 TableGen def `__gcc_qdiv`。
- **L987 EN**: Blank line separating nearby declarations or logic blocks.
  **L987 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L988 EN**: Continues the surrounding expression or declaration: `defset list<RuntimeLibcallImpl> PowiLibcallImpls = {`.
  **L988 CN**: 继续构造周围的表达式或声明：`defset list<RuntimeLibcallImpl> PowiLibcallImpls = {`。
- **L989 EN**: Declares TableGen def `__powisf2`.
  **L989 CN**: 声明 TableGen def `__powisf2`。
- **L990 EN**: Declares TableGen def `__powidf2`.
  **L990 CN**: 声明 TableGen def `__powidf2`。
- **L991 EN**: Declares TableGen def `__powixf2`.
  **L991 CN**: 声明 TableGen def `__powixf2`。
- **L992 EN**: Declares TableGen def `__powitf2_f128`.
  **L992 CN**: 声明 TableGen def `__powitf2_f128`。

### Lines 993-1024

````tablegen
def __powitf2_ppc128 : RuntimeLibcallImpl<POWI_PPCF128, "__powitf2">;
}

// Conversion
def __extendbfsf2 : RuntimeLibcallImpl<FPEXT_BF16_F32>;
def __gcc_stoq : RuntimeLibcallImpl<FPEXT_F32_PPCF128>;
def __gcc_dtoq : RuntimeLibcallImpl<FPEXT_F64_PPCF128>;
def __extendxftf2 : RuntimeLibcallImpl<FPEXT_F80_F128>;
def __extenddftf2 : RuntimeLibcallImpl<FPEXT_F64_F128>;
def __extendsftf2 : RuntimeLibcallImpl<FPEXT_F32_F128>;
def __extendhftf2 : RuntimeLibcallImpl<FPEXT_F16_F128>;
def __extendhfxf2 : RuntimeLibcallImpl<FPEXT_F16_F80>;
def __extendsfdf2 : RuntimeLibcallImpl<FPEXT_F32_F64>;
def __extendhfdf2 : RuntimeLibcallImpl<FPEXT_F16_F64>;
def __extendhfsf2 : RuntimeLibcallImpl<FPEXT_F16_F32>;
def __truncsfhf2 : RuntimeLibcallImpl<FPROUND_F32_F16>;
def __truncdfhf2 : RuntimeLibcallImpl<FPROUND_F64_F16>;
def __truncxfhf2 : RuntimeLibcallImpl<FPROUND_F80_F16>;
def __trunctfhf2_f128 : RuntimeLibcallImpl<FPROUND_F128_F16, "__trunctfhf2">;
def __trunctfhf2_ppcf128 : RuntimeLibcallImpl<FPROUND_PPCF128_F16, "__trunctfhf2">;
def __truncsfbf2 : RuntimeLibcallImpl<FPROUND_F32_BF16>;
def __truncdfbf2 : RuntimeLibcallImpl<FPROUND_F64_BF16>;
def __truncxfbf2 : RuntimeLibcallImpl<FPROUND_F80_BF16>;
def __trunctfbf2 : RuntimeLibcallImpl<FPROUND_F128_BF16>;
def __truncdfsf2 : RuntimeLibcallImpl<FPROUND_F64_F32>;
def __truncxfsf2 : RuntimeLibcallImpl<FPROUND_F80_F32>;
def __trunctfsf2 : RuntimeLibcallImpl<FPROUND_F128_F32>;
def __gcc_qtos : RuntimeLibcallImpl<FPROUND_PPCF128_F32>;
def __truncxfdf2 : RuntimeLibcallImpl<FPROUND_F80_F64>;
def __trunctfdf2 : RuntimeLibcallImpl<FPROUND_F128_F64>;
def __gcc_qtod : RuntimeLibcallImpl<FPROUND_PPCF128_F64>;
def __trunctfxf2 : RuntimeLibcallImpl<FPROUND_F128_F80>;
````
- **L993 EN**: Declares TableGen def `__powitf2_ppc128`.
  **L993 CN**: 声明 TableGen def `__powitf2_ppc128`。
- **L994 EN**: Closes the current lexical scope or compound statement.
  **L994 CN**: 结束当前词法作用域或复合语句块。
- **L995 EN**: Blank line separating nearby declarations or logic blocks.
  **L995 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L996 EN**: Comment explains nearby logic, invariants, or intent: `Conversion`.
  **L996 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Conversion`。
- **L997 EN**: Declares TableGen def `__extendbfsf2`.
  **L997 CN**: 声明 TableGen def `__extendbfsf2`。
- **L998 EN**: Declares TableGen def `__gcc_stoq`.
  **L998 CN**: 声明 TableGen def `__gcc_stoq`。
- **L999 EN**: Declares TableGen def `__gcc_dtoq`.
  **L999 CN**: 声明 TableGen def `__gcc_dtoq`。
- **L1000 EN**: Declares TableGen def `__extendxftf2`.
  **L1000 CN**: 声明 TableGen def `__extendxftf2`。
- **L1001 EN**: Declares TableGen def `__extenddftf2`.
  **L1001 CN**: 声明 TableGen def `__extenddftf2`。
- **L1002 EN**: Declares TableGen def `__extendsftf2`.
  **L1002 CN**: 声明 TableGen def `__extendsftf2`。
- **L1003 EN**: Declares TableGen def `__extendhftf2`.
  **L1003 CN**: 声明 TableGen def `__extendhftf2`。
- **L1004 EN**: Declares TableGen def `__extendhfxf2`.
  **L1004 CN**: 声明 TableGen def `__extendhfxf2`。
- **L1005 EN**: Declares TableGen def `__extendsfdf2`.
  **L1005 CN**: 声明 TableGen def `__extendsfdf2`。
- **L1006 EN**: Declares TableGen def `__extendhfdf2`.
  **L1006 CN**: 声明 TableGen def `__extendhfdf2`。
- **L1007 EN**: Declares TableGen def `__extendhfsf2`.
  **L1007 CN**: 声明 TableGen def `__extendhfsf2`。
- **L1008 EN**: Declares TableGen def `__truncsfhf2`.
  **L1008 CN**: 声明 TableGen def `__truncsfhf2`。
- **L1009 EN**: Declares TableGen def `__truncdfhf2`.
  **L1009 CN**: 声明 TableGen def `__truncdfhf2`。
- **L1010 EN**: Declares TableGen def `__truncxfhf2`.
  **L1010 CN**: 声明 TableGen def `__truncxfhf2`。
- **L1011 EN**: Declares TableGen def `__trunctfhf2_f128`.
  **L1011 CN**: 声明 TableGen def `__trunctfhf2_f128`。
- **L1012 EN**: Declares TableGen def `__trunctfhf2_ppcf128`.
  **L1012 CN**: 声明 TableGen def `__trunctfhf2_ppcf128`。
- **L1013 EN**: Declares TableGen def `__truncsfbf2`.
  **L1013 CN**: 声明 TableGen def `__truncsfbf2`。
- **L1014 EN**: Declares TableGen def `__truncdfbf2`.
  **L1014 CN**: 声明 TableGen def `__truncdfbf2`。
- **L1015 EN**: Declares TableGen def `__truncxfbf2`.
  **L1015 CN**: 声明 TableGen def `__truncxfbf2`。
- **L1016 EN**: Declares TableGen def `__trunctfbf2`.
  **L1016 CN**: 声明 TableGen def `__trunctfbf2`。
- **L1017 EN**: Declares TableGen def `__truncdfsf2`.
  **L1017 CN**: 声明 TableGen def `__truncdfsf2`。
- **L1018 EN**: Declares TableGen def `__truncxfsf2`.
  **L1018 CN**: 声明 TableGen def `__truncxfsf2`。
- **L1019 EN**: Declares TableGen def `__trunctfsf2`.
  **L1019 CN**: 声明 TableGen def `__trunctfsf2`。
- **L1020 EN**: Declares TableGen def `__gcc_qtos`.
  **L1020 CN**: 声明 TableGen def `__gcc_qtos`。
- **L1021 EN**: Declares TableGen def `__truncxfdf2`.
  **L1021 CN**: 声明 TableGen def `__truncxfdf2`。
- **L1022 EN**: Declares TableGen def `__trunctfdf2`.
  **L1022 CN**: 声明 TableGen def `__trunctfdf2`。
- **L1023 EN**: Declares TableGen def `__gcc_qtod`.
  **L1023 CN**: 声明 TableGen def `__gcc_qtod`。
- **L1024 EN**: Declares TableGen def `__trunctfxf2`.
  **L1024 CN**: 声明 TableGen def `__trunctfxf2`。

### Lines 1025-1056

````tablegen
def __fixhfsi : RuntimeLibcallImpl<FPTOSINT_F16_I32>;
def __fixhfdi : RuntimeLibcallImpl<FPTOSINT_F16_I64>;
def __fixhfti : RuntimeLibcallImpl<FPTOSINT_F16_I128>;
def __fixsfsi : RuntimeLibcallImpl<FPTOSINT_F32_I32>;
def __fixsfdi : RuntimeLibcallImpl<FPTOSINT_F32_I64>;
def __fixsfti : RuntimeLibcallImpl<FPTOSINT_F32_I128>;
def __fixdfsi : RuntimeLibcallImpl<FPTOSINT_F64_I32>;
def __fixdfdi : RuntimeLibcallImpl<FPTOSINT_F64_I64>;
def __fixdfti : RuntimeLibcallImpl<FPTOSINT_F64_I128>;
def __fixxfsi : RuntimeLibcallImpl<FPTOSINT_F80_I32>;
def __fixxfdi : RuntimeLibcallImpl<FPTOSINT_F80_I64>;
def __fixxfti : RuntimeLibcallImpl<FPTOSINT_F80_I128>;
def __fixtfsi : RuntimeLibcallImpl<FPTOSINT_F128_I32>;
def __fixtfdi_f128 : RuntimeLibcallImpl<FPTOSINT_F128_I64, "__fixtfdi">;
def __fixtfti_f128 : RuntimeLibcallImpl<FPTOSINT_F128_I128, "__fixtfti">;
def __gcc_qtou : RuntimeLibcallImpl<FPTOSINT_PPCF128_I32>;
def __fixtfdi_ppcf128 : RuntimeLibcallImpl<FPTOSINT_PPCF128_I64, "__fixtfdi">;
def __fixtfti_ppcf128 : RuntimeLibcallImpl<FPTOSINT_PPCF128_I128, "__fixtfti">;
def __fixunshfsi : RuntimeLibcallImpl<FPTOUINT_F16_I32>;
def __fixunshfdi : RuntimeLibcallImpl<FPTOUINT_F16_I64>;
def __fixunshfti : RuntimeLibcallImpl<FPTOUINT_F16_I128>;
def __fixunssfsi : RuntimeLibcallImpl<FPTOUINT_F32_I32>;
def __fixunssfdi : RuntimeLibcallImpl<FPTOUINT_F32_I64>;
def __fixunssfti : RuntimeLibcallImpl<FPTOUINT_F32_I128>;
def __fixunsdfsi : RuntimeLibcallImpl<FPTOUINT_F64_I32>;
def __fixunsdfdi : RuntimeLibcallImpl<FPTOUINT_F64_I64>;
def __fixunsdfti : RuntimeLibcallImpl<FPTOUINT_F64_I128>;
def __fixunsxfsi : RuntimeLibcallImpl<FPTOUINT_F80_I32>;
def __fixunsxfdi : RuntimeLibcallImpl<FPTOUINT_F80_I64>;
def __fixunsxfti : RuntimeLibcallImpl<FPTOUINT_F80_I128>;
def __fixunstfsi_f128 : RuntimeLibcallImpl<FPTOUINT_F128_I32, "__fixunstfsi">;
def __fixunstfdi_f128 : RuntimeLibcallImpl<FPTOUINT_F128_I64, "__fixunstfdi">;
````
- **L1025 EN**: Declares TableGen def `__fixhfsi`.
  **L1025 CN**: 声明 TableGen def `__fixhfsi`。
- **L1026 EN**: Declares TableGen def `__fixhfdi`.
  **L1026 CN**: 声明 TableGen def `__fixhfdi`。
- **L1027 EN**: Declares TableGen def `__fixhfti`.
  **L1027 CN**: 声明 TableGen def `__fixhfti`。
- **L1028 EN**: Declares TableGen def `__fixsfsi`.
  **L1028 CN**: 声明 TableGen def `__fixsfsi`。
- **L1029 EN**: Declares TableGen def `__fixsfdi`.
  **L1029 CN**: 声明 TableGen def `__fixsfdi`。
- **L1030 EN**: Declares TableGen def `__fixsfti`.
  **L1030 CN**: 声明 TableGen def `__fixsfti`。
- **L1031 EN**: Declares TableGen def `__fixdfsi`.
  **L1031 CN**: 声明 TableGen def `__fixdfsi`。
- **L1032 EN**: Declares TableGen def `__fixdfdi`.
  **L1032 CN**: 声明 TableGen def `__fixdfdi`。
- **L1033 EN**: Declares TableGen def `__fixdfti`.
  **L1033 CN**: 声明 TableGen def `__fixdfti`。
- **L1034 EN**: Declares TableGen def `__fixxfsi`.
  **L1034 CN**: 声明 TableGen def `__fixxfsi`。
- **L1035 EN**: Declares TableGen def `__fixxfdi`.
  **L1035 CN**: 声明 TableGen def `__fixxfdi`。
- **L1036 EN**: Declares TableGen def `__fixxfti`.
  **L1036 CN**: 声明 TableGen def `__fixxfti`。
- **L1037 EN**: Declares TableGen def `__fixtfsi`.
  **L1037 CN**: 声明 TableGen def `__fixtfsi`。
- **L1038 EN**: Declares TableGen def `__fixtfdi_f128`.
  **L1038 CN**: 声明 TableGen def `__fixtfdi_f128`。
- **L1039 EN**: Declares TableGen def `__fixtfti_f128`.
  **L1039 CN**: 声明 TableGen def `__fixtfti_f128`。
- **L1040 EN**: Declares TableGen def `__gcc_qtou`.
  **L1040 CN**: 声明 TableGen def `__gcc_qtou`。
- **L1041 EN**: Declares TableGen def `__fixtfdi_ppcf128`.
  **L1041 CN**: 声明 TableGen def `__fixtfdi_ppcf128`。
- **L1042 EN**: Declares TableGen def `__fixtfti_ppcf128`.
  **L1042 CN**: 声明 TableGen def `__fixtfti_ppcf128`。
- **L1043 EN**: Declares TableGen def `__fixunshfsi`.
  **L1043 CN**: 声明 TableGen def `__fixunshfsi`。
- **L1044 EN**: Declares TableGen def `__fixunshfdi`.
  **L1044 CN**: 声明 TableGen def `__fixunshfdi`。
- **L1045 EN**: Declares TableGen def `__fixunshfti`.
  **L1045 CN**: 声明 TableGen def `__fixunshfti`。
- **L1046 EN**: Declares TableGen def `__fixunssfsi`.
  **L1046 CN**: 声明 TableGen def `__fixunssfsi`。
- **L1047 EN**: Declares TableGen def `__fixunssfdi`.
  **L1047 CN**: 声明 TableGen def `__fixunssfdi`。
- **L1048 EN**: Declares TableGen def `__fixunssfti`.
  **L1048 CN**: 声明 TableGen def `__fixunssfti`。
- **L1049 EN**: Declares TableGen def `__fixunsdfsi`.
  **L1049 CN**: 声明 TableGen def `__fixunsdfsi`。
- **L1050 EN**: Declares TableGen def `__fixunsdfdi`.
  **L1050 CN**: 声明 TableGen def `__fixunsdfdi`。
- **L1051 EN**: Declares TableGen def `__fixunsdfti`.
  **L1051 CN**: 声明 TableGen def `__fixunsdfti`。
- **L1052 EN**: Declares TableGen def `__fixunsxfsi`.
  **L1052 CN**: 声明 TableGen def `__fixunsxfsi`。
- **L1053 EN**: Declares TableGen def `__fixunsxfdi`.
  **L1053 CN**: 声明 TableGen def `__fixunsxfdi`。
- **L1054 EN**: Declares TableGen def `__fixunsxfti`.
  **L1054 CN**: 声明 TableGen def `__fixunsxfti`。
- **L1055 EN**: Declares TableGen def `__fixunstfsi_f128`.
  **L1055 CN**: 声明 TableGen def `__fixunstfsi_f128`。
- **L1056 EN**: Declares TableGen def `__fixunstfdi_f128`.
  **L1056 CN**: 声明 TableGen def `__fixunstfdi_f128`。

### Lines 1057-1088

````tablegen
def __fixunstfti_f128 : RuntimeLibcallImpl<FPTOUINT_F128_I128, "__fixunstfti">;
def __fixunstfsi_ppcf128 : RuntimeLibcallImpl<FPTOUINT_PPCF128_I32, "__fixunstfsi">;
def __fixunstfdi_ppcf128 : RuntimeLibcallImpl<FPTOUINT_PPCF128_I64, "__fixunstfdi">;
def __fixunstfti_ppcf128 : RuntimeLibcallImpl<FPTOUINT_PPCF128_I128, "__fixunstfti">;
def __floatsihf : RuntimeLibcallImpl<SINTTOFP_I32_F16>;
def __floatsisf : RuntimeLibcallImpl<SINTTOFP_I32_F32>;
def __floatsidf : RuntimeLibcallImpl<SINTTOFP_I32_F64>;
def __floatsixf : RuntimeLibcallImpl<SINTTOFP_I32_F80>;
def __floatsitf : RuntimeLibcallImpl<SINTTOFP_I32_F128>;
def __gcc_itoq : RuntimeLibcallImpl<SINTTOFP_I32_PPCF128>;
def __floatdibf : RuntimeLibcallImpl<SINTTOFP_I64_BF16>;
def __floatdihf : RuntimeLibcallImpl<SINTTOFP_I64_F16>;
def __floatdisf : RuntimeLibcallImpl<SINTTOFP_I64_F32>;
def __floatdidf : RuntimeLibcallImpl<SINTTOFP_I64_F64>;
def __floatdixf : RuntimeLibcallImpl<SINTTOFP_I64_F80>;
def __floatditf_f128 : RuntimeLibcallImpl<SINTTOFP_I64_F128, "__floatditf">;
def __floatditf_ppcf128 : RuntimeLibcallImpl<SINTTOFP_I64_PPCF128, "__floatditf">;
def __floattihf : RuntimeLibcallImpl<SINTTOFP_I128_F16>;
def __floattisf : RuntimeLibcallImpl<SINTTOFP_I128_F32>;
def __floattidf : RuntimeLibcallImpl<SINTTOFP_I128_F64>;
def __floattixf : RuntimeLibcallImpl<SINTTOFP_I128_F80>;
def __floattitf_f128 : RuntimeLibcallImpl<SINTTOFP_I128_F128, "__floattitf">;
def __floattitf_ppcf128 : RuntimeLibcallImpl<SINTTOFP_I128_PPCF128, "__floattitf">;
def __floatunsihf : RuntimeLibcallImpl<UINTTOFP_I32_F16>;
def __floatunsisf : RuntimeLibcallImpl<UINTTOFP_I32_F32>;
def __floatunsidf : RuntimeLibcallImpl<UINTTOFP_I32_F64>;
def __floatunsixf : RuntimeLibcallImpl<UINTTOFP_I32_F80>;
def __floatunsitf : RuntimeLibcallImpl<UINTTOFP_I32_F128>;
def __gcc_utoq : RuntimeLibcallImpl<UINTTOFP_I32_PPCF128>;
def __floatundibf : RuntimeLibcallImpl<UINTTOFP_I64_BF16>;
def __floatundihf : RuntimeLibcallImpl<UINTTOFP_I64_F16>;
def __floatundisf : RuntimeLibcallImpl<UINTTOFP_I64_F32>;
````
- **L1057 EN**: Declares TableGen def `__fixunstfti_f128`.
  **L1057 CN**: 声明 TableGen def `__fixunstfti_f128`。
- **L1058 EN**: Declares TableGen def `__fixunstfsi_ppcf128`.
  **L1058 CN**: 声明 TableGen def `__fixunstfsi_ppcf128`。
- **L1059 EN**: Declares TableGen def `__fixunstfdi_ppcf128`.
  **L1059 CN**: 声明 TableGen def `__fixunstfdi_ppcf128`。
- **L1060 EN**: Declares TableGen def `__fixunstfti_ppcf128`.
  **L1060 CN**: 声明 TableGen def `__fixunstfti_ppcf128`。
- **L1061 EN**: Declares TableGen def `__floatsihf`.
  **L1061 CN**: 声明 TableGen def `__floatsihf`。
- **L1062 EN**: Declares TableGen def `__floatsisf`.
  **L1062 CN**: 声明 TableGen def `__floatsisf`。
- **L1063 EN**: Declares TableGen def `__floatsidf`.
  **L1063 CN**: 声明 TableGen def `__floatsidf`。
- **L1064 EN**: Declares TableGen def `__floatsixf`.
  **L1064 CN**: 声明 TableGen def `__floatsixf`。
- **L1065 EN**: Declares TableGen def `__floatsitf`.
  **L1065 CN**: 声明 TableGen def `__floatsitf`。
- **L1066 EN**: Declares TableGen def `__gcc_itoq`.
  **L1066 CN**: 声明 TableGen def `__gcc_itoq`。
- **L1067 EN**: Declares TableGen def `__floatdibf`.
  **L1067 CN**: 声明 TableGen def `__floatdibf`。
- **L1068 EN**: Declares TableGen def `__floatdihf`.
  **L1068 CN**: 声明 TableGen def `__floatdihf`。
- **L1069 EN**: Declares TableGen def `__floatdisf`.
  **L1069 CN**: 声明 TableGen def `__floatdisf`。
- **L1070 EN**: Declares TableGen def `__floatdidf`.
  **L1070 CN**: 声明 TableGen def `__floatdidf`。
- **L1071 EN**: Declares TableGen def `__floatdixf`.
  **L1071 CN**: 声明 TableGen def `__floatdixf`。
- **L1072 EN**: Declares TableGen def `__floatditf_f128`.
  **L1072 CN**: 声明 TableGen def `__floatditf_f128`。
- **L1073 EN**: Declares TableGen def `__floatditf_ppcf128`.
  **L1073 CN**: 声明 TableGen def `__floatditf_ppcf128`。
- **L1074 EN**: Declares TableGen def `__floattihf`.
  **L1074 CN**: 声明 TableGen def `__floattihf`。
- **L1075 EN**: Declares TableGen def `__floattisf`.
  **L1075 CN**: 声明 TableGen def `__floattisf`。
- **L1076 EN**: Declares TableGen def `__floattidf`.
  **L1076 CN**: 声明 TableGen def `__floattidf`。
- **L1077 EN**: Declares TableGen def `__floattixf`.
  **L1077 CN**: 声明 TableGen def `__floattixf`。
- **L1078 EN**: Declares TableGen def `__floattitf_f128`.
  **L1078 CN**: 声明 TableGen def `__floattitf_f128`。
- **L1079 EN**: Declares TableGen def `__floattitf_ppcf128`.
  **L1079 CN**: 声明 TableGen def `__floattitf_ppcf128`。
- **L1080 EN**: Declares TableGen def `__floatunsihf`.
  **L1080 CN**: 声明 TableGen def `__floatunsihf`。
- **L1081 EN**: Declares TableGen def `__floatunsisf`.
  **L1081 CN**: 声明 TableGen def `__floatunsisf`。
- **L1082 EN**: Declares TableGen def `__floatunsidf`.
  **L1082 CN**: 声明 TableGen def `__floatunsidf`。
- **L1083 EN**: Declares TableGen def `__floatunsixf`.
  **L1083 CN**: 声明 TableGen def `__floatunsixf`。
- **L1084 EN**: Declares TableGen def `__floatunsitf`.
  **L1084 CN**: 声明 TableGen def `__floatunsitf`。
- **L1085 EN**: Declares TableGen def `__gcc_utoq`.
  **L1085 CN**: 声明 TableGen def `__gcc_utoq`。
- **L1086 EN**: Declares TableGen def `__floatundibf`.
  **L1086 CN**: 声明 TableGen def `__floatundibf`。
- **L1087 EN**: Declares TableGen def `__floatundihf`.
  **L1087 CN**: 声明 TableGen def `__floatundihf`。
- **L1088 EN**: Declares TableGen def `__floatundisf`.
  **L1088 CN**: 声明 TableGen def `__floatundisf`。

### Lines 1089-1120

````tablegen
def __floatundidf : RuntimeLibcallImpl<UINTTOFP_I64_F64>;
def __floatundixf : RuntimeLibcallImpl<UINTTOFP_I64_F80>;
def __floatunditf_f128 : RuntimeLibcallImpl<UINTTOFP_I64_F128, "__floatunditf">;
def __floatunditf_ppcf128 : RuntimeLibcallImpl<UINTTOFP_I64_PPCF128, "__floatunditf">;
def __floatuntihf : RuntimeLibcallImpl<UINTTOFP_I128_F16>;
def __floatuntisf : RuntimeLibcallImpl<UINTTOFP_I128_F32>;
def __floatuntidf : RuntimeLibcallImpl<UINTTOFP_I128_F64>;
def __floatuntixf : RuntimeLibcallImpl<UINTTOFP_I128_F80>;
def __floatuntitf_f128 : RuntimeLibcallImpl<UINTTOFP_I128_F128, "__floatuntitf">;
def __floatuntitf_ppcf128 : RuntimeLibcallImpl<UINTTOFP_I128_PPCF128, "__floatuntitf">;
def __extendkftf2 : RuntimeLibcallImpl<CONVERT_F128_PPCF128>;
def __trunctfkf2 : RuntimeLibcallImpl<CONVERT_PPCF128_F128>;

// Comparison
def __eqsf2 : RuntimeLibcallImpl<OEQ_F32>;
def __eqdf2 : RuntimeLibcallImpl<OEQ_F64>;
def __eqtf2 : RuntimeLibcallImpl<OEQ_F128>;
def __gcc_qeq : RuntimeLibcallImpl<OEQ_PPCF128>;
def __nesf2 : RuntimeLibcallImpl<UNE_F32>;
def __nedf2 : RuntimeLibcallImpl<UNE_F64>;
def __netf2 : RuntimeLibcallImpl<UNE_F128>;
def __gcc_qne : RuntimeLibcallImpl<UNE_PPCF128>;
def __gesf2 : RuntimeLibcallImpl<OGE_F32>;
def __gedf2 : RuntimeLibcallImpl<OGE_F64>;
def __getf2 : RuntimeLibcallImpl<OGE_F128>;
def __gcc_qge : RuntimeLibcallImpl<OGE_PPCF128>;
def __ltsf2 : RuntimeLibcallImpl<OLT_F32>;
def __ltdf2 : RuntimeLibcallImpl<OLT_F64>;
def __lttf2 : RuntimeLibcallImpl<OLT_F128>;
def __gcc_qlt : RuntimeLibcallImpl<OLT_PPCF128>;
def __lesf2 : RuntimeLibcallImpl<OLE_F32>;
def __ledf2 : RuntimeLibcallImpl<OLE_F64>;
````
- **L1089 EN**: Declares TableGen def `__floatundidf`.
  **L1089 CN**: 声明 TableGen def `__floatundidf`。
- **L1090 EN**: Declares TableGen def `__floatundixf`.
  **L1090 CN**: 声明 TableGen def `__floatundixf`。
- **L1091 EN**: Declares TableGen def `__floatunditf_f128`.
  **L1091 CN**: 声明 TableGen def `__floatunditf_f128`。
- **L1092 EN**: Declares TableGen def `__floatunditf_ppcf128`.
  **L1092 CN**: 声明 TableGen def `__floatunditf_ppcf128`。
- **L1093 EN**: Declares TableGen def `__floatuntihf`.
  **L1093 CN**: 声明 TableGen def `__floatuntihf`。
- **L1094 EN**: Declares TableGen def `__floatuntisf`.
  **L1094 CN**: 声明 TableGen def `__floatuntisf`。
- **L1095 EN**: Declares TableGen def `__floatuntidf`.
  **L1095 CN**: 声明 TableGen def `__floatuntidf`。
- **L1096 EN**: Declares TableGen def `__floatuntixf`.
  **L1096 CN**: 声明 TableGen def `__floatuntixf`。
- **L1097 EN**: Declares TableGen def `__floatuntitf_f128`.
  **L1097 CN**: 声明 TableGen def `__floatuntitf_f128`。
- **L1098 EN**: Declares TableGen def `__floatuntitf_ppcf128`.
  **L1098 CN**: 声明 TableGen def `__floatuntitf_ppcf128`。
- **L1099 EN**: Declares TableGen def `__extendkftf2`.
  **L1099 CN**: 声明 TableGen def `__extendkftf2`。
- **L1100 EN**: Declares TableGen def `__trunctfkf2`.
  **L1100 CN**: 声明 TableGen def `__trunctfkf2`。
- **L1101 EN**: Blank line separating nearby declarations or logic blocks.
  **L1101 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1102 EN**: Comment explains nearby logic, invariants, or intent: `Comparison`.
  **L1102 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Comparison`。
- **L1103 EN**: Declares TableGen def `__eqsf2`.
  **L1103 CN**: 声明 TableGen def `__eqsf2`。
- **L1104 EN**: Declares TableGen def `__eqdf2`.
  **L1104 CN**: 声明 TableGen def `__eqdf2`。
- **L1105 EN**: Declares TableGen def `__eqtf2`.
  **L1105 CN**: 声明 TableGen def `__eqtf2`。
- **L1106 EN**: Declares TableGen def `__gcc_qeq`.
  **L1106 CN**: 声明 TableGen def `__gcc_qeq`。
- **L1107 EN**: Declares TableGen def `__nesf2`.
  **L1107 CN**: 声明 TableGen def `__nesf2`。
- **L1108 EN**: Declares TableGen def `__nedf2`.
  **L1108 CN**: 声明 TableGen def `__nedf2`。
- **L1109 EN**: Declares TableGen def `__netf2`.
  **L1109 CN**: 声明 TableGen def `__netf2`。
- **L1110 EN**: Declares TableGen def `__gcc_qne`.
  **L1110 CN**: 声明 TableGen def `__gcc_qne`。
- **L1111 EN**: Declares TableGen def `__gesf2`.
  **L1111 CN**: 声明 TableGen def `__gesf2`。
- **L1112 EN**: Declares TableGen def `__gedf2`.
  **L1112 CN**: 声明 TableGen def `__gedf2`。
- **L1113 EN**: Declares TableGen def `__getf2`.
  **L1113 CN**: 声明 TableGen def `__getf2`。
- **L1114 EN**: Declares TableGen def `__gcc_qge`.
  **L1114 CN**: 声明 TableGen def `__gcc_qge`。
- **L1115 EN**: Declares TableGen def `__ltsf2`.
  **L1115 CN**: 声明 TableGen def `__ltsf2`。
- **L1116 EN**: Declares TableGen def `__ltdf2`.
  **L1116 CN**: 声明 TableGen def `__ltdf2`。
- **L1117 EN**: Declares TableGen def `__lttf2`.
  **L1117 CN**: 声明 TableGen def `__lttf2`。
- **L1118 EN**: Declares TableGen def `__gcc_qlt`.
  **L1118 CN**: 声明 TableGen def `__gcc_qlt`。
- **L1119 EN**: Declares TableGen def `__lesf2`.
  **L1119 CN**: 声明 TableGen def `__lesf2`。
- **L1120 EN**: Declares TableGen def `__ledf2`.
  **L1120 CN**: 声明 TableGen def `__ledf2`。

### Lines 1121-1152

````tablegen
def __letf2 : RuntimeLibcallImpl<OLE_F128>;
def __gcc_qle : RuntimeLibcallImpl<OLE_PPCF128>;
def __gtsf2 : RuntimeLibcallImpl<OGT_F32>;
def __gtdf2 : RuntimeLibcallImpl<OGT_F64>;
def __gttf2 : RuntimeLibcallImpl<OGT_F128>;
def __gcc_qgt : RuntimeLibcallImpl<OGT_PPCF128>;
def __unordsf2 : RuntimeLibcallImpl<UO_F32>;
def __unorddf2 : RuntimeLibcallImpl<UO_F64>;
def __unordtf2 : RuntimeLibcallImpl<UO_F128>;
def __gcc_qunord : RuntimeLibcallImpl<UO_PPCF128>;

// Element-wise unordered-atomic memory of different sizes
foreach MemSize = [ 1, 2, 4, 8, 16 ] in {
  def __llvm_memcpy_element_unordered_atomic_#MemSize : RuntimeLibcallImpl<
    !cast<RuntimeLibcall>("MEMCPY_ELEMENT_UNORDERED_ATOMIC_"#MemSize)>;

  def __llvm_memmove_element_unordered_atomic_#MemSize : RuntimeLibcallImpl<
    !cast<RuntimeLibcall>("MEMMOVE_ELEMENT_UNORDERED_ATOMIC_"#MemSize)>;

  def __llvm_memset_element_unordered_atomic_#MemSize : RuntimeLibcallImpl<
    !cast<RuntimeLibcall>("MEMSET_ELEMENT_UNORDERED_ATOMIC_"#MemSize)>;
}

// Atomic '__sync_*' libcalls.
foreach lc = LibCalls__sync in {
  def __#!tolower(!cast<string>(lc)) : RuntimeLibcallImpl<lc>;
}

// Atomic `__atomic_*' libcalls.
foreach lc = LibCalls__atomic in {
  def __#!tolower(!cast<string>(lc)) : RuntimeLibcallImpl<lc>;
}
````
- **L1121 EN**: Declares TableGen def `__letf2`.
  **L1121 CN**: 声明 TableGen def `__letf2`。
- **L1122 EN**: Declares TableGen def `__gcc_qle`.
  **L1122 CN**: 声明 TableGen def `__gcc_qle`。
- **L1123 EN**: Declares TableGen def `__gtsf2`.
  **L1123 CN**: 声明 TableGen def `__gtsf2`。
- **L1124 EN**: Declares TableGen def `__gtdf2`.
  **L1124 CN**: 声明 TableGen def `__gtdf2`。
- **L1125 EN**: Declares TableGen def `__gttf2`.
  **L1125 CN**: 声明 TableGen def `__gttf2`。
- **L1126 EN**: Declares TableGen def `__gcc_qgt`.
  **L1126 CN**: 声明 TableGen def `__gcc_qgt`。
- **L1127 EN**: Declares TableGen def `__unordsf2`.
  **L1127 CN**: 声明 TableGen def `__unordsf2`。
- **L1128 EN**: Declares TableGen def `__unorddf2`.
  **L1128 CN**: 声明 TableGen def `__unorddf2`。
- **L1129 EN**: Declares TableGen def `__unordtf2`.
  **L1129 CN**: 声明 TableGen def `__unordtf2`。
- **L1130 EN**: Declares TableGen def `__gcc_qunord`.
  **L1130 CN**: 声明 TableGen def `__gcc_qunord`。
- **L1131 EN**: Blank line separating nearby declarations or logic blocks.
  **L1131 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1132 EN**: Comment explains nearby logic, invariants, or intent: `Element-wise unordered-atomic memory of different sizes`.
  **L1132 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Element-wise unordered-atomic memory of different sizes`。
- **L1133 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L1133 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1134 EN**: Declares TableGen def `__llvm_memcpy_element_unordered_atomic_#MemSize`.
  **L1134 CN**: 声明 TableGen def `__llvm_memcpy_element_unordered_atomic_#MemSize`。
- **L1135 EN**: Executes a call or declaration centered on `!cast<RuntimeLibcall>`.
  **L1135 CN**: 执行以 `!cast<RuntimeLibcall>` 为核心的调用或声明。
- **L1136 EN**: Blank line separating nearby declarations or logic blocks.
  **L1136 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1137 EN**: Declares TableGen def `__llvm_memmove_element_unordered_atomic_#MemSize`.
  **L1137 CN**: 声明 TableGen def `__llvm_memmove_element_unordered_atomic_#MemSize`。
- **L1138 EN**: Executes a call or declaration centered on `!cast<RuntimeLibcall>`.
  **L1138 CN**: 执行以 `!cast<RuntimeLibcall>` 为核心的调用或声明。
- **L1139 EN**: Blank line separating nearby declarations or logic blocks.
  **L1139 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1140 EN**: Declares TableGen def `__llvm_memset_element_unordered_atomic_#MemSize`.
  **L1140 CN**: 声明 TableGen def `__llvm_memset_element_unordered_atomic_#MemSize`。
- **L1141 EN**: Executes a call or declaration centered on `!cast<RuntimeLibcall>`.
  **L1141 CN**: 执行以 `!cast<RuntimeLibcall>` 为核心的调用或声明。
- **L1142 EN**: Closes the current lexical scope or compound statement.
  **L1142 CN**: 结束当前词法作用域或复合语句块。
- **L1143 EN**: Blank line separating nearby declarations or logic blocks.
  **L1143 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1144 EN**: Comment explains nearby logic, invariants, or intent: `Atomic '__sync_*' libcalls.`.
  **L1144 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Atomic '__sync_*' libcalls.`。
- **L1145 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L1145 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1146 EN**: Declares TableGen def `__#!tolower(!cast`.
  **L1146 CN**: 声明 TableGen def `__#!tolower(!cast`。
- **L1147 EN**: Closes the current lexical scope or compound statement.
  **L1147 CN**: 结束当前词法作用域或复合语句块。
- **L1148 EN**: Blank line separating nearby declarations or logic blocks.
  **L1148 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1149 EN**: Comment explains nearby logic, invariants, or intent: `Atomic `__atomic_*' libcalls.`.
  **L1149 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Atomic `__atomic_*' libcalls.`。
- **L1150 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L1150 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1151 EN**: Declares TableGen def `__#!tolower(!cast`.
  **L1151 CN**: 声明 TableGen def `__#!tolower(!cast`。
- **L1152 EN**: Closes the current lexical scope or compound statement.
  **L1152 CN**: 结束当前词法作用域或复合语句块。

### Lines 1153-1184

````tablegen

// Deoptimization
def __llvm_deoptimize : RuntimeLibcallImpl<DEOPTIMIZE>;

// Clear cache
def __clear_cache : RuntimeLibcallImpl<CLEAR_CACHE>;

//--------------------------------------------------------------------
// libm
//--------------------------------------------------------------------

def fmodf : RuntimeLibcallImpl<REM_F32>;
def fmod : RuntimeLibcallImpl<REM_F64>;
def fmodl_f128 : RuntimeLibcallImpl<REM_F128, "fmodl">;
def fmodl_f80 : RuntimeLibcallImpl<REM_F80, "fmodl">;
def fmodl_ppc128 : RuntimeLibcallImpl<REM_PPCF128, "fmodl">;

def fmaf : RuntimeLibcallImpl<FMA_F32>;
def fma : RuntimeLibcallImpl<FMA_F64>;
defm fmal : LibmLongDoubleLibCall;

def sqrtf : RuntimeLibcallImpl<SQRT_F32>;
def sqrt : RuntimeLibcallImpl<SQRT_F64>;
defm sqrtl : LibmLongDoubleLibCall;

def cbrtf : RuntimeLibcallImpl<CBRT_F32>;
def cbrt : RuntimeLibcallImpl<CBRT_F64>;
defm cbrtl : LibmLongDoubleLibCall;

def logf : RuntimeLibcallImpl<LOG_F32>;
def log : RuntimeLibcallImpl<LOG_F64>;
defm logl : LibmLongDoubleLibCall;
````
- **L1153 EN**: Blank line separating nearby declarations or logic blocks.
  **L1153 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1154 EN**: Comment explains nearby logic, invariants, or intent: `Deoptimization`.
  **L1154 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Deoptimization`。
- **L1155 EN**: Declares TableGen def `__llvm_deoptimize`.
  **L1155 CN**: 声明 TableGen def `__llvm_deoptimize`。
- **L1156 EN**: Blank line separating nearby declarations or logic blocks.
  **L1156 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1157 EN**: Comment explains nearby logic, invariants, or intent: `Clear cache`.
  **L1157 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Clear cache`。
- **L1158 EN**: Declares TableGen def `__clear_cache`.
  **L1158 CN**: 声明 TableGen def `__clear_cache`。
- **L1159 EN**: Blank line separating nearby declarations or logic blocks.
  **L1159 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1160 EN**: Comment explains nearby logic, invariants, or intent: `--------------------------------------------------------------------`.
  **L1160 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`--------------------------------------------------------------------`。
- **L1161 EN**: Comment explains nearby logic, invariants, or intent: `libm`.
  **L1161 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`libm`。
- **L1162 EN**: Comment explains nearby logic, invariants, or intent: `--------------------------------------------------------------------`.
  **L1162 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`--------------------------------------------------------------------`。
- **L1163 EN**: Blank line separating nearby declarations or logic blocks.
  **L1163 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1164 EN**: Declares TableGen def `fmodf`.
  **L1164 CN**: 声明 TableGen def `fmodf`。
- **L1165 EN**: Declares TableGen def `fmod`.
  **L1165 CN**: 声明 TableGen def `fmod`。
- **L1166 EN**: Declares TableGen def `fmodl_f128`.
  **L1166 CN**: 声明 TableGen def `fmodl_f128`。
- **L1167 EN**: Declares TableGen def `fmodl_f80`.
  **L1167 CN**: 声明 TableGen def `fmodl_f80`。
- **L1168 EN**: Declares TableGen def `fmodl_ppc128`.
  **L1168 CN**: 声明 TableGen def `fmodl_ppc128`。
- **L1169 EN**: Blank line separating nearby declarations or logic blocks.
  **L1169 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1170 EN**: Declares TableGen def `fmaf`.
  **L1170 CN**: 声明 TableGen def `fmaf`。
- **L1171 EN**: Declares TableGen def `fma`.
  **L1171 CN**: 声明 TableGen def `fma`。
- **L1172 EN**: Declares TableGen defm `fmal`.
  **L1172 CN**: 声明 TableGen defm `fmal`。
- **L1173 EN**: Blank line separating nearby declarations or logic blocks.
  **L1173 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1174 EN**: Declares TableGen def `sqrtf`.
  **L1174 CN**: 声明 TableGen def `sqrtf`。
- **L1175 EN**: Declares TableGen def `sqrt`.
  **L1175 CN**: 声明 TableGen def `sqrt`。
- **L1176 EN**: Declares TableGen defm `sqrtl`.
  **L1176 CN**: 声明 TableGen defm `sqrtl`。
- **L1177 EN**: Blank line separating nearby declarations or logic blocks.
  **L1177 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1178 EN**: Declares TableGen def `cbrtf`.
  **L1178 CN**: 声明 TableGen def `cbrtf`。
- **L1179 EN**: Declares TableGen def `cbrt`.
  **L1179 CN**: 声明 TableGen def `cbrt`。
- **L1180 EN**: Declares TableGen defm `cbrtl`.
  **L1180 CN**: 声明 TableGen defm `cbrtl`。
- **L1181 EN**: Blank line separating nearby declarations or logic blocks.
  **L1181 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1182 EN**: Declares TableGen def `logf`.
  **L1182 CN**: 声明 TableGen def `logf`。
- **L1183 EN**: Declares TableGen def `log`.
  **L1183 CN**: 声明 TableGen def `log`。
- **L1184 EN**: Declares TableGen defm `logl`.
  **L1184 CN**: 声明 TableGen defm `logl`。

### Lines 1185-1216

````tablegen

def __logf_finite : RuntimeLibcallImpl<LOG_FINITE_F32>;
def __log_finite : RuntimeLibcallImpl<LOG_FINITE_F64>;
def __logl_finite_f80 : RuntimeLibcallImpl<LOG_FINITE_F80, "__logl_finite">;
def __logl_finite_f128 : RuntimeLibcallImpl<LOG_FINITE_F128, "__logl_finite">;
def __logl_finite_ppcf128 : RuntimeLibcallImpl<LOG_FINITE_PPCF128, "__logl_finite">;

def log2f : RuntimeLibcallImpl<LOG2_F32>;
def log2 : RuntimeLibcallImpl<LOG2_F64>;
defm log2l : LibmLongDoubleLibCall;

def __log2f_finite : RuntimeLibcallImpl<LOG2_FINITE_F32>;
def __log2_finite : RuntimeLibcallImpl<LOG2_FINITE_F64>;
def __log2l_finite_f80 : RuntimeLibcallImpl<LOG2_FINITE_F80, "__log2l_finite">;
def __log2l_finite_f128 : RuntimeLibcallImpl<LOG2_FINITE_F128, "__log2l_finite">;
def __log2l_finite_ppcf128 : RuntimeLibcallImpl<LOG2_FINITE_PPCF128, "__log2l_finite">;

def log10f : RuntimeLibcallImpl<LOG10_F32>;
def log10 : RuntimeLibcallImpl<LOG10_F64>;
defm log10l : LibmLongDoubleLibCall;

def __log10f_finite : RuntimeLibcallImpl<LOG10_FINITE_F32>;
def __log10_finite : RuntimeLibcallImpl<LOG10_FINITE_F64>;
def __log10l_finite_f80 : RuntimeLibcallImpl<LOG10_FINITE_F80, "__log10l_finite">;
def __log10l_finite_f128 : RuntimeLibcallImpl<LOG10_FINITE_F128, "__log10l_finite">;
def __log10l_finite_ppcf128 : RuntimeLibcallImpl<LOG10_FINITE_PPCF128, "__log10l_finite">;

def expf : RuntimeLibcallImpl<EXP_F32>;
def exp : RuntimeLibcallImpl<EXP_F64>;
defm expl : LibmLongDoubleLibCall<"EXP">;

def __expf_finite : RuntimeLibcallImpl<EXP_FINITE_F32>;
````
- **L1185 EN**: Blank line separating nearby declarations or logic blocks.
  **L1185 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1186 EN**: Declares TableGen def `__logf_finite`.
  **L1186 CN**: 声明 TableGen def `__logf_finite`。
- **L1187 EN**: Declares TableGen def `__log_finite`.
  **L1187 CN**: 声明 TableGen def `__log_finite`。
- **L1188 EN**: Declares TableGen def `__logl_finite_f80`.
  **L1188 CN**: 声明 TableGen def `__logl_finite_f80`。
- **L1189 EN**: Declares TableGen def `__logl_finite_f128`.
  **L1189 CN**: 声明 TableGen def `__logl_finite_f128`。
- **L1190 EN**: Declares TableGen def `__logl_finite_ppcf128`.
  **L1190 CN**: 声明 TableGen def `__logl_finite_ppcf128`。
- **L1191 EN**: Blank line separating nearby declarations or logic blocks.
  **L1191 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1192 EN**: Declares TableGen def `log2f`.
  **L1192 CN**: 声明 TableGen def `log2f`。
- **L1193 EN**: Declares TableGen def `log2`.
  **L1193 CN**: 声明 TableGen def `log2`。
- **L1194 EN**: Declares TableGen defm `log2l`.
  **L1194 CN**: 声明 TableGen defm `log2l`。
- **L1195 EN**: Blank line separating nearby declarations or logic blocks.
  **L1195 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1196 EN**: Declares TableGen def `__log2f_finite`.
  **L1196 CN**: 声明 TableGen def `__log2f_finite`。
- **L1197 EN**: Declares TableGen def `__log2_finite`.
  **L1197 CN**: 声明 TableGen def `__log2_finite`。
- **L1198 EN**: Declares TableGen def `__log2l_finite_f80`.
  **L1198 CN**: 声明 TableGen def `__log2l_finite_f80`。
- **L1199 EN**: Declares TableGen def `__log2l_finite_f128`.
  **L1199 CN**: 声明 TableGen def `__log2l_finite_f128`。
- **L1200 EN**: Declares TableGen def `__log2l_finite_ppcf128`.
  **L1200 CN**: 声明 TableGen def `__log2l_finite_ppcf128`。
- **L1201 EN**: Blank line separating nearby declarations or logic blocks.
  **L1201 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1202 EN**: Declares TableGen def `log10f`.
  **L1202 CN**: 声明 TableGen def `log10f`。
- **L1203 EN**: Declares TableGen def `log10`.
  **L1203 CN**: 声明 TableGen def `log10`。
- **L1204 EN**: Declares TableGen defm `log10l`.
  **L1204 CN**: 声明 TableGen defm `log10l`。
- **L1205 EN**: Blank line separating nearby declarations or logic blocks.
  **L1205 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1206 EN**: Declares TableGen def `__log10f_finite`.
  **L1206 CN**: 声明 TableGen def `__log10f_finite`。
- **L1207 EN**: Declares TableGen def `__log10_finite`.
  **L1207 CN**: 声明 TableGen def `__log10_finite`。
- **L1208 EN**: Declares TableGen def `__log10l_finite_f80`.
  **L1208 CN**: 声明 TableGen def `__log10l_finite_f80`。
- **L1209 EN**: Declares TableGen def `__log10l_finite_f128`.
  **L1209 CN**: 声明 TableGen def `__log10l_finite_f128`。
- **L1210 EN**: Declares TableGen def `__log10l_finite_ppcf128`.
  **L1210 CN**: 声明 TableGen def `__log10l_finite_ppcf128`。
- **L1211 EN**: Blank line separating nearby declarations or logic blocks.
  **L1211 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1212 EN**: Declares TableGen def `expf`.
  **L1212 CN**: 声明 TableGen def `expf`。
- **L1213 EN**: Declares TableGen def `exp`.
  **L1213 CN**: 声明 TableGen def `exp`。
- **L1214 EN**: Declares TableGen defm `expl`.
  **L1214 CN**: 声明 TableGen defm `expl`。
- **L1215 EN**: Blank line separating nearby declarations or logic blocks.
  **L1215 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1216 EN**: Declares TableGen def `__expf_finite`.
  **L1216 CN**: 声明 TableGen def `__expf_finite`。

### Lines 1217-1248

````tablegen
def __exp_finite : RuntimeLibcallImpl<EXP_FINITE_F64>;
def __expl_finite_f80 : RuntimeLibcallImpl<EXP_FINITE_F80, "__expl_finite">;
def __expl_finite_f128 : RuntimeLibcallImpl<EXP_FINITE_F128, "__expl_finite">;
def __expl_finite_ppcf128 : RuntimeLibcallImpl<EXP_FINITE_PPCF128, "__expl_finite">;

def exp2f : RuntimeLibcallImpl<EXP2_F32>;
def exp2 : RuntimeLibcallImpl<EXP2_F64>;
defm exp2l : LibmLongDoubleLibCall<"EXP2">;

def __exp2f_finite : RuntimeLibcallImpl<EXP2_FINITE_F32>;
def __exp2_finite : RuntimeLibcallImpl<EXP2_FINITE_F64>;
def __exp2l_finite_f80 : RuntimeLibcallImpl<EXP2_FINITE_F80, "__exp2l_finite">;
def __exp2l_finite_f128 : RuntimeLibcallImpl<EXP2_FINITE_F128, "__exp2l_finite">;
def __exp2l_finite_ppcf128 : RuntimeLibcallImpl<EXP2_FINITE_PPCF128, "__exp2l_finite">;

def sinf : RuntimeLibcallImpl<SIN_F32>;
def sin : RuntimeLibcallImpl<SIN_F64>;
defm sinl : LibmLongDoubleLibCall;

def cosf : RuntimeLibcallImpl<COS_F32>;
def cos : RuntimeLibcallImpl<COS_F64>;
defm cosl : LibmLongDoubleLibCall;

def tanf : RuntimeLibcallImpl<TAN_F32>;
def tan : RuntimeLibcallImpl<TAN_F64>;
defm tanl : LibmLongDoubleLibCall;

def sinhf : RuntimeLibcallImpl<SINH_F32>;
def sinh : RuntimeLibcallImpl<SINH_F64>;
defm sinhl : LibmLongDoubleLibCall;

def coshf : RuntimeLibcallImpl<COSH_F32>;
````
- **L1217 EN**: Declares TableGen def `__exp_finite`.
  **L1217 CN**: 声明 TableGen def `__exp_finite`。
- **L1218 EN**: Declares TableGen def `__expl_finite_f80`.
  **L1218 CN**: 声明 TableGen def `__expl_finite_f80`。
- **L1219 EN**: Declares TableGen def `__expl_finite_f128`.
  **L1219 CN**: 声明 TableGen def `__expl_finite_f128`。
- **L1220 EN**: Declares TableGen def `__expl_finite_ppcf128`.
  **L1220 CN**: 声明 TableGen def `__expl_finite_ppcf128`。
- **L1221 EN**: Blank line separating nearby declarations or logic blocks.
  **L1221 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1222 EN**: Declares TableGen def `exp2f`.
  **L1222 CN**: 声明 TableGen def `exp2f`。
- **L1223 EN**: Declares TableGen def `exp2`.
  **L1223 CN**: 声明 TableGen def `exp2`。
- **L1224 EN**: Declares TableGen defm `exp2l`.
  **L1224 CN**: 声明 TableGen defm `exp2l`。
- **L1225 EN**: Blank line separating nearby declarations or logic blocks.
  **L1225 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1226 EN**: Declares TableGen def `__exp2f_finite`.
  **L1226 CN**: 声明 TableGen def `__exp2f_finite`。
- **L1227 EN**: Declares TableGen def `__exp2_finite`.
  **L1227 CN**: 声明 TableGen def `__exp2_finite`。
- **L1228 EN**: Declares TableGen def `__exp2l_finite_f80`.
  **L1228 CN**: 声明 TableGen def `__exp2l_finite_f80`。
- **L1229 EN**: Declares TableGen def `__exp2l_finite_f128`.
  **L1229 CN**: 声明 TableGen def `__exp2l_finite_f128`。
- **L1230 EN**: Declares TableGen def `__exp2l_finite_ppcf128`.
  **L1230 CN**: 声明 TableGen def `__exp2l_finite_ppcf128`。
- **L1231 EN**: Blank line separating nearby declarations or logic blocks.
  **L1231 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1232 EN**: Declares TableGen def `sinf`.
  **L1232 CN**: 声明 TableGen def `sinf`。
- **L1233 EN**: Declares TableGen def `sin`.
  **L1233 CN**: 声明 TableGen def `sin`。
- **L1234 EN**: Declares TableGen defm `sinl`.
  **L1234 CN**: 声明 TableGen defm `sinl`。
- **L1235 EN**: Blank line separating nearby declarations or logic blocks.
  **L1235 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1236 EN**: Declares TableGen def `cosf`.
  **L1236 CN**: 声明 TableGen def `cosf`。
- **L1237 EN**: Declares TableGen def `cos`.
  **L1237 CN**: 声明 TableGen def `cos`。
- **L1238 EN**: Declares TableGen defm `cosl`.
  **L1238 CN**: 声明 TableGen defm `cosl`。
- **L1239 EN**: Blank line separating nearby declarations or logic blocks.
  **L1239 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1240 EN**: Declares TableGen def `tanf`.
  **L1240 CN**: 声明 TableGen def `tanf`。
- **L1241 EN**: Declares TableGen def `tan`.
  **L1241 CN**: 声明 TableGen def `tan`。
- **L1242 EN**: Declares TableGen defm `tanl`.
  **L1242 CN**: 声明 TableGen defm `tanl`。
- **L1243 EN**: Blank line separating nearby declarations or logic blocks.
  **L1243 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1244 EN**: Declares TableGen def `sinhf`.
  **L1244 CN**: 声明 TableGen def `sinhf`。
- **L1245 EN**: Declares TableGen def `sinh`.
  **L1245 CN**: 声明 TableGen def `sinh`。
- **L1246 EN**: Declares TableGen defm `sinhl`.
  **L1246 CN**: 声明 TableGen defm `sinhl`。
- **L1247 EN**: Blank line separating nearby declarations or logic blocks.
  **L1247 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1248 EN**: Declares TableGen def `coshf`.
  **L1248 CN**: 声明 TableGen def `coshf`。

### Lines 1249-1280

````tablegen
def cosh : RuntimeLibcallImpl<COSH_F64>;
defm coshl : LibmLongDoubleLibCall;

def tanhf : RuntimeLibcallImpl<TANH_F32>;
def tanh : RuntimeLibcallImpl<TANH_F64>;
defm tanhl : LibmLongDoubleLibCall;

def asinf : RuntimeLibcallImpl<ASIN_F32>;
def asin : RuntimeLibcallImpl<ASIN_F64>;
defm asinl : LibmLongDoubleLibCall;

def acosf : RuntimeLibcallImpl<ACOS_F32>;
def acos : RuntimeLibcallImpl<ACOS_F64>;
defm acosl : LibmLongDoubleLibCall;

def atanf : RuntimeLibcallImpl<ATAN_F32>;
def atan : RuntimeLibcallImpl<ATAN_F64>;
defm atanl : LibmLongDoubleLibCall;

def atan2f : RuntimeLibcallImpl<ATAN2_F32>;
def atan2 : RuntimeLibcallImpl<ATAN2_F64>;
defm atan2l : LibmLongDoubleLibCall;

def powf : RuntimeLibcallImpl<POW_F32>;
def pow : RuntimeLibcallImpl<POW_F64>;
defm powl : LibmLongDoubleLibCall;

def __powf_finite : RuntimeLibcallImpl<POW_FINITE_F32>;
def __pow_finite : RuntimeLibcallImpl<POW_FINITE_F64>;
def __powl_finite_f80 : RuntimeLibcallImpl<POW_FINITE_F80, "__powl_finite">;
def __powl_finite_f128 : RuntimeLibcallImpl<POW_FINITE_F128, "__powl_finite">;
def __powl_finite_ppcf128 : RuntimeLibcallImpl<POW_FINITE_PPCF128, "__powl_finite">;
````
- **L1249 EN**: Declares TableGen def `cosh`.
  **L1249 CN**: 声明 TableGen def `cosh`。
- **L1250 EN**: Declares TableGen defm `coshl`.
  **L1250 CN**: 声明 TableGen defm `coshl`。
- **L1251 EN**: Blank line separating nearby declarations or logic blocks.
  **L1251 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1252 EN**: Declares TableGen def `tanhf`.
  **L1252 CN**: 声明 TableGen def `tanhf`。
- **L1253 EN**: Declares TableGen def `tanh`.
  **L1253 CN**: 声明 TableGen def `tanh`。
- **L1254 EN**: Declares TableGen defm `tanhl`.
  **L1254 CN**: 声明 TableGen defm `tanhl`。
- **L1255 EN**: Blank line separating nearby declarations or logic blocks.
  **L1255 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1256 EN**: Declares TableGen def `asinf`.
  **L1256 CN**: 声明 TableGen def `asinf`。
- **L1257 EN**: Declares TableGen def `asin`.
  **L1257 CN**: 声明 TableGen def `asin`。
- **L1258 EN**: Declares TableGen defm `asinl`.
  **L1258 CN**: 声明 TableGen defm `asinl`。
- **L1259 EN**: Blank line separating nearby declarations or logic blocks.
  **L1259 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1260 EN**: Declares TableGen def `acosf`.
  **L1260 CN**: 声明 TableGen def `acosf`。
- **L1261 EN**: Declares TableGen def `acos`.
  **L1261 CN**: 声明 TableGen def `acos`。
- **L1262 EN**: Declares TableGen defm `acosl`.
  **L1262 CN**: 声明 TableGen defm `acosl`。
- **L1263 EN**: Blank line separating nearby declarations or logic blocks.
  **L1263 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1264 EN**: Declares TableGen def `atanf`.
  **L1264 CN**: 声明 TableGen def `atanf`。
- **L1265 EN**: Declares TableGen def `atan`.
  **L1265 CN**: 声明 TableGen def `atan`。
- **L1266 EN**: Declares TableGen defm `atanl`.
  **L1266 CN**: 声明 TableGen defm `atanl`。
- **L1267 EN**: Blank line separating nearby declarations or logic blocks.
  **L1267 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1268 EN**: Declares TableGen def `atan2f`.
  **L1268 CN**: 声明 TableGen def `atan2f`。
- **L1269 EN**: Declares TableGen def `atan2`.
  **L1269 CN**: 声明 TableGen def `atan2`。
- **L1270 EN**: Declares TableGen defm `atan2l`.
  **L1270 CN**: 声明 TableGen defm `atan2l`。
- **L1271 EN**: Blank line separating nearby declarations or logic blocks.
  **L1271 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1272 EN**: Declares TableGen def `powf`.
  **L1272 CN**: 声明 TableGen def `powf`。
- **L1273 EN**: Declares TableGen def `pow`.
  **L1273 CN**: 声明 TableGen def `pow`。
- **L1274 EN**: Declares TableGen defm `powl`.
  **L1274 CN**: 声明 TableGen defm `powl`。
- **L1275 EN**: Blank line separating nearby declarations or logic blocks.
  **L1275 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1276 EN**: Declares TableGen def `__powf_finite`.
  **L1276 CN**: 声明 TableGen def `__powf_finite`。
- **L1277 EN**: Declares TableGen def `__pow_finite`.
  **L1277 CN**: 声明 TableGen def `__pow_finite`。
- **L1278 EN**: Declares TableGen def `__powl_finite_f80`.
  **L1278 CN**: 声明 TableGen def `__powl_finite_f80`。
- **L1279 EN**: Declares TableGen def `__powl_finite_f128`.
  **L1279 CN**: 声明 TableGen def `__powl_finite_f128`。
- **L1280 EN**: Declares TableGen def `__powl_finite_ppcf128`.
  **L1280 CN**: 声明 TableGen def `__powl_finite_ppcf128`。

### Lines 1281-1312

````tablegen

def ceilf : RuntimeLibcallImpl<CEIL_F32>;
def ceil : RuntimeLibcallImpl<CEIL_F64>;
defm ceill : LibmLongDoubleLibCall;

def truncf : RuntimeLibcallImpl<TRUNC_F32>;
def trunc : RuntimeLibcallImpl<TRUNC_F64>;
defm truncl : LibmLongDoubleLibCall;

def rintf : RuntimeLibcallImpl<RINT_F32>;
def rint : RuntimeLibcallImpl<RINT_F64>;
defm rintl : LibmLongDoubleLibCall;

def nearbyintf : RuntimeLibcallImpl<NEARBYINT_F32>;
def nearbyint : RuntimeLibcallImpl<NEARBYINT_F64>;
defm nearbyintl : LibmLongDoubleLibCall;

def roundf : RuntimeLibcallImpl<ROUND_F32>;
def round : RuntimeLibcallImpl<ROUND_F64>;
defm roundl : LibmLongDoubleLibCall;

def roundevenf : RuntimeLibcallImpl<ROUNDEVEN_F32>;
def roundeven : RuntimeLibcallImpl<ROUNDEVEN_F64>;
defm roundevenl : LibmLongDoubleLibCall;

def floorf : RuntimeLibcallImpl<FLOOR_F32>;
def floor : RuntimeLibcallImpl<FLOOR_F64>;
defm floorl : LibmLongDoubleLibCall;

def copysignf : RuntimeLibcallImpl<COPYSIGN_F32>;
def copysign : RuntimeLibcallImpl<COPYSIGN_F64>;
defm copysignl : LibmLongDoubleLibCall;
````
- **L1281 EN**: Blank line separating nearby declarations or logic blocks.
  **L1281 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1282 EN**: Declares TableGen def `ceilf`.
  **L1282 CN**: 声明 TableGen def `ceilf`。
- **L1283 EN**: Declares TableGen def `ceil`.
  **L1283 CN**: 声明 TableGen def `ceil`。
- **L1284 EN**: Declares TableGen defm `ceill`.
  **L1284 CN**: 声明 TableGen defm `ceill`。
- **L1285 EN**: Blank line separating nearby declarations or logic blocks.
  **L1285 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1286 EN**: Declares TableGen def `truncf`.
  **L1286 CN**: 声明 TableGen def `truncf`。
- **L1287 EN**: Declares TableGen def `trunc`.
  **L1287 CN**: 声明 TableGen def `trunc`。
- **L1288 EN**: Declares TableGen defm `truncl`.
  **L1288 CN**: 声明 TableGen defm `truncl`。
- **L1289 EN**: Blank line separating nearby declarations or logic blocks.
  **L1289 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1290 EN**: Declares TableGen def `rintf`.
  **L1290 CN**: 声明 TableGen def `rintf`。
- **L1291 EN**: Declares TableGen def `rint`.
  **L1291 CN**: 声明 TableGen def `rint`。
- **L1292 EN**: Declares TableGen defm `rintl`.
  **L1292 CN**: 声明 TableGen defm `rintl`。
- **L1293 EN**: Blank line separating nearby declarations or logic blocks.
  **L1293 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1294 EN**: Declares TableGen def `nearbyintf`.
  **L1294 CN**: 声明 TableGen def `nearbyintf`。
- **L1295 EN**: Declares TableGen def `nearbyint`.
  **L1295 CN**: 声明 TableGen def `nearbyint`。
- **L1296 EN**: Declares TableGen defm `nearbyintl`.
  **L1296 CN**: 声明 TableGen defm `nearbyintl`。
- **L1297 EN**: Blank line separating nearby declarations or logic blocks.
  **L1297 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1298 EN**: Declares TableGen def `roundf`.
  **L1298 CN**: 声明 TableGen def `roundf`。
- **L1299 EN**: Declares TableGen def `round`.
  **L1299 CN**: 声明 TableGen def `round`。
- **L1300 EN**: Declares TableGen defm `roundl`.
  **L1300 CN**: 声明 TableGen defm `roundl`。
- **L1301 EN**: Blank line separating nearby declarations or logic blocks.
  **L1301 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1302 EN**: Declares TableGen def `roundevenf`.
  **L1302 CN**: 声明 TableGen def `roundevenf`。
- **L1303 EN**: Declares TableGen def `roundeven`.
  **L1303 CN**: 声明 TableGen def `roundeven`。
- **L1304 EN**: Declares TableGen defm `roundevenl`.
  **L1304 CN**: 声明 TableGen defm `roundevenl`。
- **L1305 EN**: Blank line separating nearby declarations or logic blocks.
  **L1305 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1306 EN**: Declares TableGen def `floorf`.
  **L1306 CN**: 声明 TableGen def `floorf`。
- **L1307 EN**: Declares TableGen def `floor`.
  **L1307 CN**: 声明 TableGen def `floor`。
- **L1308 EN**: Declares TableGen defm `floorl`.
  **L1308 CN**: 声明 TableGen defm `floorl`。
- **L1309 EN**: Blank line separating nearby declarations or logic blocks.
  **L1309 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1310 EN**: Declares TableGen def `copysignf`.
  **L1310 CN**: 声明 TableGen def `copysignf`。
- **L1311 EN**: Declares TableGen def `copysign`.
  **L1311 CN**: 声明 TableGen def `copysign`。
- **L1312 EN**: Declares TableGen defm `copysignl`.
  **L1312 CN**: 声明 TableGen defm `copysignl`。

### Lines 1313-1344

````tablegen

def fminf : RuntimeLibcallImpl<FMIN_F32>;
def fmin : RuntimeLibcallImpl<FMIN_F64>;
defm fminl : LibmLongDoubleLibCall;

def fmaxf : RuntimeLibcallImpl<FMAX_F32>;
def fmax : RuntimeLibcallImpl<FMAX_F64>;
defm fmaxl : LibmLongDoubleLibCall;

def fminimumf : RuntimeLibcallImpl<FMINIMUM_F32>;
def fminimum : RuntimeLibcallImpl<FMINIMUM_F64>;
defm fminimuml : LibmLongDoubleLibCall;

def fmaximumf : RuntimeLibcallImpl<FMAXIMUM_F32>;
def fmaximum : RuntimeLibcallImpl<FMAXIMUM_F64>;
defm fmaximuml : LibmLongDoubleLibCall;

def fminimum_numf : RuntimeLibcallImpl<FMINIMUM_NUM_F32>;
def fminimum_num : RuntimeLibcallImpl<FMINIMUM_NUM_F64>;
defm fminimum_numl : LibmLongDoubleLibCall;

def fmaximum_numf : RuntimeLibcallImpl<FMAXIMUM_NUM_F32>;
def fmaximum_num : RuntimeLibcallImpl<FMAXIMUM_NUM_F64>;
defm fmaximum_numl : LibmLongDoubleLibCall;

def lroundf : RuntimeLibcallImpl<LROUND_F32>;
def lround : RuntimeLibcallImpl<LROUND_F64>;
defm lroundl : LibmLongDoubleLibCall;

def llroundf : RuntimeLibcallImpl<LLROUND_F32>;
def llround : RuntimeLibcallImpl<LLROUND_F64>;
defm llroundl : LibmLongDoubleLibCall;
````
- **L1313 EN**: Blank line separating nearby declarations or logic blocks.
  **L1313 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1314 EN**: Declares TableGen def `fminf`.
  **L1314 CN**: 声明 TableGen def `fminf`。
- **L1315 EN**: Declares TableGen def `fmin`.
  **L1315 CN**: 声明 TableGen def `fmin`。
- **L1316 EN**: Declares TableGen defm `fminl`.
  **L1316 CN**: 声明 TableGen defm `fminl`。
- **L1317 EN**: Blank line separating nearby declarations or logic blocks.
  **L1317 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1318 EN**: Declares TableGen def `fmaxf`.
  **L1318 CN**: 声明 TableGen def `fmaxf`。
- **L1319 EN**: Declares TableGen def `fmax`.
  **L1319 CN**: 声明 TableGen def `fmax`。
- **L1320 EN**: Declares TableGen defm `fmaxl`.
  **L1320 CN**: 声明 TableGen defm `fmaxl`。
- **L1321 EN**: Blank line separating nearby declarations or logic blocks.
  **L1321 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1322 EN**: Declares TableGen def `fminimumf`.
  **L1322 CN**: 声明 TableGen def `fminimumf`。
- **L1323 EN**: Declares TableGen def `fminimum`.
  **L1323 CN**: 声明 TableGen def `fminimum`。
- **L1324 EN**: Declares TableGen defm `fminimuml`.
  **L1324 CN**: 声明 TableGen defm `fminimuml`。
- **L1325 EN**: Blank line separating nearby declarations or logic blocks.
  **L1325 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1326 EN**: Declares TableGen def `fmaximumf`.
  **L1326 CN**: 声明 TableGen def `fmaximumf`。
- **L1327 EN**: Declares TableGen def `fmaximum`.
  **L1327 CN**: 声明 TableGen def `fmaximum`。
- **L1328 EN**: Declares TableGen defm `fmaximuml`.
  **L1328 CN**: 声明 TableGen defm `fmaximuml`。
- **L1329 EN**: Blank line separating nearby declarations or logic blocks.
  **L1329 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1330 EN**: Declares TableGen def `fminimum_numf`.
  **L1330 CN**: 声明 TableGen def `fminimum_numf`。
- **L1331 EN**: Declares TableGen def `fminimum_num`.
  **L1331 CN**: 声明 TableGen def `fminimum_num`。
- **L1332 EN**: Declares TableGen defm `fminimum_numl`.
  **L1332 CN**: 声明 TableGen defm `fminimum_numl`。
- **L1333 EN**: Blank line separating nearby declarations or logic blocks.
  **L1333 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1334 EN**: Declares TableGen def `fmaximum_numf`.
  **L1334 CN**: 声明 TableGen def `fmaximum_numf`。
- **L1335 EN**: Declares TableGen def `fmaximum_num`.
  **L1335 CN**: 声明 TableGen def `fmaximum_num`。
- **L1336 EN**: Declares TableGen defm `fmaximum_numl`.
  **L1336 CN**: 声明 TableGen defm `fmaximum_numl`。
- **L1337 EN**: Blank line separating nearby declarations or logic blocks.
  **L1337 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1338 EN**: Declares TableGen def `lroundf`.
  **L1338 CN**: 声明 TableGen def `lroundf`。
- **L1339 EN**: Declares TableGen def `lround`.
  **L1339 CN**: 声明 TableGen def `lround`。
- **L1340 EN**: Declares TableGen defm `lroundl`.
  **L1340 CN**: 声明 TableGen defm `lroundl`。
- **L1341 EN**: Blank line separating nearby declarations or logic blocks.
  **L1341 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1342 EN**: Declares TableGen def `llroundf`.
  **L1342 CN**: 声明 TableGen def `llroundf`。
- **L1343 EN**: Declares TableGen def `llround`.
  **L1343 CN**: 声明 TableGen def `llround`。
- **L1344 EN**: Declares TableGen defm `llroundl`.
  **L1344 CN**: 声明 TableGen defm `llroundl`。

### Lines 1345-1376

````tablegen

def lrintf : RuntimeLibcallImpl<LRINT_F32>;
def lrint : RuntimeLibcallImpl<LRINT_F64>;
defm lrintl : LibmLongDoubleLibCall;

def llrintf : RuntimeLibcallImpl<LLRINT_F32>;
def llrint : RuntimeLibcallImpl<LLRINT_F64>;
defm llrintl : LibmLongDoubleLibCall;

def ldexpf : RuntimeLibcallImpl<LDEXP_F32>;
def ldexp : RuntimeLibcallImpl<LDEXP_F64>;
defm ldexpl : LibmLongDoubleLibCall;

def frexpf : RuntimeLibcallImpl<FREXP_F32>;
def frexp : RuntimeLibcallImpl<FREXP_F64>;
defm frexpl : LibmLongDoubleLibCall;

def modff : RuntimeLibcallImpl<MODF_F32>;
def modf : RuntimeLibcallImpl<MODF_F64>;
defm modfl : LibmLongDoubleLibCall;

def nanf : RuntimeLibcallImpl<NAN_F32>;
def nan : RuntimeLibcallImpl<NAN_F64>;
defm nanl : LibmLongDoubleLibCall;

def nexttowardf : RuntimeLibcallImpl<NEXTTOWARD_F32>;
def nexttoward : RuntimeLibcallImpl<NEXTTOWARD_F64>;
defm nexttowardl : LibmLongDoubleLibCall;

def remainderf : RuntimeLibcallImpl<REMAINDER_F32>;
def remainder : RuntimeLibcallImpl<REMAINDER_F64>;
defm remainderl : LibmLongDoubleLibCall;
````
- **L1345 EN**: Blank line separating nearby declarations or logic blocks.
  **L1345 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1346 EN**: Declares TableGen def `lrintf`.
  **L1346 CN**: 声明 TableGen def `lrintf`。
- **L1347 EN**: Declares TableGen def `lrint`.
  **L1347 CN**: 声明 TableGen def `lrint`。
- **L1348 EN**: Declares TableGen defm `lrintl`.
  **L1348 CN**: 声明 TableGen defm `lrintl`。
- **L1349 EN**: Blank line separating nearby declarations or logic blocks.
  **L1349 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1350 EN**: Declares TableGen def `llrintf`.
  **L1350 CN**: 声明 TableGen def `llrintf`。
- **L1351 EN**: Declares TableGen def `llrint`.
  **L1351 CN**: 声明 TableGen def `llrint`。
- **L1352 EN**: Declares TableGen defm `llrintl`.
  **L1352 CN**: 声明 TableGen defm `llrintl`。
- **L1353 EN**: Blank line separating nearby declarations or logic blocks.
  **L1353 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1354 EN**: Declares TableGen def `ldexpf`.
  **L1354 CN**: 声明 TableGen def `ldexpf`。
- **L1355 EN**: Declares TableGen def `ldexp`.
  **L1355 CN**: 声明 TableGen def `ldexp`。
- **L1356 EN**: Declares TableGen defm `ldexpl`.
  **L1356 CN**: 声明 TableGen defm `ldexpl`。
- **L1357 EN**: Blank line separating nearby declarations or logic blocks.
  **L1357 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1358 EN**: Declares TableGen def `frexpf`.
  **L1358 CN**: 声明 TableGen def `frexpf`。
- **L1359 EN**: Declares TableGen def `frexp`.
  **L1359 CN**: 声明 TableGen def `frexp`。
- **L1360 EN**: Declares TableGen defm `frexpl`.
  **L1360 CN**: 声明 TableGen defm `frexpl`。
- **L1361 EN**: Blank line separating nearby declarations or logic blocks.
  **L1361 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1362 EN**: Declares TableGen def `modff`.
  **L1362 CN**: 声明 TableGen def `modff`。
- **L1363 EN**: Declares TableGen def `modf`.
  **L1363 CN**: 声明 TableGen def `modf`。
- **L1364 EN**: Declares TableGen defm `modfl`.
  **L1364 CN**: 声明 TableGen defm `modfl`。
- **L1365 EN**: Blank line separating nearby declarations or logic blocks.
  **L1365 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1366 EN**: Declares TableGen def `nanf`.
  **L1366 CN**: 声明 TableGen def `nanf`。
- **L1367 EN**: Declares TableGen def `nan`.
  **L1367 CN**: 声明 TableGen def `nan`。
- **L1368 EN**: Declares TableGen defm `nanl`.
  **L1368 CN**: 声明 TableGen defm `nanl`。
- **L1369 EN**: Blank line separating nearby declarations or logic blocks.
  **L1369 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1370 EN**: Declares TableGen def `nexttowardf`.
  **L1370 CN**: 声明 TableGen def `nexttowardf`。
- **L1371 EN**: Declares TableGen def `nexttoward`.
  **L1371 CN**: 声明 TableGen def `nexttoward`。
- **L1372 EN**: Declares TableGen defm `nexttowardl`.
  **L1372 CN**: 声明 TableGen defm `nexttowardl`。
- **L1373 EN**: Blank line separating nearby declarations or logic blocks.
  **L1373 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1374 EN**: Declares TableGen def `remainderf`.
  **L1374 CN**: 声明 TableGen def `remainderf`。
- **L1375 EN**: Declares TableGen def `remainder`.
  **L1375 CN**: 声明 TableGen def `remainder`。
- **L1376 EN**: Declares TableGen defm `remainderl`.
  **L1376 CN**: 声明 TableGen defm `remainderl`。

### Lines 1377-1408

````tablegen

def remquof : RuntimeLibcallImpl<REMQUO_F32>;
def remquo : RuntimeLibcallImpl<REMQUO_F64>;
defm remquol : LibmLongDoubleLibCall;

def fdimf : RuntimeLibcallImpl<FDIM_F32>;
def fdim : RuntimeLibcallImpl<FDIM_F64>;
defm fdiml : LibmLongDoubleLibCall;

def scalbnf : RuntimeLibcallImpl<SCALBN_F32>;
def scalbn : RuntimeLibcallImpl<SCALBN_F64>;
defm scalbnl : LibmLongDoubleLibCall;

def scalblnf : RuntimeLibcallImpl<SCALBLN_F32>;
def scalbln : RuntimeLibcallImpl<SCALBLN_F64>;
defm scalblnl : LibmLongDoubleLibCall;

def tgammaf : RuntimeLibcallImpl<TGAMMA_F32>;
def tgamma : RuntimeLibcallImpl<TGAMMA_F64>;
defm tgammal : LibmLongDoubleLibCall;

// Floating point environment
def fegetenv : RuntimeLibcallImpl<FEGETENV>;
def fesetenv : RuntimeLibcallImpl<FESETENV>;

// Floating point control modes
def fegetmode : RuntimeLibcallImpl<FEGETMODE>;
def fesetmode : RuntimeLibcallImpl<FESETMODE>;

//--------------------------------------------------------------------
// libc
//--------------------------------------------------------------------
````
- **L1377 EN**: Blank line separating nearby declarations or logic blocks.
  **L1377 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1378 EN**: Declares TableGen def `remquof`.
  **L1378 CN**: 声明 TableGen def `remquof`。
- **L1379 EN**: Declares TableGen def `remquo`.
  **L1379 CN**: 声明 TableGen def `remquo`。
- **L1380 EN**: Declares TableGen defm `remquol`.
  **L1380 CN**: 声明 TableGen defm `remquol`。
- **L1381 EN**: Blank line separating nearby declarations or logic blocks.
  **L1381 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1382 EN**: Declares TableGen def `fdimf`.
  **L1382 CN**: 声明 TableGen def `fdimf`。
- **L1383 EN**: Declares TableGen def `fdim`.
  **L1383 CN**: 声明 TableGen def `fdim`。
- **L1384 EN**: Declares TableGen defm `fdiml`.
  **L1384 CN**: 声明 TableGen defm `fdiml`。
- **L1385 EN**: Blank line separating nearby declarations or logic blocks.
  **L1385 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1386 EN**: Declares TableGen def `scalbnf`.
  **L1386 CN**: 声明 TableGen def `scalbnf`。
- **L1387 EN**: Declares TableGen def `scalbn`.
  **L1387 CN**: 声明 TableGen def `scalbn`。
- **L1388 EN**: Declares TableGen defm `scalbnl`.
  **L1388 CN**: 声明 TableGen defm `scalbnl`。
- **L1389 EN**: Blank line separating nearby declarations or logic blocks.
  **L1389 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1390 EN**: Declares TableGen def `scalblnf`.
  **L1390 CN**: 声明 TableGen def `scalblnf`。
- **L1391 EN**: Declares TableGen def `scalbln`.
  **L1391 CN**: 声明 TableGen def `scalbln`。
- **L1392 EN**: Declares TableGen defm `scalblnl`.
  **L1392 CN**: 声明 TableGen defm `scalblnl`。
- **L1393 EN**: Blank line separating nearby declarations or logic blocks.
  **L1393 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1394 EN**: Declares TableGen def `tgammaf`.
  **L1394 CN**: 声明 TableGen def `tgammaf`。
- **L1395 EN**: Declares TableGen def `tgamma`.
  **L1395 CN**: 声明 TableGen def `tgamma`。
- **L1396 EN**: Declares TableGen defm `tgammal`.
  **L1396 CN**: 声明 TableGen defm `tgammal`。
- **L1397 EN**: Blank line separating nearby declarations or logic blocks.
  **L1397 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1398 EN**: Comment explains nearby logic, invariants, or intent: `Floating point environment`.
  **L1398 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Floating point environment`。
- **L1399 EN**: Declares TableGen def `fegetenv`.
  **L1399 CN**: 声明 TableGen def `fegetenv`。
- **L1400 EN**: Declares TableGen def `fesetenv`.
  **L1400 CN**: 声明 TableGen def `fesetenv`。
- **L1401 EN**: Blank line separating nearby declarations or logic blocks.
  **L1401 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1402 EN**: Comment explains nearby logic, invariants, or intent: `Floating point control modes`.
  **L1402 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Floating point control modes`。
- **L1403 EN**: Declares TableGen def `fegetmode`.
  **L1403 CN**: 声明 TableGen def `fegetmode`。
- **L1404 EN**: Declares TableGen def `fesetmode`.
  **L1404 CN**: 声明 TableGen def `fesetmode`。
- **L1405 EN**: Blank line separating nearby declarations or logic blocks.
  **L1405 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1406 EN**: Comment explains nearby logic, invariants, or intent: `--------------------------------------------------------------------`.
  **L1406 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`--------------------------------------------------------------------`。
- **L1407 EN**: Comment explains nearby logic, invariants, or intent: `libc`.
  **L1407 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`libc`。
- **L1408 EN**: Comment explains nearby logic, invariants, or intent: `--------------------------------------------------------------------`.
  **L1408 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`--------------------------------------------------------------------`。

### Lines 1409-1440

````tablegen

// Memory
def memcpy : RuntimeLibcallImpl<MEMCPY>;
def memmove : RuntimeLibcallImpl<MEMMOVE>;
def memset : RuntimeLibcallImpl<MEMSET>;

def __memcpy_chk : RuntimeLibcallImpl<MEMCPY_CHK>;
def __memmove_chk : RuntimeLibcallImpl<MEMMOVE_CHK>;
def __memset_chk : RuntimeLibcallImpl<MEMSET_CHK>;

def malloc : RuntimeLibcallImpl<MALLOC>;

// DSEPass can emit calloc if it finds a pair of malloc/memset
def calloc : RuntimeLibcallImpl<CALLOC>;
def free : RuntimeLibcallImpl<FREE>;

} // End let IsDefault = true

def exp10f : RuntimeLibcallImpl<EXP10_F32>;
def exp10 : RuntimeLibcallImpl<EXP10_F64>;
def exp10l_f80 : RuntimeLibcallImpl<EXP10_F80, "exp10l">;
def exp10l_f128 : RuntimeLibcallImpl<EXP10_F128, "exp10l">;
def exp10l_ppcf128 : RuntimeLibcallImpl<EXP10_PPCF128, "exp10l">;

// Stack Protector Fail
def __stack_chk_fail : RuntimeLibcallImpl<STACKPROTECTOR_CHECK_FAIL>;

//--------------------------------------------------------------------
// Other functions from TargetLibraryInfo
//
// TODO: These need to be organized by library and added to relevant
// systems.
````
- **L1409 EN**: Blank line separating nearby declarations or logic blocks.
  **L1409 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1410 EN**: Comment explains nearby logic, invariants, or intent: `Memory`.
  **L1410 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Memory`。
- **L1411 EN**: Declares TableGen def `memcpy`.
  **L1411 CN**: 声明 TableGen def `memcpy`。
- **L1412 EN**: Declares TableGen def `memmove`.
  **L1412 CN**: 声明 TableGen def `memmove`。
- **L1413 EN**: Declares TableGen def `memset`.
  **L1413 CN**: 声明 TableGen def `memset`。
- **L1414 EN**: Blank line separating nearby declarations or logic blocks.
  **L1414 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1415 EN**: Declares TableGen def `__memcpy_chk`.
  **L1415 CN**: 声明 TableGen def `__memcpy_chk`。
- **L1416 EN**: Declares TableGen def `__memmove_chk`.
  **L1416 CN**: 声明 TableGen def `__memmove_chk`。
- **L1417 EN**: Declares TableGen def `__memset_chk`.
  **L1417 CN**: 声明 TableGen def `__memset_chk`。
- **L1418 EN**: Blank line separating nearby declarations or logic blocks.
  **L1418 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1419 EN**: Declares TableGen def `malloc`.
  **L1419 CN**: 声明 TableGen def `malloc`。
- **L1420 EN**: Blank line separating nearby declarations or logic blocks.
  **L1420 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1421 EN**: Comment explains nearby logic, invariants, or intent: `DSEPass can emit calloc if it finds a pair of malloc/memset`.
  **L1421 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`DSEPass can emit calloc if it finds a pair of malloc/memset`。
- **L1422 EN**: Declares TableGen def `calloc`.
  **L1422 CN**: 声明 TableGen def `calloc`。
- **L1423 EN**: Declares TableGen def `free`.
  **L1423 CN**: 声明 TableGen def `free`。
- **L1424 EN**: Blank line separating nearby declarations or logic blocks.
  **L1424 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1425 EN**: Continues the surrounding expression or declaration: `} // End let IsDefault = true`.
  **L1425 CN**: 继续构造周围的表达式或声明：`} // End let IsDefault = true`。
- **L1426 EN**: Blank line separating nearby declarations or logic blocks.
  **L1426 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1427 EN**: Declares TableGen def `exp10f`.
  **L1427 CN**: 声明 TableGen def `exp10f`。
- **L1428 EN**: Declares TableGen def `exp10`.
  **L1428 CN**: 声明 TableGen def `exp10`。
- **L1429 EN**: Declares TableGen def `exp10l_f80`.
  **L1429 CN**: 声明 TableGen def `exp10l_f80`。
- **L1430 EN**: Declares TableGen def `exp10l_f128`.
  **L1430 CN**: 声明 TableGen def `exp10l_f128`。
- **L1431 EN**: Declares TableGen def `exp10l_ppcf128`.
  **L1431 CN**: 声明 TableGen def `exp10l_ppcf128`。
- **L1432 EN**: Blank line separating nearby declarations or logic blocks.
  **L1432 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1433 EN**: Comment explains nearby logic, invariants, or intent: `Stack Protector Fail`.
  **L1433 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Stack Protector Fail`。
- **L1434 EN**: Declares TableGen def `__stack_chk_fail`.
  **L1434 CN**: 声明 TableGen def `__stack_chk_fail`。
- **L1435 EN**: Blank line separating nearby declarations or logic blocks.
  **L1435 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1436 EN**: Comment explains nearby logic, invariants, or intent: `--------------------------------------------------------------------`.
  **L1436 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`--------------------------------------------------------------------`。
- **L1437 EN**: Comment explains nearby logic, invariants, or intent: `Other functions from TargetLibraryInfo`.
  **L1437 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Other functions from TargetLibraryInfo`。
- **L1438 EN**: Separator comment used for visual grouping.
  **L1438 CN**: 用于视觉分组的分隔注释。
- **L1439 EN**: Comment records a pending task or caution: `TODO: These need to be organized by library and added to relevant`.
  **L1439 CN**: 注释记录了待办事项或注意点：`TODO: These need to be organized by library and added to relevant`。
- **L1440 EN**: Comment explains nearby logic, invariants, or intent: `systems.`.
  **L1440 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`systems.`。

### Lines 1441-1472

````tablegen
///
// --------------------------------------------------------------------

def __2_YAPAXI_Z : RuntimeLibcallImpl<MSVC_NEW_INT, "??2@YAPAXI@Z">;
def __2_YAPAXIABUnothrow_t_std___Z
    : RuntimeLibcallImpl<MSVC_NEW_INT_NOTHROW,
                         "??2@YAPAXIABUnothrow_t@std@@@Z">;
def __2_YAPEAX_K_Z : RuntimeLibcallImpl<MSVC_NEW_LONGLONG, "??2@YAPEAX_K@Z">;
def __2_YAPEAX_KAEBUnothrow_t_std___Z
    : RuntimeLibcallImpl<MSVC_NEW_LONGLONG_NOTHROW,
                         "??2@YAPEAX_KAEBUnothrow_t@std@@@Z">;
def __3_YAXPAX_Z : RuntimeLibcallImpl<MSVC_DELETE_PTR32, "??3@YAXPAX@Z">;
def __3_YAXPAXABUnothrow_t_std___Z
    : RuntimeLibcallImpl<MSVC_DELETE_PTR32_NOTHROW,
                         "??3@YAXPAXABUnothrow_t@std@@@Z">;
def __3_YAXPAXI_Z : RuntimeLibcallImpl<MSVC_DELETE_PTR32_INT, "??3@YAXPAXI@Z">;
def __3_YAXPEAX_Z : RuntimeLibcallImpl<MSVC_DELETE_PTR64, "??3@YAXPEAX@Z">;
def __3_YAXPEAXAEBUnothrow_t_std___Z
    : RuntimeLibcallImpl<MSVC_DELETE_PTR64_NOTHROW,
                         "??3@YAXPEAXAEBUnothrow_t@std@@@Z">;
def __3_YAXPEAX_K_Z
    : RuntimeLibcallImpl<MSVC_DELETE_PTR64_LONGLONG, "??3@YAXPEAX_K@Z">;
def ___U_YAPAXI_Z : RuntimeLibcallImpl<MSVC_NEW_ARRAY_INT, "??_U@YAPAXI@Z">;
def ___U_YAPAXIABUnothrow_t_std___Z
    : RuntimeLibcallImpl<MSVC_NEW_ARRAY_INT_NOTHROW,
                         "??_U@YAPAXIABUnothrow_t@std@@@Z">;
def ___U_YAPEAX_K_Z
    : RuntimeLibcallImpl<MSVC_NEW_ARRAY_LONGLONG, "??_U@YAPEAX_K@Z">;
def ___U_YAPEAX_KAEBUnothrow_t_std___Z
    : RuntimeLibcallImpl<MSVC_NEW_ARRAY_LONGLONG_NOTHROW,
                         "??_U@YAPEAX_KAEBUnothrow_t@std@@@Z">;
def ___V_YAXPAX_Z
````
- **L1441 EN**: Separator comment used for visual grouping.
  **L1441 CN**: 用于视觉分组的分隔注释。
- **L1442 EN**: Comment explains nearby logic, invariants, or intent: `--------------------------------------------------------------------`.
  **L1442 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`--------------------------------------------------------------------`。
- **L1443 EN**: Blank line separating nearby declarations or logic blocks.
  **L1443 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1444 EN**: Declares TableGen def `__2_YAPAXI_Z`.
  **L1444 CN**: 声明 TableGen def `__2_YAPAXI_Z`。
- **L1445 EN**: Declares TableGen def `__2_YAPAXIABUnothrow_t_std___Z`.
  **L1445 CN**: 声明 TableGen def `__2_YAPAXIABUnothrow_t_std___Z`。
- **L1446 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: RuntimeLibcallImpl<MSVC_NEW_INT_NOTHROW,`.
  **L1446 CN**: 继续一个多行参数列表、初始化器或聚合项：`: RuntimeLibcallImpl<MSVC_NEW_INT_NOTHROW,`。
- **L1447 EN**: Executes a standalone statement or declaration: `"??2@YAPAXIABUnothrow_t@std@@@Z">;`.
  **L1447 CN**: 执行一条独立语句或声明：`"??2@YAPAXIABUnothrow_t@std@@@Z">;`。
- **L1448 EN**: Declares TableGen def `__2_YAPEAX_K_Z`.
  **L1448 CN**: 声明 TableGen def `__2_YAPEAX_K_Z`。
- **L1449 EN**: Declares TableGen def `__2_YAPEAX_KAEBUnothrow_t_std___Z`.
  **L1449 CN**: 声明 TableGen def `__2_YAPEAX_KAEBUnothrow_t_std___Z`。
- **L1450 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: RuntimeLibcallImpl<MSVC_NEW_LONGLONG_NOTHROW,`.
  **L1450 CN**: 继续一个多行参数列表、初始化器或聚合项：`: RuntimeLibcallImpl<MSVC_NEW_LONGLONG_NOTHROW,`。
- **L1451 EN**: Executes a standalone statement or declaration: `"??2@YAPEAX_KAEBUnothrow_t@std@@@Z">;`.
  **L1451 CN**: 执行一条独立语句或声明：`"??2@YAPEAX_KAEBUnothrow_t@std@@@Z">;`。
- **L1452 EN**: Declares TableGen def `__3_YAXPAX_Z`.
  **L1452 CN**: 声明 TableGen def `__3_YAXPAX_Z`。
- **L1453 EN**: Declares TableGen def `__3_YAXPAXABUnothrow_t_std___Z`.
  **L1453 CN**: 声明 TableGen def `__3_YAXPAXABUnothrow_t_std___Z`。
- **L1454 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: RuntimeLibcallImpl<MSVC_DELETE_PTR32_NOTHROW,`.
  **L1454 CN**: 继续一个多行参数列表、初始化器或聚合项：`: RuntimeLibcallImpl<MSVC_DELETE_PTR32_NOTHROW,`。
- **L1455 EN**: Executes a standalone statement or declaration: `"??3@YAXPAXABUnothrow_t@std@@@Z">;`.
  **L1455 CN**: 执行一条独立语句或声明：`"??3@YAXPAXABUnothrow_t@std@@@Z">;`。
- **L1456 EN**: Declares TableGen def `__3_YAXPAXI_Z`.
  **L1456 CN**: 声明 TableGen def `__3_YAXPAXI_Z`。
- **L1457 EN**: Declares TableGen def `__3_YAXPEAX_Z`.
  **L1457 CN**: 声明 TableGen def `__3_YAXPEAX_Z`。
- **L1458 EN**: Declares TableGen def `__3_YAXPEAXAEBUnothrow_t_std___Z`.
  **L1458 CN**: 声明 TableGen def `__3_YAXPEAXAEBUnothrow_t_std___Z`。
- **L1459 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: RuntimeLibcallImpl<MSVC_DELETE_PTR64_NOTHROW,`.
  **L1459 CN**: 继续一个多行参数列表、初始化器或聚合项：`: RuntimeLibcallImpl<MSVC_DELETE_PTR64_NOTHROW,`。
- **L1460 EN**: Executes a standalone statement or declaration: `"??3@YAXPEAXAEBUnothrow_t@std@@@Z">;`.
  **L1460 CN**: 执行一条独立语句或声明：`"??3@YAXPEAXAEBUnothrow_t@std@@@Z">;`。
- **L1461 EN**: Declares TableGen def `__3_YAXPEAX_K_Z`.
  **L1461 CN**: 声明 TableGen def `__3_YAXPEAX_K_Z`。
- **L1462 EN**: Executes a standalone statement or declaration: `: RuntimeLibcallImpl<MSVC_DELETE_PTR64_LONGLONG, "??3@YAXPEAX_K@Z">;`.
  **L1462 CN**: 执行一条独立语句或声明：`: RuntimeLibcallImpl<MSVC_DELETE_PTR64_LONGLONG, "??3@YAXPEAX_K@Z">;`。
- **L1463 EN**: Declares TableGen def `___U_YAPAXI_Z`.
  **L1463 CN**: 声明 TableGen def `___U_YAPAXI_Z`。
- **L1464 EN**: Declares TableGen def `___U_YAPAXIABUnothrow_t_std___Z`.
  **L1464 CN**: 声明 TableGen def `___U_YAPAXIABUnothrow_t_std___Z`。
- **L1465 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: RuntimeLibcallImpl<MSVC_NEW_ARRAY_INT_NOTHROW,`.
  **L1465 CN**: 继续一个多行参数列表、初始化器或聚合项：`: RuntimeLibcallImpl<MSVC_NEW_ARRAY_INT_NOTHROW,`。
- **L1466 EN**: Executes a standalone statement or declaration: `"??_U@YAPAXIABUnothrow_t@std@@@Z">;`.
  **L1466 CN**: 执行一条独立语句或声明：`"??_U@YAPAXIABUnothrow_t@std@@@Z">;`。
- **L1467 EN**: Declares TableGen def `___U_YAPEAX_K_Z`.
  **L1467 CN**: 声明 TableGen def `___U_YAPEAX_K_Z`。
- **L1468 EN**: Executes a standalone statement or declaration: `: RuntimeLibcallImpl<MSVC_NEW_ARRAY_LONGLONG, "??_U@YAPEAX_K@Z">;`.
  **L1468 CN**: 执行一条独立语句或声明：`: RuntimeLibcallImpl<MSVC_NEW_ARRAY_LONGLONG, "??_U@YAPEAX_K@Z">;`。
- **L1469 EN**: Declares TableGen def `___U_YAPEAX_KAEBUnothrow_t_std___Z`.
  **L1469 CN**: 声明 TableGen def `___U_YAPEAX_KAEBUnothrow_t_std___Z`。
- **L1470 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: RuntimeLibcallImpl<MSVC_NEW_ARRAY_LONGLONG_NOTHROW,`.
  **L1470 CN**: 继续一个多行参数列表、初始化器或聚合项：`: RuntimeLibcallImpl<MSVC_NEW_ARRAY_LONGLONG_NOTHROW,`。
- **L1471 EN**: Executes a standalone statement or declaration: `"??_U@YAPEAX_KAEBUnothrow_t@std@@@Z">;`.
  **L1471 CN**: 执行一条独立语句或声明：`"??_U@YAPEAX_KAEBUnothrow_t@std@@@Z">;`。
- **L1472 EN**: Declares TableGen def `___V_YAXPAX_Z`.
  **L1472 CN**: 声明 TableGen def `___V_YAXPAX_Z`。

### Lines 1473-1504

````tablegen
    : RuntimeLibcallImpl<MSVC_DELETE_ARRAY_PTR32, "??_V@YAXPAX@Z">;
def ___V_YAXPAXABUnothrow_t_std___Z
    : RuntimeLibcallImpl<MSVC_DELETE_ARRAY_PTR32_NOTHROW,
                         "??_V@YAXPAXABUnothrow_t@std@@@Z">;
def ___V_YAXPAXI_Z
    : RuntimeLibcallImpl<MSVC_DELETE_ARRAY_PTR32_INT, "??_V@YAXPAXI@Z">;
def ___V_YAXPEAX_Z
    : RuntimeLibcallImpl<MSVC_DELETE_ARRAY_PTR64, "??_V@YAXPEAX@Z">;
def ___V_YAXPEAXAEBUnothrow_t_std___Z
    : RuntimeLibcallImpl<MSVC_DELETE_ARRAY_PTR64_NOTHROW,
                         "??_V@YAXPEAXAEBUnothrow_t@std@@@Z">;
def ___V_YAXPEAX_K_Z
    : RuntimeLibcallImpl<MSVC_DELETE_ARRAY_PTR64_LONGLONG, "??_V@YAXPEAX_K@Z">;
def _IO_getc : RuntimeLibcallImpl<UNDER_IO_GETC>;
def _IO_putc : RuntimeLibcallImpl<UNDER_IO_PUTC>;
def _ZdaPv : RuntimeLibcallImpl<ZDAPV>;
def _ZdaPvRKSt9nothrow_t : RuntimeLibcallImpl<ZDAPVRKST9NOTHROW_T>;
def _ZdaPvSt11align_val_t : RuntimeLibcallImpl<ZDAPVST11ALIGN_VAL_T>;
def _ZdaPvSt11align_val_tRKSt9nothrow_t
    : RuntimeLibcallImpl<ZDAPVST11ALIGN_VAL_TRKST9NOTHROW_T>;
def _ZdaPvj : RuntimeLibcallImpl<ZDAPVJ>;
def _ZdaPvjSt11align_val_t : RuntimeLibcallImpl<ZDAPVJST11ALIGN_VAL_T>;
def _ZdaPvm : RuntimeLibcallImpl<ZDAPVM>;
def _ZdaPvmSt11align_val_t : RuntimeLibcallImpl<ZDAPVMST11ALIGN_VAL_T>;
def _ZdlPv : RuntimeLibcallImpl<ZDLPV>;
def _ZdlPvRKSt9nothrow_t : RuntimeLibcallImpl<ZDLPVRKST9NOTHROW_T>;
def _ZdlPvSt11align_val_t : RuntimeLibcallImpl<ZDLPVST11ALIGN_VAL_T>;
def _ZdlPvSt11align_val_tRKSt9nothrow_t
    : RuntimeLibcallImpl<ZDLPVST11ALIGN_VAL_TRKST9NOTHROW_T>;
def _ZdlPvj : RuntimeLibcallImpl<ZDLPVJ>;
def _ZdlPvjSt11align_val_t : RuntimeLibcallImpl<ZDLPVJST11ALIGN_VAL_T>;
def _ZdlPvm : RuntimeLibcallImpl<ZDLPVM>;
````
- **L1473 EN**: Executes a standalone statement or declaration: `: RuntimeLibcallImpl<MSVC_DELETE_ARRAY_PTR32, "??_V@YAXPAX@Z">;`.
  **L1473 CN**: 执行一条独立语句或声明：`: RuntimeLibcallImpl<MSVC_DELETE_ARRAY_PTR32, "??_V@YAXPAX@Z">;`。
- **L1474 EN**: Declares TableGen def `___V_YAXPAXABUnothrow_t_std___Z`.
  **L1474 CN**: 声明 TableGen def `___V_YAXPAXABUnothrow_t_std___Z`。
- **L1475 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: RuntimeLibcallImpl<MSVC_DELETE_ARRAY_PTR32_NOTHROW,`.
  **L1475 CN**: 继续一个多行参数列表、初始化器或聚合项：`: RuntimeLibcallImpl<MSVC_DELETE_ARRAY_PTR32_NOTHROW,`。
- **L1476 EN**: Executes a standalone statement or declaration: `"??_V@YAXPAXABUnothrow_t@std@@@Z">;`.
  **L1476 CN**: 执行一条独立语句或声明：`"??_V@YAXPAXABUnothrow_t@std@@@Z">;`。
- **L1477 EN**: Declares TableGen def `___V_YAXPAXI_Z`.
  **L1477 CN**: 声明 TableGen def `___V_YAXPAXI_Z`。
- **L1478 EN**: Executes a standalone statement or declaration: `: RuntimeLibcallImpl<MSVC_DELETE_ARRAY_PTR32_INT, "??_V@YAXPAXI@Z">;`.
  **L1478 CN**: 执行一条独立语句或声明：`: RuntimeLibcallImpl<MSVC_DELETE_ARRAY_PTR32_INT, "??_V@YAXPAXI@Z">;`。
- **L1479 EN**: Declares TableGen def `___V_YAXPEAX_Z`.
  **L1479 CN**: 声明 TableGen def `___V_YAXPEAX_Z`。
- **L1480 EN**: Executes a standalone statement or declaration: `: RuntimeLibcallImpl<MSVC_DELETE_ARRAY_PTR64, "??_V@YAXPEAX@Z">;`.
  **L1480 CN**: 执行一条独立语句或声明：`: RuntimeLibcallImpl<MSVC_DELETE_ARRAY_PTR64, "??_V@YAXPEAX@Z">;`。
- **L1481 EN**: Declares TableGen def `___V_YAXPEAXAEBUnothrow_t_std___Z`.
  **L1481 CN**: 声明 TableGen def `___V_YAXPEAXAEBUnothrow_t_std___Z`。
- **L1482 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: RuntimeLibcallImpl<MSVC_DELETE_ARRAY_PTR64_NOTHROW,`.
  **L1482 CN**: 继续一个多行参数列表、初始化器或聚合项：`: RuntimeLibcallImpl<MSVC_DELETE_ARRAY_PTR64_NOTHROW,`。
- **L1483 EN**: Executes a standalone statement or declaration: `"??_V@YAXPEAXAEBUnothrow_t@std@@@Z">;`.
  **L1483 CN**: 执行一条独立语句或声明：`"??_V@YAXPEAXAEBUnothrow_t@std@@@Z">;`。
- **L1484 EN**: Declares TableGen def `___V_YAXPEAX_K_Z`.
  **L1484 CN**: 声明 TableGen def `___V_YAXPEAX_K_Z`。
- **L1485 EN**: Executes a standalone statement or declaration: `: RuntimeLibcallImpl<MSVC_DELETE_ARRAY_PTR64_LONGLONG, "??_V@YAXPEAX_K@Z">;`.
  **L1485 CN**: 执行一条独立语句或声明：`: RuntimeLibcallImpl<MSVC_DELETE_ARRAY_PTR64_LONGLONG, "??_V@YAXPEAX_K@Z">;`。
- **L1486 EN**: Declares TableGen def `_IO_getc`.
  **L1486 CN**: 声明 TableGen def `_IO_getc`。
- **L1487 EN**: Declares TableGen def `_IO_putc`.
  **L1487 CN**: 声明 TableGen def `_IO_putc`。
- **L1488 EN**: Declares TableGen def `_ZdaPv`.
  **L1488 CN**: 声明 TableGen def `_ZdaPv`。
- **L1489 EN**: Declares TableGen def `_ZdaPvRKSt9nothrow_t`.
  **L1489 CN**: 声明 TableGen def `_ZdaPvRKSt9nothrow_t`。
- **L1490 EN**: Declares TableGen def `_ZdaPvSt11align_val_t`.
  **L1490 CN**: 声明 TableGen def `_ZdaPvSt11align_val_t`。
- **L1491 EN**: Declares TableGen def `_ZdaPvSt11align_val_tRKSt9nothrow_t`.
  **L1491 CN**: 声明 TableGen def `_ZdaPvSt11align_val_tRKSt9nothrow_t`。
- **L1492 EN**: Executes a standalone statement or declaration: `: RuntimeLibcallImpl<ZDAPVST11ALIGN_VAL_TRKST9NOTHROW_T>;`.
  **L1492 CN**: 执行一条独立语句或声明：`: RuntimeLibcallImpl<ZDAPVST11ALIGN_VAL_TRKST9NOTHROW_T>;`。
- **L1493 EN**: Declares TableGen def `_ZdaPvj`.
  **L1493 CN**: 声明 TableGen def `_ZdaPvj`。
- **L1494 EN**: Declares TableGen def `_ZdaPvjSt11align_val_t`.
  **L1494 CN**: 声明 TableGen def `_ZdaPvjSt11align_val_t`。
- **L1495 EN**: Declares TableGen def `_ZdaPvm`.
  **L1495 CN**: 声明 TableGen def `_ZdaPvm`。
- **L1496 EN**: Declares TableGen def `_ZdaPvmSt11align_val_t`.
  **L1496 CN**: 声明 TableGen def `_ZdaPvmSt11align_val_t`。
- **L1497 EN**: Declares TableGen def `_ZdlPv`.
  **L1497 CN**: 声明 TableGen def `_ZdlPv`。
- **L1498 EN**: Declares TableGen def `_ZdlPvRKSt9nothrow_t`.
  **L1498 CN**: 声明 TableGen def `_ZdlPvRKSt9nothrow_t`。
- **L1499 EN**: Declares TableGen def `_ZdlPvSt11align_val_t`.
  **L1499 CN**: 声明 TableGen def `_ZdlPvSt11align_val_t`。
- **L1500 EN**: Declares TableGen def `_ZdlPvSt11align_val_tRKSt9nothrow_t`.
  **L1500 CN**: 声明 TableGen def `_ZdlPvSt11align_val_tRKSt9nothrow_t`。
- **L1501 EN**: Executes a standalone statement or declaration: `: RuntimeLibcallImpl<ZDLPVST11ALIGN_VAL_TRKST9NOTHROW_T>;`.
  **L1501 CN**: 执行一条独立语句或声明：`: RuntimeLibcallImpl<ZDLPVST11ALIGN_VAL_TRKST9NOTHROW_T>;`。
- **L1502 EN**: Declares TableGen def `_ZdlPvj`.
  **L1502 CN**: 声明 TableGen def `_ZdlPvj`。
- **L1503 EN**: Declares TableGen def `_ZdlPvjSt11align_val_t`.
  **L1503 CN**: 声明 TableGen def `_ZdlPvjSt11align_val_t`。
- **L1504 EN**: Declares TableGen def `_ZdlPvm`.
  **L1504 CN**: 声明 TableGen def `_ZdlPvm`。

### Lines 1505-1536

````tablegen
def _ZdlPvmSt11align_val_t : RuntimeLibcallImpl<ZDLPVMST11ALIGN_VAL_T>;
def _Znaj : RuntimeLibcallImpl<ZNAJ>;
def _ZnajRKSt9nothrow_t : RuntimeLibcallImpl<ZNAJRKST9NOTHROW_T>;
def _ZnajSt11align_val_t : RuntimeLibcallImpl<ZNAJST11ALIGN_VAL_T>;
def _ZnajSt11align_val_tRKSt9nothrow_t
    : RuntimeLibcallImpl<ZNAJST11ALIGN_VAL_TRKST9NOTHROW_T>;
def _Znam : RuntimeLibcallImpl<ZNAM>;
def _Znam12__hot_cold_t : RuntimeLibcallImpl<ZNAM12__HOT_COLD_T>;
def _ZnamRKSt9nothrow_t : RuntimeLibcallImpl<ZNAMRKST9NOTHROW_T>;
def _ZnamRKSt9nothrow_t12__hot_cold_t
    : RuntimeLibcallImpl<ZNAMRKST9NOTHROW_T12__HOT_COLD_T>;
def _ZnamSt11align_val_t : RuntimeLibcallImpl<ZNAMST11ALIGN_VAL_T>;
def _ZnamSt11align_val_t12__hot_cold_t
    : RuntimeLibcallImpl<ZNAMST11ALIGN_VAL_T12__HOT_COLD_T>;
def _ZnamSt11align_val_tRKSt9nothrow_t
    : RuntimeLibcallImpl<ZNAMST11ALIGN_VAL_TRKST9NOTHROW_T>;
def _ZnamSt11align_val_tRKSt9nothrow_t12__hot_cold_t
    : RuntimeLibcallImpl<ZNAMST11ALIGN_VAL_TRKST9NOTHROW_T12__HOT_COLD_T>;
def _Znwj : RuntimeLibcallImpl<ZNWJ>;
def _ZnwjRKSt9nothrow_t : RuntimeLibcallImpl<ZNWJRKST9NOTHROW_T>;
def _ZnwjSt11align_val_t : RuntimeLibcallImpl<ZNWJST11ALIGN_VAL_T>;
def _ZnwjSt11align_val_tRKSt9nothrow_t
    : RuntimeLibcallImpl<ZNWJST11ALIGN_VAL_TRKST9NOTHROW_T>;
def _Znwm : RuntimeLibcallImpl<ZNWM>;
def _Znwm12__hot_cold_t : RuntimeLibcallImpl<ZNWM12__HOT_COLD_T>;
def _ZnwmRKSt9nothrow_t : RuntimeLibcallImpl<ZNWMRKST9NOTHROW_T>;
def _ZnwmRKSt9nothrow_t12__hot_cold_t
    : RuntimeLibcallImpl<ZNWMRKST9NOTHROW_T12__HOT_COLD_T>;
def _ZnwmSt11align_val_t : RuntimeLibcallImpl<ZNWMST11ALIGN_VAL_T>;
def _ZnwmSt11align_val_t12__hot_cold_t
    : RuntimeLibcallImpl<ZNWMST11ALIGN_VAL_T12__HOT_COLD_T>;
def _ZnwmSt11align_val_tRKSt9nothrow_t
````
- **L1505 EN**: Declares TableGen def `_ZdlPvmSt11align_val_t`.
  **L1505 CN**: 声明 TableGen def `_ZdlPvmSt11align_val_t`。
- **L1506 EN**: Declares TableGen def `_Znaj`.
  **L1506 CN**: 声明 TableGen def `_Znaj`。
- **L1507 EN**: Declares TableGen def `_ZnajRKSt9nothrow_t`.
  **L1507 CN**: 声明 TableGen def `_ZnajRKSt9nothrow_t`。
- **L1508 EN**: Declares TableGen def `_ZnajSt11align_val_t`.
  **L1508 CN**: 声明 TableGen def `_ZnajSt11align_val_t`。
- **L1509 EN**: Declares TableGen def `_ZnajSt11align_val_tRKSt9nothrow_t`.
  **L1509 CN**: 声明 TableGen def `_ZnajSt11align_val_tRKSt9nothrow_t`。
- **L1510 EN**: Executes a standalone statement or declaration: `: RuntimeLibcallImpl<ZNAJST11ALIGN_VAL_TRKST9NOTHROW_T>;`.
  **L1510 CN**: 执行一条独立语句或声明：`: RuntimeLibcallImpl<ZNAJST11ALIGN_VAL_TRKST9NOTHROW_T>;`。
- **L1511 EN**: Declares TableGen def `_Znam`.
  **L1511 CN**: 声明 TableGen def `_Znam`。
- **L1512 EN**: Declares TableGen def `_Znam12__hot_cold_t`.
  **L1512 CN**: 声明 TableGen def `_Znam12__hot_cold_t`。
- **L1513 EN**: Declares TableGen def `_ZnamRKSt9nothrow_t`.
  **L1513 CN**: 声明 TableGen def `_ZnamRKSt9nothrow_t`。
- **L1514 EN**: Declares TableGen def `_ZnamRKSt9nothrow_t12__hot_cold_t`.
  **L1514 CN**: 声明 TableGen def `_ZnamRKSt9nothrow_t12__hot_cold_t`。
- **L1515 EN**: Executes a standalone statement or declaration: `: RuntimeLibcallImpl<ZNAMRKST9NOTHROW_T12__HOT_COLD_T>;`.
  **L1515 CN**: 执行一条独立语句或声明：`: RuntimeLibcallImpl<ZNAMRKST9NOTHROW_T12__HOT_COLD_T>;`。
- **L1516 EN**: Declares TableGen def `_ZnamSt11align_val_t`.
  **L1516 CN**: 声明 TableGen def `_ZnamSt11align_val_t`。
- **L1517 EN**: Declares TableGen def `_ZnamSt11align_val_t12__hot_cold_t`.
  **L1517 CN**: 声明 TableGen def `_ZnamSt11align_val_t12__hot_cold_t`。
- **L1518 EN**: Executes a standalone statement or declaration: `: RuntimeLibcallImpl<ZNAMST11ALIGN_VAL_T12__HOT_COLD_T>;`.
  **L1518 CN**: 执行一条独立语句或声明：`: RuntimeLibcallImpl<ZNAMST11ALIGN_VAL_T12__HOT_COLD_T>;`。
- **L1519 EN**: Declares TableGen def `_ZnamSt11align_val_tRKSt9nothrow_t`.
  **L1519 CN**: 声明 TableGen def `_ZnamSt11align_val_tRKSt9nothrow_t`。
- **L1520 EN**: Executes a standalone statement or declaration: `: RuntimeLibcallImpl<ZNAMST11ALIGN_VAL_TRKST9NOTHROW_T>;`.
  **L1520 CN**: 执行一条独立语句或声明：`: RuntimeLibcallImpl<ZNAMST11ALIGN_VAL_TRKST9NOTHROW_T>;`。
- **L1521 EN**: Declares TableGen def `_ZnamSt11align_val_tRKSt9nothrow_t12__hot_cold_t`.
  **L1521 CN**: 声明 TableGen def `_ZnamSt11align_val_tRKSt9nothrow_t12__hot_cold_t`。
- **L1522 EN**: Executes a standalone statement or declaration: `: RuntimeLibcallImpl<ZNAMST11ALIGN_VAL_TRKST9NOTHROW_T12__HOT_COLD_T>;`.
  **L1522 CN**: 执行一条独立语句或声明：`: RuntimeLibcallImpl<ZNAMST11ALIGN_VAL_TRKST9NOTHROW_T12__HOT_COLD_T>;`。
- **L1523 EN**: Declares TableGen def `_Znwj`.
  **L1523 CN**: 声明 TableGen def `_Znwj`。
- **L1524 EN**: Declares TableGen def `_ZnwjRKSt9nothrow_t`.
  **L1524 CN**: 声明 TableGen def `_ZnwjRKSt9nothrow_t`。
- **L1525 EN**: Declares TableGen def `_ZnwjSt11align_val_t`.
  **L1525 CN**: 声明 TableGen def `_ZnwjSt11align_val_t`。
- **L1526 EN**: Declares TableGen def `_ZnwjSt11align_val_tRKSt9nothrow_t`.
  **L1526 CN**: 声明 TableGen def `_ZnwjSt11align_val_tRKSt9nothrow_t`。
- **L1527 EN**: Executes a standalone statement or declaration: `: RuntimeLibcallImpl<ZNWJST11ALIGN_VAL_TRKST9NOTHROW_T>;`.
  **L1527 CN**: 执行一条独立语句或声明：`: RuntimeLibcallImpl<ZNWJST11ALIGN_VAL_TRKST9NOTHROW_T>;`。
- **L1528 EN**: Declares TableGen def `_Znwm`.
  **L1528 CN**: 声明 TableGen def `_Znwm`。
- **L1529 EN**: Declares TableGen def `_Znwm12__hot_cold_t`.
  **L1529 CN**: 声明 TableGen def `_Znwm12__hot_cold_t`。
- **L1530 EN**: Declares TableGen def `_ZnwmRKSt9nothrow_t`.
  **L1530 CN**: 声明 TableGen def `_ZnwmRKSt9nothrow_t`。
- **L1531 EN**: Declares TableGen def `_ZnwmRKSt9nothrow_t12__hot_cold_t`.
  **L1531 CN**: 声明 TableGen def `_ZnwmRKSt9nothrow_t12__hot_cold_t`。
- **L1532 EN**: Executes a standalone statement or declaration: `: RuntimeLibcallImpl<ZNWMRKST9NOTHROW_T12__HOT_COLD_T>;`.
  **L1532 CN**: 执行一条独立语句或声明：`: RuntimeLibcallImpl<ZNWMRKST9NOTHROW_T12__HOT_COLD_T>;`。
- **L1533 EN**: Declares TableGen def `_ZnwmSt11align_val_t`.
  **L1533 CN**: 声明 TableGen def `_ZnwmSt11align_val_t`。
- **L1534 EN**: Declares TableGen def `_ZnwmSt11align_val_t12__hot_cold_t`.
  **L1534 CN**: 声明 TableGen def `_ZnwmSt11align_val_t12__hot_cold_t`。
- **L1535 EN**: Executes a standalone statement or declaration: `: RuntimeLibcallImpl<ZNWMST11ALIGN_VAL_T12__HOT_COLD_T>;`.
  **L1535 CN**: 执行一条独立语句或声明：`: RuntimeLibcallImpl<ZNWMST11ALIGN_VAL_T12__HOT_COLD_T>;`。
- **L1536 EN**: Declares TableGen def `_ZnwmSt11align_val_tRKSt9nothrow_t`.
  **L1536 CN**: 声明 TableGen def `_ZnwmSt11align_val_tRKSt9nothrow_t`。

### Lines 1537-1568

````tablegen
    : RuntimeLibcallImpl<ZNWMST11ALIGN_VAL_TRKST9NOTHROW_T>;
def _ZnwmSt11align_val_tRKSt9nothrow_t12__hot_cold_t
    : RuntimeLibcallImpl<ZNWMST11ALIGN_VAL_TRKST9NOTHROW_T12__HOT_COLD_T>;
def __size_returning_new : RuntimeLibcallImpl<SIZE_RETURNING_NEW>;
def __size_returning_new_hot_cold
    : RuntimeLibcallImpl<SIZE_RETURNING_NEW_HOT_COLD>;
def __size_returning_new_aligned
    : RuntimeLibcallImpl<SIZE_RETURNING_NEW_ALIGNED>;
def __size_returning_new_aligned_hot_cold
    : RuntimeLibcallImpl<SIZE_RETURNING_NEW_ALIGNED_HOT_COLD>;
def __cxa_atexit : RuntimeLibcallImpl<CXA_ATEXIT>;
def atexit : RuntimeLibcallImpl<ATEXIT>;
def abort : RuntimeLibcallImpl<ABORT>;
def exit : RuntimeLibcallImpl<EXIT>;
def _Exit : RuntimeLibcallImpl<EXIT>;
def _ZSt9terminatev : RuntimeLibcallImpl<TERMINATE>;
def __cxa_throw : RuntimeLibcallImpl<CXA_THROW>;
def __cxa_guard_abort : RuntimeLibcallImpl<CXA_GUARD_ABORT>;
def __cxa_guard_acquire : RuntimeLibcallImpl<CXA_GUARD_ACQUIRE>;
def __cxa_guard_release : RuntimeLibcallImpl<CXA_GUARD_RELEASE>;
def __isoc99_scanf : RuntimeLibcallImpl<DUNDER_ISOC99_SCANF>;
def __isoc99_sscanf : RuntimeLibcallImpl<DUNDER_ISOC99_SSCANF>;
def __kmpc_alloc_shared : RuntimeLibcallImpl<KMPC_ALLOC_SHARED>;
def __kmpc_free_shared : RuntimeLibcallImpl<KMPC_FREE_SHARED>;
def __memccpy_chk : RuntimeLibcallImpl<MEMCCPY_CHK>;
def __mempcpy_chk : RuntimeLibcallImpl<MEMPCPY_CHK>;
def __small_fprintf : RuntimeLibcallImpl<SMALL_FPRINTF>;
def __small_printf : RuntimeLibcallImpl<SMALL_PRINTF>;
def __small_sprintf : RuntimeLibcallImpl<SMALL_SPRINTF>;
def __snprintf_chk : RuntimeLibcallImpl<SNPRINTF_CHK>;
def __sprintf_chk : RuntimeLibcallImpl<SPRINTF_CHK>;
def __stpcpy_chk : RuntimeLibcallImpl<STPCPY_CHK>;
````
- **L1537 EN**: Executes a standalone statement or declaration: `: RuntimeLibcallImpl<ZNWMST11ALIGN_VAL_TRKST9NOTHROW_T>;`.
  **L1537 CN**: 执行一条独立语句或声明：`: RuntimeLibcallImpl<ZNWMST11ALIGN_VAL_TRKST9NOTHROW_T>;`。
- **L1538 EN**: Declares TableGen def `_ZnwmSt11align_val_tRKSt9nothrow_t12__hot_cold_t`.
  **L1538 CN**: 声明 TableGen def `_ZnwmSt11align_val_tRKSt9nothrow_t12__hot_cold_t`。
- **L1539 EN**: Executes a standalone statement or declaration: `: RuntimeLibcallImpl<ZNWMST11ALIGN_VAL_TRKST9NOTHROW_T12__HOT_COLD_T>;`.
  **L1539 CN**: 执行一条独立语句或声明：`: RuntimeLibcallImpl<ZNWMST11ALIGN_VAL_TRKST9NOTHROW_T12__HOT_COLD_T>;`。
- **L1540 EN**: Declares TableGen def `__size_returning_new`.
  **L1540 CN**: 声明 TableGen def `__size_returning_new`。
- **L1541 EN**: Declares TableGen def `__size_returning_new_hot_cold`.
  **L1541 CN**: 声明 TableGen def `__size_returning_new_hot_cold`。
- **L1542 EN**: Executes a standalone statement or declaration: `: RuntimeLibcallImpl<SIZE_RETURNING_NEW_HOT_COLD>;`.
  **L1542 CN**: 执行一条独立语句或声明：`: RuntimeLibcallImpl<SIZE_RETURNING_NEW_HOT_COLD>;`。
- **L1543 EN**: Declares TableGen def `__size_returning_new_aligned`.
  **L1543 CN**: 声明 TableGen def `__size_returning_new_aligned`。
- **L1544 EN**: Executes a standalone statement or declaration: `: RuntimeLibcallImpl<SIZE_RETURNING_NEW_ALIGNED>;`.
  **L1544 CN**: 执行一条独立语句或声明：`: RuntimeLibcallImpl<SIZE_RETURNING_NEW_ALIGNED>;`。
- **L1545 EN**: Declares TableGen def `__size_returning_new_aligned_hot_cold`.
  **L1545 CN**: 声明 TableGen def `__size_returning_new_aligned_hot_cold`。
- **L1546 EN**: Executes a standalone statement or declaration: `: RuntimeLibcallImpl<SIZE_RETURNING_NEW_ALIGNED_HOT_COLD>;`.
  **L1546 CN**: 执行一条独立语句或声明：`: RuntimeLibcallImpl<SIZE_RETURNING_NEW_ALIGNED_HOT_COLD>;`。
- **L1547 EN**: Declares TableGen def `__cxa_atexit`.
  **L1547 CN**: 声明 TableGen def `__cxa_atexit`。
- **L1548 EN**: Declares TableGen def `atexit`.
  **L1548 CN**: 声明 TableGen def `atexit`。
- **L1549 EN**: Declares TableGen def `abort`.
  **L1549 CN**: 声明 TableGen def `abort`。
- **L1550 EN**: Declares TableGen def `exit`.
  **L1550 CN**: 声明 TableGen def `exit`。
- **L1551 EN**: Declares TableGen def `_Exit`.
  **L1551 CN**: 声明 TableGen def `_Exit`。
- **L1552 EN**: Declares TableGen def `_ZSt9terminatev`.
  **L1552 CN**: 声明 TableGen def `_ZSt9terminatev`。
- **L1553 EN**: Declares TableGen def `__cxa_throw`.
  **L1553 CN**: 声明 TableGen def `__cxa_throw`。
- **L1554 EN**: Declares TableGen def `__cxa_guard_abort`.
  **L1554 CN**: 声明 TableGen def `__cxa_guard_abort`。
- **L1555 EN**: Declares TableGen def `__cxa_guard_acquire`.
  **L1555 CN**: 声明 TableGen def `__cxa_guard_acquire`。
- **L1556 EN**: Declares TableGen def `__cxa_guard_release`.
  **L1556 CN**: 声明 TableGen def `__cxa_guard_release`。
- **L1557 EN**: Declares TableGen def `__isoc99_scanf`.
  **L1557 CN**: 声明 TableGen def `__isoc99_scanf`。
- **L1558 EN**: Declares TableGen def `__isoc99_sscanf`.
  **L1558 CN**: 声明 TableGen def `__isoc99_sscanf`。
- **L1559 EN**: Declares TableGen def `__kmpc_alloc_shared`.
  **L1559 CN**: 声明 TableGen def `__kmpc_alloc_shared`。
- **L1560 EN**: Declares TableGen def `__kmpc_free_shared`.
  **L1560 CN**: 声明 TableGen def `__kmpc_free_shared`。
- **L1561 EN**: Declares TableGen def `__memccpy_chk`.
  **L1561 CN**: 声明 TableGen def `__memccpy_chk`。
- **L1562 EN**: Declares TableGen def `__mempcpy_chk`.
  **L1562 CN**: 声明 TableGen def `__mempcpy_chk`。
- **L1563 EN**: Declares TableGen def `__small_fprintf`.
  **L1563 CN**: 声明 TableGen def `__small_fprintf`。
- **L1564 EN**: Declares TableGen def `__small_printf`.
  **L1564 CN**: 声明 TableGen def `__small_printf`。
- **L1565 EN**: Declares TableGen def `__small_sprintf`.
  **L1565 CN**: 声明 TableGen def `__small_sprintf`。
- **L1566 EN**: Declares TableGen def `__snprintf_chk`.
  **L1566 CN**: 声明 TableGen def `__snprintf_chk`。
- **L1567 EN**: Declares TableGen def `__sprintf_chk`.
  **L1567 CN**: 声明 TableGen def `__sprintf_chk`。
- **L1568 EN**: Declares TableGen def `__stpcpy_chk`.
  **L1568 CN**: 声明 TableGen def `__stpcpy_chk`。

### Lines 1569-1600

````tablegen
def __stpncpy_chk : RuntimeLibcallImpl<STPNCPY_CHK>;
def __strcat_chk : RuntimeLibcallImpl<STRCAT_CHK>;
def __strcpy_chk : RuntimeLibcallImpl<STRCPY_CHK>;
def __strdup : RuntimeLibcallImpl<DUNDER_STRDUP>;
def __strlcat_chk : RuntimeLibcallImpl<STRLCAT_CHK>;
def __strlcpy_chk : RuntimeLibcallImpl<STRLCPY_CHK>;
def __strlen_chk : RuntimeLibcallImpl<STRLEN_CHK>;
def __strncat_chk : RuntimeLibcallImpl<STRNCAT_CHK>;
def __strncpy_chk : RuntimeLibcallImpl<STRNCPY_CHK>;
def __strndup : RuntimeLibcallImpl<DUNDER_STRNDUP>;
def __strtok_r : RuntimeLibcallImpl<DUNDER_STRTOK_R>;
def __vsnprintf_chk : RuntimeLibcallImpl<VSNPRINTF_CHK>;
def __vsprintf_chk : RuntimeLibcallImpl<VSPRINTF_CHK>;
def abs : RuntimeLibcallImpl<ABS>;
def access : RuntimeLibcallImpl<ACCESS>;
def aligned_alloc : RuntimeLibcallImpl<ALIGNED_ALLOC>;
def atof : RuntimeLibcallImpl<ATOF>;
def atoi : RuntimeLibcallImpl<ATOI>;
def atol : RuntimeLibcallImpl<ATOL>;
def atoll : RuntimeLibcallImpl<ATOLL>;
def bcmp : RuntimeLibcallImpl<BCMP>;
def bcopy : RuntimeLibcallImpl<BCOPY>;
def cabs : RuntimeLibcallImpl<CABS_F64>;
def cabsf : RuntimeLibcallImpl<CABS_F32>;
defm cabsl : LibmLongDoubleLibCall;
def chmod : RuntimeLibcallImpl<CHMOD>;
def chown : RuntimeLibcallImpl<CHOWN>;
def clearerr : RuntimeLibcallImpl<CLEARERR>;
def closedir : RuntimeLibcallImpl<CLOSEDIR>;
def ctermid : RuntimeLibcallImpl<CTERMID>;
def execl : RuntimeLibcallImpl<EXECL>;
def execle : RuntimeLibcallImpl<EXECLE>;
````
- **L1569 EN**: Declares TableGen def `__stpncpy_chk`.
  **L1569 CN**: 声明 TableGen def `__stpncpy_chk`。
- **L1570 EN**: Declares TableGen def `__strcat_chk`.
  **L1570 CN**: 声明 TableGen def `__strcat_chk`。
- **L1571 EN**: Declares TableGen def `__strcpy_chk`.
  **L1571 CN**: 声明 TableGen def `__strcpy_chk`。
- **L1572 EN**: Declares TableGen def `__strdup`.
  **L1572 CN**: 声明 TableGen def `__strdup`。
- **L1573 EN**: Declares TableGen def `__strlcat_chk`.
  **L1573 CN**: 声明 TableGen def `__strlcat_chk`。
- **L1574 EN**: Declares TableGen def `__strlcpy_chk`.
  **L1574 CN**: 声明 TableGen def `__strlcpy_chk`。
- **L1575 EN**: Declares TableGen def `__strlen_chk`.
  **L1575 CN**: 声明 TableGen def `__strlen_chk`。
- **L1576 EN**: Declares TableGen def `__strncat_chk`.
  **L1576 CN**: 声明 TableGen def `__strncat_chk`。
- **L1577 EN**: Declares TableGen def `__strncpy_chk`.
  **L1577 CN**: 声明 TableGen def `__strncpy_chk`。
- **L1578 EN**: Declares TableGen def `__strndup`.
  **L1578 CN**: 声明 TableGen def `__strndup`。
- **L1579 EN**: Declares TableGen def `__strtok_r`.
  **L1579 CN**: 声明 TableGen def `__strtok_r`。
- **L1580 EN**: Declares TableGen def `__vsnprintf_chk`.
  **L1580 CN**: 声明 TableGen def `__vsnprintf_chk`。
- **L1581 EN**: Declares TableGen def `__vsprintf_chk`.
  **L1581 CN**: 声明 TableGen def `__vsprintf_chk`。
- **L1582 EN**: Declares TableGen def `abs`.
  **L1582 CN**: 声明 TableGen def `abs`。
- **L1583 EN**: Declares TableGen def `access`.
  **L1583 CN**: 声明 TableGen def `access`。
- **L1584 EN**: Declares TableGen def `aligned_alloc`.
  **L1584 CN**: 声明 TableGen def `aligned_alloc`。
- **L1585 EN**: Declares TableGen def `atof`.
  **L1585 CN**: 声明 TableGen def `atof`。
- **L1586 EN**: Declares TableGen def `atoi`.
  **L1586 CN**: 声明 TableGen def `atoi`。
- **L1587 EN**: Declares TableGen def `atol`.
  **L1587 CN**: 声明 TableGen def `atol`。
- **L1588 EN**: Declares TableGen def `atoll`.
  **L1588 CN**: 声明 TableGen def `atoll`。
- **L1589 EN**: Declares TableGen def `bcmp`.
  **L1589 CN**: 声明 TableGen def `bcmp`。
- **L1590 EN**: Declares TableGen def `bcopy`.
  **L1590 CN**: 声明 TableGen def `bcopy`。
- **L1591 EN**: Declares TableGen def `cabs`.
  **L1591 CN**: 声明 TableGen def `cabs`。
- **L1592 EN**: Declares TableGen def `cabsf`.
  **L1592 CN**: 声明 TableGen def `cabsf`。
- **L1593 EN**: Declares TableGen defm `cabsl`.
  **L1593 CN**: 声明 TableGen defm `cabsl`。
- **L1594 EN**: Declares TableGen def `chmod`.
  **L1594 CN**: 声明 TableGen def `chmod`。
- **L1595 EN**: Declares TableGen def `chown`.
  **L1595 CN**: 声明 TableGen def `chown`。
- **L1596 EN**: Declares TableGen def `clearerr`.
  **L1596 CN**: 声明 TableGen def `clearerr`。
- **L1597 EN**: Declares TableGen def `closedir`.
  **L1597 CN**: 声明 TableGen def `closedir`。
- **L1598 EN**: Declares TableGen def `ctermid`.
  **L1598 CN**: 声明 TableGen def `ctermid`。
- **L1599 EN**: Declares TableGen def `execl`.
  **L1599 CN**: 声明 TableGen def `execl`。
- **L1600 EN**: Declares TableGen def `execle`.
  **L1600 CN**: 声明 TableGen def `execle`。

### Lines 1601-1632

````tablegen
def execlp : RuntimeLibcallImpl<EXECLP>;
def execv : RuntimeLibcallImpl<EXECV>;
def execvP : RuntimeLibcallImpl<EXECVP>;
def execve : RuntimeLibcallImpl<EXECVE>;
def execvp : RuntimeLibcallImpl<EXECVP>;
def execvpe : RuntimeLibcallImpl<EXECVPE>;
def fclose : RuntimeLibcallImpl<FCLOSE>;
def fdopen : RuntimeLibcallImpl<FDOPEN>;
def feof : RuntimeLibcallImpl<FEOF>;
def ferror : RuntimeLibcallImpl<FERROR>;
def fflush : RuntimeLibcallImpl<FFLUSH>;
def ffs : RuntimeLibcallImpl<FFS>;
def ffsl : RuntimeLibcallImpl<FFSL>;
def ffsll : RuntimeLibcallImpl<FFSLL>;
def fgetc : RuntimeLibcallImpl<FGETC>;
def fgetc_unlocked : RuntimeLibcallImpl<FGETC_UNLOCKED>;
def fgetpos : RuntimeLibcallImpl<FGETPOS>;
def fgets : RuntimeLibcallImpl<FGETS>;
def fgets_unlocked : RuntimeLibcallImpl<FGETS_UNLOCKED>;
def fileno : RuntimeLibcallImpl<FILENO>;
def fiprintf : RuntimeLibcallImpl<FIPRINTF>;
def flockfile : RuntimeLibcallImpl<FLOCKFILE>;
def fls : RuntimeLibcallImpl<FLS>;
def flsl : RuntimeLibcallImpl<FLSL>;
def flsll : RuntimeLibcallImpl<FLSLL>;
def fopen : RuntimeLibcallImpl<FOPEN>;
def fopen64 : RuntimeLibcallImpl<FOPEN64>;
def fork : RuntimeLibcallImpl<FORK>;
def fprintf : RuntimeLibcallImpl<FPRINTF>;
def fputc : RuntimeLibcallImpl<FPUTC>;
def fputc_unlocked : RuntimeLibcallImpl<FPUTC_UNLOCKED>;
def fputs : RuntimeLibcallImpl<FPUTS>;
````
- **L1601 EN**: Declares TableGen def `execlp`.
  **L1601 CN**: 声明 TableGen def `execlp`。
- **L1602 EN**: Declares TableGen def `execv`.
  **L1602 CN**: 声明 TableGen def `execv`。
- **L1603 EN**: Declares TableGen def `execvP`.
  **L1603 CN**: 声明 TableGen def `execvP`。
- **L1604 EN**: Declares TableGen def `execve`.
  **L1604 CN**: 声明 TableGen def `execve`。
- **L1605 EN**: Declares TableGen def `execvp`.
  **L1605 CN**: 声明 TableGen def `execvp`。
- **L1606 EN**: Declares TableGen def `execvpe`.
  **L1606 CN**: 声明 TableGen def `execvpe`。
- **L1607 EN**: Declares TableGen def `fclose`.
  **L1607 CN**: 声明 TableGen def `fclose`。
- **L1608 EN**: Declares TableGen def `fdopen`.
  **L1608 CN**: 声明 TableGen def `fdopen`。
- **L1609 EN**: Declares TableGen def `feof`.
  **L1609 CN**: 声明 TableGen def `feof`。
- **L1610 EN**: Declares TableGen def `ferror`.
  **L1610 CN**: 声明 TableGen def `ferror`。
- **L1611 EN**: Declares TableGen def `fflush`.
  **L1611 CN**: 声明 TableGen def `fflush`。
- **L1612 EN**: Declares TableGen def `ffs`.
  **L1612 CN**: 声明 TableGen def `ffs`。
- **L1613 EN**: Declares TableGen def `ffsl`.
  **L1613 CN**: 声明 TableGen def `ffsl`。
- **L1614 EN**: Declares TableGen def `ffsll`.
  **L1614 CN**: 声明 TableGen def `ffsll`。
- **L1615 EN**: Declares TableGen def `fgetc`.
  **L1615 CN**: 声明 TableGen def `fgetc`。
- **L1616 EN**: Declares TableGen def `fgetc_unlocked`.
  **L1616 CN**: 声明 TableGen def `fgetc_unlocked`。
- **L1617 EN**: Declares TableGen def `fgetpos`.
  **L1617 CN**: 声明 TableGen def `fgetpos`。
- **L1618 EN**: Declares TableGen def `fgets`.
  **L1618 CN**: 声明 TableGen def `fgets`。
- **L1619 EN**: Declares TableGen def `fgets_unlocked`.
  **L1619 CN**: 声明 TableGen def `fgets_unlocked`。
- **L1620 EN**: Declares TableGen def `fileno`.
  **L1620 CN**: 声明 TableGen def `fileno`。
- **L1621 EN**: Declares TableGen def `fiprintf`.
  **L1621 CN**: 声明 TableGen def `fiprintf`。
- **L1622 EN**: Declares TableGen def `flockfile`.
  **L1622 CN**: 声明 TableGen def `flockfile`。
- **L1623 EN**: Declares TableGen def `fls`.
  **L1623 CN**: 声明 TableGen def `fls`。
- **L1624 EN**: Declares TableGen def `flsl`.
  **L1624 CN**: 声明 TableGen def `flsl`。
- **L1625 EN**: Declares TableGen def `flsll`.
  **L1625 CN**: 声明 TableGen def `flsll`。
- **L1626 EN**: Declares TableGen def `fopen`.
  **L1626 CN**: 声明 TableGen def `fopen`。
- **L1627 EN**: Declares TableGen def `fopen64`.
  **L1627 CN**: 声明 TableGen def `fopen64`。
- **L1628 EN**: Declares TableGen def `fork`.
  **L1628 CN**: 声明 TableGen def `fork`。
- **L1629 EN**: Declares TableGen def `fprintf`.
  **L1629 CN**: 声明 TableGen def `fprintf`。
- **L1630 EN**: Declares TableGen def `fputc`.
  **L1630 CN**: 声明 TableGen def `fputc`。
- **L1631 EN**: Declares TableGen def `fputc_unlocked`.
  **L1631 CN**: 声明 TableGen def `fputc_unlocked`。
- **L1632 EN**: Declares TableGen def `fputs`.
  **L1632 CN**: 声明 TableGen def `fputs`。

### Lines 1633-1664

````tablegen
def fputs_unlocked : RuntimeLibcallImpl<FPUTS_UNLOCKED>;
def fread : RuntimeLibcallImpl<FREAD>;
def fread_unlocked : RuntimeLibcallImpl<FREAD_UNLOCKED>;
def fscanf : RuntimeLibcallImpl<FSCANF>;
def fseek : RuntimeLibcallImpl<FSEEK>;
def fseeko : RuntimeLibcallImpl<FSEEKO>;
def fseeko64 : RuntimeLibcallImpl<FSEEKO64>;
def fsetpos : RuntimeLibcallImpl<FSETPOS>;
def fstat : RuntimeLibcallImpl<FSTAT>;
def fstat64 : RuntimeLibcallImpl<FSTAT64>;
def fstatvfs : RuntimeLibcallImpl<FSTATVFS>;
def fstatvfs64 : RuntimeLibcallImpl<FSTATVFS64>;
def ftell : RuntimeLibcallImpl<FTELL>;
def ftello : RuntimeLibcallImpl<FTELLO>;
def ftello64 : RuntimeLibcallImpl<FTELLO64>;
def ftrylockfile : RuntimeLibcallImpl<FTRYLOCKFILE>;
def funlockfile : RuntimeLibcallImpl<FUNLOCKFILE>;
def fwrite : RuntimeLibcallImpl<FWRITE>;
def fwrite_unlocked : RuntimeLibcallImpl<FWRITE_UNLOCKED>;
def getc : RuntimeLibcallImpl<GETC>;
def getc_unlocked : RuntimeLibcallImpl<GETC_UNLOCKED>;
def getchar : RuntimeLibcallImpl<GETCHAR>;
def getchar_unlocked : RuntimeLibcallImpl<GETCHAR_UNLOCKED>;
def getenv : RuntimeLibcallImpl<GETENV>;
def getitimer : RuntimeLibcallImpl<GETITIMER>;
def getlogin_r : RuntimeLibcallImpl<GETLOGIN_R>;
def getpwnam : RuntimeLibcallImpl<GETPWNAM>;
def gets : RuntimeLibcallImpl<GETS>;
def gettimeofday : RuntimeLibcallImpl<GETTIMEOFDAY>;
def htonl : RuntimeLibcallImpl<HTONL>;
def htons : RuntimeLibcallImpl<HTONS>;
def iprintf : RuntimeLibcallImpl<IPRINTF>;
````
- **L1633 EN**: Declares TableGen def `fputs_unlocked`.
  **L1633 CN**: 声明 TableGen def `fputs_unlocked`。
- **L1634 EN**: Declares TableGen def `fread`.
  **L1634 CN**: 声明 TableGen def `fread`。
- **L1635 EN**: Declares TableGen def `fread_unlocked`.
  **L1635 CN**: 声明 TableGen def `fread_unlocked`。
- **L1636 EN**: Declares TableGen def `fscanf`.
  **L1636 CN**: 声明 TableGen def `fscanf`。
- **L1637 EN**: Declares TableGen def `fseek`.
  **L1637 CN**: 声明 TableGen def `fseek`。
- **L1638 EN**: Declares TableGen def `fseeko`.
  **L1638 CN**: 声明 TableGen def `fseeko`。
- **L1639 EN**: Declares TableGen def `fseeko64`.
  **L1639 CN**: 声明 TableGen def `fseeko64`。
- **L1640 EN**: Declares TableGen def `fsetpos`.
  **L1640 CN**: 声明 TableGen def `fsetpos`。
- **L1641 EN**: Declares TableGen def `fstat`.
  **L1641 CN**: 声明 TableGen def `fstat`。
- **L1642 EN**: Declares TableGen def `fstat64`.
  **L1642 CN**: 声明 TableGen def `fstat64`。
- **L1643 EN**: Declares TableGen def `fstatvfs`.
  **L1643 CN**: 声明 TableGen def `fstatvfs`。
- **L1644 EN**: Declares TableGen def `fstatvfs64`.
  **L1644 CN**: 声明 TableGen def `fstatvfs64`。
- **L1645 EN**: Declares TableGen def `ftell`.
  **L1645 CN**: 声明 TableGen def `ftell`。
- **L1646 EN**: Declares TableGen def `ftello`.
  **L1646 CN**: 声明 TableGen def `ftello`。
- **L1647 EN**: Declares TableGen def `ftello64`.
  **L1647 CN**: 声明 TableGen def `ftello64`。
- **L1648 EN**: Declares TableGen def `ftrylockfile`.
  **L1648 CN**: 声明 TableGen def `ftrylockfile`。
- **L1649 EN**: Declares TableGen def `funlockfile`.
  **L1649 CN**: 声明 TableGen def `funlockfile`。
- **L1650 EN**: Declares TableGen def `fwrite`.
  **L1650 CN**: 声明 TableGen def `fwrite`。
- **L1651 EN**: Declares TableGen def `fwrite_unlocked`.
  **L1651 CN**: 声明 TableGen def `fwrite_unlocked`。
- **L1652 EN**: Declares TableGen def `getc`.
  **L1652 CN**: 声明 TableGen def `getc`。
- **L1653 EN**: Declares TableGen def `getc_unlocked`.
  **L1653 CN**: 声明 TableGen def `getc_unlocked`。
- **L1654 EN**: Declares TableGen def `getchar`.
  **L1654 CN**: 声明 TableGen def `getchar`。
- **L1655 EN**: Declares TableGen def `getchar_unlocked`.
  **L1655 CN**: 声明 TableGen def `getchar_unlocked`。
- **L1656 EN**: Declares TableGen def `getenv`.
  **L1656 CN**: 声明 TableGen def `getenv`。
- **L1657 EN**: Declares TableGen def `getitimer`.
  **L1657 CN**: 声明 TableGen def `getitimer`。
- **L1658 EN**: Declares TableGen def `getlogin_r`.
  **L1658 CN**: 声明 TableGen def `getlogin_r`。
- **L1659 EN**: Declares TableGen def `getpwnam`.
  **L1659 CN**: 声明 TableGen def `getpwnam`。
- **L1660 EN**: Declares TableGen def `gets`.
  **L1660 CN**: 声明 TableGen def `gets`。
- **L1661 EN**: Declares TableGen def `gettimeofday`.
  **L1661 CN**: 声明 TableGen def `gettimeofday`。
- **L1662 EN**: Declares TableGen def `htonl`.
  **L1662 CN**: 声明 TableGen def `htonl`。
- **L1663 EN**: Declares TableGen def `htons`.
  **L1663 CN**: 声明 TableGen def `htons`。
- **L1664 EN**: Declares TableGen def `iprintf`.
  **L1664 CN**: 声明 TableGen def `iprintf`。

### Lines 1665-1696

````tablegen
def isascii : RuntimeLibcallImpl<ISASCII>;
def isdigit : RuntimeLibcallImpl<ISDIGIT>;
def labs : RuntimeLibcallImpl<LABS>;
def lchown : RuntimeLibcallImpl<LCHOWN>;
def llabs : RuntimeLibcallImpl<LLABS>;
def lstat : RuntimeLibcallImpl<LSTAT>;
def lstat64 : RuntimeLibcallImpl<LSTAT64>;
def memalign : RuntimeLibcallImpl<MEMALIGN>;
def memccpy : RuntimeLibcallImpl<MEMCCPY>;
def memchr : RuntimeLibcallImpl<MEMCHR>;
def memcmp : RuntimeLibcallImpl<MEMCMP>;
def mempcpy : RuntimeLibcallImpl<MEMPCPY>;
def memrchr : RuntimeLibcallImpl<MEMRCHR>;
def memset_pattern16 : RuntimeLibcallImpl<MEMSET_PATTERN16>;
def memset_pattern4 : RuntimeLibcallImpl<MEMSET_PATTERN4>;
def memset_pattern8 : RuntimeLibcallImpl<MEMSET_PATTERN8>;
def mkdir : RuntimeLibcallImpl<MKDIR>;
def mktime : RuntimeLibcallImpl<MKTIME>;
def ntohl : RuntimeLibcallImpl<NTOHL>;
def ntohs : RuntimeLibcallImpl<NTOHS>;
def open : RuntimeLibcallImpl<OPEN>;
def open64 : RuntimeLibcallImpl<OPEN64>;
def opendir : RuntimeLibcallImpl<OPENDIR>;
def pclose : RuntimeLibcallImpl<PCLOSE>;
def perror : RuntimeLibcallImpl<PERROR>;
def popen : RuntimeLibcallImpl<POPEN>;
def posix_memalign : RuntimeLibcallImpl<POSIX_MEMALIGN>;
def pread : RuntimeLibcallImpl<PREAD>;
def printf : RuntimeLibcallImpl<PRINTF>;
def putc : RuntimeLibcallImpl<PUTC>;
def putc_unlocked : RuntimeLibcallImpl<PUTC_UNLOCKED>;
def putchar : RuntimeLibcallImpl<PUTCHAR>;
````
- **L1665 EN**: Declares TableGen def `isascii`.
  **L1665 CN**: 声明 TableGen def `isascii`。
- **L1666 EN**: Declares TableGen def `isdigit`.
  **L1666 CN**: 声明 TableGen def `isdigit`。
- **L1667 EN**: Declares TableGen def `labs`.
  **L1667 CN**: 声明 TableGen def `labs`。
- **L1668 EN**: Declares TableGen def `lchown`.
  **L1668 CN**: 声明 TableGen def `lchown`。
- **L1669 EN**: Declares TableGen def `llabs`.
  **L1669 CN**: 声明 TableGen def `llabs`。
- **L1670 EN**: Declares TableGen def `lstat`.
  **L1670 CN**: 声明 TableGen def `lstat`。
- **L1671 EN**: Declares TableGen def `lstat64`.
  **L1671 CN**: 声明 TableGen def `lstat64`。
- **L1672 EN**: Declares TableGen def `memalign`.
  **L1672 CN**: 声明 TableGen def `memalign`。
- **L1673 EN**: Declares TableGen def `memccpy`.
  **L1673 CN**: 声明 TableGen def `memccpy`。
- **L1674 EN**: Declares TableGen def `memchr`.
  **L1674 CN**: 声明 TableGen def `memchr`。
- **L1675 EN**: Declares TableGen def `memcmp`.
  **L1675 CN**: 声明 TableGen def `memcmp`。
- **L1676 EN**: Declares TableGen def `mempcpy`.
  **L1676 CN**: 声明 TableGen def `mempcpy`。
- **L1677 EN**: Declares TableGen def `memrchr`.
  **L1677 CN**: 声明 TableGen def `memrchr`。
- **L1678 EN**: Declares TableGen def `memset_pattern16`.
  **L1678 CN**: 声明 TableGen def `memset_pattern16`。
- **L1679 EN**: Declares TableGen def `memset_pattern4`.
  **L1679 CN**: 声明 TableGen def `memset_pattern4`。
- **L1680 EN**: Declares TableGen def `memset_pattern8`.
  **L1680 CN**: 声明 TableGen def `memset_pattern8`。
- **L1681 EN**: Declares TableGen def `mkdir`.
  **L1681 CN**: 声明 TableGen def `mkdir`。
- **L1682 EN**: Declares TableGen def `mktime`.
  **L1682 CN**: 声明 TableGen def `mktime`。
- **L1683 EN**: Declares TableGen def `ntohl`.
  **L1683 CN**: 声明 TableGen def `ntohl`。
- **L1684 EN**: Declares TableGen def `ntohs`.
  **L1684 CN**: 声明 TableGen def `ntohs`。
- **L1685 EN**: Declares TableGen def `open`.
  **L1685 CN**: 声明 TableGen def `open`。
- **L1686 EN**: Declares TableGen def `open64`.
  **L1686 CN**: 声明 TableGen def `open64`。
- **L1687 EN**: Declares TableGen def `opendir`.
  **L1687 CN**: 声明 TableGen def `opendir`。
- **L1688 EN**: Declares TableGen def `pclose`.
  **L1688 CN**: 声明 TableGen def `pclose`。
- **L1689 EN**: Declares TableGen def `perror`.
  **L1689 CN**: 声明 TableGen def `perror`。
- **L1690 EN**: Declares TableGen def `popen`.
  **L1690 CN**: 声明 TableGen def `popen`。
- **L1691 EN**: Declares TableGen def `posix_memalign`.
  **L1691 CN**: 声明 TableGen def `posix_memalign`。
- **L1692 EN**: Declares TableGen def `pread`.
  **L1692 CN**: 声明 TableGen def `pread`。
- **L1693 EN**: Declares TableGen def `printf`.
  **L1693 CN**: 声明 TableGen def `printf`。
- **L1694 EN**: Declares TableGen def `putc`.
  **L1694 CN**: 声明 TableGen def `putc`。
- **L1695 EN**: Declares TableGen def `putc_unlocked`.
  **L1695 CN**: 声明 TableGen def `putc_unlocked`。
- **L1696 EN**: Declares TableGen def `putchar`.
  **L1696 CN**: 声明 TableGen def `putchar`。

### Lines 1697-1728

````tablegen
def putchar_unlocked : RuntimeLibcallImpl<PUTCHAR_UNLOCKED>;
def puts : RuntimeLibcallImpl<PUTS>;
def pvalloc : RuntimeLibcallImpl<PVALLOC>;
def pwrite : RuntimeLibcallImpl<PWRITE>;
def qsort : RuntimeLibcallImpl<QSORT>;
def read : RuntimeLibcallImpl<READ>;
def readlink : RuntimeLibcallImpl<READLINK>;
def realloc : RuntimeLibcallImpl<REALLOC>;
def reallocf : RuntimeLibcallImpl<REALLOCF>;
def reallocarray : RuntimeLibcallImpl<REALLOCARRAY>;
def realpath : RuntimeLibcallImpl<REALPATH>;
def remove : RuntimeLibcallImpl<REMOVE>;
def rename : RuntimeLibcallImpl<RENAME>;
def rewind : RuntimeLibcallImpl<REWIND>;
def rmdir : RuntimeLibcallImpl<RMDIR>;
def scanf : RuntimeLibcallImpl<SCANF>;
def setbuf : RuntimeLibcallImpl<SETBUF>;
def setitimer : RuntimeLibcallImpl<SETITIMER>;
def setvbuf : RuntimeLibcallImpl<SETVBUF>;
def siprintf : RuntimeLibcallImpl<SIPRINTF>;
def snprintf : RuntimeLibcallImpl<SNPRINTF>;
def sprintf : RuntimeLibcallImpl<SPRINTF>;
def sscanf : RuntimeLibcallImpl<SSCANF>;
def stat : RuntimeLibcallImpl<STAT>;
def stat64 : RuntimeLibcallImpl<STAT64>;
def statvfs : RuntimeLibcallImpl<STATVFS>;
def statvfs64 : RuntimeLibcallImpl<STATVFS64>;
def stpcpy : RuntimeLibcallImpl<STPCPY>;
def stpncpy : RuntimeLibcallImpl<STPNCPY>;
def strcasecmp : RuntimeLibcallImpl<STRCASECMP>;
def strcat : RuntimeLibcallImpl<STRCAT>;
def strchr : RuntimeLibcallImpl<STRCHR>;
````
- **L1697 EN**: Declares TableGen def `putchar_unlocked`.
  **L1697 CN**: 声明 TableGen def `putchar_unlocked`。
- **L1698 EN**: Declares TableGen def `puts`.
  **L1698 CN**: 声明 TableGen def `puts`。
- **L1699 EN**: Declares TableGen def `pvalloc`.
  **L1699 CN**: 声明 TableGen def `pvalloc`。
- **L1700 EN**: Declares TableGen def `pwrite`.
  **L1700 CN**: 声明 TableGen def `pwrite`。
- **L1701 EN**: Declares TableGen def `qsort`.
  **L1701 CN**: 声明 TableGen def `qsort`。
- **L1702 EN**: Declares TableGen def `read`.
  **L1702 CN**: 声明 TableGen def `read`。
- **L1703 EN**: Declares TableGen def `readlink`.
  **L1703 CN**: 声明 TableGen def `readlink`。
- **L1704 EN**: Declares TableGen def `realloc`.
  **L1704 CN**: 声明 TableGen def `realloc`。
- **L1705 EN**: Declares TableGen def `reallocf`.
  **L1705 CN**: 声明 TableGen def `reallocf`。
- **L1706 EN**: Declares TableGen def `reallocarray`.
  **L1706 CN**: 声明 TableGen def `reallocarray`。
- **L1707 EN**: Declares TableGen def `realpath`.
  **L1707 CN**: 声明 TableGen def `realpath`。
- **L1708 EN**: Declares TableGen def `remove`.
  **L1708 CN**: 声明 TableGen def `remove`。
- **L1709 EN**: Declares TableGen def `rename`.
  **L1709 CN**: 声明 TableGen def `rename`。
- **L1710 EN**: Declares TableGen def `rewind`.
  **L1710 CN**: 声明 TableGen def `rewind`。
- **L1711 EN**: Declares TableGen def `rmdir`.
  **L1711 CN**: 声明 TableGen def `rmdir`。
- **L1712 EN**: Declares TableGen def `scanf`.
  **L1712 CN**: 声明 TableGen def `scanf`。
- **L1713 EN**: Declares TableGen def `setbuf`.
  **L1713 CN**: 声明 TableGen def `setbuf`。
- **L1714 EN**: Declares TableGen def `setitimer`.
  **L1714 CN**: 声明 TableGen def `setitimer`。
- **L1715 EN**: Declares TableGen def `setvbuf`.
  **L1715 CN**: 声明 TableGen def `setvbuf`。
- **L1716 EN**: Declares TableGen def `siprintf`.
  **L1716 CN**: 声明 TableGen def `siprintf`。
- **L1717 EN**: Declares TableGen def `snprintf`.
  **L1717 CN**: 声明 TableGen def `snprintf`。
- **L1718 EN**: Declares TableGen def `sprintf`.
  **L1718 CN**: 声明 TableGen def `sprintf`。
- **L1719 EN**: Declares TableGen def `sscanf`.
  **L1719 CN**: 声明 TableGen def `sscanf`。
- **L1720 EN**: Declares TableGen def `stat`.
  **L1720 CN**: 声明 TableGen def `stat`。
- **L1721 EN**: Declares TableGen def `stat64`.
  **L1721 CN**: 声明 TableGen def `stat64`。
- **L1722 EN**: Declares TableGen def `statvfs`.
  **L1722 CN**: 声明 TableGen def `statvfs`。
- **L1723 EN**: Declares TableGen def `statvfs64`.
  **L1723 CN**: 声明 TableGen def `statvfs64`。
- **L1724 EN**: Declares TableGen def `stpcpy`.
  **L1724 CN**: 声明 TableGen def `stpcpy`。
- **L1725 EN**: Declares TableGen def `stpncpy`.
  **L1725 CN**: 声明 TableGen def `stpncpy`。
- **L1726 EN**: Declares TableGen def `strcasecmp`.
  **L1726 CN**: 声明 TableGen def `strcasecmp`。
- **L1727 EN**: Declares TableGen def `strcat`.
  **L1727 CN**: 声明 TableGen def `strcat`。
- **L1728 EN**: Declares TableGen def `strchr`.
  **L1728 CN**: 声明 TableGen def `strchr`。

### Lines 1729-1760

````tablegen
def strcmp : RuntimeLibcallImpl<STRCMP>;
def strcoll : RuntimeLibcallImpl<STRCOLL>;
def strcpy : RuntimeLibcallImpl<STRCPY>;
def strcspn : RuntimeLibcallImpl<STRCSPN>;
def strdup : RuntimeLibcallImpl<STRDUP>;
def strlcat : RuntimeLibcallImpl<STRLCAT>;
def strlcpy : RuntimeLibcallImpl<STRLCPY>;
def strlen : RuntimeLibcallImpl<STRLEN>;
def strncasecmp : RuntimeLibcallImpl<STRNCASECMP>;
def strncat : RuntimeLibcallImpl<STRNCAT>;
def strncmp : RuntimeLibcallImpl<STRNCMP>;
def strncpy : RuntimeLibcallImpl<STRNCPY>;
def strndup : RuntimeLibcallImpl<STRNDUP>;
def strnlen : RuntimeLibcallImpl<STRNLEN>;
def strpbrk : RuntimeLibcallImpl<STRPBRK>;
def strrchr : RuntimeLibcallImpl<STRRCHR>;
def strspn : RuntimeLibcallImpl<STRSPN>;
def strstr : RuntimeLibcallImpl<STRSTR>;
def strtod : RuntimeLibcallImpl<STRTOD>;
def strtof : RuntimeLibcallImpl<STRTOF>;
def strtok : RuntimeLibcallImpl<STRTOK>;
def strtok_r : RuntimeLibcallImpl<STRTOK_R>;
def strtol : RuntimeLibcallImpl<STRTOL>;
def strtold : RuntimeLibcallImpl<STRTOLD>;
def strtoll : RuntimeLibcallImpl<STRTOLL>;
def strtoul : RuntimeLibcallImpl<STRTOUL>;
def strtoull : RuntimeLibcallImpl<STRTOULL>;
def strxfrm : RuntimeLibcallImpl<STRXFRM>;
def system : RuntimeLibcallImpl<SYSTEM>;
def times : RuntimeLibcallImpl<TIMES>;
def tmpfile : RuntimeLibcallImpl<TMPFILE>;
def tmpfile64 : RuntimeLibcallImpl<TMPFILE64>;
````
- **L1729 EN**: Declares TableGen def `strcmp`.
  **L1729 CN**: 声明 TableGen def `strcmp`。
- **L1730 EN**: Declares TableGen def `strcoll`.
  **L1730 CN**: 声明 TableGen def `strcoll`。
- **L1731 EN**: Declares TableGen def `strcpy`.
  **L1731 CN**: 声明 TableGen def `strcpy`。
- **L1732 EN**: Declares TableGen def `strcspn`.
  **L1732 CN**: 声明 TableGen def `strcspn`。
- **L1733 EN**: Declares TableGen def `strdup`.
  **L1733 CN**: 声明 TableGen def `strdup`。
- **L1734 EN**: Declares TableGen def `strlcat`.
  **L1734 CN**: 声明 TableGen def `strlcat`。
- **L1735 EN**: Declares TableGen def `strlcpy`.
  **L1735 CN**: 声明 TableGen def `strlcpy`。
- **L1736 EN**: Declares TableGen def `strlen`.
  **L1736 CN**: 声明 TableGen def `strlen`。
- **L1737 EN**: Declares TableGen def `strncasecmp`.
  **L1737 CN**: 声明 TableGen def `strncasecmp`。
- **L1738 EN**: Declares TableGen def `strncat`.
  **L1738 CN**: 声明 TableGen def `strncat`。
- **L1739 EN**: Declares TableGen def `strncmp`.
  **L1739 CN**: 声明 TableGen def `strncmp`。
- **L1740 EN**: Declares TableGen def `strncpy`.
  **L1740 CN**: 声明 TableGen def `strncpy`。
- **L1741 EN**: Declares TableGen def `strndup`.
  **L1741 CN**: 声明 TableGen def `strndup`。
- **L1742 EN**: Declares TableGen def `strnlen`.
  **L1742 CN**: 声明 TableGen def `strnlen`。
- **L1743 EN**: Declares TableGen def `strpbrk`.
  **L1743 CN**: 声明 TableGen def `strpbrk`。
- **L1744 EN**: Declares TableGen def `strrchr`.
  **L1744 CN**: 声明 TableGen def `strrchr`。
- **L1745 EN**: Declares TableGen def `strspn`.
  **L1745 CN**: 声明 TableGen def `strspn`。
- **L1746 EN**: Declares TableGen def `strstr`.
  **L1746 CN**: 声明 TableGen def `strstr`。
- **L1747 EN**: Declares TableGen def `strtod`.
  **L1747 CN**: 声明 TableGen def `strtod`。
- **L1748 EN**: Declares TableGen def `strtof`.
  **L1748 CN**: 声明 TableGen def `strtof`。
- **L1749 EN**: Declares TableGen def `strtok`.
  **L1749 CN**: 声明 TableGen def `strtok`。
- **L1750 EN**: Declares TableGen def `strtok_r`.
  **L1750 CN**: 声明 TableGen def `strtok_r`。
- **L1751 EN**: Declares TableGen def `strtol`.
  **L1751 CN**: 声明 TableGen def `strtol`。
- **L1752 EN**: Declares TableGen def `strtold`.
  **L1752 CN**: 声明 TableGen def `strtold`。
- **L1753 EN**: Declares TableGen def `strtoll`.
  **L1753 CN**: 声明 TableGen def `strtoll`。
- **L1754 EN**: Declares TableGen def `strtoul`.
  **L1754 CN**: 声明 TableGen def `strtoul`。
- **L1755 EN**: Declares TableGen def `strtoull`.
  **L1755 CN**: 声明 TableGen def `strtoull`。
- **L1756 EN**: Declares TableGen def `strxfrm`.
  **L1756 CN**: 声明 TableGen def `strxfrm`。
- **L1757 EN**: Declares TableGen def `system`.
  **L1757 CN**: 声明 TableGen def `system`。
- **L1758 EN**: Declares TableGen def `times`.
  **L1758 CN**: 声明 TableGen def `times`。
- **L1759 EN**: Declares TableGen def `tmpfile`.
  **L1759 CN**: 声明 TableGen def `tmpfile`。
- **L1760 EN**: Declares TableGen def `tmpfile64`.
  **L1760 CN**: 声明 TableGen def `tmpfile64`。

### Lines 1761-1792

````tablegen
def toascii : RuntimeLibcallImpl<TOASCII>;
def uname : RuntimeLibcallImpl<UNAME>;
def ungetc : RuntimeLibcallImpl<UNGETC>;
def unlink : RuntimeLibcallImpl<UNLINK>;
def unsetenv : RuntimeLibcallImpl<UNSETENV>;
def utime : RuntimeLibcallImpl<UTIME>;
def utimes : RuntimeLibcallImpl<UTIMES>;
def valloc : RuntimeLibcallImpl<VALLOC>;
def vec_calloc : RuntimeLibcallImpl<VEC_CALLOC>;
def vec_free : RuntimeLibcallImpl<VEC_FREE>;
def vec_malloc : RuntimeLibcallImpl<VEC_MALLOC>;
def vec_realloc : RuntimeLibcallImpl<VEC_REALLOC>;
def vfprintf : RuntimeLibcallImpl<VFPRINTF>;
def vfscanf : RuntimeLibcallImpl<VFSCANF>;
def vprintf : RuntimeLibcallImpl<VPRINTF>;
def vscanf : RuntimeLibcallImpl<VSCANF>;
def vsnprintf : RuntimeLibcallImpl<VSNPRINTF>;
def vsprintf : RuntimeLibcallImpl<VSPRINTF>;
def vsscanf : RuntimeLibcallImpl<VSSCANF>;
def wcslen : RuntimeLibcallImpl<WCSLEN>;
def write : RuntimeLibcallImpl<WRITE>;

//--------------------------------------------------------------------
// compiler-rt/libgcc but 64-bit only, not available by default
//--------------------------------------------------------------------

// Exist in libgcc and compiler-rt for 64-bit targets, or if
// COMPILER_RT_ENABLE_SOFTWARE_INT128.
defset list<RuntimeLibcallImpl> Int128RTLibcalls = {
  def __ashlti3 : RuntimeLibcallImpl<SHL_I128>;
  def __lshrti3 : RuntimeLibcallImpl<SRL_I128>;
  def __ashrti3 : RuntimeLibcallImpl<SRA_I128>;
````
- **L1761 EN**: Declares TableGen def `toascii`.
  **L1761 CN**: 声明 TableGen def `toascii`。
- **L1762 EN**: Declares TableGen def `uname`.
  **L1762 CN**: 声明 TableGen def `uname`。
- **L1763 EN**: Declares TableGen def `ungetc`.
  **L1763 CN**: 声明 TableGen def `ungetc`。
- **L1764 EN**: Declares TableGen def `unlink`.
  **L1764 CN**: 声明 TableGen def `unlink`。
- **L1765 EN**: Declares TableGen def `unsetenv`.
  **L1765 CN**: 声明 TableGen def `unsetenv`。
- **L1766 EN**: Declares TableGen def `utime`.
  **L1766 CN**: 声明 TableGen def `utime`。
- **L1767 EN**: Declares TableGen def `utimes`.
  **L1767 CN**: 声明 TableGen def `utimes`。
- **L1768 EN**: Declares TableGen def `valloc`.
  **L1768 CN**: 声明 TableGen def `valloc`。
- **L1769 EN**: Declares TableGen def `vec_calloc`.
  **L1769 CN**: 声明 TableGen def `vec_calloc`。
- **L1770 EN**: Declares TableGen def `vec_free`.
  **L1770 CN**: 声明 TableGen def `vec_free`。
- **L1771 EN**: Declares TableGen def `vec_malloc`.
  **L1771 CN**: 声明 TableGen def `vec_malloc`。
- **L1772 EN**: Declares TableGen def `vec_realloc`.
  **L1772 CN**: 声明 TableGen def `vec_realloc`。
- **L1773 EN**: Declares TableGen def `vfprintf`.
  **L1773 CN**: 声明 TableGen def `vfprintf`。
- **L1774 EN**: Declares TableGen def `vfscanf`.
  **L1774 CN**: 声明 TableGen def `vfscanf`。
- **L1775 EN**: Declares TableGen def `vprintf`.
  **L1775 CN**: 声明 TableGen def `vprintf`。
- **L1776 EN**: Declares TableGen def `vscanf`.
  **L1776 CN**: 声明 TableGen def `vscanf`。
- **L1777 EN**: Declares TableGen def `vsnprintf`.
  **L1777 CN**: 声明 TableGen def `vsnprintf`。
- **L1778 EN**: Declares TableGen def `vsprintf`.
  **L1778 CN**: 声明 TableGen def `vsprintf`。
- **L1779 EN**: Declares TableGen def `vsscanf`.
  **L1779 CN**: 声明 TableGen def `vsscanf`。
- **L1780 EN**: Declares TableGen def `wcslen`.
  **L1780 CN**: 声明 TableGen def `wcslen`。
- **L1781 EN**: Declares TableGen def `write`.
  **L1781 CN**: 声明 TableGen def `write`。
- **L1782 EN**: Blank line separating nearby declarations or logic blocks.
  **L1782 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1783 EN**: Comment explains nearby logic, invariants, or intent: `--------------------------------------------------------------------`.
  **L1783 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`--------------------------------------------------------------------`。
- **L1784 EN**: Comment explains nearby logic, invariants, or intent: `compiler-rt/libgcc but 64-bit only, not available by default`.
  **L1784 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`compiler-rt/libgcc but 64-bit only, not available by default`。
- **L1785 EN**: Comment explains nearby logic, invariants, or intent: `--------------------------------------------------------------------`.
  **L1785 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`--------------------------------------------------------------------`。
- **L1786 EN**: Blank line separating nearby declarations or logic blocks.
  **L1786 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1787 EN**: Comment explains nearby logic, invariants, or intent: `Exist in libgcc and compiler-rt for 64-bit targets, or if`.
  **L1787 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Exist in libgcc and compiler-rt for 64-bit targets, or if`。
- **L1788 EN**: Comment explains nearby logic, invariants, or intent: `COMPILER_RT_ENABLE_SOFTWARE_INT128.`.
  **L1788 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`COMPILER_RT_ENABLE_SOFTWARE_INT128.`。
- **L1789 EN**: Continues the surrounding expression or declaration: `defset list<RuntimeLibcallImpl> Int128RTLibcalls = {`.
  **L1789 CN**: 继续构造周围的表达式或声明：`defset list<RuntimeLibcallImpl> Int128RTLibcalls = {`。
- **L1790 EN**: Declares TableGen def `__ashlti3`.
  **L1790 CN**: 声明 TableGen def `__ashlti3`。
- **L1791 EN**: Declares TableGen def `__lshrti3`.
  **L1791 CN**: 声明 TableGen def `__lshrti3`。
- **L1792 EN**: Declares TableGen def `__ashrti3`.
  **L1792 CN**: 声明 TableGen def `__ashrti3`。

### Lines 1793-1824

````tablegen
  def __multi3 : RuntimeLibcallImpl<MUL_I128>;
}

//--------------------------------------------------------------------
// compiler-rt only, not available by default
//--------------------------------------------------------------------

defset list<RuntimeLibcallImpl> CompilerRTOnlyInt64Libcalls = {
  def __mulodi4 : RuntimeLibcallImpl<MULO_I64>;
}

defset list<RuntimeLibcallImpl> CompilerRTOnlyInt128Libcalls = {
  def __muloti4 : RuntimeLibcallImpl<MULO_I128>;
}

//--------------------------------------------------------------------
// Define implementation other libcalls
//--------------------------------------------------------------------

// TODO: Define other custom names targets use to override

def __exp10f : RuntimeLibcallImpl<EXP10_F32>;
def __exp10 : RuntimeLibcallImpl<EXP10_F64>;

def __sincosf_stret : RuntimeLibcallImpl<SINCOS_STRET_F32>;
def __sincos_stret : RuntimeLibcallImpl<SINCOS_STRET_F64>;

def sincosf : RuntimeLibcallImpl<SINCOS_F32>;
def sincos : RuntimeLibcallImpl<SINCOS_F64>;
defm sincosl : LibmLongDoubleLibCall;

// Exists in sun math library
````
- **L1793 EN**: Declares TableGen def `__multi3`.
  **L1793 CN**: 声明 TableGen def `__multi3`。
- **L1794 EN**: Closes the current lexical scope or compound statement.
  **L1794 CN**: 结束当前词法作用域或复合语句块。
- **L1795 EN**: Blank line separating nearby declarations or logic blocks.
  **L1795 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1796 EN**: Comment explains nearby logic, invariants, or intent: `--------------------------------------------------------------------`.
  **L1796 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`--------------------------------------------------------------------`。
- **L1797 EN**: Comment explains nearby logic, invariants, or intent: `compiler-rt only, not available by default`.
  **L1797 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`compiler-rt only, not available by default`。
- **L1798 EN**: Comment explains nearby logic, invariants, or intent: `--------------------------------------------------------------------`.
  **L1798 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`--------------------------------------------------------------------`。
- **L1799 EN**: Blank line separating nearby declarations or logic blocks.
  **L1799 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1800 EN**: Continues the surrounding expression or declaration: `defset list<RuntimeLibcallImpl> CompilerRTOnlyInt64Libcalls = {`.
  **L1800 CN**: 继续构造周围的表达式或声明：`defset list<RuntimeLibcallImpl> CompilerRTOnlyInt64Libcalls = {`。
- **L1801 EN**: Declares TableGen def `__mulodi4`.
  **L1801 CN**: 声明 TableGen def `__mulodi4`。
- **L1802 EN**: Closes the current lexical scope or compound statement.
  **L1802 CN**: 结束当前词法作用域或复合语句块。
- **L1803 EN**: Blank line separating nearby declarations or logic blocks.
  **L1803 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1804 EN**: Continues the surrounding expression or declaration: `defset list<RuntimeLibcallImpl> CompilerRTOnlyInt128Libcalls = {`.
  **L1804 CN**: 继续构造周围的表达式或声明：`defset list<RuntimeLibcallImpl> CompilerRTOnlyInt128Libcalls = {`。
- **L1805 EN**: Declares TableGen def `__muloti4`.
  **L1805 CN**: 声明 TableGen def `__muloti4`。
- **L1806 EN**: Closes the current lexical scope or compound statement.
  **L1806 CN**: 结束当前词法作用域或复合语句块。
- **L1807 EN**: Blank line separating nearby declarations or logic blocks.
  **L1807 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1808 EN**: Comment explains nearby logic, invariants, or intent: `--------------------------------------------------------------------`.
  **L1808 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`--------------------------------------------------------------------`。
- **L1809 EN**: Comment explains nearby logic, invariants, or intent: `Define implementation other libcalls`.
  **L1809 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Define implementation other libcalls`。
- **L1810 EN**: Comment explains nearby logic, invariants, or intent: `--------------------------------------------------------------------`.
  **L1810 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`--------------------------------------------------------------------`。
- **L1811 EN**: Blank line separating nearby declarations or logic blocks.
  **L1811 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1812 EN**: Comment records a pending task or caution: `TODO: Define other custom names targets use to override`.
  **L1812 CN**: 注释记录了待办事项或注意点：`TODO: Define other custom names targets use to override`。
- **L1813 EN**: Blank line separating nearby declarations or logic blocks.
  **L1813 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1814 EN**: Declares TableGen def `__exp10f`.
  **L1814 CN**: 声明 TableGen def `__exp10f`。
- **L1815 EN**: Declares TableGen def `__exp10`.
  **L1815 CN**: 声明 TableGen def `__exp10`。
- **L1816 EN**: Blank line separating nearby declarations or logic blocks.
  **L1816 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1817 EN**: Declares TableGen def `__sincosf_stret`.
  **L1817 CN**: 声明 TableGen def `__sincosf_stret`。
- **L1818 EN**: Declares TableGen def `__sincos_stret`.
  **L1818 CN**: 声明 TableGen def `__sincos_stret`。
- **L1819 EN**: Blank line separating nearby declarations or logic blocks.
  **L1819 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1820 EN**: Declares TableGen def `sincosf`.
  **L1820 CN**: 声明 TableGen def `sincosf`。
- **L1821 EN**: Declares TableGen def `sincos`.
  **L1821 CN**: 声明 TableGen def `sincos`。
- **L1822 EN**: Declares TableGen defm `sincosl`.
  **L1822 CN**: 声明 TableGen defm `sincosl`。
- **L1823 EN**: Blank line separating nearby declarations or logic blocks.
  **L1823 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1824 EN**: Comment explains nearby logic, invariants, or intent: `Exists in sun math library`.
  **L1824 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Exists in sun math library`。

### Lines 1825-1856

````tablegen
def sincospif : RuntimeLibcallImpl<SINCOSPI_F32>;
def sincospi : RuntimeLibcallImpl<SINCOSPI_F64>;
defm sincospil : LibmLongDoubleLibCall;

// Exists on macOS
def __sincospif : RuntimeLibcallImpl<SINCOSPI_F32>;
def __sincospi : RuntimeLibcallImpl<SINCOSPI_F64>;

def bzero : RuntimeLibcallImpl<BZERO>;
def __bzero : RuntimeLibcallImpl<BZERO>;

// Exception handling
defset list<RuntimeLibcallImpl> DefaultExceptionHandlingLibcalls = {
  def _Unwind_Resume : RuntimeLibcallImpl<UNWIND_RESUME>;
  def __cxa_end_cleanup : RuntimeLibcallImpl<CXA_END_CLEANUP>;
}

defset list<RuntimeLibcallImpl> SjLjExceptionHandlingLibcalls = {
  def _Unwind_SjLj_Resume : RuntimeLibcallImpl<UNWIND_RESUME>;
  def _Unwind_SjLj_Register : RuntimeLibcallImpl<UNWIND_REGISTER>;
  def _Unwind_SjLj_Unregister : RuntimeLibcallImpl<UNWIND_UNREGISTER>;
}

// Only used on wasm?
def _Unwind_CallPersonality : RuntimeLibcallImpl<UNWIND_CALL_PERSONALITY>;

// Used on OpenBSD
def __stack_smash_handler : RuntimeLibcallImpl<STACK_SMASH_HANDLER>;

def __riscv_flush_icache : RuntimeLibcallImpl<RISCV_FLUSH_ICACHE>;

def _chkstk : RuntimeLibcallImpl<STACK_PROBE>;
````
- **L1825 EN**: Declares TableGen def `sincospif`.
  **L1825 CN**: 声明 TableGen def `sincospif`。
- **L1826 EN**: Declares TableGen def `sincospi`.
  **L1826 CN**: 声明 TableGen def `sincospi`。
- **L1827 EN**: Declares TableGen defm `sincospil`.
  **L1827 CN**: 声明 TableGen defm `sincospil`。
- **L1828 EN**: Blank line separating nearby declarations or logic blocks.
  **L1828 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1829 EN**: Comment explains nearby logic, invariants, or intent: `Exists on macOS`.
  **L1829 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Exists on macOS`。
- **L1830 EN**: Declares TableGen def `__sincospif`.
  **L1830 CN**: 声明 TableGen def `__sincospif`。
- **L1831 EN**: Declares TableGen def `__sincospi`.
  **L1831 CN**: 声明 TableGen def `__sincospi`。
- **L1832 EN**: Blank line separating nearby declarations or logic blocks.
  **L1832 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1833 EN**: Declares TableGen def `bzero`.
  **L1833 CN**: 声明 TableGen def `bzero`。
- **L1834 EN**: Declares TableGen def `__bzero`.
  **L1834 CN**: 声明 TableGen def `__bzero`。
- **L1835 EN**: Blank line separating nearby declarations or logic blocks.
  **L1835 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1836 EN**: Comment explains nearby logic, invariants, or intent: `Exception handling`.
  **L1836 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Exception handling`。
- **L1837 EN**: Continues the surrounding expression or declaration: `defset list<RuntimeLibcallImpl> DefaultExceptionHandlingLibcalls = {`.
  **L1837 CN**: 继续构造周围的表达式或声明：`defset list<RuntimeLibcallImpl> DefaultExceptionHandlingLibcalls = {`。
- **L1838 EN**: Declares TableGen def `_Unwind_Resume`.
  **L1838 CN**: 声明 TableGen def `_Unwind_Resume`。
- **L1839 EN**: Declares TableGen def `__cxa_end_cleanup`.
  **L1839 CN**: 声明 TableGen def `__cxa_end_cleanup`。
- **L1840 EN**: Closes the current lexical scope or compound statement.
  **L1840 CN**: 结束当前词法作用域或复合语句块。
- **L1841 EN**: Blank line separating nearby declarations or logic blocks.
  **L1841 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1842 EN**: Continues the surrounding expression or declaration: `defset list<RuntimeLibcallImpl> SjLjExceptionHandlingLibcalls = {`.
  **L1842 CN**: 继续构造周围的表达式或声明：`defset list<RuntimeLibcallImpl> SjLjExceptionHandlingLibcalls = {`。
- **L1843 EN**: Declares TableGen def `_Unwind_SjLj_Resume`.
  **L1843 CN**: 声明 TableGen def `_Unwind_SjLj_Resume`。
- **L1844 EN**: Declares TableGen def `_Unwind_SjLj_Register`.
  **L1844 CN**: 声明 TableGen def `_Unwind_SjLj_Register`。
- **L1845 EN**: Declares TableGen def `_Unwind_SjLj_Unregister`.
  **L1845 CN**: 声明 TableGen def `_Unwind_SjLj_Unregister`。
- **L1846 EN**: Closes the current lexical scope or compound statement.
  **L1846 CN**: 结束当前词法作用域或复合语句块。
- **L1847 EN**: Blank line separating nearby declarations or logic blocks.
  **L1847 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1848 EN**: Comment explains nearby logic, invariants, or intent: `Only used on wasm?`.
  **L1848 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Only used on wasm?`。
- **L1849 EN**: Declares TableGen def `_Unwind_CallPersonality`.
  **L1849 CN**: 声明 TableGen def `_Unwind_CallPersonality`。
- **L1850 EN**: Blank line separating nearby declarations or logic blocks.
  **L1850 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1851 EN**: Comment explains nearby logic, invariants, or intent: `Used on OpenBSD`.
  **L1851 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Used on OpenBSD`。
- **L1852 EN**: Declares TableGen def `__stack_smash_handler`.
  **L1852 CN**: 声明 TableGen def `__stack_smash_handler`。
- **L1853 EN**: Blank line separating nearby declarations or logic blocks.
  **L1853 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1854 EN**: Declares TableGen def `__riscv_flush_icache`.
  **L1854 CN**: 声明 TableGen def `__riscv_flush_icache`。
- **L1855 EN**: Blank line separating nearby declarations or logic blocks.
  **L1855 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1856 EN**: Declares TableGen def `_chkstk`.
  **L1856 CN**: 声明 TableGen def `_chkstk`。

### Lines 1857-1888

````tablegen
def _alloca : RuntimeLibcallImpl<STACK_PROBE>;

def __chkstk : RuntimeLibcallImpl<STACK_PROBE>;
def ___chkstk_ms : RuntimeLibcallImpl<STACK_PROBE>;
def __chkstk_arm64ec : RuntimeLibcallImpl<STACK_PROBE, "#__chkstk_arm64ec">;

def __security_check_cookie : RuntimeLibcallImpl<SECURITY_CHECK_COOKIE>;
def __security_check_cookie_arm64ec : RuntimeLibcallImpl<SECURITY_CHECK_COOKIE,
  "#__security_check_cookie_arm64ec">;

// Safe stack.
def __safestack_pointer_address : RuntimeLibcallImpl<SAFESTACK_POINTER_ADDRESS>;
def __safestack_unsafe_stack_ptr : RuntimeLibcallImpl<SAFESTACK_UNSAFE_STACK_PTR>;

//===----------------------------------------------------------------------===//
// F128 libm Runtime Libcalls
//===----------------------------------------------------------------------===//

defset list<RuntimeLibcallImpl> LibmF128Libcalls = {
  def logf128 : RuntimeLibcallImpl<LOG_F128>;
  def log2f128 : RuntimeLibcallImpl<LOG2_F128>;
  def log10f128 : RuntimeLibcallImpl<LOG10_F128>;
  def expf128 : RuntimeLibcallImpl<EXP_F128>;
  def exp2f128 : RuntimeLibcallImpl<EXP2_F128>;
  def exp10f128 : RuntimeLibcallImpl<EXP10_F128>;
  def sinf128 : RuntimeLibcallImpl<SIN_F128>;
  def cosf128 : RuntimeLibcallImpl<COS_F128>;
  def tanf128 : RuntimeLibcallImpl<TAN_F128>;
  def tanhf128 : RuntimeLibcallImpl<TANH_F128>;
  def sincosf128 : RuntimeLibcallImpl<SINCOS_F128>;
  def powf128 : RuntimeLibcallImpl<POW_F128>;
  def fminf128 : RuntimeLibcallImpl<FMIN_F128>;
````
- **L1857 EN**: Declares TableGen def `_alloca`.
  **L1857 CN**: 声明 TableGen def `_alloca`。
- **L1858 EN**: Blank line separating nearby declarations or logic blocks.
  **L1858 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1859 EN**: Declares TableGen def `__chkstk`.
  **L1859 CN**: 声明 TableGen def `__chkstk`。
- **L1860 EN**: Declares TableGen def `___chkstk_ms`.
  **L1860 CN**: 声明 TableGen def `___chkstk_ms`。
- **L1861 EN**: Declares TableGen def `__chkstk_arm64ec`.
  **L1861 CN**: 声明 TableGen def `__chkstk_arm64ec`。
- **L1862 EN**: Blank line separating nearby declarations or logic blocks.
  **L1862 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1863 EN**: Declares TableGen def `__security_check_cookie`.
  **L1863 CN**: 声明 TableGen def `__security_check_cookie`。
- **L1864 EN**: Declares TableGen def `__security_check_cookie_arm64ec`.
  **L1864 CN**: 声明 TableGen def `__security_check_cookie_arm64ec`。
- **L1865 EN**: Executes a standalone statement or declaration: `"#__security_check_cookie_arm64ec">;`.
  **L1865 CN**: 执行一条独立语句或声明：`"#__security_check_cookie_arm64ec">;`。
- **L1866 EN**: Blank line separating nearby declarations or logic blocks.
  **L1866 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1867 EN**: Comment explains nearby logic, invariants, or intent: `Safe stack.`.
  **L1867 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Safe stack.`。
- **L1868 EN**: Declares TableGen def `__safestack_pointer_address`.
  **L1868 CN**: 声明 TableGen def `__safestack_pointer_address`。
- **L1869 EN**: Declares TableGen def `__safestack_unsafe_stack_ptr`.
  **L1869 CN**: 声明 TableGen def `__safestack_unsafe_stack_ptr`。
- **L1870 EN**: Blank line separating nearby declarations or logic blocks.
  **L1870 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1871 EN**: Banner comment marking a file or section boundary.
  **L1871 CN**: 横幅注释，用于标记文件或章节边界。
- **L1872 EN**: Comment explains nearby logic, invariants, or intent: `F128 libm Runtime Libcalls`.
  **L1872 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`F128 libm Runtime Libcalls`。
- **L1873 EN**: Banner comment marking a file or section boundary.
  **L1873 CN**: 横幅注释，用于标记文件或章节边界。
- **L1874 EN**: Blank line separating nearby declarations or logic blocks.
  **L1874 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1875 EN**: Continues the surrounding expression or declaration: `defset list<RuntimeLibcallImpl> LibmF128Libcalls = {`.
  **L1875 CN**: 继续构造周围的表达式或声明：`defset list<RuntimeLibcallImpl> LibmF128Libcalls = {`。
- **L1876 EN**: Declares TableGen def `logf128`.
  **L1876 CN**: 声明 TableGen def `logf128`。
- **L1877 EN**: Declares TableGen def `log2f128`.
  **L1877 CN**: 声明 TableGen def `log2f128`。
- **L1878 EN**: Declares TableGen def `log10f128`.
  **L1878 CN**: 声明 TableGen def `log10f128`。
- **L1879 EN**: Declares TableGen def `expf128`.
  **L1879 CN**: 声明 TableGen def `expf128`。
- **L1880 EN**: Declares TableGen def `exp2f128`.
  **L1880 CN**: 声明 TableGen def `exp2f128`。
- **L1881 EN**: Declares TableGen def `exp10f128`.
  **L1881 CN**: 声明 TableGen def `exp10f128`。
- **L1882 EN**: Declares TableGen def `sinf128`.
  **L1882 CN**: 声明 TableGen def `sinf128`。
- **L1883 EN**: Declares TableGen def `cosf128`.
  **L1883 CN**: 声明 TableGen def `cosf128`。
- **L1884 EN**: Declares TableGen def `tanf128`.
  **L1884 CN**: 声明 TableGen def `tanf128`。
- **L1885 EN**: Declares TableGen def `tanhf128`.
  **L1885 CN**: 声明 TableGen def `tanhf128`。
- **L1886 EN**: Declares TableGen def `sincosf128`.
  **L1886 CN**: 声明 TableGen def `sincosf128`。
- **L1887 EN**: Declares TableGen def `powf128`.
  **L1887 CN**: 声明 TableGen def `powf128`。
- **L1888 EN**: Declares TableGen def `fminf128`.
  **L1888 CN**: 声明 TableGen def `fminf128`。

### Lines 1889-1920

````tablegen
  def fmaxf128 : RuntimeLibcallImpl<FMAX_F128>;
  def fmodf128 : RuntimeLibcallImpl<REM_F128>;
  def sqrtf128 : RuntimeLibcallImpl<SQRT_F128>;
  def ceilf128 : RuntimeLibcallImpl<CEIL_F128>;
  def floorf128 : RuntimeLibcallImpl<FLOOR_F128>;
  def truncf128 : RuntimeLibcallImpl<TRUNC_F128>;
  def roundf128 : RuntimeLibcallImpl<ROUND_F128>;
  def lroundf128 : RuntimeLibcallImpl<LROUND_F128>;
  def llroundf128 : RuntimeLibcallImpl<LLROUND_F128>;
  def rintf128 : RuntimeLibcallImpl<RINT_F128>;
  def lrintf128 : RuntimeLibcallImpl<LRINT_F128>;
  def llrintf128 : RuntimeLibcallImpl<LLRINT_F128>;
  def nearbyintf128 : RuntimeLibcallImpl<NEARBYINT_F128>;
  def fmaf128 : RuntimeLibcallImpl<FMA_F128>;
  def frexpf128 : RuntimeLibcallImpl<FREXP_F128>;
  def cbrtf128 : RuntimeLibcallImpl<CBRT_F128>;
  def fminimumf128 : RuntimeLibcallImpl<FMINIMUM_F128>;
  def fmaximumf128 : RuntimeLibcallImpl<FMAXIMUM_F128>;
  def fminimum_numf128 : RuntimeLibcallImpl<FMINIMUM_NUM_F128>;
  def fmaximum_numf128 : RuntimeLibcallImpl<FMAXIMUM_NUM_F128>;
  def asinf128 : RuntimeLibcallImpl<ASIN_F128>;
  def acosf128 : RuntimeLibcallImpl<ACOS_F128>;
  def atanf128 : RuntimeLibcallImpl<ATAN_F128>;
  def atan2f128 : RuntimeLibcallImpl<ATAN2_F128>;
  def ldexpf128 : RuntimeLibcallImpl<LDEXP_F128>;
  def roundevenf128 : RuntimeLibcallImpl<ROUNDEVEN_F128>;
  def modff128 : RuntimeLibcallImpl<MODF_F128>;
  def sinhf128 : RuntimeLibcallImpl<SINH_F128>;
  def coshf128 : RuntimeLibcallImpl<COSH_F128>;
  def copysignf128 : RuntimeLibcallImpl<COPYSIGN_F128>;
}

````
- **L1889 EN**: Declares TableGen def `fmaxf128`.
  **L1889 CN**: 声明 TableGen def `fmaxf128`。
- **L1890 EN**: Declares TableGen def `fmodf128`.
  **L1890 CN**: 声明 TableGen def `fmodf128`。
- **L1891 EN**: Declares TableGen def `sqrtf128`.
  **L1891 CN**: 声明 TableGen def `sqrtf128`。
- **L1892 EN**: Declares TableGen def `ceilf128`.
  **L1892 CN**: 声明 TableGen def `ceilf128`。
- **L1893 EN**: Declares TableGen def `floorf128`.
  **L1893 CN**: 声明 TableGen def `floorf128`。
- **L1894 EN**: Declares TableGen def `truncf128`.
  **L1894 CN**: 声明 TableGen def `truncf128`。
- **L1895 EN**: Declares TableGen def `roundf128`.
  **L1895 CN**: 声明 TableGen def `roundf128`。
- **L1896 EN**: Declares TableGen def `lroundf128`.
  **L1896 CN**: 声明 TableGen def `lroundf128`。
- **L1897 EN**: Declares TableGen def `llroundf128`.
  **L1897 CN**: 声明 TableGen def `llroundf128`。
- **L1898 EN**: Declares TableGen def `rintf128`.
  **L1898 CN**: 声明 TableGen def `rintf128`。
- **L1899 EN**: Declares TableGen def `lrintf128`.
  **L1899 CN**: 声明 TableGen def `lrintf128`。
- **L1900 EN**: Declares TableGen def `llrintf128`.
  **L1900 CN**: 声明 TableGen def `llrintf128`。
- **L1901 EN**: Declares TableGen def `nearbyintf128`.
  **L1901 CN**: 声明 TableGen def `nearbyintf128`。
- **L1902 EN**: Declares TableGen def `fmaf128`.
  **L1902 CN**: 声明 TableGen def `fmaf128`。
- **L1903 EN**: Declares TableGen def `frexpf128`.
  **L1903 CN**: 声明 TableGen def `frexpf128`。
- **L1904 EN**: Declares TableGen def `cbrtf128`.
  **L1904 CN**: 声明 TableGen def `cbrtf128`。
- **L1905 EN**: Declares TableGen def `fminimumf128`.
  **L1905 CN**: 声明 TableGen def `fminimumf128`。
- **L1906 EN**: Declares TableGen def `fmaximumf128`.
  **L1906 CN**: 声明 TableGen def `fmaximumf128`。
- **L1907 EN**: Declares TableGen def `fminimum_numf128`.
  **L1907 CN**: 声明 TableGen def `fminimum_numf128`。
- **L1908 EN**: Declares TableGen def `fmaximum_numf128`.
  **L1908 CN**: 声明 TableGen def `fmaximum_numf128`。
- **L1909 EN**: Declares TableGen def `asinf128`.
  **L1909 CN**: 声明 TableGen def `asinf128`。
- **L1910 EN**: Declares TableGen def `acosf128`.
  **L1910 CN**: 声明 TableGen def `acosf128`。
- **L1911 EN**: Declares TableGen def `atanf128`.
  **L1911 CN**: 声明 TableGen def `atanf128`。
- **L1912 EN**: Declares TableGen def `atan2f128`.
  **L1912 CN**: 声明 TableGen def `atan2f128`。
- **L1913 EN**: Declares TableGen def `ldexpf128`.
  **L1913 CN**: 声明 TableGen def `ldexpf128`。
- **L1914 EN**: Declares TableGen def `roundevenf128`.
  **L1914 CN**: 声明 TableGen def `roundevenf128`。
- **L1915 EN**: Declares TableGen def `modff128`.
  **L1915 CN**: 声明 TableGen def `modff128`。
- **L1916 EN**: Declares TableGen def `sinhf128`.
  **L1916 CN**: 声明 TableGen def `sinhf128`。
- **L1917 EN**: Declares TableGen def `coshf128`.
  **L1917 CN**: 声明 TableGen def `coshf128`。
- **L1918 EN**: Declares TableGen def `copysignf128`.
  **L1918 CN**: 声明 TableGen def `copysignf128`。
- **L1919 EN**: Closes the current lexical scope or compound statement.
  **L1919 CN**: 结束当前词法作用域或复合语句块。
- **L1920 EN**: Blank line separating nearby declarations or logic blocks.
  **L1920 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 1921-1952

````tablegen
defset list<RuntimeLibcallImpl> LibmF128FiniteLibcalls = {
  def __logf128_finite : RuntimeLibcallImpl<LOG_FINITE_F128>;
  def __log2f128_finite : RuntimeLibcallImpl<LOG2_FINITE_F128>;
  def __log10f128_finite : RuntimeLibcallImpl<LOG10_FINITE_F128>;
  def __expf128_finite : RuntimeLibcallImpl<EXP_FINITE_F128>;
  def __exp2f128_finite : RuntimeLibcallImpl<EXP2_FINITE_F128>;
  def __exp10f128_finite : RuntimeLibcallImpl<EXP10_FINITE_F128>;
  def __powf128_finite : RuntimeLibcallImpl<POW_FINITE_F128>;
}

//--------------------------------------------------------------------
// Global variable references
//--------------------------------------------------------------------

def __stack_chk_guard : RuntimeLibcallImpl<STACK_CHECK_GUARD>;

// Name used on OpenBSD
def __guard_local : RuntimeLibcallImpl<STACK_CHECK_GUARD>;

// Name used with Windows MSVC
def __security_cookie : RuntimeLibcallImpl<STACK_CHECK_GUARD>;

// Name used on AIX
def __ssp_canary_word : RuntimeLibcallImpl<STACK_CHECK_GUARD>;

//===----------------------------------------------------------------------===//
// Common Libcall Sets
//===----------------------------------------------------------------------===//

defvar ExceptionModelCalls = (add
  LibcallImpls<(add __cxa_end_cleanup),
               ExceptionModelIsNotNone>,
````
- **L1921 EN**: Continues the surrounding expression or declaration: `defset list<RuntimeLibcallImpl> LibmF128FiniteLibcalls = {`.
  **L1921 CN**: 继续构造周围的表达式或声明：`defset list<RuntimeLibcallImpl> LibmF128FiniteLibcalls = {`。
- **L1922 EN**: Declares TableGen def `__logf128_finite`.
  **L1922 CN**: 声明 TableGen def `__logf128_finite`。
- **L1923 EN**: Declares TableGen def `__log2f128_finite`.
  **L1923 CN**: 声明 TableGen def `__log2f128_finite`。
- **L1924 EN**: Declares TableGen def `__log10f128_finite`.
  **L1924 CN**: 声明 TableGen def `__log10f128_finite`。
- **L1925 EN**: Declares TableGen def `__expf128_finite`.
  **L1925 CN**: 声明 TableGen def `__expf128_finite`。
- **L1926 EN**: Declares TableGen def `__exp2f128_finite`.
  **L1926 CN**: 声明 TableGen def `__exp2f128_finite`。
- **L1927 EN**: Declares TableGen def `__exp10f128_finite`.
  **L1927 CN**: 声明 TableGen def `__exp10f128_finite`。
- **L1928 EN**: Declares TableGen def `__powf128_finite`.
  **L1928 CN**: 声明 TableGen def `__powf128_finite`。
- **L1929 EN**: Closes the current lexical scope or compound statement.
  **L1929 CN**: 结束当前词法作用域或复合语句块。
- **L1930 EN**: Blank line separating nearby declarations or logic blocks.
  **L1930 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1931 EN**: Comment explains nearby logic, invariants, or intent: `--------------------------------------------------------------------`.
  **L1931 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`--------------------------------------------------------------------`。
- **L1932 EN**: Comment explains nearby logic, invariants, or intent: `Global variable references`.
  **L1932 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Global variable references`。
- **L1933 EN**: Comment explains nearby logic, invariants, or intent: `--------------------------------------------------------------------`.
  **L1933 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`--------------------------------------------------------------------`。
- **L1934 EN**: Blank line separating nearby declarations or logic blocks.
  **L1934 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1935 EN**: Declares TableGen def `__stack_chk_guard`.
  **L1935 CN**: 声明 TableGen def `__stack_chk_guard`。
- **L1936 EN**: Blank line separating nearby declarations or logic blocks.
  **L1936 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1937 EN**: Comment explains nearby logic, invariants, or intent: `Name used on OpenBSD`.
  **L1937 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Name used on OpenBSD`。
- **L1938 EN**: Declares TableGen def `__guard_local`.
  **L1938 CN**: 声明 TableGen def `__guard_local`。
- **L1939 EN**: Blank line separating nearby declarations or logic blocks.
  **L1939 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1940 EN**: Comment explains nearby logic, invariants, or intent: `Name used with Windows MSVC`.
  **L1940 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Name used with Windows MSVC`。
- **L1941 EN**: Declares TableGen def `__security_cookie`.
  **L1941 CN**: 声明 TableGen def `__security_cookie`。
- **L1942 EN**: Blank line separating nearby declarations or logic blocks.
  **L1942 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1943 EN**: Comment explains nearby logic, invariants, or intent: `Name used on AIX`.
  **L1943 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Name used on AIX`。
- **L1944 EN**: Declares TableGen def `__ssp_canary_word`.
  **L1944 CN**: 声明 TableGen def `__ssp_canary_word`。
- **L1945 EN**: Blank line separating nearby declarations or logic blocks.
  **L1945 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1946 EN**: Banner comment marking a file or section boundary.
  **L1946 CN**: 横幅注释，用于标记文件或章节边界。
- **L1947 EN**: Comment explains nearby logic, invariants, or intent: `Common Libcall Sets`.
  **L1947 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Common Libcall Sets`。
- **L1948 EN**: Banner comment marking a file or section boundary.
  **L1948 CN**: 横幅注释，用于标记文件或章节边界。
- **L1949 EN**: Blank line separating nearby declarations or logic blocks.
  **L1949 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1950 EN**: Continues the surrounding expression or declaration: `defvar ExceptionModelCalls = (add`.
  **L1950 CN**: 继续构造周围的表达式或声明：`defvar ExceptionModelCalls = (add`。
- **L1951 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LibcallImpls<(add __cxa_end_cleanup),`.
  **L1951 CN**: 继续一个多行参数列表、初始化器或聚合项：`LibcallImpls<(add __cxa_end_cleanup),`。
- **L1952 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ExceptionModelIsNotNone>,`.
  **L1952 CN**: 继续一个多行参数列表、初始化器或聚合项：`ExceptionModelIsNotNone>,`。

### Lines 1953-1984

````tablegen
  LibcallImpls<(add _Unwind_Resume),
               ExceptionModelHasUnwindResume>,
  LibcallImpls<(add SjLjExceptionHandlingLibcalls),
               ExceptionModelIsSjLj>
);

// FIXME: Should move to explicit opt-in to different sets of libcalls
// instead of trying to remove from a default set. We have
// unreasonable defaults like reporting f80 calls on most targets when
// they are relevant to only one.

defvar AllDefaultRuntimeLibcallImpls
  = !filter(entry, !instances<RuntimeLibcallImpl>(), entry.IsDefault);

defvar DefaultRuntimeLibcallImpls_f80 =
    !filter(entry, AllDefaultRuntimeLibcallImpls,
            !match(!cast<string>(entry.Provides), "F80"));

defvar DefaultRuntimeLibcallImpls_ppcf128 =
    !filter(entry, AllDefaultRuntimeLibcallImpls,
            !match(!cast<string>(entry.Provides), "PPCF128"));

defvar DefaultRuntimeLibcallImpls_f128 =
    !filter(entry, AllDefaultRuntimeLibcallImpls,
            !match(!cast<string>(entry.Provides), "_F128"));

// FIXME: Ideally we would just use dags everywhere, but for the
// arm64ec case we need iterable lists so we can add the # prefix
defvar DefaultRuntimeLibcallImplsBaseList =
!listremove(
  !listremove(
    !listremove(AllDefaultRuntimeLibcallImpls, Int128RTLibcalls),
````
- **L1953 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LibcallImpls<(add _Unwind_Resume),`.
  **L1953 CN**: 继续一个多行参数列表、初始化器或聚合项：`LibcallImpls<(add _Unwind_Resume),`。
- **L1954 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ExceptionModelHasUnwindResume>,`.
  **L1954 CN**: 继续一个多行参数列表、初始化器或聚合项：`ExceptionModelHasUnwindResume>,`。
- **L1955 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LibcallImpls<(add SjLjExceptionHandlingLibcalls),`.
  **L1955 CN**: 继续一个多行参数列表、初始化器或聚合项：`LibcallImpls<(add SjLjExceptionHandlingLibcalls),`。
- **L1956 EN**: Continues the surrounding expression or declaration: `ExceptionModelIsSjLj>`.
  **L1956 CN**: 继续构造周围的表达式或声明：`ExceptionModelIsSjLj>`。
- **L1957 EN**: Executes a standalone statement or declaration: `);`.
  **L1957 CN**: 执行一条独立语句或声明：`);`。
- **L1958 EN**: Blank line separating nearby declarations or logic blocks.
  **L1958 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1959 EN**: Comment records a pending task or caution: `FIXME: Should move to explicit opt-in to different sets of libcalls`.
  **L1959 CN**: 注释记录了待办事项或注意点：`FIXME: Should move to explicit opt-in to different sets of libcalls`。
- **L1960 EN**: Comment explains nearby logic, invariants, or intent: `instead of trying to remove from a default set. We have`.
  **L1960 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`instead of trying to remove from a default set. We have`。
- **L1961 EN**: Comment explains nearby logic, invariants, or intent: `unreasonable defaults like reporting f80 calls on most targets when`.
  **L1961 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`unreasonable defaults like reporting f80 calls on most targets when`。
- **L1962 EN**: Comment explains nearby logic, invariants, or intent: `they are relevant to only one.`.
  **L1962 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`they are relevant to only one.`。
- **L1963 EN**: Blank line separating nearby declarations or logic blocks.
  **L1963 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1964 EN**: Continues the surrounding expression or declaration: `defvar AllDefaultRuntimeLibcallImpls`.
  **L1964 CN**: 继续构造周围的表达式或声明：`defvar AllDefaultRuntimeLibcallImpls`。
- **L1965 EN**: Executes a call or declaration centered on `!filter`.
  **L1965 CN**: 执行以 `!filter` 为核心的调用或声明。
- **L1966 EN**: Blank line separating nearby declarations or logic blocks.
  **L1966 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1967 EN**: Continues the surrounding expression or declaration: `defvar DefaultRuntimeLibcallImpls_f80 =`.
  **L1967 CN**: 继续构造周围的表达式或声明：`defvar DefaultRuntimeLibcallImpls_f80 =`。
- **L1968 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!filter(entry, AllDefaultRuntimeLibcallImpls,`.
  **L1968 CN**: 继续一个多行参数列表、初始化器或聚合项：`!filter(entry, AllDefaultRuntimeLibcallImpls,`。
- **L1969 EN**: Executes a call or declaration centered on `!match`.
  **L1969 CN**: 执行以 `!match` 为核心的调用或声明。
- **L1970 EN**: Blank line separating nearby declarations or logic blocks.
  **L1970 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1971 EN**: Continues the surrounding expression or declaration: `defvar DefaultRuntimeLibcallImpls_ppcf128 =`.
  **L1971 CN**: 继续构造周围的表达式或声明：`defvar DefaultRuntimeLibcallImpls_ppcf128 =`。
- **L1972 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!filter(entry, AllDefaultRuntimeLibcallImpls,`.
  **L1972 CN**: 继续一个多行参数列表、初始化器或聚合项：`!filter(entry, AllDefaultRuntimeLibcallImpls,`。
- **L1973 EN**: Executes a call or declaration centered on `!match`.
  **L1973 CN**: 执行以 `!match` 为核心的调用或声明。
- **L1974 EN**: Blank line separating nearby declarations or logic blocks.
  **L1974 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1975 EN**: Continues the surrounding expression or declaration: `defvar DefaultRuntimeLibcallImpls_f128 =`.
  **L1975 CN**: 继续构造周围的表达式或声明：`defvar DefaultRuntimeLibcallImpls_f128 =`。
- **L1976 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!filter(entry, AllDefaultRuntimeLibcallImpls,`.
  **L1976 CN**: 继续一个多行参数列表、初始化器或聚合项：`!filter(entry, AllDefaultRuntimeLibcallImpls,`。
- **L1977 EN**: Executes a call or declaration centered on `!match`.
  **L1977 CN**: 执行以 `!match` 为核心的调用或声明。
- **L1978 EN**: Blank line separating nearby declarations or logic blocks.
  **L1978 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1979 EN**: Comment records a pending task or caution: `FIXME: Ideally we would just use dags everywhere, but for the`.
  **L1979 CN**: 注释记录了待办事项或注意点：`FIXME: Ideally we would just use dags everywhere, but for the`。
- **L1980 EN**: Comment explains nearby logic, invariants, or intent: `arm64ec case we need iterable lists so we can add the # prefix`.
  **L1980 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`arm64ec case we need iterable lists so we can add the # prefix`。
- **L1981 EN**: Continues the surrounding expression or declaration: `defvar DefaultRuntimeLibcallImplsBaseList =`.
  **L1981 CN**: 继续构造周围的表达式或声明：`defvar DefaultRuntimeLibcallImplsBaseList =`。
- **L1982 EN**: Continues logic associated with callable symbol `listremove`.
  **L1982 CN**: 继续与可调用符号 `listremove` 相关的逻辑。
- **L1983 EN**: Continues logic associated with callable symbol `listremove`.
  **L1983 CN**: 继续与可调用符号 `listremove` 相关的逻辑。
- **L1984 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!listremove(AllDefaultRuntimeLibcallImpls, Int128RTLibcalls),`.
  **L1984 CN**: 继续一个多行参数列表、初始化器或聚合项：`!listremove(AllDefaultRuntimeLibcallImpls, Int128RTLibcalls),`。

### Lines 1985-2016

````tablegen
                DefaultRuntimeLibcallImpls_f80),
                DefaultRuntimeLibcallImpls_ppcf128);

defvar DefaultRuntimeLibcallImpls =
  (add DefaultRuntimeLibcallImplsBaseList,
       ExceptionModelCalls);

/// Default set of libcall impls for 32-bit architectures.
defvar DefaultLibcallImpls32 = (add DefaultRuntimeLibcallImpls);

/// Default set of libcall impls for 64-bit architectures.
defvar DefaultLibcallImpls64 = (add DefaultRuntimeLibcallImpls,
                                    Int128RTLibcalls);

// TODO: Guessing sincospi added at same time as sincos_stret
defvar DarwinSinCosStret = LibcallImpls<(add __sincosf_stret, __sincos_stret,
                                             __sincospif, __sincospi),
                                        darwinHasSinCosStret>;
defvar DarwinExp10 = LibcallImpls<(add __exp10f, __exp10), darwinHasExp10>;

defvar DarwinMemsetPattern = LibcallImpls<(add memset_pattern4,
                                               memset_pattern8,
                                               memset_pattern16),
                                               darwinHasMemsetPattern>;

defvar MacOSUnlockedIO = LibcallImpls<(add
  getc_unlocked, getchar_unlocked, putc_unlocked, putchar_unlocked),
  isMacOSX>;

defvar SecurityCheckCookieIfWinMSVC =
    LibcallImpls<(add __security_check_cookie, __security_cookie),
                 isWindowsMSVCOrItaniumEnvironment>;
````
- **L1985 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultRuntimeLibcallImpls_f80),`.
  **L1985 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultRuntimeLibcallImpls_f80),`。
- **L1986 EN**: Executes a standalone statement or declaration: `DefaultRuntimeLibcallImpls_ppcf128);`.
  **L1986 CN**: 执行一条独立语句或声明：`DefaultRuntimeLibcallImpls_ppcf128);`。
- **L1987 EN**: Blank line separating nearby declarations or logic blocks.
  **L1987 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1988 EN**: Continues the surrounding expression or declaration: `defvar DefaultRuntimeLibcallImpls =`.
  **L1988 CN**: 继续构造周围的表达式或声明：`defvar DefaultRuntimeLibcallImpls =`。
- **L1989 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(add DefaultRuntimeLibcallImplsBaseList,`.
  **L1989 CN**: 继续一个多行参数列表、初始化器或聚合项：`(add DefaultRuntimeLibcallImplsBaseList,`。
- **L1990 EN**: Executes a standalone statement or declaration: `ExceptionModelCalls);`.
  **L1990 CN**: 执行一条独立语句或声明：`ExceptionModelCalls);`。
- **L1991 EN**: Blank line separating nearby declarations or logic blocks.
  **L1991 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1992 EN**: Comment explains nearby logic, invariants, or intent: `Default set of libcall impls for 32-bit architectures.`.
  **L1992 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Default set of libcall impls for 32-bit architectures.`。
- **L1993 EN**: Initializes variable `DefaultLibcallImpls32` from the right-hand expression.
  **L1993 CN**: 使用右侧表达式初始化变量 `DefaultLibcallImpls32`。
- **L1994 EN**: Blank line separating nearby declarations or logic blocks.
  **L1994 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1995 EN**: Comment explains nearby logic, invariants, or intent: `Default set of libcall impls for 64-bit architectures.`.
  **L1995 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Default set of libcall impls for 64-bit architectures.`。
- **L1996 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `defvar DefaultLibcallImpls64 = (add DefaultRuntimeLibcallImpls,`.
  **L1996 CN**: 继续一个多行参数列表、初始化器或聚合项：`defvar DefaultLibcallImpls64 = (add DefaultRuntimeLibcallImpls,`。
- **L1997 EN**: Executes a standalone statement or declaration: `Int128RTLibcalls);`.
  **L1997 CN**: 执行一条独立语句或声明：`Int128RTLibcalls);`。
- **L1998 EN**: Blank line separating nearby declarations or logic blocks.
  **L1998 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1999 EN**: Comment records a pending task or caution: `TODO: Guessing sincospi added at same time as sincos_stret`.
  **L1999 CN**: 注释记录了待办事项或注意点：`TODO: Guessing sincospi added at same time as sincos_stret`。
- **L2000 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `defvar DarwinSinCosStret = LibcallImpls<(add __sincosf_stret, __sincos_stret,`.
  **L2000 CN**: 继续一个多行参数列表、初始化器或聚合项：`defvar DarwinSinCosStret = LibcallImpls<(add __sincosf_stret, __sincos_stret,`。
- **L2001 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__sincospif, __sincospi),`.
  **L2001 CN**: 继续一个多行参数列表、初始化器或聚合项：`__sincospif, __sincospi),`。
- **L2002 EN**: Executes a standalone statement or declaration: `darwinHasSinCosStret>;`.
  **L2002 CN**: 执行一条独立语句或声明：`darwinHasSinCosStret>;`。
- **L2003 EN**: Initializes variable `DarwinExp10` from the right-hand expression.
  **L2003 CN**: 使用右侧表达式初始化变量 `DarwinExp10`。
- **L2004 EN**: Blank line separating nearby declarations or logic blocks.
  **L2004 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2005 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `defvar DarwinMemsetPattern = LibcallImpls<(add memset_pattern4,`.
  **L2005 CN**: 继续一个多行参数列表、初始化器或聚合项：`defvar DarwinMemsetPattern = LibcallImpls<(add memset_pattern4,`。
- **L2006 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `memset_pattern8,`.
  **L2006 CN**: 继续一个多行参数列表、初始化器或聚合项：`memset_pattern8,`。
- **L2007 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `memset_pattern16),`.
  **L2007 CN**: 继续一个多行参数列表、初始化器或聚合项：`memset_pattern16),`。
- **L2008 EN**: Executes a standalone statement or declaration: `darwinHasMemsetPattern>;`.
  **L2008 CN**: 执行一条独立语句或声明：`darwinHasMemsetPattern>;`。
- **L2009 EN**: Blank line separating nearby declarations or logic blocks.
  **L2009 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2010 EN**: Continues logic associated with callable symbol `LibcallImpls<`.
  **L2010 CN**: 继续与可调用符号 `LibcallImpls<` 相关的逻辑。
- **L2011 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `getc_unlocked, getchar_unlocked, putc_unlocked, putchar_unlocked),`.
  **L2011 CN**: 继续一个多行参数列表、初始化器或聚合项：`getc_unlocked, getchar_unlocked, putc_unlocked, putchar_unlocked),`。
- **L2012 EN**: Executes a standalone statement or declaration: `isMacOSX>;`.
  **L2012 CN**: 执行一条独立语句或声明：`isMacOSX>;`。
- **L2013 EN**: Blank line separating nearby declarations or logic blocks.
  **L2013 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2014 EN**: Continues the surrounding expression or declaration: `defvar SecurityCheckCookieIfWinMSVC =`.
  **L2014 CN**: 继续构造周围的表达式或声明：`defvar SecurityCheckCookieIfWinMSVC =`。
- **L2015 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LibcallImpls<(add __security_check_cookie, __security_cookie),`.
  **L2015 CN**: 继续一个多行参数列表、初始化器或聚合项：`LibcallImpls<(add __security_check_cookie, __security_cookie),`。
- **L2016 EN**: Executes a standalone statement or declaration: `isWindowsMSVCOrItaniumEnvironment>;`.
  **L2016 CN**: 执行一条独立语句或声明：`isWindowsMSVCOrItaniumEnvironment>;`。

### Lines 2017-2048

````tablegen

defvar LibmHasSinCosF32 = LibcallImpls<(add sincosf), hasSinCos>;
defvar LibmHasSinCosF64 =  LibcallImpls<(add sincos), hasSinCos>;
defvar LibmHasSinCosF80 = LibcallImpls<(add sincosl_f80), hasSinCos>;
defvar LibmHasSinCosF128 = LibcallImpls<(add sincosl_f128), hasSinCos>;
defvar LibmHasSinCosPPCF128 = LibcallImpls<(add sincosl_ppcf128), hasSinCos>;

defvar LibmHasExp10F32 = LibcallImpls<(add exp10f), hasExp10>;
defvar LibmHasExp10F64 = LibcallImpls<(add exp10), hasExp10>;
defvar LibmHasExp10F80 = LibcallImpls<(add exp10l_f80), hasExp10>;
defvar LibmHasExp10F128 = LibcallImpls<(add exp10l_f128), hasExp10>;
defvar LibmHasExp10PPCF128 = LibcallImpls<(add exp10l_ppcf128), hasExp10>;

defvar DefaultLibmExp10 = [
  LibmHasExp10F32, LibmHasExp10F64, LibmHasExp10F128
];


defvar WindowsMathRemovals = [
  ldexpf, ldexpl_f80, ldexpl_f128, ldexpl_ppcf128,
  frexpf, frexpl_f80, frexpl_f128, frexpl_ppcf128
];

defvar MostPowI = !listremove(PowiLibcallImpls, [__powitf2_f128, __powitf2_ppc128]);
defvar WindowsExclusions = !listconcat(WindowsMathRemovals, MostPowI);

// Targets which support windows should start with these as a base and
// add in calls for other OSes
defvar WinDefaultLibcallImplsBaseList =
  !listremove(DefaultRuntimeLibcallImplsBaseList,
              WindowsExclusions);

````
- **L2017 EN**: Blank line separating nearby declarations or logic blocks.
  **L2017 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2018 EN**: Initializes variable `LibmHasSinCosF32` from the right-hand expression.
  **L2018 CN**: 使用右侧表达式初始化变量 `LibmHasSinCosF32`。
- **L2019 EN**: Initializes variable `LibmHasSinCosF64` from the right-hand expression.
  **L2019 CN**: 使用右侧表达式初始化变量 `LibmHasSinCosF64`。
- **L2020 EN**: Initializes variable `LibmHasSinCosF80` from the right-hand expression.
  **L2020 CN**: 使用右侧表达式初始化变量 `LibmHasSinCosF80`。
- **L2021 EN**: Initializes variable `LibmHasSinCosF128` from the right-hand expression.
  **L2021 CN**: 使用右侧表达式初始化变量 `LibmHasSinCosF128`。
- **L2022 EN**: Initializes variable `LibmHasSinCosPPCF128` from the right-hand expression.
  **L2022 CN**: 使用右侧表达式初始化变量 `LibmHasSinCosPPCF128`。
- **L2023 EN**: Blank line separating nearby declarations or logic blocks.
  **L2023 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2024 EN**: Initializes variable `LibmHasExp10F32` from the right-hand expression.
  **L2024 CN**: 使用右侧表达式初始化变量 `LibmHasExp10F32`。
- **L2025 EN**: Initializes variable `LibmHasExp10F64` from the right-hand expression.
  **L2025 CN**: 使用右侧表达式初始化变量 `LibmHasExp10F64`。
- **L2026 EN**: Initializes variable `LibmHasExp10F80` from the right-hand expression.
  **L2026 CN**: 使用右侧表达式初始化变量 `LibmHasExp10F80`。
- **L2027 EN**: Initializes variable `LibmHasExp10F128` from the right-hand expression.
  **L2027 CN**: 使用右侧表达式初始化变量 `LibmHasExp10F128`。
- **L2028 EN**: Initializes variable `LibmHasExp10PPCF128` from the right-hand expression.
  **L2028 CN**: 使用右侧表达式初始化变量 `LibmHasExp10PPCF128`。
- **L2029 EN**: Blank line separating nearby declarations or logic blocks.
  **L2029 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2030 EN**: Continues the surrounding expression or declaration: `defvar DefaultLibmExp10 = [`.
  **L2030 CN**: 继续构造周围的表达式或声明：`defvar DefaultLibmExp10 = [`。
- **L2031 EN**: Continues the surrounding expression or declaration: `LibmHasExp10F32, LibmHasExp10F64, LibmHasExp10F128`.
  **L2031 CN**: 继续构造周围的表达式或声明：`LibmHasExp10F32, LibmHasExp10F64, LibmHasExp10F128`。
- **L2032 EN**: Executes a standalone statement or declaration: `];`.
  **L2032 CN**: 执行一条独立语句或声明：`];`。
- **L2033 EN**: Blank line separating nearby declarations or logic blocks.
  **L2033 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2034 EN**: Blank line separating nearby declarations or logic blocks.
  **L2034 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2035 EN**: Continues the surrounding expression or declaration: `defvar WindowsMathRemovals = [`.
  **L2035 CN**: 继续构造周围的表达式或声明：`defvar WindowsMathRemovals = [`。
- **L2036 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ldexpf, ldexpl_f80, ldexpl_f128, ldexpl_ppcf128,`.
  **L2036 CN**: 继续一个多行参数列表、初始化器或聚合项：`ldexpf, ldexpl_f80, ldexpl_f128, ldexpl_ppcf128,`。
- **L2037 EN**: Continues the surrounding expression or declaration: `frexpf, frexpl_f80, frexpl_f128, frexpl_ppcf128`.
  **L2037 CN**: 继续构造周围的表达式或声明：`frexpf, frexpl_f80, frexpl_f128, frexpl_ppcf128`。
- **L2038 EN**: Executes a standalone statement or declaration: `];`.
  **L2038 CN**: 执行一条独立语句或声明：`];`。
- **L2039 EN**: Blank line separating nearby declarations or logic blocks.
  **L2039 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2040 EN**: Initializes variable `MostPowI` from the right-hand expression.
  **L2040 CN**: 使用右侧表达式初始化变量 `MostPowI`。
- **L2041 EN**: Initializes variable `WindowsExclusions` from the right-hand expression.
  **L2041 CN**: 使用右侧表达式初始化变量 `WindowsExclusions`。
- **L2042 EN**: Blank line separating nearby declarations or logic blocks.
  **L2042 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2043 EN**: Comment explains nearby logic, invariants, or intent: `Targets which support windows should start with these as a base and`.
  **L2043 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Targets which support windows should start with these as a base and`。
- **L2044 EN**: Comment explains nearby logic, invariants, or intent: `add in calls for other OSes`.
  **L2044 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`add in calls for other OSes`。
- **L2045 EN**: Continues the surrounding expression or declaration: `defvar WinDefaultLibcallImplsBaseList =`.
  **L2045 CN**: 继续构造周围的表达式或声明：`defvar WinDefaultLibcallImplsBaseList =`。
- **L2046 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!listremove(DefaultRuntimeLibcallImplsBaseList,`.
  **L2046 CN**: 继续一个多行参数列表、初始化器或聚合项：`!listremove(DefaultRuntimeLibcallImplsBaseList,`。
- **L2047 EN**: Executes a standalone statement or declaration: `WindowsExclusions);`.
  **L2047 CN**: 执行一条独立语句或声明：`WindowsExclusions);`。
- **L2048 EN**: Blank line separating nearby declarations or logic blocks.
  **L2048 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 2049-2080

````tablegen
defvar WinDefaultLibcallImpls = (add WinDefaultLibcallImplsBaseList,
                                     ExceptionModelCalls);

defvar LibmHasFrexpF32 = LibcallImpls<(add frexpf), isNotOSWindowsOrIsCygwinMinGW>;
defvar LibmHasLdexpF32 = LibcallImpls<(add ldexpf), isNotOSWindowsOrIsCygwinMinGW>;

defvar LibmHasFrexpF80 = LibcallImpls<(add frexpl_f80), isNotOSWindowsOrIsCygwinMinGW>;
defvar LibmHasLdexpF80 = LibcallImpls<(add ldexpl_f80), isNotOSWindowsOrIsCygwinMinGW>;

defvar LibmHasFrexpF128 = LibcallImpls<(add frexpl_f128), isNotOSWindowsOrIsCygwinMinGW>;
defvar LibmHasLdexpF128 = LibcallImpls<(add ldexpl_f128), isNotOSWindowsOrIsCygwinMinGW>;

defvar has__stack_chk_fail = LibcallImpls<(add __stack_chk_fail), hasStackChkFail>;
defvar has__stack_chk_guard =
    LibcallImpls<(add __stack_chk_guard), hasStackChkFail>;
defvar has__stack_smash_handler = LibcallImpls<(add __stack_smash_handler), isOSOpenBSD>;
defvar has___guard_local = LibcallImpls<(add __guard_local), isOSOpenBSD>;

defvar DefaultStackProtector = (add has__stack_chk_fail, has__stack_chk_guard,
    has__stack_smash_handler, has___guard_local);

defvar LibcSafestackPointerAddress =
    LibcallImpls<(add __safestack_pointer_address), isAndroid>;

// compiler-rt sources suggest this is only available if
// (TT.isOSLinux() || TT.isOSFreeBSD() || TT.isOSNetBSD() ||
// TT.isOSSolaris()), but tests exist for other os targets.
defvar DefaultSafeStackGlobals = (add __safestack_unsafe_stack_ptr);

//===----------------------------------------------------------------------===//
// Objective-C Runtime Libcalls
//===----------------------------------------------------------------------===//
````
- **L2049 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `defvar WinDefaultLibcallImpls = (add WinDefaultLibcallImplsBaseList,`.
  **L2049 CN**: 继续一个多行参数列表、初始化器或聚合项：`defvar WinDefaultLibcallImpls = (add WinDefaultLibcallImplsBaseList,`。
- **L2050 EN**: Executes a standalone statement or declaration: `ExceptionModelCalls);`.
  **L2050 CN**: 执行一条独立语句或声明：`ExceptionModelCalls);`。
- **L2051 EN**: Blank line separating nearby declarations or logic blocks.
  **L2051 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2052 EN**: Initializes variable `LibmHasFrexpF32` from the right-hand expression.
  **L2052 CN**: 使用右侧表达式初始化变量 `LibmHasFrexpF32`。
- **L2053 EN**: Initializes variable `LibmHasLdexpF32` from the right-hand expression.
  **L2053 CN**: 使用右侧表达式初始化变量 `LibmHasLdexpF32`。
- **L2054 EN**: Blank line separating nearby declarations or logic blocks.
  **L2054 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2055 EN**: Initializes variable `LibmHasFrexpF80` from the right-hand expression.
  **L2055 CN**: 使用右侧表达式初始化变量 `LibmHasFrexpF80`。
- **L2056 EN**: Initializes variable `LibmHasLdexpF80` from the right-hand expression.
  **L2056 CN**: 使用右侧表达式初始化变量 `LibmHasLdexpF80`。
- **L2057 EN**: Blank line separating nearby declarations or logic blocks.
  **L2057 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2058 EN**: Initializes variable `LibmHasFrexpF128` from the right-hand expression.
  **L2058 CN**: 使用右侧表达式初始化变量 `LibmHasFrexpF128`。
- **L2059 EN**: Initializes variable `LibmHasLdexpF128` from the right-hand expression.
  **L2059 CN**: 使用右侧表达式初始化变量 `LibmHasLdexpF128`。
- **L2060 EN**: Blank line separating nearby declarations or logic blocks.
  **L2060 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2061 EN**: Initializes variable `has__stack_chk_fail` from the right-hand expression.
  **L2061 CN**: 使用右侧表达式初始化变量 `has__stack_chk_fail`。
- **L2062 EN**: Continues the surrounding expression or declaration: `defvar has__stack_chk_guard =`.
  **L2062 CN**: 继续构造周围的表达式或声明：`defvar has__stack_chk_guard =`。
- **L2063 EN**: Executes a call or declaration centered on `LibcallImpls<`.
  **L2063 CN**: 执行以 `LibcallImpls<` 为核心的调用或声明。
- **L2064 EN**: Initializes variable `has__stack_smash_handler` from the right-hand expression.
  **L2064 CN**: 使用右侧表达式初始化变量 `has__stack_smash_handler`。
- **L2065 EN**: Initializes variable `has___guard_local` from the right-hand expression.
  **L2065 CN**: 使用右侧表达式初始化变量 `has___guard_local`。
- **L2066 EN**: Blank line separating nearby declarations or logic blocks.
  **L2066 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2067 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `defvar DefaultStackProtector = (add has__stack_chk_fail, has__stack_chk_guard,`.
  **L2067 CN**: 继续一个多行参数列表、初始化器或聚合项：`defvar DefaultStackProtector = (add has__stack_chk_fail, has__stack_chk_guard,`。
- **L2068 EN**: Executes a standalone statement or declaration: `has__stack_smash_handler, has___guard_local);`.
  **L2068 CN**: 执行一条独立语句或声明：`has__stack_smash_handler, has___guard_local);`。
- **L2069 EN**: Blank line separating nearby declarations or logic blocks.
  **L2069 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2070 EN**: Continues the surrounding expression or declaration: `defvar LibcSafestackPointerAddress =`.
  **L2070 CN**: 继续构造周围的表达式或声明：`defvar LibcSafestackPointerAddress =`。
- **L2071 EN**: Executes a call or declaration centered on `LibcallImpls<`.
  **L2071 CN**: 执行以 `LibcallImpls<` 为核心的调用或声明。
- **L2072 EN**: Blank line separating nearby declarations or logic blocks.
  **L2072 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2073 EN**: Comment explains nearby logic, invariants, or intent: `compiler-rt sources suggest this is only available if`.
  **L2073 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`compiler-rt sources suggest this is only available if`。
- **L2074 EN**: Comment explains nearby logic, invariants, or intent: `(TT.isOSLinux() || TT.isOSFreeBSD() || TT.isOSNetBSD() ||`.
  **L2074 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`(TT.isOSLinux() || TT.isOSFreeBSD() || TT.isOSNetBSD() ||`。
- **L2075 EN**: Comment explains nearby logic, invariants, or intent: `TT.isOSSolaris()), but tests exist for other os targets.`.
  **L2075 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`TT.isOSSolaris()), but tests exist for other os targets.`。
- **L2076 EN**: Initializes variable `DefaultSafeStackGlobals` from the right-hand expression.
  **L2076 CN**: 使用右侧表达式初始化变量 `DefaultSafeStackGlobals`。
- **L2077 EN**: Blank line separating nearby declarations or logic blocks.
  **L2077 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2078 EN**: Banner comment marking a file or section boundary.
  **L2078 CN**: 横幅注释，用于标记文件或章节边界。
- **L2079 EN**: Comment explains nearby logic, invariants, or intent: `Objective-C Runtime Libcalls`.
  **L2079 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Objective-C Runtime Libcalls`。
- **L2080 EN**: Banner comment marking a file or section boundary.
  **L2080 CN**: 横幅注释，用于标记文件或章节边界。

### Lines 2081-2112

````tablegen

def objc_autorelease : RuntimeLibcallImpl<OBJC_AUTORELEASE>;
def objc_autoreleasePoolPop : RuntimeLibcallImpl<OBJC_AUTORELEASEPOOLPOP>;
def objc_autoreleasePoolPush : RuntimeLibcallImpl<OBJC_AUTORELEASEPOOLPUSH>;
def objc_autoreleaseReturnValue : RuntimeLibcallImpl<OBJC_AUTORELEASERETURNVALUE>;
def objc_copyWeak : RuntimeLibcallImpl<OBJC_COPYWEAK>;
def objc_destroyWeak : RuntimeLibcallImpl<OBJC_DESTROYWEAK>;
def objc_initWeak : RuntimeLibcallImpl<OBJC_INITWEAK>;
def objc_loadWeak : RuntimeLibcallImpl<OBJC_LOADWEAK>;
def objc_loadWeakRetained : RuntimeLibcallImpl<OBJC_LOADWEAKRETAINED>;
def objc_moveWeak : RuntimeLibcallImpl<OBJC_MOVEWEAK>;
def objc_release : RuntimeLibcallImpl<OBJC_RELEASE>;
def objc_retain : RuntimeLibcallImpl<OBJC_RETAIN>;
def objc_retainAutorelease : RuntimeLibcallImpl<OBJC_RETAINAUTORELEASE>;
def objc_retainAutoreleaseReturnValue : RuntimeLibcallImpl<OBJC_RETAINAUTORELEASERETURNVALUE>;
def objc_retainAutoreleasedReturnValue : RuntimeLibcallImpl<OBJC_RETAINAUTORELEASEDRETURNVALUE>;
def objc_claimAutoreleasedReturnValue : RuntimeLibcallImpl<OBJC_CLAIMAUTORELEASEDRETURNVALUE>;
def objc_retainBlock : RuntimeLibcallImpl<OBJC_RETAINBLOCK>;
def objc_storeStrong : RuntimeLibcallImpl<OBJC_STORESTRONG>;
def objc_storeWeak : RuntimeLibcallImpl<OBJC_STOREWEAK>;
def objc_unsafeClaimAutoreleasedReturnValue : RuntimeLibcallImpl<OBJC_UNSAFECLAIMAUTORELEASEDRETURNVALUE>;
def objc_retainedObject : RuntimeLibcallImpl<OBJC_RETAINEDOBJECT>;
def objc_unretainedObject : RuntimeLibcallImpl<OBJC_UNRETAINEDOBJECT>;
def objc_unretainedPointer : RuntimeLibcallImpl<OBJC_UNRETAINEDPOINTER>;
def objc_retain_autorelease : RuntimeLibcallImpl<OBJC_RETAIN_AUTORELEASE>;
def objc_sync_enter : RuntimeLibcallImpl<OBJC_SYNC_ENTER>;
def objc_sync_exit : RuntimeLibcallImpl<OBJC_SYNC_EXIT>;

//===----------------------------------------------------------------------===//
// AArch64 Runtime Libcalls
//===----------------------------------------------------------------------===//

````
- **L2081 EN**: Blank line separating nearby declarations or logic blocks.
  **L2081 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2082 EN**: Declares TableGen def `objc_autorelease`.
  **L2082 CN**: 声明 TableGen def `objc_autorelease`。
- **L2083 EN**: Declares TableGen def `objc_autoreleasePoolPop`.
  **L2083 CN**: 声明 TableGen def `objc_autoreleasePoolPop`。
- **L2084 EN**: Declares TableGen def `objc_autoreleasePoolPush`.
  **L2084 CN**: 声明 TableGen def `objc_autoreleasePoolPush`。
- **L2085 EN**: Declares TableGen def `objc_autoreleaseReturnValue`.
  **L2085 CN**: 声明 TableGen def `objc_autoreleaseReturnValue`。
- **L2086 EN**: Declares TableGen def `objc_copyWeak`.
  **L2086 CN**: 声明 TableGen def `objc_copyWeak`。
- **L2087 EN**: Declares TableGen def `objc_destroyWeak`.
  **L2087 CN**: 声明 TableGen def `objc_destroyWeak`。
- **L2088 EN**: Declares TableGen def `objc_initWeak`.
  **L2088 CN**: 声明 TableGen def `objc_initWeak`。
- **L2089 EN**: Declares TableGen def `objc_loadWeak`.
  **L2089 CN**: 声明 TableGen def `objc_loadWeak`。
- **L2090 EN**: Declares TableGen def `objc_loadWeakRetained`.
  **L2090 CN**: 声明 TableGen def `objc_loadWeakRetained`。
- **L2091 EN**: Declares TableGen def `objc_moveWeak`.
  **L2091 CN**: 声明 TableGen def `objc_moveWeak`。
- **L2092 EN**: Declares TableGen def `objc_release`.
  **L2092 CN**: 声明 TableGen def `objc_release`。
- **L2093 EN**: Declares TableGen def `objc_retain`.
  **L2093 CN**: 声明 TableGen def `objc_retain`。
- **L2094 EN**: Declares TableGen def `objc_retainAutorelease`.
  **L2094 CN**: 声明 TableGen def `objc_retainAutorelease`。
- **L2095 EN**: Declares TableGen def `objc_retainAutoreleaseReturnValue`.
  **L2095 CN**: 声明 TableGen def `objc_retainAutoreleaseReturnValue`。
- **L2096 EN**: Declares TableGen def `objc_retainAutoreleasedReturnValue`.
  **L2096 CN**: 声明 TableGen def `objc_retainAutoreleasedReturnValue`。
- **L2097 EN**: Declares TableGen def `objc_claimAutoreleasedReturnValue`.
  **L2097 CN**: 声明 TableGen def `objc_claimAutoreleasedReturnValue`。
- **L2098 EN**: Declares TableGen def `objc_retainBlock`.
  **L2098 CN**: 声明 TableGen def `objc_retainBlock`。
- **L2099 EN**: Declares TableGen def `objc_storeStrong`.
  **L2099 CN**: 声明 TableGen def `objc_storeStrong`。
- **L2100 EN**: Declares TableGen def `objc_storeWeak`.
  **L2100 CN**: 声明 TableGen def `objc_storeWeak`。
- **L2101 EN**: Declares TableGen def `objc_unsafeClaimAutoreleasedReturnValue`.
  **L2101 CN**: 声明 TableGen def `objc_unsafeClaimAutoreleasedReturnValue`。
- **L2102 EN**: Declares TableGen def `objc_retainedObject`.
  **L2102 CN**: 声明 TableGen def `objc_retainedObject`。
- **L2103 EN**: Declares TableGen def `objc_unretainedObject`.
  **L2103 CN**: 声明 TableGen def `objc_unretainedObject`。
- **L2104 EN**: Declares TableGen def `objc_unretainedPointer`.
  **L2104 CN**: 声明 TableGen def `objc_unretainedPointer`。
- **L2105 EN**: Declares TableGen def `objc_retain_autorelease`.
  **L2105 CN**: 声明 TableGen def `objc_retain_autorelease`。
- **L2106 EN**: Declares TableGen def `objc_sync_enter`.
  **L2106 CN**: 声明 TableGen def `objc_sync_enter`。
- **L2107 EN**: Declares TableGen def `objc_sync_exit`.
  **L2107 CN**: 声明 TableGen def `objc_sync_exit`。
- **L2108 EN**: Blank line separating nearby declarations or logic blocks.
  **L2108 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2109 EN**: Banner comment marking a file or section boundary.
  **L2109 CN**: 横幅注释，用于标记文件或章节边界。
- **L2110 EN**: Comment explains nearby logic, invariants, or intent: `AArch64 Runtime Libcalls`.
  **L2110 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`AArch64 Runtime Libcalls`。
- **L2111 EN**: Banner comment marking a file or section boundary.
  **L2111 CN**: 横幅注释，用于标记文件或章节边界。
- **L2112 EN**: Blank line separating nearby declarations or logic blocks.
  **L2112 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 2113-2144

````tablegen
defset list<RuntimeLibcallImpl> AArch64LibcallImpls = {
  foreach MemSize = [1, 2, 4, 8, 16] in {
    defm __aarch64_cas#MemSize
        : AtomicOrderSizeLibcallImpl<"OUTLINE_ATOMIC_CAS"#MemSize>;
  }

  foreach MemSize = [1, 2, 4, 8] in {
    defm __aarch64_swp#MemSize
        : AtomicOrderSizeLibcallImpl<"OUTLINE_ATOMIC_SWP"#MemSize>;
    defm __aarch64_ldadd#MemSize
        : AtomicOrderSizeLibcallImpl<"OUTLINE_ATOMIC_LDADD"#MemSize>;
    defm __aarch64_ldset#MemSize
        : AtomicOrderSizeLibcallImpl<"OUTLINE_ATOMIC_LDSET"#MemSize>;
    defm __aarch64_ldclr#MemSize
        : AtomicOrderSizeLibcallImpl<"OUTLINE_ATOMIC_LDCLR"#MemSize>;
    defm __aarch64_ldeor#MemSize
        : AtomicOrderSizeLibcallImpl<"OUTLINE_ATOMIC_LDEOR"#MemSize>;
  }

  def __arm_sc_memcpy : RuntimeLibcallImpl<SC_MEMCPY>;
  def __arm_sc_memmove : RuntimeLibcallImpl<SC_MEMMOVE>;
  def __arm_sc_memset : RuntimeLibcallImpl<SC_MEMSET>;
  def __arm_sc_memchr : RuntimeLibcallImpl<SC_MEMCHR>;
} // End AArch64LibcallImpls

def __arm_sme_state : RuntimeLibcallImpl<SMEABI_SME_STATE>;
def __arm_tpidr2_save : RuntimeLibcallImpl<SMEABI_TPIDR2_SAVE>;
def __arm_za_disable : RuntimeLibcallImpl<SMEABI_ZA_DISABLE>;
def __arm_tpidr2_restore : RuntimeLibcallImpl<SMEABI_TPIDR2_RESTORE>;
def __arm_get_current_vg : RuntimeLibcallImpl<SMEABI_GET_CURRENT_VG>;
def __arm_sme_state_size : RuntimeLibcallImpl<SMEABI_SME_STATE_SIZE>;
def __arm_sme_save : RuntimeLibcallImpl<SMEABI_SME_SAVE>;
````
- **L2113 EN**: Continues the surrounding expression or declaration: `defset list<RuntimeLibcallImpl> AArch64LibcallImpls = {`.
  **L2113 CN**: 继续构造周围的表达式或声明：`defset list<RuntimeLibcallImpl> AArch64LibcallImpls = {`。
- **L2114 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L2114 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2115 EN**: Declares TableGen defm `__aarch64_cas#MemSize`.
  **L2115 CN**: 声明 TableGen defm `__aarch64_cas#MemSize`。
- **L2116 EN**: Executes a standalone statement or declaration: `: AtomicOrderSizeLibcallImpl<"OUTLINE_ATOMIC_CAS"#MemSize>;`.
  **L2116 CN**: 执行一条独立语句或声明：`: AtomicOrderSizeLibcallImpl<"OUTLINE_ATOMIC_CAS"#MemSize>;`。
- **L2117 EN**: Closes the current lexical scope or compound statement.
  **L2117 CN**: 结束当前词法作用域或复合语句块。
- **L2118 EN**: Blank line separating nearby declarations or logic blocks.
  **L2118 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2119 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L2119 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2120 EN**: Declares TableGen defm `__aarch64_swp#MemSize`.
  **L2120 CN**: 声明 TableGen defm `__aarch64_swp#MemSize`。
- **L2121 EN**: Executes a standalone statement or declaration: `: AtomicOrderSizeLibcallImpl<"OUTLINE_ATOMIC_SWP"#MemSize>;`.
  **L2121 CN**: 执行一条独立语句或声明：`: AtomicOrderSizeLibcallImpl<"OUTLINE_ATOMIC_SWP"#MemSize>;`。
- **L2122 EN**: Declares TableGen defm `__aarch64_ldadd#MemSize`.
  **L2122 CN**: 声明 TableGen defm `__aarch64_ldadd#MemSize`。
- **L2123 EN**: Executes a standalone statement or declaration: `: AtomicOrderSizeLibcallImpl<"OUTLINE_ATOMIC_LDADD"#MemSize>;`.
  **L2123 CN**: 执行一条独立语句或声明：`: AtomicOrderSizeLibcallImpl<"OUTLINE_ATOMIC_LDADD"#MemSize>;`。
- **L2124 EN**: Declares TableGen defm `__aarch64_ldset#MemSize`.
  **L2124 CN**: 声明 TableGen defm `__aarch64_ldset#MemSize`。
- **L2125 EN**: Executes a standalone statement or declaration: `: AtomicOrderSizeLibcallImpl<"OUTLINE_ATOMIC_LDSET"#MemSize>;`.
  **L2125 CN**: 执行一条独立语句或声明：`: AtomicOrderSizeLibcallImpl<"OUTLINE_ATOMIC_LDSET"#MemSize>;`。
- **L2126 EN**: Declares TableGen defm `__aarch64_ldclr#MemSize`.
  **L2126 CN**: 声明 TableGen defm `__aarch64_ldclr#MemSize`。
- **L2127 EN**: Executes a standalone statement or declaration: `: AtomicOrderSizeLibcallImpl<"OUTLINE_ATOMIC_LDCLR"#MemSize>;`.
  **L2127 CN**: 执行一条独立语句或声明：`: AtomicOrderSizeLibcallImpl<"OUTLINE_ATOMIC_LDCLR"#MemSize>;`。
- **L2128 EN**: Declares TableGen defm `__aarch64_ldeor#MemSize`.
  **L2128 CN**: 声明 TableGen defm `__aarch64_ldeor#MemSize`。
- **L2129 EN**: Executes a standalone statement or declaration: `: AtomicOrderSizeLibcallImpl<"OUTLINE_ATOMIC_LDEOR"#MemSize>;`.
  **L2129 CN**: 执行一条独立语句或声明：`: AtomicOrderSizeLibcallImpl<"OUTLINE_ATOMIC_LDEOR"#MemSize>;`。
- **L2130 EN**: Closes the current lexical scope or compound statement.
  **L2130 CN**: 结束当前词法作用域或复合语句块。
- **L2131 EN**: Blank line separating nearby declarations or logic blocks.
  **L2131 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2132 EN**: Declares TableGen def `__arm_sc_memcpy`.
  **L2132 CN**: 声明 TableGen def `__arm_sc_memcpy`。
- **L2133 EN**: Declares TableGen def `__arm_sc_memmove`.
  **L2133 CN**: 声明 TableGen def `__arm_sc_memmove`。
- **L2134 EN**: Declares TableGen def `__arm_sc_memset`.
  **L2134 CN**: 声明 TableGen def `__arm_sc_memset`。
- **L2135 EN**: Declares TableGen def `__arm_sc_memchr`.
  **L2135 CN**: 声明 TableGen def `__arm_sc_memchr`。
- **L2136 EN**: Continues the surrounding expression or declaration: `} // End AArch64LibcallImpls`.
  **L2136 CN**: 继续构造周围的表达式或声明：`} // End AArch64LibcallImpls`。
- **L2137 EN**: Blank line separating nearby declarations or logic blocks.
  **L2137 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2138 EN**: Declares TableGen def `__arm_sme_state`.
  **L2138 CN**: 声明 TableGen def `__arm_sme_state`。
- **L2139 EN**: Declares TableGen def `__arm_tpidr2_save`.
  **L2139 CN**: 声明 TableGen def `__arm_tpidr2_save`。
- **L2140 EN**: Declares TableGen def `__arm_za_disable`.
  **L2140 CN**: 声明 TableGen def `__arm_za_disable`。
- **L2141 EN**: Declares TableGen def `__arm_tpidr2_restore`.
  **L2141 CN**: 声明 TableGen def `__arm_tpidr2_restore`。
- **L2142 EN**: Declares TableGen def `__arm_get_current_vg`.
  **L2142 CN**: 声明 TableGen def `__arm_get_current_vg`。
- **L2143 EN**: Declares TableGen def `__arm_sme_state_size`.
  **L2143 CN**: 声明 TableGen def `__arm_sme_state_size`。
- **L2144 EN**: Declares TableGen def `__arm_sme_save`.
  **L2144 CN**: 声明 TableGen def `__arm_sme_save`。

### Lines 2145-2176

````tablegen
def __arm_sme_restore : RuntimeLibcallImpl<SMEABI_SME_RESTORE>;

def SMEABI_LibCalls_PreserveMost_From_X0 : LibcallsWithCC<(add
  __arm_tpidr2_save,
  __arm_za_disable,
  __arm_tpidr2_restore),
  SMEABI_PreserveMost_From_X0>;

def SMEABI_LibCalls_PreserveMost_From_X1 : LibcallsWithCC<(add
  __arm_get_current_vg,
  __arm_sme_state_size,
  __arm_sme_save,
  __arm_sme_restore),
  SMEABI_PreserveMost_From_X1>;

def SMEABI_LibCalls_PreserveMost_From_X2 : LibcallsWithCC<(add
  __arm_sme_state),
  SMEABI_PreserveMost_From_X2>;

def isAArch64_ExceptArm64EC
    : RuntimeLibcallPredicate<"(TT.isAArch64() && !TT.isWindowsArm64EC())">;
def isWindowsArm64EC : RuntimeLibcallPredicate<"TT.isWindowsArm64EC()">;
def isAArch64_ILP64  : RuntimeLibcallPredicate<"TT.isAArch64(64)">;


def AArch64SystemLibrary : SystemRuntimeLibrary<
  isAArch64_ExceptArm64EC,
  (add WinDefaultLibcallImpls,
       LibcallImpls<(add __powisf2, __powidf2), isNotOSMSVCRT>,
       LibmHasFrexpF32, LibmHasLdexpF32,
       LibmHasFrexpF128, LibmHasLdexpF128,
       AArch64LibcallImpls,
````
- **L2145 EN**: Declares TableGen def `__arm_sme_restore`.
  **L2145 CN**: 声明 TableGen def `__arm_sme_restore`。
- **L2146 EN**: Blank line separating nearby declarations or logic blocks.
  **L2146 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2147 EN**: Declares TableGen def `SMEABI_LibCalls_PreserveMost_From_X0`.
  **L2147 CN**: 声明 TableGen def `SMEABI_LibCalls_PreserveMost_From_X0`。
- **L2148 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__arm_tpidr2_save,`.
  **L2148 CN**: 继续一个多行参数列表、初始化器或聚合项：`__arm_tpidr2_save,`。
- **L2149 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__arm_za_disable,`.
  **L2149 CN**: 继续一个多行参数列表、初始化器或聚合项：`__arm_za_disable,`。
- **L2150 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__arm_tpidr2_restore),`.
  **L2150 CN**: 继续一个多行参数列表、初始化器或聚合项：`__arm_tpidr2_restore),`。
- **L2151 EN**: Executes a standalone statement or declaration: `SMEABI_PreserveMost_From_X0>;`.
  **L2151 CN**: 执行一条独立语句或声明：`SMEABI_PreserveMost_From_X0>;`。
- **L2152 EN**: Blank line separating nearby declarations or logic blocks.
  **L2152 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2153 EN**: Declares TableGen def `SMEABI_LibCalls_PreserveMost_From_X1`.
  **L2153 CN**: 声明 TableGen def `SMEABI_LibCalls_PreserveMost_From_X1`。
- **L2154 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__arm_get_current_vg,`.
  **L2154 CN**: 继续一个多行参数列表、初始化器或聚合项：`__arm_get_current_vg,`。
- **L2155 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__arm_sme_state_size,`.
  **L2155 CN**: 继续一个多行参数列表、初始化器或聚合项：`__arm_sme_state_size,`。
- **L2156 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__arm_sme_save,`.
  **L2156 CN**: 继续一个多行参数列表、初始化器或聚合项：`__arm_sme_save,`。
- **L2157 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__arm_sme_restore),`.
  **L2157 CN**: 继续一个多行参数列表、初始化器或聚合项：`__arm_sme_restore),`。
- **L2158 EN**: Executes a standalone statement or declaration: `SMEABI_PreserveMost_From_X1>;`.
  **L2158 CN**: 执行一条独立语句或声明：`SMEABI_PreserveMost_From_X1>;`。
- **L2159 EN**: Blank line separating nearby declarations or logic blocks.
  **L2159 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2160 EN**: Declares TableGen def `SMEABI_LibCalls_PreserveMost_From_X2`.
  **L2160 CN**: 声明 TableGen def `SMEABI_LibCalls_PreserveMost_From_X2`。
- **L2161 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__arm_sme_state),`.
  **L2161 CN**: 继续一个多行参数列表、初始化器或聚合项：`__arm_sme_state),`。
- **L2162 EN**: Executes a standalone statement or declaration: `SMEABI_PreserveMost_From_X2>;`.
  **L2162 CN**: 执行一条独立语句或声明：`SMEABI_PreserveMost_From_X2>;`。
- **L2163 EN**: Blank line separating nearby declarations or logic blocks.
  **L2163 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2164 EN**: Declares TableGen def `isAArch64_ExceptArm64EC`.
  **L2164 CN**: 声明 TableGen def `isAArch64_ExceptArm64EC`。
- **L2165 EN**: Executes a call or declaration centered on `RuntimeLibcallPredicate<"`.
  **L2165 CN**: 执行以 `RuntimeLibcallPredicate<"` 为核心的调用或声明。
- **L2166 EN**: Declares TableGen def `isWindowsArm64EC`.
  **L2166 CN**: 声明 TableGen def `isWindowsArm64EC`。
- **L2167 EN**: Declares TableGen def `isAArch64_ILP64`.
  **L2167 CN**: 声明 TableGen def `isAArch64_ILP64`。
- **L2168 EN**: Blank line separating nearby declarations or logic blocks.
  **L2168 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2169 EN**: Blank line separating nearby declarations or logic blocks.
  **L2169 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2170 EN**: Declares TableGen def `AArch64SystemLibrary`.
  **L2170 CN**: 声明 TableGen def `AArch64SystemLibrary`。
- **L2171 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `isAArch64_ExceptArm64EC,`.
  **L2171 CN**: 继续一个多行参数列表、初始化器或聚合项：`isAArch64_ExceptArm64EC,`。
- **L2172 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(add WinDefaultLibcallImpls,`.
  **L2172 CN**: 继续一个多行参数列表、初始化器或聚合项：`(add WinDefaultLibcallImpls,`。
- **L2173 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LibcallImpls<(add __powisf2, __powidf2), isNotOSMSVCRT>,`.
  **L2173 CN**: 继续一个多行参数列表、初始化器或聚合项：`LibcallImpls<(add __powisf2, __powidf2), isNotOSMSVCRT>,`。
- **L2174 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LibmHasFrexpF32, LibmHasLdexpF32,`.
  **L2174 CN**: 继续一个多行参数列表、初始化器或聚合项：`LibmHasFrexpF32, LibmHasLdexpF32,`。
- **L2175 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LibmHasFrexpF128, LibmHasLdexpF128,`.
  **L2175 CN**: 继续一个多行参数列表、初始化器或聚合项：`LibmHasFrexpF128, LibmHasLdexpF128,`。
- **L2176 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `AArch64LibcallImpls,`.
  **L2176 CN**: 继续一个多行参数列表、初始化器或聚合项：`AArch64LibcallImpls,`。

### Lines 2177-2208

````tablegen
       LibcallImpls<(add Int128RTLibcalls), isAArch64_ILP64>,
       LibcallImpls<(add bzero), isOSDarwin>,
       DarwinExp10, DarwinSinCosStret, DarwinMemsetPattern,
       MacOSUnlockedIO,
       LibmHasSinCosF32, LibmHasSinCosF64, LibmHasSinCosF128,
       DefaultLibmExp10,
       DefaultStackProtector,
       SecurityCheckCookieIfWinMSVC,
       LibcallImpls<(add __chkstk), isOSWindows>,
       SMEABI_LibCalls_PreserveMost_From_X0,
       SMEABI_LibCalls_PreserveMost_From_X1,
       SMEABI_LibCalls_PreserveMost_From_X2,
       DefaultSafeStackGlobals,
       LibcSafestackPointerAddress)
>;

// Prepend a # to every name
defset list<RuntimeLibcallImpl> WinArm64ECDefaultRuntimeLibcallImpls = {
  foreach libcall = WinDefaultLibcallImplsBaseList in {
    def arm64ec_#libcall : DuplicateLibcallImplWithPrefix<libcall, "#">;
  }

  foreach libcall = AArch64LibcallImpls in {
    def arm64ec_#libcall : DuplicateLibcallImplWithPrefix<libcall, "#">;
  }
}

def arm64ec___stack_chk_fail : DuplicateLibcallImplWithPrefix<__stack_chk_fail, "#">;

defset list<RuntimeLibcallImpl> WinArm64ECDefaultExceptionHandlingLibcalls = {
  foreach libcall = DefaultExceptionHandlingLibcalls in {
    def arm64ec_#libcall : DuplicateLibcallImplWithPrefix<libcall, "#">;
````
- **L2177 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LibcallImpls<(add Int128RTLibcalls), isAArch64_ILP64>,`.
  **L2177 CN**: 继续一个多行参数列表、初始化器或聚合项：`LibcallImpls<(add Int128RTLibcalls), isAArch64_ILP64>,`。
- **L2178 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LibcallImpls<(add bzero), isOSDarwin>,`.
  **L2178 CN**: 继续一个多行参数列表、初始化器或聚合项：`LibcallImpls<(add bzero), isOSDarwin>,`。
- **L2179 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DarwinExp10, DarwinSinCosStret, DarwinMemsetPattern,`.
  **L2179 CN**: 继续一个多行参数列表、初始化器或聚合项：`DarwinExp10, DarwinSinCosStret, DarwinMemsetPattern,`。
- **L2180 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `MacOSUnlockedIO,`.
  **L2180 CN**: 继续一个多行参数列表、初始化器或聚合项：`MacOSUnlockedIO,`。
- **L2181 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LibmHasSinCosF32, LibmHasSinCosF64, LibmHasSinCosF128,`.
  **L2181 CN**: 继续一个多行参数列表、初始化器或聚合项：`LibmHasSinCosF32, LibmHasSinCosF64, LibmHasSinCosF128,`。
- **L2182 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultLibmExp10,`.
  **L2182 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultLibmExp10,`。
- **L2183 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultStackProtector,`.
  **L2183 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultStackProtector,`。
- **L2184 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `SecurityCheckCookieIfWinMSVC,`.
  **L2184 CN**: 继续一个多行参数列表、初始化器或聚合项：`SecurityCheckCookieIfWinMSVC,`。
- **L2185 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LibcallImpls<(add __chkstk), isOSWindows>,`.
  **L2185 CN**: 继续一个多行参数列表、初始化器或聚合项：`LibcallImpls<(add __chkstk), isOSWindows>,`。
- **L2186 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `SMEABI_LibCalls_PreserveMost_From_X0,`.
  **L2186 CN**: 继续一个多行参数列表、初始化器或聚合项：`SMEABI_LibCalls_PreserveMost_From_X0,`。
- **L2187 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `SMEABI_LibCalls_PreserveMost_From_X1,`.
  **L2187 CN**: 继续一个多行参数列表、初始化器或聚合项：`SMEABI_LibCalls_PreserveMost_From_X1,`。
- **L2188 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `SMEABI_LibCalls_PreserveMost_From_X2,`.
  **L2188 CN**: 继续一个多行参数列表、初始化器或聚合项：`SMEABI_LibCalls_PreserveMost_From_X2,`。
- **L2189 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultSafeStackGlobals,`.
  **L2189 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultSafeStackGlobals,`。
- **L2190 EN**: Continues the surrounding expression or declaration: `LibcSafestackPointerAddress)`.
  **L2190 CN**: 继续构造周围的表达式或声明：`LibcSafestackPointerAddress)`。
- **L2191 EN**: Executes a standalone statement or declaration: `>;`.
  **L2191 CN**: 执行一条独立语句或声明：`>;`。
- **L2192 EN**: Blank line separating nearby declarations or logic blocks.
  **L2192 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2193 EN**: Comment explains nearby logic, invariants, or intent: `Prepend a # to every name`.
  **L2193 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Prepend a # to every name`。
- **L2194 EN**: Continues the surrounding expression or declaration: `defset list<RuntimeLibcallImpl> WinArm64ECDefaultRuntimeLibcallImpls = {`.
  **L2194 CN**: 继续构造周围的表达式或声明：`defset list<RuntimeLibcallImpl> WinArm64ECDefaultRuntimeLibcallImpls = {`。
- **L2195 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L2195 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2196 EN**: Declares TableGen def `arm64ec_#libcall`.
  **L2196 CN**: 声明 TableGen def `arm64ec_#libcall`。
- **L2197 EN**: Closes the current lexical scope or compound statement.
  **L2197 CN**: 结束当前词法作用域或复合语句块。
- **L2198 EN**: Blank line separating nearby declarations or logic blocks.
  **L2198 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2199 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L2199 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2200 EN**: Declares TableGen def `arm64ec_#libcall`.
  **L2200 CN**: 声明 TableGen def `arm64ec_#libcall`。
- **L2201 EN**: Closes the current lexical scope or compound statement.
  **L2201 CN**: 结束当前词法作用域或复合语句块。
- **L2202 EN**: Closes the current lexical scope or compound statement.
  **L2202 CN**: 结束当前词法作用域或复合语句块。
- **L2203 EN**: Blank line separating nearby declarations or logic blocks.
  **L2203 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2204 EN**: Declares TableGen def `arm64ec___stack_chk_fail`.
  **L2204 CN**: 声明 TableGen def `arm64ec___stack_chk_fail`。
- **L2205 EN**: Blank line separating nearby declarations or logic blocks.
  **L2205 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2206 EN**: Continues the surrounding expression or declaration: `defset list<RuntimeLibcallImpl> WinArm64ECDefaultExceptionHandlingLibcalls = {`.
  **L2206 CN**: 继续构造周围的表达式或声明：`defset list<RuntimeLibcallImpl> WinArm64ECDefaultExceptionHandlingLibcalls = {`。
- **L2207 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L2207 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2208 EN**: Declares TableGen def `arm64ec_#libcall`.
  **L2208 CN**: 声明 TableGen def `arm64ec_#libcall`。

### Lines 2209-2240

````tablegen
  }
}

defset list<RuntimeLibcallImpl> WinArm64ECSjLjExceptionHandlingLibcalls = {
  foreach libcall = SjLjExceptionHandlingLibcalls in {
    def arm64ec_#libcall : DuplicateLibcallImplWithPrefix<libcall, "#">;
  }
}

defvar ExceptionModelCallsArm64EC = (add
  LibcallImpls<(add arm64ec___cxa_end_cleanup),
               ExceptionModelIsNotNone>,
  LibcallImpls<(add arm64ec__Unwind_Resume),
               ExceptionModelHasUnwindResume>,
  LibcallImpls<(add WinArm64ECSjLjExceptionHandlingLibcalls),
               ExceptionModelIsSjLj>
);

def WindowsARM64ECSystemLibrary
    : SystemRuntimeLibrary<isWindowsArm64EC,
                           (add WinArm64ECDefaultRuntimeLibcallImpls,
                                AvailableIf<arm64ec___stack_chk_fail, hasStackChkFail>,
                                AvailableIf<__stack_chk_guard, hasStackChkFail>,
                                LibcallImpls<(add __security_check_cookie_arm64ec,
                                                  __security_cookie),
                                              isWindowsMSVCEnvironment>,
                                __chkstk_arm64ec,
                                ExceptionModelCallsArm64EC)>;

//===----------------------------------------------------------------------===//
// AMDGPU Runtime Libcalls
//===----------------------------------------------------------------------===//
````
- **L2209 EN**: Closes the current lexical scope or compound statement.
  **L2209 CN**: 结束当前词法作用域或复合语句块。
- **L2210 EN**: Closes the current lexical scope or compound statement.
  **L2210 CN**: 结束当前词法作用域或复合语句块。
- **L2211 EN**: Blank line separating nearby declarations or logic blocks.
  **L2211 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2212 EN**: Continues the surrounding expression or declaration: `defset list<RuntimeLibcallImpl> WinArm64ECSjLjExceptionHandlingLibcalls = {`.
  **L2212 CN**: 继续构造周围的表达式或声明：`defset list<RuntimeLibcallImpl> WinArm64ECSjLjExceptionHandlingLibcalls = {`。
- **L2213 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L2213 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2214 EN**: Declares TableGen def `arm64ec_#libcall`.
  **L2214 CN**: 声明 TableGen def `arm64ec_#libcall`。
- **L2215 EN**: Closes the current lexical scope or compound statement.
  **L2215 CN**: 结束当前词法作用域或复合语句块。
- **L2216 EN**: Closes the current lexical scope or compound statement.
  **L2216 CN**: 结束当前词法作用域或复合语句块。
- **L2217 EN**: Blank line separating nearby declarations or logic blocks.
  **L2217 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2218 EN**: Continues the surrounding expression or declaration: `defvar ExceptionModelCallsArm64EC = (add`.
  **L2218 CN**: 继续构造周围的表达式或声明：`defvar ExceptionModelCallsArm64EC = (add`。
- **L2219 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LibcallImpls<(add arm64ec___cxa_end_cleanup),`.
  **L2219 CN**: 继续一个多行参数列表、初始化器或聚合项：`LibcallImpls<(add arm64ec___cxa_end_cleanup),`。
- **L2220 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ExceptionModelIsNotNone>,`.
  **L2220 CN**: 继续一个多行参数列表、初始化器或聚合项：`ExceptionModelIsNotNone>,`。
- **L2221 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LibcallImpls<(add arm64ec__Unwind_Resume),`.
  **L2221 CN**: 继续一个多行参数列表、初始化器或聚合项：`LibcallImpls<(add arm64ec__Unwind_Resume),`。
- **L2222 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ExceptionModelHasUnwindResume>,`.
  **L2222 CN**: 继续一个多行参数列表、初始化器或聚合项：`ExceptionModelHasUnwindResume>,`。
- **L2223 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LibcallImpls<(add WinArm64ECSjLjExceptionHandlingLibcalls),`.
  **L2223 CN**: 继续一个多行参数列表、初始化器或聚合项：`LibcallImpls<(add WinArm64ECSjLjExceptionHandlingLibcalls),`。
- **L2224 EN**: Continues the surrounding expression or declaration: `ExceptionModelIsSjLj>`.
  **L2224 CN**: 继续构造周围的表达式或声明：`ExceptionModelIsSjLj>`。
- **L2225 EN**: Executes a standalone statement or declaration: `);`.
  **L2225 CN**: 执行一条独立语句或声明：`);`。
- **L2226 EN**: Blank line separating nearby declarations or logic blocks.
  **L2226 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2227 EN**: Declares TableGen def `WindowsARM64ECSystemLibrary`.
  **L2227 CN**: 声明 TableGen def `WindowsARM64ECSystemLibrary`。
- **L2228 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: SystemRuntimeLibrary<isWindowsArm64EC,`.
  **L2228 CN**: 继续一个多行参数列表、初始化器或聚合项：`: SystemRuntimeLibrary<isWindowsArm64EC,`。
- **L2229 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(add WinArm64ECDefaultRuntimeLibcallImpls,`.
  **L2229 CN**: 继续一个多行参数列表、初始化器或聚合项：`(add WinArm64ECDefaultRuntimeLibcallImpls,`。
- **L2230 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `AvailableIf<arm64ec___stack_chk_fail, hasStackChkFail>,`.
  **L2230 CN**: 继续一个多行参数列表、初始化器或聚合项：`AvailableIf<arm64ec___stack_chk_fail, hasStackChkFail>,`。
- **L2231 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `AvailableIf<__stack_chk_guard, hasStackChkFail>,`.
  **L2231 CN**: 继续一个多行参数列表、初始化器或聚合项：`AvailableIf<__stack_chk_guard, hasStackChkFail>,`。
- **L2232 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LibcallImpls<(add __security_check_cookie_arm64ec,`.
  **L2232 CN**: 继续一个多行参数列表、初始化器或聚合项：`LibcallImpls<(add __security_check_cookie_arm64ec,`。
- **L2233 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__security_cookie),`.
  **L2233 CN**: 继续一个多行参数列表、初始化器或聚合项：`__security_cookie),`。
- **L2234 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `isWindowsMSVCEnvironment>,`.
  **L2234 CN**: 继续一个多行参数列表、初始化器或聚合项：`isWindowsMSVCEnvironment>,`。
- **L2235 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__chkstk_arm64ec,`.
  **L2235 CN**: 继续一个多行参数列表、初始化器或聚合项：`__chkstk_arm64ec,`。
- **L2236 EN**: Executes a standalone statement or declaration: `ExceptionModelCallsArm64EC)>;`.
  **L2236 CN**: 执行一条独立语句或声明：`ExceptionModelCallsArm64EC)>;`。
- **L2237 EN**: Blank line separating nearby declarations or logic blocks.
  **L2237 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2238 EN**: Banner comment marking a file or section boundary.
  **L2238 CN**: 横幅注释，用于标记文件或章节边界。
- **L2239 EN**: Comment explains nearby logic, invariants, or intent: `AMDGPU Runtime Libcalls`.
  **L2239 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`AMDGPU Runtime Libcalls`。
- **L2240 EN**: Banner comment marking a file or section boundary.
  **L2240 CN**: 横幅注释，用于标记文件或章节边界。

### Lines 2241-2272

````tablegen

def isAMDGPU : RuntimeLibcallPredicate<"TT.isAMDGPU()">;

// No calls.
def AMDGPUSystemLibrary : SystemRuntimeLibrary<isAMDGPU, (add)>;

//===----------------------------------------------------------------------===//
// ARM Runtime Libcalls
//===----------------------------------------------------------------------===//

def isTargetAEABIAndAAPCS_ABI : RuntimeLibcallPredicate<
  [{TT.isTargetAEABI() && isAAPCS_ABI(TT, ABIName)}]>;

// if (isTargetMachO()) {
// if (Subtarget->isThumb() && Subtarget->hasVFP2Base() &&
//  Subtarget->hasARMOps() && !Subtarget->useSoftFloat()) {

def __addsf3vfp : RuntimeLibcallImpl<ADD_F32>;
def __subsf3vfp : RuntimeLibcallImpl<SUB_F32>;
def __mulsf3vfp : RuntimeLibcallImpl<MUL_F32>;
def __divsf3vfp : RuntimeLibcallImpl<DIV_F32>;

// Double-precision floating-point arithmetic.
def __adddf3vfp : RuntimeLibcallImpl<ADD_F64>;
def __subdf3vfp : RuntimeLibcallImpl<SUB_F64>;
def __muldf3vfp : RuntimeLibcallImpl<MUL_F64>;
def __divdf3vfp : RuntimeLibcallImpl<DIV_F64>;

// Single-precision comparisons.

// TODO: Track setcc type
def __eqsf2vfp : RuntimeLibcallImpl<OEQ_F32>; // CmpInst::ICMP_NE
````
- **L2241 EN**: Blank line separating nearby declarations or logic blocks.
  **L2241 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2242 EN**: Declares TableGen def `isAMDGPU`.
  **L2242 CN**: 声明 TableGen def `isAMDGPU`。
- **L2243 EN**: Blank line separating nearby declarations or logic blocks.
  **L2243 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2244 EN**: Comment explains nearby logic, invariants, or intent: `No calls.`.
  **L2244 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`No calls.`。
- **L2245 EN**: Declares TableGen def `AMDGPUSystemLibrary`.
  **L2245 CN**: 声明 TableGen def `AMDGPUSystemLibrary`。
- **L2246 EN**: Blank line separating nearby declarations or logic blocks.
  **L2246 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2247 EN**: Banner comment marking a file or section boundary.
  **L2247 CN**: 横幅注释，用于标记文件或章节边界。
- **L2248 EN**: Comment explains nearby logic, invariants, or intent: `ARM Runtime Libcalls`.
  **L2248 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`ARM Runtime Libcalls`。
- **L2249 EN**: Banner comment marking a file or section boundary.
  **L2249 CN**: 横幅注释，用于标记文件或章节边界。
- **L2250 EN**: Blank line separating nearby declarations or logic blocks.
  **L2250 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2251 EN**: Declares TableGen def `isTargetAEABIAndAAPCS_ABI`.
  **L2251 CN**: 声明 TableGen def `isTargetAEABIAndAAPCS_ABI`。
- **L2252 EN**: Executes a call or declaration centered on `[{TT.isTargetAEABI`.
  **L2252 CN**: 执行以 `[{TT.isTargetAEABI` 为核心的调用或声明。
- **L2253 EN**: Blank line separating nearby declarations or logic blocks.
  **L2253 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2254 EN**: Comment explains nearby logic, invariants, or intent: `if (isTargetMachO()) {`.
  **L2254 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`if (isTargetMachO()) {`。
- **L2255 EN**: Comment explains nearby logic, invariants, or intent: `if (Subtarget->isThumb() && Subtarget->hasVFP2Base() &&`.
  **L2255 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`if (Subtarget->isThumb() && Subtarget->hasVFP2Base() &&`。
- **L2256 EN**: Comment explains nearby logic, invariants, or intent: `Subtarget->hasARMOps() && !Subtarget->useSoftFloat()) {`.
  **L2256 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Subtarget->hasARMOps() && !Subtarget->useSoftFloat()) {`。
- **L2257 EN**: Blank line separating nearby declarations or logic blocks.
  **L2257 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2258 EN**: Declares TableGen def `__addsf3vfp`.
  **L2258 CN**: 声明 TableGen def `__addsf3vfp`。
- **L2259 EN**: Declares TableGen def `__subsf3vfp`.
  **L2259 CN**: 声明 TableGen def `__subsf3vfp`。
- **L2260 EN**: Declares TableGen def `__mulsf3vfp`.
  **L2260 CN**: 声明 TableGen def `__mulsf3vfp`。
- **L2261 EN**: Declares TableGen def `__divsf3vfp`.
  **L2261 CN**: 声明 TableGen def `__divsf3vfp`。
- **L2262 EN**: Blank line separating nearby declarations or logic blocks.
  **L2262 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2263 EN**: Comment explains nearby logic, invariants, or intent: `Double-precision floating-point arithmetic.`.
  **L2263 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Double-precision floating-point arithmetic.`。
- **L2264 EN**: Declares TableGen def `__adddf3vfp`.
  **L2264 CN**: 声明 TableGen def `__adddf3vfp`。
- **L2265 EN**: Declares TableGen def `__subdf3vfp`.
  **L2265 CN**: 声明 TableGen def `__subdf3vfp`。
- **L2266 EN**: Declares TableGen def `__muldf3vfp`.
  **L2266 CN**: 声明 TableGen def `__muldf3vfp`。
- **L2267 EN**: Declares TableGen def `__divdf3vfp`.
  **L2267 CN**: 声明 TableGen def `__divdf3vfp`。
- **L2268 EN**: Blank line separating nearby declarations or logic blocks.
  **L2268 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2269 EN**: Comment explains nearby logic, invariants, or intent: `Single-precision comparisons.`.
  **L2269 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Single-precision comparisons.`。
- **L2270 EN**: Blank line separating nearby declarations or logic blocks.
  **L2270 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2271 EN**: Comment records a pending task or caution: `TODO: Track setcc type`.
  **L2271 CN**: 注释记录了待办事项或注意点：`TODO: Track setcc type`。
- **L2272 EN**: Declares TableGen def `__eqsf2vfp`.
  **L2272 CN**: 声明 TableGen def `__eqsf2vfp`。

### Lines 2273-2304

````tablegen
def __nesf2vfp : RuntimeLibcallImpl<UNE_F32>; // CmpInst::ICMP_NE
def __ltsf2vfp : RuntimeLibcallImpl<OLT_F32>; // CmpInst::ICMP_NE
def __lesf2vfp : RuntimeLibcallImpl<OLE_F32>; // CmpInst::ICMP_NE
def __gesf2vfp : RuntimeLibcallImpl<OGE_F32>; // CmpInst::ICMP_NE
def __gtsf2vfp : RuntimeLibcallImpl<OGT_F32>; // CmpInst::ICMP_NE
def __unordsf2vfp : RuntimeLibcallImpl<UO_F32>; // CmpInst::ICMP_NE

// Double-precision comparisons.
def __eqdf2vfp : RuntimeLibcallImpl<OEQ_F64>; // CmpInst::ICMP_NE
def __nedf2vfp : RuntimeLibcallImpl<UNE_F64>; // CmpInst::ICMP_NE
def __ltdf2vfp : RuntimeLibcallImpl<OLT_F64>; // CmpInst::ICMP_NE
def __ledf2vfp : RuntimeLibcallImpl<OLE_F64>; // CmpInst::ICMP_NE
def __gedf2vfp : RuntimeLibcallImpl<OGE_F64>; // CmpInst::ICMP_NE
def __gtdf2vfp : RuntimeLibcallImpl<OGT_F64>; // CmpInst::ICMP_NE
def __unorddf2vfp : RuntimeLibcallImpl<UO_F64>; // CmpInst::ICMP_NE

// Floating-point to integer conversions.
// i64 conversions are done via library routines even when generating VFP
// instructions, so use the same ones.
def __fixdfsivfp : RuntimeLibcallImpl<FPTOSINT_F64_I32>;
def __fixunsdfsivfp : RuntimeLibcallImpl<FPTOUINT_F64_I32>;
def __fixsfsivfp : RuntimeLibcallImpl<FPTOSINT_F32_I32>;
def __fixunssfsivfp : RuntimeLibcallImpl<FPTOUINT_F32_I32>;

// Conversions between floating types.
def __truncdfsf2vfp : RuntimeLibcallImpl<FPROUND_F64_F32>;
def __extendsfdf2vfp : RuntimeLibcallImpl<FPEXT_F32_F64>;

// Integer to floating-point conversions.
// i64 conversions are done via library routines even when generating VFP
// instructions, so use the same ones.
// FIXME: There appears to be some naming inconsistency in ARM libgcc:
````
- **L2273 EN**: Declares TableGen def `__nesf2vfp`.
  **L2273 CN**: 声明 TableGen def `__nesf2vfp`。
- **L2274 EN**: Declares TableGen def `__ltsf2vfp`.
  **L2274 CN**: 声明 TableGen def `__ltsf2vfp`。
- **L2275 EN**: Declares TableGen def `__lesf2vfp`.
  **L2275 CN**: 声明 TableGen def `__lesf2vfp`。
- **L2276 EN**: Declares TableGen def `__gesf2vfp`.
  **L2276 CN**: 声明 TableGen def `__gesf2vfp`。
- **L2277 EN**: Declares TableGen def `__gtsf2vfp`.
  **L2277 CN**: 声明 TableGen def `__gtsf2vfp`。
- **L2278 EN**: Declares TableGen def `__unordsf2vfp`.
  **L2278 CN**: 声明 TableGen def `__unordsf2vfp`。
- **L2279 EN**: Blank line separating nearby declarations or logic blocks.
  **L2279 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2280 EN**: Comment explains nearby logic, invariants, or intent: `Double-precision comparisons.`.
  **L2280 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Double-precision comparisons.`。
- **L2281 EN**: Declares TableGen def `__eqdf2vfp`.
  **L2281 CN**: 声明 TableGen def `__eqdf2vfp`。
- **L2282 EN**: Declares TableGen def `__nedf2vfp`.
  **L2282 CN**: 声明 TableGen def `__nedf2vfp`。
- **L2283 EN**: Declares TableGen def `__ltdf2vfp`.
  **L2283 CN**: 声明 TableGen def `__ltdf2vfp`。
- **L2284 EN**: Declares TableGen def `__ledf2vfp`.
  **L2284 CN**: 声明 TableGen def `__ledf2vfp`。
- **L2285 EN**: Declares TableGen def `__gedf2vfp`.
  **L2285 CN**: 声明 TableGen def `__gedf2vfp`。
- **L2286 EN**: Declares TableGen def `__gtdf2vfp`.
  **L2286 CN**: 声明 TableGen def `__gtdf2vfp`。
- **L2287 EN**: Declares TableGen def `__unorddf2vfp`.
  **L2287 CN**: 声明 TableGen def `__unorddf2vfp`。
- **L2288 EN**: Blank line separating nearby declarations or logic blocks.
  **L2288 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2289 EN**: Comment explains nearby logic, invariants, or intent: `Floating-point to integer conversions.`.
  **L2289 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Floating-point to integer conversions.`。
- **L2290 EN**: Comment explains nearby logic, invariants, or intent: `i64 conversions are done via library routines even when generating VFP`.
  **L2290 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`i64 conversions are done via library routines even when generating VFP`。
- **L2291 EN**: Comment explains nearby logic, invariants, or intent: `instructions, so use the same ones.`.
  **L2291 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`instructions, so use the same ones.`。
- **L2292 EN**: Declares TableGen def `__fixdfsivfp`.
  **L2292 CN**: 声明 TableGen def `__fixdfsivfp`。
- **L2293 EN**: Declares TableGen def `__fixunsdfsivfp`.
  **L2293 CN**: 声明 TableGen def `__fixunsdfsivfp`。
- **L2294 EN**: Declares TableGen def `__fixsfsivfp`.
  **L2294 CN**: 声明 TableGen def `__fixsfsivfp`。
- **L2295 EN**: Declares TableGen def `__fixunssfsivfp`.
  **L2295 CN**: 声明 TableGen def `__fixunssfsivfp`。
- **L2296 EN**: Blank line separating nearby declarations or logic blocks.
  **L2296 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2297 EN**: Comment explains nearby logic, invariants, or intent: `Conversions between floating types.`.
  **L2297 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Conversions between floating types.`。
- **L2298 EN**: Declares TableGen def `__truncdfsf2vfp`.
  **L2298 CN**: 声明 TableGen def `__truncdfsf2vfp`。
- **L2299 EN**: Declares TableGen def `__extendsfdf2vfp`.
  **L2299 CN**: 声明 TableGen def `__extendsfdf2vfp`。
- **L2300 EN**: Blank line separating nearby declarations or logic blocks.
  **L2300 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2301 EN**: Comment explains nearby logic, invariants, or intent: `Integer to floating-point conversions.`.
  **L2301 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Integer to floating-point conversions.`。
- **L2302 EN**: Comment explains nearby logic, invariants, or intent: `i64 conversions are done via library routines even when generating VFP`.
  **L2302 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`i64 conversions are done via library routines even when generating VFP`。
- **L2303 EN**: Comment explains nearby logic, invariants, or intent: `instructions, so use the same ones.`.
  **L2303 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`instructions, so use the same ones.`。
- **L2304 EN**: Comment records a pending task or caution: `FIXME: There appears to be some naming inconsistency in ARM libgcc:`.
  **L2304 CN**: 注释记录了待办事项或注意点：`FIXME: There appears to be some naming inconsistency in ARM libgcc:`。

### Lines 2305-2336

````tablegen
// e.g., __floatunsidf vs. __floatunssidfvfp.
def __floatsidfvfp : RuntimeLibcallImpl<SINTTOFP_I32_F64>;
def __floatunssidfvfp : RuntimeLibcallImpl<UINTTOFP_I32_F64>;
def __floatsisfvfp : RuntimeLibcallImpl<SINTTOFP_I32_F32>;
def __floatunssisfvfp : RuntimeLibcallImpl<UINTTOFP_I32_F32>;

//  // RTLIB
//  if (isAAPCS_ABI() &&
//      (isTargetAEABI() || isTargetGNUAEABI() ||
//       isTargetMuslAEABI() || isTargetAndroid())) {

// TODO: Set CallingConv = ARM_AAPCS

// Double-precision floating-point arithmetic helper functions
// RTABI chapter 4.1.2, Table 2
def __aeabi_dadd : RuntimeLibcallImpl<ADD_F64>; // CallingConv::ARM_AAPCS
def __aeabi_ddiv : RuntimeLibcallImpl<DIV_F64>; // CallingConv::ARM_AAPCS
def __aeabi_dmul : RuntimeLibcallImpl<MUL_F64>; // CallingConv::ARM_AAPCS
def __aeabi_dsub : RuntimeLibcallImpl<SUB_F64>; // CallingConv::ARM_AAPCS

// Unaligned memory access helper functions
// RTABI chapter 5.3.3
def __aeabi_uread4  : RuntimeLibcallImpl<AEABI_UREAD4>;  // CallingConv::ARM_AAPCS
def __aeabi_uread8  : RuntimeLibcallImpl<AEABI_UREAD8>;  // CallingConv::ARM_AAPCS
def __aeabi_uwrite4 : RuntimeLibcallImpl<AEABI_UWRITE4>; // CallingConv::ARM_AAPCS
def __aeabi_uwrite8 : RuntimeLibcallImpl<AEABI_UWRITE8>; // CallingConv::ARM_AAPCS

defvar AEABIOverrides = [
  __eqsf2, __eqdf2,
  __nesf2, __nedf2,
  __ltsf2, __ltdf2,
  __lesf2, __ledf2,
````
- **L2305 EN**: Comment explains nearby logic, invariants, or intent: `e.g., __floatunsidf vs. __floatunssidfvfp.`.
  **L2305 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`e.g., __floatunsidf vs. __floatunssidfvfp.`。
- **L2306 EN**: Declares TableGen def `__floatsidfvfp`.
  **L2306 CN**: 声明 TableGen def `__floatsidfvfp`。
- **L2307 EN**: Declares TableGen def `__floatunssidfvfp`.
  **L2307 CN**: 声明 TableGen def `__floatunssidfvfp`。
- **L2308 EN**: Declares TableGen def `__floatsisfvfp`.
  **L2308 CN**: 声明 TableGen def `__floatsisfvfp`。
- **L2309 EN**: Declares TableGen def `__floatunssisfvfp`.
  **L2309 CN**: 声明 TableGen def `__floatunssisfvfp`。
- **L2310 EN**: Blank line separating nearby declarations or logic blocks.
  **L2310 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2311 EN**: Comment explains nearby logic, invariants, or intent: `// RTLIB`.
  **L2311 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`// RTLIB`。
- **L2312 EN**: Comment explains nearby logic, invariants, or intent: `if (isAAPCS_ABI() &&`.
  **L2312 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`if (isAAPCS_ABI() &&`。
- **L2313 EN**: Comment explains nearby logic, invariants, or intent: `(isTargetAEABI() || isTargetGNUAEABI() ||`.
  **L2313 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`(isTargetAEABI() || isTargetGNUAEABI() ||`。
- **L2314 EN**: Comment explains nearby logic, invariants, or intent: `isTargetMuslAEABI() || isTargetAndroid())) {`.
  **L2314 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`isTargetMuslAEABI() || isTargetAndroid())) {`。
- **L2315 EN**: Blank line separating nearby declarations or logic blocks.
  **L2315 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2316 EN**: Comment records a pending task or caution: `TODO: Set CallingConv = ARM_AAPCS`.
  **L2316 CN**: 注释记录了待办事项或注意点：`TODO: Set CallingConv = ARM_AAPCS`。
- **L2317 EN**: Blank line separating nearby declarations or logic blocks.
  **L2317 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2318 EN**: Comment explains nearby logic, invariants, or intent: `Double-precision floating-point arithmetic helper functions`.
  **L2318 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Double-precision floating-point arithmetic helper functions`。
- **L2319 EN**: Comment explains nearby logic, invariants, or intent: `RTABI chapter 4.1.2, Table 2`.
  **L2319 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`RTABI chapter 4.1.2, Table 2`。
- **L2320 EN**: Declares TableGen def `__aeabi_dadd`.
  **L2320 CN**: 声明 TableGen def `__aeabi_dadd`。
- **L2321 EN**: Declares TableGen def `__aeabi_ddiv`.
  **L2321 CN**: 声明 TableGen def `__aeabi_ddiv`。
- **L2322 EN**: Declares TableGen def `__aeabi_dmul`.
  **L2322 CN**: 声明 TableGen def `__aeabi_dmul`。
- **L2323 EN**: Declares TableGen def `__aeabi_dsub`.
  **L2323 CN**: 声明 TableGen def `__aeabi_dsub`。
- **L2324 EN**: Blank line separating nearby declarations or logic blocks.
  **L2324 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2325 EN**: Comment explains nearby logic, invariants, or intent: `Unaligned memory access helper functions`.
  **L2325 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Unaligned memory access helper functions`。
- **L2326 EN**: Comment explains nearby logic, invariants, or intent: `RTABI chapter 5.3.3`.
  **L2326 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`RTABI chapter 5.3.3`。
- **L2327 EN**: Declares TableGen def `__aeabi_uread4`.
  **L2327 CN**: 声明 TableGen def `__aeabi_uread4`。
- **L2328 EN**: Declares TableGen def `__aeabi_uread8`.
  **L2328 CN**: 声明 TableGen def `__aeabi_uread8`。
- **L2329 EN**: Declares TableGen def `__aeabi_uwrite4`.
  **L2329 CN**: 声明 TableGen def `__aeabi_uwrite4`。
- **L2330 EN**: Declares TableGen def `__aeabi_uwrite8`.
  **L2330 CN**: 声明 TableGen def `__aeabi_uwrite8`。
- **L2331 EN**: Blank line separating nearby declarations or logic blocks.
  **L2331 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2332 EN**: Continues the surrounding expression or declaration: `defvar AEABIOverrides = [`.
  **L2332 CN**: 继续构造周围的表达式或声明：`defvar AEABIOverrides = [`。
- **L2333 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__eqsf2, __eqdf2,`.
  **L2333 CN**: 继续一个多行参数列表、初始化器或聚合项：`__eqsf2, __eqdf2,`。
- **L2334 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__nesf2, __nedf2,`.
  **L2334 CN**: 继续一个多行参数列表、初始化器或聚合项：`__nesf2, __nedf2,`。
- **L2335 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__ltsf2, __ltdf2,`.
  **L2335 CN**: 继续一个多行参数列表、初始化器或聚合项：`__ltsf2, __ltdf2,`。
- **L2336 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__lesf2, __ledf2,`.
  **L2336 CN**: 继续一个多行参数列表、初始化器或聚合项：`__lesf2, __ledf2,`。

### Lines 2337-2368

````tablegen
  __gesf2, __gedf2,
  __gtsf2, __gtdf2,
  __unordsf2, __unorddf2,

  __addsf3, __adddf3,
  __divsf3, __divdf3,
  __mulsf3, __muldf3,
  __subsf3, __subdf3,

  __fixdfsi, __fixunsdfsi,
  __fixdfdi, __fixunsdfdi,
  __fixsfsi, __fixunssfsi,
  __fixsfdi, __fixunssfdi,

  __floatsidf, __floatunsidf,
  __floatdidf, __floatundidf,
  __floatsisf, __floatunsisf,
  __floatdisf, __floatundisf,

  __muldi3, __ashldi3,
  __lshrdi3, __ashrdi3,

  __divsi3, __udivsi3

  // Half conversion cases are a mess and handled separately.
  //  __truncdfsf2, __truncdfhf2,
  //  __extendsfdf2,
  //  __truncsfhf2, __extendhfsf2
];

// Double-precision floating-point comparison helper functions
// RTABI chapter 4.1.2, Table 3
````
- **L2337 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__gesf2, __gedf2,`.
  **L2337 CN**: 继续一个多行参数列表、初始化器或聚合项：`__gesf2, __gedf2,`。
- **L2338 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__gtsf2, __gtdf2,`.
  **L2338 CN**: 继续一个多行参数列表、初始化器或聚合项：`__gtsf2, __gtdf2,`。
- **L2339 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__unordsf2, __unorddf2,`.
  **L2339 CN**: 继续一个多行参数列表、初始化器或聚合项：`__unordsf2, __unorddf2,`。
- **L2340 EN**: Blank line separating nearby declarations or logic blocks.
  **L2340 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2341 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__addsf3, __adddf3,`.
  **L2341 CN**: 继续一个多行参数列表、初始化器或聚合项：`__addsf3, __adddf3,`。
- **L2342 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__divsf3, __divdf3,`.
  **L2342 CN**: 继续一个多行参数列表、初始化器或聚合项：`__divsf3, __divdf3,`。
- **L2343 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__mulsf3, __muldf3,`.
  **L2343 CN**: 继续一个多行参数列表、初始化器或聚合项：`__mulsf3, __muldf3,`。
- **L2344 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__subsf3, __subdf3,`.
  **L2344 CN**: 继续一个多行参数列表、初始化器或聚合项：`__subsf3, __subdf3,`。
- **L2345 EN**: Blank line separating nearby declarations or logic blocks.
  **L2345 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2346 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__fixdfsi, __fixunsdfsi,`.
  **L2346 CN**: 继续一个多行参数列表、初始化器或聚合项：`__fixdfsi, __fixunsdfsi,`。
- **L2347 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__fixdfdi, __fixunsdfdi,`.
  **L2347 CN**: 继续一个多行参数列表、初始化器或聚合项：`__fixdfdi, __fixunsdfdi,`。
- **L2348 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__fixsfsi, __fixunssfsi,`.
  **L2348 CN**: 继续一个多行参数列表、初始化器或聚合项：`__fixsfsi, __fixunssfsi,`。
- **L2349 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__fixsfdi, __fixunssfdi,`.
  **L2349 CN**: 继续一个多行参数列表、初始化器或聚合项：`__fixsfdi, __fixunssfdi,`。
- **L2350 EN**: Blank line separating nearby declarations or logic blocks.
  **L2350 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2351 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__floatsidf, __floatunsidf,`.
  **L2351 CN**: 继续一个多行参数列表、初始化器或聚合项：`__floatsidf, __floatunsidf,`。
- **L2352 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__floatdidf, __floatundidf,`.
  **L2352 CN**: 继续一个多行参数列表、初始化器或聚合项：`__floatdidf, __floatundidf,`。
- **L2353 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__floatsisf, __floatunsisf,`.
  **L2353 CN**: 继续一个多行参数列表、初始化器或聚合项：`__floatsisf, __floatunsisf,`。
- **L2354 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__floatdisf, __floatundisf,`.
  **L2354 CN**: 继续一个多行参数列表、初始化器或聚合项：`__floatdisf, __floatundisf,`。
- **L2355 EN**: Blank line separating nearby declarations or logic blocks.
  **L2355 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2356 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__muldi3, __ashldi3,`.
  **L2356 CN**: 继续一个多行参数列表、初始化器或聚合项：`__muldi3, __ashldi3,`。
- **L2357 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__lshrdi3, __ashrdi3,`.
  **L2357 CN**: 继续一个多行参数列表、初始化器或聚合项：`__lshrdi3, __ashrdi3,`。
- **L2358 EN**: Blank line separating nearby declarations or logic blocks.
  **L2358 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2359 EN**: Continues the surrounding expression or declaration: `__divsi3, __udivsi3`.
  **L2359 CN**: 继续构造周围的表达式或声明：`__divsi3, __udivsi3`。
- **L2360 EN**: Blank line separating nearby declarations or logic blocks.
  **L2360 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2361 EN**: Comment explains nearby logic, invariants, or intent: `Half conversion cases are a mess and handled separately.`.
  **L2361 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Half conversion cases are a mess and handled separately.`。
- **L2362 EN**: Comment explains nearby logic, invariants, or intent: `__truncdfsf2, __truncdfhf2,`.
  **L2362 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`__truncdfsf2, __truncdfhf2,`。
- **L2363 EN**: Comment explains nearby logic, invariants, or intent: `__extendsfdf2,`.
  **L2363 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`__extendsfdf2,`。
- **L2364 EN**: Comment explains nearby logic, invariants, or intent: `__truncsfhf2, __extendhfsf2`.
  **L2364 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`__truncsfhf2, __extendhfsf2`。
- **L2365 EN**: Executes a standalone statement or declaration: `];`.
  **L2365 CN**: 执行一条独立语句或声明：`];`。
- **L2366 EN**: Blank line separating nearby declarations or logic blocks.
  **L2366 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2367 EN**: Comment explains nearby logic, invariants, or intent: `Double-precision floating-point comparison helper functions`.
  **L2367 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Double-precision floating-point comparison helper functions`。
- **L2368 EN**: Comment explains nearby logic, invariants, or intent: `RTABI chapter 4.1.2, Table 3`.
  **L2368 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`RTABI chapter 4.1.2, Table 3`。

### Lines 2369-2400

````tablegen
def __aeabi_dcmpeq__oeq : RuntimeLibcallImpl<OEQ_F64, "__aeabi_dcmpeq">; // CallingConv::ARM_AAPCS, CmpInst::ICMP_NE
def __aeabi_dcmpeq__une : RuntimeLibcallImpl<UNE_F64, "__aeabi_dcmpeq">; // CallingConv::ARM_AAPCS, CmpInst::ICMP_EQ
def __aeabi_dcmplt : RuntimeLibcallImpl<OLT_F64>; // CallingConv::ARM_AAPCS, CmpInst::ICMP_NE
def __aeabi_dcmple : RuntimeLibcallImpl<OLE_F64>; // CallingConv::ARM_AAPCS, CmpInst::ICMP_NE
def __aeabi_dcmpge : RuntimeLibcallImpl<OGE_F64>; // CallingConv::ARM_AAPCS, CmpInst::ICMP_NE
def __aeabi_dcmpgt : RuntimeLibcallImpl<OGT_F64>; // CallingConv::ARM_AAPCS, CmpInst::ICMP_NE
def __aeabi_dcmpun : RuntimeLibcallImpl<UO_F64>; // CallingConv::ARM_AAPCS

// Single-precision floating-point arithmetic helper functions
// RTABI chapter 4.1.2, Table 4
def __aeabi_fadd : RuntimeLibcallImpl<ADD_F32>; // CallingConv::ARM_AAPCS
def __aeabi_fdiv : RuntimeLibcallImpl<DIV_F32>; // CallingConv::ARM_AAPCS
def __aeabi_fmul : RuntimeLibcallImpl<MUL_F32>; // CallingConv::ARM_AAPCS
def __aeabi_fsub : RuntimeLibcallImpl<SUB_F32>; // CallingConv::ARM_AAPCS

// Single-precision floating-point comparison helper functions
// RTABI chapter 4.1.2, Table 5
def __aeabi_fcmpeq__oeq : RuntimeLibcallImpl<OEQ_F32, "__aeabi_fcmpeq">; // CallingConv::ARM_AAPCS, CmpInst::ICMP_NE
def __aeabi_fcmpeq__une : RuntimeLibcallImpl<UNE_F32, "__aeabi_fcmpeq">; // CallingConv::ARM_AAPCS, CmpInst::ICMP_EQ
def __aeabi_fcmplt : RuntimeLibcallImpl<OLT_F32>; // CallingConv::ARM_AAPCS, CmpInst::ICMP_NE
def __aeabi_fcmple : RuntimeLibcallImpl<OLE_F32>; // CallingConv::ARM_AAPCS, CmpInst::ICMP_NE
def __aeabi_fcmpge : RuntimeLibcallImpl<OGE_F32>; // CallingConv::ARM_AAPCS, CmpInst::ICMP_NE
def __aeabi_fcmpgt : RuntimeLibcallImpl<OGT_F32>; // CallingConv::ARM_AAPCS, CmpInst::ICMP_NE
def __aeabi_fcmpun : RuntimeLibcallImpl<UO_F32>; // CallingConv::ARM_AAPCS

// Floating-point to integer conversions.
// RTABI chapter 4.1.2, Table 6
def __aeabi_d2iz : RuntimeLibcallImpl<FPTOSINT_F64_I32>; //  CallingConv::ARM_AAPCS
def __aeabi_d2uiz : RuntimeLibcallImpl<FPTOUINT_F64_I32>; // CallingConv::ARM_AAPCS
def __aeabi_d2lz : RuntimeLibcallImpl<FPTOSINT_F64_I64>; //  CallingConv::ARM_AAPCS
def __aeabi_d2ulz : RuntimeLibcallImpl<FPTOUINT_F64_I64>; // CallingConv::ARM_AAPCS
def __aeabi_f2iz : RuntimeLibcallImpl<FPTOSINT_F32_I32>; //  CallingConv::ARM_AAPCS
````
- **L2369 EN**: Declares TableGen def `__aeabi_dcmpeq__oeq`.
  **L2369 CN**: 声明 TableGen def `__aeabi_dcmpeq__oeq`。
- **L2370 EN**: Declares TableGen def `__aeabi_dcmpeq__une`.
  **L2370 CN**: 声明 TableGen def `__aeabi_dcmpeq__une`。
- **L2371 EN**: Declares TableGen def `__aeabi_dcmplt`.
  **L2371 CN**: 声明 TableGen def `__aeabi_dcmplt`。
- **L2372 EN**: Declares TableGen def `__aeabi_dcmple`.
  **L2372 CN**: 声明 TableGen def `__aeabi_dcmple`。
- **L2373 EN**: Declares TableGen def `__aeabi_dcmpge`.
  **L2373 CN**: 声明 TableGen def `__aeabi_dcmpge`。
- **L2374 EN**: Declares TableGen def `__aeabi_dcmpgt`.
  **L2374 CN**: 声明 TableGen def `__aeabi_dcmpgt`。
- **L2375 EN**: Declares TableGen def `__aeabi_dcmpun`.
  **L2375 CN**: 声明 TableGen def `__aeabi_dcmpun`。
- **L2376 EN**: Blank line separating nearby declarations or logic blocks.
  **L2376 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2377 EN**: Comment explains nearby logic, invariants, or intent: `Single-precision floating-point arithmetic helper functions`.
  **L2377 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Single-precision floating-point arithmetic helper functions`。
- **L2378 EN**: Comment explains nearby logic, invariants, or intent: `RTABI chapter 4.1.2, Table 4`.
  **L2378 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`RTABI chapter 4.1.2, Table 4`。
- **L2379 EN**: Declares TableGen def `__aeabi_fadd`.
  **L2379 CN**: 声明 TableGen def `__aeabi_fadd`。
- **L2380 EN**: Declares TableGen def `__aeabi_fdiv`.
  **L2380 CN**: 声明 TableGen def `__aeabi_fdiv`。
- **L2381 EN**: Declares TableGen def `__aeabi_fmul`.
  **L2381 CN**: 声明 TableGen def `__aeabi_fmul`。
- **L2382 EN**: Declares TableGen def `__aeabi_fsub`.
  **L2382 CN**: 声明 TableGen def `__aeabi_fsub`。
- **L2383 EN**: Blank line separating nearby declarations or logic blocks.
  **L2383 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2384 EN**: Comment explains nearby logic, invariants, or intent: `Single-precision floating-point comparison helper functions`.
  **L2384 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Single-precision floating-point comparison helper functions`。
- **L2385 EN**: Comment explains nearby logic, invariants, or intent: `RTABI chapter 4.1.2, Table 5`.
  **L2385 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`RTABI chapter 4.1.2, Table 5`。
- **L2386 EN**: Declares TableGen def `__aeabi_fcmpeq__oeq`.
  **L2386 CN**: 声明 TableGen def `__aeabi_fcmpeq__oeq`。
- **L2387 EN**: Declares TableGen def `__aeabi_fcmpeq__une`.
  **L2387 CN**: 声明 TableGen def `__aeabi_fcmpeq__une`。
- **L2388 EN**: Declares TableGen def `__aeabi_fcmplt`.
  **L2388 CN**: 声明 TableGen def `__aeabi_fcmplt`。
- **L2389 EN**: Declares TableGen def `__aeabi_fcmple`.
  **L2389 CN**: 声明 TableGen def `__aeabi_fcmple`。
- **L2390 EN**: Declares TableGen def `__aeabi_fcmpge`.
  **L2390 CN**: 声明 TableGen def `__aeabi_fcmpge`。
- **L2391 EN**: Declares TableGen def `__aeabi_fcmpgt`.
  **L2391 CN**: 声明 TableGen def `__aeabi_fcmpgt`。
- **L2392 EN**: Declares TableGen def `__aeabi_fcmpun`.
  **L2392 CN**: 声明 TableGen def `__aeabi_fcmpun`。
- **L2393 EN**: Blank line separating nearby declarations or logic blocks.
  **L2393 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2394 EN**: Comment explains nearby logic, invariants, or intent: `Floating-point to integer conversions.`.
  **L2394 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Floating-point to integer conversions.`。
- **L2395 EN**: Comment explains nearby logic, invariants, or intent: `RTABI chapter 4.1.2, Table 6`.
  **L2395 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`RTABI chapter 4.1.2, Table 6`。
- **L2396 EN**: Declares TableGen def `__aeabi_d2iz`.
  **L2396 CN**: 声明 TableGen def `__aeabi_d2iz`。
- **L2397 EN**: Declares TableGen def `__aeabi_d2uiz`.
  **L2397 CN**: 声明 TableGen def `__aeabi_d2uiz`。
- **L2398 EN**: Declares TableGen def `__aeabi_d2lz`.
  **L2398 CN**: 声明 TableGen def `__aeabi_d2lz`。
- **L2399 EN**: Declares TableGen def `__aeabi_d2ulz`.
  **L2399 CN**: 声明 TableGen def `__aeabi_d2ulz`。
- **L2400 EN**: Declares TableGen def `__aeabi_f2iz`.
  **L2400 CN**: 声明 TableGen def `__aeabi_f2iz`。

### Lines 2401-2432

````tablegen
def __aeabi_f2uiz : RuntimeLibcallImpl<FPTOUINT_F32_I32>; // CallingConv::ARM_AAPCS
def __aeabi_f2lz : RuntimeLibcallImpl<FPTOSINT_F32_I64>; //  CallingConv::ARM_AAPCS
def __aeabi_f2ulz : RuntimeLibcallImpl<FPTOUINT_F32_I64>; // CallingConv::ARM_AAPCS

// Conversions between floating types.
// RTABI chapter 4.1.2, Table 7
def __aeabi_d2f : RuntimeLibcallImpl<FPROUND_F64_F32>; // CallingConv::ARM_AAPCS
def __aeabi_d2h : RuntimeLibcallImpl<FPROUND_F64_F16>; // CallingConv::ARM_AAPCS
def __aeabi_f2d : RuntimeLibcallImpl<FPEXT_F32_F64>; // CallingConv::ARM_AAPCS

// Integer to floating-point conversions.
// RTABI chapter 4.1.2, Table 8
def __aeabi_i2d : RuntimeLibcallImpl<SINTTOFP_I32_F64>; //  CallingConv::ARM_AAPCS
def __aeabi_ui2d : RuntimeLibcallImpl<UINTTOFP_I32_F64>; // CallingConv::ARM_AAPCS
def __aeabi_l2d : RuntimeLibcallImpl<SINTTOFP_I64_F64>; //  CallingConv::ARM_AAPCS
def __aeabi_ul2d : RuntimeLibcallImpl<UINTTOFP_I64_F64>; // CallingConv::ARM_AAPCS
def __aeabi_i2f : RuntimeLibcallImpl<SINTTOFP_I32_F32>; //  CallingConv::ARM_AAPCS
def __aeabi_ui2f : RuntimeLibcallImpl<UINTTOFP_I32_F32>; // CallingConv::ARM_AAPCS
def __aeabi_l2f : RuntimeLibcallImpl<SINTTOFP_I64_F32>; //  CallingConv::ARM_AAPCS
def __aeabi_ul2f : RuntimeLibcallImpl<UINTTOFP_I64_F32>; // CallingConv::ARM_AAPCS

// Long long helper functions
// RTABI chapter 4.2, Table 9
def __aeabi_lmul : RuntimeLibcallImpl<MUL_I64>; // CallingConv::ARM_AAPCS
def __aeabi_llsl : RuntimeLibcallImpl<SHL_I64>; // CallingConv::ARM_AAPCS
def __aeabi_llsr : RuntimeLibcallImpl<SRL_I64>; // CallingConv::ARM_AAPCS
def __aeabi_lasr : RuntimeLibcallImpl<SRA_I64>; // CallingConv::ARM_AAPCS

// Integer division functions
// RTABI chapter 4.3.1
def __aeabi_idiv : RuntimeLibcallImpl<SDIV_I32>; // CallingConv::ARM_AAPCS
def __aeabi_ldivmod : RuntimeLibcallImpl<SDIVREM_I64>; //  CallingConv::ARM_AAPCS
````
- **L2401 EN**: Declares TableGen def `__aeabi_f2uiz`.
  **L2401 CN**: 声明 TableGen def `__aeabi_f2uiz`。
- **L2402 EN**: Declares TableGen def `__aeabi_f2lz`.
  **L2402 CN**: 声明 TableGen def `__aeabi_f2lz`。
- **L2403 EN**: Declares TableGen def `__aeabi_f2ulz`.
  **L2403 CN**: 声明 TableGen def `__aeabi_f2ulz`。
- **L2404 EN**: Blank line separating nearby declarations or logic blocks.
  **L2404 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2405 EN**: Comment explains nearby logic, invariants, or intent: `Conversions between floating types.`.
  **L2405 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Conversions between floating types.`。
- **L2406 EN**: Comment explains nearby logic, invariants, or intent: `RTABI chapter 4.1.2, Table 7`.
  **L2406 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`RTABI chapter 4.1.2, Table 7`。
- **L2407 EN**: Declares TableGen def `__aeabi_d2f`.
  **L2407 CN**: 声明 TableGen def `__aeabi_d2f`。
- **L2408 EN**: Declares TableGen def `__aeabi_d2h`.
  **L2408 CN**: 声明 TableGen def `__aeabi_d2h`。
- **L2409 EN**: Declares TableGen def `__aeabi_f2d`.
  **L2409 CN**: 声明 TableGen def `__aeabi_f2d`。
- **L2410 EN**: Blank line separating nearby declarations or logic blocks.
  **L2410 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2411 EN**: Comment explains nearby logic, invariants, or intent: `Integer to floating-point conversions.`.
  **L2411 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Integer to floating-point conversions.`。
- **L2412 EN**: Comment explains nearby logic, invariants, or intent: `RTABI chapter 4.1.2, Table 8`.
  **L2412 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`RTABI chapter 4.1.2, Table 8`。
- **L2413 EN**: Declares TableGen def `__aeabi_i2d`.
  **L2413 CN**: 声明 TableGen def `__aeabi_i2d`。
- **L2414 EN**: Declares TableGen def `__aeabi_ui2d`.
  **L2414 CN**: 声明 TableGen def `__aeabi_ui2d`。
- **L2415 EN**: Declares TableGen def `__aeabi_l2d`.
  **L2415 CN**: 声明 TableGen def `__aeabi_l2d`。
- **L2416 EN**: Declares TableGen def `__aeabi_ul2d`.
  **L2416 CN**: 声明 TableGen def `__aeabi_ul2d`。
- **L2417 EN**: Declares TableGen def `__aeabi_i2f`.
  **L2417 CN**: 声明 TableGen def `__aeabi_i2f`。
- **L2418 EN**: Declares TableGen def `__aeabi_ui2f`.
  **L2418 CN**: 声明 TableGen def `__aeabi_ui2f`。
- **L2419 EN**: Declares TableGen def `__aeabi_l2f`.
  **L2419 CN**: 声明 TableGen def `__aeabi_l2f`。
- **L2420 EN**: Declares TableGen def `__aeabi_ul2f`.
  **L2420 CN**: 声明 TableGen def `__aeabi_ul2f`。
- **L2421 EN**: Blank line separating nearby declarations or logic blocks.
  **L2421 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2422 EN**: Comment explains nearby logic, invariants, or intent: `Long long helper functions`.
  **L2422 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Long long helper functions`。
- **L2423 EN**: Comment explains nearby logic, invariants, or intent: `RTABI chapter 4.2, Table 9`.
  **L2423 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`RTABI chapter 4.2, Table 9`。
- **L2424 EN**: Declares TableGen def `__aeabi_lmul`.
  **L2424 CN**: 声明 TableGen def `__aeabi_lmul`。
- **L2425 EN**: Declares TableGen def `__aeabi_llsl`.
  **L2425 CN**: 声明 TableGen def `__aeabi_llsl`。
- **L2426 EN**: Declares TableGen def `__aeabi_llsr`.
  **L2426 CN**: 声明 TableGen def `__aeabi_llsr`。
- **L2427 EN**: Declares TableGen def `__aeabi_lasr`.
  **L2427 CN**: 声明 TableGen def `__aeabi_lasr`。
- **L2428 EN**: Blank line separating nearby declarations or logic blocks.
  **L2428 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2429 EN**: Comment explains nearby logic, invariants, or intent: `Integer division functions`.
  **L2429 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Integer division functions`。
- **L2430 EN**: Comment explains nearby logic, invariants, or intent: `RTABI chapter 4.3.1`.
  **L2430 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`RTABI chapter 4.3.1`。
- **L2431 EN**: Declares TableGen def `__aeabi_idiv`.
  **L2431 CN**: 声明 TableGen def `__aeabi_idiv`。
- **L2432 EN**: Declares TableGen def `__aeabi_ldivmod`.
  **L2432 CN**: 声明 TableGen def `__aeabi_ldivmod`。

### Lines 2433-2464

````tablegen
def __aeabi_uidiv : RuntimeLibcallImpl<UDIV_I32>; // CallingConv::ARM_AAPCS
def __aeabi_uldivmod : RuntimeLibcallImpl<UDIVREM_I64>; // CallingConv::ARM_AAPCS

def __aeabi_idivmod : RuntimeLibcallImpl<SDIVREM_I32>; //  CallingConv::ARM_AAPCS
def __aeabi_uidivmod : RuntimeLibcallImpl<UDIVREM_I32>; //  CallingConv::ARM_AAPCS

// EABI dependent RTLIB

// Memory operations
// RTABI chapter 4.3.4
def __aeabi_memcpy : RuntimeLibcallImpl<MEMCPY>; // CallingConv::ARM_AAPCS
def __aeabi_memcpy4 : RuntimeLibcallImpl<AEABI_MEMCPY4>;
def __aeabi_memcpy8 : RuntimeLibcallImpl<AEABI_MEMCPY8>;

def __aeabi_memmove : RuntimeLibcallImpl<MEMMOVE>; // CallingConv::ARM_AAPCS
def __aeabi_memmove4 : RuntimeLibcallImpl<AEABI_MEMMOVE4>;
def __aeabi_memmove8 : RuntimeLibcallImpl<AEABI_MEMMOVE8>;

def __aeabi_memset : RuntimeLibcallImpl<MEMSET>; // CallingConv::ARM_AAPCS
def __aeabi_memset4 : RuntimeLibcallImpl<AEABI_MEMSET4>;
def __aeabi_memset8 : RuntimeLibcallImpl<AEABI_MEMSET8>;

def __aeabi_memclr : RuntimeLibcallImpl<AEABI_MEMCLR>;
def __aeabi_memclr4 : RuntimeLibcallImpl<AEABI_MEMCLR4>;
def __aeabi_memclr8 : RuntimeLibcallImpl<AEABI_MEMCLR8>;

// isTargetWindows()
defset list<RuntimeLibcallImpl> WindowsFPIntCastLibcalls = {
def __stoi64 : RuntimeLibcallImpl<FPTOSINT_F32_I64>; // CallingConv::ARM_AAPCS_VFP
def __dtoi64 : RuntimeLibcallImpl<FPTOSINT_F64_I64>; // CallingConv::ARM_AAPCS_VFP
def __stou64 : RuntimeLibcallImpl<FPTOUINT_F32_I64>; // CallingConv::ARM_AAPCS_VFP
def __dtou64 : RuntimeLibcallImpl<FPTOUINT_F64_I64>; // CallingConv::ARM_AAPCS_VFP
````
- **L2433 EN**: Declares TableGen def `__aeabi_uidiv`.
  **L2433 CN**: 声明 TableGen def `__aeabi_uidiv`。
- **L2434 EN**: Declares TableGen def `__aeabi_uldivmod`.
  **L2434 CN**: 声明 TableGen def `__aeabi_uldivmod`。
- **L2435 EN**: Blank line separating nearby declarations or logic blocks.
  **L2435 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2436 EN**: Declares TableGen def `__aeabi_idivmod`.
  **L2436 CN**: 声明 TableGen def `__aeabi_idivmod`。
- **L2437 EN**: Declares TableGen def `__aeabi_uidivmod`.
  **L2437 CN**: 声明 TableGen def `__aeabi_uidivmod`。
- **L2438 EN**: Blank line separating nearby declarations or logic blocks.
  **L2438 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2439 EN**: Comment explains nearby logic, invariants, or intent: `EABI dependent RTLIB`.
  **L2439 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`EABI dependent RTLIB`。
- **L2440 EN**: Blank line separating nearby declarations or logic blocks.
  **L2440 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2441 EN**: Comment explains nearby logic, invariants, or intent: `Memory operations`.
  **L2441 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Memory operations`。
- **L2442 EN**: Comment explains nearby logic, invariants, or intent: `RTABI chapter 4.3.4`.
  **L2442 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`RTABI chapter 4.3.4`。
- **L2443 EN**: Declares TableGen def `__aeabi_memcpy`.
  **L2443 CN**: 声明 TableGen def `__aeabi_memcpy`。
- **L2444 EN**: Declares TableGen def `__aeabi_memcpy4`.
  **L2444 CN**: 声明 TableGen def `__aeabi_memcpy4`。
- **L2445 EN**: Declares TableGen def `__aeabi_memcpy8`.
  **L2445 CN**: 声明 TableGen def `__aeabi_memcpy8`。
- **L2446 EN**: Blank line separating nearby declarations or logic blocks.
  **L2446 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2447 EN**: Declares TableGen def `__aeabi_memmove`.
  **L2447 CN**: 声明 TableGen def `__aeabi_memmove`。
- **L2448 EN**: Declares TableGen def `__aeabi_memmove4`.
  **L2448 CN**: 声明 TableGen def `__aeabi_memmove4`。
- **L2449 EN**: Declares TableGen def `__aeabi_memmove8`.
  **L2449 CN**: 声明 TableGen def `__aeabi_memmove8`。
- **L2450 EN**: Blank line separating nearby declarations or logic blocks.
  **L2450 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2451 EN**: Declares TableGen def `__aeabi_memset`.
  **L2451 CN**: 声明 TableGen def `__aeabi_memset`。
- **L2452 EN**: Declares TableGen def `__aeabi_memset4`.
  **L2452 CN**: 声明 TableGen def `__aeabi_memset4`。
- **L2453 EN**: Declares TableGen def `__aeabi_memset8`.
  **L2453 CN**: 声明 TableGen def `__aeabi_memset8`。
- **L2454 EN**: Blank line separating nearby declarations or logic blocks.
  **L2454 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2455 EN**: Declares TableGen def `__aeabi_memclr`.
  **L2455 CN**: 声明 TableGen def `__aeabi_memclr`。
- **L2456 EN**: Declares TableGen def `__aeabi_memclr4`.
  **L2456 CN**: 声明 TableGen def `__aeabi_memclr4`。
- **L2457 EN**: Declares TableGen def `__aeabi_memclr8`.
  **L2457 CN**: 声明 TableGen def `__aeabi_memclr8`。
- **L2458 EN**: Blank line separating nearby declarations or logic blocks.
  **L2458 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2459 EN**: Comment explains nearby logic, invariants, or intent: `isTargetWindows()`.
  **L2459 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`isTargetWindows()`。
- **L2460 EN**: Continues the surrounding expression or declaration: `defset list<RuntimeLibcallImpl> WindowsFPIntCastLibcalls = {`.
  **L2460 CN**: 继续构造周围的表达式或声明：`defset list<RuntimeLibcallImpl> WindowsFPIntCastLibcalls = {`。
- **L2461 EN**: Declares TableGen def `__stoi64`.
  **L2461 CN**: 声明 TableGen def `__stoi64`。
- **L2462 EN**: Declares TableGen def `__dtoi64`.
  **L2462 CN**: 声明 TableGen def `__dtoi64`。
- **L2463 EN**: Declares TableGen def `__stou64`.
  **L2463 CN**: 声明 TableGen def `__stou64`。
- **L2464 EN**: Declares TableGen def `__dtou64`.
  **L2464 CN**: 声明 TableGen def `__dtou64`。

### Lines 2465-2496

````tablegen
def __i64tos : RuntimeLibcallImpl<SINTTOFP_I64_F32>; // CallingConv::ARM_AAPCS_VFP
def __i64tod : RuntimeLibcallImpl<SINTTOFP_I64_F64>; // CallingConv::ARM_AAPCS_VFP
def __u64tos : RuntimeLibcallImpl<UINTTOFP_I64_F32>; // CallingConv::ARM_AAPCS_VFP
def __u64tod : RuntimeLibcallImpl<UINTTOFP_I64_F64>; // CallingConv::ARM_AAPCS_VFP
}

def __rt_sdiv : RuntimeLibcallImpl<SDIVREM_I32>; // CallingConv::ARM_AAPCS
def __rt_sdiv64 : RuntimeLibcallImpl<SDIVREM_I64>; // CallingConv::ARM_AAPCS
def __rt_udiv : RuntimeLibcallImpl<UDIVREM_I32>; // CallingConv::ARM_AAPCS
def __rt_udiv64 : RuntimeLibcallImpl<UDIVREM_I64>; // CallingConv::ARM_AAPCS

// Use divmod compiler-rt calls for iOS 5.0 and later.
//  isTargetMachO() &&
// !(isTargetIOS() && isOSVersionLT(5, 0))
def __divmodsi4 : RuntimeLibcallImpl<SDIVREM_I32>;
def __udivmodsi4 : RuntimeLibcallImpl<UDIVREM_I32>;

// FIXME: Change calling convention of FPROUND_F32_F16,
// RTLIB::FPROUND_F64_F16, FPEXT_F16_F32 for !Subtarget->isTargetWatchABI())

// In EABI, these functions have an __aeabi_ prefix, but in GNUEABI they have
// a __gnu_ prefix (which is the default).
// isTargetAEABI()
def __aeabi_f2h : RuntimeLibcallImpl<FPROUND_F32_F16>; // CallingConv::ARM_AAPCS
def __aeabi_h2f : RuntimeLibcallImpl<FPEXT_F16_F32>; // CallingConv::ARM_AAPCS

// !isTargetMachO()
def __gnu_f2h_ieee : RuntimeLibcallImpl<FPROUND_F32_F16>;
def __gnu_h2f_ieee : RuntimeLibcallImpl<FPEXT_F16_F32>;

// The half <-> float conversion functions are always soft-float on
// non-watchos platforms, but are needed for some targets which use a
````
- **L2465 EN**: Declares TableGen def `__i64tos`.
  **L2465 CN**: 声明 TableGen def `__i64tos`。
- **L2466 EN**: Declares TableGen def `__i64tod`.
  **L2466 CN**: 声明 TableGen def `__i64tod`。
- **L2467 EN**: Declares TableGen def `__u64tos`.
  **L2467 CN**: 声明 TableGen def `__u64tos`。
- **L2468 EN**: Declares TableGen def `__u64tod`.
  **L2468 CN**: 声明 TableGen def `__u64tod`。
- **L2469 EN**: Closes the current lexical scope or compound statement.
  **L2469 CN**: 结束当前词法作用域或复合语句块。
- **L2470 EN**: Blank line separating nearby declarations or logic blocks.
  **L2470 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2471 EN**: Declares TableGen def `__rt_sdiv`.
  **L2471 CN**: 声明 TableGen def `__rt_sdiv`。
- **L2472 EN**: Declares TableGen def `__rt_sdiv64`.
  **L2472 CN**: 声明 TableGen def `__rt_sdiv64`。
- **L2473 EN**: Declares TableGen def `__rt_udiv`.
  **L2473 CN**: 声明 TableGen def `__rt_udiv`。
- **L2474 EN**: Declares TableGen def `__rt_udiv64`.
  **L2474 CN**: 声明 TableGen def `__rt_udiv64`。
- **L2475 EN**: Blank line separating nearby declarations or logic blocks.
  **L2475 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2476 EN**: Comment explains nearby logic, invariants, or intent: `Use divmod compiler-rt calls for iOS 5.0 and later.`.
  **L2476 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Use divmod compiler-rt calls for iOS 5.0 and later.`。
- **L2477 EN**: Comment explains nearby logic, invariants, or intent: `isTargetMachO() &&`.
  **L2477 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`isTargetMachO() &&`。
- **L2478 EN**: Comment explains nearby logic, invariants, or intent: `(isTargetIOS() && isOSVersionLT(5, 0))`.
  **L2478 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`(isTargetIOS() && isOSVersionLT(5, 0))`。
- **L2479 EN**: Declares TableGen def `__divmodsi4`.
  **L2479 CN**: 声明 TableGen def `__divmodsi4`。
- **L2480 EN**: Declares TableGen def `__udivmodsi4`.
  **L2480 CN**: 声明 TableGen def `__udivmodsi4`。
- **L2481 EN**: Blank line separating nearby declarations or logic blocks.
  **L2481 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2482 EN**: Comment records a pending task or caution: `FIXME: Change calling convention of FPROUND_F32_F16,`.
  **L2482 CN**: 注释记录了待办事项或注意点：`FIXME: Change calling convention of FPROUND_F32_F16,`。
- **L2483 EN**: Comment explains nearby logic, invariants, or intent: `RTLIB::FPROUND_F64_F16, FPEXT_F16_F32 for !Subtarget->isTargetWatchABI())`.
  **L2483 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`RTLIB::FPROUND_F64_F16, FPEXT_F16_F32 for !Subtarget->isTargetWatchABI())`。
- **L2484 EN**: Blank line separating nearby declarations or logic blocks.
  **L2484 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2485 EN**: Comment explains nearby logic, invariants, or intent: `In EABI, these functions have an __aeabi_ prefix, but in GNUEABI they have`.
  **L2485 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`In EABI, these functions have an __aeabi_ prefix, but in GNUEABI they have`。
- **L2486 EN**: Comment explains nearby logic, invariants, or intent: `a __gnu_ prefix (which is the default).`.
  **L2486 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`a __gnu_ prefix (which is the default).`。
- **L2487 EN**: Comment explains nearby logic, invariants, or intent: `isTargetAEABI()`.
  **L2487 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`isTargetAEABI()`。
- **L2488 EN**: Declares TableGen def `__aeabi_f2h`.
  **L2488 CN**: 声明 TableGen def `__aeabi_f2h`。
- **L2489 EN**: Declares TableGen def `__aeabi_h2f`.
  **L2489 CN**: 声明 TableGen def `__aeabi_h2f`。
- **L2490 EN**: Blank line separating nearby declarations or logic blocks.
  **L2490 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2491 EN**: Comment explains nearby logic, invariants, or intent: `isTargetMachO()`.
  **L2491 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`isTargetMachO()`。
- **L2492 EN**: Declares TableGen def `__gnu_f2h_ieee`.
  **L2492 CN**: 声明 TableGen def `__gnu_f2h_ieee`。
- **L2493 EN**: Declares TableGen def `__gnu_h2f_ieee`.
  **L2493 CN**: 声明 TableGen def `__gnu_h2f_ieee`。
- **L2494 EN**: Blank line separating nearby declarations or logic blocks.
  **L2494 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2495 EN**: Comment explains nearby logic, invariants, or intent: `The half <-> float conversion functions are always soft-float on`.
  **L2495 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`The half <-> float conversion functions are always soft-float on`。
- **L2496 EN**: Comment explains nearby logic, invariants, or intent: `non-watchos platforms, but are needed for some targets which use a`.
  **L2496 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`non-watchos platforms, but are needed for some targets which use a`。

### Lines 2497-2528

````tablegen
// hard-float calling convention by default.
def ARMHalfConvertLibcallCallingConv : LibcallCallingConv<
  [{TT.isWatchABI() ? static_cast<CallingConv::ID>(DefaultCC) :
    static_cast<CallingConv::ID>(isAAPCS_ABI(TT, ABIName) ? CallingConv::ARM_AAPCS : CallingConv::ARM_APCS)}]
>;

def ARMLibgccHalfConvertCalls :
  LibcallImpls<(add __truncsfhf2, __extendhfsf2),
    RuntimeLibcallPredicate<[{!TT.isTargetAEABI() && TT.isOSBinFormatMachO()}]>> {
  let CallingConv = ARMHalfConvertLibcallCallingConv;
}

// FIXME: These conditions are probably bugged. We're using the
// default libgcc call when the other cases are replaced.
def ARMDoubleToHalfCalls :
  LibcallImpls<(add __truncdfhf2),
    RuntimeLibcallPredicate<[{!TT.isTargetAEABI()}]>> {
  let CallingConv = ARMHalfConvertLibcallCallingConv;
}

// In EABI, these functions have an __aeabi_ prefix, but in GNUEABI
// they have a __gnu_ prefix (which is the default).
def EABIHalfConvertCalls : LibcallImpls<(add __aeabi_f2h, __aeabi_h2f),
                                        isTargetAEABIAndAAPCS_ABI> {
  let CallingConv = ARM_AAPCS;
}

def GNUEABIHalfConvertCalls :
  LibcallImpls<(add __gnu_f2h_ieee, __gnu_h2f_ieee),
    RuntimeLibcallPredicate<[{!TT.isOSBinFormatMachO() &&
                              !TT.isTargetAEABI()}]>> {
  let CallingConv = ARMHalfConvertLibcallCallingConv;
````
- **L2497 EN**: Comment explains nearby logic, invariants, or intent: `hard-float calling convention by default.`.
  **L2497 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`hard-float calling convention by default.`。
- **L2498 EN**: Declares TableGen def `ARMHalfConvertLibcallCallingConv`.
  **L2498 CN**: 声明 TableGen def `ARMHalfConvertLibcallCallingConv`。
- **L2499 EN**: Continues logic associated with callable symbol `isWatchABI`.
  **L2499 CN**: 继续与可调用符号 `isWatchABI` 相关的逻辑。
- **L2500 EN**: Continues logic associated with callable symbol `ID>`.
  **L2500 CN**: 继续与可调用符号 `ID>` 相关的逻辑。
- **L2501 EN**: Executes a standalone statement or declaration: `>;`.
  **L2501 CN**: 执行一条独立语句或声明：`>;`。
- **L2502 EN**: Blank line separating nearby declarations or logic blocks.
  **L2502 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2503 EN**: Declares TableGen def `ARMLibgccHalfConvertCalls`.
  **L2503 CN**: 声明 TableGen def `ARMLibgccHalfConvertCalls`。
- **L2504 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LibcallImpls<(add __truncsfhf2, __extendhfsf2),`.
  **L2504 CN**: 继续一个多行参数列表、初始化器或聚合项：`LibcallImpls<(add __truncsfhf2, __extendhfsf2),`。
- **L2505 EN**: Starts a function, method, lambda, or structured scope: `RuntimeLibcallPredicate<[{!TT.isTargetAEABI() && TT.isOSBinFormatMachO()}]>> {`.
  **L2505 CN**: 开始一个函数、方法、lambda 或结构化作用域：`RuntimeLibcallPredicate<[{!TT.isTargetAEABI() && TT.isOSBinFormatMachO()}]>> {`。
- **L2506 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L2506 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2507 EN**: Closes the current lexical scope or compound statement.
  **L2507 CN**: 结束当前词法作用域或复合语句块。
- **L2508 EN**: Blank line separating nearby declarations or logic blocks.
  **L2508 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2509 EN**: Comment records a pending task or caution: `FIXME: These conditions are probably bugged. We're using the`.
  **L2509 CN**: 注释记录了待办事项或注意点：`FIXME: These conditions are probably bugged. We're using the`。
- **L2510 EN**: Comment explains nearby logic, invariants, or intent: `default libgcc call when the other cases are replaced.`.
  **L2510 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`default libgcc call when the other cases are replaced.`。
- **L2511 EN**: Declares TableGen def `ARMDoubleToHalfCalls`.
  **L2511 CN**: 声明 TableGen def `ARMDoubleToHalfCalls`。
- **L2512 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LibcallImpls<(add __truncdfhf2),`.
  **L2512 CN**: 继续一个多行参数列表、初始化器或聚合项：`LibcallImpls<(add __truncdfhf2),`。
- **L2513 EN**: Starts a function, method, lambda, or structured scope: `RuntimeLibcallPredicate<[{!TT.isTargetAEABI()}]>> {`.
  **L2513 CN**: 开始一个函数、方法、lambda 或结构化作用域：`RuntimeLibcallPredicate<[{!TT.isTargetAEABI()}]>> {`。
- **L2514 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L2514 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2515 EN**: Closes the current lexical scope or compound statement.
  **L2515 CN**: 结束当前词法作用域或复合语句块。
- **L2516 EN**: Blank line separating nearby declarations or logic blocks.
  **L2516 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2517 EN**: Comment explains nearby logic, invariants, or intent: `In EABI, these functions have an __aeabi_ prefix, but in GNUEABI`.
  **L2517 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`In EABI, these functions have an __aeabi_ prefix, but in GNUEABI`。
- **L2518 EN**: Comment explains nearby logic, invariants, or intent: `they have a __gnu_ prefix (which is the default).`.
  **L2518 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`they have a __gnu_ prefix (which is the default).`。
- **L2519 EN**: Declares TableGen def `EABIHalfConvertCalls`.
  **L2519 CN**: 声明 TableGen def `EABIHalfConvertCalls`。
- **L2520 EN**: Continues the surrounding expression or declaration: `isTargetAEABIAndAAPCS_ABI> {`.
  **L2520 CN**: 继续构造周围的表达式或声明：`isTargetAEABIAndAAPCS_ABI> {`。
- **L2521 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L2521 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2522 EN**: Closes the current lexical scope or compound statement.
  **L2522 CN**: 结束当前词法作用域或复合语句块。
- **L2523 EN**: Blank line separating nearby declarations or logic blocks.
  **L2523 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2524 EN**: Declares TableGen def `GNUEABIHalfConvertCalls`.
  **L2524 CN**: 声明 TableGen def `GNUEABIHalfConvertCalls`。
- **L2525 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LibcallImpls<(add __gnu_f2h_ieee, __gnu_h2f_ieee),`.
  **L2525 CN**: 继续一个多行参数列表、初始化器或聚合项：`LibcallImpls<(add __gnu_f2h_ieee, __gnu_h2f_ieee),`。
- **L2526 EN**: Continues logic associated with callable symbol `isOSBinFormatMachO`.
  **L2526 CN**: 继续与可调用符号 `isOSBinFormatMachO` 相关的逻辑。
- **L2527 EN**: Starts a function, method, lambda, or structured scope: `!TT.isTargetAEABI()}]>> {`.
  **L2527 CN**: 开始一个函数、方法、lambda 或结构化作用域：`!TT.isTargetAEABI()}]>> {`。
- **L2528 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L2528 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。

### Lines 2529-2560

````tablegen
}

def WindowARMDivRemCalls : LibcallImpls<
  (add __rt_sdiv, __rt_sdiv64, __rt_udiv, __rt_udiv64),
  isOSWindows> {
  let CallingConv = ARM_AAPCS;
}

def WindowARMFPIntCasts : LibcallImpls<
  (add WindowsFPIntCastLibcalls),
  isOSWindows> {
  let CallingConv = ARM_AAPCS_VFP;
}


// Register based DivRem for AEABI (RTABI 4.2)
def AEABIDivRemCalls : LibcallImpls<
  (add __aeabi_idivmod, __aeabi_ldivmod,
       __aeabi_uidivmod, __aeabi_uldivmod),
  RuntimeLibcallPredicate<[{hasAEABILibcalls(TT)}]>> {
  let CallingConv = ARM_AAPCS;
}

def AEABICalls : LibcallImpls<
  (add
      // Double-precision floating-point arithmetic helper functions
      // RTABI chapter 4.1.2, Table 2
      __aeabi_dadd,
      __aeabi_ddiv,
      __aeabi_dmul,
      __aeabi_dsub,

````
- **L2529 EN**: Closes the current lexical scope or compound statement.
  **L2529 CN**: 结束当前词法作用域或复合语句块。
- **L2530 EN**: Blank line separating nearby declarations or logic blocks.
  **L2530 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2531 EN**: Declares TableGen def `WindowARMDivRemCalls`.
  **L2531 CN**: 声明 TableGen def `WindowARMDivRemCalls`。
- **L2532 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(add __rt_sdiv, __rt_sdiv64, __rt_udiv, __rt_udiv64),`.
  **L2532 CN**: 继续一个多行参数列表、初始化器或聚合项：`(add __rt_sdiv, __rt_sdiv64, __rt_udiv, __rt_udiv64),`。
- **L2533 EN**: Continues the surrounding expression or declaration: `isOSWindows> {`.
  **L2533 CN**: 继续构造周围的表达式或声明：`isOSWindows> {`。
- **L2534 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L2534 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2535 EN**: Closes the current lexical scope or compound statement.
  **L2535 CN**: 结束当前词法作用域或复合语句块。
- **L2536 EN**: Blank line separating nearby declarations or logic blocks.
  **L2536 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2537 EN**: Declares TableGen def `WindowARMFPIntCasts`.
  **L2537 CN**: 声明 TableGen def `WindowARMFPIntCasts`。
- **L2538 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(add WindowsFPIntCastLibcalls),`.
  **L2538 CN**: 继续一个多行参数列表、初始化器或聚合项：`(add WindowsFPIntCastLibcalls),`。
- **L2539 EN**: Continues the surrounding expression or declaration: `isOSWindows> {`.
  **L2539 CN**: 继续构造周围的表达式或声明：`isOSWindows> {`。
- **L2540 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L2540 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2541 EN**: Closes the current lexical scope or compound statement.
  **L2541 CN**: 结束当前词法作用域或复合语句块。
- **L2542 EN**: Blank line separating nearby declarations or logic blocks.
  **L2542 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2543 EN**: Blank line separating nearby declarations or logic blocks.
  **L2543 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2544 EN**: Comment explains nearby logic, invariants, or intent: `Register based DivRem for AEABI (RTABI 4.2)`.
  **L2544 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Register based DivRem for AEABI (RTABI 4.2)`。
- **L2545 EN**: Declares TableGen def `AEABIDivRemCalls`.
  **L2545 CN**: 声明 TableGen def `AEABIDivRemCalls`。
- **L2546 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(add __aeabi_idivmod, __aeabi_ldivmod,`.
  **L2546 CN**: 继续一个多行参数列表、初始化器或聚合项：`(add __aeabi_idivmod, __aeabi_ldivmod,`。
- **L2547 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__aeabi_uidivmod, __aeabi_uldivmod),`.
  **L2547 CN**: 继续一个多行参数列表、初始化器或聚合项：`__aeabi_uidivmod, __aeabi_uldivmod),`。
- **L2548 EN**: Starts a function, method, lambda, or structured scope: `RuntimeLibcallPredicate<[{hasAEABILibcalls(TT)}]>> {`.
  **L2548 CN**: 开始一个函数、方法、lambda 或结构化作用域：`RuntimeLibcallPredicate<[{hasAEABILibcalls(TT)}]>> {`。
- **L2549 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L2549 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2550 EN**: Closes the current lexical scope or compound statement.
  **L2550 CN**: 结束当前词法作用域或复合语句块。
- **L2551 EN**: Blank line separating nearby declarations or logic blocks.
  **L2551 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2552 EN**: Declares TableGen def `AEABICalls`.
  **L2552 CN**: 声明 TableGen def `AEABICalls`。
- **L2553 EN**: Continues the surrounding expression or declaration: `(add`.
  **L2553 CN**: 继续构造周围的表达式或声明：`(add`。
- **L2554 EN**: Comment explains nearby logic, invariants, or intent: `Double-precision floating-point arithmetic helper functions`.
  **L2554 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Double-precision floating-point arithmetic helper functions`。
- **L2555 EN**: Comment explains nearby logic, invariants, or intent: `RTABI chapter 4.1.2, Table 2`.
  **L2555 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`RTABI chapter 4.1.2, Table 2`。
- **L2556 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__aeabi_dadd,`.
  **L2556 CN**: 继续一个多行参数列表、初始化器或聚合项：`__aeabi_dadd,`。
- **L2557 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__aeabi_ddiv,`.
  **L2557 CN**: 继续一个多行参数列表、初始化器或聚合项：`__aeabi_ddiv,`。
- **L2558 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__aeabi_dmul,`.
  **L2558 CN**: 继续一个多行参数列表、初始化器或聚合项：`__aeabi_dmul,`。
- **L2559 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__aeabi_dsub,`.
  **L2559 CN**: 继续一个多行参数列表、初始化器或聚合项：`__aeabi_dsub,`。
- **L2560 EN**: Blank line separating nearby declarations or logic blocks.
  **L2560 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 2561-2592

````tablegen
      // Double-precision floating-point comparison helper functions
      // RTABI chapter 4.1.2, Table 3
      __aeabi_dcmpeq__oeq,
      __aeabi_dcmpeq__une,
      __aeabi_dcmplt,
      __aeabi_dcmple,
      __aeabi_dcmpge,
      __aeabi_dcmpgt,
       __aeabi_dcmpun,

      // Single-precision floating-point arithmetic helper functions
      // RTABI chapter 4.1.2, Table 4
      __aeabi_fadd,
      __aeabi_fdiv,
      __aeabi_fmul,
      __aeabi_fsub,

      // Single-precision floating-point comparison helper functions
      // RTABI chapter 4.1.2, Table 5
      __aeabi_fcmpeq__oeq,
      __aeabi_fcmpeq__une,
      __aeabi_fcmplt,
      __aeabi_fcmple,
      __aeabi_fcmpge,
      __aeabi_fcmpgt,
      __aeabi_fcmpun,

      // Floating-point to integer conversions.
      // RTABI chapter 4.1.2, Table 6
      __aeabi_d2iz,
      __aeabi_d2uiz,
      __aeabi_d2lz,
````
- **L2561 EN**: Comment explains nearby logic, invariants, or intent: `Double-precision floating-point comparison helper functions`.
  **L2561 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Double-precision floating-point comparison helper functions`。
- **L2562 EN**: Comment explains nearby logic, invariants, or intent: `RTABI chapter 4.1.2, Table 3`.
  **L2562 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`RTABI chapter 4.1.2, Table 3`。
- **L2563 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__aeabi_dcmpeq__oeq,`.
  **L2563 CN**: 继续一个多行参数列表、初始化器或聚合项：`__aeabi_dcmpeq__oeq,`。
- **L2564 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__aeabi_dcmpeq__une,`.
  **L2564 CN**: 继续一个多行参数列表、初始化器或聚合项：`__aeabi_dcmpeq__une,`。
- **L2565 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__aeabi_dcmplt,`.
  **L2565 CN**: 继续一个多行参数列表、初始化器或聚合项：`__aeabi_dcmplt,`。
- **L2566 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__aeabi_dcmple,`.
  **L2566 CN**: 继续一个多行参数列表、初始化器或聚合项：`__aeabi_dcmple,`。
- **L2567 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__aeabi_dcmpge,`.
  **L2567 CN**: 继续一个多行参数列表、初始化器或聚合项：`__aeabi_dcmpge,`。
- **L2568 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__aeabi_dcmpgt,`.
  **L2568 CN**: 继续一个多行参数列表、初始化器或聚合项：`__aeabi_dcmpgt,`。
- **L2569 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__aeabi_dcmpun,`.
  **L2569 CN**: 继续一个多行参数列表、初始化器或聚合项：`__aeabi_dcmpun,`。
- **L2570 EN**: Blank line separating nearby declarations or logic blocks.
  **L2570 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2571 EN**: Comment explains nearby logic, invariants, or intent: `Single-precision floating-point arithmetic helper functions`.
  **L2571 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Single-precision floating-point arithmetic helper functions`。
- **L2572 EN**: Comment explains nearby logic, invariants, or intent: `RTABI chapter 4.1.2, Table 4`.
  **L2572 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`RTABI chapter 4.1.2, Table 4`。
- **L2573 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__aeabi_fadd,`.
  **L2573 CN**: 继续一个多行参数列表、初始化器或聚合项：`__aeabi_fadd,`。
- **L2574 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__aeabi_fdiv,`.
  **L2574 CN**: 继续一个多行参数列表、初始化器或聚合项：`__aeabi_fdiv,`。
- **L2575 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__aeabi_fmul,`.
  **L2575 CN**: 继续一个多行参数列表、初始化器或聚合项：`__aeabi_fmul,`。
- **L2576 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__aeabi_fsub,`.
  **L2576 CN**: 继续一个多行参数列表、初始化器或聚合项：`__aeabi_fsub,`。
- **L2577 EN**: Blank line separating nearby declarations or logic blocks.
  **L2577 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2578 EN**: Comment explains nearby logic, invariants, or intent: `Single-precision floating-point comparison helper functions`.
  **L2578 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Single-precision floating-point comparison helper functions`。
- **L2579 EN**: Comment explains nearby logic, invariants, or intent: `RTABI chapter 4.1.2, Table 5`.
  **L2579 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`RTABI chapter 4.1.2, Table 5`。
- **L2580 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__aeabi_fcmpeq__oeq,`.
  **L2580 CN**: 继续一个多行参数列表、初始化器或聚合项：`__aeabi_fcmpeq__oeq,`。
- **L2581 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__aeabi_fcmpeq__une,`.
  **L2581 CN**: 继续一个多行参数列表、初始化器或聚合项：`__aeabi_fcmpeq__une,`。
- **L2582 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__aeabi_fcmplt,`.
  **L2582 CN**: 继续一个多行参数列表、初始化器或聚合项：`__aeabi_fcmplt,`。
- **L2583 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__aeabi_fcmple,`.
  **L2583 CN**: 继续一个多行参数列表、初始化器或聚合项：`__aeabi_fcmple,`。
- **L2584 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__aeabi_fcmpge,`.
  **L2584 CN**: 继续一个多行参数列表、初始化器或聚合项：`__aeabi_fcmpge,`。
- **L2585 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__aeabi_fcmpgt,`.
  **L2585 CN**: 继续一个多行参数列表、初始化器或聚合项：`__aeabi_fcmpgt,`。
- **L2586 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__aeabi_fcmpun,`.
  **L2586 CN**: 继续一个多行参数列表、初始化器或聚合项：`__aeabi_fcmpun,`。
- **L2587 EN**: Blank line separating nearby declarations or logic blocks.
  **L2587 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2588 EN**: Comment explains nearby logic, invariants, or intent: `Floating-point to integer conversions.`.
  **L2588 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Floating-point to integer conversions.`。
- **L2589 EN**: Comment explains nearby logic, invariants, or intent: `RTABI chapter 4.1.2, Table 6`.
  **L2589 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`RTABI chapter 4.1.2, Table 6`。
- **L2590 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__aeabi_d2iz,`.
  **L2590 CN**: 继续一个多行参数列表、初始化器或聚合项：`__aeabi_d2iz,`。
- **L2591 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__aeabi_d2uiz,`.
  **L2591 CN**: 继续一个多行参数列表、初始化器或聚合项：`__aeabi_d2uiz,`。
- **L2592 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__aeabi_d2lz,`.
  **L2592 CN**: 继续一个多行参数列表、初始化器或聚合项：`__aeabi_d2lz,`。

### Lines 2593-2624

````tablegen
      __aeabi_d2ulz,
      __aeabi_f2iz,
      __aeabi_f2uiz,
      __aeabi_f2lz,
      __aeabi_f2ulz,

      // Conversions between floating types.
      // RTABI chapter 4.1.2, Table 7
      __aeabi_d2f,
      __aeabi_f2d,
      //__aeabi_h2f added separately
      //__aeabi_f2h added separately
      __aeabi_d2h,

      // Integer to floating-point conversions.
      // RTABI chapter 4.1.2, Table 8
      __aeabi_i2d,
      __aeabi_ui2d,
      __aeabi_l2d,
      __aeabi_ul2d,
      __aeabi_i2f,
      __aeabi_ui2f,
      __aeabi_l2f,
      __aeabi_ul2f,

      // Long long helper functions
      // RTABI chapter 4.2, Table 9
      __aeabi_lmul,
      __aeabi_llsl,
      __aeabi_llsr,
      __aeabi_lasr,

````
- **L2593 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__aeabi_d2ulz,`.
  **L2593 CN**: 继续一个多行参数列表、初始化器或聚合项：`__aeabi_d2ulz,`。
- **L2594 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__aeabi_f2iz,`.
  **L2594 CN**: 继续一个多行参数列表、初始化器或聚合项：`__aeabi_f2iz,`。
- **L2595 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__aeabi_f2uiz,`.
  **L2595 CN**: 继续一个多行参数列表、初始化器或聚合项：`__aeabi_f2uiz,`。
- **L2596 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__aeabi_f2lz,`.
  **L2596 CN**: 继续一个多行参数列表、初始化器或聚合项：`__aeabi_f2lz,`。
- **L2597 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__aeabi_f2ulz,`.
  **L2597 CN**: 继续一个多行参数列表、初始化器或聚合项：`__aeabi_f2ulz,`。
- **L2598 EN**: Blank line separating nearby declarations or logic blocks.
  **L2598 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2599 EN**: Comment explains nearby logic, invariants, or intent: `Conversions between floating types.`.
  **L2599 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Conversions between floating types.`。
- **L2600 EN**: Comment explains nearby logic, invariants, or intent: `RTABI chapter 4.1.2, Table 7`.
  **L2600 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`RTABI chapter 4.1.2, Table 7`。
- **L2601 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__aeabi_d2f,`.
  **L2601 CN**: 继续一个多行参数列表、初始化器或聚合项：`__aeabi_d2f,`。
- **L2602 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__aeabi_f2d,`.
  **L2602 CN**: 继续一个多行参数列表、初始化器或聚合项：`__aeabi_f2d,`。
- **L2603 EN**: Comment explains nearby logic, invariants, or intent: `__aeabi_h2f added separately`.
  **L2603 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`__aeabi_h2f added separately`。
- **L2604 EN**: Comment explains nearby logic, invariants, or intent: `__aeabi_f2h added separately`.
  **L2604 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`__aeabi_f2h added separately`。
- **L2605 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__aeabi_d2h,`.
  **L2605 CN**: 继续一个多行参数列表、初始化器或聚合项：`__aeabi_d2h,`。
- **L2606 EN**: Blank line separating nearby declarations or logic blocks.
  **L2606 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2607 EN**: Comment explains nearby logic, invariants, or intent: `Integer to floating-point conversions.`.
  **L2607 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Integer to floating-point conversions.`。
- **L2608 EN**: Comment explains nearby logic, invariants, or intent: `RTABI chapter 4.1.2, Table 8`.
  **L2608 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`RTABI chapter 4.1.2, Table 8`。
- **L2609 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__aeabi_i2d,`.
  **L2609 CN**: 继续一个多行参数列表、初始化器或聚合项：`__aeabi_i2d,`。
- **L2610 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__aeabi_ui2d,`.
  **L2610 CN**: 继续一个多行参数列表、初始化器或聚合项：`__aeabi_ui2d,`。
- **L2611 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__aeabi_l2d,`.
  **L2611 CN**: 继续一个多行参数列表、初始化器或聚合项：`__aeabi_l2d,`。
- **L2612 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__aeabi_ul2d,`.
  **L2612 CN**: 继续一个多行参数列表、初始化器或聚合项：`__aeabi_ul2d,`。
- **L2613 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__aeabi_i2f,`.
  **L2613 CN**: 继续一个多行参数列表、初始化器或聚合项：`__aeabi_i2f,`。
- **L2614 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__aeabi_ui2f,`.
  **L2614 CN**: 继续一个多行参数列表、初始化器或聚合项：`__aeabi_ui2f,`。
- **L2615 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__aeabi_l2f,`.
  **L2615 CN**: 继续一个多行参数列表、初始化器或聚合项：`__aeabi_l2f,`。
- **L2616 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__aeabi_ul2f,`.
  **L2616 CN**: 继续一个多行参数列表、初始化器或聚合项：`__aeabi_ul2f,`。
- **L2617 EN**: Blank line separating nearby declarations or logic blocks.
  **L2617 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2618 EN**: Comment explains nearby logic, invariants, or intent: `Long long helper functions`.
  **L2618 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Long long helper functions`。
- **L2619 EN**: Comment explains nearby logic, invariants, or intent: `RTABI chapter 4.2, Table 9`.
  **L2619 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`RTABI chapter 4.2, Table 9`。
- **L2620 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__aeabi_lmul,`.
  **L2620 CN**: 继续一个多行参数列表、初始化器或聚合项：`__aeabi_lmul,`。
- **L2621 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__aeabi_llsl,`.
  **L2621 CN**: 继续一个多行参数列表、初始化器或聚合项：`__aeabi_llsl,`。
- **L2622 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__aeabi_llsr,`.
  **L2622 CN**: 继续一个多行参数列表、初始化器或聚合项：`__aeabi_llsr,`。
- **L2623 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__aeabi_lasr,`.
  **L2623 CN**: 继续一个多行参数列表、初始化器或聚合项：`__aeabi_lasr,`。
- **L2624 EN**: Blank line separating nearby declarations or logic blocks.
  **L2624 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 2625-2656

````tablegen
      // Integer division functions
      // RTABI chapter 4.3.1
      __aeabi_idiv,
      __aeabi_uidiv),
  RuntimeLibcallPredicate<[{hasAEABILibcalls(TT) && isAAPCS_ABI(TT, ABIName)}]>> {
  let CallingConv = ARM_AAPCS;
}

// EABI dependent RTLIB, Memory operations
// RTABI chapter 4.3.4
def AEABI45MemCalls : LibcallImpls<
  (add __aeabi_memcpy,  __aeabi_memcpy4, __aeabi_memcpy8,
       __aeabi_memmove, __aeabi_memmove4, __aeabi_memmove8,
       __aeabi_memset, __aeabi_memset4, __aeabi_memset8,
       __aeabi_memclr, __aeabi_memclr4, __aeabi_memclr8),
  RuntimeLibcallPredicate<[{(EABIVersion == EABI::EABI4 ||
                             EABIVersion == EABI::EABI5) &&
                             hasAEABILibcalls(TT) && isAAPCS_ABI(TT, ABIName)}]>> {
  let CallingConv = ARM_AAPCS;
}

// Unaligned Memory read/write operations
// RTABI chapter 5.3.3
def AEABIUnalignedMemCalls : LibcallImpls<
  (add __aeabi_uread4,  __aeabi_uread8,
       __aeabi_uwrite4, __aeabi_uwrite8),
  RuntimeLibcallPredicate<[{hasAEABILibcalls(TT) && isAAPCS_ABI(TT, ABIName)}]>> {
  let CallingConv = ARM_AAPCS;
}

def isARMOrThumb : RuntimeLibcallPredicate<"TT.isARM() || TT.isThumb()">;

````
- **L2625 EN**: Comment explains nearby logic, invariants, or intent: `Integer division functions`.
  **L2625 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Integer division functions`。
- **L2626 EN**: Comment explains nearby logic, invariants, or intent: `RTABI chapter 4.3.1`.
  **L2626 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`RTABI chapter 4.3.1`。
- **L2627 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__aeabi_idiv,`.
  **L2627 CN**: 继续一个多行参数列表、初始化器或聚合项：`__aeabi_idiv,`。
- **L2628 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__aeabi_uidiv),`.
  **L2628 CN**: 继续一个多行参数列表、初始化器或聚合项：`__aeabi_uidiv),`。
- **L2629 EN**: Starts a function, method, lambda, or structured scope: `RuntimeLibcallPredicate<[{hasAEABILibcalls(TT) && isAAPCS_ABI(TT, ABIName)}]>> {`.
  **L2629 CN**: 开始一个函数、方法、lambda 或结构化作用域：`RuntimeLibcallPredicate<[{hasAEABILibcalls(TT) && isAAPCS_ABI(TT, ABIName)}]>> {`。
- **L2630 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L2630 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2631 EN**: Closes the current lexical scope or compound statement.
  **L2631 CN**: 结束当前词法作用域或复合语句块。
- **L2632 EN**: Blank line separating nearby declarations or logic blocks.
  **L2632 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2633 EN**: Comment explains nearby logic, invariants, or intent: `EABI dependent RTLIB, Memory operations`.
  **L2633 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`EABI dependent RTLIB, Memory operations`。
- **L2634 EN**: Comment explains nearby logic, invariants, or intent: `RTABI chapter 4.3.4`.
  **L2634 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`RTABI chapter 4.3.4`。
- **L2635 EN**: Declares TableGen def `AEABI45MemCalls`.
  **L2635 CN**: 声明 TableGen def `AEABI45MemCalls`。
- **L2636 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(add __aeabi_memcpy,  __aeabi_memcpy4, __aeabi_memcpy8,`.
  **L2636 CN**: 继续一个多行参数列表、初始化器或聚合项：`(add __aeabi_memcpy,  __aeabi_memcpy4, __aeabi_memcpy8,`。
- **L2637 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__aeabi_memmove, __aeabi_memmove4, __aeabi_memmove8,`.
  **L2637 CN**: 继续一个多行参数列表、初始化器或聚合项：`__aeabi_memmove, __aeabi_memmove4, __aeabi_memmove8,`。
- **L2638 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__aeabi_memset, __aeabi_memset4, __aeabi_memset8,`.
  **L2638 CN**: 继续一个多行参数列表、初始化器或聚合项：`__aeabi_memset, __aeabi_memset4, __aeabi_memset8,`。
- **L2639 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__aeabi_memclr, __aeabi_memclr4, __aeabi_memclr8),`.
  **L2639 CN**: 继续一个多行参数列表、初始化器或聚合项：`__aeabi_memclr, __aeabi_memclr4, __aeabi_memclr8),`。
- **L2640 EN**: Continues the surrounding expression or declaration: `RuntimeLibcallPredicate<[{(EABIVersion == EABI::EABI4 ||`.
  **L2640 CN**: 继续构造周围的表达式或声明：`RuntimeLibcallPredicate<[{(EABIVersion == EABI::EABI4 ||`。
- **L2641 EN**: Continues the surrounding expression or declaration: `EABIVersion == EABI::EABI5) &&`.
  **L2641 CN**: 继续构造周围的表达式或声明：`EABIVersion == EABI::EABI5) &&`。
- **L2642 EN**: Starts a function, method, lambda, or structured scope: `hasAEABILibcalls(TT) && isAAPCS_ABI(TT, ABIName)}]>> {`.
  **L2642 CN**: 开始一个函数、方法、lambda 或结构化作用域：`hasAEABILibcalls(TT) && isAAPCS_ABI(TT, ABIName)}]>> {`。
- **L2643 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L2643 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2644 EN**: Closes the current lexical scope or compound statement.
  **L2644 CN**: 结束当前词法作用域或复合语句块。
- **L2645 EN**: Blank line separating nearby declarations or logic blocks.
  **L2645 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2646 EN**: Comment explains nearby logic, invariants, or intent: `Unaligned Memory read/write operations`.
  **L2646 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Unaligned Memory read/write operations`。
- **L2647 EN**: Comment explains nearby logic, invariants, or intent: `RTABI chapter 5.3.3`.
  **L2647 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`RTABI chapter 5.3.3`。
- **L2648 EN**: Declares TableGen def `AEABIUnalignedMemCalls`.
  **L2648 CN**: 声明 TableGen def `AEABIUnalignedMemCalls`。
- **L2649 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(add __aeabi_uread4,  __aeabi_uread8,`.
  **L2649 CN**: 继续一个多行参数列表、初始化器或聚合项：`(add __aeabi_uread4,  __aeabi_uread8,`。
- **L2650 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__aeabi_uwrite4, __aeabi_uwrite8),`.
  **L2650 CN**: 继续一个多行参数列表、初始化器或聚合项：`__aeabi_uwrite4, __aeabi_uwrite8),`。
- **L2651 EN**: Starts a function, method, lambda, or structured scope: `RuntimeLibcallPredicate<[{hasAEABILibcalls(TT) && isAAPCS_ABI(TT, ABIName)}]>> {`.
  **L2651 CN**: 开始一个函数、方法、lambda 或结构化作用域：`RuntimeLibcallPredicate<[{hasAEABILibcalls(TT) && isAAPCS_ABI(TT, ABIName)}]>> {`。
- **L2652 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L2652 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2653 EN**: Closes the current lexical scope or compound statement.
  **L2653 CN**: 结束当前词法作用域或复合语句块。
- **L2654 EN**: Blank line separating nearby declarations or logic blocks.
  **L2654 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2655 EN**: Declares TableGen def `isARMOrThumb`.
  **L2655 CN**: 声明 TableGen def `isARMOrThumb`。
- **L2656 EN**: Blank line separating nearby declarations or logic blocks.
  **L2656 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 2657-2688

````tablegen
def ARMSystemLibrary
    : SystemRuntimeLibrary<isARMOrThumb,
      (add (sub WinDefaultLibcallImpls, ARMLibgccHalfConvertCalls,
                                        GNUEABIHalfConvertCalls,
                                        ARMDoubleToHalfCalls,
                                        AEABIOverrides),
           LibcallImpls<(add __powisf2, __powidf2), isNotOSMSVCRT>,
           LibmHasFrexpF32, LibmHasLdexpF32,
           LibmHasFrexpF128, LibmHasLdexpF128,
           WindowARMDivRemCalls,
           WindowARMFPIntCasts,
           SecurityCheckCookieIfWinMSVC,
           LibcallImpls<(add __chkstk), isOSWindows>,
           AEABIDivRemCalls,
           DarwinSinCosStret, DarwinExp10, DarwinMemsetPattern,
           LibmHasSinCosF32, LibmHasSinCosF64, LibmHasSinCosF128,
           DefaultLibmExp10,
           LibcSafestackPointerAddress,

           AEABICalls,
           AEABI45MemCalls,
           AEABIUnalignedMemCalls,
           ARMLibgccHalfConvertCalls,
           EABIHalfConvertCalls,
           GNUEABIHalfConvertCalls,
           ARMDoubleToHalfCalls,

           LibcallImpls<(add AEABIOverrides),
             RuntimeLibcallPredicate<[{
               (!hasAEABILibcalls(TT) || !isAAPCS_ABI(TT, ABIName)) &&
               !TT.isOSWindows()
            }]>>,
````
- **L2657 EN**: Declares TableGen def `ARMSystemLibrary`.
  **L2657 CN**: 声明 TableGen def `ARMSystemLibrary`。
- **L2658 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: SystemRuntimeLibrary<isARMOrThumb,`.
  **L2658 CN**: 继续一个多行参数列表、初始化器或聚合项：`: SystemRuntimeLibrary<isARMOrThumb,`。
- **L2659 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(add (sub WinDefaultLibcallImpls, ARMLibgccHalfConvertCalls,`.
  **L2659 CN**: 继续一个多行参数列表、初始化器或聚合项：`(add (sub WinDefaultLibcallImpls, ARMLibgccHalfConvertCalls,`。
- **L2660 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `GNUEABIHalfConvertCalls,`.
  **L2660 CN**: 继续一个多行参数列表、初始化器或聚合项：`GNUEABIHalfConvertCalls,`。
- **L2661 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ARMDoubleToHalfCalls,`.
  **L2661 CN**: 继续一个多行参数列表、初始化器或聚合项：`ARMDoubleToHalfCalls,`。
- **L2662 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `AEABIOverrides),`.
  **L2662 CN**: 继续一个多行参数列表、初始化器或聚合项：`AEABIOverrides),`。
- **L2663 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LibcallImpls<(add __powisf2, __powidf2), isNotOSMSVCRT>,`.
  **L2663 CN**: 继续一个多行参数列表、初始化器或聚合项：`LibcallImpls<(add __powisf2, __powidf2), isNotOSMSVCRT>,`。
- **L2664 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LibmHasFrexpF32, LibmHasLdexpF32,`.
  **L2664 CN**: 继续一个多行参数列表、初始化器或聚合项：`LibmHasFrexpF32, LibmHasLdexpF32,`。
- **L2665 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LibmHasFrexpF128, LibmHasLdexpF128,`.
  **L2665 CN**: 继续一个多行参数列表、初始化器或聚合项：`LibmHasFrexpF128, LibmHasLdexpF128,`。
- **L2666 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `WindowARMDivRemCalls,`.
  **L2666 CN**: 继续一个多行参数列表、初始化器或聚合项：`WindowARMDivRemCalls,`。
- **L2667 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `WindowARMFPIntCasts,`.
  **L2667 CN**: 继续一个多行参数列表、初始化器或聚合项：`WindowARMFPIntCasts,`。
- **L2668 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `SecurityCheckCookieIfWinMSVC,`.
  **L2668 CN**: 继续一个多行参数列表、初始化器或聚合项：`SecurityCheckCookieIfWinMSVC,`。
- **L2669 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LibcallImpls<(add __chkstk), isOSWindows>,`.
  **L2669 CN**: 继续一个多行参数列表、初始化器或聚合项：`LibcallImpls<(add __chkstk), isOSWindows>,`。
- **L2670 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `AEABIDivRemCalls,`.
  **L2670 CN**: 继续一个多行参数列表、初始化器或聚合项：`AEABIDivRemCalls,`。
- **L2671 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DarwinSinCosStret, DarwinExp10, DarwinMemsetPattern,`.
  **L2671 CN**: 继续一个多行参数列表、初始化器或聚合项：`DarwinSinCosStret, DarwinExp10, DarwinMemsetPattern,`。
- **L2672 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LibmHasSinCosF32, LibmHasSinCosF64, LibmHasSinCosF128,`.
  **L2672 CN**: 继续一个多行参数列表、初始化器或聚合项：`LibmHasSinCosF32, LibmHasSinCosF64, LibmHasSinCosF128,`。
- **L2673 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultLibmExp10,`.
  **L2673 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultLibmExp10,`。
- **L2674 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LibcSafestackPointerAddress,`.
  **L2674 CN**: 继续一个多行参数列表、初始化器或聚合项：`LibcSafestackPointerAddress,`。
- **L2675 EN**: Blank line separating nearby declarations or logic blocks.
  **L2675 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2676 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `AEABICalls,`.
  **L2676 CN**: 继续一个多行参数列表、初始化器或聚合项：`AEABICalls,`。
- **L2677 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `AEABI45MemCalls,`.
  **L2677 CN**: 继续一个多行参数列表、初始化器或聚合项：`AEABI45MemCalls,`。
- **L2678 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `AEABIUnalignedMemCalls,`.
  **L2678 CN**: 继续一个多行参数列表、初始化器或聚合项：`AEABIUnalignedMemCalls,`。
- **L2679 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ARMLibgccHalfConvertCalls,`.
  **L2679 CN**: 继续一个多行参数列表、初始化器或聚合项：`ARMLibgccHalfConvertCalls,`。
- **L2680 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `EABIHalfConvertCalls,`.
  **L2680 CN**: 继续一个多行参数列表、初始化器或聚合项：`EABIHalfConvertCalls,`。
- **L2681 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `GNUEABIHalfConvertCalls,`.
  **L2681 CN**: 继续一个多行参数列表、初始化器或聚合项：`GNUEABIHalfConvertCalls,`。
- **L2682 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ARMDoubleToHalfCalls,`.
  **L2682 CN**: 继续一个多行参数列表、初始化器或聚合项：`ARMDoubleToHalfCalls,`。
- **L2683 EN**: Blank line separating nearby declarations or logic blocks.
  **L2683 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2684 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LibcallImpls<(add AEABIOverrides),`.
  **L2684 CN**: 继续一个多行参数列表、初始化器或聚合项：`LibcallImpls<(add AEABIOverrides),`。
- **L2685 EN**: Continues the surrounding expression or declaration: `RuntimeLibcallPredicate<[{`.
  **L2685 CN**: 继续构造周围的表达式或声明：`RuntimeLibcallPredicate<[{`。
- **L2686 EN**: Continues logic associated with callable symbol `hasAEABILibcalls`.
  **L2686 CN**: 继续与可调用符号 `hasAEABILibcalls` 相关的逻辑。
- **L2687 EN**: Continues logic associated with callable symbol `isOSWindows`.
  **L2687 CN**: 继续与可调用符号 `isOSWindows` 相关的逻辑。
- **L2688 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `}]>>,`.
  **L2688 CN**: 继续一个多行参数列表、初始化器或聚合项：`}]>>,`。

### Lines 2689-2720

````tablegen
           // Use divmod compiler-rt calls for iOS 5.0 and later.
           LibcallImpls<(add __divmodsi4, __udivmodsi4),
                        RuntimeLibcallPredicate<[{TT.isOSBinFormatMachO() &&
                                                  (!TT.isiOS() || !TT.isOSVersionLT(5, 0))}]>>,
           DefaultStackProtector,
           DefaultSafeStackGlobals)> {
  let DefaultLibcallCallingConv = LibcallCallingConv<[{
     TT.isOSDarwin() ? CallingConv::C :
        (FloatABI == FloatABI::Hard ? CallingConv::ARM_AAPCS_VFP
                                    : CallingConv::ARM_AAPCS)
  }]>;
}

//===----------------------------------------------------------------------===//
// AVR Runtime Libcalls
//===----------------------------------------------------------------------===//

// Several of the runtime library functions use a special calling conv
def __divmodqi4 : RuntimeLibcallImpl<SDIVREM_I8>;
def __divmodhi4 : RuntimeLibcallImpl<SDIVREM_I16>;
def __udivmodqi4 : RuntimeLibcallImpl<UDIVREM_I8>;
def __udivmodhi4 : RuntimeLibcallImpl<UDIVREM_I16>;

//def __divmodsi4 : RuntimeLibcallImpl<SDIVREM_I32>;
//def __udivmodsi4 : RuntimeLibcallImpl<UDIVREM_I32>;

// Standard sinf/cosf name replaced with "sin" and "cos". Define a
// separate Impl so we can set a different type signature.
def avr_sin : RuntimeLibcallImpl<SIN_F32, "sin">;
def avr_cos : RuntimeLibcallImpl<COS_F32, "cos">;

def isAVR : RuntimeLibcallPredicate<"TT.getArch() == Triple::avr">;
````
- **L2689 EN**: Comment explains nearby logic, invariants, or intent: `Use divmod compiler-rt calls for iOS 5.0 and later.`.
  **L2689 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Use divmod compiler-rt calls for iOS 5.0 and later.`。
- **L2690 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LibcallImpls<(add __divmodsi4, __udivmodsi4),`.
  **L2690 CN**: 继续一个多行参数列表、初始化器或聚合项：`LibcallImpls<(add __divmodsi4, __udivmodsi4),`。
- **L2691 EN**: Continues logic associated with callable symbol `isOSBinFormatMachO`.
  **L2691 CN**: 继续与可调用符号 `isOSBinFormatMachO` 相关的逻辑。
- **L2692 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(!TT.isiOS() || !TT.isOSVersionLT(5, 0))}]>>,`.
  **L2692 CN**: 继续一个多行参数列表、初始化器或聚合项：`(!TT.isiOS() || !TT.isOSVersionLT(5, 0))}]>>,`。
- **L2693 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultStackProtector,`.
  **L2693 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultStackProtector,`。
- **L2694 EN**: Continues the surrounding expression or declaration: `DefaultSafeStackGlobals)> {`.
  **L2694 CN**: 继续构造周围的表达式或声明：`DefaultSafeStackGlobals)> {`。
- **L2695 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L2695 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2696 EN**: Continues logic associated with callable symbol `isOSDarwin`.
  **L2696 CN**: 继续与可调用符号 `isOSDarwin` 相关的逻辑。
- **L2697 EN**: Continues the surrounding expression or declaration: `(FloatABI == FloatABI::Hard ? CallingConv::ARM_AAPCS_VFP`.
  **L2697 CN**: 继续构造周围的表达式或声明：`(FloatABI == FloatABI::Hard ? CallingConv::ARM_AAPCS_VFP`。
- **L2698 EN**: Continues the surrounding expression or declaration: `: CallingConv::ARM_AAPCS)`.
  **L2698 CN**: 继续构造周围的表达式或声明：`: CallingConv::ARM_AAPCS)`。
- **L2699 EN**: Executes a standalone statement or declaration: `}]>;`.
  **L2699 CN**: 执行一条独立语句或声明：`}]>;`。
- **L2700 EN**: Closes the current lexical scope or compound statement.
  **L2700 CN**: 结束当前词法作用域或复合语句块。
- **L2701 EN**: Blank line separating nearby declarations or logic blocks.
  **L2701 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2702 EN**: Banner comment marking a file or section boundary.
  **L2702 CN**: 横幅注释，用于标记文件或章节边界。
- **L2703 EN**: Comment explains nearby logic, invariants, or intent: `AVR Runtime Libcalls`.
  **L2703 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`AVR Runtime Libcalls`。
- **L2704 EN**: Banner comment marking a file or section boundary.
  **L2704 CN**: 横幅注释，用于标记文件或章节边界。
- **L2705 EN**: Blank line separating nearby declarations or logic blocks.
  **L2705 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2706 EN**: Comment explains nearby logic, invariants, or intent: `Several of the runtime library functions use a special calling conv`.
  **L2706 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Several of the runtime library functions use a special calling conv`。
- **L2707 EN**: Declares TableGen def `__divmodqi4`.
  **L2707 CN**: 声明 TableGen def `__divmodqi4`。
- **L2708 EN**: Declares TableGen def `__divmodhi4`.
  **L2708 CN**: 声明 TableGen def `__divmodhi4`。
- **L2709 EN**: Declares TableGen def `__udivmodqi4`.
  **L2709 CN**: 声明 TableGen def `__udivmodqi4`。
- **L2710 EN**: Declares TableGen def `__udivmodhi4`.
  **L2710 CN**: 声明 TableGen def `__udivmodhi4`。
- **L2711 EN**: Blank line separating nearby declarations or logic blocks.
  **L2711 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2712 EN**: Comment explains nearby logic, invariants, or intent: `def __divmodsi4 : RuntimeLibcallImpl<SDIVREM_I32>;`.
  **L2712 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`def __divmodsi4 : RuntimeLibcallImpl<SDIVREM_I32>;`。
- **L2713 EN**: Comment explains nearby logic, invariants, or intent: `def __udivmodsi4 : RuntimeLibcallImpl<UDIVREM_I32>;`.
  **L2713 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`def __udivmodsi4 : RuntimeLibcallImpl<UDIVREM_I32>;`。
- **L2714 EN**: Blank line separating nearby declarations or logic blocks.
  **L2714 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2715 EN**: Comment explains nearby logic, invariants, or intent: `Standard sinf/cosf name replaced with "sin" and "cos". Define a`.
  **L2715 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Standard sinf/cosf name replaced with "sin" and "cos". Define a`。
- **L2716 EN**: Comment explains nearby logic, invariants, or intent: `separate Impl so we can set a different type signature.`.
  **L2716 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`separate Impl so we can set a different type signature.`。
- **L2717 EN**: Declares TableGen def `avr_sin`.
  **L2717 CN**: 声明 TableGen def `avr_sin`。
- **L2718 EN**: Declares TableGen def `avr_cos`.
  **L2718 CN**: 声明 TableGen def `avr_cos`。
- **L2719 EN**: Blank line separating nearby declarations or logic blocks.
  **L2719 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2720 EN**: Declares TableGen def `isAVR`.
  **L2720 CN**: 声明 TableGen def `isAVR`。

### Lines 2721-2752

````tablegen

def AVRSystemLibrary
    : SystemRuntimeLibrary<
          isAVR,
          (add (sub DefaultRuntimeLibcallImpls,

               // Division rtlib functions (not supported), use divmod
               // functions instead
               __divqi3, __divhi3, __divsi3, __udivqi3, __udivhi3, __udivsi3,

               // Modulus rtlib functions (not supported), use divmod functions
               // instead
               __modqi3, __modhi3, __modsi3, __umodqi3, __umodhi3, __umodsi3,

               // Standard f64 names are replaced
               sin, cos, sinf, cosf),

              // Several of the runtime library functions use a special calling
              // conv
              LibcallsWithCC<(add __divmodqi4, __divmodhi4, __udivmodqi4,
                                 __udivmodhi4),
                             AVR_BUILTIN>,
              __divmodsi4, __udivmodsi4,
              // Trigonometric rtlib functions
              avr_sin, avr_cos)>;

//===----------------------------------------------------------------------===//
// DXIL Runtime Libcalls
//===----------------------------------------------------------------------===//

def isDXIL : RuntimeLibcallPredicate<"TT.isDXIL()">;

````
- **L2721 EN**: Blank line separating nearby declarations or logic blocks.
  **L2721 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2722 EN**: Declares TableGen def `AVRSystemLibrary`.
  **L2722 CN**: 声明 TableGen def `AVRSystemLibrary`。
- **L2723 EN**: Continues the surrounding expression or declaration: `: SystemRuntimeLibrary<`.
  **L2723 CN**: 继续构造周围的表达式或声明：`: SystemRuntimeLibrary<`。
- **L2724 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `isAVR,`.
  **L2724 CN**: 继续一个多行参数列表、初始化器或聚合项：`isAVR,`。
- **L2725 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(add (sub DefaultRuntimeLibcallImpls,`.
  **L2725 CN**: 继续一个多行参数列表、初始化器或聚合项：`(add (sub DefaultRuntimeLibcallImpls,`。
- **L2726 EN**: Blank line separating nearby declarations or logic blocks.
  **L2726 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2727 EN**: Comment explains nearby logic, invariants, or intent: `Division rtlib functions (not supported), use divmod`.
  **L2727 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Division rtlib functions (not supported), use divmod`。
- **L2728 EN**: Comment explains nearby logic, invariants, or intent: `functions instead`.
  **L2728 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`functions instead`。
- **L2729 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__divqi3, __divhi3, __divsi3, __udivqi3, __udivhi3, __udivsi3,`.
  **L2729 CN**: 继续一个多行参数列表、初始化器或聚合项：`__divqi3, __divhi3, __divsi3, __udivqi3, __udivhi3, __udivsi3,`。
- **L2730 EN**: Blank line separating nearby declarations or logic blocks.
  **L2730 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2731 EN**: Comment explains nearby logic, invariants, or intent: `Modulus rtlib functions (not supported), use divmod functions`.
  **L2731 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Modulus rtlib functions (not supported), use divmod functions`。
- **L2732 EN**: Comment explains nearby logic, invariants, or intent: `instead`.
  **L2732 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`instead`。
- **L2733 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__modqi3, __modhi3, __modsi3, __umodqi3, __umodhi3, __umodsi3,`.
  **L2733 CN**: 继续一个多行参数列表、初始化器或聚合项：`__modqi3, __modhi3, __modsi3, __umodqi3, __umodhi3, __umodsi3,`。
- **L2734 EN**: Blank line separating nearby declarations or logic blocks.
  **L2734 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2735 EN**: Comment explains nearby logic, invariants, or intent: `Standard f64 names are replaced`.
  **L2735 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Standard f64 names are replaced`。
- **L2736 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `sin, cos, sinf, cosf),`.
  **L2736 CN**: 继续一个多行参数列表、初始化器或聚合项：`sin, cos, sinf, cosf),`。
- **L2737 EN**: Blank line separating nearby declarations or logic blocks.
  **L2737 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2738 EN**: Comment explains nearby logic, invariants, or intent: `Several of the runtime library functions use a special calling`.
  **L2738 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Several of the runtime library functions use a special calling`。
- **L2739 EN**: Comment explains nearby logic, invariants, or intent: `conv`.
  **L2739 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`conv`。
- **L2740 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LibcallsWithCC<(add __divmodqi4, __divmodhi4, __udivmodqi4,`.
  **L2740 CN**: 继续一个多行参数列表、初始化器或聚合项：`LibcallsWithCC<(add __divmodqi4, __divmodhi4, __udivmodqi4,`。
- **L2741 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__udivmodhi4),`.
  **L2741 CN**: 继续一个多行参数列表、初始化器或聚合项：`__udivmodhi4),`。
- **L2742 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `AVR_BUILTIN>,`.
  **L2742 CN**: 继续一个多行参数列表、初始化器或聚合项：`AVR_BUILTIN>,`。
- **L2743 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__divmodsi4, __udivmodsi4,`.
  **L2743 CN**: 继续一个多行参数列表、初始化器或聚合项：`__divmodsi4, __udivmodsi4,`。
- **L2744 EN**: Comment explains nearby logic, invariants, or intent: `Trigonometric rtlib functions`.
  **L2744 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Trigonometric rtlib functions`。
- **L2745 EN**: Executes a standalone statement or declaration: `avr_sin, avr_cos)>;`.
  **L2745 CN**: 执行一条独立语句或声明：`avr_sin, avr_cos)>;`。
- **L2746 EN**: Blank line separating nearby declarations or logic blocks.
  **L2746 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2747 EN**: Banner comment marking a file or section boundary.
  **L2747 CN**: 横幅注释，用于标记文件或章节边界。
- **L2748 EN**: Comment explains nearby logic, invariants, or intent: `DXIL Runtime Libcalls`.
  **L2748 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`DXIL Runtime Libcalls`。
- **L2749 EN**: Banner comment marking a file or section boundary.
  **L2749 CN**: 横幅注释，用于标记文件或章节边界。
- **L2750 EN**: Blank line separating nearby declarations or logic blocks.
  **L2750 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2751 EN**: Declares TableGen def `isDXIL`.
  **L2751 CN**: 声明 TableGen def `isDXIL`。
- **L2752 EN**: Blank line separating nearby declarations or logic blocks.
  **L2752 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 2753-2784

````tablegen
// No calls
def DXILSystemLibrary : SystemRuntimeLibrary<isDXIL, (add)>;

//===----------------------------------------------------------------------===//
// Hexagon Runtime Libcalls
//===----------------------------------------------------------------------===//

defset list<RuntimeLibcallImpl> HexagonLibcalls = {
def __hexagon_divsi3 : RuntimeLibcallImpl<SDIV_I32>;
def __hexagon_divdi3 : RuntimeLibcallImpl<SDIV_I64>;
def __hexagon_udivsi3 : RuntimeLibcallImpl<UDIV_I32>;
def __hexagon_udivdi3 : RuntimeLibcallImpl<UDIV_I64>;
def __hexagon_modsi3 : RuntimeLibcallImpl<SREM_I32>;
def __hexagon_moddi3 : RuntimeLibcallImpl<SREM_I64>;
def __hexagon_umodsi3 : RuntimeLibcallImpl<UREM_I32>;
def __hexagon_umoddi3 : RuntimeLibcallImpl<UREM_I64>;

def __hexagon_adddf3 : RuntimeLibcallImpl<ADD_F64>;
def __hexagon_fast_adddf3 : RuntimeLibcallImpl<FAST_ADD_F64>;

def __hexagon_subdf3 : RuntimeLibcallImpl<SUB_F64>;
def __hexagon_fast_subdf3 : RuntimeLibcallImpl<FAST_SUB_F64>;

def __hexagon_muldf3 : RuntimeLibcallImpl<MUL_F64>;
def __hexagon_fast_muldf3 : RuntimeLibcallImpl<FAST_MUL_F64>;

def __hexagon_divdf3 : RuntimeLibcallImpl<DIV_F64>;
def __hexagon_fast_divdf3 : RuntimeLibcallImpl<FAST_DIV_F64>;

def __hexagon_divsf3 : RuntimeLibcallImpl<DIV_F32>;
def __hexagon_fast_divsf3 : RuntimeLibcallImpl<FAST_DIV_F32>;

````
- **L2753 EN**: Comment explains nearby logic, invariants, or intent: `No calls`.
  **L2753 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`No calls`。
- **L2754 EN**: Declares TableGen def `DXILSystemLibrary`.
  **L2754 CN**: 声明 TableGen def `DXILSystemLibrary`。
- **L2755 EN**: Blank line separating nearby declarations or logic blocks.
  **L2755 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2756 EN**: Banner comment marking a file or section boundary.
  **L2756 CN**: 横幅注释，用于标记文件或章节边界。
- **L2757 EN**: Comment explains nearby logic, invariants, or intent: `Hexagon Runtime Libcalls`.
  **L2757 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Hexagon Runtime Libcalls`。
- **L2758 EN**: Banner comment marking a file or section boundary.
  **L2758 CN**: 横幅注释，用于标记文件或章节边界。
- **L2759 EN**: Blank line separating nearby declarations or logic blocks.
  **L2759 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2760 EN**: Continues the surrounding expression or declaration: `defset list<RuntimeLibcallImpl> HexagonLibcalls = {`.
  **L2760 CN**: 继续构造周围的表达式或声明：`defset list<RuntimeLibcallImpl> HexagonLibcalls = {`。
- **L2761 EN**: Declares TableGen def `__hexagon_divsi3`.
  **L2761 CN**: 声明 TableGen def `__hexagon_divsi3`。
- **L2762 EN**: Declares TableGen def `__hexagon_divdi3`.
  **L2762 CN**: 声明 TableGen def `__hexagon_divdi3`。
- **L2763 EN**: Declares TableGen def `__hexagon_udivsi3`.
  **L2763 CN**: 声明 TableGen def `__hexagon_udivsi3`。
- **L2764 EN**: Declares TableGen def `__hexagon_udivdi3`.
  **L2764 CN**: 声明 TableGen def `__hexagon_udivdi3`。
- **L2765 EN**: Declares TableGen def `__hexagon_modsi3`.
  **L2765 CN**: 声明 TableGen def `__hexagon_modsi3`。
- **L2766 EN**: Declares TableGen def `__hexagon_moddi3`.
  **L2766 CN**: 声明 TableGen def `__hexagon_moddi3`。
- **L2767 EN**: Declares TableGen def `__hexagon_umodsi3`.
  **L2767 CN**: 声明 TableGen def `__hexagon_umodsi3`。
- **L2768 EN**: Declares TableGen def `__hexagon_umoddi3`.
  **L2768 CN**: 声明 TableGen def `__hexagon_umoddi3`。
- **L2769 EN**: Blank line separating nearby declarations or logic blocks.
  **L2769 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2770 EN**: Declares TableGen def `__hexagon_adddf3`.
  **L2770 CN**: 声明 TableGen def `__hexagon_adddf3`。
- **L2771 EN**: Declares TableGen def `__hexagon_fast_adddf3`.
  **L2771 CN**: 声明 TableGen def `__hexagon_fast_adddf3`。
- **L2772 EN**: Blank line separating nearby declarations or logic blocks.
  **L2772 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2773 EN**: Declares TableGen def `__hexagon_subdf3`.
  **L2773 CN**: 声明 TableGen def `__hexagon_subdf3`。
- **L2774 EN**: Declares TableGen def `__hexagon_fast_subdf3`.
  **L2774 CN**: 声明 TableGen def `__hexagon_fast_subdf3`。
- **L2775 EN**: Blank line separating nearby declarations or logic blocks.
  **L2775 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2776 EN**: Declares TableGen def `__hexagon_muldf3`.
  **L2776 CN**: 声明 TableGen def `__hexagon_muldf3`。
- **L2777 EN**: Declares TableGen def `__hexagon_fast_muldf3`.
  **L2777 CN**: 声明 TableGen def `__hexagon_fast_muldf3`。
- **L2778 EN**: Blank line separating nearby declarations or logic blocks.
  **L2778 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2779 EN**: Declares TableGen def `__hexagon_divdf3`.
  **L2779 CN**: 声明 TableGen def `__hexagon_divdf3`。
- **L2780 EN**: Declares TableGen def `__hexagon_fast_divdf3`.
  **L2780 CN**: 声明 TableGen def `__hexagon_fast_divdf3`。
- **L2781 EN**: Blank line separating nearby declarations or logic blocks.
  **L2781 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2782 EN**: Declares TableGen def `__hexagon_divsf3`.
  **L2782 CN**: 声明 TableGen def `__hexagon_divsf3`。
- **L2783 EN**: Declares TableGen def `__hexagon_fast_divsf3`.
  **L2783 CN**: 声明 TableGen def `__hexagon_fast_divsf3`。
- **L2784 EN**: Blank line separating nearby declarations or logic blocks.
  **L2784 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 2785-2816

````tablegen
def __hexagon_sqrtf : RuntimeLibcallImpl<SQRT_F32>;
def __hexagon_fast2_sqrtf : RuntimeLibcallImpl<FAST_SQRT_F32>;

// This is the only fast library function for sqrtd.
def __hexagon_fast2_sqrtdf2 : RuntimeLibcallImpl<FAST_SQRT_F64>;

def __hexagon_memcpy_likely_aligned_min32bytes_mult8bytes
    : RuntimeLibcallImpl<HEXAGON_MEMCPY_LIKELY_ALIGNED_MIN32BYTES_MULT8BYTES>;

def hexagon_memcpy_forward_vp4cp4n2 : RuntimeLibcallImpl<HEXAGON_VOLATILE_MEMCPY>;
}

def isHexagon : RuntimeLibcallPredicate<"TT.getArch() == Triple::hexagon">;

def HexagonSystemLibrary
    : SystemRuntimeLibrary<isHexagon,
    (add (sub DefaultLibcallImpls32,
    __adddf3, __divsf3, __udivsi3, __udivdi3,
    __umoddi3, __divdf3, __muldf3, __divsi3, __subdf3, sqrtf,
    __divdi3, __umodsi3, __moddi3, __modsi3), HexagonLibcalls,
    LibmHasSinCosF32, LibmHasSinCosF64, LibmHasSinCosF128,
    exp10f, exp10, exp10l_f128, __stack_chk_fail, __stack_chk_guard,
    DefaultSafeStackGlobals)>;

//===----------------------------------------------------------------------===//
// Lanai Runtime Libcalls
//===----------------------------------------------------------------------===//

def isLanai : RuntimeLibcallPredicate<"TT.getArch() == Triple::lanai">;

// Use fast calling convention for library functions.
def LanaiSystemLibrary
````
- **L2785 EN**: Declares TableGen def `__hexagon_sqrtf`.
  **L2785 CN**: 声明 TableGen def `__hexagon_sqrtf`。
- **L2786 EN**: Declares TableGen def `__hexagon_fast2_sqrtf`.
  **L2786 CN**: 声明 TableGen def `__hexagon_fast2_sqrtf`。
- **L2787 EN**: Blank line separating nearby declarations or logic blocks.
  **L2787 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2788 EN**: Comment explains nearby logic, invariants, or intent: `This is the only fast library function for sqrtd.`.
  **L2788 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`This is the only fast library function for sqrtd.`。
- **L2789 EN**: Declares TableGen def `__hexagon_fast2_sqrtdf2`.
  **L2789 CN**: 声明 TableGen def `__hexagon_fast2_sqrtdf2`。
- **L2790 EN**: Blank line separating nearby declarations or logic blocks.
  **L2790 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2791 EN**: Declares TableGen def `__hexagon_memcpy_likely_aligned_min32bytes_mult8bytes`.
  **L2791 CN**: 声明 TableGen def `__hexagon_memcpy_likely_aligned_min32bytes_mult8bytes`。
- **L2792 EN**: Executes a standalone statement or declaration: `: RuntimeLibcallImpl<HEXAGON_MEMCPY_LIKELY_ALIGNED_MIN32BYTES_MULT8BYTES>;`.
  **L2792 CN**: 执行一条独立语句或声明：`: RuntimeLibcallImpl<HEXAGON_MEMCPY_LIKELY_ALIGNED_MIN32BYTES_MULT8BYTES>;`。
- **L2793 EN**: Blank line separating nearby declarations or logic blocks.
  **L2793 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2794 EN**: Declares TableGen def `hexagon_memcpy_forward_vp4cp4n2`.
  **L2794 CN**: 声明 TableGen def `hexagon_memcpy_forward_vp4cp4n2`。
- **L2795 EN**: Closes the current lexical scope or compound statement.
  **L2795 CN**: 结束当前词法作用域或复合语句块。
- **L2796 EN**: Blank line separating nearby declarations or logic blocks.
  **L2796 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2797 EN**: Declares TableGen def `isHexagon`.
  **L2797 CN**: 声明 TableGen def `isHexagon`。
- **L2798 EN**: Blank line separating nearby declarations or logic blocks.
  **L2798 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2799 EN**: Declares TableGen def `HexagonSystemLibrary`.
  **L2799 CN**: 声明 TableGen def `HexagonSystemLibrary`。
- **L2800 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: SystemRuntimeLibrary<isHexagon,`.
  **L2800 CN**: 继续一个多行参数列表、初始化器或聚合项：`: SystemRuntimeLibrary<isHexagon,`。
- **L2801 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(add (sub DefaultLibcallImpls32,`.
  **L2801 CN**: 继续一个多行参数列表、初始化器或聚合项：`(add (sub DefaultLibcallImpls32,`。
- **L2802 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__adddf3, __divsf3, __udivsi3, __udivdi3,`.
  **L2802 CN**: 继续一个多行参数列表、初始化器或聚合项：`__adddf3, __divsf3, __udivsi3, __udivdi3,`。
- **L2803 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__umoddi3, __divdf3, __muldf3, __divsi3, __subdf3, sqrtf,`.
  **L2803 CN**: 继续一个多行参数列表、初始化器或聚合项：`__umoddi3, __divdf3, __muldf3, __divsi3, __subdf3, sqrtf,`。
- **L2804 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__divdi3, __umodsi3, __moddi3, __modsi3), HexagonLibcalls,`.
  **L2804 CN**: 继续一个多行参数列表、初始化器或聚合项：`__divdi3, __umodsi3, __moddi3, __modsi3), HexagonLibcalls,`。
- **L2805 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LibmHasSinCosF32, LibmHasSinCosF64, LibmHasSinCosF128,`.
  **L2805 CN**: 继续一个多行参数列表、初始化器或聚合项：`LibmHasSinCosF32, LibmHasSinCosF64, LibmHasSinCosF128,`。
- **L2806 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `exp10f, exp10, exp10l_f128, __stack_chk_fail, __stack_chk_guard,`.
  **L2806 CN**: 继续一个多行参数列表、初始化器或聚合项：`exp10f, exp10, exp10l_f128, __stack_chk_fail, __stack_chk_guard,`。
- **L2807 EN**: Executes a standalone statement or declaration: `DefaultSafeStackGlobals)>;`.
  **L2807 CN**: 执行一条独立语句或声明：`DefaultSafeStackGlobals)>;`。
- **L2808 EN**: Blank line separating nearby declarations or logic blocks.
  **L2808 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2809 EN**: Banner comment marking a file or section boundary.
  **L2809 CN**: 横幅注释，用于标记文件或章节边界。
- **L2810 EN**: Comment explains nearby logic, invariants, or intent: `Lanai Runtime Libcalls`.
  **L2810 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Lanai Runtime Libcalls`。
- **L2811 EN**: Banner comment marking a file or section boundary.
  **L2811 CN**: 横幅注释，用于标记文件或章节边界。
- **L2812 EN**: Blank line separating nearby declarations or logic blocks.
  **L2812 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2813 EN**: Declares TableGen def `isLanai`.
  **L2813 CN**: 声明 TableGen def `isLanai`。
- **L2814 EN**: Blank line separating nearby declarations or logic blocks.
  **L2814 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2815 EN**: Comment explains nearby logic, invariants, or intent: `Use fast calling convention for library functions.`.
  **L2815 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Use fast calling convention for library functions.`。
- **L2816 EN**: Declares TableGen def `LanaiSystemLibrary`.
  **L2816 CN**: 声明 TableGen def `LanaiSystemLibrary`。

### Lines 2817-2848

````tablegen
    : SystemRuntimeLibrary<isLanai, (add DefaultRuntimeLibcallImpls,
                                        __stack_chk_fail, __stack_chk_guard)> {
  let DefaultLibcallCallingConv = FASTCC;
}

//===----------------------------------------------------------------------===//
// Mips16 Runtime Libcalls
//===----------------------------------------------------------------------===//

// Hard float libcalls
def __mips16_adddf3 : RuntimeLibcallImpl<ADD_F64>;
def __mips16_addsf3 : RuntimeLibcallImpl<ADD_F32>;
def __mips16_divdf3 : RuntimeLibcallImpl<DIV_F64>;
def __mips16_divsf3 : RuntimeLibcallImpl<DIV_F32>;
def __mips16_eqdf2 : RuntimeLibcallImpl<OEQ_F64>;
def __mips16_eqsf2 : RuntimeLibcallImpl<OEQ_F32>;
def __mips16_extendsfdf2 : RuntimeLibcallImpl<FPEXT_F32_F64>;
def __mips16_fix_truncdfsi : RuntimeLibcallImpl<FPTOSINT_F64_I32>;
def __mips16_fix_truncsfsi : RuntimeLibcallImpl<FPTOSINT_F32_I32>;
def __mips16_floatsidf : RuntimeLibcallImpl<SINTTOFP_I32_F64>;
def __mips16_floatsisf : RuntimeLibcallImpl<SINTTOFP_I32_F32>;
def __mips16_floatunsidf : RuntimeLibcallImpl<UINTTOFP_I32_F64>;
def __mips16_floatunsisf : RuntimeLibcallImpl<UINTTOFP_I32_F32>;
def __mips16_gedf2 : RuntimeLibcallImpl<OGE_F64>;
def __mips16_gesf2 : RuntimeLibcallImpl<OGE_F32>;
def __mips16_gtdf2 : RuntimeLibcallImpl<OGT_F64>;
def __mips16_gtsf2 : RuntimeLibcallImpl<OGT_F32>;
def __mips16_ledf2 : RuntimeLibcallImpl<OLE_F64>;
def __mips16_lesf2 : RuntimeLibcallImpl<OLE_F32>;
def __mips16_ltdf2 : RuntimeLibcallImpl<OLT_F64>;
def __mips16_ltsf2 : RuntimeLibcallImpl<OLT_F32>;
def __mips16_muldf3 : RuntimeLibcallImpl<MUL_F64>;
````
- **L2817 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: SystemRuntimeLibrary<isLanai, (add DefaultRuntimeLibcallImpls,`.
  **L2817 CN**: 继续一个多行参数列表、初始化器或聚合项：`: SystemRuntimeLibrary<isLanai, (add DefaultRuntimeLibcallImpls,`。
- **L2818 EN**: Continues the surrounding expression or declaration: `__stack_chk_fail, __stack_chk_guard)> {`.
  **L2818 CN**: 继续构造周围的表达式或声明：`__stack_chk_fail, __stack_chk_guard)> {`。
- **L2819 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L2819 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2820 EN**: Closes the current lexical scope or compound statement.
  **L2820 CN**: 结束当前词法作用域或复合语句块。
- **L2821 EN**: Blank line separating nearby declarations or logic blocks.
  **L2821 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2822 EN**: Banner comment marking a file or section boundary.
  **L2822 CN**: 横幅注释，用于标记文件或章节边界。
- **L2823 EN**: Comment explains nearby logic, invariants, or intent: `Mips16 Runtime Libcalls`.
  **L2823 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Mips16 Runtime Libcalls`。
- **L2824 EN**: Banner comment marking a file or section boundary.
  **L2824 CN**: 横幅注释，用于标记文件或章节边界。
- **L2825 EN**: Blank line separating nearby declarations or logic blocks.
  **L2825 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2826 EN**: Comment explains nearby logic, invariants, or intent: `Hard float libcalls`.
  **L2826 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Hard float libcalls`。
- **L2827 EN**: Declares TableGen def `__mips16_adddf3`.
  **L2827 CN**: 声明 TableGen def `__mips16_adddf3`。
- **L2828 EN**: Declares TableGen def `__mips16_addsf3`.
  **L2828 CN**: 声明 TableGen def `__mips16_addsf3`。
- **L2829 EN**: Declares TableGen def `__mips16_divdf3`.
  **L2829 CN**: 声明 TableGen def `__mips16_divdf3`。
- **L2830 EN**: Declares TableGen def `__mips16_divsf3`.
  **L2830 CN**: 声明 TableGen def `__mips16_divsf3`。
- **L2831 EN**: Declares TableGen def `__mips16_eqdf2`.
  **L2831 CN**: 声明 TableGen def `__mips16_eqdf2`。
- **L2832 EN**: Declares TableGen def `__mips16_eqsf2`.
  **L2832 CN**: 声明 TableGen def `__mips16_eqsf2`。
- **L2833 EN**: Declares TableGen def `__mips16_extendsfdf2`.
  **L2833 CN**: 声明 TableGen def `__mips16_extendsfdf2`。
- **L2834 EN**: Declares TableGen def `__mips16_fix_truncdfsi`.
  **L2834 CN**: 声明 TableGen def `__mips16_fix_truncdfsi`。
- **L2835 EN**: Declares TableGen def `__mips16_fix_truncsfsi`.
  **L2835 CN**: 声明 TableGen def `__mips16_fix_truncsfsi`。
- **L2836 EN**: Declares TableGen def `__mips16_floatsidf`.
  **L2836 CN**: 声明 TableGen def `__mips16_floatsidf`。
- **L2837 EN**: Declares TableGen def `__mips16_floatsisf`.
  **L2837 CN**: 声明 TableGen def `__mips16_floatsisf`。
- **L2838 EN**: Declares TableGen def `__mips16_floatunsidf`.
  **L2838 CN**: 声明 TableGen def `__mips16_floatunsidf`。
- **L2839 EN**: Declares TableGen def `__mips16_floatunsisf`.
  **L2839 CN**: 声明 TableGen def `__mips16_floatunsisf`。
- **L2840 EN**: Declares TableGen def `__mips16_gedf2`.
  **L2840 CN**: 声明 TableGen def `__mips16_gedf2`。
- **L2841 EN**: Declares TableGen def `__mips16_gesf2`.
  **L2841 CN**: 声明 TableGen def `__mips16_gesf2`。
- **L2842 EN**: Declares TableGen def `__mips16_gtdf2`.
  **L2842 CN**: 声明 TableGen def `__mips16_gtdf2`。
- **L2843 EN**: Declares TableGen def `__mips16_gtsf2`.
  **L2843 CN**: 声明 TableGen def `__mips16_gtsf2`。
- **L2844 EN**: Declares TableGen def `__mips16_ledf2`.
  **L2844 CN**: 声明 TableGen def `__mips16_ledf2`。
- **L2845 EN**: Declares TableGen def `__mips16_lesf2`.
  **L2845 CN**: 声明 TableGen def `__mips16_lesf2`。
- **L2846 EN**: Declares TableGen def `__mips16_ltdf2`.
  **L2846 CN**: 声明 TableGen def `__mips16_ltdf2`。
- **L2847 EN**: Declares TableGen def `__mips16_ltsf2`.
  **L2847 CN**: 声明 TableGen def `__mips16_ltsf2`。
- **L2848 EN**: Declares TableGen def `__mips16_muldf3`.
  **L2848 CN**: 声明 TableGen def `__mips16_muldf3`。

### Lines 2849-2880

````tablegen
def __mips16_mulsf3 : RuntimeLibcallImpl<MUL_F32>;
def __mips16_nedf2 : RuntimeLibcallImpl<UNE_F64>;
def __mips16_nesf2 : RuntimeLibcallImpl<UNE_F32>;
def __mips16_ret_dc : RuntimeLibcallImpl<MIPS16_RET_DC>;
def __mips16_ret_df : RuntimeLibcallImpl<MIPS16_RET_DF>;
def __mips16_ret_sc : RuntimeLibcallImpl<MIPS16_RET_SC>;
def __mips16_ret_sf : RuntimeLibcallImpl<MIPS16_RET_SF>;
def __mips16_subdf3 : RuntimeLibcallImpl<SUB_F64>;
def __mips16_subsf3 : RuntimeLibcallImpl<SUB_F32>;
def __mips16_truncdfsf2 : RuntimeLibcallImpl<FPROUND_F64_F32>;
def __mips16_unorddf2 : RuntimeLibcallImpl<UO_F64>;
def __mips16_unordsf2 : RuntimeLibcallImpl<UO_F32>;

//===----------------------------------------------------------------------===//
// MSP430 Runtime Libcalls
//===----------------------------------------------------------------------===//

// EABI Libcalls - EABI Section 6.2

// Floating point conversions - EABI Table 6
def __mspabi_cvtdf : RuntimeLibcallImpl<FPROUND_F64_F32>;
def __mspabi_cvtfd : RuntimeLibcallImpl<FPEXT_F32_F64>;

// The following is NOT implemented in libgcc
//def __mspabi_fixdi : RuntimeLibcallImpl<FPTOSINT_F64_I16>;
def __mspabi_fixdli : RuntimeLibcallImpl<FPTOSINT_F64_I32>;
def __mspabi_fixdlli : RuntimeLibcallImpl<FPTOSINT_F64_I64>;

// The following is NOT implemented in libgcc
//def __mspabi_fixdu : RuntimeLibcallImpl<FPTOUINT_F64_I16>;
def __mspabi_fixdul : RuntimeLibcallImpl<FPTOUINT_F64_I32>;
def __mspabi_fixdull : RuntimeLibcallImpl<FPTOUINT_F64_I64>;
````
- **L2849 EN**: Declares TableGen def `__mips16_mulsf3`.
  **L2849 CN**: 声明 TableGen def `__mips16_mulsf3`。
- **L2850 EN**: Declares TableGen def `__mips16_nedf2`.
  **L2850 CN**: 声明 TableGen def `__mips16_nedf2`。
- **L2851 EN**: Declares TableGen def `__mips16_nesf2`.
  **L2851 CN**: 声明 TableGen def `__mips16_nesf2`。
- **L2852 EN**: Declares TableGen def `__mips16_ret_dc`.
  **L2852 CN**: 声明 TableGen def `__mips16_ret_dc`。
- **L2853 EN**: Declares TableGen def `__mips16_ret_df`.
  **L2853 CN**: 声明 TableGen def `__mips16_ret_df`。
- **L2854 EN**: Declares TableGen def `__mips16_ret_sc`.
  **L2854 CN**: 声明 TableGen def `__mips16_ret_sc`。
- **L2855 EN**: Declares TableGen def `__mips16_ret_sf`.
  **L2855 CN**: 声明 TableGen def `__mips16_ret_sf`。
- **L2856 EN**: Declares TableGen def `__mips16_subdf3`.
  **L2856 CN**: 声明 TableGen def `__mips16_subdf3`。
- **L2857 EN**: Declares TableGen def `__mips16_subsf3`.
  **L2857 CN**: 声明 TableGen def `__mips16_subsf3`。
- **L2858 EN**: Declares TableGen def `__mips16_truncdfsf2`.
  **L2858 CN**: 声明 TableGen def `__mips16_truncdfsf2`。
- **L2859 EN**: Declares TableGen def `__mips16_unorddf2`.
  **L2859 CN**: 声明 TableGen def `__mips16_unorddf2`。
- **L2860 EN**: Declares TableGen def `__mips16_unordsf2`.
  **L2860 CN**: 声明 TableGen def `__mips16_unordsf2`。
- **L2861 EN**: Blank line separating nearby declarations or logic blocks.
  **L2861 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2862 EN**: Banner comment marking a file or section boundary.
  **L2862 CN**: 横幅注释，用于标记文件或章节边界。
- **L2863 EN**: Comment explains nearby logic, invariants, or intent: `MSP430 Runtime Libcalls`.
  **L2863 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`MSP430 Runtime Libcalls`。
- **L2864 EN**: Banner comment marking a file or section boundary.
  **L2864 CN**: 横幅注释，用于标记文件或章节边界。
- **L2865 EN**: Blank line separating nearby declarations or logic blocks.
  **L2865 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2866 EN**: Comment explains nearby logic, invariants, or intent: `EABI Libcalls - EABI Section 6.2`.
  **L2866 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`EABI Libcalls - EABI Section 6.2`。
- **L2867 EN**: Blank line separating nearby declarations or logic blocks.
  **L2867 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2868 EN**: Comment explains nearby logic, invariants, or intent: `Floating point conversions - EABI Table 6`.
  **L2868 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Floating point conversions - EABI Table 6`。
- **L2869 EN**: Declares TableGen def `__mspabi_cvtdf`.
  **L2869 CN**: 声明 TableGen def `__mspabi_cvtdf`。
- **L2870 EN**: Declares TableGen def `__mspabi_cvtfd`.
  **L2870 CN**: 声明 TableGen def `__mspabi_cvtfd`。
- **L2871 EN**: Blank line separating nearby declarations or logic blocks.
  **L2871 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2872 EN**: Comment explains nearby logic, invariants, or intent: `The following is NOT implemented in libgcc`.
  **L2872 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`The following is NOT implemented in libgcc`。
- **L2873 EN**: Comment explains nearby logic, invariants, or intent: `def __mspabi_fixdi : RuntimeLibcallImpl<FPTOSINT_F64_I16>;`.
  **L2873 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`def __mspabi_fixdi : RuntimeLibcallImpl<FPTOSINT_F64_I16>;`。
- **L2874 EN**: Declares TableGen def `__mspabi_fixdli`.
  **L2874 CN**: 声明 TableGen def `__mspabi_fixdli`。
- **L2875 EN**: Declares TableGen def `__mspabi_fixdlli`.
  **L2875 CN**: 声明 TableGen def `__mspabi_fixdlli`。
- **L2876 EN**: Blank line separating nearby declarations or logic blocks.
  **L2876 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2877 EN**: Comment explains nearby logic, invariants, or intent: `The following is NOT implemented in libgcc`.
  **L2877 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`The following is NOT implemented in libgcc`。
- **L2878 EN**: Comment explains nearby logic, invariants, or intent: `def __mspabi_fixdu : RuntimeLibcallImpl<FPTOUINT_F64_I16>;`.
  **L2878 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`def __mspabi_fixdu : RuntimeLibcallImpl<FPTOUINT_F64_I16>;`。
- **L2879 EN**: Declares TableGen def `__mspabi_fixdul`.
  **L2879 CN**: 声明 TableGen def `__mspabi_fixdul`。
- **L2880 EN**: Declares TableGen def `__mspabi_fixdull`.
  **L2880 CN**: 声明 TableGen def `__mspabi_fixdull`。

### Lines 2881-2912

````tablegen

// The following is NOT implemented in libgcc
//def __mspabi_fixfi : RuntimeLibcallImpl<FPTOSINT_F32_I16>;
def __mspabi_fixfli : RuntimeLibcallImpl<FPTOSINT_F32_I32>;
def __mspabi_fixflli : RuntimeLibcallImpl<FPTOSINT_F32_I64>;

// The following is NOT implemented in libgcc
//def __mspabi_fixfu : RuntimeLibcallImpl<FPTOUINT_F32_I16>;
def __mspabi_fixful : RuntimeLibcallImpl<FPTOUINT_F32_I32>;
def __mspabi_fixfull : RuntimeLibcallImpl<FPTOUINT_F32_I64>;

// TODO The following IS implemented in libgcc
//def __mspabi_fltid : RuntimeLibcallImpl<SINTTOFP_I16_F64>;
def __mspabi_fltlid : RuntimeLibcallImpl<SINTTOFP_I32_F64>;

// TODO The following IS implemented in libgcc but is not in the EABI
def __mspabi_fltllid : RuntimeLibcallImpl<SINTTOFP_I64_F64>;

// TODO The following IS implemented in libgcc
//def __mspabi_fltud : RuntimeLibcallImpl<UINTTOFP_I16_F64>;
def __mspabi_fltuld : RuntimeLibcallImpl<UINTTOFP_I32_F64>;

// The following IS implemented in libgcc but is not in the EABI
def __mspabi_fltulld : RuntimeLibcallImpl<UINTTOFP_I64_F64>;

// TODO The following IS implemented in libgcc
//def __mspabi_fltif : RuntimeLibcallImpl<SINTTOFP_I16_F32>;
def __mspabi_fltlif : RuntimeLibcallImpl<SINTTOFP_I32_F32>;

// TODO The following IS implemented in libgcc but is not in the EABI
def __mspabi_fltllif : RuntimeLibcallImpl<SINTTOFP_I64_F32>;

````
- **L2881 EN**: Blank line separating nearby declarations or logic blocks.
  **L2881 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2882 EN**: Comment explains nearby logic, invariants, or intent: `The following is NOT implemented in libgcc`.
  **L2882 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`The following is NOT implemented in libgcc`。
- **L2883 EN**: Comment explains nearby logic, invariants, or intent: `def __mspabi_fixfi : RuntimeLibcallImpl<FPTOSINT_F32_I16>;`.
  **L2883 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`def __mspabi_fixfi : RuntimeLibcallImpl<FPTOSINT_F32_I16>;`。
- **L2884 EN**: Declares TableGen def `__mspabi_fixfli`.
  **L2884 CN**: 声明 TableGen def `__mspabi_fixfli`。
- **L2885 EN**: Declares TableGen def `__mspabi_fixflli`.
  **L2885 CN**: 声明 TableGen def `__mspabi_fixflli`。
- **L2886 EN**: Blank line separating nearby declarations or logic blocks.
  **L2886 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2887 EN**: Comment explains nearby logic, invariants, or intent: `The following is NOT implemented in libgcc`.
  **L2887 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`The following is NOT implemented in libgcc`。
- **L2888 EN**: Comment explains nearby logic, invariants, or intent: `def __mspabi_fixfu : RuntimeLibcallImpl<FPTOUINT_F32_I16>;`.
  **L2888 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`def __mspabi_fixfu : RuntimeLibcallImpl<FPTOUINT_F32_I16>;`。
- **L2889 EN**: Declares TableGen def `__mspabi_fixful`.
  **L2889 CN**: 声明 TableGen def `__mspabi_fixful`。
- **L2890 EN**: Declares TableGen def `__mspabi_fixfull`.
  **L2890 CN**: 声明 TableGen def `__mspabi_fixfull`。
- **L2891 EN**: Blank line separating nearby declarations or logic blocks.
  **L2891 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2892 EN**: Comment records a pending task or caution: `TODO The following IS implemented in libgcc`.
  **L2892 CN**: 注释记录了待办事项或注意点：`TODO The following IS implemented in libgcc`。
- **L2893 EN**: Comment explains nearby logic, invariants, or intent: `def __mspabi_fltid : RuntimeLibcallImpl<SINTTOFP_I16_F64>;`.
  **L2893 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`def __mspabi_fltid : RuntimeLibcallImpl<SINTTOFP_I16_F64>;`。
- **L2894 EN**: Declares TableGen def `__mspabi_fltlid`.
  **L2894 CN**: 声明 TableGen def `__mspabi_fltlid`。
- **L2895 EN**: Blank line separating nearby declarations or logic blocks.
  **L2895 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2896 EN**: Comment records a pending task or caution: `TODO The following IS implemented in libgcc but is not in the EABI`.
  **L2896 CN**: 注释记录了待办事项或注意点：`TODO The following IS implemented in libgcc but is not in the EABI`。
- **L2897 EN**: Declares TableGen def `__mspabi_fltllid`.
  **L2897 CN**: 声明 TableGen def `__mspabi_fltllid`。
- **L2898 EN**: Blank line separating nearby declarations or logic blocks.
  **L2898 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2899 EN**: Comment records a pending task or caution: `TODO The following IS implemented in libgcc`.
  **L2899 CN**: 注释记录了待办事项或注意点：`TODO The following IS implemented in libgcc`。
- **L2900 EN**: Comment explains nearby logic, invariants, or intent: `def __mspabi_fltud : RuntimeLibcallImpl<UINTTOFP_I16_F64>;`.
  **L2900 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`def __mspabi_fltud : RuntimeLibcallImpl<UINTTOFP_I16_F64>;`。
- **L2901 EN**: Declares TableGen def `__mspabi_fltuld`.
  **L2901 CN**: 声明 TableGen def `__mspabi_fltuld`。
- **L2902 EN**: Blank line separating nearby declarations or logic blocks.
  **L2902 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2903 EN**: Comment explains nearby logic, invariants, or intent: `The following IS implemented in libgcc but is not in the EABI`.
  **L2903 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`The following IS implemented in libgcc but is not in the EABI`。
- **L2904 EN**: Declares TableGen def `__mspabi_fltulld`.
  **L2904 CN**: 声明 TableGen def `__mspabi_fltulld`。
- **L2905 EN**: Blank line separating nearby declarations or logic blocks.
  **L2905 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2906 EN**: Comment records a pending task or caution: `TODO The following IS implemented in libgcc`.
  **L2906 CN**: 注释记录了待办事项或注意点：`TODO The following IS implemented in libgcc`。
- **L2907 EN**: Comment explains nearby logic, invariants, or intent: `def __mspabi_fltif : RuntimeLibcallImpl<SINTTOFP_I16_F32>;`.
  **L2907 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`def __mspabi_fltif : RuntimeLibcallImpl<SINTTOFP_I16_F32>;`。
- **L2908 EN**: Declares TableGen def `__mspabi_fltlif`.
  **L2908 CN**: 声明 TableGen def `__mspabi_fltlif`。
- **L2909 EN**: Blank line separating nearby declarations or logic blocks.
  **L2909 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2910 EN**: Comment records a pending task or caution: `TODO The following IS implemented in libgcc but is not in the EABI`.
  **L2910 CN**: 注释记录了待办事项或注意点：`TODO The following IS implemented in libgcc but is not in the EABI`。
- **L2911 EN**: Declares TableGen def `__mspabi_fltllif`.
  **L2911 CN**: 声明 TableGen def `__mspabi_fltllif`。
- **L2912 EN**: Blank line separating nearby declarations or logic blocks.
  **L2912 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 2913-2944

````tablegen
// TODO The following IS implemented in libgcc
//def __mspabi_fltuf : RuntimeLibcallImpl<UINTTOFP_I16_F32>;
def __mspabi_fltulf : RuntimeLibcallImpl<UINTTOFP_I32_F32>;

// The following IS implemented in libgcc but is not in the EABI
def __mspabi_fltullf : RuntimeLibcallImpl<UINTTOFP_I64_F32>;

// Floating point comparisons - EABI Table 7
def __mspabi_cmpd__oeq : RuntimeLibcallImpl<OEQ_F64, "__mspabi_cmpd">;
def __mspabi_cmpd__une : RuntimeLibcallImpl<UNE_F64, "__mspabi_cmpd">;
def __mspabi_cmpd__oge : RuntimeLibcallImpl<OGE_F64, "__mspabi_cmpd">;
def __mspabi_cmpd__olt : RuntimeLibcallImpl<OLT_F64, "__mspabi_cmpd">;
def __mspabi_cmpd__ole : RuntimeLibcallImpl<OLE_F64, "__mspabi_cmpd">;
def __mspabi_cmpd__ogt : RuntimeLibcallImpl<OGT_F64, "__mspabi_cmpd">;
def __mspabi_cmpf__oeq : RuntimeLibcallImpl<OEQ_F32, "__mspabi_cmpf">;
def __mspabi_cmpf__une : RuntimeLibcallImpl<UNE_F32, "__mspabi_cmpf">;
def __mspabi_cmpf__oge : RuntimeLibcallImpl<OGE_F32, "__mspabi_cmpf">;
def __mspabi_cmpf__olt : RuntimeLibcallImpl<OLT_F32, "__mspabi_cmpf">;
def __mspabi_cmpf__ole : RuntimeLibcallImpl<OLE_F32, "__mspabi_cmpf">;
def __mspabi_cmpf__ogt : RuntimeLibcallImpl<OGT_F32, "__mspabi_cmpf">;

// Floating point arithmetic - EABI Table 8
def __mspabi_addd : RuntimeLibcallImpl<ADD_F64>;
def __mspabi_addf : RuntimeLibcallImpl<ADD_F32>;
def __mspabi_divd : RuntimeLibcallImpl<DIV_F64>;
def __mspabi_divf : RuntimeLibcallImpl<DIV_F32>;
def __mspabi_mpyd : RuntimeLibcallImpl<MUL_F64>;
def __mspabi_mpyf : RuntimeLibcallImpl<MUL_F32>;
def __mspabi_subd : RuntimeLibcallImpl<SUB_F64>;
def __mspabi_subf : RuntimeLibcallImpl<SUB_F32>;

// The following are NOT implemented in libgcc
````
- **L2913 EN**: Comment records a pending task or caution: `TODO The following IS implemented in libgcc`.
  **L2913 CN**: 注释记录了待办事项或注意点：`TODO The following IS implemented in libgcc`。
- **L2914 EN**: Comment explains nearby logic, invariants, or intent: `def __mspabi_fltuf : RuntimeLibcallImpl<UINTTOFP_I16_F32>;`.
  **L2914 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`def __mspabi_fltuf : RuntimeLibcallImpl<UINTTOFP_I16_F32>;`。
- **L2915 EN**: Declares TableGen def `__mspabi_fltulf`.
  **L2915 CN**: 声明 TableGen def `__mspabi_fltulf`。
- **L2916 EN**: Blank line separating nearby declarations or logic blocks.
  **L2916 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2917 EN**: Comment explains nearby logic, invariants, or intent: `The following IS implemented in libgcc but is not in the EABI`.
  **L2917 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`The following IS implemented in libgcc but is not in the EABI`。
- **L2918 EN**: Declares TableGen def `__mspabi_fltullf`.
  **L2918 CN**: 声明 TableGen def `__mspabi_fltullf`。
- **L2919 EN**: Blank line separating nearby declarations or logic blocks.
  **L2919 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2920 EN**: Comment explains nearby logic, invariants, or intent: `Floating point comparisons - EABI Table 7`.
  **L2920 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Floating point comparisons - EABI Table 7`。
- **L2921 EN**: Declares TableGen def `__mspabi_cmpd__oeq`.
  **L2921 CN**: 声明 TableGen def `__mspabi_cmpd__oeq`。
- **L2922 EN**: Declares TableGen def `__mspabi_cmpd__une`.
  **L2922 CN**: 声明 TableGen def `__mspabi_cmpd__une`。
- **L2923 EN**: Declares TableGen def `__mspabi_cmpd__oge`.
  **L2923 CN**: 声明 TableGen def `__mspabi_cmpd__oge`。
- **L2924 EN**: Declares TableGen def `__mspabi_cmpd__olt`.
  **L2924 CN**: 声明 TableGen def `__mspabi_cmpd__olt`。
- **L2925 EN**: Declares TableGen def `__mspabi_cmpd__ole`.
  **L2925 CN**: 声明 TableGen def `__mspabi_cmpd__ole`。
- **L2926 EN**: Declares TableGen def `__mspabi_cmpd__ogt`.
  **L2926 CN**: 声明 TableGen def `__mspabi_cmpd__ogt`。
- **L2927 EN**: Declares TableGen def `__mspabi_cmpf__oeq`.
  **L2927 CN**: 声明 TableGen def `__mspabi_cmpf__oeq`。
- **L2928 EN**: Declares TableGen def `__mspabi_cmpf__une`.
  **L2928 CN**: 声明 TableGen def `__mspabi_cmpf__une`。
- **L2929 EN**: Declares TableGen def `__mspabi_cmpf__oge`.
  **L2929 CN**: 声明 TableGen def `__mspabi_cmpf__oge`。
- **L2930 EN**: Declares TableGen def `__mspabi_cmpf__olt`.
  **L2930 CN**: 声明 TableGen def `__mspabi_cmpf__olt`。
- **L2931 EN**: Declares TableGen def `__mspabi_cmpf__ole`.
  **L2931 CN**: 声明 TableGen def `__mspabi_cmpf__ole`。
- **L2932 EN**: Declares TableGen def `__mspabi_cmpf__ogt`.
  **L2932 CN**: 声明 TableGen def `__mspabi_cmpf__ogt`。
- **L2933 EN**: Blank line separating nearby declarations or logic blocks.
  **L2933 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2934 EN**: Comment explains nearby logic, invariants, or intent: `Floating point arithmetic - EABI Table 8`.
  **L2934 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Floating point arithmetic - EABI Table 8`。
- **L2935 EN**: Declares TableGen def `__mspabi_addd`.
  **L2935 CN**: 声明 TableGen def `__mspabi_addd`。
- **L2936 EN**: Declares TableGen def `__mspabi_addf`.
  **L2936 CN**: 声明 TableGen def `__mspabi_addf`。
- **L2937 EN**: Declares TableGen def `__mspabi_divd`.
  **L2937 CN**: 声明 TableGen def `__mspabi_divd`。
- **L2938 EN**: Declares TableGen def `__mspabi_divf`.
  **L2938 CN**: 声明 TableGen def `__mspabi_divf`。
- **L2939 EN**: Declares TableGen def `__mspabi_mpyd`.
  **L2939 CN**: 声明 TableGen def `__mspabi_mpyd`。
- **L2940 EN**: Declares TableGen def `__mspabi_mpyf`.
  **L2940 CN**: 声明 TableGen def `__mspabi_mpyf`。
- **L2941 EN**: Declares TableGen def `__mspabi_subd`.
  **L2941 CN**: 声明 TableGen def `__mspabi_subd`。
- **L2942 EN**: Declares TableGen def `__mspabi_subf`.
  **L2942 CN**: 声明 TableGen def `__mspabi_subf`。
- **L2943 EN**: Blank line separating nearby declarations or logic blocks.
  **L2943 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2944 EN**: Comment explains nearby logic, invariants, or intent: `The following are NOT implemented in libgcc`.
  **L2944 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`The following are NOT implemented in libgcc`。

### Lines 2945-2976

````tablegen
// def __mspabi_negd : RuntimeLibcallImpl<NEG_F64>;
// def __mspabi_negf : RuntimeLibcallImpl<NEG_F32>;

// Universal Integer Operations - EABI Table 9
def __mspabi_divi : RuntimeLibcallImpl<SDIV_I16>;
def __mspabi_divli : RuntimeLibcallImpl<SDIV_I32>;
def __mspabi_divlli : RuntimeLibcallImpl<SDIV_I64>;
def __mspabi_divu : RuntimeLibcallImpl<UDIV_I16>;
def __mspabi_divul : RuntimeLibcallImpl<UDIV_I32>;
def __mspabi_divull : RuntimeLibcallImpl<UDIV_I64>;
def __mspabi_remi : RuntimeLibcallImpl<SREM_I16>;
def __mspabi_remli : RuntimeLibcallImpl<SREM_I32>;
def __mspabi_remlli : RuntimeLibcallImpl<SREM_I64>;
def __mspabi_remu : RuntimeLibcallImpl<UREM_I16>;
def __mspabi_remul : RuntimeLibcallImpl<UREM_I32>;
def __mspabi_remull : RuntimeLibcallImpl<UREM_I64>;

// Bitwise Operations - EABI Table 10
// TODO: __mspabi_[srli/srai/slli] ARE implemented in libgcc
def __mspabi_srll : RuntimeLibcallImpl<SRL_I32>;
def __mspabi_sral : RuntimeLibcallImpl<SRA_I32>;
def __mspabi_slll : RuntimeLibcallImpl<SHL_I32>;
// __mspabi_[srlll/srall/sllll/rlli/rlll] are NOT implemented in libgcc

// hasHWMult16()
// Integer Multiply - EABI Table 9
//def __mspabi_mpyi_hw : RuntimeLibcallImpl<MUL_I16>;
def __mspabi_mpyl_hw : RuntimeLibcallImpl<MUL_I32>;
def __mspabi_mpyll_hw : RuntimeLibcallImpl<MUL_I64>;
// TODO The __mspabi_mpysl*_hw functions ARE implemented in libgcc
// TODO The __mspabi_mpyul*_hw functions ARE implemented in libgcc

````
- **L2945 EN**: Comment explains nearby logic, invariants, or intent: `def __mspabi_negd : RuntimeLibcallImpl<NEG_F64>;`.
  **L2945 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`def __mspabi_negd : RuntimeLibcallImpl<NEG_F64>;`。
- **L2946 EN**: Comment explains nearby logic, invariants, or intent: `def __mspabi_negf : RuntimeLibcallImpl<NEG_F32>;`.
  **L2946 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`def __mspabi_negf : RuntimeLibcallImpl<NEG_F32>;`。
- **L2947 EN**: Blank line separating nearby declarations or logic blocks.
  **L2947 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2948 EN**: Comment explains nearby logic, invariants, or intent: `Universal Integer Operations - EABI Table 9`.
  **L2948 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Universal Integer Operations - EABI Table 9`。
- **L2949 EN**: Declares TableGen def `__mspabi_divi`.
  **L2949 CN**: 声明 TableGen def `__mspabi_divi`。
- **L2950 EN**: Declares TableGen def `__mspabi_divli`.
  **L2950 CN**: 声明 TableGen def `__mspabi_divli`。
- **L2951 EN**: Declares TableGen def `__mspabi_divlli`.
  **L2951 CN**: 声明 TableGen def `__mspabi_divlli`。
- **L2952 EN**: Declares TableGen def `__mspabi_divu`.
  **L2952 CN**: 声明 TableGen def `__mspabi_divu`。
- **L2953 EN**: Declares TableGen def `__mspabi_divul`.
  **L2953 CN**: 声明 TableGen def `__mspabi_divul`。
- **L2954 EN**: Declares TableGen def `__mspabi_divull`.
  **L2954 CN**: 声明 TableGen def `__mspabi_divull`。
- **L2955 EN**: Declares TableGen def `__mspabi_remi`.
  **L2955 CN**: 声明 TableGen def `__mspabi_remi`。
- **L2956 EN**: Declares TableGen def `__mspabi_remli`.
  **L2956 CN**: 声明 TableGen def `__mspabi_remli`。
- **L2957 EN**: Declares TableGen def `__mspabi_remlli`.
  **L2957 CN**: 声明 TableGen def `__mspabi_remlli`。
- **L2958 EN**: Declares TableGen def `__mspabi_remu`.
  **L2958 CN**: 声明 TableGen def `__mspabi_remu`。
- **L2959 EN**: Declares TableGen def `__mspabi_remul`.
  **L2959 CN**: 声明 TableGen def `__mspabi_remul`。
- **L2960 EN**: Declares TableGen def `__mspabi_remull`.
  **L2960 CN**: 声明 TableGen def `__mspabi_remull`。
- **L2961 EN**: Blank line separating nearby declarations or logic blocks.
  **L2961 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2962 EN**: Comment explains nearby logic, invariants, or intent: `Bitwise Operations - EABI Table 10`.
  **L2962 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Bitwise Operations - EABI Table 10`。
- **L2963 EN**: Comment records a pending task or caution: `TODO: __mspabi_[srli/srai/slli] ARE implemented in libgcc`.
  **L2963 CN**: 注释记录了待办事项或注意点：`TODO: __mspabi_[srli/srai/slli] ARE implemented in libgcc`。
- **L2964 EN**: Declares TableGen def `__mspabi_srll`.
  **L2964 CN**: 声明 TableGen def `__mspabi_srll`。
- **L2965 EN**: Declares TableGen def `__mspabi_sral`.
  **L2965 CN**: 声明 TableGen def `__mspabi_sral`。
- **L2966 EN**: Declares TableGen def `__mspabi_slll`.
  **L2966 CN**: 声明 TableGen def `__mspabi_slll`。
- **L2967 EN**: Comment explains nearby logic, invariants, or intent: `__mspabi_[srlll/srall/sllll/rlli/rlll] are NOT implemented in libgcc`.
  **L2967 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`__mspabi_[srlll/srall/sllll/rlli/rlll] are NOT implemented in libgcc`。
- **L2968 EN**: Blank line separating nearby declarations or logic blocks.
  **L2968 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2969 EN**: Comment explains nearby logic, invariants, or intent: `hasHWMult16()`.
  **L2969 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`hasHWMult16()`。
- **L2970 EN**: Comment explains nearby logic, invariants, or intent: `Integer Multiply - EABI Table 9`.
  **L2970 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Integer Multiply - EABI Table 9`。
- **L2971 EN**: Comment explains nearby logic, invariants, or intent: `def __mspabi_mpyi_hw : RuntimeLibcallImpl<MUL_I16>;`.
  **L2971 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`def __mspabi_mpyi_hw : RuntimeLibcallImpl<MUL_I16>;`。
- **L2972 EN**: Declares TableGen def `__mspabi_mpyl_hw`.
  **L2972 CN**: 声明 TableGen def `__mspabi_mpyl_hw`。
- **L2973 EN**: Declares TableGen def `__mspabi_mpyll_hw`.
  **L2973 CN**: 声明 TableGen def `__mspabi_mpyll_hw`。
- **L2974 EN**: Comment records a pending task or caution: `TODO The __mspabi_mpysl*_hw functions ARE implemented in libgcc`.
  **L2974 CN**: 注释记录了待办事项或注意点：`TODO The __mspabi_mpysl*_hw functions ARE implemented in libgcc`。
- **L2975 EN**: Comment records a pending task or caution: `TODO The __mspabi_mpyul*_hw functions ARE implemented in libgcc`.
  **L2975 CN**: 注释记录了待办事项或注意点：`TODO The __mspabi_mpyul*_hw functions ARE implemented in libgcc`。
- **L2976 EN**: Blank line separating nearby declarations or logic blocks.
  **L2976 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 2977-3008

````tablegen
// hasHWMult32()
// Integer Multiply - EABI Table 9
def __mspabi_mpyi_hw : RuntimeLibcallImpl<MUL_I16>;
def __mspabi_mpyl_hw32 : RuntimeLibcallImpl<MUL_I32>;
def __mspabi_mpyll_hw32 : RuntimeLibcallImpl<MUL_I64>;
// TODO The __mspabi_mpysl*_hw32 functions ARE implemented in libgcc
// TODO The __mspabi_mpyul*_hw32 functions ARE implemented in libgcc

// hasHWMultF5()
// Integer Multiply - EABI Table 9
def __mspabi_mpyi_f5hw : RuntimeLibcallImpl<MUL_I16>;
def __mspabi_mpyl_f5hw : RuntimeLibcallImpl<MUL_I32>;
def __mspabi_mpyll_f5hw : RuntimeLibcallImpl<MUL_I64>;
// TODO The __mspabi_mpysl*_f5hw functions ARE implemented in libgcc
// TODO The __mspabi_mpyul*_f5hw functions ARE implemented in libgcc

// NoHWMult
// Integer Multiply - EABI Table 9
def __mspabi_mpyi : RuntimeLibcallImpl<MUL_I16>;
def __mspabi_mpyl : RuntimeLibcallImpl<MUL_I32>;
def __mspabi_mpyll : RuntimeLibcallImpl<MUL_I64>;
// The __mspabi_mpysl* functions are NOT implemented in libgcc
// The __mspabi_mpyul* functions are NOT implemented in libgcc

// setLibcallCallingConv(MUL_I64, CallingConv::MSP430_BUILTIN);

def isMSP430 : RuntimeLibcallPredicate<"TT.getArch() == Triple::msp430">;

defvar MSP430DefaultOptOut = [
  __addsf3, __divsf3, __extendsfdf2, __truncdfsf2, __fixsfsi,
  __fixsfdi, __fixunssfsi, __mulsf3, __eqsf2, __gesf2, __gtsf2,
  __divhi3, __divsi3, __ashlsi3, __floatsidf, __floatsisf,
````
- **L2977 EN**: Comment explains nearby logic, invariants, or intent: `hasHWMult32()`.
  **L2977 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`hasHWMult32()`。
- **L2978 EN**: Comment explains nearby logic, invariants, or intent: `Integer Multiply - EABI Table 9`.
  **L2978 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Integer Multiply - EABI Table 9`。
- **L2979 EN**: Declares TableGen def `__mspabi_mpyi_hw`.
  **L2979 CN**: 声明 TableGen def `__mspabi_mpyi_hw`。
- **L2980 EN**: Declares TableGen def `__mspabi_mpyl_hw32`.
  **L2980 CN**: 声明 TableGen def `__mspabi_mpyl_hw32`。
- **L2981 EN**: Declares TableGen def `__mspabi_mpyll_hw32`.
  **L2981 CN**: 声明 TableGen def `__mspabi_mpyll_hw32`。
- **L2982 EN**: Comment records a pending task or caution: `TODO The __mspabi_mpysl*_hw32 functions ARE implemented in libgcc`.
  **L2982 CN**: 注释记录了待办事项或注意点：`TODO The __mspabi_mpysl*_hw32 functions ARE implemented in libgcc`。
- **L2983 EN**: Comment records a pending task or caution: `TODO The __mspabi_mpyul*_hw32 functions ARE implemented in libgcc`.
  **L2983 CN**: 注释记录了待办事项或注意点：`TODO The __mspabi_mpyul*_hw32 functions ARE implemented in libgcc`。
- **L2984 EN**: Blank line separating nearby declarations or logic blocks.
  **L2984 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2985 EN**: Comment explains nearby logic, invariants, or intent: `hasHWMultF5()`.
  **L2985 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`hasHWMultF5()`。
- **L2986 EN**: Comment explains nearby logic, invariants, or intent: `Integer Multiply - EABI Table 9`.
  **L2986 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Integer Multiply - EABI Table 9`。
- **L2987 EN**: Declares TableGen def `__mspabi_mpyi_f5hw`.
  **L2987 CN**: 声明 TableGen def `__mspabi_mpyi_f5hw`。
- **L2988 EN**: Declares TableGen def `__mspabi_mpyl_f5hw`.
  **L2988 CN**: 声明 TableGen def `__mspabi_mpyl_f5hw`。
- **L2989 EN**: Declares TableGen def `__mspabi_mpyll_f5hw`.
  **L2989 CN**: 声明 TableGen def `__mspabi_mpyll_f5hw`。
- **L2990 EN**: Comment records a pending task or caution: `TODO The __mspabi_mpysl*_f5hw functions ARE implemented in libgcc`.
  **L2990 CN**: 注释记录了待办事项或注意点：`TODO The __mspabi_mpysl*_f5hw functions ARE implemented in libgcc`。
- **L2991 EN**: Comment records a pending task or caution: `TODO The __mspabi_mpyul*_f5hw functions ARE implemented in libgcc`.
  **L2991 CN**: 注释记录了待办事项或注意点：`TODO The __mspabi_mpyul*_f5hw functions ARE implemented in libgcc`。
- **L2992 EN**: Blank line separating nearby declarations or logic blocks.
  **L2992 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2993 EN**: Comment explains nearby logic, invariants, or intent: `NoHWMult`.
  **L2993 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`NoHWMult`。
- **L2994 EN**: Comment explains nearby logic, invariants, or intent: `Integer Multiply - EABI Table 9`.
  **L2994 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Integer Multiply - EABI Table 9`。
- **L2995 EN**: Declares TableGen def `__mspabi_mpyi`.
  **L2995 CN**: 声明 TableGen def `__mspabi_mpyi`。
- **L2996 EN**: Declares TableGen def `__mspabi_mpyl`.
  **L2996 CN**: 声明 TableGen def `__mspabi_mpyl`。
- **L2997 EN**: Declares TableGen def `__mspabi_mpyll`.
  **L2997 CN**: 声明 TableGen def `__mspabi_mpyll`。
- **L2998 EN**: Comment explains nearby logic, invariants, or intent: `The __mspabi_mpysl* functions are NOT implemented in libgcc`.
  **L2998 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`The __mspabi_mpysl* functions are NOT implemented in libgcc`。
- **L2999 EN**: Comment explains nearby logic, invariants, or intent: `The __mspabi_mpyul* functions are NOT implemented in libgcc`.
  **L2999 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`The __mspabi_mpyul* functions are NOT implemented in libgcc`。
- **L3000 EN**: Blank line separating nearby declarations or logic blocks.
  **L3000 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3001 EN**: Comment explains nearby logic, invariants, or intent: `setLibcallCallingConv(MUL_I64, CallingConv::MSP430_BUILTIN);`.
  **L3001 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`setLibcallCallingConv(MUL_I64, CallingConv::MSP430_BUILTIN);`。
- **L3002 EN**: Blank line separating nearby declarations or logic blocks.
  **L3002 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3003 EN**: Declares TableGen def `isMSP430`.
  **L3003 CN**: 声明 TableGen def `isMSP430`。
- **L3004 EN**: Blank line separating nearby declarations or logic blocks.
  **L3004 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3005 EN**: Continues the surrounding expression or declaration: `defvar MSP430DefaultOptOut = [`.
  **L3005 CN**: 继续构造周围的表达式或声明：`defvar MSP430DefaultOptOut = [`。
- **L3006 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__addsf3, __divsf3, __extendsfdf2, __truncdfsf2, __fixsfsi,`.
  **L3006 CN**: 继续一个多行参数列表、初始化器或聚合项：`__addsf3, __divsf3, __extendsfdf2, __truncdfsf2, __fixsfsi,`。
- **L3007 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__fixsfdi, __fixunssfsi, __mulsf3, __eqsf2, __gesf2, __gtsf2,`.
  **L3007 CN**: 继续一个多行参数列表、初始化器或聚合项：`__fixsfdi, __fixunssfsi, __mulsf3, __eqsf2, __gesf2, __gtsf2,`。
- **L3008 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__divhi3, __divsi3, __ashlsi3, __floatsidf, __floatsisf,`.
  **L3008 CN**: 继续一个多行参数列表、初始化器或聚合项：`__divhi3, __divsi3, __ashlsi3, __floatsidf, __floatsisf,`。

### Lines 3009-3040

````tablegen
  __ashrsi3, __modhi3, __udivsi3, __fixdfsi, __fixunssfdi,
  __udivhi3, __umodsi3, __nesf2, __lesf2, __floatundisf,
  __fixdfdi, __fixunsdfsi, __modsi3, __floatunsisf,
  __fixunsdfdi, __ltsf2, __floatdisf, __floatdidf,
  __lshrsi3, __subsf3, __umodhi3, __floatunsidf,
  __floatundidf, __gtdf2, __eqdf2, __gedf2, __ltdf2, __ledf2,
  __adddf3, __divdf3, __divdi3, __moddi3,
  __muldf3, __subdf3, __udivdi3, __umoddi3
];

// EABI Libcalls - EABI Section 6.2
def MSP430SystemLibrary
    : SystemRuntimeLibrary<isMSP430,
    (add (sub DefaultRuntimeLibcallImpls, MSP430DefaultOptOut),
      exp10f, exp10, exp10l_f128,

      // Floating point conversions - EABI Table 6
      __mspabi_cvtdf,
      __mspabi_cvtfd,
      // The following is NOT implemented in libgcc
      //__mspabi_fixdi,
      __mspabi_fixdli,
      __mspabi_fixdlli,
      // The following is NOT implemented in libgcc
      // __mspabi_fixdu
      __mspabi_fixdul,
      __mspabi_fixdull,
      // The following is NOT implemented in libgcc
      //__mspabi_fixfi,
      __mspabi_fixfli,
      __mspabi_fixflli,
      // The following is NOT implemented in libgcc
````
- **L3009 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__ashrsi3, __modhi3, __udivsi3, __fixdfsi, __fixunssfdi,`.
  **L3009 CN**: 继续一个多行参数列表、初始化器或聚合项：`__ashrsi3, __modhi3, __udivsi3, __fixdfsi, __fixunssfdi,`。
- **L3010 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__udivhi3, __umodsi3, __nesf2, __lesf2, __floatundisf,`.
  **L3010 CN**: 继续一个多行参数列表、初始化器或聚合项：`__udivhi3, __umodsi3, __nesf2, __lesf2, __floatundisf,`。
- **L3011 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__fixdfdi, __fixunsdfsi, __modsi3, __floatunsisf,`.
  **L3011 CN**: 继续一个多行参数列表、初始化器或聚合项：`__fixdfdi, __fixunsdfsi, __modsi3, __floatunsisf,`。
- **L3012 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__fixunsdfdi, __ltsf2, __floatdisf, __floatdidf,`.
  **L3012 CN**: 继续一个多行参数列表、初始化器或聚合项：`__fixunsdfdi, __ltsf2, __floatdisf, __floatdidf,`。
- **L3013 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__lshrsi3, __subsf3, __umodhi3, __floatunsidf,`.
  **L3013 CN**: 继续一个多行参数列表、初始化器或聚合项：`__lshrsi3, __subsf3, __umodhi3, __floatunsidf,`。
- **L3014 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__floatundidf, __gtdf2, __eqdf2, __gedf2, __ltdf2, __ledf2,`.
  **L3014 CN**: 继续一个多行参数列表、初始化器或聚合项：`__floatundidf, __gtdf2, __eqdf2, __gedf2, __ltdf2, __ledf2,`。
- **L3015 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__adddf3, __divdf3, __divdi3, __moddi3,`.
  **L3015 CN**: 继续一个多行参数列表、初始化器或聚合项：`__adddf3, __divdf3, __divdi3, __moddi3,`。
- **L3016 EN**: Continues the surrounding expression or declaration: `__muldf3, __subdf3, __udivdi3, __umoddi3`.
  **L3016 CN**: 继续构造周围的表达式或声明：`__muldf3, __subdf3, __udivdi3, __umoddi3`。
- **L3017 EN**: Executes a standalone statement or declaration: `];`.
  **L3017 CN**: 执行一条独立语句或声明：`];`。
- **L3018 EN**: Blank line separating nearby declarations or logic blocks.
  **L3018 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3019 EN**: Comment explains nearby logic, invariants, or intent: `EABI Libcalls - EABI Section 6.2`.
  **L3019 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`EABI Libcalls - EABI Section 6.2`。
- **L3020 EN**: Declares TableGen def `MSP430SystemLibrary`.
  **L3020 CN**: 声明 TableGen def `MSP430SystemLibrary`。
- **L3021 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: SystemRuntimeLibrary<isMSP430,`.
  **L3021 CN**: 继续一个多行参数列表、初始化器或聚合项：`: SystemRuntimeLibrary<isMSP430,`。
- **L3022 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(add (sub DefaultRuntimeLibcallImpls, MSP430DefaultOptOut),`.
  **L3022 CN**: 继续一个多行参数列表、初始化器或聚合项：`(add (sub DefaultRuntimeLibcallImpls, MSP430DefaultOptOut),`。
- **L3023 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `exp10f, exp10, exp10l_f128,`.
  **L3023 CN**: 继续一个多行参数列表、初始化器或聚合项：`exp10f, exp10, exp10l_f128,`。
- **L3024 EN**: Blank line separating nearby declarations or logic blocks.
  **L3024 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3025 EN**: Comment explains nearby logic, invariants, or intent: `Floating point conversions - EABI Table 6`.
  **L3025 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Floating point conversions - EABI Table 6`。
- **L3026 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__mspabi_cvtdf,`.
  **L3026 CN**: 继续一个多行参数列表、初始化器或聚合项：`__mspabi_cvtdf,`。
- **L3027 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__mspabi_cvtfd,`.
  **L3027 CN**: 继续一个多行参数列表、初始化器或聚合项：`__mspabi_cvtfd,`。
- **L3028 EN**: Comment explains nearby logic, invariants, or intent: `The following is NOT implemented in libgcc`.
  **L3028 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`The following is NOT implemented in libgcc`。
- **L3029 EN**: Comment explains nearby logic, invariants, or intent: `__mspabi_fixdi,`.
  **L3029 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`__mspabi_fixdi,`。
- **L3030 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__mspabi_fixdli,`.
  **L3030 CN**: 继续一个多行参数列表、初始化器或聚合项：`__mspabi_fixdli,`。
- **L3031 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__mspabi_fixdlli,`.
  **L3031 CN**: 继续一个多行参数列表、初始化器或聚合项：`__mspabi_fixdlli,`。
- **L3032 EN**: Comment explains nearby logic, invariants, or intent: `The following is NOT implemented in libgcc`.
  **L3032 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`The following is NOT implemented in libgcc`。
- **L3033 EN**: Comment explains nearby logic, invariants, or intent: `__mspabi_fixdu`.
  **L3033 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`__mspabi_fixdu`。
- **L3034 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__mspabi_fixdul,`.
  **L3034 CN**: 继续一个多行参数列表、初始化器或聚合项：`__mspabi_fixdul,`。
- **L3035 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__mspabi_fixdull,`.
  **L3035 CN**: 继续一个多行参数列表、初始化器或聚合项：`__mspabi_fixdull,`。
- **L3036 EN**: Comment explains nearby logic, invariants, or intent: `The following is NOT implemented in libgcc`.
  **L3036 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`The following is NOT implemented in libgcc`。
- **L3037 EN**: Comment explains nearby logic, invariants, or intent: `__mspabi_fixfi,`.
  **L3037 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`__mspabi_fixfi,`。
- **L3038 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__mspabi_fixfli,`.
  **L3038 CN**: 继续一个多行参数列表、初始化器或聚合项：`__mspabi_fixfli,`。
- **L3039 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__mspabi_fixflli,`.
  **L3039 CN**: 继续一个多行参数列表、初始化器或聚合项：`__mspabi_fixflli,`。
- **L3040 EN**: Comment explains nearby logic, invariants, or intent: `The following is NOT implemented in libgcc`.
  **L3040 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`The following is NOT implemented in libgcc`。

### Lines 3041-3072

````tablegen
      //__mspabi_fixfu,
      __mspabi_fixful,
      __mspabi_fixfull,
      // TODO The following IS implemented in libgcc
      //__mspabi_fltid
      __mspabi_fltlid,
      // TODO The following IS implemented in libgcc but is not in the EABI
      __mspabi_fltllid,
      // TODO The following IS implemented in libgcc
      //__mspabi_fltud,
      __mspabi_fltuld,
      // The following IS implemented in libgcc but is not in the EABI
      __mspabi_fltulld,
      // TODO The following IS implemented in libgcc
      //__mspabi_fltif,
      __mspabi_fltlif,
      // TODO The following IS implemented in libgcc but is not in the EABI
      __mspabi_fltllif,
      // TODO The following IS implemented in libgcc
      //__mspabi_fltuf,
      __mspabi_fltulf,
      // The following IS implemented in libgcc but is not in the EABI
      __mspabi_fltullf,

      // Floating point comparisons - EABI Table 7
      LibcallsWithCC<(add __mspabi_cmpd__oeq,
                          __mspabi_cmpd__une,
                          __mspabi_cmpd__oge,
                          __mspabi_cmpd__olt,
                          __mspabi_cmpd__ole,
                          __mspabi_cmpd__ogt), MSP430_BUILTIN>,
      __mspabi_cmpf__oeq,
````
- **L3041 EN**: Comment explains nearby logic, invariants, or intent: `__mspabi_fixfu,`.
  **L3041 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`__mspabi_fixfu,`。
- **L3042 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__mspabi_fixful,`.
  **L3042 CN**: 继续一个多行参数列表、初始化器或聚合项：`__mspabi_fixful,`。
- **L3043 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__mspabi_fixfull,`.
  **L3043 CN**: 继续一个多行参数列表、初始化器或聚合项：`__mspabi_fixfull,`。
- **L3044 EN**: Comment records a pending task or caution: `TODO The following IS implemented in libgcc`.
  **L3044 CN**: 注释记录了待办事项或注意点：`TODO The following IS implemented in libgcc`。
- **L3045 EN**: Comment explains nearby logic, invariants, or intent: `__mspabi_fltid`.
  **L3045 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`__mspabi_fltid`。
- **L3046 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__mspabi_fltlid,`.
  **L3046 CN**: 继续一个多行参数列表、初始化器或聚合项：`__mspabi_fltlid,`。
- **L3047 EN**: Comment records a pending task or caution: `TODO The following IS implemented in libgcc but is not in the EABI`.
  **L3047 CN**: 注释记录了待办事项或注意点：`TODO The following IS implemented in libgcc but is not in the EABI`。
- **L3048 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__mspabi_fltllid,`.
  **L3048 CN**: 继续一个多行参数列表、初始化器或聚合项：`__mspabi_fltllid,`。
- **L3049 EN**: Comment records a pending task or caution: `TODO The following IS implemented in libgcc`.
  **L3049 CN**: 注释记录了待办事项或注意点：`TODO The following IS implemented in libgcc`。
- **L3050 EN**: Comment explains nearby logic, invariants, or intent: `__mspabi_fltud,`.
  **L3050 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`__mspabi_fltud,`。
- **L3051 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__mspabi_fltuld,`.
  **L3051 CN**: 继续一个多行参数列表、初始化器或聚合项：`__mspabi_fltuld,`。
- **L3052 EN**: Comment explains nearby logic, invariants, or intent: `The following IS implemented in libgcc but is not in the EABI`.
  **L3052 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`The following IS implemented in libgcc but is not in the EABI`。
- **L3053 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__mspabi_fltulld,`.
  **L3053 CN**: 继续一个多行参数列表、初始化器或聚合项：`__mspabi_fltulld,`。
- **L3054 EN**: Comment records a pending task or caution: `TODO The following IS implemented in libgcc`.
  **L3054 CN**: 注释记录了待办事项或注意点：`TODO The following IS implemented in libgcc`。
- **L3055 EN**: Comment explains nearby logic, invariants, or intent: `__mspabi_fltif,`.
  **L3055 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`__mspabi_fltif,`。
- **L3056 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__mspabi_fltlif,`.
  **L3056 CN**: 继续一个多行参数列表、初始化器或聚合项：`__mspabi_fltlif,`。
- **L3057 EN**: Comment records a pending task or caution: `TODO The following IS implemented in libgcc but is not in the EABI`.
  **L3057 CN**: 注释记录了待办事项或注意点：`TODO The following IS implemented in libgcc but is not in the EABI`。
- **L3058 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__mspabi_fltllif,`.
  **L3058 CN**: 继续一个多行参数列表、初始化器或聚合项：`__mspabi_fltllif,`。
- **L3059 EN**: Comment records a pending task or caution: `TODO The following IS implemented in libgcc`.
  **L3059 CN**: 注释记录了待办事项或注意点：`TODO The following IS implemented in libgcc`。
- **L3060 EN**: Comment explains nearby logic, invariants, or intent: `__mspabi_fltuf,`.
  **L3060 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`__mspabi_fltuf,`。
- **L3061 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__mspabi_fltulf,`.
  **L3061 CN**: 继续一个多行参数列表、初始化器或聚合项：`__mspabi_fltulf,`。
- **L3062 EN**: Comment explains nearby logic, invariants, or intent: `The following IS implemented in libgcc but is not in the EABI`.
  **L3062 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`The following IS implemented in libgcc but is not in the EABI`。
- **L3063 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__mspabi_fltullf,`.
  **L3063 CN**: 继续一个多行参数列表、初始化器或聚合项：`__mspabi_fltullf,`。
- **L3064 EN**: Blank line separating nearby declarations or logic blocks.
  **L3064 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3065 EN**: Comment explains nearby logic, invariants, or intent: `Floating point comparisons - EABI Table 7`.
  **L3065 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Floating point comparisons - EABI Table 7`。
- **L3066 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LibcallsWithCC<(add __mspabi_cmpd__oeq,`.
  **L3066 CN**: 继续一个多行参数列表、初始化器或聚合项：`LibcallsWithCC<(add __mspabi_cmpd__oeq,`。
- **L3067 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__mspabi_cmpd__une,`.
  **L3067 CN**: 继续一个多行参数列表、初始化器或聚合项：`__mspabi_cmpd__une,`。
- **L3068 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__mspabi_cmpd__oge,`.
  **L3068 CN**: 继续一个多行参数列表、初始化器或聚合项：`__mspabi_cmpd__oge,`。
- **L3069 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__mspabi_cmpd__olt,`.
  **L3069 CN**: 继续一个多行参数列表、初始化器或聚合项：`__mspabi_cmpd__olt,`。
- **L3070 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__mspabi_cmpd__ole,`.
  **L3070 CN**: 继续一个多行参数列表、初始化器或聚合项：`__mspabi_cmpd__ole,`。
- **L3071 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__mspabi_cmpd__ogt), MSP430_BUILTIN>,`.
  **L3071 CN**: 继续一个多行参数列表、初始化器或聚合项：`__mspabi_cmpd__ogt), MSP430_BUILTIN>,`。
- **L3072 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__mspabi_cmpf__oeq,`.
  **L3072 CN**: 继续一个多行参数列表、初始化器或聚合项：`__mspabi_cmpf__oeq,`。

### Lines 3073-3104

````tablegen
      __mspabi_cmpf__une,
      __mspabi_cmpf__oge,
      __mspabi_cmpf__olt,
      __mspabi_cmpf__ole,
      __mspabi_cmpf__ogt,

      // Floating point arithmetic - EABI Table 8
      LibcallsWithCC<(add __mspabi_addd,
      __mspabi_subd,
      __mspabi_mpyd,
      __mspabi_divd), MSP430_BUILTIN>,

      __mspabi_addf,
      __mspabi_subf,
      __mspabi_mpyf,
      __mspabi_divf,

      // The following are NOT implemented in libgcc
      // __mspabi_negd,
      // __mspabi_negf,

      // Universal Integer Operations - EABI Table 9
      __mspabi_divi,
      __mspabi_divli,
      LibcallsWithCC<(add __mspabi_divlli), MSP430_BUILTIN>,
      __mspabi_divu,
      __mspabi_divul,
      LibcallsWithCC<(add __mspabi_divull), MSP430_BUILTIN>,
      __mspabi_remi,
      __mspabi_remli,
      LibcallsWithCC<(add __mspabi_remlli), MSP430_BUILTIN>,
      __mspabi_remu,
````
- **L3073 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__mspabi_cmpf__une,`.
  **L3073 CN**: 继续一个多行参数列表、初始化器或聚合项：`__mspabi_cmpf__une,`。
- **L3074 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__mspabi_cmpf__oge,`.
  **L3074 CN**: 继续一个多行参数列表、初始化器或聚合项：`__mspabi_cmpf__oge,`。
- **L3075 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__mspabi_cmpf__olt,`.
  **L3075 CN**: 继续一个多行参数列表、初始化器或聚合项：`__mspabi_cmpf__olt,`。
- **L3076 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__mspabi_cmpf__ole,`.
  **L3076 CN**: 继续一个多行参数列表、初始化器或聚合项：`__mspabi_cmpf__ole,`。
- **L3077 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__mspabi_cmpf__ogt,`.
  **L3077 CN**: 继续一个多行参数列表、初始化器或聚合项：`__mspabi_cmpf__ogt,`。
- **L3078 EN**: Blank line separating nearby declarations or logic blocks.
  **L3078 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3079 EN**: Comment explains nearby logic, invariants, or intent: `Floating point arithmetic - EABI Table 8`.
  **L3079 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Floating point arithmetic - EABI Table 8`。
- **L3080 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LibcallsWithCC<(add __mspabi_addd,`.
  **L3080 CN**: 继续一个多行参数列表、初始化器或聚合项：`LibcallsWithCC<(add __mspabi_addd,`。
- **L3081 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__mspabi_subd,`.
  **L3081 CN**: 继续一个多行参数列表、初始化器或聚合项：`__mspabi_subd,`。
- **L3082 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__mspabi_mpyd,`.
  **L3082 CN**: 继续一个多行参数列表、初始化器或聚合项：`__mspabi_mpyd,`。
- **L3083 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__mspabi_divd), MSP430_BUILTIN>,`.
  **L3083 CN**: 继续一个多行参数列表、初始化器或聚合项：`__mspabi_divd), MSP430_BUILTIN>,`。
- **L3084 EN**: Blank line separating nearby declarations or logic blocks.
  **L3084 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3085 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__mspabi_addf,`.
  **L3085 CN**: 继续一个多行参数列表、初始化器或聚合项：`__mspabi_addf,`。
- **L3086 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__mspabi_subf,`.
  **L3086 CN**: 继续一个多行参数列表、初始化器或聚合项：`__mspabi_subf,`。
- **L3087 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__mspabi_mpyf,`.
  **L3087 CN**: 继续一个多行参数列表、初始化器或聚合项：`__mspabi_mpyf,`。
- **L3088 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__mspabi_divf,`.
  **L3088 CN**: 继续一个多行参数列表、初始化器或聚合项：`__mspabi_divf,`。
- **L3089 EN**: Blank line separating nearby declarations or logic blocks.
  **L3089 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3090 EN**: Comment explains nearby logic, invariants, or intent: `The following are NOT implemented in libgcc`.
  **L3090 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`The following are NOT implemented in libgcc`。
- **L3091 EN**: Comment explains nearby logic, invariants, or intent: `__mspabi_negd,`.
  **L3091 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`__mspabi_negd,`。
- **L3092 EN**: Comment explains nearby logic, invariants, or intent: `__mspabi_negf,`.
  **L3092 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`__mspabi_negf,`。
- **L3093 EN**: Blank line separating nearby declarations or logic blocks.
  **L3093 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3094 EN**: Comment explains nearby logic, invariants, or intent: `Universal Integer Operations - EABI Table 9`.
  **L3094 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Universal Integer Operations - EABI Table 9`。
- **L3095 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__mspabi_divi,`.
  **L3095 CN**: 继续一个多行参数列表、初始化器或聚合项：`__mspabi_divi,`。
- **L3096 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__mspabi_divli,`.
  **L3096 CN**: 继续一个多行参数列表、初始化器或聚合项：`__mspabi_divli,`。
- **L3097 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LibcallsWithCC<(add __mspabi_divlli), MSP430_BUILTIN>,`.
  **L3097 CN**: 继续一个多行参数列表、初始化器或聚合项：`LibcallsWithCC<(add __mspabi_divlli), MSP430_BUILTIN>,`。
- **L3098 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__mspabi_divu,`.
  **L3098 CN**: 继续一个多行参数列表、初始化器或聚合项：`__mspabi_divu,`。
- **L3099 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__mspabi_divul,`.
  **L3099 CN**: 继续一个多行参数列表、初始化器或聚合项：`__mspabi_divul,`。
- **L3100 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LibcallsWithCC<(add __mspabi_divull), MSP430_BUILTIN>,`.
  **L3100 CN**: 继续一个多行参数列表、初始化器或聚合项：`LibcallsWithCC<(add __mspabi_divull), MSP430_BUILTIN>,`。
- **L3101 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__mspabi_remi,`.
  **L3101 CN**: 继续一个多行参数列表、初始化器或聚合项：`__mspabi_remi,`。
- **L3102 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__mspabi_remli,`.
  **L3102 CN**: 继续一个多行参数列表、初始化器或聚合项：`__mspabi_remli,`。
- **L3103 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LibcallsWithCC<(add __mspabi_remlli), MSP430_BUILTIN>,`.
  **L3103 CN**: 继续一个多行参数列表、初始化器或聚合项：`LibcallsWithCC<(add __mspabi_remlli), MSP430_BUILTIN>,`。
- **L3104 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__mspabi_remu,`.
  **L3104 CN**: 继续一个多行参数列表、初始化器或聚合项：`__mspabi_remu,`。

### Lines 3105-3136

````tablegen
      __mspabi_remul,
      LibcallsWithCC<(add __mspabi_remull), MSP430_BUILTIN>,

      // May not be used depending on subtarget
      LibcallsWithCC<(add __mspabi_mpyll), MSP430_BUILTIN>,

      // Bitwise Operations - EABI Table 10
      // TODO: __mspabi_[srli/srai/slli] ARE implemented in libgcc
      __mspabi_srll,
      __mspabi_sral,
      __mspabi_slll,
      // __mspabi_[srlll/srall/sllll/rlli/rlll] are NOT implemented in libgcc

      __stack_chk_fail,
      __stack_chk_guard
  )
>;

//===----------------------------------------------------------------------===//
// NVPTX Runtime Libcalls
//===----------------------------------------------------------------------===//

def isNVPTX : RuntimeLibcallPredicate<"TT.isNVPTX()">;

// No calls.
def NVPTXSystemLibrary : SystemRuntimeLibrary<isNVPTX, (add)>;

//===----------------------------------------------------------------------===//
// PPC Runtime Libcalls
//===----------------------------------------------------------------------===//

// For IEEE quad-precision libcall names, PPC uses "kf" instead of "tf".
````
- **L3105 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__mspabi_remul,`.
  **L3105 CN**: 继续一个多行参数列表、初始化器或聚合项：`__mspabi_remul,`。
- **L3106 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LibcallsWithCC<(add __mspabi_remull), MSP430_BUILTIN>,`.
  **L3106 CN**: 继续一个多行参数列表、初始化器或聚合项：`LibcallsWithCC<(add __mspabi_remull), MSP430_BUILTIN>,`。
- **L3107 EN**: Blank line separating nearby declarations or logic blocks.
  **L3107 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3108 EN**: Comment explains nearby logic, invariants, or intent: `May not be used depending on subtarget`.
  **L3108 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`May not be used depending on subtarget`。
- **L3109 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LibcallsWithCC<(add __mspabi_mpyll), MSP430_BUILTIN>,`.
  **L3109 CN**: 继续一个多行参数列表、初始化器或聚合项：`LibcallsWithCC<(add __mspabi_mpyll), MSP430_BUILTIN>,`。
- **L3110 EN**: Blank line separating nearby declarations or logic blocks.
  **L3110 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3111 EN**: Comment explains nearby logic, invariants, or intent: `Bitwise Operations - EABI Table 10`.
  **L3111 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Bitwise Operations - EABI Table 10`。
- **L3112 EN**: Comment records a pending task or caution: `TODO: __mspabi_[srli/srai/slli] ARE implemented in libgcc`.
  **L3112 CN**: 注释记录了待办事项或注意点：`TODO: __mspabi_[srli/srai/slli] ARE implemented in libgcc`。
- **L3113 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__mspabi_srll,`.
  **L3113 CN**: 继续一个多行参数列表、初始化器或聚合项：`__mspabi_srll,`。
- **L3114 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__mspabi_sral,`.
  **L3114 CN**: 继续一个多行参数列表、初始化器或聚合项：`__mspabi_sral,`。
- **L3115 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__mspabi_slll,`.
  **L3115 CN**: 继续一个多行参数列表、初始化器或聚合项：`__mspabi_slll,`。
- **L3116 EN**: Comment explains nearby logic, invariants, or intent: `__mspabi_[srlll/srall/sllll/rlli/rlll] are NOT implemented in libgcc`.
  **L3116 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`__mspabi_[srlll/srall/sllll/rlli/rlll] are NOT implemented in libgcc`。
- **L3117 EN**: Blank line separating nearby declarations or logic blocks.
  **L3117 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3118 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__stack_chk_fail,`.
  **L3118 CN**: 继续一个多行参数列表、初始化器或聚合项：`__stack_chk_fail,`。
- **L3119 EN**: Continues the surrounding expression or declaration: `__stack_chk_guard`.
  **L3119 CN**: 继续构造周围的表达式或声明：`__stack_chk_guard`。
- **L3120 EN**: Continues the surrounding expression or declaration: `)`.
  **L3120 CN**: 继续构造周围的表达式或声明：`)`。
- **L3121 EN**: Executes a standalone statement or declaration: `>;`.
  **L3121 CN**: 执行一条独立语句或声明：`>;`。
- **L3122 EN**: Blank line separating nearby declarations or logic blocks.
  **L3122 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3123 EN**: Banner comment marking a file or section boundary.
  **L3123 CN**: 横幅注释，用于标记文件或章节边界。
- **L3124 EN**: Comment explains nearby logic, invariants, or intent: `NVPTX Runtime Libcalls`.
  **L3124 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`NVPTX Runtime Libcalls`。
- **L3125 EN**: Banner comment marking a file or section boundary.
  **L3125 CN**: 横幅注释，用于标记文件或章节边界。
- **L3126 EN**: Blank line separating nearby declarations or logic blocks.
  **L3126 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3127 EN**: Declares TableGen def `isNVPTX`.
  **L3127 CN**: 声明 TableGen def `isNVPTX`。
- **L3128 EN**: Blank line separating nearby declarations or logic blocks.
  **L3128 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3129 EN**: Comment explains nearby logic, invariants, or intent: `No calls.`.
  **L3129 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`No calls.`。
- **L3130 EN**: Declares TableGen def `NVPTXSystemLibrary`.
  **L3130 CN**: 声明 TableGen def `NVPTXSystemLibrary`。
- **L3131 EN**: Blank line separating nearby declarations or logic blocks.
  **L3131 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3132 EN**: Banner comment marking a file or section boundary.
  **L3132 CN**: 横幅注释，用于标记文件或章节边界。
- **L3133 EN**: Comment explains nearby logic, invariants, or intent: `PPC Runtime Libcalls`.
  **L3133 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`PPC Runtime Libcalls`。
- **L3134 EN**: Banner comment marking a file or section boundary.
  **L3134 CN**: 横幅注释，用于标记文件或章节边界。
- **L3135 EN**: Blank line separating nearby declarations or logic blocks.
  **L3135 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3136 EN**: Comment explains nearby logic, invariants, or intent: `For IEEE quad-precision libcall names, PPC uses "kf" instead of "tf".`.
  **L3136 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`For IEEE quad-precision libcall names, PPC uses "kf" instead of "tf".`。

### Lines 3137-3168

````tablegen
defset list<RuntimeLibcallImpl> PPCRuntimeLibcalls = {
  def __addkf3 : RuntimeLibcallImpl<ADD_F128>;
  def __subkf3 : RuntimeLibcallImpl<SUB_F128>;
  def __mulkf3 : RuntimeLibcallImpl<MUL_F128>;
  def __divkf3 : RuntimeLibcallImpl<DIV_F128>;
  def __powikf2 : RuntimeLibcallImpl<POWI_F128>;
  def __extendsfkf2 : RuntimeLibcallImpl<FPEXT_F32_F128>;
  def __extenddfkf2 : RuntimeLibcallImpl<FPEXT_F64_F128>;
  def __trunckfhf2 : RuntimeLibcallImpl<FPROUND_F128_F16>;
  def __trunckfsf2 : RuntimeLibcallImpl<FPROUND_F128_F32>;
  def __trunckfdf2 : RuntimeLibcallImpl<FPROUND_F128_F64>;
  def __fixkfsi : RuntimeLibcallImpl<FPTOSINT_F128_I32>;
  def __fixkfdi : RuntimeLibcallImpl<FPTOSINT_F128_I64>;
  def __fixkfti : RuntimeLibcallImpl<FPTOSINT_F128_I128>;
  def __fixunskfsi : RuntimeLibcallImpl<FPTOUINT_F128_I32>;
  def __fixunskfdi : RuntimeLibcallImpl<FPTOUINT_F128_I64>;
  def __fixunskfti : RuntimeLibcallImpl<FPTOUINT_F128_I128>;
  def __floatsikf : RuntimeLibcallImpl<SINTTOFP_I32_F128>;
  def __floatdikf : RuntimeLibcallImpl<SINTTOFP_I64_F128>;
  def __floattikf : RuntimeLibcallImpl<SINTTOFP_I128_F128>;
  def __floatunsikf : RuntimeLibcallImpl<UINTTOFP_I32_F128>;
  def __floatundikf : RuntimeLibcallImpl<UINTTOFP_I64_F128>;
  def __floatuntikf : RuntimeLibcallImpl<UINTTOFP_I128_F128>;
  def __eqkf2 : RuntimeLibcallImpl<OEQ_F128>;
  def __nekf2 : RuntimeLibcallImpl<UNE_F128>;
  def __gekf2 : RuntimeLibcallImpl<OGE_F128>;
  def __ltkf2 : RuntimeLibcallImpl<OLT_F128>;
  def __lekf2 : RuntimeLibcallImpl<OLE_F128>;
  def __gtkf2 : RuntimeLibcallImpl<OGT_F128>;
  def __unordkf2 : RuntimeLibcallImpl<UO_F128>;
}

````
- **L3137 EN**: Continues the surrounding expression or declaration: `defset list<RuntimeLibcallImpl> PPCRuntimeLibcalls = {`.
  **L3137 CN**: 继续构造周围的表达式或声明：`defset list<RuntimeLibcallImpl> PPCRuntimeLibcalls = {`。
- **L3138 EN**: Declares TableGen def `__addkf3`.
  **L3138 CN**: 声明 TableGen def `__addkf3`。
- **L3139 EN**: Declares TableGen def `__subkf3`.
  **L3139 CN**: 声明 TableGen def `__subkf3`。
- **L3140 EN**: Declares TableGen def `__mulkf3`.
  **L3140 CN**: 声明 TableGen def `__mulkf3`。
- **L3141 EN**: Declares TableGen def `__divkf3`.
  **L3141 CN**: 声明 TableGen def `__divkf3`。
- **L3142 EN**: Declares TableGen def `__powikf2`.
  **L3142 CN**: 声明 TableGen def `__powikf2`。
- **L3143 EN**: Declares TableGen def `__extendsfkf2`.
  **L3143 CN**: 声明 TableGen def `__extendsfkf2`。
- **L3144 EN**: Declares TableGen def `__extenddfkf2`.
  **L3144 CN**: 声明 TableGen def `__extenddfkf2`。
- **L3145 EN**: Declares TableGen def `__trunckfhf2`.
  **L3145 CN**: 声明 TableGen def `__trunckfhf2`。
- **L3146 EN**: Declares TableGen def `__trunckfsf2`.
  **L3146 CN**: 声明 TableGen def `__trunckfsf2`。
- **L3147 EN**: Declares TableGen def `__trunckfdf2`.
  **L3147 CN**: 声明 TableGen def `__trunckfdf2`。
- **L3148 EN**: Declares TableGen def `__fixkfsi`.
  **L3148 CN**: 声明 TableGen def `__fixkfsi`。
- **L3149 EN**: Declares TableGen def `__fixkfdi`.
  **L3149 CN**: 声明 TableGen def `__fixkfdi`。
- **L3150 EN**: Declares TableGen def `__fixkfti`.
  **L3150 CN**: 声明 TableGen def `__fixkfti`。
- **L3151 EN**: Declares TableGen def `__fixunskfsi`.
  **L3151 CN**: 声明 TableGen def `__fixunskfsi`。
- **L3152 EN**: Declares TableGen def `__fixunskfdi`.
  **L3152 CN**: 声明 TableGen def `__fixunskfdi`。
- **L3153 EN**: Declares TableGen def `__fixunskfti`.
  **L3153 CN**: 声明 TableGen def `__fixunskfti`。
- **L3154 EN**: Declares TableGen def `__floatsikf`.
  **L3154 CN**: 声明 TableGen def `__floatsikf`。
- **L3155 EN**: Declares TableGen def `__floatdikf`.
  **L3155 CN**: 声明 TableGen def `__floatdikf`。
- **L3156 EN**: Declares TableGen def `__floattikf`.
  **L3156 CN**: 声明 TableGen def `__floattikf`。
- **L3157 EN**: Declares TableGen def `__floatunsikf`.
  **L3157 CN**: 声明 TableGen def `__floatunsikf`。
- **L3158 EN**: Declares TableGen def `__floatundikf`.
  **L3158 CN**: 声明 TableGen def `__floatundikf`。
- **L3159 EN**: Declares TableGen def `__floatuntikf`.
  **L3159 CN**: 声明 TableGen def `__floatuntikf`。
- **L3160 EN**: Declares TableGen def `__eqkf2`.
  **L3160 CN**: 声明 TableGen def `__eqkf2`。
- **L3161 EN**: Declares TableGen def `__nekf2`.
  **L3161 CN**: 声明 TableGen def `__nekf2`。
- **L3162 EN**: Declares TableGen def `__gekf2`.
  **L3162 CN**: 声明 TableGen def `__gekf2`。
- **L3163 EN**: Declares TableGen def `__ltkf2`.
  **L3163 CN**: 声明 TableGen def `__ltkf2`。
- **L3164 EN**: Declares TableGen def `__lekf2`.
  **L3164 CN**: 声明 TableGen def `__lekf2`。
- **L3165 EN**: Declares TableGen def `__gtkf2`.
  **L3165 CN**: 声明 TableGen def `__gtkf2`。
- **L3166 EN**: Declares TableGen def `__unordkf2`.
  **L3166 CN**: 声明 TableGen def `__unordkf2`。
- **L3167 EN**: Closes the current lexical scope or compound statement.
  **L3167 CN**: 结束当前词法作用域或复合语句块。
- **L3168 EN**: Blank line separating nearby declarations or logic blocks.
  **L3168 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 3169-3200

````tablegen
defset list<RuntimeLibcallImpl> PPC64AIXCallList = {
  def ___memccpy64 : RuntimeLibcallImpl<MEMCCPY>;
  def ___memcmp64 : RuntimeLibcallImpl<MEMCMP>;
  def ___memmove64 : RuntimeLibcallImpl<MEMMOVE>;
  def ___memset64 : RuntimeLibcallImpl<MEMSET>;
  def ___bzero64 : RuntimeLibcallImpl<BZERO>;
  def ___strcmp64 : RuntimeLibcallImpl<STRCMP>;
  def ___strcpy64 : RuntimeLibcallImpl<STRCPY>;
  def ___strlen64 : RuntimeLibcallImpl<STRLEN>;
  def ___strstr64 : RuntimeLibcallImpl<STRSTR>;
}

defset list<RuntimeLibcallImpl> PPC32AIXCallList = {
  def ___memccpy : RuntimeLibcallImpl<MEMCCPY>;
  def ___memcmp : RuntimeLibcallImpl<MEMCMP>;
  def ___memmove : RuntimeLibcallImpl<MEMMOVE>;
  def ___memset : RuntimeLibcallImpl<MEMSET>;
  def ___bzero : RuntimeLibcallImpl<BZERO>;
  def ___strcmp : RuntimeLibcallImpl<STRCMP>;
  def ___strcpy : RuntimeLibcallImpl<STRCPY>;
  def ___strlen : RuntimeLibcallImpl<STRLEN>;
  def ___strstr : RuntimeLibcallImpl<STRSTR>;
}

defvar PPCOverrides = !foreach(entry, PPCRuntimeLibcalls, entry.Provides);

def isPPC : RuntimeLibcallPredicate<"TT.isPPC()">;
def isPPC32 : RuntimeLibcallPredicate<"TT.isPPC32()">;
def isPPC64 : RuntimeLibcallPredicate<"TT.isPPC64()">;
def isAIX : RuntimeLibcallPredicate<"TT.isOSAIX()">;
def isNotAIX : RuntimeLibcallPredicate<"!TT.isOSAIX()">;
def isPPC32_AIX : RuntimeLibcallPredicate<"(TT.isPPC32() && TT.isOSAIX())">;
````
- **L3169 EN**: Continues the surrounding expression or declaration: `defset list<RuntimeLibcallImpl> PPC64AIXCallList = {`.
  **L3169 CN**: 继续构造周围的表达式或声明：`defset list<RuntimeLibcallImpl> PPC64AIXCallList = {`。
- **L3170 EN**: Declares TableGen def `___memccpy64`.
  **L3170 CN**: 声明 TableGen def `___memccpy64`。
- **L3171 EN**: Declares TableGen def `___memcmp64`.
  **L3171 CN**: 声明 TableGen def `___memcmp64`。
- **L3172 EN**: Declares TableGen def `___memmove64`.
  **L3172 CN**: 声明 TableGen def `___memmove64`。
- **L3173 EN**: Declares TableGen def `___memset64`.
  **L3173 CN**: 声明 TableGen def `___memset64`。
- **L3174 EN**: Declares TableGen def `___bzero64`.
  **L3174 CN**: 声明 TableGen def `___bzero64`。
- **L3175 EN**: Declares TableGen def `___strcmp64`.
  **L3175 CN**: 声明 TableGen def `___strcmp64`。
- **L3176 EN**: Declares TableGen def `___strcpy64`.
  **L3176 CN**: 声明 TableGen def `___strcpy64`。
- **L3177 EN**: Declares TableGen def `___strlen64`.
  **L3177 CN**: 声明 TableGen def `___strlen64`。
- **L3178 EN**: Declares TableGen def `___strstr64`.
  **L3178 CN**: 声明 TableGen def `___strstr64`。
- **L3179 EN**: Closes the current lexical scope or compound statement.
  **L3179 CN**: 结束当前词法作用域或复合语句块。
- **L3180 EN**: Blank line separating nearby declarations or logic blocks.
  **L3180 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3181 EN**: Continues the surrounding expression or declaration: `defset list<RuntimeLibcallImpl> PPC32AIXCallList = {`.
  **L3181 CN**: 继续构造周围的表达式或声明：`defset list<RuntimeLibcallImpl> PPC32AIXCallList = {`。
- **L3182 EN**: Declares TableGen def `___memccpy`.
  **L3182 CN**: 声明 TableGen def `___memccpy`。
- **L3183 EN**: Declares TableGen def `___memcmp`.
  **L3183 CN**: 声明 TableGen def `___memcmp`。
- **L3184 EN**: Declares TableGen def `___memmove`.
  **L3184 CN**: 声明 TableGen def `___memmove`。
- **L3185 EN**: Declares TableGen def `___memset`.
  **L3185 CN**: 声明 TableGen def `___memset`。
- **L3186 EN**: Declares TableGen def `___bzero`.
  **L3186 CN**: 声明 TableGen def `___bzero`。
- **L3187 EN**: Declares TableGen def `___strcmp`.
  **L3187 CN**: 声明 TableGen def `___strcmp`。
- **L3188 EN**: Declares TableGen def `___strcpy`.
  **L3188 CN**: 声明 TableGen def `___strcpy`。
- **L3189 EN**: Declares TableGen def `___strlen`.
  **L3189 CN**: 声明 TableGen def `___strlen`。
- **L3190 EN**: Declares TableGen def `___strstr`.
  **L3190 CN**: 声明 TableGen def `___strstr`。
- **L3191 EN**: Closes the current lexical scope or compound statement.
  **L3191 CN**: 结束当前词法作用域或复合语句块。
- **L3192 EN**: Blank line separating nearby declarations or logic blocks.
  **L3192 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3193 EN**: Initializes variable `PPCOverrides` from the right-hand expression.
  **L3193 CN**: 使用右侧表达式初始化变量 `PPCOverrides`。
- **L3194 EN**: Blank line separating nearby declarations or logic blocks.
  **L3194 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3195 EN**: Declares TableGen def `isPPC`.
  **L3195 CN**: 声明 TableGen def `isPPC`。
- **L3196 EN**: Declares TableGen def `isPPC32`.
  **L3196 CN**: 声明 TableGen def `isPPC32`。
- **L3197 EN**: Declares TableGen def `isPPC64`.
  **L3197 CN**: 声明 TableGen def `isPPC64`。
- **L3198 EN**: Declares TableGen def `isAIX`.
  **L3198 CN**: 声明 TableGen def `isAIX`。
- **L3199 EN**: Declares TableGen def `isNotAIX`.
  **L3199 CN**: 声明 TableGen def `isNotAIX`。
- **L3200 EN**: Declares TableGen def `isPPC32_AIX`.
  **L3200 CN**: 声明 TableGen def `isPPC32_AIX`。

### Lines 3201-3232

````tablegen
def isPPC64_AIX : RuntimeLibcallPredicate<"(TT.isPPC64() && TT.isOSAIX())">;

def AIX32Calls : LibcallImpls<(add PPC32AIXCallList), isPPC32_AIX>;
def AIX64Calls : LibcallImpls<(add PPC64AIXCallList), isPPC64_AIX>;

// FIXME: Current emission behavior with multiple implementations is
// janky. We need to filter out the conflicting cases with different
// f128 names, and then add the overrides. We should switch to
// explicitly adding subsets of the default calls.
def PPCSystemLibrary
    : SystemRuntimeLibrary<isPPC,
      (add PPCRuntimeLibcalls,
           (sub DefaultRuntimeLibcallImpls, memcpy,
                DefaultRuntimeLibcallImpls_f128),
           __extendkftf2, __trunctfkf2,
           DefaultRuntimeLibcallImpls_ppcf128,
           exp10f, exp10, exp10l_ppcf128,
           LibmF128Libcalls, AIX32Calls, AIX64Calls,
           LibmHasSinCosF32, LibmHasSinCosF64, LibmHasSinCosF128,
           LibmHasSinCosPPCF128,
           AvailableIf<memcpy, isNotAIX>,
           LibcallImpls<(add Int128RTLibcalls), isPPC64>,
           has__stack_smash_handler,
           has___guard_local,
           AvailableIf<__ssp_canary_word, isAIX>,
           AvailableIf<vec_calloc, isAIX>,
           AvailableIf<vec_malloc, isAIX>,
           AvailableIf<vec_realloc, isAIX>,
           AvailableIf<vec_free, isAIX>,
           AvailableIf<__stack_chk_fail, isNotOSOpenBSD>,
           AvailableIf<__stack_chk_guard, isNotOSAIXAndNotOSOpenBSD>)>;

````
- **L3201 EN**: Declares TableGen def `isPPC64_AIX`.
  **L3201 CN**: 声明 TableGen def `isPPC64_AIX`。
- **L3202 EN**: Blank line separating nearby declarations or logic blocks.
  **L3202 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3203 EN**: Declares TableGen def `AIX32Calls`.
  **L3203 CN**: 声明 TableGen def `AIX32Calls`。
- **L3204 EN**: Declares TableGen def `AIX64Calls`.
  **L3204 CN**: 声明 TableGen def `AIX64Calls`。
- **L3205 EN**: Blank line separating nearby declarations or logic blocks.
  **L3205 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3206 EN**: Comment records a pending task or caution: `FIXME: Current emission behavior with multiple implementations is`.
  **L3206 CN**: 注释记录了待办事项或注意点：`FIXME: Current emission behavior with multiple implementations is`。
- **L3207 EN**: Comment explains nearby logic, invariants, or intent: `janky. We need to filter out the conflicting cases with different`.
  **L3207 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`janky. We need to filter out the conflicting cases with different`。
- **L3208 EN**: Comment explains nearby logic, invariants, or intent: `f128 names, and then add the overrides. We should switch to`.
  **L3208 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`f128 names, and then add the overrides. We should switch to`。
- **L3209 EN**: Comment explains nearby logic, invariants, or intent: `explicitly adding subsets of the default calls.`.
  **L3209 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`explicitly adding subsets of the default calls.`。
- **L3210 EN**: Declares TableGen def `PPCSystemLibrary`.
  **L3210 CN**: 声明 TableGen def `PPCSystemLibrary`。
- **L3211 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: SystemRuntimeLibrary<isPPC,`.
  **L3211 CN**: 继续一个多行参数列表、初始化器或聚合项：`: SystemRuntimeLibrary<isPPC,`。
- **L3212 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(add PPCRuntimeLibcalls,`.
  **L3212 CN**: 继续一个多行参数列表、初始化器或聚合项：`(add PPCRuntimeLibcalls,`。
- **L3213 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(sub DefaultRuntimeLibcallImpls, memcpy,`.
  **L3213 CN**: 继续一个多行参数列表、初始化器或聚合项：`(sub DefaultRuntimeLibcallImpls, memcpy,`。
- **L3214 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultRuntimeLibcallImpls_f128),`.
  **L3214 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultRuntimeLibcallImpls_f128),`。
- **L3215 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__extendkftf2, __trunctfkf2,`.
  **L3215 CN**: 继续一个多行参数列表、初始化器或聚合项：`__extendkftf2, __trunctfkf2,`。
- **L3216 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultRuntimeLibcallImpls_ppcf128,`.
  **L3216 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultRuntimeLibcallImpls_ppcf128,`。
- **L3217 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `exp10f, exp10, exp10l_ppcf128,`.
  **L3217 CN**: 继续一个多行参数列表、初始化器或聚合项：`exp10f, exp10, exp10l_ppcf128,`。
- **L3218 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LibmF128Libcalls, AIX32Calls, AIX64Calls,`.
  **L3218 CN**: 继续一个多行参数列表、初始化器或聚合项：`LibmF128Libcalls, AIX32Calls, AIX64Calls,`。
- **L3219 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LibmHasSinCosF32, LibmHasSinCosF64, LibmHasSinCosF128,`.
  **L3219 CN**: 继续一个多行参数列表、初始化器或聚合项：`LibmHasSinCosF32, LibmHasSinCosF64, LibmHasSinCosF128,`。
- **L3220 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LibmHasSinCosPPCF128,`.
  **L3220 CN**: 继续一个多行参数列表、初始化器或聚合项：`LibmHasSinCosPPCF128,`。
- **L3221 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `AvailableIf<memcpy, isNotAIX>,`.
  **L3221 CN**: 继续一个多行参数列表、初始化器或聚合项：`AvailableIf<memcpy, isNotAIX>,`。
- **L3222 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LibcallImpls<(add Int128RTLibcalls), isPPC64>,`.
  **L3222 CN**: 继续一个多行参数列表、初始化器或聚合项：`LibcallImpls<(add Int128RTLibcalls), isPPC64>,`。
- **L3223 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `has__stack_smash_handler,`.
  **L3223 CN**: 继续一个多行参数列表、初始化器或聚合项：`has__stack_smash_handler,`。
- **L3224 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `has___guard_local,`.
  **L3224 CN**: 继续一个多行参数列表、初始化器或聚合项：`has___guard_local,`。
- **L3225 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `AvailableIf<__ssp_canary_word, isAIX>,`.
  **L3225 CN**: 继续一个多行参数列表、初始化器或聚合项：`AvailableIf<__ssp_canary_word, isAIX>,`。
- **L3226 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `AvailableIf<vec_calloc, isAIX>,`.
  **L3226 CN**: 继续一个多行参数列表、初始化器或聚合项：`AvailableIf<vec_calloc, isAIX>,`。
- **L3227 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `AvailableIf<vec_malloc, isAIX>,`.
  **L3227 CN**: 继续一个多行参数列表、初始化器或聚合项：`AvailableIf<vec_malloc, isAIX>,`。
- **L3228 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `AvailableIf<vec_realloc, isAIX>,`.
  **L3228 CN**: 继续一个多行参数列表、初始化器或聚合项：`AvailableIf<vec_realloc, isAIX>,`。
- **L3229 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `AvailableIf<vec_free, isAIX>,`.
  **L3229 CN**: 继续一个多行参数列表、初始化器或聚合项：`AvailableIf<vec_free, isAIX>,`。
- **L3230 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `AvailableIf<__stack_chk_fail, isNotOSOpenBSD>,`.
  **L3230 CN**: 继续一个多行参数列表、初始化器或聚合项：`AvailableIf<__stack_chk_fail, isNotOSOpenBSD>,`。
- **L3231 EN**: Executes a standalone statement or declaration: `AvailableIf<__stack_chk_guard, isNotOSAIXAndNotOSOpenBSD>)>;`.
  **L3231 CN**: 执行一条独立语句或声明：`AvailableIf<__stack_chk_guard, isNotOSAIXAndNotOSOpenBSD>)>;`。
- **L3232 EN**: Blank line separating nearby declarations or logic blocks.
  **L3232 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 3233-3264

````tablegen
//===----------------------------------------------------------------------===//
// RISCV Runtime Libcalls
//===----------------------------------------------------------------------===//

def isRISCV : RuntimeLibcallPredicate<"TT.isRISCV()">;
def isRISCV64 : RuntimeLibcallPredicate<"TT.isRISCV64()">;

def RISCVSystemLibrary
    : SystemRuntimeLibrary<isRISCV,
      (add DefaultRuntimeLibcallImpls,
           exp10f, exp10, exp10l_f128,
           __riscv_flush_icache,
           LibcallImpls<(add Int128RTLibcalls), isRISCV64>,
           DefaultStackProtector,
           DefaultSafeStackGlobals)>;

//===----------------------------------------------------------------------===//
// SPARC Runtime Libcalls
//===----------------------------------------------------------------------===//

// Subtarget->useSoftMulDiv()
def sparc_umul : RuntimeLibcallImpl<MUL_I32, ".umul">;
def sparc_div : RuntimeLibcallImpl<SDIV_I32, ".div">;
def sparc_udiv : RuntimeLibcallImpl<UDIV_I32, ".udiv">;
def sparc_rem : RuntimeLibcallImpl<SREM_I32, ".rem">;
def sparc_urem : RuntimeLibcallImpl<UREM_I32, ".urem">;

// SPARC64 && !Subtarget->useSoftFloat()
def _Qp_add : RuntimeLibcallImpl<ADD_F128>;
def _Qp_sub : RuntimeLibcallImpl<SUB_F128>;
def _Qp_mul : RuntimeLibcallImpl<MUL_F128>;
def _Qp_div : RuntimeLibcallImpl<DIV_F128>;
````
- **L3233 EN**: Banner comment marking a file or section boundary.
  **L3233 CN**: 横幅注释，用于标记文件或章节边界。
- **L3234 EN**: Comment explains nearby logic, invariants, or intent: `RISCV Runtime Libcalls`.
  **L3234 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`RISCV Runtime Libcalls`。
- **L3235 EN**: Banner comment marking a file or section boundary.
  **L3235 CN**: 横幅注释，用于标记文件或章节边界。
- **L3236 EN**: Blank line separating nearby declarations or logic blocks.
  **L3236 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3237 EN**: Declares TableGen def `isRISCV`.
  **L3237 CN**: 声明 TableGen def `isRISCV`。
- **L3238 EN**: Declares TableGen def `isRISCV64`.
  **L3238 CN**: 声明 TableGen def `isRISCV64`。
- **L3239 EN**: Blank line separating nearby declarations or logic blocks.
  **L3239 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3240 EN**: Declares TableGen def `RISCVSystemLibrary`.
  **L3240 CN**: 声明 TableGen def `RISCVSystemLibrary`。
- **L3241 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: SystemRuntimeLibrary<isRISCV,`.
  **L3241 CN**: 继续一个多行参数列表、初始化器或聚合项：`: SystemRuntimeLibrary<isRISCV,`。
- **L3242 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(add DefaultRuntimeLibcallImpls,`.
  **L3242 CN**: 继续一个多行参数列表、初始化器或聚合项：`(add DefaultRuntimeLibcallImpls,`。
- **L3243 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `exp10f, exp10, exp10l_f128,`.
  **L3243 CN**: 继续一个多行参数列表、初始化器或聚合项：`exp10f, exp10, exp10l_f128,`。
- **L3244 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__riscv_flush_icache,`.
  **L3244 CN**: 继续一个多行参数列表、初始化器或聚合项：`__riscv_flush_icache,`。
- **L3245 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LibcallImpls<(add Int128RTLibcalls), isRISCV64>,`.
  **L3245 CN**: 继续一个多行参数列表、初始化器或聚合项：`LibcallImpls<(add Int128RTLibcalls), isRISCV64>,`。
- **L3246 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultStackProtector,`.
  **L3246 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultStackProtector,`。
- **L3247 EN**: Executes a standalone statement or declaration: `DefaultSafeStackGlobals)>;`.
  **L3247 CN**: 执行一条独立语句或声明：`DefaultSafeStackGlobals)>;`。
- **L3248 EN**: Blank line separating nearby declarations or logic blocks.
  **L3248 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3249 EN**: Banner comment marking a file or section boundary.
  **L3249 CN**: 横幅注释，用于标记文件或章节边界。
- **L3250 EN**: Comment explains nearby logic, invariants, or intent: `SPARC Runtime Libcalls`.
  **L3250 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`SPARC Runtime Libcalls`。
- **L3251 EN**: Banner comment marking a file or section boundary.
  **L3251 CN**: 横幅注释，用于标记文件或章节边界。
- **L3252 EN**: Blank line separating nearby declarations or logic blocks.
  **L3252 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3253 EN**: Comment explains nearby logic, invariants, or intent: `Subtarget->useSoftMulDiv()`.
  **L3253 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Subtarget->useSoftMulDiv()`。
- **L3254 EN**: Declares TableGen def `sparc_umul`.
  **L3254 CN**: 声明 TableGen def `sparc_umul`。
- **L3255 EN**: Declares TableGen def `sparc_div`.
  **L3255 CN**: 声明 TableGen def `sparc_div`。
- **L3256 EN**: Declares TableGen def `sparc_udiv`.
  **L3256 CN**: 声明 TableGen def `sparc_udiv`。
- **L3257 EN**: Declares TableGen def `sparc_rem`.
  **L3257 CN**: 声明 TableGen def `sparc_rem`。
- **L3258 EN**: Declares TableGen def `sparc_urem`.
  **L3258 CN**: 声明 TableGen def `sparc_urem`。
- **L3259 EN**: Blank line separating nearby declarations or logic blocks.
  **L3259 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3260 EN**: Comment explains nearby logic, invariants, or intent: `SPARC64 && !Subtarget->useSoftFloat()`.
  **L3260 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`SPARC64 && !Subtarget->useSoftFloat()`。
- **L3261 EN**: Declares TableGen def `_Qp_add`.
  **L3261 CN**: 声明 TableGen def `_Qp_add`。
- **L3262 EN**: Declares TableGen def `_Qp_sub`.
  **L3262 CN**: 声明 TableGen def `_Qp_sub`。
- **L3263 EN**: Declares TableGen def `_Qp_mul`.
  **L3263 CN**: 声明 TableGen def `_Qp_mul`。
- **L3264 EN**: Declares TableGen def `_Qp_div`.
  **L3264 CN**: 声明 TableGen def `_Qp_div`。

### Lines 3265-3296

````tablegen
def _Qp_sqrt : RuntimeLibcallImpl<SQRT_F128>;
def _Qp_qtoi : RuntimeLibcallImpl<FPTOSINT_F128_I32>;
def _Qp_qtoui : RuntimeLibcallImpl<FPTOUINT_F128_I32>;
def _Qp_itoq : RuntimeLibcallImpl<SINTTOFP_I32_F128>;
def _Qp_uitoq : RuntimeLibcallImpl<UINTTOFP_I32_F128>;
def _Qp_qtox : RuntimeLibcallImpl<FPTOSINT_F128_I64>;
def _Qp_qtoux : RuntimeLibcallImpl<FPTOUINT_F128_I64>;
def _Qp_xtoq : RuntimeLibcallImpl<SINTTOFP_I64_F128>;
def _Qp_uxtoq : RuntimeLibcallImpl<UINTTOFP_I64_F128>;
def _Qp_stoq : RuntimeLibcallImpl<FPEXT_F32_F128>;
def _Qp_dtoq : RuntimeLibcallImpl<FPEXT_F64_F128>;
def _Qp_qtos : RuntimeLibcallImpl<FPROUND_F128_F32>;
def _Qp_qtod : RuntimeLibcallImpl<FPROUND_F128_F64>;

// SPARC32 && !Subtarget->useSoftFloat()
def _Q_add : RuntimeLibcallImpl<ADD_F128>;
def _Q_sub : RuntimeLibcallImpl<SUB_F128>;
def _Q_mul : RuntimeLibcallImpl<MUL_F128>;
def _Q_div : RuntimeLibcallImpl<DIV_F128>;
def _Q_sqrt : RuntimeLibcallImpl<SQRT_F128>;
def _Q_qtoi : RuntimeLibcallImpl<FPTOSINT_F128_I32>;
def _Q_qtou : RuntimeLibcallImpl<FPTOUINT_F128_I32>;
def _Q_itoq : RuntimeLibcallImpl<SINTTOFP_I32_F128>;
def _Q_utoq : RuntimeLibcallImpl<UINTTOFP_I32_F128>;
def _Q_stoq : RuntimeLibcallImpl<FPEXT_F32_F128>;
def _Q_dtoq : RuntimeLibcallImpl<FPEXT_F64_F128>;
def _Q_qtos : RuntimeLibcallImpl<FPROUND_F128_F32>;
def _Q_qtod : RuntimeLibcallImpl<FPROUND_F128_F64>;

// SPARC32 && (Subtarget->hasHardQuad() || !Subtarget->useSoftFloat())
def _Q_qtoll : RuntimeLibcallImpl<FPTOSINT_F128_I64>;
def _Q_qtoull : RuntimeLibcallImpl<FPTOUINT_F128_I64>;
````
- **L3265 EN**: Declares TableGen def `_Qp_sqrt`.
  **L3265 CN**: 声明 TableGen def `_Qp_sqrt`。
- **L3266 EN**: Declares TableGen def `_Qp_qtoi`.
  **L3266 CN**: 声明 TableGen def `_Qp_qtoi`。
- **L3267 EN**: Declares TableGen def `_Qp_qtoui`.
  **L3267 CN**: 声明 TableGen def `_Qp_qtoui`。
- **L3268 EN**: Declares TableGen def `_Qp_itoq`.
  **L3268 CN**: 声明 TableGen def `_Qp_itoq`。
- **L3269 EN**: Declares TableGen def `_Qp_uitoq`.
  **L3269 CN**: 声明 TableGen def `_Qp_uitoq`。
- **L3270 EN**: Declares TableGen def `_Qp_qtox`.
  **L3270 CN**: 声明 TableGen def `_Qp_qtox`。
- **L3271 EN**: Declares TableGen def `_Qp_qtoux`.
  **L3271 CN**: 声明 TableGen def `_Qp_qtoux`。
- **L3272 EN**: Declares TableGen def `_Qp_xtoq`.
  **L3272 CN**: 声明 TableGen def `_Qp_xtoq`。
- **L3273 EN**: Declares TableGen def `_Qp_uxtoq`.
  **L3273 CN**: 声明 TableGen def `_Qp_uxtoq`。
- **L3274 EN**: Declares TableGen def `_Qp_stoq`.
  **L3274 CN**: 声明 TableGen def `_Qp_stoq`。
- **L3275 EN**: Declares TableGen def `_Qp_dtoq`.
  **L3275 CN**: 声明 TableGen def `_Qp_dtoq`。
- **L3276 EN**: Declares TableGen def `_Qp_qtos`.
  **L3276 CN**: 声明 TableGen def `_Qp_qtos`。
- **L3277 EN**: Declares TableGen def `_Qp_qtod`.
  **L3277 CN**: 声明 TableGen def `_Qp_qtod`。
- **L3278 EN**: Blank line separating nearby declarations or logic blocks.
  **L3278 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3279 EN**: Comment explains nearby logic, invariants, or intent: `SPARC32 && !Subtarget->useSoftFloat()`.
  **L3279 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`SPARC32 && !Subtarget->useSoftFloat()`。
- **L3280 EN**: Declares TableGen def `_Q_add`.
  **L3280 CN**: 声明 TableGen def `_Q_add`。
- **L3281 EN**: Declares TableGen def `_Q_sub`.
  **L3281 CN**: 声明 TableGen def `_Q_sub`。
- **L3282 EN**: Declares TableGen def `_Q_mul`.
  **L3282 CN**: 声明 TableGen def `_Q_mul`。
- **L3283 EN**: Declares TableGen def `_Q_div`.
  **L3283 CN**: 声明 TableGen def `_Q_div`。
- **L3284 EN**: Declares TableGen def `_Q_sqrt`.
  **L3284 CN**: 声明 TableGen def `_Q_sqrt`。
- **L3285 EN**: Declares TableGen def `_Q_qtoi`.
  **L3285 CN**: 声明 TableGen def `_Q_qtoi`。
- **L3286 EN**: Declares TableGen def `_Q_qtou`.
  **L3286 CN**: 声明 TableGen def `_Q_qtou`。
- **L3287 EN**: Declares TableGen def `_Q_itoq`.
  **L3287 CN**: 声明 TableGen def `_Q_itoq`。
- **L3288 EN**: Declares TableGen def `_Q_utoq`.
  **L3288 CN**: 声明 TableGen def `_Q_utoq`。
- **L3289 EN**: Declares TableGen def `_Q_stoq`.
  **L3289 CN**: 声明 TableGen def `_Q_stoq`。
- **L3290 EN**: Declares TableGen def `_Q_dtoq`.
  **L3290 CN**: 声明 TableGen def `_Q_dtoq`。
- **L3291 EN**: Declares TableGen def `_Q_qtos`.
  **L3291 CN**: 声明 TableGen def `_Q_qtos`。
- **L3292 EN**: Declares TableGen def `_Q_qtod`.
  **L3292 CN**: 声明 TableGen def `_Q_qtod`。
- **L3293 EN**: Blank line separating nearby declarations or logic blocks.
  **L3293 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3294 EN**: Comment explains nearby logic, invariants, or intent: `SPARC32 && (Subtarget->hasHardQuad() || !Subtarget->useSoftFloat())`.
  **L3294 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`SPARC32 && (Subtarget->hasHardQuad() || !Subtarget->useSoftFloat())`。
- **L3295 EN**: Declares TableGen def `_Q_qtoll`.
  **L3295 CN**: 声明 TableGen def `_Q_qtoll`。
- **L3296 EN**: Declares TableGen def `_Q_qtoull`.
  **L3296 CN**: 声明 TableGen def `_Q_qtoull`。

### Lines 3297-3328

````tablegen
def _Q_lltoq : RuntimeLibcallImpl<SINTTOFP_I64_F128>;
def _Q_ulltoq : RuntimeLibcallImpl<UINTTOFP_I64_F128>;

def isSPARC : RuntimeLibcallPredicate<"TT.isSPARC()">;
def isSPARC32 : RuntimeLibcallPredicate<"TT.isSPARC32()">;
def isSPARC64 : RuntimeLibcallPredicate<"TT.isSPARC64()">;

defvar SPARC64_MulDivCalls = [
  __mulsi3, __divsi3, __modsi3, __udivsi3, __umodsi3
];

def SPARCSystemLibrary
    : SystemRuntimeLibrary<isSPARC,
      (add (sub DefaultLibcallImpls32, SPARC64_MulDivCalls),
            sparc_umul, sparc_div, sparc_udiv, sparc_rem, sparc_urem,
       LibcallImpls<(add _Q_qtoll, _Q_qtoull, _Q_lltoq, _Q_ulltoq), isSPARC32>,
       LibcallImpls<(add SPARC64_MulDivCalls, Int128RTLibcalls), isSPARC64>,
       LibmHasSinCosF32, LibmHasSinCosF64, LibmHasSinCosF128,
       has__stack_chk_fail,
       has__stack_smash_handler,
       has___guard_local,
       DefaultSafeStackGlobals,
       AvailableIf<__stack_chk_guard, isNotOSLinuxAndNotOSOpenBSD>)
>;

//===----------------------------------------------------------------------===//
// SPIRV Runtime Libcalls
//===----------------------------------------------------------------------===//

def isSPIRV : RuntimeLibcallPredicate<"TT.isSPIRV()">;

// No calls FIXME: Add memcpy/memset is a hack to skip
````
- **L3297 EN**: Declares TableGen def `_Q_lltoq`.
  **L3297 CN**: 声明 TableGen def `_Q_lltoq`。
- **L3298 EN**: Declares TableGen def `_Q_ulltoq`.
  **L3298 CN**: 声明 TableGen def `_Q_ulltoq`。
- **L3299 EN**: Blank line separating nearby declarations or logic blocks.
  **L3299 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3300 EN**: Declares TableGen def `isSPARC`.
  **L3300 CN**: 声明 TableGen def `isSPARC`。
- **L3301 EN**: Declares TableGen def `isSPARC32`.
  **L3301 CN**: 声明 TableGen def `isSPARC32`。
- **L3302 EN**: Declares TableGen def `isSPARC64`.
  **L3302 CN**: 声明 TableGen def `isSPARC64`。
- **L3303 EN**: Blank line separating nearby declarations or logic blocks.
  **L3303 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3304 EN**: Continues the surrounding expression or declaration: `defvar SPARC64_MulDivCalls = [`.
  **L3304 CN**: 继续构造周围的表达式或声明：`defvar SPARC64_MulDivCalls = [`。
- **L3305 EN**: Continues the surrounding expression or declaration: `__mulsi3, __divsi3, __modsi3, __udivsi3, __umodsi3`.
  **L3305 CN**: 继续构造周围的表达式或声明：`__mulsi3, __divsi3, __modsi3, __udivsi3, __umodsi3`。
- **L3306 EN**: Executes a standalone statement or declaration: `];`.
  **L3306 CN**: 执行一条独立语句或声明：`];`。
- **L3307 EN**: Blank line separating nearby declarations or logic blocks.
  **L3307 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3308 EN**: Declares TableGen def `SPARCSystemLibrary`.
  **L3308 CN**: 声明 TableGen def `SPARCSystemLibrary`。
- **L3309 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: SystemRuntimeLibrary<isSPARC,`.
  **L3309 CN**: 继续一个多行参数列表、初始化器或聚合项：`: SystemRuntimeLibrary<isSPARC,`。
- **L3310 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(add (sub DefaultLibcallImpls32, SPARC64_MulDivCalls),`.
  **L3310 CN**: 继续一个多行参数列表、初始化器或聚合项：`(add (sub DefaultLibcallImpls32, SPARC64_MulDivCalls),`。
- **L3311 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `sparc_umul, sparc_div, sparc_udiv, sparc_rem, sparc_urem,`.
  **L3311 CN**: 继续一个多行参数列表、初始化器或聚合项：`sparc_umul, sparc_div, sparc_udiv, sparc_rem, sparc_urem,`。
- **L3312 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LibcallImpls<(add _Q_qtoll, _Q_qtoull, _Q_lltoq, _Q_ulltoq), isSPARC32>,`.
  **L3312 CN**: 继续一个多行参数列表、初始化器或聚合项：`LibcallImpls<(add _Q_qtoll, _Q_qtoull, _Q_lltoq, _Q_ulltoq), isSPARC32>,`。
- **L3313 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LibcallImpls<(add SPARC64_MulDivCalls, Int128RTLibcalls), isSPARC64>,`.
  **L3313 CN**: 继续一个多行参数列表、初始化器或聚合项：`LibcallImpls<(add SPARC64_MulDivCalls, Int128RTLibcalls), isSPARC64>,`。
- **L3314 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LibmHasSinCosF32, LibmHasSinCosF64, LibmHasSinCosF128,`.
  **L3314 CN**: 继续一个多行参数列表、初始化器或聚合项：`LibmHasSinCosF32, LibmHasSinCosF64, LibmHasSinCosF128,`。
- **L3315 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `has__stack_chk_fail,`.
  **L3315 CN**: 继续一个多行参数列表、初始化器或聚合项：`has__stack_chk_fail,`。
- **L3316 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `has__stack_smash_handler,`.
  **L3316 CN**: 继续一个多行参数列表、初始化器或聚合项：`has__stack_smash_handler,`。
- **L3317 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `has___guard_local,`.
  **L3317 CN**: 继续一个多行参数列表、初始化器或聚合项：`has___guard_local,`。
- **L3318 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultSafeStackGlobals,`.
  **L3318 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultSafeStackGlobals,`。
- **L3319 EN**: Continues the surrounding expression or declaration: `AvailableIf<__stack_chk_guard, isNotOSLinuxAndNotOSOpenBSD>)`.
  **L3319 CN**: 继续构造周围的表达式或声明：`AvailableIf<__stack_chk_guard, isNotOSLinuxAndNotOSOpenBSD>)`。
- **L3320 EN**: Executes a standalone statement or declaration: `>;`.
  **L3320 CN**: 执行一条独立语句或声明：`>;`。
- **L3321 EN**: Blank line separating nearby declarations or logic blocks.
  **L3321 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3322 EN**: Banner comment marking a file or section boundary.
  **L3322 CN**: 横幅注释，用于标记文件或章节边界。
- **L3323 EN**: Comment explains nearby logic, invariants, or intent: `SPIRV Runtime Libcalls`.
  **L3323 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`SPIRV Runtime Libcalls`。
- **L3324 EN**: Banner comment marking a file or section boundary.
  **L3324 CN**: 横幅注释，用于标记文件或章节边界。
- **L3325 EN**: Blank line separating nearby declarations or logic blocks.
  **L3325 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3326 EN**: Declares TableGen def `isSPIRV`.
  **L3326 CN**: 声明 TableGen def `isSPIRV`。
- **L3327 EN**: Blank line separating nearby declarations or logic blocks.
  **L3327 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3328 EN**: Comment records a pending task or caution: `No calls FIXME: Add memcpy/memset is a hack to skip`.
  **L3328 CN**: 注释记录了待办事项或注意点：`No calls FIXME: Add memcpy/memset is a hack to skip`。

### Lines 3329-3360

````tablegen
// PreISelIntrinsicLowering in favor of SPIRVPrepareFunctions;
// probably should remove that and just use the default one.
def SPIRVSystemLibrary : SystemRuntimeLibrary<isSPIRV, (add memset, memcpy)>;

//===----------------------------------------------------------------------===//
// Windows Runtime Libcalls
//===----------------------------------------------------------------------===//

defset list<RuntimeLibcallImpl> WindowsDivRemMulLibcalls = {
def _alldiv : RuntimeLibcallImpl<SDIV_I64>;
def _aulldiv : RuntimeLibcallImpl<UDIV_I64>;
def _allrem : RuntimeLibcallImpl<SREM_I64>;
def _aullrem : RuntimeLibcallImpl<UREM_I64>;
def _allmul : RuntimeLibcallImpl<MUL_I64>;
}

// FIXME: Should have utility function to filter by known provider.
defvar WindowsDivRemMulLibcallOverrides = [
  __divdi3, __udivdi3, __moddi3, __umoddi3, __muldi3
];

//===----------------------------------------------------------------------===//
// X86 Runtime Libcalls
//===----------------------------------------------------------------------===//

def isX86_32 : RuntimeLibcallPredicate<"TT.getArch() == Triple::x86">;
def isX86_64 : RuntimeLibcallPredicate<"TT.getArch() == Triple::x86_64">;
def isX86 : RuntimeLibcallPredicate<"TT.isX86()">;

def isCygwinMinGW64 : RuntimeLibcallPredicate<
  [{TT.isOSCygMing() && TT.getArch() == Triple::x86_64}]>;
def isCygwinMinGW32 : RuntimeLibcallPredicate<
````
- **L3329 EN**: Comment explains nearby logic, invariants, or intent: `PreISelIntrinsicLowering in favor of SPIRVPrepareFunctions;`.
  **L3329 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`PreISelIntrinsicLowering in favor of SPIRVPrepareFunctions;`。
- **L3330 EN**: Comment explains nearby logic, invariants, or intent: `probably should remove that and just use the default one.`.
  **L3330 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`probably should remove that and just use the default one.`。
- **L3331 EN**: Declares TableGen def `SPIRVSystemLibrary`.
  **L3331 CN**: 声明 TableGen def `SPIRVSystemLibrary`。
- **L3332 EN**: Blank line separating nearby declarations or logic blocks.
  **L3332 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3333 EN**: Banner comment marking a file or section boundary.
  **L3333 CN**: 横幅注释，用于标记文件或章节边界。
- **L3334 EN**: Comment explains nearby logic, invariants, or intent: `Windows Runtime Libcalls`.
  **L3334 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Windows Runtime Libcalls`。
- **L3335 EN**: Banner comment marking a file or section boundary.
  **L3335 CN**: 横幅注释，用于标记文件或章节边界。
- **L3336 EN**: Blank line separating nearby declarations or logic blocks.
  **L3336 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3337 EN**: Continues the surrounding expression or declaration: `defset list<RuntimeLibcallImpl> WindowsDivRemMulLibcalls = {`.
  **L3337 CN**: 继续构造周围的表达式或声明：`defset list<RuntimeLibcallImpl> WindowsDivRemMulLibcalls = {`。
- **L3338 EN**: Declares TableGen def `_alldiv`.
  **L3338 CN**: 声明 TableGen def `_alldiv`。
- **L3339 EN**: Declares TableGen def `_aulldiv`.
  **L3339 CN**: 声明 TableGen def `_aulldiv`。
- **L3340 EN**: Declares TableGen def `_allrem`.
  **L3340 CN**: 声明 TableGen def `_allrem`。
- **L3341 EN**: Declares TableGen def `_aullrem`.
  **L3341 CN**: 声明 TableGen def `_aullrem`。
- **L3342 EN**: Declares TableGen def `_allmul`.
  **L3342 CN**: 声明 TableGen def `_allmul`。
- **L3343 EN**: Closes the current lexical scope or compound statement.
  **L3343 CN**: 结束当前词法作用域或复合语句块。
- **L3344 EN**: Blank line separating nearby declarations or logic blocks.
  **L3344 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3345 EN**: Comment records a pending task or caution: `FIXME: Should have utility function to filter by known provider.`.
  **L3345 CN**: 注释记录了待办事项或注意点：`FIXME: Should have utility function to filter by known provider.`。
- **L3346 EN**: Continues the surrounding expression or declaration: `defvar WindowsDivRemMulLibcallOverrides = [`.
  **L3346 CN**: 继续构造周围的表达式或声明：`defvar WindowsDivRemMulLibcallOverrides = [`。
- **L3347 EN**: Continues the surrounding expression or declaration: `__divdi3, __udivdi3, __moddi3, __umoddi3, __muldi3`.
  **L3347 CN**: 继续构造周围的表达式或声明：`__divdi3, __udivdi3, __moddi3, __umoddi3, __muldi3`。
- **L3348 EN**: Executes a standalone statement or declaration: `];`.
  **L3348 CN**: 执行一条独立语句或声明：`];`。
- **L3349 EN**: Blank line separating nearby declarations or logic blocks.
  **L3349 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3350 EN**: Banner comment marking a file or section boundary.
  **L3350 CN**: 横幅注释，用于标记文件或章节边界。
- **L3351 EN**: Comment explains nearby logic, invariants, or intent: `X86 Runtime Libcalls`.
  **L3351 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`X86 Runtime Libcalls`。
- **L3352 EN**: Banner comment marking a file or section boundary.
  **L3352 CN**: 横幅注释，用于标记文件或章节边界。
- **L3353 EN**: Blank line separating nearby declarations or logic blocks.
  **L3353 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3354 EN**: Declares TableGen def `isX86_32`.
  **L3354 CN**: 声明 TableGen def `isX86_32`。
- **L3355 EN**: Declares TableGen def `isX86_64`.
  **L3355 CN**: 声明 TableGen def `isX86_64`。
- **L3356 EN**: Declares TableGen def `isX86`.
  **L3356 CN**: 声明 TableGen def `isX86`。
- **L3357 EN**: Blank line separating nearby declarations or logic blocks.
  **L3357 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3358 EN**: Declares TableGen def `isCygwinMinGW64`.
  **L3358 CN**: 声明 TableGen def `isCygwinMinGW64`。
- **L3359 EN**: Executes a call or declaration centered on `[{TT.isOSCygMing`.
  **L3359 CN**: 执行以 `[{TT.isOSCygMing` 为核心的调用或声明。
- **L3360 EN**: Declares TableGen def `isCygwinMinGW32`.
  **L3360 CN**: 声明 TableGen def `isCygwinMinGW32`。

### Lines 3361-3392

````tablegen
  [{TT.isOSCygMing() && TT.getArch() == Triple::x86}]>;

def isWin32NotCygMing : RuntimeLibcallPredicate<
  [{TT.getArch() == Triple::x86 &&
     (TT.isOSWindows() || TT.isUEFI()) && !TT.isOSCygMing()}]>;
def isWin64NotCygMing : RuntimeLibcallPredicate<
  [{TT.getArch() == Triple::x86_64 &&
    (TT.isOSWindows() || TT.isUEFI()) && !TT.isOSCygMing()}]>;


// Some darwins have an optimized __bzero/bzero function.
def darwinHas__bzero : RuntimeLibcallPredicate<"TT.isMacOSX() && !TT.isMacOSXVersionLT(10, 6)">;

// FIXME: This is has ldexpl/frexpl plus use f128 for long double.
def hasExpFrexplLdexplF128
  : RuntimeLibcallPredicate<[{(!TT.isOSWindows() || TT.isOSCygMing()) && !TT.isGNUEnvironment()}]>;

// Use the f128 variants of math functions on x86
defvar X86_F128_Libcalls = LibcallImpls<(add LibmF128Libcalls, LibmF128FiniteLibcalls), isGNUEnvironment>;

defvar SinCosF32F64Libcalls = LibcallImpls<(add sincosf, sincos), hasSinCos_f32_f64>;

defvar MemChkLibcalls = [__memcpy_chk, __memset_chk, __memmove_chk];

defvar X86CommonLibcalls =
  (add (sub WinDefaultLibcallImpls, WindowsDivRemMulLibcallOverrides, MemChkLibcalls),
       DarwinSinCosStret, DarwinExp10, DarwinMemsetPattern, MacOSUnlockedIO,
       X86_F128_Libcalls,
       LibmHasSinCosF80, // FIXME: Depends on long double
       SinCosF32F64Libcalls,
       LibcallImpls<(add __bzero), darwinHas__bzero>,
       LibmHasFrexpF32, LibmHasLdexpF32,
````
- **L3361 EN**: Executes a call or declaration centered on `[{TT.isOSCygMing`.
  **L3361 CN**: 执行以 `[{TT.isOSCygMing` 为核心的调用或声明。
- **L3362 EN**: Blank line separating nearby declarations or logic blocks.
  **L3362 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3363 EN**: Declares TableGen def `isWin32NotCygMing`.
  **L3363 CN**: 声明 TableGen def `isWin32NotCygMing`。
- **L3364 EN**: Continues logic associated with callable symbol `getArch`.
  **L3364 CN**: 继续与可调用符号 `getArch` 相关的逻辑。
- **L3365 EN**: Executes a call or declaration centered on `statement`.
  **L3365 CN**: 执行以 `statement` 为核心的调用或声明。
- **L3366 EN**: Declares TableGen def `isWin64NotCygMing`.
  **L3366 CN**: 声明 TableGen def `isWin64NotCygMing`。
- **L3367 EN**: Continues logic associated with callable symbol `getArch`.
  **L3367 CN**: 继续与可调用符号 `getArch` 相关的逻辑。
- **L3368 EN**: Executes a call or declaration centered on `statement`.
  **L3368 CN**: 执行以 `statement` 为核心的调用或声明。
- **L3369 EN**: Blank line separating nearby declarations or logic blocks.
  **L3369 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3370 EN**: Blank line separating nearby declarations or logic blocks.
  **L3370 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3371 EN**: Comment explains nearby logic, invariants, or intent: `Some darwins have an optimized __bzero/bzero function.`.
  **L3371 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Some darwins have an optimized __bzero/bzero function.`。
- **L3372 EN**: Declares TableGen def `darwinHas__bzero`.
  **L3372 CN**: 声明 TableGen def `darwinHas__bzero`。
- **L3373 EN**: Blank line separating nearby declarations or logic blocks.
  **L3373 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3374 EN**: Comment records a pending task or caution: `FIXME: This is has ldexpl/frexpl plus use f128 for long double.`.
  **L3374 CN**: 注释记录了待办事项或注意点：`FIXME: This is has ldexpl/frexpl plus use f128 for long double.`。
- **L3375 EN**: Declares TableGen def `hasExpFrexplLdexplF128`.
  **L3375 CN**: 声明 TableGen def `hasExpFrexplLdexplF128`。
- **L3376 EN**: Executes a call or declaration centered on `RuntimeLibcallPredicate<[{`.
  **L3376 CN**: 执行以 `RuntimeLibcallPredicate<[{` 为核心的调用或声明。
- **L3377 EN**: Blank line separating nearby declarations or logic blocks.
  **L3377 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3378 EN**: Comment explains nearby logic, invariants, or intent: `Use the f128 variants of math functions on x86`.
  **L3378 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Use the f128 variants of math functions on x86`。
- **L3379 EN**: Initializes variable `X86_F128_Libcalls` from the right-hand expression.
  **L3379 CN**: 使用右侧表达式初始化变量 `X86_F128_Libcalls`。
- **L3380 EN**: Blank line separating nearby declarations or logic blocks.
  **L3380 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3381 EN**: Initializes variable `SinCosF32F64Libcalls` from the right-hand expression.
  **L3381 CN**: 使用右侧表达式初始化变量 `SinCosF32F64Libcalls`。
- **L3382 EN**: Blank line separating nearby declarations or logic blocks.
  **L3382 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3383 EN**: Initializes variable `MemChkLibcalls` from the right-hand expression.
  **L3383 CN**: 使用右侧表达式初始化变量 `MemChkLibcalls`。
- **L3384 EN**: Blank line separating nearby declarations or logic blocks.
  **L3384 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3385 EN**: Continues the surrounding expression or declaration: `defvar X86CommonLibcalls =`.
  **L3385 CN**: 继续构造周围的表达式或声明：`defvar X86CommonLibcalls =`。
- **L3386 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(add (sub WinDefaultLibcallImpls, WindowsDivRemMulLibcallOverrides, MemChkLibcalls),`.
  **L3386 CN**: 继续一个多行参数列表、初始化器或聚合项：`(add (sub WinDefaultLibcallImpls, WindowsDivRemMulLibcallOverrides, MemChkLibcalls),`。
- **L3387 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DarwinSinCosStret, DarwinExp10, DarwinMemsetPattern, MacOSUnlockedIO,`.
  **L3387 CN**: 继续一个多行参数列表、初始化器或聚合项：`DarwinSinCosStret, DarwinExp10, DarwinMemsetPattern, MacOSUnlockedIO,`。
- **L3388 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `X86_F128_Libcalls,`.
  **L3388 CN**: 继续一个多行参数列表、初始化器或聚合项：`X86_F128_Libcalls,`。
- **L3389 EN**: Continues the surrounding expression or declaration: `LibmHasSinCosF80, // FIXME: Depends on long double`.
  **L3389 CN**: 继续构造周围的表达式或声明：`LibmHasSinCosF80, // FIXME: Depends on long double`。
- **L3390 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `SinCosF32F64Libcalls,`.
  **L3390 CN**: 继续一个多行参数列表、初始化器或聚合项：`SinCosF32F64Libcalls,`。
- **L3391 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LibcallImpls<(add __bzero), darwinHas__bzero>,`.
  **L3391 CN**: 继续一个多行参数列表、初始化器或聚合项：`LibcallImpls<(add __bzero), darwinHas__bzero>,`。
- **L3392 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LibmHasFrexpF32, LibmHasLdexpF32,`.
  **L3392 CN**: 继续一个多行参数列表、初始化器或聚合项：`LibmHasFrexpF32, LibmHasLdexpF32,`。

### Lines 3393-3424

````tablegen
       LibmHasFrexpF80, LibmHasLdexpF80,
       LibcallImpls<(add frexpl_f128, ldexpl_f128, exp10l_f128), hasExpFrexplLdexplF128>,
       DefaultRuntimeLibcallImpls_f80,
       LibmHasExp10F32, LibmHasExp10F64, LibmHasExp10F80,
       LibcallImpls<(add MostPowI), isNotOSMSVCRT>,
       SecurityCheckCookieIfWinMSVC,
       // FIXME: MSVCRT doesn't have powi. The f128 case is added as a
       // hack for one test relying on it.
       __powitf2_f128,
       DefaultStackProtector,
       DefaultSafeStackGlobals,
       LibcallImpls<(add MemChkLibcalls), isNotPS>,
       LibcallImpls<(add ___chkstk_ms), isCygwinMinGW64>,
       LibcallImpls<(add __chkstk), isWin64NotCygMing>,
       LibcallImpls<(add _alloca), isCygwinMinGW32>,
       LibcallImpls<(add _chkstk), isWin32NotCygMing>,
       LibcSafestackPointerAddress
);

defvar Windows32DivRemMulCalls =
  LibcallsWithCC<(add WindowsDivRemMulLibcalls), X86_STDCALL,
  RuntimeLibcallPredicate<"TT.isWindowsMSVCEnvironment() || TT.isWindowsItaniumEnvironment()">>;

defvar NotWindows32DivRemMulCalls =
  LibcallImpls<(add WindowsDivRemMulLibcallOverrides),
    RuntimeLibcallPredicate<"!TT.isWindowsMSVCEnvironment() && !TT.isWindowsItaniumEnvironment()">>;

def X86_32SystemLibrary
    : SystemRuntimeLibrary<isX86_32,
      (add X86CommonLibcalls,
             NotWindows32DivRemMulCalls,
             Windows32DivRemMulCalls)>;
````
- **L3393 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LibmHasFrexpF80, LibmHasLdexpF80,`.
  **L3393 CN**: 继续一个多行参数列表、初始化器或聚合项：`LibmHasFrexpF80, LibmHasLdexpF80,`。
- **L3394 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LibcallImpls<(add frexpl_f128, ldexpl_f128, exp10l_f128), hasExpFrexplLdexplF128>,`.
  **L3394 CN**: 继续一个多行参数列表、初始化器或聚合项：`LibcallImpls<(add frexpl_f128, ldexpl_f128, exp10l_f128), hasExpFrexplLdexplF128>,`。
- **L3395 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultRuntimeLibcallImpls_f80,`.
  **L3395 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultRuntimeLibcallImpls_f80,`。
- **L3396 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LibmHasExp10F32, LibmHasExp10F64, LibmHasExp10F80,`.
  **L3396 CN**: 继续一个多行参数列表、初始化器或聚合项：`LibmHasExp10F32, LibmHasExp10F64, LibmHasExp10F80,`。
- **L3397 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LibcallImpls<(add MostPowI), isNotOSMSVCRT>,`.
  **L3397 CN**: 继续一个多行参数列表、初始化器或聚合项：`LibcallImpls<(add MostPowI), isNotOSMSVCRT>,`。
- **L3398 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `SecurityCheckCookieIfWinMSVC,`.
  **L3398 CN**: 继续一个多行参数列表、初始化器或聚合项：`SecurityCheckCookieIfWinMSVC,`。
- **L3399 EN**: Comment records a pending task or caution: `FIXME: MSVCRT doesn't have powi. The f128 case is added as a`.
  **L3399 CN**: 注释记录了待办事项或注意点：`FIXME: MSVCRT doesn't have powi. The f128 case is added as a`。
- **L3400 EN**: Comment explains nearby logic, invariants, or intent: `hack for one test relying on it.`.
  **L3400 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`hack for one test relying on it.`。
- **L3401 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__powitf2_f128,`.
  **L3401 CN**: 继续一个多行参数列表、初始化器或聚合项：`__powitf2_f128,`。
- **L3402 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultStackProtector,`.
  **L3402 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultStackProtector,`。
- **L3403 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultSafeStackGlobals,`.
  **L3403 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultSafeStackGlobals,`。
- **L3404 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LibcallImpls<(add MemChkLibcalls), isNotPS>,`.
  **L3404 CN**: 继续一个多行参数列表、初始化器或聚合项：`LibcallImpls<(add MemChkLibcalls), isNotPS>,`。
- **L3405 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LibcallImpls<(add ___chkstk_ms), isCygwinMinGW64>,`.
  **L3405 CN**: 继续一个多行参数列表、初始化器或聚合项：`LibcallImpls<(add ___chkstk_ms), isCygwinMinGW64>,`。
- **L3406 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LibcallImpls<(add __chkstk), isWin64NotCygMing>,`.
  **L3406 CN**: 继续一个多行参数列表、初始化器或聚合项：`LibcallImpls<(add __chkstk), isWin64NotCygMing>,`。
- **L3407 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LibcallImpls<(add _alloca), isCygwinMinGW32>,`.
  **L3407 CN**: 继续一个多行参数列表、初始化器或聚合项：`LibcallImpls<(add _alloca), isCygwinMinGW32>,`。
- **L3408 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LibcallImpls<(add _chkstk), isWin32NotCygMing>,`.
  **L3408 CN**: 继续一个多行参数列表、初始化器或聚合项：`LibcallImpls<(add _chkstk), isWin32NotCygMing>,`。
- **L3409 EN**: Continues the surrounding expression or declaration: `LibcSafestackPointerAddress`.
  **L3409 CN**: 继续构造周围的表达式或声明：`LibcSafestackPointerAddress`。
- **L3410 EN**: Executes a standalone statement or declaration: `);`.
  **L3410 CN**: 执行一条独立语句或声明：`);`。
- **L3411 EN**: Blank line separating nearby declarations or logic blocks.
  **L3411 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3412 EN**: Continues the surrounding expression or declaration: `defvar Windows32DivRemMulCalls =`.
  **L3412 CN**: 继续构造周围的表达式或声明：`defvar Windows32DivRemMulCalls =`。
- **L3413 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LibcallsWithCC<(add WindowsDivRemMulLibcalls), X86_STDCALL,`.
  **L3413 CN**: 继续一个多行参数列表、初始化器或聚合项：`LibcallsWithCC<(add WindowsDivRemMulLibcalls), X86_STDCALL,`。
- **L3414 EN**: Executes a call or declaration centered on `RuntimeLibcallPredicate<"TT.isWindowsMSVCEnvironment`.
  **L3414 CN**: 执行以 `RuntimeLibcallPredicate<"TT.isWindowsMSVCEnvironment` 为核心的调用或声明。
- **L3415 EN**: Blank line separating nearby declarations or logic blocks.
  **L3415 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3416 EN**: Continues the surrounding expression or declaration: `defvar NotWindows32DivRemMulCalls =`.
  **L3416 CN**: 继续构造周围的表达式或声明：`defvar NotWindows32DivRemMulCalls =`。
- **L3417 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LibcallImpls<(add WindowsDivRemMulLibcallOverrides),`.
  **L3417 CN**: 继续一个多行参数列表、初始化器或聚合项：`LibcallImpls<(add WindowsDivRemMulLibcallOverrides),`。
- **L3418 EN**: Executes a call or declaration centered on `RuntimeLibcallPredicate<"!TT.isWindowsMSVCEnvironment`.
  **L3418 CN**: 执行以 `RuntimeLibcallPredicate<"!TT.isWindowsMSVCEnvironment` 为核心的调用或声明。
- **L3419 EN**: Blank line separating nearby declarations or logic blocks.
  **L3419 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3420 EN**: Declares TableGen def `X86_32SystemLibrary`.
  **L3420 CN**: 声明 TableGen def `X86_32SystemLibrary`。
- **L3421 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: SystemRuntimeLibrary<isX86_32,`.
  **L3421 CN**: 继续一个多行参数列表、初始化器或聚合项：`: SystemRuntimeLibrary<isX86_32,`。
- **L3422 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(add X86CommonLibcalls,`.
  **L3422 CN**: 继续一个多行参数列表、初始化器或聚合项：`(add X86CommonLibcalls,`。
- **L3423 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NotWindows32DivRemMulCalls,`.
  **L3423 CN**: 继续一个多行参数列表、初始化器或聚合项：`NotWindows32DivRemMulCalls,`。
- **L3424 EN**: Executes a standalone statement or declaration: `Windows32DivRemMulCalls)>;`.
  **L3424 CN**: 执行一条独立语句或声明：`Windows32DivRemMulCalls)>;`。

### Lines 3425-3456

````tablegen

def X86_64SystemLibrary
    : SystemRuntimeLibrary<isX86_64,
      (add X86CommonLibcalls, Int128RTLibcalls)>;

//===----------------------------------------------------------------------===//
// XCore Runtime Libcalls
//===----------------------------------------------------------------------===//

def __memcpy_4 : RuntimeLibcallImpl<MEMCPY_ALIGN_4>;

def isXCore : RuntimeLibcallPredicate<"TT.getArch() == Triple::xcore">;
def XCoreSystemLibrary
    : SystemRuntimeLibrary<isXCore,
    (add DefaultRuntimeLibcallImpls,
         exp10f, exp10, exp10l_f128,
         __memcpy_4,
         iprintf, siprintf, fiprintf,
         LibcallImpls<(add LibmF128Libcalls, LibmF128FiniteLibcalls), isGNUEnvironment>
)>;

//===----------------------------------------------------------------------===//
// ZOS Runtime Libcalls
//===----------------------------------------------------------------------===//

defset list<RuntimeLibcallImpl> ZOSRuntimeLibcalls = {
  def zos___TRNC_B : RuntimeLibcallImpl<TRUNC_F64, "@@TRNC@B">;
  def zos___FTRC_B : RuntimeLibcallImpl<TRUNC_F32, "@@FTRC@B">;
  def zos___LTRC_B : RuntimeLibcallImpl<TRUNC_F128, "@@LTRC@B">;
  def zos___WSQT_B : RuntimeLibcallImpl<SQRT_F64, "@@WSQT@B">;
  def zos___FSQT_B : RuntimeLibcallImpl<SQRT_F32, "@@FSQT@B">;
  def zos___LSQT_B : RuntimeLibcallImpl<SQRT_F128, "@@LSQT@B">;
````
- **L3425 EN**: Blank line separating nearby declarations or logic blocks.
  **L3425 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3426 EN**: Declares TableGen def `X86_64SystemLibrary`.
  **L3426 CN**: 声明 TableGen def `X86_64SystemLibrary`。
- **L3427 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: SystemRuntimeLibrary<isX86_64,`.
  **L3427 CN**: 继续一个多行参数列表、初始化器或聚合项：`: SystemRuntimeLibrary<isX86_64,`。
- **L3428 EN**: Executes a call or declaration centered on `statement`.
  **L3428 CN**: 执行以 `statement` 为核心的调用或声明。
- **L3429 EN**: Blank line separating nearby declarations or logic blocks.
  **L3429 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3430 EN**: Banner comment marking a file or section boundary.
  **L3430 CN**: 横幅注释，用于标记文件或章节边界。
- **L3431 EN**: Comment explains nearby logic, invariants, or intent: `XCore Runtime Libcalls`.
  **L3431 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`XCore Runtime Libcalls`。
- **L3432 EN**: Banner comment marking a file or section boundary.
  **L3432 CN**: 横幅注释，用于标记文件或章节边界。
- **L3433 EN**: Blank line separating nearby declarations or logic blocks.
  **L3433 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3434 EN**: Declares TableGen def `__memcpy_4`.
  **L3434 CN**: 声明 TableGen def `__memcpy_4`。
- **L3435 EN**: Blank line separating nearby declarations or logic blocks.
  **L3435 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3436 EN**: Declares TableGen def `isXCore`.
  **L3436 CN**: 声明 TableGen def `isXCore`。
- **L3437 EN**: Declares TableGen def `XCoreSystemLibrary`.
  **L3437 CN**: 声明 TableGen def `XCoreSystemLibrary`。
- **L3438 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: SystemRuntimeLibrary<isXCore,`.
  **L3438 CN**: 继续一个多行参数列表、初始化器或聚合项：`: SystemRuntimeLibrary<isXCore,`。
- **L3439 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(add DefaultRuntimeLibcallImpls,`.
  **L3439 CN**: 继续一个多行参数列表、初始化器或聚合项：`(add DefaultRuntimeLibcallImpls,`。
- **L3440 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `exp10f, exp10, exp10l_f128,`.
  **L3440 CN**: 继续一个多行参数列表、初始化器或聚合项：`exp10f, exp10, exp10l_f128,`。
- **L3441 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__memcpy_4,`.
  **L3441 CN**: 继续一个多行参数列表、初始化器或聚合项：`__memcpy_4,`。
- **L3442 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `iprintf, siprintf, fiprintf,`.
  **L3442 CN**: 继续一个多行参数列表、初始化器或聚合项：`iprintf, siprintf, fiprintf,`。
- **L3443 EN**: Continues logic associated with callable symbol `LibcallImpls<`.
  **L3443 CN**: 继续与可调用符号 `LibcallImpls<` 相关的逻辑。
- **L3444 EN**: Executes a standalone statement or declaration: `)>;`.
  **L3444 CN**: 执行一条独立语句或声明：`)>;`。
- **L3445 EN**: Blank line separating nearby declarations or logic blocks.
  **L3445 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3446 EN**: Banner comment marking a file or section boundary.
  **L3446 CN**: 横幅注释，用于标记文件或章节边界。
- **L3447 EN**: Comment explains nearby logic, invariants, or intent: `ZOS Runtime Libcalls`.
  **L3447 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`ZOS Runtime Libcalls`。
- **L3448 EN**: Banner comment marking a file or section boundary.
  **L3448 CN**: 横幅注释，用于标记文件或章节边界。
- **L3449 EN**: Blank line separating nearby declarations or logic blocks.
  **L3449 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3450 EN**: Continues the surrounding expression or declaration: `defset list<RuntimeLibcallImpl> ZOSRuntimeLibcalls = {`.
  **L3450 CN**: 继续构造周围的表达式或声明：`defset list<RuntimeLibcallImpl> ZOSRuntimeLibcalls = {`。
- **L3451 EN**: Declares TableGen def `zos___TRNC_B`.
  **L3451 CN**: 声明 TableGen def `zos___TRNC_B`。
- **L3452 EN**: Declares TableGen def `zos___FTRC_B`.
  **L3452 CN**: 声明 TableGen def `zos___FTRC_B`。
- **L3453 EN**: Declares TableGen def `zos___LTRC_B`.
  **L3453 CN**: 声明 TableGen def `zos___LTRC_B`。
- **L3454 EN**: Declares TableGen def `zos___WSQT_B`.
  **L3454 CN**: 声明 TableGen def `zos___WSQT_B`。
- **L3455 EN**: Declares TableGen def `zos___FSQT_B`.
  **L3455 CN**: 声明 TableGen def `zos___FSQT_B`。
- **L3456 EN**: Declares TableGen def `zos___LSQT_B`.
  **L3456 CN**: 声明 TableGen def `zos___LSQT_B`。

### Lines 3457-3488

````tablegen
  def zos___SSIN_B : RuntimeLibcallImpl<SIN_F64, "@@SSIN@B">;
  def zos___FSIN_B : RuntimeLibcallImpl<SIN_F32, "@@FSIN@B">;
  def zos___LSIN_B : RuntimeLibcallImpl<SIN_F128, "@@LSIN@B">;
  def zos___ROUN_B : RuntimeLibcallImpl<ROUND_F64, "@@ROUN@B">;
  def zos___ROUNFB : RuntimeLibcallImpl<ROUND_F32, "@@ROUNFB">;
  def zos___ROUNLB : RuntimeLibcallImpl<ROUND_F128, "@@ROUNLB">;
  def zos___SRNT_B : RuntimeLibcallImpl<RINT_F64, "@@SRNT@B">;
  def zos___RINTFB : RuntimeLibcallImpl<RINT_F32, "@@RINTFB">;
  def zos___RINTLB : RuntimeLibcallImpl<RINT_F128, "@@RINTLB">;
  def zos___WFMD_B : RuntimeLibcallImpl<REM_F64, "@@WFMD@B">;
  def zos___FFMD_B : RuntimeLibcallImpl<REM_F32, "@@FFMD@B">;
  def zos___LFMD_B : RuntimeLibcallImpl<REM_F128, "@@LFMD@B">;
  def zos___WPOW_B : RuntimeLibcallImpl<POW_F64, "@@WPOW@B">;
  def zos___FPOW_B : RuntimeLibcallImpl<POW_F32, "@@FPOW@B">;
  def zos___LPOW_B : RuntimeLibcallImpl<POW_F128, "@@LPOW@B">;
  def zos___NBYI_B : RuntimeLibcallImpl<NEARBYINT_F64, "@@NBYI@B">;
  def zos___NBYIFB : RuntimeLibcallImpl<NEARBYINT_F32, "@@NBYIFB">;
  def zos___NBYILB : RuntimeLibcallImpl<NEARBYINT_F128, "@@NBYILB">;
  def zos___ROND_B : RuntimeLibcallImpl<LROUND_F64, "@@ROND@B">;
  def zos___FRND_B : RuntimeLibcallImpl<LROUND_F32, "@@FRND@B">;
  def zos___LRND_B : RuntimeLibcallImpl<LROUND_F128, "@@LRND@B">;
  def zos___LRNT_B : RuntimeLibcallImpl<LRINT_F64, "@@LRNT@B">;
  def zos___LRNTFB : RuntimeLibcallImpl<LRINT_F32, "@@LRNTFB">;
  def zos___LRNTLB : RuntimeLibcallImpl<LRINT_F128, "@@LRNTLB">;
  def zos___WLOG_B : RuntimeLibcallImpl<LOG_F64, "@@WLOG@B">;
  def zos___FLOG_B : RuntimeLibcallImpl<LOG_F32, "@@FLOG@B">;
  def zos___LLOG_B : RuntimeLibcallImpl<LOG_F128, "@@LLOG@B">;
  def zos___LOG2_B : RuntimeLibcallImpl<LOG2_F64, "@@LOG2@B">;
  def zos___FLG2_B : RuntimeLibcallImpl<LOG2_F32, "@@FLG2@B">;
  def zos___LLG2_B : RuntimeLibcallImpl<LOG2_F128, "@@LLG2@B">;
  def zos___WLG1_B : RuntimeLibcallImpl<LOG10_F64, "@@WLG1@B">;
  def zos___FLG1_B : RuntimeLibcallImpl<LOG10_F32, "@@FLG1@B">;
````
- **L3457 EN**: Declares TableGen def `zos___SSIN_B`.
  **L3457 CN**: 声明 TableGen def `zos___SSIN_B`。
- **L3458 EN**: Declares TableGen def `zos___FSIN_B`.
  **L3458 CN**: 声明 TableGen def `zos___FSIN_B`。
- **L3459 EN**: Declares TableGen def `zos___LSIN_B`.
  **L3459 CN**: 声明 TableGen def `zos___LSIN_B`。
- **L3460 EN**: Declares TableGen def `zos___ROUN_B`.
  **L3460 CN**: 声明 TableGen def `zos___ROUN_B`。
- **L3461 EN**: Declares TableGen def `zos___ROUNFB`.
  **L3461 CN**: 声明 TableGen def `zos___ROUNFB`。
- **L3462 EN**: Declares TableGen def `zos___ROUNLB`.
  **L3462 CN**: 声明 TableGen def `zos___ROUNLB`。
- **L3463 EN**: Declares TableGen def `zos___SRNT_B`.
  **L3463 CN**: 声明 TableGen def `zos___SRNT_B`。
- **L3464 EN**: Declares TableGen def `zos___RINTFB`.
  **L3464 CN**: 声明 TableGen def `zos___RINTFB`。
- **L3465 EN**: Declares TableGen def `zos___RINTLB`.
  **L3465 CN**: 声明 TableGen def `zos___RINTLB`。
- **L3466 EN**: Declares TableGen def `zos___WFMD_B`.
  **L3466 CN**: 声明 TableGen def `zos___WFMD_B`。
- **L3467 EN**: Declares TableGen def `zos___FFMD_B`.
  **L3467 CN**: 声明 TableGen def `zos___FFMD_B`。
- **L3468 EN**: Declares TableGen def `zos___LFMD_B`.
  **L3468 CN**: 声明 TableGen def `zos___LFMD_B`。
- **L3469 EN**: Declares TableGen def `zos___WPOW_B`.
  **L3469 CN**: 声明 TableGen def `zos___WPOW_B`。
- **L3470 EN**: Declares TableGen def `zos___FPOW_B`.
  **L3470 CN**: 声明 TableGen def `zos___FPOW_B`。
- **L3471 EN**: Declares TableGen def `zos___LPOW_B`.
  **L3471 CN**: 声明 TableGen def `zos___LPOW_B`。
- **L3472 EN**: Declares TableGen def `zos___NBYI_B`.
  **L3472 CN**: 声明 TableGen def `zos___NBYI_B`。
- **L3473 EN**: Declares TableGen def `zos___NBYIFB`.
  **L3473 CN**: 声明 TableGen def `zos___NBYIFB`。
- **L3474 EN**: Declares TableGen def `zos___NBYILB`.
  **L3474 CN**: 声明 TableGen def `zos___NBYILB`。
- **L3475 EN**: Declares TableGen def `zos___ROND_B`.
  **L3475 CN**: 声明 TableGen def `zos___ROND_B`。
- **L3476 EN**: Declares TableGen def `zos___FRND_B`.
  **L3476 CN**: 声明 TableGen def `zos___FRND_B`。
- **L3477 EN**: Declares TableGen def `zos___LRND_B`.
  **L3477 CN**: 声明 TableGen def `zos___LRND_B`。
- **L3478 EN**: Declares TableGen def `zos___LRNT_B`.
  **L3478 CN**: 声明 TableGen def `zos___LRNT_B`。
- **L3479 EN**: Declares TableGen def `zos___LRNTFB`.
  **L3479 CN**: 声明 TableGen def `zos___LRNTFB`。
- **L3480 EN**: Declares TableGen def `zos___LRNTLB`.
  **L3480 CN**: 声明 TableGen def `zos___LRNTLB`。
- **L3481 EN**: Declares TableGen def `zos___WLOG_B`.
  **L3481 CN**: 声明 TableGen def `zos___WLOG_B`。
- **L3482 EN**: Declares TableGen def `zos___FLOG_B`.
  **L3482 CN**: 声明 TableGen def `zos___FLOG_B`。
- **L3483 EN**: Declares TableGen def `zos___LLOG_B`.
  **L3483 CN**: 声明 TableGen def `zos___LLOG_B`。
- **L3484 EN**: Declares TableGen def `zos___LOG2_B`.
  **L3484 CN**: 声明 TableGen def `zos___LOG2_B`。
- **L3485 EN**: Declares TableGen def `zos___FLG2_B`.
  **L3485 CN**: 声明 TableGen def `zos___FLG2_B`。
- **L3486 EN**: Declares TableGen def `zos___LLG2_B`.
  **L3486 CN**: 声明 TableGen def `zos___LLG2_B`。
- **L3487 EN**: Declares TableGen def `zos___WLG1_B`.
  **L3487 CN**: 声明 TableGen def `zos___WLG1_B`。
- **L3488 EN**: Declares TableGen def `zos___FLG1_B`.
  **L3488 CN**: 声明 TableGen def `zos___FLG1_B`。

### Lines 3489-3520

````tablegen
  def zos___LLG1_B : RuntimeLibcallImpl<LOG10_F128, "@@LLG1@B">;
  def zos___LLRD_B : RuntimeLibcallImpl<LLROUND_F64, "@@LLRD@B">;
  def zos___LLRDFB : RuntimeLibcallImpl<LLROUND_F32, "@@LLRDFB">;
  def zos___LLRDLB : RuntimeLibcallImpl<LLROUND_F128, "@@LLRDLB">;
  def zos___LLRT_B : RuntimeLibcallImpl<LLRINT_F64, "@@LLRT@B">;
  def zos___LLRTFB : RuntimeLibcallImpl<LLRINT_F32, "@@LLRTFB">;
  def zos___LLRTLB : RuntimeLibcallImpl<LLRINT_F128, "@@LLRTLB">;
  def zos___SLXP_B : RuntimeLibcallImpl<LDEXP_F64, "@@SLXP@B">;
  def zos___FLXP_B : RuntimeLibcallImpl<LDEXP_F32, "@@FLXP@B">;
  def zos___LLXP_B : RuntimeLibcallImpl<LDEXP_F128, "@@LLXP@B">;
  def zos___SFXP_B : RuntimeLibcallImpl<FREXP_F64, "@@SFXP@B">;
  def zos___FFXP_B : RuntimeLibcallImpl<FREXP_F32, "@@FFXP@B">;
  def zos___LFXP_B : RuntimeLibcallImpl<FREXP_F128, "@@LFXP@B">;
  def zos___FMIN_B : RuntimeLibcallImpl<FMIN_F64, "@@FMIN@B">;
  def zos___FMINFB : RuntimeLibcallImpl<FMIN_F32, "@@FMINFB">;
  def zos___FMINLB : RuntimeLibcallImpl<FMIN_F128, "@@FMINLB">;
  def zos___FMA_B : RuntimeLibcallImpl<FMA_F64, "@@FMA@B">;
  def zos___FMAFB : RuntimeLibcallImpl<FMA_F32, "@@FMAFB">;
  def zos___FMALB : RuntimeLibcallImpl<FMA_F128, "@@FMALB">;
  def zos___FMAX_B : RuntimeLibcallImpl<FMAX_F64, "@@FMAX@B">;
  def zos___FMAXFB : RuntimeLibcallImpl<FMAX_F32, "@@FMAXFB">;
  def zos___FMAXLB : RuntimeLibcallImpl<FMAX_F128, "@@FMAXLB">;
  def zos___SFLR_B : RuntimeLibcallImpl<FLOOR_F64, "@@SFLR@B">;
  def zos___FFLR_B : RuntimeLibcallImpl<FLOOR_F32, "@@FFLR@B">;
  def zos___LFLR_B : RuntimeLibcallImpl<FLOOR_F128, "@@LFLR@B">;
  def zos___WEXP_B : RuntimeLibcallImpl<EXP_F64, "@@WEXP@B">;
  def zos___FEXP_B : RuntimeLibcallImpl<EXP_F32, "@@FEXP@B">;
  def zos___LEXP_B : RuntimeLibcallImpl<EXP_F128, "@@LEXP@B">;
  def zos___EXP2_B : RuntimeLibcallImpl<EXP2_F64, "@@EXP2@B">;
  def zos___FXP2_B : RuntimeLibcallImpl<EXP2_F32, "@@FXP2@B">;
  def zos___LXP2_B : RuntimeLibcallImpl<EXP2_F128, "@@LXP2@B">;
  def zos___SCOS_B : RuntimeLibcallImpl<COS_F64, "@@SCOS@B">;
````
- **L3489 EN**: Declares TableGen def `zos___LLG1_B`.
  **L3489 CN**: 声明 TableGen def `zos___LLG1_B`。
- **L3490 EN**: Declares TableGen def `zos___LLRD_B`.
  **L3490 CN**: 声明 TableGen def `zos___LLRD_B`。
- **L3491 EN**: Declares TableGen def `zos___LLRDFB`.
  **L3491 CN**: 声明 TableGen def `zos___LLRDFB`。
- **L3492 EN**: Declares TableGen def `zos___LLRDLB`.
  **L3492 CN**: 声明 TableGen def `zos___LLRDLB`。
- **L3493 EN**: Declares TableGen def `zos___LLRT_B`.
  **L3493 CN**: 声明 TableGen def `zos___LLRT_B`。
- **L3494 EN**: Declares TableGen def `zos___LLRTFB`.
  **L3494 CN**: 声明 TableGen def `zos___LLRTFB`。
- **L3495 EN**: Declares TableGen def `zos___LLRTLB`.
  **L3495 CN**: 声明 TableGen def `zos___LLRTLB`。
- **L3496 EN**: Declares TableGen def `zos___SLXP_B`.
  **L3496 CN**: 声明 TableGen def `zos___SLXP_B`。
- **L3497 EN**: Declares TableGen def `zos___FLXP_B`.
  **L3497 CN**: 声明 TableGen def `zos___FLXP_B`。
- **L3498 EN**: Declares TableGen def `zos___LLXP_B`.
  **L3498 CN**: 声明 TableGen def `zos___LLXP_B`。
- **L3499 EN**: Declares TableGen def `zos___SFXP_B`.
  **L3499 CN**: 声明 TableGen def `zos___SFXP_B`。
- **L3500 EN**: Declares TableGen def `zos___FFXP_B`.
  **L3500 CN**: 声明 TableGen def `zos___FFXP_B`。
- **L3501 EN**: Declares TableGen def `zos___LFXP_B`.
  **L3501 CN**: 声明 TableGen def `zos___LFXP_B`。
- **L3502 EN**: Declares TableGen def `zos___FMIN_B`.
  **L3502 CN**: 声明 TableGen def `zos___FMIN_B`。
- **L3503 EN**: Declares TableGen def `zos___FMINFB`.
  **L3503 CN**: 声明 TableGen def `zos___FMINFB`。
- **L3504 EN**: Declares TableGen def `zos___FMINLB`.
  **L3504 CN**: 声明 TableGen def `zos___FMINLB`。
- **L3505 EN**: Declares TableGen def `zos___FMA_B`.
  **L3505 CN**: 声明 TableGen def `zos___FMA_B`。
- **L3506 EN**: Declares TableGen def `zos___FMAFB`.
  **L3506 CN**: 声明 TableGen def `zos___FMAFB`。
- **L3507 EN**: Declares TableGen def `zos___FMALB`.
  **L3507 CN**: 声明 TableGen def `zos___FMALB`。
- **L3508 EN**: Declares TableGen def `zos___FMAX_B`.
  **L3508 CN**: 声明 TableGen def `zos___FMAX_B`。
- **L3509 EN**: Declares TableGen def `zos___FMAXFB`.
  **L3509 CN**: 声明 TableGen def `zos___FMAXFB`。
- **L3510 EN**: Declares TableGen def `zos___FMAXLB`.
  **L3510 CN**: 声明 TableGen def `zos___FMAXLB`。
- **L3511 EN**: Declares TableGen def `zos___SFLR_B`.
  **L3511 CN**: 声明 TableGen def `zos___SFLR_B`。
- **L3512 EN**: Declares TableGen def `zos___FFLR_B`.
  **L3512 CN**: 声明 TableGen def `zos___FFLR_B`。
- **L3513 EN**: Declares TableGen def `zos___LFLR_B`.
  **L3513 CN**: 声明 TableGen def `zos___LFLR_B`。
- **L3514 EN**: Declares TableGen def `zos___WEXP_B`.
  **L3514 CN**: 声明 TableGen def `zos___WEXP_B`。
- **L3515 EN**: Declares TableGen def `zos___FEXP_B`.
  **L3515 CN**: 声明 TableGen def `zos___FEXP_B`。
- **L3516 EN**: Declares TableGen def `zos___LEXP_B`.
  **L3516 CN**: 声明 TableGen def `zos___LEXP_B`。
- **L3517 EN**: Declares TableGen def `zos___EXP2_B`.
  **L3517 CN**: 声明 TableGen def `zos___EXP2_B`。
- **L3518 EN**: Declares TableGen def `zos___FXP2_B`.
  **L3518 CN**: 声明 TableGen def `zos___FXP2_B`。
- **L3519 EN**: Declares TableGen def `zos___LXP2_B`.
  **L3519 CN**: 声明 TableGen def `zos___LXP2_B`。
- **L3520 EN**: Declares TableGen def `zos___SCOS_B`.
  **L3520 CN**: 声明 TableGen def `zos___SCOS_B`。

### Lines 3521-3552

````tablegen
  def zos___FCOS_B : RuntimeLibcallImpl<COS_F32, "@@FCOS@B">;
  def zos___LCOS_B : RuntimeLibcallImpl<COS_F128, "@@LCOS@B">;
  def zos___DCPY_B : RuntimeLibcallImpl<COPYSIGN_F64, "@@DCPY@B">;
  def zos___FCPY_B : RuntimeLibcallImpl<COPYSIGN_F32, "@@FCPY@B">;
  def zos___LCPY_B : RuntimeLibcallImpl<COPYSIGN_F128, "@@LCPY@B">;
  def zos___SCEL_B : RuntimeLibcallImpl<CEIL_F64, "@@SCEL@B">;
  def zos___FCEL_B : RuntimeLibcallImpl<CEIL_F32, "@@FCEL@B">;
  def zos___LCEL_B : RuntimeLibcallImpl<CEIL_F128, "@@LCEL@B">;
  def zos___SCRT_B : RuntimeLibcallImpl<CBRT_F64, "@@SCRT@B">;
  def zos___FCBT_B : RuntimeLibcallImpl<CBRT_F32, "@@FCBT@B">;
  def zos___LCBT_B : RuntimeLibcallImpl<CBRT_F128, "@@LCBT@B">;
}

def isSystemZZOS : RuntimeLibcallPredicate<"(TT.isSystemZ() && TT.isOSzOS())">;
def isZOS : RuntimeLibcallPredicate<"TT.isOSzOS()">;
def SystemZZOSSystemLibrary
    : SystemRuntimeLibrary<
          isSystemZZOS, (add DefaultLibcallImpls64,
                            LibcallImpls<(add ZOSRuntimeLibcalls), isZOS>)>;

//===----------------------------------------------------------------------===//
// WebAssembly Runtime Libcalls
//===----------------------------------------------------------------------===//

// Define the emscripten name for return address helper.
// TODO: when implementing other Wasm backends, make this generic or only do
// this on emscripten depending on what they end up doing.
def emscripten_return_address : RuntimeLibcallImpl<RETURN_ADDRESS>;

def isWasm : RuntimeLibcallPredicate<"TT.isWasm()">;
def isOSEmscripten : RuntimeLibcallPredicate<[{TT.isOSEmscripten()}]>;

````
- **L3521 EN**: Declares TableGen def `zos___FCOS_B`.
  **L3521 CN**: 声明 TableGen def `zos___FCOS_B`。
- **L3522 EN**: Declares TableGen def `zos___LCOS_B`.
  **L3522 CN**: 声明 TableGen def `zos___LCOS_B`。
- **L3523 EN**: Declares TableGen def `zos___DCPY_B`.
  **L3523 CN**: 声明 TableGen def `zos___DCPY_B`。
- **L3524 EN**: Declares TableGen def `zos___FCPY_B`.
  **L3524 CN**: 声明 TableGen def `zos___FCPY_B`。
- **L3525 EN**: Declares TableGen def `zos___LCPY_B`.
  **L3525 CN**: 声明 TableGen def `zos___LCPY_B`。
- **L3526 EN**: Declares TableGen def `zos___SCEL_B`.
  **L3526 CN**: 声明 TableGen def `zos___SCEL_B`。
- **L3527 EN**: Declares TableGen def `zos___FCEL_B`.
  **L3527 CN**: 声明 TableGen def `zos___FCEL_B`。
- **L3528 EN**: Declares TableGen def `zos___LCEL_B`.
  **L3528 CN**: 声明 TableGen def `zos___LCEL_B`。
- **L3529 EN**: Declares TableGen def `zos___SCRT_B`.
  **L3529 CN**: 声明 TableGen def `zos___SCRT_B`。
- **L3530 EN**: Declares TableGen def `zos___FCBT_B`.
  **L3530 CN**: 声明 TableGen def `zos___FCBT_B`。
- **L3531 EN**: Declares TableGen def `zos___LCBT_B`.
  **L3531 CN**: 声明 TableGen def `zos___LCBT_B`。
- **L3532 EN**: Closes the current lexical scope or compound statement.
  **L3532 CN**: 结束当前词法作用域或复合语句块。
- **L3533 EN**: Blank line separating nearby declarations or logic blocks.
  **L3533 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3534 EN**: Declares TableGen def `isSystemZZOS`.
  **L3534 CN**: 声明 TableGen def `isSystemZZOS`。
- **L3535 EN**: Declares TableGen def `isZOS`.
  **L3535 CN**: 声明 TableGen def `isZOS`。
- **L3536 EN**: Declares TableGen def `SystemZZOSSystemLibrary`.
  **L3536 CN**: 声明 TableGen def `SystemZZOSSystemLibrary`。
- **L3537 EN**: Continues the surrounding expression or declaration: `: SystemRuntimeLibrary<`.
  **L3537 CN**: 继续构造周围的表达式或声明：`: SystemRuntimeLibrary<`。
- **L3538 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `isSystemZZOS, (add DefaultLibcallImpls64,`.
  **L3538 CN**: 继续一个多行参数列表、初始化器或聚合项：`isSystemZZOS, (add DefaultLibcallImpls64,`。
- **L3539 EN**: Executes a call or declaration centered on `LibcallImpls<`.
  **L3539 CN**: 执行以 `LibcallImpls<` 为核心的调用或声明。
- **L3540 EN**: Blank line separating nearby declarations or logic blocks.
  **L3540 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3541 EN**: Banner comment marking a file or section boundary.
  **L3541 CN**: 横幅注释，用于标记文件或章节边界。
- **L3542 EN**: Comment explains nearby logic, invariants, or intent: `WebAssembly Runtime Libcalls`.
  **L3542 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`WebAssembly Runtime Libcalls`。
- **L3543 EN**: Banner comment marking a file or section boundary.
  **L3543 CN**: 横幅注释，用于标记文件或章节边界。
- **L3544 EN**: Blank line separating nearby declarations or logic blocks.
  **L3544 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3545 EN**: Comment explains nearby logic, invariants, or intent: `Define the emscripten name for return address helper.`.
  **L3545 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Define the emscripten name for return address helper.`。
- **L3546 EN**: Comment records a pending task or caution: `TODO: when implementing other Wasm backends, make this generic or only do`.
  **L3546 CN**: 注释记录了待办事项或注意点：`TODO: when implementing other Wasm backends, make this generic or only do`。
- **L3547 EN**: Comment explains nearby logic, invariants, or intent: `this on emscripten depending on what they end up doing.`.
  **L3547 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`this on emscripten depending on what they end up doing.`。
- **L3548 EN**: Declares TableGen def `emscripten_return_address`.
  **L3548 CN**: 声明 TableGen def `emscripten_return_address`。
- **L3549 EN**: Blank line separating nearby declarations or logic blocks.
  **L3549 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3550 EN**: Declares TableGen def `isWasm`.
  **L3550 CN**: 声明 TableGen def `isWasm`。
- **L3551 EN**: Declares TableGen def `isOSEmscripten`.
  **L3551 CN**: 声明 TableGen def `isOSEmscripten`。
- **L3552 EN**: Blank line separating nearby declarations or logic blocks.
  **L3552 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 3553-3584

````tablegen
// Define the emscripten name for return address helper.
// TODO: when implementing other Wasm backends, make this generic or only do
// this on emscripten depending on what they end up doing.
def WasmSystemLibrary
    : SystemRuntimeLibrary<isWasm,
      (add DefaultRuntimeLibcallImpls, Int128RTLibcalls,
           CompilerRTOnlyInt64Libcalls, CompilerRTOnlyInt128Libcalls,
           exp10f, exp10,
           _Unwind_CallPersonality,
           emscripten_return_address,
           LibcallImpls<(add __small_printf,
                             __small_sprintf,
                             __small_fprintf), isOSEmscripten>,
           __stack_chk_fail, __stack_chk_guard)>;

//===----------------------------------------------------------------------===//
// Legacy Default Runtime Libcalls
//===----------------------------------------------------------------------===//

// TODO: Should make every target explicit.
def isDefaultLibcallArch : RuntimeLibcallPredicate<[{
  TT.isMIPS() || TT.isLoongArch() || TT.isVE() || TT.isBPF() ||
  TT.getArch() == Triple::csky || TT.getArch() == Triple::arc ||
  TT.getArch() == Triple::m68k ||  TT.getArch() == Triple::xtensa ||
  (TT.isSystemZ() && !TT.isOSzOS())
}]>;


def isArch64Bit : RuntimeLibcallPredicate<[{TT.isArch64Bit()}]>;
def LegacyDefaultSystemLibrary
    : SystemRuntimeLibrary<isDefaultLibcallArch,
      (add DefaultRuntimeLibcallImpls,
````
- **L3553 EN**: Comment explains nearby logic, invariants, or intent: `Define the emscripten name for return address helper.`.
  **L3553 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Define the emscripten name for return address helper.`。
- **L3554 EN**: Comment records a pending task or caution: `TODO: when implementing other Wasm backends, make this generic or only do`.
  **L3554 CN**: 注释记录了待办事项或注意点：`TODO: when implementing other Wasm backends, make this generic or only do`。
- **L3555 EN**: Comment explains nearby logic, invariants, or intent: `this on emscripten depending on what they end up doing.`.
  **L3555 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`this on emscripten depending on what they end up doing.`。
- **L3556 EN**: Declares TableGen def `WasmSystemLibrary`.
  **L3556 CN**: 声明 TableGen def `WasmSystemLibrary`。
- **L3557 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: SystemRuntimeLibrary<isWasm,`.
  **L3557 CN**: 继续一个多行参数列表、初始化器或聚合项：`: SystemRuntimeLibrary<isWasm,`。
- **L3558 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(add DefaultRuntimeLibcallImpls, Int128RTLibcalls,`.
  **L3558 CN**: 继续一个多行参数列表、初始化器或聚合项：`(add DefaultRuntimeLibcallImpls, Int128RTLibcalls,`。
- **L3559 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `CompilerRTOnlyInt64Libcalls, CompilerRTOnlyInt128Libcalls,`.
  **L3559 CN**: 继续一个多行参数列表、初始化器或聚合项：`CompilerRTOnlyInt64Libcalls, CompilerRTOnlyInt128Libcalls,`。
- **L3560 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `exp10f, exp10,`.
  **L3560 CN**: 继续一个多行参数列表、初始化器或聚合项：`exp10f, exp10,`。
- **L3561 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `_Unwind_CallPersonality,`.
  **L3561 CN**: 继续一个多行参数列表、初始化器或聚合项：`_Unwind_CallPersonality,`。
- **L3562 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `emscripten_return_address,`.
  **L3562 CN**: 继续一个多行参数列表、初始化器或聚合项：`emscripten_return_address,`。
- **L3563 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LibcallImpls<(add __small_printf,`.
  **L3563 CN**: 继续一个多行参数列表、初始化器或聚合项：`LibcallImpls<(add __small_printf,`。
- **L3564 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__small_sprintf,`.
  **L3564 CN**: 继续一个多行参数列表、初始化器或聚合项：`__small_sprintf,`。
- **L3565 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__small_fprintf), isOSEmscripten>,`.
  **L3565 CN**: 继续一个多行参数列表、初始化器或聚合项：`__small_fprintf), isOSEmscripten>,`。
- **L3566 EN**: Executes a standalone statement or declaration: `__stack_chk_fail, __stack_chk_guard)>;`.
  **L3566 CN**: 执行一条独立语句或声明：`__stack_chk_fail, __stack_chk_guard)>;`。
- **L3567 EN**: Blank line separating nearby declarations or logic blocks.
  **L3567 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3568 EN**: Banner comment marking a file or section boundary.
  **L3568 CN**: 横幅注释，用于标记文件或章节边界。
- **L3569 EN**: Comment explains nearby logic, invariants, or intent: `Legacy Default Runtime Libcalls`.
  **L3569 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Legacy Default Runtime Libcalls`。
- **L3570 EN**: Banner comment marking a file or section boundary.
  **L3570 CN**: 横幅注释，用于标记文件或章节边界。
- **L3571 EN**: Blank line separating nearby declarations or logic blocks.
  **L3571 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3572 EN**: Comment records a pending task or caution: `TODO: Should make every target explicit.`.
  **L3572 CN**: 注释记录了待办事项或注意点：`TODO: Should make every target explicit.`。
- **L3573 EN**: Declares TableGen def `isDefaultLibcallArch`.
  **L3573 CN**: 声明 TableGen def `isDefaultLibcallArch`。
- **L3574 EN**: Continues logic associated with callable symbol `isMIPS`.
  **L3574 CN**: 继续与可调用符号 `isMIPS` 相关的逻辑。
- **L3575 EN**: Continues logic associated with callable symbol `getArch`.
  **L3575 CN**: 继续与可调用符号 `getArch` 相关的逻辑。
- **L3576 EN**: Continues logic associated with callable symbol `getArch`.
  **L3576 CN**: 继续与可调用符号 `getArch` 相关的逻辑。
- **L3577 EN**: Continues logic associated with callable symbol `isSystemZ`.
  **L3577 CN**: 继续与可调用符号 `isSystemZ` 相关的逻辑。
- **L3578 EN**: Executes a standalone statement or declaration: `}]>;`.
  **L3578 CN**: 执行一条独立语句或声明：`}]>;`。
- **L3579 EN**: Blank line separating nearby declarations or logic blocks.
  **L3579 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3580 EN**: Blank line separating nearby declarations or logic blocks.
  **L3580 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3581 EN**: Declares TableGen def `isArch64Bit`.
  **L3581 CN**: 声明 TableGen def `isArch64Bit`。
- **L3582 EN**: Declares TableGen def `LegacyDefaultSystemLibrary`.
  **L3582 CN**: 声明 TableGen def `LegacyDefaultSystemLibrary`。
- **L3583 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: SystemRuntimeLibrary<isDefaultLibcallArch,`.
  **L3583 CN**: 继续一个多行参数列表、初始化器或聚合项：`: SystemRuntimeLibrary<isDefaultLibcallArch,`。
- **L3584 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(add DefaultRuntimeLibcallImpls,`.
  **L3584 CN**: 继续一个多行参数列表、初始化器或聚合项：`(add DefaultRuntimeLibcallImpls,`。

### Lines 3585-3616

````tablegen
         LibmHasSinCosF32, LibmHasSinCosF64, LibmHasSinCosF128,
         exp10f, exp10, exp10l_f128,
         __powisf2, __powidf2, __powitf2_f128,
         LibcallImpls<(add Int128RTLibcalls), isArch64Bit>,
         DefaultStackProtector, DefaultSafeStackGlobals
)>;

//===----------------------------------------------------------------------===//
// Vector math libraries
//===----------------------------------------------------------------------===//

//===----------------------------------------------------------------------===//
// Accelerate framework functions
//===----------------------------------------------------------------------===//

defset list<RuntimeLibcallImpl> ACCELERATE_VECFUNCS = {
  def vacosf : RuntimeLibcallImpl<ACOS_V4F32>;
  def vacoshf : RuntimeLibcallImpl<ACOSH_V4F32>;
  def vasinf : RuntimeLibcallImpl<ASIN_V4F32>;
  def vasinhf : RuntimeLibcallImpl<ASINH_V4F32>;
  def vatan2f : RuntimeLibcallImpl<ATAN2_V4F32>;
  def vatanf : RuntimeLibcallImpl<ATAN_V4F32>;
  def vatanhf : RuntimeLibcallImpl<ATANH_V4F32>;
  def vceilf : RuntimeLibcallImpl<CEIL_V4F32>;
  def vcosf : RuntimeLibcallImpl<COS_V4F32>;
  def vcoshf : RuntimeLibcallImpl<COSH_V4F32>;
  def vexpf : RuntimeLibcallImpl<EXP_V4F32>;
  def vexpm1f : RuntimeLibcallImpl<EXPM1_V4F32>;
  def vfabsf : RuntimeLibcallImpl<FABS_V4F32>;
  def vfloorf : RuntimeLibcallImpl<FLOOR_V4F32>;
  def vlog10f : RuntimeLibcallImpl<LOG10_V4F32>;
  def vlog1pf : RuntimeLibcallImpl<LOG1P_V4F32>;
````
- **L3585 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LibmHasSinCosF32, LibmHasSinCosF64, LibmHasSinCosF128,`.
  **L3585 CN**: 继续一个多行参数列表、初始化器或聚合项：`LibmHasSinCosF32, LibmHasSinCosF64, LibmHasSinCosF128,`。
- **L3586 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `exp10f, exp10, exp10l_f128,`.
  **L3586 CN**: 继续一个多行参数列表、初始化器或聚合项：`exp10f, exp10, exp10l_f128,`。
- **L3587 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__powisf2, __powidf2, __powitf2_f128,`.
  **L3587 CN**: 继续一个多行参数列表、初始化器或聚合项：`__powisf2, __powidf2, __powitf2_f128,`。
- **L3588 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LibcallImpls<(add Int128RTLibcalls), isArch64Bit>,`.
  **L3588 CN**: 继续一个多行参数列表、初始化器或聚合项：`LibcallImpls<(add Int128RTLibcalls), isArch64Bit>,`。
- **L3589 EN**: Continues the surrounding expression or declaration: `DefaultStackProtector, DefaultSafeStackGlobals`.
  **L3589 CN**: 继续构造周围的表达式或声明：`DefaultStackProtector, DefaultSafeStackGlobals`。
- **L3590 EN**: Executes a standalone statement or declaration: `)>;`.
  **L3590 CN**: 执行一条独立语句或声明：`)>;`。
- **L3591 EN**: Blank line separating nearby declarations or logic blocks.
  **L3591 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3592 EN**: Banner comment marking a file or section boundary.
  **L3592 CN**: 横幅注释，用于标记文件或章节边界。
- **L3593 EN**: Comment explains nearby logic, invariants, or intent: `Vector math libraries`.
  **L3593 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Vector math libraries`。
- **L3594 EN**: Banner comment marking a file or section boundary.
  **L3594 CN**: 横幅注释，用于标记文件或章节边界。
- **L3595 EN**: Blank line separating nearby declarations or logic blocks.
  **L3595 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3596 EN**: Banner comment marking a file or section boundary.
  **L3596 CN**: 横幅注释，用于标记文件或章节边界。
- **L3597 EN**: Comment explains nearby logic, invariants, or intent: `Accelerate framework functions`.
  **L3597 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Accelerate framework functions`。
- **L3598 EN**: Banner comment marking a file or section boundary.
  **L3598 CN**: 横幅注释，用于标记文件或章节边界。
- **L3599 EN**: Blank line separating nearby declarations or logic blocks.
  **L3599 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3600 EN**: Continues the surrounding expression or declaration: `defset list<RuntimeLibcallImpl> ACCELERATE_VECFUNCS = {`.
  **L3600 CN**: 继续构造周围的表达式或声明：`defset list<RuntimeLibcallImpl> ACCELERATE_VECFUNCS = {`。
- **L3601 EN**: Declares TableGen def `vacosf`.
  **L3601 CN**: 声明 TableGen def `vacosf`。
- **L3602 EN**: Declares TableGen def `vacoshf`.
  **L3602 CN**: 声明 TableGen def `vacoshf`。
- **L3603 EN**: Declares TableGen def `vasinf`.
  **L3603 CN**: 声明 TableGen def `vasinf`。
- **L3604 EN**: Declares TableGen def `vasinhf`.
  **L3604 CN**: 声明 TableGen def `vasinhf`。
- **L3605 EN**: Declares TableGen def `vatan2f`.
  **L3605 CN**: 声明 TableGen def `vatan2f`。
- **L3606 EN**: Declares TableGen def `vatanf`.
  **L3606 CN**: 声明 TableGen def `vatanf`。
- **L3607 EN**: Declares TableGen def `vatanhf`.
  **L3607 CN**: 声明 TableGen def `vatanhf`。
- **L3608 EN**: Declares TableGen def `vceilf`.
  **L3608 CN**: 声明 TableGen def `vceilf`。
- **L3609 EN**: Declares TableGen def `vcosf`.
  **L3609 CN**: 声明 TableGen def `vcosf`。
- **L3610 EN**: Declares TableGen def `vcoshf`.
  **L3610 CN**: 声明 TableGen def `vcoshf`。
- **L3611 EN**: Declares TableGen def `vexpf`.
  **L3611 CN**: 声明 TableGen def `vexpf`。
- **L3612 EN**: Declares TableGen def `vexpm1f`.
  **L3612 CN**: 声明 TableGen def `vexpm1f`。
- **L3613 EN**: Declares TableGen def `vfabsf`.
  **L3613 CN**: 声明 TableGen def `vfabsf`。
- **L3614 EN**: Declares TableGen def `vfloorf`.
  **L3614 CN**: 声明 TableGen def `vfloorf`。
- **L3615 EN**: Declares TableGen def `vlog10f`.
  **L3615 CN**: 声明 TableGen def `vlog10f`。
- **L3616 EN**: Declares TableGen def `vlog1pf`.
  **L3616 CN**: 声明 TableGen def `vlog1pf`。

### Lines 3617-3648

````tablegen
  def vlogbf : RuntimeLibcallImpl<LOGB_V4F32>;
  def vlogf : RuntimeLibcallImpl<LOG_V4F32>;
  def vsinf : RuntimeLibcallImpl<SIN_V4F32>;
  def vsinhf : RuntimeLibcallImpl<SINH_V4F32>;
  def vsqrtf : RuntimeLibcallImpl<SQRT_V4F32>;
  def vtanf : RuntimeLibcallImpl<TAN_V4F32>;
  def vtanhf : RuntimeLibcallImpl<TANH_V4F32>;
}

//===----------------------------------------------------------------------===//
// Darwin_libsystem_m vector functions
//===----------------------------------------------------------------------===//

defset list<RuntimeLibcallImpl> DARWIN_LIBSYSTEM_M_VECFUNCS = {
  def _simd_acos_d2 : RuntimeLibcallImpl<ACOS_V2F64>;
  def _simd_acos_f4 : RuntimeLibcallImpl<ACOS_V4F32>;
  def _simd_acosh_d2 : RuntimeLibcallImpl<ACOSH_V2F64>;
  def _simd_acosh_f4 : RuntimeLibcallImpl<ACOSH_V4F32>;
  def _simd_asin_d2 : RuntimeLibcallImpl<ASIN_V2F64>;
  def _simd_asin_f4 : RuntimeLibcallImpl<ASIN_V4F32>;
  def _simd_asinh_d2 : RuntimeLibcallImpl<ASINH_V2F64>;
  def _simd_asinh_f4 : RuntimeLibcallImpl<ASINH_V4F32>;
  def _simd_atan2_d2 : RuntimeLibcallImpl<ATAN2_V2F64>;
  def _simd_atan2_f4 : RuntimeLibcallImpl<ATAN2_V4F32>;
  def _simd_atan_d2 : RuntimeLibcallImpl<ATAN_V2F64>;
  def _simd_atan_f4 : RuntimeLibcallImpl<ATAN_V4F32>;
  def _simd_atanh_d2 : RuntimeLibcallImpl<ATANH_V2F64>;
  def _simd_atanh_f4 : RuntimeLibcallImpl<ATANH_V4F32>;
  def _simd_cbrt_d2 : RuntimeLibcallImpl<CBRT_V2F64>;
  def _simd_cbrt_f4 : RuntimeLibcallImpl<CBRT_V4F32>;
  def _simd_cos_d2 : RuntimeLibcallImpl<COS_V2F64>;
  def _simd_cos_f4 : RuntimeLibcallImpl<COS_V4F32>;
````
- **L3617 EN**: Declares TableGen def `vlogbf`.
  **L3617 CN**: 声明 TableGen def `vlogbf`。
- **L3618 EN**: Declares TableGen def `vlogf`.
  **L3618 CN**: 声明 TableGen def `vlogf`。
- **L3619 EN**: Declares TableGen def `vsinf`.
  **L3619 CN**: 声明 TableGen def `vsinf`。
- **L3620 EN**: Declares TableGen def `vsinhf`.
  **L3620 CN**: 声明 TableGen def `vsinhf`。
- **L3621 EN**: Declares TableGen def `vsqrtf`.
  **L3621 CN**: 声明 TableGen def `vsqrtf`。
- **L3622 EN**: Declares TableGen def `vtanf`.
  **L3622 CN**: 声明 TableGen def `vtanf`。
- **L3623 EN**: Declares TableGen def `vtanhf`.
  **L3623 CN**: 声明 TableGen def `vtanhf`。
- **L3624 EN**: Closes the current lexical scope or compound statement.
  **L3624 CN**: 结束当前词法作用域或复合语句块。
- **L3625 EN**: Blank line separating nearby declarations or logic blocks.
  **L3625 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3626 EN**: Banner comment marking a file or section boundary.
  **L3626 CN**: 横幅注释，用于标记文件或章节边界。
- **L3627 EN**: Comment explains nearby logic, invariants, or intent: `Darwin_libsystem_m vector functions`.
  **L3627 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Darwin_libsystem_m vector functions`。
- **L3628 EN**: Banner comment marking a file or section boundary.
  **L3628 CN**: 横幅注释，用于标记文件或章节边界。
- **L3629 EN**: Blank line separating nearby declarations or logic blocks.
  **L3629 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3630 EN**: Continues the surrounding expression or declaration: `defset list<RuntimeLibcallImpl> DARWIN_LIBSYSTEM_M_VECFUNCS = {`.
  **L3630 CN**: 继续构造周围的表达式或声明：`defset list<RuntimeLibcallImpl> DARWIN_LIBSYSTEM_M_VECFUNCS = {`。
- **L3631 EN**: Declares TableGen def `_simd_acos_d2`.
  **L3631 CN**: 声明 TableGen def `_simd_acos_d2`。
- **L3632 EN**: Declares TableGen def `_simd_acos_f4`.
  **L3632 CN**: 声明 TableGen def `_simd_acos_f4`。
- **L3633 EN**: Declares TableGen def `_simd_acosh_d2`.
  **L3633 CN**: 声明 TableGen def `_simd_acosh_d2`。
- **L3634 EN**: Declares TableGen def `_simd_acosh_f4`.
  **L3634 CN**: 声明 TableGen def `_simd_acosh_f4`。
- **L3635 EN**: Declares TableGen def `_simd_asin_d2`.
  **L3635 CN**: 声明 TableGen def `_simd_asin_d2`。
- **L3636 EN**: Declares TableGen def `_simd_asin_f4`.
  **L3636 CN**: 声明 TableGen def `_simd_asin_f4`。
- **L3637 EN**: Declares TableGen def `_simd_asinh_d2`.
  **L3637 CN**: 声明 TableGen def `_simd_asinh_d2`。
- **L3638 EN**: Declares TableGen def `_simd_asinh_f4`.
  **L3638 CN**: 声明 TableGen def `_simd_asinh_f4`。
- **L3639 EN**: Declares TableGen def `_simd_atan2_d2`.
  **L3639 CN**: 声明 TableGen def `_simd_atan2_d2`。
- **L3640 EN**: Declares TableGen def `_simd_atan2_f4`.
  **L3640 CN**: 声明 TableGen def `_simd_atan2_f4`。
- **L3641 EN**: Declares TableGen def `_simd_atan_d2`.
  **L3641 CN**: 声明 TableGen def `_simd_atan_d2`。
- **L3642 EN**: Declares TableGen def `_simd_atan_f4`.
  **L3642 CN**: 声明 TableGen def `_simd_atan_f4`。
- **L3643 EN**: Declares TableGen def `_simd_atanh_d2`.
  **L3643 CN**: 声明 TableGen def `_simd_atanh_d2`。
- **L3644 EN**: Declares TableGen def `_simd_atanh_f4`.
  **L3644 CN**: 声明 TableGen def `_simd_atanh_f4`。
- **L3645 EN**: Declares TableGen def `_simd_cbrt_d2`.
  **L3645 CN**: 声明 TableGen def `_simd_cbrt_d2`。
- **L3646 EN**: Declares TableGen def `_simd_cbrt_f4`.
  **L3646 CN**: 声明 TableGen def `_simd_cbrt_f4`。
- **L3647 EN**: Declares TableGen def `_simd_cos_d2`.
  **L3647 CN**: 声明 TableGen def `_simd_cos_d2`。
- **L3648 EN**: Declares TableGen def `_simd_cos_f4`.
  **L3648 CN**: 声明 TableGen def `_simd_cos_f4`。

### Lines 3649-3680

````tablegen
  def _simd_cosh_d2 : RuntimeLibcallImpl<COSH_V2F64>;
  def _simd_cosh_f4 : RuntimeLibcallImpl<COSH_V4F32>;
  def _simd_erf_d2 : RuntimeLibcallImpl<ERF_V2F64>;
  def _simd_erf_f4 : RuntimeLibcallImpl<ERF_V4F32>;
  def _simd_exp_d2 : RuntimeLibcallImpl<EXP_V2F64>;
  def _simd_exp_f4 : RuntimeLibcallImpl<EXP_V4F32>;
  def _simd_pow_d2 : RuntimeLibcallImpl<POW_V2F64>;
  def _simd_pow_f4 : RuntimeLibcallImpl<POW_V4F32>;
  def _simd_sin_d2 : RuntimeLibcallImpl<SIN_V2F64>;
  def _simd_sin_f4 : RuntimeLibcallImpl<SIN_V4F32>;
  def _simd_sinh_d2 : RuntimeLibcallImpl<SINH_V2F64>;
  def _simd_sinh_f4 : RuntimeLibcallImpl<SINH_V4F32>;
  def _simd_tan_d2 : RuntimeLibcallImpl<TAN_V2F64>;
  def _simd_tan_f4 : RuntimeLibcallImpl<TAN_V4F32>;
  def _simd_tanh_d2 : RuntimeLibcallImpl<TANH_V2F64>;
  def _simd_tanh_f4 : RuntimeLibcallImpl<TANH_V4F32>;
}

//===----------------------------------------------------------------------===//
// GLIBC Vector Math library LIBMVEC functions
//===----------------------------------------------------------------------===//

defvar LIBMVECPrefix = "LIBMVEC_";

class LibmvecLibcall<RuntimeLibcall P>
    : RuntimeLibcallImpl<P, !substr(NAME, !size(LIBMVECPrefix))>;

defset list<RuntimeLibcallImpl> LIBMVEC_X86_VECFUNCS = {
  def LIBMVEC__ZGVbN2v___exp_finite : LibmvecLibcall<EXP_FINITE_V2F64>;
  def LIBMVEC__ZGVbN2v___log_finite : LibmvecLibcall<LOG_FINITE_V2F64>;
  def LIBMVEC__ZGVbN2v_cos : LibmvecLibcall<COS_V2F64>;
  def LIBMVEC__ZGVbN2v_exp : LibmvecLibcall<EXP_V2F64>;
````
- **L3649 EN**: Declares TableGen def `_simd_cosh_d2`.
  **L3649 CN**: 声明 TableGen def `_simd_cosh_d2`。
- **L3650 EN**: Declares TableGen def `_simd_cosh_f4`.
  **L3650 CN**: 声明 TableGen def `_simd_cosh_f4`。
- **L3651 EN**: Declares TableGen def `_simd_erf_d2`.
  **L3651 CN**: 声明 TableGen def `_simd_erf_d2`。
- **L3652 EN**: Declares TableGen def `_simd_erf_f4`.
  **L3652 CN**: 声明 TableGen def `_simd_erf_f4`。
- **L3653 EN**: Declares TableGen def `_simd_exp_d2`.
  **L3653 CN**: 声明 TableGen def `_simd_exp_d2`。
- **L3654 EN**: Declares TableGen def `_simd_exp_f4`.
  **L3654 CN**: 声明 TableGen def `_simd_exp_f4`。
- **L3655 EN**: Declares TableGen def `_simd_pow_d2`.
  **L3655 CN**: 声明 TableGen def `_simd_pow_d2`。
- **L3656 EN**: Declares TableGen def `_simd_pow_f4`.
  **L3656 CN**: 声明 TableGen def `_simd_pow_f4`。
- **L3657 EN**: Declares TableGen def `_simd_sin_d2`.
  **L3657 CN**: 声明 TableGen def `_simd_sin_d2`。
- **L3658 EN**: Declares TableGen def `_simd_sin_f4`.
  **L3658 CN**: 声明 TableGen def `_simd_sin_f4`。
- **L3659 EN**: Declares TableGen def `_simd_sinh_d2`.
  **L3659 CN**: 声明 TableGen def `_simd_sinh_d2`。
- **L3660 EN**: Declares TableGen def `_simd_sinh_f4`.
  **L3660 CN**: 声明 TableGen def `_simd_sinh_f4`。
- **L3661 EN**: Declares TableGen def `_simd_tan_d2`.
  **L3661 CN**: 声明 TableGen def `_simd_tan_d2`。
- **L3662 EN**: Declares TableGen def `_simd_tan_f4`.
  **L3662 CN**: 声明 TableGen def `_simd_tan_f4`。
- **L3663 EN**: Declares TableGen def `_simd_tanh_d2`.
  **L3663 CN**: 声明 TableGen def `_simd_tanh_d2`。
- **L3664 EN**: Declares TableGen def `_simd_tanh_f4`.
  **L3664 CN**: 声明 TableGen def `_simd_tanh_f4`。
- **L3665 EN**: Closes the current lexical scope or compound statement.
  **L3665 CN**: 结束当前词法作用域或复合语句块。
- **L3666 EN**: Blank line separating nearby declarations or logic blocks.
  **L3666 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3667 EN**: Banner comment marking a file or section boundary.
  **L3667 CN**: 横幅注释，用于标记文件或章节边界。
- **L3668 EN**: Comment explains nearby logic, invariants, or intent: `GLIBC Vector Math library LIBMVEC functions`.
  **L3668 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`GLIBC Vector Math library LIBMVEC functions`。
- **L3669 EN**: Banner comment marking a file or section boundary.
  **L3669 CN**: 横幅注释，用于标记文件或章节边界。
- **L3670 EN**: Blank line separating nearby declarations or logic blocks.
  **L3670 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3671 EN**: Initializes variable `LIBMVECPrefix` from the right-hand expression.
  **L3671 CN**: 使用右侧表达式初始化变量 `LIBMVECPrefix`。
- **L3672 EN**: Blank line separating nearby declarations or logic blocks.
  **L3672 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3673 EN**: Declares class `LibmvecLibcall<RuntimeLibcall`.
  **L3673 CN**: 声明 class `LibmvecLibcall<RuntimeLibcall`。
- **L3674 EN**: Executes a call or declaration centered on `!substr`.
  **L3674 CN**: 执行以 `!substr` 为核心的调用或声明。
- **L3675 EN**: Blank line separating nearby declarations or logic blocks.
  **L3675 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3676 EN**: Continues the surrounding expression or declaration: `defset list<RuntimeLibcallImpl> LIBMVEC_X86_VECFUNCS = {`.
  **L3676 CN**: 继续构造周围的表达式或声明：`defset list<RuntimeLibcallImpl> LIBMVEC_X86_VECFUNCS = {`。
- **L3677 EN**: Declares TableGen def `LIBMVEC__ZGVbN2v___exp_finite`.
  **L3677 CN**: 声明 TableGen def `LIBMVEC__ZGVbN2v___exp_finite`。
- **L3678 EN**: Declares TableGen def `LIBMVEC__ZGVbN2v___log_finite`.
  **L3678 CN**: 声明 TableGen def `LIBMVEC__ZGVbN2v___log_finite`。
- **L3679 EN**: Declares TableGen def `LIBMVEC__ZGVbN2v_cos`.
  **L3679 CN**: 声明 TableGen def `LIBMVEC__ZGVbN2v_cos`。
- **L3680 EN**: Declares TableGen def `LIBMVEC__ZGVbN2v_exp`.
  **L3680 CN**: 声明 TableGen def `LIBMVEC__ZGVbN2v_exp`。

### Lines 3681-3712

````tablegen
  def LIBMVEC__ZGVbN2v_log : LibmvecLibcall<LOG_V2F64>;
  def LIBMVEC__ZGVbN2v_sin : LibmvecLibcall<SIN_V2F64>;
  def LIBMVEC__ZGVbN2v_tan : LibmvecLibcall<TAN_V2F64>;
  def LIBMVEC__ZGVbN2vv___pow_finite : LibmvecLibcall<POW_FINITE_V2F64>;
  def LIBMVEC__ZGVbN2vv_pow : LibmvecLibcall<POW_V2F64>;
  def LIBMVEC__ZGVbN4v___expf_finite : LibmvecLibcall<EXP_FINITE_V4F64>;
  def LIBMVEC__ZGVbN4v___logf_finite : LibmvecLibcall<LOG_FINITE_V4F64>;
  def LIBMVEC__ZGVbN4v_cosf : LibmvecLibcall<COS_V4F32>;
  def LIBMVEC__ZGVbN4v_expf : LibmvecLibcall<EXP_V4F32>;
  def LIBMVEC__ZGVbN4v_logf : LibmvecLibcall<LOG_V4F32>;
  def LIBMVEC__ZGVbN4v_sinf : LibmvecLibcall<SIN_V4F32>;
  def LIBMVEC__ZGVbN4v_tanf : LibmvecLibcall<TAN_V4F32>;
  def LIBMVEC__ZGVbN4vv___powf_finite : LibmvecLibcall<POW_FINITE_V4F64>;
  def LIBMVEC__ZGVbN4vv_powf : LibmvecLibcall<POW_V4F32>;
  def LIBMVEC__ZGVdN4v___exp_finite : LibmvecLibcall<EXP_FINITE_V4F64>;
  def LIBMVEC__ZGVdN4v___log_finite : LibmvecLibcall<LOG_FINITE_V4F64>;
  def LIBMVEC__ZGVdN4v_cos : LibmvecLibcall<COS_V4F64>;
  def LIBMVEC__ZGVdN4v_exp : LibmvecLibcall<EXP_V4F64>;
  def LIBMVEC__ZGVdN4v_log : LibmvecLibcall<LOG_V4F64>;
  def LIBMVEC__ZGVdN4v_sin : LibmvecLibcall<SIN_V4F64>;
  def LIBMVEC__ZGVdN4v_tan : LibmvecLibcall<TAN_V4F64>;
  def LIBMVEC__ZGVdN4vv___pow_finite : LibmvecLibcall<POW_FINITE_V4F64>;
  def LIBMVEC__ZGVdN4vv_pow : LibmvecLibcall<POW_V4F64>;
  def LIBMVEC__ZGVdN8v___expf_finite : LibmvecLibcall<EXP_FINITE_V8F64>;
  def LIBMVEC__ZGVdN8v___logf_finite : LibmvecLibcall<LOG_FINITE_V8F64>;
  def LIBMVEC__ZGVdN8v_cosf : LibmvecLibcall<COS_V8F32>;
  def LIBMVEC__ZGVdN8v_expf : LibmvecLibcall<EXP_V8F32>;
  def LIBMVEC__ZGVdN8v_logf : LibmvecLibcall<LOG_V8F32>;
  def LIBMVEC__ZGVdN8v_sinf : LibmvecLibcall<SIN_V8F32>;
  def LIBMVEC__ZGVdN8v_tanf : LibmvecLibcall<TAN_V8F32>;
  def LIBMVEC__ZGVdN8vv___powf_finite : LibmvecLibcall<POW_FINITE_V8F64>;
  def LIBMVEC__ZGVdN8vv_powf : LibmvecLibcall<POW_V8F32>;
````
- **L3681 EN**: Declares TableGen def `LIBMVEC__ZGVbN2v_log`.
  **L3681 CN**: 声明 TableGen def `LIBMVEC__ZGVbN2v_log`。
- **L3682 EN**: Declares TableGen def `LIBMVEC__ZGVbN2v_sin`.
  **L3682 CN**: 声明 TableGen def `LIBMVEC__ZGVbN2v_sin`。
- **L3683 EN**: Declares TableGen def `LIBMVEC__ZGVbN2v_tan`.
  **L3683 CN**: 声明 TableGen def `LIBMVEC__ZGVbN2v_tan`。
- **L3684 EN**: Declares TableGen def `LIBMVEC__ZGVbN2vv___pow_finite`.
  **L3684 CN**: 声明 TableGen def `LIBMVEC__ZGVbN2vv___pow_finite`。
- **L3685 EN**: Declares TableGen def `LIBMVEC__ZGVbN2vv_pow`.
  **L3685 CN**: 声明 TableGen def `LIBMVEC__ZGVbN2vv_pow`。
- **L3686 EN**: Declares TableGen def `LIBMVEC__ZGVbN4v___expf_finite`.
  **L3686 CN**: 声明 TableGen def `LIBMVEC__ZGVbN4v___expf_finite`。
- **L3687 EN**: Declares TableGen def `LIBMVEC__ZGVbN4v___logf_finite`.
  **L3687 CN**: 声明 TableGen def `LIBMVEC__ZGVbN4v___logf_finite`。
- **L3688 EN**: Declares TableGen def `LIBMVEC__ZGVbN4v_cosf`.
  **L3688 CN**: 声明 TableGen def `LIBMVEC__ZGVbN4v_cosf`。
- **L3689 EN**: Declares TableGen def `LIBMVEC__ZGVbN4v_expf`.
  **L3689 CN**: 声明 TableGen def `LIBMVEC__ZGVbN4v_expf`。
- **L3690 EN**: Declares TableGen def `LIBMVEC__ZGVbN4v_logf`.
  **L3690 CN**: 声明 TableGen def `LIBMVEC__ZGVbN4v_logf`。
- **L3691 EN**: Declares TableGen def `LIBMVEC__ZGVbN4v_sinf`.
  **L3691 CN**: 声明 TableGen def `LIBMVEC__ZGVbN4v_sinf`。
- **L3692 EN**: Declares TableGen def `LIBMVEC__ZGVbN4v_tanf`.
  **L3692 CN**: 声明 TableGen def `LIBMVEC__ZGVbN4v_tanf`。
- **L3693 EN**: Declares TableGen def `LIBMVEC__ZGVbN4vv___powf_finite`.
  **L3693 CN**: 声明 TableGen def `LIBMVEC__ZGVbN4vv___powf_finite`。
- **L3694 EN**: Declares TableGen def `LIBMVEC__ZGVbN4vv_powf`.
  **L3694 CN**: 声明 TableGen def `LIBMVEC__ZGVbN4vv_powf`。
- **L3695 EN**: Declares TableGen def `LIBMVEC__ZGVdN4v___exp_finite`.
  **L3695 CN**: 声明 TableGen def `LIBMVEC__ZGVdN4v___exp_finite`。
- **L3696 EN**: Declares TableGen def `LIBMVEC__ZGVdN4v___log_finite`.
  **L3696 CN**: 声明 TableGen def `LIBMVEC__ZGVdN4v___log_finite`。
- **L3697 EN**: Declares TableGen def `LIBMVEC__ZGVdN4v_cos`.
  **L3697 CN**: 声明 TableGen def `LIBMVEC__ZGVdN4v_cos`。
- **L3698 EN**: Declares TableGen def `LIBMVEC__ZGVdN4v_exp`.
  **L3698 CN**: 声明 TableGen def `LIBMVEC__ZGVdN4v_exp`。
- **L3699 EN**: Declares TableGen def `LIBMVEC__ZGVdN4v_log`.
  **L3699 CN**: 声明 TableGen def `LIBMVEC__ZGVdN4v_log`。
- **L3700 EN**: Declares TableGen def `LIBMVEC__ZGVdN4v_sin`.
  **L3700 CN**: 声明 TableGen def `LIBMVEC__ZGVdN4v_sin`。
- **L3701 EN**: Declares TableGen def `LIBMVEC__ZGVdN4v_tan`.
  **L3701 CN**: 声明 TableGen def `LIBMVEC__ZGVdN4v_tan`。
- **L3702 EN**: Declares TableGen def `LIBMVEC__ZGVdN4vv___pow_finite`.
  **L3702 CN**: 声明 TableGen def `LIBMVEC__ZGVdN4vv___pow_finite`。
- **L3703 EN**: Declares TableGen def `LIBMVEC__ZGVdN4vv_pow`.
  **L3703 CN**: 声明 TableGen def `LIBMVEC__ZGVdN4vv_pow`。
- **L3704 EN**: Declares TableGen def `LIBMVEC__ZGVdN8v___expf_finite`.
  **L3704 CN**: 声明 TableGen def `LIBMVEC__ZGVdN8v___expf_finite`。
- **L3705 EN**: Declares TableGen def `LIBMVEC__ZGVdN8v___logf_finite`.
  **L3705 CN**: 声明 TableGen def `LIBMVEC__ZGVdN8v___logf_finite`。
- **L3706 EN**: Declares TableGen def `LIBMVEC__ZGVdN8v_cosf`.
  **L3706 CN**: 声明 TableGen def `LIBMVEC__ZGVdN8v_cosf`。
- **L3707 EN**: Declares TableGen def `LIBMVEC__ZGVdN8v_expf`.
  **L3707 CN**: 声明 TableGen def `LIBMVEC__ZGVdN8v_expf`。
- **L3708 EN**: Declares TableGen def `LIBMVEC__ZGVdN8v_logf`.
  **L3708 CN**: 声明 TableGen def `LIBMVEC__ZGVdN8v_logf`。
- **L3709 EN**: Declares TableGen def `LIBMVEC__ZGVdN8v_sinf`.
  **L3709 CN**: 声明 TableGen def `LIBMVEC__ZGVdN8v_sinf`。
- **L3710 EN**: Declares TableGen def `LIBMVEC__ZGVdN8v_tanf`.
  **L3710 CN**: 声明 TableGen def `LIBMVEC__ZGVdN8v_tanf`。
- **L3711 EN**: Declares TableGen def `LIBMVEC__ZGVdN8vv___powf_finite`.
  **L3711 CN**: 声明 TableGen def `LIBMVEC__ZGVdN8vv___powf_finite`。
- **L3712 EN**: Declares TableGen def `LIBMVEC__ZGVdN8vv_powf`.
  **L3712 CN**: 声明 TableGen def `LIBMVEC__ZGVdN8vv_powf`。

### Lines 3713-3744

````tablegen
}

defset list<RuntimeLibcallImpl> LIBMVEC_AARCH64_VECFUNCS = {
  def LIBMVEC__ZGVnN2v_acos : LibmvecLibcall<ACOS_V2F64>;
  def LIBMVEC__ZGVnN2v_acosf : LibmvecLibcall<ACOS_V2F32>;
  def LIBMVEC__ZGVnN2v_acosh : LibmvecLibcall<ACOSH_V2F64>;
  def LIBMVEC__ZGVnN2v_acoshf : LibmvecLibcall<ACOSH_V2F32>;
  def LIBMVEC__ZGVnN2v_asin : LibmvecLibcall<ASIN_V2F64>;
  def LIBMVEC__ZGVnN2v_asinf : LibmvecLibcall<ASIN_V2F32>;
  def LIBMVEC__ZGVnN2v_asinh : LibmvecLibcall<ASINH_V2F64>;
  def LIBMVEC__ZGVnN2v_asinhf : LibmvecLibcall<ASINH_V2F32>;
  def LIBMVEC__ZGVnN2v_atan : LibmvecLibcall<ATAN_V2F64>;
  def LIBMVEC__ZGVnN2v_atanf : LibmvecLibcall<ATAN_V2F32>;
  def LIBMVEC__ZGVnN2v_atanh : LibmvecLibcall<ATANH_V2F64>;
  def LIBMVEC__ZGVnN2v_atanhf : LibmvecLibcall<ATANH_V2F32>;
  def LIBMVEC__ZGVnN2v_cbrt : LibmvecLibcall<CBRT_V2F64>;
  def LIBMVEC__ZGVnN2v_cbrtf : LibmvecLibcall<CBRT_V2F32>;
  def LIBMVEC__ZGVnN2v_cos : LibmvecLibcall<COS_V2F64>;
  def LIBMVEC__ZGVnN2v_cosf : LibmvecLibcall<COS_V2F32>;
  def LIBMVEC__ZGVnN2v_cosh : LibmvecLibcall<COSH_V2F64>;
  def LIBMVEC__ZGVnN2v_coshf : LibmvecLibcall<COSH_V2F32>;
  def LIBMVEC__ZGVnN2v_erf : LibmvecLibcall<ERF_V2F64>;
  def LIBMVEC__ZGVnN2v_erfc : LibmvecLibcall<ERFC_V2F64>;
  def LIBMVEC__ZGVnN2v_erfcf : LibmvecLibcall<ERFC_V2F32>;
  def LIBMVEC__ZGVnN2v_erff : LibmvecLibcall<ERF_V2F32>;
  def LIBMVEC__ZGVnN2v_exp : LibmvecLibcall<EXP_V2F64>;
  def LIBMVEC__ZGVnN2v_exp10 : LibmvecLibcall<EXP10_V2F64>;
  def LIBMVEC__ZGVnN2v_exp10f : LibmvecLibcall<EXP10_V2F32>;
  def LIBMVEC__ZGVnN2v_exp2 : LibmvecLibcall<EXP2_V2F64>;
  def LIBMVEC__ZGVnN2v_exp2f : LibmvecLibcall<EXP2_V2F32>;
  def LIBMVEC__ZGVnN2v_expf : LibmvecLibcall<EXP_V2F32>;
  def LIBMVEC__ZGVnN2v_expm1 : LibmvecLibcall<EXPM1_V2F64>;
````
- **L3713 EN**: Closes the current lexical scope or compound statement.
  **L3713 CN**: 结束当前词法作用域或复合语句块。
- **L3714 EN**: Blank line separating nearby declarations or logic blocks.
  **L3714 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3715 EN**: Continues the surrounding expression or declaration: `defset list<RuntimeLibcallImpl> LIBMVEC_AARCH64_VECFUNCS = {`.
  **L3715 CN**: 继续构造周围的表达式或声明：`defset list<RuntimeLibcallImpl> LIBMVEC_AARCH64_VECFUNCS = {`。
- **L3716 EN**: Declares TableGen def `LIBMVEC__ZGVnN2v_acos`.
  **L3716 CN**: 声明 TableGen def `LIBMVEC__ZGVnN2v_acos`。
- **L3717 EN**: Declares TableGen def `LIBMVEC__ZGVnN2v_acosf`.
  **L3717 CN**: 声明 TableGen def `LIBMVEC__ZGVnN2v_acosf`。
- **L3718 EN**: Declares TableGen def `LIBMVEC__ZGVnN2v_acosh`.
  **L3718 CN**: 声明 TableGen def `LIBMVEC__ZGVnN2v_acosh`。
- **L3719 EN**: Declares TableGen def `LIBMVEC__ZGVnN2v_acoshf`.
  **L3719 CN**: 声明 TableGen def `LIBMVEC__ZGVnN2v_acoshf`。
- **L3720 EN**: Declares TableGen def `LIBMVEC__ZGVnN2v_asin`.
  **L3720 CN**: 声明 TableGen def `LIBMVEC__ZGVnN2v_asin`。
- **L3721 EN**: Declares TableGen def `LIBMVEC__ZGVnN2v_asinf`.
  **L3721 CN**: 声明 TableGen def `LIBMVEC__ZGVnN2v_asinf`。
- **L3722 EN**: Declares TableGen def `LIBMVEC__ZGVnN2v_asinh`.
  **L3722 CN**: 声明 TableGen def `LIBMVEC__ZGVnN2v_asinh`。
- **L3723 EN**: Declares TableGen def `LIBMVEC__ZGVnN2v_asinhf`.
  **L3723 CN**: 声明 TableGen def `LIBMVEC__ZGVnN2v_asinhf`。
- **L3724 EN**: Declares TableGen def `LIBMVEC__ZGVnN2v_atan`.
  **L3724 CN**: 声明 TableGen def `LIBMVEC__ZGVnN2v_atan`。
- **L3725 EN**: Declares TableGen def `LIBMVEC__ZGVnN2v_atanf`.
  **L3725 CN**: 声明 TableGen def `LIBMVEC__ZGVnN2v_atanf`。
- **L3726 EN**: Declares TableGen def `LIBMVEC__ZGVnN2v_atanh`.
  **L3726 CN**: 声明 TableGen def `LIBMVEC__ZGVnN2v_atanh`。
- **L3727 EN**: Declares TableGen def `LIBMVEC__ZGVnN2v_atanhf`.
  **L3727 CN**: 声明 TableGen def `LIBMVEC__ZGVnN2v_atanhf`。
- **L3728 EN**: Declares TableGen def `LIBMVEC__ZGVnN2v_cbrt`.
  **L3728 CN**: 声明 TableGen def `LIBMVEC__ZGVnN2v_cbrt`。
- **L3729 EN**: Declares TableGen def `LIBMVEC__ZGVnN2v_cbrtf`.
  **L3729 CN**: 声明 TableGen def `LIBMVEC__ZGVnN2v_cbrtf`。
- **L3730 EN**: Declares TableGen def `LIBMVEC__ZGVnN2v_cos`.
  **L3730 CN**: 声明 TableGen def `LIBMVEC__ZGVnN2v_cos`。
- **L3731 EN**: Declares TableGen def `LIBMVEC__ZGVnN2v_cosf`.
  **L3731 CN**: 声明 TableGen def `LIBMVEC__ZGVnN2v_cosf`。
- **L3732 EN**: Declares TableGen def `LIBMVEC__ZGVnN2v_cosh`.
  **L3732 CN**: 声明 TableGen def `LIBMVEC__ZGVnN2v_cosh`。
- **L3733 EN**: Declares TableGen def `LIBMVEC__ZGVnN2v_coshf`.
  **L3733 CN**: 声明 TableGen def `LIBMVEC__ZGVnN2v_coshf`。
- **L3734 EN**: Declares TableGen def `LIBMVEC__ZGVnN2v_erf`.
  **L3734 CN**: 声明 TableGen def `LIBMVEC__ZGVnN2v_erf`。
- **L3735 EN**: Declares TableGen def `LIBMVEC__ZGVnN2v_erfc`.
  **L3735 CN**: 声明 TableGen def `LIBMVEC__ZGVnN2v_erfc`。
- **L3736 EN**: Declares TableGen def `LIBMVEC__ZGVnN2v_erfcf`.
  **L3736 CN**: 声明 TableGen def `LIBMVEC__ZGVnN2v_erfcf`。
- **L3737 EN**: Declares TableGen def `LIBMVEC__ZGVnN2v_erff`.
  **L3737 CN**: 声明 TableGen def `LIBMVEC__ZGVnN2v_erff`。
- **L3738 EN**: Declares TableGen def `LIBMVEC__ZGVnN2v_exp`.
  **L3738 CN**: 声明 TableGen def `LIBMVEC__ZGVnN2v_exp`。
- **L3739 EN**: Declares TableGen def `LIBMVEC__ZGVnN2v_exp10`.
  **L3739 CN**: 声明 TableGen def `LIBMVEC__ZGVnN2v_exp10`。
- **L3740 EN**: Declares TableGen def `LIBMVEC__ZGVnN2v_exp10f`.
  **L3740 CN**: 声明 TableGen def `LIBMVEC__ZGVnN2v_exp10f`。
- **L3741 EN**: Declares TableGen def `LIBMVEC__ZGVnN2v_exp2`.
  **L3741 CN**: 声明 TableGen def `LIBMVEC__ZGVnN2v_exp2`。
- **L3742 EN**: Declares TableGen def `LIBMVEC__ZGVnN2v_exp2f`.
  **L3742 CN**: 声明 TableGen def `LIBMVEC__ZGVnN2v_exp2f`。
- **L3743 EN**: Declares TableGen def `LIBMVEC__ZGVnN2v_expf`.
  **L3743 CN**: 声明 TableGen def `LIBMVEC__ZGVnN2v_expf`。
- **L3744 EN**: Declares TableGen def `LIBMVEC__ZGVnN2v_expm1`.
  **L3744 CN**: 声明 TableGen def `LIBMVEC__ZGVnN2v_expm1`。

### Lines 3745-3776

````tablegen
  def LIBMVEC__ZGVnN2v_expm1f : LibmvecLibcall<EXPM1_V2F32>;
  def LIBMVEC__ZGVnN2v_log : LibmvecLibcall<LOG_V2F64>;
  def LIBMVEC__ZGVnN2v_log10 : LibmvecLibcall<LOG10_V2F64>;
  def LIBMVEC__ZGVnN2v_log10f : LibmvecLibcall<LOG10_V2F32>;
  def LIBMVEC__ZGVnN2v_log1p : LibmvecLibcall<LOG1P_V2F64>;
  def LIBMVEC__ZGVnN2v_log1pf : LibmvecLibcall<LOG1P_V2F32>;
  def LIBMVEC__ZGVnN2v_log2 : LibmvecLibcall<LOG2_V2F64>;
  def LIBMVEC__ZGVnN2v_log2f : LibmvecLibcall<LOG2_V2F32>;
  def LIBMVEC__ZGVnN2v_logf : LibmvecLibcall<LOG_V2F32>;
  def LIBMVEC__ZGVnN2v_sin : LibmvecLibcall<SIN_V2F64>;
  def LIBMVEC__ZGVnN2v_sinf : LibmvecLibcall<SIN_V2F32>;
  def LIBMVEC__ZGVnN2v_sinh : LibmvecLibcall<SINH_V2F64>;
  def LIBMVEC__ZGVnN2v_sinhf : LibmvecLibcall<SINH_V2F32>;
  def LIBMVEC__ZGVnN2v_tan : LibmvecLibcall<TAN_V2F64>;
  def LIBMVEC__ZGVnN2v_tanf : LibmvecLibcall<TAN_V2F32>;
  def LIBMVEC__ZGVnN2v_tanh : LibmvecLibcall<TANH_V2F64>;
  def LIBMVEC__ZGVnN2v_tanhf : LibmvecLibcall<TANH_V2F32>;
  def LIBMVEC__ZGVnN2vv_atan2 : LibmvecLibcall<ATAN2_V2F64>;
  def LIBMVEC__ZGVnN2vv_atan2f : LibmvecLibcall<ATAN2_V2F32>;
  def LIBMVEC__ZGVnN2vv_hypot : LibmvecLibcall<HYPOT_V2F64>;
  def LIBMVEC__ZGVnN2vv_hypotf : LibmvecLibcall<HYPOT_V2F32>;
  def LIBMVEC__ZGVnN2vv_pow : LibmvecLibcall<POW_V2F64>;
  def LIBMVEC__ZGVnN2vv_powf : LibmvecLibcall<POW_V2F32>;
  def LIBMVEC__ZGVnN4v_acosf : LibmvecLibcall<ACOS_V4F32>;
  def LIBMVEC__ZGVnN4v_acoshf : LibmvecLibcall<ACOSH_V4F32>;
  def LIBMVEC__ZGVnN4v_asinf : LibmvecLibcall<ASIN_V4F32>;
  def LIBMVEC__ZGVnN4v_asinhf : LibmvecLibcall<ASINH_V4F32>;
  def LIBMVEC__ZGVnN4v_atanf : LibmvecLibcall<ATAN_V4F32>;
  def LIBMVEC__ZGVnN4v_atanhf : LibmvecLibcall<ATANH_V4F32>;
  def LIBMVEC__ZGVnN4v_cbrtf : LibmvecLibcall<CBRT_V4F32>;
  def LIBMVEC__ZGVnN4v_cosf : LibmvecLibcall<COS_V4F32>;
  def LIBMVEC__ZGVnN4v_coshf : LibmvecLibcall<COSH_V4F32>;
````
- **L3745 EN**: Declares TableGen def `LIBMVEC__ZGVnN2v_expm1f`.
  **L3745 CN**: 声明 TableGen def `LIBMVEC__ZGVnN2v_expm1f`。
- **L3746 EN**: Declares TableGen def `LIBMVEC__ZGVnN2v_log`.
  **L3746 CN**: 声明 TableGen def `LIBMVEC__ZGVnN2v_log`。
- **L3747 EN**: Declares TableGen def `LIBMVEC__ZGVnN2v_log10`.
  **L3747 CN**: 声明 TableGen def `LIBMVEC__ZGVnN2v_log10`。
- **L3748 EN**: Declares TableGen def `LIBMVEC__ZGVnN2v_log10f`.
  **L3748 CN**: 声明 TableGen def `LIBMVEC__ZGVnN2v_log10f`。
- **L3749 EN**: Declares TableGen def `LIBMVEC__ZGVnN2v_log1p`.
  **L3749 CN**: 声明 TableGen def `LIBMVEC__ZGVnN2v_log1p`。
- **L3750 EN**: Declares TableGen def `LIBMVEC__ZGVnN2v_log1pf`.
  **L3750 CN**: 声明 TableGen def `LIBMVEC__ZGVnN2v_log1pf`。
- **L3751 EN**: Declares TableGen def `LIBMVEC__ZGVnN2v_log2`.
  **L3751 CN**: 声明 TableGen def `LIBMVEC__ZGVnN2v_log2`。
- **L3752 EN**: Declares TableGen def `LIBMVEC__ZGVnN2v_log2f`.
  **L3752 CN**: 声明 TableGen def `LIBMVEC__ZGVnN2v_log2f`。
- **L3753 EN**: Declares TableGen def `LIBMVEC__ZGVnN2v_logf`.
  **L3753 CN**: 声明 TableGen def `LIBMVEC__ZGVnN2v_logf`。
- **L3754 EN**: Declares TableGen def `LIBMVEC__ZGVnN2v_sin`.
  **L3754 CN**: 声明 TableGen def `LIBMVEC__ZGVnN2v_sin`。
- **L3755 EN**: Declares TableGen def `LIBMVEC__ZGVnN2v_sinf`.
  **L3755 CN**: 声明 TableGen def `LIBMVEC__ZGVnN2v_sinf`。
- **L3756 EN**: Declares TableGen def `LIBMVEC__ZGVnN2v_sinh`.
  **L3756 CN**: 声明 TableGen def `LIBMVEC__ZGVnN2v_sinh`。
- **L3757 EN**: Declares TableGen def `LIBMVEC__ZGVnN2v_sinhf`.
  **L3757 CN**: 声明 TableGen def `LIBMVEC__ZGVnN2v_sinhf`。
- **L3758 EN**: Declares TableGen def `LIBMVEC__ZGVnN2v_tan`.
  **L3758 CN**: 声明 TableGen def `LIBMVEC__ZGVnN2v_tan`。
- **L3759 EN**: Declares TableGen def `LIBMVEC__ZGVnN2v_tanf`.
  **L3759 CN**: 声明 TableGen def `LIBMVEC__ZGVnN2v_tanf`。
- **L3760 EN**: Declares TableGen def `LIBMVEC__ZGVnN2v_tanh`.
  **L3760 CN**: 声明 TableGen def `LIBMVEC__ZGVnN2v_tanh`。
- **L3761 EN**: Declares TableGen def `LIBMVEC__ZGVnN2v_tanhf`.
  **L3761 CN**: 声明 TableGen def `LIBMVEC__ZGVnN2v_tanhf`。
- **L3762 EN**: Declares TableGen def `LIBMVEC__ZGVnN2vv_atan2`.
  **L3762 CN**: 声明 TableGen def `LIBMVEC__ZGVnN2vv_atan2`。
- **L3763 EN**: Declares TableGen def `LIBMVEC__ZGVnN2vv_atan2f`.
  **L3763 CN**: 声明 TableGen def `LIBMVEC__ZGVnN2vv_atan2f`。
- **L3764 EN**: Declares TableGen def `LIBMVEC__ZGVnN2vv_hypot`.
  **L3764 CN**: 声明 TableGen def `LIBMVEC__ZGVnN2vv_hypot`。
- **L3765 EN**: Declares TableGen def `LIBMVEC__ZGVnN2vv_hypotf`.
  **L3765 CN**: 声明 TableGen def `LIBMVEC__ZGVnN2vv_hypotf`。
- **L3766 EN**: Declares TableGen def `LIBMVEC__ZGVnN2vv_pow`.
  **L3766 CN**: 声明 TableGen def `LIBMVEC__ZGVnN2vv_pow`。
- **L3767 EN**: Declares TableGen def `LIBMVEC__ZGVnN2vv_powf`.
  **L3767 CN**: 声明 TableGen def `LIBMVEC__ZGVnN2vv_powf`。
- **L3768 EN**: Declares TableGen def `LIBMVEC__ZGVnN4v_acosf`.
  **L3768 CN**: 声明 TableGen def `LIBMVEC__ZGVnN4v_acosf`。
- **L3769 EN**: Declares TableGen def `LIBMVEC__ZGVnN4v_acoshf`.
  **L3769 CN**: 声明 TableGen def `LIBMVEC__ZGVnN4v_acoshf`。
- **L3770 EN**: Declares TableGen def `LIBMVEC__ZGVnN4v_asinf`.
  **L3770 CN**: 声明 TableGen def `LIBMVEC__ZGVnN4v_asinf`。
- **L3771 EN**: Declares TableGen def `LIBMVEC__ZGVnN4v_asinhf`.
  **L3771 CN**: 声明 TableGen def `LIBMVEC__ZGVnN4v_asinhf`。
- **L3772 EN**: Declares TableGen def `LIBMVEC__ZGVnN4v_atanf`.
  **L3772 CN**: 声明 TableGen def `LIBMVEC__ZGVnN4v_atanf`。
- **L3773 EN**: Declares TableGen def `LIBMVEC__ZGVnN4v_atanhf`.
  **L3773 CN**: 声明 TableGen def `LIBMVEC__ZGVnN4v_atanhf`。
- **L3774 EN**: Declares TableGen def `LIBMVEC__ZGVnN4v_cbrtf`.
  **L3774 CN**: 声明 TableGen def `LIBMVEC__ZGVnN4v_cbrtf`。
- **L3775 EN**: Declares TableGen def `LIBMVEC__ZGVnN4v_cosf`.
  **L3775 CN**: 声明 TableGen def `LIBMVEC__ZGVnN4v_cosf`。
- **L3776 EN**: Declares TableGen def `LIBMVEC__ZGVnN4v_coshf`.
  **L3776 CN**: 声明 TableGen def `LIBMVEC__ZGVnN4v_coshf`。

### Lines 3777-3808

````tablegen
  def LIBMVEC__ZGVnN4v_erfcf : LibmvecLibcall<ERFC_V4F32>;
  def LIBMVEC__ZGVnN4v_erff : LibmvecLibcall<ERF_V4F32>;
  def LIBMVEC__ZGVnN4v_exp10f : LibmvecLibcall<EXP10_V4F32>;
  def LIBMVEC__ZGVnN4v_exp2f : LibmvecLibcall<EXP2_V4F32>;
  def LIBMVEC__ZGVnN4v_expf : LibmvecLibcall<EXP_V4F32>;
  def LIBMVEC__ZGVnN4v_expm1f : LibmvecLibcall<EXPM1_V4F32>;
  def LIBMVEC__ZGVnN4v_log10f : LibmvecLibcall<LOG10_V4F32>;
  def LIBMVEC__ZGVnN4v_log1pf : LibmvecLibcall<LOG1P_V4F32>;
  def LIBMVEC__ZGVnN4v_log2f : LibmvecLibcall<LOG2_V4F32>;
  def LIBMVEC__ZGVnN4v_logf : LibmvecLibcall<LOG_V4F32>;
  def LIBMVEC__ZGVnN4v_sinf : LibmvecLibcall<SIN_V4F32>;
  def LIBMVEC__ZGVnN4v_sinhf : LibmvecLibcall<SINH_V4F32>;
  def LIBMVEC__ZGVnN4v_tanf : LibmvecLibcall<TAN_V4F32>;
  def LIBMVEC__ZGVnN4v_tanhf : LibmvecLibcall<TANH_V4F32>;
  def LIBMVEC__ZGVnN4vv_atan2f : LibmvecLibcall<ATAN2_V4F32>;
  def LIBMVEC__ZGVnN4vv_hypotf : LibmvecLibcall<HYPOT_V4F32>;
  def LIBMVEC__ZGVnN4vv_powf : LibmvecLibcall<POW_V4F32>;
  def LIBMVEC__ZGVsMxv_acos : LibmvecLibcall<ACOS_NXV2F64>;
  def LIBMVEC__ZGVsMxv_acosf : LibmvecLibcall<ACOS_NXV4F32>;
  def LIBMVEC__ZGVsMxv_acosh : LibmvecLibcall<ACOSH_NXV2F64>;
  def LIBMVEC__ZGVsMxv_acoshf : LibmvecLibcall<ACOSH_NXV4F32>;
  def LIBMVEC__ZGVsMxv_asin : LibmvecLibcall<ASIN_NXV2F64>;
  def LIBMVEC__ZGVsMxv_asinf : LibmvecLibcall<ASIN_NXV4F32>;
  def LIBMVEC__ZGVsMxv_asinh : LibmvecLibcall<ASINH_NXV2F64>;
  def LIBMVEC__ZGVsMxv_asinhf : LibmvecLibcall<ASINH_NXV4F32>;
  def LIBMVEC__ZGVsMxv_atan : LibmvecLibcall<ATAN_NXV2F64>;
  def LIBMVEC__ZGVsMxv_atanf : LibmvecLibcall<ATAN_NXV4F32>;
  def LIBMVEC__ZGVsMxv_atanh : LibmvecLibcall<ATANH_NXV2F64>;
  def LIBMVEC__ZGVsMxv_atanhf : LibmvecLibcall<ATANH_NXV4F32>;
  def LIBMVEC__ZGVsMxv_cbrt : LibmvecLibcall<CBRT_NXV2F64>;
  def LIBMVEC__ZGVsMxv_cbrtf : LibmvecLibcall<CBRT_NXV4F32>;
  def LIBMVEC__ZGVsMxv_cos : LibmvecLibcall<COS_NXV2F64>;
````
- **L3777 EN**: Declares TableGen def `LIBMVEC__ZGVnN4v_erfcf`.
  **L3777 CN**: 声明 TableGen def `LIBMVEC__ZGVnN4v_erfcf`。
- **L3778 EN**: Declares TableGen def `LIBMVEC__ZGVnN4v_erff`.
  **L3778 CN**: 声明 TableGen def `LIBMVEC__ZGVnN4v_erff`。
- **L3779 EN**: Declares TableGen def `LIBMVEC__ZGVnN4v_exp10f`.
  **L3779 CN**: 声明 TableGen def `LIBMVEC__ZGVnN4v_exp10f`。
- **L3780 EN**: Declares TableGen def `LIBMVEC__ZGVnN4v_exp2f`.
  **L3780 CN**: 声明 TableGen def `LIBMVEC__ZGVnN4v_exp2f`。
- **L3781 EN**: Declares TableGen def `LIBMVEC__ZGVnN4v_expf`.
  **L3781 CN**: 声明 TableGen def `LIBMVEC__ZGVnN4v_expf`。
- **L3782 EN**: Declares TableGen def `LIBMVEC__ZGVnN4v_expm1f`.
  **L3782 CN**: 声明 TableGen def `LIBMVEC__ZGVnN4v_expm1f`。
- **L3783 EN**: Declares TableGen def `LIBMVEC__ZGVnN4v_log10f`.
  **L3783 CN**: 声明 TableGen def `LIBMVEC__ZGVnN4v_log10f`。
- **L3784 EN**: Declares TableGen def `LIBMVEC__ZGVnN4v_log1pf`.
  **L3784 CN**: 声明 TableGen def `LIBMVEC__ZGVnN4v_log1pf`。
- **L3785 EN**: Declares TableGen def `LIBMVEC__ZGVnN4v_log2f`.
  **L3785 CN**: 声明 TableGen def `LIBMVEC__ZGVnN4v_log2f`。
- **L3786 EN**: Declares TableGen def `LIBMVEC__ZGVnN4v_logf`.
  **L3786 CN**: 声明 TableGen def `LIBMVEC__ZGVnN4v_logf`。
- **L3787 EN**: Declares TableGen def `LIBMVEC__ZGVnN4v_sinf`.
  **L3787 CN**: 声明 TableGen def `LIBMVEC__ZGVnN4v_sinf`。
- **L3788 EN**: Declares TableGen def `LIBMVEC__ZGVnN4v_sinhf`.
  **L3788 CN**: 声明 TableGen def `LIBMVEC__ZGVnN4v_sinhf`。
- **L3789 EN**: Declares TableGen def `LIBMVEC__ZGVnN4v_tanf`.
  **L3789 CN**: 声明 TableGen def `LIBMVEC__ZGVnN4v_tanf`。
- **L3790 EN**: Declares TableGen def `LIBMVEC__ZGVnN4v_tanhf`.
  **L3790 CN**: 声明 TableGen def `LIBMVEC__ZGVnN4v_tanhf`。
- **L3791 EN**: Declares TableGen def `LIBMVEC__ZGVnN4vv_atan2f`.
  **L3791 CN**: 声明 TableGen def `LIBMVEC__ZGVnN4vv_atan2f`。
- **L3792 EN**: Declares TableGen def `LIBMVEC__ZGVnN4vv_hypotf`.
  **L3792 CN**: 声明 TableGen def `LIBMVEC__ZGVnN4vv_hypotf`。
- **L3793 EN**: Declares TableGen def `LIBMVEC__ZGVnN4vv_powf`.
  **L3793 CN**: 声明 TableGen def `LIBMVEC__ZGVnN4vv_powf`。
- **L3794 EN**: Declares TableGen def `LIBMVEC__ZGVsMxv_acos`.
  **L3794 CN**: 声明 TableGen def `LIBMVEC__ZGVsMxv_acos`。
- **L3795 EN**: Declares TableGen def `LIBMVEC__ZGVsMxv_acosf`.
  **L3795 CN**: 声明 TableGen def `LIBMVEC__ZGVsMxv_acosf`。
- **L3796 EN**: Declares TableGen def `LIBMVEC__ZGVsMxv_acosh`.
  **L3796 CN**: 声明 TableGen def `LIBMVEC__ZGVsMxv_acosh`。
- **L3797 EN**: Declares TableGen def `LIBMVEC__ZGVsMxv_acoshf`.
  **L3797 CN**: 声明 TableGen def `LIBMVEC__ZGVsMxv_acoshf`。
- **L3798 EN**: Declares TableGen def `LIBMVEC__ZGVsMxv_asin`.
  **L3798 CN**: 声明 TableGen def `LIBMVEC__ZGVsMxv_asin`。
- **L3799 EN**: Declares TableGen def `LIBMVEC__ZGVsMxv_asinf`.
  **L3799 CN**: 声明 TableGen def `LIBMVEC__ZGVsMxv_asinf`。
- **L3800 EN**: Declares TableGen def `LIBMVEC__ZGVsMxv_asinh`.
  **L3800 CN**: 声明 TableGen def `LIBMVEC__ZGVsMxv_asinh`。
- **L3801 EN**: Declares TableGen def `LIBMVEC__ZGVsMxv_asinhf`.
  **L3801 CN**: 声明 TableGen def `LIBMVEC__ZGVsMxv_asinhf`。
- **L3802 EN**: Declares TableGen def `LIBMVEC__ZGVsMxv_atan`.
  **L3802 CN**: 声明 TableGen def `LIBMVEC__ZGVsMxv_atan`。
- **L3803 EN**: Declares TableGen def `LIBMVEC__ZGVsMxv_atanf`.
  **L3803 CN**: 声明 TableGen def `LIBMVEC__ZGVsMxv_atanf`。
- **L3804 EN**: Declares TableGen def `LIBMVEC__ZGVsMxv_atanh`.
  **L3804 CN**: 声明 TableGen def `LIBMVEC__ZGVsMxv_atanh`。
- **L3805 EN**: Declares TableGen def `LIBMVEC__ZGVsMxv_atanhf`.
  **L3805 CN**: 声明 TableGen def `LIBMVEC__ZGVsMxv_atanhf`。
- **L3806 EN**: Declares TableGen def `LIBMVEC__ZGVsMxv_cbrt`.
  **L3806 CN**: 声明 TableGen def `LIBMVEC__ZGVsMxv_cbrt`。
- **L3807 EN**: Declares TableGen def `LIBMVEC__ZGVsMxv_cbrtf`.
  **L3807 CN**: 声明 TableGen def `LIBMVEC__ZGVsMxv_cbrtf`。
- **L3808 EN**: Declares TableGen def `LIBMVEC__ZGVsMxv_cos`.
  **L3808 CN**: 声明 TableGen def `LIBMVEC__ZGVsMxv_cos`。

### Lines 3809-3840

````tablegen
  def LIBMVEC__ZGVsMxv_cosf : LibmvecLibcall<COS_NXV4F32>;
  def LIBMVEC__ZGVsMxv_cosh : LibmvecLibcall<COSH_NXV2F64>;
  def LIBMVEC__ZGVsMxv_coshf : LibmvecLibcall<COSH_NXV4F32>;
  def LIBMVEC__ZGVsMxv_erf : LibmvecLibcall<ERF_NXV2F64>;
  def LIBMVEC__ZGVsMxv_erfc : LibmvecLibcall<ERFC_NXV2F64>;
  def LIBMVEC__ZGVsMxv_erfcf : LibmvecLibcall<ERFC_NXV4F32>;
  def LIBMVEC__ZGVsMxv_erff : LibmvecLibcall<ERF_NXV4F32>;
  def LIBMVEC__ZGVsMxv_exp : LibmvecLibcall<EXP_NXV2F64>;
  def LIBMVEC__ZGVsMxv_exp10 : LibmvecLibcall<EXP10_NXV2F64>;
  def LIBMVEC__ZGVsMxv_exp10f : LibmvecLibcall<EXP10_NXV4F32>;
  def LIBMVEC__ZGVsMxv_exp2 : LibmvecLibcall<EXP2_NXV2F64>;
  def LIBMVEC__ZGVsMxv_exp2f : LibmvecLibcall<EXP2_NXV4F32>;
  def LIBMVEC__ZGVsMxv_expf : LibmvecLibcall<EXP_NXV4F32>;
  def LIBMVEC__ZGVsMxv_expm1 : LibmvecLibcall<EXPM1_NXV2F64>;
  def LIBMVEC__ZGVsMxv_expm1f : LibmvecLibcall<EXPM1_NXV4F32>;
  def LIBMVEC__ZGVsMxv_log : LibmvecLibcall<LOG_NXV2F64>;
  def LIBMVEC__ZGVsMxv_log10 : LibmvecLibcall<LOG10_NXV2F64>;
  def LIBMVEC__ZGVsMxv_log10f : LibmvecLibcall<LOG10_NXV4F32>;
  def LIBMVEC__ZGVsMxv_log1p : LibmvecLibcall<LOG1P_NXV2F64>;
  def LIBMVEC__ZGVsMxv_log1pf : LibmvecLibcall<LOG1P_NXV4F32>;
  def LIBMVEC__ZGVsMxv_log2 : LibmvecLibcall<LOG2_NXV2F64>;
  def LIBMVEC__ZGVsMxv_log2f : LibmvecLibcall<LOG2_NXV4F32>;
  def LIBMVEC__ZGVsMxv_logf : LibmvecLibcall<LOG_NXV4F32>;
  def LIBMVEC__ZGVsMxv_sin : LibmvecLibcall<SIN_NXV2F64>;
  def LIBMVEC__ZGVsMxv_sinf : LibmvecLibcall<SIN_NXV4F32>;
  def LIBMVEC__ZGVsMxv_sinh : LibmvecLibcall<SINH_NXV2F64>;
  def LIBMVEC__ZGVsMxv_sinhf : LibmvecLibcall<SINH_NXV4F32>;
  def LIBMVEC__ZGVsMxv_tan : LibmvecLibcall<TAN_NXV2F64>;
  def LIBMVEC__ZGVsMxv_tanf : LibmvecLibcall<TAN_NXV4F32>;
  def LIBMVEC__ZGVsMxv_tanh : LibmvecLibcall<TANH_NXV2F64>;
  def LIBMVEC__ZGVsMxv_tanhf : LibmvecLibcall<TANH_NXV4F32>;
  def LIBMVEC__ZGVsMxvv_atan2 : LibmvecLibcall<ATAN2_NXV2F64>;
````
- **L3809 EN**: Declares TableGen def `LIBMVEC__ZGVsMxv_cosf`.
  **L3809 CN**: 声明 TableGen def `LIBMVEC__ZGVsMxv_cosf`。
- **L3810 EN**: Declares TableGen def `LIBMVEC__ZGVsMxv_cosh`.
  **L3810 CN**: 声明 TableGen def `LIBMVEC__ZGVsMxv_cosh`。
- **L3811 EN**: Declares TableGen def `LIBMVEC__ZGVsMxv_coshf`.
  **L3811 CN**: 声明 TableGen def `LIBMVEC__ZGVsMxv_coshf`。
- **L3812 EN**: Declares TableGen def `LIBMVEC__ZGVsMxv_erf`.
  **L3812 CN**: 声明 TableGen def `LIBMVEC__ZGVsMxv_erf`。
- **L3813 EN**: Declares TableGen def `LIBMVEC__ZGVsMxv_erfc`.
  **L3813 CN**: 声明 TableGen def `LIBMVEC__ZGVsMxv_erfc`。
- **L3814 EN**: Declares TableGen def `LIBMVEC__ZGVsMxv_erfcf`.
  **L3814 CN**: 声明 TableGen def `LIBMVEC__ZGVsMxv_erfcf`。
- **L3815 EN**: Declares TableGen def `LIBMVEC__ZGVsMxv_erff`.
  **L3815 CN**: 声明 TableGen def `LIBMVEC__ZGVsMxv_erff`。
- **L3816 EN**: Declares TableGen def `LIBMVEC__ZGVsMxv_exp`.
  **L3816 CN**: 声明 TableGen def `LIBMVEC__ZGVsMxv_exp`。
- **L3817 EN**: Declares TableGen def `LIBMVEC__ZGVsMxv_exp10`.
  **L3817 CN**: 声明 TableGen def `LIBMVEC__ZGVsMxv_exp10`。
- **L3818 EN**: Declares TableGen def `LIBMVEC__ZGVsMxv_exp10f`.
  **L3818 CN**: 声明 TableGen def `LIBMVEC__ZGVsMxv_exp10f`。
- **L3819 EN**: Declares TableGen def `LIBMVEC__ZGVsMxv_exp2`.
  **L3819 CN**: 声明 TableGen def `LIBMVEC__ZGVsMxv_exp2`。
- **L3820 EN**: Declares TableGen def `LIBMVEC__ZGVsMxv_exp2f`.
  **L3820 CN**: 声明 TableGen def `LIBMVEC__ZGVsMxv_exp2f`。
- **L3821 EN**: Declares TableGen def `LIBMVEC__ZGVsMxv_expf`.
  **L3821 CN**: 声明 TableGen def `LIBMVEC__ZGVsMxv_expf`。
- **L3822 EN**: Declares TableGen def `LIBMVEC__ZGVsMxv_expm1`.
  **L3822 CN**: 声明 TableGen def `LIBMVEC__ZGVsMxv_expm1`。
- **L3823 EN**: Declares TableGen def `LIBMVEC__ZGVsMxv_expm1f`.
  **L3823 CN**: 声明 TableGen def `LIBMVEC__ZGVsMxv_expm1f`。
- **L3824 EN**: Declares TableGen def `LIBMVEC__ZGVsMxv_log`.
  **L3824 CN**: 声明 TableGen def `LIBMVEC__ZGVsMxv_log`。
- **L3825 EN**: Declares TableGen def `LIBMVEC__ZGVsMxv_log10`.
  **L3825 CN**: 声明 TableGen def `LIBMVEC__ZGVsMxv_log10`。
- **L3826 EN**: Declares TableGen def `LIBMVEC__ZGVsMxv_log10f`.
  **L3826 CN**: 声明 TableGen def `LIBMVEC__ZGVsMxv_log10f`。
- **L3827 EN**: Declares TableGen def `LIBMVEC__ZGVsMxv_log1p`.
  **L3827 CN**: 声明 TableGen def `LIBMVEC__ZGVsMxv_log1p`。
- **L3828 EN**: Declares TableGen def `LIBMVEC__ZGVsMxv_log1pf`.
  **L3828 CN**: 声明 TableGen def `LIBMVEC__ZGVsMxv_log1pf`。
- **L3829 EN**: Declares TableGen def `LIBMVEC__ZGVsMxv_log2`.
  **L3829 CN**: 声明 TableGen def `LIBMVEC__ZGVsMxv_log2`。
- **L3830 EN**: Declares TableGen def `LIBMVEC__ZGVsMxv_log2f`.
  **L3830 CN**: 声明 TableGen def `LIBMVEC__ZGVsMxv_log2f`。
- **L3831 EN**: Declares TableGen def `LIBMVEC__ZGVsMxv_logf`.
  **L3831 CN**: 声明 TableGen def `LIBMVEC__ZGVsMxv_logf`。
- **L3832 EN**: Declares TableGen def `LIBMVEC__ZGVsMxv_sin`.
  **L3832 CN**: 声明 TableGen def `LIBMVEC__ZGVsMxv_sin`。
- **L3833 EN**: Declares TableGen def `LIBMVEC__ZGVsMxv_sinf`.
  **L3833 CN**: 声明 TableGen def `LIBMVEC__ZGVsMxv_sinf`。
- **L3834 EN**: Declares TableGen def `LIBMVEC__ZGVsMxv_sinh`.
  **L3834 CN**: 声明 TableGen def `LIBMVEC__ZGVsMxv_sinh`。
- **L3835 EN**: Declares TableGen def `LIBMVEC__ZGVsMxv_sinhf`.
  **L3835 CN**: 声明 TableGen def `LIBMVEC__ZGVsMxv_sinhf`。
- **L3836 EN**: Declares TableGen def `LIBMVEC__ZGVsMxv_tan`.
  **L3836 CN**: 声明 TableGen def `LIBMVEC__ZGVsMxv_tan`。
- **L3837 EN**: Declares TableGen def `LIBMVEC__ZGVsMxv_tanf`.
  **L3837 CN**: 声明 TableGen def `LIBMVEC__ZGVsMxv_tanf`。
- **L3838 EN**: Declares TableGen def `LIBMVEC__ZGVsMxv_tanh`.
  **L3838 CN**: 声明 TableGen def `LIBMVEC__ZGVsMxv_tanh`。
- **L3839 EN**: Declares TableGen def `LIBMVEC__ZGVsMxv_tanhf`.
  **L3839 CN**: 声明 TableGen def `LIBMVEC__ZGVsMxv_tanhf`。
- **L3840 EN**: Declares TableGen def `LIBMVEC__ZGVsMxvv_atan2`.
  **L3840 CN**: 声明 TableGen def `LIBMVEC__ZGVsMxvv_atan2`。

### Lines 3841-3872

````tablegen
  def LIBMVEC__ZGVsMxvv_atan2f : LibmvecLibcall<ATAN2_NXV4F32>;
  def LIBMVEC__ZGVsMxvv_hypot : LibmvecLibcall<HYPOT_NXV2F64>;
  def LIBMVEC__ZGVsMxvv_hypotf : LibmvecLibcall<HYPOT_NXV4F32>;
  def LIBMVEC__ZGVsMxvv_pow : LibmvecLibcall<POW_NXV2F64>;
  def LIBMVEC__ZGVsMxvv_powf : LibmvecLibcall<POW_NXV4F32>;
}

//===----------------------------------------------------------------------===//
// IBM MASS vector library (MASSV) functions
//===----------------------------------------------------------------------===//

defset list<RuntimeLibcallImpl> MASSV_VECFUNCS = {
  def __acosd2 : RuntimeLibcallImpl<ACOS_V2F64>;
  def __acosf4 : RuntimeLibcallImpl<ACOS_V4F32>;
  def __acoshd2 : RuntimeLibcallImpl<ACOSH_V2F64>;
  def __acoshf4 : RuntimeLibcallImpl<ACOSH_V4F32>;
  def __asind2 : RuntimeLibcallImpl<ASIN_V2F64>;
  def __asinf4 : RuntimeLibcallImpl<ASIN_V4F32>;
  def __asinhd2 : RuntimeLibcallImpl<ASINH_V2F64>;
  def __asinhf4 : RuntimeLibcallImpl<ASINH_V4F32>;
  def __atan2d2 : RuntimeLibcallImpl<ATAN2_V2F64>;
  def __atan2f4 : RuntimeLibcallImpl<ATAN2_V4F32>;
  def __atand2 : RuntimeLibcallImpl<ATAN_V2F64>;
  def __atanf4 : RuntimeLibcallImpl<ATAN_V4F32>;
  def __atanhd2 : RuntimeLibcallImpl<ATANH_V2F64>;
  def __atanhf4 : RuntimeLibcallImpl<ATANH_V4F32>;
  def __cbrtd2 : RuntimeLibcallImpl<CBRT_V2F64>;
  def __cbrtf4 : RuntimeLibcallImpl<CBRT_V4F32>;
  def __cosd2 : RuntimeLibcallImpl<COS_V2F64>;
  def __cosf4 : RuntimeLibcallImpl<COS_V4F32>;
  def __coshd2 : RuntimeLibcallImpl<COSH_V2F64>;
  def __coshf4 : RuntimeLibcallImpl<COSH_V4F32>;
````
- **L3841 EN**: Declares TableGen def `LIBMVEC__ZGVsMxvv_atan2f`.
  **L3841 CN**: 声明 TableGen def `LIBMVEC__ZGVsMxvv_atan2f`。
- **L3842 EN**: Declares TableGen def `LIBMVEC__ZGVsMxvv_hypot`.
  **L3842 CN**: 声明 TableGen def `LIBMVEC__ZGVsMxvv_hypot`。
- **L3843 EN**: Declares TableGen def `LIBMVEC__ZGVsMxvv_hypotf`.
  **L3843 CN**: 声明 TableGen def `LIBMVEC__ZGVsMxvv_hypotf`。
- **L3844 EN**: Declares TableGen def `LIBMVEC__ZGVsMxvv_pow`.
  **L3844 CN**: 声明 TableGen def `LIBMVEC__ZGVsMxvv_pow`。
- **L3845 EN**: Declares TableGen def `LIBMVEC__ZGVsMxvv_powf`.
  **L3845 CN**: 声明 TableGen def `LIBMVEC__ZGVsMxvv_powf`。
- **L3846 EN**: Closes the current lexical scope or compound statement.
  **L3846 CN**: 结束当前词法作用域或复合语句块。
- **L3847 EN**: Blank line separating nearby declarations or logic blocks.
  **L3847 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3848 EN**: Banner comment marking a file or section boundary.
  **L3848 CN**: 横幅注释，用于标记文件或章节边界。
- **L3849 EN**: Comment explains nearby logic, invariants, or intent: `IBM MASS vector library (MASSV) functions`.
  **L3849 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`IBM MASS vector library (MASSV) functions`。
- **L3850 EN**: Banner comment marking a file or section boundary.
  **L3850 CN**: 横幅注释，用于标记文件或章节边界。
- **L3851 EN**: Blank line separating nearby declarations or logic blocks.
  **L3851 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3852 EN**: Continues the surrounding expression or declaration: `defset list<RuntimeLibcallImpl> MASSV_VECFUNCS = {`.
  **L3852 CN**: 继续构造周围的表达式或声明：`defset list<RuntimeLibcallImpl> MASSV_VECFUNCS = {`。
- **L3853 EN**: Declares TableGen def `__acosd2`.
  **L3853 CN**: 声明 TableGen def `__acosd2`。
- **L3854 EN**: Declares TableGen def `__acosf4`.
  **L3854 CN**: 声明 TableGen def `__acosf4`。
- **L3855 EN**: Declares TableGen def `__acoshd2`.
  **L3855 CN**: 声明 TableGen def `__acoshd2`。
- **L3856 EN**: Declares TableGen def `__acoshf4`.
  **L3856 CN**: 声明 TableGen def `__acoshf4`。
- **L3857 EN**: Declares TableGen def `__asind2`.
  **L3857 CN**: 声明 TableGen def `__asind2`。
- **L3858 EN**: Declares TableGen def `__asinf4`.
  **L3858 CN**: 声明 TableGen def `__asinf4`。
- **L3859 EN**: Declares TableGen def `__asinhd2`.
  **L3859 CN**: 声明 TableGen def `__asinhd2`。
- **L3860 EN**: Declares TableGen def `__asinhf4`.
  **L3860 CN**: 声明 TableGen def `__asinhf4`。
- **L3861 EN**: Declares TableGen def `__atan2d2`.
  **L3861 CN**: 声明 TableGen def `__atan2d2`。
- **L3862 EN**: Declares TableGen def `__atan2f4`.
  **L3862 CN**: 声明 TableGen def `__atan2f4`。
- **L3863 EN**: Declares TableGen def `__atand2`.
  **L3863 CN**: 声明 TableGen def `__atand2`。
- **L3864 EN**: Declares TableGen def `__atanf4`.
  **L3864 CN**: 声明 TableGen def `__atanf4`。
- **L3865 EN**: Declares TableGen def `__atanhd2`.
  **L3865 CN**: 声明 TableGen def `__atanhd2`。
- **L3866 EN**: Declares TableGen def `__atanhf4`.
  **L3866 CN**: 声明 TableGen def `__atanhf4`。
- **L3867 EN**: Declares TableGen def `__cbrtd2`.
  **L3867 CN**: 声明 TableGen def `__cbrtd2`。
- **L3868 EN**: Declares TableGen def `__cbrtf4`.
  **L3868 CN**: 声明 TableGen def `__cbrtf4`。
- **L3869 EN**: Declares TableGen def `__cosd2`.
  **L3869 CN**: 声明 TableGen def `__cosd2`。
- **L3870 EN**: Declares TableGen def `__cosf4`.
  **L3870 CN**: 声明 TableGen def `__cosf4`。
- **L3871 EN**: Declares TableGen def `__coshd2`.
  **L3871 CN**: 声明 TableGen def `__coshd2`。
- **L3872 EN**: Declares TableGen def `__coshf4`.
  **L3872 CN**: 声明 TableGen def `__coshf4`。

### Lines 3873-3904

````tablegen
  def __exp2d2 : RuntimeLibcallImpl<EXP2_V2F64>;
  def __exp2f4 : RuntimeLibcallImpl<EXP2_V4F32>;
  def __expd2 : RuntimeLibcallImpl<EXP_V2F64>;
  def __expf4 : RuntimeLibcallImpl<EXP_V4F32>;
  def __expm1d2 : RuntimeLibcallImpl<EXPM1_V2F64>;
  def __expm1f4 : RuntimeLibcallImpl<EXPM1_V4F32>;
  def __log10d2 : RuntimeLibcallImpl<LOG10_V2F64>;
  def __log10f4 : RuntimeLibcallImpl<LOG10_V4F32>;
  def __log1pd2 : RuntimeLibcallImpl<LOG1P_V2F64>;
  def __log1pf4 : RuntimeLibcallImpl<LOG1P_V4F32>;
  def __log2d2 : RuntimeLibcallImpl<LOG2_V2F64>;
  def __log2f4 : RuntimeLibcallImpl<LOG2_V4F32>;
  def __logd2 : RuntimeLibcallImpl<LOG_V2F64>;
  def __logf4 : RuntimeLibcallImpl<LOG_V4F32>;
  def __powd2 : RuntimeLibcallImpl<POW_V2F64>;
  def __powf4 : RuntimeLibcallImpl<POW_V4F32>;
  def __sind2 : RuntimeLibcallImpl<SIN_V2F64>;
  def __sinf4 : RuntimeLibcallImpl<SIN_V4F32>;
  def __sinhd2 : RuntimeLibcallImpl<SINH_V2F64>;
  def __sinhf4 : RuntimeLibcallImpl<SINH_V4F32>;
  def __tand2 : RuntimeLibcallImpl<TAN_V2F64>;
  def __tanf4 : RuntimeLibcallImpl<TAN_V4F32>;
  def __tanhd2 : RuntimeLibcallImpl<TANH_V2F64>;
  def __tanhf4 : RuntimeLibcallImpl<TANH_V4F32>;
}

//===----------------------------------------------------------------------===//
// Intel SVML library functions
//===----------------------------------------------------------------------===//

defset list<RuntimeLibcallImpl> SVML_VECFUNCS = {
  def __svml_cos2 : RuntimeLibcallImpl<COS_V2F64>;
````
- **L3873 EN**: Declares TableGen def `__exp2d2`.
  **L3873 CN**: 声明 TableGen def `__exp2d2`。
- **L3874 EN**: Declares TableGen def `__exp2f4`.
  **L3874 CN**: 声明 TableGen def `__exp2f4`。
- **L3875 EN**: Declares TableGen def `__expd2`.
  **L3875 CN**: 声明 TableGen def `__expd2`。
- **L3876 EN**: Declares TableGen def `__expf4`.
  **L3876 CN**: 声明 TableGen def `__expf4`。
- **L3877 EN**: Declares TableGen def `__expm1d2`.
  **L3877 CN**: 声明 TableGen def `__expm1d2`。
- **L3878 EN**: Declares TableGen def `__expm1f4`.
  **L3878 CN**: 声明 TableGen def `__expm1f4`。
- **L3879 EN**: Declares TableGen def `__log10d2`.
  **L3879 CN**: 声明 TableGen def `__log10d2`。
- **L3880 EN**: Declares TableGen def `__log10f4`.
  **L3880 CN**: 声明 TableGen def `__log10f4`。
- **L3881 EN**: Declares TableGen def `__log1pd2`.
  **L3881 CN**: 声明 TableGen def `__log1pd2`。
- **L3882 EN**: Declares TableGen def `__log1pf4`.
  **L3882 CN**: 声明 TableGen def `__log1pf4`。
- **L3883 EN**: Declares TableGen def `__log2d2`.
  **L3883 CN**: 声明 TableGen def `__log2d2`。
- **L3884 EN**: Declares TableGen def `__log2f4`.
  **L3884 CN**: 声明 TableGen def `__log2f4`。
- **L3885 EN**: Declares TableGen def `__logd2`.
  **L3885 CN**: 声明 TableGen def `__logd2`。
- **L3886 EN**: Declares TableGen def `__logf4`.
  **L3886 CN**: 声明 TableGen def `__logf4`。
- **L3887 EN**: Declares TableGen def `__powd2`.
  **L3887 CN**: 声明 TableGen def `__powd2`。
- **L3888 EN**: Declares TableGen def `__powf4`.
  **L3888 CN**: 声明 TableGen def `__powf4`。
- **L3889 EN**: Declares TableGen def `__sind2`.
  **L3889 CN**: 声明 TableGen def `__sind2`。
- **L3890 EN**: Declares TableGen def `__sinf4`.
  **L3890 CN**: 声明 TableGen def `__sinf4`。
- **L3891 EN**: Declares TableGen def `__sinhd2`.
  **L3891 CN**: 声明 TableGen def `__sinhd2`。
- **L3892 EN**: Declares TableGen def `__sinhf4`.
  **L3892 CN**: 声明 TableGen def `__sinhf4`。
- **L3893 EN**: Declares TableGen def `__tand2`.
  **L3893 CN**: 声明 TableGen def `__tand2`。
- **L3894 EN**: Declares TableGen def `__tanf4`.
  **L3894 CN**: 声明 TableGen def `__tanf4`。
- **L3895 EN**: Declares TableGen def `__tanhd2`.
  **L3895 CN**: 声明 TableGen def `__tanhd2`。
- **L3896 EN**: Declares TableGen def `__tanhf4`.
  **L3896 CN**: 声明 TableGen def `__tanhf4`。
- **L3897 EN**: Closes the current lexical scope or compound statement.
  **L3897 CN**: 结束当前词法作用域或复合语句块。
- **L3898 EN**: Blank line separating nearby declarations or logic blocks.
  **L3898 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3899 EN**: Banner comment marking a file or section boundary.
  **L3899 CN**: 横幅注释，用于标记文件或章节边界。
- **L3900 EN**: Comment explains nearby logic, invariants, or intent: `Intel SVML library functions`.
  **L3900 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Intel SVML library functions`。
- **L3901 EN**: Banner comment marking a file or section boundary.
  **L3901 CN**: 横幅注释，用于标记文件或章节边界。
- **L3902 EN**: Blank line separating nearby declarations or logic blocks.
  **L3902 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3903 EN**: Continues the surrounding expression or declaration: `defset list<RuntimeLibcallImpl> SVML_VECFUNCS = {`.
  **L3903 CN**: 继续构造周围的表达式或声明：`defset list<RuntimeLibcallImpl> SVML_VECFUNCS = {`。
- **L3904 EN**: Declares TableGen def `__svml_cos2`.
  **L3904 CN**: 声明 TableGen def `__svml_cos2`。

### Lines 3905-3936

````tablegen
  def __svml_cos4 : RuntimeLibcallImpl<COS_V4F64>;
  def __svml_cos8 : RuntimeLibcallImpl<COS_V8F64>;
  def __svml_cosf16 : RuntimeLibcallImpl<COS_V16F32>;
  def __svml_cosf4 : RuntimeLibcallImpl<COS_V4F32>;
  def __svml_cosf8 : RuntimeLibcallImpl<COS_V8F32>;
  def __svml_exp2 : RuntimeLibcallImpl<EXP_V2F64>;
  def __svml_exp22 : RuntimeLibcallImpl<EXP2_V2F64>;
  def __svml_exp24 : RuntimeLibcallImpl<EXP2_V4F64>;
  def __svml_exp28 : RuntimeLibcallImpl<EXP2_V8F64>;
  def __svml_exp2f16 : RuntimeLibcallImpl<EXP2_V16F32>;
  def __svml_exp2f4 : RuntimeLibcallImpl<EXP2_V4F32>;
  def __svml_exp2f8 : RuntimeLibcallImpl<EXP2_V8F32>;
  def __svml_exp4 : RuntimeLibcallImpl<EXP_V4F64>;
  def __svml_exp8 : RuntimeLibcallImpl<EXP_V8F64>;
  def __svml_expf16 : RuntimeLibcallImpl<EXP_V16F32>;
  def __svml_expf4 : RuntimeLibcallImpl<EXP_V4F32>;
  def __svml_expf8 : RuntimeLibcallImpl<EXP_V8F32>;
  def __svml_log102 : RuntimeLibcallImpl<LOG10_V2F64>;
  def __svml_log104 : RuntimeLibcallImpl<LOG10_V4F64>;
  def __svml_log108 : RuntimeLibcallImpl<LOG10_V8F64>;
  def __svml_log10f16 : RuntimeLibcallImpl<LOG10_V16F32>;
  def __svml_log10f4 : RuntimeLibcallImpl<LOG10_V4F32>;
  def __svml_log10f8 : RuntimeLibcallImpl<LOG10_V8F32>;
  def __svml_log2 : RuntimeLibcallImpl<LOG_V2F64>;
  def __svml_log22 : RuntimeLibcallImpl<LOG2_V2F64>;
  def __svml_log24 : RuntimeLibcallImpl<LOG2_V4F64>;
  def __svml_log28 : RuntimeLibcallImpl<LOG2_V8F64>;
  def __svml_log2f16 : RuntimeLibcallImpl<LOG2_V16F32>;
  def __svml_log2f4 : RuntimeLibcallImpl<LOG2_V4F32>;
  def __svml_log2f8 : RuntimeLibcallImpl<LOG2_V8F32>;
  def __svml_log4 : RuntimeLibcallImpl<LOG_V4F64>;
  def __svml_log8 : RuntimeLibcallImpl<LOG_V8F64>;
````
- **L3905 EN**: Declares TableGen def `__svml_cos4`.
  **L3905 CN**: 声明 TableGen def `__svml_cos4`。
- **L3906 EN**: Declares TableGen def `__svml_cos8`.
  **L3906 CN**: 声明 TableGen def `__svml_cos8`。
- **L3907 EN**: Declares TableGen def `__svml_cosf16`.
  **L3907 CN**: 声明 TableGen def `__svml_cosf16`。
- **L3908 EN**: Declares TableGen def `__svml_cosf4`.
  **L3908 CN**: 声明 TableGen def `__svml_cosf4`。
- **L3909 EN**: Declares TableGen def `__svml_cosf8`.
  **L3909 CN**: 声明 TableGen def `__svml_cosf8`。
- **L3910 EN**: Declares TableGen def `__svml_exp2`.
  **L3910 CN**: 声明 TableGen def `__svml_exp2`。
- **L3911 EN**: Declares TableGen def `__svml_exp22`.
  **L3911 CN**: 声明 TableGen def `__svml_exp22`。
- **L3912 EN**: Declares TableGen def `__svml_exp24`.
  **L3912 CN**: 声明 TableGen def `__svml_exp24`。
- **L3913 EN**: Declares TableGen def `__svml_exp28`.
  **L3913 CN**: 声明 TableGen def `__svml_exp28`。
- **L3914 EN**: Declares TableGen def `__svml_exp2f16`.
  **L3914 CN**: 声明 TableGen def `__svml_exp2f16`。
- **L3915 EN**: Declares TableGen def `__svml_exp2f4`.
  **L3915 CN**: 声明 TableGen def `__svml_exp2f4`。
- **L3916 EN**: Declares TableGen def `__svml_exp2f8`.
  **L3916 CN**: 声明 TableGen def `__svml_exp2f8`。
- **L3917 EN**: Declares TableGen def `__svml_exp4`.
  **L3917 CN**: 声明 TableGen def `__svml_exp4`。
- **L3918 EN**: Declares TableGen def `__svml_exp8`.
  **L3918 CN**: 声明 TableGen def `__svml_exp8`。
- **L3919 EN**: Declares TableGen def `__svml_expf16`.
  **L3919 CN**: 声明 TableGen def `__svml_expf16`。
- **L3920 EN**: Declares TableGen def `__svml_expf4`.
  **L3920 CN**: 声明 TableGen def `__svml_expf4`。
- **L3921 EN**: Declares TableGen def `__svml_expf8`.
  **L3921 CN**: 声明 TableGen def `__svml_expf8`。
- **L3922 EN**: Declares TableGen def `__svml_log102`.
  **L3922 CN**: 声明 TableGen def `__svml_log102`。
- **L3923 EN**: Declares TableGen def `__svml_log104`.
  **L3923 CN**: 声明 TableGen def `__svml_log104`。
- **L3924 EN**: Declares TableGen def `__svml_log108`.
  **L3924 CN**: 声明 TableGen def `__svml_log108`。
- **L3925 EN**: Declares TableGen def `__svml_log10f16`.
  **L3925 CN**: 声明 TableGen def `__svml_log10f16`。
- **L3926 EN**: Declares TableGen def `__svml_log10f4`.
  **L3926 CN**: 声明 TableGen def `__svml_log10f4`。
- **L3927 EN**: Declares TableGen def `__svml_log10f8`.
  **L3927 CN**: 声明 TableGen def `__svml_log10f8`。
- **L3928 EN**: Declares TableGen def `__svml_log2`.
  **L3928 CN**: 声明 TableGen def `__svml_log2`。
- **L3929 EN**: Declares TableGen def `__svml_log22`.
  **L3929 CN**: 声明 TableGen def `__svml_log22`。
- **L3930 EN**: Declares TableGen def `__svml_log24`.
  **L3930 CN**: 声明 TableGen def `__svml_log24`。
- **L3931 EN**: Declares TableGen def `__svml_log28`.
  **L3931 CN**: 声明 TableGen def `__svml_log28`。
- **L3932 EN**: Declares TableGen def `__svml_log2f16`.
  **L3932 CN**: 声明 TableGen def `__svml_log2f16`。
- **L3933 EN**: Declares TableGen def `__svml_log2f4`.
  **L3933 CN**: 声明 TableGen def `__svml_log2f4`。
- **L3934 EN**: Declares TableGen def `__svml_log2f8`.
  **L3934 CN**: 声明 TableGen def `__svml_log2f8`。
- **L3935 EN**: Declares TableGen def `__svml_log4`.
  **L3935 CN**: 声明 TableGen def `__svml_log4`。
- **L3936 EN**: Declares TableGen def `__svml_log8`.
  **L3936 CN**: 声明 TableGen def `__svml_log8`。

### Lines 3937-3968

````tablegen
  def __svml_logf16 : RuntimeLibcallImpl<LOG_V16F32>;
  def __svml_logf4 : RuntimeLibcallImpl<LOG_V4F32>;
  def __svml_logf8 : RuntimeLibcallImpl<LOG_V8F32>;
  def __svml_pow2 : RuntimeLibcallImpl<POW_V2F64>;
  def __svml_pow4 : RuntimeLibcallImpl<POW_V4F64>;
  def __svml_pow8 : RuntimeLibcallImpl<POW_V8F64>;
  def __svml_powf16 : RuntimeLibcallImpl<POW_V16F32>;
  def __svml_powf4 : RuntimeLibcallImpl<POW_V4F32>;
  def __svml_powf8 : RuntimeLibcallImpl<POW_V8F32>;
  def __svml_sin2 : RuntimeLibcallImpl<SIN_V2F64>;
  def __svml_sin4 : RuntimeLibcallImpl<SIN_V4F64>;
  def __svml_sin8 : RuntimeLibcallImpl<SIN_V8F64>;
  def __svml_sinf16 : RuntimeLibcallImpl<SIN_V16F32>;
  def __svml_sinf4 : RuntimeLibcallImpl<SIN_V4F32>;
  def __svml_sinf8 : RuntimeLibcallImpl<SIN_V8F32>;
  def __svml_sqrt2 : RuntimeLibcallImpl<SQRT_V2F64>;
  def __svml_sqrt4 : RuntimeLibcallImpl<SQRT_V4F64>;
  def __svml_sqrt8 : RuntimeLibcallImpl<SQRT_V8F64>;
  def __svml_sqrtf16 : RuntimeLibcallImpl<SQRT_V16F32>;
  def __svml_sqrtf4 : RuntimeLibcallImpl<SQRT_V4F32>;
  def __svml_sqrtf8 : RuntimeLibcallImpl<SQRT_V8F32>;
  def __svml_tan2 : RuntimeLibcallImpl<TAN_V2F64>;
  def __svml_tan4 : RuntimeLibcallImpl<TAN_V4F64>;
  def __svml_tan8 : RuntimeLibcallImpl<TAN_V8F64>;
  def __svml_tanf16 : RuntimeLibcallImpl<TAN_V16F32>;
  def __svml_tanf4 : RuntimeLibcallImpl<TAN_V4F32>;
  def __svml_tanf8 : RuntimeLibcallImpl<TAN_V8F32>;
}

//===----------------------------------------------------------------------===//
// SIMD Library for Evaluating Elementary Functions
//===----------------------------------------------------------------------===//
````
- **L3937 EN**: Declares TableGen def `__svml_logf16`.
  **L3937 CN**: 声明 TableGen def `__svml_logf16`。
- **L3938 EN**: Declares TableGen def `__svml_logf4`.
  **L3938 CN**: 声明 TableGen def `__svml_logf4`。
- **L3939 EN**: Declares TableGen def `__svml_logf8`.
  **L3939 CN**: 声明 TableGen def `__svml_logf8`。
- **L3940 EN**: Declares TableGen def `__svml_pow2`.
  **L3940 CN**: 声明 TableGen def `__svml_pow2`。
- **L3941 EN**: Declares TableGen def `__svml_pow4`.
  **L3941 CN**: 声明 TableGen def `__svml_pow4`。
- **L3942 EN**: Declares TableGen def `__svml_pow8`.
  **L3942 CN**: 声明 TableGen def `__svml_pow8`。
- **L3943 EN**: Declares TableGen def `__svml_powf16`.
  **L3943 CN**: 声明 TableGen def `__svml_powf16`。
- **L3944 EN**: Declares TableGen def `__svml_powf4`.
  **L3944 CN**: 声明 TableGen def `__svml_powf4`。
- **L3945 EN**: Declares TableGen def `__svml_powf8`.
  **L3945 CN**: 声明 TableGen def `__svml_powf8`。
- **L3946 EN**: Declares TableGen def `__svml_sin2`.
  **L3946 CN**: 声明 TableGen def `__svml_sin2`。
- **L3947 EN**: Declares TableGen def `__svml_sin4`.
  **L3947 CN**: 声明 TableGen def `__svml_sin4`。
- **L3948 EN**: Declares TableGen def `__svml_sin8`.
  **L3948 CN**: 声明 TableGen def `__svml_sin8`。
- **L3949 EN**: Declares TableGen def `__svml_sinf16`.
  **L3949 CN**: 声明 TableGen def `__svml_sinf16`。
- **L3950 EN**: Declares TableGen def `__svml_sinf4`.
  **L3950 CN**: 声明 TableGen def `__svml_sinf4`。
- **L3951 EN**: Declares TableGen def `__svml_sinf8`.
  **L3951 CN**: 声明 TableGen def `__svml_sinf8`。
- **L3952 EN**: Declares TableGen def `__svml_sqrt2`.
  **L3952 CN**: 声明 TableGen def `__svml_sqrt2`。
- **L3953 EN**: Declares TableGen def `__svml_sqrt4`.
  **L3953 CN**: 声明 TableGen def `__svml_sqrt4`。
- **L3954 EN**: Declares TableGen def `__svml_sqrt8`.
  **L3954 CN**: 声明 TableGen def `__svml_sqrt8`。
- **L3955 EN**: Declares TableGen def `__svml_sqrtf16`.
  **L3955 CN**: 声明 TableGen def `__svml_sqrtf16`。
- **L3956 EN**: Declares TableGen def `__svml_sqrtf4`.
  **L3956 CN**: 声明 TableGen def `__svml_sqrtf4`。
- **L3957 EN**: Declares TableGen def `__svml_sqrtf8`.
  **L3957 CN**: 声明 TableGen def `__svml_sqrtf8`。
- **L3958 EN**: Declares TableGen def `__svml_tan2`.
  **L3958 CN**: 声明 TableGen def `__svml_tan2`。
- **L3959 EN**: Declares TableGen def `__svml_tan4`.
  **L3959 CN**: 声明 TableGen def `__svml_tan4`。
- **L3960 EN**: Declares TableGen def `__svml_tan8`.
  **L3960 CN**: 声明 TableGen def `__svml_tan8`。
- **L3961 EN**: Declares TableGen def `__svml_tanf16`.
  **L3961 CN**: 声明 TableGen def `__svml_tanf16`。
- **L3962 EN**: Declares TableGen def `__svml_tanf4`.
  **L3962 CN**: 声明 TableGen def `__svml_tanf4`。
- **L3963 EN**: Declares TableGen def `__svml_tanf8`.
  **L3963 CN**: 声明 TableGen def `__svml_tanf8`。
- **L3964 EN**: Closes the current lexical scope or compound statement.
  **L3964 CN**: 结束当前词法作用域或复合语句块。
- **L3965 EN**: Blank line separating nearby declarations or logic blocks.
  **L3965 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3966 EN**: Banner comment marking a file or section boundary.
  **L3966 CN**: 横幅注释，用于标记文件或章节边界。
- **L3967 EN**: Comment explains nearby logic, invariants, or intent: `SIMD Library for Evaluating Elementary Functions`.
  **L3967 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`SIMD Library for Evaluating Elementary Functions`。
- **L3968 EN**: Banner comment marking a file or section boundary.
  **L3968 CN**: 横幅注释，用于标记文件或章节边界。

### Lines 3969-4000

````tablegen

defset list<RuntimeLibcallImpl> SLEEFGNUABI_VF2_VECFUNCS = {
  def _ZGVnN2v_acos : RuntimeLibcallImpl<ACOS_V2F64>;
  def _ZGVnN2v_acosh : RuntimeLibcallImpl<ACOSH_V2F64>;
  def _ZGVnN2v_asin : RuntimeLibcallImpl<ASIN_V2F64>;
  def _ZGVnN2v_asinh : RuntimeLibcallImpl<ASINH_V2F64>;
  def _ZGVnN2v_atan : RuntimeLibcallImpl<ATAN_V2F64>;
  def _ZGVnN2v_atanh : RuntimeLibcallImpl<ATANH_V2F64>;
  def _ZGVnN2v_cbrt : RuntimeLibcallImpl<CBRT_V2F64>;
  def _ZGVnN2v_cos : RuntimeLibcallImpl<COS_V2F64>;
  def _ZGVnN2v_cosh : RuntimeLibcallImpl<COSH_V2F64>;
  def _ZGVnN2v_cospi : RuntimeLibcallImpl<COSPI_V2F64>;
  def _ZGVnN2v_erf : RuntimeLibcallImpl<ERF_V2F64>;
  def _ZGVnN2v_erfc : RuntimeLibcallImpl<ERFC_V2F64>;
  def _ZGVnN2v_exp : RuntimeLibcallImpl<EXP_V2F64>;
  def _ZGVnN2v_exp10 : RuntimeLibcallImpl<EXP10_V2F64>;
  def _ZGVnN2v_exp2 : RuntimeLibcallImpl<EXP2_V2F64>;
  def _ZGVnN2v_expm1 : RuntimeLibcallImpl<EXPM1_V2F64>;
  def _ZGVnN2v_ilogb : RuntimeLibcallImpl<ILOGB_V2F64>;
  def _ZGVnN2v_lgamma : RuntimeLibcallImpl<LGAMMA_V2F64>;
  def _ZGVnN2v_log : RuntimeLibcallImpl<LOG_V2F64>;
  def _ZGVnN2v_log10 : RuntimeLibcallImpl<LOG10_V2F64>;
  def _ZGVnN2v_log1p : RuntimeLibcallImpl<LOG1P_V2F64>;
  def _ZGVnN2v_log2 : RuntimeLibcallImpl<LOG2_V2F64>;
  def _ZGVnN2v_sin : RuntimeLibcallImpl<SIN_V2F64>;
  def _ZGVnN2v_sinh : RuntimeLibcallImpl<SINH_V2F64>;
  def _ZGVnN2v_sinpi : RuntimeLibcallImpl<SINPI_V2F64>;
  def _ZGVnN2v_sqrt : RuntimeLibcallImpl<SQRT_V2F64>;
  def _ZGVnN2v_tan : RuntimeLibcallImpl<TAN_V2F64>;
  def _ZGVnN2v_tanh : RuntimeLibcallImpl<TANH_V2F64>;
  def _ZGVnN2v_tgamma : RuntimeLibcallImpl<TGAMMA_V2F64>;
  def _ZGVnN2vl8_modf : RuntimeLibcallImpl<MODF_V2F64>;
````
- **L3969 EN**: Blank line separating nearby declarations or logic blocks.
  **L3969 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3970 EN**: Continues the surrounding expression or declaration: `defset list<RuntimeLibcallImpl> SLEEFGNUABI_VF2_VECFUNCS = {`.
  **L3970 CN**: 继续构造周围的表达式或声明：`defset list<RuntimeLibcallImpl> SLEEFGNUABI_VF2_VECFUNCS = {`。
- **L3971 EN**: Declares TableGen def `_ZGVnN2v_acos`.
  **L3971 CN**: 声明 TableGen def `_ZGVnN2v_acos`。
- **L3972 EN**: Declares TableGen def `_ZGVnN2v_acosh`.
  **L3972 CN**: 声明 TableGen def `_ZGVnN2v_acosh`。
- **L3973 EN**: Declares TableGen def `_ZGVnN2v_asin`.
  **L3973 CN**: 声明 TableGen def `_ZGVnN2v_asin`。
- **L3974 EN**: Declares TableGen def `_ZGVnN2v_asinh`.
  **L3974 CN**: 声明 TableGen def `_ZGVnN2v_asinh`。
- **L3975 EN**: Declares TableGen def `_ZGVnN2v_atan`.
  **L3975 CN**: 声明 TableGen def `_ZGVnN2v_atan`。
- **L3976 EN**: Declares TableGen def `_ZGVnN2v_atanh`.
  **L3976 CN**: 声明 TableGen def `_ZGVnN2v_atanh`。
- **L3977 EN**: Declares TableGen def `_ZGVnN2v_cbrt`.
  **L3977 CN**: 声明 TableGen def `_ZGVnN2v_cbrt`。
- **L3978 EN**: Declares TableGen def `_ZGVnN2v_cos`.
  **L3978 CN**: 声明 TableGen def `_ZGVnN2v_cos`。
- **L3979 EN**: Declares TableGen def `_ZGVnN2v_cosh`.
  **L3979 CN**: 声明 TableGen def `_ZGVnN2v_cosh`。
- **L3980 EN**: Declares TableGen def `_ZGVnN2v_cospi`.
  **L3980 CN**: 声明 TableGen def `_ZGVnN2v_cospi`。
- **L3981 EN**: Declares TableGen def `_ZGVnN2v_erf`.
  **L3981 CN**: 声明 TableGen def `_ZGVnN2v_erf`。
- **L3982 EN**: Declares TableGen def `_ZGVnN2v_erfc`.
  **L3982 CN**: 声明 TableGen def `_ZGVnN2v_erfc`。
- **L3983 EN**: Declares TableGen def `_ZGVnN2v_exp`.
  **L3983 CN**: 声明 TableGen def `_ZGVnN2v_exp`。
- **L3984 EN**: Declares TableGen def `_ZGVnN2v_exp10`.
  **L3984 CN**: 声明 TableGen def `_ZGVnN2v_exp10`。
- **L3985 EN**: Declares TableGen def `_ZGVnN2v_exp2`.
  **L3985 CN**: 声明 TableGen def `_ZGVnN2v_exp2`。
- **L3986 EN**: Declares TableGen def `_ZGVnN2v_expm1`.
  **L3986 CN**: 声明 TableGen def `_ZGVnN2v_expm1`。
- **L3987 EN**: Declares TableGen def `_ZGVnN2v_ilogb`.
  **L3987 CN**: 声明 TableGen def `_ZGVnN2v_ilogb`。
- **L3988 EN**: Declares TableGen def `_ZGVnN2v_lgamma`.
  **L3988 CN**: 声明 TableGen def `_ZGVnN2v_lgamma`。
- **L3989 EN**: Declares TableGen def `_ZGVnN2v_log`.
  **L3989 CN**: 声明 TableGen def `_ZGVnN2v_log`。
- **L3990 EN**: Declares TableGen def `_ZGVnN2v_log10`.
  **L3990 CN**: 声明 TableGen def `_ZGVnN2v_log10`。
- **L3991 EN**: Declares TableGen def `_ZGVnN2v_log1p`.
  **L3991 CN**: 声明 TableGen def `_ZGVnN2v_log1p`。
- **L3992 EN**: Declares TableGen def `_ZGVnN2v_log2`.
  **L3992 CN**: 声明 TableGen def `_ZGVnN2v_log2`。
- **L3993 EN**: Declares TableGen def `_ZGVnN2v_sin`.
  **L3993 CN**: 声明 TableGen def `_ZGVnN2v_sin`。
- **L3994 EN**: Declares TableGen def `_ZGVnN2v_sinh`.
  **L3994 CN**: 声明 TableGen def `_ZGVnN2v_sinh`。
- **L3995 EN**: Declares TableGen def `_ZGVnN2v_sinpi`.
  **L3995 CN**: 声明 TableGen def `_ZGVnN2v_sinpi`。
- **L3996 EN**: Declares TableGen def `_ZGVnN2v_sqrt`.
  **L3996 CN**: 声明 TableGen def `_ZGVnN2v_sqrt`。
- **L3997 EN**: Declares TableGen def `_ZGVnN2v_tan`.
  **L3997 CN**: 声明 TableGen def `_ZGVnN2v_tan`。
- **L3998 EN**: Declares TableGen def `_ZGVnN2v_tanh`.
  **L3998 CN**: 声明 TableGen def `_ZGVnN2v_tanh`。
- **L3999 EN**: Declares TableGen def `_ZGVnN2v_tgamma`.
  **L3999 CN**: 声明 TableGen def `_ZGVnN2v_tgamma`。
- **L4000 EN**: Declares TableGen def `_ZGVnN2vl8_modf`.
  **L4000 CN**: 声明 TableGen def `_ZGVnN2vl8_modf`。

### Lines 4001-4032

````tablegen
  def _ZGVnN2vl8l8_sincos : RuntimeLibcallImpl<SINCOS_V2F64>;
  def _ZGVnN2vl8l8_sincospi : RuntimeLibcallImpl<SINCOSPI_V2F64>;
  def _ZGVnN2vv_atan2 : RuntimeLibcallImpl<ATAN2_V2F64>;
  def _ZGVnN2vv_copysign : RuntimeLibcallImpl<COPYSIGN_V2F64>;
  def _ZGVnN2vv_fdim : RuntimeLibcallImpl<FDIM_V2F64>;
  def _ZGVnN2vv_fmax : RuntimeLibcallImpl<FMAX_V2F64>;
  def _ZGVnN2vv_fmin : RuntimeLibcallImpl<FMIN_V2F64>;
  def _ZGVnN2vv_fmod : RuntimeLibcallImpl<REM_V2F64>;
  def _ZGVnN2vv_hypot : RuntimeLibcallImpl<HYPOT_V2F64>;
  def _ZGVnN2vv_ldexp : RuntimeLibcallImpl<LDEXP_V2F64>;
  def _ZGVnN2vv_nextafter : RuntimeLibcallImpl<NEXTAFTER_V2F64>;
  def _ZGVnN2vv_pow : RuntimeLibcallImpl<POW_V2F64>;
  def _ZGVnN2vvv_fma : RuntimeLibcallImpl<FMA_V2F64>;
}

defset list<RuntimeLibcallImpl> SLEEFGNUABI_VF4_VECFUNCS = {
  def _ZGVnN4v_acosf : RuntimeLibcallImpl<ACOS_V4F32>;
  def _ZGVnN4v_acoshf : RuntimeLibcallImpl<ACOSH_V4F32>;
  def _ZGVnN4v_asinf : RuntimeLibcallImpl<ASIN_V4F32>;
  def _ZGVnN4v_asinhf : RuntimeLibcallImpl<ASINH_V4F32>;
  def _ZGVnN4v_atanf : RuntimeLibcallImpl<ATAN_V4F32>;
  def _ZGVnN4v_atanhf : RuntimeLibcallImpl<ATANH_V4F32>;
  def _ZGVnN4v_cbrtf : RuntimeLibcallImpl<CBRT_V4F32>;
  def _ZGVnN4v_cosf : RuntimeLibcallImpl<COS_V4F32>;
  def _ZGVnN4v_coshf : RuntimeLibcallImpl<COSH_V4F32>;
  def _ZGVnN4v_cospif : RuntimeLibcallImpl<COSPI_V4F32>;
  def _ZGVnN4v_erfcf : RuntimeLibcallImpl<ERFC_V4F32>;
  def _ZGVnN4v_erff : RuntimeLibcallImpl<ERF_V4F32>;
  def _ZGVnN4v_exp10f : RuntimeLibcallImpl<EXP10_V4F32>;
  def _ZGVnN4v_exp2f : RuntimeLibcallImpl<EXP2_V4F32>;
  def _ZGVnN4v_expf : RuntimeLibcallImpl<EXP_V4F32>;
  def _ZGVnN4v_expm1f : RuntimeLibcallImpl<EXPM1_V4F32>;
````
- **L4001 EN**: Declares TableGen def `_ZGVnN2vl8l8_sincos`.
  **L4001 CN**: 声明 TableGen def `_ZGVnN2vl8l8_sincos`。
- **L4002 EN**: Declares TableGen def `_ZGVnN2vl8l8_sincospi`.
  **L4002 CN**: 声明 TableGen def `_ZGVnN2vl8l8_sincospi`。
- **L4003 EN**: Declares TableGen def `_ZGVnN2vv_atan2`.
  **L4003 CN**: 声明 TableGen def `_ZGVnN2vv_atan2`。
- **L4004 EN**: Declares TableGen def `_ZGVnN2vv_copysign`.
  **L4004 CN**: 声明 TableGen def `_ZGVnN2vv_copysign`。
- **L4005 EN**: Declares TableGen def `_ZGVnN2vv_fdim`.
  **L4005 CN**: 声明 TableGen def `_ZGVnN2vv_fdim`。
- **L4006 EN**: Declares TableGen def `_ZGVnN2vv_fmax`.
  **L4006 CN**: 声明 TableGen def `_ZGVnN2vv_fmax`。
- **L4007 EN**: Declares TableGen def `_ZGVnN2vv_fmin`.
  **L4007 CN**: 声明 TableGen def `_ZGVnN2vv_fmin`。
- **L4008 EN**: Declares TableGen def `_ZGVnN2vv_fmod`.
  **L4008 CN**: 声明 TableGen def `_ZGVnN2vv_fmod`。
- **L4009 EN**: Declares TableGen def `_ZGVnN2vv_hypot`.
  **L4009 CN**: 声明 TableGen def `_ZGVnN2vv_hypot`。
- **L4010 EN**: Declares TableGen def `_ZGVnN2vv_ldexp`.
  **L4010 CN**: 声明 TableGen def `_ZGVnN2vv_ldexp`。
- **L4011 EN**: Declares TableGen def `_ZGVnN2vv_nextafter`.
  **L4011 CN**: 声明 TableGen def `_ZGVnN2vv_nextafter`。
- **L4012 EN**: Declares TableGen def `_ZGVnN2vv_pow`.
  **L4012 CN**: 声明 TableGen def `_ZGVnN2vv_pow`。
- **L4013 EN**: Declares TableGen def `_ZGVnN2vvv_fma`.
  **L4013 CN**: 声明 TableGen def `_ZGVnN2vvv_fma`。
- **L4014 EN**: Closes the current lexical scope or compound statement.
  **L4014 CN**: 结束当前词法作用域或复合语句块。
- **L4015 EN**: Blank line separating nearby declarations or logic blocks.
  **L4015 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L4016 EN**: Continues the surrounding expression or declaration: `defset list<RuntimeLibcallImpl> SLEEFGNUABI_VF4_VECFUNCS = {`.
  **L4016 CN**: 继续构造周围的表达式或声明：`defset list<RuntimeLibcallImpl> SLEEFGNUABI_VF4_VECFUNCS = {`。
- **L4017 EN**: Declares TableGen def `_ZGVnN4v_acosf`.
  **L4017 CN**: 声明 TableGen def `_ZGVnN4v_acosf`。
- **L4018 EN**: Declares TableGen def `_ZGVnN4v_acoshf`.
  **L4018 CN**: 声明 TableGen def `_ZGVnN4v_acoshf`。
- **L4019 EN**: Declares TableGen def `_ZGVnN4v_asinf`.
  **L4019 CN**: 声明 TableGen def `_ZGVnN4v_asinf`。
- **L4020 EN**: Declares TableGen def `_ZGVnN4v_asinhf`.
  **L4020 CN**: 声明 TableGen def `_ZGVnN4v_asinhf`。
- **L4021 EN**: Declares TableGen def `_ZGVnN4v_atanf`.
  **L4021 CN**: 声明 TableGen def `_ZGVnN4v_atanf`。
- **L4022 EN**: Declares TableGen def `_ZGVnN4v_atanhf`.
  **L4022 CN**: 声明 TableGen def `_ZGVnN4v_atanhf`。
- **L4023 EN**: Declares TableGen def `_ZGVnN4v_cbrtf`.
  **L4023 CN**: 声明 TableGen def `_ZGVnN4v_cbrtf`。
- **L4024 EN**: Declares TableGen def `_ZGVnN4v_cosf`.
  **L4024 CN**: 声明 TableGen def `_ZGVnN4v_cosf`。
- **L4025 EN**: Declares TableGen def `_ZGVnN4v_coshf`.
  **L4025 CN**: 声明 TableGen def `_ZGVnN4v_coshf`。
- **L4026 EN**: Declares TableGen def `_ZGVnN4v_cospif`.
  **L4026 CN**: 声明 TableGen def `_ZGVnN4v_cospif`。
- **L4027 EN**: Declares TableGen def `_ZGVnN4v_erfcf`.
  **L4027 CN**: 声明 TableGen def `_ZGVnN4v_erfcf`。
- **L4028 EN**: Declares TableGen def `_ZGVnN4v_erff`.
  **L4028 CN**: 声明 TableGen def `_ZGVnN4v_erff`。
- **L4029 EN**: Declares TableGen def `_ZGVnN4v_exp10f`.
  **L4029 CN**: 声明 TableGen def `_ZGVnN4v_exp10f`。
- **L4030 EN**: Declares TableGen def `_ZGVnN4v_exp2f`.
  **L4030 CN**: 声明 TableGen def `_ZGVnN4v_exp2f`。
- **L4031 EN**: Declares TableGen def `_ZGVnN4v_expf`.
  **L4031 CN**: 声明 TableGen def `_ZGVnN4v_expf`。
- **L4032 EN**: Declares TableGen def `_ZGVnN4v_expm1f`.
  **L4032 CN**: 声明 TableGen def `_ZGVnN4v_expm1f`。

### Lines 4033-4064

````tablegen
  def _ZGVnN4v_ilogbf : RuntimeLibcallImpl<ILOGB_V4F32>;
  def _ZGVnN4v_lgammaf : RuntimeLibcallImpl<LGAMMA_V4F32>;
  def _ZGVnN4v_log10f : RuntimeLibcallImpl<LOG10_V4F32>;
  def _ZGVnN4v_log1pf : RuntimeLibcallImpl<LOG1P_V4F32>;
  def _ZGVnN4v_log2f : RuntimeLibcallImpl<LOG2_V4F32>;
  def _ZGVnN4v_logf : RuntimeLibcallImpl<LOG_V4F32>;
  def _ZGVnN4v_sinf : RuntimeLibcallImpl<SIN_V4F32>;
  def _ZGVnN4v_sinhf : RuntimeLibcallImpl<SINH_V4F32>;
  def _ZGVnN4v_sinpif : RuntimeLibcallImpl<SINPI_V4F32>;
  def _ZGVnN4v_sqrtf : RuntimeLibcallImpl<SQRT_V4F32>;
  def _ZGVnN4v_tanf : RuntimeLibcallImpl<TAN_V4F32>;
  def _ZGVnN4v_tanhf : RuntimeLibcallImpl<TANH_V4F32>;
  def _ZGVnN4v_tgammaf : RuntimeLibcallImpl<TGAMMA_V4F32>;
  def _ZGVnN4vl4_modff : RuntimeLibcallImpl<MODF_V4F32>;
  def _ZGVnN4vl4l4_sincosf : RuntimeLibcallImpl<SINCOS_V4F32>;
  def _ZGVnN4vl4l4_sincospif : RuntimeLibcallImpl<SINCOSPI_V4F32>;
  def _ZGVnN4vv_atan2f : RuntimeLibcallImpl<ATAN2_V4F32>;
  def _ZGVnN4vv_copysignf : RuntimeLibcallImpl<COPYSIGN_V4F32>;
  def _ZGVnN4vv_fdimf : RuntimeLibcallImpl<FDIM_V4F32>;
  def _ZGVnN4vv_fmaxf : RuntimeLibcallImpl<FMAX_V4F32>;
  def _ZGVnN4vv_fminf : RuntimeLibcallImpl<FMIN_V4F32>;
  def _ZGVnN4vv_fmodf : RuntimeLibcallImpl<REM_V4F32>;
  def _ZGVnN4vv_hypotf : RuntimeLibcallImpl<HYPOT_V4F32>;
  def _ZGVnN4vv_ldexpf : RuntimeLibcallImpl<LDEXP_V4F32>;
  def _ZGVnN4vv_nextafterf : RuntimeLibcallImpl<NEXTAFTER_V4F32>;
  def _ZGVnN4vv_powf : RuntimeLibcallImpl<POW_V4F32>;
  def _ZGVnN4vvv_fmaf : RuntimeLibcallImpl<FMA_V4F32>;
}

defset list<RuntimeLibcallImpl> SLEEFGNUABI_SCALABLE_VECFUNCS = {
  def _ZGVsMxv_acos : RuntimeLibcallImpl<ACOS_NXV2F64>;
  def _ZGVsMxv_acosf : RuntimeLibcallImpl<ACOS_NXV4F32>;
````
- **L4033 EN**: Declares TableGen def `_ZGVnN4v_ilogbf`.
  **L4033 CN**: 声明 TableGen def `_ZGVnN4v_ilogbf`。
- **L4034 EN**: Declares TableGen def `_ZGVnN4v_lgammaf`.
  **L4034 CN**: 声明 TableGen def `_ZGVnN4v_lgammaf`。
- **L4035 EN**: Declares TableGen def `_ZGVnN4v_log10f`.
  **L4035 CN**: 声明 TableGen def `_ZGVnN4v_log10f`。
- **L4036 EN**: Declares TableGen def `_ZGVnN4v_log1pf`.
  **L4036 CN**: 声明 TableGen def `_ZGVnN4v_log1pf`。
- **L4037 EN**: Declares TableGen def `_ZGVnN4v_log2f`.
  **L4037 CN**: 声明 TableGen def `_ZGVnN4v_log2f`。
- **L4038 EN**: Declares TableGen def `_ZGVnN4v_logf`.
  **L4038 CN**: 声明 TableGen def `_ZGVnN4v_logf`。
- **L4039 EN**: Declares TableGen def `_ZGVnN4v_sinf`.
  **L4039 CN**: 声明 TableGen def `_ZGVnN4v_sinf`。
- **L4040 EN**: Declares TableGen def `_ZGVnN4v_sinhf`.
  **L4040 CN**: 声明 TableGen def `_ZGVnN4v_sinhf`。
- **L4041 EN**: Declares TableGen def `_ZGVnN4v_sinpif`.
  **L4041 CN**: 声明 TableGen def `_ZGVnN4v_sinpif`。
- **L4042 EN**: Declares TableGen def `_ZGVnN4v_sqrtf`.
  **L4042 CN**: 声明 TableGen def `_ZGVnN4v_sqrtf`。
- **L4043 EN**: Declares TableGen def `_ZGVnN4v_tanf`.
  **L4043 CN**: 声明 TableGen def `_ZGVnN4v_tanf`。
- **L4044 EN**: Declares TableGen def `_ZGVnN4v_tanhf`.
  **L4044 CN**: 声明 TableGen def `_ZGVnN4v_tanhf`。
- **L4045 EN**: Declares TableGen def `_ZGVnN4v_tgammaf`.
  **L4045 CN**: 声明 TableGen def `_ZGVnN4v_tgammaf`。
- **L4046 EN**: Declares TableGen def `_ZGVnN4vl4_modff`.
  **L4046 CN**: 声明 TableGen def `_ZGVnN4vl4_modff`。
- **L4047 EN**: Declares TableGen def `_ZGVnN4vl4l4_sincosf`.
  **L4047 CN**: 声明 TableGen def `_ZGVnN4vl4l4_sincosf`。
- **L4048 EN**: Declares TableGen def `_ZGVnN4vl4l4_sincospif`.
  **L4048 CN**: 声明 TableGen def `_ZGVnN4vl4l4_sincospif`。
- **L4049 EN**: Declares TableGen def `_ZGVnN4vv_atan2f`.
  **L4049 CN**: 声明 TableGen def `_ZGVnN4vv_atan2f`。
- **L4050 EN**: Declares TableGen def `_ZGVnN4vv_copysignf`.
  **L4050 CN**: 声明 TableGen def `_ZGVnN4vv_copysignf`。
- **L4051 EN**: Declares TableGen def `_ZGVnN4vv_fdimf`.
  **L4051 CN**: 声明 TableGen def `_ZGVnN4vv_fdimf`。
- **L4052 EN**: Declares TableGen def `_ZGVnN4vv_fmaxf`.
  **L4052 CN**: 声明 TableGen def `_ZGVnN4vv_fmaxf`。
- **L4053 EN**: Declares TableGen def `_ZGVnN4vv_fminf`.
  **L4053 CN**: 声明 TableGen def `_ZGVnN4vv_fminf`。
- **L4054 EN**: Declares TableGen def `_ZGVnN4vv_fmodf`.
  **L4054 CN**: 声明 TableGen def `_ZGVnN4vv_fmodf`。
- **L4055 EN**: Declares TableGen def `_ZGVnN4vv_hypotf`.
  **L4055 CN**: 声明 TableGen def `_ZGVnN4vv_hypotf`。
- **L4056 EN**: Declares TableGen def `_ZGVnN4vv_ldexpf`.
  **L4056 CN**: 声明 TableGen def `_ZGVnN4vv_ldexpf`。
- **L4057 EN**: Declares TableGen def `_ZGVnN4vv_nextafterf`.
  **L4057 CN**: 声明 TableGen def `_ZGVnN4vv_nextafterf`。
- **L4058 EN**: Declares TableGen def `_ZGVnN4vv_powf`.
  **L4058 CN**: 声明 TableGen def `_ZGVnN4vv_powf`。
- **L4059 EN**: Declares TableGen def `_ZGVnN4vvv_fmaf`.
  **L4059 CN**: 声明 TableGen def `_ZGVnN4vvv_fmaf`。
- **L4060 EN**: Closes the current lexical scope or compound statement.
  **L4060 CN**: 结束当前词法作用域或复合语句块。
- **L4061 EN**: Blank line separating nearby declarations or logic blocks.
  **L4061 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L4062 EN**: Continues the surrounding expression or declaration: `defset list<RuntimeLibcallImpl> SLEEFGNUABI_SCALABLE_VECFUNCS = {`.
  **L4062 CN**: 继续构造周围的表达式或声明：`defset list<RuntimeLibcallImpl> SLEEFGNUABI_SCALABLE_VECFUNCS = {`。
- **L4063 EN**: Declares TableGen def `_ZGVsMxv_acos`.
  **L4063 CN**: 声明 TableGen def `_ZGVsMxv_acos`。
- **L4064 EN**: Declares TableGen def `_ZGVsMxv_acosf`.
  **L4064 CN**: 声明 TableGen def `_ZGVsMxv_acosf`。

### Lines 4065-4096

````tablegen
  def _ZGVsMxv_acosh : RuntimeLibcallImpl<ACOSH_NXV2F64>;
  def _ZGVsMxv_acoshf : RuntimeLibcallImpl<ACOSH_NXV4F32>;
  def _ZGVsMxv_asin : RuntimeLibcallImpl<ASIN_NXV2F64>;
  def _ZGVsMxv_asinf : RuntimeLibcallImpl<ASIN_NXV4F32>;
  def _ZGVsMxv_asinh : RuntimeLibcallImpl<ASINH_NXV2F64>;
  def _ZGVsMxv_asinhf : RuntimeLibcallImpl<ASINH_NXV4F32>;
  def _ZGVsMxv_atan : RuntimeLibcallImpl<ATAN_NXV2F64>;
  def _ZGVsMxv_atanf : RuntimeLibcallImpl<ATAN_NXV4F32>;
  def _ZGVsMxv_atanh : RuntimeLibcallImpl<ATANH_NXV2F64>;
  def _ZGVsMxv_atanhf : RuntimeLibcallImpl<ATANH_NXV4F32>;
  def _ZGVsMxv_cbrt : RuntimeLibcallImpl<CBRT_NXV2F64>;
  def _ZGVsMxv_cbrtf : RuntimeLibcallImpl<CBRT_NXV4F32>;
  def _ZGVsMxv_cos : RuntimeLibcallImpl<COS_NXV2F64>;
  def _ZGVsMxv_cosf : RuntimeLibcallImpl<COS_NXV4F32>;
  def _ZGVsMxv_cosh : RuntimeLibcallImpl<COSH_NXV2F64>;
  def _ZGVsMxv_coshf : RuntimeLibcallImpl<COSH_NXV4F32>;
  def _ZGVsMxv_cospi : RuntimeLibcallImpl<COSPI_NXV2F64>;
  def _ZGVsMxv_cospif : RuntimeLibcallImpl<COSPI_NXV4F32>;
  def _ZGVsMxv_erf : RuntimeLibcallImpl<ERF_NXV2F64>;
  def _ZGVsMxv_erfc : RuntimeLibcallImpl<ERFC_NXV2F64>;
  def _ZGVsMxv_erfcf : RuntimeLibcallImpl<ERFC_NXV4F32>;
  def _ZGVsMxv_erff : RuntimeLibcallImpl<ERF_NXV4F32>;
  def _ZGVsMxv_exp : RuntimeLibcallImpl<EXP_NXV2F64>;
  def _ZGVsMxv_exp10 : RuntimeLibcallImpl<EXP10_NXV2F64>;
  def _ZGVsMxv_exp10f : RuntimeLibcallImpl<EXP10_NXV4F32>;
  def _ZGVsMxv_exp2 : RuntimeLibcallImpl<EXP2_NXV2F64>;
  def _ZGVsMxv_exp2f : RuntimeLibcallImpl<EXP2_NXV4F32>;
  def _ZGVsMxv_expf : RuntimeLibcallImpl<EXP_NXV4F32>;
  def _ZGVsMxv_expm1 : RuntimeLibcallImpl<EXPM1_NXV2F64>;
  def _ZGVsMxv_expm1f : RuntimeLibcallImpl<EXPM1_NXV4F32>;
  def _ZGVsMxv_ilogb : RuntimeLibcallImpl<ILOGB_NXV2F64>;
  def _ZGVsMxv_ilogbf : RuntimeLibcallImpl<ILOGB_NXV4F32>;
````
- **L4065 EN**: Declares TableGen def `_ZGVsMxv_acosh`.
  **L4065 CN**: 声明 TableGen def `_ZGVsMxv_acosh`。
- **L4066 EN**: Declares TableGen def `_ZGVsMxv_acoshf`.
  **L4066 CN**: 声明 TableGen def `_ZGVsMxv_acoshf`。
- **L4067 EN**: Declares TableGen def `_ZGVsMxv_asin`.
  **L4067 CN**: 声明 TableGen def `_ZGVsMxv_asin`。
- **L4068 EN**: Declares TableGen def `_ZGVsMxv_asinf`.
  **L4068 CN**: 声明 TableGen def `_ZGVsMxv_asinf`。
- **L4069 EN**: Declares TableGen def `_ZGVsMxv_asinh`.
  **L4069 CN**: 声明 TableGen def `_ZGVsMxv_asinh`。
- **L4070 EN**: Declares TableGen def `_ZGVsMxv_asinhf`.
  **L4070 CN**: 声明 TableGen def `_ZGVsMxv_asinhf`。
- **L4071 EN**: Declares TableGen def `_ZGVsMxv_atan`.
  **L4071 CN**: 声明 TableGen def `_ZGVsMxv_atan`。
- **L4072 EN**: Declares TableGen def `_ZGVsMxv_atanf`.
  **L4072 CN**: 声明 TableGen def `_ZGVsMxv_atanf`。
- **L4073 EN**: Declares TableGen def `_ZGVsMxv_atanh`.
  **L4073 CN**: 声明 TableGen def `_ZGVsMxv_atanh`。
- **L4074 EN**: Declares TableGen def `_ZGVsMxv_atanhf`.
  **L4074 CN**: 声明 TableGen def `_ZGVsMxv_atanhf`。
- **L4075 EN**: Declares TableGen def `_ZGVsMxv_cbrt`.
  **L4075 CN**: 声明 TableGen def `_ZGVsMxv_cbrt`。
- **L4076 EN**: Declares TableGen def `_ZGVsMxv_cbrtf`.
  **L4076 CN**: 声明 TableGen def `_ZGVsMxv_cbrtf`。
- **L4077 EN**: Declares TableGen def `_ZGVsMxv_cos`.
  **L4077 CN**: 声明 TableGen def `_ZGVsMxv_cos`。
- **L4078 EN**: Declares TableGen def `_ZGVsMxv_cosf`.
  **L4078 CN**: 声明 TableGen def `_ZGVsMxv_cosf`。
- **L4079 EN**: Declares TableGen def `_ZGVsMxv_cosh`.
  **L4079 CN**: 声明 TableGen def `_ZGVsMxv_cosh`。
- **L4080 EN**: Declares TableGen def `_ZGVsMxv_coshf`.
  **L4080 CN**: 声明 TableGen def `_ZGVsMxv_coshf`。
- **L4081 EN**: Declares TableGen def `_ZGVsMxv_cospi`.
  **L4081 CN**: 声明 TableGen def `_ZGVsMxv_cospi`。
- **L4082 EN**: Declares TableGen def `_ZGVsMxv_cospif`.
  **L4082 CN**: 声明 TableGen def `_ZGVsMxv_cospif`。
- **L4083 EN**: Declares TableGen def `_ZGVsMxv_erf`.
  **L4083 CN**: 声明 TableGen def `_ZGVsMxv_erf`。
- **L4084 EN**: Declares TableGen def `_ZGVsMxv_erfc`.
  **L4084 CN**: 声明 TableGen def `_ZGVsMxv_erfc`。
- **L4085 EN**: Declares TableGen def `_ZGVsMxv_erfcf`.
  **L4085 CN**: 声明 TableGen def `_ZGVsMxv_erfcf`。
- **L4086 EN**: Declares TableGen def `_ZGVsMxv_erff`.
  **L4086 CN**: 声明 TableGen def `_ZGVsMxv_erff`。
- **L4087 EN**: Declares TableGen def `_ZGVsMxv_exp`.
  **L4087 CN**: 声明 TableGen def `_ZGVsMxv_exp`。
- **L4088 EN**: Declares TableGen def `_ZGVsMxv_exp10`.
  **L4088 CN**: 声明 TableGen def `_ZGVsMxv_exp10`。
- **L4089 EN**: Declares TableGen def `_ZGVsMxv_exp10f`.
  **L4089 CN**: 声明 TableGen def `_ZGVsMxv_exp10f`。
- **L4090 EN**: Declares TableGen def `_ZGVsMxv_exp2`.
  **L4090 CN**: 声明 TableGen def `_ZGVsMxv_exp2`。
- **L4091 EN**: Declares TableGen def `_ZGVsMxv_exp2f`.
  **L4091 CN**: 声明 TableGen def `_ZGVsMxv_exp2f`。
- **L4092 EN**: Declares TableGen def `_ZGVsMxv_expf`.
  **L4092 CN**: 声明 TableGen def `_ZGVsMxv_expf`。
- **L4093 EN**: Declares TableGen def `_ZGVsMxv_expm1`.
  **L4093 CN**: 声明 TableGen def `_ZGVsMxv_expm1`。
- **L4094 EN**: Declares TableGen def `_ZGVsMxv_expm1f`.
  **L4094 CN**: 声明 TableGen def `_ZGVsMxv_expm1f`。
- **L4095 EN**: Declares TableGen def `_ZGVsMxv_ilogb`.
  **L4095 CN**: 声明 TableGen def `_ZGVsMxv_ilogb`。
- **L4096 EN**: Declares TableGen def `_ZGVsMxv_ilogbf`.
  **L4096 CN**: 声明 TableGen def `_ZGVsMxv_ilogbf`。

### Lines 4097-4128

````tablegen
  def _ZGVsMxv_lgamma : RuntimeLibcallImpl<LGAMMA_NXV2F64>;
  def _ZGVsMxv_lgammaf : RuntimeLibcallImpl<LGAMMA_NXV4F32>;
  def _ZGVsMxv_log : RuntimeLibcallImpl<LOG_NXV2F64>;
  def _ZGVsMxv_log10 : RuntimeLibcallImpl<LOG10_NXV2F64>;
  def _ZGVsMxv_log10f : RuntimeLibcallImpl<LOG10_NXV4F32>;
  def _ZGVsMxv_log1p : RuntimeLibcallImpl<LOG1P_NXV2F64>;
  def _ZGVsMxv_log1pf : RuntimeLibcallImpl<LOG1P_NXV4F32>;
  def _ZGVsMxv_log2 : RuntimeLibcallImpl<LOG2_NXV2F64>;
  def _ZGVsMxv_log2f : RuntimeLibcallImpl<LOG2_NXV4F32>;
  def _ZGVsMxv_logf : RuntimeLibcallImpl<LOG_NXV4F32>;
  def _ZGVsMxv_sin : RuntimeLibcallImpl<SIN_NXV2F64>;
  def _ZGVsMxv_sinf : RuntimeLibcallImpl<SIN_NXV4F32>;
  def _ZGVsMxv_sinh : RuntimeLibcallImpl<SINH_NXV2F64>;
  def _ZGVsMxv_sinhf : RuntimeLibcallImpl<SINH_NXV4F32>;
  def _ZGVsMxv_sinpi : RuntimeLibcallImpl<SINPI_NXV2F64>;
  def _ZGVsMxv_sinpif : RuntimeLibcallImpl<SINPI_NXV4F32>;
  def _ZGVsMxv_sqrt : RuntimeLibcallImpl<SQRT_NXV2F64>;
  def _ZGVsMxv_sqrtf : RuntimeLibcallImpl<SQRT_NXV4F32>;
  def _ZGVsMxv_tan : RuntimeLibcallImpl<TAN_NXV2F64>;
  def _ZGVsMxv_tanf : RuntimeLibcallImpl<TAN_NXV4F32>;
  def _ZGVsMxv_tanh : RuntimeLibcallImpl<TANH_NXV2F64>;
  def _ZGVsMxv_tanhf : RuntimeLibcallImpl<TANH_NXV4F32>;
  def _ZGVsMxv_tgamma : RuntimeLibcallImpl<TGAMMA_NXV2F64>;
  def _ZGVsMxv_tgammaf : RuntimeLibcallImpl<TGAMMA_NXV4F32>;
  def _ZGVsMxvv_atan2 : RuntimeLibcallImpl<ATAN2_NXV2F64>;
  def _ZGVsMxvv_atan2f : RuntimeLibcallImpl<ATAN2_NXV4F32>;
  def _ZGVsMxvv_copysign : RuntimeLibcallImpl<COPYSIGN_NXV2F64>;
  def _ZGVsMxvv_copysignf : RuntimeLibcallImpl<COPYSIGN_NXV4F32>;
  def _ZGVsMxvv_fdim : RuntimeLibcallImpl<FDIM_NXV2F64>;
  def _ZGVsMxvv_fdimf : RuntimeLibcallImpl<FDIM_NXV4F32>;
  def _ZGVsMxvv_fmax : RuntimeLibcallImpl<FMAX_NXV2F64>;
  def _ZGVsMxvv_fmaxf : RuntimeLibcallImpl<FMAX_NXV4F32>;
````
- **L4097 EN**: Declares TableGen def `_ZGVsMxv_lgamma`.
  **L4097 CN**: 声明 TableGen def `_ZGVsMxv_lgamma`。
- **L4098 EN**: Declares TableGen def `_ZGVsMxv_lgammaf`.
  **L4098 CN**: 声明 TableGen def `_ZGVsMxv_lgammaf`。
- **L4099 EN**: Declares TableGen def `_ZGVsMxv_log`.
  **L4099 CN**: 声明 TableGen def `_ZGVsMxv_log`。
- **L4100 EN**: Declares TableGen def `_ZGVsMxv_log10`.
  **L4100 CN**: 声明 TableGen def `_ZGVsMxv_log10`。
- **L4101 EN**: Declares TableGen def `_ZGVsMxv_log10f`.
  **L4101 CN**: 声明 TableGen def `_ZGVsMxv_log10f`。
- **L4102 EN**: Declares TableGen def `_ZGVsMxv_log1p`.
  **L4102 CN**: 声明 TableGen def `_ZGVsMxv_log1p`。
- **L4103 EN**: Declares TableGen def `_ZGVsMxv_log1pf`.
  **L4103 CN**: 声明 TableGen def `_ZGVsMxv_log1pf`。
- **L4104 EN**: Declares TableGen def `_ZGVsMxv_log2`.
  **L4104 CN**: 声明 TableGen def `_ZGVsMxv_log2`。
- **L4105 EN**: Declares TableGen def `_ZGVsMxv_log2f`.
  **L4105 CN**: 声明 TableGen def `_ZGVsMxv_log2f`。
- **L4106 EN**: Declares TableGen def `_ZGVsMxv_logf`.
  **L4106 CN**: 声明 TableGen def `_ZGVsMxv_logf`。
- **L4107 EN**: Declares TableGen def `_ZGVsMxv_sin`.
  **L4107 CN**: 声明 TableGen def `_ZGVsMxv_sin`。
- **L4108 EN**: Declares TableGen def `_ZGVsMxv_sinf`.
  **L4108 CN**: 声明 TableGen def `_ZGVsMxv_sinf`。
- **L4109 EN**: Declares TableGen def `_ZGVsMxv_sinh`.
  **L4109 CN**: 声明 TableGen def `_ZGVsMxv_sinh`。
- **L4110 EN**: Declares TableGen def `_ZGVsMxv_sinhf`.
  **L4110 CN**: 声明 TableGen def `_ZGVsMxv_sinhf`。
- **L4111 EN**: Declares TableGen def `_ZGVsMxv_sinpi`.
  **L4111 CN**: 声明 TableGen def `_ZGVsMxv_sinpi`。
- **L4112 EN**: Declares TableGen def `_ZGVsMxv_sinpif`.
  **L4112 CN**: 声明 TableGen def `_ZGVsMxv_sinpif`。
- **L4113 EN**: Declares TableGen def `_ZGVsMxv_sqrt`.
  **L4113 CN**: 声明 TableGen def `_ZGVsMxv_sqrt`。
- **L4114 EN**: Declares TableGen def `_ZGVsMxv_sqrtf`.
  **L4114 CN**: 声明 TableGen def `_ZGVsMxv_sqrtf`。
- **L4115 EN**: Declares TableGen def `_ZGVsMxv_tan`.
  **L4115 CN**: 声明 TableGen def `_ZGVsMxv_tan`。
- **L4116 EN**: Declares TableGen def `_ZGVsMxv_tanf`.
  **L4116 CN**: 声明 TableGen def `_ZGVsMxv_tanf`。
- **L4117 EN**: Declares TableGen def `_ZGVsMxv_tanh`.
  **L4117 CN**: 声明 TableGen def `_ZGVsMxv_tanh`。
- **L4118 EN**: Declares TableGen def `_ZGVsMxv_tanhf`.
  **L4118 CN**: 声明 TableGen def `_ZGVsMxv_tanhf`。
- **L4119 EN**: Declares TableGen def `_ZGVsMxv_tgamma`.
  **L4119 CN**: 声明 TableGen def `_ZGVsMxv_tgamma`。
- **L4120 EN**: Declares TableGen def `_ZGVsMxv_tgammaf`.
  **L4120 CN**: 声明 TableGen def `_ZGVsMxv_tgammaf`。
- **L4121 EN**: Declares TableGen def `_ZGVsMxvv_atan2`.
  **L4121 CN**: 声明 TableGen def `_ZGVsMxvv_atan2`。
- **L4122 EN**: Declares TableGen def `_ZGVsMxvv_atan2f`.
  **L4122 CN**: 声明 TableGen def `_ZGVsMxvv_atan2f`。
- **L4123 EN**: Declares TableGen def `_ZGVsMxvv_copysign`.
  **L4123 CN**: 声明 TableGen def `_ZGVsMxvv_copysign`。
- **L4124 EN**: Declares TableGen def `_ZGVsMxvv_copysignf`.
  **L4124 CN**: 声明 TableGen def `_ZGVsMxvv_copysignf`。
- **L4125 EN**: Declares TableGen def `_ZGVsMxvv_fdim`.
  **L4125 CN**: 声明 TableGen def `_ZGVsMxvv_fdim`。
- **L4126 EN**: Declares TableGen def `_ZGVsMxvv_fdimf`.
  **L4126 CN**: 声明 TableGen def `_ZGVsMxvv_fdimf`。
- **L4127 EN**: Declares TableGen def `_ZGVsMxvv_fmax`.
  **L4127 CN**: 声明 TableGen def `_ZGVsMxvv_fmax`。
- **L4128 EN**: Declares TableGen def `_ZGVsMxvv_fmaxf`.
  **L4128 CN**: 声明 TableGen def `_ZGVsMxvv_fmaxf`。

### Lines 4129-4160

````tablegen
  def _ZGVsMxvv_fmin : RuntimeLibcallImpl<FMIN_NXV2F64>;
  def _ZGVsMxvv_fminf : RuntimeLibcallImpl<FMIN_NXV4F32>;
  def _ZGVsMxvv_fmod : RuntimeLibcallImpl<REM_NXV2F64>;
  def _ZGVsMxvv_fmodf : RuntimeLibcallImpl<REM_NXV4F32>;
  def _ZGVsMxvv_hypot : RuntimeLibcallImpl<HYPOT_NXV2F64>;
  def _ZGVsMxvv_hypotf : RuntimeLibcallImpl<HYPOT_NXV4F32>;
  def _ZGVsMxvv_ldexp : RuntimeLibcallImpl<LDEXP_NXV2F64>;
  def _ZGVsMxvv_ldexpf : RuntimeLibcallImpl<LDEXP_NXV4F32>;
  def _ZGVsMxvv_nextafter : RuntimeLibcallImpl<NEXTAFTER_NXV2F64>;
  def _ZGVsMxvv_nextafterf : RuntimeLibcallImpl<NEXTAFTER_NXV4F32>;
  def _ZGVsMxvv_pow : RuntimeLibcallImpl<POW_NXV2F64>;
  def _ZGVsMxvv_powf : RuntimeLibcallImpl<POW_NXV4F32>;
  def _ZGVsMxvvv_fma : RuntimeLibcallImpl<FMA_NXV2F64>;
  def _ZGVsMxvvv_fmaf : RuntimeLibcallImpl<FMA_NXV4F32>;
  def _ZGVsNxvl8_modf : RuntimeLibcallImpl<MODF_NXV2F64>;
  def _ZGVsNxvl4_modff : RuntimeLibcallImpl<MODF_NXV4F32>;
  def _ZGVsNxvl4l4_sincosf : RuntimeLibcallImpl<SINCOS_NXV4F32>;
  def _ZGVsNxvl4l4_sincospif : RuntimeLibcallImpl<SINCOSPI_NXV4F32>;
  def _ZGVsNxvl8l8_sincos : RuntimeLibcallImpl<SINCOS_NXV2F64>;
  def _ZGVsNxvl8l8_sincospi : RuntimeLibcallImpl<SINCOSPI_NXV2F64>;
}

defset list<RuntimeLibcallImpl> SLEEFGNUABI_SCALABLE_VECFUNCS_RISCV = {
  def Sleef_acosdx_u10rvvm2 : RuntimeLibcallImpl<ACOS_NXV2F64>;
  def Sleef_acosfx_u10rvvm2 : RuntimeLibcallImpl<ACOS_NXV4F32>;
  def Sleef_acoshdx_u10rvvm2 : RuntimeLibcallImpl<ACOSH_NXV2F64>;
  def Sleef_acoshfx_u10rvvm2 : RuntimeLibcallImpl<ACOSH_NXV4F32>;
  def Sleef_asindx_u10rvvm2 : RuntimeLibcallImpl<ASIN_NXV2F64>;
  def Sleef_asinfx_u10rvvm2 : RuntimeLibcallImpl<ASIN_NXV4F32>;
  def Sleef_asinhdx_u10rvvm2 : RuntimeLibcallImpl<ASINH_NXV2F64>;
  def Sleef_asinhfx_u10rvvm2 : RuntimeLibcallImpl<ASINH_NXV4F32>;
  def Sleef_atan2dx_u10rvvm2 : RuntimeLibcallImpl<ATAN2_NXV2F64>;
````
- **L4129 EN**: Declares TableGen def `_ZGVsMxvv_fmin`.
  **L4129 CN**: 声明 TableGen def `_ZGVsMxvv_fmin`。
- **L4130 EN**: Declares TableGen def `_ZGVsMxvv_fminf`.
  **L4130 CN**: 声明 TableGen def `_ZGVsMxvv_fminf`。
- **L4131 EN**: Declares TableGen def `_ZGVsMxvv_fmod`.
  **L4131 CN**: 声明 TableGen def `_ZGVsMxvv_fmod`。
- **L4132 EN**: Declares TableGen def `_ZGVsMxvv_fmodf`.
  **L4132 CN**: 声明 TableGen def `_ZGVsMxvv_fmodf`。
- **L4133 EN**: Declares TableGen def `_ZGVsMxvv_hypot`.
  **L4133 CN**: 声明 TableGen def `_ZGVsMxvv_hypot`。
- **L4134 EN**: Declares TableGen def `_ZGVsMxvv_hypotf`.
  **L4134 CN**: 声明 TableGen def `_ZGVsMxvv_hypotf`。
- **L4135 EN**: Declares TableGen def `_ZGVsMxvv_ldexp`.
  **L4135 CN**: 声明 TableGen def `_ZGVsMxvv_ldexp`。
- **L4136 EN**: Declares TableGen def `_ZGVsMxvv_ldexpf`.
  **L4136 CN**: 声明 TableGen def `_ZGVsMxvv_ldexpf`。
- **L4137 EN**: Declares TableGen def `_ZGVsMxvv_nextafter`.
  **L4137 CN**: 声明 TableGen def `_ZGVsMxvv_nextafter`。
- **L4138 EN**: Declares TableGen def `_ZGVsMxvv_nextafterf`.
  **L4138 CN**: 声明 TableGen def `_ZGVsMxvv_nextafterf`。
- **L4139 EN**: Declares TableGen def `_ZGVsMxvv_pow`.
  **L4139 CN**: 声明 TableGen def `_ZGVsMxvv_pow`。
- **L4140 EN**: Declares TableGen def `_ZGVsMxvv_powf`.
  **L4140 CN**: 声明 TableGen def `_ZGVsMxvv_powf`。
- **L4141 EN**: Declares TableGen def `_ZGVsMxvvv_fma`.
  **L4141 CN**: 声明 TableGen def `_ZGVsMxvvv_fma`。
- **L4142 EN**: Declares TableGen def `_ZGVsMxvvv_fmaf`.
  **L4142 CN**: 声明 TableGen def `_ZGVsMxvvv_fmaf`。
- **L4143 EN**: Declares TableGen def `_ZGVsNxvl8_modf`.
  **L4143 CN**: 声明 TableGen def `_ZGVsNxvl8_modf`。
- **L4144 EN**: Declares TableGen def `_ZGVsNxvl4_modff`.
  **L4144 CN**: 声明 TableGen def `_ZGVsNxvl4_modff`。
- **L4145 EN**: Declares TableGen def `_ZGVsNxvl4l4_sincosf`.
  **L4145 CN**: 声明 TableGen def `_ZGVsNxvl4l4_sincosf`。
- **L4146 EN**: Declares TableGen def `_ZGVsNxvl4l4_sincospif`.
  **L4146 CN**: 声明 TableGen def `_ZGVsNxvl4l4_sincospif`。
- **L4147 EN**: Declares TableGen def `_ZGVsNxvl8l8_sincos`.
  **L4147 CN**: 声明 TableGen def `_ZGVsNxvl8l8_sincos`。
- **L4148 EN**: Declares TableGen def `_ZGVsNxvl8l8_sincospi`.
  **L4148 CN**: 声明 TableGen def `_ZGVsNxvl8l8_sincospi`。
- **L4149 EN**: Closes the current lexical scope or compound statement.
  **L4149 CN**: 结束当前词法作用域或复合语句块。
- **L4150 EN**: Blank line separating nearby declarations or logic blocks.
  **L4150 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L4151 EN**: Continues the surrounding expression or declaration: `defset list<RuntimeLibcallImpl> SLEEFGNUABI_SCALABLE_VECFUNCS_RISCV = {`.
  **L4151 CN**: 继续构造周围的表达式或声明：`defset list<RuntimeLibcallImpl> SLEEFGNUABI_SCALABLE_VECFUNCS_RISCV = {`。
- **L4152 EN**: Declares TableGen def `Sleef_acosdx_u10rvvm2`.
  **L4152 CN**: 声明 TableGen def `Sleef_acosdx_u10rvvm2`。
- **L4153 EN**: Declares TableGen def `Sleef_acosfx_u10rvvm2`.
  **L4153 CN**: 声明 TableGen def `Sleef_acosfx_u10rvvm2`。
- **L4154 EN**: Declares TableGen def `Sleef_acoshdx_u10rvvm2`.
  **L4154 CN**: 声明 TableGen def `Sleef_acoshdx_u10rvvm2`。
- **L4155 EN**: Declares TableGen def `Sleef_acoshfx_u10rvvm2`.
  **L4155 CN**: 声明 TableGen def `Sleef_acoshfx_u10rvvm2`。
- **L4156 EN**: Declares TableGen def `Sleef_asindx_u10rvvm2`.
  **L4156 CN**: 声明 TableGen def `Sleef_asindx_u10rvvm2`。
- **L4157 EN**: Declares TableGen def `Sleef_asinfx_u10rvvm2`.
  **L4157 CN**: 声明 TableGen def `Sleef_asinfx_u10rvvm2`。
- **L4158 EN**: Declares TableGen def `Sleef_asinhdx_u10rvvm2`.
  **L4158 CN**: 声明 TableGen def `Sleef_asinhdx_u10rvvm2`。
- **L4159 EN**: Declares TableGen def `Sleef_asinhfx_u10rvvm2`.
  **L4159 CN**: 声明 TableGen def `Sleef_asinhfx_u10rvvm2`。
- **L4160 EN**: Declares TableGen def `Sleef_atan2dx_u10rvvm2`.
  **L4160 CN**: 声明 TableGen def `Sleef_atan2dx_u10rvvm2`。

### Lines 4161-4192

````tablegen
  def Sleef_atan2fx_u10rvvm2 : RuntimeLibcallImpl<ATAN2_NXV4F32>;
  def Sleef_atandx_u10rvvm2 : RuntimeLibcallImpl<ATAN_NXV2F64>;
  def Sleef_atanfx_u10rvvm2 : RuntimeLibcallImpl<ATAN_NXV4F32>;
  def Sleef_atanhdx_u10rvvm2 : RuntimeLibcallImpl<ATANH_NXV2F64>;
  def Sleef_atanhfx_u10rvvm2 : RuntimeLibcallImpl<ATANH_NXV4F32>;
  def Sleef_cbrtdx_u10rvvm2 : RuntimeLibcallImpl<CBRT_NXV2F64>;
  def Sleef_cbrtfx_u10rvvm2 : RuntimeLibcallImpl<CBRT_NXV4F32>;
  def Sleef_copysigndx_rvvm2 : RuntimeLibcallImpl<COPYSIGN_NXV2F64>;
  def Sleef_copysignfx_rvvm2 : RuntimeLibcallImpl<COPYSIGN_NXV4F32>;
  def Sleef_cosdx_u10rvvm2 : RuntimeLibcallImpl<COS_NXV2F64>;
  def Sleef_cosfx_u10rvvm2 : RuntimeLibcallImpl<COS_NXV4F32>;
  def Sleef_coshdx_u10rvvm2 : RuntimeLibcallImpl<COSH_NXV2F64>;
  def Sleef_coshfx_u10rvvm2 : RuntimeLibcallImpl<COSH_NXV4F32>;
  def Sleef_cospidx_u05rvvm2 : RuntimeLibcallImpl<COSPI_NXV2F64>;
  def Sleef_cospifx_u05rvvm2 : RuntimeLibcallImpl<COSPI_NXV4F32>;
  def Sleef_erfcdx_u15rvvm2 : RuntimeLibcallImpl<ERFC_NXV2F64>;
  def Sleef_erfcfx_u15rvvm2 : RuntimeLibcallImpl<ERFC_NXV4F32>;
  def Sleef_erfdx_u10rvvm2 : RuntimeLibcallImpl<ERF_NXV2F64>;
  def Sleef_erffx_u10rvvm2 : RuntimeLibcallImpl<ERF_NXV4F32>;
  def Sleef_exp10dx_u10rvvm2 : RuntimeLibcallImpl<EXP10_NXV2F64>;
  def Sleef_exp10fx_u10rvvm2 : RuntimeLibcallImpl<EXP10_NXV4F32>;
  def Sleef_exp2dx_u10rvvm2 : RuntimeLibcallImpl<EXP2_NXV2F64>;
  def Sleef_exp2fx_u10rvvm2 : RuntimeLibcallImpl<EXP2_NXV4F32>;
  def Sleef_expdx_u10rvvm2 : RuntimeLibcallImpl<EXP_NXV2F64>;
  def Sleef_expfx_u10rvvm2 : RuntimeLibcallImpl<EXP_NXV4F32>;
  def Sleef_expm1dx_u10rvvm2 : RuntimeLibcallImpl<EXPM1_NXV2F64>;
  def Sleef_expm1fx_u10rvvm2 : RuntimeLibcallImpl<EXPM1_NXV4F32>;
  def Sleef_fdimdx_rvvm2 : RuntimeLibcallImpl<FDIM_NXV2F64>;
  def Sleef_fdimfx_rvvm2 : RuntimeLibcallImpl<FDIM_NXV4F32>;
  def Sleef_fmadx_rvvm2 : RuntimeLibcallImpl<FMA_NXV2F64>;
  def Sleef_fmafx_rvvm2 : RuntimeLibcallImpl<FMA_NXV4F32>;
  def Sleef_fmaxdx_rvvm2 : RuntimeLibcallImpl<FMAX_NXV2F64>;
````
- **L4161 EN**: Declares TableGen def `Sleef_atan2fx_u10rvvm2`.
  **L4161 CN**: 声明 TableGen def `Sleef_atan2fx_u10rvvm2`。
- **L4162 EN**: Declares TableGen def `Sleef_atandx_u10rvvm2`.
  **L4162 CN**: 声明 TableGen def `Sleef_atandx_u10rvvm2`。
- **L4163 EN**: Declares TableGen def `Sleef_atanfx_u10rvvm2`.
  **L4163 CN**: 声明 TableGen def `Sleef_atanfx_u10rvvm2`。
- **L4164 EN**: Declares TableGen def `Sleef_atanhdx_u10rvvm2`.
  **L4164 CN**: 声明 TableGen def `Sleef_atanhdx_u10rvvm2`。
- **L4165 EN**: Declares TableGen def `Sleef_atanhfx_u10rvvm2`.
  **L4165 CN**: 声明 TableGen def `Sleef_atanhfx_u10rvvm2`。
- **L4166 EN**: Declares TableGen def `Sleef_cbrtdx_u10rvvm2`.
  **L4166 CN**: 声明 TableGen def `Sleef_cbrtdx_u10rvvm2`。
- **L4167 EN**: Declares TableGen def `Sleef_cbrtfx_u10rvvm2`.
  **L4167 CN**: 声明 TableGen def `Sleef_cbrtfx_u10rvvm2`。
- **L4168 EN**: Declares TableGen def `Sleef_copysigndx_rvvm2`.
  **L4168 CN**: 声明 TableGen def `Sleef_copysigndx_rvvm2`。
- **L4169 EN**: Declares TableGen def `Sleef_copysignfx_rvvm2`.
  **L4169 CN**: 声明 TableGen def `Sleef_copysignfx_rvvm2`。
- **L4170 EN**: Declares TableGen def `Sleef_cosdx_u10rvvm2`.
  **L4170 CN**: 声明 TableGen def `Sleef_cosdx_u10rvvm2`。
- **L4171 EN**: Declares TableGen def `Sleef_cosfx_u10rvvm2`.
  **L4171 CN**: 声明 TableGen def `Sleef_cosfx_u10rvvm2`。
- **L4172 EN**: Declares TableGen def `Sleef_coshdx_u10rvvm2`.
  **L4172 CN**: 声明 TableGen def `Sleef_coshdx_u10rvvm2`。
- **L4173 EN**: Declares TableGen def `Sleef_coshfx_u10rvvm2`.
  **L4173 CN**: 声明 TableGen def `Sleef_coshfx_u10rvvm2`。
- **L4174 EN**: Declares TableGen def `Sleef_cospidx_u05rvvm2`.
  **L4174 CN**: 声明 TableGen def `Sleef_cospidx_u05rvvm2`。
- **L4175 EN**: Declares TableGen def `Sleef_cospifx_u05rvvm2`.
  **L4175 CN**: 声明 TableGen def `Sleef_cospifx_u05rvvm2`。
- **L4176 EN**: Declares TableGen def `Sleef_erfcdx_u15rvvm2`.
  **L4176 CN**: 声明 TableGen def `Sleef_erfcdx_u15rvvm2`。
- **L4177 EN**: Declares TableGen def `Sleef_erfcfx_u15rvvm2`.
  **L4177 CN**: 声明 TableGen def `Sleef_erfcfx_u15rvvm2`。
- **L4178 EN**: Declares TableGen def `Sleef_erfdx_u10rvvm2`.
  **L4178 CN**: 声明 TableGen def `Sleef_erfdx_u10rvvm2`。
- **L4179 EN**: Declares TableGen def `Sleef_erffx_u10rvvm2`.
  **L4179 CN**: 声明 TableGen def `Sleef_erffx_u10rvvm2`。
- **L4180 EN**: Declares TableGen def `Sleef_exp10dx_u10rvvm2`.
  **L4180 CN**: 声明 TableGen def `Sleef_exp10dx_u10rvvm2`。
- **L4181 EN**: Declares TableGen def `Sleef_exp10fx_u10rvvm2`.
  **L4181 CN**: 声明 TableGen def `Sleef_exp10fx_u10rvvm2`。
- **L4182 EN**: Declares TableGen def `Sleef_exp2dx_u10rvvm2`.
  **L4182 CN**: 声明 TableGen def `Sleef_exp2dx_u10rvvm2`。
- **L4183 EN**: Declares TableGen def `Sleef_exp2fx_u10rvvm2`.
  **L4183 CN**: 声明 TableGen def `Sleef_exp2fx_u10rvvm2`。
- **L4184 EN**: Declares TableGen def `Sleef_expdx_u10rvvm2`.
  **L4184 CN**: 声明 TableGen def `Sleef_expdx_u10rvvm2`。
- **L4185 EN**: Declares TableGen def `Sleef_expfx_u10rvvm2`.
  **L4185 CN**: 声明 TableGen def `Sleef_expfx_u10rvvm2`。
- **L4186 EN**: Declares TableGen def `Sleef_expm1dx_u10rvvm2`.
  **L4186 CN**: 声明 TableGen def `Sleef_expm1dx_u10rvvm2`。
- **L4187 EN**: Declares TableGen def `Sleef_expm1fx_u10rvvm2`.
  **L4187 CN**: 声明 TableGen def `Sleef_expm1fx_u10rvvm2`。
- **L4188 EN**: Declares TableGen def `Sleef_fdimdx_rvvm2`.
  **L4188 CN**: 声明 TableGen def `Sleef_fdimdx_rvvm2`。
- **L4189 EN**: Declares TableGen def `Sleef_fdimfx_rvvm2`.
  **L4189 CN**: 声明 TableGen def `Sleef_fdimfx_rvvm2`。
- **L4190 EN**: Declares TableGen def `Sleef_fmadx_rvvm2`.
  **L4190 CN**: 声明 TableGen def `Sleef_fmadx_rvvm2`。
- **L4191 EN**: Declares TableGen def `Sleef_fmafx_rvvm2`.
  **L4191 CN**: 声明 TableGen def `Sleef_fmafx_rvvm2`。
- **L4192 EN**: Declares TableGen def `Sleef_fmaxdx_rvvm2`.
  **L4192 CN**: 声明 TableGen def `Sleef_fmaxdx_rvvm2`。

### Lines 4193-4224

````tablegen
  def Sleef_fmaxfx_rvvm2 : RuntimeLibcallImpl<FMAX_NXV4F32>;
  def Sleef_fmindx_u10rvvm2 : RuntimeLibcallImpl<FMIN_NXV2F64>;
  def Sleef_fminfx_u10rvvm2 : RuntimeLibcallImpl<FMIN_NXV4F32>;
  def Sleef_fmoddx_rvvm2 : RuntimeLibcallImpl<REM_NXV2F64>;
  def Sleef_fmodfx_rvvm2 : RuntimeLibcallImpl<REM_NXV4F32>;
  def Sleef_hypotdx_u05rvvm2 : RuntimeLibcallImpl<HYPOT_NXV2F64>;
  def Sleef_hypotfx_u05rvvm2 : RuntimeLibcallImpl<HYPOT_NXV4F32>;
  def Sleef_ilogbdx_rvvm2 : RuntimeLibcallImpl<ILOGB_NXV2F64>;
  def Sleef_ilogbfx_rvvm2 : RuntimeLibcallImpl<ILOGB_NXV4F32>;
  def Sleef_ldexpdx_rvvm2 : RuntimeLibcallImpl<LDEXP_NXV2F64>;
  def Sleef_ldexpfx_rvvm2 : RuntimeLibcallImpl<LDEXP_NXV4F32>;
  def Sleef_lgammadx_u10rvvm2 : RuntimeLibcallImpl<LGAMMA_NXV2F64>;
  def Sleef_lgammafx_u10rvvm2 : RuntimeLibcallImpl<LGAMMA_NXV4F32>;
  def Sleef_log10dx_u10rvvm2 : RuntimeLibcallImpl<LOG10_NXV2F64>;
  def Sleef_log10fx_u10rvvm2 : RuntimeLibcallImpl<LOG10_NXV4F32>;
  def Sleef_log1pdx_u10rvvm2 : RuntimeLibcallImpl<LOG1P_NXV2F64>;
  def Sleef_log1pfx_u10rvvm2 : RuntimeLibcallImpl<LOG1P_NXV4F32>;
  def Sleef_log2dx_u10rvvm2 : RuntimeLibcallImpl<LOG2_NXV2F64>;
  def Sleef_log2fx_u10rvvm2 : RuntimeLibcallImpl<LOG2_NXV4F32>;
  def Sleef_logdx_u10rvvm2 : RuntimeLibcallImpl<LOG_NXV2F64>;
  def Sleef_logfx_u10rvvm2 : RuntimeLibcallImpl<LOG_NXV4F32>;
  def Sleef_modfdx_rvvm2 : RuntimeLibcallImpl<MODF_NXV2F64>;
  def Sleef_modffx_rvvm2 : RuntimeLibcallImpl<MODF_NXV4F32>;
  def Sleef_nextafterdx_rvvm2 : RuntimeLibcallImpl<NEXTAFTER_NXV2F64>;
  def Sleef_nextafterfx_rvvm2 : RuntimeLibcallImpl<NEXTAFTER_NXV4F32>;
  def Sleef_powdx_u10rvvm2 : RuntimeLibcallImpl<POW_NXV2F64>;
  def Sleef_powfx_u10rvvm2 : RuntimeLibcallImpl<POW_NXV4F32>;
  def Sleef_sincosdx_u10rvvm2 : RuntimeLibcallImpl<SINCOS_NXV2F64>;
  def Sleef_sincosfx_u10rvvm2 : RuntimeLibcallImpl<SINCOS_NXV4F32>;
  def Sleef_sincospidx_u10rvvm2 : RuntimeLibcallImpl<SINCOSPI_NXV2F64>;
  def Sleef_sincospifx_u10rvvm2 : RuntimeLibcallImpl<SINCOSPI_NXV4F32>;
  def Sleef_sindx_u10rvvm2 : RuntimeLibcallImpl<SIN_NXV2F64>;
````
- **L4193 EN**: Declares TableGen def `Sleef_fmaxfx_rvvm2`.
  **L4193 CN**: 声明 TableGen def `Sleef_fmaxfx_rvvm2`。
- **L4194 EN**: Declares TableGen def `Sleef_fmindx_u10rvvm2`.
  **L4194 CN**: 声明 TableGen def `Sleef_fmindx_u10rvvm2`。
- **L4195 EN**: Declares TableGen def `Sleef_fminfx_u10rvvm2`.
  **L4195 CN**: 声明 TableGen def `Sleef_fminfx_u10rvvm2`。
- **L4196 EN**: Declares TableGen def `Sleef_fmoddx_rvvm2`.
  **L4196 CN**: 声明 TableGen def `Sleef_fmoddx_rvvm2`。
- **L4197 EN**: Declares TableGen def `Sleef_fmodfx_rvvm2`.
  **L4197 CN**: 声明 TableGen def `Sleef_fmodfx_rvvm2`。
- **L4198 EN**: Declares TableGen def `Sleef_hypotdx_u05rvvm2`.
  **L4198 CN**: 声明 TableGen def `Sleef_hypotdx_u05rvvm2`。
- **L4199 EN**: Declares TableGen def `Sleef_hypotfx_u05rvvm2`.
  **L4199 CN**: 声明 TableGen def `Sleef_hypotfx_u05rvvm2`。
- **L4200 EN**: Declares TableGen def `Sleef_ilogbdx_rvvm2`.
  **L4200 CN**: 声明 TableGen def `Sleef_ilogbdx_rvvm2`。
- **L4201 EN**: Declares TableGen def `Sleef_ilogbfx_rvvm2`.
  **L4201 CN**: 声明 TableGen def `Sleef_ilogbfx_rvvm2`。
- **L4202 EN**: Declares TableGen def `Sleef_ldexpdx_rvvm2`.
  **L4202 CN**: 声明 TableGen def `Sleef_ldexpdx_rvvm2`。
- **L4203 EN**: Declares TableGen def `Sleef_ldexpfx_rvvm2`.
  **L4203 CN**: 声明 TableGen def `Sleef_ldexpfx_rvvm2`。
- **L4204 EN**: Declares TableGen def `Sleef_lgammadx_u10rvvm2`.
  **L4204 CN**: 声明 TableGen def `Sleef_lgammadx_u10rvvm2`。
- **L4205 EN**: Declares TableGen def `Sleef_lgammafx_u10rvvm2`.
  **L4205 CN**: 声明 TableGen def `Sleef_lgammafx_u10rvvm2`。
- **L4206 EN**: Declares TableGen def `Sleef_log10dx_u10rvvm2`.
  **L4206 CN**: 声明 TableGen def `Sleef_log10dx_u10rvvm2`。
- **L4207 EN**: Declares TableGen def `Sleef_log10fx_u10rvvm2`.
  **L4207 CN**: 声明 TableGen def `Sleef_log10fx_u10rvvm2`。
- **L4208 EN**: Declares TableGen def `Sleef_log1pdx_u10rvvm2`.
  **L4208 CN**: 声明 TableGen def `Sleef_log1pdx_u10rvvm2`。
- **L4209 EN**: Declares TableGen def `Sleef_log1pfx_u10rvvm2`.
  **L4209 CN**: 声明 TableGen def `Sleef_log1pfx_u10rvvm2`。
- **L4210 EN**: Declares TableGen def `Sleef_log2dx_u10rvvm2`.
  **L4210 CN**: 声明 TableGen def `Sleef_log2dx_u10rvvm2`。
- **L4211 EN**: Declares TableGen def `Sleef_log2fx_u10rvvm2`.
  **L4211 CN**: 声明 TableGen def `Sleef_log2fx_u10rvvm2`。
- **L4212 EN**: Declares TableGen def `Sleef_logdx_u10rvvm2`.
  **L4212 CN**: 声明 TableGen def `Sleef_logdx_u10rvvm2`。
- **L4213 EN**: Declares TableGen def `Sleef_logfx_u10rvvm2`.
  **L4213 CN**: 声明 TableGen def `Sleef_logfx_u10rvvm2`。
- **L4214 EN**: Declares TableGen def `Sleef_modfdx_rvvm2`.
  **L4214 CN**: 声明 TableGen def `Sleef_modfdx_rvvm2`。
- **L4215 EN**: Declares TableGen def `Sleef_modffx_rvvm2`.
  **L4215 CN**: 声明 TableGen def `Sleef_modffx_rvvm2`。
- **L4216 EN**: Declares TableGen def `Sleef_nextafterdx_rvvm2`.
  **L4216 CN**: 声明 TableGen def `Sleef_nextafterdx_rvvm2`。
- **L4217 EN**: Declares TableGen def `Sleef_nextafterfx_rvvm2`.
  **L4217 CN**: 声明 TableGen def `Sleef_nextafterfx_rvvm2`。
- **L4218 EN**: Declares TableGen def `Sleef_powdx_u10rvvm2`.
  **L4218 CN**: 声明 TableGen def `Sleef_powdx_u10rvvm2`。
- **L4219 EN**: Declares TableGen def `Sleef_powfx_u10rvvm2`.
  **L4219 CN**: 声明 TableGen def `Sleef_powfx_u10rvvm2`。
- **L4220 EN**: Declares TableGen def `Sleef_sincosdx_u10rvvm2`.
  **L4220 CN**: 声明 TableGen def `Sleef_sincosdx_u10rvvm2`。
- **L4221 EN**: Declares TableGen def `Sleef_sincosfx_u10rvvm2`.
  **L4221 CN**: 声明 TableGen def `Sleef_sincosfx_u10rvvm2`。
- **L4222 EN**: Declares TableGen def `Sleef_sincospidx_u10rvvm2`.
  **L4222 CN**: 声明 TableGen def `Sleef_sincospidx_u10rvvm2`。
- **L4223 EN**: Declares TableGen def `Sleef_sincospifx_u10rvvm2`.
  **L4223 CN**: 声明 TableGen def `Sleef_sincospifx_u10rvvm2`。
- **L4224 EN**: Declares TableGen def `Sleef_sindx_u10rvvm2`.
  **L4224 CN**: 声明 TableGen def `Sleef_sindx_u10rvvm2`。

### Lines 4225-4256

````tablegen
  def Sleef_sinfx_u10rvvm2 : RuntimeLibcallImpl<SIN_NXV4F32>;
  def Sleef_sinhdx_u10rvvm2 : RuntimeLibcallImpl<SINH_NXV2F64>;
  def Sleef_sinhfx_u10rvvm2 : RuntimeLibcallImpl<SINH_NXV4F32>;
  def Sleef_sinpidx_u05rvvm2 : RuntimeLibcallImpl<SINPI_NXV2F64>;
  def Sleef_sinpifx_u05rvvm2 : RuntimeLibcallImpl<SINPI_NXV4F32>;
  def Sleef_sqrtdx_u05rvvm2 : RuntimeLibcallImpl<SQRT_NXV2F64>;
  def Sleef_sqrtfx_u05rvvm2 : RuntimeLibcallImpl<SQRT_NXV4F32>;
  def Sleef_tandx_u10rvvm2 : RuntimeLibcallImpl<TAN_NXV2F64>;
  def Sleef_tanfx_u10rvvm2 : RuntimeLibcallImpl<TAN_NXV4F32>;
  def Sleef_tanhdx_u10rvvm2 : RuntimeLibcallImpl<TANH_NXV2F64>;
  def Sleef_tanhfx_u10rvvm2 : RuntimeLibcallImpl<TANH_NXV4F32>;
  def Sleef_tgammadx_u10rvvm2 : RuntimeLibcallImpl<TGAMMA_NXV2F64>;
  def Sleef_tgammafx_u10rvvm2 : RuntimeLibcallImpl<TGAMMA_NXV4F32>;
}

//===----------------------------------------------------------------------===//
// Arm Performance Libraries (ARMPL) functions
//===----------------------------------------------------------------------===//

defset list<RuntimeLibcallImpl> ARMPL_VECFUNCS = {
  def armpl_svacos_f32_x : RuntimeLibcallImpl<ACOS_NXV4F32>;
  def armpl_svacos_f64_x : RuntimeLibcallImpl<ACOS_NXV2F64>;
  def armpl_svacosh_f32_x : RuntimeLibcallImpl<ACOSH_NXV4F32>;
  def armpl_svacosh_f64_x : RuntimeLibcallImpl<ACOSH_NXV2F64>;
  def armpl_svasin_f32_x : RuntimeLibcallImpl<ASIN_NXV4F32>;
  def armpl_svasin_f64_x : RuntimeLibcallImpl<ASIN_NXV2F64>;
  def armpl_svasinh_f32_x : RuntimeLibcallImpl<ASINH_NXV4F32>;
  def armpl_svasinh_f64_x : RuntimeLibcallImpl<ASINH_NXV2F64>;
  def armpl_svatan2_f32_x : RuntimeLibcallImpl<ATAN2_NXV4F32>;
  def armpl_svatan2_f64_x : RuntimeLibcallImpl<ATAN2_NXV2F64>;
  def armpl_svatan_f32_x : RuntimeLibcallImpl<ATAN_NXV4F32>;
  def armpl_svatan_f64_x : RuntimeLibcallImpl<ATAN_NXV2F64>;
````
- **L4225 EN**: Declares TableGen def `Sleef_sinfx_u10rvvm2`.
  **L4225 CN**: 声明 TableGen def `Sleef_sinfx_u10rvvm2`。
- **L4226 EN**: Declares TableGen def `Sleef_sinhdx_u10rvvm2`.
  **L4226 CN**: 声明 TableGen def `Sleef_sinhdx_u10rvvm2`。
- **L4227 EN**: Declares TableGen def `Sleef_sinhfx_u10rvvm2`.
  **L4227 CN**: 声明 TableGen def `Sleef_sinhfx_u10rvvm2`。
- **L4228 EN**: Declares TableGen def `Sleef_sinpidx_u05rvvm2`.
  **L4228 CN**: 声明 TableGen def `Sleef_sinpidx_u05rvvm2`。
- **L4229 EN**: Declares TableGen def `Sleef_sinpifx_u05rvvm2`.
  **L4229 CN**: 声明 TableGen def `Sleef_sinpifx_u05rvvm2`。
- **L4230 EN**: Declares TableGen def `Sleef_sqrtdx_u05rvvm2`.
  **L4230 CN**: 声明 TableGen def `Sleef_sqrtdx_u05rvvm2`。
- **L4231 EN**: Declares TableGen def `Sleef_sqrtfx_u05rvvm2`.
  **L4231 CN**: 声明 TableGen def `Sleef_sqrtfx_u05rvvm2`。
- **L4232 EN**: Declares TableGen def `Sleef_tandx_u10rvvm2`.
  **L4232 CN**: 声明 TableGen def `Sleef_tandx_u10rvvm2`。
- **L4233 EN**: Declares TableGen def `Sleef_tanfx_u10rvvm2`.
  **L4233 CN**: 声明 TableGen def `Sleef_tanfx_u10rvvm2`。
- **L4234 EN**: Declares TableGen def `Sleef_tanhdx_u10rvvm2`.
  **L4234 CN**: 声明 TableGen def `Sleef_tanhdx_u10rvvm2`。
- **L4235 EN**: Declares TableGen def `Sleef_tanhfx_u10rvvm2`.
  **L4235 CN**: 声明 TableGen def `Sleef_tanhfx_u10rvvm2`。
- **L4236 EN**: Declares TableGen def `Sleef_tgammadx_u10rvvm2`.
  **L4236 CN**: 声明 TableGen def `Sleef_tgammadx_u10rvvm2`。
- **L4237 EN**: Declares TableGen def `Sleef_tgammafx_u10rvvm2`.
  **L4237 CN**: 声明 TableGen def `Sleef_tgammafx_u10rvvm2`。
- **L4238 EN**: Closes the current lexical scope or compound statement.
  **L4238 CN**: 结束当前词法作用域或复合语句块。
- **L4239 EN**: Blank line separating nearby declarations or logic blocks.
  **L4239 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L4240 EN**: Banner comment marking a file or section boundary.
  **L4240 CN**: 横幅注释，用于标记文件或章节边界。
- **L4241 EN**: Comment explains nearby logic, invariants, or intent: `Arm Performance Libraries (ARMPL) functions`.
  **L4241 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Arm Performance Libraries (ARMPL) functions`。
- **L4242 EN**: Banner comment marking a file or section boundary.
  **L4242 CN**: 横幅注释，用于标记文件或章节边界。
- **L4243 EN**: Blank line separating nearby declarations or logic blocks.
  **L4243 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L4244 EN**: Continues the surrounding expression or declaration: `defset list<RuntimeLibcallImpl> ARMPL_VECFUNCS = {`.
  **L4244 CN**: 继续构造周围的表达式或声明：`defset list<RuntimeLibcallImpl> ARMPL_VECFUNCS = {`。
- **L4245 EN**: Declares TableGen def `armpl_svacos_f32_x`.
  **L4245 CN**: 声明 TableGen def `armpl_svacos_f32_x`。
- **L4246 EN**: Declares TableGen def `armpl_svacos_f64_x`.
  **L4246 CN**: 声明 TableGen def `armpl_svacos_f64_x`。
- **L4247 EN**: Declares TableGen def `armpl_svacosh_f32_x`.
  **L4247 CN**: 声明 TableGen def `armpl_svacosh_f32_x`。
- **L4248 EN**: Declares TableGen def `armpl_svacosh_f64_x`.
  **L4248 CN**: 声明 TableGen def `armpl_svacosh_f64_x`。
- **L4249 EN**: Declares TableGen def `armpl_svasin_f32_x`.
  **L4249 CN**: 声明 TableGen def `armpl_svasin_f32_x`。
- **L4250 EN**: Declares TableGen def `armpl_svasin_f64_x`.
  **L4250 CN**: 声明 TableGen def `armpl_svasin_f64_x`。
- **L4251 EN**: Declares TableGen def `armpl_svasinh_f32_x`.
  **L4251 CN**: 声明 TableGen def `armpl_svasinh_f32_x`。
- **L4252 EN**: Declares TableGen def `armpl_svasinh_f64_x`.
  **L4252 CN**: 声明 TableGen def `armpl_svasinh_f64_x`。
- **L4253 EN**: Declares TableGen def `armpl_svatan2_f32_x`.
  **L4253 CN**: 声明 TableGen def `armpl_svatan2_f32_x`。
- **L4254 EN**: Declares TableGen def `armpl_svatan2_f64_x`.
  **L4254 CN**: 声明 TableGen def `armpl_svatan2_f64_x`。
- **L4255 EN**: Declares TableGen def `armpl_svatan_f32_x`.
  **L4255 CN**: 声明 TableGen def `armpl_svatan_f32_x`。
- **L4256 EN**: Declares TableGen def `armpl_svatan_f64_x`.
  **L4256 CN**: 声明 TableGen def `armpl_svatan_f64_x`。

### Lines 4257-4288

````tablegen
  def armpl_svatanh_f32_x : RuntimeLibcallImpl<ATANH_NXV4F32>;
  def armpl_svatanh_f64_x : RuntimeLibcallImpl<ATANH_NXV2F64>;
  def armpl_svcbrt_f32_x : RuntimeLibcallImpl<CBRT_NXV4F32>;
  def armpl_svcbrt_f64_x : RuntimeLibcallImpl<CBRT_NXV2F64>;
  def armpl_svcopysign_f32_x : RuntimeLibcallImpl<COPYSIGN_NXV4F32>;
  def armpl_svcopysign_f64_x : RuntimeLibcallImpl<COPYSIGN_NXV2F64>;
  def armpl_svcos_f32_x : RuntimeLibcallImpl<COS_NXV4F32>;
  def armpl_svcos_f64_x : RuntimeLibcallImpl<COS_NXV2F64>;
  def armpl_svcosh_f32_x : RuntimeLibcallImpl<COSH_NXV4F32>;
  def armpl_svcosh_f64_x : RuntimeLibcallImpl<COSH_NXV2F64>;
  def armpl_svcospi_f32_x : RuntimeLibcallImpl<COSPI_NXV4F32>;
  def armpl_svcospi_f64_x : RuntimeLibcallImpl<COSPI_NXV2F64>;
  def armpl_sverf_f32_x : RuntimeLibcallImpl<ERF_NXV4F32>;
  def armpl_sverf_f64_x : RuntimeLibcallImpl<ERF_NXV2F64>;
  def armpl_sverfc_f32_x : RuntimeLibcallImpl<ERFC_NXV4F32>;
  def armpl_sverfc_f64_x : RuntimeLibcallImpl<ERFC_NXV2F64>;
  def armpl_svexp10_f32_x : RuntimeLibcallImpl<EXP10_NXV4F32>;
  def armpl_svexp10_f64_x : RuntimeLibcallImpl<EXP10_NXV2F64>;
  def armpl_svexp2_f32_x : RuntimeLibcallImpl<EXP2_NXV4F32>;
  def armpl_svexp2_f64_x : RuntimeLibcallImpl<EXP2_NXV2F64>;
  def armpl_svexp_f32_x : RuntimeLibcallImpl<EXP_NXV4F32>;
  def armpl_svexp_f64_x : RuntimeLibcallImpl<EXP_NXV2F64>;
  def armpl_svexpm1_f32_x : RuntimeLibcallImpl<EXPM1_NXV4F32>;
  def armpl_svexpm1_f64_x : RuntimeLibcallImpl<EXPM1_NXV2F64>;
  def armpl_svfdim_f32_x : RuntimeLibcallImpl<FDIM_NXV4F32>;
  def armpl_svfdim_f64_x : RuntimeLibcallImpl<FDIM_NXV2F64>;
  def armpl_svfma_f32_x : RuntimeLibcallImpl<FMA_NXV4F32>;
  def armpl_svfma_f64_x : RuntimeLibcallImpl<FMA_NXV2F64>;
  def armpl_svfmax_f32_x : RuntimeLibcallImpl<FMAX_NXV4F32>;
  def armpl_svfmax_f64_x : RuntimeLibcallImpl<FMAX_NXV2F64>;
  def armpl_svfmin_f32_x : RuntimeLibcallImpl<FMIN_NXV4F32>;
  def armpl_svfmin_f64_x : RuntimeLibcallImpl<FMIN_NXV2F64>;
````
- **L4257 EN**: Declares TableGen def `armpl_svatanh_f32_x`.
  **L4257 CN**: 声明 TableGen def `armpl_svatanh_f32_x`。
- **L4258 EN**: Declares TableGen def `armpl_svatanh_f64_x`.
  **L4258 CN**: 声明 TableGen def `armpl_svatanh_f64_x`。
- **L4259 EN**: Declares TableGen def `armpl_svcbrt_f32_x`.
  **L4259 CN**: 声明 TableGen def `armpl_svcbrt_f32_x`。
- **L4260 EN**: Declares TableGen def `armpl_svcbrt_f64_x`.
  **L4260 CN**: 声明 TableGen def `armpl_svcbrt_f64_x`。
- **L4261 EN**: Declares TableGen def `armpl_svcopysign_f32_x`.
  **L4261 CN**: 声明 TableGen def `armpl_svcopysign_f32_x`。
- **L4262 EN**: Declares TableGen def `armpl_svcopysign_f64_x`.
  **L4262 CN**: 声明 TableGen def `armpl_svcopysign_f64_x`。
- **L4263 EN**: Declares TableGen def `armpl_svcos_f32_x`.
  **L4263 CN**: 声明 TableGen def `armpl_svcos_f32_x`。
- **L4264 EN**: Declares TableGen def `armpl_svcos_f64_x`.
  **L4264 CN**: 声明 TableGen def `armpl_svcos_f64_x`。
- **L4265 EN**: Declares TableGen def `armpl_svcosh_f32_x`.
  **L4265 CN**: 声明 TableGen def `armpl_svcosh_f32_x`。
- **L4266 EN**: Declares TableGen def `armpl_svcosh_f64_x`.
  **L4266 CN**: 声明 TableGen def `armpl_svcosh_f64_x`。
- **L4267 EN**: Declares TableGen def `armpl_svcospi_f32_x`.
  **L4267 CN**: 声明 TableGen def `armpl_svcospi_f32_x`。
- **L4268 EN**: Declares TableGen def `armpl_svcospi_f64_x`.
  **L4268 CN**: 声明 TableGen def `armpl_svcospi_f64_x`。
- **L4269 EN**: Declares TableGen def `armpl_sverf_f32_x`.
  **L4269 CN**: 声明 TableGen def `armpl_sverf_f32_x`。
- **L4270 EN**: Declares TableGen def `armpl_sverf_f64_x`.
  **L4270 CN**: 声明 TableGen def `armpl_sverf_f64_x`。
- **L4271 EN**: Declares TableGen def `armpl_sverfc_f32_x`.
  **L4271 CN**: 声明 TableGen def `armpl_sverfc_f32_x`。
- **L4272 EN**: Declares TableGen def `armpl_sverfc_f64_x`.
  **L4272 CN**: 声明 TableGen def `armpl_sverfc_f64_x`。
- **L4273 EN**: Declares TableGen def `armpl_svexp10_f32_x`.
  **L4273 CN**: 声明 TableGen def `armpl_svexp10_f32_x`。
- **L4274 EN**: Declares TableGen def `armpl_svexp10_f64_x`.
  **L4274 CN**: 声明 TableGen def `armpl_svexp10_f64_x`。
- **L4275 EN**: Declares TableGen def `armpl_svexp2_f32_x`.
  **L4275 CN**: 声明 TableGen def `armpl_svexp2_f32_x`。
- **L4276 EN**: Declares TableGen def `armpl_svexp2_f64_x`.
  **L4276 CN**: 声明 TableGen def `armpl_svexp2_f64_x`。
- **L4277 EN**: Declares TableGen def `armpl_svexp_f32_x`.
  **L4277 CN**: 声明 TableGen def `armpl_svexp_f32_x`。
- **L4278 EN**: Declares TableGen def `armpl_svexp_f64_x`.
  **L4278 CN**: 声明 TableGen def `armpl_svexp_f64_x`。
- **L4279 EN**: Declares TableGen def `armpl_svexpm1_f32_x`.
  **L4279 CN**: 声明 TableGen def `armpl_svexpm1_f32_x`。
- **L4280 EN**: Declares TableGen def `armpl_svexpm1_f64_x`.
  **L4280 CN**: 声明 TableGen def `armpl_svexpm1_f64_x`。
- **L4281 EN**: Declares TableGen def `armpl_svfdim_f32_x`.
  **L4281 CN**: 声明 TableGen def `armpl_svfdim_f32_x`。
- **L4282 EN**: Declares TableGen def `armpl_svfdim_f64_x`.
  **L4282 CN**: 声明 TableGen def `armpl_svfdim_f64_x`。
- **L4283 EN**: Declares TableGen def `armpl_svfma_f32_x`.
  **L4283 CN**: 声明 TableGen def `armpl_svfma_f32_x`。
- **L4284 EN**: Declares TableGen def `armpl_svfma_f64_x`.
  **L4284 CN**: 声明 TableGen def `armpl_svfma_f64_x`。
- **L4285 EN**: Declares TableGen def `armpl_svfmax_f32_x`.
  **L4285 CN**: 声明 TableGen def `armpl_svfmax_f32_x`。
- **L4286 EN**: Declares TableGen def `armpl_svfmax_f64_x`.
  **L4286 CN**: 声明 TableGen def `armpl_svfmax_f64_x`。
- **L4287 EN**: Declares TableGen def `armpl_svfmin_f32_x`.
  **L4287 CN**: 声明 TableGen def `armpl_svfmin_f32_x`。
- **L4288 EN**: Declares TableGen def `armpl_svfmin_f64_x`.
  **L4288 CN**: 声明 TableGen def `armpl_svfmin_f64_x`。

### Lines 4289-4320

````tablegen
  def armpl_svfmod_f32_x : RuntimeLibcallImpl<REM_NXV4F32>;
  def armpl_svfmod_f64_x : RuntimeLibcallImpl<REM_NXV2F64>;
  def armpl_svhypot_f32_x : RuntimeLibcallImpl<HYPOT_NXV4F32>;
  def armpl_svhypot_f64_x : RuntimeLibcallImpl<HYPOT_NXV2F64>;
  def armpl_svilogb_f32_x : RuntimeLibcallImpl<ILOGB_NXV4F32>;
  def armpl_svilogb_f64_x : RuntimeLibcallImpl<ILOGB_NXV2F64>;
  def armpl_svldexp_f32_x : RuntimeLibcallImpl<LDEXP_NXV4F32>;
  def armpl_svldexp_f64_x : RuntimeLibcallImpl<LDEXP_NXV2F64>;
  def armpl_svlgamma_f32_x : RuntimeLibcallImpl<LGAMMA_NXV4F32>;
  def armpl_svlgamma_f64_x : RuntimeLibcallImpl<LGAMMA_NXV2F64>;
  def armpl_svlog10_f32_x : RuntimeLibcallImpl<LOG10_NXV4F32>;
  def armpl_svlog10_f64_x : RuntimeLibcallImpl<LOG10_NXV2F64>;
  def armpl_svlog1p_f32_x : RuntimeLibcallImpl<LOG1P_NXV4F32>;
  def armpl_svlog1p_f64_x : RuntimeLibcallImpl<LOG1P_NXV2F64>;
  def armpl_svlog2_f32_x : RuntimeLibcallImpl<LOG2_NXV4F32>;
  def armpl_svlog2_f64_x : RuntimeLibcallImpl<LOG2_NXV2F64>;
  def armpl_svlog_f32_x : RuntimeLibcallImpl<LOG_NXV4F32>;
  def armpl_svlog_f64_x : RuntimeLibcallImpl<LOG_NXV2F64>;
  def armpl_svmodf_f32_x : RuntimeLibcallImpl<MODF_NXV4F32>;
  def armpl_svmodf_f64_x : RuntimeLibcallImpl<MODF_NXV2F64>;
  def armpl_svnextafter_f32_x : RuntimeLibcallImpl<NEXTAFTER_NXV4F32>;
  def armpl_svnextafter_f64_x : RuntimeLibcallImpl<NEXTAFTER_NXV2F64>;
  def armpl_svpow_f32_x : RuntimeLibcallImpl<POW_NXV4F32>;
  def armpl_svpow_f64_x : RuntimeLibcallImpl<POW_NXV2F64>;
  def armpl_svsin_f32_x : RuntimeLibcallImpl<SIN_NXV4F32>;
  def armpl_svsin_f64_x : RuntimeLibcallImpl<SIN_NXV2F64>;
  def armpl_svsincos_f32_x : RuntimeLibcallImpl<SINCOS_NXV4F32>;
  def armpl_svsincos_f64_x : RuntimeLibcallImpl<SINCOS_NXV2F64>;
  def armpl_svsincospi_f32_x : RuntimeLibcallImpl<SINCOSPI_NXV4F32>;
  def armpl_svsincospi_f64_x : RuntimeLibcallImpl<SINCOSPI_NXV2F64>;
  def armpl_svsinh_f32_x : RuntimeLibcallImpl<SINH_NXV4F32>;
  def armpl_svsinh_f64_x : RuntimeLibcallImpl<SINH_NXV2F64>;
````
- **L4289 EN**: Declares TableGen def `armpl_svfmod_f32_x`.
  **L4289 CN**: 声明 TableGen def `armpl_svfmod_f32_x`。
- **L4290 EN**: Declares TableGen def `armpl_svfmod_f64_x`.
  **L4290 CN**: 声明 TableGen def `armpl_svfmod_f64_x`。
- **L4291 EN**: Declares TableGen def `armpl_svhypot_f32_x`.
  **L4291 CN**: 声明 TableGen def `armpl_svhypot_f32_x`。
- **L4292 EN**: Declares TableGen def `armpl_svhypot_f64_x`.
  **L4292 CN**: 声明 TableGen def `armpl_svhypot_f64_x`。
- **L4293 EN**: Declares TableGen def `armpl_svilogb_f32_x`.
  **L4293 CN**: 声明 TableGen def `armpl_svilogb_f32_x`。
- **L4294 EN**: Declares TableGen def `armpl_svilogb_f64_x`.
  **L4294 CN**: 声明 TableGen def `armpl_svilogb_f64_x`。
- **L4295 EN**: Declares TableGen def `armpl_svldexp_f32_x`.
  **L4295 CN**: 声明 TableGen def `armpl_svldexp_f32_x`。
- **L4296 EN**: Declares TableGen def `armpl_svldexp_f64_x`.
  **L4296 CN**: 声明 TableGen def `armpl_svldexp_f64_x`。
- **L4297 EN**: Declares TableGen def `armpl_svlgamma_f32_x`.
  **L4297 CN**: 声明 TableGen def `armpl_svlgamma_f32_x`。
- **L4298 EN**: Declares TableGen def `armpl_svlgamma_f64_x`.
  **L4298 CN**: 声明 TableGen def `armpl_svlgamma_f64_x`。
- **L4299 EN**: Declares TableGen def `armpl_svlog10_f32_x`.
  **L4299 CN**: 声明 TableGen def `armpl_svlog10_f32_x`。
- **L4300 EN**: Declares TableGen def `armpl_svlog10_f64_x`.
  **L4300 CN**: 声明 TableGen def `armpl_svlog10_f64_x`。
- **L4301 EN**: Declares TableGen def `armpl_svlog1p_f32_x`.
  **L4301 CN**: 声明 TableGen def `armpl_svlog1p_f32_x`。
- **L4302 EN**: Declares TableGen def `armpl_svlog1p_f64_x`.
  **L4302 CN**: 声明 TableGen def `armpl_svlog1p_f64_x`。
- **L4303 EN**: Declares TableGen def `armpl_svlog2_f32_x`.
  **L4303 CN**: 声明 TableGen def `armpl_svlog2_f32_x`。
- **L4304 EN**: Declares TableGen def `armpl_svlog2_f64_x`.
  **L4304 CN**: 声明 TableGen def `armpl_svlog2_f64_x`。
- **L4305 EN**: Declares TableGen def `armpl_svlog_f32_x`.
  **L4305 CN**: 声明 TableGen def `armpl_svlog_f32_x`。
- **L4306 EN**: Declares TableGen def `armpl_svlog_f64_x`.
  **L4306 CN**: 声明 TableGen def `armpl_svlog_f64_x`。
- **L4307 EN**: Declares TableGen def `armpl_svmodf_f32_x`.
  **L4307 CN**: 声明 TableGen def `armpl_svmodf_f32_x`。
- **L4308 EN**: Declares TableGen def `armpl_svmodf_f64_x`.
  **L4308 CN**: 声明 TableGen def `armpl_svmodf_f64_x`。
- **L4309 EN**: Declares TableGen def `armpl_svnextafter_f32_x`.
  **L4309 CN**: 声明 TableGen def `armpl_svnextafter_f32_x`。
- **L4310 EN**: Declares TableGen def `armpl_svnextafter_f64_x`.
  **L4310 CN**: 声明 TableGen def `armpl_svnextafter_f64_x`。
- **L4311 EN**: Declares TableGen def `armpl_svpow_f32_x`.
  **L4311 CN**: 声明 TableGen def `armpl_svpow_f32_x`。
- **L4312 EN**: Declares TableGen def `armpl_svpow_f64_x`.
  **L4312 CN**: 声明 TableGen def `armpl_svpow_f64_x`。
- **L4313 EN**: Declares TableGen def `armpl_svsin_f32_x`.
  **L4313 CN**: 声明 TableGen def `armpl_svsin_f32_x`。
- **L4314 EN**: Declares TableGen def `armpl_svsin_f64_x`.
  **L4314 CN**: 声明 TableGen def `armpl_svsin_f64_x`。
- **L4315 EN**: Declares TableGen def `armpl_svsincos_f32_x`.
  **L4315 CN**: 声明 TableGen def `armpl_svsincos_f32_x`。
- **L4316 EN**: Declares TableGen def `armpl_svsincos_f64_x`.
  **L4316 CN**: 声明 TableGen def `armpl_svsincos_f64_x`。
- **L4317 EN**: Declares TableGen def `armpl_svsincospi_f32_x`.
  **L4317 CN**: 声明 TableGen def `armpl_svsincospi_f32_x`。
- **L4318 EN**: Declares TableGen def `armpl_svsincospi_f64_x`.
  **L4318 CN**: 声明 TableGen def `armpl_svsincospi_f64_x`。
- **L4319 EN**: Declares TableGen def `armpl_svsinh_f32_x`.
  **L4319 CN**: 声明 TableGen def `armpl_svsinh_f32_x`。
- **L4320 EN**: Declares TableGen def `armpl_svsinh_f64_x`.
  **L4320 CN**: 声明 TableGen def `armpl_svsinh_f64_x`。

### Lines 4321-4352

````tablegen
  def armpl_svsinpi_f32_x : RuntimeLibcallImpl<SINPI_NXV4F32>;
  def armpl_svsinpi_f64_x : RuntimeLibcallImpl<SINPI_NXV2F64>;
  def armpl_svsqrt_f32_x : RuntimeLibcallImpl<SQRT_NXV4F32>;
  def armpl_svsqrt_f64_x : RuntimeLibcallImpl<SQRT_NXV2F64>;
  def armpl_svtan_f32_x : RuntimeLibcallImpl<TAN_NXV4F32>;
  def armpl_svtan_f64_x : RuntimeLibcallImpl<TAN_NXV2F64>;
  def armpl_svtanh_f32_x : RuntimeLibcallImpl<TANH_NXV4F32>;
  def armpl_svtanh_f64_x : RuntimeLibcallImpl<TANH_NXV2F64>;
  def armpl_svtgamma_f32_x : RuntimeLibcallImpl<TGAMMA_NXV4F32>;
  def armpl_svtgamma_f64_x : RuntimeLibcallImpl<TGAMMA_NXV2F64>;
  def armpl_vacoshq_f32 : RuntimeLibcallImpl<ACOSH_V4F32>;
  def armpl_vacoshq_f64 : RuntimeLibcallImpl<ACOSH_V2F64>;
  def armpl_vacosq_f32 : RuntimeLibcallImpl<ACOS_V4F32>;
  def armpl_vacosq_f64 : RuntimeLibcallImpl<ACOS_V2F64>;
  def armpl_vasinhq_f32 : RuntimeLibcallImpl<ASINH_V4F32>;
  def armpl_vasinhq_f64 : RuntimeLibcallImpl<ASINH_V2F64>;
  def armpl_vasinq_f32 : RuntimeLibcallImpl<ASIN_V4F32>;
  def armpl_vasinq_f64 : RuntimeLibcallImpl<ASIN_V2F64>;
  def armpl_vatan2q_f32 : RuntimeLibcallImpl<ATAN2_V4F32>;
  def armpl_vatan2q_f64 : RuntimeLibcallImpl<ATAN2_V2F64>;
  def armpl_vatanhq_f32 : RuntimeLibcallImpl<ATANH_V4F32>;
  def armpl_vatanhq_f64 : RuntimeLibcallImpl<ATANH_V2F64>;
  def armpl_vatanq_f32 : RuntimeLibcallImpl<ATAN_V4F32>;
  def armpl_vatanq_f64 : RuntimeLibcallImpl<ATAN_V2F64>;
  def armpl_vcbrtq_f32 : RuntimeLibcallImpl<CBRT_V4F32>;
  def armpl_vcbrtq_f64 : RuntimeLibcallImpl<CBRT_V2F64>;
  def armpl_vcopysignq_f32 : RuntimeLibcallImpl<COPYSIGN_V4F32>;
  def armpl_vcopysignq_f64 : RuntimeLibcallImpl<COPYSIGN_V2F64>;
  def armpl_vcoshq_f32 : RuntimeLibcallImpl<COSH_V4F32>;
  def armpl_vcoshq_f64 : RuntimeLibcallImpl<COSH_V2F64>;
  def armpl_vcospiq_f32 : RuntimeLibcallImpl<COSPI_V4F32>;
  def armpl_vcospiq_f64 : RuntimeLibcallImpl<COSPI_V2F64>;
````
- **L4321 EN**: Declares TableGen def `armpl_svsinpi_f32_x`.
  **L4321 CN**: 声明 TableGen def `armpl_svsinpi_f32_x`。
- **L4322 EN**: Declares TableGen def `armpl_svsinpi_f64_x`.
  **L4322 CN**: 声明 TableGen def `armpl_svsinpi_f64_x`。
- **L4323 EN**: Declares TableGen def `armpl_svsqrt_f32_x`.
  **L4323 CN**: 声明 TableGen def `armpl_svsqrt_f32_x`。
- **L4324 EN**: Declares TableGen def `armpl_svsqrt_f64_x`.
  **L4324 CN**: 声明 TableGen def `armpl_svsqrt_f64_x`。
- **L4325 EN**: Declares TableGen def `armpl_svtan_f32_x`.
  **L4325 CN**: 声明 TableGen def `armpl_svtan_f32_x`。
- **L4326 EN**: Declares TableGen def `armpl_svtan_f64_x`.
  **L4326 CN**: 声明 TableGen def `armpl_svtan_f64_x`。
- **L4327 EN**: Declares TableGen def `armpl_svtanh_f32_x`.
  **L4327 CN**: 声明 TableGen def `armpl_svtanh_f32_x`。
- **L4328 EN**: Declares TableGen def `armpl_svtanh_f64_x`.
  **L4328 CN**: 声明 TableGen def `armpl_svtanh_f64_x`。
- **L4329 EN**: Declares TableGen def `armpl_svtgamma_f32_x`.
  **L4329 CN**: 声明 TableGen def `armpl_svtgamma_f32_x`。
- **L4330 EN**: Declares TableGen def `armpl_svtgamma_f64_x`.
  **L4330 CN**: 声明 TableGen def `armpl_svtgamma_f64_x`。
- **L4331 EN**: Declares TableGen def `armpl_vacoshq_f32`.
  **L4331 CN**: 声明 TableGen def `armpl_vacoshq_f32`。
- **L4332 EN**: Declares TableGen def `armpl_vacoshq_f64`.
  **L4332 CN**: 声明 TableGen def `armpl_vacoshq_f64`。
- **L4333 EN**: Declares TableGen def `armpl_vacosq_f32`.
  **L4333 CN**: 声明 TableGen def `armpl_vacosq_f32`。
- **L4334 EN**: Declares TableGen def `armpl_vacosq_f64`.
  **L4334 CN**: 声明 TableGen def `armpl_vacosq_f64`。
- **L4335 EN**: Declares TableGen def `armpl_vasinhq_f32`.
  **L4335 CN**: 声明 TableGen def `armpl_vasinhq_f32`。
- **L4336 EN**: Declares TableGen def `armpl_vasinhq_f64`.
  **L4336 CN**: 声明 TableGen def `armpl_vasinhq_f64`。
- **L4337 EN**: Declares TableGen def `armpl_vasinq_f32`.
  **L4337 CN**: 声明 TableGen def `armpl_vasinq_f32`。
- **L4338 EN**: Declares TableGen def `armpl_vasinq_f64`.
  **L4338 CN**: 声明 TableGen def `armpl_vasinq_f64`。
- **L4339 EN**: Declares TableGen def `armpl_vatan2q_f32`.
  **L4339 CN**: 声明 TableGen def `armpl_vatan2q_f32`。
- **L4340 EN**: Declares TableGen def `armpl_vatan2q_f64`.
  **L4340 CN**: 声明 TableGen def `armpl_vatan2q_f64`。
- **L4341 EN**: Declares TableGen def `armpl_vatanhq_f32`.
  **L4341 CN**: 声明 TableGen def `armpl_vatanhq_f32`。
- **L4342 EN**: Declares TableGen def `armpl_vatanhq_f64`.
  **L4342 CN**: 声明 TableGen def `armpl_vatanhq_f64`。
- **L4343 EN**: Declares TableGen def `armpl_vatanq_f32`.
  **L4343 CN**: 声明 TableGen def `armpl_vatanq_f32`。
- **L4344 EN**: Declares TableGen def `armpl_vatanq_f64`.
  **L4344 CN**: 声明 TableGen def `armpl_vatanq_f64`。
- **L4345 EN**: Declares TableGen def `armpl_vcbrtq_f32`.
  **L4345 CN**: 声明 TableGen def `armpl_vcbrtq_f32`。
- **L4346 EN**: Declares TableGen def `armpl_vcbrtq_f64`.
  **L4346 CN**: 声明 TableGen def `armpl_vcbrtq_f64`。
- **L4347 EN**: Declares TableGen def `armpl_vcopysignq_f32`.
  **L4347 CN**: 声明 TableGen def `armpl_vcopysignq_f32`。
- **L4348 EN**: Declares TableGen def `armpl_vcopysignq_f64`.
  **L4348 CN**: 声明 TableGen def `armpl_vcopysignq_f64`。
- **L4349 EN**: Declares TableGen def `armpl_vcoshq_f32`.
  **L4349 CN**: 声明 TableGen def `armpl_vcoshq_f32`。
- **L4350 EN**: Declares TableGen def `armpl_vcoshq_f64`.
  **L4350 CN**: 声明 TableGen def `armpl_vcoshq_f64`。
- **L4351 EN**: Declares TableGen def `armpl_vcospiq_f32`.
  **L4351 CN**: 声明 TableGen def `armpl_vcospiq_f32`。
- **L4352 EN**: Declares TableGen def `armpl_vcospiq_f64`.
  **L4352 CN**: 声明 TableGen def `armpl_vcospiq_f64`。

### Lines 4353-4384

````tablegen
  def armpl_vcosq_f32 : RuntimeLibcallImpl<COS_V4F32>;
  def armpl_vcosq_f64 : RuntimeLibcallImpl<COS_V2F64>;
  def armpl_verfcq_f32 : RuntimeLibcallImpl<ERFC_V4F32>;
  def armpl_verfcq_f64 : RuntimeLibcallImpl<ERFC_V2F64>;
  def armpl_verfq_f32 : RuntimeLibcallImpl<ERF_V4F32>;
  def armpl_verfq_f64 : RuntimeLibcallImpl<ERF_V2F64>;
  def armpl_vexp10q_f32 : RuntimeLibcallImpl<EXP10_V4F32>;
  def armpl_vexp10q_f64 : RuntimeLibcallImpl<EXP10_V2F64>;
  def armpl_vexp2q_f32 : RuntimeLibcallImpl<EXP2_V4F32>;
  def armpl_vexp2q_f64 : RuntimeLibcallImpl<EXP2_V2F64>;
  def armpl_vexpm1q_f32 : RuntimeLibcallImpl<EXPM1_V4F32>;
  def armpl_vexpm1q_f64 : RuntimeLibcallImpl<EXPM1_V2F64>;
  def armpl_vexpq_f32 : RuntimeLibcallImpl<EXP_V4F32>;
  def armpl_vexpq_f64 : RuntimeLibcallImpl<EXP_V2F64>;
  def armpl_vfdimq_f32 : RuntimeLibcallImpl<FDIM_V4F32>;
  def armpl_vfdimq_f64 : RuntimeLibcallImpl<FDIM_V2F64>;
  def armpl_vfmaq_f32 : RuntimeLibcallImpl<FMA_V4F32>;
  def armpl_vfmaq_f64 : RuntimeLibcallImpl<FMA_V2F64>;
  def armpl_vfmaxq_f32 : RuntimeLibcallImpl<FMAX_V4F32>;
  def armpl_vfmaxq_f64 : RuntimeLibcallImpl<FMAX_V2F64>;
  def armpl_vfminq_f32 : RuntimeLibcallImpl<FMIN_V4F32>;
  def armpl_vfminq_f64 : RuntimeLibcallImpl<FMIN_V2F64>;
  def armpl_vfmodq_f32 : RuntimeLibcallImpl<REM_V4F32>;
  def armpl_vfmodq_f64 : RuntimeLibcallImpl<REM_V2F64>;
  def armpl_vhypotq_f32 : RuntimeLibcallImpl<HYPOT_V4F32>;
  def armpl_vhypotq_f64 : RuntimeLibcallImpl<HYPOT_V2F64>;
  def armpl_vilogbq_f32 : RuntimeLibcallImpl<ILOGB_V4F32>;
  def armpl_vilogbq_f64 : RuntimeLibcallImpl<ILOGB_V2F64>;
  def armpl_vldexpq_f32 : RuntimeLibcallImpl<LDEXP_V4F32>;
  def armpl_vldexpq_f64 : RuntimeLibcallImpl<LDEXP_V2F64>;
  def armpl_vlgammaq_f32 : RuntimeLibcallImpl<LGAMMA_V4F32>;
  def armpl_vlgammaq_f64 : RuntimeLibcallImpl<LGAMMA_V2F64>;
````
- **L4353 EN**: Declares TableGen def `armpl_vcosq_f32`.
  **L4353 CN**: 声明 TableGen def `armpl_vcosq_f32`。
- **L4354 EN**: Declares TableGen def `armpl_vcosq_f64`.
  **L4354 CN**: 声明 TableGen def `armpl_vcosq_f64`。
- **L4355 EN**: Declares TableGen def `armpl_verfcq_f32`.
  **L4355 CN**: 声明 TableGen def `armpl_verfcq_f32`。
- **L4356 EN**: Declares TableGen def `armpl_verfcq_f64`.
  **L4356 CN**: 声明 TableGen def `armpl_verfcq_f64`。
- **L4357 EN**: Declares TableGen def `armpl_verfq_f32`.
  **L4357 CN**: 声明 TableGen def `armpl_verfq_f32`。
- **L4358 EN**: Declares TableGen def `armpl_verfq_f64`.
  **L4358 CN**: 声明 TableGen def `armpl_verfq_f64`。
- **L4359 EN**: Declares TableGen def `armpl_vexp10q_f32`.
  **L4359 CN**: 声明 TableGen def `armpl_vexp10q_f32`。
- **L4360 EN**: Declares TableGen def `armpl_vexp10q_f64`.
  **L4360 CN**: 声明 TableGen def `armpl_vexp10q_f64`。
- **L4361 EN**: Declares TableGen def `armpl_vexp2q_f32`.
  **L4361 CN**: 声明 TableGen def `armpl_vexp2q_f32`。
- **L4362 EN**: Declares TableGen def `armpl_vexp2q_f64`.
  **L4362 CN**: 声明 TableGen def `armpl_vexp2q_f64`。
- **L4363 EN**: Declares TableGen def `armpl_vexpm1q_f32`.
  **L4363 CN**: 声明 TableGen def `armpl_vexpm1q_f32`。
- **L4364 EN**: Declares TableGen def `armpl_vexpm1q_f64`.
  **L4364 CN**: 声明 TableGen def `armpl_vexpm1q_f64`。
- **L4365 EN**: Declares TableGen def `armpl_vexpq_f32`.
  **L4365 CN**: 声明 TableGen def `armpl_vexpq_f32`。
- **L4366 EN**: Declares TableGen def `armpl_vexpq_f64`.
  **L4366 CN**: 声明 TableGen def `armpl_vexpq_f64`。
- **L4367 EN**: Declares TableGen def `armpl_vfdimq_f32`.
  **L4367 CN**: 声明 TableGen def `armpl_vfdimq_f32`。
- **L4368 EN**: Declares TableGen def `armpl_vfdimq_f64`.
  **L4368 CN**: 声明 TableGen def `armpl_vfdimq_f64`。
- **L4369 EN**: Declares TableGen def `armpl_vfmaq_f32`.
  **L4369 CN**: 声明 TableGen def `armpl_vfmaq_f32`。
- **L4370 EN**: Declares TableGen def `armpl_vfmaq_f64`.
  **L4370 CN**: 声明 TableGen def `armpl_vfmaq_f64`。
- **L4371 EN**: Declares TableGen def `armpl_vfmaxq_f32`.
  **L4371 CN**: 声明 TableGen def `armpl_vfmaxq_f32`。
- **L4372 EN**: Declares TableGen def `armpl_vfmaxq_f64`.
  **L4372 CN**: 声明 TableGen def `armpl_vfmaxq_f64`。
- **L4373 EN**: Declares TableGen def `armpl_vfminq_f32`.
  **L4373 CN**: 声明 TableGen def `armpl_vfminq_f32`。
- **L4374 EN**: Declares TableGen def `armpl_vfminq_f64`.
  **L4374 CN**: 声明 TableGen def `armpl_vfminq_f64`。
- **L4375 EN**: Declares TableGen def `armpl_vfmodq_f32`.
  **L4375 CN**: 声明 TableGen def `armpl_vfmodq_f32`。
- **L4376 EN**: Declares TableGen def `armpl_vfmodq_f64`.
  **L4376 CN**: 声明 TableGen def `armpl_vfmodq_f64`。
- **L4377 EN**: Declares TableGen def `armpl_vhypotq_f32`.
  **L4377 CN**: 声明 TableGen def `armpl_vhypotq_f32`。
- **L4378 EN**: Declares TableGen def `armpl_vhypotq_f64`.
  **L4378 CN**: 声明 TableGen def `armpl_vhypotq_f64`。
- **L4379 EN**: Declares TableGen def `armpl_vilogbq_f32`.
  **L4379 CN**: 声明 TableGen def `armpl_vilogbq_f32`。
- **L4380 EN**: Declares TableGen def `armpl_vilogbq_f64`.
  **L4380 CN**: 声明 TableGen def `armpl_vilogbq_f64`。
- **L4381 EN**: Declares TableGen def `armpl_vldexpq_f32`.
  **L4381 CN**: 声明 TableGen def `armpl_vldexpq_f32`。
- **L4382 EN**: Declares TableGen def `armpl_vldexpq_f64`.
  **L4382 CN**: 声明 TableGen def `armpl_vldexpq_f64`。
- **L4383 EN**: Declares TableGen def `armpl_vlgammaq_f32`.
  **L4383 CN**: 声明 TableGen def `armpl_vlgammaq_f32`。
- **L4384 EN**: Declares TableGen def `armpl_vlgammaq_f64`.
  **L4384 CN**: 声明 TableGen def `armpl_vlgammaq_f64`。

### Lines 4385-4416

````tablegen
  def armpl_vlog10q_f32 : RuntimeLibcallImpl<LOG10_V4F32>;
  def armpl_vlog10q_f64 : RuntimeLibcallImpl<LOG10_V2F64>;
  def armpl_vlog1pq_f32 : RuntimeLibcallImpl<LOG1P_V4F32>;
  def armpl_vlog1pq_f64 : RuntimeLibcallImpl<LOG1P_V2F64>;
  def armpl_vlog2q_f32 : RuntimeLibcallImpl<LOG2_V4F32>;
  def armpl_vlog2q_f64 : RuntimeLibcallImpl<LOG2_V2F64>;
  def armpl_vlogq_f32 : RuntimeLibcallImpl<LOG_V4F32>;
  def armpl_vlogq_f64 : RuntimeLibcallImpl<LOG_V2F64>;
  def armpl_vmodfq_f32 : RuntimeLibcallImpl<MODF_V4F32>;
  def armpl_vmodfq_f64 : RuntimeLibcallImpl<MODF_V2F64>;
  def armpl_vnextafterq_f32 : RuntimeLibcallImpl<NEXTAFTER_V4F32>;
  def armpl_vnextafterq_f64 : RuntimeLibcallImpl<NEXTAFTER_V2F64>;
  def armpl_vpowq_f32 : RuntimeLibcallImpl<POW_V4F32>;
  def armpl_vpowq_f64 : RuntimeLibcallImpl<POW_V2F64>;
  def armpl_vsincospiq_f32 : RuntimeLibcallImpl<SINCOSPI_V4F32>;
  def armpl_vsincospiq_f64 : RuntimeLibcallImpl<SINCOSPI_V2F64>;
  def armpl_vsincosq_f32 : RuntimeLibcallImpl<SINCOS_V4F32>;
  def armpl_vsincosq_f64 : RuntimeLibcallImpl<SINCOS_V2F64>;
  def armpl_vsinhq_f32 : RuntimeLibcallImpl<SINH_V4F32>;
  def armpl_vsinhq_f64 : RuntimeLibcallImpl<SINH_V2F64>;
  def armpl_vsinpiq_f32 : RuntimeLibcallImpl<SINPI_V4F32>;
  def armpl_vsinpiq_f64 : RuntimeLibcallImpl<SINPI_V2F64>;
  def armpl_vsinq_f32 : RuntimeLibcallImpl<SIN_V4F32>;
  def armpl_vsinq_f64 : RuntimeLibcallImpl<SIN_V2F64>;
  def armpl_vsqrtq_f32 : RuntimeLibcallImpl<SQRT_V4F32>;
  def armpl_vsqrtq_f64 : RuntimeLibcallImpl<SQRT_V2F64>;
  def armpl_vtanhq_f32 : RuntimeLibcallImpl<TANH_V4F32>;
  def armpl_vtanhq_f64 : RuntimeLibcallImpl<TANH_V2F64>;
  def armpl_vtanq_f32 : RuntimeLibcallImpl<TAN_V4F32>;
  def armpl_vtanq_f64 : RuntimeLibcallImpl<TAN_V2F64>;
  def armpl_vtgammaq_f32 : RuntimeLibcallImpl<TGAMMA_V4F32>;
  def armpl_vtgammaq_f64 : RuntimeLibcallImpl<TGAMMA_V2F64>;
````
- **L4385 EN**: Declares TableGen def `armpl_vlog10q_f32`.
  **L4385 CN**: 声明 TableGen def `armpl_vlog10q_f32`。
- **L4386 EN**: Declares TableGen def `armpl_vlog10q_f64`.
  **L4386 CN**: 声明 TableGen def `armpl_vlog10q_f64`。
- **L4387 EN**: Declares TableGen def `armpl_vlog1pq_f32`.
  **L4387 CN**: 声明 TableGen def `armpl_vlog1pq_f32`。
- **L4388 EN**: Declares TableGen def `armpl_vlog1pq_f64`.
  **L4388 CN**: 声明 TableGen def `armpl_vlog1pq_f64`。
- **L4389 EN**: Declares TableGen def `armpl_vlog2q_f32`.
  **L4389 CN**: 声明 TableGen def `armpl_vlog2q_f32`。
- **L4390 EN**: Declares TableGen def `armpl_vlog2q_f64`.
  **L4390 CN**: 声明 TableGen def `armpl_vlog2q_f64`。
- **L4391 EN**: Declares TableGen def `armpl_vlogq_f32`.
  **L4391 CN**: 声明 TableGen def `armpl_vlogq_f32`。
- **L4392 EN**: Declares TableGen def `armpl_vlogq_f64`.
  **L4392 CN**: 声明 TableGen def `armpl_vlogq_f64`。
- **L4393 EN**: Declares TableGen def `armpl_vmodfq_f32`.
  **L4393 CN**: 声明 TableGen def `armpl_vmodfq_f32`。
- **L4394 EN**: Declares TableGen def `armpl_vmodfq_f64`.
  **L4394 CN**: 声明 TableGen def `armpl_vmodfq_f64`。
- **L4395 EN**: Declares TableGen def `armpl_vnextafterq_f32`.
  **L4395 CN**: 声明 TableGen def `armpl_vnextafterq_f32`。
- **L4396 EN**: Declares TableGen def `armpl_vnextafterq_f64`.
  **L4396 CN**: 声明 TableGen def `armpl_vnextafterq_f64`。
- **L4397 EN**: Declares TableGen def `armpl_vpowq_f32`.
  **L4397 CN**: 声明 TableGen def `armpl_vpowq_f32`。
- **L4398 EN**: Declares TableGen def `armpl_vpowq_f64`.
  **L4398 CN**: 声明 TableGen def `armpl_vpowq_f64`。
- **L4399 EN**: Declares TableGen def `armpl_vsincospiq_f32`.
  **L4399 CN**: 声明 TableGen def `armpl_vsincospiq_f32`。
- **L4400 EN**: Declares TableGen def `armpl_vsincospiq_f64`.
  **L4400 CN**: 声明 TableGen def `armpl_vsincospiq_f64`。
- **L4401 EN**: Declares TableGen def `armpl_vsincosq_f32`.
  **L4401 CN**: 声明 TableGen def `armpl_vsincosq_f32`。
- **L4402 EN**: Declares TableGen def `armpl_vsincosq_f64`.
  **L4402 CN**: 声明 TableGen def `armpl_vsincosq_f64`。
- **L4403 EN**: Declares TableGen def `armpl_vsinhq_f32`.
  **L4403 CN**: 声明 TableGen def `armpl_vsinhq_f32`。
- **L4404 EN**: Declares TableGen def `armpl_vsinhq_f64`.
  **L4404 CN**: 声明 TableGen def `armpl_vsinhq_f64`。
- **L4405 EN**: Declares TableGen def `armpl_vsinpiq_f32`.
  **L4405 CN**: 声明 TableGen def `armpl_vsinpiq_f32`。
- **L4406 EN**: Declares TableGen def `armpl_vsinpiq_f64`.
  **L4406 CN**: 声明 TableGen def `armpl_vsinpiq_f64`。
- **L4407 EN**: Declares TableGen def `armpl_vsinq_f32`.
  **L4407 CN**: 声明 TableGen def `armpl_vsinq_f32`。
- **L4408 EN**: Declares TableGen def `armpl_vsinq_f64`.
  **L4408 CN**: 声明 TableGen def `armpl_vsinq_f64`。
- **L4409 EN**: Declares TableGen def `armpl_vsqrtq_f32`.
  **L4409 CN**: 声明 TableGen def `armpl_vsqrtq_f32`。
- **L4410 EN**: Declares TableGen def `armpl_vsqrtq_f64`.
  **L4410 CN**: 声明 TableGen def `armpl_vsqrtq_f64`。
- **L4411 EN**: Declares TableGen def `armpl_vtanhq_f32`.
  **L4411 CN**: 声明 TableGen def `armpl_vtanhq_f32`。
- **L4412 EN**: Declares TableGen def `armpl_vtanhq_f64`.
  **L4412 CN**: 声明 TableGen def `armpl_vtanhq_f64`。
- **L4413 EN**: Declares TableGen def `armpl_vtanq_f32`.
  **L4413 CN**: 声明 TableGen def `armpl_vtanq_f32`。
- **L4414 EN**: Declares TableGen def `armpl_vtanq_f64`.
  **L4414 CN**: 声明 TableGen def `armpl_vtanq_f64`。
- **L4415 EN**: Declares TableGen def `armpl_vtgammaq_f32`.
  **L4415 CN**: 声明 TableGen def `armpl_vtgammaq_f32`。
- **L4416 EN**: Declares TableGen def `armpl_vtgammaq_f64`.
  **L4416 CN**: 声明 TableGen def `armpl_vtgammaq_f64`。

### Lines 4417-4448

````tablegen
}

//===----------------------------------------------------------------------===//
// AMD vector math library (AMDLIBM) functions
//===----------------------------------------------------------------------===//

defset list<RuntimeLibcallImpl> AMDLIBM_VECFUNCS = {
  def amd_vrd2_atan : RuntimeLibcallImpl<ATAN_V2F64>;
  def amd_vrd2_cbrt : RuntimeLibcallImpl<CBRT_V2F64>;
  def amd_vrd2_cos : RuntimeLibcallImpl<COS_V2F64>;
  def amd_vrd2_erf : RuntimeLibcallImpl<ERF_V2F64>;
  def amd_vrd2_exp : RuntimeLibcallImpl<EXP_V2F64>;
  def amd_vrd2_exp10 : RuntimeLibcallImpl<EXP10_V2F64>;
  def amd_vrd2_exp2 : RuntimeLibcallImpl<EXP2_V2F64>;
  def amd_vrd2_log : RuntimeLibcallImpl<LOG_V2F64>;
  def amd_vrd2_log10 : RuntimeLibcallImpl<LOG10_V2F64>;
  def amd_vrd2_log1p : RuntimeLibcallImpl<LOG1P_V2F64>;
  def amd_vrd2_log2 : RuntimeLibcallImpl<LOG2_V2F64>;
  def amd_vrd2_pow : RuntimeLibcallImpl<POW_V2F64>;
  def amd_vrd2_sin : RuntimeLibcallImpl<SIN_V2F64>;
  def amd_vrd2_sincos : RuntimeLibcallImpl<SINCOS_V2F64>;
  def amd_vrd2_tan : RuntimeLibcallImpl<TAN_V2F64>;
  def amd_vrd4_atan : RuntimeLibcallImpl<ATAN_V4F64>;
  def amd_vrd4_cos : RuntimeLibcallImpl<COS_V4F64>;
  def amd_vrd4_erf : RuntimeLibcallImpl<ERF_V4F32>;
  def amd_vrd4_exp : RuntimeLibcallImpl<EXP_V4F64>;
  def amd_vrd4_exp2 : RuntimeLibcallImpl<EXP2_V4F64>;
  def amd_vrd4_log : RuntimeLibcallImpl<LOG_V4F64>;
  def amd_vrd4_log2 : RuntimeLibcallImpl<LOG2_V4F64>;
  def amd_vrd4_pow : RuntimeLibcallImpl<POW_V4F64>;
  def amd_vrd4_sin : RuntimeLibcallImpl<SIN_V4F64>;
  def amd_vrd4_sincos : RuntimeLibcallImpl<SINCOS_V4F64>;
````
- **L4417 EN**: Closes the current lexical scope or compound statement.
  **L4417 CN**: 结束当前词法作用域或复合语句块。
- **L4418 EN**: Blank line separating nearby declarations or logic blocks.
  **L4418 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L4419 EN**: Banner comment marking a file or section boundary.
  **L4419 CN**: 横幅注释，用于标记文件或章节边界。
- **L4420 EN**: Comment explains nearby logic, invariants, or intent: `AMD vector math library (AMDLIBM) functions`.
  **L4420 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`AMD vector math library (AMDLIBM) functions`。
- **L4421 EN**: Banner comment marking a file or section boundary.
  **L4421 CN**: 横幅注释，用于标记文件或章节边界。
- **L4422 EN**: Blank line separating nearby declarations or logic blocks.
  **L4422 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L4423 EN**: Continues the surrounding expression or declaration: `defset list<RuntimeLibcallImpl> AMDLIBM_VECFUNCS = {`.
  **L4423 CN**: 继续构造周围的表达式或声明：`defset list<RuntimeLibcallImpl> AMDLIBM_VECFUNCS = {`。
- **L4424 EN**: Declares TableGen def `amd_vrd2_atan`.
  **L4424 CN**: 声明 TableGen def `amd_vrd2_atan`。
- **L4425 EN**: Declares TableGen def `amd_vrd2_cbrt`.
  **L4425 CN**: 声明 TableGen def `amd_vrd2_cbrt`。
- **L4426 EN**: Declares TableGen def `amd_vrd2_cos`.
  **L4426 CN**: 声明 TableGen def `amd_vrd2_cos`。
- **L4427 EN**: Declares TableGen def `amd_vrd2_erf`.
  **L4427 CN**: 声明 TableGen def `amd_vrd2_erf`。
- **L4428 EN**: Declares TableGen def `amd_vrd2_exp`.
  **L4428 CN**: 声明 TableGen def `amd_vrd2_exp`。
- **L4429 EN**: Declares TableGen def `amd_vrd2_exp10`.
  **L4429 CN**: 声明 TableGen def `amd_vrd2_exp10`。
- **L4430 EN**: Declares TableGen def `amd_vrd2_exp2`.
  **L4430 CN**: 声明 TableGen def `amd_vrd2_exp2`。
- **L4431 EN**: Declares TableGen def `amd_vrd2_log`.
  **L4431 CN**: 声明 TableGen def `amd_vrd2_log`。
- **L4432 EN**: Declares TableGen def `amd_vrd2_log10`.
  **L4432 CN**: 声明 TableGen def `amd_vrd2_log10`。
- **L4433 EN**: Declares TableGen def `amd_vrd2_log1p`.
  **L4433 CN**: 声明 TableGen def `amd_vrd2_log1p`。
- **L4434 EN**: Declares TableGen def `amd_vrd2_log2`.
  **L4434 CN**: 声明 TableGen def `amd_vrd2_log2`。
- **L4435 EN**: Declares TableGen def `amd_vrd2_pow`.
  **L4435 CN**: 声明 TableGen def `amd_vrd2_pow`。
- **L4436 EN**: Declares TableGen def `amd_vrd2_sin`.
  **L4436 CN**: 声明 TableGen def `amd_vrd2_sin`。
- **L4437 EN**: Declares TableGen def `amd_vrd2_sincos`.
  **L4437 CN**: 声明 TableGen def `amd_vrd2_sincos`。
- **L4438 EN**: Declares TableGen def `amd_vrd2_tan`.
  **L4438 CN**: 声明 TableGen def `amd_vrd2_tan`。
- **L4439 EN**: Declares TableGen def `amd_vrd4_atan`.
  **L4439 CN**: 声明 TableGen def `amd_vrd4_atan`。
- **L4440 EN**: Declares TableGen def `amd_vrd4_cos`.
  **L4440 CN**: 声明 TableGen def `amd_vrd4_cos`。
- **L4441 EN**: Declares TableGen def `amd_vrd4_erf`.
  **L4441 CN**: 声明 TableGen def `amd_vrd4_erf`。
- **L4442 EN**: Declares TableGen def `amd_vrd4_exp`.
  **L4442 CN**: 声明 TableGen def `amd_vrd4_exp`。
- **L4443 EN**: Declares TableGen def `amd_vrd4_exp2`.
  **L4443 CN**: 声明 TableGen def `amd_vrd4_exp2`。
- **L4444 EN**: Declares TableGen def `amd_vrd4_log`.
  **L4444 CN**: 声明 TableGen def `amd_vrd4_log`。
- **L4445 EN**: Declares TableGen def `amd_vrd4_log2`.
  **L4445 CN**: 声明 TableGen def `amd_vrd4_log2`。
- **L4446 EN**: Declares TableGen def `amd_vrd4_pow`.
  **L4446 CN**: 声明 TableGen def `amd_vrd4_pow`。
- **L4447 EN**: Declares TableGen def `amd_vrd4_sin`.
  **L4447 CN**: 声明 TableGen def `amd_vrd4_sin`。
- **L4448 EN**: Declares TableGen def `amd_vrd4_sincos`.
  **L4448 CN**: 声明 TableGen def `amd_vrd4_sincos`。

### Lines 4449-4480

````tablegen
  def amd_vrd4_tan : RuntimeLibcallImpl<TAN_V4F64>;
  def amd_vrd8_asin : RuntimeLibcallImpl<ASIN_V8F64>;
  def amd_vrd8_atan : RuntimeLibcallImpl<ATAN_V8F64>;
  def amd_vrd8_cos : RuntimeLibcallImpl<COS_V8F64>;
  def amd_vrd8_erf : RuntimeLibcallImpl<ERF_V8F32>;
  def amd_vrd8_exp : RuntimeLibcallImpl<EXP_V8F64>;
  def amd_vrd8_exp2 : RuntimeLibcallImpl<EXP2_V8F64>;
  def amd_vrd8_log : RuntimeLibcallImpl<LOG_V8F64>;
  def amd_vrd8_log2 : RuntimeLibcallImpl<LOG2_V8F64>;
  def amd_vrd8_pow : RuntimeLibcallImpl<POW_V8F64>;
  def amd_vrd8_sin : RuntimeLibcallImpl<SIN_V8F64>;
  def amd_vrd8_sincos : RuntimeLibcallImpl<SINCOS_V8F64>;
  def amd_vrd8_tan : RuntimeLibcallImpl<TAN_V8F64>;
  def amd_vrd2_erfinv : RuntimeLibcallImpl<ERFINV_V2F64>;
  def amd_vrd4_erfinv : RuntimeLibcallImpl<ERFINV_V4F64>;
  def amd_vrd8_erfinv : RuntimeLibcallImpl<ERFINV_V8F64>;
  def amd_vrd2_erfcinv : RuntimeLibcallImpl<ERFCINV_V2F64>;
  def amd_vrd4_erfcinv : RuntimeLibcallImpl<ERFCINV_V4F64>;
  def amd_vrd8_erfcinv : RuntimeLibcallImpl<ERFCINV_V8F64>;
  def amd_vrs16_acosf : RuntimeLibcallImpl<ACOS_V16F32>;
  def amd_vrs16_asinf : RuntimeLibcallImpl<ASIN_V16F32>;
  def amd_vrs16_atanf : RuntimeLibcallImpl<ATAN_V16F32>;
  def amd_vrs16_cosf : RuntimeLibcallImpl<COS_V16F32>;
  def amd_vrs16_erff : RuntimeLibcallImpl<ERF_V16F32>;
  def amd_vrs16_exp2f : RuntimeLibcallImpl<EXP2_V16F32>;
  def amd_vrs16_expf : RuntimeLibcallImpl<EXP_V16F32>;
  def amd_vrs16_log10f : RuntimeLibcallImpl<LOG10_V16F32>;
  def amd_vrs16_log2f : RuntimeLibcallImpl<LOG2_V16F32>;
  def amd_vrs16_logf : RuntimeLibcallImpl<LOG_V16F32>;
  def amd_vrs16_powf : RuntimeLibcallImpl<POW_V16F32>;
  def amd_vrs16_sincosf : RuntimeLibcallImpl<SINCOS_V16F32>;
  def amd_vrs16_sinf : RuntimeLibcallImpl<SIN_V16F32>;
````
- **L4449 EN**: Declares TableGen def `amd_vrd4_tan`.
  **L4449 CN**: 声明 TableGen def `amd_vrd4_tan`。
- **L4450 EN**: Declares TableGen def `amd_vrd8_asin`.
  **L4450 CN**: 声明 TableGen def `amd_vrd8_asin`。
- **L4451 EN**: Declares TableGen def `amd_vrd8_atan`.
  **L4451 CN**: 声明 TableGen def `amd_vrd8_atan`。
- **L4452 EN**: Declares TableGen def `amd_vrd8_cos`.
  **L4452 CN**: 声明 TableGen def `amd_vrd8_cos`。
- **L4453 EN**: Declares TableGen def `amd_vrd8_erf`.
  **L4453 CN**: 声明 TableGen def `amd_vrd8_erf`。
- **L4454 EN**: Declares TableGen def `amd_vrd8_exp`.
  **L4454 CN**: 声明 TableGen def `amd_vrd8_exp`。
- **L4455 EN**: Declares TableGen def `amd_vrd8_exp2`.
  **L4455 CN**: 声明 TableGen def `amd_vrd8_exp2`。
- **L4456 EN**: Declares TableGen def `amd_vrd8_log`.
  **L4456 CN**: 声明 TableGen def `amd_vrd8_log`。
- **L4457 EN**: Declares TableGen def `amd_vrd8_log2`.
  **L4457 CN**: 声明 TableGen def `amd_vrd8_log2`。
- **L4458 EN**: Declares TableGen def `amd_vrd8_pow`.
  **L4458 CN**: 声明 TableGen def `amd_vrd8_pow`。
- **L4459 EN**: Declares TableGen def `amd_vrd8_sin`.
  **L4459 CN**: 声明 TableGen def `amd_vrd8_sin`。
- **L4460 EN**: Declares TableGen def `amd_vrd8_sincos`.
  **L4460 CN**: 声明 TableGen def `amd_vrd8_sincos`。
- **L4461 EN**: Declares TableGen def `amd_vrd8_tan`.
  **L4461 CN**: 声明 TableGen def `amd_vrd8_tan`。
- **L4462 EN**: Declares TableGen def `amd_vrd2_erfinv`.
  **L4462 CN**: 声明 TableGen def `amd_vrd2_erfinv`。
- **L4463 EN**: Declares TableGen def `amd_vrd4_erfinv`.
  **L4463 CN**: 声明 TableGen def `amd_vrd4_erfinv`。
- **L4464 EN**: Declares TableGen def `amd_vrd8_erfinv`.
  **L4464 CN**: 声明 TableGen def `amd_vrd8_erfinv`。
- **L4465 EN**: Declares TableGen def `amd_vrd2_erfcinv`.
  **L4465 CN**: 声明 TableGen def `amd_vrd2_erfcinv`。
- **L4466 EN**: Declares TableGen def `amd_vrd4_erfcinv`.
  **L4466 CN**: 声明 TableGen def `amd_vrd4_erfcinv`。
- **L4467 EN**: Declares TableGen def `amd_vrd8_erfcinv`.
  **L4467 CN**: 声明 TableGen def `amd_vrd8_erfcinv`。
- **L4468 EN**: Declares TableGen def `amd_vrs16_acosf`.
  **L4468 CN**: 声明 TableGen def `amd_vrs16_acosf`。
- **L4469 EN**: Declares TableGen def `amd_vrs16_asinf`.
  **L4469 CN**: 声明 TableGen def `amd_vrs16_asinf`。
- **L4470 EN**: Declares TableGen def `amd_vrs16_atanf`.
  **L4470 CN**: 声明 TableGen def `amd_vrs16_atanf`。
- **L4471 EN**: Declares TableGen def `amd_vrs16_cosf`.
  **L4471 CN**: 声明 TableGen def `amd_vrs16_cosf`。
- **L4472 EN**: Declares TableGen def `amd_vrs16_erff`.
  **L4472 CN**: 声明 TableGen def `amd_vrs16_erff`。
- **L4473 EN**: Declares TableGen def `amd_vrs16_exp2f`.
  **L4473 CN**: 声明 TableGen def `amd_vrs16_exp2f`。
- **L4474 EN**: Declares TableGen def `amd_vrs16_expf`.
  **L4474 CN**: 声明 TableGen def `amd_vrs16_expf`。
- **L4475 EN**: Declares TableGen def `amd_vrs16_log10f`.
  **L4475 CN**: 声明 TableGen def `amd_vrs16_log10f`。
- **L4476 EN**: Declares TableGen def `amd_vrs16_log2f`.
  **L4476 CN**: 声明 TableGen def `amd_vrs16_log2f`。
- **L4477 EN**: Declares TableGen def `amd_vrs16_logf`.
  **L4477 CN**: 声明 TableGen def `amd_vrs16_logf`。
- **L4478 EN**: Declares TableGen def `amd_vrs16_powf`.
  **L4478 CN**: 声明 TableGen def `amd_vrs16_powf`。
- **L4479 EN**: Declares TableGen def `amd_vrs16_sincosf`.
  **L4479 CN**: 声明 TableGen def `amd_vrs16_sincosf`。
- **L4480 EN**: Declares TableGen def `amd_vrs16_sinf`.
  **L4480 CN**: 声明 TableGen def `amd_vrs16_sinf`。

### Lines 4481-4512

````tablegen
  def amd_vrs16_tanf : RuntimeLibcallImpl<TAN_V16F32>;
  def amd_vrs16_tanhf : RuntimeLibcallImpl<TANH_V16F32>;
  def amd_vrs4_acosf : RuntimeLibcallImpl<ACOS_V4F32>;
  def amd_vrs4_asinf : RuntimeLibcallImpl<ASIN_V4F32>;
  def amd_vrs4_atanf : RuntimeLibcallImpl<ATAN_V4F32>;
  def amd_vrs4_cbrtf : RuntimeLibcallImpl<CBRT_V4F32>;
  def amd_vrs4_cosf : RuntimeLibcallImpl<COS_V4F32>;
  def amd_vrs4_coshf : RuntimeLibcallImpl<COSH_V4F32>;
  def amd_vrs4_erff : RuntimeLibcallImpl<ERF_V4F32>;
  def amd_vrs4_exp10f : RuntimeLibcallImpl<EXP10_V4F32>;
  def amd_vrs4_exp2f : RuntimeLibcallImpl<EXP2_V4F32>;
  def amd_vrs4_expf : RuntimeLibcallImpl<EXP_V4F32>;
  def amd_vrs4_expm1f : RuntimeLibcallImpl<EXPM1_V4F32>;
  def amd_vrs4_log10f : RuntimeLibcallImpl<LOG10_V4F32>;
  def amd_vrs4_log1pf : RuntimeLibcallImpl<LOG1P_V4F32>;
  def amd_vrs4_log2f : RuntimeLibcallImpl<LOG2_V4F32>;
  def amd_vrs4_logf : RuntimeLibcallImpl<LOG_V4F32>;
  def amd_vrs4_powf : RuntimeLibcallImpl<POW_V4F32>;
  def amd_vrs4_sincosf : RuntimeLibcallImpl<SINCOS_V4F32>;
  def amd_vrs4_sinf : RuntimeLibcallImpl<SIN_V4F32>;
  def amd_vrs4_tanf : RuntimeLibcallImpl<TAN_V4F32>;
  def amd_vrs4_tanhf : RuntimeLibcallImpl<TANH_V4F32>;
  def amd_vrs8_acosf : RuntimeLibcallImpl<ACOS_V8F32>;
  def amd_vrs8_asinf : RuntimeLibcallImpl<ASIN_V8F32>;
  def amd_vrs8_atanf : RuntimeLibcallImpl<ATAN_V8F32>;
  def amd_vrs8_cosf : RuntimeLibcallImpl<COS_V8F32>;
  def amd_vrs8_coshf : RuntimeLibcallImpl<COSH_V8F32>;
  def amd_vrs8_erff : RuntimeLibcallImpl<ERF_V8F32>;
  def amd_vrs8_exp2f : RuntimeLibcallImpl<EXP2_V8F32>;
  def amd_vrs8_expf : RuntimeLibcallImpl<EXP_V8F32>;
  def amd_vrs8_log10f : RuntimeLibcallImpl<LOG10_V8F32>;
  def amd_vrs8_log2f : RuntimeLibcallImpl<LOG2_V8F32>;
````
- **L4481 EN**: Declares TableGen def `amd_vrs16_tanf`.
  **L4481 CN**: 声明 TableGen def `amd_vrs16_tanf`。
- **L4482 EN**: Declares TableGen def `amd_vrs16_tanhf`.
  **L4482 CN**: 声明 TableGen def `amd_vrs16_tanhf`。
- **L4483 EN**: Declares TableGen def `amd_vrs4_acosf`.
  **L4483 CN**: 声明 TableGen def `amd_vrs4_acosf`。
- **L4484 EN**: Declares TableGen def `amd_vrs4_asinf`.
  **L4484 CN**: 声明 TableGen def `amd_vrs4_asinf`。
- **L4485 EN**: Declares TableGen def `amd_vrs4_atanf`.
  **L4485 CN**: 声明 TableGen def `amd_vrs4_atanf`。
- **L4486 EN**: Declares TableGen def `amd_vrs4_cbrtf`.
  **L4486 CN**: 声明 TableGen def `amd_vrs4_cbrtf`。
- **L4487 EN**: Declares TableGen def `amd_vrs4_cosf`.
  **L4487 CN**: 声明 TableGen def `amd_vrs4_cosf`。
- **L4488 EN**: Declares TableGen def `amd_vrs4_coshf`.
  **L4488 CN**: 声明 TableGen def `amd_vrs4_coshf`。
- **L4489 EN**: Declares TableGen def `amd_vrs4_erff`.
  **L4489 CN**: 声明 TableGen def `amd_vrs4_erff`。
- **L4490 EN**: Declares TableGen def `amd_vrs4_exp10f`.
  **L4490 CN**: 声明 TableGen def `amd_vrs4_exp10f`。
- **L4491 EN**: Declares TableGen def `amd_vrs4_exp2f`.
  **L4491 CN**: 声明 TableGen def `amd_vrs4_exp2f`。
- **L4492 EN**: Declares TableGen def `amd_vrs4_expf`.
  **L4492 CN**: 声明 TableGen def `amd_vrs4_expf`。
- **L4493 EN**: Declares TableGen def `amd_vrs4_expm1f`.
  **L4493 CN**: 声明 TableGen def `amd_vrs4_expm1f`。
- **L4494 EN**: Declares TableGen def `amd_vrs4_log10f`.
  **L4494 CN**: 声明 TableGen def `amd_vrs4_log10f`。
- **L4495 EN**: Declares TableGen def `amd_vrs4_log1pf`.
  **L4495 CN**: 声明 TableGen def `amd_vrs4_log1pf`。
- **L4496 EN**: Declares TableGen def `amd_vrs4_log2f`.
  **L4496 CN**: 声明 TableGen def `amd_vrs4_log2f`。
- **L4497 EN**: Declares TableGen def `amd_vrs4_logf`.
  **L4497 CN**: 声明 TableGen def `amd_vrs4_logf`。
- **L4498 EN**: Declares TableGen def `amd_vrs4_powf`.
  **L4498 CN**: 声明 TableGen def `amd_vrs4_powf`。
- **L4499 EN**: Declares TableGen def `amd_vrs4_sincosf`.
  **L4499 CN**: 声明 TableGen def `amd_vrs4_sincosf`。
- **L4500 EN**: Declares TableGen def `amd_vrs4_sinf`.
  **L4500 CN**: 声明 TableGen def `amd_vrs4_sinf`。
- **L4501 EN**: Declares TableGen def `amd_vrs4_tanf`.
  **L4501 CN**: 声明 TableGen def `amd_vrs4_tanf`。
- **L4502 EN**: Declares TableGen def `amd_vrs4_tanhf`.
  **L4502 CN**: 声明 TableGen def `amd_vrs4_tanhf`。
- **L4503 EN**: Declares TableGen def `amd_vrs8_acosf`.
  **L4503 CN**: 声明 TableGen def `amd_vrs8_acosf`。
- **L4504 EN**: Declares TableGen def `amd_vrs8_asinf`.
  **L4504 CN**: 声明 TableGen def `amd_vrs8_asinf`。
- **L4505 EN**: Declares TableGen def `amd_vrs8_atanf`.
  **L4505 CN**: 声明 TableGen def `amd_vrs8_atanf`。
- **L4506 EN**: Declares TableGen def `amd_vrs8_cosf`.
  **L4506 CN**: 声明 TableGen def `amd_vrs8_cosf`。
- **L4507 EN**: Declares TableGen def `amd_vrs8_coshf`.
  **L4507 CN**: 声明 TableGen def `amd_vrs8_coshf`。
- **L4508 EN**: Declares TableGen def `amd_vrs8_erff`.
  **L4508 CN**: 声明 TableGen def `amd_vrs8_erff`。
- **L4509 EN**: Declares TableGen def `amd_vrs8_exp2f`.
  **L4509 CN**: 声明 TableGen def `amd_vrs8_exp2f`。
- **L4510 EN**: Declares TableGen def `amd_vrs8_expf`.
  **L4510 CN**: 声明 TableGen def `amd_vrs8_expf`。
- **L4511 EN**: Declares TableGen def `amd_vrs8_log10f`.
  **L4511 CN**: 声明 TableGen def `amd_vrs8_log10f`。
- **L4512 EN**: Declares TableGen def `amd_vrs8_log2f`.
  **L4512 CN**: 声明 TableGen def `amd_vrs8_log2f`。

### Lines 4513-4531

````tablegen
  def amd_vrs8_logf : RuntimeLibcallImpl<LOG_V8F32>;
  def amd_vrs8_powf : RuntimeLibcallImpl<POW_V8F32>;
  def amd_vrs8_sincosf : RuntimeLibcallImpl<SINCOS_V8F32>;
  def amd_vrs8_sinf : RuntimeLibcallImpl<SIN_V8F32>;
  def amd_vrs8_tanf : RuntimeLibcallImpl<TAN_V8F32>;
  def amd_vrs8_tanhf : RuntimeLibcallImpl<TANH_V8F32>;
  def amd_vrd2_cdfnorm : RuntimeLibcallImpl<CDFNORM_V2F64>;
  def amd_vrd4_cdfnorm : RuntimeLibcallImpl<CDFNORM_V4F64>;
  def amd_vrd8_cdfnorm : RuntimeLibcallImpl<CDFNORM_V8F64>;
  def amd_vrd2_cdfnorminv : RuntimeLibcallImpl<CDFNORMINV_V2F64>;
  def amd_vrd4_cdfnorminv : RuntimeLibcallImpl<CDFNORMINV_V4F64>;
  def amd_vrd8_cdfnorminv : RuntimeLibcallImpl<CDFNORMINV_V8F64>;
  def amd_vrs4_roundf : RuntimeLibcallImpl<ROUND_V4F32>;
  def amd_vrs8_roundf : RuntimeLibcallImpl<ROUND_V8F32>;
  def amd_vrs16_roundf : RuntimeLibcallImpl<ROUND_V16F32>;
  def amd_vrd2_round : RuntimeLibcallImpl<ROUND_V2F64>;
  def amd_vrd4_round : RuntimeLibcallImpl<ROUND_V4F64>;
  def amd_vrd8_round : RuntimeLibcallImpl<ROUND_V8F64>;
}
````
- **L4513 EN**: Declares TableGen def `amd_vrs8_logf`.
  **L4513 CN**: 声明 TableGen def `amd_vrs8_logf`。
- **L4514 EN**: Declares TableGen def `amd_vrs8_powf`.
  **L4514 CN**: 声明 TableGen def `amd_vrs8_powf`。
- **L4515 EN**: Declares TableGen def `amd_vrs8_sincosf`.
  **L4515 CN**: 声明 TableGen def `amd_vrs8_sincosf`。
- **L4516 EN**: Declares TableGen def `amd_vrs8_sinf`.
  **L4516 CN**: 声明 TableGen def `amd_vrs8_sinf`。
- **L4517 EN**: Declares TableGen def `amd_vrs8_tanf`.
  **L4517 CN**: 声明 TableGen def `amd_vrs8_tanf`。
- **L4518 EN**: Declares TableGen def `amd_vrs8_tanhf`.
  **L4518 CN**: 声明 TableGen def `amd_vrs8_tanhf`。
- **L4519 EN**: Declares TableGen def `amd_vrd2_cdfnorm`.
  **L4519 CN**: 声明 TableGen def `amd_vrd2_cdfnorm`。
- **L4520 EN**: Declares TableGen def `amd_vrd4_cdfnorm`.
  **L4520 CN**: 声明 TableGen def `amd_vrd4_cdfnorm`。
- **L4521 EN**: Declares TableGen def `amd_vrd8_cdfnorm`.
  **L4521 CN**: 声明 TableGen def `amd_vrd8_cdfnorm`。
- **L4522 EN**: Declares TableGen def `amd_vrd2_cdfnorminv`.
  **L4522 CN**: 声明 TableGen def `amd_vrd2_cdfnorminv`。
- **L4523 EN**: Declares TableGen def `amd_vrd4_cdfnorminv`.
  **L4523 CN**: 声明 TableGen def `amd_vrd4_cdfnorminv`。
- **L4524 EN**: Declares TableGen def `amd_vrd8_cdfnorminv`.
  **L4524 CN**: 声明 TableGen def `amd_vrd8_cdfnorminv`。
- **L4525 EN**: Declares TableGen def `amd_vrs4_roundf`.
  **L4525 CN**: 声明 TableGen def `amd_vrs4_roundf`。
- **L4526 EN**: Declares TableGen def `amd_vrs8_roundf`.
  **L4526 CN**: 声明 TableGen def `amd_vrs8_roundf`。
- **L4527 EN**: Declares TableGen def `amd_vrs16_roundf`.
  **L4527 CN**: 声明 TableGen def `amd_vrs16_roundf`。
- **L4528 EN**: Declares TableGen def `amd_vrd2_round`.
  **L4528 CN**: 声明 TableGen def `amd_vrd2_round`。
- **L4529 EN**: Declares TableGen def `amd_vrd4_round`.
  **L4529 CN**: 声明 TableGen def `amd_vrd4_round`。
- **L4530 EN**: Declares TableGen def `amd_vrd8_round`.
  **L4530 CN**: 声明 TableGen def `amd_vrd8_round`。
- **L4531 EN**: Closes the current lexical scope or compound statement.
  **L4531 CN**: 结束当前词法作用域或复合语句块。

## Key Concepts / 关键概念

- **TableGen record modeling / TableGen 记录建模**
- **Function-level IR management / 函数级 IR 管理**
- **Instruction semantics / 指令语义**
- **Intrinsic declaration and lowering hooks / Intrinsic 声明与降级钩子**
- **Typed error propagation / 类型化错误传播**

## Dependencies / 依赖关系

- `llvm/IR/RuntimeLibcallsImpl.td`: Provides LLVM IR core types, instructions, and ownership utilities. / 提供LLVM IR 核心类型、指令与所有权辅助组件。
