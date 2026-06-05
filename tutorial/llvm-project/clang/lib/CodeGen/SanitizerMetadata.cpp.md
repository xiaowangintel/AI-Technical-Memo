# SanitizerMetadata.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `clang/lib/CodeGen/SanitizerMetadata.cpp`
- **Repository / 仓库**: `llvm-project` (`/root/xw/llvm-project`)
- **Purpose (EN) / 目的（英文）**: Implements the SanitizerMetadata portion of Clang's CodeGen pipeline.
- **Purpose (CN) / 目的（中文）**: 实现 Clang CodeGen 流水线中的 SanitizerMetadata 部分。

## Line-by-Line Analysis / 逐行分析

### Lines 1-12
```cpp
 1: //===--- SanitizerMetadata.cpp - Ignored entities for sanitizers ----------===//
 2: //
 3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
 4: // See https://llvm.org/LICENSE.txt for license information.
 5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
 6: //
 7: //===----------------------------------------------------------------------===//
 8: //
 9: // Class which emits metadata consumed by sanitizer instrumentation passes.
10: //
11: //===----------------------------------------------------------------------===//
12: #include "SanitizerMetadata.h"
```
- **EN**: This block imports local CodeGen headers `SanitizerMetadata.h`; contains preprocessor structure that shapes compilation of this unit.
- **CN**: 该代码块引入 本地 CodeGen 头文件 `SanitizerMetadata.h`；包含影响本编译单元构建方式的预处理结构。

### Lines 13-24
```cpp
13: #include "CodeGenModule.h"
14: #include "clang/AST/Attr.h"
15: #include "clang/AST/Type.h"
16: 
17: using namespace clang;
18: using namespace CodeGen;
19: 
20: SanitizerMetadata::SanitizerMetadata(CodeGenModule &CGM) : CGM(CGM) {}
21: 
22: static bool isAsanHwasanMemTagOrTysan(const SanitizerSet &SS) {
23:   return SS.hasOneOf(SanitizerKind::Address | SanitizerKind::KernelAddress |
24:                      SanitizerKind::HWAddress | SanitizerKind::MemTag |
```
- **EN**: This block imports local CodeGen headers `CodeGenModule.h`; Clang headers `clang/AST/Attr.h`, `clang/AST/Type.h`; opens or references namespaces `clang`, `CodeGen`; defines callable entry points like `SanitizerMetadata`, `isAsanHwasanMemTagOrTysan`; returns or forwards computed values for the surrounding Clang CodeGen support logic; contains preprocessor structure that shapes compilation of this unit.
- **CN**: 该代码块引入 本地 CodeGen 头文件 `CodeGenModule.h`；Clang 头文件 `clang/AST/Attr.h`, `clang/AST/Type.h`；打开或引用命名空间 `clang`, `CodeGen`；定义可调用入口，例如 `SanitizerMetadata`, `isAsanHwasanMemTagOrTysan`；为周围的 Clang CodeGen 支撑逻辑 逻辑返回或转发计算结果；包含影响本编译单元构建方式的预处理结构。

### Lines 25-36
```cpp
25:                      SanitizerKind::Type);
26: }
27: 
28: static SanitizerMask expandKernelSanitizerMasks(SanitizerMask Mask) {
29:   if (Mask & (SanitizerKind::Address | SanitizerKind::KernelAddress))
30:     Mask |= SanitizerKind::Address | SanitizerKind::KernelAddress;
31:   // Note: KHWASan doesn't support globals.
32:   return Mask;
33: }
34: 
35: void SanitizerMetadata::reportGlobal(llvm::GlobalVariable *GV,
36:                                      SourceLocation Loc, StringRef Name,
```
- **EN**: This block defines callable entry points like `expandKernelSanitizerMasks`; uses control flow (if) to specialize Clang CodeGen support.
- **CN**: 该代码块定义可调用入口，例如 `expandKernelSanitizerMasks`；通过控制流（if）细化 Clang CodeGen 支撑逻辑 行为。

### Lines 37-48
```cpp
37:                                      QualType Ty,
38:                                      SanitizerMask NoSanitizeAttrMask,
39:                                      bool IsDynInit) {
40:   SanitizerSet FsanitizeArgument = CGM.getLangOpts().Sanitize;
41:   if (!isAsanHwasanMemTagOrTysan(FsanitizeArgument))
42:     return;
43: 
44:   FsanitizeArgument.Mask = expandKernelSanitizerMasks(FsanitizeArgument.Mask);
45:   NoSanitizeAttrMask = expandKernelSanitizerMasks(NoSanitizeAttrMask);
46:   SanitizerSet NoSanitizeAttrSet = {NoSanitizeAttrMask &
47:                                     FsanitizeArgument.Mask};
48: 
```
- **EN**: This block uses control flow (if) to specialize Clang CodeGen support.
- **CN**: 该代码块通过控制流（if）细化 Clang CodeGen 支撑逻辑 行为。

### Lines 49-60
```cpp
49:   llvm::GlobalVariable::SanitizerMetadata Meta;
50:   if (GV->hasSanitizerMetadata())
51:     Meta = GV->getSanitizerMetadata();
52: 
53:   Meta.NoAddress |= NoSanitizeAttrSet.hasOneOf(SanitizerKind::Address);
54:   Meta.NoAddress |= CGM.isInNoSanitizeList(
55:       FsanitizeArgument.Mask & SanitizerKind::Address, GV, Loc, Ty);
56: 
57:   Meta.NoHWAddress |= NoSanitizeAttrSet.hasOneOf(SanitizerKind::HWAddress);
58:   Meta.NoHWAddress |= CGM.isInNoSanitizeList(
59:       FsanitizeArgument.Mask & SanitizerKind::HWAddress, GV, Loc, Ty);
60: 
```
- **EN**: This block uses control flow (if) to specialize Clang CodeGen support.
- **CN**: 该代码块通过控制流（if）细化 Clang CodeGen 支撑逻辑 行为。

### Lines 61-72
```cpp
61:   Meta.Memtag |=
62:       static_cast<bool>(FsanitizeArgument.Mask & SanitizerKind::MemtagGlobals);
63:   Meta.Memtag &= !NoSanitizeAttrSet.hasOneOf(SanitizerKind::MemTag);
64:   Meta.Memtag &= !CGM.isInNoSanitizeList(
65:       FsanitizeArgument.Mask & SanitizerKind::MemTag, GV, Loc, Ty);
66: 
67:   Meta.IsDynInit = IsDynInit && !Meta.NoAddress &&
68:                    FsanitizeArgument.has(SanitizerKind::Address) &&
69:                    !CGM.isInNoSanitizeList(SanitizerKind::Address |
70:                                                SanitizerKind::KernelAddress,
71:                                            GV, Loc, Ty, "init");
72: 
```
- **EN**: This block provides glue code, layout, or delimiters needed by the surrounding Clang CodeGen support implementation.
- **CN**: 该代码块为周围的 Clang CodeGen 支撑逻辑 实现提供必要的胶水代码、布局或分隔结构。

### Lines 73-84
```cpp
73:   GV->setSanitizerMetadata(Meta);
74: 
75:   if (Ty.isNull() || !CGM.getLangOpts().Sanitize.has(SanitizerKind::Type) ||
76:       NoSanitizeAttrMask & SanitizerKind::Type)
77:     return;
78: 
79:   llvm::MDNode *TBAAInfo = CGM.getTBAATypeInfo(Ty);
80:   if (!TBAAInfo || TBAAInfo == CGM.getTBAATypeInfo(CGM.getContext().CharTy))
81:     return;
82: 
83:   llvm::Metadata *GlobalMetadata[] = {llvm::ConstantAsMetadata::get(GV),
84:                                       TBAAInfo};
```
- **EN**: This block uses control flow (if) to specialize Clang CodeGen support.
- **CN**: 该代码块通过控制流（if）细化 Clang CodeGen 支撑逻辑 行为。

### Lines 85-96
```cpp
85: 
86:   // Metadata for the global already registered.
87:   if (llvm::MDNode::getIfExists(CGM.getLLVMContext(), GlobalMetadata))
88:     return;
89: 
90:   llvm::MDNode *ThisGlobal =
91:       llvm::MDNode::get(CGM.getLLVMContext(), GlobalMetadata);
92:   llvm::NamedMDNode *TysanGlobals =
93:       CGM.getModule().getOrInsertNamedMetadata("llvm.tysan.globals");
94:   TysanGlobals->addOperand(ThisGlobal);
95: }
96: 
```
- **EN**: This block spells out callable entry points like `get`; uses control flow (if) to specialize Clang CodeGen support.
- **CN**: 该代码块给出可调用入口的声明，例如 `get`；通过控制流（if）细化 Clang CodeGen 支撑逻辑 行为。

### Lines 97-108
```cpp
 97: void SanitizerMetadata::reportGlobal(llvm::GlobalVariable *GV, const VarDecl &D,
 98:                                      bool IsDynInit) {
 99:   if (!isAsanHwasanMemTagOrTysan(CGM.getLangOpts().Sanitize))
100:     return;
101:   std::string QualName;
102:   llvm::raw_string_ostream OS(QualName);
103:   D.printQualifiedName(OS);
104: 
105:   auto getNoSanitizeMask = [](const VarDecl &D) {
106:     if (D.hasAttr<DisableSanitizerInstrumentationAttr>())
107:       return SanitizerKind::All;
108: 
```
- **EN**: This block defines callable entry points like `reportGlobal`, `OS`; uses control flow (if) to specialize Clang CodeGen support.
- **CN**: 该代码块定义可调用入口，例如 `reportGlobal`, `OS`；通过控制流（if）细化 Clang CodeGen 支撑逻辑 行为。

### Lines 109-120
```cpp
109:     SanitizerMask NoSanitizeMask;
110:     for (auto *Attr : D.specific_attrs<NoSanitizeAttr>())
111:       NoSanitizeMask |= Attr->getMask();
112: 
113:     // External definitions and incomplete types get handled at the place they
114:     // are defined.
115:     if (D.hasExternalStorage() || D.getType()->isIncompleteType())
116:       NoSanitizeMask |= SanitizerKind::Type;
117: 
118:     return NoSanitizeMask;
119:   };
120: 
```
- **EN**: This block uses control flow (if, for) to specialize Clang CodeGen support.
- **CN**: 该代码块通过控制流（if, for）细化 Clang CodeGen 支撑逻辑 行为。

### Lines 121-127
```cpp
121:   reportGlobal(GV, D.getLocation(), QualName, D.getType(), getNoSanitizeMask(D),
122:                IsDynInit);
123: }
124: 
125: void SanitizerMetadata::disableSanitizerForGlobal(llvm::GlobalVariable *GV) {
126:   reportGlobal(GV, SourceLocation(), "", QualType(), SanitizerKind::All);
127: }
```
- **EN**: This block defines callable entry points like `reportGlobal`, `disableSanitizerForGlobal`.
- **CN**: 该代码块定义可调用入口，例如 `reportGlobal`, `disableSanitizerForGlobal`。

## Key Concepts / 关键概念

- **SanitizerKind**: Frequently referenced symbol that likely anchors a key abstraction in this file. / 高频出现的符号，通常对应此文件中的关键抽象。
- **CGM**: Central symbol in this file's implementation of Clang CodeGen support. / 是该文件实现 Clang CodeGen 支撑逻辑 时的核心符号。
- **Meta**: Frequently referenced symbol that likely anchors a key abstraction in this file. / 高频出现的符号，通常对应此文件中的关键抽象。
- **Mask**: Frequently referenced symbol that likely anchors a key abstraction in this file. / 高频出现的符号，通常对应此文件中的关键抽象。
- **FsanitizeArgument**: Frequently referenced symbol that likely anchors a key abstraction in this file. / 高频出现的符号，通常对应此文件中的关键抽象。
- **Address**: Frequently referenced symbol that likely anchors a key abstraction in this file. / 高频出现的符号，通常对应此文件中的关键抽象。
- **Type**: Frequently referenced symbol that likely anchors a key abstraction in this file. / 高频出现的符号，通常对应此文件中的关键抽象。
- **Loc**: Frequently referenced symbol that likely anchors a key abstraction in this file. / 高频出现的符号，通常对应此文件中的关键抽象。

## Dependencies / 依赖关系

- **Local CodeGen / 本地 CodeGen**: `SanitizerMetadata.h`, `CodeGenModule.h`
- **Clang libraries / Clang 库**: `clang/AST/Attr.h`, `clang/AST/Type.h`
