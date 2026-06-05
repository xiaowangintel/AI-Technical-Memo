# PerfSupportPlugin.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `llvm/lib/ExecutionEngine/Orc/Debugging/PerfSupportPlugin.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 作用**:
  - **EN**: Implements ORC JIT infrastructure such as execution sessions, JITDylibs, materialization, symbol lookup, and execution helpers.
  - **CN**: 实现 ORC JIT 基础设施，例如 ExecutionSession、JITDylib、物化流程、符号查找以及执行辅助组件。

## Line-by-Line Analysis / 逐行分析

### Lines 1-11
```cpp
//===----- PerfSupportPlugin.cpp --- Utils for perf support -----*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// Handles support for registering code with perf
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM file banner, license notice, and high-level file description.
- **CN**: 包含 LLVM 标准文件头、许可证声明以及文件级说明。

### Lines 12-20
```cpp

#include "llvm/ExecutionEngine/Orc/Debugging/PerfSupportPlugin.h"

#include "llvm/ExecutionEngine/Orc/Debugging/DebugInfoSupport.h"
#include "llvm/ExecutionEngine/Orc/LookupAndRecordAddrs.h"
#include "llvm/ExecutionEngine/Orc/Shared/WrapperFunctionUtils.h"

#define DEBUG_TYPE "orc"

```
- **EN**: Pulls in the headers needed for this implementation, including `llvm/ExecutionEngine/Orc/Debugging/PerfSupportPlugin.h`, `llvm/ExecutionEngine/Orc/Debugging/DebugInfoSupport.h`, `llvm/ExecutionEngine/Orc/LookupAndRecordAddrs.h`, `llvm/ExecutionEngine/Orc/Shared/WrapperFunctionUtils.h`.
- **CN**: 引入该实现所需的头文件，其中包括 `llvm/ExecutionEngine/Orc/Debugging/PerfSupportPlugin.h`, `llvm/ExecutionEngine/Orc/Debugging/DebugInfoSupport.h`, `llvm/ExecutionEngine/Orc/LookupAndRecordAddrs.h`, `llvm/ExecutionEngine/Orc/Shared/WrapperFunctionUtils.h`。

### Lines 21-38
```cpp
using namespace llvm;
using namespace llvm::orc;
using namespace llvm::jitlink;

namespace {

// Creates an EH frame header prepared for a 32-bit relative relocation
// to the start of the .eh_frame section. Absolute injects a 64-bit absolute
// address space offset 4 bytes from the start instead of 4 bytes
Expected<std::string> createX64EHFrameHeader(Section &EHFrame,
                                             llvm::endianness endianness,
                                             bool absolute) {
  uint8_t Version = 1;
  uint8_t EhFramePtrEnc = 0;
  if (absolute) {
    EhFramePtrEnc |= dwarf::DW_EH_PE_sdata8 | dwarf::DW_EH_PE_absptr;
  } else {
    EhFramePtrEnc |= dwarf::DW_EH_PE_sdata4 | dwarf::DW_EH_PE_datarel;
```
- **EN**: Introduces declarations for `llvm`, `llvm::orc`, `llvm::jitlink`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `llvm`, `llvm::orc`, `llvm::jitlink` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 39-50
```cpp
  }
  uint8_t FDECountEnc = dwarf::DW_EH_PE_omit;
  uint8_t TableEnc = dwarf::DW_EH_PE_omit;
  // X86_64_64 relocation to the start of the .eh_frame section
  uint32_t EHFrameRelocation = 0;
  // uint32_t FDECount = 0;
  // Skip the FDE binary search table
  // We'd have to reprocess the CIEs to get this information,
  // which seems like more trouble than it's worth
  // TODO consider implementing this.
  // binary search table goes here

```
- **EN**: Contains supporting implementation details for the surrounding subsystem logic.
- **CN**: 包含周边子系统逻辑所需的辅助实现细节。

### Lines 51-68
```cpp
  size_t HeaderSize =
      (sizeof(Version) + sizeof(EhFramePtrEnc) + sizeof(FDECountEnc) +
       sizeof(TableEnc) +
       (absolute ? sizeof(uint64_t) : sizeof(EHFrameRelocation)));
  std::string HeaderContent(HeaderSize, '\0');
  BinaryStreamWriter Writer(
      MutableArrayRef<uint8_t>(
          reinterpret_cast<uint8_t *>(HeaderContent.data()), HeaderSize),
      endianness);
  if (auto Err = Writer.writeInteger(Version))
    return std::move(Err);
  if (auto Err = Writer.writeInteger(EhFramePtrEnc))
    return std::move(Err);
  if (auto Err = Writer.writeInteger(FDECountEnc))
    return std::move(Err);
  if (auto Err = Writer.writeInteger(TableEnc))
    return std::move(Err);
  if (absolute) {
```
- **EN**: Implements logic around `HeaderContent`, `Writer`, `MutableArrayRef<uint8_t>`, `data`, and 1 more symbols; this block handles relocation, fixup, or symbol-resolution work; drives emission, layout, or binary encoding behavior; returns subsystem-specific computed results.
- **CN**: 围绕 `HeaderContent`, `Writer`, `MutableArrayRef<uint8_t>`, `data`, and 1 more symbols 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，驱动输出、布局或二进制编码行为，返回子系统相关的计算结果。

### Lines 69-78
```cpp
    uint64_t EHFrameAddr = SectionRange(EHFrame).getStart().getValue();
    if (auto Err = Writer.writeInteger(EHFrameAddr))
      return std::move(Err);
  } else {
    if (auto Err = Writer.writeInteger(EHFrameRelocation))
      return std::move(Err);
  }
  return HeaderContent;
}

```
- **EN**: Implements logic around `SectionRange`, `move`; this block handles relocation, fixup, or symbol-resolution work; drives emission, layout, or binary encoding behavior; returns subsystem-specific computed results.
- **CN**: 围绕 `SectionRange`, `move` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，驱动输出、布局或二进制编码行为，返回子系统相关的计算结果。

### Lines 79-96
```cpp
constexpr StringRef RegisterPerfStartSymbolName =
    "llvm_orc_registerJITLoaderPerfStart";
constexpr StringRef RegisterPerfEndSymbolName =
    "llvm_orc_registerJITLoaderPerfEnd";
constexpr StringRef RegisterPerfImplSymbolName =
    "llvm_orc_registerJITLoaderPerfImpl";

static PerfJITCodeLoadRecord
getCodeLoadRecord(const Symbol &Sym, std::atomic<uint64_t> &CodeIndex) {
  PerfJITCodeLoadRecord Record;
  auto Name = *Sym.getName();
  auto Addr = Sym.getAddress();
  auto Size = Sym.getSize();
  Record.Prefix.Id = PerfJITRecordType::JIT_CODE_LOAD;
  // Runtime sets PID
  Record.Pid = 0;
  // Runtime sets TID
  Record.Tid = 0;
```
- **EN**: Implements logic around `getCodeLoadRecord`, `getName`, `getAddress`, `getSize`.
- **CN**: 围绕 `getCodeLoadRecord`, `getName`, `getAddress`, `getSize` 实现具体逻辑。

### Lines 97-113
```cpp
  Record.Vma = Addr.getValue();
  Record.CodeAddr = Addr.getValue();
  Record.CodeSize = Size;
  Record.CodeIndex = CodeIndex++;
  Record.Name = Name.str();
  // Initialize last, once all the other fields are filled
  Record.Prefix.TotalSize =
      (2 * sizeof(uint32_t)   // id, total_size
       + sizeof(uint64_t)     // timestamp
       + 2 * sizeof(uint32_t) // pid, tid
       + 4 * sizeof(uint64_t) // vma, code_addr, code_size, code_index
       + Name.size() + 1      // symbol name
       + Record.CodeSize      // code
      );
  return Record;
}

```
- **EN**: Implements logic around `getValue`, `str`, `size`; this block returns subsystem-specific computed results.
- **CN**: 围绕 `getValue`, `str`, `size` 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 114-131
```cpp
static std::optional<PerfJITDebugInfoRecord>
getDebugInfoRecord(const Symbol &Sym, DWARFContext &DC) {
  auto &Section = Sym.getSection();
  auto Addr = Sym.getAddress();
  auto Size = Sym.getSize();
  auto SAddr = object::SectionedAddress{Addr.getValue(), Section.getOrdinal()};
  LLVM_DEBUG(dbgs() << "Getting debug info for symbol " << Sym.getName()
                    << " at address " << Addr.getValue() << " with size "
                    << Size << "\n"
                    << "Section ordinal: " << Section.getOrdinal() << "\n");
  auto LInfo = DC.getLineInfoForAddressRange(
      SAddr, Size, DILineInfoSpecifier::FileLineInfoKind::AbsoluteFilePath);
  if (LInfo.empty()) {
    // No line info available
    LLVM_DEBUG(dbgs() << "No line info available\n");
    return std::nullopt;
  }
  PerfJITDebugInfoRecord Record;
```
- **EN**: Implements logic around `getDebugInfoRecord`, `getSection`, `getAddress`, `getSize`, and 3 more symbols; this block returns subsystem-specific computed results.
- **CN**: 围绕 `getDebugInfoRecord`, `getSection`, `getAddress`, `getSize`, and 3 more symbols 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 132-149
```cpp
  Record.Prefix.Id = PerfJITRecordType::JIT_CODE_DEBUG_INFO;
  Record.CodeAddr = Addr.getValue();
  for (const auto &Entry : LInfo) {
    auto Addr = Entry.first;
    // The function re-created by perf is preceded by a elf
    // header. Need to adjust for that, otherwise the results are
    // wrong.
    Addr += 0x40;
    Record.Entries.push_back({Addr, Entry.second.Line,
                              Entry.second.Discriminator,
                              Entry.second.FileName});
  }
  size_t EntriesBytes = (2   // record header
                         + 2 // record fields
                         ) *
                        sizeof(uint64_t);
  for (const auto &Entry : Record.Entries) {
    EntriesBytes +=
```
- **EN**: Implements logic around `getValue`, `push_back`.
- **CN**: 围绕 `getValue`, `push_back` 实现具体逻辑。

### Lines 150-159
```cpp
        sizeof(uint64_t) + 2 * sizeof(uint32_t); // Addr, Line/Discrim
    EntriesBytes += Entry.Name.size() + 1;       // Name
  }
  Record.Prefix.TotalSize = EntriesBytes;
  LLVM_DEBUG(dbgs() << "Created debug info record\n"
                    << "Total size: " << Record.Prefix.TotalSize << "\n"
                    << "Nr entries: " << Record.Entries.size() << "\n");
  return Record;
}

```
- **EN**: Implements logic around `size`; this block returns subsystem-specific computed results.
- **CN**: 围绕 `size` 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 160-177
```cpp
static Expected<PerfJITCodeUnwindingInfoRecord>
getUnwindingRecord(LinkGraph &G) {
  PerfJITCodeUnwindingInfoRecord Record;
  Record.Prefix.Id = PerfJITRecordType::JIT_CODE_UNWINDING_INFO;
  Record.Prefix.TotalSize = 0;
  auto Eh_frame = G.findSectionByName(".eh_frame");
  if (!Eh_frame) {
    LLVM_DEBUG(dbgs() << "No .eh_frame section found\n");
    return Record;
  }
  if (!G.getTargetTriple().isOSBinFormatELF()) {
    LLVM_DEBUG(dbgs() << "Not an ELF file, will not emit unwinding info\n");
    return Record;
  }
  auto SR = SectionRange(*Eh_frame);
  auto EHFrameSize = SR.getSize();
  auto Eh_frame_hdr = G.findSectionByName(".eh_frame_hdr");
  if (!Eh_frame_hdr) {
```
- **EN**: Implements logic around `getUnwindingRecord`, `findSectionByName`, `SectionRange`, `getSize`; this block manipulates JITLink graph structures or link-time passes; returns subsystem-specific computed results.
- **CN**: 围绕 `getUnwindingRecord`, `findSectionByName`, `SectionRange`, `getSize` 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass，返回子系统相关的计算结果。

### Lines 178-195
```cpp
    if (G.getTargetTriple().getArch() == Triple::x86_64) {
      auto Hdr = createX64EHFrameHeader(*Eh_frame, G.getEndianness(), true);
      if (!Hdr)
        return Hdr.takeError();
      Record.EHFrameHdr = std::move(*Hdr);
    } else {
      LLVM_DEBUG(dbgs() << "No .eh_frame_hdr section found\n");
      return Record;
    }
    Record.EHFrameHdrAddr = 0;
    Record.EHFrameHdrSize = Record.EHFrameHdr.size();
    Record.UnwindDataSize = EHFrameSize + Record.EHFrameHdrSize;
    Record.MappedSize = 0; // Because the EHFrame header was not mapped
  } else {
    auto SR = SectionRange(*Eh_frame_hdr);
    Record.EHFrameHdrAddr = SR.getStart().getValue();
    Record.EHFrameHdrSize = SR.getSize();
    Record.UnwindDataSize = EHFrameSize + Record.EHFrameHdrSize;
```
- **EN**: Implements logic around `createX64EHFrameHeader`, `takeError`, `move`, `size`, and 3 more symbols; this block returns subsystem-specific computed results.
- **CN**: 围绕 `createX64EHFrameHeader`, `takeError`, `move`, `size`, and 3 more symbols 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 196-213
```cpp
    Record.MappedSize = Record.UnwindDataSize;
  }
  Record.EHFrameAddr = SR.getStart().getValue();
  Record.Prefix.TotalSize =
      (2 * sizeof(uint32_t) // id, total_size
       + sizeof(uint64_t)   // timestamp
       +
       3 * sizeof(uint64_t) // unwind_data_size, eh_frame_hdr_size, mapped_size
       + Record.UnwindDataSize // eh_frame_hdr, eh_frame
      );
  LLVM_DEBUG(dbgs() << "Created unwind record\n"
                    << "Total size: " << Record.Prefix.TotalSize << "\n"
                    << "Unwind size: " << Record.UnwindDataSize << "\n"
                    << "EHFrame size: " << EHFrameSize << "\n"
                    << "EHFrameHdr size: " << Record.EHFrameHdrSize << "\n");
  return Record;
}

```
- **EN**: Implements logic around `getStart`; this block returns subsystem-specific computed results.
- **CN**: 围绕 `getStart` 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 214-231
```cpp
static PerfJITRecordBatch getRecords(ExecutionSession &ES, LinkGraph &G,
                                     std::atomic<uint64_t> &CodeIndex,
                                     bool EmitDebugInfo, bool EmitUnwindInfo) {
  std::unique_ptr<DWARFContext> DC;
  StringMap<std::unique_ptr<MemoryBuffer>> DCBacking;
  if (EmitDebugInfo) {
    auto EDC = createDWARFContext(G);
    if (!EDC) {
      ES.reportError(EDC.takeError());
      EmitDebugInfo = false;
    } else {
      DC = std::move(EDC->first);
      DCBacking = std::move(EDC->second);
    }
  }
  PerfJITRecordBatch Batch;
  for (auto Sym : G.defined_symbols()) {
    if (!Sym->hasName() || !Sym->isCallable())
```
- **EN**: Implements logic around `getRecords`, `createDWARFContext`, `reportError`, `move`; this block coordinates ORC symbol lookup or materialization state; manipulates JITLink graph structures or link-time passes.
- **CN**: 围绕 `getRecords`, `createDWARFContext`, `reportError`, `move` 实现具体逻辑；这一段协调 ORC 符号查找或物化状态，操作 JITLink 图结构或链接阶段 pass。

### Lines 232-249
```cpp
      continue;
    if (EmitDebugInfo) {
      auto DebugInfo = getDebugInfoRecord(*Sym, *DC);
      if (DebugInfo)
        Batch.DebugInfoRecords.push_back(std::move(*DebugInfo));
    }
    Batch.CodeLoadRecords.push_back(getCodeLoadRecord(*Sym, CodeIndex));
  }
  if (EmitUnwindInfo) {
    auto UWR = getUnwindingRecord(G);
    if (!UWR) {
      ES.reportError(UWR.takeError());
    } else {
      Batch.UnwindingRecord = std::move(*UWR);
    }
  } else {
    Batch.UnwindingRecord.Prefix.TotalSize = 0;
  }
```
- **EN**: Implements logic around `getDebugInfoRecord`, `push_back`, `getUnwindingRecord`, `reportError`, and 1 more symbols.
- **CN**: 围绕 `getDebugInfoRecord`, `push_back`, `getUnwindingRecord`, `reportError`, and 1 more symbols 实现具体逻辑。

### Lines 250-267
```cpp
  return Batch;
}
} // namespace

PerfSupportPlugin::PerfSupportPlugin(ExecutorProcessControl &EPC,
                                     ExecutorAddr RegisterPerfStartAddr,
                                     ExecutorAddr RegisterPerfEndAddr,
                                     ExecutorAddr RegisterPerfImplAddr,
                                     bool EmitDebugInfo, bool EmitUnwindInfo)
    : EPC(EPC), RegisterPerfStartAddr(RegisterPerfStartAddr),
      RegisterPerfEndAddr(RegisterPerfEndAddr),
      RegisterPerfImplAddr(RegisterPerfImplAddr), CodeIndex(0),
      EmitDebugInfo(EmitDebugInfo), EmitUnwindInfo(EmitUnwindInfo) {
  cantFail(EPC.callSPSWrapper<void()>(RegisterPerfStartAddr));
}
PerfSupportPlugin::~PerfSupportPlugin() {
  cantFail(EPC.callSPSWrapper<void()>(RegisterPerfEndAddr));
}
```
- **EN**: Implements logic around `PerfSupportPlugin`, `EPC`, `RegisterPerfEndAddr`, `RegisterPerfImplAddr`, and 3 more symbols; this block returns subsystem-specific computed results.
- **CN**: 围绕 `PerfSupportPlugin`, `EPC`, `RegisterPerfEndAddr`, `RegisterPerfImplAddr`, and 3 more symbols 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 268-283
```cpp

void PerfSupportPlugin::modifyPassConfig(MaterializationResponsibility &MR,
                                         LinkGraph &G,
                                         PassConfiguration &Config) {
  Config.PostFixupPasses.push_back([this](LinkGraph &G) {
    auto Batch = getRecords(EPC.getExecutionSession(), G, CodeIndex,
                            EmitDebugInfo, EmitUnwindInfo);
    G.allocActions().push_back(
        {cantFail(shared::WrapperFunctionCall::Create<
                  shared::SPSArgList<shared::SPSPerfJITRecordBatch>>(
             RegisterPerfImplAddr, Batch)),
         {}});
    return Error::success();
  });
}

```
- **EN**: Implements logic around `modifyPassConfig`, `push_back`, `getRecords`, `allocActions`, and 3 more symbols; this block coordinates ORC symbol lookup or materialization state; manipulates JITLink graph structures or link-time passes; handles relocation, fixup, or symbol-resolution work; returns subsystem-specific computed results.
- **CN**: 围绕 `modifyPassConfig`, `push_back`, `getRecords`, `allocActions`, and 3 more symbols 实现具体逻辑；这一段协调 ORC 符号查找或物化状态，操作 JITLink 图结构或链接阶段 pass，处理重定位、fixup 或符号解析工作，返回子系统相关的计算结果。

### Lines 284-301
```cpp
Expected<std::unique_ptr<PerfSupportPlugin>>
PerfSupportPlugin::Create(ExecutorProcessControl &EPC, JITDylib &JD,
                          bool EmitDebugInfo, bool EmitUnwindInfo) {
  if (!EPC.getTargetTriple().isOSBinFormatELF()) {
    return make_error<StringError>(
        "Perf support only available for ELF LinkGraphs!",
        inconvertibleErrorCode());
  }
  auto &ES = EPC.getExecutionSession();
  ExecutorAddr StartAddr, EndAddr, ImplAddr;
  if (auto Err = lookupAndRecordAddrs(
          ES, LookupKind::Static, makeJITDylibSearchOrder({&JD}),
          {{ES.intern(RegisterPerfStartSymbolName), &StartAddr},
           {ES.intern(RegisterPerfEndSymbolName), &EndAddr},
           {ES.intern(RegisterPerfImplSymbolName), &ImplAddr}}))
    return std::move(Err);
  return std::make_unique<PerfSupportPlugin>(EPC, StartAddr, EndAddr, ImplAddr,
                                             EmitDebugInfo, EmitUnwindInfo);
```
- **EN**: Implements logic around `Create`, `make_error<StringError>`, `inconvertibleErrorCode`, `getExecutionSession`, and 4 more symbols; this block coordinates ORC symbol lookup or materialization state; manipulates JITLink graph structures or link-time passes; returns subsystem-specific computed results.
- **CN**: 围绕 `Create`, `make_error<StringError>`, `inconvertibleErrorCode`, `getExecutionSession`, and 4 more symbols 实现具体逻辑；这一段协调 ORC 符号查找或物化状态，操作 JITLink 图结构或链接阶段 pass，返回子系统相关的计算结果。

### Lines 302-302
```cpp
}
```
- **EN**: Contains supporting implementation details for the surrounding subsystem logic.
- **CN**: 包含周边子系统逻辑所需的辅助实现细节。

## Key Concepts / 关键概念

- **ORC JIT / ORC JIT**:
  - **EN**: Coordinates JITDylibs, symbol materialization, execution sessions, and asynchronous compilation flows
  - **CN**: 协调 JITDylib、符号物化、ExecutionSession 与异步编译流程
- **Symbol materialization / 符号物化**:
  - **EN**: Defers code or data generation until symbols are requested
  - **CN**: 将代码或数据的生成延迟到符号真正被请求时
- **Relocations and fixups / 重定位与 fixup**:
  - **EN**: Bridges symbolic references to concrete addresses or relocation records
  - **CN**: 把符号引用连接到具体地址或重定位记录
- **Debug/unwind metadata / 调试与展开元数据**:
  - **EN**: Emits or manages metadata needed for debugging and stack unwinding
  - **CN**: 输出或管理调试与栈展开所需的元数据

## Dependencies / 依赖关系

- **Direct includes / 直接包含**: `llvm/ExecutionEngine/Orc/Debugging/PerfSupportPlugin.h`, `llvm/ExecutionEngine/Orc/Debugging/DebugInfoSupport.h`, `llvm/ExecutionEngine/Orc/LookupAndRecordAddrs.h`, `llvm/ExecutionEngine/Orc/Shared/WrapperFunctionUtils.h`
- **LLVM subsystems / LLVM 子系统**: ExecutionEngine
- **External tooling / 外部工具**: Profiler or debugger integration points are present in this file. / 本文件包含与性能分析器或调试器的集成点。
