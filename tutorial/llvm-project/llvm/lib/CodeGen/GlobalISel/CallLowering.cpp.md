# CallLowering.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `llvm/lib/CodeGen/GlobalISel/CallLowering.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose (EN)**: This file contains `Call lowering` logic inside LLVM CodeGen.
- **Purpose (CN)**: 该文件在 LLVM CodeGen 中实现与“Call lowering”相关的逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-20

````cpp
//===-- lib/CodeGen/GlobalISel/CallLowering.cpp - Call lowering -----------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
///
/// \file
/// This file implements some simple delegations needed for call lowering.
///
//===----------------------------------------------------------------------===//

#include "llvm/CodeGen/GlobalISel/CallLowering.h"
#include "llvm/CodeGen/Analysis.h"
#include "llvm/CodeGen/CallingConvLower.h"
#include "llvm/CodeGen/GlobalISel/MachineIRBuilder.h"
#include "llvm/CodeGen/GlobalISel/Utils.h"
#include "llvm/CodeGen/MachineFrameInfo.h"
#include "llvm/CodeGen/MachineOperand.h"
````
- **L1 EN**: Comment documents: `===-- lib/CodeGen/GlobalISel/CallLowering.cpp - Call lowering ----------…`.
  **L1 CN**: 注释说明：`===-- lib/CodeGen/GlobalISel/CallLowering.cpp - Call lowering ----------…`。
- **L2 EN**: Continues the surrounding comment block.
  **L2 CN**: 延续周围的注释块。
- **L3 EN**: Comment documents: `Part of the LLVM Project, under the Apache License v2.0 with LLVM Except…`.
  **L3 CN**: 注释说明：`Part of the LLVM Project, under the Apache License v2.0 with LLVM Except…`。
- **L4 EN**: Comment documents: `See https://llvm.org/LICENSE.txt for license information.`.
  **L4 CN**: 注释说明：`See https://llvm.org/LICENSE.txt for license information.`。
- **L5 EN**: Comment documents: `SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception`.
  **L5 CN**: 注释说明：`SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception`。
- **L6 EN**: Continues the surrounding comment block.
  **L6 CN**: 延续周围的注释块。
- **L7 EN**: Comment documents: `===---------------------------------------------------------------------…`.
  **L7 CN**: 注释说明：`===---------------------------------------------------------------------…`。
- **L8 EN**: Continues the surrounding comment block.
  **L8 CN**: 延续周围的注释块。
- **L9 EN**: Comment documents: `\file`.
  **L9 CN**: 注释说明：`\file`。
- **L10 EN**: Comment documents: `This file implements some simple delegations needed for call lowering.`.
  **L10 CN**: 注释说明：`This file implements some simple delegations needed for call lowering.`。
- **L11 EN**: Continues the surrounding comment block.
  **L11 CN**: 延续周围的注释块。
- **L12 EN**: Comment documents: `===---------------------------------------------------------------------…`.
  **L12 CN**: 注释说明：`===---------------------------------------------------------------------…`。
- **L13 EN**: Separates nearby statements for readability.
  **L13 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L14 EN**: Includes LLVM header `llvm/CodeGen/GlobalISel/CallLowering.h` for CallLowering support.
  **L14 CN**: 引入 LLVM 头文件 `llvm/CodeGen/GlobalISel/CallLowering.h`，用于 CallLowering 相关支持。
- **L15 EN**: Includes LLVM header `llvm/CodeGen/Analysis.h` for Analysis support.
  **L15 CN**: 引入 LLVM 头文件 `llvm/CodeGen/Analysis.h`，用于 Analysis 相关支持。
- **L16 EN**: Includes LLVM header `llvm/CodeGen/CallingConvLower.h` for CallingConvLower support.
  **L16 CN**: 引入 LLVM 头文件 `llvm/CodeGen/CallingConvLower.h`，用于 CallingConvLower 相关支持。
- **L17 EN**: Includes LLVM header `llvm/CodeGen/GlobalISel/MachineIRBuilder.h` for MachineIRBuilder support.
  **L17 CN**: 引入 LLVM 头文件 `llvm/CodeGen/GlobalISel/MachineIRBuilder.h`，用于 MachineIRBuilder 相关支持。
- **L18 EN**: Includes LLVM header `llvm/CodeGen/GlobalISel/Utils.h` for Utils support.
  **L18 CN**: 引入 LLVM 头文件 `llvm/CodeGen/GlobalISel/Utils.h`，用于 Utils 相关支持。
- **L19 EN**: Includes LLVM header `llvm/CodeGen/MachineFrameInfo.h` for MachineFrameInfo support.
  **L19 CN**: 引入 LLVM 头文件 `llvm/CodeGen/MachineFrameInfo.h`，用于 MachineFrameInfo 相关支持。
- **L20 EN**: Includes LLVM header `llvm/CodeGen/MachineOperand.h` for MachineOperand support.
  **L20 CN**: 引入 LLVM 头文件 `llvm/CodeGen/MachineOperand.h`，用于 MachineOperand 相关支持。

### Lines 21-40

````cpp
#include "llvm/CodeGen/MachineRegisterInfo.h"
#include "llvm/CodeGen/TargetLowering.h"
#include "llvm/IR/DataLayout.h"
#include "llvm/IR/LLVMContext.h"
#include "llvm/IR/Module.h"
#include "llvm/Target/TargetMachine.h"

#define DEBUG_TYPE "call-lowering"

using namespace llvm;

void CallLowering::anchor() {}

/// Helper function which updates \p Flags based on the contents of \p Attrs.
static void addFlagsFromAttrSet(ISD::ArgFlagsTy &Flags, AttributeSet Attrs) {
  if (!Attrs.hasAttributes())
    return;

  // TODO: There are missing flags. Add them here.
  if (Attrs.hasAttribute(Attribute::SExt))
````
- **L21 EN**: Includes LLVM header `llvm/CodeGen/MachineRegisterInfo.h` for MachineRegisterInfo support.
  **L21 CN**: 引入 LLVM 头文件 `llvm/CodeGen/MachineRegisterInfo.h`，用于 MachineRegisterInfo 相关支持。
- **L22 EN**: Includes LLVM header `llvm/CodeGen/TargetLowering.h` for TargetLowering support.
  **L22 CN**: 引入 LLVM 头文件 `llvm/CodeGen/TargetLowering.h`，用于 TargetLowering 相关支持。
- **L23 EN**: Includes LLVM header `llvm/IR/DataLayout.h` for DataLayout support.
  **L23 CN**: 引入 LLVM 头文件 `llvm/IR/DataLayout.h`，用于 DataLayout 相关支持。
- **L24 EN**: Includes LLVM header `llvm/IR/LLVMContext.h` for LLVMContext support.
  **L24 CN**: 引入 LLVM 头文件 `llvm/IR/LLVMContext.h`，用于 LLVMContext 相关支持。
- **L25 EN**: Includes LLVM header `llvm/IR/Module.h` for Module support.
  **L25 CN**: 引入 LLVM 头文件 `llvm/IR/Module.h`，用于 Module 相关支持。
- **L26 EN**: Includes LLVM header `llvm/Target/TargetMachine.h` for TargetMachine support.
  **L26 CN**: 引入 LLVM 头文件 `llvm/Target/TargetMachine.h`，用于 TargetMachine 相关支持。
- **L27 EN**: Separates nearby statements for readability.
  **L27 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L28 EN**: Defines the LLVM debug channel used by this file.
  **L28 CN**: 定义该文件使用的 LLVM 调试通道。
- **L29 EN**: Separates nearby statements for readability.
  **L29 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L30 EN**: Imports namespace `llvm` into this translation unit.
  **L30 CN**: 将命名空间 `llvm` 引入当前编译单元。
- **L31 EN**: Separates nearby statements for readability.
  **L31 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L32 EN**: Provides part of the signature for `anchor`.
  **L32 CN**: 给出 `anchor` 的一部分签名。
- **L33 EN**: Separates nearby statements for readability.
  **L33 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L34 EN**: Comment documents: `Helper function which updates \p Flags based on the contents of \p Attrs…`.
  **L34 CN**: 注释说明：`Helper function which updates \p Flags based on the contents of \p Attrs…`。
- **L35 EN**: Begins the definition of `addFlagsFromAttrSet`.
  **L35 CN**: 开始定义 `addFlagsFromAttrSet`。
- **L36 EN**: Begins a conditional branch.
  **L36 CN**: 开始一个条件分支。
- **L37 EN**: Returns control to the caller.
  **L37 CN**: 将控制流返回给调用者。
- **L38 EN**: Separates nearby statements for readability.
  **L38 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L39 EN**: Comment documents: `TODO: There are missing flags. Add them here.`.
  **L39 CN**: 注释说明：`TODO: There are missing flags. Add them here.`。
- **L40 EN**: Begins a conditional branch.
  **L40 CN**: 开始一个条件分支。

### Lines 41-60

````cpp
    Flags.setSExt();
  if (Attrs.hasAttribute(Attribute::ZExt))
    Flags.setZExt();
  if (Attrs.hasAttribute(Attribute::InReg))
    Flags.setInReg();
  if (Attrs.hasAttribute(Attribute::StructRet))
    Flags.setSRet();
  if (Attrs.hasAttribute(Attribute::Nest))
    Flags.setNest();
  if (Attrs.hasAttribute(Attribute::ByVal))
    Flags.setByVal();
  if (Attrs.hasAttribute(Attribute::ByRef))
    Flags.setByRef();
  if (Attrs.hasAttribute(Attribute::Preallocated))
    Flags.setPreallocated();
  if (Attrs.hasAttribute(Attribute::InAlloca))
    Flags.setInAlloca();
  if (Attrs.hasAttribute(Attribute::Returned))
    Flags.setReturned();
  if (Attrs.hasAttribute(Attribute::SwiftSelf))
````
- **L41 EN**: Executes statement `Flags.setSExt();`.
  **L41 CN**: 执行语句 `Flags.setSExt();`。
- **L42 EN**: Begins a conditional branch.
  **L42 CN**: 开始一个条件分支。
- **L43 EN**: Executes statement `Flags.setZExt();`.
  **L43 CN**: 执行语句 `Flags.setZExt();`。
- **L44 EN**: Begins a conditional branch.
  **L44 CN**: 开始一个条件分支。
- **L45 EN**: Executes statement `Flags.setInReg();`.
  **L45 CN**: 执行语句 `Flags.setInReg();`。
- **L46 EN**: Begins a conditional branch.
  **L46 CN**: 开始一个条件分支。
- **L47 EN**: Executes statement `Flags.setSRet();`.
  **L47 CN**: 执行语句 `Flags.setSRet();`。
- **L48 EN**: Begins a conditional branch.
  **L48 CN**: 开始一个条件分支。
- **L49 EN**: Executes statement `Flags.setNest();`.
  **L49 CN**: 执行语句 `Flags.setNest();`。
- **L50 EN**: Begins a conditional branch.
  **L50 CN**: 开始一个条件分支。
- **L51 EN**: Executes statement `Flags.setByVal();`.
  **L51 CN**: 执行语句 `Flags.setByVal();`。
- **L52 EN**: Begins a conditional branch.
  **L52 CN**: 开始一个条件分支。
- **L53 EN**: Executes statement `Flags.setByRef();`.
  **L53 CN**: 执行语句 `Flags.setByRef();`。
- **L54 EN**: Begins a conditional branch.
  **L54 CN**: 开始一个条件分支。
- **L55 EN**: Executes statement `Flags.setPreallocated();`.
  **L55 CN**: 执行语句 `Flags.setPreallocated();`。
- **L56 EN**: Begins a conditional branch.
  **L56 CN**: 开始一个条件分支。
- **L57 EN**: Executes statement `Flags.setInAlloca();`.
  **L57 CN**: 执行语句 `Flags.setInAlloca();`。
- **L58 EN**: Begins a conditional branch.
  **L58 CN**: 开始一个条件分支。
- **L59 EN**: Executes statement `Flags.setReturned();`.
  **L59 CN**: 执行语句 `Flags.setReturned();`。
- **L60 EN**: Begins a conditional branch.
  **L60 CN**: 开始一个条件分支。

### Lines 61-80

````cpp
    Flags.setSwiftSelf();
  if (Attrs.hasAttribute(Attribute::SwiftAsync))
    Flags.setSwiftAsync();
  if (Attrs.hasAttribute(Attribute::SwiftError))
    Flags.setSwiftError();
}

ISD::ArgFlagsTy CallLowering::getAttributesForArgIdx(const CallBase &Call,
                                                     unsigned ArgIdx) const {
  ISD::ArgFlagsTy Flags;
  const AttributeList &Attrs = Call.getAttributes();
  addFlagsFromAttrSet(Flags, Attrs.getParamAttrs(ArgIdx));
  if (const Function *F = Call.getCalledFunction())
    addFlagsFromAttrSet(Flags, F->getAttributes().getParamAttrs(ArgIdx));
  return Flags;
}

ISD::ArgFlagsTy
CallLowering::getAttributesForReturn(const CallBase &Call) const {
  ISD::ArgFlagsTy Flags;
````
- **L61 EN**: Executes statement `Flags.setSwiftSelf();`.
  **L61 CN**: 执行语句 `Flags.setSwiftSelf();`。
- **L62 EN**: Begins a conditional branch.
  **L62 CN**: 开始一个条件分支。
- **L63 EN**: Executes statement `Flags.setSwiftAsync();`.
  **L63 CN**: 执行语句 `Flags.setSwiftAsync();`。
- **L64 EN**: Begins a conditional branch.
  **L64 CN**: 开始一个条件分支。
- **L65 EN**: Executes statement `Flags.setSwiftError();`.
  **L65 CN**: 执行语句 `Flags.setSwiftError();`。
- **L66 EN**: Closes the current scope.
  **L66 CN**: 关闭当前作用域。
- **L67 EN**: Separates nearby statements for readability.
  **L67 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L68 EN**: Provides part of the signature for `getAttributesForArgIdx`.
  **L68 CN**: 给出 `getAttributesForArgIdx` 的一部分签名。
- **L69 EN**: Starts block `unsigned ArgIdx) const`.
  **L69 CN**: 开始代码块 `unsigned ArgIdx) const`。
- **L70 EN**: Executes statement `ISD::ArgFlagsTy Flags;`.
  **L70 CN**: 执行语句 `ISD::ArgFlagsTy Flags;`。
- **L71 EN**: Assigns or initializes `const AttributeList &Attrs`.
  **L71 CN**: 对 `const AttributeList &Attrs` 进行赋值或初始化。
- **L72 EN**: Executes statement `addFlagsFromAttrSet(Flags, Attrs.getParamAttrs(ArgIdx));`.
  **L72 CN**: 执行语句 `addFlagsFromAttrSet(Flags, Attrs.getParamAttrs(ArgIdx));`。
- **L73 EN**: Begins a conditional branch.
  **L73 CN**: 开始一个条件分支。
- **L74 EN**: Executes statement `addFlagsFromAttrSet(Flags, F->getAttributes().getParamAttrs(ArgIdx));`.
  **L74 CN**: 执行语句 `addFlagsFromAttrSet(Flags, F->getAttributes().getParamAttrs(ArgIdx));`。
- **L75 EN**: Returns `Flags` to the caller.
  **L75 CN**: 向调用者返回 `Flags`。
- **L76 EN**: Closes the current scope.
  **L76 CN**: 关闭当前作用域。
- **L77 EN**: Separates nearby statements for readability.
  **L77 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L78 EN**: Continues logic with `ISD::ArgFlagsTy`.
  **L78 CN**: 继续处理逻辑：`ISD::ArgFlagsTy`。
- **L79 EN**: Begins the definition of `getAttributesForReturn`.
  **L79 CN**: 开始定义 `getAttributesForReturn`。
- **L80 EN**: Executes statement `ISD::ArgFlagsTy Flags;`.
  **L80 CN**: 执行语句 `ISD::ArgFlagsTy Flags;`。

### Lines 81-100

````cpp
  addFlagsFromAttrSet(Flags, Call.getAttributes().getRetAttrs());
  if (const Function *F = Call.getCalledFunction())
    addFlagsFromAttrSet(Flags, F->getAttributes().getRetAttrs());
  return Flags;
}

void CallLowering::addArgFlagsFromAttributes(ISD::ArgFlagsTy &Flags,
                                             const AttributeList &Attrs,
                                             unsigned OpIdx) const {
  addFlagsFromAttrSet(Flags, Attrs.getAttributes(OpIdx));
}

bool CallLowering::lowerCall(MachineIRBuilder &MIRBuilder, const CallBase &CB,
                             ArrayRef<Register> ResRegs,
                             ArrayRef<ArrayRef<Register>> ArgRegs,
                             Register SwiftErrorVReg,
                             std::optional<PtrAuthInfo> PAI,
                             Register ConvergenceCtrlToken,
                             std::function<Register()> GetCalleeReg) const {
  CallLoweringInfo Info;
````
- **L81 EN**: Executes statement `addFlagsFromAttrSet(Flags, Call.getAttributes().getRetAttrs());`.
  **L81 CN**: 执行语句 `addFlagsFromAttrSet(Flags, Call.getAttributes().getRetAttrs());`。
- **L82 EN**: Begins a conditional branch.
  **L82 CN**: 开始一个条件分支。
- **L83 EN**: Executes statement `addFlagsFromAttrSet(Flags, F->getAttributes().getRetAttrs());`.
  **L83 CN**: 执行语句 `addFlagsFromAttrSet(Flags, F->getAttributes().getRetAttrs());`。
- **L84 EN**: Returns `Flags` to the caller.
  **L84 CN**: 向调用者返回 `Flags`。
- **L85 EN**: Closes the current scope.
  **L85 CN**: 关闭当前作用域。
- **L86 EN**: Separates nearby statements for readability.
  **L86 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L87 EN**: Provides part of the signature for `addArgFlagsFromAttributes`.
  **L87 CN**: 给出 `addArgFlagsFromAttributes` 的一部分签名。
- **L88 EN**: Continues logic with `const AttributeList &Attrs,`.
  **L88 CN**: 继续处理逻辑：`const AttributeList &Attrs,`。
- **L89 EN**: Starts block `unsigned OpIdx) const`.
  **L89 CN**: 开始代码块 `unsigned OpIdx) const`。
- **L90 EN**: Executes statement `addFlagsFromAttrSet(Flags, Attrs.getAttributes(OpIdx));`.
  **L90 CN**: 执行语句 `addFlagsFromAttrSet(Flags, Attrs.getAttributes(OpIdx));`。
- **L91 EN**: Closes the current scope.
  **L91 CN**: 关闭当前作用域。
- **L92 EN**: Separates nearby statements for readability.
  **L92 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L93 EN**: Provides part of the signature for `lowerCall`.
  **L93 CN**: 给出 `lowerCall` 的一部分签名。
- **L94 EN**: Continues logic with `ArrayRef<Register> ResRegs,`.
  **L94 CN**: 继续处理逻辑：`ArrayRef<Register> ResRegs,`。
- **L95 EN**: Continues logic with `ArrayRef<ArrayRef<Register>> ArgRegs,`.
  **L95 CN**: 继续处理逻辑：`ArrayRef<ArrayRef<Register>> ArgRegs,`。
- **L96 EN**: Continues logic with `Register SwiftErrorVReg,`.
  **L96 CN**: 继续处理逻辑：`Register SwiftErrorVReg,`。
- **L97 EN**: Continues logic with `std::optional<PtrAuthInfo> PAI,`.
  **L97 CN**: 继续处理逻辑：`std::optional<PtrAuthInfo> PAI,`。
- **L98 EN**: Continues logic with `Register ConvergenceCtrlToken,`.
  **L98 CN**: 继续处理逻辑：`Register ConvergenceCtrlToken,`。
- **L99 EN**: Begins the definition of `Register`.
  **L99 CN**: 开始定义 `Register`。
- **L100 EN**: Executes statement `CallLoweringInfo Info;`.
  **L100 CN**: 执行语句 `CallLoweringInfo Info;`。

### Lines 101-120

````cpp
  const DataLayout &DL = MIRBuilder.getDataLayout();
  MachineFunction &MF = MIRBuilder.getMF();
  MachineRegisterInfo &MRI = MF.getRegInfo();
  bool CanBeTailCalled = CB.isTailCall() &&
                         isInTailCallPosition(CB, MF.getTarget()) &&
                         (MF.getFunction()
                              .getFnAttribute("disable-tail-calls")
                              .getValueAsString() != "true");

  CallingConv::ID CallConv = CB.getCallingConv();
  Type *RetTy = CB.getType();
  bool IsVarArg = CB.getFunctionType()->isVarArg();

  SmallVector<BaseArgInfo, 4> SplitArgs;
  getReturnInfo(CallConv, RetTy, CB.getAttributes(), SplitArgs, DL);
  Info.CanLowerReturn = canLowerReturn(MF, CallConv, SplitArgs, IsVarArg);

  Info.IsConvergent = CB.isConvergent();

  if (!Info.CanLowerReturn) {
````
- **L101 EN**: Assigns or initializes `const DataLayout &DL`.
  **L101 CN**: 对 `const DataLayout &DL` 进行赋值或初始化。
- **L102 EN**: Assigns or initializes `MachineFunction &MF`.
  **L102 CN**: 对 `MachineFunction &MF` 进行赋值或初始化。
- **L103 EN**: Assigns or initializes `MachineRegisterInfo &MRI`.
  **L103 CN**: 对 `MachineRegisterInfo &MRI` 进行赋值或初始化。
- **L104 EN**: Continues logic with `bool CanBeTailCalled = CB.isTailCall() &&`.
  **L104 CN**: 继续处理逻辑：`bool CanBeTailCalled = CB.isTailCall() &&`。
- **L105 EN**: Continues logic with `isInTailCallPosition(CB, MF.getTarget()) &&`.
  **L105 CN**: 继续处理逻辑：`isInTailCallPosition(CB, MF.getTarget()) &&`。
- **L106 EN**: Continues logic with `(MF.getFunction()`.
  **L106 CN**: 继续处理逻辑：`(MF.getFunction()`。
- **L107 EN**: Continues logic with `.getFnAttribute("disable-tail-calls")`.
  **L107 CN**: 继续处理逻辑：`.getFnAttribute("disable-tail-calls")`。
- **L108 EN**: Assigns or initializes `.getValueAsString() !`.
  **L108 CN**: 对 `.getValueAsString() !` 进行赋值或初始化。
- **L109 EN**: Separates nearby statements for readability.
  **L109 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L110 EN**: Assigns or initializes `CallingConv::ID CallConv`.
  **L110 CN**: 对 `CallingConv::ID CallConv` 进行赋值或初始化。
- **L111 EN**: Assigns or initializes `Type *RetTy`.
  **L111 CN**: 对 `Type *RetTy` 进行赋值或初始化。
- **L112 EN**: Assigns or initializes `bool IsVarArg`.
  **L112 CN**: 对 `bool IsVarArg` 进行赋值或初始化。
- **L113 EN**: Separates nearby statements for readability.
  **L113 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L114 EN**: Executes statement `SmallVector<BaseArgInfo, 4> SplitArgs;`.
  **L114 CN**: 执行语句 `SmallVector<BaseArgInfo, 4> SplitArgs;`。
- **L115 EN**: Executes statement `getReturnInfo(CallConv, RetTy, CB.getAttributes(), SplitArgs, DL);`.
  **L115 CN**: 执行语句 `getReturnInfo(CallConv, RetTy, CB.getAttributes(), SplitArgs, DL);`。
- **L116 EN**: Assigns or initializes `Info.CanLowerReturn`.
  **L116 CN**: 对 `Info.CanLowerReturn` 进行赋值或初始化。
- **L117 EN**: Separates nearby statements for readability.
  **L117 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L118 EN**: Assigns or initializes `Info.IsConvergent`.
  **L118 CN**: 对 `Info.IsConvergent` 进行赋值或初始化。
- **L119 EN**: Separates nearby statements for readability.
  **L119 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L120 EN**: Begins a conditional branch.
  **L120 CN**: 开始一个条件分支。

### Lines 121-140

````cpp
    // Callee requires sret demotion.
    insertSRetOutgoingArgument(MIRBuilder, CB, Info);

    // The sret demotion isn't compatible with tail-calls, since the sret
    // argument points into the caller's stack frame.
    CanBeTailCalled = false;
  }

  // First step is to marshall all the function's parameters into the correct
  // physregs and memory locations. Gather the sequence of argument types that
  // we'll pass to the assigner function.
  unsigned i = 0;
  unsigned NumFixedArgs = CB.getFunctionType()->getNumParams();
  for (const auto &Arg : CB.args()) {
    ArgInfo OrigArg{ArgRegs[i], *Arg.get(), i, getAttributesForArgIdx(CB, i)};
    setArgFlags(OrigArg, i + AttributeList::FirstArgIndex, DL, CB);
    if (i >= NumFixedArgs)
      OrigArg.Flags[0].setVarArg();

    // If we have an explicit sret argument that is an Instruction, (i.e., it
````
- **L121 EN**: Comment documents: `Callee requires sret demotion.`.
  **L121 CN**: 注释说明：`Callee requires sret demotion.`。
- **L122 EN**: Executes statement `insertSRetOutgoingArgument(MIRBuilder, CB, Info);`.
  **L122 CN**: 执行语句 `insertSRetOutgoingArgument(MIRBuilder, CB, Info);`。
- **L123 EN**: Separates nearby statements for readability.
  **L123 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L124 EN**: Comment documents: `The sret demotion isn't compatible with tail-calls, since the sret`.
  **L124 CN**: 注释说明：`The sret demotion isn't compatible with tail-calls, since the sret`。
- **L125 EN**: Comment documents: `argument points into the caller's stack frame.`.
  **L125 CN**: 注释说明：`argument points into the caller's stack frame.`。
- **L126 EN**: Assigns or initializes `CanBeTailCalled`.
  **L126 CN**: 对 `CanBeTailCalled` 进行赋值或初始化。
- **L127 EN**: Closes the current scope.
  **L127 CN**: 关闭当前作用域。
- **L128 EN**: Separates nearby statements for readability.
  **L128 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L129 EN**: Comment documents: `First step is to marshall all the function's parameters into the correct`.
  **L129 CN**: 注释说明：`First step is to marshall all the function's parameters into the correct`。
- **L130 EN**: Comment documents: `physregs and memory locations. Gather the sequence of argument types tha…`.
  **L130 CN**: 注释说明：`physregs and memory locations. Gather the sequence of argument types tha…`。
- **L131 EN**: Comment documents: `we'll pass to the assigner function.`.
  **L131 CN**: 注释说明：`we'll pass to the assigner function.`。
- **L132 EN**: Assigns or initializes `unsigned i`.
  **L132 CN**: 对 `unsigned i` 进行赋值或初始化。
- **L133 EN**: Assigns or initializes `unsigned NumFixedArgs`.
  **L133 CN**: 对 `unsigned NumFixedArgs` 进行赋值或初始化。
- **L134 EN**: Starts a loop over a sequence or range.
  **L134 CN**: 开始遍历序列或范围的循环。
- **L135 EN**: Executes statement `ArgInfo OrigArg{ArgRegs[i], *Arg.get(), i, getAttributesForArgIdx(CB, i)…`.
  **L135 CN**: 执行语句 `ArgInfo OrigArg{ArgRegs[i], *Arg.get(), i, getAttributesForArgIdx(CB, i)…`。
- **L136 EN**: Executes statement `setArgFlags(OrigArg, i + AttributeList::FirstArgIndex, DL, CB);`.
  **L136 CN**: 执行语句 `setArgFlags(OrigArg, i + AttributeList::FirstArgIndex, DL, CB);`。
- **L137 EN**: Begins a conditional branch.
  **L137 CN**: 开始一个条件分支。
- **L138 EN**: Executes statement `OrigArg.Flags[0].setVarArg();`.
  **L138 CN**: 执行语句 `OrigArg.Flags[0].setVarArg();`。
- **L139 EN**: Separates nearby statements for readability.
  **L139 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L140 EN**: Comment documents: `If we have an explicit sret argument that is an Instruction, (i.e., it`.
  **L140 CN**: 注释说明：`If we have an explicit sret argument that is an Instruction, (i.e., it`。

### Lines 141-160

````cpp
    // might point to function-local memory), we can't meaningfully tail-call.
    if (OrigArg.Flags[0].isSRet() && isa<Instruction>(&Arg))
      CanBeTailCalled = false;

    Info.OrigArgs.push_back(OrigArg);
    ++i;
  }

  // Try looking through a bitcast from one function type to another.
  // Commonly happens with calls to objc_msgSend().
  const Value *CalleeV = CB.getCalledOperand()->stripPointerCasts();

  // If IRTranslator chose to drop the ptrauth info, we can turn this into
  // a direct call.
  if (!PAI && CB.countOperandBundlesOfType(LLVMContext::OB_ptrauth)) {
    CalleeV = cast<ConstantPtrAuth>(CalleeV)->getPointer();
    assert(isa<Function>(CalleeV));
  }

  if (const Function *F = dyn_cast<Function>(CalleeV)) {
````
- **L141 EN**: Comment documents: `might point to function-local memory), we can't meaningfully tail-call.`.
  **L141 CN**: 注释说明：`might point to function-local memory), we can't meaningfully tail-call.`。
- **L142 EN**: Begins a conditional branch.
  **L142 CN**: 开始一个条件分支。
- **L143 EN**: Assigns or initializes `CanBeTailCalled`.
  **L143 CN**: 对 `CanBeTailCalled` 进行赋值或初始化。
- **L144 EN**: Separates nearby statements for readability.
  **L144 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L145 EN**: Executes statement `Info.OrigArgs.push_back(OrigArg);`.
  **L145 CN**: 执行语句 `Info.OrigArgs.push_back(OrigArg);`。
- **L146 EN**: Executes statement `++i;`.
  **L146 CN**: 执行语句 `++i;`。
- **L147 EN**: Closes the current scope.
  **L147 CN**: 关闭当前作用域。
- **L148 EN**: Separates nearby statements for readability.
  **L148 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L149 EN**: Comment documents: `Try looking through a bitcast from one function type to another.`.
  **L149 CN**: 注释说明：`Try looking through a bitcast from one function type to another.`。
- **L150 EN**: Comment documents: `Commonly happens with calls to objc_msgSend().`.
  **L150 CN**: 注释说明：`Commonly happens with calls to objc_msgSend().`。
- **L151 EN**: Assigns or initializes `const Value *CalleeV`.
  **L151 CN**: 对 `const Value *CalleeV` 进行赋值或初始化。
- **L152 EN**: Separates nearby statements for readability.
  **L152 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L153 EN**: Comment documents: `If IRTranslator chose to drop the ptrauth info, we can turn this into`.
  **L153 CN**: 注释说明：`If IRTranslator chose to drop the ptrauth info, we can turn this into`。
- **L154 EN**: Comment documents: `a direct call.`.
  **L154 CN**: 注释说明：`a direct call.`。
- **L155 EN**: Begins a conditional branch.
  **L155 CN**: 开始一个条件分支。
- **L156 EN**: Assigns or initializes `CalleeV`.
  **L156 CN**: 对 `CalleeV` 进行赋值或初始化。
- **L157 EN**: Checks an invariant in debug builds.
  **L157 CN**: 在调试构建中检查一个不变量。
- **L158 EN**: Closes the current scope.
  **L158 CN**: 关闭当前作用域。
- **L159 EN**: Separates nearby statements for readability.
  **L159 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L160 EN**: Begins a conditional branch.
  **L160 CN**: 开始一个条件分支。

### Lines 161-180

````cpp
    if (F->hasFnAttribute(Attribute::NonLazyBind)) {
      LLT Ty = getLLTForType(*F->getType(), DL);
      Register Reg = MIRBuilder.buildGlobalValue(Ty, F).getReg(0);
      Info.Callee = MachineOperand::CreateReg(Reg, false);
    } else {
      Info.Callee = MachineOperand::CreateGA(F, 0);
    }
  } else if (isa<GlobalIFunc>(CalleeV) || isa<GlobalAlias>(CalleeV)) {
    // IR IFuncs and Aliases can't be forward declared (only defined), so the
    // callee must be in the same TU and therefore we can direct-call it without
    // worrying about it being out of range.
    Info.Callee = MachineOperand::CreateGA(cast<GlobalValue>(CalleeV), 0);
  } else
    Info.Callee = MachineOperand::CreateReg(GetCalleeReg(), false);

  Register ReturnHintAlignReg;
  Align ReturnHintAlign;

  Info.OrigRet = ArgInfo{ResRegs, RetTy, 0, getAttributesForReturn(CB)};

````
- **L161 EN**: Begins a conditional branch.
  **L161 CN**: 开始一个条件分支。
- **L162 EN**: Assigns or initializes `LLT Ty`.
  **L162 CN**: 对 `LLT Ty` 进行赋值或初始化。
- **L163 EN**: Assigns or initializes `Register Reg`.
  **L163 CN**: 对 `Register Reg` 进行赋值或初始化。
- **L164 EN**: Declares function or method `CreateReg`.
  **L164 CN**: 声明函数或方法 `CreateReg`。
- **L165 EN**: Starts block `} else`.
  **L165 CN**: 开始代码块 `} else`。
- **L166 EN**: Declares function or method `CreateGA`.
  **L166 CN**: 声明函数或方法 `CreateGA`。
- **L167 EN**: Closes the current scope.
  **L167 CN**: 关闭当前作用域。
- **L168 EN**: Starts block `} else if (isa<GlobalIFunc>(CalleeV) || isa<GlobalAlias>(CalleeV))`.
  **L168 CN**: 开始代码块 `} else if (isa<GlobalIFunc>(CalleeV) || isa<GlobalAlias>(CalleeV))`。
- **L169 EN**: Comment documents: `IR IFuncs and Aliases can't be forward declared (only defined), so the`.
  **L169 CN**: 注释说明：`IR IFuncs and Aliases can't be forward declared (only defined), so the`。
- **L170 EN**: Comment documents: `callee must be in the same TU and therefore we can direct-call it withou…`.
  **L170 CN**: 注释说明：`callee must be in the same TU and therefore we can direct-call it withou…`。
- **L171 EN**: Comment documents: `worrying about it being out of range.`.
  **L171 CN**: 注释说明：`worrying about it being out of range.`。
- **L172 EN**: Declares function or method `CreateGA`.
  **L172 CN**: 声明函数或方法 `CreateGA`。
- **L173 EN**: Continues logic with `} else`.
  **L173 CN**: 继续处理逻辑：`} else`。
- **L174 EN**: Declares function or method `CreateReg`.
  **L174 CN**: 声明函数或方法 `CreateReg`。
- **L175 EN**: Separates nearby statements for readability.
  **L175 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L176 EN**: Executes statement `Register ReturnHintAlignReg;`.
  **L176 CN**: 执行语句 `Register ReturnHintAlignReg;`。
- **L177 EN**: Executes statement `Align ReturnHintAlign;`.
  **L177 CN**: 执行语句 `Align ReturnHintAlign;`。
- **L178 EN**: Separates nearby statements for readability.
  **L178 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L179 EN**: Assigns or initializes `Info.OrigRet`.
  **L179 CN**: 对 `Info.OrigRet` 进行赋值或初始化。
- **L180 EN**: Separates nearby statements for readability.
  **L180 CN**: 空行，用于分隔相邻语句并提升可读性。

### Lines 181-200

````cpp
  if (!Info.OrigRet.Ty->isVoidTy()) {
    setArgFlags(Info.OrigRet, AttributeList::ReturnIndex, DL, CB);

    if (MaybeAlign Alignment = CB.getRetAlign()) {
      if (*Alignment > Align(1)) {
        ReturnHintAlignReg = MRI.cloneVirtualRegister(ResRegs[0]);
        Info.OrigRet.Regs[0] = ReturnHintAlignReg;
        ReturnHintAlign = *Alignment;
      }
    }
  }

  auto Bundle = CB.getOperandBundle(LLVMContext::OB_kcfi);
  if (Bundle && CB.isIndirectCall()) {
    Info.CFIType = cast<ConstantInt>(Bundle->Inputs[0]);
    assert(Info.CFIType->getType()->isIntegerTy(32) && "Invalid CFI type");
  }

  if (auto Bundle = CB.getOperandBundle(LLVMContext::OB_deactivation_symbol)) {
    Info.DeactivationSymbol = cast<GlobalValue>(Bundle->Inputs[0]);
````
- **L181 EN**: Begins a conditional branch.
  **L181 CN**: 开始一个条件分支。
- **L182 EN**: Executes statement `setArgFlags(Info.OrigRet, AttributeList::ReturnIndex, DL, CB);`.
  **L182 CN**: 执行语句 `setArgFlags(Info.OrigRet, AttributeList::ReturnIndex, DL, CB);`。
- **L183 EN**: Separates nearby statements for readability.
  **L183 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L184 EN**: Begins a conditional branch.
  **L184 CN**: 开始一个条件分支。
- **L185 EN**: Begins a conditional branch.
  **L185 CN**: 开始一个条件分支。
- **L186 EN**: Assigns or initializes `ReturnHintAlignReg`.
  **L186 CN**: 对 `ReturnHintAlignReg` 进行赋值或初始化。
- **L187 EN**: Assigns or initializes `Info.OrigRet.Regs[0]`.
  **L187 CN**: 对 `Info.OrigRet.Regs[0]` 进行赋值或初始化。
- **L188 EN**: Assigns or initializes `ReturnHintAlign`.
  **L188 CN**: 对 `ReturnHintAlign` 进行赋值或初始化。
- **L189 EN**: Closes the current scope.
  **L189 CN**: 关闭当前作用域。
- **L190 EN**: Closes the current scope.
  **L190 CN**: 关闭当前作用域。
- **L191 EN**: Closes the current scope.
  **L191 CN**: 关闭当前作用域。
- **L192 EN**: Separates nearby statements for readability.
  **L192 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L193 EN**: Assigns or initializes `auto Bundle`.
  **L193 CN**: 对 `auto Bundle` 进行赋值或初始化。
- **L194 EN**: Begins a conditional branch.
  **L194 CN**: 开始一个条件分支。
- **L195 EN**: Assigns or initializes `Info.CFIType`.
  **L195 CN**: 对 `Info.CFIType` 进行赋值或初始化。
- **L196 EN**: Checks an invariant in debug builds.
  **L196 CN**: 在调试构建中检查一个不变量。
- **L197 EN**: Closes the current scope.
  **L197 CN**: 关闭当前作用域。
- **L198 EN**: Separates nearby statements for readability.
  **L198 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L199 EN**: Begins a conditional branch.
  **L199 CN**: 开始一个条件分支。
- **L200 EN**: Assigns or initializes `Info.DeactivationSymbol`.
  **L200 CN**: 对 `Info.DeactivationSymbol` 进行赋值或初始化。

### Lines 201-220

````cpp
  }

  Info.CB = &CB;
  Info.KnownCallees = CB.getMetadata(LLVMContext::MD_callees);
  Info.CallConv = CallConv;
  Info.SwiftErrorVReg = SwiftErrorVReg;
  Info.PAI = PAI;
  Info.ConvergenceCtrlToken = ConvergenceCtrlToken;
  Info.IsMustTailCall = CB.isMustTailCall();
  Info.IsTailCall = CanBeTailCalled;
  Info.IsVarArg = IsVarArg;
  if (!lowerCall(MIRBuilder, Info))
    return false;

  if (ReturnHintAlignReg && !Info.LoweredTailCall) {
    MIRBuilder.buildAssertAlign(ResRegs[0], ReturnHintAlignReg,
                                ReturnHintAlign);
  }

  return true;
````
- **L201 EN**: Closes the current scope.
  **L201 CN**: 关闭当前作用域。
- **L202 EN**: Separates nearby statements for readability.
  **L202 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L203 EN**: Assigns or initializes `Info.CB`.
  **L203 CN**: 对 `Info.CB` 进行赋值或初始化。
- **L204 EN**: Assigns or initializes `Info.KnownCallees`.
  **L204 CN**: 对 `Info.KnownCallees` 进行赋值或初始化。
- **L205 EN**: Assigns or initializes `Info.CallConv`.
  **L205 CN**: 对 `Info.CallConv` 进行赋值或初始化。
- **L206 EN**: Assigns or initializes `Info.SwiftErrorVReg`.
  **L206 CN**: 对 `Info.SwiftErrorVReg` 进行赋值或初始化。
- **L207 EN**: Assigns or initializes `Info.PAI`.
  **L207 CN**: 对 `Info.PAI` 进行赋值或初始化。
- **L208 EN**: Assigns or initializes `Info.ConvergenceCtrlToken`.
  **L208 CN**: 对 `Info.ConvergenceCtrlToken` 进行赋值或初始化。
- **L209 EN**: Assigns or initializes `Info.IsMustTailCall`.
  **L209 CN**: 对 `Info.IsMustTailCall` 进行赋值或初始化。
- **L210 EN**: Assigns or initializes `Info.IsTailCall`.
  **L210 CN**: 对 `Info.IsTailCall` 进行赋值或初始化。
- **L211 EN**: Assigns or initializes `Info.IsVarArg`.
  **L211 CN**: 对 `Info.IsVarArg` 进行赋值或初始化。
- **L212 EN**: Begins a conditional branch.
  **L212 CN**: 开始一个条件分支。
- **L213 EN**: Returns `false` to the caller.
  **L213 CN**: 向调用者返回 `false`。
- **L214 EN**: Separates nearby statements for readability.
  **L214 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L215 EN**: Begins a conditional branch.
  **L215 CN**: 开始一个条件分支。
- **L216 EN**: Continues logic with `MIRBuilder.buildAssertAlign(ResRegs[0], ReturnHintAlignReg,`.
  **L216 CN**: 继续处理逻辑：`MIRBuilder.buildAssertAlign(ResRegs[0], ReturnHintAlignReg,`。
- **L217 EN**: Executes statement `ReturnHintAlign);`.
  **L217 CN**: 执行语句 `ReturnHintAlign);`。
- **L218 EN**: Closes the current scope.
  **L218 CN**: 关闭当前作用域。
- **L219 EN**: Separates nearby statements for readability.
  **L219 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L220 EN**: Returns `true` to the caller.
  **L220 CN**: 向调用者返回 `true`。

### Lines 221-240

````cpp
}

template <typename FuncInfoTy>
void CallLowering::setArgFlags(CallLowering::ArgInfo &Arg, unsigned OpIdx,
                               const DataLayout &DL,
                               const FuncInfoTy &FuncInfo) const {
  auto &Flags = Arg.Flags[0];
  const AttributeList &Attrs = FuncInfo.getAttributes();
  addArgFlagsFromAttributes(Flags, Attrs, OpIdx);

  PointerType *PtrTy = dyn_cast<PointerType>(Arg.Ty->getScalarType());
  if (PtrTy) {
    Flags.setPointer();
    Flags.setPointerAddrSpace(PtrTy->getPointerAddressSpace());
  }

  Align MemAlign = DL.getABITypeAlign(Arg.Ty);
  if (Flags.isByVal() || Flags.isInAlloca() || Flags.isPreallocated() ||
      Flags.isByRef()) {
    assert(OpIdx >= AttributeList::FirstArgIndex);
````
- **L221 EN**: Closes the current scope.
  **L221 CN**: 关闭当前作用域。
- **L222 EN**: Separates nearby statements for readability.
  **L222 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L223 EN**: Introduces a template parameter list.
  **L223 CN**: 引入模板参数列表。
- **L224 EN**: Provides part of the signature for `setArgFlags`.
  **L224 CN**: 给出 `setArgFlags` 的一部分签名。
- **L225 EN**: Continues logic with `const DataLayout &DL,`.
  **L225 CN**: 继续处理逻辑：`const DataLayout &DL,`。
- **L226 EN**: Starts block `const FuncInfoTy &FuncInfo) const`.
  **L226 CN**: 开始代码块 `const FuncInfoTy &FuncInfo) const`。
- **L227 EN**: Assigns or initializes `auto &Flags`.
  **L227 CN**: 对 `auto &Flags` 进行赋值或初始化。
- **L228 EN**: Assigns or initializes `const AttributeList &Attrs`.
  **L228 CN**: 对 `const AttributeList &Attrs` 进行赋值或初始化。
- **L229 EN**: Executes statement `addArgFlagsFromAttributes(Flags, Attrs, OpIdx);`.
  **L229 CN**: 执行语句 `addArgFlagsFromAttributes(Flags, Attrs, OpIdx);`。
- **L230 EN**: Separates nearby statements for readability.
  **L230 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L231 EN**: Assigns or initializes `PointerType *PtrTy`.
  **L231 CN**: 对 `PointerType *PtrTy` 进行赋值或初始化。
- **L232 EN**: Begins a conditional branch.
  **L232 CN**: 开始一个条件分支。
- **L233 EN**: Executes statement `Flags.setPointer();`.
  **L233 CN**: 执行语句 `Flags.setPointer();`。
- **L234 EN**: Executes statement `Flags.setPointerAddrSpace(PtrTy->getPointerAddressSpace());`.
  **L234 CN**: 执行语句 `Flags.setPointerAddrSpace(PtrTy->getPointerAddressSpace());`。
- **L235 EN**: Closes the current scope.
  **L235 CN**: 关闭当前作用域。
- **L236 EN**: Separates nearby statements for readability.
  **L236 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L237 EN**: Assigns or initializes `Align MemAlign`.
  **L237 CN**: 对 `Align MemAlign` 进行赋值或初始化。
- **L238 EN**: Begins a conditional branch.
  **L238 CN**: 开始一个条件分支。
- **L239 EN**: Starts block `Flags.isByRef())`.
  **L239 CN**: 开始代码块 `Flags.isByRef())`。
- **L240 EN**: Checks an invariant in debug builds.
  **L240 CN**: 在调试构建中检查一个不变量。

### Lines 241-260

````cpp
    unsigned ParamIdx = OpIdx - AttributeList::FirstArgIndex;

    Type *ElementTy = FuncInfo.getParamByValType(ParamIdx);
    if (!ElementTy)
      ElementTy = FuncInfo.getParamByRefType(ParamIdx);
    if (!ElementTy)
      ElementTy = FuncInfo.getParamInAllocaType(ParamIdx);
    if (!ElementTy)
      ElementTy = FuncInfo.getParamPreallocatedType(ParamIdx);

    assert(ElementTy && "Must have byval, inalloca or preallocated type");

    uint64_t MemSize = DL.getTypeAllocSize(ElementTy);
    if (Flags.isByRef())
      Flags.setByRefSize(MemSize);
    else
      Flags.setByValSize(MemSize);

    // For ByVal, alignment should be passed from FE.  BE will guess if
    // this info is not there but there are cases it cannot get right.
````
- **L241 EN**: Assigns or initializes `unsigned ParamIdx`.
  **L241 CN**: 对 `unsigned ParamIdx` 进行赋值或初始化。
- **L242 EN**: Separates nearby statements for readability.
  **L242 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L243 EN**: Assigns or initializes `Type *ElementTy`.
  **L243 CN**: 对 `Type *ElementTy` 进行赋值或初始化。
- **L244 EN**: Begins a conditional branch.
  **L244 CN**: 开始一个条件分支。
- **L245 EN**: Assigns or initializes `ElementTy`.
  **L245 CN**: 对 `ElementTy` 进行赋值或初始化。
- **L246 EN**: Begins a conditional branch.
  **L246 CN**: 开始一个条件分支。
- **L247 EN**: Assigns or initializes `ElementTy`.
  **L247 CN**: 对 `ElementTy` 进行赋值或初始化。
- **L248 EN**: Begins a conditional branch.
  **L248 CN**: 开始一个条件分支。
- **L249 EN**: Assigns or initializes `ElementTy`.
  **L249 CN**: 对 `ElementTy` 进行赋值或初始化。
- **L250 EN**: Separates nearby statements for readability.
  **L250 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L251 EN**: Checks an invariant in debug builds.
  **L251 CN**: 在调试构建中检查一个不变量。
- **L252 EN**: Separates nearby statements for readability.
  **L252 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L253 EN**: Assigns or initializes `uint64_t MemSize`.
  **L253 CN**: 对 `uint64_t MemSize` 进行赋值或初始化。
- **L254 EN**: Begins a conditional branch.
  **L254 CN**: 开始一个条件分支。
- **L255 EN**: Executes statement `Flags.setByRefSize(MemSize);`.
  **L255 CN**: 执行语句 `Flags.setByRefSize(MemSize);`。
- **L256 EN**: Handles the fallback branch.
  **L256 CN**: 处理兜底分支。
- **L257 EN**: Executes statement `Flags.setByValSize(MemSize);`.
  **L257 CN**: 执行语句 `Flags.setByValSize(MemSize);`。
- **L258 EN**: Separates nearby statements for readability.
  **L258 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L259 EN**: Comment documents: `For ByVal, alignment should be passed from FE. BE will guess if`.
  **L259 CN**: 注释说明：`For ByVal, alignment should be passed from FE. BE will guess if`。
- **L260 EN**: Comment documents: `this info is not there but there are cases it cannot get right.`.
  **L260 CN**: 注释说明：`this info is not there but there are cases it cannot get right.`。

### Lines 261-280

````cpp
    if (auto ParamAlign = FuncInfo.getParamStackAlign(ParamIdx))
      MemAlign = *ParamAlign;
    else if ((ParamAlign = FuncInfo.getParamAlign(ParamIdx)))
      MemAlign = *ParamAlign;
    else
      MemAlign = getTLI()->getByValTypeAlignment(ElementTy, DL);
  } else if (OpIdx >= AttributeList::FirstArgIndex) {
    if (auto ParamAlign =
            FuncInfo.getParamStackAlign(OpIdx - AttributeList::FirstArgIndex))
      MemAlign = *ParamAlign;
  }
  Flags.setMemAlign(MemAlign);
  Flags.setOrigAlign(DL.getABITypeAlign(Arg.Ty));

  // Don't try to use the returned attribute if the argument is marked as
  // swiftself, since it won't be passed in x0.
  if (Flags.isSwiftSelf())
    Flags.setReturned(false);
}

````
- **L261 EN**: Begins a conditional branch.
  **L261 CN**: 开始一个条件分支。
- **L262 EN**: Assigns or initializes `MemAlign`.
  **L262 CN**: 对 `MemAlign` 进行赋值或初始化。
- **L263 EN**: Checks an alternate conditional path.
  **L263 CN**: 检查一个备用条件分支。
- **L264 EN**: Assigns or initializes `MemAlign`.
  **L264 CN**: 对 `MemAlign` 进行赋值或初始化。
- **L265 EN**: Handles the fallback branch.
  **L265 CN**: 处理兜底分支。
- **L266 EN**: Assigns or initializes `MemAlign`.
  **L266 CN**: 对 `MemAlign` 进行赋值或初始化。
- **L267 EN**: Starts block `} else if (OpIdx >= AttributeList::FirstArgIndex)`.
  **L267 CN**: 开始代码块 `} else if (OpIdx >= AttributeList::FirstArgIndex)`。
- **L268 EN**: Begins a conditional branch.
  **L268 CN**: 开始一个条件分支。
- **L269 EN**: Continues logic with `FuncInfo.getParamStackAlign(OpIdx - AttributeList::FirstArgIndex))`.
  **L269 CN**: 继续处理逻辑：`FuncInfo.getParamStackAlign(OpIdx - AttributeList::FirstArgIndex))`。
- **L270 EN**: Assigns or initializes `MemAlign`.
  **L270 CN**: 对 `MemAlign` 进行赋值或初始化。
- **L271 EN**: Closes the current scope.
  **L271 CN**: 关闭当前作用域。
- **L272 EN**: Executes statement `Flags.setMemAlign(MemAlign);`.
  **L272 CN**: 执行语句 `Flags.setMemAlign(MemAlign);`。
- **L273 EN**: Executes statement `Flags.setOrigAlign(DL.getABITypeAlign(Arg.Ty));`.
  **L273 CN**: 执行语句 `Flags.setOrigAlign(DL.getABITypeAlign(Arg.Ty));`。
- **L274 EN**: Separates nearby statements for readability.
  **L274 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L275 EN**: Comment documents: `Don't try to use the returned attribute if the argument is marked as`.
  **L275 CN**: 注释说明：`Don't try to use the returned attribute if the argument is marked as`。
- **L276 EN**: Comment documents: `swiftself, since it won't be passed in x0.`.
  **L276 CN**: 注释说明：`swiftself, since it won't be passed in x0.`。
- **L277 EN**: Begins a conditional branch.
  **L277 CN**: 开始一个条件分支。
- **L278 EN**: Executes statement `Flags.setReturned(false);`.
  **L278 CN**: 执行语句 `Flags.setReturned(false);`。
- **L279 EN**: Closes the current scope.
  **L279 CN**: 关闭当前作用域。
- **L280 EN**: Separates nearby statements for readability.
  **L280 CN**: 空行，用于分隔相邻语句并提升可读性。

### Lines 281-300

````cpp
template void
CallLowering::setArgFlags<Function>(CallLowering::ArgInfo &Arg, unsigned OpIdx,
                                    const DataLayout &DL,
                                    const Function &FuncInfo) const;

template void
CallLowering::setArgFlags<CallBase>(CallLowering::ArgInfo &Arg, unsigned OpIdx,
                                    const DataLayout &DL,
                                    const CallBase &FuncInfo) const;

void CallLowering::splitToValueTypes(const ArgInfo &OrigArg,
                                     SmallVectorImpl<ArgInfo> &SplitArgs,
                                     const DataLayout &DL,
                                     CallingConv::ID CallConv,
                                     SmallVectorImpl<TypeSize> *Offsets) const {
  SmallVector<Type *, 4> SplitTys;
  ComputeValueTypes(DL, OrigArg.Ty, SplitTys, Offsets);

  if (SplitTys.size() == 0)
    return;
````
- **L281 EN**: Continues logic with `template void`.
  **L281 CN**: 继续处理逻辑：`template void`。
- **L282 EN**: Provides part of the signature for `function`.
  **L282 CN**: 给出 `function` 的一部分签名。
- **L283 EN**: Continues logic with `const DataLayout &DL,`.
  **L283 CN**: 继续处理逻辑：`const DataLayout &DL,`。
- **L284 EN**: Executes statement `const Function &FuncInfo) const;`.
  **L284 CN**: 执行语句 `const Function &FuncInfo) const;`。
- **L285 EN**: Separates nearby statements for readability.
  **L285 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L286 EN**: Continues logic with `template void`.
  **L286 CN**: 继续处理逻辑：`template void`。
- **L287 EN**: Provides part of the signature for `function`.
  **L287 CN**: 给出 `function` 的一部分签名。
- **L288 EN**: Continues logic with `const DataLayout &DL,`.
  **L288 CN**: 继续处理逻辑：`const DataLayout &DL,`。
- **L289 EN**: Executes statement `const CallBase &FuncInfo) const;`.
  **L289 CN**: 执行语句 `const CallBase &FuncInfo) const;`。
- **L290 EN**: Separates nearby statements for readability.
  **L290 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L291 EN**: Provides part of the signature for `splitToValueTypes`.
  **L291 CN**: 给出 `splitToValueTypes` 的一部分签名。
- **L292 EN**: Continues logic with `SmallVectorImpl<ArgInfo> &SplitArgs,`.
  **L292 CN**: 继续处理逻辑：`SmallVectorImpl<ArgInfo> &SplitArgs,`。
- **L293 EN**: Continues logic with `const DataLayout &DL,`.
  **L293 CN**: 继续处理逻辑：`const DataLayout &DL,`。
- **L294 EN**: Continues logic with `CallingConv::ID CallConv,`.
  **L294 CN**: 继续处理逻辑：`CallingConv::ID CallConv,`。
- **L295 EN**: Starts block `SmallVectorImpl<TypeSize> *Offsets) const`.
  **L295 CN**: 开始代码块 `SmallVectorImpl<TypeSize> *Offsets) const`。
- **L296 EN**: Executes statement `SmallVector<Type *, 4> SplitTys;`.
  **L296 CN**: 执行语句 `SmallVector<Type *, 4> SplitTys;`。
- **L297 EN**: Executes statement `ComputeValueTypes(DL, OrigArg.Ty, SplitTys, Offsets);`.
  **L297 CN**: 执行语句 `ComputeValueTypes(DL, OrigArg.Ty, SplitTys, Offsets);`。
- **L298 EN**: Separates nearby statements for readability.
  **L298 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L299 EN**: Begins a conditional branch.
  **L299 CN**: 开始一个条件分支。
- **L300 EN**: Returns control to the caller.
  **L300 CN**: 将控制流返回给调用者。

### Lines 301-320

````cpp

  if (SplitTys.size() == 1) {
    // No splitting to do, but we want to replace the original type (e.g. [1 x
    // double] -> double).
    SplitArgs.emplace_back(OrigArg.Regs[0], SplitTys[0], OrigArg.OrigArgIndex,
                           OrigArg.Flags[0], OrigArg.OrigValue);
    return;
  }

  // Create one ArgInfo for each virtual register in the original ArgInfo.
  assert(OrigArg.Regs.size() == SplitTys.size() && "Regs / types mismatch");

  bool NeedsRegBlock = TLI->functionArgumentNeedsConsecutiveRegisters(
      OrigArg.Ty, CallConv, false, DL);
  for (unsigned i = 0, e = SplitTys.size(); i < e; ++i) {
    SplitArgs.emplace_back(OrigArg.Regs[i], SplitTys[i], OrigArg.OrigArgIndex,
                           OrigArg.Flags[0]);
    if (NeedsRegBlock)
      SplitArgs.back().Flags[0].setInConsecutiveRegs();
  }
````
- **L301 EN**: Separates nearby statements for readability.
  **L301 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L302 EN**: Begins a conditional branch.
  **L302 CN**: 开始一个条件分支。
- **L303 EN**: Comment documents: `No splitting to do, but we want to replace the original type (e.g. [1 x`.
  **L303 CN**: 注释说明：`No splitting to do, but we want to replace the original type (e.g. [1 x`。
- **L304 EN**: Comment documents: `double] -> double).`.
  **L304 CN**: 注释说明：`double] -> double).`。
- **L305 EN**: Continues logic with `SplitArgs.emplace_back(OrigArg.Regs[0], SplitTys[0], OrigArg.OrigArgInde…`.
  **L305 CN**: 继续处理逻辑：`SplitArgs.emplace_back(OrigArg.Regs[0], SplitTys[0], OrigArg.OrigArgInde…`。
- **L306 EN**: Executes statement `OrigArg.Flags[0], OrigArg.OrigValue);`.
  **L306 CN**: 执行语句 `OrigArg.Flags[0], OrigArg.OrigValue);`。
- **L307 EN**: Returns control to the caller.
  **L307 CN**: 将控制流返回给调用者。
- **L308 EN**: Closes the current scope.
  **L308 CN**: 关闭当前作用域。
- **L309 EN**: Separates nearby statements for readability.
  **L309 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L310 EN**: Comment documents: `Create one ArgInfo for each virtual register in the original ArgInfo.`.
  **L310 CN**: 注释说明：`Create one ArgInfo for each virtual register in the original ArgInfo.`。
- **L311 EN**: Checks an invariant in debug builds.
  **L311 CN**: 在调试构建中检查一个不变量。
- **L312 EN**: Separates nearby statements for readability.
  **L312 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L313 EN**: Continues logic with `bool NeedsRegBlock = TLI->functionArgumentNeedsConsecutiveRegisters(`.
  **L313 CN**: 继续处理逻辑：`bool NeedsRegBlock = TLI->functionArgumentNeedsConsecutiveRegisters(`。
- **L314 EN**: Executes statement `OrigArg.Ty, CallConv, false, DL);`.
  **L314 CN**: 执行语句 `OrigArg.Ty, CallConv, false, DL);`。
- **L315 EN**: Starts a loop over a sequence or range.
  **L315 CN**: 开始遍历序列或范围的循环。
- **L316 EN**: Continues logic with `SplitArgs.emplace_back(OrigArg.Regs[i], SplitTys[i], OrigArg.OrigArgInde…`.
  **L316 CN**: 继续处理逻辑：`SplitArgs.emplace_back(OrigArg.Regs[i], SplitTys[i], OrigArg.OrigArgInde…`。
- **L317 EN**: Executes statement `OrigArg.Flags[0]);`.
  **L317 CN**: 执行语句 `OrigArg.Flags[0]);`。
- **L318 EN**: Begins a conditional branch.
  **L318 CN**: 开始一个条件分支。
- **L319 EN**: Executes statement `SplitArgs.back().Flags[0].setInConsecutiveRegs();`.
  **L319 CN**: 执行语句 `SplitArgs.back().Flags[0].setInConsecutiveRegs();`。
- **L320 EN**: Closes the current scope.
  **L320 CN**: 关闭当前作用域。

### Lines 321-340

````cpp

  SplitArgs.back().Flags[0].setInConsecutiveRegsLast();
}

/// Pack values \p SrcRegs to cover the vector type result \p DstRegs.
static MachineInstrBuilder
mergeVectorRegsToResultRegs(MachineIRBuilder &B, ArrayRef<Register> DstRegs,
                            ArrayRef<Register> SrcRegs) {
  MachineRegisterInfo &MRI = *B.getMRI();
  LLT LLTy = MRI.getType(DstRegs[0]);
  LLT PartLLT = MRI.getType(SrcRegs[0]);

  // Deal with v3s16 split into v2s16
  LLT LCMTy = getCoverTy(LLTy, PartLLT);
  if (LCMTy == LLTy) {
    // Common case where no padding is needed.
    assert(DstRegs.size() == 1);

    SmallVector<Register, 8> ConcatRegs(SrcRegs.size());
    llvm::copy(SrcRegs, ConcatRegs.begin());
````
- **L321 EN**: Separates nearby statements for readability.
  **L321 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L322 EN**: Executes statement `SplitArgs.back().Flags[0].setInConsecutiveRegsLast();`.
  **L322 CN**: 执行语句 `SplitArgs.back().Flags[0].setInConsecutiveRegsLast();`。
- **L323 EN**: Closes the current scope.
  **L323 CN**: 关闭当前作用域。
- **L324 EN**: Separates nearby statements for readability.
  **L324 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L325 EN**: Comment documents: `Pack values \p SrcRegs to cover the vector type result \p DstRegs.`.
  **L325 CN**: 注释说明：`Pack values \p SrcRegs to cover the vector type result \p DstRegs.`。
- **L326 EN**: Continues logic with `static MachineInstrBuilder`.
  **L326 CN**: 继续处理逻辑：`static MachineInstrBuilder`。
- **L327 EN**: Continues logic with `mergeVectorRegsToResultRegs(MachineIRBuilder &B, ArrayRef<Register> DstR…`.
  **L327 CN**: 继续处理逻辑：`mergeVectorRegsToResultRegs(MachineIRBuilder &B, ArrayRef<Register> DstR…`。
- **L328 EN**: Starts block `ArrayRef<Register> SrcRegs)`.
  **L328 CN**: 开始代码块 `ArrayRef<Register> SrcRegs)`。
- **L329 EN**: Assigns or initializes `MachineRegisterInfo &MRI`.
  **L329 CN**: 对 `MachineRegisterInfo &MRI` 进行赋值或初始化。
- **L330 EN**: Assigns or initializes `LLT LLTy`.
  **L330 CN**: 对 `LLT LLTy` 进行赋值或初始化。
- **L331 EN**: Assigns or initializes `LLT PartLLT`.
  **L331 CN**: 对 `LLT PartLLT` 进行赋值或初始化。
- **L332 EN**: Separates nearby statements for readability.
  **L332 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L333 EN**: Comment documents: `Deal with v3s16 split into v2s16`.
  **L333 CN**: 注释说明：`Deal with v3s16 split into v2s16`。
- **L334 EN**: Assigns or initializes `LLT LCMTy`.
  **L334 CN**: 对 `LLT LCMTy` 进行赋值或初始化。
- **L335 EN**: Begins a conditional branch.
  **L335 CN**: 开始一个条件分支。
- **L336 EN**: Comment documents: `Common case where no padding is needed.`.
  **L336 CN**: 注释说明：`Common case where no padding is needed.`。
- **L337 EN**: Checks an invariant in debug builds.
  **L337 CN**: 在调试构建中检查一个不变量。
- **L338 EN**: Separates nearby statements for readability.
  **L338 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L339 EN**: Declares function or method `ConcatRegs`.
  **L339 CN**: 声明函数或方法 `ConcatRegs`。
- **L340 EN**: Declares function or method `copy`.
  **L340 CN**: 声明函数或方法 `copy`。

### Lines 341-360

````cpp

    if (LLTy.getScalarType() != PartLLT.getScalarType())
      for (size_t I = 0, E = SrcRegs.size(); I != E; ++I) {
        auto BitcastDst =
            MRI.getType(SrcRegs[I]).changeElementType(LLTy.getScalarType());
        ConcatRegs[I] = B.buildBitcast(BitcastDst, SrcRegs[I]).getReg(0);
      }

    return B.buildConcatVectors(DstRegs[0], ConcatRegs);
  }

  // We need to create an unmerge to the result registers, which may require
  // widening the original value.
  Register UnmergeSrcReg;
  if (LCMTy.getSizeInBits() != PartLLT.getSizeInBits()) {
    assert(DstRegs.size() == 1);
    return B.buildDeleteTrailingVectorElements(
        DstRegs[0], B.buildMergeLikeInstr(LCMTy, SrcRegs));
  } else {
    // We don't need to widen anything if we're extracting a scalar which was
````
- **L341 EN**: Separates nearby statements for readability.
  **L341 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L342 EN**: Begins a conditional branch.
  **L342 CN**: 开始一个条件分支。
- **L343 EN**: Starts a loop over a sequence or range.
  **L343 CN**: 开始遍历序列或范围的循环。
- **L344 EN**: Continues logic with `auto BitcastDst =`.
  **L344 CN**: 继续处理逻辑：`auto BitcastDst =`。
- **L345 EN**: Executes statement `MRI.getType(SrcRegs[I]).changeElementType(LLTy.getScalarType());`.
  **L345 CN**: 执行语句 `MRI.getType(SrcRegs[I]).changeElementType(LLTy.getScalarType());`。
- **L346 EN**: Assigns or initializes `ConcatRegs[I]`.
  **L346 CN**: 对 `ConcatRegs[I]` 进行赋值或初始化。
- **L347 EN**: Closes the current scope.
  **L347 CN**: 关闭当前作用域。
- **L348 EN**: Separates nearby statements for readability.
  **L348 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L349 EN**: Returns `B.buildConcatVectors(DstRegs[0], ConcatRegs)` to the caller.
  **L349 CN**: 向调用者返回 `B.buildConcatVectors(DstRegs[0], ConcatRegs)`。
- **L350 EN**: Closes the current scope.
  **L350 CN**: 关闭当前作用域。
- **L351 EN**: Separates nearby statements for readability.
  **L351 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L352 EN**: Comment documents: `We need to create an unmerge to the result registers, which may require`.
  **L352 CN**: 注释说明：`We need to create an unmerge to the result registers, which may require`。
- **L353 EN**: Comment documents: `widening the original value.`.
  **L353 CN**: 注释说明：`widening the original value.`。
- **L354 EN**: Executes statement `Register UnmergeSrcReg;`.
  **L354 CN**: 执行语句 `Register UnmergeSrcReg;`。
- **L355 EN**: Begins a conditional branch.
  **L355 CN**: 开始一个条件分支。
- **L356 EN**: Checks an invariant in debug builds.
  **L356 CN**: 在调试构建中检查一个不变量。
- **L357 EN**: Returns `B.buildDeleteTrailingVectorElements(` to the caller.
  **L357 CN**: 向调用者返回 `B.buildDeleteTrailingVectorElements(`。
- **L358 EN**: Executes statement `DstRegs[0], B.buildMergeLikeInstr(LCMTy, SrcRegs));`.
  **L358 CN**: 执行语句 `DstRegs[0], B.buildMergeLikeInstr(LCMTy, SrcRegs));`。
- **L359 EN**: Starts block `} else`.
  **L359 CN**: 开始代码块 `} else`。
- **L360 EN**: Comment documents: `We don't need to widen anything if we're extracting a scalar which was`.
  **L360 CN**: 注释说明：`We don't need to widen anything if we're extracting a scalar which was`。

### Lines 361-380

````cpp
    // promoted to a vector e.g. s8 -> v4s8 -> s8
    assert(SrcRegs.size() == 1);
    UnmergeSrcReg = SrcRegs[0];
  }

  size_t NumDst = LCMTy.getSizeInBits() / LLTy.getSizeInBits();

  SmallVector<Register, 8> PadDstRegs(NumDst);
  llvm::copy(DstRegs, PadDstRegs.begin());

  // Create the excess dead defs for the unmerge.
  for (size_t I = DstRegs.size(); I != NumDst; ++I)
    PadDstRegs[I] = MRI.createGenericVirtualRegister(LLTy);

  if (PartLLT != LCMTy)
    UnmergeSrcReg = B.buildBitcast(LCMTy, UnmergeSrcReg).getReg(0);

  if (PadDstRegs.size() == 1)
    return B.buildDeleteTrailingVectorElements(DstRegs[0], UnmergeSrcReg);
  return B.buildUnmerge(PadDstRegs, UnmergeSrcReg);
````
- **L361 EN**: Comment documents: `promoted to a vector e.g. s8 -> v4s8 -> s8`.
  **L361 CN**: 注释说明：`promoted to a vector e.g. s8 -> v4s8 -> s8`。
- **L362 EN**: Checks an invariant in debug builds.
  **L362 CN**: 在调试构建中检查一个不变量。
- **L363 EN**: Assigns or initializes `UnmergeSrcReg`.
  **L363 CN**: 对 `UnmergeSrcReg` 进行赋值或初始化。
- **L364 EN**: Closes the current scope.
  **L364 CN**: 关闭当前作用域。
- **L365 EN**: Separates nearby statements for readability.
  **L365 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L366 EN**: Assigns or initializes `size_t NumDst`.
  **L366 CN**: 对 `size_t NumDst` 进行赋值或初始化。
- **L367 EN**: Separates nearby statements for readability.
  **L367 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L368 EN**: Declares function or method `PadDstRegs`.
  **L368 CN**: 声明函数或方法 `PadDstRegs`。
- **L369 EN**: Declares function or method `copy`.
  **L369 CN**: 声明函数或方法 `copy`。
- **L370 EN**: Separates nearby statements for readability.
  **L370 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L371 EN**: Comment documents: `Create the excess dead defs for the unmerge.`.
  **L371 CN**: 注释说明：`Create the excess dead defs for the unmerge.`。
- **L372 EN**: Starts a loop over a sequence or range.
  **L372 CN**: 开始遍历序列或范围的循环。
- **L373 EN**: Assigns or initializes `PadDstRegs[I]`.
  **L373 CN**: 对 `PadDstRegs[I]` 进行赋值或初始化。
- **L374 EN**: Separates nearby statements for readability.
  **L374 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L375 EN**: Begins a conditional branch.
  **L375 CN**: 开始一个条件分支。
- **L376 EN**: Assigns or initializes `UnmergeSrcReg`.
  **L376 CN**: 对 `UnmergeSrcReg` 进行赋值或初始化。
- **L377 EN**: Separates nearby statements for readability.
  **L377 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L378 EN**: Begins a conditional branch.
  **L378 CN**: 开始一个条件分支。
- **L379 EN**: Returns `B.buildDeleteTrailingVectorElements(DstRegs[0], UnmergeSrcReg)` to the caller.
  **L379 CN**: 向调用者返回 `B.buildDeleteTrailingVectorElements(DstRegs[0], UnmergeSrcReg)`。
- **L380 EN**: Returns `B.buildUnmerge(PadDstRegs, UnmergeSrcReg)` to the caller.
  **L380 CN**: 向调用者返回 `B.buildUnmerge(PadDstRegs, UnmergeSrcReg)`。

### Lines 381-400

````cpp
}

void CallLowering::buildCopyFromRegs(MachineIRBuilder &B,
                                     ArrayRef<Register> OrigRegs,
                                     ArrayRef<Register> Regs, LLT LLTy,
                                     LLT PartLLT, const ISD::ArgFlagsTy Flags) {
  MachineRegisterInfo &MRI = *B.getMRI();

  if (PartLLT == LLTy) {
    // We should have avoided introducing a new virtual register, and just
    // directly assigned here.
    assert(OrigRegs[0] == Regs[0]);
    return;
  }

  if (PartLLT.getSizeInBits() == LLTy.getSizeInBits() && OrigRegs.size() == 1 &&
      Regs.size() == 1) {
    B.buildBitcast(OrigRegs[0], Regs[0]);
    return;
  }
````
- **L381 EN**: Closes the current scope.
  **L381 CN**: 关闭当前作用域。
- **L382 EN**: Separates nearby statements for readability.
  **L382 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L383 EN**: Provides part of the signature for `buildCopyFromRegs`.
  **L383 CN**: 给出 `buildCopyFromRegs` 的一部分签名。
- **L384 EN**: Continues logic with `ArrayRef<Register> OrigRegs,`.
  **L384 CN**: 继续处理逻辑：`ArrayRef<Register> OrigRegs,`。
- **L385 EN**: Continues logic with `ArrayRef<Register> Regs, LLT LLTy,`.
  **L385 CN**: 继续处理逻辑：`ArrayRef<Register> Regs, LLT LLTy,`。
- **L386 EN**: Starts block `LLT PartLLT, const ISD::ArgFlagsTy Flags)`.
  **L386 CN**: 开始代码块 `LLT PartLLT, const ISD::ArgFlagsTy Flags)`。
- **L387 EN**: Assigns or initializes `MachineRegisterInfo &MRI`.
  **L387 CN**: 对 `MachineRegisterInfo &MRI` 进行赋值或初始化。
- **L388 EN**: Separates nearby statements for readability.
  **L388 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L389 EN**: Begins a conditional branch.
  **L389 CN**: 开始一个条件分支。
- **L390 EN**: Comment documents: `We should have avoided introducing a new virtual register, and just`.
  **L390 CN**: 注释说明：`We should have avoided introducing a new virtual register, and just`。
- **L391 EN**: Comment documents: `directly assigned here.`.
  **L391 CN**: 注释说明：`directly assigned here.`。
- **L392 EN**: Checks an invariant in debug builds.
  **L392 CN**: 在调试构建中检查一个不变量。
- **L393 EN**: Returns control to the caller.
  **L393 CN**: 将控制流返回给调用者。
- **L394 EN**: Closes the current scope.
  **L394 CN**: 关闭当前作用域。
- **L395 EN**: Separates nearby statements for readability.
  **L395 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L396 EN**: Begins a conditional branch.
  **L396 CN**: 开始一个条件分支。
- **L397 EN**: Starts block `Regs.size() == 1)`.
  **L397 CN**: 开始代码块 `Regs.size() == 1)`。
- **L398 EN**: Executes statement `B.buildBitcast(OrigRegs[0], Regs[0]);`.
  **L398 CN**: 执行语句 `B.buildBitcast(OrigRegs[0], Regs[0]);`。
- **L399 EN**: Returns control to the caller.
  **L399 CN**: 将控制流返回给调用者。
- **L400 EN**: Closes the current scope.
  **L400 CN**: 关闭当前作用域。

### Lines 401-420

````cpp

  // A vector PartLLT needs extending to LLTy's element size.
  // E.g. <2 x s64> = G_SEXT <2 x s32>.
  if (PartLLT.isVector() == LLTy.isVector() &&
      PartLLT.getScalarSizeInBits() > LLTy.getScalarSizeInBits() &&
      (!PartLLT.isVector() ||
       PartLLT.getElementCount() == LLTy.getElementCount()) &&
      OrigRegs.size() == 1 && Regs.size() == 1) {
    Register SrcReg = Regs[0];

    LLT LocTy = MRI.getType(SrcReg);

    if (Flags.isSExt()) {
      SrcReg = B.buildAssertSExt(LocTy, SrcReg, LLTy.getScalarSizeInBits())
                   .getReg(0);
    } else if (Flags.isZExt()) {
      SrcReg = B.buildAssertZExt(LocTy, SrcReg, LLTy.getScalarSizeInBits())
                   .getReg(0);
    }

````
- **L401 EN**: Separates nearby statements for readability.
  **L401 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L402 EN**: Comment documents: `A vector PartLLT needs extending to LLTy's element size.`.
  **L402 CN**: 注释说明：`A vector PartLLT needs extending to LLTy's element size.`。
- **L403 EN**: Comment documents: `E.g. <2 x s64> = G_SEXT <2 x s32>.`.
  **L403 CN**: 注释说明：`E.g. <2 x s64> = G_SEXT <2 x s32>.`。
- **L404 EN**: Begins a conditional branch.
  **L404 CN**: 开始一个条件分支。
- **L405 EN**: Continues logic with `PartLLT.getScalarSizeInBits() > LLTy.getScalarSizeInBits() &&`.
  **L405 CN**: 继续处理逻辑：`PartLLT.getScalarSizeInBits() > LLTy.getScalarSizeInBits() &&`。
- **L406 EN**: Continues logic with `(!PartLLT.isVector() ||`.
  **L406 CN**: 继续处理逻辑：`(!PartLLT.isVector() ||`。
- **L407 EN**: Continues logic with `PartLLT.getElementCount() == LLTy.getElementCount()) &&`.
  **L407 CN**: 继续处理逻辑：`PartLLT.getElementCount() == LLTy.getElementCount()) &&`。
- **L408 EN**: Starts block `OrigRegs.size() == 1 && Regs.size() == 1)`.
  **L408 CN**: 开始代码块 `OrigRegs.size() == 1 && Regs.size() == 1)`。
- **L409 EN**: Assigns or initializes `Register SrcReg`.
  **L409 CN**: 对 `Register SrcReg` 进行赋值或初始化。
- **L410 EN**: Separates nearby statements for readability.
  **L410 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L411 EN**: Assigns or initializes `LLT LocTy`.
  **L411 CN**: 对 `LLT LocTy` 进行赋值或初始化。
- **L412 EN**: Separates nearby statements for readability.
  **L412 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L413 EN**: Begins a conditional branch.
  **L413 CN**: 开始一个条件分支。
- **L414 EN**: Continues logic with `SrcReg = B.buildAssertSExt(LocTy, SrcReg, LLTy.getScalarSizeInBits())`.
  **L414 CN**: 继续处理逻辑：`SrcReg = B.buildAssertSExt(LocTy, SrcReg, LLTy.getScalarSizeInBits())`。
- **L415 EN**: Executes statement `.getReg(0);`.
  **L415 CN**: 执行语句 `.getReg(0);`。
- **L416 EN**: Starts block `} else if (Flags.isZExt())`.
  **L416 CN**: 开始代码块 `} else if (Flags.isZExt())`。
- **L417 EN**: Continues logic with `SrcReg = B.buildAssertZExt(LocTy, SrcReg, LLTy.getScalarSizeInBits())`.
  **L417 CN**: 继续处理逻辑：`SrcReg = B.buildAssertZExt(LocTy, SrcReg, LLTy.getScalarSizeInBits())`。
- **L418 EN**: Executes statement `.getReg(0);`.
  **L418 CN**: 执行语句 `.getReg(0);`。
- **L419 EN**: Closes the current scope.
  **L419 CN**: 关闭当前作用域。
- **L420 EN**: Separates nearby statements for readability.
  **L420 CN**: 空行，用于分隔相邻语句并提升可读性。

### Lines 421-440

````cpp
    // Sometimes pointers are passed zero extended.
    LLT OrigTy = MRI.getType(OrigRegs[0]);
    if (OrigTy.isPointer()) {
      LLT IntPtrTy = LLT::scalar(OrigTy.getSizeInBits());
      B.buildIntToPtr(OrigRegs[0], B.buildTrunc(IntPtrTy, SrcReg));
      return;
    }

    B.buildTrunc(OrigRegs[0], SrcReg);
    return;
  }

  if (!LLTy.isVector() && !PartLLT.isVector()) {
    assert(OrigRegs.size() == 1);
    LLT OrigTy = MRI.getType(OrigRegs[0]);

    unsigned SrcSize = PartLLT.getSizeInBits().getFixedValue() * Regs.size();
    if (SrcSize == OrigTy.getSizeInBits())
      B.buildMergeValues(OrigRegs[0], Regs);
    else {
````
- **L421 EN**: Comment documents: `Sometimes pointers are passed zero extended.`.
  **L421 CN**: 注释说明：`Sometimes pointers are passed zero extended.`。
- **L422 EN**: Assigns or initializes `LLT OrigTy`.
  **L422 CN**: 对 `LLT OrigTy` 进行赋值或初始化。
- **L423 EN**: Begins a conditional branch.
  **L423 CN**: 开始一个条件分支。
- **L424 EN**: Declares function or method `scalar`.
  **L424 CN**: 声明函数或方法 `scalar`。
- **L425 EN**: Executes statement `B.buildIntToPtr(OrigRegs[0], B.buildTrunc(IntPtrTy, SrcReg));`.
  **L425 CN**: 执行语句 `B.buildIntToPtr(OrigRegs[0], B.buildTrunc(IntPtrTy, SrcReg));`。
- **L426 EN**: Returns control to the caller.
  **L426 CN**: 将控制流返回给调用者。
- **L427 EN**: Closes the current scope.
  **L427 CN**: 关闭当前作用域。
- **L428 EN**: Separates nearby statements for readability.
  **L428 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L429 EN**: Executes statement `B.buildTrunc(OrigRegs[0], SrcReg);`.
  **L429 CN**: 执行语句 `B.buildTrunc(OrigRegs[0], SrcReg);`。
- **L430 EN**: Returns control to the caller.
  **L430 CN**: 将控制流返回给调用者。
- **L431 EN**: Closes the current scope.
  **L431 CN**: 关闭当前作用域。
- **L432 EN**: Separates nearby statements for readability.
  **L432 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L433 EN**: Begins a conditional branch.
  **L433 CN**: 开始一个条件分支。
- **L434 EN**: Checks an invariant in debug builds.
  **L434 CN**: 在调试构建中检查一个不变量。
- **L435 EN**: Assigns or initializes `LLT OrigTy`.
  **L435 CN**: 对 `LLT OrigTy` 进行赋值或初始化。
- **L436 EN**: Separates nearby statements for readability.
  **L436 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L437 EN**: Assigns or initializes `unsigned SrcSize`.
  **L437 CN**: 对 `unsigned SrcSize` 进行赋值或初始化。
- **L438 EN**: Begins a conditional branch.
  **L438 CN**: 开始一个条件分支。
- **L439 EN**: Executes statement `B.buildMergeValues(OrigRegs[0], Regs);`.
  **L439 CN**: 执行语句 `B.buildMergeValues(OrigRegs[0], Regs);`。
- **L440 EN**: Handles the fallback branch.
  **L440 CN**: 处理兜底分支。

### Lines 441-460

````cpp
      auto Widened = B.buildMergeLikeInstr(LLT::scalar(SrcSize), Regs);
      B.buildTrunc(OrigRegs[0], Widened);
    }

    return;
  }

  if (PartLLT.isVector()) {
    assert(OrigRegs.size() == 1);
    SmallVector<Register> CastRegs(Regs);

    // If PartLLT is a mismatched vector in both number of elements and element
    // size, e.g. PartLLT == v2s64 and LLTy is v3s32, then first coerce it to
    // have the same elt type, i.e. v4s32.
    // TODO: Extend this coersion to element multiples other than just 2.
    if (TypeSize::isKnownGT(PartLLT.getSizeInBits(), LLTy.getSizeInBits()) &&
        PartLLT.getScalarSizeInBits() == LLTy.getScalarSizeInBits() * 2 &&
        Regs.size() == 1) {
      LLT NewTy = PartLLT.changeElementType(LLTy.getElementType())
                      .changeElementCount(PartLLT.getElementCount() * 2);
````
- **L441 EN**: Declares function or method `buildMergeLikeInstr`.
  **L441 CN**: 声明函数或方法 `buildMergeLikeInstr`。
- **L442 EN**: Executes statement `B.buildTrunc(OrigRegs[0], Widened);`.
  **L442 CN**: 执行语句 `B.buildTrunc(OrigRegs[0], Widened);`。
- **L443 EN**: Closes the current scope.
  **L443 CN**: 关闭当前作用域。
- **L444 EN**: Separates nearby statements for readability.
  **L444 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L445 EN**: Returns control to the caller.
  **L445 CN**: 将控制流返回给调用者。
- **L446 EN**: Closes the current scope.
  **L446 CN**: 关闭当前作用域。
- **L447 EN**: Separates nearby statements for readability.
  **L447 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L448 EN**: Begins a conditional branch.
  **L448 CN**: 开始一个条件分支。
- **L449 EN**: Checks an invariant in debug builds.
  **L449 CN**: 在调试构建中检查一个不变量。
- **L450 EN**: Declares function or method `CastRegs`.
  **L450 CN**: 声明函数或方法 `CastRegs`。
- **L451 EN**: Separates nearby statements for readability.
  **L451 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L452 EN**: Comment documents: `If PartLLT is a mismatched vector in both number of elements and element`.
  **L452 CN**: 注释说明：`If PartLLT is a mismatched vector in both number of elements and element`。
- **L453 EN**: Comment documents: `size, e.g. PartLLT == v2s64 and LLTy is v3s32, then first coerce it to`.
  **L453 CN**: 注释说明：`size, e.g. PartLLT == v2s64 and LLTy is v3s32, then first coerce it to`。
- **L454 EN**: Comment documents: `have the same elt type, i.e. v4s32.`.
  **L454 CN**: 注释说明：`have the same elt type, i.e. v4s32.`。
- **L455 EN**: Comment documents: `TODO: Extend this coersion to element multiples other than just 2.`.
  **L455 CN**: 注释说明：`TODO: Extend this coersion to element multiples other than just 2.`。
- **L456 EN**: Begins a conditional branch.
  **L456 CN**: 开始一个条件分支。
- **L457 EN**: Continues logic with `PartLLT.getScalarSizeInBits() == LLTy.getScalarSizeInBits() * 2 &&`.
  **L457 CN**: 继续处理逻辑：`PartLLT.getScalarSizeInBits() == LLTy.getScalarSizeInBits() * 2 &&`。
- **L458 EN**: Starts block `Regs.size() == 1)`.
  **L458 CN**: 开始代码块 `Regs.size() == 1)`。
- **L459 EN**: Continues logic with `LLT NewTy = PartLLT.changeElementType(LLTy.getElementType())`.
  **L459 CN**: 继续处理逻辑：`LLT NewTy = PartLLT.changeElementType(LLTy.getElementType())`。
- **L460 EN**: Executes statement `.changeElementCount(PartLLT.getElementCount() * 2);`.
  **L460 CN**: 执行语句 `.changeElementCount(PartLLT.getElementCount() * 2);`。

### Lines 461-480

````cpp
      CastRegs[0] = B.buildBitcast(NewTy, Regs[0]).getReg(0);
      PartLLT = NewTy;
    }

    if (LLTy.getScalarSizeInBits() == PartLLT.getScalarSizeInBits()) {
      mergeVectorRegsToResultRegs(B, OrigRegs, CastRegs);
    } else {
      unsigned I = 0;
      LLT GCDTy = getGCDType(LLTy, PartLLT);

      // We are both splitting a vector, and bitcasting its element types. Cast
      // the source pieces into the appropriate number of pieces with the result
      // element type.
      for (Register SrcReg : CastRegs)
        CastRegs[I++] = B.buildBitcast(GCDTy, SrcReg).getReg(0);
      mergeVectorRegsToResultRegs(B, OrigRegs, CastRegs);
    }

    return;
  }
````
- **L461 EN**: Assigns or initializes `CastRegs[0]`.
  **L461 CN**: 对 `CastRegs[0]` 进行赋值或初始化。
- **L462 EN**: Assigns or initializes `PartLLT`.
  **L462 CN**: 对 `PartLLT` 进行赋值或初始化。
- **L463 EN**: Closes the current scope.
  **L463 CN**: 关闭当前作用域。
- **L464 EN**: Separates nearby statements for readability.
  **L464 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L465 EN**: Begins a conditional branch.
  **L465 CN**: 开始一个条件分支。
- **L466 EN**: Executes statement `mergeVectorRegsToResultRegs(B, OrigRegs, CastRegs);`.
  **L466 CN**: 执行语句 `mergeVectorRegsToResultRegs(B, OrigRegs, CastRegs);`。
- **L467 EN**: Starts block `} else`.
  **L467 CN**: 开始代码块 `} else`。
- **L468 EN**: Assigns or initializes `unsigned I`.
  **L468 CN**: 对 `unsigned I` 进行赋值或初始化。
- **L469 EN**: Assigns or initializes `LLT GCDTy`.
  **L469 CN**: 对 `LLT GCDTy` 进行赋值或初始化。
- **L470 EN**: Separates nearby statements for readability.
  **L470 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L471 EN**: Comment documents: `We are both splitting a vector, and bitcasting its element types. Cast`.
  **L471 CN**: 注释说明：`We are both splitting a vector, and bitcasting its element types. Cast`。
- **L472 EN**: Comment documents: `the source pieces into the appropriate number of pieces with the result`.
  **L472 CN**: 注释说明：`the source pieces into the appropriate number of pieces with the result`。
- **L473 EN**: Comment documents: `element type.`.
  **L473 CN**: 注释说明：`element type.`。
- **L474 EN**: Starts a loop over a sequence or range.
  **L474 CN**: 开始遍历序列或范围的循环。
- **L475 EN**: Assigns or initializes `CastRegs[I++]`.
  **L475 CN**: 对 `CastRegs[I++]` 进行赋值或初始化。
- **L476 EN**: Executes statement `mergeVectorRegsToResultRegs(B, OrigRegs, CastRegs);`.
  **L476 CN**: 执行语句 `mergeVectorRegsToResultRegs(B, OrigRegs, CastRegs);`。
- **L477 EN**: Closes the current scope.
  **L477 CN**: 关闭当前作用域。
- **L478 EN**: Separates nearby statements for readability.
  **L478 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L479 EN**: Returns control to the caller.
  **L479 CN**: 将控制流返回给调用者。
- **L480 EN**: Closes the current scope.
  **L480 CN**: 关闭当前作用域。

### Lines 481-500

````cpp

  assert(LLTy.isVector() && !PartLLT.isVector());

  LLT DstEltTy = LLTy.getElementType();

  // Pointer information was discarded. We'll need to coerce some register types
  // to avoid violating type constraints.
  LLT RealDstEltTy = MRI.getType(OrigRegs[0]).getElementType();

  assert(DstEltTy.getSizeInBits() == RealDstEltTy.getSizeInBits());

  if (DstEltTy == PartLLT) {
    // Vector was trivially scalarized.

    if (RealDstEltTy.isPointer()) {
      for (Register Reg : Regs)
        MRI.setType(Reg, RealDstEltTy);
    }

    B.buildBuildVector(OrigRegs[0], Regs);
````
- **L481 EN**: Separates nearby statements for readability.
  **L481 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L482 EN**: Checks an invariant in debug builds.
  **L482 CN**: 在调试构建中检查一个不变量。
- **L483 EN**: Separates nearby statements for readability.
  **L483 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L484 EN**: Assigns or initializes `LLT DstEltTy`.
  **L484 CN**: 对 `LLT DstEltTy` 进行赋值或初始化。
- **L485 EN**: Separates nearby statements for readability.
  **L485 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L486 EN**: Comment documents: `Pointer information was discarded. We'll need to coerce some register ty…`.
  **L486 CN**: 注释说明：`Pointer information was discarded. We'll need to coerce some register ty…`。
- **L487 EN**: Comment documents: `to avoid violating type constraints.`.
  **L487 CN**: 注释说明：`to avoid violating type constraints.`。
- **L488 EN**: Assigns or initializes `LLT RealDstEltTy`.
  **L488 CN**: 对 `LLT RealDstEltTy` 进行赋值或初始化。
- **L489 EN**: Separates nearby statements for readability.
  **L489 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L490 EN**: Checks an invariant in debug builds.
  **L490 CN**: 在调试构建中检查一个不变量。
- **L491 EN**: Separates nearby statements for readability.
  **L491 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L492 EN**: Begins a conditional branch.
  **L492 CN**: 开始一个条件分支。
- **L493 EN**: Comment documents: `Vector was trivially scalarized.`.
  **L493 CN**: 注释说明：`Vector was trivially scalarized.`。
- **L494 EN**: Separates nearby statements for readability.
  **L494 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L495 EN**: Begins a conditional branch.
  **L495 CN**: 开始一个条件分支。
- **L496 EN**: Starts a loop over a sequence or range.
  **L496 CN**: 开始遍历序列或范围的循环。
- **L497 EN**: Executes statement `MRI.setType(Reg, RealDstEltTy);`.
  **L497 CN**: 执行语句 `MRI.setType(Reg, RealDstEltTy);`。
- **L498 EN**: Closes the current scope.
  **L498 CN**: 关闭当前作用域。
- **L499 EN**: Separates nearby statements for readability.
  **L499 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L500 EN**: Executes statement `B.buildBuildVector(OrigRegs[0], Regs);`.
  **L500 CN**: 执行语句 `B.buildBuildVector(OrigRegs[0], Regs);`。

### Lines 501-520

````cpp
  } else if (DstEltTy.getSizeInBits() > PartLLT.getSizeInBits()) {
    // Deal with vector with 64-bit elements decomposed to 32-bit
    // registers. Need to create intermediate 64-bit elements.
    SmallVector<Register, 8> EltMerges;
    int PartsPerElt =
        divideCeil(DstEltTy.getSizeInBits(), PartLLT.getSizeInBits());
    LLT ExtendedPartTy = LLT::scalar(PartLLT.getSizeInBits() * PartsPerElt);

    for (int I = 0, NumElts = LLTy.getNumElements(); I != NumElts; ++I) {
      auto Merge =
          B.buildMergeLikeInstr(ExtendedPartTy, Regs.take_front(PartsPerElt));
      if (ExtendedPartTy.getSizeInBits() > RealDstEltTy.getSizeInBits())
        Merge = B.buildTrunc(RealDstEltTy, Merge);
      // Fix the type in case this is really a vector of pointers.
      MRI.setType(Merge.getReg(0), RealDstEltTy);
      EltMerges.push_back(Merge.getReg(0));
      Regs = Regs.drop_front(PartsPerElt);
    }

    B.buildBuildVector(OrigRegs[0], EltMerges);
````
- **L501 EN**: Starts block `} else if (DstEltTy.getSizeInBits() > PartLLT.getSizeInBits())`.
  **L501 CN**: 开始代码块 `} else if (DstEltTy.getSizeInBits() > PartLLT.getSizeInBits())`。
- **L502 EN**: Comment documents: `Deal with vector with 64-bit elements decomposed to 32-bit`.
  **L502 CN**: 注释说明：`Deal with vector with 64-bit elements decomposed to 32-bit`。
- **L503 EN**: Comment documents: `registers. Need to create intermediate 64-bit elements.`.
  **L503 CN**: 注释说明：`registers. Need to create intermediate 64-bit elements.`。
- **L504 EN**: Executes statement `SmallVector<Register, 8> EltMerges;`.
  **L504 CN**: 执行语句 `SmallVector<Register, 8> EltMerges;`。
- **L505 EN**: Continues logic with `int PartsPerElt =`.
  **L505 CN**: 继续处理逻辑：`int PartsPerElt =`。
- **L506 EN**: Executes statement `divideCeil(DstEltTy.getSizeInBits(), PartLLT.getSizeInBits());`.
  **L506 CN**: 执行语句 `divideCeil(DstEltTy.getSizeInBits(), PartLLT.getSizeInBits());`。
- **L507 EN**: Declares function or method `scalar`.
  **L507 CN**: 声明函数或方法 `scalar`。
- **L508 EN**: Separates nearby statements for readability.
  **L508 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L509 EN**: Starts a loop over a sequence or range.
  **L509 CN**: 开始遍历序列或范围的循环。
- **L510 EN**: Continues logic with `auto Merge =`.
  **L510 CN**: 继续处理逻辑：`auto Merge =`。
- **L511 EN**: Executes statement `B.buildMergeLikeInstr(ExtendedPartTy, Regs.take_front(PartsPerElt));`.
  **L511 CN**: 执行语句 `B.buildMergeLikeInstr(ExtendedPartTy, Regs.take_front(PartsPerElt));`。
- **L512 EN**: Begins a conditional branch.
  **L512 CN**: 开始一个条件分支。
- **L513 EN**: Assigns or initializes `Merge`.
  **L513 CN**: 对 `Merge` 进行赋值或初始化。
- **L514 EN**: Comment documents: `Fix the type in case this is really a vector of pointers.`.
  **L514 CN**: 注释说明：`Fix the type in case this is really a vector of pointers.`。
- **L515 EN**: Executes statement `MRI.setType(Merge.getReg(0), RealDstEltTy);`.
  **L515 CN**: 执行语句 `MRI.setType(Merge.getReg(0), RealDstEltTy);`。
- **L516 EN**: Executes statement `EltMerges.push_back(Merge.getReg(0));`.
  **L516 CN**: 执行语句 `EltMerges.push_back(Merge.getReg(0));`。
- **L517 EN**: Assigns or initializes `Regs`.
  **L517 CN**: 对 `Regs` 进行赋值或初始化。
- **L518 EN**: Closes the current scope.
  **L518 CN**: 关闭当前作用域。
- **L519 EN**: Separates nearby statements for readability.
  **L519 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L520 EN**: Executes statement `B.buildBuildVector(OrigRegs[0], EltMerges);`.
  **L520 CN**: 执行语句 `B.buildBuildVector(OrigRegs[0], EltMerges);`。

### Lines 521-540

````cpp
  } else {
    // Vector was split, and elements promoted to a wider type.
    // FIXME: Should handle floating point promotions.
    unsigned NumElts = LLTy.getNumElements();
    LLT BVType = LLT::fixed_vector(NumElts, PartLLT);

    Register BuildVec;
    if (NumElts == Regs.size())
      BuildVec = B.buildBuildVector(BVType, Regs).getReg(0);
    else {
      // Vector elements are packed in the inputs.
      // e.g. we have a <4 x s16> but 2 x s32 in regs.
      assert(NumElts > Regs.size());
      LLT SrcEltTy = MRI.getType(Regs[0]);

      LLT OriginalEltTy = MRI.getType(OrigRegs[0]).getElementType();

      // Input registers contain packed elements.
      // Determine how many elements per reg.
      assert((SrcEltTy.getSizeInBits() % OriginalEltTy.getSizeInBits()) == 0);
````
- **L521 EN**: Starts block `} else`.
  **L521 CN**: 开始代码块 `} else`。
- **L522 EN**: Comment documents: `Vector was split, and elements promoted to a wider type.`.
  **L522 CN**: 注释说明：`Vector was split, and elements promoted to a wider type.`。
- **L523 EN**: Comment documents: `FIXME: Should handle floating point promotions.`.
  **L523 CN**: 注释说明：`FIXME: Should handle floating point promotions.`。
- **L524 EN**: Assigns or initializes `unsigned NumElts`.
  **L524 CN**: 对 `unsigned NumElts` 进行赋值或初始化。
- **L525 EN**: Declares function or method `fixed_vector`.
  **L525 CN**: 声明函数或方法 `fixed_vector`。
- **L526 EN**: Separates nearby statements for readability.
  **L526 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L527 EN**: Executes statement `Register BuildVec;`.
  **L527 CN**: 执行语句 `Register BuildVec;`。
- **L528 EN**: Begins a conditional branch.
  **L528 CN**: 开始一个条件分支。
- **L529 EN**: Assigns or initializes `BuildVec`.
  **L529 CN**: 对 `BuildVec` 进行赋值或初始化。
- **L530 EN**: Handles the fallback branch.
  **L530 CN**: 处理兜底分支。
- **L531 EN**: Comment documents: `Vector elements are packed in the inputs.`.
  **L531 CN**: 注释说明：`Vector elements are packed in the inputs.`。
- **L532 EN**: Comment documents: `e.g. we have a <4 x s16> but 2 x s32 in regs.`.
  **L532 CN**: 注释说明：`e.g. we have a <4 x s16> but 2 x s32 in regs.`。
- **L533 EN**: Checks an invariant in debug builds.
  **L533 CN**: 在调试构建中检查一个不变量。
- **L534 EN**: Assigns or initializes `LLT SrcEltTy`.
  **L534 CN**: 对 `LLT SrcEltTy` 进行赋值或初始化。
- **L535 EN**: Separates nearby statements for readability.
  **L535 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L536 EN**: Assigns or initializes `LLT OriginalEltTy`.
  **L536 CN**: 对 `LLT OriginalEltTy` 进行赋值或初始化。
- **L537 EN**: Separates nearby statements for readability.
  **L537 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L538 EN**: Comment documents: `Input registers contain packed elements.`.
  **L538 CN**: 注释说明：`Input registers contain packed elements.`。
- **L539 EN**: Comment documents: `Determine how many elements per reg.`.
  **L539 CN**: 注释说明：`Determine how many elements per reg.`。
- **L540 EN**: Checks an invariant in debug builds.
  **L540 CN**: 在调试构建中检查一个不变量。

### Lines 541-560

````cpp
      unsigned EltPerReg =
          (SrcEltTy.getSizeInBits() / OriginalEltTy.getSizeInBits());

      SmallVector<Register, 0> BVRegs;
      BVRegs.reserve(Regs.size() * EltPerReg);
      for (Register R : Regs) {
        auto Unmerge = B.buildUnmerge(OriginalEltTy, R);
        for (unsigned K = 0; K < EltPerReg; ++K)
          BVRegs.push_back(B.buildAnyExt(PartLLT, Unmerge.getReg(K)).getReg(0));
      }

      // We may have some more elements in BVRegs, e.g. if we have 2 s32 pieces
      // for a <3 x s16> vector. We should have less than EltPerReg extra items.
      if (BVRegs.size() > NumElts) {
        assert((BVRegs.size() - NumElts) < EltPerReg);
        BVRegs.truncate(NumElts);
      }
      BuildVec = B.buildBuildVector(BVType, BVRegs).getReg(0);
    }
    B.buildTrunc(OrigRegs[0], BuildVec);
````
- **L541 EN**: Continues logic with `unsigned EltPerReg =`.
  **L541 CN**: 继续处理逻辑：`unsigned EltPerReg =`。
- **L542 EN**: Executes statement `(SrcEltTy.getSizeInBits() / OriginalEltTy.getSizeInBits());`.
  **L542 CN**: 执行语句 `(SrcEltTy.getSizeInBits() / OriginalEltTy.getSizeInBits());`。
- **L543 EN**: Separates nearby statements for readability.
  **L543 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L544 EN**: Executes statement `SmallVector<Register, 0> BVRegs;`.
  **L544 CN**: 执行语句 `SmallVector<Register, 0> BVRegs;`。
- **L545 EN**: Executes statement `BVRegs.reserve(Regs.size() * EltPerReg);`.
  **L545 CN**: 执行语句 `BVRegs.reserve(Regs.size() * EltPerReg);`。
- **L546 EN**: Starts a loop over a sequence or range.
  **L546 CN**: 开始遍历序列或范围的循环。
- **L547 EN**: Assigns or initializes `auto Unmerge`.
  **L547 CN**: 对 `auto Unmerge` 进行赋值或初始化。
- **L548 EN**: Starts a loop over a sequence or range.
  **L548 CN**: 开始遍历序列或范围的循环。
- **L549 EN**: Executes statement `BVRegs.push_back(B.buildAnyExt(PartLLT, Unmerge.getReg(K)).getReg(0));`.
  **L549 CN**: 执行语句 `BVRegs.push_back(B.buildAnyExt(PartLLT, Unmerge.getReg(K)).getReg(0));`。
- **L550 EN**: Closes the current scope.
  **L550 CN**: 关闭当前作用域。
- **L551 EN**: Separates nearby statements for readability.
  **L551 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L552 EN**: Comment documents: `We may have some more elements in BVRegs, e.g. if we have 2 s32 pieces`.
  **L552 CN**: 注释说明：`We may have some more elements in BVRegs, e.g. if we have 2 s32 pieces`。
- **L553 EN**: Comment documents: `for a <3 x s16> vector. We should have less than EltPerReg extra items.`.
  **L553 CN**: 注释说明：`for a <3 x s16> vector. We should have less than EltPerReg extra items.`。
- **L554 EN**: Begins a conditional branch.
  **L554 CN**: 开始一个条件分支。
- **L555 EN**: Checks an invariant in debug builds.
  **L555 CN**: 在调试构建中检查一个不变量。
- **L556 EN**: Executes statement `BVRegs.truncate(NumElts);`.
  **L556 CN**: 执行语句 `BVRegs.truncate(NumElts);`。
- **L557 EN**: Closes the current scope.
  **L557 CN**: 关闭当前作用域。
- **L558 EN**: Assigns or initializes `BuildVec`.
  **L558 CN**: 对 `BuildVec` 进行赋值或初始化。
- **L559 EN**: Closes the current scope.
  **L559 CN**: 关闭当前作用域。
- **L560 EN**: Executes statement `B.buildTrunc(OrigRegs[0], BuildVec);`.
  **L560 CN**: 执行语句 `B.buildTrunc(OrigRegs[0], BuildVec);`。

### Lines 561-580

````cpp
  }
}

void CallLowering::buildCopyToRegs(MachineIRBuilder &B,
                                   ArrayRef<Register> DstRegs, Register SrcReg,
                                   LLT SrcTy, LLT PartTy, unsigned ExtendOp) {
  // We could just insert a regular copy, but this is unreachable at the moment.
  assert(SrcTy != PartTy && "identical part types shouldn't reach here");

  const TypeSize PartSize = PartTy.getSizeInBits();

  if (PartSize == SrcTy.getSizeInBits() && DstRegs.size() == 1) {
    // TODO: Handle int<->ptr casts. It just happens the ABI lowering
    // assignments are not pointer aware.
    B.buildBitcast(DstRegs[0], SrcReg);
    return;
  }

  if (PartTy.isVector() == SrcTy.isVector() &&
      PartTy.getScalarSizeInBits() > SrcTy.getScalarSizeInBits()) {
````
- **L561 EN**: Closes the current scope.
  **L561 CN**: 关闭当前作用域。
- **L562 EN**: Closes the current scope.
  **L562 CN**: 关闭当前作用域。
- **L563 EN**: Separates nearby statements for readability.
  **L563 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L564 EN**: Provides part of the signature for `buildCopyToRegs`.
  **L564 CN**: 给出 `buildCopyToRegs` 的一部分签名。
- **L565 EN**: Continues logic with `ArrayRef<Register> DstRegs, Register SrcReg,`.
  **L565 CN**: 继续处理逻辑：`ArrayRef<Register> DstRegs, Register SrcReg,`。
- **L566 EN**: Starts block `LLT SrcTy, LLT PartTy, unsigned ExtendOp)`.
  **L566 CN**: 开始代码块 `LLT SrcTy, LLT PartTy, unsigned ExtendOp)`。
- **L567 EN**: Comment documents: `We could just insert a regular copy, but this is unreachable at the mome…`.
  **L567 CN**: 注释说明：`We could just insert a regular copy, but this is unreachable at the mome…`。
- **L568 EN**: Checks an invariant in debug builds.
  **L568 CN**: 在调试构建中检查一个不变量。
- **L569 EN**: Separates nearby statements for readability.
  **L569 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L570 EN**: Assigns or initializes `const TypeSize PartSize`.
  **L570 CN**: 对 `const TypeSize PartSize` 进行赋值或初始化。
- **L571 EN**: Separates nearby statements for readability.
  **L571 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L572 EN**: Begins a conditional branch.
  **L572 CN**: 开始一个条件分支。
- **L573 EN**: Comment documents: `TODO: Handle int<->ptr casts. It just happens the ABI lowering`.
  **L573 CN**: 注释说明：`TODO: Handle int<->ptr casts. It just happens the ABI lowering`。
- **L574 EN**: Comment documents: `assignments are not pointer aware.`.
  **L574 CN**: 注释说明：`assignments are not pointer aware.`。
- **L575 EN**: Executes statement `B.buildBitcast(DstRegs[0], SrcReg);`.
  **L575 CN**: 执行语句 `B.buildBitcast(DstRegs[0], SrcReg);`。
- **L576 EN**: Returns control to the caller.
  **L576 CN**: 将控制流返回给调用者。
- **L577 EN**: Closes the current scope.
  **L577 CN**: 关闭当前作用域。
- **L578 EN**: Separates nearby statements for readability.
  **L578 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L579 EN**: Begins a conditional branch.
  **L579 CN**: 开始一个条件分支。
- **L580 EN**: Starts block `PartTy.getScalarSizeInBits() > SrcTy.getScalarSizeInBits())`.
  **L580 CN**: 开始代码块 `PartTy.getScalarSizeInBits() > SrcTy.getScalarSizeInBits())`。

### Lines 581-600

````cpp
    assert(DstRegs.size() == 1);
    B.buildInstr(ExtendOp, {DstRegs[0]}, {SrcReg});
    return;
  }

  if (SrcTy.isVector() && !PartTy.isVector() &&
      TypeSize::isKnownGT(PartSize, SrcTy.getElementType().getSizeInBits()) &&
      SrcTy.getElementCount() == ElementCount::getFixed(DstRegs.size())) {
    // Vector was scalarized, and the elements extended.
    auto UnmergeToEltTy = B.buildUnmerge(SrcTy.getElementType(), SrcReg);
    for (int i = 0, e = DstRegs.size(); i != e; ++i)
      B.buildAnyExt(DstRegs[i], UnmergeToEltTy.getReg(i));
    return;
  }

  if (SrcTy.isVector() && PartTy.isVector() &&
      PartTy.getSizeInBits() == SrcTy.getSizeInBits() &&
      ElementCount::isKnownLT(SrcTy.getElementCount(),
                              PartTy.getElementCount())) {
    // A coercion like: v2f32 -> v4f32 or nxv2f32 -> nxv4f32
````
- **L581 EN**: Checks an invariant in debug builds.
  **L581 CN**: 在调试构建中检查一个不变量。
- **L582 EN**: Executes statement `B.buildInstr(ExtendOp, {DstRegs[0]}, {SrcReg});`.
  **L582 CN**: 执行语句 `B.buildInstr(ExtendOp, {DstRegs[0]}, {SrcReg});`。
- **L583 EN**: Returns control to the caller.
  **L583 CN**: 将控制流返回给调用者。
- **L584 EN**: Closes the current scope.
  **L584 CN**: 关闭当前作用域。
- **L585 EN**: Separates nearby statements for readability.
  **L585 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L586 EN**: Begins a conditional branch.
  **L586 CN**: 开始一个条件分支。
- **L587 EN**: Provides part of the signature for `isKnownGT`.
  **L587 CN**: 给出 `isKnownGT` 的一部分签名。
- **L588 EN**: Begins the definition of `getElementCount`.
  **L588 CN**: 开始定义 `getElementCount`。
- **L589 EN**: Comment documents: `Vector was scalarized, and the elements extended.`.
  **L589 CN**: 注释说明：`Vector was scalarized, and the elements extended.`。
- **L590 EN**: Assigns or initializes `auto UnmergeToEltTy`.
  **L590 CN**: 对 `auto UnmergeToEltTy` 进行赋值或初始化。
- **L591 EN**: Starts a loop over a sequence or range.
  **L591 CN**: 开始遍历序列或范围的循环。
- **L592 EN**: Executes statement `B.buildAnyExt(DstRegs[i], UnmergeToEltTy.getReg(i));`.
  **L592 CN**: 执行语句 `B.buildAnyExt(DstRegs[i], UnmergeToEltTy.getReg(i));`。
- **L593 EN**: Returns control to the caller.
  **L593 CN**: 将控制流返回给调用者。
- **L594 EN**: Closes the current scope.
  **L594 CN**: 关闭当前作用域。
- **L595 EN**: Separates nearby statements for readability.
  **L595 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L596 EN**: Begins a conditional branch.
  **L596 CN**: 开始一个条件分支。
- **L597 EN**: Continues logic with `PartTy.getSizeInBits() == SrcTy.getSizeInBits() &&`.
  **L597 CN**: 继续处理逻辑：`PartTy.getSizeInBits() == SrcTy.getSizeInBits() &&`。
- **L598 EN**: Provides part of the signature for `isKnownLT`.
  **L598 CN**: 给出 `isKnownLT` 的一部分签名。
- **L599 EN**: Starts block `PartTy.getElementCount()))`.
  **L599 CN**: 开始代码块 `PartTy.getElementCount()))`。
- **L600 EN**: Comment documents: `A coercion like: v2f32 -> v4f32 or nxv2f32 -> nxv4f32`.
  **L600 CN**: 注释说明：`A coercion like: v2f32 -> v4f32 or nxv2f32 -> nxv4f32`。

### Lines 601-620

````cpp
    Register DstReg = DstRegs.front();
    B.buildPadVectorWithUndefElements(DstReg, SrcReg);
    return;
  }

  LLT GCDTy = getGCDType(SrcTy, PartTy);
  if (GCDTy == PartTy) {
    // If this already evenly divisible, we can create a simple unmerge.
    B.buildUnmerge(DstRegs, SrcReg);
    return;
  }

  if (SrcTy.isVector() && !PartTy.isVector() &&
      SrcTy.getScalarSizeInBits() > PartTy.getSizeInBits()) {
    LLT ExtTy =
        LLT::vector(SrcTy.getElementCount(),
                    LLT::scalar(PartTy.getScalarSizeInBits() * DstRegs.size() /
                                SrcTy.getNumElements()));
    auto Ext = B.buildAnyExt(ExtTy, SrcReg);
    B.buildUnmerge(DstRegs, Ext);
````
- **L601 EN**: Assigns or initializes `Register DstReg`.
  **L601 CN**: 对 `Register DstReg` 进行赋值或初始化。
- **L602 EN**: Executes statement `B.buildPadVectorWithUndefElements(DstReg, SrcReg);`.
  **L602 CN**: 执行语句 `B.buildPadVectorWithUndefElements(DstReg, SrcReg);`。
- **L603 EN**: Returns control to the caller.
  **L603 CN**: 将控制流返回给调用者。
- **L604 EN**: Closes the current scope.
  **L604 CN**: 关闭当前作用域。
- **L605 EN**: Separates nearby statements for readability.
  **L605 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L606 EN**: Assigns or initializes `LLT GCDTy`.
  **L606 CN**: 对 `LLT GCDTy` 进行赋值或初始化。
- **L607 EN**: Begins a conditional branch.
  **L607 CN**: 开始一个条件分支。
- **L608 EN**: Comment documents: `If this already evenly divisible, we can create a simple unmerge.`.
  **L608 CN**: 注释说明：`If this already evenly divisible, we can create a simple unmerge.`。
- **L609 EN**: Executes statement `B.buildUnmerge(DstRegs, SrcReg);`.
  **L609 CN**: 执行语句 `B.buildUnmerge(DstRegs, SrcReg);`。
- **L610 EN**: Returns control to the caller.
  **L610 CN**: 将控制流返回给调用者。
- **L611 EN**: Closes the current scope.
  **L611 CN**: 关闭当前作用域。
- **L612 EN**: Separates nearby statements for readability.
  **L612 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L613 EN**: Begins a conditional branch.
  **L613 CN**: 开始一个条件分支。
- **L614 EN**: Starts block `SrcTy.getScalarSizeInBits() > PartTy.getSizeInBits())`.
  **L614 CN**: 开始代码块 `SrcTy.getScalarSizeInBits() > PartTy.getSizeInBits())`。
- **L615 EN**: Continues logic with `LLT ExtTy =`.
  **L615 CN**: 继续处理逻辑：`LLT ExtTy =`。
- **L616 EN**: Provides part of the signature for `vector`.
  **L616 CN**: 给出 `vector` 的一部分签名。
- **L617 EN**: Provides part of the signature for `scalar`.
  **L617 CN**: 给出 `scalar` 的一部分签名。
- **L618 EN**: Executes statement `SrcTy.getNumElements()));`.
  **L618 CN**: 执行语句 `SrcTy.getNumElements()));`。
- **L619 EN**: Assigns or initializes `auto Ext`.
  **L619 CN**: 对 `auto Ext` 进行赋值或初始化。
- **L620 EN**: Executes statement `B.buildUnmerge(DstRegs, Ext);`.
  **L620 CN**: 执行语句 `B.buildUnmerge(DstRegs, Ext);`。

### Lines 621-640

````cpp
    return;
  }

  MachineRegisterInfo &MRI = *B.getMRI();
  LLT DstTy = MRI.getType(DstRegs[0]);
  LLT CoverTy = getCoverTy(SrcTy, PartTy);
  if (SrcTy.isVector() && DstRegs.size() > 1) {
    TypeSize FullCoverSize =
        DstTy.getSizeInBits().multiplyCoefficientBy(DstRegs.size());

    LLT EltTy = SrcTy.getElementType();
    TypeSize EltSize = EltTy.getSizeInBits();
    if (FullCoverSize.isKnownMultipleOf(EltSize)) {
      TypeSize VecSize = FullCoverSize.divideCoefficientBy(EltSize);
      CoverTy =
          LLT::vector(ElementCount::get(VecSize, VecSize.isScalable()), EltTy);
    }
  }

  if (PartTy.isVector() && CoverTy == PartTy) {
````
- **L621 EN**: Returns control to the caller.
  **L621 CN**: 将控制流返回给调用者。
- **L622 EN**: Closes the current scope.
  **L622 CN**: 关闭当前作用域。
- **L623 EN**: Separates nearby statements for readability.
  **L623 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L624 EN**: Assigns or initializes `MachineRegisterInfo &MRI`.
  **L624 CN**: 对 `MachineRegisterInfo &MRI` 进行赋值或初始化。
- **L625 EN**: Assigns or initializes `LLT DstTy`.
  **L625 CN**: 对 `LLT DstTy` 进行赋值或初始化。
- **L626 EN**: Assigns or initializes `LLT CoverTy`.
  **L626 CN**: 对 `LLT CoverTy` 进行赋值或初始化。
- **L627 EN**: Begins a conditional branch.
  **L627 CN**: 开始一个条件分支。
- **L628 EN**: Continues logic with `TypeSize FullCoverSize =`.
  **L628 CN**: 继续处理逻辑：`TypeSize FullCoverSize =`。
- **L629 EN**: Executes statement `DstTy.getSizeInBits().multiplyCoefficientBy(DstRegs.size());`.
  **L629 CN**: 执行语句 `DstTy.getSizeInBits().multiplyCoefficientBy(DstRegs.size());`。
- **L630 EN**: Separates nearby statements for readability.
  **L630 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L631 EN**: Assigns or initializes `LLT EltTy`.
  **L631 CN**: 对 `LLT EltTy` 进行赋值或初始化。
- **L632 EN**: Assigns or initializes `TypeSize EltSize`.
  **L632 CN**: 对 `TypeSize EltSize` 进行赋值或初始化。
- **L633 EN**: Begins a conditional branch.
  **L633 CN**: 开始一个条件分支。
- **L634 EN**: Assigns or initializes `TypeSize VecSize`.
  **L634 CN**: 对 `TypeSize VecSize` 进行赋值或初始化。
- **L635 EN**: Continues logic with `CoverTy =`.
  **L635 CN**: 继续处理逻辑：`CoverTy =`。
- **L636 EN**: Declares function or method `vector`.
  **L636 CN**: 声明函数或方法 `vector`。
- **L637 EN**: Closes the current scope.
  **L637 CN**: 关闭当前作用域。
- **L638 EN**: Closes the current scope.
  **L638 CN**: 关闭当前作用域。
- **L639 EN**: Separates nearby statements for readability.
  **L639 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L640 EN**: Begins a conditional branch.
  **L640 CN**: 开始一个条件分支。

### Lines 641-660

````cpp
    assert(DstRegs.size() == 1);
    B.buildPadVectorWithUndefElements(DstRegs[0], SrcReg);
    return;
  }

  const unsigned DstSize = DstTy.getSizeInBits();
  const unsigned SrcSize = SrcTy.getSizeInBits();
  unsigned CoveringSize = CoverTy.getSizeInBits();

  Register UnmergeSrc = SrcReg;

  if (!CoverTy.isVector() && CoveringSize != SrcSize) {
    // For scalars, it's common to be able to use a simple extension.
    if (SrcTy.isScalar() && DstTy.isScalar()) {
      CoveringSize = alignTo(SrcSize, DstSize);
      LLT CoverTy = LLT::scalar(CoveringSize);
      UnmergeSrc = B.buildInstr(ExtendOp, {CoverTy}, {SrcReg}).getReg(0);
    } else {
      // Widen to the common type.
      // FIXME: This should respect the extend type
````
- **L641 EN**: Checks an invariant in debug builds.
  **L641 CN**: 在调试构建中检查一个不变量。
- **L642 EN**: Executes statement `B.buildPadVectorWithUndefElements(DstRegs[0], SrcReg);`.
  **L642 CN**: 执行语句 `B.buildPadVectorWithUndefElements(DstRegs[0], SrcReg);`。
- **L643 EN**: Returns control to the caller.
  **L643 CN**: 将控制流返回给调用者。
- **L644 EN**: Closes the current scope.
  **L644 CN**: 关闭当前作用域。
- **L645 EN**: Separates nearby statements for readability.
  **L645 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L646 EN**: Assigns or initializes `const unsigned DstSize`.
  **L646 CN**: 对 `const unsigned DstSize` 进行赋值或初始化。
- **L647 EN**: Assigns or initializes `const unsigned SrcSize`.
  **L647 CN**: 对 `const unsigned SrcSize` 进行赋值或初始化。
- **L648 EN**: Assigns or initializes `unsigned CoveringSize`.
  **L648 CN**: 对 `unsigned CoveringSize` 进行赋值或初始化。
- **L649 EN**: Separates nearby statements for readability.
  **L649 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L650 EN**: Assigns or initializes `Register UnmergeSrc`.
  **L650 CN**: 对 `Register UnmergeSrc` 进行赋值或初始化。
- **L651 EN**: Separates nearby statements for readability.
  **L651 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L652 EN**: Begins a conditional branch.
  **L652 CN**: 开始一个条件分支。
- **L653 EN**: Comment documents: `For scalars, it's common to be able to use a simple extension.`.
  **L653 CN**: 注释说明：`For scalars, it's common to be able to use a simple extension.`。
- **L654 EN**: Begins a conditional branch.
  **L654 CN**: 开始一个条件分支。
- **L655 EN**: Assigns or initializes `CoveringSize`.
  **L655 CN**: 对 `CoveringSize` 进行赋值或初始化。
- **L656 EN**: Declares function or method `scalar`.
  **L656 CN**: 声明函数或方法 `scalar`。
- **L657 EN**: Assigns or initializes `UnmergeSrc`.
  **L657 CN**: 对 `UnmergeSrc` 进行赋值或初始化。
- **L658 EN**: Starts block `} else`.
  **L658 CN**: 开始代码块 `} else`。
- **L659 EN**: Comment documents: `Widen to the common type.`.
  **L659 CN**: 注释说明：`Widen to the common type.`。
- **L660 EN**: Comment documents: `FIXME: This should respect the extend type`.
  **L660 CN**: 注释说明：`FIXME: This should respect the extend type`。

### Lines 661-680

````cpp
      Register Undef = B.buildUndef(SrcTy).getReg(0);
      SmallVector<Register, 8> MergeParts(1, SrcReg);
      for (unsigned Size = SrcSize; Size != CoveringSize; Size += SrcSize)
        MergeParts.push_back(Undef);
      UnmergeSrc = B.buildMergeLikeInstr(CoverTy, MergeParts).getReg(0);
    }
  }

  if (CoverTy.isVector() && CoveringSize != SrcSize)
    UnmergeSrc = B.buildPadVectorWithUndefElements(CoverTy, SrcReg).getReg(0);

  B.buildUnmerge(DstRegs, UnmergeSrc);
}

bool CallLowering::determineAndHandleAssignments(
    ValueHandler &Handler, ValueAssigner &Assigner,
    SmallVectorImpl<ArgInfo> &Args, MachineIRBuilder &MIRBuilder,
    CallingConv::ID CallConv, bool IsVarArg,
    ArrayRef<Register> ThisReturnRegs) const {
  MachineFunction &MF = MIRBuilder.getMF();
````
- **L661 EN**: Assigns or initializes `Register Undef`.
  **L661 CN**: 对 `Register Undef` 进行赋值或初始化。
- **L662 EN**: Declares function or method `MergeParts`.
  **L662 CN**: 声明函数或方法 `MergeParts`。
- **L663 EN**: Starts a loop over a sequence or range.
  **L663 CN**: 开始遍历序列或范围的循环。
- **L664 EN**: Executes statement `MergeParts.push_back(Undef);`.
  **L664 CN**: 执行语句 `MergeParts.push_back(Undef);`。
- **L665 EN**: Assigns or initializes `UnmergeSrc`.
  **L665 CN**: 对 `UnmergeSrc` 进行赋值或初始化。
- **L666 EN**: Closes the current scope.
  **L666 CN**: 关闭当前作用域。
- **L667 EN**: Closes the current scope.
  **L667 CN**: 关闭当前作用域。
- **L668 EN**: Separates nearby statements for readability.
  **L668 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L669 EN**: Begins a conditional branch.
  **L669 CN**: 开始一个条件分支。
- **L670 EN**: Assigns or initializes `UnmergeSrc`.
  **L670 CN**: 对 `UnmergeSrc` 进行赋值或初始化。
- **L671 EN**: Separates nearby statements for readability.
  **L671 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L672 EN**: Executes statement `B.buildUnmerge(DstRegs, UnmergeSrc);`.
  **L672 CN**: 执行语句 `B.buildUnmerge(DstRegs, UnmergeSrc);`。
- **L673 EN**: Closes the current scope.
  **L673 CN**: 关闭当前作用域。
- **L674 EN**: Separates nearby statements for readability.
  **L674 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L675 EN**: Provides part of the signature for `determineAndHandleAssignments`.
  **L675 CN**: 给出 `determineAndHandleAssignments` 的一部分签名。
- **L676 EN**: Continues logic with `ValueHandler &Handler, ValueAssigner &Assigner,`.
  **L676 CN**: 继续处理逻辑：`ValueHandler &Handler, ValueAssigner &Assigner,`。
- **L677 EN**: Continues logic with `SmallVectorImpl<ArgInfo> &Args, MachineIRBuilder &MIRBuilder,`.
  **L677 CN**: 继续处理逻辑：`SmallVectorImpl<ArgInfo> &Args, MachineIRBuilder &MIRBuilder,`。
- **L678 EN**: Continues logic with `CallingConv::ID CallConv, bool IsVarArg,`.
  **L678 CN**: 继续处理逻辑：`CallingConv::ID CallConv, bool IsVarArg,`。
- **L679 EN**: Starts block `ArrayRef<Register> ThisReturnRegs) const`.
  **L679 CN**: 开始代码块 `ArrayRef<Register> ThisReturnRegs) const`。
- **L680 EN**: Assigns or initializes `MachineFunction &MF`.
  **L680 CN**: 对 `MachineFunction &MF` 进行赋值或初始化。

### Lines 681-700

````cpp
  const Function &F = MF.getFunction();
  SmallVector<CCValAssign, 16> ArgLocs;

  CCState CCInfo(CallConv, IsVarArg, MF, ArgLocs, F.getContext());
  if (!determineAssignments(Assigner, Args, CCInfo))
    return false;

  return handleAssignments(Handler, Args, CCInfo, ArgLocs, MIRBuilder,
                           ThisReturnRegs);
}

static unsigned extendOpFromFlags(llvm::ISD::ArgFlagsTy Flags) {
  if (Flags.isSExt())
    return TargetOpcode::G_SEXT;
  if (Flags.isZExt())
    return TargetOpcode::G_ZEXT;
  return TargetOpcode::G_ANYEXT;
}

bool CallLowering::determineAssignments(ValueAssigner &Assigner,
````
- **L681 EN**: Assigns or initializes `const Function &F`.
  **L681 CN**: 对 `const Function &F` 进行赋值或初始化。
- **L682 EN**: Executes statement `SmallVector<CCValAssign, 16> ArgLocs;`.
  **L682 CN**: 执行语句 `SmallVector<CCValAssign, 16> ArgLocs;`。
- **L683 EN**: Separates nearby statements for readability.
  **L683 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L684 EN**: Declares function or method `CCInfo`.
  **L684 CN**: 声明函数或方法 `CCInfo`。
- **L685 EN**: Begins a conditional branch.
  **L685 CN**: 开始一个条件分支。
- **L686 EN**: Returns `false` to the caller.
  **L686 CN**: 向调用者返回 `false`。
- **L687 EN**: Separates nearby statements for readability.
  **L687 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L688 EN**: Returns `handleAssignments(Handler, Args, CCInfo, ArgLocs, MIRBuilder,` to the caller.
  **L688 CN**: 向调用者返回 `handleAssignments(Handler, Args, CCInfo, ArgLocs, MIRBuilder,`。
- **L689 EN**: Executes statement `ThisReturnRegs);`.
  **L689 CN**: 执行语句 `ThisReturnRegs);`。
- **L690 EN**: Closes the current scope.
  **L690 CN**: 关闭当前作用域。
- **L691 EN**: Separates nearby statements for readability.
  **L691 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L692 EN**: Begins the definition of `extendOpFromFlags`.
  **L692 CN**: 开始定义 `extendOpFromFlags`。
- **L693 EN**: Begins a conditional branch.
  **L693 CN**: 开始一个条件分支。
- **L694 EN**: Returns `TargetOpcode::G_SEXT` to the caller.
  **L694 CN**: 向调用者返回 `TargetOpcode::G_SEXT`。
- **L695 EN**: Begins a conditional branch.
  **L695 CN**: 开始一个条件分支。
- **L696 EN**: Returns `TargetOpcode::G_ZEXT` to the caller.
  **L696 CN**: 向调用者返回 `TargetOpcode::G_ZEXT`。
- **L697 EN**: Returns `TargetOpcode::G_ANYEXT` to the caller.
  **L697 CN**: 向调用者返回 `TargetOpcode::G_ANYEXT`。
- **L698 EN**: Closes the current scope.
  **L698 CN**: 关闭当前作用域。
- **L699 EN**: Separates nearby statements for readability.
  **L699 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L700 EN**: Provides part of the signature for `determineAssignments`.
  **L700 CN**: 给出 `determineAssignments` 的一部分签名。

### Lines 701-720

````cpp
                                        SmallVectorImpl<ArgInfo> &Args,
                                        CCState &CCInfo) const {
  LLVMContext &Ctx = CCInfo.getContext();
  const DataLayout &DL = CCInfo.getMachineFunction().getDataLayout();
  const CallingConv::ID CallConv = CCInfo.getCallingConv();

  unsigned NumArgs = Args.size();
  for (unsigned i = 0; i != NumArgs; ++i) {
    EVT CurVT = TLI->getValueType(DL, Args[i].Ty);

    MVT NewVT = TLI->getRegisterTypeForCallingConv(Ctx, CallConv, CurVT);

    // If we need to split the type over multiple regs, check it's a scenario
    // we currently support.
    unsigned NumParts =
        TLI->getNumRegistersForCallingConv(Ctx, CallConv, CurVT);

    if (NumParts == 1) {
      // Try to use the register type if we couldn't assign the VT.
      if (Assigner.assignArg(i, CurVT, NewVT, NewVT, CCValAssign::Full, Args[i],
````
- **L701 EN**: Continues logic with `SmallVectorImpl<ArgInfo> &Args,`.
  **L701 CN**: 继续处理逻辑：`SmallVectorImpl<ArgInfo> &Args,`。
- **L702 EN**: Starts block `CCState &CCInfo) const`.
  **L702 CN**: 开始代码块 `CCState &CCInfo) const`。
- **L703 EN**: Assigns or initializes `LLVMContext &Ctx`.
  **L703 CN**: 对 `LLVMContext &Ctx` 进行赋值或初始化。
- **L704 EN**: Assigns or initializes `const DataLayout &DL`.
  **L704 CN**: 对 `const DataLayout &DL` 进行赋值或初始化。
- **L705 EN**: Assigns or initializes `const CallingConv::ID CallConv`.
  **L705 CN**: 对 `const CallingConv::ID CallConv` 进行赋值或初始化。
- **L706 EN**: Separates nearby statements for readability.
  **L706 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L707 EN**: Assigns or initializes `unsigned NumArgs`.
  **L707 CN**: 对 `unsigned NumArgs` 进行赋值或初始化。
- **L708 EN**: Starts a loop over a sequence or range.
  **L708 CN**: 开始遍历序列或范围的循环。
- **L709 EN**: Assigns or initializes `EVT CurVT`.
  **L709 CN**: 对 `EVT CurVT` 进行赋值或初始化。
- **L710 EN**: Separates nearby statements for readability.
  **L710 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L711 EN**: Assigns or initializes `MVT NewVT`.
  **L711 CN**: 对 `MVT NewVT` 进行赋值或初始化。
- **L712 EN**: Separates nearby statements for readability.
  **L712 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L713 EN**: Comment documents: `If we need to split the type over multiple regs, check it's a scenario`.
  **L713 CN**: 注释说明：`If we need to split the type over multiple regs, check it's a scenario`。
- **L714 EN**: Comment documents: `we currently support.`.
  **L714 CN**: 注释说明：`we currently support.`。
- **L715 EN**: Continues logic with `unsigned NumParts =`.
  **L715 CN**: 继续处理逻辑：`unsigned NumParts =`。
- **L716 EN**: Executes statement `TLI->getNumRegistersForCallingConv(Ctx, CallConv, CurVT);`.
  **L716 CN**: 执行语句 `TLI->getNumRegistersForCallingConv(Ctx, CallConv, CurVT);`。
- **L717 EN**: Separates nearby statements for readability.
  **L717 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L718 EN**: Begins a conditional branch.
  **L718 CN**: 开始一个条件分支。
- **L719 EN**: Comment documents: `Try to use the register type if we couldn't assign the VT.`.
  **L719 CN**: 注释说明：`Try to use the register type if we couldn't assign the VT.`。
- **L720 EN**: Begins a conditional branch.
  **L720 CN**: 开始一个条件分支。

### Lines 721-740

````cpp
                             Args[i].Flags[0], CCInfo))
        return false;
      continue;
    }

    // For incoming arguments (physregs to vregs), we could have values in
    // physregs (or memlocs) which we want to extract and copy to vregs.
    // During this, we might have to deal with the LLT being split across
    // multiple regs, so we have to record this information for later.
    //
    // If we have outgoing args, then we have the opposite case. We have a
    // vreg with an LLT which we want to assign to a physical location, and
    // we might have to record that the value has to be split later.

    // We're handling an incoming arg which is split over multiple regs.
    // E.g. passing an s128 on AArch64.
    ISD::ArgFlagsTy OrigFlags = Args[i].Flags[0];
    Args[i].Flags.clear();

    for (unsigned Part = 0; Part < NumParts; ++Part) {
````
- **L721 EN**: Continues logic with `Args[i].Flags[0], CCInfo))`.
  **L721 CN**: 继续处理逻辑：`Args[i].Flags[0], CCInfo))`。
- **L722 EN**: Returns `false` to the caller.
  **L722 CN**: 向调用者返回 `false`。
- **L723 EN**: Skips to the next loop iteration.
  **L723 CN**: 跳到下一次循环迭代。
- **L724 EN**: Closes the current scope.
  **L724 CN**: 关闭当前作用域。
- **L725 EN**: Separates nearby statements for readability.
  **L725 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L726 EN**: Comment documents: `For incoming arguments (physregs to vregs), we could have values in`.
  **L726 CN**: 注释说明：`For incoming arguments (physregs to vregs), we could have values in`。
- **L727 EN**: Comment documents: `physregs (or memlocs) which we want to extract and copy to vregs.`.
  **L727 CN**: 注释说明：`physregs (or memlocs) which we want to extract and copy to vregs.`。
- **L728 EN**: Comment documents: `During this, we might have to deal with the LLT being split across`.
  **L728 CN**: 注释说明：`During this, we might have to deal with the LLT being split across`。
- **L729 EN**: Comment documents: `multiple regs, so we have to record this information for later.`.
  **L729 CN**: 注释说明：`multiple regs, so we have to record this information for later.`。
- **L730 EN**: Continues the surrounding comment block.
  **L730 CN**: 延续周围的注释块。
- **L731 EN**: Comment documents: `If we have outgoing args, then we have the opposite case. We have a`.
  **L731 CN**: 注释说明：`If we have outgoing args, then we have the opposite case. We have a`。
- **L732 EN**: Comment documents: `vreg with an LLT which we want to assign to a physical location, and`.
  **L732 CN**: 注释说明：`vreg with an LLT which we want to assign to a physical location, and`。
- **L733 EN**: Comment documents: `we might have to record that the value has to be split later.`.
  **L733 CN**: 注释说明：`we might have to record that the value has to be split later.`。
- **L734 EN**: Separates nearby statements for readability.
  **L734 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L735 EN**: Comment documents: `We're handling an incoming arg which is split over multiple regs.`.
  **L735 CN**: 注释说明：`We're handling an incoming arg which is split over multiple regs.`。
- **L736 EN**: Comment documents: `E.g. passing an s128 on AArch64.`.
  **L736 CN**: 注释说明：`E.g. passing an s128 on AArch64.`。
- **L737 EN**: Assigns or initializes `ISD::ArgFlagsTy OrigFlags`.
  **L737 CN**: 对 `ISD::ArgFlagsTy OrigFlags` 进行赋值或初始化。
- **L738 EN**: Executes statement `Args[i].Flags.clear();`.
  **L738 CN**: 执行语句 `Args[i].Flags.clear();`。
- **L739 EN**: Separates nearby statements for readability.
  **L739 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L740 EN**: Starts a loop over a sequence or range.
  **L740 CN**: 开始遍历序列或范围的循环。

### Lines 741-760

````cpp
      ISD::ArgFlagsTy Flags = OrigFlags;
      if (Part == 0) {
        Flags.setSplit();
      } else {
        Flags.setOrigAlign(Align(1));
        if (Part == NumParts - 1)
          Flags.setSplitEnd();
      }

      Args[i].Flags.push_back(Flags);
      if (Assigner.assignArg(i, CurVT, NewVT, NewVT, CCValAssign::Full, Args[i],
                             Args[i].Flags[Part], CCInfo)) {
        // Still couldn't assign this smaller part type for some reason.
        return false;
      }
    }
  }

  return true;
}
````
- **L741 EN**: Assigns or initializes `ISD::ArgFlagsTy Flags`.
  **L741 CN**: 对 `ISD::ArgFlagsTy Flags` 进行赋值或初始化。
- **L742 EN**: Begins a conditional branch.
  **L742 CN**: 开始一个条件分支。
- **L743 EN**: Executes statement `Flags.setSplit();`.
  **L743 CN**: 执行语句 `Flags.setSplit();`。
- **L744 EN**: Starts block `} else`.
  **L744 CN**: 开始代码块 `} else`。
- **L745 EN**: Executes statement `Flags.setOrigAlign(Align(1));`.
  **L745 CN**: 执行语句 `Flags.setOrigAlign(Align(1));`。
- **L746 EN**: Begins a conditional branch.
  **L746 CN**: 开始一个条件分支。
- **L747 EN**: Executes statement `Flags.setSplitEnd();`.
  **L747 CN**: 执行语句 `Flags.setSplitEnd();`。
- **L748 EN**: Closes the current scope.
  **L748 CN**: 关闭当前作用域。
- **L749 EN**: Separates nearby statements for readability.
  **L749 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L750 EN**: Executes statement `Args[i].Flags.push_back(Flags);`.
  **L750 CN**: 执行语句 `Args[i].Flags.push_back(Flags);`。
- **L751 EN**: Begins a conditional branch.
  **L751 CN**: 开始一个条件分支。
- **L752 EN**: Starts block `Args[i].Flags[Part], CCInfo))`.
  **L752 CN**: 开始代码块 `Args[i].Flags[Part], CCInfo))`。
- **L753 EN**: Comment documents: `Still couldn't assign this smaller part type for some reason.`.
  **L753 CN**: 注释说明：`Still couldn't assign this smaller part type for some reason.`。
- **L754 EN**: Returns `false` to the caller.
  **L754 CN**: 向调用者返回 `false`。
- **L755 EN**: Closes the current scope.
  **L755 CN**: 关闭当前作用域。
- **L756 EN**: Closes the current scope.
  **L756 CN**: 关闭当前作用域。
- **L757 EN**: Closes the current scope.
  **L757 CN**: 关闭当前作用域。
- **L758 EN**: Separates nearby statements for readability.
  **L758 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L759 EN**: Returns `true` to the caller.
  **L759 CN**: 向调用者返回 `true`。
- **L760 EN**: Closes the current scope.
  **L760 CN**: 关闭当前作用域。

### Lines 761-780

````cpp

bool CallLowering::handleAssignments(ValueHandler &Handler,
                                     SmallVectorImpl<ArgInfo> &Args,
                                     CCState &CCInfo,
                                     SmallVectorImpl<CCValAssign> &ArgLocs,
                                     MachineIRBuilder &MIRBuilder,
                                     ArrayRef<Register> ThisReturnRegs) const {
  MachineFunction &MF = MIRBuilder.getMF();
  MachineRegisterInfo &MRI = MF.getRegInfo();
  const Function &F = MF.getFunction();
  const DataLayout &DL = F.getDataLayout();

  const unsigned NumArgs = Args.size();

  // Stores thunks for outgoing register assignments. This is used so we delay
  // generating register copies until mem loc assignments are done. We do this
  // so that if the target is using the delayed stack protector feature, we can
  // find the split point of the block accurately. E.g. if we have:
  // G_STORE %val, %memloc
  // $x0 = COPY %foo
````
- **L761 EN**: Separates nearby statements for readability.
  **L761 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L762 EN**: Provides part of the signature for `handleAssignments`.
  **L762 CN**: 给出 `handleAssignments` 的一部分签名。
- **L763 EN**: Continues logic with `SmallVectorImpl<ArgInfo> &Args,`.
  **L763 CN**: 继续处理逻辑：`SmallVectorImpl<ArgInfo> &Args,`。
- **L764 EN**: Continues logic with `CCState &CCInfo,`.
  **L764 CN**: 继续处理逻辑：`CCState &CCInfo,`。
- **L765 EN**: Continues logic with `SmallVectorImpl<CCValAssign> &ArgLocs,`.
  **L765 CN**: 继续处理逻辑：`SmallVectorImpl<CCValAssign> &ArgLocs,`。
- **L766 EN**: Continues logic with `MachineIRBuilder &MIRBuilder,`.
  **L766 CN**: 继续处理逻辑：`MachineIRBuilder &MIRBuilder,`。
- **L767 EN**: Starts block `ArrayRef<Register> ThisReturnRegs) const`.
  **L767 CN**: 开始代码块 `ArrayRef<Register> ThisReturnRegs) const`。
- **L768 EN**: Assigns or initializes `MachineFunction &MF`.
  **L768 CN**: 对 `MachineFunction &MF` 进行赋值或初始化。
- **L769 EN**: Assigns or initializes `MachineRegisterInfo &MRI`.
  **L769 CN**: 对 `MachineRegisterInfo &MRI` 进行赋值或初始化。
- **L770 EN**: Assigns or initializes `const Function &F`.
  **L770 CN**: 对 `const Function &F` 进行赋值或初始化。
- **L771 EN**: Assigns or initializes `const DataLayout &DL`.
  **L771 CN**: 对 `const DataLayout &DL` 进行赋值或初始化。
- **L772 EN**: Separates nearby statements for readability.
  **L772 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L773 EN**: Assigns or initializes `const unsigned NumArgs`.
  **L773 CN**: 对 `const unsigned NumArgs` 进行赋值或初始化。
- **L774 EN**: Separates nearby statements for readability.
  **L774 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L775 EN**: Comment documents: `Stores thunks for outgoing register assignments. This is used so we dela…`.
  **L775 CN**: 注释说明：`Stores thunks for outgoing register assignments. This is used so we dela…`。
- **L776 EN**: Comment documents: `generating register copies until mem loc assignments are done. We do thi…`.
  **L776 CN**: 注释说明：`generating register copies until mem loc assignments are done. We do thi…`。
- **L777 EN**: Comment documents: `so that if the target is using the delayed stack protector feature, we c…`.
  **L777 CN**: 注释说明：`so that if the target is using the delayed stack protector feature, we c…`。
- **L778 EN**: Comment documents: `find the split point of the block accurately. E.g. if we have:`.
  **L778 CN**: 注释说明：`find the split point of the block accurately. E.g. if we have:`。
- **L779 EN**: Comment documents: `G_STORE %val, %memloc`.
  **L779 CN**: 注释说明：`G_STORE %val, %memloc`。
- **L780 EN**: Comment documents: `$x0 = COPY %foo`.
  **L780 CN**: 注释说明：`$x0 = COPY %foo`。

### Lines 781-800

````cpp
  // $x1 = COPY %bar
  // CALL func
  // ... then the split point for the block will correctly be at, and including,
  // the copy to $x0. If instead the G_STORE instruction immediately precedes
  // the CALL, then we'd prematurely choose the CALL as the split point, thus
  // generating a split block with a CALL that uses undefined physregs.
  SmallVector<std::function<void()>> DelayedOutgoingRegAssignments;

  for (unsigned i = 0, j = 0; i != NumArgs; ++i, ++j) {
    assert(j < ArgLocs.size() && "Skipped too many arg locs");
    CCValAssign &VA = ArgLocs[j];
    assert(VA.getValNo() == i && "Location doesn't correspond to current arg");

    if (VA.needsCustom()) {
      std::function<void()> Thunk;
      unsigned NumArgRegs = Handler.assignCustomValue(
          Args[i], ArrayRef(ArgLocs).slice(j), &Thunk);
      if (Thunk)
        DelayedOutgoingRegAssignments.emplace_back(Thunk);
      if (!NumArgRegs)
````
- **L781 EN**: Comment documents: `$x1 = COPY %bar`.
  **L781 CN**: 注释说明：`$x1 = COPY %bar`。
- **L782 EN**: Comment documents: `CALL func`.
  **L782 CN**: 注释说明：`CALL func`。
- **L783 EN**: Comment documents: `... then the split point for the block will correctly be at, and includi…`.
  **L783 CN**: 注释说明：`... then the split point for the block will correctly be at, and includi…`。
- **L784 EN**: Comment documents: `the copy to $x0. If instead the G_STORE instruction immediately precedes`.
  **L784 CN**: 注释说明：`the copy to $x0. If instead the G_STORE instruction immediately precedes`。
- **L785 EN**: Comment documents: `the CALL, then we'd prematurely choose the CALL as the split point, thus`.
  **L785 CN**: 注释说明：`the CALL, then we'd prematurely choose the CALL as the split point, thus`。
- **L786 EN**: Comment documents: `generating a split block with a CALL that uses undefined physregs.`.
  **L786 CN**: 注释说明：`generating a split block with a CALL that uses undefined physregs.`。
- **L787 EN**: Declares function or method `void`.
  **L787 CN**: 声明函数或方法 `void`。
- **L788 EN**: Separates nearby statements for readability.
  **L788 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L789 EN**: Starts a loop over a sequence or range.
  **L789 CN**: 开始遍历序列或范围的循环。
- **L790 EN**: Checks an invariant in debug builds.
  **L790 CN**: 在调试构建中检查一个不变量。
- **L791 EN**: Assigns or initializes `CCValAssign &VA`.
  **L791 CN**: 对 `CCValAssign &VA` 进行赋值或初始化。
- **L792 EN**: Checks an invariant in debug builds.
  **L792 CN**: 在调试构建中检查一个不变量。
- **L793 EN**: Separates nearby statements for readability.
  **L793 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L794 EN**: Begins a conditional branch.
  **L794 CN**: 开始一个条件分支。
- **L795 EN**: Declares function or method `void`.
  **L795 CN**: 声明函数或方法 `void`。
- **L796 EN**: Continues logic with `unsigned NumArgRegs = Handler.assignCustomValue(`.
  **L796 CN**: 继续处理逻辑：`unsigned NumArgRegs = Handler.assignCustomValue(`。
- **L797 EN**: Executes statement `Args[i], ArrayRef(ArgLocs).slice(j), &Thunk);`.
  **L797 CN**: 执行语句 `Args[i], ArrayRef(ArgLocs).slice(j), &Thunk);`。
- **L798 EN**: Begins a conditional branch.
  **L798 CN**: 开始一个条件分支。
- **L799 EN**: Executes statement `DelayedOutgoingRegAssignments.emplace_back(Thunk);`.
  **L799 CN**: 执行语句 `DelayedOutgoingRegAssignments.emplace_back(Thunk);`。
- **L800 EN**: Begins a conditional branch.
  **L800 CN**: 开始一个条件分支。

### Lines 801-820

````cpp
        return false;
      j += (NumArgRegs - 1);
      continue;
    }

    auto AllocaAddressSpace = MF.getDataLayout().getAllocaAddrSpace();

    const MVT ValVT = VA.getValVT();
    const MVT LocVT = VA.getLocVT();

    const LLT LocTy = getLLTForMVT(LocVT);
    const LLT ValTy = getLLTForMVT(ValVT);
    const LLT NewLLT = Handler.isIncomingArgumentHandler() ? LocTy : ValTy;
    const EVT OrigVT = TLI->getValueType(DL, Args[i].Ty);
    // Use the EVT here to strip pointerness.
    const LLT OrigTy = getLLTForType(*OrigVT.getTypeForEVT(F.getContext()), DL);
    const LLT PointerTy = LLT::pointer(
        AllocaAddressSpace, DL.getPointerSizeInBits(AllocaAddressSpace));

    // Expected to be multiple regs for a single incoming arg.
````
- **L801 EN**: Returns `false` to the caller.
  **L801 CN**: 向调用者返回 `false`。
- **L802 EN**: Assigns or initializes `j +`.
  **L802 CN**: 对 `j +` 进行赋值或初始化。
- **L803 EN**: Skips to the next loop iteration.
  **L803 CN**: 跳到下一次循环迭代。
- **L804 EN**: Closes the current scope.
  **L804 CN**: 关闭当前作用域。
- **L805 EN**: Separates nearby statements for readability.
  **L805 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L806 EN**: Assigns or initializes `auto AllocaAddressSpace`.
  **L806 CN**: 对 `auto AllocaAddressSpace` 进行赋值或初始化。
- **L807 EN**: Separates nearby statements for readability.
  **L807 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L808 EN**: Assigns or initializes `const MVT ValVT`.
  **L808 CN**: 对 `const MVT ValVT` 进行赋值或初始化。
- **L809 EN**: Assigns or initializes `const MVT LocVT`.
  **L809 CN**: 对 `const MVT LocVT` 进行赋值或初始化。
- **L810 EN**: Separates nearby statements for readability.
  **L810 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L811 EN**: Assigns or initializes `const LLT LocTy`.
  **L811 CN**: 对 `const LLT LocTy` 进行赋值或初始化。
- **L812 EN**: Assigns or initializes `const LLT ValTy`.
  **L812 CN**: 对 `const LLT ValTy` 进行赋值或初始化。
- **L813 EN**: Assigns or initializes `const LLT NewLLT`.
  **L813 CN**: 对 `const LLT NewLLT` 进行赋值或初始化。
- **L814 EN**: Assigns or initializes `const EVT OrigVT`.
  **L814 CN**: 对 `const EVT OrigVT` 进行赋值或初始化。
- **L815 EN**: Comment documents: `Use the EVT here to strip pointerness.`.
  **L815 CN**: 注释说明：`Use the EVT here to strip pointerness.`。
- **L816 EN**: Assigns or initializes `const LLT OrigTy`.
  **L816 CN**: 对 `const LLT OrigTy` 进行赋值或初始化。
- **L817 EN**: Provides part of the signature for `pointer`.
  **L817 CN**: 给出 `pointer` 的一部分签名。
- **L818 EN**: Executes statement `AllocaAddressSpace, DL.getPointerSizeInBits(AllocaAddressSpace));`.
  **L818 CN**: 执行语句 `AllocaAddressSpace, DL.getPointerSizeInBits(AllocaAddressSpace));`。
- **L819 EN**: Separates nearby statements for readability.
  **L819 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L820 EN**: Comment documents: `Expected to be multiple regs for a single incoming arg.`.
  **L820 CN**: 注释说明：`Expected to be multiple regs for a single incoming arg.`。

### Lines 821-840

````cpp
    // There should be Regs.size() ArgLocs per argument.
    // This should be the same as getNumRegistersForCallingConv
    const unsigned NumParts = Args[i].Flags.size();

    // Now split the registers into the assigned types.
    Args[i].OrigRegs.assign(Args[i].Regs.begin(), Args[i].Regs.end());

    if (NumParts != 1 || NewLLT != OrigTy) {
      // If we can't directly assign the register, we need one or more
      // intermediate values.
      Args[i].Regs.resize(NumParts);

      // When we have indirect parameter passing we are receiving a pointer,
      // that points to the actual value, so we need one "temporary" pointer.
      if (VA.getLocInfo() == CCValAssign::Indirect) {
        if (Handler.isIncomingArgumentHandler())
          Args[i].Regs[0] = MRI.createGenericVirtualRegister(PointerTy);
      } else {
        // For each split register, create and assign a vreg that will store
        // the incoming component of the larger value. These will later be
````
- **L821 EN**: Comment documents: `There should be Regs.size() ArgLocs per argument.`.
  **L821 CN**: 注释说明：`There should be Regs.size() ArgLocs per argument.`。
- **L822 EN**: Comment documents: `This should be the same as getNumRegistersForCallingConv`.
  **L822 CN**: 注释说明：`This should be the same as getNumRegistersForCallingConv`。
- **L823 EN**: Assigns or initializes `const unsigned NumParts`.
  **L823 CN**: 对 `const unsigned NumParts` 进行赋值或初始化。
- **L824 EN**: Separates nearby statements for readability.
  **L824 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L825 EN**: Comment documents: `Now split the registers into the assigned types.`.
  **L825 CN**: 注释说明：`Now split the registers into the assigned types.`。
- **L826 EN**: Executes statement `Args[i].OrigRegs.assign(Args[i].Regs.begin(), Args[i].Regs.end());`.
  **L826 CN**: 执行语句 `Args[i].OrigRegs.assign(Args[i].Regs.begin(), Args[i].Regs.end());`。
- **L827 EN**: Separates nearby statements for readability.
  **L827 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L828 EN**: Begins a conditional branch.
  **L828 CN**: 开始一个条件分支。
- **L829 EN**: Comment documents: `If we can't directly assign the register, we need one or more`.
  **L829 CN**: 注释说明：`If we can't directly assign the register, we need one or more`。
- **L830 EN**: Comment documents: `intermediate values.`.
  **L830 CN**: 注释说明：`intermediate values.`。
- **L831 EN**: Executes statement `Args[i].Regs.resize(NumParts);`.
  **L831 CN**: 执行语句 `Args[i].Regs.resize(NumParts);`。
- **L832 EN**: Separates nearby statements for readability.
  **L832 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L833 EN**: Comment documents: `When we have indirect parameter passing we are receiving a pointer,`.
  **L833 CN**: 注释说明：`When we have indirect parameter passing we are receiving a pointer,`。
- **L834 EN**: Comment documents: `that points to the actual value, so we need one "temporary" pointer.`.
  **L834 CN**: 注释说明：`that points to the actual value, so we need one "temporary" pointer.`。
- **L835 EN**: Begins a conditional branch.
  **L835 CN**: 开始一个条件分支。
- **L836 EN**: Begins a conditional branch.
  **L836 CN**: 开始一个条件分支。
- **L837 EN**: Assigns or initializes `Args[i].Regs[0]`.
  **L837 CN**: 对 `Args[i].Regs[0]` 进行赋值或初始化。
- **L838 EN**: Starts block `} else`.
  **L838 CN**: 开始代码块 `} else`。
- **L839 EN**: Comment documents: `For each split register, create and assign a vreg that will store`.
  **L839 CN**: 注释说明：`For each split register, create and assign a vreg that will store`。
- **L840 EN**: Comment documents: `the incoming component of the larger value. These will later be`.
  **L840 CN**: 注释说明：`the incoming component of the larger value. These will later be`。

### Lines 841-860

````cpp
        // merged to form the final vreg.
        for (unsigned Part = 0; Part < NumParts; ++Part)
          Args[i].Regs[Part] = MRI.createGenericVirtualRegister(NewLLT);
      }
    }

    assert((j + (NumParts - 1)) < ArgLocs.size() &&
           "Too many regs for number of args");

    // Coerce into outgoing value types before register assignment.
    if (!Handler.isIncomingArgumentHandler() && OrigTy != ValTy &&
        VA.getLocInfo() != CCValAssign::Indirect) {
      assert(Args[i].OrigRegs.size() == 1);
      buildCopyToRegs(MIRBuilder, Args[i].Regs, Args[i].OrigRegs[0], OrigTy,
                      ValTy, extendOpFromFlags(Args[i].Flags[0]));
    }

    bool IndirectParameterPassingHandled = false;
    bool BigEndianPartOrdering = TLI->hasBigEndianPartOrdering(OrigVT, DL);
    for (unsigned Part = 0; Part < NumParts; ++Part) {
````
- **L841 EN**: Comment documents: `merged to form the final vreg.`.
  **L841 CN**: 注释说明：`merged to form the final vreg.`。
- **L842 EN**: Starts a loop over a sequence or range.
  **L842 CN**: 开始遍历序列或范围的循环。
- **L843 EN**: Assigns or initializes `Args[i].Regs[Part]`.
  **L843 CN**: 对 `Args[i].Regs[Part]` 进行赋值或初始化。
- **L844 EN**: Closes the current scope.
  **L844 CN**: 关闭当前作用域。
- **L845 EN**: Closes the current scope.
  **L845 CN**: 关闭当前作用域。
- **L846 EN**: Separates nearby statements for readability.
  **L846 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L847 EN**: Checks an invariant in debug builds.
  **L847 CN**: 在调试构建中检查一个不变量。
- **L848 EN**: Executes statement `"Too many regs for number of args");`.
  **L848 CN**: 执行语句 `"Too many regs for number of args");`。
- **L849 EN**: Separates nearby statements for readability.
  **L849 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L850 EN**: Comment documents: `Coerce into outgoing value types before register assignment.`.
  **L850 CN**: 注释说明：`Coerce into outgoing value types before register assignment.`。
- **L851 EN**: Begins a conditional branch.
  **L851 CN**: 开始一个条件分支。
- **L852 EN**: Starts block `VA.getLocInfo() != CCValAssign::Indirect)`.
  **L852 CN**: 开始代码块 `VA.getLocInfo() != CCValAssign::Indirect)`。
- **L853 EN**: Checks an invariant in debug builds.
  **L853 CN**: 在调试构建中检查一个不变量。
- **L854 EN**: Continues logic with `buildCopyToRegs(MIRBuilder, Args[i].Regs, Args[i].OrigRegs[0], OrigTy,`.
  **L854 CN**: 继续处理逻辑：`buildCopyToRegs(MIRBuilder, Args[i].Regs, Args[i].OrigRegs[0], OrigTy,`。
- **L855 EN**: Declares function or method `extendOpFromFlags`.
  **L855 CN**: 声明函数或方法 `extendOpFromFlags`。
- **L856 EN**: Closes the current scope.
  **L856 CN**: 关闭当前作用域。
- **L857 EN**: Separates nearby statements for readability.
  **L857 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L858 EN**: Assigns or initializes `bool IndirectParameterPassingHandled`.
  **L858 CN**: 对 `bool IndirectParameterPassingHandled` 进行赋值或初始化。
- **L859 EN**: Assigns or initializes `bool BigEndianPartOrdering`.
  **L859 CN**: 对 `bool BigEndianPartOrdering` 进行赋值或初始化。
- **L860 EN**: Starts a loop over a sequence or range.
  **L860 CN**: 开始遍历序列或范围的循环。

### Lines 861-880

````cpp
      assert((VA.getLocInfo() != CCValAssign::Indirect || Part == 0) &&
             "Only the first parameter should be processed when "
             "handling indirect passing!");
      Register ArgReg = Args[i].Regs[Part];
      // There should be Regs.size() ArgLocs per argument.
      unsigned Idx = BigEndianPartOrdering ? NumParts - 1 - Part : Part;
      CCValAssign &VA = ArgLocs[j + Idx];
      const ISD::ArgFlagsTy Flags = Args[i].Flags[Part];

      // We found an indirect parameter passing, and we have an
      // OutgoingValueHandler as our handler (so we are at the call site or the
      // return value). In this case, start the construction of the following
      // GMIR, that is responsible for the preparation of indirect parameter
      // passing:
      //
      // %1(indirectly passed type) = The value to pass
      // %3(pointer) = G_FRAME_INDEX %stack.0
      // G_STORE %1, %3 :: (store (s128), align 8)
      //
      // After this GMIR, the remaining part of the loop body will decide how
````
- **L861 EN**: Checks an invariant in debug builds.
  **L861 CN**: 在调试构建中检查一个不变量。
- **L862 EN**: Continues logic with `"Only the first parameter should be processed when "`.
  **L862 CN**: 继续处理逻辑：`"Only the first parameter should be processed when "`。
- **L863 EN**: Executes statement `"handling indirect passing!");`.
  **L863 CN**: 执行语句 `"handling indirect passing!");`。
- **L864 EN**: Assigns or initializes `Register ArgReg`.
  **L864 CN**: 对 `Register ArgReg` 进行赋值或初始化。
- **L865 EN**: Comment documents: `There should be Regs.size() ArgLocs per argument.`.
  **L865 CN**: 注释说明：`There should be Regs.size() ArgLocs per argument.`。
- **L866 EN**: Assigns or initializes `unsigned Idx`.
  **L866 CN**: 对 `unsigned Idx` 进行赋值或初始化。
- **L867 EN**: Assigns or initializes `CCValAssign &VA`.
  **L867 CN**: 对 `CCValAssign &VA` 进行赋值或初始化。
- **L868 EN**: Assigns or initializes `const ISD::ArgFlagsTy Flags`.
  **L868 CN**: 对 `const ISD::ArgFlagsTy Flags` 进行赋值或初始化。
- **L869 EN**: Separates nearby statements for readability.
  **L869 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L870 EN**: Comment documents: `We found an indirect parameter passing, and we have an`.
  **L870 CN**: 注释说明：`We found an indirect parameter passing, and we have an`。
- **L871 EN**: Comment documents: `OutgoingValueHandler as our handler (so we are at the call site or the`.
  **L871 CN**: 注释说明：`OutgoingValueHandler as our handler (so we are at the call site or the`。
- **L872 EN**: Comment documents: `return value). In this case, start the construction of the following`.
  **L872 CN**: 注释说明：`return value). In this case, start the construction of the following`。
- **L873 EN**: Comment documents: `GMIR, that is responsible for the preparation of indirect parameter`.
  **L873 CN**: 注释说明：`GMIR, that is responsible for the preparation of indirect parameter`。
- **L874 EN**: Comment documents: `passing:`.
  **L874 CN**: 注释说明：`passing:`。
- **L875 EN**: Continues the surrounding comment block.
  **L875 CN**: 延续周围的注释块。
- **L876 EN**: Comment documents: `%1(indirectly passed type) = The value to pass`.
  **L876 CN**: 注释说明：`%1(indirectly passed type) = The value to pass`。
- **L877 EN**: Comment documents: `%3(pointer) = G_FRAME_INDEX %stack.0`.
  **L877 CN**: 注释说明：`%3(pointer) = G_FRAME_INDEX %stack.0`。
- **L878 EN**: Comment documents: `G_STORE %1, %3 :: (store (s128), align 8)`.
  **L878 CN**: 注释说明：`G_STORE %1, %3 :: (store (s128), align 8)`。
- **L879 EN**: Continues the surrounding comment block.
  **L879 CN**: 延续周围的注释块。
- **L880 EN**: Comment documents: `After this GMIR, the remaining part of the loop body will decide how`.
  **L880 CN**: 注释说明：`After this GMIR, the remaining part of the loop body will decide how`。

### Lines 881-900

````cpp
      // to get the value to the caller and we break out of the loop.
      if (VA.getLocInfo() == CCValAssign::Indirect &&
          !Handler.isIncomingArgumentHandler()) {
        Align AlignmentForStored = DL.getPrefTypeAlign(Args[i].Ty);
        MachineFrameInfo &MFI = MF.getFrameInfo();
        // Get some space on the stack for the value, so later we can pass it
        // as a reference.
        int FrameIdx = MFI.CreateStackObject(OrigTy.getScalarSizeInBits(),
                                             AlignmentForStored, false);
        Register PointerToStackReg =
            MIRBuilder.buildFrameIndex(PointerTy, FrameIdx).getReg(0);
        MachinePointerInfo StackPointerMPO =
            MachinePointerInfo::getFixedStack(MF, FrameIdx);
        // Store the value in the previously created stack space.
        MIRBuilder.buildStore(Args[i].OrigRegs[Part], PointerToStackReg,
                              StackPointerMPO,
                              inferAlignFromPtrInfo(MF, StackPointerMPO));

        ArgReg = PointerToStackReg;
        IndirectParameterPassingHandled = true;
````
- **L881 EN**: Comment documents: `to get the value to the caller and we break out of the loop.`.
  **L881 CN**: 注释说明：`to get the value to the caller and we break out of the loop.`。
- **L882 EN**: Begins a conditional branch.
  **L882 CN**: 开始一个条件分支。
- **L883 EN**: Starts block `!Handler.isIncomingArgumentHandler())`.
  **L883 CN**: 开始代码块 `!Handler.isIncomingArgumentHandler())`。
- **L884 EN**: Assigns or initializes `Align AlignmentForStored`.
  **L884 CN**: 对 `Align AlignmentForStored` 进行赋值或初始化。
- **L885 EN**: Assigns or initializes `MachineFrameInfo &MFI`.
  **L885 CN**: 对 `MachineFrameInfo &MFI` 进行赋值或初始化。
- **L886 EN**: Comment documents: `Get some space on the stack for the value, so later we can pass it`.
  **L886 CN**: 注释说明：`Get some space on the stack for the value, so later we can pass it`。
- **L887 EN**: Comment documents: `as a reference.`.
  **L887 CN**: 注释说明：`as a reference.`。
- **L888 EN**: Continues logic with `int FrameIdx = MFI.CreateStackObject(OrigTy.getScalarSizeInBits(),`.
  **L888 CN**: 继续处理逻辑：`int FrameIdx = MFI.CreateStackObject(OrigTy.getScalarSizeInBits(),`。
- **L889 EN**: Executes statement `AlignmentForStored, false);`.
  **L889 CN**: 执行语句 `AlignmentForStored, false);`。
- **L890 EN**: Continues logic with `Register PointerToStackReg =`.
  **L890 CN**: 继续处理逻辑：`Register PointerToStackReg =`。
- **L891 EN**: Executes statement `MIRBuilder.buildFrameIndex(PointerTy, FrameIdx).getReg(0);`.
  **L891 CN**: 执行语句 `MIRBuilder.buildFrameIndex(PointerTy, FrameIdx).getReg(0);`。
- **L892 EN**: Continues logic with `MachinePointerInfo StackPointerMPO =`.
  **L892 CN**: 继续处理逻辑：`MachinePointerInfo StackPointerMPO =`。
- **L893 EN**: Declares function or method `getFixedStack`.
  **L893 CN**: 声明函数或方法 `getFixedStack`。
- **L894 EN**: Comment documents: `Store the value in the previously created stack space.`.
  **L894 CN**: 注释说明：`Store the value in the previously created stack space.`。
- **L895 EN**: Continues logic with `MIRBuilder.buildStore(Args[i].OrigRegs[Part], PointerToStackReg,`.
  **L895 CN**: 继续处理逻辑：`MIRBuilder.buildStore(Args[i].OrigRegs[Part], PointerToStackReg,`。
- **L896 EN**: Continues logic with `StackPointerMPO,`.
  **L896 CN**: 继续处理逻辑：`StackPointerMPO,`。
- **L897 EN**: Executes statement `inferAlignFromPtrInfo(MF, StackPointerMPO));`.
  **L897 CN**: 执行语句 `inferAlignFromPtrInfo(MF, StackPointerMPO));`。
- **L898 EN**: Separates nearby statements for readability.
  **L898 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L899 EN**: Assigns or initializes `ArgReg`.
  **L899 CN**: 对 `ArgReg` 进行赋值或初始化。
- **L900 EN**: Assigns or initializes `IndirectParameterPassingHandled`.
  **L900 CN**: 对 `IndirectParameterPassingHandled` 进行赋值或初始化。

### Lines 901-920

````cpp
      }

      if (VA.isMemLoc() && !Flags.isByVal()) {
        // Individual pieces may have been spilled to the stack and others
        // passed in registers.

        // TODO: The memory size may be larger than the value we need to
        // store. We may need to adjust the offset for big endian targets.
        LLT MemTy = Handler.getStackValueStoreType(DL, VA, Flags);

        MachinePointerInfo MPO;
        Register StackAddr =
            Handler.getStackAddress(VA.getLocInfo() == CCValAssign::Indirect
                                        ? PointerTy.getSizeInBytes()
                                        : MemTy.getSizeInBytes(),
                                    VA.getLocMemOffset(), MPO, Flags);

        // Finish the handling of indirect passing from the passers
        // (OutgoingParameterHandler) side.
        // This branch is needed, so the pointer to the value is loaded onto the
````
- **L901 EN**: Closes the current scope.
  **L901 CN**: 关闭当前作用域。
- **L902 EN**: Separates nearby statements for readability.
  **L902 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L903 EN**: Begins a conditional branch.
  **L903 CN**: 开始一个条件分支。
- **L904 EN**: Comment documents: `Individual pieces may have been spilled to the stack and others`.
  **L904 CN**: 注释说明：`Individual pieces may have been spilled to the stack and others`。
- **L905 EN**: Comment documents: `passed in registers.`.
  **L905 CN**: 注释说明：`passed in registers.`。
- **L906 EN**: Separates nearby statements for readability.
  **L906 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L907 EN**: Comment documents: `TODO: The memory size may be larger than the value we need to`.
  **L907 CN**: 注释说明：`TODO: The memory size may be larger than the value we need to`。
- **L908 EN**: Comment documents: `store. We may need to adjust the offset for big endian targets.`.
  **L908 CN**: 注释说明：`store. We may need to adjust the offset for big endian targets.`。
- **L909 EN**: Assigns or initializes `LLT MemTy`.
  **L909 CN**: 对 `LLT MemTy` 进行赋值或初始化。
- **L910 EN**: Separates nearby statements for readability.
  **L910 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L911 EN**: Executes statement `MachinePointerInfo MPO;`.
  **L911 CN**: 执行语句 `MachinePointerInfo MPO;`。
- **L912 EN**: Continues logic with `Register StackAddr =`.
  **L912 CN**: 继续处理逻辑：`Register StackAddr =`。
- **L913 EN**: Continues logic with `Handler.getStackAddress(VA.getLocInfo() == CCValAssign::Indirect`.
  **L913 CN**: 继续处理逻辑：`Handler.getStackAddress(VA.getLocInfo() == CCValAssign::Indirect`。
- **L914 EN**: Continues logic with `? PointerTy.getSizeInBytes()`.
  **L914 CN**: 继续处理逻辑：`? PointerTy.getSizeInBytes()`。
- **L915 EN**: Continues logic with `: MemTy.getSizeInBytes(),`.
  **L915 CN**: 继续处理逻辑：`: MemTy.getSizeInBytes(),`。
- **L916 EN**: Executes statement `VA.getLocMemOffset(), MPO, Flags);`.
  **L916 CN**: 执行语句 `VA.getLocMemOffset(), MPO, Flags);`。
- **L917 EN**: Separates nearby statements for readability.
  **L917 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L918 EN**: Comment documents: `Finish the handling of indirect passing from the passers`.
  **L918 CN**: 注释说明：`Finish the handling of indirect passing from the passers`。
- **L919 EN**: Comment documents: `(OutgoingParameterHandler) side.`.
  **L919 CN**: 注释说明：`(OutgoingParameterHandler) side.`。
- **L920 EN**: Comment documents: `This branch is needed, so the pointer to the value is loaded onto the`.
  **L920 CN**: 注释说明：`This branch is needed, so the pointer to the value is loaded onto the`。

### Lines 921-940

````cpp
        // stack.
        if (VA.getLocInfo() == CCValAssign::Indirect)
          Handler.assignValueToAddress(ArgReg, StackAddr, PointerTy, MPO, VA);
        else
          Handler.assignValueToAddress(Args[i], Part, StackAddr, MemTy, MPO,
                                       VA);
      } else if (VA.isMemLoc() && Flags.isByVal()) {
        assert(Args[i].Regs.size() == 1 && "didn't expect split byval pointer");

        if (Handler.isIncomingArgumentHandler()) {
          // We just need to copy the frame index value to the pointer.
          MachinePointerInfo MPO;
          Register StackAddr = Handler.getStackAddress(
              Flags.getByValSize(), VA.getLocMemOffset(), MPO, Flags);
          MIRBuilder.buildCopy(Args[i].Regs[0], StackAddr);
        } else {
          // For outgoing byval arguments, insert the implicit copy byval
          // implies, such that writes in the callee do not modify the caller's
          // value.
          uint64_t MemSize = Flags.getByValSize();
````
- **L921 EN**: Comment documents: `stack.`.
  **L921 CN**: 注释说明：`stack.`。
- **L922 EN**: Begins a conditional branch.
  **L922 CN**: 开始一个条件分支。
- **L923 EN**: Executes statement `Handler.assignValueToAddress(ArgReg, StackAddr, PointerTy, MPO, VA);`.
  **L923 CN**: 执行语句 `Handler.assignValueToAddress(ArgReg, StackAddr, PointerTy, MPO, VA);`。
- **L924 EN**: Handles the fallback branch.
  **L924 CN**: 处理兜底分支。
- **L925 EN**: Continues logic with `Handler.assignValueToAddress(Args[i], Part, StackAddr, MemTy, MPO,`.
  **L925 CN**: 继续处理逻辑：`Handler.assignValueToAddress(Args[i], Part, StackAddr, MemTy, MPO,`。
- **L926 EN**: Executes statement `VA);`.
  **L926 CN**: 执行语句 `VA);`。
- **L927 EN**: Starts block `} else if (VA.isMemLoc() && Flags.isByVal())`.
  **L927 CN**: 开始代码块 `} else if (VA.isMemLoc() && Flags.isByVal())`。
- **L928 EN**: Checks an invariant in debug builds.
  **L928 CN**: 在调试构建中检查一个不变量。
- **L929 EN**: Separates nearby statements for readability.
  **L929 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L930 EN**: Begins a conditional branch.
  **L930 CN**: 开始一个条件分支。
- **L931 EN**: Comment documents: `We just need to copy the frame index value to the pointer.`.
  **L931 CN**: 注释说明：`We just need to copy the frame index value to the pointer.`。
- **L932 EN**: Executes statement `MachinePointerInfo MPO;`.
  **L932 CN**: 执行语句 `MachinePointerInfo MPO;`。
- **L933 EN**: Continues logic with `Register StackAddr = Handler.getStackAddress(`.
  **L933 CN**: 继续处理逻辑：`Register StackAddr = Handler.getStackAddress(`。
- **L934 EN**: Executes statement `Flags.getByValSize(), VA.getLocMemOffset(), MPO, Flags);`.
  **L934 CN**: 执行语句 `Flags.getByValSize(), VA.getLocMemOffset(), MPO, Flags);`。
- **L935 EN**: Executes statement `MIRBuilder.buildCopy(Args[i].Regs[0], StackAddr);`.
  **L935 CN**: 执行语句 `MIRBuilder.buildCopy(Args[i].Regs[0], StackAddr);`。
- **L936 EN**: Starts block `} else`.
  **L936 CN**: 开始代码块 `} else`。
- **L937 EN**: Comment documents: `For outgoing byval arguments, insert the implicit copy byval`.
  **L937 CN**: 注释说明：`For outgoing byval arguments, insert the implicit copy byval`。
- **L938 EN**: Comment documents: `implies, such that writes in the callee do not modify the caller's`.
  **L938 CN**: 注释说明：`implies, such that writes in the callee do not modify the caller's`。
- **L939 EN**: Comment documents: `value.`.
  **L939 CN**: 注释说明：`value.`。
- **L940 EN**: Assigns or initializes `uint64_t MemSize`.
  **L940 CN**: 对 `uint64_t MemSize` 进行赋值或初始化。

### Lines 941-960

````cpp
          int64_t Offset = VA.getLocMemOffset();

          MachinePointerInfo DstMPO;
          Register StackAddr =
              Handler.getStackAddress(MemSize, Offset, DstMPO, Flags);

          MachinePointerInfo SrcMPO(Args[i].OrigValue);
          if (!Args[i].OrigValue) {
            // We still need to accurately track the stack address space if we
            // don't know the underlying value.
            const LLT PtrTy = MRI.getType(StackAddr);
            SrcMPO = MachinePointerInfo(PtrTy.getAddressSpace());
          }

          Align DstAlign = std::max(Flags.getNonZeroByValAlign(),
                                    inferAlignFromPtrInfo(MF, DstMPO));

          Align SrcAlign = std::max(Flags.getNonZeroByValAlign(),
                                    inferAlignFromPtrInfo(MF, SrcMPO));

````
- **L941 EN**: Assigns or initializes `int64_t Offset`.
  **L941 CN**: 对 `int64_t Offset` 进行赋值或初始化。
- **L942 EN**: Separates nearby statements for readability.
  **L942 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L943 EN**: Executes statement `MachinePointerInfo DstMPO;`.
  **L943 CN**: 执行语句 `MachinePointerInfo DstMPO;`。
- **L944 EN**: Continues logic with `Register StackAddr =`.
  **L944 CN**: 继续处理逻辑：`Register StackAddr =`。
- **L945 EN**: Executes statement `Handler.getStackAddress(MemSize, Offset, DstMPO, Flags);`.
  **L945 CN**: 执行语句 `Handler.getStackAddress(MemSize, Offset, DstMPO, Flags);`。
- **L946 EN**: Separates nearby statements for readability.
  **L946 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L947 EN**: Declares function or method `SrcMPO`.
  **L947 CN**: 声明函数或方法 `SrcMPO`。
- **L948 EN**: Begins a conditional branch.
  **L948 CN**: 开始一个条件分支。
- **L949 EN**: Comment documents: `We still need to accurately track the stack address space if we`.
  **L949 CN**: 注释说明：`We still need to accurately track the stack address space if we`。
- **L950 EN**: Comment documents: `don't know the underlying value.`.
  **L950 CN**: 注释说明：`don't know the underlying value.`。
- **L951 EN**: Assigns or initializes `const LLT PtrTy`.
  **L951 CN**: 对 `const LLT PtrTy` 进行赋值或初始化。
- **L952 EN**: Assigns or initializes `SrcMPO`.
  **L952 CN**: 对 `SrcMPO` 进行赋值或初始化。
- **L953 EN**: Closes the current scope.
  **L953 CN**: 关闭当前作用域。
- **L954 EN**: Separates nearby statements for readability.
  **L954 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L955 EN**: Provides part of the signature for `max`.
  **L955 CN**: 给出 `max` 的一部分签名。
- **L956 EN**: Executes statement `inferAlignFromPtrInfo(MF, DstMPO));`.
  **L956 CN**: 执行语句 `inferAlignFromPtrInfo(MF, DstMPO));`。
- **L957 EN**: Separates nearby statements for readability.
  **L957 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L958 EN**: Provides part of the signature for `max`.
  **L958 CN**: 给出 `max` 的一部分签名。
- **L959 EN**: Executes statement `inferAlignFromPtrInfo(MF, SrcMPO));`.
  **L959 CN**: 执行语句 `inferAlignFromPtrInfo(MF, SrcMPO));`。
- **L960 EN**: Separates nearby statements for readability.
  **L960 CN**: 空行，用于分隔相邻语句并提升可读性。

### Lines 961-980

````cpp
          Handler.copyArgumentMemory(Args[i], StackAddr, Args[i].Regs[0],
                                     DstMPO, DstAlign, SrcMPO, SrcAlign,
                                     MemSize, VA);
        }
      } else if (i == 0 && !ThisReturnRegs.empty() &&
                 Handler.isIncomingArgumentHandler() &&
                 isTypeIsValidForThisReturn(ValVT)) {
        Handler.assignValueToReg(ArgReg, ThisReturnRegs[Part], VA, Flags);
      } else if (Handler.isIncomingArgumentHandler()) {
        Handler.assignValueToReg(ArgReg, VA.getLocReg(), VA, Flags);
      } else {
        DelayedOutgoingRegAssignments.emplace_back([=, &Handler]() {
          Handler.assignValueToReg(ArgReg, VA.getLocReg(), VA, Flags);
        });
      }

      // Finish the handling of indirect parameter passing when receiving
      // the value (we are in the called function or the caller when receiving
      // the return value).
      if (VA.getLocInfo() == CCValAssign::Indirect &&
````
- **L961 EN**: Continues logic with `Handler.copyArgumentMemory(Args[i], StackAddr, Args[i].Regs[0],`.
  **L961 CN**: 继续处理逻辑：`Handler.copyArgumentMemory(Args[i], StackAddr, Args[i].Regs[0],`。
- **L962 EN**: Continues logic with `DstMPO, DstAlign, SrcMPO, SrcAlign,`.
  **L962 CN**: 继续处理逻辑：`DstMPO, DstAlign, SrcMPO, SrcAlign,`。
- **L963 EN**: Executes statement `MemSize, VA);`.
  **L963 CN**: 执行语句 `MemSize, VA);`。
- **L964 EN**: Closes the current scope.
  **L964 CN**: 关闭当前作用域。
- **L965 EN**: Continues logic with `} else if (i == 0 && !ThisReturnRegs.empty() &&`.
  **L965 CN**: 继续处理逻辑：`} else if (i == 0 && !ThisReturnRegs.empty() &&`。
- **L966 EN**: Continues logic with `Handler.isIncomingArgumentHandler() &&`.
  **L966 CN**: 继续处理逻辑：`Handler.isIncomingArgumentHandler() &&`。
- **L967 EN**: Starts block `isTypeIsValidForThisReturn(ValVT))`.
  **L967 CN**: 开始代码块 `isTypeIsValidForThisReturn(ValVT))`。
- **L968 EN**: Executes statement `Handler.assignValueToReg(ArgReg, ThisReturnRegs[Part], VA, Flags);`.
  **L968 CN**: 执行语句 `Handler.assignValueToReg(ArgReg, ThisReturnRegs[Part], VA, Flags);`。
- **L969 EN**: Starts block `} else if (Handler.isIncomingArgumentHandler())`.
  **L969 CN**: 开始代码块 `} else if (Handler.isIncomingArgumentHandler())`。
- **L970 EN**: Executes statement `Handler.assignValueToReg(ArgReg, VA.getLocReg(), VA, Flags);`.
  **L970 CN**: 执行语句 `Handler.assignValueToReg(ArgReg, VA.getLocReg(), VA, Flags);`。
- **L971 EN**: Starts block `} else`.
  **L971 CN**: 开始代码块 `} else`。
- **L972 EN**: Starts block `DelayedOutgoingRegAssignments.emplace_back([=, &Handler]()`.
  **L972 CN**: 开始代码块 `DelayedOutgoingRegAssignments.emplace_back([=, &Handler]()`。
- **L973 EN**: Executes statement `Handler.assignValueToReg(ArgReg, VA.getLocReg(), VA, Flags);`.
  **L973 CN**: 执行语句 `Handler.assignValueToReg(ArgReg, VA.getLocReg(), VA, Flags);`。
- **L974 EN**: Executes statement `});`.
  **L974 CN**: 执行语句 `});`。
- **L975 EN**: Closes the current scope.
  **L975 CN**: 关闭当前作用域。
- **L976 EN**: Separates nearby statements for readability.
  **L976 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L977 EN**: Comment documents: `Finish the handling of indirect parameter passing when receiving`.
  **L977 CN**: 注释说明：`Finish the handling of indirect parameter passing when receiving`。
- **L978 EN**: Comment documents: `the value (we are in the called function or the caller when receiving`.
  **L978 CN**: 注释说明：`the value (we are in the called function or the caller when receiving`。
- **L979 EN**: Comment documents: `the return value).`.
  **L979 CN**: 注释说明：`the return value).`。
- **L980 EN**: Begins a conditional branch.
  **L980 CN**: 开始一个条件分支。

### Lines 981-1000

````cpp
          Handler.isIncomingArgumentHandler()) {
        Align Alignment = DL.getABITypeAlign(Args[i].Ty);
        MachinePointerInfo MPO = MachinePointerInfo::getUnknownStack(MF);

        // Since we are doing indirect parameter passing, we know that the value
        // in the temporary register is not the value passed to the function,
        // but rather a pointer to that value. Let's load that value into the
        // virtual register where the parameter should go.
        MIRBuilder.buildLoad(Args[i].OrigRegs[0], Args[i].Regs[0], MPO,
                             Alignment);

        IndirectParameterPassingHandled = true;
      }

      if (IndirectParameterPassingHandled)
        break;
    }

    // Now that all pieces have been assigned, re-pack the register typed values
    // into the original value typed registers. This is only necessary, when
````
- **L981 EN**: Starts block `Handler.isIncomingArgumentHandler())`.
  **L981 CN**: 开始代码块 `Handler.isIncomingArgumentHandler())`。
- **L982 EN**: Assigns or initializes `Align Alignment`.
  **L982 CN**: 对 `Align Alignment` 进行赋值或初始化。
- **L983 EN**: Declares function or method `getUnknownStack`.
  **L983 CN**: 声明函数或方法 `getUnknownStack`。
- **L984 EN**: Separates nearby statements for readability.
  **L984 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L985 EN**: Comment documents: `Since we are doing indirect parameter passing, we know that the value`.
  **L985 CN**: 注释说明：`Since we are doing indirect parameter passing, we know that the value`。
- **L986 EN**: Comment documents: `in the temporary register is not the value passed to the function,`.
  **L986 CN**: 注释说明：`in the temporary register is not the value passed to the function,`。
- **L987 EN**: Comment documents: `but rather a pointer to that value. Let's load that value into the`.
  **L987 CN**: 注释说明：`but rather a pointer to that value. Let's load that value into the`。
- **L988 EN**: Comment documents: `virtual register where the parameter should go.`.
  **L988 CN**: 注释说明：`virtual register where the parameter should go.`。
- **L989 EN**: Continues logic with `MIRBuilder.buildLoad(Args[i].OrigRegs[0], Args[i].Regs[0], MPO,`.
  **L989 CN**: 继续处理逻辑：`MIRBuilder.buildLoad(Args[i].OrigRegs[0], Args[i].Regs[0], MPO,`。
- **L990 EN**: Executes statement `Alignment);`.
  **L990 CN**: 执行语句 `Alignment);`。
- **L991 EN**: Separates nearby statements for readability.
  **L991 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L992 EN**: Assigns or initializes `IndirectParameterPassingHandled`.
  **L992 CN**: 对 `IndirectParameterPassingHandled` 进行赋值或初始化。
- **L993 EN**: Closes the current scope.
  **L993 CN**: 关闭当前作用域。
- **L994 EN**: Separates nearby statements for readability.
  **L994 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L995 EN**: Begins a conditional branch.
  **L995 CN**: 开始一个条件分支。
- **L996 EN**: Breaks out of the current control-flow construct.
  **L996 CN**: 跳出当前控制流结构。
- **L997 EN**: Closes the current scope.
  **L997 CN**: 关闭当前作用域。
- **L998 EN**: Separates nearby statements for readability.
  **L998 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L999 EN**: Comment documents: `Now that all pieces have been assigned, re-pack the register typed value…`.
  **L999 CN**: 注释说明：`Now that all pieces have been assigned, re-pack the register typed value…`。
- **L1000 EN**: Comment documents: `into the original value typed registers. This is only necessary, when`.
  **L1000 CN**: 注释说明：`into the original value typed registers. This is only necessary, when`。

### Lines 1001-1020

````cpp
    // the value was passed in multiple registers, not indirectly.
    if (Handler.isIncomingArgumentHandler() && OrigVT != LocVT &&
        !IndirectParameterPassingHandled) {
      // Merge the split registers into the expected larger result vregs of
      // the original call.
      buildCopyFromRegs(MIRBuilder, Args[i].OrigRegs, Args[i].Regs, OrigTy,
                        LocTy, Args[i].Flags[0]);
    }

    j += NumParts - 1;
  }
  for (auto &Fn : DelayedOutgoingRegAssignments)
    Fn();

  return true;
}

void CallLowering::insertSRetLoads(MachineIRBuilder &MIRBuilder, Type *RetTy,
                                   ArrayRef<Register> VRegs, Register DemoteReg,
                                   int FI) const {
````
- **L1001 EN**: Comment documents: `the value was passed in multiple registers, not indirectly.`.
  **L1001 CN**: 注释说明：`the value was passed in multiple registers, not indirectly.`。
- **L1002 EN**: Begins a conditional branch.
  **L1002 CN**: 开始一个条件分支。
- **L1003 EN**: Starts block `!IndirectParameterPassingHandled)`.
  **L1003 CN**: 开始代码块 `!IndirectParameterPassingHandled)`。
- **L1004 EN**: Comment documents: `Merge the split registers into the expected larger result vregs of`.
  **L1004 CN**: 注释说明：`Merge the split registers into the expected larger result vregs of`。
- **L1005 EN**: Comment documents: `the original call.`.
  **L1005 CN**: 注释说明：`the original call.`。
- **L1006 EN**: Continues logic with `buildCopyFromRegs(MIRBuilder, Args[i].OrigRegs, Args[i].Regs, OrigTy,`.
  **L1006 CN**: 继续处理逻辑：`buildCopyFromRegs(MIRBuilder, Args[i].OrigRegs, Args[i].Regs, OrigTy,`。
- **L1007 EN**: Executes statement `LocTy, Args[i].Flags[0]);`.
  **L1007 CN**: 执行语句 `LocTy, Args[i].Flags[0]);`。
- **L1008 EN**: Closes the current scope.
  **L1008 CN**: 关闭当前作用域。
- **L1009 EN**: Separates nearby statements for readability.
  **L1009 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L1010 EN**: Assigns or initializes `j +`.
  **L1010 CN**: 对 `j +` 进行赋值或初始化。
- **L1011 EN**: Closes the current scope.
  **L1011 CN**: 关闭当前作用域。
- **L1012 EN**: Starts a loop over a sequence or range.
  **L1012 CN**: 开始遍历序列或范围的循环。
- **L1013 EN**: Executes statement `Fn();`.
  **L1013 CN**: 执行语句 `Fn();`。
- **L1014 EN**: Separates nearby statements for readability.
  **L1014 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L1015 EN**: Returns `true` to the caller.
  **L1015 CN**: 向调用者返回 `true`。
- **L1016 EN**: Closes the current scope.
  **L1016 CN**: 关闭当前作用域。
- **L1017 EN**: Separates nearby statements for readability.
  **L1017 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L1018 EN**: Provides part of the signature for `insertSRetLoads`.
  **L1018 CN**: 给出 `insertSRetLoads` 的一部分签名。
- **L1019 EN**: Continues logic with `ArrayRef<Register> VRegs, Register DemoteReg,`.
  **L1019 CN**: 继续处理逻辑：`ArrayRef<Register> VRegs, Register DemoteReg,`。
- **L1020 EN**: Starts block `int FI) const`.
  **L1020 CN**: 开始代码块 `int FI) const`。

### Lines 1021-1040

````cpp
  MachineFunction &MF = MIRBuilder.getMF();
  MachineRegisterInfo &MRI = MF.getRegInfo();
  const DataLayout &DL = MF.getDataLayout();

  SmallVector<EVT, 4> SplitVTs;
  SmallVector<uint64_t, 4> Offsets;
  ComputeValueVTs(*TLI, DL, RetTy, SplitVTs, /*MemVTs=*/nullptr, &Offsets, 0);

  assert(VRegs.size() == SplitVTs.size());

  unsigned NumValues = SplitVTs.size();
  Align BaseAlign = DL.getPrefTypeAlign(RetTy);
  Type *RetPtrTy =
      PointerType::get(RetTy->getContext(), DL.getAllocaAddrSpace());
  LLT OffsetLLTy = getLLTForType(*DL.getIndexType(RetPtrTy), DL);

  MachinePointerInfo PtrInfo = MachinePointerInfo::getFixedStack(MF, FI);

  for (unsigned I = 0; I < NumValues; ++I) {
    Register Addr;
````
- **L1021 EN**: Assigns or initializes `MachineFunction &MF`.
  **L1021 CN**: 对 `MachineFunction &MF` 进行赋值或初始化。
- **L1022 EN**: Assigns or initializes `MachineRegisterInfo &MRI`.
  **L1022 CN**: 对 `MachineRegisterInfo &MRI` 进行赋值或初始化。
- **L1023 EN**: Assigns or initializes `const DataLayout &DL`.
  **L1023 CN**: 对 `const DataLayout &DL` 进行赋值或初始化。
- **L1024 EN**: Separates nearby statements for readability.
  **L1024 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L1025 EN**: Executes statement `SmallVector<EVT, 4> SplitVTs;`.
  **L1025 CN**: 执行语句 `SmallVector<EVT, 4> SplitVTs;`。
- **L1026 EN**: Executes statement `SmallVector<uint64_t, 4> Offsets;`.
  **L1026 CN**: 执行语句 `SmallVector<uint64_t, 4> Offsets;`。
- **L1027 EN**: Assigns or initializes `ComputeValueVTs(*TLI, DL, RetTy, SplitVTs, /*MemVTs`.
  **L1027 CN**: 对 `ComputeValueVTs(*TLI, DL, RetTy, SplitVTs, /*MemVTs` 进行赋值或初始化。
- **L1028 EN**: Separates nearby statements for readability.
  **L1028 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L1029 EN**: Checks an invariant in debug builds.
  **L1029 CN**: 在调试构建中检查一个不变量。
- **L1030 EN**: Separates nearby statements for readability.
  **L1030 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L1031 EN**: Assigns or initializes `unsigned NumValues`.
  **L1031 CN**: 对 `unsigned NumValues` 进行赋值或初始化。
- **L1032 EN**: Assigns or initializes `Align BaseAlign`.
  **L1032 CN**: 对 `Align BaseAlign` 进行赋值或初始化。
- **L1033 EN**: Continues logic with `Type *RetPtrTy =`.
  **L1033 CN**: 继续处理逻辑：`Type *RetPtrTy =`。
- **L1034 EN**: Declares function or method `get`.
  **L1034 CN**: 声明函数或方法 `get`。
- **L1035 EN**: Assigns or initializes `LLT OffsetLLTy`.
  **L1035 CN**: 对 `LLT OffsetLLTy` 进行赋值或初始化。
- **L1036 EN**: Separates nearby statements for readability.
  **L1036 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L1037 EN**: Declares function or method `getFixedStack`.
  **L1037 CN**: 声明函数或方法 `getFixedStack`。
- **L1038 EN**: Separates nearby statements for readability.
  **L1038 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L1039 EN**: Starts a loop over a sequence or range.
  **L1039 CN**: 开始遍历序列或范围的循环。
- **L1040 EN**: Executes statement `Register Addr;`.
  **L1040 CN**: 执行语句 `Register Addr;`。

### Lines 1041-1060

````cpp
    MIRBuilder.materializeObjectPtrOffset(Addr, DemoteReg, OffsetLLTy,
                                          Offsets[I]);
    auto *MMO = MF.getMachineMemOperand(PtrInfo, MachineMemOperand::MOLoad,
                                        MRI.getType(VRegs[I]),
                                        commonAlignment(BaseAlign, Offsets[I]));
    MIRBuilder.buildLoad(VRegs[I], Addr, *MMO);
  }
}

void CallLowering::insertSRetStores(MachineIRBuilder &MIRBuilder, Type *RetTy,
                                    ArrayRef<Register> VRegs,
                                    Register DemoteReg) const {
  MachineFunction &MF = MIRBuilder.getMF();
  MachineRegisterInfo &MRI = MF.getRegInfo();
  const DataLayout &DL = MF.getDataLayout();

  SmallVector<EVT, 4> SplitVTs;
  SmallVector<uint64_t, 4> Offsets;
  ComputeValueVTs(*TLI, DL, RetTy, SplitVTs, /*MemVTs=*/nullptr, &Offsets, 0);

````
- **L1041 EN**: Continues logic with `MIRBuilder.materializeObjectPtrOffset(Addr, DemoteReg, OffsetLLTy,`.
  **L1041 CN**: 继续处理逻辑：`MIRBuilder.materializeObjectPtrOffset(Addr, DemoteReg, OffsetLLTy,`。
- **L1042 EN**: Executes statement `Offsets[I]);`.
  **L1042 CN**: 执行语句 `Offsets[I]);`。
- **L1043 EN**: Continues logic with `auto *MMO = MF.getMachineMemOperand(PtrInfo, MachineMemOperand::MOLoad,`.
  **L1043 CN**: 继续处理逻辑：`auto *MMO = MF.getMachineMemOperand(PtrInfo, MachineMemOperand::MOLoad,`。
- **L1044 EN**: Continues logic with `MRI.getType(VRegs[I]),`.
  **L1044 CN**: 继续处理逻辑：`MRI.getType(VRegs[I]),`。
- **L1045 EN**: Executes statement `commonAlignment(BaseAlign, Offsets[I]));`.
  **L1045 CN**: 执行语句 `commonAlignment(BaseAlign, Offsets[I]));`。
- **L1046 EN**: Executes statement `MIRBuilder.buildLoad(VRegs[I], Addr, *MMO);`.
  **L1046 CN**: 执行语句 `MIRBuilder.buildLoad(VRegs[I], Addr, *MMO);`。
- **L1047 EN**: Closes the current scope.
  **L1047 CN**: 关闭当前作用域。
- **L1048 EN**: Closes the current scope.
  **L1048 CN**: 关闭当前作用域。
- **L1049 EN**: Separates nearby statements for readability.
  **L1049 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L1050 EN**: Provides part of the signature for `insertSRetStores`.
  **L1050 CN**: 给出 `insertSRetStores` 的一部分签名。
- **L1051 EN**: Continues logic with `ArrayRef<Register> VRegs,`.
  **L1051 CN**: 继续处理逻辑：`ArrayRef<Register> VRegs,`。
- **L1052 EN**: Starts block `Register DemoteReg) const`.
  **L1052 CN**: 开始代码块 `Register DemoteReg) const`。
- **L1053 EN**: Assigns or initializes `MachineFunction &MF`.
  **L1053 CN**: 对 `MachineFunction &MF` 进行赋值或初始化。
- **L1054 EN**: Assigns or initializes `MachineRegisterInfo &MRI`.
  **L1054 CN**: 对 `MachineRegisterInfo &MRI` 进行赋值或初始化。
- **L1055 EN**: Assigns or initializes `const DataLayout &DL`.
  **L1055 CN**: 对 `const DataLayout &DL` 进行赋值或初始化。
- **L1056 EN**: Separates nearby statements for readability.
  **L1056 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L1057 EN**: Executes statement `SmallVector<EVT, 4> SplitVTs;`.
  **L1057 CN**: 执行语句 `SmallVector<EVT, 4> SplitVTs;`。
- **L1058 EN**: Executes statement `SmallVector<uint64_t, 4> Offsets;`.
  **L1058 CN**: 执行语句 `SmallVector<uint64_t, 4> Offsets;`。
- **L1059 EN**: Assigns or initializes `ComputeValueVTs(*TLI, DL, RetTy, SplitVTs, /*MemVTs`.
  **L1059 CN**: 对 `ComputeValueVTs(*TLI, DL, RetTy, SplitVTs, /*MemVTs` 进行赋值或初始化。
- **L1060 EN**: Separates nearby statements for readability.
  **L1060 CN**: 空行，用于分隔相邻语句并提升可读性。

### Lines 1061-1080

````cpp
  assert(VRegs.size() == SplitVTs.size());

  unsigned NumValues = SplitVTs.size();
  Align BaseAlign = DL.getPrefTypeAlign(RetTy);
  unsigned AS = DL.getAllocaAddrSpace();
  LLT OffsetLLTy = getLLTForType(*DL.getIndexType(RetTy->getContext(), AS), DL);

  MachinePointerInfo PtrInfo(AS);

  for (unsigned I = 0; I < NumValues; ++I) {
    Register Addr;
    MIRBuilder.materializeObjectPtrOffset(Addr, DemoteReg, OffsetLLTy,
                                          Offsets[I]);
    auto *MMO = MF.getMachineMemOperand(PtrInfo, MachineMemOperand::MOStore,
                                        MRI.getType(VRegs[I]),
                                        commonAlignment(BaseAlign, Offsets[I]));
    MIRBuilder.buildStore(VRegs[I], Addr, *MMO);
  }
}

````
- **L1061 EN**: Checks an invariant in debug builds.
  **L1061 CN**: 在调试构建中检查一个不变量。
- **L1062 EN**: Separates nearby statements for readability.
  **L1062 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L1063 EN**: Assigns or initializes `unsigned NumValues`.
  **L1063 CN**: 对 `unsigned NumValues` 进行赋值或初始化。
- **L1064 EN**: Assigns or initializes `Align BaseAlign`.
  **L1064 CN**: 对 `Align BaseAlign` 进行赋值或初始化。
- **L1065 EN**: Assigns or initializes `unsigned AS`.
  **L1065 CN**: 对 `unsigned AS` 进行赋值或初始化。
- **L1066 EN**: Assigns or initializes `LLT OffsetLLTy`.
  **L1066 CN**: 对 `LLT OffsetLLTy` 进行赋值或初始化。
- **L1067 EN**: Separates nearby statements for readability.
  **L1067 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L1068 EN**: Declares function or method `PtrInfo`.
  **L1068 CN**: 声明函数或方法 `PtrInfo`。
- **L1069 EN**: Separates nearby statements for readability.
  **L1069 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L1070 EN**: Starts a loop over a sequence or range.
  **L1070 CN**: 开始遍历序列或范围的循环。
- **L1071 EN**: Executes statement `Register Addr;`.
  **L1071 CN**: 执行语句 `Register Addr;`。
- **L1072 EN**: Continues logic with `MIRBuilder.materializeObjectPtrOffset(Addr, DemoteReg, OffsetLLTy,`.
  **L1072 CN**: 继续处理逻辑：`MIRBuilder.materializeObjectPtrOffset(Addr, DemoteReg, OffsetLLTy,`。
- **L1073 EN**: Executes statement `Offsets[I]);`.
  **L1073 CN**: 执行语句 `Offsets[I]);`。
- **L1074 EN**: Continues logic with `auto *MMO = MF.getMachineMemOperand(PtrInfo, MachineMemOperand::MOStore,`.
  **L1074 CN**: 继续处理逻辑：`auto *MMO = MF.getMachineMemOperand(PtrInfo, MachineMemOperand::MOStore,`。
- **L1075 EN**: Continues logic with `MRI.getType(VRegs[I]),`.
  **L1075 CN**: 继续处理逻辑：`MRI.getType(VRegs[I]),`。
- **L1076 EN**: Executes statement `commonAlignment(BaseAlign, Offsets[I]));`.
  **L1076 CN**: 执行语句 `commonAlignment(BaseAlign, Offsets[I]));`。
- **L1077 EN**: Executes statement `MIRBuilder.buildStore(VRegs[I], Addr, *MMO);`.
  **L1077 CN**: 执行语句 `MIRBuilder.buildStore(VRegs[I], Addr, *MMO);`。
- **L1078 EN**: Closes the current scope.
  **L1078 CN**: 关闭当前作用域。
- **L1079 EN**: Closes the current scope.
  **L1079 CN**: 关闭当前作用域。
- **L1080 EN**: Separates nearby statements for readability.
  **L1080 CN**: 空行，用于分隔相邻语句并提升可读性。

### Lines 1081-1100

````cpp
void CallLowering::insertSRetIncomingArgument(
    const Function &F, SmallVectorImpl<ArgInfo> &SplitArgs, Register &DemoteReg,
    MachineRegisterInfo &MRI, const DataLayout &DL) const {
  unsigned AS = DL.getAllocaAddrSpace();
  DemoteReg = MRI.createGenericVirtualRegister(
      LLT::pointer(AS, DL.getPointerSizeInBits(AS)));

  Type *PtrTy = PointerType::get(F.getContext(), AS);

  SmallVector<EVT, 1> ValueVTs;
  ComputeValueVTs(*TLI, DL, PtrTy, ValueVTs);

  // NOTE: Assume that a pointer won't get split into more than one VT.
  assert(ValueVTs.size() == 1);

  ArgInfo DemoteArg(DemoteReg, ValueVTs[0].getTypeForEVT(PtrTy->getContext()),
                    ArgInfo::NoArgIndex);
  setArgFlags(DemoteArg, AttributeList::ReturnIndex, DL, F);
  DemoteArg.Flags[0].setSRet();
  SplitArgs.insert(SplitArgs.begin(), DemoteArg);
````
- **L1081 EN**: Provides part of the signature for `insertSRetIncomingArgument`.
  **L1081 CN**: 给出 `insertSRetIncomingArgument` 的一部分签名。
- **L1082 EN**: Continues logic with `const Function &F, SmallVectorImpl<ArgInfo> &SplitArgs, Register &Demote…`.
  **L1082 CN**: 继续处理逻辑：`const Function &F, SmallVectorImpl<ArgInfo> &SplitArgs, Register &Demote…`。
- **L1083 EN**: Starts block `MachineRegisterInfo &MRI, const DataLayout &DL) const`.
  **L1083 CN**: 开始代码块 `MachineRegisterInfo &MRI, const DataLayout &DL) const`。
- **L1084 EN**: Assigns or initializes `unsigned AS`.
  **L1084 CN**: 对 `unsigned AS` 进行赋值或初始化。
- **L1085 EN**: Continues logic with `DemoteReg = MRI.createGenericVirtualRegister(`.
  **L1085 CN**: 继续处理逻辑：`DemoteReg = MRI.createGenericVirtualRegister(`。
- **L1086 EN**: Declares function or method `pointer`.
  **L1086 CN**: 声明函数或方法 `pointer`。
- **L1087 EN**: Separates nearby statements for readability.
  **L1087 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L1088 EN**: Declares function or method `get`.
  **L1088 CN**: 声明函数或方法 `get`。
- **L1089 EN**: Separates nearby statements for readability.
  **L1089 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L1090 EN**: Executes statement `SmallVector<EVT, 1> ValueVTs;`.
  **L1090 CN**: 执行语句 `SmallVector<EVT, 1> ValueVTs;`。
- **L1091 EN**: Executes statement `ComputeValueVTs(*TLI, DL, PtrTy, ValueVTs);`.
  **L1091 CN**: 执行语句 `ComputeValueVTs(*TLI, DL, PtrTy, ValueVTs);`。
- **L1092 EN**: Separates nearby statements for readability.
  **L1092 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L1093 EN**: Comment documents: `NOTE: Assume that a pointer won't get split into more than one VT.`.
  **L1093 CN**: 注释说明：`NOTE: Assume that a pointer won't get split into more than one VT.`。
- **L1094 EN**: Checks an invariant in debug builds.
  **L1094 CN**: 在调试构建中检查一个不变量。
- **L1095 EN**: Separates nearby statements for readability.
  **L1095 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L1096 EN**: Provides part of the signature for `DemoteArg`.
  **L1096 CN**: 给出 `DemoteArg` 的一部分签名。
- **L1097 EN**: Executes statement `ArgInfo::NoArgIndex);`.
  **L1097 CN**: 执行语句 `ArgInfo::NoArgIndex);`。
- **L1098 EN**: Executes statement `setArgFlags(DemoteArg, AttributeList::ReturnIndex, DL, F);`.
  **L1098 CN**: 执行语句 `setArgFlags(DemoteArg, AttributeList::ReturnIndex, DL, F);`。
- **L1099 EN**: Executes statement `DemoteArg.Flags[0].setSRet();`.
  **L1099 CN**: 执行语句 `DemoteArg.Flags[0].setSRet();`。
- **L1100 EN**: Executes statement `SplitArgs.insert(SplitArgs.begin(), DemoteArg);`.
  **L1100 CN**: 执行语句 `SplitArgs.insert(SplitArgs.begin(), DemoteArg);`。

### Lines 1101-1120

````cpp
}

void CallLowering::insertSRetOutgoingArgument(MachineIRBuilder &MIRBuilder,
                                              const CallBase &CB,
                                              CallLoweringInfo &Info) const {
  const DataLayout &DL = MIRBuilder.getDataLayout();
  Type *RetTy = CB.getType();
  unsigned AS = DL.getAllocaAddrSpace();
  LLT FramePtrTy = LLT::pointer(AS, DL.getPointerSizeInBits(AS));

  int FI = MIRBuilder.getMF().getFrameInfo().CreateStackObject(
      DL.getTypeAllocSize(RetTy), DL.getPrefTypeAlign(RetTy), false);

  Register DemoteReg = MIRBuilder.buildFrameIndex(FramePtrTy, FI).getReg(0);
  ArgInfo DemoteArg(DemoteReg, PointerType::get(RetTy->getContext(), AS),
                    ArgInfo::NoArgIndex);
  setArgFlags(DemoteArg, AttributeList::ReturnIndex, DL, CB);
  DemoteArg.Flags[0].setSRet();

  Info.OrigArgs.insert(Info.OrigArgs.begin(), DemoteArg);
````
- **L1101 EN**: Closes the current scope.
  **L1101 CN**: 关闭当前作用域。
- **L1102 EN**: Separates nearby statements for readability.
  **L1102 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L1103 EN**: Provides part of the signature for `insertSRetOutgoingArgument`.
  **L1103 CN**: 给出 `insertSRetOutgoingArgument` 的一部分签名。
- **L1104 EN**: Continues logic with `const CallBase &CB,`.
  **L1104 CN**: 继续处理逻辑：`const CallBase &CB,`。
- **L1105 EN**: Starts block `CallLoweringInfo &Info) const`.
  **L1105 CN**: 开始代码块 `CallLoweringInfo &Info) const`。
- **L1106 EN**: Assigns or initializes `const DataLayout &DL`.
  **L1106 CN**: 对 `const DataLayout &DL` 进行赋值或初始化。
- **L1107 EN**: Assigns or initializes `Type *RetTy`.
  **L1107 CN**: 对 `Type *RetTy` 进行赋值或初始化。
- **L1108 EN**: Assigns or initializes `unsigned AS`.
  **L1108 CN**: 对 `unsigned AS` 进行赋值或初始化。
- **L1109 EN**: Declares function or method `pointer`.
  **L1109 CN**: 声明函数或方法 `pointer`。
- **L1110 EN**: Separates nearby statements for readability.
  **L1110 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L1111 EN**: Continues logic with `int FI = MIRBuilder.getMF().getFrameInfo().CreateStackObject(`.
  **L1111 CN**: 继续处理逻辑：`int FI = MIRBuilder.getMF().getFrameInfo().CreateStackObject(`。
- **L1112 EN**: Executes statement `DL.getTypeAllocSize(RetTy), DL.getPrefTypeAlign(RetTy), false);`.
  **L1112 CN**: 执行语句 `DL.getTypeAllocSize(RetTy), DL.getPrefTypeAlign(RetTy), false);`。
- **L1113 EN**: Separates nearby statements for readability.
  **L1113 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L1114 EN**: Assigns or initializes `Register DemoteReg`.
  **L1114 CN**: 对 `Register DemoteReg` 进行赋值或初始化。
- **L1115 EN**: Provides part of the signature for `DemoteArg`.
  **L1115 CN**: 给出 `DemoteArg` 的一部分签名。
- **L1116 EN**: Executes statement `ArgInfo::NoArgIndex);`.
  **L1116 CN**: 执行语句 `ArgInfo::NoArgIndex);`。
- **L1117 EN**: Executes statement `setArgFlags(DemoteArg, AttributeList::ReturnIndex, DL, CB);`.
  **L1117 CN**: 执行语句 `setArgFlags(DemoteArg, AttributeList::ReturnIndex, DL, CB);`。
- **L1118 EN**: Executes statement `DemoteArg.Flags[0].setSRet();`.
  **L1118 CN**: 执行语句 `DemoteArg.Flags[0].setSRet();`。
- **L1119 EN**: Separates nearby statements for readability.
  **L1119 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L1120 EN**: Executes statement `Info.OrigArgs.insert(Info.OrigArgs.begin(), DemoteArg);`.
  **L1120 CN**: 执行语句 `Info.OrigArgs.insert(Info.OrigArgs.begin(), DemoteArg);`。

### Lines 1121-1140

````cpp
  Info.DemoteStackIndex = FI;
  Info.DemoteRegister = DemoteReg;
}

bool CallLowering::checkReturn(CCState &CCInfo,
                               SmallVectorImpl<BaseArgInfo> &Outs,
                               CCAssignFn *Fn) const {
  for (unsigned I = 0, E = Outs.size(); I < E; ++I) {
    MVT VT = MVT::getVT(Outs[I].Ty);
    if (Fn(I, VT, VT, CCValAssign::Full, Outs[I].Flags[0], Outs[I].Ty, CCInfo))
      return false;
  }
  return true;
}

void CallLowering::getReturnInfo(CallingConv::ID CallConv, Type *RetTy,
                                 AttributeList Attrs,
                                 SmallVectorImpl<BaseArgInfo> &Outs,
                                 const DataLayout &DL) const {
  LLVMContext &Context = RetTy->getContext();
````
- **L1121 EN**: Assigns or initializes `Info.DemoteStackIndex`.
  **L1121 CN**: 对 `Info.DemoteStackIndex` 进行赋值或初始化。
- **L1122 EN**: Assigns or initializes `Info.DemoteRegister`.
  **L1122 CN**: 对 `Info.DemoteRegister` 进行赋值或初始化。
- **L1123 EN**: Closes the current scope.
  **L1123 CN**: 关闭当前作用域。
- **L1124 EN**: Separates nearby statements for readability.
  **L1124 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L1125 EN**: Provides part of the signature for `checkReturn`.
  **L1125 CN**: 给出 `checkReturn` 的一部分签名。
- **L1126 EN**: Continues logic with `SmallVectorImpl<BaseArgInfo> &Outs,`.
  **L1126 CN**: 继续处理逻辑：`SmallVectorImpl<BaseArgInfo> &Outs,`。
- **L1127 EN**: Starts block `CCAssignFn *Fn) const`.
  **L1127 CN**: 开始代码块 `CCAssignFn *Fn) const`。
- **L1128 EN**: Starts a loop over a sequence or range.
  **L1128 CN**: 开始遍历序列或范围的循环。
- **L1129 EN**: Declares function or method `getVT`.
  **L1129 CN**: 声明函数或方法 `getVT`。
- **L1130 EN**: Begins a conditional branch.
  **L1130 CN**: 开始一个条件分支。
- **L1131 EN**: Returns `false` to the caller.
  **L1131 CN**: 向调用者返回 `false`。
- **L1132 EN**: Closes the current scope.
  **L1132 CN**: 关闭当前作用域。
- **L1133 EN**: Returns `true` to the caller.
  **L1133 CN**: 向调用者返回 `true`。
- **L1134 EN**: Closes the current scope.
  **L1134 CN**: 关闭当前作用域。
- **L1135 EN**: Separates nearby statements for readability.
  **L1135 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L1136 EN**: Provides part of the signature for `getReturnInfo`.
  **L1136 CN**: 给出 `getReturnInfo` 的一部分签名。
- **L1137 EN**: Continues logic with `AttributeList Attrs,`.
  **L1137 CN**: 继续处理逻辑：`AttributeList Attrs,`。
- **L1138 EN**: Continues logic with `SmallVectorImpl<BaseArgInfo> &Outs,`.
  **L1138 CN**: 继续处理逻辑：`SmallVectorImpl<BaseArgInfo> &Outs,`。
- **L1139 EN**: Starts block `const DataLayout &DL) const`.
  **L1139 CN**: 开始代码块 `const DataLayout &DL) const`。
- **L1140 EN**: Assigns or initializes `LLVMContext &Context`.
  **L1140 CN**: 对 `LLVMContext &Context` 进行赋值或初始化。

### Lines 1141-1160

````cpp
  ISD::ArgFlagsTy Flags = ISD::ArgFlagsTy();

  SmallVector<EVT, 4> SplitVTs;
  ComputeValueVTs(*TLI, DL, RetTy, SplitVTs);
  addArgFlagsFromAttributes(Flags, Attrs, AttributeList::ReturnIndex);

  for (EVT VT : SplitVTs) {
    unsigned NumParts =
        TLI->getNumRegistersForCallingConv(Context, CallConv, VT);
    MVT RegVT = TLI->getRegisterTypeForCallingConv(Context, CallConv, VT);
    Type *PartTy = EVT(RegVT).getTypeForEVT(Context);

    for (unsigned I = 0; I < NumParts; ++I) {
      Outs.emplace_back(PartTy, Flags);
    }
  }
}

bool CallLowering::checkReturnTypeForCallConv(MachineFunction &MF) const {
  const auto &F = MF.getFunction();
````
- **L1141 EN**: Declares function or method `ArgFlagsTy`.
  **L1141 CN**: 声明函数或方法 `ArgFlagsTy`。
- **L1142 EN**: Separates nearby statements for readability.
  **L1142 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L1143 EN**: Executes statement `SmallVector<EVT, 4> SplitVTs;`.
  **L1143 CN**: 执行语句 `SmallVector<EVT, 4> SplitVTs;`。
- **L1144 EN**: Executes statement `ComputeValueVTs(*TLI, DL, RetTy, SplitVTs);`.
  **L1144 CN**: 执行语句 `ComputeValueVTs(*TLI, DL, RetTy, SplitVTs);`。
- **L1145 EN**: Executes statement `addArgFlagsFromAttributes(Flags, Attrs, AttributeList::ReturnIndex);`.
  **L1145 CN**: 执行语句 `addArgFlagsFromAttributes(Flags, Attrs, AttributeList::ReturnIndex);`。
- **L1146 EN**: Separates nearby statements for readability.
  **L1146 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L1147 EN**: Starts a loop over a sequence or range.
  **L1147 CN**: 开始遍历序列或范围的循环。
- **L1148 EN**: Continues logic with `unsigned NumParts =`.
  **L1148 CN**: 继续处理逻辑：`unsigned NumParts =`。
- **L1149 EN**: Executes statement `TLI->getNumRegistersForCallingConv(Context, CallConv, VT);`.
  **L1149 CN**: 执行语句 `TLI->getNumRegistersForCallingConv(Context, CallConv, VT);`。
- **L1150 EN**: Assigns or initializes `MVT RegVT`.
  **L1150 CN**: 对 `MVT RegVT` 进行赋值或初始化。
- **L1151 EN**: Assigns or initializes `Type *PartTy`.
  **L1151 CN**: 对 `Type *PartTy` 进行赋值或初始化。
- **L1152 EN**: Separates nearby statements for readability.
  **L1152 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L1153 EN**: Starts a loop over a sequence or range.
  **L1153 CN**: 开始遍历序列或范围的循环。
- **L1154 EN**: Executes statement `Outs.emplace_back(PartTy, Flags);`.
  **L1154 CN**: 执行语句 `Outs.emplace_back(PartTy, Flags);`。
- **L1155 EN**: Closes the current scope.
  **L1155 CN**: 关闭当前作用域。
- **L1156 EN**: Closes the current scope.
  **L1156 CN**: 关闭当前作用域。
- **L1157 EN**: Closes the current scope.
  **L1157 CN**: 关闭当前作用域。
- **L1158 EN**: Separates nearby statements for readability.
  **L1158 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L1159 EN**: Begins the definition of `checkReturnTypeForCallConv`.
  **L1159 CN**: 开始定义 `checkReturnTypeForCallConv`。
- **L1160 EN**: Assigns or initializes `const auto &F`.
  **L1160 CN**: 对 `const auto &F` 进行赋值或初始化。

### Lines 1161-1180

````cpp
  Type *ReturnType = F.getReturnType();
  CallingConv::ID CallConv = F.getCallingConv();

  SmallVector<BaseArgInfo, 4> SplitArgs;
  getReturnInfo(CallConv, ReturnType, F.getAttributes(), SplitArgs,
                MF.getDataLayout());
  return canLowerReturn(MF, CallConv, SplitArgs, F.isVarArg());
}

bool CallLowering::parametersInCSRMatch(
    const MachineRegisterInfo &MRI, const uint32_t *CallerPreservedMask,
    const SmallVectorImpl<CCValAssign> &OutLocs,
    const SmallVectorImpl<ArgInfo> &OutArgs) const {
  for (unsigned i = 0; i < OutLocs.size(); ++i) {
    const auto &ArgLoc = OutLocs[i];
    // If it's not a register, it's fine.
    if (!ArgLoc.isRegLoc())
      continue;

    MCRegister PhysReg = ArgLoc.getLocReg();
````
- **L1161 EN**: Assigns or initializes `Type *ReturnType`.
  **L1161 CN**: 对 `Type *ReturnType` 进行赋值或初始化。
- **L1162 EN**: Assigns or initializes `CallingConv::ID CallConv`.
  **L1162 CN**: 对 `CallingConv::ID CallConv` 进行赋值或初始化。
- **L1163 EN**: Separates nearby statements for readability.
  **L1163 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L1164 EN**: Executes statement `SmallVector<BaseArgInfo, 4> SplitArgs;`.
  **L1164 CN**: 执行语句 `SmallVector<BaseArgInfo, 4> SplitArgs;`。
- **L1165 EN**: Continues logic with `getReturnInfo(CallConv, ReturnType, F.getAttributes(), SplitArgs,`.
  **L1165 CN**: 继续处理逻辑：`getReturnInfo(CallConv, ReturnType, F.getAttributes(), SplitArgs,`。
- **L1166 EN**: Executes statement `MF.getDataLayout());`.
  **L1166 CN**: 执行语句 `MF.getDataLayout());`。
- **L1167 EN**: Returns `canLowerReturn(MF, CallConv, SplitArgs, F.isVarArg())` to the caller.
  **L1167 CN**: 向调用者返回 `canLowerReturn(MF, CallConv, SplitArgs, F.isVarArg())`。
- **L1168 EN**: Closes the current scope.
  **L1168 CN**: 关闭当前作用域。
- **L1169 EN**: Separates nearby statements for readability.
  **L1169 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L1170 EN**: Provides part of the signature for `parametersInCSRMatch`.
  **L1170 CN**: 给出 `parametersInCSRMatch` 的一部分签名。
- **L1171 EN**: Continues logic with `const MachineRegisterInfo &MRI, const uint32_t *CallerPreservedMask,`.
  **L1171 CN**: 继续处理逻辑：`const MachineRegisterInfo &MRI, const uint32_t *CallerPreservedMask,`。
- **L1172 EN**: Continues logic with `const SmallVectorImpl<CCValAssign> &OutLocs,`.
  **L1172 CN**: 继续处理逻辑：`const SmallVectorImpl<CCValAssign> &OutLocs,`。
- **L1173 EN**: Starts block `const SmallVectorImpl<ArgInfo> &OutArgs) const`.
  **L1173 CN**: 开始代码块 `const SmallVectorImpl<ArgInfo> &OutArgs) const`。
- **L1174 EN**: Starts a loop over a sequence or range.
  **L1174 CN**: 开始遍历序列或范围的循环。
- **L1175 EN**: Assigns or initializes `const auto &ArgLoc`.
  **L1175 CN**: 对 `const auto &ArgLoc` 进行赋值或初始化。
- **L1176 EN**: Comment documents: `If it's not a register, it's fine.`.
  **L1176 CN**: 注释说明：`If it's not a register, it's fine.`。
- **L1177 EN**: Begins a conditional branch.
  **L1177 CN**: 开始一个条件分支。
- **L1178 EN**: Skips to the next loop iteration.
  **L1178 CN**: 跳到下一次循环迭代。
- **L1179 EN**: Separates nearby statements for readability.
  **L1179 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L1180 EN**: Assigns or initializes `MCRegister PhysReg`.
  **L1180 CN**: 对 `MCRegister PhysReg` 进行赋值或初始化。

### Lines 1181-1200

````cpp

    // Only look at callee-saved registers.
    if (MachineOperand::clobbersPhysReg(CallerPreservedMask, PhysReg))
      continue;

    LLVM_DEBUG(
        dbgs()
        << "... Call has an argument passed in a callee-saved register.\n");

    // Check if it was copied from.
    const ArgInfo &OutInfo = OutArgs[i];

    if (OutInfo.Regs.size() > 1) {
      LLVM_DEBUG(
          dbgs() << "... Cannot handle arguments in multiple registers.\n");
      return false;
    }

    // Check if we copy the register, walking through copies from virtual
    // registers. Note that getDefIgnoringCopies does not ignore copies from
````
- **L1181 EN**: Separates nearby statements for readability.
  **L1181 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L1182 EN**: Comment documents: `Only look at callee-saved registers.`.
  **L1182 CN**: 注释说明：`Only look at callee-saved registers.`。
- **L1183 EN**: Begins a conditional branch.
  **L1183 CN**: 开始一个条件分支。
- **L1184 EN**: Skips to the next loop iteration.
  **L1184 CN**: 跳到下一次循环迭代。
- **L1185 EN**: Separates nearby statements for readability.
  **L1185 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L1186 EN**: Emits debug-only tracing logic.
  **L1186 CN**: 发出仅在调试时启用的跟踪逻辑。
- **L1187 EN**: Continues logic with `dbgs()`.
  **L1187 CN**: 继续处理逻辑：`dbgs()`。
- **L1188 EN**: Executes statement `<< "... Call has an argument passed in a callee-saved register.\n");`.
  **L1188 CN**: 执行语句 `<< "... Call has an argument passed in a callee-saved register.\n");`。
- **L1189 EN**: Separates nearby statements for readability.
  **L1189 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L1190 EN**: Comment documents: `Check if it was copied from.`.
  **L1190 CN**: 注释说明：`Check if it was copied from.`。
- **L1191 EN**: Assigns or initializes `const ArgInfo &OutInfo`.
  **L1191 CN**: 对 `const ArgInfo &OutInfo` 进行赋值或初始化。
- **L1192 EN**: Separates nearby statements for readability.
  **L1192 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L1193 EN**: Begins a conditional branch.
  **L1193 CN**: 开始一个条件分支。
- **L1194 EN**: Emits debug-only tracing logic.
  **L1194 CN**: 发出仅在调试时启用的跟踪逻辑。
- **L1195 EN**: Executes statement `dbgs() << "... Cannot handle arguments in multiple registers.\n");`.
  **L1195 CN**: 执行语句 `dbgs() << "... Cannot handle arguments in multiple registers.\n");`。
- **L1196 EN**: Returns `false` to the caller.
  **L1196 CN**: 向调用者返回 `false`。
- **L1197 EN**: Closes the current scope.
  **L1197 CN**: 关闭当前作用域。
- **L1198 EN**: Separates nearby statements for readability.
  **L1198 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L1199 EN**: Comment documents: `Check if we copy the register, walking through copies from virtual`.
  **L1199 CN**: 注释说明：`Check if we copy the register, walking through copies from virtual`。
- **L1200 EN**: Comment documents: `registers. Note that getDefIgnoringCopies does not ignore copies from`.
  **L1200 CN**: 注释说明：`registers. Note that getDefIgnoringCopies does not ignore copies from`。

### Lines 1201-1220

````cpp
    // physical registers.
    MachineInstr *RegDef = getDefIgnoringCopies(OutInfo.Regs[0], MRI);
    if (!RegDef || RegDef->getOpcode() != TargetOpcode::COPY) {
      LLVM_DEBUG(
          dbgs()
          << "... Parameter was not copied into a VReg, cannot tail call.\n");
      return false;
    }

    // Got a copy. Verify that it's the same as the register we want.
    Register CopyRHS = RegDef->getOperand(1).getReg();
    if (CopyRHS != PhysReg) {
      LLVM_DEBUG(dbgs() << "... Callee-saved register was not copied into "
                           "VReg, cannot tail call.\n");
      return false;
    }
  }

  return true;
}
````
- **L1201 EN**: Comment documents: `physical registers.`.
  **L1201 CN**: 注释说明：`physical registers.`。
- **L1202 EN**: Assigns or initializes `MachineInstr *RegDef`.
  **L1202 CN**: 对 `MachineInstr *RegDef` 进行赋值或初始化。
- **L1203 EN**: Begins a conditional branch.
  **L1203 CN**: 开始一个条件分支。
- **L1204 EN**: Emits debug-only tracing logic.
  **L1204 CN**: 发出仅在调试时启用的跟踪逻辑。
- **L1205 EN**: Continues logic with `dbgs()`.
  **L1205 CN**: 继续处理逻辑：`dbgs()`。
- **L1206 EN**: Executes statement `<< "... Parameter was not copied into a VReg, cannot tail call.\n");`.
  **L1206 CN**: 执行语句 `<< "... Parameter was not copied into a VReg, cannot tail call.\n");`。
- **L1207 EN**: Returns `false` to the caller.
  **L1207 CN**: 向调用者返回 `false`。
- **L1208 EN**: Closes the current scope.
  **L1208 CN**: 关闭当前作用域。
- **L1209 EN**: Separates nearby statements for readability.
  **L1209 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L1210 EN**: Comment documents: `Got a copy. Verify that it's the same as the register we want.`.
  **L1210 CN**: 注释说明：`Got a copy. Verify that it's the same as the register we want.`。
- **L1211 EN**: Assigns or initializes `Register CopyRHS`.
  **L1211 CN**: 对 `Register CopyRHS` 进行赋值或初始化。
- **L1212 EN**: Begins a conditional branch.
  **L1212 CN**: 开始一个条件分支。
- **L1213 EN**: Emits debug-only tracing logic.
  **L1213 CN**: 发出仅在调试时启用的跟踪逻辑。
- **L1214 EN**: Executes statement `"VReg, cannot tail call.\n");`.
  **L1214 CN**: 执行语句 `"VReg, cannot tail call.\n");`。
- **L1215 EN**: Returns `false` to the caller.
  **L1215 CN**: 向调用者返回 `false`。
- **L1216 EN**: Closes the current scope.
  **L1216 CN**: 关闭当前作用域。
- **L1217 EN**: Closes the current scope.
  **L1217 CN**: 关闭当前作用域。
- **L1218 EN**: Separates nearby statements for readability.
  **L1218 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L1219 EN**: Returns `true` to the caller.
  **L1219 CN**: 向调用者返回 `true`。
- **L1220 EN**: Closes the current scope.
  **L1220 CN**: 关闭当前作用域。

### Lines 1221-1240

````cpp

bool CallLowering::resultsCompatible(CallLoweringInfo &Info,
                                     MachineFunction &MF,
                                     SmallVectorImpl<ArgInfo> &InArgs,
                                     ValueAssigner &CalleeAssigner,
                                     ValueAssigner &CallerAssigner) const {
  const Function &F = MF.getFunction();
  CallingConv::ID CalleeCC = Info.CallConv;
  CallingConv::ID CallerCC = F.getCallingConv();

  if (CallerCC == CalleeCC)
    return true;

  SmallVector<CCValAssign, 16> ArgLocs1;
  CCState CCInfo1(CalleeCC, Info.IsVarArg, MF, ArgLocs1, F.getContext());
  if (!determineAssignments(CalleeAssigner, InArgs, CCInfo1))
    return false;

  SmallVector<CCValAssign, 16> ArgLocs2;
  CCState CCInfo2(CallerCC, F.isVarArg(), MF, ArgLocs2, F.getContext());
````
- **L1221 EN**: Separates nearby statements for readability.
  **L1221 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L1222 EN**: Provides part of the signature for `resultsCompatible`.
  **L1222 CN**: 给出 `resultsCompatible` 的一部分签名。
- **L1223 EN**: Continues logic with `MachineFunction &MF,`.
  **L1223 CN**: 继续处理逻辑：`MachineFunction &MF,`。
- **L1224 EN**: Continues logic with `SmallVectorImpl<ArgInfo> &InArgs,`.
  **L1224 CN**: 继续处理逻辑：`SmallVectorImpl<ArgInfo> &InArgs,`。
- **L1225 EN**: Continues logic with `ValueAssigner &CalleeAssigner,`.
  **L1225 CN**: 继续处理逻辑：`ValueAssigner &CalleeAssigner,`。
- **L1226 EN**: Starts block `ValueAssigner &CallerAssigner) const`.
  **L1226 CN**: 开始代码块 `ValueAssigner &CallerAssigner) const`。
- **L1227 EN**: Assigns or initializes `const Function &F`.
  **L1227 CN**: 对 `const Function &F` 进行赋值或初始化。
- **L1228 EN**: Assigns or initializes `CallingConv::ID CalleeCC`.
  **L1228 CN**: 对 `CallingConv::ID CalleeCC` 进行赋值或初始化。
- **L1229 EN**: Assigns or initializes `CallingConv::ID CallerCC`.
  **L1229 CN**: 对 `CallingConv::ID CallerCC` 进行赋值或初始化。
- **L1230 EN**: Separates nearby statements for readability.
  **L1230 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L1231 EN**: Begins a conditional branch.
  **L1231 CN**: 开始一个条件分支。
- **L1232 EN**: Returns `true` to the caller.
  **L1232 CN**: 向调用者返回 `true`。
- **L1233 EN**: Separates nearby statements for readability.
  **L1233 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L1234 EN**: Executes statement `SmallVector<CCValAssign, 16> ArgLocs1;`.
  **L1234 CN**: 执行语句 `SmallVector<CCValAssign, 16> ArgLocs1;`。
- **L1235 EN**: Declares function or method `CCInfo1`.
  **L1235 CN**: 声明函数或方法 `CCInfo1`。
- **L1236 EN**: Begins a conditional branch.
  **L1236 CN**: 开始一个条件分支。
- **L1237 EN**: Returns `false` to the caller.
  **L1237 CN**: 向调用者返回 `false`。
- **L1238 EN**: Separates nearby statements for readability.
  **L1238 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L1239 EN**: Executes statement `SmallVector<CCValAssign, 16> ArgLocs2;`.
  **L1239 CN**: 执行语句 `SmallVector<CCValAssign, 16> ArgLocs2;`。
- **L1240 EN**: Declares function or method `CCInfo2`.
  **L1240 CN**: 声明函数或方法 `CCInfo2`。

### Lines 1241-1260

````cpp
  if (!determineAssignments(CallerAssigner, InArgs, CCInfo2))
    return false;

  // We need the argument locations to match up exactly. If there's more in
  // one than the other, then we are done.
  if (ArgLocs1.size() != ArgLocs2.size())
    return false;

  // Make sure that each location is passed in exactly the same way.
  for (unsigned i = 0, e = ArgLocs1.size(); i < e; ++i) {
    const CCValAssign &Loc1 = ArgLocs1[i];
    const CCValAssign &Loc2 = ArgLocs2[i];

    // We need both of them to be the same. So if one is a register and one
    // isn't, we're done.
    if (Loc1.isRegLoc() != Loc2.isRegLoc())
      return false;

    if (Loc1.isRegLoc()) {
      // If they don't have the same register location, we're done.
````
- **L1241 EN**: Begins a conditional branch.
  **L1241 CN**: 开始一个条件分支。
- **L1242 EN**: Returns `false` to the caller.
  **L1242 CN**: 向调用者返回 `false`。
- **L1243 EN**: Separates nearby statements for readability.
  **L1243 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L1244 EN**: Comment documents: `We need the argument locations to match up exactly. If there's more in`.
  **L1244 CN**: 注释说明：`We need the argument locations to match up exactly. If there's more in`。
- **L1245 EN**: Comment documents: `one than the other, then we are done.`.
  **L1245 CN**: 注释说明：`one than the other, then we are done.`。
- **L1246 EN**: Begins a conditional branch.
  **L1246 CN**: 开始一个条件分支。
- **L1247 EN**: Returns `false` to the caller.
  **L1247 CN**: 向调用者返回 `false`。
- **L1248 EN**: Separates nearby statements for readability.
  **L1248 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L1249 EN**: Comment documents: `Make sure that each location is passed in exactly the same way.`.
  **L1249 CN**: 注释说明：`Make sure that each location is passed in exactly the same way.`。
- **L1250 EN**: Starts a loop over a sequence or range.
  **L1250 CN**: 开始遍历序列或范围的循环。
- **L1251 EN**: Assigns or initializes `const CCValAssign &Loc1`.
  **L1251 CN**: 对 `const CCValAssign &Loc1` 进行赋值或初始化。
- **L1252 EN**: Assigns or initializes `const CCValAssign &Loc2`.
  **L1252 CN**: 对 `const CCValAssign &Loc2` 进行赋值或初始化。
- **L1253 EN**: Separates nearby statements for readability.
  **L1253 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L1254 EN**: Comment documents: `We need both of them to be the same. So if one is a register and one`.
  **L1254 CN**: 注释说明：`We need both of them to be the same. So if one is a register and one`。
- **L1255 EN**: Comment documents: `isn't, we're done.`.
  **L1255 CN**: 注释说明：`isn't, we're done.`。
- **L1256 EN**: Begins a conditional branch.
  **L1256 CN**: 开始一个条件分支。
- **L1257 EN**: Returns `false` to the caller.
  **L1257 CN**: 向调用者返回 `false`。
- **L1258 EN**: Separates nearby statements for readability.
  **L1258 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L1259 EN**: Begins a conditional branch.
  **L1259 CN**: 开始一个条件分支。
- **L1260 EN**: Comment documents: `If they don't have the same register location, we're done.`.
  **L1260 CN**: 注释说明：`If they don't have the same register location, we're done.`。

### Lines 1261-1280

````cpp
      if (Loc1.getLocReg() != Loc2.getLocReg())
        return false;

      // They matched, so we can move to the next ArgLoc.
      continue;
    }

    // Loc1 wasn't a RegLoc, so they both must be MemLocs. Check if they match.
    if (Loc1.getLocMemOffset() != Loc2.getLocMemOffset())
      return false;
  }

  return true;
}

LLT CallLowering::ValueHandler::getStackValueStoreType(
    const DataLayout &DL, const CCValAssign &VA, ISD::ArgFlagsTy Flags) const {
  const MVT ValVT = VA.getValVT();
  if (ValVT != MVT::iPTR) {
    LLT ValTy(ValVT);
````
- **L1261 EN**: Begins a conditional branch.
  **L1261 CN**: 开始一个条件分支。
- **L1262 EN**: Returns `false` to the caller.
  **L1262 CN**: 向调用者返回 `false`。
- **L1263 EN**: Separates nearby statements for readability.
  **L1263 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L1264 EN**: Comment documents: `They matched, so we can move to the next ArgLoc.`.
  **L1264 CN**: 注释说明：`They matched, so we can move to the next ArgLoc.`。
- **L1265 EN**: Skips to the next loop iteration.
  **L1265 CN**: 跳到下一次循环迭代。
- **L1266 EN**: Closes the current scope.
  **L1266 CN**: 关闭当前作用域。
- **L1267 EN**: Separates nearby statements for readability.
  **L1267 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L1268 EN**: Comment documents: `Loc1 wasn't a RegLoc, so they both must be MemLocs. Check if they match.`.
  **L1268 CN**: 注释说明：`Loc1 wasn't a RegLoc, so they both must be MemLocs. Check if they match.`。
- **L1269 EN**: Begins a conditional branch.
  **L1269 CN**: 开始一个条件分支。
- **L1270 EN**: Returns `false` to the caller.
  **L1270 CN**: 向调用者返回 `false`。
- **L1271 EN**: Closes the current scope.
  **L1271 CN**: 关闭当前作用域。
- **L1272 EN**: Separates nearby statements for readability.
  **L1272 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L1273 EN**: Returns `true` to the caller.
  **L1273 CN**: 向调用者返回 `true`。
- **L1274 EN**: Closes the current scope.
  **L1274 CN**: 关闭当前作用域。
- **L1275 EN**: Separates nearby statements for readability.
  **L1275 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L1276 EN**: Provides part of the signature for `getStackValueStoreType`.
  **L1276 CN**: 给出 `getStackValueStoreType` 的一部分签名。
- **L1277 EN**: Starts block `const DataLayout &DL, const CCValAssign &VA, ISD::ArgFlagsTy Flags) cons…`.
  **L1277 CN**: 开始代码块 `const DataLayout &DL, const CCValAssign &VA, ISD::ArgFlagsTy Flags) cons…`。
- **L1278 EN**: Assigns or initializes `const MVT ValVT`.
  **L1278 CN**: 对 `const MVT ValVT` 进行赋值或初始化。
- **L1279 EN**: Begins a conditional branch.
  **L1279 CN**: 开始一个条件分支。
- **L1280 EN**: Declares function or method `ValTy`.
  **L1280 CN**: 声明函数或方法 `ValTy`。

### Lines 1281-1300

````cpp

    // We lost the pointeriness going through CCValAssign, so try to restore it
    // based on the flags.
    if (Flags.isPointer()) {
      LLT PtrTy = LLT::pointer(Flags.getPointerAddrSpace(),
                               ValTy.getScalarSizeInBits());
      if (ValVT.isVector() && ValVT.getVectorNumElements() != 1)
        return LLT::vector(ValTy.getElementCount(), PtrTy);
      return PtrTy;
    }

    return ValTy;
  }

  unsigned AddrSpace = Flags.getPointerAddrSpace();
  return LLT::pointer(AddrSpace, DL.getPointerSize(AddrSpace));
}

void CallLowering::ValueHandler::copyArgumentMemory(
    const ArgInfo &Arg, Register DstPtr, Register SrcPtr,
````
- **L1281 EN**: Separates nearby statements for readability.
  **L1281 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L1282 EN**: Comment documents: `We lost the pointeriness going through CCValAssign, so try to restore it`.
  **L1282 CN**: 注释说明：`We lost the pointeriness going through CCValAssign, so try to restore it`。
- **L1283 EN**: Comment documents: `based on the flags.`.
  **L1283 CN**: 注释说明：`based on the flags.`。
- **L1284 EN**: Begins a conditional branch.
  **L1284 CN**: 开始一个条件分支。
- **L1285 EN**: Provides part of the signature for `pointer`.
  **L1285 CN**: 给出 `pointer` 的一部分签名。
- **L1286 EN**: Executes statement `ValTy.getScalarSizeInBits());`.
  **L1286 CN**: 执行语句 `ValTy.getScalarSizeInBits());`。
- **L1287 EN**: Begins a conditional branch.
  **L1287 CN**: 开始一个条件分支。
- **L1288 EN**: Returns `LLT::vector(ValTy.getElementCount(), PtrTy)` to the caller.
  **L1288 CN**: 向调用者返回 `LLT::vector(ValTy.getElementCount(), PtrTy)`。
- **L1289 EN**: Returns `PtrTy` to the caller.
  **L1289 CN**: 向调用者返回 `PtrTy`。
- **L1290 EN**: Closes the current scope.
  **L1290 CN**: 关闭当前作用域。
- **L1291 EN**: Separates nearby statements for readability.
  **L1291 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L1292 EN**: Returns `ValTy` to the caller.
  **L1292 CN**: 向调用者返回 `ValTy`。
- **L1293 EN**: Closes the current scope.
  **L1293 CN**: 关闭当前作用域。
- **L1294 EN**: Separates nearby statements for readability.
  **L1294 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L1295 EN**: Assigns or initializes `unsigned AddrSpace`.
  **L1295 CN**: 对 `unsigned AddrSpace` 进行赋值或初始化。
- **L1296 EN**: Returns `LLT::pointer(AddrSpace, DL.getPointerSize(AddrSpace))` to the caller.
  **L1296 CN**: 向调用者返回 `LLT::pointer(AddrSpace, DL.getPointerSize(AddrSpace))`。
- **L1297 EN**: Closes the current scope.
  **L1297 CN**: 关闭当前作用域。
- **L1298 EN**: Separates nearby statements for readability.
  **L1298 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L1299 EN**: Provides part of the signature for `copyArgumentMemory`.
  **L1299 CN**: 给出 `copyArgumentMemory` 的一部分签名。
- **L1300 EN**: Continues logic with `const ArgInfo &Arg, Register DstPtr, Register SrcPtr,`.
  **L1300 CN**: 继续处理逻辑：`const ArgInfo &Arg, Register DstPtr, Register SrcPtr,`。

### Lines 1301-1320

````cpp
    const MachinePointerInfo &DstPtrInfo, Align DstAlign,
    const MachinePointerInfo &SrcPtrInfo, Align SrcAlign, uint64_t MemSize,
    CCValAssign &VA) const {
  MachineFunction &MF = MIRBuilder.getMF();
  MachineMemOperand *SrcMMO = MF.getMachineMemOperand(
      SrcPtrInfo,
      MachineMemOperand::MOLoad | MachineMemOperand::MODereferenceable, MemSize,
      SrcAlign);

  MachineMemOperand *DstMMO = MF.getMachineMemOperand(
      DstPtrInfo,
      MachineMemOperand::MOStore | MachineMemOperand::MODereferenceable,
      MemSize, DstAlign);

  const LLT PtrTy = MRI.getType(DstPtr);
  const LLT SizeTy = LLT::integer(PtrTy.getSizeInBits());

  auto SizeConst = MIRBuilder.buildConstant(SizeTy, MemSize);
  MIRBuilder.buildMemCpy(DstPtr, SrcPtr, SizeConst, *DstMMO, *SrcMMO);
}
````
- **L1301 EN**: Continues logic with `const MachinePointerInfo &DstPtrInfo, Align DstAlign,`.
  **L1301 CN**: 继续处理逻辑：`const MachinePointerInfo &DstPtrInfo, Align DstAlign,`。
- **L1302 EN**: Continues logic with `const MachinePointerInfo &SrcPtrInfo, Align SrcAlign, uint64_t MemSize,`.
  **L1302 CN**: 继续处理逻辑：`const MachinePointerInfo &SrcPtrInfo, Align SrcAlign, uint64_t MemSize,`。
- **L1303 EN**: Starts block `CCValAssign &VA) const`.
  **L1303 CN**: 开始代码块 `CCValAssign &VA) const`。
- **L1304 EN**: Assigns or initializes `MachineFunction &MF`.
  **L1304 CN**: 对 `MachineFunction &MF` 进行赋值或初始化。
- **L1305 EN**: Continues logic with `MachineMemOperand *SrcMMO = MF.getMachineMemOperand(`.
  **L1305 CN**: 继续处理逻辑：`MachineMemOperand *SrcMMO = MF.getMachineMemOperand(`。
- **L1306 EN**: Continues logic with `SrcPtrInfo,`.
  **L1306 CN**: 继续处理逻辑：`SrcPtrInfo,`。
- **L1307 EN**: Continues logic with `MachineMemOperand::MOLoad | MachineMemOperand::MODereferenceable, MemSiz…`.
  **L1307 CN**: 继续处理逻辑：`MachineMemOperand::MOLoad | MachineMemOperand::MODereferenceable, MemSiz…`。
- **L1308 EN**: Executes statement `SrcAlign);`.
  **L1308 CN**: 执行语句 `SrcAlign);`。
- **L1309 EN**: Separates nearby statements for readability.
  **L1309 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L1310 EN**: Continues logic with `MachineMemOperand *DstMMO = MF.getMachineMemOperand(`.
  **L1310 CN**: 继续处理逻辑：`MachineMemOperand *DstMMO = MF.getMachineMemOperand(`。
- **L1311 EN**: Continues logic with `DstPtrInfo,`.
  **L1311 CN**: 继续处理逻辑：`DstPtrInfo,`。
- **L1312 EN**: Continues logic with `MachineMemOperand::MOStore | MachineMemOperand::MODereferenceable,`.
  **L1312 CN**: 继续处理逻辑：`MachineMemOperand::MOStore | MachineMemOperand::MODereferenceable,`。
- **L1313 EN**: Executes statement `MemSize, DstAlign);`.
  **L1313 CN**: 执行语句 `MemSize, DstAlign);`。
- **L1314 EN**: Separates nearby statements for readability.
  **L1314 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L1315 EN**: Assigns or initializes `const LLT PtrTy`.
  **L1315 CN**: 对 `const LLT PtrTy` 进行赋值或初始化。
- **L1316 EN**: Declares function or method `integer`.
  **L1316 CN**: 声明函数或方法 `integer`。
- **L1317 EN**: Separates nearby statements for readability.
  **L1317 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L1318 EN**: Assigns or initializes `auto SizeConst`.
  **L1318 CN**: 对 `auto SizeConst` 进行赋值或初始化。
- **L1319 EN**: Executes statement `MIRBuilder.buildMemCpy(DstPtr, SrcPtr, SizeConst, *DstMMO, *SrcMMO);`.
  **L1319 CN**: 执行语句 `MIRBuilder.buildMemCpy(DstPtr, SrcPtr, SizeConst, *DstMMO, *SrcMMO);`。
- **L1320 EN**: Closes the current scope.
  **L1320 CN**: 关闭当前作用域。

### Lines 1321-1340

````cpp

Register CallLowering::ValueHandler::extendRegister(Register ValReg,
                                                    const CCValAssign &VA,
                                                    unsigned MaxSizeBits) {
  LLT LocTy{VA.getLocVT()};
  LLT ValTy{VA.getValVT()};

  if (LocTy.getSizeInBits() == ValTy.getSizeInBits())
    return ValReg;

  if (LocTy.isScalar() && MaxSizeBits && MaxSizeBits < LocTy.getSizeInBits()) {
    if (MaxSizeBits <= ValTy.getSizeInBits())
      return ValReg;
    LocTy = LLT::scalar(MaxSizeBits);
  }

  const LLT ValRegTy = MRI.getType(ValReg);
  if (ValRegTy.isPointer()) {
    // The x32 ABI wants to zero extend 32-bit pointers to 64-bit registers, so
    // we have to cast to do the extension.
````
- **L1321 EN**: Separates nearby statements for readability.
  **L1321 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L1322 EN**: Provides part of the signature for `extendRegister`.
  **L1322 CN**: 给出 `extendRegister` 的一部分签名。
- **L1323 EN**: Continues logic with `const CCValAssign &VA,`.
  **L1323 CN**: 继续处理逻辑：`const CCValAssign &VA,`。
- **L1324 EN**: Starts block `unsigned MaxSizeBits)`.
  **L1324 CN**: 开始代码块 `unsigned MaxSizeBits)`。
- **L1325 EN**: Executes statement `LLT LocTy{VA.getLocVT()};`.
  **L1325 CN**: 执行语句 `LLT LocTy{VA.getLocVT()};`。
- **L1326 EN**: Executes statement `LLT ValTy{VA.getValVT()};`.
  **L1326 CN**: 执行语句 `LLT ValTy{VA.getValVT()};`。
- **L1327 EN**: Separates nearby statements for readability.
  **L1327 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L1328 EN**: Begins a conditional branch.
  **L1328 CN**: 开始一个条件分支。
- **L1329 EN**: Returns `ValReg` to the caller.
  **L1329 CN**: 向调用者返回 `ValReg`。
- **L1330 EN**: Separates nearby statements for readability.
  **L1330 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L1331 EN**: Begins a conditional branch.
  **L1331 CN**: 开始一个条件分支。
- **L1332 EN**: Begins a conditional branch.
  **L1332 CN**: 开始一个条件分支。
- **L1333 EN**: Returns `ValReg` to the caller.
  **L1333 CN**: 向调用者返回 `ValReg`。
- **L1334 EN**: Declares function or method `scalar`.
  **L1334 CN**: 声明函数或方法 `scalar`。
- **L1335 EN**: Closes the current scope.
  **L1335 CN**: 关闭当前作用域。
- **L1336 EN**: Separates nearby statements for readability.
  **L1336 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L1337 EN**: Assigns or initializes `const LLT ValRegTy`.
  **L1337 CN**: 对 `const LLT ValRegTy` 进行赋值或初始化。
- **L1338 EN**: Begins a conditional branch.
  **L1338 CN**: 开始一个条件分支。
- **L1339 EN**: Comment documents: `The x32 ABI wants to zero extend 32-bit pointers to 64-bit registers, so`.
  **L1339 CN**: 注释说明：`The x32 ABI wants to zero extend 32-bit pointers to 64-bit registers, so`。
- **L1340 EN**: Comment documents: `we have to cast to do the extension.`.
  **L1340 CN**: 注释说明：`we have to cast to do the extension.`。

### Lines 1341-1360

````cpp
    LLT IntPtrTy = LLT::scalar(ValRegTy.getSizeInBits());
    ValReg = MIRBuilder.buildPtrToInt(IntPtrTy, ValReg).getReg(0);
  }

  switch (VA.getLocInfo()) {
  default:
    break;
  case CCValAssign::Full:
  case CCValAssign::BCvt:
    // FIXME: bitconverting between vector types may or may not be a
    // nop in big-endian situations.
    return ValReg;
  case CCValAssign::AExt: {
    auto MIB = MIRBuilder.buildAnyExt(LocTy, ValReg);
    return MIB.getReg(0);
  }
  case CCValAssign::SExt: {
    Register NewReg = MRI.createGenericVirtualRegister(LocTy);
    MIRBuilder.buildSExt(NewReg, ValReg);
    return NewReg;
````
- **L1341 EN**: Declares function or method `scalar`.
  **L1341 CN**: 声明函数或方法 `scalar`。
- **L1342 EN**: Assigns or initializes `ValReg`.
  **L1342 CN**: 对 `ValReg` 进行赋值或初始化。
- **L1343 EN**: Closes the current scope.
  **L1343 CN**: 关闭当前作用域。
- **L1344 EN**: Separates nearby statements for readability.
  **L1344 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L1345 EN**: Starts a multi-way branch.
  **L1345 CN**: 开始一个多路分支。
- **L1346 EN**: Handles the default switch case.
  **L1346 CN**: 处理 switch 的默认分支。
- **L1347 EN**: Breaks out of the current control-flow construct.
  **L1347 CN**: 跳出当前控制流结构。
- **L1348 EN**: Handles one switch case.
  **L1348 CN**: 处理一个 switch 分支。
- **L1349 EN**: Handles one switch case.
  **L1349 CN**: 处理一个 switch 分支。
- **L1350 EN**: Comment documents: `FIXME: bitconverting between vector types may or may not be a`.
  **L1350 CN**: 注释说明：`FIXME: bitconverting between vector types may or may not be a`。
- **L1351 EN**: Comment documents: `nop in big-endian situations.`.
  **L1351 CN**: 注释说明：`nop in big-endian situations.`。
- **L1352 EN**: Returns `ValReg` to the caller.
  **L1352 CN**: 向调用者返回 `ValReg`。
- **L1353 EN**: Handles one switch case.
  **L1353 CN**: 处理一个 switch 分支。
- **L1354 EN**: Assigns or initializes `auto MIB`.
  **L1354 CN**: 对 `auto MIB` 进行赋值或初始化。
- **L1355 EN**: Returns `MIB.getReg(0)` to the caller.
  **L1355 CN**: 向调用者返回 `MIB.getReg(0)`。
- **L1356 EN**: Closes the current scope.
  **L1356 CN**: 关闭当前作用域。
- **L1357 EN**: Handles one switch case.
  **L1357 CN**: 处理一个 switch 分支。
- **L1358 EN**: Assigns or initializes `Register NewReg`.
  **L1358 CN**: 对 `Register NewReg` 进行赋值或初始化。
- **L1359 EN**: Executes statement `MIRBuilder.buildSExt(NewReg, ValReg);`.
  **L1359 CN**: 执行语句 `MIRBuilder.buildSExt(NewReg, ValReg);`。
- **L1360 EN**: Returns `NewReg` to the caller.
  **L1360 CN**: 向调用者返回 `NewReg`。

### Lines 1361-1380

````cpp
  }
  case CCValAssign::ZExt: {
    Register NewReg = MRI.createGenericVirtualRegister(LocTy);
    MIRBuilder.buildZExt(NewReg, ValReg);
    return NewReg;
  }
  }
  llvm_unreachable("unable to extend register");
}

void CallLowering::ValueAssigner::anchor() {}

Register CallLowering::IncomingValueHandler::buildExtensionHint(
    const CCValAssign &VA, Register SrcReg, LLT NarrowTy) {
  switch (VA.getLocInfo()) {
  case CCValAssign::LocInfo::ZExt: {
    return MIRBuilder
        .buildAssertZExt(MRI.cloneVirtualRegister(SrcReg), SrcReg,
                         NarrowTy.getScalarSizeInBits())
        .getReg(0);
````
- **L1361 EN**: Closes the current scope.
  **L1361 CN**: 关闭当前作用域。
- **L1362 EN**: Handles one switch case.
  **L1362 CN**: 处理一个 switch 分支。
- **L1363 EN**: Assigns or initializes `Register NewReg`.
  **L1363 CN**: 对 `Register NewReg` 进行赋值或初始化。
- **L1364 EN**: Executes statement `MIRBuilder.buildZExt(NewReg, ValReg);`.
  **L1364 CN**: 执行语句 `MIRBuilder.buildZExt(NewReg, ValReg);`。
- **L1365 EN**: Returns `NewReg` to the caller.
  **L1365 CN**: 向调用者返回 `NewReg`。
- **L1366 EN**: Closes the current scope.
  **L1366 CN**: 关闭当前作用域。
- **L1367 EN**: Closes the current scope.
  **L1367 CN**: 关闭当前作用域。
- **L1368 EN**: Executes statement `llvm_unreachable("unable to extend register");`.
  **L1368 CN**: 执行语句 `llvm_unreachable("unable to extend register");`。
- **L1369 EN**: Closes the current scope.
  **L1369 CN**: 关闭当前作用域。
- **L1370 EN**: Separates nearby statements for readability.
  **L1370 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L1371 EN**: Provides part of the signature for `anchor`.
  **L1371 CN**: 给出 `anchor` 的一部分签名。
- **L1372 EN**: Separates nearby statements for readability.
  **L1372 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L1373 EN**: Provides part of the signature for `buildExtensionHint`.
  **L1373 CN**: 给出 `buildExtensionHint` 的一部分签名。
- **L1374 EN**: Starts block `const CCValAssign &VA, Register SrcReg, LLT NarrowTy)`.
  **L1374 CN**: 开始代码块 `const CCValAssign &VA, Register SrcReg, LLT NarrowTy)`。
- **L1375 EN**: Starts a multi-way branch.
  **L1375 CN**: 开始一个多路分支。
- **L1376 EN**: Handles one switch case.
  **L1376 CN**: 处理一个 switch 分支。
- **L1377 EN**: Returns `MIRBuilder` to the caller.
  **L1377 CN**: 向调用者返回 `MIRBuilder`。
- **L1378 EN**: Continues logic with `.buildAssertZExt(MRI.cloneVirtualRegister(SrcReg), SrcReg,`.
  **L1378 CN**: 继续处理逻辑：`.buildAssertZExt(MRI.cloneVirtualRegister(SrcReg), SrcReg,`。
- **L1379 EN**: Continues logic with `NarrowTy.getScalarSizeInBits())`.
  **L1379 CN**: 继续处理逻辑：`NarrowTy.getScalarSizeInBits())`。
- **L1380 EN**: Executes statement `.getReg(0);`.
  **L1380 CN**: 执行语句 `.getReg(0);`。

### Lines 1381-1400

````cpp
  }
  case CCValAssign::LocInfo::SExt: {
    return MIRBuilder
        .buildAssertSExt(MRI.cloneVirtualRegister(SrcReg), SrcReg,
                         NarrowTy.getScalarSizeInBits())
        .getReg(0);
    break;
  }
  default:
    return SrcReg;
  }
}

/// Check if we can use a basic COPY instruction between the two types.
///
/// We're currently building on top of the infrastructure using MVT, which loses
/// pointer information in the CCValAssign. We accept copies from physical
/// registers that have been reported as integers if it's to an equivalent sized
/// pointer LLT.
static bool isCopyCompatibleType(LLT SrcTy, LLT DstTy) {
````
- **L1381 EN**: Closes the current scope.
  **L1381 CN**: 关闭当前作用域。
- **L1382 EN**: Handles one switch case.
  **L1382 CN**: 处理一个 switch 分支。
- **L1383 EN**: Returns `MIRBuilder` to the caller.
  **L1383 CN**: 向调用者返回 `MIRBuilder`。
- **L1384 EN**: Continues logic with `.buildAssertSExt(MRI.cloneVirtualRegister(SrcReg), SrcReg,`.
  **L1384 CN**: 继续处理逻辑：`.buildAssertSExt(MRI.cloneVirtualRegister(SrcReg), SrcReg,`。
- **L1385 EN**: Continues logic with `NarrowTy.getScalarSizeInBits())`.
  **L1385 CN**: 继续处理逻辑：`NarrowTy.getScalarSizeInBits())`。
- **L1386 EN**: Executes statement `.getReg(0);`.
  **L1386 CN**: 执行语句 `.getReg(0);`。
- **L1387 EN**: Breaks out of the current control-flow construct.
  **L1387 CN**: 跳出当前控制流结构。
- **L1388 EN**: Closes the current scope.
  **L1388 CN**: 关闭当前作用域。
- **L1389 EN**: Handles the default switch case.
  **L1389 CN**: 处理 switch 的默认分支。
- **L1390 EN**: Returns `SrcReg` to the caller.
  **L1390 CN**: 向调用者返回 `SrcReg`。
- **L1391 EN**: Closes the current scope.
  **L1391 CN**: 关闭当前作用域。
- **L1392 EN**: Closes the current scope.
  **L1392 CN**: 关闭当前作用域。
- **L1393 EN**: Separates nearby statements for readability.
  **L1393 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L1394 EN**: Comment documents: `Check if we can use a basic COPY instruction between the two types.`.
  **L1394 CN**: 注释说明：`Check if we can use a basic COPY instruction between the two types.`。
- **L1395 EN**: Continues the surrounding comment block.
  **L1395 CN**: 延续周围的注释块。
- **L1396 EN**: Comment documents: `We're currently building on top of the infrastructure using MVT, which l…`.
  **L1396 CN**: 注释说明：`We're currently building on top of the infrastructure using MVT, which l…`。
- **L1397 EN**: Comment documents: `pointer information in the CCValAssign. We accept copies from physical`.
  **L1397 CN**: 注释说明：`pointer information in the CCValAssign. We accept copies from physical`。
- **L1398 EN**: Comment documents: `registers that have been reported as integers if it's to an equivalent s…`.
  **L1398 CN**: 注释说明：`registers that have been reported as integers if it's to an equivalent s…`。
- **L1399 EN**: Comment documents: `pointer LLT.`.
  **L1399 CN**: 注释说明：`pointer LLT.`。
- **L1400 EN**: Begins the definition of `isCopyCompatibleType`.
  **L1400 CN**: 开始定义 `isCopyCompatibleType`。

### Lines 1401-1420

````cpp
  if (SrcTy == DstTy)
    return true;

  if (SrcTy.getSizeInBits() != DstTy.getSizeInBits())
    return false;

  SrcTy = SrcTy.getScalarType();
  DstTy = DstTy.getScalarType();

  return (SrcTy.isPointer() && DstTy.isScalar()) ||
         (DstTy.isPointer() && SrcTy.isScalar());
}

void CallLowering::IncomingValueHandler::assignValueToReg(
    Register ValVReg, Register PhysReg, const CCValAssign &VA,
    ISD::ArgFlagsTy Flags) {
  const MVT LocVT = VA.getLocVT();
  const LLT LocTy = getLLTForMVT(LocVT);
  const LLT RegTy = MRI.getType(ValVReg);

````
- **L1401 EN**: Begins a conditional branch.
  **L1401 CN**: 开始一个条件分支。
- **L1402 EN**: Returns `true` to the caller.
  **L1402 CN**: 向调用者返回 `true`。
- **L1403 EN**: Separates nearby statements for readability.
  **L1403 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L1404 EN**: Begins a conditional branch.
  **L1404 CN**: 开始一个条件分支。
- **L1405 EN**: Returns `false` to the caller.
  **L1405 CN**: 向调用者返回 `false`。
- **L1406 EN**: Separates nearby statements for readability.
  **L1406 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L1407 EN**: Assigns or initializes `SrcTy`.
  **L1407 CN**: 对 `SrcTy` 进行赋值或初始化。
- **L1408 EN**: Assigns or initializes `DstTy`.
  **L1408 CN**: 对 `DstTy` 进行赋值或初始化。
- **L1409 EN**: Separates nearby statements for readability.
  **L1409 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L1410 EN**: Returns `(SrcTy.isPointer() && DstTy.isScalar()) ||` to the caller.
  **L1410 CN**: 向调用者返回 `(SrcTy.isPointer() && DstTy.isScalar()) ||`。
- **L1411 EN**: Executes statement `(DstTy.isPointer() && SrcTy.isScalar());`.
  **L1411 CN**: 执行语句 `(DstTy.isPointer() && SrcTy.isScalar());`。
- **L1412 EN**: Closes the current scope.
  **L1412 CN**: 关闭当前作用域。
- **L1413 EN**: Separates nearby statements for readability.
  **L1413 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L1414 EN**: Provides part of the signature for `assignValueToReg`.
  **L1414 CN**: 给出 `assignValueToReg` 的一部分签名。
- **L1415 EN**: Continues logic with `Register ValVReg, Register PhysReg, const CCValAssign &VA,`.
  **L1415 CN**: 继续处理逻辑：`Register ValVReg, Register PhysReg, const CCValAssign &VA,`。
- **L1416 EN**: Starts block `ISD::ArgFlagsTy Flags)`.
  **L1416 CN**: 开始代码块 `ISD::ArgFlagsTy Flags)`。
- **L1417 EN**: Assigns or initializes `const MVT LocVT`.
  **L1417 CN**: 对 `const MVT LocVT` 进行赋值或初始化。
- **L1418 EN**: Assigns or initializes `const LLT LocTy`.
  **L1418 CN**: 对 `const LLT LocTy` 进行赋值或初始化。
- **L1419 EN**: Assigns or initializes `const LLT RegTy`.
  **L1419 CN**: 对 `const LLT RegTy` 进行赋值或初始化。
- **L1420 EN**: Separates nearby statements for readability.
  **L1420 CN**: 空行，用于分隔相邻语句并提升可读性。

### Lines 1421-1429

````cpp
  if (isCopyCompatibleType(RegTy, LocTy)) {
    MIRBuilder.buildCopy(ValVReg, PhysReg);
    return;
  }

  auto Copy = MIRBuilder.buildCopy(LocTy, PhysReg);
  auto Hint = buildExtensionHint(VA, Copy.getReg(0), RegTy);
  MIRBuilder.buildTrunc(ValVReg, Hint);
}
````
- **L1421 EN**: Begins a conditional branch.
  **L1421 CN**: 开始一个条件分支。
- **L1422 EN**: Executes statement `MIRBuilder.buildCopy(ValVReg, PhysReg);`.
  **L1422 CN**: 执行语句 `MIRBuilder.buildCopy(ValVReg, PhysReg);`。
- **L1423 EN**: Returns control to the caller.
  **L1423 CN**: 将控制流返回给调用者。
- **L1424 EN**: Closes the current scope.
  **L1424 CN**: 关闭当前作用域。
- **L1425 EN**: Separates nearby statements for readability.
  **L1425 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L1426 EN**: Assigns or initializes `auto Copy`.
  **L1426 CN**: 对 `auto Copy` 进行赋值或初始化。
- **L1427 EN**: Assigns or initializes `auto Hint`.
  **L1427 CN**: 对 `auto Hint` 进行赋值或初始化。
- **L1428 EN**: Executes statement `MIRBuilder.buildTrunc(ValVReg, Hint);`.
  **L1428 CN**: 执行语句 `MIRBuilder.buildTrunc(ValVReg, Hint);`。
- **L1429 EN**: Closes the current scope.
  **L1429 CN**: 关闭当前作用域。

## Key Concepts / 关键概念
- **Register management** / **寄存器管理**
- **Machine function processing** / **MachineFunction 处理**
- **Machine instruction manipulation** / **机器指令操作**
- **GlobalISel pipeline** / **GlobalISel 流水线**
- **Spill and reload handling** / **溢出与重载处理**
- **Stack frame management** / **栈帧管理**
- **Control-flow updates** / **控制流更新**
- **Debug information handling** / **调试信息处理**

## Dependencies / 依赖关系
- **LLVM headers / LLVM 头文件**: `llvm/CodeGen/GlobalISel/CallLowering.h`, `llvm/CodeGen/Analysis.h`, `llvm/CodeGen/CallingConvLower.h`, `llvm/CodeGen/GlobalISel/MachineIRBuilder.h`, `llvm/CodeGen/GlobalISel/Utils.h`, `llvm/CodeGen/MachineFrameInfo.h`, `llvm/CodeGen/MachineOperand.h`, `llvm/CodeGen/MachineRegisterInfo.h`, `llvm/CodeGen/TargetLowering.h`, `llvm/IR/DataLayout.h`, `llvm/IR/LLVMContext.h`, `llvm/IR/Module.h`, `llvm/Target/TargetMachine.h`
- **Primary dependency domains / 主要依赖域**: CodeGen internals, LLVM support utilities, and C++ runtime helpers. / CodeGen 内部组件、LLVM 支持工具以及 C++ 运行时辅助设施。
