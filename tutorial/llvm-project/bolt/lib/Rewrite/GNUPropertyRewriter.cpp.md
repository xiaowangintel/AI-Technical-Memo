# GNUPropertyRewriter.cpp — Code Analysis / 代码分析

## Source / 来源

- File / 文件: `bolt/lib/Rewrite/GNUPropertyRewriter.cpp`
- Repository / 仓库: `llvm-project`
- Purpose / 用途: bolt/Rewrite/GNUPropertyRewriter.cpp Read the .note.gnu.property section.. It also sits in code that implements BOLT rewriting, patching, and binary emission logic. / 该文件实现 BOLT 重写、补丁与二进制输出逻辑。 源码头部说明其职责是：bolt/Rewrite/GNUPropertyRewriter.cpp Read the .note.gnu.property section.。

## Line-by-Line Analysis / 逐行分析

### Lines 1-12

```cpp
//===- bolt/Rewrite/GNUPropertyRewriter.cpp -------------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// Read the .note.gnu.property section.
//
//===----------------------------------------------------------------------===//
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 13-21

```cpp
#include "bolt/Rewrite/MetadataRewriter.h"
#include "bolt/Rewrite/MetadataRewriters.h"
#include "llvm/Support/Errc.h"

using namespace llvm;
using namespace bolt;

namespace {
```

- EN: Pulls in 3 header(s) from local project, LLVM dependencies needed by this range. Works inside namespace scope `llvm`, `bolt` to organize symbols. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `llvm`, `bolt`.
- CN: 这里引入 3 个头文件，为本段提供本地模块、LLVM 或系统层面的依赖。这里位于命名空间 `llvm`, `bolt` 中，用于组织符号作用域。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `llvm`, `bolt`。

### Lines 22-29

```cpp
class GNUPropertyRewriter final : public MetadataRewriter {

  Expected<uint32_t> decodeGNUPropertyNote(StringRef Desc);

public:
  GNUPropertyRewriter(StringRef Name, BinaryContext &BC)
      : MetadataRewriter(Name, BC) {}
```

- EN: Introduces type definitions such as `GNUPropertyRewriter`. Declares or implements routines including `decodeGNUPropertyNote`, `GNUPropertyRewriter`, `MetadataRewriter`. Notable symbols here include `GNUPropertyRewriter`, `decodeGNUPropertyNote`, `MetadataRewriter`.
- CN: 这里引入类型定义，例如 `GNUPropertyRewriter`。这里声明或实现函数，例如 `decodeGNUPropertyNote`, `GNUPropertyRewriter`, `MetadataRewriter`。这里较值得关注的符号包括 `GNUPropertyRewriter`, `decodeGNUPropertyNote`, `MetadataRewriter`。

### Lines 30-39

```cpp
  Error sectionInitializer() override;
};

Error GNUPropertyRewriter::sectionInitializer() {

  ErrorOr<BinarySection &> Sec =
      BC.getUniqueSectionByName(".note.gnu.property");
  if (!Sec)
    return Error::success();
```

- EN: Declares or implements routines including `sectionInitializer`. Notable symbols here include `sectionInitializer`.
- CN: 这里声明或实现函数，例如 `sectionInitializer`。这里较值得关注的符号包括 `sectionInitializer`。

### Lines 40-50

```cpp
  // Accumulate feature bits
  uint32_t FeaturesAcc = 0;

  StringRef Buf = Sec->getContents();
  DataExtractor DE(Buf, BC.AsmInfo->isLittleEndian());
  DataExtractor::Cursor Cursor(0);
  while (Cursor && !DE.eof(Cursor)) {
    const uint32_t NameSz = DE.getU32(Cursor);
    const uint32_t DescSz = DE.getU32(Cursor);
    const uint32_t Type = DE.getU32(Cursor);
```

- EN: Declares or implements routines including `getContents`, `DE`, `Cursor`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `getContents`, `DE`, `Cursor`.
- CN: 这里声明或实现函数，例如 `getContents`, `DE`, `Cursor`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `getContents`, `DE`, `Cursor`。

### Lines 51-64

```cpp
    StringRef Name =
        NameSz ? Buf.slice(Cursor.tell(), Cursor.tell() + NameSz) : "<empty>";
    Cursor.seek(alignTo(Cursor.tell() + NameSz, 4));

    const uint64_t DescOffset = Cursor.tell();
    StringRef Desc =
        DescSz ? Buf.slice(DescOffset, DescOffset + DescSz) : "<empty>";
    Cursor.seek(alignTo(DescOffset + DescSz, 4));
    if (!Cursor)
      return createStringError(
          errc::executable_format_error,
          "out of bounds while reading .note.gnu.property section: %s",
          toString(Cursor.takeError()).c_str());
```

- EN: Declares or implements routines including `toString`. Notable symbols here include `toString`.
- CN: 这里声明或实现函数，例如 `toString`。这里较值得关注的符号包括 `toString`。

### Lines 65-73

```cpp
    if (Type == ELF::NT_GNU_PROPERTY_TYPE_0 && Name.starts_with("GNU") &&
        DescSz) {
      auto Features = decodeGNUPropertyNote(Desc);
      if (!Features)
        return Features.takeError();
      FeaturesAcc |= *Features;
    }
  }
```

- EN: Declares or implements routines including `decodeGNUPropertyNote`. Notable symbols here include `decodeGNUPropertyNote`.
- CN: 这里声明或实现函数，例如 `decodeGNUPropertyNote`。这里较值得关注的符号包括 `decodeGNUPropertyNote`。

### Lines 74-82

```cpp
  if (BC.isAArch64()) {
    BC.setUsesBTI(FeaturesAcc & llvm::ELF::GNU_PROPERTY_AARCH64_FEATURE_1_BTI);
    if (BC.usesBTI())
      BC.outs() << "BOLT-INFO: binary is using BTI\n";
  }

  return Error::success();
}
```

- EN: Contains local control flow that updates state or selects among execution branches.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。

### Lines 83-98

```cpp
/// \p Desc contains an array of property descriptors. Each member has the
/// following structure:
/// typedef struct {
///   Elf_Word pr_type;
///   Elf_Word pr_datasz;
///   unsigned char pr_data[PR_DATASZ];
///   unsigned char pr_padding[PR_PADDING];
/// } Elf_Prop;
///
/// As there is no guarantee that the features are encoded in which element of
/// the array, we have to read all, and OR together the result.
Expected<uint32_t> GNUPropertyRewriter::decodeGNUPropertyNote(StringRef Desc) {
  DataExtractor DE(Desc, BC.AsmInfo->isLittleEndian());
  DataExtractor::Cursor Cursor(0);
  const uint32_t Align = BC.AsmInfo->getCodePointerSize();
```

- EN: Declares or implements routines including `decodeGNUPropertyNote`, `DE`, `Cursor`, `getCodePointerSize`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `decodeGNUPropertyNote`, `DE`, `Cursor`, `getCodePointerSize`.
- CN: 这里声明或实现函数，例如 `decodeGNUPropertyNote`, `DE`, `Cursor`, `getCodePointerSize`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `decodeGNUPropertyNote`, `DE`, `Cursor`, `getCodePointerSize`。

### Lines 99-112

```cpp
  std::optional<uint32_t> Features = 0;
  while (Cursor && !DE.eof(Cursor)) {
    const uint32_t PrType = DE.getU32(Cursor);
    const uint32_t PrDataSz = DE.getU32(Cursor);

    const uint64_t PrDataStart = Cursor.tell();
    const uint64_t PrDataEnd = PrDataStart + PrDataSz;
    Cursor.seek(PrDataEnd);
    if (!Cursor)
      return createStringError(
          errc::executable_format_error,
          "out of bounds while reading .note.gnu.property section: %s",
          toString(Cursor.takeError()).c_str());
```

- EN: Declares or implements routines including `toString`. Notable symbols here include `toString`.
- CN: 这里声明或实现函数，例如 `toString`。这里较值得关注的符号包括 `toString`。

### Lines 113-129

```cpp
    if (PrType == llvm::ELF::GNU_PROPERTY_AARCH64_FEATURE_1_AND) {
      if (PrDataSz != 4) {
        return createStringError(
            errc::executable_format_error,
            "Property descriptor size has to be 4 bytes on AArch64\n");
      }
      DataExtractor::Cursor Tmp(PrDataStart);
      // PrDataSz = 4 -> PrData is uint32_t
      const uint32_t FeaturesItem = DE.getU32(Tmp);
      if (!Tmp)
        return createStringError(
            errc::executable_format_error,
            "failed to read property from .note.gnu.property section: %s",
            toString(Tmp.takeError()).c_str());
      Features = Features ? (*Features | FeaturesItem) : FeaturesItem;
    }
```

- EN: Declares or implements routines including `Tmp`, `toString`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `Tmp`, `toString`.
- CN: 这里声明或实现函数，例如 `Tmp`, `toString`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `Tmp`, `toString`。

### Lines 130-140

```cpp
    Cursor.seek(alignTo(PrDataEnd, Align));
    if (!Cursor)
      return createStringError(errc::executable_format_error,
                               "out of bounds while reading property array in "
                               ".note.gnu.property section: %s",
                               toString(Cursor.takeError()).c_str());
  }
  return Features.value_or(0u);
}
} // namespace
```

- EN: Declares or implements routines including `toString`. Notable symbols here include `toString`.
- CN: 这里声明或实现函数，例如 `toString`。这里较值得关注的符号包括 `toString`。

### Lines 141-144

```cpp
std::unique_ptr<MetadataRewriter>
llvm::bolt::createGNUPropertyRewriter(BinaryContext &BC) {
  return std::make_unique<GNUPropertyRewriter>("gnu-property-rewriter", BC);
}
```

- EN: Declares or implements routines including `createGNUPropertyRewriter`. Notable symbols here include `createGNUPropertyRewriter`.
- CN: 这里声明或实现函数，例如 `createGNUPropertyRewriter`。这里较值得关注的符号包括 `createGNUPropertyRewriter`。

## Key Concepts / 关键概念

- `GNUPropertyRewriter`: class or struct interface / 类或结构体接口
- `decodeGNUPropertyNote`: function or method entry point / 函数或方法入口
- `GNUPropertyRewriter`: function or method entry point / 函数或方法入口
- `MetadataRewriter`: function or method entry point / 函数或方法入口
- `sectionInitializer`: function or method entry point / 函数或方法入口
- `getContents`: function or method entry point / 函数或方法入口
- `llvm`: namespace scope / 命名空间作用域
- `bolt`: namespace scope / 命名空间作用域

## Dependencies / 依赖关系

- Local headers / 本地头文件: `bolt/Rewrite/MetadataRewriter.h`, `bolt/Rewrite/MetadataRewriters.h`
- LLVM headers / LLVM 头文件: `llvm/Support/Errc.h`
- Directory context / 目录上下文: `bolt/lib/Rewrite` neighbors usually cooperate with this file to provide the surrounding subsystem / `bolt/lib/Rewrite` 下的相邻文件通常与本文件协作组成对应子系统
