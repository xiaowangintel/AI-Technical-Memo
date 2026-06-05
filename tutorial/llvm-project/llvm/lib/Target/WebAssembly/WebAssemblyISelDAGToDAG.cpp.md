# WebAssemblyISelDAGToDAG.cpp — Code Analysis / 代码分析

## Source / 来源

- **File**: `llvm/lib/Target/WebAssembly/WebAssemblyISelDAGToDAG.cpp`
- **Repository**: `llvm-project`
- **Purpose (EN)**: This file defines an instruction selector for the WebAssembly target.
- **Purpose (CN)**: 该文件位于 `llvm/lib/Target/WebAssembly/WebAssemblyISelDAGToDAG.cpp`，主要负责 WebAssembly 后端的DAG 到 DAG 指令选择。 文件内容以具体实现、辅助函数和后端决策逻辑为主。

## Line-by-Line Analysis / 逐行分析

### Lines 1-6

```cpp
//- WebAssemblyISelDAGToDAG.cpp - A dag to dag inst selector for WebAssembly -//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
```
- **EN**: Sets up the file banner, licensing information, and the initial context for the rest of the source file.
- **CN**: 这一段给出文件横幅、许可证信息以及后续源码部分的初始上下文。

### Lines 7-11

```cpp
//===----------------------------------------------------------------------===//
///
/// \file
/// This file defines an instruction selector for the WebAssembly target.
///
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "\file".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“\file”。

### Lines 12-18

```cpp
//===----------------------------------------------------------------------===//

#include "MCTargetDesc/WebAssemblyMCTargetDesc.h"
#include "WebAssembly.h"
#include "WebAssemblyISelLowering.h"
#include "WebAssemblyTargetMachine.h"
#include "WebAssemblyUtilities.h"
```
- **EN**: Pulls in direct dependencies required by this DAG-to-DAG instruction selection, so later declarations can reuse LLVM infrastructure and target-specific helpers.
- **CN**: 这一段引入该DAG 到 DAG 指令选择所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。

### Lines 19-25

```cpp
#include "llvm/BinaryFormat/Wasm.h"
#include "llvm/CodeGen/MachineFrameInfo.h"
#include "llvm/CodeGen/SelectionDAGISel.h"
#include "llvm/CodeGen/WasmEHInfo.h"
#include "llvm/IR/DiagnosticInfo.h"
#include "llvm/IR/Function.h" // To access function attributes.
#include "llvm/IR/IntrinsicsWebAssembly.h"
```
- **EN**: Pulls in direct dependencies required by this DAG-to-DAG instruction selection, so later declarations can reuse LLVM infrastructure and target-specific helpers. This range works in the SelectionDAG-based lowering pipeline.
- **CN**: 这一段引入该DAG 到 DAG 指令选择所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。 这一段工作在基于 SelectionDAG 的 lowering 流水线中。

### Lines 26-32

```cpp
#include "llvm/MC/MCSymbolWasm.h"
#include "llvm/Support/Debug.h"
#include "llvm/Support/KnownBits.h"
#include "llvm/Support/raw_ostream.h"

using namespace llvm;
```
- **EN**: Pulls in direct dependencies required by this DAG-to-DAG instruction selection, so later declarations can reuse LLVM infrastructure and target-specific helpers.
- **CN**: 这一段引入该DAG 到 DAG 指令选择所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。

### Lines 33-35

```cpp
#define DEBUG_TYPE "wasm-isel"
#define PASS_NAME "WebAssembly Instruction Selection"
```
- **EN**: Defines or closes the header guard so the declarations in this header are only processed once per translation unit.
- **CN**: 这里定义或结束头文件保护宏，确保该头文件中的声明在一个编译单元内只会被处理一次。

### Lines 36-73

```cpp
//===--------------------------------------------------------------------===//
/// WebAssembly-specific code to select WebAssembly machine instructions for
/// SelectionDAG operations.
///
namespace {
class WebAssemblyDAGToDAGISel final : public SelectionDAGISel {
  /// Keep a pointer to the WebAssemblySubtarget around so that we can make the
  /// right decision when generating code for different targets.
  const WebAssemblySubtarget *Subtarget;

public:
  WebAssemblyDAGToDAGISel() = delete;

  WebAssemblyDAGToDAGISel(WebAssemblyTargetMachine &TM,
                          CodeGenOptLevel OptLevel)
      : SelectionDAGISel(TM, OptLevel), Subtarget(nullptr) {}

  bool runOnMachineFunction(MachineFunction &MF) override {
    LLVM_DEBUG(dbgs() << "********** ISelDAGToDAG **********\n"
                         "********** Function: "
                      << MF.getName() << '\n');

    Subtarget = &MF.getSubtarget<WebAssemblySubtarget>();

    return SelectionDAGISel::runOnMachineFunction(MF);
  }

  void PreprocessISelDAG() override;

  void Select(SDNode *Node) override;

  bool SelectInlineAsmMemoryOperand(const SDValue &Op,
                                    InlineAsm::ConstraintCode ConstraintID,
                                    std::vector<SDValue> &OutOps) override;

  bool SelectAddrOperands32(SDValue Op, SDValue &Offset, SDValue &Addr);
  bool SelectAddrOperands64(SDValue Op, SDValue &Offset, SDValue &Addr);
  bool SelectAtomicAddrOperands(SDNode *Op, SDValue N, SDValue &Offset,
```
- **EN**: Declares a backend-facing type `WebAssemblyDAGToDAGISel`, `SelectionDAGISel`, `Subtarget` and outlines the API or state that nearby code will rely on. This range works in the SelectionDAG-based lowering pipeline.
- **CN**: 这里声明面向后端的类型 `WebAssemblyDAGToDAGISel`, `SelectionDAGISel`, `Subtarget`，并勾勒出周边代码会依赖的接口或状态。 这一段工作在基于 SelectionDAG 的 lowering 流水线中。

### Lines 74-80

```cpp
                                SDValue &Addr, SDValue &Order, bool Is64);
  bool SelectAtomicAddrOperands32(SDNode *Op, SDValue N, SDValue &Offset,
                                  SDValue &Addr, SDValue &Order);
  bool SelectAtomicAddrOperands64(SDNode *Op, SDValue N, SDValue &Offset,
                                  SDValue &Addr, SDValue &Order);

// Include the pieces autogenerated from the target description.
```
- **EN**: Declares function entry points including `SelectAtomicAddrOperands32`, `SelectAtomicAddrOperands64` that other backend components call later. This range works in the SelectionDAG-based lowering pipeline.
- **CN**: 这里声明后续其他后端组件会调用的函数入口，包括 `SelectAtomicAddrOperands32`, `SelectAtomicAddrOperands64`。 这一段工作在基于 SelectionDAG 的 lowering 流水线中。

### Lines 81-91

```cpp
#include "WebAssemblyGenDAGISel.inc"

private:
  // add select functions here...

  bool SelectAddrOperands(MVT AddrType, unsigned ConstOpc, SDValue Op,
                          SDValue &Offset, SDValue &Addr);
  bool SelectAddrAddOperands(MVT OffsetType, SDValue N, SDValue &Offset,
                             SDValue &Addr);
};
```
- **EN**: Pulls in direct dependencies required by this DAG-to-DAG instruction selection, so later declarations can reuse LLVM infrastructure and target-specific helpers. This range works in the SelectionDAG-based lowering pipeline.
- **CN**: 这一段引入该DAG 到 DAG 指令选择所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。 这一段工作在基于 SelectionDAG 的 lowering 流水线中。

### Lines 92-129

```cpp
class WebAssemblyDAGToDAGISelLegacy : public SelectionDAGISelLegacy {
public:
  static char ID;
  explicit WebAssemblyDAGToDAGISelLegacy(WebAssemblyTargetMachine &TM,
                                         CodeGenOptLevel OptLevel)
      : SelectionDAGISelLegacy(
            ID, std::make_unique<WebAssemblyDAGToDAGISel>(TM, OptLevel)) {}
};
} // end anonymous namespace

char WebAssemblyDAGToDAGISelLegacy::ID;

INITIALIZE_PASS(WebAssemblyDAGToDAGISelLegacy, DEBUG_TYPE, PASS_NAME, false,
                false)

void WebAssemblyDAGToDAGISel::PreprocessISelDAG() {
  // Stack objects that should be allocated to locals are hoisted to WebAssembly
  // locals when they are first used.  However for those without uses, we hoist
  // them here.  It would be nice if there were some hook to do this when they
  // are added to the MachineFrameInfo, but that's not the case right now.
  MachineFrameInfo &FrameInfo = MF->getFrameInfo();
  for (int Idx = 0; Idx < FrameInfo.getObjectIndexEnd(); Idx++)
    WebAssemblyFrameLowering::getLocalForStackObject(*MF, Idx);

  SelectionDAGISel::PreprocessISelDAG();
}

static SDValue getTagSymNode(int Tag, SelectionDAG *DAG) {
  assert(Tag == WebAssembly::CPP_EXCEPTION || Tag == WebAssembly::C_LONGJMP);
  auto &MF = DAG->getMachineFunction();
  const auto &TLI = DAG->getTargetLoweringInfo();
  MVT PtrVT = TLI.getPointerTy(DAG->getDataLayout());
  const char *SymName = Tag == WebAssembly::CPP_EXCEPTION
                            ? MF.createExternalSymbolName("__cpp_exception")
                            : MF.createExternalSymbolName("__c_longjmp");
  return DAG->getTargetExternalSymbol(SymName, PtrVT);
}
```
- **EN**: Declares a backend-facing type `WebAssemblyDAGToDAGISelLegacy`, `SelectionDAGISelLegacy`, `INITIALIZE_PASS` and outlines the API or state that nearby code will rely on. This range works in the SelectionDAG-based lowering pipeline.
- **CN**: 这里声明面向后端的类型 `WebAssemblyDAGToDAGISelLegacy`, `SelectionDAGISelLegacy`, `INITIALIZE_PASS`，并勾勒出周边代码会依赖的接口或状态。 这一段工作在基于 SelectionDAG 的 lowering 流水线中。

### Lines 130-167

```cpp
static APInt encodeFunctionSignature(SelectionDAG *DAG, SDLoc &DL,
                                     SmallVector<MVT, 4> &Returns,
                                     SmallVector<MVT, 4> &Params) {
  auto toWasmValType = [](MVT VT) {
    if (VT == MVT::i32) {
      return wasm::ValType::I32;
    }
    if (VT == MVT::i64) {
      return wasm::ValType::I64;
    }
    if (VT == MVT::f32) {
      return wasm::ValType::F32;
    }
    if (VT == MVT::f64) {
      return wasm::ValType::F64;
    }
    if (VT == MVT::externref) {
      return wasm::ValType::EXTERNREF;
    }
    if (VT == MVT::funcref) {
      return wasm::ValType::FUNCREF;
    }
    if (VT == MVT::exnref) {
      return wasm::ValType::EXNREF;
    }
    LLVM_DEBUG(errs() << "Unhandled type for llvm.wasm.ref.test.func: " << VT
                      << "\n");
    llvm_unreachable("Unhandled type for llvm.wasm.ref.test.func");
  };
  auto NParams = Params.size();
  auto NReturns = Returns.size();
  auto BitWidth = (NParams + NReturns + 2) * 64;
  auto Sig = APInt(BitWidth, 0);

  // Annoying special case: if getSignificantBits() <= 64 then InstrEmitter will
  // emit an Imm instead of a CImm. It simplifies WebAssemblyMCInstLower if we
  // always emit a CImm. So xor NParams with 0x7ffffff to ensure
  // getSignificantBits() > 64
```
- **EN**: Implements helper routine(s) `encodeFunctionSignature`, `errs`, `llvm_unreachable` for this portion of the WebAssembly backend DAG-to-DAG instruction selection. This range works in the SelectionDAG-based lowering pipeline.
- **CN**: 这里实现了 WebAssembly 后端该部分DAG 到 DAG 指令选择所需的辅助例程 `encodeFunctionSignature`, `errs`, `llvm_unreachable`。 这一段工作在基于 SelectionDAG 的 lowering 流水线中。

### Lines 168-205

```cpp
  Sig |= NReturns ^ 0x7ffffff;
  for (auto &Return : Returns) {
    auto V = toWasmValType(Return);
    Sig <<= 64;
    Sig |= (int64_t)V;
  }
  Sig <<= 64;
  Sig |= NParams;
  for (auto &Param : Params) {
    auto V = toWasmValType(Param);
    Sig <<= 64;
    Sig |= (int64_t)V;
  }
  return Sig;
}

static unsigned getWebAssemblyMemoryOrder(AtomicOrdering Ordering) {
  unsigned OrderVal = wasm::WASM_MEM_ORDER_SEQ_CST;
  switch (Ordering) {
  case AtomicOrdering::Unordered:
  case AtomicOrdering::Monotonic:
  case AtomicOrdering::Acquire:
  case AtomicOrdering::Release:
  case AtomicOrdering::AcquireRelease:
    OrderVal = wasm::WASM_MEM_ORDER_ACQ_REL;
    break;
  case AtomicOrdering::SequentiallyConsistent:
    OrderVal = wasm::WASM_MEM_ORDER_SEQ_CST;
    break;
  default:
    llvm_unreachable("Invalid atomic ordering");
  }
  return OrderVal;
}

void WebAssemblyDAGToDAGISel::Select(SDNode *Node) {
  // If we have a custom node, we already have selected!
  if (Node->isMachineOpcode()) {
```
- **EN**: Implements helper routine(s) `toWasmValType`, `getWebAssemblyMemoryOrder`, `llvm_unreachable` for this portion of the WebAssembly backend DAG-to-DAG instruction selection.
- **CN**: 这里实现了 WebAssembly 后端该部分DAG 到 DAG 指令选择所需的辅助例程 `toWasmValType`, `getWebAssemblyMemoryOrder`, `llvm_unreachable`。

### Lines 206-243

```cpp
    LLVM_DEBUG(errs() << "== "; Node->dump(CurDAG); errs() << "\n");
    Node->setNodeId(-1);
    return;
  }

  MVT PtrVT = TLI->getPointerTy(CurDAG->getDataLayout());
  auto GlobalGetIns = PtrVT == MVT::i64 ? WebAssembly::GLOBAL_GET_I64
                                        : WebAssembly::GLOBAL_GET_I32;

  SDLoc DL(Node);
  MachineFunction &MF = CurDAG->getMachineFunction();
  switch (Node->getOpcode()) {
  case ISD::ATOMIC_FENCE: {
    if (!MF.getSubtarget<WebAssemblySubtarget>().hasAtomics())
      break;

    uint64_t SyncScopeID = Node->getConstantOperandVal(2);
    MachineSDNode *Fence = nullptr;
    switch (SyncScopeID) {
    case SyncScope::SingleThread:
      // We lower a single-thread fence to a pseudo compiler barrier instruction
      // preventing instruction reordering. This will not be emitted in final
      // binary.
      Fence = CurDAG->getMachineNode(WebAssembly::COMPILER_FENCE,
                                     DL,                 // debug loc
                                     MVT::Other,         // outchain type
                                     Node->getOperand(0) // inchain
      );
      break;
    case SyncScope::System: {
      unsigned Order = wasm::WASM_MEM_ORDER_SEQ_CST;
      if (MF.getSubtarget<WebAssemblySubtarget>().hasRelaxedAtomics()) {
        auto Ordering =
            static_cast<AtomicOrdering>(Node->getConstantOperandVal(1));
        Order = getWebAssemblyMemoryOrder(Ordering);
      }
      Fence = CurDAG->getMachineNode(
          WebAssembly::ATOMIC_FENCE,
```
- **EN**: Implements helper routine(s) `errs`, `dump`, `setNodeId` for this portion of the WebAssembly backend DAG-to-DAG instruction selection. This range works in the SelectionDAG-based lowering pipeline.
- **CN**: 这里实现了 WebAssembly 后端该部分DAG 到 DAG 指令选择所需的辅助例程 `errs`, `dump`, `setNodeId`。 这一段工作在基于 SelectionDAG 的 lowering 流水线中。

### Lines 244-281

```cpp
          DL,                                             // debug loc
          MVT::Other,                                     // outchain type
          CurDAG->getTargetConstant(Order, DL, MVT::i32), // order
          Node->getOperand(0)                             // inchain
      );
      break;
    }
    default:
      llvm_unreachable("Unknown scope!");
    }

    ReplaceNode(Node, Fence);
    CurDAG->RemoveDeadNode(Node);
    return;
  }

  case ISD::INTRINSIC_WO_CHAIN: {
    unsigned IntNo = Node->getConstantOperandVal(0);
    switch (IntNo) {
    case Intrinsic::wasm_tls_size: {
      MachineSDNode *TLSSize = CurDAG->getMachineNode(
          GlobalGetIns, DL, PtrVT,
          CurDAG->getTargetExternalSymbol("__tls_size", PtrVT));
      ReplaceNode(Node, TLSSize);
      return;
    }

    case Intrinsic::wasm_tls_align: {
      MachineSDNode *TLSAlign = CurDAG->getMachineNode(
          GlobalGetIns, DL, PtrVT,
          CurDAG->getTargetExternalSymbol("__tls_align", PtrVT));
      ReplaceNode(Node, TLSAlign);
      return;
    }
    case Intrinsic::wasm_ref_test_func: {
      // First emit the TABLE_GET instruction to convert function pointer ==>
      // funcref
      MachineFunction &MF = CurDAG->getMachineFunction();
```
- **EN**: Implements helper routine(s) `getTargetConstant`, `getOperand`, `llvm_unreachable` for this portion of the WebAssembly backend DAG-to-DAG instruction selection. This range works in the SelectionDAG-based lowering pipeline.
- **CN**: 这里实现了 WebAssembly 后端该部分DAG 到 DAG 指令选择所需的辅助例程 `getTargetConstant`, `getOperand`, `llvm_unreachable`。 这一段工作在基于 SelectionDAG 的 lowering 流水线中。

### Lines 282-319

```cpp
      auto PtrVT = MVT::getIntegerVT(MF.getDataLayout().getPointerSizeInBits());
      MCSymbol *Table = WebAssembly::getOrCreateFunctionTableSymbol(
          MF.getContext(), Subtarget);
      SDValue TableSym = CurDAG->getMCSymbol(Table, PtrVT);
      SDValue FuncPtr = Node->getOperand(1);
      if (Subtarget->hasAddr64() && FuncPtr.getValueType() == MVT::i64) {
        // table.get expects an i32 but on 64 bit platforms the function pointer
        // is an i64. In that case, i32.wrap_i64 to convert.
        FuncPtr = SDValue(CurDAG->getMachineNode(WebAssembly::I32_WRAP_I64, DL,
                                                 MVT::i32, FuncPtr),
                          0);
      }
      SDValue FuncRef =
          SDValue(CurDAG->getMachineNode(WebAssembly::TABLE_GET_FUNCREF, DL,
                                         MVT::funcref, TableSym, FuncPtr),
                  0);

      // Encode the signature information into the type index placeholder.
      // This gets decoded and converted into the actual type signature in
      // WebAssemblyMCInstLower.cpp.
      SmallVector<MVT, 4> Params;
      SmallVector<MVT, 4> Returns;

      bool IsParam = false;
      // Operand 0 is the return register, Operand 1 is the function pointer.
      // The remaining operands encode the type of the function we are testing
      // for.
      for (unsigned I = 2, E = Node->getNumOperands(); I < E; ++I) {
        MVT VT = Node->getOperand(I).getValueType().getSimpleVT();
        if (VT == MVT::Untyped) {
          IsParam = true;
          continue;
        }
        if (IsParam) {
          Params.push_back(VT);
        } else {
          Returns.push_back(VT);
        }
```
- **EN**: Implements helper routine(s) `getIntegerVT`, `getDataLayout`, `getPointerSizeInBits` for this portion of the WebAssembly backend DAG-to-DAG instruction selection. This range works in the SelectionDAG-based lowering pipeline.
- **CN**: 这里实现了 WebAssembly 后端该部分DAG 到 DAG 指令选择所需的辅助例程 `getIntegerVT`, `getDataLayout`, `getPointerSizeInBits`。 这一段工作在基于 SelectionDAG 的 lowering 流水线中。

### Lines 320-357

```cpp
      }
      auto Sig = encodeFunctionSignature(CurDAG, DL, Returns, Params);

      auto SigOp = CurDAG->getTargetConstant(
          Sig, DL, EVT::getIntegerVT(*CurDAG->getContext(), Sig.getBitWidth()));
      MachineSDNode *RefTestNode = CurDAG->getMachineNode(
          WebAssembly::REF_TEST_FUNCREF, DL, MVT::i32, {SigOp, FuncRef});
      ReplaceNode(Node, RefTestNode);
      return;
    }
    }
    break;
  }

  case ISD::INTRINSIC_W_CHAIN: {
    unsigned IntNo = Node->getConstantOperandVal(1);
    const auto &TLI = CurDAG->getTargetLoweringInfo();
    MVT PtrVT = TLI.getPointerTy(CurDAG->getDataLayout());
    switch (IntNo) {
    case Intrinsic::wasm_tls_base: {
      MachineSDNode *TLSBase = CurDAG->getMachineNode(
          GlobalGetIns, DL, PtrVT, MVT::Other,
          CurDAG->getTargetExternalSymbol("__tls_base", PtrVT),
          Node->getOperand(0));
      ReplaceNode(Node, TLSBase);
      return;
    }

    case Intrinsic::wasm_catch: {
      int Tag = Node->getConstantOperandVal(2);
      SDValue SymNode = getTagSymNode(Tag, CurDAG);
      unsigned CatchOpcode = WebAssembly::WasmUseLegacyEH
                                 ? WebAssembly::CATCH_LEGACY
                                 : WebAssembly::CATCH;
      MachineSDNode *Catch =
          CurDAG->getMachineNode(CatchOpcode, DL,
                                 {
                                     PtrVT,     // exception pointer
```
- **EN**: Implements helper routine(s) `encodeFunctionSignature`, `getTargetConstant`, `getIntegerVT` for this portion of the WebAssembly backend DAG-to-DAG instruction selection. This range works in the SelectionDAG-based lowering pipeline.
- **CN**: 这里实现了 WebAssembly 后端该部分DAG 到 DAG 指令选择所需的辅助例程 `encodeFunctionSignature`, `getTargetConstant`, `getIntegerVT`。 这一段工作在基于 SelectionDAG 的 lowering 流水线中。

### Lines 358-395

```cpp
                                     MVT::Other // outchain type
                                 },
                                 {
                                     SymNode,            // exception symbol
                                     Node->getOperand(0) // inchain
                                 });
      ReplaceNode(Node, Catch);
      return;
    }
    }
    break;
  }

  case ISD::INTRINSIC_VOID: {
    unsigned IntNo = Node->getConstantOperandVal(1);
    switch (IntNo) {
    case Intrinsic::wasm_throw: {
      int Tag = Node->getConstantOperandVal(2);
      SDValue SymNode = getTagSymNode(Tag, CurDAG);
      MachineSDNode *Throw =
          CurDAG->getMachineNode(WebAssembly::THROW, DL,
                                 MVT::Other, // outchain type
                                 {
                                     SymNode,             // exception symbol
                                     Node->getOperand(3), // thrown value
                                     Node->getOperand(0)  // inchain
                                 });
      ReplaceNode(Node, Throw);
      return;
    }
    case Intrinsic::wasm_rethrow: {
      // RETHROW's BB argument will be populated in LateEHPrepare. Just use a
      // '0' as a placeholder for now.
      MachineSDNode *Rethrow = CurDAG->getMachineNode(
          WebAssembly::RETHROW, DL,
          MVT::Other, // outchain type
          {
              CurDAG->getConstant(0, DL, MVT::i32), // placeholder
```
- **EN**: Implements helper routine(s) `getOperand`, `ReplaceNode`, `getConstantOperandVal` for this portion of the WebAssembly backend DAG-to-DAG instruction selection. This range works in the SelectionDAG-based lowering pipeline.
- **CN**: 这里实现了 WebAssembly 后端该部分DAG 到 DAG 指令选择所需的辅助例程 `getOperand`, `ReplaceNode`, `getConstantOperandVal`。 这一段工作在基于 SelectionDAG 的 lowering 流水线中。

### Lines 396-433

```cpp
              Node->getOperand(0)                   // inchain
          });
      ReplaceNode(Node, Rethrow);
      return;
    }
    }
    break;
  }

  case WebAssemblyISD::CALL:
  case WebAssemblyISD::RET_CALL: {
    // CALL has both variable operands and variable results, but ISel only
    // supports one or the other. Split calls into two nodes glued together, one
    // for the operands and one for the results. These two nodes will be
    // recombined in a custom inserter hook into a single MachineInstr.
    SmallVector<SDValue, 16> Ops;
    for (size_t i = 1; i < Node->getNumOperands(); ++i) {
      SDValue Op = Node->getOperand(i);
      // Remove the wrapper when the call target is a function, an external
      // symbol (which will be lowered to a library function), or an alias of
      // a function. If the target is not a function/external symbol, we
      // shouldn't remove the wrapper, because we cannot call it directly and
      // instead we want it to be loaded with a CONST instruction and called
      // with a call_indirect later.
      if (i == 1 && Op->getOpcode() == WebAssemblyISD::Wrapper) {
        SDValue NewOp = Op->getOperand(0);
        if (auto *GlobalOp = dyn_cast<GlobalAddressSDNode>(NewOp.getNode())) {
          if (isa<Function>(
                  GlobalOp->getGlobal()->stripPointerCastsAndAliases()))
            Op = NewOp;
        } else if (isa<ExternalSymbolSDNode>(NewOp.getNode())) {
          Op = NewOp;
        }
      }
      Ops.push_back(Op);
    }

    // Add the chain last
```
- **EN**: Implements helper routine(s) `getOperand`, `ReplaceNode`, `getNumOperands` for this portion of the WebAssembly backend DAG-to-DAG instruction selection. It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这里实现了 WebAssembly 后端该部分DAG 到 DAG 指令选择所需的辅助例程 `getOperand`, `ReplaceNode`, `getNumOperands`。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

### Lines 434-471

```cpp
    Ops.push_back(Node->getOperand(0));
    MachineSDNode *CallParams =
        CurDAG->getMachineNode(WebAssembly::CALL_PARAMS, DL, MVT::Glue, Ops);

    unsigned Results = Node->getOpcode() == WebAssemblyISD::CALL
                           ? WebAssembly::CALL_RESULTS
                           : WebAssembly::RET_CALL_RESULTS;

    SDValue Link(CallParams, 0);
    MachineSDNode *CallResults =
        CurDAG->getMachineNode(Results, DL, Node->getVTList(), Link);
    ReplaceNode(Node, CallResults);
    return;
  }

  default:
    break;
  }

  // Select the default instruction.
  SelectCode(Node);
}

bool WebAssemblyDAGToDAGISel::SelectInlineAsmMemoryOperand(
    const SDValue &Op, InlineAsm::ConstraintCode ConstraintID,
    std::vector<SDValue> &OutOps) {
  switch (ConstraintID) {
  case InlineAsm::ConstraintCode::m:
    // We just support simple memory operands that just have a single address
    // operand and need no special handling.
    OutOps.push_back(Op);
    return false;
  default:
    break;
  }

  return true;
}
```
- **EN**: Implements helper routine(s) `push_back`, `getOperand`, `getMachineNode` for this portion of the WebAssembly backend DAG-to-DAG instruction selection. This range works in the SelectionDAG-based lowering pipeline.
- **CN**: 这里实现了 WebAssembly 后端该部分DAG 到 DAG 指令选择所需的辅助例程 `push_back`, `getOperand`, `getMachineNode`。 这一段工作在基于 SelectionDAG 的 lowering 流水线中。

### Lines 472-509

```cpp

bool WebAssemblyDAGToDAGISel::SelectAddrAddOperands(MVT OffsetType, SDValue N,
                                                    SDValue &Offset,
                                                    SDValue &Addr) {
  assert(N.getNumOperands() == 2 && "Attempting to fold in a non-binary op");

  // WebAssembly constant offsets are performed as unsigned with infinite
  // precision, so we need to check for NoUnsignedWrap so that we don't fold an
  // offset for an add that needs wrapping.
  if (N.getOpcode() == ISD::ADD && !N.getNode()->getFlags().hasNoUnsignedWrap())
    return false;

  for (size_t i = 0; i < 2; ++i) {
    SDValue Op = N.getOperand(i);
    SDValue OtherOp = N.getOperand(i == 0 ? 1 : 0);

    // Folds constants in an add into the offset.
    if (ConstantSDNode *CN = dyn_cast<ConstantSDNode>(Op)) {
      Offset =
          CurDAG->getTargetConstant(CN->getZExtValue(), SDLoc(N), OffsetType);
      Addr = OtherOp;
      return true;
    }

    // Fold target global addresses into the offset.
    if (!TM.isPositionIndependent()) {
      if (Op.getOpcode() == WebAssemblyISD::Wrapper)
        Op = Op.getOperand(0);

      if (Op.getOpcode() == ISD::TargetGlobalAddress) {
        Addr = OtherOp;
        Offset = Op;
        return true;
      }
    }
  }
  return false;
}
```
- **EN**: Implements helper routine(s) `SelectAddrAddOperands`, `getNumOperands`, `getOpcode` for this portion of the WebAssembly backend DAG-to-DAG instruction selection. This range works in the SelectionDAG-based lowering pipeline.
- **CN**: 这里实现了 WebAssembly 后端该部分DAG 到 DAG 指令选择所需的辅助例程 `SelectAddrAddOperands`, `getNumOperands`, `getOpcode`。 这一段工作在基于 SelectionDAG 的 lowering 流水线中。

### Lines 510-547

```cpp

bool WebAssemblyDAGToDAGISel::SelectAddrOperands(MVT AddrType,
                                                 unsigned ConstOpc, SDValue N,
                                                 SDValue &Offset,
                                                 SDValue &Addr) {
  SDLoc DL(N);

  // Fold target global addresses into the offset.
  if (!TM.isPositionIndependent()) {
    SDValue Op(N);
    if (Op.getOpcode() == WebAssemblyISD::Wrapper)
      Op = Op.getOperand(0);

    if (Op.getOpcode() == ISD::TargetGlobalAddress) {
      Offset = Op;
      Addr = SDValue(
          CurDAG->getMachineNode(ConstOpc, DL, AddrType,
                                 CurDAG->getTargetConstant(0, DL, AddrType)),
          0);
      return true;
    }
  }

  // Fold anything inside an add into the offset.
  if (N.getOpcode() == ISD::ADD &&
      SelectAddrAddOperands(AddrType, N, Offset, Addr))
    return true;

  // Likewise, treat an 'or' node as an 'add' if the or'ed bits are known to be
  // zero and fold them into the offset too.
  if (N.getOpcode() == ISD::OR) {
    bool OrIsAdd;
    if (ConstantSDNode *CN = dyn_cast<ConstantSDNode>(N.getOperand(1))) {
      OrIsAdd =
          CurDAG->MaskedValueIsZero(N->getOperand(0), CN->getAPIntValue());
    } else {
      KnownBits Known0 = CurDAG->computeKnownBits(N->getOperand(0), 0);
      KnownBits Known1 = CurDAG->computeKnownBits(N->getOperand(1), 0);
```
- **EN**: Implements helper routine(s) `SelectAddrOperands`, `DL`, `isPositionIndependent` for this portion of the WebAssembly backend DAG-to-DAG instruction selection. This range works in the SelectionDAG-based lowering pipeline.
- **CN**: 这里实现了 WebAssembly 后端该部分DAG 到 DAG 指令选择所需的辅助例程 `SelectAddrOperands`, `DL`, `isPositionIndependent`。 这一段工作在基于 SelectionDAG 的 lowering 流水线中。

### Lines 548-585

```cpp
      OrIsAdd = (~Known0.Zero & ~Known1.Zero) == 0;
    }

    if (OrIsAdd && SelectAddrAddOperands(AddrType, N, Offset, Addr))
      return true;
  }

  // Fold constant addresses into the offset.
  if (ConstantSDNode *CN = dyn_cast<ConstantSDNode>(N)) {
    Offset = CurDAG->getTargetConstant(CN->getZExtValue(), DL, AddrType);
    Addr = SDValue(
        CurDAG->getMachineNode(ConstOpc, DL, AddrType,
                               CurDAG->getTargetConstant(0, DL, AddrType)),
        0);
    return true;
  }

  // Else it's a plain old load/store with no offset.
  Offset = CurDAG->getTargetConstant(0, DL, AddrType);
  Addr = N;
  return true;
}

bool WebAssemblyDAGToDAGISel::SelectAddrOperands32(SDValue Op, SDValue &Offset,
                                                   SDValue &Addr) {
  return SelectAddrOperands(MVT::i32, WebAssembly::CONST_I32, Op, Offset, Addr);
}

bool WebAssemblyDAGToDAGISel::SelectAddrOperands64(SDValue Op, SDValue &Offset,
                                                   SDValue &Addr) {
  return SelectAddrOperands(MVT::i64, WebAssembly::CONST_I64, Op, Offset, Addr);
}

static MemSDNode *findMemSDNode(SDNode *N) {
  while (N) {
    if (auto *MemNode = dyn_cast<MemSDNode>(N))
      return MemNode;
    switch (N->getOpcode()) {
```
- **EN**: Implements helper routine(s) `SelectAddrAddOperands`, `getTargetConstant`, `getZExtValue` for this portion of the WebAssembly backend DAG-to-DAG instruction selection. This range works in the SelectionDAG-based lowering pipeline.
- **CN**: 这里实现了 WebAssembly 后端该部分DAG 到 DAG 指令选择所需的辅助例程 `SelectAddrAddOperands`, `getTargetConstant`, `getZExtValue`。 这一段工作在基于 SelectionDAG 的 lowering 流水线中。

### Lines 586-623

```cpp
    case ISD::ZERO_EXTEND:
    case ISD::SIGN_EXTEND:
    case ISD::ANY_EXTEND:
    case ISD::SIGN_EXTEND_INREG:
    case ISD::AssertZext:
    case ISD::AssertSext:
    case ISD::TRUNCATE:
    case ISD::BITCAST:
    case ISD::AND:
      N = N->getOperand(0).getNode();
      break;
    default:
      return nullptr;
    }
  }
  return nullptr;
}

bool WebAssemblyDAGToDAGISel::SelectAtomicAddrOperands(SDNode *Op, SDValue N,
                                                       SDValue &Offset,
                                                       SDValue &Addr,
                                                       SDValue &Order,
                                                       bool Is64) {
  auto *MemNode = findMemSDNode(Op);
  if (!MemNode)
    return false;

  bool Match = Is64 ? SelectAddrOperands64(N, Offset, Addr)
                    : SelectAddrOperands32(N, Offset, Addr);
  if (!Match)
    return false;

  auto Ordering = MemNode->getMergedOrdering();
  unsigned OrderVal = wasm::WASM_MEM_ORDER_SEQ_CST;
  if (Subtarget->hasRelaxedAtomics())
    OrderVal = getWebAssemblyMemoryOrder(Ordering);
  Order = CurDAG->getTargetConstant(OrderVal, SDLoc(Op), MVT::i32);
  return true;
```
- **EN**: Implements helper routine(s) `getOperand`, `getNode`, `SelectAtomicAddrOperands` for this portion of the WebAssembly backend DAG-to-DAG instruction selection. This range works in the SelectionDAG-based lowering pipeline.
- **CN**: 这里实现了 WebAssembly 后端该部分DAG 到 DAG 指令选择所需的辅助例程 `getOperand`, `getNode`, `SelectAtomicAddrOperands`。 这一段工作在基于 SelectionDAG 的 lowering 流水线中。

### Lines 624-645

```cpp
}

bool WebAssemblyDAGToDAGISel::SelectAtomicAddrOperands32(SDNode *Op, SDValue N,
                                                         SDValue &Offset,
                                                         SDValue &Addr,
                                                         SDValue &Order) {
  return SelectAtomicAddrOperands(Op, N, Offset, Addr, Order, /*Is64=*/false);
}

bool WebAssemblyDAGToDAGISel::SelectAtomicAddrOperands64(SDNode *Op, SDValue N,
                                                         SDValue &Offset,
                                                         SDValue &Addr,
                                                         SDValue &Order) {
  return SelectAtomicAddrOperands(Op, N, Offset, Addr, Order, /*Is64=*/true);
}

/// This pass converts a legalized DAG into a WebAssembly-specific DAG, ready
/// for instruction scheduling.
FunctionPass *llvm::createWebAssemblyISelDag(WebAssemblyTargetMachine &TM,
                                             CodeGenOptLevel OptLevel) {
  return new WebAssemblyDAGToDAGISelLegacy(TM, OptLevel);
}
```
- **EN**: Implements helper routine(s) `SelectAtomicAddrOperands32`, `SelectAtomicAddrOperands`, `SelectAtomicAddrOperands64` for this portion of the WebAssembly backend DAG-to-DAG instruction selection. This range works in the SelectionDAG-based lowering pipeline.
- **CN**: 这里实现了 WebAssembly 后端该部分DAG 到 DAG 指令选择所需的辅助例程 `SelectAtomicAddrOperands32`, `SelectAtomicAddrOperands`, `SelectAtomicAddrOperands64`。 这一段工作在基于 SelectionDAG 的 lowering 流水线中。

## Key Concepts / 关键概念

- WebAssembly backend integration / WebAssembly 后端集成
- DAG-to-DAG instruction selection / DAG 到 DAG 指令选择
- SelectionDAG lowering / SelectionDAG lowering
- Stack frame management / 栈帧管理
- Subtarget features / 子目标特性
- Instruction selection or opcode handling / 指令选择或操作码处理
- Target machine configuration / 目标机器配置
- Object format integration / 目标文件格式集成

## Dependencies / 依赖关系

### Direct Includes / 直接包含

- `MCTargetDesc/WebAssemblyMCTargetDesc.h`
- `WebAssembly.h`
- `WebAssemblyISelLowering.h`
- `WebAssemblyTargetMachine.h`
- `WebAssemblyUtilities.h`
- `llvm/BinaryFormat/Wasm.h`
- `llvm/CodeGen/MachineFrameInfo.h`
- `llvm/CodeGen/SelectionDAGISel.h`
- `llvm/CodeGen/WasmEHInfo.h`
- `llvm/IR/DiagnosticInfo.h`
- `llvm/IR/Function.h`
- `llvm/IR/IntrinsicsWebAssembly.h`
- `llvm/MC/MCSymbolWasm.h`
- `llvm/Support/Debug.h`
- `llvm/Support/KnownBits.h`
- `llvm/Support/raw_ostream.h`
- `WebAssemblyGenDAGISel.inc`

### Important Collaborators / 重要协作组件

- LLVM CodeGen layer / LLVM CodeGen 层
- LLVM MC layer / LLVM MC 层
- LLVM Support utilities / LLVM Support 工具层
- WebAssembly target-specific helpers / WebAssembly 目标专用辅助组件
