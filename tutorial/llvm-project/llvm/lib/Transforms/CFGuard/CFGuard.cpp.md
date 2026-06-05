# CFGuard.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `llvm/lib/Transforms/CFGuard/CFGuard.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**: \file This file contains the IR transform to add Microsoft's Control Flow Guard checks on Windows targets. / 该文件位于 `Transforms/CFGuard`，主要实现 `CFGuard` 相关的 LLVM 变换逻辑、辅助流程以及 pass 接线代码。

## Line-by-Line Analysis / 逐行分析

### Lines 1-20

```cpp
//===-- CFGuard.cpp - Control Flow Guard checks -----------------*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
///
/// \file
/// This file contains the IR transform to add Microsoft's Control Flow Guard
/// checks on Windows targets.
///
//===----------------------------------------------------------------------===//

#include "llvm/Transforms/CFGuard.h"
#include "llvm/ADT/SmallVector.h"
#include "llvm/ADT/Statistic.h"
#include "llvm/IR/CallingConv.h"
#include "llvm/IR/IRBuilder.h"
#include "llvm/IR/Instruction.h"
```

- **L1**: Banner comment marking a file section boundary. / 横幅注释，用于标记文件分节。
- **L2**: Separator comment used to visually break up sections. / 分隔性注释，用于在视觉上划分小节。
- **L3**: Comment documents the nearby logic or transformation intent: `Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.`. / 注释说明了附近代码的逻辑或变换意图：`Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.`。
- **L4**: Comment documents the nearby logic or transformation intent: `See https://llvm.org/LICENSE.txt for license information.`. / 注释说明了附近代码的逻辑或变换意图：`See https://llvm.org/LICENSE.txt for license information.`。
- **L5**: Comment documents the nearby logic or transformation intent: `SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception`. / 注释说明了附近代码的逻辑或变换意图：`SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception`。
- **L6**: Separator comment used to visually break up sections. / 分隔性注释，用于在视觉上划分小节。
- **L7**: Banner comment marking a file section boundary. / 横幅注释，用于标记文件分节。
- **L8**: Separator comment used to visually break up sections. / 分隔性注释，用于在视觉上划分小节。
- **L9**: Comment documents the nearby logic or transformation intent: `\file`. / 注释说明了附近代码的逻辑或变换意图：`\file`。
- **L10**: Comment documents the nearby logic or transformation intent: `This file contains the IR transform to add Microsoft's Control Flow Guard`. / 注释说明了附近代码的逻辑或变换意图：`This file contains the IR transform to add Microsoft's Control Flow Guard`。
- **L11**: Comment documents the nearby logic or transformation intent: `checks on Windows targets.`. / 注释说明了附近代码的逻辑或变换意图：`checks on Windows targets.`。
- **L12**: Separator comment used to visually break up sections. / 分隔性注释，用于在视觉上划分小节。
- **L13**: Banner comment marking a file section boundary. / 横幅注释，用于标记文件分节。
- **L14**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L15**: Includes "llvm/Transforms/CFGuard.h" to access transform-specific declarations. / 引入 "llvm/Transforms/CFGuard.h" 以使用变换相关声明。
- **L16**: Includes "llvm/ADT/SmallVector.h" to access LLVM ADT data structures/utilities. / 引入 "llvm/ADT/SmallVector.h" 以使用LLVM ADT 数据结构/工具。
- **L17**: Includes "llvm/ADT/Statistic.h" to access LLVM ADT data structures/utilities. / 引入 "llvm/ADT/Statistic.h" 以使用LLVM ADT 数据结构/工具。
- **L18**: Includes "llvm/IR/CallingConv.h" to access LLVM IR core types and builders. / 引入 "llvm/IR/CallingConv.h" 以使用LLVM IR 核心类型与构造工具。
- **L19**: Includes "llvm/IR/IRBuilder.h" to access LLVM IR core types and builders. / 引入 "llvm/IR/IRBuilder.h" 以使用LLVM IR 核心类型与构造工具。
- **L20**: Includes "llvm/IR/Instruction.h" to access LLVM IR core types and builders. / 引入 "llvm/IR/Instruction.h" 以使用LLVM IR 核心类型与构造工具。

### Lines 21-40

```cpp
#include "llvm/IR/Module.h"
#include "llvm/InitializePasses.h"
#include "llvm/Pass.h"
#include "llvm/TargetParser/Triple.h"

using namespace llvm;

using OperandBundleDef = OperandBundleDefT<Value *>;

#define DEBUG_TYPE "cfguard"

STATISTIC(CFGuardCounter, "Number of Control Flow Guard checks added");

constexpr StringRef GuardCheckFunctionName = "__guard_check_icall_fptr";
constexpr StringRef GuardDispatchFunctionName = "__guard_dispatch_icall_fptr";

namespace {

/// Adds Control Flow Guard (CFG) checks on indirect function calls/invokes.
/// These checks ensure that the target address corresponds to the start of an
```

- **L21**: Includes "llvm/IR/Module.h" to access LLVM IR core types and builders. / 引入 "llvm/IR/Module.h" 以使用LLVM IR 核心类型与构造工具。
- **L22**: Includes "llvm/InitializePasses.h" to access local declarations used by this file. / 引入 "llvm/InitializePasses.h" 以使用本文件使用的本地声明。
- **L23**: Includes "llvm/Pass.h" to access local declarations used by this file. / 引入 "llvm/Pass.h" 以使用本文件使用的本地声明。
- **L24**: Includes "llvm/TargetParser/Triple.h" to access local declarations used by this file. / 引入 "llvm/TargetParser/Triple.h" 以使用本文件使用的本地声明。
- **L25**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L26**: Brings namespace `llvm` into the local scope. / 将命名空间 `llvm` 引入当前作用域。
- **L27**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L28**: Defines type or value alias `OperandBundleDef`. / 定义类型或数值别名 `OperandBundleDef`。
- **L29**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L30**: Defines macro `DEBUG_TYPE` for later conditional logic, flags, or diagnostics. / 定义宏 `DEBUG_TYPE`，供后续条件逻辑、标志位或诊断使用。
- **L31**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L32**: Registers LLVM statistic counter `CFGuardCounter`. / 注册 LLVM 统计计数器 `CFGuardCounter`。
- **L33**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L34**: Initializes variable `GuardCheckFunctionName` from the right-hand expression. / 使用右侧表达式初始化变量 `GuardCheckFunctionName`。
- **L35**: Initializes variable `GuardDispatchFunctionName` from the right-hand expression. / 使用右侧表达式初始化变量 `GuardDispatchFunctionName`。
- **L36**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L37**: Opens namespace scope ``. / 打开命名空间作用域 ``。
- **L38**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L39**: Comment documents the nearby logic or transformation intent: `Adds Control Flow Guard (CFG) checks on indirect function calls/invokes.`. / 注释说明了附近代码的逻辑或变换意图：`Adds Control Flow Guard (CFG) checks on indirect function calls/invokes.`。
- **L40**: Comment documents the nearby logic or transformation intent: `These checks ensure that the target address corresponds to the start of an`. / 注释说明了附近代码的逻辑或变换意图：`These checks ensure that the target address corresponds to the start of an`。

### Lines 41-60

```cpp
/// address-taken function.
class CFGuardImpl {
public:
  using Mechanism = CFGuardPass::Mechanism;

  /// Inserts a Control Flow Guard (CFG) check on an indirect call using the CFG
  /// check mechanism. When the image is loaded, the loader puts the appropriate
  /// guard check function pointer in the __guard_check_icall_fptr global
  /// symbol. This checks that the target address is a valid address-taken
  /// function. The address of the target function is passed to the guard check
  /// function in an architecture-specific register (e.g. ECX on 32-bit X86,
  /// X15 on Aarch64, and R0 on ARM). The guard check function has no return
  /// value (if the target is invalid, the guard check funtion will raise an
  /// error).
  ///
  /// For example, the following LLVM IR:
  /// \code
  ///   %func_ptr = alloca i32 ()*, align 8
  ///   store i32 ()* @target_func, i32 ()** %func_ptr, align 8
  ///   %0 = load i32 ()*, i32 ()** %func_ptr, align 8
```

- **L41**: Comment documents the nearby logic or transformation intent: `address-taken function.`. / 注释说明了附近代码的逻辑或变换意图：`address-taken function.`。
- **L42**: Declares class `CFGuardImpl`. / 声明 class `CFGuardImpl`。
- **L43**: Sets the following members to `public` access. / 将后续成员的访问级别设为 `public`。
- **L44**: Defines type or value alias `Mechanism`. / 定义类型或数值别名 `Mechanism`。
- **L45**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L46**: Comment documents the nearby logic or transformation intent: `Inserts a Control Flow Guard (CFG) check on an indirect call using the CFG`. / 注释说明了附近代码的逻辑或变换意图：`Inserts a Control Flow Guard (CFG) check on an indirect call using the CFG`。
- **L47**: Comment documents the nearby logic or transformation intent: `check mechanism. When the image is loaded, the loader puts the appropriate`. / 注释说明了附近代码的逻辑或变换意图：`check mechanism. When the image is loaded, the loader puts the appropriate`。
- **L48**: Comment documents the nearby logic or transformation intent: `guard check function pointer in the __guard_check_icall_fptr global`. / 注释说明了附近代码的逻辑或变换意图：`guard check function pointer in the __guard_check_icall_fptr global`。
- **L49**: Comment documents the nearby logic or transformation intent: `symbol. This checks that the target address is a valid address-taken`. / 注释说明了附近代码的逻辑或变换意图：`symbol. This checks that the target address is a valid address-taken`。
- **L50**: Comment documents the nearby logic or transformation intent: `function. The address of the target function is passed to the guard check`. / 注释说明了附近代码的逻辑或变换意图：`function. The address of the target function is passed to the guard check`。
- **L51**: Comment documents the nearby logic or transformation intent: `function in an architecture-specific register (e.g. ECX on 32-bit X86,`. / 注释说明了附近代码的逻辑或变换意图：`function in an architecture-specific register (e.g. ECX on 32-bit X86,`。
- **L52**: Comment documents the nearby logic or transformation intent: `X15 on Aarch64, and R0 on ARM). The guard check function has no return`. / 注释说明了附近代码的逻辑或变换意图：`X15 on Aarch64, and R0 on ARM). The guard check function has no return`。
- **L53**: Comment documents the nearby logic or transformation intent: `value (if the target is invalid, the guard check funtion will raise an`. / 注释说明了附近代码的逻辑或变换意图：`value (if the target is invalid, the guard check funtion will raise an`。
- **L54**: Comment documents the nearby logic or transformation intent: `error).`. / 注释说明了附近代码的逻辑或变换意图：`error).`。
- **L55**: Separator comment used to visually break up sections. / 分隔性注释，用于在视觉上划分小节。
- **L56**: Comment documents the nearby logic or transformation intent: `For example, the following LLVM IR:`. / 注释说明了附近代码的逻辑或变换意图：`For example, the following LLVM IR:`。
- **L57**: Comment documents the nearby logic or transformation intent: `\code`. / 注释说明了附近代码的逻辑或变换意图：`\code`。
- **L58**: Comment documents the nearby logic or transformation intent: `%func_ptr = alloca i32 ()*, align 8`. / 注释说明了附近代码的逻辑或变换意图：`%func_ptr = alloca i32 ()*, align 8`。
- **L59**: Comment documents the nearby logic or transformation intent: `store i32 ()* @target_func, i32 ()** %func_ptr, align 8`. / 注释说明了附近代码的逻辑或变换意图：`store i32 ()* @target_func, i32 ()** %func_ptr, align 8`。
- **L60**: Comment documents the nearby logic or transformation intent: `%0 = load i32 ()*, i32 ()** %func_ptr, align 8`. / 注释说明了附近代码的逻辑或变换意图：`%0 = load i32 ()*, i32 ()** %func_ptr, align 8`。

### Lines 61-80

```cpp
  ///   %1 = call i32 %0()
  /// \endcode
  ///
  /// is transformed to:
  /// \code
  ///   %func_ptr = alloca i32 ()*, align 8
  ///   store i32 ()* @target_func, i32 ()** %func_ptr, align 8
  ///   %0 = load i32 ()*, i32 ()** %func_ptr, align 8
  ///   %1 = load void (i8*)*, void (i8*)** @__guard_check_icall_fptr
  ///   %2 = bitcast i32 ()* %0 to i8*
  ///   call cfguard_checkcc void %1(i8* %2)
  ///   %3 = call i32 %0()
  /// \endcode
  ///
  /// For example, the following X86 assembly code:
  /// \code
  ///   movl  $_target_func, %eax
  ///   calll *%eax
  /// \endcode
  ///
```

- **L61**: Comment documents the nearby logic or transformation intent: `%1 = call i32 %0()`. / 注释说明了附近代码的逻辑或变换意图：`%1 = call i32 %0()`。
- **L62**: Comment documents the nearby logic or transformation intent: `\endcode`. / 注释说明了附近代码的逻辑或变换意图：`\endcode`。
- **L63**: Separator comment used to visually break up sections. / 分隔性注释，用于在视觉上划分小节。
- **L64**: Comment documents the nearby logic or transformation intent: `is transformed to:`. / 注释说明了附近代码的逻辑或变换意图：`is transformed to:`。
- **L65**: Comment documents the nearby logic or transformation intent: `\code`. / 注释说明了附近代码的逻辑或变换意图：`\code`。
- **L66**: Comment documents the nearby logic or transformation intent: `%func_ptr = alloca i32 ()*, align 8`. / 注释说明了附近代码的逻辑或变换意图：`%func_ptr = alloca i32 ()*, align 8`。
- **L67**: Comment documents the nearby logic or transformation intent: `store i32 ()* @target_func, i32 ()** %func_ptr, align 8`. / 注释说明了附近代码的逻辑或变换意图：`store i32 ()* @target_func, i32 ()** %func_ptr, align 8`。
- **L68**: Comment documents the nearby logic or transformation intent: `%0 = load i32 ()*, i32 ()** %func_ptr, align 8`. / 注释说明了附近代码的逻辑或变换意图：`%0 = load i32 ()*, i32 ()** %func_ptr, align 8`。
- **L69**: Comment documents the nearby logic or transformation intent: `%1 = load void (i8*)*, void (i8*)** @__guard_check_icall_fptr`. / 注释说明了附近代码的逻辑或变换意图：`%1 = load void (i8*)*, void (i8*)** @__guard_check_icall_fptr`。
- **L70**: Comment documents the nearby logic or transformation intent: `%2 = bitcast i32 ()* %0 to i8*`. / 注释说明了附近代码的逻辑或变换意图：`%2 = bitcast i32 ()* %0 to i8*`。
- **L71**: Comment documents the nearby logic or transformation intent: `call cfguard_checkcc void %1(i8* %2)`. / 注释说明了附近代码的逻辑或变换意图：`call cfguard_checkcc void %1(i8* %2)`。
- **L72**: Comment documents the nearby logic or transformation intent: `%3 = call i32 %0()`. / 注释说明了附近代码的逻辑或变换意图：`%3 = call i32 %0()`。
- **L73**: Comment documents the nearby logic or transformation intent: `\endcode`. / 注释说明了附近代码的逻辑或变换意图：`\endcode`。
- **L74**: Separator comment used to visually break up sections. / 分隔性注释，用于在视觉上划分小节。
- **L75**: Comment documents the nearby logic or transformation intent: `For example, the following X86 assembly code:`. / 注释说明了附近代码的逻辑或变换意图：`For example, the following X86 assembly code:`。
- **L76**: Comment documents the nearby logic or transformation intent: `\code`. / 注释说明了附近代码的逻辑或变换意图：`\code`。
- **L77**: Comment documents the nearby logic or transformation intent: `movl  $_target_func, %eax`. / 注释说明了附近代码的逻辑或变换意图：`movl  $_target_func, %eax`。
- **L78**: Comment documents the nearby logic or transformation intent: `calll *%eax`. / 注释说明了附近代码的逻辑或变换意图：`calll *%eax`。
- **L79**: Comment documents the nearby logic or transformation intent: `\endcode`. / 注释说明了附近代码的逻辑或变换意图：`\endcode`。
- **L80**: Separator comment used to visually break up sections. / 分隔性注释，用于在视觉上划分小节。

### Lines 81-100

```cpp
  /// is transformed to:
  /// \code
  /// 	movl	$_target_func, %ecx
  /// 	calll	*___guard_check_icall_fptr
  /// 	calll	*%ecx
  /// \endcode
  ///
  /// \param CB indirect call to instrument.
  void insertCFGuardCheck(CallBase *CB);

  /// Inserts a Control Flow Guard (CFG) check on an indirect call using the CFG
  /// dispatch mechanism. When the image is loaded, the loader puts the
  /// appropriate guard check function pointer in the
  /// __guard_dispatch_icall_fptr global symbol. This checks that the target
  /// address is a valid address-taken function and, if so, tail calls the
  /// target. The target address is passed in an architecture-specific register
  /// (e.g. RAX on X86_64), with all other arguments for the target function
  /// passed as usual.
  ///
  /// For example, the following LLVM IR:
```

- **L81**: Comment documents the nearby logic or transformation intent: `is transformed to:`. / 注释说明了附近代码的逻辑或变换意图：`is transformed to:`。
- **L82**: Comment documents the nearby logic or transformation intent: `\code`. / 注释说明了附近代码的逻辑或变换意图：`\code`。
- **L83**: Comment documents the nearby logic or transformation intent: `movl	$_target_func, %ecx`. / 注释说明了附近代码的逻辑或变换意图：`movl	$_target_func, %ecx`。
- **L84**: Comment documents the nearby logic or transformation intent: `calll	*___guard_check_icall_fptr`. / 注释说明了附近代码的逻辑或变换意图：`calll	*___guard_check_icall_fptr`。
- **L85**: Comment documents the nearby logic or transformation intent: `calll	*%ecx`. / 注释说明了附近代码的逻辑或变换意图：`calll	*%ecx`。
- **L86**: Comment documents the nearby logic or transformation intent: `\endcode`. / 注释说明了附近代码的逻辑或变换意图：`\endcode`。
- **L87**: Separator comment used to visually break up sections. / 分隔性注释，用于在视觉上划分小节。
- **L88**: Comment documents the nearby logic or transformation intent: `\param CB indirect call to instrument.`. / 注释说明了附近代码的逻辑或变换意图：`\param CB indirect call to instrument.`。
- **L89**: Executes call or statement centered on `insertCFGuardCheck`. / 执行以 `insertCFGuardCheck` 为核心的调用或语句。
- **L90**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L91**: Comment documents the nearby logic or transformation intent: `Inserts a Control Flow Guard (CFG) check on an indirect call using the CFG`. / 注释说明了附近代码的逻辑或变换意图：`Inserts a Control Flow Guard (CFG) check on an indirect call using the CFG`。
- **L92**: Comment documents the nearby logic or transformation intent: `dispatch mechanism. When the image is loaded, the loader puts the`. / 注释说明了附近代码的逻辑或变换意图：`dispatch mechanism. When the image is loaded, the loader puts the`。
- **L93**: Comment documents the nearby logic or transformation intent: `appropriate guard check function pointer in the`. / 注释说明了附近代码的逻辑或变换意图：`appropriate guard check function pointer in the`。
- **L94**: Comment documents the nearby logic or transformation intent: `__guard_dispatch_icall_fptr global symbol. This checks that the target`. / 注释说明了附近代码的逻辑或变换意图：`__guard_dispatch_icall_fptr global symbol. This checks that the target`。
- **L95**: Comment documents the nearby logic or transformation intent: `address is a valid address-taken function and, if so, tail calls the`. / 注释说明了附近代码的逻辑或变换意图：`address is a valid address-taken function and, if so, tail calls the`。
- **L96**: Comment documents the nearby logic or transformation intent: `target. The target address is passed in an architecture-specific register`. / 注释说明了附近代码的逻辑或变换意图：`target. The target address is passed in an architecture-specific register`。
- **L97**: Comment documents the nearby logic or transformation intent: `(e.g. RAX on X86_64), with all other arguments for the target function`. / 注释说明了附近代码的逻辑或变换意图：`(e.g. RAX on X86_64), with all other arguments for the target function`。
- **L98**: Comment documents the nearby logic or transformation intent: `passed as usual.`. / 注释说明了附近代码的逻辑或变换意图：`passed as usual.`。
- **L99**: Separator comment used to visually break up sections. / 分隔性注释，用于在视觉上划分小节。
- **L100**: Comment documents the nearby logic or transformation intent: `For example, the following LLVM IR:`. / 注释说明了附近代码的逻辑或变换意图：`For example, the following LLVM IR:`。

### Lines 101-120

```cpp
  /// \code
  ///   %func_ptr = alloca i32 ()*, align 8
  ///   store i32 ()* @target_func, i32 ()** %func_ptr, align 8
  ///   %0 = load i32 ()*, i32 ()** %func_ptr, align 8
  ///   %1 = call i32 %0()
  /// \endcode
  ///
  /// is transformed to:
  /// \code
  ///   %func_ptr = alloca i32 ()*, align 8
  ///   store i32 ()* @target_func, i32 ()** %func_ptr, align 8
  ///   %0 = load i32 ()*, i32 ()** %func_ptr, align 8
  ///   %1 = load i32 ()*, i32 ()** @__guard_dispatch_icall_fptr
  ///   %2 = call i32 %1() [ "cfguardtarget"(i32 ()* %0) ]
  /// \endcode
  ///
  /// For example, the following X86_64 assembly code:
  /// \code
  ///   leaq   target_func(%rip), %rax
  ///	  callq  *%rax
```

- **L101**: Comment documents the nearby logic or transformation intent: `\code`. / 注释说明了附近代码的逻辑或变换意图：`\code`。
- **L102**: Comment documents the nearby logic or transformation intent: `%func_ptr = alloca i32 ()*, align 8`. / 注释说明了附近代码的逻辑或变换意图：`%func_ptr = alloca i32 ()*, align 8`。
- **L103**: Comment documents the nearby logic or transformation intent: `store i32 ()* @target_func, i32 ()** %func_ptr, align 8`. / 注释说明了附近代码的逻辑或变换意图：`store i32 ()* @target_func, i32 ()** %func_ptr, align 8`。
- **L104**: Comment documents the nearby logic or transformation intent: `%0 = load i32 ()*, i32 ()** %func_ptr, align 8`. / 注释说明了附近代码的逻辑或变换意图：`%0 = load i32 ()*, i32 ()** %func_ptr, align 8`。
- **L105**: Comment documents the nearby logic or transformation intent: `%1 = call i32 %0()`. / 注释说明了附近代码的逻辑或变换意图：`%1 = call i32 %0()`。
- **L106**: Comment documents the nearby logic or transformation intent: `\endcode`. / 注释说明了附近代码的逻辑或变换意图：`\endcode`。
- **L107**: Separator comment used to visually break up sections. / 分隔性注释，用于在视觉上划分小节。
- **L108**: Comment documents the nearby logic or transformation intent: `is transformed to:`. / 注释说明了附近代码的逻辑或变换意图：`is transformed to:`。
- **L109**: Comment documents the nearby logic or transformation intent: `\code`. / 注释说明了附近代码的逻辑或变换意图：`\code`。
- **L110**: Comment documents the nearby logic or transformation intent: `%func_ptr = alloca i32 ()*, align 8`. / 注释说明了附近代码的逻辑或变换意图：`%func_ptr = alloca i32 ()*, align 8`。
- **L111**: Comment documents the nearby logic or transformation intent: `store i32 ()* @target_func, i32 ()** %func_ptr, align 8`. / 注释说明了附近代码的逻辑或变换意图：`store i32 ()* @target_func, i32 ()** %func_ptr, align 8`。
- **L112**: Comment documents the nearby logic or transformation intent: `%0 = load i32 ()*, i32 ()** %func_ptr, align 8`. / 注释说明了附近代码的逻辑或变换意图：`%0 = load i32 ()*, i32 ()** %func_ptr, align 8`。
- **L113**: Comment documents the nearby logic or transformation intent: `%1 = load i32 ()*, i32 ()** @__guard_dispatch_icall_fptr`. / 注释说明了附近代码的逻辑或变换意图：`%1 = load i32 ()*, i32 ()** @__guard_dispatch_icall_fptr`。
- **L114**: Comment documents the nearby logic or transformation intent: `%2 = call i32 %1() [ "cfguardtarget"(i32 ()* %0) ]`. / 注释说明了附近代码的逻辑或变换意图：`%2 = call i32 %1() [ "cfguardtarget"(i32 ()* %0) ]`。
- **L115**: Comment documents the nearby logic or transformation intent: `\endcode`. / 注释说明了附近代码的逻辑或变换意图：`\endcode`。
- **L116**: Separator comment used to visually break up sections. / 分隔性注释，用于在视觉上划分小节。
- **L117**: Comment documents the nearby logic or transformation intent: `For example, the following X86_64 assembly code:`. / 注释说明了附近代码的逻辑或变换意图：`For example, the following X86_64 assembly code:`。
- **L118**: Comment documents the nearby logic or transformation intent: `\code`. / 注释说明了附近代码的逻辑或变换意图：`\code`。
- **L119**: Comment documents the nearby logic or transformation intent: `leaq   target_func(%rip), %rax`. / 注释说明了附近代码的逻辑或变换意图：`leaq   target_func(%rip), %rax`。
- **L120**: Comment documents the nearby logic or transformation intent: `callq  *%rax`. / 注释说明了附近代码的逻辑或变换意图：`callq  *%rax`。

### Lines 121-140

```cpp
  /// \endcode
  ///
  /// is transformed to:
  /// \code
  ///   leaq   target_func(%rip), %rax
  ///   callq  *__guard_dispatch_icall_fptr(%rip)
  /// \endcode
  ///
  /// \param CB indirect call to instrument.
  void insertCFGuardDispatch(CallBase *CB);

  bool doInitialization(Module &M);
  bool runOnFunction(Function &F);

private:
  // Only add checks if the module has them enabled.
  ControlFlowGuardMode CFGuardModuleFlag = ControlFlowGuardMode::Disabled;
  Mechanism GuardMechanism = Mechanism::Check;
  FunctionType *GuardFnType = nullptr;
  PointerType *GuardFnPtrType = nullptr;
```

- **L121**: Comment documents the nearby logic or transformation intent: `\endcode`. / 注释说明了附近代码的逻辑或变换意图：`\endcode`。
- **L122**: Separator comment used to visually break up sections. / 分隔性注释，用于在视觉上划分小节。
- **L123**: Comment documents the nearby logic or transformation intent: `is transformed to:`. / 注释说明了附近代码的逻辑或变换意图：`is transformed to:`。
- **L124**: Comment documents the nearby logic or transformation intent: `\code`. / 注释说明了附近代码的逻辑或变换意图：`\code`。
- **L125**: Comment documents the nearby logic or transformation intent: `leaq   target_func(%rip), %rax`. / 注释说明了附近代码的逻辑或变换意图：`leaq   target_func(%rip), %rax`。
- **L126**: Comment documents the nearby logic or transformation intent: `callq  *__guard_dispatch_icall_fptr(%rip)`. / 注释说明了附近代码的逻辑或变换意图：`callq  *__guard_dispatch_icall_fptr(%rip)`。
- **L127**: Comment documents the nearby logic or transformation intent: `\endcode`. / 注释说明了附近代码的逻辑或变换意图：`\endcode`。
- **L128**: Separator comment used to visually break up sections. / 分隔性注释，用于在视觉上划分小节。
- **L129**: Comment documents the nearby logic or transformation intent: `\param CB indirect call to instrument.`. / 注释说明了附近代码的逻辑或变换意图：`\param CB indirect call to instrument.`。
- **L130**: Executes call or statement centered on `insertCFGuardDispatch`. / 执行以 `insertCFGuardDispatch` 为核心的调用或语句。
- **L131**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L132**: Executes call or statement centered on `doInitialization`. / 执行以 `doInitialization` 为核心的调用或语句。
- **L133**: Executes call or statement centered on `runOnFunction`. / 执行以 `runOnFunction` 为核心的调用或语句。
- **L134**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L135**: Sets the following members to `private` access. / 将后续成员的访问级别设为 `private`。
- **L136**: Comment documents the nearby logic or transformation intent: `Only add checks if the module has them enabled.`. / 注释说明了附近代码的逻辑或变换意图：`Only add checks if the module has them enabled.`。
- **L137**: Initializes variable `CFGuardModuleFlag` from the right-hand expression. / 使用右侧表达式初始化变量 `CFGuardModuleFlag`。
- **L138**: Initializes variable `GuardMechanism` from the right-hand expression. / 使用右侧表达式初始化变量 `GuardMechanism`。
- **L139**: Executes a standalone statement or declaration: `FunctionType *GuardFnType = nullptr;`. / 执行一条独立语句或声明：`FunctionType *GuardFnType = nullptr;`。
- **L140**: Executes a standalone statement or declaration: `PointerType *GuardFnPtrType = nullptr;`. / 执行一条独立语句或声明：`PointerType *GuardFnPtrType = nullptr;`。

### Lines 141-160

```cpp
  Constant *GuardFnGlobal = nullptr;
};

class CFGuard : public FunctionPass {
  CFGuardImpl Impl;

public:
  static char ID;

  // Default constructor required for the INITIALIZE_PASS macro.
  CFGuard() : FunctionPass(ID) {}

  bool doInitialization(Module &M) override { return Impl.doInitialization(M); }
  bool runOnFunction(Function &F) override { return Impl.runOnFunction(F); }
};

} // end anonymous namespace

void CFGuardImpl::insertCFGuardCheck(CallBase *CB) {
  assert(CB->getModule()->getTargetTriple().isOSWindows() &&
```

- **L141**: Executes a standalone statement or declaration: `Constant *GuardFnGlobal = nullptr;`. / 执行一条独立语句或声明：`Constant *GuardFnGlobal = nullptr;`。
- **L142**: Closes the current declaration scope, such as a class or struct. / 结束当前声明作用域，例如类或结构体。
- **L143**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L144**: Declares class `CFGuard`. / 声明 class `CFGuard`。
- **L145**: Executes a standalone statement or declaration: `CFGuardImpl Impl;`. / 执行一条独立语句或声明：`CFGuardImpl Impl;`。
- **L146**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L147**: Sets the following members to `public` access. / 将后续成员的访问级别设为 `public`。
- **L148**: Executes a standalone statement or declaration: `static char ID;`. / 执行一条独立语句或声明：`static char ID;`。
- **L149**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L150**: Comment documents the nearby logic or transformation intent: `Default constructor required for the INITIALIZE_PASS macro.`. / 注释说明了附近代码的逻辑或变换意图：`Default constructor required for the INITIALIZE_PASS macro.`。
- **L151**: Continues the surrounding expression or declaration: `CFGuard() : FunctionPass(ID) {}`. / 继续构造周围的表达式或声明：`CFGuard() : FunctionPass(ID) {}`。
- **L152**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L153**: Continues the surrounding expression or declaration: `bool doInitialization(Module &M) override { return Impl.doInitialization(M); }`. / 继续构造周围的表达式或声明：`bool doInitialization(Module &M) override { return Impl.doInitialization(M); }`。
- **L154**: Continues the surrounding expression or declaration: `bool runOnFunction(Function &F) override { return Impl.runOnFunction(F); }`. / 继续构造周围的表达式或声明：`bool runOnFunction(Function &F) override { return Impl.runOnFunction(F); }`。
- **L155**: Closes the current declaration scope, such as a class or struct. / 结束当前声明作用域，例如类或结构体。
- **L156**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L157**: Continues the surrounding expression or declaration: `} // end anonymous namespace`. / 继续构造周围的表达式或声明：`} // end anonymous namespace`。
- **L158**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L159**: Starts a function, method, or lambda body: `void CFGuardImpl::insertCFGuardCheck(CallBase *CB) {`. / 开始一个函数、方法或 lambda 的主体：`void CFGuardImpl::insertCFGuardCheck(CallBase *CB) {`。
- **L160**: Checks an internal invariant in debug builds. / 在调试构建中检查内部不变式。

### Lines 161-180

```cpp
         "Only applicable for Windows targets");
  assert(CB->isIndirectCall() &&
         "Control Flow Guard checks can only be added to indirect calls");

  IRBuilder<> B(CB);
  Value *CalledOperand = CB->getCalledOperand();

  // If the indirect call is called within catchpad or cleanuppad,
  // we need to copy "funclet" bundle of the call.
  SmallVector<llvm::OperandBundleDef, 1> Bundles;
  if (auto Bundle = CB->getOperandBundle(LLVMContext::OB_funclet))
    Bundles.push_back(OperandBundleDef(*Bundle));

  // Load the global symbol as a pointer to the check function.
  LoadInst *GuardCheckLoad = B.CreateLoad(GuardFnPtrType, GuardFnGlobal);

  // Create new call instruction. The CFGuard check should always be a call,
  // even if the original CallBase is an Invoke or CallBr instruction.
  CallInst *GuardCheck =
      B.CreateCall(GuardFnType, GuardCheckLoad, {CalledOperand}, Bundles);
```

- **L161**: Executes a standalone statement or declaration: `"Only applicable for Windows targets");`. / 执行一条独立语句或声明：`"Only applicable for Windows targets");`。
- **L162**: Checks an internal invariant in debug builds. / 在调试构建中检查内部不变式。
- **L163**: Executes a standalone statement or declaration: `"Control Flow Guard checks can only be added to indirect calls");`. / 执行一条独立语句或声明：`"Control Flow Guard checks can only be added to indirect calls");`。
- **L164**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L165**: Executes call or statement centered on `B`. / 执行以 `B` 为核心的调用或语句。
- **L166**: Executes call or statement centered on `CB->getCalledOperand`. / 执行以 `CB->getCalledOperand` 为核心的调用或语句。
- **L167**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L168**: Comment documents the nearby logic or transformation intent: `If the indirect call is called within catchpad or cleanuppad,`. / 注释说明了附近代码的逻辑或变换意图：`If the indirect call is called within catchpad or cleanuppad,`。
- **L169**: Comment documents the nearby logic or transformation intent: `we need to copy "funclet" bundle of the call.`. / 注释说明了附近代码的逻辑或变换意图：`we need to copy "funclet" bundle of the call.`。
- **L170**: Executes a standalone statement or declaration: `SmallVector<llvm::OperandBundleDef, 1> Bundles;`. / 执行一条独立语句或声明：`SmallVector<llvm::OperandBundleDef, 1> Bundles;`。
- **L171**: Begins a `if` control-flow statement and evaluates its condition. / 开始 `if` 控制流语句并计算其条件。
- **L172**: Executes call or statement centered on `Bundles.push_back`. / 执行以 `Bundles.push_back` 为核心的调用或语句。
- **L173**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L174**: Comment documents the nearby logic or transformation intent: `Load the global symbol as a pointer to the check function.`. / 注释说明了附近代码的逻辑或变换意图：`Load the global symbol as a pointer to the check function.`。
- **L175**: Executes call or statement centered on `B.CreateLoad`. / 执行以 `B.CreateLoad` 为核心的调用或语句。
- **L176**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L177**: Comment documents the nearby logic or transformation intent: `Create new call instruction. The CFGuard check should always be a call,`. / 注释说明了附近代码的逻辑或变换意图：`Create new call instruction. The CFGuard check should always be a call,`。
- **L178**: Comment documents the nearby logic or transformation intent: `even if the original CallBase is an Invoke or CallBr instruction.`. / 注释说明了附近代码的逻辑或变换意图：`even if the original CallBase is an Invoke or CallBr instruction.`。
- **L179**: Continues the surrounding expression or declaration: `CallInst *GuardCheck =`. / 继续构造周围的表达式或声明：`CallInst *GuardCheck =`。
- **L180**: Executes call or statement centered on `B.CreateCall`. / 执行以 `B.CreateCall` 为核心的调用或语句。

### Lines 181-200

```cpp

  // Ensure that the first argument is passed in the correct register
  // (e.g. ECX on 32-bit X86 targets).
  GuardCheck->setCallingConv(CallingConv::CFGuard_Check);
}

void CFGuardImpl::insertCFGuardDispatch(CallBase *CB) {
  assert(CB->getModule()->getTargetTriple().isOSWindows() &&
         "Only applicable for Windows targets");
  assert(CB->isIndirectCall() &&
         "Control Flow Guard checks can only be added to indirect calls");

  IRBuilder<> B(CB);
  Value *CalledOperand = CB->getCalledOperand();
  Type *CalledOperandType = CalledOperand->getType();

  // Load the global as a pointer to a function of the same type.
  LoadInst *GuardDispatchLoad = B.CreateLoad(CalledOperandType, GuardFnGlobal);

  // Add the original call target as a cfguardtarget operand bundle.
```

- **L181**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L182**: Comment documents the nearby logic or transformation intent: `Ensure that the first argument is passed in the correct register`. / 注释说明了附近代码的逻辑或变换意图：`Ensure that the first argument is passed in the correct register`。
- **L183**: Comment documents the nearby logic or transformation intent: `(e.g. ECX on 32-bit X86 targets).`. / 注释说明了附近代码的逻辑或变换意图：`(e.g. ECX on 32-bit X86 targets).`。
- **L184**: Executes call or statement centered on `GuardCheck->setCallingConv`. / 执行以 `GuardCheck->setCallingConv` 为核心的调用或语句。
- **L185**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L186**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L187**: Starts a function, method, or lambda body: `void CFGuardImpl::insertCFGuardDispatch(CallBase *CB) {`. / 开始一个函数、方法或 lambda 的主体：`void CFGuardImpl::insertCFGuardDispatch(CallBase *CB) {`。
- **L188**: Checks an internal invariant in debug builds. / 在调试构建中检查内部不变式。
- **L189**: Executes a standalone statement or declaration: `"Only applicable for Windows targets");`. / 执行一条独立语句或声明：`"Only applicable for Windows targets");`。
- **L190**: Checks an internal invariant in debug builds. / 在调试构建中检查内部不变式。
- **L191**: Executes a standalone statement or declaration: `"Control Flow Guard checks can only be added to indirect calls");`. / 执行一条独立语句或声明：`"Control Flow Guard checks can only be added to indirect calls");`。
- **L192**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L193**: Executes call or statement centered on `B`. / 执行以 `B` 为核心的调用或语句。
- **L194**: Executes call or statement centered on `CB->getCalledOperand`. / 执行以 `CB->getCalledOperand` 为核心的调用或语句。
- **L195**: Executes call or statement centered on `CalledOperand->getType`. / 执行以 `CalledOperand->getType` 为核心的调用或语句。
- **L196**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L197**: Comment documents the nearby logic or transformation intent: `Load the global as a pointer to a function of the same type.`. / 注释说明了附近代码的逻辑或变换意图：`Load the global as a pointer to a function of the same type.`。
- **L198**: Executes call or statement centered on `B.CreateLoad`. / 执行以 `B.CreateLoad` 为核心的调用或语句。
- **L199**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L200**: Comment documents the nearby logic or transformation intent: `Add the original call target as a cfguardtarget operand bundle.`. / 注释说明了附近代码的逻辑或变换意图：`Add the original call target as a cfguardtarget operand bundle.`。

### Lines 201-220

```cpp
  SmallVector<llvm::OperandBundleDef, 1> Bundles;
  CB->getOperandBundlesAsDefs(Bundles);
  Bundles.emplace_back("cfguardtarget", CalledOperand);

  // Create a copy of the call/invoke instruction and add the new bundle.
  assert((isa<CallInst>(CB) || isa<InvokeInst>(CB)) &&
         "Unknown indirect call type");
  CallBase *NewCB = CallBase::Create(CB, Bundles, CB->getIterator());

  // Change the target of the call to be the guard dispatch function.
  NewCB->setCalledOperand(GuardDispatchLoad);

  // Replace the original call/invoke with the new instruction.
  CB->replaceAllUsesWith(NewCB);

  // Delete the original call/invoke.
  CB->eraseFromParent();
}

bool CFGuardImpl::doInitialization(Module &M) {
```

- **L201**: Executes a standalone statement or declaration: `SmallVector<llvm::OperandBundleDef, 1> Bundles;`. / 执行一条独立语句或声明：`SmallVector<llvm::OperandBundleDef, 1> Bundles;`。
- **L202**: Executes call or statement centered on `CB->getOperandBundlesAsDefs`. / 执行以 `CB->getOperandBundlesAsDefs` 为核心的调用或语句。
- **L203**: Executes call or statement centered on `Bundles.emplace_back`. / 执行以 `Bundles.emplace_back` 为核心的调用或语句。
- **L204**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L205**: Comment documents the nearby logic or transformation intent: `Create a copy of the call/invoke instruction and add the new bundle.`. / 注释说明了附近代码的逻辑或变换意图：`Create a copy of the call/invoke instruction and add the new bundle.`。
- **L206**: Checks an internal invariant in debug builds. / 在调试构建中检查内部不变式。
- **L207**: Executes a standalone statement or declaration: `"Unknown indirect call type");`. / 执行一条独立语句或声明：`"Unknown indirect call type");`。
- **L208**: Executes call or statement centered on `CallBase::Create`. / 执行以 `CallBase::Create` 为核心的调用或语句。
- **L209**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L210**: Comment documents the nearby logic or transformation intent: `Change the target of the call to be the guard dispatch function.`. / 注释说明了附近代码的逻辑或变换意图：`Change the target of the call to be the guard dispatch function.`。
- **L211**: Executes call or statement centered on `NewCB->setCalledOperand`. / 执行以 `NewCB->setCalledOperand` 为核心的调用或语句。
- **L212**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L213**: Comment documents the nearby logic or transformation intent: `Replace the original call/invoke with the new instruction.`. / 注释说明了附近代码的逻辑或变换意图：`Replace the original call/invoke with the new instruction.`。
- **L214**: Executes call or statement centered on `CB->replaceAllUsesWith`. / 执行以 `CB->replaceAllUsesWith` 为核心的调用或语句。
- **L215**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L216**: Comment documents the nearby logic or transformation intent: `Delete the original call/invoke.`. / 注释说明了附近代码的逻辑或变换意图：`Delete the original call/invoke.`。
- **L217**: Executes call or statement centered on `CB->eraseFromParent`. / 执行以 `CB->eraseFromParent` 为核心的调用或语句。
- **L218**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L219**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L220**: Starts a function, method, or lambda body: `bool CFGuardImpl::doInitialization(Module &M) {`. / 开始一个函数、方法或 lambda 的主体：`bool CFGuardImpl::doInitialization(Module &M) {`。

### Lines 221-240

```cpp
  // Check if this module has the cfguard flag and read its value.
  CFGuardModuleFlag = M.getControlFlowGuardMode();

  // Skip modules for which CFGuard checks have been disabled.
  if (CFGuardModuleFlag != ControlFlowGuardMode::Enabled)
    return false;

  // Determine the guard mechanism to use.
  ControlFlowGuardMechanism MechanismOverride =
      ControlFlowGuardMechanism::Automatic;
  if (auto *CI = mdconst::dyn_extract_or_null<ConstantInt>(
          M.getModuleFlag("cfguard-mechanism")))
    MechanismOverride =
        static_cast<ControlFlowGuardMechanism>(CI->getZExtValue());
  switch (MechanismOverride) {
  case ControlFlowGuardMechanism::Check:
    GuardMechanism = Mechanism::Check;
    break;
  case ControlFlowGuardMechanism::Dispatch:
    GuardMechanism = Mechanism::Dispatch;
```

- **L221**: Comment documents the nearby logic or transformation intent: `Check if this module has the cfguard flag and read its value.`. / 注释说明了附近代码的逻辑或变换意图：`Check if this module has the cfguard flag and read its value.`。
- **L222**: Executes call or statement centered on `M.getControlFlowGuardMode`. / 执行以 `M.getControlFlowGuardMode` 为核心的调用或语句。
- **L223**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L224**: Comment documents the nearby logic or transformation intent: `Skip modules for which CFGuard checks have been disabled.`. / 注释说明了附近代码的逻辑或变换意图：`Skip modules for which CFGuard checks have been disabled.`。
- **L225**: Begins a `if` control-flow statement and evaluates its condition. / 开始 `if` 控制流语句并计算其条件。
- **L226**: Returns from the current function with `false`. / 以 `false` 从当前函数返回。
- **L227**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L228**: Comment documents the nearby logic or transformation intent: `Determine the guard mechanism to use.`. / 注释说明了附近代码的逻辑或变换意图：`Determine the guard mechanism to use.`。
- **L229**: Continues the surrounding expression or declaration: `ControlFlowGuardMechanism MechanismOverride =`. / 继续构造周围的表达式或声明：`ControlFlowGuardMechanism MechanismOverride =`。
- **L230**: Executes a standalone statement or declaration: `ControlFlowGuardMechanism::Automatic;`. / 执行一条独立语句或声明：`ControlFlowGuardMechanism::Automatic;`。
- **L231**: Begins a `if` control-flow statement and evaluates its condition. / 开始 `if` 控制流语句并计算其条件。
- **L232**: Continues the surrounding expression or declaration: `M.getModuleFlag("cfguard-mechanism")))`. / 继续构造周围的表达式或声明：`M.getModuleFlag("cfguard-mechanism")))`。
- **L233**: Continues the surrounding expression or declaration: `MechanismOverride =`. / 继续构造周围的表达式或声明：`MechanismOverride =`。
- **L234**: Executes call or statement centered on `static_cast<ControlFlowGuardMechanism>`. / 执行以 `static_cast<ControlFlowGuardMechanism>` 为核心的调用或语句。
- **L235**: Begins a `switch` control-flow statement and evaluates its condition. / 开始 `switch` 控制流语句并计算其条件。
- **L236**: Introduces a switch dispatch label: `case ControlFlowGuardMechanism::Check:`. / 引入一个 switch 分发标签：`case ControlFlowGuardMechanism::Check:`。
- **L237**: Executes a standalone statement or declaration: `GuardMechanism = Mechanism::Check;`. / 执行一条独立语句或声明：`GuardMechanism = Mechanism::Check;`。
- **L238**: Exits the nearest loop or switch statement. / 退出最近的循环或 switch 语句。
- **L239**: Introduces a switch dispatch label: `case ControlFlowGuardMechanism::Dispatch:`. / 引入一个 switch 分发标签：`case ControlFlowGuardMechanism::Dispatch:`。
- **L240**: Executes a standalone statement or declaration: `GuardMechanism = Mechanism::Dispatch;`. / 执行一条独立语句或声明：`GuardMechanism = Mechanism::Dispatch;`。

### Lines 241-260

```cpp
    break;
  default:
    // X86_64 uses dispatch; all other architectures use check.
    GuardMechanism =
        M.getTargetTriple().isX86_64() ? Mechanism::Dispatch : Mechanism::Check;
    break;
  }

  // Set up prototypes for the guard check and dispatch functions.
  GuardFnType =
      FunctionType::get(Type::getVoidTy(M.getContext()),
                        {PointerType::getUnqual(M.getContext())}, false);
  GuardFnPtrType = PointerType::get(M.getContext(), 0);

  StringRef GuardFnName = GuardMechanism == Mechanism::Check
                              ? GuardCheckFunctionName
                              : GuardDispatchFunctionName;
  GuardFnGlobal = M.getOrInsertGlobal(GuardFnName, GuardFnPtrType, [&] {
    auto *Var = new GlobalVariable(M, GuardFnPtrType, false,
                                   GlobalVariable::ExternalLinkage, nullptr,
```

- **L241**: Exits the nearest loop or switch statement. / 退出最近的循环或 switch 语句。
- **L242**: Introduces a switch dispatch label: `default:`. / 引入一个 switch 分发标签：`default:`。
- **L243**: Comment documents the nearby logic or transformation intent: `X86_64 uses dispatch; all other architectures use check.`. / 注释说明了附近代码的逻辑或变换意图：`X86_64 uses dispatch; all other architectures use check.`。
- **L244**: Continues the surrounding expression or declaration: `GuardMechanism =`. / 继续构造周围的表达式或声明：`GuardMechanism =`。
- **L245**: Executes call or statement centered on `M.getTargetTriple`. / 执行以 `M.getTargetTriple` 为核心的调用或语句。
- **L246**: Exits the nearest loop or switch statement. / 退出最近的循环或 switch 语句。
- **L247**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L248**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L249**: Comment documents the nearby logic or transformation intent: `Set up prototypes for the guard check and dispatch functions.`. / 注释说明了附近代码的逻辑或变换意图：`Set up prototypes for the guard check and dispatch functions.`。
- **L250**: Continues the surrounding expression or declaration: `GuardFnType =`. / 继续构造周围的表达式或声明：`GuardFnType =`。
- **L251**: Continues a multi-line argument list or initializer: `FunctionType::get(Type::getVoidTy(M.getContext()),`. / 继续一个多行参数列表或初始化器：`FunctionType::get(Type::getVoidTy(M.getContext()),`。
- **L252**: Executes call or statement centered on `{PointerType::getUnqual`. / 执行以 `{PointerType::getUnqual` 为核心的调用或语句。
- **L253**: Executes call or statement centered on `PointerType::get`. / 执行以 `PointerType::get` 为核心的调用或语句。
- **L254**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L255**: Continues the surrounding expression or declaration: `StringRef GuardFnName = GuardMechanism == Mechanism::Check`. / 继续构造周围的表达式或声明：`StringRef GuardFnName = GuardMechanism == Mechanism::Check`。
- **L256**: Continues the surrounding expression or declaration: `? GuardCheckFunctionName`. / 继续构造周围的表达式或声明：`? GuardCheckFunctionName`。
- **L257**: Executes a standalone statement or declaration: `: GuardDispatchFunctionName;`. / 执行一条独立语句或声明：`: GuardDispatchFunctionName;`。
- **L258**: Starts a function, method, or lambda body: `GuardFnGlobal = M.getOrInsertGlobal(GuardFnName, GuardFnPtrType, [&] {`. / 开始一个函数、方法或 lambda 的主体：`GuardFnGlobal = M.getOrInsertGlobal(GuardFnName, GuardFnPtrType, [&] {`。
- **L259**: Continues a multi-line argument list or initializer: `auto *Var = new GlobalVariable(M, GuardFnPtrType, false,`. / 继续一个多行参数列表或初始化器：`auto *Var = new GlobalVariable(M, GuardFnPtrType, false,`。
- **L260**: Continues a multi-line argument list or initializer: `GlobalVariable::ExternalLinkage, nullptr,`. / 继续一个多行参数列表或初始化器：`GlobalVariable::ExternalLinkage, nullptr,`。

### Lines 261-280

```cpp
                                   GuardFnName);
    Var->setDSOLocal(true);
    return Var;
  });

  return true;
}

bool CFGuardImpl::runOnFunction(Function &F) {
  // Skip modules for which CFGuard checks have been disabled.
  if (CFGuardModuleFlag != ControlFlowGuardMode::Enabled)
    return false;

  SmallVector<CallBase *, 8> IndirectCalls;

  // Iterate over the instructions to find all indirect call/invoke/callbr
  // instructions. Make a separate list of pointers to indirect
  // call/invoke/callbr instructions because the original instructions will be
  // deleted as the checks are added.
  for (BasicBlock &BB : F) {
```

- **L261**: Executes a standalone statement or declaration: `GuardFnName);`. / 执行一条独立语句或声明：`GuardFnName);`。
- **L262**: Executes call or statement centered on `Var->setDSOLocal`. / 执行以 `Var->setDSOLocal` 为核心的调用或语句。
- **L263**: Returns from the current function with `Var`. / 以 `Var` 从当前函数返回。
- **L264**: Executes a standalone statement or declaration: `});`. / 执行一条独立语句或声明：`});`。
- **L265**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L266**: Returns from the current function with `true`. / 以 `true` 从当前函数返回。
- **L267**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L268**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L269**: Starts a function, method, or lambda body: `bool CFGuardImpl::runOnFunction(Function &F) {`. / 开始一个函数、方法或 lambda 的主体：`bool CFGuardImpl::runOnFunction(Function &F) {`。
- **L270**: Comment documents the nearby logic or transformation intent: `Skip modules for which CFGuard checks have been disabled.`. / 注释说明了附近代码的逻辑或变换意图：`Skip modules for which CFGuard checks have been disabled.`。
- **L271**: Begins a `if` control-flow statement and evaluates its condition. / 开始 `if` 控制流语句并计算其条件。
- **L272**: Returns from the current function with `false`. / 以 `false` 从当前函数返回。
- **L273**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L274**: Executes a standalone statement or declaration: `SmallVector<CallBase *, 8> IndirectCalls;`. / 执行一条独立语句或声明：`SmallVector<CallBase *, 8> IndirectCalls;`。
- **L275**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L276**: Comment documents the nearby logic or transformation intent: `Iterate over the instructions to find all indirect call/invoke/callbr`. / 注释说明了附近代码的逻辑或变换意图：`Iterate over the instructions to find all indirect call/invoke/callbr`。
- **L277**: Comment documents the nearby logic or transformation intent: `instructions. Make a separate list of pointers to indirect`. / 注释说明了附近代码的逻辑或变换意图：`instructions. Make a separate list of pointers to indirect`。
- **L278**: Comment documents the nearby logic or transformation intent: `call/invoke/callbr instructions because the original instructions will be`. / 注释说明了附近代码的逻辑或变换意图：`call/invoke/callbr instructions because the original instructions will be`。
- **L279**: Comment documents the nearby logic or transformation intent: `deleted as the checks are added.`. / 注释说明了附近代码的逻辑或变换意图：`deleted as the checks are added.`。
- **L280**: Begins a `for` control-flow statement and evaluates its condition. / 开始 `for` 控制流语句并计算其条件。

### Lines 281-300

```cpp
    for (Instruction &I : BB) {
      auto *CB = dyn_cast<CallBase>(&I);
      if (CB && CB->isIndirectCall() && !CB->hasFnAttr("guard_nocf")) {
        IndirectCalls.push_back(CB);
        CFGuardCounter++;
      }
    }
  }

  // If no checks are needed, return early.
  if (IndirectCalls.empty())
    return false;

  // For each indirect call/invoke, add the appropriate dispatch or check.
  if (GuardMechanism == Mechanism::Dispatch) {
    for (CallBase *CB : IndirectCalls)
      insertCFGuardDispatch(CB);
  } else {
    for (CallBase *CB : IndirectCalls)
      insertCFGuardCheck(CB);
```

- **L281**: Begins a `for` control-flow statement and evaluates its condition. / 开始 `for` 控制流语句并计算其条件。
- **L282**: Executes call or statement centered on `dyn_cast<CallBase>`. / 执行以 `dyn_cast<CallBase>` 为核心的调用或语句。
- **L283**: Begins a `if` control-flow statement and evaluates its condition. / 开始 `if` 控制流语句并计算其条件。
- **L284**: Executes call or statement centered on `IndirectCalls.push_back`. / 执行以 `IndirectCalls.push_back` 为核心的调用或语句。
- **L285**: Executes a standalone statement or declaration: `CFGuardCounter++;`. / 执行一条独立语句或声明：`CFGuardCounter++;`。
- **L286**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L287**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L288**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L289**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L290**: Comment documents the nearby logic or transformation intent: `If no checks are needed, return early.`. / 注释说明了附近代码的逻辑或变换意图：`If no checks are needed, return early.`。
- **L291**: Begins a `if` control-flow statement and evaluates its condition. / 开始 `if` 控制流语句并计算其条件。
- **L292**: Returns from the current function with `false`. / 以 `false` 从当前函数返回。
- **L293**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L294**: Comment documents the nearby logic or transformation intent: `For each indirect call/invoke, add the appropriate dispatch or check.`. / 注释说明了附近代码的逻辑或变换意图：`For each indirect call/invoke, add the appropriate dispatch or check.`。
- **L295**: Begins a `if` control-flow statement and evaluates its condition. / 开始 `if` 控制流语句并计算其条件。
- **L296**: Begins a `for` control-flow statement and evaluates its condition. / 开始 `for` 控制流语句并计算其条件。
- **L297**: Executes call or statement centered on `insertCFGuardDispatch`. / 执行以 `insertCFGuardDispatch` 为核心的调用或语句。
- **L298**: Continues the surrounding expression or declaration: `} else {`. / 继续构造周围的表达式或声明：`} else {`。
- **L299**: Begins a `for` control-flow statement and evaluates its condition. / 开始 `for` 控制流语句并计算其条件。
- **L300**: Executes call or statement centered on `insertCFGuardCheck`. / 执行以 `insertCFGuardCheck` 为核心的调用或语句。

### Lines 301-320

```cpp
  }

  return true;
}

PreservedAnalyses CFGuardPass::run(Function &F, FunctionAnalysisManager &FAM) {
  CFGuardImpl Impl;
  bool Changed = Impl.doInitialization(*F.getParent());
  Changed |= Impl.runOnFunction(F);
  return Changed ? PreservedAnalyses::none() : PreservedAnalyses::all();
}

char CFGuard::ID = 0;
INITIALIZE_PASS(CFGuard, "CFGuard", "CFGuard", false, false)

FunctionPass *llvm::createCFGuardPass() { return new CFGuard(); }

bool llvm::isCFGuardCall(const CallBase *CB) {
  return CB->getCallingConv() == CallingConv::CFGuard_Check ||
         CB->countOperandBundlesOfType(LLVMContext::OB_cfguardtarget);
```

- **L301**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L302**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L303**: Returns from the current function with `true`. / 以 `true` 从当前函数返回。
- **L304**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L305**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L306**: Starts a function, method, or lambda body: `PreservedAnalyses CFGuardPass::run(Function &F, FunctionAnalysisManager &FAM) {`. / 开始一个函数、方法或 lambda 的主体：`PreservedAnalyses CFGuardPass::run(Function &F, FunctionAnalysisManager &FAM) {`。
- **L307**: Executes a standalone statement or declaration: `CFGuardImpl Impl;`. / 执行一条独立语句或声明：`CFGuardImpl Impl;`。
- **L308**: Initializes variable `Changed` from the right-hand expression. / 使用右侧表达式初始化变量 `Changed`。
- **L309**: Executes call or statement centered on `Impl.runOnFunction`. / 执行以 `Impl.runOnFunction` 为核心的调用或语句。
- **L310**: Returns from the current function with `Changed ? PreservedAnalyses::none() : PreservedAnalyses::all()`. / 以 `Changed ? PreservedAnalyses::none() : PreservedAnalyses::all()` 从当前函数返回。
- **L311**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L312**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L313**: Executes a standalone statement or declaration: `char CFGuard::ID = 0;`. / 执行一条独立语句或声明：`char CFGuard::ID = 0;`。
- **L314**: Continues the surrounding expression or declaration: `INITIALIZE_PASS(CFGuard, "CFGuard", "CFGuard", false, false)`. / 继续构造周围的表达式或声明：`INITIALIZE_PASS(CFGuard, "CFGuard", "CFGuard", false, false)`。
- **L315**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L316**: Continues the surrounding expression or declaration: `FunctionPass *llvm::createCFGuardPass() { return new CFGuard(); }`. / 继续构造周围的表达式或声明：`FunctionPass *llvm::createCFGuardPass() { return new CFGuard(); }`。
- **L317**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L318**: Starts a function, method, or lambda body: `bool llvm::isCFGuardCall(const CallBase *CB) {`. / 开始一个函数、方法或 lambda 的主体：`bool llvm::isCFGuardCall(const CallBase *CB) {`。
- **L319**: Returns from the current function with `CB->getCallingConv() == CallingConv::CFGuard_Check ||`. / 以 `CB->getCallingConv() == CallingConv::CFGuard_Check ||` 从当前函数返回。
- **L320**: Executes call or statement centered on `CB->countOperandBundlesOfType`. / 执行以 `CB->countOperandBundlesOfType` 为核心的调用或语句。

### Lines 321-329

```cpp
}

bool llvm::isCFGuardFunction(const GlobalValue *GV) {
  if (GV->getLinkage() != GlobalValue::ExternalLinkage)
    return false;

  StringRef Name = GV->getName();
  return Name == GuardCheckFunctionName || Name == GuardDispatchFunctionName;
}
```

- **L321**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L322**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L323**: Starts a function, method, or lambda body: `bool llvm::isCFGuardFunction(const GlobalValue *GV) {`. / 开始一个函数、方法或 lambda 的主体：`bool llvm::isCFGuardFunction(const GlobalValue *GV) {`。
- **L324**: Begins a `if` control-flow statement and evaluates its condition. / 开始 `if` 控制流语句并计算其条件。
- **L325**: Returns from the current function with `false`. / 以 `false` 从当前函数返回。
- **L326**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L327**: Initializes variable `Name` from the right-hand expression. / 使用右侧表达式初始化变量 `Name`。
- **L328**: Returns from the current function with `Name == GuardCheckFunctionName || Name == GuardDispatchFunctionName`. / 以 `Name == GuardCheckFunctionName || Name == GuardDispatchFunctionName` 从当前函数返回。
- **L329**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。

## Key Concepts / 关键概念

- **CFGuard transform pipeline / CFGuard 变换流水线**
- **IR construction and rewriting / IR 构造与重写**
- **Function-level traversal and updates / 函数级遍历与更新**
- **Module-wide coordination / 模块范围的协调**
- **Pass statistics and instrumentation / Pass 统计与观测**
- **Analysis preservation reporting / 分析保持情况报告**

## Dependencies / 依赖关系

- `llvm/Transforms/CFGuard.h`: Provides transform-specific declarations. / 提供变换相关声明。
- `llvm/ADT/SmallVector.h`: Provides LLVM ADT data structures/utilities. / 提供LLVM ADT 数据结构/工具。
- `llvm/ADT/Statistic.h`: Provides LLVM ADT data structures/utilities. / 提供LLVM ADT 数据结构/工具。
- `llvm/IR/CallingConv.h`: Provides LLVM IR core types and builders. / 提供LLVM IR 核心类型与构造工具。
- `llvm/IR/IRBuilder.h`: Provides LLVM IR core types and builders. / 提供LLVM IR 核心类型与构造工具。
- `llvm/IR/Instruction.h`: Provides LLVM IR core types and builders. / 提供LLVM IR 核心类型与构造工具。
- `llvm/IR/Module.h`: Provides LLVM IR core types and builders. / 提供LLVM IR 核心类型与构造工具。
- `llvm/InitializePasses.h`: Provides local declarations used by this file. / 提供本文件使用的本地声明。
- `llvm/Pass.h`: Provides local declarations used by this file. / 提供本文件使用的本地声明。
- `llvm/TargetParser/Triple.h`: Provides local declarations used by this file. / 提供本文件使用的本地声明。
