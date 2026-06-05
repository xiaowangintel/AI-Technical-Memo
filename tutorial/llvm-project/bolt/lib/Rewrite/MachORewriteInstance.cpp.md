# MachORewriteInstance.cpp — Code Analysis / 代码分析

## Source / 来源

- File / 文件: `bolt/lib/Rewrite/MachORewriteInstance.cpp`
- Repository / 仓库: `llvm-project`
- Purpose / 用途: MachO rewriter. It also sits in code that implements BOLT rewriting, patching, and binary emission logic. / 该文件实现 BOLT 重写、补丁与二进制输出逻辑。 源码头部说明其职责是：MachO rewriter。

## Line-by-Line Analysis / 逐行分析

### Lines 1-8

```cpp
//===- bolt/Rewrite/MachORewriteInstance.cpp - MachO rewriter -------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 9-26

```cpp
#include "bolt/Rewrite/MachORewriteInstance.h"
#include "bolt/Core/BinaryContext.h"
#include "bolt/Core/BinaryEmitter.h"
#include "bolt/Core/BinaryFunction.h"
#include "bolt/Core/JumpTable.h"
#include "bolt/Core/MCPlusBuilder.h"
#include "bolt/Passes/Instrumentation.h"
#include "bolt/Passes/PatchEntries.h"
#include "bolt/Profile/DataReader.h"
#include "bolt/Rewrite/BinaryPassManager.h"
#include "bolt/Rewrite/ExecutableFileMemoryManager.h"
#include "bolt/Rewrite/JITLinkLinker.h"
#include "bolt/Rewrite/RewriteInstance.h"
#include "bolt/RuntimeLibs/InstrumentationRuntimeLibrary.h"
#include "bolt/Utils/CommandLineOpts.h"
#include "bolt/Utils/Utils.h"
#include "llvm/MC/MCObjectStreamer.h"
#include "llvm/Support/Errc.h"
```

- EN: Pulls in 18 header(s) from local project, LLVM dependencies needed by this range. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里引入 18 个头文件，为本段提供本地模块、LLVM 或系统层面的依赖。这里的注释记录了设计假设、不变量或使用说明。

### Lines 27-44

```cpp
#include "llvm/Support/FileSystem.h"
#include "llvm/Support/ToolOutputFile.h"
#include <memory>
#include <optional>

namespace opts {

using namespace llvm;
extern cl::opt<unsigned> AlignText;
// FIXME! Upstream change
// extern cl::opt<bool> CheckOverlappingElements;
extern cl::opt<bool> Instrument;
extern cl::opt<bool> InstrumentCalls;
extern cl::opt<bolt::JumpTableSupportLevel> JumpTables;
extern cl::opt<bool> KeepTmp;
extern cl::opt<bool> NeverPrint;
extern cl::opt<std::string> OutputFilename;
extern cl::opt<bool> PrintAfterBranchFixup;
```

- EN: Pulls in 4 header(s) from LLVM, system dependencies needed by this range. Works inside namespace scope `opts`, `llvm` to organize symbols. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `opts`, `llvm`.
- CN: 这里引入 4 个头文件，为本段提供本地模块、LLVM 或系统层面的依赖。这里位于命名空间 `opts`, `llvm` 中，用于组织符号作用域。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `opts`, `llvm`。

### Lines 45-54

```cpp
extern cl::opt<bool> PrintFinalized;
extern cl::opt<bool> PrintNormalized;
extern cl::opt<bool> PrintReordered;
extern cl::opt<bool> PrintSections;
extern cl::opt<bool> PrintDisasm;
extern cl::opt<bool> PrintCFG;
extern cl::opt<std::string> RuntimeInstrumentationLib;
extern cl::opt<unsigned> Verbosity;
} // namespace opts
```

- EN: Works inside namespace scope `opts` to organize symbols. Notable symbols here include `opts`.
- CN: 这里位于命名空间 `opts` 中，用于组织符号作用域。这里较值得关注的符号包括 `opts`。

### Lines 55-70

```cpp
namespace llvm {
namespace bolt {

#define DEBUG_TYPE "bolt"

Expected<std::unique_ptr<MachORewriteInstance>>
MachORewriteInstance::create(object::MachOObjectFile *InputFile,
                             StringRef ToolPath) {
  Error Err = Error::success();
  auto MachORI =
      std::make_unique<MachORewriteInstance>(InputFile, ToolPath, Err);
  if (Err)
    return std::move(Err);
  return std::move(MachORI);
}
```

- EN: Establishes header-guard or prologue macros so the file can be compiled safely. Works inside namespace scope `llvm`, `bolt` to organize symbols. Declares or implements routines including `success`. Defines macros such as `DEBUG_TYPE` for constants or compile-time switches.
- CN: 这里建立头文件保护或前导宏，保证源码可被安全编译与重复包含控制。这里位于命名空间 `llvm`, `bolt` 中，用于组织符号作用域。这里声明或实现函数，例如 `success`。这里定义宏 `DEBUG_TYPE`，用于常量或编译期开关。

### Lines 71-88

```cpp
MachORewriteInstance::MachORewriteInstance(object::MachOObjectFile *InputFile,
                                           StringRef ToolPath, Error &Err)
    : InputFile(InputFile), ToolPath(ToolPath) {
  ErrorAsOutParameter EAO(&Err);
  Relocation::Arch = InputFile->makeTriple().getArch();
  auto BCOrErr = BinaryContext::createBinaryContext(
      InputFile->makeTriple(), std::make_shared<orc::SymbolStringPool>(),
      InputFile->getFileName(), nullptr,
      /* IsPIC */ true, DWARFContext::create(*InputFile),
      {llvm::outs(), llvm::errs()});
  if (Error E = BCOrErr.takeError()) {
    Err = std::move(E);
    return;
  }
  BC = std::move(BCOrErr.get());
  BC->initializeTarget(std::unique_ptr<MCPlusBuilder>(
      createMCPlusBuilder(BC->TheTriple->getArch(), BC->MIA.get(),
                          BC->MII.get(), BC->MRI.get(), BC->STI.get())));
```

- EN: Declares or implements routines including `InputFile`, `EAO`, `makeTriple`, `getFileName`, `outs`, and 2 more. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `InputFile`, `EAO`, `makeTriple`, `getFileName`, `outs`, `move`.
- CN: 这里声明或实现函数，例如 `InputFile`, `EAO`, `makeTriple`, `getFileName`, `outs`, and 2 more。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `InputFile`, `EAO`, `makeTriple`, `getFileName`, `outs`, `move`。

### Lines 89-96

```cpp
  if (opts::Instrument)
    BC->setRuntimeLibrary(std::make_unique<InstrumentationRuntimeLibrary>());
}

Error MachORewriteInstance::setProfile(StringRef Filename) {
  if (!sys::fs::exists(Filename))
    return errorCodeToError(make_error_code(errc::no_such_file_or_directory));
```

- EN: Declares or implements routines including `setRuntimeLibrary`, `setProfile`. Notable symbols here include `setRuntimeLibrary`, `setProfile`.
- CN: 这里声明或实现函数，例如 `setRuntimeLibrary`, `setProfile`。这里较值得关注的符号包括 `setRuntimeLibrary`, `setProfile`。

### Lines 97-107

```cpp
  if (ProfileReader) {
    // Already exists
    return make_error<StringError>(
        Twine("multiple profiles specified: ") + ProfileReader->getFilename() +
        " and " + Filename, inconvertibleErrorCode());
  }

  ProfileReader = std::make_unique<DataReader>(Filename);
  return Error::success();
}
```

- EN: Declares or implements routines including `Twine`, `inconvertibleErrorCode`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `Twine`, `inconvertibleErrorCode`.
- CN: 这里声明或实现函数，例如 `Twine`, `inconvertibleErrorCode`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `Twine`, `inconvertibleErrorCode`。

### Lines 108-121

```cpp
void MachORewriteInstance::preprocessProfileData() {
  if (!ProfileReader)
    return;
  if (Error E = ProfileReader->preprocessProfile(*BC))
    report_error("cannot pre-process profile", std::move(E));
}

void MachORewriteInstance::processProfileDataPreCFG() {
  if (!ProfileReader)
    return;
  if (Error E = ProfileReader->readProfilePreCFG(*BC))
    report_error("cannot read profile pre-CFG", std::move(E));
}
```

- EN: Declares or implements routines including `preprocessProfileData`, `report_error`, `processProfileDataPreCFG`. Notable symbols here include `preprocessProfileData`, `report_error`, `processProfileDataPreCFG`.
- CN: 这里声明或实现函数，例如 `preprocessProfileData`, `report_error`, `processProfileDataPreCFG`。这里较值得关注的符号包括 `preprocessProfileData`, `report_error`, `processProfileDataPreCFG`。

### Lines 122-139

```cpp
void MachORewriteInstance::processProfileData() {
  if (!ProfileReader)
    return;
  if (Error E = ProfileReader->readProfile(*BC))
    report_error("cannot read profile", std::move(E));
}

void MachORewriteInstance::readSpecialSections() {
  for (const object::SectionRef &Section : InputFile->sections()) {
    Expected<StringRef> SectionName = Section.getName();;
    check_error(SectionName.takeError(), "cannot get section name");
    // Only register sections with names.
    if (!SectionName->empty()) {
      BC->registerSection(Section);
      LLVM_DEBUG(
          dbgs() << "BOLT-DEBUG: registering section " << *SectionName
                 << " @ 0x" << Twine::utohexstr(Section.getAddress()) << ":0x"
                 << Twine::utohexstr(Section.getAddress() + Section.getSize())
```

- EN: Declares or implements routines including `processProfileData`, `report_error`, `readSpecialSections`, `check_error`, `registerSection`, and 2 more. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `processProfileData`, `report_error`, `readSpecialSections`, `check_error`, `registerSection`, `dbgs`.
- CN: 这里声明或实现函数，例如 `processProfileData`, `report_error`, `readSpecialSections`, `check_error`, `registerSection`, and 2 more。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `processProfileData`, `report_error`, `readSpecialSections`, `check_error`, `registerSection`, `dbgs`。

### Lines 140-149

```cpp
                 << "\n");
    }
  }

  if (opts::PrintSections) {
    outs() << "BOLT-INFO: Sections from original binary:\n";
    BC->printSections(outs());
  }
}
```

- EN: Declares or implements routines including `outs`, `printSections`. Notable symbols here include `outs`, `printSections`.
- CN: 这里声明或实现函数，例如 `outs`, `printSections`。这里较值得关注的符号包括 `outs`, `printSections`。

### Lines 150-158

```cpp
namespace {

struct DataInCodeRegion {
  explicit DataInCodeRegion(DiceRef D) {
    D.getOffset(Offset);
    D.getLength(Length);
    D.getKind(Kind);
  }
```

- EN: Introduces type definitions such as `DataInCodeRegion`. Declares or implements routines including `DataInCodeRegion`. Notable symbols here include `DataInCodeRegion`.
- CN: 这里引入类型定义，例如 `DataInCodeRegion`。这里声明或实现函数，例如 `DataInCodeRegion`。这里较值得关注的符号包括 `DataInCodeRegion`。

### Lines 159-176

```cpp
  uint32_t Offset;
  uint16_t Length;
  uint16_t Kind;
};

std::vector<DataInCodeRegion> readDataInCode(const MachOObjectFile &O) {
  const MachO::linkedit_data_command DataInCodeLC =
      O.getDataInCodeLoadCommand();
  const uint32_t NumberOfEntries =
      DataInCodeLC.datasize / sizeof(MachO::data_in_code_entry);
  std::vector<DataInCodeRegion> DataInCode;
  DataInCode.reserve(NumberOfEntries);
  for (auto I = O.begin_dices(), E = O.end_dices(); I != E; ++I)
    DataInCode.emplace_back(*I);
  llvm::stable_sort(DataInCode, [](DataInCodeRegion LHS, DataInCodeRegion RHS) {
    return LHS.Offset < RHS.Offset;
  });
  return DataInCode;
```

- EN: Declares or implements routines including `readDataInCode`, `sizeof`, `stable_sort`. Notable symbols here include `readDataInCode`, `sizeof`, `stable_sort`.
- CN: 这里声明或实现函数，例如 `readDataInCode`, `sizeof`, `stable_sort`。这里较值得关注的符号包括 `readDataInCode`, `sizeof`, `stable_sort`。

### Lines 177-194

```cpp
}

std::optional<uint64_t> readStartAddress(const MachOObjectFile &O) {
  std::optional<uint64_t> StartOffset;
  std::optional<uint64_t> TextVMAddr;
  for (const object::MachOObjectFile::LoadCommandInfo &LC : O.load_commands()) {
    switch (LC.C.cmd) {
    case MachO::LC_MAIN: {
      MachO::entry_point_command LCMain = O.getEntryPointCommand(LC);
      StartOffset = LCMain.entryoff;
      break;
    }
    case MachO::LC_SEGMENT: {
      MachO::segment_command LCSeg = O.getSegmentLoadCommand(LC);
      StringRef SegmentName(LCSeg.segname,
                            strnlen(LCSeg.segname, sizeof(LCSeg.segname)));
      if (SegmentName == "__TEXT")
        TextVMAddr = LCSeg.vmaddr;
```

- EN: Declares or implements routines including `readStartAddress`, `strnlen`. Notable symbols here include `readStartAddress`, `strnlen`.
- CN: 这里声明或实现函数，例如 `readStartAddress`, `strnlen`。这里较值得关注的符号包括 `readStartAddress`, `strnlen`。

### Lines 195-212

```cpp
      break;
    }
    case MachO::LC_SEGMENT_64: {
      MachO::segment_command_64 LCSeg = O.getSegment64LoadCommand(LC);
      StringRef SegmentName(LCSeg.segname,
                            strnlen(LCSeg.segname, sizeof(LCSeg.segname)));
      if (SegmentName == "__TEXT")
        TextVMAddr = LCSeg.vmaddr;
      break;
    }
    default:
      continue;
    }
  }
  return (TextVMAddr && StartOffset)
             ? std::optional<uint64_t>(*TextVMAddr + *StartOffset)
             : std::nullopt;
}
```

- EN: Declares or implements routines including `strnlen`. Notable symbols here include `strnlen`.
- CN: 这里声明或实现函数，例如 `strnlen`。这里较值得关注的符号包括 `strnlen`。

### Lines 213-230

```cpp

} // anonymous namespace

void MachORewriteInstance::discoverFileObjects() {
  std::vector<SymbolRef> FunctionSymbols;
  for (const SymbolRef &S : InputFile->symbols()) {
    SymbolRef::Type Type = cantFail(S.getType(), "cannot get symbol type");
    if (Type == SymbolRef::ST_Function)
      FunctionSymbols.push_back(S);
  }
  if (FunctionSymbols.empty())
    return;
  llvm::stable_sort(
      FunctionSymbols, [](const SymbolRef &LHS, const SymbolRef &RHS) {
        return cantFail(LHS.getValue()) < cantFail(RHS.getValue());
      });
  for (size_t Index = 0; Index < FunctionSymbols.size(); ++Index) {
    const uint64_t Address = cantFail(FunctionSymbols[Index].getValue());
```

- EN: Works inside namespace scope `void` to organize symbols. Declares or implements routines including `discoverFileObjects`, `cantFail`. Notable symbols here include `discoverFileObjects`, `cantFail`, `void`.
- CN: 这里位于命名空间 `void` 中，用于组织符号作用域。这里声明或实现函数，例如 `discoverFileObjects`, `cantFail`。这里较值得关注的符号包括 `discoverFileObjects`, `cantFail`, `void`。

### Lines 231-239

```cpp
    ErrorOr<BinarySection &> Section = BC->getSectionForAddress(Address);
    // TODO: It happens for some symbols (e.g. __mh_execute_header).
    // Add proper logic to handle them correctly.
    if (!Section) {
      errs() << "BOLT-WARNING: no section found for address " << Address
             << "\n";
      continue;
    }
```

- EN: Declares or implements routines including `getSectionForAddress`, `errs`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `getSectionForAddress`, `errs`.
- CN: 这里声明或实现函数，例如 `getSectionForAddress`, `errs`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `getSectionForAddress`, `errs`。

### Lines 240-249

```cpp
    std::string SymbolName =
        cantFail(FunctionSymbols[Index].getName(), "cannot get symbol name")
            .str();
    // Uniquify names of local symbols.
    if (!(cantFail(FunctionSymbols[Index].getFlags()) & SymbolRef::SF_Global))
      SymbolName = NR.uniquify(SymbolName);

    section_iterator S = cantFail(FunctionSymbols[Index].getSection());
    uint64_t EndAddress = S->getAddress() + S->getSize();
```

- EN: Declares or implements routines including `cantFail`, `getAddress`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `cantFail`, `getAddress`.
- CN: 这里声明或实现函数，例如 `cantFail`, `getAddress`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `cantFail`, `getAddress`。

### Lines 250-258

```cpp
    size_t NFIndex = Index + 1;
    // Skip aliases.
    while (NFIndex < FunctionSymbols.size() &&
           cantFail(FunctionSymbols[NFIndex].getValue()) == Address)
      ++NFIndex;
    if (NFIndex < FunctionSymbols.size() &&
        S == cantFail(FunctionSymbols[NFIndex].getSection()))
      EndAddress = cantFail(FunctionSymbols[NFIndex].getValue());
```

- EN: Declares or implements routines including `cantFail`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `cantFail`.
- CN: 这里声明或实现函数，例如 `cantFail`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `cantFail`。

### Lines 259-266

```cpp
    const uint64_t SymbolSize = EndAddress - Address;
    const auto It = BC->getBinaryFunctions().find(Address);
    if (It == BC->getBinaryFunctions().end()) {
      BinaryFunction *Function = BC->createBinaryFunction(
          std::move(SymbolName), *Section, Address, SymbolSize);
      if (!opts::Instrument)
        Function->setOutputAddress(Function->getAddress());
```

- EN: Declares or implements routines including `getBinaryFunctions`, `move`, `setOutputAddress`. Notable symbols here include `getBinaryFunctions`, `move`, `setOutputAddress`.
- CN: 这里声明或实现函数，例如 `getBinaryFunctions`, `move`, `setOutputAddress`。这里较值得关注的符号包括 `getBinaryFunctions`, `move`, `setOutputAddress`。

### Lines 267-277

```cpp
    } else {
      It->second.addAlternativeName(std::move(SymbolName));
    }
  }

  const std::vector<DataInCodeRegion> DataInCode = readDataInCode(*InputFile);

  for (auto &BFI : BC->getBinaryFunctions()) {
    BinaryFunction &Function = BFI.second;
    Function.setMaxSize(Function.getSize());
```

- EN: Declares or implements routines including `readDataInCode`. Notable symbols here include `readDataInCode`.
- CN: 这里声明或实现函数，例如 `readDataInCode`。这里较值得关注的符号包括 `readDataInCode`。

### Lines 278-290

```cpp
    ErrorOr<ArrayRef<uint8_t>> FunctionData = Function.getData();
    if (!FunctionData) {
      errs() << "BOLT-ERROR: corresponding section is non-executable or "
             << "empty for function " << Function << '\n';
      continue;
    }

    // Treat zero-sized functions as non-simple ones.
    if (Function.getSize() == 0) {
      Function.setSimple(false);
      continue;
    }
```

- EN: Declares or implements routines including `errs`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `errs`.
- CN: 这里声明或实现函数，例如 `errs`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `errs`。

### Lines 291-305

```cpp
    // Offset of the function in the file.
    const auto *FileBegin =
        reinterpret_cast<const uint8_t *>(InputFile->getData().data());
    Function.setFileOffset(FunctionData->begin() - FileBegin);

    // Treat functions which contain data in code as non-simple ones.
    const auto It = std::lower_bound(
        DataInCode.cbegin(), DataInCode.cend(), Function.getFileOffset(),
        [](DataInCodeRegion D, uint64_t Offset) { return D.Offset < Offset; });
    if (It != DataInCode.cend() &&
        It->Offset + It->Length <=
            Function.getFileOffset() + Function.getMaxSize())
      Function.setSimple(false);
  }
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 306-319

```cpp
  BC->StartFunctionAddress = readStartAddress(*InputFile);
}

void MachORewriteInstance::disassembleFunctions() {
  for (auto &BFI : BC->getBinaryFunctions()) {
    BinaryFunction &Function = BFI.second;
    if (!Function.isSimple())
      continue;
    BC->logBOLTErrorsAndQuitOnFatal(Function.disassemble());
    if (opts::PrintDisasm)
      Function.print(outs(), "after disassembly");
  }
}
```

- EN: Declares or implements routines including `readStartAddress`, `disassembleFunctions`, `logBOLTErrorsAndQuitOnFatal`. Notable symbols here include `readStartAddress`, `disassembleFunctions`, `logBOLTErrorsAndQuitOnFatal`.
- CN: 这里声明或实现函数，例如 `readStartAddress`, `disassembleFunctions`, `logBOLTErrorsAndQuitOnFatal`。这里较值得关注的符号包括 `readStartAddress`, `disassembleFunctions`, `logBOLTErrorsAndQuitOnFatal`。

### Lines 320-328

```cpp
void MachORewriteInstance::buildFunctionsCFG() {
  for (auto &BFI : BC->getBinaryFunctions()) {
    BinaryFunction &Function = BFI.second;
    if (!Function.isSimple())
      continue;
    BC->logBOLTErrorsAndQuitOnFatal(Function.buildCFG(/*AllocId*/ 0));
  }
}
```

- EN: Declares or implements routines including `buildFunctionsCFG`, `logBOLTErrorsAndQuitOnFatal`. Notable symbols here include `buildFunctionsCFG`, `logBOLTErrorsAndQuitOnFatal`.
- CN: 这里声明或实现函数，例如 `buildFunctionsCFG`, `logBOLTErrorsAndQuitOnFatal`。这里较值得关注的符号包括 `buildFunctionsCFG`, `logBOLTErrorsAndQuitOnFatal`。

### Lines 329-339

```cpp
void MachORewriteInstance::postProcessFunctions() {
  for (auto &BFI : BC->getBinaryFunctions()) {
    BinaryFunction &Function = BFI.second;
    if (Function.empty())
      continue;
    Function.postProcessCFG();
    if (opts::PrintCFG)
      Function.print(outs(), "after building cfg");
  }
}
```

- EN: Declares or implements routines including `postProcessFunctions`. Notable symbols here include `postProcessFunctions`.
- CN: 这里声明或实现函数，例如 `postProcessFunctions`。这里较值得关注的符号包括 `postProcessFunctions`。

### Lines 340-348

```cpp
void MachORewriteInstance::runOptimizationPasses() {
  BinaryFunctionPassManager Manager(*BC);
  if (opts::Instrument) {
    Manager.registerPass(std::make_unique<PatchEntries>());
    Manager.registerPass(std::make_unique<Instrumentation>(opts::NeverPrint));
  }

  Manager.registerPass(std::make_unique<ShortenInstructions>(opts::NeverPrint));
```

- EN: Declares or implements routines including `runOptimizationPasses`, `Manager`. Notable symbols here include `runOptimizationPasses`, `Manager`.
- CN: 这里声明或实现函数，例如 `runOptimizationPasses`, `Manager`。这里较值得关注的符号包括 `runOptimizationPasses`, `Manager`。

### Lines 349-361

```cpp
  Manager.registerPass(std::make_unique<RemoveNops>(opts::NeverPrint));

  Manager.registerPass(std::make_unique<NormalizeCFG>(opts::PrintNormalized));

  Manager.registerPass(
      std::make_unique<ReorderBasicBlocks>(opts::PrintReordered));
  Manager.registerPass(
      std::make_unique<FixupBranches>(opts::PrintAfterBranchFixup));
  Manager.registerPass(std::make_unique<PopulateOutputFunctions>());
  // This pass should always run last.*
  Manager.registerPass(
      std::make_unique<FinalizeFunctions>(opts::PrintFinalized));
```

- EN: Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里的注释记录了设计假设、不变量或使用说明。

### Lines 362-378

```cpp
  BC->logBOLTErrorsAndQuitOnFatal(Manager.runPasses());
}

void MachORewriteInstance::mapInstrumentationSection(
    StringRef SectionName, BOLTLinker::SectionMapper MapSection) {
  if (!opts::Instrument)
    return;
  ErrorOr<BinarySection &> Section = BC->getUniqueSectionByName(SectionName);
  if (!Section) {
    llvm::errs() << "Cannot find " + SectionName + " section\n";
    exit(1);
  }
  if (!Section->hasValidSectionID())
    return;
  MapSection(*Section, Section->getAddress());
}
```

- EN: Declares or implements routines including `logBOLTErrorsAndQuitOnFatal`, `getUniqueSectionByName`, `errs`, `exit`, `MapSection`. Notable symbols here include `logBOLTErrorsAndQuitOnFatal`, `getUniqueSectionByName`, `errs`, `exit`, `MapSection`.
- CN: 这里声明或实现函数，例如 `logBOLTErrorsAndQuitOnFatal`, `getUniqueSectionByName`, `errs`, `exit`, `MapSection`。这里较值得关注的符号包括 `logBOLTErrorsAndQuitOnFatal`, `getUniqueSectionByName`, `errs`, `exit`, `MapSection`。

### Lines 379-392

```cpp
void MachORewriteInstance::mapCodeSections(
    BOLTLinker::SectionMapper MapSection) {
  for (BinaryFunction *Function : BC->getAllBinaryFunctions()) {
    if (!Function->isEmitted())
      continue;
    if (Function->getOutputAddress() == 0)
      continue;
    ErrorOr<BinarySection &> FuncSection = Function->getCodeSection();
    if (!FuncSection)
      report_error(
          (Twine("Cannot find section for function ") + Function->getOneName())
              .str(),
          FuncSection.getError());
```

- EN: Declares or implements routines including `getCodeSection`. Notable symbols here include `getCodeSection`.
- CN: 这里声明或实现函数，例如 `getCodeSection`。这里较值得关注的符号包括 `getCodeSection`。

### Lines 393-401

```cpp
    FuncSection->setOutputAddress(Function->getOutputAddress());
    LLVM_DEBUG(dbgs() << "BOLT: mapping 0x"
                 << Twine::utohexstr(FuncSection->getAllocAddress()) << " to 0x"
                 << Twine::utohexstr(Function->getOutputAddress()) << '\n');
    MapSection(*FuncSection, Function->getOutputAddress());
    Function->setImageAddress(FuncSection->getAllocAddress());
    Function->setImageSize(FuncSection->getOutputSize());
  }
```

- EN: Declares or implements routines including `setOutputAddress`, `LLVM_DEBUG`, `utohexstr`, `MapSection`, `setImageAddress`, and 1 more. Notable symbols here include `setOutputAddress`, `LLVM_DEBUG`, `utohexstr`, `MapSection`, `setImageAddress`, `setImageSize`.
- CN: 这里声明或实现函数，例如 `setOutputAddress`, `LLVM_DEBUG`, `utohexstr`, `MapSection`, `setImageAddress`, and 1 more。这里较值得关注的符号包括 `setOutputAddress`, `LLVM_DEBUG`, `utohexstr`, `MapSection`, `setImageAddress`, `setImageSize`。

### Lines 402-419

```cpp
  if (opts::Instrument) {
    ErrorOr<BinarySection &> BOLT = BC->getUniqueSectionByName("__bolt");
    if (!BOLT) {
      llvm::errs() << "Cannot find __bolt section\n";
      exit(1);
    }
    uint64_t Addr = BOLT->getAddress();
    for (BinaryFunction *Function : BC->getAllBinaryFunctions()) {
      if (!Function->isEmitted())
        continue;
      if (Function->getOutputAddress() != 0)
        continue;
      ErrorOr<BinarySection &> FuncSection = Function->getCodeSection();
      assert(FuncSection && "cannot find section for function");
      Addr = llvm::alignTo(Addr, 4);
      FuncSection->setOutputAddress(Addr);
      MapSection(*FuncSection, Addr);
      Function->setFileOffset(Addr - BOLT->getAddress() +
```

- EN: Declares or implements routines including `getUniqueSectionByName`, `errs`, `exit`, `getAddress`, `getCodeSection`, and 5 more. Notable symbols here include `getUniqueSectionByName`, `errs`, `exit`, `getAddress`, `getCodeSection`, `assert`.
- CN: 这里声明或实现函数，例如 `getUniqueSectionByName`, `errs`, `exit`, `getAddress`, `getCodeSection`, and 5 more。这里较值得关注的符号包括 `getUniqueSectionByName`, `errs`, `exit`, `getAddress`, `getCodeSection`, `assert`。

### Lines 420-428

```cpp
                              BOLT->getInputFileOffset());
      Function->setImageAddress(FuncSection->getAllocAddress());
      Function->setImageSize(FuncSection->getOutputSize());
      BC->registerNameAtAddress(Function->getOneName(), Addr, 0, 0);
      Addr += FuncSection->getOutputSize();
    }
  }
}
```

- EN: Declares or implements routines including `getInputFileOffset`, `setImageAddress`, `setImageSize`, `registerNameAtAddress`, `getOutputSize`. Notable symbols here include `getInputFileOffset`, `setImageAddress`, `setImageSize`, `registerNameAtAddress`, `getOutputSize`.
- CN: 这里声明或实现函数，例如 `getInputFileOffset`, `setImageAddress`, `setImageSize`, `registerNameAtAddress`, `getOutputSize`。这里较值得关注的符号包括 `getInputFileOffset`, `setImageAddress`, `setImageSize`, `registerNameAtAddress`, `getOutputSize`。

### Lines 429-438

```cpp
void MachORewriteInstance::emitAndLink() {
  std::error_code EC;
  std::unique_ptr<::llvm::ToolOutputFile> TempOut =
      std::make_unique<::llvm::ToolOutputFile>(
          opts::OutputFilename + ".bolt.o", EC, sys::fs::OF_None);
  check_error(EC, "cannot create output object file");

  if (opts::KeepTmp)
    TempOut->keep();
```

- EN: Declares or implements routines including `emitAndLink`, `check_error`, `keep`. Notable symbols here include `emitAndLink`, `check_error`, `keep`.
- CN: 这里声明或实现函数，例如 `emitAndLink`, `check_error`, `keep`。这里较值得关注的符号包括 `emitAndLink`, `check_error`, `keep`。

### Lines 439-446

```cpp
  std::unique_ptr<buffer_ostream> BOS =
      std::make_unique<buffer_ostream>(TempOut->os());
  raw_pwrite_stream *OS = BOS.get();
  auto Streamer = BC->createStreamer(*OS);

  emitBinaryContext(*Streamer, *BC, getOrgSecPrefix());
  Streamer->finish();
```

- EN: Declares or implements routines including `createStreamer`, `emitBinaryContext`, `finish`. Notable symbols here include `createStreamer`, `emitBinaryContext`, `finish`.
- CN: 这里声明或实现函数，例如 `createStreamer`, `emitBinaryContext`, `finish`。这里较值得关注的符号包括 `createStreamer`, `emitBinaryContext`, `finish`。

### Lines 447-457

```cpp
  std::unique_ptr<MemoryBuffer> ObjectMemBuffer =
      MemoryBuffer::getMemBuffer(BOS->str(), "in-memory object file", false);
  std::unique_ptr<object::ObjectFile> Obj = cantFail(
      object::ObjectFile::createObjectFile(ObjectMemBuffer->getMemBufferRef()),
      "error creating in-memory object");
  assert(Obj && "createObjectFile cannot return nullptr");

  auto EFMM = std::make_unique<ExecutableFileMemoryManager>(*BC);
  EFMM->setNewSecPrefix(getNewSecPrefix());
  EFMM->setOrgSecPrefix(getOrgSecPrefix());
```

- EN: Declares or implements routines including `getMemBuffer`, `createObjectFile`, `assert`, `setNewSecPrefix`, `setOrgSecPrefix`. Notable symbols here include `getMemBuffer`, `createObjectFile`, `assert`, `setNewSecPrefix`, `setOrgSecPrefix`.
- CN: 这里声明或实现函数，例如 `getMemBuffer`, `createObjectFile`, `assert`, `setNewSecPrefix`, `setOrgSecPrefix`。这里较值得关注的符号包括 `getMemBuffer`, `createObjectFile`, `assert`, `setNewSecPrefix`, `setOrgSecPrefix`。

### Lines 458-470

```cpp
  Linker = std::make_unique<JITLinkLinker>(*BC, std::move(EFMM));
  Linker->loadObject(ObjectMemBuffer->getMemBufferRef(),
                     [this](auto MapSection) {
                       // Assign addresses to all sections. If key corresponds
                       // to the object created by ourselves, call our regular
                       // mapping function. If we are loading additional objects
                       // as part of runtime libraries for instrumentation,
                       // treat them as extra sections.
                       mapCodeSections(MapSection);
                       mapInstrumentationSection("__counters", MapSection);
                       mapInstrumentationSection("__tables", MapSection);
                     });
```

- EN: Declares or implements routines including `loadObject`, `mapCodeSections`, `mapInstrumentationSection`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `loadObject`, `mapCodeSections`, `mapInstrumentationSection`.
- CN: 这里声明或实现函数，例如 `loadObject`, `mapCodeSections`, `mapInstrumentationSection`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `loadObject`, `mapCodeSections`, `mapInstrumentationSection`。

### Lines 471-484

```cpp
  // TODO: Refactor addRuntimeLibSections to work properly on Mach-O
  // and use it here.
  // if (auto *RtLibrary = BC->getRuntimeLibrary()) {
  //   RtLibrary->link(*BC, ToolPath, *Linker, [this](auto MapSection) {
  //     mapInstrumentationSection("I__setup", MapSection);
  //     mapInstrumentationSection("I__fini", MapSection);
  //     mapInstrumentationSection("I__data", MapSection);
  //     mapInstrumentationSection("I__text", MapSection);
  //     mapInstrumentationSection("I__cstring", MapSection);
  //     mapInstrumentationSection("I__literal16", MapSection);
  //   });
  // }
}
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 485-502

```cpp
void MachORewriteInstance::writeInstrumentationSection(StringRef SectionName,
                                                       raw_pwrite_stream &OS) {
  if (!opts::Instrument)
    return;
  ErrorOr<BinarySection &> Section = BC->getUniqueSectionByName(SectionName);
  if (!Section) {
    llvm::errs() << "Cannot find " + SectionName + " section\n";
    exit(1);
  }
  if (!Section->hasValidSectionID())
    return;
  assert(Section->getInputFileOffset() &&
         "Section input offset cannot be zero");
  assert(Section->getAllocAddress() && "Section alloc address cannot be zero");
  assert(Section->getOutputSize() && "Section output size cannot be zero");
  OS.pwrite(reinterpret_cast<char *>(Section->getAllocAddress()),
            Section->getOutputSize(), Section->getInputFileOffset());
}
```

- EN: Declares or implements routines including `getUniqueSectionByName`, `errs`, `exit`, `assert`, `getOutputSize`. Notable symbols here include `getUniqueSectionByName`, `errs`, `exit`, `assert`, `getOutputSize`.
- CN: 这里声明或实现函数，例如 `getUniqueSectionByName`, `errs`, `exit`, `assert`, `getOutputSize`。这里较值得关注的符号包括 `getUniqueSectionByName`, `errs`, `exit`, `assert`, `getOutputSize`。

### Lines 503-511

```cpp

void MachORewriteInstance::rewriteFile() {
  std::error_code EC;
  Out = std::make_unique<ToolOutputFile>(opts::OutputFilename, EC,
                                         sys::fs::OF_None);
  check_error(EC, "cannot create output executable file");
  raw_fd_ostream &OS = Out->os();
  OS << InputFile->getData();
```

- EN: Declares or implements routines including `rewriteFile`, `check_error`, `os`, `getData`. Notable symbols here include `rewriteFile`, `check_error`, `os`, `getData`.
- CN: 这里声明或实现函数，例如 `rewriteFile`, `check_error`, `os`, `getData`。这里较值得关注的符号包括 `rewriteFile`, `check_error`, `os`, `getData`。

### Lines 512-524

```cpp
  for (auto &BFI : BC->getBinaryFunctions()) {
    BinaryFunction &Function = BFI.second;
    if (!Function.isSimple())
      continue;
    assert(Function.isEmitted() && "Simple function has not been emitted");
    if (!opts::Instrument && (Function.getImageSize() > Function.getMaxSize()))
      continue;
    if (opts::Verbosity >= 2)
      outs() << "BOLT: rewriting function \"" << Function << "\"\n";
    OS.pwrite(reinterpret_cast<char *>(Function.getImageAddress()),
              Function.getImageSize(), Function.getFileOffset());
  }
```

- EN: Declares or implements routines including `assert`, `outs`. Notable symbols here include `assert`, `outs`.
- CN: 这里声明或实现函数，例如 `assert`, `outs`。这里较值得关注的符号包括 `assert`, `outs`。

### Lines 525-532

```cpp
  for (const BinaryFunction *Function : BC->getInjectedBinaryFunctions()) {
    OS.pwrite(reinterpret_cast<char *>(Function->getImageAddress()),
              Function->getImageSize(), Function->getFileOffset());
  }

  writeInstrumentationSection("__counters", OS);
  writeInstrumentationSection("__tables", OS);
```

- EN: Declares or implements routines including `getImageSize`, `writeInstrumentationSection`. Notable symbols here include `getImageSize`, `writeInstrumentationSection`.
- CN: 这里声明或实现函数，例如 `getImageSize`, `writeInstrumentationSection`。这里较值得关注的符号包括 `getImageSize`, `writeInstrumentationSection`。

### Lines 533-541

```cpp
  // TODO: Refactor addRuntimeLibSections to work properly on Mach-O and
  // use it here.
  writeInstrumentationSection("I__setup", OS);
  writeInstrumentationSection("I__fini", OS);
  writeInstrumentationSection("I__data", OS);
  writeInstrumentationSection("I__text", OS);
  writeInstrumentationSection("I__cstring", OS);
  writeInstrumentationSection("I__literal16", OS);
```

- EN: Declares or implements routines including `writeInstrumentationSection`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `writeInstrumentationSection`.
- CN: 这里声明或实现函数，例如 `writeInstrumentationSection`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `writeInstrumentationSection`。

### Lines 542-549

```cpp
  Out->keep();
  EC = sys::fs::setPermissions(
      opts::OutputFilename,
      static_cast<sys::fs::perms>(sys::fs::perms::all_all &
                                  ~sys::fs::getUmask()));
  check_error(EC, "cannot set permissions of output file");
}
```

- EN: Declares or implements routines including `keep`, `getUmask`, `check_error`. Notable symbols here include `keep`, `getUmask`, `check_error`.
- CN: 这里声明或实现函数，例如 `keep`, `getUmask`, `check_error`。这里较值得关注的符号包括 `keep`, `getUmask`, `check_error`。

### Lines 550-561

```cpp
void MachORewriteInstance::adjustCommandLineOptions() {
//FIXME! Upstream change
//  opts::CheckOverlappingElements = false;
  if (!opts::AlignText.getNumOccurrences())
    opts::AlignText = BC->PageAlign;
  if (opts::Instrument.getNumOccurrences())
    opts::ForcePatch = true;
  opts::JumpTables = JTS_MOVE;
  opts::InstrumentCalls = false;
  opts::RuntimeInstrumentationLib = "libbolt_rt_instr_osx.a";
}
```

- EN: Declares or implements routines including `adjustCommandLineOptions`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `adjustCommandLineOptions`.
- CN: 这里声明或实现函数，例如 `adjustCommandLineOptions`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `adjustCommandLineOptions`。

### Lines 562-570

```cpp
void MachORewriteInstance::run() {
  adjustCommandLineOptions();

  readSpecialSections();

  discoverFileObjects();

  preprocessProfileData();
```

- EN: Declares or implements routines including `run`, `adjustCommandLineOptions`, `readSpecialSections`, `discoverFileObjects`, `preprocessProfileData`. Notable symbols here include `run`, `adjustCommandLineOptions`, `readSpecialSections`, `discoverFileObjects`, `preprocessProfileData`.
- CN: 这里声明或实现函数，例如 `run`, `adjustCommandLineOptions`, `readSpecialSections`, `discoverFileObjects`, `preprocessProfileData`。这里较值得关注的符号包括 `run`, `adjustCommandLineOptions`, `readSpecialSections`, `discoverFileObjects`, `preprocessProfileData`。

### Lines 571-578

```cpp
  disassembleFunctions();

  processProfileDataPreCFG();

  buildFunctionsCFG();

  processProfileData();
```

- EN: Declares or implements routines including `disassembleFunctions`, `processProfileDataPreCFG`, `buildFunctionsCFG`, `processProfileData`. Notable symbols here include `disassembleFunctions`, `processProfileDataPreCFG`, `buildFunctionsCFG`, `processProfileData`.
- CN: 这里声明或实现函数，例如 `disassembleFunctions`, `processProfileDataPreCFG`, `buildFunctionsCFG`, `processProfileData`。这里较值得关注的符号包括 `disassembleFunctions`, `processProfileDataPreCFG`, `buildFunctionsCFG`, `processProfileData`。

### Lines 579-587

```cpp
  postProcessFunctions();

  runOptimizationPasses();

  emitAndLink();

  rewriteFile();
}
```

- EN: Declares or implements routines including `postProcessFunctions`, `runOptimizationPasses`, `emitAndLink`, `rewriteFile`. Notable symbols here include `postProcessFunctions`, `runOptimizationPasses`, `emitAndLink`, `rewriteFile`.
- CN: 这里声明或实现函数，例如 `postProcessFunctions`, `runOptimizationPasses`, `emitAndLink`, `rewriteFile`。这里较值得关注的符号包括 `postProcessFunctions`, `runOptimizationPasses`, `emitAndLink`, `rewriteFile`。

### Lines 588-591

```cpp
MachORewriteInstance::~MachORewriteInstance() {}

} // namespace bolt
} // namespace llvm
```

- EN: Works inside namespace scope `bolt`, `llvm` to organize symbols. Declares or implements routines including `MachORewriteInstance`. Notable symbols here include `MachORewriteInstance`, `bolt`, `llvm`.
- CN: 这里位于命名空间 `bolt`, `llvm` 中，用于组织符号作用域。这里声明或实现函数，例如 `MachORewriteInstance`。这里较值得关注的符号包括 `MachORewriteInstance`, `bolt`, `llvm`。

## Key Concepts / 关键概念

- `DataInCodeRegion`: class or struct interface / 类或结构体接口
- `success`: function or method entry point / 函数或方法入口
- `InputFile`: function or method entry point / 函数或方法入口
- `EAO`: function or method entry point / 函数或方法入口
- `makeTriple`: function or method entry point / 函数或方法入口
- `getFileName`: function or method entry point / 函数或方法入口
- `DEBUG_TYPE`: macro or compile-time switch / 宏或编译期开关
- `opts`: namespace scope / 命名空间作用域

## Dependencies / 依赖关系

- Local headers / 本地头文件: `bolt/Rewrite/MachORewriteInstance.h`, `bolt/Core/BinaryContext.h`, `bolt/Core/BinaryEmitter.h`, `bolt/Core/BinaryFunction.h`, `bolt/Core/JumpTable.h`, `bolt/Core/MCPlusBuilder.h`, `bolt/Passes/Instrumentation.h`, `bolt/Passes/PatchEntries.h`, `bolt/Profile/DataReader.h`, `bolt/Rewrite/BinaryPassManager.h`, `bolt/Rewrite/ExecutableFileMemoryManager.h`, `bolt/Rewrite/JITLinkLinker.h`, `bolt/Rewrite/RewriteInstance.h`, `bolt/RuntimeLibs/InstrumentationRuntimeLibrary.h`, `bolt/Utils/CommandLineOpts.h`, `bolt/Utils/Utils.h`
- LLVM headers / LLVM 头文件: `llvm/MC/MCObjectStreamer.h`, `llvm/Support/Errc.h`, `llvm/Support/FileSystem.h`, `llvm/Support/ToolOutputFile.h`
- System headers / 系统头文件: `memory`, `optional`
- Directory context / 目录上下文: `bolt/lib/Rewrite` neighbors usually cooperate with this file to provide the surrounding subsystem / `bolt/lib/Rewrite` 下的相邻文件通常与本文件协作组成对应子系统
