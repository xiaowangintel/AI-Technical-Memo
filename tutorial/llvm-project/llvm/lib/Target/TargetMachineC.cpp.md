# TargetMachineC.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `llvm/lib/Target/TargetMachineC.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 作用**:
  - **EN**: Defines the target machine, data layout, pass pipeline hooks, and subtarget creation logic.
  - **CN**: 定义目标机器、数据布局、Pass 流水线钩子以及子目标创建逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-11
```cpp
//===-- TargetMachine.cpp -------------------------------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file implements the LLVM-C part of TargetMachine.h
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM file banner, license notice, and high-level file description.
- **CN**: 包含 LLVM 标准文件头、许可证声明以及该文件的高层说明。

### Lines 12-29
```cpp

#include "llvm-c/Core.h"
#include "llvm-c/TargetMachine.h"
#include "llvm/Analysis/TargetTransformInfo.h"
#include "llvm/IR/DataLayout.h"
#include "llvm/IR/LegacyPassManager.h"
#include "llvm/IR/Module.h"
#include "llvm/MC/TargetRegistry.h"
#include "llvm/Support/CBindingWrapping.h"
#include "llvm/Support/FileSystem.h"
#include "llvm/Support/raw_ostream.h"
#include "llvm/Target/CodeGenCWrappers.h"
#include "llvm/Target/TargetMachine.h"
#include "llvm/TargetParser/Host.h"
#include "llvm/TargetParser/SubtargetFeature.h"
#include <cstring>
#include <optional>

```
- **EN**: Pulls in the headers needed for this implementation, including `llvm-c/Core.h`, `llvm-c/TargetMachine.h`, `llvm/Analysis/TargetTransformInfo.h`, `llvm/IR/DataLayout.h`.
- **CN**: 引入该实现所需的头文件，其中包括 `llvm-c/Core.h`, `llvm-c/TargetMachine.h`, `llvm/Analysis/TargetTransformInfo.h`, `llvm/IR/DataLayout.h`。

### Lines 30-44
```cpp
using namespace llvm;

namespace llvm {

/// Options for LLVMCreateTargetMachine().
struct LLVMTargetMachineOptions {
  std::string CPU;
  std::string Features;
  std::string ABI;
  CodeGenOptLevel OL = CodeGenOptLevel::Default;
  std::optional<Reloc::Model> RM;
  std::optional<CodeModel::Model> CM;
  bool JIT;
};

```
- **EN**: Introduces declarations for `llvm`, `LLVMTargetMachineOptions`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `llvm`, `LLVMTargetMachineOptions` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 45-62
```cpp
} // namespace llvm

DEFINE_SIMPLE_CONVERSION_FUNCTIONS(LLVMTargetMachineOptions,
                                   LLVMTargetMachineOptionsRef)

static TargetMachine *unwrap(LLVMTargetMachineRef P) {
  return reinterpret_cast<TargetMachine *>(P);
}
static Target *unwrap(LLVMTargetRef P) {
  return reinterpret_cast<Target*>(P);
}
static LLVMTargetMachineRef wrap(const TargetMachine *P) {
  return reinterpret_cast<LLVMTargetMachineRef>(const_cast<TargetMachine *>(P));
}
static LLVMTargetRef wrap(const Target * P) {
  return reinterpret_cast<LLVMTargetRef>(const_cast<Target*>(P));
}

```
- **EN**: Introduces declarations for `llvm`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `llvm` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 63-74
```cpp
LLVMTargetRef LLVMGetFirstTarget() {
  if (TargetRegistry::targets().begin() == TargetRegistry::targets().end()) {
    return nullptr;
  }

  const Target *target = &*TargetRegistry::targets().begin();
  return wrap(target);
}
LLVMTargetRef LLVMGetNextTarget(LLVMTargetRef T) {
  return wrap(unwrap(T)->getNext());
}

```
- **EN**: Implements logic around `targets`, `wrap`; this block applies conditional target rules; returns target-specific results.
- **CN**: 围绕 `targets`, `wrap` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果。

### Lines 75-85
```cpp
LLVMTargetRef LLVMGetTargetFromName(const char *Name) {
  StringRef NameRef = Name;
  auto I = find_if(TargetRegistry::targets(),
                   [&](const Target &T) { return T.getName() == NameRef; });
  return I != TargetRegistry::targets().end() ? wrap(&*I) : nullptr;
}

LLVMBool LLVMGetTargetFromTriple(const char* TripleStr, LLVMTargetRef *T,
                                 char **ErrorMessage) {
  std::string Error;

```
- **EN**: Implements logic around `find_if`, `getName`, `targets`; this block returns target-specific results.
- **CN**: 围绕 `find_if`, `getName`, `targets` 实现具体逻辑；这一段返回目标相关结果。

### Lines 86-95
```cpp
  Triple TT(TripleStr);
  *T = wrap(TargetRegistry::lookupTarget(TT, Error));

  if (!*T) {
    if (ErrorMessage)
      *ErrorMessage = strdup(Error.c_str());

    return 1;
  }

```
- **EN**: Implements logic around `TT`, `wrap`, `strdup`; this block applies conditional target rules; returns target-specific results.
- **CN**: 围绕 `TT`, `wrap`, `strdup` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果。

### Lines 96-106
```cpp
  return 0;
}

const char * LLVMGetTargetName(LLVMTargetRef T) {
  return unwrap(T)->getName();
}

const char * LLVMGetTargetDescription(LLVMTargetRef T) {
  return unwrap(T)->getShortDescription();
}

```
- **EN**: Implements logic around `unwrap`; this block returns target-specific results.
- **CN**: 围绕 `unwrap` 实现具体逻辑；这一段返回目标相关结果。

### Lines 107-118
```cpp
LLVMBool LLVMTargetHasJIT(LLVMTargetRef T) {
  return unwrap(T)->hasJIT();
}

LLVMBool LLVMTargetHasTargetMachine(LLVMTargetRef T) {
  return unwrap(T)->hasTargetMachine();
}

LLVMBool LLVMTargetHasAsmBackend(LLVMTargetRef T) {
  return unwrap(T)->hasMCAsmBackend();
}

```
- **EN**: Implements logic around `unwrap`; this block returns target-specific results.
- **CN**: 围绕 `unwrap` 实现具体逻辑；这一段返回目标相关结果。

### Lines 119-131
```cpp
LLVMTargetMachineOptionsRef LLVMCreateTargetMachineOptions(void) {
  return wrap(new LLVMTargetMachineOptions());
}

void LLVMDisposeTargetMachineOptions(LLVMTargetMachineOptionsRef Options) {
  delete unwrap(Options);
}

void LLVMTargetMachineOptionsSetCPU(LLVMTargetMachineOptionsRef Options,
                                    const char *CPU) {
  unwrap(Options)->CPU = CPU;
}

```
- **EN**: Implements logic around `wrap`, `unwrap`; this block returns target-specific results.
- **CN**: 围绕 `wrap`, `unwrap` 实现具体逻辑；这一段返回目标相关结果。

### Lines 132-141
```cpp
void LLVMTargetMachineOptionsSetFeatures(LLVMTargetMachineOptionsRef Options,
                                         const char *Features) {
  unwrap(Options)->Features = Features;
}

void LLVMTargetMachineOptionsSetABI(LLVMTargetMachineOptionsRef Options,
                                    const char *ABI) {
  unwrap(Options)->ABI = ABI;
}

```
- **EN**: Implements logic around `unwrap`.
- **CN**: 围绕 `unwrap` 实现具体逻辑。

### Lines 142-159
```cpp
void LLVMTargetMachineOptionsSetCodeGenOptLevel(
    LLVMTargetMachineOptionsRef Options, LLVMCodeGenOptLevel Level) {
  CodeGenOptLevel OL;

  switch (Level) {
  case LLVMCodeGenLevelNone:
    OL = CodeGenOptLevel::None;
    break;
  case LLVMCodeGenLevelLess:
    OL = CodeGenOptLevel::Less;
    break;
  case LLVMCodeGenLevelAggressive:
    OL = CodeGenOptLevel::Aggressive;
    break;
  case LLVMCodeGenLevelDefault:
    OL = CodeGenOptLevel::Default;
    break;
  }
```
- **EN**: Implements target-specific case analysis using a `switch` over opcodes, fixups, or enum values.
- **CN**: 通过对 opcode、fixup 或枚举值执行 `switch` 分析来实现目标相关逻辑。

### Lines 160-177
```cpp

  unwrap(Options)->OL = OL;
}

void LLVMTargetMachineOptionsSetRelocMode(LLVMTargetMachineOptionsRef Options,
                                          LLVMRelocMode Reloc) {
  std::optional<Reloc::Model> RM;

  switch (Reloc) {
  case LLVMRelocStatic:
    RM = Reloc::Static;
    break;
  case LLVMRelocPIC:
    RM = Reloc::PIC_;
    break;
  case LLVMRelocDynamicNoPic:
    RM = Reloc::DynamicNoPIC;
    break;
```
- **EN**: Implements logic around `unwrap`; this block uses `switch`-based dispatch; maps fixups or relocations.
- **CN**: 围绕 `unwrap` 实现具体逻辑；这一段使用 `switch` 分派，映射 fixup 或重定位。

### Lines 178-190
```cpp
  case LLVMRelocROPI:
    RM = Reloc::ROPI;
    break;
  case LLVMRelocRWPI:
    RM = Reloc::RWPI;
    break;
  case LLVMRelocROPI_RWPI:
    RM = Reloc::ROPI_RWPI;
    break;
  case LLVMRelocDefault:
    break;
  }

```
- **EN**: Contains supporting implementation details for the surrounding backend logic.
- **CN**: 包含周边后端逻辑所需的辅助实现细节。

### Lines 191-199
```cpp
  unwrap(Options)->RM = RM;
}

void LLVMTargetMachineOptionsSetCodeModel(LLVMTargetMachineOptionsRef Options,
                                          LLVMCodeModel CodeModel) {
  auto CM = unwrap(CodeModel, unwrap(Options)->JIT);
  unwrap(Options)->CM = CM;
}

```
- **EN**: Implements logic around `unwrap`.
- **CN**: 围绕 `unwrap` 实现具体逻辑。

### Lines 200-210
```cpp
LLVMTargetMachineRef
LLVMCreateTargetMachineWithOptions(LLVMTargetRef T, const char *TripleStr,
                                   LLVMTargetMachineOptionsRef Options) {
  auto *Opt = unwrap(Options);
  TargetOptions TO;
  TO.MCOptions.ABIName = Opt->ABI;
  return wrap(unwrap(T)->createTargetMachine(Triple(TripleStr), Opt->CPU,
                                             Opt->Features, TO, Opt->RM,
                                             Opt->CM, Opt->OL, Opt->JIT));
}

```
- **EN**: Implements logic around `unwrap`, `wrap`; this block returns target-specific results.
- **CN**: 围绕 `unwrap`, `wrap` 实现具体逻辑；这一段返回目标相关结果。

### Lines 211-222
```cpp
LLVMTargetMachineRef
LLVMCreateTargetMachine(LLVMTargetRef T, const char *Triple, const char *CPU,
                        const char *Features, LLVMCodeGenOptLevel Level,
                        LLVMRelocMode Reloc, LLVMCodeModel CodeModel) {
  auto *Options = LLVMCreateTargetMachineOptions();

  LLVMTargetMachineOptionsSetCPU(Options, CPU);
  LLVMTargetMachineOptionsSetFeatures(Options, Features);
  LLVMTargetMachineOptionsSetCodeGenOptLevel(Options, Level);
  LLVMTargetMachineOptionsSetRelocMode(Options, Reloc);
  LLVMTargetMachineOptionsSetCodeModel(Options, CodeModel);

```
- **EN**: Provides registration, factory, or initialization glue that connects this file to the wider backend.
- **CN**: 提供注册、工厂或初始化胶水代码，把本文件接入更大的后端体系。

### Lines 223-235
```cpp
  auto *Machine = LLVMCreateTargetMachineWithOptions(T, Triple, Options);

  LLVMDisposeTargetMachineOptions(Options);
  return Machine;
}

void LLVMDisposeTargetMachine(LLVMTargetMachineRef T) { delete unwrap(T); }

LLVMTargetRef LLVMGetTargetMachineTarget(LLVMTargetMachineRef T) {
  const Target* target = &(unwrap(T)->getTarget());
  return wrap(target);
}

```
- **EN**: Implements logic around `unwrap`, `wrap`; this block returns target-specific results.
- **CN**: 围绕 `unwrap`, `wrap` 实现具体逻辑；这一段返回目标相关结果。

### Lines 236-245
```cpp
char* LLVMGetTargetMachineTriple(LLVMTargetMachineRef T) {
  std::string StringRep = unwrap(T)->getTargetTriple().str();
  return strdup(StringRep.c_str());
}

char* LLVMGetTargetMachineCPU(LLVMTargetMachineRef T) {
  std::string StringRep = std::string(unwrap(T)->getTargetCPU());
  return strdup(StringRep.c_str());
}

```
- **EN**: Implements logic around `unwrap`, `strdup`, `string`; this block returns target-specific results.
- **CN**: 围绕 `unwrap`, `strdup`, `string` 实现具体逻辑；这一段返回目标相关结果。

### Lines 246-255
```cpp
char* LLVMGetTargetMachineFeatureString(LLVMTargetMachineRef T) {
  std::string StringRep = std::string(unwrap(T)->getTargetFeatureString());
  return strdup(StringRep.c_str());
}

void LLVMSetTargetMachineAsmVerbosity(LLVMTargetMachineRef T,
                                      LLVMBool VerboseAsm) {
  unwrap(T)->Options.MCOptions.AsmVerbose = VerboseAsm;
}

```
- **EN**: Implements logic around `string`, `strdup`, `unwrap`; this block returns target-specific results.
- **CN**: 围绕 `string`, `strdup`, `unwrap` 实现具体逻辑；这一段返回目标相关结果。

### Lines 256-273
```cpp
void LLVMSetTargetMachineFastISel(LLVMTargetMachineRef T, LLVMBool Enable) {
  unwrap(T)->setFastISel(Enable);
}

void LLVMSetTargetMachineGlobalISel(LLVMTargetMachineRef T, LLVMBool Enable) {
  unwrap(T)->setGlobalISel(Enable);
}

void LLVMSetTargetMachineGlobalISelAbort(LLVMTargetMachineRef T,
                                         LLVMGlobalISelAbortMode Mode) {
  GlobalISelAbortMode AM = GlobalISelAbortMode::Enable;
  switch (Mode) {
  case LLVMGlobalISelAbortDisable:
    AM = GlobalISelAbortMode::Disable;
    break;
  case LLVMGlobalISelAbortEnable:
    AM = GlobalISelAbortMode::Enable;
    break;
```
- **EN**: Implements logic around `unwrap`; this block uses `switch`-based dispatch.
- **CN**: 围绕 `unwrap` 实现具体逻辑；这一段使用 `switch` 分派。

### Lines 274-286
```cpp
  case LLVMGlobalISelAbortDisableWithDiag:
    AM = GlobalISelAbortMode::DisableWithDiag;
    break;
  }

  unwrap(T)->setGlobalISelAbort(AM);
}

void LLVMSetTargetMachineMachineOutliner(LLVMTargetMachineRef T,
                                         LLVMBool Enable) {
  unwrap(T)->setMachineOutliner(Enable);
}

```
- **EN**: Implements logic around `unwrap`.
- **CN**: 围绕 `unwrap` 实现具体逻辑。

### Lines 287-297
```cpp
LLVMTargetDataRef LLVMCreateTargetDataLayout(LLVMTargetMachineRef T) {
  return wrap(new DataLayout(unwrap(T)->createDataLayout()));
}

static LLVMBool LLVMTargetMachineEmit(LLVMTargetMachineRef T, LLVMModuleRef M,
                                      raw_pwrite_stream &OS,
                                      LLVMCodeGenFileType codegen,
                                      char **ErrorMessage) {
  TargetMachine* TM = unwrap(T);
  Module* Mod = unwrap(M);

```
- **EN**: Implements logic around `wrap`, `unwrap`; this block returns target-specific results.
- **CN**: 围绕 `wrap`, `unwrap` 实现具体逻辑；这一段返回目标相关结果。

### Lines 298-315
```cpp
  legacy::PassManager pass;

  std::string error;

  Mod->setDataLayout(TM->createDataLayout());

  CodeGenFileType ft;
  switch (codegen) {
    case LLVMAssemblyFile:
      ft = CodeGenFileType::AssemblyFile;
      break;
    default:
      ft = CodeGenFileType::ObjectFile;
      break;
  }
  if (TM->addPassesToEmitFile(pass, OS, nullptr, ft)) {
    error = "TargetMachine can't emit a file of this type";
    *ErrorMessage = strdup(error.c_str());
```
- **EN**: Implements logic around `setDataLayout`, `strdup`; this block uses `switch`-based dispatch; applies conditional target rules.
- **CN**: 围绕 `setDataLayout`, `strdup` 实现具体逻辑；这一段使用 `switch` 分派，应用条件化的目标规则。

### Lines 316-324
```cpp
    return true;
  }

  pass.run(*Mod);

  OS.flush();
  return false;
}

```
- **EN**: Implements logic around `run`, `flush`; this block returns target-specific results.
- **CN**: 围绕 `run`, `flush` 实现具体逻辑；这一段返回目标相关结果。

### Lines 325-339
```cpp
LLVMBool LLVMTargetMachineEmitToFile(LLVMTargetMachineRef T, LLVMModuleRef M,
                                     const char *Filename,
                                     LLVMCodeGenFileType codegen,
                                     char **ErrorMessage) {
  std::error_code EC;
  raw_fd_ostream dest(Filename, EC, sys::fs::OF_None);
  if (EC) {
    *ErrorMessage = strdup(EC.message().c_str());
    return true;
  }
  bool Result = LLVMTargetMachineEmit(T, M, dest, codegen, ErrorMessage);
  dest.flush();
  return Result;
}

```
- **EN**: Implements logic around `dest`, `strdup`, `flush`; this block applies conditional target rules; returns target-specific results.
- **CN**: 围绕 `dest`, `strdup`, `flush` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果。

### Lines 340-352
```cpp
LLVMBool LLVMTargetMachineEmitToMemoryBuffer(LLVMTargetMachineRef T,
  LLVMModuleRef M, LLVMCodeGenFileType codegen, char** ErrorMessage,
  LLVMMemoryBufferRef *OutMemBuf) {
  SmallString<0> CodeString;
  raw_svector_ostream OStream(CodeString);
  bool Result = LLVMTargetMachineEmit(T, M, OStream, codegen, ErrorMessage);

  StringRef Data = OStream.str();
  *OutMemBuf =
      LLVMCreateMemoryBufferWithMemoryRangeCopy(Data.data(), Data.size(), "");
  return Result;
}

```
- **EN**: Implements logic around `OStream`, `str`; this block returns target-specific results.
- **CN**: 围绕 `OStream`, `str` 实现具体逻辑；这一段返回目标相关结果。

### Lines 353-364
```cpp
char *LLVMGetDefaultTargetTriple(void) {
  return strdup(sys::getDefaultTargetTriple().c_str());
}

char *LLVMNormalizeTargetTriple(const char* triple) {
  return strdup(Triple::normalize(StringRef(triple)).c_str());
}

char *LLVMGetHostCPUName(void) {
  return strdup(sys::getHostCPUName().data());
}

```
- **EN**: Implements logic around `strdup`; this block returns target-specific results.
- **CN**: 围绕 `strdup` 实现具体逻辑；这一段返回目标相关结果。

### Lines 365-376
```cpp
char *LLVMGetHostCPUFeatures(void) {
  SubtargetFeatures Features;
  for (const auto &[Feature, IsEnabled] : sys::getHostCPUFeatures())
    Features.AddFeature(Feature, IsEnabled);

  return strdup(Features.getString().c_str());
}

void LLVMAddAnalysisPasses(LLVMTargetMachineRef T, LLVMPassManagerRef PM) {
  unwrap(PM)->add(
      createTargetTransformInfoWrapperPass(unwrap(T)->getTargetIRAnalysis()));
}
```
- **EN**: Implements logic around `AddFeature`, `strdup`, `unwrap`, `createTargetTransformInfoWrapperPass`; this block returns target-specific results.
- **CN**: 围绕 `AddFeature`, `strdup`, `unwrap`, `createTargetTransformInfoWrapperPass` 实现具体逻辑；这一段返回目标相关结果。

## Key Concepts / 关键概念

- **Target configuration / 目标配置**:
  - **EN**: Owns data layout, passes, and per-function subtarget selection
  - **CN**: 管理数据布局、Pass 与按函数的子目标选择
- **Fixups and relocations / Fixup 与重定位**:
  - **EN**: Bridges symbolic references to concrete relocation records
  - **CN**: 把符号引用连接到具体重定位记录

## Dependencies / 依赖关系

- **Direct includes / 直接包含**: `llvm-c/Core.h`, `llvm-c/TargetMachine.h`, `llvm/Analysis/TargetTransformInfo.h`, `llvm/IR/DataLayout.h`, `llvm/IR/LegacyPassManager.h`, `llvm/IR/Module.h`, `llvm/MC/TargetRegistry.h`, `llvm/Support/CBindingWrapping.h`, `llvm/Support/FileSystem.h`, `llvm/Support/raw_ostream.h`, `llvm/Target/CodeGenCWrappers.h`, `llvm/Target/TargetMachine.h` ... (+4 more)
- **LLVM subsystems / LLVM 子系统**: MC, IR, Support
