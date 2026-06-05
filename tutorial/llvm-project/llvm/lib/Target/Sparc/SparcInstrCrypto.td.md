# SparcInstrCrypto.td — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `llvm/lib/Target/Sparc/SparcInstrCrypto.td`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 作用**:
  - **EN**: Defines backend metadata in TableGen DSL that is later expanded into generated tables and matcher code.
  - **CN**: 使用 TableGen DSL 定义后端元数据，随后会展开为生成的表和匹配代码。
- **Note / 说明**:
  - **EN**: This file uses LLVM TableGen DSL, so many definitions are declarative records rather than executable C++ statements.
  - **CN**: 该文件使用 LLVM TableGen DSL，因此许多内容是声明式记录，而不是可执行的 C++ 语句。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```tablegen
//===----------- SparcInstrCrypto.td - cryptographic extensions -----------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM file banner, license notice, and high-level file description.
- **CN**: 包含 LLVM 标准文件头、许可证声明以及该文件的高层说明。

### Lines 8-12
```tablegen
//
// This file contains instruction formats, definitions and patterns needed for
// cryptographic instructions on SPARC.
//===----------------------------------------------------------------------===//

```
- **EN**: Documents the next group of declarations or explains a target-specific rule.
- **CN**: 为接下来的声明分组提供说明，或解释目标相关规则。

### Lines 13-19
```tablegen

// Convenience template for 4-operand instructions
class FourOpImm<string OpcStr, bits<6> op3val, bits<4> op5val,
                RegisterClass RC>
    : F3_4<op3val, op5val, (outs RC:$rd), (ins RC:$rs1, RC:$rs2, simm5Op:$rs3),
      !strconcat(OpcStr, " $rs1, $rs2, $rs3, $rd")>;

```
- **EN**: Declares TableGen records such as `FourOpImm`; this block describes instruction encoding bits.
- **CN**: 声明了 `FourOpImm` 等 TableGen 记录；这一段同时描述指令编码位。

### Lines 20-29
```tablegen
let Predicates = [HasCrypto] in {
def AES_EROUND01 : FourOp<"aes_eround01", 0b011001, 0b0000, DFPRegs>;
def AES_EROUND23 : FourOp<"aes_eround23", 0b011001, 0b0001, DFPRegs>;
def AES_DROUND01 : FourOp<"aes_dround01", 0b011001, 0b0010, DFPRegs>;
def AES_DROUND23 : FourOp<"aes_dround23", 0b011001, 0b0011, DFPRegs>;
def AES_EROUND01_LAST : FourOp<"aes_eround01_l", 0b011001, 0b0100, DFPRegs>;
def AES_EROUND23_LAST : FourOp<"aes_eround23_l", 0b011001, 0b0101, DFPRegs>;
def AES_DROUND01_LAST : FourOp<"aes_dround01_l", 0b011001, 0b0110, DFPRegs>;
def AES_DROUND23_LAST : FourOp<"aes_dround23_l", 0b011001, 0b0111, DFPRegs>;
def AES_KEXPAND0  : F3_3<2, 0b110110, 0b100110000,
```
- **EN**: Declares TableGen records such as `AES_EROUND01`, `AES_EROUND23`, `AES_DROUND01`, `AES_DROUND23`, ...; this block specializes shared fields with `let` assignments.
- **CN**: 声明了 `AES_EROUND01`, `AES_EROUND23`, `AES_DROUND01`, `AES_DROUND23`, ... 等 TableGen 记录；这一段同时通过 `let` 语句特化共享字段。

### Lines 30-36
```tablegen
                         (outs DFPRegs:$rd), (ins DFPRegs:$rs1, DFPRegs:$rs2),
                         "aes_kexpand0 $rs1, $rs2, $rd", []>;
def AES_KEXPAND1 : FourOpImm<"aes_kexpand1", 0b011001, 0b1000, DFPRegs>;
def AES_KEXPAND2  : F3_3<2, 0b110110, 0b100110001,
                         (outs DFPRegs:$rd), (ins DFPRegs:$rs1, DFPRegs:$rs2),
                         "aes_kexpand2 $rs1, $rs2, $rd", []>;

```
- **EN**: Declares TableGen records such as `AES_KEXPAND1`, `AES_KEXPAND2` for the backend description.
- **CN**: 为后端描述声明了 `AES_KEXPAND1`, `AES_KEXPAND2` 等 TableGen 记录。

### Lines 37-44
```tablegen
def CAMELLIA_F : FourOp<"camellia_f", 0b011001, 0b1100, DFPRegs>;
def CAMELLIA_FL  : F3_3<2, 0b110110, 0b100111100,
                        (outs DFPRegs:$rd), (ins DFPRegs:$rs1, DFPRegs:$rs2),
                        "camellia_fl $rs1, $rs2, $rd", []>;
def CAMELLIA_FLI : F3_3<2, 0b110110, 0b100111101,
                        (outs DFPRegs:$rd), (ins DFPRegs:$rs1, DFPRegs:$rs2),
                        "camellia_fli $rs1, $rs2, $rd", []>;

```
- **EN**: Declares TableGen records such as `CAMELLIA_F`, `CAMELLIA_FL`, `CAMELLIA_FLI` for the backend description.
- **CN**: 为后端描述声明了 `CAMELLIA_F`, `CAMELLIA_FL`, `CAMELLIA_FLI` 等 TableGen 记录。

### Lines 45-54
```tablegen
def CRC32C : F3_3<2, 0b110110, 0b101000111,
                  (outs DFPRegs:$rd), (ins DFPRegs:$rs1, DFPRegs:$rs2),
                  "crc32c $rs1, $rs2, $rd", []>;

def DES_ROUND : FourOp<"des_round", 0b011001, 0b1001, DFPRegs>;
let rs2 = 0 in {
def DES_IP  : F3_3<2, 0b110110, 0b100110100,
                   (outs DFPRegs:$rd), (ins DFPRegs:$rs1),
                   "des_ip $rs1, $rd", []>;
def DES_IIP  : F3_3<2, 0b110110, 0b100110101,
```
- **EN**: Declares TableGen records such as `CRC32C`, `DES_ROUND`, `DES_IP`, `DES_IIP`; this block specializes shared fields with `let` assignments.
- **CN**: 声明了 `CRC32C`, `DES_ROUND`, `DES_IP`, `DES_IIP` 等 TableGen 记录；这一段同时通过 `let` 语句特化共享字段。

### Lines 55-61
```tablegen
                    (outs DFPRegs:$rd), (ins DFPRegs:$rs1),
                    "des_iip $rs1, $rd", []>;
}
def DES_KEXPAND : F3_3<2, 0b110110, 0b100110110,
                       (outs DFPRegs:$rd), (ins DFPRegs:$rs1, simm5Op:$rs2),
                       "des_kexpand $rs1, $rs2, $rd", []>;

```
- **EN**: Declares TableGen records such as `DES_KEXPAND` for the backend description.
- **CN**: 为后端描述声明了 `DES_KEXPAND` 等 TableGen 记录。

### Lines 62-71
```tablegen
let rs1 = 0, rs2 = 0, rd = 0 in {
let Uses = [D0, D1, D2, D5, D6, D7, D8, D9, D10, D11],
    Defs = [D0, D1, D2, D3, D4, D5, D6, D7] in
def MD5 : F3_3<2, 0b110110, 0b101000000, (outs), (ins), "md5", []>;
let Uses = [D0, D1, D2, D4, D5, D6, D7, D8, D9, D10, D11],
    Defs = [D0, D1, D2] in
def SHA1 : F3_3<2, 0b110110, 0b101000001, (outs), (ins), "sha1", []>;
let Uses = [D0, D1, D2, D3, D4, D5, D6, D7, D8, D9, D10, D11],
    Defs = [D0, D1, D2, D3] in
def SHA256 : F3_3<2, 0b110110, 0b101000010, (outs), (ins), "sha256", []>;
```
- **EN**: Declares TableGen records such as `MD5`, `SHA1`, `SHA256`; this block specializes shared fields with `let` assignments.
- **CN**: 声明了 `MD5`, `SHA1`, `SHA256` 等 TableGen 记录；这一段同时通过 `let` 语句特化共享字段。

### Lines 72-77
```tablegen
let Uses = [D0, D1, D2, D3, D4, D5, D6, D7, D8, D9, D10, D11,
            D12, D13, D14, D15, D16, D17, D18, D19, D20, D21, D22, D23],
    Defs = [D0, D1, D2, D3, D4, D5, D6, D7] in
def SHA512 : F3_3<2, 0b110110, 0b101000011, (outs), (ins), "sha512", []>;
}

```
- **EN**: Declares TableGen records such as `SHA512`; this block specializes shared fields with `let` assignments.
- **CN**: 声明了 `SHA512` 等 TableGen 记录；这一段同时通过 `let` 语句特化共享字段。

### Lines 78-87
```tablegen
// These instructions use and clobber all DFP and non-reserved Int registers.
let rs1 = 0, rd = 0,
Uses = [ D0,  D1,  D2,  D3,  D4,  D5,  D6,  D7,
         D8,  D9, D10, D11, D12, D13, D14, D15,
        D16, D17, D18, D19, D20, D21, D22, D23,
        D24, D25, D26, D27, D28, D29, D30, D31,
         O0,  O1,  O2,  O3,  O4,  O5,
         L0,  L1,  L2,  L3,  L4,  L5,  L6,  L7,
         I0,  I1,  I2,  I3,  I4,  I5 ],
Defs = [ D0,  D1,  D2,  D3,  D4,  D5,  D6,  D7,
```
- **EN**: Contains supporting implementation details for the surrounding backend logic.
- **CN**: 包含周边后端逻辑所需的辅助实现细节。

### Lines 88-97
```tablegen
         D8,  D9, D10, D11, D12, D13, D14, D15,
        D16, D17, D18, D19, D20, D21, D22, D23,
        D24, D25, D26, D27, D28, D29, D30, D31,
         O0,  O1,  O2,  O3,  O4,  O5,
         L0,  L1,  L2,  L3,  L4,  L5,  L6,  L7,
         I0,  I1,  I2,  I3,  I4,  I5 ] in {
def MPMUL   : F3_3<2, 0b110110, 0b101001000, (outs), (ins simm5Op:$rs2), "mpmul $rs2", []>;
def MONTMUL : F3_3<2, 0b110110, 0b101001001, (outs), (ins simm5Op:$rs2), "montmul $rs2", []>;
def MONTSQR : F3_3<2, 0b110110, 0b101001010, (outs), (ins simm5Op:$rs2), "montsqr $rs2", []>;
}
```
- **EN**: Declares TableGen records such as `MPMUL`, `MONTMUL`, `MONTSQR` for the backend description.
- **CN**: 为后端描述声明了 `MPMUL`, `MONTMUL`, `MONTSQR` 等 TableGen 记录。

### Lines 98-98
```tablegen
} // Predicates = [HasCrypto]
```
- **EN**: Contains supporting implementation details for the surrounding backend logic.
- **CN**: 包含周边后端逻辑所需的辅助实现细节。

## Key Concepts / 关键概念

- **TableGen DSL / TableGen 领域专用语言**:
  - **EN**: Declarative backend description expanded into generated tables
  - **CN**: 以声明式方式描述后端并生成表格代码
- **Generated backend tables / 生成的后端表**:
  - **EN**: The records here feed TableGen generators that emit `.inc` matcher/data tables
  - **CN**: 此处记录会喂给 TableGen 生成器，产出 `.inc` 匹配器/数据表

## Dependencies / 依赖关系

- **LLVM subsystems / LLVM 子系统**: TableGen generators
