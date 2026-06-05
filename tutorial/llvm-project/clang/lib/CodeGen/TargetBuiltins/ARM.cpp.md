# ARM.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `clang/lib/CodeGen/TargetBuiltins/ARM.cpp`
- **Repository / 仓库**: `llvm-project` (`/root/xw/llvm-project`)
- **Purpose (EN) / 目的（英文）**: Implements builtin handling and lowering logic for the ARM backend.
- **Purpose (CN) / 目的（中文）**: 实现 ARM 后端的内建函数处理与降级逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-30
```cpp
 1: //===---------- ARM.cpp - Emit LLVM Code for builtins ---------------------===//
 2: //
 3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
 4: // See https://llvm.org/LICENSE.txt for license information.
 5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
 6: //
 7: //===----------------------------------------------------------------------===//
 8: //
 9: // This contains code to emit Builtin calls as LLVM code.
10: //
11: //===----------------------------------------------------------------------===//
12: 
13: #include "ABIInfo.h"
14: #include "CGBuiltin.h"
15: #include "CGDebugInfo.h"
16: #include "TargetInfo.h"
17: #include "clang/Basic/AArch64CodeGenUtils.h"
18: #include "clang/Basic/TargetBuiltins.h"
19: #include "llvm/IR/InlineAsm.h"
20: #include "llvm/IR/IntrinsicsAArch64.h"
21: #include "llvm/IR/IntrinsicsARM.h"
22: #include "llvm/IR/IntrinsicsBPF.h"
23: #include "llvm/TargetParser/AArch64TargetParser.h"
24: 
25: #include <numeric>
26: 
27: using namespace clang;
28: using namespace CodeGen;
29: using namespace llvm;
30: using namespace clang::aarch64;
```
- **EN**: This block imports local CodeGen headers `ABIInfo.h`, `CGBuiltin.h`, `CGDebugInfo.h`, and 1 more; Clang headers `clang/Basic/AArch64CodeGenUtils.h`, `clang/Basic/TargetBuiltins.h`; LLVM headers `llvm/IR/InlineAsm.h`, `llvm/IR/IntrinsicsAArch64.h`, `llvm/IR/IntrinsicsARM.h`, and 2 more; other headers `numeric`; opens or references namespaces `clang`, `CodeGen`, `llvm`; contains preprocessor structure that shapes compilation of this unit.
- **CN**: 该代码块引入 本地 CodeGen 头文件 `ABIInfo.h`, `CGBuiltin.h`, `CGDebugInfo.h`, and 1 more；Clang 头文件 `clang/Basic/AArch64CodeGenUtils.h`, `clang/Basic/TargetBuiltins.h`；LLVM 头文件 `llvm/IR/InlineAsm.h`, `llvm/IR/IntrinsicsAArch64.h`, `llvm/IR/IntrinsicsARM.h`, and 2 more；其他头文件 `numeric`；打开或引用命名空间 `clang`, `CodeGen`, `llvm`；包含影响本编译单元构建方式的预处理结构。

### Lines 31-60
```cpp
31: 
32: static std::optional<CodeGenFunction::MSVCIntrin>
33: translateAarch64ToMsvcIntrin(unsigned BuiltinID) {
34:   using MSVCIntrin = CodeGenFunction::MSVCIntrin;
35:   switch (BuiltinID) {
36:   default:
37:     return std::nullopt;
38:   case clang::AArch64::BI_BitScanForward:
39:   case clang::AArch64::BI_BitScanForward64:
40:     return MSVCIntrin::_BitScanForward;
41:   case clang::AArch64::BI_BitScanReverse:
42:   case clang::AArch64::BI_BitScanReverse64:
43:     return MSVCIntrin::_BitScanReverse;
44:   case clang::AArch64::BI_InterlockedAnd64:
45:     return MSVCIntrin::_InterlockedAnd;
46:   case clang::AArch64::BI_InterlockedExchange64:
47:     return MSVCIntrin::_InterlockedExchange;
48:   case clang::AArch64::BI_InterlockedExchangeAdd64:
49:     return MSVCIntrin::_InterlockedExchangeAdd;
50:   case clang::AArch64::BI_InterlockedExchangeSub64:
51:     return MSVCIntrin::_InterlockedExchangeSub;
52:   case clang::AArch64::BI_InterlockedOr64:
53:     return MSVCIntrin::_InterlockedOr;
54:   case clang::AArch64::BI_InterlockedXor64:
55:     return MSVCIntrin::_InterlockedXor;
56:   case clang::AArch64::BI_InterlockedDecrement64:
57:     return MSVCIntrin::_InterlockedDecrement;
58:   case clang::AArch64::BI_InterlockedIncrement64:
59:     return MSVCIntrin::_InterlockedIncrement;
60:   case clang::AArch64::BI_InterlockedExchangeAdd8_acq:
```
- **EN**: This block defines callable entry points like `translateAarch64ToMsvcIntrin`; uses control flow (switch, case) to specialize target builtin lowering.
- **CN**: 该代码块定义可调用入口，例如 `translateAarch64ToMsvcIntrin`；通过控制流（switch, case）细化 目标内建函数降级 行为。

### Lines 61-90
```cpp
61:   case clang::AArch64::BI_InterlockedExchangeAdd16_acq:
62:   case clang::AArch64::BI_InterlockedExchangeAdd_acq:
63:   case clang::AArch64::BI_InterlockedExchangeAdd64_acq:
64:     return MSVCIntrin::_InterlockedExchangeAdd_acq;
65:   case clang::AArch64::BI_InterlockedExchangeAdd8_rel:
66:   case clang::AArch64::BI_InterlockedExchangeAdd16_rel:
67:   case clang::AArch64::BI_InterlockedExchangeAdd_rel:
68:   case clang::AArch64::BI_InterlockedExchangeAdd64_rel:
69:     return MSVCIntrin::_InterlockedExchangeAdd_rel;
70:   case clang::AArch64::BI_InterlockedExchangeAdd8_nf:
71:   case clang::AArch64::BI_InterlockedExchangeAdd16_nf:
72:   case clang::AArch64::BI_InterlockedExchangeAdd_nf:
73:   case clang::AArch64::BI_InterlockedExchangeAdd64_nf:
74:     return MSVCIntrin::_InterlockedExchangeAdd_nf;
75:   case clang::AArch64::BI_InterlockedExchange8_acq:
76:   case clang::AArch64::BI_InterlockedExchange16_acq:
77:   case clang::AArch64::BI_InterlockedExchange_acq:
78:   case clang::AArch64::BI_InterlockedExchange64_acq:
79:   case clang::AArch64::BI_InterlockedExchangePointer_acq:
80:     return MSVCIntrin::_InterlockedExchange_acq;
81:   case clang::AArch64::BI_InterlockedExchange8_rel:
82:   case clang::AArch64::BI_InterlockedExchange16_rel:
83:   case clang::AArch64::BI_InterlockedExchange_rel:
84:   case clang::AArch64::BI_InterlockedExchange64_rel:
85:   case clang::AArch64::BI_InterlockedExchangePointer_rel:
86:     return MSVCIntrin::_InterlockedExchange_rel;
87:   case clang::AArch64::BI_InterlockedExchange8_nf:
88:   case clang::AArch64::BI_InterlockedExchange16_nf:
89:   case clang::AArch64::BI_InterlockedExchange_nf:
90:   case clang::AArch64::BI_InterlockedExchange64_nf:
```
- **EN**: This block uses control flow (case) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（case）细化 目标内建函数降级 行为。

### Lines 91-120
```cpp
 91:   case clang::AArch64::BI_InterlockedExchangePointer_nf:
 92:     return MSVCIntrin::_InterlockedExchange_nf;
 93:   case clang::AArch64::BI_InterlockedCompareExchange8_acq:
 94:   case clang::AArch64::BI_InterlockedCompareExchange16_acq:
 95:   case clang::AArch64::BI_InterlockedCompareExchange_acq:
 96:   case clang::AArch64::BI_InterlockedCompareExchange64_acq:
 97:   case clang::AArch64::BI_InterlockedCompareExchangePointer_acq:
 98:     return MSVCIntrin::_InterlockedCompareExchange_acq;
 99:   case clang::AArch64::BI_InterlockedCompareExchange8_rel:
100:   case clang::AArch64::BI_InterlockedCompareExchange16_rel:
101:   case clang::AArch64::BI_InterlockedCompareExchange_rel:
102:   case clang::AArch64::BI_InterlockedCompareExchange64_rel:
103:   case clang::AArch64::BI_InterlockedCompareExchangePointer_rel:
104:     return MSVCIntrin::_InterlockedCompareExchange_rel;
105:   case clang::AArch64::BI_InterlockedCompareExchange8_nf:
106:   case clang::AArch64::BI_InterlockedCompareExchange16_nf:
107:   case clang::AArch64::BI_InterlockedCompareExchange_nf:
108:   case clang::AArch64::BI_InterlockedCompareExchange64_nf:
109:     return MSVCIntrin::_InterlockedCompareExchange_nf;
110:   case clang::AArch64::BI_InterlockedCompareExchange128:
111:     return MSVCIntrin::_InterlockedCompareExchange128;
112:   case clang::AArch64::BI_InterlockedCompareExchange128_acq:
113:     return MSVCIntrin::_InterlockedCompareExchange128_acq;
114:   case clang::AArch64::BI_InterlockedCompareExchange128_nf:
115:     return MSVCIntrin::_InterlockedCompareExchange128_nf;
116:   case clang::AArch64::BI_InterlockedCompareExchange128_rel:
117:     return MSVCIntrin::_InterlockedCompareExchange128_rel;
118:   case clang::AArch64::BI_InterlockedOr8_acq:
119:   case clang::AArch64::BI_InterlockedOr16_acq:
120:   case clang::AArch64::BI_InterlockedOr_acq:
```
- **EN**: This block uses control flow (case) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（case）细化 目标内建函数降级 行为。

### Lines 121-150
```cpp
121:   case clang::AArch64::BI_InterlockedOr64_acq:
122:     return MSVCIntrin::_InterlockedOr_acq;
123:   case clang::AArch64::BI_InterlockedOr8_rel:
124:   case clang::AArch64::BI_InterlockedOr16_rel:
125:   case clang::AArch64::BI_InterlockedOr_rel:
126:   case clang::AArch64::BI_InterlockedOr64_rel:
127:     return MSVCIntrin::_InterlockedOr_rel;
128:   case clang::AArch64::BI_InterlockedOr8_nf:
129:   case clang::AArch64::BI_InterlockedOr16_nf:
130:   case clang::AArch64::BI_InterlockedOr_nf:
131:   case clang::AArch64::BI_InterlockedOr64_nf:
132:     return MSVCIntrin::_InterlockedOr_nf;
133:   case clang::AArch64::BI_InterlockedXor8_acq:
134:   case clang::AArch64::BI_InterlockedXor16_acq:
135:   case clang::AArch64::BI_InterlockedXor_acq:
136:   case clang::AArch64::BI_InterlockedXor64_acq:
137:     return MSVCIntrin::_InterlockedXor_acq;
138:   case clang::AArch64::BI_InterlockedXor8_rel:
139:   case clang::AArch64::BI_InterlockedXor16_rel:
140:   case clang::AArch64::BI_InterlockedXor_rel:
141:   case clang::AArch64::BI_InterlockedXor64_rel:
142:     return MSVCIntrin::_InterlockedXor_rel;
143:   case clang::AArch64::BI_InterlockedXor8_nf:
144:   case clang::AArch64::BI_InterlockedXor16_nf:
145:   case clang::AArch64::BI_InterlockedXor_nf:
146:   case clang::AArch64::BI_InterlockedXor64_nf:
147:     return MSVCIntrin::_InterlockedXor_nf;
148:   case clang::AArch64::BI_InterlockedAnd8_acq:
149:   case clang::AArch64::BI_InterlockedAnd16_acq:
150:   case clang::AArch64::BI_InterlockedAnd_acq:
```
- **EN**: This block uses control flow (case) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（case）细化 目标内建函数降级 行为。

### Lines 151-180
```cpp
151:   case clang::AArch64::BI_InterlockedAnd64_acq:
152:     return MSVCIntrin::_InterlockedAnd_acq;
153:   case clang::AArch64::BI_InterlockedAnd8_rel:
154:   case clang::AArch64::BI_InterlockedAnd16_rel:
155:   case clang::AArch64::BI_InterlockedAnd_rel:
156:   case clang::AArch64::BI_InterlockedAnd64_rel:
157:     return MSVCIntrin::_InterlockedAnd_rel;
158:   case clang::AArch64::BI_InterlockedAnd8_nf:
159:   case clang::AArch64::BI_InterlockedAnd16_nf:
160:   case clang::AArch64::BI_InterlockedAnd_nf:
161:   case clang::AArch64::BI_InterlockedAnd64_nf:
162:     return MSVCIntrin::_InterlockedAnd_nf;
163:   case clang::AArch64::BI_InterlockedIncrement16_acq:
164:   case clang::AArch64::BI_InterlockedIncrement_acq:
165:   case clang::AArch64::BI_InterlockedIncrement64_acq:
166:     return MSVCIntrin::_InterlockedIncrement_acq;
167:   case clang::AArch64::BI_InterlockedIncrement16_rel:
168:   case clang::AArch64::BI_InterlockedIncrement_rel:
169:   case clang::AArch64::BI_InterlockedIncrement64_rel:
170:     return MSVCIntrin::_InterlockedIncrement_rel;
171:   case clang::AArch64::BI_InterlockedIncrement16_nf:
172:   case clang::AArch64::BI_InterlockedIncrement_nf:
173:   case clang::AArch64::BI_InterlockedIncrement64_nf:
174:     return MSVCIntrin::_InterlockedIncrement_nf;
175:   case clang::AArch64::BI_InterlockedDecrement16_acq:
176:   case clang::AArch64::BI_InterlockedDecrement_acq:
177:   case clang::AArch64::BI_InterlockedDecrement64_acq:
178:     return MSVCIntrin::_InterlockedDecrement_acq;
179:   case clang::AArch64::BI_InterlockedDecrement16_rel:
180:   case clang::AArch64::BI_InterlockedDecrement_rel:
```
- **EN**: This block uses control flow (case) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（case）细化 目标内建函数降级 行为。

### Lines 181-210
```cpp
181:   case clang::AArch64::BI_InterlockedDecrement64_rel:
182:     return MSVCIntrin::_InterlockedDecrement_rel;
183:   case clang::AArch64::BI_InterlockedDecrement16_nf:
184:   case clang::AArch64::BI_InterlockedDecrement_nf:
185:   case clang::AArch64::BI_InterlockedDecrement64_nf:
186:     return MSVCIntrin::_InterlockedDecrement_nf;
187:   }
188:   llvm_unreachable("must return from switch");
189: }
190: 
191: static std::optional<CodeGenFunction::MSVCIntrin>
192: translateArmToMsvcIntrin(unsigned BuiltinID) {
193:   using MSVCIntrin = CodeGenFunction::MSVCIntrin;
194:   switch (BuiltinID) {
195:   default:
196:     return std::nullopt;
197:   case clang::ARM::BI_BitScanForward:
198:   case clang::ARM::BI_BitScanForward64:
199:     return MSVCIntrin::_BitScanForward;
200:   case clang::ARM::BI_BitScanReverse:
201:   case clang::ARM::BI_BitScanReverse64:
202:     return MSVCIntrin::_BitScanReverse;
203:   case clang::ARM::BI_InterlockedAnd64:
204:     return MSVCIntrin::_InterlockedAnd;
205:   case clang::ARM::BI_InterlockedExchange64:
206:     return MSVCIntrin::_InterlockedExchange;
207:   case clang::ARM::BI_InterlockedExchangeAdd64:
208:     return MSVCIntrin::_InterlockedExchangeAdd;
209:   case clang::ARM::BI_InterlockedExchangeSub64:
210:     return MSVCIntrin::_InterlockedExchangeSub;
```
- **EN**: This block defines callable entry points like `translateArmToMsvcIntrin`; uses control flow (switch, case) to specialize target builtin lowering; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块定义可调用入口，例如 `translateArmToMsvcIntrin`；通过控制流（switch, case）细化 目标内建函数降级 行为；使用断言或不可达标记保护关键不变量。

### Lines 211-240
```cpp
211:   case clang::ARM::BI_InterlockedOr64:
212:     return MSVCIntrin::_InterlockedOr;
213:   case clang::ARM::BI_InterlockedXor64:
214:     return MSVCIntrin::_InterlockedXor;
215:   case clang::ARM::BI_InterlockedDecrement64:
216:     return MSVCIntrin::_InterlockedDecrement;
217:   case clang::ARM::BI_InterlockedIncrement64:
218:     return MSVCIntrin::_InterlockedIncrement;
219:   case clang::ARM::BI_InterlockedExchangeAdd8_acq:
220:   case clang::ARM::BI_InterlockedExchangeAdd16_acq:
221:   case clang::ARM::BI_InterlockedExchangeAdd_acq:
222:   case clang::ARM::BI_InterlockedExchangeAdd64_acq:
223:     return MSVCIntrin::_InterlockedExchangeAdd_acq;
224:   case clang::ARM::BI_InterlockedExchangeAdd8_rel:
225:   case clang::ARM::BI_InterlockedExchangeAdd16_rel:
226:   case clang::ARM::BI_InterlockedExchangeAdd_rel:
227:   case clang::ARM::BI_InterlockedExchangeAdd64_rel:
228:     return MSVCIntrin::_InterlockedExchangeAdd_rel;
229:   case clang::ARM::BI_InterlockedExchangeAdd8_nf:
230:   case clang::ARM::BI_InterlockedExchangeAdd16_nf:
231:   case clang::ARM::BI_InterlockedExchangeAdd_nf:
232:   case clang::ARM::BI_InterlockedExchangeAdd64_nf:
233:     return MSVCIntrin::_InterlockedExchangeAdd_nf;
234:   case clang::ARM::BI_InterlockedExchange8_acq:
235:   case clang::ARM::BI_InterlockedExchange16_acq:
236:   case clang::ARM::BI_InterlockedExchange_acq:
237:   case clang::ARM::BI_InterlockedExchange64_acq:
238:   case clang::ARM::BI_InterlockedExchangePointer_acq:
239:     return MSVCIntrin::_InterlockedExchange_acq;
240:   case clang::ARM::BI_InterlockedExchange8_rel:
```
- **EN**: This block uses control flow (case) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（case）细化 目标内建函数降级 行为。

### Lines 241-270
```cpp
241:   case clang::ARM::BI_InterlockedExchange16_rel:
242:   case clang::ARM::BI_InterlockedExchange_rel:
243:   case clang::ARM::BI_InterlockedExchange64_rel:
244:   case clang::ARM::BI_InterlockedExchangePointer_rel:
245:     return MSVCIntrin::_InterlockedExchange_rel;
246:   case clang::ARM::BI_InterlockedExchange8_nf:
247:   case clang::ARM::BI_InterlockedExchange16_nf:
248:   case clang::ARM::BI_InterlockedExchange_nf:
249:   case clang::ARM::BI_InterlockedExchange64_nf:
250:   case clang::ARM::BI_InterlockedExchangePointer_nf:
251:     return MSVCIntrin::_InterlockedExchange_nf;
252:   case clang::ARM::BI_InterlockedCompareExchange8_acq:
253:   case clang::ARM::BI_InterlockedCompareExchange16_acq:
254:   case clang::ARM::BI_InterlockedCompareExchange_acq:
255:   case clang::ARM::BI_InterlockedCompareExchange64_acq:
256:   case clang::ARM::BI_InterlockedCompareExchangePointer_acq:
257:     return MSVCIntrin::_InterlockedCompareExchange_acq;
258:   case clang::ARM::BI_InterlockedCompareExchange8_rel:
259:   case clang::ARM::BI_InterlockedCompareExchange16_rel:
260:   case clang::ARM::BI_InterlockedCompareExchange_rel:
261:   case clang::ARM::BI_InterlockedCompareExchange64_rel:
262:   case clang::ARM::BI_InterlockedCompareExchangePointer_rel:
263:     return MSVCIntrin::_InterlockedCompareExchange_rel;
264:   case clang::ARM::BI_InterlockedCompareExchange8_nf:
265:   case clang::ARM::BI_InterlockedCompareExchange16_nf:
266:   case clang::ARM::BI_InterlockedCompareExchange_nf:
267:   case clang::ARM::BI_InterlockedCompareExchange64_nf:
268:     return MSVCIntrin::_InterlockedCompareExchange_nf;
269:   case clang::ARM::BI_InterlockedOr8_acq:
270:   case clang::ARM::BI_InterlockedOr16_acq:
```
- **EN**: This block uses control flow (case) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（case）细化 目标内建函数降级 行为。

### Lines 271-300
```cpp
271:   case clang::ARM::BI_InterlockedOr_acq:
272:   case clang::ARM::BI_InterlockedOr64_acq:
273:     return MSVCIntrin::_InterlockedOr_acq;
274:   case clang::ARM::BI_InterlockedOr8_rel:
275:   case clang::ARM::BI_InterlockedOr16_rel:
276:   case clang::ARM::BI_InterlockedOr_rel:
277:   case clang::ARM::BI_InterlockedOr64_rel:
278:     return MSVCIntrin::_InterlockedOr_rel;
279:   case clang::ARM::BI_InterlockedOr8_nf:
280:   case clang::ARM::BI_InterlockedOr16_nf:
281:   case clang::ARM::BI_InterlockedOr_nf:
282:   case clang::ARM::BI_InterlockedOr64_nf:
283:     return MSVCIntrin::_InterlockedOr_nf;
284:   case clang::ARM::BI_InterlockedXor8_acq:
285:   case clang::ARM::BI_InterlockedXor16_acq:
286:   case clang::ARM::BI_InterlockedXor_acq:
287:   case clang::ARM::BI_InterlockedXor64_acq:
288:     return MSVCIntrin::_InterlockedXor_acq;
289:   case clang::ARM::BI_InterlockedXor8_rel:
290:   case clang::ARM::BI_InterlockedXor16_rel:
291:   case clang::ARM::BI_InterlockedXor_rel:
292:   case clang::ARM::BI_InterlockedXor64_rel:
293:     return MSVCIntrin::_InterlockedXor_rel;
294:   case clang::ARM::BI_InterlockedXor8_nf:
295:   case clang::ARM::BI_InterlockedXor16_nf:
296:   case clang::ARM::BI_InterlockedXor_nf:
297:   case clang::ARM::BI_InterlockedXor64_nf:
298:     return MSVCIntrin::_InterlockedXor_nf;
299:   case clang::ARM::BI_InterlockedAnd8_acq:
300:   case clang::ARM::BI_InterlockedAnd16_acq:
```
- **EN**: This block uses control flow (case) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（case）细化 目标内建函数降级 行为。

### Lines 301-330
```cpp
301:   case clang::ARM::BI_InterlockedAnd_acq:
302:   case clang::ARM::BI_InterlockedAnd64_acq:
303:     return MSVCIntrin::_InterlockedAnd_acq;
304:   case clang::ARM::BI_InterlockedAnd8_rel:
305:   case clang::ARM::BI_InterlockedAnd16_rel:
306:   case clang::ARM::BI_InterlockedAnd_rel:
307:   case clang::ARM::BI_InterlockedAnd64_rel:
308:     return MSVCIntrin::_InterlockedAnd_rel;
309:   case clang::ARM::BI_InterlockedAnd8_nf:
310:   case clang::ARM::BI_InterlockedAnd16_nf:
311:   case clang::ARM::BI_InterlockedAnd_nf:
312:   case clang::ARM::BI_InterlockedAnd64_nf:
313:     return MSVCIntrin::_InterlockedAnd_nf;
314:   case clang::ARM::BI_InterlockedIncrement16_acq:
315:   case clang::ARM::BI_InterlockedIncrement_acq:
316:   case clang::ARM::BI_InterlockedIncrement64_acq:
317:     return MSVCIntrin::_InterlockedIncrement_acq;
318:   case clang::ARM::BI_InterlockedIncrement16_rel:
319:   case clang::ARM::BI_InterlockedIncrement_rel:
320:   case clang::ARM::BI_InterlockedIncrement64_rel:
321:     return MSVCIntrin::_InterlockedIncrement_rel;
322:   case clang::ARM::BI_InterlockedIncrement16_nf:
323:   case clang::ARM::BI_InterlockedIncrement_nf:
324:   case clang::ARM::BI_InterlockedIncrement64_nf:
325:     return MSVCIntrin::_InterlockedIncrement_nf;
326:   case clang::ARM::BI_InterlockedDecrement16_acq:
327:   case clang::ARM::BI_InterlockedDecrement_acq:
328:   case clang::ARM::BI_InterlockedDecrement64_acq:
329:     return MSVCIntrin::_InterlockedDecrement_acq;
330:   case clang::ARM::BI_InterlockedDecrement16_rel:
```
- **EN**: This block uses control flow (case) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（case）细化 目标内建函数降级 行为。

### Lines 331-360
```cpp
331:   case clang::ARM::BI_InterlockedDecrement_rel:
332:   case clang::ARM::BI_InterlockedDecrement64_rel:
333:     return MSVCIntrin::_InterlockedDecrement_rel;
334:   case clang::ARM::BI_InterlockedDecrement16_nf:
335:   case clang::ARM::BI_InterlockedDecrement_nf:
336:   case clang::ARM::BI_InterlockedDecrement64_nf:
337:     return MSVCIntrin::_InterlockedDecrement_nf;
338:   }
339:   llvm_unreachable("must return from switch");
340: }
341: 
342: // Emit an intrinsic where all operands are of the same type as the result.
343: // Depending on mode, this may be a constrained floating-point intrinsic.
344: static Value *emitCallMaybeConstrainedFPBuiltin(CodeGenFunction &CGF,
345:                                                 unsigned IntrinsicID,
346:                                                 unsigned ConstrainedIntrinsicID,
347:                                                 llvm::Type *Ty,
348:                                                 ArrayRef<Value *> Args) {
349:   Function *F;
350:   if (CGF.Builder.getIsFPConstrained())
351:     F = CGF.CGM.getIntrinsic(ConstrainedIntrinsicID, Ty);
352:   else
353:     F = CGF.CGM.getIntrinsic(IntrinsicID, Ty);
354: 
355:   if (CGF.Builder.getIsFPConstrained())
356:     return CGF.Builder.CreateConstrainedFPCall(F, Args);
357: 
358:   return CGF.Builder.CreateCall(F, Args);
359: }
360: 
```
- **EN**: This block uses control flow (if, switch, case) to specialize target builtin lowering; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块通过控制流（if, switch, case）细化 目标内建函数降级 行为；使用断言或不可达标记保护关键不变量。

### Lines 361-390
```cpp
361: static llvm::FixedVectorType *GetNeonType(CodeGenFunction *CGF,
362:                                           NeonTypeFlags TypeFlags,
363:                                           bool HasFastHalfType = true,
364:                                           bool V1Ty = false,
365:                                           bool AllowBFloatArgsAndRet = true) {
366:   int IsQuad = TypeFlags.isQuad();
367:   switch (TypeFlags.getEltType()) {
368:   case NeonTypeFlags::Int8:
369:   case NeonTypeFlags::Poly8:
370:   case NeonTypeFlags::MFloat8:
371:     return llvm::FixedVectorType::get(CGF->Int8Ty, V1Ty ? 1 : (8 << IsQuad));
372:   case NeonTypeFlags::Int16:
373:   case NeonTypeFlags::Poly16:
374:     return llvm::FixedVectorType::get(CGF->Int16Ty, V1Ty ? 1 : (4 << IsQuad));
375:   case NeonTypeFlags::BFloat16:
376:     if (AllowBFloatArgsAndRet)
377:       return llvm::FixedVectorType::get(CGF->BFloatTy, V1Ty ? 1 : (4 << IsQuad));
378:     return llvm::FixedVectorType::get(CGF->Int16Ty, V1Ty ? 1 : (4 << IsQuad));
379:   case NeonTypeFlags::Float16:
380:     if (HasFastHalfType)
381:       return llvm::FixedVectorType::get(CGF->HalfTy, V1Ty ? 1 : (4 << IsQuad));
382:     return llvm::FixedVectorType::get(CGF->Int16Ty, V1Ty ? 1 : (4 << IsQuad));
383:   case NeonTypeFlags::Int32:
384:     return llvm::FixedVectorType::get(CGF->Int32Ty, V1Ty ? 1 : (2 << IsQuad));
385:   case NeonTypeFlags::Int64:
386:   case NeonTypeFlags::Poly64:
387:     return llvm::FixedVectorType::get(CGF->Int64Ty, V1Ty ? 1 : (1 << IsQuad));
388:   case NeonTypeFlags::Poly128:
389:     // FIXME: i128 and f128 doesn't get fully support in Clang and llvm.
390:     // There is a lot of i128 and f128 API missing.
```
- **EN**: This block defines callable entry points like `get`; uses control flow (if, switch, case) to specialize target builtin lowering.
- **CN**: 该代码块定义可调用入口，例如 `get`；通过控制流（if, switch, case）细化 目标内建函数降级 行为。

### Lines 391-420
```cpp
391:     // so we use v16i8 to represent poly128 and get pattern matched.
392:     return llvm::FixedVectorType::get(CGF->Int8Ty, 16);
393:   case NeonTypeFlags::Float32:
394:     return llvm::FixedVectorType::get(CGF->FloatTy, V1Ty ? 1 : (2 << IsQuad));
395:   case NeonTypeFlags::Float64:
396:     return llvm::FixedVectorType::get(CGF->DoubleTy, V1Ty ? 1 : (1 << IsQuad));
397:   }
398:   llvm_unreachable("Unknown vector element type!");
399: }
400: 
401: static llvm::VectorType *GetFloatNeonType(CodeGenFunction *CGF,
402:                                           NeonTypeFlags IntTypeFlags) {
403:   int IsQuad = IntTypeFlags.isQuad();
404:   switch (IntTypeFlags.getEltType()) {
405:   case NeonTypeFlags::Int16:
406:     return llvm::FixedVectorType::get(CGF->HalfTy, (4 << IsQuad));
407:   case NeonTypeFlags::Int32:
408:     return llvm::FixedVectorType::get(CGF->FloatTy, (2 << IsQuad));
409:   case NeonTypeFlags::Int64:
410:     return llvm::FixedVectorType::get(CGF->DoubleTy, (1 << IsQuad));
411:   default:
412:     llvm_unreachable("Type can't be converted to floating-point!");
413:   }
414: }
415: 
416: Value *CodeGenFunction::EmitNeonSplat(Value *V, Constant *C,
417:                                       const ElementCount &Count) {
418:   Value *SV = llvm::ConstantVector::getSplat(Count, C);
419:   return Builder.CreateShuffleVector(V, V, SV, "lane");
420: }
```
- **EN**: This block defines callable entry points like `get`; uses control flow (switch, case) to specialize target builtin lowering; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块定义可调用入口，例如 `get`；通过控制流（switch, case）细化 目标内建函数降级 行为；使用断言或不可达标记保护关键不变量。

### Lines 421-450
```cpp
421: 
422: Value *CodeGenFunction::EmitNeonSplat(Value *V, Constant *C) {
423:   ElementCount EC = cast<llvm::VectorType>(V->getType())->getElementCount();
424:   return EmitNeonSplat(V, C, EC);
425: }
426: 
427: Value *CodeGenFunction::EmitNeonCall(Function *F, SmallVectorImpl<Value*> &Ops,
428:                                      const char *name,
429:                                      unsigned shift, bool rightshift) {
430:   unsigned j = 0;
431:   for (Function::const_arg_iterator ai = F->arg_begin(), ae = F->arg_end();
432:        ai != ae; ++ai, ++j) {
433:     if (F->isConstrainedFPIntrinsic())
434:       if (ai->getType()->isMetadataTy())
435:         continue;
436:     if (shift > 0 && shift == j)
437:       Ops[j] = EmitNeonShiftVector(Ops[j], ai->getType(), rightshift);
438:     else
439:       Ops[j] = Builder.CreateBitCast(Ops[j], ai->getType(), name);
440:   }
441: 
442:   if (F->isConstrainedFPIntrinsic())
443:     return Builder.CreateConstrainedFPCall(F, Ops, name);
444:   return Builder.CreateCall(F, Ops, name);
445: }
446: 
447: Value *CodeGenFunction::EmitFP8NeonCall(unsigned IID,
448:                                         ArrayRef<llvm::Type *> Tys,
449:                                         SmallVectorImpl<Value *> &Ops,
450:                                         const CallExpr *E, const char *name) {
```
- **EN**: This block defines callable entry points like `EmitNeonSplat`; uses control flow (if, for) to specialize target builtin lowering.
- **CN**: 该代码块定义可调用入口，例如 `EmitNeonSplat`；通过控制流（if, for）细化 目标内建函数降级 行为。

### Lines 451-480
```cpp
451:   Builder.CreateCall(CGM.getIntrinsic(Intrinsic::aarch64_set_fpmr),
452:                      Ops.pop_back_val());
453:   return EmitNeonCall(CGM.getIntrinsic(IID, Tys), Ops, name);
454: }
455: 
456: llvm::Value *CodeGenFunction::EmitFP8NeonFDOTCall(
457:     unsigned IID, bool ExtendLaneArg, llvm::Type *RetTy,
458:     SmallVectorImpl<llvm::Value *> &Ops, const CallExpr *E, const char *name) {
459: 
460:   const unsigned ElemCount = Ops[0]->getType()->getPrimitiveSizeInBits() /
461:                              RetTy->getPrimitiveSizeInBits();
462:   llvm::Type *Tys[] = {llvm::FixedVectorType::get(RetTy, ElemCount),
463:                        Ops[1]->getType()};
464:   if (ExtendLaneArg) {
465:     auto *VT = llvm::FixedVectorType::get(Int8Ty, 16);
466:     Ops[2] = Builder.CreateInsertVector(VT, PoisonValue::get(VT), Ops[2],
467:                                         uint64_t(0));
468:   }
469:   return EmitFP8NeonCall(IID, Tys, Ops, E, name);
470: }
471: 
472: llvm::Value *CodeGenFunction::EmitFP8NeonFMLACall(
473:     unsigned IID, bool ExtendLaneArg, llvm::Type *RetTy,
474:     SmallVectorImpl<llvm::Value *> &Ops, const CallExpr *E, const char *name) {
475: 
476:   if (ExtendLaneArg) {
477:     auto *VT = llvm::FixedVectorType::get(Int8Ty, 16);
478:     Ops[2] = Builder.CreateInsertVector(VT, PoisonValue::get(VT), Ops[2],
479:                                         uint64_t(0));
480:   }
```
- **EN**: This block defines callable entry points like `EmitNeonCall`, `uint64_t`, `EmitFP8NeonCall`; uses control flow (if) to specialize target builtin lowering.
- **CN**: 该代码块定义可调用入口，例如 `EmitNeonCall`, `uint64_t`, `EmitFP8NeonCall`；通过控制流（if）细化 目标内建函数降级 行为。

### Lines 481-510
```cpp
481:   const unsigned ElemCount = Ops[0]->getType()->getPrimitiveSizeInBits() /
482:                              RetTy->getPrimitiveSizeInBits();
483:   return EmitFP8NeonCall(IID, {llvm::FixedVectorType::get(RetTy, ElemCount)},
484:                          Ops, E, name);
485: }
486: 
487: Value *CodeGenFunction::EmitNeonShiftVector(Value *V, llvm::Type *Ty,
488:                                             bool neg) {
489:   int SV = cast<ConstantInt>(V)->getSExtValue();
490:   return ConstantInt::getSigned(Ty, neg ? -SV : SV);
491: }
492: 
493: Value *CodeGenFunction::EmitFP8NeonCvtCall(unsigned IID, llvm::Type *Ty0,
494:                                            llvm::Type *Ty1, bool Extract,
495:                                            SmallVectorImpl<llvm::Value *> &Ops,
496:                                            const CallExpr *E,
497:                                            const char *name) {
498:   llvm::Type *Tys[] = {Ty0, Ty1};
499:   if (Extract) {
500:     // Op[0] is mfloat8x16_t, but the intrinsic converts only the lower part of
501:     // the vector.
502:     Tys[1] = llvm::FixedVectorType::get(Int8Ty, 8);
503:     Ops[0] = Builder.CreateExtractVector(Tys[1], Ops[0], uint64_t(0));
504:   }
505:   return EmitFP8NeonCall(IID, Tys, Ops, E, name);
506: }
507: 
508: // Right-shift a vector by a constant.
509: Value *CodeGenFunction::EmitNeonRShiftImm(Value *Vec, Value *Shift,
510:                                           llvm::Type *Ty, bool usgn,
```
- **EN**: This block defines callable entry points like `getSigned`, `EmitFP8NeonCall`; uses control flow (if) to specialize target builtin lowering.
- **CN**: 该代码块定义可调用入口，例如 `getSigned`, `EmitFP8NeonCall`；通过控制流（if）细化 目标内建函数降级 行为。

### Lines 511-540
```cpp
511:                                           const char *name) {
512:   llvm::VectorType *VTy = cast<llvm::VectorType>(Ty);
513: 
514:   int ShiftAmt = cast<ConstantInt>(Shift)->getSExtValue();
515:   int EltSize = VTy->getScalarSizeInBits();
516: 
517:   Vec = Builder.CreateBitCast(Vec, Ty);
518: 
519:   // lshr/ashr are undefined when the shift amount is equal to the vector
520:   // element size.
521:   if (ShiftAmt == EltSize) {
522:     if (usgn) {
523:       // Right-shifting an unsigned value by its size yields 0.
524:       return llvm::ConstantAggregateZero::get(VTy);
525:     } else {
526:       // Right-shifting a signed value by its size is equivalent
527:       // to a shift of size-1.
528:       --ShiftAmt;
529:       Shift = ConstantInt::get(VTy->getElementType(), ShiftAmt);
530:     }
531:   }
532: 
533:   Shift = EmitNeonShiftVector(Shift, Ty, false);
534:   if (usgn)
535:     return Builder.CreateLShr(Vec, Shift, name);
536:   return Builder.CreateAShr(Vec, Shift, name);
537: }
538: 
539: // clang-format off
540: static const ARMVectorIntrinsicInfo ARMSIMDIntrinsicMap [] = {
```
- **EN**: This block defines callable entry points like `get`; uses control flow (if) to specialize target builtin lowering.
- **CN**: 该代码块定义可调用入口，例如 `get`；通过控制流（if）细化 目标内建函数降级 行为。

### Lines 541-570
```cpp
541:   NEONMAP1(__a32_vcvt_bf16_f32, arm_neon_vcvtfp2bf, 0),
542:   NEONMAP0(splat_lane_v),
543:   NEONMAP0(splat_laneq_v),
544:   NEONMAP0(splatq_lane_v),
545:   NEONMAP0(splatq_laneq_v),
546:   NEONMAP2(vabd_v, arm_neon_vabdu, arm_neon_vabds, Add1ArgType | UnsignedAlts),
547:   NEONMAP2(vabdq_v, arm_neon_vabdu, arm_neon_vabds, Add1ArgType | UnsignedAlts),
548:   NEONMAP1(vabs_v, arm_neon_vabs, 0),
549:   NEONMAP1(vabsq_v, arm_neon_vabs, 0),
550:   NEONMAP0(vadd_v),
551:   NEONMAP0(vaddhn_v),
552:   NEONMAP0(vaddq_v),
553:   NEONMAP1(vaesdq_u8, arm_neon_aesd, 0),
554:   NEONMAP1(vaeseq_u8, arm_neon_aese, 0),
555:   NEONMAP1(vaesimcq_u8, arm_neon_aesimc, 0),
556:   NEONMAP1(vaesmcq_u8, arm_neon_aesmc, 0),
557:   NEONMAP1(vbfdot_f32, arm_neon_bfdot, 0),
558:   NEONMAP1(vbfdotq_f32, arm_neon_bfdot, 0),
559:   NEONMAP1(vbfmlalbq_f32, arm_neon_bfmlalb, 0),
560:   NEONMAP1(vbfmlaltq_f32, arm_neon_bfmlalt, 0),
561:   NEONMAP1(vbfmmlaq_f32, arm_neon_bfmmla, 0),
562:   NEONMAP1(vbsl_v, arm_neon_vbsl, AddRetType),
563:   NEONMAP1(vbslq_v, arm_neon_vbsl, AddRetType),
564:   NEONMAP1(vcadd_rot270_f16, arm_neon_vcadd_rot270, Add1ArgType),
565:   NEONMAP1(vcadd_rot270_f32, arm_neon_vcadd_rot270, Add1ArgType),
566:   NEONMAP1(vcadd_rot90_f16, arm_neon_vcadd_rot90, Add1ArgType),
567:   NEONMAP1(vcadd_rot90_f32, arm_neon_vcadd_rot90, Add1ArgType),
568:   NEONMAP1(vcaddq_rot270_f16, arm_neon_vcadd_rot270, Add1ArgType),
569:   NEONMAP1(vcaddq_rot270_f32, arm_neon_vcadd_rot270, Add1ArgType),
570:   NEONMAP1(vcaddq_rot270_f64, arm_neon_vcadd_rot270, Add1ArgType),
```
- **EN**: This block provides glue code, layout, or delimiters needed by the surrounding target builtin lowering implementation.
- **CN**: 该代码块为周围的 目标内建函数降级 实现提供必要的胶水代码、布局或分隔结构。

### Lines 571-600
```cpp
571:   NEONMAP1(vcaddq_rot90_f16, arm_neon_vcadd_rot90, Add1ArgType),
572:   NEONMAP1(vcaddq_rot90_f32, arm_neon_vcadd_rot90, Add1ArgType),
573:   NEONMAP1(vcaddq_rot90_f64, arm_neon_vcadd_rot90, Add1ArgType),
574:   NEONMAP1(vcage_v, arm_neon_vacge, 0),
575:   NEONMAP1(vcageq_v, arm_neon_vacge, 0),
576:   NEONMAP1(vcagt_v, arm_neon_vacgt, 0),
577:   NEONMAP1(vcagtq_v, arm_neon_vacgt, 0),
578:   NEONMAP1(vcale_v, arm_neon_vacge, 0),
579:   NEONMAP1(vcaleq_v, arm_neon_vacge, 0),
580:   NEONMAP1(vcalt_v, arm_neon_vacgt, 0),
581:   NEONMAP1(vcaltq_v, arm_neon_vacgt, 0),
582:   NEONMAP0(vceqz_v),
583:   NEONMAP0(vceqzq_v),
584:   NEONMAP0(vcgez_v),
585:   NEONMAP0(vcgezq_v),
586:   NEONMAP0(vcgtz_v),
587:   NEONMAP0(vcgtzq_v),
588:   NEONMAP0(vclez_v),
589:   NEONMAP0(vclezq_v),
590:   NEONMAP1(vcls_v, arm_neon_vcls, Add1ArgType),
591:   NEONMAP1(vclsq_v, arm_neon_vcls, Add1ArgType),
592:   NEONMAP0(vcltz_v),
593:   NEONMAP0(vcltzq_v),
594:   NEONMAP1(vclz_v, ctlz, Add1ArgType),
595:   NEONMAP1(vclzq_v, ctlz, Add1ArgType),
596:   NEONMAP1(vcnt_v, ctpop, Add1ArgType),
597:   NEONMAP1(vcntq_v, ctpop, Add1ArgType),
598:   NEONMAP1(vcvt_f16_f32, arm_neon_vcvtfp2hf, 0),
599:   NEONMAP0(vcvt_f16_s16),
600:   NEONMAP0(vcvt_f16_u16),
```
- **EN**: This block provides glue code, layout, or delimiters needed by the surrounding target builtin lowering implementation.
- **CN**: 该代码块为周围的 目标内建函数降级 实现提供必要的胶水代码、布局或分隔结构。

### Lines 601-630
```cpp
601:   NEONMAP1(vcvt_f32_f16, arm_neon_vcvthf2fp, 0),
602:   NEONMAP0(vcvt_f32_v),
603:   NEONMAP1(vcvt_n_f16_s16, arm_neon_vcvtfxs2fp, 0),
604:   NEONMAP1(vcvt_n_f16_u16, arm_neon_vcvtfxu2fp, 0),
605:   NEONMAP2(vcvt_n_f32_v, arm_neon_vcvtfxu2fp, arm_neon_vcvtfxs2fp, 0),
606:   NEONMAP1(vcvt_n_s16_f16, arm_neon_vcvtfp2fxs, 0),
607:   NEONMAP1(vcvt_n_s32_v, arm_neon_vcvtfp2fxs, 0),
608:   NEONMAP1(vcvt_n_s64_v, arm_neon_vcvtfp2fxs, 0),
609:   NEONMAP1(vcvt_n_u16_f16, arm_neon_vcvtfp2fxu, 0),
610:   NEONMAP1(vcvt_n_u32_v, arm_neon_vcvtfp2fxu, 0),
611:   NEONMAP1(vcvt_n_u64_v, arm_neon_vcvtfp2fxu, 0),
612:   NEONMAP0(vcvt_s16_f16),
613:   NEONMAP0(vcvt_s32_v),
614:   NEONMAP0(vcvt_s64_v),
615:   NEONMAP0(vcvt_u16_f16),
616:   NEONMAP0(vcvt_u32_v),
617:   NEONMAP0(vcvt_u64_v),
618:   NEONMAP1(vcvta_s16_f16, arm_neon_vcvtas, 0),
619:   NEONMAP1(vcvta_s32_v, arm_neon_vcvtas, 0),
620:   NEONMAP1(vcvta_s64_v, arm_neon_vcvtas, 0),
621:   NEONMAP1(vcvta_u16_f16, arm_neon_vcvtau, 0),
622:   NEONMAP1(vcvta_u32_v, arm_neon_vcvtau, 0),
623:   NEONMAP1(vcvta_u64_v, arm_neon_vcvtau, 0),
624:   NEONMAP1(vcvtaq_s16_f16, arm_neon_vcvtas, 0),
625:   NEONMAP1(vcvtaq_s32_v, arm_neon_vcvtas, 0),
626:   NEONMAP1(vcvtaq_s64_v, arm_neon_vcvtas, 0),
627:   NEONMAP1(vcvtaq_u16_f16, arm_neon_vcvtau, 0),
628:   NEONMAP1(vcvtaq_u32_v, arm_neon_vcvtau, 0),
629:   NEONMAP1(vcvtaq_u64_v, arm_neon_vcvtau, 0),
630:   NEONMAP1(vcvth_bf16_f32, arm_neon_vcvtbfp2bf, 0),
```
- **EN**: This block provides glue code, layout, or delimiters needed by the surrounding target builtin lowering implementation.
- **CN**: 该代码块为周围的 目标内建函数降级 实现提供必要的胶水代码、布局或分隔结构。

### Lines 631-660
```cpp
631:   NEONMAP1(vcvtm_s16_f16, arm_neon_vcvtms, 0),
632:   NEONMAP1(vcvtm_s32_v, arm_neon_vcvtms, 0),
633:   NEONMAP1(vcvtm_s64_v, arm_neon_vcvtms, 0),
634:   NEONMAP1(vcvtm_u16_f16, arm_neon_vcvtmu, 0),
635:   NEONMAP1(vcvtm_u32_v, arm_neon_vcvtmu, 0),
636:   NEONMAP1(vcvtm_u64_v, arm_neon_vcvtmu, 0),
637:   NEONMAP1(vcvtmq_s16_f16, arm_neon_vcvtms, 0),
638:   NEONMAP1(vcvtmq_s32_v, arm_neon_vcvtms, 0),
639:   NEONMAP1(vcvtmq_s64_v, arm_neon_vcvtms, 0),
640:   NEONMAP1(vcvtmq_u16_f16, arm_neon_vcvtmu, 0),
641:   NEONMAP1(vcvtmq_u32_v, arm_neon_vcvtmu, 0),
642:   NEONMAP1(vcvtmq_u64_v, arm_neon_vcvtmu, 0),
643:   NEONMAP1(vcvtn_s16_f16, arm_neon_vcvtns, 0),
644:   NEONMAP1(vcvtn_s32_v, arm_neon_vcvtns, 0),
645:   NEONMAP1(vcvtn_s64_v, arm_neon_vcvtns, 0),
646:   NEONMAP1(vcvtn_u16_f16, arm_neon_vcvtnu, 0),
647:   NEONMAP1(vcvtn_u32_v, arm_neon_vcvtnu, 0),
648:   NEONMAP1(vcvtn_u64_v, arm_neon_vcvtnu, 0),
649:   NEONMAP1(vcvtnq_s16_f16, arm_neon_vcvtns, 0),
650:   NEONMAP1(vcvtnq_s32_v, arm_neon_vcvtns, 0),
651:   NEONMAP1(vcvtnq_s64_v, arm_neon_vcvtns, 0),
652:   NEONMAP1(vcvtnq_u16_f16, arm_neon_vcvtnu, 0),
653:   NEONMAP1(vcvtnq_u32_v, arm_neon_vcvtnu, 0),
654:   NEONMAP1(vcvtnq_u64_v, arm_neon_vcvtnu, 0),
655:   NEONMAP1(vcvtp_s16_f16, arm_neon_vcvtps, 0),
656:   NEONMAP1(vcvtp_s32_v, arm_neon_vcvtps, 0),
657:   NEONMAP1(vcvtp_s64_v, arm_neon_vcvtps, 0),
658:   NEONMAP1(vcvtp_u16_f16, arm_neon_vcvtpu, 0),
659:   NEONMAP1(vcvtp_u32_v, arm_neon_vcvtpu, 0),
660:   NEONMAP1(vcvtp_u64_v, arm_neon_vcvtpu, 0),
```
- **EN**: This block provides glue code, layout, or delimiters needed by the surrounding target builtin lowering implementation.
- **CN**: 该代码块为周围的 目标内建函数降级 实现提供必要的胶水代码、布局或分隔结构。

### Lines 661-690
```cpp
661:   NEONMAP1(vcvtpq_s16_f16, arm_neon_vcvtps, 0),
662:   NEONMAP1(vcvtpq_s32_v, arm_neon_vcvtps, 0),
663:   NEONMAP1(vcvtpq_s64_v, arm_neon_vcvtps, 0),
664:   NEONMAP1(vcvtpq_u16_f16, arm_neon_vcvtpu, 0),
665:   NEONMAP1(vcvtpq_u32_v, arm_neon_vcvtpu, 0),
666:   NEONMAP1(vcvtpq_u64_v, arm_neon_vcvtpu, 0),
667:   NEONMAP0(vcvtq_f16_s16),
668:   NEONMAP0(vcvtq_f16_u16),
669:   NEONMAP0(vcvtq_f32_v),
670:   NEONMAP1(vcvtq_n_f16_s16, arm_neon_vcvtfxs2fp, 0),
671:   NEONMAP1(vcvtq_n_f16_u16, arm_neon_vcvtfxu2fp, 0),
672:   NEONMAP2(vcvtq_n_f32_v, arm_neon_vcvtfxu2fp, arm_neon_vcvtfxs2fp, 0),
673:   NEONMAP1(vcvtq_n_s16_f16, arm_neon_vcvtfp2fxs, 0),
674:   NEONMAP1(vcvtq_n_s32_v, arm_neon_vcvtfp2fxs, 0),
675:   NEONMAP1(vcvtq_n_s64_v, arm_neon_vcvtfp2fxs, 0),
676:   NEONMAP1(vcvtq_n_u16_f16, arm_neon_vcvtfp2fxu, 0),
677:   NEONMAP1(vcvtq_n_u32_v, arm_neon_vcvtfp2fxu, 0),
678:   NEONMAP1(vcvtq_n_u64_v, arm_neon_vcvtfp2fxu, 0),
679:   NEONMAP0(vcvtq_s16_f16),
680:   NEONMAP0(vcvtq_s32_v),
681:   NEONMAP0(vcvtq_s64_v),
682:   NEONMAP0(vcvtq_u16_f16),
683:   NEONMAP0(vcvtq_u32_v),
684:   NEONMAP0(vcvtq_u64_v),
685:   NEONMAP1(vdot_s32, arm_neon_sdot, 0),
686:   NEONMAP1(vdot_u32, arm_neon_udot, 0),
687:   NEONMAP1(vdotq_s32, arm_neon_sdot, 0),
688:   NEONMAP1(vdotq_u32, arm_neon_udot, 0),
689:   NEONMAP0(vext_v),
690:   NEONMAP0(vextq_v),
```
- **EN**: This block provides glue code, layout, or delimiters needed by the surrounding target builtin lowering implementation.
- **CN**: 该代码块为周围的 目标内建函数降级 实现提供必要的胶水代码、布局或分隔结构。

### Lines 691-720
```cpp
691:   NEONMAP0(vfma_v),
692:   NEONMAP0(vfmaq_v),
693:   NEONMAP2(vhadd_v, arm_neon_vhaddu, arm_neon_vhadds, Add1ArgType | UnsignedAlts),
694:   NEONMAP2(vhaddq_v, arm_neon_vhaddu, arm_neon_vhadds, Add1ArgType | UnsignedAlts),
695:   NEONMAP2(vhsub_v, arm_neon_vhsubu, arm_neon_vhsubs, Add1ArgType | UnsignedAlts),
696:   NEONMAP2(vhsubq_v, arm_neon_vhsubu, arm_neon_vhsubs, Add1ArgType | UnsignedAlts),
697:   NEONMAP0(vld1_dup_v),
698:   NEONMAP1(vld1_v, arm_neon_vld1, 0),
699:   NEONMAP1(vld1_x2_v, arm_neon_vld1x2, 0),
700:   NEONMAP1(vld1_x3_v, arm_neon_vld1x3, 0),
701:   NEONMAP1(vld1_x4_v, arm_neon_vld1x4, 0),
702:   NEONMAP0(vld1q_dup_v),
703:   NEONMAP1(vld1q_v, arm_neon_vld1, 0),
704:   NEONMAP1(vld1q_x2_v, arm_neon_vld1x2, 0),
705:   NEONMAP1(vld1q_x3_v, arm_neon_vld1x3, 0),
706:   NEONMAP1(vld1q_x4_v, arm_neon_vld1x4, 0),
707:   NEONMAP1(vld2_dup_v, arm_neon_vld2dup, 0),
708:   NEONMAP1(vld2_lane_v, arm_neon_vld2lane, 0),
709:   NEONMAP1(vld2_v, arm_neon_vld2, 0),
710:   NEONMAP1(vld2q_dup_v, arm_neon_vld2dup, 0),
711:   NEONMAP1(vld2q_lane_v, arm_neon_vld2lane, 0),
712:   NEONMAP1(vld2q_v, arm_neon_vld2, 0),
713:   NEONMAP1(vld3_dup_v, arm_neon_vld3dup, 0),
714:   NEONMAP1(vld3_lane_v, arm_neon_vld3lane, 0),
715:   NEONMAP1(vld3_v, arm_neon_vld3, 0),
716:   NEONMAP1(vld3q_dup_v, arm_neon_vld3dup, 0),
717:   NEONMAP1(vld3q_lane_v, arm_neon_vld3lane, 0),
718:   NEONMAP1(vld3q_v, arm_neon_vld3, 0),
719:   NEONMAP1(vld4_dup_v, arm_neon_vld4dup, 0),
720:   NEONMAP1(vld4_lane_v, arm_neon_vld4lane, 0),
```
- **EN**: This block provides glue code, layout, or delimiters needed by the surrounding target builtin lowering implementation.
- **CN**: 该代码块为周围的 目标内建函数降级 实现提供必要的胶水代码、布局或分隔结构。

### Lines 721-750
```cpp
721:   NEONMAP1(vld4_v, arm_neon_vld4, 0),
722:   NEONMAP1(vld4q_dup_v, arm_neon_vld4dup, 0),
723:   NEONMAP1(vld4q_lane_v, arm_neon_vld4lane, 0),
724:   NEONMAP1(vld4q_v, arm_neon_vld4, 0),
725:   NEONMAP2(vmax_v, arm_neon_vmaxu, arm_neon_vmaxs, Add1ArgType | UnsignedAlts),
726:   NEONMAP1(vmaxnm_v, arm_neon_vmaxnm, Add1ArgType),
727:   NEONMAP1(vmaxnmq_v, arm_neon_vmaxnm, Add1ArgType),
728:   NEONMAP2(vmaxq_v, arm_neon_vmaxu, arm_neon_vmaxs, Add1ArgType | UnsignedAlts),
729:   NEONMAP2(vmin_v, arm_neon_vminu, arm_neon_vmins, Add1ArgType | UnsignedAlts),
730:   NEONMAP1(vminnm_v, arm_neon_vminnm, Add1ArgType),
731:   NEONMAP1(vminnmq_v, arm_neon_vminnm, Add1ArgType),
732:   NEONMAP2(vminq_v, arm_neon_vminu, arm_neon_vmins, Add1ArgType | UnsignedAlts),
733:   NEONMAP1(vmmlaq_s32, arm_neon_smmla, 0),
734:   NEONMAP1(vmmlaq_u32, arm_neon_ummla, 0),
735:   NEONMAP0(vmovl_v),
736:   NEONMAP0(vmovn_v),
737:   NEONMAP1(vmul_v, arm_neon_vmulp, Add1ArgType),
738:   NEONMAP0(vmull_v),
739:   NEONMAP1(vmulq_v, arm_neon_vmulp, Add1ArgType),
740:   NEONMAP2(vpadal_v, arm_neon_vpadalu, arm_neon_vpadals, UnsignedAlts),
741:   NEONMAP2(vpadalq_v, arm_neon_vpadalu, arm_neon_vpadals, UnsignedAlts),
742:   NEONMAP1(vpadd_v, arm_neon_vpadd, Add1ArgType),
743:   NEONMAP2(vpaddl_v, arm_neon_vpaddlu, arm_neon_vpaddls, UnsignedAlts),
744:   NEONMAP2(vpaddlq_v, arm_neon_vpaddlu, arm_neon_vpaddls, UnsignedAlts),
745:   NEONMAP1(vpaddq_v, arm_neon_vpadd, Add1ArgType),
746:   NEONMAP2(vpmax_v, arm_neon_vpmaxu, arm_neon_vpmaxs, Add1ArgType | UnsignedAlts),
747:   NEONMAP2(vpmin_v, arm_neon_vpminu, arm_neon_vpmins, Add1ArgType | UnsignedAlts),
748:   NEONMAP1(vqabs_v, arm_neon_vqabs, Add1ArgType),
749:   NEONMAP1(vqabsq_v, arm_neon_vqabs, Add1ArgType),
750:   NEONMAP2(vqadd_v, uadd_sat, sadd_sat, Add1ArgType | UnsignedAlts),
```
- **EN**: This block provides glue code, layout, or delimiters needed by the surrounding target builtin lowering implementation.
- **CN**: 该代码块为周围的 目标内建函数降级 实现提供必要的胶水代码、布局或分隔结构。

### Lines 751-780
```cpp
751:   NEONMAP2(vqaddq_v, uadd_sat, sadd_sat, Add1ArgType | UnsignedAlts),
752:   NEONMAP2(vqdmlal_v, arm_neon_vqdmull, sadd_sat, 0),
753:   NEONMAP2(vqdmlsl_v, arm_neon_vqdmull, ssub_sat, 0),
754:   NEONMAP1(vqdmulh_v, arm_neon_vqdmulh, Add1ArgType),
755:   NEONMAP1(vqdmulhq_v, arm_neon_vqdmulh, Add1ArgType),
756:   NEONMAP1(vqdmull_v, arm_neon_vqdmull, Add1ArgType),
757:   NEONMAP2(vqmovn_v, arm_neon_vqmovnu, arm_neon_vqmovns, Add1ArgType | UnsignedAlts),
758:   NEONMAP1(vqmovun_v, arm_neon_vqmovnsu, Add1ArgType),
759:   NEONMAP1(vqneg_v, arm_neon_vqneg, Add1ArgType),
760:   NEONMAP1(vqnegq_v, arm_neon_vqneg, Add1ArgType),
761:   NEONMAP1(vqrdmlah_s16, arm_neon_vqrdmlah, Add1ArgType),
762:   NEONMAP1(vqrdmlah_s32, arm_neon_vqrdmlah, Add1ArgType),
763:   NEONMAP1(vqrdmlahq_s16, arm_neon_vqrdmlah, Add1ArgType),
764:   NEONMAP1(vqrdmlahq_s32, arm_neon_vqrdmlah, Add1ArgType),
765:   NEONMAP1(vqrdmlsh_s16, arm_neon_vqrdmlsh, Add1ArgType),
766:   NEONMAP1(vqrdmlsh_s32, arm_neon_vqrdmlsh, Add1ArgType),
767:   NEONMAP1(vqrdmlshq_s16, arm_neon_vqrdmlsh, Add1ArgType),
768:   NEONMAP1(vqrdmlshq_s32, arm_neon_vqrdmlsh, Add1ArgType),
769:   NEONMAP1(vqrdmulh_v, arm_neon_vqrdmulh, Add1ArgType),
770:   NEONMAP1(vqrdmulhq_v, arm_neon_vqrdmulh, Add1ArgType),
771:   NEONMAP2(vqrshl_v, arm_neon_vqrshiftu, arm_neon_vqrshifts, Add1ArgType | UnsignedAlts),
772:   NEONMAP2(vqrshlq_v, arm_neon_vqrshiftu, arm_neon_vqrshifts, Add1ArgType | UnsignedAlts),
773:   NEONMAP2(vqshl_n_v, arm_neon_vqshiftu, arm_neon_vqshifts, UnsignedAlts),
774:   NEONMAP2(vqshl_v, arm_neon_vqshiftu, arm_neon_vqshifts, Add1ArgType | UnsignedAlts),
775:   NEONMAP2(vqshlq_n_v, arm_neon_vqshiftu, arm_neon_vqshifts, UnsignedAlts),
776:   NEONMAP2(vqshlq_v, arm_neon_vqshiftu, arm_neon_vqshifts, Add1ArgType | UnsignedAlts),
777:   NEONMAP1(vqshlu_n_v, arm_neon_vqshiftsu, 0),
778:   NEONMAP1(vqshluq_n_v, arm_neon_vqshiftsu, 0),
779:   NEONMAP2(vqsub_v, usub_sat, ssub_sat, Add1ArgType | UnsignedAlts),
780:   NEONMAP2(vqsubq_v, usub_sat, ssub_sat, Add1ArgType | UnsignedAlts),
```
- **EN**: This block provides glue code, layout, or delimiters needed by the surrounding target builtin lowering implementation.
- **CN**: 该代码块为周围的 目标内建函数降级 实现提供必要的胶水代码、布局或分隔结构。

### Lines 781-810
```cpp
781:   NEONMAP1(vraddhn_v, arm_neon_vraddhn, Add1ArgType),
782:   NEONMAP2(vrecpe_v, arm_neon_vrecpe, arm_neon_vrecpe, 0),
783:   NEONMAP2(vrecpeq_v, arm_neon_vrecpe, arm_neon_vrecpe, 0),
784:   NEONMAP1(vrecps_v, arm_neon_vrecps, Add1ArgType),
785:   NEONMAP1(vrecpsq_v, arm_neon_vrecps, Add1ArgType),
786:   NEONMAP2(vrhadd_v, arm_neon_vrhaddu, arm_neon_vrhadds, Add1ArgType | UnsignedAlts),
787:   NEONMAP2(vrhaddq_v, arm_neon_vrhaddu, arm_neon_vrhadds, Add1ArgType | UnsignedAlts),
788:   NEONMAP1(vrnd_v, trunc, Add1ArgType),
789:   NEONMAP1(vrnda_v, round, Add1ArgType),
790:   NEONMAP1(vrndaq_v, round, Add1ArgType),
791:   NEONMAP0(vrndi_v),
792:   NEONMAP0(vrndiq_v),
793:   NEONMAP1(vrndm_v, floor, Add1ArgType),
794:   NEONMAP1(vrndmq_v, floor, Add1ArgType),
795:   NEONMAP1(vrndn_v, roundeven, Add1ArgType),
796:   NEONMAP1(vrndnq_v, roundeven, Add1ArgType),
797:   NEONMAP1(vrndp_v, ceil, Add1ArgType),
798:   NEONMAP1(vrndpq_v, ceil, Add1ArgType),
799:   NEONMAP1(vrndq_v, trunc, Add1ArgType),
800:   NEONMAP1(vrndx_v, rint, Add1ArgType),
801:   NEONMAP1(vrndxq_v, rint, Add1ArgType),
802:   NEONMAP2(vrshl_v, arm_neon_vrshiftu, arm_neon_vrshifts, Add1ArgType | UnsignedAlts),
803:   NEONMAP2(vrshlq_v, arm_neon_vrshiftu, arm_neon_vrshifts, Add1ArgType | UnsignedAlts),
804:   NEONMAP2(vrshr_n_v, arm_neon_vrshiftu, arm_neon_vrshifts, UnsignedAlts),
805:   NEONMAP2(vrshrq_n_v, arm_neon_vrshiftu, arm_neon_vrshifts, UnsignedAlts),
806:   NEONMAP2(vrsqrte_v, arm_neon_vrsqrte, arm_neon_vrsqrte, 0),
807:   NEONMAP2(vrsqrteq_v, arm_neon_vrsqrte, arm_neon_vrsqrte, 0),
808:   NEONMAP1(vrsqrts_v, arm_neon_vrsqrts, Add1ArgType),
809:   NEONMAP1(vrsqrtsq_v, arm_neon_vrsqrts, Add1ArgType),
810:   NEONMAP1(vrsubhn_v, arm_neon_vrsubhn, Add1ArgType),
```
- **EN**: This block provides glue code, layout, or delimiters needed by the surrounding target builtin lowering implementation.
- **CN**: 该代码块为周围的 目标内建函数降级 实现提供必要的胶水代码、布局或分隔结构。

### Lines 811-840
```cpp
811:   NEONMAP1(vsha1su0q_u32, arm_neon_sha1su0, 0),
812:   NEONMAP1(vsha1su1q_u32, arm_neon_sha1su1, 0),
813:   NEONMAP1(vsha256h2q_u32, arm_neon_sha256h2, 0),
814:   NEONMAP1(vsha256hq_u32, arm_neon_sha256h, 0),
815:   NEONMAP1(vsha256su0q_u32, arm_neon_sha256su0, 0),
816:   NEONMAP1(vsha256su1q_u32, arm_neon_sha256su1, 0),
817:   NEONMAP0(vshl_n_v),
818:   NEONMAP2(vshl_v, arm_neon_vshiftu, arm_neon_vshifts, Add1ArgType | UnsignedAlts),
819:   NEONMAP0(vshll_n_v),
820:   NEONMAP0(vshlq_n_v),
821:   NEONMAP2(vshlq_v, arm_neon_vshiftu, arm_neon_vshifts, Add1ArgType | UnsignedAlts),
822:   NEONMAP0(vshr_n_v),
823:   NEONMAP0(vshrn_n_v),
824:   NEONMAP0(vshrq_n_v),
825:   NEONMAP1(vst1_v, arm_neon_vst1, 0),
826:   NEONMAP1(vst1_x2_v, arm_neon_vst1x2, 0),
827:   NEONMAP1(vst1_x3_v, arm_neon_vst1x3, 0),
828:   NEONMAP1(vst1_x4_v, arm_neon_vst1x4, 0),
829:   NEONMAP1(vst1q_v, arm_neon_vst1, 0),
830:   NEONMAP1(vst1q_x2_v, arm_neon_vst1x2, 0),
831:   NEONMAP1(vst1q_x3_v, arm_neon_vst1x3, 0),
832:   NEONMAP1(vst1q_x4_v, arm_neon_vst1x4, 0),
833:   NEONMAP1(vst2_lane_v, arm_neon_vst2lane, 0),
834:   NEONMAP1(vst2_v, arm_neon_vst2, 0),
835:   NEONMAP1(vst2q_lane_v, arm_neon_vst2lane, 0),
836:   NEONMAP1(vst2q_v, arm_neon_vst2, 0),
837:   NEONMAP1(vst3_lane_v, arm_neon_vst3lane, 0),
838:   NEONMAP1(vst3_v, arm_neon_vst3, 0),
839:   NEONMAP1(vst3q_lane_v, arm_neon_vst3lane, 0),
840:   NEONMAP1(vst3q_v, arm_neon_vst3, 0),
```
- **EN**: This block provides glue code, layout, or delimiters needed by the surrounding target builtin lowering implementation.
- **CN**: 该代码块为周围的 目标内建函数降级 实现提供必要的胶水代码、布局或分隔结构。

### Lines 841-870
```cpp
841:   NEONMAP1(vst4_lane_v, arm_neon_vst4lane, 0),
842:   NEONMAP1(vst4_v, arm_neon_vst4, 0),
843:   NEONMAP1(vst4q_lane_v, arm_neon_vst4lane, 0),
844:   NEONMAP1(vst4q_v, arm_neon_vst4, 0),
845:   NEONMAP0(vsubhn_v),
846:   NEONMAP0(vtrn_v),
847:   NEONMAP0(vtrnq_v),
848:   NEONMAP0(vtst_v),
849:   NEONMAP0(vtstq_v),
850:   NEONMAP1(vusdot_s32, arm_neon_usdot, 0),
851:   NEONMAP1(vusdotq_s32, arm_neon_usdot, 0),
852:   NEONMAP1(vusmmlaq_s32, arm_neon_usmmla, 0),
853:   NEONMAP0(vuzp_v),
854:   NEONMAP0(vuzpq_v),
855:   NEONMAP0(vzip_v),
856:   NEONMAP0(vzipq_v)
857: };
858: 
859: // clang-format on
860: 
861: // Some intrinsics are equivalent for codegen.
862: static const std::pair<unsigned, unsigned> NEONEquivalentIntrinsicMap[] = {
863:   { NEON::BI__builtin_neon_splat_lane_bf16, NEON::BI__builtin_neon_splat_lane_v, },
864:   { NEON::BI__builtin_neon_splat_laneq_bf16, NEON::BI__builtin_neon_splat_laneq_v, },
865:   { NEON::BI__builtin_neon_splatq_lane_bf16, NEON::BI__builtin_neon_splatq_lane_v, },
866:   { NEON::BI__builtin_neon_splatq_laneq_bf16, NEON::BI__builtin_neon_splatq_laneq_v, },
867:   { NEON::BI__builtin_neon_vabd_f16, NEON::BI__builtin_neon_vabd_v, },
868:   { NEON::BI__builtin_neon_vabdq_f16, NEON::BI__builtin_neon_vabdq_v, },
869:   { NEON::BI__builtin_neon_vabs_f16, NEON::BI__builtin_neon_vabs_v, },
870:   { NEON::BI__builtin_neon_vabsq_f16, NEON::BI__builtin_neon_vabsq_v, },
```
- **EN**: This block documents intent or context for the surrounding target builtin lowering code.
- **CN**: 该代码块说明周围 目标内建函数降级 代码的意图或上下文。

### Lines 871-900
```cpp
871:   { NEON::BI__builtin_neon_vcage_f16, NEON::BI__builtin_neon_vcage_v, },
872:   { NEON::BI__builtin_neon_vcageq_f16, NEON::BI__builtin_neon_vcageq_v, },
873:   { NEON::BI__builtin_neon_vcagt_f16, NEON::BI__builtin_neon_vcagt_v, },
874:   { NEON::BI__builtin_neon_vcagtq_f16, NEON::BI__builtin_neon_vcagtq_v, },
875:   { NEON::BI__builtin_neon_vcale_f16, NEON::BI__builtin_neon_vcale_v, },
876:   { NEON::BI__builtin_neon_vcaleq_f16, NEON::BI__builtin_neon_vcaleq_v, },
877:   { NEON::BI__builtin_neon_vcalt_f16, NEON::BI__builtin_neon_vcalt_v, },
878:   { NEON::BI__builtin_neon_vcaltq_f16, NEON::BI__builtin_neon_vcaltq_v, },
879:   { NEON::BI__builtin_neon_vceqz_f16, NEON::BI__builtin_neon_vceqz_v, },
880:   { NEON::BI__builtin_neon_vceqzq_f16, NEON::BI__builtin_neon_vceqzq_v, },
881:   { NEON::BI__builtin_neon_vcgez_f16, NEON::BI__builtin_neon_vcgez_v, },
882:   { NEON::BI__builtin_neon_vcgezq_f16, NEON::BI__builtin_neon_vcgezq_v, },
883:   { NEON::BI__builtin_neon_vcgtz_f16, NEON::BI__builtin_neon_vcgtz_v, },
884:   { NEON::BI__builtin_neon_vcgtzq_f16, NEON::BI__builtin_neon_vcgtzq_v, },
885:   { NEON::BI__builtin_neon_vclez_f16, NEON::BI__builtin_neon_vclez_v, },
886:   { NEON::BI__builtin_neon_vclezq_f16, NEON::BI__builtin_neon_vclezq_v, },
887:   { NEON::BI__builtin_neon_vcltz_f16, NEON::BI__builtin_neon_vcltz_v, },
888:   { NEON::BI__builtin_neon_vcltzq_f16, NEON::BI__builtin_neon_vcltzq_v, },
889:   { NEON::BI__builtin_neon_vfma_f16, NEON::BI__builtin_neon_vfma_v, },
890:   { NEON::BI__builtin_neon_vfma_lane_f16, NEON::BI__builtin_neon_vfma_lane_v, },
891:   { NEON::BI__builtin_neon_vfma_laneq_f16, NEON::BI__builtin_neon_vfma_laneq_v, },
892:   { NEON::BI__builtin_neon_vfmaq_f16, NEON::BI__builtin_neon_vfmaq_v, },
893:   { NEON::BI__builtin_neon_vfmaq_lane_f16, NEON::BI__builtin_neon_vfmaq_lane_v, },
894:   { NEON::BI__builtin_neon_vfmaq_laneq_f16, NEON::BI__builtin_neon_vfmaq_laneq_v, },
895:   { NEON::BI__builtin_neon_vld1_bf16_x2, NEON::BI__builtin_neon_vld1_x2_v },
896:   { NEON::BI__builtin_neon_vld1_bf16_x3, NEON::BI__builtin_neon_vld1_x3_v },
897:   { NEON::BI__builtin_neon_vld1_bf16_x4, NEON::BI__builtin_neon_vld1_x4_v },
898:   { NEON::BI__builtin_neon_vld1_bf16, NEON::BI__builtin_neon_vld1_v },
899:   { NEON::BI__builtin_neon_vld1_dup_bf16, NEON::BI__builtin_neon_vld1_dup_v },
900:   { NEON::BI__builtin_neon_vld1_lane_bf16, NEON::BI__builtin_neon_vld1_lane_v },
```
- **EN**: This block provides glue code, layout, or delimiters needed by the surrounding target builtin lowering implementation.
- **CN**: 该代码块为周围的 目标内建函数降级 实现提供必要的胶水代码、布局或分隔结构。

### Lines 901-930
```cpp
901:   { NEON::BI__builtin_neon_vld1q_bf16_x2, NEON::BI__builtin_neon_vld1q_x2_v },
902:   { NEON::BI__builtin_neon_vld1q_bf16_x3, NEON::BI__builtin_neon_vld1q_x3_v },
903:   { NEON::BI__builtin_neon_vld1q_bf16_x4, NEON::BI__builtin_neon_vld1q_x4_v },
904:   { NEON::BI__builtin_neon_vld1q_bf16, NEON::BI__builtin_neon_vld1q_v },
905:   { NEON::BI__builtin_neon_vld1q_dup_bf16, NEON::BI__builtin_neon_vld1q_dup_v },
906:   { NEON::BI__builtin_neon_vld1q_lane_bf16, NEON::BI__builtin_neon_vld1q_lane_v },
907:   { NEON::BI__builtin_neon_vld2_bf16, NEON::BI__builtin_neon_vld2_v },
908:   { NEON::BI__builtin_neon_vld2_dup_bf16, NEON::BI__builtin_neon_vld2_dup_v },
909:   { NEON::BI__builtin_neon_vld2_lane_bf16, NEON::BI__builtin_neon_vld2_lane_v },
910:   { NEON::BI__builtin_neon_vld2q_bf16, NEON::BI__builtin_neon_vld2q_v },
911:   { NEON::BI__builtin_neon_vld2q_dup_bf16, NEON::BI__builtin_neon_vld2q_dup_v },
912:   { NEON::BI__builtin_neon_vld2q_lane_bf16, NEON::BI__builtin_neon_vld2q_lane_v },
913:   { NEON::BI__builtin_neon_vld3_bf16, NEON::BI__builtin_neon_vld3_v },
914:   { NEON::BI__builtin_neon_vld3_dup_bf16, NEON::BI__builtin_neon_vld3_dup_v },
915:   { NEON::BI__builtin_neon_vld3_lane_bf16, NEON::BI__builtin_neon_vld3_lane_v },
916:   { NEON::BI__builtin_neon_vld3q_bf16, NEON::BI__builtin_neon_vld3q_v },
917:   { NEON::BI__builtin_neon_vld3q_dup_bf16, NEON::BI__builtin_neon_vld3q_dup_v },
918:   { NEON::BI__builtin_neon_vld3q_lane_bf16, NEON::BI__builtin_neon_vld3q_lane_v },
919:   { NEON::BI__builtin_neon_vld4_bf16, NEON::BI__builtin_neon_vld4_v },
920:   { NEON::BI__builtin_neon_vld4_dup_bf16, NEON::BI__builtin_neon_vld4_dup_v },
921:   { NEON::BI__builtin_neon_vld4_lane_bf16, NEON::BI__builtin_neon_vld4_lane_v },
922:   { NEON::BI__builtin_neon_vld4q_bf16, NEON::BI__builtin_neon_vld4q_v },
923:   { NEON::BI__builtin_neon_vld4q_dup_bf16, NEON::BI__builtin_neon_vld4q_dup_v },
924:   { NEON::BI__builtin_neon_vld4q_lane_bf16, NEON::BI__builtin_neon_vld4q_lane_v },
925:   { NEON::BI__builtin_neon_vmax_f16, NEON::BI__builtin_neon_vmax_v, },
926:   { NEON::BI__builtin_neon_vmaxnm_f16, NEON::BI__builtin_neon_vmaxnm_v, },
927:   { NEON::BI__builtin_neon_vmaxnmq_f16, NEON::BI__builtin_neon_vmaxnmq_v, },
928:   { NEON::BI__builtin_neon_vmaxq_f16, NEON::BI__builtin_neon_vmaxq_v, },
929:   { NEON::BI__builtin_neon_vmin_f16, NEON::BI__builtin_neon_vmin_v, },
930:   { NEON::BI__builtin_neon_vminnm_f16, NEON::BI__builtin_neon_vminnm_v, },
```
- **EN**: This block provides glue code, layout, or delimiters needed by the surrounding target builtin lowering implementation.
- **CN**: 该代码块为周围的 目标内建函数降级 实现提供必要的胶水代码、布局或分隔结构。

### Lines 931-960
```cpp
931:   { NEON::BI__builtin_neon_vminnmq_f16, NEON::BI__builtin_neon_vminnmq_v, },
932:   { NEON::BI__builtin_neon_vminq_f16, NEON::BI__builtin_neon_vminq_v, },
933:   { NEON::BI__builtin_neon_vmulx_f16, NEON::BI__builtin_neon_vmulx_v, },
934:   { NEON::BI__builtin_neon_vmulxq_f16, NEON::BI__builtin_neon_vmulxq_v, },
935:   { NEON::BI__builtin_neon_vpadd_f16, NEON::BI__builtin_neon_vpadd_v, },
936:   { NEON::BI__builtin_neon_vpaddq_f16, NEON::BI__builtin_neon_vpaddq_v, },
937:   { NEON::BI__builtin_neon_vpmax_f16, NEON::BI__builtin_neon_vpmax_v, },
938:   { NEON::BI__builtin_neon_vpmaxnm_f16, NEON::BI__builtin_neon_vpmaxnm_v, },
939:   { NEON::BI__builtin_neon_vpmaxnmq_f16, NEON::BI__builtin_neon_vpmaxnmq_v, },
940:   { NEON::BI__builtin_neon_vpmaxq_f16, NEON::BI__builtin_neon_vpmaxq_v, },
941:   { NEON::BI__builtin_neon_vpmin_f16, NEON::BI__builtin_neon_vpmin_v, },
942:   { NEON::BI__builtin_neon_vpminnm_f16, NEON::BI__builtin_neon_vpminnm_v, },
943:   { NEON::BI__builtin_neon_vpminnmq_f16, NEON::BI__builtin_neon_vpminnmq_v, },
944:   { NEON::BI__builtin_neon_vpminq_f16, NEON::BI__builtin_neon_vpminq_v, },
945:   { NEON::BI__builtin_neon_vrecpe_f16, NEON::BI__builtin_neon_vrecpe_v, },
946:   { NEON::BI__builtin_neon_vrecpeq_f16, NEON::BI__builtin_neon_vrecpeq_v, },
947:   { NEON::BI__builtin_neon_vrecps_f16, NEON::BI__builtin_neon_vrecps_v, },
948:   { NEON::BI__builtin_neon_vrecpsq_f16, NEON::BI__builtin_neon_vrecpsq_v, },
949:   { NEON::BI__builtin_neon_vrnd_f16, NEON::BI__builtin_neon_vrnd_v, },
950:   { NEON::BI__builtin_neon_vrnda_f16, NEON::BI__builtin_neon_vrnda_v, },
951:   { NEON::BI__builtin_neon_vrndaq_f16, NEON::BI__builtin_neon_vrndaq_v, },
952:   { NEON::BI__builtin_neon_vrndi_f16, NEON::BI__builtin_neon_vrndi_v, },
953:   { NEON::BI__builtin_neon_vrndiq_f16, NEON::BI__builtin_neon_vrndiq_v, },
954:   { NEON::BI__builtin_neon_vrndm_f16, NEON::BI__builtin_neon_vrndm_v, },
955:   { NEON::BI__builtin_neon_vrndmq_f16, NEON::BI__builtin_neon_vrndmq_v, },
956:   { NEON::BI__builtin_neon_vrndn_f16, NEON::BI__builtin_neon_vrndn_v, },
957:   { NEON::BI__builtin_neon_vrndnq_f16, NEON::BI__builtin_neon_vrndnq_v, },
958:   { NEON::BI__builtin_neon_vrndp_f16, NEON::BI__builtin_neon_vrndp_v, },
959:   { NEON::BI__builtin_neon_vrndpq_f16, NEON::BI__builtin_neon_vrndpq_v, },
960:   { NEON::BI__builtin_neon_vrndq_f16, NEON::BI__builtin_neon_vrndq_v, },
```
- **EN**: This block provides glue code, layout, or delimiters needed by the surrounding target builtin lowering implementation.
- **CN**: 该代码块为周围的 目标内建函数降级 实现提供必要的胶水代码、布局或分隔结构。

### Lines 961-990
```cpp
961:   { NEON::BI__builtin_neon_vrndx_f16, NEON::BI__builtin_neon_vrndx_v, },
962:   { NEON::BI__builtin_neon_vrndxq_f16, NEON::BI__builtin_neon_vrndxq_v, },
963:   { NEON::BI__builtin_neon_vrsqrte_f16, NEON::BI__builtin_neon_vrsqrte_v, },
964:   { NEON::BI__builtin_neon_vrsqrteq_f16, NEON::BI__builtin_neon_vrsqrteq_v, },
965:   { NEON::BI__builtin_neon_vrsqrts_f16, NEON::BI__builtin_neon_vrsqrts_v, },
966:   { NEON::BI__builtin_neon_vrsqrtsq_f16, NEON::BI__builtin_neon_vrsqrtsq_v, },
967:   { NEON::BI__builtin_neon_vsqrt_f16, NEON::BI__builtin_neon_vsqrt_v, },
968:   { NEON::BI__builtin_neon_vsqrtq_f16, NEON::BI__builtin_neon_vsqrtq_v, },
969:   { NEON::BI__builtin_neon_vst1_bf16_x2, NEON::BI__builtin_neon_vst1_x2_v },
970:   { NEON::BI__builtin_neon_vst1_bf16_x3, NEON::BI__builtin_neon_vst1_x3_v },
971:   { NEON::BI__builtin_neon_vst1_bf16_x4, NEON::BI__builtin_neon_vst1_x4_v },
972:   { NEON::BI__builtin_neon_vst1_bf16, NEON::BI__builtin_neon_vst1_v },
973:   { NEON::BI__builtin_neon_vst1_lane_bf16, NEON::BI__builtin_neon_vst1_lane_v },
974:   { NEON::BI__builtin_neon_vst1q_bf16_x2, NEON::BI__builtin_neon_vst1q_x2_v },
975:   { NEON::BI__builtin_neon_vst1q_bf16_x3, NEON::BI__builtin_neon_vst1q_x3_v },
976:   { NEON::BI__builtin_neon_vst1q_bf16_x4, NEON::BI__builtin_neon_vst1q_x4_v },
977:   { NEON::BI__builtin_neon_vst1q_bf16, NEON::BI__builtin_neon_vst1q_v },
978:   { NEON::BI__builtin_neon_vst1q_lane_bf16, NEON::BI__builtin_neon_vst1q_lane_v },
979:   { NEON::BI__builtin_neon_vst2_bf16, NEON::BI__builtin_neon_vst2_v },
980:   { NEON::BI__builtin_neon_vst2_lane_bf16, NEON::BI__builtin_neon_vst2_lane_v },
981:   { NEON::BI__builtin_neon_vst2q_bf16, NEON::BI__builtin_neon_vst2q_v },
982:   { NEON::BI__builtin_neon_vst2q_lane_bf16, NEON::BI__builtin_neon_vst2q_lane_v },
983:   { NEON::BI__builtin_neon_vst3_bf16, NEON::BI__builtin_neon_vst3_v },
984:   { NEON::BI__builtin_neon_vst3_lane_bf16, NEON::BI__builtin_neon_vst3_lane_v },
985:   { NEON::BI__builtin_neon_vst3q_bf16, NEON::BI__builtin_neon_vst3q_v },
986:   { NEON::BI__builtin_neon_vst3q_lane_bf16, NEON::BI__builtin_neon_vst3q_lane_v },
987:   { NEON::BI__builtin_neon_vst4_bf16, NEON::BI__builtin_neon_vst4_v },
988:   { NEON::BI__builtin_neon_vst4_lane_bf16, NEON::BI__builtin_neon_vst4_lane_v },
989:   { NEON::BI__builtin_neon_vst4q_bf16, NEON::BI__builtin_neon_vst4q_v },
990:   { NEON::BI__builtin_neon_vst4q_lane_bf16, NEON::BI__builtin_neon_vst4q_lane_v },
```
- **EN**: This block provides glue code, layout, or delimiters needed by the surrounding target builtin lowering implementation.
- **CN**: 该代码块为周围的 目标内建函数降级 实现提供必要的胶水代码、布局或分隔结构。

### Lines 991-1020
```cpp
 991:   // The mangling rules cause us to have one ID for each type for vldap1(q)_lane
 992:   // and vstl1(q)_lane, but codegen is equivalent for all of them. Choose an
 993:   // arbitrary one to be handled as tha canonical variation.
 994:   { NEON::BI__builtin_neon_vldap1_lane_u64, NEON::BI__builtin_neon_vldap1_lane_s64 },
 995:   { NEON::BI__builtin_neon_vldap1_lane_f64, NEON::BI__builtin_neon_vldap1_lane_s64 },
 996:   { NEON::BI__builtin_neon_vldap1_lane_p64, NEON::BI__builtin_neon_vldap1_lane_s64 },
 997:   { NEON::BI__builtin_neon_vldap1q_lane_u64, NEON::BI__builtin_neon_vldap1q_lane_s64 },
 998:   { NEON::BI__builtin_neon_vldap1q_lane_f64, NEON::BI__builtin_neon_vldap1q_lane_s64 },
 999:   { NEON::BI__builtin_neon_vldap1q_lane_p64, NEON::BI__builtin_neon_vldap1q_lane_s64 },
1000:   { NEON::BI__builtin_neon_vstl1_lane_u64, NEON::BI__builtin_neon_vstl1_lane_s64 },
1001:   { NEON::BI__builtin_neon_vstl1_lane_f64, NEON::BI__builtin_neon_vstl1_lane_s64 },
1002:   { NEON::BI__builtin_neon_vstl1_lane_p64, NEON::BI__builtin_neon_vstl1_lane_s64 },
1003:   { NEON::BI__builtin_neon_vstl1q_lane_u64, NEON::BI__builtin_neon_vstl1q_lane_s64 },
1004:   { NEON::BI__builtin_neon_vstl1q_lane_f64, NEON::BI__builtin_neon_vstl1q_lane_s64 },
1005:   { NEON::BI__builtin_neon_vstl1q_lane_p64, NEON::BI__builtin_neon_vstl1q_lane_s64 },
1006: };
1007: 
1008: #undef NEONMAP0
1009: #undef NEONMAP1
1010: #undef NEONMAP2
1011: 
1012: #define SVEMAP1(NameBase, LLVMIntrinsic, TypeModifier)                         \
1013:   {                                                                            \
1014:     #NameBase, SVE::BI__builtin_sve_##NameBase, Intrinsic::LLVMIntrinsic, 0,   \
1015:         TypeModifier                                                           \
1016:   }
1017: 
1018: #define SVEMAP2(NameBase, TypeModifier)                                        \
1019:   { #NameBase, SVE::BI__builtin_sve_##NameBase, 0, 0, TypeModifier }
1020: static const ARMVectorIntrinsicInfo AArch64SVEIntrinsicMap[] = {
```
- **EN**: This block contains preprocessor structure that shapes compilation of this unit.
- **CN**: 该代码块包含影响本编译单元构建方式的预处理结构。

### Lines 1021-1050
```cpp
1021: #define GET_SVE_LLVM_INTRINSIC_MAP
1022: #include "clang/Basic/arm_sve_builtin_cg.inc"
1023: #include "clang/Basic/BuiltinsAArch64NeonSVEBridge_cg.def"
1024: #undef GET_SVE_LLVM_INTRINSIC_MAP
1025: };
1026: 
1027: #undef SVEMAP1
1028: #undef SVEMAP2
1029: 
1030: #define SMEMAP1(NameBase, LLVMIntrinsic, TypeModifier)                         \
1031:   {                                                                            \
1032:     #NameBase, SME::BI__builtin_sme_##NameBase, Intrinsic::LLVMIntrinsic, 0,   \
1033:         TypeModifier                                                           \
1034:   }
1035: 
1036: #define SMEMAP2(NameBase, TypeModifier)                                        \
1037:   { #NameBase, SME::BI__builtin_sme_##NameBase, 0, 0, TypeModifier }
1038: static const ARMVectorIntrinsicInfo AArch64SMEIntrinsicMap[] = {
1039: #define GET_SME_LLVM_INTRINSIC_MAP
1040: #include "clang/Basic/arm_sme_builtin_cg.inc"
1041: #undef GET_SME_LLVM_INTRINSIC_MAP
1042: };
1043: 
1044: #undef SMEMAP1
1045: #undef SMEMAP2
1046: 
1047: static bool NEONSIMDIntrinsicsProvenSorted = false;
1048: 
1049: static bool AArch64SIMDIntrinsicsProvenSorted = false;
1050: static bool AArch64SISDIntrinsicsProvenSorted = false;
```
- **EN**: This block imports Clang headers `clang/Basic/arm_sve_builtin_cg.inc`, `clang/Basic/BuiltinsAArch64NeonSVEBridge_cg.def`, `clang/Basic/arm_sme_builtin_cg.inc`; contains preprocessor structure that shapes compilation of this unit.
- **CN**: 该代码块引入 Clang 头文件 `clang/Basic/arm_sve_builtin_cg.inc`, `clang/Basic/BuiltinsAArch64NeonSVEBridge_cg.def`, `clang/Basic/arm_sme_builtin_cg.inc`；包含影响本编译单元构建方式的预处理结构。

### Lines 1051-1080
```cpp
1051: static bool AArch64SVEIntrinsicsProvenSorted = false;
1052: static bool AArch64SMEIntrinsicsProvenSorted = false;
1053: 
1054: // Check if Builtin `BuiltinId` is present in `IntrinsicMap`. If yes, returns
1055: // the corresponding info struct.
1056: static const ARMVectorIntrinsicInfo *
1057: findARMVectorIntrinsicInMap(ArrayRef<ARMVectorIntrinsicInfo> IntrinsicMap,
1058:                             unsigned BuiltinID, bool &MapProvenSorted) {
1059: 
1060: #ifndef NDEBUG
1061:   if (!MapProvenSorted) {
1062:     assert(llvm::is_sorted(IntrinsicMap));
1063:     MapProvenSorted = true;
1064:   }
1065: #endif
1066: 
1067:   const ARMVectorIntrinsicInfo *Builtin =
1068:       llvm::lower_bound(IntrinsicMap, BuiltinID);
1069: 
1070:   if (Builtin != IntrinsicMap.end() && Builtin->BuiltinID == BuiltinID)
1071:     return Builtin;
1072: 
1073:   return nullptr;
1074: }
1075: 
1076: Function *CodeGenFunction::LookupNeonLLVMIntrinsic(unsigned IntrinsicID,
1077:                                                    unsigned Modifier,
1078:                                                    llvm::Type *ArgType,
1079:                                                    const CallExpr *E) {
1080:   int VectorSize = 0;
```
- **EN**: This block defines callable entry points like `findARMVectorIntrinsicInMap`, `lower_bound`; uses control flow (if) to specialize target builtin lowering; guards important invariants with assertions or unreachable markers; contains preprocessor structure that shapes compilation of this unit.
- **CN**: 该代码块定义可调用入口，例如 `findARMVectorIntrinsicInMap`, `lower_bound`；通过控制流（if）细化 目标内建函数降级 行为；使用断言或不可达标记保护关键不变量；包含影响本编译单元构建方式的预处理结构。

### Lines 1081-1110
```cpp
1081:   if (Modifier & Use64BitVectors)
1082:     VectorSize = 64;
1083:   else if (Modifier & Use128BitVectors)
1084:     VectorSize = 128;
1085: 
1086:   // Return type.
1087:   SmallVector<llvm::Type *, 3> Tys;
1088:   if (Modifier & AddRetType) {
1089:     llvm::Type *Ty = ConvertType(E->getCallReturnType(getContext()));
1090:     if (Modifier & VectorizeRetType)
1091:       Ty = llvm::FixedVectorType::get(
1092:           Ty, VectorSize ? VectorSize / Ty->getPrimitiveSizeInBits() : 1);
1093: 
1094:     Tys.push_back(Ty);
1095:   }
1096: 
1097:   // Arguments.
1098:   if (Modifier & VectorizeArgTypes) {
1099:     int Elts = VectorSize ? VectorSize / ArgType->getPrimitiveSizeInBits() : 1;
1100:     ArgType = llvm::FixedVectorType::get(ArgType, Elts);
1101:   }
1102: 
1103:   if (Modifier & (Add1ArgType | Add2ArgTypes))
1104:     Tys.push_back(ArgType);
1105: 
1106:   if (Modifier & Add2ArgTypes)
1107:     Tys.push_back(ArgType);
1108: 
1109:   if (Modifier & InventFloatType)
1110:     Tys.push_back(FloatTy);
```
- **EN**: This block uses control flow (if) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（if）细化 目标内建函数降级 行为。

### Lines 1111-1140
```cpp
1111: 
1112:   return CGM.getIntrinsic(IntrinsicID, Tys);
1113: }
1114: 
1115: //===----------------------------------------------------------------------===//
1116: //  Emit-helpers
1117: //===----------------------------------------------------------------------===//
1118: static Value *EmitCommonNeonSISDBuiltinExpr(
1119:     CodeGenFunction &CGF, const ARMVectorIntrinsicInfo &SISDInfo,
1120:     SmallVectorImpl<Value *> &Ops, const CallExpr *E) {
1121:   assert(SISDInfo.LLVMIntrinsic && "Generic code assumes a valid intrinsic");
1122: 
1123:   switch (SISDInfo.BuiltinID) {
1124:   case NEON::BI__builtin_neon_vcled_s64:
1125:   case NEON::BI__builtin_neon_vcled_u64:
1126:   case NEON::BI__builtin_neon_vcles_f32:
1127:   case NEON::BI__builtin_neon_vcled_f64:
1128:   case NEON::BI__builtin_neon_vcltd_s64:
1129:   case NEON::BI__builtin_neon_vcltd_u64:
1130:   case NEON::BI__builtin_neon_vclts_f32:
1131:   case NEON::BI__builtin_neon_vcltd_f64:
1132:   case NEON::BI__builtin_neon_vcales_f32:
1133:   case NEON::BI__builtin_neon_vcaled_f64:
1134:   case NEON::BI__builtin_neon_vcalts_f32:
1135:   case NEON::BI__builtin_neon_vcaltd_f64:
1136:     // Only one direction of comparisons actually exist, cmle is actually a cmge
1137:     // with swapped operands. The table gives us the right intrinsic but we
1138:     // still need to do the swap.
1139:     std::swap(Ops[0], Ops[1]);
1140:     break;
```
- **EN**: This block defines callable entry points like `swap`; uses control flow (switch, case) to specialize target builtin lowering; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块定义可调用入口，例如 `swap`；通过控制流（switch, case）细化 目标内建函数降级 行为；使用断言或不可达标记保护关键不变量。

### Lines 1141-1170
```cpp
1141:   }
1142: 
1143:   // Use fptosi.sat/fptoui.sat unless under strict FP.
1144:   unsigned LLVMIntrinsic = SISDInfo.LLVMIntrinsic;
1145:   if (!CGF.Builder.getIsFPConstrained()) {
1146:     if (LLVMIntrinsic == Intrinsic::aarch64_neon_fcvtzs)
1147:       LLVMIntrinsic = Intrinsic::fptosi_sat;
1148:     else if (LLVMIntrinsic == Intrinsic::aarch64_neon_fcvtzu)
1149:       LLVMIntrinsic = Intrinsic::fptoui_sat;
1150:   }
1151:   llvm::Type *ArgTy = CGF.ConvertType(E->getArg(0)->getType());
1152:   Function *F = CGF.LookupNeonLLVMIntrinsic(LLVMIntrinsic,
1153:                                             SISDInfo.TypeModifier, ArgTy, E);
1154: 
1155:   int j = 0;
1156:   ConstantInt *C0 = ConstantInt::get(CGF.SizeTy, 0);
1157:   for (Function::const_arg_iterator ai = F->arg_begin(), ae = F->arg_end();
1158:        ai != ae; ++ai, ++j) {
1159:     llvm::Type *ArgTy = ai->getType();
1160:     if (Ops[j]->getType()->getPrimitiveSizeInBits() ==
1161:              ArgTy->getPrimitiveSizeInBits())
1162:       continue;
1163:     assert(
1164:         ArgTy->isVectorTy() && !Ops[j]->getType()->isVectorTy() &&
1165:         "Expecting vector LLVM intrinsic type and scalar Clang builtin type!");
1166: 
1167:     // The constant argument to an _n_ intrinsic always has Int32Ty, so truncate
1168:     // it before inserting.
1169:     Ops[j] = CGF.Builder.CreateTruncOrBitCast(
1170:         Ops[j], cast<llvm::VectorType>(ArgTy)->getElementType());
```
- **EN**: This block uses control flow (if, for) to specialize target builtin lowering; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块通过控制流（if, for）细化 目标内建函数降级 行为；使用断言或不可达标记保护关键不变量。

### Lines 1171-1200
```cpp
1171:     Ops[j] =
1172:         CGF.Builder.CreateInsertElement(PoisonValue::get(ArgTy), Ops[j], C0);
1173:   }
1174: 
1175:   Value *Result = CGF.EmitNeonCall(F, Ops, SISDInfo.NameHint);
1176:   llvm::Type *ResultType = CGF.ConvertType(E->getType());
1177:   if (ResultType->getPrimitiveSizeInBits().getFixedValue() <
1178:       Result->getType()->getPrimitiveSizeInBits().getFixedValue())
1179:     return CGF.Builder.CreateExtractElement(Result, C0);
1180: 
1181:   return CGF.Builder.CreateBitCast(Result, ResultType, SISDInfo.NameHint);
1182: }
1183: 
1184: Value *CodeGenFunction::EmitCommonNeonBuiltinExpr(
1185:     unsigned BuiltinID, unsigned LLVMIntrinsic, unsigned AltLLVMIntrinsic,
1186:     const char *NameHint, unsigned Modifier, const CallExpr *E,
1187:     SmallVectorImpl<llvm::Value *> &Ops, Address PtrOp0, Address PtrOp1,
1188:     llvm::Triple::ArchType Arch) {
1189: 
1190:   // Extract the trailing immediate argument that encodes the type discriminator
1191:   // for this overloaded intrinsic.
1192:   // TODO: Move to the parent code that takes care of argument processing.
1193:   const Expr *Arg = E->getArg(E->getNumArgs() - 1);
1194:   std::optional<llvm::APSInt> NeonTypeConst =
1195:       Arg->getIntegerConstantExpr(getContext());
1196:   if (!NeonTypeConst)
1197:     return nullptr;
1198: 
1199:   // Determine the type of this overloaded NEON intrinsic.
1200:   NeonTypeFlags Type(NeonTypeConst->getZExtValue());
```
- **EN**: This block defines callable entry points like `Type`; uses control flow (if) to specialize target builtin lowering.
- **CN**: 该代码块定义可调用入口，例如 `Type`；通过控制流（if）细化 目标内建函数降级 行为。

### Lines 1201-1230
```cpp
1201:   const bool Usgn = Type.isUnsigned();
1202:   const bool Quad = Type.isQuad();
1203:   const bool Floating = Type.isFloatingPoint();
1204:   const bool HasFastHalfType = getTarget().hasFastHalfType();
1205:   const bool AllowBFloatArgsAndRet =
1206:       getTargetHooks().getABIInfo().allowBFloatArgsAndRet();
1207: 
1208:   llvm::FixedVectorType *VTy =
1209:       GetNeonType(this, Type, HasFastHalfType, false, AllowBFloatArgsAndRet);
1210:   llvm::Type *Ty = VTy;
1211:   if (!Ty)
1212:     return nullptr;
1213: 
1214:   auto getAlignmentValue32 = [&](Address addr) -> Value* {
1215:     return Builder.getInt32(addr.getAlignment().getQuantity());
1216:   };
1217: 
1218:   unsigned Int = LLVMIntrinsic;
1219:   if ((Modifier & UnsignedAlts) && !Usgn)
1220:     Int = AltLLVMIntrinsic;
1221: 
1222:   switch (BuiltinID) {
1223:   default: break;
1224:   case NEON::BI__builtin_neon_splat_lane_v:
1225:   case NEON::BI__builtin_neon_splat_laneq_v:
1226:   case NEON::BI__builtin_neon_splatq_lane_v:
1227:   case NEON::BI__builtin_neon_splatq_laneq_v: {
1228:     auto NumElements = VTy->getElementCount();
1229:     if (BuiltinID == NEON::BI__builtin_neon_splatq_lane_v)
1230:       NumElements = NumElements * 2;
```
- **EN**: This block defines callable entry points like `getTargetHooks`, `GetNeonType`; uses control flow (if, switch, case) to specialize target builtin lowering.
- **CN**: 该代码块定义可调用入口，例如 `getTargetHooks`, `GetNeonType`；通过控制流（if, switch, case）细化 目标内建函数降级 行为。

### Lines 1231-1260
```cpp
1231:     if (BuiltinID == NEON::BI__builtin_neon_splat_laneq_v)
1232:       NumElements = NumElements.divideCoefficientBy(2);
1233: 
1234:     Ops[0] = Builder.CreateBitCast(Ops[0], VTy);
1235:     return EmitNeonSplat(Ops[0], cast<ConstantInt>(Ops[1]), NumElements);
1236:   }
1237:   case NEON::BI__builtin_neon_vpadd_v:
1238:   case NEON::BI__builtin_neon_vpaddq_v:
1239:     // We don't allow fp/int overloading of intrinsics.
1240:     if (VTy->getElementType()->isFloatingPointTy() &&
1241:         Int == Intrinsic::aarch64_neon_addp)
1242:       Int = Intrinsic::aarch64_neon_faddp;
1243:     break;
1244:   case NEON::BI__builtin_neon_vabs_v:
1245:   case NEON::BI__builtin_neon_vabsq_v:
1246:     if (VTy->getElementType()->isFloatingPointTy())
1247:       return EmitNeonCall(CGM.getIntrinsic(Intrinsic::fabs, Ty), Ops, "vabs");
1248:     return EmitNeonCall(CGM.getIntrinsic(LLVMIntrinsic, Ty), Ops, "vabs");
1249:   case NEON::BI__builtin_neon_vadd_v:
1250:   case NEON::BI__builtin_neon_vaddq_v: {
1251:     llvm::Type *VTy = llvm::FixedVectorType::get(Int8Ty, Quad ? 16 : 8);
1252:     Ops[0] = Builder.CreateBitCast(Ops[0], VTy);
1253:     Ops[1] = Builder.CreateBitCast(Ops[1], VTy);
1254:     Ops[0] =  Builder.CreateXor(Ops[0], Ops[1]);
1255:     return Builder.CreateBitCast(Ops[0], Ty);
1256:   }
1257:   case NEON::BI__builtin_neon_vaddhn_v: {
1258:     llvm::FixedVectorType *SrcTy =
1259:         llvm::FixedVectorType::getExtendedElementVectorType(VTy);
1260: 
```
- **EN**: This block defines callable entry points like `EmitNeonSplat`, `EmitNeonCall`, `getExtendedElementVectorType`; uses control flow (if, case) to specialize target builtin lowering.
- **CN**: 该代码块定义可调用入口，例如 `EmitNeonSplat`, `EmitNeonCall`, `getExtendedElementVectorType`；通过控制流（if, case）细化 目标内建函数降级 行为。

### Lines 1261-1290
```cpp
1261:     // %sum = add <4 x i32> %lhs, %rhs
1262:     Ops[0] = Builder.CreateBitCast(Ops[0], SrcTy);
1263:     Ops[1] = Builder.CreateBitCast(Ops[1], SrcTy);
1264:     Ops[0] = Builder.CreateAdd(Ops[0], Ops[1], "vaddhn");
1265: 
1266:     // %high = lshr <4 x i32> %sum, <i32 16, i32 16, i32 16, i32 16>
1267:     Constant *ShiftAmt =
1268:         ConstantInt::get(SrcTy, SrcTy->getScalarSizeInBits() / 2);
1269:     Ops[0] = Builder.CreateLShr(Ops[0], ShiftAmt, "vaddhn");
1270: 
1271:     // %res = trunc <4 x i32> %high to <4 x i16>
1272:     return Builder.CreateTrunc(Ops[0], VTy, "vaddhn");
1273:   }
1274:   case NEON::BI__builtin_neon_vcale_v:
1275:   case NEON::BI__builtin_neon_vcaleq_v:
1276:   case NEON::BI__builtin_neon_vcalt_v:
1277:   case NEON::BI__builtin_neon_vcaltq_v:
1278:     std::swap(Ops[0], Ops[1]);
1279:     [[fallthrough]];
1280:   case NEON::BI__builtin_neon_vcage_v:
1281:   case NEON::BI__builtin_neon_vcageq_v:
1282:   case NEON::BI__builtin_neon_vcagt_v:
1283:   case NEON::BI__builtin_neon_vcagtq_v: {
1284:     llvm::Type *Ty;
1285:     switch (VTy->getScalarSizeInBits()) {
1286:     default: llvm_unreachable("unexpected type");
1287:     case 32:
1288:       Ty = FloatTy;
1289:       break;
1290:     case 64:
```
- **EN**: This block defines callable entry points like `get`, `swap`; uses control flow (switch, case) to specialize target builtin lowering; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块定义可调用入口，例如 `get`, `swap`；通过控制流（switch, case）细化 目标内建函数降级 行为；使用断言或不可达标记保护关键不变量。

### Lines 1291-1320
```cpp
1291:       Ty = DoubleTy;
1292:       break;
1293:     case 16:
1294:       Ty = HalfTy;
1295:       break;
1296:     }
1297:     auto *VecFlt = llvm::FixedVectorType::get(Ty, VTy->getNumElements());
1298:     llvm::Type *Tys[] = { VTy, VecFlt };
1299:     Function *F = CGM.getIntrinsic(LLVMIntrinsic, Tys);
1300:     return EmitNeonCall(F, Ops, NameHint);
1301:   }
1302:   case NEON::BI__builtin_neon_vceqz_v:
1303:   case NEON::BI__builtin_neon_vceqzq_v:
1304:     return EmitAArch64CompareBuiltinExpr(
1305:         Ops[0], Ty, Floating ? ICmpInst::FCMP_OEQ : ICmpInst::ICMP_EQ, "vceqz");
1306:   case NEON::BI__builtin_neon_vcgez_v:
1307:   case NEON::BI__builtin_neon_vcgezq_v:
1308:     return EmitAArch64CompareBuiltinExpr(
1309:         Ops[0], Ty, Floating ? ICmpInst::FCMP_OGE : ICmpInst::ICMP_SGE,
1310:         "vcgez");
1311:   case NEON::BI__builtin_neon_vclez_v:
1312:   case NEON::BI__builtin_neon_vclezq_v:
1313:     return EmitAArch64CompareBuiltinExpr(
1314:         Ops[0], Ty, Floating ? ICmpInst::FCMP_OLE : ICmpInst::ICMP_SLE,
1315:         "vclez");
1316:   case NEON::BI__builtin_neon_vcgtz_v:
1317:   case NEON::BI__builtin_neon_vcgtzq_v:
1318:     return EmitAArch64CompareBuiltinExpr(
1319:         Ops[0], Ty, Floating ? ICmpInst::FCMP_OGT : ICmpInst::ICMP_SGT,
1320:         "vcgtz");
```
- **EN**: This block defines callable entry points like `EmitNeonCall`, `EmitAArch64CompareBuiltinExpr`; uses control flow (case) to specialize target builtin lowering.
- **CN**: 该代码块定义可调用入口，例如 `EmitNeonCall`, `EmitAArch64CompareBuiltinExpr`；通过控制流（case）细化 目标内建函数降级 行为。

### Lines 1321-1350
```cpp
1321:   case NEON::BI__builtin_neon_vcltz_v:
1322:   case NEON::BI__builtin_neon_vcltzq_v:
1323:     return EmitAArch64CompareBuiltinExpr(
1324:         Ops[0], Ty, Floating ? ICmpInst::FCMP_OLT : ICmpInst::ICMP_SLT,
1325:         "vcltz");
1326:   case NEON::BI__builtin_neon_vclz_v:
1327:   case NEON::BI__builtin_neon_vclzq_v:
1328:     // We generate target-independent intrinsic, which needs a second argument
1329:     // for whether or not clz of zero is undefined; on ARM it isn't.
1330:     Ops.push_back(Builder.getInt1(getTarget().isCLZForZeroUndef()));
1331:     break;
1332:   case NEON::BI__builtin_neon_vcvt_f32_v:
1333:   case NEON::BI__builtin_neon_vcvtq_f32_v:
1334:     Ops[0] = Builder.CreateBitCast(Ops[0], Ty);
1335:     Ty = GetNeonType(this, NeonTypeFlags(NeonTypeFlags::Float32, false, Quad),
1336:                      HasFastHalfType);
1337:     return Usgn ? Builder.CreateUIToFP(Ops[0], Ty, "vcvt")
1338:                 : Builder.CreateSIToFP(Ops[0], Ty, "vcvt");
1339:   case NEON::BI__builtin_neon_vcvt_f16_s16:
1340:   case NEON::BI__builtin_neon_vcvt_f16_u16:
1341:   case NEON::BI__builtin_neon_vcvtq_f16_s16:
1342:   case NEON::BI__builtin_neon_vcvtq_f16_u16:
1343:     Ops[0] = Builder.CreateBitCast(Ops[0], Ty);
1344:     Ty = GetNeonType(this, NeonTypeFlags(NeonTypeFlags::Float16, false, Quad),
1345:                      HasFastHalfType);
1346:     return Usgn ? Builder.CreateUIToFP(Ops[0], Ty, "vcvt")
1347:                 : Builder.CreateSIToFP(Ops[0], Ty, "vcvt");
1348:   case NEON::BI__builtin_neon_vcvt_n_f16_s16:
1349:   case NEON::BI__builtin_neon_vcvt_n_f16_u16:
1350:   case NEON::BI__builtin_neon_vcvtq_n_f16_s16:
```
- **EN**: This block spells out callable entry points like `EmitAArch64CompareBuiltinExpr`; uses control flow (case) to specialize target builtin lowering.
- **CN**: 该代码块给出可调用入口的声明，例如 `EmitAArch64CompareBuiltinExpr`；通过控制流（case）细化 目标内建函数降级 行为。

### Lines 1351-1380
```cpp
1351:   case NEON::BI__builtin_neon_vcvtq_n_f16_u16: {
1352:     llvm::Type *Tys[2] = { GetFloatNeonType(this, Type), Ty };
1353:     Function *F = CGM.getIntrinsic(Int, Tys);
1354:     return EmitNeonCall(F, Ops, "vcvt_n");
1355:   }
1356:   case NEON::BI__builtin_neon_vcvt_n_f32_v:
1357:   case NEON::BI__builtin_neon_vcvt_n_f64_v:
1358:   case NEON::BI__builtin_neon_vcvtq_n_f32_v:
1359:   case NEON::BI__builtin_neon_vcvtq_n_f64_v: {
1360:     llvm::Type *Tys[2] = { GetFloatNeonType(this, Type), Ty };
1361:     Int = Usgn ? LLVMIntrinsic : AltLLVMIntrinsic;
1362:     Function *F = CGM.getIntrinsic(Int, Tys);
1363:     return EmitNeonCall(F, Ops, "vcvt_n");
1364:   }
1365:   case NEON::BI__builtin_neon_vcvt_n_s16_f16:
1366:   case NEON::BI__builtin_neon_vcvt_n_s32_v:
1367:   case NEON::BI__builtin_neon_vcvt_n_u16_f16:
1368:   case NEON::BI__builtin_neon_vcvt_n_u32_v:
1369:   case NEON::BI__builtin_neon_vcvt_n_s64_v:
1370:   case NEON::BI__builtin_neon_vcvt_n_u64_v:
1371:   case NEON::BI__builtin_neon_vcvtq_n_s16_f16:
1372:   case NEON::BI__builtin_neon_vcvtq_n_s32_v:
1373:   case NEON::BI__builtin_neon_vcvtq_n_u16_f16:
1374:   case NEON::BI__builtin_neon_vcvtq_n_u32_v:
1375:   case NEON::BI__builtin_neon_vcvtq_n_s64_v:
1376:   case NEON::BI__builtin_neon_vcvtq_n_u64_v: {
1377:     llvm::Type *Tys[2] = { Ty, GetFloatNeonType(this, Type) };
1378:     Function *F = CGM.getIntrinsic(LLVMIntrinsic, Tys);
1379:     return EmitNeonCall(F, Ops, "vcvt_n");
1380:   }
```
- **EN**: This block defines callable entry points like `EmitNeonCall`; uses control flow (case) to specialize target builtin lowering.
- **CN**: 该代码块定义可调用入口，例如 `EmitNeonCall`；通过控制流（case）细化 目标内建函数降级 行为。

### Lines 1381-1410
```cpp
1381:   case NEON::BI__builtin_neon_vcvt_s32_v:
1382:   case NEON::BI__builtin_neon_vcvt_u32_v:
1383:   case NEON::BI__builtin_neon_vcvt_s64_v:
1384:   case NEON::BI__builtin_neon_vcvt_u64_v:
1385:   case NEON::BI__builtin_neon_vcvt_s16_f16:
1386:   case NEON::BI__builtin_neon_vcvt_u16_f16:
1387:   case NEON::BI__builtin_neon_vcvtq_s32_v:
1388:   case NEON::BI__builtin_neon_vcvtq_u32_v:
1389:   case NEON::BI__builtin_neon_vcvtq_s64_v:
1390:   case NEON::BI__builtin_neon_vcvtq_u64_v:
1391:   case NEON::BI__builtin_neon_vcvtq_s16_f16:
1392:   case NEON::BI__builtin_neon_vcvtq_u16_f16: {
1393:     Ops[0] = Builder.CreateBitCast(Ops[0], GetFloatNeonType(this, Type));
1394:     if (Int) {
1395:       // AArch64: use fptosi.sat/fptoui.sat unless under strict FP.
1396:       if (!Builder.getIsFPConstrained())
1397:         Int = Usgn ? Intrinsic::fptoui_sat : Intrinsic::fptosi_sat;
1398:       llvm::Type *Tys[2] = {Ty, Ops[0]->getType()};
1399:       return EmitNeonCall(CGM.getIntrinsic(Int, Tys), Ops, "vcvtz");
1400:     }
1401:     // FIXME: ARM uses plain fptoui/fptosi which have UB on out-of-range
1402:     // values. These should also use saturating intrinsics.
1403:     return Usgn ? Builder.CreateFPToUI(Ops[0], Ty, "vcvt")
1404:                 : Builder.CreateFPToSI(Ops[0], Ty, "vcvt");
1405:   }
1406:   case NEON::BI__builtin_neon_vcvta_s16_f16:
1407:   case NEON::BI__builtin_neon_vcvta_s32_v:
1408:   case NEON::BI__builtin_neon_vcvta_s64_v:
1409:   case NEON::BI__builtin_neon_vcvta_u16_f16:
1410:   case NEON::BI__builtin_neon_vcvta_u32_v:
```
- **EN**: This block defines callable entry points like `EmitNeonCall`; uses control flow (if, case) to specialize target builtin lowering.
- **CN**: 该代码块定义可调用入口，例如 `EmitNeonCall`；通过控制流（if, case）细化 目标内建函数降级 行为。

### Lines 1411-1440
```cpp
1411:   case NEON::BI__builtin_neon_vcvta_u64_v:
1412:   case NEON::BI__builtin_neon_vcvtaq_s16_f16:
1413:   case NEON::BI__builtin_neon_vcvtaq_s32_v:
1414:   case NEON::BI__builtin_neon_vcvtaq_s64_v:
1415:   case NEON::BI__builtin_neon_vcvtaq_u16_f16:
1416:   case NEON::BI__builtin_neon_vcvtaq_u32_v:
1417:   case NEON::BI__builtin_neon_vcvtaq_u64_v:
1418:   case NEON::BI__builtin_neon_vcvtn_s16_f16:
1419:   case NEON::BI__builtin_neon_vcvtn_s32_v:
1420:   case NEON::BI__builtin_neon_vcvtn_s64_v:
1421:   case NEON::BI__builtin_neon_vcvtn_u16_f16:
1422:   case NEON::BI__builtin_neon_vcvtn_u32_v:
1423:   case NEON::BI__builtin_neon_vcvtn_u64_v:
1424:   case NEON::BI__builtin_neon_vcvtnq_s16_f16:
1425:   case NEON::BI__builtin_neon_vcvtnq_s32_v:
1426:   case NEON::BI__builtin_neon_vcvtnq_s64_v:
1427:   case NEON::BI__builtin_neon_vcvtnq_u16_f16:
1428:   case NEON::BI__builtin_neon_vcvtnq_u32_v:
1429:   case NEON::BI__builtin_neon_vcvtnq_u64_v:
1430:   case NEON::BI__builtin_neon_vcvtp_s16_f16:
1431:   case NEON::BI__builtin_neon_vcvtp_s32_v:
1432:   case NEON::BI__builtin_neon_vcvtp_s64_v:
1433:   case NEON::BI__builtin_neon_vcvtp_u16_f16:
1434:   case NEON::BI__builtin_neon_vcvtp_u32_v:
1435:   case NEON::BI__builtin_neon_vcvtp_u64_v:
1436:   case NEON::BI__builtin_neon_vcvtpq_s16_f16:
1437:   case NEON::BI__builtin_neon_vcvtpq_s32_v:
1438:   case NEON::BI__builtin_neon_vcvtpq_s64_v:
1439:   case NEON::BI__builtin_neon_vcvtpq_u16_f16:
1440:   case NEON::BI__builtin_neon_vcvtpq_u32_v:
```
- **EN**: This block uses control flow (case) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（case）细化 目标内建函数降级 行为。

### Lines 1441-1470
```cpp
1441:   case NEON::BI__builtin_neon_vcvtpq_u64_v:
1442:   case NEON::BI__builtin_neon_vcvtm_s16_f16:
1443:   case NEON::BI__builtin_neon_vcvtm_s32_v:
1444:   case NEON::BI__builtin_neon_vcvtm_s64_v:
1445:   case NEON::BI__builtin_neon_vcvtm_u16_f16:
1446:   case NEON::BI__builtin_neon_vcvtm_u32_v:
1447:   case NEON::BI__builtin_neon_vcvtm_u64_v:
1448:   case NEON::BI__builtin_neon_vcvtmq_s16_f16:
1449:   case NEON::BI__builtin_neon_vcvtmq_s32_v:
1450:   case NEON::BI__builtin_neon_vcvtmq_s64_v:
1451:   case NEON::BI__builtin_neon_vcvtmq_u16_f16:
1452:   case NEON::BI__builtin_neon_vcvtmq_u32_v:
1453:   case NEON::BI__builtin_neon_vcvtmq_u64_v: {
1454:     llvm::Type *Tys[2] = { Ty, GetFloatNeonType(this, Type) };
1455:     return EmitNeonCall(CGM.getIntrinsic(LLVMIntrinsic, Tys), Ops, NameHint);
1456:   }
1457:   case NEON::BI__builtin_neon_vcvtx_f32_v: {
1458:     llvm::Type *Tys[2] = { VTy->getTruncatedElementVectorType(VTy), Ty};
1459:     return EmitNeonCall(CGM.getIntrinsic(LLVMIntrinsic, Tys), Ops, NameHint);
1460: 
1461:   }
1462:   case NEON::BI__builtin_neon_vext_v:
1463:   case NEON::BI__builtin_neon_vextq_v: {
1464:     int CV = cast<ConstantInt>(Ops[2])->getSExtValue();
1465:     SmallVector<int, 16> Indices;
1466:     for (unsigned i = 0, e = VTy->getNumElements(); i != e; ++i)
1467:       Indices.push_back(i+CV);
1468: 
1469:     Ops[0] = Builder.CreateBitCast(Ops[0], Ty);
1470:     Ops[1] = Builder.CreateBitCast(Ops[1], Ty);
```
- **EN**: This block defines callable entry points like `EmitNeonCall`; uses control flow (for, case) to specialize target builtin lowering.
- **CN**: 该代码块定义可调用入口，例如 `EmitNeonCall`；通过控制流（for, case）细化 目标内建函数降级 行为。

### Lines 1471-1500
```cpp
1471:     return Builder.CreateShuffleVector(Ops[0], Ops[1], Indices, "vext");
1472:   }
1473:   case NEON::BI__builtin_neon_vfma_v:
1474:   case NEON::BI__builtin_neon_vfmaq_v: {
1475:     Ops[0] = Builder.CreateBitCast(Ops[0], Ty);
1476:     Ops[1] = Builder.CreateBitCast(Ops[1], Ty);
1477:     Ops[2] = Builder.CreateBitCast(Ops[2], Ty);
1478: 
1479:     // NEON intrinsic puts accumulator first, unlike the LLVM fma.
1480:     return emitCallMaybeConstrainedFPBuiltin(
1481:         *this, Intrinsic::fma, Intrinsic::experimental_constrained_fma, Ty,
1482:         {Ops[1], Ops[2], Ops[0]});
1483:   }
1484:   case NEON::BI__builtin_neon_vld1_v:
1485:   case NEON::BI__builtin_neon_vld1q_v: {
1486:     llvm::Type *Tys[] = {Ty, Int8PtrTy};
1487:     Ops.push_back(getAlignmentValue32(PtrOp0));
1488:     return EmitNeonCall(CGM.getIntrinsic(LLVMIntrinsic, Tys), Ops, "vld1");
1489:   }
1490:   case NEON::BI__builtin_neon_vld1_x2_v:
1491:   case NEON::BI__builtin_neon_vld1q_x2_v:
1492:   case NEON::BI__builtin_neon_vld1_x3_v:
1493:   case NEON::BI__builtin_neon_vld1q_x3_v:
1494:   case NEON::BI__builtin_neon_vld1_x4_v:
1495:   case NEON::BI__builtin_neon_vld1q_x4_v: {
1496:     llvm::Type *Tys[2] = {VTy, DefaultPtrTy};
1497:     Function *F = CGM.getIntrinsic(LLVMIntrinsic, Tys);
1498:     Ops[1] = Builder.CreateCall(F, Ops[1], "vld1xN");
1499:     return Builder.CreateDefaultAlignedStore(Ops[1], Ops[0]);
1500:   }
```
- **EN**: This block defines callable entry points like `EmitNeonCall`; uses control flow (case) to specialize target builtin lowering.
- **CN**: 该代码块定义可调用入口，例如 `EmitNeonCall`；通过控制流（case）细化 目标内建函数降级 行为。

### Lines 1501-1530
```cpp
1501:   case NEON::BI__builtin_neon_vld2_v:
1502:   case NEON::BI__builtin_neon_vld2q_v:
1503:   case NEON::BI__builtin_neon_vld3_v:
1504:   case NEON::BI__builtin_neon_vld3q_v:
1505:   case NEON::BI__builtin_neon_vld4_v:
1506:   case NEON::BI__builtin_neon_vld4q_v:
1507:   case NEON::BI__builtin_neon_vld2_dup_v:
1508:   case NEON::BI__builtin_neon_vld2q_dup_v:
1509:   case NEON::BI__builtin_neon_vld3_dup_v:
1510:   case NEON::BI__builtin_neon_vld3q_dup_v:
1511:   case NEON::BI__builtin_neon_vld4_dup_v:
1512:   case NEON::BI__builtin_neon_vld4q_dup_v: {
1513:     llvm::Type *Tys[] = {Ty, Int8PtrTy};
1514:     Function *F = CGM.getIntrinsic(LLVMIntrinsic, Tys);
1515:     Value *Align = getAlignmentValue32(PtrOp1);
1516:     Ops[1] = Builder.CreateCall(F, {Ops[1], Align}, NameHint);
1517:     return Builder.CreateDefaultAlignedStore(Ops[1], Ops[0]);
1518:   }
1519:   case NEON::BI__builtin_neon_vld1_dup_v:
1520:   case NEON::BI__builtin_neon_vld1q_dup_v: {
1521:     Value *V = PoisonValue::get(Ty);
1522:     PtrOp0 = PtrOp0.withElementType(VTy->getElementType());
1523:     LoadInst *Ld = Builder.CreateLoad(PtrOp0);
1524:     llvm::Constant *CI = ConstantInt::get(SizeTy, 0);
1525:     Ops[0] = Builder.CreateInsertElement(V, Ld, CI);
1526:     return EmitNeonSplat(Ops[0], CI);
1527:   }
1528:   case NEON::BI__builtin_neon_vld2_lane_v:
1529:   case NEON::BI__builtin_neon_vld2q_lane_v:
1530:   case NEON::BI__builtin_neon_vld3_lane_v:
```
- **EN**: This block defines callable entry points like `EmitNeonSplat`; uses control flow (case) to specialize target builtin lowering.
- **CN**: 该代码块定义可调用入口，例如 `EmitNeonSplat`；通过控制流（case）细化 目标内建函数降级 行为。

### Lines 1531-1560
```cpp
1531:   case NEON::BI__builtin_neon_vld3q_lane_v:
1532:   case NEON::BI__builtin_neon_vld4_lane_v:
1533:   case NEON::BI__builtin_neon_vld4q_lane_v: {
1534:     llvm::Type *Tys[] = {Ty, Int8PtrTy};
1535:     Function *F = CGM.getIntrinsic(LLVMIntrinsic, Tys);
1536:     for (unsigned I = 2; I < Ops.size() - 1; ++I)
1537:       Ops[I] = Builder.CreateBitCast(Ops[I], Ty);
1538:     Ops.push_back(getAlignmentValue32(PtrOp1));
1539:     Ops[1] = Builder.CreateCall(F, ArrayRef(Ops).slice(1), NameHint);
1540:     return Builder.CreateDefaultAlignedStore(Ops[1], Ops[0]);
1541:   }
1542:   case NEON::BI__builtin_neon_vmovl_v: {
1543:     llvm::FixedVectorType *DTy =
1544:         llvm::FixedVectorType::getTruncatedElementVectorType(VTy);
1545:     Ops[0] = Builder.CreateBitCast(Ops[0], DTy);
1546:     if (Usgn)
1547:       return Builder.CreateZExt(Ops[0], Ty, "vmovl");
1548:     return Builder.CreateSExt(Ops[0], Ty, "vmovl");
1549:   }
1550:   case NEON::BI__builtin_neon_vmovn_v: {
1551:     llvm::FixedVectorType *QTy =
1552:         llvm::FixedVectorType::getExtendedElementVectorType(VTy);
1553:     Ops[0] = Builder.CreateBitCast(Ops[0], QTy);
1554:     return Builder.CreateTrunc(Ops[0], Ty, "vmovn");
1555:   }
1556:   case NEON::BI__builtin_neon_vmull_v:
1557:     // FIXME: the integer vmull operations could be emitted in terms of pure
1558:     // LLVM IR (2 exts followed by a mul). Unfortunately LLVM has a habit of
1559:     // hoisting the exts outside loops. Until global ISel comes along that can
1560:     // see through such movement this leads to bad CodeGen. So we need an
```
- **EN**: This block defines callable entry points like `getTruncatedElementVectorType`, `getExtendedElementVectorType`; uses control flow (if, for, case) to specialize target builtin lowering.
- **CN**: 该代码块定义可调用入口，例如 `getTruncatedElementVectorType`, `getExtendedElementVectorType`；通过控制流（if, for, case）细化 目标内建函数降级 行为。

### Lines 1561-1590
```cpp
1561:     // intrinsic for now.
1562:     Int = Usgn ? Intrinsic::arm_neon_vmullu : Intrinsic::arm_neon_vmulls;
1563:     Int = Type.isPoly() ? (unsigned)Intrinsic::arm_neon_vmullp : Int;
1564:     return EmitNeonCall(CGM.getIntrinsic(Int, Ty), Ops, "vmull");
1565:   case NEON::BI__builtin_neon_vpadal_v:
1566:   case NEON::BI__builtin_neon_vpadalq_v: {
1567:     // The source operand type has twice as many elements of half the size.
1568:     unsigned EltBits = VTy->getElementType()->getPrimitiveSizeInBits();
1569:     llvm::Type *EltTy =
1570:       llvm::IntegerType::get(getLLVMContext(), EltBits / 2);
1571:     auto *NarrowTy =
1572:         llvm::FixedVectorType::get(EltTy, VTy->getNumElements() * 2);
1573:     llvm::Type *Tys[2] = { Ty, NarrowTy };
1574:     return EmitNeonCall(CGM.getIntrinsic(Int, Tys), Ops, NameHint);
1575:   }
1576:   case NEON::BI__builtin_neon_vpaddl_v:
1577:   case NEON::BI__builtin_neon_vpaddlq_v: {
1578:     // The source operand type has twice as many elements of half the size.
1579:     unsigned EltBits = VTy->getElementType()->getPrimitiveSizeInBits();
1580:     llvm::Type *EltTy = llvm::IntegerType::get(getLLVMContext(), EltBits / 2);
1581:     auto *NarrowTy =
1582:         llvm::FixedVectorType::get(EltTy, VTy->getNumElements() * 2);
1583:     llvm::Type *Tys[2] = { Ty, NarrowTy };
1584:     return EmitNeonCall(CGM.getIntrinsic(Int, Tys), Ops, "vpaddl");
1585:   }
1586:   case NEON::BI__builtin_neon_vqdmlal_v:
1587:   case NEON::BI__builtin_neon_vqdmlsl_v: {
1588:     SmallVector<Value *, 2> MulOps(Ops.begin() + 1, Ops.end());
1589:     Ops[1] =
1590:         EmitNeonCall(CGM.getIntrinsic(LLVMIntrinsic, Ty), MulOps, "vqdmlal");
```
- **EN**: This block defines callable entry points like `EmitNeonCall`, `get`, `MulOps`; uses control flow (case) to specialize target builtin lowering.
- **CN**: 该代码块定义可调用入口，例如 `EmitNeonCall`, `get`, `MulOps`；通过控制流（case）细化 目标内建函数降级 行为。

### Lines 1591-1620
```cpp
1591:     Ops.resize(2);
1592:     return EmitNeonCall(CGM.getIntrinsic(AltLLVMIntrinsic, Ty), Ops, NameHint);
1593:   }
1594:   case NEON::BI__builtin_neon_vqdmulhq_lane_v:
1595:   case NEON::BI__builtin_neon_vqdmulh_lane_v:
1596:   case NEON::BI__builtin_neon_vqrdmulhq_lane_v:
1597:   case NEON::BI__builtin_neon_vqrdmulh_lane_v: {
1598:     auto *RTy = cast<llvm::FixedVectorType>(Ty);
1599:     if (BuiltinID == NEON::BI__builtin_neon_vqdmulhq_lane_v ||
1600:         BuiltinID == NEON::BI__builtin_neon_vqrdmulhq_lane_v)
1601:       RTy = llvm::FixedVectorType::get(RTy->getElementType(),
1602:                                        RTy->getNumElements() * 2);
1603:     llvm::Type *Tys[2] = {
1604:         RTy, GetNeonType(this, NeonTypeFlags(Type.getEltType(), false,
1605:                                              /*isQuad*/ false))};
1606:     return EmitNeonCall(CGM.getIntrinsic(Int, Tys), Ops, NameHint);
1607:   }
1608:   case NEON::BI__builtin_neon_vqdmulhq_laneq_v:
1609:   case NEON::BI__builtin_neon_vqdmulh_laneq_v:
1610:   case NEON::BI__builtin_neon_vqrdmulhq_laneq_v:
1611:   case NEON::BI__builtin_neon_vqrdmulh_laneq_v: {
1612:     llvm::Type *Tys[2] = {
1613:         Ty, GetNeonType(this, NeonTypeFlags(Type.getEltType(), false,
1614:                                             /*isQuad*/ true))};
1615:     return EmitNeonCall(CGM.getIntrinsic(Int, Tys), Ops, NameHint);
1616:   }
1617:   case NEON::BI__builtin_neon_vqshl_n_v:
1618:   case NEON::BI__builtin_neon_vqshlq_n_v:
1619:     return EmitNeonCall(CGM.getIntrinsic(Int, Ty), Ops, "vqshl_n",
1620:                         1, false);
```
- **EN**: This block defines callable entry points like `EmitNeonCall`, `GetNeonType`; uses control flow (if, case) to specialize target builtin lowering.
- **CN**: 该代码块定义可调用入口，例如 `EmitNeonCall`, `GetNeonType`；通过控制流（if, case）细化 目标内建函数降级 行为。

### Lines 1621-1650
```cpp
1621:   case NEON::BI__builtin_neon_vqshlu_n_v:
1622:   case NEON::BI__builtin_neon_vqshluq_n_v:
1623:     return EmitNeonCall(CGM.getIntrinsic(Int, Ty), Ops, "vqshlu_n",
1624:                         1, false);
1625:   case NEON::BI__builtin_neon_vrecpe_v:
1626:   case NEON::BI__builtin_neon_vrecpeq_v:
1627:   case NEON::BI__builtin_neon_vrsqrte_v:
1628:   case NEON::BI__builtin_neon_vrsqrteq_v:
1629:     Int = Ty->isFPOrFPVectorTy() ? LLVMIntrinsic : AltLLVMIntrinsic;
1630:     return EmitNeonCall(CGM.getIntrinsic(Int, Ty), Ops, NameHint);
1631:   case NEON::BI__builtin_neon_vrndi_v:
1632:   case NEON::BI__builtin_neon_vrndiq_v:
1633:     Int = Builder.getIsFPConstrained()
1634:               ? Intrinsic::experimental_constrained_nearbyint
1635:               : Intrinsic::nearbyint;
1636:     return EmitNeonCall(CGM.getIntrinsic(Int, Ty), Ops, NameHint);
1637:   case NEON::BI__builtin_neon_vrshr_n_v:
1638:   case NEON::BI__builtin_neon_vrshrq_n_v:
1639:     return EmitNeonCall(CGM.getIntrinsic(Int, Ty), Ops, "vrshr_n",
1640:                         1, true);
1641:   case NEON::BI__builtin_neon_vsha512hq_u64:
1642:   case NEON::BI__builtin_neon_vsha512h2q_u64:
1643:   case NEON::BI__builtin_neon_vsha512su0q_u64:
1644:   case NEON::BI__builtin_neon_vsha512su1q_u64: {
1645:     Function *F = CGM.getIntrinsic(Int);
1646:     return EmitNeonCall(F, Ops, "");
1647:   }
1648:   case NEON::BI__builtin_neon_vshl_n_v:
1649:   case NEON::BI__builtin_neon_vshlq_n_v:
1650:     Ops[1] = EmitNeonShiftVector(Ops[1], Ty, false);
```
- **EN**: This block defines callable entry points like `EmitNeonCall`; uses control flow (case) to specialize target builtin lowering.
- **CN**: 该代码块定义可调用入口，例如 `EmitNeonCall`；通过控制流（case）细化 目标内建函数降级 行为。

### Lines 1651-1680
```cpp
1651:     return Builder.CreateShl(Builder.CreateBitCast(Ops[0],Ty), Ops[1],
1652:                              "vshl_n");
1653:   case NEON::BI__builtin_neon_vshll_n_v: {
1654:     llvm::FixedVectorType *SrcTy =
1655:         llvm::FixedVectorType::getTruncatedElementVectorType(VTy);
1656:     Ops[0] = Builder.CreateBitCast(Ops[0], SrcTy);
1657:     if (Usgn)
1658:       Ops[0] = Builder.CreateZExt(Ops[0], VTy);
1659:     else
1660:       Ops[0] = Builder.CreateSExt(Ops[0], VTy);
1661:     Ops[1] = EmitNeonShiftVector(Ops[1], VTy, false);
1662:     return Builder.CreateShl(Ops[0], Ops[1], "vshll_n");
1663:   }
1664:   case NEON::BI__builtin_neon_vshrn_n_v: {
1665:     llvm::FixedVectorType *SrcTy =
1666:         llvm::FixedVectorType::getExtendedElementVectorType(VTy);
1667:     Ops[0] = Builder.CreateBitCast(Ops[0], SrcTy);
1668:     Ops[1] = EmitNeonShiftVector(Ops[1], SrcTy, false);
1669:     if (Usgn)
1670:       Ops[0] = Builder.CreateLShr(Ops[0], Ops[1]);
1671:     else
1672:       Ops[0] = Builder.CreateAShr(Ops[0], Ops[1]);
1673:     return Builder.CreateTrunc(Ops[0], Ty, "vshrn_n");
1674:   }
1675:   case NEON::BI__builtin_neon_vshr_n_v:
1676:   case NEON::BI__builtin_neon_vshrq_n_v:
1677:     return EmitNeonRShiftImm(Ops[0], Ops[1], Ty, Usgn, "vshr_n");
1678:   case NEON::BI__builtin_neon_vst1_v:
1679:   case NEON::BI__builtin_neon_vst1q_v:
1680:   case NEON::BI__builtin_neon_vst2_v:
```
- **EN**: This block defines callable entry points like `getTruncatedElementVectorType`, `getExtendedElementVectorType`, `EmitNeonRShiftImm`; uses control flow (if, case) to specialize target builtin lowering.
- **CN**: 该代码块定义可调用入口，例如 `getTruncatedElementVectorType`, `getExtendedElementVectorType`, `EmitNeonRShiftImm`；通过控制流（if, case）细化 目标内建函数降级 行为。

### Lines 1681-1710
```cpp
1681:   case NEON::BI__builtin_neon_vst2q_v:
1682:   case NEON::BI__builtin_neon_vst3_v:
1683:   case NEON::BI__builtin_neon_vst3q_v:
1684:   case NEON::BI__builtin_neon_vst4_v:
1685:   case NEON::BI__builtin_neon_vst4q_v:
1686:   case NEON::BI__builtin_neon_vst2_lane_v:
1687:   case NEON::BI__builtin_neon_vst2q_lane_v:
1688:   case NEON::BI__builtin_neon_vst3_lane_v:
1689:   case NEON::BI__builtin_neon_vst3q_lane_v:
1690:   case NEON::BI__builtin_neon_vst4_lane_v:
1691:   case NEON::BI__builtin_neon_vst4q_lane_v: {
1692:     llvm::Type *Tys[] = {Int8PtrTy, Ty};
1693:     Ops.push_back(getAlignmentValue32(PtrOp0));
1694:     return EmitNeonCall(CGM.getIntrinsic(Int, Tys), Ops, "");
1695:   }
1696:   case NEON::BI__builtin_neon_vsm3partw1q_u32:
1697:   case NEON::BI__builtin_neon_vsm3partw2q_u32:
1698:   case NEON::BI__builtin_neon_vsm3ss1q_u32:
1699:   case NEON::BI__builtin_neon_vsm4ekeyq_u32:
1700:   case NEON::BI__builtin_neon_vsm4eq_u32: {
1701:     Function *F = CGM.getIntrinsic(Int);
1702:     return EmitNeonCall(F, Ops, "");
1703:   }
1704:   case NEON::BI__builtin_neon_vsm3tt1aq_u32:
1705:   case NEON::BI__builtin_neon_vsm3tt1bq_u32:
1706:   case NEON::BI__builtin_neon_vsm3tt2aq_u32:
1707:   case NEON::BI__builtin_neon_vsm3tt2bq_u32: {
1708:     Function *F = CGM.getIntrinsic(Int);
1709:     Ops[3] = Builder.CreateZExt(Ops[3], Int64Ty);
1710:     return EmitNeonCall(F, Ops, "");
```
- **EN**: This block defines callable entry points like `EmitNeonCall`; uses control flow (case) to specialize target builtin lowering.
- **CN**: 该代码块定义可调用入口，例如 `EmitNeonCall`；通过控制流（case）细化 目标内建函数降级 行为。

### Lines 1711-1740
```cpp
1711:   }
1712:   case NEON::BI__builtin_neon_vst1_x2_v:
1713:   case NEON::BI__builtin_neon_vst1q_x2_v:
1714:   case NEON::BI__builtin_neon_vst1_x3_v:
1715:   case NEON::BI__builtin_neon_vst1q_x3_v:
1716:   case NEON::BI__builtin_neon_vst1_x4_v:
1717:   case NEON::BI__builtin_neon_vst1q_x4_v: {
1718:     // TODO: Currently in AArch32 mode the pointer operand comes first, whereas
1719:     // in AArch64 it comes last. We may want to stick to one or another.
1720:     if (Arch == llvm::Triple::aarch64 || Arch == llvm::Triple::aarch64_be ||
1721:         Arch == llvm::Triple::aarch64_32) {
1722:       llvm::Type *Tys[2] = {VTy, DefaultPtrTy};
1723:       std::rotate(Ops.begin(), Ops.begin() + 1, Ops.end());
1724:       return EmitNeonCall(CGM.getIntrinsic(LLVMIntrinsic, Tys), Ops, "");
1725:     }
1726:     llvm::Type *Tys[2] = {DefaultPtrTy, VTy};
1727:     return EmitNeonCall(CGM.getIntrinsic(LLVMIntrinsic, Tys), Ops, "");
1728:   }
1729:   case NEON::BI__builtin_neon_vsubhn_v: {
1730:     llvm::FixedVectorType *SrcTy =
1731:         llvm::FixedVectorType::getExtendedElementVectorType(VTy);
1732: 
1733:     // %sum = add <4 x i32> %lhs, %rhs
1734:     Ops[0] = Builder.CreateBitCast(Ops[0], SrcTy);
1735:     Ops[1] = Builder.CreateBitCast(Ops[1], SrcTy);
1736:     Ops[0] = Builder.CreateSub(Ops[0], Ops[1], "vsubhn");
1737: 
1738:     // %high = lshr <4 x i32> %sum, <i32 16, i32 16, i32 16, i32 16>
1739:     Constant *ShiftAmt =
1740:         ConstantInt::get(SrcTy, SrcTy->getScalarSizeInBits() / 2);
```
- **EN**: This block defines callable entry points like `rotate`, `EmitNeonCall`, `getExtendedElementVectorType`, `get`; uses control flow (if, case) to specialize target builtin lowering.
- **CN**: 该代码块定义可调用入口，例如 `rotate`, `EmitNeonCall`, `getExtendedElementVectorType`, `get`；通过控制流（if, case）细化 目标内建函数降级 行为。

### Lines 1741-1770
```cpp
1741:     Ops[0] = Builder.CreateLShr(Ops[0], ShiftAmt, "vsubhn");
1742: 
1743:     // %res = trunc <4 x i32> %high to <4 x i16>
1744:     return Builder.CreateTrunc(Ops[0], VTy, "vsubhn");
1745:   }
1746:   case NEON::BI__builtin_neon_vtrn_v:
1747:   case NEON::BI__builtin_neon_vtrnq_v: {
1748:     Ops[1] = Builder.CreateBitCast(Ops[1], Ty);
1749:     Ops[2] = Builder.CreateBitCast(Ops[2], Ty);
1750:     Value *SV = nullptr;
1751: 
1752:     for (unsigned vi = 0; vi != 2; ++vi) {
1753:       SmallVector<int, 16> Indices;
1754:       for (unsigned i = 0, e = VTy->getNumElements(); i != e; i += 2) {
1755:         Indices.push_back(i+vi);
1756:         Indices.push_back(i+e+vi);
1757:       }
1758:       Value *Addr = Builder.CreateConstInBoundsGEP1_32(Ty, Ops[0], vi);
1759:       SV = Builder.CreateShuffleVector(Ops[1], Ops[2], Indices, "vtrn");
1760:       SV = Builder.CreateDefaultAlignedStore(SV, Addr);
1761:     }
1762:     return SV;
1763:   }
1764:   case NEON::BI__builtin_neon_vtst_v:
1765:   case NEON::BI__builtin_neon_vtstq_v: {
1766:     Ops[0] = Builder.CreateBitCast(Ops[0], Ty);
1767:     Ops[1] = Builder.CreateBitCast(Ops[1], Ty);
1768:     Ops[0] = Builder.CreateAnd(Ops[0], Ops[1]);
1769:     Ops[0] = Builder.CreateICmp(ICmpInst::ICMP_NE, Ops[0],
1770:                                 ConstantAggregateZero::get(Ty));
```
- **EN**: This block defines callable entry points like `get`; uses control flow (for, case) to specialize target builtin lowering.
- **CN**: 该代码块定义可调用入口，例如 `get`；通过控制流（for, case）细化 目标内建函数降级 行为。

### Lines 1771-1800
```cpp
1771:     return Builder.CreateSExt(Ops[0], Ty, "vtst");
1772:   }
1773:   case NEON::BI__builtin_neon_vuzp_v:
1774:   case NEON::BI__builtin_neon_vuzpq_v: {
1775:     Ops[1] = Builder.CreateBitCast(Ops[1], Ty);
1776:     Ops[2] = Builder.CreateBitCast(Ops[2], Ty);
1777:     Value *SV = nullptr;
1778: 
1779:     for (unsigned vi = 0; vi != 2; ++vi) {
1780:       SmallVector<int, 16> Indices;
1781:       for (unsigned i = 0, e = VTy->getNumElements(); i != e; ++i)
1782:         Indices.push_back(2*i+vi);
1783: 
1784:       Value *Addr = Builder.CreateConstInBoundsGEP1_32(Ty, Ops[0], vi);
1785:       SV = Builder.CreateShuffleVector(Ops[1], Ops[2], Indices, "vuzp");
1786:       SV = Builder.CreateDefaultAlignedStore(SV, Addr);
1787:     }
1788:     return SV;
1789:   }
1790:   case NEON::BI__builtin_neon_vxarq_u64: {
1791:     Function *F = CGM.getIntrinsic(Int);
1792:     Ops[2] = Builder.CreateZExt(Ops[2], Int64Ty);
1793:     return EmitNeonCall(F, Ops, "");
1794:   }
1795:   case NEON::BI__builtin_neon_vzip_v:
1796:   case NEON::BI__builtin_neon_vzipq_v: {
1797:     Ops[1] = Builder.CreateBitCast(Ops[1], Ty);
1798:     Ops[2] = Builder.CreateBitCast(Ops[2], Ty);
1799:     Value *SV = nullptr;
1800: 
```
- **EN**: This block defines callable entry points like `EmitNeonCall`; uses control flow (for, case) to specialize target builtin lowering.
- **CN**: 该代码块定义可调用入口，例如 `EmitNeonCall`；通过控制流（for, case）细化 目标内建函数降级 行为。

### Lines 1801-1830
```cpp
1801:     for (unsigned vi = 0; vi != 2; ++vi) {
1802:       SmallVector<int, 16> Indices;
1803:       for (unsigned i = 0, e = VTy->getNumElements(); i != e; i += 2) {
1804:         Indices.push_back((i + vi*e) >> 1);
1805:         Indices.push_back(((i + vi*e) >> 1)+e);
1806:       }
1807:       Value *Addr = Builder.CreateConstInBoundsGEP1_32(Ty, Ops[0], vi);
1808:       SV = Builder.CreateShuffleVector(Ops[1], Ops[2], Indices, "vzip");
1809:       SV = Builder.CreateDefaultAlignedStore(SV, Addr);
1810:     }
1811:     return SV;
1812:   }
1813:   case NEON::BI__builtin_neon_vdot_s32:
1814:   case NEON::BI__builtin_neon_vdot_u32:
1815:   case NEON::BI__builtin_neon_vdotq_s32:
1816:   case NEON::BI__builtin_neon_vdotq_u32: {
1817:     auto *InputTy =
1818:         llvm::FixedVectorType::get(Int8Ty, Ty->getPrimitiveSizeInBits() / 8);
1819:     llvm::Type *Tys[2] = { Ty, InputTy };
1820:     return EmitNeonCall(CGM.getIntrinsic(Int, Tys), Ops, "vdot");
1821:   }
1822:   case NEON::BI__builtin_neon_vfmlal_low_f16:
1823:   case NEON::BI__builtin_neon_vfmlalq_low_f16: {
1824:     auto *InputTy =
1825:         llvm::FixedVectorType::get(HalfTy, Ty->getPrimitiveSizeInBits() / 16);
1826:     llvm::Type *Tys[2] = { Ty, InputTy };
1827:     return EmitNeonCall(CGM.getIntrinsic(Int, Tys), Ops, "vfmlal_low");
1828:   }
1829:   case NEON::BI__builtin_neon_vfmlsl_low_f16:
1830:   case NEON::BI__builtin_neon_vfmlslq_low_f16: {
```
- **EN**: This block defines callable entry points like `get`, `EmitNeonCall`; uses control flow (for, case) to specialize target builtin lowering.
- **CN**: 该代码块定义可调用入口，例如 `get`, `EmitNeonCall`；通过控制流（for, case）细化 目标内建函数降级 行为。

### Lines 1831-1860
```cpp
1831:     auto *InputTy =
1832:         llvm::FixedVectorType::get(HalfTy, Ty->getPrimitiveSizeInBits() / 16);
1833:     llvm::Type *Tys[2] = { Ty, InputTy };
1834:     return EmitNeonCall(CGM.getIntrinsic(Int, Tys), Ops, "vfmlsl_low");
1835:   }
1836:   case NEON::BI__builtin_neon_vfmlal_high_f16:
1837:   case NEON::BI__builtin_neon_vfmlalq_high_f16: {
1838:     auto *InputTy =
1839:         llvm::FixedVectorType::get(HalfTy, Ty->getPrimitiveSizeInBits() / 16);
1840:     llvm::Type *Tys[2] = { Ty, InputTy };
1841:     return EmitNeonCall(CGM.getIntrinsic(Int, Tys), Ops, "vfmlal_high");
1842:   }
1843:   case NEON::BI__builtin_neon_vfmlsl_high_f16:
1844:   case NEON::BI__builtin_neon_vfmlslq_high_f16: {
1845:     auto *InputTy =
1846:         llvm::FixedVectorType::get(HalfTy, Ty->getPrimitiveSizeInBits() / 16);
1847:     llvm::Type *Tys[2] = { Ty, InputTy };
1848:     return EmitNeonCall(CGM.getIntrinsic(Int, Tys), Ops, "vfmlsl_high");
1849:   }
1850:   case NEON::BI__builtin_neon_vmmlaq_s32:
1851:   case NEON::BI__builtin_neon_vmmlaq_u32: {
1852:     auto *InputTy =
1853:         llvm::FixedVectorType::get(Int8Ty, Ty->getPrimitiveSizeInBits() / 8);
1854:     llvm::Type *Tys[2] = { Ty, InputTy };
1855:     return EmitNeonCall(CGM.getIntrinsic(LLVMIntrinsic, Tys), Ops, "vmmla");
1856:   }
1857:   case NEON::BI__builtin_neon_vmmlaq_f16_f16:
1858:   case NEON::BI__builtin_neon_vmmlaq_f32_f16: {
1859:     auto *InputTy =
1860:         llvm::FixedVectorType::get(HalfTy, Ty->getPrimitiveSizeInBits() / 16);
```
- **EN**: This block defines callable entry points like `get`, `EmitNeonCall`; uses control flow (case) to specialize target builtin lowering.
- **CN**: 该代码块定义可调用入口，例如 `get`, `EmitNeonCall`；通过控制流（case）细化 目标内建函数降级 行为。

### Lines 1861-1890
```cpp
1861:     llvm::Type *Tys[2] = {Ty, InputTy};
1862:     return EmitNeonCall(CGM.getIntrinsic(Int, Tys), Ops, "fmmla");
1863:   }
1864:   case NEON::BI__builtin_neon_vusmmlaq_s32: {
1865:     auto *InputTy =
1866:         llvm::FixedVectorType::get(Int8Ty, Ty->getPrimitiveSizeInBits() / 8);
1867:     llvm::Type *Tys[2] = { Ty, InputTy };
1868:     return EmitNeonCall(CGM.getIntrinsic(Int, Tys), Ops, "vusmmla");
1869:   }
1870:   case NEON::BI__builtin_neon_vusdot_s32:
1871:   case NEON::BI__builtin_neon_vusdotq_s32: {
1872:     auto *InputTy =
1873:         llvm::FixedVectorType::get(Int8Ty, Ty->getPrimitiveSizeInBits() / 8);
1874:     llvm::Type *Tys[2] = { Ty, InputTy };
1875:     return EmitNeonCall(CGM.getIntrinsic(Int, Tys), Ops, "vusdot");
1876:   }
1877:   case NEON::BI__builtin_neon_vbfdot_f32:
1878:   case NEON::BI__builtin_neon_vbfdotq_f32: {
1879:     llvm::Type *InputTy =
1880:         llvm::FixedVectorType::get(BFloatTy, Ty->getPrimitiveSizeInBits() / 16);
1881:     llvm::Type *Tys[2] = { Ty, InputTy };
1882:     return EmitNeonCall(CGM.getIntrinsic(Int, Tys), Ops, "vbfdot");
1883:   }
1884:   case NEON::BI__builtin_neon___a32_vcvt_bf16_f32: {
1885:     llvm::Type *Tys[1] = { Ty };
1886:     Function *F = CGM.getIntrinsic(Int, Tys);
1887:     return EmitNeonCall(F, Ops, "vcvtfp2bf");
1888:   }
1889: 
1890:   }
```
- **EN**: This block defines callable entry points like `EmitNeonCall`, `get`; uses control flow (case) to specialize target builtin lowering.
- **CN**: 该代码块定义可调用入口，例如 `EmitNeonCall`, `get`；通过控制流（case）细化 目标内建函数降级 行为。

### Lines 1891-1920
```cpp
1891: 
1892:   assert(Int && "Expected valid intrinsic number");
1893: 
1894:   // Determine the type(s) of this overloaded AArch64 intrinsic.
1895:   Function *F = LookupNeonLLVMIntrinsic(Int, Modifier, Ty, E);
1896: 
1897:   Value *Result = EmitNeonCall(F, Ops, NameHint);
1898:   llvm::Type *ResultType = ConvertType(E->getType());
1899:   // AArch64 intrinsic one-element vector type cast to
1900:   // scalar type expected by the builtin
1901:   return Builder.CreateBitCast(Result, ResultType, NameHint);
1902: }
1903: 
1904: Value *
1905: CodeGenFunction::EmitAArch64CompareBuiltinExpr(Value *Op, llvm::Type *Ty,
1906:                                                const CmpInst::Predicate Pred,
1907:                                                const Twine &Name) {
1908: 
1909:   if (isa<FixedVectorType>(Ty)) {
1910:     // Vector types are cast to i8 vectors. Recover original type.
1911:     Op = Builder.CreateBitCast(Op, Ty);
1912:   }
1913: 
1914:   Constant *zero = Constant::getNullValue(Op->getType());
1915: 
1916:   if (CmpInst::isFPPredicate(Pred)) {
1917:     if (Pred == CmpInst::FCMP_OEQ)
1918:       Op = Builder.CreateFCmp(Pred, Op, zero);
1919:     else
1920:       Op = Builder.CreateFCmpS(Pred, Op, zero);
```
- **EN**: This block defines callable entry points like `EmitAArch64CompareBuiltinExpr`; uses control flow (if) to specialize target builtin lowering; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块定义可调用入口，例如 `EmitAArch64CompareBuiltinExpr`；通过控制流（if）细化 目标内建函数降级 行为；使用断言或不可达标记保护关键不变量。

### Lines 1921-1950
```cpp
1921:   } else {
1922:     Op = Builder.CreateICmp(Pred, Op, zero);
1923:   }
1924: 
1925:   llvm::Type *ResTy = Ty;
1926:   if (auto *VTy = dyn_cast<FixedVectorType>(Ty))
1927:     ResTy = FixedVectorType::get(
1928:         IntegerType::get(getLLVMContext(), VTy->getScalarSizeInBits()),
1929:         VTy->getNumElements());
1930: 
1931:   return Builder.CreateSExt(Op, ResTy, Name);
1932: }
1933: 
1934: static Value *packTBLDVectorList(CodeGenFunction &CGF, ArrayRef<Value *> Ops,
1935:                                  Value *ExtOp, Value *IndexOp,
1936:                                  llvm::Type *ResTy, unsigned IntID,
1937:                                  const char *Name) {
1938:   SmallVector<Value *, 2> TblOps;
1939:   if (ExtOp)
1940:     TblOps.push_back(ExtOp);
1941: 
1942:   // Build a vector containing sequential number like (0, 1, 2, ..., 15)
1943:   SmallVector<int, 16> Indices;
1944:   auto *TblTy = cast<llvm::FixedVectorType>(Ops[0]->getType());
1945:   for (unsigned i = 0, e = TblTy->getNumElements(); i != e; ++i) {
1946:     Indices.push_back(2*i);
1947:     Indices.push_back(2*i+1);
1948:   }
1949: 
1950:   int PairPos = 0, End = Ops.size() - 1;
```
- **EN**: This block uses control flow (if, for) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（if, for）细化 目标内建函数降级 行为。

### Lines 1951-1980
```cpp
1951:   while (PairPos < End) {
1952:     TblOps.push_back(CGF.Builder.CreateShuffleVector(Ops[PairPos],
1953:                                                      Ops[PairPos+1], Indices,
1954:                                                      Name));
1955:     PairPos += 2;
1956:   }
1957: 
1958:   // If there's an odd number of 64-bit lookup table, fill the high 64-bit
1959:   // of the 128-bit lookup table with zero.
1960:   if (PairPos == End) {
1961:     Value *ZeroTbl = ConstantAggregateZero::get(TblTy);
1962:     TblOps.push_back(CGF.Builder.CreateShuffleVector(Ops[PairPos],
1963:                                                      ZeroTbl, Indices, Name));
1964:   }
1965: 
1966:   Function *TblF;
1967:   TblOps.push_back(IndexOp);
1968:   TblF = CGF.CGM.getIntrinsic(IntID, ResTy);
1969: 
1970:   return CGF.EmitNeonCall(TblF, TblOps, Name);
1971: }
1972: 
1973: Value *CodeGenFunction::GetValueForARMHint(unsigned BuiltinID) {
1974:   unsigned Value;
1975:   switch (BuiltinID) {
1976:   default:
1977:     return nullptr;
1978:   case clang::ARM::BI__builtin_arm_nop:
1979:     Value = 0;
1980:     break;
```
- **EN**: This block uses control flow (if, switch, while, case) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（if, switch, while, case）细化 目标内建函数降级 行为。

### Lines 1981-2010
```cpp
1981:   case clang::ARM::BI__builtin_arm_yield:
1982:   case clang::ARM::BI__yield:
1983:     Value = 1;
1984:     break;
1985:   case clang::ARM::BI__builtin_arm_wfe:
1986:   case clang::ARM::BI__wfe:
1987:     Value = 2;
1988:     break;
1989:   case clang::ARM::BI__builtin_arm_wfi:
1990:   case clang::ARM::BI__wfi:
1991:     Value = 3;
1992:     break;
1993:   case clang::ARM::BI__builtin_arm_sev:
1994:   case clang::ARM::BI__sev:
1995:     Value = 4;
1996:     break;
1997:   case clang::ARM::BI__builtin_arm_sevl:
1998:   case clang::ARM::BI__sevl:
1999:     Value = 5;
2000:     break;
2001:   }
2002: 
2003:   return Builder.CreateCall(CGM.getIntrinsic(Intrinsic::arm_hint),
2004:                             llvm::ConstantInt::get(Int32Ty, Value));
2005: }
2006: 
2007: enum SpecialRegisterAccessKind {
2008:   NormalRead,
2009:   VolatileRead,
2010:   Write,
```
- **EN**: This block introduces declarations such as `SpecialRegisterAccessKind`; defines callable entry points like `get`; uses control flow (case) to specialize target builtin lowering.
- **CN**: 该代码块给出诸如 `SpecialRegisterAccessKind` 的声明；定义可调用入口，例如 `get`；通过控制流（case）细化 目标内建函数降级 行为。

### Lines 2011-2040
```cpp
2011: };
2012: 
2013: // Generates the IR for the read/write special register builtin,
2014: // ValueType is the type of the value that is to be written or read,
2015: // RegisterType is the type of the register being written to or read from.
2016: static Value *EmitSpecialRegisterBuiltin(CodeGenFunction &CGF,
2017:                                          const CallExpr *E,
2018:                                          llvm::Type *RegisterType,
2019:                                          llvm::Type *ValueType,
2020:                                          SpecialRegisterAccessKind AccessKind,
2021:                                          StringRef SysReg = "") {
2022:   // write and register intrinsics only support 32, 64 and 128 bit operations.
2023:   assert((RegisterType->isIntegerTy(32) || RegisterType->isIntegerTy(64) ||
2024:           RegisterType->isIntegerTy(128)) &&
2025:          "Unsupported size for register.");
2026: 
2027:   CodeGen::CGBuilderTy &Builder = CGF.Builder;
2028:   CodeGen::CodeGenModule &CGM = CGF.CGM;
2029:   LLVMContext &Context = CGM.getLLVMContext();
2030: 
2031:   if (SysReg.empty()) {
2032:     const Expr *SysRegStrExpr = E->getArg(0)->IgnoreParenCasts();
2033:     SysReg = cast<clang::StringLiteral>(SysRegStrExpr)->getString();
2034:   }
2035: 
2036:   llvm::Metadata *Ops[] = { llvm::MDString::get(Context, SysReg) };
2037:   llvm::MDNode *RegName = llvm::MDNode::get(Context, Ops);
2038:   llvm::Value *Metadata = llvm::MetadataAsValue::get(Context, RegName);
2039: 
2040:   llvm::Type *Types[] = { RegisterType };
```
- **EN**: This block uses control flow (if, for) to specialize target builtin lowering; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块通过控制流（if, for）细化 目标内建函数降级 行为；使用断言或不可达标记保护关键不变量。

### Lines 2041-2070
```cpp
2041: 
2042:   bool MixedTypes = RegisterType->isIntegerTy(64) && ValueType->isIntegerTy(32);
2043:   assert(!(RegisterType->isIntegerTy(32) && ValueType->isIntegerTy(64))
2044:             && "Can't fit 64-bit value in 32-bit register");
2045: 
2046:   if (AccessKind != Write) {
2047:     assert(AccessKind == NormalRead || AccessKind == VolatileRead);
2048:     llvm::Function *F = CGM.getIntrinsic(
2049:         AccessKind == VolatileRead ? Intrinsic::read_volatile_register
2050:                                    : Intrinsic::read_register,
2051:         Types);
2052:     llvm::Value *Call = Builder.CreateCall(F, Metadata);
2053: 
2054:     if (MixedTypes)
2055:       // Read into 64 bit register and then truncate result to 32 bit.
2056:       return Builder.CreateTrunc(Call, ValueType);
2057: 
2058:     if (ValueType->isPointerTy())
2059:       // Have i32/i64 result (Call) but want to return a VoidPtrTy (i8*).
2060:       return Builder.CreateIntToPtr(Call, ValueType);
2061: 
2062:     return Call;
2063:   }
2064: 
2065:   llvm::Function *F = CGM.getIntrinsic(Intrinsic::write_register, Types);
2066:   llvm::Value *ArgValue = CGF.EmitScalarExpr(E->getArg(1));
2067:   if (MixedTypes) {
2068:     // Extend 32 bit write value to 64 bit to pass to write.
2069:     ArgValue = Builder.CreateZExt(ArgValue, RegisterType);
2070:     return Builder.CreateCall(F, { Metadata, ArgValue });
```
- **EN**: This block uses control flow (if) to specialize target builtin lowering; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块通过控制流（if）细化 目标内建函数降级 行为；使用断言或不可达标记保护关键不变量。

### Lines 2071-2100
```cpp
2071:   }
2072: 
2073:   if (ValueType->isPointerTy()) {
2074:     // Have VoidPtrTy ArgValue but want to return an i32/i64.
2075:     ArgValue = Builder.CreatePtrToInt(ArgValue, RegisterType);
2076:     return Builder.CreateCall(F, { Metadata, ArgValue });
2077:   }
2078: 
2079:   return Builder.CreateCall(F, { Metadata, ArgValue });
2080: }
2081: 
2082: static Value *EmitRangePrefetchBuiltin(CodeGenFunction &CGF, unsigned BuiltinID,
2083:                                        const CallExpr *E) {
2084:   CodeGen::CGBuilderTy &Builder = CGF.Builder;
2085:   CodeGen::CodeGenModule &CGM = CGF.CGM;
2086:   SmallVector<llvm::Value *, 4> Ops;
2087: 
2088:   auto getIntArg = [&](unsigned ArgNo) {
2089:     Expr::EvalResult Result;
2090:     if (!E->getArg(ArgNo)->EvaluateAsInt(Result, CGM.getContext()))
2091:       llvm_unreachable("Expected constant argument to range prefetch.");
2092:     return Result.Val.getInt().getExtValue();
2093:   };
2094: 
2095:   Ops.push_back(CGF.EmitScalarExpr(E->getArg(0))); /*Addr*/
2096:   Ops.push_back(CGF.EmitScalarExpr(E->getArg(1))); /*Access Kind*/
2097:   Ops.push_back(CGF.EmitScalarExpr(E->getArg(2))); /*Policy*/
2098: 
2099:   if (BuiltinID == clang::AArch64::BI__builtin_arm_range_prefetch_x) {
2100:     auto Length = getIntArg(3);
```
- **EN**: This block uses control flow (if) to specialize target builtin lowering; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块通过控制流（if）细化 目标内建函数降级 行为；使用断言或不可达标记保护关键不变量。

### Lines 2101-2130
```cpp
2101:     auto Count = getIntArg(4) - 1;
2102:     auto Stride = getIntArg(5);
2103:     auto Distance = getIntArg(6);
2104: 
2105:     // Map ReuseDistance given in bytes to four bits representing decreasing
2106:     // powers of two in the range 512MiB (0b0001) to 32KiB (0b1111). Values
2107:     // are rounded up to the nearest power of 2, starting at 32KiB. Any value
2108:     // over the maximum is represented by 0 (distance not known).
2109:     if (Distance > 0) {
2110:       Distance = llvm::Log2_32_Ceil(Distance);
2111:       if (Distance < 15)
2112:         Distance = 15;
2113:       else if (Distance > 29)
2114:         Distance = 0;
2115:       else
2116:         Distance = 30 - Distance;
2117:     }
2118: 
2119:     uint64_t Mask22 = (1ULL << 22) - 1;
2120:     uint64_t Mask16 = (1ULL << 16) - 1;
2121:     uint64_t Metadata = (Distance << 60) | ((Stride & Mask22) << 38) |
2122:                         ((Count & Mask16) << 22) | (Length & Mask22);
2123: 
2124:     Ops.push_back(llvm::ConstantInt::get(Builder.getInt64Ty(), Metadata));
2125:   } else
2126:     Ops.push_back(CGF.EmitScalarExpr(E->getArg(3)));
2127: 
2128:   return Builder.CreateCall(CGM.getIntrinsic(Intrinsic::aarch64_range_prefetch),
2129:                             Ops);
2130: }
```
- **EN**: This block uses control flow (if) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（if）细化 目标内建函数降级 行为。

### Lines 2131-2160
```cpp
2131: 
2132: /// Return true if BuiltinID is an overloaded Neon intrinsic with an extra
2133: /// argument that specifies the vector type. The additional argument is meant
2134: /// for Sema checking (see `CheckNeonBuiltinFunctionCall`) and this function
2135: /// should be kept consistent with the logic in Sema.
2136: /// TODO: Make this return false for SISD builtins.
2137: static bool HasExtraNeonArgument(unsigned BuiltinID) {
2138:   // Required by the headers included below, but not in this particular
2139:   // function.
2140:   [[maybe_unused]] int PtrArgNum = -1;
2141:   [[maybe_unused]] bool HasConstPtr = false;
2142: 
2143:   // The mask encodes the type. We don't care about the actual value. Instead,
2144:   // we just check whether its been set.
2145:   uint64_t mask = 0;
2146:   switch (BuiltinID) {
2147: #define GET_NEON_OVERLOAD_CHECK
2148: #include "clang/Basic/arm_fp16.inc"
2149: #include "clang/Basic/arm_neon.inc"
2150: #undef GET_NEON_OVERLOAD_CHECK
2151:   // Non-neon builtins for controling VFP that take extra argument for
2152:   // discriminating the type.
2153:   case ARM::BI__builtin_arm_vcvtr_f:
2154:   case ARM::BI__builtin_arm_vcvtr_d:
2155:     mask = 1;
2156:   }
2157: 
2158:   if (mask)
2159:     return true;
2160: 
```
- **EN**: This block imports Clang headers `clang/Basic/arm_fp16.inc`, `clang/Basic/arm_neon.inc`; defines callable entry points like `HasExtraNeonArgument`; uses control flow (if, switch, case) to specialize target builtin lowering; contains preprocessor structure that shapes compilation of this unit.
- **CN**: 该代码块引入 Clang 头文件 `clang/Basic/arm_fp16.inc`, `clang/Basic/arm_neon.inc`；定义可调用入口，例如 `HasExtraNeonArgument`；通过控制流（if, switch, case）细化 目标内建函数降级 行为；包含影响本编译单元构建方式的预处理结构。

### Lines 2161-2190
```cpp
2161:   return false;
2162: }
2163: 
2164: Value *CodeGenFunction::EmitARMBuiltinExpr(unsigned BuiltinID,
2165:                                            const CallExpr *E,
2166:                                            ReturnValueSlot ReturnValue,
2167:                                            llvm::Triple::ArchType Arch) {
2168:   if (auto Hint = GetValueForARMHint(BuiltinID))
2169:     return Hint;
2170: 
2171:   if (BuiltinID == clang::ARM::BI__emit) {
2172:     bool IsThumb = getTarget().getTriple().getArch() == llvm::Triple::thumb;
2173:     llvm::FunctionType *FTy =
2174:         llvm::FunctionType::get(VoidTy, /*Variadic=*/false);
2175: 
2176:     Expr::EvalResult Result;
2177:     if (!E->getArg(0)->EvaluateAsInt(Result, CGM.getContext()))
2178:       llvm_unreachable("Sema will ensure that the parameter is constant");
2179: 
2180:     llvm::APSInt Value = Result.Val.getInt();
2181:     uint64_t ZExtValue = Value.zextOrTrunc(IsThumb ? 16 : 32).getZExtValue();
2182: 
2183:     llvm::InlineAsm *Emit =
2184:         IsThumb ? InlineAsm::get(FTy, ".inst.n 0x" + utohexstr(ZExtValue), "",
2185:                                  /*hasSideEffects=*/true)
2186:                 : InlineAsm::get(FTy, ".inst 0x" + utohexstr(ZExtValue), "",
2187:                                  /*hasSideEffects=*/true);
2188: 
2189:     return Builder.CreateCall(Emit);
2190:   }
```
- **EN**: This block defines callable entry points like `get`; uses control flow (if) to specialize target builtin lowering; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块定义可调用入口，例如 `get`；通过控制流（if）细化 目标内建函数降级 行为；使用断言或不可达标记保护关键不变量。

### Lines 2191-2220
```cpp
2191: 
2192:   if (BuiltinID == clang::ARM::BI__builtin_arm_dbg) {
2193:     Value *Option = EmitScalarExpr(E->getArg(0));
2194:     return Builder.CreateCall(CGM.getIntrinsic(Intrinsic::arm_dbg), Option);
2195:   }
2196: 
2197:   if (BuiltinID == clang::ARM::BI__builtin_arm_prefetch) {
2198:     Value *Address = EmitScalarExpr(E->getArg(0));
2199:     Value *RW      = EmitScalarExpr(E->getArg(1));
2200:     Value *IsData  = EmitScalarExpr(E->getArg(2));
2201: 
2202:     // Locality is not supported on ARM target
2203:     Value *Locality = llvm::ConstantInt::get(Int32Ty, 3);
2204: 
2205:     Function *F = CGM.getIntrinsic(Intrinsic::prefetch, Address->getType());
2206:     return Builder.CreateCall(F, {Address, RW, Locality, IsData});
2207:   }
2208: 
2209:   if (BuiltinID == clang::ARM::BI__builtin_arm_rbit) {
2210:     llvm::Value *Arg = EmitScalarExpr(E->getArg(0));
2211:     return Builder.CreateCall(
2212:         CGM.getIntrinsic(Intrinsic::bitreverse, Arg->getType()), Arg, "rbit");
2213:   }
2214: 
2215:   if (BuiltinID == clang::ARM::BI__builtin_arm_clz ||
2216:       BuiltinID == clang::ARM::BI__builtin_arm_clz64) {
2217:     llvm::Value *Arg = EmitScalarExpr(E->getArg(0));
2218:     Function *F = CGM.getIntrinsic(Intrinsic::ctlz, Arg->getType());
2219:     Value *Res = Builder.CreateCall(F, {Arg, Builder.getInt1(false)});
2220:     if (BuiltinID == clang::ARM::BI__builtin_arm_clz64)
```
- **EN**: This block uses control flow (if) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（if）细化 目标内建函数降级 行为。

### Lines 2221-2250
```cpp
2221:       Res = Builder.CreateTrunc(Res, Builder.getInt32Ty());
2222:     return Res;
2223:   }
2224: 
2225: 
2226:   if (BuiltinID == clang::ARM::BI__builtin_arm_cls) {
2227:     llvm::Value *Arg = EmitScalarExpr(E->getArg(0));
2228:     return Builder.CreateCall(CGM.getIntrinsic(Intrinsic::arm_cls), Arg, "cls");
2229:   }
2230:   if (BuiltinID == clang::ARM::BI__builtin_arm_cls64) {
2231:     llvm::Value *Arg = EmitScalarExpr(E->getArg(0));
2232:     return Builder.CreateCall(CGM.getIntrinsic(Intrinsic::arm_cls64), Arg,
2233:                               "cls");
2234:   }
2235: 
2236:   if (BuiltinID == clang::ARM::BI__clear_cache) {
2237:     assert(E->getNumArgs() == 2 && "__clear_cache takes 2 arguments");
2238:     const FunctionDecl *FD = E->getDirectCallee();
2239:     Value *Ops[2];
2240:     for (unsigned i = 0; i < 2; i++)
2241:       Ops[i] = EmitScalarExpr(E->getArg(i));
2242:     llvm::Type *Ty = CGM.getTypes().ConvertType(FD->getType());
2243:     llvm::FunctionType *FTy = cast<llvm::FunctionType>(Ty);
2244:     StringRef Name = FD->getName();
2245:     return EmitNounwindRuntimeCall(CGM.CreateRuntimeFunction(FTy, Name), Ops);
2246:   }
2247: 
2248:   if (BuiltinID == clang::ARM::BI__builtin_arm_mcrr ||
2249:       BuiltinID == clang::ARM::BI__builtin_arm_mcrr2) {
2250:     Function *F;
```
- **EN**: This block defines callable entry points like `EmitNounwindRuntimeCall`; uses control flow (if, for) to specialize target builtin lowering; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块定义可调用入口，例如 `EmitNounwindRuntimeCall`；通过控制流（if, for）细化 目标内建函数降级 行为；使用断言或不可达标记保护关键不变量。

### Lines 2251-2280
```cpp
2251: 
2252:     switch (BuiltinID) {
2253:     default: llvm_unreachable("unexpected builtin");
2254:     case clang::ARM::BI__builtin_arm_mcrr:
2255:       F = CGM.getIntrinsic(Intrinsic::arm_mcrr);
2256:       break;
2257:     case clang::ARM::BI__builtin_arm_mcrr2:
2258:       F = CGM.getIntrinsic(Intrinsic::arm_mcrr2);
2259:       break;
2260:     }
2261: 
2262:     // MCRR{2} instruction has 5 operands but
2263:     // the intrinsic has 4 because Rt and Rt2
2264:     // are represented as a single unsigned 64
2265:     // bit integer in the intrinsic definition
2266:     // but internally it's represented as 2 32
2267:     // bit integers.
2268: 
2269:     Value *Coproc = EmitScalarExpr(E->getArg(0));
2270:     Value *Opc1 = EmitScalarExpr(E->getArg(1));
2271:     Value *RtAndRt2 = EmitScalarExpr(E->getArg(2));
2272:     Value *CRm = EmitScalarExpr(E->getArg(3));
2273: 
2274:     Value *C1 = llvm::ConstantInt::get(Int64Ty, 32);
2275:     Value *Rt = Builder.CreateTruncOrBitCast(RtAndRt2, Int32Ty);
2276:     Value *Rt2 = Builder.CreateLShr(RtAndRt2, C1);
2277:     Rt2 = Builder.CreateTruncOrBitCast(Rt2, Int32Ty);
2278: 
2279:     return Builder.CreateCall(F, {Coproc, Opc1, Rt, Rt2, CRm});
2280:   }
```
- **EN**: This block uses control flow (switch, case) to specialize target builtin lowering; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块通过控制流（switch, case）细化 目标内建函数降级 行为；使用断言或不可达标记保护关键不变量。

### Lines 2281-2310
```cpp
2281: 
2282:   if (BuiltinID == clang::ARM::BI__builtin_arm_mrrc ||
2283:       BuiltinID == clang::ARM::BI__builtin_arm_mrrc2) {
2284:     Function *F;
2285: 
2286:     switch (BuiltinID) {
2287:     default: llvm_unreachable("unexpected builtin");
2288:     case clang::ARM::BI__builtin_arm_mrrc:
2289:       F = CGM.getIntrinsic(Intrinsic::arm_mrrc);
2290:       break;
2291:     case clang::ARM::BI__builtin_arm_mrrc2:
2292:       F = CGM.getIntrinsic(Intrinsic::arm_mrrc2);
2293:       break;
2294:     }
2295: 
2296:     Value *Coproc = EmitScalarExpr(E->getArg(0));
2297:     Value *Opc1 = EmitScalarExpr(E->getArg(1));
2298:     Value *CRm  = EmitScalarExpr(E->getArg(2));
2299:     Value *RtAndRt2 = Builder.CreateCall(F, {Coproc, Opc1, CRm});
2300: 
2301:     // Returns an unsigned 64 bit integer, represented
2302:     // as two 32 bit integers.
2303: 
2304:     Value *Rt = Builder.CreateExtractValue(RtAndRt2, 1);
2305:     Value *Rt1 = Builder.CreateExtractValue(RtAndRt2, 0);
2306:     Rt = Builder.CreateZExt(Rt, Int64Ty);
2307:     Rt1 = Builder.CreateZExt(Rt1, Int64Ty);
2308: 
2309:     Value *ShiftCast = llvm::ConstantInt::get(Int64Ty, 32);
2310:     RtAndRt2 = Builder.CreateShl(Rt, ShiftCast, "shl", true);
```
- **EN**: This block uses control flow (if, switch, case) to specialize target builtin lowering; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块通过控制流（if, switch, case）细化 目标内建函数降级 行为；使用断言或不可达标记保护关键不变量。

### Lines 2311-2340
```cpp
2311:     RtAndRt2 = Builder.CreateOr(RtAndRt2, Rt1);
2312: 
2313:     return Builder.CreateBitCast(RtAndRt2, ConvertType(E->getType()));
2314:   }
2315: 
2316:   if (BuiltinID == clang::ARM::BI__builtin_arm_ldrexd ||
2317:       ((BuiltinID == clang::ARM::BI__builtin_arm_ldrex ||
2318:         BuiltinID == clang::ARM::BI__builtin_arm_ldaex) &&
2319:        getContext().getTypeSize(E->getType()) == 64) ||
2320:       BuiltinID == clang::ARM::BI__ldrexd) {
2321:     Function *F;
2322: 
2323:     switch (BuiltinID) {
2324:     default: llvm_unreachable("unexpected builtin");
2325:     case clang::ARM::BI__builtin_arm_ldaex:
2326:       F = CGM.getIntrinsic(Intrinsic::arm_ldaexd);
2327:       break;
2328:     case clang::ARM::BI__builtin_arm_ldrexd:
2329:     case clang::ARM::BI__builtin_arm_ldrex:
2330:     case clang::ARM::BI__ldrexd:
2331:       F = CGM.getIntrinsic(Intrinsic::arm_ldrexd);
2332:       break;
2333:     }
2334: 
2335:     Value *LdPtr = EmitScalarExpr(E->getArg(0));
2336:     Value *Val = Builder.CreateCall(F, LdPtr, "ldrexd");
2337: 
2338:     Value *Val0 = Builder.CreateExtractValue(Val, 1);
2339:     Value *Val1 = Builder.CreateExtractValue(Val, 0);
2340:     Val0 = Builder.CreateZExt(Val0, Int64Ty);
```
- **EN**: This block uses control flow (if, switch, case) to specialize target builtin lowering; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块通过控制流（if, switch, case）细化 目标内建函数降级 行为；使用断言或不可达标记保护关键不变量。

### Lines 2341-2370
```cpp
2341:     Val1 = Builder.CreateZExt(Val1, Int64Ty);
2342: 
2343:     Value *ShiftCst = llvm::ConstantInt::get(Int64Ty, 32);
2344:     Val = Builder.CreateShl(Val0, ShiftCst, "shl", true /* nuw */);
2345:     Val = Builder.CreateOr(Val, Val1);
2346:     return Builder.CreateBitCast(Val, ConvertType(E->getType()));
2347:   }
2348: 
2349:   if (BuiltinID == clang::ARM::BI__builtin_arm_ldrex ||
2350:       BuiltinID == clang::ARM::BI__builtin_arm_ldaex) {
2351:     Value *LoadAddr = EmitScalarExpr(E->getArg(0));
2352: 
2353:     QualType Ty = E->getType();
2354:     llvm::Type *RealResTy = ConvertType(Ty);
2355:     llvm::Type *IntTy =
2356:         llvm::IntegerType::get(getLLVMContext(), getContext().getTypeSize(Ty));
2357: 
2358:     Function *F = CGM.getIntrinsic(
2359:         BuiltinID == clang::ARM::BI__builtin_arm_ldaex ? Intrinsic::arm_ldaex
2360:                                                        : Intrinsic::arm_ldrex,
2361:         DefaultPtrTy);
2362:     CallInst *Val = Builder.CreateCall(F, LoadAddr, "ldrex");
2363:     Val->addParamAttr(
2364:         0, Attribute::get(getLLVMContext(), Attribute::ElementType, IntTy));
2365: 
2366:     if (RealResTy->isPointerTy())
2367:       return Builder.CreateIntToPtr(Val, RealResTy);
2368:     else {
2369:       llvm::Type *IntResTy = llvm::IntegerType::get(
2370:           getLLVMContext(), CGM.getDataLayout().getTypeSizeInBits(RealResTy));
```
- **EN**: This block defines callable entry points like `get`, `getLLVMContext`; uses control flow (if) to specialize target builtin lowering.
- **CN**: 该代码块定义可调用入口，例如 `get`, `getLLVMContext`；通过控制流（if）细化 目标内建函数降级 行为。

### Lines 2371-2400
```cpp
2371:       return Builder.CreateBitCast(Builder.CreateTruncOrBitCast(Val, IntResTy),
2372:                                    RealResTy);
2373:     }
2374:   }
2375: 
2376:   if (BuiltinID == clang::ARM::BI__builtin_arm_strexd ||
2377:       ((BuiltinID == clang::ARM::BI__builtin_arm_stlex ||
2378:         BuiltinID == clang::ARM::BI__builtin_arm_strex) &&
2379:        getContext().getTypeSize(E->getArg(0)->getType()) == 64)) {
2380:     Function *F = CGM.getIntrinsic(
2381:         BuiltinID == clang::ARM::BI__builtin_arm_stlex ? Intrinsic::arm_stlexd
2382:                                                        : Intrinsic::arm_strexd);
2383:     llvm::Type *STy = llvm::StructType::get(Int32Ty, Int32Ty);
2384: 
2385:     Address Tmp = CreateMemTempWithoutCast(E->getArg(0)->getType());
2386:     Value *Val = EmitScalarExpr(E->getArg(0));
2387:     Builder.CreateStore(Val, Tmp);
2388: 
2389:     Address LdPtr = Tmp.withElementType(STy);
2390:     Val = Builder.CreateLoad(LdPtr);
2391: 
2392:     Value *Arg0 = Builder.CreateExtractValue(Val, 0);
2393:     Value *Arg1 = Builder.CreateExtractValue(Val, 1);
2394:     Value *StPtr = EmitScalarExpr(E->getArg(1));
2395:     return Builder.CreateCall(F, {Arg0, Arg1, StPtr}, "strexd");
2396:   }
2397: 
2398:   if (BuiltinID == clang::ARM::BI__builtin_arm_strex ||
2399:       BuiltinID == clang::ARM::BI__builtin_arm_stlex) {
2400:     Value *StoreVal = EmitScalarExpr(E->getArg(0));
```
- **EN**: This block uses control flow (if) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（if）细化 目标内建函数降级 行为。

### Lines 2401-2430
```cpp
2401:     Value *StoreAddr = EmitScalarExpr(E->getArg(1));
2402: 
2403:     QualType Ty = E->getArg(0)->getType();
2404:     llvm::Type *StoreTy =
2405:         llvm::IntegerType::get(getLLVMContext(), getContext().getTypeSize(Ty));
2406: 
2407:     if (StoreVal->getType()->isPointerTy())
2408:       StoreVal = Builder.CreatePtrToInt(StoreVal, Int32Ty);
2409:     else {
2410:       llvm::Type *IntTy = llvm::IntegerType::get(
2411:           getLLVMContext(),
2412:           CGM.getDataLayout().getTypeSizeInBits(StoreVal->getType()));
2413:       StoreVal = Builder.CreateBitCast(StoreVal, IntTy);
2414:       StoreVal = Builder.CreateZExtOrBitCast(StoreVal, Int32Ty);
2415:     }
2416: 
2417:     Function *F = CGM.getIntrinsic(
2418:         BuiltinID == clang::ARM::BI__builtin_arm_stlex ? Intrinsic::arm_stlex
2419:                                                        : Intrinsic::arm_strex,
2420:         StoreAddr->getType());
2421: 
2422:     CallInst *CI = Builder.CreateCall(F, {StoreVal, StoreAddr}, "strex");
2423:     CI->addParamAttr(
2424:         1, Attribute::get(getLLVMContext(), Attribute::ElementType, StoreTy));
2425:     return CI;
2426:   }
2427: 
2428:   if (BuiltinID == clang::ARM::BI__builtin_arm_clrex) {
2429:     Function *F = CGM.getIntrinsic(Intrinsic::arm_clrex);
2430:     return Builder.CreateCall(F);
```
- **EN**: This block defines callable entry points like `get`, `getLLVMContext`; uses control flow (if) to specialize target builtin lowering.
- **CN**: 该代码块定义可调用入口，例如 `get`, `getLLVMContext`；通过控制流（if）细化 目标内建函数降级 行为。

### Lines 2431-2460
```cpp
2431:   }
2432: 
2433:   // CRC32
2434:   Intrinsic::ID CRCIntrinsicID = Intrinsic::not_intrinsic;
2435:   switch (BuiltinID) {
2436:   case clang::ARM::BI__builtin_arm_crc32b:
2437:     CRCIntrinsicID = Intrinsic::arm_crc32b; break;
2438:   case clang::ARM::BI__builtin_arm_crc32cb:
2439:     CRCIntrinsicID = Intrinsic::arm_crc32cb; break;
2440:   case clang::ARM::BI__builtin_arm_crc32h:
2441:     CRCIntrinsicID = Intrinsic::arm_crc32h; break;
2442:   case clang::ARM::BI__builtin_arm_crc32ch:
2443:     CRCIntrinsicID = Intrinsic::arm_crc32ch; break;
2444:   case clang::ARM::BI__builtin_arm_crc32w:
2445:   case clang::ARM::BI__builtin_arm_crc32d:
2446:     CRCIntrinsicID = Intrinsic::arm_crc32w; break;
2447:   case clang::ARM::BI__builtin_arm_crc32cw:
2448:   case clang::ARM::BI__builtin_arm_crc32cd:
2449:     CRCIntrinsicID = Intrinsic::arm_crc32cw; break;
2450:   }
2451: 
2452:   if (CRCIntrinsicID != Intrinsic::not_intrinsic) {
2453:     Value *Arg0 = EmitScalarExpr(E->getArg(0));
2454:     Value *Arg1 = EmitScalarExpr(E->getArg(1));
2455: 
2456:     // crc32{c,}d intrinsics are implemented as two calls to crc32{c,}w
2457:     // intrinsics, hence we need different codegen for these cases.
2458:     if (BuiltinID == clang::ARM::BI__builtin_arm_crc32d ||
2459:         BuiltinID == clang::ARM::BI__builtin_arm_crc32cd) {
2460:       Value *C1 = llvm::ConstantInt::get(Int64Ty, 32);
```
- **EN**: This block uses control flow (if, switch, case) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（if, switch, case）细化 目标内建函数降级 行为。

### Lines 2461-2490
```cpp
2461:       Value *Arg1a = Builder.CreateTruncOrBitCast(Arg1, Int32Ty);
2462:       Value *Arg1b = Builder.CreateLShr(Arg1, C1);
2463:       Arg1b = Builder.CreateTruncOrBitCast(Arg1b, Int32Ty);
2464: 
2465:       Function *F = CGM.getIntrinsic(CRCIntrinsicID);
2466:       Value *Res = Builder.CreateCall(F, {Arg0, Arg1a});
2467:       return Builder.CreateCall(F, {Res, Arg1b});
2468:     } else {
2469:       Arg1 = Builder.CreateZExtOrBitCast(Arg1, Int32Ty);
2470: 
2471:       Function *F = CGM.getIntrinsic(CRCIntrinsicID);
2472:       return Builder.CreateCall(F, {Arg0, Arg1});
2473:     }
2474:   }
2475: 
2476:   if (BuiltinID == clang::ARM::BI__builtin_arm_rsr ||
2477:       BuiltinID == clang::ARM::BI__builtin_arm_rsr64 ||
2478:       BuiltinID == clang::ARM::BI__builtin_arm_rsrp ||
2479:       BuiltinID == clang::ARM::BI__builtin_arm_wsr ||
2480:       BuiltinID == clang::ARM::BI__builtin_arm_wsr64 ||
2481:       BuiltinID == clang::ARM::BI__builtin_arm_wsrp) {
2482: 
2483:     SpecialRegisterAccessKind AccessKind = Write;
2484:     if (BuiltinID == clang::ARM::BI__builtin_arm_rsr ||
2485:         BuiltinID == clang::ARM::BI__builtin_arm_rsr64 ||
2486:         BuiltinID == clang::ARM::BI__builtin_arm_rsrp)
2487:       AccessKind = VolatileRead;
2488: 
2489:     bool IsPointerBuiltin = BuiltinID == clang::ARM::BI__builtin_arm_rsrp ||
2490:                             BuiltinID == clang::ARM::BI__builtin_arm_wsrp;
```
- **EN**: This block uses control flow (if) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（if）细化 目标内建函数降级 行为。

### Lines 2491-2520
```cpp
2491: 
2492:     bool Is64Bit = BuiltinID == clang::ARM::BI__builtin_arm_rsr64 ||
2493:                    BuiltinID == clang::ARM::BI__builtin_arm_wsr64;
2494: 
2495:     llvm::Type *ValueType;
2496:     llvm::Type *RegisterType;
2497:     if (IsPointerBuiltin) {
2498:       ValueType = VoidPtrTy;
2499:       RegisterType = Int32Ty;
2500:     } else if (Is64Bit) {
2501:       ValueType = RegisterType = Int64Ty;
2502:     } else {
2503:       ValueType = RegisterType = Int32Ty;
2504:     }
2505: 
2506:     return EmitSpecialRegisterBuiltin(*this, E, RegisterType, ValueType,
2507:                                       AccessKind);
2508:   }
2509: 
2510:   if (BuiltinID == ARM::BI__builtin_sponentry) {
2511:     llvm::Function *F = CGM.getIntrinsic(Intrinsic::sponentry, AllocaInt8PtrTy);
2512:     return Builder.CreateCall(F);
2513:   }
2514: 
2515:   // Handle MSVC intrinsics before argument evaluation to prevent double
2516:   // evaluation.
2517:   if (std::optional<MSVCIntrin> MsvcIntId = translateArmToMsvcIntrin(BuiltinID))
2518:     return EmitMSVCBuiltinExpr(*MsvcIntId, E);
2519: 
2520:   // Deal with MVE builtins
```
- **EN**: This block defines callable entry points like `EmitSpecialRegisterBuiltin`; uses control flow (if) to specialize target builtin lowering.
- **CN**: 该代码块定义可调用入口，例如 `EmitSpecialRegisterBuiltin`；通过控制流（if）细化 目标内建函数降级 行为。

### Lines 2521-2550
```cpp
2521:   if (Value *Result = EmitARMMVEBuiltinExpr(BuiltinID, E, ReturnValue, Arch))
2522:     return Result;
2523:   // Handle CDE builtins
2524:   if (Value *Result = EmitARMCDEBuiltinExpr(BuiltinID, E, ReturnValue, Arch))
2525:     return Result;
2526: 
2527:   // Some intrinsics are equivalent - if they are use the base intrinsic ID.
2528:   auto It = llvm::find_if(NEONEquivalentIntrinsicMap, [BuiltinID](auto &P) {
2529:     return P.first == BuiltinID;
2530:   });
2531:   if (It != end(NEONEquivalentIntrinsicMap))
2532:     BuiltinID = It->second;
2533: 
2534:   // Find out if any arguments are required to be integer constant
2535:   // expressions.
2536:   unsigned ICEArguments = 0;
2537:   ASTContext::GetBuiltinTypeError Error;
2538:   getContext().GetBuiltinType(BuiltinID, Error, &ICEArguments);
2539:   assert(Error == ASTContext::GE_None && "Should not codegen an error");
2540: 
2541:   auto getAlignmentValue32 = [&](Address addr) -> Value* {
2542:     return Builder.getInt32(addr.getAlignment().getQuantity());
2543:   };
2544: 
2545:   Address PtrOp0 = Address::invalid();
2546:   Address PtrOp1 = Address::invalid();
2547:   SmallVector<Value*, 4> Ops;
2548:   bool HasExtraArg = HasExtraNeonArgument(BuiltinID);
2549:   unsigned NumArgs = E->getNumArgs() - (HasExtraArg ? 1 : 0);
2550:   for (unsigned i = 0, e = NumArgs; i != e; i++) {
```
- **EN**: This block defines callable entry points like `getContext`; uses control flow (if, for) to specialize target builtin lowering; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块定义可调用入口，例如 `getContext`；通过控制流（if, for）细化 目标内建函数降级 行为；使用断言或不可达标记保护关键不变量。

### Lines 2551-2580
```cpp
2551:     if (i == 0) {
2552:       switch (BuiltinID) {
2553:       case NEON::BI__builtin_neon_vld1_v:
2554:       case NEON::BI__builtin_neon_vld1q_v:
2555:       case NEON::BI__builtin_neon_vld1q_lane_v:
2556:       case NEON::BI__builtin_neon_vld1_lane_v:
2557:       case NEON::BI__builtin_neon_vld1_dup_v:
2558:       case NEON::BI__builtin_neon_vld1q_dup_v:
2559:       case NEON::BI__builtin_neon_vst1_v:
2560:       case NEON::BI__builtin_neon_vst1q_v:
2561:       case NEON::BI__builtin_neon_vst1q_lane_v:
2562:       case NEON::BI__builtin_neon_vst1_lane_v:
2563:       case NEON::BI__builtin_neon_vst2_v:
2564:       case NEON::BI__builtin_neon_vst2q_v:
2565:       case NEON::BI__builtin_neon_vst2_lane_v:
2566:       case NEON::BI__builtin_neon_vst2q_lane_v:
2567:       case NEON::BI__builtin_neon_vst3_v:
2568:       case NEON::BI__builtin_neon_vst3q_v:
2569:       case NEON::BI__builtin_neon_vst3_lane_v:
2570:       case NEON::BI__builtin_neon_vst3q_lane_v:
2571:       case NEON::BI__builtin_neon_vst4_v:
2572:       case NEON::BI__builtin_neon_vst4q_v:
2573:       case NEON::BI__builtin_neon_vst4_lane_v:
2574:       case NEON::BI__builtin_neon_vst4q_lane_v:
2575:         // Get the alignment for the argument in addition to the value;
2576:         // we'll use it later.
2577:         PtrOp0 = EmitPointerWithAlignment(E->getArg(0));
2578:         Ops.push_back(PtrOp0.emitRawPointer(*this));
2579:         continue;
2580:       }
```
- **EN**: This block uses control flow (if, switch, case) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（if, switch, case）细化 目标内建函数降级 行为。

### Lines 2581-2610
```cpp
2581:     }
2582:     if (i == 1) {
2583:       switch (BuiltinID) {
2584:       case NEON::BI__builtin_neon_vld2_v:
2585:       case NEON::BI__builtin_neon_vld2q_v:
2586:       case NEON::BI__builtin_neon_vld3_v:
2587:       case NEON::BI__builtin_neon_vld3q_v:
2588:       case NEON::BI__builtin_neon_vld4_v:
2589:       case NEON::BI__builtin_neon_vld4q_v:
2590:       case NEON::BI__builtin_neon_vld2_lane_v:
2591:       case NEON::BI__builtin_neon_vld2q_lane_v:
2592:       case NEON::BI__builtin_neon_vld3_lane_v:
2593:       case NEON::BI__builtin_neon_vld3q_lane_v:
2594:       case NEON::BI__builtin_neon_vld4_lane_v:
2595:       case NEON::BI__builtin_neon_vld4q_lane_v:
2596:       case NEON::BI__builtin_neon_vld2_dup_v:
2597:       case NEON::BI__builtin_neon_vld2q_dup_v:
2598:       case NEON::BI__builtin_neon_vld3_dup_v:
2599:       case NEON::BI__builtin_neon_vld3q_dup_v:
2600:       case NEON::BI__builtin_neon_vld4_dup_v:
2601:       case NEON::BI__builtin_neon_vld4q_dup_v:
2602:         // Get the alignment for the argument in addition to the value;
2603:         // we'll use it later.
2604:         PtrOp1 = EmitPointerWithAlignment(E->getArg(1));
2605:         Ops.push_back(PtrOp1.emitRawPointer(*this));
2606:         continue;
2607:       }
2608:     }
2609: 
2610:     Ops.push_back(EmitScalarOrConstFoldImmArg(ICEArguments, i, E));
```
- **EN**: This block uses control flow (if, switch, case) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（if, switch, case）细化 目标内建函数降级 行为。

### Lines 2611-2640
```cpp
2611:   }
2612: 
2613:   switch (BuiltinID) {
2614:   default: break;
2615: 
2616:   case NEON::BI__builtin_neon_vget_lane_i8:
2617:   case NEON::BI__builtin_neon_vget_lane_i16:
2618:   case NEON::BI__builtin_neon_vget_lane_i32:
2619:   case NEON::BI__builtin_neon_vget_lane_i64:
2620:   case NEON::BI__builtin_neon_vget_lane_bf16:
2621:   case NEON::BI__builtin_neon_vget_lane_f32:
2622:   case NEON::BI__builtin_neon_vgetq_lane_i8:
2623:   case NEON::BI__builtin_neon_vgetq_lane_i16:
2624:   case NEON::BI__builtin_neon_vgetq_lane_i32:
2625:   case NEON::BI__builtin_neon_vgetq_lane_i64:
2626:   case NEON::BI__builtin_neon_vgetq_lane_bf16:
2627:   case NEON::BI__builtin_neon_vgetq_lane_f32:
2628:   case NEON::BI__builtin_neon_vduph_lane_bf16:
2629:   case NEON::BI__builtin_neon_vduph_laneq_bf16:
2630:     return Builder.CreateExtractElement(Ops[0], Ops[1], "vget_lane");
2631: 
2632:   case NEON::BI__builtin_neon_vrndns_f32: {
2633:     Value *Arg = EmitScalarExpr(E->getArg(0));
2634:     llvm::Type *Tys[] = {Arg->getType()};
2635:     Function *F = CGM.getIntrinsic(Intrinsic::roundeven, Tys);
2636:     return Builder.CreateCall(F, {Arg}, "vrndn"); }
2637: 
2638:   case NEON::BI__builtin_neon_vset_lane_i8:
2639:   case NEON::BI__builtin_neon_vset_lane_i16:
2640:   case NEON::BI__builtin_neon_vset_lane_i32:
```
- **EN**: This block uses control flow (switch, case) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（switch, case）细化 目标内建函数降级 行为。

### Lines 2641-2670
```cpp
2641:   case NEON::BI__builtin_neon_vset_lane_i64:
2642:   case NEON::BI__builtin_neon_vset_lane_bf16:
2643:   case NEON::BI__builtin_neon_vset_lane_f32:
2644:   case NEON::BI__builtin_neon_vsetq_lane_i8:
2645:   case NEON::BI__builtin_neon_vsetq_lane_i16:
2646:   case NEON::BI__builtin_neon_vsetq_lane_i32:
2647:   case NEON::BI__builtin_neon_vsetq_lane_i64:
2648:   case NEON::BI__builtin_neon_vsetq_lane_bf16:
2649:   case NEON::BI__builtin_neon_vsetq_lane_f32:
2650:     return Builder.CreateInsertElement(Ops[1], Ops[0], Ops[2], "vset_lane");
2651: 
2652:   case NEON::BI__builtin_neon_vsha1h_u32:
2653:     return EmitNeonCall(CGM.getIntrinsic(Intrinsic::arm_neon_sha1h), Ops,
2654:                         "vsha1h");
2655:   case NEON::BI__builtin_neon_vsha1cq_u32:
2656:     return EmitNeonCall(CGM.getIntrinsic(Intrinsic::arm_neon_sha1c), Ops,
2657:                         "vsha1h");
2658:   case NEON::BI__builtin_neon_vsha1pq_u32:
2659:     return EmitNeonCall(CGM.getIntrinsic(Intrinsic::arm_neon_sha1p), Ops,
2660:                         "vsha1h");
2661:   case NEON::BI__builtin_neon_vsha1mq_u32:
2662:     return EmitNeonCall(CGM.getIntrinsic(Intrinsic::arm_neon_sha1m), Ops,
2663:                         "vsha1h");
2664: 
2665:   case NEON::BI__builtin_neon_vcvth_bf16_f32: {
2666:     return EmitNeonCall(CGM.getIntrinsic(Intrinsic::arm_neon_vcvtbfp2bf), Ops,
2667:                         "vcvtbfp2bf");
2668:   }
2669: 
2670:   // The ARM _MoveToCoprocessor builtins put the input register value as
```
- **EN**: This block defines callable entry points like `EmitNeonCall`; uses control flow (case) to specialize target builtin lowering.
- **CN**: 该代码块定义可调用入口，例如 `EmitNeonCall`；通过控制流（case）细化 目标内建函数降级 行为。

### Lines 2671-2700
```cpp
2671:   // the first argument, but the LLVM intrinsic expects it as the third one.
2672:   case clang::ARM::BI_MoveToCoprocessor:
2673:   case clang::ARM::BI_MoveToCoprocessor2: {
2674:     Function *F = CGM.getIntrinsic(BuiltinID == clang::ARM::BI_MoveToCoprocessor
2675:                                        ? Intrinsic::arm_mcr
2676:                                        : Intrinsic::arm_mcr2);
2677:     return Builder.CreateCall(F, {Ops[1], Ops[2], Ops[0],
2678:                                   Ops[3], Ops[4], Ops[5]});
2679:   }
2680:   }
2681: 
2682:   // Get the last argument, which specifies the vector type.
2683:   assert(HasExtraArg);
2684:   const Expr *Arg = E->getArg(E->getNumArgs()-1);
2685:   std::optional<llvm::APSInt> Result =
2686:       Arg->getIntegerConstantExpr(getContext());
2687:   if (!Result)
2688:     return nullptr;
2689: 
2690:   if (BuiltinID == clang::ARM::BI__builtin_arm_vcvtr_f ||
2691:       BuiltinID == clang::ARM::BI__builtin_arm_vcvtr_d) {
2692:     // Determine the overloaded type of this builtin.
2693:     llvm::Type *Ty;
2694:     if (BuiltinID == clang::ARM::BI__builtin_arm_vcvtr_f)
2695:       Ty = FloatTy;
2696:     else
2697:       Ty = DoubleTy;
2698: 
2699:     // Determine whether this is an unsigned conversion or not.
2700:     bool usgn = Result->getZExtValue() == 1;
```
- **EN**: This block uses control flow (if, case) to specialize target builtin lowering; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块通过控制流（if, case）细化 目标内建函数降级 行为；使用断言或不可达标记保护关键不变量。

### Lines 2701-2730
```cpp
2701:     unsigned Int = usgn ? Intrinsic::arm_vcvtru : Intrinsic::arm_vcvtr;
2702: 
2703:     // Call the appropriate intrinsic.
2704:     Function *F = CGM.getIntrinsic(Int, Ty);
2705:     return Builder.CreateCall(F, Ops, "vcvtr");
2706:   }
2707: 
2708:   // Determine the type of this overloaded NEON intrinsic.
2709:   NeonTypeFlags Type = Result->getZExtValue();
2710:   bool usgn = Type.isUnsigned();
2711:   bool rightShift = false;
2712: 
2713:   llvm::FixedVectorType *VTy =
2714:       GetNeonType(this, Type, getTarget().hasFastHalfType(), false,
2715:                   getTarget().hasBFloat16Type());
2716:   llvm::Type *Ty = VTy;
2717:   if (!Ty)
2718:     return nullptr;
2719: 
2720:   // Many NEON builtins have identical semantics and uses in ARM and
2721:   // AArch64. Emit these in a single function.
2722:   auto IntrinsicMap = ArrayRef(ARMSIMDIntrinsicMap);
2723:   const ARMVectorIntrinsicInfo *Builtin = findARMVectorIntrinsicInMap(
2724:       IntrinsicMap, BuiltinID, NEONSIMDIntrinsicsProvenSorted);
2725:   if (Builtin)
2726:     return EmitCommonNeonBuiltinExpr(
2727:         Builtin->BuiltinID, Builtin->LLVMIntrinsic, Builtin->AltLLVMIntrinsic,
2728:         Builtin->NameHint, Builtin->TypeModifier, E, Ops, PtrOp0, PtrOp1, Arch);
2729: 
2730:   unsigned Int;
```
- **EN**: This block spells out callable entry points like `GetNeonType`; uses control flow (if) to specialize target builtin lowering.
- **CN**: 该代码块给出可调用入口的声明，例如 `GetNeonType`；通过控制流（if）细化 目标内建函数降级 行为。

### Lines 2731-2760
```cpp
2731:   switch (BuiltinID) {
2732:   default: return nullptr;
2733:   case NEON::BI__builtin_neon_vld1q_lane_v:
2734:     // Handle 64-bit integer elements as a special case.  Use shuffles of
2735:     // one-element vectors to avoid poor code for i64 in the backend.
2736:     if (VTy->getElementType()->isIntegerTy(64)) {
2737:       // Extract the other lane.
2738:       Ops[1] = Builder.CreateBitCast(Ops[1], Ty);
2739:       int Lane = cast<ConstantInt>(Ops[2])->getZExtValue();
2740:       Value *SV = llvm::ConstantVector::get(ConstantInt::get(Int32Ty, 1-Lane));
2741:       Ops[1] = Builder.CreateShuffleVector(Ops[1], Ops[1], SV);
2742:       // Load the value as a one-element vector.
2743:       Ty = llvm::FixedVectorType::get(VTy->getElementType(), 1);
2744:       llvm::Type *Tys[] = {Ty, Int8PtrTy};
2745:       Function *F = CGM.getIntrinsic(Intrinsic::arm_neon_vld1, Tys);
2746:       Value *Align = getAlignmentValue32(PtrOp0);
2747:       Value *Ld = Builder.CreateCall(F, {Ops[0], Align});
2748:       // Combine them.
2749:       int Indices[] = {1 - Lane, Lane};
2750:       return Builder.CreateShuffleVector(Ops[1], Ld, Indices, "vld1q_lane");
2751:     }
2752:     [[fallthrough]];
2753:   case NEON::BI__builtin_neon_vld1_lane_v: {
2754:     Ops[1] = Builder.CreateBitCast(Ops[1], Ty);
2755:     PtrOp0 = PtrOp0.withElementType(VTy->getElementType());
2756:     Value *Ld = Builder.CreateLoad(PtrOp0);
2757:     return Builder.CreateInsertElement(Ops[1], Ld, Ops[2], "vld1_lane");
2758:   }
2759:   case NEON::BI__builtin_neon_vqrshrn_n_v:
2760:     Int =
```
- **EN**: This block uses control flow (if, switch, case) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（if, switch, case）细化 目标内建函数降级 行为。

### Lines 2761-2790
```cpp
2761:       usgn ? Intrinsic::arm_neon_vqrshiftnu : Intrinsic::arm_neon_vqrshiftns;
2762:     return EmitNeonCall(CGM.getIntrinsic(Int, Ty), Ops, "vqrshrn_n",
2763:                         1, true);
2764:   case NEON::BI__builtin_neon_vqrshrun_n_v:
2765:     return EmitNeonCall(CGM.getIntrinsic(Intrinsic::arm_neon_vqrshiftnsu, Ty),
2766:                         Ops, "vqrshrun_n", 1, true);
2767:   case NEON::BI__builtin_neon_vqshrn_n_v:
2768:     Int = usgn ? Intrinsic::arm_neon_vqshiftnu : Intrinsic::arm_neon_vqshiftns;
2769:     return EmitNeonCall(CGM.getIntrinsic(Int, Ty), Ops, "vqshrn_n",
2770:                         1, true);
2771:   case NEON::BI__builtin_neon_vqshrun_n_v:
2772:     return EmitNeonCall(CGM.getIntrinsic(Intrinsic::arm_neon_vqshiftnsu, Ty),
2773:                         Ops, "vqshrun_n", 1, true);
2774:   case NEON::BI__builtin_neon_vrecpe_v:
2775:   case NEON::BI__builtin_neon_vrecpeq_v:
2776:     return EmitNeonCall(CGM.getIntrinsic(Intrinsic::arm_neon_vrecpe, Ty),
2777:                         Ops, "vrecpe");
2778:   case NEON::BI__builtin_neon_vrshrn_n_v:
2779:     return EmitNeonCall(CGM.getIntrinsic(Intrinsic::arm_neon_vrshiftn, Ty),
2780:                         Ops, "vrshrn_n", 1, true);
2781:   case NEON::BI__builtin_neon_vrsra_n_v:
2782:   case NEON::BI__builtin_neon_vrsraq_n_v:
2783:     Ops[0] = Builder.CreateBitCast(Ops[0], Ty);
2784:     Ops[1] = Builder.CreateBitCast(Ops[1], Ty);
2785:     Ops[2] = EmitNeonShiftVector(Ops[2], Ty, true);
2786:     Int = usgn ? Intrinsic::arm_neon_vrshiftu : Intrinsic::arm_neon_vrshifts;
2787:     Ops[1] = Builder.CreateCall(CGM.getIntrinsic(Int, Ty), {Ops[1], Ops[2]});
2788:     return Builder.CreateAdd(Ops[0], Ops[1], "vrsra_n");
2789:   case NEON::BI__builtin_neon_vsri_n_v:
2790:   case NEON::BI__builtin_neon_vsriq_n_v:
```
- **EN**: This block defines callable entry points like `EmitNeonCall`; uses control flow (case) to specialize target builtin lowering.
- **CN**: 该代码块定义可调用入口，例如 `EmitNeonCall`；通过控制流（case）细化 目标内建函数降级 行为。

### Lines 2791-2820
```cpp
2791:     rightShift = true;
2792:     [[fallthrough]];
2793:   case NEON::BI__builtin_neon_vsli_n_v:
2794:   case NEON::BI__builtin_neon_vsliq_n_v:
2795:     Ops[2] = EmitNeonShiftVector(Ops[2], Ty, rightShift);
2796:     return EmitNeonCall(CGM.getIntrinsic(Intrinsic::arm_neon_vshiftins, Ty),
2797:                         Ops, "vsli_n");
2798:   case NEON::BI__builtin_neon_vsra_n_v:
2799:   case NEON::BI__builtin_neon_vsraq_n_v:
2800:     Ops[0] = Builder.CreateBitCast(Ops[0], Ty);
2801:     Ops[1] = EmitNeonRShiftImm(Ops[1], Ops[2], Ty, usgn, "vsra_n");
2802:     return Builder.CreateAdd(Ops[0], Ops[1]);
2803:   case NEON::BI__builtin_neon_vst1q_lane_v:
2804:     // Handle 64-bit integer elements as a special case.  Use a shuffle to get
2805:     // a one-element vector and avoid poor code for i64 in the backend.
2806:     if (VTy->getElementType()->isIntegerTy(64)) {
2807:       Ops[1] = Builder.CreateBitCast(Ops[1], Ty);
2808:       Value *SV = llvm::ConstantVector::get(cast<llvm::Constant>(Ops[2]));
2809:       Ops[1] = Builder.CreateShuffleVector(Ops[1], Ops[1], SV);
2810:       Ops[2] = getAlignmentValue32(PtrOp0);
2811:       llvm::Type *Tys[] = {Int8PtrTy, Ops[1]->getType()};
2812:       return Builder.CreateCall(CGM.getIntrinsic(Intrinsic::arm_neon_vst1,
2813:                                                  Tys), Ops);
2814:     }
2815:     [[fallthrough]];
2816:   case NEON::BI__builtin_neon_vst1_lane_v: {
2817:     Ops[1] = Builder.CreateBitCast(Ops[1], Ty);
2818:     Ops[1] = Builder.CreateExtractElement(Ops[1], Ops[2]);
2819:     return Builder.CreateStore(Ops[1],
2820:                                PtrOp0.withElementType(Ops[1]->getType()));
```
- **EN**: This block defines callable entry points like `EmitNeonCall`; uses control flow (if, case) to specialize target builtin lowering.
- **CN**: 该代码块定义可调用入口，例如 `EmitNeonCall`；通过控制流（if, case）细化 目标内建函数降级 行为。

### Lines 2821-2850
```cpp
2821:   }
2822:   case NEON::BI__builtin_neon_vtbl1_v:
2823:     return EmitNeonCall(CGM.getIntrinsic(Intrinsic::arm_neon_vtbl1),
2824:                         Ops, "vtbl1");
2825:   case NEON::BI__builtin_neon_vtbl2_v:
2826:     return EmitNeonCall(CGM.getIntrinsic(Intrinsic::arm_neon_vtbl2),
2827:                         Ops, "vtbl2");
2828:   case NEON::BI__builtin_neon_vtbl3_v:
2829:     return EmitNeonCall(CGM.getIntrinsic(Intrinsic::arm_neon_vtbl3),
2830:                         Ops, "vtbl3");
2831:   case NEON::BI__builtin_neon_vtbl4_v:
2832:     return EmitNeonCall(CGM.getIntrinsic(Intrinsic::arm_neon_vtbl4),
2833:                         Ops, "vtbl4");
2834:   case NEON::BI__builtin_neon_vtbx1_v:
2835:     return EmitNeonCall(CGM.getIntrinsic(Intrinsic::arm_neon_vtbx1),
2836:                         Ops, "vtbx1");
2837:   case NEON::BI__builtin_neon_vtbx2_v:
2838:     return EmitNeonCall(CGM.getIntrinsic(Intrinsic::arm_neon_vtbx2),
2839:                         Ops, "vtbx2");
2840:   case NEON::BI__builtin_neon_vtbx3_v:
2841:     return EmitNeonCall(CGM.getIntrinsic(Intrinsic::arm_neon_vtbx3),
2842:                         Ops, "vtbx3");
2843:   case NEON::BI__builtin_neon_vtbx4_v:
2844:     return EmitNeonCall(CGM.getIntrinsic(Intrinsic::arm_neon_vtbx4),
2845:                         Ops, "vtbx4");
2846:   }
2847: }
2848: 
2849: template<typename Integer>
2850: static Integer GetIntegerConstantValue(const Expr *E, ASTContext &Context) {
```
- **EN**: This block defines callable entry points like `EmitNeonCall`, `GetIntegerConstantValue`; uses control flow (case) to specialize target builtin lowering.
- **CN**: 该代码块定义可调用入口，例如 `EmitNeonCall`, `GetIntegerConstantValue`；通过控制流（case）细化 目标内建函数降级 行为。

### Lines 2851-2880
```cpp
2851:   return E->getIntegerConstantExpr(Context)->getExtValue();
2852: }
2853: 
2854: static llvm::Value *SignOrZeroExtend(CGBuilderTy &Builder, llvm::Value *V,
2855:                                      llvm::Type *T, bool Unsigned) {
2856:   // Helper function called by Tablegen-constructed ARM MVE builtin codegen,
2857:   // which finds it convenient to specify signed/unsigned as a boolean flag.
2858:   return Unsigned ? Builder.CreateZExt(V, T) : Builder.CreateSExt(V, T);
2859: }
2860: 
2861: static llvm::Value *MVEImmediateShr(CGBuilderTy &Builder, llvm::Value *V,
2862:                                     uint32_t Shift, bool Unsigned) {
2863:   // MVE helper function for integer shift right. This must handle signed vs
2864:   // unsigned, and also deal specially with the case where the shift count is
2865:   // equal to the lane size. In LLVM IR, an LShr with that parameter would be
2866:   // undefined behavior, but in MVE it's legal, so we must convert it to code
2867:   // that is not undefined in IR.
2868:   unsigned LaneBits = cast<llvm::VectorType>(V->getType())
2869:                           ->getElementType()
2870:                           ->getPrimitiveSizeInBits();
2871:   if (Shift == LaneBits) {
2872:     // An unsigned shift of the full lane size always generates zero, so we can
2873:     // simply emit a zero vector. A signed shift of the full lane size does the
2874:     // same thing as shifting by one bit fewer.
2875:     if (Unsigned)
2876:       return llvm::Constant::getNullValue(V->getType());
2877:     else
2878:       --Shift;
2879:   }
2880:   return Unsigned ? Builder.CreateLShr(V, Shift) : Builder.CreateAShr(V, Shift);
```
- **EN**: This block uses control flow (if) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（if）细化 目标内建函数降级 行为。

### Lines 2881-2910
```cpp
2881: }
2882: 
2883: static llvm::Value *ARMMVEVectorSplat(CGBuilderTy &Builder, llvm::Value *V) {
2884:   // MVE-specific helper function for a vector splat, which infers the element
2885:   // count of the output vector by knowing that MVE vectors are all 128 bits
2886:   // wide.
2887:   unsigned Elements = 128 / V->getType()->getPrimitiveSizeInBits();
2888:   return Builder.CreateVectorSplat(Elements, V);
2889: }
2890: 
2891: static llvm::Value *ARMMVEVectorReinterpret(CGBuilderTy &Builder,
2892:                                             CodeGenFunction *CGF,
2893:                                             llvm::Value *V,
2894:                                             llvm::Type *DestType) {
2895:   // Convert one MVE vector type into another by reinterpreting its in-register
2896:   // format.
2897:   //
2898:   // Little-endian, this is identical to a bitcast (which reinterprets the
2899:   // memory format). But big-endian, they're not necessarily the same, because
2900:   // the register and memory formats map to each other differently depending on
2901:   // the lane size.
2902:   //
2903:   // We generate a bitcast whenever we can (if we're little-endian, or if the
2904:   // lane sizes are the same anyway). Otherwise we fall back to an IR intrinsic
2905:   // that performs the different kind of reinterpretation.
2906:   if (CGF->getTarget().isBigEndian() &&
2907:       V->getType()->getScalarSizeInBits() != DestType->getScalarSizeInBits()) {
2908:     return Builder.CreateCall(
2909:         CGF->CGM.getIntrinsic(Intrinsic::arm_mve_vreinterpretq,
2910:                               {DestType, V->getType()}),
```
- **EN**: This block uses control flow (if) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（if）细化 目标内建函数降级 行为。

### Lines 2911-2940
```cpp
2911:         V);
2912:   } else {
2913:     return Builder.CreateBitCast(V, DestType);
2914:   }
2915: }
2916: 
2917: static llvm::Value *VectorUnzip(CGBuilderTy &Builder, llvm::Value *V, bool Odd) {
2918:   // Make a shufflevector that extracts every other element of a vector (evens
2919:   // or odds, as desired).
2920:   SmallVector<int, 16> Indices;
2921:   unsigned InputElements =
2922:       cast<llvm::FixedVectorType>(V->getType())->getNumElements();
2923:   for (unsigned i = 0; i < InputElements; i += 2)
2924:     Indices.push_back(i + Odd);
2925:   return Builder.CreateShuffleVector(V, Indices);
2926: }
2927: 
2928: static llvm::Value *VectorZip(CGBuilderTy &Builder, llvm::Value *V0,
2929:                               llvm::Value *V1) {
2930:   // Make a shufflevector that interleaves two vectors element by element.
2931:   assert(V0->getType() == V1->getType() && "Can't zip different vector types");
2932:   SmallVector<int, 16> Indices;
2933:   unsigned InputElements =
2934:       cast<llvm::FixedVectorType>(V0->getType())->getNumElements();
2935:   for (unsigned i = 0; i < InputElements; i++) {
2936:     Indices.push_back(i);
2937:     Indices.push_back(i + InputElements);
2938:   }
2939:   return Builder.CreateShuffleVector(V0, V1, Indices);
2940: }
```
- **EN**: This block uses control flow (for) to specialize target builtin lowering; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块通过控制流（for）细化 目标内建函数降级 行为；使用断言或不可达标记保护关键不变量。

### Lines 2941-2970
```cpp
2941: 
2942: template<unsigned HighBit, unsigned OtherBits>
2943: static llvm::Value *ARMMVEConstantSplat(CGBuilderTy &Builder, llvm::Type *VT) {
2944:   // MVE-specific helper function to make a vector splat of a constant such as
2945:   // UINT_MAX or INT_MIN, in which all bits below the highest one are equal.
2946:   llvm::Type *T = cast<llvm::VectorType>(VT)->getElementType();
2947:   unsigned LaneBits = T->getPrimitiveSizeInBits();
2948:   uint32_t Value = HighBit << (LaneBits - 1);
2949:   if (OtherBits)
2950:     Value |= (1UL << (LaneBits - 1)) - 1;
2951:   llvm::Value *Lane = llvm::ConstantInt::get(T, Value);
2952:   return ARMMVEVectorSplat(Builder, Lane);
2953: }
2954: 
2955: static llvm::Value *ARMMVEVectorElementReverse(CGBuilderTy &Builder,
2956:                                                llvm::Value *V,
2957:                                                unsigned ReverseWidth) {
2958:   // MVE-specific helper function which reverses the elements of a
2959:   // vector within every (ReverseWidth)-bit collection of lanes.
2960:   SmallVector<int, 16> Indices;
2961:   unsigned LaneSize = V->getType()->getScalarSizeInBits();
2962:   unsigned Elements = 128 / LaneSize;
2963:   unsigned Mask = ReverseWidth / LaneSize - 1;
2964:   for (unsigned i = 0; i < Elements; i++)
2965:     Indices.push_back(i ^ Mask);
2966:   return Builder.CreateShuffleVector(V, Indices);
2967: }
2968: 
2969: static llvm::Value *ARMMVECreateSIToFP(CGBuilderTy &Builder,
2970:                                        CodeGenFunction *CGF, llvm::Value *V,
```
- **EN**: This block defines callable entry points like `ARMMVEVectorSplat`; uses control flow (if, for) to specialize target builtin lowering.
- **CN**: 该代码块定义可调用入口，例如 `ARMMVEVectorSplat`；通过控制流（if, for）细化 目标内建函数降级 行为。

### Lines 2971-3000
```cpp
2971:                                        llvm::Type *Ty) {
2972:   return Builder.CreateCall(
2973:       CGF->CGM.getIntrinsic(Intrinsic::arm_mve_vcvt_fp_int, {Ty, V->getType()}),
2974:       {V, llvm::ConstantInt::get(Builder.getInt32Ty(), 0)});
2975: }
2976: 
2977: static llvm::Value *ARMMVECreateUIToFP(CGBuilderTy &Builder,
2978:                                        CodeGenFunction *CGF, llvm::Value *V,
2979:                                        llvm::Type *Ty) {
2980:   return Builder.CreateCall(
2981:       CGF->CGM.getIntrinsic(Intrinsic::arm_mve_vcvt_fp_int, {Ty, V->getType()}),
2982:       {V, llvm::ConstantInt::get(Builder.getInt32Ty(), 1)});
2983: }
2984: 
2985: static llvm::Value *ARMMVECreateFPToSI(CGBuilderTy &Builder,
2986:                                        CodeGenFunction *CGF, llvm::Value *V,
2987:                                        llvm::Type *Ty) {
2988:   return Builder.CreateCall(
2989:       CGF->CGM.getIntrinsic(Intrinsic::arm_mve_vcvt_int_fp, {Ty, V->getType()}),
2990:       {V, llvm::ConstantInt::get(Builder.getInt32Ty(), 0)});
2991: }
2992: 
2993: static llvm::Value *ARMMVECreateFPToUI(CGBuilderTy &Builder,
2994:                                        CodeGenFunction *CGF, llvm::Value *V,
2995:                                        llvm::Type *Ty) {
2996:   return Builder.CreateCall(
2997:       CGF->CGM.getIntrinsic(Intrinsic::arm_mve_vcvt_int_fp, {Ty, V->getType()}),
2998:       {V, llvm::ConstantInt::get(Builder.getInt32Ty(), 1)});
2999: }
3000: 
```
- **EN**: This block provides glue code, layout, or delimiters needed by the surrounding target builtin lowering implementation.
- **CN**: 该代码块为周围的 目标内建函数降级 实现提供必要的胶水代码、布局或分隔结构。

### Lines 3001-3030
```cpp
3001: Value *CodeGenFunction::EmitARMMVEBuiltinExpr(unsigned BuiltinID,
3002:                                               const CallExpr *E,
3003:                                               ReturnValueSlot ReturnValue,
3004:                                               llvm::Triple::ArchType Arch) {
3005:   enum class CustomCodeGen { VLD24, VST24 } CustomCodeGenType;
3006:   Intrinsic::ID IRIntr;
3007:   unsigned NumVectors;
3008: 
3009:   // Code autogenerated by Tablegen will handle all the simple builtins.
3010:   switch (BuiltinID) {
3011:     #include "clang/Basic/arm_mve_builtin_cg.inc"
3012: 
3013:     // If we didn't match an MVE builtin id at all, go back to the
3014:     // main EmitARMBuiltinExpr.
3015:   default:
3016:     return nullptr;
3017:   }
3018: 
3019:   // Anything that breaks from that switch is an MVE builtin that
3020:   // needs handwritten code to generate.
3021: 
3022:   switch (CustomCodeGenType) {
3023: 
3024:   case CustomCodeGen::VLD24: {
3025:     llvm::SmallVector<Value *, 4> Ops;
3026:     llvm::SmallVector<llvm::Type *, 4> Tys;
3027: 
3028:     auto MvecCType = E->getType();
3029:     auto MvecLType = ConvertType(MvecCType);
3030:     assert(MvecLType->isStructTy() &&
```
- **EN**: This block imports Clang headers `clang/Basic/arm_mve_builtin_cg.inc`; introduces declarations such as `CustomCodeGen`; uses control flow (switch, case) to specialize target builtin lowering; guards important invariants with assertions or unreachable markers; contains preprocessor structure that shapes compilation of this unit.
- **CN**: 该代码块引入 Clang 头文件 `clang/Basic/arm_mve_builtin_cg.inc`；给出诸如 `CustomCodeGen` 的声明；通过控制流（switch, case）细化 目标内建函数降级 行为；使用断言或不可达标记保护关键不变量；包含影响本编译单元构建方式的预处理结构。

### Lines 3031-3060
```cpp
3031:            "Return type for vld[24]q should be a struct");
3032:     assert(MvecLType->getStructNumElements() == 1 &&
3033:            "Return-type struct for vld[24]q should have one element");
3034:     auto MvecLTypeInner = MvecLType->getStructElementType(0);
3035:     assert(MvecLTypeInner->isArrayTy() &&
3036:            "Return-type struct for vld[24]q should contain an array");
3037:     assert(MvecLTypeInner->getArrayNumElements() == NumVectors &&
3038:            "Array member of return-type struct vld[24]q has wrong length");
3039:     auto VecLType = MvecLTypeInner->getArrayElementType();
3040: 
3041:     Tys.push_back(VecLType);
3042: 
3043:     auto Addr = E->getArg(0);
3044:     Ops.push_back(EmitScalarExpr(Addr));
3045:     Tys.push_back(ConvertType(Addr->getType()));
3046: 
3047:     Function *F = CGM.getIntrinsic(IRIntr, ArrayRef(Tys));
3048:     Value *LoadResult = Builder.CreateCall(F, Ops);
3049:     Value *MvecOut = PoisonValue::get(MvecLType);
3050:     for (unsigned i = 0; i < NumVectors; ++i) {
3051:       Value *Vec = Builder.CreateExtractValue(LoadResult, i);
3052:       MvecOut = Builder.CreateInsertValue(MvecOut, Vec, {0, i});
3053:     }
3054: 
3055:     if (ReturnValue.isNull())
3056:       return MvecOut;
3057:     else
3058:       return Builder.CreateStore(MvecOut, ReturnValue.getAddress());
3059:   }
3060: 
```
- **EN**: This block introduces declarations such as `for`, `vld`; uses control flow (if, for) to specialize target builtin lowering; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块给出诸如 `for`, `vld` 的声明；通过控制流（if, for）细化 目标内建函数降级 行为；使用断言或不可达标记保护关键不变量。

### Lines 3061-3090
```cpp
3061:   case CustomCodeGen::VST24: {
3062:     llvm::SmallVector<Value *, 4> Ops;
3063:     llvm::SmallVector<llvm::Type *, 4> Tys;
3064: 
3065:     auto Addr = E->getArg(0);
3066:     Ops.push_back(EmitScalarExpr(Addr));
3067:     Tys.push_back(ConvertType(Addr->getType()));
3068: 
3069:     auto MvecCType = E->getArg(1)->getType();
3070:     auto MvecLType = ConvertType(MvecCType);
3071:     assert(MvecLType->isStructTy() && "Data type for vst2q should be a struct");
3072:     assert(MvecLType->getStructNumElements() == 1 &&
3073:            "Data-type struct for vst2q should have one element");
3074:     auto MvecLTypeInner = MvecLType->getStructElementType(0);
3075:     assert(MvecLTypeInner->isArrayTy() &&
3076:            "Data-type struct for vst2q should contain an array");
3077:     assert(MvecLTypeInner->getArrayNumElements() == NumVectors &&
3078:            "Array member of return-type struct vld[24]q has wrong length");
3079:     auto VecLType = MvecLTypeInner->getArrayElementType();
3080: 
3081:     Tys.push_back(VecLType);
3082: 
3083:     AggValueSlot MvecSlot = CreateAggTemp(MvecCType);
3084:     EmitAggExpr(E->getArg(1), MvecSlot);
3085:     auto Mvec = Builder.CreateLoad(MvecSlot.getAddress());
3086:     for (unsigned i = 0; i < NumVectors; i++)
3087:       Ops.push_back(Builder.CreateExtractValue(Mvec, {0, i}));
3088: 
3089:     Function *F = CGM.getIntrinsic(IRIntr, ArrayRef(Tys));
3090:     Value *ToReturn = nullptr;
```
- **EN**: This block introduces declarations such as `for`, `vld`; defines callable entry points like `EmitAggExpr`; uses control flow (for, case) to specialize target builtin lowering; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块给出诸如 `for`, `vld` 的声明；定义可调用入口，例如 `EmitAggExpr`；通过控制流（for, case）细化 目标内建函数降级 行为；使用断言或不可达标记保护关键不变量。

### Lines 3091-3120
```cpp
3091:     for (unsigned i = 0; i < NumVectors; i++) {
3092:       Ops.push_back(llvm::ConstantInt::get(Int32Ty, i));
3093:       ToReturn = Builder.CreateCall(F, Ops);
3094:       Ops.pop_back();
3095:     }
3096:     return ToReturn;
3097:   }
3098:   }
3099:   llvm_unreachable("unknown custom codegen type.");
3100: }
3101: 
3102: Value *CodeGenFunction::EmitARMCDEBuiltinExpr(unsigned BuiltinID,
3103:                                               const CallExpr *E,
3104:                                               ReturnValueSlot ReturnValue,
3105:                                               llvm::Triple::ArchType Arch) {
3106:   switch (BuiltinID) {
3107:   default:
3108:     return nullptr;
3109: #include "clang/Basic/arm_cde_builtin_cg.inc"
3110:   }
3111: }
3112: 
3113: static Value *EmitAArch64TblBuiltinExpr(CodeGenFunction &CGF, unsigned BuiltinID,
3114:                                       const CallExpr *E,
3115:                                       SmallVectorImpl<Value *> &Ops,
3116:                                       llvm::Triple::ArchType Arch) {
3117:   unsigned int Int = 0;
3118:   const char *s = nullptr;
3119: 
3120:   switch (BuiltinID) {
```
- **EN**: This block imports Clang headers `clang/Basic/arm_cde_builtin_cg.inc`; uses control flow (switch, for) to specialize target builtin lowering; guards important invariants with assertions or unreachable markers; contains preprocessor structure that shapes compilation of this unit.
- **CN**: 该代码块引入 Clang 头文件 `clang/Basic/arm_cde_builtin_cg.inc`；通过控制流（switch, for）细化 目标内建函数降级 行为；使用断言或不可达标记保护关键不变量；包含影响本编译单元构建方式的预处理结构。

### Lines 3121-3150
```cpp
3121:   default:
3122:     return nullptr;
3123:   case NEON::BI__builtin_neon_vtbl1_v:
3124:   case NEON::BI__builtin_neon_vqtbl1_v:
3125:   case NEON::BI__builtin_neon_vqtbl1q_v:
3126:   case NEON::BI__builtin_neon_vtbl2_v:
3127:   case NEON::BI__builtin_neon_vqtbl2_v:
3128:   case NEON::BI__builtin_neon_vqtbl2q_v:
3129:   case NEON::BI__builtin_neon_vtbl3_v:
3130:   case NEON::BI__builtin_neon_vqtbl3_v:
3131:   case NEON::BI__builtin_neon_vqtbl3q_v:
3132:   case NEON::BI__builtin_neon_vtbl4_v:
3133:   case NEON::BI__builtin_neon_vqtbl4_v:
3134:   case NEON::BI__builtin_neon_vqtbl4q_v:
3135:     break;
3136:   case NEON::BI__builtin_neon_vtbx1_v:
3137:   case NEON::BI__builtin_neon_vqtbx1_v:
3138:   case NEON::BI__builtin_neon_vqtbx1q_v:
3139:   case NEON::BI__builtin_neon_vtbx2_v:
3140:   case NEON::BI__builtin_neon_vqtbx2_v:
3141:   case NEON::BI__builtin_neon_vqtbx2q_v:
3142:   case NEON::BI__builtin_neon_vtbx3_v:
3143:   case NEON::BI__builtin_neon_vqtbx3_v:
3144:   case NEON::BI__builtin_neon_vqtbx3q_v:
3145:   case NEON::BI__builtin_neon_vtbx4_v:
3146:   case NEON::BI__builtin_neon_vqtbx4_v:
3147:   case NEON::BI__builtin_neon_vqtbx4q_v:
3148:     break;
3149:   }
3150: 
```
- **EN**: This block uses control flow (case) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（case）细化 目标内建函数降级 行为。

### Lines 3151-3180
```cpp
3151:   assert(E->getNumArgs() >= 3);
3152: 
3153:   // Get the last argument, which specifies the vector type.
3154:   const Expr *Arg = E->getArg(E->getNumArgs() - 1);
3155:   std::optional<llvm::APSInt> Result =
3156:       Arg->getIntegerConstantExpr(CGF.getContext());
3157:   if (!Result)
3158:     return nullptr;
3159: 
3160:   // Determine the type of this overloaded NEON intrinsic.
3161:   NeonTypeFlags Type = Result->getZExtValue();
3162:   llvm::FixedVectorType *Ty = GetNeonType(&CGF, Type);
3163:   if (!Ty)
3164:     return nullptr;
3165: 
3166:   CodeGen::CGBuilderTy &Builder = CGF.Builder;
3167: 
3168:   // AArch64 scalar builtins are not overloaded, they do not have an extra
3169:   // argument that specifies the vector type, need to handle each case.
3170:   switch (BuiltinID) {
3171:   case NEON::BI__builtin_neon_vtbl1_v: {
3172:     return packTBLDVectorList(CGF, ArrayRef(Ops).slice(0, 1), nullptr, Ops[1],
3173:                               Ty, Intrinsic::aarch64_neon_tbl1, "vtbl1");
3174:   }
3175:   case NEON::BI__builtin_neon_vtbl2_v: {
3176:     return packTBLDVectorList(CGF, ArrayRef(Ops).slice(0, 2), nullptr, Ops[2],
3177:                               Ty, Intrinsic::aarch64_neon_tbl1, "vtbl1");
3178:   }
3179:   case NEON::BI__builtin_neon_vtbl3_v: {
3180:     return packTBLDVectorList(CGF, ArrayRef(Ops).slice(0, 3), nullptr, Ops[3],
```
- **EN**: This block defines callable entry points like `packTBLDVectorList`; uses control flow (if, switch, case) to specialize target builtin lowering; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块定义可调用入口，例如 `packTBLDVectorList`；通过控制流（if, switch, case）细化 目标内建函数降级 行为；使用断言或不可达标记保护关键不变量。

### Lines 3181-3210
```cpp
3181:                               Ty, Intrinsic::aarch64_neon_tbl2, "vtbl2");
3182:   }
3183:   case NEON::BI__builtin_neon_vtbl4_v: {
3184:     return packTBLDVectorList(CGF, ArrayRef(Ops).slice(0, 4), nullptr, Ops[4],
3185:                               Ty, Intrinsic::aarch64_neon_tbl2, "vtbl2");
3186:   }
3187:   case NEON::BI__builtin_neon_vtbx1_v: {
3188:     Value *TblRes =
3189:         packTBLDVectorList(CGF, ArrayRef(Ops).slice(1, 1), nullptr, Ops[2], Ty,
3190:                            Intrinsic::aarch64_neon_tbl1, "vtbl1");
3191: 
3192:     llvm::Constant *EightV = ConstantInt::get(Ty, 8);
3193:     Value *CmpRes = Builder.CreateICmp(ICmpInst::ICMP_UGE, Ops[2], EightV);
3194:     CmpRes = Builder.CreateSExt(CmpRes, Ty);
3195: 
3196:     Value *EltsFromInput = Builder.CreateAnd(CmpRes, Ops[0]);
3197:     Value *EltsFromTbl = Builder.CreateAnd(Builder.CreateNot(CmpRes), TblRes);
3198:     return Builder.CreateOr(EltsFromInput, EltsFromTbl, "vtbx");
3199:   }
3200:   case NEON::BI__builtin_neon_vtbx2_v: {
3201:     return packTBLDVectorList(CGF, ArrayRef(Ops).slice(1, 2), Ops[0], Ops[3],
3202:                               Ty, Intrinsic::aarch64_neon_tbx1, "vtbx1");
3203:   }
3204:   case NEON::BI__builtin_neon_vtbx3_v: {
3205:     Value *TblRes =
3206:         packTBLDVectorList(CGF, ArrayRef(Ops).slice(1, 3), nullptr, Ops[4], Ty,
3207:                            Intrinsic::aarch64_neon_tbl2, "vtbl2");
3208: 
3209:     llvm::Constant *TwentyFourV = ConstantInt::get(Ty, 24);
3210:     Value *CmpRes = Builder.CreateICmp(ICmpInst::ICMP_UGE, Ops[4],
```
- **EN**: This block defines callable entry points like `packTBLDVectorList`; uses control flow (case) to specialize target builtin lowering.
- **CN**: 该代码块定义可调用入口，例如 `packTBLDVectorList`；通过控制流（case）细化 目标内建函数降级 行为。

### Lines 3211-3240
```cpp
3211:                                            TwentyFourV);
3212:     CmpRes = Builder.CreateSExt(CmpRes, Ty);
3213: 
3214:     Value *EltsFromInput = Builder.CreateAnd(CmpRes, Ops[0]);
3215:     Value *EltsFromTbl = Builder.CreateAnd(Builder.CreateNot(CmpRes), TblRes);
3216:     return Builder.CreateOr(EltsFromInput, EltsFromTbl, "vtbx");
3217:   }
3218:   case NEON::BI__builtin_neon_vtbx4_v: {
3219:     return packTBLDVectorList(CGF, ArrayRef(Ops).slice(1, 4), Ops[0], Ops[5],
3220:                               Ty, Intrinsic::aarch64_neon_tbx2, "vtbx2");
3221:   }
3222:   case NEON::BI__builtin_neon_vqtbl1_v:
3223:   case NEON::BI__builtin_neon_vqtbl1q_v:
3224:     Int = Intrinsic::aarch64_neon_tbl1; s = "vtbl1"; break;
3225:   case NEON::BI__builtin_neon_vqtbl2_v:
3226:   case NEON::BI__builtin_neon_vqtbl2q_v: {
3227:     Int = Intrinsic::aarch64_neon_tbl2; s = "vtbl2"; break;
3228:   case NEON::BI__builtin_neon_vqtbl3_v:
3229:   case NEON::BI__builtin_neon_vqtbl3q_v:
3230:     Int = Intrinsic::aarch64_neon_tbl3; s = "vtbl3"; break;
3231:   case NEON::BI__builtin_neon_vqtbl4_v:
3232:   case NEON::BI__builtin_neon_vqtbl4q_v:
3233:     Int = Intrinsic::aarch64_neon_tbl4; s = "vtbl4"; break;
3234:   case NEON::BI__builtin_neon_vqtbx1_v:
3235:   case NEON::BI__builtin_neon_vqtbx1q_v:
3236:     Int = Intrinsic::aarch64_neon_tbx1; s = "vtbx1"; break;
3237:   case NEON::BI__builtin_neon_vqtbx2_v:
3238:   case NEON::BI__builtin_neon_vqtbx2q_v:
3239:     Int = Intrinsic::aarch64_neon_tbx2; s = "vtbx2"; break;
3240:   case NEON::BI__builtin_neon_vqtbx3_v:
```
- **EN**: This block defines callable entry points like `packTBLDVectorList`; uses control flow (case) to specialize target builtin lowering.
- **CN**: 该代码块定义可调用入口，例如 `packTBLDVectorList`；通过控制流（case）细化 目标内建函数降级 行为。

### Lines 3241-3270
```cpp
3241:   case NEON::BI__builtin_neon_vqtbx3q_v:
3242:     Int = Intrinsic::aarch64_neon_tbx3; s = "vtbx3"; break;
3243:   case NEON::BI__builtin_neon_vqtbx4_v:
3244:   case NEON::BI__builtin_neon_vqtbx4q_v:
3245:     Int = Intrinsic::aarch64_neon_tbx4; s = "vtbx4"; break;
3246:   }
3247:   }
3248: 
3249:   if (!Int)
3250:     return nullptr;
3251: 
3252:   Function *F = CGF.CGM.getIntrinsic(Int, Ty);
3253:   return CGF.EmitNeonCall(F, Ops, s);
3254: }
3255: 
3256: Value *CodeGenFunction::vectorWrapScalar16(Value *Op) {
3257:   auto *VTy = llvm::FixedVectorType::get(Int16Ty, 4);
3258:   Op = Builder.CreateBitCast(Op, Int16Ty);
3259:   Value *V = PoisonValue::get(VTy);
3260:   llvm::Constant *CI = ConstantInt::get(SizeTy, 0);
3261:   Op = Builder.CreateInsertElement(V, Op, CI);
3262:   return Op;
3263: }
3264: 
3265: /// SVEBuiltinMemEltTy - Returns the memory element type for this memory
3266: /// access builtin.  Only required if it can't be inferred from the base pointer
3267: /// operand.
3268: llvm::Type *CodeGenFunction::SVEBuiltinMemEltTy(const SVETypeFlags &TypeFlags) {
3269:   switch (TypeFlags.getMemEltType()) {
3270:   case SVETypeFlags::MemEltTyDefault:
```
- **EN**: This block uses control flow (if, switch, case) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（if, switch, case）细化 目标内建函数降级 行为。

### Lines 3271-3300
```cpp
3271:     return getEltType(TypeFlags);
3272:   case SVETypeFlags::MemEltTyInt8:
3273:     return Builder.getInt8Ty();
3274:   case SVETypeFlags::MemEltTyInt16:
3275:     return Builder.getInt16Ty();
3276:   case SVETypeFlags::MemEltTyInt32:
3277:     return Builder.getInt32Ty();
3278:   case SVETypeFlags::MemEltTyInt64:
3279:     return Builder.getInt64Ty();
3280:   }
3281:   llvm_unreachable("Unknown MemEltType");
3282: }
3283: 
3284: llvm::Type *CodeGenFunction::getEltType(const SVETypeFlags &TypeFlags) {
3285:   switch (TypeFlags.getEltType()) {
3286:   default:
3287:     llvm_unreachable("Invalid SVETypeFlag!");
3288: 
3289:   case SVETypeFlags::EltTyMFloat8:
3290:   case SVETypeFlags::EltTyInt8:
3291:     return Builder.getInt8Ty();
3292:   case SVETypeFlags::EltTyInt16:
3293:     return Builder.getInt16Ty();
3294:   case SVETypeFlags::EltTyInt32:
3295:     return Builder.getInt32Ty();
3296:   case SVETypeFlags::EltTyInt64:
3297:     return Builder.getInt64Ty();
3298:   case SVETypeFlags::EltTyInt128:
3299:     return Builder.getInt128Ty();
3300: 
```
- **EN**: This block defines callable entry points like `getEltType`; uses control flow (switch, case) to specialize target builtin lowering; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块定义可调用入口，例如 `getEltType`；通过控制流（switch, case）细化 目标内建函数降级 行为；使用断言或不可达标记保护关键不变量。

### Lines 3301-3330
```cpp
3301:   case SVETypeFlags::EltTyFloat16:
3302:     return Builder.getHalfTy();
3303:   case SVETypeFlags::EltTyFloat32:
3304:     return Builder.getFloatTy();
3305:   case SVETypeFlags::EltTyFloat64:
3306:     return Builder.getDoubleTy();
3307: 
3308:   case SVETypeFlags::EltTyBFloat16:
3309:     return Builder.getBFloatTy();
3310: 
3311:   case SVETypeFlags::EltTyBool8:
3312:   case SVETypeFlags::EltTyBool16:
3313:   case SVETypeFlags::EltTyBool32:
3314:   case SVETypeFlags::EltTyBool64:
3315:     return Builder.getInt1Ty();
3316:   }
3317: }
3318: 
3319: // Return the llvm predicate vector type corresponding to the specified element
3320: // TypeFlags.
3321: llvm::ScalableVectorType *
3322: CodeGenFunction::getSVEPredType(const SVETypeFlags &TypeFlags) {
3323:   switch (TypeFlags.getEltType()) {
3324:   default: llvm_unreachable("Unhandled SVETypeFlag!");
3325: 
3326:   case SVETypeFlags::EltTyInt8:
3327:     return llvm::ScalableVectorType::get(Builder.getInt1Ty(), 16);
3328:   case SVETypeFlags::EltTyInt16:
3329:     return llvm::ScalableVectorType::get(Builder.getInt1Ty(), 8);
3330:   case SVETypeFlags::EltTyInt32:
```
- **EN**: This block defines callable entry points like `getSVEPredType`, `get`; uses control flow (switch, case) to specialize target builtin lowering; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块定义可调用入口，例如 `getSVEPredType`, `get`；通过控制流（switch, case）细化 目标内建函数降级 行为；使用断言或不可达标记保护关键不变量。

### Lines 3331-3360
```cpp
3331:     return llvm::ScalableVectorType::get(Builder.getInt1Ty(), 4);
3332:   case SVETypeFlags::EltTyInt64:
3333:     return llvm::ScalableVectorType::get(Builder.getInt1Ty(), 2);
3334: 
3335:   case SVETypeFlags::EltTyBFloat16:
3336:     return llvm::ScalableVectorType::get(Builder.getInt1Ty(), 8);
3337:   case SVETypeFlags::EltTyFloat16:
3338:     return llvm::ScalableVectorType::get(Builder.getInt1Ty(), 8);
3339:   case SVETypeFlags::EltTyFloat32:
3340:     return llvm::ScalableVectorType::get(Builder.getInt1Ty(), 4);
3341:   case SVETypeFlags::EltTyFloat64:
3342:     return llvm::ScalableVectorType::get(Builder.getInt1Ty(), 2);
3343: 
3344:   case SVETypeFlags::EltTyBool8:
3345:     return llvm::ScalableVectorType::get(Builder.getInt1Ty(), 16);
3346:   case SVETypeFlags::EltTyBool16:
3347:     return llvm::ScalableVectorType::get(Builder.getInt1Ty(), 8);
3348:   case SVETypeFlags::EltTyBool32:
3349:     return llvm::ScalableVectorType::get(Builder.getInt1Ty(), 4);
3350:   case SVETypeFlags::EltTyBool64:
3351:     return llvm::ScalableVectorType::get(Builder.getInt1Ty(), 2);
3352:   }
3353: }
3354: 
3355: // Return the llvm vector type corresponding to the specified element TypeFlags.
3356: llvm::ScalableVectorType *
3357: CodeGenFunction::getSVEType(const SVETypeFlags &TypeFlags) {
3358:   switch (TypeFlags.getEltType()) {
3359:   default:
3360:     llvm_unreachable("Invalid SVETypeFlag!");
```
- **EN**: This block defines callable entry points like `get`, `getSVEType`; uses control flow (switch, case) to specialize target builtin lowering; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块定义可调用入口，例如 `get`, `getSVEType`；通过控制流（switch, case）细化 目标内建函数降级 行为；使用断言或不可达标记保护关键不变量。

### Lines 3361-3390
```cpp
3361: 
3362:   case SVETypeFlags::EltTyInt8:
3363:     return llvm::ScalableVectorType::get(Builder.getInt8Ty(), 16);
3364:   case SVETypeFlags::EltTyInt16:
3365:     return llvm::ScalableVectorType::get(Builder.getInt16Ty(), 8);
3366:   case SVETypeFlags::EltTyInt32:
3367:     return llvm::ScalableVectorType::get(Builder.getInt32Ty(), 4);
3368:   case SVETypeFlags::EltTyInt64:
3369:     return llvm::ScalableVectorType::get(Builder.getInt64Ty(), 2);
3370: 
3371:   case SVETypeFlags::EltTyMFloat8:
3372:     return llvm::ScalableVectorType::get(Builder.getInt8Ty(), 16);
3373:   case SVETypeFlags::EltTyFloat16:
3374:     return llvm::ScalableVectorType::get(Builder.getHalfTy(), 8);
3375:   case SVETypeFlags::EltTyBFloat16:
3376:     return llvm::ScalableVectorType::get(Builder.getBFloatTy(), 8);
3377:   case SVETypeFlags::EltTyFloat32:
3378:     return llvm::ScalableVectorType::get(Builder.getFloatTy(), 4);
3379:   case SVETypeFlags::EltTyFloat64:
3380:     return llvm::ScalableVectorType::get(Builder.getDoubleTy(), 2);
3381: 
3382:   case SVETypeFlags::EltTyBool8:
3383:     return llvm::ScalableVectorType::get(Builder.getInt1Ty(), 16);
3384:   case SVETypeFlags::EltTyBool16:
3385:     return llvm::ScalableVectorType::get(Builder.getInt1Ty(), 8);
3386:   case SVETypeFlags::EltTyBool32:
3387:     return llvm::ScalableVectorType::get(Builder.getInt1Ty(), 4);
3388:   case SVETypeFlags::EltTyBool64:
3389:     return llvm::ScalableVectorType::get(Builder.getInt1Ty(), 2);
3390:   }
```
- **EN**: This block spells out callable entry points like `get`; uses control flow (case) to specialize target builtin lowering.
- **CN**: 该代码块给出可调用入口的声明，例如 `get`；通过控制流（case）细化 目标内建函数降级 行为。

### Lines 3391-3420
```cpp
3391: }
3392: 
3393: llvm::Value *
3394: CodeGenFunction::EmitSVEAllTruePred(const SVETypeFlags &TypeFlags) {
3395:   Function *Ptrue =
3396:       CGM.getIntrinsic(Intrinsic::aarch64_sve_ptrue, getSVEPredType(TypeFlags));
3397:   return Builder.CreateCall(Ptrue, {Builder.getInt32(/*SV_ALL*/ 31)});
3398: }
3399: 
3400: constexpr unsigned SVEBitsPerBlock = 128;
3401: 
3402: static llvm::ScalableVectorType *getSVEVectorForElementType(llvm::Type *EltTy) {
3403:   unsigned NumElts = SVEBitsPerBlock / EltTy->getScalarSizeInBits();
3404:   return llvm::ScalableVectorType::get(EltTy, NumElts);
3405: }
3406: 
3407: // Reinterpret the input predicate so that it can be used to correctly isolate
3408: // the elements of the specified datatype.
3409: Value *CodeGenFunction::EmitSVEPredicateCast(Value *Pred,
3410:                                              llvm::ScalableVectorType *VTy) {
3411: 
3412:   if (isa<TargetExtType>(Pred->getType()) &&
3413:       cast<TargetExtType>(Pred->getType())->getName() == "aarch64.svcount")
3414:     return Pred;
3415: 
3416:   auto *RTy = llvm::VectorType::get(IntegerType::get(getLLVMContext(), 1), VTy);
3417:   if (Pred->getType() == RTy)
3418:     return Pred;
3419: 
3420:   unsigned IntID;
```
- **EN**: This block defines callable entry points like `EmitSVEAllTruePred`, `get`; uses control flow (if) to specialize target builtin lowering.
- **CN**: 该代码块定义可调用入口，例如 `EmitSVEAllTruePred`, `get`；通过控制流（if）细化 目标内建函数降级 行为。

### Lines 3421-3450
```cpp
3421:   llvm::Type *IntrinsicTy;
3422:   switch (VTy->getMinNumElements()) {
3423:   default:
3424:     llvm_unreachable("unsupported element count!");
3425:   case 1:
3426:   case 2:
3427:   case 4:
3428:   case 8:
3429:     IntID = Intrinsic::aarch64_sve_convert_from_svbool;
3430:     IntrinsicTy = RTy;
3431:     break;
3432:   case 16:
3433:     IntID = Intrinsic::aarch64_sve_convert_to_svbool;
3434:     IntrinsicTy = Pred->getType();
3435:     break;
3436:   }
3437: 
3438:   Function *F = CGM.getIntrinsic(IntID, IntrinsicTy);
3439:   Value *C = Builder.CreateCall(F, Pred);
3440:   assert(C->getType() == RTy && "Unexpected return type!");
3441:   return C;
3442: }
3443: 
3444: Value *CodeGenFunction::EmitSVEPredicateTupleCast(Value *PredTuple,
3445:                                                   llvm::StructType *Ty) {
3446:   if (PredTuple->getType() == Ty)
3447:     return PredTuple;
3448: 
3449:   Value *Ret = llvm::PoisonValue::get(Ty);
3450:   for (unsigned I = 0; I < Ty->getNumElements(); ++I) {
```
- **EN**: This block uses control flow (if, switch, for, case) to specialize target builtin lowering; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块通过控制流（if, switch, for, case）细化 目标内建函数降级 行为；使用断言或不可达标记保护关键不变量。

### Lines 3451-3480
```cpp
3451:     Value *Pred = Builder.CreateExtractValue(PredTuple, I);
3452:     Pred = EmitSVEPredicateCast(
3453:         Pred, cast<llvm::ScalableVectorType>(Ty->getTypeAtIndex(I)));
3454:     Ret = Builder.CreateInsertValue(Ret, Pred, I);
3455:   }
3456: 
3457:   return Ret;
3458: }
3459: 
3460: Value *CodeGenFunction::EmitSVEGatherLoad(const SVETypeFlags &TypeFlags,
3461:                                           SmallVectorImpl<Value *> &Ops,
3462:                                           unsigned IntID) {
3463:   auto *ResultTy = getSVEType(TypeFlags);
3464:   auto *OverloadedTy =
3465:       llvm::ScalableVectorType::get(SVEBuiltinMemEltTy(TypeFlags), ResultTy);
3466:   Function *F = CGM.getIntrinsic(IntID, {OverloadedTy, Ops[1]->getType()});
3467: 
3468:   // At the ACLE level there's only one predicate type, svbool_t, which is
3469:   // mapped to <n x 16 x i1>. However, this might be incompatible with the
3470:   // actual type being loaded. For example, when loading doubles (i64) the
3471:   // predicate should be <n x 2 x i1> instead. At the IR level the type of
3472:   // the predicate and the data being loaded must match. Cast to the type
3473:   // expected by the intrinsic. The intrinsic itself should be defined in
3474:   // a way than enforces relations between parameter types.
3475:   Ops[0] = EmitSVEPredicateCast(
3476:       Ops[0], cast<llvm::ScalableVectorType>(F->getArg(0)->getType()));
3477: 
3478:   // Pass 0 when the offset is missing. This can only be applied when using
3479:   // the "vector base" addressing mode for which ACLE allows no offset. The
3480:   // corresponding LLVM IR always requires an offset.
```
- **EN**: This block defines callable entry points like `get`; returns or forwards computed values for the surrounding target builtin lowering logic.
- **CN**: 该代码块定义可调用入口，例如 `get`；为周围的 目标内建函数降级 逻辑返回或转发计算结果。

### Lines 3481-3510
```cpp
3481:   if (Ops.size() == 2) {
3482:     assert(Ops[1]->getType()->isVectorTy() && "Scalar base requires an offset");
3483:     Ops.push_back(ConstantInt::get(Int64Ty, 0));
3484:   }
3485: 
3486:   // For "vector base, scalar index" scale the index so that it becomes a
3487:   // scalar offset.
3488:   if (!TypeFlags.isByteIndexed() && Ops[1]->getType()->isVectorTy()) {
3489:     unsigned BytesPerElt =
3490:         OverloadedTy->getElementType()->getScalarSizeInBits() / 8;
3491:     Ops[2] = Builder.CreateShl(Ops[2], Log2_32(BytesPerElt));
3492:   }
3493: 
3494:   Value *Call = Builder.CreateCall(F, Ops);
3495: 
3496:   // The following sext/zext is only needed when ResultTy != OverloadedTy. In
3497:   // other cases it's folded into a nop.
3498:   return TypeFlags.isZExtReturn() ? Builder.CreateZExt(Call, ResultTy)
3499:                                   : Builder.CreateSExt(Call, ResultTy);
3500: }
3501: 
3502: Value *CodeGenFunction::EmitSVEScatterStore(const SVETypeFlags &TypeFlags,
3503:                                             SmallVectorImpl<Value *> &Ops,
3504:                                             unsigned IntID) {
3505:   auto *SrcDataTy = getSVEType(TypeFlags);
3506:   auto *OverloadedTy =
3507:       llvm::ScalableVectorType::get(SVEBuiltinMemEltTy(TypeFlags), SrcDataTy);
3508: 
3509:   // In ACLE the source data is passed in the last argument, whereas in LLVM IR
3510:   // it's the first argument. Move it accordingly.
```
- **EN**: This block defines callable entry points like `get`; uses control flow (if) to specialize target builtin lowering; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块定义可调用入口，例如 `get`；通过控制流（if）细化 目标内建函数降级 行为；使用断言或不可达标记保护关键不变量。

### Lines 3511-3540
```cpp
3511:   Ops.insert(Ops.begin(), Ops.pop_back_val());
3512: 
3513:   Function *F = CGM.getIntrinsic(IntID, {OverloadedTy, Ops[2]->getType()});
3514: 
3515:   // Pass 0 when the offset is missing. This can only be applied when using
3516:   // the "vector base" addressing mode for which ACLE allows no offset. The
3517:   // corresponding LLVM IR always requires an offset.
3518:   if (Ops.size() == 3) {
3519:     assert(Ops[1]->getType()->isVectorTy() && "Scalar base requires an offset");
3520:     Ops.push_back(ConstantInt::get(Int64Ty, 0));
3521:   }
3522: 
3523:   // Truncation is needed when SrcDataTy != OverloadedTy. In other cases it's
3524:   // folded into a nop.
3525:   Ops[0] = Builder.CreateTrunc(Ops[0], OverloadedTy);
3526: 
3527:   // At the ACLE level there's only one predicate type, svbool_t, which is
3528:   // mapped to <n x 16 x i1>. However, this might be incompatible with the
3529:   // actual type being stored. For example, when storing doubles (i64) the
3530:   // predicated should be <n x 2 x i1> instead. At the IR level the type of
3531:   // the predicate and the data being stored must match. Cast to the type
3532:   // expected by the intrinsic. The intrinsic itself should be defined in
3533:   // a way that enforces relations between parameter types.
3534:   Ops[1] = EmitSVEPredicateCast(
3535:       Ops[1], cast<llvm::ScalableVectorType>(F->getArg(1)->getType()));
3536: 
3537:   // For "vector base, scalar index" scale the index so that it becomes a
3538:   // scalar offset.
3539:   if (!TypeFlags.isByteIndexed() && Ops[2]->getType()->isVectorTy()) {
3540:     unsigned BytesPerElt =
```
- **EN**: This block uses control flow (if) to specialize target builtin lowering; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块通过控制流（if）细化 目标内建函数降级 行为；使用断言或不可达标记保护关键不变量。

### Lines 3541-3570
```cpp
3541:         OverloadedTy->getElementType()->getScalarSizeInBits() / 8;
3542:     Ops[3] = Builder.CreateShl(Ops[3], Log2_32(BytesPerElt));
3543:   }
3544: 
3545:   return Builder.CreateCall(F, Ops);
3546: }
3547: 
3548: Value *CodeGenFunction::EmitSVEGatherPrefetch(const SVETypeFlags &TypeFlags,
3549:                                               SmallVectorImpl<Value *> &Ops,
3550:                                               unsigned IntID) {
3551:   // The gather prefetches are overloaded on the vector input - this can either
3552:   // be the vector of base addresses or vector of offsets.
3553:   auto *OverloadedTy = dyn_cast<llvm::ScalableVectorType>(Ops[1]->getType());
3554:   if (!OverloadedTy)
3555:     OverloadedTy = cast<llvm::ScalableVectorType>(Ops[2]->getType());
3556: 
3557:   // Cast the predicate from svbool_t to the right number of elements.
3558:   Ops[0] = EmitSVEPredicateCast(Ops[0], OverloadedTy);
3559: 
3560:   // vector + imm addressing modes
3561:   if (Ops[1]->getType()->isVectorTy()) {
3562:     if (Ops.size() == 3) {
3563:       // Pass 0 for 'vector+imm' when the index is omitted.
3564:       Ops.push_back(ConstantInt::get(Int64Ty, 0));
3565: 
3566:       // The sv_prfop is the last operand in the builtin and IR intrinsic.
3567:       std::swap(Ops[2], Ops[3]);
3568:     } else {
3569:       // Index needs to be passed as scaled offset.
3570:       llvm::Type *MemEltTy = SVEBuiltinMemEltTy(TypeFlags);
```
- **EN**: This block defines callable entry points like `swap`; uses control flow (if) to specialize target builtin lowering.
- **CN**: 该代码块定义可调用入口，例如 `swap`；通过控制流（if）细化 目标内建函数降级 行为。

### Lines 3571-3600
```cpp
3571:       unsigned BytesPerElt = MemEltTy->getPrimitiveSizeInBits() / 8;
3572:       if (BytesPerElt > 1)
3573:         Ops[2] = Builder.CreateShl(Ops[2], Log2_32(BytesPerElt));
3574:     }
3575: 
3576:     Function *F = CGM.getIntrinsic(IntID, OverloadedTy);
3577:     return Builder.CreateCall(F, Ops);
3578:   }
3579: 
3580:   Function *F = CGM.getIntrinsic(IntID, {Ops[1]->getType(), OverloadedTy});
3581:   return Builder.CreateCall(F, Ops);
3582: }
3583: 
3584: Value *CodeGenFunction::EmitSVEStructLoad(const SVETypeFlags &TypeFlags,
3585:                                           SmallVectorImpl<Value*> &Ops,
3586:                                           unsigned IntID) {
3587:   llvm::ScalableVectorType *VTy = getSVEType(TypeFlags);
3588:   Value *Predicate = EmitSVEPredicateCast(Ops[0], VTy);
3589:   Value *BasePtr = Ops[1];
3590: 
3591:   // Does the load have an offset?
3592:   if (Ops.size() > 2)
3593:     BasePtr = Builder.CreateGEP(VTy, BasePtr, Ops[2]);
3594: 
3595:   Function *F = CGM.getIntrinsic(IntID, {VTy, BasePtr->getType()});
3596:   return Builder.CreateCall(F, {Predicate, BasePtr});
3597: }
3598: 
3599: Value *CodeGenFunction::EmitSVEStructStore(const SVETypeFlags &TypeFlags,
3600:                                            SmallVectorImpl<Value*> &Ops,
```
- **EN**: This block uses control flow (if) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（if）细化 目标内建函数降级 行为。

### Lines 3601-3630
```cpp
3601:                                            unsigned IntID) {
3602:   llvm::ScalableVectorType *VTy = getSVEType(TypeFlags);
3603: 
3604:   unsigned N;
3605:   switch (IntID) {
3606:   case Intrinsic::aarch64_sve_st2:
3607:   case Intrinsic::aarch64_sve_st1_pn_x2:
3608:   case Intrinsic::aarch64_sve_stnt1_pn_x2:
3609:   case Intrinsic::aarch64_sve_st2q:
3610:     N = 2;
3611:     break;
3612:   case Intrinsic::aarch64_sve_st3:
3613:   case Intrinsic::aarch64_sve_st3q:
3614:     N = 3;
3615:     break;
3616:   case Intrinsic::aarch64_sve_st4:
3617:   case Intrinsic::aarch64_sve_st1_pn_x4:
3618:   case Intrinsic::aarch64_sve_stnt1_pn_x4:
3619:   case Intrinsic::aarch64_sve_st4q:
3620:     N = 4;
3621:     break;
3622:   default:
3623:     llvm_unreachable("unknown intrinsic!");
3624:   }
3625: 
3626:   Value *Predicate = EmitSVEPredicateCast(Ops[0], VTy);
3627:   Value *BasePtr = Ops[1];
3628: 
3629:   // Does the store have an offset?
3630:   if (Ops.size() > (2 + N))
```
- **EN**: This block uses control flow (if, switch, case) to specialize target builtin lowering; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块通过控制流（if, switch, case）细化 目标内建函数降级 行为；使用断言或不可达标记保护关键不变量。

### Lines 3631-3660
```cpp
3631:     BasePtr = Builder.CreateGEP(VTy, BasePtr, Ops[2]);
3632: 
3633:   // The llvm.aarch64.sve.st2/3/4 intrinsics take legal part vectors, so we
3634:   // need to break up the tuple vector.
3635:   SmallVector<llvm::Value*, 5> Operands;
3636:   for (unsigned I = Ops.size() - N; I < Ops.size(); ++I)
3637:     Operands.push_back(Ops[I]);
3638:   Operands.append({Predicate, BasePtr});
3639:   Function *F = CGM.getIntrinsic(IntID, {VTy, BasePtr->getType()});
3640: 
3641:   return Builder.CreateCall(F, Operands);
3642: }
3643: 
3644: // SVE2's svpmullb and svpmullt builtins are similar to the svpmullb_pair and
3645: // svpmullt_pair intrinsics, with the exception that their results are bitcast
3646: // to a wider type.
3647: Value *CodeGenFunction::EmitSVEPMull(const SVETypeFlags &TypeFlags,
3648:                                      SmallVectorImpl<Value *> &Ops,
3649:                                      unsigned BuiltinID) {
3650:   // Splat scalar operand to vector (intrinsics with _n infix)
3651:   if (TypeFlags.hasSplatOperand()) {
3652:     unsigned OpNo = TypeFlags.getSplatOperand();
3653:     Ops[OpNo] = EmitSVEDupX(Ops[OpNo]);
3654:   }
3655: 
3656:   // The pair-wise function has a narrower overloaded type.
3657:   Function *F = CGM.getIntrinsic(BuiltinID, Ops[0]->getType());
3658:   Value *Call = Builder.CreateCall(F, {Ops[0], Ops[1]});
3659: 
3660:   // Now bitcast to the wider result type.
```
- **EN**: This block uses control flow (if, for) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（if, for）细化 目标内建函数降级 行为。

### Lines 3661-3690
```cpp
3661:   llvm::ScalableVectorType *Ty = getSVEType(TypeFlags);
3662:   return EmitSVEReinterpret(Call, Ty);
3663: }
3664: 
3665: Value *CodeGenFunction::EmitSVEMovl(const SVETypeFlags &TypeFlags,
3666:                                     ArrayRef<Value *> Ops, unsigned BuiltinID) {
3667:   llvm::Type *OverloadedTy = getSVEType(TypeFlags);
3668:   Function *F = CGM.getIntrinsic(BuiltinID, OverloadedTy);
3669:   return Builder.CreateCall(F, {Ops[0], Builder.getInt32(0)});
3670: }
3671: 
3672: Value *CodeGenFunction::EmitSVEPrefetchLoad(const SVETypeFlags &TypeFlags,
3673:                                             SmallVectorImpl<Value *> &Ops,
3674:                                             unsigned BuiltinID) {
3675:   auto *MemEltTy = SVEBuiltinMemEltTy(TypeFlags);
3676:   auto *VectorTy = getSVEVectorForElementType(MemEltTy);
3677:   auto *MemoryTy = llvm::ScalableVectorType::get(MemEltTy, VectorTy);
3678: 
3679:   Value *Predicate = EmitSVEPredicateCast(Ops[0], MemoryTy);
3680:   Value *BasePtr = Ops[1];
3681: 
3682:   // Implement the index operand if not omitted.
3683:   if (Ops.size() > 3)
3684:     BasePtr = Builder.CreateGEP(MemoryTy, BasePtr, Ops[2]);
3685: 
3686:   Value *PrfOp = Ops.back();
3687: 
3688:   llvm::Type *Tys[2] = {Predicate->getType(), BasePtr->getType()};
3689:   Function *F = CGM.getIntrinsic(BuiltinID, Tys);
3690:   return Builder.CreateCall(F, {Predicate, BasePtr, PrfOp});
```
- **EN**: This block defines callable entry points like `EmitSVEReinterpret`; uses control flow (if) to specialize target builtin lowering.
- **CN**: 该代码块定义可调用入口，例如 `EmitSVEReinterpret`；通过控制流（if）细化 目标内建函数降级 行为。

### Lines 3691-3720
```cpp
3691: }
3692: 
3693: Value *CodeGenFunction::EmitSVEMaskedLoad(const CallExpr *E,
3694:                                           llvm::Type *ReturnTy,
3695:                                           SmallVectorImpl<Value *> &Ops,
3696:                                           unsigned IntrinsicID,
3697:                                           bool IsZExtReturn) {
3698:   QualType LangPTy = E->getArg(1)->getType();
3699:   llvm::Type *MemEltTy = CGM.getTypes().ConvertType(
3700:       LangPTy->castAs<PointerType>()->getPointeeType());
3701: 
3702:   // Mfloat8 types is stored as a vector, so extra work
3703:   // to extract sclar element type is necessary.
3704:   if (MemEltTy->isVectorTy()) {
3705:     assert(MemEltTy == FixedVectorType::get(Int8Ty, 1) &&
3706:            "Only <1 x i8> expected");
3707:     MemEltTy = cast<llvm::VectorType>(MemEltTy)->getElementType();
3708:   }
3709: 
3710:   // The vector type that is returned may be different from the
3711:   // eventual type loaded from memory.
3712:   auto VectorTy = cast<llvm::ScalableVectorType>(ReturnTy);
3713:   llvm::ScalableVectorType *MemoryTy = nullptr;
3714:   llvm::ScalableVectorType *PredTy = nullptr;
3715:   bool IsQuadLoad = false;
3716:   switch (IntrinsicID) {
3717:   case Intrinsic::aarch64_sve_ld1uwq:
3718:   case Intrinsic::aarch64_sve_ld1udq:
3719:     MemoryTy = llvm::ScalableVectorType::get(MemEltTy, 1);
3720:     PredTy = llvm::ScalableVectorType::get(
```
- **EN**: This block uses control flow (if, switch, case) to specialize target builtin lowering; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块通过控制流（if, switch, case）细化 目标内建函数降级 行为；使用断言或不可达标记保护关键不变量。

### Lines 3721-3750
```cpp
3721:         llvm::Type::getInt1Ty(getLLVMContext()), 1);
3722:     IsQuadLoad = true;
3723:     break;
3724:   default:
3725:     MemoryTy = llvm::ScalableVectorType::get(MemEltTy, VectorTy);
3726:     PredTy = MemoryTy;
3727:     break;
3728:   }
3729: 
3730:   Value *Predicate = EmitSVEPredicateCast(Ops[0], PredTy);
3731:   Value *BasePtr = Ops[1];
3732: 
3733:   // Does the load have an offset?
3734:   if (Ops.size() > 2)
3735:     BasePtr = Builder.CreateGEP(MemoryTy, BasePtr, Ops[2]);
3736: 
3737:   llvm::Type *Tys[2] = {IsQuadLoad ? VectorTy : MemoryTy, BasePtr->getType()};
3738:   Function *F = CGM.getIntrinsic(IntrinsicID, Tys);
3739:   auto *Load = Builder.CreateCall(F, {Predicate, BasePtr});
3740:   auto TBAAInfo = CGM.getTBAAAccessInfo(LangPTy->getPointeeType());
3741:   CGM.DecorateInstructionWithTBAA(Load, TBAAInfo);
3742: 
3743:   if (IsQuadLoad)
3744:     return Load;
3745: 
3746:   return IsZExtReturn ? Builder.CreateZExt(Load, VectorTy)
3747:                       : Builder.CreateSExt(Load, VectorTy);
3748: }
3749: 
3750: Value *CodeGenFunction::EmitSVEMaskedStore(const CallExpr *E,
```
- **EN**: This block defines callable entry points like `getInt1Ty`; uses control flow (if) to specialize target builtin lowering.
- **CN**: 该代码块定义可调用入口，例如 `getInt1Ty`；通过控制流（if）细化 目标内建函数降级 行为。

### Lines 3751-3780
```cpp
3751:                                            SmallVectorImpl<Value *> &Ops,
3752:                                            unsigned IntrinsicID) {
3753:   QualType LangPTy = E->getArg(1)->getType();
3754:   llvm::Type *MemEltTy = CGM.getTypes().ConvertType(
3755:       LangPTy->castAs<PointerType>()->getPointeeType());
3756: 
3757:   // Mfloat8 types is stored as a vector, so extra work
3758:   // to extract sclar element type is necessary.
3759:   if (MemEltTy->isVectorTy()) {
3760:     assert(MemEltTy == FixedVectorType::get(Int8Ty, 1) &&
3761:            "Only <1 x i8> expected");
3762:     MemEltTy = cast<llvm::VectorType>(MemEltTy)->getElementType();
3763:   }
3764: 
3765:   // The vector type that is stored may be different from the
3766:   // eventual type stored to memory.
3767:   auto VectorTy = cast<llvm::ScalableVectorType>(Ops.back()->getType());
3768:   auto MemoryTy = llvm::ScalableVectorType::get(MemEltTy, VectorTy);
3769: 
3770:   auto PredTy = MemoryTy;
3771:   auto AddrMemoryTy = MemoryTy;
3772:   bool IsQuadStore = false;
3773: 
3774:   switch (IntrinsicID) {
3775:   case Intrinsic::aarch64_sve_st1wq:
3776:   case Intrinsic::aarch64_sve_st1dq:
3777:     AddrMemoryTy = llvm::ScalableVectorType::get(MemEltTy, 1);
3778:     PredTy =
3779:         llvm::ScalableVectorType::get(IntegerType::get(getLLVMContext(), 1), 1);
3780:     IsQuadStore = true;
```
- **EN**: This block defines callable entry points like `get`; uses control flow (if, switch, case) to specialize target builtin lowering; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块定义可调用入口，例如 `get`；通过控制流（if, switch, case）细化 目标内建函数降级 行为；使用断言或不可达标记保护关键不变量。

### Lines 3781-3810
```cpp
3781:     break;
3782:   default:
3783:     break;
3784:   }
3785:   Value *Predicate = EmitSVEPredicateCast(Ops[0], PredTy);
3786:   Value *BasePtr = Ops[1];
3787: 
3788:   // Does the store have an offset?
3789:   if (Ops.size() == 4)
3790:     BasePtr = Builder.CreateGEP(AddrMemoryTy, BasePtr, Ops[2]);
3791: 
3792:   // Last value is always the data
3793:   Value *Val =
3794:       IsQuadStore ? Ops.back() : Builder.CreateTrunc(Ops.back(), MemoryTy);
3795: 
3796:   llvm::Type *Tys[2] = {IsQuadStore ? VectorTy : MemoryTy, BasePtr->getType()};
3797:   Function *F = CGM.getIntrinsic(IntrinsicID, Tys);
3798:   auto *Store = Builder.CreateCall(F, {Val, Predicate, BasePtr});
3799:   auto TBAAInfo = CGM.getTBAAAccessInfo(LangPTy->getPointeeType());
3800:   CGM.DecorateInstructionWithTBAA(Store, TBAAInfo);
3801:   return Store;
3802: }
3803: 
3804: Value *CodeGenFunction::EmitSMELd1St1(const SVETypeFlags &TypeFlags,
3805:                                       SmallVectorImpl<Value *> &Ops,
3806:                                       unsigned IntID) {
3807:   Ops[2] = EmitSVEPredicateCast(
3808:       Ops[2], getSVEVectorForElementType(SVEBuiltinMemEltTy(TypeFlags)));
3809: 
3810:   SmallVector<Value *> NewOps;
```
- **EN**: This block defines callable entry points like `getSVEVectorForElementType`; uses control flow (if) to specialize target builtin lowering.
- **CN**: 该代码块定义可调用入口，例如 `getSVEVectorForElementType`；通过控制流（if）细化 目标内建函数降级 行为。

### Lines 3811-3840
```cpp
3811:   NewOps.push_back(Ops[2]);
3812: 
3813:   llvm::Value *BasePtr = Ops[3];
3814:   llvm::Value *RealSlice = Ops[1];
3815:   // If the intrinsic contains the vnum parameter, multiply it with the vector
3816:   // size in bytes.
3817:   if (Ops.size() == 5) {
3818:     Function *StreamingVectorLength =
3819:         CGM.getIntrinsic(Intrinsic::aarch64_sme_cntsd);
3820:     llvm::Value *StreamingVectorLengthCall =
3821:         Builder.CreateMul(Builder.CreateCall(StreamingVectorLength),
3822:                           llvm::ConstantInt::get(Int64Ty, 8), "svl",
3823:                           /* HasNUW */ true, /* HasNSW */ true);
3824:     llvm::Value *Mulvl =
3825:         Builder.CreateMul(StreamingVectorLengthCall, Ops[4], "mulvl");
3826:     // The type of the ptr parameter is void *, so use Int8Ty here.
3827:     BasePtr = Builder.CreateGEP(Int8Ty, Ops[3], Mulvl);
3828:     RealSlice = Builder.CreateZExt(RealSlice, Int64Ty);
3829:     RealSlice = Builder.CreateAdd(RealSlice, Ops[4]);
3830:     RealSlice = Builder.CreateTrunc(RealSlice, Int32Ty);
3831:   }
3832:   NewOps.push_back(BasePtr);
3833:   NewOps.push_back(Ops[0]);
3834:   NewOps.push_back(RealSlice);
3835:   Function *F = CGM.getIntrinsic(IntID, BasePtr->getType());
3836:   return Builder.CreateCall(F, NewOps);
3837: }
3838: 
3839: Value *CodeGenFunction::EmitSMEReadWrite(const SVETypeFlags &TypeFlags,
3840:                                          SmallVectorImpl<Value *> &Ops,
```
- **EN**: This block defines callable entry points like `get`; uses control flow (if) to specialize target builtin lowering.
- **CN**: 该代码块定义可调用入口，例如 `get`；通过控制流（if）细化 目标内建函数降级 行为。

### Lines 3841-3870
```cpp
3841:                                          unsigned IntID) {
3842:   auto *VecTy = getSVEType(TypeFlags);
3843:   Function *F = CGM.getIntrinsic(IntID, VecTy);
3844:   if (TypeFlags.isReadZA())
3845:     Ops[1] = EmitSVEPredicateCast(Ops[1], VecTy);
3846:   else if (TypeFlags.isWriteZA())
3847:     Ops[2] = EmitSVEPredicateCast(Ops[2], VecTy);
3848:   return Builder.CreateCall(F, Ops);
3849: }
3850: 
3851: Value *CodeGenFunction::EmitSMEZero(const SVETypeFlags &TypeFlags,
3852:                                     SmallVectorImpl<Value *> &Ops,
3853:                                     unsigned IntID) {
3854:   // svzero_za() intrinsic zeros the entire za tile and has no paramters.
3855:   if (Ops.size() == 0)
3856:     Ops.push_back(llvm::ConstantInt::get(Int32Ty, 255));
3857:   Function *F = CGM.getIntrinsic(IntID, {});
3858:   return Builder.CreateCall(F, Ops);
3859: }
3860: 
3861: Value *CodeGenFunction::EmitSMELdrStr(const SVETypeFlags &TypeFlags,
3862:                                       SmallVectorImpl<Value *> &Ops,
3863:                                       unsigned IntID) {
3864:   if (Ops.size() == 2)
3865:     Ops.push_back(Builder.getInt32(0));
3866:   else
3867:     Ops[2] = Builder.CreateIntCast(Ops[2], Int32Ty, true);
3868:   Function *F = CGM.getIntrinsic(IntID, Ops[1]->getType());
3869:   return Builder.CreateCall(F, Ops);
3870: }
```
- **EN**: This block uses control flow (if) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（if）细化 目标内建函数降级 行为。

### Lines 3871-3900
```cpp
3871: 
3872: // Limit the usage of scalable llvm IR generated by the ACLE by using the
3873: // sve dup.x intrinsic instead of IRBuilder::CreateVectorSplat.
3874: Value *CodeGenFunction::EmitSVEDupX(Value *Scalar, llvm::Type *Ty) {
3875:   return Builder.CreateVectorSplat(
3876:       cast<llvm::VectorType>(Ty)->getElementCount(), Scalar);
3877: }
3878: 
3879: Value *CodeGenFunction::EmitSVEDupX(Value *Scalar) {
3880:   if (auto *Ty = Scalar->getType(); Ty->isVectorTy()) {
3881: #ifndef NDEBUG
3882:     auto *VecTy = cast<llvm::VectorType>(Ty);
3883:     ElementCount EC = VecTy->getElementCount();
3884:     assert(EC.isScalar() && VecTy->getElementType() == Int8Ty &&
3885:            "Only <1 x i8> expected");
3886: #endif
3887:     Scalar = Builder.CreateExtractElement(Scalar, uint64_t(0));
3888:   }
3889:   return EmitSVEDupX(Scalar, getSVEVectorForElementType(Scalar->getType()));
3890: }
3891: 
3892: Value *CodeGenFunction::EmitSVEReinterpret(Value *Val, llvm::Type *Ty) {
3893:   // FIXME: For big endian this needs an additional REV, or needs a separate
3894:   // intrinsic that is code-generated as a no-op, because the LLVM bitcast
3895:   // instruction is defined as 'bitwise' equivalent from memory point of
3896:   // view (when storing/reloading), whereas the svreinterpret builtin
3897:   // implements bitwise equivalent cast from register point of view.
3898:   // LLVM CodeGen for a bitcast must add an explicit REV for big-endian.
3899: 
3900:   if (auto *StructTy = dyn_cast<StructType>(Ty)) {
```
- **EN**: This block defines callable entry points like `EmitSVEDupX`; uses control flow (if) to specialize target builtin lowering; guards important invariants with assertions or unreachable markers; contains preprocessor structure that shapes compilation of this unit.
- **CN**: 该代码块定义可调用入口，例如 `EmitSVEDupX`；通过控制流（if）细化 目标内建函数降级 行为；使用断言或不可达标记保护关键不变量；包含影响本编译单元构建方式的预处理结构。

### Lines 3901-3930
```cpp
3901:     Value *Tuple = llvm::PoisonValue::get(Ty);
3902: 
3903:     for (unsigned I = 0; I < StructTy->getNumElements(); ++I) {
3904:       Value *In = Builder.CreateExtractValue(Val, I);
3905:       Value *Out = Builder.CreateBitCast(In, StructTy->getTypeAtIndex(I));
3906:       Tuple = Builder.CreateInsertValue(Tuple, Out, I);
3907:     }
3908: 
3909:     return Tuple;
3910:   }
3911: 
3912:   return Builder.CreateBitCast(Val, Ty);
3913: }
3914: 
3915: static void InsertExplicitZeroOperand(CGBuilderTy &Builder, llvm::Type *Ty,
3916:                                       SmallVectorImpl<Value *> &Ops) {
3917:   auto *SplatZero = Constant::getNullValue(Ty);
3918:   Ops.insert(Ops.begin(), SplatZero);
3919: }
3920: 
3921: static void InsertExplicitUndefOperand(CGBuilderTy &Builder, llvm::Type *Ty,
3922:                                        SmallVectorImpl<Value *> &Ops) {
3923:   auto *SplatUndef = UndefValue::get(Ty);
3924:   Ops.insert(Ops.begin(), SplatUndef);
3925: }
3926: 
3927: SmallVector<llvm::Type *, 2>
3928: CodeGenFunction::getSVEOverloadTypes(const SVETypeFlags &TypeFlags,
3929:                                      llvm::Type *ResultType,
3930:                                      ArrayRef<Value *> Ops) {
```
- **EN**: This block defines callable entry points like `InsertExplicitZeroOperand`, `InsertExplicitUndefOperand`, `getSVEOverloadTypes`; uses control flow (for) to specialize target builtin lowering.
- **CN**: 该代码块定义可调用入口，例如 `InsertExplicitZeroOperand`, `InsertExplicitUndefOperand`, `getSVEOverloadTypes`；通过控制流（for）细化 目标内建函数降级 行为。

### Lines 3931-3960
```cpp
3931:   if (TypeFlags.isOverloadNone())
3932:     return {};
3933: 
3934:   llvm::Type *DefaultType = getSVEType(TypeFlags);
3935: 
3936:   if (TypeFlags.isOverloadWhileOrMultiVecCvt())
3937:     return {DefaultType, Ops[1]->getType()};
3938: 
3939:   if (TypeFlags.isOverloadWhileRW())
3940:     return {getSVEPredType(TypeFlags), Ops[0]->getType()};
3941: 
3942:   if (TypeFlags.isOverloadFirstandLast())
3943:     return {Ops[0]->getType(), Ops.back()->getType()};
3944: 
3945:   if (TypeFlags.isReductionQV() && !ResultType->isScalableTy() &&
3946:       ResultType->isVectorTy())
3947:     return {ResultType, Ops[1]->getType()};
3948: 
3949:   assert(TypeFlags.isOverloadDefault() && "Unexpected value for overloads");
3950:   return {DefaultType};
3951: }
3952: 
3953: Value *CodeGenFunction::EmitSVETupleSetOrGet(const SVETypeFlags &TypeFlags,
3954:                                              ArrayRef<Value *> Ops) {
3955:   assert((TypeFlags.isTupleSet() || TypeFlags.isTupleGet()) &&
3956:          "Expects TypleFlags.isTupleSet() or TypeFlags.isTupleGet()");
3957:   unsigned Idx = cast<ConstantInt>(Ops[1])->getZExtValue();
3958: 
3959:   if (TypeFlags.isTupleSet())
3960:     return Builder.CreateInsertValue(Ops[0], Ops[2], Idx);
```
- **EN**: This block uses control flow (if, for) to specialize target builtin lowering; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块通过控制流（if, for）细化 目标内建函数降级 行为；使用断言或不可达标记保护关键不变量。

### Lines 3961-3990
```cpp
3961:   return Builder.CreateExtractValue(Ops[0], Idx);
3962: }
3963: 
3964: Value *CodeGenFunction::EmitSVETupleCreate(const SVETypeFlags &TypeFlags,
3965:                                            llvm::Type *Ty,
3966:                                            ArrayRef<Value *> Ops) {
3967:   assert(TypeFlags.isTupleCreate() && "Expects TypleFlag isTupleCreate");
3968: 
3969:   Value *Tuple = llvm::PoisonValue::get(Ty);
3970:   for (unsigned Idx = 0; Idx < Ops.size(); Idx++)
3971:     Tuple = Builder.CreateInsertValue(Tuple, Ops[Idx], Idx);
3972: 
3973:   return Tuple;
3974: }
3975: 
3976: void CodeGenFunction::GetAArch64SVEProcessedOperands(
3977:     unsigned BuiltinID, const CallExpr *E, SmallVectorImpl<Value *> &Ops,
3978:     SVETypeFlags TypeFlags) {
3979:   // Find out if any arguments are required to be integer constant expressions.
3980:   unsigned ICEArguments = 0;
3981:   ASTContext::GetBuiltinTypeError Error;
3982:   getContext().GetBuiltinType(BuiltinID, Error, &ICEArguments);
3983:   assert(Error == ASTContext::GE_None && "Should not codegen an error");
3984: 
3985:   // Tuple set/get only requires one insert/extract vector, which is
3986:   // created by EmitSVETupleSetOrGet.
3987:   bool IsTupleGetOrSet = TypeFlags.isTupleSet() || TypeFlags.isTupleGet();
3988: 
3989:   for (unsigned i = 0, e = E->getNumArgs(); i != e; i++) {
3990:     bool IsICE = ICEArguments & (1 << i);
```
- **EN**: This block defines callable entry points like `GetAArch64SVEProcessedOperands`, `getContext`; uses control flow (for) to specialize target builtin lowering; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块定义可调用入口，例如 `GetAArch64SVEProcessedOperands`, `getContext`；通过控制流（for）细化 目标内建函数降级 行为；使用断言或不可达标记保护关键不变量。

### Lines 3991-4020
```cpp
3991:     Value *Arg = EmitScalarExpr(E->getArg(i));
3992: 
3993:     if (IsICE) {
3994:       // If this is required to be a constant, constant fold it so that we know
3995:       // that the generated intrinsic gets a ConstantInt.
3996:       std::optional<llvm::APSInt> Result =
3997:           E->getArg(i)->getIntegerConstantExpr(getContext());
3998:       assert(Result && "Expected argument to be a constant");
3999: 
4000:       // Immediates for SVE llvm intrinsics are always 32bit.  We can safely
4001:       // truncate because the immediate has been range checked and no valid
4002:       // immediate requires more than a handful of bits.
4003:       *Result = Result->extOrTrunc(32);
4004:       Ops.push_back(llvm::ConstantInt::get(getLLVMContext(), *Result));
4005:       continue;
4006:     }
4007: 
4008:     if (isa<StructType>(Arg->getType()) && !IsTupleGetOrSet) {
4009:       for (unsigned I = 0; I < Arg->getType()->getStructNumElements(); ++I)
4010:         Ops.push_back(Builder.CreateExtractValue(Arg, I));
4011: 
4012:       continue;
4013:     }
4014: 
4015:     Ops.push_back(Arg);
4016:   }
4017: }
4018: 
4019: Value *CodeGenFunction::EmitAArch64SVEBuiltinExpr(unsigned BuiltinID,
4020:                                                   const CallExpr *E) {
```
- **EN**: This block uses control flow (if, for) to specialize target builtin lowering; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块通过控制流（if, for）细化 目标内建函数降级 行为；使用断言或不可达标记保护关键不变量。

### Lines 4021-4050
```cpp
4021:   llvm::Type *Ty = ConvertType(E->getType());
4022:   if (BuiltinID >= SVE::BI__builtin_sve_reinterpret_s8_s8 &&
4023:       BuiltinID <= SVE::BI__builtin_sve_reinterpret_f64_f64_x4) {
4024:     Value *Val = EmitScalarExpr(E->getArg(0));
4025:     return EmitSVEReinterpret(Val, Ty);
4026:   }
4027: 
4028:   auto *Builtin = findARMVectorIntrinsicInMap(AArch64SVEIntrinsicMap, BuiltinID,
4029:                                               AArch64SVEIntrinsicsProvenSorted);
4030: 
4031:   llvm::SmallVector<Value *, 4> Ops;
4032:   SVETypeFlags TypeFlags(Builtin->TypeModifier);
4033:   GetAArch64SVEProcessedOperands(BuiltinID, E, Ops, TypeFlags);
4034: 
4035:   if (TypeFlags.isLoad())
4036:     return EmitSVEMaskedLoad(E, Ty, Ops, Builtin->LLVMIntrinsic,
4037:                              TypeFlags.isZExtReturn());
4038:   if (TypeFlags.isStore())
4039:     return EmitSVEMaskedStore(E, Ops, Builtin->LLVMIntrinsic);
4040:   if (TypeFlags.isGatherLoad())
4041:     return EmitSVEGatherLoad(TypeFlags, Ops, Builtin->LLVMIntrinsic);
4042:   if (TypeFlags.isScatterStore())
4043:     return EmitSVEScatterStore(TypeFlags, Ops, Builtin->LLVMIntrinsic);
4044:   if (TypeFlags.isPrefetch())
4045:     return EmitSVEPrefetchLoad(TypeFlags, Ops, Builtin->LLVMIntrinsic);
4046:   if (TypeFlags.isGatherPrefetch())
4047:     return EmitSVEGatherPrefetch(TypeFlags, Ops, Builtin->LLVMIntrinsic);
4048:   if (TypeFlags.isStructLoad())
4049:     return EmitSVEStructLoad(TypeFlags, Ops, Builtin->LLVMIntrinsic);
4050:   if (TypeFlags.isStructStore())
```
- **EN**: This block defines callable entry points like `EmitSVEReinterpret`, `TypeFlags`, `GetAArch64SVEProcessedOperands`; uses control flow (if) to specialize target builtin lowering.
- **CN**: 该代码块定义可调用入口，例如 `EmitSVEReinterpret`, `TypeFlags`, `GetAArch64SVEProcessedOperands`；通过控制流（if）细化 目标内建函数降级 行为。

### Lines 4051-4080
```cpp
4051:     return EmitSVEStructStore(TypeFlags, Ops, Builtin->LLVMIntrinsic);
4052:   if (TypeFlags.isTupleSet() || TypeFlags.isTupleGet())
4053:     return EmitSVETupleSetOrGet(TypeFlags, Ops);
4054:   if (TypeFlags.isTupleCreate())
4055:     return EmitSVETupleCreate(TypeFlags, Ty, Ops);
4056:   if (TypeFlags.isUndef())
4057:     return UndefValue::get(Ty);
4058: 
4059:   // Handle built-ins for which there is a corresponding LLVM Intrinsic.
4060:   // -------------------------------------------------------------------
4061:   if (Builtin->LLVMIntrinsic != 0) {
4062:     // Emit set FPMR for intrinsics that require it
4063:     if (TypeFlags.setsFPMR())
4064:       Builder.CreateCall(CGM.getIntrinsic(Intrinsic::aarch64_set_fpmr),
4065:                          Ops.pop_back_val());
4066:     if (TypeFlags.getMergeType() == SVETypeFlags::MergeZeroExp)
4067:       InsertExplicitZeroOperand(Builder, Ty, Ops);
4068: 
4069:     if (TypeFlags.getMergeType() == SVETypeFlags::MergeAnyExp)
4070:       InsertExplicitUndefOperand(Builder, Ty, Ops);
4071: 
4072:     // Some ACLE builtins leave out the argument to specify the predicate
4073:     // pattern, which is expected to be expanded to an SV_ALL pattern.
4074:     if (TypeFlags.isAppendSVALL())
4075:       Ops.push_back(Builder.getInt32(/*SV_ALL*/ 31));
4076:     if (TypeFlags.isInsertOp1SVALL())
4077:       Ops.insert(&Ops[1], Builder.getInt32(/*SV_ALL*/ 31));
4078: 
4079:     // Predicates must match the main datatype.
4080:     for (Value *&Op : Ops)
```
- **EN**: This block defines callable entry points like `EmitSVEStructStore`; uses control flow (if, for) to specialize target builtin lowering.
- **CN**: 该代码块定义可调用入口，例如 `EmitSVEStructStore`；通过控制流（if, for）细化 目标内建函数降级 行为。

### Lines 4081-4110
```cpp
4081:       if (auto PredTy = dyn_cast<llvm::VectorType>(Op->getType()))
4082:         if (PredTy->getElementType()->isIntegerTy(1))
4083:           Op = EmitSVEPredicateCast(Op, getSVEType(TypeFlags));
4084: 
4085:     // Splat scalar operand to vector (intrinsics with _n infix)
4086:     if (TypeFlags.hasSplatOperand()) {
4087:       unsigned OpNo = TypeFlags.getSplatOperand();
4088:       Ops[OpNo] = EmitSVEDupX(Ops[OpNo]);
4089:     }
4090: 
4091:     if (TypeFlags.isReverseCompare())
4092:       std::swap(Ops[1], Ops[2]);
4093:     else if (TypeFlags.isReverseUSDOT())
4094:       std::swap(Ops[1], Ops[2]);
4095:     else if (TypeFlags.isReverseMergeAnyBinOp() &&
4096:              TypeFlags.getMergeType() == SVETypeFlags::MergeAny)
4097:       std::swap(Ops[1], Ops[2]);
4098:     else if (TypeFlags.isReverseMergeAnyAccOp() &&
4099:              TypeFlags.getMergeType() == SVETypeFlags::MergeAny)
4100:       std::swap(Ops[1], Ops[3]);
4101: 
4102:     // Predicated intrinsics with _z suffix need a select w/ zeroinitializer.
4103:     if (TypeFlags.getMergeType() == SVETypeFlags::MergeZero) {
4104:       llvm::Type *OpndTy = Ops[1]->getType();
4105:       auto *SplatZero = Constant::getNullValue(OpndTy);
4106:       Ops[1] = Builder.CreateSelect(Ops[0], Ops[1], SplatZero);
4107:     }
4108: 
4109:     Function *F = CGM.getIntrinsic(Builtin->LLVMIntrinsic,
4110:                                    getSVEOverloadTypes(TypeFlags, Ty, Ops));
```
- **EN**: This block defines callable entry points like `getSVEOverloadTypes`; uses control flow (if) to specialize target builtin lowering.
- **CN**: 该代码块定义可调用入口，例如 `getSVEOverloadTypes`；通过控制流（if）细化 目标内建函数降级 行为。

### Lines 4111-4140
```cpp
4111:     Value *Call = Builder.CreateCall(F, Ops);
4112: 
4113:     if (Call->getType() == Ty)
4114:       return Call;
4115: 
4116:     // Predicate results must be converted to svbool_t.
4117:     if (auto PredTy = dyn_cast<llvm::ScalableVectorType>(Ty))
4118:       return EmitSVEPredicateCast(Call, PredTy);
4119:     if (auto PredTupleTy = dyn_cast<llvm::StructType>(Ty))
4120:       return EmitSVEPredicateTupleCast(Call, PredTupleTy);
4121: 
4122:     llvm_unreachable("unsupported element count!");
4123:   }
4124: 
4125:   switch (BuiltinID) {
4126:   default:
4127:     return nullptr;
4128: 
4129:   case SVE::BI__builtin_sve_svreinterpret_b: {
4130:     auto SVCountTy =
4131:         llvm::TargetExtType::get(getLLVMContext(), "aarch64.svcount");
4132:     Function *CastFromSVCountF =
4133:         CGM.getIntrinsic(Intrinsic::aarch64_sve_convert_to_svbool, SVCountTy);
4134:     return Builder.CreateCall(CastFromSVCountF, Ops[0]);
4135:   }
4136:   case SVE::BI__builtin_sve_svreinterpret_c: {
4137:     auto SVCountTy =
4138:         llvm::TargetExtType::get(getLLVMContext(), "aarch64.svcount");
4139:     Function *CastToSVCountF =
4140:         CGM.getIntrinsic(Intrinsic::aarch64_sve_convert_from_svbool, SVCountTy);
```
- **EN**: This block defines callable entry points like `get`; uses control flow (if, switch, case) to specialize target builtin lowering; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块定义可调用入口，例如 `get`；通过控制流（if, switch, case）细化 目标内建函数降级 行为；使用断言或不可达标记保护关键不变量。

### Lines 4141-4170
```cpp
4141:     return Builder.CreateCall(CastToSVCountF, Ops[0]);
4142:   }
4143: 
4144:   case SVE::BI__builtin_sve_svpsel_lane_b8:
4145:   case SVE::BI__builtin_sve_svpsel_lane_b16:
4146:   case SVE::BI__builtin_sve_svpsel_lane_b32:
4147:   case SVE::BI__builtin_sve_svpsel_lane_b64:
4148:   case SVE::BI__builtin_sve_svpsel_lane_c8:
4149:   case SVE::BI__builtin_sve_svpsel_lane_c16:
4150:   case SVE::BI__builtin_sve_svpsel_lane_c32:
4151:   case SVE::BI__builtin_sve_svpsel_lane_c64: {
4152:     bool IsSVCount = isa<TargetExtType>(Ops[0]->getType());
4153:     assert(((!IsSVCount || cast<TargetExtType>(Ops[0]->getType())->getName() ==
4154:                                "aarch64.svcount")) &&
4155:            "Unexpected TargetExtType");
4156:     auto SVCountTy =
4157:         llvm::TargetExtType::get(getLLVMContext(), "aarch64.svcount");
4158:     Function *CastFromSVCountF =
4159:         CGM.getIntrinsic(Intrinsic::aarch64_sve_convert_to_svbool, SVCountTy);
4160:     Function *CastToSVCountF =
4161:         CGM.getIntrinsic(Intrinsic::aarch64_sve_convert_from_svbool, SVCountTy);
4162: 
4163:     auto OverloadedTy = getSVEType(SVETypeFlags(Builtin->TypeModifier));
4164:     Function *F = CGM.getIntrinsic(Intrinsic::aarch64_sve_psel, OverloadedTy);
4165:     llvm::Value *Ops0 =
4166:         IsSVCount ? Builder.CreateCall(CastFromSVCountF, Ops[0]) : Ops[0];
4167:     llvm::Value *Ops1 = EmitSVEPredicateCast(Ops[1], OverloadedTy);
4168:     llvm::Value *PSel = Builder.CreateCall(F, {Ops0, Ops1, Ops[2]});
4169:     return IsSVCount ? Builder.CreateCall(CastToSVCountF, PSel) : PSel;
4170:   }
```
- **EN**: This block defines callable entry points like `get`; uses control flow (case) to specialize target builtin lowering; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块定义可调用入口，例如 `get`；通过控制流（case）细化 目标内建函数降级 行为；使用断言或不可达标记保护关键不变量。

### Lines 4171-4200
```cpp
4171:   case SVE::BI__builtin_sve_svmov_b_z: {
4172:     // svmov_b_z(pg, op) <=> svand_b_z(pg, op, op)
4173:     SVETypeFlags TypeFlags(Builtin->TypeModifier);
4174:     llvm::Type* OverloadedTy = getSVEType(TypeFlags);
4175:     Function *F = CGM.getIntrinsic(Intrinsic::aarch64_sve_and_z, OverloadedTy);
4176:     return Builder.CreateCall(F, {Ops[0], Ops[1], Ops[1]});
4177:   }
4178: 
4179:   case SVE::BI__builtin_sve_svnot_b_z: {
4180:     // svnot_b_z(pg, op) <=> sveor_b_z(pg, op, pg)
4181:     SVETypeFlags TypeFlags(Builtin->TypeModifier);
4182:     llvm::Type* OverloadedTy = getSVEType(TypeFlags);
4183:     Function *F = CGM.getIntrinsic(Intrinsic::aarch64_sve_eor_z, OverloadedTy);
4184:     return Builder.CreateCall(F, {Ops[0], Ops[1], Ops[0]});
4185:   }
4186: 
4187:   case SVE::BI__builtin_sve_svmovlb_u16:
4188:   case SVE::BI__builtin_sve_svmovlb_u32:
4189:   case SVE::BI__builtin_sve_svmovlb_u64:
4190:     return EmitSVEMovl(TypeFlags, Ops, Intrinsic::aarch64_sve_ushllb);
4191: 
4192:   case SVE::BI__builtin_sve_svmovlb_s16:
4193:   case SVE::BI__builtin_sve_svmovlb_s32:
4194:   case SVE::BI__builtin_sve_svmovlb_s64:
4195:     return EmitSVEMovl(TypeFlags, Ops, Intrinsic::aarch64_sve_sshllb);
4196: 
4197:   case SVE::BI__builtin_sve_svmovlt_u16:
4198:   case SVE::BI__builtin_sve_svmovlt_u32:
4199:   case SVE::BI__builtin_sve_svmovlt_u64:
4200:     return EmitSVEMovl(TypeFlags, Ops, Intrinsic::aarch64_sve_ushllt);
```
- **EN**: This block defines callable entry points like `TypeFlags`, `EmitSVEMovl`; uses control flow (case) to specialize target builtin lowering.
- **CN**: 该代码块定义可调用入口，例如 `TypeFlags`, `EmitSVEMovl`；通过控制流（case）细化 目标内建函数降级 行为。

### Lines 4201-4230
```cpp
4201: 
4202:   case SVE::BI__builtin_sve_svmovlt_s16:
4203:   case SVE::BI__builtin_sve_svmovlt_s32:
4204:   case SVE::BI__builtin_sve_svmovlt_s64:
4205:     return EmitSVEMovl(TypeFlags, Ops, Intrinsic::aarch64_sve_sshllt);
4206: 
4207:   case SVE::BI__builtin_sve_svpmullt_u16:
4208:   case SVE::BI__builtin_sve_svpmullt_u64:
4209:   case SVE::BI__builtin_sve_svpmullt_n_u16:
4210:   case SVE::BI__builtin_sve_svpmullt_n_u64:
4211:     return EmitSVEPMull(TypeFlags, Ops, Intrinsic::aarch64_sve_pmullt_pair);
4212: 
4213:   case SVE::BI__builtin_sve_svpmullb_u16:
4214:   case SVE::BI__builtin_sve_svpmullb_u64:
4215:   case SVE::BI__builtin_sve_svpmullb_n_u16:
4216:   case SVE::BI__builtin_sve_svpmullb_n_u64:
4217:     return EmitSVEPMull(TypeFlags, Ops, Intrinsic::aarch64_sve_pmullb_pair);
4218: 
4219:   case SVE::BI__builtin_sve_svdup_n_b8:
4220:   case SVE::BI__builtin_sve_svdup_n_b16:
4221:   case SVE::BI__builtin_sve_svdup_n_b32:
4222:   case SVE::BI__builtin_sve_svdup_n_b64: {
4223:     Value *CmpNE =
4224:         Builder.CreateICmpNE(Ops[0], Constant::getNullValue(Ops[0]->getType()));
4225:     llvm::ScalableVectorType *OverloadedTy = getSVEType(TypeFlags);
4226:     Value *Dup = EmitSVEDupX(CmpNE, OverloadedTy);
4227:     return EmitSVEPredicateCast(Dup, cast<llvm::ScalableVectorType>(Ty));
4228:   }
4229: 
4230:   case SVE::BI__builtin_sve_svdupq_n_b8:
```
- **EN**: This block defines callable entry points like `EmitSVEMovl`, `EmitSVEPMull`, `EmitSVEPredicateCast`; uses control flow (case) to specialize target builtin lowering.
- **CN**: 该代码块定义可调用入口，例如 `EmitSVEMovl`, `EmitSVEPMull`, `EmitSVEPredicateCast`；通过控制流（case）细化 目标内建函数降级 行为。

### Lines 4231-4260
```cpp
4231:   case SVE::BI__builtin_sve_svdupq_n_b16:
4232:   case SVE::BI__builtin_sve_svdupq_n_b32:
4233:   case SVE::BI__builtin_sve_svdupq_n_b64:
4234:   case SVE::BI__builtin_sve_svdupq_n_u8:
4235:   case SVE::BI__builtin_sve_svdupq_n_s8:
4236:   case SVE::BI__builtin_sve_svdupq_n_u64:
4237:   case SVE::BI__builtin_sve_svdupq_n_f64:
4238:   case SVE::BI__builtin_sve_svdupq_n_s64:
4239:   case SVE::BI__builtin_sve_svdupq_n_u16:
4240:   case SVE::BI__builtin_sve_svdupq_n_f16:
4241:   case SVE::BI__builtin_sve_svdupq_n_bf16:
4242:   case SVE::BI__builtin_sve_svdupq_n_s16:
4243:   case SVE::BI__builtin_sve_svdupq_n_u32:
4244:   case SVE::BI__builtin_sve_svdupq_n_f32:
4245:   case SVE::BI__builtin_sve_svdupq_n_s32: {
4246:     // These builtins are implemented by storing each element to an array and using
4247:     // ld1rq to materialize a vector.
4248:     unsigned NumOpnds = Ops.size();
4249: 
4250:     bool IsBoolTy =
4251:         cast<llvm::VectorType>(Ty)->getElementType()->isIntegerTy(1);
4252: 
4253:     // For svdupq_n_b* the element type of is an integer of type 128/numelts,
4254:     // so that the compare can use the width that is natural for the expected
4255:     // number of predicate lanes.
4256:     llvm::Type *EltTy = Ops[0]->getType();
4257:     if (IsBoolTy)
4258:       EltTy = IntegerType::get(getLLVMContext(), SVEBitsPerBlock / NumOpnds);
4259: 
4260:     SmallVector<llvm::Value *, 16> VecOps;
```
- **EN**: This block uses control flow (if, case) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（if, case）细化 目标内建函数降级 行为。

### Lines 4261-4290
```cpp
4261:     for (unsigned I = 0; I < NumOpnds; ++I)
4262:         VecOps.push_back(Builder.CreateZExt(Ops[I], EltTy));
4263:     Value *Vec = BuildVector(VecOps);
4264: 
4265:     llvm::Type *OverloadedTy = getSVEVectorForElementType(EltTy);
4266:     Value *InsertSubVec = Builder.CreateInsertVector(
4267:         OverloadedTy, PoisonValue::get(OverloadedTy), Vec, uint64_t(0));
4268: 
4269:     Function *F =
4270:         CGM.getIntrinsic(Intrinsic::aarch64_sve_dupq_lane, OverloadedTy);
4271:     Value *DupQLane =
4272:         Builder.CreateCall(F, {InsertSubVec, Builder.getInt64(0)});
4273: 
4274:     if (!IsBoolTy)
4275:       return DupQLane;
4276: 
4277:     SVETypeFlags TypeFlags(Builtin->TypeModifier);
4278:     Value *Pred = EmitSVEAllTruePred(TypeFlags);
4279: 
4280:     // For svdupq_n_b* we need to add an additional 'cmpne' with '0'.
4281:     F = CGM.getIntrinsic(NumOpnds == 2 ? Intrinsic::aarch64_sve_cmpne
4282:                                        : Intrinsic::aarch64_sve_cmpne_wide,
4283:                          OverloadedTy);
4284:     Value *Call = Builder.CreateCall(
4285:         F, {Pred, DupQLane, EmitSVEDupX(Builder.getInt64(0))});
4286:     return EmitSVEPredicateCast(Call, cast<llvm::ScalableVectorType>(Ty));
4287:   }
4288: 
4289:   case SVE::BI__builtin_sve_svpfalse_b:
4290:     return ConstantInt::getFalse(Ty);
```
- **EN**: This block defines callable entry points like `get`, `TypeFlags`, `EmitSVEPredicateCast`, `getFalse`; uses control flow (if, for, case) to specialize target builtin lowering.
- **CN**: 该代码块定义可调用入口，例如 `get`, `TypeFlags`, `EmitSVEPredicateCast`, `getFalse`；通过控制流（if, for, case）细化 目标内建函数降级 行为。

### Lines 4291-4320
```cpp
4291: 
4292:   case SVE::BI__builtin_sve_svpfalse_c: {
4293:     auto SVBoolTy = ScalableVectorType::get(Builder.getInt1Ty(), 16);
4294:     Function *CastToSVCountF =
4295:         CGM.getIntrinsic(Intrinsic::aarch64_sve_convert_from_svbool, Ty);
4296:     return Builder.CreateCall(CastToSVCountF, ConstantInt::getFalse(SVBoolTy));
4297:   }
4298: 
4299:   case SVE::BI__builtin_sve_svlen_bf16:
4300:   case SVE::BI__builtin_sve_svlen_f16:
4301:   case SVE::BI__builtin_sve_svlen_f32:
4302:   case SVE::BI__builtin_sve_svlen_f64:
4303:   case SVE::BI__builtin_sve_svlen_s8:
4304:   case SVE::BI__builtin_sve_svlen_s16:
4305:   case SVE::BI__builtin_sve_svlen_s32:
4306:   case SVE::BI__builtin_sve_svlen_s64:
4307:   case SVE::BI__builtin_sve_svlen_u8:
4308:   case SVE::BI__builtin_sve_svlen_u16:
4309:   case SVE::BI__builtin_sve_svlen_u32:
4310:   case SVE::BI__builtin_sve_svlen_u64: {
4311:     SVETypeFlags TF(Builtin->TypeModifier);
4312:     return Builder.CreateElementCount(Ty, getSVEType(TF)->getElementCount());
4313:   }
4314: 
4315:   case SVE::BI__builtin_sve_svtbl2_u8:
4316:   case SVE::BI__builtin_sve_svtbl2_s8:
4317:   case SVE::BI__builtin_sve_svtbl2_u16:
4318:   case SVE::BI__builtin_sve_svtbl2_s16:
4319:   case SVE::BI__builtin_sve_svtbl2_u32:
4320:   case SVE::BI__builtin_sve_svtbl2_s32:
```
- **EN**: This block defines callable entry points like `TF`; uses control flow (case) to specialize target builtin lowering.
- **CN**: 该代码块定义可调用入口，例如 `TF`；通过控制流（case）细化 目标内建函数降级 行为。

### Lines 4321-4350
```cpp
4321:   case SVE::BI__builtin_sve_svtbl2_u64:
4322:   case SVE::BI__builtin_sve_svtbl2_s64:
4323:   case SVE::BI__builtin_sve_svtbl2_f16:
4324:   case SVE::BI__builtin_sve_svtbl2_bf16:
4325:   case SVE::BI__builtin_sve_svtbl2_f32:
4326:   case SVE::BI__builtin_sve_svtbl2_f64: {
4327:     SVETypeFlags TF(Builtin->TypeModifier);
4328:     Function *F = CGM.getIntrinsic(Intrinsic::aarch64_sve_tbl2, getSVEType(TF));
4329:     return Builder.CreateCall(F, Ops);
4330:   }
4331: 
4332:   case SVE::BI__builtin_sve_svset_neonq_s8:
4333:   case SVE::BI__builtin_sve_svset_neonq_s16:
4334:   case SVE::BI__builtin_sve_svset_neonq_s32:
4335:   case SVE::BI__builtin_sve_svset_neonq_s64:
4336:   case SVE::BI__builtin_sve_svset_neonq_u8:
4337:   case SVE::BI__builtin_sve_svset_neonq_u16:
4338:   case SVE::BI__builtin_sve_svset_neonq_u32:
4339:   case SVE::BI__builtin_sve_svset_neonq_u64:
4340:   case SVE::BI__builtin_sve_svset_neonq_f16:
4341:   case SVE::BI__builtin_sve_svset_neonq_f32:
4342:   case SVE::BI__builtin_sve_svset_neonq_f64:
4343:   case SVE::BI__builtin_sve_svset_neonq_bf16: {
4344:     return Builder.CreateInsertVector(Ty, Ops[0], Ops[1], uint64_t(0));
4345:   }
4346: 
4347:   case SVE::BI__builtin_sve_svget_neonq_s8:
4348:   case SVE::BI__builtin_sve_svget_neonq_s16:
4349:   case SVE::BI__builtin_sve_svget_neonq_s32:
4350:   case SVE::BI__builtin_sve_svget_neonq_s64:
```
- **EN**: This block defines callable entry points like `TF`; uses control flow (case) to specialize target builtin lowering.
- **CN**: 该代码块定义可调用入口，例如 `TF`；通过控制流（case）细化 目标内建函数降级 行为。

### Lines 4351-4380
```cpp
4351:   case SVE::BI__builtin_sve_svget_neonq_u8:
4352:   case SVE::BI__builtin_sve_svget_neonq_u16:
4353:   case SVE::BI__builtin_sve_svget_neonq_u32:
4354:   case SVE::BI__builtin_sve_svget_neonq_u64:
4355:   case SVE::BI__builtin_sve_svget_neonq_f16:
4356:   case SVE::BI__builtin_sve_svget_neonq_f32:
4357:   case SVE::BI__builtin_sve_svget_neonq_f64:
4358:   case SVE::BI__builtin_sve_svget_neonq_bf16: {
4359:     return Builder.CreateExtractVector(Ty, Ops[0], uint64_t(0));
4360:   }
4361: 
4362:   case SVE::BI__builtin_sve_svdup_neonq_s8:
4363:   case SVE::BI__builtin_sve_svdup_neonq_s16:
4364:   case SVE::BI__builtin_sve_svdup_neonq_s32:
4365:   case SVE::BI__builtin_sve_svdup_neonq_s64:
4366:   case SVE::BI__builtin_sve_svdup_neonq_u8:
4367:   case SVE::BI__builtin_sve_svdup_neonq_u16:
4368:   case SVE::BI__builtin_sve_svdup_neonq_u32:
4369:   case SVE::BI__builtin_sve_svdup_neonq_u64:
4370:   case SVE::BI__builtin_sve_svdup_neonq_f16:
4371:   case SVE::BI__builtin_sve_svdup_neonq_f32:
4372:   case SVE::BI__builtin_sve_svdup_neonq_f64:
4373:   case SVE::BI__builtin_sve_svdup_neonq_bf16: {
4374:     Value *Insert = Builder.CreateInsertVector(Ty, PoisonValue::get(Ty), Ops[0],
4375:                                                uint64_t(0));
4376:     return Builder.CreateIntrinsic(Intrinsic::aarch64_sve_dupq_lane, {Ty},
4377:                                    {Insert, Builder.getInt64(0)});
4378:   }
4379:   }
4380: 
```
- **EN**: This block defines callable entry points like `uint64_t`; uses control flow (case) to specialize target builtin lowering.
- **CN**: 该代码块定义可调用入口，例如 `uint64_t`；通过控制流（case）细化 目标内建函数降级 行为。

### Lines 4381-4410
```cpp
4381:   /// Should not happen
4382:   return nullptr;
4383: }
4384: 
4385: static void swapCommutativeSMEOperands(unsigned BuiltinID,
4386:                                        SmallVectorImpl<Value *> &Ops) {
4387:   unsigned MultiVec;
4388:   switch (BuiltinID) {
4389:   default:
4390:     return;
4391:   case SME::BI__builtin_sme_svsumla_za32_s8_vg4x1:
4392:     MultiVec = 1;
4393:     break;
4394:   case SME::BI__builtin_sme_svsumla_za32_s8_vg4x2:
4395:   case SME::BI__builtin_sme_svsudot_za32_s8_vg1x2:
4396:     MultiVec = 2;
4397:     break;
4398:   case SME::BI__builtin_sme_svsudot_za32_s8_vg1x4:
4399:   case SME::BI__builtin_sme_svsumla_za32_s8_vg4x4:
4400:     MultiVec = 4;
4401:     break;
4402:   }
4403: 
4404:   if (MultiVec > 0)
4405:     for (unsigned I = 0; I < MultiVec; ++I)
4406:       std::swap(Ops[I + 1], Ops[I + 1 + MultiVec]);
4407: }
4408: 
4409: Value *CodeGenFunction::EmitAArch64SMEBuiltinExpr(unsigned BuiltinID,
4410:                                                   const CallExpr *E) {
```
- **EN**: This block defines callable entry points like `swapCommutativeSMEOperands`, `swap`; uses control flow (if, switch, for, case) to specialize target builtin lowering.
- **CN**: 该代码块定义可调用入口，例如 `swapCommutativeSMEOperands`, `swap`；通过控制流（if, switch, for, case）细化 目标内建函数降级 行为。

### Lines 4411-4440
```cpp
4411:   auto *Builtin = findARMVectorIntrinsicInMap(AArch64SMEIntrinsicMap, BuiltinID,
4412:                                               AArch64SMEIntrinsicsProvenSorted);
4413: 
4414:   llvm::SmallVector<Value *, 4> Ops;
4415:   SVETypeFlags TypeFlags(Builtin->TypeModifier);
4416:   GetAArch64SVEProcessedOperands(BuiltinID, E, Ops, TypeFlags);
4417: 
4418:   if (TypeFlags.isLoad() || TypeFlags.isStore())
4419:     return EmitSMELd1St1(TypeFlags, Ops, Builtin->LLVMIntrinsic);
4420:   if (TypeFlags.isReadZA() || TypeFlags.isWriteZA())
4421:     return EmitSMEReadWrite(TypeFlags, Ops, Builtin->LLVMIntrinsic);
4422:   if (BuiltinID == SME::BI__builtin_sme_svzero_mask_za ||
4423:       BuiltinID == SME::BI__builtin_sme_svzero_za)
4424:     return EmitSMEZero(TypeFlags, Ops, Builtin->LLVMIntrinsic);
4425:   if (BuiltinID == SME::BI__builtin_sme_svldr_vnum_za ||
4426:       BuiltinID == SME::BI__builtin_sme_svstr_vnum_za ||
4427:       BuiltinID == SME::BI__builtin_sme_svldr_za ||
4428:       BuiltinID == SME::BI__builtin_sme_svstr_za)
4429:     return EmitSMELdrStr(TypeFlags, Ops, Builtin->LLVMIntrinsic);
4430: 
4431:   // Emit set FPMR for intrinsics that require it
4432:   if (TypeFlags.setsFPMR())
4433:     Builder.CreateCall(CGM.getIntrinsic(Intrinsic::aarch64_set_fpmr),
4434:                        Ops.pop_back_val());
4435:   // Handle builtins which require their multi-vector operands to be swapped
4436:   swapCommutativeSMEOperands(BuiltinID, Ops);
4437: 
4438:   auto isCntsBuiltin = [&]() {
4439:     switch (BuiltinID) {
4440:     default:
```
- **EN**: This block defines callable entry points like `TypeFlags`, `GetAArch64SVEProcessedOperands`, `swapCommutativeSMEOperands`; uses control flow (if, switch) to specialize target builtin lowering.
- **CN**: 该代码块定义可调用入口，例如 `TypeFlags`, `GetAArch64SVEProcessedOperands`, `swapCommutativeSMEOperands`；通过控制流（if, switch）细化 目标内建函数降级 行为。

### Lines 4441-4470
```cpp
4441:       return 0;
4442:     case SME::BI__builtin_sme_svcntsb:
4443:       return 8;
4444:     case SME::BI__builtin_sme_svcntsh:
4445:       return 4;
4446:     case SME::BI__builtin_sme_svcntsw:
4447:       return 2;
4448:     }
4449:   };
4450: 
4451:   if (auto Mul = isCntsBuiltin()) {
4452:     llvm::Value *Cntd =
4453:         Builder.CreateCall(CGM.getIntrinsic(Intrinsic::aarch64_sme_cntsd));
4454:     return Builder.CreateMul(Cntd, llvm::ConstantInt::get(Int64Ty, Mul),
4455:                              "mulsvl", /* HasNUW */ true, /* HasNSW */ true);
4456:   }
4457: 
4458:   // Should not happen!
4459:   if (Builtin->LLVMIntrinsic == 0)
4460:     return nullptr;
4461: 
4462:   // Predicates must match the main datatype.
4463:   for (Value *&Op : Ops)
4464:     if (auto PredTy = dyn_cast<llvm::VectorType>(Op->getType()))
4465:       if (PredTy->getElementType()->isIntegerTy(1))
4466:         Op = EmitSVEPredicateCast(Op, getSVEType(TypeFlags));
4467: 
4468:   if (BuiltinID == SME::BI__builtin_sme_svldr_zt ||
4469:       BuiltinID == SME::BI__builtin_sme_svstr_zt) {
4470:     Function *F = CGM.getIntrinsic(Builtin->LLVMIntrinsic, Ops[1]->getType());
```
- **EN**: This block uses control flow (if, for, case) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（if, for, case）细化 目标内建函数降级 行为。

### Lines 4471-4500
```cpp
4471:     return Builder.CreateCall(F, Ops);
4472:   }
4473: 
4474:   Function *F =
4475:       TypeFlags.isOverloadNone()
4476:           ? CGM.getIntrinsic(Builtin->LLVMIntrinsic)
4477:           : CGM.getIntrinsic(Builtin->LLVMIntrinsic, {getSVEType(TypeFlags)});
4478: 
4479:   return Builder.CreateCall(F, Ops);
4480: }
4481: 
4482: /// Helper for the read/write/add/inc X18 builtins: read the X18 register and
4483: /// return it as an i8 pointer.
4484: Value *readX18AsPtr(CodeGenFunction &CGF) {
4485:   LLVMContext &Context = CGF.CGM.getLLVMContext();
4486:   llvm::Metadata *Ops[] = {llvm::MDString::get(Context, "x18")};
4487:   llvm::MDNode *RegName = llvm::MDNode::get(Context, Ops);
4488:   llvm::Value *Metadata = llvm::MetadataAsValue::get(Context, RegName);
4489:   llvm::Function *F =
4490:       CGF.CGM.getIntrinsic(Intrinsic::read_register, {CGF.Int64Ty});
4491:   llvm::Value *X18 = CGF.Builder.CreateCall(F, Metadata);
4492:   return CGF.Builder.CreateIntToPtr(X18, CGF.Int8PtrTy);
4493: }
4494: 
4495: Value *CodeGenFunction::EmitAArch64BuiltinExpr(unsigned BuiltinID,
4496:                                                const CallExpr *E,
4497:                                                llvm::Triple::ArchType Arch) {
4498:   if (BuiltinID >= clang::AArch64::FirstSVEBuiltin &&
4499:       BuiltinID <= clang::AArch64::LastSVEBuiltin)
4500:     return EmitAArch64SVEBuiltinExpr(BuiltinID, E);
```
- **EN**: This block uses control flow (if) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（if）细化 目标内建函数降级 行为。

### Lines 4501-4530
```cpp
4501: 
4502:   if (BuiltinID >= clang::AArch64::FirstSMEBuiltin &&
4503:       BuiltinID <= clang::AArch64::LastSMEBuiltin)
4504:     return EmitAArch64SMEBuiltinExpr(BuiltinID, E);
4505: 
4506:   if (BuiltinID == Builtin::BI__builtin_cpu_supports)
4507:     return EmitAArch64CpuSupports(E);
4508: 
4509:   unsigned HintID = static_cast<unsigned>(-1);
4510:   switch (BuiltinID) {
4511:   default: break;
4512:   case clang::AArch64::BI__builtin_arm_nop:
4513:     HintID = 0;
4514:     break;
4515:   case clang::AArch64::BI__builtin_arm_yield:
4516:   case clang::AArch64::BI__yield:
4517:     HintID = 1;
4518:     break;
4519:   case clang::AArch64::BI__builtin_arm_wfe:
4520:   case clang::AArch64::BI__wfe:
4521:     HintID = 2;
4522:     break;
4523:   case clang::AArch64::BI__builtin_arm_wfi:
4524:   case clang::AArch64::BI__wfi:
4525:     HintID = 3;
4526:     break;
4527:   case clang::AArch64::BI__builtin_arm_sev:
4528:   case clang::AArch64::BI__sev:
4529:     HintID = 4;
4530:     break;
```
- **EN**: This block uses control flow (if, switch, case) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（if, switch, case）细化 目标内建函数降级 行为。

### Lines 4531-4560
```cpp
4531:   case clang::AArch64::BI__builtin_arm_sevl:
4532:   case clang::AArch64::BI__sevl:
4533:     HintID = 5;
4534:     break;
4535:   }
4536: 
4537:   if (HintID != static_cast<unsigned>(-1)) {
4538:     Function *F = CGM.getIntrinsic(Intrinsic::aarch64_hint);
4539:     return Builder.CreateCall(F, llvm::ConstantInt::get(Int32Ty, HintID));
4540:   }
4541: 
4542:   if (BuiltinID == clang::AArch64::BI__builtin_arm_trap) {
4543:     Function *F = CGM.getIntrinsic(Intrinsic::aarch64_break);
4544:     llvm::Value *Arg = EmitScalarExpr(E->getArg(0));
4545:     return Builder.CreateCall(F, Builder.CreateZExt(Arg, CGM.Int32Ty));
4546:   }
4547: 
4548:   if (BuiltinID == clang::AArch64::BI__builtin_arm_get_sme_state) {
4549:     // Create call to __arm_sme_state and store the results to the two pointers.
4550:     CallInst *CI = EmitRuntimeCall(CGM.CreateRuntimeFunction(
4551:         llvm::FunctionType::get(StructType::get(CGM.Int64Ty, CGM.Int64Ty), {},
4552:                                 false),
4553:         "__arm_sme_state"));
4554:     auto Attrs = AttributeList().addFnAttribute(getLLVMContext(),
4555:                                                 "aarch64_pstate_sm_compatible");
4556:     CI->setAttributes(Attrs);
4557:     CI->setCallingConv(
4558:         llvm::CallingConv::
4559:             AArch64_SME_ABI_Support_Routines_PreserveMost_From_X2);
4560:     Builder.CreateStore(Builder.CreateExtractValue(CI, 0),
```
- **EN**: This block uses control flow (if, case) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（if, case）细化 目标内建函数降级 行为。

### Lines 4561-4590
```cpp
4561:                         EmitPointerWithAlignment(E->getArg(0)));
4562:     return Builder.CreateStore(Builder.CreateExtractValue(CI, 1),
4563:                                EmitPointerWithAlignment(E->getArg(1)));
4564:   }
4565: 
4566:   if (BuiltinID == clang::AArch64::BI__builtin_arm_rbit) {
4567:     assert((getContext().getTypeSize(E->getType()) == 32) &&
4568:            "rbit of unusual size!");
4569:     llvm::Value *Arg = EmitScalarExpr(E->getArg(0));
4570:     return Builder.CreateCall(
4571:         CGM.getIntrinsic(Intrinsic::bitreverse, Arg->getType()), Arg, "rbit");
4572:   }
4573:   if (BuiltinID == clang::AArch64::BI__builtin_arm_rbit64) {
4574:     assert((getContext().getTypeSize(E->getType()) == 64) &&
4575:            "rbit of unusual size!");
4576:     llvm::Value *Arg = EmitScalarExpr(E->getArg(0));
4577:     return Builder.CreateCall(
4578:         CGM.getIntrinsic(Intrinsic::bitreverse, Arg->getType()), Arg, "rbit");
4579:   }
4580: 
4581:   if (BuiltinID == clang::AArch64::BI__builtin_arm_clz ||
4582:       BuiltinID == clang::AArch64::BI__builtin_arm_clz64) {
4583:     llvm::Value *Arg = EmitScalarExpr(E->getArg(0));
4584:     Function *F = CGM.getIntrinsic(Intrinsic::ctlz, Arg->getType());
4585:     Value *Res = Builder.CreateCall(F, {Arg, Builder.getInt1(false)});
4586:     if (BuiltinID == clang::AArch64::BI__builtin_arm_clz64)
4587:       Res = Builder.CreateTrunc(Res, Builder.getInt32Ty());
4588:     return Res;
4589:   }
4590: 
```
- **EN**: This block defines callable entry points like `EmitPointerWithAlignment`; uses control flow (if) to specialize target builtin lowering; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块定义可调用入口，例如 `EmitPointerWithAlignment`；通过控制流（if）细化 目标内建函数降级 行为；使用断言或不可达标记保护关键不变量。

### Lines 4591-4620
```cpp
4591:   if (BuiltinID == clang::AArch64::BI__builtin_arm_cls) {
4592:     llvm::Value *Arg = EmitScalarExpr(E->getArg(0));
4593:     return Builder.CreateCall(CGM.getIntrinsic(Intrinsic::aarch64_cls), Arg,
4594:                               "cls");
4595:   }
4596:   if (BuiltinID == clang::AArch64::BI__builtin_arm_cls64) {
4597:     llvm::Value *Arg = EmitScalarExpr(E->getArg(0));
4598:     return Builder.CreateCall(CGM.getIntrinsic(Intrinsic::aarch64_cls64), Arg,
4599:                               "cls");
4600:   }
4601: 
4602:   if (BuiltinID == clang::AArch64::BI__builtin_arm_rint32zf ||
4603:       BuiltinID == clang::AArch64::BI__builtin_arm_rint32z) {
4604:     llvm::Value *Arg = EmitScalarExpr(E->getArg(0));
4605:     llvm::Type *Ty = Arg->getType();
4606:     return Builder.CreateCall(CGM.getIntrinsic(Intrinsic::aarch64_frint32z, Ty),
4607:                               Arg, "frint32z");
4608:   }
4609: 
4610:   if (BuiltinID == clang::AArch64::BI__builtin_arm_rint64zf ||
4611:       BuiltinID == clang::AArch64::BI__builtin_arm_rint64z) {
4612:     llvm::Value *Arg = EmitScalarExpr(E->getArg(0));
4613:     llvm::Type *Ty = Arg->getType();
4614:     return Builder.CreateCall(CGM.getIntrinsic(Intrinsic::aarch64_frint64z, Ty),
4615:                               Arg, "frint64z");
4616:   }
4617: 
4618:   if (BuiltinID == clang::AArch64::BI__builtin_arm_rint32xf ||
4619:       BuiltinID == clang::AArch64::BI__builtin_arm_rint32x) {
4620:     llvm::Value *Arg = EmitScalarExpr(E->getArg(0));
```
- **EN**: This block uses control flow (if) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（if）细化 目标内建函数降级 行为。

### Lines 4621-4650
```cpp
4621:     llvm::Type *Ty = Arg->getType();
4622:     return Builder.CreateCall(CGM.getIntrinsic(Intrinsic::aarch64_frint32x, Ty),
4623:                               Arg, "frint32x");
4624:   }
4625: 
4626:   if (BuiltinID == clang::AArch64::BI__builtin_arm_rint64xf ||
4627:       BuiltinID == clang::AArch64::BI__builtin_arm_rint64x) {
4628:     llvm::Value *Arg = EmitScalarExpr(E->getArg(0));
4629:     llvm::Type *Ty = Arg->getType();
4630:     return Builder.CreateCall(CGM.getIntrinsic(Intrinsic::aarch64_frint64x, Ty),
4631:                               Arg, "frint64x");
4632:   }
4633: 
4634:   if (BuiltinID == clang::AArch64::BI__builtin_arm_jcvt) {
4635:     assert((getContext().getTypeSize(E->getType()) == 32) &&
4636:            "__jcvt of unusual size!");
4637:     llvm::Value *Arg = EmitScalarExpr(E->getArg(0));
4638:     return Builder.CreateCall(
4639:         CGM.getIntrinsic(Intrinsic::aarch64_fjcvtzs), Arg);
4640:   }
4641: 
4642:   if (BuiltinID == clang::AArch64::BI__builtin_arm_ld64b ||
4643:       BuiltinID == clang::AArch64::BI__builtin_arm_st64b ||
4644:       BuiltinID == clang::AArch64::BI__builtin_arm_st64bv ||
4645:       BuiltinID == clang::AArch64::BI__builtin_arm_st64bv0) {
4646:     llvm::Value *MemAddr = EmitScalarExpr(E->getArg(0));
4647:     llvm::Value *ValPtr = EmitScalarExpr(E->getArg(1));
4648: 
4649:     if (BuiltinID == clang::AArch64::BI__builtin_arm_ld64b) {
4650:       // Load from the address via an LLVM intrinsic, receiving a
```
- **EN**: This block uses control flow (if) to specialize target builtin lowering; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块通过控制流（if）细化 目标内建函数降级 行为；使用断言或不可达标记保护关键不变量。

### Lines 4651-4680
```cpp
4651:       // tuple of 8 i64 words, and store each one to ValPtr.
4652:       Function *F = CGM.getIntrinsic(Intrinsic::aarch64_ld64b);
4653:       llvm::Value *Val = Builder.CreateCall(F, MemAddr);
4654:       llvm::Value *ToRet;
4655:       for (size_t i = 0; i < 8; i++) {
4656:         llvm::Value *ValOffsetPtr =
4657:             Builder.CreateGEP(Int64Ty, ValPtr, Builder.getInt32(i));
4658:         Address Addr =
4659:             Address(ValOffsetPtr, Int64Ty, CharUnits::fromQuantity(8));
4660:         ToRet = Builder.CreateStore(Builder.CreateExtractValue(Val, i), Addr);
4661:       }
4662:       return ToRet;
4663:     }
4664: 
4665:     // Load 8 i64 words from ValPtr, and store them to the address
4666:     // via an LLVM intrinsic.
4667:     SmallVector<llvm::Value *, 9> Args;
4668:     Args.push_back(MemAddr);
4669:     for (size_t i = 0; i < 8; i++) {
4670:       llvm::Value *ValOffsetPtr =
4671:           Builder.CreateGEP(Int64Ty, ValPtr, Builder.getInt32(i));
4672:       Address Addr = Address(ValOffsetPtr, Int64Ty, CharUnits::fromQuantity(8));
4673:       Args.push_back(Builder.CreateLoad(Addr));
4674:     }
4675: 
4676:     auto Intr = (BuiltinID == clang::AArch64::BI__builtin_arm_st64b
4677:                      ? Intrinsic::aarch64_st64b
4678:                  : BuiltinID == clang::AArch64::BI__builtin_arm_st64bv
4679:                      ? Intrinsic::aarch64_st64bv
4680:                      : Intrinsic::aarch64_st64bv0);
```
- **EN**: This block defines callable entry points like `Address`; uses control flow (for) to specialize target builtin lowering.
- **CN**: 该代码块定义可调用入口，例如 `Address`；通过控制流（for）细化 目标内建函数降级 行为。

### Lines 4681-4710
```cpp
4681:     Function *F = CGM.getIntrinsic(Intr);
4682:     return Builder.CreateCall(F, Args);
4683:   }
4684: 
4685:   if (BuiltinID == clang::AArch64::BI__builtin_arm_rndr ||
4686:       BuiltinID == clang::AArch64::BI__builtin_arm_rndrrs) {
4687: 
4688:     auto Intr = (BuiltinID == clang::AArch64::BI__builtin_arm_rndr
4689:                      ? Intrinsic::aarch64_rndr
4690:                      : Intrinsic::aarch64_rndrrs);
4691:     Function *F = CGM.getIntrinsic(Intr);
4692:     llvm::Value *Val = Builder.CreateCall(F);
4693:     Value *RandomValue = Builder.CreateExtractValue(Val, 0);
4694:     Value *Status = Builder.CreateExtractValue(Val, 1);
4695: 
4696:     Address MemAddress = EmitPointerWithAlignment(E->getArg(0));
4697:     Builder.CreateStore(RandomValue, MemAddress);
4698:     Status = Builder.CreateZExt(Status, Int32Ty);
4699:     return Status;
4700:   }
4701: 
4702:   if (BuiltinID == clang::AArch64::BI__clear_cache) {
4703:     assert(E->getNumArgs() == 2 && "__clear_cache takes 2 arguments");
4704:     const FunctionDecl *FD = E->getDirectCallee();
4705:     Value *Ops[2];
4706:     for (unsigned i = 0; i < 2; i++)
4707:       Ops[i] = EmitScalarExpr(E->getArg(i));
4708:     llvm::Type *Ty = CGM.getTypes().ConvertType(FD->getType());
4709:     llvm::FunctionType *FTy = cast<llvm::FunctionType>(Ty);
4710:     StringRef Name = FD->getName();
```
- **EN**: This block uses control flow (if, for) to specialize target builtin lowering; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块通过控制流（if, for）细化 目标内建函数降级 行为；使用断言或不可达标记保护关键不变量。

### Lines 4711-4740
```cpp
4711:     return EmitNounwindRuntimeCall(CGM.CreateRuntimeFunction(FTy, Name), Ops);
4712:   }
4713: 
4714:   if ((BuiltinID == clang::AArch64::BI__builtin_arm_ldrex ||
4715:        BuiltinID == clang::AArch64::BI__builtin_arm_ldaex) &&
4716:       getContext().getTypeSize(E->getType()) == 128) {
4717:     Function *F =
4718:         CGM.getIntrinsic(BuiltinID == clang::AArch64::BI__builtin_arm_ldaex
4719:                              ? Intrinsic::aarch64_ldaxp
4720:                              : Intrinsic::aarch64_ldxp);
4721: 
4722:     Value *LdPtr = EmitScalarExpr(E->getArg(0));
4723:     Value *Val = Builder.CreateCall(F, LdPtr, "ldxp");
4724: 
4725:     Value *Val0 = Builder.CreateExtractValue(Val, 1);
4726:     Value *Val1 = Builder.CreateExtractValue(Val, 0);
4727:     llvm::Type *Int128Ty = llvm::IntegerType::get(getLLVMContext(), 128);
4728:     Val0 = Builder.CreateZExt(Val0, Int128Ty);
4729:     Val1 = Builder.CreateZExt(Val1, Int128Ty);
4730: 
4731:     Value *ShiftCst = llvm::ConstantInt::get(Int128Ty, 64);
4732:     Val = Builder.CreateShl(Val0, ShiftCst, "shl", true /* nuw */);
4733:     Val = Builder.CreateOr(Val, Val1);
4734:     return Builder.CreateBitCast(Val, ConvertType(E->getType()));
4735:   } else if (BuiltinID == clang::AArch64::BI__builtin_arm_ldrex ||
4736:              BuiltinID == clang::AArch64::BI__builtin_arm_ldaex) {
4737:     Value *LoadAddr = EmitScalarExpr(E->getArg(0));
4738: 
4739:     QualType Ty = E->getType();
4740:     llvm::Type *RealResTy = ConvertType(Ty);
```
- **EN**: This block defines callable entry points like `EmitNounwindRuntimeCall`; uses control flow (if) to specialize target builtin lowering.
- **CN**: 该代码块定义可调用入口，例如 `EmitNounwindRuntimeCall`；通过控制流（if）细化 目标内建函数降级 行为。

### Lines 4741-4770
```cpp
4741:     llvm::Type *IntTy =
4742:         llvm::IntegerType::get(getLLVMContext(), getContext().getTypeSize(Ty));
4743: 
4744:     Function *F =
4745:         CGM.getIntrinsic(BuiltinID == clang::AArch64::BI__builtin_arm_ldaex
4746:                              ? Intrinsic::aarch64_ldaxr
4747:                              : Intrinsic::aarch64_ldxr,
4748:                          DefaultPtrTy);
4749:     CallInst *Val = Builder.CreateCall(F, LoadAddr, "ldxr");
4750:     Val->addParamAttr(
4751:         0, Attribute::get(getLLVMContext(), Attribute::ElementType, IntTy));
4752: 
4753:     if (RealResTy->isPointerTy())
4754:       return Builder.CreateIntToPtr(Val, RealResTy);
4755: 
4756:     llvm::Type *IntResTy = llvm::IntegerType::get(
4757:         getLLVMContext(), CGM.getDataLayout().getTypeSizeInBits(RealResTy));
4758:     return Builder.CreateBitCast(Builder.CreateTruncOrBitCast(Val, IntResTy),
4759:                                  RealResTy);
4760:   }
4761: 
4762:   if ((BuiltinID == clang::AArch64::BI__builtin_arm_strex ||
4763:        BuiltinID == clang::AArch64::BI__builtin_arm_stlex) &&
4764:       getContext().getTypeSize(E->getArg(0)->getType()) == 128) {
4765:     Function *F =
4766:         CGM.getIntrinsic(BuiltinID == clang::AArch64::BI__builtin_arm_stlex
4767:                              ? Intrinsic::aarch64_stlxp
4768:                              : Intrinsic::aarch64_stxp);
4769:     llvm::Type *STy = llvm::StructType::get(Int64Ty, Int64Ty);
4770: 
```
- **EN**: This block defines callable entry points like `get`, `getLLVMContext`; uses control flow (if) to specialize target builtin lowering.
- **CN**: 该代码块定义可调用入口，例如 `get`, `getLLVMContext`；通过控制流（if）细化 目标内建函数降级 行为。

### Lines 4771-4800
```cpp
4771:     Address Tmp = CreateMemTempWithoutCast(E->getArg(0)->getType());
4772:     EmitAnyExprToMem(E->getArg(0), Tmp, Qualifiers(), /*init*/ true);
4773: 
4774:     Tmp = Tmp.withElementType(STy);
4775:     llvm::Value *Val = Builder.CreateLoad(Tmp);
4776: 
4777:     Value *Arg0 = Builder.CreateExtractValue(Val, 0);
4778:     Value *Arg1 = Builder.CreateExtractValue(Val, 1);
4779:     Value *StPtr = EmitScalarExpr(E->getArg(1));
4780:     return Builder.CreateCall(F, {Arg0, Arg1, StPtr}, "stxp");
4781:   }
4782: 
4783:   if (BuiltinID == clang::AArch64::BI__builtin_arm_strex ||
4784:       BuiltinID == clang::AArch64::BI__builtin_arm_stlex) {
4785:     Value *StoreVal = EmitScalarExpr(E->getArg(0));
4786:     Value *StoreAddr = EmitScalarExpr(E->getArg(1));
4787: 
4788:     QualType Ty = E->getArg(0)->getType();
4789:     llvm::Type *StoreTy =
4790:         llvm::IntegerType::get(getLLVMContext(), getContext().getTypeSize(Ty));
4791: 
4792:     if (StoreVal->getType()->isPointerTy())
4793:       StoreVal = Builder.CreatePtrToInt(StoreVal, Int64Ty);
4794:     else {
4795:       llvm::Type *IntTy = llvm::IntegerType::get(
4796:           getLLVMContext(),
4797:           CGM.getDataLayout().getTypeSizeInBits(StoreVal->getType()));
4798:       StoreVal = Builder.CreateBitCast(StoreVal, IntTy);
4799:       StoreVal = Builder.CreateZExtOrBitCast(StoreVal, Int64Ty);
4800:     }
```
- **EN**: This block defines callable entry points like `EmitAnyExprToMem`, `get`, `getLLVMContext`; uses control flow (if) to specialize target builtin lowering.
- **CN**: 该代码块定义可调用入口，例如 `EmitAnyExprToMem`, `get`, `getLLVMContext`；通过控制流（if）细化 目标内建函数降级 行为。

### Lines 4801-4830
```cpp
4801: 
4802:     Function *F =
4803:         CGM.getIntrinsic(BuiltinID == clang::AArch64::BI__builtin_arm_stlex
4804:                              ? Intrinsic::aarch64_stlxr
4805:                              : Intrinsic::aarch64_stxr,
4806:                          StoreAddr->getType());
4807:     CallInst *CI = Builder.CreateCall(F, {StoreVal, StoreAddr}, "stxr");
4808:     CI->addParamAttr(
4809:         1, Attribute::get(getLLVMContext(), Attribute::ElementType, StoreTy));
4810:     return CI;
4811:   }
4812: 
4813:   if (BuiltinID == clang::AArch64::BI__getReg) {
4814:     Expr::EvalResult Result;
4815:     if (!E->getArg(0)->EvaluateAsInt(Result, CGM.getContext()))
4816:       llvm_unreachable("Sema will ensure that the parameter is constant");
4817: 
4818:     llvm::APSInt Value = Result.Val.getInt();
4819:     LLVMContext &Context = CGM.getLLVMContext();
4820:     std::string Reg = Value == 31 ? "sp" : "x" + toString(Value, 10);
4821: 
4822:     llvm::Metadata *Ops[] = {llvm::MDString::get(Context, Reg)};
4823:     llvm::MDNode *RegName = llvm::MDNode::get(Context, Ops);
4824:     llvm::Value *Metadata = llvm::MetadataAsValue::get(Context, RegName);
4825: 
4826:     llvm::Function *F =
4827:         CGM.getIntrinsic(Intrinsic::read_register, {Int64Ty});
4828:     return Builder.CreateCall(F, Metadata);
4829:   }
4830: 
```
- **EN**: This block defines callable entry points like `get`; uses control flow (if) to specialize target builtin lowering; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块定义可调用入口，例如 `get`；通过控制流（if）细化 目标内建函数降级 行为；使用断言或不可达标记保护关键不变量。

### Lines 4831-4860
```cpp
4831:   if (BuiltinID == clang::AArch64::BI__break) {
4832:     Expr::EvalResult Result;
4833:     if (!E->getArg(0)->EvaluateAsInt(Result, CGM.getContext()))
4834:       llvm_unreachable("Sema will ensure that the parameter is constant");
4835: 
4836:     llvm::Function *F = CGM.getIntrinsic(Intrinsic::aarch64_break);
4837:     return Builder.CreateCall(F, {EmitScalarExpr(E->getArg(0))});
4838:   }
4839: 
4840:   if (BuiltinID == clang::AArch64::BI__builtin_arm_clrex) {
4841:     Function *F = CGM.getIntrinsic(Intrinsic::aarch64_clrex);
4842:     return Builder.CreateCall(F);
4843:   }
4844: 
4845:   if (BuiltinID == clang::AArch64::BI_ReadWriteBarrier)
4846:     return Builder.CreateFence(llvm::AtomicOrdering::SequentiallyConsistent,
4847:                                llvm::SyncScope::SingleThread);
4848: 
4849:   // CRC32
4850:   Intrinsic::ID CRCIntrinsicID = Intrinsic::not_intrinsic;
4851:   switch (BuiltinID) {
4852:   case clang::AArch64::BI__builtin_arm_crc32b:
4853:     CRCIntrinsicID = Intrinsic::aarch64_crc32b; break;
4854:   case clang::AArch64::BI__builtin_arm_crc32cb:
4855:     CRCIntrinsicID = Intrinsic::aarch64_crc32cb; break;
4856:   case clang::AArch64::BI__builtin_arm_crc32h:
4857:     CRCIntrinsicID = Intrinsic::aarch64_crc32h; break;
4858:   case clang::AArch64::BI__builtin_arm_crc32ch:
4859:     CRCIntrinsicID = Intrinsic::aarch64_crc32ch; break;
4860:   case clang::AArch64::BI__builtin_arm_crc32w:
```
- **EN**: This block uses control flow (if, switch, case) to specialize target builtin lowering; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块通过控制流（if, switch, case）细化 目标内建函数降级 行为；使用断言或不可达标记保护关键不变量。

### Lines 4861-4890
```cpp
4861:     CRCIntrinsicID = Intrinsic::aarch64_crc32w; break;
4862:   case clang::AArch64::BI__builtin_arm_crc32cw:
4863:     CRCIntrinsicID = Intrinsic::aarch64_crc32cw; break;
4864:   case clang::AArch64::BI__builtin_arm_crc32d:
4865:     CRCIntrinsicID = Intrinsic::aarch64_crc32x; break;
4866:   case clang::AArch64::BI__builtin_arm_crc32cd:
4867:     CRCIntrinsicID = Intrinsic::aarch64_crc32cx; break;
4868:   }
4869: 
4870:   if (CRCIntrinsicID != Intrinsic::not_intrinsic) {
4871:     Value *Arg0 = EmitScalarExpr(E->getArg(0));
4872:     Value *Arg1 = EmitScalarExpr(E->getArg(1));
4873:     Function *F = CGM.getIntrinsic(CRCIntrinsicID);
4874: 
4875:     llvm::Type *DataTy = F->getFunctionType()->getParamType(1);
4876:     Arg1 = Builder.CreateZExtOrBitCast(Arg1, DataTy);
4877: 
4878:     return Builder.CreateCall(F, {Arg0, Arg1});
4879:   }
4880: 
4881:   // Memory Operations (MOPS)
4882:   if (BuiltinID == AArch64::BI__builtin_arm_mops_memset_tag) {
4883:     Value *Dst = EmitScalarExpr(E->getArg(0));
4884:     Value *Val = EmitScalarExpr(E->getArg(1));
4885:     Value *Size = EmitScalarExpr(E->getArg(2));
4886:     Val = Builder.CreateTrunc(Val, Int8Ty);
4887:     Size = Builder.CreateIntCast(Size, Int64Ty, false);
4888:     return Builder.CreateCall(
4889:         CGM.getIntrinsic(Intrinsic::aarch64_mops_memset_tag), {Dst, Val, Size});
4890:   }
```
- **EN**: This block uses control flow (if, case) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（if, case）细化 目标内建函数降级 行为。

### Lines 4891-4920
```cpp
4891: 
4892:   if (BuiltinID == AArch64::BI__builtin_arm_range_prefetch ||
4893:       BuiltinID == AArch64::BI__builtin_arm_range_prefetch_x)
4894:     return EmitRangePrefetchBuiltin(*this, BuiltinID, E);
4895: 
4896:   // Memory Tagging Extensions (MTE) Intrinsics
4897:   Intrinsic::ID MTEIntrinsicID = Intrinsic::not_intrinsic;
4898:   switch (BuiltinID) {
4899:   case clang::AArch64::BI__builtin_arm_irg:
4900:     MTEIntrinsicID = Intrinsic::aarch64_irg; break;
4901:   case clang::AArch64::BI__builtin_arm_addg:
4902:     MTEIntrinsicID = Intrinsic::aarch64_addg; break;
4903:   case clang::AArch64::BI__builtin_arm_gmi:
4904:     MTEIntrinsicID = Intrinsic::aarch64_gmi; break;
4905:   case clang::AArch64::BI__builtin_arm_ldg:
4906:     MTEIntrinsicID = Intrinsic::aarch64_ldg; break;
4907:   case clang::AArch64::BI__builtin_arm_stg:
4908:     MTEIntrinsicID = Intrinsic::aarch64_stg; break;
4909:   case clang::AArch64::BI__builtin_arm_subp:
4910:     MTEIntrinsicID = Intrinsic::aarch64_subp; break;
4911:   }
4912: 
4913:   if (MTEIntrinsicID != Intrinsic::not_intrinsic) {
4914:     if (MTEIntrinsicID == Intrinsic::aarch64_irg) {
4915:       Value *Pointer = EmitScalarExpr(E->getArg(0));
4916:       Value *Mask = EmitScalarExpr(E->getArg(1));
4917: 
4918:       Mask = Builder.CreateZExt(Mask, Int64Ty);
4919:       return Builder.CreateCall(CGM.getIntrinsic(MTEIntrinsicID),
4920:                                 {Pointer, Mask});
```
- **EN**: This block uses control flow (if, switch, case) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（if, switch, case）细化 目标内建函数降级 行为。

### Lines 4921-4950
```cpp
4921:     }
4922:     if (MTEIntrinsicID == Intrinsic::aarch64_addg) {
4923:       Value *Pointer = EmitScalarExpr(E->getArg(0));
4924:       Value *TagOffset = EmitScalarExpr(E->getArg(1));
4925: 
4926:       TagOffset = Builder.CreateZExt(TagOffset, Int64Ty);
4927:       return Builder.CreateCall(CGM.getIntrinsic(MTEIntrinsicID),
4928:                                 {Pointer, TagOffset});
4929:     }
4930:     if (MTEIntrinsicID == Intrinsic::aarch64_gmi) {
4931:       Value *Pointer = EmitScalarExpr(E->getArg(0));
4932:       Value *ExcludedMask = EmitScalarExpr(E->getArg(1));
4933: 
4934:       ExcludedMask = Builder.CreateZExt(ExcludedMask, Int64Ty);
4935:       return Builder.CreateCall(
4936:                        CGM.getIntrinsic(MTEIntrinsicID), {Pointer, ExcludedMask});
4937:     }
4938:     // Although it is possible to supply a different return
4939:     // address (first arg) to this intrinsic, for now we set
4940:     // return address same as input address.
4941:     if (MTEIntrinsicID == Intrinsic::aarch64_ldg) {
4942:       Value *TagAddress = EmitScalarExpr(E->getArg(0));
4943:       return Builder.CreateCall(CGM.getIntrinsic(MTEIntrinsicID),
4944:                                 {TagAddress, TagAddress});
4945:     }
4946:     // Although it is possible to supply a different tag (to set)
4947:     // to this intrinsic (as first arg), for now we supply
4948:     // the tag that is in input address arg (common use case).
4949:     if (MTEIntrinsicID == Intrinsic::aarch64_stg) {
4950:       Value *TagAddress = EmitScalarExpr(E->getArg(0));
```
- **EN**: This block uses control flow (if) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（if）细化 目标内建函数降级 行为。

### Lines 4951-4980
```cpp
4951:       return Builder.CreateCall(CGM.getIntrinsic(MTEIntrinsicID),
4952:                                 {TagAddress, TagAddress});
4953:     }
4954:     if (MTEIntrinsicID == Intrinsic::aarch64_subp) {
4955:       Value *PointerA = EmitScalarExpr(E->getArg(0));
4956:       Value *PointerB = EmitScalarExpr(E->getArg(1));
4957:       return Builder.CreateCall(
4958:                        CGM.getIntrinsic(MTEIntrinsicID), {PointerA, PointerB});
4959:     }
4960:   }
4961: 
4962:   if (BuiltinID == clang::AArch64::BI__builtin_arm_rsr ||
4963:       BuiltinID == clang::AArch64::BI__builtin_arm_rsr64 ||
4964:       BuiltinID == clang::AArch64::BI__builtin_arm_rsr128 ||
4965:       BuiltinID == clang::AArch64::BI__builtin_arm_rsrp ||
4966:       BuiltinID == clang::AArch64::BI__builtin_arm_wsr ||
4967:       BuiltinID == clang::AArch64::BI__builtin_arm_wsr64 ||
4968:       BuiltinID == clang::AArch64::BI__builtin_arm_wsr128 ||
4969:       BuiltinID == clang::AArch64::BI__builtin_arm_wsrp) {
4970: 
4971:     SpecialRegisterAccessKind AccessKind = Write;
4972:     if (BuiltinID == clang::AArch64::BI__builtin_arm_rsr ||
4973:         BuiltinID == clang::AArch64::BI__builtin_arm_rsr64 ||
4974:         BuiltinID == clang::AArch64::BI__builtin_arm_rsr128 ||
4975:         BuiltinID == clang::AArch64::BI__builtin_arm_rsrp)
4976:       AccessKind = VolatileRead;
4977: 
4978:     bool IsPointerBuiltin = BuiltinID == clang::AArch64::BI__builtin_arm_rsrp ||
4979:                             BuiltinID == clang::AArch64::BI__builtin_arm_wsrp;
4980: 
```
- **EN**: This block uses control flow (if) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（if）细化 目标内建函数降级 行为。

### Lines 4981-5010
```cpp
4981:     bool Is32Bit = BuiltinID == clang::AArch64::BI__builtin_arm_rsr ||
4982:                    BuiltinID == clang::AArch64::BI__builtin_arm_wsr;
4983: 
4984:     bool Is128Bit = BuiltinID == clang::AArch64::BI__builtin_arm_rsr128 ||
4985:                     BuiltinID == clang::AArch64::BI__builtin_arm_wsr128;
4986: 
4987:     llvm::Type *ValueType;
4988:     llvm::Type *RegisterType = Int64Ty;
4989:     if (Is32Bit) {
4990:       ValueType = Int32Ty;
4991:     } else if (Is128Bit) {
4992:       llvm::Type *Int128Ty =
4993:           llvm::IntegerType::getInt128Ty(CGM.getLLVMContext());
4994:       ValueType = Int128Ty;
4995:       RegisterType = Int128Ty;
4996:     } else if (IsPointerBuiltin) {
4997:       ValueType = VoidPtrTy;
4998:     } else {
4999:       ValueType = Int64Ty;
5000:     };
5001: 
5002:     return EmitSpecialRegisterBuiltin(*this, E, RegisterType, ValueType,
5003:                                       AccessKind);
5004:   }
5005: 
5006:   if (BuiltinID == clang::AArch64::BI_ReadStatusReg ||
5007:       BuiltinID == clang::AArch64::BI_WriteStatusReg) {
5008:     LLVMContext &Context = CGM.getLLVMContext();
5009: 
5010:     unsigned SysReg =
```
- **EN**: This block defines callable entry points like `getInt128Ty`, `EmitSpecialRegisterBuiltin`; uses control flow (if) to specialize target builtin lowering.
- **CN**: 该代码块定义可调用入口，例如 `getInt128Ty`, `EmitSpecialRegisterBuiltin`；通过控制流（if）细化 目标内建函数降级 行为。

### Lines 5011-5040
```cpp
5011:       E->getArg(0)->EvaluateKnownConstInt(getContext()).getZExtValue();
5012: 
5013:     std::string SysRegStr;
5014:     llvm::raw_string_ostream(SysRegStr)
5015:         << (0b10 | SysReg >> 14) << ":" << ((SysReg >> 11) & 7) << ":"
5016:         << ((SysReg >> 7) & 15) << ":" << ((SysReg >> 3) & 15) << ":"
5017:         << (SysReg & 7);
5018: 
5019:     llvm::Metadata *Ops[] = { llvm::MDString::get(Context, SysRegStr) };
5020:     llvm::MDNode *RegName = llvm::MDNode::get(Context, Ops);
5021:     llvm::Value *Metadata = llvm::MetadataAsValue::get(Context, RegName);
5022: 
5023:     llvm::Type *RegisterType = Int64Ty;
5024:     llvm::Type *Types[] = { RegisterType };
5025: 
5026:     if (BuiltinID == clang::AArch64::BI_ReadStatusReg) {
5027:       llvm::Function *F = CGM.getIntrinsic(Intrinsic::read_register, Types);
5028: 
5029:       return Builder.CreateCall(F, Metadata);
5030:     }
5031: 
5032:     llvm::Function *F = CGM.getIntrinsic(Intrinsic::write_register, Types);
5033:     llvm::Value *ArgValue = EmitScalarExpr(E->getArg(1));
5034:     llvm::Value *Result = Builder.CreateCall(F, {Metadata, ArgValue});
5035: 
5036:     return Result;
5037:   }
5038: 
5039:   if (BuiltinID == clang::AArch64::BI__sys) {
5040:     unsigned SysReg =
```
- **EN**: This block defines callable entry points like `raw_string_ostream`; uses control flow (if) to specialize target builtin lowering.
- **CN**: 该代码块定义可调用入口，例如 `raw_string_ostream`；通过控制流（if）细化 目标内建函数降级 行为。

### Lines 5041-5070
```cpp
5041:         E->getArg(0)->EvaluateKnownConstInt(getContext()).getZExtValue();
5042:     const unsigned Op1 = SysReg >> 11;
5043:     const unsigned CRn = (SysReg >> 7) & 0xf;
5044:     const unsigned CRm = (SysReg >> 3) & 0xf;
5045:     const unsigned Op2 = SysReg & 0x7;
5046: 
5047:     Builder.CreateCall(CGM.getIntrinsic(Intrinsic::aarch64_sys),
5048:                        {Builder.getInt32(Op1), Builder.getInt32(CRn),
5049:                         Builder.getInt32(CRm), Builder.getInt32(Op2),
5050:                         EmitScalarExpr(E->getArg(1))});
5051: 
5052:     // Return 0 for convenience, even though MSVC returns some other undefined
5053:     // value.
5054:     return ConstantInt::get(Builder.getInt32Ty(), 0);
5055:   }
5056: 
5057:   if (BuiltinID == clang::AArch64::BI_AddressOfReturnAddress) {
5058:     llvm::Function *F =
5059:         CGM.getIntrinsic(Intrinsic::addressofreturnaddress, AllocaInt8PtrTy);
5060:     return Builder.CreateCall(F);
5061:   }
5062: 
5063:   if (BuiltinID == clang::AArch64::BI__builtin_sponentry) {
5064:     llvm::Function *F = CGM.getIntrinsic(Intrinsic::sponentry, AllocaInt8PtrTy);
5065:     return Builder.CreateCall(F);
5066:   }
5067: 
5068:   if (BuiltinID == clang::AArch64::BI__mulh ||
5069:       BuiltinID == clang::AArch64::BI__umulh) {
5070:     llvm::Type *ResType = ConvertType(E->getType());
```
- **EN**: This block defines callable entry points like `get`; uses control flow (if) to specialize target builtin lowering.
- **CN**: 该代码块定义可调用入口，例如 `get`；通过控制流（if）细化 目标内建函数降级 行为。

### Lines 5071-5100
```cpp
5071:     llvm::Type *Int128Ty = llvm::IntegerType::get(getLLVMContext(), 128);
5072: 
5073:     bool IsSigned = BuiltinID == clang::AArch64::BI__mulh;
5074:     Value *LHS =
5075:         Builder.CreateIntCast(EmitScalarExpr(E->getArg(0)), Int128Ty, IsSigned);
5076:     Value *RHS =
5077:         Builder.CreateIntCast(EmitScalarExpr(E->getArg(1)), Int128Ty, IsSigned);
5078: 
5079:     Value *MulResult, *HigherBits;
5080:     if (IsSigned) {
5081:       MulResult = Builder.CreateNSWMul(LHS, RHS);
5082:       HigherBits = Builder.CreateAShr(MulResult, 64);
5083:     } else {
5084:       MulResult = Builder.CreateNUWMul(LHS, RHS);
5085:       HigherBits = Builder.CreateLShr(MulResult, 64);
5086:     }
5087:     HigherBits = Builder.CreateIntCast(HigherBits, ResType, IsSigned);
5088: 
5089:     return HigherBits;
5090:   }
5091: 
5092:   if (BuiltinID == AArch64::BI__writex18byte ||
5093:       BuiltinID == AArch64::BI__writex18word ||
5094:       BuiltinID == AArch64::BI__writex18dword ||
5095:       BuiltinID == AArch64::BI__writex18qword) {
5096:     // Process the args first
5097:     Value *OffsetArg = EmitScalarExpr(E->getArg(0));
5098:     Value *DataArg = EmitScalarExpr(E->getArg(1));
5099: 
5100:     // Read x18 as i8*
```
- **EN**: This block uses control flow (if) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（if）细化 目标内建函数降级 行为。

### Lines 5101-5130
```cpp
5101:     llvm::Value *X18 = readX18AsPtr(*this);
5102: 
5103:     // Store val at x18 + offset
5104:     Value *Offset = Builder.CreateZExt(OffsetArg, Int64Ty);
5105:     Value *Ptr = Builder.CreateGEP(Int8Ty, X18, Offset);
5106:     StoreInst *Store =
5107:         Builder.CreateAlignedStore(DataArg, Ptr, CharUnits::One());
5108:     return Store;
5109:   }
5110: 
5111:   if (BuiltinID == AArch64::BI__readx18byte ||
5112:       BuiltinID == AArch64::BI__readx18word ||
5113:       BuiltinID == AArch64::BI__readx18dword ||
5114:       BuiltinID == AArch64::BI__readx18qword) {
5115:     // Process the args first
5116:     Value *OffsetArg = EmitScalarExpr(E->getArg(0));
5117: 
5118:     // Read x18 as i8*
5119:     llvm::Value *X18 = readX18AsPtr(*this);
5120: 
5121:     // Load x18 + offset
5122:     Value *Offset = Builder.CreateZExt(OffsetArg, Int64Ty);
5123:     Value *Ptr = Builder.CreateGEP(Int8Ty, X18, Offset);
5124:     llvm::Type *IntTy = ConvertType(E->getType());
5125:     LoadInst *Load = Builder.CreateAlignedLoad(IntTy, Ptr, CharUnits::One());
5126:     return Load;
5127:   }
5128: 
5129:   if (BuiltinID == AArch64::BI__addx18byte ||
5130:       BuiltinID == AArch64::BI__addx18word ||
```
- **EN**: This block uses control flow (if) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（if）细化 目标内建函数降级 行为。

### Lines 5131-5160
```cpp
5131:       BuiltinID == AArch64::BI__addx18dword ||
5132:       BuiltinID == AArch64::BI__addx18qword ||
5133:       BuiltinID == AArch64::BI__incx18byte ||
5134:       BuiltinID == AArch64::BI__incx18word ||
5135:       BuiltinID == AArch64::BI__incx18dword ||
5136:       BuiltinID == AArch64::BI__incx18qword) {
5137:     llvm::Type *IntTy;
5138:     bool isIncrement;
5139:     switch (BuiltinID) {
5140:     case AArch64::BI__incx18byte:
5141:       IntTy = Int8Ty;
5142:       isIncrement = true;
5143:       break;
5144:     case AArch64::BI__incx18word:
5145:       IntTy = Int16Ty;
5146:       isIncrement = true;
5147:       break;
5148:     case AArch64::BI__incx18dword:
5149:       IntTy = Int32Ty;
5150:       isIncrement = true;
5151:       break;
5152:     case AArch64::BI__incx18qword:
5153:       IntTy = Int64Ty;
5154:       isIncrement = true;
5155:       break;
5156:     default:
5157:       IntTy = ConvertType(E->getArg(1)->getType());
5158:       isIncrement = false;
5159:       break;
5160:     }
```
- **EN**: This block uses control flow (switch, case) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（switch, case）细化 目标内建函数降级 行为。

### Lines 5161-5190
```cpp
5161:     // Process the args first
5162:     Value *OffsetArg = EmitScalarExpr(E->getArg(0));
5163:     Value *ValToAdd =
5164:         isIncrement ? ConstantInt::get(IntTy, 1) : EmitScalarExpr(E->getArg(1));
5165: 
5166:     // Read x18 as i8*
5167:     llvm::Value *X18 = readX18AsPtr(*this);
5168: 
5169:     // Load x18 + offset
5170:     Value *Offset = Builder.CreateZExt(OffsetArg, Int64Ty);
5171:     Value *Ptr = Builder.CreateGEP(Int8Ty, X18, Offset);
5172:     LoadInst *Load = Builder.CreateAlignedLoad(IntTy, Ptr, CharUnits::One());
5173: 
5174:     // Add values
5175:     Value *AddResult = Builder.CreateAdd(Load, ValToAdd);
5176: 
5177:     // Store val at x18 + offset
5178:     StoreInst *Store =
5179:         Builder.CreateAlignedStore(AddResult, Ptr, CharUnits::One());
5180:     return Store;
5181:   }
5182: 
5183:   if (BuiltinID == AArch64::BI_CopyDoubleFromInt64 ||
5184:       BuiltinID == AArch64::BI_CopyFloatFromInt32 ||
5185:       BuiltinID == AArch64::BI_CopyInt32FromFloat ||
5186:       BuiltinID == AArch64::BI_CopyInt64FromDouble) {
5187:     Value *Arg = EmitScalarExpr(E->getArg(0));
5188:     llvm::Type *RetTy = ConvertType(E->getType());
5189:     return Builder.CreateBitCast(Arg, RetTy);
5190:   }
```
- **EN**: This block uses control flow (if) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（if）细化 目标内建函数降级 行为。

### Lines 5191-5220
```cpp
5191: 
5192:   if (BuiltinID == AArch64::BI_CountLeadingOnes ||
5193:       BuiltinID == AArch64::BI_CountLeadingOnes64 ||
5194:       BuiltinID == AArch64::BI_CountLeadingZeros ||
5195:       BuiltinID == AArch64::BI_CountLeadingZeros64) {
5196:     Value *Arg = EmitScalarExpr(E->getArg(0));
5197:     llvm::Type *ArgType = Arg->getType();
5198: 
5199:     if (BuiltinID == AArch64::BI_CountLeadingOnes ||
5200:         BuiltinID == AArch64::BI_CountLeadingOnes64)
5201:       Arg = Builder.CreateXor(Arg, Constant::getAllOnesValue(ArgType));
5202: 
5203:     Function *F = CGM.getIntrinsic(Intrinsic::ctlz, ArgType);
5204:     Value *Result = Builder.CreateCall(F, {Arg, Builder.getInt1(false)});
5205: 
5206:     if (BuiltinID == AArch64::BI_CountLeadingOnes64 ||
5207:         BuiltinID == AArch64::BI_CountLeadingZeros64)
5208:       Result = Builder.CreateTrunc(Result, Builder.getInt32Ty());
5209:     return Result;
5210:   }
5211: 
5212:   if (BuiltinID == AArch64::BI_CountLeadingSigns ||
5213:       BuiltinID == AArch64::BI_CountLeadingSigns64) {
5214:     Value *Arg = EmitScalarExpr(E->getArg(0));
5215: 
5216:     Function *F = (BuiltinID == AArch64::BI_CountLeadingSigns)
5217:                       ? CGM.getIntrinsic(Intrinsic::aarch64_cls)
5218:                       : CGM.getIntrinsic(Intrinsic::aarch64_cls64);
5219: 
5220:     Value *Result = Builder.CreateCall(F, Arg, "cls");
```
- **EN**: This block uses control flow (if) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（if）细化 目标内建函数降级 行为。

### Lines 5221-5250
```cpp
5221:     if (BuiltinID == AArch64::BI_CountLeadingSigns64)
5222:       Result = Builder.CreateTrunc(Result, Builder.getInt32Ty());
5223:     return Result;
5224:   }
5225: 
5226:   if (BuiltinID == AArch64::BI_CountOneBits ||
5227:       BuiltinID == AArch64::BI_CountOneBits64) {
5228:     Value *ArgValue = EmitScalarExpr(E->getArg(0));
5229:     llvm::Type *ArgType = ArgValue->getType();
5230:     Function *F = CGM.getIntrinsic(Intrinsic::ctpop, ArgType);
5231: 
5232:     Value *Result = Builder.CreateCall(F, ArgValue);
5233:     if (BuiltinID == AArch64::BI_CountOneBits64)
5234:       Result = Builder.CreateTrunc(Result, Builder.getInt32Ty());
5235:     return Result;
5236:   }
5237: 
5238:   if (BuiltinID == AArch64::BI__prefetch) {
5239:     Value *Address = EmitScalarExpr(E->getArg(0));
5240:     Value *RW = llvm::ConstantInt::get(Int32Ty, 0);
5241:     Value *Locality = ConstantInt::get(Int32Ty, 3);
5242:     Value *Data = llvm::ConstantInt::get(Int32Ty, 1);
5243:     Function *F = CGM.getIntrinsic(Intrinsic::prefetch, Address->getType());
5244:     return Builder.CreateCall(F, {Address, RW, Locality, Data});
5245:   }
5246: 
5247:   if (BuiltinID == AArch64::BI__hlt) {
5248:     Function *F = CGM.getIntrinsic(Intrinsic::aarch64_hlt);
5249:     Builder.CreateCall(F, {EmitScalarExpr(E->getArg(0))});
5250: 
```
- **EN**: This block uses control flow (if) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（if）细化 目标内建函数降级 行为。

### Lines 5251-5280
```cpp
5251:     // Return 0 for convenience, even though MSVC returns some other undefined
5252:     // value.
5253:     return ConstantInt::get(Builder.getInt32Ty(), 0);
5254:   }
5255: 
5256:   if (BuiltinID == NEON::BI__builtin_neon_vcvth_bf16_f32)
5257:     return Builder.CreateFPTrunc(
5258:         Builder.CreateBitCast(EmitScalarExpr(E->getArg(0)),
5259:                               Builder.getFloatTy()),
5260:         Builder.getBFloatTy());
5261: 
5262:   // Handle MSVC intrinsics before argument evaluation to prevent double
5263:   // evaluation.
5264:   if (std::optional<MSVCIntrin> MsvcIntId =
5265:           translateAarch64ToMsvcIntrin(BuiltinID))
5266:     return EmitMSVCBuiltinExpr(*MsvcIntId, E);
5267: 
5268:   // Some intrinsics are equivalent - if they are use the base intrinsic ID.
5269:   auto It = llvm::find_if(NEONEquivalentIntrinsicMap, [BuiltinID](auto &P) {
5270:     return P.first == BuiltinID;
5271:   });
5272:   if (It != end(NEONEquivalentIntrinsicMap))
5273:     BuiltinID = It->second;
5274: 
5275:   // Check whether this is an SISD builtin.
5276:   auto SISDMap = ArrayRef(AArch64SISDIntrinsicMap);
5277:   const ARMVectorIntrinsicInfo *Builtin = findARMVectorIntrinsicInMap(
5278:       SISDMap, BuiltinID, AArch64SISDIntrinsicsProvenSorted);
5279:   bool IsSISD = (Builtin != nullptr);
5280: 
```
- **EN**: This block defines callable entry points like `get`; uses control flow (if) to specialize target builtin lowering.
- **CN**: 该代码块定义可调用入口，例如 `get`；通过控制流（if）细化 目标内建函数降级 行为。

### Lines 5281-5310
```cpp
5281:   // Find out if any arguments are required to be integer constant
5282:   // expressions.
5283:   unsigned ICEArguments = 0;
5284:   ASTContext::GetBuiltinTypeError Error;
5285:   getContext().GetBuiltinType(BuiltinID, Error, &ICEArguments);
5286:   assert(Error == ASTContext::GE_None && "Should not codegen an error");
5287: 
5288:   llvm::SmallVector<Value*, 4> Ops;
5289:   Address PtrOp0 = Address::invalid();
5290:   // Note the assumption that SISD intrinsics do not contain extra arguments.
5291:   // TODO: Fold this into a single function call instead of, effectively, two
5292:   // separate checks.
5293:   bool HasExtraArg = !IsSISD && HasExtraNeonArgument(BuiltinID);
5294:   unsigned NumArgs = E->getNumArgs() - (HasExtraArg ? 1 : 0);
5295:   for (unsigned i = 0, e = NumArgs; i != e; i++) {
5296:     if (i == 0) {
5297:       switch (BuiltinID) {
5298:       case NEON::BI__builtin_neon_vld1_v:
5299:       case NEON::BI__builtin_neon_vld1q_v:
5300:       case NEON::BI__builtin_neon_vld1_dup_v:
5301:       case NEON::BI__builtin_neon_vld1q_dup_v:
5302:       case NEON::BI__builtin_neon_vld1_lane_v:
5303:       case NEON::BI__builtin_neon_vld1q_lane_v:
5304:       case NEON::BI__builtin_neon_vst1_v:
5305:       case NEON::BI__builtin_neon_vst1q_v:
5306:       case NEON::BI__builtin_neon_vst1_lane_v:
5307:       case NEON::BI__builtin_neon_vst1q_lane_v:
5308:       case NEON::BI__builtin_neon_vldap1_lane_s64:
5309:       case NEON::BI__builtin_neon_vldap1q_lane_s64:
5310:       case NEON::BI__builtin_neon_vstl1_lane_s64:
```
- **EN**: This block defines callable entry points like `getContext`; uses control flow (if, switch, for, case) to specialize target builtin lowering; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块定义可调用入口，例如 `getContext`；通过控制流（if, switch, for, case）细化 目标内建函数降级 行为；使用断言或不可达标记保护关键不变量。

### Lines 5311-5340
```cpp
5311:       case NEON::BI__builtin_neon_vstl1q_lane_s64:
5312:         // Get the alignment for the argument in addition to the value;
5313:         // we'll use it later.
5314:         PtrOp0 = EmitPointerWithAlignment(E->getArg(0));
5315:         Ops.push_back(PtrOp0.emitRawPointer(*this));
5316:         continue;
5317:       }
5318:     }
5319:     Ops.push_back(EmitScalarOrConstFoldImmArg(ICEArguments, i, E));
5320:   }
5321: 
5322:   if (Builtin) {
5323:     Value *Result = EmitCommonNeonSISDBuiltinExpr(*this, *Builtin, Ops, E);
5324:     assert(Result && "SISD intrinsic should have been handled");
5325:     return Result;
5326:   }
5327: 
5328:   const Expr *Arg = E->getArg(E->getNumArgs()-1);
5329:   NeonTypeFlags Type(0);
5330:   if (std::optional<llvm::APSInt> Result =
5331:           Arg->getIntegerConstantExpr(getContext()))
5332:     // Determine the type of this overloaded NEON intrinsic.
5333:     Type = NeonTypeFlags(Result->getZExtValue());
5334: 
5335:   bool usgn = Type.isUnsigned();
5336:   bool quad = Type.isQuad();
5337:   unsigned Int;
5338: 
5339:   // Not all intrinsics handled by the common case work for AArch64 yet, so only
5340:   // defer to common code if it's been added to our special map.
```
- **EN**: This block defines callable entry points like `Type`; uses control flow (if, case) to specialize target builtin lowering; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块定义可调用入口，例如 `Type`；通过控制流（if, case）细化 目标内建函数降级 行为；使用断言或不可达标记保护关键不变量。

### Lines 5341-5370
```cpp
5341:   Builtin = findARMVectorIntrinsicInMap(AArch64SIMDIntrinsicMap, BuiltinID,
5342:                                         AArch64SIMDIntrinsicsProvenSorted);
5343: 
5344:   if (Builtin)
5345:     return EmitCommonNeonBuiltinExpr(
5346:         Builtin->BuiltinID, Builtin->LLVMIntrinsic, Builtin->AltLLVMIntrinsic,
5347:         Builtin->NameHint, Builtin->TypeModifier, E, Ops,
5348:         /*never use addresses*/ Address::invalid(), Address::invalid(), Arch);
5349: 
5350:   if (Value *V = EmitAArch64TblBuiltinExpr(*this, BuiltinID, E, Ops, Arch))
5351:     return V;
5352: 
5353:   // Handle non-overloaded intrinsics first.
5354:   switch (BuiltinID) {
5355:   default: break;
5356:   case NEON::BI__builtin_neon_vabsh_f16:
5357:     return EmitNeonCall(CGM.getIntrinsic(Intrinsic::fabs, HalfTy), Ops, "vabs");
5358:   case NEON::BI__builtin_neon_vaddq_p128: {
5359:     llvm::Type *Ty = GetNeonType(this, NeonTypeFlags::Poly128);
5360:     Ops[0] = Builder.CreateBitCast(Ops[0], Ty);
5361:     Ops[1] = Builder.CreateBitCast(Ops[1], Ty);
5362:     Ops[0] =  Builder.CreateXor(Ops[0], Ops[1]);
5363:     llvm::Type *Int128Ty = llvm::Type::getIntNTy(getLLVMContext(), 128);
5364:     return Builder.CreateBitCast(Ops[0], Int128Ty);
5365:   }
5366:   case NEON::BI__builtin_neon_vldrq_p128: {
5367:     llvm::Type *Int128Ty = llvm::Type::getIntNTy(getLLVMContext(), 128);
5368:     return Builder.CreateAlignedLoad(Int128Ty, Ops[0],
5369:                                      CharUnits::fromQuantity(16));
5370:   }
```
- **EN**: This block defines callable entry points like `EmitNeonCall`, `fromQuantity`; uses control flow (if, switch, case) to specialize target builtin lowering.
- **CN**: 该代码块定义可调用入口，例如 `EmitNeonCall`, `fromQuantity`；通过控制流（if, switch, case）细化 目标内建函数降级 行为。

### Lines 5371-5400
```cpp
5371:   case NEON::BI__builtin_neon_vstrq_p128: {
5372:     return Builder.CreateDefaultAlignedStore(Ops[1], Ops[0]);
5373:   }
5374:   case NEON::BI__builtin_neon_vcvts_f32_u32:
5375:   case NEON::BI__builtin_neon_vcvtd_f64_u64:
5376:     usgn = true;
5377:     [[fallthrough]];
5378:   case NEON::BI__builtin_neon_vcvts_f32_s32:
5379:   case NEON::BI__builtin_neon_vcvtd_f64_s64: {
5380:     bool Is64 = Ops[0]->getType()->getPrimitiveSizeInBits() == 64;
5381:     llvm::Type *InTy = Is64 ? Int64Ty : Int32Ty;
5382:     llvm::Type *FTy = Is64 ? DoubleTy : FloatTy;
5383:     Ops[0] = Builder.CreateBitCast(Ops[0], InTy);
5384:     if (usgn)
5385:       return Builder.CreateUIToFP(Ops[0], FTy);
5386:     return Builder.CreateSIToFP(Ops[0], FTy);
5387:   }
5388:   case NEON::BI__builtin_neon_vcvth_f16_u16:
5389:   case NEON::BI__builtin_neon_vcvth_f16_u32:
5390:   case NEON::BI__builtin_neon_vcvth_f16_u64:
5391:     usgn = true;
5392:     [[fallthrough]];
5393:   case NEON::BI__builtin_neon_vcvth_f16_s16:
5394:   case NEON::BI__builtin_neon_vcvth_f16_s32:
5395:   case NEON::BI__builtin_neon_vcvth_f16_s64: {
5396:     llvm::Type *FTy = HalfTy;
5397:     llvm::Type *InTy;
5398:     if (Ops[0]->getType()->getPrimitiveSizeInBits() == 64)
5399:       InTy = Int64Ty;
5400:     else if (Ops[0]->getType()->getPrimitiveSizeInBits() == 32)
```
- **EN**: This block uses control flow (if, case) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（if, case）细化 目标内建函数降级 行为。

### Lines 5401-5430
```cpp
5401:       InTy = Int32Ty;
5402:     else
5403:       InTy = Int16Ty;
5404:     Ops[0] = Builder.CreateBitCast(Ops[0], InTy);
5405:     if (usgn)
5406:       return Builder.CreateUIToFP(Ops[0], FTy);
5407:     return Builder.CreateSIToFP(Ops[0], FTy);
5408:   }
5409:   case NEON::BI__builtin_neon_vcvtah_u16_f16:
5410:   case NEON::BI__builtin_neon_vcvtmh_u16_f16:
5411:   case NEON::BI__builtin_neon_vcvtnh_u16_f16:
5412:   case NEON::BI__builtin_neon_vcvtph_u16_f16:
5413:   case NEON::BI__builtin_neon_vcvtah_s16_f16:
5414:   case NEON::BI__builtin_neon_vcvtmh_s16_f16:
5415:   case NEON::BI__builtin_neon_vcvtnh_s16_f16:
5416:   case NEON::BI__builtin_neon_vcvtph_s16_f16: {
5417:     llvm::Type *InTy = Int16Ty;
5418:     llvm::Type* FTy  = HalfTy;
5419:     llvm::Type *Tys[2] = {InTy, FTy};
5420:     switch (BuiltinID) {
5421:     default: llvm_unreachable("missing builtin ID in switch!");
5422:     case NEON::BI__builtin_neon_vcvtah_u16_f16:
5423:       Int = Intrinsic::aarch64_neon_fcvtau; break;
5424:     case NEON::BI__builtin_neon_vcvtmh_u16_f16:
5425:       Int = Intrinsic::aarch64_neon_fcvtmu; break;
5426:     case NEON::BI__builtin_neon_vcvtnh_u16_f16:
5427:       Int = Intrinsic::aarch64_neon_fcvtnu; break;
5428:     case NEON::BI__builtin_neon_vcvtph_u16_f16:
5429:       Int = Intrinsic::aarch64_neon_fcvtpu; break;
5430:     case NEON::BI__builtin_neon_vcvtah_s16_f16:
```
- **EN**: This block uses control flow (if, switch, case) to specialize target builtin lowering; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块通过控制流（if, switch, case）细化 目标内建函数降级 行为；使用断言或不可达标记保护关键不变量。

### Lines 5431-5460
```cpp
5431:       Int = Intrinsic::aarch64_neon_fcvtas; break;
5432:     case NEON::BI__builtin_neon_vcvtmh_s16_f16:
5433:       Int = Intrinsic::aarch64_neon_fcvtms; break;
5434:     case NEON::BI__builtin_neon_vcvtnh_s16_f16:
5435:       Int = Intrinsic::aarch64_neon_fcvtns; break;
5436:     case NEON::BI__builtin_neon_vcvtph_s16_f16:
5437:       Int = Intrinsic::aarch64_neon_fcvtps; break;
5438:     }
5439:     return EmitNeonCall(CGM.getIntrinsic(Int, Tys), Ops, "fcvt");
5440:   }
5441:   case NEON::BI__builtin_neon_vcaleh_f16:
5442:   case NEON::BI__builtin_neon_vcalth_f16:
5443:   case NEON::BI__builtin_neon_vcageh_f16:
5444:   case NEON::BI__builtin_neon_vcagth_f16: {
5445:     llvm::Type* InTy = Int32Ty;
5446:     llvm::Type* FTy  = HalfTy;
5447:     llvm::Type *Tys[2] = {InTy, FTy};
5448:     switch (BuiltinID) {
5449:     default: llvm_unreachable("missing builtin ID in switch!");
5450:     case NEON::BI__builtin_neon_vcageh_f16:
5451:       Int = Intrinsic::aarch64_neon_facge; break;
5452:     case NEON::BI__builtin_neon_vcagth_f16:
5453:       Int = Intrinsic::aarch64_neon_facgt; break;
5454:     case NEON::BI__builtin_neon_vcaleh_f16:
5455:       Int = Intrinsic::aarch64_neon_facge; std::swap(Ops[0], Ops[1]); break;
5456:     case NEON::BI__builtin_neon_vcalth_f16:
5457:       Int = Intrinsic::aarch64_neon_facgt; std::swap(Ops[0], Ops[1]); break;
5458:     }
5459:     Ops[0] = EmitNeonCall(CGM.getIntrinsic(Int, Tys), Ops, "facg");
5460:     return Builder.CreateTrunc(Ops[0], Int16Ty);
```
- **EN**: This block defines callable entry points like `EmitNeonCall`; uses control flow (switch, case) to specialize target builtin lowering; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块定义可调用入口，例如 `EmitNeonCall`；通过控制流（switch, case）细化 目标内建函数降级 行为；使用断言或不可达标记保护关键不变量。

### Lines 5461-5490
```cpp
5461:   }
5462:   case NEON::BI__builtin_neon_vcvth_n_s16_f16:
5463:   case NEON::BI__builtin_neon_vcvth_n_u16_f16: {
5464:     llvm::Type* InTy = Int32Ty;
5465:     llvm::Type* FTy  = HalfTy;
5466:     llvm::Type *Tys[2] = {InTy, FTy};
5467:     switch (BuiltinID) {
5468:     default: llvm_unreachable("missing builtin ID in switch!");
5469:     case NEON::BI__builtin_neon_vcvth_n_s16_f16:
5470:       Int = Intrinsic::aarch64_neon_vcvtfp2fxs; break;
5471:     case NEON::BI__builtin_neon_vcvth_n_u16_f16:
5472:       Int = Intrinsic::aarch64_neon_vcvtfp2fxu; break;
5473:     }
5474:     Ops[0] = EmitNeonCall(CGM.getIntrinsic(Int, Tys), Ops, "fcvth_n");
5475:     return Builder.CreateTrunc(Ops[0], Int16Ty);
5476:   }
5477:   case NEON::BI__builtin_neon_vcvth_n_f16_s16:
5478:   case NEON::BI__builtin_neon_vcvth_n_f16_u16: {
5479:     llvm::Type* FTy  = HalfTy;
5480:     llvm::Type* InTy = Int32Ty;
5481:     llvm::Type *Tys[2] = {FTy, InTy};
5482:     switch (BuiltinID) {
5483:     default: llvm_unreachable("missing builtin ID in switch!");
5484:     case NEON::BI__builtin_neon_vcvth_n_f16_s16:
5485:       Int = Intrinsic::aarch64_neon_vcvtfxs2fp;
5486:       Ops[0] = Builder.CreateSExt(Ops[0], InTy, "sext");
5487:       break;
5488:     case NEON::BI__builtin_neon_vcvth_n_f16_u16:
5489:       Int = Intrinsic::aarch64_neon_vcvtfxu2fp;
5490:       Ops[0] = Builder.CreateZExt(Ops[0], InTy);
```
- **EN**: This block uses control flow (switch, case) to specialize target builtin lowering; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块通过控制流（switch, case）细化 目标内建函数降级 行为；使用断言或不可达标记保护关键不变量。

### Lines 5491-5520
```cpp
5491:       break;
5492:     }
5493:     return EmitNeonCall(CGM.getIntrinsic(Int, Tys), Ops, "fcvth_n");
5494:   }
5495:   case NEON::BI__builtin_neon_vpaddd_s64: {
5496:     // TODO: Isn't this handled by
5497:     // EmitCommonNeonSISDBuiltinExpr?
5498:     auto *Ty = llvm::FixedVectorType::get(Int64Ty, 2);
5499:     // The vector is v2f64, so make sure it's bitcast to that.
5500:     Ops[0] = Builder.CreateBitCast(Ops[0], Ty, "v2i64");
5501:     llvm::Value *Idx0 = llvm::ConstantInt::get(SizeTy, 0);
5502:     llvm::Value *Idx1 = llvm::ConstantInt::get(SizeTy, 1);
5503:     Value *Op0 = Builder.CreateExtractElement(Ops[0], Idx0, "lane0");
5504:     Value *Op1 = Builder.CreateExtractElement(Ops[0], Idx1, "lane1");
5505:     // Pairwise addition of a v2f64 into a scalar f64.
5506:     return Builder.CreateAdd(Op0, Op1, "vpaddd");
5507:   }
5508:   case NEON::BI__builtin_neon_vpaddd_f64: {
5509:     auto *Ty = llvm::FixedVectorType::get(DoubleTy, 2);
5510:     // The vector is v2f64, so make sure it's bitcast to that.
5511:     Ops[0] = Builder.CreateBitCast(Ops[0], Ty, "v2f64");
5512:     llvm::Value *Idx0 = llvm::ConstantInt::get(SizeTy, 0);
5513:     llvm::Value *Idx1 = llvm::ConstantInt::get(SizeTy, 1);
5514:     Value *Op0 = Builder.CreateExtractElement(Ops[0], Idx0, "lane0");
5515:     Value *Op1 = Builder.CreateExtractElement(Ops[0], Idx1, "lane1");
5516:     // Pairwise addition of a v2f64 into a scalar f64.
5517:     return Builder.CreateFAdd(Op0, Op1, "vpaddd");
5518:   }
5519:   case NEON::BI__builtin_neon_vpadds_f32: {
5520:     auto *Ty = llvm::FixedVectorType::get(FloatTy, 2);
```
- **EN**: This block defines callable entry points like `EmitNeonCall`; uses control flow (case) to specialize target builtin lowering.
- **CN**: 该代码块定义可调用入口，例如 `EmitNeonCall`；通过控制流（case）细化 目标内建函数降级 行为。

### Lines 5521-5550
```cpp
5521:     // The vector is v2f32, so make sure it's bitcast to that.
5522:     Ops[0] = Builder.CreateBitCast(Ops[0], Ty, "v2f32");
5523:     llvm::Value *Idx0 = llvm::ConstantInt::get(SizeTy, 0);
5524:     llvm::Value *Idx1 = llvm::ConstantInt::get(SizeTy, 1);
5525:     Value *Op0 = Builder.CreateExtractElement(Ops[0], Idx0, "lane0");
5526:     Value *Op1 = Builder.CreateExtractElement(Ops[0], Idx1, "lane1");
5527:     // Pairwise addition of a v2f32 into a scalar f32.
5528:     return Builder.CreateFAdd(Op0, Op1, "vpaddd");
5529:   }
5530:   case NEON::BI__builtin_neon_vceqzd_s64:
5531:     return EmitAArch64CompareBuiltinExpr(
5532:         Ops[0], ConvertType(E->getCallReturnType(getContext())),
5533:         ICmpInst::ICMP_EQ, "vceqz");
5534:   case NEON::BI__builtin_neon_vceqzd_f64:
5535:   case NEON::BI__builtin_neon_vceqzs_f32:
5536:   case NEON::BI__builtin_neon_vceqzh_f16:
5537:     return EmitAArch64CompareBuiltinExpr(
5538:         Ops[0], ConvertType(E->getCallReturnType(getContext())),
5539:         ICmpInst::FCMP_OEQ, "vceqz");
5540:   case NEON::BI__builtin_neon_vcgezd_s64:
5541:     return EmitAArch64CompareBuiltinExpr(
5542:         Ops[0], ConvertType(E->getCallReturnType(getContext())),
5543:         ICmpInst::ICMP_SGE, "vcgez");
5544:   case NEON::BI__builtin_neon_vcgezd_f64:
5545:   case NEON::BI__builtin_neon_vcgezs_f32:
5546:   case NEON::BI__builtin_neon_vcgezh_f16:
5547:     return EmitAArch64CompareBuiltinExpr(
5548:         Ops[0], ConvertType(E->getCallReturnType(getContext())),
5549:         ICmpInst::FCMP_OGE, "vcgez");
5550:   case NEON::BI__builtin_neon_vclezd_s64:
```
- **EN**: This block spells out callable entry points like `EmitAArch64CompareBuiltinExpr`; uses control flow (case) to specialize target builtin lowering.
- **CN**: 该代码块给出可调用入口的声明，例如 `EmitAArch64CompareBuiltinExpr`；通过控制流（case）细化 目标内建函数降级 行为。

### Lines 5551-5580
```cpp
5551:     return EmitAArch64CompareBuiltinExpr(
5552:         Ops[0], ConvertType(E->getCallReturnType(getContext())),
5553:         ICmpInst::ICMP_SLE, "vclez");
5554:   case NEON::BI__builtin_neon_vclezd_f64:
5555:   case NEON::BI__builtin_neon_vclezs_f32:
5556:   case NEON::BI__builtin_neon_vclezh_f16:
5557:     return EmitAArch64CompareBuiltinExpr(
5558:         Ops[0], ConvertType(E->getCallReturnType(getContext())),
5559:         ICmpInst::FCMP_OLE, "vclez");
5560:   case NEON::BI__builtin_neon_vcgtzd_s64:
5561:     return EmitAArch64CompareBuiltinExpr(
5562:         Ops[0], ConvertType(E->getCallReturnType(getContext())),
5563:         ICmpInst::ICMP_SGT, "vcgtz");
5564:   case NEON::BI__builtin_neon_vcgtzd_f64:
5565:   case NEON::BI__builtin_neon_vcgtzs_f32:
5566:   case NEON::BI__builtin_neon_vcgtzh_f16:
5567:     return EmitAArch64CompareBuiltinExpr(
5568:         Ops[0], ConvertType(E->getCallReturnType(getContext())),
5569:         ICmpInst::FCMP_OGT, "vcgtz");
5570:   case NEON::BI__builtin_neon_vcltzd_s64:
5571:     return EmitAArch64CompareBuiltinExpr(
5572:         Ops[0], ConvertType(E->getCallReturnType(getContext())),
5573:         ICmpInst::ICMP_SLT, "vcltz");
5574: 
5575:   case NEON::BI__builtin_neon_vcltzd_f64:
5576:   case NEON::BI__builtin_neon_vcltzs_f32:
5577:   case NEON::BI__builtin_neon_vcltzh_f16:
5578:     return EmitAArch64CompareBuiltinExpr(
5579:         Ops[0], ConvertType(E->getCallReturnType(getContext())),
5580:         ICmpInst::FCMP_OLT, "vcltz");
```
- **EN**: This block spells out callable entry points like `EmitAArch64CompareBuiltinExpr`; uses control flow (case) to specialize target builtin lowering.
- **CN**: 该代码块给出可调用入口的声明，例如 `EmitAArch64CompareBuiltinExpr`；通过控制流（case）细化 目标内建函数降级 行为。

### Lines 5581-5610
```cpp
5581: 
5582:   case NEON::BI__builtin_neon_vceqzd_u64: {
5583:     return EmitAArch64CompareBuiltinExpr(
5584:         Ops[0], ConvertType(E->getCallReturnType(getContext())),
5585:         ICmpInst::ICMP_EQ, "vceqzd");
5586:   }
5587:   case NEON::BI__builtin_neon_vceqd_f64:
5588:   case NEON::BI__builtin_neon_vcled_f64:
5589:   case NEON::BI__builtin_neon_vcltd_f64:
5590:   case NEON::BI__builtin_neon_vcged_f64:
5591:   case NEON::BI__builtin_neon_vcgtd_f64: {
5592:     llvm::CmpInst::Predicate P;
5593:     switch (BuiltinID) {
5594:     default: llvm_unreachable("missing builtin ID in switch!");
5595:     case NEON::BI__builtin_neon_vceqd_f64: P = llvm::FCmpInst::FCMP_OEQ; break;
5596:     case NEON::BI__builtin_neon_vcled_f64: P = llvm::FCmpInst::FCMP_OLE; break;
5597:     case NEON::BI__builtin_neon_vcltd_f64: P = llvm::FCmpInst::FCMP_OLT; break;
5598:     case NEON::BI__builtin_neon_vcged_f64: P = llvm::FCmpInst::FCMP_OGE; break;
5599:     case NEON::BI__builtin_neon_vcgtd_f64: P = llvm::FCmpInst::FCMP_OGT; break;
5600:     }
5601:     Ops[0] = Builder.CreateBitCast(Ops[0], DoubleTy);
5602:     Ops[1] = Builder.CreateBitCast(Ops[1], DoubleTy);
5603:     if (P == llvm::FCmpInst::FCMP_OEQ)
5604:       Ops[0] = Builder.CreateFCmp(P, Ops[0], Ops[1]);
5605:     else
5606:       Ops[0] = Builder.CreateFCmpS(P, Ops[0], Ops[1]);
5607:     return Builder.CreateSExt(Ops[0], Int64Ty, "vcmpd");
5608:   }
5609:   case NEON::BI__builtin_neon_vceqs_f32:
5610:   case NEON::BI__builtin_neon_vcles_f32:
```
- **EN**: This block defines callable entry points like `EmitAArch64CompareBuiltinExpr`; uses control flow (if, switch, case) to specialize target builtin lowering; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块定义可调用入口，例如 `EmitAArch64CompareBuiltinExpr`；通过控制流（if, switch, case）细化 目标内建函数降级 行为；使用断言或不可达标记保护关键不变量。

### Lines 5611-5640
```cpp
5611:   case NEON::BI__builtin_neon_vclts_f32:
5612:   case NEON::BI__builtin_neon_vcges_f32:
5613:   case NEON::BI__builtin_neon_vcgts_f32: {
5614:     llvm::CmpInst::Predicate P;
5615:     switch (BuiltinID) {
5616:     default: llvm_unreachable("missing builtin ID in switch!");
5617:     case NEON::BI__builtin_neon_vceqs_f32: P = llvm::FCmpInst::FCMP_OEQ; break;
5618:     case NEON::BI__builtin_neon_vcles_f32: P = llvm::FCmpInst::FCMP_OLE; break;
5619:     case NEON::BI__builtin_neon_vclts_f32: P = llvm::FCmpInst::FCMP_OLT; break;
5620:     case NEON::BI__builtin_neon_vcges_f32: P = llvm::FCmpInst::FCMP_OGE; break;
5621:     case NEON::BI__builtin_neon_vcgts_f32: P = llvm::FCmpInst::FCMP_OGT; break;
5622:     }
5623:     Ops[0] = Builder.CreateBitCast(Ops[0], FloatTy);
5624:     Ops[1] = Builder.CreateBitCast(Ops[1], FloatTy);
5625:     if (P == llvm::FCmpInst::FCMP_OEQ)
5626:       Ops[0] = Builder.CreateFCmp(P, Ops[0], Ops[1]);
5627:     else
5628:       Ops[0] = Builder.CreateFCmpS(P, Ops[0], Ops[1]);
5629:     return Builder.CreateSExt(Ops[0], Int32Ty, "vcmpd");
5630:   }
5631:   case NEON::BI__builtin_neon_vceqh_f16:
5632:   case NEON::BI__builtin_neon_vcleh_f16:
5633:   case NEON::BI__builtin_neon_vclth_f16:
5634:   case NEON::BI__builtin_neon_vcgeh_f16:
5635:   case NEON::BI__builtin_neon_vcgth_f16: {
5636:     llvm::CmpInst::Predicate P;
5637:     switch (BuiltinID) {
5638:     default: llvm_unreachable("missing builtin ID in switch!");
5639:     case NEON::BI__builtin_neon_vceqh_f16: P = llvm::FCmpInst::FCMP_OEQ; break;
5640:     case NEON::BI__builtin_neon_vcleh_f16: P = llvm::FCmpInst::FCMP_OLE; break;
```
- **EN**: This block uses control flow (if, switch, case) to specialize target builtin lowering; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块通过控制流（if, switch, case）细化 目标内建函数降级 行为；使用断言或不可达标记保护关键不变量。

### Lines 5641-5670
```cpp
5641:     case NEON::BI__builtin_neon_vclth_f16: P = llvm::FCmpInst::FCMP_OLT; break;
5642:     case NEON::BI__builtin_neon_vcgeh_f16: P = llvm::FCmpInst::FCMP_OGE; break;
5643:     case NEON::BI__builtin_neon_vcgth_f16: P = llvm::FCmpInst::FCMP_OGT; break;
5644:     }
5645:     Ops[0] = Builder.CreateBitCast(Ops[0], HalfTy);
5646:     Ops[1] = Builder.CreateBitCast(Ops[1], HalfTy);
5647:     if (P == llvm::FCmpInst::FCMP_OEQ)
5648:       Ops[0] = Builder.CreateFCmp(P, Ops[0], Ops[1]);
5649:     else
5650:       Ops[0] = Builder.CreateFCmpS(P, Ops[0], Ops[1]);
5651:     return Builder.CreateSExt(Ops[0], Int16Ty, "vcmpd");
5652:   }
5653:   case NEON::BI__builtin_neon_vceqd_s64:
5654:   case NEON::BI__builtin_neon_vceqd_u64:
5655:   case NEON::BI__builtin_neon_vcgtd_s64:
5656:   case NEON::BI__builtin_neon_vcgtd_u64:
5657:   case NEON::BI__builtin_neon_vcltd_s64:
5658:   case NEON::BI__builtin_neon_vcltd_u64:
5659:   case NEON::BI__builtin_neon_vcged_u64:
5660:   case NEON::BI__builtin_neon_vcged_s64:
5661:   case NEON::BI__builtin_neon_vcled_u64:
5662:   case NEON::BI__builtin_neon_vcled_s64: {
5663:     llvm::CmpInst::Predicate P;
5664:     switch (BuiltinID) {
5665:     default: llvm_unreachable("missing builtin ID in switch!");
5666:     case NEON::BI__builtin_neon_vceqd_s64:
5667:     case NEON::BI__builtin_neon_vceqd_u64:P = llvm::ICmpInst::ICMP_EQ;break;
5668:     case NEON::BI__builtin_neon_vcgtd_s64:P = llvm::ICmpInst::ICMP_SGT;break;
5669:     case NEON::BI__builtin_neon_vcgtd_u64:P = llvm::ICmpInst::ICMP_UGT;break;
5670:     case NEON::BI__builtin_neon_vcltd_s64:P = llvm::ICmpInst::ICMP_SLT;break;
```
- **EN**: This block uses control flow (if, switch, case) to specialize target builtin lowering; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块通过控制流（if, switch, case）细化 目标内建函数降级 行为；使用断言或不可达标记保护关键不变量。

### Lines 5671-5700
```cpp
5671:     case NEON::BI__builtin_neon_vcltd_u64:P = llvm::ICmpInst::ICMP_ULT;break;
5672:     case NEON::BI__builtin_neon_vcged_u64:P = llvm::ICmpInst::ICMP_UGE;break;
5673:     case NEON::BI__builtin_neon_vcged_s64:P = llvm::ICmpInst::ICMP_SGE;break;
5674:     case NEON::BI__builtin_neon_vcled_u64:P = llvm::ICmpInst::ICMP_ULE;break;
5675:     case NEON::BI__builtin_neon_vcled_s64:P = llvm::ICmpInst::ICMP_SLE;break;
5676:     }
5677:     Ops[0] = Builder.CreateBitCast(Ops[0], Int64Ty);
5678:     Ops[1] = Builder.CreateBitCast(Ops[1], Int64Ty);
5679:     Ops[0] = Builder.CreateICmp(P, Ops[0], Ops[1]);
5680:     return Builder.CreateSExt(Ops[0], Int64Ty, "vceqd");
5681:   }
5682:   case NEON::BI__builtin_neon_vnegd_s64:
5683:     return Builder.CreateNeg(Ops[0], "vnegd");
5684:   case NEON::BI__builtin_neon_vnegh_f16:
5685:     return Builder.CreateFNeg(Ops[0], "vnegh");
5686:   case NEON::BI__builtin_neon_vtstd_s64:
5687:   case NEON::BI__builtin_neon_vtstd_u64: {
5688:     Ops[0] = Builder.CreateBitCast(Ops[0], Int64Ty);
5689:     Ops[1] = Builder.CreateBitCast(Ops[1], Int64Ty);
5690:     Ops[0] = Builder.CreateAnd(Ops[0], Ops[1]);
5691:     Ops[0] = Builder.CreateICmp(ICmpInst::ICMP_NE, Ops[0],
5692:                                 llvm::Constant::getNullValue(Int64Ty));
5693:     return Builder.CreateSExt(Ops[0], Int64Ty, "vtstd");
5694:   }
5695:   case NEON::BI__builtin_neon_vset_lane_i8:
5696:   case NEON::BI__builtin_neon_vset_lane_i16:
5697:   case NEON::BI__builtin_neon_vset_lane_i32:
5698:   case NEON::BI__builtin_neon_vset_lane_i64:
5699:   case NEON::BI__builtin_neon_vset_lane_bf16:
5700:   case NEON::BI__builtin_neon_vset_lane_f32:
```
- **EN**: This block defines callable entry points like `getNullValue`; uses control flow (case) to specialize target builtin lowering.
- **CN**: 该代码块定义可调用入口，例如 `getNullValue`；通过控制流（case）细化 目标内建函数降级 行为。

### Lines 5701-5730
```cpp
5701:   case NEON::BI__builtin_neon_vsetq_lane_i8:
5702:   case NEON::BI__builtin_neon_vsetq_lane_i16:
5703:   case NEON::BI__builtin_neon_vsetq_lane_i32:
5704:   case NEON::BI__builtin_neon_vsetq_lane_i64:
5705:   case NEON::BI__builtin_neon_vsetq_lane_bf16:
5706:   case NEON::BI__builtin_neon_vsetq_lane_f32:
5707:     return Builder.CreateInsertElement(Ops[1], Ops[0], Ops[2], "vset_lane");
5708:   case NEON::BI__builtin_neon_vset_lane_f64:
5709:     // The vector type needs a cast for the v1f64 variant.
5710:     Ops[1] =
5711:         Builder.CreateBitCast(Ops[1], llvm::FixedVectorType::get(DoubleTy, 1));
5712:     return Builder.CreateInsertElement(Ops[1], Ops[0], Ops[2], "vset_lane");
5713:   case NEON::BI__builtin_neon_vset_lane_mf8:
5714:   case NEON::BI__builtin_neon_vsetq_lane_mf8:
5715:     // The input vector type needs a cast to scalar type.
5716:     Ops[0] =
5717:         Builder.CreateBitCast(Ops[0], llvm::Type::getInt8Ty(getLLVMContext()));
5718:     return Builder.CreateInsertElement(Ops[1], Ops[0], Ops[2], "vset_lane");
5719:   case NEON::BI__builtin_neon_vsetq_lane_f64:
5720:     // The vector type needs a cast for the v2f64 variant.
5721:     Ops[1] =
5722:         Builder.CreateBitCast(Ops[1], llvm::FixedVectorType::get(DoubleTy, 2));
5723:     return Builder.CreateInsertElement(Ops[1], Ops[0], Ops[2], "vset_lane");
5724: 
5725:   case NEON::BI__builtin_neon_vget_lane_i8:
5726:   case NEON::BI__builtin_neon_vdupb_lane_i8:
5727:     Ops[0] =
5728:         Builder.CreateBitCast(Ops[0], llvm::FixedVectorType::get(Int8Ty, 8));
5729:     return Builder.CreateExtractElement(Ops[0], Ops[1], "vget_lane");
5730:   case NEON::BI__builtin_neon_vgetq_lane_i8:
```
- **EN**: This block uses control flow (case) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（case）细化 目标内建函数降级 行为。

### Lines 5731-5760
```cpp
5731:   case NEON::BI__builtin_neon_vdupb_laneq_i8:
5732:     Ops[0] =
5733:         Builder.CreateBitCast(Ops[0], llvm::FixedVectorType::get(Int8Ty, 16));
5734:     return Builder.CreateExtractElement(Ops[0], Ops[1], "vgetq_lane");
5735:   case NEON::BI__builtin_neon_vget_lane_mf8:
5736:   case NEON::BI__builtin_neon_vdupb_lane_mf8:
5737:   case NEON::BI__builtin_neon_vgetq_lane_mf8:
5738:   case NEON::BI__builtin_neon_vdupb_laneq_mf8:
5739:     return Builder.CreateExtractElement(Ops[0], Ops[1], "vget_lane");
5740:   case NEON::BI__builtin_neon_vget_lane_i16:
5741:   case NEON::BI__builtin_neon_vduph_lane_i16:
5742:     Ops[0] =
5743:         Builder.CreateBitCast(Ops[0], llvm::FixedVectorType::get(Int16Ty, 4));
5744:     return Builder.CreateExtractElement(Ops[0], Ops[1], "vget_lane");
5745:   case NEON::BI__builtin_neon_vgetq_lane_i16:
5746:   case NEON::BI__builtin_neon_vduph_laneq_i16:
5747:     Ops[0] =
5748:         Builder.CreateBitCast(Ops[0], llvm::FixedVectorType::get(Int16Ty, 8));
5749:     return Builder.CreateExtractElement(Ops[0], Ops[1], "vgetq_lane");
5750:   case NEON::BI__builtin_neon_vget_lane_i32:
5751:   case NEON::BI__builtin_neon_vdups_lane_i32:
5752:     Ops[0] =
5753:         Builder.CreateBitCast(Ops[0], llvm::FixedVectorType::get(Int32Ty, 2));
5754:     return Builder.CreateExtractElement(Ops[0], Ops[1], "vget_lane");
5755:   case NEON::BI__builtin_neon_vdups_lane_f32:
5756:     Ops[0] =
5757:         Builder.CreateBitCast(Ops[0], llvm::FixedVectorType::get(FloatTy, 2));
5758:     return Builder.CreateExtractElement(Ops[0], Ops[1], "vdups_lane");
5759:   case NEON::BI__builtin_neon_vgetq_lane_i32:
5760:   case NEON::BI__builtin_neon_vdups_laneq_i32:
```
- **EN**: This block uses control flow (case) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（case）细化 目标内建函数降级 行为。

### Lines 5761-5790
```cpp
5761:     Ops[0] =
5762:         Builder.CreateBitCast(Ops[0], llvm::FixedVectorType::get(Int32Ty, 4));
5763:     return Builder.CreateExtractElement(Ops[0], Ops[1], "vgetq_lane");
5764:   case NEON::BI__builtin_neon_vget_lane_i64:
5765:   case NEON::BI__builtin_neon_vdupd_lane_i64:
5766:     Ops[0] =
5767:         Builder.CreateBitCast(Ops[0], llvm::FixedVectorType::get(Int64Ty, 1));
5768:     return Builder.CreateExtractElement(Ops[0], Ops[1], "vget_lane");
5769:   case NEON::BI__builtin_neon_vdupd_lane_f64:
5770:     Ops[0] =
5771:         Builder.CreateBitCast(Ops[0], llvm::FixedVectorType::get(DoubleTy, 1));
5772:     return Builder.CreateExtractElement(Ops[0], Ops[1], "vdupd_lane");
5773:   case NEON::BI__builtin_neon_vgetq_lane_i64:
5774:   case NEON::BI__builtin_neon_vdupd_laneq_i64:
5775:     Ops[0] =
5776:         Builder.CreateBitCast(Ops[0], llvm::FixedVectorType::get(Int64Ty, 2));
5777:     return Builder.CreateExtractElement(Ops[0], Ops[1], "vgetq_lane");
5778:   case NEON::BI__builtin_neon_vget_lane_f32:
5779:     Ops[0] =
5780:         Builder.CreateBitCast(Ops[0], llvm::FixedVectorType::get(FloatTy, 2));
5781:     return Builder.CreateExtractElement(Ops[0], Ops[1], "vget_lane");
5782:   case NEON::BI__builtin_neon_vget_lane_f64:
5783:     Ops[0] =
5784:         Builder.CreateBitCast(Ops[0], llvm::FixedVectorType::get(DoubleTy, 1));
5785:     return Builder.CreateExtractElement(Ops[0], Ops[1], "vget_lane");
5786:   case NEON::BI__builtin_neon_vgetq_lane_f32:
5787:   case NEON::BI__builtin_neon_vdups_laneq_f32:
5788:     Ops[0] =
5789:         Builder.CreateBitCast(Ops[0], llvm::FixedVectorType::get(FloatTy, 4));
5790:     return Builder.CreateExtractElement(Ops[0], Ops[1], "vgetq_lane");
```
- **EN**: This block uses control flow (case) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（case）细化 目标内建函数降级 行为。

### Lines 5791-5820
```cpp
5791:   case NEON::BI__builtin_neon_vgetq_lane_f64:
5792:   case NEON::BI__builtin_neon_vdupd_laneq_f64:
5793:     Ops[0] =
5794:         Builder.CreateBitCast(Ops[0], llvm::FixedVectorType::get(DoubleTy, 2));
5795:     return Builder.CreateExtractElement(Ops[0], Ops[1], "vgetq_lane");
5796:   case NEON::BI__builtin_neon_vaddh_f16:
5797:     return Builder.CreateFAdd(Ops[0], Ops[1], "vaddh");
5798:   case NEON::BI__builtin_neon_vsubh_f16:
5799:     return Builder.CreateFSub(Ops[0], Ops[1], "vsubh");
5800:   case NEON::BI__builtin_neon_vmulh_f16:
5801:     return Builder.CreateFMul(Ops[0], Ops[1], "vmulh");
5802:   case NEON::BI__builtin_neon_vdivh_f16:
5803:     return Builder.CreateFDiv(Ops[0], Ops[1], "vdivh");
5804:   case NEON::BI__builtin_neon_vfmah_f16:
5805:     // NEON intrinsic puts accumulator first, unlike the LLVM fma.
5806:     return emitCallMaybeConstrainedFPBuiltin(
5807:         *this, Intrinsic::fma, Intrinsic::experimental_constrained_fma, HalfTy,
5808:         {Ops[1], Ops[2], Ops[0]});
5809:   case NEON::BI__builtin_neon_vfmsh_f16: {
5810:     Value *Neg = Builder.CreateFNeg(Ops[1], "vsubh");
5811: 
5812:     // NEON intrinsic puts accumulator first, unlike the LLVM fma.
5813:     return emitCallMaybeConstrainedFPBuiltin(
5814:         *this, Intrinsic::fma, Intrinsic::experimental_constrained_fma, HalfTy,
5815:         {Neg, Ops[2], Ops[0]});
5816:   }
5817:   case NEON::BI__builtin_neon_vaddd_s64:
5818:   case NEON::BI__builtin_neon_vaddd_u64:
5819:     return Builder.CreateAdd(Ops[0], Ops[1], "vaddd");
5820:   case NEON::BI__builtin_neon_vsubd_s64:
```
- **EN**: This block uses control flow (case) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（case）细化 目标内建函数降级 行为。

### Lines 5821-5850
```cpp
5821:   case NEON::BI__builtin_neon_vsubd_u64:
5822:     return Builder.CreateSub(Ops[0], Ops[1], "vsubd");
5823:   case NEON::BI__builtin_neon_vqdmlalh_s16:
5824:   case NEON::BI__builtin_neon_vqdmlslh_s16: {
5825:     SmallVector<Value *, 2> ProductOps;
5826:     ProductOps.push_back(vectorWrapScalar16(Ops[1]));
5827:     ProductOps.push_back(vectorWrapScalar16(Ops[2]));
5828:     auto *VTy = llvm::FixedVectorType::get(Int32Ty, 4);
5829:     Ops[1] = EmitNeonCall(CGM.getIntrinsic(Intrinsic::aarch64_neon_sqdmull, VTy),
5830:                           ProductOps, "vqdmlXl");
5831:     Constant *CI = ConstantInt::get(SizeTy, 0);
5832:     Ops[1] = Builder.CreateExtractElement(Ops[1], CI, "lane0");
5833: 
5834:     unsigned AccumInt = BuiltinID == NEON::BI__builtin_neon_vqdmlalh_s16
5835:                                         ? Intrinsic::aarch64_neon_sqadd
5836:                                         : Intrinsic::aarch64_neon_sqsub;
5837:     // Drop the 2nd multiplication argument before the accumulation
5838:     Ops.pop_back();
5839:     return EmitNeonCall(CGM.getIntrinsic(AccumInt, Int32Ty), Ops, "vqdmlXl");
5840:   }
5841:   case NEON::BI__builtin_neon_vqshlud_n_s64: {
5842:     Ops[1] = Builder.CreateZExt(Ops[1], Int64Ty);
5843:     return EmitNeonCall(CGM.getIntrinsic(Intrinsic::aarch64_neon_sqshlu, Int64Ty),
5844:                         Ops, "vqshlu_n");
5845:   }
5846:   case NEON::BI__builtin_neon_vqshld_n_u64:
5847:   case NEON::BI__builtin_neon_vqshld_n_s64: {
5848:     Int = BuiltinID == NEON::BI__builtin_neon_vqshld_n_u64
5849:               ? Intrinsic::aarch64_neon_uqshl
5850:               : Intrinsic::aarch64_neon_sqshl;
```
- **EN**: This block defines callable entry points like `EmitNeonCall`; uses control flow (case) to specialize target builtin lowering.
- **CN**: 该代码块定义可调用入口，例如 `EmitNeonCall`；通过控制流（case）细化 目标内建函数降级 行为。

### Lines 5851-5880
```cpp
5851:     Ops[1] = Builder.CreateZExt(Ops[1], Int64Ty);
5852:     return EmitNeonCall(CGM.getIntrinsic(Int, Int64Ty), Ops, "vqshl_n");
5853:   }
5854:   case NEON::BI__builtin_neon_vrshrd_n_u64:
5855:   case NEON::BI__builtin_neon_vrshrd_n_s64: {
5856:     Int = BuiltinID == NEON::BI__builtin_neon_vrshrd_n_u64
5857:               ? Intrinsic::aarch64_neon_urshl
5858:               : Intrinsic::aarch64_neon_srshl;
5859:     int SV = cast<ConstantInt>(Ops[1])->getSExtValue();
5860:     Ops[1] = ConstantInt::get(Int64Ty, -SV);
5861:     return EmitNeonCall(CGM.getIntrinsic(Int, Int64Ty), Ops, "vrshr_n");
5862:   }
5863:   case NEON::BI__builtin_neon_vrsrad_n_u64:
5864:   case NEON::BI__builtin_neon_vrsrad_n_s64: {
5865:     Int = BuiltinID == NEON::BI__builtin_neon_vrsrad_n_u64
5866:               ? Intrinsic::aarch64_neon_urshl
5867:               : Intrinsic::aarch64_neon_srshl;
5868:     Ops[1] = Builder.CreateBitCast(Ops[1], Int64Ty);
5869:     Ops[2] = Builder.CreateNeg(Ops[2]);
5870:     Ops[1] = Builder.CreateCall(CGM.getIntrinsic(Int, Int64Ty),
5871:                                 {Ops[1], Builder.CreateSExt(Ops[2], Int64Ty)});
5872:     return Builder.CreateAdd(Ops[0], Builder.CreateBitCast(Ops[1], Int64Ty));
5873:   }
5874:   case NEON::BI__builtin_neon_vshld_n_s64:
5875:   case NEON::BI__builtin_neon_vshld_n_u64: {
5876:     llvm::ConstantInt *Amt = cast<ConstantInt>(Ops[1]);
5877:     return Builder.CreateShl(
5878:         Ops[0], ConstantInt::get(Int64Ty, Amt->getZExtValue()), "shld_n");
5879:   }
5880:   case NEON::BI__builtin_neon_vshrd_n_s64: {
```
- **EN**: This block defines callable entry points like `EmitNeonCall`, `get`; uses control flow (case) to specialize target builtin lowering.
- **CN**: 该代码块定义可调用入口，例如 `EmitNeonCall`, `get`；通过控制流（case）细化 目标内建函数降级 行为。

### Lines 5881-5910
```cpp
5881:     llvm::ConstantInt *Amt = cast<ConstantInt>(Ops[1]);
5882:     return Builder.CreateAShr(
5883:         Ops[0], ConstantInt::get(Int64Ty, std::min(static_cast<uint64_t>(63),
5884:                                                    Amt->getZExtValue())),
5885:         "shrd_n");
5886:   }
5887:   case NEON::BI__builtin_neon_vshrd_n_u64: {
5888:     llvm::ConstantInt *Amt = cast<ConstantInt>(Ops[1]);
5889:     uint64_t ShiftAmt = Amt->getZExtValue();
5890:     // Right-shifting an unsigned value by its size yields 0.
5891:     if (ShiftAmt == 64)
5892:       return ConstantInt::get(Int64Ty, 0);
5893:     return Builder.CreateLShr(Ops[0], ConstantInt::get(Int64Ty, ShiftAmt),
5894:                               "shrd_n");
5895:   }
5896:   case NEON::BI__builtin_neon_vsrad_n_s64: {
5897:     llvm::ConstantInt *Amt = cast<ConstantInt>(Ops[2]);
5898:     Ops[1] = Builder.CreateAShr(
5899:         Ops[1], ConstantInt::get(Int64Ty, std::min(static_cast<uint64_t>(63),
5900:                                                    Amt->getZExtValue())),
5901:         "shrd_n");
5902:     return Builder.CreateAdd(Ops[0], Ops[1]);
5903:   }
5904:   case NEON::BI__builtin_neon_vsrad_n_u64: {
5905:     llvm::ConstantInt *Amt = cast<ConstantInt>(Ops[2]);
5906:     uint64_t ShiftAmt = Amt->getZExtValue();
5907:     // Right-shifting an unsigned value by its size yields 0.
5908:     // As Op + 0 = Op, return Ops[0] directly.
5909:     if (ShiftAmt == 64)
5910:       return Ops[0];
```
- **EN**: This block defines callable entry points like `get`; uses control flow (if, case) to specialize target builtin lowering.
- **CN**: 该代码块定义可调用入口，例如 `get`；通过控制流（if, case）细化 目标内建函数降级 行为。

### Lines 5911-5940
```cpp
5911:     Ops[1] = Builder.CreateLShr(Ops[1], ConstantInt::get(Int64Ty, ShiftAmt),
5912:                                 "shrd_n");
5913:     return Builder.CreateAdd(Ops[0], Ops[1]);
5914:   }
5915:   case NEON::BI__builtin_neon_vqdmlalh_lane_s16:
5916:   case NEON::BI__builtin_neon_vqdmlalh_laneq_s16:
5917:   case NEON::BI__builtin_neon_vqdmlslh_lane_s16:
5918:   case NEON::BI__builtin_neon_vqdmlslh_laneq_s16: {
5919:     Ops[2] = Builder.CreateExtractElement(Ops[2], Ops[3], "lane");
5920:     SmallVector<Value *, 2> ProductOps;
5921:     ProductOps.push_back(vectorWrapScalar16(Ops[1]));
5922:     ProductOps.push_back(vectorWrapScalar16(Ops[2]));
5923:     auto *VTy = llvm::FixedVectorType::get(Int32Ty, 4);
5924:     Ops[1] = EmitNeonCall(CGM.getIntrinsic(Intrinsic::aarch64_neon_sqdmull, VTy),
5925:                           ProductOps, "vqdmlXl");
5926:     Constant *CI = ConstantInt::get(SizeTy, 0);
5927:     Ops[1] = Builder.CreateExtractElement(Ops[1], CI, "lane0");
5928:     // Drop lane-selection and the corresponding vector argument (these have
5929:     // already been used)
5930:     Ops.pop_back_n(2);
5931: 
5932:     unsigned AccInt = (BuiltinID == NEON::BI__builtin_neon_vqdmlalh_lane_s16 ||
5933:                        BuiltinID == NEON::BI__builtin_neon_vqdmlalh_laneq_s16)
5934:                           ? Intrinsic::aarch64_neon_sqadd
5935:                           : Intrinsic::aarch64_neon_sqsub;
5936:     return EmitNeonCall(CGM.getIntrinsic(AccInt, Int32Ty), Ops, "vqdmlXl");
5937:   }
5938:   case NEON::BI__builtin_neon_vqdmlals_s32:
5939:   case NEON::BI__builtin_neon_vqdmlsls_s32: {
5940:     SmallVector<Value *, 2> ProductOps;
```
- **EN**: This block defines callable entry points like `EmitNeonCall`; uses control flow (case) to specialize target builtin lowering.
- **CN**: 该代码块定义可调用入口，例如 `EmitNeonCall`；通过控制流（case）细化 目标内建函数降级 行为。

### Lines 5941-5970
```cpp
5941:     ProductOps.push_back(Ops[1]);
5942:     ProductOps.push_back(Ops[2]);
5943:     Ops[1] =
5944:         EmitNeonCall(CGM.getIntrinsic(Intrinsic::aarch64_neon_sqdmulls_scalar),
5945:                      ProductOps, "vqdmlXl");
5946: 
5947:     unsigned AccumInt = BuiltinID == NEON::BI__builtin_neon_vqdmlals_s32
5948:                                         ? Intrinsic::aarch64_neon_sqadd
5949:                                         : Intrinsic::aarch64_neon_sqsub;
5950:     // Drop the 2nd multiplication argument before the accumulation
5951:     Ops.pop_back();
5952:     return EmitNeonCall(CGM.getIntrinsic(AccumInt, Int64Ty), Ops, "vqdmlXl");
5953:   }
5954:   case NEON::BI__builtin_neon_vqdmlals_lane_s32:
5955:   case NEON::BI__builtin_neon_vqdmlals_laneq_s32:
5956:   case NEON::BI__builtin_neon_vqdmlsls_lane_s32:
5957:   case NEON::BI__builtin_neon_vqdmlsls_laneq_s32: {
5958:     Ops[2] = Builder.CreateExtractElement(Ops[2], Ops[3], "lane");
5959:     SmallVector<Value *, 2> ProductOps;
5960:     ProductOps.push_back(Ops[1]);
5961:     ProductOps.push_back(Ops[2]);
5962:     Ops[1] =
5963:         EmitNeonCall(CGM.getIntrinsic(Intrinsic::aarch64_neon_sqdmulls_scalar),
5964:                      ProductOps, "vqdmlXl");
5965:     // Drop lane-selection and the corresponding vector argument (these have
5966:     // already been used)
5967:     Ops.pop_back_n(2);
5968: 
5969:     unsigned AccInt = (BuiltinID == NEON::BI__builtin_neon_vqdmlals_lane_s32 ||
5970:                        BuiltinID == NEON::BI__builtin_neon_vqdmlals_laneq_s32)
```
- **EN**: This block defines callable entry points like `EmitNeonCall`; uses control flow (case) to specialize target builtin lowering.
- **CN**: 该代码块定义可调用入口，例如 `EmitNeonCall`；通过控制流（case）细化 目标内建函数降级 行为。

### Lines 5971-6000
```cpp
5971:                           ? Intrinsic::aarch64_neon_sqadd
5972:                           : Intrinsic::aarch64_neon_sqsub;
5973:     return EmitNeonCall(CGM.getIntrinsic(AccInt, Int64Ty), Ops, "vqdmlXl");
5974:   }
5975:   case NEON::BI__builtin_neon_vget_lane_bf16:
5976:   case NEON::BI__builtin_neon_vduph_lane_bf16:
5977:   case NEON::BI__builtin_neon_vduph_lane_f16: {
5978:     return Builder.CreateExtractElement(Ops[0], Ops[1], "vget_lane");
5979:   }
5980:   case NEON::BI__builtin_neon_vgetq_lane_bf16:
5981:   case NEON::BI__builtin_neon_vduph_laneq_bf16:
5982:   case NEON::BI__builtin_neon_vduph_laneq_f16: {
5983:     return Builder.CreateExtractElement(Ops[0], Ops[1], "vgetq_lane");
5984:   }
5985:   case NEON::BI__builtin_neon_vcvt_bf16_f32: {
5986:     llvm::Type *V4F32 = FixedVectorType::get(Builder.getFloatTy(), 4);
5987:     llvm::Type *V4BF16 = FixedVectorType::get(Builder.getBFloatTy(), 4);
5988:     return Builder.CreateFPTrunc(Builder.CreateBitCast(Ops[0], V4F32), V4BF16);
5989:   }
5990:   case NEON::BI__builtin_neon_vcvtq_low_bf16_f32: {
5991:     SmallVector<int, 16> ConcatMask(8);
5992:     std::iota(ConcatMask.begin(), ConcatMask.end(), 0);
5993:     llvm::Type *V4F32 = FixedVectorType::get(Builder.getFloatTy(), 4);
5994:     llvm::Type *V4BF16 = FixedVectorType::get(Builder.getBFloatTy(), 4);
5995:     llvm::Value *Trunc =
5996:         Builder.CreateFPTrunc(Builder.CreateBitCast(Ops[0], V4F32), V4BF16);
5997:     return Builder.CreateShuffleVector(
5998:         Trunc, ConstantAggregateZero::get(V4BF16), ConcatMask);
5999:   }
6000:   case NEON::BI__builtin_neon_vcvtq_high_bf16_f32: {
```
- **EN**: This block defines callable entry points like `EmitNeonCall`, `ConcatMask`, `iota`, `get`; uses control flow (case) to specialize target builtin lowering.
- **CN**: 该代码块定义可调用入口，例如 `EmitNeonCall`, `ConcatMask`, `iota`, `get`；通过控制流（case）细化 目标内建函数降级 行为。

### Lines 6001-6030
```cpp
6001:     SmallVector<int, 16> ConcatMask(8);
6002:     std::iota(ConcatMask.begin(), ConcatMask.end(), 0);
6003:     SmallVector<int, 16> LoMask(4);
6004:     std::iota(LoMask.begin(), LoMask.end(), 0);
6005:     llvm::Type *V4F32 = FixedVectorType::get(Builder.getFloatTy(), 4);
6006:     llvm::Type *V4BF16 = FixedVectorType::get(Builder.getBFloatTy(), 4);
6007:     llvm::Type *V8BF16 = FixedVectorType::get(Builder.getBFloatTy(), 8);
6008:     llvm::Value *Inactive = Builder.CreateShuffleVector(
6009:         Builder.CreateBitCast(Ops[0], V8BF16), LoMask);
6010:     llvm::Value *Trunc =
6011:         Builder.CreateFPTrunc(Builder.CreateBitCast(Ops[1], V4F32), V4BF16);
6012:     return Builder.CreateShuffleVector(Inactive, Trunc, ConcatMask);
6013:   }
6014: 
6015:   case clang::AArch64::BI_InterlockedAdd:
6016:   case clang::AArch64::BI_InterlockedAdd_acq:
6017:   case clang::AArch64::BI_InterlockedAdd_rel:
6018:   case clang::AArch64::BI_InterlockedAdd_nf:
6019:   case clang::AArch64::BI_InterlockedAdd64:
6020:   case clang::AArch64::BI_InterlockedAdd64_acq:
6021:   case clang::AArch64::BI_InterlockedAdd64_rel:
6022:   case clang::AArch64::BI_InterlockedAdd64_nf: {
6023:     Address DestAddr = CheckAtomicAlignment(*this, E);
6024:     Value *Val = Ops[1];
6025:     llvm::AtomicOrdering Ordering;
6026:     switch (BuiltinID) {
6027:     case clang::AArch64::BI_InterlockedAdd:
6028:     case clang::AArch64::BI_InterlockedAdd64:
6029:       Ordering = llvm::AtomicOrdering::SequentiallyConsistent;
6030:       break;
```
- **EN**: This block defines callable entry points like `ConcatMask`, `iota`, `LoMask`; uses control flow (switch, case) to specialize target builtin lowering.
- **CN**: 该代码块定义可调用入口，例如 `ConcatMask`, `iota`, `LoMask`；通过控制流（switch, case）细化 目标内建函数降级 行为。

### Lines 6031-6060
```cpp
6031:     case clang::AArch64::BI_InterlockedAdd_acq:
6032:     case clang::AArch64::BI_InterlockedAdd64_acq:
6033:       Ordering = llvm::AtomicOrdering::Acquire;
6034:       break;
6035:     case clang::AArch64::BI_InterlockedAdd_rel:
6036:     case clang::AArch64::BI_InterlockedAdd64_rel:
6037:       Ordering = llvm::AtomicOrdering::Release;
6038:       break;
6039:     case clang::AArch64::BI_InterlockedAdd_nf:
6040:     case clang::AArch64::BI_InterlockedAdd64_nf:
6041:       Ordering = llvm::AtomicOrdering::Monotonic;
6042:       break;
6043:     default:
6044:       llvm_unreachable("missing builtin ID in switch!");
6045:     }
6046:     AtomicRMWInst *RMWI =
6047:         Builder.CreateAtomicRMW(AtomicRMWInst::Add, DestAddr, Val, Ordering);
6048:     return Builder.CreateAdd(RMWI, Val);
6049:   }
6050:   }
6051: 
6052:   llvm::FixedVectorType *VTy = GetNeonType(this, Type);
6053:   llvm::Type *Ty = VTy;
6054:   if (!Ty)
6055:     return nullptr;
6056: 
6057:   bool ExtractLow = false;
6058:   bool ExtendLaneArg = false;
6059:   switch (BuiltinID) {
6060:   default: return nullptr;
```
- **EN**: This block uses control flow (if, switch, case) to specialize target builtin lowering; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块通过控制流（if, switch, case）细化 目标内建函数降级 行为；使用断言或不可达标记保护关键不变量。

### Lines 6061-6090
```cpp
6061:   case NEON::BI__builtin_neon_vbsl_v:
6062:   case NEON::BI__builtin_neon_vbslq_v: {
6063:     llvm::Type *BitTy = llvm::VectorType::getInteger(VTy);
6064:     Ops[0] = Builder.CreateBitCast(Ops[0], BitTy, "vbsl");
6065:     Ops[1] = Builder.CreateBitCast(Ops[1], BitTy, "vbsl");
6066:     Ops[2] = Builder.CreateBitCast(Ops[2], BitTy, "vbsl");
6067: 
6068:     Ops[1] = Builder.CreateAnd(Ops[0], Ops[1], "vbsl");
6069:     Ops[2] = Builder.CreateAnd(Builder.CreateNot(Ops[0]), Ops[2], "vbsl");
6070:     Ops[0] = Builder.CreateOr(Ops[1], Ops[2], "vbsl");
6071:     return Builder.CreateBitCast(Ops[0], Ty);
6072:   }
6073:   case NEON::BI__builtin_neon_vfma_lane_v:
6074:   case NEON::BI__builtin_neon_vfmaq_lane_v: { // Only used for FP types
6075:     // The ARM builtins (and instructions) have the addend as the first
6076:     // operand, but the 'fma' intrinsics have it last. Swap it around here.
6077:     Value *Addend = Ops[0];
6078:     Value *Multiplicand = Ops[1];
6079:     Value *LaneSource = Ops[2];
6080:     Ops[0] = Multiplicand;
6081:     Ops[1] = LaneSource;
6082:     Ops[2] = Addend;
6083: 
6084:     // Now adjust things to handle the lane access.
6085:     auto *SourceTy = BuiltinID == NEON::BI__builtin_neon_vfmaq_lane_v
6086:                          ? llvm::FixedVectorType::get(VTy->getElementType(),
6087:                                                       VTy->getNumElements() / 2)
6088:                          : VTy;
6089:     llvm::Constant *cst = cast<Constant>(Ops[3]);
6090:     Value *SV = llvm::ConstantVector::getSplat(VTy->getElementCount(), cst);
```
- **EN**: This block uses control flow (for, case) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（for, case）细化 目标内建函数降级 行为。

### Lines 6091-6120
```cpp
6091:     Ops[1] = Builder.CreateBitCast(Ops[1], SourceTy);
6092:     Ops[1] = Builder.CreateShuffleVector(Ops[1], Ops[1], SV, "lane");
6093: 
6094:     Ops.pop_back();
6095:     Int = Builder.getIsFPConstrained() ? Intrinsic::experimental_constrained_fma
6096:                                        : Intrinsic::fma;
6097:     return EmitNeonCall(CGM.getIntrinsic(Int, Ty), Ops, "fmla");
6098:   }
6099:   case NEON::BI__builtin_neon_vfma_laneq_v: {
6100:     auto *VTy = cast<llvm::FixedVectorType>(Ty);
6101:     // v1f64 fma should be mapped to Neon scalar f64 fma
6102:     if (VTy && VTy->getElementType() == DoubleTy) {
6103:       Ops[0] = Builder.CreateBitCast(Ops[0], DoubleTy);
6104:       Ops[1] = Builder.CreateBitCast(Ops[1], DoubleTy);
6105:       llvm::FixedVectorType *VTy =
6106:           GetNeonType(this, NeonTypeFlags(NeonTypeFlags::Float64, false, true));
6107:       Ops[2] = Builder.CreateBitCast(Ops[2], VTy);
6108:       Ops[2] = Builder.CreateExtractElement(Ops[2], Ops[3], "extract");
6109:       Value *Result;
6110:       Result = emitCallMaybeConstrainedFPBuiltin(
6111:           *this, Intrinsic::fma, Intrinsic::experimental_constrained_fma,
6112:           DoubleTy, {Ops[1], Ops[2], Ops[0]});
6113:       return Builder.CreateBitCast(Result, Ty);
6114:     }
6115:     Ops[0] = Builder.CreateBitCast(Ops[0], Ty);
6116:     Ops[1] = Builder.CreateBitCast(Ops[1], Ty);
6117: 
6118:     auto *STy = llvm::FixedVectorType::get(VTy->getElementType(),
6119:                                            VTy->getNumElements() * 2);
6120:     Ops[2] = Builder.CreateBitCast(Ops[2], STy);
```
- **EN**: This block defines callable entry points like `EmitNeonCall`, `GetNeonType`; uses control flow (if, case) to specialize target builtin lowering.
- **CN**: 该代码块定义可调用入口，例如 `EmitNeonCall`, `GetNeonType`；通过控制流（if, case）细化 目标内建函数降级 行为。

### Lines 6121-6150
```cpp
6121:     Value *SV = llvm::ConstantVector::getSplat(VTy->getElementCount(),
6122:                                                cast<ConstantInt>(Ops[3]));
6123:     Ops[2] = Builder.CreateShuffleVector(Ops[2], Ops[2], SV, "lane");
6124: 
6125:     return emitCallMaybeConstrainedFPBuiltin(
6126:         *this, Intrinsic::fma, Intrinsic::experimental_constrained_fma, Ty,
6127:         {Ops[2], Ops[1], Ops[0]});
6128:   }
6129:   case NEON::BI__builtin_neon_vfmaq_laneq_v: {
6130:     Ops[0] = Builder.CreateBitCast(Ops[0], Ty);
6131:     Ops[1] = Builder.CreateBitCast(Ops[1], Ty);
6132: 
6133:     Ops[2] = Builder.CreateBitCast(Ops[2], Ty);
6134:     Ops[2] = EmitNeonSplat(Ops[2], cast<ConstantInt>(Ops[3]));
6135:     return emitCallMaybeConstrainedFPBuiltin(
6136:         *this, Intrinsic::fma, Intrinsic::experimental_constrained_fma, Ty,
6137:         {Ops[2], Ops[1], Ops[0]});
6138:   }
6139:   case NEON::BI__builtin_neon_vfmah_lane_f16:
6140:   case NEON::BI__builtin_neon_vfmas_lane_f32:
6141:   case NEON::BI__builtin_neon_vfmah_laneq_f16:
6142:   case NEON::BI__builtin_neon_vfmas_laneq_f32:
6143:   case NEON::BI__builtin_neon_vfmad_lane_f64:
6144:   case NEON::BI__builtin_neon_vfmad_laneq_f64: {
6145:     llvm::Type *Ty = ConvertType(E->getCallReturnType(getContext()));
6146:     Ops[2] = Builder.CreateExtractElement(Ops[2], Ops[3], "extract");
6147:     return emitCallMaybeConstrainedFPBuiltin(
6148:         *this, Intrinsic::fma, Intrinsic::experimental_constrained_fma, Ty,
6149:         {Ops[1], Ops[2], Ops[0]});
6150:   }
```
- **EN**: This block uses control flow (case) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（case）细化 目标内建函数降级 行为。

### Lines 6151-6180
```cpp
6151:   case NEON::BI__builtin_neon_vmull_v:
6152:     // FIXME: improve sharing scheme to cope with 3 alternative LLVM intrinsics.
6153:     Int = usgn ? Intrinsic::aarch64_neon_umull : Intrinsic::aarch64_neon_smull;
6154:     if (Type.isPoly()) Int = Intrinsic::aarch64_neon_pmull;
6155:     return EmitNeonCall(CGM.getIntrinsic(Int, Ty), Ops, "vmull");
6156:   case NEON::BI__builtin_neon_vmax_v:
6157:   case NEON::BI__builtin_neon_vmaxq_v:
6158:     // FIXME: improve sharing scheme to cope with 3 alternative LLVM intrinsics.
6159:     Int = usgn ? Intrinsic::aarch64_neon_umax : Intrinsic::aarch64_neon_smax;
6160:     if (Ty->isFPOrFPVectorTy()) Int = Intrinsic::aarch64_neon_fmax;
6161:     return EmitNeonCall(CGM.getIntrinsic(Int, Ty), Ops, "vmax");
6162:   case NEON::BI__builtin_neon_vmaxh_f16: {
6163:     Int = Intrinsic::aarch64_neon_fmax;
6164:     return EmitNeonCall(CGM.getIntrinsic(Int, HalfTy), Ops, "vmax");
6165:   }
6166:   case NEON::BI__builtin_neon_vmin_v:
6167:   case NEON::BI__builtin_neon_vminq_v:
6168:     // FIXME: improve sharing scheme to cope with 3 alternative LLVM intrinsics.
6169:     Int = usgn ? Intrinsic::aarch64_neon_umin : Intrinsic::aarch64_neon_smin;
6170:     if (Ty->isFPOrFPVectorTy()) Int = Intrinsic::aarch64_neon_fmin;
6171:     return EmitNeonCall(CGM.getIntrinsic(Int, Ty), Ops, "vmin");
6172:   case NEON::BI__builtin_neon_vminh_f16: {
6173:     Int = Intrinsic::aarch64_neon_fmin;
6174:     return EmitNeonCall(CGM.getIntrinsic(Int, HalfTy), Ops, "vmin");
6175:   }
6176:   case NEON::BI__builtin_neon_vabd_v:
6177:   case NEON::BI__builtin_neon_vabdq_v:
6178:     // FIXME: improve sharing scheme to cope with 3 alternative LLVM intrinsics.
6179:     Int = usgn ? Intrinsic::aarch64_neon_uabd : Intrinsic::aarch64_neon_sabd;
6180:     if (Ty->isFPOrFPVectorTy()) Int = Intrinsic::aarch64_neon_fabd;
```
- **EN**: This block defines callable entry points like `EmitNeonCall`; uses control flow (if, case) to specialize target builtin lowering.
- **CN**: 该代码块定义可调用入口，例如 `EmitNeonCall`；通过控制流（if, case）细化 目标内建函数降级 行为。

### Lines 6181-6210
```cpp
6181:     return EmitNeonCall(CGM.getIntrinsic(Int, Ty), Ops, "vabd");
6182:   case NEON::BI__builtin_neon_vpadal_v:
6183:   case NEON::BI__builtin_neon_vpadalq_v: {
6184:     unsigned ArgElts = VTy->getNumElements();
6185:     llvm::IntegerType *EltTy = cast<IntegerType>(VTy->getElementType());
6186:     unsigned BitWidth = EltTy->getBitWidth();
6187:     auto *ArgTy = llvm::FixedVectorType::get(
6188:         llvm::IntegerType::get(getLLVMContext(), BitWidth / 2), 2 * ArgElts);
6189:     llvm::Type* Tys[2] = { VTy, ArgTy };
6190:     Int = usgn ? Intrinsic::aarch64_neon_uaddlp : Intrinsic::aarch64_neon_saddlp;
6191:     SmallVector<llvm::Value*, 1> TmpOps;
6192:     TmpOps.push_back(Ops[1]);
6193:     Function *F = CGM.getIntrinsic(Int, Tys);
6194:     llvm::Value *tmp = EmitNeonCall(F, TmpOps, "vpadal");
6195:     llvm::Value *addend = Builder.CreateBitCast(Ops[0], tmp->getType());
6196:     return Builder.CreateAdd(tmp, addend);
6197:   }
6198:   case NEON::BI__builtin_neon_vpmin_v:
6199:   case NEON::BI__builtin_neon_vpminq_v:
6200:     // FIXME: improve sharing scheme to cope with 3 alternative LLVM intrinsics.
6201:     Int = usgn ? Intrinsic::aarch64_neon_uminp : Intrinsic::aarch64_neon_sminp;
6202:     if (Ty->isFPOrFPVectorTy()) Int = Intrinsic::aarch64_neon_fminp;
6203:     return EmitNeonCall(CGM.getIntrinsic(Int, Ty), Ops, "vpmin");
6204:   case NEON::BI__builtin_neon_vpmax_v:
6205:   case NEON::BI__builtin_neon_vpmaxq_v:
6206:     // FIXME: improve sharing scheme to cope with 3 alternative LLVM intrinsics.
6207:     Int = usgn ? Intrinsic::aarch64_neon_umaxp : Intrinsic::aarch64_neon_smaxp;
6208:     if (Ty->isFPOrFPVectorTy()) Int = Intrinsic::aarch64_neon_fmaxp;
6209:     return EmitNeonCall(CGM.getIntrinsic(Int, Ty), Ops, "vpmax");
6210:   case NEON::BI__builtin_neon_vminnm_v:
```
- **EN**: This block defines callable entry points like `EmitNeonCall`, `get`; uses control flow (if, case) to specialize target builtin lowering.
- **CN**: 该代码块定义可调用入口，例如 `EmitNeonCall`, `get`；通过控制流（if, case）细化 目标内建函数降级 行为。

### Lines 6211-6240
```cpp
6211:   case NEON::BI__builtin_neon_vminnmq_v:
6212:     Int = Intrinsic::aarch64_neon_fminnm;
6213:     return EmitNeonCall(CGM.getIntrinsic(Int, Ty), Ops, "vminnm");
6214:   case NEON::BI__builtin_neon_vminnmh_f16:
6215:     Int = Intrinsic::aarch64_neon_fminnm;
6216:     return EmitNeonCall(CGM.getIntrinsic(Int, HalfTy), Ops, "vminnm");
6217:   case NEON::BI__builtin_neon_vmaxnm_v:
6218:   case NEON::BI__builtin_neon_vmaxnmq_v:
6219:     Int = Intrinsic::aarch64_neon_fmaxnm;
6220:     return EmitNeonCall(CGM.getIntrinsic(Int, Ty), Ops, "vmaxnm");
6221:   case NEON::BI__builtin_neon_vmaxnmh_f16:
6222:     Int = Intrinsic::aarch64_neon_fmaxnm;
6223:     return EmitNeonCall(CGM.getIntrinsic(Int, HalfTy), Ops, "vmaxnm");
6224:   case NEON::BI__builtin_neon_vrecpss_f32: {
6225:     return EmitNeonCall(CGM.getIntrinsic(Intrinsic::aarch64_neon_frecps, FloatTy),
6226:                         Ops, "vrecps");
6227:   }
6228:   case NEON::BI__builtin_neon_vrecpsd_f64:
6229:     return EmitNeonCall(CGM.getIntrinsic(Intrinsic::aarch64_neon_frecps, DoubleTy),
6230:                         Ops, "vrecps");
6231:   case NEON::BI__builtin_neon_vrecpsh_f16:
6232:     return EmitNeonCall(CGM.getIntrinsic(Intrinsic::aarch64_neon_frecps, HalfTy),
6233:                         Ops, "vrecps");
6234:   case NEON::BI__builtin_neon_vqshrun_n_v:
6235:     Int = Intrinsic::aarch64_neon_sqshrun;
6236:     return EmitNeonCall(CGM.getIntrinsic(Int, Ty), Ops, "vqshrun_n");
6237:   case NEON::BI__builtin_neon_vqrshrun_n_v:
6238:     Int = Intrinsic::aarch64_neon_sqrshrun;
6239:     return EmitNeonCall(CGM.getIntrinsic(Int, Ty), Ops, "vqrshrun_n");
6240:   case NEON::BI__builtin_neon_vqshrn_n_v:
```
- **EN**: This block defines callable entry points like `EmitNeonCall`; uses control flow (case) to specialize target builtin lowering.
- **CN**: 该代码块定义可调用入口，例如 `EmitNeonCall`；通过控制流（case）细化 目标内建函数降级 行为。

### Lines 6241-6270
```cpp
6241:     Int = usgn ? Intrinsic::aarch64_neon_uqshrn : Intrinsic::aarch64_neon_sqshrn;
6242:     return EmitNeonCall(CGM.getIntrinsic(Int, Ty), Ops, "vqshrn_n");
6243:   case NEON::BI__builtin_neon_vrshrn_n_v:
6244:     Int = Intrinsic::aarch64_neon_rshrn;
6245:     return EmitNeonCall(CGM.getIntrinsic(Int, Ty), Ops, "vrshrn_n");
6246:   case NEON::BI__builtin_neon_vqrshrn_n_v:
6247:     Int = usgn ? Intrinsic::aarch64_neon_uqrshrn : Intrinsic::aarch64_neon_sqrshrn;
6248:     return EmitNeonCall(CGM.getIntrinsic(Int, Ty), Ops, "vqrshrn_n");
6249:   case NEON::BI__builtin_neon_vrndah_f16: {
6250:     Int = Builder.getIsFPConstrained()
6251:               ? Intrinsic::experimental_constrained_round
6252:               : Intrinsic::round;
6253:     return EmitNeonCall(CGM.getIntrinsic(Int, HalfTy), Ops, "vrnda");
6254:   }
6255:   case NEON::BI__builtin_neon_vrnda_v:
6256:   case NEON::BI__builtin_neon_vrndaq_v: {
6257:     Int = Builder.getIsFPConstrained()
6258:               ? Intrinsic::experimental_constrained_round
6259:               : Intrinsic::round;
6260:     return EmitNeonCall(CGM.getIntrinsic(Int, Ty), Ops, "vrnda");
6261:   }
6262:   case NEON::BI__builtin_neon_vrndih_f16: {
6263:     Int = Builder.getIsFPConstrained()
6264:               ? Intrinsic::experimental_constrained_nearbyint
6265:               : Intrinsic::nearbyint;
6266:     return EmitNeonCall(CGM.getIntrinsic(Int, HalfTy), Ops, "vrndi");
6267:   }
6268:   case NEON::BI__builtin_neon_vrndmh_f16: {
6269:     Int = Builder.getIsFPConstrained()
6270:               ? Intrinsic::experimental_constrained_floor
```
- **EN**: This block defines callable entry points like `EmitNeonCall`; uses control flow (case) to specialize target builtin lowering.
- **CN**: 该代码块定义可调用入口，例如 `EmitNeonCall`；通过控制流（case）细化 目标内建函数降级 行为。

### Lines 6271-6300
```cpp
6271:               : Intrinsic::floor;
6272:     return EmitNeonCall(CGM.getIntrinsic(Int, HalfTy), Ops, "vrndm");
6273:   }
6274:   case NEON::BI__builtin_neon_vrndm_v:
6275:   case NEON::BI__builtin_neon_vrndmq_v: {
6276:     Int = Builder.getIsFPConstrained()
6277:               ? Intrinsic::experimental_constrained_floor
6278:               : Intrinsic::floor;
6279:     return EmitNeonCall(CGM.getIntrinsic(Int, Ty), Ops, "vrndm");
6280:   }
6281:   case NEON::BI__builtin_neon_vrndnh_f16: {
6282:     Int = Builder.getIsFPConstrained()
6283:               ? Intrinsic::experimental_constrained_roundeven
6284:               : Intrinsic::roundeven;
6285:     return EmitNeonCall(CGM.getIntrinsic(Int, HalfTy), Ops, "vrndn");
6286:   }
6287:   case NEON::BI__builtin_neon_vrndn_v:
6288:   case NEON::BI__builtin_neon_vrndnq_v: {
6289:     Int = Builder.getIsFPConstrained()
6290:               ? Intrinsic::experimental_constrained_roundeven
6291:               : Intrinsic::roundeven;
6292:     return EmitNeonCall(CGM.getIntrinsic(Int, Ty), Ops, "vrndn");
6293:   }
6294:   case NEON::BI__builtin_neon_vrndns_f32: {
6295:     Int = Builder.getIsFPConstrained()
6296:               ? Intrinsic::experimental_constrained_roundeven
6297:               : Intrinsic::roundeven;
6298:     return EmitNeonCall(CGM.getIntrinsic(Int, FloatTy), Ops, "vrndn");
6299:   }
6300:   case NEON::BI__builtin_neon_vrndph_f16: {
```
- **EN**: This block defines callable entry points like `EmitNeonCall`; uses control flow (case) to specialize target builtin lowering.
- **CN**: 该代码块定义可调用入口，例如 `EmitNeonCall`；通过控制流（case）细化 目标内建函数降级 行为。

### Lines 6301-6330
```cpp
6301:     Int = Builder.getIsFPConstrained()
6302:               ? Intrinsic::experimental_constrained_ceil
6303:               : Intrinsic::ceil;
6304:     return EmitNeonCall(CGM.getIntrinsic(Int, HalfTy), Ops, "vrndp");
6305:   }
6306:   case NEON::BI__builtin_neon_vrndp_v:
6307:   case NEON::BI__builtin_neon_vrndpq_v: {
6308:     Int = Builder.getIsFPConstrained()
6309:               ? Intrinsic::experimental_constrained_ceil
6310:               : Intrinsic::ceil;
6311:     return EmitNeonCall(CGM.getIntrinsic(Int, Ty), Ops, "vrndp");
6312:   }
6313:   case NEON::BI__builtin_neon_vrndxh_f16: {
6314:     Int = Builder.getIsFPConstrained()
6315:               ? Intrinsic::experimental_constrained_rint
6316:               : Intrinsic::rint;
6317:     return EmitNeonCall(CGM.getIntrinsic(Int, HalfTy), Ops, "vrndx");
6318:   }
6319:   case NEON::BI__builtin_neon_vrndx_v:
6320:   case NEON::BI__builtin_neon_vrndxq_v: {
6321:     Int = Builder.getIsFPConstrained()
6322:               ? Intrinsic::experimental_constrained_rint
6323:               : Intrinsic::rint;
6324:     return EmitNeonCall(CGM.getIntrinsic(Int, Ty), Ops, "vrndx");
6325:   }
6326:   case NEON::BI__builtin_neon_vrndh_f16: {
6327:     Int = Builder.getIsFPConstrained()
6328:               ? Intrinsic::experimental_constrained_trunc
6329:               : Intrinsic::trunc;
6330:     return EmitNeonCall(CGM.getIntrinsic(Int, HalfTy), Ops, "vrndz");
```
- **EN**: This block defines callable entry points like `EmitNeonCall`; uses control flow (case) to specialize target builtin lowering.
- **CN**: 该代码块定义可调用入口，例如 `EmitNeonCall`；通过控制流（case）细化 目标内建函数降级 行为。

### Lines 6331-6360
```cpp
6331:   }
6332:   case NEON::BI__builtin_neon_vrnd32x_f32:
6333:   case NEON::BI__builtin_neon_vrnd32xq_f32:
6334:   case NEON::BI__builtin_neon_vrnd32x_f64:
6335:   case NEON::BI__builtin_neon_vrnd32xq_f64: {
6336:     Int = Intrinsic::aarch64_neon_frint32x;
6337:     return EmitNeonCall(CGM.getIntrinsic(Int, Ty), Ops, "vrnd32x");
6338:   }
6339:   case NEON::BI__builtin_neon_vrnd32z_f32:
6340:   case NEON::BI__builtin_neon_vrnd32zq_f32:
6341:   case NEON::BI__builtin_neon_vrnd32z_f64:
6342:   case NEON::BI__builtin_neon_vrnd32zq_f64: {
6343:     Int = Intrinsic::aarch64_neon_frint32z;
6344:     return EmitNeonCall(CGM.getIntrinsic(Int, Ty), Ops, "vrnd32z");
6345:   }
6346:   case NEON::BI__builtin_neon_vrnd64x_f32:
6347:   case NEON::BI__builtin_neon_vrnd64xq_f32:
6348:   case NEON::BI__builtin_neon_vrnd64x_f64:
6349:   case NEON::BI__builtin_neon_vrnd64xq_f64: {
6350:     Int = Intrinsic::aarch64_neon_frint64x;
6351:     return EmitNeonCall(CGM.getIntrinsic(Int, Ty), Ops, "vrnd64x");
6352:   }
6353:   case NEON::BI__builtin_neon_vrnd64z_f32:
6354:   case NEON::BI__builtin_neon_vrnd64zq_f32:
6355:   case NEON::BI__builtin_neon_vrnd64z_f64:
6356:   case NEON::BI__builtin_neon_vrnd64zq_f64: {
6357:     Int = Intrinsic::aarch64_neon_frint64z;
6358:     return EmitNeonCall(CGM.getIntrinsic(Int, Ty), Ops, "vrnd64z");
6359:   }
6360:   case NEON::BI__builtin_neon_vrnd_v:
```
- **EN**: This block defines callable entry points like `EmitNeonCall`; uses control flow (case) to specialize target builtin lowering.
- **CN**: 该代码块定义可调用入口，例如 `EmitNeonCall`；通过控制流（case）细化 目标内建函数降级 行为。

### Lines 6361-6390
```cpp
6361:   case NEON::BI__builtin_neon_vrndq_v: {
6362:     Int = Builder.getIsFPConstrained()
6363:               ? Intrinsic::experimental_constrained_trunc
6364:               : Intrinsic::trunc;
6365:     return EmitNeonCall(CGM.getIntrinsic(Int, Ty), Ops, "vrndz");
6366:   }
6367:   case NEON::BI__builtin_neon_vcvt_f64_v:
6368:   case NEON::BI__builtin_neon_vcvtq_f64_v:
6369:     Ops[0] = Builder.CreateBitCast(Ops[0], Ty);
6370:     Ty = GetNeonType(this, NeonTypeFlags(NeonTypeFlags::Float64, false, quad));
6371:     return usgn ? Builder.CreateUIToFP(Ops[0], Ty, "vcvt")
6372:                 : Builder.CreateSIToFP(Ops[0], Ty, "vcvt");
6373:   case NEON::BI__builtin_neon_vcvt_f64_f32: {
6374:     assert(Type.getEltType() == NeonTypeFlags::Float64 && quad &&
6375:            "unexpected vcvt_f64_f32 builtin");
6376:     NeonTypeFlags SrcFlag = NeonTypeFlags(NeonTypeFlags::Float32, false, false);
6377:     Ops[0] = Builder.CreateBitCast(Ops[0], GetNeonType(this, SrcFlag));
6378: 
6379:     return Builder.CreateFPExt(Ops[0], Ty, "vcvt");
6380:   }
6381:   case NEON::BI__builtin_neon_vcvt_f32_f64: {
6382:     assert(Type.getEltType() == NeonTypeFlags::Float32 &&
6383:            "unexpected vcvt_f32_f64 builtin");
6384:     NeonTypeFlags SrcFlag = NeonTypeFlags(NeonTypeFlags::Float64, false, true);
6385:     Ops[0] = Builder.CreateBitCast(Ops[0], GetNeonType(this, SrcFlag));
6386: 
6387:     return Builder.CreateFPTrunc(Ops[0], Ty, "vcvt");
6388:   }
6389:   case NEON::BI__builtin_neon_vcvta_s16_f16:
6390:   case NEON::BI__builtin_neon_vcvta_u16_f16:
```
- **EN**: This block defines callable entry points like `EmitNeonCall`; uses control flow (case) to specialize target builtin lowering; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块定义可调用入口，例如 `EmitNeonCall`；通过控制流（case）细化 目标内建函数降级 行为；使用断言或不可达标记保护关键不变量。

### Lines 6391-6420
```cpp
6391:   case NEON::BI__builtin_neon_vcvta_s32_v:
6392:   case NEON::BI__builtin_neon_vcvtaq_s16_f16:
6393:   case NEON::BI__builtin_neon_vcvtaq_s32_v:
6394:   case NEON::BI__builtin_neon_vcvta_u32_v:
6395:   case NEON::BI__builtin_neon_vcvtaq_u16_f16:
6396:   case NEON::BI__builtin_neon_vcvtaq_u32_v:
6397:   case NEON::BI__builtin_neon_vcvta_s64_v:
6398:   case NEON::BI__builtin_neon_vcvtaq_s64_v:
6399:   case NEON::BI__builtin_neon_vcvta_u64_v:
6400:   case NEON::BI__builtin_neon_vcvtaq_u64_v: {
6401:     Int = usgn ? Intrinsic::aarch64_neon_fcvtau : Intrinsic::aarch64_neon_fcvtas;
6402:     llvm::Type *Tys[2] = { Ty, GetFloatNeonType(this, Type) };
6403:     return EmitNeonCall(CGM.getIntrinsic(Int, Tys), Ops, "vcvta");
6404:   }
6405:   case NEON::BI__builtin_neon_vcvtm_s16_f16:
6406:   case NEON::BI__builtin_neon_vcvtm_s32_v:
6407:   case NEON::BI__builtin_neon_vcvtmq_s16_f16:
6408:   case NEON::BI__builtin_neon_vcvtmq_s32_v:
6409:   case NEON::BI__builtin_neon_vcvtm_u16_f16:
6410:   case NEON::BI__builtin_neon_vcvtm_u32_v:
6411:   case NEON::BI__builtin_neon_vcvtmq_u16_f16:
6412:   case NEON::BI__builtin_neon_vcvtmq_u32_v:
6413:   case NEON::BI__builtin_neon_vcvtm_s64_v:
6414:   case NEON::BI__builtin_neon_vcvtmq_s64_v:
6415:   case NEON::BI__builtin_neon_vcvtm_u64_v:
6416:   case NEON::BI__builtin_neon_vcvtmq_u64_v: {
6417:     Int = usgn ? Intrinsic::aarch64_neon_fcvtmu : Intrinsic::aarch64_neon_fcvtms;
6418:     llvm::Type *Tys[2] = { Ty, GetFloatNeonType(this, Type) };
6419:     return EmitNeonCall(CGM.getIntrinsic(Int, Tys), Ops, "vcvtm");
6420:   }
```
- **EN**: This block defines callable entry points like `EmitNeonCall`; uses control flow (case) to specialize target builtin lowering.
- **CN**: 该代码块定义可调用入口，例如 `EmitNeonCall`；通过控制流（case）细化 目标内建函数降级 行为。

### Lines 6421-6450
```cpp
6421:   case NEON::BI__builtin_neon_vcvtn_s16_f16:
6422:   case NEON::BI__builtin_neon_vcvtn_s32_v:
6423:   case NEON::BI__builtin_neon_vcvtnq_s16_f16:
6424:   case NEON::BI__builtin_neon_vcvtnq_s32_v:
6425:   case NEON::BI__builtin_neon_vcvtn_u16_f16:
6426:   case NEON::BI__builtin_neon_vcvtn_u32_v:
6427:   case NEON::BI__builtin_neon_vcvtnq_u16_f16:
6428:   case NEON::BI__builtin_neon_vcvtnq_u32_v:
6429:   case NEON::BI__builtin_neon_vcvtn_s64_v:
6430:   case NEON::BI__builtin_neon_vcvtnq_s64_v:
6431:   case NEON::BI__builtin_neon_vcvtn_u64_v:
6432:   case NEON::BI__builtin_neon_vcvtnq_u64_v: {
6433:     Int = usgn ? Intrinsic::aarch64_neon_fcvtnu : Intrinsic::aarch64_neon_fcvtns;
6434:     llvm::Type *Tys[2] = { Ty, GetFloatNeonType(this, Type) };
6435:     return EmitNeonCall(CGM.getIntrinsic(Int, Tys), Ops, "vcvtn");
6436:   }
6437:   case NEON::BI__builtin_neon_vcvtp_s16_f16:
6438:   case NEON::BI__builtin_neon_vcvtp_s32_v:
6439:   case NEON::BI__builtin_neon_vcvtpq_s16_f16:
6440:   case NEON::BI__builtin_neon_vcvtpq_s32_v:
6441:   case NEON::BI__builtin_neon_vcvtp_u16_f16:
6442:   case NEON::BI__builtin_neon_vcvtp_u32_v:
6443:   case NEON::BI__builtin_neon_vcvtpq_u16_f16:
6444:   case NEON::BI__builtin_neon_vcvtpq_u32_v:
6445:   case NEON::BI__builtin_neon_vcvtp_s64_v:
6446:   case NEON::BI__builtin_neon_vcvtpq_s64_v:
6447:   case NEON::BI__builtin_neon_vcvtp_u64_v:
6448:   case NEON::BI__builtin_neon_vcvtpq_u64_v: {
6449:     Int = usgn ? Intrinsic::aarch64_neon_fcvtpu : Intrinsic::aarch64_neon_fcvtps;
6450:     llvm::Type *Tys[2] = { Ty, GetFloatNeonType(this, Type) };
```
- **EN**: This block defines callable entry points like `EmitNeonCall`; uses control flow (case) to specialize target builtin lowering.
- **CN**: 该代码块定义可调用入口，例如 `EmitNeonCall`；通过控制流（case）细化 目标内建函数降级 行为。

### Lines 6451-6480
```cpp
6451:     return EmitNeonCall(CGM.getIntrinsic(Int, Tys), Ops, "vcvtp");
6452:   }
6453:   case NEON::BI__builtin_neon_vmulx_v:
6454:   case NEON::BI__builtin_neon_vmulxq_v: {
6455:     Int = Intrinsic::aarch64_neon_fmulx;
6456:     return EmitNeonCall(CGM.getIntrinsic(Int, Ty), Ops, "vmulx");
6457:   }
6458:   case NEON::BI__builtin_neon_vmulxh_lane_f16:
6459:   case NEON::BI__builtin_neon_vmulxh_laneq_f16: {
6460:     // vmulx_lane should be mapped to Neon scalar mulx after
6461:     // extracting the scalar element
6462:     Ops[1] = Builder.CreateExtractElement(Ops[1], Ops[2], "extract");
6463:     Ops.pop_back();
6464:     Int = Intrinsic::aarch64_neon_fmulx;
6465:     return EmitNeonCall(CGM.getIntrinsic(Int, HalfTy), Ops, "vmulx");
6466:   }
6467:   case NEON::BI__builtin_neon_vmul_lane_v:
6468:   case NEON::BI__builtin_neon_vmul_laneq_v: {
6469:     // v1f64 vmul_lane should be mapped to Neon scalar mul lane
6470:     bool Quad = false;
6471:     if (BuiltinID == NEON::BI__builtin_neon_vmul_laneq_v)
6472:       Quad = true;
6473:     Ops[0] = Builder.CreateBitCast(Ops[0], DoubleTy);
6474:     llvm::FixedVectorType *VTy =
6475:         GetNeonType(this, NeonTypeFlags(NeonTypeFlags::Float64, false, Quad));
6476:     Ops[1] = Builder.CreateBitCast(Ops[1], VTy);
6477:     Ops[1] = Builder.CreateExtractElement(Ops[1], Ops[2], "extract");
6478:     Value *Result = Builder.CreateFMul(Ops[0], Ops[1]);
6479:     return Builder.CreateBitCast(Result, Ty);
6480:   }
```
- **EN**: This block defines callable entry points like `EmitNeonCall`, `GetNeonType`; uses control flow (if, case) to specialize target builtin lowering.
- **CN**: 该代码块定义可调用入口，例如 `EmitNeonCall`, `GetNeonType`；通过控制流（if, case）细化 目标内建函数降级 行为。

### Lines 6481-6510
```cpp
6481:   case NEON::BI__builtin_neon_vpmaxnm_v:
6482:   case NEON::BI__builtin_neon_vpmaxnmq_v: {
6483:     Int = Intrinsic::aarch64_neon_fmaxnmp;
6484:     return EmitNeonCall(CGM.getIntrinsic(Int, Ty), Ops, "vpmaxnm");
6485:   }
6486:   case NEON::BI__builtin_neon_vpminnm_v:
6487:   case NEON::BI__builtin_neon_vpminnmq_v: {
6488:     Int = Intrinsic::aarch64_neon_fminnmp;
6489:     return EmitNeonCall(CGM.getIntrinsic(Int, Ty), Ops, "vpminnm");
6490:   }
6491:   case NEON::BI__builtin_neon_vsqrth_f16: {
6492:     Int = Builder.getIsFPConstrained()
6493:               ? Intrinsic::experimental_constrained_sqrt
6494:               : Intrinsic::sqrt;
6495:     return EmitNeonCall(CGM.getIntrinsic(Int, HalfTy), Ops, "vsqrt");
6496:   }
6497:   case NEON::BI__builtin_neon_vsqrt_v:
6498:   case NEON::BI__builtin_neon_vsqrtq_v: {
6499:     Int = Builder.getIsFPConstrained()
6500:               ? Intrinsic::experimental_constrained_sqrt
6501:               : Intrinsic::sqrt;
6502:     Ops[0] = Builder.CreateBitCast(Ops[0], Ty);
6503:     return EmitNeonCall(CGM.getIntrinsic(Int, Ty), Ops, "vsqrt");
6504:   }
6505:   case NEON::BI__builtin_neon_vrbit_v:
6506:   case NEON::BI__builtin_neon_vrbitq_v: {
6507:     Int = Intrinsic::bitreverse;
6508:     return EmitNeonCall(CGM.getIntrinsic(Int, Ty), Ops, "vrbit");
6509:   }
6510:   case NEON::BI__builtin_neon_vmaxv_f16: {
```
- **EN**: This block defines callable entry points like `EmitNeonCall`; uses control flow (case) to specialize target builtin lowering.
- **CN**: 该代码块定义可调用入口，例如 `EmitNeonCall`；通过控制流（case）细化 目标内建函数降级 行为。

### Lines 6511-6540
```cpp
6511:     Int = Intrinsic::aarch64_neon_fmaxv;
6512:     Ty = HalfTy;
6513:     VTy = llvm::FixedVectorType::get(HalfTy, 4);
6514:     llvm::Type *Tys[2] = {Ty, VTy};
6515:     return EmitNeonCall(CGM.getIntrinsic(Int, Tys), Ops, "vmaxv");
6516:   }
6517:   case NEON::BI__builtin_neon_vmaxvq_f16: {
6518:     Int = Intrinsic::aarch64_neon_fmaxv;
6519:     Ty = HalfTy;
6520:     VTy = llvm::FixedVectorType::get(HalfTy, 8);
6521:     llvm::Type *Tys[2] = {Ty, VTy};
6522:     return EmitNeonCall(CGM.getIntrinsic(Int, Tys), Ops, "vmaxv");
6523:   }
6524:   case NEON::BI__builtin_neon_vminv_f16: {
6525:     Int = Intrinsic::aarch64_neon_fminv;
6526:     Ty = HalfTy;
6527:     VTy = llvm::FixedVectorType::get(HalfTy, 4);
6528:     llvm::Type *Tys[2] = {Ty, VTy};
6529:     return EmitNeonCall(CGM.getIntrinsic(Int, Tys), Ops, "vminv");
6530:   }
6531:   case NEON::BI__builtin_neon_vminvq_f16: {
6532:     Int = Intrinsic::aarch64_neon_fminv;
6533:     Ty = HalfTy;
6534:     VTy = llvm::FixedVectorType::get(HalfTy, 8);
6535:     llvm::Type *Tys[2] = {Ty, VTy};
6536:     return EmitNeonCall(CGM.getIntrinsic(Int, Tys), Ops, "vminv");
6537:   }
6538:   case NEON::BI__builtin_neon_vmaxnmv_f16: {
6539:     Int = Intrinsic::aarch64_neon_fmaxnmv;
6540:     Ty = HalfTy;
```
- **EN**: This block defines callable entry points like `EmitNeonCall`; uses control flow (case) to specialize target builtin lowering.
- **CN**: 该代码块定义可调用入口，例如 `EmitNeonCall`；通过控制流（case）细化 目标内建函数降级 行为。

### Lines 6541-6570
```cpp
6541:     VTy = llvm::FixedVectorType::get(HalfTy, 4);
6542:     llvm::Type *Tys[2] = {Ty, VTy};
6543:     return EmitNeonCall(CGM.getIntrinsic(Int, Tys), Ops, "vmaxnmv");
6544:   }
6545:   case NEON::BI__builtin_neon_vmaxnmvq_f16: {
6546:     Int = Intrinsic::aarch64_neon_fmaxnmv;
6547:     Ty = HalfTy;
6548:     VTy = llvm::FixedVectorType::get(HalfTy, 8);
6549:     llvm::Type *Tys[2] = {Ty, VTy};
6550:     return EmitNeonCall(CGM.getIntrinsic(Int, Tys), Ops, "vmaxnmv");
6551:   }
6552:   case NEON::BI__builtin_neon_vminnmv_f16: {
6553:     Int = Intrinsic::aarch64_neon_fminnmv;
6554:     Ty = HalfTy;
6555:     VTy = llvm::FixedVectorType::get(HalfTy, 4);
6556:     llvm::Type *Tys[2] = {Ty, VTy};
6557:     return EmitNeonCall(CGM.getIntrinsic(Int, Tys), Ops, "vminnmv");
6558:     return Builder.CreateTrunc(Ops[0], HalfTy);
6559:   }
6560:   case NEON::BI__builtin_neon_vminnmvq_f16: {
6561:     Int = Intrinsic::aarch64_neon_fminnmv;
6562:     Ty = HalfTy;
6563:     VTy = llvm::FixedVectorType::get(HalfTy, 8);
6564:     llvm::Type *Tys[2] = {Ty, VTy};
6565:     return EmitNeonCall(CGM.getIntrinsic(Int, Tys), Ops, "vminnmv");
6566:   }
6567:   case NEON::BI__builtin_neon_vmul_n_f64: {
6568:     Ops[0] = Builder.CreateBitCast(Ops[0], DoubleTy);
6569:     Value *RHS = Builder.CreateBitCast(Ops[1], DoubleTy);
6570:     return Builder.CreateFMul(Ops[0], RHS);
```
- **EN**: This block defines callable entry points like `EmitNeonCall`; uses control flow (case) to specialize target builtin lowering.
- **CN**: 该代码块定义可调用入口，例如 `EmitNeonCall`；通过控制流（case）细化 目标内建函数降级 行为。

### Lines 6571-6600
```cpp
6571:   }
6572:   case NEON::BI__builtin_neon_vaddlv_u8:
6573:   case NEON::BI__builtin_neon_vaddlvq_u8:
6574:   case NEON::BI__builtin_neon_vaddlv_u16:
6575:   case NEON::BI__builtin_neon_vaddlvq_u16: {
6576:     Int = Intrinsic::aarch64_neon_uaddlv;
6577:     Ty = Int32Ty;
6578:     VTy = cast<llvm::FixedVectorType>(Ops[0]->getType());
6579:     llvm::Type *Tys[2] = {Ty, VTy};
6580:     Value *Result = EmitNeonCall(CGM.getIntrinsic(Int, Tys), Ops, "vaddlv");
6581:     if (VTy->getElementType()->getPrimitiveSizeInBits() == 8)
6582:       return Builder.CreateTrunc(Result, Int16Ty);
6583:     return Result;
6584:   }
6585:   case NEON::BI__builtin_neon_vaddlv_s8:
6586:   case NEON::BI__builtin_neon_vaddlvq_s8:
6587:   case NEON::BI__builtin_neon_vaddlv_s16:
6588:   case NEON::BI__builtin_neon_vaddlvq_s16: {
6589:     Int = Intrinsic::aarch64_neon_saddlv;
6590:     Ty = Int32Ty;
6591:     VTy = cast<llvm::FixedVectorType>(Ops[0]->getType());
6592:     llvm::Type *Tys[2] = {Ty, VTy};
6593:     Value *Result = EmitNeonCall(CGM.getIntrinsic(Int, Tys), Ops, "vaddlv");
6594:     if (VTy->getElementType()->getPrimitiveSizeInBits() == 8)
6595:       return Builder.CreateTrunc(Result, Int16Ty);
6596:     return Result;
6597:   }
6598:   case NEON::BI__builtin_neon_vsri_n_v:
6599:   case NEON::BI__builtin_neon_vsriq_n_v: {
6600:     Int = Intrinsic::aarch64_neon_vsri;
```
- **EN**: This block uses control flow (if, case) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（if, case）细化 目标内建函数降级 行为。

### Lines 6601-6630
```cpp
6601:     llvm::Function *Intrin = CGM.getIntrinsic(Int, Ty);
6602:     return EmitNeonCall(Intrin, Ops, "vsri_n");
6603:   }
6604:   case NEON::BI__builtin_neon_vsli_n_v:
6605:   case NEON::BI__builtin_neon_vsliq_n_v: {
6606:     Int = Intrinsic::aarch64_neon_vsli;
6607:     llvm::Function *Intrin = CGM.getIntrinsic(Int, Ty);
6608:     return EmitNeonCall(Intrin, Ops, "vsli_n");
6609:   }
6610:   case NEON::BI__builtin_neon_vsra_n_v:
6611:   case NEON::BI__builtin_neon_vsraq_n_v:
6612:     Ops[0] = Builder.CreateBitCast(Ops[0], Ty);
6613:     Ops[1] = EmitNeonRShiftImm(Ops[1], Ops[2], Ty, usgn, "vsra_n");
6614:     return Builder.CreateAdd(Ops[0], Ops[1]);
6615:   case NEON::BI__builtin_neon_vrsra_n_v:
6616:   case NEON::BI__builtin_neon_vrsraq_n_v: {
6617:     Int = usgn ? Intrinsic::aarch64_neon_urshl : Intrinsic::aarch64_neon_srshl;
6618:     SmallVector<llvm::Value*,2> TmpOps;
6619:     TmpOps.push_back(Ops[1]);
6620:     TmpOps.push_back(Ops[2]);
6621:     Function* F = CGM.getIntrinsic(Int, Ty);
6622:     llvm::Value *tmp = EmitNeonCall(F, TmpOps, "vrshr_n", 1, true);
6623:     Ops[0] = Builder.CreateBitCast(Ops[0], VTy);
6624:     return Builder.CreateAdd(Ops[0], tmp);
6625:   }
6626:   case NEON::BI__builtin_neon_vld1_v:
6627:   case NEON::BI__builtin_neon_vld1q_v: {
6628:     return Builder.CreateAlignedLoad(VTy, Ops[0], PtrOp0.getAlignment());
6629:   }
6630:   case NEON::BI__builtin_neon_vst1_v:
```
- **EN**: This block defines callable entry points like `EmitNeonCall`; uses control flow (case) to specialize target builtin lowering.
- **CN**: 该代码块定义可调用入口，例如 `EmitNeonCall`；通过控制流（case）细化 目标内建函数降级 行为。

### Lines 6631-6660
```cpp
6631:   case NEON::BI__builtin_neon_vst1q_v:
6632:     Ops[1] = Builder.CreateBitCast(Ops[1], VTy);
6633:     return Builder.CreateAlignedStore(Ops[1], Ops[0], PtrOp0.getAlignment());
6634:   case NEON::BI__builtin_neon_vld1_lane_v:
6635:   case NEON::BI__builtin_neon_vld1q_lane_v: {
6636:     Ops[1] = Builder.CreateBitCast(Ops[1], Ty);
6637:     Ops[0] = Builder.CreateAlignedLoad(VTy->getElementType(), Ops[0],
6638:                                        PtrOp0.getAlignment());
6639:     return Builder.CreateInsertElement(Ops[1], Ops[0], Ops[2], "vld1_lane");
6640:   }
6641:   case NEON::BI__builtin_neon_vldap1_lane_s64:
6642:   case NEON::BI__builtin_neon_vldap1q_lane_s64: {
6643:     Ops[1] = Builder.CreateBitCast(Ops[1], Ty);
6644:     llvm::LoadInst *LI = Builder.CreateAlignedLoad(
6645:         VTy->getElementType(), Ops[0], PtrOp0.getAlignment());
6646:     LI->setAtomic(llvm::AtomicOrdering::Acquire);
6647:     Ops[0] = LI;
6648:     return Builder.CreateInsertElement(Ops[1], Ops[0], Ops[2], "vldap1_lane");
6649:   }
6650:   case NEON::BI__builtin_neon_vld1_dup_v:
6651:   case NEON::BI__builtin_neon_vld1q_dup_v: {
6652:     Value *V = PoisonValue::get(Ty);
6653:     Ops[0] = Builder.CreateAlignedLoad(VTy->getElementType(), Ops[0],
6654:                                        PtrOp0.getAlignment());
6655:     llvm::Constant *CI = ConstantInt::get(Int32Ty, 0);
6656:     Ops[0] = Builder.CreateInsertElement(V, Ops[0], CI);
6657:     return EmitNeonSplat(Ops[0], CI);
6658:   }
6659:   case NEON::BI__builtin_neon_vst1_lane_v:
6660:   case NEON::BI__builtin_neon_vst1q_lane_v:
```
- **EN**: This block defines callable entry points like `EmitNeonSplat`; uses control flow (case) to specialize target builtin lowering.
- **CN**: 该代码块定义可调用入口，例如 `EmitNeonSplat`；通过控制流（case）细化 目标内建函数降级 行为。

### Lines 6661-6690
```cpp
6661:     Ops[1] = Builder.CreateBitCast(Ops[1], Ty);
6662:     Ops[1] = Builder.CreateExtractElement(Ops[1], Ops[2]);
6663:     return Builder.CreateAlignedStore(Ops[1], Ops[0], PtrOp0.getAlignment());
6664:   case NEON::BI__builtin_neon_vstl1_lane_s64:
6665:   case NEON::BI__builtin_neon_vstl1q_lane_s64: {
6666:     Ops[1] = Builder.CreateBitCast(Ops[1], Ty);
6667:     Ops[1] = Builder.CreateExtractElement(Ops[1], Ops[2]);
6668:     llvm::StoreInst *SI =
6669:         Builder.CreateAlignedStore(Ops[1], Ops[0], PtrOp0.getAlignment());
6670:     SI->setAtomic(llvm::AtomicOrdering::Release);
6671:     return SI;
6672:   }
6673:   case NEON::BI__builtin_neon_vld2_v:
6674:   case NEON::BI__builtin_neon_vld2q_v: {
6675:     llvm::Type *Tys[2] = {VTy, DefaultPtrTy};
6676:     Function *F = CGM.getIntrinsic(Intrinsic::aarch64_neon_ld2, Tys);
6677:     Ops[1] = Builder.CreateCall(F, Ops[1], "vld2");
6678:     return Builder.CreateDefaultAlignedStore(Ops[1], Ops[0]);
6679:   }
6680:   case NEON::BI__builtin_neon_vld3_v:
6681:   case NEON::BI__builtin_neon_vld3q_v: {
6682:     llvm::Type *Tys[2] = {VTy, DefaultPtrTy};
6683:     Function *F = CGM.getIntrinsic(Intrinsic::aarch64_neon_ld3, Tys);
6684:     Ops[1] = Builder.CreateCall(F, Ops[1], "vld3");
6685:     return Builder.CreateDefaultAlignedStore(Ops[1], Ops[0]);
6686:   }
6687:   case NEON::BI__builtin_neon_vld4_v:
6688:   case NEON::BI__builtin_neon_vld4q_v: {
6689:     llvm::Type *Tys[2] = {VTy, DefaultPtrTy};
6690:     Function *F = CGM.getIntrinsic(Intrinsic::aarch64_neon_ld4, Tys);
```
- **EN**: This block uses control flow (case) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（case）细化 目标内建函数降级 行为。

### Lines 6691-6720
```cpp
6691:     Ops[1] = Builder.CreateCall(F, Ops[1], "vld4");
6692:     return Builder.CreateDefaultAlignedStore(Ops[1], Ops[0]);
6693:   }
6694:   case NEON::BI__builtin_neon_vld2_dup_v:
6695:   case NEON::BI__builtin_neon_vld2q_dup_v: {
6696:     llvm::Type *Tys[2] = {VTy, DefaultPtrTy};
6697:     Function *F = CGM.getIntrinsic(Intrinsic::aarch64_neon_ld2r, Tys);
6698:     Ops[1] = Builder.CreateCall(F, Ops[1], "vld2");
6699:     return Builder.CreateDefaultAlignedStore(Ops[1], Ops[0]);
6700:   }
6701:   case NEON::BI__builtin_neon_vld3_dup_v:
6702:   case NEON::BI__builtin_neon_vld3q_dup_v: {
6703:     llvm::Type *Tys[2] = {VTy, DefaultPtrTy};
6704:     Function *F = CGM.getIntrinsic(Intrinsic::aarch64_neon_ld3r, Tys);
6705:     Ops[1] = Builder.CreateCall(F, Ops[1], "vld3");
6706:     return Builder.CreateDefaultAlignedStore(Ops[1], Ops[0]);
6707:   }
6708:   case NEON::BI__builtin_neon_vld4_dup_v:
6709:   case NEON::BI__builtin_neon_vld4q_dup_v: {
6710:     llvm::Type *Tys[2] = {VTy, DefaultPtrTy};
6711:     Function *F = CGM.getIntrinsic(Intrinsic::aarch64_neon_ld4r, Tys);
6712:     Ops[1] = Builder.CreateCall(F, Ops[1], "vld4");
6713:     return Builder.CreateDefaultAlignedStore(Ops[1], Ops[0]);
6714:   }
6715:   case NEON::BI__builtin_neon_vld2_lane_v:
6716:   case NEON::BI__builtin_neon_vld2q_lane_v: {
6717:     llvm::Type *Tys[2] = { VTy, Ops[1]->getType() };
6718:     Function *F = CGM.getIntrinsic(Intrinsic::aarch64_neon_ld2lane, Tys);
6719:     std::rotate(Ops.begin() + 1, Ops.begin() + 2, Ops.end());
6720:     Ops[1] = Builder.CreateBitCast(Ops[1], Ty);
```
- **EN**: This block defines callable entry points like `rotate`; uses control flow (case) to specialize target builtin lowering.
- **CN**: 该代码块定义可调用入口，例如 `rotate`；通过控制流（case）细化 目标内建函数降级 行为。

### Lines 6721-6750
```cpp
6721:     Ops[2] = Builder.CreateBitCast(Ops[2], Ty);
6722:     Ops[3] = Builder.CreateZExt(Ops[3], Int64Ty);
6723:     Ops[1] = Builder.CreateCall(F, ArrayRef(Ops).slice(1), "vld2_lane");
6724:     return Builder.CreateDefaultAlignedStore(Ops[1], Ops[0]);
6725:   }
6726:   case NEON::BI__builtin_neon_vld3_lane_v:
6727:   case NEON::BI__builtin_neon_vld3q_lane_v: {
6728:     llvm::Type *Tys[2] = { VTy, Ops[1]->getType() };
6729:     Function *F = CGM.getIntrinsic(Intrinsic::aarch64_neon_ld3lane, Tys);
6730:     std::rotate(Ops.begin() + 1, Ops.begin() + 2, Ops.end());
6731:     Ops[1] = Builder.CreateBitCast(Ops[1], Ty);
6732:     Ops[2] = Builder.CreateBitCast(Ops[2], Ty);
6733:     Ops[3] = Builder.CreateBitCast(Ops[3], Ty);
6734:     Ops[4] = Builder.CreateZExt(Ops[4], Int64Ty);
6735:     Ops[1] = Builder.CreateCall(F, ArrayRef(Ops).slice(1), "vld3_lane");
6736:     return Builder.CreateDefaultAlignedStore(Ops[1], Ops[0]);
6737:   }
6738:   case NEON::BI__builtin_neon_vld4_lane_v:
6739:   case NEON::BI__builtin_neon_vld4q_lane_v: {
6740:     llvm::Type *Tys[2] = { VTy, Ops[1]->getType() };
6741:     Function *F = CGM.getIntrinsic(Intrinsic::aarch64_neon_ld4lane, Tys);
6742:     std::rotate(Ops.begin() + 1, Ops.begin() + 2, Ops.end());
6743:     Ops[1] = Builder.CreateBitCast(Ops[1], Ty);
6744:     Ops[2] = Builder.CreateBitCast(Ops[2], Ty);
6745:     Ops[3] = Builder.CreateBitCast(Ops[3], Ty);
6746:     Ops[4] = Builder.CreateBitCast(Ops[4], Ty);
6747:     Ops[5] = Builder.CreateZExt(Ops[5], Int64Ty);
6748:     Ops[1] = Builder.CreateCall(F, ArrayRef(Ops).slice(1), "vld4_lane");
6749:     return Builder.CreateDefaultAlignedStore(Ops[1], Ops[0]);
6750:   }
```
- **EN**: This block defines callable entry points like `rotate`; uses control flow (case) to specialize target builtin lowering.
- **CN**: 该代码块定义可调用入口，例如 `rotate`；通过控制流（case）细化 目标内建函数降级 行为。

### Lines 6751-6780
```cpp
6751:   case NEON::BI__builtin_neon_vst2_v:
6752:   case NEON::BI__builtin_neon_vst2q_v: {
6753:     std::rotate(Ops.begin(), Ops.begin() + 1, Ops.end());
6754:     llvm::Type *Tys[2] = { VTy, Ops[2]->getType() };
6755:     return EmitNeonCall(CGM.getIntrinsic(Intrinsic::aarch64_neon_st2, Tys),
6756:                         Ops, "");
6757:   }
6758:   case NEON::BI__builtin_neon_vst2_lane_v:
6759:   case NEON::BI__builtin_neon_vst2q_lane_v: {
6760:     std::rotate(Ops.begin(), Ops.begin() + 1, Ops.end());
6761:     Ops[2] = Builder.CreateZExt(Ops[2], Int64Ty);
6762:     llvm::Type *Tys[2] = { VTy, Ops[3]->getType() };
6763:     return EmitNeonCall(CGM.getIntrinsic(Intrinsic::aarch64_neon_st2lane, Tys),
6764:                         Ops, "");
6765:   }
6766:   case NEON::BI__builtin_neon_vst3_v:
6767:   case NEON::BI__builtin_neon_vst3q_v: {
6768:     std::rotate(Ops.begin(), Ops.begin() + 1, Ops.end());
6769:     llvm::Type *Tys[2] = { VTy, Ops[3]->getType() };
6770:     return EmitNeonCall(CGM.getIntrinsic(Intrinsic::aarch64_neon_st3, Tys),
6771:                         Ops, "");
6772:   }
6773:   case NEON::BI__builtin_neon_vst3_lane_v:
6774:   case NEON::BI__builtin_neon_vst3q_lane_v: {
6775:     std::rotate(Ops.begin(), Ops.begin() + 1, Ops.end());
6776:     Ops[3] = Builder.CreateZExt(Ops[3], Int64Ty);
6777:     llvm::Type *Tys[2] = { VTy, Ops[4]->getType() };
6778:     return EmitNeonCall(CGM.getIntrinsic(Intrinsic::aarch64_neon_st3lane, Tys),
6779:                         Ops, "");
6780:   }
```
- **EN**: This block defines callable entry points like `rotate`, `EmitNeonCall`; uses control flow (case) to specialize target builtin lowering.
- **CN**: 该代码块定义可调用入口，例如 `rotate`, `EmitNeonCall`；通过控制流（case）细化 目标内建函数降级 行为。

### Lines 6781-6810
```cpp
6781:   case NEON::BI__builtin_neon_vst4_v:
6782:   case NEON::BI__builtin_neon_vst4q_v: {
6783:     std::rotate(Ops.begin(), Ops.begin() + 1, Ops.end());
6784:     llvm::Type *Tys[2] = { VTy, Ops[4]->getType() };
6785:     return EmitNeonCall(CGM.getIntrinsic(Intrinsic::aarch64_neon_st4, Tys),
6786:                         Ops, "");
6787:   }
6788:   case NEON::BI__builtin_neon_vst4_lane_v:
6789:   case NEON::BI__builtin_neon_vst4q_lane_v: {
6790:     std::rotate(Ops.begin(), Ops.begin() + 1, Ops.end());
6791:     Ops[4] = Builder.CreateZExt(Ops[4], Int64Ty);
6792:     llvm::Type *Tys[2] = { VTy, Ops[5]->getType() };
6793:     return EmitNeonCall(CGM.getIntrinsic(Intrinsic::aarch64_neon_st4lane, Tys),
6794:                         Ops, "");
6795:   }
6796:   case NEON::BI__builtin_neon_vtrn_v:
6797:   case NEON::BI__builtin_neon_vtrnq_v: {
6798:     Ops[1] = Builder.CreateBitCast(Ops[1], Ty);
6799:     Ops[2] = Builder.CreateBitCast(Ops[2], Ty);
6800:     Value *SV = nullptr;
6801: 
6802:     for (unsigned vi = 0; vi != 2; ++vi) {
6803:       SmallVector<int, 16> Indices;
6804:       for (unsigned i = 0, e = VTy->getNumElements(); i != e; i += 2) {
6805:         Indices.push_back(i+vi);
6806:         Indices.push_back(i+e+vi);
6807:       }
6808:       Value *Addr = Builder.CreateConstInBoundsGEP1_32(Ty, Ops[0], vi);
6809:       SV = Builder.CreateShuffleVector(Ops[1], Ops[2], Indices, "vtrn");
6810:       SV = Builder.CreateDefaultAlignedStore(SV, Addr);
```
- **EN**: This block defines callable entry points like `rotate`, `EmitNeonCall`; uses control flow (for, case) to specialize target builtin lowering.
- **CN**: 该代码块定义可调用入口，例如 `rotate`, `EmitNeonCall`；通过控制流（for, case）细化 目标内建函数降级 行为。

### Lines 6811-6840
```cpp
6811:     }
6812:     return SV;
6813:   }
6814:   case NEON::BI__builtin_neon_vuzp_v:
6815:   case NEON::BI__builtin_neon_vuzpq_v: {
6816:     Ops[1] = Builder.CreateBitCast(Ops[1], Ty);
6817:     Ops[2] = Builder.CreateBitCast(Ops[2], Ty);
6818:     Value *SV = nullptr;
6819: 
6820:     for (unsigned vi = 0; vi != 2; ++vi) {
6821:       SmallVector<int, 16> Indices;
6822:       for (unsigned i = 0, e = VTy->getNumElements(); i != e; ++i)
6823:         Indices.push_back(2*i+vi);
6824: 
6825:       Value *Addr = Builder.CreateConstInBoundsGEP1_32(Ty, Ops[0], vi);
6826:       SV = Builder.CreateShuffleVector(Ops[1], Ops[2], Indices, "vuzp");
6827:       SV = Builder.CreateDefaultAlignedStore(SV, Addr);
6828:     }
6829:     return SV;
6830:   }
6831:   case NEON::BI__builtin_neon_vzip_v:
6832:   case NEON::BI__builtin_neon_vzipq_v: {
6833:     Ops[1] = Builder.CreateBitCast(Ops[1], Ty);
6834:     Ops[2] = Builder.CreateBitCast(Ops[2], Ty);
6835:     Value *SV = nullptr;
6836: 
6837:     for (unsigned vi = 0; vi != 2; ++vi) {
6838:       SmallVector<int, 16> Indices;
6839:       for (unsigned i = 0, e = VTy->getNumElements(); i != e; i += 2) {
6840:         Indices.push_back((i + vi*e) >> 1);
```
- **EN**: This block uses control flow (for, case) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（for, case）细化 目标内建函数降级 行为。

### Lines 6841-6870
```cpp
6841:         Indices.push_back(((i + vi*e) >> 1)+e);
6842:       }
6843:       Value *Addr = Builder.CreateConstInBoundsGEP1_32(Ty, Ops[0], vi);
6844:       SV = Builder.CreateShuffleVector(Ops[1], Ops[2], Indices, "vzip");
6845:       SV = Builder.CreateDefaultAlignedStore(SV, Addr);
6846:     }
6847:     return SV;
6848:   }
6849:   case NEON::BI__builtin_neon_vqtbl1q_v: {
6850:     return EmitNeonCall(CGM.getIntrinsic(Intrinsic::aarch64_neon_tbl1, Ty),
6851:                         Ops, "vtbl1");
6852:   }
6853:   case NEON::BI__builtin_neon_vqtbl2q_v: {
6854:     return EmitNeonCall(CGM.getIntrinsic(Intrinsic::aarch64_neon_tbl2, Ty),
6855:                         Ops, "vtbl2");
6856:   }
6857:   case NEON::BI__builtin_neon_vqtbl3q_v: {
6858:     return EmitNeonCall(CGM.getIntrinsic(Intrinsic::aarch64_neon_tbl3, Ty),
6859:                         Ops, "vtbl3");
6860:   }
6861:   case NEON::BI__builtin_neon_vqtbl4q_v: {
6862:     return EmitNeonCall(CGM.getIntrinsic(Intrinsic::aarch64_neon_tbl4, Ty),
6863:                         Ops, "vtbl4");
6864:   }
6865:   case NEON::BI__builtin_neon_vqtbx1q_v: {
6866:     return EmitNeonCall(CGM.getIntrinsic(Intrinsic::aarch64_neon_tbx1, Ty),
6867:                         Ops, "vtbx1");
6868:   }
6869:   case NEON::BI__builtin_neon_vqtbx2q_v: {
6870:     return EmitNeonCall(CGM.getIntrinsic(Intrinsic::aarch64_neon_tbx2, Ty),
```
- **EN**: This block defines callable entry points like `EmitNeonCall`; uses control flow (case) to specialize target builtin lowering.
- **CN**: 该代码块定义可调用入口，例如 `EmitNeonCall`；通过控制流（case）细化 目标内建函数降级 行为。

### Lines 6871-6900
```cpp
6871:                         Ops, "vtbx2");
6872:   }
6873:   case NEON::BI__builtin_neon_vqtbx3q_v: {
6874:     return EmitNeonCall(CGM.getIntrinsic(Intrinsic::aarch64_neon_tbx3, Ty),
6875:                         Ops, "vtbx3");
6876:   }
6877:   case NEON::BI__builtin_neon_vqtbx4q_v: {
6878:     return EmitNeonCall(CGM.getIntrinsic(Intrinsic::aarch64_neon_tbx4, Ty),
6879:                         Ops, "vtbx4");
6880:   }
6881:   case NEON::BI__builtin_neon_vsqadd_v:
6882:   case NEON::BI__builtin_neon_vsqaddq_v: {
6883:     Int = Intrinsic::aarch64_neon_usqadd;
6884:     return EmitNeonCall(CGM.getIntrinsic(Int, Ty), Ops, "vsqadd");
6885:   }
6886:   case NEON::BI__builtin_neon_vuqadd_v:
6887:   case NEON::BI__builtin_neon_vuqaddq_v: {
6888:     Int = Intrinsic::aarch64_neon_suqadd;
6889:     return EmitNeonCall(CGM.getIntrinsic(Int, Ty), Ops, "vuqadd");
6890:   }
6891: 
6892:   case NEON::BI__builtin_neon_vluti2_laneq_mf8:
6893:   case NEON::BI__builtin_neon_vluti2_laneq_bf16:
6894:   case NEON::BI__builtin_neon_vluti2_laneq_f16:
6895:   case NEON::BI__builtin_neon_vluti2_laneq_p16:
6896:   case NEON::BI__builtin_neon_vluti2_laneq_p8:
6897:   case NEON::BI__builtin_neon_vluti2_laneq_s16:
6898:   case NEON::BI__builtin_neon_vluti2_laneq_s8:
6899:   case NEON::BI__builtin_neon_vluti2_laneq_u16:
6900:   case NEON::BI__builtin_neon_vluti2_laneq_u8: {
```
- **EN**: This block defines callable entry points like `EmitNeonCall`; uses control flow (case) to specialize target builtin lowering.
- **CN**: 该代码块定义可调用入口，例如 `EmitNeonCall`；通过控制流（case）细化 目标内建函数降级 行为。

### Lines 6901-6930
```cpp
6901:     Int = Intrinsic::aarch64_neon_vluti2_laneq;
6902:     llvm::Type *Tys[2];
6903:     Tys[0] = Ty;
6904:     Tys[1] = GetNeonType(this, NeonTypeFlags(Type.getEltType(), false,
6905:                                              /*isQuad*/ false));
6906:     return EmitNeonCall(CGM.getIntrinsic(Int, Tys), Ops, "vluti2_laneq");
6907:   }
6908:   case NEON::BI__builtin_neon_vluti2q_laneq_mf8:
6909:   case NEON::BI__builtin_neon_vluti2q_laneq_bf16:
6910:   case NEON::BI__builtin_neon_vluti2q_laneq_f16:
6911:   case NEON::BI__builtin_neon_vluti2q_laneq_p16:
6912:   case NEON::BI__builtin_neon_vluti2q_laneq_p8:
6913:   case NEON::BI__builtin_neon_vluti2q_laneq_s16:
6914:   case NEON::BI__builtin_neon_vluti2q_laneq_s8:
6915:   case NEON::BI__builtin_neon_vluti2q_laneq_u16:
6916:   case NEON::BI__builtin_neon_vluti2q_laneq_u8: {
6917:     Int = Intrinsic::aarch64_neon_vluti2_laneq;
6918:     llvm::Type *Tys[2];
6919:     Tys[0] = Ty;
6920:     Tys[1] = GetNeonType(this, NeonTypeFlags(Type.getEltType(), false,
6921:                                              /*isQuad*/ true));
6922:     return EmitNeonCall(CGM.getIntrinsic(Int, Tys), Ops, "vluti2_laneq");
6923:   }
6924:   case NEON::BI__builtin_neon_vluti2_lane_mf8:
6925:   case NEON::BI__builtin_neon_vluti2_lane_bf16:
6926:   case NEON::BI__builtin_neon_vluti2_lane_f16:
6927:   case NEON::BI__builtin_neon_vluti2_lane_p16:
6928:   case NEON::BI__builtin_neon_vluti2_lane_p8:
6929:   case NEON::BI__builtin_neon_vluti2_lane_s16:
6930:   case NEON::BI__builtin_neon_vluti2_lane_s8:
```
- **EN**: This block defines callable entry points like `EmitNeonCall`; uses control flow (case) to specialize target builtin lowering.
- **CN**: 该代码块定义可调用入口，例如 `EmitNeonCall`；通过控制流（case）细化 目标内建函数降级 行为。

### Lines 6931-6960
```cpp
6931:   case NEON::BI__builtin_neon_vluti2_lane_u16:
6932:   case NEON::BI__builtin_neon_vluti2_lane_u8: {
6933:     Int = Intrinsic::aarch64_neon_vluti2_lane;
6934:     llvm::Type *Tys[2];
6935:     Tys[0] = Ty;
6936:     Tys[1] = GetNeonType(this, NeonTypeFlags(Type.getEltType(), false,
6937:                                              /*isQuad*/ false));
6938:     return EmitNeonCall(CGM.getIntrinsic(Int, Tys), Ops, "vluti2_lane");
6939:   }
6940:   case NEON::BI__builtin_neon_vluti2q_lane_mf8:
6941:   case NEON::BI__builtin_neon_vluti2q_lane_bf16:
6942:   case NEON::BI__builtin_neon_vluti2q_lane_f16:
6943:   case NEON::BI__builtin_neon_vluti2q_lane_p16:
6944:   case NEON::BI__builtin_neon_vluti2q_lane_p8:
6945:   case NEON::BI__builtin_neon_vluti2q_lane_s16:
6946:   case NEON::BI__builtin_neon_vluti2q_lane_s8:
6947:   case NEON::BI__builtin_neon_vluti2q_lane_u16:
6948:   case NEON::BI__builtin_neon_vluti2q_lane_u8: {
6949:     Int = Intrinsic::aarch64_neon_vluti2_lane;
6950:     llvm::Type *Tys[2];
6951:     Tys[0] = Ty;
6952:     Tys[1] = GetNeonType(this, NeonTypeFlags(Type.getEltType(), false,
6953:                                              /*isQuad*/ true));
6954:     return EmitNeonCall(CGM.getIntrinsic(Int, Tys), Ops, "vluti2_lane");
6955:   }
6956:   case NEON::BI__builtin_neon_vluti4q_lane_mf8:
6957:   case NEON::BI__builtin_neon_vluti4q_lane_p8:
6958:   case NEON::BI__builtin_neon_vluti4q_lane_s8:
6959:   case NEON::BI__builtin_neon_vluti4q_lane_u8: {
6960:     Int = Intrinsic::aarch64_neon_vluti4q_lane;
```
- **EN**: This block defines callable entry points like `EmitNeonCall`; uses control flow (case) to specialize target builtin lowering.
- **CN**: 该代码块定义可调用入口，例如 `EmitNeonCall`；通过控制流（case）细化 目标内建函数降级 行为。

### Lines 6961-6990
```cpp
6961:     return EmitNeonCall(CGM.getIntrinsic(Int, Ty), Ops, "vluti4q_lane");
6962:   }
6963:   case NEON::BI__builtin_neon_vluti4q_laneq_mf8:
6964:   case NEON::BI__builtin_neon_vluti4q_laneq_p8:
6965:   case NEON::BI__builtin_neon_vluti4q_laneq_s8:
6966:   case NEON::BI__builtin_neon_vluti4q_laneq_u8: {
6967:     Int = Intrinsic::aarch64_neon_vluti4q_laneq;
6968:     return EmitNeonCall(CGM.getIntrinsic(Int, Ty), Ops, "vluti4q_laneq");
6969:   }
6970:   case NEON::BI__builtin_neon_vluti4q_lane_bf16_x2:
6971:   case NEON::BI__builtin_neon_vluti4q_lane_f16_x2:
6972:   case NEON::BI__builtin_neon_vluti4q_lane_p16_x2:
6973:   case NEON::BI__builtin_neon_vluti4q_lane_s16_x2:
6974:   case NEON::BI__builtin_neon_vluti4q_lane_u16_x2: {
6975:     Int = Intrinsic::aarch64_neon_vluti4q_lane_x2;
6976:     return EmitNeonCall(CGM.getIntrinsic(Int, Ty), Ops, "vluti4q_lane_x2");
6977:   }
6978:   case NEON::BI__builtin_neon_vluti4q_laneq_bf16_x2:
6979:   case NEON::BI__builtin_neon_vluti4q_laneq_f16_x2:
6980:   case NEON::BI__builtin_neon_vluti4q_laneq_p16_x2:
6981:   case NEON::BI__builtin_neon_vluti4q_laneq_s16_x2:
6982:   case NEON::BI__builtin_neon_vluti4q_laneq_u16_x2: {
6983:     Int = Intrinsic::aarch64_neon_vluti4q_laneq_x2;
6984:     return EmitNeonCall(CGM.getIntrinsic(Int, Ty), Ops, "vluti4q_laneq_x2");
6985:   }
6986:   case NEON::BI__builtin_neon_vmmlaq_f16_mf8_fpm:
6987:     return EmitFP8NeonCall(Intrinsic::aarch64_neon_fmmla,
6988:                            {llvm::FixedVectorType::get(HalfTy, 8),
6989:                             llvm::FixedVectorType::get(Int8Ty, 16)},
6990:                            Ops, E, "fmmla");
```
- **EN**: This block defines callable entry points like `EmitNeonCall`; uses control flow (case) to specialize target builtin lowering.
- **CN**: 该代码块定义可调用入口，例如 `EmitNeonCall`；通过控制流（case）细化 目标内建函数降级 行为。

### Lines 6991-7020
```cpp
6991:   case NEON::BI__builtin_neon_vmmlaq_f32_mf8_fpm:
6992:     return EmitFP8NeonCall(Intrinsic::aarch64_neon_fmmla,
6993:                            {llvm::FixedVectorType::get(FloatTy, 4),
6994:                             llvm::FixedVectorType::get(Int8Ty, 16)},
6995:                            Ops, E, "fmmla");
6996:   case NEON::BI__builtin_neon_vcvt1_low_bf16_mf8_fpm:
6997:     ExtractLow = true;
6998:     [[fallthrough]];
6999:   case NEON::BI__builtin_neon_vcvt1_bf16_mf8_fpm:
7000:   case NEON::BI__builtin_neon_vcvt1_high_bf16_mf8_fpm:
7001:     return EmitFP8NeonCvtCall(Intrinsic::aarch64_neon_fp8_cvtl1,
7002:                               llvm::FixedVectorType::get(BFloatTy, 8),
7003:                               Ops[0]->getType(), ExtractLow, Ops, E, "vbfcvt1");
7004:   case NEON::BI__builtin_neon_vcvt2_low_bf16_mf8_fpm:
7005:     ExtractLow = true;
7006:     [[fallthrough]];
7007:   case NEON::BI__builtin_neon_vcvt2_bf16_mf8_fpm:
7008:   case NEON::BI__builtin_neon_vcvt2_high_bf16_mf8_fpm:
7009:     return EmitFP8NeonCvtCall(Intrinsic::aarch64_neon_fp8_cvtl2,
7010:                               llvm::FixedVectorType::get(BFloatTy, 8),
7011:                               Ops[0]->getType(), ExtractLow, Ops, E, "vbfcvt2");
7012:   case NEON::BI__builtin_neon_vcvt1_low_f16_mf8_fpm:
7013:     ExtractLow = true;
7014:     [[fallthrough]];
7015:   case NEON::BI__builtin_neon_vcvt1_f16_mf8_fpm:
7016:   case NEON::BI__builtin_neon_vcvt1_high_f16_mf8_fpm:
7017:     return EmitFP8NeonCvtCall(Intrinsic::aarch64_neon_fp8_cvtl1,
7018:                               llvm::FixedVectorType::get(HalfTy, 8),
7019:                               Ops[0]->getType(), ExtractLow, Ops, E, "vbfcvt1");
7020:   case NEON::BI__builtin_neon_vcvt2_low_f16_mf8_fpm:
```
- **EN**: This block defines callable entry points like `EmitFP8NeonCvtCall`; uses control flow (case) to specialize target builtin lowering.
- **CN**: 该代码块定义可调用入口，例如 `EmitFP8NeonCvtCall`；通过控制流（case）细化 目标内建函数降级 行为。

### Lines 7021-7050
```cpp
7021:     ExtractLow = true;
7022:     [[fallthrough]];
7023:   case NEON::BI__builtin_neon_vcvt2_f16_mf8_fpm:
7024:   case NEON::BI__builtin_neon_vcvt2_high_f16_mf8_fpm:
7025:     return EmitFP8NeonCvtCall(Intrinsic::aarch64_neon_fp8_cvtl2,
7026:                               llvm::FixedVectorType::get(HalfTy, 8),
7027:                               Ops[0]->getType(), ExtractLow, Ops, E, "vbfcvt2");
7028:   case NEON::BI__builtin_neon_vcvt_mf8_f32_fpm:
7029:     return EmitFP8NeonCvtCall(Intrinsic::aarch64_neon_fp8_fcvtn,
7030:                               llvm::FixedVectorType::get(Int8Ty, 8),
7031:                               Ops[0]->getType(), false, Ops, E, "vfcvtn");
7032:   case NEON::BI__builtin_neon_vcvt_mf8_f16_fpm:
7033:     return EmitFP8NeonCvtCall(Intrinsic::aarch64_neon_fp8_fcvtn,
7034:                               llvm::FixedVectorType::get(Int8Ty, 8),
7035:                               llvm::FixedVectorType::get(HalfTy, 4), false, Ops,
7036:                               E, "vfcvtn");
7037:   case NEON::BI__builtin_neon_vcvtq_mf8_f16_fpm:
7038:     return EmitFP8NeonCvtCall(Intrinsic::aarch64_neon_fp8_fcvtn,
7039:                               llvm::FixedVectorType::get(Int8Ty, 16),
7040:                               llvm::FixedVectorType::get(HalfTy, 8), false, Ops,
7041:                               E, "vfcvtn");
7042:   case NEON::BI__builtin_neon_vcvt_high_mf8_f32_fpm: {
7043:     llvm::Type *Ty = llvm::FixedVectorType::get(Int8Ty, 16);
7044:     Ops[0] = Builder.CreateInsertVector(Ty, PoisonValue::get(Ty), Ops[0],
7045:                                         uint64_t(0));
7046:     return EmitFP8NeonCvtCall(Intrinsic::aarch64_neon_fp8_fcvtn2, Ty,
7047:                               Ops[1]->getType(), false, Ops, E, "vfcvtn2");
7048:   }
7049: 
7050:   case NEON::BI__builtin_neon_vdot_f16_mf8_fpm:
```
- **EN**: This block defines callable entry points like `EmitFP8NeonCvtCall`, `uint64_t`; uses control flow (case) to specialize target builtin lowering.
- **CN**: 该代码块定义可调用入口，例如 `EmitFP8NeonCvtCall`, `uint64_t`；通过控制流（case）细化 目标内建函数降级 行为。

### Lines 7051-7080
```cpp
7051:   case NEON::BI__builtin_neon_vdotq_f16_mf8_fpm:
7052:     return EmitFP8NeonFDOTCall(Intrinsic::aarch64_neon_fp8_fdot2, false, HalfTy,
7053:                                Ops, E, "fdot2");
7054:   case NEON::BI__builtin_neon_vdot_lane_f16_mf8_fpm:
7055:   case NEON::BI__builtin_neon_vdotq_lane_f16_mf8_fpm:
7056:     ExtendLaneArg = true;
7057:     [[fallthrough]];
7058:   case NEON::BI__builtin_neon_vdot_laneq_f16_mf8_fpm:
7059:   case NEON::BI__builtin_neon_vdotq_laneq_f16_mf8_fpm:
7060:     return EmitFP8NeonFDOTCall(Intrinsic::aarch64_neon_fp8_fdot2_lane,
7061:                                ExtendLaneArg, HalfTy, Ops, E, "fdot2_lane");
7062:   case NEON::BI__builtin_neon_vdot_f32_mf8_fpm:
7063:   case NEON::BI__builtin_neon_vdotq_f32_mf8_fpm:
7064:     return EmitFP8NeonFDOTCall(Intrinsic::aarch64_neon_fp8_fdot4, false,
7065:                                FloatTy, Ops, E, "fdot4");
7066:   case NEON::BI__builtin_neon_vdot_lane_f32_mf8_fpm:
7067:   case NEON::BI__builtin_neon_vdotq_lane_f32_mf8_fpm:
7068:     ExtendLaneArg = true;
7069:     [[fallthrough]];
7070:   case NEON::BI__builtin_neon_vdot_laneq_f32_mf8_fpm:
7071:   case NEON::BI__builtin_neon_vdotq_laneq_f32_mf8_fpm:
7072:     return EmitFP8NeonFDOTCall(Intrinsic::aarch64_neon_fp8_fdot4_lane,
7073:                                ExtendLaneArg, FloatTy, Ops, E, "fdot4_lane");
7074: 
7075:   case NEON::BI__builtin_neon_vmlalbq_f16_mf8_fpm:
7076:     return EmitFP8NeonCall(Intrinsic::aarch64_neon_fp8_fmlalb,
7077:                            {llvm::FixedVectorType::get(HalfTy, 8)}, Ops, E,
7078:                            "vmlal");
7079:   case NEON::BI__builtin_neon_vmlaltq_f16_mf8_fpm:
7080:     return EmitFP8NeonCall(Intrinsic::aarch64_neon_fp8_fmlalt,
```
- **EN**: This block defines callable entry points like `EmitFP8NeonFDOTCall`; uses control flow (case) to specialize target builtin lowering.
- **CN**: 该代码块定义可调用入口，例如 `EmitFP8NeonFDOTCall`；通过控制流（case）细化 目标内建函数降级 行为。

### Lines 7081-7110
```cpp
7081:                            {llvm::FixedVectorType::get(HalfTy, 8)}, Ops, E,
7082:                            "vmlal");
7083:   case NEON::BI__builtin_neon_vmlallbbq_f32_mf8_fpm:
7084:     return EmitFP8NeonCall(Intrinsic::aarch64_neon_fp8_fmlallbb,
7085:                            {llvm::FixedVectorType::get(FloatTy, 4)}, Ops, E,
7086:                            "vmlall");
7087:   case NEON::BI__builtin_neon_vmlallbtq_f32_mf8_fpm:
7088:     return EmitFP8NeonCall(Intrinsic::aarch64_neon_fp8_fmlallbt,
7089:                            {llvm::FixedVectorType::get(FloatTy, 4)}, Ops, E,
7090:                            "vmlall");
7091:   case NEON::BI__builtin_neon_vmlalltbq_f32_mf8_fpm:
7092:     return EmitFP8NeonCall(Intrinsic::aarch64_neon_fp8_fmlalltb,
7093:                            {llvm::FixedVectorType::get(FloatTy, 4)}, Ops, E,
7094:                            "vmlall");
7095:   case NEON::BI__builtin_neon_vmlallttq_f32_mf8_fpm:
7096:     return EmitFP8NeonCall(Intrinsic::aarch64_neon_fp8_fmlalltt,
7097:                            {llvm::FixedVectorType::get(FloatTy, 4)}, Ops, E,
7098:                            "vmlall");
7099:   case NEON::BI__builtin_neon_vmlalbq_lane_f16_mf8_fpm:
7100:     ExtendLaneArg = true;
7101:     [[fallthrough]];
7102:   case NEON::BI__builtin_neon_vmlalbq_laneq_f16_mf8_fpm:
7103:     return EmitFP8NeonFMLACall(Intrinsic::aarch64_neon_fp8_fmlalb_lane,
7104:                                ExtendLaneArg, HalfTy, Ops, E, "vmlal_lane");
7105:   case NEON::BI__builtin_neon_vmlaltq_lane_f16_mf8_fpm:
7106:     ExtendLaneArg = true;
7107:     [[fallthrough]];
7108:   case NEON::BI__builtin_neon_vmlaltq_laneq_f16_mf8_fpm:
7109:     return EmitFP8NeonFMLACall(Intrinsic::aarch64_neon_fp8_fmlalt_lane,
7110:                                ExtendLaneArg, HalfTy, Ops, E, "vmlal_lane");
```
- **EN**: This block defines callable entry points like `EmitFP8NeonFMLACall`; uses control flow (case) to specialize target builtin lowering.
- **CN**: 该代码块定义可调用入口，例如 `EmitFP8NeonFMLACall`；通过控制流（case）细化 目标内建函数降级 行为。

### Lines 7111-7140
```cpp
7111:   case NEON::BI__builtin_neon_vmlallbbq_lane_f32_mf8_fpm:
7112:     ExtendLaneArg = true;
7113:     [[fallthrough]];
7114:   case NEON::BI__builtin_neon_vmlallbbq_laneq_f32_mf8_fpm:
7115:     return EmitFP8NeonFMLACall(Intrinsic::aarch64_neon_fp8_fmlallbb_lane,
7116:                                ExtendLaneArg, FloatTy, Ops, E, "vmlall_lane");
7117:   case NEON::BI__builtin_neon_vmlallbtq_lane_f32_mf8_fpm:
7118:     ExtendLaneArg = true;
7119:     [[fallthrough]];
7120:   case NEON::BI__builtin_neon_vmlallbtq_laneq_f32_mf8_fpm:
7121:     return EmitFP8NeonFMLACall(Intrinsic::aarch64_neon_fp8_fmlallbt_lane,
7122:                                ExtendLaneArg, FloatTy, Ops, E, "vmlall_lane");
7123:   case NEON::BI__builtin_neon_vmlalltbq_lane_f32_mf8_fpm:
7124:     ExtendLaneArg = true;
7125:     [[fallthrough]];
7126:   case NEON::BI__builtin_neon_vmlalltbq_laneq_f32_mf8_fpm:
7127:     return EmitFP8NeonFMLACall(Intrinsic::aarch64_neon_fp8_fmlalltb_lane,
7128:                                ExtendLaneArg, FloatTy, Ops, E, "vmlall_lane");
7129:   case NEON::BI__builtin_neon_vmlallttq_lane_f32_mf8_fpm:
7130:     ExtendLaneArg = true;
7131:     [[fallthrough]];
7132:   case NEON::BI__builtin_neon_vmlallttq_laneq_f32_mf8_fpm:
7133:     return EmitFP8NeonFMLACall(Intrinsic::aarch64_neon_fp8_fmlalltt_lane,
7134:                                ExtendLaneArg, FloatTy, Ops, E, "vmlall_lane");
7135:   case NEON::BI__builtin_neon_vamin_f16:
7136:   case NEON::BI__builtin_neon_vaminq_f16:
7137:   case NEON::BI__builtin_neon_vamin_f32:
7138:   case NEON::BI__builtin_neon_vaminq_f32:
7139:   case NEON::BI__builtin_neon_vaminq_f64: {
7140:     Int = Intrinsic::aarch64_neon_famin;
```
- **EN**: This block defines callable entry points like `EmitFP8NeonFMLACall`; uses control flow (case) to specialize target builtin lowering.
- **CN**: 该代码块定义可调用入口，例如 `EmitFP8NeonFMLACall`；通过控制流（case）细化 目标内建函数降级 行为。

### Lines 7141-7170
```cpp
7141:     return EmitNeonCall(CGM.getIntrinsic(Int, Ty), Ops, "famin");
7142:   }
7143:   case NEON::BI__builtin_neon_vamax_f16:
7144:   case NEON::BI__builtin_neon_vamaxq_f16:
7145:   case NEON::BI__builtin_neon_vamax_f32:
7146:   case NEON::BI__builtin_neon_vamaxq_f32:
7147:   case NEON::BI__builtin_neon_vamaxq_f64: {
7148:     Int = Intrinsic::aarch64_neon_famax;
7149:     return EmitNeonCall(CGM.getIntrinsic(Int, Ty), Ops, "famax");
7150:   }
7151:   case NEON::BI__builtin_neon_vscale_f16:
7152:   case NEON::BI__builtin_neon_vscaleq_f16:
7153:   case NEON::BI__builtin_neon_vscale_f32:
7154:   case NEON::BI__builtin_neon_vscaleq_f32:
7155:   case NEON::BI__builtin_neon_vscaleq_f64: {
7156:     Int = Intrinsic::aarch64_neon_fp8_fscale;
7157:     return EmitNeonCall(CGM.getIntrinsic(Int, Ty), Ops, "fscale");
7158:   }
7159:   }
7160: }
7161: 
7162: Value *CodeGenFunction::EmitBPFBuiltinExpr(unsigned BuiltinID,
7163:                                            const CallExpr *E) {
7164:   assert((BuiltinID == BPF::BI__builtin_preserve_field_info ||
7165:           BuiltinID == BPF::BI__builtin_btf_type_id ||
7166:           BuiltinID == BPF::BI__builtin_preserve_type_info ||
7167:           BuiltinID == BPF::BI__builtin_preserve_enum_value) &&
7168:          "unexpected BPF builtin");
7169: 
7170:   // A sequence number, injected into IR builtin functions, to
```
- **EN**: This block defines callable entry points like `EmitNeonCall`; uses control flow (case) to specialize target builtin lowering; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块定义可调用入口，例如 `EmitNeonCall`；通过控制流（case）细化 目标内建函数降级 行为；使用断言或不可达标记保护关键不变量。

### Lines 7171-7200
```cpp
7171:   // prevent CSE given the only difference of the function
7172:   // may just be the debuginfo metadata.
7173:   static uint32_t BuiltinSeqNum;
7174: 
7175:   switch (BuiltinID) {
7176:   default:
7177:     llvm_unreachable("Unexpected BPF builtin");
7178:   case BPF::BI__builtin_preserve_field_info: {
7179:     const Expr *Arg = E->getArg(0);
7180:     bool IsBitField = Arg->IgnoreParens()->getObjectKind() == OK_BitField;
7181: 
7182:     if (!getDebugInfo()) {
7183:       CGM.Error(E->getExprLoc(),
7184:                 "using __builtin_preserve_field_info() without -g");
7185:       return IsBitField ? EmitLValue(Arg).getRawBitFieldPointer(*this)
7186:                         : EmitLValue(Arg).emitRawPointer(*this);
7187:     }
7188: 
7189:     // Enable underlying preserve_*_access_index() generation.
7190:     bool OldIsInPreservedAIRegion = IsInPreservedAIRegion;
7191:     IsInPreservedAIRegion = true;
7192:     Value *FieldAddr = IsBitField ? EmitLValue(Arg).getRawBitFieldPointer(*this)
7193:                                   : EmitLValue(Arg).emitRawPointer(*this);
7194:     IsInPreservedAIRegion = OldIsInPreservedAIRegion;
7195: 
7196:     ConstantInt *C = cast<ConstantInt>(EmitScalarExpr(E->getArg(1)));
7197:     Value *InfoKind = ConstantInt::get(Int64Ty, C->getSExtValue());
7198: 
7199:     // Built the IR for the preserve_field_info intrinsic.
7200:     llvm::Function *FnGetFieldInfo = Intrinsic::getOrInsertDeclaration(
```
- **EN**: This block defines callable entry points like `EmitLValue`; uses control flow (if, switch, case) to specialize target builtin lowering; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块定义可调用入口，例如 `EmitLValue`；通过控制流（if, switch, case）细化 目标内建函数降级 行为；使用断言或不可达标记保护关键不变量。

### Lines 7201-7230
```cpp
7201:         &CGM.getModule(), Intrinsic::bpf_preserve_field_info,
7202:         {FieldAddr->getType()});
7203:     return Builder.CreateCall(FnGetFieldInfo, {FieldAddr, InfoKind});
7204:   }
7205:   case BPF::BI__builtin_btf_type_id:
7206:   case BPF::BI__builtin_preserve_type_info: {
7207:     if (!getDebugInfo()) {
7208:       CGM.Error(E->getExprLoc(), "using builtin function without -g");
7209:       return nullptr;
7210:     }
7211: 
7212:     const Expr *Arg0 = E->getArg(0);
7213:     llvm::DIType *DbgInfo = getDebugInfo()->getOrCreateStandaloneType(
7214:         Arg0->getType(), Arg0->getExprLoc());
7215: 
7216:     ConstantInt *Flag = cast<ConstantInt>(EmitScalarExpr(E->getArg(1)));
7217:     Value *FlagValue = ConstantInt::get(Int64Ty, Flag->getSExtValue());
7218:     Value *SeqNumVal = ConstantInt::get(Int32Ty, BuiltinSeqNum++);
7219: 
7220:     llvm::Function *FnDecl;
7221:     if (BuiltinID == BPF::BI__builtin_btf_type_id)
7222:       FnDecl = Intrinsic::getOrInsertDeclaration(
7223:           &CGM.getModule(), Intrinsic::bpf_btf_type_id, {});
7224:     else
7225:       FnDecl = Intrinsic::getOrInsertDeclaration(
7226:           &CGM.getModule(), Intrinsic::bpf_preserve_type_info, {});
7227:     CallInst *Fn = Builder.CreateCall(FnDecl, {SeqNumVal, FlagValue});
7228:     Fn->setMetadata(LLVMContext::MD_preserve_access_index, DbgInfo);
7229:     return Fn;
7230:   }
```
- **EN**: This block uses control flow (if, case) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（if, case）细化 目标内建函数降级 行为。

### Lines 7231-7260
```cpp
7231:   case BPF::BI__builtin_preserve_enum_value: {
7232:     if (!getDebugInfo()) {
7233:       CGM.Error(E->getExprLoc(), "using builtin function without -g");
7234:       return nullptr;
7235:     }
7236: 
7237:     const Expr *Arg0 = E->getArg(0);
7238:     llvm::DIType *DbgInfo = getDebugInfo()->getOrCreateStandaloneType(
7239:         Arg0->getType(), Arg0->getExprLoc());
7240: 
7241:     // Find enumerator
7242:     const auto *UO = cast<UnaryOperator>(Arg0->IgnoreParens());
7243:     const auto *CE = cast<CStyleCastExpr>(UO->getSubExpr());
7244:     const auto *DR = cast<DeclRefExpr>(CE->getSubExpr());
7245:     const auto *Enumerator = cast<EnumConstantDecl>(DR->getDecl());
7246: 
7247:     auto InitVal = Enumerator->getInitVal();
7248:     std::string InitValStr;
7249:     if (InitVal.isNegative() || InitVal > uint64_t(INT64_MAX))
7250:       InitValStr = std::to_string(InitVal.getSExtValue());
7251:     else
7252:       InitValStr = std::to_string(InitVal.getZExtValue());
7253:     std::string EnumStr = Enumerator->getNameAsString() + ":" + InitValStr;
7254:     Value *EnumStrVal = Builder.CreateGlobalString(EnumStr);
7255: 
7256:     ConstantInt *Flag = cast<ConstantInt>(EmitScalarExpr(E->getArg(1)));
7257:     Value *FlagValue = ConstantInt::get(Int64Ty, Flag->getSExtValue());
7258:     Value *SeqNumVal = ConstantInt::get(Int32Ty, BuiltinSeqNum++);
7259: 
7260:     llvm::Function *IntrinsicFn = Intrinsic::getOrInsertDeclaration(
```
- **EN**: This block uses control flow (if, case) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（if, case）细化 目标内建函数降级 行为。

### Lines 7261-7290
```cpp
7261:         &CGM.getModule(), Intrinsic::bpf_preserve_enum_value, {});
7262:     CallInst *Fn =
7263:         Builder.CreateCall(IntrinsicFn, {SeqNumVal, EnumStrVal, FlagValue});
7264:     Fn->setMetadata(LLVMContext::MD_preserve_access_index, DbgInfo);
7265:     return Fn;
7266:   }
7267:   }
7268: }
7269: 
7270: llvm::Value *CodeGenFunction::
7271: BuildVector(ArrayRef<llvm::Value*> Ops) {
7272:   assert((Ops.size() & (Ops.size() - 1)) == 0 &&
7273:          "Not a power-of-two sized vector!");
7274:   bool AllConstants = true;
7275:   for (unsigned i = 0, e = Ops.size(); i != e && AllConstants; ++i)
7276:     AllConstants &= isa<Constant>(Ops[i]);
7277: 
7278:   // If this is a constant vector, create a ConstantVector.
7279:   if (AllConstants) {
7280:     SmallVector<llvm::Constant*, 16> CstOps;
7281:     for (llvm::Value *Op : Ops)
7282:       CstOps.push_back(cast<Constant>(Op));
7283:     return llvm::ConstantVector::get(CstOps);
7284:   }
7285: 
7286:   // Otherwise, insertelement the values to build the vector.
7287:   Value *Result = llvm::PoisonValue::get(
7288:       llvm::FixedVectorType::get(Ops[0]->getType(), Ops.size()));
7289: 
7290:   for (unsigned i = 0, e = Ops.size(); i != e; ++i)
```
- **EN**: This block defines callable entry points like `BuildVector`, `get`; uses control flow (if, for) to specialize target builtin lowering; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块定义可调用入口，例如 `BuildVector`, `get`；通过控制流（if, for）细化 目标内建函数降级 行为；使用断言或不可达标记保护关键不变量。

### Lines 7291-7320
```cpp
7291:     Result = Builder.CreateInsertElement(Result, Ops[i], Builder.getInt64(i));
7292: 
7293:   return Result;
7294: }
7295: 
7296: Value *CodeGenFunction::EmitAArch64CpuInit() {
7297:   llvm::FunctionType *FTy = llvm::FunctionType::get(VoidTy, false);
7298:   llvm::FunctionCallee Func =
7299:       CGM.CreateRuntimeFunction(FTy, "__init_cpu_features_resolver");
7300:   cast<llvm::GlobalValue>(Func.getCallee())->setDSOLocal(true);
7301:   cast<llvm::GlobalValue>(Func.getCallee())
7302:       ->setDLLStorageClass(llvm::GlobalValue::DefaultStorageClass);
7303:   return Builder.CreateCall(Func);
7304: }
7305: 
7306: Value *CodeGenFunction::EmitAArch64CpuSupports(const CallExpr *E) {
7307:   const Expr *ArgExpr = E->getArg(0)->IgnoreParenCasts();
7308:   StringRef ArgStr = cast<StringLiteral>(ArgExpr)->getString();
7309:   llvm::SmallVector<StringRef, 8> OrigFeatures;
7310:   ArgStr.split(OrigFeatures, "+");
7311:   llvm::SmallVector<StringRef, 8> Features;
7312:   for (StringRef Feature : OrigFeatures) {
7313:     Feature = Feature.trim();
7314:     if (!llvm::AArch64::parseFMVExtension(Feature))
7315:       return Builder.getFalse();
7316:     if (Feature != "default")
7317:       Features.push_back(Feature);
7318:   }
7319:   return EmitAArch64CpuSupports(Features);
7320: }
```
- **EN**: This block defines callable entry points like `EmitAArch64CpuSupports`; uses control flow (if, for) to specialize target builtin lowering.
- **CN**: 该代码块定义可调用入口，例如 `EmitAArch64CpuSupports`；通过控制流（if, for）细化 目标内建函数降级 行为。

### Lines 7321-7346
```cpp
7321: 
7322: llvm::Value *
7323: CodeGenFunction::EmitAArch64CpuSupports(ArrayRef<StringRef> FeaturesStrs) {
7324:   llvm::APInt FeaturesMask = llvm::AArch64::getCpuSupportsMask(FeaturesStrs);
7325:   Value *Result = Builder.getTrue();
7326:   if (FeaturesMask != 0) {
7327:     // Get features from structure in runtime library
7328:     // struct {
7329:     //   unsigned long long features;
7330:     // } __aarch64_cpu_features;
7331:     llvm::Type *STy = llvm::StructType::get(Int64Ty);
7332:     llvm::Constant *AArch64CPUFeatures =
7333:         CGM.CreateRuntimeVariable(STy, "__aarch64_cpu_features");
7334:     cast<llvm::GlobalValue>(AArch64CPUFeatures)->setDSOLocal(true);
7335:     llvm::Value *CpuFeatures = Builder.CreateGEP(
7336:         STy, AArch64CPUFeatures,
7337:         {ConstantInt::get(Int32Ty, 0), ConstantInt::get(Int32Ty, 0)});
7338:     Value *Features = Builder.CreateAlignedLoad(Int64Ty, CpuFeatures,
7339:                                                 CharUnits::fromQuantity(8));
7340:     Value *Mask = Builder.getInt(FeaturesMask.trunc(64));
7341:     Value *Bitset = Builder.CreateAnd(Features, Mask);
7342:     Value *Cmp = Builder.CreateICmpEQ(Bitset, Mask);
7343:     Result = Builder.CreateAnd(Result, Cmp);
7344:   }
7345:   return Result;
7346: }
```
- **EN**: This block defines callable entry points like `EmitAArch64CpuSupports`, `fromQuantity`; uses control flow (if) to specialize target builtin lowering.
- **CN**: 该代码块定义可调用入口，例如 `EmitAArch64CpuSupports`, `fromQuantity`；通过控制流（if）细化 目标内建函数降级 行为。

## Key Concepts / 关键概念

- **Ops**: Frequently referenced symbol that likely anchors a key abstraction in this file. / 高频出现的符号，通常对应此文件中的关键抽象。
- **NEON**: Central symbol in this file's implementation of target builtin lowering. / 是该文件实现 目标内建函数降级 时的核心符号。
- **Builder**: Acts as a construction helper that incrementally assembles target builtin lowering state. / 充当构建辅助器，逐步组装 目标内建函数降级 状态。
- **Value**: Frequently referenced symbol that likely anchors a key abstraction in this file. / 高频出现的符号，通常对应此文件中的关键抽象。
- **Intrinsic**: Frequently referenced symbol that likely anchors a key abstraction in this file. / 高频出现的符号，通常对应此文件中的关键抽象。
- **CGM**: Central symbol in this file's implementation of target builtin lowering. / 是该文件实现 目标内建函数降级 时的核心符号。
- **getIntrinsic**: Frequently referenced symbol that likely anchors a key abstraction in this file. / 高频出现的符号，通常对应此文件中的关键抽象。
- **BuiltinID**: Frequently referenced symbol that likely anchors a key abstraction in this file. / 高频出现的符号，通常对应此文件中的关键抽象。

## Dependencies / 依赖关系

- **Local CodeGen / 本地 CodeGen**: `ABIInfo.h`, `CGBuiltin.h`, `CGDebugInfo.h`, `TargetInfo.h`
- **Clang libraries / Clang 库**: `clang/Basic/AArch64CodeGenUtils.h`, `clang/Basic/TargetBuiltins.h`, `clang/Basic/arm_sve_builtin_cg.inc`, `clang/Basic/BuiltinsAArch64NeonSVEBridge_cg.def`, `clang/Basic/arm_sme_builtin_cg.inc`, `clang/Basic/arm_fp16.inc`, `clang/Basic/arm_neon.inc`, `clang/Basic/arm_mve_builtin_cg.inc`, and 1 more
- **LLVM libraries / LLVM 库**: `llvm/IR/InlineAsm.h`, `llvm/IR/IntrinsicsAArch64.h`, `llvm/IR/IntrinsicsARM.h`, `llvm/IR/IntrinsicsBPF.h`, `llvm/TargetParser/AArch64TargetParser.h`
- **Other headers / 其他头文件**: `numeric`
