# WebAssemblyTypeUtilities.cpp — Code Analysis / 代码分析

## Source / 来源

- **File**: `llvm/lib/Target/WebAssembly/Utils/WebAssemblyTypeUtilities.cpp`
- **Repository**: `llvm-project`
- **Purpose (EN)**: This file implements several utility functions for WebAssembly type parsing.
- **Purpose (CN)**: 该文件位于 `llvm/lib/Target/WebAssembly/Utils/WebAssemblyTypeUtilities.cpp`，主要负责 WebAssembly 后端的后端共享工具函数。 文件内容以具体实现、辅助函数和后端决策逻辑为主。

## Line-by-Line Analysis / 逐行分析

### Lines 1-6

```cpp
//===-- WebAssemblyTypeUtilities.cpp - WebAssembly Type Utility Functions -===//
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
/// This file implements several utility functions for WebAssembly type parsing.
///
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "\file".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“\file”。

### Lines 12-18

```cpp
//===----------------------------------------------------------------------===//

#include "WebAssemblyTypeUtilities.h"
#include "llvm/ADT/StringSwitch.h"

// Get register classes enum.
#define GET_REGINFO_ENUM
```
- **EN**: Pulls in direct dependencies required by this shared backend utility helpers, so later declarations can reuse LLVM infrastructure and target-specific helpers.
- **CN**: 这一段引入该后端共享工具函数所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。

### Lines 19-56

```cpp
#include "WebAssemblyGenRegisterInfo.inc"

using namespace llvm;

MVT WebAssembly::parseMVT(StringRef Type) {
  return StringSwitch<MVT>(Type)
      .Case("i32", MVT::i32)
      .Case("i64", MVT::i64)
      .Case("f32", MVT::f32)
      .Case("f64", MVT::f64)
      .Case("i64", MVT::i64)
      .Case("v16i8", MVT::v16i8)
      .Case("v8i16", MVT::v8i16)
      .Case("v4i32", MVT::v4i32)
      .Case("v2i64", MVT::v2i64)
      .Case("funcref", MVT::funcref)
      .Case("externref", MVT::externref)
      .Case("exnref", MVT::exnref)
      .Default(MVT::INVALID_SIMPLE_VALUE_TYPE);
}

wasm::ValType WebAssembly::toValType(MVT Type) {
  switch (Type.SimpleTy) {
  case MVT::i32:
    return wasm::ValType::I32;
  case MVT::i64:
    return wasm::ValType::I64;
  case MVT::f32:
    return wasm::ValType::F32;
  case MVT::f64:
    return wasm::ValType::F64;
  case MVT::v16i8:
  case MVT::v8i16:
  case MVT::v4i32:
  case MVT::v2i64:
  case MVT::v8f16:
  case MVT::v4f32:
  case MVT::v2f64:
```
- **EN**: Pulls in direct dependencies required by this shared backend utility helpers, so later declarations can reuse LLVM infrastructure and target-specific helpers. Register assignment and register-class constraints matter here.
- **CN**: 这一段引入该后端共享工具函数所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。 这里重点涉及寄存器分配与寄存器类约束。

### Lines 57-94

```cpp
    return wasm::ValType::V128;
  case MVT::funcref:
    return wasm::ValType::FUNCREF;
  case MVT::externref:
    return wasm::ValType::EXTERNREF;
  case MVT::exnref:
    return wasm::ValType::EXNREF;
  default:
    llvm_unreachable("unexpected type");
  }
}

void WebAssembly::wasmSymbolSetType(MCSymbolWasm *Sym, const Type *GlobalVT,
                                    ArrayRef<MVT> VTs) {
  assert(!Sym->getType());

  // Tables are represented as Arrays in LLVM IR therefore
  // they reach this point as aggregate Array types with an element type
  // that is a reference type.
  wasm::ValType ValTy;
  bool IsTable = false;
  if (WebAssembly::isWebAssemblyTableType(GlobalVT)) {
    IsTable = true;
    const Type *ElTy = GlobalVT->getArrayElementType();
    if (WebAssembly::isWebAssemblyExternrefType(ElTy))
      ValTy = wasm::ValType::EXTERNREF;
    else if (WebAssembly::isWebAssemblyFuncrefType(ElTy))
      ValTy = wasm::ValType::FUNCREF;
    else
      report_fatal_error("unhandled reference type");
  } else if (VTs.size() == 1) {
    ValTy = WebAssembly::toValType(VTs[0]);
  } else
    report_fatal_error("Aggregate globals not yet implemented");

  if (IsTable) {
    Sym->setType(wasm::WASM_SYMBOL_TYPE_TABLE);
    Sym->setTableType(ValTy);
```
- **EN**: Implements helper routine(s) `llvm_unreachable`, `wasmSymbolSetType`, `getType` for this portion of the WebAssembly backend shared backend utility helpers.
- **CN**: 这里实现了 WebAssembly 后端该部分后端共享工具函数所需的辅助例程 `llvm_unreachable`, `wasmSymbolSetType`, `getType`。

### Lines 95-99

```cpp
  } else {
    Sym->setType(wasm::WASM_SYMBOL_TYPE_GLOBAL);
    Sym->setGlobalType(wasm::WasmGlobalType{uint8_t(ValTy), /*Mutable=*/true});
  }
}
```
- **EN**: Implements helper routine(s) `setType`, `setGlobalType`, `uint8_t` for this portion of the WebAssembly backend shared backend utility helpers.
- **CN**: 这里实现了 WebAssembly 后端该部分后端共享工具函数所需的辅助例程 `setType`, `setGlobalType`, `uint8_t`。

## Key Concepts / 关键概念

- WebAssembly backend integration / WebAssembly 后端集成
- Shared backend utility helpers / 后端共享工具函数
- Register modeling / 寄存器建模
- Object format integration / 目标文件格式集成
- Exception-handling support / 异常处理支持

## Dependencies / 依赖关系

### Direct Includes / 直接包含

- `WebAssemblyTypeUtilities.h`
- `llvm/ADT/StringSwitch.h`
- `WebAssemblyGenRegisterInfo.inc`

### Important Collaborators / 重要协作组件

- WebAssembly target-specific helpers / WebAssembly 目标专用辅助组件
