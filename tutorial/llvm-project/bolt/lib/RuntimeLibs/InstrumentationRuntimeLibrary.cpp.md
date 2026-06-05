# InstrumentationRuntimeLibrary.cpp — Code Analysis / 代码分析

## Source / 来源

- File / 文件: `bolt/lib/RuntimeLibs/InstrumentationRuntimeLibrary.cpp`
- Repository / 仓库: `llvm-project`
- Purpose / 用途: bolt/RuntimeLibs/InstrumentationRuntimeLibrary.cpp This file implements the InstrumentationRuntimeLibrary class.. It also sits in code that belongs to the BOLT post-link optimizer subsystem. / 该文件属于 BOLT 链接后优化器子系统。 源码头部说明其职责是：bolt/RuntimeLibs/InstrumentationRuntimeLibrary.cpp This file implements the InstrumentationRuntimeLibrary class.。

## Line-by-Line Analysis / 逐行分析

### Lines 1-12

```cpp
//===- bolt/RuntimeLibs/InstrumentationRuntimeLibrary.cpp -----------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file implements the InstrumentationRuntimeLibrary class.
//
//===----------------------------------------------------------------------===//
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 13-21

```cpp
#include "bolt/RuntimeLibs/InstrumentationRuntimeLibrary.h"
#include "bolt/Core/BinaryFunction.h"
#include "bolt/Core/JumpTable.h"
#include "bolt/Core/Linker.h"
#include "bolt/Utils/CommandLineOpts.h"
#include "llvm/MC/MCStreamer.h"
#include "llvm/Support/Alignment.h"
#include "llvm/Support/CommandLine.h"
```

- EN: Pulls in 8 header(s) from local project, LLVM dependencies needed by this range. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里引入 8 个头文件，为本段提供本地模块、LLVM 或系统层面的依赖。这里的注释记录了设计假设、不变量或使用说明。

### Lines 22-31

```cpp
using namespace llvm;
using namespace bolt;

namespace opts {

cl::opt<std::string> RuntimeInstrumentationLib(
    "runtime-instrumentation-lib",
    cl::desc("specify path of the runtime instrumentation library"),
    cl::init("libbolt_rt_instr.a"), cl::cat(BoltOptCategory));
```

- EN: Works inside namespace scope `llvm`, `bolt`, `opts` to organize symbols. Declares or implements routines including `desc`, `init`. Notable symbols here include `desc`, `init`, `llvm`, `bolt`, `opts`.
- CN: 这里位于命名空间 `llvm`, `bolt`, `opts` 中，用于组织符号作用域。这里声明或实现函数，例如 `desc`, `init`。这里较值得关注的符号包括 `desc`, `init`, `llvm`, `bolt`, `opts`。

### Lines 32-41

```cpp
extern cl::opt<bool> InstrumentationFileAppendPID;
extern cl::opt<bool> ConservativeInstrumentation;
extern cl::opt<std::string> InstrumentationFilename;
extern cl::opt<std::string> InstrumentationBinpath;
extern cl::opt<uint32_t> InstrumentationMaxSize;
extern cl::opt<uint32_t> InstrumentationSleepTime;
extern cl::opt<bool> InstrumentationNoCountersClear;
extern cl::opt<bool> InstrumentationWaitForks;
extern cl::opt<JumpTableSupportLevel> JumpTables;
```

- EN: Continues the surrounding implementation, wiring declarations, state, and helper logic together.
- CN: 这里继续展开周边实现，衔接声明、状态与辅助逻辑。

### Lines 42-59

```cpp
} // namespace opts

void InstrumentationRuntimeLibrary::adjustCommandLineOptions(
    const BinaryContext &BC) const {
  if (!BC.HasRelocations) {
    errs() << "BOLT-ERROR: instrumentation runtime libraries require "
              "relocations\n";
    exit(1);
  }
  if (opts::JumpTables != JTS_MOVE) {
    opts::JumpTables = JTS_MOVE;
    outs() << "BOLT-INFO: forcing -jump-tables=move for instrumentation\n";
  }
  if (!BC.StartFunctionAddress) {
    errs() << "BOLT-ERROR: instrumentation runtime libraries require a known "
              "entry point of "
              "the input binary\n";
    exit(1);
```

- EN: Works inside namespace scope `opts` to organize symbols. Declares or implements routines including `errs`, `exit`, `outs`. Notable symbols here include `errs`, `exit`, `outs`, `opts`.
- CN: 这里位于命名空间 `opts` 中，用于组织符号作用域。这里声明或实现函数，例如 `errs`, `exit`, `outs`。这里较值得关注的符号包括 `errs`, `exit`, `outs`, `opts`。

### Lines 60-68

```cpp
  }

  if (BC.IsStaticExecutable && !opts::InstrumentationSleepTime) {
    errs() << "BOLT-ERROR: instrumentation of static binary currently does not "
              "support profile output on binary finalization, so it "
              "requires -instrumentation-sleep-time=N (N>0) usage\n";
    exit(1);
  }
```

- EN: Declares or implements routines including `errs`, `exit`. Notable symbols here include `errs`, `exit`.
- CN: 这里声明或实现函数，例如 `errs`, `exit`。这里较值得关注的符号包括 `errs`, `exit`。

### Lines 69-77

```cpp
  if (opts::InstrumentationWaitForks && opts::InstrumentationFileAppendPID) {
    errs() << "BOLT-ERROR: instrumentation-file-append-pid is not compatible "
              "with instrumentation-wait-forks. If you want a separate profile "
              "for each fork, it can only be dumped in the end of process when "
              "instrumentation-file-append-pid is used.\n";
    exit(1);
  }
}
```

- EN: Declares or implements routines including `errs`, `exit`. Notable symbols here include `errs`, `exit`.
- CN: 这里声明或实现函数，例如 `errs`, `exit`。这里较值得关注的符号包括 `errs`, `exit`。

### Lines 78-86

```cpp
void InstrumentationRuntimeLibrary::emitBinary(BinaryContext &BC,
                                               MCStreamer &Streamer) {
  MCSection *Section = BC.isELF()
                           ? static_cast<MCSection *>(BC.Ctx->getELFSection(
                                 ".bolt.instr.counters", ELF::SHT_PROGBITS,
                                 BinarySection::getFlags(/*IsReadOnly=*/false,
                                                         /*IsText=*/false,
                                                         /*IsAllocatable=*/true)
```

- EN: Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里的注释记录了设计假设、不变量或使用说明。

### Lines 87-94

```cpp
                                     ))
                           : static_cast<MCSection *>(BC.Ctx->getMachOSection(
                                 "__BOLT", "__counters", MachO::S_REGULAR,
                                 SectionKind::getData()));

  Section->setAlignment(llvm::Align(BC.RegularPageSize));
  Streamer.switchSection(Section);
```

- EN: Declares or implements routines including `getData`, `setAlignment`. Notable symbols here include `getData`, `setAlignment`.
- CN: 这里声明或实现函数，例如 `getData`, `setAlignment`。这里较值得关注的符号包括 `getData`, `setAlignment`。

### Lines 95-103

```cpp
  // EmitOffset is used to determine padding size for data alignment
  uint64_t EmitOffset = 0;

  auto emitLabel = [&Streamer](MCSymbol *Symbol, bool IsGlobal = true) {
    Streamer.emitLabel(Symbol);
    if (IsGlobal)
      Streamer.emitSymbolAttribute(Symbol, MCSymbolAttr::MCSA_Global);
  };
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 104-117

```cpp
  auto emitLabelByName = [&BC, emitLabel](StringRef Name,
                                          bool IsGlobal = true) {
    MCSymbol *Symbol = BC.Ctx->getOrCreateSymbol(Name);
    emitLabel(Symbol, IsGlobal);
  };

  auto emitPadding = [&Streamer, &EmitOffset](unsigned Size) {
    const uint64_t Padding = alignTo(EmitOffset, Size) - EmitOffset;
    if (Padding) {
      Streamer.emitFill(Padding, 0);
      EmitOffset += Padding;
    }
  };
```

- EN: Declares or implements routines including `getOrCreateSymbol`, `emitLabel`, `alignTo`. Notable symbols here include `getOrCreateSymbol`, `emitLabel`, `alignTo`.
- CN: 这里声明或实现函数，例如 `getOrCreateSymbol`, `emitLabel`, `alignTo`。这里较值得关注的符号包括 `getOrCreateSymbol`, `emitLabel`, `alignTo`。

### Lines 118-133

```cpp
  auto emitDataSize = [&EmitOffset](unsigned Size) { EmitOffset += Size; };

  auto emitDataPadding = [emitPadding, emitDataSize](unsigned Size) {
    emitPadding(Size);
    emitDataSize(Size);
  };

  auto emitFill = [&Streamer, emitDataSize,
                   emitLabel](unsigned Size, MCSymbol *Symbol = nullptr,
                              uint8_t Byte = 0) {
    emitDataSize(Size);
    if (Symbol)
      emitLabel(Symbol, /*IsGlobal*/ false);
    Streamer.emitFill(Size, Byte);
  };
```

- EN: Declares or implements routines including `emitPadding`, `emitDataSize`, `emitLabel`. Notable symbols here include `emitPadding`, `emitDataSize`, `emitLabel`.
- CN: 这里声明或实现函数，例如 `emitPadding`, `emitDataSize`, `emitLabel`。这里较值得关注的符号包括 `emitPadding`, `emitDataSize`, `emitLabel`。

### Lines 134-144

```cpp
  auto emitValue = [&BC, &Streamer, emitDataPadding,
                    emitLabel](MCSymbol *Symbol, const MCExpr *Value) {
    const unsigned Psize = BC.AsmInfo->getCodePointerSize();
    emitDataPadding(Psize);
    emitLabel(Symbol);
    if (Value)
      Streamer.emitValue(Value, Psize);
    else
      Streamer.emitFill(Psize, 0);
  };
```

- EN: Declares or implements routines including `getCodePointerSize`, `emitDataPadding`, `emitLabel`. Notable symbols here include `getCodePointerSize`, `emitDataPadding`, `emitLabel`.
- CN: 这里声明或实现函数，例如 `getCodePointerSize`, `emitDataPadding`, `emitLabel`。这里较值得关注的符号包括 `getCodePointerSize`, `emitDataPadding`, `emitLabel`。

### Lines 145-159

```cpp
  auto emitIntValue = [&Streamer, emitDataPadding, emitLabelByName](
                          StringRef Name, uint64_t Value, unsigned Size = 4) {
    emitDataPadding(Size);
    emitLabelByName(Name);
    Streamer.emitIntValue(Value, Size);
  };

  auto emitString = [&Streamer, emitDataSize, emitLabelByName,
                     emitFill](StringRef Name, StringRef Contents) {
    emitDataSize(Contents.size());
    emitLabelByName(Name);
    Streamer.emitBytes(Contents);
    emitFill(1);
  };
```

- EN: Declares or implements routines including `emitDataPadding`, `emitLabelByName`, `emitDataSize`, `emitFill`. Notable symbols here include `emitDataPadding`, `emitLabelByName`, `emitDataSize`, `emitFill`.
- CN: 这里声明或实现函数，例如 `emitDataPadding`, `emitLabelByName`, `emitDataSize`, `emitFill`。这里较值得关注的符号包括 `emitDataPadding`, `emitLabelByName`, `emitDataSize`, `emitFill`。

### Lines 160-167

```cpp
  // All of the following symbols will be exported as globals to be used by the
  // instrumentation runtime library to dump the instrumentation data to disk.
  // Label marking start of the memory region containing instrumentation
  // counters, total vector size is Counters.size() 8-byte counters
  emitLabelByName("__bolt_instr_locations");
  for (MCSymbol *const &Label : Summary->Counters)
    emitFill(sizeof(uint64_t), Label);
```

- EN: Declares or implements routines including `emitLabelByName`, `emitFill`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `emitLabelByName`, `emitFill`.
- CN: 这里声明或实现函数，例如 `emitLabelByName`, `emitFill`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `emitLabelByName`, `emitFill`。

### Lines 168-185

```cpp
  emitPadding(BC.RegularPageSize);
  emitIntValue("__bolt_instr_max_size", opts::InstrumentationMaxSize);
  emitIntValue("__bolt_instr_sleep_time", opts::InstrumentationSleepTime);
  emitIntValue("__bolt_instr_no_counters_clear",
               !!opts::InstrumentationNoCountersClear, 1);
  emitIntValue("__bolt_instr_conservative", !!opts::ConservativeInstrumentation,
               1);
  emitIntValue("__bolt_instr_wait_forks", !!opts::InstrumentationWaitForks, 1);
  emitIntValue("__bolt_num_counters", Summary->Counters.size());
  emitValue(Summary->IndCallCounterFuncPtr, nullptr);
  emitValue(Summary->IndTailCallCounterFuncPtr, nullptr);
  emitIntValue("__bolt_instr_num_ind_calls",
               Summary->IndCallDescriptions.size());
  emitIntValue("__bolt_instr_num_ind_targets",
               Summary->IndCallTargetDescriptions.size());
  emitIntValue("__bolt_instr_num_funcs", Summary->FunctionDescriptions.size());
  emitString("__bolt_instr_filename", opts::InstrumentationFilename);
  emitString("__bolt_instr_binpath", opts::InstrumentationBinpath);
```

- EN: Declares or implements routines including `emitPadding`, `emitIntValue`, `emitValue`, `emitString`. Notable symbols here include `emitPadding`, `emitIntValue`, `emitValue`, `emitString`.
- CN: 这里声明或实现函数，例如 `emitPadding`, `emitIntValue`, `emitValue`, `emitString`。这里较值得关注的符号包括 `emitPadding`, `emitIntValue`, `emitValue`, `emitString`。

### Lines 186-196

```cpp
  emitIntValue("__bolt_instr_use_pid", !!opts::InstrumentationFileAppendPID, 1);

  if (BC.isMachO()) {
    MCSection *TablesSection = BC.Ctx->getMachOSection(
        "__BOLT", "__tables", MachO::S_REGULAR, SectionKind::getData());
    TablesSection->setAlignment(llvm::Align(BC.RegularPageSize));
    Streamer.switchSection(TablesSection);
    emitString("__bolt_instr_tables", buildTables(BC));
  }
}
```

- EN: Declares or implements routines including `emitIntValue`, `getData`, `setAlignment`, `emitString`. Notable symbols here include `emitIntValue`, `getData`, `setAlignment`, `emitString`.
- CN: 这里声明或实现函数，例如 `emitIntValue`, `getData`, `setAlignment`, `emitString`。这里较值得关注的符号包括 `emitIntValue`, `getData`, `setAlignment`, `emitString`。

### Lines 197-205

```cpp
void InstrumentationRuntimeLibrary::link(
    BinaryContext &BC, StringRef ToolPath, BOLTLinker &Linker,
    BOLTLinker::SectionsMapper MapSections) {
  std::string LibPath = getLibPath(ToolPath, opts::RuntimeInstrumentationLib);
  loadLibrary(LibPath, Linker, MapSections);

  if (BC.isMachO())
    return;
```

- EN: Declares or implements routines including `getLibPath`, `loadLibrary`. Notable symbols here include `getLibPath`, `loadLibrary`.
- CN: 这里声明或实现函数，例如 `getLibPath`, `loadLibrary`。这里较值得关注的符号包括 `getLibPath`, `loadLibrary`。

### Lines 206-215

```cpp
  std::optional<BOLTLinker::SymbolInfo> FiniSymInfo =
      Linker.lookupSymbolInfo("__bolt_instr_fini");
  if (!FiniSymInfo) {
    errs() << "BOLT-ERROR: instrumentation library does not define "
              "__bolt_instr_fini: "
           << LibPath << "\n";
    exit(1);
  }
  RuntimeFiniAddress = FiniSymInfo->Address;
```

- EN: Declares or implements routines including `errs`, `exit`. Notable symbols here include `errs`, `exit`.
- CN: 这里声明或实现函数，例如 `errs`, `exit`。这里较值得关注的符号包括 `errs`, `exit`。

### Lines 216-225

```cpp
  std::optional<BOLTLinker::SymbolInfo> StartSymInfo =
      Linker.lookupSymbolInfo("__bolt_instr_start");
  if (!StartSymInfo) {
    errs() << "BOLT-ERROR: instrumentation library does not define "
              "__bolt_instr_start: "
           << LibPath << "\n";
    exit(1);
  }
  RuntimeStartAddress = StartSymInfo->Address;
```

- EN: Declares or implements routines including `errs`, `exit`. Notable symbols here include `errs`, `exit`.
- CN: 这里声明或实现函数，例如 `errs`, `exit`。这里较值得关注的符号包括 `errs`, `exit`。

### Lines 226-235

```cpp
  outs() << "BOLT-INFO: output linked against instrumentation runtime "
            "library, lib entry point is 0x"
         << Twine::utohexstr(RuntimeStartAddress) << "\n";

  std::optional<BOLTLinker::SymbolInfo> ClearSymInfo =
      Linker.lookupSymbolInfo("__bolt_instr_clear_counters");
  const uint64_t ClearSymAddress = ClearSymInfo ? ClearSymInfo->Address : 0;
  outs() << "BOLT-INFO: clear procedure is 0x"
         << Twine::utohexstr(ClearSymAddress) << "\n";
```

- EN: Declares or implements routines including `outs`, `utohexstr`. Notable symbols here include `outs`, `utohexstr`.
- CN: 这里声明或实现函数，例如 `outs`, `utohexstr`。这里较值得关注的符号包括 `outs`, `utohexstr`。

### Lines 236-250

```cpp
  emitTablesAsELFNote(BC);
}

std::string InstrumentationRuntimeLibrary::buildTables(BinaryContext &BC) {
  std::string TablesStr;
  raw_string_ostream OS(TablesStr);

  // This is sync'ed with runtime/instr.cpp:readDescriptions()
  auto getOutputAddress = [](const BinaryFunction &Func,
                             uint64_t Offset) -> uint64_t {
    return Offset == 0
               ? Func.getOutputAddress()
               : Func.translateInputToOutputAddress(Func.getAddress() + Offset);
  };
```

- EN: Declares or implements routines including `emitTablesAsELFNote`, `buildTables`, `OS`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `emitTablesAsELFNote`, `buildTables`, `OS`.
- CN: 这里声明或实现函数，例如 `emitTablesAsELFNote`, `buildTables`, `OS`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `emitTablesAsELFNote`, `buildTables`, `OS`。

### Lines 251-258

```cpp
  // Indirect targets need to be sorted for fast lookup during runtime
  llvm::sort(Summary->IndCallTargetDescriptions,
             [&](const IndCallTargetDescription &A,
                 const IndCallTargetDescription &B) {
               return getOutputAddress(*A.Target, A.ToLoc.Offset) <
                      getOutputAddress(*B.Target, B.ToLoc.Offset);
             });
```

- EN: Declares or implements routines including `getOutputAddress`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `getOutputAddress`.
- CN: 这里声明或实现函数，例如 `getOutputAddress`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `getOutputAddress`。

### Lines 259-268

```cpp
  // Start of the vector with descriptions (one CounterDescription for each
  // counter), vector size is Counters.size() CounterDescription-sized elmts
  const size_t IDSize =
      Summary->IndCallDescriptions.size() * sizeof(IndCallDescription);
  OS.write(reinterpret_cast<const char *>(&IDSize), 4);
  for (const IndCallDescription &Desc : Summary->IndCallDescriptions) {
    OS.write(reinterpret_cast<const char *>(&Desc.FromLoc.FuncString), 4);
    OS.write(reinterpret_cast<const char *>(&Desc.FromLoc.Offset), 4);
  }
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 269-280

```cpp
  const size_t ITDSize = Summary->IndCallTargetDescriptions.size() *
                         sizeof(IndCallTargetDescription);
  OS.write(reinterpret_cast<const char *>(&ITDSize), 4);
  for (const IndCallTargetDescription &Desc :
       Summary->IndCallTargetDescriptions) {
    OS.write(reinterpret_cast<const char *>(&Desc.ToLoc.FuncString), 4);
    OS.write(reinterpret_cast<const char *>(&Desc.ToLoc.Offset), 4);
    uint64_t TargetFuncAddress =
        getOutputAddress(*Desc.Target, Desc.ToLoc.Offset);
    OS.write(reinterpret_cast<const char *>(&TargetFuncAddress), 8);
  }
```

- EN: Declares or implements routines including `getOutputAddress`. Notable symbols here include `getOutputAddress`.
- CN: 这里声明或实现函数，例如 `getOutputAddress`。这里较值得关注的符号包括 `getOutputAddress`。

### Lines 281-298

```cpp
  uint32_t FuncDescSize = Summary->getFDSize();
  OS.write(reinterpret_cast<const char *>(&FuncDescSize), 4);
  for (const FunctionDescription &Desc : Summary->FunctionDescriptions) {
    const size_t LeafNum = Desc.LeafNodes.size();
    OS.write(reinterpret_cast<const char *>(&LeafNum), 4);
    for (const InstrumentedNode &LeafNode : Desc.LeafNodes) {
      OS.write(reinterpret_cast<const char *>(&LeafNode.Node), 4);
      OS.write(reinterpret_cast<const char *>(&LeafNode.Counter), 4);
    }
    const size_t EdgesNum = Desc.Edges.size();
    OS.write(reinterpret_cast<const char *>(&EdgesNum), 4);
    for (const EdgeDescription &Edge : Desc.Edges) {
      OS.write(reinterpret_cast<const char *>(&Edge.FromLoc.FuncString), 4);
      OS.write(reinterpret_cast<const char *>(&Edge.FromLoc.Offset), 4);
      OS.write(reinterpret_cast<const char *>(&Edge.FromNode), 4);
      OS.write(reinterpret_cast<const char *>(&Edge.ToLoc.FuncString), 4);
      OS.write(reinterpret_cast<const char *>(&Edge.ToLoc.Offset), 4);
      OS.write(reinterpret_cast<const char *>(&Edge.ToNode), 4);
```

- EN: Declares or implements routines including `getFDSize`. Notable symbols here include `getFDSize`.
- CN: 这里声明或实现函数，例如 `getFDSize`。这里较值得关注的符号包括 `getFDSize`。

### Lines 299-316

```cpp
      OS.write(reinterpret_cast<const char *>(&Edge.Counter), 4);
    }
    const size_t CallsNum = Desc.Calls.size();
    OS.write(reinterpret_cast<const char *>(&CallsNum), 4);
    for (const CallDescription &Call : Desc.Calls) {
      OS.write(reinterpret_cast<const char *>(&Call.FromLoc.FuncString), 4);
      OS.write(reinterpret_cast<const char *>(&Call.FromLoc.Offset), 4);
      OS.write(reinterpret_cast<const char *>(&Call.FromNode), 4);
      OS.write(reinterpret_cast<const char *>(&Call.ToLoc.FuncString), 4);
      OS.write(reinterpret_cast<const char *>(&Call.ToLoc.Offset), 4);
      OS.write(reinterpret_cast<const char *>(&Call.Counter), 4);
      uint64_t TargetFuncAddress =
          getOutputAddress(*Call.Target, Call.ToLoc.Offset);
      OS.write(reinterpret_cast<const char *>(&TargetFuncAddress), 8);
    }
    const size_t EntryNum = Desc.EntryNodes.size();
    OS.write(reinterpret_cast<const char *>(&EntryNum), 4);
    for (const EntryNode &EntryNode : Desc.EntryNodes) {
```

- EN: Declares or implements routines including `getOutputAddress`. Notable symbols here include `getOutputAddress`.
- CN: 这里声明或实现函数，例如 `getOutputAddress`。这里较值得关注的符号包括 `getOutputAddress`。

### Lines 317-325

```cpp
      OS.write(reinterpret_cast<const char *>(&EntryNode.Node), 8);
      uint64_t TargetFuncAddress =
          getOutputAddress(*Desc.Function, EntryNode.Address);
      OS.write(reinterpret_cast<const char *>(&TargetFuncAddress), 8);
    }
  }
  // Our string table lives immediately after descriptions vector
  OS << Summary->StringTable;
```

- EN: Declares or implements routines including `getOutputAddress`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `getOutputAddress`.
- CN: 这里声明或实现函数，例如 `getOutputAddress`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `getOutputAddress`。

### Lines 326-337

```cpp
  return TablesStr;
}

void InstrumentationRuntimeLibrary::emitTablesAsELFNote(BinaryContext &BC) {
  std::string TablesStr = buildTables(BC);
  const std::string BoltInfo = BinarySection::encodeELFNote(
      "BOLT", TablesStr, BinarySection::NT_BOLT_INSTRUMENTATION_TABLES);
  BC.registerOrUpdateNoteSection(".bolt.instr.tables", copyByteArray(BoltInfo),
                                 BoltInfo.size(),
                                 /*Alignment=*/1,
                                 /*IsReadOnly=*/true, ELF::SHT_NOTE);
}
```

- EN: Declares or implements routines including `emitTablesAsELFNote`, `buildTables`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `emitTablesAsELFNote`, `buildTables`.
- CN: 这里声明或实现函数，例如 `emitTablesAsELFNote`, `buildTables`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `emitTablesAsELFNote`, `buildTables`。

## Key Concepts / 关键概念

- `desc`: function or method entry point / 函数或方法入口
- `init`: function or method entry point / 函数或方法入口
- `errs`: function or method entry point / 函数或方法入口
- `exit`: function or method entry point / 函数或方法入口
- `outs`: function or method entry point / 函数或方法入口
- `llvm`: namespace scope / 命名空间作用域
- `bolt`: namespace scope / 命名空间作用域

## Dependencies / 依赖关系

- Local headers / 本地头文件: `bolt/RuntimeLibs/InstrumentationRuntimeLibrary.h`, `bolt/Core/BinaryFunction.h`, `bolt/Core/JumpTable.h`, `bolt/Core/Linker.h`, `bolt/Utils/CommandLineOpts.h`
- LLVM headers / LLVM 头文件: `llvm/MC/MCStreamer.h`, `llvm/Support/Alignment.h`, `llvm/Support/CommandLine.h`
- Directory context / 目录上下文: `bolt/lib/RuntimeLibs` neighbors usually cooperate with this file to provide the surrounding subsystem / `bolt/lib/RuntimeLibs` 下的相邻文件通常与本文件协作组成对应子系统
