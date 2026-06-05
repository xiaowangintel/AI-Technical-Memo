# WebAssemblyAsmTypeCheck.cpp — Code Analysis / 代码分析

## Source / 来源

- **File**: `llvm/lib/Target/WebAssembly/AsmParser/WebAssemblyAsmTypeCheck.cpp`
- **Repository**: `llvm-project`
- **Purpose (EN)**: This file is part of the WebAssembly Assembler.
- **Purpose (CN)**: 该文件位于 `llvm/lib/Target/WebAssembly/AsmParser/WebAssemblyAsmTypeCheck.cpp`，主要负责 WebAssembly 后端的目标相关汇编语法解析器。 文件内容以具体实现、辅助函数和后端决策逻辑为主。

## Line-by-Line Analysis / 逐行分析

### Lines 1-6

```cpp
//==- WebAssemblyAsmTypeCheck.cpp - Assembler for WebAssembly -*- C++ -*-==//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
```
- **EN**: Sets up the file banner, licensing information, and the initial context for the rest of the source file.
- **CN**: 这一段给出文件横幅、许可证信息以及后续源码部分的初始上下文。

### Lines 7-13

```cpp
//===----------------------------------------------------------------------===//
///
/// \file
/// This file is part of the WebAssembly Assembler.
///
/// It contains code to translate a parsed .s file into MCInsts.
///
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "\file". The logic interacts with LLVM's MC layer.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“\file”。 相关逻辑会与 LLVM 的 MC 层交互。

### Lines 14-20

```cpp
//===----------------------------------------------------------------------===//

#include "AsmParser/WebAssemblyAsmTypeCheck.h"
#include "MCTargetDesc/WebAssemblyMCAsmInfo.h"
#include "MCTargetDesc/WebAssemblyMCTargetDesc.h"
#include "MCTargetDesc/WebAssemblyMCTypeUtilities.h"
#include "MCTargetDesc/WebAssemblyTargetStreamer.h"
```
- **EN**: Pulls in direct dependencies required by this assembly parser for target-specific syntax, so later declarations can reuse LLVM infrastructure and target-specific helpers. The logic interacts with LLVM's MC layer.
- **CN**: 这一段引入该目标相关汇编语法解析器所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。 相关逻辑会与 LLVM 的 MC 层交互。

### Lines 21-27

```cpp
#include "TargetInfo/WebAssemblyTargetInfo.h"
#include "llvm/ADT/StringExtras.h"
#include "llvm/MC/MCContext.h"
#include "llvm/MC/MCExpr.h"
#include "llvm/MC/MCInst.h"
#include "llvm/MC/MCInstrInfo.h"
#include "llvm/MC/MCParser/MCParsedAsmOperand.h"
```
- **EN**: Pulls in direct dependencies required by this assembly parser for target-specific syntax, so later declarations can reuse LLVM infrastructure and target-specific helpers. The logic interacts with LLVM's MC layer.
- **CN**: 这一段引入该目标相关汇编语法解析器所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。 相关逻辑会与 LLVM 的 MC 层交互。

### Lines 28-34

```cpp
#include "llvm/MC/MCParser/MCTargetAsmParser.h"
#include "llvm/MC/MCSectionWasm.h"
#include "llvm/MC/MCStreamer.h"
#include "llvm/MC/MCSubtargetInfo.h"
#include "llvm/MC/MCSymbol.h"
#include "llvm/MC/MCSymbolWasm.h"
#include "llvm/MC/TargetRegistry.h"
```
- **EN**: Pulls in direct dependencies required by this assembly parser for target-specific syntax, so later declarations can reuse LLVM infrastructure and target-specific helpers. Subtarget feature gating influences the behavior here.
- **CN**: 这一段引入该目标相关汇编语法解析器所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。 子目标特性裁剪会影响这里的行为。

### Lines 35-44

```cpp
#include "llvm/Support/Compiler.h"
#include "llvm/Support/SourceMgr.h"
#include <sstream>

using namespace llvm;

#define DEBUG_TYPE "wasm-asm-parser"

extern StringRef getMnemonic(unsigned Opc);
```
- **EN**: Pulls in direct dependencies required by this assembly parser for target-specific syntax, so later declarations can reuse LLVM infrastructure and target-specific helpers. Notable symbols in this range include `getMnemonic`.
- **CN**: 这一段引入该目标相关汇编语法解析器所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。 该区间中较显眼的符号包括 `getMnemonic`。

### Lines 45-82

```cpp
namespace llvm {

WebAssemblyAsmTypeCheck::WebAssemblyAsmTypeCheck(MCAsmParser &Parser,
                                                 const MCInstrInfo &MII,
                                                 bool Is64)
    : Parser(Parser), MII(MII), Is64(Is64) {}

void WebAssemblyAsmTypeCheck::funcDecl(const wasm::WasmSignature &Sig) {
  LocalTypes.assign(Sig.Params.begin(), Sig.Params.end());
  BlockInfoStack.push_back({Sig, 0, false});
}

void WebAssemblyAsmTypeCheck::localDecl(
    const SmallVectorImpl<wasm::ValType> &Locals) {
  llvm::append_range(LocalTypes, Locals);
}

void WebAssemblyAsmTypeCheck::dumpTypeStack(Twine Msg) {
  LLVM_DEBUG({ dbgs() << Msg << getTypesString(Stack) << "\n"; });
}

bool WebAssemblyAsmTypeCheck::typeError(SMLoc ErrorLoc, const Twine &Msg) {
  dumpTypeStack("current stack: ");
  return Parser.Error(ErrorLoc, Msg);
}

bool WebAssemblyAsmTypeCheck::match(StackType TypeA, StackType TypeB) {
  // These should have been filtered out in checkTypes()
  assert(!std::get_if<Polymorphic>(&TypeA) &&
         !std::get_if<Polymorphic>(&TypeB));

  if (TypeA == TypeB)
    return false;
  if (std::get_if<Any>(&TypeA) || std::get_if<Any>(&TypeB))
    return false;

  if (std::get_if<Ref>(&TypeB))
    std::swap(TypeA, TypeB);
```
- **EN**: Opens or re-enters namespace scope so backend symbols stay grouped under LLVM naming conventions. Stack-frame layout or stack access is relevant in this range.
- **CN**: 这里进入或重新进入命名空间作用域，使后端符号按照 LLVM 的命名约定组织在一起。 该区间与栈帧布局或栈访问相关。

### Lines 83-120

```cpp
  assert(std::get_if<wasm::ValType>(&TypeB));
  if (std::get_if<Ref>(&TypeA) &&
      WebAssembly::isRefType(std::get<wasm::ValType>(TypeB)))
    return false;
  return true;
}

std::string WebAssemblyAsmTypeCheck::getTypesString(ArrayRef<StackType> Types,
                                                    size_t StartPos) {
  SmallVector<std::string, 4> TypeStrs;
  for (auto I = Types.size(); I > StartPos; I--) {
    if (std::get_if<Polymorphic>(&Types[I - 1])) {
      TypeStrs.push_back("...");
      break;
    }
    if (std::get_if<Any>(&Types[I - 1]))
      TypeStrs.push_back("any");
    else if (std::get_if<Ref>(&Types[I - 1]))
      TypeStrs.push_back("ref");
    else
      TypeStrs.push_back(
          WebAssembly::typeToString(std::get<wasm::ValType>(Types[I - 1])));
  }

  std::string S;
  raw_string_ostream SS(S);
  SS << "[";
  ListSeparator LS;
  for (StringRef Type : reverse(TypeStrs))
    SS << LS << Type;
  SS << "]";
  return SS.str();
}

std::string
WebAssemblyAsmTypeCheck::getTypesString(ArrayRef<wasm::ValType> Types,
                                        size_t StartPos) {
  return getTypesString(valTypesToStackTypes(Types), StartPos);
```
- **EN**: Implements helper routine(s) `isRefType`, `getTypesString`, `size` for this portion of the WebAssembly backend assembly parser for target-specific syntax. Stack-frame layout or stack access is relevant in this range.
- **CN**: 这里实现了 WebAssembly 后端该部分目标相关汇编语法解析器所需的辅助例程 `isRefType`, `getTypesString`, `size`。 该区间与栈帧布局或栈访问相关。

### Lines 121-158

```cpp
}

SmallVector<WebAssemblyAsmTypeCheck::StackType, 4>
WebAssemblyAsmTypeCheck::valTypesToStackTypes(
    ArrayRef<wasm::ValType> ValTypes) {
  SmallVector<StackType, 4> Types(ValTypes.size());
  llvm::transform(ValTypes, Types.begin(),
                  [](wasm::ValType Val) -> StackType { return Val; });
  return Types;
}

bool WebAssemblyAsmTypeCheck::checkTypes(SMLoc ErrorLoc,
                                         ArrayRef<wasm::ValType> ValTypes,
                                         bool ExactMatch) {
  return checkTypes(ErrorLoc, valTypesToStackTypes(ValTypes), ExactMatch);
}

bool WebAssemblyAsmTypeCheck::checkTypes(SMLoc ErrorLoc,
                                         ArrayRef<StackType> Types,
                                         bool ExactMatch) {
  auto StackI = Stack.size();
  auto TypeI = Types.size();
  assert(!BlockInfoStack.empty());
  auto BlockStackStartPos = BlockInfoStack.back().StackStartPos;
  bool Error = false;
  bool PolymorphicStack = false;
  // Compare elements one by one from the stack top
  for (; StackI > BlockStackStartPos && TypeI > 0; StackI--, TypeI--) {
    // If the stack is polymorphic, we assume all types in 'Types' have been
    // compared and matched
    if (std::get_if<Polymorphic>(&Stack[StackI - 1])) {
      TypeI = 0;
      break;
    }
    if (match(Stack[StackI - 1], Types[TypeI - 1])) {
      Error = true;
      break;
    }
```
- **EN**: Implements helper routine(s) `valTypesToStackTypes`, `Types`, `size` for this portion of the WebAssembly backend assembly parser for target-specific syntax. Stack-frame layout or stack access is relevant in this range.
- **CN**: 这里实现了 WebAssembly 后端该部分目标相关汇编语法解析器所需的辅助例程 `valTypesToStackTypes`, `Types`, `size`。 该区间与栈帧布局或栈访问相关。

### Lines 159-196

```cpp
  }

  // If the stack top is polymorphic, the stack is in the polymorphic state.
  if (StackI > BlockStackStartPos &&
      std::get_if<Polymorphic>(&Stack[StackI - 1]))
    PolymorphicStack = true;

  // Even if no match failure has happened in the loop above, if not all
  // elements of Types has been matched, that means we don't have enough
  // elements on the stack.
  //
  // Also, if not all elements of the Stack has been matched and when
  // 'ExactMatch' is true and the current stack is not polymorphic, that means
  // we have superfluous elements remaining on the stack (e.g. at the end of a
  // function).
  if (TypeI > 0 ||
      (ExactMatch && !PolymorphicStack && StackI > BlockStackStartPos))
    Error = true;

  if (!Error)
    return false;

  auto StackStartPos = ExactMatch
                           ? BlockStackStartPos
                           : std::max((int)BlockStackStartPos,
                                      (int)Stack.size() - (int)Types.size());
  return typeError(ErrorLoc, "type mismatch, expected " +
                                 getTypesString(Types) + " but got " +
                                 getTypesString(Stack, StackStartPos));
}

bool WebAssemblyAsmTypeCheck::popTypes(SMLoc ErrorLoc,
                                       ArrayRef<wasm::ValType> ValTypes,
                                       bool ExactMatch) {
  return popTypes(ErrorLoc, valTypesToStackTypes(ValTypes), ExactMatch);
}

bool WebAssemblyAsmTypeCheck::popTypes(SMLoc ErrorLoc,
```
- **EN**: Implements helper routine(s) `stack`, `max`, `size` for this portion of the WebAssembly backend assembly parser for target-specific syntax. Stack-frame layout or stack access is relevant in this range.
- **CN**: 这里实现了 WebAssembly 后端该部分目标相关汇编语法解析器所需的辅助例程 `stack`, `max`, `size`。 该区间与栈帧布局或栈访问相关。

### Lines 197-234

```cpp
                                       ArrayRef<StackType> Types,
                                       bool ExactMatch) {
  bool Error = checkTypes(ErrorLoc, Types, ExactMatch);
  auto NumPops = std::min(Stack.size() - BlockInfoStack.back().StackStartPos,
                          Types.size());
  for (size_t I = 0, E = NumPops; I != E; I++) {
    if (std::get_if<Polymorphic>(&Stack.back()))
      break;
    Stack.pop_back();
  }
  return Error;
}

bool WebAssemblyAsmTypeCheck::popType(SMLoc ErrorLoc, StackType Type) {
  return popTypes(ErrorLoc, {Type});
}

bool WebAssemblyAsmTypeCheck::popRefType(SMLoc ErrorLoc) {
  return popType(ErrorLoc, Ref{});
}

bool WebAssemblyAsmTypeCheck::popAnyType(SMLoc ErrorLoc) {
  return popType(ErrorLoc, Any{});
}

void WebAssemblyAsmTypeCheck::pushTypes(ArrayRef<wasm::ValType> ValTypes) {
  Stack.append(valTypesToStackTypes(ValTypes));
}

bool WebAssemblyAsmTypeCheck::getLocal(SMLoc ErrorLoc, const MCOperand &LocalOp,
                                       wasm::ValType &Type) {
  auto Local = static_cast<size_t>(LocalOp.getImm());
  if (Local >= LocalTypes.size())
    return typeError(ErrorLoc, StringRef("no local type specified for index ") +
                                   std::to_string(Local));
  Type = LocalTypes[Local];
  return false;
}
```
- **EN**: Implements helper routine(s) `checkTypes`, `min`, `size` for this portion of the WebAssembly backend assembly parser for target-specific syntax. Stack-frame layout or stack access is relevant in this range.
- **CN**: 这里实现了 WebAssembly 后端该部分目标相关汇编语法解析器所需的辅助例程 `checkTypes`, `min`, `size`。 该区间与栈帧布局或栈访问相关。

### Lines 235-272

```cpp

bool WebAssemblyAsmTypeCheck::checkSig(SMLoc ErrorLoc,
                                       const wasm::WasmSignature &Sig) {
  bool Error = popTypes(ErrorLoc, Sig.Params);
  pushTypes(Sig.Returns);
  return Error;
}

bool WebAssemblyAsmTypeCheck::getSymRef(SMLoc ErrorLoc, const MCOperand &SymOp,
                                        const MCSymbolRefExpr *&SymRef) {
  if (!SymOp.isExpr())
    return typeError(ErrorLoc, StringRef("expected expression operand"));
  SymRef = dyn_cast<MCSymbolRefExpr>(SymOp.getExpr());
  if (!SymRef)
    return typeError(ErrorLoc, StringRef("expected symbol operand"));
  return false;
}

bool WebAssemblyAsmTypeCheck::getGlobal(SMLoc ErrorLoc,
                                        const MCOperand &GlobalOp,
                                        wasm::ValType &Type) {
  const MCSymbolRefExpr *SymRef;
  if (getSymRef(ErrorLoc, GlobalOp, SymRef))
    return true;
  auto *WasmSym = static_cast<const MCSymbolWasm *>(&SymRef->getSymbol());
  switch (WasmSym->getType().value_or(wasm::WASM_SYMBOL_TYPE_DATA)) {
  case wasm::WASM_SYMBOL_TYPE_GLOBAL:
    Type = static_cast<wasm::ValType>(WasmSym->getGlobalType().Type);
    break;
  case wasm::WASM_SYMBOL_TYPE_FUNCTION:
  case wasm::WASM_SYMBOL_TYPE_DATA:
    switch (SymRef->getSpecifier()) {
    case WebAssembly::S_GOT:
    case WebAssembly::S_GOT_TLS:
      Type = Is64 ? wasm::ValType::I64 : wasm::ValType::I32;
      return false;
    default:
      break;
```
- **EN**: Implements helper routine(s) `checkSig`, `popTypes`, `pushTypes` for this portion of the WebAssembly backend assembly parser for target-specific syntax.
- **CN**: 这里实现了 WebAssembly 后端该部分目标相关汇编语法解析器所需的辅助例程 `checkSig`, `popTypes`, `pushTypes`。

### Lines 273-310

```cpp
    }
    [[fallthrough]];
  default:
    return typeError(ErrorLoc, StringRef("symbol ") + WasmSym->getName() +
                                   ": missing .globaltype");
  }
  return false;
}

bool WebAssemblyAsmTypeCheck::getTable(SMLoc ErrorLoc, const MCOperand &TableOp,
                                       wasm::ValType &Type) {
  const MCSymbolRefExpr *SymRef;
  if (getSymRef(ErrorLoc, TableOp, SymRef))
    return true;
  auto *WasmSym = static_cast<const MCSymbolWasm *>(&SymRef->getSymbol());
  if (WasmSym->getType().value_or(wasm::WASM_SYMBOL_TYPE_DATA) !=
      wasm::WASM_SYMBOL_TYPE_TABLE)
    return typeError(ErrorLoc, StringRef("symbol ") + WasmSym->getName() +
                                   ": missing .tabletype");
  Type = static_cast<wasm::ValType>(WasmSym->getTableType().ElemType);
  return false;
}

bool WebAssemblyAsmTypeCheck::getSignature(SMLoc ErrorLoc,
                                           const MCOperand &SigOp,
                                           wasm::WasmSymbolType Type,
                                           const wasm::WasmSignature *&Sig) {
  const MCSymbolRefExpr *SymRef = nullptr;
  if (getSymRef(ErrorLoc, SigOp, SymRef))
    return true;
  auto *WasmSym = static_cast<const MCSymbolWasm *>(&SymRef->getSymbol());
  Sig = WasmSym->getSignature();

  if (!Sig || WasmSym->getType() != Type) {
    const char *TypeName = nullptr;
    switch (Type) {
    case wasm::WASM_SYMBOL_TYPE_FUNCTION:
      TypeName = "func";
```
- **EN**: Implements helper routine(s) `typeError`, `StringRef`, `getName` for this portion of the WebAssembly backend assembly parser for target-specific syntax.
- **CN**: 这里实现了 WebAssembly 后端该部分目标相关汇编语法解析器所需的辅助例程 `typeError`, `StringRef`, `getName`。

### Lines 311-348

```cpp
      break;
    case wasm::WASM_SYMBOL_TYPE_TAG:
      TypeName = "tag";
      break;
    default:
      llvm_unreachable("Signature symbol should either be a function or a tag");
    }
    return typeError(ErrorLoc, StringRef("symbol ") + WasmSym->getName() +
                                   ": missing ." + TypeName + "type");
  }
  return false;
}

bool WebAssemblyAsmTypeCheck::endOfFunction(SMLoc ErrorLoc, bool ExactMatch) {
  assert(!BlockInfoStack.empty());
  const auto &FuncInfo = BlockInfoStack[0];
  return checkTypes(ErrorLoc, FuncInfo.Sig.Returns, ExactMatch);
}

// Unlike checkTypes() family, this just compare the equivalence of the two
// ValType vectors
static bool compareTypes(ArrayRef<wasm::ValType> TypesA,
                         ArrayRef<wasm::ValType> TypesB) {
  if (TypesA.size() != TypesB.size())
    return true;
  for (size_t I = 0, E = TypesA.size(); I < E; I++)
    if (TypesA[I] != TypesB[I])
      return true;
  return false;
}

bool WebAssemblyAsmTypeCheck::checkTryTable(SMLoc ErrorLoc,
                                            const MCInst &Inst) {
  bool Error = false;
  unsigned OpIdx = 1; // OpIdx 0 is the block type
  int64_t NumCatches = Inst.getOperand(OpIdx++).getImm();
  for (int64_t I = 0; I < NumCatches; I++) {
    int64_t Opcode = Inst.getOperand(OpIdx++).getImm();
```
- **EN**: Implements helper routine(s) `llvm_unreachable`, `typeError`, `StringRef` for this portion of the WebAssembly backend assembly parser for target-specific syntax. Stack-frame layout or stack access is relevant in this range.
- **CN**: 这里实现了 WebAssembly 后端该部分目标相关汇编语法解析器所需的辅助例程 `llvm_unreachable`, `typeError`, `StringRef`。 该区间与栈帧布局或栈访问相关。

### Lines 349-386

```cpp
    std::string ErrorMsgBase =
        "try_table: catch index " + std::to_string(I) + ": ";

    const wasm::WasmSignature *Sig = nullptr;
    SmallVector<wasm::ValType> SentTypes;
    if (Opcode == wasm::WASM_OPCODE_CATCH ||
        Opcode == wasm::WASM_OPCODE_CATCH_REF) {
      if (!getSignature(ErrorLoc, Inst.getOperand(OpIdx++),
                        wasm::WASM_SYMBOL_TYPE_TAG, Sig))
        llvm::append_range(SentTypes, Sig->Params);
      else
        Error = true;
    }
    if (Opcode == wasm::WASM_OPCODE_CATCH_REF ||
        Opcode == wasm::WASM_OPCODE_CATCH_ALL_REF) {
      SentTypes.push_back(wasm::ValType::EXNREF);
    }

    unsigned Level = Inst.getOperand(OpIdx++).getImm();
    if (Level < BlockInfoStack.size()) {
      const auto &DestBlockInfo =
          BlockInfoStack[BlockInfoStack.size() - Level - 1];
      ArrayRef<wasm::ValType> DestTypes;
      if (DestBlockInfo.IsLoop)
        DestTypes = DestBlockInfo.Sig.Params;
      else
        DestTypes = DestBlockInfo.Sig.Returns;
      if (compareTypes(SentTypes, DestTypes)) {
        std::string ErrorMsg =
            ErrorMsgBase + "type mismatch, catch tag type is " +
            getTypesString(SentTypes) + ", but destination's type is " +
            getTypesString(DestTypes);
        Error |= typeError(ErrorLoc, ErrorMsg);
      }
    } else {
      Error = typeError(ErrorLoc, ErrorMsgBase + "invalid depth " +
                                      std::to_string(Level));
    }
```
- **EN**: Implements helper routine(s) `to_string`, `getSignature`, `getOperand` for this portion of the WebAssembly backend assembly parser for target-specific syntax. Stack-frame layout or stack access is relevant in this range.
- **CN**: 这里实现了 WebAssembly 后端该部分目标相关汇编语法解析器所需的辅助例程 `to_string`, `getSignature`, `getOperand`。 该区间与栈帧布局或栈访问相关。

### Lines 387-424

```cpp
  }
  return Error;
}

bool WebAssemblyAsmTypeCheck::typeCheck(SMLoc ErrorLoc, const MCInst &Inst,
                                        OperandVector &Operands) {
  auto Opc = Inst.getOpcode();
  auto Name = getMnemonic(Opc);
  dumpTypeStack("typechecking " + Name + ": ");
  wasm::ValType Type;

  if (Name == "local.get") {
    if (!getLocal(Operands[1]->getStartLoc(), Inst.getOperand(0), Type)) {
      pushType(Type);
      return false;
    }
    pushType(Any{});
    return true;
  }

  if (Name == "local.set") {
    if (!getLocal(Operands[1]->getStartLoc(), Inst.getOperand(0), Type))
      return popType(ErrorLoc, Type);
    popType(ErrorLoc, Any{});
    return true;
  }

  if (Name == "local.tee") {
    if (!getLocal(Operands[1]->getStartLoc(), Inst.getOperand(0), Type)) {
      bool Error = popType(ErrorLoc, Type);
      pushType(Type);
      return Error;
    }
    popType(ErrorLoc, Any{});
    pushType(Any{});
    return true;
  }
```
- **EN**: Implements helper routine(s) `typeCheck`, `getOpcode`, `getMnemonic` for this portion of the WebAssembly backend assembly parser for target-specific syntax. Stack-frame layout or stack access is relevant in this range.
- **CN**: 这里实现了 WebAssembly 后端该部分目标相关汇编语法解析器所需的辅助例程 `typeCheck`, `getOpcode`, `getMnemonic`。 该区间与栈帧布局或栈访问相关。

### Lines 425-462

```cpp
  if (Name == "global.get") {
    if (!getGlobal(Operands[1]->getStartLoc(), Inst.getOperand(0), Type)) {
      pushType(Type);
      return false;
    }
    pushType(Any{});
    return true;
  }

  if (Name == "global.set") {
    if (!getGlobal(Operands[1]->getStartLoc(), Inst.getOperand(0), Type))
      return popType(ErrorLoc, Type);
    popType(ErrorLoc, Any{});
    return true;
  }

  if (Name == "table.get") {
    bool Error = popType(ErrorLoc, wasm::ValType::I32);
    if (!getTable(Operands[1]->getStartLoc(), Inst.getOperand(0), Type)) {
      pushType(Type);
      return Error;
    }
    pushType(Any{});
    return true;
  }

  if (Name == "table.set") {
    bool Error = false;
    SmallVector<StackType, 2> PopTypes;
    PopTypes.push_back(wasm::ValType::I32);
    if (!getTable(Operands[1]->getStartLoc(), Inst.getOperand(0), Type)) {
      PopTypes.push_back(Type);
    } else {
      Error = true;
      PopTypes.push_back(Any{});
    }
    Error |= popTypes(ErrorLoc, PopTypes);
    return Error;
```
- **EN**: Implements helper routine(s) `getGlobal`, `getStartLoc`, `getOperand` for this portion of the WebAssembly backend assembly parser for target-specific syntax. Stack-frame layout or stack access is relevant in this range.
- **CN**: 这里实现了 WebAssembly 后端该部分目标相关汇编语法解析器所需的辅助例程 `getGlobal`, `getStartLoc`, `getOperand`。 该区间与栈帧布局或栈访问相关。

### Lines 463-500

```cpp
  }

  if (Name == "table.size") {
    bool Error = getTable(Operands[1]->getStartLoc(), Inst.getOperand(0), Type);
    pushType(wasm::ValType::I32);
    return Error;
  }

  if (Name == "table.grow") {
    bool Error = false;
    SmallVector<StackType, 2> PopTypes;
    if (!getTable(Operands[1]->getStartLoc(), Inst.getOperand(0), Type)) {
      PopTypes.push_back(Type);
    } else {
      Error = true;
      PopTypes.push_back(Any{});
    }
    PopTypes.push_back(wasm::ValType::I32);
    Error |= popTypes(ErrorLoc, PopTypes);
    pushType(wasm::ValType::I32);
    return Error;
  }

  if (Name == "table.fill") {
    bool Error = false;
    SmallVector<StackType, 2> PopTypes;
    PopTypes.push_back(wasm::ValType::I32);
    if (!getTable(Operands[1]->getStartLoc(), Inst.getOperand(0), Type)) {
      PopTypes.push_back(Type);
    } else {
      Error = true;
      PopTypes.push_back(Any{});
    }
    PopTypes.push_back(wasm::ValType::I32);
    Error |= popTypes(ErrorLoc, PopTypes);
    return Error;
  }
```
- **EN**: Implements helper routine(s) `getTable`, `getStartLoc`, `getOperand` for this portion of the WebAssembly backend assembly parser for target-specific syntax. Stack-frame layout or stack access is relevant in this range.
- **CN**: 这里实现了 WebAssembly 后端该部分目标相关汇编语法解析器所需的辅助例程 `getTable`, `getStartLoc`, `getOperand`。 该区间与栈帧布局或栈访问相关。

### Lines 501-538

```cpp
  if (Name == "memory.fill") {
    Type = Is64 ? wasm::ValType::I64 : wasm::ValType::I32;
    bool Error = popType(ErrorLoc, Type);
    Error |= popType(ErrorLoc, wasm::ValType::I32);
    Error |= popType(ErrorLoc, Type);
    return Error;
  }

  if (Name == "memory.copy") {
    Type = Is64 ? wasm::ValType::I64 : wasm::ValType::I32;
    bool Error = popType(ErrorLoc, Type);
    Error |= popType(ErrorLoc, Type);
    Error |= popType(ErrorLoc, Type);
    return Error;
  }

  if (Name == "memory.init") {
    Type = Is64 ? wasm::ValType::I64 : wasm::ValType::I32;
    bool Error = popType(ErrorLoc, wasm::ValType::I32);
    Error |= popType(ErrorLoc, wasm::ValType::I32);
    Error |= popType(ErrorLoc, Type);
    return Error;
  }

  if (Name == "drop") {
    return popType(ErrorLoc, Any{});
  }

  if (Name == "block" || Name == "loop" || Name == "if" || Name == "try" ||
      Name == "try_table") {
    bool Error = Name == "if" && popType(ErrorLoc, wasm::ValType::I32);
    // Pop block input parameters and check their types are correct
    Error |= popTypes(ErrorLoc, LastSig.Params);
    if (Name == "try_table")
      Error |= checkTryTable(ErrorLoc, Inst);
    // Push a new block info
    BlockInfoStack.push_back({LastSig, Stack.size(), Name == "loop"});
    // Push back block input parameters
```
- **EN**: Implements helper routine(s) `popType`, `popTypes`, `checkTryTable` for this portion of the WebAssembly backend assembly parser for target-specific syntax. Stack-frame layout or stack access is relevant in this range.
- **CN**: 这里实现了 WebAssembly 后端该部分目标相关汇编语法解析器所需的辅助例程 `popType`, `popTypes`, `checkTryTable`。 该区间与栈帧布局或栈访问相关。

### Lines 539-576

```cpp
    pushTypes(LastSig.Params);
    return Error;
  }

  if (Name == "end_block" || Name == "end_loop" || Name == "end_if" ||
      Name == "end_try" || Name == "delegate" || Name == "end_try_table" ||
      Name == "else" || Name == "catch" || Name == "catch_all") {
    assert(!BlockInfoStack.empty());
    // Check if the types on the stack match with the block return type
    const auto &LastBlockInfo = BlockInfoStack.back();
    bool Error = checkTypes(ErrorLoc, LastBlockInfo.Sig.Returns, true);
    // Pop all types added to the stack for the current block level
    Stack.truncate(LastBlockInfo.StackStartPos);
    if (Name == "else") {
      // 'else' expects the block input parameters to be on the stack, in the
      // same way we entered 'if'
      pushTypes(LastBlockInfo.Sig.Params);
    } else if (Name == "catch") {
      // 'catch' instruction pushes values whose types are specified in the
      // tag's 'params' part
      const wasm::WasmSignature *Sig = nullptr;
      if (!getSignature(Operands[1]->getStartLoc(), Inst.getOperand(0),
                        wasm::WASM_SYMBOL_TYPE_TAG, Sig))
        pushTypes(Sig->Params);
      else
        Error = true;
    } else if (Name == "catch_all") {
      // 'catch_all' does not push anything onto the stack
    } else {
      // For normal end markers, push block return value types onto the stack
      // and pop the block info
      pushTypes(LastBlockInfo.Sig.Returns);
      BlockInfoStack.pop_back();
    }
    return Error;
  }

  if (Name == "br" || Name == "br_if") {
```
- **EN**: Implements helper routine(s) `pushTypes`, `empty`, `back` for this portion of the WebAssembly backend assembly parser for target-specific syntax. Stack-frame layout or stack access is relevant in this range.
- **CN**: 这里实现了 WebAssembly 后端该部分目标相关汇编语法解析器所需的辅助例程 `pushTypes`, `empty`, `back`。 该区间与栈帧布局或栈访问相关。

### Lines 577-614

```cpp
    bool Error = false;
    if (Name == "br_if")
      Error |= popType(ErrorLoc, wasm::ValType::I32); // cond
    const MCOperand &Operand = Inst.getOperand(0);
    if (Operand.isImm()) {
      unsigned Level = Operand.getImm();
      if (Level < BlockInfoStack.size()) {
        const auto &DestBlockInfo =
            BlockInfoStack[BlockInfoStack.size() - Level - 1];
        if (DestBlockInfo.IsLoop)
          Error |= checkTypes(ErrorLoc, DestBlockInfo.Sig.Params, false);
        else
          Error |= checkTypes(ErrorLoc, DestBlockInfo.Sig.Returns, false);
      } else {
        Error = typeError(ErrorLoc, StringRef("br: invalid depth ") +
                                        std::to_string(Level));
      }
    } else {
      Error =
          typeError(Operands[1]->getStartLoc(), "depth should be an integer");
    }
    if (Name == "br")
      pushType(Polymorphic{});
    return Error;
  }

  if (Name == "return") {
    bool Error = endOfFunction(ErrorLoc, false);
    pushType(Polymorphic{});
    return Error;
  }

  if (Name == "call_indirect" || Name == "return_call_indirect") {
    // Function value.
    bool Error = popType(ErrorLoc, wasm::ValType::I32);
    Error |= checkSig(ErrorLoc, LastSig);
    if (Name == "return_call_indirect") {
      Error |= endOfFunction(ErrorLoc, false);
```
- **EN**: Implements helper routine(s) `popType`, `getOperand`, `isImm` for this portion of the WebAssembly backend assembly parser for target-specific syntax. Stack-frame layout or stack access is relevant in this range.
- **CN**: 这里实现了 WebAssembly 后端该部分目标相关汇编语法解析器所需的辅助例程 `popType`, `getOperand`, `isImm`。 该区间与栈帧布局或栈访问相关。

### Lines 615-652

```cpp
      pushType(Polymorphic{});
    }
    return Error;
  }

  if (Name == "call_ref" || Name == "return_call_ref") {
    // Funcref target popped from the stack, followed by the signature's
    // parameters; pushes the signature's results.
    bool Error = popType(ErrorLoc, wasm::ValType::FUNCREF);
    Error |= checkSig(ErrorLoc, LastSig);
    if (Name == "return_call_ref") {
      Error |= endOfFunction(ErrorLoc, false);
      pushType(Polymorphic{});
    }
    return Error;
  }

  if (Name == "select") {
    // Typed select pops an i32 condition and two values of each declared
    // type, then pushes the declared types back. The result type list lives
    // in the MCInst operands as a count followed by that many valtype bytes.
    bool Error = popType(ErrorLoc, wasm::ValType::I32);
    if (Inst.getNumOperands() == 0)
      return typeError(ErrorLoc, "select missing type-list operand");
    uint64_t Count = uint64_t(Inst.getOperand(0).getImm());
    if (Count > uint64_t(Inst.getNumOperands() - 1))
      return typeError(ErrorLoc,
                       "select type-list count exceeds operand count");
    SmallVector<wasm::ValType, 1> Types;
    Types.reserve(Count);
    for (uint64_t I = 0; I < Count; ++I)
      Types.push_back(
          static_cast<wasm::ValType>(Inst.getOperand(1 + I).getImm()));
    Error |= popTypes(ErrorLoc, Types);
    Error |= popTypes(ErrorLoc, Types);
    pushTypes(Types);
    return Error;
  }
```
- **EN**: Implements helper routine(s) `pushType`, `popType`, `checkSig` for this portion of the WebAssembly backend assembly parser for target-specific syntax. The logic interacts with LLVM's MC layer.
- **CN**: 这里实现了 WebAssembly 后端该部分目标相关汇编语法解析器所需的辅助例程 `pushType`, `popType`, `checkSig`。 相关逻辑会与 LLVM 的 MC 层交互。

### Lines 653-690

```cpp

  if (Name == "call" || Name == "return_call") {
    bool Error = false;
    const wasm::WasmSignature *Sig = nullptr;
    if (!getSignature(Operands[1]->getStartLoc(), Inst.getOperand(0),
                      wasm::WASM_SYMBOL_TYPE_FUNCTION, Sig))
      Error |= checkSig(ErrorLoc, *Sig);
    else
      Error = true;
    if (Name == "return_call") {
      Error |= endOfFunction(ErrorLoc, false);
      pushType(Polymorphic{});
    }
    return Error;
  }

  if (Name == "unreachable") {
    pushType(Polymorphic{});
    return false;
  }

  if (Name == "ref.is_null") {
    bool Error = popRefType(ErrorLoc);
    pushType(wasm::ValType::I32);
    return Error;
  }

  if (Name == "throw") {
    bool Error = false;
    const wasm::WasmSignature *Sig = nullptr;
    if (!getSignature(Operands[1]->getStartLoc(), Inst.getOperand(0),
                      wasm::WASM_SYMBOL_TYPE_TAG, Sig))
      Error |= checkSig(ErrorLoc, *Sig);
    else
      Error = true;
    pushType(Polymorphic{});
    return Error;
  }
```
- **EN**: Implements helper routine(s) `getSignature`, `getStartLoc`, `getOperand` for this portion of the WebAssembly backend assembly parser for target-specific syntax.
- **CN**: 这里实现了 WebAssembly 后端该部分目标相关汇编语法解析器所需的辅助例程 `getSignature`, `getStartLoc`, `getOperand`。

### Lines 691-723

```cpp

  if (Name == "throw_ref") {
    bool Error = popType(ErrorLoc, wasm::ValType::EXNREF);
    pushType(Polymorphic{});
    return Error;
  }

  // The current instruction is a stack instruction which doesn't have
  // explicit operands that indicate push/pop types, so we get those from
  // the register version of the same instruction.
  auto RegOpc = WebAssembly::getRegisterOpcode(Opc);
  assert(RegOpc != -1 && "Failed to get register version of MC instruction");
  const auto &II = MII.get(RegOpc);
  // First pop all the uses off the stack and check them.
  SmallVector<wasm::ValType, 4> PopTypes;
  for (unsigned I = II.getNumDefs(); I < II.getNumOperands(); I++) {
    const auto &Op = II.operands()[I];
    if (Op.OperandType == MCOI::OPERAND_REGISTER)
      PopTypes.push_back(WebAssembly::regClassToValType(Op.RegClass));
  }
  bool Error = popTypes(ErrorLoc, PopTypes);
  SmallVector<wasm::ValType, 4> PushTypes;
  // Now push all the defs onto the stack.
  for (unsigned I = 0; I < II.getNumDefs(); I++) {
    const auto &Op = II.operands()[I];
    assert(Op.OperandType == MCOI::OPERAND_REGISTER && "Register expected");
    PushTypes.push_back(WebAssembly::regClassToValType(Op.RegClass));
  }
  pushTypes(PushTypes);
  return Error;
}

} // end namespace llvm
```
- **EN**: Implements helper routine(s) `popType`, `pushType`, `getRegisterOpcode` for this portion of the WebAssembly backend assembly parser for target-specific syntax.
- **CN**: 这里实现了 WebAssembly 后端该部分目标相关汇编语法解析器所需的辅助例程 `popType`, `pushType`, `getRegisterOpcode`。

## Key Concepts / 关键概念

- WebAssembly backend integration / WebAssembly 后端集成
- Assembly parser for target-specific syntax / 目标相关汇编语法解析器
- Stack frame management / 栈帧管理
- Subtarget features / 子目标特性
- Assembly parsing / 汇编解析
- Assembly or MC emission / 汇编或 MC 发射
- Instruction selection or opcode handling / 指令选择或操作码处理
- Object format integration / 目标文件格式集成

## Dependencies / 依赖关系

### Direct Includes / 直接包含

- `AsmParser/WebAssemblyAsmTypeCheck.h`
- `MCTargetDesc/WebAssemblyMCAsmInfo.h`
- `MCTargetDesc/WebAssemblyMCTargetDesc.h`
- `MCTargetDesc/WebAssemblyMCTypeUtilities.h`
- `MCTargetDesc/WebAssemblyTargetStreamer.h`
- `TargetInfo/WebAssemblyTargetInfo.h`
- `llvm/ADT/StringExtras.h`
- `llvm/MC/MCContext.h`
- `llvm/MC/MCExpr.h`
- `llvm/MC/MCInst.h`
- `llvm/MC/MCInstrInfo.h`
- `llvm/MC/MCParser/MCParsedAsmOperand.h`
- `llvm/MC/MCParser/MCTargetAsmParser.h`
- `llvm/MC/MCSectionWasm.h`
- `llvm/MC/MCStreamer.h`
- `llvm/MC/MCSubtargetInfo.h`
- `llvm/MC/MCSymbol.h`
- `llvm/MC/MCSymbolWasm.h`
- `llvm/MC/TargetRegistry.h`
- `llvm/Support/Compiler.h`
- `llvm/Support/SourceMgr.h`
- `sstream`

### Important Collaborators / 重要协作组件

- LLVM MC layer / LLVM MC 层
- LLVM Support utilities / LLVM Support 工具层
- WebAssembly target-specific helpers / WebAssembly 目标专用辅助组件
