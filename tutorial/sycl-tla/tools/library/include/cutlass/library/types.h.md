# types.h — Code Analysis / 代码分析

## Source / 来源
- **Path / 路径:** `tools/library/include/cutlass/library/types.h`
- **Purpose (EN):** This file declares library metadata for the CUTLASS library metadata layer.
- **目的 (CN):** 该文件声明了面向CUTLASS 库元数据层的库元数据逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-20
```cpp
1: /***************************************************************************************************
2:  * Copyright (c) 2023 - 2025 NVIDIA CORPORATION & AFFILIATES. All rights reserved.
3:  * SPDX-License-Identifier: BSD-3-Clause
4:  *
5:  * Redistribution and use in source and binary forms, with or without
6:  * modification, are permitted provided that the following conditions are met:
7:  *
8:  * 1. Redistributions of source code must retain the above copyright notice, this
9:  * list of conditions and the following disclaimer.
10:  *
11:  * 2. Redistributions in binary form must reproduce the above copyright notice,
12:  * this list of conditions and the following disclaimer in the documentation
13:  * and/or other materials provided with the distribution.
14:  *
15:  * 3. Neither the name of the copyright holder nor the names of its
16:  * contributors may be used to endorse or promote products derived from
17:  * this software without specific prior written permission.
18:  *
19:  * THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS"
20:  * AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE
```
- **EN:** License header and copyright terms for the file.
- **CN:** 文件的许可证头和版权条款。

### Lines 21-30
```cpp
21:  * IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE ARE
22:  * DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT HOLDER OR CONTRIBUTORS BE LIABLE
23:  * FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL
24:  * DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR
25:  * SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER
26:  * CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY,
27:  * OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE
28:  * OF THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.
29:  *
30:  **************************************************************************************************/
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 32-32
```cpp
32:  #pragma once
```
- **EN:** Uses include guards or pragmas to ensure the declarations are processed only once.
- **CN:** 使用 include guard 或 pragma，确保这些声明只被处理一次。

### Lines 34-34
```cpp
34:  /////////////////////////////////////////////////////////////////////////////////////////////////
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 36-37
```cpp
36: namespace cutlass {
37: namespace library {
```
- **EN:** Supporting logic for the library metadata implementation.
- **CN:** 库元数据实现的辅助逻辑。

### Lines 39-39
```cpp
39: /////////////////////////////////////////////////////////////////////////////////////////////////
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 41-41
```cpp
41: /// Layout type identifier
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 42-55
```cpp
42: enum class LayoutTypeID {
43:   kUnknown,
44:   kColumnMajor,
45:   kRowMajor,
46:   kBlockScalingTensor,          
47:   kColumnMajorInterleavedK2,
48:   kRowMajorInterleavedK2,
49:   kColumnMajorInterleavedK4,
50:   kRowMajorInterleavedK4,
51:   kColumnMajorInterleavedK16,
52:   kRowMajorInterleavedK16,
53:   kColumnMajorInterleavedK32,
54:   kRowMajorInterleavedK32,
55:   kColumnMajorInterleavedK64,
```
- **EN:** Defines `LayoutTypeID` for layout identifiers for matrix/tensor storage; representative values include `kUnknown`, `kColumnMajor`, `kRowMajor`, `kBlockScalingTensor`, `kColumnMajorInterleavedK2`.
- **CN:** 定义 `LayoutTypeID` 来表示矩阵/张量存储布局标识；代表性取值包括 `kUnknown`, `kColumnMajor`, `kRowMajor`, `kBlockScalingTensor`, `kColumnMajorInterleavedK2`。

### Lines 56-66
```cpp
56:   kRowMajorInterleavedK64,
57:   kTensorNCHW,
58:   kTensorNCDHW,
59:   kTensorNHWC,
60:   kTensorNDHWC,
61:   kTensorNC32HW32,
62:   kTensorC32RSK32,
63:   kTensorNC64HW64,
64:   kTensorC64RSK64,
65:   kInvalid
66: };
```
- **EN:** Supporting logic for the library metadata implementation.
- **CN:** 库元数据实现的辅助逻辑。

### Lines 68-68
```cpp
68: /// Numeric data type
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 69-82
```cpp
69: enum class NumericTypeID {
70:   kUnknown,
71:   kVoid,
72:   kB1,
73:   kU2,
74:   kU4,
75:   kU8,
76:   kU16,
77:   kU32,
78:   kU64,
79:   kS2,
80:   kS4,
81:   kS8,
82:   kS16,
```
- **EN:** Defines `NumericTypeID` for numeric element type identifiers; representative values include `kUnknown`, `kVoid`, `kB1`, `kU2`, `kU4`.
- **CN:** 定义 `NumericTypeID` 来表示数值元素类型标识；代表性取值包括 `kUnknown`, `kVoid`, `kB1`, `kU2`, `kU4`。

### Lines 83-86
```cpp
83:   kS32,
84:   kS64,
85:   kFE4M3,
86:   kFE5M2,
```
- **EN:** Supporting logic for the library metadata implementation.
- **CN:** 库元数据实现的辅助逻辑。

### Lines 88-95
```cpp
88:   kFE2M3,
89:   kFE3M2,
90:   kFE2M1,
91:   kFUE8M0, 
92:   kFUE4M3, 
93:   kF8,
94:   kF6,
95:   kF4,
```
- **EN:** Supporting logic for the library metadata implementation.
- **CN:** 库元数据实现的辅助逻辑。

### Lines 97-110
```cpp
97:   kF16,
98:   kBF16, 
99:   kTF32,
100:   kF32,
101:   kF64,
102:   kCF16,
103:   kCBF16,
104:   kCF32,
105:   kCTF32,
106:   kCF64,
107:   kCS2,
108:   kCS4,
109:   kCS8,
110:   kCS16,
```
- **EN:** Supporting logic for the library metadata implementation.
- **CN:** 库元数据实现的辅助逻辑。

### Lines 111-120
```cpp
111:   kCS32,
112:   kCS64,
113:   kCU2,
114:   kCU4,
115:   kCU8,
116:   kCU16,
117:   kCU32,
118:   kCU64,
119:   kInvalid
120: };
```
- **EN:** Supporting logic for the library metadata implementation.
- **CN:** 库元数据实现的辅助逻辑。

### Lines 122-122
```cpp
122: /// Enumerated type describing a transformation on a complex value.
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 123-127
```cpp
123: enum class ComplexTransform {
124:   kNone,
125:   kConjugate,
126:   kInvalid
127: };
```
- **EN:** Defines `ComplexTransform` for complex-number transform modes; representative values include `kNone`, `kConjugate`, `kInvalid`.
- **CN:** 定义 `ComplexTransform` 来表示复数变换模式；代表性取值包括 `kNone`, `kConjugate`, `kInvalid`。

### Lines 129-129
```cpp
129: /// Providers
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 130-138
```cpp
130: enum class Provider {
131:   kNone,
132:   kCUTLASS,
133:   kReferenceHost,
134:   kReferenceDevice,
135:   kCUBLAS,
136:   kCUDNN,
137:   kInvalid
138: };
```
- **EN:** Defines `Provider` for backend/provider identifiers; representative values include `kNone`, `kCUTLASS`, `kReferenceHost`, `kReferenceDevice`, `kCUBLAS`.
- **CN:** 定义 `Provider` 来表示后端/提供者标识；代表性取值包括 `kNone`, `kCUTLASS`, `kReferenceHost`, `kReferenceDevice`, `kCUBLAS`。

### Lines 140-140
```cpp
140: /////////////////////////////////////////////////////////////////////////////////////////////////
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 142-142
```cpp
142: /// Enumeration indicating the kind of operation
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 143-156
```cpp
143: enum class OperationKind {
144:   kGemm,
145:   kBlockScaledGemm,
146:   kBlockwiseGemm,
147:   kRankK,
148:   kRank2K,
149:   kTrmm,
150:   kSymm,
151:   kConv2d,
152:   kConv3d,
153:   kEqGemm,
154:   kSparseGemm,
155:   kReduction,
156:   kGroupedGemm,
```
- **EN:** Defines `OperationKind` for operation categories handled by the library; representative values include `kGemm`, `kBlockScaledGemm`, `kBlockwiseGemm`, `kRankK`, `kRank2K`.
- **CN:** 定义 `OperationKind` 来表示库支持的操作类别；代表性取值包括 `kGemm`, `kBlockScaledGemm`, `kBlockwiseGemm`, `kRankK`, `kRank2K`。

### Lines 157-158
```cpp
157:   kInvalid
158: };
```
- **EN:** Supporting logic for the library metadata implementation.
- **CN:** 库元数据实现的辅助逻辑。

### Lines 160-160
```cpp
160: /// Enumeration indicating whether scalars are in host or device memory
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 161-165
```cpp
161: enum class ScalarPointerMode {
162:   kHost,
163:   kDevice,
164:   kInvalid
165: };
```
- **EN:** Defines `ScalarPointerMode` for locations of scalar parameters; representative values include `kHost`, `kDevice`, `kInvalid`.
- **CN:** 定义 `ScalarPointerMode` 来表示标量参数所在位置；代表性取值包括 `kHost`, `kDevice`, `kInvalid`。

### Lines 167-167
```cpp
167: /// Describes how reductions are performed across threadblocks
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 168-174
```cpp
168: enum class SplitKMode {
169:   kNone,
170:   kSerial,
171:   kParallel,
172:   kParallelSerial,
173:   kInvalid
174: };
```
- **EN:** Defines `SplitKMode` for split-K reduction strategies; representative values include `kNone`, `kSerial`, `kParallel`, `kParallelSerial`, `kInvalid`.
- **CN:** 定义 `SplitKMode` 来表示Split-K 归约策略；代表性取值包括 `kNone`, `kSerial`, `kParallel`, `kParallelSerial`, `kInvalid`。

### Lines 176-176
```cpp
176: /// Indicates the classificaition of the math instruction
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 177-184
```cpp
177: enum class OpcodeClassID {
178:   kSimt,
179:   kTensorOp,
180:   kWmmaTensorOp,
181:   kSparseTensorOp,
182:   kBlockScaledOp,                
183:   kInvalid
184: };
```
- **EN:** Defines `OpcodeClassID` for instruction-class identifiers; representative values include `kSimt`, `kTensorOp`, `kWmmaTensorOp`, `kSparseTensorOp`, `kBlockScaledOp`.
- **CN:** 定义 `OpcodeClassID` 来表示指令类别标识；代表性取值包括 `kSimt`, `kTensorOp`, `kWmmaTensorOp`, `kSparseTensorOp`, `kBlockScaledOp`。

### Lines 186-199
```cpp
186: enum class MathOperationID {
187:   kAdd,
188:   kMultiplyAdd,
189:   kMultiplyAddSaturate,
190:   kMultiplyAddMixedInputUpcast,
191:   kMultiplyAddFastBF16,
192:   kMultiplyAddFastF16,
193:   kMultiplyAddFastF32,
194:   kMultiplyAddComplex,
195:   kMultiplyAddComplexFastF32,
196:   kMultiplyAddGaussianComplex,
197:   kXorPopc,
198:   kInvalid
199: };
```
- **EN:** Defines `MathOperationID` for math primitive identifiers; representative values include `kAdd`, `kMultiplyAdd`, `kMultiplyAddSaturate`, `kMultiplyAddMixedInputUpcast`, `kMultiplyAddFastBF16`.
- **CN:** 定义 `MathOperationID` 来表示数学原语标识；代表性取值包括 `kAdd`, `kMultiplyAdd`, `kMultiplyAddSaturate`, `kMultiplyAddMixedInputUpcast`, `kMultiplyAddFastBF16`。

### Lines 201-201
```cpp
201: /////////////////////////////////////////////////////////////////////////////////////////////////
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 203-203
```cpp
203: /// Enumeration indicating what kind of GEMM operation to perform
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 204-213
```cpp
204: enum class GemmKind {
205:   kGemm,
206:   kBlockScaledGemm,                
207:   kSparse,
208:   kUniversal,
209:   kPlanarComplex,
210:   kPlanarComplexArray,
211:   kGrouped,
212:   kInvalid
213: };
```
- **EN:** Defines `GemmKind` for GEMM variants; representative values include `kGemm`, `kBlockScaledGemm`, `kSparse`, `kUniversal`, `kPlanarComplex`.
- **CN:** 定义 `GemmKind` 来表示GEMM 变体；代表性取值包括 `kGemm`, `kBlockScaledGemm`, `kSparse`, `kUniversal`, `kPlanarComplex`。

### Lines 215-215
```cpp
215: /// Enumeration indicating what kind of RankK update operation to perform
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 216-219
```cpp
216: enum class RankKKind {
217:   kUniversal,
218:   kInvalid
219: };
```
- **EN:** Defines `RankKKind` for the `RankKKind` value set used by this component; representative values include `kUniversal`, `kInvalid`.
- **CN:** 定义 `RankKKind` 来表示该组件使用的 `RankKKind` 取值集合；代表性取值包括 `kUniversal`, `kInvalid`。

### Lines 221-221
```cpp
221: /// Enumeration indicating what kind of TRMM operation to perform
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 222-225
```cpp
222: enum class TrmmKind {
223:   kUniversal,
224:   kInvalid
225: };
```
- **EN:** Defines `TrmmKind` for the `TrmmKind` value set used by this component; representative values include `kUniversal`, `kInvalid`.
- **CN:** 定义 `TrmmKind` 来表示该组件使用的 `TrmmKind` 取值集合；代表性取值包括 `kUniversal`, `kInvalid`。

### Lines 227-227
```cpp
227: /// Enumeration indicating what kind of SYMM/HEMM operation to perform
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 228-231
```cpp
228: enum class SymmKind {
229:   kUniversal,
230:   kInvalid
231: };
```
- **EN:** Defines `SymmKind` for the `SymmKind` value set used by this component; representative values include `kUniversal`, `kInvalid`.
- **CN:** 定义 `SymmKind` 来表示该组件使用的 `SymmKind` 取值集合；代表性取值包括 `kUniversal`, `kInvalid`。

### Lines 233-233
```cpp
233: /// Enumeration indicating what kind of Conv2d operation to perform
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 234-240
```cpp
234: enum class ConvKind {
235:   kUnknown,
236:   kFprop,
237:   kDgrad,
238:   kWgrad,
239:   kInvalid
240: };
```
- **EN:** Defines `ConvKind` for convolution directions; representative values include `kUnknown`, `kFprop`, `kDgrad`, `kWgrad`, `kInvalid`.
- **CN:** 定义 `ConvKind` 来表示卷积方向；代表性取值包括 `kUnknown`, `kFprop`, `kDgrad`, `kWgrad`, `kInvalid`。

### Lines 242-246
```cpp
242: enum class ConvModeID {
243:   kCrossCorrelation,
244:   kConvolution,
245:   kInvalid
246: };
```
- **EN:** Defines `ConvModeID` for the `ConvModeID` value set used by this component; representative values include `kCrossCorrelation`, `kConvolution`, `kInvalid`.
- **CN:** 定义 `ConvModeID` 来表示该组件使用的 `ConvModeID` 取值集合；代表性取值包括 `kCrossCorrelation`, `kConvolution`, `kInvalid`。

### Lines 248-248
```cpp
248: // Iterator algorithm enum in order of general performance-efficiency
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 249-256
```cpp
249: enum class IteratorAlgorithmID {
250:   kNone,
251:   kAnalytic,
252:   kOptimized,
253:   kFixedChannels,
254:   kFewChannels,
255:   kInvalid
256: };
```
- **EN:** Defines `IteratorAlgorithmID` for the `IteratorAlgorithmID` value set used by this component; representative values include `kNone`, `kAnalytic`, `kOptimized`, `kFixedChannels`, `kFewChannels`.
- **CN:** 定义 `IteratorAlgorithmID` 来表示该组件使用的 `IteratorAlgorithmID` 取值集合；代表性取值包括 `kNone`, `kAnalytic`, `kOptimized`, `kFixedChannels`, `kFewChannels`。

### Lines 259-268
```cpp
259: enum class EpilogueKind {
260:   kUnknown,
261:   kConversion,
262:   kLinearCombination,
263:   kLinearCombinationClamp,
264:   kLinearCombinationPlanarComplex,
265:   kLinearCombinationRelu,
266:   kLinearCombinationSigmoid,
267:   kInvalid
268: };
```
- **EN:** Defines `EpilogueKind` for the `EpilogueKind` value set used by this component; representative values include `kUnknown`, `kConversion`, `kLinearCombination`, `kLinearCombinationClamp`, `kLinearCombinationPlanarComplex`.
- **CN:** 定义 `EpilogueKind` 来表示该组件使用的 `EpilogueKind` 取值集合；代表性取值包括 `kUnknown`, `kConversion`, `kLinearCombination`, `kLinearCombinationClamp`, `kLinearCombinationPlanarComplex`。

### Lines 271-277
```cpp
271: enum class RuntimeDatatype {
272:   kStatic,
273:   kE4M3,
274:   kE5M2,
275:   kE3M2,
276:   kE2M3,
277:   kE2M1,
```
- **EN:** Defines `RuntimeDatatype` for the `RuntimeDatatype` value set used by this component; representative values include `kStatic`, `kE4M3`, `kE5M2`, `kE3M2`, `kE2M3`.
- **CN:** 定义 `RuntimeDatatype` 来表示该组件使用的 `RuntimeDatatype` 取值集合；代表性取值包括 `kStatic`, `kE4M3`, `kE5M2`, `kE3M2`, `kE2M3`。

### Lines 279-280
```cpp
279:   kInvalid
280: };
```
- **EN:** Supporting logic for the library metadata implementation.
- **CN:** 库元数据实现的辅助逻辑。

### Lines 283-288
```cpp
283: enum class RasterOrder {
284:   kAlongN,
285:   kAlongM,
286:   kHeuristic,
287:   kInvalid
288: };
```
- **EN:** Defines `RasterOrder` for the `RasterOrder` value set used by this component; representative values include `kAlongN`, `kAlongM`, `kHeuristic`, `kInvalid`.
- **CN:** 定义 `RasterOrder` 来表示该组件使用的 `RasterOrder` 取值集合；代表性取值包括 `kAlongN`, `kAlongM`, `kHeuristic`, `kInvalid`。

### Lines 290-290
```cpp
290: /////////////////////////////////////////////////////////////////////////////////////////////////
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 292-293
```cpp
292: } // namespace library
293: } // namespace cutlass
```
- **EN:** Supporting logic for the library metadata implementation.
- **CN:** 库元数据实现的辅助逻辑。

### Lines 295-295
```cpp
295: /////////////////////////////////////////////////////////////////////////////////////////////////
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

## Key Concepts / 关键概念
- **Strongly typed enums / 强类型枚举**
- **Structured type design / 结构化类型设计**
- **CUDA ecosystem integration / CUDA 生态集成**
- **Reference implementation / 参考实现**
- **Tensor manipulation / 张量处理**
- **GEMM specialization / GEMM 特化**

## Dependencies / 依赖关系
- **Runtime/backends / 运行时与后端:** `cuBLAS`, `cuDNN`
