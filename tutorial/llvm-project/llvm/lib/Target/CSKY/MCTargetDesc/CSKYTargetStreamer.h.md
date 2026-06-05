# CSKYTargetStreamer.h — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/CSKY/MCTargetDesc/CSKYTargetStreamer.h`
- Repository: `/root/xw/llvm-project`
- Purpose (EN): Defines target-specific assembly streamer hooks for directives and custom MC output.
- 目的（中文）: 定义目标专用的汇编 streamer 钩子，用于处理指令伪操作和自定义 MC 输出。

## Line-by-Line Analysis / 逐行分析

### Lines 1-20

```cpp
   1: //===-- CSKYTargetStreamer.h - CSKY Target Streamer ----------*- C++ -*----===//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: 
   9: #ifndef LLVM_LIB_TARGET_CSKY_CSKYTARGETSTREAMER_H
  10: #define LLVM_LIB_TARGET_CSKY_CSKYTARGETSTREAMER_H
  11: 
  12: #include "MCTargetDesc/CSKYMCAsmInfo.h"
  13: #include "llvm/MC/ConstantPools.h"
  14: #include "llvm/MC/MCStreamer.h"
  15: 
  16: namespace llvm {
  17: 
  18: class CSKYConstantPool {
  19:   using EntryVecTy = SmallVector<ConstantPoolEntry, 4>;
  20:   EntryVecTy Entries;
```

- EN: This opening chunk establishes the file context, including comments, includes, and the first declarations that set up the backend component. The included headers pull in LLVM core, CodeGen, MC, and sibling target declarations needed by this compilation unit. The code enters the LLVM namespace, placing the target implementation into LLVM’s standard backend structure. This chunk introduces interfaces or data structures such as CSKYConstantPool, which organize the target-specific behavior exposed by the file.
- 中文: 这一开头部分建立了文件上下文，包括注释、头文件包含以及用于搭建后端组件的首批声明。 这些头文件引入了 LLVM 核心、CodeGen、MC 以及同目标后端的相关声明，供当前编译单元使用。 代码进入 LLVM 命名空间，使该目标实现位于 LLVM 标准后端结构之中。 这一段引入了 CSKYConstantPool 等接口或数据结构，用于组织该文件暴露的目标专用行为。

### Lines 21-40

```cpp
  21:   std::map<int64_t, const MCSymbolRefExpr *> CachedEntries;
  22: 
  23:   MCSection *CurrentSection = nullptr;
  24: 
  25: public:
  26:   // Initialize a new empty constant pool
  27:   CSKYConstantPool() = default;
  28: 
  29:   // Add a new entry to the constant pool in the next slot.
  30:   // \param Value is the new entry to put in the constant pool.
  31:   // \param Size is the size in bytes of the entry
  32:   //
  33:   // \returns a MCExpr that references the newly inserted value
  34:   const MCExpr *addEntry(MCStreamer &Streamer, const MCExpr *Value,
  35:                          unsigned Size, SMLoc Loc, const MCExpr *AdjustExpr);
  36: 
  37:   void emitAll(MCStreamer &Streamer);
  38: 
  39:   // Return true if the constant pool is empty
  40:   bool empty();
```

- EN: This range continues the implementation of the backend component described by CSKYTargetStreamer.h, adding declarations or executable logic tied to the file’s main purpose.
- 中文: 这一段继续实现该后端组件的核心职责，补充与文件主要目标相关的声明或可执行逻辑。

### Lines 41-60

```cpp
  41: 
  42:   void clearCache();
  43: };
  44: 
  45: class CSKYTargetStreamer : public MCTargetStreamer {
  46: public:
  47:   typedef struct {
  48:     const MCSymbol *sym;
  49:     CSKY::Specifier kind;
  50:   } SymbolIndex;
  51: 
  52: protected:
  53:   std::unique_ptr<CSKYConstantPool> ConstantPool;
  54: 
  55:   DenseMap<SymbolIndex, const MCExpr *> ConstantMap;
  56: 
  57:   unsigned ConstantCounter = 0;
  58: 
  59: public:
  60:   CSKYTargetStreamer(MCStreamer &S);
```

- EN: This chunk introduces interfaces or data structures such as CSKYTargetStreamer, which organize the target-specific behavior exposed by the file.
- 中文: 这一段引入了 CSKYTargetStreamer 等接口或数据结构，用于组织该文件暴露的目标专用行为。

### Lines 61-80

```cpp
  61: 
  62:   virtual void emitTextAttribute(unsigned Attribute, StringRef String);
  63:   virtual void emitAttribute(unsigned Attribute, unsigned Value);
  64:   virtual void finishAttributeSection();
  65: 
  66:   virtual void emitTargetAttributes(const MCSubtargetInfo &STI);
  67:   /// Add a new entry to the constant pool for the current section and return an
  68:   /// MCExpr that can be used to refer to the constant pool location.
  69:   const MCExpr *addConstantPoolEntry(const MCExpr *, SMLoc Loc,
  70:                                      const MCExpr *AdjustExpr = nullptr);
  71: 
  72:   void emitCurrentConstantPool();
  73: 
  74:   void finish() override;
  75: };
  76: 
  77: template <> struct DenseMapInfo<CSKYTargetStreamer::SymbolIndex> {
  78:   static inline CSKYTargetStreamer::SymbolIndex getEmptyKey() {
  79:     return {nullptr, CSKY::S_Invalid};
  80:   }
```

- EN: This chunk introduces interfaces or data structures such as DenseMapInfo, which organize the target-specific behavior exposed by the file. Function bodies or method definitions such as getEmptyKey contain the concrete backend logic executed by LLVM passes or MC helpers.
- 中文: 这一段引入了 DenseMapInfo 等接口或数据结构，用于组织该文件暴露的目标专用行为。 getEmptyKey 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。

### Lines 81-100

```cpp
  81:   static inline CSKYTargetStreamer::SymbolIndex getTombstoneKey() {
  82:     return {nullptr, CSKY::S_Invalid};
  83:   }
  84:   static unsigned getHashValue(const CSKYTargetStreamer::SymbolIndex &V) {
  85:     return hash_combine(DenseMapInfo<const MCSymbol *>::getHashValue(V.sym),
  86:                         DenseMapInfo<int>::getHashValue(V.kind));
  87:   }
  88:   static bool isEqual(const CSKYTargetStreamer::SymbolIndex &A,
  89:                       const CSKYTargetStreamer::SymbolIndex &B) {
  90:     return A.sym == B.sym && A.kind == B.kind;
  91:   }
  92: };
  93: 
  94: class formatted_raw_ostream;
  95: 
  96: class CSKYTargetAsmStreamer : public CSKYTargetStreamer {
  97:   formatted_raw_ostream &OS;
  98: 
  99:   void emitAttribute(unsigned Attribute, unsigned Value) override;
 100:   void emitTextAttribute(unsigned Attribute, StringRef String) override;
```

- EN: This chunk introduces interfaces or data structures such as formatted_raw_ostream, CSKYTargetAsmStreamer, which organize the target-specific behavior exposed by the file. Function bodies or method definitions such as getTombstoneKey, getHashValue, isEqual contain the concrete backend logic executed by LLVM passes or MC helpers.
- 中文: 这一段引入了 formatted_raw_ostream, CSKYTargetAsmStreamer 等接口或数据结构，用于组织该文件暴露的目标专用行为。 getTombstoneKey, getHashValue, isEqual 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。

### Lines 101-110

```cpp
 101:   void finishAttributeSection() override;
 102: 
 103: public:
 104:   CSKYTargetAsmStreamer(MCStreamer &S, formatted_raw_ostream &OS)
 105:       : CSKYTargetStreamer(S), OS(OS) {}
 106: };
 107: 
 108: } // namespace llvm
 109: 
 110: #endif // LLVM_LIB_TARGET_CSKY_CSKYTARGETSTREAMER_H
```

- EN: The code enters the LLVM namespace, placing the target implementation into LLVM’s standard backend structure. Function bodies or method definitions such as CSKYTargetAsmStreamer contain the concrete backend logic executed by LLVM passes or MC helpers.
- 中文: 代码进入 LLVM 命名空间，使该目标实现位于 LLVM 标准后端结构之中。 CSKYTargetAsmStreamer 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。

## Key Concepts / 关键概念

- MC streaming and emission / MC 流式输出
- CPU feature modelling / CPU 特性建模

## Dependencies / 依赖关系

- Direct includes / 直接包含: `MCTargetDesc/CSKYMCAsmInfo.h`, `llvm/MC/ConstantPools.h`, `llvm/MC/MCStreamer.h`
- LLVM subsystems / LLVM 子系统: LLVM MC
- Local companions / 本地配套文件: `CSKYTargetStreamer.cpp`
