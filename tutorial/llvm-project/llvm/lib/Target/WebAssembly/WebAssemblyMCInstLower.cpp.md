# WebAssemblyMCInstLower.cpp — Code Analysis / 代码分析

## Source / 来源

- **File**: `llvm/lib/Target/WebAssembly/WebAssemblyMCInstLower.cpp`
- **Repository**: `llvm-project`
- **Purpose (EN)**: This file contains code to lower WebAssembly MachineInstrs to their corresponding MCInst records.
- **Purpose (CN)**: 该文件位于 `llvm/lib/Target/WebAssembly/WebAssemblyMCInstLower.cpp`，主要负责 WebAssembly 后端的该后端的 MC 层支持。 文件内容以具体实现、辅助函数和后端决策逻辑为主。

## Line-by-Line Analysis / 逐行分析

### Lines 1-6

```cpp
// WebAssemblyMCInstLower.cpp - Convert WebAssembly MachineInstr to an MCInst //
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
```
- **EN**: Sets up the file banner, licensing information, and the initial context for the rest of the source file. It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这一段给出文件横幅、许可证信息以及后续源码部分的初始上下文。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

### Lines 7-12

```cpp
//===----------------------------------------------------------------------===//
///
/// \file
/// This file contains code to lower WebAssembly MachineInstrs to their
/// corresponding MCInst records.
///
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "\file". It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“\file”。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

### Lines 13-19

```cpp
//===----------------------------------------------------------------------===//

#include "WebAssemblyMCInstLower.h"
#include "MCTargetDesc/WebAssemblyMCAsmInfo.h"
#include "MCTargetDesc/WebAssemblyMCTargetDesc.h"
#include "MCTargetDesc/WebAssemblyMCTypeUtilities.h"
#include "TargetInfo/WebAssemblyTargetInfo.h"
```
- **EN**: Pulls in direct dependencies required by this MC layer support for the backend, so later declarations can reuse LLVM infrastructure and target-specific helpers. The logic interacts with LLVM's MC layer.
- **CN**: 这一段引入该该后端的 MC 层支持所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。 相关逻辑会与 LLVM 的 MC 层交互。

### Lines 20-26

```cpp
#include "Utils/WebAssemblyTypeUtilities.h"
#include "WebAssemblyAsmPrinter.h"
#include "WebAssemblyMachineFunctionInfo.h"
#include "WebAssemblyUtilities.h"
#include "llvm/ADT/APInt.h"
#include "llvm/ADT/SmallVector.h"
#include "llvm/BinaryFormat/Wasm.h"
```
- **EN**: Pulls in direct dependencies required by this MC layer support for the backend, so later declarations can reuse LLVM infrastructure and target-specific helpers.
- **CN**: 这一段引入该该后端的 MC 层支持所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。

### Lines 27-33

```cpp
#include "llvm/CodeGen/AsmPrinter.h"
#include "llvm/CodeGen/MachineFunction.h"
#include "llvm/CodeGen/MachineOperand.h"
#include "llvm/IR/Constants.h"
#include "llvm/MC/MCAsmInfo.h"
#include "llvm/MC/MCContext.h"
#include "llvm/MC/MCExpr.h"
```
- **EN**: Pulls in direct dependencies required by this MC layer support for the backend, so later declarations can reuse LLVM infrastructure and target-specific helpers. It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这一段引入该该后端的 MC 层支持所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

### Lines 34-71

```cpp
#include "llvm/MC/MCInst.h"
#include "llvm/MC/MCSymbolWasm.h"
#include "llvm/Support/ErrorHandling.h"
#include "llvm/Support/raw_ostream.h"

using namespace llvm;

// This disables the removal of registers when lowering into MC, as required
// by some current tests.
static cl::opt<bool>
    WasmKeepRegisters("wasm-keep-registers", cl::Hidden,
                      cl::desc("WebAssembly: output stack registers in"
                               " instruction output for test purposes only."),
                      cl::init(false));

static void removeRegisterOperands(const MachineInstr *MI, MCInst &OutMI);

MCSymbol *
WebAssemblyMCInstLower::GetGlobalAddressSymbol(const MachineOperand &MO) const {
  const GlobalValue *Global = MO.getGlobal();
  if (!isa<Function>(Global)) {
    auto *WasmSym = static_cast<MCSymbolWasm *>(Printer.getSymbol(Global));
    // If the symbol doesn't have an explicit WasmSymbolType yet and the
    // GlobalValue is actually a WebAssembly global, then ensure the symbol is a
    // WASM_SYMBOL_TYPE_GLOBAL.
    if (WebAssembly::isWasmVarAddressSpace(Global->getAddressSpace()) &&
        !WasmSym->getType()) {
      const MachineFunction &MF = *MO.getParent()->getParent()->getParent();
      const TargetMachine &TM = MF.getTarget();
      const Function &CurrentFunc = MF.getFunction();
      Type *GlobalVT = Global->getValueType();
      SmallVector<MVT, 1> VTs;
      computeLegalValueVTs(CurrentFunc, TM, GlobalVT, VTs);

      WebAssembly::wasmSymbolSetType(WasmSym, GlobalVT, VTs);
    }
    return WasmSym;
  }
```
- **EN**: Pulls in direct dependencies required by this MC layer support for the backend, so later declarations can reuse LLVM infrastructure and target-specific helpers. It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这一段引入该该后端的 MC 层支持所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

### Lines 72-109

```cpp

  const auto *FuncTy = cast<FunctionType>(Global->getValueType());
  const MachineFunction &MF = *MO.getParent()->getParent()->getParent();
  const TargetMachine &TM = MF.getTarget();
  const Function &CurrentFunc = MF.getFunction();

  SmallVector<MVT, 1> ResultMVTs;
  SmallVector<MVT, 4> ParamMVTs;
  const auto *const F = dyn_cast<Function>(Global);
  computeSignatureVTs(FuncTy, F, CurrentFunc, TM, ParamMVTs, ResultMVTs);
  auto Signature = signatureFromMVTs(Ctx, ResultMVTs, ParamMVTs);

  bool InvokeDetected = false;
  auto *WasmSym = Printer.getMCSymbolForFunction(F, Signature, InvokeDetected);
  WasmSym->setSignature(Signature);
  WasmSym->setType(wasm::WASM_SYMBOL_TYPE_FUNCTION);
  return WasmSym;
}

MCSymbol *WebAssemblyMCInstLower::GetExternalSymbolSymbol(
    const MachineOperand &MO) const {
  return Printer.getOrCreateWasmSymbol(MO.getSymbolName());
}

MCOperand WebAssemblyMCInstLower::lowerSymbolOperand(const MachineOperand &MO,
                                                     MCSymbol *Sym) const {
  auto Spec = WebAssembly::S_None;
  unsigned TargetFlags = MO.getTargetFlags();

  switch (TargetFlags) {
    case WebAssemblyII::MO_NO_FLAG:
      break;
    case WebAssemblyII::MO_GOT_TLS:
      Spec = WebAssembly::S_GOT_TLS;
      break;
    case WebAssemblyII::MO_GOT:
      Spec = WebAssembly::S_GOT;
      break;
```
- **EN**: Implements helper routine(s) `getValueType`, `getParent`, `getTarget` for this portion of the WebAssembly backend MC layer support for the backend. It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这里实现了 WebAssembly 后端该部分该后端的 MC 层支持所需的辅助例程 `getValueType`, `getParent`, `getTarget`。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

### Lines 110-147

```cpp
    case WebAssemblyII::MO_MEMORY_BASE_REL:
      Spec = WebAssembly::S_MBREL;
      break;
    case WebAssemblyII::MO_TLS_BASE_REL:
      Spec = WebAssembly::S_TLSREL;
      break;
    case WebAssemblyII::MO_TABLE_BASE_REL:
      Spec = WebAssembly::S_TBREL;
      break;
    default:
      llvm_unreachable("Unknown target flag on GV operand");
  }

  const MCExpr *Expr = MCSymbolRefExpr::create(Sym, Spec, Ctx);

  if (MO.getOffset() != 0) {
    const auto *WasmSym = static_cast<const MCSymbolWasm *>(Sym);
    if (TargetFlags == WebAssemblyII::MO_GOT)
      report_fatal_error("GOT symbol references do not support offsets");
    if (WasmSym->isFunction())
      report_fatal_error("Function addresses with offsets not supported");
    if (WasmSym->isGlobal())
      report_fatal_error("Global indexes with offsets not supported");
    if (WasmSym->isTag())
      report_fatal_error("Tag indexes with offsets not supported");
    if (WasmSym->isTable())
      report_fatal_error("Table indexes with offsets not supported");

    Expr = MCBinaryExpr::createAdd(
        Expr, MCConstantExpr::create(MO.getOffset(), Ctx), Ctx);
  }

  return MCOperand::createExpr(Expr);
}

MCOperand WebAssemblyMCInstLower::lowerTypeIndexOperand(
    SmallVectorImpl<wasm::ValType> &&Returns,
    SmallVectorImpl<wasm::ValType> &&Params) const {
```
- **EN**: Implements helper routine(s) `llvm_unreachable`, `create`, `getOffset` for this portion of the WebAssembly backend MC layer support for the backend. The logic interacts with LLVM's MC layer.
- **CN**: 这里实现了 WebAssembly 后端该部分该后端的 MC 层支持所需的辅助例程 `llvm_unreachable`, `create`, `getOffset`。 相关逻辑会与 LLVM 的 MC 层交互。

### Lines 148-185

```cpp
  auto Signature = Ctx.createWasmSignature();
  Signature->Returns = std::move(Returns);
  Signature->Params = std::move(Params);
  auto *Sym =
      static_cast<MCSymbolWasm *>(Printer.createTempSymbol("typeindex"));
  Sym->setSignature(Signature);
  Sym->setType(wasm::WASM_SYMBOL_TYPE_FUNCTION);
  const MCExpr *Expr =
      MCSymbolRefExpr::create(Sym, WebAssembly::S_TYPEINDEX, Ctx);
  return MCOperand::createExpr(Expr);
}

MCOperand
WebAssemblyMCInstLower::lowerEncodedFunctionSignature(const APInt &Sig) const {
  // For APInt a word is 64 bits on all architectures, see definition in APInt.h
  auto NumWords = Sig.getNumWords();
  SmallVector<wasm::ValType, 4> Params;
  SmallVector<wasm::ValType, 2> Returns;

  int Idx = NumWords;
  auto GetWord = [&Idx, &Sig]() {
    Idx--;
    return Sig.extractBitsAsZExtValue(64, 64 * Idx);
  };
  // Annoying special case: if getSignificantBits() <= 64 then InstrEmitter will
  // emit an Imm instead of a CImm. It simplifies WebAssemblyMCInstLower if we
  // always emit a CImm. So xor NParams with 0x7ffffff to ensure
  // getSignificantBits() > 64
  // See encodeFunctionSignature in WebAssemblyISelDAGtoDAG.cpp
  int NReturns = GetWord() ^ 0x7ffffff;
  for (int I = 0; I < NReturns; I++) {
    Returns.push_back(static_cast<wasm::ValType>(GetWord()));
  }
  int NParams = GetWord();
  for (int I = 0; I < NParams; I++) {
    Params.push_back(static_cast<wasm::ValType>(GetWord()));
  }
  return lowerTypeIndexOperand(std::move(Returns), std::move(Params));
```
- **EN**: Implements helper routine(s) `createWasmSignature`, `move`, `createTempSymbol` for this portion of the WebAssembly backend MC layer support for the backend. The logic interacts with LLVM's MC layer.
- **CN**: 这里实现了 WebAssembly 后端该部分该后端的 MC 层支持所需的辅助例程 `createWasmSignature`, `move`, `createTempSymbol`。 相关逻辑会与 LLVM 的 MC 层交互。

### Lines 186-223

```cpp
}

static void getFunctionReturns(const MachineInstr *MI,
                               SmallVectorImpl<wasm::ValType> &Returns) {
  const Function &F = MI->getMF()->getFunction();
  const TargetMachine &TM = MI->getMF()->getTarget();
  Type *RetTy = F.getReturnType();
  SmallVector<MVT, 4> CallerRetTys;
  computeLegalValueVTs(F, TM, RetTy, CallerRetTys);
  valTypesFromMVTs(CallerRetTys, Returns);
}

void WebAssemblyMCInstLower::lower(const MachineInstr *MI,
                                   MCInst &OutMI) const {
  OutMI.setOpcode(MI->getOpcode());

  const MCInstrDesc &Desc = MI->getDesc();
  unsigned NumVariadicDefs = MI->getNumExplicitDefs() - Desc.getNumDefs();
  const MachineFunction *MF = MI->getMF();
  const auto &TLI =
      *MF->getSubtarget<WebAssemblySubtarget>().getTargetLowering();
  wasm::ValType PtrTy = TLI.getPointerTy(MF->getDataLayout()) == MVT::i32
                            ? wasm::ValType::I32
                            : wasm::ValType::I64;

  for (unsigned I = 0, E = MI->getNumOperands(); I != E; ++I) {
    const MachineOperand &MO = MI->getOperand(I);

    MCOperand MCOp;
    switch (MO.getType()) {
    default:
      MI->print(errs());
      llvm_unreachable("unknown operand type");
    case MachineOperand::MO_MachineBasicBlock:
      MI->print(errs());
      llvm_unreachable("MachineBasicBlock operand should have been rewritten");
    case MachineOperand::MO_Register: {
      // Ignore all implicit register operands.
```
- **EN**: Implements helper routine(s) `getFunctionReturns`, `getMF`, `getFunction` for this portion of the WebAssembly backend MC layer support for the backend. It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这里实现了 WebAssembly 后端该部分该后端的 MC 层支持所需的辅助例程 `getFunctionReturns`, `getMF`, `getFunction`。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

### Lines 224-261

```cpp
      if (MO.isImplicit())
        continue;
      const WebAssemblyFunctionInfo &MFI =
          *MI->getParent()->getParent()->getInfo<WebAssemblyFunctionInfo>();
      unsigned WAReg = MFI.getWAReg(MO.getReg());
      MCOp = MCOperand::createReg(WAReg);
      break;
    }
    case llvm::MachineOperand::MO_CImmediate: {
      // Lower type index placeholder for ref.test
      // Currently this is the only way that CImmediates show up so panic if we
      // get confused.
      unsigned DescIndex = I - NumVariadicDefs;
      assert(DescIndex < Desc.NumOperands && "unexpected CImmediate operand");
      auto Operands = Desc.operands();
      const MCOperandInfo &Info = Operands[DescIndex];
      assert(Info.OperandType == WebAssembly::OPERAND_TYPEINDEX &&
             "unexpected CImmediate operand");
      (void)Info;
      MCOp = lowerEncodedFunctionSignature(MO.getCImm()->getValue());
      break;
    }
    case MachineOperand::MO_Immediate: {
      unsigned DescIndex = I - NumVariadicDefs;
      if (DescIndex < Desc.NumOperands) {
        auto Operands = Desc.operands();
        const MCOperandInfo &Info = Operands[DescIndex];
        // Replace type index placeholder with actual type index. The type index
        // placeholders are Immediates and have an operand type of
        // OPERAND_TYPEINDEX or OPERAND_SIGNATURE.
        if (Info.OperandType == WebAssembly::OPERAND_TYPEINDEX) {
          // Lower type index placeholder for a CALL_INDIRECT instruction
          SmallVector<wasm::ValType, 4> Returns;
          SmallVector<wasm::ValType, 4> Params;

          const MachineRegisterInfo &MRI =
              MI->getParent()->getParent()->getRegInfo();
          for (const MachineOperand &MO : MI->defs())
```
- **EN**: Implements helper routine(s) `isImplicit`, `getParent`, `getWAReg` for this portion of the WebAssembly backend MC layer support for the backend. It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这里实现了 WebAssembly 后端该部分该后端的 MC 层支持所需的辅助例程 `isImplicit`, `getParent`, `getWAReg`。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

### Lines 262-299

```cpp
            Returns.push_back(WebAssembly::regClassToValType(
                MRI.getRegClass(MO.getReg())->getID()));
          for (const MachineOperand &MO : MI->explicit_uses())
            if (MO.isReg())
              Params.push_back(WebAssembly::regClassToValType(
                  MRI.getRegClass(MO.getReg())->getID()));

          // call_indirect instructions have a callee operand at the end which
          // doesn't count as a param.
          if (WebAssembly::isCallIndirect(MI->getOpcode()))
            Params.pop_back();

          // return_call_indirect instructions have the return type of the
          // caller
          if (MI->getOpcode() == WebAssembly::RET_CALL_INDIRECT)
            getFunctionReturns(MI, Returns);

          MCOp = lowerTypeIndexOperand(std::move(Returns), std::move(Params));
          break;
        }
        if (Info.OperandType == WebAssembly::OPERAND_SIGNATURE) {
          // Lower type index placeholder for blocks
          auto BT = static_cast<WebAssembly::BlockType>(MO.getImm());
          assert(BT != WebAssembly::BlockType::Invalid);
          if (BT == WebAssembly::BlockType::Multivalue) {
            SmallVector<wasm::ValType, 2> Returns;
            // Multivalue blocks are emitted in two cases:
            // 1. When the blocks will never be exited and are at the ends of
            //    functions (see
            //    WebAssemblyCFGStackify::fixEndsAtEndOfFunction). In this case
            //    the exact multivalue signature can always be inferred from the
            //    return type of the parent function.
            // 2. (catch_ref ...) clause in try_table instruction. Currently all
            //    tags we support (cpp_exception and c_longjmp) throws a single
            //    pointer, so the multivalue signature for this case will be
            //    (ptr, exnref). Having MO_CATCH_BLOCK_SIG target flags means
            //    this is a destination of a catch_ref.
            if (MO.getTargetFlags() == WebAssemblyII::MO_CATCH_BLOCK_SIG) {
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "call_indirect instructions have a callee operand at the end which". It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“call_indirect instructions have a callee operand at the end which”。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

### Lines 300-337

```cpp
              Returns = {PtrTy, wasm::ValType::EXNREF};
            } else
              getFunctionReturns(MI, Returns);
            MCOp = lowerTypeIndexOperand(std::move(Returns),
                                         SmallVector<wasm::ValType, 4>());
            break;
          }
        }
      }
      MCOp = MCOperand::createImm(MO.getImm());
      break;
    }
    case MachineOperand::MO_FPImmediate: {
      const ConstantFP *Imm = MO.getFPImm();
      const uint64_t BitPattern =
          Imm->getValueAPF().bitcastToAPInt().getZExtValue();
      if (Imm->getType()->isFloatTy())
        MCOp = MCOperand::createSFPImm(static_cast<uint32_t>(BitPattern));
      else if (Imm->getType()->isDoubleTy())
        MCOp = MCOperand::createDFPImm(BitPattern);
      else
        llvm_unreachable("unknown floating point immediate type");
      break;
    }
    case MachineOperand::MO_GlobalAddress:
      MCOp = lowerSymbolOperand(MO, GetGlobalAddressSymbol(MO));
      break;
    case MachineOperand::MO_ExternalSymbol:
      MCOp = lowerSymbolOperand(MO, GetExternalSymbolSymbol(MO));
      break;
    case MachineOperand::MO_MCSymbol:
      assert(MO.getTargetFlags() == 0 &&
             "WebAssembly does not use target flags on MCSymbol");
      MCOp = lowerSymbolOperand(MO, MO.getMCSymbol());
      break;
    }

    OutMI.addOperand(MCOp);
```
- **EN**: Implements helper routine(s) `getFunctionReturns`, `lowerTypeIndexOperand`, `move` for this portion of the WebAssembly backend MC layer support for the backend. It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这里实现了 WebAssembly 后端该部分该后端的 MC 层支持所需的辅助例程 `getFunctionReturns`, `lowerTypeIndexOperand`, `move`。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

### Lines 338-375

```cpp
  }

  if (!WasmKeepRegisters)
    removeRegisterOperands(MI, OutMI);
  else if (Desc.variadicOpsAreDefs())
    OutMI.insert(OutMI.begin(), MCOperand::createImm(MI->getNumExplicitDefs()));
}

static void removeRegisterOperands(const MachineInstr *MI, MCInst &OutMI) {
  // Remove all uses of stackified registers to bring the instruction format
  // into its final stack form used thruout MC, and transition opcodes to
  // their _S variant.
  // We do this separate from the above code that still may need these
  // registers for e.g. call_indirect signatures.
  // See comments in lib/Target/WebAssembly/WebAssemblyInstrFormats.td for
  // details.
  // TODO: the code above creates new registers which are then removed here.
  // That code could be slightly simplified by not doing that, though maybe
  // it is simpler conceptually to keep the code above in "register mode"
  // until this transition point.
  // FIXME: we are not processing inline assembly, which contains register
  // operands, because it is used by later target generic code.
  if (MI->isDebugInstr() || MI->isLabel() || MI->isInlineAsm())
    return;

  // Transform to _S instruction.
  auto RegOpcode = OutMI.getOpcode();
  auto StackOpcode = WebAssembly::getStackOpcode(RegOpcode);
  assert(StackOpcode != -1 && "Failed to stackify instruction");
  OutMI.setOpcode(StackOpcode);

  // Remove register operands.
  for (auto I = OutMI.getNumOperands(); I; --I) {
    auto &MO = OutMI.getOperand(I - 1);
    if (MO.isReg()) {
      OutMI.erase(&MO);
    }
  }
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Remove all uses of stackified registers to bring the instruction format". It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Remove all uses of stackified registers to bring the instruction format”。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

### Lines 376-376

```cpp
}
```
- **EN**: Continues the WebAssembly backend MC layer support for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 WebAssembly 后端的该后端的 MC 层支持，补充该行区间所需的声明、辅助逻辑或实现细节。

## Key Concepts / 关键概念

- WebAssembly backend integration / WebAssembly 后端集成
- MC layer support for the backend / 该后端的 MC 层支持
- Register modeling / 寄存器建模
- Stack frame management / 栈帧管理
- Subtarget features / 子目标特性
- Assembly or MC emission / 汇编或 MC 发射
- Instruction selection or opcode handling / 指令选择或操作码处理
- Target machine configuration / 目标机器配置

## Dependencies / 依赖关系

### Direct Includes / 直接包含

- `WebAssemblyMCInstLower.h`
- `MCTargetDesc/WebAssemblyMCAsmInfo.h`
- `MCTargetDesc/WebAssemblyMCTargetDesc.h`
- `MCTargetDesc/WebAssemblyMCTypeUtilities.h`
- `TargetInfo/WebAssemblyTargetInfo.h`
- `Utils/WebAssemblyTypeUtilities.h`
- `WebAssemblyAsmPrinter.h`
- `WebAssemblyMachineFunctionInfo.h`
- `WebAssemblyUtilities.h`
- `llvm/ADT/APInt.h`
- `llvm/ADT/SmallVector.h`
- `llvm/BinaryFormat/Wasm.h`
- `llvm/CodeGen/AsmPrinter.h`
- `llvm/CodeGen/MachineFunction.h`
- `llvm/CodeGen/MachineOperand.h`
- `llvm/IR/Constants.h`
- `llvm/MC/MCAsmInfo.h`
- `llvm/MC/MCContext.h`
- `llvm/MC/MCExpr.h`
- `llvm/MC/MCInst.h`
- `llvm/MC/MCSymbolWasm.h`
- `llvm/Support/ErrorHandling.h`
- `llvm/Support/raw_ostream.h`

### Important Collaborators / 重要协作组件

- LLVM CodeGen layer / LLVM CodeGen 层
- LLVM MC layer / LLVM MC 层
- LLVM Support utilities / LLVM Support 工具层
- WebAssembly target-specific helpers / WebAssembly 目标专用辅助组件
