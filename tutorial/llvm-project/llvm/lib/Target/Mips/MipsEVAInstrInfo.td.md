# MipsEVAInstrInfo.td — Code Analysis / 代码分析

## Source / 来源
- File: `llvm/lib/Target/Mips/MipsEVAInstrInfo.td`
- Repository: `llvm-project`
- Purpose (EN): This file describes Mips EVA ASE instructions.
- 用途 (CN): 使用 LLVM TableGen DSL 定义 Mips 后端中的 `MipsEVAInstrInfo`，涵盖指令语义、调度提示以及机器级辅助逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-11
```tablegen
//===- MipsEVAInstrInfo.td - EVA ASE instructions -*- tablegen ------------*-=//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file describes Mips EVA ASE instructions.
//
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 13-17
```tablegen
//===----------------------------------------------------------------------===//
//
// Instruction encodings
//
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 19-24
```tablegen
// Memory Load/Store EVA encodings
class LBE_ENC     : SPECIAL3_EVA_LOAD_STORE_FM<OPCODE6_LBE>;
class LBuE_ENC    : SPECIAL3_EVA_LOAD_STORE_FM<OPCODE6_LBuE>;
class LHE_ENC     : SPECIAL3_EVA_LOAD_STORE_FM<OPCODE6_LHE>;
class LHuE_ENC    : SPECIAL3_EVA_LOAD_STORE_FM<OPCODE6_LHuE>;
class LWE_ENC     : SPECIAL3_EVA_LOAD_STORE_FM<OPCODE6_LWE>;
```
- EN: Declares reusable TableGen class `LBE_ENC` for `MipsEVAInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsEVAInstrInfo` 声明可复用的 TableGen 类 `LBE_ENC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 26-28
```tablegen
class SBE_ENC     : SPECIAL3_EVA_LOAD_STORE_FM<OPCODE6_SBE>;
class SHE_ENC     : SPECIAL3_EVA_LOAD_STORE_FM<OPCODE6_SHE>;
class SWE_ENC     : SPECIAL3_EVA_LOAD_STORE_FM<OPCODE6_SWE>;
```
- EN: Declares reusable TableGen class `SBE_ENC` for `MipsEVAInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsEVAInstrInfo` 声明可复用的 TableGen 类 `SBE_ENC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 30-34
```tablegen
// load/store left/right EVA encodings
class LWLE_ENC    : SPECIAL3_EVA_LOAD_STORE_FM<OPCODE6_LWLE>;
class LWRE_ENC    : SPECIAL3_EVA_LOAD_STORE_FM<OPCODE6_LWRE>;
class SWLE_ENC    : SPECIAL3_EVA_LOAD_STORE_FM<OPCODE6_SWLE>;
class SWRE_ENC    : SPECIAL3_EVA_LOAD_STORE_FM<OPCODE6_SWRE>;
```
- EN: Declares reusable TableGen class `LWLE_ENC` for `MipsEVAInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsEVAInstrInfo` 声明可复用的 TableGen 类 `LWLE_ENC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 36-38
```tablegen
// Load-linked EVA, Store-conditional EVA encodings
class LLE_ENC     : SPECIAL3_EVA_LOAD_STORE_FM<OPCODE6_LLE>;
class SCE_ENC     : SPECIAL3_EVA_LOAD_STORE_FM<OPCODE6_SCE>;
```
- EN: Declares reusable TableGen class `LLE_ENC` for `MipsEVAInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsEVAInstrInfo` 声明可复用的 TableGen 类 `LLE_ENC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 40-41
```tablegen
class TLBINV_ENC  : TLB_FM<OPCODE6_TLBINV>;
class TLBINVF_ENC : TLB_FM<OPCODE6_TLBINVF>;
```
- EN: Declares reusable TableGen class `TLBINV_ENC` for `MipsEVAInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsEVAInstrInfo` 声明可复用的 TableGen 类 `TLBINV_ENC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 43-44
```tablegen
class CACHEE_ENC  : SPECIAL3_EVA_LOAD_STORE_FM<OPCODE6_CACHEE>;
class PREFE_ENC   : SPECIAL3_EVA_LOAD_STORE_FM<OPCODE6_PREFE>;
```
- EN: Declares reusable TableGen class `CACHEE_ENC` for `MipsEVAInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsEVAInstrInfo` 声明可复用的 TableGen 类 `CACHEE_ENC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 46-50
```tablegen
//===----------------------------------------------------------------------===//
//
// Instruction descriptions
//
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 52-62
```tablegen
// Memory Load/Store EVA descriptions
class LOAD_EVA_DESC_BASE<string instr_asm, RegisterOperand GPROpnd> {
  dag OutOperandList = (outs GPROpnd:$rt);
  dag InOperandList = (ins mem_simm9:$addr);
  string AsmString = !strconcat(instr_asm, "\t$rt, $addr");
  list<dag> Pattern = [];
  string DecoderMethod = "DecodeMemEVA";
  bit canFoldAsLoad = 1;
  string BaseOpcode = instr_asm;
  bit mayLoad = 1;
}
```
- EN: Declares reusable TableGen class `LOAD_EVA_DESC_BASE` for `MipsEVAInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsEVAInstrInfo` 声明可复用的 TableGen 类 `LOAD_EVA_DESC_BASE`，通常用于抽象共享字段、谓词或编码结构。

### Lines 64-68
```tablegen
class LBE_DESC  : LOAD_EVA_DESC_BASE<"lbe",  GPR32Opnd>;
class LBuE_DESC : LOAD_EVA_DESC_BASE<"lbue", GPR32Opnd>;
class LHE_DESC  : LOAD_EVA_DESC_BASE<"lhe",  GPR32Opnd>;
class LHuE_DESC : LOAD_EVA_DESC_BASE<"lhue", GPR32Opnd>;
class LWE_DESC  : LOAD_EVA_DESC_BASE<"lwe",  GPR32Opnd>;
```
- EN: Declares reusable TableGen class `LBE_DESC` for `MipsEVAInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsEVAInstrInfo` 声明可复用的 TableGen 类 `LBE_DESC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 70-78
```tablegen
class STORE_EVA_DESC_BASE<string instr_asm, RegisterOperand GPROpnd> {
  dag OutOperandList = (outs);
  dag InOperandList = (ins GPROpnd:$rt, mem_simm9:$addr);
  string AsmString = !strconcat(instr_asm, "\t$rt, $addr");
  list<dag> Pattern = [];
  string DecoderMethod = "DecodeMemEVA";
  string BaseOpcode = instr_asm;
  bit mayStore = 1;
}
```
- EN: Declares reusable TableGen class `STORE_EVA_DESC_BASE` for `MipsEVAInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsEVAInstrInfo` 声明可复用的 TableGen 类 `STORE_EVA_DESC_BASE`，通常用于抽象共享字段、谓词或编码结构。

### Lines 80-82
```tablegen
class SBE_DESC : STORE_EVA_DESC_BASE<"sbe", GPR32Opnd>;
class SHE_DESC : STORE_EVA_DESC_BASE<"she", GPR32Opnd>;
class SWE_DESC : STORE_EVA_DESC_BASE<"swe", GPR32Opnd>;
```
- EN: Declares reusable TableGen class `SBE_DESC` for `MipsEVAInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsEVAInstrInfo` 声明可复用的 TableGen 类 `SBE_DESC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 84-96
```tablegen
// Load/Store Left/Right EVA descriptions
class LOAD_LEFT_RIGHT_EVA_DESC_BASE<string instr_asm, RegisterOperand GPROpnd> {
  dag OutOperandList = (outs GPROpnd:$rt);
  dag InOperandList = (ins mem_simm9:$addr, GPROpnd:$src);
  string AsmString = !strconcat(instr_asm, "\t$rt, $addr");
  list<dag> Pattern = [];
  string DecoderMethod = "DecodeMemEVA";
  string BaseOpcode = instr_asm;
  string Constraints = "$src = $rt";
  bit canFoldAsLoad = 1;
  bit mayLoad = 1;
  bit mayStore = 0;
}
```
- EN: Declares reusable TableGen class `LOAD_LEFT_RIGHT_EVA_DESC_BASE` for `MipsEVAInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsEVAInstrInfo` 声明可复用的 TableGen 类 `LOAD_LEFT_RIGHT_EVA_DESC_BASE`，通常用于抽象共享字段、谓词或编码结构。

### Lines 98-99
```tablegen
class LWLE_DESC : LOAD_LEFT_RIGHT_EVA_DESC_BASE<"lwle", GPR32Opnd>;
class LWRE_DESC : LOAD_LEFT_RIGHT_EVA_DESC_BASE<"lwre", GPR32Opnd>;
```
- EN: Declares reusable TableGen class `LWLE_DESC` for `MipsEVAInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsEVAInstrInfo` 声明可复用的 TableGen 类 `LWLE_DESC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 101-110
```tablegen
class STORE_LEFT_RIGHT_EVA_DESC_BASE<string instr_asm, RegisterOperand GPROpnd> {
  dag OutOperandList = (outs);
  dag InOperandList = (ins GPROpnd:$rt, mem_simm9:$addr);
  string AsmString = !strconcat(instr_asm, "\t$rt, $addr");
  list<dag> Pattern = [];
  string DecoderMethod = "DecodeMemEVA";
  string BaseOpcode = instr_asm;
  bit mayLoad = 0;
  bit mayStore = 1;
}
```
- EN: Declares reusable TableGen class `STORE_LEFT_RIGHT_EVA_DESC_BASE` for `MipsEVAInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsEVAInstrInfo` 声明可复用的 TableGen 类 `STORE_LEFT_RIGHT_EVA_DESC_BASE`，通常用于抽象共享字段、谓词或编码结构。

### Lines 112-113
```tablegen
class SWLE_DESC : STORE_LEFT_RIGHT_EVA_DESC_BASE<"swle", GPR32Opnd>;
class SWRE_DESC : STORE_LEFT_RIGHT_EVA_DESC_BASE<"swre", GPR32Opnd>;
```
- EN: Declares reusable TableGen class `SWLE_DESC` for `MipsEVAInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsEVAInstrInfo` 声明可复用的 TableGen 类 `SWLE_DESC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 115-124
```tablegen
// Load-linked EVA, Store-conditional EVA descriptions
class LLE_DESC_BASE<string instr_asm, RegisterOperand GPROpnd> {
  dag OutOperandList = (outs GPROpnd:$rt);
  dag InOperandList = (ins mem_simm9:$addr);
  string AsmString = !strconcat(instr_asm, "\t$rt, $addr");
  list<dag> Pattern = [];
  string BaseOpcode = instr_asm;
  bit mayLoad = 1;
  string DecoderMethod = "DecodeMemEVA";
}
```
- EN: Declares reusable TableGen class `LLE_DESC_BASE` for `MipsEVAInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsEVAInstrInfo` 声明可复用的 TableGen 类 `LLE_DESC_BASE`，通常用于抽象共享字段、谓词或编码结构。

### Lines 126-126
```tablegen
class LLE_DESC : LLE_DESC_BASE<"lle", GPR32Opnd>;
```
- EN: Declares reusable TableGen class `LLE_DESC` for `MipsEVAInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsEVAInstrInfo` 声明可复用的 TableGen 类 `LLE_DESC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 128-137
```tablegen
class SCE_DESC_BASE<string instr_asm, RegisterOperand GPROpnd> {
  dag OutOperandList = (outs GPROpnd:$dst);
  dag InOperandList = (ins GPROpnd:$rt, mem_simm9:$addr);
  string AsmString = !strconcat(instr_asm, "\t$rt, $addr");
  list<dag> Pattern = [];
  string BaseOpcode = instr_asm;
  bit mayStore = 1;
  string Constraints = "$rt = $dst";
  string DecoderMethod = "DecodeMemEVA";
}
```
- EN: Declares reusable TableGen class `SCE_DESC_BASE` for `MipsEVAInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsEVAInstrInfo` 声明可复用的 TableGen 类 `SCE_DESC_BASE`，通常用于抽象共享字段、谓词或编码结构。

### Lines 139-139
```tablegen
class SCE_DESC : SCE_DESC_BASE<"sce", GPR32Opnd>;
```
- EN: Declares reusable TableGen class `SCE_DESC` for `MipsEVAInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsEVAInstrInfo` 声明可复用的 TableGen 类 `SCE_DESC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 141-146
```tablegen
class TLB_DESC_BASE<string instr_asm> {
  dag OutOperandList = (outs);
  dag InOperandList = (ins);
  string AsmString = instr_asm;
  list<dag> Pattern = [];
}
```
- EN: Declares reusable TableGen class `TLB_DESC_BASE` for `MipsEVAInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsEVAInstrInfo` 声明可复用的 TableGen 类 `TLB_DESC_BASE`，通常用于抽象共享字段、谓词或编码结构。

### Lines 148-149
```tablegen
class TLBINV_DESC  : TLB_DESC_BASE<"tlbinv">;
class TLBINVF_DESC : TLB_DESC_BASE<"tlbinvf">;
```
- EN: Declares reusable TableGen class `TLBINV_DESC` for `MipsEVAInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsEVAInstrInfo` 声明可复用的 TableGen 类 `TLBINV_DESC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 151-154
```tablegen
class CACHEE_DESC_BASE<string instr_asm, Operand MemOpnd> {
  // CACHEE puts the "hint" immediate where the encoding would otherwise have "rt"
  bits<5> hint;
  bits<5> rt = hint;
```
- EN: Declares reusable TableGen class `CACHEE_DESC_BASE` for `MipsEVAInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsEVAInstrInfo` 声明可复用的 TableGen 类 `CACHEE_DESC_BASE`，通常用于抽象共享字段、谓词或编码结构。

### Lines 156-162
```tablegen
  dag OutOperandList = (outs);
  dag InOperandList = (ins  MemOpnd:$addr, uimm5:$hint);
  string AsmString = !strconcat(instr_asm, "\t$hint, $addr");
  list<dag> Pattern = [];
  string BaseOpcode = instr_asm;
  string DecoderMethod = "DecodeCacheeOp_CacheOpR6";
}
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 164-165
```tablegen
class CACHEE_DESC  : CACHEE_DESC_BASE<"cachee", mem_simm9>;
class PREFE_DESC   : CACHEE_DESC_BASE<"prefe", mem_simm9>;
```
- EN: Declares reusable TableGen class `CACHEE_DESC` for `MipsEVAInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsEVAInstrInfo` 声明可复用的 TableGen 类 `CACHEE_DESC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 167-171
```tablegen
//===----------------------------------------------------------------------===//
//
// Instruction definitions
//
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 173-182
```tablegen
let AdditionalPredicates = [NotInMicroMips] in {
  /// Load and Store EVA Instructions
  def LBE     : MMRel, LBE_ENC, LBE_DESC, ISA_MIPS32R2, ASE_EVA;
  def LBuE    : MMRel, LBuE_ENC, LBuE_DESC, ISA_MIPS32R2, ASE_EVA;
  def LHE     : MMRel, LHE_ENC, LHE_DESC, ISA_MIPS32R2, ASE_EVA;
  def LHuE    : MMRel, LHuE_ENC, LHuE_DESC, ISA_MIPS32R2, ASE_EVA;
  def LWE     : MMRel, LWE_ENC, LWE_DESC, ISA_MIPS32R2, ASE_EVA;
  def SBE     : MMRel, SBE_ENC, SBE_DESC, ISA_MIPS32R2, ASE_EVA;
  def SHE     : MMRel, SHE_ENC, SHE_DESC, ISA_MIPS32R2, ASE_EVA;
  def SWE     : MMRel, SWE_ENC, SWE_DESC, ISA_MIPS32R2, ASE_EVA;
```
- EN: Defines TableGen record `LBE` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `LBE`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 184-188
```tablegen
  /// load/store left/right EVA
  def LWLE    : MMRel, LWLE_ENC, LWLE_DESC, ISA_MIPS32R2_NOT_32R6_64R6, ASE_EVA;
  def LWRE    : MMRel, LWRE_ENC, LWRE_DESC, ISA_MIPS32R2_NOT_32R6_64R6, ASE_EVA;
  def SWLE    : MMRel, SWLE_ENC, SWLE_DESC, ISA_MIPS32R2_NOT_32R6_64R6, ASE_EVA;
  def SWRE    : MMRel, SWRE_ENC, SWRE_DESC, ISA_MIPS32R2_NOT_32R6_64R6, ASE_EVA;
```
- EN: Defines TableGen record `LWLE` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `LWLE`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 190-192
```tablegen
  /// Load-linked EVA, Store-conditional EVA
  def LLE     : MMRel, LLE_ENC, LLE_DESC, ISA_MIPS32R2, ASE_EVA;
  def SCE     : MMRel, SCE_ENC, SCE_DESC, ISA_MIPS32R2, ASE_EVA;
```
- EN: Defines TableGen record `LLE` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `LLE`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 194-196
```tablegen
  /// TLB invalidate instructions
  def TLBINV  : TLBINV_ENC, TLBINV_DESC, ISA_MIPS32R2, ASE_EVA;
  def TLBINVF : TLBINVF_ENC, TLBINVF_DESC, ISA_MIPS32R2, ASE_EVA;
```
- EN: Defines TableGen record `TLBINV` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `TLBINV`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 198-201
```tablegen
  /// EVA versions of cache and pref
  def CACHEE  : MMRel, CACHEE_ENC, CACHEE_DESC, ISA_MIPS32R2, ASE_EVA;
  def PREFE   : MMRel, PREFE_ENC, PREFE_DESC, ISA_MIPS32R2, ASE_EVA;
}
```
- EN: Defines TableGen record `CACHEE` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `CACHEE`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

## Key Concepts / 关键概念

- EN: Primary role: instruction semantics, scheduling hints, and machine-level helpers.
  - CN: 核心职责：指令语义、调度提示以及机器级辅助逻辑。
- EN: LLVM TableGen DSL records drive generated enums, matchers, encoders, or scheduling data.
  - CN: LLVM TableGen DSL 记录会驱动生成枚举、匹配器、编码器或调度数据。
- EN: SelectionDAG patterns or node profiles connect IR-level intent to target instructions.
  - CN: SelectionDAG 模式或节点轮廓把 IR 层意图连接到目标指令。
- EN: Bitfield assignments describe exact instruction encodings and per-instruction flags.
  - CN: 位字段赋值用于描述精确的指令编码以及每条指令的标志位。
- EN: Predicates and feature gates control when records apply to a subtarget or ISA variant.
  - CN: 谓词与特性门控决定记录在何种子目标或 ISA 变体下生效。

## Dependencies / 依赖关系

- EN: TableGen includes: `(none)`.
  - CN: TableGen 包含项：`(none)`。
- EN: Generated backend tables depend on tblgen consuming these records and emitting C++/inc fragments.
  - CN: 生成后的后端表依赖 tblgen 消费这些记录并产出 C++/inc 片段。
