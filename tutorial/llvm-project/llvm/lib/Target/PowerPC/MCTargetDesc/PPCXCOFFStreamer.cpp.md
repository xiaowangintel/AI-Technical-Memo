# PPCXCOFFStreamer.cpp — Code Analysis / 代码分析

## Source / 来源

- **File**: `llvm/lib/Target/PowerPC/MCTargetDesc/PPCXCOFFStreamer.cpp`
- **Repository**: `llvm-project`
- **Purpose (EN)**: This file provides MC layer support for the backend for the PowerPC backend.
- **Purpose (CN)**: 该文件位于 `llvm/lib/Target/PowerPC/MCTargetDesc/PPCXCOFFStreamer.cpp`，主要负责 PowerPC 后端的该后端的 MC 层支持。 文件内容以具体实现、辅助函数和后端决策逻辑为主。

## Line-by-Line Analysis / 逐行分析

### Lines 1-6

```cpp
//===-------- PPCXCOFFStreamer.cpp - XCOFF Object Output ------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
```
- **EN**: Sets up the file banner, licensing information, and the initial context for the rest of the source file.
- **CN**: 这一段给出文件横幅、许可证信息以及后续源码部分的初始上下文。

### Lines 7-17

```cpp
//===----------------------------------------------------------------------===//
//
// This is a custom MCXCOFFStreamer for PowerPC.
//
// The purpose of the custom XCOFF streamer is to allow us to intercept
// instructions as they are being emitted and align all 8 byte instructions
// to a 64 byte boundary if required (by adding a 4 byte nop). This is important
// because 8 byte instructions are not allowed to cross 64 byte boundaries
// and by aligning anything that is within 4 bytes of the boundary we can
// guarantee that the 8 byte instructions do not cross that boundary.
//
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "This is a custom MCXCOFFStreamer for PowerPC.". Notable symbols in this range include `required`.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“This is a custom MCXCOFFStreamer for PowerPC.”。 该区间中较显眼的符号包括 `required`。

### Lines 18-24

```cpp
//===----------------------------------------------------------------------===//

#include "PPCXCOFFStreamer.h"
#include "PPCMCCodeEmitter.h"
#include "llvm/MC/MCAsmBackend.h"
#include "llvm/MC/MCAssembler.h"
#include "llvm/MC/MCCodeEmitter.h"
```
- **EN**: Pulls in direct dependencies required by this MC layer support for the backend, so later declarations can reuse LLVM infrastructure and target-specific helpers. The logic interacts with LLVM's MC layer.
- **CN**: 这一段引入该该后端的 MC 层支持所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。 相关逻辑会与 LLVM 的 MC 层交互。

### Lines 25-62

```cpp
#include "llvm/MC/MCObjectWriter.h"
#include "llvm/MC/TargetRegistry.h"

using namespace llvm;

PPCXCOFFStreamer::PPCXCOFFStreamer(MCContext &Context,
                                   std::unique_ptr<MCAsmBackend> MAB,
                                   std::unique_ptr<MCObjectWriter> OW,
                                   std::unique_ptr<MCCodeEmitter> Emitter)
    : MCXCOFFStreamer(Context, std::move(MAB), std::move(OW),
                      std::move(Emitter)) {}

void PPCXCOFFStreamer::emitPrefixedInstruction(const MCInst &Inst,
                                               const MCSubtargetInfo &STI) {
  // Prefixed instructions must not cross a 64-byte boundary (i.e. prefix is
  // before the boundary and the remaining 4-bytes are after the boundary). In
  // order to achieve this, a nop is added prior to any such boundary-crossing
  // prefixed instruction. Align to 64 bytes if possible but add a maximum of 4
  // bytes when trying to do that. If alignment requires adding more than 4
  // bytes then the instruction won't be aligned.
  emitCodeAlignment(Align(64), &STI, 4);

  // Emit the instruction.
  // Since the previous emit created a new fragment then adding this instruction
  // also forces the addition of a new fragment. Inst is now the first
  // instruction in that new fragment.
  MCXCOFFStreamer::emitInstruction(Inst, STI);
}

void PPCXCOFFStreamer::emitInstruction(const MCInst &Inst,
                                       const MCSubtargetInfo &STI) {
  auto &Emitter = static_cast<PPCMCCodeEmitter &>(getAssembler().getEmitter());

  // Special handling is only for prefixed instructions.
  if (!Emitter.isPrefixedInstruction(Inst)) {
    MCXCOFFStreamer::emitInstruction(Inst, STI);
    return;
  }
```
- **EN**: Pulls in direct dependencies required by this MC layer support for the backend, so later declarations can reuse LLVM infrastructure and target-specific helpers. Subtarget feature gating influences the behavior here.
- **CN**: 这一段引入该该后端的 MC 层支持所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。 子目标特性裁剪会影响这里的行为。

### Lines 63-73

```cpp
  emitPrefixedInstruction(Inst, STI);
}

MCStreamer *
llvm::createPPCXCOFFStreamer(const Triple &, MCContext &C,
                             std::unique_ptr<MCAsmBackend> &&MAB,
                             std::unique_ptr<MCObjectWriter> &&OW,
                             std::unique_ptr<MCCodeEmitter> &&Emitter) {
  return new PPCXCOFFStreamer(C, std::move(MAB), std::move(OW),
                              std::move(Emitter));
}
```
- **EN**: Implements helper routine(s) `emitPrefixedInstruction`, `createPPCXCOFFStreamer`, `PPCXCOFFStreamer` for this portion of the PowerPC backend MC layer support for the backend. The logic interacts with LLVM's MC layer.
- **CN**: 这里实现了 PowerPC 后端该部分该后端的 MC 层支持所需的辅助例程 `emitPrefixedInstruction`, `createPPCXCOFFStreamer`, `PPCXCOFFStreamer`。 相关逻辑会与 LLVM 的 MC 层交互。

## Key Concepts / 关键概念

- PowerPC backend integration / PowerPC 后端集成
- MC layer support for the backend / 该后端的 MC 层支持
- Subtarget features / 子目标特性
- Assembly or MC emission / 汇编或 MC 发射
- Exception-handling support / 异常处理支持

## Dependencies / 依赖关系

### Direct Includes / 直接包含

- `PPCXCOFFStreamer.h`
- `PPCMCCodeEmitter.h`
- `llvm/MC/MCAsmBackend.h`
- `llvm/MC/MCAssembler.h`
- `llvm/MC/MCCodeEmitter.h`
- `llvm/MC/MCObjectWriter.h`
- `llvm/MC/TargetRegistry.h`

### Important Collaborators / 重要协作组件

- LLVM MC layer / LLVM MC 层
- PowerPC target-specific helpers / PowerPC 目标专用辅助组件
