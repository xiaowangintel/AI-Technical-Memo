# SPIRVInstrInfo.td — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/SPIRV/SPIRVInstrInfo.td`
- Repository: `llvm-project`
- Purpose (EN): This file describes the SPIR-V instructions in TableGen format.
- 目的（中文）: 该文件使用 TableGen DSL 描述目标后端元数据，并驱动自动生成代码。
- Language: TableGen DSL (TableGen DSL noted below)
- Note: This file is written in TableGen DSL, so many records are declarative descriptions consumed by LLVM code generators.
- 说明：该文件使用 TableGen DSL 编写，因此许多记录都是供 LLVM 代码生成器消费的声明式描述。

## Line-by-Line Analysis / 逐行分析

> Note / 说明: To keep the document readable, the source is analyzed in contiguous line ranges while preserving the original order. Each code block prefixes original line numbers.

### Lines 1-90
```tablegen
 1: //===-- SPIRVInstrInfo.td - Target Description for SPIR-V Target ----------===//
 2: //
 3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
 4: // See https://llvm.org/LICENSE.txt for license information.
 5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
 6: //
 7: //===----------------------------------------------------------------------===//
 8: //
 9: // This file describes the SPIR-V instructions in TableGen format.
10: //
11: //===----------------------------------------------------------------------===//
12:
13: include "SPIRVInstrFormats.td"
14: include "SPIRVSymbolicOperands.td"
15:
16: // Codegen only metadata instructions
17: let isCodeGenOnly=1 in {
18:   def ASSIGN_TYPE: Pseudo<(outs ID:$dst_id), (ins ID:$src_id, TYPE:$src_ty)>;
19: }
20:
21: def UNKNOWN_type
22:     : UnknownOp<(outs TYPE:$type), (ins i32imm:$opcode, variable_ops), " ">;
23:
24: def SPVTypeBin : SDTypeProfile<1, 2, []>;
25:
26: def assigntype : SDNode<"SPIRVISD::AssignType", SPVTypeBin>;
27:
28: def : GINodeEquiv<ASSIGN_TYPE, assigntype>;
29:
30: class BinOp<string name, bits<16> opCode, list<dag> pattern=[]>
31:                 : Op<opCode, (outs ID:$dst), (ins TYPE:$src_ty, ID:$src, ID:$src2),
32:                   "$dst = "#name#" $src_ty $src $src2", pattern>;
33:
34: class BinOpTyped<string name, bits<16> opCode, RegisterClass CID, SDNode node>
35:                 : Op<opCode, (outs CID:$dst), (ins TYPE:$src_ty, CID:$src, CID:$src2),
36:                   "$dst = "#name#" $src_ty $src $src2",
37:                   [(set CID:$dst, (assigntype (node CID:$src, CID:$src2), TYPE:$src_ty))]>;
38:
39: class TernOpTyped<string name, bits<16> opCode, RegisterClass CCond, RegisterClass CID, SDNode node>
40:                 : Op<opCode, (outs CID:$dst), (ins TYPE:$src_ty, CCond:$cond, CID:$src1, CID:$src2),
41:                   "$dst = "#name#" $src_ty $cond $src1 $src2",
42:                   [(set CID:$dst, (assigntype (node CCond:$cond, CID:$src1, CID:$src2), TYPE:$src_ty))]>;
43:
44: multiclass BinOpTypedGen<string name, bits<16> opCode, SDNode node, bit genF = 0, bit genV = 0> {
45:   if genF then
46:     def S: BinOpTyped<name, opCode, fID, node>;
47:   else
48:     def S: BinOpTyped<name, opCode, iID, node>;
49:   if genV then {
50:     if genF then
51:       def V: BinOpTyped<name, opCode, vfID, node>;
52:     else
53:       def V: BinOpTyped<name, opCode, viID, node>;
54:   }
55: }
56:
57: multiclass TernOpTypedGen<string name, bits<16> opCode, SDNode node, bit genP = 1, bit genI = 1, bit genF = 0, bit genV = 0> {
58:   if genP then {
59:     def SPSCond: TernOpTyped<name, opCode, iID, pID, node>;
60:     def SPVCond: TernOpTyped<name, opCode, viID, pID, node>;
61:   }
62:   if genI then {
63:     def SISCond: TernOpTyped<name, opCode, iID, iID, node>;
64:     def SIVCond: TernOpTyped<name, opCode, viID, iID, node>;
65:   }
66:   if genF then {
67:     def SFSCond: TernOpTyped<name, opCode, iID, fID, node>;
68:     def SFVCond: TernOpTyped<name, opCode, viID, fID, node>;
69:   }
70:   if genV then {
71:     if genP then {
72:       def VPSCond: TernOpTyped<name, opCode, iID, vpID, node>;
73:       def VPVCond: TernOpTyped<name, opCode, viID, vpID, node>;
74:     }
75:     if genI then {
76:       def VISCond: TernOpTyped<name, opCode, iID, viID, node>;
77:       def VIVCond: TernOpTyped<name, opCode, viID, viID, node>;
78:     }
79:     if genF then {
80:       def VFSCond: TernOpTyped<name, opCode, iID, vfID, node>;
81:       def VFVCond: TernOpTyped<name, opCode, viID, vfID, node>;
82:     }
83:   }
84: }
85:
86: class UnOp<string name, bits<16> opCode, list<dag> pattern=[]>
87:                 : Op<opCode, (outs ID:$dst), (ins TYPE:$type, ID:$src),
88:                   "$dst = "#name#" $type $src", pattern>;
89: class UnOpTyped<string name, bits<16> opCode, RegisterClass CID, SDNode node>
90:                 : Op<opCode, (outs CID:$dst), (ins TYPE:$src_ty, CID:$src),
```
- EN: This range uses TableGen DSL to describe records such as ASSIGN_TYPE, UNKNOWN_type, SPVTypeBin, assigntype; LLVM later expands these declarations into generated backend tables and helper code.
- CN: 这一段使用 TableGen DSL 描述 ASSIGN_TYPE、UNKNOWN_type、SPVTypeBin、assigntype 等记录；LLVM 后续会把这些声明展开为生成的后端表和辅助代码。

### Lines 91-180
```tablegen
 91:                   "$dst = "#name#" $src_ty $src", [(set CID:$dst, (assigntype (node CID:$src), TYPE:$src_ty))]>;
 92:
 93: class SimpleOp<string name, bits<16> opCode>: Op<opCode, (outs), (ins), name>;
 94:
 95: // 3.42.1 Miscellaneous Instructions
 96:
 97: def OpNop: SimpleOp<"OpNop", 0>;
 98: def OpUndef: Op<1, (outs ID:$res), (ins TYPE:$type), "$res = OpUndef $type">;
 99: def OpSizeOf: Op<321, (outs ID:$res), (ins TYPE:$ty, ID:$ptr), "$res = OpSizeOf $ty $ptr">;
100:
101: //  - SPV_KHR_expect_assume : Expect assume instructions
102: def OpAssumeTrueKHR: Op<5630, (outs), (ins ID:$cond), "OpAssumeTrueKHR $cond">;
103: def OpExpectKHR: Op<5631, (outs ID:$res), (ins TYPE:$ty, ID:$val, ID:$expected), "$res = OpExpectKHR $ty $val $expected">;
104:
105: // 3.42.2 Debug Instructions
106:
107: def OpSourceContinued: Op<2, (outs), (ins StringImm:$str, variable_ops),
108:                   "OpSourceContinued $str">;
109: def OpSource: Op<3, (outs), (ins SourceLanguage:$lang, i32imm:$version, variable_ops),
110:                   "OpSource $lang $version">;
111: def OpSourceExtension: Op<4, (outs), (ins StringImm:$extension, variable_ops),
112:                   "OpSourceExtension $extension">;
113: def OpName: Op<5, (outs), (ins ANY:$tar, StringImm:$name, variable_ops), "OpName $tar $name">;
114: def OpMemberName: Op<6, (outs), (ins TYPE:$ty, i32imm:$mem, StringImm:$name, variable_ops),
115:                   "OpMemberName $ty $mem $name">;
116: def OpString: Op<7, (outs ID:$r), (ins StringImm:$s, variable_ops), "$r = OpString $s">;
117: def OpLine: Op<8, (outs), (ins ID:$file, i32imm:$ln, i32imm:$col), "OpLine $file $ln $col">;
118: def OpNoLine: Op<317, (outs), (ins), "OpNoLine">;
119: def OpModuleProcessed: Op<330, (outs), (ins StringImm:$process, variable_ops),
120:                   "OpModuleProcessed $process">;
121:
122: // 3.42.3 Annotation Instructions
123:
124: def OpDecorate: Op<71, (outs), (ins ANY:$target, Decoration:$dec, variable_ops),
125:                   "OpDecorate $target $dec">;
126: def OpMemberDecorate: Op<72, (outs), (ins TYPE:$t, i32imm:$m, Decoration:$d, variable_ops),
127:                   "OpMemberDecorate $t $m $d">;
128:
129: // TODO Currently some deprecated opcodes are missing: OpDecorationGroup,
130: // OpGroupDecorate and OpGroupMemberDecorate
131:
132: def OpDecorateId: Op<332, (outs), (ins ANY:$target, Decoration:$dec, variable_ops),
133:                   "OpDecorateId $target $dec">;
134: def OpDecorateString: Op<5632, (outs), (ins ANY:$t, Decoration:$d, StringImm:$s, variable_ops),
135:                   "OpDecorateString $t $d $s">;
136: def OpMemberDecorateString: Op<5633, (outs),
137:                   (ins TYPE:$ty, i32imm:$mem, Decoration:$dec, StringImm:$str, variable_ops),
138:                   "OpMemberDecorateString $ty $mem $dec $str">;
139:
140: // 3.42.4 Extension Instructions
141:
142: def OpExtension: Op<10, (outs), (ins StringImm:$name, variable_ops), "OpExtension $name">;
143: def OpExtInstImport: Op<11, (outs ID:$res), (ins StringImm:$extInstsName, variable_ops),
144:                   "$res = OpExtInstImport $extInstsName">;
145: // $set should have been a register by the SPIR-V specification,
146: // however, OpExtInst and OpExtInstImport get its own special case treatment
147: // after instruction selection, so `i32imm` is the correct definition from the
148: // perspective of the instruction selection pass
149: def OpExtInst: Op<12, (outs ID:$res), (ins TYPE:$ty, i32imm:$set, Extension:$inst, variable_ops),
150:                   "$res = OpExtInst $ty $set $inst">;
151: // 3.42.5 Mode-Setting Instructions
152:
153: def OpMemoryModel: Op<14, (outs), (ins AddressingModel:$addr, MemoryModel:$mem),
154:                   "OpMemoryModel $addr $mem">;
155: def OpEntryPoint: Op<15, (outs),
156:                   (ins ExecutionModel:$model, ID:$entry, StringImm:$name, variable_ops),
157:                   "OpEntryPoint $model $entry $name">;
158: def OpExecutionMode: Op<16, (outs), (ins ID:$entry, ExecutionMode:$mode, variable_ops),
159:                   "OpExecutionMode $entry $mode">;
160: def OpCapability: Op<17, (outs), (ins Capability:$cap), "OpCapability $cap">;
161: def OpExecutionModeId: Op<331, (outs), (ins ID:$entry, ExecutionMode:$mode, variable_ops),
162:                   "OpExecutionModeId $entry $mode">;
163:
164: // 3.42.6 Type-Declaration Instructions
165:
166: def OpTypeVoid: Op<19, (outs TYPE:$type), (ins), "$type = OpTypeVoid">;
167: def OpTypeBool: Op<20, (outs TYPE:$type), (ins), "$type = OpTypeBool">;
168: def OpTypeInt: Op<21, (outs TYPE:$type), (ins i32imm:$width, i32imm:$signedness),
169:                   "$type = OpTypeInt $width $signedness">;
170: def OpTypeFloat: Op<22, (outs TYPE:$type), (ins i32imm:$width, variable_ops),
171:                   "$type = OpTypeFloat $width">;
172: def OpTypeVector: Op<23, (outs TYPE:$type), (ins TYPE:$compType, i32imm:$compCount),
173:                   "$type = OpTypeVector $compType $compCount">;
174: def OpTypeMatrix: Op<24, (outs TYPE:$type), (ins TYPE:$colType, i32imm:$colCount),
175:                   "$type = OpTypeMatrix $colType $colCount">;
176: def OpTypeImage: Op<25, (outs TYPE:$res), (ins TYPE:$sampTy, Dim:$dim, i32imm:$depth,
177:       i32imm:$arrayed, i32imm:$MS, i32imm:$sampled, ImageFormat:$imFormat, variable_ops),
178:                   "$res = OpTypeImage $sampTy $dim $depth $arrayed $MS $sampled $imFormat">;
179: def OpTypeSampler: Op<26, (outs TYPE:$res), (ins), "$res = OpTypeSampler">;
180: def OpTypeSampledImage: Op<27, (outs TYPE:$res), (ins TYPE:$imageType),
```
- EN: This range uses TableGen DSL to describe records such as SimpleOp, OpNop, OpUndef, OpSizeOf; LLVM later expands these declarations into generated backend tables and helper code.
- CN: 这一段使用 TableGen DSL 描述 SimpleOp、OpNop、OpUndef、OpSizeOf 等记录；LLVM 后续会把这些声明展开为生成的后端表和辅助代码。

### Lines 181-270
```tablegen
181:                   "$res = OpTypeSampledImage $imageType">;
182: def OpTypeArray: Op<28, (outs TYPE:$type), (ins TYPE:$elementType, ID:$length),
183:                   "$type = OpTypeArray $elementType $length">;
184: def OpTypeRuntimeArray: Op<29, (outs TYPE:$type), (ins TYPE:$elementType),
185:                   "$type = OpTypeRuntimeArray $elementType">;
186: def OpTypeStruct: Op<30, (outs TYPE:$res), (ins variable_ops), "$res = OpTypeStruct">;
187: def OpTypeStructContinuedINTEL: Op<6090, (outs), (ins variable_ops),
188:                   "OpTypeStructContinuedINTEL">;
189: def OpTypeOpaque: Op<31, (outs TYPE:$res), (ins StringImm:$name, variable_ops),
190:                   "$res = OpTypeOpaque $name">;
191: def OpTypePointer: Op<32, (outs TYPE:$res), (ins StorageClass:$storage, TYPE:$type),
192:                   "$res = OpTypePointer $storage $type">;
193: def OpTypeFunction: Op<33, (outs TYPE:$funcType), (ins TYPE:$returnType, variable_ops),
194:                   "$funcType = OpTypeFunction $returnType">;
195: def OpTypeEvent: Op<34, (outs TYPE:$res), (ins), "$res = OpTypeEvent">;
196: def OpTypeDeviceEvent: Op<35, (outs TYPE:$res), (ins), "$res = OpTypeDeviceEvent">;
197: def OpTypeReserveId: Op<36, (outs TYPE:$res), (ins), "$res = OpTypeReserveId">;
198: def OpTypeQueue: Op<37, (outs TYPE:$res), (ins), "$res = OpTypeQueue">;
199: def OpTypePipe: Op<38, (outs TYPE:$res), (ins AccessQualifier:$a), "$res = OpTypePipe $a">;
200: def OpTypeForwardPointer: Op<39, (outs), (ins TYPE:$ptrType, StorageClass:$storageClass),
201:                   "OpTypeForwardPointer $ptrType $storageClass">;
202: def OpTypePipeStorage: Op<322, (outs TYPE:$res), (ins), "$res = OpTypePipeStorage">;
203: def OpTypeNamedBarrier: Op<327, (outs TYPE:$res), (ins), "$res = OpTypeNamedBarrier">;
204: def OpTypeAccelerationStructureNV: Op<5341, (outs TYPE:$res), (ins),
205:                   "$res = OpTypeAccelerationStructureNV">;
206: def OpTypeCooperativeMatrixNV: Op<5358, (outs TYPE:$res),
207:                   (ins TYPE:$compType, ID:$scope, ID:$rows, ID:$cols),
208:                   "$res = OpTypeCooperativeMatrixNV $compType $scope $rows $cols">;
209: def OpTypeCooperativeMatrixKHR: Op<4456, (outs TYPE:$res),
210:                   (ins TYPE:$compType, ID:$scope, ID:$rows, ID:$cols, ID:$use),
211:                   "$res = OpTypeCooperativeMatrixKHR $compType $scope $rows $cols $use">;
212:
213: // 3.42.7 Constant-Creation Instructions
214:
215: multiclass IntFPImm<bits<16> opCode, string name> {
216:   def I: Op<opCode, (outs iID:$dst), (ins TYPE:$type, iID:$src, variable_ops),
217:                   "$dst = "#name#" $type">;
218:   def F: Op<opCode, (outs fID:$dst), (ins TYPE:$type, fID:$src, variable_ops),
219:                   "$dst = "#name#" $type">;
220: }
221: defm OpConstant: IntFPImm<43, "OpConstant">;
222:
223: def ConstPseudoTrue: IntImmLeaf<i64, [{ return Imm.getBitWidth() == 1 && Imm.getZExtValue() == 1; }]>;
224: def ConstPseudoFalse: IntImmLeaf<i64, [{ return Imm.getBitWidth() == 1 && Imm.getZExtValue() == 0; }]>;
225: def OpConstantTrue: Op<41, (outs iID:$dst), (ins TYPE:$src_ty), "$dst = OpConstantTrue $src_ty",
226:                       [(set iID:$dst, (assigntype ConstPseudoTrue, TYPE:$src_ty))]>;
227: def OpConstantFalse: Op<42, (outs iID:$dst), (ins TYPE:$src_ty), "$dst = OpConstantFalse $src_ty",
228:                       [(set iID:$dst, (assigntype ConstPseudoFalse, TYPE:$src_ty))]>;
229:
230: def OpConstantComposite: Op<44, (outs ID:$res), (ins TYPE:$type, variable_ops),
231:                   "$res = OpConstantComposite $type">;
232: def OpConstantCompositeContinuedINTEL: Op<6091, (outs), (ins variable_ops),
233:                   "OpConstantCompositeContinuedINTEL">;
234:
235: def OpConstantSampler: Op<45, (outs ID:$res),
236:                   (ins TYPE:$t, SamplerAddressingMode:$s, i32imm:$p, SamplerFilterMode:$f),
237:                   "$res = OpConstantSampler $t $s $p $f">;
238: def OpConstantNull: Op<46, (outs ID:$dst), (ins TYPE:$src_ty), "$dst = OpConstantNull $src_ty">;
239:
240: def OpSpecConstantTrue: Op<48, (outs ID:$r), (ins TYPE:$t), "$r = OpSpecConstantTrue $t">;
241: def OpSpecConstantFalse: Op<49, (outs ID:$r), (ins TYPE:$t), "$r = OpSpecConstantFalse $t">;
242: def OpSpecConstant: Op<50, (outs ID:$res), (ins TYPE:$type, i32imm:$imm, variable_ops),
243:                   "$res = OpSpecConstant $type $imm">;
244: def OpSpecConstantComposite: Op<51, (outs ID:$res), (ins TYPE:$type, variable_ops),
245:                   "$res = OpSpecConstantComposite $type">;
246: def OpSpecConstantCompositeContinuedINTEL: Op<6092, (outs), (ins variable_ops),
247:                   "OpSpecConstantCompositeContinuedINTEL">;
248: def OpSpecConstantOp: Op<52, (outs ID:$res), (ins TYPE:$t, SpecConstantOpOperands:$c, ID:$o, variable_ops),
249:                   "$res = OpSpecConstantOp $t $c $o">;
250:
251: // 3.42.8 Memory Instructions
252:
253: def OpVariable: Op<59, (outs ID:$res), (ins TYPE:$type, StorageClass:$sc, variable_ops),
254:                   "$res = OpVariable $type $sc">;
255: def OpImageTexelPointer: Op<60, (outs ID:$res),
256:                   (ins TYPE:$resType, ID:$image, ID:$coord, ID:$sample),
257:                   "$res = OpImageTexelPointer $resType $image $coord $sample">;
258: def OpLoad: Op<61, (outs ID:$res), (ins TYPE:$resType, ID:$pointer, variable_ops),
259:                   "$res = OpLoad $resType $pointer">;
260: def OpStore: Op<62, (outs), (ins ID:$pointer, ID:$objectToStore, variable_ops),
261:                   "OpStore $pointer $objectToStore">;
262: def OpCopyMemory: Op<63, (outs), (ins ID:$dest, ID:$src, variable_ops),
263:                   "OpCopyMemory $dest $src">;
264: def OpCopyMemorySized: Op<64, (outs), (ins ID:$dest, ID:$src, ID:$size, variable_ops),
265:                   "OpCopyMemorySized $dest $src $size">;
266: def OpAccessChain: Op<65, (outs ID:$res), (ins TYPE:$type, ID:$base, variable_ops),
267:                   "$res = OpAccessChain $type $base">;
268: def OpInBoundsAccessChain: Op<66, (outs ID:$res),
269:                   (ins TYPE:$type, ID:$base, variable_ops),
270:                   "$res = OpInBoundsAccessChain $type $base">;
```
- EN: This range uses TableGen DSL to describe records such as OpTypeArray, OpTypeRuntimeArray, OpTypeStruct, OpTypeStructContinuedINTEL; LLVM later expands these declarations into generated backend tables and helper code.
- CN: 这一段使用 TableGen DSL 描述 OpTypeArray、OpTypeRuntimeArray、OpTypeStruct、OpTypeStructContinuedINTEL 等记录；LLVM 后续会把这些声明展开为生成的后端表和辅助代码。

### Lines 271-360
```tablegen
271: def OpPtrAccessChain: Op<67, (outs ID:$res),
272:                   (ins TYPE:$type, ID:$base, ID:$element, variable_ops),
273:                   "$res = OpPtrAccessChain $type $base $element">;
274: def OpArrayLength: Op<68, (outs ID:$res), (ins TYPE:$resTy, ID:$struct, i32imm:$arrayMember),
275:                   "$res = OpArrayLength $resTy $struct $arrayMember">;
276: def OpGenericPtrMemSemantics: Op<69, (outs ID:$res), (ins TYPE:$resType, ID:$pointer),
277:                   "$res = OpGenericPtrMemSemantics $resType $pointer">;
278: def OpInBoundsPtrAccessChain: Op<70, (outs ID:$res),
279:                   (ins TYPE:$type, ID:$base, ID:$element, variable_ops),
280:                   "$res = OpInBoundsPtrAccessChain $type $base $element">;
281: def OpPtrEqual: Op<401, (outs ID:$res), (ins TYPE:$resType, ID:$a, ID:$b),
282:                   "$res = OpPtrEqual $resType $a $b">;
283: def OpPtrNotEqual: Op<402, (outs ID:$res), (ins TYPE:$resType, ID:$a, ID:$b),
284:                   "$res = OpPtrNotEqual $resType $a $b">;
285: def OpPtrDiff: Op<403, (outs ID:$res), (ins TYPE:$resType, ID:$a, ID:$b),
286:                   "$res = OpPtrDiff $resType $a $b">;
287:
288: // - SPV_INTEL_variable_length_array
289:
290: def OpVariableLengthArrayINTEL: Op<5818, (outs ID:$res), (ins TYPE:$type, ID:$length),
291:                   "$res = OpVariableLengthArrayINTEL $type $length">;
292: def OpSaveMemoryINTEL: Op<5819, (outs ID:$res), (ins TYPE:$type),
293:                   "$res = OpSaveMemoryINTEL $type">;
294: def OpRestoreMemoryINTEL: Op<5820, (outs), (ins ID:$ptr),
295:                   "OpRestoreMemoryINTEL $ptr">;
296:
297: // 3.42.9 Function Instructions
298:
299: def OpFunction: Op<54, (outs ID:$func),
300:                   (ins TYPE:$resType, FunctionControl:$funcControl, TYPE:$funcType),
301:                   "$func = OpFunction $resType $funcControl $funcType">;
302: def OpFunctionParameter: Op<55, (outs ID:$arg), (ins TYPE:$type),
303:                   "$arg = OpFunctionParameter $type">;
304: def OpFunctionEnd: Op<56, (outs), (ins), "OpFunctionEnd"> {
305:   let isTerminator=1;
306: }
307: def OpFunctionCall: Op<57, (outs ID:$res), (ins TYPE:$resType, ID:$function, variable_ops),
308:                   "$res = OpFunctionCall $resType $function">;
309:
310: // 3.42.10 Image Instructions
311:
312: def OpSampledImage: BinOp<"OpSampledImage", 86>;
313:
314: def OpImageSampleImplicitLod: Op<87, (outs ID:$res),
315:                   (ins TYPE:$type, ID:$sampledImage, ID:$coord, variable_ops),
316:                   "$res = OpImageSampleImplicitLod $type $sampledImage $coord">;
317: def OpImageSampleExplicitLod: Op<88, (outs ID:$res),
318:                   (ins TYPE:$ty, ID:$sImage, ID:$uv, ImageOperand:$op, ID:$i, variable_ops),
319:                   "$res = OpImageSampleExplicitLod $ty $sImage $uv $op $i">;
320:
321: def OpImageSampleDrefImplicitLod: Op<89, (outs ID:$res),
322:                   (ins TYPE:$type, ID:$sampledImage, ID:$coord, ID:$dref, variable_ops),
323:                   "$res = OpImageSampleDrefImplicitLod $type $sampledImage $coord $dref">;
324: def OpImageSampleDrefExplicitLod: Op<90, (outs ID:$res),
325:                   (ins TYPE:$ty, ID:$im, ID:$uv, ID:$d, ImageOperand:$op, ID:$i, variable_ops),
326:                   "$res = OpImageSampleDrefExplicitLod $ty $im $uv $d $op $i">;
327:
328: def OpImageSampleProjImplicitLod: Op<91, (outs ID:$res),
329:                   (ins TYPE:$type, ID:$sampledImage, ID:$coord, variable_ops),
330:                   "$res = OpImageSampleProjImplicitLod $type $sampledImage $coord">;
331: def OpImageSampleProjExplicitLod: Op<92, (outs ID:$res),
332:                   (ins TYPE:$ty, ID:$im, ID:$uv, ID:$d, ImageOperand:$op, ID:$i, variable_ops),
333:                   "$res = OpImageSampleProjExplicitLod $ty $im $uv $op $i">;
334:
335: def OpImageSampleProjDrefImplicitLod: Op<93, (outs ID:$res),
336:                   (ins TYPE:$type, ID:$sampledImage, ID:$coord, ID:$dref, variable_ops),
337:                   "$res = OpImageSampleProjDrefImplicitLod $type $sampledImage $coord $dref">;
338: def OpImageSampleProjDrefExplicitLod: Op<94, (outs ID:$res),
339:                   (ins TYPE:$ty, ID:$im, ID:$uv, ID:$d, ImageOperand:$op, ID:$i, variable_ops),
340:                   "$res = OpImageSampleProjDrefExplicitLod $ty $im $uv $d $op $i">;
341:
342: def OpImageFetch: Op<95, (outs ID:$res),
343:                   (ins TYPE:$type, ID:$image, ID:$coord, variable_ops),
344:                   "$res = OpImageFetch $type $image $coord">;
345: def OpImageGather: Op<96, (outs ID:$res),
346:                   (ins TYPE:$type, ID:$sampledImage, ID:$coord, ID:$component, variable_ops),
347:                   "$res = OpImageGather $type $sampledImage $coord $component">;
348: def OpImageDrefGather: Op<97, (outs ID:$res),
349:                   (ins TYPE:$type, ID:$sampledImage, ID:$coord, ID:$dref, variable_ops),
350:                   "$res = OpImageDrefGather $type $sampledImage $coord $dref">;
351:
352: def OpImageRead: Op<98, (outs ID:$res),
353:                   (ins TYPE:$type, ID:$image, ID:$coord, variable_ops),
354:                   "$res = OpImageRead $type $image $coord">;
355: def OpImageWrite: Op<99, (outs), (ins ID:$image, ID:$coord, ID:$texel, variable_ops),
356:                   "OpImageWrite $image $coord $texel">;
357:
358: def OpImage: UnOp<"OpImage", 100>;
359: def OpImageQueryFormat: UnOp<"OpImageQueryFormat", 101>;
360: def OpImageQueryOrder: UnOp<"OpImageQueryOrder", 102>;
```
- EN: This range uses TableGen DSL to describe records such as OpPtrAccessChain, OpArrayLength, OpGenericPtrMemSemantics, OpInBoundsPtrAccessChain; LLVM later expands these declarations into generated backend tables and helper code.
- CN: 这一段使用 TableGen DSL 描述 OpPtrAccessChain、OpArrayLength、OpGenericPtrMemSemantics、OpInBoundsPtrAccessChain 等记录；LLVM 后续会把这些声明展开为生成的后端表和辅助代码。

### Lines 361-450
```tablegen
361: def OpImageQuerySizeLod: BinOp<"OpImageQuerySizeLod", 103>;
362: def OpImageQuerySize: UnOp<"OpImageQuerySize", 104>;
363: def OpImageQueryLod: BinOp<"OpImageQueryLod", 105>;
364: def OpImageQueryLevels: UnOp<"OpImageQueryLevels", 106>;
365: def OpImageQuerySamples: UnOp<"OpImageQuerySamples", 107>;
366:
367: def OpImageSparseSampleImplicitLod: Op<305, (outs ID:$res),
368:                   (ins TYPE:$type, ID:$sampledImage, ID:$coord, variable_ops),
369:                   "$res = OpImageSparseSampleImplicitLod $type $sampledImage $coord">;
370: def OpImageSparseSampleExplicitLod: Op<306, (outs ID:$res),
371:                   (ins TYPE:$ty, ID:$sImage, ID:$uv, ImageOperand:$op, ID:$i, variable_ops),
372:                   "$res = OpImageSparseSampleExplicitLod $ty $sImage $uv $op $i">;
373:
374: def OpImageSparseSampleDrefImplicitLod: Op<307, (outs ID:$res),
375:                   (ins TYPE:$type, ID:$sampledImg, ID:$coord, ID:$dref, variable_ops),
376:                   "$res = OpImageSparseSampleDrefImplicitLod $type $sampledImg $coord $dref">;
377: def OpImageSparseSampleDrefExplicitLod: Op<308, (outs ID:$res),
378:                   (ins TYPE:$ty, ID:$im, ID:$uv, ID:$d, ImageOperand:$op, ID:$i, variable_ops),
379:                   "$res = OpImageSparseSampleDrefExplicitLod $ty $im $uv $d $op $i">;
380:
381: def OpImageSparseSampleProjImplicitLod: Op<309, (outs ID:$res),
382:                   (ins TYPE:$type, ID:$sampledImage, ID:$coord, variable_ops),
383:                   "$res = OpImageSparseSampleProjImplicitLod $type $sampledImage $coord">;
384: def OpImageSparseSampleProjExplicitLod: Op<310, (outs ID:$res),
385:                   (ins TYPE:$ty, ID:$im, ID:$uv, ID:$d, ImageOperand:$op, ID:$i, variable_ops),
386:                   "$res = OpImageSparseSampleProjExplicitLod $ty $im $uv $op $i">;
387:
388: def OpImageSparseSampleProjDrefImplicitLod: Op<311, (outs ID:$res),
389:                   (ins TYPE:$type, ID:$sImage, ID:$coord, ID:$dref, variable_ops),
390:                   "$res = OpImageSparseSampleProjDrefImplicitLod $type $sImage $coord $dref">;
391: def OpImageSparseSampleProjDrefExplicitLod: Op<312, (outs ID:$res),
392:                   (ins TYPE:$ty, ID:$im, ID:$uv, ID:$d, ImageOperand:$op, ID:$i, variable_ops),
393:                   "$res = OpImageSparseSampleProjDrefExplicitLod $ty $im $uv $d $op $i">;
394:
395: def OpImageSparseFetch: Op<313, (outs ID:$res),
396:                   (ins TYPE:$type, ID:$image, ID:$coord, variable_ops),
397:                   "$res = OpImageSparseFetch $type $image $coord">;
398: def OpImageSparseGather: Op<314, (outs ID:$res),
399:                   (ins TYPE:$type, ID:$sampledImage, ID:$coord, ID:$component, variable_ops),
400:                   "$res = OpImageSparseGather $type $sampledImage $coord $component">;
401: def OpImageSparseDrefGather: Op<315, (outs ID:$res),
402:                   (ins TYPE:$type, ID:$sampledImage, ID:$coord, ID:$dref, variable_ops),
403:                   "$res = OpImageSparseDrefGather $type $sampledImage $coord $dref">;
404:
405: def OpImageSparseTexelsResident: UnOp<"OpImageSparseTexelsResident", 316>;
406:
407: def OpImageSparseRead: Op<320, (outs ID:$res),
408:                   (ins TYPE:$type, ID:$image, ID:$coord, variable_ops),
409:                   "$res = OpImageSparseRead $type $image $coord">;
410:
411: def OpImageSampleFootprintNV: Op<5283, (outs ID:$res),
412:                   (ins TYPE:$ty, ID:$sImg, ID:$uv, ID:$granularity, ID:$coarse, variable_ops),
413:                   "$res = OpImageSampleFootprintNV $ty $sImg $uv $granularity $coarse">;
414:
415: // 3.42.11 Conversion instructions
416:
417: def OpConvertFToU : UnOp<"OpConvertFToU", 109>;
418: def OpConvertFToS : UnOp<"OpConvertFToS", 110>;
419: def OpConvertSToF : UnOp<"OpConvertSToF", 111>;
420: def OpConvertUToF : UnOp<"OpConvertUToF", 112>;
421:
422: def OpUConvert : UnOp<"OpUConvert", 113>;
423: def OpSConvert : UnOp<"OpSConvert", 114>;
424: def OpFConvert : UnOp<"OpFConvert", 115>;
425:
426: def OpQuantizeToF16 : UnOp<"OpQuantizeToF16", 116>;
427:
428: def OpConvertPtrToU : UnOp<"OpConvertPtrToU", 117>;
429:
430: def OpSatConvertSToU : UnOp<"OpSatConvertSToU", 118>;
431: def OpSatConvertUToS : UnOp<"OpSatConvertUToS", 119>;
432:
433: def OpConvertUToPtr : UnOp<"OpConvertUToPtr", 120>;
434: def OpPtrCastToGeneric : UnOp<"OpPtrCastToGeneric", 121>;
435: def OpGenericCastToPtr : UnOp<"OpGenericCastToPtr", 122>;
436: def OpGenericCastToPtrExplicit : Op<123, (outs ID:$r), (ins TYPE:$t, ID:$p, StorageClass:$s),
437:                               "$r = OpGenericCastToPtrExplicit $t $p $s">;
438: def OpBitcast : UnOp<"OpBitcast", 124>;
439:
440: // SPV_INTEL_usm_storage_classes
441: def OpPtrCastToCrossWorkgroupINTEL : UnOp<"OpPtrCastToCrossWorkgroupINTEL", 5934>;
442: def OpCrossWorkgroupCastToPtrINTEL : UnOp<"OpCrossWorkgroupCastToPtrINTEL", 5938>;
443:
444: // SPV_INTEL_bfloat16_conversion
445: def OpConvertFToBF16INTEL : UnOp<"OpConvertFToBF16INTEL", 6116>;
446: def OpConvertBF16ToFINTEL : UnOp<"OpConvertBF16ToFINTEL", 6117>;
447:
448: // SPV_INTEL_tensor_float32_conversion
449: def OpRoundFToTF32INTEL : UnOp<"OpRoundFToTF32INTEL", 6426>;
450:
```
- EN: This range uses TableGen DSL to describe records such as OpImageQuerySizeLod, OpImageQuerySize, OpImageQueryLod, OpImageQueryLevels; LLVM later expands these declarations into generated backend tables and helper code.
- CN: 这一段使用 TableGen DSL 描述 OpImageQuerySizeLod、OpImageQuerySize、OpImageQueryLod、OpImageQueryLevels 等记录；LLVM 后续会把这些声明展开为生成的后端表和辅助代码。

### Lines 451-540
```tablegen
451: // 3.42.12 Composite Instructions
452:
453: def OpVectorExtractDynamic: Op<77, (outs ID:$res), (ins TYPE:$type, ID:$vec, ID:$idx),
454:                   "$res = OpVectorExtractDynamic $type $vec $idx">;
455: def OpVectorInsertDynamic: Op<78, (outs ID:$res), (ins TYPE:$ty, ID:$vec, ID:$comp, ID:$idx),
456:                   "$res = OpVectorInsertDynamic $ty $vec $comp $idx">;
457: def OpVectorShuffle: Op<79, (outs ID:$res), (ins TYPE:$ty, ID:$v1, ID:$v2, variable_ops),
458:                   "$res = OpVectorShuffle $ty $v1 $v2">;
459: def OpCompositeConstruct: Op<80, (outs ID:$res), (ins TYPE:$type, variable_ops),
460:                   "$res = OpCompositeConstruct $type">;
461: def OpCompositeConstructContinuedINTEL: Op<6096, (outs), (ins variable_ops),
462:                   "OpCompositeConstructContinuedINTEL">;
463: def OpCompositeExtract: Op<81, (outs ID:$res), (ins TYPE:$type, ID:$base, variable_ops),
464:                   "$res = OpCompositeExtract $type $base">;
465: def OpCompositeInsert: Op<82, (outs ID:$r), (ins TYPE:$ty, ID:$obj, ID:$base, variable_ops),
466:                   "$r = OpCompositeInsert $ty $obj $base">;
467: def OpCopyObject: UnOp<"OpCopyObject", 83>;
468: def OpTranspose: UnOp<"OpTranspose", 84>;
469: def OpCopyLogical: UnOp<"OpCopyLogical", 400>;
470:
471: // 3.42.13 Arithmetic Instructions
472:
473: def OpSNegate: UnOp<"OpSNegate", 126>;
474: def OpFNegate: UnOpTyped<"OpFNegate", 127, fID, fneg>;
475: def OpFNegateV: UnOpTyped<"OpFNegate", 127, vfID, fneg>;
476: defm OpIAdd: BinOpTypedGen<"OpIAdd", 128, add, 0, 1>;
477: defm OpFAdd: BinOpTypedGen<"OpFAdd", 129, fadd, 1, 1>;
478: defm OpStrictFAdd: BinOpTypedGen<"OpFAdd", 129, strict_fadd, 1, 1>;
479:
480: defm OpISub: BinOpTypedGen<"OpISub", 130, sub, 0, 1>;
481: defm OpFSub: BinOpTypedGen<"OpFSub", 131, fsub, 1, 1>;
482: defm OpStrictFSub: BinOpTypedGen<"OpFSub", 131, strict_fsub, 1, 1>;
483:
484: defm OpIMul: BinOpTypedGen<"OpIMul", 132, mul, 0, 1>;
485: defm OpFMul: BinOpTypedGen<"OpFMul", 133, fmul, 1, 1>;
486: defm OpStrictFMul: BinOpTypedGen<"OpFMul", 133, strict_fmul, 1, 1>;
487:
488: defm OpUDiv: BinOpTypedGen<"OpUDiv", 134, udiv, 0, 1>;
489: defm OpSDiv: BinOpTypedGen<"OpSDiv", 135, sdiv, 0, 1>;
490: defm OpFDiv: BinOpTypedGen<"OpFDiv", 136, fdiv, 1, 1>;
491: defm OpStrictFDiv: BinOpTypedGen<"OpFDiv", 136, strict_fdiv, 1, 1>;
492:
493: defm OpUMod: BinOpTypedGen<"OpUMod", 137, urem, 0, 1>;
494: defm OpSRem: BinOpTypedGen<"OpSRem", 138, srem, 0, 1>;
495:
496: def OpSMod: BinOp<"OpSMod", 139>;
497:
498: defm OpFRem: BinOpTypedGen<"OpFRem", 140, frem, 1, 1>;
499: defm OpStrictFRem: BinOpTypedGen<"OpFRem", 140, strict_frem, 1, 1>;
500:
501: def OpFMod: BinOp<"OpFMod", 141>;
502:
503: def OpVectorTimesScalar: BinOp<"OpVectorTimesScalar", 142>;
504: def OpMatrixTimesScalar: BinOp<"OpMatrixTimesScalar", 143>;
505: def OpVectorTimesMatrix: BinOp<"OpVectorTimesMatrix", 144>;
506: def OpMatrixTimesVector: BinOp<"OpMatrixTimesVector", 145>;
507: def OpMatrixTimesMatrix: BinOp<"OpMatrixTimesMatrix", 146>;
508:
509: def OpOuterProduct: BinOp<"OpOuterProduct", 147>;
510: def OpDot: BinOp<"OpDot", 148>;
511:
512: defm OpIAddCarry: BinOpTypedGen<"OpIAddCarry", 149, addc, 0, 1>;
513: defm OpISubBorrow: BinOpTypedGen<"OpISubBorrow", 150, subc, 0, 1>;
514: def OpUMulExtended: BinOp<"OpUMulExtended", 151>;
515: def OpSMulExtended: BinOp<"OpSMulExtended", 152>;
516:
517: def OpSDot: Op<4450, (outs ID:$res), (ins TYPE:$type, ID:$vec1, ID:$vec2, variable_ops),
518:                   "$res = OpSDot $type $vec1 $vec2">;
519: def OpUDot: Op<4451, (outs ID:$res), (ins TYPE:$type, ID:$vec1, ID:$vec2, variable_ops),
520:                   "$res = OpUDot $type $vec1 $vec2">;
521: def OpSUDot: Op<4452, (outs ID:$res), (ins TYPE:$type, ID:$vec1, ID:$vec2, variable_ops),
522:                   "$res = OpSUDot $type $vec1 $vec2">;
523: def OpSDotAccSat: Op<4453, (outs ID:$res), (ins TYPE:$type, ID:$vec1, ID:$vec2, ID:$acc, variable_ops),
524:                   "$res = OpSDotAccSat $type $vec1 $vec2 $acc">;
525: def OpUDotAccSat: Op<4454, (outs ID:$res), (ins TYPE:$type, ID:$vec1, ID:$vec2, ID:$acc, variable_ops),
526:                   "$res = OpUDotAccSat $type $vec1 $vec2 $acc">;
527: def OpSUDotAccSat: Op<4455, (outs ID:$res), (ins TYPE:$type, ID:$vec1, ID:$vec2, ID:$acc, variable_ops),
528:                   "$res = OpSUDotAccSat $type $vec1 $vec2 $acc">;
529:
530: def OpFmaKHR: Op<4427, (outs ID:$res), (ins TYPE:$type, ID:$a, ID:$b, ID:$c),
531:                   "$res = OpFmaKHR $type $a $b $c">;
532:
533: // 3.42.14 Bit Instructions
534:
535: defm OpShiftRightLogical: BinOpTypedGen<"OpShiftRightLogical", 194, srl, 0, 1>;
536: defm OpShiftRightArithmetic: BinOpTypedGen<"OpShiftRightArithmetic", 195, sra, 0, 1>;
537: defm OpShiftLeftLogical: BinOpTypedGen<"OpShiftLeftLogical", 196, shl, 0, 1>;
538:
539: defm OpBitwiseOr: BinOpTypedGen<"OpBitwiseOr", 197, or, 0, 1>;
540: defm OpBitwiseXor: BinOpTypedGen<"OpBitwiseXor", 198, xor, 0, 1>;
```
- EN: This range uses TableGen DSL to describe records such as OpVectorExtractDynamic, OpVectorInsertDynamic, OpVectorShuffle, OpCompositeConstruct; LLVM later expands these declarations into generated backend tables and helper code.
- CN: 这一段使用 TableGen DSL 描述 OpVectorExtractDynamic、OpVectorInsertDynamic、OpVectorShuffle、OpCompositeConstruct 等记录；LLVM 后续会把这些声明展开为生成的后端表和辅助代码。

### Lines 541-630
```tablegen
541: defm OpBitwiseAnd: BinOpTypedGen<"OpBitwiseAnd", 199, and, 0, 1>;
542: def OpNot: UnOp<"OpNot", 200>;
543:
544: def OpBitFieldInsert: Op<201, (outs ID:$res),
545:                   (ins TYPE:$ty, ID:$base, ID:$insert, ID:$offset, ID:$count),
546:                   "$res = OpBitFieldInsert $ty $base $insert $offset $count">;
547: def OpBitFieldSExtract: Op<202, (outs ID:$res),
548:                   (ins TYPE:$ty, ID:$base, ID:$offset, ID:$count),
549:                   "$res = OpBitFieldSExtract $ty $base $offset $count">;
550: def OpBitFieldUExtract: Op<203, (outs ID:$res),
551:                   (ins TYPE:$ty, ID:$base, ID:$offset, ID:$count),
552:                   "$res = OpBitFieldUExtract $ty $base $offset $count">;
553: def OpBitReverse: Op<204, (outs ID:$r), (ins TYPE:$ty, ID:$b), "$r = OpBitReverse $ty $b">;
554: def OpBitCount: Op<205, (outs ID:$r), (ins TYPE:$ty, ID:$b), "$r = OpBitCount $ty $b">;
555:
556: // 3.42.15 Relational and Logical Instructions
557:
558: def OpAny: Op<154, (outs ID:$res), (ins TYPE:$ty, ID:$vec),
559:                   "$res = OpAny $ty $vec">;
560: def OpAll: Op<155, (outs ID:$res), (ins TYPE:$ty, ID:$vec),
561:                   "$res = OpAll $ty $vec">;
562:
563: def OpIsNan: UnOp<"OpIsNan", 156>;
564: def OpIsInf: UnOp<"OpIsInf", 157>;
565: def OpIsFinite: UnOp<"OpIsFinite", 158>;
566: def OpIsNormal: UnOp<"OpIsNormal", 159>;
567: def OpSignBitSet: UnOp<"OpSignBitSet", 160>;
568:
569: def OpLessOrGreater: BinOp<"OpLessOrGreater", 161>;
570: def OpOrdered: BinOp<"OpOrdered", 162>;
571: def OpUnordered: BinOp<"OpUnordered", 163>;
572:
573: def OpLogicalEqual: BinOp<"OpLogicalEqual", 164>;
574: def OpLogicalNotEqual: BinOp<"OpLogicalNotEqual", 165>;
575: def OpLogicalOr: BinOp<"OpLogicalOr", 166>;
576: def OpLogicalAnd: BinOp<"OpLogicalAnd", 167>;
577: def OpLogicalNot: UnOp<"OpLogicalNot", 168>;
578:
579: defm OpSelect: TernOpTypedGen<"OpSelect", 169, select, 1, 1, 1, 1>;
580:
581: def OpIEqual: BinOp<"OpIEqual", 170>;
582: def OpINotEqual: BinOp<"OpINotEqual", 171>;
583:
584: def OpUGreaterThan: BinOp<"OpUGreaterThan", 172>;
585: def OpSGreaterThan: BinOp<"OpSGreaterThan", 173>;
586: def OpUGreaterThanEqual: BinOp<"OpUGreaterThanEqual", 174>;
587: def OpSGreaterThanEqual: BinOp<"OpSGreaterThanEqual", 175>;
588: def OpULessThan: BinOp<"OpULessThan", 176>;
589: def OpSLessThan: BinOp<"OpSLessThan", 177>;
590: def OpULessThanEqual: BinOp<"OpULessThanEqual", 178>;
591: def OpSLessThanEqual: BinOp<"OpSLessThanEqual", 179>;
592:
593: def OpFOrdEqual: BinOp<"OpFOrdEqual", 180>;
594: def OpFUnordEqual: BinOp<"OpFUnordEqual", 181>;
595: def OpFOrdNotEqual: BinOp<"OpFOrdNotEqual", 182>;
596: def OpFUnordNotEqual: BinOp<"OpFUnordNotEqual", 183>;
597:
598: def OpFOrdLessThan: BinOp<"OpFOrdLessThan", 184>;
599: def OpFUnordLessThan: BinOp<"OpFUnordLessThan", 185>;
600: def OpFOrdGreaterThan: BinOp<"OpFOrdGreaterThan", 186>;
601: def OpFUnordGreaterThan: BinOp<"OpFUnordGreaterThan", 187>;
602:
603: def OpFOrdLessThanEqual: BinOp<"OpFOrdLessThanEqual", 188>;
604: def OpFUnordLessThanEqual: BinOp<"OpFUnordLessThanEqual", 189>;
605: def OpFOrdGreaterThanEqual: BinOp<"OpFOrdGreaterThanEqual", 190>;
606: def OpFUnordGreaterThanEqual: BinOp<"OpFUnordGreaterThanEqual", 191>;
607:
608: // 3.42.16 Derivative Instructions
609:
610: def OpDPdx: UnOp<"OpDPdx", 207>;
611: def OpDPdy: UnOp<"OpDPdy", 208>;
612: def OpFwidth: UnOp<"OpFwidth", 209>;
613:
614: def OpDPdxFine: UnOp<"OpDPdxFine", 210>;
615: def OpDPdyFine: UnOp<"OpDPdyFine", 211>;
616: def OpFwidthFine: UnOp<"OpFwidthFine", 212>;
617:
618: def OpDPdxCoarse: UnOp<"OpDPdxCoarse", 213>;
619: def OpDPdyCoarse: UnOp<"OpDPdyCoarse", 214>;
620: def OpFwidthCoarse: UnOp<"OpFwidthCoarse", 215>;
621:
622: // 3.42.17 Control-Flow Instructions
623:
624: def OpPhi: Op<245, (outs ID:$res), (ins TYPE:$type, ID:$var0, ID:$block0, variable_ops),
625:                   "$res = OpPhi $type $var0 $block0">;
626: def OpLoopMerge: Op<246, (outs), (ins unknown:$merge, unknown:$continue, LoopControl:$lc, variable_ops),
627:                   "OpLoopMerge $merge $continue $lc">;
628: def OpLoopControlINTEL: Op<5887, (outs), (ins LoopControl:$lc, variable_ops),
629:                   "OpLoopControlINTEL $lc">;
630: def OpSelectionMerge: Op<247, (outs), (ins unknown:$merge, SelectionControl:$sc),
```
- EN: This range uses TableGen DSL to describe records such as OpBitwiseAnd, OpNot, OpBitFieldInsert, OpBitFieldSExtract; LLVM later expands these declarations into generated backend tables and helper code.
- CN: 这一段使用 TableGen DSL 描述 OpBitwiseAnd、OpNot、OpBitFieldInsert、OpBitFieldSExtract 等记录；LLVM 后续会把这些声明展开为生成的后端表和辅助代码。

### Lines 631-720
```tablegen
631:                   "OpSelectionMerge $merge $sc">;
632: def OpLabel: Op<248, (outs ID:$label), (ins), "$label = OpLabel">;
633: let isBarrier = 1, isTerminator = 1, isBranch = 1 in {
634:   def OpBranch: Op<249, (outs), (ins unknown:$label), "OpBranch $label">;
635:   def OpBranchConditional: Op<250, (outs), (ins ID:$cond, unknown:$true, unknown:$false, variable_ops),
636:                   "OpBranchConditional $cond $true $false">;
637:   def OpSwitch: Op<251, (outs), (ins ID:$sel, ID:$dflt, variable_ops), "OpSwitch $sel $dflt">;
638: }
639: let isReturn = 1, hasDelaySlot = 0, isBarrier = 0, isTerminator = 1, isNotDuplicable = 1 in {
640:   def OpKill: SimpleOp<"OpKill", 252>;
641:   def OpReturn: SimpleOp<"OpReturn", 253>;
642:   def OpReturnValue: Op<254, (outs), (ins ID:$ret), "OpReturnValue $ret">;
643:   def OpUnreachable: SimpleOp<"OpUnreachable", 255>;
644:   def OpAbortKHR: Op<5121, (outs), (ins TYPE:$msg_type, ID:$msg),
645:                   "OpAbortKHR $msg_type $msg">;
646: }
647: def OpLifetimeStart: Op<256, (outs), (ins ID:$ptr, i32imm:$sz), "OpLifetimeStart $ptr $sz">;
648: def OpLifetimeStop: Op<257, (outs), (ins ID:$ptr, i32imm:$sz), "OpLifetimeStop $ptr $sz">;
649: def OpDemoteToHelperInvocation: SimpleOp<"OpDemoteToHelperInvocation", 5380>;
650:
651: // 3.42.18 Atomic Instructions
652:
653: class AtomicOp<string name, bits<16> opCode>: Op<opCode, (outs ID:$res),
654:                   (ins TYPE:$ty, ID:$ptr, ID:$sc, ID:$sem),
655:                   "$res = "#name#" $ty $ptr $sc $sem">;
656:
657: class AtomicOpVal<string name, bits<16> opCode>: Op<opCode, (outs ID:$res),
658:                   (ins TYPE:$ty, ID:$ptr, ID:$sc, ID:$sem, ID:$val),
659:                   "$res = "#name#" $ty $ptr $sc $sem $val">;
660:
661: def OpAtomicLoad: AtomicOp<"OpAtomicLoad", 227>;
662:
663: def OpAtomicStore: Op<228, (outs), (ins ID:$ptr, ID:$sc, ID:$sem, ID:$val),
664:                   "OpAtomicStore $ptr $sc $sem $val">;
665: def OpAtomicExchange: Op<229, (outs ID:$res),
666:                   (ins TYPE:$ty, ID:$ptr, ID:$sc, ID:$sem, ID:$val),
667:                   "$res = OpAtomicExchange $ty $ptr $sc $sem $val">;
668: def OpAtomicCompareExchange: Op<230, (outs ID:$res),
669:                   (ins TYPE:$ty, ID:$ptr, ID:$sc, ID:$eq,
670:                    ID:$neq, ID:$val, ID:$cmp),
671:                   "$res = OpAtomicCompareExchange $ty $ptr $sc $eq $neq $val $cmp">;
672: def OpAtomicCompareExchangeWeak: Op<231, (outs ID:$res),
673:                    (ins TYPE:$ty, ID:$ptr, ID:$sc, ID:$eq,
674:                     ID:$neq, ID:$val, ID:$cmp),
675:                    "$res = OpAtomicCompareExchangeWeak $ty $ptr $sc $eq $neq $val $cmp">;
676:
677: def OpAtomicIIncrement: AtomicOp<"OpAtomicIIncrement", 232>;
678: def OpAtomicIDecrement: AtomicOp<"OpAtomicIDecrement", 233>;
679:
680: def OpAtomicIAdd: AtomicOpVal<"OpAtomicIAdd", 234>;
681: def OpAtomicISub: AtomicOpVal<"OpAtomicISub", 235>;
682:
683: def OpAtomicSMin: AtomicOpVal<"OpAtomicSMin", 236>;
684: def OpAtomicUMin: AtomicOpVal<"OpAtomicUMin", 237>;
685: def OpAtomicSMax: AtomicOpVal<"OpAtomicSMax", 238>;
686: def OpAtomicUMax: AtomicOpVal<"OpAtomicUMax", 239>;
687:
688: def OpAtomicAnd: AtomicOpVal<"OpAtomicAnd", 240>;
689: def OpAtomicOr: AtomicOpVal<"OpAtomicOr", 241>;
690: def OpAtomicXor: AtomicOpVal<"OpAtomicXor", 242>;
691:
692: def OpAtomicFAddEXT: AtomicOpVal<"OpAtomicFAddEXT", 6035>;
693: def OpAtomicFMinEXT: AtomicOpVal<"OpAtomicFMinEXT", 5614>;
694: def OpAtomicFMaxEXT: AtomicOpVal<"OpAtomicFMaxEXT", 5615>;
695:
696: def OpAtomicFlagTestAndSet: AtomicOp<"OpAtomicFlagTestAndSet", 318>;
697: def OpAtomicFlagClear: Op<319, (outs), (ins ID:$ptr, ID:$sc, ID:$sem),
698:                   "OpAtomicFlagClear $ptr $sc $sem">;
699:
700: // 3.42.19 Primitive Instructions
701:
702: def OpEmitVertex: SimpleOp<"OpEmitVertex", 218>;
703: def OpEndPrimitive: SimpleOp<"OpEndPrimitive", 219>;
704: def OpEmitStreamVertex: Op<220, (outs), (ins ID:$stream), "OpEmitStreamVertex $stream">;
705: def OpEndStreamPrimitive: Op<221, (outs), (ins ID:$stream), "OpEndStreamPrimitive $stream">;
706:
707: // 3.42.20 Barrier Instructions
708:
709: def OpControlBarrier: Op<224, (outs), (ins ID:$exec, ID:$mem, ID:$sem),
710:                   "OpControlBarrier $exec $mem $sem">;
711: def OpMemoryBarrier: Op<225, (outs), (ins ID:$mem, ID:$sem),
712:                   "OpMemoryBarrier $mem $sem">;
713: def OpNamedBarrierInitialize: UnOp<"OpNamedBarrierInitialize", 328>;
714: def OpMemoryNamedBarrier: Op<329, (outs), (ins ID:$barr, ID:$mem, ID:$sem),
715:                   "OpMemoryNamedBarrier $barr $mem $sem">;
716:
717: // SPV_INTEL_split_barrier
718: def OpControlBarrierArriveINTEL: Op<6142, (outs), (ins ID:$exec, ID:$mem, ID:$sem),
719:                   "OpControlBarrierArriveINTEL $exec $mem $sem">;
720: def OpControlBarrierWaitINTEL: Op<6143, (outs), (ins ID:$exec, ID:$mem, ID:$sem),
```
- EN: This range uses TableGen DSL to describe records such as OpLabel, OpBranch, OpBranchConditional, OpSwitch; LLVM later expands these declarations into generated backend tables and helper code.
- CN: 这一段使用 TableGen DSL 描述 OpLabel、OpBranch、OpBranchConditional、OpSwitch 等记录；LLVM 后续会把这些声明展开为生成的后端表和辅助代码。

### Lines 721-810
```tablegen
721:                   "OpControlBarrierWaitINTEL $exec $mem $sem">;
722:
723: // 3.42.21. Group and Subgroup Instructions
724:
725: def OpGroupAsyncCopy: Op<259, (outs ID:$res), (ins TYPE:$ty, ID:$scope,
726:                   ID:$dst, ID:$src, ID:$nelts, ID:$stride, ID:$event),
727:                   "$res = OpGroupAsyncCopy $ty $scope $dst $src $nelts $stride $event">;
728: def OpGroupWaitEvents: Op<260, (outs), (ins ID:$scope, ID:$nelts, ID:$elist),
729:                   "OpGroupWaitEvents $scope $nelts $elist">;
730: def OpGroupAll: Op<261, (outs ID:$res), (ins TYPE:$ty, ID:$scope, ID:$pr),
731:                   "$res = OpGroupAll $ty $scope $pr">;
732: def OpGroupAny: Op<262, (outs ID:$res), (ins TYPE:$ty, ID:$scope, ID:$pr),
733:                   "$res = OpGroupAny $ty $scope $pr">;
734: def OpGroupBroadcast: Op<263, (outs ID:$res), (ins TYPE:$ty, ID:$scope,
735:                                ID:$val, ID:$id),
736:                   "$res = OpGroupBroadcast $ty $scope $val $id">;
737: class OpGroup<string name, bits<16> opCode>: Op<opCode, (outs ID:$res),
738:                   (ins TYPE:$ty, ID:$scope, GroupOperation:$groupOp, ID:$x),
739:                   "$res = OpGroup"#name#" $ty $scope $groupOp $x">;
740: def OpGroupIAdd: OpGroup<"IAdd", 264>;
741: def OpGroupFAdd: OpGroup<"FAdd", 265>;
742: def OpGroupFMin: OpGroup<"FMin", 266>;
743: def OpGroupUMin: OpGroup<"UMin", 267>;
744: def OpGroupSMin: OpGroup<"SMin", 268>;
745: def OpGroupFMax: OpGroup<"FMax", 269>;
746: def OpGroupUMax: OpGroup<"UMax", 270>;
747: def OpGroupSMax: OpGroup<"SMax", 271>;
748:
749: def OpSubgroupMatrixMultiplyAccumulateINTEL: Op<6237, (outs ID:$res),
750:         (ins TYPE:$ty, ID:$KDim, ID:$A, ID:$B, ID:$C, variable_ops),
751:                   "$res = OpSubgroupMatrixMultiplyAccumulateINTEL $ty $KDim $A $B $C">;
752:
753: // TODO: 3.42.22. Device-Side Enqueue Instructions
754: def OpEnqueueKernel: Op<292, (outs ID:$res), (ins TYPE:$type, ID:$queue, ID:$flags, ID:$NDR, ID:$nevents, ID:$wevents,
755:                                               ID:$revent, ID:$invoke, ID:$param, ID:$psize, ID:$palign, variable_ops),
756:                   "$res = OpEnqueueKernel $type $queue $flags $NDR $nevents $wevents $revent $invoke $param $psize $palign">;
757: def OpRetainEvent: Op<297, (outs), (ins ID:$event), "OpRetainEvent $event">;
758: def OpReleaseEvent: Op<298, (outs), (ins ID:$event), "OpReleaseEvent $event">;
759: def OpCreateUserEvent: Op<299, (outs ID:$res), (ins TYPE:$type),
760:                   "$res = OpCreateUserEvent $type">;
761: def OpIsValidEvent: Op<300, (outs ID:$res), (ins TYPE:$type, ID:$event),
762:                   "$res = OpIsValidEvent $type $event ">;
763: def OpSetUserEventStatus: Op<301, (outs), (ins ID:$event, ID:$status),
764:                   "OpSetUserEventStatus $event $status">;
765: def OpCaptureEventProfilingInfo: Op<302, (outs),
766:                   (ins ID:$event, ID:$info, ID:$value),
767:                   "OpCaptureEventProfilingInfo $event $info $value">;
768: def OpGetDefaultQueue: Op<303, (outs ID:$res), (ins TYPE:$type),
769:                   "$res = OpGetDefaultQueue $type">;
770: def OpBuildNDRange: Op<304, (outs ID:$res), (ins TYPE:$type, ID:$GWS, ID:$LWS, ID:$GWO),
771:                   "$res = OpBuildNDRange $type $GWS $LWS $GWO">;
772:
773: // 3.42.23. Pipe Instructions
774:
775: def OpReadPipe: Op<274, (outs ID:$res), (ins TYPE:$type, ID:$Pipe, ID:$Pointer, ID:$PcktSize, ID:$PcktAlign),
776:                   "$res = OpReadPipe $type $Pipe $Pointer $PcktSize $PcktAlign">;
777: def OpWritePipe: Op<275, (outs ID:$res), (ins TYPE:$type, ID:$Pipe, ID:$Pointer, ID:$PcktSize, ID:$PcktAlign),
778:                   "$res = OpWritePipe $type $Pipe $Pointer $PcktSize $PcktAlign">;
779: def OpReservedReadPipe : Op<276, (outs ID:$res), (ins TYPE:$type, ID:$Pipe, ID:$ReserveId, ID:$Index, ID:$Pointer, ID:$PcktSize, ID:$PcktAlign),
780:                   "$res = OpReservedReadPipe $type $Pipe $ReserveId $Index $Pointer $PcktSize $PcktAlign">;
781: def OpReservedWritePipe : Op<277, (outs ID:$res), (ins TYPE:$type, ID:$Pipe, ID:$ReserveId, ID:$Index, ID:$Pointer, ID:$PcktSize, ID:$PcktAlign), 
782:                   "$res = OpReservedWritePipe $type $Pipe $ReserveId $Index $Pointer $PcktSize $PcktAlign">;
783: def OpReserveReadPipePackets : Op<278, (outs ID:$res), (ins TYPE:$type, ID:$Pipe, ID:$NumPckts, ID:$PcktSize, ID:$PcktAlign),
784:                   "$res = OpReserveReadPipePackets $type $Pipe $NumPckts $PcktSize $PcktAlign">;
785: def OpReserveWritePipePackets : Op<279, (outs ID:$res), (ins TYPE:$type, ID:$Pipe, ID:$NumPckts, ID:$PcktSize, ID:$PcktAlign),
786:                   "$res = OpReserveWritePipePackets $type $Pipe $NumPckts $PcktSize $PcktAlign">;
787: def OpCommitReadPipe : Op<280, (outs), (ins ID:$Pipe, ID:$ReserveId, ID:$PcktSize, ID:$PcktAlign),
788:                   "OpCommitReadPipe $Pipe $ReserveId $PcktSize $PcktAlign">;
789: def OpCommitWritePipe : Op<281, (outs), (ins ID:$Pipe, ID:$ReserveId, ID:$PcktSize, ID:$PcktAlign),
790:                   "OpCommitWritePipe $Pipe $ReserveId $PcktSize $PcktAlign">;
791: def OpIsValidReserveId : Op<282, (outs ID:$res), (ins TYPE:$type, ID:$ReserveId),
792:                   "$res = OpIsValidReserveId $type $ReserveId">;
793: def OpGetNumPipePackets : Op<283, (outs ID:$res), (ins TYPE:$type, ID:$Pipe, ID:$PacketSize, ID:$PacketAlign),
794:                   "$res = OpGetNumPipePackets $type $Pipe $PacketSize $PacketAlign">;
795: def OpGetMaxPipePackets : Op<284, (outs ID:$res), (ins TYPE:$type, ID:$Pipe, ID:$PacketSize, ID:$PacketAlign),
796:                   "$res = OpGetMaxPipePackets $type $Pipe $PacketSize $PacketAlign">;
797: def OpGroupReserveReadPipePackets : Op<285, (outs ID:$res), (ins TYPE:$type, ID:$Scope, ID:$Pipe, ID:$NumPckts, ID:$PacketSize, ID:$PacketAlign),
798:                   "$res = OpGroupReserveReadPipePackets $type $Scope $Pipe $NumPckts $PacketSize $PacketAlign">;
799: def OpGroupReserveWritePipePackets : Op<286, (outs ID:$res), (ins TYPE:$type, ID:$Scope, ID:$Pipe, ID:$NumPckts, ID:$PacketSize, ID:$PacketAlign),
800:                   "$res = OpGroupReserveWritePipePackets $type $Scope $Pipe $NumPckts $PacketSize $PacketAlign">;
801: def OpGroupCommitReadPipe : Op<287, (outs), (ins ID:$Scope, ID:$Pipe, ID:$ReserveId, ID:$PacketSize, ID:$PacketAlign),
802:                   "OpGroupCommitReadPipe $Scope $Pipe $ReserveId $PacketSize $PacketAlign">;
803: def OpGroupCommitWritePipe : Op<288, (outs), (ins ID:$Scope, ID:$Pipe, ID:$ReserveId, ID:$PacketSize, ID:$PacketAlign),
804:                   "OpGroupCommitWritePipe $Scope $Pipe $ReserveId $PacketSize $PacketAlign">;
805:
806: // 3.42.24. Non-Uniform Instructions
807:
808: def OpGroupNonUniformElect: Op<333, (outs ID:$res), (ins TYPE:$ty, ID:$scope),
809:                   "$res = OpGroupNonUniformElect $ty $scope">;
810: class OpGroupNU3<string name, bits<16> opCode>: Op<opCode,
```
- EN: This range uses TableGen DSL to describe records such as OpGroupAsyncCopy, OpGroupWaitEvents, OpGroupAll, OpGroupAny; LLVM later expands these declarations into generated backend tables and helper code.
- CN: 这一段使用 TableGen DSL 描述 OpGroupAsyncCopy、OpGroupWaitEvents、OpGroupAll、OpGroupAny 等记录；LLVM 后续会把这些声明展开为生成的后端表和辅助代码。

### Lines 811-900
```tablegen
811:                   (outs ID:$res), (ins TYPE:$ty, ID:$scope, ID:$pred),
812:                   "$res = OpGroupNonUniform"#name#" $ty $scope $pred">;
813: class OpGroupNU4<string name, bits<16> opCode>: Op<opCode,
814:                   (outs ID:$res), (ins TYPE:$ty, ID:$scope, ID:$val, ID:$id),
815:                   "$res = OpGroupNonUniform"#name#" $ty $scope $val $id">;
816: def OpGroupNonUniformAll: OpGroupNU3<"All", 334>;
817: def OpGroupNonUniformAny: OpGroupNU3<"Any", 335>;
818: def OpGroupNonUniformAllEqual: OpGroupNU3<"AllEqual", 336>;
819: def OpGroupNonUniformBroadcast: OpGroupNU4<"Broadcast", 337>;
820: def OpGroupNonUniformBroadcastFirst: OpGroupNU3<"BroadcastFirst", 338>;
821: def OpGroupNonUniformBallot: OpGroupNU3<"Ballot", 339>;
822: def OpGroupNonUniformInverseBallot: OpGroupNU3<"InverseBallot", 340>;
823: def OpGroupNonUniformBallotBitExtract: OpGroupNU4<"BallotBitExtract", 341>;
824: def OpGroupNonUniformBallotBitCount: Op<342, (outs ID:$res),
825:                   (ins TYPE:$ty, ID:$scope, GroupOperation:$groupOp, ID:$val),
826:                   "$res = OpGroupNonUniformBallotBitCount "
827:                           "$ty $scope $groupOp $val">;
828: def OpGroupNonUniformBallotFindLSB: OpGroupNU3<"BallotFindLSB", 343>;
829: def OpGroupNonUniformBallotFindMSB: OpGroupNU3<"BallotFindMSB", 344>;
830: def OpGroupNonUniformShuffle: OpGroupNU4<"Shuffle", 345>;
831: def OpGroupNonUniformShuffleXor: OpGroupNU4<"ShuffleXor", 346>;
832: def OpGroupNonUniformShuffleUp: OpGroupNU4<"ShuffleUp", 347>;
833: def OpGroupNonUniformShuffleDown: OpGroupNU4<"ShuffleDown", 348>;
834: class OpGroupNUGroup<string name, bits<16> opCode>: Op<opCode, (outs ID:$res),
835:                   (ins TYPE:$ty, ID:$scope, GroupOperation:$groupOp,
836:                    ID:$val, variable_ops),
837:                   "$res = OpGroupNonUniform"#name#" $ty $scope $groupOp $val">;
838: def OpGroupNonUniformIAdd: OpGroupNUGroup<"IAdd", 349>;
839: def OpGroupNonUniformFAdd: OpGroupNUGroup<"FAdd", 350>;
840: def OpGroupNonUniformIMul: OpGroupNUGroup<"IMul", 351>;
841: def OpGroupNonUniformFMul: OpGroupNUGroup<"FMul", 352>;
842: def OpGroupNonUniformSMin: OpGroupNUGroup<"SMin", 353>;
843: def OpGroupNonUniformUMin: OpGroupNUGroup<"UMin", 354>;
844: def OpGroupNonUniformFMin: OpGroupNUGroup<"FMin", 355>;
845: def OpGroupNonUniformSMax: OpGroupNUGroup<"SMax", 356>;
846: def OpGroupNonUniformUMax: OpGroupNUGroup<"UMax", 357>;
847: def OpGroupNonUniformFMax: OpGroupNUGroup<"FMax", 358>;
848: def OpGroupNonUniformBitwiseAnd: OpGroupNUGroup<"BitwiseAnd", 359>;
849: def OpGroupNonUniformBitwiseOr: OpGroupNUGroup<"BitwiseOr", 360>;
850: def OpGroupNonUniformBitwiseXor: OpGroupNUGroup<"BitwiseXor", 361>;
851: def OpGroupNonUniformLogicalAnd: OpGroupNUGroup<"LogicalAnd", 362>;
852: def OpGroupNonUniformLogicalOr: OpGroupNUGroup<"LogicalOr", 363>;
853: def OpGroupNonUniformLogicalXor: OpGroupNUGroup<"LogicalXor", 364>;
854: def OpGroupNonUniformQuadSwap: OpGroupNU4<"QuadSwap", 366>;
855:
856: // SPV_KHR_subgroup_rotate
857: def OpGroupNonUniformRotateKHR: Op<4431, (outs ID:$res),
858:                   (ins TYPE:$type, ID:$scope, ID:$value, ID:$delta, variable_ops),
859:                   "$res = OpGroupNonUniformRotateKHR $type $scope $value $delta">;
860:
861: // SPV_KHR_shader_clock
862: def OpReadClockKHR: Op<5056, (outs ID:$res),
863:                   (ins TYPE:$type, ID:$scope),
864:                   "$res = OpReadClockKHR $type $scope">;
865:
866: // 3.49.7, Constant-Creation Instructions
867:
868: //  - SPV_INTEL_function_pointers
869: def OpConstantFunctionPointerINTEL: Op<5600, (outs ID:$res), (ins TYPE:$ty, ID:$fun), "$res = OpConstantFunctionPointerINTEL $ty $fun">;
870:
871: // 3.49.9. Function Instructions
872:
873: //  - SPV_INTEL_function_pointers
874: def OpFunctionPointerCallINTEL: Op<5601, (outs ID:$res), (ins TYPE:$ty, ID:$funPtr, variable_ops), "$res = OpFunctionPointerCallINTEL $ty $funPtr">;
875:
876: // 3.49.21. Group and Subgroup Instructions
877:
878: // - SPV_INTEL_subgroups
879: def OpSubgroupShuffleINTEL: Op<5571, (outs ID:$res), (ins TYPE:$type, ID:$data, ID:$invocationId),
880:                   "$res = OpSubgroupShuffleINTEL $type $data $invocationId">;
881: def OpSubgroupShuffleDownINTEL: Op<5572, (outs ID:$res), (ins TYPE:$type, ID:$current, ID:$next, ID:$delta),
882:                   "$res = OpSubgroupShuffleDownINTEL $type $current $next $delta">;
883: def OpSubgroupShuffleUpINTEL: Op<5573, (outs ID:$res), (ins TYPE:$type, ID:$previous, ID:$current, ID:$delta),
884:                   "$res = OpSubgroupShuffleUpINTEL $type $previous $current $delta">;
885: def OpSubgroupShuffleXorINTEL: Op<5574, (outs ID:$res), (ins TYPE:$type, ID:$data, ID:$value),
886:                   "$res = OpSubgroupShuffleXorINTEL $type $data $value">;
887: def OpSubgroupBlockReadINTEL: Op<5575, (outs ID:$res), (ins TYPE:$type, ID:$ptr),
888:                   "$res = OpSubgroupBlockReadINTEL $type $ptr">;
889: def OpSubgroupBlockWriteINTEL: Op<5576, (outs), (ins ID:$ptr, ID:$data),
890:                   "OpSubgroupBlockWriteINTEL $ptr $data">;
891: def OpSubgroupImageBlockReadINTEL: Op<5577, (outs ID:$res), (ins TYPE:$type, ID:$image, ID:$coordinate),
892:                   "$res = OpSubgroupImageBlockReadINTEL $type $image $coordinate">;
893: def OpSubgroupImageBlockWriteINTEL: Op<5578, (outs), (ins ID:$image, ID:$coordinate, ID:$data),
894:                   "OpSubgroupImageBlockWriteINTEL $image $coordinate $data">;
895:
896: // SPV_INTEL_media_block_io
897: def OpSubgroupImageMediaBlockReadINTEL: Op<5580, (outs ID:$res), (ins TYPE:$type, ID:$image, ID:$coordinate, ID:$width, ID:$height),
898:                   "$res = OpSubgroupImageMediaBlockReadINTEL $type $image $coordinate $width $height">;
899: def OpSubgroupImageMediaBlockWriteINTEL: Op<5581, (outs), (ins ID:$image, ID:$coordinate, ID:$width, ID:$height, ID:$data),
900:                   "OpSubgroupImageMediaBlockWriteINTEL $image $coordinate $width $height $data">;
```
- EN: This range uses TableGen DSL to describe records such as OpGroupNU4, OpGroupNonUniformAll, OpGroupNonUniformAny, OpGroupNonUniformAllEqual; LLVM later expands these declarations into generated backend tables and helper code.
- CN: 这一段使用 TableGen DSL 描述 OpGroupNU4、OpGroupNonUniformAll、OpGroupNonUniformAny、OpGroupNonUniformAllEqual 等记录；LLVM 后续会把这些声明展开为生成的后端表和辅助代码。

### Lines 901-990
```tablegen
901:
902: // - SPV_KHR_uniform_group_instructions
903: def OpGroupIMulKHR: Op<6401, (outs ID:$res), (ins TYPE:$type, ID:$scope, i32imm:$groupOp, ID:$value),
904:                   "$res = OpGroupIMulKHR $type $scope $groupOp $value">;
905: def OpGroupFMulKHR: Op<6402, (outs ID:$res), (ins TYPE:$type, ID:$scope, i32imm:$groupOp, ID:$value),
906:                   "$res = OpGroupFMulKHR $type $scope $groupOp $value">;
907: def OpGroupBitwiseAndKHR: Op<6403, (outs ID:$res), (ins TYPE:$type, ID:$scope, i32imm:$groupOp, ID:$value),
908:                   "$res = OpGroupBitwiseAndKHR $type $scope $groupOp $value">;
909: def OpGroupBitwiseOrKHR: Op<6404, (outs ID:$res), (ins TYPE:$type, ID:$scope, i32imm:$groupOp, ID:$value),
910:                   "$res = OpGroupBitwiseOrKHR $type $scope $groupOp $value">;
911: def OpGroupBitwiseXorKHR: Op<6405, (outs ID:$res), (ins TYPE:$type, ID:$scope, i32imm:$groupOp, ID:$value),
912:                   "$res = OpGroupBitwiseXorKHR $type $scope $groupOp $value">;
913: def OpGroupLogicalAndKHR: Op<6406, (outs ID:$res), (ins TYPE:$type, ID:$scope, i32imm:$groupOp, ID:$value),
914:                   "$res = OpGroupLogicalAndKHR $type $scope $groupOp $value">;
915: def OpGroupLogicalOrKHR: Op<6407, (outs ID:$res), (ins TYPE:$type, ID:$scope, i32imm:$groupOp, ID:$value),
916:                   "$res = OpGroupLogicalOrKHR $type $scope $groupOp $value">;
917: def OpGroupLogicalXorKHR: Op<6408, (outs ID:$res), (ins TYPE:$type, ID:$scope, i32imm:$groupOp, ID:$value),
918:                   "$res = OpGroupLogicalXorKHR $type $scope $groupOp $value">;
919:
920: // Inline Assembly Instructions
921: def OpAsmTargetINTEL: Op<5609, (outs ID:$res), (ins StringImm:$str, variable_ops), "$res = OpAsmTargetINTEL $str">;
922: def OpAsmINTEL: Op<5610, (outs ID:$res), (ins TYPE:$type, TYPE:$asm_type, ID:$target,
923:                                           StringImm:$asm, StringImm:$constraints, variable_ops),
924:                   "$res = OpAsmINTEL $type $asm_type $target $asm">;
925: def OpAsmCallINTEL: Op<5611, (outs ID:$res), (ins TYPE:$type, ID:$asm, variable_ops),
926:                   "$res = OpAsmCallINTEL $type $asm">;
927:
928: // SPV_KHR_cooperative_matrix
929: def OpCooperativeMatrixLoadKHR: Op<4457, (outs ID:$res),
930:                   (ins TYPE:$resType, ID:$pointer, ID:$memory_layout, variable_ops),
931:                   "$res = OpCooperativeMatrixLoadKHR $resType $pointer $memory_layout">;
932: def OpCooperativeMatrixStoreKHR: Op<4458, (outs),
933:                   (ins ID:$pointer, ID:$objectToStore, ID:$memory_layout, variable_ops),
934:                   "OpCooperativeMatrixStoreKHR $pointer $objectToStore $memory_layout">;
935: def OpCooperativeMatrixMulAddKHR: Op<4459, (outs ID:$res),
936:                   (ins TYPE:$type, ID:$A, ID:$B, ID:$C, variable_ops),
937:                   "$res = OpCooperativeMatrixMulAddKHR $type $A $B $C">;
938: def OpCooperativeMatrixLengthKHR: Op<4460, (outs ID:$res), (ins TYPE:$type, TYPE:$coop_matr_type),
939:                   "$res = OpCooperativeMatrixLengthKHR $type $coop_matr_type">;
940:
941: // SPV_INTEL_joint_matrix
942: def OpCooperativeMatrixLoadCheckedINTEL: Op<6193, (outs ID:$res),
943:                   (ins TYPE:$resType, ID:$pointer, ID:$xOffset, ID:$yOffset, ID:$memory_layout, ID:$height, ID:$width, variable_ops),
944:                   "$res = OpCooperativeMatrixLoadCheckedINTEL $resType $pointer $xOffset $yOffset $memory_layout $height $width">;
945: def OpCooperativeMatrixStoreCheckedINTEL: Op<6194, (outs),
946:                   (ins ID:$pointer, ID:$xOffset, ID:$yOffset, ID:$objectToStore, ID:$memory_layout, ID:$height, ID:$width, variable_ops),
947:                   "OpCooperativeMatrixStoreCheckedINTEL $pointer $xOffset $yOffset $objectToStore $memory_layout $height $width">;
948: def OpCooperativeMatrixConstructCheckedINTEL: Op<6195, (outs ID:$res),
949:                   (ins TYPE:$resType, ID:$xOffset, ID:$yOffset, ID:$height, ID:$width, ID:$value),
950:                   "$res = OpCooperativeMatrixConstructCheckedINTEL $resType $xOffset $yOffset $height $width $value">;
951: def OpCooperativeMatrixGetElementCoordINTEL: Op<6440, (outs ID:$res),
952:                   (ins TYPE:$resType, ID:$matrix, ID:$index),
953:                   "$res = OpCooperativeMatrixGetElementCoordINTEL $resType $matrix $index">;
954: def OpCooperativeMatrixPrefetchINTEL: Op<6449, (outs),
955:                   (ins ID:$pointer, ID:$rows, ID:$columns, i32imm:$cacheLevel, ID:$memory_layout, variable_ops),
956:                   "OpCooperativeMatrixPrefetchINTEL $pointer $rows $columns $cacheLevel $memory_layout">;
957:
958: // SPV_EXT_arithmetic_fence
959: def OpArithmeticFenceEXT: Op<6145, (outs ID:$res), (ins TYPE:$type, ID:$target),
960:                   "$res = OpArithmeticFenceEXT $type $target">;
961:
962: // SPV_INTEL_bindless_images
963: def OpConvertHandleToImageINTEL: Op<6529, (outs ID:$res), (ins TYPE:$type, ID:$operand),
964:                   "$res = OpConvertHandleToImageINTEL $type $operand">;
965: def OpConvertHandleToSamplerINTEL: Op<6530, (outs ID:$res), (ins TYPE:$type, ID:$operand),
966:                   "$res = OpConvertHandleToSamplerINTEL $type $operand">;
967: def OpConvertHandleToSampledImageINTEL: Op<6531, (outs ID:$res), (ins TYPE:$type, ID:$operand),
968:                   "$res = OpConvertHandleToSampledImageINTEL $type $operand">;
969:
970: // SPV_INTEL_memory_access_aliasing
971: def OpAliasDomainDeclINTEL: Op<5911, (outs ID:$res), (ins variable_ops),
972:                   "$res = OpAliasDomainDeclINTEL">;
973: def OpAliasScopeDeclINTEL: Op<5912, (outs ID:$res), (ins ID:$AliasDomain, variable_ops),
974:                   "$res = OpAliasScopeDeclINTEL $AliasDomain">;
975: def OpAliasScopeListDeclINTEL: Op<5913, (outs ID:$res), (ins variable_ops),
976:                   "$res = OpAliasScopeListDeclINTEL">;
977:
978: // SPV_INTEL_ternary_bitwise_function
979: def OpBitwiseFunctionINTEL: Op<6242, (outs ID:$res), (ins TYPE:$type, ID:$a, ID:$b, ID:$c, ID:$lut_index),
980:                   "$res = OpBitwiseFunctionINTEL $type $a $b $c $lut_index">;
981:
982: // SPV_ALTERA_arbitrary_precision_floating_point
983: def OpArbitraryFloatGTALTERA: Op<5850, (outs ID:$res), (ins TYPE:$type, ID:$A, i32imm:$Ma , ID:$B, i32imm:$Mb), 
984:                   "$res = OpArbitraryFloatGTALTERA $type $A $Ma $B $Mb">;
985: def OpArbitraryFloatGEALTERA: Op<5851, (outs ID:$res), (ins TYPE:$type, ID:$A, i32imm:$Ma , ID:$B, i32imm:$Mb), 
986:                   "$res = OpArbitraryFloatGEALTERA $type $A $Ma $B $Mb">;
987: def OpArbitraryFloatLTALTERA: Op<5852, (outs ID:$res), (ins TYPE:$type, ID:$A, i32imm:$Ma , ID:$B, i32imm:$Mb), 
988:                   "$res = OpArbitraryFloatLTALTERA $type $A $Ma $B $Mb">;
989: def OpArbitraryFloatLEALTERA: Op<5853, (outs ID:$res), (ins TYPE:$type, ID:$A, i32imm:$Ma , ID:$B, i32imm:$Mb), 
990:                   "$res = OpArbitraryFloatLEALTERA $type $A $Ma $B $Mb">;
```
- EN: This range uses TableGen DSL to describe records such as OpGroupIMulKHR, OpGroupFMulKHR, OpGroupBitwiseAndKHR, OpGroupBitwiseOrKHR; LLVM later expands these declarations into generated backend tables and helper code.
- CN: 这一段使用 TableGen DSL 描述 OpGroupIMulKHR、OpGroupFMulKHR、OpGroupBitwiseAndKHR、OpGroupBitwiseOrKHR 等记录；LLVM 后续会把这些声明展开为生成的后端表和辅助代码。

### Lines 991-1080
```tablegen
 991: def OpArbitraryFloatEQALTERA: Op<5854, (outs ID:$res), (ins TYPE:$type, ID:$A, i32imm:$Ma , ID:$B, i32imm:$Mb), 
 992:                   "$res = OpArbitraryFloatEQALTERA $type $A $Ma $B $Mb">;
 993: def OpArbitraryFloatRecipALTERA: Op<5855, (outs ID:$res),
 994:                   (ins TYPE:$type, ID:$A, i32imm:$Ma, i32imm:$Mresult, i32imm:$subnormalMode, i32imm:$RoundingMode, i32imm:$RoundingAccuracy),
 995:                   "$res = OpArbitraryFloatRecipALTERA $type $A $Ma $Mresult $subnormalMode $RoundingMode $RoundingAccuracy">;
 996: def OpArbitraryFloatCbrtALTERA: Op<5857, (outs ID:$res),
 997:                   (ins TYPE:$type, ID:$A, i32imm:$Ma, i32imm:$Mresult, i32imm:$subnormalMode, i32imm:$RoundingMode, i32imm:$RoundingAccuracy),
 998:                   "$res = OpArbitraryFloatCbrtALTERA $type $A $Ma $Mresult $subnormalMode $RoundingMode $RoundingAccuracy">;
 999: def OpArbitraryFloatHypotALTERA: Op<5858, (outs ID:$res),
1000:                   (ins TYPE:$type, ID:$A, i32imm:$Ma, ID:$B, i32imm:$Mb, i32imm:$Mresult, i32imm:$subnormalMode, i32imm:$RoundingMode, i32imm:$RoundingAccuracy),
1001:                   "$res = OpArbitraryFloatHypotALTERA $type $A $Ma $B $Mb $Mresult $subnormalMode $RoundingMode $RoundingAccuracy">;
1002: def OpArbitraryFloatSqrtALTERA: Op<5859, (outs ID:$res),
1003:                   (ins TYPE:$type, ID:$A, i32imm:$Ma, i32imm:$Mresult, i32imm:$subnormalMode, i32imm:$RoundingMode, i32imm:$RoundingAccuracy),
1004:                   "$res = OpArbitraryFloatSqrtALTERA $type $A $Ma $Mresult $subnormalMode $RoundingMode $RoundingAccuracy">;
1005: def OpArbitraryFloatLogALTERA: Op<5860, (outs ID:$res),
1006:                   (ins TYPE:$type, ID:$A, i32imm:$Ma, i32imm:$Mresult, i32imm:$subnormalMode, i32imm:$RoundingMode, i32imm:$RoundingAccuracy),
1007:                   "$res = OpArbitraryFloatLogALTERA $type $A $Ma $Mresult $subnormalMode $RoundingMode $RoundingAccuracy">;
1008: def OpArbitraryFloatLog2ALTERA: Op<5861, (outs ID:$res),
1009:                   (ins TYPE:$type, ID:$A, i32imm:$Ma, i32imm:$Mresult, i32imm:$subnormalMode, i32imm:$RoundingMode, i32imm:$RoundingAccuracy),
1010:                   "$res = OpArbitraryFloatLog2ALTERA $type $A $Ma $Mresult $subnormalMode $RoundingMode $RoundingAccuracy">;
1011: def OpArbitraryFloatLog10ALTERA: Op<5862, (outs ID:$res),
1012:                   (ins TYPE:$type, ID:$A, i32imm:$Ma, i32imm:$Mresult, i32imm:$subnormalMode, i32imm:$RoundingMode, i32imm:$RoundingAccuracy),
1013:                   "$res = OpArbitraryFloatLog10ALTERA $type $A $Ma $Mresult $subnormalMode $RoundingMode $RoundingAccuracy">;
1014: def OpArbitraryFloatLog1pALTERA: Op<5863, (outs ID:$res),
1015:                   (ins TYPE:$type, ID:$A, i32imm:$Ma, i32imm:$Mresult, i32imm:$subnormalMode, i32imm:$RoundingMode, i32imm:$RoundingAccuracy),
1016:                   "$res = OpArbitraryFloatLog1pALTERA $type $A $Ma $Mresult $subnormalMode $RoundingMode $RoundingAccuracy">;
1017: def OpArbitraryFloatExpALTERA: Op<5864, (outs ID:$res),
1018:                   (ins TYPE:$type, ID:$A, i32imm:$Ma, i32imm:$Mresult, i32imm:$subnormalMode, i32imm:$RoundingMode, i32imm:$RoundingAccuracy),
1019:                   "$res = OpArbitraryFloatExpALTERA $type $A $Ma $Mresult $subnormalMode $RoundingMode $RoundingAccuracy">;
1020: def OpArbitraryFloatExp2ALTERA: Op<5865, (outs ID:$res),
1021:                   (ins TYPE:$type, ID:$A, i32imm:$Ma, i32imm:$Mresult, i32imm:$subnormalMode, i32imm:$RoundingMode, i32imm:$RoundingAccuracy),
1022:                   "$res = OpArbitraryFloatExp2ALTERA $type $A $Ma $Mresult $subnormalMode $RoundingMode $RoundingAccuracy">;
1023: def OpArbitraryFloatExp10ALTERA: Op<5866, (outs ID:$res),
1024:                   (ins TYPE:$type, ID:$A, i32imm:$Ma, i32imm:$Mresult, i32imm:$subnormalMode, i32imm:$RoundingMode, i32imm:$RoundingAccuracy),
1025:                   "$res = OpArbitraryFloatExp10ALTERA $type $A $Ma $Mresult $subnormalMode $RoundingMode $RoundingAccuracy">;
1026: def OpArbitraryFloatExpm1ALTERA: Op<5867, (outs ID:$res),
1027:                   (ins TYPE:$type, ID:$A, i32imm:$Ma, i32imm:$Mresult, i32imm:$subnormalMode, i32imm:$RoundingMode, i32imm:$RoundingAccuracy),
1028:                   "$res = OpArbitraryFloatExpm1ALTERA $type $A $Ma $Mresult $subnormalMode $RoundingMode $RoundingAccuracy">;
1029: def OpArbitraryFloatSinALTERA: Op<5868, (outs ID:$res),
1030:                   (ins TYPE:$type, ID:$A, i32imm:$Ma, i32imm:$Mresult, i32imm:$subnormalMode, i32imm:$RoundingMode, i32imm:$RoundingAccuracy),
1031:                   "$res = OpArbitraryFloatSinALTERA $type $A $Ma $Mresult $subnormalMode $RoundingMode $RoundingAccuracy">;
1032: def OpArbitraryFloatCosALTERA: Op<5869, (outs ID:$res),
1033:                   (ins TYPE:$type, ID:$A, i32imm:$Ma, i32imm:$Mresult, i32imm:$subnormalMode, i32imm:$RoundingMode, i32imm:$RoundingAccuracy),
1034:                   "$res = OpArbitraryFloatCosALTERA $type $A $Ma $Mresult $subnormalMode $RoundingMode $RoundingAccuracy">;
1035: def OpArbitraryFloatSinCosALTERA: Op<5870, (outs ID:$res),
1036:                   (ins TYPE:$type, ID:$A, i32imm:$Ma, i32imm:$Mresult, i32imm:$subnormalMode, i32imm:$RoundingMode, i32imm:$RoundingAccuracy),
1037:                   "$res = OpArbitraryFloatSinCosALTERA $type $A $Ma $Mresult $subnormalMode $RoundingMode $RoundingAccuracy">;
1038: def OpArbitraryFloatSinPiALTERA: Op<5871, (outs ID:$res),
1039:                   (ins TYPE:$type, ID:$A, i32imm:$Ma, i32imm:$Mresult, i32imm:$subnormalMode, i32imm:$RoundingMode, i32imm:$RoundingAccuracy),
1040:                   "$res = OpArbitraryFloatSinPiALTERA $type $A $Ma $Mresult $subnormalMode $RoundingMode $RoundingAccuracy">;
1041: def OpArbitraryFloatCosPiALTERA: Op<5872, (outs ID:$res),
1042:                   (ins TYPE:$type, ID:$A, i32imm:$Ma, i32imm:$Mresult, i32imm:$subnormalMode, i32imm:$RoundingMode, i32imm:$RoundingAccuracy),
1043:                   "$res = OpArbitraryFloatCosPiALTERA $type $A $Ma $Mresult $subnormalMode $RoundingMode $RoundingAccuracy">;
1044: def OpArbitraryFloatSinCosPiALTERA: Op<5840, (outs ID:$res),
1045:                   (ins TYPE:$type, ID:$A, i32imm:$Ma, i32imm:$Mresult, i32imm:$subnormalMode, i32imm:$RoundingMode, i32imm:$RoundingAccuracy),
1046:                   "$res = OpArbitraryFloatSinCosPiALTERA $type $A $Ma $Mresult $subnormalMode $RoundingMode $RoundingAccuracy">;
1047: def OpArbitraryFloatAddALTERA: Op<5846, (outs ID:$res),
1048:                   (ins TYPE:$type, ID:$src, i32imm:$src_mwidth, ID:$src2, i32imm:$src2_mwidth, i32imm:$res_mwidth, i32imm:$subnorm, i32imm:$fproundingmode, i32imm:$roundaccuracymode),
1049:                   "$res = OpArbitraryFloatAddALTERA $type $src $src_mwidth $src2 $src2_mwidth $res_mwidth $subnorm $fproundingmode $roundaccuracymode">;
1050: def OpArbitraryFloatSubALTERA: Op<5847, (outs ID:$res), (ins TYPE:$type, ID:$src, i32imm:$src_mwidth, ID:$src2, i32imm:$src2_mwidth, i32imm:$res_mwidth, i32imm:$subnorm, i32imm:$fproundingmode, i32imm:$roundaccuracymode),
1051:                   "$res = OpArbitraryFloatSubALTERA $type $src $src_mwidth $src2 $src2_mwidth $res_mwidth $subnorm $fproundingmode $roundaccuracymode">;
1052: def OpArbitraryFloatMulALTERA: Op<5848, (outs ID:$res), (ins TYPE:$type, ID:$src, i32imm:$src_mwidth, ID:$src2, i32imm:$src2_mwidth, i32imm:$res_mwidth, i32imm:$subnorm, i32imm:$fproundingmode, i32imm:$roundaccuracymode),
1053:                   "$res = OpArbitraryFloatMulALTERA $type $src $src_mwidth $src2 $src2_mwidth $res_mwidth $subnorm $fproundingmode $roundaccuracymode">;
1054: def OpArbitraryFloatDivALTERA: Op<5849, (outs ID:$res), (ins TYPE:$type, ID:$src, i32imm:$src_mwidth, ID:$src2, i32imm:$src2_mwidth, i32imm:$res_mwidth, i32imm:$subnorm, i32imm:$fproundingmode, i32imm:$roundaccuracymode),
1055:                   "$res = OpArbitraryFloatDivALTERA $type $src $src_mwidth $src2 $src2_mwidth $res_mwidth $subnorm $fproundingmode $roundaccuracymode">;
1056: def OpArbitraryFloatRSqrtALTERA: Op<5856, (outs ID:$res), (ins TYPE:$type, ID:$src, i32imm:$src_mwidth, i32imm:$res_mwidth, i32imm:$subnorm, i32imm:$fproundingmode, i32imm:$roundaccuracymode),
1057:                   "$res = OpArbitraryFloatRSqrtALTERA $type $src $src_mwidth $res_mwidth $subnorm $fproundingmode $roundaccuracymode">;
1058: def OpArbitraryFloatASinALTERA: Op<5873, (outs ID:$res), (ins TYPE:$type, ID:$src, i32imm:$src_mwidth, i32imm:$res_mwidth, i32imm:$subnorm, i32imm:$fproundingmode, i32imm:$roundaccuracymode),
1059:                   "$res = OpArbitraryFloatASinALTERA $type $src $src_mwidth $res_mwidth $subnorm $fproundingmode $roundaccuracymode">;
1060: def OpArbitraryFloatASinPiALTERA: Op<5874, (outs ID:$res), (ins TYPE:$type, ID:$src, i32imm:$src_mwidth, i32imm:$res_mwidth, i32imm:$subnorm, i32imm:$fproundingmode, i32imm:$roundaccuracymode),
1061:                   "$res = OpArbitraryFloatASinPiALTERA $type $src $src_mwidth $res_mwidth $subnorm $fproundingmode $roundaccuracymode">;
1062: def OpArbitraryFloatACosALTERA : Op<5875, (outs ID:$res), (ins TYPE:$type, ID:$src, i32imm:$src_mwidth, i32imm:$res_mwidth, i32imm:$subnorm, i32imm:$fproundingmode, i32imm:$roundaccuracymode),
1063:                   "$res = OpArbitraryFloatACosALTERA $type $src $src_mwidth $res_mwidth $subnorm $fproundingmode $roundaccuracymode">;
1064: def OpArbitraryFloatACosPiALTERA: Op<5876, (outs ID:$res), (ins TYPE:$type, ID:$src, i32imm:$src_mwidth, i32imm:$res_mwidth, i32imm:$subnorm, i32imm:$fproundingmode, i32imm:$roundaccuracymode),
1065:                   "$res = OpArbitraryFloatACosPiALTERA $type $src $src_mwidth $res_mwidth $subnorm $fproundingmode $roundaccuracymode">;
1066: def OpArbitraryFloatATanALTERA: Op<5877, (outs ID:$res), (ins TYPE:$type, ID:$src, i32imm:$src_mwidth, i32imm:$res_mwidth, i32imm:$subnorm, i32imm:$fproundingmode, i32imm:$roundaccuracymode),
1067:                   "$res = OpArbitraryFloatATanALTERA $type $src $src_mwidth $res_mwidth $subnorm $fproundingmode $roundaccuracymode">;
1068: def OpArbitraryFloatATanPiALTERA: Op<5878, (outs ID:$res), (ins TYPE:$type, ID:$src, i32imm:$src_mwidth, i32imm:$res_mwidth, i32imm:$subnorm, i32imm:$fproundingmode, i32imm:$roundaccuracymode),
1069:                   "$res = OpArbitraryFloatATanPiALTERA $type $src $src_mwidth $res_mwidth $subnorm $fproundingmode $roundaccuracymode">;
1070: def OpArbitraryFloatATan2ALTERA: Op<5879, (outs ID:$res), (ins TYPE:$type, ID:$src, i32imm:$src_mwidth, ID:$src2, i32imm:$src2_mwidth, i32imm:$res_mwidth, i32imm:$subnorm, i32imm:$fproundingmode, i32imm:$roundaccuracymode),
1071:                   "$res = OpArbitraryFloatATan2ALTERA $type $src $src_mwidth $src2 $src2_mwidth $res_mwidth $subnorm $fproundingmode $roundaccuracymode">;
1072: def OpArbitraryFloatPowALTERA: Op<5880, (outs ID:$res), (ins TYPE:$type, ID:$src, i32imm:$src_mwidth, ID:$src2, i32imm:$src2_mwidth, i32imm:$res_mwidth, i32imm:$subnorm, i32imm:$fproundingmode, i32imm:$roundaccuracymode),
1073:                   "$res = OpArbitraryFloatPowALTERA $type $src $src_mwidth $src2 $src2_mwidth $res_mwidth $subnorm $fproundingmode $roundaccuracymode">;
1074: def OpArbitraryFloatPowRALTERA: Op<5881, (outs ID:$res), (ins TYPE:$type, ID:$src, i32imm:$src_mwidth, ID:$src2, i1imm:$src2_sign, i32imm:$res_mwidth, i32imm:$subnorm, i32imm:$fproundingmode, i32imm:$roundaccuracymode),
1075:                   "$res = OpArbitraryFloatPowRALTERA $type $src $src_mwidth $src2 $src2_sign $res_mwidth $subnorm $fproundingmode $roundaccuracymode">;
1076: def OpArbitraryFloatPowNALTERA: Op<5882, (outs ID:$res), (ins TYPE:$type, ID:$src, i32imm:$src_mwidth, ID:$src2, i1imm:$src2_sign, i32imm:$res_mwidth, i32imm:$subnorm, i32imm:$fproundingmode, i32imm:$roundaccuracymode),
1077:                   "$res = OpArbitraryFloatPowNALTERA $type $src $src_mwidth $src2 $src2_sign $res_mwidth $subnorm $fproundingmode $roundaccuracymode">;
1078: def OpArbitraryFloatCastALTERA: Op<5841, (outs ID:$res), (ins TYPE:$type, ID:$src, i32imm:$src_mwidth, i32imm:$res_mwidth, i32imm:$subnorm, i32imm:$fproundingmode, i32imm:$roundignAccuracy),
1079:                   "$res = OpArbitraryFloatCastALTERA $type $src $src_mwidth $res_mwidth $subnorm $fproundingmode $roundignAccuracy">;
1080: def OpArbitraryFloatCastFromIntALTERA: Op<5842, (outs ID:$res), (ins TYPE:$type, ID:$src, i32imm:$res_mwidth, i32imm:$fromsign, i32imm:$subnormalmode, i32imm:$fproundingmode, i32imm:$roundignAccuracy),
```
- EN: This range uses TableGen DSL to describe records such as OpArbitraryFloatEQALTERA, OpArbitraryFloatRecipALTERA, OpArbitraryFloatCbrtALTERA, OpArbitraryFloatHypotALTERA; LLVM later expands these declarations into generated backend tables and helper code.
- CN: 这一段使用 TableGen DSL 描述 OpArbitraryFloatEQALTERA、OpArbitraryFloatRecipALTERA、OpArbitraryFloatCbrtALTERA、OpArbitraryFloatHypotALTERA 等记录；LLVM 后续会把这些声明展开为生成的后端表和辅助代码。

### Lines 1081-1142
```tablegen
1081:                   "$res = OpArbitraryFloatCastFromIntALTERA $type $src $res_mwidth $fromsign $subnormalmode $fproundingmode $roundignAccuracy">;
1082: def OpArbitraryFloatCastToIntALTERA: Op<5838, (outs ID:$res), (ins TYPE:$type, ID:$src, i32imm:$res_mwidth, i32imm:$tosign, i32imm:$subnormalmode, i32imm:$fproundingmode, i32imm:$roundignAccuracy),
1083:                   "$res = OpArbitraryFloatCastToIntALTERA $type $src $res_mwidth $tosign $subnormalmode $fproundingmode $roundignAccuracy">;
1084:
1085: // SPV_INTEL_2d_block_io
1086: def OpSubgroup2DBlockLoadINTEL: Op<6231, (outs), (ins ID:$element_size, ID:$block_width, ID:$block_height,
1087:                   ID:$block_count, ID:$src_base_ptr, ID:$memory_width, ID:$memory_height, ID:$memory_pitch, ID:$coord, ID:$dst_ptr),
1088:                   "OpSubgroup2DBlockLoadINTEL $element_size $block_width $block_height $block_count $src_base_ptr $memory_width $memory_height $memory_pitch $coord $dst_ptr">;
1089: def OpSubgroup2DBlockLoadTransposeINTEL: Op<6233, (outs), (ins ID:$element_size, ID:$block_width, ID:$block_height,
1090:                   ID:$block_count, ID:$src_base_ptr, ID:$memory_width, ID:$memory_height, ID:$memory_pitch, ID:$coord, ID:$dst_ptr),
1091:                   "OpSubgroup2DBlockLoadTransposeINTEL $element_size $block_width $block_height $block_count $src_base_ptr $memory_width $memory_height $memory_pitch $coord $dst_ptr">;
1092: def OpSubgroup2DBlockLoadTransformINTEL: Op<6232, (outs), (ins ID:$element_size, ID:$block_width, ID:$block_height,
1093:                   ID:$block_count, ID:$src_base_ptr, ID:$memory_width, ID:$memory_height, ID:$memory_pitch, ID:$coord, ID:$dst_ptr),
1094:                   "OpSubgroup2DBlockLoadTransformINTEL $element_size $block_width $block_height $block_count $src_base_ptr $memory_width $memory_height $memory_pitch $coord $dst_ptr">;
1095: def OpSubgroup2DBlockPrefetchINTEL: Op<6234, (outs), (ins ID:$element_size, ID:$block_width, ID:$block_height,
1096:                   ID:$block_count, ID:$src_base_ptr, ID:$memory_width, ID:$memory_height, ID:$memory_pitch, ID:$coord),
1097:                   "OpSubgroup2DBlockPrefetchINTEL $element_size $block_width $block_height $block_count $src_base_ptr $memory_width $memory_height $memory_pitch $coord">;
1098: def OpSubgroup2DBlockStoreINTEL: Op<6235, (outs), (ins ID:$element_size, ID:$block_width, ID:$block_height,
1099:                   ID:$block_count, ID:$src_ptr, ID:$dst_base_ptr, ID:$memory_width, ID:$memory_height, ID:$memory_pitch, ID:$coord),
1100:                   "OpSubgroup2DBlockStoreINTEL $element_size $block_width $block_height $block_count $src_ptr $dst_base_ptr $memory_width $memory_height $memory_pitch $coord">;
1101:
1102: // SPV_INTEL_predicated_io
1103: def OpPredicatedLoadINTEL: Op<6528, (outs ID:$res), (ins TYPE:$resType, ID:$ptr, ID:$predicate, ID:$default_value, variable_ops),
1104:                   "$res = OpPredicatedLoadINTEL $resType $ptr $predicate $default_value">;
1105: def OpPredicatedStoreINTEL: Op<6529, (outs), (ins ID:$ptr, ID:$object, ID:$predicate, variable_ops),
1106:                   "OpPredicatedStoreINTEL $ptr $object $predicate">;
1107:
1108: //SPV_ALTERA_blocking_pipes
1109: def OpReadPipeBlockingALTERA :Op<5946, (outs), (ins ID:$pipe, ID:$pointer, ID:$packetSize, ID:$packetAlignment),
1110:                    "OpReadPipeBlockingALTERA $pipe $pointer $packetSize $packetAlignment">;
1111: def OpWritePipeBlockingALTERA :Op<5946, (outs), (ins ID:$pipe, ID:$pointer, ID:$packetSize, ID:$packetAlignment),
1112:                    "OpWritePipeBlockingALTERA $pipe $pointer $packetSize $packetAlignment">;
1113:
1114: //SPV_ALTERA_arbitrary_precision_fixed_point
1115: def OpFixedSqrtALTERA: Op<5923, (outs ID:$res), (ins TYPE:$result_type, ID:$input, i32imm:$sign, i32imm:$l, i32imm:$rl, i32imm:$q, i32imm:$o),
1116:       "$res = OpFixedSqrtALTERA $result_type $input $sign $l $rl $q $o">;
1117: def OpFixedRecipALTERA: Op<5924, (outs ID:$res), (ins TYPE:$result_type, ID:$input, i32imm:$sign, i32imm:$l, i32imm:$rl, i32imm:$q, i32imm:$o),
1118:       "$res = OpFixedRecipALTERA $result_type $input $sign $l $rl $q $o">;
1119: def OpFixedRsqrtALTERA: Op<5925, (outs ID:$res), (ins TYPE:$result_type, ID:$input, i32imm:$sign, i32imm:$l, i32imm:$rl, i32imm:$q, i32imm:$o),
1120:       "$res = OpFixedRsqrtALTERA $result_type $input $sign $l $rl $q $o">;
1121: def OpFixedSinALTERA: Op<5926, (outs ID:$res), (ins TYPE:$result_type, ID:$input, i32imm:$sign, i32imm:$l, i32imm:$rl, i32imm:$q, i32imm:$o),
1122:       "$res = OpFixedSinALTERA $result_type $input $sign $l $rl $q $o">;
1123: def OpFixedCosALTERA: Op<5927, (outs ID:$res), (ins TYPE:$result_type, ID:$input, i32imm:$sign, i32imm:$l, i32imm:$rl, i32imm:$q, i32imm:$o),
1124:       "$res = OpFixedCosALTERA $result_type $input $sign $l $rl $q $o">;
1125: def OpFixedSinCosALTERA: Op<5928, (outs ID:$res), (ins TYPE:$result_type, ID:$input, i32imm:$sign, i32imm:$l, i32imm:$rl, i32imm:$q, i32imm:$o),
1126:       "$res = OpFixedSinCosALTERA $result_type $input $sign $l $rl $q $o">;
1127: def OpFixedSinPiALTERA: Op<5929, (outs ID:$res), (ins TYPE:$result_type, ID:$input, i32imm:$sign, i32imm:$l, i32imm:$rl, i32imm:$q, i32imm:$o),
1128:       "$res = OpFixedSinPiALTERA $result_type $input $sign $l $rl $q $o">;
1129: def OpFixedCosPiALTERA: Op<5930, (outs ID:$res), (ins TYPE:$result_type, ID:$input, i32imm:$sign, i32imm:$l, i32imm:$rl, i32imm:$q, i32imm:$o),
1130:       "$res = OpFixedCosPiALTERA $result_type $input $sign $l $rl $q $o">;
1131: def OpFixedSinCosPiALTERA: Op<5931, (outs ID:$res), (ins TYPE:$result_type, ID:$input, i32imm:$sign, i32imm:$l, i32imm:$rl, i32imm:$q, i32imm:$o),
1132:       "$res = OpFixedSinCosPiALTERA $result_type $input $sign $l $rl $q $o">;
1133: def OpFixedLogALTERA: Op<5932, (outs ID:$res), (ins TYPE:$result_type, ID:$input, i32imm:$sign, i32imm:$l, i32imm:$rl, i32imm:$q, i32imm:$o),
1134:       "$res = OpFixedLogALTERA $result_type $input $sign $l $rl $q $o">;
1135: def OpFixedExpALTERA: Op<5933, (outs ID:$res), (ins TYPE:$result_type, ID:$input, i32imm:$sign, i32imm:$l, i32imm:$rl, i32imm:$q, i32imm:$o),
1136:       "$res = OpFixedExpALTERA $result_type $input $sign $l $rl $q $o">;
1137:
1138: //SPV_INTEL_masked_gather_scatter
1139: def OpMaskedGatherINTEL: Op<6428, (outs ID:$res), (ins TYPE:$resType, ID:$ptrs, ID:$alignment, ID:$mask, ID:$fillEmpty),
1140:                   "$res = OpMaskedGatherINTEL $resType $ptrs $alignment $mask $fillEmpty">;
1141: def OpMaskedScatterINTEL: Op<6429, (outs), (ins ID:$ptrs, ID:$alignment, ID:$mask, ID:$values),
1142:                   "OpMaskedScatterINTEL $ptrs $alignment $mask $values">;
```
- EN: This range uses TableGen DSL to describe records such as OpArbitraryFloatCastToIntALTERA, OpSubgroup2DBlockLoadINTEL, OpSubgroup2DBlockLoadTransposeINTEL, OpSubgroup2DBlockLoadTransformINTEL; LLVM later expands these declarations into generated backend tables and helper code.
- CN: 这一段使用 TableGen DSL 描述 OpArbitraryFloatCastToIntALTERA、OpSubgroup2DBlockLoadINTEL、OpSubgroup2DBlockLoadTransposeINTEL、OpSubgroup2DBlockLoadTransformINTEL 等记录；LLVM 后续会把这些声明展开为生成的后端表和辅助代码。

## Key Concepts / 关键概念

- EN: Instruction info files centralize opcode semantics, scheduling hooks, and target-specific machine properties.
  - CN: 指令信息文件集中描述操作码语义、调度钩子以及目标相关机器属性。
- EN: TableGen files use declarative records that LLVM expands into generated C++ tables and helper code.
  - CN: TableGen 文件使用声明式记录，LLVM 会将其展开为生成的 C++ 表和辅助代码。
- EN: SPIR-V backend code must preserve rich type information and encode capabilities/extensions explicitly.
  - CN: SPIR-V 后端需要保留丰富的类型信息，并显式编码 capability 与 extension。
- EN: Key symbols in this file include ASSIGN_TYPE, UNKNOWN_type, SPVTypeBin, assigntype, BinOp, which anchor the file's main abstractions.
  - CN: 该文件中的关键符号包括 ASSIGN_TYPE, UNKNOWN_type, SPVTypeBin, assigntype, BinOp，它们构成了本文件的核心抽象。

## Dependencies / 依赖关系

- Local backend headers / 本地后端头文件:
  - `SPIRVInstrFormats.td`
  - `SPIRVSymbolicOperands.td`
- TableGen pipeline / TableGen 流水线: records in this file are consumed by LLVM TableGen emitters to produce generated lookup tables and helper code.
- TableGen 流水线说明：该文件中的记录会被 LLVM 的 TableGen 生成器消费，进而产出查找表和辅助代码。
