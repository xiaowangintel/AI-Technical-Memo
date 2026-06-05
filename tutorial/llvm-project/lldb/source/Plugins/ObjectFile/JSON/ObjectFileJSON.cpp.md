# ObjectFileJSON.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `lldb/source/Plugins/ObjectFile/JSON/ObjectFileJSON.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: Implements the LLDB support logic associated with `ObjectFileJSON`.
  - **CN**: 实现与 `ObjectFileJSON` 相关的 LLDB 支持逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```cpp
//===-- ObjectFileJSON.cpp ------------------------------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM/LLDB file banner, license notice, and high-level file description.
- **CN**: 包含 LLVM/LLDB 标准文件头、许可证声明以及文件的高层说明。

### Lines 8-20
```cpp

#include "Plugins/ObjectFile/JSON/ObjectFileJSON.h"
#include "lldb/Core/Module.h"
#include "lldb/Core/ModuleSpec.h"
#include "lldb/Core/PluginManager.h"
#include "lldb/Core/Section.h"
#include "lldb/Symbol/Symbol.h"
#include "lldb/Target/Target.h"
#include "lldb/Utility/LLDBLog.h"
#include "lldb/Utility/Log.h"
#include "llvm/ADT/DenseSet.h"
#include <optional>

```
- **EN**: Pulls in the headers needed by this translation unit, including `Plugins/ObjectFile/JSON/ObjectFileJSON.h`, `lldb/Core/Module.h`, `lldb/Core/ModuleSpec.h`, `lldb/Core/PluginManager.h`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `Plugins/ObjectFile/JSON/ObjectFileJSON.h`, `lldb/Core/Module.h`, `lldb/Core/ModuleSpec.h`, `lldb/Core/PluginManager.h`。

### Lines 21-28
```cpp
using namespace llvm;
using namespace lldb;
using namespace lldb_private;

LLDB_PLUGIN_DEFINE(ObjectFileJSON)

char ObjectFileJSON::ID;

```
- **EN**: Implements logic around `LLDB_PLUGIN_DEFINE`.
- **CN**: 围绕 `LLDB_PLUGIN_DEFINE` 实现具体逻辑。

### Lines 29-38
```cpp
void ObjectFileJSON::Initialize() {
  PluginManager::RegisterPlugin(GetPluginNameStatic(),
                                GetPluginDescriptionStatic(), CreateInstance,
                                CreateMemoryInstance, GetModuleSpecifications);
}

void ObjectFileJSON::Terminate() {
  PluginManager::UnregisterPlugin(CreateInstance);
}

```
- **EN**: Implements logic around `Initialize`, `RegisterPlugin`, `GetPluginDescriptionStatic`, `Terminate`, and 1 more symbols.
- **CN**: 围绕 `Initialize`, `RegisterPlugin`, `GetPluginDescriptionStatic`, `Terminate`, and 1 more symbols 实现具体逻辑。

### Lines 39-52
```cpp
ObjectFile *ObjectFileJSON::CreateInstance(const ModuleSP &module_sp,
                                           DataExtractorSP extractor_sp,
                                           offset_t data_offset,
                                           const FileSpec *file,
                                           offset_t file_offset,
                                           offset_t length) {
  if (!extractor_sp || !extractor_sp->HasData()) {
    DataBufferSP data_sp = MapFileData(*file, length, file_offset);
    if (!data_sp)
      return nullptr;
    extractor_sp = std::make_shared<DataExtractor>(data_sp);
    data_offset = 0;
  }
  if (!extractor_sp->HasData())
```
- **EN**: Implements logic around `CreateInstance`, `HasData`, `MapFileData`, `make_shared`; this block abstracts host OS resources such as files, terminals, sockets, or platforms.
- **CN**: 围绕 `CreateInstance`, `HasData`, `MapFileData`, `make_shared` 实现具体逻辑；该代码块抽象文件、终端、套接字或平台等主机操作系统资源。

### Lines 53-66
```cpp
    return nullptr;

  if (!MagicBytesMatch(extractor_sp->GetSubsetExtractorSP(data_offset)))
    return nullptr;

  // Update the data to contain the entire file if it doesn't already.
  if (extractor_sp->GetByteSize() < length) {
    DataBufferSP data_sp = MapFileData(*file, length, file_offset);
    if (!data_sp)
      return nullptr;
    extractor_sp->SetData(data_sp);
    data_offset = 0;
  }

```
- **EN**: Implements logic around `MagicBytesMatch`, `GetByteSize`, `MapFileData`, `SetData`; this block abstracts host OS resources such as files, terminals, sockets, or platforms.
- **CN**: 围绕 `MagicBytesMatch`, `GetByteSize`, `MapFileData`, `SetData` 实现具体逻辑；该代码块抽象文件、终端、套接字或平台等主机操作系统资源。

### Lines 67-77
```cpp
  Log *log = GetLog(LLDBLog::Symbols);

  auto text = llvm::StringRef((const char *)extractor_sp->GetData().data());

  Expected<json::Value> json = json::parse(text);
  if (!json) {
    LLDB_LOG_ERROR(log, json.takeError(),
                   "failed to parse JSON object file: {0}");
    return nullptr;
  }

```
- **EN**: Implements logic around `GetLog`, `StringRef`, `parse`, `LLDB_LOG_ERROR`; this block propagates recoverable errors, status objects, or diagnostics; abstracts host OS resources such as files, terminals, sockets, or platforms.
- **CN**: 围绕 `GetLog`, `StringRef`, `parse`, `LLDB_LOG_ERROR` 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息，并抽象文件、终端、套接字或平台等主机操作系统资源。

### Lines 78-85
```cpp
  json::Path::Root root;
  Header header;
  if (!fromJSON(*json, header, root)) {
    LLDB_LOG_ERROR(log, root.getError(),
                   "failed to parse JSON object file header: {0}");
    return nullptr;
  }

```
- **EN**: Implements logic around `fromJSON`, `LLDB_LOG_ERROR`; this block propagates recoverable errors, status objects, or diagnostics; abstracts host OS resources such as files, terminals, sockets, or platforms.
- **CN**: 围绕 `fromJSON`, `LLDB_LOG_ERROR` 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息，并抽象文件、终端、套接字或平台等主机操作系统资源。

### Lines 86-97
```cpp
  ArchSpec arch(header.triple);
  UUID uuid;
  uuid.SetFromStringRef(header.uuid);
  Type type = header.type.value_or(eTypeDebugInfo);

  Body body;
  if (!fromJSON(*json, body, root)) {
    LLDB_LOG_ERROR(log, root.getError(),
                   "failed to parse JSON object file body: {0}");
    return nullptr;
  }

```
- **EN**: Implements logic around `arch`, `SetFromStringRef`, `value_or`, `fromJSON`, and 1 more symbols; this block propagates recoverable errors, status objects, or diagnostics; abstracts host OS resources such as files, terminals, sockets, or platforms; models machine state, stack inspection, or disassembly-oriented metadata.
- **CN**: 围绕 `arch`, `SetFromStringRef`, `value_or`, `fromJSON`, and 1 more symbols 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息，并抽象文件、终端、套接字或平台等主机操作系统资源，并建模机器状态、栈检查或面向反汇编的元数据。

### Lines 98-110
```cpp
  return new ObjectFileJSON(module_sp, extractor_sp, data_offset, file,
                            file_offset, length, std::move(arch),
                            std::move(uuid), type, std::move(body.symbols),
                            std::move(body.sections));
}

ObjectFile *ObjectFileJSON::CreateMemoryInstance(const ModuleSP &module_sp,
                                                 WritableDataBufferSP data_sp,
                                                 const ProcessSP &process_sp,
                                                 addr_t header_addr) {
  return nullptr;
}

```
- **EN**: Implements logic around `ObjectFileJSON`, `move`, `CreateMemoryInstance`; this block abstracts host OS resources such as files, terminals, sockets, or platforms; models machine state, stack inspection, or disassembly-oriented metadata.
- **CN**: 围绕 `ObjectFileJSON`, `move`, `CreateMemoryInstance` 实现具体逻辑；该代码块抽象文件、终端、套接字或平台等主机操作系统资源，并建模机器状态、栈检查或面向反汇编的元数据。

### Lines 111-117
```cpp
ModuleSpecList
ObjectFileJSON::GetModuleSpecifications(const FileSpec &file,
                                        DataExtractorSP &extractor_sp,
                                        offset_t file_offset, offset_t length) {
  if (!extractor_sp || !MagicBytesMatch(extractor_sp))
    return {};

```
- **EN**: Implements logic around `GetModuleSpecifications`, `MagicBytesMatch`; this block abstracts host OS resources such as files, terminals, sockets, or platforms.
- **CN**: 围绕 `GetModuleSpecifications`, `MagicBytesMatch` 实现具体逻辑；该代码块抽象文件、终端、套接字或平台等主机操作系统资源。

### Lines 118-125
```cpp
  // Update the data to contain the entire file if it doesn't already.
  if (extractor_sp->GetByteSize() < length) {
    if (DataBufferSP file_data_sp = MapFileData(file, length, file_offset))
      extractor_sp->SetData(file_data_sp);
    if (!extractor_sp->HasData())
      return {};
  }

```
- **EN**: Implements logic around `GetByteSize`, `MapFileData`, `SetData`, `HasData`; this block abstracts host OS resources such as files, terminals, sockets, or platforms.
- **CN**: 围绕 `GetByteSize`, `MapFileData`, `SetData`, `HasData` 实现具体逻辑；该代码块抽象文件、终端、套接字或平台等主机操作系统资源。

### Lines 126-136
```cpp
  Log *log = GetLog(LLDBLog::Symbols);

  auto text = llvm::StringRef((const char *)extractor_sp->GetData().data());

  Expected<json::Value> json = json::parse(text);
  if (!json) {
    LLDB_LOG_ERROR(log, json.takeError(),
                   "failed to parse JSON object file: {0}");
    return {};
  }

```
- **EN**: Implements logic around `GetLog`, `StringRef`, `parse`, `LLDB_LOG_ERROR`; this block propagates recoverable errors, status objects, or diagnostics; abstracts host OS resources such as files, terminals, sockets, or platforms.
- **CN**: 围绕 `GetLog`, `StringRef`, `parse`, `LLDB_LOG_ERROR` 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息，并抽象文件、终端、套接字或平台等主机操作系统资源。

### Lines 137-144
```cpp
  json::Path::Root root;
  Header header;
  if (!fromJSON(*json, header, root)) {
    LLDB_LOG_ERROR(log, root.getError(),
                   "failed to parse JSON object file header: {0}");
    return {};
  }

```
- **EN**: Implements logic around `fromJSON`, `LLDB_LOG_ERROR`; this block propagates recoverable errors, status objects, or diagnostics; abstracts host OS resources such as files, terminals, sockets, or platforms.
- **CN**: 围绕 `fromJSON`, `LLDB_LOG_ERROR` 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息，并抽象文件、终端、套接字或平台等主机操作系统资源。

### Lines 145-155
```cpp
  ArchSpec arch(header.triple);
  UUID uuid;
  uuid.SetFromStringRef(header.uuid);

  ModuleSpec spec(file, std::move(arch));
  spec.GetUUID() = std::move(uuid);
  ModuleSpecList specs;
  specs.Append(spec);
  return specs;
}

```
- **EN**: Implements logic around `arch`, `SetFromStringRef`, `spec`, `GetUUID`, and 1 more symbols; this block abstracts host OS resources such as files, terminals, sockets, or platforms; models machine state, stack inspection, or disassembly-oriented metadata.
- **CN**: 围绕 `arch`, `SetFromStringRef`, `spec`, `GetUUID`, and 1 more symbols 实现具体逻辑；该代码块抽象文件、终端、套接字或平台等主机操作系统资源，并建模机器状态、栈检查或面向反汇编的元数据。

### Lines 156-166
```cpp
ObjectFileJSON::ObjectFileJSON(const ModuleSP &module_sp,
                               DataExtractorSP extractor_sp,
                               offset_t data_offset, const FileSpec *file,
                               offset_t offset, offset_t length, ArchSpec arch,
                               UUID uuid, Type type,
                               std::vector<JSONSymbol> symbols,
                               std::vector<JSONSection> sections)
    : ObjectFile(module_sp, file, offset, length, extractor_sp, data_offset),
      m_arch(std::move(arch)), m_uuid(std::move(uuid)), m_type(type),
      m_symbols(std::move(symbols)), m_sections(std::move(sections)) {}

```
- **EN**: Implements logic around `ObjectFileJSON`, `ObjectFile`, `m_arch`, `m_symbols`; this block abstracts host OS resources such as files, terminals, sockets, or platforms; models machine state, stack inspection, or disassembly-oriented metadata.
- **CN**: 围绕 `ObjectFileJSON`, `ObjectFile`, `m_arch`, `m_symbols` 实现具体逻辑；该代码块抽象文件、终端、套接字或平台等主机操作系统资源，并建模机器状态、栈检查或面向反汇编的元数据。

### Lines 167-180
```cpp
bool ObjectFileJSON::ParseHeader() {
  // We already parsed the header during initialization.
  return true;
}

void ObjectFileJSON::ParseSymtab(Symtab &symtab) {
  Log *log = GetLog(LLDBLog::Symbols);
  SectionList *section_list = GetModule()->GetSectionList();
  for (JSONSymbol json_symbol : m_symbols) {
    llvm::Expected<Symbol> symbol = Symbol::FromJSON(json_symbol, section_list);
    if (!symbol) {
      LLDB_LOG_ERROR(log, symbol.takeError(), "invalid symbol: {0}");
      continue;
    }
```
- **EN**: Implements logic around `ParseHeader`, `ParseSymtab`, `GetLog`, `GetModule`, and 2 more symbols; this block propagates recoverable errors, status objects, or diagnostics; maps executable state back to modules, symbols, sections, or addresses.
- **CN**: 围绕 `ParseHeader`, `ParseSymtab`, `GetLog`, `GetModule`, and 2 more symbols 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息，并将可执行状态映射回模块、符号、节区或地址。

### Lines 181-190
```cpp
    symtab.AddSymbol(*symbol);
  }
  symtab.Finalize();
}

void ObjectFileJSON::CreateSections(SectionList &unified_section_list) {
  if (m_sections_up)
    return;
  m_sections_up = std::make_unique<SectionList>();

```
- **EN**: Implements logic around `AddSymbol`, `Finalize`, `CreateSections`, `make_unique`; this block maps executable state back to modules, symbols, sections, or addresses.
- **CN**: 围绕 `AddSymbol`, `Finalize`, `CreateSections`, `make_unique` 实现具体逻辑；该代码块将可执行状态映射回模块、符号、节区或地址。

### Lines 191-204
```cpp
  lldb::user_id_t id = 0;
  for (const auto &json_section : m_sections) {
    auto make_section = [this, &id](const JSONSection &section,
                                    SectionSP parent_section_sp =
                                        nullptr) -> SectionSP {
      SectionSP section_sp;
      auto sect_id = section.user_id.value_or(id + 1);
      if (!section.user_id.has_value())
        ++id;
      const auto name = ConstString(section.name);
      const auto sect_type = section.type.value_or(eSectionTypeCode);
      const auto vm_addr = section.address.value_or(0);
      const auto vm_size = section.size.value_or(0);
      const auto file_offset = section.file_offset.value_or(0);
```
- **EN**: Implements logic around `value_or`, `has_value`, `ConstString`; this block maps executable state back to modules, symbols, sections, or addresses.
- **CN**: 围绕 `value_or`, `has_value`, `ConstString` 实现具体逻辑；该代码块将可执行状态映射回模块、符号、节区或地址。

### Lines 205-213
```cpp
      const auto file_size = section.file_size.value_or(0);
      const auto log2align = section.log2align.value_or(0);
      const auto flags = section.flags.value_or(0);
      if (parent_section_sp) {
        section_sp = std::make_shared<Section>(
            parent_section_sp, GetModule(), this, sect_id, name, sect_type,
            vm_addr - parent_section_sp->GetFileAddress(), vm_size, file_offset,
            file_size, log2align, flags);

```
- **EN**: Implements logic around `value_or`, `make_shared`, `GetModule`, `GetFileAddress`; this block maps executable state back to modules, symbols, sections, or addresses.
- **CN**: 围绕 `value_or`, `make_shared`, `GetModule`, `GetFileAddress` 实现具体逻辑；该代码块将可执行状态映射回模块、符号、节区或地址。

### Lines 214-227
```cpp
      } else {
        section_sp = std::make_shared<Section>(
            GetModule(), this, sect_id, name, sect_type, vm_addr, vm_size,
            file_offset, file_size, log2align, flags);
      }
      // Set permissions
      uint32_t permissions = 0;
      if (section.read.value_or(0))
        permissions |= lldb::ePermissionsReadable;
      if (section.write.value_or(0))
        permissions |= lldb::ePermissionsWritable;
      if (section.execute.value_or(0))
        permissions |= lldb::ePermissionsExecutable;
      if (permissions)
```
- **EN**: Implements logic around `make_shared`, `GetModule`, `value_or`; this block maps executable state back to modules, symbols, sections, or addresses.
- **CN**: 围绕 `make_shared`, `GetModule`, `value_or` 实现具体逻辑；该代码块将可执行状态映射回模块、符号、节区或地址。

### Lines 228-239
```cpp
        section_sp->SetPermissions(permissions);
      section_sp->SetIsFake(section.fake.value_or(false));
      section_sp->SetIsEncrypted(section.encrypted.value_or(false));
      section_sp->SetIsThreadSpecific(section.thread_specific.value_or(false));
      return section_sp;
    };
    auto section_sp = make_section(json_section);
    for (const auto &subsection : json_section.subsections) {
      SectionSP subsection_sp = make_section(subsection, section_sp);
      section_sp->GetChildren().AddSection(subsection_sp);
    }

```
- **EN**: Implements logic around `SetPermissions`, `SetIsFake`, `SetIsEncrypted`, `SetIsThreadSpecific`, and 2 more symbols; this block maps executable state back to modules, symbols, sections, or addresses.
- **CN**: 围绕 `SetPermissions`, `SetIsFake`, `SetIsEncrypted`, `SetIsThreadSpecific`, and 2 more symbols 实现具体逻辑；该代码块将可执行状态映射回模块、符号、节区或地址。

### Lines 240-250
```cpp
    m_sections_up->AddSection(section_sp);
    unified_section_list.AddSection(section_sp);
  }
}

bool ObjectFileJSON::SetLoadAddress(Target &target, lldb::addr_t value,
                                    bool value_is_offset) {
  Log *log(GetLog(LLDBLog::DynamicLoader));
  if (!m_sections_up)
    return true;

```
- **EN**: Implements logic around `AddSection`, `SetLoadAddress`, `log`; this block coordinates debugger runtime objects, events, or asynchronous control flow.
- **CN**: 围绕 `AddSection`, `SetLoadAddress`, `log` 实现具体逻辑；该代码块协调调试器运行时对象、事件或异步控制流。

### Lines 251-262
```cpp
  addr_t slide = value;
  if (!value_is_offset) {
    addr_t lowest_addr = LLDB_INVALID_ADDRESS;
    for (const SectionSP &section_sp : *m_sections_up) {
      addr_t section_load_addr = section_sp->GetFileAddress();
      lowest_addr = std::min(lowest_addr, section_load_addr);
    }
    if (lowest_addr == LLDB_INVALID_ADDRESS)
      return false;
    slide = value - lowest_addr;
  }

```
- **EN**: Implements logic around `GetFileAddress`, `min`.
- **CN**: 围绕 `GetFileAddress`, `min` 实现具体逻辑。

### Lines 263-274
```cpp
  // Apply slide to each section's file address.
  for (const SectionSP &section_sp : *m_sections_up) {
    addr_t section_load_addr = section_sp->GetFileAddress();
    if (section_load_addr != LLDB_INVALID_ADDRESS) {
      LLDB_LOG(log,
               "ObjectFileJSON::SetLoadAddress section {0} to load addr {1:x}",
               section_sp->GetName(), section_load_addr + slide);
      target.SetSectionLoadAddress(section_sp, section_load_addr + slide,
                                   /*warn_multiple=*/true);
    }
  }

```
- **EN**: Implements logic around `GetFileAddress`, `LLDB_LOG`, `GetName`, `SetSectionLoadAddress`; this block coordinates debugger runtime objects, events, or asynchronous control flow; maps executable state back to modules, symbols, sections, or addresses; abstracts host OS resources such as files, terminals, sockets, or platforms.
- **CN**: 围绕 `GetFileAddress`, `LLDB_LOG`, `GetName`, `SetSectionLoadAddress` 实现具体逻辑；该代码块协调调试器运行时对象、事件或异步控制流，并将可执行状态映射回模块、符号、节区或地址，并抽象文件、终端、套接字或平台等主机操作系统资源。

### Lines 275-283
```cpp
  return true;
}

bool ObjectFileJSON::MagicBytesMatch(DataExtractorSP extractor_sp) {
  lldb::offset_t offset = 0;
  char magic = extractor_sp->GetU8(&offset);
  return magic == '{';
}

```
- **EN**: Implements logic around `MagicBytesMatch`, `GetU8`.
- **CN**: 围绕 `MagicBytesMatch`, `GetU8` 实现具体逻辑。

### Lines 284-292
```cpp
namespace lldb_private {

bool fromJSON(const json::Value &value, ObjectFileJSON::Header &header,
              json::Path path) {
  json::ObjectMapper o(value, path);
  return o && o.map("triple", header.triple) && o.map("uuid", header.uuid) &&
         o.map("type", header.type);
}

```
- **EN**: Introduces declarations for `lldb_private`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `lldb_private` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 293-299
```cpp
bool fromJSON(const json::Value &value, ObjectFileJSON::Body &body,
              json::Path path) {
  json::ObjectMapper o(value, path);
  return o && o.mapOptional("symbols", body.symbols) &&
         o.mapOptional("sections", body.sections);
}

```
- **EN**: Implements logic around `fromJSON`, `o`, `mapOptional`.
- **CN**: 围绕 `fromJSON`, `o`, `mapOptional` 实现具体逻辑。

### Lines 300-300
```cpp
} // namespace lldb_private
```
- **EN**: Contains supporting implementation details for the surrounding LLDB subsystem.
- **CN**: 包含周边 LLDB 子系统所需的辅助实现细节。

## Key Concepts / 关键概念

- **LLDB architecture / LLDB 架构**:
  - **EN**: Explains how LLDB organizes debugger APIs, runtime objects, and platform-specific helpers.
  - **CN**: 说明 LLDB 如何组织调试器 API、运行时对象以及平台相关辅助组件。
- **Symbolication / 符号化**:
  - **EN**: Relates raw addresses back to modules, sections, symbols, and source locations.
  - **CN**: 将原始地址关联回模块、节区、符号与源码位置。
- **Runtime control / 运行时控制**:
  - **EN**: Models debugger ownership over targets, processes, threads, and event delivery.
  - **CN**: 建模调试器对目标、进程、线程以及事件投递的控制。
- **OS integration / 操作系统集成**:
  - **EN**: Bridges LLDB abstractions to host files, sockets, terminals, and native process facilities.
  - **CN**: 将 LLDB 抽象连接到主机文件、套接字、终端与本地进程设施。

## Dependencies / 依赖关系

- **Direct LLDB/LLVM includes / 直接的 LLDB/LLVM 包含**: `Plugins/ObjectFile/JSON/ObjectFileJSON.h`, `lldb/Core/Module.h`, `lldb/Core/ModuleSpec.h`, `lldb/Core/PluginManager.h`, `lldb/Core/Section.h`, `lldb/Symbol/Symbol.h`, `lldb/Target/Target.h`, `lldb/Utility/LLDBLog.h`, `lldb/Utility/Log.h`, `llvm/ADT/DenseSet.h`
- **Standard-library headers / 标准库头文件**: `<optional>`
- **Subsystem categories / 子系统类别**: LLDB core debugger abstractions / LLDB 核心调试抽象 (4), shared LLDB utility classes / 共享 LLDB 工具类 (2), symbol and debug-info abstractions / 符号与调试信息抽象 (1), target, process, and thread control / 目标、进程与线程控制 (1), LLVM ADT containers and utility types / LLVM ADT 容器与工具类型 (1)
