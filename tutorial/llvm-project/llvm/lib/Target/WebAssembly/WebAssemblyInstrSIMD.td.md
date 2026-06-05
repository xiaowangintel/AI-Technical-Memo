# WebAssemblyInstrSIMD.td — Code Analysis / 代码分析

## Source / 来源

- **File**: `llvm/lib/Target/WebAssembly/WebAssemblyInstrSIMD.td`
- **Repository**: `llvm-project`
- **Purpose (EN)**: WebAssembly SIMD operand code-gen constructs. It is written in LLVM TableGen DSL and feeds generated backend tables.
- **Purpose (CN)**: 该文件位于 `llvm/lib/Target/WebAssembly/WebAssemblyInstrSIMD.td`，主要负责 WebAssembly 后端的该后端的 TableGen 声明。 它使用 LLVM TableGen DSL 编写，用于描述声明式目标信息并驱动生成代码。
- **Language**: TableGen DSL / TableGen 领域特定语言

## Line-by-Line Analysis / 逐行分析

### Lines 1-6

```tablegen
// WebAssemblyInstrSIMD.td - WebAssembly SIMD codegen support -*- tablegen -*-//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
```
- **EN**: Sets up the file banner, licensing information, and the initial context for the rest of the source file.
- **CN**: 这一段给出文件横幅、许可证信息以及后续源码部分的初始上下文。

### Lines 7-11

```tablegen
//===----------------------------------------------------------------------===//
///
/// \file
/// WebAssembly SIMD operand code-gen constructs.
///
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "\file".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“\file”。

### Lines 12-19

```tablegen
//===----------------------------------------------------------------------===//

// Instructions using the SIMD opcode prefix and requiring one of the SIMD
// feature predicates.
multiclass ABSTRACT_SIMD_I<dag oops_r, dag iops_r, dag oops_s, dag iops_s,
                           list<dag> pattern_r, string asmstr_r,
                           string asmstr_s, bits<32> simdop,
                           list<Predicate> reqs> {
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Instructions using the SIMD opcode prefix and requiring one of the SIMD". These declarations feed generated pattern-matching logic.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Instructions using the SIMD opcode prefix and requiring one of the SIMD”。 这些声明会进入生成式模式匹配逻辑。

### Lines 20-26

```tablegen
  defm "" : I<oops_r, iops_r, oops_s, iops_s, pattern_r, asmstr_r, asmstr_s,
              !if(!ge(simdop, 0x100),
                  !or(0xfd0000, !and(0xffff, simdop)),
                  !or(0xfd00, !and(0xff, simdop)))>,
            Requires<reqs>;
}
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 27-34

```tablegen
multiclass SIMD_I<dag oops_r, dag iops_r, dag oops_s, dag iops_s,
                  list<dag> pattern_r, string asmstr_r = "",
                  string asmstr_s = "", bits<32> simdop = -1,
                  list<Predicate> reqs = []> {
  defm "" : ABSTRACT_SIMD_I<oops_r, iops_r, oops_s, iops_s, pattern_r, asmstr_r,
                            asmstr_s, simdop, !listconcat([HasSIMD128], reqs)>;
}
```
- **EN**: Adds declarative TableGen records such as `SIMD_I` that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `SIMD_I`，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 35-41

```tablegen
multiclass RELAXED_I<dag oops_r, dag iops_r, dag oops_s, dag iops_s,
                     list<dag> pattern_r, string asmstr_r = "",
                     string asmstr_s = "", bits<32> simdop = -1> {
  defm "" : ABSTRACT_SIMD_I<oops_r, iops_r, oops_s, iops_s, pattern_r, asmstr_r,
                            asmstr_s, simdop, [HasRelaxedSIMD]>;
}
```
- **EN**: Adds declarative TableGen records such as `RELAXED_I` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `RELAXED_I`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 42-49

```tablegen
multiclass HALF_PRECISION_I<dag oops_r, dag iops_r, dag oops_s, dag iops_s,
                            list<dag> pattern_r, string asmstr_r = "",
                            string asmstr_s = "", bits<32> simdop = -1> {
  defm "" : ABSTRACT_SIMD_I<oops_r, iops_r, oops_s, iops_s, pattern_r, asmstr_r,
                            asmstr_s, simdop, [HasFP16]>;
}
```
- **EN**: Adds declarative TableGen records such as `HALF_PRECISION_I` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `HALF_PRECISION_I`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 50-59

```tablegen
defm "" : ARGUMENT<V128, v16i8>;
defm "" : ARGUMENT<V128, v8i16>;
defm "" : ARGUMENT<V128, v4i32>;
defm "" : ARGUMENT<V128, v2i64>;
defm "" : ARGUMENT<V128, v4f32>;
defm "" : ARGUMENT<V128, v2f64>;
defm "" : ARGUMENT<V128, v8f16>;

// Constrained immediate argument types. Allow any value from the minimum signed
// value to the maximum unsigned value for the lane size.
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 60-67

```tablegen
foreach SIZE = [8, 16] in
def ImmI#SIZE : ImmLeaf<i32,
  // -2^(n-1) <= Imm < 2^n
  "return -(1 << ("#SIZE#" - 1)) <= Imm && Imm < (1 << "#SIZE#");"
>;
foreach SIZE = [2, 4, 8, 16, 32] in
def LaneIdx#SIZE : ImmLeaf<i32, "return 0 <= Imm && Imm < "#SIZE#";">;
```
- **EN**: Adds declarative TableGen records such as `ImmI`, `LaneIdx` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `ImmI`, `LaneIdx`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 68-80

```tablegen
class Vec {
  ValueType vt;
  ValueType int_vt;
  ValueType lane_vt;
  WebAssemblyRegClass lane_rc;
  int lane_bits;
  ImmLeaf lane_idx;
  SDPatternOperator lane_load;
  PatFrag splat;
  string prefix;
  Vec split;
}
```
- **EN**: Declares a backend-facing type `Vec` and outlines the API or state that nearby code will rely on. These declarations feed generated pattern-matching logic.
- **CN**: 这里声明面向后端的类型 `Vec`，并勾勒出周边代码会依赖的接口或状态。 这些声明会进入生成式模式匹配逻辑。

### Lines 81-87

```tablegen
def I8x16 : Vec {
  let vt = v16i8;
  let int_vt = vt;
  let lane_vt = i32;
  let lane_rc = I32;
  let lane_bits = 8;
  let lane_idx = LaneIdx16;
```
- **EN**: Adds declarative TableGen records such as `I8x16` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `I8x16`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 88-94

```tablegen
  let lane_load = extloadi8;
  let splat = PatFrag<(ops node:$x), (v16i8 (splat_vector (i8 $x)))>;
  let prefix = "i8x16";
}

def I16x8 : Vec {
  let vt = v8i16;
```
- **EN**: Adds declarative TableGen records such as `I16x8` that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `I16x8`，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 95-101

```tablegen
  let int_vt = vt;
  let lane_vt = i32;
  let lane_rc = I32;
  let lane_bits = 16;
  let lane_idx = LaneIdx8;
  let lane_load = extloadi16;
  let splat = PatFrag<(ops node:$x), (v8i16 (splat_vector (i16 $x)))>;
```
- **EN**: Declares function entry points that other backend components call later. These declarations feed generated pattern-matching logic.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。 这些声明会进入生成式模式匹配逻辑。

### Lines 102-108

```tablegen
  let prefix = "i16x8";
  let split = I8x16;
}

def I32x4 : Vec {
  let vt = v4i32;
  let int_vt = vt;
```
- **EN**: Adds declarative TableGen records such as `I32x4` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `I32x4`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 109-115

```tablegen
  let lane_vt = i32;
  let lane_rc = I32;
  let lane_bits = 32;
  let lane_idx = LaneIdx4;
  let lane_load = load;
  let splat = PatFrag<(ops node:$x), (v4i32 (splat_vector (i32 $x)))>;
  let prefix = "i32x4";
```
- **EN**: Declares function entry points that other backend components call later. These declarations feed generated pattern-matching logic.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。 这些声明会进入生成式模式匹配逻辑。

### Lines 116-122

```tablegen
  let split = I16x8;
}

def I64x2 : Vec {
  let vt = v2i64;
  let int_vt = vt;
  let lane_vt = i64;
```
- **EN**: Adds declarative TableGen records such as `I64x2` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `I64x2`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 123-131

```tablegen
  let lane_rc = I64;
  let lane_bits = 64;
  let lane_idx = LaneIdx2;
  let lane_load = load;
  let splat = PatFrag<(ops node:$x), (v2i64 (splat_vector (i64 $x)))>;
  let prefix = "i64x2";
  let split = I32x4;
}
```
- **EN**: Declares function entry points that other backend components call later. These declarations feed generated pattern-matching logic.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。 这些声明会进入生成式模式匹配逻辑。

### Lines 132-138

```tablegen
def F32x4 : Vec {
  let vt = v4f32;
  let int_vt = v4i32;
  let lane_vt = f32;
  let lane_rc = F32;
  let lane_bits = 32;
  let lane_idx = LaneIdx4;
```
- **EN**: Adds declarative TableGen records such as `F32x4` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `F32x4`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 139-145

```tablegen
  let lane_load = load;
  let splat = PatFrag<(ops node:$x), (v4f32 (splat_vector (f32 $x)))>;
  let prefix = "f32x4";
}

def F64x2 : Vec {
  let vt = v2f64;
```
- **EN**: Adds declarative TableGen records such as `F64x2` that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `F64x2`，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 146-152

```tablegen
  let int_vt = v2i64;
  let lane_vt = f64;
  let lane_rc = F64;
  let lane_bits = 64;
  let lane_idx = LaneIdx2;
  let lane_load = load;
  let splat = PatFrag<(ops node:$x), (v2f64 (splat_vector (f64 $x)))>;
```
- **EN**: Declares function entry points that other backend components call later. These declarations feed generated pattern-matching logic.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。 这些声明会进入生成式模式匹配逻辑。

### Lines 153-159

```tablegen
  let prefix = "f64x2";
}

def F16x8 : Vec {
 let vt = v8f16;
 let int_vt = v8i16;
 let lane_vt = f32;
```
- **EN**: Adds declarative TableGen records such as `F16x8` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `F16x8`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 160-172

```tablegen
 let lane_rc = F32;
 let lane_bits = 16;
 let lane_idx = LaneIdx8;
 let lane_load = int_wasm_loadf16_f32;
 let splat = PatFrag<(ops node:$x), (v8f16 (splat_vector (f16 $x)))>;
 let prefix = "f16x8";
}

// TODO: Remove StdVecs when the F16x8 works every where StdVecs is used.
defvar StdVecs = [I8x16, I16x8, I32x4, I64x2, F32x4, F64x2];
defvar AllVecs = !listconcat(StdVecs, [F16x8]);
defvar IntVecs = [I8x16, I16x8, I32x4, I64x2];
```
- **EN**: Declares function entry points that other backend components call later. These declarations feed generated pattern-matching logic.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。 这些声明会进入生成式模式匹配逻辑。

### Lines 173-174

```tablegen
//===----------------------------------------------------------------------===//
// Load and store
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Load and store".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Load and store”。

### Lines 175-183

```tablegen
//===----------------------------------------------------------------------===//

// Load: v128.load
let mayLoad = 1, UseNamedOperandTable = 1 in {
defm LOAD_V128_A32 :
  SIMD_I<(outs V128:$dst), (ins P2Align:$p2align, offset32_op:$off, I32:$addr),
         (outs), (ins P2Align:$p2align, offset32_op:$off), [],
         "v128.load\t$dst, ${off}(${addr})$p2align",
         "v128.load\t$off$p2align", 0>;
```
- **EN**: Adds declarative TableGen records such as `LOAD_V128_A32` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `LOAD_V128_A32`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 184-191

```tablegen
defm LOAD_V128_A64 :
  SIMD_I<(outs V128:$dst), (ins P2Align:$p2align, offset64_op:$off, I64:$addr),
         (outs), (ins P2Align:$p2align, offset64_op:$off), [],
         "v128.load\t$dst, ${off}(${addr})$p2align",
         "v128.load\t$off$p2align", 0>;
}

// Def load patterns from WebAssemblyInstrMemory.td for vector types
```
- **EN**: Adds declarative TableGen records such as `LOAD_V128_A64` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `LOAD_V128_A64`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 192-198

```tablegen
foreach vec = AllVecs in {
defm : LoadPat<vec.vt, load, "LOAD_V128">;
}

// v128.loadX_splat
multiclass SIMDLoadSplat<int size, bits<32> simdop> {
  let mayLoad = 1, UseNamedOperandTable = 1 in {
```
- **EN**: Adds declarative TableGen records such as `SIMDLoadSplat` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `SIMDLoadSplat`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 199-205

```tablegen
  defm LOAD#size#_SPLAT_A32 :
    SIMD_I<(outs V128:$dst),
           (ins P2Align:$p2align, offset32_op:$off, I32:$addr),
           (outs),
           (ins P2Align:$p2align, offset32_op:$off), [],
           "v128.load"#size#"_splat\t$dst, ${off}(${addr})$p2align",
           "v128.load"#size#"_splat\t$off$p2align", simdop>;
```
- **EN**: Continues the WebAssembly backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 WebAssembly 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 206-215

```tablegen
  defm LOAD#size#_SPLAT_A64 :
    SIMD_I<(outs V128:$dst),
           (ins P2Align:$p2align, offset64_op:$off, I64:$addr),
           (outs),
           (ins P2Align:$p2align, offset64_op:$off), [],
           "v128.load"#size#"_splat\t$dst, ${off}(${addr})$p2align",
           "v128.load"#size#"_splat\t$off$p2align", simdop>;
  }
}
```
- **EN**: Continues the WebAssembly backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 WebAssembly 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 216-222

```tablegen
defm "" : SIMDLoadSplat<8, 7>;
defm "" : SIMDLoadSplat<16, 8>;
defm "" : SIMDLoadSplat<32, 9>;
defm "" : SIMDLoadSplat<64, 10>;

foreach vec = StdVecs in {
  defvar inst = "LOAD"#vec.lane_bits#"_SPLAT";
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 223-233

```tablegen
  defm : LoadPat<vec.vt,
                 PatFrag<(ops node:$addr), (splat_vector (vec.lane_vt (vec.lane_load node:$addr)))>,
                 inst>;
}

// Match f16x8.splat of a loaded f16 value and fold it into v128.load16_splat.
def loadf16_any : PatFrags<(ops node:$addr), [
  (extloadf16 node:$addr),
  (int_wasm_loadf16_f32 node:$addr)
]>;
```
- **EN**: Adds declarative TableGen records such as `loadf16_any` that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `loadf16_any`，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 234-242

```tablegen
multiclass LoadSplatFP16Pat<SDPatternOperator kind> {
  def : Pat<(v8f16 (int_wasm_splat_f16x8 (kind (AddrOps32 offset32_op:$offset, I32:$addr)))),
            (LOAD16_SPLAT_A32 0, offset32_op:$offset, I32:$addr)>,
        Requires<[HasAddr32]>;
  def : Pat<(v8f16 (int_wasm_splat_f16x8 (kind (AddrOps64 offset64_op:$offset, I64:$addr)))),
            (LOAD16_SPLAT_A64 0, offset64_op:$offset, I64:$addr)>,
        Requires<[HasAddr64]>;
}
```
- **EN**: Adds declarative TableGen records such as `LoadSplatFP16Pat` that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `LoadSplatFP16Pat`，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 243-249

```tablegen
defm : LoadSplatFP16Pat<loadf16_any>;

// Load and extend
multiclass SIMDLoadExtend<Vec vec, string loadPat, bits<32> simdop> {
  defvar signed = vec.prefix#".load"#loadPat#"_s";
  defvar unsigned = vec.prefix#".load"#loadPat#"_u";
  let mayLoad = 1, UseNamedOperandTable = 1 in {
```
- **EN**: Adds declarative TableGen records such as `SIMDLoadExtend` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `SIMDLoadExtend`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 250-261

```tablegen
  defm LOAD_EXTEND_S_#vec#_A32 :
    SIMD_I<(outs V128:$dst),
           (ins P2Align:$p2align, offset32_op:$off, I32:$addr),
           (outs), (ins P2Align:$p2align, offset32_op:$off), [],
           signed#"\t$dst, ${off}(${addr})$p2align",
           signed#"\t$off$p2align", simdop>;
  defm LOAD_EXTEND_U_#vec#_A32 :
    SIMD_I<(outs V128:$dst),
           (ins P2Align:$p2align, offset32_op:$off, I32:$addr),
           (outs), (ins P2Align:$p2align, offset32_op:$off), [],
           unsigned#"\t$dst, ${off}(${addr})$p2align",
           unsigned#"\t$off$p2align", !add(simdop, 1)>;
```
- **EN**: Implements helper routine(s) for this portion of the WebAssembly backend TableGen declarations for the backend.
- **CN**: 这里实现了 WebAssembly 后端该部分该后端的 TableGen 声明所需的辅助例程。

### Lines 262-276

```tablegen
  defm LOAD_EXTEND_S_#vec#_A64 :
    SIMD_I<(outs V128:$dst),
           (ins P2Align:$p2align, offset64_op:$off, I64:$addr),
           (outs), (ins P2Align:$p2align, offset64_op:$off), [],
           signed#"\t$dst, ${off}(${addr})$p2align",
           signed#"\t$off$p2align", simdop>;
  defm LOAD_EXTEND_U_#vec#_A64 :
    SIMD_I<(outs V128:$dst),
           (ins P2Align:$p2align, offset64_op:$off, I64:$addr),
           (outs), (ins P2Align:$p2align, offset64_op:$off), [],
           unsigned#"\t$dst, ${off}(${addr})$p2align",
           unsigned#"\t$off$p2align", !add(simdop, 1)>;
  }
}
```
- **EN**: Implements helper routine(s) for this portion of the WebAssembly backend TableGen declarations for the backend.
- **CN**: 这里实现了 WebAssembly 后端该部分该后端的 TableGen 声明所需的辅助例程。

### Lines 277-286

```tablegen
defm "" : SIMDLoadExtend<I16x8, "8x8", 1>;
defm "" : SIMDLoadExtend<I32x4, "16x4", 3>;
defm "" : SIMDLoadExtend<I64x2, "32x2", 5>;

foreach vec = [I16x8, I32x4, I64x2] in
foreach exts = [["sextloadvi", "_S"],
                ["zextloadvi", "_U"],
                ["extloadvi", "_U"]] in {
defvar loadpat = !cast<PatFrag>(exts[0]#vec.split.lane_bits);
defvar inst = "LOAD_EXTEND"#exts[1]#"_"#vec;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 287-293

```tablegen
defm : LoadPat<vec.vt, loadpat, inst>;
}

// Load lane into zero vector
multiclass SIMDLoadZero<Vec vec, bits<32> simdop> {
  defvar name = "v128.load"#vec.lane_bits#"_zero";
  let mayLoad = 1, UseNamedOperandTable = 1 in {
```
- **EN**: Adds declarative TableGen records such as `SIMDLoadZero` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `SIMDLoadZero`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 294-308

```tablegen
  defm LOAD_ZERO_#vec.lane_bits#_A32 :
    SIMD_I<(outs V128:$dst),
           (ins P2Align:$p2align, offset32_op:$off, I32:$addr),
           (outs), (ins P2Align:$p2align, offset32_op:$off), [],
           name#"\t$dst, ${off}(${addr})$p2align",
           name#"\t$off$p2align", simdop>;
  defm LOAD_ZERO_#vec.lane_bits#_A64 :
    SIMD_I<(outs V128:$dst),
           (ins P2Align:$p2align, offset64_op:$off, I64:$addr),
           (outs), (ins P2Align:$p2align, offset64_op:$off), [],
           name#"\t$dst, ${off}(${addr})$p2align",
           name#"\t$off$p2align", simdop>;
  } // mayLoad = 1, UseNamedOperandTable = 1
}
```
- **EN**: Continues the WebAssembly backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 WebAssembly 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 309-316

```tablegen
defm "" : SIMDLoadZero<I32x4, 0x5c>;
defm "" : SIMDLoadZero<I64x2, 0x5d>;

// Use load_zero to load scalars into vectors as well where possible.
// TODO: i16, and i8 scalars
foreach vec = [I32x4, I64x2, F32x4, F64x2] in {
  defvar inst = "LOAD_ZERO_"#vec.lane_bits;
  defvar pat = PatFrag<(ops node:$addr), (scalar_to_vector (vec.lane_vt (load $addr)))>;
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Use load_zero to load scalars into vectors as well where possible.". These declarations feed generated pattern-matching logic.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Use load_zero to load scalars into vectors as well where possible.”。 这些声明会进入生成式模式匹配逻辑。

### Lines 317-323

```tablegen
  defm : LoadPat<vec.vt, pat, inst>;
}

foreach vec = [I32x4, I64x2] in {
  defvar inst = "LOAD_ZERO_"#vec.lane_bits;
  defvar pat = PatFrag<(ops node:$ptr),
    (vector_insert (vec.splat (vec.lane_vt 0)), (vec.lane_vt (load $ptr)), 0)>;
```
- **EN**: Implements helper routine(s) for this portion of the WebAssembly backend TableGen declarations for the backend. These declarations feed generated pattern-matching logic.
- **CN**: 这里实现了 WebAssembly 后端该部分该后端的 TableGen 声明所需的辅助例程。 这些声明会进入生成式模式匹配逻辑。

### Lines 324-330

```tablegen
  defm : LoadPat<vec.vt, pat, inst>;
}

foreach vec = [F32x4, F64x2] in {
  defvar inst = "LOAD_ZERO_"#vec.lane_bits;
  defvar pat = PatFrag<(ops node:$ptr),
    (vector_insert (vec.splat (vec.lane_vt fpimm0)), (vec.lane_vt (load $ptr)), 0)>;
```
- **EN**: Implements helper routine(s) for this portion of the WebAssembly backend TableGen declarations for the backend. These declarations feed generated pattern-matching logic.
- **CN**: 这里实现了 WebAssembly 后端该部分该后端的 TableGen 声明所需的辅助例程。 这些声明会进入生成式模式匹配逻辑。

### Lines 331-337

```tablegen
  defm : LoadPat<vec.vt, pat, inst>;
}

// Load lane
multiclass SIMDLoadLane<bits<32> lane_bits, bits<32> simdop> {
  defvar name = "v128.load"#lane_bits#"_lane";
  let mayLoad = 1, UseNamedOperandTable = 1 in {
```
- **EN**: Adds declarative TableGen records such as `SIMDLoadLane` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `SIMDLoadLane`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 338-344

```tablegen
  defm LOAD_LANE_#lane_bits#_A32 :
    SIMD_I<(outs V128:$dst),
           (ins P2Align:$p2align, offset32_op:$off, vec_i8imm_op:$idx,
                I32:$addr, V128:$vec),
           (outs), (ins P2Align:$p2align, offset32_op:$off, vec_i8imm_op:$idx),
           [], name#"\t$dst, ${off}(${addr})$p2align, $vec, $idx",
           name#"\t$off$p2align, $idx", simdop>;
```
- **EN**: Continues the WebAssembly backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 WebAssembly 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 345-354

```tablegen
  defm LOAD_LANE_#lane_bits#_A64 :
    SIMD_I<(outs V128:$dst),
           (ins P2Align:$p2align, offset64_op:$off, vec_i8imm_op:$idx,
                I64:$addr, V128:$vec),
           (outs), (ins P2Align:$p2align, offset64_op:$off, vec_i8imm_op:$idx),
           [], name#"\t$dst, ${off}(${addr})$p2align, $vec, $idx",
           name#"\t$off$p2align, $idx", simdop>;
  } // mayLoad = 1, UseNamedOperandTable = 1
}
```
- **EN**: Continues the WebAssembly backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 WebAssembly 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 355-364

```tablegen
defm "" : SIMDLoadLane<8, 0x54>;
defm "" : SIMDLoadLane<16, 0x55>;
defm "" : SIMDLoadLane<32, 0x56>;
defm "" : SIMDLoadLane<64, 0x57>;

// Select loads, possibly including a constant offset.
multiclass LoadLanePat<Vec vec, SDPatternOperator kind> {
  defvar load_lane_a32 = !cast<NI>("LOAD_LANE_"#vec.lane_bits#"_A32");
  defvar load_lane_a64 = !cast<NI>("LOAD_LANE_"#vec.lane_bits#"_A64");
```
- **EN**: Adds declarative TableGen records such as `LoadLanePat` that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `LoadLanePat`，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 365-375

```tablegen
  def : Pat<(vec.vt (kind (AddrOps32 offset32_op:$offset, I32:$addr),
              (vec.vt V128:$vec), (i32 vec.lane_idx:$idx))),
            (load_lane_a32 0, $offset, imm:$idx, $addr, $vec)>,
        Requires<[HasAddr32]>;

  def : Pat<(vec.vt (kind (AddrOps64 offset64_op:$offset, I64:$addr),
              (vec.vt V128:$vec), (i32 vec.lane_idx:$idx))),
            (load_lane_a64 0, $offset, imm:$idx, $addr, $vec)>,
        Requires<[HasAddr64]>;
}
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 376-386

```tablegen
def load8_lane :
  PatFrag<(ops node:$ptr, node:$vec, node:$idx),
          (vector_insert $vec, (i32 (extloadi8 $ptr)), $idx)>;
def load16_lane :
  PatFrag<(ops node:$ptr, node:$vec, node:$idx),
          (vector_insert $vec, (i32 (extloadi16 $ptr)), $idx)>;
def load32_lane :
  PatFrags<(ops node:$ptr, node:$vec, node:$idx), [
           (vector_insert $vec, (i32 (load $ptr)), $idx),
           (vector_insert $vec, (f32 (load $ptr)), $idx)
]>;
```
- **EN**: Adds declarative TableGen records such as `load8_lane`, `load16_lane`, `load32_lane` that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `load8_lane`, `load16_lane`, `load32_lane`，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 387-393

```tablegen
def load64_lane :
  PatFrags<(ops node:$ptr, node:$vec, node:$idx), [
           (vector_insert $vec, (i64 (load $ptr)), $idx),
           (vector_insert $vec, (f64 (load $ptr)), $idx)
]>;

defm : LoadLanePat<I8x16, load8_lane>;
```
- **EN**: Adds declarative TableGen records such as `load64_lane` that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `load64_lane`，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 394-400

```tablegen
defm : LoadLanePat<I16x8, load16_lane>;
defm : LoadLanePat<I32x4, load32_lane>;
defm : LoadLanePat<I64x2, load64_lane>;
defm : LoadLanePat<F32x4, load32_lane>;
defm : LoadLanePat<F64x2, load64_lane>;

// Store: v128.store
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 401-414

```tablegen
let mayStore = 1, UseNamedOperandTable = 1 in {
defm STORE_V128_A32 :
  SIMD_I<(outs), (ins P2Align:$p2align, offset32_op:$off, I32:$addr, V128:$vec),
         (outs), (ins P2Align:$p2align, offset32_op:$off), [],
         "v128.store\t${off}(${addr})$p2align, $vec",
         "v128.store\t$off$p2align", 11>;
defm STORE_V128_A64 :
  SIMD_I<(outs), (ins P2Align:$p2align, offset64_op:$off, I64:$addr, V128:$vec),
         (outs), (ins P2Align:$p2align, offset64_op:$off), [],
         "v128.store\t${off}(${addr})$p2align, $vec",
         "v128.store\t$off$p2align", 11>;
}

// Def store patterns from WebAssemblyInstrMemory.td for vector types
```
- **EN**: Adds declarative TableGen records such as `STORE_V128_A32`, `STORE_V128_A64` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `STORE_V128_A32`, `STORE_V128_A64`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 415-421

```tablegen
foreach vec = AllVecs in {
defm : StorePat<vec.vt, store, "STORE_V128">;
}

// Store lane
multiclass SIMDStoreLane<Vec vec, bits<32> simdop> {
  defvar name = "v128.store"#vec.lane_bits#"_lane";
```
- **EN**: Adds declarative TableGen records such as `SIMDStoreLane` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `SIMDStoreLane`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 422-429

```tablegen
  let mayStore = 1, UseNamedOperandTable = 1 in {
  defm STORE_LANE_#vec#_A32 :
    SIMD_I<(outs),
           (ins P2Align:$p2align, offset32_op:$off, vec_i8imm_op:$idx,
                I32:$addr, V128:$vec),
           (outs), (ins P2Align:$p2align, offset32_op:$off, vec_i8imm_op:$idx),
           [], name#"\t${off}(${addr})$p2align, $vec, $idx",
           name#"\t$off$p2align, $idx", simdop>;
```
- **EN**: Continues the WebAssembly backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 WebAssembly 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 430-439

```tablegen
  defm STORE_LANE_#vec#_A64 :
    SIMD_I<(outs),
           (ins P2Align:$p2align, offset64_op:$off, vec_i8imm_op:$idx,
                I64:$addr, V128:$vec),
           (outs), (ins P2Align:$p2align, offset64_op:$off, vec_i8imm_op:$idx),
           [], name#"\t${off}(${addr})$p2align, $vec, $idx",
           name#"\t$off$p2align, $idx", simdop>;
  } // mayStore = 1, UseNamedOperandTable = 1
}
```
- **EN**: Continues the WebAssembly backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 WebAssembly 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 440-450

```tablegen
defm "" : SIMDStoreLane<I8x16, 0x58>;
defm "" : SIMDStoreLane<I16x8, 0x59>;
defm "" : SIMDStoreLane<I32x4, 0x5a>;
defm "" : SIMDStoreLane<I64x2, 0x5b>;

multiclass StoreLanePat<Vec vec, SDPatternOperator kind> {
  def : Pat<(kind (AddrOps32 offset32_op:$offset, I32:$addr),
                  (vec.vt V128:$vec),
                  (i32 vec.lane_idx:$idx)),
            (!cast<NI>("STORE_LANE_"#vec#"_A32") 0, $offset, imm:$idx, $addr, $vec)>,
        Requires<[HasAddr32]>;
```
- **EN**: Adds declarative TableGen records such as `StoreLanePat` that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `StoreLanePat`，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 451-457

```tablegen
  def : Pat<(kind (AddrOps64 offset64_op:$offset, I64:$addr),
                  (vec.vt V128:$vec),
                  (i32 vec.lane_idx:$idx)),
            (!cast<NI>("STORE_LANE_"#vec#"_A64") 0, $offset, imm:$idx, $addr, $vec)>,
        Requires<[HasAddr64]>;
}
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 458-466

```tablegen
def store8_lane :
  PatFrag<(ops node:$ptr, node:$vec, node:$idx),
          (truncstorei8 (i32 (vector_extract $vec, $idx)), $ptr)>;
def store16_lane :
  PatFrag<(ops node:$ptr, node:$vec, node:$idx),
          (truncstorei16 (i32 (vector_extract $vec, $idx)), $ptr)>;
def store32_lane :
  PatFrag<(ops node:$ptr, node:$vec, node:$idx),
          (store (i32 (vector_extract $vec, $idx)), $ptr)>;
```
- **EN**: Adds declarative TableGen records such as `store8_lane`, `store16_lane`, `store32_lane` that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `store8_lane`, `store16_lane`, `store32_lane`，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 467-473

```tablegen
def store64_lane :
  PatFrag<(ops node:$ptr, node:$vec, node:$idx),
          (store (i64 (vector_extract $vec, $idx)), $ptr)>;
// TODO: floating point lanes as well

let AddedComplexity = 1 in {
defm : StoreLanePat<I8x16, store8_lane>;
```
- **EN**: Adds declarative TableGen records such as `store64_lane` that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `store64_lane`，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 474-478

```tablegen
defm : StoreLanePat<I16x8, store16_lane>;
defm : StoreLanePat<I32x4, store32_lane>;
defm : StoreLanePat<I64x2, store64_lane>;
}
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 479-480

```tablegen
//===----------------------------------------------------------------------===//
// Constructing SIMD values
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Constructing SIMD values".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Constructing SIMD values”。

### Lines 481-491

```tablegen
//===----------------------------------------------------------------------===//

// Constant: v128.const
multiclass ConstVec<Vec vec, dag ops, dag pat, string args> {
  let isMoveImm = 1, isReMaterializable = 1 in
  defm CONST_V128_#vec : SIMD_I<(outs V128:$dst), ops, (outs), ops,
                                 [(set V128:$dst, (vec.vt pat))],
                                 "v128.const\t$dst, "#args,
                                 "v128.const\t"#args, 12>;
}
```
- **EN**: Adds declarative TableGen records such as `ConstVec` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `ConstVec`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 492-506

```tablegen
defm "" : ConstVec<I8x16,
                   (ins vec_i8imm_op:$i0, vec_i8imm_op:$i1,
                        vec_i8imm_op:$i2, vec_i8imm_op:$i3,
                        vec_i8imm_op:$i4, vec_i8imm_op:$i5,
                        vec_i8imm_op:$i6, vec_i8imm_op:$i7,
                        vec_i8imm_op:$i8, vec_i8imm_op:$i9,
                        vec_i8imm_op:$iA, vec_i8imm_op:$iB,
                        vec_i8imm_op:$iC, vec_i8imm_op:$iD,
                        vec_i8imm_op:$iE, vec_i8imm_op:$iF),
                   (build_vector ImmI8:$i0, ImmI8:$i1, ImmI8:$i2, ImmI8:$i3,
                                 ImmI8:$i4, ImmI8:$i5, ImmI8:$i6, ImmI8:$i7,
                                 ImmI8:$i8, ImmI8:$i9, ImmI8:$iA, ImmI8:$iB,
                                 ImmI8:$iC, ImmI8:$iD, ImmI8:$iE, ImmI8:$iF),
                   !strconcat("$i0, $i1, $i2, $i3, $i4, $i5, $i6, $i7, ",
                              "$i8, $i9, $iA, $iB, $iC, $iD, $iE, $iF")>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 507-515

```tablegen
defm "" : ConstVec<I16x8,
                   (ins vec_i16imm_op:$i0, vec_i16imm_op:$i1,
                        vec_i16imm_op:$i2, vec_i16imm_op:$i3,
                        vec_i16imm_op:$i4, vec_i16imm_op:$i5,
                        vec_i16imm_op:$i6, vec_i16imm_op:$i7),
                   (build_vector
                     ImmI16:$i0, ImmI16:$i1, ImmI16:$i2, ImmI16:$i3,
                     ImmI16:$i4, ImmI16:$i5, ImmI16:$i6, ImmI16:$i7),
                   "$i0, $i1, $i2, $i3, $i4, $i5, $i6, $i7">;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 516-522

```tablegen
let IsCanonical = 1 in
defm "" : ConstVec<I32x4,
                   (ins vec_i32imm_op:$i0, vec_i32imm_op:$i1,
                        vec_i32imm_op:$i2, vec_i32imm_op:$i3),
                   (build_vector (i32 imm:$i0), (i32 imm:$i1),
                                 (i32 imm:$i2), (i32 imm:$i3)),
                   "$i0, $i1, $i2, $i3">;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 523-532

```tablegen
defm "" : ConstVec<I64x2,
                   (ins vec_i64imm_op:$i0, vec_i64imm_op:$i1),
                   (build_vector (i64 imm:$i0), (i64 imm:$i1)),
                   "$i0, $i1">;
defm "" : ConstVec<F32x4,
                   (ins f32imm_op:$i0, f32imm_op:$i1,
                        f32imm_op:$i2, f32imm_op:$i3),
                   (build_vector (f32 fpimm:$i0), (f32 fpimm:$i1),
                                 (f32 fpimm:$i2), (f32 fpimm:$i3)),
                   "$i0, $i1, $i2, $i3">;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 533-542

```tablegen
defm "" : ConstVec<F64x2,
                  (ins f64imm_op:$i0, f64imm_op:$i1),
                  (build_vector (f64 fpimm:$i0), (f64 fpimm:$i1)),
                  "$i0, $i1">;

// Match splat(x) -> const.v128(x, ..., x)
foreach vec = StdVecs in {
  defvar numEls = !div(vec.vt.Size, vec.lane_bits);
  defvar isFloat = !or(!eq(vec.lane_vt, f32), !eq(vec.lane_vt, f64));
  defvar immKind = !if(isFloat, fpimm, imm);
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 543-549

```tablegen
  def : Pat<(vec.splat (vec.lane_vt immKind:$x)),
            !dag(!cast<NI>("CONST_V128_"#vec),
                 !listsplat((vec.lane_vt immKind:$x), numEls),
                 ?)>;
}

// Shuffle lanes: shuffle
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 550-580

```tablegen
defm SHUFFLE :
  SIMD_I<(outs V128:$dst),
         (ins V128:$x, V128:$y,
           vec_i8imm_op:$m0, vec_i8imm_op:$m1,
           vec_i8imm_op:$m2, vec_i8imm_op:$m3,
           vec_i8imm_op:$m4, vec_i8imm_op:$m5,
           vec_i8imm_op:$m6, vec_i8imm_op:$m7,
           vec_i8imm_op:$m8, vec_i8imm_op:$m9,
           vec_i8imm_op:$mA, vec_i8imm_op:$mB,
           vec_i8imm_op:$mC, vec_i8imm_op:$mD,
           vec_i8imm_op:$mE, vec_i8imm_op:$mF),
         (outs),
         (ins
           vec_i8imm_op:$m0, vec_i8imm_op:$m1,
           vec_i8imm_op:$m2, vec_i8imm_op:$m3,
           vec_i8imm_op:$m4, vec_i8imm_op:$m5,
           vec_i8imm_op:$m6, vec_i8imm_op:$m7,
           vec_i8imm_op:$m8, vec_i8imm_op:$m9,
           vec_i8imm_op:$mA, vec_i8imm_op:$mB,
           vec_i8imm_op:$mC, vec_i8imm_op:$mD,
           vec_i8imm_op:$mE, vec_i8imm_op:$mF),
         [],
         "i8x16.shuffle\t$dst, $x, $y, "#
           "$m0, $m1, $m2, $m3, $m4, $m5, $m6, $m7, "#
           "$m8, $m9, $mA, $mB, $mC, $mD, $mE, $mF",
         "i8x16.shuffle\t"#
           "$m0, $m1, $m2, $m3, $m4, $m5, $m6, $m7, "#
           "$m8, $m9, $mA, $mB, $mC, $mD, $mE, $mF",
         13>;

// Shuffles after custom lowering
```
- **EN**: Adds declarative TableGen records such as `SHUFFLE` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `SHUFFLE`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 581-599

```tablegen
def wasm_shuffle_t : SDTypeProfile<1, 18, []>;
def wasm_shuffle : SDNode<"WebAssemblyISD::SHUFFLE", wasm_shuffle_t>;
foreach vec = AllVecs in {
// The @llvm.wasm.shuffle intrinsic has immediate arguments that become TargetConstants.
def : Pat<(vec.vt (wasm_shuffle (vec.vt V128:$x), (vec.vt V128:$y),
            (i32 timm:$m0), (i32 timm:$m1),
            (i32 timm:$m2), (i32 timm:$m3),
            (i32 timm:$m4), (i32 timm:$m5),
            (i32 timm:$m6), (i32 timm:$m7),
            (i32 timm:$m8), (i32 timm:$m9),
            (i32 timm:$mA), (i32 timm:$mB),
            (i32 timm:$mC), (i32 timm:$mD),
            (i32 timm:$mE), (i32 timm:$mF))),
          (SHUFFLE $x, $y,
            imm:$m0, imm:$m1, imm:$m2, imm:$m3,
            imm:$m4, imm:$m5, imm:$m6, imm:$m7,
            imm:$m8, imm:$m9, imm:$mA, imm:$mB,
            imm:$mC, imm:$mD, imm:$mE, imm:$mF)>;
// Normal shufflevector instructions may have normal constant arguemnts.
```
- **EN**: Adds declarative TableGen records such as `wasm_shuffle_t`, `wasm_shuffle` that LLVM later expands into generated tables or helper code. This range works in the SelectionDAG-based lowering pipeline.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `wasm_shuffle_t`, `wasm_shuffle`，LLVM 随后会把它们展开成生成表或辅助代码。 这一段工作在基于 SelectionDAG 的 lowering 流水线中。

### Lines 600-616

```tablegen
def : Pat<(vec.vt (wasm_shuffle (vec.vt V128:$x), (vec.vt V128:$y),
            (i32 LaneIdx32:$m0), (i32 LaneIdx32:$m1),
            (i32 LaneIdx32:$m2), (i32 LaneIdx32:$m3),
            (i32 LaneIdx32:$m4), (i32 LaneIdx32:$m5),
            (i32 LaneIdx32:$m6), (i32 LaneIdx32:$m7),
            (i32 LaneIdx32:$m8), (i32 LaneIdx32:$m9),
            (i32 LaneIdx32:$mA), (i32 LaneIdx32:$mB),
            (i32 LaneIdx32:$mC), (i32 LaneIdx32:$mD),
            (i32 LaneIdx32:$mE), (i32 LaneIdx32:$mF))),
          (SHUFFLE $x, $y,
            imm:$m0, imm:$m1, imm:$m2, imm:$m3,
            imm:$m4, imm:$m5, imm:$m6, imm:$m7,
            imm:$m8, imm:$m9, imm:$mA, imm:$mB,
            imm:$mC, imm:$mD, imm:$mE, imm:$mF)>;
}

// Swizzle lanes: i8x16.swizzle
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 617-624

```tablegen
def wasm_swizzle_t : SDTypeProfile<1, 2, []>;
def wasm_swizzle : SDNode<"WebAssemblyISD::SWIZZLE", wasm_swizzle_t>;
defm SWIZZLE :
  SIMD_I<(outs V128:$dst), (ins V128:$src, V128:$mask), (outs), (ins),
         [(set (v16i8 V128:$dst),
           (wasm_swizzle (v16i8 V128:$src), (v16i8 V128:$mask)))],
         "i8x16.swizzle\t$dst, $src, $mask", "i8x16.swizzle", 14>;
```
- **EN**: Adds declarative TableGen records such as `wasm_swizzle_t`, `wasm_swizzle`, `SWIZZLE` that LLVM later expands into generated tables or helper code. This range works in the SelectionDAG-based lowering pipeline.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `wasm_swizzle_t`, `wasm_swizzle`, `SWIZZLE`，LLVM 随后会把它们展开成生成表或辅助代码。 这一段工作在基于 SelectionDAG 的 lowering 流水线中。

### Lines 625-636

```tablegen
def : Pat<(int_wasm_swizzle (v16i8 V128:$src), (v16i8 V128:$mask)),
          (SWIZZLE $src, $mask)>;

multiclass Splat<Vec vec, bits<32> simdop> {
  defm SPLAT_#vec : SIMD_I<(outs V128:$dst), (ins vec.lane_rc:$x),
                           (outs), (ins),
                           [(set (vec.vt V128:$dst),
                              (vec.splat vec.lane_rc:$x))],
                           vec.prefix#".splat\t$dst, $x", vec.prefix#".splat",
                           simdop>;
}
```
- **EN**: Adds declarative TableGen records such as `Splat` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `Splat`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 637-645

```tablegen
defm "" : Splat<I8x16, 15>;
defm "" : Splat<I16x8, 16>;
defm "" : Splat<I32x4, 17>;
defm "" : Splat<I64x2, 18>;
defm "" : Splat<F32x4, 19>;
defm "" : Splat<F64x2, 20>;

// Half values are not fully supported so an intrinsic is used instead of a
// regular Splat pattern as above.
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 646-652

```tablegen
defm SPLAT_F16x8 :
  HALF_PRECISION_I<(outs V128:$dst), (ins F32:$x),
                   (outs), (ins),
                   [(set (v8f16 V128:$dst), (int_wasm_splat_f16x8 F32:$x))],
                   "f16x8.splat\t$dst, $x", "f16x8.splat", 0x120>;

// scalar_to_vector leaves high lanes undefined, so can be a splat
```
- **EN**: Adds declarative TableGen records such as `SPLAT_F16x8` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `SPLAT_F16x8`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 653-656

```tablegen
foreach vec = StdVecs in
def : Pat<(vec.vt (scalar_to_vector (vec.lane_vt vec.lane_rc:$x))),
          (!cast<Instruction>("SPLAT_"#vec) $x)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 657-658

```tablegen
//===----------------------------------------------------------------------===//
// Accessing lanes
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Accessing lanes".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Accessing lanes”。

### Lines 659-669

```tablegen
//===----------------------------------------------------------------------===//

// Extract lane as a scalar: extract_lane / extract_lane_s / extract_lane_u
multiclass ExtractLane<Vec vec, bits<32> simdop, string suffix = ""> {
  defm EXTRACT_LANE_#vec#suffix :
      SIMD_I<(outs vec.lane_rc:$dst), (ins V128:$vec, vec_i8imm_op:$idx),
             (outs), (ins vec_i8imm_op:$idx), [],
             vec.prefix#".extract_lane"#suffix#"\t$dst, $vec, $idx",
             vec.prefix#".extract_lane"#suffix#"\t$idx", simdop>;
}
```
- **EN**: Adds declarative TableGen records such as `ExtractLane` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `ExtractLane`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 670-676

```tablegen
defm "" : ExtractLane<I8x16, 21, "_s">;
defm "" : ExtractLane<I8x16, 22, "_u">;
defm "" : ExtractLane<I16x8, 24, "_s">;
defm "" : ExtractLane<I16x8, 25, "_u">;
defm "" : ExtractLane<I32x4, 27>;
defm "" : ExtractLane<I64x2, 29>;
defm "" : ExtractLane<F32x4, 31>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 677-684

```tablegen
defm "" : ExtractLane<F64x2, 33>;

def : Pat<(vector_extract (v16i8 V128:$vec), (i32 LaneIdx16:$idx)),
          (EXTRACT_LANE_I8x16_u $vec, imm:$idx)>;
def : Pat<(vector_extract (v8i16 V128:$vec), (i32 LaneIdx8:$idx)),
          (EXTRACT_LANE_I16x8_u $vec, imm:$idx)>;
def : Pat<(vector_extract (v4i32 V128:$vec), (i32 LaneIdx4:$idx)),
          (EXTRACT_LANE_I32x4 $vec, imm:$idx)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 685-691

```tablegen
def : Pat<(vector_extract (v4f32 V128:$vec), (i32 LaneIdx4:$idx)),
          (EXTRACT_LANE_F32x4 $vec, imm:$idx)>;
def : Pat<(vector_extract (v2i64 V128:$vec), (i32 LaneIdx2:$idx)),
          (EXTRACT_LANE_I64x2 $vec, imm:$idx)>;
def : Pat<(vector_extract (v2f64 V128:$vec), (i32 LaneIdx2:$idx)),
          (EXTRACT_LANE_F64x2 $vec, imm:$idx)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 692-700

```tablegen
def : Pat<
  (sext_inreg (vector_extract (v16i8 V128:$vec), (i32 LaneIdx16:$idx)), i8),
  (EXTRACT_LANE_I8x16_s $vec, imm:$idx)>;
def : Pat<
  (and (vector_extract (v16i8 V128:$vec), (i32 LaneIdx16:$idx)), (i32 0xff)),
  (EXTRACT_LANE_I8x16_u $vec, imm:$idx)>;
def : Pat<
  (sext_inreg (vector_extract (v8i16 V128:$vec), (i32 LaneIdx8:$idx)), i16),
  (EXTRACT_LANE_I16x8_s $vec, imm:$idx)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 701-713

```tablegen
def : Pat<
  (and (vector_extract (v8i16 V128:$vec), (i32 LaneIdx8:$idx)), (i32 0xffff)),
  (EXTRACT_LANE_I16x8_u $vec, imm:$idx)>;

defm EXTRACT_LANE_F16x8 :
  HALF_PRECISION_I<(outs F32:$dst), (ins V128:$vec, vec_i8imm_op:$idx),
                   (outs), (ins vec_i8imm_op:$idx),
                   [(set (f32 F32:$dst), (int_wasm_extract_lane_f16x8
                    (v8f16 V128:$vec), (i32 LaneIdx8:$idx)))],
                   "f16x8.extract_lane\t$dst, $vec, $idx",
                   "f16x8.extract_lane\t$idx", 0x121>;

// Replace lane value: replace_lane
```
- **EN**: Adds declarative TableGen records such as `EXTRACT_LANE_F16x8` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `EXTRACT_LANE_F16x8`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 714-725

```tablegen
multiclass ReplaceLane<Vec vec, bits<32> simdop> {
  defm REPLACE_LANE_#vec :
    SIMD_I<(outs V128:$dst), (ins V128:$vec, vec_i8imm_op:$idx, vec.lane_rc:$x),
           (outs), (ins vec_i8imm_op:$idx),
           [(set V128:$dst, (vector_insert
             (vec.vt V128:$vec),
             (vec.lane_vt vec.lane_rc:$x),
             (i32 vec.lane_idx:$idx)))],
           vec.prefix#".replace_lane\t$dst, $vec, $idx, $x",
           vec.prefix#".replace_lane\t$idx", simdop>;
}
```
- **EN**: Adds declarative TableGen records such as `ReplaceLane` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `ReplaceLane`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 726-735

```tablegen
defm "" : ReplaceLane<I8x16, 23>;
defm "" : ReplaceLane<I16x8, 26>;
defm "" : ReplaceLane<I32x4, 28>;
defm "" : ReplaceLane<I64x2, 30>;
defm "" : ReplaceLane<F32x4, 32>;
defm "" : ReplaceLane<F64x2, 34>;

// For now use an intrinsic for f16x8.replace_lane instead of ReplaceLane above
// since LLVM IR generated with half type arguments is not well supported and
// creates conversions from f16->f32.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "For now use an intrinsic for f16x8.replace_lane instead of ReplaceLane above".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“For now use an intrinsic for f16x8.replace_lane instead of ReplaceLane above”。

### Lines 736-746

```tablegen
defm REPLACE_LANE_F16x8 :
  HALF_PRECISION_I<(outs V128:$dst), (ins V128:$vec, vec_i8imm_op:$idx, F32:$x),
                   (outs), (ins vec_i8imm_op:$idx),
                   [(set (v8f16 V128:$dst), (int_wasm_replace_lane_f16x8
                     (v8f16 V128:$vec),
                     (i32 LaneIdx8:$idx),
                     (f32 F32:$x)))],
                   "f16x8.replace_lane\t$dst, $vec, $idx, $x",
                   "f16x8.replace_lane\t$idx", 0x122>;

// Lower undef lane indices to zero
```
- **EN**: Adds declarative TableGen records such as `REPLACE_LANE_F16x8` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `REPLACE_LANE_F16x8`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 747-754

```tablegen
def : Pat<(vector_insert (v16i8 V128:$vec), I32:$x, undef),
          (REPLACE_LANE_I8x16 $vec, 0, $x)>;
def : Pat<(vector_insert (v8i16 V128:$vec), I32:$x, undef),
          (REPLACE_LANE_I16x8 $vec, 0, $x)>;
def : Pat<(vector_insert (v4i32 V128:$vec), I32:$x, undef),
          (REPLACE_LANE_I32x4 $vec, 0, $x)>;
def : Pat<(vector_insert (v2i64 V128:$vec), I64:$x, undef),
          (REPLACE_LANE_I64x2 $vec, 0, $x)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 755-759

```tablegen
def : Pat<(vector_insert (v4f32 V128:$vec), F32:$x, undef),
          (REPLACE_LANE_F32x4 $vec, 0, $x)>;
def : Pat<(vector_insert (v2f64 V128:$vec), F64:$x, undef),
          (REPLACE_LANE_F64x2 $vec, 0, $x)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 760-761

```tablegen
//===----------------------------------------------------------------------===//
// Comparisons
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Comparisons".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Comparisons”。

### Lines 762-773

```tablegen
//===----------------------------------------------------------------------===//

multiclass SIMDCondition<Vec vec, string name, CondCode cond, bits<32> simdop,
                         list<Predicate> reqs = []> {
  defm _#vec :
    SIMD_I<(outs V128:$dst), (ins V128:$lhs, V128:$rhs), (outs), (ins),
           [(set (vec.int_vt V128:$dst),
             (setcc (vec.vt V128:$lhs), (vec.vt V128:$rhs), cond))],
           vec.prefix#"."#name#"\t$dst, $lhs, $rhs",
           vec.prefix#"."#name, simdop, reqs>;
}
```
- **EN**: Adds declarative TableGen records such as `SIMDCondition` that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `SIMDCondition`，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 774-780

```tablegen
multiclass HalfPrecisionCondition<Vec vec, string name, CondCode cond,
                                  bits<32> simdop> {
  defm "" : SIMDCondition<vec, name, cond, simdop, [HasFP16]>;
}

multiclass SIMDConditionInt<string name, CondCode cond, bits<32> baseInst> {
  defm "" : SIMDCondition<I8x16, name, cond, baseInst>;
```
- **EN**: Adds declarative TableGen records such as `HalfPrecisionCondition`, `SIMDConditionInt` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `HalfPrecisionCondition`, `SIMDConditionInt`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 781-787

```tablegen
  defm "" : SIMDCondition<I16x8, name, cond, !add(baseInst, 10)>;
  defm "" : SIMDCondition<I32x4, name, cond, !add(baseInst, 20)>;
}

multiclass SIMDConditionFP<string name, CondCode cond, bits<32> baseInst> {
  defm "" : SIMDCondition<F32x4, name, cond, baseInst>;
  defm "" : SIMDCondition<F64x2, name, cond, !add(baseInst, 6)>;
```
- **EN**: Adds declarative TableGen records such as `SIMDConditionFP` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `SIMDConditionFP`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 788-794

```tablegen
  defm "" : HalfPrecisionCondition<F16x8, name, cond, !add(baseInst, 246)>;
}

// Equality: eq
let isCommutable = 1 in {
defm EQ : SIMDConditionInt<"eq", SETEQ, 35>;
defm EQ : SIMDCondition<I64x2, "eq", SETEQ, 214>;
```
- **EN**: Adds declarative TableGen records such as `EQ` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `EQ`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 795-801

```tablegen
defm EQ : SIMDConditionFP<"eq", SETOEQ, 65>;
} // isCommutable = 1

// Non-equality: ne
let isCommutable = 1 in {
defm NE : SIMDConditionInt<"ne", SETNE, 36>;
defm NE : SIMDCondition<I64x2, "ne", SETNE, 215>;
```
- **EN**: Adds declarative TableGen records such as `EQ`, `NE` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `EQ`, `NE`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 802-808

```tablegen
defm NE : SIMDConditionFP<"ne", SETUNE, 66>;
} // isCommutable = 1

// Less than: lt_s / lt_u / lt
defm LT_S : SIMDConditionInt<"lt_s", SETLT, 37>;
defm LT_S : SIMDCondition<I64x2, "lt_s", SETLT, 216>;
defm LT_U : SIMDConditionInt<"lt_u", SETULT, 38>;
```
- **EN**: Adds declarative TableGen records such as `NE`, `LT_S`, `LT_U` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `NE`, `LT_S`, `LT_U`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 809-817

```tablegen
defm LT : SIMDConditionFP<"lt", SETOLT, 67>;

// Greater than: gt_s / gt_u / gt
defm GT_S : SIMDConditionInt<"gt_s", SETGT, 39>;
defm GT_S : SIMDCondition<I64x2, "gt_s", SETGT, 217>;
defm GT_U : SIMDConditionInt<"gt_u", SETUGT, 40>;
defm GT : SIMDConditionFP<"gt", SETOGT, 68>;

// Less than or equal: le_s / le_u / le
```
- **EN**: Adds declarative TableGen records such as `LT`, `GT_S`, `GT_U` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `LT`, `GT_S`, `GT_U`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 818-824

```tablegen
defm LE_S : SIMDConditionInt<"le_s", SETLE, 41>;
defm LE_S : SIMDCondition<I64x2, "le_s", SETLE, 218>;
defm LE_U : SIMDConditionInt<"le_u", SETULE, 42>;
defm LE : SIMDConditionFP<"le", SETOLE, 69>;

// Greater than or equal: ge_s / ge_u / ge
defm GE_S : SIMDConditionInt<"ge_s", SETGE, 43>;
```
- **EN**: Adds declarative TableGen records such as `LE_S`, `LE_U`, `LE` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `LE_S`, `LE_U`, `LE`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 825-831

```tablegen
defm GE_S : SIMDCondition<I64x2, "ge_s", SETGE, 219>;
defm GE_U : SIMDConditionInt<"ge_u", SETUGE, 44>;
defm GE : SIMDConditionFP<"ge", SETOGE, 70>;

// Lower float comparisons that don't care about NaN to standard WebAssembly
// float comparisons. These instructions are generated with nnan and in the
// target-independent expansion of unordered comparisons and ordered ne.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Lower float comparisons that don't care about NaN to standard WebAssembly". Notable symbols in this range include `GE_S`, `GE_U`, `GE`.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Lower float comparisons that don't care about NaN to standard WebAssembly”。 该区间中较显眼的符号包括 `GE_S`, `GE_U`, `GE`。

### Lines 832-838

```tablegen
foreach nodes = [[seteq, EQ_F32x4], [setne, NE_F32x4], [setlt, LT_F32x4],
                 [setgt, GT_F32x4], [setle, LE_F32x4], [setge, GE_F32x4]] in
def : Pat<(v4i32 (nodes[0] (v4f32 V128:$lhs), (v4f32 V128:$rhs))),
          (nodes[1] $lhs, $rhs)>;

foreach nodes = [[seteq, EQ_F64x2], [setne, NE_F64x2], [setlt, LT_F64x2],
                 [setgt, GT_F64x2], [setle, LE_F64x2], [setge, GE_F64x2]] in
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 839-841

```tablegen
def : Pat<(v2i64 (nodes[0] (v2f64 V128:$lhs), (v2f64 V128:$rhs))),
          (nodes[1] $lhs, $rhs)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 842-843

```tablegen
//===----------------------------------------------------------------------===//
// Bitwise operations
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Bitwise operations".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Bitwise operations”。

### Lines 844-855

```tablegen
//===----------------------------------------------------------------------===//

multiclass SIMDBinary<Vec vec, SDPatternOperator node, string name,
                      bits<32> simdop, list<Predicate> reqs = []> {
  defm _#vec : SIMD_I<(outs V128:$dst), (ins V128:$lhs, V128:$rhs),
                      (outs), (ins),
                      [(set (vec.vt V128:$dst),
                        (node (vec.vt V128:$lhs), (vec.vt V128:$rhs)))],
                      vec.prefix#"."#name#"\t$dst, $lhs, $rhs",
                      vec.prefix#"."#name, simdop, reqs>;
}
```
- **EN**: Adds declarative TableGen records such as `SIMDBinary` that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `SIMDBinary`，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 856-862

```tablegen
multiclass HalfPrecisionBinary<Vec vec, SDPatternOperator node, string name,
                               bits<32> simdop> {
  defm "" : SIMDBinary<vec, node, name, simdop, [HasFP16]>;
}

multiclass SIMDBitwise<SDPatternOperator node, string name, bits<32> simdop,
                       bit commutable = false> {
```
- **EN**: Adds declarative TableGen records such as `HalfPrecisionBinary`, `SIMDBitwise` that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `HalfPrecisionBinary`, `SIMDBitwise`，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 863-871

```tablegen
  let isCommutable = commutable in
  defm "" : SIMD_I<(outs V128:$dst), (ins V128:$lhs, V128:$rhs),
                   (outs), (ins), [],
                   "v128."#name#"\t$dst, $lhs, $rhs", "v128."#name, simdop>;
  foreach vec = IntVecs in
  def : Pat<(node (vec.vt V128:$lhs), (vec.vt V128:$rhs)),
            (!cast<NI>(NAME) $lhs, $rhs)>;
}
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 872-880

```tablegen
multiclass SIMDUnary<Vec vec, SDPatternOperator node, string name,
                     bits<32> simdop, list<Predicate> reqs = []> {
  defm _#vec : SIMD_I<(outs V128:$dst), (ins V128:$v), (outs), (ins),
                      [(set (vec.vt V128:$dst),
                        (vec.vt (node (vec.vt V128:$v))))],
                      vec.prefix#"."#name#"\t$dst, $v",
                      vec.prefix#"."#name, simdop, reqs>;
}
```
- **EN**: Adds declarative TableGen records such as `SIMDUnary` that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `SIMDUnary`，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 881-888

```tablegen
multiclass HalfPrecisionUnary<Vec vec, SDPatternOperator node, string name,
                              bits<32> simdop> {
  defm "" : SIMDUnary<vec, node, name, simdop, [HasFP16]>;
}

// Bitwise logic: v128.not
defm NOT : SIMD_I<(outs V128:$dst), (ins V128:$v), (outs), (ins), [],
                  "v128.not\t$dst, $v", "v128.not", 77>;
```
- **EN**: Adds declarative TableGen records such as `HalfPrecisionUnary`, `NOT` that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `HalfPrecisionUnary`, `NOT`，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 889-897

```tablegen
foreach vec = IntVecs in
def : Pat<(vnot (vec.vt V128:$v)), (NOT $v)>;

// Bitwise logic: v128.and / v128.or / v128.xor
defm AND : SIMDBitwise<and, "and", 78, true>;
defm OR : SIMDBitwise<or, "or", 80, true>;
defm XOR : SIMDBitwise<xor, "xor", 81, true>;

// Bitwise logic: v128.andnot
```
- **EN**: Adds declarative TableGen records such as `AND`, `OR`, `XOR` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `AND`, `OR`, `XOR`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 898-905

```tablegen
def andnot : PatFrag<(ops node:$left, node:$right), (and $left, (vnot $right))>;
defm ANDNOT : SIMDBitwise<andnot, "andnot", 79>;

// Bitwise select: v128.bitselect
defm BITSELECT :
  SIMD_I<(outs V128:$dst), (ins V128:$v1, V128:$v2, V128:$c), (outs), (ins), [],
         "v128.bitselect\t$dst, $v1, $v2, $c", "v128.bitselect", 82>;
```
- **EN**: Adds declarative TableGen records such as `andnot`, `ANDNOT`, `BITSELECT` that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `andnot`, `ANDNOT`, `BITSELECT`，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 906-912

```tablegen
foreach vec = StdVecs in
def : Pat<(vec.vt (int_wasm_bitselect
            (vec.vt V128:$v1), (vec.vt V128:$v2), (vec.vt V128:$c))),
          (BITSELECT $v1, $v2, $c)>;

// Bitselect is equivalent to (c & v1) | (~c & v2)
foreach vec = IntVecs in
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 913-924

```tablegen
def : Pat<(vec.vt (or (and (vec.vt V128:$c), (vec.vt V128:$v1)),
            (and (vnot V128:$c), (vec.vt V128:$v2)))),
          (BITSELECT $v1, $v2, $c)>;

// Bitselect is also equivalent to ((v1 ^ v2) & c) ^ v2
foreach vec = IntVecs in
def : Pat<(vec.vt (xor (and (xor (vec.vt V128:$v1), (vec.vt V128:$v2)),
                            (vec.vt V128:$c)),
                       (vec.vt V128:$v2))),
          (BITSELECT $v1, $v2, $c)>;

// Same pattern with `c` negated so `a` and `b` get swapped.
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 925-934

```tablegen
foreach vec = IntVecs in
def : Pat<(vec.vt (xor (and (xor (vec.vt V128:$v1), (vec.vt V128:$v2)),
                            (vnot (vec.vt V128:$c))),
                       (vec.vt V128:$v2))),
          (BITSELECT $v2, $v1, $c)>;

// vselect(cond, vnot(X), zero) => andnot(cond, X)
// DAGCombiner converts and(vnot(X), sext(cmp)) into vselect(cmp, vnot(X), zero)
// which the generic vselect pattern below lowers to v128.not + v128.bitselect.
// Match it first and emit a single v128.andnot instead.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "vselect(cond, vnot(X), zero) => andnot(cond, X)".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“vselect(cond, vnot(X), zero) => andnot(cond, X)”。

### Lines 935-941

```tablegen
foreach vec = IntVecs in
def : Pat<(vec.vt (vselect
            (vec.int_vt V128:$c), (vnot (vec.vt V128:$x)), immAllZerosV)),
          (ANDNOT $c, $x)>;

// Also implement vselect in terms of bitselect
foreach vec = StdVecs in
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 942-950

```tablegen
def : Pat<(vec.vt (vselect
            (vec.int_vt V128:$c), (vec.vt V128:$v1), (vec.vt V128:$v2))),
          (BITSELECT $v1, $v2, $c)>;

// MVP select on v128 values
defm SELECT_V128 :
  I<(outs V128:$dst), (ins V128:$lhs, V128:$rhs, I32:$cond), (outs), (ins), [],
    "v128.select\t$dst, $lhs, $rhs, $cond", "v128.select", 0x1b>;
```
- **EN**: Adds declarative TableGen records such as `SELECT_V128` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `SELECT_V128`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 951-957

```tablegen
foreach vec = StdVecs in {
def : Pat<(select I32:$cond, (vec.vt V128:$lhs), (vec.vt V128:$rhs)),
          (SELECT_V128 $lhs, $rhs, $cond)>;

// ISD::SELECT requires its operand to conform to getBooleanContents, but
// WebAssembly's select interprets any non-zero value as true, so we can fold
// a setne with 0 into a select.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "ISD::SELECT requires its operand to conform to getBooleanContents, but". This range works in the SelectionDAG-based lowering pipeline.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“ISD::SELECT requires its operand to conform to getBooleanContents, but”。 这一段工作在基于 SelectionDAG 的 lowering 流水线中。

### Lines 958-967

```tablegen
def : Pat<(select
            (i32 (setne I32:$cond, 0)), (vec.vt V128:$lhs), (vec.vt V128:$rhs)),
          (SELECT_V128 $lhs, $rhs, $cond)>;

// And again, this time with seteq instead of setne and the arms reversed.
def : Pat<(select
            (i32 (seteq I32:$cond, 0)), (vec.vt V128:$lhs), (vec.vt V128:$rhs)),
          (SELECT_V128 $rhs, $lhs, $cond)>;
} // foreach vec
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 968-969

```tablegen
//===----------------------------------------------------------------------===//
// Integer unary arithmetic
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Integer unary arithmetic".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Integer unary arithmetic”。

### Lines 970-979

```tablegen
//===----------------------------------------------------------------------===//

multiclass SIMDUnaryInt<SDPatternOperator node, string name, bits<32> baseInst> {
  defm "" : SIMDUnary<I8x16, node, name, baseInst>;
  defm "" : SIMDUnary<I16x8, node, name, !add(baseInst, 32)>;
  defm "" : SIMDUnary<I32x4, node, name, !add(baseInst, 64)>;
  defm "" : SIMDUnary<I64x2, node, name, !add(baseInst, 96)>;
}

// Integer vector negation
```
- **EN**: Adds declarative TableGen records such as `SIMDUnaryInt` that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `SIMDUnaryInt`，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 980-988

```tablegen
def ivneg : PatFrag<(ops node:$in), (sub immAllZerosV, $in)>;

// Integer absolute value: abs
defm ABS : SIMDUnaryInt<abs, "abs", 96>;

// Integer negation: neg
defm NEG : SIMDUnaryInt<ivneg, "neg", 97>;

// Population count: popcnt
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Integer absolute value: abs". These declarations feed generated pattern-matching logic.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Integer absolute value: abs”。 这些声明会进入生成式模式匹配逻辑。

### Lines 989-995

```tablegen
defm POPCNT : SIMDUnary<I8x16, ctpop, "popcnt", 0x62>;

// Any lane true: any_true
defm ANYTRUE : SIMD_I<(outs I32:$dst), (ins V128:$vec), (outs), (ins), [],
                      "v128.any_true\t$dst, $vec", "v128.any_true", 0x53>;

foreach vec = IntVecs in
```
- **EN**: Adds declarative TableGen records such as `POPCNT`, `ANYTRUE` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `POPCNT`, `ANYTRUE`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 996-1006

```tablegen
def : Pat<(int_wasm_anytrue (vec.vt V128:$vec)), (ANYTRUE V128:$vec)>;

// All lanes true: all_true
multiclass SIMDAllTrue<Vec vec, bits<32> simdop> {
  defm ALLTRUE_#vec : SIMD_I<(outs I32:$dst), (ins V128:$vec), (outs), (ins),
                             [(set I32:$dst,
                               (i32 (int_wasm_alltrue (vec.vt V128:$vec))))],
                             vec.prefix#".all_true\t$dst, $vec",
                             vec.prefix#".all_true", simdop>;
}
```
- **EN**: Adds declarative TableGen records such as `SIMDAllTrue` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `SIMDAllTrue`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1007-1013

```tablegen
defm "" : SIMDAllTrue<I8x16, 0x63>;
defm "" : SIMDAllTrue<I16x8, 0x83>;
defm "" : SIMDAllTrue<I32x4, 0xa3>;
defm "" : SIMDAllTrue<I64x2, 0xc3>;

// Reductions already return 0 or 1, so and 1, setne 0, and seteq 1
// can be folded out
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Reductions already return 0 or 1, so and 1, setne 0, and seteq 1".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Reductions already return 0 or 1, so and 1, setne 0, and seteq 1”。

### Lines 1014-1025

```tablegen
foreach reduction =
  [["int_wasm_anytrue", "ANYTRUE", "I8x16"],
   ["int_wasm_anytrue", "ANYTRUE", "I16x8"],
   ["int_wasm_anytrue", "ANYTRUE", "I32x4"],
   ["int_wasm_anytrue", "ANYTRUE", "I64x2"],
   ["int_wasm_alltrue", "ALLTRUE_I8x16", "I8x16"],
   ["int_wasm_alltrue", "ALLTRUE_I16x8", "I16x8"],
   ["int_wasm_alltrue", "ALLTRUE_I32x4", "I32x4"],
   ["int_wasm_alltrue", "ALLTRUE_I64x2", "I64x2"]] in {
defvar intrinsic = !cast<Intrinsic>(reduction[0]);
defvar inst = !cast<NI>(reduction[1]);
defvar vec = !cast<Vec>(reduction[2]);
```
- **EN**: Continues the WebAssembly backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 WebAssembly 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 1026-1032

```tablegen
def : Pat<(i32 (and (i32 (intrinsic (vec.vt V128:$x))), (i32 1))), (inst $x)>;
def : Pat<(i32 (setne (i32 (intrinsic (vec.vt V128:$x))), (i32 0))), (inst $x)>;
def : Pat<(i32 (setne (i32 (intrinsic (vec.vt V128:$x))), (i32 1))), (i32 (EQZ_I32 (inst $x)))>;
def : Pat<(i32 (seteq (i32 (intrinsic (vec.vt V128:$x))), (i32 1))), (inst $x)>;
def : Pat<(i32 (seteq (i32 (intrinsic (vec.vt V128:$x))), (i32 0))), (i32 (EQZ_I32 (inst $x)))>;
}
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1033-1040

```tablegen
multiclass SIMDBitmask<Vec vec, bits<32> simdop> {
  defm _#vec : SIMD_I<(outs I32:$dst), (ins V128:$vec), (outs), (ins),
                      [(set I32:$dst,
                         (i32 (int_wasm_bitmask (vec.vt V128:$vec))))],
                      vec.prefix#".bitmask\t$dst, $vec", vec.prefix#".bitmask",
                      simdop>;
}
```
- **EN**: Adds declarative TableGen records such as `SIMDBitmask` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `SIMDBitmask`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1041-1045

```tablegen
defm BITMASK : SIMDBitmask<I8x16, 100>;
defm BITMASK : SIMDBitmask<I16x8, 132>;
defm BITMASK : SIMDBitmask<I32x4, 164>;
defm BITMASK : SIMDBitmask<I64x2, 196>;
```
- **EN**: Adds declarative TableGen records such as `BITMASK` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `BITMASK`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1046-1047

```tablegen
//===----------------------------------------------------------------------===//
// Bit shifts
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Bit shifts".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Bit shifts”。

### Lines 1048-1056

```tablegen
//===----------------------------------------------------------------------===//

multiclass SIMDShift<Vec vec, SDNode node, string name, bits<32> simdop> {
  defm _#vec : SIMD_I<(outs V128:$dst), (ins V128:$vec, I32:$x), (outs), (ins),
                      [(set (vec.vt V128:$dst), (node V128:$vec, I32:$x))],
                      vec.prefix#"."#name#"\t$dst, $vec, $x",
                      vec.prefix#"."#name, simdop>;
}
```
- **EN**: Adds declarative TableGen records such as `SIMDShift` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `SIMDShift`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1057-1065

```tablegen
multiclass SIMDShiftInt<SDNode node, string name, bits<32> baseInst> {
  defm "" : SIMDShift<I8x16, node, name, baseInst>;
  defm "" : SIMDShift<I16x8, node, name, !add(baseInst, 32)>;
  defm "" : SIMDShift<I32x4, node, name, !add(baseInst, 64)>;
  defm "" : SIMDShift<I64x2, node, name, !add(baseInst, 96)>;
}

// WebAssembly SIMD shifts are nonstandard in that the shift amount is
// an i32 rather than a vector, so they need custom nodes.
```
- **EN**: Adds declarative TableGen records such as `SIMDShiftInt` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `SIMDShiftInt`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1066-1072

```tablegen
def wasm_shift_t :
  SDTypeProfile<1, 2, [SDTCisVec<0>, SDTCisSameAs<0, 1>, SDTCisVT<2, i32>]>;
def wasm_shl : SDNode<"WebAssemblyISD::VEC_SHL", wasm_shift_t>;
def wasm_shr_s : SDNode<"WebAssemblyISD::VEC_SHR_S", wasm_shift_t>;
def wasm_shr_u : SDNode<"WebAssemblyISD::VEC_SHR_U", wasm_shift_t>;

// Left shift by scalar: shl
```
- **EN**: Adds declarative TableGen records such as `wasm_shift_t`, `wasm_shl`, `wasm_shr_s` that LLVM later expands into generated tables or helper code. This range works in the SelectionDAG-based lowering pipeline.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `wasm_shift_t`, `wasm_shl`, `wasm_shr_s`，LLVM 随后会把它们展开成生成表或辅助代码。 这一段工作在基于 SelectionDAG 的 lowering 流水线中。

### Lines 1073-1079

```tablegen
defm SHL : SIMDShiftInt<wasm_shl, "shl", 107>;

// Right shift by scalar: shr_s / shr_u
defm SHR_S : SIMDShiftInt<wasm_shr_s, "shr_s", 108>;
defm SHR_U : SIMDShiftInt<wasm_shr_u, "shr_u", 109>;

// Optimize away an explicit mask on a shift count.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Right shift by scalar: shr_s / shr_u". Notable symbols in this range include `SHL`, `SHR_S`, `SHR_U`.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Right shift by scalar: shr_s / shr_u”。 该区间中较显眼的符号包括 `SHL`, `SHR_S`, `SHR_U`。

### Lines 1080-1086

```tablegen
def : Pat<(wasm_shl (v16i8 V128:$lhs), (and I32:$rhs, 7)),
          (SHL_I8x16 V128:$lhs, I32:$rhs)>;
def : Pat<(wasm_shr_s (v16i8 V128:$lhs), (and I32:$rhs, 7)),
          (SHR_S_I8x16 V128:$lhs, I32:$rhs)>;
def : Pat<(wasm_shr_u (v16i8 V128:$lhs), (and I32:$rhs, 7)),
          (SHR_U_I8x16 V128:$lhs, I32:$rhs)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1087-1093

```tablegen
def : Pat<(wasm_shl (v8i16 V128:$lhs), (and I32:$rhs, 15)),
          (SHL_I16x8 V128:$lhs, I32:$rhs)>;
def : Pat<(wasm_shr_s (v8i16 V128:$lhs), (and I32:$rhs, 15)),
          (SHR_S_I16x8 V128:$lhs, I32:$rhs)>;
def : Pat<(wasm_shr_u (v8i16 V128:$lhs), (and I32:$rhs, 15)),
          (SHR_U_I16x8 V128:$lhs, I32:$rhs)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1094-1100

```tablegen
def : Pat<(wasm_shl (v4i32 V128:$lhs), (and I32:$rhs, 31)),
          (SHL_I32x4 V128:$lhs, I32:$rhs)>;
def : Pat<(wasm_shr_s (v4i32 V128:$lhs), (and I32:$rhs, 31)),
          (SHR_S_I32x4 V128:$lhs, I32:$rhs)>;
def : Pat<(wasm_shr_u (v4i32 V128:$lhs), (and I32:$rhs, 31)),
          (SHR_U_I32x4 V128:$lhs, I32:$rhs)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1101-1108

```tablegen
def : Pat<(wasm_shl (v2i64 V128:$lhs), (and I32:$rhs, 63)),
          (SHL_I64x2 V128:$lhs, I32:$rhs)>;
def : Pat<(wasm_shr_s (v2i64 V128:$lhs), (and I32:$rhs, 63)),
          (SHR_S_I64x2 V128:$lhs, I32:$rhs)>;
def : Pat<(wasm_shr_u (v2i64 V128:$lhs), (and I32:$rhs, 63)),
          (SHR_U_I64x2 V128:$lhs, I32:$rhs)>;
def : Pat<(wasm_shl (v2i64 V128:$lhs), (trunc (and I64:$rhs, 63))),
          (SHL_I64x2 V128:$lhs, (I32_WRAP_I64 I64:$rhs))>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1109-1113

```tablegen
def : Pat<(wasm_shr_s (v2i64 V128:$lhs), (trunc (and I64:$rhs, 63))),
          (SHR_S_I64x2 V128:$lhs, (I32_WRAP_I64 I64:$rhs))>;
def : Pat<(wasm_shr_u (v2i64 V128:$lhs), (trunc (and I64:$rhs, 63))),
          (SHR_U_I64x2 V128:$lhs, (I32_WRAP_I64 I64:$rhs))>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1114-1115

```tablegen
//===----------------------------------------------------------------------===//
// Integer binary arithmetic
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Integer binary arithmetic".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Integer binary arithmetic”。

### Lines 1116-1123

```tablegen
//===----------------------------------------------------------------------===//

multiclass SIMDBinaryIntNoI8x16<SDPatternOperator node, string name, bits<32> baseInst> {
  defm "" : SIMDBinary<I16x8, node, name, !add(baseInst, 32)>;
  defm "" : SIMDBinary<I32x4, node, name, !add(baseInst, 64)>;
  defm "" : SIMDBinary<I64x2, node, name, !add(baseInst, 96)>;
}
```
- **EN**: Adds declarative TableGen records such as `SIMDBinaryIntNoI8x16` that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `SIMDBinaryIntNoI8x16`，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 1124-1130

```tablegen
multiclass SIMDBinaryIntSmall<SDPatternOperator node, string name, bits<32> baseInst> {
  defm "" : SIMDBinary<I8x16, node, name, baseInst>;
  defm "" : SIMDBinary<I16x8, node, name, !add(baseInst, 32)>;
}

multiclass SIMDBinaryIntNoI64x2<SDPatternOperator node, string name, bits<32> baseInst> {
  defm "" : SIMDBinaryIntSmall<node, name, baseInst>;
```
- **EN**: Adds declarative TableGen records such as `SIMDBinaryIntSmall`, `SIMDBinaryIntNoI64x2` that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `SIMDBinaryIntSmall`, `SIMDBinaryIntNoI64x2`，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 1131-1139

```tablegen
  defm "" : SIMDBinary<I32x4, node, name, !add(baseInst, 64)>;
}

multiclass SIMDBinaryInt<SDPatternOperator node, string name, bits<32> baseInst> {
  defm "" : SIMDBinaryIntNoI64x2<node, name, baseInst>;
  defm "" : SIMDBinary<I64x2, node, name, !add(baseInst, 96)>;
}

// Integer addition: add / add_sat_s / add_sat_u
```
- **EN**: Adds declarative TableGen records such as `SIMDBinaryInt` that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `SIMDBinaryInt`，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 1140-1146

```tablegen
let isCommutable = 1 in {
defm ADD : SIMDBinaryInt<add, "add", 110>;
defm ADD_SAT_S : SIMDBinaryIntSmall<saddsat, "add_sat_s", 111>;
defm ADD_SAT_U : SIMDBinaryIntSmall<uaddsat, "add_sat_u", 112>;
} // isCommutable = 1

// Integer subtraction: sub / sub_sat_s / sub_sat_u
```
- **EN**: Adds declarative TableGen records such as `ADD`, `ADD_SAT_S`, `ADD_SAT_U` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `ADD`, `ADD_SAT_S`, `ADD_SAT_U`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1147-1155

```tablegen
defm SUB : SIMDBinaryInt<sub, "sub", 113>;
defm SUB_SAT_S : SIMDBinaryIntSmall<ssubsat, "sub_sat_s", 114>;
defm SUB_SAT_U : SIMDBinaryIntSmall<usubsat, "sub_sat_u", 115>;

// Integer multiplication: mul
let isCommutable = 1 in
defm MUL : SIMDBinaryIntNoI8x16<mul, "mul", 117>;

// Integer min_s / min_u / max_s / max_u
```
- **EN**: Adds declarative TableGen records such as `SUB`, `SUB_SAT_S`, `SUB_SAT_U` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `SUB`, `SUB_SAT_S`, `SUB_SAT_U`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1156-1163

```tablegen
let isCommutable = 1 in {
defm MIN_S : SIMDBinaryIntNoI64x2<smin, "min_s", 118>;
defm MIN_U : SIMDBinaryIntNoI64x2<umin, "min_u", 119>;
defm MAX_S : SIMDBinaryIntNoI64x2<smax, "max_s", 120>;
defm MAX_U : SIMDBinaryIntNoI64x2<umax, "max_u", 121>;
} // isCommutable = 1

// Integer unsigned rounding average: avgr_u
```
- **EN**: Adds declarative TableGen records such as `MIN_S`, `MIN_U`, `MAX_S` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `MIN_S`, `MIN_U`, `MAX_S`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1164-1171

```tablegen
let isCommutable = 1 in {
defm AVGR_U : SIMDBinary<I8x16, int_wasm_avgr_unsigned, "avgr_u", 123>;
defm AVGR_U : SIMDBinary<I16x8, int_wasm_avgr_unsigned, "avgr_u", 155>;
}

def add_nuw : PatFrag<(ops node:$lhs, node:$rhs), (add $lhs, $rhs),
                      "return N->getFlags().hasNoUnsignedWrap();">;
```
- **EN**: Adds declarative TableGen records such as `AVGR_U`, `add_nuw` that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `AVGR_U`, `add_nuw`，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 1172-1180

```tablegen
foreach vec = [I8x16, I16x8] in {
defvar inst = !cast<NI>("AVGR_U_"#vec);
def : Pat<(wasm_shr_u
            (add_nuw
              (add_nuw (vec.vt V128:$lhs), (vec.vt V128:$rhs)),
              (vec.splat (i32 1))),
            (i32 1)),
          (inst $lhs, $rhs)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1181-1187

```tablegen
def : Pat<(vec.vt (avgceilu (vec.vt V128:$lhs), (vec.vt V128:$rhs))),
          (inst $lhs, $rhs)>;
}

// Widening dot product: i32x4.dot_i16x8_s
def dot_t : SDTypeProfile<1, 2, [SDTCisVT<0, v4i32>, SDTCisVT<1, v8i16>, SDTCisVT<2, v8i16>]>;
def wasm_dot : SDNode<"WebAssemblyISD::DOT", dot_t>;
```
- **EN**: Adds declarative TableGen records such as `dot_t`, `wasm_dot` that LLVM later expands into generated tables or helper code. This range works in the SelectionDAG-based lowering pipeline.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `dot_t`, `wasm_dot`，LLVM 随后会把它们展开成生成表或辅助代码。 这一段工作在基于 SelectionDAG 的 lowering 流水线中。

### Lines 1188-1196

```tablegen
let isCommutable = 1 in
defm DOT : SIMD_I<(outs V128:$dst), (ins V128:$lhs, V128:$rhs), (outs), (ins),
                  [(set V128:$dst, (int_wasm_dot V128:$lhs, V128:$rhs))],
                  "i32x4.dot_i16x8_s\t$dst, $lhs, $rhs", "i32x4.dot_i16x8_s",
                  186>;
def : Pat<(wasm_dot V128:$lhs, V128:$rhs),
          (DOT $lhs, $rhs)>;

// Extending multiplication: extmul_{low,high}_P, extmul_high
```
- **EN**: Adds declarative TableGen records such as `DOT` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `DOT`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1197-1204

```tablegen
def extend_t : SDTypeProfile<1, 1, [SDTCisVec<0>, SDTCisVec<1>]>;
def extend_low_s : SDNode<"WebAssemblyISD::EXTEND_LOW_S", extend_t>;
def extend_high_s : SDNode<"WebAssemblyISD::EXTEND_HIGH_S", extend_t>;
def extend_low_u : SDNode<"WebAssemblyISD::EXTEND_LOW_U", extend_t>;
def extend_high_u : SDNode<"WebAssemblyISD::EXTEND_HIGH_U", extend_t>;

multiclass SIMDExtBinary<Vec vec, SDPatternOperator node, string name,
                         bits<32> simdop> {
```
- **EN**: Adds declarative TableGen records such as `extend_t`, `extend_low_s`, `extend_high_s` that LLVM later expands into generated tables or helper code. This range works in the SelectionDAG-based lowering pipeline.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `extend_t`, `extend_low_s`, `extend_high_s`，LLVM 随后会把它们展开成生成表或辅助代码。 这一段工作在基于 SelectionDAG 的 lowering 流水线中。

### Lines 1205-1212

```tablegen
  defm _#vec : SIMD_I<(outs V128:$dst), (ins V128:$lhs, V128:$rhs),
                      (outs), (ins),
                      [(set (vec.vt V128:$dst), (node
                         (vec.split.vt V128:$lhs),(vec.split.vt V128:$rhs)))],
                      vec.prefix#"."#name#"\t$dst, $lhs, $rhs",
                      vec.prefix#"."#name, simdop>;
}
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 1213-1219

```tablegen
class ExtMulPat<SDNode extend> :
  PatFrag<(ops node:$lhs, node:$rhs),
          (mul (extend $lhs), (extend $rhs))> {}

def extmul_low_s : ExtMulPat<extend_low_s>;
def extmul_high_s : ExtMulPat<extend_high_s>;
def extmul_low_u : ExtMulPat<extend_low_u>;
```
- **EN**: Declares a backend-facing type `ExtMulPat`, `extmul_low_s`, `extmul_high_s` and outlines the API or state that nearby code will rely on. These declarations feed generated pattern-matching logic.
- **CN**: 这里声明面向后端的类型 `ExtMulPat`, `extmul_low_s`, `extmul_high_s`，并勾勒出周边代码会依赖的接口或状态。 这些声明会进入生成式模式匹配逻辑。

### Lines 1220-1227

```tablegen
def extmul_high_u : ExtMulPat<extend_high_u>;

defm EXTMUL_LOW_S :
  SIMDExtBinary<I16x8, extmul_low_s, "extmul_low_i8x16_s", 0x9c>;
defm EXTMUL_HIGH_S :
  SIMDExtBinary<I16x8, extmul_high_s, "extmul_high_i8x16_s", 0x9d>;
defm EXTMUL_LOW_U :
  SIMDExtBinary<I16x8, extmul_low_u, "extmul_low_i8x16_u", 0x9e>;
```
- **EN**: Adds declarative TableGen records such as `extmul_high_u`, `EXTMUL_LOW_S`, `EXTMUL_HIGH_S` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `extmul_high_u`, `EXTMUL_LOW_S`, `EXTMUL_HIGH_S`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1228-1234

```tablegen
defm EXTMUL_HIGH_U :
  SIMDExtBinary<I16x8, extmul_high_u, "extmul_high_i8x16_u", 0x9f>;

defm EXTMUL_LOW_S :
  SIMDExtBinary<I32x4, extmul_low_s, "extmul_low_i16x8_s", 0xbc>;
defm EXTMUL_HIGH_S :
  SIMDExtBinary<I32x4, extmul_high_s, "extmul_high_i16x8_s", 0xbd>;
```
- **EN**: Adds declarative TableGen records such as `EXTMUL_HIGH_U`, `EXTMUL_LOW_S`, `EXTMUL_HIGH_S` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `EXTMUL_HIGH_U`, `EXTMUL_LOW_S`, `EXTMUL_HIGH_S`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1235-1241

```tablegen
defm EXTMUL_LOW_U :
  SIMDExtBinary<I32x4, extmul_low_u, "extmul_low_i16x8_u", 0xbe>;
defm EXTMUL_HIGH_U :
  SIMDExtBinary<I32x4, extmul_high_u, "extmul_high_i16x8_u", 0xbf>;

defm EXTMUL_LOW_S :
  SIMDExtBinary<I64x2, extmul_low_s, "extmul_low_i32x4_s", 0xdc>;
```
- **EN**: Adds declarative TableGen records such as `EXTMUL_LOW_U`, `EXTMUL_HIGH_U`, `EXTMUL_LOW_S` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `EXTMUL_LOW_U`, `EXTMUL_HIGH_U`, `EXTMUL_LOW_S`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1242-1249

```tablegen
defm EXTMUL_HIGH_S :
  SIMDExtBinary<I64x2, extmul_high_s, "extmul_high_i32x4_s", 0xdd>;
defm EXTMUL_LOW_U :
  SIMDExtBinary<I64x2, extmul_low_u, "extmul_low_i32x4_u", 0xde>;
defm EXTMUL_HIGH_U :
  SIMDExtBinary<I64x2, extmul_high_u, "extmul_high_i32x4_u", 0xdf>;

// Pattern for i32x4.dot_i16x8_s
```
- **EN**: Adds declarative TableGen records such as `EXTMUL_HIGH_S`, `EXTMUL_LOW_U`, `EXTMUL_HIGH_U` that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `EXTMUL_HIGH_S`, `EXTMUL_LOW_U`, `EXTMUL_HIGH_U`，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 1250-1269

```tablegen
def : Pat<
  (v4i32 (add
    (wasm_shuffle
      (v4i32 (extmul_low_s v8i16:$lhs, v8i16:$rhs)),
      (v4i32 (extmul_high_s v8i16:$lhs, v8i16:$rhs)),
      (i32 0), (i32 1), (i32 2), (i32 3),
      (i32 8), (i32 9), (i32 10), (i32 11),
      (i32 16), (i32 17), (i32 18), (i32 19),
      (i32 24), (i32 25), (i32 26), (i32 27)),
    (wasm_shuffle
      (v4i32 (extmul_low_s v8i16:$lhs, v8i16:$rhs)),
      (v4i32 (extmul_high_s v8i16:$lhs, v8i16:$rhs)),
      (i32 4), (i32 5), (i32 6), (i32 7),
      (i32 12), (i32 13), (i32 14), (i32 15),
      (i32 20), (i32 21), (i32 22), (i32 23),
      (i32 28), (i32 29), (i32 30), (i32 31)))
  ),
  (v4i32 (DOT v8i16:$lhs, v8i16:$rhs))
>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1270-1271

```tablegen
//===----------------------------------------------------------------------===//
// Floating-point unary arithmetic
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Floating-point unary arithmetic".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Floating-point unary arithmetic”。

### Lines 1272-1278

```tablegen
//===----------------------------------------------------------------------===//

multiclass SIMDUnaryFP<SDNode node, string name, bits<32> baseInst> {
  defm "" : SIMDUnary<F32x4, node, name, baseInst>;
  defm "" : SIMDUnary<F64x2, node, name, !add(baseInst, 12)>;
  // Unlike F32x4 and F64x2 there's not a gap in the opcodes between "neg" and
  // "sqrt" so subtract one from the offset.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Unlike F32x4 and F64x2 there's not a gap in the opcodes between "neg" and". Notable symbols in this range include `SIMDUnaryFP`.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Unlike F32x4 and F64x2 there's not a gap in the opcodes between "neg" and”。 该区间中较显眼的符号包括 `SIMDUnaryFP`。

### Lines 1279-1286

```tablegen
  defm "" : HalfPrecisionUnary<F16x8, node, name,
                               !add(baseInst,!if(!eq(name, "sqrt"), 79, 80))>;
}

// Absolute value: abs
defm ABS : SIMDUnaryFP<fabs, "abs", 224>;

// Negation: neg
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Absolute value: abs". Notable symbols in this range include `ABS`.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Absolute value: abs”。 该区间中较显眼的符号包括 `ABS`。

### Lines 1287-1293

```tablegen
defm NEG : SIMDUnaryFP<fneg, "neg", 225>;

// Square root: sqrt
defm SQRT : SIMDUnaryFP<fsqrt, "sqrt", 227>;

// Rounding: ceil, floor, trunc, nearest
defm CEIL : SIMDUnary<F32x4, fceil, "ceil", 0x67>;
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Square root: sqrt". Notable symbols in this range include `NEG`, `SQRT`, `CEIL`.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Square root: sqrt”。 该区间中较显眼的符号包括 `NEG`, `SQRT`, `CEIL`。

### Lines 1294-1300

```tablegen
defm FLOOR : SIMDUnary<F32x4, ffloor, "floor", 0x68>;
defm TRUNC: SIMDUnary<F32x4, ftrunc, "trunc", 0x69>;
defm NEAREST: SIMDUnary<F32x4, fnearbyint, "nearest", 0x6a>;
defm CEIL : SIMDUnary<F64x2, fceil, "ceil", 0x74>;
defm FLOOR : SIMDUnary<F64x2, ffloor, "floor", 0x75>;
defm TRUNC: SIMDUnary<F64x2, ftrunc, "trunc", 0x7a>;
defm NEAREST: SIMDUnary<F64x2, fnearbyint, "nearest", 0x94>;
```
- **EN**: Adds declarative TableGen records such as `FLOOR`, `TRUNC`, `NEAREST` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `FLOOR`, `TRUNC`, `NEAREST`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1301-1307

```tablegen
defm CEIL : HalfPrecisionUnary<F16x8, fceil, "ceil", 0x133>;
defm FLOOR : HalfPrecisionUnary<F16x8, ffloor, "floor", 0x134>;
defm TRUNC : HalfPrecisionUnary<F16x8, ftrunc, "trunc", 0x135>;
defm NEAREST : HalfPrecisionUnary<F16x8, fnearbyint, "nearest", 0x136>;

// WebAssembly doesn't expose inexact exceptions, so map frint to fnearbyint.
def : Pat<(v4f32 (frint (v4f32 V128:$src))), (NEAREST_F32x4 V128:$src)>;
```
- **EN**: Adds declarative TableGen records such as `CEIL`, `FLOOR`, `TRUNC` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `CEIL`, `FLOOR`, `TRUNC`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1308-1315

```tablegen
def : Pat<(v2f64 (frint (v2f64 V128:$src))), (NEAREST_F64x2 V128:$src)>;
def : Pat<(v8f16 (frint (v8f16 V128:$src))), (NEAREST_F16x8 V128:$src)>;

// WebAssembly always rounds ties-to-even, so map froundeven to fnearbyint.
def : Pat<(v4f32 (froundeven (v4f32 V128:$src))), (NEAREST_F32x4 V128:$src)>;
def : Pat<(v2f64 (froundeven (v2f64 V128:$src))), (NEAREST_F64x2 V128:$src)>;
def : Pat<(v8f16 (froundeven (v8f16 V128:$src))), (NEAREST_F16x8 V128:$src)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1316-1317

```tablegen
//===----------------------------------------------------------------------===//
// Floating-point binary arithmetic
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Floating-point binary arithmetic".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Floating-point binary arithmetic”。

### Lines 1318-1326

```tablegen
//===----------------------------------------------------------------------===//

multiclass SIMDBinaryFP<SDPatternOperator node, string name, bits<32> baseInst> {
  defm "" : SIMDBinary<F32x4, node, name, baseInst>;
  defm "" : SIMDBinary<F64x2, node, name, !add(baseInst, 12)>;
  defm "" : HalfPrecisionBinary<F16x8, node, name, !add(baseInst, 89)>;
}

// Addition: add
```
- **EN**: Adds declarative TableGen records such as `SIMDBinaryFP` that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `SIMDBinaryFP`，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 1327-1333

```tablegen
let isCommutable = 1 in
defm ADD : SIMDBinaryFP<fadd, "add", 228>;

// Subtraction: sub
defm SUB : SIMDBinaryFP<fsub, "sub", 229>;

// Multiplication: mul
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Subtraction: sub". Notable symbols in this range include `ADD`, `SUB`.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Subtraction: sub”。 该区间中较显眼的符号包括 `ADD`, `SUB`。

### Lines 1334-1340

```tablegen
let isCommutable = 1 in
defm MUL : SIMDBinaryFP<fmul, "mul", 230>;

// Division: div
defm DIV : SIMDBinaryFP<fdiv, "div", 231>;

// NaN-propagating minimum: min
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Division: div". Notable symbols in this range include `MUL`, `DIV`.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Division: div”。 该区间中较显眼的符号包括 `MUL`, `DIV`。

### Lines 1341-1356

```tablegen
defm MIN : SIMDBinaryFP<fminimum, "min", 232>;

// NaN-propagating maximum: max
defm MAX : SIMDBinaryFP<fmaximum, "max", 233>;

// Pseudo-minimum: pmin
def pmin : PatFrags<(ops node:$lhs, node:$rhs), [
                    (vselect (setolt $rhs, $lhs), $rhs, $lhs),
                    (vselect (setole $rhs, $lhs), $rhs, $lhs),
                    (vselect (setogt $lhs, $rhs), $rhs, $lhs),
                    (vselect (setoge $lhs, $rhs), $rhs, $lhs),
                    (vselect (setlt $lhs, $rhs), $lhs, $rhs),
                    (vselect (setle $lhs, $rhs), $lhs, $rhs),
                    (vselect (setgt $lhs, $rhs), $rhs, $lhs),
                    (vselect (setge $lhs, $rhs), $rhs, $lhs)
]>;
```
- **EN**: Adds declarative TableGen records such as `MIN`, `MAX`, `pmin` that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `MIN`, `MAX`, `pmin`，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 1357-1369

```tablegen
defm PMIN : SIMDBinaryFP<pmin, "pmin", 234>;

// Pseudo-maximum: pmax
def pmax : PatFrags<(ops node:$lhs, node:$rhs), [
                    (vselect (setogt $rhs, $lhs), $rhs, $lhs),
                    (vselect (setoge $rhs, $lhs), $rhs, $lhs),
                    (vselect (setolt $lhs, $rhs), $rhs, $lhs),
                    (vselect (setole $lhs, $rhs), $rhs, $lhs),
                    (vselect (setgt $lhs, $rhs), $lhs, $rhs),
                    (vselect (setge $lhs, $rhs), $lhs, $rhs),
                    (vselect (setlt $lhs, $rhs), $rhs, $lhs),
                    (vselect (setle $lhs, $rhs), $rhs, $lhs)
]>;
```
- **EN**: Adds declarative TableGen records such as `PMIN`, `pmax` that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `PMIN`, `pmax`，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 1370-1377

```tablegen
defm PMAX : SIMDBinaryFP<pmax, "pmax", 235>;

multiclass PMinMaxInt<Vec vec, NI baseMinInst, NI baseMaxInst> {
  def : Pat<(vec.int_vt (vselect
              (setolt (vec.vt (bitconvert V128:$rhs)),
                      (vec.vt (bitconvert V128:$lhs))),
              V128:$rhs, V128:$lhs)),
            (baseMinInst $lhs, $rhs)>;
```
- **EN**: Adds declarative TableGen records such as `PMAX`, `PMinMaxInt` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `PMAX`, `PMinMaxInt`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1378-1386

```tablegen
  def : Pat<(vec.int_vt (vselect
              (setolt (vec.vt (bitconvert V128:$lhs)),
                      (vec.vt (bitconvert V128:$rhs))),
              V128:$rhs, V128:$lhs)),
            (baseMaxInst $lhs, $rhs)>;
}
// Also match the pmin/pmax cases where the operands are int vectors (but the
// comparison is still a floating point comparison). This can happen when using
// the wasm_simd128.h intrinsics because v128_t is an integer vector.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Also match the pmin/pmax cases where the operands are int vectors (but the".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Also match the pmin/pmax cases where the operands are int vectors (but the”。

### Lines 1387-1393

```tablegen
foreach vec = [F32x4, F64x2, F16x8] in {
  defvar pmin = !cast<NI>("PMIN_"#vec);
  defvar pmax = !cast<NI>("PMAX_"#vec);
  defm : PMinMaxInt<vec, pmin, pmax>;
}

// And match the pmin/pmax LLVM intrinsics as well
```
- **EN**: Continues the WebAssembly backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 WebAssembly 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 1394-1401

```tablegen
def : Pat<(v4f32 (int_wasm_pmin (v4f32 V128:$lhs), (v4f32 V128:$rhs))),
          (PMIN_F32x4 V128:$lhs, V128:$rhs)>;
def : Pat<(v4f32 (int_wasm_pmax (v4f32 V128:$lhs), (v4f32 V128:$rhs))),
          (PMAX_F32x4 V128:$lhs, V128:$rhs)>;
def : Pat<(v2f64 (int_wasm_pmin (v2f64 V128:$lhs), (v2f64 V128:$rhs))),
          (PMIN_F64x2 V128:$lhs, V128:$rhs)>;
def : Pat<(v2f64 (int_wasm_pmax (v2f64 V128:$lhs), (v2f64 V128:$rhs))),
          (PMAX_F64x2 V128:$lhs, V128:$rhs)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1402-1406

```tablegen
def : Pat<(v8f16 (int_wasm_pmin (v8f16 V128:$lhs), (v8f16 V128:$rhs))),
          (PMIN_F16x8 V128:$lhs, V128:$rhs)>;
def : Pat<(v8f16 (int_wasm_pmax (v8f16 V128:$lhs), (v8f16 V128:$rhs))),
          (PMAX_F16x8 V128:$lhs, V128:$rhs)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1407-1408

```tablegen
//===----------------------------------------------------------------------===//
// Conversions
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Conversions".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Conversions”。

### Lines 1409-1418

```tablegen
//===----------------------------------------------------------------------===//

multiclass SIMDConvert<Vec vec, Vec arg, SDPatternOperator op, string name,
                       bits<32> simdop, list<Predicate> reqs = []> {
  defm op#_#vec :
    SIMD_I<(outs V128:$dst), (ins V128:$vec), (outs), (ins),
           [(set (vec.vt V128:$dst), (vec.vt (op (arg.vt V128:$vec))))],
           vec.prefix#"."#name#"\t$dst, $vec", vec.prefix#"."#name, simdop, reqs>;
}
```
- **EN**: Adds declarative TableGen records such as `SIMDConvert` that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `SIMDConvert`，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 1419-1425

```tablegen
multiclass HalfPrecisionConvert<Vec vec, Vec arg, SDPatternOperator op,
                                string name, bits<32> simdop> {
  defm "" : SIMDConvert<vec, arg, op, name, simdop, [HasFP16]>;
}

// Floating point to integer with saturation: trunc_sat
defm "" : SIMDConvert<I32x4, F32x4, fp_to_sint, "trunc_sat_f32x4_s", 248>;
```
- **EN**: Adds declarative TableGen records such as `HalfPrecisionConvert` that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `HalfPrecisionConvert`，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 1426-1432

```tablegen
defm "" : SIMDConvert<I32x4, F32x4, fp_to_uint, "trunc_sat_f32x4_u", 249>;
defm "" : HalfPrecisionConvert<I16x8, F16x8, fp_to_sint, "trunc_sat_f16x8_s", 0x145>;
defm "" : HalfPrecisionConvert<I16x8, F16x8, fp_to_uint, "trunc_sat_f16x8_u", 0x146>;

// Support the saturating variety as well.
def trunc_s_sat32 : PatFrag<(ops node:$x), (fp_to_sint_sat $x, i32)>;
def trunc_u_sat32 : PatFrag<(ops node:$x), (fp_to_uint_sat $x, i32)>;
```
- **EN**: Adds declarative TableGen records such as `trunc_s_sat32`, `trunc_u_sat32` that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `trunc_s_sat32`, `trunc_u_sat32`，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 1433-1440

```tablegen
def : Pat<(v4i32 (trunc_s_sat32 (v4f32 V128:$src))), (fp_to_sint_I32x4 $src)>;
def : Pat<(v4i32 (trunc_u_sat32 (v4f32 V128:$src))), (fp_to_uint_I32x4 $src)>;

def trunc_s_sat16 : PatFrag<(ops node:$x), (fp_to_sint_sat $x, i16)>;
def trunc_u_sat16 : PatFrag<(ops node:$x), (fp_to_uint_sat $x, i16)>;
def : Pat<(v8i16 (trunc_s_sat16 (v8f16 V128:$src))), (fp_to_sint_I16x8 $src)>;
def : Pat<(v8i16 (trunc_u_sat16 (v8f16 V128:$src))), (fp_to_uint_I16x8 $src)>;
```
- **EN**: Adds declarative TableGen records such as `trunc_s_sat16`, `trunc_u_sat16` that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `trunc_s_sat16`, `trunc_u_sat16`，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 1441-1447

```tablegen
def trunc_sat_zero_t : SDTypeProfile<1, 1, [SDTCisVec<0>, SDTCisVec<1>]>;
def trunc_sat_zero_s :
  SDNode<"WebAssemblyISD::TRUNC_SAT_ZERO_S", trunc_sat_zero_t>;
def trunc_sat_zero_u :
  SDNode<"WebAssemblyISD::TRUNC_SAT_ZERO_U", trunc_sat_zero_t>;
defm "" : SIMDConvert<I32x4, F64x2, trunc_sat_zero_s, "trunc_sat_f64x2_s_zero",
                      0xfc>;
```
- **EN**: Adds declarative TableGen records such as `trunc_sat_zero_t`, `trunc_sat_zero_s`, `trunc_sat_zero_u` that LLVM later expands into generated tables or helper code. This range works in the SelectionDAG-based lowering pipeline.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `trunc_sat_zero_t`, `trunc_sat_zero_s`, `trunc_sat_zero_u`，LLVM 随后会把它们展开成生成表或辅助代码。 这一段工作在基于 SelectionDAG 的 lowering 流水线中。

### Lines 1448-1454

```tablegen
defm "" : SIMDConvert<I32x4, F64x2, trunc_sat_zero_u, "trunc_sat_f64x2_u_zero",
                      0xfd>;

// Integer to floating point: convert
def convert_low_t : SDTypeProfile<1, 1, [SDTCisVec<0>, SDTCisVec<1>]>;
def convert_low_s : SDNode<"WebAssemblyISD::CONVERT_LOW_S", convert_low_t>;
def convert_low_u : SDNode<"WebAssemblyISD::CONVERT_LOW_U", convert_low_t>;
```
- **EN**: Adds declarative TableGen records such as `convert_low_t`, `convert_low_s`, `convert_low_u` that LLVM later expands into generated tables or helper code. This range works in the SelectionDAG-based lowering pipeline.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `convert_low_t`, `convert_low_s`, `convert_low_u`，LLVM 随后会把它们展开成生成表或辅助代码。 这一段工作在基于 SelectionDAG 的 lowering 流水线中。

### Lines 1455-1463

```tablegen
defm "" : SIMDConvert<F32x4, I32x4, sint_to_fp, "convert_i32x4_s", 250>;
defm "" : SIMDConvert<F32x4, I32x4, uint_to_fp, "convert_i32x4_u", 251>;
defm "" : SIMDConvert<F64x2, I32x4, convert_low_s, "convert_low_i32x4_s", 0xfe>;
defm "" : SIMDConvert<F64x2, I32x4, convert_low_u, "convert_low_i32x4_u", 0xff>;
defm "" : HalfPrecisionConvert<F16x8, I16x8, sint_to_fp, "convert_i16x8_s", 0x147>;
defm "" : HalfPrecisionConvert<F16x8, I16x8, uint_to_fp, "convert_i16x8_u", 0x148>;

// Extending operations
// TODO: refactor this to be uniform for i64x2 if the numbering is not changed.
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1464-1470

```tablegen
multiclass SIMDExtend<Vec vec, bits<32> baseInst> {
  defm "" : SIMDConvert<vec, vec.split, extend_low_s,
                        "extend_low_"#vec.split.prefix#"_s", baseInst>;
  defm "" : SIMDConvert<vec, vec.split, extend_high_s,
                        "extend_high_"#vec.split.prefix#"_s", !add(baseInst, 1)>;
  defm "" : SIMDConvert<vec, vec.split, extend_low_u,
                        "extend_low_"#vec.split.prefix#"_u", !add(baseInst, 2)>;
```
- **EN**: Adds declarative TableGen records such as `SIMDExtend` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `SIMDExtend`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1471-1479

```tablegen
  defm "" : SIMDConvert<vec, vec.split, extend_high_u,
                        "extend_high_"#vec.split.prefix#"_u", !add(baseInst, 3)>;
}

defm "" : SIMDExtend<I16x8, 0x87>;
defm "" : SIMDExtend<I32x4, 0xa7>;
defm "" : SIMDExtend<I64x2, 0xc7>;

// Narrowing operations
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1480-1486

```tablegen
multiclass SIMDNarrow<Vec vec, bits<32> baseInst> {
  defvar name = vec.split.prefix#".narrow_"#vec.prefix;
  defm NARROW_S_#vec.split :
    SIMD_I<(outs V128:$dst), (ins V128:$low, V128:$high), (outs), (ins),
           [(set (vec.split.vt V128:$dst), (vec.split.vt (int_wasm_narrow_signed
             (vec.vt V128:$low), (vec.vt V128:$high))))],
           name#"_s\t$dst, $low, $high", name#"_s", baseInst>;
```
- **EN**: Adds declarative TableGen records such as `SIMDNarrow` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `SIMDNarrow`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1487-1493

```tablegen
  defm NARROW_U_#vec.split :
    SIMD_I<(outs V128:$dst), (ins V128:$low, V128:$high), (outs), (ins),
           [(set (vec.split.vt V128:$dst), (vec.split.vt (int_wasm_narrow_unsigned
             (vec.vt V128:$low), (vec.vt V128:$high))))],
           name#"_u\t$dst, $low, $high", name#"_u", !add(baseInst, 1)>;
}
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 1494-1501

```tablegen
defm "" : SIMDNarrow<I16x8, 101>;
defm "" : SIMDNarrow<I32x4, 133>;

// WebAssemblyISD::NARROW_U
def wasm_narrow_t : SDTypeProfile<1, 2, []>;
def wasm_narrow_u : SDNode<"WebAssemblyISD::NARROW_U", wasm_narrow_t>;
def : Pat<(v16i8 (wasm_narrow_u (v8i16 V128:$left), (v8i16 V128:$right))),
          (NARROW_U_I8x16 $left, $right)>;
```
- **EN**: Adds declarative TableGen records such as `wasm_narrow_t`, `wasm_narrow_u` that LLVM later expands into generated tables or helper code. This range works in the SelectionDAG-based lowering pipeline.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `wasm_narrow_t`, `wasm_narrow_u`，LLVM 随后会把它们展开成生成表或辅助代码。 这一段工作在基于 SelectionDAG 的 lowering 流水线中。

### Lines 1502-1509

```tablegen
def : Pat<(v8i16 (wasm_narrow_u (v4i32 V128:$left), (v4i32 V128:$right))),
          (NARROW_U_I16x8 $left, $right)>;

// Recognize a saturating truncation and convert into the corresponding
// narrow_TYPE_s or narrow_TYPE_u instruction.
multiclass SignedSaturatingTruncate<ValueType input, ValueType output,
                                    Instruction narrow, int minval,
                                    int maxval, int mask> {
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Recognize a saturating truncation and convert into the corresponding". Notable symbols in this range include `SignedSaturatingTruncate`.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Recognize a saturating truncation and convert into the corresponding”。 该区间中较显眼的符号包括 `SignedSaturatingTruncate`。

### Lines 1510-1519

```tablegen
  def : Pat<
    (output (wasm_narrow_u
      (and (smin (smax (input V128:$a), (splat_vector (i32 minval))),
           (splat_vector (i32 maxval))), (splat_vector (i32 mask))),
      (and (smin (smax (input V128:$b), (splat_vector (i32 minval))),
           (splat_vector (i32 maxval))), (splat_vector (i32 mask)))
    )),
    (narrow V128:$a, V128:$b)
  >;
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 1520-1530

```tablegen
  def : Pat<
    (output (wasm_narrow_u
      (and (smax (smin (input V128:$a), (splat_vector (i32 maxval))),
           (splat_vector (i32 minval))), (splat_vector (i32 mask))),
      (and (smax (smin (input V128:$b), (splat_vector (i32 maxval))),
           (splat_vector (i32 minval))), (splat_vector (i32 mask)))
    )),
    (narrow V128:$a, V128:$b)
  >;
}
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 1531-1544

```tablegen
defm : SignedSaturatingTruncate<v8i16, v16i8, NARROW_S_I8x16, -128, 127, 0xFF>;
defm : SignedSaturatingTruncate<v4i32, v8i16, NARROW_S_I16x8, -32768, 32767, 0xFFFF>;

multiclass UnsignedSaturatingTruncate<ValueType input, ValueType output,
                                    Instruction narrow, int maxval> {
  def : Pat<
    (output (wasm_narrow_u
      (umin (input V128:$a), (splat_vector (i32 maxval))),
      (umin (input V128:$b), (splat_vector (i32 maxval)))
    )),
    (narrow V128:$a, V128:$b)
  >;
}
```
- **EN**: Adds declarative TableGen records such as `UnsignedSaturatingTruncate` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `UnsignedSaturatingTruncate`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1545-1552

```tablegen
defm : UnsignedSaturatingTruncate<v8i16, v16i8, NARROW_U_I8x16, 0xFF>;
defm : UnsignedSaturatingTruncate<v4i32, v8i16, NARROW_U_I16x8, 0xFFFF>;

// Bitcasts are nops
// Matching bitcast t1 to t1 causes strange errors, so avoid repeating types
foreach t1 = AllVecs in
foreach t2 = AllVecs in
if !ne(t1, t2) then
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Bitcasts are nops".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Bitcasts are nops”。

### Lines 1553-1560

```tablegen
def : Pat<(t1.vt (bitconvert (t2.vt V128:$v))), (t1.vt V128:$v)>;

// Extended pairwise addition
def extadd_pairwise_u : SDNode<"WebAssemblyISD::EXT_ADD_PAIRWISE_U", extend_t>;
def extadd_pairwise_s : SDNode<"WebAssemblyISD::EXT_ADD_PAIRWISE_S", extend_t>;

defm "" : SIMDConvert<I16x8, I8x16, extadd_pairwise_s,
                      "extadd_pairwise_i8x16_s", 0x7c>;
```
- **EN**: Adds declarative TableGen records such as `extadd_pairwise_u`, `extadd_pairwise_s` that LLVM later expands into generated tables or helper code. This range works in the SelectionDAG-based lowering pipeline.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `extadd_pairwise_u`, `extadd_pairwise_s`，LLVM 随后会把它们展开成生成表或辅助代码。 这一段工作在基于 SelectionDAG 的 lowering 流水线中。

### Lines 1561-1567

```tablegen
defm "" : SIMDConvert<I16x8, I8x16, extadd_pairwise_u,
                      "extadd_pairwise_i8x16_u", 0x7d>;
defm "" : SIMDConvert<I32x4, I16x8, extadd_pairwise_s,
                      "extadd_pairwise_i16x8_s", 0x7e>;
defm "" : SIMDConvert<I32x4, I16x8, extadd_pairwise_u,
                      "extadd_pairwise_i16x8_u", 0x7f>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1568-1576

```tablegen
def : Pat<(v4i32 (int_wasm_extadd_pairwise_unsigned (v8i16 V128:$in))),
          (extadd_pairwise_u_I32x4 V128:$in)>;
def : Pat<(v8i16 (int_wasm_extadd_pairwise_unsigned (v16i8 V128:$in))),
          (extadd_pairwise_u_I16x8 V128:$in)>;
def : Pat<(v4i32 (int_wasm_extadd_pairwise_signed (v8i16 V128:$in))),
          (extadd_pairwise_s_I32x4 V128:$in)>;
def : Pat<(v8i16 (int_wasm_extadd_pairwise_signed (v16i8 V128:$in))),
          (extadd_pairwise_s_I16x8 V128:$in)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1577-1595

```tablegen
multiclass ExtAddPairwiseExtendShuffle<ValueType from_ty, ValueType to_ty, string suffix,
                              int a0, int a1, int a2, int a3, int a4, int a5, int a6, int a7,
                              int b0, int b1, int b2, int b3, int b4, int b5, int b6, int b7> {
  foreach sign = ["s", "u"] in {
    def : Pat<(to_ty (add
                    (!cast<SDNode>("extend_low_"#sign) (from_ty (wasm_shuffle (from_ty V128:$vec), (from_ty srcvalue),
                                  (i32 a0), (i32 a1), (i32 a2), (i32 a3),
                                  (i32 a4), (i32 a5), (i32 a6), (i32 a7),
                                  (i32 srcvalue), (i32 srcvalue), (i32 srcvalue), (i32 srcvalue),
                                  (i32 srcvalue), (i32 srcvalue), (i32 srcvalue), (i32 srcvalue)))),
                    (!cast<SDNode>("extend_low_"#sign) (from_ty (wasm_shuffle (from_ty V128:$vec), (from_ty srcvalue),
                                  (i32 b0), (i32 b1), (i32 b2), (i32 b3),
                                  (i32 b4), (i32 b5), (i32 b6), (i32 b7),
                                  (i32 srcvalue), (i32 srcvalue), (i32 srcvalue), (i32 srcvalue),
                                  (i32 srcvalue), (i32 srcvalue), (i32 srcvalue), (i32 srcvalue)))))),
              (!cast<Instruction>("extadd_pairwise_"#sign#"_"#suffix) V128:$vec)>;
  }
}
```
- **EN**: Adds declarative TableGen records such as `ExtAddPairwiseExtendShuffle` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `ExtAddPairwiseExtendShuffle`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1596-1602

```tablegen
defm : ExtAddPairwiseExtendShuffle<v8i16, v4i32, "I32x4",
                         0, 1, 4, 5, 8, 9, 12, 13,
                         2, 3, 6, 7, 10, 11, 14, 15>;
defm : ExtAddPairwiseExtendShuffle<v16i8, v8i16, "I16x8",
                         0, 2, 4, 6, 8, 10, 12, 14,
                         1, 3, 5, 7, 9, 11, 13, 15>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1603-1611

```tablegen
multiclass ExtAddPairwiseShuffleExtend<ValueType from_ty, ValueType to_ty, string suffix,
                                       int a0,  int a1,  int a2,  int a3,
                                       int a4,  int a5,  int a6,  int a7,
                                       int a8,  int a9,  int a10, int a11,
                                       int a12, int a13, int a14, int a15,
                                       int b0,  int b1,  int b2,  int b3,
                                       int b4,  int b5,  int b6,  int b7,
                                       int b8,  int b9,  int b10, int b11,
                                       int b12, int b13, int b14, int b15> {
```
- **EN**: Adds declarative TableGen records such as `ExtAddPairwiseShuffleExtend` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `ExtAddPairwiseShuffleExtend`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1612-1631

```tablegen
  foreach sign = ["s", "u"] in {
    def : Pat<(to_ty (add
                      (wasm_shuffle
                        (to_ty (!cast<SDNode>("extend_low_"#sign)  (from_ty V128:$vec))),
                        (to_ty (!cast<SDNode>("extend_high_"#sign) (from_ty V128:$vec))),
                        (i32 a0),  (i32 a1),  (i32 a2),  (i32 a3),
                        (i32 a4),  (i32 a5),  (i32 a6),  (i32 a7),
                        (i32 a8),  (i32 a9),  (i32 a10), (i32 a11),
                        (i32 a12), (i32 a13), (i32 a14), (i32 a15)),
                      (wasm_shuffle
                        (to_ty (!cast<SDNode>("extend_low_"#sign)  (from_ty V128:$vec))),
                        (to_ty (!cast<SDNode>("extend_high_"#sign) (from_ty V128:$vec))),
                        (i32 b0),  (i32 b1),  (i32 b2),  (i32 b3),
                        (i32 b4),  (i32 b5),  (i32 b6),  (i32 b7),
                        (i32 b8),  (i32 b9),  (i32 b10), (i32 b11),
                        (i32 b12), (i32 b13), (i32 b14), (i32 b15)))),
              (!cast<Instruction>("extadd_pairwise_"#sign#"_"#suffix) V128:$vec)>;
  }
}
```
- **EN**: Implements helper routine(s) for this portion of the WebAssembly backend TableGen declarations for the backend.
- **CN**: 这里实现了 WebAssembly 后端该部分该后端的 TableGen 声明所需的辅助例程。

### Lines 1632-1641

```tablegen
defm : ExtAddPairwiseShuffleExtend<v8i16, v4i32, "I32x4",
                                   0,  1,  2,  3,
                                   8,  9,  10, 11,
                                   16, 17, 18, 19,
                                   24, 25, 26, 27,
                                   4,  5,  6,  7,
                                   12, 13, 14, 15,
                                   20, 21, 22, 23,
                                   28, 29, 30, 31>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1642-1652

```tablegen
defm : ExtAddPairwiseShuffleExtend<v16i8, v8i16, "I16x8",
                                   0,  1,  4,  5,
                                   8,  9,  12, 13,
                                   16, 17, 20, 21,
                                   24, 25, 28, 29,
                                   2,  3,  6,  7,
                                   10, 11, 14, 15,
                                   18, 19, 22, 23,
                                   26, 27, 30, 31>;

// f64x2 <-> f32x4 and f32x4 <-> f16x8 conversions
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1653-1659

```tablegen
def demote_t : SDTypeProfile<1, 1, [SDTCisVec<0>, SDTCisVec<1>]>;
def demote_zero : SDNode<"WebAssemblyISD::DEMOTE_ZERO", demote_t>;
defm "" : SIMDConvert<F32x4, F64x2, demote_zero,
                      "demote_f64x2_zero", 0x5e>;
defm "" : HalfPrecisionConvert<F16x8, F32x4, demote_zero, "demote_f32x4_zero", 0x149>;

def promote_t : SDTypeProfile<1, 1, [SDTCisVec<0>, SDTCisVec<1>]>;
```
- **EN**: Adds declarative TableGen records such as `demote_t`, `demote_zero`, `promote_t` that LLVM later expands into generated tables or helper code. This range works in the SelectionDAG-based lowering pipeline.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `demote_t`, `demote_zero`, `promote_t`，LLVM 随后会把它们展开成生成表或辅助代码。 这一段工作在基于 SelectionDAG 的 lowering 流水线中。

### Lines 1660-1666

```tablegen
def promote_low : SDNode<"WebAssemblyISD::PROMOTE_LOW", promote_t>;
defm "" : SIMDConvert<F64x2, F32x4, promote_low, "promote_low_f32x4", 0x5f>;
defm "" : HalfPrecisionConvert<F32x4, I16x8, promote_low, "promote_low_f16x8",
                               0x14b>;

// Lower extending loads to load64_zero + promote_low
def extloadv2f32 : PatFrag<(ops node:$ptr), (extload node:$ptr)> {
```
- **EN**: Adds declarative TableGen records such as `promote_low`, `extloadv2f32` that LLVM later expands into generated tables or helper code. This range works in the SelectionDAG-based lowering pipeline.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `promote_low`, `extloadv2f32`，LLVM 随后会把它们展开成生成表或辅助代码。 这一段工作在基于 SelectionDAG 的 lowering 流水线中。

### Lines 1667-1673

```tablegen
  let MemoryVT = v2f32;
}
// Adapted from the body of LoadPatNoOffset
// TODO: other addressing patterns
def : Pat<(v2f64 (extloadv2f32 (i32 I32:$addr))),
          (promote_low_F64x2 (LOAD_ZERO_64_A32 0, 0, I32:$addr))>,
      Requires<[HasAddr32]>;
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Adapted from the body of LoadPatNoOffset".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Adapted from the body of LoadPatNoOffset”。

### Lines 1674-1677

```tablegen
def : Pat<(v2f64 (extloadv2f32 (i64 I64:$addr))),
          (promote_low_F64x2 (LOAD_ZERO_64_A64 0, 0, I64:$addr))>,
      Requires<[HasAddr64]>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1678-1679

```tablegen
//===----------------------------------------------------------------------===//
// Saturating Rounding Q-Format Multiplication
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Saturating Rounding Q-Format Multiplication".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Saturating Rounding Q-Format Multiplication”。

### Lines 1680-1684

```tablegen
//===----------------------------------------------------------------------===//

defm Q15MULR_SAT_S :
  SIMDBinary<I16x8, int_wasm_q15mulr_sat_signed, "q15mulr_sat_s", 0x82>;
```
- **EN**: Adds declarative TableGen records such as `Q15MULR_SAT_S` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `Q15MULR_SAT_S`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1685-1686

```tablegen
//===----------------------------------------------------------------------===//
// Partial reductions, using: dot, extmul and extadd_pairwise
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Partial reductions, using: dot, extmul and extadd_pairwise".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Partial reductions, using: dot, extmul and extadd_pairwise”。

### Lines 1687-1697

```tablegen
//===----------------------------------------------------------------------===//
// MLA: v8i16 -> v4i32
def : Pat<(v4i32 (partial_reduce_smla (v4i32 V128:$acc), (v8i16 V128:$lhs),
                                                         (v8i16 V128:$rhs))),
          (ADD_I32x4 (DOT $lhs, $rhs), $acc)>;
def : Pat<(v4i32 (partial_reduce_umla (v4i32 V128:$acc), (v8i16 V128:$lhs),
                                                         (v8i16 V128:$rhs))),
          (ADD_I32x4 (ADD_I32x4 (EXTMUL_LOW_U_I32x4 $lhs, $rhs),
                                (EXTMUL_HIGH_U_I32x4 $lhs, $rhs)),
                     $acc)>;
// MLA: v16i8 -> v4i32
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1698-1709

```tablegen
def : Pat<(v4i32 (partial_reduce_smla (v4i32 V128:$acc), (v16i8 V128:$lhs),
                                                         (v16i8 V128:$rhs))),
          (ADD_I32x4 (ADD_I32x4 (extadd_pairwise_s_I32x4 (EXTMUL_LOW_S_I16x8 $lhs, $rhs)),
                                (extadd_pairwise_s_I32x4 (EXTMUL_HIGH_S_I16x8 $lhs, $rhs))),
                     $acc)>;
def : Pat<(v4i32 (partial_reduce_umla (v4i32 V128:$acc), (v16i8 V128:$lhs),
                                                         (v16i8 V128:$rhs))),
          (ADD_I32x4 (ADD_I32x4 (extadd_pairwise_u_I32x4 (EXTMUL_LOW_U_I16x8 $lhs, $rhs)),
                                (extadd_pairwise_u_I32x4 (EXTMUL_HIGH_U_I16x8 $lhs, $rhs))),
                     $acc)>;

// Accumulate: v8i16 -> v4i32
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1710-1718

```tablegen
def : Pat<(v4i32 (partial_reduce_smla (v4i32 V128:$acc), (v8i16 V128:$in),
                                                         (I16x8.splat (i32 1)))),
          (ADD_I32x4 (extadd_pairwise_s_I32x4 $in), $acc)>;

def : Pat<(v4i32 (partial_reduce_umla (v4i32 V128:$acc), (v8i16 V128:$in),
                                                         (I16x8.splat (i32 1)))),
          (ADD_I32x4 (extadd_pairwise_u_I32x4 $in), $acc)>;

// Accumulate: v16i8 -> v4i32
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1719-1727

```tablegen
def : Pat<(v4i32 (partial_reduce_smla (v4i32 V128:$acc), (v16i8 V128:$in),
                                                         (I8x16.splat (i32 1)))),
          (ADD_I32x4 (extadd_pairwise_s_I32x4 (extadd_pairwise_s_I16x8 $in)),
                     $acc)>;
def : Pat<(v4i32 (partial_reduce_umla (v4i32 V128:$acc), (v16i8 V128:$in),
                                                         (I8x16.splat (i32 1)))),
          (ADD_I32x4 (extadd_pairwise_u_I32x4 (extadd_pairwise_u_I16x8 $in)),
                     $acc)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1728-1729

```tablegen
//===----------------------------------------------------------------------===//
// Relaxed swizzle
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Relaxed swizzle".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Relaxed swizzle”。

### Lines 1730-1737

```tablegen
//===----------------------------------------------------------------------===//

defm RELAXED_SWIZZLE :
  RELAXED_I<(outs V128:$dst), (ins V128:$src, V128:$mask), (outs), (ins),
         [(set (v16i8 V128:$dst),
           (int_wasm_relaxed_swizzle (v16i8 V128:$src), (v16i8 V128:$mask)))],
         "i8x16.relaxed_swizzle\t$dst, $src, $mask", "i8x16.relaxed_swizzle", 0x100>;
```
- **EN**: Adds declarative TableGen records such as `RELAXED_SWIZZLE` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `RELAXED_SWIZZLE`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1738-1739

```tablegen
//===----------------------------------------------------------------------===//
// Relaxed floating-point to int conversions
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Relaxed floating-point to int conversions".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Relaxed floating-point to int conversions”。

### Lines 1740-1748

```tablegen
//===----------------------------------------------------------------------===//

multiclass RelaxedConvert<Vec vec, Vec arg, SDPatternOperator op, string name, bits<32> simdop> {
  defm op#_#vec :
    RELAXED_I<(outs V128:$dst), (ins V128:$vec), (outs), (ins),
              [(set (vec.vt V128:$dst), (vec.vt (op (arg.vt V128:$vec))))],
              vec.prefix#"."#name#"\t$dst, $vec", vec.prefix#"."#name, simdop>;
}
```
- **EN**: Adds declarative TableGen records such as `RelaxedConvert` that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `RelaxedConvert`，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 1749-1757

```tablegen
defm "" : RelaxedConvert<I32x4, F32x4, int_wasm_relaxed_trunc_signed,
                         "relaxed_trunc_f32x4_s", 0x101>;
defm "" : RelaxedConvert<I32x4, F32x4, int_wasm_relaxed_trunc_unsigned,
                         "relaxed_trunc_f32x4_u", 0x102>;
defm "" : RelaxedConvert<I32x4, F64x2, int_wasm_relaxed_trunc_signed_zero,
                         "relaxed_trunc_f64x2_s_zero", 0x103>;
defm "" : RelaxedConvert<I32x4, F64x2, int_wasm_relaxed_trunc_unsigned_zero,
                         "relaxed_trunc_f64x2_u_zero", 0x104>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1758-1759

```tablegen
//===----------------------------------------------------------------------===//
// Relaxed (Negative) Multiply-Add  (madd/nmadd)
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Relaxed (Negative) Multiply-Add  (madd/nmadd)".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Relaxed (Negative) Multiply-Add  (madd/nmadd)”。

### Lines 1760-1769

```tablegen
//===----------------------------------------------------------------------===//

multiclass RELAXED_SIMDMADD<Vec vec, bits<32> simdopA, bits<32> simdopS,
                            list<Predicate> reqs> {
  defm MADD_#vec :
    SIMD_I<(outs V128:$dst), (ins V128:$a, V128:$b, V128:$c), (outs), (ins),
           [(set (vec.vt V128:$dst), (int_wasm_relaxed_madd
             (vec.vt V128:$a), (vec.vt V128:$b), (vec.vt V128:$c)))],
           vec.prefix#".relaxed_madd\t$dst, $a, $b, $c",
           vec.prefix#".relaxed_madd", simdopA, reqs>;
```
- **EN**: Adds declarative TableGen records such as `RELAXED_SIMDMADD` that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `RELAXED_SIMDMADD`，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 1770-1776

```tablegen
  defm NMADD_#vec :
    SIMD_I<(outs V128:$dst), (ins V128:$a, V128:$b, V128:$c), (outs), (ins),
           [(set (vec.vt V128:$dst), (int_wasm_relaxed_nmadd
             (vec.vt V128:$a), (vec.vt V128:$b), (vec.vt V128:$c)))],
           vec.prefix#".relaxed_nmadd\t$dst, $a, $b, $c",
           vec.prefix#".relaxed_nmadd", simdopS, reqs>;
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 1777-1783

```tablegen
  def : Pat<(fadd_contract (fmul_contract (vec.vt V128:$a), (vec.vt V128:$b)), (vec.vt V128:$c)),
            (!cast<Instruction>("MADD_"#vec) V128:$a, V128:$b, V128:$c)>, Requires<reqs>;
  def : Pat<(fmuladd (vec.vt V128:$a), (vec.vt V128:$b), (vec.vt V128:$c)),
             (!cast<Instruction>("MADD_"#vec) V128:$a, V128:$b, V128:$c)>, Requires<reqs>;

  def : Pat<(fsub_contract (vec.vt V128:$c), (fmul_contract (vec.vt V128:$a), (vec.vt V128:$b))),
            (!cast<Instruction>("NMADD_"#vec) V128:$a, V128:$b, V128:$c)>, Requires<reqs>;
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 1784-1790

```tablegen
  def : Pat<(fmuladd (fneg (vec.vt V128:$a)), (vec.vt V128:$b), (vec.vt V128:$c)),
             (!cast<Instruction>("NMADD_"#vec) V128:$a, V128:$b, V128:$c)>, Requires<reqs>;
}

defm "" : RELAXED_SIMDMADD<F32x4, 0x105, 0x106, [HasRelaxedSIMD]>;
defm "" : RELAXED_SIMDMADD<F64x2, 0x107, 0x108, [HasRelaxedSIMD]>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1791-1792

```tablegen
//===----------------------------------------------------------------------===//
// FP16 (Negative) Multiply-Add  (madd/nmadd)
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "FP16 (Negative) Multiply-Add  (madd/nmadd)".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“FP16 (Negative) Multiply-Add  (madd/nmadd)”。

### Lines 1793-1802

```tablegen
//===----------------------------------------------------------------------===//

multiclass HALF_PRECISION_SIMDMADD<Vec vec, bits<32> simdopA, bits<32> simdopS,
                                   list<Predicate> reqs> {
  defm MADD_#vec :
    SIMD_I<(outs V128:$dst), (ins V128:$a, V128:$b, V128:$c), (outs), (ins),
           [(set (vec.vt V128:$dst), (fma
             (vec.vt V128:$a), (vec.vt V128:$b), (vec.vt V128:$c)))],
           vec.prefix#".madd\t$dst, $a, $b, $c",
           vec.prefix#".madd", simdopA, reqs>;
```
- **EN**: Adds declarative TableGen records such as `HALF_PRECISION_SIMDMADD` that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `HALF_PRECISION_SIMDMADD`，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 1803-1809

```tablegen
  defm NMADD_#vec :
    SIMD_I<(outs V128:$dst), (ins V128:$a, V128:$b, V128:$c), (outs), (ins),
           [(set (vec.vt V128:$dst), (fma
             (fneg (vec.vt V128:$a)), (vec.vt V128:$b), (vec.vt V128:$c)))],
           vec.prefix#".nmadd\t$dst, $a, $b, $c",
           vec.prefix#".nmadd", simdopS, reqs>;
}
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 1810-1817

```tablegen
defm "" : HALF_PRECISION_SIMDMADD<F16x8, 0x14e, 0x14f, [HasFP16]>;

// TODO: I think separate intrinsics should be introduced for these FP16 operations.
def : Pat<(v8f16 (int_wasm_relaxed_madd (v8f16 V128:$a), (v8f16 V128:$b), (v8f16 V128:$c))),
          (MADD_F16x8 V128:$a, V128:$b, V128:$c)>;
def : Pat<(v8f16 (int_wasm_relaxed_nmadd (v8f16 V128:$a), (v8f16 V128:$b), (v8f16 V128:$c))),
          (NMADD_F16x8 V128:$a, V128:$b, V128:$c)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1818-1819

```tablegen
//===----------------------------------------------------------------------===//
// Laneselect
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Laneselect".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Laneselect”。

### Lines 1820-1830

```tablegen
//===----------------------------------------------------------------------===//

multiclass SIMDLANESELECT<Vec vec, bits<32> op> {
  defm LANESELECT_#vec :
    RELAXED_I<(outs V128:$dst), (ins V128:$a, V128:$b, V128:$c), (outs), (ins),
              [(set (vec.vt V128:$dst), (int_wasm_relaxed_laneselect
                (vec.vt V128:$a), (vec.vt V128:$b), (vec.vt V128:$c)))],
              vec.prefix#".relaxed_laneselect\t$dst, $a, $b, $c",
              vec.prefix#".relaxed_laneselect", op>;
}
```
- **EN**: Adds declarative TableGen records such as `SIMDLANESELECT` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `SIMDLANESELECT`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1831-1835

```tablegen
defm "" : SIMDLANESELECT<I8x16, 0x109>;
defm "" : SIMDLANESELECT<I16x8, 0x10a>;
defm "" : SIMDLANESELECT<I32x4, 0x10b>;
defm "" : SIMDLANESELECT<I64x2, 0x10c>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1836-1837

```tablegen
//===----------------------------------------------------------------------===//
// Relaxed floating-point min and max.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Relaxed floating-point min and max.".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Relaxed floating-point min and max.”。

### Lines 1838-1849

```tablegen
//===----------------------------------------------------------------------===//

multiclass RelaxedBinary<Vec vec, SDPatternOperator node, string name,
                         bits<32> simdop> {
  defm _#vec : RELAXED_I<(outs V128:$dst), (ins V128:$lhs, V128:$rhs),
                         (outs), (ins),
                         [(set (vec.vt V128:$dst),
                           (node (vec.vt V128:$lhs), (vec.vt V128:$rhs)))],
                         vec.prefix#"."#name#"\t$dst, $lhs, $rhs",
                         vec.prefix#"."#name, simdop>;
}
```
- **EN**: Adds declarative TableGen records such as `RelaxedBinary` that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `RelaxedBinary`，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 1850-1858

```tablegen
defm SIMD_RELAXED_FMIN :
   RelaxedBinary<F32x4, int_wasm_relaxed_min, "relaxed_min", 0x10d>;
defm SIMD_RELAXED_FMAX :
   RelaxedBinary<F32x4, int_wasm_relaxed_max, "relaxed_max", 0x10e>;
defm SIMD_RELAXED_FMIN :
   RelaxedBinary<F64x2, int_wasm_relaxed_min, "relaxed_min", 0x10f>;
defm SIMD_RELAXED_FMAX :
   RelaxedBinary<F64x2, int_wasm_relaxed_max, "relaxed_max", 0x110>;
```
- **EN**: Adds declarative TableGen records such as `SIMD_RELAXED_FMIN`, `SIMD_RELAXED_FMAX` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `SIMD_RELAXED_FMIN`, `SIMD_RELAXED_FMAX`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1859-1871

```tablegen
def relaxed_fmin : SDNode<"WebAssemblyISD::RELAXED_FMIN", SDTFPBinOp>;
def relaxed_fmax : SDNode<"WebAssemblyISD::RELAXED_FMAX", SDTFPBinOp>;

def relaxed_pmin :
  PatFrag<(ops node:$lhs, node:$rhs), (pmin $lhs, $rhs), [{
  return (N->getFlags().hasNoNaNs() ||
          (CurDAG->isKnownNeverNaN(N->getOperand(1)) &&
           CurDAG->isKnownNeverNaN(N->getOperand(2)))) &&
         (N->getFlags().hasNoSignedZeros() ||
          CurDAG->isKnownNeverLogicalZero(N->getOperand(1)) ||
          CurDAG->isKnownNeverLogicalZero(N->getOperand(2)));
}]>;
```
- **EN**: Adds declarative TableGen records such as `relaxed_fmin`, `relaxed_fmax`, `relaxed_pmin` that LLVM later expands into generated tables or helper code. This range works in the SelectionDAG-based lowering pipeline.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `relaxed_fmin`, `relaxed_fmax`, `relaxed_pmin`，LLVM 随后会把它们展开成生成表或辅助代码。 这一段工作在基于 SelectionDAG 的 lowering 流水线中。

### Lines 1872-1881

```tablegen
def relaxed_pmax :
  PatFrag<(ops node:$lhs, node:$rhs), (pmax $lhs, $rhs), [{
  return (N->getFlags().hasNoNaNs() ||
          ((CurDAG->isKnownNeverNaN(N->getOperand(1))) &&
           CurDAG->isKnownNeverNaN(N->getOperand(2)))) &&
         (N->getFlags().hasNoSignedZeros() ||
          CurDAG->isKnownNeverLogicalZero(N->getOperand(1)) ||
          CurDAG->isKnownNeverLogicalZero(N->getOperand(2)));
}]>;
```
- **EN**: Adds declarative TableGen records such as `relaxed_pmax` that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `relaxed_pmax`，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 1882-1888

```tablegen
let Predicates = [HasRelaxedSIMD] in {
  foreach vec = [F32x4, F64x2] in {
    defvar relaxed_min = !cast<NI>("SIMD_RELAXED_FMIN_"#vec);
    defvar relaxed_max = !cast<NI>("SIMD_RELAXED_FMAX_"#vec);

    def : Pat<(vec.vt (relaxed_fmin (vec.vt V128:$lhs), (vec.vt V128:$rhs))),
              (relaxed_min V128:$lhs, V128:$rhs)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 1889-1895

```tablegen
    def : Pat<(vec.vt (relaxed_fmax (vec.vt V128:$lhs), (vec.vt V128:$rhs))),
              (relaxed_max V128:$lhs, V128:$rhs)>;

    // Transform pmin/max-supposed patterns to relaxed min max
    let AddedComplexity = 1 in {
      def : Pat<(vec.vt (relaxed_pmin (vec.vt V128:$lhs), (vec.vt V128:$rhs))),
                (relaxed_min $lhs, $rhs)>;
```
- **EN**: Implements helper routine(s) for this portion of the WebAssembly backend TableGen declarations for the backend.
- **CN**: 这里实现了 WebAssembly 后端该部分该后端的 TableGen 声明所需的辅助例程。

### Lines 1896-1902

```tablegen
      def : Pat<(vec.vt (relaxed_pmax (vec.vt V128:$lhs), (vec.vt V128:$rhs))),
                (relaxed_max $lhs, $rhs)>;
      defm : PMinMaxInt<vec, relaxed_min, relaxed_max>;
    }
  }
}
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 1903-1904

```tablegen
//===----------------------------------------------------------------------===//
// Relaxed rounding q15 multiplication
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Relaxed rounding q15 multiplication".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Relaxed rounding q15 multiplication”。

### Lines 1905-1910

```tablegen
//===----------------------------------------------------------------------===//

defm RELAXED_Q15MULR_S :
  RelaxedBinary<I16x8, int_wasm_relaxed_q15mulr_signed, "relaxed_q15mulr_s",
                0x111>;
```
- **EN**: Adds declarative TableGen records such as `RELAXED_Q15MULR_S` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `RELAXED_Q15MULR_S`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1911-1912

```tablegen
//===----------------------------------------------------------------------===//
// Relaxed integer dot product
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Relaxed integer dot product".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Relaxed integer dot product”。

### Lines 1913-1921

```tablegen
//===----------------------------------------------------------------------===//

defm RELAXED_DOT :
  RELAXED_I<(outs V128:$dst), (ins V128:$lhs, V128:$rhs), (outs), (ins),
            [(set (v8i16 V128:$dst), (int_wasm_relaxed_dot_i8x16_i7x16_signed
               (v16i8 V128:$lhs), (v16i8 V128:$rhs)))],
            "i16x8.relaxed_dot_i8x16_i7x16_s\t$dst, $lhs, $rhs",
            "i16x8.relaxed_dot_i8x16_i7x16_s", 0x112>;
```
- **EN**: Adds declarative TableGen records such as `RELAXED_DOT` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `RELAXED_DOT`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1922-1941

```tablegen
def : Pat<
  (v8i16 (add
    (wasm_shuffle
      (v8i16 (extmul_low_s v16i8:$lhs, v16i8:$rhs)),
      (v8i16 (extmul_high_s v16i8:$lhs, v16i8:$rhs)),
      (i32 0), (i32 1), (i32 4), (i32 5),
      (i32 8), (i32 9), (i32 12), (i32 13),
      (i32 16), (i32 17), (i32 20), (i32 21),
      (i32 24), (i32 25), (i32 28), (i32 29)),
    (wasm_shuffle
      (v8i16 (extmul_low_s v16i8:$lhs, v16i8:$rhs)),
      (v8i16 (extmul_high_s v16i8:$lhs, v16i8:$rhs)),
      (i32 2), (i32 3), (i32 6), (i32 7),
      (i32 10), (i32 11), (i32 14), (i32 15),
      (i32 18), (i32 19), (i32 22), (i32 23),
      (i32 26), (i32 27), (i32 30), (i32 31)))
  ),
  (v8i16 (RELAXED_DOT v16i8:$lhs, v16i8:$rhs))
>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1942-1949

```tablegen
defm RELAXED_DOT_ADD :
  RELAXED_I<(outs V128:$dst), (ins V128:$lhs, V128:$rhs, V128:$acc),
            (outs), (ins),
            [(set (v4i32 V128:$dst), (int_wasm_relaxed_dot_i8x16_i7x16_add_signed
               (v16i8 V128:$lhs), (v16i8 V128:$rhs), (v4i32 V128:$acc)))],
            "i32x4.relaxed_dot_i8x16_i7x16_add_s\t$dst, $lhs, $rhs, $acc",
            "i32x4.relaxed_dot_i8x16_i7x16_add_s", 0x113>;
```
- **EN**: Adds declarative TableGen records such as `RELAXED_DOT_ADD` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `RELAXED_DOT_ADD`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1950-1957

```tablegen
def : Pat<
  (v4i32 (add
    (v4i32 (int_wasm_extadd_pairwise_signed
      (v8i16 (int_wasm_relaxed_dot_i8x16_i7x16_signed v16i8:$lhs, v16i8:$rhs)))),
    (v4i32 V128:$acc))),
  (v4i32 (RELAXED_DOT_ADD v16i8:$lhs, v16i8:$rhs, (v4i32 V128:$acc)))
    >;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1958-1961

```tablegen
def : Pat<(v4i32 (partial_reduce_smla (v4i32 V128:$acc), (v16i8 V128:$lhs),
                                                         (v16i8 V128:$rhs))),
          (RELAXED_DOT_ADD $lhs, $rhs, $acc)>, Requires<[HasRelaxedSIMD]>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1962-1963

```tablegen
//===----------------------------------------------------------------------===//
// Relaxed BFloat16 dot product
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Relaxed BFloat16 dot product".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Relaxed BFloat16 dot product”。

### Lines 1964-1972

```tablegen
//===----------------------------------------------------------------------===//

defm RELAXED_DOT_BFLOAT :
  RELAXED_I<(outs V128:$dst), (ins V128:$lhs, V128:$rhs, V128:$acc),
            (outs), (ins),
            [(set (v4f32 V128:$dst), (int_wasm_relaxed_dot_bf16x8_add_f32
               (v8i16 V128:$lhs), (v8i16 V128:$rhs), (v4f32 V128:$acc)))],
            "f32x4.relaxed_dot_bf16x8_add_f32\t$dst, $lhs, $rhs, $acc",
            "f32x4.relaxed_dot_bf16x8_add_f32", 0x114>;
```
- **EN**: Adds declarative TableGen records such as `RELAXED_DOT_BFLOAT` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `RELAXED_DOT_BFLOAT`，LLVM 随后会把它们展开成生成表或辅助代码。

## Key Concepts / 关键概念

- WebAssembly backend integration / WebAssembly 后端集成
- TableGen declarations for the backend / 该后端的 TableGen 声明
- LLVM TableGen DSL / LLVM TableGen 领域特定语言
- Instruction selection or opcode handling / 指令选择或操作码处理
- Declarative TableGen records / 声明式 TableGen 记录
- Vector or SIMD behavior / 向量或 SIMD 行为
- Exception-handling support / 异常处理支持

## Dependencies / 依赖关系

### Direct Includes / 直接包含

- None explicitly included in the source text / 源码文本中没有显式包含项

### Important Collaborators / 重要协作组件

- Generated backend tables / 生成式后端表
- WebAssembly target-specific helpers / WebAssembly 目标专用辅助组件
