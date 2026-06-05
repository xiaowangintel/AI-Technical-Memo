# ToolUtilities.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `mlir/lib/Support/ToolUtilities.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: This file defines common utilities for implementing MLIR tools.
  - **CN**: 实现供 MLIR 库与工具共享使用的支持工具。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```cpp
//===- ToolUtilities.cpp - MLIR Tool Utilities ----------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM/MLIR file banner, license notice, and high-level summary.
- **CN**: 包含 LLVM/MLIR 标准文件头、许可证声明以及高层摘要。

### Lines 8-12
```cpp
//
// This file defines common utilities for implementing MLIR tools.
//
//===----------------------------------------------------------------------===//

```
- **EN**: Documents the next declarations or records design constraints for the surrounding implementation.
- **CN**: 为接下来的声明提供说明，或记录周边实现的设计约束。

### Lines 13-19
```cpp
#include "mlir/Support/ToolUtilities.h"
#include "mlir/Support/LLVM.h"
#include "llvm/Support/SourceMgr.h"
#include "llvm/Support/raw_ostream.h"
#include <string>
#include <utility>

```
- **EN**: Pulls in the declarations needed by this translation unit, including `mlir/Support/ToolUtilities.h`, `mlir/Support/LLVM.h`, `llvm/Support/SourceMgr.h`, `llvm/Support/raw_ostream.h`.
- **CN**: 引入该编译单元所需的声明，其中包括 `mlir/Support/ToolUtilities.h`, `mlir/Support/LLVM.h`, `llvm/Support/SourceMgr.h`, `llvm/Support/raw_ostream.h`。

### Lines 20-29
```cpp
using namespace mlir;

LogicalResult
mlir::splitAndProcessBuffer(std::unique_ptr<llvm::MemoryBuffer> originalBuffer,
                            ChunkBufferHandler processChunkBuffer,
                            raw_ostream &os, llvm::StringRef inputSplitMarker,
                            llvm::StringRef outputSplitMarker) {
  llvm::MemoryBufferRef originalBufferRef = originalBuffer->getMemBufferRef();
  // If splitting is disabled, we process the full input buffer.
  if (inputSplitMarker.empty())
```
- **EN**: Implements logic around `splitAndProcessBuffer`, `getMemBufferRef`, `empty`; this block makes success/failure or diagnostics explicit through MLIR result utilities.
- **CN**: 围绕 `splitAndProcessBuffer`、`getMemBufferRef`、`empty` 实现具体逻辑；该代码块通过 MLIR 结果工具显式表达成功/失败或诊断。

### Lines 30-39
```cpp
    return processChunkBuffer(std::move(originalBuffer), originalBufferRef, os);

  const int inputSplitMarkerLen = inputSplitMarker.size();

  SmallVector<StringRef, 8> rawSourceBuffers;
  const int checkLen = 2;
  // Split dropping the last checkLen chars to enable flagging near misses.
  originalBufferRef.getBuffer().split(rawSourceBuffers,
                                      inputSplitMarker.drop_back(checkLen));
  if (rawSourceBuffers.empty())
```
- **EN**: Implements logic around `processChunkBuffer`, `size`, `getBuffer`, `drop_back`, and 1 more symbols.
- **CN**: 围绕 `processChunkBuffer`、`size`、`getBuffer`、`drop_back` 等另外 1 个符号 实现具体逻辑。

### Lines 40-45
```cpp
    return success();

  // Add the original buffer to the source manager.
  llvm::SourceMgr fileSourceMgr;
  fileSourceMgr.AddNewSourceBuffer(std::move(originalBuffer), SMLoc());

```
- **EN**: Implements logic around `success`, `AddNewSourceBuffer`.
- **CN**: 围绕 `success`、`AddNewSourceBuffer` 实现具体逻辑。

### Lines 46-55
```cpp
  // Flag near misses by iterating over all the sub-buffers found when splitting
  // with the prefix of the splitMarker. Use a sliding window where we only add
  // a buffer as a sourceBuffer if terminated by a full match of the
  // splitMarker, else flag a warning (if near miss) and extend the size of the
  // buffer under consideration.
  SmallVector<StringRef, 8> sourceBuffers;
  StringRef prev;
  for (auto buffer : rawSourceBuffers) {
    if (prev.empty()) {
      prev = buffer;
```
- **EN**: Implements logic around `empty`.
- **CN**: 围绕 `empty` 实现具体逻辑。

### Lines 56-65
```cpp
      continue;
    }

    // Check that suffix is as expected and doesn't have any dash post.
    bool expectedSuffix =
        buffer.starts_with(inputSplitMarker.take_back(checkLen)) &&
        buffer.size() > checkLen && buffer[checkLen] != '0';
    if (expectedSuffix) {
      sourceBuffers.push_back(prev);
      prev = buffer.drop_front(checkLen);
```
- **EN**: Implements logic around `starts_with`, `size`, `push_back`, `drop_front`.
- **CN**: 围绕 `starts_with`、`size`、`push_back`、`drop_front` 实现具体逻辑。

### Lines 66-75
```cpp
    } else {
      // TODO: Consider making this a failure.
      auto splitLoc = SMLoc::getFromPointer(buffer.data());
      fileSourceMgr.PrintMessage(llvm::errs(), splitLoc,
                                 llvm::SourceMgr::DK_Warning,
                                 "near miss with file split marker");
      prev = StringRef(prev.data(), prev.size() + inputSplitMarkerLen -
                                        checkLen + buffer.size());
    }
  }
```
- **EN**: Implements logic around `getFromPointer`, `PrintMessage`, `StringRef`, `size`; this block parses or prints textual MLIR representations.
- **CN**: 围绕 `getFromPointer`、`PrintMessage`、`StringRef`、`size` 实现具体逻辑；该代码块解析或打印文本形式的 MLIR 表示。

### Lines 76-85
```cpp
  if (!prev.empty())
    sourceBuffers.push_back(prev);

  // Process each chunk in turn.
  bool hadFailure = false;
  auto interleaveFn = [&](StringRef subBuffer) {
    auto splitLoc = SMLoc::getFromPointer(subBuffer.data());
    unsigned splitLine = fileSourceMgr.getLineAndColumn(splitLoc).first;
    std::string name((Twine("within split at ") +
                      originalBufferRef.getBufferIdentifier() + ":" +
```
- **EN**: Implements logic around `empty`, `push_back`, `getFromPointer`, `getLineAndColumn`, and 2 more symbols.
- **CN**: 围绕 `empty`、`push_back`、`getFromPointer`、`getLineAndColumn` 等另外 2 个符号 实现具体逻辑。

### Lines 86-95
```cpp
                      Twine(splitLine) + " offset ")
                         .str());
    // Use MemoryBufferRef to avoid copying the buffer & keep at same location
    // relative to the original buffer.
    auto subMemBuffer =
        llvm::MemoryBuffer::getMemBuffer(llvm::MemoryBufferRef(subBuffer, name),
                                         /*RequiresNullTerminator=*/false);
    if (failed(
            processChunkBuffer(std::move(subMemBuffer), originalBufferRef, os)))
      hadFailure = true;
```
- **EN**: Implements logic around `Twine`, `str`, `getMemBuffer`, `failed`, and 1 more symbols; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `Twine`、`str`、`getMemBuffer`、`failed` 等另外 1 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 96-103
```cpp
  };
  llvm::interleave(sourceBuffers, os, interleaveFn,
                   (llvm::Twine(outputSplitMarker) + "\n").str());

  // If any fails, then return a failure of the tool.
  return failure(hadFailure);
}

```
- **EN**: Implements logic around `interleave`, `Twine`, `failure`.
- **CN**: 围绕 `interleave`、`Twine`、`failure` 实现具体逻辑。

### Lines 104-113
```cpp
LogicalResult
mlir::splitAndProcessBuffer(std::unique_ptr<llvm::MemoryBuffer> originalBuffer,
                            NoSourceChunkBufferHandler processChunkBuffer,
                            raw_ostream &os, llvm::StringRef inputSplitMarker,
                            llvm::StringRef outputSplitMarker) {
  auto process = [&](std::unique_ptr<llvm::MemoryBuffer> chunkBuffer,
                     const llvm::MemoryBufferRef &, raw_ostream &os) {
    return processChunkBuffer(std::move(chunkBuffer), os);
  };
  return splitAndProcessBuffer(std::move(originalBuffer), process, os,
```
- **EN**: Implements logic around `splitAndProcessBuffer`, `processChunkBuffer`; this block makes success/failure or diagnostics explicit through MLIR result utilities.
- **CN**: 围绕 `splitAndProcessBuffer`、`processChunkBuffer` 实现具体逻辑；该代码块通过 MLIR 结果工具显式表达成功/失败或诊断。

### Lines 114-115
```cpp
                               inputSplitMarker, outputSplitMarker);
}
```
- **EN**: Contains supporting implementation details for the surrounding MLIR component.
- **CN**: 包含周边 MLIR 组件所需的辅助实现细节。

## Key Concepts / 关键概念

- **Shared support utilities / 共享支持工具**:
  - **EN**: Provides reusable helpers that are intentionally lower level than dialect-specific logic.
  - **CN**: 提供刻意保持在方言逻辑之下层级的可复用辅助能力。
- **Structured diagnostics / 结构化诊断**:
  - **EN**: Failures are surfaced through `LogicalResult`, diagnostics, or related reporting helpers.
  - **CN**: 通过 `LogicalResult`、诊断对象或相关报告工具显式传播失败。
- **Streaming output / 流式输出**:
  - **EN**: Text, diagnostics, or generated content are emitted through LLVM stream APIs.
  - **CN**: 通过 LLVM 流 API 输出文本、诊断或生成内容。
- **LLVM ADT containers / LLVM ADT 容器**:
  - **EN**: The implementation uses LLVM containers for performance-conscious storage and traversal.
  - **CN**: 该实现使用 LLVM 容器来进行兼顾性能的存储与遍历。

## Dependencies / 依赖关系

- **Direct MLIR/LLVM includes / 直接的 MLIR/LLVM 包含**: `mlir/Support/ToolUtilities.h`, `mlir/Support/LLVM.h`, `llvm/Support/SourceMgr.h`, `llvm/Support/raw_ostream.h`
- **Standard-library headers / 标准库头文件**: `<string>`, `<utility>`
- **Subsystem categories / 子系统类别**: shared MLIR support helpers / 共享的 MLIR 支持工具 (2), LLVM support-library helpers / LLVM Support 库辅助工具 (2)
