# WebAssemblyTargetMachine.cpp — Code Analysis / 代码分析

## Source / 来源

- **File**: `llvm/lib/Target/WebAssembly/WebAssemblyTargetMachine.cpp`
- **Repository**: `llvm-project`
- **Purpose (EN)**: This file defines the WebAssembly-specific subclass of TargetMachine.
- **Purpose (CN)**: 该文件位于 `llvm/lib/Target/WebAssembly/WebAssemblyTargetMachine.cpp`，主要负责 WebAssembly 后端的目标机器配置。 文件内容以具体实现、辅助函数和后端决策逻辑为主。

## Line-by-Line Analysis / 逐行分析

### Lines 1-6

```cpp
//===- WebAssemblyTargetMachine.cpp - Define TargetMachine for WebAssembly -==//
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
/// This file defines the WebAssembly-specific subclass of TargetMachine.
///
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "\file".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“\file”。

### Lines 12-18

```cpp
//===----------------------------------------------------------------------===//

#include "WebAssemblyTargetMachine.h"
#include "MCTargetDesc/WebAssemblyMCTargetDesc.h"
#include "TargetInfo/WebAssemblyTargetInfo.h"
#include "WebAssembly.h"
#include "WebAssemblyISelLowering.h"
```
- **EN**: Pulls in direct dependencies required by this target machine configuration, so later declarations can reuse LLVM infrastructure and target-specific helpers.
- **CN**: 这一段引入该目标机器配置所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。

### Lines 19-25

```cpp
#include "WebAssemblyMachineFunctionInfo.h"
#include "WebAssemblyTargetObjectFile.h"
#include "WebAssemblyTargetTransformInfo.h"
#include "WebAssemblyUtilities.h"
#include "llvm/CodeGen/GlobalISel/IRTranslator.h"
#include "llvm/CodeGen/GlobalISel/InstructionSelect.h"
#include "llvm/CodeGen/GlobalISel/Legalizer.h"
```
- **EN**: Pulls in direct dependencies required by this target machine configuration, so later declarations can reuse LLVM infrastructure and target-specific helpers. This range participates in the GlobalISel pipeline.
- **CN**: 这一段引入该目标机器配置所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。 这一段参与 GlobalISel 流水线。

### Lines 26-32

```cpp
#include "llvm/CodeGen/GlobalISel/RegBankSelect.h"
#include "llvm/CodeGen/MIRParser/MIParser.h"
#include "llvm/CodeGen/Passes.h"
#include "llvm/CodeGen/RegAllocRegistry.h"
#include "llvm/CodeGen/TargetPassConfig.h"
#include "llvm/IR/Function.h"
#include "llvm/InitializePasses.h"
```
- **EN**: Pulls in direct dependencies required by this target machine configuration, so later declarations can reuse LLVM infrastructure and target-specific helpers. This range participates in the GlobalISel pipeline.
- **CN**: 这一段引入该目标机器配置所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。 这一段参与 GlobalISel 流水线。

### Lines 33-41

```cpp
#include "llvm/MC/TargetRegistry.h"
#include "llvm/Support/Compiler.h"
#include "llvm/Target/TargetOptions.h"
#include "llvm/Transforms/Scalar.h"
#include "llvm/Transforms/Scalar/LowerAtomicPass.h"
#include "llvm/Transforms/Utils.h"
#include <optional>
using namespace llvm;
```
- **EN**: Pulls in direct dependencies required by this target machine configuration, so later declarations can reuse LLVM infrastructure and target-specific helpers.
- **CN**: 这一段引入该目标机器配置所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。

### Lines 42-79

```cpp
#define DEBUG_TYPE "wasm"

// A command-line option to keep implicit locals
// for the purpose of testing with lit/llc ONLY.
// This produces output which is not valid WebAssembly, and is not supported
// by assemblers/disassemblers and other MC based tools.
static cl::opt<bool> WasmDisableExplicitLocals(
    "wasm-disable-explicit-locals", cl::Hidden,
    cl::desc("WebAssembly: output implicit locals in"
             " instruction output for test purposes only."),
    cl::init(false));

// Exception handling & setjmp-longjmp handling related options.

// Emscripten's asm.js-style exception handling
cl::opt<bool> WebAssembly::WasmEnableEmEH(
    "enable-emscripten-cxx-exceptions",
    cl::desc("WebAssembly Emscripten-style exception handling"),
    cl::init(false));
// Emscripten's asm.js-style setjmp/longjmp handling
cl::opt<bool> WebAssembly::WasmEnableEmSjLj(
    "enable-emscripten-sjlj",
    cl::desc("WebAssembly Emscripten-style setjmp/longjmp handling"),
    cl::init(false));
// Exception handling using wasm EH instructions
cl::opt<bool>
    WebAssembly::WasmEnableEH("wasm-enable-eh",
                              cl::desc("WebAssembly exception handling"));
// setjmp/longjmp handling using wasm EH instrutions
cl::opt<bool> WebAssembly::WasmEnableSjLj(
    "wasm-enable-sjlj", cl::desc("WebAssembly setjmp/longjmp handling"));
// If true, use the legacy Wasm EH proposal:
// https://github.com/WebAssembly/exception-handling/blob/main/proposals/exception-handling/legacy/Exceptions.md
// And if false, use the standardized Wasm EH proposal:
// https://github.com/WebAssembly/exception-handling/blob/main/proposals/exception-handling/Exceptions.md
// Currently set to true by default because not all major web browsers turn on
// the new standard proposal by default, but will later change to false.
cl::opt<bool> WebAssembly::WasmUseLegacyEH(
```
- **EN**: Defines or closes the header guard so the declarations in this header are only processed once per translation unit. Notable symbols in this range include `WasmDisableExplicitLocals`, `desc`, `init`.
- **CN**: 这里定义或结束头文件保护宏，确保该头文件中的声明在一个编译单元内只会被处理一次。 该区间中较显眼的符号包括 `WasmDisableExplicitLocals`, `desc`, `init`。

### Lines 80-117

```cpp
    "wasm-use-legacy-eh", cl::desc("WebAssembly exception handling (legacy)"),
    cl::init(true));

extern "C" LLVM_ABI LLVM_EXTERNAL_VISIBILITY void
LLVMInitializeWebAssemblyTarget() {
  // Register the target.
  RegisterTargetMachine<WebAssemblyTargetMachine> X(
      getTheWebAssemblyTarget32());
  RegisterTargetMachine<WebAssemblyTargetMachine> Y(
      getTheWebAssemblyTarget64());

  // Register backend passes
  auto &PR = *PassRegistry::getPassRegistry();
  initializeGlobalISel(PR);
  initializeWebAssemblyPreLegalizerCombinerPass(PR);
  initializeWebAssemblyPostLegalizerCombinerPass(PR);
  initializeWebAssemblyAddMissingPrototypesPass(PR);
  initializeWebAssemblyLowerEmscriptenEHSjLjPass(PR);
  initializeLowerGlobalDtorsLegacyPassPass(PR);
  initializeFixFunctionBitcastsPass(PR);
  initializeOptimizeReturnedPass(PR);
  initializeWebAssemblyRefTypeMem2LocalPass(PR);
  initializeWebAssemblyArgumentMovePass(PR);
  initializeWebAssemblyAsmPrinterPass(PR);
  initializeWebAssemblySetP2AlignOperandsPass(PR);
  initializeWebAssemblyReplacePhysRegsPass(PR);
  initializeWebAssemblyOptimizeLiveIntervalsPass(PR);
  initializeWebAssemblyMemIntrinsicResultsPass(PR);
  initializeWebAssemblyRegStackifyPass(PR);
  initializeWebAssemblyRegColoringPass(PR);
  initializeWebAssemblyNullifyDebugValueListsPass(PR);
  initializeWebAssemblyFixIrreducibleControlFlowPass(PR);
  initializeWebAssemblyLateEHPreparePass(PR);
  initializeWebAssemblyExceptionInfoPass(PR);
  initializeWebAssemblyCFGSortPass(PR);
  initializeWebAssemblyCFGStackifyPass(PR);
  initializeWebAssemblyExplicitLocalsPass(PR);
  initializeWebAssemblyLowerBrUnlessPass(PR);
```
- **EN**: Implements helper routine(s) `desc`, `handling`, `init` for this portion of the WebAssembly backend target machine configuration. This range participates in the GlobalISel pipeline.
- **CN**: 这里实现了 WebAssembly 后端该部分目标机器配置所需的辅助例程 `desc`, `handling`, `init`。 这一段参与 GlobalISel 流水线。

### Lines 118-126

```cpp
  initializeWebAssemblyRegNumberingPass(PR);
  initializeWebAssemblyDebugFixupPass(PR);
  initializeWebAssemblyPeepholePass(PR);
  initializeWebAssemblyMCLowerPrePassPass(PR);
  initializeWebAssemblyLowerRefTypesIntPtrConvPass(PR);
  initializeWebAssemblyFixBrTableDefaultsPass(PR);
  initializeWebAssemblyDAGToDAGISelLegacyPass(PR);
}
```
- **EN**: Declares function entry points including `initializeWebAssemblyRegNumberingPass`, `initializeWebAssemblyDebugFixupPass`, `initializeWebAssemblyPeepholePass` that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口，包括 `initializeWebAssemblyRegNumberingPass`, `initializeWebAssemblyDebugFixupPass`, `initializeWebAssemblyPeepholePass`。

### Lines 127-128

```cpp
//===----------------------------------------------------------------------===//
// WebAssembly Lowering public interface.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "WebAssembly Lowering public interface.".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“WebAssembly Lowering public interface.”。

### Lines 129-166

```cpp
//===----------------------------------------------------------------------===//

static Reloc::Model getEffectiveRelocModel(std::optional<Reloc::Model> RM) {
  // Default to static relocation model.  This should always be more optimial
  // than PIC since the static linker can determine all global addresses and
  // assume direct function calls.
  return RM.value_or(Reloc::Static);
}

using WebAssembly::WasmEnableEH;
using WebAssembly::WasmEnableEmEH;
using WebAssembly::WasmEnableEmSjLj;
using WebAssembly::WasmEnableSjLj;

static void basicCheckForEHAndSjLj(TargetMachine *TM) {

  // You can't enable two modes of EH at the same time
  if (WasmEnableEmEH && WasmEnableEH)
    report_fatal_error(
        "-enable-emscripten-cxx-exceptions not allowed with -wasm-enable-eh");
  // You can't enable two modes of SjLj at the same time
  if (WasmEnableEmSjLj && WasmEnableSjLj)
    report_fatal_error(
        "-enable-emscripten-sjlj not allowed with -wasm-enable-sjlj");
  // You can't mix Emscripten EH with Wasm SjLj.
  if (WasmEnableEmEH && WasmEnableSjLj)
    report_fatal_error(
        "-enable-emscripten-cxx-exceptions not allowed with -wasm-enable-sjlj");

  if (TM->Options.ExceptionModel == ExceptionHandling::None) {
    // FIXME: These flags should be removed in favor of directly using the
    // generically configured ExceptionsType
    if (WebAssembly::WasmEnableEH || WebAssembly::WasmEnableSjLj)
      TM->Options.ExceptionModel = ExceptionHandling::Wasm;
  }

  // Basic Correctness checking related to -exception-model
  if (TM->Options.ExceptionModel != ExceptionHandling::None &&
```
- **EN**: Implements helper routine(s) `getEffectiveRelocModel`, `value_or`, `basicCheckForEHAndSjLj` for this portion of the WebAssembly backend target machine configuration.
- **CN**: 这里实现了 WebAssembly 后端该部分目标机器配置所需的辅助例程 `getEffectiveRelocModel`, `value_or`, `basicCheckForEHAndSjLj`。

### Lines 167-204

```cpp
      TM->Options.ExceptionModel != ExceptionHandling::Wasm)
    report_fatal_error("-exception-model should be either 'none' or 'wasm'");
  if (WasmEnableEmEH && TM->Options.ExceptionModel == ExceptionHandling::Wasm)
    report_fatal_error("-exception-model=wasm not allowed with "
                       "-enable-emscripten-cxx-exceptions");
  if (WasmEnableEH && TM->Options.ExceptionModel != ExceptionHandling::Wasm)
    report_fatal_error(
        "-wasm-enable-eh only allowed with -exception-model=wasm");
  if (WasmEnableSjLj && TM->Options.ExceptionModel != ExceptionHandling::Wasm)
    report_fatal_error(
        "-wasm-enable-sjlj only allowed with -exception-model=wasm");
  if ((!WasmEnableEH && !WasmEnableSjLj) &&
      TM->Options.ExceptionModel == ExceptionHandling::Wasm)
    report_fatal_error(
        "-exception-model=wasm only allowed with at least one of "
        "-wasm-enable-eh or -wasm-enable-sjlj");

  // Currently it is allowed to mix Wasm EH with Emscripten SjLj as an interim
  // measure, but some code will error out at compile time in this combination.
  // See WebAssemblyLowerEmscriptenEHSjLj pass for details.
}

/// Create an WebAssembly architecture model.
///
WebAssemblyTargetMachine::WebAssemblyTargetMachine(
    const Target &T, const Triple &TT, StringRef CPU, StringRef FS,
    const TargetOptions &Options, std::optional<Reloc::Model> RM,
    std::optional<CodeModel::Model> CM, CodeGenOptLevel OL, bool JIT)
    : CodeGenTargetMachineImpl(T, TT.computeDataLayout(), TT, CPU, FS, Options,
                               getEffectiveRelocModel(RM),
                               getEffectiveCodeModel(CM, CodeModel::Large), OL),
      TLOF(new WebAssemblyTargetObjectFile()),
      UsesMultivalueABI(Options.MCOptions.getABIName() == "experimental-mv") {
  // WebAssembly type-checks instructions, but a noreturn function with a return
  // type that doesn't match the context will cause a check failure. So we lower
  // LLVM 'unreachable' to ISD::TRAP and then lower that to WebAssembly's
  // 'unreachable' instructions which is meant for that case. Formerly, we also
  // needed to add checks to SP failure emission in the instruction selection
```
- **EN**: Implements helper routine(s) `report_fatal_error`, `WebAssemblyTargetMachine`, `CodeGenTargetMachineImpl` for this portion of the WebAssembly backend target machine configuration. This range works in the SelectionDAG-based lowering pipeline.
- **CN**: 这里实现了 WebAssembly 后端该部分目标机器配置所需的辅助例程 `report_fatal_error`, `WebAssemblyTargetMachine`, `CodeGenTargetMachineImpl`。 这一段工作在基于 SelectionDAG 的 lowering 流水线中。

### Lines 205-242

```cpp
  // backends, but this has since been tied to TrapUnreachable and is no longer
  // necessary.
  this->Options.TrapUnreachable = true;
  this->Options.NoTrapAfterNoreturn = false;

  // WebAssembly treats each function as an independent unit. Force
  // -ffunction-sections, effectively, so that we can emit them independently.
  this->Options.FunctionSections = true;
  this->Options.DataSections = true;
  this->Options.UniqueSectionNames = true;

  basicCheckForEHAndSjLj(this);
  initAsmInfo();

  LLT::setUseExtended(true);

  // Note that we don't use setRequiresStructuredCFG(true). It disables
  // optimizations than we're ok with, and want, such as critical edge
  // splitting and tail merging.
}

WebAssemblyTargetMachine::~WebAssemblyTargetMachine() = default; // anchor.

const WebAssemblySubtarget *WebAssemblyTargetMachine::getSubtargetImpl() const {
  return getSubtargetImpl(std::string(getTargetCPU()),
                          std::string(getTargetFeatureString()));
}

const WebAssemblySubtarget *
WebAssemblyTargetMachine::getSubtargetImpl(std::string CPU,
                                           std::string FS) const {
  auto &I = SubtargetMap[CPU + FS];
  if (!I) {
    I = std::make_unique<WebAssemblySubtarget>(TargetTriple, CPU, FS, *this);
  }
  return I.get();
}
```
- **EN**: Implements helper routine(s) `basicCheckForEHAndSjLj`, `initAsmInfo`, `setUseExtended` for this portion of the WebAssembly backend target machine configuration. Subtarget feature gating influences the behavior here.
- **CN**: 这里实现了 WebAssembly 后端该部分目标机器配置所需的辅助例程 `basicCheckForEHAndSjLj`, `initAsmInfo`, `setUseExtended`。 子目标特性裁剪会影响这里的行为。

### Lines 243-260

```cpp
const WebAssemblySubtarget *
WebAssemblyTargetMachine::getSubtargetImpl(const Function &F) const {
  Attribute CPUAttr = F.getFnAttribute("target-cpu");
  Attribute FSAttr = F.getFnAttribute("target-features");

  std::string CPU =
      CPUAttr.isValid() ? CPUAttr.getValueAsString().str() : TargetCPU;
  std::string FS =
      FSAttr.isValid() ? FSAttr.getValueAsString().str() : TargetFS;

  // This needs to be done before we create a new subtarget since any
  // creation will depend on the TM and the code generation flags on the
  // function that reside in TargetOptions.
  resetTargetOptions(F);

  return getSubtargetImpl(CPU, FS);
}
```
- **EN**: Implements helper routine(s) `getSubtargetImpl`, `getFnAttribute`, `isValid` for this portion of the WebAssembly backend target machine configuration. Subtarget feature gating influences the behavior here.
- **CN**: 这里实现了 WebAssembly 后端该部分目标机器配置所需的辅助例程 `getSubtargetImpl`, `getFnAttribute`, `isValid`。 子目标特性裁剪会影响这里的行为。

### Lines 261-298

```cpp
namespace {

class CoalesceFeaturesAndStripAtomics final : public ModulePass {
  // Take the union of all features used in the module and use it for each
  // function individually, since having multiple feature sets in one module
  // currently does not make sense for WebAssembly. If atomics are not enabled,
  // also strip atomic operations and thread local storage.
  static char ID;
  WebAssemblyTargetMachine *WasmTM;

public:
  CoalesceFeaturesAndStripAtomics(WebAssemblyTargetMachine *WasmTM)
      : ModulePass(ID), WasmTM(WasmTM) {}

  bool runOnModule(Module &M) override {
    FeatureBitset Features = coalesceFeatures(M);

    std::string FeatureStr = getFeatureString(Features);
    WasmTM->setTargetFeatureString(FeatureStr);
    for (auto &F : M)
      replaceFeatures(F, FeatureStr);

    bool StrippedAtomics = false;
    bool StrippedTLS = false;

    if (!Features[WebAssembly::FeatureAtomics]) {
      StrippedAtomics = stripAtomics(M);
      StrippedTLS = stripThreadLocals(M);
    } else if (!Features[WebAssembly::FeatureBulkMemory]) {
      StrippedTLS |= stripThreadLocals(M);
    }

    if (StrippedAtomics && !StrippedTLS)
      stripThreadLocals(M);
    else if (StrippedTLS && !StrippedAtomics)
      stripAtomics(M);

    recordFeatures(M, Features, StrippedAtomics || StrippedTLS);
```
- **EN**: Declares a backend-facing type `CoalesceFeaturesAndStripAtomics`, `ModulePass`, `WasmTM` and outlines the API or state that nearby code will rely on. Subtarget feature gating influences the behavior here.
- **CN**: 这里声明面向后端的类型 `CoalesceFeaturesAndStripAtomics`, `ModulePass`, `WasmTM`，并勾勒出周边代码会依赖的接口或状态。 子目标特性裁剪会影响这里的行为。

### Lines 299-336

```cpp

    // Conservatively assume we have made some change
    return true;
  }

private:
  FeatureBitset coalesceFeatures(const Module &M) {
    // Union the features of all defined functions. Start with an empty set, so
    // that if a feature is disabled in every function, we'll compute it as
    // disabled. If any function lacks a target-features attribute, it'll
    // default to the target CPU from the `TargetMachine`.
    FeatureBitset Features;
    bool AnyDefinedFuncs = false;
    for (auto &F : M) {
      if (F.isDeclaration())
        continue;

      Features |= WasmTM->getSubtargetImpl(F)->getFeatureBits();
      AnyDefinedFuncs = true;
    }

    // If we have no defined functions, use the target CPU from the
    // `TargetMachine`.
    if (!AnyDefinedFuncs) {
      Features =
          WasmTM
              ->getSubtargetImpl(std::string(WasmTM->getTargetCPU()),
                                 std::string(WasmTM->getTargetFeatureString()))
              ->getFeatureBits();
    }

    return Features;
  }

  static std::string getFeatureString(const FeatureBitset &Features) {
    std::string Ret;
    for (const SubtargetFeatureKV &KV : WebAssemblyFeatureKV) {
      if (Features[KV.Value])
```
- **EN**: Implements helper routine(s) `coalesceFeatures`, `isDeclaration`, `getSubtargetImpl` for this portion of the WebAssembly backend target machine configuration. Subtarget feature gating influences the behavior here.
- **CN**: 这里实现了 WebAssembly 后端该部分目标机器配置所需的辅助例程 `coalesceFeatures`, `isDeclaration`, `getSubtargetImpl`。 子目标特性裁剪会影响这里的行为。

### Lines 337-374

```cpp
        Ret += (StringRef("+") + KV.Key + ",").str();
      else
        Ret += (StringRef("-") + KV.Key + ",").str();
    }
    // remove trailing ','
    Ret.pop_back();
    return Ret;
  }

  void replaceFeatures(Function &F, const std::string &Features) {
    F.removeFnAttr("target-features");
    F.removeFnAttr("target-cpu");
    F.addFnAttr("target-features", Features);
  }

  bool stripAtomics(Module &M) {
    // Detect whether any atomics will be lowered, since there is no way to tell
    // whether the LowerAtomic pass lowers e.g. stores.
    bool Stripped = false;
    for (auto &F : M) {
      for (auto &B : F) {
        for (auto &I : B) {
          if (I.isAtomic()) {
            Stripped = true;
            goto done;
          }
        }
      }
    }

  done:
    if (!Stripped)
      return false;

    LowerAtomicPass Lowerer;
    FunctionAnalysisManager FAM;
    for (auto &F : M)
      Lowerer.run(F, FAM);
```
- **EN**: Implements helper routine(s) `StringRef`, `str`, `pop_back` for this portion of the WebAssembly backend target machine configuration. Subtarget feature gating influences the behavior here.
- **CN**: 这里实现了 WebAssembly 后端该部分目标机器配置所需的辅助例程 `StringRef`, `str`, `pop_back`。 子目标特性裁剪会影响这里的行为。

### Lines 375-412

```cpp

    return true;
  }

  bool stripThreadLocals(Module &M) {
    bool Stripped = false;
    for (auto &GV : M.globals()) {
      if (GV.isThreadLocal()) {
        // replace `@llvm.threadlocal.address.pX(GV)` with `GV`.
        for (Use &U : make_early_inc_range(GV.uses())) {
          if (IntrinsicInst *II = dyn_cast<IntrinsicInst>(U.getUser())) {
            if (II->getIntrinsicID() == Intrinsic::threadlocal_address &&
                II->getArgOperand(0) == &GV) {
              II->replaceAllUsesWith(&GV);
              II->eraseFromParent();
            }
          }
        }

        Stripped = true;
        GV.setThreadLocal(false);
      }
    }
    return Stripped;
  }

  void recordFeatures(Module &M, const FeatureBitset &Features, bool Stripped) {
    for (const SubtargetFeatureKV &KV : WebAssemblyFeatureKV) {
      if (Features[KV.Value]) {
        // Mark features as used
        std::string MDKey = (StringRef("wasm-feature-") + KV.Key).str();
        M.addModuleFlag(Module::ModFlagBehavior::Error, MDKey,
                        wasm::WASM_FEATURE_PREFIX_USED);
      }
    }
    // Code compiled without atomics or bulk-memory may have had its atomics or
    // thread-local data lowered to nonatomic operations or non-thread-local
    // data. In that case, we mark the pseudo-feature "shared-mem" as disallowed
```
- **EN**: Implements helper routine(s) `stripThreadLocals`, `globals`, `isThreadLocal` for this portion of the WebAssembly backend target machine configuration. Subtarget feature gating influences the behavior here.
- **CN**: 这里实现了 WebAssembly 后端该部分目标机器配置所需的辅助例程 `stripThreadLocals`, `globals`, `isThreadLocal`。 子目标特性裁剪会影响这里的行为。

### Lines 413-423

```cpp
    // to tell the linker that it would be unsafe to allow this code ot be used
    // in a module with shared memory.
    if (Stripped) {
      M.addModuleFlag(Module::ModFlagBehavior::Error, "wasm-feature-shared-mem",
                      wasm::WASM_FEATURE_PREFIX_DISALLOWED);
    }
  }
};
char CoalesceFeaturesAndStripAtomics::ID = 0;

/// WebAssembly Code Generator Pass Configuration Options.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "to tell the linker that it would be unsafe to allow this code ot be used". Subtarget feature gating influences the behavior here.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“to tell the linker that it would be unsafe to allow this code ot be used”。 子目标特性裁剪会影响这里的行为。

### Lines 424-461

```cpp
class WebAssemblyPassConfig final : public TargetPassConfig {
public:
  WebAssemblyPassConfig(WebAssemblyTargetMachine &TM, PassManagerBase &PM)
      : TargetPassConfig(TM, PM) {}

  WebAssemblyTargetMachine &getWebAssemblyTargetMachine() const {
    return getTM<WebAssemblyTargetMachine>();
  }

  FunctionPass *createTargetRegisterAllocator(bool) override;

  void addIRPasses() override;
  void addISelPrepare() override;
  bool addInstSelector() override;
  void addOptimizedRegAlloc() override;
  void addPostRegAlloc() override;
  bool addGCPasses() override { return false; }
  void addPreEmitPass() override;
  bool addPreISel() override;

  // No reg alloc
  bool addRegAssignAndRewriteFast() override { return false; }

  // No reg alloc
  bool addRegAssignAndRewriteOptimized() override { return false; }

  bool addIRTranslator() override;
  void addPreLegalizeMachineIR() override;
  bool addLegalizeMachineIR() override;
  void addPreRegBankSelect() override;
  bool addRegBankSelect() override;
  bool addGlobalInstructionSelect() override;
};
} // end anonymous namespace

MachineFunctionInfo *WebAssemblyTargetMachine::createMachineFunctionInfo(
    BumpPtrAllocator &Allocator, const Function &F,
    const TargetSubtargetInfo *STI) const {
```
- **EN**: Declares a backend-facing type `WebAssemblyPassConfig`, `TargetPassConfig`, `getWebAssemblyTargetMachine` and outlines the API or state that nearby code will rely on. Subtarget feature gating influences the behavior here.
- **CN**: 这里声明面向后端的类型 `WebAssemblyPassConfig`, `TargetPassConfig`, `getWebAssemblyTargetMachine`，并勾勒出周边代码会依赖的接口或状态。 子目标特性裁剪会影响这里的行为。

### Lines 462-479

```cpp
  return WebAssemblyFunctionInfo::create<WebAssemblyFunctionInfo>(Allocator, F,
                                                                  STI);
}

TargetTransformInfo
WebAssemblyTargetMachine::getTargetTransformInfo(const Function &F) const {
  return TargetTransformInfo(std::make_unique<WebAssemblyTTIImpl>(this, F));
}

TargetPassConfig *
WebAssemblyTargetMachine::createPassConfig(PassManagerBase &PM) {
  return new WebAssemblyPassConfig(*this, PM);
}

FunctionPass *WebAssemblyPassConfig::createTargetRegisterAllocator(bool) {
  return nullptr; // No reg alloc
}
```
- **EN**: Implements helper routine(s) `getTargetTransformInfo`, `TargetTransformInfo`, `createPassConfig` for this portion of the WebAssembly backend target machine configuration.
- **CN**: 这里实现了 WebAssembly 后端该部分目标机器配置所需的辅助例程 `getTargetTransformInfo`, `TargetTransformInfo`, `createPassConfig`。

### Lines 480-482

```cpp
//===----------------------------------------------------------------------===//
// The following functions are called from lib/CodeGen/Passes.cpp to modify
// the CodeGen pass sequence.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "The following functions are called from lib/CodeGen/Passes.cpp to modify".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“The following functions are called from lib/CodeGen/Passes.cpp to modify”。

### Lines 483-520

```cpp
//===----------------------------------------------------------------------===//

void WebAssemblyPassConfig::addIRPasses() {
  // Add signatures to prototype-less function declarations
  addPass(createWebAssemblyAddMissingPrototypes());

  // Lower .llvm.global_dtors into .llvm.global_ctors with __cxa_atexit calls.
  addPass(createLowerGlobalDtorsLegacyPass());

  // Fix function bitcasts, as WebAssembly requires caller and callee signatures
  // to match.
  addPass(createWebAssemblyFixFunctionBitcasts());

  // Optimize "returned" function attributes.
  if (getOptLevel() != CodeGenOptLevel::None)
    addPass(createWebAssemblyOptimizeReturned());

  // If exception handling is not enabled and setjmp/longjmp handling is
  // enabled, we lower invokes into calls and delete unreachable landingpad
  // blocks. Lowering invokes when there is no EH support is done in
  // TargetPassConfig::addPassesToHandleExceptions, but that runs after these IR
  // passes and Emscripten SjLj handling expects all invokes to be lowered
  // before.
  if (!WasmEnableEmEH && !WasmEnableEH) {
    addPass(createLowerInvokePass());
    // The lower invoke pass may create unreachable code. Remove it in order not
    // to process dead blocks in setjmp/longjmp handling.
    addPass(createUnreachableBlockEliminationPass());
  }

  // Handle exceptions and setjmp/longjmp if enabled. Unlike Wasm EH preparation
  // done in WasmEHPrepare pass, Wasm SjLj preparation shares libraries and
  // transformation algorithms with Emscripten SjLj, so we run
  // LowerEmscriptenEHSjLj pass also when Wasm SjLj is enabled.
  if (WasmEnableEmEH || WasmEnableEmSjLj || WasmEnableSjLj)
    addPass(createWebAssemblyLowerEmscriptenEHSjLj());

  // Expand indirectbr instructions to switches.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Add signatures to prototype-less function declarations".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Add signatures to prototype-less function declarations”。

### Lines 521-558

```cpp
  addPass(createIndirectBrExpandPass());

  // Try to expand `vecreduce_{and, or}` into `{any, all}_true`.
  addPass(createWebAssemblyReduceToAnyAllTrue(getWebAssemblyTargetMachine()));

  TargetPassConfig::addIRPasses();
}

void WebAssemblyPassConfig::addISelPrepare() {
  // We need to move reference type allocas to WASM_ADDRESS_SPACE_VAR so that
  // loads and stores are promoted to local.gets/local.sets.
  addPass(createWebAssemblyRefTypeMem2Local());
  // Lower atomics and TLS if necessary
  addPass(new CoalesceFeaturesAndStripAtomics(&getWebAssemblyTargetMachine()));

  // This is a no-op if atomics are not used in the module
  addPass(createAtomicExpandLegacyPass());

  TargetPassConfig::addISelPrepare();
}

bool WebAssemblyPassConfig::addInstSelector() {
  (void)TargetPassConfig::addInstSelector();
  addPass(
      createWebAssemblyISelDag(getWebAssemblyTargetMachine(), getOptLevel()));
  // Run the argument-move pass immediately after the ScheduleDAG scheduler
  // so that we can fix up the ARGUMENT instructions before anything else
  // sees them in the wrong place.
  addPass(createWebAssemblyArgumentMove());
  // Set the p2align operands. This information is present during ISel, however
  // it's inconvenient to collect. Collect it now, and update the immediate
  // operands.
  addPass(createWebAssemblySetP2AlignOperands());

  // Eliminate range checks and add default targets to br_table instructions.
  addPass(createWebAssemblyFixBrTableDefaults());

  // unreachable is terminator, non-terminator instruction after it is not
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Try to expand `vecreduce_{and, or}` into `{any, all}_true`.". Subtarget feature gating influences the behavior here.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Try to expand `vecreduce_{and, or}` into `{any, all}_true`.”。 子目标特性裁剪会影响这里的行为。

### Lines 559-596

```cpp
  // allowed.
  addPass(createWebAssemblyCleanCodeAfterTrap());

  return false;
}

void WebAssemblyPassConfig::addOptimizedRegAlloc() {
  // Currently RegisterCoalesce degrades wasm debug info quality by a
  // significant margin. As a quick fix, disable this for -O1, which is often
  // used for debugging large applications. Disabling this increases code size
  // of Emscripten core benchmarks by ~5%, which is acceptable for -O1, which is
  // usually not used for production builds.
  // TODO Investigate why RegisterCoalesce degrades debug info quality and fix
  // it properly
  if (getOptLevel() == CodeGenOptLevel::Less)
    disablePass(&RegisterCoalescerID);
  TargetPassConfig::addOptimizedRegAlloc();
}

void WebAssemblyPassConfig::addPostRegAlloc() {
  // TODO: The following CodeGen passes don't currently support code containing
  // virtual registers. Consider removing their restrictions and re-enabling
  // them.

  // These functions all require the NoVRegs property.
  disablePass(&MachineLateInstrsCleanupID);
  disablePass(&MachineCopyPropagationID);
  disablePass(&PostRAMachineSinkingID);
  disablePass(&PostRASchedulerID);
  disablePass(&FuncletLayoutID);
  disablePass(&StackMapLivenessID);
  disablePass(&PatchableFunctionID);
  disablePass(&ShrinkWrapID);
  disablePass(&RemoveLoadsIntoFakeUsesID);

  // This pass hurts code size for wasm because it can generate irreducible
  // control flow.
  disablePass(&MachineBlockPlacementID);
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "allowed.". Stack-frame layout or stack access is relevant in this range.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“allowed.”。 该区间与栈帧布局或栈访问相关。

### Lines 597-634

```cpp

  TargetPassConfig::addPostRegAlloc();
}

void WebAssemblyPassConfig::addPreEmitPass() {
  TargetPassConfig::addPreEmitPass();

  // Nullify DBG_VALUE_LISTs that we cannot handle.
  addPass(createWebAssemblyNullifyDebugValueLists());

  // Remove any unreachable blocks that may be left floating around.
  // Rare, but possible. Needed for WebAssemblyFixIrreducibleControlFlow.
  addPass(&UnreachableMachineBlockElimID);

  // Eliminate multiple-entry loops.
  addPass(createWebAssemblyFixIrreducibleControlFlow());

  // Do various transformations for exception handling.
  // Every CFG-changing optimizations should come before this.
  if (TM->Options.ExceptionModel == ExceptionHandling::Wasm)
    addPass(createWebAssemblyLateEHPrepare());

  // Now that we have a prologue and epilogue and all frame indices are
  // rewritten, eliminate SP and FP. This allows them to be stackified,
  // colored, and numbered with the rest of the registers.
  addPass(createWebAssemblyReplacePhysRegs());

  // Preparations and optimizations related to register stackification.
  if (getOptLevel() != CodeGenOptLevel::None) {
    // Depend on LiveIntervals and perform some optimizations on it.
    addPass(createWebAssemblyOptimizeLiveIntervals());

    // Prepare memory intrinsic calls for register stackifying.
    addPass(createWebAssemblyMemIntrinsicResults());
  }

  // Mark registers as representing wasm's value stack. This is a key
  // code-compression technique in WebAssembly. We run this pass (and
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Nullify DBG_VALUE_LISTs that we cannot handle.". Register assignment and register-class constraints matter here.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Nullify DBG_VALUE_LISTs that we cannot handle.”。 这里重点涉及寄存器分配与寄存器类约束。

### Lines 635-672

```cpp
  // MemIntrinsicResults above) very late, so that it sees as much code as
  // possible, including code emitted by PEI and expanded by late tail
  // duplication.
  addPass(createWebAssemblyRegStackify(getOptLevel()));

  if (getOptLevel() != CodeGenOptLevel::None) {
    // Run the register coloring pass to reduce the total number of registers.
    // This runs after stackification so that it doesn't consider registers
    // that become stackified.
    addPass(createWebAssemblyRegColoring());
  }

  // Sort the blocks of the CFG into topological order, a prerequisite for
  // BLOCK and LOOP markers.
  addPass(createWebAssemblyCFGSort());

  // Insert BLOCK and LOOP markers.
  addPass(createWebAssemblyCFGStackify());

  // Insert explicit local.get and local.set operators.
  if (!WasmDisableExplicitLocals)
    addPass(createWebAssemblyExplicitLocals());

  // Lower br_unless into br_if.
  addPass(createWebAssemblyLowerBrUnless());

  // Perform the very last peephole optimizations on the code.
  if (getOptLevel() != CodeGenOptLevel::None)
    addPass(createWebAssemblyPeephole());

  // Create a mapping from LLVM CodeGen virtual registers to wasm registers.
  addPass(createWebAssemblyRegNumbering());

  // Fix debug_values whose defs have been stackified.
  if (!WasmDisableExplicitLocals)
    addPass(createWebAssemblyDebugFixup());

  // Collect information to prepare for MC lowering / asm printing.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "MemIntrinsicResults above) very late, so that it sees as much code as". Stack-frame layout or stack access is relevant in this range.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“MemIntrinsicResults above) very late, so that it sees as much code as”。 该区间与栈帧布局或栈访问相关。

### Lines 673-710

```cpp
  addPass(createWebAssemblyMCLowerPrePass());
}

bool WebAssemblyPassConfig::addPreISel() {
  TargetPassConfig::addPreISel();
  addPass(createWebAssemblyLowerRefTypesIntPtrConv());
  return false;
}

bool WebAssemblyPassConfig::addIRTranslator() {
  addPass(new IRTranslator());
  return false;
}

void WebAssemblyPassConfig::addPreLegalizeMachineIR() {
  if (getOptLevel() != CodeGenOptLevel::None) {
    addPass(createWebAssemblyPreLegalizerCombiner());
  }
}
bool WebAssemblyPassConfig::addLegalizeMachineIR() {
  addPass(new Legalizer());
  return false;
}

void WebAssemblyPassConfig::addPreRegBankSelect() {
  if (getOptLevel() != CodeGenOptLevel::None) {
    addPass(createWebAssemblyPostLegalizerCombiner());
  }
}

bool WebAssemblyPassConfig::addRegBankSelect() {
  addPass(new RegBankSelect());
  return false;
}

bool WebAssemblyPassConfig::addGlobalInstructionSelect() {
  addPass(new InstructionSelect(getOptLevel()));
```
- **EN**: Implements helper routine(s) `addPass`, `createWebAssemblyMCLowerPrePass`, `addPreISel` for this portion of the WebAssembly backend target machine configuration.
- **CN**: 这里实现了 WebAssembly 后端该部分目标机器配置所需的辅助例程 `addPass`, `createWebAssemblyMCLowerPrePass`, `addPreISel`。

### Lines 711-740

```cpp
  // We insert only if ISelDAG won't insert these at a later point.
  if (isGlobalISelAbortEnabled()) {
    addPass(createWebAssemblyArgumentMove());
    addPass(createWebAssemblySetP2AlignOperands());
    addPass(createWebAssemblyFixBrTableDefaults());
    addPass(createWebAssemblyCleanCodeAfterTrap());
  }

  return false;
}

yaml::MachineFunctionInfo *
WebAssemblyTargetMachine::createDefaultFuncInfoYAML() const {
  return new yaml::WebAssemblyFunctionInfo();
}

yaml::MachineFunctionInfo *WebAssemblyTargetMachine::convertFuncInfoToYAML(
    const MachineFunction &MF) const {
  const auto *MFI = MF.getInfo<WebAssemblyFunctionInfo>();
  return new yaml::WebAssemblyFunctionInfo(MF, *MFI);
}

bool WebAssemblyTargetMachine::parseMachineFunctionInfo(
    const yaml::MachineFunctionInfo &MFI, PerFunctionMIParsingState &PFS,
    SMDiagnostic &Error, SMRange &SourceRange) const {
  const auto &YamlMFI = static_cast<const yaml::WebAssemblyFunctionInfo &>(MFI);
  MachineFunction &MF = PFS.MF;
  MF.getInfo<WebAssemblyFunctionInfo>()->initializeBaseYamlFields(MF, YamlMFI);
  return false;
}
```
- **EN**: Implements helper routine(s) `isGlobalISelAbortEnabled`, `addPass`, `createWebAssemblyArgumentMove` for this portion of the WebAssembly backend target machine configuration. This range participates in the GlobalISel pipeline.
- **CN**: 这里实现了 WebAssembly 后端该部分目标机器配置所需的辅助例程 `isGlobalISelAbortEnabled`, `addPass`, `createWebAssemblyArgumentMove`。 这一段参与 GlobalISel 流水线。

## Key Concepts / 关键概念

- WebAssembly backend integration / WebAssembly 后端集成
- Target machine configuration / 目标机器配置
- GlobalISel pipeline / GlobalISel 流水线
- SelectionDAG lowering / SelectionDAG lowering
- Register modeling / 寄存器建模
- Stack frame management / 栈帧管理
- Subtarget features / 子目标特性
- Assembly or MC emission / 汇编或 MC 发射

## Dependencies / 依赖关系

### Direct Includes / 直接包含

- `WebAssemblyTargetMachine.h`
- `MCTargetDesc/WebAssemblyMCTargetDesc.h`
- `TargetInfo/WebAssemblyTargetInfo.h`
- `WebAssembly.h`
- `WebAssemblyISelLowering.h`
- `WebAssemblyMachineFunctionInfo.h`
- `WebAssemblyTargetObjectFile.h`
- `WebAssemblyTargetTransformInfo.h`
- `WebAssemblyUtilities.h`
- `llvm/CodeGen/GlobalISel/IRTranslator.h`
- `llvm/CodeGen/GlobalISel/InstructionSelect.h`
- `llvm/CodeGen/GlobalISel/Legalizer.h`
- `llvm/CodeGen/GlobalISel/RegBankSelect.h`
- `llvm/CodeGen/MIRParser/MIParser.h`
- `llvm/CodeGen/Passes.h`
- `llvm/CodeGen/RegAllocRegistry.h`
- `llvm/CodeGen/TargetPassConfig.h`
- `llvm/IR/Function.h`
- `llvm/InitializePasses.h`
- `llvm/MC/TargetRegistry.h`
- `llvm/Support/Compiler.h`
- `llvm/Target/TargetOptions.h`
- `llvm/Transforms/Scalar.h`
- `llvm/Transforms/Scalar/LowerAtomicPass.h`
- `llvm/Transforms/Utils.h`
- `optional`

### Important Collaborators / 重要协作组件

- LLVM CodeGen layer / LLVM CodeGen 层
- LLVM Support utilities / LLVM Support 工具层
- WebAssembly target-specific helpers / WebAssembly 目标专用辅助组件
