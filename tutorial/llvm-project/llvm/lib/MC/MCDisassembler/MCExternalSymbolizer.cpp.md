# MCExternalSymbolizer.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `llvm/lib/MC/MCDisassembler/MCExternalSymbolizer.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 作用**:
  - **EN**: This file implements External symbolizer.
  - **CN**: 实现 LLVM MC 反汇编支持，把机器码字节解码为 MC 指令与符号化操作数。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```cpp
//===-- MCExternalSymbolizer.cpp - External symbolizer --------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM file banner, license notice, and high-level file description.
- **CN**: 包含 LLVM 标准文件头、许可证声明以及文件级说明。

### Lines 8-16
```cpp

#include "llvm/MC/MCDisassembler/MCExternalSymbolizer.h"
#include "llvm/MC/MCContext.h"
#include "llvm/MC/MCExpr.h"
#include "llvm/MC/MCInst.h"
#include "llvm/MC/TargetRegistry.h"
#include "llvm/Support/raw_ostream.h"
#include <cstring>

```
- **EN**: Pulls in the headers needed for this implementation, including `llvm/MC/MCDisassembler/MCExternalSymbolizer.h`, `llvm/MC/MCContext.h`, `llvm/MC/MCExpr.h`, `llvm/MC/MCInst.h`.
- **CN**: 引入该实现所需的头文件，其中包括 `llvm/MC/MCDisassembler/MCExternalSymbolizer.h`, `llvm/MC/MCContext.h`, `llvm/MC/MCExpr.h`, `llvm/MC/MCInst.h`。

### Lines 17-30
```cpp
using namespace llvm;

namespace llvm {
class Triple;
}

// This function tries to add a symbolic operand in place of the immediate
// Value in the MCInst. The immediate Value has had any PC adjustment made by
// the caller. If the instruction is a branch instruction then IsBranch is true,
// else false. If the getOpInfo() function was set as part of the
// setupForSymbolicDisassembly() call then that function is called to get any
// symbolic information at the Address for this instruction. If that returns
// non-zero then the symbolic information it returns is used to create an MCExpr
// and that is added as an operand to the MCInst. If getOpInfo() returns zero
```
- **EN**: Introduces declarations for `llvm`, `Triple`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `llvm`, `Triple` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 31-41
```cpp
// and IsBranch is true then a symbol look up for Value is done and if a symbol
// is found an MCExpr is created with that, else an MCExpr with Value is
// created. This function returns true if it adds an operand to the MCInst and
// false otherwise.
bool MCExternalSymbolizer::tryAddingSymbolicOperand(
    MCInst &MI, raw_ostream &cStream, int64_t Value, uint64_t Address,
    bool IsBranch, uint64_t Offset, uint64_t OpSize, uint64_t InstSize) {
  struct LLVMOpInfo1 SymbolicOp;
  std::memset(&SymbolicOp, '\0', sizeof(struct LLVMOpInfo1));
  SymbolicOp.Value = Value;

```
- **EN**: Introduces declarations for `LLVMOpInfo1`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `LLVMOpInfo1` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 42-55
```cpp
  if (!GetOpInfo ||
      !GetOpInfo(DisInfo, Address, Offset, OpSize, InstSize, 1, &SymbolicOp)) {
    // Clear SymbolicOp.Value from above and also all other fields.
    std::memset(&SymbolicOp, '\0', sizeof(struct LLVMOpInfo1));

    // At this point, GetOpInfo() did not find any relocation information about
    // this operand and we are left to use the SymbolLookUp() call back to guess
    // if the Value is the address of a symbol.  In the case this is a branch
    // that always makes sense to guess.  But in the case of an immediate it is
    // a bit more questionable if it is an address of a symbol or some other
    // reference.  So if the immediate Value comes from a width of 1 byte,
    // OpSize, we will not guess it is an address of a symbol.  Because in
    // object files assembled starting at address 0 this usually leads to
    // incorrect symbolication.
```
- **EN**: Introduces declarations for `LLVMOpInfo1`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `LLVMOpInfo1` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 56-69
```cpp
    if (!SymbolLookUp || (OpSize == 1 && !IsBranch))
      return false;

    uint64_t ReferenceType;
    if (IsBranch)
       ReferenceType = LLVMDisassembler_ReferenceType_In_Branch;
    else
       ReferenceType = LLVMDisassembler_ReferenceType_InOut_None;
    const char *ReferenceName;
    const char *Name = SymbolLookUp(DisInfo, Value, &ReferenceType, Address,
                                    &ReferenceName);
    if (Name) {
      SymbolicOp.AddSymbol.Name = Name;
      SymbolicOp.AddSymbol.Present = true;
```
- **EN**: Implements logic around `SymbolLookUp`; this block decodes machine-code bytes into symbolic instruction form; updates MC section or symbol state; returns subsystem-specific computed results.
- **CN**: 围绕 `SymbolLookUp` 实现具体逻辑；这一段把机器码字节解码为符号化指令形式，更新 MC 节区或符号状态，返回子系统相关的计算结果。

### Lines 70-83
```cpp
      // If Name is a C++ symbol name put the human readable name in a comment.
      if(ReferenceType == LLVMDisassembler_ReferenceType_DeMangled_Name)
        cStream << ReferenceName;
    }
    // For branches always create an MCExpr so it gets printed as hex address.
    else if (IsBranch) {
      SymbolicOp.Value = Value;
    }
    if(ReferenceType == LLVMDisassembler_ReferenceType_Out_SymbolStub)
      cStream << "symbol stub for: " << ReferenceName;
    else if(ReferenceType == LLVMDisassembler_ReferenceType_Out_Objc_Message)
      cStream << "Objc message: " << ReferenceName;
    if (!Name && !IsBranch)
      return false;
```
- **EN**: Contains supporting implementation details for the surrounding subsystem logic.
- **CN**: 包含周边子系统逻辑所需的辅助实现细节。

### Lines 84-96
```cpp
  }

  const MCExpr *Add = nullptr;
  if (SymbolicOp.AddSymbol.Present) {
    if (SymbolicOp.AddSymbol.Name) {
      StringRef Name(SymbolicOp.AddSymbol.Name);
      MCSymbol *Sym = Ctx.getOrCreateSymbol(Name);
      Add = MCSymbolRefExpr::create(Sym, Ctx);
    } else {
      Add = MCConstantExpr::create((int)SymbolicOp.AddSymbol.Value, Ctx);
    }
  }

```
- **EN**: Implements logic around `Name`, `getOrCreateSymbol`, `create`; this block updates MC section or symbol state.
- **CN**: 围绕 `Name`, `getOrCreateSymbol`, `create` 实现具体逻辑；这一段更新 MC 节区或符号状态。

### Lines 97-107
```cpp
  const MCExpr *Sub = nullptr;
  if (SymbolicOp.SubtractSymbol.Present) {
      if (SymbolicOp.SubtractSymbol.Name) {
      StringRef Name(SymbolicOp.SubtractSymbol.Name);
      MCSymbol *Sym = Ctx.getOrCreateSymbol(Name);
      Sub = MCSymbolRefExpr::create(Sym, Ctx);
    } else {
      Sub = MCConstantExpr::create((int)SymbolicOp.SubtractSymbol.Value, Ctx);
    }
  }

```
- **EN**: Implements logic around `Name`, `getOrCreateSymbol`, `create`; this block updates MC section or symbol state.
- **CN**: 围绕 `Name`, `getOrCreateSymbol`, `create` 实现具体逻辑；这一段更新 MC 节区或符号状态。

### Lines 108-121
```cpp
  const MCExpr *Off = nullptr;
  if (SymbolicOp.Value != 0)
    Off = MCConstantExpr::create(SymbolicOp.Value, Ctx);

  const MCExpr *Expr;
  if (Sub) {
    const MCExpr *LHS;
    if (Add)
      LHS = MCBinaryExpr::createSub(Add, Sub, Ctx);
    else
      LHS = MCUnaryExpr::createMinus(Sub, Ctx);
    if (Off)
      Expr = MCBinaryExpr::createAdd(LHS, Off, Ctx);
    else
```
- **EN**: Implements logic around `create`, `createSub`, `createMinus`, `createAdd`; this block updates MC section or symbol state.
- **CN**: 围绕 `create`, `createSub`, `createMinus`, `createAdd` 实现具体逻辑；这一段更新 MC 节区或符号状态。

### Lines 122-134
```cpp
      Expr = LHS;
  } else if (Add) {
    if (Off)
      Expr = MCBinaryExpr::createAdd(Add, Off, Ctx);
    else
      Expr = Add;
  } else {
    if (Off)
      Expr = Off;
    else
      Expr = MCConstantExpr::create(0, Ctx);
  }

```
- **EN**: Implements logic around `createAdd`, `create`.
- **CN**: 围绕 `createAdd`, `create` 实现具体逻辑。

### Lines 135-142
```cpp
  Expr = RelInfo->createExprForCAPIVariantKind(Expr, SymbolicOp.VariantKind);
  if (!Expr)
    return false;

  MI.addOperand(MCOperand::createExpr(Expr));
  return true;
}

```
- **EN**: Implements logic around `createExprForCAPIVariantKind`, `addOperand`; this block parses assembly syntax or operands; updates MC section or symbol state; returns subsystem-specific computed results.
- **CN**: 围绕 `createExprForCAPIVariantKind`, `addOperand` 实现具体逻辑；这一段解析汇编语法或操作数，更新 MC 节区或符号状态，返回子系统相关的计算结果。

### Lines 143-156
```cpp
// This function tries to add a comment as to what is being referenced by a load
// instruction with the base register that is the Pc.  These can often be values
// in a literal pool near the Address of the instruction. The Address of the
// instruction and its immediate Value are used as a possible literal pool entry.
// The SymbolLookUp call back will return the name of a symbol referenced by the
// literal pool's entry if the referenced address is that of a symbol. Or it
// will return a pointer to a literal 'C' string if the referenced address of
// the literal pool's entry is an address into a section with C string literals.
// Or if the reference is to an Objective-C data structure it will return a
// specific reference type for it and a string.
void MCExternalSymbolizer::tryAddingPcLoadReferenceComment(raw_ostream &cStream,
                                                           int64_t Value,
                                                           uint64_t Address) {
  if (SymbolLookUp) {
```
- **EN**: Implements logic around `tryAddingPcLoadReferenceComment`; this block updates MC section or symbol state; returns subsystem-specific computed results.
- **CN**: 围绕 `tryAddingPcLoadReferenceComment` 实现具体逻辑；这一段更新 MC 节区或符号状态，返回子系统相关的计算结果。

### Lines 157-170
```cpp
    uint64_t ReferenceType = LLVMDisassembler_ReferenceType_In_PCrel_Load;
    const char *ReferenceName;
    (void)SymbolLookUp(DisInfo, Value, &ReferenceType, Address, &ReferenceName);
    if(ReferenceType == LLVMDisassembler_ReferenceType_Out_LitPool_SymAddr)
      cStream << "literal pool symbol address: " << ReferenceName;
    else if(ReferenceType ==
            LLVMDisassembler_ReferenceType_Out_LitPool_CstrAddr) {
      cStream << "literal pool for: \"";
      cStream.write_escaped(ReferenceName);
      cStream << "\"";
    }
    else if(ReferenceType ==
            LLVMDisassembler_ReferenceType_Out_Objc_CFString_Ref)
      cStream << "Objc cfstring ref: @\"" << ReferenceName << "\"";
```
- **EN**: Implements logic around `SymbolLookUp`, `write_escaped`; this block decodes machine-code bytes into symbolic instruction form; drives emission, layout, or binary encoding behavior; updates MC section or symbol state.
- **CN**: 围绕 `SymbolLookUp`, `write_escaped` 实现具体逻辑；这一段把机器码字节解码为符号化指令形式，驱动输出、布局或二进制编码行为，更新 MC 节区或符号状态。

### Lines 171-184
```cpp
    else if(ReferenceType ==
            LLVMDisassembler_ReferenceType_Out_Objc_Message)
      cStream << "Objc message: " << ReferenceName;
    else if(ReferenceType ==
            LLVMDisassembler_ReferenceType_Out_Objc_Message_Ref)
      cStream << "Objc message ref: " << ReferenceName;
    else if(ReferenceType ==
            LLVMDisassembler_ReferenceType_Out_Objc_Selector_Ref)
      cStream << "Objc selector ref: " << ReferenceName;
    else if(ReferenceType ==
            LLVMDisassembler_ReferenceType_Out_Objc_Class_Ref)
      cStream << "Objc class ref: " << ReferenceName;
  }
}
```
- **EN**: Introduces declarations for `ref:`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `ref:` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 185-192
```cpp

MCSymbolizer *
llvm::createMCSymbolizer(const Triple &TT, LLVMOpInfoCallback GetOpInfo,
                         LLVMSymbolLookupCallback SymbolLookUp, void *DisInfo,
                         MCContext *Ctx,
                         std::unique_ptr<MCRelocationInfo> &&RelInfo) {
  assert(Ctx && "No MCContext given for symbolic disassembly");

```
- **EN**: Implements logic around `createMCSymbolizer`, `assert`; this block handles relocation, fixup, or symbol-resolution work; updates MC section or symbol state.
- **CN**: 围绕 `createMCSymbolizer`, `assert` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，更新 MC 节区或符号状态。

### Lines 193-195
```cpp
  return new MCExternalSymbolizer(*Ctx, std::move(RelInfo), GetOpInfo,
                                  SymbolLookUp, DisInfo);
}
```
- **EN**: Implements logic around `MCExternalSymbolizer`; this block updates MC section or symbol state; returns subsystem-specific computed results.
- **CN**: 围绕 `MCExternalSymbolizer` 实现具体逻辑；这一段更新 MC 节区或符号状态，返回子系统相关的计算结果。

## Key Concepts / 关键概念

- **Disassembly / 反汇编**:
  - **EN**: Decodes raw bytes into MCInst objects and tries to recover symbolic references
  - **CN**: 将原始字节解码为 MCInst，并尝试恢复符号引用
- **Relocations and fixups / 重定位与 fixup**:
  - **EN**: Bridges symbolic references to concrete addresses or relocation records
  - **CN**: 把符号引用连接到具体地址或重定位记录
- **MC instruction model / MC 指令模型**:
  - **EN**: Uses MCInst/MCOperand abstractions to represent machine instructions generically
  - **CN**: 使用 MCInst/MCOperand 抽象以通用方式表示机器指令
- **Object layout / 目标文件布局**:
  - **EN**: Organizes fragments into sections and computes final offsets
  - **CN**: 把片段组织到节区中并计算最终偏移

## Dependencies / 依赖关系

- **Direct includes / 直接包含**: `llvm/MC/MCDisassembler/MCExternalSymbolizer.h`, `llvm/MC/MCContext.h`, `llvm/MC/MCExpr.h`, `llvm/MC/MCInst.h`, `llvm/MC/TargetRegistry.h`, `llvm/Support/raw_ostream.h`, `cstring`
- **LLVM subsystems / LLVM 子系统**: MC, Support
