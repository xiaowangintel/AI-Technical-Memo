# CodeGenData.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `llvm/lib/CGData/CodeGenData.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: This file contains support for codegen data that has stable summary which can be used to optimize the code in the subsequent codegen.
  - **CN**: 实现代码生成剖析数据的数据结构、读取器与写出器。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```cpp
//===-- CodeGenData.cpp ---------------------------------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM file banner, license notice, and high-level file description.
- **CN**: 包含 LLVM 标准文件头、许可证声明以及文件的高层描述。

### Lines 8-21
```cpp
//
// This file contains support for codegen data that has stable summary which
// can be used to optimize the code in the subsequent codegen.
//
//===----------------------------------------------------------------------===//

#include "llvm/Bitcode/BitcodeWriter.h"
#include "llvm/CGData/CodeGenDataReader.h"
#include "llvm/CGData/OutlinedHashTreeRecord.h"
#include "llvm/CGData/StableFunctionMapRecord.h"
#include "llvm/Object/ObjectFile.h"
#include "llvm/Support/Caching.h"
#include "llvm/Support/CommandLine.h"
#include "llvm/Support/WithColor.h"
```
- **EN**: Pulls in the headers needed by this translation unit, including `llvm/Bitcode/BitcodeWriter.h`, `llvm/CGData/CodeGenDataReader.h`, `llvm/CGData/OutlinedHashTreeRecord.h`, `llvm/CGData/StableFunctionMapRecord.h`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `llvm/Bitcode/BitcodeWriter.h`, `llvm/CGData/CodeGenDataReader.h`, `llvm/CGData/OutlinedHashTreeRecord.h`, `llvm/CGData/StableFunctionMapRecord.h`。

### Lines 22-34
```cpp

#define DEBUG_TYPE "cg-data"

using namespace llvm;
using namespace cgdata;

static cl::opt<bool>
    CodeGenDataGenerate("codegen-data-generate", cl::init(false), cl::Hidden,
                        cl::desc("Emit CodeGen Data into custom sections"));
static cl::opt<std::string>
    CodeGenDataUsePath("codegen-data-use-path", cl::init(""), cl::Hidden,
                       cl::desc("File path to where .cgdata file is read"));

```
- **EN**: Defines preprocessor-controlled structure, feature gates, or compile-time constants.
- **CN**: 定义受预处理器控制的结构、特性开关或编译期常量。

### Lines 35-42
```cpp
namespace llvm {
cl::opt<bool> CodeGenDataThinLTOTwoRounds(
    "codegen-data-thinlto-two-rounds", cl::init(false), cl::Hidden,
    cl::desc("Enable two-round ThinLTO code generation. The first round "
             "emits codegen data, while the second round uses the emitted "
             "codegen data for further optimizations."));
} // end namespace llvm

```
- **EN**: Introduces declarations for `llvm`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `llvm` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 43-56
```cpp
static std::string getCGDataErrString(cgdata_error Err,
                                      const std::string &ErrMsg = "") {
  std::string Msg;
  raw_string_ostream OS(Msg);

  switch (Err) {
  case cgdata_error::success:
    OS << "success";
    break;
  case cgdata_error::eof:
    OS << "end of File";
    break;
  case cgdata_error::bad_magic:
    OS << "invalid codegen data (bad magic)";
```
- **EN**: Implements logic around `getCGDataErrString`, `OS`, `data`; this block propagates recoverable errors through LLVM error utilities; uses `switch`-style dispatch.
- **CN**: 围绕 `getCGDataErrString`, `OS`, `data` 实现具体逻辑；该代码块通过 LLVM 错误工具传播可恢复错误，并使用 `switch` 风格分派。

### Lines 57-70
```cpp
    break;
  case cgdata_error::bad_header:
    OS << "invalid codegen data (file header is corrupt)";
    break;
  case cgdata_error::empty_cgdata:
    OS << "empty codegen data";
    break;
  case cgdata_error::malformed:
    OS << "malformed codegen data";
    break;
  case cgdata_error::unsupported_version:
    OS << "unsupported codegen data version";
    break;
  }
```
- **EN**: Implements logic around `data`.
- **CN**: 围绕 `data` 实现具体逻辑。

### Lines 71-78
```cpp

  // If optional error message is not empty, append it to the message.
  if (!ErrMsg.empty())
    OS << ": " << ErrMsg;

  return OS.str();
}

```
- **EN**: Implements logic around `empty`, `str`; this block propagates recoverable errors through LLVM error utilities.
- **CN**: 围绕 `empty`, `str` 实现具体逻辑；该代码块通过 LLVM 错误工具传播可恢复错误。

### Lines 79-86
```cpp
namespace {

// FIXME: This class is only here to support the transition to llvm::Error. It
// will be removed once this transition is complete. Clients should prefer to
// deal with the Error value directly, rather than converting to error_code.
class CGDataErrorCategoryType : public std::error_category {
  const char *name() const noexcept override { return "llvm.cgdata"; }

```
- **EN**: Introduces declarations for `is`, `CGDataErrorCategoryType`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `is`, `CGDataErrorCategoryType` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 87-93
```cpp
  std::string message(int IE) const override {
    return getCGDataErrString(static_cast<cgdata_error>(IE));
  }
};

} // end anonymous namespace

```
- **EN**: Implements logic around `message`, `getCGDataErrString`.
- **CN**: 围绕 `message`, `getCGDataErrString` 实现具体逻辑。

### Lines 94-102
```cpp
const std::error_category &llvm::cgdata_category() {
  static CGDataErrorCategoryType ErrorCategory;
  return ErrorCategory;
}

std::string CGDataError::message() const {
  return getCGDataErrString(Err, Msg);
}

```
- **EN**: Implements logic around `cgdata_category`, `message`, `getCGDataErrString`.
- **CN**: 围绕 `cgdata_category`, `message`, `getCGDataErrString` 实现具体逻辑。

### Lines 103-112
```cpp
char CGDataError::ID = 0;

namespace {

const char *CodeGenDataSectNameCommon[] = {
#define CG_DATA_SECT_ENTRY(Kind, SectNameCommon, SectNameCoff, Prefix)         \
  SectNameCommon,
#include "llvm/CGData/CodeGenData.inc"
};

```
- **EN**: Pulls in the headers needed by this translation unit, including `llvm/CGData/CodeGenData.inc`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `llvm/CGData/CodeGenData.inc`。

### Lines 113-123
```cpp
const char *CodeGenDataSectNameCoff[] = {
#define CG_DATA_SECT_ENTRY(Kind, SectNameCommon, SectNameCoff, Prefix)         \
  SectNameCoff,
#include "llvm/CGData/CodeGenData.inc"
};

const char *CodeGenDataSectNamePrefix[] = {
#define CG_DATA_SECT_ENTRY(Kind, SectNameCommon, SectNameCoff, Prefix) Prefix,
#include "llvm/CGData/CodeGenData.inc"
};

```
- **EN**: Pulls in the headers needed by this translation unit, including `llvm/CGData/CodeGenData.inc`, `llvm/CGData/CodeGenData.inc`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `llvm/CGData/CodeGenData.inc`, `llvm/CGData/CodeGenData.inc`。

### Lines 124-132
```cpp
} // namespace

namespace llvm {

std::string getCodeGenDataSectionName(CGDataSectKind CGSK,
                                      Triple::ObjectFormatType OF,
                                      bool AddSegmentInfo) {
  std::string SectName;

```
- **EN**: Introduces declarations for `llvm`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `llvm` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 133-140
```cpp
  if (OF == Triple::MachO && AddSegmentInfo)
    SectName = CodeGenDataSectNamePrefix[CGSK];

  if (OF == Triple::COFF)
    SectName += CodeGenDataSectNameCoff[CGSK];
  else
    SectName += CodeGenDataSectNameCommon[CGSK];

```
- **EN**: Contains supporting implementation details for the surrounding LLVM library component.
- **CN**: 包含周边 LLVM 库组件所需的辅助实现细节。

### Lines 141-150
```cpp
  return SectName;
}

std::unique_ptr<CodeGenData> CodeGenData::Instance = nullptr;
std::once_flag CodeGenData::OnceFlag;

CodeGenData &CodeGenData::getInstance() {
  std::call_once(CodeGenData::OnceFlag, []() {
    Instance = std::unique_ptr<CodeGenData>(new CodeGenData());

```
- **EN**: Implements logic around `getInstance`, `call_once`, `unique_ptr`.
- **CN**: 围绕 `getInstance`, `call_once`, `unique_ptr` 实现具体逻辑。

### Lines 151-164
```cpp
    if (CodeGenDataGenerate || CodeGenDataThinLTOTwoRounds)
      Instance->EmitCGData = true;
    else if (!CodeGenDataUsePath.empty()) {
      // Initialize the global CGData if the input file name is given.
      // We do not error-out when failing to parse the input file.
      // Instead, just emit an warning message and fall back as if no CGData
      // were available.
      auto FS = vfs::getRealFileSystem();
      auto ReaderOrErr = CodeGenDataReader::create(CodeGenDataUsePath, *FS);
      if (Error E = ReaderOrErr.takeError()) {
        warn(std::move(E), CodeGenDataUsePath);
        return;
      }
      // Publish each CGData based on the data type in the header.
```
- **EN**: Implements logic around `empty`, `getRealFileSystem`, `create`, `takeError`, and 1 more symbols; this block propagates recoverable errors through LLVM error utilities; parses or classifies structured input; emits or serializes data to an external representation.
- **CN**: 围绕 `empty`, `getRealFileSystem`, `create`, `takeError`, and 1 more symbols 实现具体逻辑；该代码块通过 LLVM 错误工具传播可恢复错误，并解析或分类结构化输入，并把数据输出或序列化为外部表示。

### Lines 165-174
```cpp
      auto Reader = ReaderOrErr->get();
      if (Reader->hasOutlinedHashTree())
        Instance->publishOutlinedHashTree(Reader->releaseOutlinedHashTree());
      if (Reader->hasStableFunctionMap())
        Instance->publishStableFunctionMap(Reader->releaseStableFunctionMap());
    }
  });
  return *Instance;
}

```
- **EN**: Implements logic around `get`, `hasOutlinedHashTree`, `publishOutlinedHashTree`, `hasStableFunctionMap`, and 1 more symbols.
- **CN**: 围绕 `get`, `hasOutlinedHashTree`, `publishOutlinedHashTree`, `hasStableFunctionMap`, and 1 more symbols 实现具体逻辑。

### Lines 175-188
```cpp
namespace IndexedCGData {

Expected<Header> Header::readFromBuffer(const unsigned char *Curr) {
  using namespace support;

  static_assert(std::is_standard_layout_v<llvm::IndexedCGData::Header>,
                "The header should be standard layout type since we use offset "
                "of fields to read.");
  Header H;
  H.Magic = endian::readNext<uint64_t, endianness::little, unaligned>(Curr);
  if (H.Magic != IndexedCGData::Magic)
    return make_error<CGDataError>(cgdata_error::bad_magic);
  H.Version = endian::readNext<uint32_t, endianness::little, unaligned>(Curr);
  if (H.Version > IndexedCGData::CGDataVersion::CurrentVersion)
```
- **EN**: Introduces declarations for `IndexedCGData`, `support`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `IndexedCGData`, `support` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 189-200
```cpp
    return make_error<CGDataError>(cgdata_error::unsupported_version);
  H.DataKind = endian::readNext<uint32_t, endianness::little, unaligned>(Curr);

  static_assert(IndexedCGData::CGDataVersion::CurrentVersion == Version4,
                "Please update the offset computation below if a new field has "
                "been added to the header.");
  H.OutlinedHashTreeOffset =
      endian::readNext<uint64_t, endianness::little, unaligned>(Curr);
  if (H.Version >= 2)
    H.StableFunctionMapOffset =
        endian::readNext<uint64_t, endianness::little, unaligned>(Curr);

```
- **EN**: Implements logic around `make_error`, `unaligned>`, `static_assert`.
- **CN**: 围绕 `make_error`, `unaligned>`, `static_assert` 实现具体逻辑。

### Lines 201-207
```cpp
  return H;
}

} // end namespace IndexedCGData

namespace cgdata {

```
- **EN**: Introduces declarations for `IndexedCGData`, `cgdata`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `IndexedCGData`, `cgdata` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 208-216
```cpp
void warn(Twine Message, StringRef Whence, StringRef Hint) {
  WithColor::warning();
  if (!Whence.empty())
    errs() << Whence << ": ";
  errs() << Message << "\n";
  if (!Hint.empty())
    WithColor::note() << Hint << "\n";
}

```
- **EN**: Implements logic around `warn`, `warning`, `empty`, `errs`, and 1 more symbols.
- **CN**: 围绕 `warn`, `warning`, `empty`, `errs`, and 1 more symbols 实现具体逻辑。

### Lines 217-224
```cpp
void warn(Error E, StringRef Whence) {
  if (E.isA<CGDataError>()) {
    handleAllErrors(std::move(E), [&](const CGDataError &IPE) {
      warn(IPE.message(), Whence, "");
    });
  }
}

```
- **EN**: Implements logic around `warn`, `isA`, `handleAllErrors`; this block propagates recoverable errors through LLVM error utilities.
- **CN**: 围绕 `warn`, `isA`, `handleAllErrors` 实现具体逻辑；该代码块通过 LLVM 错误工具传播可恢复错误。

### Lines 225-234
```cpp
void saveModuleForTwoRounds(const Module &TheModule, unsigned Task,
                            AddStreamFn AddStream) {
  LLVM_DEBUG(dbgs() << "Saving module: " << TheModule.getModuleIdentifier()
                    << " in Task " << Task << "\n");
  Expected<std::unique_ptr<CachedFileStream>> StreamOrErr =
      AddStream(Task, TheModule.getModuleIdentifier());
  if (Error Err = StreamOrErr.takeError())
    report_fatal_error(std::move(Err));
  std::unique_ptr<CachedFileStream> &Stream = *StreamOrErr;

```
- **EN**: Implements logic around `saveModuleForTwoRounds`, `dbgs`, `AddStream`, `takeError`, and 1 more symbols; this block propagates recoverable errors through LLVM error utilities.
- **CN**: 围绕 `saveModuleForTwoRounds`, `dbgs`, `AddStream`, `takeError`, and 1 more symbols 实现具体逻辑；该代码块通过 LLVM 错误工具传播可恢复错误。

### Lines 235-241
```cpp
  WriteBitcodeToFile(TheModule, *Stream->OS,
                     /*ShouldPreserveUseListOrder=*/true);

  if (Error Err = Stream->commit())
    report_fatal_error(std::move(Err));
}

```
- **EN**: Implements logic around `WriteBitcodeToFile`, `commit`, `report_fatal_error`; this block propagates recoverable errors through LLVM error utilities; emits or serializes data to an external representation.
- **CN**: 围绕 `WriteBitcodeToFile`, `commit`, `report_fatal_error` 实现具体逻辑；该代码块通过 LLVM 错误工具传播可恢复错误，并把数据输出或序列化为外部表示。

### Lines 242-255
```cpp
std::unique_ptr<Module> loadModuleForTwoRounds(BitcodeModule &OrigModule,
                                               unsigned Task,
                                               LLVMContext &Context,
                                               ArrayRef<StringRef> IRFiles) {
  LLVM_DEBUG(dbgs() << "Loading module: " << OrigModule.getModuleIdentifier()
                    << " in Task " << Task << "\n");
  auto FileBuffer = MemoryBuffer::getMemBuffer(
      IRFiles[Task], "in-memory IR file", /*RequiresNullTerminator=*/false);
  auto RestoredModule = parseBitcodeFile(*FileBuffer, Context);
  if (!RestoredModule)
    report_fatal_error(
        Twine("Failed to parse optimized bitcode loaded for Task: ") +
        Twine(Task) + "\n");

```
- **EN**: Implements logic around `loadModuleForTwoRounds`, `dbgs`, `getMemBuffer`, `parseBitcodeFile`, and 2 more symbols; this block parses or classifies structured input.
- **CN**: 围绕 `loadModuleForTwoRounds`, `dbgs`, `getMemBuffer`, `parseBitcodeFile`, and 2 more symbols 实现具体逻辑；该代码块解析或分类结构化输入。

### Lines 256-269
```cpp
  // Restore the original module identifier.
  (*RestoredModule)->setModuleIdentifier(OrigModule.getModuleIdentifier());
  return std::move(*RestoredModule);
}

Expected<stable_hash> mergeCodeGenData(ArrayRef<StringRef> ObjFiles) {
  OutlinedHashTreeRecord GlobalOutlineRecord;
  StableFunctionMapRecord GlobalStableFunctionMapRecord;
  stable_hash CombinedHash = 0;
  for (auto File : ObjFiles) {
    if (File.empty())
      continue;
    std::unique_ptr<MemoryBuffer> Buffer = MemoryBuffer::getMemBuffer(
        File, "in-memory object file", /*RequiresNullTerminator=*/false);
```
- **EN**: Implements logic around `setModuleIdentifier`, `move`, `mergeCodeGenData`, `empty`, and 1 more symbols.
- **CN**: 围绕 `setModuleIdentifier`, `move`, `mergeCodeGenData`, `empty`, and 1 more symbols 实现具体逻辑。

### Lines 270-281
```cpp
    Expected<std::unique_ptr<object::ObjectFile>> BinOrErr =
        object::ObjectFile::createObjectFile(Buffer->getMemBufferRef());
    if (!BinOrErr)
      return BinOrErr.takeError();

    std::unique_ptr<object::ObjectFile> &Obj = BinOrErr.get();
    if (auto E = CodeGenDataReader::mergeFromObjectFile(
            Obj.get(), GlobalOutlineRecord, GlobalStableFunctionMapRecord,
            &CombinedHash))
      return E;
  }

```
- **EN**: Implements logic around `createObjectFile`, `takeError`, `get`, `mergeFromObjectFile`; this block propagates recoverable errors through LLVM error utilities.
- **CN**: 围绕 `createObjectFile`, `takeError`, `get`, `mergeFromObjectFile` 实现具体逻辑；该代码块通过 LLVM 错误工具传播可恢复错误。

### Lines 282-289
```cpp
  GlobalStableFunctionMapRecord.finalize();

  if (!GlobalOutlineRecord.empty())
    cgdata::publishOutlinedHashTree(std::move(GlobalOutlineRecord.HashTree));
  if (!GlobalStableFunctionMapRecord.empty())
    cgdata::publishStableFunctionMap(
        std::move(GlobalStableFunctionMapRecord.FunctionMap));

```
- **EN**: Implements logic around `finalize`, `empty`, `publishOutlinedHashTree`, `publishStableFunctionMap`, and 1 more symbols.
- **CN**: 围绕 `finalize`, `empty`, `publishOutlinedHashTree`, `publishStableFunctionMap`, and 1 more symbols 实现具体逻辑。

### Lines 290-295
```cpp
  return CombinedHash;
}

} // end namespace cgdata

} // end namespace llvm
```
- **EN**: Introduces declarations for `cgdata`, `llvm`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `cgdata`, `llvm` 等声明，建立本文件后续使用的类型或命名空间。

## Key Concepts / 关键概念

- **CodeGen profiling data / 代码生成剖析数据**:
  - **EN**: Stores or transfers profile-like data consumed by code-generation workflows.
  - **CN**: 存储或传输代码生成流程消费的剖析类数据。
- **Structured error handling / 结构化错误处理**:
  - **EN**: Uses `Expected`, `Error`, or related helpers to make failures explicit.
  - **CN**: 使用 `Expected`、`Error` 或相关辅助工具显式表示失败。

## Dependencies / 依赖关系

- **Direct LLVM/local includes / 直接的 LLVM/本地包含**: `llvm/Bitcode/BitcodeWriter.h`, `llvm/CGData/CodeGenDataReader.h`, `llvm/CGData/OutlinedHashTreeRecord.h`, `llvm/CGData/StableFunctionMapRecord.h`, `llvm/Object/ObjectFile.h`, `llvm/Support/Caching.h`, `llvm/Support/CommandLine.h`, `llvm/Support/WithColor.h`, `llvm/CGData/CodeGenData.inc`
- **Subsystem categories / 子系统类别**: support-library helpers / Support 库辅助功能 (3), bitcode serialization APIs / bitcode 序列化 API (1), object-file reading abstractions / 目标文件读取抽象 (1)
