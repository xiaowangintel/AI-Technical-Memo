# FileOutputBuffer.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `llvm/lib/Support/FileOutputBuffer.cpp`
- Repository: `llvm-project`
- Purpose (EN): Utility for creating a in-memory buffer that will be written to a file.
- Purpose (CN): 该文件位于 LLVM 的 `Support` 目录中，主要实现与 `FileOutputBuffer` 相关的接口、数据结构和辅助逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-40

```cpp
//===- FileOutputBuffer.cpp - File Output Buffer ----------------*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// Utility for creating a in-memory buffer that will be written to a file.
//
//===----------------------------------------------------------------------===//

#include "llvm/Support/FileOutputBuffer.h"
#include "llvm/Support/Errc.h"
#include "llvm/Support/FileSystem.h"
#include "llvm/Support/Memory.h"
#include "llvm/Support/TimeProfiler.h"
#include <system_error>

#if !defined(_MSC_VER) && !defined(__MINGW32__)
#include <unistd.h>
#else
#include <io.h>
#endif

using namespace llvm;
using namespace llvm::sys;

namespace {
// A FileOutputBuffer which creates a temporary file in the same directory
// as the final output file. The final output file is atomically replaced
// with the temporary file on commit().
class OnDiskBuffer : public FileOutputBuffer {
public:
  OnDiskBuffer(StringRef Path, fs::TempFile Temp, fs::mapped_file_region Buf)
      : FileOutputBuffer(Path), Buffer(std::move(Buf)), Temp(std::move(Temp)) {}

  uint8_t *getBufferStart() const override { return (uint8_t *)Buffer.data(); }

  uint8_t *getBufferEnd() const override {
```
- EN: Introduces the file banner, comments, and the initial setup for this translation unit.
  CN: 这一段给出文件横幅、注释说明以及该编译单元的初始设置。
- EN: Brings in 8 direct dependencies, including `llvm/Support/FileOutputBuffer.h`, `llvm/Support/Errc.h`, `llvm/Support/FileSystem.h`, `llvm/Support/Memory.h`.
  CN: 引入了 8 个直接依赖，其中包括 `llvm/Support/FileOutputBuffer.h`, `llvm/Support/Errc.h`, `llvm/Support/FileSystem.h`, `llvm/Support/Memory.h`。
- EN: This section centers on `OnDiskBuffer` and implements the local control flow and bookkeeping.
  CN: 这一段主要围绕 `OnDiskBuffer` 等符号展开，负责实现局部控制流程与状态维护。

### Lines 41-80

```cpp
    return (uint8_t *)Buffer.data() + Buffer.size();
  }

  size_t getBufferSize() const override { return Buffer.size(); }

  Error commit() override {
    llvm::TimeTraceScope timeScope("Commit buffer to disk");

    // Unmap buffer, letting OS flush dirty pages to file on disk.
    Buffer.unmap();

    // Atomically replace the existing file with the new one.
    return Temp.keep(FinalPath);
  }

  ~OnDiskBuffer() override {
    // Close the mapping before deleting the temp file, so that the removal
    // succeeds.
    Buffer.unmap();
    consumeError(Temp.discard());
  }

  void discard() override {
    // Delete the temp file if it still was open, but keeping the mapping
    // active.
    consumeError(Temp.discard());
  }

private:
  fs::mapped_file_region Buffer;
  fs::TempFile Temp;
};

// A FileOutputBuffer which keeps data in memory and writes to the final
// output file on commit(). This is used only when we cannot use OnDiskBuffer.
class InMemoryBuffer : public FileOutputBuffer {
public:
  InMemoryBuffer(StringRef Path, MemoryBlock Buf, std::size_t BufSize,
                 unsigned Mode)
      : FileOutputBuffer(Path), Buffer(Buf), BufferSize(BufSize),
```
- EN: This section centers on `timeScope`, `consumeError` and implements the local control flow and bookkeeping.
  CN: 这一段主要围绕 `timeScope`, `consumeError` 等符号展开，负责实现局部控制流程与状态维护。
- EN: In this range, the code propagates LLVM-style errors and invariants and returns the resulting value to its callers.
  CN: 在这一段中，代码传播 LLVM 风格的错误处理并维护不变量，并将结果返回给调用方。

### Lines 81-120

```cpp
        Mode(Mode) {}

  uint8_t *getBufferStart() const override { return (uint8_t *)Buffer.base(); }

  uint8_t *getBufferEnd() const override {
    return (uint8_t *)Buffer.base() + BufferSize;
  }

  size_t getBufferSize() const override { return BufferSize; }

  Error commit() override {
    if (FinalPath == "-") {
      llvm::outs() << StringRef((const char *)Buffer.base(), BufferSize);
      llvm::outs().flush();
      return Error::success();
    }

    using namespace sys::fs;
    int FD;
    std::error_code EC;
    if (auto EC =
            openFileForWrite(FinalPath, FD, CD_CreateAlways, OF_Delete, Mode))
      return errorCodeToError(EC);
    raw_fd_ostream OS(FD, /*shouldClose=*/true, /*unbuffered=*/true);
    OS << StringRef((const char *)Buffer.base(), BufferSize);
    return Error::success();
  }

private:
  // Buffer may actually contain a larger memory block than BufferSize
  OwningMemoryBlock Buffer;
  size_t BufferSize;
  unsigned Mode;
};
} // namespace

static Expected<std::unique_ptr<InMemoryBuffer>>
createInMemoryBuffer(StringRef Path, size_t Size, unsigned Mode) {
  std::error_code EC;
  MemoryBlock MB = Memory::allocateMappedMemory(
```
- EN: This section centers on `Mode`, `outs`, `success` and creates and initializes supporting objects or state.
  CN: 这一段主要围绕 `Mode`, `outs`, `success` 等符号展开，负责创建并初始化辅助对象或状态。
- EN: In this range, the code checks conditions and handles edge cases and propagates LLVM-style errors and invariants.
  CN: 在这一段中，代码检查条件并处理边界情况，并传播 LLVM 风格的错误处理并维护不变量。

### Lines 121-160

```cpp
      Size, nullptr, sys::Memory::MF_READ | sys::Memory::MF_WRITE, EC);
  if (EC)
    return errorCodeToError(EC);
  return std::make_unique<InMemoryBuffer>(Path, MB, Size, Mode);
}

static Expected<std::unique_ptr<FileOutputBuffer>>
createOnDiskBuffer(StringRef Path, size_t Size, unsigned Mode) {
  Expected<fs::TempFile> FileOrErr =
      fs::TempFile::create(Path + ".tmp%%%%%%%", Mode);
  if (!FileOrErr)
    return FileOrErr.takeError();
  fs::TempFile File = std::move(*FileOrErr);

  if (auto EC = fs::resize_file_before_mapping_readwrite(File.FD, Size)) {
    consumeError(File.discard());
    return errorCodeToError(EC);
  }

  // Mmap it.
  std::error_code EC;
  fs::mapped_file_region MappedFile =
      fs::mapped_file_region(fs::convertFDToNativeFile(File.FD),
                             fs::mapped_file_region::readwrite, Size, 0, EC);

  // mmap(2) can fail if the underlying filesystem does not support it.
  // If that happens, we fall back to in-memory buffer as the last resort.
  if (EC) {
    consumeError(File.discard());
    return createInMemoryBuffer(Path, Size, Mode);
  }

  return std::make_unique<OnDiskBuffer>(Path, std::move(File),
                                         std::move(MappedFile));
}

// Create an instance of FileOutputBuffer.
Expected<std::unique_ptr<FileOutputBuffer>>
FileOutputBuffer::create(StringRef Path, size_t Size, unsigned Flags) {
  // Handle "-" as stdout just like llvm::raw_ostream does.
```
- EN: This section centers on `createOnDiskBuffer`, `create`, `consumeError` and creates and initializes supporting objects or state.
  CN: 这一段主要围绕 `createOnDiskBuffer`, `create`, `consumeError` 等符号展开，负责创建并初始化辅助对象或状态。
- EN: In this range, the code checks conditions and handles edge cases and propagates LLVM-style errors and invariants.
  CN: 在这一段中，代码检查条件并处理边界情况，并传播 LLVM 风格的错误处理并维护不变量。

### Lines 161-196

```cpp
  if (Path == "-")
    return createInMemoryBuffer("-", Size, /*Mode=*/0);

  unsigned Mode = fs::all_read | fs::all_write;
  if (Flags & F_executable)
    Mode |= fs::all_exe;

  // If Size is zero, don't use mmap which will fail with EINVAL.
  if (Size == 0)
    return createInMemoryBuffer(Path, Size, Mode);

  fs::file_status Stat;
  fs::status(Path, Stat);

  // Usually, we want to create OnDiskBuffer to create a temporary file in
  // the same directory as the destination file and atomically replaces it
  // by rename(2).
  //
  // However, if the destination file is a special file, we don't want to
  // use rename (e.g. we don't want to replace /dev/null with a regular
  // file.) If that's the case, we create an in-memory buffer, open the
  // destination file and write to it on commit().
  switch (Stat.type()) {
  case fs::file_type::directory_file:
    return errorCodeToError(errc::is_a_directory);
  case fs::file_type::regular_file:
  case fs::file_type::file_not_found:
  case fs::file_type::status_error:
    if (Flags & F_mmap)
      return createInMemoryBuffer(Path, Size, Mode);
    else
      return createOnDiskBuffer(Path, Size, Mode);
  default:
    return createInMemoryBuffer(Path, Size, Mode);
  }
}
```
- EN: This section centers on `status`, `errorCodeToError`, `createOnDiskBuffer` and creates and initializes supporting objects or state.
  CN: 这一段主要围绕 `status`, `errorCodeToError`, `createOnDiskBuffer` 等符号展开，负责创建并初始化辅助对象或状态。
- EN: In this range, the code uses a `switch` to dispatch behavior by kind or encoding and checks conditions and handles edge cases.
  CN: 在这一段中，代码使用 `switch` 按类型或编码分派行为，并检查条件并处理边界情况。

## Key Concepts / 关键概念
- Domain / 领域: LLVM support utilities / LLVM 支撑工具
- Core symbols / 核心符号: `OnDiskBuffer`, `InMemoryBuffer`, `timeScope`, `consumeError` / 该文件围绕这些类型或函数组织主要逻辑。
- Data flow / 数据流: Formatting, emission, and outward serialization. / 重点关注格式化、输出与序列化。
- Error model / 错误模型: LLVM-style `Error`/`Expected` handling and invariant checks. / 使用 LLVM 风格的 `Error`/`Expected` 处理与不变量检查。

## Dependencies / 依赖关系
- LLVM headers / LLVM 头文件: `llvm/Support/FileOutputBuffer.h`, `llvm/Support/Errc.h`, `llvm/Support/FileSystem.h`, `llvm/Support/Memory.h`, `llvm/Support/TimeProfiler.h`
- Standard library / 标准库: `system_error`, `unistd.h`
- Other/system headers / 其他或系统头文件: `io.h`
- Related symbols / 相关符号: `OnDiskBuffer`, `InMemoryBuffer`, `timeScope`, `consumeError`, `outs`
