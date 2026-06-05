# SPIRVBuiltins.td — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/SPIRV/SPIRVBuiltins.td`
- Repository: `llvm-project`
- Purpose (EN): TableGen records defining implementation details of demangled builtin functions and types.
- 目的（中文）: 该文件使用 TableGen DSL 描述目标后端元数据，并驱动自动生成代码。
- Language: TableGen DSL (TableGen DSL noted below)
- Note: This file is written in TableGen DSL, so many records are declarative descriptions consumed by LLVM code generators.
- 说明：该文件使用 TableGen DSL 编写，因此许多记录都是供 LLVM 代码生成器消费的声明式描述。

## Line-by-Line Analysis / 逐行分析

> Note / 说明: To keep the document readable, the source is analyzed in contiguous line ranges while preserving the original order. Each code block prefixes original line numbers.

### Lines 1-90
```tablegen
 1: //===-- SPIRVBuiltins.td - Describe SPIRV Builtins ---------*- tablegen -*-===//
 2:  //
 3:  // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
 4:  // See https://llvm.org/LICENSE.txt for license information.
 5:  // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
 6:  //
 7:  //===----------------------------------------------------------------------===//
 8:  //
 9:  // TableGen records defining implementation details of demangled builtin
10:  // functions and types.
11:  //
12:  //===----------------------------------------------------------------------===//
13:
14: // Define SPIR-V external builtin/instruction sets
15: def InstructionSet : GenericEnum {
16:   let FilterClass = "InstructionSet";
17:   let NameField = "Name";
18:   let ValueField = "Value";
19: }
20:
21: class InstructionSet<bits<32> value> {
22:   string Name = NAME;
23:   bits<32> Value = value;
24: }
25:
26: def OpenCL_std : InstructionSet<0>;
27: def GLSL_std_450 : InstructionSet<1>;
28: def SPV_AMD_shader_trinary_minmax : InstructionSet<2>;
29: def NonSemantic_Shader_DebugInfo_100 : InstructionSet<3>;
30:
31: // Define various builtin groups
32: def BuiltinGroup : GenericEnum {
33:   let FilterClass = "BuiltinGroup";
34: }
35:
36: class BuiltinGroup;
37:
38: def Extended : BuiltinGroup;
39: def Relational : BuiltinGroup;
40: def Group : BuiltinGroup;
41: def Variable : BuiltinGroup;
42: def Atomic : BuiltinGroup;
43: def Barrier : BuiltinGroup;
44: def Dot : BuiltinGroup;
45: def IntegerDot : BuiltinGroup;
46: def Wave : BuiltinGroup;
47: def GetQuery : BuiltinGroup;
48: def ImageSizeQuery : BuiltinGroup;
49: def ImageMiscQuery : BuiltinGroup;
50: def Convert : BuiltinGroup;
51: def ReadImage : BuiltinGroup;
52: def WriteImage : BuiltinGroup;
53: def SampleImage : BuiltinGroup;
54: def Select : BuiltinGroup;
55: def SpecConstant : BuiltinGroup;
56: def Enqueue : BuiltinGroup;
57: def AsyncCopy : BuiltinGroup;
58: def VectorLoadStore : BuiltinGroup;
59: def LoadStore : BuiltinGroup;
60: def IntelSubgroups : BuiltinGroup;
61: def AtomicFloating : BuiltinGroup;
62: def GroupUniform : BuiltinGroup;
63: def KernelClock : BuiltinGroup;
64: def CastToPtr : BuiltinGroup;
65: def Construct : BuiltinGroup;
66: def CoopMatr : BuiltinGroup;
67: def ICarryBorrow : BuiltinGroup;
68: def MulExtended : BuiltinGroup;
69: def ExtendedBitOps : BuiltinGroup;
70: def BindlessINTEL : BuiltinGroup;
71: def TernaryBitwiseINTEL : BuiltinGroup;
72: def Block2DLoadStore : BuiltinGroup;
73: def Pipe : BuiltinGroup;
74: def PredicatedLoadStore : BuiltinGroup;
75: def Arithmetic : BuiltinGroup;
76: def ArbitraryPrecisionFixedPoint : BuiltinGroup;
77: def BlockingPipes : BuiltinGroup;
78: def ImageChannelDataTypes : BuiltinGroup;
79: def ArbitraryFloatingPoint: BuiltinGroup;
80:
81: //===----------------------------------------------------------------------===//
82: // Class defining a demangled builtin record. The information in the record
83: // should be used to expand the builtin into either native SPIR-V instructions
84: // or an external call (in case of builtins without a direct mapping).
85: //
86: // name is the demangled name of the given builtin.
87: // set specifies which external instruction set the builtin belongs to.
88: // group specifies to which implementation group given record belongs.
89: // minNumArgs is the minimum required number of arguments for lowering.
90: // maxNumArgs specifies the maximum used number of arguments for lowering.
```
- EN: This range uses TableGen DSL to describe records such as InstructionSet, OpenCL_std, GLSL_std_450, SPV_AMD_shader_trinary_minmax; LLVM later expands these declarations into generated backend tables and helper code.
- CN: 这一段使用 TableGen DSL 描述 InstructionSet、OpenCL_std、GLSL_std_450、SPV_AMD_shader_trinary_minmax 等记录；LLVM 后续会把这些声明展开为生成的后端表和辅助代码。

### Lines 91-180
```tablegen
 91: //===----------------------------------------------------------------------===//
 92: class DemangledBuiltin<string name, InstructionSet set, BuiltinGroup group, bits<8> minNumArgs, bits<8> maxNumArgs> {
 93:   string Name = name;
 94:   InstructionSet Set = set;
 95:   BuiltinGroup Group = group;
 96:   bits<8> MinNumArgs = minNumArgs;
 97:   bits<8> MaxNumArgs = maxNumArgs;
 98: }
 99:
100: // Table gathering all the builtins.
101: def DemangledBuiltins : GenericTable {
102:   let FilterClass = "DemangledBuiltin";
103:   let Fields = ["Name", "Set", "Group", "MinNumArgs", "MaxNumArgs"];
104:   string TypeOf_Set = "InstructionSet";
105:   string TypeOf_Group = "BuiltinGroup";
106: }
107:
108: // Function to lookup builtins by their demangled name and set.
109: def lookupBuiltin : SearchIndex {
110:   let Table = DemangledBuiltins;
111:   let Key = ["Name", "Set"];
112: }
113:
114: // Dot builtin record:
115: def : DemangledBuiltin<"dot", OpenCL_std, Dot, 2, 2>;
116: def : DemangledBuiltin<"__spirv_Dot", OpenCL_std, Dot, 2, 2>;
117: def : DemangledBuiltin<"dot_acc_sat", OpenCL_std, IntegerDot, 3, 3>;
118: def : DemangledBuiltin<"__spirv_DotAccSat", OpenCL_std, IntegerDot, 3, 3>;
119:
120: // Image builtin records:
121: def : DemangledBuiltin<"read_imagei", OpenCL_std, ReadImage, 2, 4>;
122: def : DemangledBuiltin<"read_imageui", OpenCL_std, ReadImage, 2, 4>;
123: def : DemangledBuiltin<"read_imagef", OpenCL_std, ReadImage, 2, 4>;
124: def : DemangledBuiltin<"__spirv_ImageRead", OpenCL_std, ReadImage, 2, 0>;
125:
126: def : DemangledBuiltin<"write_imagef", OpenCL_std, WriteImage, 3, 4>;
127: def : DemangledBuiltin<"write_imagei", OpenCL_std, WriteImage, 3, 4>;
128: def : DemangledBuiltin<"write_imageui", OpenCL_std, WriteImage, 3, 4>;
129: def : DemangledBuiltin<"write_imageh", OpenCL_std, WriteImage, 3, 4>;
130: def : DemangledBuiltin<"__spirv_ImageWrite", OpenCL_std, WriteImage, 3, 0>;
131:
132: def : DemangledBuiltin<"__translate_sampler_initializer", OpenCL_std, SampleImage, 1, 1>;
133: def : DemangledBuiltin<"__spirv_SampledImage", OpenCL_std, SampleImage, 2, 2>;
134: def : DemangledBuiltin<"__spirv_ImageSampleExplicitLod", OpenCL_std, SampleImage, 3, 4>;
135:
136: // Select builtin record:
137: def : DemangledBuiltin<"__spirv_Select", OpenCL_std, Select, 3, 3>;
138:
139: // Composite Construct builtin record:
140: def : DemangledBuiltin<"__spirv_CompositeConstruct", OpenCL_std, Construct, 1, 0>;
141:
142: //===----------------------------------------------------------------------===//
143: // Class defining an extended builtin record used for lowering into an
144: // OpExtInst instruction.
145: //
146: // name is the demangled name of the given builtin.
147: // set specifies which external instruction set the builtin belongs to.
148: // number specifies the number of the instruction in the external set.
149: //===----------------------------------------------------------------------===//
150: class ExtendedBuiltin<string name, InstructionSet set, int number> {
151:   string Name = name;
152:   InstructionSet Set = set;
153:   bits<32> Number = number;
154: }
155:
156: // Table gathering all the extended builtins.
157: def ExtendedBuiltins : GenericTable {
158:   let FilterClass = "ExtendedBuiltin";
159:   let Fields = ["Name", "Set", "Number"];
160:   string TypeOf_Set = "InstructionSet";
161: }
162:
163: // Function to lookup extended builtins by their name and set.
164: def lookupExtendedBuiltin : SearchIndex {
165:   let Table = ExtendedBuiltins;
166:   let Key = ["Name", "Set"];
167: }
168:
169: // Function to lookup extended builtins by their set and number.
170: def lookupExtendedBuiltinBySetAndNumber : SearchIndex {
171:   let Table = ExtendedBuiltins;
172:   let Key = ["Set", "Number"];
173: }
174:
175: // OpenCL extended instruction enums
176: def OpenCLExtInst : GenericEnum {
177:   let FilterClass = "OpenCLExtInst";
178:   let NameField = "Name";
179:   let ValueField = "Value";
180: }
```
- EN: This range uses TableGen DSL to describe records such as DemangledBuiltin, DemangledBuiltins, lookupBuiltin, ExtendedBuiltin; LLVM later expands these declarations into generated backend tables and helper code.
- CN: 这一段使用 TableGen DSL 描述 DemangledBuiltin、DemangledBuiltins、lookupBuiltin、ExtendedBuiltin 等记录；LLVM 后续会把这些声明展开为生成的后端表和辅助代码。

### Lines 181-270
```tablegen
181:
182: class OpenCLExtInst<string name, bits<32> value> {
183:   string Name = name;
184:   bits<32> Value = value;
185: }
186:
187: // GLSL extended instruction enums
188: def GLSLExtInst : GenericEnum {
189:   let FilterClass = "GLSLExtInst";
190:   let NameField = "Name";
191:   let ValueField = "Value";
192: }
193:
194: class GLSLExtInst<string name, bits<32> value> {
195:   string Name = name;
196:   bits<32> Value = value;
197: }
198:
199: def NonSemanticExtInst : GenericEnum {
200:   let FilterClass = "NonSemanticExtInst";
201:   let NameField = "Name";
202:   let ValueField = "Value";
203: }
204:
205: class NonSemanticExtInst<string name, bits<32> value> {
206:   string Name = name;
207:   bits<32> Value = value;
208: }
209:
210: // Multiclass used to define at the same time both a demangled builtin record
211: // and a corresponding extended builtin record.
212: multiclass DemangledExtendedBuiltin<string name, InstructionSet set, int number, bits<8> minNumArgs = 1, bits<8> maxNumArgs = 3> {
213:   def : DemangledBuiltin<name, set, Extended, minNumArgs, maxNumArgs>;
214:   def : ExtendedBuiltin<name, set, number>;
215:
216:   if !eq(set, OpenCL_std) then {
217:     def : OpenCLExtInst<name, number>;
218:   }
219:
220:   if !eq(set, GLSL_std_450) then {
221:     def : GLSLExtInst<name, number>;
222:   }
223:
224:   if !eq(set, NonSemantic_Shader_DebugInfo_100) then {
225:     def : NonSemanticExtInst<name, number>;
226:   }
227: }
228:
229: // Extended builtin records:
230: //
231: // OpenCL math builtins (1 arg).
232: defm : DemangledExtendedBuiltin<"acos", OpenCL_std, 0, 1, 1>;
233: defm : DemangledExtendedBuiltin<"acosh", OpenCL_std, 1, 1, 1>;
234: defm : DemangledExtendedBuiltin<"acospi", OpenCL_std, 2, 1, 1>;
235: defm : DemangledExtendedBuiltin<"asin", OpenCL_std, 3, 1, 1>;
236: defm : DemangledExtendedBuiltin<"asinh", OpenCL_std, 4, 1, 1>;
237: defm : DemangledExtendedBuiltin<"asinpi", OpenCL_std, 5, 1, 1>;
238: defm : DemangledExtendedBuiltin<"atan", OpenCL_std, 6, 1, 1>;
239: defm : DemangledExtendedBuiltin<"atanh", OpenCL_std, 8, 1, 1>;
240: defm : DemangledExtendedBuiltin<"atanpi", OpenCL_std, 9, 1, 1>;
241: defm : DemangledExtendedBuiltin<"cbrt", OpenCL_std, 11, 1, 1>;
242: defm : DemangledExtendedBuiltin<"ceil", OpenCL_std, 12, 1, 1>;
243: defm : DemangledExtendedBuiltin<"cos", OpenCL_std, 14, 1, 1>;
244: defm : DemangledExtendedBuiltin<"cosh", OpenCL_std, 15, 1, 1>;
245: defm : DemangledExtendedBuiltin<"cospi", OpenCL_std, 16, 1, 1>;
246: defm : DemangledExtendedBuiltin<"erfc", OpenCL_std, 17, 1, 1>;
247: defm : DemangledExtendedBuiltin<"erf", OpenCL_std, 18, 1, 1>;
248: defm : DemangledExtendedBuiltin<"exp", OpenCL_std, 19, 1, 1>;
249: defm : DemangledExtendedBuiltin<"exp2", OpenCL_std, 20, 1, 1>;
250: defm : DemangledExtendedBuiltin<"exp10", OpenCL_std, 21, 1, 1>;
251: defm : DemangledExtendedBuiltin<"expm1", OpenCL_std, 22, 1, 1>;
252: defm : DemangledExtendedBuiltin<"fabs", OpenCL_std, 23, 1, 1>;
253: defm : DemangledExtendedBuiltin<"floor", OpenCL_std, 25, 1, 1>;
254: defm : DemangledExtendedBuiltin<"ilogb", OpenCL_std, 33, 1, 1>;
255: defm : DemangledExtendedBuiltin<"lgamma", OpenCL_std, 35, 1, 1>;
256: defm : DemangledExtendedBuiltin<"log", OpenCL_std, 37, 1, 1>;
257: defm : DemangledExtendedBuiltin<"log2", OpenCL_std, 38, 1, 1>;
258: defm : DemangledExtendedBuiltin<"log10", OpenCL_std, 39, 1, 1>;
259: defm : DemangledExtendedBuiltin<"log1p", OpenCL_std, 40, 1, 1>;
260: defm : DemangledExtendedBuiltin<"logb", OpenCL_std, 41, 1, 1>;
261: defm : DemangledExtendedBuiltin<"nan", OpenCL_std, 46, 1, 1>;
262: defm : DemangledExtendedBuiltin<"rint", OpenCL_std, 53, 1, 1>;
263: defm : DemangledExtendedBuiltin<"round", OpenCL_std, 55, 1, 1>;
264: defm : DemangledExtendedBuiltin<"rsqrt", OpenCL_std, 56, 1, 1>;
265: defm : DemangledExtendedBuiltin<"sin", OpenCL_std, 57, 1, 1>;
266: defm : DemangledExtendedBuiltin<"sinh", OpenCL_std, 59, 1, 1>;
267: defm : DemangledExtendedBuiltin<"sinpi", OpenCL_std, 60, 1, 1>;
268: defm : DemangledExtendedBuiltin<"sqrt", OpenCL_std, 61, 1, 1>;
269: defm : DemangledExtendedBuiltin<"tan", OpenCL_std, 62, 1, 1>;
270: defm : DemangledExtendedBuiltin<"tanh", OpenCL_std, 63, 1, 1>;
```
- EN: This range uses TableGen DSL to describe records such as OpenCLExtInst, GLSLExtInst, NonSemanticExtInst, DemangledExtendedBuiltin; LLVM later expands these declarations into generated backend tables and helper code.
- CN: 这一段使用 TableGen DSL 描述 OpenCLExtInst、GLSLExtInst、NonSemanticExtInst、DemangledExtendedBuiltin 等记录；LLVM 后续会把这些声明展开为生成的后端表和辅助代码。

### Lines 271-360
```tablegen
271: defm : DemangledExtendedBuiltin<"tanpi", OpenCL_std, 64, 1, 1>;
272: defm : DemangledExtendedBuiltin<"tgamma", OpenCL_std, 65, 1, 1>;
273: defm : DemangledExtendedBuiltin<"trunc", OpenCL_std, 66, 1, 1>;
274: defm : DemangledExtendedBuiltin<"half_cos", OpenCL_std, 67, 1, 1>;
275: defm : DemangledExtendedBuiltin<"half_exp", OpenCL_std, 69, 1, 1>;
276: defm : DemangledExtendedBuiltin<"half_exp2", OpenCL_std, 70, 1, 1>;
277: defm : DemangledExtendedBuiltin<"half_exp10", OpenCL_std, 71, 1, 1>;
278: defm : DemangledExtendedBuiltin<"half_log", OpenCL_std, 72, 1, 1>;
279: defm : DemangledExtendedBuiltin<"half_log2", OpenCL_std, 73, 1, 1>;
280: defm : DemangledExtendedBuiltin<"half_log10", OpenCL_std, 74, 1, 1>;
281: defm : DemangledExtendedBuiltin<"half_recip", OpenCL_std, 76, 1, 1>;
282: defm : DemangledExtendedBuiltin<"half_rsqrt", OpenCL_std, 77, 1, 1>;
283: defm : DemangledExtendedBuiltin<"half_sin", OpenCL_std, 78, 1, 1>;
284: defm : DemangledExtendedBuiltin<"half_sqrt", OpenCL_std, 79, 1, 1>;
285: defm : DemangledExtendedBuiltin<"half_tan", OpenCL_std, 80, 1, 1>;
286: defm : DemangledExtendedBuiltin<"native_cos", OpenCL_std, 81, 1, 1>;
287: defm : DemangledExtendedBuiltin<"native_exp", OpenCL_std, 83, 1, 1>;
288: defm : DemangledExtendedBuiltin<"native_exp2", OpenCL_std, 84, 1, 1>;
289: defm : DemangledExtendedBuiltin<"native_exp10", OpenCL_std, 85, 1, 1>;
290: defm : DemangledExtendedBuiltin<"native_log", OpenCL_std, 86, 1, 1>;
291: defm : DemangledExtendedBuiltin<"native_log2", OpenCL_std, 87, 1, 1>;
292: defm : DemangledExtendedBuiltin<"native_log10", OpenCL_std, 88, 1, 1>;
293: defm : DemangledExtendedBuiltin<"native_recip", OpenCL_std, 90, 1, 1>;
294: defm : DemangledExtendedBuiltin<"native_rsqrt", OpenCL_std, 91, 1, 1>;
295: defm : DemangledExtendedBuiltin<"native_sin", OpenCL_std, 92, 1, 1>;
296: defm : DemangledExtendedBuiltin<"native_sqrt", OpenCL_std, 93, 1, 1>;
297: defm : DemangledExtendedBuiltin<"native_tan", OpenCL_std, 94, 1, 1>;
298: defm : DemangledExtendedBuiltin<"s_abs", OpenCL_std, 141, 1, 1>;
299: defm : DemangledExtendedBuiltin<"clz", OpenCL_std, 151, 1, 1>;
300: defm : DemangledExtendedBuiltin<"ctz", OpenCL_std, 152, 1, 1>;
301: defm : DemangledExtendedBuiltin<"popcount", OpenCL_std, 166, 1, 1>;
302: defm : DemangledExtendedBuiltin<"u_abs", OpenCL_std, 201, 1, 1>;
303: defm : DemangledExtendedBuiltin<"degrees", OpenCL_std, 96, 1, 1>;
304: defm : DemangledExtendedBuiltin<"radians", OpenCL_std, 100, 1, 1>;
305: defm : DemangledExtendedBuiltin<"sign", OpenCL_std, 103, 1, 1>;
306: defm : DemangledExtendedBuiltin<"length", OpenCL_std, 106, 1, 1>;
307: defm : DemangledExtendedBuiltin<"normalize", OpenCL_std, 107, 1, 1>;
308: defm : DemangledExtendedBuiltin<"fast_length", OpenCL_std, 109, 1, 1>;
309: defm : DemangledExtendedBuiltin<"fast_normalize", OpenCL_std, 110, 1, 1>;
310: // OpenCL math builtins (2 args).
311: defm : DemangledExtendedBuiltin<"atan2", OpenCL_std, 7, 2, 2>;
312: defm : DemangledExtendedBuiltin<"atan2pi", OpenCL_std, 10, 2, 2>;
313: defm : DemangledExtendedBuiltin<"copysign", OpenCL_std, 13, 2, 2>;
314: defm : DemangledExtendedBuiltin<"fdim", OpenCL_std, 24, 2, 2>;
315: defm : DemangledExtendedBuiltin<"fmax", OpenCL_std, 27, 2, 2>;
316: defm : DemangledExtendedBuiltin<"fmin", OpenCL_std, 28, 2, 2>;
317: defm : DemangledExtendedBuiltin<"fmod", OpenCL_std, 29, 2, 2>;
318: defm : DemangledExtendedBuiltin<"fract", OpenCL_std, 30, 2, 2>;
319: defm : DemangledExtendedBuiltin<"frexp", OpenCL_std, 31, 2, 2>;
320: defm : DemangledExtendedBuiltin<"hypot", OpenCL_std, 32, 2, 2>;
321: defm : DemangledExtendedBuiltin<"ldexp", OpenCL_std, 34, 2, 2>;
322: defm : DemangledExtendedBuiltin<"lgamma_r", OpenCL_std, 36, 2, 2>;
323: defm : DemangledExtendedBuiltin<"maxmag", OpenCL_std, 43, 2, 2>;
324: defm : DemangledExtendedBuiltin<"minmag", OpenCL_std, 44, 2, 2>;
325: defm : DemangledExtendedBuiltin<"modf", OpenCL_std, 45, 2, 2>;
326: defm : DemangledExtendedBuiltin<"nextafter", OpenCL_std, 47, 2, 2>;
327: defm : DemangledExtendedBuiltin<"pow", OpenCL_std, 48, 2, 2>;
328: defm : DemangledExtendedBuiltin<"pown", OpenCL_std, 49, 2, 2>;
329: defm : DemangledExtendedBuiltin<"powr", OpenCL_std, 50, 2, 2>;
330: defm : DemangledExtendedBuiltin<"remainder", OpenCL_std, 51, 2, 2>;
331: defm : DemangledExtendedBuiltin<"rootn", OpenCL_std, 54, 2, 2>;
332: defm : DemangledExtendedBuiltin<"sincos", OpenCL_std, 58, 2, 2>;
333: defm : DemangledExtendedBuiltin<"half_divide", OpenCL_std, 68, 2, 2>;
334: defm : DemangledExtendedBuiltin<"half_powr", OpenCL_std, 75, 2, 2>;
335: defm : DemangledExtendedBuiltin<"native_divide", OpenCL_std, 82, 2, 2>;
336: defm : DemangledExtendedBuiltin<"native_powr", OpenCL_std, 89, 2, 2>;
337: defm : DemangledExtendedBuiltin<"s_abs_diff", OpenCL_std, 142, 2, 2>;
338: defm : DemangledExtendedBuiltin<"s_add_sat", OpenCL_std, 143, 2, 2>;
339: defm : DemangledExtendedBuiltin<"u_add_sat", OpenCL_std, 144, 2, 2>;
340: defm : DemangledExtendedBuiltin<"s_hadd", OpenCL_std, 145, 2, 2>;
341: defm : DemangledExtendedBuiltin<"u_hadd", OpenCL_std, 146, 2, 2>;
342: defm : DemangledExtendedBuiltin<"s_rhadd", OpenCL_std, 147, 2, 2>;
343: defm : DemangledExtendedBuiltin<"u_rhadd", OpenCL_std, 148, 2, 2>;
344: defm : DemangledExtendedBuiltin<"s_max", OpenCL_std, 156, 2, 2>;
345: defm : DemangledExtendedBuiltin<"u_max", OpenCL_std, 157, 2, 2>;
346: defm : DemangledExtendedBuiltin<"s_min", OpenCL_std, 158, 2, 2>;
347: defm : DemangledExtendedBuiltin<"u_min", OpenCL_std, 159, 2, 2>;
348: defm : DemangledExtendedBuiltin<"s_mul_hi", OpenCL_std, 160, 2, 2>;
349: defm : DemangledExtendedBuiltin<"rotate", OpenCL_std, 161, 2, 2>;
350: defm : DemangledExtendedBuiltin<"s_sub_sat", OpenCL_std, 162, 2, 2>;
351: defm : DemangledExtendedBuiltin<"u_sub_sat", OpenCL_std, 163, 2, 2>;
352: defm : DemangledExtendedBuiltin<"u_upsample", OpenCL_std, 164, 2, 2>;
353: defm : DemangledExtendedBuiltin<"s_upsample", OpenCL_std, 165, 2, 2>;
354: defm : DemangledExtendedBuiltin<"s_mul24", OpenCL_std, 169, 2, 2>;
355: defm : DemangledExtendedBuiltin<"u_mul24", OpenCL_std, 170, 2, 2>;
356: defm : DemangledExtendedBuiltin<"u_abs_diff", OpenCL_std, 202, 2, 2>;
357: defm : DemangledExtendedBuiltin<"u_mul_hi", OpenCL_std, 203, 2, 2>;
358: defm : DemangledExtendedBuiltin<"fmax_common", OpenCL_std, 97, 2, 2>;
359: defm : DemangledExtendedBuiltin<"fmin_common", OpenCL_std, 98, 2, 2>;
360: defm : DemangledExtendedBuiltin<"step", OpenCL_std, 101, 2, 2>;
```
- EN: This range uses TableGen DSL to describe records such as backend logic; LLVM later expands these declarations into generated backend tables and helper code.
- CN: 这一段使用 TableGen DSL 描述 后端逻辑 等记录；LLVM 后续会把这些声明展开为生成的后端表和辅助代码。

### Lines 361-450
```tablegen
361: defm : DemangledExtendedBuiltin<"cross", OpenCL_std, 104, 2, 2>;
362: defm : DemangledExtendedBuiltin<"distance", OpenCL_std, 105, 2, 2>;
363: defm : DemangledExtendedBuiltin<"fast_distance", OpenCL_std, 108, 2, 2>;
364: defm : DemangledExtendedBuiltin<"shuffle", OpenCL_std, 182, 2, 2>;
365: defm : DemangledExtendedBuiltin<"prefetch", OpenCL_std, 185, 2, 2>;
366: defm : DemangledExtendedBuiltin<"vloadn", OpenCL_std, 171, 2, 2>;
367: defm : DemangledExtendedBuiltin<"vload_half", OpenCL_std, 173, 2, 2>;
368: defm : DemangledExtendedBuiltin<"vload_halfn", OpenCL_std, 174, 2, 2>;
369: defm : DemangledExtendedBuiltin<"vloada_halfn", OpenCL_std, 179, 2, 2>;
370: // OpenCL math builtins (3 args).
371: defm : DemangledExtendedBuiltin<"fma", OpenCL_std, 26, 3, 3>;
372: defm : DemangledExtendedBuiltin<"mad", OpenCL_std, 42, 3, 3>;
373: defm : DemangledExtendedBuiltin<"remquo", OpenCL_std, 52, 3, 3>;
374: defm : DemangledExtendedBuiltin<"s_clamp", OpenCL_std, 149, 3, 3>;
375: defm : DemangledExtendedBuiltin<"u_clamp", OpenCL_std, 150, 3, 3>;
376: defm : DemangledExtendedBuiltin<"s_mad_hi", OpenCL_std, 153, 3, 3>;
377: defm : DemangledExtendedBuiltin<"u_mad_sat", OpenCL_std, 154, 3, 3>;
378: defm : DemangledExtendedBuiltin<"s_mad_sat", OpenCL_std, 155, 3, 3>;
379: defm : DemangledExtendedBuiltin<"s_mad24", OpenCL_std, 167, 3, 3>;
380: defm : DemangledExtendedBuiltin<"u_mad24", OpenCL_std, 168, 3, 3>;
381: defm : DemangledExtendedBuiltin<"u_mad_hi", OpenCL_std, 204, 3, 3>;
382: defm : DemangledExtendedBuiltin<"fclamp", OpenCL_std, 95, 3, 3>;
383: defm : DemangledExtendedBuiltin<"mix", OpenCL_std, 99, 3, 3>;
384: defm : DemangledExtendedBuiltin<"smoothstep", OpenCL_std, 102, 3, 3>;
385: defm : DemangledExtendedBuiltin<"bitselect", OpenCL_std, 186, 3, 3>;
386: defm : DemangledExtendedBuiltin<"select", OpenCL_std, 187, 3, 3>;
387: defm : DemangledExtendedBuiltin<"shuffle2", OpenCL_std, 183, 3, 3>;
388: defm : DemangledExtendedBuiltin<"vstoren", OpenCL_std, 172, 3, 3>;
389: defm : DemangledExtendedBuiltin<"vstore_half", OpenCL_std, 175, 3, 3>;
390: defm : DemangledExtendedBuiltin<"vstore_half_r", OpenCL_std, 176, 3, 3>;
391: defm : DemangledExtendedBuiltin<"vstore_halfn", OpenCL_std, 177, 3, 3>;
392: defm : DemangledExtendedBuiltin<"vstore_halfn_r", OpenCL_std, 178, 3, 3>;
393: defm : DemangledExtendedBuiltin<"vstorea_halfn", OpenCL_std, 180, 3, 3>;
394: defm : DemangledExtendedBuiltin<"vstorea_halfn_r", OpenCL_std, 181, 3, 3>;
395: // OpenCL printf (variadic: 1+).
396: defm : DemangledExtendedBuiltin<"printf", OpenCL_std, 184, 1, 0>;
397:
398: defm : DemangledExtendedBuiltin<"Round", GLSL_std_450, 1>;
399: defm : DemangledExtendedBuiltin<"RoundEven", GLSL_std_450, 2>;
400: defm : DemangledExtendedBuiltin<"Trunc", GLSL_std_450, 3>;
401: defm : DemangledExtendedBuiltin<"FAbs", GLSL_std_450, 4>;
402: defm : DemangledExtendedBuiltin<"SAbs", GLSL_std_450, 5>;
403: defm : DemangledExtendedBuiltin<"FSign", GLSL_std_450, 6>;
404: defm : DemangledExtendedBuiltin<"SSign", GLSL_std_450, 7>;
405: defm : DemangledExtendedBuiltin<"Floor", GLSL_std_450, 8>;
406: defm : DemangledExtendedBuiltin<"Ceil", GLSL_std_450, 9>;
407: defm : DemangledExtendedBuiltin<"Fract", GLSL_std_450, 10>;
408: defm : DemangledExtendedBuiltin<"Radians", GLSL_std_450, 11>;
409: defm : DemangledExtendedBuiltin<"Degrees", GLSL_std_450, 12>;
410: defm : DemangledExtendedBuiltin<"Sin", GLSL_std_450, 13>;
411: defm : DemangledExtendedBuiltin<"Cos", GLSL_std_450, 14>;
412: defm : DemangledExtendedBuiltin<"Tan", GLSL_std_450, 15>;
413: defm : DemangledExtendedBuiltin<"Asin", GLSL_std_450, 16>;
414: defm : DemangledExtendedBuiltin<"Acos", GLSL_std_450, 17>;
415: defm : DemangledExtendedBuiltin<"Atan", GLSL_std_450, 18>;
416: defm : DemangledExtendedBuiltin<"Sinh", GLSL_std_450, 19>;
417: defm : DemangledExtendedBuiltin<"Cosh", GLSL_std_450, 20>;
418: defm : DemangledExtendedBuiltin<"Tanh", GLSL_std_450, 21>;
419: defm : DemangledExtendedBuiltin<"Asinh", GLSL_std_450, 22>;
420: defm : DemangledExtendedBuiltin<"Acosh", GLSL_std_450, 23>;
421: defm : DemangledExtendedBuiltin<"Atanh", GLSL_std_450, 24>;
422: defm : DemangledExtendedBuiltin<"Atan2", GLSL_std_450, 25>;
423: defm : DemangledExtendedBuiltin<"Pow", GLSL_std_450, 26>;
424: defm : DemangledExtendedBuiltin<"Exp", GLSL_std_450, 27>;
425: defm : DemangledExtendedBuiltin<"Log", GLSL_std_450, 28>;
426: defm : DemangledExtendedBuiltin<"Exp2", GLSL_std_450, 29>;
427: defm : DemangledExtendedBuiltin<"Log2", GLSL_std_450, 30>;
428: defm : DemangledExtendedBuiltin<"Sqrt", GLSL_std_450, 31>;
429: defm : DemangledExtendedBuiltin<"InverseSqrt", GLSL_std_450, 32>;
430: defm : DemangledExtendedBuiltin<"Determinant", GLSL_std_450, 33>;
431: defm : DemangledExtendedBuiltin<"MatrixInverse", GLSL_std_450, 34>;
432: defm : DemangledExtendedBuiltin<"Modf", GLSL_std_450, 35>;
433: defm : DemangledExtendedBuiltin<"ModfStruct", GLSL_std_450, 36>;
434: defm : DemangledExtendedBuiltin<"FMin", GLSL_std_450, 37>;
435: defm : DemangledExtendedBuiltin<"UMin", GLSL_std_450, 38>;
436: defm : DemangledExtendedBuiltin<"SMin", GLSL_std_450, 39>;
437: defm : DemangledExtendedBuiltin<"FMax", GLSL_std_450, 40>;
438: defm : DemangledExtendedBuiltin<"UMax", GLSL_std_450, 41>;
439: defm : DemangledExtendedBuiltin<"SMax", GLSL_std_450, 42>;
440: defm : DemangledExtendedBuiltin<"FClamp", GLSL_std_450, 43>;
441: defm : DemangledExtendedBuiltin<"UClamp", GLSL_std_450, 44>;
442: defm : DemangledExtendedBuiltin<"SClamp", GLSL_std_450, 45>;
443: defm : DemangledExtendedBuiltin<"FMix", GLSL_std_450, 46>;
444: defm : DemangledExtendedBuiltin<"Step", GLSL_std_450, 48>;
445: defm : DemangledExtendedBuiltin<"SmoothStep", GLSL_std_450, 49>;
446: defm : DemangledExtendedBuiltin<"Fma", GLSL_std_450, 50>;
447: defm : DemangledExtendedBuiltin<"Frexp", GLSL_std_450, 51>;
448: defm : DemangledExtendedBuiltin<"FrexpStruct", GLSL_std_450, 52>;
449: defm : DemangledExtendedBuiltin<"Ldexp", GLSL_std_450, 53>;
450: defm : DemangledExtendedBuiltin<"PackSnorm4x8", GLSL_std_450, 54>;
```
- EN: This range uses TableGen DSL to describe records such as backend logic; LLVM later expands these declarations into generated backend tables and helper code.
- CN: 这一段使用 TableGen DSL 描述 后端逻辑 等记录；LLVM 后续会把这些声明展开为生成的后端表和辅助代码。

### Lines 451-540
```tablegen
451: defm : DemangledExtendedBuiltin<"PackUnorm4x8", GLSL_std_450, 55>;
452: defm : DemangledExtendedBuiltin<"PackSnorm2x16", GLSL_std_450, 56>;
453: defm : DemangledExtendedBuiltin<"PackUnorm2x16", GLSL_std_450, 57>;
454: defm : DemangledExtendedBuiltin<"PackHalf2x16", GLSL_std_450, 58>;
455: defm : DemangledExtendedBuiltin<"PackDouble2x32", GLSL_std_450, 59>;
456: defm : DemangledExtendedBuiltin<"UnpackSnorm2x16", GLSL_std_450, 60>;
457: defm : DemangledExtendedBuiltin<"UnpackUnorm2x16", GLSL_std_450, 61>;
458: defm : DemangledExtendedBuiltin<"UnpackHalf2x16", GLSL_std_450, 62>;
459: defm : DemangledExtendedBuiltin<"UnpackSnorm4x8", GLSL_std_450, 63>;
460: defm : DemangledExtendedBuiltin<"UnpackUnorm4x8", GLSL_std_450, 64>;
461: defm : DemangledExtendedBuiltin<"UnpackDouble2x32", GLSL_std_450, 65>;
462: defm : DemangledExtendedBuiltin<"Length", GLSL_std_450, 66>;
463: defm : DemangledExtendedBuiltin<"Distance", GLSL_std_450, 67>;
464: defm : DemangledExtendedBuiltin<"Cross", GLSL_std_450, 68>;
465: defm : DemangledExtendedBuiltin<"Normalize", GLSL_std_450, 69>;
466: defm : DemangledExtendedBuiltin<"FaceForward", GLSL_std_450, 70>;
467: defm : DemangledExtendedBuiltin<"Reflect", GLSL_std_450, 71>;
468: defm : DemangledExtendedBuiltin<"Refract", GLSL_std_450, 72>;
469: defm : DemangledExtendedBuiltin<"FindILsb", GLSL_std_450, 73>;
470: defm : DemangledExtendedBuiltin<"FindSMsb", GLSL_std_450, 74>;
471: defm : DemangledExtendedBuiltin<"FindUMsb", GLSL_std_450, 75>;
472: defm : DemangledExtendedBuiltin<"InterpolateAtCentroid", GLSL_std_450, 76>;
473: defm : DemangledExtendedBuiltin<"InterpolateAtSample", GLSL_std_450, 77>;
474: defm : DemangledExtendedBuiltin<"InterpolateAtOffset", GLSL_std_450, 78>;
475: defm : DemangledExtendedBuiltin<"NMin", GLSL_std_450, 79>;
476: defm : DemangledExtendedBuiltin<"NMax", GLSL_std_450, 80>;
477: defm : DemangledExtendedBuiltin<"NClamp", GLSL_std_450, 81>;
478:
479: defm : DemangledExtendedBuiltin<"DebugInfoNone", NonSemantic_Shader_DebugInfo_100, 0>;
480: defm : DemangledExtendedBuiltin<"DebugCompilationUnit", NonSemantic_Shader_DebugInfo_100, 1>;
481: defm : DemangledExtendedBuiltin<"DebugTypeBasic", NonSemantic_Shader_DebugInfo_100, 2>;
482: defm : DemangledExtendedBuiltin<"DebugTypePointer", NonSemantic_Shader_DebugInfo_100, 3>;
483: defm : DemangledExtendedBuiltin<"DebugTypeQualifier", NonSemantic_Shader_DebugInfo_100, 4>;
484: defm : DemangledExtendedBuiltin<"DebugTypeArray", NonSemantic_Shader_DebugInfo_100, 5>;
485: defm : DemangledExtendedBuiltin<"DebugTypeVector", NonSemantic_Shader_DebugInfo_100, 6>;
486: defm : DemangledExtendedBuiltin<"DebugTypedef", NonSemantic_Shader_DebugInfo_100, 7>;
487: defm : DemangledExtendedBuiltin<"DebugTypeFunction", NonSemantic_Shader_DebugInfo_100, 8>;
488: defm : DemangledExtendedBuiltin<"DebugTypeEnum", NonSemantic_Shader_DebugInfo_100, 9>;
489: defm : DemangledExtendedBuiltin<"DebugTypeComposite", NonSemantic_Shader_DebugInfo_100, 10>;
490: defm : DemangledExtendedBuiltin<"DebugTypeMember", NonSemantic_Shader_DebugInfo_100, 11>;
491: defm : DemangledExtendedBuiltin<"DebugTypeInheritance", NonSemantic_Shader_DebugInfo_100, 12>;
492: defm : DemangledExtendedBuiltin<"DebugTypePtrToMember", NonSemantic_Shader_DebugInfo_100, 13>;
493: defm : DemangledExtendedBuiltin<"DebugTypeTemplate", NonSemantic_Shader_DebugInfo_100, 14>;
494: defm : DemangledExtendedBuiltin<"DebugTypeTemplateParameter", NonSemantic_Shader_DebugInfo_100, 15>;
495: defm : DemangledExtendedBuiltin<"DebugTypeTemplateTemplateParameter", NonSemantic_Shader_DebugInfo_100, 16>;
496: defm : DemangledExtendedBuiltin<"DebugTypeTemplateParameterPack", NonSemantic_Shader_DebugInfo_100, 17>;
497: defm : DemangledExtendedBuiltin<"DebugGlobalVariable", NonSemantic_Shader_DebugInfo_100, 18>;
498: defm : DemangledExtendedBuiltin<"DebugFunctionDeclaration", NonSemantic_Shader_DebugInfo_100, 19>;
499: defm : DemangledExtendedBuiltin<"DebugFunction", NonSemantic_Shader_DebugInfo_100, 20>;
500: defm : DemangledExtendedBuiltin<"DebugLexicalBlock", NonSemantic_Shader_DebugInfo_100, 21>;
501: defm : DemangledExtendedBuiltin<"DebugLexicalBlockDiscriminator", NonSemantic_Shader_DebugInfo_100, 22>;
502: defm : DemangledExtendedBuiltin<"DebugScope", NonSemantic_Shader_DebugInfo_100, 23>;
503: defm : DemangledExtendedBuiltin<"DebugNoScope", NonSemantic_Shader_DebugInfo_100, 24>;
504: defm : DemangledExtendedBuiltin<"DebugInlinedAt", NonSemantic_Shader_DebugInfo_100, 25>;
505: defm : DemangledExtendedBuiltin<"DebugLocalVariable", NonSemantic_Shader_DebugInfo_100, 26>;
506: defm : DemangledExtendedBuiltin<"DebugInlinedVariable", NonSemantic_Shader_DebugInfo_100, 27>;
507: defm : DemangledExtendedBuiltin<"DebugDeclare", NonSemantic_Shader_DebugInfo_100, 28>;
508: defm : DemangledExtendedBuiltin<"DebugValue", NonSemantic_Shader_DebugInfo_100, 29>;
509: defm : DemangledExtendedBuiltin<"DebugOperation", NonSemantic_Shader_DebugInfo_100, 30>;
510: defm : DemangledExtendedBuiltin<"DebugExpression", NonSemantic_Shader_DebugInfo_100, 31>;
511: defm : DemangledExtendedBuiltin<"DebugMacroDef", NonSemantic_Shader_DebugInfo_100, 32>;
512: defm : DemangledExtendedBuiltin<"DebugMacroUndef", NonSemantic_Shader_DebugInfo_100, 33>;
513: defm : DemangledExtendedBuiltin<"DebugImportedEntity", NonSemantic_Shader_DebugInfo_100, 34>;
514: defm : DemangledExtendedBuiltin<"DebugSource", NonSemantic_Shader_DebugInfo_100, 35>;
515: defm : DemangledExtendedBuiltin<"DebugFunctionDefinition", NonSemantic_Shader_DebugInfo_100, 101>;
516: defm : DemangledExtendedBuiltin<"DebugSourceContinued", NonSemantic_Shader_DebugInfo_100, 102>;
517: defm : DemangledExtendedBuiltin<"DebugLine", NonSemantic_Shader_DebugInfo_100, 103>;
518: defm : DemangledExtendedBuiltin<"DebugNoLine", NonSemantic_Shader_DebugInfo_100, 104>;
519: defm : DemangledExtendedBuiltin<"DebugBuildIdentifier", NonSemantic_Shader_DebugInfo_100, 105>;
520: defm : DemangledExtendedBuiltin<"DebugStoragePath", NonSemantic_Shader_DebugInfo_100, 106>;
521: defm : DemangledExtendedBuiltin<"DebugEntryPoint", NonSemantic_Shader_DebugInfo_100, 107>;
522: defm : DemangledExtendedBuiltin<"DebugTypeMatrix", NonSemantic_Shader_DebugInfo_100, 108>;
523: //===----------------------------------------------------------------------===//
524: // Class defining an native builtin record used for direct translation into a
525: // SPIR-V instruction.
526: //
527: // name is the demangled name of the given builtin.
528: // set specifies which external instruction set the builtin belongs to.
529: // opcode specifies the SPIR-V operation code of the generated instruction.
530: //===----------------------------------------------------------------------===//
531: class NativeBuiltin<string name, InstructionSet set, Op operation> {
532:   string Name = name;
533:   InstructionSet Set = set;
534:   Op Opcode = operation;
535: }
536:
537: // Table gathering all the native builtins.
538: def NativeBuiltins : GenericTable {
539:   let FilterClass = "NativeBuiltin";
540:   let Fields = ["Name", "Set", "Opcode"];
```
- EN: This range uses TableGen DSL to describe records such as NativeBuiltin, NativeBuiltins; LLVM later expands these declarations into generated backend tables and helper code.
- CN: 这一段使用 TableGen DSL 描述 NativeBuiltin、NativeBuiltins 等记录；LLVM 后续会把这些声明展开为生成的后端表和辅助代码。

### Lines 541-630
```tablegen
541:   string TypeOf_Set = "InstructionSet";
542: }
543:
544: // Function to lookup native builtins by their name and set.
545: def lookupNativeBuiltin : SearchIndex {
546:   let Table = NativeBuiltins;
547:   let Key = ["Name", "Set"];
548: }
549:
550: // Multiclass used to define at the same time both an incoming builtin record
551: // and a corresponding native builtin record.
552: multiclass DemangledNativeBuiltin<string name, InstructionSet set, BuiltinGroup group, bits<8> minNumArgs, bits<8> maxNumArgs, Op operation> {
553:   def : DemangledBuiltin<name, set, group, minNumArgs, maxNumArgs>;
554:   def : NativeBuiltin<name, set, operation>;
555: }
556:
557: // Relational builtin records:
558: defm : DemangledNativeBuiltin<"isequal", OpenCL_std, Relational, 2, 2, OpFOrdEqual>;
559: defm : DemangledNativeBuiltin<"__spirv_FOrdEqual", OpenCL_std, Relational, 2, 2, OpFOrdEqual>;
560: defm : DemangledNativeBuiltin<"isnotequal", OpenCL_std, Relational, 2, 2, OpFUnordNotEqual>;
561: defm : DemangledNativeBuiltin<"__spirv_FUnordNotEqual", OpenCL_std, Relational, 2, 2, OpFUnordNotEqual>;
562: defm : DemangledNativeBuiltin<"isgreater", OpenCL_std, Relational, 2, 2, OpFOrdGreaterThan>;
563: defm : DemangledNativeBuiltin<"__spirv_FOrdGreaterThan", OpenCL_std, Relational, 2, 2, OpFOrdGreaterThan>;
564: defm : DemangledNativeBuiltin<"isgreaterequal", OpenCL_std, Relational, 2, 2, OpFOrdGreaterThanEqual>;
565: defm : DemangledNativeBuiltin<"__spirv_FOrdGreaterThanEqual", OpenCL_std, Relational, 2, 2, OpFOrdGreaterThanEqual>;
566: defm : DemangledNativeBuiltin<"isless", OpenCL_std, Relational, 2, 2, OpFOrdLessThan>;
567: defm : DemangledNativeBuiltin<"__spirv_FOrdLessThan", OpenCL_std, Relational, 2, 2, OpFOrdLessThan>;
568: defm : DemangledNativeBuiltin<"islessequal", OpenCL_std, Relational, 2, 2, OpFOrdLessThanEqual>;
569: defm : DemangledNativeBuiltin<"__spirv_FOrdLessThanEqual", OpenCL_std, Relational, 2, 2, OpFOrdLessThanEqual>;
570: defm : DemangledNativeBuiltin<"islessgreater", OpenCL_std, Relational, 2, 2, OpFOrdNotEqual>;
571: defm : DemangledNativeBuiltin<"__spirv_FOrdNotEqual", OpenCL_std, Relational, 2, 2, OpFOrdNotEqual>;
572: defm : DemangledNativeBuiltin<"isordered", OpenCL_std, Relational, 2, 2, OpOrdered>;
573: defm : DemangledNativeBuiltin<"__spirv_Ordered", OpenCL_std, Relational, 2, 2, OpOrdered>;
574: defm : DemangledNativeBuiltin<"isunordered", OpenCL_std, Relational, 2, 2, OpUnordered>;
575: defm : DemangledNativeBuiltin<"__spirv_Unordered", OpenCL_std, Relational, 2, 2, OpUnordered>;
576: defm : DemangledNativeBuiltin<"isfinite", OpenCL_std, Relational, 1, 1, OpIsFinite>;
577: defm : DemangledNativeBuiltin<"__spirv_IsFinite", OpenCL_std, Relational, 1, 1, OpIsFinite>;
578: defm : DemangledNativeBuiltin<"isinf", OpenCL_std, Relational, 1, 1, OpIsInf>;
579: defm : DemangledNativeBuiltin<"__spirv_IsInf", OpenCL_std, Relational, 1, 1, OpIsInf>;
580: defm : DemangledNativeBuiltin<"isnan", OpenCL_std, Relational, 1, 1, OpIsNan>;
581: defm : DemangledNativeBuiltin<"__spirv_IsNan", OpenCL_std, Relational, 1, 1, OpIsNan>;
582: defm : DemangledNativeBuiltin<"isnormal", OpenCL_std, Relational, 1, 1, OpIsNormal>;
583: defm : DemangledNativeBuiltin<"__spirv_IsNormal", OpenCL_std, Relational, 1, 1, OpIsNormal>;
584: defm : DemangledNativeBuiltin<"signbit", OpenCL_std, Relational, 1, 1, OpSignBitSet>;
585: defm : DemangledNativeBuiltin<"__spirv_SignBitSet", OpenCL_std, Relational, 1, 1, OpSignBitSet>;
586: defm : DemangledNativeBuiltin<"any", OpenCL_std, Relational, 1, 1, OpAny>;
587: defm : DemangledNativeBuiltin<"__spirv_Any", OpenCL_std, Relational, 1, 1, OpAny>;
588: defm : DemangledNativeBuiltin<"all", OpenCL_std, Relational, 1, 1, OpAll>;
589: defm : DemangledNativeBuiltin<"__spirv_All", OpenCL_std, Relational, 1, 1, OpAll>;
590:
591: // Atomic builtin records:
592: defm : DemangledNativeBuiltin<"atomic_init", OpenCL_std, Atomic, 2, 2, OpStore>;
593: defm : DemangledNativeBuiltin<"atomic_load", OpenCL_std, Atomic, 1, 1, OpAtomicLoad>;
594: defm : DemangledNativeBuiltin<"atomic_load_explicit", OpenCL_std, Atomic, 2, 3, OpAtomicLoad>;
595: defm : DemangledNativeBuiltin<"__spirv_AtomicLoad", OpenCL_std, Atomic, 3, 3, OpAtomicLoad>;
596: defm : DemangledNativeBuiltin<"atomic_store", OpenCL_std, Atomic, 2, 2, OpAtomicStore>;
597: defm : DemangledNativeBuiltin<"atomic_store_explicit", OpenCL_std, Atomic, 2, 4, OpAtomicStore>;
598: defm : DemangledNativeBuiltin<"__spirv_AtomicStore", OpenCL_std, Atomic, 4, 4, OpAtomicStore>;
599: defm : DemangledNativeBuiltin<"atomic_compare_exchange_strong", OpenCL_std, Atomic, 3, 6, OpAtomicCompareExchange>;
600: defm : DemangledNativeBuiltin<"__spirv_AtomicCompareExchange", OpenCL_std, Atomic, 6, 6, OpAtomicCompareExchange>;
601: defm : DemangledNativeBuiltin<"atomic_compare_exchange_strong_explicit", OpenCL_std, Atomic, 5, 6, OpAtomicCompareExchange>;
602: defm : DemangledNativeBuiltin<"atomic_compare_exchange_weak", OpenCL_std, Atomic, 3, 6, OpAtomicCompareExchangeWeak>;
603: defm : DemangledNativeBuiltin<"atomic_compare_exchange_weak_explicit", OpenCL_std, Atomic, 5, 6, OpAtomicCompareExchangeWeak>;
604: defm : DemangledNativeBuiltin<"__spirv_AtomicCompareExchangeWeak", OpenCL_std, Atomic, 6, 6, OpAtomicCompareExchangeWeak>;
605: defm : DemangledNativeBuiltin<"atom_cmpxchg", OpenCL_std, Atomic, 3, 6, OpAtomicCompareExchange>;
606: defm : DemangledNativeBuiltin<"atomic_cmpxchg", OpenCL_std, Atomic, 3, 6, OpAtomicCompareExchange>;
607: defm : DemangledNativeBuiltin<"atom_add", OpenCL_std, Atomic, 2, 4, OpAtomicIAdd>;
608: defm : DemangledNativeBuiltin<"atomic_add", OpenCL_std, Atomic, 2, 4, OpAtomicIAdd>;
609: defm : DemangledNativeBuiltin<"__spirv_AtomicIAdd", OpenCL_std, Atomic, 4, 4, OpAtomicIAdd>;
610: defm : DemangledNativeBuiltin<"atom_sub", OpenCL_std, Atomic, 2, 4, OpAtomicISub>;
611: defm : DemangledNativeBuiltin<"atomic_sub", OpenCL_std, Atomic, 2, 4, OpAtomicISub>;
612: defm : DemangledNativeBuiltin<"__spirv_AtomicISub", OpenCL_std, Atomic, 4, 4, OpAtomicISub>;
613: defm : DemangledNativeBuiltin<"atom_or", OpenCL_std, Atomic, 2, 4, OpAtomicOr>;
614: defm : DemangledNativeBuiltin<"atomic_or", OpenCL_std, Atomic, 2, 4, OpAtomicOr>;
615: defm : DemangledNativeBuiltin<"__spirv_AtomicOr", OpenCL_std, Atomic, 4, 4, OpAtomicOr>;
616: defm : DemangledNativeBuiltin<"atom_xor", OpenCL_std, Atomic, 2, 4, OpAtomicXor>;
617: defm : DemangledNativeBuiltin<"atomic_xor", OpenCL_std, Atomic, 2, 4, OpAtomicXor>;
618: defm : DemangledNativeBuiltin<"__spirv_AtomicXor", OpenCL_std, Atomic, 4, 4, OpAtomicXor>;
619: defm : DemangledNativeBuiltin<"atom_and", OpenCL_std, Atomic, 2, 4, OpAtomicAnd>;
620: defm : DemangledNativeBuiltin<"atomic_and", OpenCL_std, Atomic, 2, 4, OpAtomicAnd>;
621: defm : DemangledNativeBuiltin<"__spirv_AtomicAnd", OpenCL_std, Atomic, 4, 4, OpAtomicAnd>;
622: defm : DemangledNativeBuiltin<"atomic_exchange", OpenCL_std, Atomic, 2, 4, OpAtomicExchange>;
623: defm : DemangledNativeBuiltin<"atomic_exchange_explicit", OpenCL_std, Atomic, 2, 4, OpAtomicExchange>;
624: defm : DemangledNativeBuiltin<"AtomicEx__spirv_change", OpenCL_std, Atomic, 2, 4, OpAtomicExchange>;
625: defm : DemangledNativeBuiltin<"__spirv_AtomicExchange", OpenCL_std, Atomic, 4, 4, OpAtomicExchange>;
626: defm : DemangledNativeBuiltin<"atomic_work_item_fence", OpenCL_std, Atomic, 1, 3, OpMemoryBarrier>;
627: defm : DemangledNativeBuiltin<"__spirv_MemoryBarrier", OpenCL_std, Atomic, 2, 2, OpMemoryBarrier>;
628: defm : DemangledNativeBuiltin<"atomic_fetch_add", OpenCL_std, Atomic, 2, 4, OpAtomicIAdd>;
629: defm : DemangledNativeBuiltin<"atomic_fetch_sub", OpenCL_std, Atomic, 2, 4, OpAtomicISub>;
630: defm : DemangledNativeBuiltin<"atomic_fetch_or", OpenCL_std, Atomic, 2, 4, OpAtomicOr>;
```
- EN: This range uses TableGen DSL to describe records such as lookupNativeBuiltin, DemangledNativeBuiltin; LLVM later expands these declarations into generated backend tables and helper code.
- CN: 这一段使用 TableGen DSL 描述 lookupNativeBuiltin、DemangledNativeBuiltin 等记录；LLVM 后续会把这些声明展开为生成的后端表和辅助代码。

### Lines 631-720
```tablegen
631: defm : DemangledNativeBuiltin<"atomic_fetch_xor", OpenCL_std, Atomic, 2, 4, OpAtomicXor>;
632: defm : DemangledNativeBuiltin<"atomic_fetch_and", OpenCL_std, Atomic, 2, 4, OpAtomicAnd>;
633: defm : DemangledNativeBuiltin<"atomic_fetch_add_explicit", OpenCL_std, Atomic, 3, 4, OpAtomicIAdd>;
634: defm : DemangledNativeBuiltin<"atomic_fetch_sub_explicit", OpenCL_std, Atomic, 3, 4, OpAtomicISub>;
635: defm : DemangledNativeBuiltin<"atomic_fetch_or_explicit", OpenCL_std, Atomic, 3, 4, OpAtomicOr>;
636: defm : DemangledNativeBuiltin<"atomic_fetch_xor_explicit", OpenCL_std, Atomic, 3, 4, OpAtomicXor>;
637: defm : DemangledNativeBuiltin<"atomic_fetch_and_explicit", OpenCL_std, Atomic, 3, 4, OpAtomicAnd>;
638: defm : DemangledNativeBuiltin<"s_atomic_fetch_min", OpenCL_std, Atomic, 2, 4, OpAtomicSMin>;
639: defm : DemangledNativeBuiltin<"s_atomic_fetch_max", OpenCL_std, Atomic, 2, 4, OpAtomicSMax>;
640: defm : DemangledNativeBuiltin<"u_atomic_fetch_min", OpenCL_std, Atomic, 2, 4, OpAtomicUMin>;
641: defm : DemangledNativeBuiltin<"u_atomic_fetch_max", OpenCL_std, Atomic, 2, 4, OpAtomicUMax>;
642: defm : DemangledNativeBuiltin<"s_atomic_fetch_min_explicit", OpenCL_std, Atomic, 3, 4, OpAtomicSMin>;
643: defm : DemangledNativeBuiltin<"s_atomic_fetch_max_explicit", OpenCL_std, Atomic, 3, 4, OpAtomicSMax>;
644: defm : DemangledNativeBuiltin<"u_atomic_fetch_min_explicit", OpenCL_std, Atomic, 3, 4, OpAtomicUMin>;
645: defm : DemangledNativeBuiltin<"u_atomic_fetch_max_explicit", OpenCL_std, Atomic, 3, 4, OpAtomicUMax>;
646: defm : DemangledNativeBuiltin<"s_atom_min", OpenCL_std, Atomic, 2, 2, OpAtomicSMin>;
647: defm : DemangledNativeBuiltin<"s_atom_max", OpenCL_std, Atomic, 2, 2, OpAtomicSMax>;
648: defm : DemangledNativeBuiltin<"u_atom_min", OpenCL_std, Atomic, 2, 2, OpAtomicUMin>;
649: defm : DemangledNativeBuiltin<"u_atom_max", OpenCL_std, Atomic, 2, 2, OpAtomicUMax>;
650: defm : DemangledNativeBuiltin<"atomic_flag_test_and_set", OpenCL_std, Atomic, 1, 1, OpAtomicFlagTestAndSet>;
651: defm : DemangledNativeBuiltin<"__spirv_AtomicFlagTestAndSet", OpenCL_std, Atomic, 3, 3, OpAtomicFlagTestAndSet>;
652: defm : DemangledNativeBuiltin<"atomic_flag_test_and_set_explicit", OpenCL_std, Atomic, 2, 3, OpAtomicFlagTestAndSet>;
653: defm : DemangledNativeBuiltin<"atomic_flag_clear", OpenCL_std, Atomic, 1, 1, OpAtomicFlagClear>;
654: defm : DemangledNativeBuiltin<"__spirv_AtomicFlagClear", OpenCL_std, Atomic, 3, 3, OpAtomicFlagClear>;
655: defm : DemangledNativeBuiltin<"atomic_flag_clear_explicit", OpenCL_std, Atomic, 2, 3, OpAtomicFlagClear>;
656: defm : DemangledNativeBuiltin<"__spirv_AtomicSMin", OpenCL_std, Atomic, 4, 4, OpAtomicSMin>;
657: defm : DemangledNativeBuiltin<"__spirv_AtomicSMax", OpenCL_std, Atomic, 4, 4, OpAtomicSMax>;
658: defm : DemangledNativeBuiltin<"__spirv_AtomicUMin", OpenCL_std, Atomic, 4, 4, OpAtomicUMin>;
659: defm : DemangledNativeBuiltin<"__spirv_AtomicUMax", OpenCL_std, Atomic, 4, 4, OpAtomicUMax>;
660:
661: // Pipe builtin records.
662: defm : DemangledNativeBuiltin<"__read_pipe_2", OpenCL_std, Pipe, 4, 4, OpReadPipe>;
663: defm : DemangledNativeBuiltin<"__write_pipe_2", OpenCL_std, Pipe, 4, 4, OpWritePipe>;
664: defm : DemangledNativeBuiltin<"__read_pipe_4", OpenCL_std, Pipe, 6, 6, OpReservedReadPipe>;
665: defm : DemangledNativeBuiltin<"__write_pipe_4", OpenCL_std, Pipe, 6, 6, OpReservedWritePipe>;
666: defm : DemangledNativeBuiltin<"__reserve_read_pipe", OpenCL_std, Pipe, 4, 4, OpReserveReadPipePackets>;
667: defm : DemangledNativeBuiltin<"__reserve_write_pipe", OpenCL_std, Pipe, 4, 4, OpReserveWritePipePackets>;
668: defm : DemangledNativeBuiltin<"__commit_read_pipe", OpenCL_std, Pipe, 4, 4, OpCommitReadPipe>;
669: defm : DemangledNativeBuiltin<"__commit_write_pipe", OpenCL_std, Pipe, 4, 4, OpCommitWritePipe>;
670: defm : DemangledNativeBuiltin<"is_valid_reserve_id", OpenCL_std, Pipe, 1, 1, OpIsValidReserveId>;
671: defm : DemangledNativeBuiltin<"__get_pipe_num_packets_ro", OpenCL_std, Pipe, 3, 3, OpGetNumPipePackets>;
672: defm : DemangledNativeBuiltin<"__get_pipe_max_packets_ro", OpenCL_std, Pipe, 3, 3, OpGetMaxPipePackets>;
673: defm : DemangledNativeBuiltin<"__get_pipe_num_packets_wo", OpenCL_std, Pipe, 3, 3, OpGetNumPipePackets>;
674: defm : DemangledNativeBuiltin<"__get_pipe_max_packets_wo", OpenCL_std, Pipe, 3, 3, OpGetMaxPipePackets>;
675: defm : DemangledNativeBuiltin<"__work_group_reserve_read_pipe", OpenCL_std, Pipe, 4, 4, OpGroupReserveReadPipePackets>;
676: defm : DemangledNativeBuiltin<"__work_group_reserve_write_pipe", OpenCL_std, Pipe, 4, 4, OpGroupReserveWritePipePackets>;
677: defm : DemangledNativeBuiltin<"__work_group_commit_read_pipe", OpenCL_std, Pipe, 4, 4, OpGroupCommitReadPipe>;
678: defm : DemangledNativeBuiltin<"__work_group_commit_write_pipe", OpenCL_std, Pipe, 4, 4, OpGroupCommitWritePipe>;
679: defm : DemangledNativeBuiltin<"__sub_group_reserve_read_pipe", OpenCL_std, Pipe, 4, 4, OpGroupReserveReadPipePackets>;
680: defm : DemangledNativeBuiltin<"__sub_group_reserve_write_pipe", OpenCL_std, Pipe, 4, 4, OpGroupReserveWritePipePackets>;
681: defm : DemangledNativeBuiltin<"__sub_group_commit_read_pipe", OpenCL_std, Pipe, 4, 4, OpGroupCommitReadPipe>;
682: defm : DemangledNativeBuiltin<"__sub_group_commit_write_pipe", OpenCL_std, Pipe, 4, 4, OpGroupCommitWritePipe>;
683:
684: // Barrier builtin records:
685: defm : DemangledNativeBuiltin<"barrier", OpenCL_std, Barrier, 1, 3, OpControlBarrier>;
686: defm : DemangledNativeBuiltin<"work_group_barrier", OpenCL_std, Barrier, 1, 3, OpControlBarrier>;
687: defm : DemangledNativeBuiltin<"__spirv_ControlBarrier", OpenCL_std, Barrier, 3, 3, OpControlBarrier>;
688:
689: // ICarryBorrow builtin record:
690: defm : DemangledNativeBuiltin<"__spirv_IAddCarry", OpenCL_std, ICarryBorrow, 3, 3, OpIAddCarryS>;
691: defm : DemangledNativeBuiltin<"__spirv_IAddCarry", GLSL_std_450, ICarryBorrow, 3, 3, OpIAddCarryS>;
692: defm : DemangledNativeBuiltin<"__spirv_ISubBorrow", OpenCL_std, ICarryBorrow, 3, 3, OpISubBorrowS>;
693: defm : DemangledNativeBuiltin<"__spirv_ISubBorrow", GLSL_std_450, ICarryBorrow, 3, 3, OpISubBorrowS>;
694:
695: // MulExtended builtin records:
696: defm : DemangledNativeBuiltin<"__spirv_UMulExtended", OpenCL_std, MulExtended, 2, 3, OpUMulExtended>;
697: defm : DemangledNativeBuiltin<"__spirv_UMulExtended", GLSL_std_450, MulExtended, 2, 3, OpUMulExtended>;
698: defm : DemangledNativeBuiltin<"__spirv_SMulExtended", OpenCL_std, MulExtended, 2, 3, OpSMulExtended>;
699: defm : DemangledNativeBuiltin<"__spirv_SMulExtended", GLSL_std_450, MulExtended, 2, 3, OpSMulExtended>;
700:
701: // Arithmetic builtin records:
702: defm : DemangledNativeBuiltin<"__spirv_FMod", OpenCL_std, Arithmetic, 2, 2, OpFMod>;
703:
704: // cl_intel_split_work_group_barrier
705: defm : DemangledNativeBuiltin<"intel_work_group_barrier_arrive", OpenCL_std, Barrier, 1, 2, OpControlBarrierArriveINTEL>;
706: defm : DemangledNativeBuiltin<"__spirv_ControlBarrierArriveINTEL", OpenCL_std, Barrier, 3, 3, OpControlBarrierArriveINTEL>;
707: defm : DemangledNativeBuiltin<"intel_work_group_barrier_wait", OpenCL_std, Barrier, 1, 2, OpControlBarrierWaitINTEL>;
708: defm : DemangledNativeBuiltin<"__spirv_ControlBarrierWaitINTEL", OpenCL_std, Barrier, 3, 3, OpControlBarrierWaitINTEL>;
709:
710: // Kernel enqueue builtin records:
711: defm : DemangledNativeBuiltin<"__enqueue_kernel_basic", OpenCL_std, Enqueue, 5, 5, OpEnqueueKernel>;
712: defm : DemangledNativeBuiltin<"__enqueue_kernel_basic_events", OpenCL_std, Enqueue, 8, 8, OpEnqueueKernel>;
713: defm : DemangledNativeBuiltin<"__enqueue_kernel_varargs", OpenCL_std, Enqueue, 7, 7, OpEnqueueKernel>;
714: defm : DemangledNativeBuiltin<"__enqueue_kernel_events_varargs", OpenCL_std, Enqueue, 10, 10, OpEnqueueKernel>;
715: defm : DemangledNativeBuiltin<"__spirv_EnqueueKernel", OpenCL_std, Enqueue, 10, 0, OpEnqueueKernel>;
716: defm : DemangledNativeBuiltin<"retain_event", OpenCL_std, Enqueue, 1, 1, OpRetainEvent>;
717: defm : DemangledNativeBuiltin<"__spirv_RetainEvent", OpenCL_std, Enqueue, 1, 1, OpRetainEvent>;
718: defm : DemangledNativeBuiltin<"release_event", OpenCL_std, Enqueue, 1, 1, OpReleaseEvent>;
719: defm : DemangledNativeBuiltin<"__spirv_ReleaseEvent", OpenCL_std, Enqueue, 1, 1, OpReleaseEvent>;
720: defm : DemangledNativeBuiltin<"create_user_event", OpenCL_std, Enqueue, 0, 0, OpCreateUserEvent>;
```
- EN: This range uses TableGen DSL to describe records such as backend logic; LLVM later expands these declarations into generated backend tables and helper code.
- CN: 这一段使用 TableGen DSL 描述 后端逻辑 等记录；LLVM 后续会把这些声明展开为生成的后端表和辅助代码。

### Lines 721-810
```tablegen
721: defm : DemangledNativeBuiltin<"__spirv_CreateUserEvent", OpenCL_std, Enqueue, 0, 0, OpCreateUserEvent>;
722: defm : DemangledNativeBuiltin<"is_valid_event", OpenCL_std, Enqueue, 1, 1, OpIsValidEvent>;
723: defm : DemangledNativeBuiltin<"__spirv_IsValidEvent", OpenCL_std, Enqueue, 1, 1, OpIsValidEvent>;
724: defm : DemangledNativeBuiltin<"set_user_event_status", OpenCL_std, Enqueue, 2, 2, OpSetUserEventStatus>;
725: defm : DemangledNativeBuiltin<"__spirv_SetUserEventStatus", OpenCL_std, Enqueue, 2, 2, OpSetUserEventStatus>;
726: defm : DemangledNativeBuiltin<"capture_event_profiling_info", OpenCL_std, Enqueue, 3, 3, OpCaptureEventProfilingInfo>;
727: defm : DemangledNativeBuiltin<"__spirv_CaptureEventProfilingInfo", OpenCL_std, Enqueue, 3, 3, OpCaptureEventProfilingInfo>;
728: defm : DemangledNativeBuiltin<"get_default_queue", OpenCL_std, Enqueue, 0, 0, OpGetDefaultQueue>;
729: defm : DemangledNativeBuiltin<"__spirv_GetDefaultQueue", OpenCL_std, Enqueue, 0, 0, OpGetDefaultQueue>;
730: defm : DemangledNativeBuiltin<"ndrange_1D", OpenCL_std, Enqueue, 1, 4, OpBuildNDRange>;
731: defm : DemangledNativeBuiltin<"ndrange_2D", OpenCL_std, Enqueue, 1, 4, OpBuildNDRange>;
732: defm : DemangledNativeBuiltin<"ndrange_3D", OpenCL_std, Enqueue, 1, 4, OpBuildNDRange>;
733:
734: // Spec constant builtin records:
735: defm : DemangledNativeBuiltin<"__spirv_SpecConstant", OpenCL_std, SpecConstant, 2, 2, OpSpecConstant>;
736: defm : DemangledNativeBuiltin<"__spirv_SpecConstant", GLSL_std_450,
737:                               SpecConstant, 2, 2, OpSpecConstant>;
738: defm : DemangledNativeBuiltin<"__spirv_SpecConstantComposite", OpenCL_std, SpecConstant, 1, 0, OpSpecConstantComposite>;
739:
740: // Async Copy and Prefetch builtin records:
741: defm : DemangledNativeBuiltin<"async_work_group_copy", OpenCL_std, AsyncCopy, 4, 4, OpGroupAsyncCopy>;
742: defm : DemangledNativeBuiltin<"async_work_group_strided_copy", OpenCL_std, AsyncCopy, 5, 5, OpGroupAsyncCopy>;
743: defm : DemangledNativeBuiltin<"__spirv_GroupAsyncCopy", OpenCL_std, AsyncCopy, 6, 6, OpGroupAsyncCopy>;
744: defm : DemangledNativeBuiltin<"wait_group_events", OpenCL_std, AsyncCopy, 2, 2, OpGroupWaitEvents>;
745: defm : DemangledNativeBuiltin<"__spirv_GroupWaitEvents", OpenCL_std, AsyncCopy, 3, 3, OpGroupWaitEvents>;
746:
747: // Load and store builtin records:
748: defm : DemangledNativeBuiltin<"__spirv_Load", OpenCL_std, LoadStore, 1, 3, OpLoad>;
749: defm : DemangledNativeBuiltin<"__spirv_Store", OpenCL_std, LoadStore, 2, 4, OpStore>;
750:
751: // Address Space Qualifier Functions/Pointers Conversion Instructions:
752: defm : DemangledNativeBuiltin<"to_global", OpenCL_std, CastToPtr, 1, 1, OpGenericCastToPtrExplicit>;
753: defm : DemangledNativeBuiltin<"to_local", OpenCL_std, CastToPtr, 1, 1, OpGenericCastToPtrExplicit>;
754: defm : DemangledNativeBuiltin<"to_private", OpenCL_std, CastToPtr, 1, 1, OpGenericCastToPtrExplicit>;
755: defm : DemangledNativeBuiltin<"__spirv_GenericCastToPtr_ToGlobal", OpenCL_std, CastToPtr, 2, 2, OpGenericCastToPtr>;
756: defm : DemangledNativeBuiltin<"__spirv_GenericCastToPtr_ToLocal", OpenCL_std, CastToPtr, 2, 2, OpGenericCastToPtr>;
757: defm : DemangledNativeBuiltin<"__spirv_GenericCastToPtr_ToPrivate", OpenCL_std, CastToPtr, 2, 2, OpGenericCastToPtr>;
758: defm : DemangledNativeBuiltin<"__spirv_GenericCastToPtrExplicit_ToGlobal", OpenCL_std, CastToPtr, 2, 2, OpGenericCastToPtrExplicit>;
759: defm : DemangledNativeBuiltin<"__spirv_GenericCastToPtrExplicit_ToLocal", OpenCL_std, CastToPtr, 2, 2, OpGenericCastToPtrExplicit>;
760: defm : DemangledNativeBuiltin<"__spirv_GenericCastToPtrExplicit_ToPrivate", OpenCL_std, CastToPtr, 2, 2, OpGenericCastToPtrExplicit>;
761:
762: // Cooperative Matrix builtin records:
763: defm : DemangledNativeBuiltin<"__spirv_CooperativeMatrixLoadKHR", OpenCL_std, CoopMatr, 2, 4, OpCooperativeMatrixLoadKHR>;
764: defm : DemangledNativeBuiltin<"__spirv_CooperativeMatrixStoreKHR", OpenCL_std, CoopMatr, 3, 5, OpCooperativeMatrixStoreKHR>;
765: defm : DemangledNativeBuiltin<"__spirv_CooperativeMatrixMulAddKHR", OpenCL_std, CoopMatr, 3, 4, OpCooperativeMatrixMulAddKHR>;
766: defm : DemangledNativeBuiltin<"__spirv_CooperativeMatrixLengthKHR", OpenCL_std, CoopMatr, 1, 1, OpCooperativeMatrixLengthKHR>;
767:
768: // Cooperative Matrix Intel builtin records:
769: defm : DemangledNativeBuiltin<"__spirv_CooperativeMatrixPrefetchINTEL", OpenCL_std, CoopMatr, 5, 7, OpCooperativeMatrixPrefetchINTEL>;
770: defm : DemangledNativeBuiltin<"__spirv_CooperativeMatrixLoadCheckedINTEL", OpenCL_std, CoopMatr, 6, 8, OpCooperativeMatrixLoadCheckedINTEL>;
771: defm : DemangledNativeBuiltin<"__spirv_CooperativeMatrixStoreCheckedINTEL", OpenCL_std, CoopMatr, 7, 9, OpCooperativeMatrixStoreCheckedINTEL>;
772: defm : DemangledNativeBuiltin<"__spirv_CooperativeMatrixConstructCheckedINTEL", OpenCL_std, CoopMatr, 5, 5, OpCooperativeMatrixConstructCheckedINTEL>;
773: defm : DemangledNativeBuiltin<"__spirv_CooperativeMatrixGetElementCoordINTEL", OpenCL_std, CoopMatr, 2, 2, OpCooperativeMatrixGetElementCoordINTEL>;
774:
775: // Arbitrary Precision Floating Point builtin records:
776: defm : DemangledNativeBuiltin<"__spirv_ArbitraryFloatGTALTERA", OpenCL_std, ArbitraryFloatingPoint, 4, 4, OpArbitraryFloatGTALTERA>;
777: defm : DemangledNativeBuiltin<"__spirv_ArbitraryFloatGEALTERA", OpenCL_std, ArbitraryFloatingPoint, 4, 4, OpArbitraryFloatGEALTERA>;
778: defm : DemangledNativeBuiltin<"__spirv_ArbitraryFloatLTALTERA", OpenCL_std, ArbitraryFloatingPoint, 4, 4, OpArbitraryFloatLTALTERA>;
779: defm : DemangledNativeBuiltin<"__spirv_ArbitraryFloatLEALTERA", OpenCL_std, ArbitraryFloatingPoint, 4, 4, OpArbitraryFloatLEALTERA>;
780: defm : DemangledNativeBuiltin<"__spirv_ArbitraryFloatEQALTERA", OpenCL_std, ArbitraryFloatingPoint, 4, 4, OpArbitraryFloatEQALTERA>;
781: defm : DemangledNativeBuiltin<"__spirv_ArbitraryFloatRecipALTERA", OpenCL_std, ArbitraryFloatingPoint, 6, 6, OpArbitraryFloatRecipALTERA>;
782: defm : DemangledNativeBuiltin<"__spirv_ArbitraryFloatCbrtALTERA", OpenCL_std, ArbitraryFloatingPoint, 6, 6, OpArbitraryFloatCbrtALTERA>;
783: defm : DemangledNativeBuiltin<"__spirv_ArbitraryFloatHypotALTERA", OpenCL_std, ArbitraryFloatingPoint, 8, 8, OpArbitraryFloatHypotALTERA>;
784: defm : DemangledNativeBuiltin<"__spirv_ArbitraryFloatSqrtALTERA", OpenCL_std, ArbitraryFloatingPoint, 6, 6, OpArbitraryFloatSqrtALTERA>;
785: defm : DemangledNativeBuiltin<"__spirv_ArbitraryFloatLogALTERA", OpenCL_std, ArbitraryFloatingPoint, 6, 6, OpArbitraryFloatLogALTERA>;
786: defm : DemangledNativeBuiltin<"__spirv_ArbitraryFloatLog2ALTERA", OpenCL_std, ArbitraryFloatingPoint, 6, 6, OpArbitraryFloatLog2ALTERA>;
787: defm : DemangledNativeBuiltin<"__spirv_ArbitraryFloatLog10ALTERA", OpenCL_std, ArbitraryFloatingPoint, 6, 6, OpArbitraryFloatLog10ALTERA>;
788: defm : DemangledNativeBuiltin<"__spirv_ArbitraryFloatLog1pALTERA", OpenCL_std, ArbitraryFloatingPoint, 6, 6, OpArbitraryFloatLog1pALTERA>;
789: defm : DemangledNativeBuiltin<"__spirv_ArbitraryFloatExpALTERA", OpenCL_std, ArbitraryFloatingPoint, 6, 6, OpArbitraryFloatExpALTERA>;
790: defm : DemangledNativeBuiltin<"__spirv_ArbitraryFloatExp2ALTERA", OpenCL_std, ArbitraryFloatingPoint, 6, 6, OpArbitraryFloatExp2ALTERA>;
791: defm : DemangledNativeBuiltin<"__spirv_ArbitraryFloatExp10ALTERA", OpenCL_std, ArbitraryFloatingPoint, 6, 6, OpArbitraryFloatExp10ALTERA>;
792: defm : DemangledNativeBuiltin<"__spirv_ArbitraryFloatExpm1ALTERA", OpenCL_std, ArbitraryFloatingPoint, 6, 6, OpArbitraryFloatExpm1ALTERA>;
793: defm : DemangledNativeBuiltin<"__spirv_ArbitraryFloatSinALTERA", OpenCL_std, ArbitraryFloatingPoint, 6, 6, OpArbitraryFloatSinALTERA>;
794: defm : DemangledNativeBuiltin<"__spirv_ArbitraryFloatCosALTERA", OpenCL_std, ArbitraryFloatingPoint, 6, 6, OpArbitraryFloatCosALTERA>;
795: defm : DemangledNativeBuiltin<"__spirv_ArbitraryFloatSinCosALTERA", OpenCL_std, ArbitraryFloatingPoint, 6, 6, OpArbitraryFloatSinCosALTERA>;
796: defm : DemangledNativeBuiltin<"__spirv_ArbitraryFloatSinPiALTERA", OpenCL_std, ArbitraryFloatingPoint, 6, 6, OpArbitraryFloatSinPiALTERA>;
797: defm : DemangledNativeBuiltin<"__spirv_ArbitraryFloatCosPiALTERA", OpenCL_std, ArbitraryFloatingPoint, 6, 6, OpArbitraryFloatCosPiALTERA>;
798: defm : DemangledNativeBuiltin<"__spirv_ArbitraryFloatSinCosPiALTERA", OpenCL_std, ArbitraryFloatingPoint, 6, 6, OpArbitraryFloatSinCosPiALTERA>;
799: defm : DemangledNativeBuiltin<"__spirv_ArbitraryFloatAddALTERA", OpenCL_std, ArbitraryFloatingPoint, 8, 8, OpArbitraryFloatAddALTERA>;
800: defm : DemangledNativeBuiltin<"__spirv_ArbitraryFloatSubALTERA", OpenCL_std, ArbitraryFloatingPoint, 8, 8, OpArbitraryFloatSubALTERA>;
801: defm : DemangledNativeBuiltin<"__spirv_ArbitraryFloatMulALTERA", OpenCL_std, ArbitraryFloatingPoint, 8, 8, OpArbitraryFloatMulALTERA>;
802: defm : DemangledNativeBuiltin<"__spirv_ArbitraryFloatDivALTERA", OpenCL_std, ArbitraryFloatingPoint, 8, 8, OpArbitraryFloatDivALTERA>;
803: defm : DemangledNativeBuiltin<"__spirv_ArbitraryFloatRSqrtALTERA", OpenCL_std, ArbitraryFloatingPoint, 6, 6, OpArbitraryFloatRSqrtALTERA>;
804: defm : DemangledNativeBuiltin<"__spirv_ArbitraryFloatASinALTERA", OpenCL_std, ArbitraryFloatingPoint, 6, 6, OpArbitraryFloatASinALTERA>;
805: defm : DemangledNativeBuiltin<"__spirv_ArbitraryFloatASinPiALTERA", OpenCL_std, ArbitraryFloatingPoint, 6, 6, OpArbitraryFloatASinPiALTERA>;
806: defm : DemangledNativeBuiltin<"__spirv_ArbitraryFloatACosALTERA", OpenCL_std, ArbitraryFloatingPoint, 6, 6, OpArbitraryFloatACosALTERA>;
807: defm : DemangledNativeBuiltin<"__spirv_ArbitraryFloatACosPiALTERA", OpenCL_std, ArbitraryFloatingPoint, 6, 6, OpArbitraryFloatACosPiALTERA>;
808: defm : DemangledNativeBuiltin<"__spirv_ArbitraryFloatATanALTERA", OpenCL_std, ArbitraryFloatingPoint, 6, 6, OpArbitraryFloatATanALTERA>;
809: defm : DemangledNativeBuiltin<"__spirv_ArbitraryFloatATanPiALTERA", OpenCL_std, ArbitraryFloatingPoint, 6, 6, OpArbitraryFloatATanPiALTERA>;
810: defm : DemangledNativeBuiltin<"__spirv_ArbitraryFloatATan2ALTERA", OpenCL_std, ArbitraryFloatingPoint, 8, 8, OpArbitraryFloatATan2ALTERA>;
```
- EN: This range uses TableGen DSL to describe records such as backend logic; LLVM later expands these declarations into generated backend tables and helper code.
- CN: 这一段使用 TableGen DSL 描述 后端逻辑 等记录；LLVM 后续会把这些声明展开为生成的后端表和辅助代码。

### Lines 811-900
```tablegen
811: defm : DemangledNativeBuiltin<"__spirv_ArbitraryFloatPowALTERA", OpenCL_std, ArbitraryFloatingPoint, 8, 8, OpArbitraryFloatPowALTERA>;
812: defm : DemangledNativeBuiltin<"__spirv_ArbitraryFloatPowRALTERA", OpenCL_std, ArbitraryFloatingPoint, 8, 8, OpArbitraryFloatPowRALTERA>;
813: defm : DemangledNativeBuiltin<"__spirv_ArbitraryFloatPowNALTERA", OpenCL_std, ArbitraryFloatingPoint, 8, 8, OpArbitraryFloatPowNALTERA>;
814: defm : DemangledNativeBuiltin<"__spirv_ArbitraryFloatCastALTERA", OpenCL_std, ArbitraryFloatingPoint, 6, 6, OpArbitraryFloatCastALTERA>;
815: defm : DemangledNativeBuiltin<"__spirv_ArbitraryFloatCastFromIntALTERA", OpenCL_std, ArbitraryFloatingPoint, 6, 6, OpArbitraryFloatCastFromIntALTERA>;
816: defm : DemangledNativeBuiltin<"__spirv_ArbitraryFloatCastToIntALTERA", OpenCL_std, ArbitraryFloatingPoint, 6, 6, OpArbitraryFloatCastToIntALTERA>;
817:
818: // SPV_INTEL_bindless_images builtin records:
819: defm : DemangledNativeBuiltin<"__spirv_ConvertHandleToImageINTEL", OpenCL_std, BindlessINTEL, 1, 1, OpConvertHandleToImageINTEL>;
820: defm : DemangledNativeBuiltin<"__spirv_ConvertHandleToSamplerINTEL", OpenCL_std, BindlessINTEL, 1, 1, OpConvertHandleToSamplerINTEL>;
821: defm : DemangledNativeBuiltin<"__spirv_ConvertHandleToSampledImageINTEL", OpenCL_std, BindlessINTEL, 1, 1, OpConvertHandleToSampledImageINTEL>;
822:
823: // SPV_INTEL_ternary_bitwise_function builtin records:
824: defm : DemangledNativeBuiltin<"__spirv_BitwiseFunctionINTEL", OpenCL_std, TernaryBitwiseINTEL, 4, 4, OpBitwiseFunctionINTEL>;
825:
826: // SPV_INTEL_2d_block_io builtin records
827: defm : DemangledNativeBuiltin<"__spirv_Subgroup2DBlockLoadINTEL", OpenCL_std, Block2DLoadStore, 10, 10, OpSubgroup2DBlockLoadINTEL>;
828: defm : DemangledNativeBuiltin<"__spirv_Subgroup2DBlockLoadTransposeINTEL", OpenCL_std, Block2DLoadStore, 10, 10, OpSubgroup2DBlockLoadTransposeINTEL>;
829: defm : DemangledNativeBuiltin<"__spirv_Subgroup2DBlockLoadTransformINTEL", OpenCL_std, Block2DLoadStore, 10, 10, OpSubgroup2DBlockLoadTransformINTEL>;
830: defm : DemangledNativeBuiltin<"__spirv_Subgroup2DBlockPrefetchINTEL", OpenCL_std, Block2DLoadStore, 9, 9, OpSubgroup2DBlockPrefetchINTEL>;
831: defm : DemangledNativeBuiltin<"__spirv_Subgroup2DBlockStoreINTEL", OpenCL_std, Block2DLoadStore, 10, 10, OpSubgroup2DBlockStoreINTEL>;
832:
833: // SPV_INTEL_predicated_io builtin records
834: defm : DemangledNativeBuiltin<"__spirv_PredicatedLoadINTEL", OpenCL_std, PredicatedLoadStore, 3, 4, OpPredicatedLoadINTEL>;
835: defm : DemangledNativeBuiltin<"__spirv_PredicatedStoreINTEL", OpenCL_std, PredicatedLoadStore, 3, 4, OpPredicatedStoreINTEL>;
836:
837: //===----------------------------------------------------------------------===//
838: // Class defining a work/sub group builtin that should be translated into a
839: // SPIR-V instruction using the defined properties.
840: //
841: // name is the demangled name of the given builtin.
842: // opcode specifies the SPIR-V operation code of the generated instruction.
843: //===----------------------------------------------------------------------===//
844: class GroupBuiltin<string name, Op operation> {
845:   string Name = name;
846:   Op Opcode = operation;
847:   bits<32> GroupOperation = !cond(!not(!eq(!find(name, "group_reduce"), -1)) : Reduce.Value,
848:                                   !not(!eq(!find(name, "group_scan_inclusive"), -1)) : InclusiveScan.Value,
849:                                   !not(!eq(!find(name, "group_scan_exclusive"), -1)) : ExclusiveScan.Value,
850:                                   !not(!eq(!find(name, "group_ballot_bit_count"), -1)) : Reduce.Value,
851:                                   !not(!eq(!find(name, "group_ballot_inclusive_scan"), -1)) : InclusiveScan.Value,
852:                                   !not(!eq(!find(name, "group_ballot_exclusive_scan"), -1)) : ExclusiveScan.Value,
853:                                   !not(!eq(!find(name, "group_non_uniform_reduce"), -1)) : Reduce.Value,
854:                                   !not(!eq(!find(name, "group_non_uniform_scan_inclusive"), -1)) : InclusiveScan.Value,
855:                                   !not(!eq(!find(name, "group_non_uniform_scan_exclusive"), -1)) : ExclusiveScan.Value,
856:                                   !not(!eq(!find(name, "group_non_uniform_reduce_logical"), -1)) : Reduce.Value,
857:                                   !not(!eq(!find(name, "group_non_uniform_scan_inclusive_logical"), -1)) : InclusiveScan.Value,
858:                                   !not(!eq(!find(name, "group_non_uniform_scan_exclusive_logical"), -1)) : ExclusiveScan.Value,
859:                                   !not(!eq(!find(name, "group_clustered_reduce"), -1)) : ClusteredReduce.Value,
860:                                   !not(!eq(!find(name, "group_clustered_reduce_logical"), -1)) : ClusteredReduce.Value,
861:                                   true : 0);
862:   bit IsElect = !eq(operation, OpGroupNonUniformElect);
863:   bit IsAllOrAny = !or(!eq(operation, OpGroupAll),
864:                        !eq(operation, OpGroupAny),
865:                        !eq(operation, OpGroupNonUniformAll),
866:                        !eq(operation, OpGroupNonUniformAny));
867:   bit IsAllEqual = !eq(operation, OpGroupNonUniformAllEqual);
868:   bit IsBallot = !eq(operation, OpGroupNonUniformBallot);
869:   bit IsInverseBallot = !eq(operation, OpGroupNonUniformInverseBallot);
870:   bit IsBallotBitExtract = !eq(operation, OpGroupNonUniformBallotBitExtract);
871:   bit IsBallotFindBit = !or(!eq(operation, OpGroupNonUniformBallotFindLSB),
872:                             !eq(operation, OpGroupNonUniformBallotFindMSB));
873:   bit IsLogical = !or(!eq(operation, OpGroupNonUniformLogicalAnd),
874:                       !eq(operation, OpGroupNonUniformLogicalOr),
875:                       !eq(operation, OpGroupNonUniformLogicalXor),
876:                       !eq(operation, OpGroupLogicalAndKHR),
877:                       !eq(operation, OpGroupLogicalOrKHR),
878:                       !eq(operation, OpGroupLogicalXorKHR));
879:   bit NoGroupOperation = !or(IsElect, IsAllOrAny, IsAllEqual,
880:                              IsBallot, IsInverseBallot,
881:                              IsBallotBitExtract, IsBallotFindBit,
882:                              !eq(operation, OpSubgroupMatrixMultiplyAccumulateINTEL),
883:                              !eq(operation, OpGroupNonUniformShuffle),
884:                              !eq(operation, OpGroupNonUniformShuffleXor),
885:                              !eq(operation, OpGroupNonUniformShuffleUp),
886:                              !eq(operation, OpGroupNonUniformShuffleDown),
887:                              !eq(operation, OpGroupBroadcast),
888:                              !eq(operation, OpGroupNonUniformBroadcast),
889:                              !eq(operation, OpGroupNonUniformBroadcastFirst),
890:                              !eq(operation, OpGroupNonUniformRotateKHR));
891:   bit HasBoolArg = !or(!and(IsAllOrAny, !eq(IsAllEqual, false)), IsBallot, IsLogical);
892: }
893:
894: // Table gathering all the work/sub group builtins.
895: def GroupBuiltins : GenericTable {
896:   let FilterClass = "GroupBuiltin";
897:   let Fields = ["Name", "Opcode", "GroupOperation", "IsElect", "IsAllOrAny",
898:                 "IsAllEqual", "IsBallot", "IsInverseBallot", "IsBallotBitExtract",
899:                 "IsBallotFindBit", "IsLogical", "NoGroupOperation", "HasBoolArg"];
900: }
```
- EN: This range uses TableGen DSL to describe records such as GroupBuiltin, GroupBuiltins; LLVM later expands these declarations into generated backend tables and helper code.
- CN: 这一段使用 TableGen DSL 描述 GroupBuiltin、GroupBuiltins 等记录；LLVM 后续会把这些声明展开为生成的后端表和辅助代码。

### Lines 901-990
```tablegen
901:
902: // Function to lookup group builtins by their name and set.
903: def lookupGroupBuiltin : SearchIndex {
904:   let Table = GroupBuiltins;
905:   let Key = ["Name"];
906: }
907:
908: // Multiclass used to define at the same time both incoming builtin records
909: // and corresponding work/sub group builtin records.
910: defvar OnlyWork = 0; defvar OnlySub = 1; defvar WorkOrSub = 2;
911: multiclass DemangledGroupBuiltin<string name, int level /* OnlyWork/OnlySub/... */, Op operation> {
912:   assert !and(!ge(level, 0), !le(level, 2)), "group level is invalid: " # level;
913:
914:   if !or(!eq(level, OnlyWork), !eq(level, WorkOrSub)) then {
915:     def : DemangledBuiltin<!strconcat("work_", name), OpenCL_std, Group, 0, 4>;
916:     def : GroupBuiltin<!strconcat("work_", name), operation>;
917:   }
918:
919:   if !or(!eq(level, OnlySub), !eq(level, WorkOrSub)) then {
920:     def : DemangledBuiltin<!strconcat("sub_", name), OpenCL_std, Group, 0, 4>;
921:     def : GroupBuiltin<!strconcat("sub_", name), operation>;
922:   }
923: }
924:
925: multiclass DemangledGroupBuiltinWrapper<string name, bits<8> minNumArgs, bits<8> maxNumArgs, Op operation> {
926:     def : DemangledBuiltin<name, OpenCL_std, Group, minNumArgs, maxNumArgs>;
927:     def : GroupBuiltin<name, operation>;
928: }
929:
930: defm : DemangledGroupBuiltin<"group_all", WorkOrSub, OpGroupAll>;
931: defm : DemangledGroupBuiltinWrapper<"__spirv_GroupAll", 2, 2, OpGroupAll>;
932: defm : DemangledGroupBuiltin<"group_any", WorkOrSub, OpGroupAny>;
933: defm : DemangledGroupBuiltinWrapper<"__spirv_GroupAny", 2, 2, OpGroupAny>;
934: defm : DemangledGroupBuiltin<"group_broadcast", WorkOrSub, OpGroupBroadcast>;
935: defm : DemangledGroupBuiltinWrapper<"__spirv_GroupBroadcast", 3, 3, OpGroupBroadcast>;
936: defm : DemangledGroupBuiltin<"group_non_uniform_broadcast", OnlySub, OpGroupNonUniformBroadcast>;
937: defm : DemangledGroupBuiltinWrapper<"__spirv_GroupNonUniformBroadcast", 3, 3, OpGroupNonUniformBroadcast>;
938: defm : DemangledGroupBuiltin<"group_broadcast_first", OnlySub, OpGroupNonUniformBroadcastFirst>;
939: defm : DemangledGroupBuiltinWrapper<"__spirv_GroupNonUniformBroadcastFirst", 2, 2, OpGroupNonUniformBroadcastFirst>;
940:
941: // cl_khr_subgroup_non_uniform_vote
942: defm : DemangledGroupBuiltin<"group_elect", OnlySub, OpGroupNonUniformElect>;
943: defm : DemangledGroupBuiltinWrapper<"__spirv_GroupNonUniformElect", 1, 1, OpGroupNonUniformElect>;
944: defm : DemangledGroupBuiltin<"group_non_uniform_all", OnlySub, OpGroupNonUniformAll>;
945: defm : DemangledGroupBuiltinWrapper<"__spirv_GroupNonUniformAll", 2, 2, OpGroupNonUniformAll>;
946: defm : DemangledGroupBuiltin<"group_non_uniform_any", OnlySub, OpGroupNonUniformAny>;
947: defm : DemangledGroupBuiltinWrapper<"__spirv_GroupNonUniformAny", 2, 2, OpGroupNonUniformAny>;
948: defm : DemangledGroupBuiltin<"group_non_uniform_all_equal", OnlySub, OpGroupNonUniformAllEqual>;
949: defm : DemangledGroupBuiltinWrapper<"__spirv_GroupNonUniformAllEqual", 2, 2, OpGroupNonUniformAllEqual>;
950:
951: // cl_khr_subgroup_ballot
952: defm : DemangledGroupBuiltin<"group_ballot", OnlySub, OpGroupNonUniformBallot>;
953: defm : DemangledGroupBuiltinWrapper<"__spirv_GroupNonUniformBallot", 2, 2, OpGroupNonUniformBallot>;
954: defm : DemangledGroupBuiltin<"group_inverse_ballot", OnlySub, OpGroupNonUniformInverseBallot>;
955: defm : DemangledGroupBuiltinWrapper<"__spirv_GroupNonUniformInverseBallot", 2, 2, OpGroupNonUniformInverseBallot>;
956: defm : DemangledGroupBuiltin<"group_ballot_bit_extract", OnlySub, OpGroupNonUniformBallotBitExtract>;
957: defm : DemangledGroupBuiltinWrapper<"__spirv_GroupNonUniformBallotBitExtract", 3, 3, OpGroupNonUniformBallotBitExtract>;
958: defm : DemangledGroupBuiltin<"group_ballot_bit_count", OnlySub, OpGroupNonUniformBallotBitCount>;
959: defm : DemangledGroupBuiltin<"group_ballot_inclusive_scan", OnlySub, OpGroupNonUniformBallotBitCount>;
960: defm : DemangledGroupBuiltin<"group_ballot_exclusive_scan", OnlySub, OpGroupNonUniformBallotBitCount>;
961: defm : DemangledGroupBuiltinWrapper<"__spirv_GroupNonUniformBallotBitCount", 3, 3, OpGroupNonUniformBallotBitCount>;
962: defm : DemangledGroupBuiltin<"group_ballot_find_lsb", OnlySub, OpGroupNonUniformBallotFindLSB>;
963: defm : DemangledGroupBuiltinWrapper<"__spirv_GroupNonUniformBallotFindLSB", 2, 2, OpGroupNonUniformBallotFindLSB>;
964: defm : DemangledGroupBuiltin<"group_ballot_find_msb", OnlySub, OpGroupNonUniformBallotFindMSB>;
965: defm : DemangledGroupBuiltinWrapper<"__spirv_GroupNonUniformBallotFindMSB", 2, 2, OpGroupNonUniformBallotFindMSB>;
966:
967: // SPV_INTEL_subgroup_matrix_multiply_accumulate
968: defm : DemangledGroupBuiltinWrapper<"__spirv_SubgroupMatrixMultiplyAccumulateINTEL", 4, 5, OpSubgroupMatrixMultiplyAccumulateINTEL>;
969:
970: // cl_khr_subgroup_shuffle
971: defm : DemangledGroupBuiltin<"group_shuffle", OnlySub, OpGroupNonUniformShuffle>;
972: defm : DemangledGroupBuiltinWrapper<"__spirv_GroupNonUniformShuffle", 3, 3, OpGroupNonUniformShuffle>;
973: defm : DemangledGroupBuiltin<"group_shuffle_xor", OnlySub, OpGroupNonUniformShuffleXor>;
974: defm : DemangledGroupBuiltinWrapper<"__spirv_GroupNonUniformShuffleXor", 3, 3, OpGroupNonUniformShuffleXor>;
975:
976: // cl_khr_subgroup_shuffle_relative
977: defm : DemangledGroupBuiltin<"group_shuffle_up", OnlySub, OpGroupNonUniformShuffleUp>;
978: defm : DemangledGroupBuiltinWrapper<"__spirv_GroupNonUniformShuffleUp", 3, 3, OpGroupNonUniformShuffleUp>;
979: defm : DemangledGroupBuiltin<"group_shuffle_down", OnlySub, OpGroupNonUniformShuffleDown>;
980: defm : DemangledGroupBuiltinWrapper<"__spirv_GroupNonUniformShuffleDown", 3, 3, OpGroupNonUniformShuffleDown>;
981:
982: defm : DemangledGroupBuiltin<"group_iadd", WorkOrSub, OpGroupIAdd>;
983: defm : DemangledGroupBuiltin<"group_reduce_adds", WorkOrSub, OpGroupIAdd>;
984: defm : DemangledGroupBuiltin<"group_scan_exclusive_adds", WorkOrSub, OpGroupIAdd>;
985: defm : DemangledGroupBuiltin<"group_scan_inclusive_adds", WorkOrSub, OpGroupIAdd>;
986: defm : DemangledGroupBuiltin<"group_reduce_addu", WorkOrSub, OpGroupIAdd>;
987: defm : DemangledGroupBuiltin<"group_scan_exclusive_addu", WorkOrSub, OpGroupIAdd>;
988: defm : DemangledGroupBuiltin<"group_scan_inclusive_addu", WorkOrSub, OpGroupIAdd>;
989: defm : DemangledGroupBuiltinWrapper<"__spirv_GroupIAdd", 3, 3, OpGroupIAdd>;
990:
```
- EN: This range uses TableGen DSL to describe records such as lookupGroupBuiltin, DemangledGroupBuiltin, DemangledGroupBuiltinWrapper; LLVM later expands these declarations into generated backend tables and helper code.
- CN: 这一段使用 TableGen DSL 描述 lookupGroupBuiltin、DemangledGroupBuiltin、DemangledGroupBuiltinWrapper 等记录；LLVM 后续会把这些声明展开为生成的后端表和辅助代码。

### Lines 991-1080
```tablegen
 991: defm : DemangledGroupBuiltin<"group_fadd", WorkOrSub, OpGroupFAdd>;
 992: defm : DemangledGroupBuiltin<"group_reduce_addf", WorkOrSub, OpGroupFAdd>;
 993: defm : DemangledGroupBuiltin<"group_scan_exclusive_addf", WorkOrSub, OpGroupFAdd>;
 994: defm : DemangledGroupBuiltin<"group_scan_inclusive_addf", WorkOrSub, OpGroupFAdd>;
 995: defm : DemangledGroupBuiltinWrapper<"__spirv_GroupFAdd", 3, 3, OpGroupFAdd>;
 996:
 997: defm : DemangledGroupBuiltin<"group_fmin", WorkOrSub, OpGroupFMin>;
 998: defm : DemangledGroupBuiltin<"group_reduce_minf", WorkOrSub, OpGroupFMin>;
 999: defm : DemangledGroupBuiltin<"group_scan_exclusive_minf", WorkOrSub, OpGroupFMin>;
1000: defm : DemangledGroupBuiltin<"group_scan_inclusive_minf", WorkOrSub, OpGroupFMin>;
1001: defm : DemangledGroupBuiltinWrapper<"__spirv_GroupFMin", 3, 3, OpGroupFMin>;
1002:
1003: defm : DemangledGroupBuiltin<"group_umin", WorkOrSub, OpGroupUMin>;
1004: defm : DemangledGroupBuiltin<"group_reduce_minu", WorkOrSub, OpGroupUMin>;
1005: defm : DemangledGroupBuiltin<"group_scan_exclusive_minu", WorkOrSub, OpGroupUMin>;
1006: defm : DemangledGroupBuiltin<"group_scan_inclusive_minu", WorkOrSub, OpGroupUMin>;
1007: defm : DemangledGroupBuiltinWrapper<"__spirv_GroupUMin", 3, 3, OpGroupUMin>;
1008:
1009: defm : DemangledGroupBuiltin<"group_smin", WorkOrSub, OpGroupSMin>;
1010: defm : DemangledGroupBuiltin<"group_reduce_mins", WorkOrSub, OpGroupSMin>;
1011: defm : DemangledGroupBuiltin<"group_scan_exclusive_mins", WorkOrSub, OpGroupSMin>;
1012: defm : DemangledGroupBuiltin<"group_scan_inclusive_mins", WorkOrSub, OpGroupSMin>;
1013: defm : DemangledGroupBuiltinWrapper<"__spirv_GroupSMin", 3, 3, OpGroupSMin>;
1014:
1015: defm : DemangledGroupBuiltin<"group_fmax", WorkOrSub, OpGroupFMax>;
1016: defm : DemangledGroupBuiltin<"group_reduce_maxf", WorkOrSub, OpGroupFMax>;
1017: defm : DemangledGroupBuiltin<"group_scan_exclusive_maxf", WorkOrSub, OpGroupFMax>;
1018: defm : DemangledGroupBuiltin<"group_scan_inclusive_maxf", WorkOrSub, OpGroupFMax>;
1019: defm : DemangledGroupBuiltinWrapper<"__spirv_GroupFMax", 3, 3, OpGroupFMax>;
1020:
1021: defm : DemangledGroupBuiltin<"group_umax", WorkOrSub, OpGroupUMax>;
1022: defm : DemangledGroupBuiltin<"group_reduce_maxu", WorkOrSub, OpGroupUMax>;
1023: defm : DemangledGroupBuiltin<"group_scan_exclusive_maxu", WorkOrSub, OpGroupUMax>;
1024: defm : DemangledGroupBuiltin<"group_scan_inclusive_maxu", WorkOrSub, OpGroupUMax>;
1025: defm : DemangledGroupBuiltinWrapper<"__spirv_GroupUMax", 3, 3, OpGroupUMax>;
1026:
1027: defm : DemangledGroupBuiltin<"group_smax", WorkOrSub, OpGroupSMax>;
1028: defm : DemangledGroupBuiltin<"group_reduce_maxs", WorkOrSub, OpGroupSMax>;
1029: defm : DemangledGroupBuiltin<"group_scan_exclusive_maxs", WorkOrSub, OpGroupSMax>;
1030: defm : DemangledGroupBuiltin<"group_scan_inclusive_maxs", WorkOrSub, OpGroupSMax>;
1031: defm : DemangledGroupBuiltinWrapper<"__spirv_GroupSMax", 3, 3, OpGroupSMax>;
1032:
1033: // cl_khr_subgroup_non_uniform_arithmetic
1034: defm : DemangledGroupBuiltin<"group_non_uniform_iadd", WorkOrSub, OpGroupNonUniformIAdd>;
1035: defm : DemangledGroupBuiltin<"group_non_uniform_reduce_addu", WorkOrSub, OpGroupNonUniformIAdd>;
1036: defm : DemangledGroupBuiltin<"group_non_uniform_reduce_adds", WorkOrSub, OpGroupNonUniformIAdd>;
1037: defm : DemangledGroupBuiltin<"group_non_uniform_scan_inclusive_addu", WorkOrSub, OpGroupNonUniformIAdd>;
1038: defm : DemangledGroupBuiltin<"group_non_uniform_scan_inclusive_adds", WorkOrSub, OpGroupNonUniformIAdd>;
1039: defm : DemangledGroupBuiltin<"group_non_uniform_scan_exclusive_addu", WorkOrSub, OpGroupNonUniformIAdd>;
1040: defm : DemangledGroupBuiltin<"group_non_uniform_scan_exclusive_adds", WorkOrSub, OpGroupNonUniformIAdd>;
1041: defm : DemangledGroupBuiltin<"group_clustered_reduce_addu", WorkOrSub, OpGroupNonUniformIAdd>;
1042: defm : DemangledGroupBuiltin<"group_clustered_reduce_adds", WorkOrSub, OpGroupNonUniformIAdd>;
1043: defm : DemangledGroupBuiltinWrapper<"__spirv_GroupNonUniformIAdd", 3, 4, OpGroupNonUniformIAdd>;
1044:
1045: defm : DemangledGroupBuiltin<"group_non_uniform_fadd", WorkOrSub, OpGroupNonUniformFAdd>;
1046: defm : DemangledGroupBuiltin<"group_non_uniform_reduce_addf", WorkOrSub, OpGroupNonUniformFAdd>;
1047: defm : DemangledGroupBuiltin<"group_non_uniform_reduce_addh", WorkOrSub, OpGroupNonUniformFAdd>;
1048: defm : DemangledGroupBuiltin<"group_non_uniform_reduce_addd", WorkOrSub, OpGroupNonUniformFAdd>;
1049: defm : DemangledGroupBuiltin<"group_non_uniform_scan_inclusive_addf", WorkOrSub, OpGroupNonUniformFAdd>;
1050: defm : DemangledGroupBuiltin<"group_non_uniform_scan_inclusive_addh", WorkOrSub, OpGroupNonUniformFAdd>;
1051: defm : DemangledGroupBuiltin<"group_non_uniform_scan_inclusive_addd", WorkOrSub, OpGroupNonUniformFAdd>;
1052: defm : DemangledGroupBuiltin<"group_non_uniform_scan_exclusive_addf", WorkOrSub, OpGroupNonUniformFAdd>;
1053: defm : DemangledGroupBuiltin<"group_non_uniform_scan_exclusive_addh", WorkOrSub, OpGroupNonUniformFAdd>;
1054: defm : DemangledGroupBuiltin<"group_non_uniform_scan_exclusive_addd", WorkOrSub, OpGroupNonUniformFAdd>;
1055: defm : DemangledGroupBuiltin<"group_clustered_reduce_addf", WorkOrSub, OpGroupNonUniformFAdd>;
1056: defm : DemangledGroupBuiltin<"group_clustered_reduce_addh", WorkOrSub, OpGroupNonUniformFAdd>;
1057: defm : DemangledGroupBuiltin<"group_clustered_reduce_addd", WorkOrSub, OpGroupNonUniformFAdd>;
1058: defm : DemangledGroupBuiltinWrapper<"__spirv_GroupNonUniformFAdd", 3, 4, OpGroupNonUniformFAdd>;
1059:
1060: defm : DemangledGroupBuiltin<"group_non_uniform_imul", WorkOrSub, OpGroupNonUniformIMul>;
1061: defm : DemangledGroupBuiltin<"group_non_uniform_reduce_mulu", WorkOrSub, OpGroupNonUniformIMul>;
1062: defm : DemangledGroupBuiltin<"group_non_uniform_reduce_muls", WorkOrSub, OpGroupNonUniformIMul>;
1063: defm : DemangledGroupBuiltin<"group_non_uniform_scan_inclusive_mulu", WorkOrSub, OpGroupNonUniformIMul>;
1064: defm : DemangledGroupBuiltin<"group_non_uniform_scan_inclusive_muls", WorkOrSub, OpGroupNonUniformIMul>;
1065: defm : DemangledGroupBuiltin<"group_non_uniform_scan_exclusive_mulu", WorkOrSub, OpGroupNonUniformIMul>;
1066: defm : DemangledGroupBuiltin<"group_non_uniform_scan_exclusive_muls", WorkOrSub, OpGroupNonUniformIMul>;
1067: defm : DemangledGroupBuiltin<"group_clustered_reduce_mulu", WorkOrSub, OpGroupNonUniformIMul>;
1068: defm : DemangledGroupBuiltin<"group_clustered_reduce_muls", WorkOrSub, OpGroupNonUniformIMul>;
1069: defm : DemangledGroupBuiltinWrapper<"__spirv_GroupNonUniformIMul", 3, 4, OpGroupNonUniformIMul>;
1070:
1071: defm : DemangledGroupBuiltin<"group_non_uniform_fmul", WorkOrSub, OpGroupNonUniformFMul>;
1072: defm : DemangledGroupBuiltin<"group_non_uniform_reduce_mulf", WorkOrSub, OpGroupNonUniformFMul>;
1073: defm : DemangledGroupBuiltin<"group_non_uniform_reduce_mulh", WorkOrSub, OpGroupNonUniformFMul>;
1074: defm : DemangledGroupBuiltin<"group_non_uniform_reduce_muld", WorkOrSub, OpGroupNonUniformFMul>;
1075: defm : DemangledGroupBuiltin<"group_non_uniform_scan_inclusive_mulf", WorkOrSub, OpGroupNonUniformFMul>;
1076: defm : DemangledGroupBuiltin<"group_non_uniform_scan_inclusive_mulh", WorkOrSub, OpGroupNonUniformFMul>;
1077: defm : DemangledGroupBuiltin<"group_non_uniform_scan_inclusive_muld", WorkOrSub, OpGroupNonUniformFMul>;
1078: defm : DemangledGroupBuiltin<"group_non_uniform_scan_exclusive_mulf", WorkOrSub, OpGroupNonUniformFMul>;
1079: defm : DemangledGroupBuiltin<"group_non_uniform_scan_exclusive_mulh", WorkOrSub, OpGroupNonUniformFMul>;
1080: defm : DemangledGroupBuiltin<"group_non_uniform_scan_exclusive_muld", WorkOrSub, OpGroupNonUniformFMul>;
```
- EN: This range uses TableGen DSL to describe records such as backend logic; LLVM later expands these declarations into generated backend tables and helper code.
- CN: 这一段使用 TableGen DSL 描述 后端逻辑 等记录；LLVM 后续会把这些声明展开为生成的后端表和辅助代码。

### Lines 1081-1170
```tablegen
1081: defm : DemangledGroupBuiltin<"group_clustered_reduce_mulf", WorkOrSub, OpGroupNonUniformFMul>;
1082: defm : DemangledGroupBuiltin<"group_clustered_reduce_mulh", WorkOrSub, OpGroupNonUniformFMul>;
1083: defm : DemangledGroupBuiltin<"group_clustered_reduce_muld", WorkOrSub, OpGroupNonUniformFMul>;
1084: defm : DemangledGroupBuiltinWrapper<"__spirv_GroupNonUniformFMul", 3, 4, OpGroupNonUniformFMul>;
1085:
1086: defm : DemangledGroupBuiltin<"group_non_uniform_smin", WorkOrSub, OpGroupNonUniformSMin>;
1087: defm : DemangledGroupBuiltin<"group_non_uniform_reduce_mins", WorkOrSub, OpGroupNonUniformSMin>;
1088: defm : DemangledGroupBuiltin<"group_non_uniform_scan_inclusive_mins", WorkOrSub, OpGroupNonUniformSMin>;
1089: defm : DemangledGroupBuiltin<"group_non_uniform_scan_exclusive_mins", WorkOrSub, OpGroupNonUniformSMin>;
1090: defm : DemangledGroupBuiltin<"group_clustered_reduce_mins", WorkOrSub, OpGroupNonUniformSMin>;
1091: defm : DemangledGroupBuiltinWrapper<"__spirv_GroupNonUniformSMin", 3, 4, OpGroupNonUniformSMin>;
1092:
1093: defm : DemangledGroupBuiltin<"group_non_uniform_umin", WorkOrSub, OpGroupNonUniformUMin>;
1094: defm : DemangledGroupBuiltin<"group_non_uniform_reduce_minu", WorkOrSub, OpGroupNonUniformUMin>;
1095: defm : DemangledGroupBuiltin<"group_non_uniform_scan_inclusive_minu", WorkOrSub, OpGroupNonUniformUMin>;
1096: defm : DemangledGroupBuiltin<"group_non_uniform_scan_exclusive_minu", WorkOrSub, OpGroupNonUniformUMin>;
1097: defm : DemangledGroupBuiltin<"group_clustered_reduce_minu", WorkOrSub, OpGroupNonUniformUMin>;
1098: defm : DemangledGroupBuiltinWrapper<"__spirv_GroupNonUniformUMin", 3, 4, OpGroupNonUniformUMin>;
1099:
1100: defm : DemangledGroupBuiltin<"group_non_uniform_fmin", WorkOrSub, OpGroupNonUniformFMin>;
1101: defm : DemangledGroupBuiltin<"group_non_uniform_reduce_minf", WorkOrSub, OpGroupNonUniformFMin>;
1102: defm : DemangledGroupBuiltin<"group_non_uniform_reduce_minh", WorkOrSub, OpGroupNonUniformFMin>;
1103: defm : DemangledGroupBuiltin<"group_non_uniform_reduce_mind", WorkOrSub, OpGroupNonUniformFMin>;
1104: defm : DemangledGroupBuiltin<"group_non_uniform_scan_inclusive_minf", WorkOrSub, OpGroupNonUniformFMin>;
1105: defm : DemangledGroupBuiltin<"group_non_uniform_scan_inclusive_minh", WorkOrSub, OpGroupNonUniformFMin>;
1106: defm : DemangledGroupBuiltin<"group_non_uniform_scan_inclusive_mind", WorkOrSub, OpGroupNonUniformFMin>;
1107: defm : DemangledGroupBuiltin<"group_non_uniform_scan_exclusive_minf", WorkOrSub, OpGroupNonUniformFMin>;
1108: defm : DemangledGroupBuiltin<"group_non_uniform_scan_exclusive_minh", WorkOrSub, OpGroupNonUniformFMin>;
1109: defm : DemangledGroupBuiltin<"group_non_uniform_scan_exclusive_mind", WorkOrSub, OpGroupNonUniformFMin>;
1110: defm : DemangledGroupBuiltin<"group_clustered_reduce_minf", WorkOrSub, OpGroupNonUniformFMin>;
1111: defm : DemangledGroupBuiltin<"group_clustered_reduce_minh", WorkOrSub, OpGroupNonUniformFMin>;
1112: defm : DemangledGroupBuiltin<"group_clustered_reduce_mind", WorkOrSub, OpGroupNonUniformFMin>;
1113: defm : DemangledGroupBuiltinWrapper<"__spirv_GroupNonUniformFMin", 3, 4, OpGroupNonUniformFMin>;
1114:
1115: defm : DemangledGroupBuiltin<"group_non_uniform_smax", WorkOrSub, OpGroupNonUniformSMax>;
1116: defm : DemangledGroupBuiltin<"group_non_uniform_reduce_maxs", WorkOrSub, OpGroupNonUniformSMax>;
1117: defm : DemangledGroupBuiltin<"group_non_uniform_scan_inclusive_maxs", WorkOrSub, OpGroupNonUniformSMax>;
1118: defm : DemangledGroupBuiltin<"group_non_uniform_scan_exclusive_maxs", WorkOrSub, OpGroupNonUniformSMax>;
1119: defm : DemangledGroupBuiltin<"group_clustered_reduce_maxs", WorkOrSub, OpGroupNonUniformSMax>;
1120: defm : DemangledGroupBuiltinWrapper<"__spirv_GroupNonUniformSMax", 3, 4, OpGroupNonUniformSMax>;
1121:
1122: defm : DemangledGroupBuiltin<"group_non_uniform_umax", WorkOrSub, OpGroupNonUniformUMax>;
1123: defm : DemangledGroupBuiltin<"group_non_uniform_reduce_maxu", WorkOrSub, OpGroupNonUniformUMax>;
1124: defm : DemangledGroupBuiltin<"group_non_uniform_scan_inclusive_maxu", WorkOrSub, OpGroupNonUniformUMax>;
1125: defm : DemangledGroupBuiltin<"group_non_uniform_scan_exclusive_maxu", WorkOrSub, OpGroupNonUniformUMax>;
1126: defm : DemangledGroupBuiltin<"group_clustered_reduce_maxu", WorkOrSub, OpGroupNonUniformUMax>;
1127: defm : DemangledGroupBuiltinWrapper<"__spirv_GroupNonUniformUMax", 3, 4, OpGroupNonUniformUMax>;
1128:
1129: defm : DemangledGroupBuiltin<"group_non_uniform_fmax", WorkOrSub, OpGroupNonUniformFMax>;
1130: defm : DemangledGroupBuiltin<"group_non_uniform_reduce_maxf", WorkOrSub, OpGroupNonUniformFMax>;
1131: defm : DemangledGroupBuiltin<"group_non_uniform_reduce_maxh", WorkOrSub, OpGroupNonUniformFMax>;
1132: defm : DemangledGroupBuiltin<"group_non_uniform_reduce_maxd", WorkOrSub, OpGroupNonUniformFMax>;
1133: defm : DemangledGroupBuiltin<"group_non_uniform_scan_inclusive_maxf", WorkOrSub, OpGroupNonUniformFMax>;
1134: defm : DemangledGroupBuiltin<"group_non_uniform_scan_inclusive_maxh", WorkOrSub, OpGroupNonUniformFMax>;
1135: defm : DemangledGroupBuiltin<"group_non_uniform_scan_inclusive_maxd", WorkOrSub, OpGroupNonUniformFMax>;
1136: defm : DemangledGroupBuiltin<"group_non_uniform_scan_exclusive_maxf", WorkOrSub, OpGroupNonUniformFMax>;
1137: defm : DemangledGroupBuiltin<"group_non_uniform_scan_exclusive_maxh", WorkOrSub, OpGroupNonUniformFMax>;
1138: defm : DemangledGroupBuiltin<"group_non_uniform_scan_exclusive_maxd", WorkOrSub, OpGroupNonUniformFMax>;
1139: defm : DemangledGroupBuiltin<"group_clustered_reduce_maxf", WorkOrSub, OpGroupNonUniformFMax>;
1140: defm : DemangledGroupBuiltin<"group_clustered_reduce_maxh", WorkOrSub, OpGroupNonUniformFMax>;
1141: defm : DemangledGroupBuiltin<"group_clustered_reduce_maxd", WorkOrSub, OpGroupNonUniformFMax>;
1142: defm : DemangledGroupBuiltinWrapper<"__spirv_GroupNonUniformFMax", 3, 4, OpGroupNonUniformFMax>;
1143:
1144: defm : DemangledGroupBuiltin<"group_non_uniform_iand", WorkOrSub, OpGroupNonUniformBitwiseAnd>;
1145: defm : DemangledGroupBuiltin<"group_non_uniform_reduce_andu", WorkOrSub, OpGroupNonUniformBitwiseAnd>;
1146: defm : DemangledGroupBuiltin<"group_non_uniform_reduce_ands", WorkOrSub, OpGroupNonUniformBitwiseAnd>;
1147: defm : DemangledGroupBuiltin<"group_non_uniform_scan_inclusive_andu", WorkOrSub, OpGroupNonUniformBitwiseAnd>;
1148: defm : DemangledGroupBuiltin<"group_non_uniform_scan_inclusive_ands", WorkOrSub, OpGroupNonUniformBitwiseAnd>;
1149: defm : DemangledGroupBuiltin<"group_non_uniform_scan_exclusive_andu", WorkOrSub, OpGroupNonUniformBitwiseAnd>;
1150: defm : DemangledGroupBuiltin<"group_non_uniform_scan_exclusive_ands", WorkOrSub, OpGroupNonUniformBitwiseAnd>;
1151: defm : DemangledGroupBuiltin<"group_clustered_reduce_andu", WorkOrSub, OpGroupNonUniformBitwiseAnd>;
1152: defm : DemangledGroupBuiltin<"group_clustered_reduce_ands", WorkOrSub, OpGroupNonUniformBitwiseAnd>;
1153: defm : DemangledGroupBuiltinWrapper<"__spirv_GroupNonUniformBitwiseAnd", 3, 4, OpGroupNonUniformBitwiseAnd>;
1154:
1155: defm : DemangledGroupBuiltin<"group_non_uniform_ior", WorkOrSub, OpGroupNonUniformBitwiseOr>;
1156: defm : DemangledGroupBuiltin<"group_non_uniform_reduce_oru", WorkOrSub, OpGroupNonUniformBitwiseOr>;
1157: defm : DemangledGroupBuiltin<"group_non_uniform_reduce_ors", WorkOrSub, OpGroupNonUniformBitwiseOr>;
1158: defm : DemangledGroupBuiltin<"group_non_uniform_scan_inclusive_oru", WorkOrSub, OpGroupNonUniformBitwiseOr>;
1159: defm : DemangledGroupBuiltin<"group_non_uniform_scan_inclusive_ors", WorkOrSub, OpGroupNonUniformBitwiseOr>;
1160: defm : DemangledGroupBuiltin<"group_non_uniform_scan_exclusive_oru", WorkOrSub, OpGroupNonUniformBitwiseOr>;
1161: defm : DemangledGroupBuiltin<"group_non_uniform_scan_exclusive_ors", WorkOrSub, OpGroupNonUniformBitwiseOr>;
1162: defm : DemangledGroupBuiltin<"group_clustered_reduce_oru", WorkOrSub, OpGroupNonUniformBitwiseOr>;
1163: defm : DemangledGroupBuiltin<"group_clustered_reduce_ors", WorkOrSub, OpGroupNonUniformBitwiseOr>;
1164: defm : DemangledGroupBuiltinWrapper<"__spirv_GroupNonUniformBitwiseOr", 3, 4, OpGroupNonUniformBitwiseOr>;
1165:
1166: defm : DemangledGroupBuiltin<"group_non_uniform_ixor", WorkOrSub, OpGroupNonUniformBitwiseXor>;
1167: defm : DemangledGroupBuiltin<"group_non_uniform_reduce_xoru", WorkOrSub, OpGroupNonUniformBitwiseXor>;
1168: defm : DemangledGroupBuiltin<"group_non_uniform_reduce_xors", WorkOrSub, OpGroupNonUniformBitwiseXor>;
1169: defm : DemangledGroupBuiltin<"group_non_uniform_scan_inclusive_xoru", WorkOrSub, OpGroupNonUniformBitwiseXor>;
1170: defm : DemangledGroupBuiltin<"group_non_uniform_scan_inclusive_xors", WorkOrSub, OpGroupNonUniformBitwiseXor>;
```
- EN: This range uses TableGen DSL to describe records such as backend logic; LLVM later expands these declarations into generated backend tables and helper code.
- CN: 这一段使用 TableGen DSL 描述 后端逻辑 等记录；LLVM 后续会把这些声明展开为生成的后端表和辅助代码。

### Lines 1171-1260
```tablegen
1171: defm : DemangledGroupBuiltin<"group_non_uniform_scan_exclusive_xoru", WorkOrSub, OpGroupNonUniformBitwiseXor>;
1172: defm : DemangledGroupBuiltin<"group_non_uniform_scan_exclusive_xors", WorkOrSub, OpGroupNonUniformBitwiseXor>;
1173: defm : DemangledGroupBuiltin<"group_clustered_reduce_xoru", WorkOrSub, OpGroupNonUniformBitwiseXor>;
1174: defm : DemangledGroupBuiltin<"group_clustered_reduce_xors", WorkOrSub, OpGroupNonUniformBitwiseXor>;
1175: defm : DemangledGroupBuiltinWrapper<"__spirv_GroupNonUniformBitwiseXor", 3, 4, OpGroupNonUniformBitwiseXor>;
1176:
1177: defm : DemangledGroupBuiltin<"group_non_uniform_logical_iand", WorkOrSub, OpGroupNonUniformLogicalAnd>;
1178: defm : DemangledGroupBuiltin<"group_non_uniform_reduce_logical_ands", WorkOrSub, OpGroupNonUniformLogicalAnd>;
1179: defm : DemangledGroupBuiltin<"group_non_uniform_scan_inclusive_logical_ands", WorkOrSub, OpGroupNonUniformLogicalAnd>;
1180: defm : DemangledGroupBuiltin<"group_non_uniform_scan_exclusive_logical_ands", WorkOrSub, OpGroupNonUniformLogicalAnd>;
1181: defm : DemangledGroupBuiltin<"group_clustered_reduce_logical_and", WorkOrSub, OpGroupNonUniformLogicalAnd>;
1182: defm : DemangledGroupBuiltinWrapper<"__spirv_GroupNonUniformLogicalAnd", 3, 4, OpGroupNonUniformLogicalAnd>;
1183:
1184: defm : DemangledGroupBuiltin<"group_non_uniform_logical_ior", WorkOrSub, OpGroupNonUniformLogicalOr>;
1185: defm : DemangledGroupBuiltin<"group_non_uniform_reduce_logical_ors", WorkOrSub, OpGroupNonUniformLogicalOr>;
1186: defm : DemangledGroupBuiltin<"group_non_uniform_scan_inclusive_logical_ors", WorkOrSub, OpGroupNonUniformLogicalOr>;
1187: defm : DemangledGroupBuiltin<"group_non_uniform_scan_exclusive_logical_ors", WorkOrSub, OpGroupNonUniformLogicalOr>;
1188: defm : DemangledGroupBuiltin<"group_clustered_reduce_logical_or", WorkOrSub, OpGroupNonUniformLogicalOr>;
1189: defm : DemangledGroupBuiltinWrapper<"__spirv_GroupNonUniformLogicalOr", 3, 4, OpGroupNonUniformLogicalOr>;
1190:
1191: defm : DemangledGroupBuiltin<"group_non_uniform_logical_ixor", WorkOrSub, OpGroupNonUniformLogicalXor>;
1192: defm : DemangledGroupBuiltin<"group_non_uniform_reduce_logical_xors", WorkOrSub, OpGroupNonUniformLogicalXor>;
1193: defm : DemangledGroupBuiltin<"group_non_uniform_scan_inclusive_logical_xors", WorkOrSub, OpGroupNonUniformLogicalXor>;
1194: defm : DemangledGroupBuiltin<"group_non_uniform_scan_exclusive_logical_xors", WorkOrSub, OpGroupNonUniformLogicalXor>;
1195: defm : DemangledGroupBuiltin<"group_clustered_reduce_logical_xor", WorkOrSub, OpGroupNonUniformLogicalXor>;
1196: defm : DemangledGroupBuiltinWrapper<"__spirv_GroupNonUniformLogicalXor", 3, 4, OpGroupNonUniformLogicalXor>;
1197:
1198: // cl_khr_subgroup_rotate / SPV_KHR_subgroup_rotate
1199: defm : DemangledGroupBuiltin<"group_rotate", OnlySub, OpGroupNonUniformRotateKHR>;
1200: defm : DemangledGroupBuiltin<"group_clustered_rotate", OnlySub, OpGroupNonUniformRotateKHR>;
1201:
1202: // cl_khr_work_group_uniform_arithmetic / SPV_KHR_uniform_group_instructions
1203: defm : DemangledGroupBuiltin<"group_reduce_imul", OnlyWork, OpGroupIMulKHR>;
1204: defm : DemangledGroupBuiltin<"group_reduce_mulu", OnlyWork, OpGroupIMulKHR>;
1205: defm : DemangledGroupBuiltin<"group_reduce_muls", OnlyWork, OpGroupIMulKHR>;
1206: defm : DemangledGroupBuiltin<"group_scan_inclusive_imul", OnlyWork, OpGroupIMulKHR>;
1207: defm : DemangledGroupBuiltin<"group_scan_inclusive_mulu", OnlyWork, OpGroupIMulKHR>;
1208: defm : DemangledGroupBuiltin<"group_scan_inclusive_muls", OnlyWork, OpGroupIMulKHR>;
1209: defm : DemangledGroupBuiltin<"group_scan_exclusive_imul", OnlyWork, OpGroupIMulKHR>;
1210: defm : DemangledGroupBuiltin<"group_scan_exclusive_mulu", OnlyWork, OpGroupIMulKHR>;
1211: defm : DemangledGroupBuiltin<"group_scan_exclusive_muls", OnlyWork, OpGroupIMulKHR>;
1212:
1213: defm : DemangledGroupBuiltin<"group_reduce_mulf", OnlyWork, OpGroupFMulKHR>;
1214: defm : DemangledGroupBuiltin<"group_reduce_mulh", OnlyWork, OpGroupFMulKHR>;
1215: defm : DemangledGroupBuiltin<"group_reduce_muld", OnlyWork, OpGroupFMulKHR>;
1216: defm : DemangledGroupBuiltin<"group_scan_inclusive_mulf", OnlyWork, OpGroupFMulKHR>;
1217: defm : DemangledGroupBuiltin<"group_scan_inclusive_mulh", OnlyWork, OpGroupFMulKHR>;
1218: defm : DemangledGroupBuiltin<"group_scan_inclusive_muld", OnlyWork, OpGroupFMulKHR>;
1219: defm : DemangledGroupBuiltin<"group_scan_exclusive_mulf", OnlyWork, OpGroupFMulKHR>;
1220: defm : DemangledGroupBuiltin<"group_scan_exclusive_mulh", OnlyWork, OpGroupFMulKHR>;
1221: defm : DemangledGroupBuiltin<"group_scan_exclusive_muld", OnlyWork, OpGroupFMulKHR>;
1222:
1223: defm : DemangledGroupBuiltin<"group_scan_exclusive_and", OnlyWork, OpGroupBitwiseAndKHR>;
1224: defm : DemangledGroupBuiltin<"group_scan_inclusive_and", OnlyWork, OpGroupBitwiseAndKHR>;
1225: defm : DemangledGroupBuiltin<"group_reduce_and", OnlyWork, OpGroupBitwiseAndKHR>;
1226:
1227: defm : DemangledGroupBuiltin<"group_scan_exclusive_or", OnlyWork, OpGroupBitwiseOrKHR>;
1228: defm : DemangledGroupBuiltin<"group_scan_inclusive_or", OnlyWork, OpGroupBitwiseOrKHR>;
1229: defm : DemangledGroupBuiltin<"group_reduce_or", OnlyWork, OpGroupBitwiseOrKHR>;
1230:
1231: defm : DemangledGroupBuiltin<"group_scan_exclusive_xor", OnlyWork, OpGroupBitwiseXorKHR>;
1232: defm : DemangledGroupBuiltin<"group_scan_inclusive_xor", OnlyWork, OpGroupBitwiseXorKHR>;
1233: defm : DemangledGroupBuiltin<"group_reduce_xor", OnlyWork, OpGroupBitwiseXorKHR>;
1234:
1235: defm : DemangledGroupBuiltin<"group_scan_exclusive_logical_and", OnlyWork, OpGroupLogicalAndKHR>;
1236: defm : DemangledGroupBuiltin<"group_scan_inclusive_logical_and", OnlyWork, OpGroupLogicalAndKHR>;
1237: defm : DemangledGroupBuiltin<"group_reduce_logical_and", OnlyWork, OpGroupLogicalAndKHR>;
1238:
1239: defm : DemangledGroupBuiltin<"group_scan_exclusive_logical_or", OnlyWork, OpGroupLogicalOrKHR>;
1240: defm : DemangledGroupBuiltin<"group_scan_inclusive_logical_or", OnlyWork, OpGroupLogicalOrKHR>;
1241: defm : DemangledGroupBuiltin<"group_reduce_logical_or", OnlyWork, OpGroupLogicalOrKHR>;
1242:
1243: defm : DemangledGroupBuiltin<"group_scan_exclusive_logical_xor", OnlyWork, OpGroupLogicalXorKHR>;
1244: defm : DemangledGroupBuiltin<"group_scan_inclusive_logical_xor", OnlyWork, OpGroupLogicalXorKHR>;
1245: defm : DemangledGroupBuiltin<"group_reduce_logical_xor", OnlyWork, OpGroupLogicalXorKHR>;
1246:
1247: // cl_khr_kernel_clock / SPV_KHR_shader_clock
1248: defm : DemangledNativeBuiltin<"clock_read_device", OpenCL_std, KernelClock, 0, 0, OpReadClockKHR>;
1249: defm : DemangledNativeBuiltin<"clock_read_work_group", OpenCL_std, KernelClock, 0, 0, OpReadClockKHR>;
1250: defm : DemangledNativeBuiltin<"clock_read_sub_group", OpenCL_std, KernelClock, 0, 0, OpReadClockKHR>;
1251: defm : DemangledNativeBuiltin<"clock_read_hilo_device", OpenCL_std, KernelClock, 0, 0, OpReadClockKHR>;
1252: defm : DemangledNativeBuiltin<"clock_read_hilo_work_group", OpenCL_std, KernelClock, 0, 0, OpReadClockKHR>;
1253: defm : DemangledNativeBuiltin<"clock_read_hilo_sub_group", OpenCL_std, KernelClock, 0, 0, OpReadClockKHR>;
1254:
1255: //SPV_ALTERA_blocking_pipes
1256: defm : DemangledNativeBuiltin<"__spirv_WritePipeBlockingINTEL", OpenCL_std, BlockingPipes, 0, 0, OpWritePipeBlockingALTERA>;
1257: defm : DemangledNativeBuiltin<"__spirv_ReadPipeBlockingINTEL", OpenCL_std, BlockingPipes, 0, 0, OpReadPipeBlockingALTERA>;
1258: defm : DemangledNativeBuiltin<"__spirv_ReadClockKHR", OpenCL_std, KernelClock, 1, 1, OpReadClockKHR>;
1259:
1260: //SPV_ALTERA_arbitrary_precision_fixed_point
```
- EN: This range uses TableGen DSL to describe records such as backend logic; LLVM later expands these declarations into generated backend tables and helper code.
- CN: 这一段使用 TableGen DSL 描述 后端逻辑 等记录；LLVM 后续会把这些声明展开为生成的后端表和辅助代码。

### Lines 1261-1350
```tablegen
1261: defm : DemangledNativeBuiltin<"__spirv_FixedSqrtINTEL", OpenCL_std, ArbitraryPrecisionFixedPoint, 6 , 8, OpFixedSqrtALTERA>;
1262: defm : DemangledNativeBuiltin<"__spirv_FixedRecipINTEL", OpenCL_std, ArbitraryPrecisionFixedPoint, 6 , 8, OpFixedRecipALTERA>;
1263: defm : DemangledNativeBuiltin<"__spirv_FixedRsqrtINTEL", OpenCL_std, ArbitraryPrecisionFixedPoint, 6 , 8, OpFixedRsqrtALTERA>;
1264: defm : DemangledNativeBuiltin<"__spirv_FixedSinINTEL", OpenCL_std, ArbitraryPrecisionFixedPoint, 6 , 8, OpFixedSinALTERA>;
1265: defm : DemangledNativeBuiltin<"__spirv_FixedCosINTEL", OpenCL_std, ArbitraryPrecisionFixedPoint, 6 , 8, OpFixedCosALTERA>;
1266: defm : DemangledNativeBuiltin<"__spirv_FixedSinCosINTEL", OpenCL_std, ArbitraryPrecisionFixedPoint, 6 , 8, OpFixedSinCosALTERA>;
1267: defm : DemangledNativeBuiltin<"__spirv_FixedSinPiINTEL", OpenCL_std, ArbitraryPrecisionFixedPoint, 6 , 8, OpFixedSinPiALTERA>;
1268: defm : DemangledNativeBuiltin<"__spirv_FixedCosPiINTEL", OpenCL_std, ArbitraryPrecisionFixedPoint, 6 , 8, OpFixedCosPiALTERA>;
1269: defm : DemangledNativeBuiltin<"__spirv_FixedSinCosPiINTEL", OpenCL_std, ArbitraryPrecisionFixedPoint, 6 , 8, OpFixedSinCosPiALTERA>;
1270: defm : DemangledNativeBuiltin<"__spirv_FixedLogINTEL", OpenCL_std, ArbitraryPrecisionFixedPoint, 6 , 8, OpFixedLogALTERA>;
1271: defm : DemangledNativeBuiltin<"__spirv_FixedExpINTEL", OpenCL_std, ArbitraryPrecisionFixedPoint, 6 , 8, OpFixedExpALTERA>;
1272:
1273: //===----------------------------------------------------------------------===//
1274: // Class defining an atomic instruction on floating-point numbers.
1275: //
1276: // name is the demangled name of the given builtin.
1277: // opcode specifies the SPIR-V operation code of the generated instruction.
1278: //===----------------------------------------------------------------------===//
1279: class AtomicFloatingBuiltin<string name, Op operation> {
1280:   string Name = name;
1281:   Op Opcode = operation;
1282: }
1283:
1284: // Table gathering all builtins for atomic instructions on floating-point numbers
1285: def AtomicFloatingBuiltins : GenericTable {
1286:   let FilterClass = "AtomicFloatingBuiltin";
1287:   let Fields = ["Name", "Opcode"];
1288: }
1289:
1290: // Function to lookup builtins by their name and set.
1291: def lookupAtomicFloatingBuiltin : SearchIndex {
1292:   let Table = AtomicFloatingBuiltins;
1293:   let Key = ["Name"];
1294: }
1295:
1296: // Multiclass used to define incoming demangled builtin records and
1297: // corresponding builtin records for atomic instructions on floating-point numbers.
1298: multiclass DemangledAtomicFloatingBuiltin<string name, bits<8> minNumArgs, bits<8> maxNumArgs, Op operation> {
1299:   def : DemangledBuiltin<!strconcat("__spirv_AtomicF", name), OpenCL_std, AtomicFloating, minNumArgs, maxNumArgs>;
1300:   def : AtomicFloatingBuiltin<!strconcat("__spirv_AtomicF", name), operation>;
1301: }
1302:
1303: // SPV_EXT_shader_atomic_float_add, SPV_EXT_shader_atomic_float_min_max, SPV_EXT_shader_atomic_float16_add
1304: // Atomic add, min and max instruction on floating-point numbers:
1305: defm : DemangledAtomicFloatingBuiltin<"AddEXT", 4, 4, OpAtomicFAddEXT>;
1306: defm : DemangledAtomicFloatingBuiltin<"MinEXT", 4, 4, OpAtomicFMinEXT>;
1307: defm : DemangledAtomicFloatingBuiltin<"MaxEXT", 4, 4, OpAtomicFMaxEXT>;
1308:
1309: //===----------------------------------------------------------------------===//
1310: // Class defining a sub group builtin that should be translated into a
1311: // SPIR-V instruction using the SPV_INTEL_subgroups extension.
1312: //
1313: // name is the demangled name of the given builtin.
1314: // opcode specifies the SPIR-V operation code of the generated instruction.
1315: //===----------------------------------------------------------------------===//
1316: class IntelSubgroupsBuiltin<string name, Op operation> {
1317:   string Name = name;
1318:   Op Opcode = operation;
1319:   bit IsBlock = !or(!eq(operation, OpSubgroupBlockReadINTEL),
1320:                     !eq(operation, OpSubgroupBlockWriteINTEL),
1321:                     !eq(operation, OpSubgroupImageMediaBlockReadINTEL),
1322:                     !eq(operation, OpSubgroupImageMediaBlockWriteINTEL));
1323:   bit IsWrite = !or(!eq(operation, OpSubgroupBlockWriteINTEL),
1324:                     !eq(operation, OpSubgroupImageMediaBlockWriteINTEL));
1325:   bit IsMedia = !or(!eq(operation, OpSubgroupImageMediaBlockReadINTEL),
1326:                     !eq(operation, OpSubgroupImageMediaBlockWriteINTEL));
1327: }
1328:
1329: // Table gathering all the Intel sub group builtins.
1330: def IntelSubgroupsBuiltins : GenericTable {
1331:   let FilterClass = "IntelSubgroupsBuiltin";
1332:   let Fields = ["Name", "Opcode", "IsBlock", "IsWrite", "IsMedia"];
1333: }
1334:
1335: // Function to lookup group builtins by their name and set.
1336: def lookupIntelSubgroupsBuiltin : SearchIndex {
1337:   let Table = IntelSubgroupsBuiltins;
1338:   let Key = ["Name"];
1339: }
1340:
1341: // Multiclass used to define incoming builtin records for the SPV_INTEL_subgroups extension
1342: // and corresponding work/sub group builtin records.
1343: multiclass DemangledIntelSubgroupsBuiltin<string name, bits<8> minNumArgs, bits<8> maxNumArgs, Op operation> {
1344:   def : DemangledBuiltin<!strconcat("intel_sub_group_", name), OpenCL_std, IntelSubgroups, minNumArgs, maxNumArgs>;
1345:   def : IntelSubgroupsBuiltin<!strconcat("intel_sub_group_", name), operation>;
1346: }
1347:
1348: // cl_intel_subgroups
1349: defm : DemangledIntelSubgroupsBuiltin<"shuffle", 2, 2, OpSubgroupShuffleINTEL>;
1350: defm : DemangledIntelSubgroupsBuiltin<"shuffle_down", 3, 3, OpSubgroupShuffleDownINTEL>;
```
- EN: This range uses TableGen DSL to describe records such as AtomicFloatingBuiltin, AtomicFloatingBuiltins, lookupAtomicFloatingBuiltin, DemangledAtomicFloatingBuiltin; LLVM later expands these declarations into generated backend tables and helper code.
- CN: 这一段使用 TableGen DSL 描述 AtomicFloatingBuiltin、AtomicFloatingBuiltins、lookupAtomicFloatingBuiltin、DemangledAtomicFloatingBuiltin 等记录；LLVM 后续会把这些声明展开为生成的后端表和辅助代码。

### Lines 1351-1440
```tablegen
1351: defm : DemangledIntelSubgroupsBuiltin<"shuffle_up", 3, 3, OpSubgroupShuffleUpINTEL>;
1352: defm : DemangledIntelSubgroupsBuiltin<"shuffle_xor", 2, 2, OpSubgroupShuffleXorINTEL>;
1353: foreach i = ["", "2", "4", "8"] in {
1354:   // cl_intel_subgroups, cl_intel_subgroup_local_block_io
1355:   defm : DemangledIntelSubgroupsBuiltin<!strconcat("block_read",  i), 1, 2, OpSubgroupBlockReadINTEL>;
1356:   defm : DemangledIntelSubgroupsBuiltin<!strconcat("block_write", i), 2, 3, OpSubgroupBlockWriteINTEL>;
1357:   // cl_intel_subgroups_short, cl_intel_subgroup_local_block_io
1358:   defm : DemangledIntelSubgroupsBuiltin<!strconcat("block_read_ui",  i), 1, 2, OpSubgroupBlockReadINTEL>;
1359:   defm : DemangledIntelSubgroupsBuiltin<!strconcat("block_write_ui", i), 2, 3, OpSubgroupBlockWriteINTEL>;
1360:   // cl_intel_media_block_io
1361:   defm : DemangledIntelSubgroupsBuiltin<!strconcat("media_block_read",  i), 4, 4, OpSubgroupImageMediaBlockReadINTEL>;
1362:   defm : DemangledIntelSubgroupsBuiltin<!strconcat("media_block_read_ui",  i), 4, 4, OpSubgroupImageMediaBlockReadINTEL>;
1363:   defm : DemangledIntelSubgroupsBuiltin<!strconcat("media_block_write", i), 5, 5, OpSubgroupImageMediaBlockWriteINTEL>;
1364:   defm : DemangledIntelSubgroupsBuiltin<!strconcat("media_block_write_ui", i), 5, 5, OpSubgroupImageMediaBlockWriteINTEL>;
1365: }
1366: // cl_intel_subgroups_char, cl_intel_subgroups_short, cl_intel_subgroups_long, cl_intel_media_block_io, cl_intel_subgroup_local_block_io
1367: foreach i = ["", "2", "4", "8", "16"] in {
1368:   foreach j = ["c", "s", "l"] in {
1369:     defm : DemangledIntelSubgroupsBuiltin<!strconcat("block_read_u", j,  i), 1, 2, OpSubgroupBlockReadINTEL>;
1370:     defm : DemangledIntelSubgroupsBuiltin<!strconcat("block_write_u", j, i), 2, 3, OpSubgroupBlockWriteINTEL>;
1371:     defm : DemangledIntelSubgroupsBuiltin<!strconcat("media_block_read_u", j, i), 4, 4, OpSubgroupImageMediaBlockReadINTEL>;
1372:     defm : DemangledIntelSubgroupsBuiltin<!strconcat("media_block_write_u", j, i), 5, 5, OpSubgroupImageMediaBlockWriteINTEL>;
1373:   }
1374: }
1375: // OpSubgroupImageBlockReadINTEL and OpSubgroupImageBlockWriteINTEL are to be resolved later on (in code)
1376:
1377: // Multiclass used to define builtin wrappers for the SPV_INTEL_subgroups and the SPV_INTEL_media_block_io extensions.
1378: multiclass DemangledIntelSubgroupsBuiltinWrapper<string name, bits<8> numArgs, Op operation> {
1379:   def : DemangledBuiltin<!strconcat("__spirv_", name), OpenCL_std, IntelSubgroups, numArgs, numArgs>;
1380:   def : IntelSubgroupsBuiltin<!strconcat("__spirv_", name), operation>;
1381: }
1382:
1383: defm : DemangledIntelSubgroupsBuiltinWrapper<"SubgroupShuffleINTEL", 2, OpSubgroupShuffleINTEL>;
1384: defm : DemangledIntelSubgroupsBuiltinWrapper<"SubgroupShuffleDownINTEL", 3, OpSubgroupShuffleDownINTEL>;
1385: defm : DemangledIntelSubgroupsBuiltinWrapper<"SubgroupShuffleUpINTEL", 3, OpSubgroupShuffleUpINTEL>;
1386: defm : DemangledIntelSubgroupsBuiltinWrapper<"SubgroupShuffleXorINTEL", 2, OpSubgroupShuffleXorINTEL>;
1387: defm : DemangledIntelSubgroupsBuiltinWrapper<"SubgroupBlockReadINTEL", 1, OpSubgroupBlockReadINTEL>;
1388: defm : DemangledIntelSubgroupsBuiltinWrapper<"SubgroupBlockWriteINTEL", 2, OpSubgroupBlockWriteINTEL>;
1389: defm : DemangledIntelSubgroupsBuiltinWrapper<"SubgroupImageBlockReadINTEL", 2, OpSubgroupImageBlockReadINTEL>;
1390: defm : DemangledIntelSubgroupsBuiltinWrapper<"SubgroupImageBlockWriteINTEL", 3, OpSubgroupImageBlockWriteINTEL>;
1391: defm : DemangledIntelSubgroupsBuiltinWrapper<"SubgroupImageMediaBlockReadINTEL", 4, OpSubgroupImageMediaBlockReadINTEL>;
1392: defm : DemangledIntelSubgroupsBuiltinWrapper<"SubgroupImageMediaBlockWriteINTEL", 5, OpSubgroupImageMediaBlockWriteINTEL>;
1393:
1394: //===----------------------------------------------------------------------===//
1395: // Class defining a builtin for group operations within uniform control flow.
1396: // It should be translated into a SPIR-V instruction using
1397: // the SPV_KHR_uniform_group_instructions extension.
1398: //
1399: // name is the demangled name of the given builtin.
1400: // opcode specifies the SPIR-V operation code of the generated instruction.
1401: //===----------------------------------------------------------------------===//
1402: class GroupUniformBuiltin<string name, Op operation> {
1403:   string Name = name;
1404:   Op Opcode = operation;
1405:   bit IsLogical = !or(!eq(operation, OpGroupLogicalAndKHR),
1406:                       !eq(operation, OpGroupLogicalOrKHR),
1407:                       !eq(operation, OpGroupLogicalXorKHR));
1408: }
1409:
1410: // Table gathering all the Intel sub group builtins.
1411: def GroupUniformBuiltins : GenericTable {
1412:   let FilterClass = "GroupUniformBuiltin";
1413:   let Fields = ["Name", "Opcode", "IsLogical"];
1414: }
1415:
1416: // Function to lookup group builtins by their name and set.
1417: def lookupGroupUniformBuiltin : SearchIndex {
1418:   let Table = GroupUniformBuiltins;
1419:   let Key = ["Name"];
1420: }
1421:
1422: // Multiclass used to define incoming builtin records for
1423: // the SPV_KHR_uniform_group_instructions extension
1424: // and corresponding work group builtin records.
1425: multiclass DemangledGroupUniformBuiltin<string name, bits<8> minNumArgs, bits<8> maxNumArgs, Op operation> {
1426:   def : DemangledBuiltin<!strconcat("__spirv_Group", name), OpenCL_std, GroupUniform, minNumArgs, maxNumArgs>;
1427:   def : GroupUniformBuiltin<!strconcat("__spirv_Group", name), operation>;
1428: }
1429:
1430: // cl_khr_work_group_uniform_arithmetic / SPV_KHR_uniform_group_instructions
1431: defm : DemangledGroupUniformBuiltin<"IMulKHR", 3, 3, OpGroupIMulKHR>;
1432: defm : DemangledGroupUniformBuiltin<"FMulKHR", 3, 3, OpGroupFMulKHR>;
1433: defm : DemangledGroupUniformBuiltin<"BitwiseAndKHR", 3, 3, OpGroupBitwiseAndKHR>;
1434: defm : DemangledGroupUniformBuiltin<"BitwiseOrKHR", 3, 3, OpGroupBitwiseOrKHR>;
1435: defm : DemangledGroupUniformBuiltin<"BitwiseXorKHR", 3, 3, OpGroupBitwiseXorKHR>;
1436: defm : DemangledGroupUniformBuiltin<"LogicalAndKHR", 3, 3, OpGroupLogicalAndKHR>;
1437: defm : DemangledGroupUniformBuiltin<"LogicalOrKHR", 3, 3, OpGroupLogicalOrKHR>;
1438: defm : DemangledGroupUniformBuiltin<"LogicalXorKHR", 3, 3, OpGroupLogicalXorKHR>;
1439:
1440: //===----------------------------------------------------------------------===//
```
- EN: This range uses TableGen DSL to describe records such as DemangledIntelSubgroupsBuiltinWrapper, GroupUniformBuiltin, GroupUniformBuiltins, lookupGroupUniformBuiltin; LLVM later expands these declarations into generated backend tables and helper code.
- CN: 这一段使用 TableGen DSL 描述 DemangledIntelSubgroupsBuiltinWrapper、GroupUniformBuiltin、GroupUniformBuiltins、lookupGroupUniformBuiltin 等记录；LLVM 后续会把这些声明展开为生成的后端表和辅助代码。

### Lines 1441-1530
```tablegen
1441: // Class defining a get builtin record used for lowering builtin calls such as
1442: // "get_sub_group_eq_mask" or "get_global_id" to SPIR-V instructions.
1443: //
1444: // name is the demangled name of the given builtin.
1445: // set specifies which external instruction set the builtin belongs to.
1446: // value specifies the value of the BuiltIn enum.
1447: //===----------------------------------------------------------------------===//
1448: class GetBuiltin<string name, InstructionSet set, BuiltIn value> {
1449:   string Name = name;
1450:   InstructionSet Set = set;
1451:   BuiltIn Value = value;
1452: }
1453:
1454: // Table gathering all the get builtin records.
1455: def GetBuiltins : GenericTable {
1456:   let FilterClass = "GetBuiltin";
1457:   let Fields = ["Name", "Set", "Value"];
1458:   string TypeOf_Set = "InstructionSet";
1459:   string TypeOf_Value = "BuiltIn";
1460: }
1461:
1462: // Function to lookup get builtin records by their name and set.
1463: def lookupGetBuiltin : SearchIndex {
1464:   let Table = GetBuiltins;
1465:   let Key = ["Name", "Set"];
1466: }
1467:
1468: // Multiclass used to define at the same time both a demangled builtin record
1469: // and a corresponding get builtin record.
1470: multiclass DemangledGetBuiltin<string name, InstructionSet set, BuiltinGroup group, BuiltIn value> {
1471:   def : DemangledBuiltin<name, set, group, 0, 1>;
1472:   def : GetBuiltin<name, set, value>;
1473: }
1474:
1475: // Builtin variable records:
1476: defm : DemangledGetBuiltin<"get_global_linear_id", OpenCL_std, Variable, GlobalLinearId>;
1477: defm : DemangledGetBuiltin<"get_local_linear_id", OpenCL_std, Variable, LocalInvocationIndex>;
1478: defm : DemangledGetBuiltin<"get_work_dim", OpenCL_std, Variable, WorkDim>;
1479: defm : DemangledGetBuiltin<"get_sub_group_size", OpenCL_std, Variable, SubgroupSize>;
1480: defm : DemangledGetBuiltin<"get_max_sub_group_size", OpenCL_std, Variable, SubgroupMaxSize>;
1481: defm : DemangledGetBuiltin<"get_num_sub_groups", OpenCL_std, Variable, NumSubgroups>;
1482: defm : DemangledGetBuiltin<"get_enqueued_num_sub_groups", OpenCL_std, Variable, NumEnqueuedSubgroups>;
1483: defm : DemangledGetBuiltin<"get_sub_group_id", OpenCL_std, Variable, SubgroupId>;
1484: defm : DemangledGetBuiltin<"get_sub_group_local_id", OpenCL_std, Variable, SubgroupLocalInvocationId>;
1485: defm : DemangledGetBuiltin<"get_sub_group_eq_mask", OpenCL_std, Variable, SubgroupEqMask>;
1486: defm : DemangledGetBuiltin<"get_sub_group_ge_mask", OpenCL_std, Variable, SubgroupGeMask>;
1487: defm : DemangledGetBuiltin<"get_sub_group_gt_mask", OpenCL_std, Variable, SubgroupGtMask>;
1488: defm : DemangledGetBuiltin<"get_sub_group_le_mask", OpenCL_std, Variable, SubgroupLeMask>;
1489: defm : DemangledGetBuiltin<"get_sub_group_lt_mask", OpenCL_std, Variable, SubgroupLtMask>;
1490: defm : DemangledGetBuiltin<"__spirv_BuiltInGlobalLinearId", OpenCL_std, Variable, GlobalLinearId>;
1491: defm : DemangledGetBuiltin<"__spirv_BuiltInLocalInvocationIndex", OpenCL_std, Variable, LocalInvocationIndex>;
1492: defm : DemangledGetBuiltin<"__spirv_BuiltInWorkDim", OpenCL_std, Variable, WorkDim>;
1493: defm : DemangledGetBuiltin<"__spirv_BuiltInSubgroupSize", OpenCL_std, Variable, SubgroupSize>;
1494: defm : DemangledGetBuiltin<"__spirv_BuiltInSubgroupMaxSize", OpenCL_std, Variable, SubgroupMaxSize>;
1495: defm : DemangledGetBuiltin<"__spirv_BuiltInNumSubgroups", OpenCL_std, Variable, NumSubgroups>;
1496: defm : DemangledGetBuiltin<"__spirv_BuiltInNumEnqueuedSubgroups", OpenCL_std, Variable, NumEnqueuedSubgroups>;
1497: defm : DemangledGetBuiltin<"__spirv_BuiltInSubgroupId", OpenCL_std, Variable, SubgroupId>;
1498: defm : DemangledGetBuiltin<"__spirv_BuiltInSubgroupLocalInvocationId", OpenCL_std, Variable, SubgroupLocalInvocationId>;
1499: defm : DemangledGetBuiltin<"__spirv_BuiltInSubgroupEqMask", OpenCL_std, Variable, SubgroupEqMask>;
1500: defm : DemangledGetBuiltin<"__spirv_BuiltInSubgroupEqMaskKHR", OpenCL_std, Variable, SubgroupEqMask>;
1501: defm : DemangledGetBuiltin<"__spirv_BuiltInSubgroupGeMask", OpenCL_std, Variable, SubgroupGeMask>;
1502: defm : DemangledGetBuiltin<"__spirv_BuiltInSubgroupGeMaskKHR", OpenCL_std, Variable, SubgroupGeMask>;
1503: defm : DemangledGetBuiltin<"__spirv_BuiltInSubgroupGtMask", OpenCL_std, Variable, SubgroupGtMask>;
1504: defm : DemangledGetBuiltin<"__spirv_BuiltInSubgroupGtMaskKHR", OpenCL_std, Variable, SubgroupGtMask>;
1505: defm : DemangledGetBuiltin<"__spirv_BuiltInSubgroupLeMask", OpenCL_std, Variable, SubgroupLeMask>;
1506: defm : DemangledGetBuiltin<"__spirv_BuiltInSubgroupLeMaskKHR", OpenCL_std, Variable, SubgroupLeMask>;
1507: defm : DemangledGetBuiltin<"__spirv_BuiltInSubgroupLtMask", OpenCL_std, Variable, SubgroupLtMask>;
1508: defm : DemangledGetBuiltin<"__spirv_BuiltInSubgroupLtMaskKHR", OpenCL_std, Variable, SubgroupLtMask>;
1509:
1510: // GetQuery builtin records:
1511: defm : DemangledGetBuiltin<"get_local_id", OpenCL_std, GetQuery, LocalInvocationId>;
1512: defm : DemangledGetBuiltin<"get_global_id", OpenCL_std, GetQuery, GlobalInvocationId>;
1513: defm : DemangledGetBuiltin<"get_local_size", OpenCL_std, GetQuery, WorkgroupSize>;
1514: defm : DemangledGetBuiltin<"get_global_size", OpenCL_std, GetQuery, GlobalSize>;
1515: defm : DemangledGetBuiltin<"get_group_id", OpenCL_std, GetQuery, WorkgroupId>;
1516: defm : DemangledGetBuiltin<"get_enqueued_local_size", OpenCL_std, GetQuery, EnqueuedWorkgroupSize>;
1517: defm : DemangledGetBuiltin<"get_num_groups", OpenCL_std, GetQuery, NumWorkgroups>;
1518: defm : DemangledGetBuiltin<"get_global_offset", OpenCL_std, GetQuery, GlobalOffset>;
1519: defm : DemangledGetBuiltin<"__spirv_BuiltInLocalInvocationId", OpenCL_std, GetQuery, LocalInvocationId>;
1520: defm : DemangledGetBuiltin<"__spirv_BuiltInGlobalInvocationId", OpenCL_std, GetQuery, GlobalInvocationId>;
1521: defm : DemangledGetBuiltin<"__spirv_BuiltInWorkgroupSize", OpenCL_std, GetQuery, WorkgroupSize>;
1522: defm : DemangledGetBuiltin<"__spirv_BuiltInGlobalSize", OpenCL_std, GetQuery, GlobalSize>;
1523: defm : DemangledGetBuiltin<"__spirv_BuiltInWorkgroupId", OpenCL_std, GetQuery, WorkgroupId>;
1524: defm : DemangledGetBuiltin<"__spirv_BuiltInEnqueuedWorkgroupSize", OpenCL_std, GetQuery, EnqueuedWorkgroupSize>;
1525: defm : DemangledGetBuiltin<"__spirv_BuiltInNumWorkgroups", OpenCL_std, GetQuery, NumWorkgroups>;
1526: defm : DemangledGetBuiltin<"__spirv_BuiltInGlobalOffset", OpenCL_std, GetQuery, GlobalOffset>;
1527: defm : DemangledGetBuiltin<"__hlsl_wave_get_lane_index", GLSL_std_450, Wave, SubgroupLocalInvocationId>;
1528:
1529: //===----------------------------------------------------------------------===//
1530: // Class defining an image query builtin record used for lowering the OpenCL
```
- EN: This range uses TableGen DSL to describe records such as GetBuiltin, GetBuiltins, lookupGetBuiltin, DemangledGetBuiltin; LLVM later expands these declarations into generated backend tables and helper code.
- CN: 这一段使用 TableGen DSL 描述 GetBuiltin、GetBuiltins、lookupGetBuiltin、DemangledGetBuiltin 等记录；LLVM 后续会把这些声明展开为生成的后端表和辅助代码。

### Lines 1531-1620
```tablegen
1531: // "get_image_*" calls into OpImageQuerySize/OpImageQuerySizeLod instructions.
1532: //
1533: // name is the demangled name of the given builtin.
1534: // set specifies which external instruction set the builtin belongs to.
1535: // component specifies the unsigned number of the query component.
1536: //===----------------------------------------------------------------------===//
1537: class ImageQueryBuiltin<string name, InstructionSet set, bits<32> component> {
1538:   string Name = name;
1539:   InstructionSet Set = set;
1540:   bits<32> Component = component;
1541: }
1542:
1543: // Table gathering all the image query builtins.
1544: def ImageQueryBuiltins : GenericTable {
1545:   let FilterClass = "ImageQueryBuiltin";
1546:   let Fields = ["Name", "Set", "Component"];
1547:   string TypeOf_Set = "InstructionSet";
1548: }
1549:
1550: // Function to lookup image query builtins by their name and set.
1551: def lookupImageQueryBuiltin : SearchIndex {
1552:   let Table = ImageQueryBuiltins;
1553:   let Key = ["Name", "Set"];
1554: }
1555:
1556: // Multiclass used to define at the same time both a demangled builtin record
1557: // and a corresponding image query builtin record.
1558: multiclass DemangledImageQueryBuiltin<string name, InstructionSet set, int component> {
1559:   def : DemangledBuiltin<name, set, ImageSizeQuery, 1, 1>;
1560:   def : ImageQueryBuiltin<name, set, component>;
1561: }
1562:
1563: // Image query builtin records:
1564: defm : DemangledImageQueryBuiltin<"get_image_width", OpenCL_std, 0>;
1565: defm : DemangledImageQueryBuiltin<"get_image_height", OpenCL_std, 1>;
1566: defm : DemangledImageQueryBuiltin<"get_image_depth", OpenCL_std, 2>;
1567: defm : DemangledImageQueryBuiltin<"get_image_dim", OpenCL_std, 0>;
1568: defm : DemangledImageQueryBuiltin<"get_image_array_size", OpenCL_std, 3>;
1569:
1570: defm : DemangledNativeBuiltin<"get_image_num_samples", OpenCL_std, ImageMiscQuery, 1, 1, OpImageQuerySamples>;
1571: defm : DemangledNativeBuiltin<"get_image_num_mip_levels", OpenCL_std, ImageMiscQuery, 1, 1, OpImageQueryLevels>;
1572: defm : DemangledNativeBuiltin<"get_image_channel_data_type", OpenCL_std, ImageChannelDataTypes, 1, 1, OpImageQueryFormat>;
1573:
1574: //===----------------------------------------------------------------------===//
1575: // Class defining a "convert_destType<_sat><_roundingMode>" call record for
1576: // lowering into OpConvert instructions.
1577: //
1578: // name is the demangled name of the given builtin.
1579: // set specifies which external instruction set the builtin belongs to.
1580: //===----------------------------------------------------------------------===//
1581: class ConvertBuiltin<string name, InstructionSet set> {
1582:   string Name = name;
1583:   InstructionSet Set = set;
1584:   bit IsDestinationSigned = !eq(!find(name, "convert_u"), -1);
1585:   bit IsSaturated = !not(!eq(!find(name, "_sat"), -1));
1586:   bit IsRounded = !not(!eq(!find(name, "_rt"), -1));
1587:   bit IsBfloat16 = !or(!not(!eq(!find(name, "BF16"), -1)),
1588:                        !not(!eq(!find(name, "bfloat16"), -1)));
1589:   bit IsTF32 = !or(!not(!eq(!find(name, "TF32"), -1)),
1590:                        !not(!eq(!find(name, "tensor_float32"), -1)));
1591:   FPRoundingMode RoundingMode = !cond(!not(!eq(!find(name, "_rte"), -1)) : RTE,
1592:                                   !not(!eq(!find(name, "_rtz"), -1)) : RTZ,
1593:                                   !not(!eq(!find(name, "_rtp"), -1)) : RTP,
1594:                                   !not(!eq(!find(name, "_rtn"), -1)) : RTN,
1595:                                   true : RTE);
1596: }
1597:
1598: // Table gathering all the convert builtins.
1599: def ConvertBuiltins : GenericTable {
1600:   let FilterClass = "ConvertBuiltin";
1601:   let Fields = ["Name", "Set", "IsDestinationSigned", "IsSaturated",
1602:                 "IsRounded", "IsBfloat16", "IsTF32", "RoundingMode"];
1603:   string TypeOf_Set = "InstructionSet";
1604:   string TypeOf_RoundingMode = "FPRoundingMode";
1605: }
1606:
1607: // Function to lookup convert builtins by their name and set.
1608: def lookupConvertBuiltin : SearchIndex {
1609:   let Table = ConvertBuiltins;
1610:   let Key = ["Name", "Set"];
1611: }
1612:
1613: // Multiclass used to define at the same time both a demangled builtin records
1614: // and a corresponding convert builtin records.
1615: multiclass DemangledConvertBuiltin<string name, InstructionSet set> {
1616:   // Create records for scalar and 2, 4, 8, and 16 element vector conversions.
1617:   foreach i = ["", "2", "3", "4", "8", "16"] in {
1618:     // Also create records for each rounding mode.
1619:     foreach j = ["", "_rte", "_rtz", "_rtp", "_rtn"] in {
1620:       def : DemangledBuiltin<!strconcat(name, i, j), set, Convert, 1, 1>;
```
- EN: This range uses TableGen DSL to describe records such as ImageQueryBuiltin, ImageQueryBuiltins, lookupImageQueryBuiltin, DemangledImageQueryBuiltin; LLVM later expands these declarations into generated backend tables and helper code.
- CN: 这一段使用 TableGen DSL 描述 ImageQueryBuiltin、ImageQueryBuiltins、lookupImageQueryBuiltin、DemangledImageQueryBuiltin 等记录；LLVM 后续会把这些声明展开为生成的后端表和辅助代码。

### Lines 1621-1710
```tablegen
1621:       def : ConvertBuiltin<!strconcat(name, i, j), set>;
1622:
1623:       // Create records with the "_sat" modifier for all conversions except
1624:       // those targeting floating-point types.
1625:       if !eq(!find(name, "float"), -1) then {
1626:         def : DemangledBuiltin<!strconcat(name, i, "_sat", j), set, Convert, 1, 1>;
1627:         def : ConvertBuiltin<!strconcat(name, i, "_sat", j), set>;
1628:       }
1629:     }
1630:   }
1631: }
1632:
1633: // Explicit conversion builtin records:
1634: defm : DemangledConvertBuiltin<"convert_char", OpenCL_std>;
1635: defm : DemangledConvertBuiltin<"convert_uchar", OpenCL_std>;
1636: defm : DemangledConvertBuiltin<"convert_short", OpenCL_std>;
1637: defm : DemangledConvertBuiltin<"convert_ushort", OpenCL_std>;
1638: defm : DemangledConvertBuiltin<"convert_int", OpenCL_std>;
1639: defm : DemangledConvertBuiltin<"convert_uint", OpenCL_std>;
1640: defm : DemangledConvertBuiltin<"convert_long", OpenCL_std>;
1641: defm : DemangledConvertBuiltin<"convert_ulong", OpenCL_std>;
1642: defm : DemangledConvertBuiltin<"convert_float", OpenCL_std>;
1643:
1644: defm : DemangledNativeBuiltin<"__spirv_ConvertFToU", OpenCL_std, Convert, 1, 1, OpConvertFToU>;
1645: defm : DemangledNativeBuiltin<"__spirv_ConvertFToS", OpenCL_std, Convert, 1, 1, OpConvertFToS>;
1646: defm : DemangledNativeBuiltin<"__spirv_ConvertSToF", OpenCL_std, Convert, 1, 1, OpConvertSToF>;
1647: defm : DemangledNativeBuiltin<"__spirv_ConvertUToF", OpenCL_std, Convert, 1, 1, OpConvertUToF>;
1648: defm : DemangledNativeBuiltin<"__spirv_UConvert", OpenCL_std, Convert, 1, 1, OpUConvert>;
1649: defm : DemangledNativeBuiltin<"__spirv_SConvert", OpenCL_std, Convert, 1, 1, OpSConvert>;
1650: defm : DemangledNativeBuiltin<"__spirv_FConvert", OpenCL_std, Convert, 1, 1, OpFConvert>;
1651: defm : DemangledNativeBuiltin<"__spirv_QuantizeToF16", OpenCL_std, Convert, 1, 1, OpQuantizeToF16>;
1652: defm : DemangledNativeBuiltin<"__spirv_QuantizeToF16", GLSL_std_450, Convert, 1, 1, OpQuantizeToF16>;
1653: defm : DemangledNativeBuiltin<"__spirv_ConvertPtrToU", OpenCL_std, Convert, 1, 1, OpConvertPtrToU>;
1654: defm : DemangledNativeBuiltin<"__spirv_SatConvertSToU", OpenCL_std, Convert, 1, 1, OpSatConvertSToU>;
1655: defm : DemangledNativeBuiltin<"__spirv_SatConvertUToS", OpenCL_std, Convert, 1, 1, OpSatConvertUToS>;
1656: defm : DemangledNativeBuiltin<"__spirv_ConvertUToPtr", OpenCL_std, Convert, 1, 1, OpConvertUToPtr>;
1657:
1658: // cl_khr_extended_bit_ops / SPV_KHR_bit_instructions
1659: defm : DemangledNativeBuiltin<"bitfield_insert", OpenCL_std, ExtendedBitOps, 4, 4, OpBitFieldInsert>;
1660: defm : DemangledNativeBuiltin<"__spirv_BitFieldInsert", OpenCL_std, ExtendedBitOps, 4, 4, OpBitFieldInsert>;
1661: defm : DemangledNativeBuiltin<"bitfield_extract_signed", OpenCL_std, ExtendedBitOps, 3, 3, OpBitFieldSExtract>;
1662: defm : DemangledNativeBuiltin<"__spirv_BitFieldSExtract", OpenCL_std, ExtendedBitOps, 3, 3, OpBitFieldSExtract>;
1663: defm : DemangledNativeBuiltin<"bitfield_extract_unsigned", OpenCL_std, ExtendedBitOps, 3, 3, OpBitFieldUExtract>;
1664: defm : DemangledNativeBuiltin<"__spirv_BitFieldUExtract", OpenCL_std, ExtendedBitOps, 3, 3, OpBitFieldUExtract>;
1665: defm : DemangledNativeBuiltin<"bit_reverse", OpenCL_std, ExtendedBitOps, 1, 1, OpBitReverse>;
1666: defm : DemangledNativeBuiltin<"__spirv_BitReverse", OpenCL_std, ExtendedBitOps, 1, 1, OpBitReverse>;
1667:
1668: // cl_intel_bfloat16_conversions / SPV_INTEL_bfloat16_conversion
1669: // Multiclass used to define at the same time both a demangled builtin records
1670: // and a corresponding convert builtin records.
1671: multiclass DemangledBF16ConvertBuiltin<string name1, string name2> {
1672:   // Create records for scalar and vector conversions.
1673:   foreach i = ["", "2", "3", "4", "8", "16"] in {
1674:     def : DemangledBuiltin<!strconcat("intel_convert_", name1, i, name2, i), OpenCL_std, Convert, 1, 1>;
1675:     def : ConvertBuiltin<!strconcat("intel_convert_", name1, i, name2, i), OpenCL_std>;
1676:   }
1677: }
1678:
1679: defm : DemangledBF16ConvertBuiltin<"bfloat16", "_as_ushort">;
1680: defm : DemangledBF16ConvertBuiltin<"as_bfloat16", "_float">;
1681:
1682: foreach conv = ["FToBF16INTEL", "BF16ToFINTEL"] in {
1683:   def : DemangledBuiltin<!strconcat("__spirv_Convert", conv), OpenCL_std, Convert, 1, 1>;
1684:   def : ConvertBuiltin<!strconcat("__spirv_Convert", conv), OpenCL_std>;
1685: }
1686:
1687: // cl_intel_tensor_float32_conversions / SPV_INTEL_tensor_float32_conversion
1688: // Multiclass used to define at the same time both a demangled builtin record
1689: // and a corresponding convert builtin record.
1690: multiclass DemangledTF32RoundBuiltin<string name1, string name2> {
1691:   // Create records for scalar and vector conversions.
1692:   foreach i = ["", "2", "3", "4", "8", "16"] in {
1693:     def : DemangledBuiltin<!strconcat("intel_round_", name1, i, name2, i), OpenCL_std, Convert, 1, 1>;
1694:     def : ConvertBuiltin<!strconcat("intel_round_", name1, i, name2, i), OpenCL_std>;
1695:   }
1696: }
1697:
1698: defm : DemangledTF32RoundBuiltin<"tensor_float32", "_as_float">;
1699: defm : DemangledTF32RoundBuiltin<"as_tensor_float32", "_float">;
1700:
1701: foreach conv = ["FToTF32INTEL"] in {
1702:   def : DemangledBuiltin<!strconcat("__spirv_Round", conv), OpenCL_std, Convert, 1, 1>;
1703:   def : ConvertBuiltin<!strconcat("__spirv_Round", conv), OpenCL_std>;
1704: }
1705:
1706: //===----------------------------------------------------------------------===//
1707: // Class defining a vector data load/store builtin record used for lowering
1708: // into OpExtInst instruction.
1709: //
1710: // name is the demangled name of the given builtin.
```
- EN: This range uses TableGen DSL to describe records such as DemangledBF16ConvertBuiltin, DemangledTF32RoundBuiltin; LLVM later expands these declarations into generated backend tables and helper code.
- CN: 这一段使用 TableGen DSL 描述 DemangledBF16ConvertBuiltin、DemangledTF32RoundBuiltin 等记录；LLVM 后续会把这些声明展开为生成的后端表和辅助代码。

### Lines 1711-1800
```tablegen
1711: // set specifies which external instruction set the builtin belongs to.
1712: // number specifies the number of the instruction in the external set.
1713: //===----------------------------------------------------------------------===//
1714: class VectorLoadStoreBuiltin<string name, InstructionSet set, int number> {
1715:   string Name = name;
1716:   InstructionSet Set = set;
1717:   bits<32> Number = number;
1718:   bits<32> ElementCount = !cond(!not(!eq(!find(name, "2"), -1)) : 2,
1719:                                 !not(!eq(!find(name, "3"), -1)) : 3,
1720:                                 !not(!eq(!find(name, "4"), -1)) : 4,
1721:                                 !not(!eq(!find(name, "8"), -1)) : 8,
1722:                                 !not(!eq(!find(name, "16"), -1)) : 16,
1723:                                 true : 1);
1724:   bit IsRounded = !not(!eq(!find(name, "_rt"), -1));
1725:   FPRoundingMode RoundingMode = !cond(!not(!eq(!find(name, "_rte"), -1)) : RTE,
1726:                                       !not(!eq(!find(name, "_rtz"), -1)) : RTZ,
1727:                                       !not(!eq(!find(name, "_rtp"), -1)) : RTP,
1728:                                       !not(!eq(!find(name, "_rtn"), -1)) : RTN,
1729:                                       true : RTE);
1730: }
1731:
1732: // Table gathering all the vector data load/store builtins.
1733: def VectorLoadStoreBuiltins : GenericTable {
1734:   let FilterClass = "VectorLoadStoreBuiltin";
1735:   let Fields = ["Name", "Set", "Number", "ElementCount", "IsRounded", "RoundingMode"];
1736:   string TypeOf_Set = "InstructionSet";
1737:   string TypeOf_RoundingMode = "FPRoundingMode";
1738: }
1739:
1740: // Function to lookup vector data load/store builtins by their name and set.
1741: def lookupVectorLoadStoreBuiltin : SearchIndex {
1742:   let Table = VectorLoadStoreBuiltins;
1743:   let Key = ["Name", "Set"];
1744: }
1745:
1746: // Multiclass used to define at the same time both a demangled builtin record
1747: // and a corresponding vector data load/store builtin record.
1748: multiclass DemangledVectorLoadStoreBuiltin<string name, bits<8> minNumArgs, bits<8> maxNumArgs, int number> {
1749:   def : DemangledBuiltin<name, OpenCL_std, VectorLoadStore, minNumArgs, maxNumArgs>;
1750:   def : VectorLoadStoreBuiltin<name, OpenCL_std, number>;
1751: }
1752:
1753: // Create records for scalar and 2, 4, 8, and 16 vector element count.
1754: foreach i = ["", "2", "3", "4", "8", "16"] in {
1755:   if !eq(i, "") then {
1756:     defm : DemangledVectorLoadStoreBuiltin<"vload_half", 2, 2, 173>;
1757:     defm : DemangledVectorLoadStoreBuiltin<"vstore_half", 3, 3, 175>;
1758:   } else {
1759:     defm : DemangledVectorLoadStoreBuiltin<!strconcat("vload_half", i), 2, 2, 174>;
1760:     defm : DemangledVectorLoadStoreBuiltin<!strconcat("vstore_half", i), 3, 3, 177>;
1761:   }
1762:   defm : DemangledVectorLoadStoreBuiltin<!strconcat("vload", i), 2, 2, 171>;
1763:   defm : DemangledVectorLoadStoreBuiltin<!strconcat("vstore", i), 3, 3, 172>;
1764:   defm : DemangledVectorLoadStoreBuiltin<!strconcat("vloada_half", i), 2, 2, 174>;
1765:   defm : DemangledVectorLoadStoreBuiltin<!strconcat("vstorea_half", i), 3, 3, 180>;
1766:
1767:   // Also create records for each rounding mode.
1768:   foreach j = ["_rte", "_rtz", "_rtp", "_rtn"] in {
1769:     if !eq(i, "") then {
1770:       defm : DemangledVectorLoadStoreBuiltin<!strconcat("vstore_half", j), 3, 3, 176>;
1771:     } else {
1772:       defm : DemangledVectorLoadStoreBuiltin<!strconcat("vstore_half", i, j), 3, 3, 178>;
1773:     }
1774:     defm : DemangledVectorLoadStoreBuiltin<!strconcat("vstorea_half", i, j), 3, 3, 181>;
1775:   }
1776: }
1777:
1778: //===----------------------------------------------------------------------===//
1779: // Class defining implementation details of SPIR-V builtin types. The info
1780: // in the record is used for lowering into OpType.
1781: //
1782: // name is the name of the given SPIR-V builtin type.
1783: // operation specifies the SPIR-V opcode the StructType should be lowered to.
1784: //===----------------------------------------------------------------------===//
1785: class BuiltinType<string name, Op operation> {
1786:   string Name = name;
1787:   Op Opcode = operation;
1788: }
1789:
1790: // Table gathering all the builtin type records.
1791: def BuiltinTypes : GenericTable {
1792:   let FilterClass = "BuiltinType";
1793:   let Fields = ["Name", "Opcode"];
1794: }
1795:
1796: // Function to lookup builtin types by their demangled name.
1797: def lookupBuiltinType : SearchIndex {
1798:   let Table = BuiltinTypes;
1799:   let Key = ["Name"];
1800: }
```
- EN: This range uses TableGen DSL to describe records such as VectorLoadStoreBuiltin, VectorLoadStoreBuiltins, lookupVectorLoadStoreBuiltin, DemangledVectorLoadStoreBuiltin; LLVM later expands these declarations into generated backend tables and helper code.
- CN: 这一段使用 TableGen DSL 描述 VectorLoadStoreBuiltin、VectorLoadStoreBuiltins、lookupVectorLoadStoreBuiltin、DemangledVectorLoadStoreBuiltin 等记录；LLVM 后续会把这些声明展开为生成的后端表和辅助代码。

### Lines 1801-1890
```tablegen
1801:
1802: def : BuiltinType<"spirv.ReserveId", OpTypeReserveId>;
1803: def : BuiltinType<"spirv.PipeStorage", OpTypePipeStorage>;
1804: def : BuiltinType<"spirv.Queue", OpTypeQueue>;
1805: def : BuiltinType<"spirv.Event", OpTypeEvent>;
1806: def : BuiltinType<"spirv.Sampler", OpTypeSampler>;
1807: def : BuiltinType<"spirv.DeviceEvent", OpTypeDeviceEvent>;
1808: def : BuiltinType<"spirv.Image", OpTypeImage>;
1809: def : BuiltinType<"spirv.SignedImage", OpTypeImage>;
1810: def : BuiltinType<"spirv.SampledImage", OpTypeSampledImage>;
1811: def : BuiltinType<"spirv.Pipe", OpTypePipe>;
1812: def : BuiltinType<"spirv.CooperativeMatrixKHR", OpTypeCooperativeMatrixKHR>;
1813:
1814: //===----------------------------------------------------------------------===//
1815: // Class matching an OpenCL builtin type name to an equivalent SPIR-V
1816: // builtin type literal.
1817: //
1818: // name is the name of the given OpenCL builtin type.
1819: // spirvTypeLiteral is the literal of an equivalent SPIR-V builtin type.
1820: //===----------------------------------------------------------------------===//
1821: class OpenCLType<string name, string spirvTypeLiteral> {
1822:   string Name = name;
1823:   string SpirvTypeLiteral = spirvTypeLiteral;
1824: }
1825:
1826: // Table gathering all the OpenCL type records.
1827: def OpenCLTypes : GenericTable {
1828:   let FilterClass = "OpenCLType";
1829:   let Fields = ["Name", "SpirvTypeLiteral"];
1830: }
1831:
1832: // Function to lookup OpenCL types by their name.
1833: def lookupOpenCLType : SearchIndex {
1834:   let Table = OpenCLTypes;
1835:   let Key = ["Name"];
1836: }
1837:
1838: def : OpenCLType<"opencl.reserve_id_t", "spirv.ReserveId">;
1839: def : OpenCLType<"opencl.event_t", "spirv.Event">;
1840: def : OpenCLType<"opencl.queue_t", "spirv.Queue">;
1841: def : OpenCLType<"opencl.sampler_t", "spirv.Sampler">;
1842: def : OpenCLType<"opencl.clk_event_t", "spirv.DeviceEvent">;
1843:
1844: foreach aq = ["_t", "_ro_t", "_wo_t", "_rw_t"] in {
1845:   defvar p = !cond(!not(!eq(!find(aq, "_rw_t"), -1)) : "2",
1846:                    !not(!eq(!find(aq, "_wo_t"), -1)) : "1",
1847:                                                 true : "0");
1848:   def : OpenCLType<!strconcat("opencl.pipe", aq), 
1849:                    !strconcat("spirv.Pipe._", p)>;
1850: }
1851:
1852: foreach aq = ["_t", "_ro_t", "_wo_t", "_rw_t"] in {
1853:   defvar p7 = !cond(!not(!eq(!find(aq, "_rw_t"), -1)) : "2",
1854:                     !not(!eq(!find(aq, "_wo_t"), -1)) : "1",
1855:                                                  true : "0");
1856:
1857:   def : OpenCLType<!strconcat("opencl.image1d", aq), 
1858:                    !strconcat("spirv.Image._void_0_0_0_0_0_0_", p7)>;
1859:   def : OpenCLType<!strconcat("opencl.image1d_array", aq), 
1860:                    !strconcat("spirv.Image._void_0_0_1_0_0_0_", p7)>;
1861:   def : OpenCLType<!strconcat("opencl.image1d_buffer", aq), 
1862:                    !strconcat("spirv.Image._void_5_0_0_0_0_0_", p7)>;
1863:
1864:   foreach a1 = ["", "_array"] in {
1865:     foreach a2 = ["", "_msaa"] in {
1866:       foreach a3 = ["", "_depth"] in {
1867:         defvar p2 = !cond(!not(!eq(!find(a3, "_depth"), -1)) : "1", true : "0");
1868:         defvar p3 = !cond(!not(!eq(!find(a1, "_array"), -1))  : "1", true : "0");
1869:         defvar p4 = !cond(!not(!eq(!find(a2, "msaa"), -1))  : "1", true : "0");
1870:
1871:         def : OpenCLType<!strconcat("opencl.image2d", a1, a2, a3, aq), 
1872:                          !strconcat("spirv.Image._void_1_", p2 , "_", p3, "_", p4, "_0_0_", p7)>;
1873:       }
1874:     }
1875:   }
1876:   
1877:   def : OpenCLType<!strconcat("opencl.image3d", aq), 
1878:                    !strconcat("spirv.Image._void_2_0_0_0_0_0_", p7)>;
1879: }
1880:
1881: //===----------------------------------------------------------------------===//
1882: // Classes definining various OpenCL enums.
1883: //===----------------------------------------------------------------------===//
1884:
1885: // OpenCL memory_scope enum
1886: def CLMemoryScope : GenericEnum {
1887:   let FilterClass = "CLMemoryScope";
1888:   let NameField = "Name";
1889:   let ValueField = "Value";
1890: }
```
- EN: This range uses TableGen DSL to describe records such as OpenCLType, OpenCLTypes, lookupOpenCLType, CLMemoryScope; LLVM later expands these declarations into generated backend tables and helper code.
- CN: 这一段使用 TableGen DSL 描述 OpenCLType、OpenCLTypes、lookupOpenCLType、CLMemoryScope 等记录；LLVM 后续会把这些声明展开为生成的后端表和辅助代码。

### Lines 1891-1980
```tablegen
1891:
1892: class CLMemoryScope<bits<32> value> {
1893:   string Name = NAME;
1894:   bits<32> Value = value;
1895: }
1896:
1897: def memory_scope_work_item : CLMemoryScope<0>;
1898: def memory_scope_work_group : CLMemoryScope<1>;
1899: def memory_scope_device : CLMemoryScope<2>;
1900: def memory_scope_all_svm_devices : CLMemoryScope<3>;
1901: def memory_scope_sub_group : CLMemoryScope<4>;
1902:
1903: // OpenCL sampler addressing mode/bitmask enum
1904: def CLSamplerAddressingMode : GenericEnum {
1905:   let FilterClass = "CLSamplerAddressingMode";
1906:   let NameField = "Name";
1907:   let ValueField = "Value";
1908: }
1909:
1910: class CLSamplerAddressingMode<bits<32> value> {
1911:   string Name = NAME;
1912:   bits<32> Value = value;
1913: }
1914:
1915: def CLK_ADDRESS_NONE : CLSamplerAddressingMode<0x0>;
1916: def CLK_ADDRESS_CLAMP : CLSamplerAddressingMode<0x4>;
1917: def CLK_ADDRESS_CLAMP_TO_EDGE : CLSamplerAddressingMode<0x2>;
1918: def CLK_ADDRESS_REPEAT : CLSamplerAddressingMode<0x6>;
1919: def CLK_ADDRESS_MIRRORED_REPEAT : CLSamplerAddressingMode<0x8>;
1920: def CLK_ADDRESS_MODE_MASK : CLSamplerAddressingMode<0xE>;
1921: def CLK_NORMALIZED_COORDS_FALSE : CLSamplerAddressingMode<0x0>;
1922: def CLK_NORMALIZED_COORDS_TRUE : CLSamplerAddressingMode<0x1>;
1923: def CLK_FILTER_NEAREST : CLSamplerAddressingMode<0x10>;
1924: def CLK_FILTER_LINEAR : CLSamplerAddressingMode<0x20>;
1925:
1926: // OpenCL memory fences
1927: def CLMemoryFenceFlags : GenericEnum {
1928:   let FilterClass = "CLMemoryFenceFlags";
1929:   let NameField = "Name";
1930:   let ValueField = "Value";
1931: }
1932:
1933: class CLMemoryFenceFlags<bits<32> value> {
1934:   string Name = NAME;
1935:   bits<32> Value = value;
1936: }
1937:
1938: def CLK_LOCAL_MEM_FENCE : CLMemoryFenceFlags<0x1>;
1939: def CLK_GLOBAL_MEM_FENCE : CLMemoryFenceFlags<0x2>;
1940: def CLK_IMAGE_MEM_FENCE : CLMemoryFenceFlags<0x4>;
1941:
1942: //===----------------------------------------------------------------------===//
1943: // Class defining dot builtins that should be translated into a
1944: // SPIR-V instruction using SPIR-V 1.6 or SPV_KHR_integer_dot_product extension.
1945: //
1946: // name is the demangled name of the given builtin.
1947: // opcode specifies the SPIR-V operation code of the generated instruction.
1948: // isSwapRequired specifies if the operands need to be swapped (the SPIR-V extension
1949: // has only one instruction for arguments of different signedness).
1950: //===----------------------------------------------------------------------===//
1951: class IntegerDotProductBuiltin<string name, Op operation> {
1952:   string Name = name;
1953:   Op Opcode = operation;
1954:   bit IsSwapReq = !not(!eq(!find(name, "_us"), -1));
1955: }
1956:
1957: // Table gathering all the integer dot product builtins.
1958: def IntegerDotProductBuiltins : GenericTable {
1959:   let FilterClass = "IntegerDotProductBuiltin";
1960:   let Fields = ["Name", "Opcode", "IsSwapReq"];
1961: }
1962:
1963: // Function to lookup group builtins by their name and set.
1964: def lookupIntegerDotProductBuiltin : SearchIndex {
1965:   let Table = IntegerDotProductBuiltins;
1966:   let Key = ["Name"];
1967: }
1968:
1969: // Multiclass used to define incoming builtin records for the SPV_KHR_integer_dot_product extension.
1970: multiclass DemangledIntegerDotProductBuiltin<string name, bits<8> minNumArgs, bits<8> maxNumArgs, Op operation> {
1971:   def : DemangledBuiltin<!strconcat("dot", name), OpenCL_std, IntegerDot, minNumArgs, maxNumArgs>;
1972:   def : IntegerDotProductBuiltin<!strconcat("dot", name), operation>;
1973: }
1974:
1975: // cl_khr_integer_dot_product
1976: defm : DemangledIntegerDotProductBuiltin<"_4x8packed_uu_uint", 2, 3, OpUDot>;
1977: defm : DemangledIntegerDotProductBuiltin<"_4x8packed_ss_int", 2, 3, OpSDot>;
1978: defm : DemangledIntegerDotProductBuiltin<"_4x8packed_us_int", 2, 3, OpSUDot>;
1979: defm : DemangledIntegerDotProductBuiltin<"_4x8packed_su_int", 2, 3, OpSUDot>;
1980:
```
- EN: This range uses TableGen DSL to describe records such as CLMemoryScope, memory_scope_work_item, memory_scope_work_group, memory_scope_device; LLVM later expands these declarations into generated backend tables and helper code.
- CN: 这一段使用 TableGen DSL 描述 CLMemoryScope、memory_scope_work_item、memory_scope_work_group、memory_scope_device 等记录；LLVM 后续会把这些声明展开为生成的后端表和辅助代码。

### Lines 1981-1984
```tablegen
1981: defm : DemangledIntegerDotProductBuiltin<"_acc_sat_4x8packed_uu_uint", 3, 4, OpUDotAccSat>;
1982: defm : DemangledIntegerDotProductBuiltin<"_acc_sat_4x8packed_ss_int", 3, 4, OpSDotAccSat>;
1983: defm : DemangledIntegerDotProductBuiltin<"_acc_sat_4x8packed_us_int", 3, 4, OpSUDotAccSat>;
1984: defm : DemangledIntegerDotProductBuiltin<"_acc_sat_4x8packed_su_int", 3, 4, OpSUDotAccSat>;
```
- EN: This range uses TableGen DSL to describe records such as backend logic; LLVM later expands these declarations into generated backend tables and helper code.
- CN: 这一段使用 TableGen DSL 描述 后端逻辑 等记录；LLVM 后续会把这些声明展开为生成的后端表和辅助代码。

## Key Concepts / 关键概念

- EN: TableGen files use declarative records that LLVM expands into generated C++ tables and helper code.
  - CN: TableGen 文件使用声明式记录，LLVM 会将其展开为生成的 C++ 表和辅助代码。
- EN: SPIR-V backend code must preserve rich type information and encode capabilities/extensions explicitly.
  - CN: SPIR-V 后端需要保留丰富的类型信息，并显式编码 capability 与 extension。
- EN: Key symbols in this file include InstructionSet, OpenCL_std, GLSL_std_450, SPV_AMD_shader_trinary_minmax, NonSemantic_Shader_DebugInfo_100, which anchor the file's main abstractions.
  - CN: 该文件中的关键符号包括 InstructionSet, OpenCL_std, GLSL_std_450, SPV_AMD_shader_trinary_minmax, NonSemantic_Shader_DebugInfo_100，它们构成了本文件的核心抽象。

## Dependencies / 依赖关系

- TableGen pipeline / TableGen 流水线: records in this file are consumed by LLVM TableGen emitters to produce generated lookup tables and helper code.
- TableGen 流水线说明：该文件中的记录会被 LLVM 的 TableGen 生成器消费，进而产出查找表和辅助代码。
- This file has no explicit textual includes; its effective dependencies come from surrounding generated or linked LLVM components.
- 该文件没有显式文本 include；其实际依赖主要来自周边生成代码或链接到的 LLVM 组件。
