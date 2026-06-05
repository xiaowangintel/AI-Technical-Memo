# CBufferDataLayout.cpp — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/DirectX/CBufferDataLayout.cpp`
- Repository: `llvm-project`
- Purpose (EN): Utils to help cbuffer layout.
- 目的（中文）: 该文件提供 LLVM 目标后端所需的辅助逻辑。
- Language: C++ source

## Line-by-Line Analysis / 逐行分析

> Note / 说明: To keep the document readable, the source is analyzed in contiguous line ranges while preserving the original order. Each code block prefixes original line numbers.

### Lines 1-40
```cpp
 1: //===- Target/DirectX/CBufferDataLayout.cpp - Cbuffer layout helper -------===//
 2: //
 3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
 4: // See https://llvm.org/LICENSE.txt for license information.
 5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
 6: //
 7: //===----------------------------------------------------------------------===//
 8: //
 9: // Utils to help cbuffer layout.
10: //
11: //===----------------------------------------------------------------------===//
12:
13: #include "CBufferDataLayout.h"
14:
15: #include "llvm/IR/DerivedTypes.h"
16: #include "llvm/IR/IRBuilder.h"
17:
18: namespace llvm {
19: namespace dxil {
20:
21: // Implement cbuffer layout in
22: // https://learn.microsoft.com/en-us/windows/win32/direct3dhlsl/dx-graphics-hlsl-packing-rules
23: class LegacyCBufferLayout {
24:   struct LegacyStructLayout {
25:     StructType *ST;
26:     SmallVector<uint32_t> Offsets;
27:     TypeSize Size = {0, false};
28:     std::pair<uint32_t, uint32_t> getElementLegacyOffset(unsigned Idx) const {
29:       assert(Idx < Offsets.size() && "Invalid element idx!");
30:       uint32_t Offset = Offsets[Idx];
31:       uint32_t Ch = Offset & (RowAlign - 1);
32:       return std::make_pair((Offset - Ch) / RowAlign, Ch);
33:     }
34:   };
35:
36: public:
37:   LegacyCBufferLayout(const DataLayout &DL) : DL(DL) {}
38:   TypeSize getTypeAllocSizeInBytes(Type *Ty);
39:
40: private:
```
- EN: This range defines or declares important types such as LegacyCBufferLayout, LegacyStructLayout, getElementLegacyOffset, assert, shaping the data model used by CBufferDataLayout.cpp.
- CN: 这一段定义或声明了 LegacyCBufferLayout、LegacyStructLayout、getElementLegacyOffset、assert 等关键类型，构成 CBufferDataLayout.cpp 使用的数据模型。

### Lines 41-80
```cpp
41:   TypeSize applyRowAlign(TypeSize Offset, Type *EltTy);
42:   TypeSize getTypeAllocSize(Type *Ty);
43:   LegacyStructLayout &getStructLayout(StructType *ST);
44:   const DataLayout &DL;
45:   SmallDenseMap<StructType *, LegacyStructLayout> StructLayouts;
46:   // 4 Dwords align.
47:   static const uint32_t RowAlign = 16;
48:   static TypeSize alignTo4Dwords(TypeSize Offset) {
49:     return alignTo(Offset, RowAlign);
50:   }
51: };
52:
53: TypeSize LegacyCBufferLayout::getTypeAllocSizeInBytes(Type *Ty) {
54:   return getTypeAllocSize(Ty);
55: }
56:
57: TypeSize LegacyCBufferLayout::applyRowAlign(TypeSize Offset, Type *EltTy) {
58:   TypeSize AlignedOffset = alignTo4Dwords(Offset);
59:
60:   if (AlignedOffset == Offset)
61:     return Offset;
62:
63:   if (isa<StructType>(EltTy) || isa<ArrayType>(EltTy))
64:     return AlignedOffset;
65:   TypeSize Size = DL.getTypeStoreSize(EltTy);
66:   if ((Offset + Size) > AlignedOffset)
67:     return AlignedOffset;
68:   else
69:     return Offset;
70: }
71:
72: TypeSize LegacyCBufferLayout::getTypeAllocSize(Type *Ty) {
73:   if (auto *ST = dyn_cast<StructType>(Ty)) {
74:     LegacyStructLayout &Layout = getStructLayout(ST);
75:     return Layout.Size;
76:   } else if (auto *AT = dyn_cast<ArrayType>(Ty)) {
77:     unsigned NumElts = AT->getNumElements();
78:     if (NumElts == 0)
79:       return TypeSize::getFixed(0);
80:
```
- EN: This range implements operational logic in helpers such as applyRowAlign, getTypeAllocSize, getStructLayout, alignTo4Dwords, translating backend policy into executable code.
- CN: 这一段实现了 applyRowAlign、getTypeAllocSize、getStructLayout、alignTo4Dwords 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 81-120
```cpp
 81:     TypeSize EltSize = getTypeAllocSize(AT->getElementType());
 82:     TypeSize AlignedEltSize = alignTo4Dwords(EltSize);
 83:     // Each new element start 4 dwords aligned.
 84:     return TypeSize::getFixed(AlignedEltSize * (NumElts - 1) + EltSize);
 85:   } else {
 86:     // NOTE: Use type store size, not align to ABI on basic types for legacy
 87:     // layout.
 88:     return DL.getTypeStoreSize(Ty);
 89:   }
 90: }
 91:
 92: LegacyCBufferLayout::LegacyStructLayout &
 93: LegacyCBufferLayout::getStructLayout(StructType *ST) {
 94:   auto it = StructLayouts.find(ST);
 95:   if (it != StructLayouts.end())
 96:     return it->second;
 97:
 98:   TypeSize Offset = TypeSize::getFixed(0);
 99:   LegacyStructLayout Layout;
100:   Layout.ST = ST;
101:   for (Type *EltTy : ST->elements()) {
102:     TypeSize EltSize = getTypeAllocSize(EltTy);
103:     if (TypeSize ScalarSize = EltTy->getScalarType()->getPrimitiveSizeInBits())
104:       Offset = alignTo(Offset, ScalarSize >> 3);
105:     Offset = applyRowAlign(Offset, EltTy);
106:     Layout.Offsets.emplace_back(Offset);
107:     Offset = Offset.getWithIncrement(EltSize);
108:   }
109:   Layout.Size = Offset;
110:   StructLayouts[ST] = Layout;
111:   return StructLayouts[ST];
112: }
113:
114: CBufferDataLayout::CBufferDataLayout(const DataLayout &DL, const bool IsLegacy)
115:     : DL(DL), IsLegacyLayout(IsLegacy),
116:       LegacyDL(IsLegacy ? std::make_unique<LegacyCBufferLayout>(DL) : nullptr) {
117: }
118:
119: CBufferDataLayout::~CBufferDataLayout() = default;
120:
```
- EN: This range implements operational logic in helpers such as getTypeAllocSize, alignTo4Dwords, TypeSize::getFixed, getTypeStoreSize, translating backend policy into executable code.
- CN: 这一段实现了 getTypeAllocSize、alignTo4Dwords、TypeSize::getFixed、getTypeStoreSize 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 121-129
```cpp
121: llvm::TypeSize CBufferDataLayout::getTypeAllocSizeInBytes(Type *Ty) {
122:   if (IsLegacyLayout)
123:     return LegacyDL->getTypeAllocSizeInBytes(Ty);
124:   else
125:     return DL.getTypeAllocSize(Ty);
126: }
127:
128: } // namespace dxil
129: } // namespace llvm
```
- EN: This range implements operational logic in helpers such as CBufferDataLayout::getTypeAllocSizeInBytes, getTypeAllocSizeInBytes, getTypeAllocSize, translating backend policy into executable code.
- CN: 这一段实现了 CBufferDataLayout::getTypeAllocSizeInBytes、getTypeAllocSizeInBytes、getTypeAllocSize 等操作逻辑，把后端策略落实为可执行的代码路径。

## Key Concepts / 关键概念

- EN: DirectX/DXIL backend code models HLSL/DXIL semantics, including resources, shader stages, and validation-friendly IR forms.
  - CN: DirectX/DXIL 后端代码建模了 HLSL/DXIL 语义，包括资源、着色器阶段以及便于验证的 IR 形式。
- EN: Key symbols in this file include LegacyCBufferLayout, LegacyStructLayout, getElementLegacyOffset, assert, std::make_pair, which anchor the file's main abstractions.
  - CN: 该文件中的关键符号包括 LegacyCBufferLayout, LegacyStructLayout, getElementLegacyOffset, assert, std::make_pair，它们构成了本文件的核心抽象。

## Dependencies / 依赖关系

- Local backend headers / 本地后端头文件:
  - `CBufferDataLayout.h`
- LLVM infrastructure / LLVM 基础设施:
  - `llvm/IR/DerivedTypes.h`
  - `llvm/IR/IRBuilder.h`
