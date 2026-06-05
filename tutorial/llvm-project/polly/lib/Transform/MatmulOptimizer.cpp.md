# MatmulOptimizer.cpp — Code Analysis / 代码分析

## Source / 来源

- **File**: `polly/lib/Transform/MatmulOptimizer.cpp`
- **Repository**: `llvm-project` / `llvm-project`
- **Purpose (EN)**: Implements Polly's matrix-multiplication pattern detection and schedule rewrite machinery.
- **用途（CN）**: 实现 Polly 的矩阵乘法模式识别与调度重写机制。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```cpp
1: //===- MatmulOptimizer.cpp -----------------------------------------------===//
2: //
3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
4: // See https://llvm.org/LICENSE.txt for license information.
5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
6: //
7: //===----------------------------------------------------------------------===//
```
- **EN**: Documents intent, invariants, or algorithmic background for the surrounding code.
- **CN**: 这里说明了周边代码的设计意图、不变量或算法背景。

### Lines 9-42
```cpp
9: #include "polly/MatmulOptimizer.h"
10: #include "polly/DependenceInfo.h"
11: #include "polly/Options.h"
12: #include "polly/ScheduleTreeTransform.h"
13: #include "polly/ScopInfo.h"
14: #include "polly/Simplify.h"
15: #include "polly/Support/GICHelper.h"
16: #include "polly/Support/ISLTools.h"
17: #include "llvm/ADT/ArrayRef.h"
18: #include "llvm/ADT/DenseSet.h"
19: #include "llvm/ADT/Sequence.h"
20: #include "llvm/ADT/SetOperations.h"
21: #include "llvm/ADT/SmallVector.h"
22: #include "llvm/ADT/StringRef.h"
23: #include "llvm/ADT/iterator_range.h"
24: #include "llvm/Analysis/TargetTransformInfo.h"
25: #include "llvm/IR/DataLayout.h"
26: #include "llvm/IR/Function.h"
27: #include "llvm/IR/Module.h"
28: #include "llvm/Support/CommandLine.h"
29: #include "llvm/Support/Debug.h"
30: #include "llvm/Support/TypeSize.h"
31: #include "llvm/Support/raw_ostream.h"
32: #include "isl/ctx.h"
33: #include "isl/schedule_node.h"
34: #include "isl/schedule_type.h"
35: #include "isl/union_map.h"
36: #include "isl/union_set.h"
37: #include <algorithm>
38: #include <cassert>
39: #include <cmath>
40: #include <cstdint>
41: #include <string>
42: #include <vector>
```
- **EN**: Brings in the headers required by this compilation unit, covering Polly, LLVM, ISL, or standard-library facilities.
- **CN**: 这里引入当前编译单元所需的头文件，覆盖 Polly、LLVM、ISL 或标准库能力。

### Lines 44-45
```cpp
44: #include "polly/Support/PollyDebug.h"
45: #define DEBUG_TYPE "polly-opt-isl"
```
- **EN**: Brings in the headers required by this compilation unit, covering Polly, LLVM, ISL, or standard-library facilities.
- **CN**: 这里引入当前编译单元所需的头文件，覆盖 Polly、LLVM、ISL 或标准库能力。

### Lines 47-47
```cpp
47: using namespace llvm;
```
- **EN**: Introduces namespace aliases/shorthands so later code can refer to LLVM or Polly entities more compactly.
- **CN**: 这里引入命名空间简写，使后续代码能更紧凑地引用 LLVM 或 Polly 实体。

### Lines 48-48
```cpp
48: using namespace polly;
```
- **EN**: Introduces namespace aliases/shorthands so later code can refer to LLVM or Polly entities more compactly.
- **CN**: 这里引入命名空间简写，使后续代码能更紧凑地引用 LLVM 或 Polly 实体。

### Lines 50-52
```cpp
50: namespace llvm {
51: class Value;
52: }
```
- **EN**: Defines `Value`, a named type that packages structured state or classification rules for the surrounding algorithm.
- **CN**: 这里定义了 `Value`，它是一个具名类型，用于为周边算法封装结构化状态或分类规则。

### Lines 54-59
```cpp
54: static cl::opt<int> LatencyVectorFma(
55:     "polly-target-latency-vector-fma",
56:     cl::desc("The minimal number of cycles between issuing two "
57:              "dependent consecutive vector fused multiply-add "
58:              "instructions."),
59:     cl::Hidden, cl::init(8), cl::cat(PollyCategory));
```
- **EN**: Declares the command-line option `LatencyVectorFma`, allowing Polly to tune this optimization according to target or experiment settings.
- **CN**: 这里声明命令行选项 `LatencyVectorFma`，使 Polly 能根据目标平台或实验设置调节该优化。

### Lines 61-66
```cpp
61: static cl::opt<int> ThroughputVectorFma(
62:     "polly-target-throughput-vector-fma",
63:     cl::desc("A throughput of the processor floating-point arithmetic units "
64:              "expressed in the number of vector fused multiply-add "
65:              "instructions per clock cycle."),
66:     cl::Hidden, cl::init(1), cl::cat(PollyCategory));
```
- **EN**: Declares the command-line option `ThroughputVectorFma`, allowing Polly to tune this optimization according to target or experiment settings.
- **CN**: 这里声明命令行选项 `ThroughputVectorFma`，使 Polly 能根据目标平台或实验设置调节该优化。

### Lines 68-71
```cpp
68: static cl::opt<int> FirstCacheLevelSize(
69:     "polly-target-1st-cache-level-size",
70:     cl::desc("The size of the first cache level specified in bytes."),
71:     cl::Hidden, cl::init(-1), cl::cat(PollyCategory));
```
- **EN**: Declares the command-line option `FirstCacheLevelSize`, allowing Polly to tune this optimization according to target or experiment settings.
- **CN**: 这里声明命令行选项 `FirstCacheLevelSize`，使 Polly 能根据目标平台或实验设置调节该优化。

### Lines 73-77
```cpp
73: static cl::opt<int> FirstCacheLevelDefaultSize(
74:     "polly-target-1st-cache-level-default-size",
75:     cl::desc("The default size of the first cache level specified in bytes"
76:              " (if not enough were provided by the TargetTransformInfo)."),
77:     cl::Hidden, cl::init(32768), cl::cat(PollyCategory));
```
- **EN**: Declares the command-line option `FirstCacheLevelDefaultSize`, allowing Polly to tune this optimization according to target or experiment settings.
- **CN**: 这里声明命令行选项 `FirstCacheLevelDefaultSize`，使 Polly 能根据目标平台或实验设置调节该优化。

### Lines 79-82
```cpp
79: static cl::opt<int> SecondCacheLevelSize(
80:     "polly-target-2nd-cache-level-size",
81:     cl::desc("The size of the second level specified in bytes."), cl::Hidden,
82:     cl::init(-1), cl::cat(PollyCategory));
```
- **EN**: Declares the command-line option `SecondCacheLevelSize`, allowing Polly to tune this optimization according to target or experiment settings.
- **CN**: 这里声明命令行选项 `SecondCacheLevelSize`，使 Polly 能根据目标平台或实验设置调节该优化。

### Lines 84-88
```cpp
84: static cl::opt<int> SecondCacheLevelDefaultSize(
85:     "polly-target-2nd-cache-level-default-size",
86:     cl::desc("The default size of the second cache level specified in bytes"
87:              " (if not enough were provided by the TargetTransformInfo)."),
88:     cl::Hidden, cl::init(262144), cl::cat(PollyCategory));
```
- **EN**: Declares the command-line option `SecondCacheLevelDefaultSize`, allowing Polly to tune this optimization according to target or experiment settings.
- **CN**: 这里声明命令行选项 `SecondCacheLevelDefaultSize`，使 Polly 能根据目标平台或实验设置调节该优化。

### Lines 90-96
```cpp
90: // This option, along with --polly-target-2nd-cache-level-associativity,
91: // --polly-target-1st-cache-level-size, and --polly-target-2st-cache-level-size
92: // represent the parameters of the target cache, which do not have typical
93: // values that can be used by default. However, to apply the pattern matching
94: // optimizations, we use the values of the parameters of Intel Core i7-3820
95: // SandyBridge in case the parameters are not specified or not provided by the
96: // TargetTransformInfo.
```
- **EN**: Documents intent, invariants, or algorithmic background for the surrounding code.
- **CN**: 这里说明了周边代码的设计意图、不变量或算法背景。

### Lines 97-100
```cpp
97: static cl::opt<int> FirstCacheLevelAssociativity(
98:     "polly-target-1st-cache-level-associativity",
99:     cl::desc("The associativity of the first cache level."), cl::Hidden,
100:     cl::init(-1), cl::cat(PollyCategory));
```
- **EN**: Declares the command-line option `FirstCacheLevelAssociativity`, allowing Polly to tune this optimization according to target or experiment settings.
- **CN**: 这里声明命令行选项 `FirstCacheLevelAssociativity`，使 Polly 能根据目标平台或实验设置调节该优化。

### Lines 102-106
```cpp
102: static cl::opt<int> FirstCacheLevelDefaultAssociativity(
103:     "polly-target-1st-cache-level-default-associativity",
104:     cl::desc("The default associativity of the first cache level"
105:              " (if not enough were provided by the TargetTransformInfo)."),
106:     cl::Hidden, cl::init(8), cl::cat(PollyCategory));
```
- **EN**: Declares the command-line option `FirstCacheLevelDefaultAssociativity`, allowing Polly to tune this optimization according to target or experiment settings.
- **CN**: 这里声明命令行选项 `FirstCacheLevelDefaultAssociativity`，使 Polly 能根据目标平台或实验设置调节该优化。

### Lines 108-111
```cpp
108: static cl::opt<int> SecondCacheLevelAssociativity(
109:     "polly-target-2nd-cache-level-associativity",
110:     cl::desc("The associativity of the second cache level."), cl::Hidden,
111:     cl::init(-1), cl::cat(PollyCategory));
```
- **EN**: Declares the command-line option `SecondCacheLevelAssociativity`, allowing Polly to tune this optimization according to target or experiment settings.
- **CN**: 这里声明命令行选项 `SecondCacheLevelAssociativity`，使 Polly 能根据目标平台或实验设置调节该优化。

### Lines 113-117
```cpp
113: static cl::opt<int> SecondCacheLevelDefaultAssociativity(
114:     "polly-target-2nd-cache-level-default-associativity",
115:     cl::desc("The default associativity of the second cache level"
116:              " (if not enough were provided by the TargetTransformInfo)."),
117:     cl::Hidden, cl::init(8), cl::cat(PollyCategory));
```
- **EN**: Declares the command-line option `SecondCacheLevelDefaultAssociativity`, allowing Polly to tune this optimization according to target or experiment settings.
- **CN**: 这里声明命令行选项 `SecondCacheLevelDefaultAssociativity`，使 Polly 能根据目标平台或实验设置调节该优化。

### Lines 119-123
```cpp
119: static cl::opt<int> VectorRegisterBitwidth(
120:     "polly-target-vector-register-bitwidth",
121:     cl::desc("The size in bits of a vector register (if not set, this "
122:              "information is taken from LLVM's target information."),
123:     cl::Hidden, cl::init(-1), cl::cat(PollyCategory));
```
- **EN**: Declares the command-line option `VectorRegisterBitwidth`, allowing Polly to tune this optimization according to target or experiment settings.
- **CN**: 这里声明命令行选项 `VectorRegisterBitwidth`，使 Polly 能根据目标平台或实验设置调节该优化。

### Lines 125-129
```cpp
125: static cl::opt<int> PollyPatternMatchingNcQuotient(
126:     "polly-pattern-matching-nc-quotient",
127:     cl::desc("Quotient that is obtained by dividing Nc, the parameter of the"
128:              "macro-kernel, by Nr, the parameter of the micro-kernel"),
129:     cl::Hidden, cl::init(256), cl::cat(PollyCategory));
```
- **EN**: Declares the command-line option `PollyPatternMatchingNcQuotient`, allowing Polly to tune this optimization according to target or experiment settings.
- **CN**: 这里声明命令行选项 `PollyPatternMatchingNcQuotient`，使 Polly 能根据目标平台或实验设置调节该优化。

### Lines 131-135
```cpp
131: static cl::opt<bool>
132:     PMBasedTCOpts("polly-tc-opt",
133:                   cl::desc("Perform optimizations of tensor contractions based "
134:                            "on pattern matching"),
135:                   cl::init(false), cl::ZeroOrMore, cl::cat(PollyCategory));
```
- **EN**: Declares the command-line option `PMBasedTCOpts`, allowing Polly to tune this optimization according to target or experiment settings.
- **CN**: 这里声明命令行选项 `PMBasedTCOpts`，使 Polly 能根据目标平台或实验设置调节该优化。

### Lines 137-141
```cpp
137: static cl::opt<bool>
138:     PMBasedMMMOpts("polly-matmul-opt",
139:                    cl::desc("Perform optimizations of matrix multiplications "
140:                             "based on pattern matching"),
141:                    cl::init(true), cl::ZeroOrMore, cl::cat(PollyCategory));
```
- **EN**: Declares the command-line option `PMBasedMMMOpts`, allowing Polly to tune this optimization according to target or experiment settings.
- **CN**: 这里声明命令行选项 `PMBasedMMMOpts`，使 Polly 能根据目标平台或实验设置调节该优化。

### Lines 143-147
```cpp
143: static cl::opt<int> OptComputeOut(
144:     "polly-tc-dependences-computeout",
145:     cl::desc("Bound the dependence analysis by a maximal amount of "
146:              "computational steps (0 means no bound)"),
147:     cl::Hidden, cl::init(500000), cl::ZeroOrMore, cl::cat(PollyCategory));
```
- **EN**: Declares the command-line option `OptComputeOut`, allowing Polly to tune this optimization according to target or experiment settings.
- **CN**: 这里声明命令行选项 `OptComputeOut`，使 Polly 能根据目标平台或实验设置调节该优化。

### Lines 149-157
```cpp
149: namespace {
150: /// Parameters of the micro kernel.
151: ///
152: /// Parameters, which determine sizes of rank-1 (i.e., outer product) update
153: /// used in the optimized matrix multiplication.
154: struct MicroKernelParamsTy {
155:   int Mr;
156:   int Nr;
157: };
```
- **EN**: Defines `MicroKernelParamsTy`, a named type that packages structured state or classification rules for the surrounding algorithm.
- **CN**: 这里定义了 `MicroKernelParamsTy`，它是一个具名类型，用于为周边算法封装结构化状态或分类规则。

### Lines 159-162
```cpp
159: /// Parameters of the macro kernel.
160: ///
161: /// Parameters, which determine sizes of blocks of partitioned matrices
162: /// used in the optimized matrix multiplication.
```
- **EN**: Documents intent, invariants, or algorithmic background for the surrounding code.
- **CN**: 这里说明了周边代码的设计意图、不变量或算法背景。

### Lines 163-167
```cpp
163: struct MacroKernelParamsTy {
164:   int Mc;
165:   int Nc;
166:   int Kc;
167: };
```
- **EN**: Defines `MacroKernelParamsTy`, a named type that packages structured state or classification rules for the surrounding algorithm.
- **CN**: 这里定义了 `MacroKernelParamsTy`，它是一个具名类型，用于为周边算法封装结构化状态或分类规则。

### Lines 169-172
```cpp
169: /// Parameters of the matrix multiplication operands.
170: ///
171: /// Parameters, which describe access relations that represent operands of the
172: /// matrix multiplication.
```
- **EN**: Documents intent, invariants, or algorithmic background for the surrounding code.
- **CN**: 这里说明了周边代码的设计意图、不变量或算法背景。

### Lines 173-181
```cpp
173: struct MatMulInfoTy {
174:   MemoryAccess *A = nullptr;
175:   MemoryAccess *B = nullptr;
176:   MemoryAccess *ReadFromC = nullptr;
177:   MemoryAccess *WriteToC = nullptr;
178:   int i = -1;
179:   int j = -1;
180:   int k = -1;
181: };
```
- **EN**: Defines `MatMulInfoTy`, a named type that packages structured state or classification rules for the surrounding algorithm.
- **CN**: 这里定义了 `MatMulInfoTy`，它是一个具名类型，用于为周边算法封装结构化状态或分类规则。

### Lines 183-212
```cpp
183: /// Parameters of the tensor contraction operands.
184: ///
185: /// A general d-dimensional tensor T ∈ R ^ Nu0 x ... x Nud−1 can be defined
186: /// as the set of scalar elements indexed by the set of indices u0 ... ud,
187: ///
188: /// T ≡ {Anu0...nud−1 ∈ R | (u0,...,ud−1) ∈ Nu0 x ... x Nud−1}.
189: ///
190: /// Let A, B, and C be dA, dB, and dC-dimensional tensors, respectively.
191: /// Let the free and the contracted indices of the tensor A be grouped into
192: /// two bundles I = i0...ir−1 and P = p0...pt−1, respectively. Similarly,
193: /// the free and the contracted indices of B are grouped into bundles
194: /// J = j0..js−1 and P and the free indices of C are grouped into
195: /// bundles I and J.
196: ///
197: /// Tensor contraction (TC) of tensors A, B into tensor C can be represented as
198: /// C(shuffle(I,J))=∑α·A(shuffle(I,P))·B(shuffle(P,J))+β·C(shuffle(I,J)),
199: /// where ∑ is a summation over all contracted indices of P,
200: /// α, β ∈ R, Npi is the length of the tensor dimension that corresponds
201: /// to the index pi, A(shuffle(I, P)), B(shuffle(P, J)), C(shuffle(I, J)) are
202: /// accesses to tensors A, B, C, respectively,
203: /// shuffle(I, J), shuffle(I, P), and shuffle(P, J) are permutations of
204: /// the enclosed indices.
205: ///
206: /// Multiplication of C(shuffle(I,J)) by β can be moved into a different SCoP
207: /// statement by loop distribution, which is done by the isl scheduler.
208: //  If β is not equal to one, the optimization of TC of Polly requires
209: /// such a transformation.
210: ///
211: /// TCInfoTy contains parameters, which describe access relations that represent
212: /// operands of the tensor contraction.
```
- **EN**: Documents intent, invariants, or algorithmic background for the surrounding code.
- **CN**: 这里说明了周边代码的设计意图、不变量或算法背景。

### Lines 213-219
```cpp
213: struct TCInfoTy {
214:   /// @{
215:   /// Memory accesses that represent reading from tensors, which are operands of
216:   /// the tensor contraction.
217:   MemoryAccess *A = nullptr;
218:   MemoryAccess *B = nullptr;
219:   /// @}
```
- **EN**: Defines `TCInfoTy`, a named type that packages structured state or classification rules for the surrounding algorithm.
- **CN**: 这里定义了 `TCInfoTy`，它是一个具名类型，用于为周边算法封装结构化状态或分类规则。

### Lines 221-223
```cpp
221:   /// @{
222:   /// Memory accesses that represent reading from and writing into the tensor,
223:   /// which contains the result of the tensor contraction.
```
- **EN**: Documents intent, invariants, or algorithmic background for the surrounding code.
- **CN**: 这里说明了周边代码的设计意图、不变量或算法背景。

### Lines 224-224
```cpp
224:   MemoryAccess *ReadFromC = nullptr;
```
- **EN**: Carries supporting computation, data movement, or bookkeeping for the surrounding algorithm.
- **CN**: 这里承担周边算法所需的辅助计算、数据传递或簿记工作。

### Lines 225-225
```cpp
225:   MemoryAccess *WriteToC = nullptr;
```
- **EN**: Carries supporting computation, data movement, or bookkeeping for the surrounding algorithm.
- **CN**: 这里承担周边算法所需的辅助计算、数据传递或簿记工作。

### Lines 226-226
```cpp
226:   /// @}
```
- **EN**: Documents intent, invariants, or algorithmic background for the surrounding code.
- **CN**: 这里说明了周边代码的设计意图、不变量或算法背景。

### Lines 228-230
```cpp
228:   /// @{
229:   /// Input dimensions of the schedule space, which represent free
230:   /// indices of tensors.
```
- **EN**: Documents intent, invariants, or algorithmic background for the surrounding code.
- **CN**: 这里说明了周边代码的设计意图、不变量或算法背景。

### Lines 231-231
```cpp
231:   SmallDenseSet<int> I;
```
- **EN**: Carries supporting computation, data movement, or bookkeeping for the surrounding algorithm.
- **CN**: 这里承担周边算法所需的辅助计算、数据传递或簿记工作。

### Lines 232-232
```cpp
232:   SmallDenseSet<int> J;
```
- **EN**: Carries supporting computation, data movement, or bookkeeping for the surrounding algorithm.
- **CN**: 这里承担周边算法所需的辅助计算、数据传递或簿记工作。

### Lines 233-233
```cpp
233:   /// @}
```
- **EN**: Documents intent, invariants, or algorithmic background for the surrounding code.
- **CN**: 这里说明了周边代码的设计意图、不变量或算法背景。

### Lines 235-236
```cpp
235:   /// Input dimension of the schedule space, which represents contracted
236:   /// indices of tensors.
```
- **EN**: Documents intent, invariants, or algorithmic background for the surrounding code.
- **CN**: 这里说明了周边代码的设计意图、不变量或算法背景。

### Lines 237-237
```cpp
237:   SmallDenseSet<int> P;
```
- **EN**: Carries supporting computation, data movement, or bookkeeping for the surrounding algorithm.
- **CN**: 这里承担周边算法所需的辅助计算、数据传递或簿记工作。

### Lines 239-244
```cpp
239:   /// @{
240:   /// Sizes of tensor dimensions for corresponding input dimensions of
241:   /// the schedule space. The size of the tensor dimension can be larger than
242:   /// the size of the corresponding input dimension of the schedule space.
243:   /// This does not correspond to a tensor contraction. However, such a pattern
244:   /// will be optimized by the transformation.
```
- **EN**: Documents intent, invariants, or algorithmic background for the surrounding code.
- **CN**: 这里说明了周边代码的设计意图、不变量或算法背景。

### Lines 245-245
```cpp
245:   SmallVector<int> DimensionSizes;
```
- **EN**: Carries supporting computation, data movement, or bookkeeping for the surrounding algorithm.
- **CN**: 这里承担周边算法所需的辅助计算、数据传递或簿记工作。

### Lines 246-246
```cpp
246:   SmallVector<int> ADimensions;
```
- **EN**: Carries supporting computation, data movement, or bookkeeping for the surrounding algorithm.
- **CN**: 这里承担周边算法所需的辅助计算、数据传递或簿记工作。

### Lines 247-247
```cpp
247:   SmallVector<int> BDimensions;
```
- **EN**: Carries supporting computation, data movement, or bookkeeping for the surrounding algorithm.
- **CN**: 这里承担周边算法所需的辅助计算、数据传递或簿记工作。

### Lines 248-248
```cpp
248:   SmallVector<int> CDimensions;
```
- **EN**: Carries supporting computation, data movement, or bookkeeping for the surrounding algorithm.
- **CN**: 这里承担周边算法所需的辅助计算、数据传递或簿记工作。

### Lines 249-249
```cpp
249:   /// @}
```
- **EN**: Documents intent, invariants, or algorithmic background for the surrounding code.
- **CN**: 这里说明了周边代码的设计意图、不变量或算法背景。

### Lines 251-253
```cpp
251:   /// @{
252:   /// Permutations of indices of I, J, and P, which describe operands of
253:   /// the tensor contraction and its result.
```
- **EN**: Documents intent, invariants, or algorithmic background for the surrounding code.
- **CN**: 这里说明了周边代码的设计意图、不变量或算法背景。

### Lines 254-254
```cpp
254:   SmallVector<int> OrderedI;
```
- **EN**: Carries supporting computation, data movement, or bookkeeping for the surrounding algorithm.
- **CN**: 这里承担周边算法所需的辅助计算、数据传递或簿记工作。

### Lines 255-255
```cpp
255:   SmallVector<int> OrderedJ;
```
- **EN**: Carries supporting computation, data movement, or bookkeeping for the surrounding algorithm.
- **CN**: 这里承担周边算法所需的辅助计算、数据传递或簿记工作。

### Lines 256-256
```cpp
256:   SmallVector<int> OrderedP;
```
- **EN**: Carries supporting computation, data movement, or bookkeeping for the surrounding algorithm.
- **CN**: 这里承担周边算法所需的辅助计算、数据传递或簿记工作。

### Lines 257-257
```cpp
257:   /// @}
```
- **EN**: Documents intent, invariants, or algorithmic background for the surrounding code.
- **CN**: 这里说明了周边代码的设计意图、不变量或算法背景。

### Lines 258-258
```cpp
258: };
```
- **EN**: Closes the current block or scope and hands control back to the surrounding context.
- **CN**: 这里关闭当前代码块或作用域，并将控制流交还给外层上下文。

### Lines 260-263
```cpp
260: /// Create an isl::union_set, which describes the option of the form
261: /// [isolate[] -> unroll[x]].
262: ///
263: /// @param Ctx An isl::ctx, which is used to create the isl::union_set.
```
- **EN**: Documents intent, invariants, or algorithmic background for the surrounding code.
- **CN**: 这里说明了周边代码的设计意图、不变量或算法背景。

### Lines 264-274
```cpp
264: static isl::union_set getUnrollIsolatedSetOptions(isl::ctx Ctx) {
265:   isl::space Space = isl::space(Ctx, 0, 0, 1);
266:   isl::map UnrollIsolatedSetOption = isl::map::universe(Space);
267:   isl::id DimInId = isl::id::alloc(Ctx, "isolate", nullptr);
268:   isl::id DimOutId = isl::id::alloc(Ctx, "unroll", nullptr);
269:   UnrollIsolatedSetOption =
270:       UnrollIsolatedSetOption.set_tuple_id(isl::dim::in, DimInId);
271:   UnrollIsolatedSetOption =
272:       UnrollIsolatedSetOption.set_tuple_id(isl::dim::out, DimOutId);
273:   return UnrollIsolatedSetOption.wrap();
274: }
```
- **EN**: Introduces or continues `getUnrollIsolatedSetOptions`, a routine that carries out one step of this file's analysis or transformation pipeline.
- **CN**: 这里引入或继续 `getUnrollIsolatedSetOptions`，它是该文件分析或变换流水线中的一个步骤函数。

### Lines 276-285
```cpp
276: /// Permute the two dimensions of the isl map.
277: ///
278: /// Permute @p DstPos and @p SrcPos dimensions of the isl map @p Map that
279: /// have type @p DimType.
280: ///
281: /// @param Map     The isl map to be modified.
282: /// @param DimType The type of the dimensions.
283: /// @param DstPos  The first dimension.
284: /// @param SrcPos  The second dimension.
285: /// @return        The modified map.
```
- **EN**: Documents intent, invariants, or algorithmic background for the surrounding code.
- **CN**: 这里说明了周边代码的设计意图、不变量或算法背景。

### Lines 286-310
```cpp
286: static isl::map permuteDimensions(isl::map Map, isl::dim DimType,
287:                                   unsigned DstPos, unsigned SrcPos) {
288:   assert(DstPos < unsignedFromIslSize(Map.dim(DimType)) &&
289:          SrcPos < unsignedFromIslSize(Map.dim(DimType)));
290:   if (DstPos == SrcPos)
291:     return Map;
292:   isl::id DimId;
293:   if (Map.has_tuple_id(DimType))
294:     DimId = Map.get_tuple_id(DimType);
295:   auto FreeDim = DimType == isl::dim::in ? isl::dim::out : isl::dim::in;
296:   isl::id FreeDimId;
297:   if (Map.has_tuple_id(FreeDim))
298:     FreeDimId = Map.get_tuple_id(FreeDim);
299:   auto MaxDim = std::max(DstPos, SrcPos);
300:   auto MinDim = std::min(DstPos, SrcPos);
301:   Map = Map.move_dims(FreeDim, 0, DimType, MaxDim, 1);
302:   Map = Map.move_dims(FreeDim, 0, DimType, MinDim, 1);
303:   Map = Map.move_dims(DimType, MinDim, FreeDim, 1, 1);
304:   Map = Map.move_dims(DimType, MaxDim, FreeDim, 0, 1);
305:   if (!DimId.is_null())
306:     Map = Map.set_tuple_id(DimType, DimId);
307:   if (!FreeDimId.is_null())
308:     Map = Map.set_tuple_id(FreeDim, FreeDimId);
309:   return Map;
310: }
```
- **EN**: Checks a condition and selects the appropriate handling path, preserving correctness for special cases.
- **CN**: 这里检查条件并选择合适的处理路径，以保证特殊情况的正确性。

### Lines 312-323
```cpp
312: /// Check the form of the access relation.
313: ///
314: /// Check that the access relation @p AccMap has the form M[i][j], where i
315: /// is a @p FirstPos and j is a @p SecondPos.
316: ///
317: /// @param AccMap    The access relation to be checked.
318: /// @param FirstPos  The index of the input dimension that is mapped to
319: ///                  the first output dimension.
320: /// @param SecondPos The index of the input dimension that is mapped to the
321: ///                  second output dimension.
322: /// @return          True in case @p AccMap has the expected form and false,
323: ///                  otherwise.
```
- **EN**: Documents intent, invariants, or algorithmic background for the surrounding code.
- **CN**: 这里说明了周边代码的设计意图、不变量或算法背景。

### Lines 324-327
```cpp
324: static bool isMatMulOperandAcc(isl::set Domain, isl::map AccMap, int &FirstPos,
325:                                int &SecondPos) {
326:   isl::space Space = AccMap.get_space();
327:   isl::map Universe = isl::map::universe(Space);
```
- **EN**: Introduces or continues `isMatMulOperandAcc`, a routine that carries out one step of this file's analysis or transformation pipeline.
- **CN**: 这里引入或继续 `isMatMulOperandAcc`，它是该文件分析或变换流水线中的一个步骤函数。

### Lines 329-330
```cpp
329:   if (unsignedFromIslSize(Space.dim(isl::dim::out)) != 2)
330:     return false;
```
- **EN**: Checks a condition and selects the appropriate handling path, preserving correctness for special cases.
- **CN**: 这里检查条件并选择合适的处理路径，以保证特殊情况的正确性。

### Lines 332-338
```cpp
332:   // MatMul has the form:
333:   // for (i = 0; i < N; i++)
334:   //   for (j = 0; j < M; j++)
335:   //     for (k = 0; k < P; k++)
336:   //       C[i, j] += A[i, k] * B[k, j]
337:   //
338:   // Permutation of three outer loops: 3! = 6 possibilities.
```
- **EN**: Documents intent, invariants, or algorithmic background for the surrounding code.
- **CN**: 这里说明了周边代码的设计意图、不变量或算法背景。

### Lines 339-339
```cpp
339:   int FirstDims[] = {0, 0, 1, 1, 2, 2};
```
- **EN**: Carries supporting computation, data movement, or bookkeeping for the surrounding algorithm.
- **CN**: 这里承担周边算法所需的辅助计算、数据传递或簿记工作。

### Lines 340-340
```cpp
340:   int SecondDims[] = {1, 2, 2, 0, 0, 1};
```
- **EN**: Carries supporting computation, data movement, or bookkeeping for the surrounding algorithm.
- **CN**: 这里承担周边算法所需的辅助计算、数据传递或簿记工作。

### Lines 341-344
```cpp
341:   for (int i = 0; i < 6; i += 1) {
342:     auto PossibleMatMul =
343:         Universe.equate(isl::dim::in, FirstDims[i], isl::dim::out, 0)
344:             .equate(isl::dim::in, SecondDims[i], isl::dim::out, 1);
```
- **EN**: Executes an iterative step of the algorithm, usually scanning statements, dimensions, accesses, operands, or schedule nodes.
- **CN**: 这里执行算法中的迭代步骤，通常遍历语句、维度、访问、操作数或调度节点。

### Lines 346-346
```cpp
346:     AccMap = AccMap.intersect_domain(Domain);
```
- **EN**: Introduces or continues `intersect_domain`, a routine that carries out one step of this file's analysis or transformation pipeline.
- **CN**: 这里引入或继续 `intersect_domain`，它是该文件分析或变换流水线中的一个步骤函数。

### Lines 347-347
```cpp
347:     PossibleMatMul = PossibleMatMul.intersect_domain(Domain);
```
- **EN**: Introduces or continues `intersect_domain`, a routine that carries out one step of this file's analysis or transformation pipeline.
- **CN**: 这里引入或继续 `intersect_domain`，它是该文件分析或变换流水线中的一个步骤函数。

### Lines 349-353
```cpp
349:     // If AccMap spans entire domain (Non-partial write),
350:     // compute FirstPos and SecondPos.
351:     // If AccMap != PossibleMatMul here (the two maps have been gisted at
352:     // this point), it means that the writes are not complete, or in other
353:     // words, it is a Partial write and Partial writes must be rejected.
```
- **EN**: Documents intent, invariants, or algorithmic background for the surrounding code.
- **CN**: 这里说明了周边代码的设计意图、不变量或算法背景。

### Lines 354-362
```cpp
354:     if (AccMap.is_equal(PossibleMatMul)) {
355:       if (FirstPos != -1 && FirstPos != FirstDims[i])
356:         continue;
357:       FirstPos = FirstDims[i];
358:       if (SecondPos != -1 && SecondPos != SecondDims[i])
359:         continue;
360:       SecondPos = SecondDims[i];
361:       return true;
362:     }
```
- **EN**: Checks a condition and selects the appropriate handling path, preserving correctness for special cases.
- **CN**: 这里检查条件并选择合适的处理路径，以保证特殊情况的正确性。

### Lines 363-363
```cpp
363:   }
```
- **EN**: Closes the current block or scope and hands control back to the surrounding context.
- **CN**: 这里关闭当前代码块或作用域，并将控制流交还给外层上下文。

### Lines 365-365
```cpp
365:   return false;
```
- **EN**: Returns the computed value or forwards the current result to the caller.
- **CN**: 这里返回计算结果，或将当前结果继续传递给调用方。

### Lines 366-366
```cpp
366: }
```
- **EN**: Closes the current block or scope and hands control back to the surrounding context.
- **CN**: 这里关闭当前代码块或作用域，并将控制流交还给外层上下文。

### Lines 368-378
```cpp
368: /// Does the memory access represent a non-scalar operand of the matrix
369: /// multiplication.
370: ///
371: /// Check that the memory access @p MemAccess is the read access to a non-scalar
372: /// operand of the matrix multiplication or its result.
373: ///
374: /// @param MemAccess The memory access to be checked.
375: /// @param MMI       Parameters of the matrix multiplication operands.
376: /// @return          True in case the memory access represents the read access
377: ///                  to a non-scalar operand of the matrix multiplication and
378: ///                  false, otherwise.
```
- **EN**: Documents intent, invariants, or algorithmic background for the surrounding code.
- **CN**: 这里说明了周边代码的设计意图、不变量或算法背景。

### Lines 379-398
```cpp
379: static bool isMatMulNonScalarReadAccess(MemoryAccess *MemAccess,
380:                                         MatMulInfoTy &MMI) {
381:   if (!MemAccess->isLatestArrayKind() || !MemAccess->isRead())
382:     return false;
383:   auto AccMap = MemAccess->getLatestAccessRelation();
384:   isl::set StmtDomain = MemAccess->getStatement()->getDomain();
385:   if (isMatMulOperandAcc(StmtDomain, AccMap, MMI.i, MMI.j) && !MMI.ReadFromC) {
386:     MMI.ReadFromC = MemAccess;
387:     return true;
388:   }
389:   if (isMatMulOperandAcc(StmtDomain, AccMap, MMI.i, MMI.k) && !MMI.A) {
390:     MMI.A = MemAccess;
391:     return true;
392:   }
393:   if (isMatMulOperandAcc(StmtDomain, AccMap, MMI.k, MMI.j) && !MMI.B) {
394:     MMI.B = MemAccess;
395:     return true;
396:   }
397:   return false;
398: }
```
- **EN**: Checks a condition and selects the appropriate handling path, preserving correctness for special cases.
- **CN**: 这里检查条件并选择合适的处理路径，以保证特殊情况的正确性。

### Lines 400-412
```cpp
400: /// Check accesses to operands of the matrix multiplication.
401: ///
402: /// Check that accesses of the SCoP statement, which corresponds to
403: /// the partial schedule @p PartialSchedule, are scalar in terms of loops
404: /// containing the matrix multiplication, in case they do not represent
405: /// accesses to the non-scalar operands of the matrix multiplication or
406: /// its result.
407: ///
408: /// @param  PartialSchedule The partial schedule of the SCoP statement.
409: /// @param  MMI             Parameters of the matrix multiplication operands.
410: /// @return                 True in case the corresponding SCoP statement
411: ///                         represents matrix multiplication and false,
412: ///                         otherwise.
```
- **EN**: Documents intent, invariants, or algorithmic background for the surrounding code.
- **CN**: 这里说明了周边代码的设计意图、不变量或算法背景。

### Lines 413-426
```cpp
413: static bool containsOnlyMatrMultAcc(isl::map PartialSchedule,
414:                                     MatMulInfoTy &MMI) {
415:   auto InputDimId = PartialSchedule.get_tuple_id(isl::dim::in);
416:   auto *Stmt = static_cast<ScopStmt *>(InputDimId.get_user());
417:   unsigned OutDimNum = unsignedFromIslSize(PartialSchedule.range_tuple_dim());
418:   assert(OutDimNum > 2 && "In case of the matrix multiplication the loop nest "
419:                           "and, consequently, the corresponding scheduling "
420:                           "functions have at least three dimensions.");
421:   auto MapI =
422:       permuteDimensions(PartialSchedule, isl::dim::out, MMI.i, OutDimNum - 1);
423:   auto MapJ =
424:       permuteDimensions(PartialSchedule, isl::dim::out, MMI.j, OutDimNum - 1);
425:   auto MapK =
426:       permuteDimensions(PartialSchedule, isl::dim::out, MMI.k, OutDimNum - 1);
```
- **EN**: Introduces or continues `containsOnlyMatrMultAcc`, a routine that carries out one step of this file's analysis or transformation pipeline.
- **CN**: 这里引入或继续 `containsOnlyMatrMultAcc`，它是该文件分析或变换流水线中的一个步骤函数。

### Lines 428-428
```cpp
428:   auto Accesses = getAccessesInOrder(*Stmt);
```
- **EN**: Introduces or continues `getAccessesInOrder`, a routine that carries out one step of this file's analysis or transformation pipeline.
- **CN**: 这里引入或继续 `getAccessesInOrder`，它是该文件分析或变换流水线中的一个步骤函数。

### Lines 429-436
```cpp
429:   for (auto *MemA = Accesses.begin(); MemA != Accesses.end() - 1; MemA++) {
430:     auto *MemAccessPtr = *MemA;
431:     if (MemAccessPtr->isLatestArrayKind() && MemAccessPtr != MMI.WriteToC &&
432:         !isMatMulNonScalarReadAccess(MemAccessPtr, MMI) &&
433:         !(MemAccessPtr->isStrideZero(MapI) &&
434:           MemAccessPtr->isStrideZero(MapJ) && MemAccessPtr->isStrideZero(MapK)))
435:       return false;
436:   }
```
- **EN**: Executes an iterative step of the algorithm, usually scanning statements, dimensions, accesses, operands, or schedule nodes.
- **CN**: 这里执行算法中的迭代步骤，通常遍历语句、维度、访问、操作数或调度节点。

### Lines 437-437
```cpp
437:   return true;
```
- **EN**: Returns the computed value or forwards the current result to the caller.
- **CN**: 这里返回计算结果，或将当前结果继续传递给调用方。

### Lines 438-438
```cpp
438: }
```
- **EN**: Closes the current block or scope and hands control back to the surrounding context.
- **CN**: 这里关闭当前代码块或作用域，并将控制流交还给外层上下文。

### Lines 440-453
```cpp
440: /// Check for dependencies corresponding to the matrix multiplication.
441: ///
442: /// Check that there is only true dependence of the form
443: /// S(..., k, ...) -> S(..., k + 1, …), where S is the SCoP statement
444: /// represented by @p Schedule and k is @p Pos. Such a dependence corresponds
445: /// to the dependency produced by the matrix multiplication.
446: ///
447: /// @param  Schedule The schedule of the SCoP statement.
448: /// @param  D The SCoP dependencies.
449: /// @param  Pos The parameter to describe an acceptable true dependence.
450: ///             In case it has a negative value, try to determine its
451: ///             acceptable value.
452: /// @return True in case dependencies correspond to the matrix multiplication
453: ///         and false, otherwise.
```
- **EN**: Documents intent, invariants, or algorithmic background for the surrounding code.
- **CN**: 这里说明了周边代码的设计意图、不变量或算法背景。

### Lines 454-473
```cpp
454: static bool containsOnlyMatMulDep(isl::map Schedule, const Dependences *D,
455:                                   int &Pos) {
456:   isl::union_map Dep = D->getDependences(Dependences::TYPE_RAW);
457:   isl::union_map Red = D->getDependences(Dependences::TYPE_RED);
458:   if (!Red.is_null())
459:     Dep = Dep.unite(Red);
460:   auto DomainSpace = Schedule.get_space().domain();
461:   auto Space = DomainSpace.map_from_domain_and_range(DomainSpace);
462:   auto Deltas = Dep.extract_map(Space).deltas();
463:   int DeltasDimNum = unsignedFromIslSize(Deltas.dim(isl::dim::set));
464:   for (int i = 0; i < DeltasDimNum; i++) {
465:     auto Val = Deltas.plain_get_val_if_fixed(isl::dim::set, i);
466:     Pos = Pos < 0 && Val.is_one() ? i : Pos;
467:     if (Val.is_nan() || !(Val.is_zero() || (i == Pos && Val.is_one())))
468:       return false;
469:   }
470:   if (DeltasDimNum == 0 || Pos < 0)
471:     return false;
472:   return true;
473: }
```
- **EN**: Executes an iterative step of the algorithm, usually scanning statements, dimensions, accesses, operands, or schedule nodes.
- **CN**: 这里执行算法中的迭代步骤，通常遍历语句、维度、访问、操作数或调度节点。

### Lines 475-497
```cpp
475: /// Check if the SCoP statement could probably be optimized with analytical
476: /// modeling.
477: ///
478: /// containsMatrMult tries to determine whether the following conditions
479: /// are true:
480: /// 1. The last memory access modeling an array, MA1, represents writing to
481: ///    memory and has the form S(..., i1, ..., i2, ...) -> M(i1, i2) or
482: ///    S(..., i2, ..., i1, ...) -> M(i1, i2), where S is the SCoP statement
483: ///    under consideration.
484: /// 2. There is only one loop-carried true dependency, and it has the
485: ///    form S(..., i3, ...) -> S(..., i3 + 1, ...), and there are no
486: ///    loop-carried or anti dependencies.
487: /// 3. SCoP contains three access relations, MA2, MA3, and MA4 that represent
488: ///    reading from memory and have the form S(..., i3, ...) -> M(i1, i3),
489: ///    S(..., i3, ...) -> M(i3, i2), S(...) -> M(i1, i2), respectively,
490: ///    and all memory accesses of the SCoP that are different from MA1, MA2,
491: ///    MA3, and MA4 have stride 0, if the innermost loop is exchanged with any
492: ///    of loops i1, i2 and i3.
493: ///
494: /// @param PartialSchedule The PartialSchedule that contains a SCoP statement
495: ///        to check.
496: /// @D     The SCoP dependencies.
497: /// @MMI   Parameters of the matrix multiplication operands.
```
- **EN**: Documents intent, invariants, or algorithmic background for the surrounding code.
- **CN**: 这里说明了周边代码的设计意图、不变量或算法背景。

### Lines 498-503
```cpp
498: static bool containsMatrMult(isl::map PartialSchedule, const Dependences *D,
499:                              MatMulInfoTy &MMI) {
500:   auto InputDimsId = PartialSchedule.get_tuple_id(isl::dim::in);
501:   auto *Stmt = static_cast<ScopStmt *>(InputDimsId.get_user());
502:   if (Stmt->size() <= 1)
503:     return false;
```
- **EN**: Checks a condition and selects the appropriate handling path, preserving correctness for special cases.
- **CN**: 这里检查条件并选择合适的处理路径，以保证特殊情况的正确性。

### Lines 505-505
```cpp
505:   auto Accesses = getAccessesInOrder(*Stmt);
```
- **EN**: Introduces or continues `getAccessesInOrder`, a routine that carries out one step of this file's analysis or transformation pipeline.
- **CN**: 这里引入或继续 `getAccessesInOrder`，它是该文件分析或变换流水线中的一个步骤函数。

### Lines 506-517
```cpp
506:   for (auto *MemA = Accesses.end() - 1; MemA != Accesses.begin(); MemA--) {
507:     auto *MemAccessPtr = *MemA;
508:     if (!MemAccessPtr->isLatestArrayKind())
509:       continue;
510:     if (!MemAccessPtr->isWrite())
511:       return false;
512:     auto AccMap = MemAccessPtr->getLatestAccessRelation();
513:     if (!isMatMulOperandAcc(Stmt->getDomain(), AccMap, MMI.i, MMI.j))
514:       return false;
515:     MMI.WriteToC = MemAccessPtr;
516:     break;
517:   }
```
- **EN**: Executes an iterative step of the algorithm, usually scanning statements, dimensions, accesses, operands, or schedule nodes.
- **CN**: 这里执行算法中的迭代步骤，通常遍历语句、维度、访问、操作数或调度节点。

### Lines 519-520
```cpp
519:   if (!containsOnlyMatMulDep(PartialSchedule, D, MMI.k))
520:     return false;
```
- **EN**: Checks a condition and selects the appropriate handling path, preserving correctness for special cases.
- **CN**: 这里检查条件并选择合适的处理路径，以保证特殊情况的正确性。

### Lines 522-523
```cpp
522:   if (!MMI.WriteToC || !containsOnlyMatrMultAcc(PartialSchedule, MMI))
523:     return false;
```
- **EN**: Checks a condition and selects the appropriate handling path, preserving correctness for special cases.
- **CN**: 这里检查条件并选择合适的处理路径，以保证特殊情况的正确性。

### Lines 525-526
```cpp
525:   if (!MMI.A || !MMI.B || !MMI.ReadFromC)
526:     return false;
```
- **EN**: Checks a condition and selects the appropriate handling path, preserving correctness for special cases.
- **CN**: 这里检查条件并选择合适的处理路径，以保证特殊情况的正确性。

### Lines 527-527
```cpp
527:   return true;
```
- **EN**: Returns the computed value or forwards the current result to the caller.
- **CN**: 这里返回计算结果，或将当前结果继续传递给调用方。

### Lines 528-528
```cpp
528: }
```
- **EN**: Closes the current block or scope and hands control back to the surrounding context.
- **CN**: 这里关闭当前代码块或作用域，并将控制流交还给外层上下文。

### Lines 530-536
```cpp
530: /// Permute two dimensions of the band node.
531: ///
532: /// Permute FirstDim and SecondDim dimensions of the Node.
533: ///
534: /// @param Node The band node to be modified.
535: /// @param FirstDim The first dimension to be permuted.
536: /// @param SecondDim The second dimension to be permuted.
```
- **EN**: Documents intent, invariants, or algorithmic background for the surrounding code.
- **CN**: 这里说明了周边代码的设计意图、不变量或算法背景。

### Lines 537-553
```cpp
537: static isl::schedule_node permuteBandNodeDimensions(isl::schedule_node Node,
538:                                                     unsigned FirstDim,
539:                                                     unsigned SecondDim) {
540:   assert(isl_schedule_node_get_type(Node.get()) == isl_schedule_node_band &&
541:          (unsigned)isl_schedule_node_band_n_member(Node.get()) >
542:              std::max(FirstDim, SecondDim));
543:   auto PartialSchedule =
544:       isl::manage(isl_schedule_node_band_get_partial_schedule(Node.get()));
545:   auto PartialScheduleFirstDim = PartialSchedule.at(FirstDim);
546:   auto PartialScheduleSecondDim = PartialSchedule.at(SecondDim);
547:   PartialSchedule =
548:       PartialSchedule.set_union_pw_aff(SecondDim, PartialScheduleFirstDim);
549:   PartialSchedule =
550:       PartialSchedule.set_union_pw_aff(FirstDim, PartialScheduleSecondDim);
551:   Node = isl::manage(isl_schedule_node_delete(Node.release()));
552:   return Node.insert_partial_schedule(PartialSchedule);
553: }
```
- **EN**: Introduces or continues `permuteBandNodeDimensions`, a routine that carries out one step of this file's analysis or transformation pipeline.
- **CN**: 这里引入或继续 `permuteBandNodeDimensions`，它是该文件分析或变换流水线中的一个步骤函数。

### Lines 555-562
```cpp
555: static isl::schedule_node
556: createMicroKernel(isl::schedule_node Node,
557:                   MicroKernelParamsTy MicroKernelParams) {
558:   Node = applyRegisterTiling(Node, {MicroKernelParams.Mr, MicroKernelParams.Nr},
559:                              1);
560:   Node = Node.parent().parent();
561:   return permuteBandNodeDimensions(Node, 0, 1).child(0).child(0);
562: }
```
- **EN**: Introduces or continues `createMicroKernel`, a routine that carries out one step of this file's analysis or transformation pipeline.
- **CN**: 这里引入或继续 `createMicroKernel`，它是该文件分析或变换流水线中的一个步骤函数。

### Lines 564-573
```cpp
564: /// Create the BLIS macro-kernel.
565: ///
566: /// We create the BLIS macro-kernel by applying a combination of tiling
567: /// of dimensions of the band node and interchanging of two innermost
568: /// modified dimensions. The values of MacroKernelParams's fields are used
569: /// as tile sizes.
570: ///
571: /// @param Node The schedule node to be modified.
572: /// @param MacroKernelParams Parameters of the macro kernel
573: ///                          to be used as tile sizes.
```
- **EN**: Documents intent, invariants, or algorithmic background for the surrounding code.
- **CN**: 这里说明了周边代码的设计意图、不变量或算法背景。

### Lines 574-589
```cpp
574: static isl::schedule_node
575: createMacroKernel(isl::schedule_node Node,
576:                   MacroKernelParamsTy MacroKernelParams) {
577:   assert(isl_schedule_node_get_type(Node.get()) == isl_schedule_node_band);
578:   if (MacroKernelParams.Mc == 1 && MacroKernelParams.Nc == 1 &&
579:       MacroKernelParams.Kc == 1)
580:     return Node;
581:   int DimOutNum = isl_schedule_node_band_n_member(Node.get());
582:   std::vector<int> TileSizes(DimOutNum, 1);
583:   TileSizes[DimOutNum - 3] = MacroKernelParams.Mc;
584:   TileSizes[DimOutNum - 2] = MacroKernelParams.Nc;
585:   TileSizes[DimOutNum - 1] = MacroKernelParams.Kc;
586:   Node = tileNode(Node, "1st level tiling", TileSizes, 1);
587:   Node = Node.parent().parent();
588:   Node = permuteBandNodeDimensions(Node, DimOutNum - 2, DimOutNum - 1);
589:   Node = permuteBandNodeDimensions(Node, DimOutNum - 3, DimOutNum - 1);
```
- **EN**: Checks a condition and selects the appropriate handling path, preserving correctness for special cases.
- **CN**: 这里检查条件并选择合适的处理路径，以保证特殊情况的正确性。

### Lines 591-591
```cpp
591:   return Node.child(0).child(0);
```
- **EN**: Introduces or continues `child`, a routine that carries out one step of this file's analysis or transformation pipeline.
- **CN**: 这里引入或继续 `child`，它是该文件分析或变换流水线中的一个步骤函数。

### Lines 592-592
```cpp
592: }
```
- **EN**: Closes the current block or scope and hands control back to the surrounding context.
- **CN**: 这里关闭当前代码块或作用域，并将控制流交还给外层上下文。

### Lines 594-599
```cpp
594: /// Get the size of the widest type of the matrix multiplication operands
595: /// in bytes, including alignment padding.
596: ///
597: /// @param MMI Parameters of the matrix multiplication operands.
598: /// @return The size of the widest type of the matrix multiplication operands
599: ///         in bytes, including alignment padding.
```
- **EN**: Documents intent, invariants, or algorithmic background for the surrounding code.
- **CN**: 这里说明了周边代码的设计意图、不变量或算法背景。

### Lines 600-607
```cpp
600: static uint64_t getMatMulAlignTypeSize(const MatMulInfoTy &MMI) {
601:   auto *S = MMI.A->getStatement()->getParent();
602:   auto &DL = S->getFunction().getParent()->getDataLayout();
603:   auto ElementSizeA = DL.getTypeAllocSize(MMI.A->getElementType());
604:   auto ElementSizeB = DL.getTypeAllocSize(MMI.B->getElementType());
605:   auto ElementSizeC = DL.getTypeAllocSize(MMI.WriteToC->getElementType());
606:   return std::max({ElementSizeA, ElementSizeB, ElementSizeC});
607: }
```
- **EN**: Introduces or continues `getMatMulAlignTypeSize`, a routine that carries out one step of this file's analysis or transformation pipeline.
- **CN**: 这里引入或继续 `getMatMulAlignTypeSize`，它是该文件分析或变换流水线中的一个步骤函数。

### Lines 609-614
```cpp
609: /// Get the size of the widest type of the matrix multiplication operands
610: /// in bits.
611: ///
612: /// @param MMI Parameters of the matrix multiplication operands.
613: /// @return The size of the widest type of the matrix multiplication operands
614: ///         in bits.
```
- **EN**: Documents intent, invariants, or algorithmic background for the surrounding code.
- **CN**: 这里说明了周边代码的设计意图、不变量或算法背景。

### Lines 615-622
```cpp
615: static uint64_t getMatMulTypeSize(const MatMulInfoTy &MMI) {
616:   auto *S = MMI.A->getStatement()->getParent();
617:   auto &DL = S->getFunction().getParent()->getDataLayout();
618:   auto ElementSizeA = DL.getTypeSizeInBits(MMI.A->getElementType());
619:   auto ElementSizeB = DL.getTypeSizeInBits(MMI.B->getElementType());
620:   auto ElementSizeC = DL.getTypeSizeInBits(MMI.WriteToC->getElementType());
621:   return std::max({ElementSizeA, ElementSizeB, ElementSizeC});
622: }
```
- **EN**: Introduces or continues `getMatMulTypeSize`, a routine that carries out one step of this file's analysis or transformation pipeline.
- **CN**: 这里引入或继续 `getMatMulTypeSize`，它是该文件分析或变换流水线中的一个步骤函数。

### Lines 624-635
```cpp
624: /// Get parameters of the BLIS micro kernel.
625: ///
626: /// We choose the Mr and Nr parameters of the micro kernel to be large enough
627: /// such that no stalls caused by the combination of latencies and dependencies
628: /// are introduced during the updates of the resulting matrix of the matrix
629: /// multiplication. However, they should also be as small as possible to
630: /// release more registers for entries of multiplied matrices.
631: ///
632: /// @param TTI Target Transform Info.
633: /// @param MMI Parameters of the matrix multiplication operands.
634: /// @return The structure of type MicroKernelParamsTy.
635: /// @see MicroKernelParamsTy
```
- **EN**: Documents intent, invariants, or algorithmic background for the surrounding code.
- **CN**: 这里说明了周边代码的设计意图、不变量或算法背景。

### Lines 636-638
```cpp
636: static MicroKernelParamsTy getMicroKernelParams(const TargetTransformInfo *TTI,
637:                                                 const MatMulInfoTy &MMI) {
638:   assert(TTI && "The target transform info should be provided.");
```
- **EN**: Introduces or continues `getMicroKernelParams`, a routine that carries out one step of this file's analysis or transformation pipeline.
- **CN**: 这里引入或继续 `getMicroKernelParams`，它是该文件分析或变换流水线中的一个步骤函数。

### Lines 640-641
```cpp
640:   // Nvec - Number of double-precision floating-point numbers that can be hold
641:   // by a vector register. Use 2 by default.
```
- **EN**: Documents intent, invariants, or algorithmic background for the surrounding code.
- **CN**: 这里说明了周边代码的设计意图、不变量或算法背景。

### Lines 642-642
```cpp
642:   long RegisterBitwidth = VectorRegisterBitwidth;
```
- **EN**: Carries supporting computation, data movement, or bookkeeping for the surrounding algorithm.
- **CN**: 这里承担周边算法所需的辅助计算、数据传递或簿记工作。

### Lines 644-646
```cpp
644:   if (RegisterBitwidth == -1)
645:     RegisterBitwidth =
646:         TTI->getRegisterBitWidth(TargetTransformInfo::RGK_FixedWidthVector);
```
- **EN**: Checks a condition and selects the appropriate handling path, preserving correctness for special cases.
- **CN**: 这里检查条件并选择合适的处理路径，以保证特殊情况的正确性。

### Lines 647-647
```cpp
647:   auto ElementSize = getMatMulTypeSize(MMI);
```
- **EN**: Introduces or continues `getMatMulTypeSize`, a routine that carries out one step of this file's analysis or transformation pipeline.
- **CN**: 这里引入或继续 `getMatMulTypeSize`，它是该文件分析或变换流水线中的一个步骤函数。

### Lines 648-649
```cpp
648:   assert(ElementSize > 0 && "The element size of the matrix multiplication "
649:                             "operands should be greater than zero.");
```
- **EN**: Introduces or continues `assert`, a routine that carries out one step of this file's analysis or transformation pipeline.
- **CN**: 这里引入或继续 `assert`，它是该文件分析或变换流水线中的一个步骤函数。

### Lines 650-650
```cpp
650:   auto Nvec = RegisterBitwidth / ElementSize;
```
- **EN**: Carries supporting computation, data movement, or bookkeeping for the surrounding algorithm.
- **CN**: 这里承担周边算法所需的辅助计算、数据传递或簿记工作。

### Lines 651-652
```cpp
651:   if (Nvec == 0)
652:     Nvec = 2;
```
- **EN**: Checks a condition and selects the appropriate handling path, preserving correctness for special cases.
- **CN**: 这里检查条件并选择合适的处理路径，以保证特殊情况的正确性。

### Lines 653-655
```cpp
653:   int Nr = ceil(sqrt((double)(Nvec * LatencyVectorFma * ThroughputVectorFma)) /
654:                 Nvec) *
655:            Nvec;
```
- **EN**: Carries supporting computation, data movement, or bookkeeping for the surrounding algorithm.
- **CN**: 这里承担周边算法所需的辅助计算、数据传递或簿记工作。

### Lines 656-656
```cpp
656:   int Mr = ceil((double)(Nvec * LatencyVectorFma * ThroughputVectorFma / Nr));
```
- **EN**: Introduces or continues `ceil`, a routine that carries out one step of this file's analysis or transformation pipeline.
- **CN**: 这里引入或继续 `ceil`，它是该文件分析或变换流水线中的一个步骤函数。

### Lines 657-657
```cpp
657:   return {Mr, Nr};
```
- **EN**: Returns the computed value or forwards the current result to the caller.
- **CN**: 这里返回计算结果，或将当前结果继续传递给调用方。

### Lines 658-658
```cpp
658: }
```
- **EN**: Closes the current block or scope and hands control back to the surrounding context.
- **CN**: 这里关闭当前代码块或作用域，并将控制流交还给外层上下文。

### Lines 660-662
```cpp
660: /// Determine parameters of the target cache.
661: ///
662: /// @param TTI Target Transform Info.
```
- **EN**: Documents intent, invariants, or algorithmic background for the surrounding code.
- **CN**: 这里说明了周边代码的设计意图、不变量或算法背景。

### Lines 663-694
```cpp
663: static void getTargetCacheParameters(const llvm::TargetTransformInfo *TTI) {
664:   auto L1DCache = llvm::TargetTransformInfo::CacheLevel::L1D;
665:   auto L2DCache = llvm::TargetTransformInfo::CacheLevel::L2D;
666:   if (FirstCacheLevelSize == -1) {
667:     if (TTI->getCacheSize(L1DCache))
668:       FirstCacheLevelSize = TTI->getCacheSize(L1DCache).value();
669:     else
670:       FirstCacheLevelSize = static_cast<int>(FirstCacheLevelDefaultSize);
671:   }
672:   if (SecondCacheLevelSize == -1) {
673:     if (TTI->getCacheSize(L2DCache))
674:       SecondCacheLevelSize = TTI->getCacheSize(L2DCache).value();
675:     else
676:       SecondCacheLevelSize = static_cast<int>(SecondCacheLevelDefaultSize);
677:   }
678:   if (FirstCacheLevelAssociativity == -1) {
679:     if (TTI->getCacheAssociativity(L1DCache))
680:       FirstCacheLevelAssociativity =
681:           TTI->getCacheAssociativity(L1DCache).value();
682:     else
683:       FirstCacheLevelAssociativity =
684:           static_cast<int>(FirstCacheLevelDefaultAssociativity);
685:   }
686:   if (SecondCacheLevelAssociativity == -1) {
687:     if (TTI->getCacheAssociativity(L2DCache))
688:       SecondCacheLevelAssociativity =
689:           TTI->getCacheAssociativity(L2DCache).value();
690:     else
691:       SecondCacheLevelAssociativity =
692:           static_cast<int>(SecondCacheLevelDefaultAssociativity);
693:   }
694: }
```
- **EN**: Checks a condition and selects the appropriate handling path, preserving correctness for special cases.
- **CN**: 这里检查条件并选择合适的处理路径，以保证特殊情况的正确性。

### Lines 696-710
```cpp
696: /// Get parameters of the BLIS macro kernel.
697: ///
698: /// During the computation of matrix multiplication, blocks of partitioned
699: /// matrices are mapped to different layers of the memory hierarchy.
700: /// To optimize data reuse, blocks should be ideally kept in cache between
701: /// iterations. Since parameters of the macro kernel determine sizes of these
702: /// blocks, there are upper and lower bounds on these parameters.
703: ///
704: /// @param TTI Target Transform Info.
705: /// @param MicroKernelParams Parameters of the micro-kernel
706: ///                          to be taken into account.
707: /// @param MMI Parameters of the matrix multiplication operands.
708: /// @return The structure of type MacroKernelParamsTy.
709: /// @see MacroKernelParamsTy
710: /// @see MicroKernelParamsTy
```
- **EN**: Documents intent, invariants, or algorithmic background for the surrounding code.
- **CN**: 这里说明了周边代码的设计意图、不变量或算法背景。

### Lines 711-730
```cpp
711: static MacroKernelParamsTy
712: getMacroKernelParams(const llvm::TargetTransformInfo *TTI,
713:                      const MicroKernelParamsTy &MicroKernelParams,
714:                      const MatMulInfoTy &MMI) {
715:   getTargetCacheParameters(TTI);
716:   // According to www.cs.utexas.edu/users/flame/pubs/TOMS-BLIS-Analytical.pdf,
717:   // it requires information about the first two levels of a cache to determine
718:   // all the parameters of a macro-kernel. It also checks that an associativity
719:   // degree of a cache level is greater than two. Otherwise, another algorithm
720:   // for determination of the parameters should be used.
721:   if (!(MicroKernelParams.Mr > 0 && MicroKernelParams.Nr > 0 &&
722:         FirstCacheLevelSize > 0 && SecondCacheLevelSize > 0 &&
723:         FirstCacheLevelAssociativity > 2 && SecondCacheLevelAssociativity > 2))
724:     return {1, 1, 1};
725:   // The quotient should be greater than zero.
726:   if (PollyPatternMatchingNcQuotient <= 0)
727:     return {1, 1, 1};
728:   int Car = floor(
729:       (FirstCacheLevelAssociativity - 1) /
730:       (1 + static_cast<double>(MicroKernelParams.Nr) / MicroKernelParams.Mr));
```
- **EN**: Checks a condition and selects the appropriate handling path, preserving correctness for special cases.
- **CN**: 这里检查条件并选择合适的处理路径，以保证特殊情况的正确性。

### Lines 732-735
```cpp
732:   // Car can be computed to be zero since it is floor to int.
733:   // On Mac OS, division by 0 does not raise a signal. This causes negative
734:   // tile sizes to be computed. Prevent division by Cac==0 by early returning
735:   // if this happens.
```
- **EN**: Documents intent, invariants, or algorithmic background for the surrounding code.
- **CN**: 这里说明了周边代码的设计意图、不变量或算法背景。

### Lines 736-737
```cpp
736:   if (Car == 0)
737:     return {1, 1, 1};
```
- **EN**: Checks a condition and selects the appropriate handling path, preserving correctness for special cases.
- **CN**: 这里检查条件并选择合适的处理路径，以保证特殊情况的正确性。

### Lines 739-739
```cpp
739:   auto ElementSize = getMatMulAlignTypeSize(MMI);
```
- **EN**: Introduces or continues `getMatMulAlignTypeSize`, a routine that carries out one step of this file's analysis or transformation pipeline.
- **CN**: 这里引入或继续 `getMatMulAlignTypeSize`，它是该文件分析或变换流水线中的一个步骤函数。

### Lines 740-741
```cpp
740:   assert(ElementSize > 0 && "The element size of the matrix multiplication "
741:                             "operands should be greater than zero.");
```
- **EN**: Introduces or continues `assert`, a routine that carries out one step of this file's analysis or transformation pipeline.
- **CN**: 这里引入或继续 `assert`，它是该文件分析或变换流水线中的一个步骤函数。

### Lines 742-743
```cpp
742:   int Kc = (Car * FirstCacheLevelSize) /
743:            (MicroKernelParams.Mr * FirstCacheLevelAssociativity * ElementSize);
```
- **EN**: Carries supporting computation, data movement, or bookkeeping for the surrounding algorithm.
- **CN**: 这里承担周边算法所需的辅助计算、数据传递或簿记工作。

### Lines 744-746
```cpp
744:   double Cac =
745:       static_cast<double>(Kc * ElementSize * SecondCacheLevelAssociativity) /
746:       SecondCacheLevelSize;
```
- **EN**: Carries supporting computation, data movement, or bookkeeping for the surrounding algorithm.
- **CN**: 这里承担周边算法所需的辅助计算、数据传递或簿记工作。

### Lines 747-747
```cpp
747:   int Mc = floor((SecondCacheLevelAssociativity - 2) / Cac);
```
- **EN**: Introduces or continues `floor`, a routine that carries out one step of this file's analysis or transformation pipeline.
- **CN**: 这里引入或继续 `floor`，它是该文件分析或变换流水线中的一个步骤函数。

### Lines 748-748
```cpp
748:   int Nc = PollyPatternMatchingNcQuotient * MicroKernelParams.Nr;
```
- **EN**: Carries supporting computation, data movement, or bookkeeping for the surrounding algorithm.
- **CN**: 这里承担周边算法所需的辅助计算、数据传递或簿记工作。

### Lines 750-751
```cpp
750:   assert(Mc > 0 && Nc > 0 && Kc > 0 &&
751:          "Matrix block sizes should be  greater than zero");
```
- **EN**: Introduces or continues `assert`, a routine that carries out one step of this file's analysis or transformation pipeline.
- **CN**: 这里引入或继续 `assert`，它是该文件分析或变换流水线中的一个步骤函数。

### Lines 752-752
```cpp
752:   return {Mc, Nc, Kc};
```
- **EN**: Returns the computed value or forwards the current result to the caller.
- **CN**: 这里返回计算结果，或将当前结果继续传递给调用方。

### Lines 753-753
```cpp
753: }
```
- **EN**: Closes the current block or scope and hands control back to the surrounding context.
- **CN**: 这里关闭当前代码块或作用域，并将控制流交还给外层上下文。

### Lines 755-780
```cpp
755: /// Create an access relation that is specific to
756: ///        the matrix multiplication pattern.
757: ///
758: /// Create an access relation of the following form:
759: /// [O0, O1, O2, O3, O4, O5, O6, O7, O8] -> [OI, O5, OJ]
760: /// where I is @p FirstDim, J is @p SecondDim.
761: ///
762: /// It can be used, for example, to create relations that helps to consequently
763: /// access elements of operands of a matrix multiplication after creation of
764: /// the BLIS micro and macro kernels.
765: ///
766: /// @see ScheduleTreeOptimizer::createMicroKernel
767: /// @see ScheduleTreeOptimizer::createMacroKernel
768: ///
769: /// Subsequently, the described access relation is applied to the range of
770: /// @p MapOldIndVar, that is used to map original induction variables to
771: /// the ones, which are produced by schedule transformations. It helps to
772: /// define relations using a new space and, at the same time, keep them
773: /// in the original one.
774: ///
775: /// @param MapOldIndVar The relation, which maps original induction variables
776: ///                     to the ones, which are produced by schedule
777: ///                     transformations.
778: /// @param FirstDim, SecondDim The input dimensions that are used to define
779: ///        the specified access relation.
780: /// @return The specified access relation.
```
- **EN**: Documents intent, invariants, or algorithmic background for the surrounding code.
- **CN**: 这里说明了周边代码的设计意图、不变量或算法背景。

### Lines 781-789
```cpp
781: static isl::map getMatMulAccRel(isl::map MapOldIndVar, unsigned FirstDim,
782:                                 unsigned SecondDim) {
783:   auto AccessRelSpace = isl::space(MapOldIndVar.ctx(), 0, 9, 3);
784:   auto AccessRel = isl::map::universe(AccessRelSpace);
785:   AccessRel = AccessRel.equate(isl::dim::in, FirstDim, isl::dim::out, 0);
786:   AccessRel = AccessRel.equate(isl::dim::in, 5, isl::dim::out, 1);
787:   AccessRel = AccessRel.equate(isl::dim::in, SecondDim, isl::dim::out, 2);
788:   return MapOldIndVar.apply_range(AccessRel);
789: }
```
- **EN**: Introduces or continues `getMatMulAccRel`, a routine that carries out one step of this file's analysis or transformation pipeline.
- **CN**: 这里引入或继续 `getMatMulAccRel`，它是该文件分析或变换流水线中的一个步骤函数。

### Lines 791-796
```cpp
791: static isl::schedule_node createExtensionNode(isl::schedule_node Node,
792:                                               isl::map ExtensionMap) {
793:   auto Extension = isl::union_map(ExtensionMap);
794:   auto NewNode = isl::schedule_node::from_extension(Extension);
795:   return Node.graft_before(NewNode);
796: }
```
- **EN**: Introduces or continues `createExtensionNode`, a routine that carries out one step of this file's analysis or transformation pipeline.
- **CN**: 这里引入或继续 `createExtensionNode`，它是该文件分析或变换流水线中的一个步骤函数。

### Lines 798-804
```cpp
798: static isl::schedule_node optimizePackedB(isl::schedule_node Node,
799:                                           ScopStmt *Stmt, isl::map MapOldIndVar,
800:                                           MicroKernelParamsTy MicroParams,
801:                                           MacroKernelParamsTy MacroParams,
802:                                           MatMulInfoTy &MMI) {
803:   Scop *S = Stmt->getParent();
804:   isl::set Domain = Stmt->getDomain();
```
- **EN**: Introduces or continues `optimizePackedB`, a routine that carries out one step of this file's analysis or transformation pipeline.
- **CN**: 这里引入或继续 `optimizePackedB`，它是该文件分析或变换流水线中的一个步骤函数。

### Lines 806-806
```cpp
806:   // Create packed array.
```
- **EN**: Documents intent, invariants, or algorithmic background for the surrounding code.
- **CN**: 这里说明了周边代码的设计意图、不变量或算法背景。

### Lines 807-807
```cpp
807:   unsigned FirstDimSize = MacroParams.Nc / MicroParams.Nr;
```
- **EN**: Carries supporting computation, data movement, or bookkeeping for the surrounding algorithm.
- **CN**: 这里承担周边算法所需的辅助计算、数据传递或簿记工作。

### Lines 808-808
```cpp
808:   unsigned SecondDimSize = MacroParams.Kc;
```
- **EN**: Carries supporting computation, data movement, or bookkeeping for the surrounding algorithm.
- **CN**: 这里承担周边算法所需的辅助计算、数据传递或簿记工作。

### Lines 809-809
```cpp
809:   unsigned ThirdDimSize = MicroParams.Nr;
```
- **EN**: Carries supporting computation, data movement, or bookkeeping for the surrounding algorithm.
- **CN**: 这里承担周边算法所需的辅助计算、数据传递或簿记工作。

### Lines 810-812
```cpp
810:   ScopArrayInfo *PackedB =
811:       S->createScopArrayInfo(MMI.B->getElementType(), "Packed_B",
812:                              {FirstDimSize, SecondDimSize, ThirdDimSize});
```
- **EN**: Carries supporting computation, data movement, or bookkeeping for the surrounding algorithm.
- **CN**: 这里承担周边算法所需的辅助计算、数据传递或簿记工作。

### Lines 814-814
```cpp
814:   // Compute the access relation for copying from B to PackedB.
```
- **EN**: Documents intent, invariants, or algorithmic background for the surrounding code.
- **CN**: 这里说明了周边代码的设计意图、不变量或算法背景。

### Lines 815-815
```cpp
815:   isl::map AccRelB = MMI.B->getLatestAccessRelation();
```
- **EN**: Introduces or continues `getLatestAccessRelation`, a routine that carries out one step of this file's analysis or transformation pipeline.
- **CN**: 这里引入或继续 `getLatestAccessRelation`，它是该文件分析或变换流水线中的一个步骤函数。

### Lines 816-816
```cpp
816:   isl::map AccRelPackedB = getMatMulAccRel(MapOldIndVar, 3, 7);
```
- **EN**: Introduces or continues `getMatMulAccRel`, a routine that carries out one step of this file's analysis or transformation pipeline.
- **CN**: 这里引入或继续 `getMatMulAccRel`，它是该文件分析或变换流水线中的一个步骤函数。

### Lines 817-818
```cpp
817:   AccRelPackedB =
818:       AccRelPackedB.set_tuple_id(isl::dim::out, PackedB->getBasePtrId());
```
- **EN**: Introduces or continues `set_tuple_id`, a routine that carries out one step of this file's analysis or transformation pipeline.
- **CN**: 这里引入或继续 `set_tuple_id`，它是该文件分析或变换流水线中的一个步骤函数。

### Lines 820-820
```cpp
820:   // Create the copy statement and redirect access.
```
- **EN**: Documents intent, invariants, or algorithmic background for the surrounding code.
- **CN**: 这里说明了周边代码的设计意图、不变量或算法背景。

### Lines 821-821
```cpp
821:   ScopStmt *CopyStmt = S->addScopStmt(AccRelB, AccRelPackedB, Domain);
```
- **EN**: Introduces or continues `addScopStmt`, a routine that carries out one step of this file's analysis or transformation pipeline.
- **CN**: 这里引入或继续 `addScopStmt`，它是该文件分析或变换流水线中的一个步骤函数。

### Lines 822-822
```cpp
822:   MMI.B->setNewAccessRelation(AccRelPackedB);
```
- **EN**: Introduces or continues `setNewAccessRelation`, a routine that carries out one step of this file's analysis or transformation pipeline.
- **CN**: 这里引入或继续 `setNewAccessRelation`，它是该文件分析或变换流水线中的一个步骤函数。

### Lines 824-824
```cpp
824:   unsigned Dim = unsignedFromIslSize(MapOldIndVar.range_tuple_dim());
```
- **EN**: Introduces or continues `unsignedFromIslSize`, a routine that carries out one step of this file's analysis or transformation pipeline.
- **CN**: 这里引入或继续 `unsignedFromIslSize`，它是该文件分析或变换流水线中的一个步骤函数。

### Lines 825-825
```cpp
825:   assert(Dim >= 2);
```
- **EN**: Introduces or continues `assert`, a routine that carries out one step of this file's analysis or transformation pipeline.
- **CN**: 这里引入或继续 `assert`，它是该文件分析或变换流水线中的一个步骤函数。

### Lines 826-826
```cpp
826:   // Insert into the schedule tree.
```
- **EN**: Documents intent, invariants, or algorithmic background for the surrounding code.
- **CN**: 这里说明了周边代码的设计意图、不变量或算法背景。

### Lines 827-827
```cpp
827:   isl::map ExtMap = MapOldIndVar.project_out(isl::dim::out, 2, Dim - 2);
```
- **EN**: Introduces or continues `project_out`, a routine that carries out one step of this file's analysis or transformation pipeline.
- **CN**: 这里引入或继续 `project_out`，它是该文件分析或变换流水线中的一个步骤函数。

### Lines 828-828
```cpp
828:   ExtMap = ExtMap.reverse();
```
- **EN**: Introduces or continues `reverse`, a routine that carries out one step of this file's analysis or transformation pipeline.
- **CN**: 这里引入或继续 `reverse`，它是该文件分析或变换流水线中的一个步骤函数。

### Lines 829-829
```cpp
829:   ExtMap = ExtMap.fix_si(isl::dim::out, MMI.i, 0);
```
- **EN**: Introduces or continues `fix_si`, a routine that carries out one step of this file's analysis or transformation pipeline.
- **CN**: 这里引入或继续 `fix_si`，它是该文件分析或变换流水线中的一个步骤函数。

### Lines 830-830
```cpp
830:   ExtMap = ExtMap.intersect_range(Domain);
```
- **EN**: Introduces or continues `intersect_range`, a routine that carries out one step of this file's analysis or transformation pipeline.
- **CN**: 这里引入或继续 `intersect_range`，它是该文件分析或变换流水线中的一个步骤函数。

### Lines 831-831
```cpp
831:   ExtMap = ExtMap.set_tuple_id(isl::dim::out, CopyStmt->getDomainId());
```
- **EN**: Introduces or continues `set_tuple_id`, a routine that carries out one step of this file's analysis or transformation pipeline.
- **CN**: 这里引入或继续 `set_tuple_id`，它是该文件分析或变换流水线中的一个步骤函数。

### Lines 832-832
```cpp
832:   return createExtensionNode(Node, ExtMap);
```
- **EN**: Introduces or continues `createExtensionNode`, a routine that carries out one step of this file's analysis or transformation pipeline.
- **CN**: 这里引入或继续 `createExtensionNode`，它是该文件分析或变换流水线中的一个步骤函数。

### Lines 833-833
```cpp
833: }
```
- **EN**: Closes the current block or scope and hands control back to the surrounding context.
- **CN**: 这里关闭当前代码块或作用域，并将控制流交还给外层上下文。

### Lines 835-843
```cpp
835: static isl::schedule_node optimizePackedA(isl::schedule_node Node, ScopStmt *,
836:                                           isl::map MapOldIndVar,
837:                                           MicroKernelParamsTy MicroParams,
838:                                           MacroKernelParamsTy MacroParams,
839:                                           MatMulInfoTy &MMI) {
840:   isl::id InputDimsId = MapOldIndVar.get_tuple_id(isl::dim::in);
841:   ScopStmt *Stmt = static_cast<ScopStmt *>(InputDimsId.get_user());
842:   isl::set Domain = Stmt->getDomain();
843:   isl::id DomainId = Domain.get_tuple_id();
```
- **EN**: Introduces or continues `optimizePackedA`, a routine that carries out one step of this file's analysis or transformation pipeline.
- **CN**: 这里引入或继续 `optimizePackedA`，它是该文件分析或变换流水线中的一个步骤函数。

### Lines 845-845
```cpp
845:   // Create the packed array.
```
- **EN**: Documents intent, invariants, or algorithmic background for the surrounding code.
- **CN**: 这里说明了周边代码的设计意图、不变量或算法背景。

### Lines 846-846
```cpp
846:   unsigned FirstDimSize = MacroParams.Mc / MicroParams.Mr;
```
- **EN**: Carries supporting computation, data movement, or bookkeeping for the surrounding algorithm.
- **CN**: 这里承担周边算法所需的辅助计算、数据传递或簿记工作。

### Lines 847-847
```cpp
847:   unsigned SecondDimSize = MacroParams.Kc;
```
- **EN**: Carries supporting computation, data movement, or bookkeeping for the surrounding algorithm.
- **CN**: 这里承担周边算法所需的辅助计算、数据传递或簿记工作。

### Lines 848-848
```cpp
848:   unsigned ThirdDimSize = MicroParams.Mr;
```
- **EN**: Carries supporting computation, data movement, or bookkeeping for the surrounding algorithm.
- **CN**: 这里承担周边算法所需的辅助计算、数据传递或簿记工作。

### Lines 849-851
```cpp
849:   ScopArrayInfo *PackedA = Stmt->getParent()->createScopArrayInfo(
850:       MMI.A->getElementType(), "Packed_A",
851:       {FirstDimSize, SecondDimSize, ThirdDimSize});
```
- **EN**: Carries supporting computation, data movement, or bookkeeping for the surrounding algorithm.
- **CN**: 这里承担周边算法所需的辅助计算、数据传递或簿记工作。

### Lines 853-853
```cpp
853:   // Compute the access relation for copying from A to PackedA.
```
- **EN**: Documents intent, invariants, or algorithmic background for the surrounding code.
- **CN**: 这里说明了周边代码的设计意图、不变量或算法背景。

### Lines 854-854
```cpp
854:   isl::map AccRelA = MMI.A->getLatestAccessRelation();
```
- **EN**: Introduces or continues `getLatestAccessRelation`, a routine that carries out one step of this file's analysis or transformation pipeline.
- **CN**: 这里引入或继续 `getLatestAccessRelation`，它是该文件分析或变换流水线中的一个步骤函数。

### Lines 855-855
```cpp
855:   isl::map AccRelPackedA = getMatMulAccRel(MapOldIndVar, 4, 6);
```
- **EN**: Introduces or continues `getMatMulAccRel`, a routine that carries out one step of this file's analysis or transformation pipeline.
- **CN**: 这里引入或继续 `getMatMulAccRel`，它是该文件分析或变换流水线中的一个步骤函数。

### Lines 856-857
```cpp
856:   AccRelPackedA =
857:       AccRelPackedA.set_tuple_id(isl::dim::out, PackedA->getBasePtrId());
```
- **EN**: Introduces or continues `set_tuple_id`, a routine that carries out one step of this file's analysis or transformation pipeline.
- **CN**: 这里引入或继续 `set_tuple_id`，它是该文件分析或变换流水线中的一个步骤函数。

### Lines 858-858
```cpp
858:   // { MemrefA[] -> PackedA[] }
```
- **EN**: Documents intent, invariants, or algorithmic background for the surrounding code.
- **CN**: 这里说明了周边代码的设计意图、不变量或算法背景。

### Lines 859-859
```cpp
859:   isl::map PackedATranslator = AccRelPackedA.apply_domain(AccRelA);
```
- **EN**: Introduces or continues `apply_domain`, a routine that carries out one step of this file's analysis or transformation pipeline.
- **CN**: 这里引入或继续 `apply_domain`，它是该文件分析或变换流水线中的一个步骤函数。

### Lines 861-865
```cpp
861:   // Compute the domain for the copy statement.
862:   // Construct the copy statement domain out of the 3 outermost scatter
863:   // dimensions (to match the 3 band nodes surrounding the extension node) and
864:   // the array elements to copy (one statement instance per array element).
865:   // { Scatter[] }
```
- **EN**: Documents intent, invariants, or algorithmic background for the surrounding code.
- **CN**: 这里说明了周边代码的设计意图、不变量或算法背景。

### Lines 866-866
```cpp
866:   isl::set ScatterDomain = MapOldIndVar.intersect_domain(Domain).range();
```
- **EN**: Introduces or continues `intersect_domain`, a routine that carries out one step of this file's analysis or transformation pipeline.
- **CN**: 这里引入或继续 `intersect_domain`，它是该文件分析或变换流水线中的一个步骤函数。

### Lines 867-867
```cpp
867:   // { Scatter[] -> OutermostScatter[] }
```
- **EN**: Documents intent, invariants, or algorithmic background for the surrounding code.
- **CN**: 这里说明了周边代码的设计意图、不变量或算法背景。

### Lines 868-869
```cpp
868:   isl::map OuterDomainMap =
869:       makeIdentityMap(ScatterDomain, true).project_out(isl::dim::out, 3, 6);
```
- **EN**: Introduces or continues `makeIdentityMap`, a routine that carries out one step of this file's analysis or transformation pipeline.
- **CN**: 这里引入或继续 `makeIdentityMap`，它是该文件分析或变换流水线中的一个步骤函数。

### Lines 870-870
```cpp
870:   // { Scatter[] -> MemrefA[] }
```
- **EN**: Documents intent, invariants, or algorithmic background for the surrounding code.
- **CN**: 这里说明了周边代码的设计意图、不变量或算法背景。

### Lines 871-871
```cpp
871:   isl::map CopyFrom = MapOldIndVar.reverse().apply_range(AccRelA);
```
- **EN**: Introduces or continues `reverse`, a routine that carries out one step of this file's analysis or transformation pipeline.
- **CN**: 这里引入或继续 `reverse`，它是该文件分析或变换流水线中的一个步骤函数。

### Lines 872-872
```cpp
872:   // { Scatter[] -> CopyStmt[] }
```
- **EN**: Documents intent, invariants, or algorithmic background for the surrounding code.
- **CN**: 这里说明了周边代码的设计意图、不变量或算法背景。

### Lines 873-873
```cpp
873:   isl::map DomainTranslator = OuterDomainMap.range_product(CopyFrom);
```
- **EN**: Introduces or continues `range_product`, a routine that carries out one step of this file's analysis or transformation pipeline.
- **CN**: 这里引入或继续 `range_product`，它是该文件分析或变换流水线中的一个步骤函数。

### Lines 874-874
```cpp
874:   // { CopyStmt[] }
```
- **EN**: Documents intent, invariants, or algorithmic background for the surrounding code.
- **CN**: 这里说明了周边代码的设计意图、不变量或算法背景。

### Lines 875-875
```cpp
875:   isl::set CopyDomain = DomainTranslator.range();
```
- **EN**: Introduces or continues `range`, a routine that carries out one step of this file's analysis or transformation pipeline.
- **CN**: 这里引入或继续 `range`，它是该文件分析或变换流水线中的一个步骤函数。

### Lines 877-878
```cpp
877:   // Translate the access relations to the new domain.
878:   // { CopyStmt[] -> MemrefA[] }
```
- **EN**: Documents intent, invariants, or algorithmic background for the surrounding code.
- **CN**: 这里说明了周边代码的设计意图、不变量或算法背景。

### Lines 879-879
```cpp
879:   CopyFrom = CopyFrom.apply_domain(DomainTranslator);
```
- **EN**: Introduces or continues `apply_domain`, a routine that carries out one step of this file's analysis or transformation pipeline.
- **CN**: 这里引入或继续 `apply_domain`，它是该文件分析或变换流水线中的一个步骤函数。

### Lines 880-880
```cpp
880:   // { CopyStmt[] -> PackedA[] }
```
- **EN**: Documents intent, invariants, or algorithmic background for the surrounding code.
- **CN**: 这里说明了周边代码的设计意图、不变量或算法背景。

### Lines 881-881
```cpp
881:   isl::map CopyTo = CopyFrom.apply_range(PackedATranslator);
```
- **EN**: Introduces or continues `apply_range`, a routine that carries out one step of this file's analysis or transformation pipeline.
- **CN**: 这里引入或继续 `apply_range`，它是该文件分析或变换流水线中的一个步骤函数。

### Lines 883-883
```cpp
883:   // Create the copy statement and redirect access.
```
- **EN**: Documents intent, invariants, or algorithmic background for the surrounding code.
- **CN**: 这里说明了周边代码的设计意图、不变量或算法背景。

### Lines 884-885
```cpp
884:   ScopStmt *CopyStmt =
885:       Stmt->getParent()->addScopStmt(CopyFrom, CopyTo, CopyDomain);
```
- **EN**: Introduces or continues `getParent`, a routine that carries out one step of this file's analysis or transformation pipeline.
- **CN**: 这里引入或继续 `getParent`，它是该文件分析或变换流水线中的一个步骤函数。

### Lines 886-886
```cpp
886:   MMI.A->setNewAccessRelation(AccRelPackedA);
```
- **EN**: Introduces or continues `setNewAccessRelation`, a routine that carries out one step of this file's analysis or transformation pipeline.
- **CN**: 这里引入或继续 `setNewAccessRelation`，它是该文件分析或变换流水线中的一个步骤函数。

### Lines 888-889
```cpp
888:   // Insert into the schedule tree.
889:   // { Scatter[] -> CopyStmt[] }
```
- **EN**: Documents intent, invariants, or algorithmic background for the surrounding code.
- **CN**: 这里说明了周边代码的设计意图、不变量或算法背景。

### Lines 890-890
```cpp
890:   isl::map ExtScatterCopy = makeIdentityMap(CopyStmt->getDomain(), true);
```
- **EN**: Introduces or continues `makeIdentityMap`, a routine that carries out one step of this file's analysis or transformation pipeline.
- **CN**: 这里引入或继续 `makeIdentityMap`，它是该文件分析或变换流水线中的一个步骤函数。

### Lines 891-891
```cpp
891:   ExtScatterCopy = ExtScatterCopy.project_out(isl::dim::in, 3, 2);
```
- **EN**: Introduces or continues `project_out`, a routine that carries out one step of this file's analysis or transformation pipeline.
- **CN**: 这里引入或继续 `project_out`，它是该文件分析或变换流水线中的一个步骤函数。

### Lines 892-892
```cpp
892:   return createExtensionNode(Node, ExtScatterCopy);
```
- **EN**: Introduces or continues `createExtensionNode`, a routine that carries out one step of this file's analysis or transformation pipeline.
- **CN**: 这里引入或继续 `createExtensionNode`，它是该文件分析或变换流水线中的一个步骤函数。

### Lines 893-893
```cpp
893: }
```
- **EN**: Closes the current block or scope and hands control back to the surrounding context.
- **CN**: 这里关闭当前代码块或作用域，并将控制流交还给外层上下文。

### Lines 895-926
```cpp
895: /// Apply the packing transformation.
896: ///
897: /// The packing transformation can be described as a data-layout
898: /// transformation that requires to introduce a new array, copy data
899: /// to the array, and change memory access locations to reference the array.
900: /// It can be used to ensure that elements of the new array are read in-stride
901: /// access, aligned to cache lines boundaries, and preloaded into certain cache
902: /// levels.
903: ///
904: /// As an example let us consider the packing of the array A that would help
905: /// to read its elements with in-stride access. An access to the array A
906: /// is represented by an access relation that has the form
907: /// S[i, j, k] -> A[i, k]. The scheduling function of the SCoP statement S has
908: /// the form S[i,j, k] -> [floor((j mod Nc) / Nr), floor((i mod Mc) / Mr),
909: /// k mod Kc, j mod Nr, i mod Mr].
910: ///
911: /// To ensure that elements of the array A are read in-stride access, we add
912: /// a new array Packed_A[Mc/Mr][Kc][Mr] to the SCoP, using
913: /// Scop::createScopArrayInfo, change the access relation
914: /// S[i, j, k] -> A[i, k] to
915: /// S[i, j, k] -> Packed_A[floor((i mod Mc) / Mr), k mod Kc, i mod Mr], using
916: /// MemoryAccess::setNewAccessRelation, and copy the data to the array, using
917: /// the copy statement created by Scop::addScopStmt.
918: ///
919: /// @param Node The schedule node to be optimized.
920: /// @param MapOldIndVar The relation, which maps original induction variables
921: ///                     to the ones, which are produced by schedule
922: ///                     transformations.
923: /// @param MicroParams, MacroParams Parameters of the BLIS kernel
924: ///                                 to be taken into account.
925: /// @param MMI Parameters of the matrix multiplication operands.
926: /// @return The optimized schedule node.
```
- **EN**: Documents intent, invariants, or algorithmic background for the surrounding code.
- **CN**: 这里说明了周边代码的设计意图、不变量或算法背景。

### Lines 927-933
```cpp
927: static isl::schedule_node
928: optimizeDataLayoutMatrMulPattern(isl::schedule_node Node, isl::map MapOldIndVar,
929:                                  MicroKernelParamsTy MicroParams,
930:                                  MacroKernelParamsTy MacroParams,
931:                                  MatMulInfoTy &MMI) {
932:   isl::id InputDimsId = MapOldIndVar.get_tuple_id(isl::dim::in);
933:   ScopStmt *Stmt = static_cast<ScopStmt *>(InputDimsId.get_user());
```
- **EN**: Introduces or continues `optimizeDataLayoutMatrMulPattern`, a routine that carries out one step of this file's analysis or transformation pipeline.
- **CN**: 这里引入或继续 `optimizeDataLayoutMatrMulPattern`，它是该文件分析或变换流水线中的一个步骤函数。

### Lines 935-935
```cpp
935:   Node = Node.parent().parent().parent().parent().parent().parent();
```
- **EN**: Introduces or continues `parent`, a routine that carries out one step of this file's analysis or transformation pipeline.
- **CN**: 这里引入或继续 `parent`，它是该文件分析或变换流水线中的一个步骤函数。

### Lines 936-936
```cpp
936:   Node = isl::manage(isl_schedule_node_band_split(Node.release(), 2));
```
- **EN**: Introduces or continues `isl::manage`, a routine that carries out one step of this file's analysis or transformation pipeline.
- **CN**: 这里引入或继续 `isl::manage`，它是该文件分析或变换流水线中的一个步骤函数。

### Lines 938-938
```cpp
938:   Node = Node.child(0);
```
- **EN**: Introduces or continues `child`, a routine that carries out one step of this file's analysis or transformation pipeline.
- **CN**: 这里引入或继续 `child`，它是该文件分析或变换流水线中的一个步骤函数。

### Lines 939-940
```cpp
939:   Node =
940:       optimizePackedB(Node, Stmt, MapOldIndVar, MicroParams, MacroParams, MMI);
```
- **EN**: Introduces or continues `optimizePackedB`, a routine that carries out one step of this file's analysis or transformation pipeline.
- **CN**: 这里引入或继续 `optimizePackedB`，它是该文件分析或变换流水线中的一个步骤函数。

### Lines 942-942
```cpp
942:   Node = Node.child(0);
```
- **EN**: Introduces or continues `child`, a routine that carries out one step of this file's analysis or transformation pipeline.
- **CN**: 这里引入或继续 `child`，它是该文件分析或变换流水线中的一个步骤函数。

### Lines 943-944
```cpp
943:   Node =
944:       optimizePackedA(Node, Stmt, MapOldIndVar, MicroParams, MacroParams, MMI);
```
- **EN**: Introduces or continues `optimizePackedA`, a routine that carries out one step of this file's analysis or transformation pipeline.
- **CN**: 这里引入或继续 `optimizePackedA`，它是该文件分析或变换流水线中的一个步骤函数。

### Lines 946-946
```cpp
946:   return Node.child(0).child(0).child(0).child(0).child(0);
```
- **EN**: Introduces or continues `child`, a routine that carries out one step of this file's analysis or transformation pipeline.
- **CN**: 这里引入或继续 `child`，它是该文件分析或变换流水线中的一个步骤函数。

### Lines 947-947
```cpp
947: }
```
- **EN**: Closes the current block or scope and hands control back to the surrounding context.
- **CN**: 这里关闭当前代码块或作用域，并将控制流交还给外层上下文。

### Lines 949-960
```cpp
949: /// Get a relation mapping induction variables produced by schedule
950: /// transformations to the original ones.
951: ///
952: /// @param Node The schedule node produced as the result of creation
953: ///        of the BLIS kernels.
954: /// @param MicroKernelParams, MacroKernelParams Parameters of the BLIS kernel
955: ///                                             to be taken into account.
956: /// @return  The relation mapping original induction variables to the ones
957: ///          produced by schedule transformation.
958: /// @see ScheduleTreeOptimizer::createMicroKernel
959: /// @see ScheduleTreeOptimizer::createMacroKernel
960: /// @see getMacroKernelParams
```
- **EN**: Documents intent, invariants, or algorithmic background for the surrounding code.
- **CN**: 这里说明了周边代码的设计意图、不变量或算法背景。

### Lines 961-972
```cpp
961: static isl::map
962: getInductionVariablesSubstitution(isl::schedule_node Node,
963:                                   MicroKernelParamsTy MicroKernelParams,
964:                                   MacroKernelParamsTy MacroKernelParams) {
965:   auto Child = Node.child(0);
966:   auto UnMapOldIndVar = Child.get_prefix_schedule_union_map();
967:   auto MapOldIndVar = isl::map::from_union_map(UnMapOldIndVar);
968:   unsigned Dim = unsignedFromIslSize(MapOldIndVar.range_tuple_dim());
969:   if (Dim > 9u)
970:     return MapOldIndVar.project_out(isl::dim::out, 0, Dim - 9);
971:   return MapOldIndVar;
972: }
```
- **EN**: Checks a condition and selects the appropriate handling path, preserving correctness for special cases.
- **CN**: 这里检查条件并选择合适的处理路径，以保证特殊情况的正确性。

### Lines 974-987
```cpp
974: /// Isolate a set of partial tile prefixes and unroll the isolated part.
975: ///
976: /// The set should ensure that it contains only partial tile prefixes that have
977: /// exactly Mr x Nr iterations of the two innermost loops produced by
978: /// the optimization of the matrix multiplication. Mr and Nr are parameters of
979: /// the micro-kernel.
980: ///
981: /// In case of parametric bounds, this helps to auto-vectorize the unrolled
982: /// innermost loops, using the SLP vectorizer.
983: ///
984: /// @param Node              The schedule node to be modified.
985: /// @param MicroKernelParams Parameters of the micro-kernel
986: ///                          to be taken into account.
987: /// @return The modified isl_schedule_node.
```
- **EN**: Documents intent, invariants, or algorithmic background for the surrounding code.
- **CN**: 这里说明了周边代码的设计意图、不变量或算法背景。

### Lines 988-998
```cpp
988: static isl::schedule_node
989: isolateAndUnrollMatMulInnerLoops(isl::schedule_node Node,
990:                                  MicroKernelParamsTy MicroKernelParams) {
991:   isl::schedule_node Child = Node.child(0);
992:   isl::union_map UnMapOldIndVar = Child.get_prefix_schedule_relation();
993:   isl::set Prefix = isl::map::from_union_map(UnMapOldIndVar).range();
994:   unsigned Dims = unsignedFromIslSize(Prefix.tuple_dim());
995:   assert(Dims >= 1);
996:   Prefix = Prefix.project_out(isl::dim::set, Dims - 1, 1);
997:   Prefix = getPartialTilePrefixes(Prefix, MicroKernelParams.Nr);
998:   Prefix = getPartialTilePrefixes(Prefix, MicroKernelParams.Mr);
```
- **EN**: Introduces or continues `isolateAndUnrollMatMulInnerLoops`, a routine that carries out one step of this file's analysis or transformation pipeline.
- **CN**: 这里引入或继续 `isolateAndUnrollMatMulInnerLoops`，它是该文件分析或变换流水线中的一个步骤函数。

### Lines 1000-1001
```cpp
1000:   isl::union_set IsolateOption =
1001:       getIsolateOptions(Prefix.add_dims(isl::dim::set, 3), 3);
```
- **EN**: Introduces or continues `getIsolateOptions`, a routine that carries out one step of this file's analysis or transformation pipeline.
- **CN**: 这里引入或继续 `getIsolateOptions`，它是该文件分析或变换流水线中的一个步骤函数。

### Lines 1002-1002
```cpp
1002:   isl::ctx Ctx = Node.ctx();
```
- **EN**: Introduces or continues `ctx`, a routine that carries out one step of this file's analysis or transformation pipeline.
- **CN**: 这里引入或继续 `ctx`，它是该文件分析或变换流水线中的一个步骤函数。

### Lines 1003-1003
```cpp
1003:   auto Options = IsolateOption.unite(getDimOptions(Ctx, "unroll"));
```
- **EN**: Introduces or continues `unite`, a routine that carries out one step of this file's analysis or transformation pipeline.
- **CN**: 这里引入或继续 `unite`，它是该文件分析或变换流水线中的一个步骤函数。

### Lines 1004-1004
```cpp
1004:   Options = Options.unite(getUnrollIsolatedSetOptions(Ctx));
```
- **EN**: Introduces or continues `unite`, a routine that carries out one step of this file's analysis or transformation pipeline.
- **CN**: 这里引入或继续 `unite`，它是该文件分析或变换流水线中的一个步骤函数。

### Lines 1005-1005
```cpp
1005:   Node = Node.as<isl::schedule_node_band>().set_ast_build_options(Options);
```
- **EN**: Introduces or continues `as<isl::schedule_node_band>`, a routine that carries out one step of this file's analysis or transformation pipeline.
- **CN**: 这里引入或继续 `as<isl::schedule_node_band>`，它是该文件分析或变换流水线中的一个步骤函数。

### Lines 1006-1006
```cpp
1006:   Node = Node.parent().parent().parent();
```
- **EN**: Introduces or continues `parent`, a routine that carries out one step of this file's analysis or transformation pipeline.
- **CN**: 这里引入或继续 `parent`，它是该文件分析或变换流水线中的一个步骤函数。

### Lines 1007-1007
```cpp
1007:   IsolateOption = getIsolateOptions(Prefix, 3);
```
- **EN**: Introduces or continues `getIsolateOptions`, a routine that carries out one step of this file's analysis or transformation pipeline.
- **CN**: 这里引入或继续 `getIsolateOptions`，它是该文件分析或变换流水线中的一个步骤函数。

### Lines 1008-1008
```cpp
1008:   Options = IsolateOption.unite(getDimOptions(Ctx, "separate"));
```
- **EN**: Introduces or continues `unite`, a routine that carries out one step of this file's analysis or transformation pipeline.
- **CN**: 这里引入或继续 `unite`，它是该文件分析或变换流水线中的一个步骤函数。

### Lines 1009-1009
```cpp
1009:   Node = Node.as<isl::schedule_node_band>().set_ast_build_options(Options);
```
- **EN**: Introduces or continues `as<isl::schedule_node_band>`, a routine that carries out one step of this file's analysis or transformation pipeline.
- **CN**: 这里引入或继续 `as<isl::schedule_node_band>`，它是该文件分析或变换流水线中的一个步骤函数。

### Lines 1010-1010
```cpp
1010:   Node = Node.child(0).child(0).child(0);
```
- **EN**: Introduces or continues `child`, a routine that carries out one step of this file's analysis or transformation pipeline.
- **CN**: 这里引入或继续 `child`，它是该文件分析或变换流水线中的一个步骤函数。

### Lines 1011-1011
```cpp
1011:   return Node;
```
- **EN**: Returns the computed value or forwards the current result to the caller.
- **CN**: 这里返回计算结果，或将当前结果继续传递给调用方。

### Lines 1012-1012
```cpp
1012: }
```
- **EN**: Closes the current block or scope and hands control back to the surrounding context.
- **CN**: 这里关闭当前代码块或作用域，并将控制流交还给外层上下文。

### Lines 1014-1017
```cpp
1014: /// Insert "Loop Vectorizer Disabled" mark node.
1015: ///
1016: /// @param Node The child of the mark node to be inserted.
1017: /// @return The modified isl_schedule_node.
```
- **EN**: Documents intent, invariants, or algorithmic background for the surrounding code.
- **CN**: 这里说明了周边代码的设计意图、不变量或算法背景。

### Lines 1018-1021
```cpp
1018: static isl::schedule_node markLoopVectorizerDisabled(isl::schedule_node Node) {
1019:   auto Id = isl::id::alloc(Node.ctx(), "Loop Vectorizer Disabled", nullptr);
1020:   return Node.insert_mark(Id).child(0);
1021: }
```
- **EN**: Introduces or continues `markLoopVectorizerDisabled`, a routine that carries out one step of this file's analysis or transformation pipeline.
- **CN**: 这里引入或继续 `markLoopVectorizerDisabled`，它是该文件分析或变换流水线中的一个步骤函数。

### Lines 1023-1030
```cpp
1023: /// Restore the initial ordering of dimensions of the band node
1024: ///
1025: /// In case the band node represents all the dimensions of the iteration
1026: /// domain, recreate the band node to restore the initial ordering of the
1027: /// dimensions.
1028: ///
1029: /// @param Node The band node to be modified.
1030: /// @return The modified schedule node.
```
- **EN**: Documents intent, invariants, or algorithmic background for the surrounding code.
- **CN**: 这里说明了周边代码的设计意图、不变量或算法背景。

### Lines 1031-1049
```cpp
1031: static isl::schedule_node
1032: getBandNodeWithOriginDimOrder(isl::schedule_node Node) {
1033:   assert(isl_schedule_node_get_type(Node.get()) == isl_schedule_node_band);
1034:   if (isl_schedule_node_get_type(Node.child(0).get()) != isl_schedule_node_leaf)
1035:     return Node;
1036:   auto Domain = Node.get_universe_domain();
1037:   assert(isl_union_set_n_set(Domain.get()) == 1);
1038:   if (Node.get_schedule_depth().release() != 0 ||
1039:       (unsignedFromIslSize(isl::set(Domain).tuple_dim()) !=
1040:        unsignedFromIslSize(Node.as<isl::schedule_node_band>().n_member())))
1041:     return Node;
1042:   Node = isl::manage(isl_schedule_node_delete(Node.copy()));
1043:   auto PartialSchedulePwAff = Domain.identity_union_pw_multi_aff();
1044:   auto PartialScheduleMultiPwAff =
1045:       isl::multi_union_pw_aff(PartialSchedulePwAff);
1046:   PartialScheduleMultiPwAff =
1047:       PartialScheduleMultiPwAff.reset_tuple_id(isl::dim::set);
1048:   return Node.insert_partial_schedule(PartialScheduleMultiPwAff);
1049: }
```
- **EN**: Checks a condition and selects the appropriate handling path, preserving correctness for special cases.
- **CN**: 这里检查条件并选择合适的处理路径，以保证特殊情况的正确性。

### Lines 1051-1081
```cpp
1051: static isl::schedule_node optimizeMatMulPattern(isl::schedule_node Node,
1052:                                                 const TargetTransformInfo *TTI,
1053:                                                 MatMulInfoTy &MMI) {
1054:   assert(TTI && "The target transform info should be provided.");
1055:   int DimOutNum = isl_schedule_node_band_n_member(Node.get());
1056:   assert(DimOutNum > 2 && "In case of the matrix multiplication the loop nest "
1057:                           "and, consequently, the corresponding scheduling "
1058:                           "functions have at least three dimensions.");
1059:   Node = getBandNodeWithOriginDimOrder(Node);
1060:   Node = permuteBandNodeDimensions(Node, MMI.i, DimOutNum - 3);
1061:   int NewJ = MMI.j == DimOutNum - 3 ? MMI.i : MMI.j;
1062:   int NewK = MMI.k == DimOutNum - 3 ? MMI.i : MMI.k;
1063:   Node = permuteBandNodeDimensions(Node, NewJ, DimOutNum - 2);
1064:   NewK = NewK == DimOutNum - 2 ? NewJ : NewK;
1065:   Node = permuteBandNodeDimensions(Node, NewK, DimOutNum - 1);
1066:   auto MicroKernelParams = getMicroKernelParams(TTI, MMI);
1067:   auto MacroKernelParams = getMacroKernelParams(TTI, MicroKernelParams, MMI);
1068:   Node = createMacroKernel(Node, MacroKernelParams);
1069:   Node = createMicroKernel(Node, MicroKernelParams);
1070:   if (MacroKernelParams.Mc == 1 || MacroKernelParams.Nc == 1 ||
1071:       MacroKernelParams.Kc == 1)
1072:     return Node;
1073:   auto MapOldIndVar = getInductionVariablesSubstitution(Node, MicroKernelParams,
1074:                                                         MacroKernelParams);
1075:   if (MapOldIndVar.is_null())
1076:     return Node;
1077:   Node = markLoopVectorizerDisabled(Node.parent()).child(0);
1078:   Node = isolateAndUnrollMatMulInnerLoops(Node, MicroKernelParams);
1079:   return optimizeDataLayoutMatrMulPattern(Node, MapOldIndVar, MicroKernelParams,
1080:                                           MacroKernelParams, MMI);
1081: }
```
- **EN**: Checks a condition and selects the appropriate handling path, preserving correctness for special cases.
- **CN**: 这里检查条件并选择合适的处理路径，以保证特殊情况的正确性。

### Lines 1083-1102
```cpp
1083: /// Check if this node contains a partial schedule that could
1084: ///        probably be optimized with analytical modeling.
1085: ///
1086: /// isMatrMultPattern tries to determine whether the following conditions
1087: /// are true:
1088: /// 1. the partial schedule contains only one statement.
1089: /// 2. there are exactly three input dimensions.
1090: /// 3. all memory accesses of the statement will have stride 0 or 1, if we
1091: ///    interchange loops (switch the variable used in the inner loop to
1092: ///    the outer loop).
1093: /// 4. all memory accesses of the statement except from the last one, are
1094: ///    read memory access and the last one is write memory access.
1095: /// 5. all subscripts of the last memory access of the statement don't
1096: ///    contain the variable used in the inner loop.
1097: /// If this is the case, we could try to use an approach that is similar to
1098: /// the one used to get close-to-peak performance of matrix multiplications.
1099: ///
1100: /// @param Node The node to check.
1101: /// @param D    The SCoP dependencies.
1102: /// @param MMI  Parameters of the matrix multiplication operands.
```
- **EN**: Documents intent, invariants, or algorithmic background for the surrounding code.
- **CN**: 这里说明了周边代码的设计意图、不变量或算法背景。

### Lines 1103-1115
```cpp
1103: static bool isMatrMultPattern(isl::schedule_node Node, const Dependences *D,
1104:                               MatMulInfoTy &MMI) {
1105:   auto PartialSchedule = isl::manage(
1106:       isl_schedule_node_band_get_partial_schedule_union_map(Node.get()));
1107:   if (isl_schedule_node_band_n_member(Node.get()) < 3 ||
1108:       Node.get_schedule_depth().release() != 0 ||
1109:       isl_union_map_n_map(PartialSchedule.get()) != 1)
1110:     return false;
1111:   auto NewPartialSchedule = isl::map::from_union_map(PartialSchedule);
1112:   if (containsMatrMult(NewPartialSchedule, D, MMI))
1113:     return true;
1114:   return false;
1115: }
```
- **EN**: Checks a condition and selects the appropriate handling path, preserving correctness for special cases.
- **CN**: 这里检查条件并选择合适的处理路径，以保证特殊情况的正确性。

### Lines 1117-1125
```cpp
1117: /// Get the dimension size.
1118: ///
1119: /// Return the size of the dimension @p Pos, which is obtained from @p SAI.
1120: /// Return -1 in the case of the first dimension of a multi-dimensional array,
1121: /// since the ScopArrayInfo class does not carry size information.
1122: ///
1123: /// @param SAI The information about the array.
1124: /// @param Pos The position of the dimension.
1125: /// @return The size of the dimension.
```
- **EN**: Defines `does`, a named type that packages structured state or classification rules for the surrounding algorithm.
- **CN**: 这里定义了 `does`，它是一个具名类型，用于为周边算法封装结构化状态或分类规则。

### Lines 1126-1136
```cpp
1126: static int getDimSize(const ScopArrayInfo *SAI, unsigned Pos) {
1127:   if (Pos == 0)
1128:     return -1;
1129:   const llvm::SCEV *SCEVDimSize = SAI->getDimensionSize(Pos);
1130:   assert(SCEVDimSize);
1131:   auto *ConstantDimSize = dyn_cast<const SCEVConstant>(SCEVDimSize);
1132:   assert(ConstantDimSize);
1133:   auto *IntDimSize = dyn_cast<ConstantInt>(ConstantDimSize->getValue());
1134:   assert(IntDimSize);
1135:   return IntDimSize->getSExtValue();
1136: }
```
- **EN**: Checks a condition and selects the appropriate handling path, preserving correctness for special cases.
- **CN**: 这里检查条件并选择合适的处理路径，以保证特殊情况的正确性。

### Lines 1138-1147
```cpp
1138: /// Check whether the access relation has the specified form.
1139: ///
1140: /// Check that the access relation @p AccMap has the form T[I0, …, In], where
1141: /// indexes I0, …, In are specified by @p Dimensions.
1142: ///
1143: /// @param Domain     The domain of the access relation.
1144: /// @param AccMap     The access relation to be checked.
1145: /// @param Dimensions The permutation of the subset of the input dimensions.
1146: /// @return True if @p AccMap has the expected form and false,
1147: ///         otherwise.
```
- **EN**: Documents intent, invariants, or algorithmic background for the surrounding code.
- **CN**: 这里说明了周边代码的设计意图、不变量或算法背景。

### Lines 1148-1152
```cpp
1148: static bool isCorrectAccessMap(isl::set Domain, isl::map AccMap,
1149:                                ArrayRef<int> Dimensions) {
1150:   isl::space Space = AccMap.get_space();
1151:   if (unsignedFromIslSize(Space.dim(isl::dim::out)) != Dimensions.size())
1152:     return false;
```
- **EN**: Checks a condition and selects the appropriate handling path, preserving correctness for special cases.
- **CN**: 这里检查条件并选择合适的处理路径，以保证特殊情况的正确性。

### Lines 1154-1156
```cpp
1154:   // Create an access relation of the following form:
1155:   // [I0, …, Im] -> [Il, …, In], where indexes
1156:   // Il, …, In are specified by @p Dimensions.
```
- **EN**: Documents intent, invariants, or algorithmic background for the surrounding code.
- **CN**: 这里说明了周边代码的设计意图、不变量或算法背景。

### Lines 1157-1157
```cpp
1157:   isl::map PossibleTensor = isl::map::universe(Space);
```
- **EN**: Introduces or continues `isl::map::universe`, a routine that carries out one step of this file's analysis or transformation pipeline.
- **CN**: 这里引入或继续 `isl::map::universe`，它是该文件分析或变换流水线中的一个步骤函数。

### Lines 1158-1158
```cpp
1158:   unsigned DimInSize = unsignedFromIslSize(Space.dim(isl::dim::in));
```
- **EN**: Introduces or continues `unsignedFromIslSize`, a routine that carries out one step of this file's analysis or transformation pipeline.
- **CN**: 这里引入或继续 `unsignedFromIslSize`，它是该文件分析或变换流水线中的一个步骤函数。

### Lines 1159-1165
```cpp
1159:   for (unsigned i = 0; i < Dimensions.size(); i++) {
1160:     const int InPos = Dimensions[i];
1161:     if ((InPos >= static_cast<int>(DimInSize)) || (InPos < 0))
1162:       return false;
1163:     PossibleTensor =
1164:         PossibleTensor.equate(isl::dim::in, InPos, isl::dim::out, i);
1165:   }
```
- **EN**: Executes an iterative step of the algorithm, usually scanning statements, dimensions, accesses, operands, or schedule nodes.
- **CN**: 这里执行算法中的迭代步骤，通常遍历语句、维度、访问、操作数或调度节点。

### Lines 1167-1167
```cpp
1167:   AccMap = AccMap.intersect_domain(Domain);
```
- **EN**: Introduces or continues `intersect_domain`, a routine that carries out one step of this file's analysis or transformation pipeline.
- **CN**: 这里引入或继续 `intersect_domain`，它是该文件分析或变换流水线中的一个步骤函数。

### Lines 1168-1168
```cpp
1168:   PossibleTensor = PossibleTensor.intersect_domain(Domain);
```
- **EN**: Introduces or continues `intersect_domain`, a routine that carries out one step of this file's analysis or transformation pipeline.
- **CN**: 这里引入或继续 `intersect_domain`，它是该文件分析或变换流水线中的一个步骤函数。

### Lines 1170-1172
```cpp
1170:   // If AccMap != PossibleTensor here (the two maps have been gisted at
1171:   // this point), it means that the writes are not complete, or in other
1172:   // words, it is a Partial write and Partial writes must be rejected.
```
- **EN**: Documents intent, invariants, or algorithmic background for the surrounding code.
- **CN**: 这里说明了周边代码的设计意图、不变量或算法背景。

### Lines 1173-1173
```cpp
1173:   return AccMap.is_equal(PossibleTensor);
```
- **EN**: Introduces or continues `is_equal`, a routine that carries out one step of this file's analysis or transformation pipeline.
- **CN**: 这里引入或继续 `is_equal`，它是该文件分析或变换流水线中的一个步骤函数。

### Lines 1174-1174
```cpp
1174: }
```
- **EN**: Closes the current block or scope and hands control back to the surrounding context.
- **CN**: 这里关闭当前代码块或作用域，并将控制流交还给外层上下文。

### Lines 1176-1188
```cpp
1176: /// Check whether the access represents the tensor contraction operand.
1177: ///
1178: /// Check that the access relation @p AccMap has the form T[i1, …, in].
1179: /// Obtained indexes i1, …, in, their sizes and their permutation are stored
1180: /// into @p IndexSet, @p DimensionSizes, and @p Dimensions, respectively.
1181: ///
1182: /// @param Domain         The domain of the access relation.
1183: /// @param AccMap         The access relation to be checked.
1184: /// @param IndexSet       The subset of the input dimensions.
1185: /// @param DimensionSizes Sizes of the input dimensions of @p Dimensions.
1186: /// @param Dimensions     The permutation of the subset of the input dimensions.
1187: /// @return True if @p AccMap has the expected form and false,
1188: ///         otherwise.
```
- **EN**: Documents intent, invariants, or algorithmic background for the surrounding code.
- **CN**: 这里说明了周边代码的设计意图、不变量或算法背景。

### Lines 1189-1195
```cpp
1189: static bool isTCOperandAcc(isl::set Domain, isl::map AccMap,
1190:                            SmallDenseSet<int> &IndexSet,
1191:                            SmallVectorImpl<int> &DimensionSizes,
1192:                            SmallVectorImpl<int> &Dimensions) {
1193:   isl::id Id = AccMap.get_tuple_id(isl::dim::out);
1194:   const ScopArrayInfo *SAI = ScopArrayInfo::getFromId(Id);
1195:   assert(SAI && "AccMap should represent memory access");
```
- **EN**: Introduces or continues `isTCOperandAcc`, a routine that carries out one step of this file's analysis or transformation pipeline.
- **CN**: 这里引入或继续 `isTCOperandAcc`，它是该文件分析或变换流水线中的一个步骤函数。

### Lines 1197-1213
```cpp
1197:   // Fix values of output dimensions with respect to their positions.
1198:   // In the case of the tensor contraction, values of output dimensions are
1199:   // fixed and form a permutation of a subset of values of input dimensions.
1200:   //
1201:   // For example, in the case of Stmt[i][j][k] -> A[k][i], which represents
1202:   // the operand of the tensor contraction, we get the following map by fixing
1203:   // the output dimensions Stmt[1][j][0] -> A[0][1].
1204:   //
1205:   // We store the permutation of the subset of the input dimensions {2, 0} into
1206:   // @p Dimensions.
1207:   //
1208:   // The obtained permutation and the isCorrectAccessMap function are used to
1209:   // check whether the access relation @p AccMap represents the tensor
1210:   // contraction operand. For example, in the case of
1211:   // Stmt[i][j][k] -> A[i-1][j+1], we get Stmt[1][0][k] -> A[0][1] and,
1212:   // consequently, {1, 0}, which is rejected by isCorrectAccessMap,
1213:   // since it corresponds to Stmt[i][j][k] -> A[j][i].
```
- **EN**: Documents intent, invariants, or algorithmic background for the surrounding code.
- **CN**: 这里说明了周边代码的设计意图、不变量或算法背景。

### Lines 1214-1214
```cpp
1214:   isl::map CheckMap = isl::manage(AccMap.copy());
```
- **EN**: Introduces or continues `isl::manage`, a routine that carries out one step of this file's analysis or transformation pipeline.
- **CN**: 这里引入或继续 `isl::manage`，它是该文件分析或变换流水线中的一个步骤函数。

### Lines 1215-1215
```cpp
1215:   unsigned OutDimNum = unsignedFromIslSize(CheckMap.dim(isl::dim::out));
```
- **EN**: Introduces or continues `unsignedFromIslSize`, a routine that carries out one step of this file's analysis or transformation pipeline.
- **CN**: 这里引入或继续 `unsignedFromIslSize`，它是该文件分析或变换流水线中的一个步骤函数。

### Lines 1216-1217
```cpp
1216:   for (unsigned i = 0; i < OutDimNum; i++)
1217:     CheckMap = CheckMap.fix_si(isl::dim::out, i, i);
```
- **EN**: Executes an iterative step of the algorithm, usually scanning statements, dimensions, accesses, operands, or schedule nodes.
- **CN**: 这里执行算法中的迭代步骤，通常遍历语句、维度、访问、操作数或调度节点。

### Lines 1219-1219
```cpp
1219:   // Try to obtain the permutation and sizes of corresponding input dimensions.
```
- **EN**: Documents intent, invariants, or algorithmic background for the surrounding code.
- **CN**: 这里说明了周边代码的设计意图、不变量或算法背景。

### Lines 1220-1220
```cpp
1220:   Dimensions.assign(OutDimNum, -1);
```
- **EN**: Introduces or continues `assign`, a routine that carries out one step of this file's analysis or transformation pipeline.
- **CN**: 这里引入或继续 `assign`，它是该文件分析或变换流水线中的一个步骤函数。

### Lines 1221-1236
```cpp
1221:   for (unsigned i : rangeIslSize(0, CheckMap.dim(isl::dim::in))) {
1222:     isl::val Val = getConstant(CheckMap, isl::dim::in, i);
1223:     if (!Val.is_int())
1224:       continue;
1225:     int OutPos = -1;
1226:     llvm::APInt ValAPInt = APIntFromVal(Val);
1227:     if (ValAPInt.isSignedIntN(32))
1228:       OutPos = ValAPInt.getSExtValue();
1229:     if ((OutPos < 0) || (OutPos >= static_cast<int>(OutDimNum)) ||
1230:         IndexSet.count(i))
1231:       return false;
1232:     IndexSet.insert(i);
1233:     Dimensions[OutPos] = i;
1234:     if (DimensionSizes[i] <= 0)
1235:       DimensionSizes[i] = getDimSize(SAI, OutPos);
1236:   }
```
- **EN**: Executes an iterative step of the algorithm, usually scanning statements, dimensions, accesses, operands, or schedule nodes.
- **CN**: 这里执行算法中的迭代步骤，通常遍历语句、维度、访问、操作数或调度节点。

### Lines 1238-1238
```cpp
1238:   return isCorrectAccessMap(Domain, AccMap, Dimensions);
```
- **EN**: Introduces or continues `isCorrectAccessMap`, a routine that carries out one step of this file's analysis or transformation pipeline.
- **CN**: 这里引入或继续 `isCorrectAccessMap`，它是该文件分析或变换流水线中的一个步骤函数。

### Lines 1239-1239
```cpp
1239: }
```
- **EN**: Closes the current block or scope and hands control back to the surrounding context.
- **CN**: 这里关闭当前代码块或作用域，并将控制流交还给外层上下文。

### Lines 1241-1246
```cpp
1241: /// Find the intersection of two sets.
1242: ///
1243: /// Find the intersection of the set @p A and the set @p B.
1244: ///
1245: /// @param A, B Sets to intersect.
1246: /// @return The set intersection.
```
- **EN**: Documents intent, invariants, or algorithmic background for the surrounding code.
- **CN**: 这里说明了周边代码的设计意图、不变量或算法背景。

### Lines 1247-1252
```cpp
1247: static SmallDenseSet<int> intersect(const SmallDenseSet<int> &A,
1248:                                     const SmallDenseSet<int> &B) {
1249:   SmallDenseSet<int> Intersection = A;
1250:   set_intersect(Intersection, B);
1251:   return Intersection;
1252: }
```
- **EN**: Introduces or continues `intersect`, a routine that carries out one step of this file's analysis or transformation pipeline.
- **CN**: 这里引入或继续 `intersect`，它是该文件分析或变换流水线中的一个步骤函数。

### Lines 1254-1259
```cpp
1254: /// Check whether the set is a superset.
1255: ///
1256: /// Check that the set @p A is a superset of @p B.
1257: ///
1258: /// @param A, B Sets to be checked.
1259: /// @return True if the set A is a superset of B.
```
- **EN**: Documents intent, invariants, or algorithmic background for the surrounding code.
- **CN**: 这里说明了周边代码的设计意图、不变量或算法背景。

### Lines 1260-1263
```cpp
1260: static bool isSuperset(const SmallDenseSet<int> &A,
1261:                        const SmallDenseSet<int> &B) {
1262:   return intersect(A, B).size() == B.size();
1263: }
```
- **EN**: Introduces or continues `isSuperset`, a routine that carries out one step of this file's analysis or transformation pipeline.
- **CN**: 这里引入或继续 `isSuperset`，它是该文件分析或变换流水线中的一个步骤函数。

### Lines 1265-1270
```cpp
1265: /// Find the union of two sets.
1266: ///
1267: /// Find the union of the set @p A and the set @p B.
1268: ///
1269: /// @param A, B Sets to unite.
1270: /// @return The set union.
```
- **EN**: Documents intent, invariants, or algorithmic background for the surrounding code.
- **CN**: 这里说明了周边代码的设计意图、不变量或算法背景。

### Lines 1271-1276
```cpp
1271: static SmallDenseSet<int> unite(const SmallDenseSet<int> &A,
1272:                                 const SmallDenseSet<int> &B) {
1273:   SmallDenseSet<int> Union = A;
1274:   set_union(Union, B);
1275:   return Union;
1276: }
```
- **EN**: Introduces or continues `unite`, a routine that carries out one step of this file's analysis or transformation pipeline.
- **CN**: 这里引入或继续 `unite`，它是该文件分析或变换流水线中的一个步骤函数。

### Lines 1278-1286
```cpp
1278: /// Determine the access that writes to the tensor, which contains
1279: /// the result of the tensor contraction.
1280: ///
1281: /// @param Domain        The domain of the statement.
1282: /// @param Stmt          The statement, which writes to memory.
1283: /// @param TCI           The information about the tensor contraction.
1284: /// @param IandJIndexSet The set, which contains free indexes of tensors.
1285: /// @return The determined MemoryAccess, or nullptr if there is no necessary
1286: ///         access within the SCoP.
```
- **EN**: Documents intent, invariants, or algorithmic background for the surrounding code.
- **CN**: 这里说明了周边代码的设计意图、不变量或算法背景。

### Lines 1287-1298
```cpp
1287: static MemoryAccess *getWriteAccess(isl::set Domain, ScopStmt *Stmt,
1288:                                     TCInfoTy &TCI,
1289:                                     SmallDenseSet<int> &IandJIndexSet) {
1290:   TCI.WriteToC = nullptr;
1291:   SmallVector<MemoryAccess *, 32> Accesses = getAccessesInOrder(*Stmt);
1292:   for (MemoryAccess *MemA : reverse(Accesses)) {
1293:     // A TC-like does not contain write scalar memory accesses
1294:     if (!MemA->isLatestArrayKind())
1295:       return nullptr;
1296:     // The last memory access should be a write memory access.
1297:     if (!MemA->isWrite())
1298:       return nullptr;
```
- **EN**: Executes an iterative step of the algorithm, usually scanning statements, dimensions, accesses, operands, or schedule nodes.
- **CN**: 这里执行算法中的迭代步骤，通常遍历语句、维度、访问、操作数或调度节点。

### Lines 1300-1300
```cpp
1300:     isl::map AccMap = MemA->getLatestAccessRelation();
```
- **EN**: Introduces or continues `getLatestAccessRelation`, a routine that carries out one step of this file's analysis or transformation pipeline.
- **CN**: 这里引入或继续 `getLatestAccessRelation`，它是该文件分析或变换流水线中的一个步骤函数。

### Lines 1301-1303
```cpp
1301:     if (!isTCOperandAcc(Domain, AccMap, IandJIndexSet, TCI.DimensionSizes,
1302:                         TCI.CDimensions))
1303:       return nullptr;
```
- **EN**: Checks a condition and selects the appropriate handling path, preserving correctness for special cases.
- **CN**: 这里检查条件并选择合适的处理路径，以保证特殊情况的正确性。

### Lines 1305-1305
```cpp
1305:     return MemA;
```
- **EN**: Returns the computed value or forwards the current result to the caller.
- **CN**: 这里返回计算结果，或将当前结果继续传递给调用方。

### Lines 1306-1306
```cpp
1306:   }
```
- **EN**: Closes the current block or scope and hands control back to the surrounding context.
- **CN**: 这里关闭当前代码块或作用域，并将控制流交还给外层上下文。

### Lines 1307-1307
```cpp
1307:   return nullptr;
```
- **EN**: Returns the computed value or forwards the current result to the caller.
- **CN**: 这里返回计算结果，或将当前结果继续传递给调用方。

### Lines 1308-1308
```cpp
1308: }
```
- **EN**: Closes the current block or scope and hands control back to the surrounding context.
- **CN**: 这里关闭当前代码块或作用域，并将控制流交还给外层上下文。

### Lines 1310-1319
```cpp
1310: /// Determine an access, which reads elements of an operand of the tensor
1311: /// contraction
1312: ///
1313: /// @param MemAccessPtr  The access, which reads elements of the tensor.
1314: /// @param IndexSet      The set, which contains indexes of the tensors.
1315: /// @param IandJIndexSet The set, which contains free indexes of tensors.
1316: /// @param Dimensions    The permutation of the subset of the input dimensions.
1317: /// @param TCI           The information about the tensor contraction.
1318: /// @return True if the memory access @p MemAccessPtr corresponds
1319: ///         to the tensor contraction.
```
- **EN**: Documents intent, invariants, or algorithmic background for the surrounding code.
- **CN**: 这里说明了周边代码的设计意图、不变量或算法背景。

### Lines 1320-1327
```cpp
1320: static bool setReadAccess(MemoryAccess *MemAccessPtr,
1321:                           const SmallDenseSet<int> &IndexSet,
1322:                           const SmallDenseSet<int> &IandJIndexSet,
1323:                           ArrayRef<int> Dimensions, TCInfoTy &TCI) {
1324:   if (!TCI.A) {
1325:     // Probably IndexSet is a union of I and P sets.
1326:     if (!isSuperset(IndexSet, TCI.P))
1327:       return false;
```
- **EN**: Checks a condition and selects the appropriate handling path, preserving correctness for special cases.
- **CN**: 这里检查条件并选择合适的处理路径，以保证特殊情况的正确性。

### Lines 1329-1329
```cpp
1329:     // Obtain the set I.
```
- **EN**: Documents intent, invariants, or algorithmic background for the surrounding code.
- **CN**: 这里说明了周边代码的设计意图、不变量或算法背景。

### Lines 1330-1330
```cpp
1330:     TCI.I = set_difference(IndexSet, TCI.P);
```
- **EN**: Introduces or continues `set_difference`, a routine that carries out one step of this file's analysis or transformation pipeline.
- **CN**: 这里引入或继续 `set_difference`，它是该文件分析或变换流水线中的一个步骤函数。

### Lines 1331-1332
```cpp
1331:     if (!isSuperset(IandJIndexSet, TCI.I))
1332:       return false;
```
- **EN**: Checks a condition and selects the appropriate handling path, preserving correctness for special cases.
- **CN**: 这里检查条件并选择合适的处理路径，以保证特殊情况的正确性。

### Lines 1334-1334
```cpp
1334:     // Obtain the set J.
```
- **EN**: Documents intent, invariants, or algorithmic background for the surrounding code.
- **CN**: 这里说明了周边代码的设计意图、不变量或算法背景。

### Lines 1335-1335
```cpp
1335:     TCI.J = set_difference(IandJIndexSet, TCI.I);
```
- **EN**: Introduces or continues `set_difference`, a routine that carries out one step of this file's analysis or transformation pipeline.
- **CN**: 这里引入或继续 `set_difference`，它是该文件分析或变换流水线中的一个步骤函数。

### Lines 1337-1337
```cpp
1337:     // Set the first operand of the tensor contraction.
```
- **EN**: Documents intent, invariants, or algorithmic background for the surrounding code.
- **CN**: 这里说明了周边代码的设计意图、不变量或算法背景。

### Lines 1338-1338
```cpp
1338:     TCI.A = MemAccessPtr;
```
- **EN**: Carries supporting computation, data movement, or bookkeeping for the surrounding algorithm.
- **CN**: 这里承担周边算法所需的辅助计算、数据传递或簿记工作。

### Lines 1339-1340
```cpp
1339:     llvm::replace(TCI.ADimensions, TCI.ADimensions.begin(),
1340:                   TCI.ADimensions.end(), Dimensions.begin(), Dimensions.end());
```
- **EN**: Introduces or continues `llvm::replace`, a routine that carries out one step of this file's analysis or transformation pipeline.
- **CN**: 这里引入或继续 `llvm::replace`，它是该文件分析或变换流水线中的一个步骤函数。

### Lines 1341-1341
```cpp
1341:     return true;
```
- **EN**: Returns the computed value or forwards the current result to the caller.
- **CN**: 这里返回计算结果，或将当前结果继续传递给调用方。

### Lines 1342-1342
```cpp
1342:   }
```
- **EN**: Closes the current block or scope and hands control back to the surrounding context.
- **CN**: 这里关闭当前代码块或作用域，并将控制流交还给外层上下文。

### Lines 1344-1347
```cpp
1344:   if (!TCI.B) {
1345:     // IndexSet should be a union of J and P sets.
1346:     if (unite(TCI.P, TCI.J) != IndexSet)
1347:       return false;
```
- **EN**: Checks a condition and selects the appropriate handling path, preserving correctness for special cases.
- **CN**: 这里检查条件并选择合适的处理路径，以保证特殊情况的正确性。

### Lines 1349-1349
```cpp
1349:     // Set the second operand of the tensor contraction.
```
- **EN**: Documents intent, invariants, or algorithmic background for the surrounding code.
- **CN**: 这里说明了周边代码的设计意图、不变量或算法背景。

### Lines 1350-1350
```cpp
1350:     TCI.B = MemAccessPtr;
```
- **EN**: Carries supporting computation, data movement, or bookkeeping for the surrounding algorithm.
- **CN**: 这里承担周边算法所需的辅助计算、数据传递或簿记工作。

### Lines 1351-1352
```cpp
1351:     llvm::replace(TCI.BDimensions, TCI.BDimensions.begin(),
1352:                   TCI.BDimensions.end(), Dimensions.begin(), Dimensions.end());
```
- **EN**: Introduces or continues `llvm::replace`, a routine that carries out one step of this file's analysis or transformation pipeline.
- **CN**: 这里引入或继续 `llvm::replace`，它是该文件分析或变换流水线中的一个步骤函数。

### Lines 1353-1353
```cpp
1353:     return true;
```
- **EN**: Returns the computed value or forwards the current result to the caller.
- **CN**: 这里返回计算结果，或将当前结果继续传递给调用方。

### Lines 1354-1354
```cpp
1354:   }
```
- **EN**: Closes the current block or scope and hands control back to the surrounding context.
- **CN**: 这里关闭当前代码块或作用域，并将控制流交还给外层上下文。

### Lines 1356-1356
```cpp
1356:   return false;
```
- **EN**: Returns the computed value or forwards the current result to the caller.
- **CN**: 这里返回计算结果，或将当前结果继续传递给调用方。

### Lines 1357-1357
```cpp
1357: }
```
- **EN**: Closes the current block or scope and hands control back to the surrounding context.
- **CN**: 这里关闭当前代码块或作用域，并将控制流交还给外层上下文。

### Lines 1359-1368
```cpp
1359: /// Check that all memory accesses of the statement, except from the last
1360: /// one, are read memory accesses, which read elements of operands of the tensor
1361: /// contraction and its result.
1362: ///
1363: /// @param Domain        The domain of the statement.
1364: /// @param Stmt          The statement, which writes to memory.
1365: /// @param TCI           The information about the tensor contraction.
1366: /// @param IandJIndexSet The set, which contains free indexes of tensors.
1367: /// @return True if all read memory accesses of the statement @p Stmt correspond
1368: ///         to the tensor contraction.
```
- **EN**: Documents intent, invariants, or algorithmic background for the surrounding code.
- **CN**: 这里说明了周边代码的设计意图、不变量或算法背景。

### Lines 1369-1376
```cpp
1369: static bool setReadAccesses(isl::set Domain, ScopStmt *Stmt, TCInfoTy &TCI,
1370:                             SmallDenseSet<int> &IandJIndexSet) {
1371:   TCI.A = nullptr;
1372:   TCI.B = nullptr;
1373:   TCI.ReadFromC = nullptr;
1374:   SmallVector<MemoryAccess *, 32> Accesses = getAccessesInOrder(*Stmt);
1375:   for (auto *MemA = Accesses.begin(); *MemA != TCI.WriteToC; MemA++) {
1376:     MemoryAccess *MemAccessPtr = *MemA;
```
- **EN**: Executes an iterative step of the algorithm, usually scanning statements, dimensions, accesses, operands, or schedule nodes.
- **CN**: 这里执行算法中的迭代步骤，通常遍历语句、维度、访问、操作数或调度节点。

### Lines 1378-1379
```cpp
1378:     // All memory accesses, except from the last one, should be read memory
1379:     // accesses.
```
- **EN**: Documents intent, invariants, or algorithmic background for the surrounding code.
- **CN**: 这里说明了周边代码的设计意图、不变量或算法背景。

### Lines 1380-1381
```cpp
1380:     if (MemAccessPtr->isWrite())
1381:       return false;
```
- **EN**: Checks a condition and selects the appropriate handling path, preserving correctness for special cases.
- **CN**: 这里检查条件并选择合适的处理路径，以保证特殊情况的正确性。

### Lines 1383-1383
```cpp
1383:     isl::map AccMap = MemAccessPtr->getLatestAccessRelation();
```
- **EN**: Introduces or continues `getLatestAccessRelation`, a routine that carries out one step of this file's analysis or transformation pipeline.
- **CN**: 这里引入或继续 `getLatestAccessRelation`，它是该文件分析或变换流水线中的一个步骤函数。

### Lines 1385-1391
```cpp
1385:     if (!MemAccessPtr->isLatestArrayKind()) {
1386:       // Check whether the scalar read memory access is not partial.
1387:       if (!Domain.is_subset(AccMap.domain()))
1388:         return false;
1389:       continue;
1390:       return false;
1391:     }
```
- **EN**: Checks a condition and selects the appropriate handling path, preserving correctness for special cases.
- **CN**: 这里检查条件并选择合适的处理路径，以保证特殊情况的正确性。

### Lines 1393-1394
```cpp
1393:     // There is only one memory access, which reads elements of the result of
1394:     // the tensor contraction.
```
- **EN**: Documents intent, invariants, or algorithmic background for the surrounding code.
- **CN**: 这里说明了周边代码的设计意图、不变量或算法背景。

### Lines 1395-1400
```cpp
1395:     if (AccMap.is_equal(TCI.WriteToC->getLatestAccessRelation())) {
1396:       if (TCI.ReadFromC)
1397:         return false;
1398:       TCI.ReadFromC = MemAccessPtr;
1399:       continue;
1400:     }
```
- **EN**: Checks a condition and selects the appropriate handling path, preserving correctness for special cases.
- **CN**: 这里检查条件并选择合适的处理路径，以保证特殊情况的正确性。

### Lines 1402-1402
```cpp
1402:     SmallVector<int> Dimensions;
```
- **EN**: Carries supporting computation, data movement, or bookkeeping for the surrounding algorithm.
- **CN**: 这里承担周边算法所需的辅助计算、数据传递或簿记工作。

### Lines 1403-1403
```cpp
1403:     SmallDenseSet<int> IndexSet;
```
- **EN**: Carries supporting computation, data movement, or bookkeeping for the surrounding algorithm.
- **CN**: 这里承担周边算法所需的辅助计算、数据传递或簿记工作。

### Lines 1404-1406
```cpp
1404:     if (!isTCOperandAcc(Domain, AccMap, IndexSet, TCI.DimensionSizes,
1405:                         Dimensions))
1406:       return false;
```
- **EN**: Checks a condition and selects the appropriate handling path, preserving correctness for special cases.
- **CN**: 这里检查条件并选择合适的处理路径，以保证特殊情况的正确性。

### Lines 1408-1409
```cpp
1408:     if (!setReadAccess(MemAccessPtr, IndexSet, IandJIndexSet, Dimensions, TCI))
1409:       return false;
```
- **EN**: Checks a condition and selects the appropriate handling path, preserving correctness for special cases.
- **CN**: 这里检查条件并选择合适的处理路径，以保证特殊情况的正确性。

### Lines 1410-1410
```cpp
1410:   }
```
- **EN**: Closes the current block or scope and hands control back to the surrounding context.
- **CN**: 这里关闭当前代码块或作用域，并将控制流交还给外层上下文。

### Lines 1412-1413
```cpp
1412:   // Check that there are read memory accesses, which read elements of operands
1413:   // of the tensor contraction and its result.
```
- **EN**: Documents intent, invariants, or algorithmic background for the surrounding code.
- **CN**: 这里说明了周边代码的设计意图、不变量或算法背景。

### Lines 1414-1414
```cpp
1414:   return TCI.ReadFromC && TCI.A && TCI.B;
```
- **EN**: Returns the computed value or forwards the current result to the caller.
- **CN**: 这里返回计算结果，或将当前结果继续传递给调用方。

### Lines 1415-1415
```cpp
1415: }
```
- **EN**: Closes the current block or scope and hands control back to the surrounding context.
- **CN**: 这里关闭当前代码块或作用域，并将控制流交还给外层上下文。

### Lines 1417-1428
```cpp
1417: /// Check accesses to operands of the tensor contraction.
1418: ///
1419: /// Check that accesses of the SCoP statement, which corresponds to
1420: /// the partial schedule @p PartialSchedule, represent accesses
1421: /// to the non-scalar operands of the tensor contraction.
1422: ///
1423: /// @param  Domain          The domain of the SCoP statement.
1424: /// @param  PartialSchedule The partial schedule of the SCoP statement.
1425: /// @param  TCI             Parameters of the tensor contraction operands.
1426: /// @return                 True if the corresponding SCoP statement
1427: ///                         represents tensor contraction and false,
1428: ///                         otherwise.
```
- **EN**: Documents intent, invariants, or algorithmic background for the surrounding code.
- **CN**: 这里说明了周边代码的设计意图、不变量或算法背景。

### Lines 1429-1432
```cpp
1429: static bool containsOnlyTCAcc(isl::set Domain, isl::map PartialSchedule,
1430:                               TCInfoTy &TCI) {
1431:   isl::id InputDimsId = PartialSchedule.get_tuple_id(isl::dim::in);
1432:   ScopStmt *Stmt = static_cast<ScopStmt *>(InputDimsId.get_user());
```
- **EN**: Introduces or continues `containsOnlyTCAcc`, a routine that carries out one step of this file's analysis or transformation pipeline.
- **CN**: 这里引入或继续 `containsOnlyTCAcc`，它是该文件分析或变换流水线中的一个步骤函数。

### Lines 1434-1435
```cpp
1434:   // In region statements, the order of memory accesses execution is not
1435:   // predictable at compile-time.
```
- **EN**: Documents intent, invariants, or algorithmic background for the surrounding code.
- **CN**: 这里说明了周边代码的设计意图、不变量或算法背景。

### Lines 1436-1437
```cpp
1436:   if ((Stmt->size() <= 1) || Stmt->isRegionStmt())
1437:     return false;
```
- **EN**: Checks a condition and selects the appropriate handling path, preserving correctness for special cases.
- **CN**: 这里检查条件并选择合适的处理路径，以保证特殊情况的正确性。

### Lines 1439-1439
```cpp
1439:   unsigned DimNum = unsignedFromIslSize(PartialSchedule.dim(isl::dim::in));
```
- **EN**: Introduces or continues `unsignedFromIslSize`, a routine that carries out one step of this file's analysis or transformation pipeline.
- **CN**: 这里引入或继续 `unsignedFromIslSize`，它是该文件分析或变换流水线中的一个步骤函数。

### Lines 1440-1440
```cpp
1440:   TCI.DimensionSizes.resize(DimNum);
```
- **EN**: Introduces or continues `resize`, a routine that carries out one step of this file's analysis or transformation pipeline.
- **CN**: 这里引入或继续 `resize`，它是该文件分析或变换流水线中的一个步骤函数。

### Lines 1441-1441
```cpp
1441:   SmallDenseSet<int> IandJIndexSet;
```
- **EN**: Carries supporting computation, data movement, or bookkeeping for the surrounding algorithm.
- **CN**: 这里承担周边算法所需的辅助计算、数据传递或簿记工作。

### Lines 1443-1443
```cpp
1443:   TCI.WriteToC = getWriteAccess(Domain, Stmt, TCI, IandJIndexSet);
```
- **EN**: Introduces or continues `getWriteAccess`, a routine that carries out one step of this file's analysis or transformation pipeline.
- **CN**: 这里引入或继续 `getWriteAccess`，它是该文件分析或变换流水线中的一个步骤函数。

### Lines 1444-1445
```cpp
1444:   if (!TCI.WriteToC)
1445:     return false;
```
- **EN**: Checks a condition and selects the appropriate handling path, preserving correctness for special cases.
- **CN**: 这里检查条件并选择合适的处理路径，以保证特殊情况的正确性。

### Lines 1447-1448
```cpp
1447:   if (intersect(IandJIndexSet, TCI.P).size() != 0)
1448:     return false;
```
- **EN**: Checks a condition and selects the appropriate handling path, preserving correctness for special cases.
- **CN**: 这里检查条件并选择合适的处理路径，以保证特殊情况的正确性。

### Lines 1450-1451
```cpp
1450:   if (!setReadAccesses(Domain, Stmt, TCI, IandJIndexSet))
1451:     return false;
```
- **EN**: Checks a condition and selects the appropriate handling path, preserving correctness for special cases.
- **CN**: 这里检查条件并选择合适的处理路径，以保证特殊情况的正确性。

### Lines 1453-1453
```cpp
1453:   return true;
```
- **EN**: Returns the computed value or forwards the current result to the caller.
- **CN**: 这里返回计算结果，或将当前结果继续传递给调用方。

### Lines 1454-1454
```cpp
1454: }
```
- **EN**: Closes the current block or scope and hands control back to the surrounding context.
- **CN**: 这里关闭当前代码块或作用域，并将控制流交还给外层上下文。

### Lines 1456-1477
```cpp
1456: /// Check that dependency corresponds to the tensor contraction carried over
1457: /// loop dimension @p Dim.
1458: ///
1459: /// Check that the dependency has the form
1460: /// S(..., ki, max(k(i + 1)), ..., max(kn), ...) ->
1461: /// S(..., ki + 1, min(k(i + 1)), ..., min(kn), ...), where S is the SCoP
1462: /// statement. For this purpose, we analyze the set @p DepDelta, which
1463: /// represents the differences between image elements and domain elements of
1464: /// the corresponding map.
1465: ///
1466: /// @param  DepDelta    The set contains the differences between image elements
1467: ///                     and corresponding domain elements of the map, which
1468: ///                     represents the dependency.
1469: /// @param  Dim         The position of the index ki.
1470: /// @param  BoundDeltas In the case of indexes of ki, the difference between
1471: ///                     image elements and corresponding domain elements
1472: ///                     corresponds to the difference between lexicographic
1473: ///                     minimum and lexicographic maximum of the corresponding
1474: ///                     dimension of the domain of the statement.
1475: /// @param  IndexSet    Obtained indexes ki, which describe the dependency.
1476: /// @return True if dependencies correspond to the tensor contraction
1477: ///         and false, otherwise.
```
- **EN**: Documents intent, invariants, or algorithmic background for the surrounding code.
- **CN**: 这里说明了周边代码的设计意图、不变量或算法背景。

### Lines 1478-1485
```cpp
1478: static bool isReductionCarriedOverDim(isl::set DepDelta, unsigned Dim,
1479:                                       isl::pw_multi_aff BoundDeltas,
1480:                                       const SmallDenseSet<int> &IndexSet) {
1481:   isl::space Space = DepDelta.get_space();
1482:   isl::set Superset = isl::set::universe(Space);
1483:   for (unsigned i = 0; i < Dim; i += 1)
1484:     Superset = Superset.fix_si(isl::dim::set, i, 0);
1485:   Superset = Superset.fix_si(isl::dim::set, Dim, 1);
```
- **EN**: Executes an iterative step of the algorithm, usually scanning statements, dimensions, accesses, operands, or schedule nodes.
- **CN**: 这里执行算法中的迭代步骤，通常遍历语句、维度、访问、操作数或调度节点。

### Lines 1487-1490
```cpp
1487:   // Check that the difference between the image element and the domain element
1488:   // is equal to one in the case of the index ki. Image elements and
1489:   // corresponding domain elements should be equal in the case of positions,
1490:   // which are lower than the specified position.
```
- **EN**: Documents intent, invariants, or algorithmic background for the surrounding code.
- **CN**: 这里说明了周边代码的设计意图、不变量或算法背景。

### Lines 1491-1492
```cpp
1491:   if (!DepDelta.is_subset(Superset))
1492:     return false;
```
- **EN**: Checks a condition and selects the appropriate handling path, preserving correctness for special cases.
- **CN**: 这里检查条件并选择合适的处理路径，以保证特殊情况的正确性。

### Lines 1494-1495
```cpp
1494:   // Compute a set, which is used to analyze how values of
1495:   // the domain are related to the map that describes the dependency.
```
- **EN**: Documents intent, invariants, or algorithmic background for the surrounding code.
- **CN**: 这里说明了周边代码的设计意图、不变量或算法背景。

### Lines 1496-1496
```cpp
1496:   isl_pw_multi_aff *DepDeltaPW = isl_pw_multi_aff_from_set(DepDelta.copy());
```
- **EN**: Introduces or continues `isl_pw_multi_aff_from_set`, a routine that carries out one step of this file's analysis or transformation pipeline.
- **CN**: 这里引入或继续 `isl_pw_multi_aff_from_set`，它是该文件分析或变换流水线中的一个步骤函数。

### Lines 1497-1497
```cpp
1497:   BoundDeltas = BoundDeltas.add(isl::manage(DepDeltaPW));
```
- **EN**: Introduces or continues `add`, a routine that carries out one step of this file's analysis or transformation pipeline.
- **CN**: 这里引入或继续 `add`，它是该文件分析或变换流水线中的一个步骤函数。

### Lines 1498-1498
```cpp
1498:   isl_set *ComplementRawSet = isl_set_from_pw_multi_aff(BoundDeltas.release());
```
- **EN**: Introduces or continues `isl_set_from_pw_multi_aff`, a routine that carries out one step of this file's analysis or transformation pipeline.
- **CN**: 这里引入或继续 `isl_set_from_pw_multi_aff`，它是该文件分析或变换流水线中的一个步骤函数。

### Lines 1499-1499
```cpp
1499:   isl::set Complement = isl::manage(ComplementRawSet);
```
- **EN**: Introduces or continues `isl::manage`, a routine that carries out one step of this file's analysis or transformation pipeline.
- **CN**: 这里引入或继续 `isl::manage`，它是该文件分析或变换流水线中的一个步骤函数。

### Lines 1501-1508
```cpp
1501:   for (unsigned i : rangeIslSize(Dim + 1, DepDelta.dim(isl::dim::set))) {
1502:     if (!IndexSet.count(i)) {
1503:       // Check the difference between the image element and the domain element
1504:       // in the case of indexes, which do not describe the dependency.
1505:       if (DepDelta.plain_get_val_if_fixed(isl::dim::set, i).is_zero())
1506:         continue;
1507:       return false;
1508:     }
```
- **EN**: Executes an iterative step of the algorithm, usually scanning statements, dimensions, accesses, operands, or schedule nodes.
- **CN**: 这里执行算法中的迭代步骤，通常遍历语句、维度、访问、操作数或调度节点。

### Lines 1510-1513
```cpp
1510:     // In the case of other indexes, which describe the dependency,
1511:     // the difference between the image element and the domain element
1512:     // should be equal to the difference between lexicographic minimum and
1513:     // lexicographic maximum of the domain of the statement.
```
- **EN**: Documents intent, invariants, or algorithmic background for the surrounding code.
- **CN**: 这里说明了周边代码的设计意图、不变量或算法背景。

### Lines 1514-1515
```cpp
1514:     if (!Complement.plain_get_val_if_fixed(isl::dim::set, i).is_zero())
1515:       return false;
```
- **EN**: Checks a condition and selects the appropriate handling path, preserving correctness for special cases.
- **CN**: 这里检查条件并选择合适的处理路径，以保证特殊情况的正确性。

### Lines 1516-1516
```cpp
1516:   }
```
- **EN**: Closes the current block or scope and hands control back to the surrounding context.
- **CN**: 这里关闭当前代码块或作用域，并将控制流交还给外层上下文。

### Lines 1518-1518
```cpp
1518:   return true;
```
- **EN**: Returns the computed value or forwards the current result to the caller.
- **CN**: 这里返回计算结果，或将当前结果继续传递给调用方。

### Lines 1519-1519
```cpp
1519: }
```
- **EN**: Closes the current block or scope and hands control back to the surrounding context.
- **CN**: 这里关闭当前代码块或作用域，并将控制流交还给外层上下文。

### Lines 1521-1563
```cpp
1521: /// Check whether dependencies are over the complete domain.
1522: ///
1523: /// In the case of the tensor contraction RAW, WAW, WAR dependencies
1524: /// have the form
1525: /// S(..., ki, max(k(i + 1)), ..., max(kn), ...) ->
1526: /// S(..., ki + 1, min(k(i + 1)), ..., min(kn), ...), where S is the SCoP
1527: /// statement. Consequently, the domain of the dependencies
1528: /// can be described as
1529: /// Domain / Domain ∩ S(…, max(kn),…) ∩ S(…, max(k(i + 1)),…),
1530: /// where Domain is the domain of the statement S.
1531: ///
1532: /// For example, in the case of the following tensor contraction,
1533: /// corresponding domains will have the following form.
1534: ///
1535: /// An example of the tensor contraction:
1536: /// for (i = 0; i < 1024; i++)
1537: ///   for (j = 0; j < 1024; j++)
1538: ///     for (l = 0; l < 64; ++l)
1539: ///       for (w = 0; w < 64; ++w)
1540: ///         C[i][j] += A[i][l][w] * B[w][j][l];
1541: ///
1542: /// The domain of the statement:
1543: /// { S[i0, i1, i2, i3] : i0 >= 0 and i0 <= 1023 and
1544: ///                       i1 >= 0 and i1 <= 1023 and
1545: ///                       i2 >= 0 and i2 <= 63 and
1546: ///                       i3 >= 0 and i3 <= 63 }
1547: ///
1548: /// The domain of the dependencies:
1549: /// { S[i0, i1, i2, i3] : (i0 >= 0 and i0 <= 1023 and
1550: ///                        i1 >= 0 and i1 <= 1023 and
1551: ///                        i2 >= 0 and i2 <= 63 and
1552: ///                        i3 >= 0 and i3 <= 62) or
1553: ///                       (i3 = 63 and i0 >= 0 and i0 <= 1023 and
1554: ///                        i1 >= 0 and i1 <= 1023 and
1555: ///                        i2 >= 0 and i2 <= 62) }
1556: ///
1557: /// @param  Domain       The domain of the statement.
1558: /// @param  DepsForStmt  RAW and RED dependencies for the statement.
1559: /// @param  UpperBound   The lexicographic maximum of the elements in
1560: ///                      the @p Domain.
1561: /// @param  IndexSet     Obtained indexes ki, which describe the dependencies.
1562: /// @return True if dependencies are over the complete domain
1563: ///         and false, otherwise.
```
- **EN**: Documents intent, invariants, or algorithmic background for the surrounding code.
- **CN**: 这里说明了周边代码的设计意图、不变量或算法背景。

### Lines 1564-1568
```cpp
1564: static bool areDepsOverCompleteDomain(isl::set Domain, isl::map DepsForStmt,
1565:                                       isl::pw_multi_aff UpperBound,
1566:                                       SmallDenseSet<int> &IndexSet) {
1567:   isl_set *UpperBoundRawSet = isl_set_from_pw_multi_aff(UpperBound.copy());
1568:   isl::set UpperBoundSet = isl::manage(UpperBoundRawSet);
```
- **EN**: Introduces or continues `areDepsOverCompleteDomain`, a routine that carries out one step of this file's analysis or transformation pipeline.
- **CN**: 这里引入或继续 `areDepsOverCompleteDomain`，它是该文件分析或变换流水线中的一个步骤函数。

### Lines 1570-1570
```cpp
1570:   isl::set DomainRed = isl::manage(Domain.copy());
```
- **EN**: Introduces or continues `isl::manage`, a routine that carries out one step of this file's analysis or transformation pipeline.
- **CN**: 这里引入或继续 `isl::manage`，它是该文件分析或变换流水线中的一个步骤函数。

### Lines 1571-1577
```cpp
1571:   for (const auto It : IndexSet) {
1572:     isl::val FixedVal = UpperBoundSet.plain_get_val_if_fixed(isl::dim::set, It);
1573:     if (FixedVal.is_nan())
1574:       return false;
1575:     DomainRed = isl::manage(
1576:         isl_set_fix_val(DomainRed.copy(), isl_dim_set, It, FixedVal.release()));
1577:   }
```
- **EN**: Executes an iterative step of the algorithm, usually scanning statements, dimensions, accesses, operands, or schedule nodes.
- **CN**: 这里执行算法中的迭代步骤，通常遍历语句、维度、访问、操作数或调度节点。

### Lines 1578-1579
```cpp
1578:   return DepsForStmt.domain().intersect(Domain).is_equal(
1579:       Domain.subtract(DomainRed));
```
- **EN**: Introduces or continues `domain`, a routine that carries out one step of this file's analysis or transformation pipeline.
- **CN**: 这里引入或继续 `domain`，它是该文件分析或变换流水线中的一个步骤函数。

### Lines 1580-1580
```cpp
1580: }
```
- **EN**: Closes the current block or scope and hands control back to the surrounding context.
- **CN**: 这里关闭当前代码块或作用域，并将控制流交还给外层上下文。

### Lines 1582-1598
```cpp
1582: /// Check that dependencies correspond to the tensor contraction.
1583: ///
1584: /// Check that there are only true dependencies of the form
1585: /// S(..., ki, max(k(i + 1)), ..., max(kn), ...) ->
1586: /// S(..., ki + 1, min(k(i + 1)), ..., min(kn), ...), where S is the SCoP
1587: /// statement represented by @p Schedule. Such dependencies are produced by
1588: /// the tensor contraction. Obtained indexes ki are stored into @p IndexSet.
1589: ///
1590: /// The form of anti and output dependencies is specified implicitly by
1591: /// the form the SCoP statement, which is checked by subsequent analysis.
1592: ///
1593: /// @param  Schedule The schedule of the SCoP statement.
1594: /// @param  D        The SCoP dependencies.
1595: /// @param  Domain   The domain of the statement.
1596: /// @param  IndexSet Obtained indexes ki, which describe the dependencies.
1597: /// @return True if dependencies correspond to the tensor contraction
1598: ///         and false, otherwise.
```
- **EN**: Documents intent, invariants, or algorithmic background for the surrounding code.
- **CN**: 这里说明了周边代码的设计意图、不变量或算法背景。

### Lines 1599-1601
```cpp
1599: static bool containsOnlyTcDeps(isl::map Schedule, const Dependences *D,
1600:                                SmallDenseSet<int> &IndexSet, isl::set Domain) {
1601:   IslMaxOperationsGuard MaxOpGuard(Schedule.ctx().get(), OptComputeOut);
```
- **EN**: Introduces or continues `containsOnlyTcDeps`, a routine that carries out one step of this file's analysis or transformation pipeline.
- **CN**: 这里引入或继续 `containsOnlyTcDeps`，它是该文件分析或变换流水线中的一个步骤函数。

### Lines 1603-1604
```cpp
1603:   isl::union_map Dep =
1604:       D->getDependences(Dependences::TYPE_RAW | Dependences::TYPE_RED);
```
- **EN**: Introduces or continues `getDependences`, a routine that carries out one step of this file's analysis or transformation pipeline.
- **CN**: 这里引入或继续 `getDependences`，它是该文件分析或变换流水线中的一个步骤函数。

### Lines 1606-1606
```cpp
1606:   isl::space DomainSpace = Schedule.get_space().domain();
```
- **EN**: Introduces or continues `get_space`, a routine that carries out one step of this file's analysis or transformation pipeline.
- **CN**: 这里引入或继续 `get_space`，它是该文件分析或变换流水线中的一个步骤函数。

### Lines 1607-1607
```cpp
1607:   isl::space Space = DomainSpace.map_from_domain_and_range(DomainSpace);
```
- **EN**: Introduces or continues `map_from_domain_and_range`, a routine that carries out one step of this file's analysis or transformation pipeline.
- **CN**: 这里引入或继续 `map_from_domain_and_range`，它是该文件分析或变换流水线中的一个步骤函数。

### Lines 1608-1608
```cpp
1608:   isl::map DepsForStmt = Dep.extract_map(Space);
```
- **EN**: Introduces or continues `extract_map`, a routine that carries out one step of this file's analysis or transformation pipeline.
- **CN**: 这里引入或继续 `extract_map`，它是该文件分析或变换流水线中的一个步骤函数。

### Lines 1609-1609
```cpp
1609:   isl::set DepDeltas = DepsForStmt.deltas();
```
- **EN**: Introduces or continues `deltas`, a routine that carries out one step of this file's analysis or transformation pipeline.
- **CN**: 这里引入或继续 `deltas`，它是该文件分析或变换流水线中的一个步骤函数。

### Lines 1610-1610
```cpp
1610:   isl::size DeltasDimNum = DepDeltas.dim(isl::dim::set);
```
- **EN**: Introduces or continues `dim`, a routine that carries out one step of this file's analysis or transformation pipeline.
- **CN**: 这里引入或继续 `dim`，它是该文件分析或变换流水线中的一个步骤函数。

### Lines 1611-1611
```cpp
1611:   isl::pw_multi_aff LowerBound = Domain.lexmin_pw_multi_aff();
```
- **EN**: Introduces or continues `lexmin_pw_multi_aff`, a routine that carries out one step of this file's analysis or transformation pipeline.
- **CN**: 这里引入或继续 `lexmin_pw_multi_aff`，它是该文件分析或变换流水线中的一个步骤函数。

### Lines 1612-1612
```cpp
1612:   isl::pw_multi_aff UpperBound = Domain.lexmax_pw_multi_aff();
```
- **EN**: Introduces or continues `lexmax_pw_multi_aff`, a routine that carries out one step of this file's analysis or transformation pipeline.
- **CN**: 这里引入或继续 `lexmax_pw_multi_aff`，它是该文件分析或变换流水线中的一个步骤函数。

### Lines 1613-1613
```cpp
1613:   isl::pw_multi_aff BoundDeltas = UpperBound.sub(LowerBound);
```
- **EN**: Introduces or continues `sub`, a routine that carries out one step of this file's analysis or transformation pipeline.
- **CN**: 这里引入或继续 `sub`，它是该文件分析或变换流水线中的一个步骤函数。

### Lines 1615-1621
```cpp
1615:   for (int i : reverse(rangeIslSize(0, DeltasDimNum))) {
1616:     // In the case of the tensor contraction, the difference between image
1617:     // elements and domain elements lies on a hyperplane where a dimension
1618:     // has the fixed value one.
1619:     isl::set Intersection = DepDeltas.fix_si(isl::dim::set, i, 1);
1620:     if (Intersection.is_empty())
1621:       continue;
```
- **EN**: Executes an iterative step of the algorithm, usually scanning statements, dimensions, accesses, operands, or schedule nodes.
- **CN**: 这里执行算法中的迭代步骤，通常遍历语句、维度、访问、操作数或调度节点。

### Lines 1623-1624
```cpp
1623:     if (!isReductionCarriedOverDim(Intersection, i, BoundDeltas, IndexSet))
1624:       return false;
```
- **EN**: Checks a condition and selects the appropriate handling path, preserving correctness for special cases.
- **CN**: 这里检查条件并选择合适的处理路径，以保证特殊情况的正确性。

### Lines 1626-1626
```cpp
1626:     IndexSet.insert(i);
```
- **EN**: Introduces or continues `insert`, a routine that carries out one step of this file's analysis or transformation pipeline.
- **CN**: 这里引入或继续 `insert`，它是该文件分析或变换流水线中的一个步骤函数。

### Lines 1627-1627
```cpp
1627:     DepDeltas = DepDeltas.subtract(Intersection);
```
- **EN**: Introduces or continues `subtract`, a routine that carries out one step of this file's analysis or transformation pipeline.
- **CN**: 这里引入或继续 `subtract`，它是该文件分析或变换流水线中的一个步骤函数。

### Lines 1628-1628
```cpp
1628:   }
```
- **EN**: Closes the current block or scope and hands control back to the surrounding context.
- **CN**: 这里关闭当前代码块或作用域，并将控制流交还给外层上下文。

### Lines 1630-1631
```cpp
1630:   // In the case of the tensor contraction, all dependencies should have
1631:   // the previously described form.
```
- **EN**: Documents intent, invariants, or algorithmic background for the surrounding code.
- **CN**: 这里说明了周边代码的设计意图、不变量或算法背景。

### Lines 1632-1633
```cpp
1632:   if ((unsignedFromIslSize(DeltasDimNum) == 0) || !DepDeltas.is_empty())
1633:     return false;
```
- **EN**: Checks a condition and selects the appropriate handling path, preserving correctness for special cases.
- **CN**: 这里检查条件并选择合适的处理路径，以保证特殊情况的正确性。

### Lines 1635-1635
```cpp
1635:   return areDepsOverCompleteDomain(Domain, DepsForStmt, UpperBound, IndexSet);
```
- **EN**: Introduces or continues `areDepsOverCompleteDomain`, a routine that carries out one step of this file's analysis or transformation pipeline.
- **CN**: 这里引入或继续 `areDepsOverCompleteDomain`，它是该文件分析或变换流水线中的一个步骤函数。

### Lines 1636-1636
```cpp
1636: }
```
- **EN**: Closes the current block or scope and hands control back to the surrounding context.
- **CN**: 这里关闭当前代码块或作用域，并将控制流交还给外层上下文。

### Lines 1638-1665
```cpp
1638: /// Check if the SCoP statement could probably be optimized with analytical
1639: /// modeling.
1640: ///
1641: /// containsTCInfoTy tries to determine whether the following conditions
1642: /// are true:
1643: ///
1644: /// 1. The last memory access modeling an array, MA1, represents writing to
1645: ///    memory and has the form S(..., I, ..., J, ...) -> M(shuffle(I, J)),
1646: ///    where S is the SCoP statement under consideration and shuffle(I, J)
1647: ///    is a permutation of indexes of sets I and J.
1648: /// 2. There are only true dependencies of the form
1649: ///    S(..., ki, max(k(i + 1)), ..., max(kn), ...) ->
1650: ///    S(..., ki + 1, min(k(i + 1)), ..., min(kn), ...), where S is the SCoP
1651: ///    statement represented by @p Schedule and ki are indexes of the set P.
1652: /// 3. SCoP contains an arbitrary number of reads from constants and only three
1653: ///    access relations, MA2, MA3, and MA4 that represent reading from memory
1654: ///    and have the form
1655: ///    S(..., I, ..., P, ...) -> M(shuffle(I, P)),
1656: ///    S(..., P, ..., J, ...) -> M(shuffle(J, P)),
1657: ///    S(...) -> M(shuffle(I, J)), respectively.
1658: ///
1659: /// @param  PartialSchedule The PartialSchedule that contains a SCoP statement
1660: ///                         to check.
1661: /// @param  D               The SCoP dependencies.
1662: /// @param  TCI             Parameters of the tensor contraction operands.
1663: /// @param  Domain          The domain of the statement.
1664: /// @return True if dependencies and memory accesses correspond to the tensor
1665: ///              contraction and false, otherwise.
```
- **EN**: Documents intent, invariants, or algorithmic background for the surrounding code.
- **CN**: 这里说明了周边代码的设计意图、不变量或算法背景。

### Lines 1666-1669
```cpp
1666: static bool containsTCInfoTy(isl::map PartialSchedule, const Dependences *D,
1667:                              TCInfoTy &TCI, isl::set Domain) {
1668:   if (!containsOnlyTcDeps(PartialSchedule, D, TCI.P, Domain))
1669:     return false;
```
- **EN**: Checks a condition and selects the appropriate handling path, preserving correctness for special cases.
- **CN**: 这里检查条件并选择合适的处理路径，以保证特殊情况的正确性。

### Lines 1671-1671
```cpp
1671:   // TODO: handle cases of scalar multiplication if needed.
```
- **EN**: Documents intent, invariants, or algorithmic background for the surrounding code.
- **CN**: 这里说明了周边代码的设计意图、不变量或算法背景。

### Lines 1672-1673
```cpp
1672:   if (TCI.P.size() == 0)
1673:     return false;
```
- **EN**: Checks a condition and selects the appropriate handling path, preserving correctness for special cases.
- **CN**: 这里检查条件并选择合适的处理路径，以保证特殊情况的正确性。

### Lines 1675-1676
```cpp
1675:   if (!containsOnlyTCAcc(Domain, PartialSchedule, TCI))
1676:     return false;
```
- **EN**: Checks a condition and selects the appropriate handling path, preserving correctness for special cases.
- **CN**: 这里检查条件并选择合适的处理路径，以保证特殊情况的正确性。

### Lines 1678-1678
```cpp
1678:   // TODO: handle cases of GEMV if needed.
```
- **EN**: Documents intent, invariants, or algorithmic background for the surrounding code.
- **CN**: 这里说明了周边代码的设计意图、不变量或算法背景。

### Lines 1679-1680
```cpp
1679:   if ((TCI.I.size() == 0) || (TCI.J.size() == 0))
1680:     return false;
```
- **EN**: Checks a condition and selects the appropriate handling path, preserving correctness for special cases.
- **CN**: 这里检查条件并选择合适的处理路径，以保证特殊情况的正确性。

### Lines 1682-1682
```cpp
1682:   return true;
```
- **EN**: Returns the computed value or forwards the current result to the caller.
- **CN**: 这里返回计算结果，或将当前结果继续传递给调用方。

### Lines 1683-1683
```cpp
1683: }
```
- **EN**: Closes the current block or scope and hands control back to the surrounding context.
- **CN**: 这里关闭当前代码块或作用域，并将控制流交还给外层上下文。

### Lines 1685-1739
```cpp
1685: /// Check if this node contains a partial schedule that could
1686: /// probably be optimized with analytical modeling.
1687: ///
1688: /// isTCPattern is used to determine whether the SCoP represents a TC-like
1689: /// kernel [1], which is a perfectly nested set of loops, with a data usage
1690: /// pattern that is similar to that produced by the tensor contraction.
1691: ///
1692: /// A TC-like kernel can be defined as follows:
1693: ///
1694: /// 1. It satisfies the requirements of the polyhedral model.
1695: /// 2. Without loss of generality, it contains three nonempty bundles of
1696: ///    one-dimensional for-loops with induction variables that are grouped into
1697: ///    bundles I = i0...i(r-1), J = j0..j(s-1), and P = p0...p(t-1), and they
1698: ///    are incremented by one.
1699: /// 3. The innermost loop body can be represented as a statement of the form
1700: ///    C(shuffle(I, J)) = E(A(shuffle(I, P)), B(shuffle(P, J)),
1701: ///    C(shuffle(I, J))), where A(shuffle(I, P)), B(shuffle(P, J)),
1702: ///    C(shuffle(I, J)) are accesses to tensors A, B, C, respectively,
1703: ///    shuffle(I, J), shuffle(I, P), and shuffle(P, J) are permutations of the
1704: ///    enclosed indices, and E is an expression that contains reads from
1705: ///    the tensors A, B, C, and an arbitrary number of reads from constants
1706: ///    with respect to bundles I, J, and P.
1707: ///
1708: /// TC can be considered as a particular case of a TC-like kernel.
1709: ///
1710: /// The order of loops with indexes from P should be preserved. Otherwise,
1711: /// isTCPattern should check if a commutative operation is used.
1712: ///
1713: /// isTCPattern performs the following steps to check whether the SCoP
1714: /// corresponds to a definition of a TC-like kernel:
1715: ///
1716: /// 1. Checks that the node is the innermost band node.
1717: /// 2. Checks that the partial schedule contains only one statement.
1718: /// 3. Check that all ancestors of the node contain all band nodes for
1719: ///    the statement and only mark nodes interleave such band nodes. This
1720: ///    corresponds to a straightforward implementation of TC.
1721: /// 4. Analyses the dependencies to determine contraction dimensions.
1722: /// 5. Check that the last memory access modeling an array, represents writing
1723: ///    to the result of the TC-like kernel.
1724: /// 6. Check that SCoP contains only three access relations that represent
1725: ///    reading of the operands of the TC-like kernel and an arbitrary number of
1726: ///    reads from constants.
1727: ///
1728: /// [1] - Gareev R., Grosser T., Kruse M. High-Performance Generalized Tensor
1729: ///       Operations: A Compiler-Oriented Approach // ACM Transactions
1730: ///       Architecture and Code Optimization (TACO). 2018.
1731: ///       Vol. 15, no. 3. P. 34:1–34:27. DOI: 10.1145/3235029.
1732: ///
1733: /// If this is the case, we could logically represent tensors as matrices and
1734: /// apply algorithms, which are used to get close-to-peak performance of
1735: /// matrix multiplications in manually tuned BLAS libraries (e.g., BLIS).
1736: ///
1737: /// @param Node The node to check.
1738: /// @param D    The SCoP dependencies.
1739: /// @param TCI  Parameters of the tensor contraction operands.
```
- **EN**: Documents intent, invariants, or algorithmic background for the surrounding code.
- **CN**: 这里说明了周边代码的设计意图、不变量或算法背景。

### Lines 1740-1745
```cpp
1740: static bool isTCPattern(isl::schedule_node Node, const Dependences *D,
1741:                         TCInfoTy &TCI) {
1742:   Node = Node.child(0);
1743:   isl::union_map PartialSchedule = Node.get_prefix_schedule_union_map();
1744:   isl::union_set Domain = Node.domain();
1745:   Node = Node.parent();
```
- **EN**: Introduces or continues `isTCPattern`, a routine that carries out one step of this file's analysis or transformation pipeline.
- **CN**: 这里引入或继续 `isTCPattern`，它是该文件分析或变换流水线中的一个步骤函数。

### Lines 1747-1748
```cpp
1747:   // The partial schedule should contain only one statement.
1748:   // TODO: This constraint should not be intrinsic to the algorithm.
```
- **EN**: Documents intent, invariants, or algorithmic background for the surrounding code.
- **CN**: 这里说明了周边代码的设计意图、不变量或算法背景。

### Lines 1749-1750
```cpp
1749:   if (isl_union_set_n_set(Domain.get()) != 1)
1750:     return false;
```
- **EN**: Checks a condition and selects the appropriate handling path, preserving correctness for special cases.
- **CN**: 这里检查条件并选择合适的处理路径，以保证特殊情况的正确性。

### Lines 1752-1752
```cpp
1752:   isl_schedule_node_type NodeType = isl_schedule_node_get_type(Node.get());
```
- **EN**: Introduces or continues `isl_schedule_node_get_type`, a routine that carries out one step of this file's analysis or transformation pipeline.
- **CN**: 这里引入或继续 `isl_schedule_node_get_type`，它是该文件分析或变换流水线中的一个步骤函数。

### Lines 1754-1796
```cpp
1754:   // Check that all ancestors of the node contain all band nodes for
1755:   // the statement, which represents the TC-like kernel, and only mark nodes
1756:   // interleave such band nodes. This corresponds to a straightforward
1757:   // implementation of TC with/without DeLICM applied.
1758:   //
1759:   // For example, this covers the matrix multiplication pattern after a full
1760:   // run of -polly-optree and -polly-delicm, where the write access is not
1761:   // through the original memory access, but through a PHI node that was
1762:   // delicmed. Subsequently, such band nodes will be replaced by a single band
1763:   // node.
1764:   //
1765:   // The corresponding schedule can be the following, where Stmt_for_body8
1766:   // contains the matrix multiplication:
1767:   //
1768:   // domain: "{ Stmt_for_body8[i0, i1, i2]  : 0 <= i0 <= 1599 and
1769:   //                                          0 <= i1 <= 1799 and
1770:   //                                          0 <= i2 <= 2199;
1771:   //            Stmt_for_body3[i0, i1] :      0 <= i0 <= 1599 and
1772:   //                                          0 <= i1 <= 1799;
1773:   //            Stmt_for_body3_last[i0, i1] : 0 <= i0 <= 1599 and
1774:   //                                          0 <= i1 <= 1799 }"
1775:   // child:
1776:   //  sequence:
1777:   //  - filter: "{ Stmt_for_body3[i0, i1] }"
1778:   //    child:
1779:   //      schedule: "[{ Stmt_for_body3[i0, i1] -> [(i0)] },
1780:   //                  { Stmt_for_body3[i0, i1] -> [(i1)] }]"
1781:   //      permutable: 1
1782:   //      coincident: [ 1, 1 ]
1783:   //  - filter: "{ Stmt_for_body3_last[i0, i1] }"
1784:   //    child:
1785:   //      schedule: "[{ Stmt_for_body3_last[i0, i1] -> [(i0)] },
1786:   //                  { Stmt_for_body3_last[i0, i1] -> [(i1)] }]"
1787:   //      permutable: 1
1788:   //      coincident: [ 1, 1 ]
1789:   //  - filter: "{ Stmt_for_body8[i0, i1, i2] }"
1790:   //    child:
1791:   //      schedule: "[{ Stmt_for_body8[i0, i1, i2] -> [(i0)] },
1792:   //                  { Stmt_for_body8[i0, i1, i2] -> [(i1)] },
1793:   //                  { Stmt_for_body8[i0, i1, i2] -> [(i2)] }]"
1794:   //      permutable: 1
1795:   //      coincident: [ 1, 1, 0 ]
1796:   //
```
- **EN**: Documents intent, invariants, or algorithmic background for the surrounding code.
- **CN**: 这里说明了周边代码的设计意图、不变量或算法背景。

### Lines 1797-1803
```cpp
1797:   while (NodeType != isl_schedule_node_domain) {
1798:     if (NodeType == isl_schedule_node_filter) {
1799:       if (!Node.parent().isa<isl::schedule_node_sequence>() ||
1800:           !Node.parent().parent().isa<isl::schedule_node_domain>())
1801:         return false;
1802:       break;
1803:     }
```
- **EN**: Checks a condition and selects the appropriate handling path, preserving correctness for special cases.
- **CN**: 这里检查条件并选择合适的处理路径，以保证特殊情况的正确性。

### Lines 1805-1807
```cpp
1805:     if ((NodeType != isl_schedule_node_band) &&
1806:         (NodeType != isl_schedule_node_mark))
1807:       return false;
```
- **EN**: Checks a condition and selects the appropriate handling path, preserving correctness for special cases.
- **CN**: 这里检查条件并选择合适的处理路径，以保证特殊情况的正确性。

### Lines 1809-1809
```cpp
1809:     Node = Node.parent();
```
- **EN**: Introduces or continues `parent`, a routine that carries out one step of this file's analysis or transformation pipeline.
- **CN**: 这里引入或继续 `parent`，它是该文件分析或变换流水线中的一个步骤函数。

### Lines 1810-1810
```cpp
1810:     NodeType = isl_schedule_node_get_type(Node.get());
```
- **EN**: Introduces or continues `isl_schedule_node_get_type`, a routine that carries out one step of this file's analysis or transformation pipeline.
- **CN**: 这里引入或继续 `isl_schedule_node_get_type`，它是该文件分析或变换流水线中的一个步骤函数。

### Lines 1811-1811
```cpp
1811:   }
```
- **EN**: Closes the current block or scope and hands control back to the surrounding context.
- **CN**: 这里关闭当前代码块或作用域，并将控制流交还给外层上下文。

### Lines 1813-1813
```cpp
1813:   isl::map PartialScheduleMap = isl::map::from_union_map(PartialSchedule);
```
- **EN**: Introduces or continues `isl::map::from_union_map`, a routine that carries out one step of this file's analysis or transformation pipeline.
- **CN**: 这里引入或继续 `isl::map::from_union_map`，它是该文件分析或变换流水线中的一个步骤函数。

### Lines 1814-1815
```cpp
1814:   if (containsTCInfoTy(PartialScheduleMap, D, TCI, isl::set(Domain)))
1815:     return true;
```
- **EN**: Checks a condition and selects the appropriate handling path, preserving correctness for special cases.
- **CN**: 这里检查条件并选择合适的处理路径，以保证特殊情况的正确性。

### Lines 1817-1817
```cpp
1817:   return false;
```
- **EN**: Returns the computed value or forwards the current result to the caller.
- **CN**: 这里返回计算结果，或将当前结果继续传递给调用方。

### Lines 1818-1818
```cpp
1818: }
```
- **EN**: Closes the current block or scope and hands control back to the surrounding context.
- **CN**: 这里关闭当前代码块或作用域，并将控制流交还给外层上下文。

### Lines 1820-1820
```cpp
1820: } // namespace
```
- **EN**: Closes the current block or scope and hands control back to the surrounding context.
- **CN**: 这里关闭当前代码块或作用域，并将控制流交还给外层上下文。

### Lines 1822-1835
```cpp
1822: isl::schedule_node
1823: polly::tryOptimizeMatMulPattern(isl::schedule_node Node,
1824:                                 const llvm::TargetTransformInfo *TTI,
1825:                                 const Dependences *D) {
1826:   TCInfoTy TCI;
1827:   if (PMBasedTCOpts && isTCPattern(Node, D, TCI))
1828:     POLLY_DEBUG(dbgs() << "The tensor contraction pattern was detected\n");
1829:   MatMulInfoTy MMI;
1830:   if (PMBasedMMMOpts && isMatrMultPattern(Node, D, MMI)) {
1831:     POLLY_DEBUG(dbgs() << "The matrix multiplication pattern was detected\n");
1832:     return optimizeMatMulPattern(Node, TTI, MMI);
1833:   }
1834:   return {};
1835: }
```
- **EN**: Checks a condition and selects the appropriate handling path, preserving correctness for special cases.
- **CN**: 这里检查条件并选择合适的处理路径，以保证特殊情况的正确性。

## Key Concepts / 关键概念

- **Matmul pattern matching** / **矩阵乘法模式匹配**
- **Cache-aware tiling** / **缓存感知分块**
- **Schedule rewriting** / **调度重写**
- **ISL set/map modeling** / **ISL 集合/映射建模**
- **Schedule manipulation** / **调度操作**
- **Memory/access reasoning** / **内存/访问推理**

## Dependencies / 依赖关系

- **EN**: Polly headers such as polly/MatmulOptimizer.h, polly/DependenceInfo.h, polly/Options.h, polly/ScheduleTreeTransform.h
- **CN**: Polly 头文件，例如 polly/MatmulOptimizer.h, polly/DependenceInfo.h, polly/Options.h, polly/ScheduleTreeTransform.h
- **EN**: LLVM infrastructure headers such as llvm/ADT/ArrayRef.h, llvm/ADT/DenseSet.h, llvm/ADT/Sequence.h, llvm/ADT/SetOperations.h
- **CN**: LLVM 基础设施头文件，例如 llvm/ADT/ArrayRef.h, llvm/ADT/DenseSet.h, llvm/ADT/Sequence.h, llvm/ADT/SetOperations.h
- **EN**: ISL interfaces such as isl/ctx.h, isl/schedule_node.h, isl/schedule_type.h, isl/union_map.h
- **CN**: ISL 接口，例如 isl/ctx.h, isl/schedule_node.h, isl/schedule_type.h, isl/union_map.h
- **EN**: Standard library facilities such as algorithm, cassert, cmath, cstdint
- **CN**: 标准库能力，例如 algorithm, cassert, cmath, cstdint
