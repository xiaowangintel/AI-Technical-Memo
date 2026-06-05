# NVPTX.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `clang/lib/CodeGen/TargetBuiltins/NVPTX.cpp`
- **Repository / 仓库**: `llvm-project` (`/root/xw/llvm-project`)
- **Purpose (EN) / 目的（英文）**: Implements builtin handling and lowering logic for the NVPTX backend.
- **Purpose (CN) / 目的（中文）**: 实现 NVPTX 后端的内建函数处理与降级逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-20
```cpp
 1: //===-------- NVPTX.cpp - Emit LLVM Code for builtins ---------------------===//
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
13: #include "CGBuiltin.h"
14: #include "clang/Basic/TargetBuiltins.h"
15: #include "llvm/IR/IntrinsicsNVPTX.h"
16: 
17: using namespace clang;
18: using namespace CodeGen;
19: using namespace llvm;
20: 
```
- **EN**: This block imports local CodeGen headers `CGBuiltin.h`; Clang headers `clang/Basic/TargetBuiltins.h`; LLVM headers `llvm/IR/IntrinsicsNVPTX.h`; opens or references namespaces `clang`, `CodeGen`, `llvm`; contains preprocessor structure that shapes compilation of this unit.
- **CN**: 该代码块引入 本地 CodeGen 头文件 `CGBuiltin.h`；Clang 头文件 `clang/Basic/TargetBuiltins.h`；LLVM 头文件 `llvm/IR/IntrinsicsNVPTX.h`；打开或引用命名空间 `clang`, `CodeGen`, `llvm`；包含影响本编译单元构建方式的预处理结构。

### Lines 21-40
```cpp
21: namespace {
22: // Helper classes for mapping MMA builtins to particular LLVM intrinsic variant.
23: struct NVPTXMmaLdstInfo {
24:   unsigned NumResults;  // Number of elements to load/store
25:   // Intrinsic IDs for row/col variants. 0 if particular layout is unsupported.
26:   unsigned IID_col;
27:   unsigned IID_row;
28: };
29: 
30: #define MMA_INTR(geom_op_type, layout) \
31:   Intrinsic::nvvm_wmma_##geom_op_type##_##layout##_stride
32: #define MMA_LDST(n, geom_op_type)                                              \
33:   { n, MMA_INTR(geom_op_type, col), MMA_INTR(geom_op_type, row) }
34: 
35: static NVPTXMmaLdstInfo getNVPTXMmaLdstInfo(unsigned BuiltinID) {
36:   switch (BuiltinID) {
37:   // FP MMA loads
38:   case NVPTX::BI__hmma_m16n16k16_ld_a:
39:     return MMA_LDST(8, m16n16k16_load_a_f16);
40:   case NVPTX::BI__hmma_m16n16k16_ld_b:
```
- **EN**: This block introduces declarations such as `NVPTXMmaLdstInfo`; defines callable entry points like `getNVPTXMmaLdstInfo`, `MMA_LDST`; uses control flow (switch, case) to specialize target builtin lowering; contains preprocessor structure that shapes compilation of this unit.
- **CN**: 该代码块给出诸如 `NVPTXMmaLdstInfo` 的声明；定义可调用入口，例如 `getNVPTXMmaLdstInfo`, `MMA_LDST`；通过控制流（switch, case）细化 目标内建函数降级 行为；包含影响本编译单元构建方式的预处理结构。

### Lines 41-60
```cpp
41:     return MMA_LDST(8, m16n16k16_load_b_f16);
42:   case NVPTX::BI__hmma_m16n16k16_ld_c_f16:
43:     return MMA_LDST(4, m16n16k16_load_c_f16);
44:   case NVPTX::BI__hmma_m16n16k16_ld_c_f32:
45:     return MMA_LDST(8, m16n16k16_load_c_f32);
46:   case NVPTX::BI__hmma_m32n8k16_ld_a:
47:     return MMA_LDST(8, m32n8k16_load_a_f16);
48:   case NVPTX::BI__hmma_m32n8k16_ld_b:
49:     return MMA_LDST(8, m32n8k16_load_b_f16);
50:   case NVPTX::BI__hmma_m32n8k16_ld_c_f16:
51:     return MMA_LDST(4, m32n8k16_load_c_f16);
52:   case NVPTX::BI__hmma_m32n8k16_ld_c_f32:
53:     return MMA_LDST(8, m32n8k16_load_c_f32);
54:   case NVPTX::BI__hmma_m8n32k16_ld_a:
55:     return MMA_LDST(8, m8n32k16_load_a_f16);
56:   case NVPTX::BI__hmma_m8n32k16_ld_b:
57:     return MMA_LDST(8, m8n32k16_load_b_f16);
58:   case NVPTX::BI__hmma_m8n32k16_ld_c_f16:
59:     return MMA_LDST(4, m8n32k16_load_c_f16);
60:   case NVPTX::BI__hmma_m8n32k16_ld_c_f32:
```
- **EN**: This block spells out callable entry points like `MMA_LDST`; uses control flow (case) to specialize target builtin lowering.
- **CN**: 该代码块给出可调用入口的声明，例如 `MMA_LDST`；通过控制流（case）细化 目标内建函数降级 行为。

### Lines 61-80
```cpp
61:     return MMA_LDST(8, m8n32k16_load_c_f32);
62: 
63:   // Integer MMA loads
64:   case NVPTX::BI__imma_m16n16k16_ld_a_s8:
65:     return MMA_LDST(2, m16n16k16_load_a_s8);
66:   case NVPTX::BI__imma_m16n16k16_ld_a_u8:
67:     return MMA_LDST(2, m16n16k16_load_a_u8);
68:   case NVPTX::BI__imma_m16n16k16_ld_b_s8:
69:     return MMA_LDST(2, m16n16k16_load_b_s8);
70:   case NVPTX::BI__imma_m16n16k16_ld_b_u8:
71:     return MMA_LDST(2, m16n16k16_load_b_u8);
72:   case NVPTX::BI__imma_m16n16k16_ld_c:
73:     return MMA_LDST(8, m16n16k16_load_c_s32);
74:   case NVPTX::BI__imma_m32n8k16_ld_a_s8:
75:     return MMA_LDST(4, m32n8k16_load_a_s8);
76:   case NVPTX::BI__imma_m32n8k16_ld_a_u8:
77:     return MMA_LDST(4, m32n8k16_load_a_u8);
78:   case NVPTX::BI__imma_m32n8k16_ld_b_s8:
79:     return MMA_LDST(1, m32n8k16_load_b_s8);
80:   case NVPTX::BI__imma_m32n8k16_ld_b_u8:
```
- **EN**: This block spells out callable entry points like `MMA_LDST`; uses control flow (case) to specialize target builtin lowering.
- **CN**: 该代码块给出可调用入口的声明，例如 `MMA_LDST`；通过控制流（case）细化 目标内建函数降级 行为。

### Lines 81-100
```cpp
 81:     return MMA_LDST(1, m32n8k16_load_b_u8);
 82:   case NVPTX::BI__imma_m32n8k16_ld_c:
 83:     return MMA_LDST(8, m32n8k16_load_c_s32);
 84:   case NVPTX::BI__imma_m8n32k16_ld_a_s8:
 85:     return MMA_LDST(1, m8n32k16_load_a_s8);
 86:   case NVPTX::BI__imma_m8n32k16_ld_a_u8:
 87:     return MMA_LDST(1, m8n32k16_load_a_u8);
 88:   case NVPTX::BI__imma_m8n32k16_ld_b_s8:
 89:     return MMA_LDST(4, m8n32k16_load_b_s8);
 90:   case NVPTX::BI__imma_m8n32k16_ld_b_u8:
 91:     return MMA_LDST(4, m8n32k16_load_b_u8);
 92:   case NVPTX::BI__imma_m8n32k16_ld_c:
 93:     return MMA_LDST(8, m8n32k16_load_c_s32);
 94: 
 95:   // Sub-integer MMA loads.
 96:   // Only row/col layout is supported by A/B fragments.
 97:   case NVPTX::BI__imma_m8n8k32_ld_a_s4:
 98:     return {1, 0, MMA_INTR(m8n8k32_load_a_s4, row)};
 99:   case NVPTX::BI__imma_m8n8k32_ld_a_u4:
100:     return {1, 0, MMA_INTR(m8n8k32_load_a_u4, row)};
```
- **EN**: This block defines callable entry points like `MMA_LDST`; uses control flow (case) to specialize target builtin lowering.
- **CN**: 该代码块定义可调用入口，例如 `MMA_LDST`；通过控制流（case）细化 目标内建函数降级 行为。

### Lines 101-120
```cpp
101:   case NVPTX::BI__imma_m8n8k32_ld_b_s4:
102:     return {1, MMA_INTR(m8n8k32_load_b_s4, col), 0};
103:   case NVPTX::BI__imma_m8n8k32_ld_b_u4:
104:     return {1, MMA_INTR(m8n8k32_load_b_u4, col), 0};
105:   case NVPTX::BI__imma_m8n8k32_ld_c:
106:     return MMA_LDST(2, m8n8k32_load_c_s32);
107:   case NVPTX::BI__bmma_m8n8k128_ld_a_b1:
108:     return {1, 0, MMA_INTR(m8n8k128_load_a_b1, row)};
109:   case NVPTX::BI__bmma_m8n8k128_ld_b_b1:
110:     return {1, MMA_INTR(m8n8k128_load_b_b1, col), 0};
111:   case NVPTX::BI__bmma_m8n8k128_ld_c:
112:     return MMA_LDST(2, m8n8k128_load_c_s32);
113: 
114:   // Double MMA loads
115:   case NVPTX::BI__dmma_m8n8k4_ld_a:
116:     return MMA_LDST(1, m8n8k4_load_a_f64);
117:   case NVPTX::BI__dmma_m8n8k4_ld_b:
118:     return MMA_LDST(1, m8n8k4_load_b_f64);
119:   case NVPTX::BI__dmma_m8n8k4_ld_c:
120:     return MMA_LDST(2, m8n8k4_load_c_f64);
```
- **EN**: This block defines callable entry points like `MMA_LDST`; uses control flow (case) to specialize target builtin lowering.
- **CN**: 该代码块定义可调用入口，例如 `MMA_LDST`；通过控制流（case）细化 目标内建函数降级 行为。

### Lines 121-140
```cpp
121: 
122:   // Alternate float MMA loads
123:   case NVPTX::BI__mma_bf16_m16n16k16_ld_a:
124:     return MMA_LDST(4, m16n16k16_load_a_bf16);
125:   case NVPTX::BI__mma_bf16_m16n16k16_ld_b:
126:     return MMA_LDST(4, m16n16k16_load_b_bf16);
127:   case NVPTX::BI__mma_bf16_m8n32k16_ld_a:
128:     return MMA_LDST(2, m8n32k16_load_a_bf16);
129:   case NVPTX::BI__mma_bf16_m8n32k16_ld_b:
130:     return MMA_LDST(8, m8n32k16_load_b_bf16);
131:   case NVPTX::BI__mma_bf16_m32n8k16_ld_a:
132:     return MMA_LDST(8, m32n8k16_load_a_bf16);
133:   case NVPTX::BI__mma_bf16_m32n8k16_ld_b:
134:     return MMA_LDST(2, m32n8k16_load_b_bf16);
135:   case NVPTX::BI__mma_tf32_m16n16k8_ld_a:
136:     return MMA_LDST(4, m16n16k8_load_a_tf32);
137:   case NVPTX::BI__mma_tf32_m16n16k8_ld_b:
138:     return MMA_LDST(4, m16n16k8_load_b_tf32);
139:   case NVPTX::BI__mma_tf32_m16n16k8_ld_c:
140:     return MMA_LDST(8, m16n16k8_load_c_f32);
```
- **EN**: This block spells out callable entry points like `MMA_LDST`; uses control flow (case) to specialize target builtin lowering.
- **CN**: 该代码块给出可调用入口的声明，例如 `MMA_LDST`；通过控制流（case）细化 目标内建函数降级 行为。

### Lines 141-160
```cpp
141: 
142:   // NOTE: We need to follow inconsitent naming scheme used by NVCC.  Unlike
143:   // PTX and LLVM IR where stores always use fragment D, NVCC builtins always
144:   // use fragment C for both loads and stores.
145:   // FP MMA stores.
146:   case NVPTX::BI__hmma_m16n16k16_st_c_f16:
147:     return MMA_LDST(4, m16n16k16_store_d_f16);
148:   case NVPTX::BI__hmma_m16n16k16_st_c_f32:
149:     return MMA_LDST(8, m16n16k16_store_d_f32);
150:   case NVPTX::BI__hmma_m32n8k16_st_c_f16:
151:     return MMA_LDST(4, m32n8k16_store_d_f16);
152:   case NVPTX::BI__hmma_m32n8k16_st_c_f32:
153:     return MMA_LDST(8, m32n8k16_store_d_f32);
154:   case NVPTX::BI__hmma_m8n32k16_st_c_f16:
155:     return MMA_LDST(4, m8n32k16_store_d_f16);
156:   case NVPTX::BI__hmma_m8n32k16_st_c_f32:
157:     return MMA_LDST(8, m8n32k16_store_d_f32);
158: 
159:   // Integer and sub-integer MMA stores.
160:   // Another naming quirk. Unlike other MMA builtins that use PTX types in the
```
- **EN**: This block spells out callable entry points like `MMA_LDST`; uses control flow (case) to specialize target builtin lowering.
- **CN**: 该代码块给出可调用入口的声明，例如 `MMA_LDST`；通过控制流（case）细化 目标内建函数降级 行为。

### Lines 161-180
```cpp
161:   // name, integer loads/stores use LLVM's i32.
162:   case NVPTX::BI__imma_m16n16k16_st_c_i32:
163:     return MMA_LDST(8, m16n16k16_store_d_s32);
164:   case NVPTX::BI__imma_m32n8k16_st_c_i32:
165:     return MMA_LDST(8, m32n8k16_store_d_s32);
166:   case NVPTX::BI__imma_m8n32k16_st_c_i32:
167:     return MMA_LDST(8, m8n32k16_store_d_s32);
168:   case NVPTX::BI__imma_m8n8k32_st_c_i32:
169:     return MMA_LDST(2, m8n8k32_store_d_s32);
170:   case NVPTX::BI__bmma_m8n8k128_st_c_i32:
171:     return MMA_LDST(2, m8n8k128_store_d_s32);
172: 
173:   // Double MMA store
174:   case NVPTX::BI__dmma_m8n8k4_st_c_f64:
175:     return MMA_LDST(2, m8n8k4_store_d_f64);
176: 
177:   // Alternate float MMA store
178:   case NVPTX::BI__mma_m16n16k8_st_c_f32:
179:     return MMA_LDST(8, m16n16k8_store_d_f32);
180: 
```
- **EN**: This block spells out callable entry points like `MMA_LDST`; uses control flow (case) to specialize target builtin lowering.
- **CN**: 该代码块给出可调用入口的声明，例如 `MMA_LDST`；通过控制流（case）细化 目标内建函数降级 行为。

### Lines 181-200
```cpp
181:   default:
182:     llvm_unreachable("Unknown MMA builtin");
183:   }
184: }
185: #undef MMA_LDST
186: #undef MMA_INTR
187: 
188: 
189: struct NVPTXMmaInfo {
190:   unsigned NumEltsA;
191:   unsigned NumEltsB;
192:   unsigned NumEltsC;
193:   unsigned NumEltsD;
194: 
195:   // Variants are ordered by layout-A/layout-B/satf, where 'row' has priority
196:   // over 'col' for layout. The index of non-satf variants is expected to match
197:   // the undocumented layout constants used by CUDA's mma.hpp.
198:   std::array<unsigned, 8> Variants;
199: 
200:   unsigned getMMAIntrinsic(int Layout, bool Satf) {
```
- **EN**: This block introduces declarations such as `NVPTXMmaInfo`; defines callable entry points like `getMMAIntrinsic`; guards important invariants with assertions or unreachable markers; contains preprocessor structure that shapes compilation of this unit.
- **CN**: 该代码块给出诸如 `NVPTXMmaInfo` 的声明；定义可调用入口，例如 `getMMAIntrinsic`；使用断言或不可达标记保护关键不变量；包含影响本编译单元构建方式的预处理结构。

### Lines 201-220
```cpp
201:     unsigned Index = Layout + 4 * Satf;
202:     if (Index >= Variants.size())
203:       return 0;
204:     return Variants[Index];
205:   }
206: };
207: 
208:   // Returns an intrinsic that matches Layout and Satf for valid combinations of
209:   // Layout and Satf, 0 otherwise.
210: static NVPTXMmaInfo getNVPTXMmaInfo(unsigned BuiltinID) {
211:   // clang-format off
212: #define MMA_VARIANTS(geom, type)                                    \
213:       Intrinsic::nvvm_wmma_##geom##_mma_row_row_##type,             \
214:       Intrinsic::nvvm_wmma_##geom##_mma_row_col_##type,             \
215:       Intrinsic::nvvm_wmma_##geom##_mma_col_row_##type,             \
216:       Intrinsic::nvvm_wmma_##geom##_mma_col_col_##type
217: #define MMA_SATF_VARIANTS(geom, type)                               \
218:       MMA_VARIANTS(geom, type),                                     \
219:       Intrinsic::nvvm_wmma_##geom##_mma_row_row_##type##_satfinite, \
220:       Intrinsic::nvvm_wmma_##geom##_mma_row_col_##type##_satfinite, \
```
- **EN**: This block defines callable entry points like `getNVPTXMmaInfo`; uses control flow (if) to specialize target builtin lowering; contains preprocessor structure that shapes compilation of this unit.
- **CN**: 该代码块定义可调用入口，例如 `getNVPTXMmaInfo`；通过控制流（if）细化 目标内建函数降级 行为；包含影响本编译单元构建方式的预处理结构。

### Lines 221-240
```cpp
221:       Intrinsic::nvvm_wmma_##geom##_mma_col_row_##type##_satfinite, \
222:       Intrinsic::nvvm_wmma_##geom##_mma_col_col_##type##_satfinite
223: // Sub-integer MMA only supports row.col layout.
224: #define MMA_VARIANTS_I4(geom, type) \
225:       0, \
226:       Intrinsic::nvvm_wmma_##geom##_mma_row_col_##type,             \
227:       0, \
228:       0, \
229:       0, \
230:       Intrinsic::nvvm_wmma_##geom##_mma_row_col_##type##_satfinite, \
231:       0, \
232:       0
233: // b1 MMA does not support .satfinite.
234: #define MMA_VARIANTS_B1_XOR(geom, type) \
235:       0, \
236:       Intrinsic::nvvm_wmma_##geom##_mma_xor_popc_row_col_##type,             \
237:       0, \
238:       0, \
239:       0, \
240:       0, \
```
- **EN**: This block contains preprocessor structure that shapes compilation of this unit.
- **CN**: 该代码块包含影响本编译单元构建方式的预处理结构。

### Lines 241-260
```cpp
241:       0, \
242:       0
243: #define MMA_VARIANTS_B1_AND(geom, type) \
244:       0, \
245:       Intrinsic::nvvm_wmma_##geom##_mma_and_popc_row_col_##type,             \
246:       0, \
247:       0, \
248:       0, \
249:       0, \
250:       0, \
251:       0
252:   // clang-format on
253:   switch (BuiltinID) {
254:   // FP MMA
255:   // Note that 'type' argument of MMA_SATF_VARIANTS uses D_C notation, while
256:   // NumEltsN of return value are ordered as A,B,C,D.
257:   case NVPTX::BI__hmma_m16n16k16_mma_f16f16:
258:     return {8, 8, 4, 4, {{MMA_SATF_VARIANTS(m16n16k16, f16_f16)}}};
259:   case NVPTX::BI__hmma_m16n16k16_mma_f32f16:
260:     return {8, 8, 4, 8, {{MMA_SATF_VARIANTS(m16n16k16, f32_f16)}}};
```
- **EN**: This block uses control flow (switch, case) to specialize target builtin lowering; contains preprocessor structure that shapes compilation of this unit.
- **CN**: 该代码块通过控制流（switch, case）细化 目标内建函数降级 行为；包含影响本编译单元构建方式的预处理结构。

### Lines 261-280
```cpp
261:   case NVPTX::BI__hmma_m16n16k16_mma_f16f32:
262:     return {8, 8, 8, 4, {{MMA_SATF_VARIANTS(m16n16k16, f16_f32)}}};
263:   case NVPTX::BI__hmma_m16n16k16_mma_f32f32:
264:     return {8, 8, 8, 8, {{MMA_SATF_VARIANTS(m16n16k16, f32_f32)}}};
265:   case NVPTX::BI__hmma_m32n8k16_mma_f16f16:
266:     return {8, 8, 4, 4, {{MMA_SATF_VARIANTS(m32n8k16, f16_f16)}}};
267:   case NVPTX::BI__hmma_m32n8k16_mma_f32f16:
268:     return {8, 8, 4, 8, {{MMA_SATF_VARIANTS(m32n8k16, f32_f16)}}};
269:   case NVPTX::BI__hmma_m32n8k16_mma_f16f32:
270:     return {8, 8, 8, 4, {{MMA_SATF_VARIANTS(m32n8k16, f16_f32)}}};
271:   case NVPTX::BI__hmma_m32n8k16_mma_f32f32:
272:     return {8, 8, 8, 8, {{MMA_SATF_VARIANTS(m32n8k16, f32_f32)}}};
273:   case NVPTX::BI__hmma_m8n32k16_mma_f16f16:
274:     return {8, 8, 4, 4, {{MMA_SATF_VARIANTS(m8n32k16, f16_f16)}}};
275:   case NVPTX::BI__hmma_m8n32k16_mma_f32f16:
276:     return {8, 8, 4, 8, {{MMA_SATF_VARIANTS(m8n32k16, f32_f16)}}};
277:   case NVPTX::BI__hmma_m8n32k16_mma_f16f32:
278:     return {8, 8, 8, 4, {{MMA_SATF_VARIANTS(m8n32k16, f16_f32)}}};
279:   case NVPTX::BI__hmma_m8n32k16_mma_f32f32:
280:     return {8, 8, 8, 8, {{MMA_SATF_VARIANTS(m8n32k16, f32_f32)}}};
```
- **EN**: This block uses control flow (case) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（case）细化 目标内建函数降级 行为。

### Lines 281-300
```cpp
281: 
282:   // Integer MMA
283:   case NVPTX::BI__imma_m16n16k16_mma_s8:
284:     return {2, 2, 8, 8, {{MMA_SATF_VARIANTS(m16n16k16, s8)}}};
285:   case NVPTX::BI__imma_m16n16k16_mma_u8:
286:     return {2, 2, 8, 8, {{MMA_SATF_VARIANTS(m16n16k16, u8)}}};
287:   case NVPTX::BI__imma_m32n8k16_mma_s8:
288:     return {4, 1, 8, 8, {{MMA_SATF_VARIANTS(m32n8k16, s8)}}};
289:   case NVPTX::BI__imma_m32n8k16_mma_u8:
290:     return {4, 1, 8, 8, {{MMA_SATF_VARIANTS(m32n8k16, u8)}}};
291:   case NVPTX::BI__imma_m8n32k16_mma_s8:
292:     return {1, 4, 8, 8, {{MMA_SATF_VARIANTS(m8n32k16, s8)}}};
293:   case NVPTX::BI__imma_m8n32k16_mma_u8:
294:     return {1, 4, 8, 8, {{MMA_SATF_VARIANTS(m8n32k16, u8)}}};
295: 
296:   // Sub-integer MMA
297:   case NVPTX::BI__imma_m8n8k32_mma_s4:
298:     return {1, 1, 2, 2, {{MMA_VARIANTS_I4(m8n8k32, s4)}}};
299:   case NVPTX::BI__imma_m8n8k32_mma_u4:
300:     return {1, 1, 2, 2, {{MMA_VARIANTS_I4(m8n8k32, u4)}}};
```
- **EN**: This block uses control flow (case) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（case）细化 目标内建函数降级 行为。

### Lines 301-320
```cpp
301:   case NVPTX::BI__bmma_m8n8k128_mma_xor_popc_b1:
302:     return {1, 1, 2, 2, {{MMA_VARIANTS_B1_XOR(m8n8k128, b1)}}};
303:   case NVPTX::BI__bmma_m8n8k128_mma_and_popc_b1:
304:     return {1, 1, 2, 2, {{MMA_VARIANTS_B1_AND(m8n8k128, b1)}}};
305: 
306:   // Double MMA
307:   case NVPTX::BI__dmma_m8n8k4_mma_f64:
308:     return {1, 1, 2, 2, {{MMA_VARIANTS(m8n8k4, f64)}}};
309: 
310:   // Alternate FP MMA
311:   case NVPTX::BI__mma_bf16_m16n16k16_mma_f32:
312:     return {4, 4, 8, 8, {{MMA_VARIANTS(m16n16k16, bf16)}}};
313:   case NVPTX::BI__mma_bf16_m8n32k16_mma_f32:
314:     return {2, 8, 8, 8, {{MMA_VARIANTS(m8n32k16, bf16)}}};
315:   case NVPTX::BI__mma_bf16_m32n8k16_mma_f32:
316:     return {8, 2, 8, 8, {{MMA_VARIANTS(m32n8k16, bf16)}}};
317:   case NVPTX::BI__mma_tf32_m16n16k8_mma_f32:
318:     return {4, 4, 8, 8, {{MMA_VARIANTS(m16n16k8, tf32)}}};
319:   default:
320:     llvm_unreachable("Unexpected builtin ID.");
```
- **EN**: This block uses control flow (case) to specialize target builtin lowering; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块通过控制流（case）细化 目标内建函数降级 行为；使用断言或不可达标记保护关键不变量。

### Lines 321-340
```cpp
321:   }
322: #undef MMA_VARIANTS
323: #undef MMA_SATF_VARIANTS
324: #undef MMA_VARIANTS_I4
325: #undef MMA_VARIANTS_B1_AND
326: #undef MMA_VARIANTS_B1_XOR
327: }
328: 
329: static Value *MakeLdu(unsigned IntrinsicID, CodeGenFunction &CGF,
330:                       const CallExpr *E) {
331:   Value *Ptr = CGF.EmitScalarExpr(E->getArg(0));
332:   QualType ArgType = E->getArg(0)->getType();
333:   clang::CharUnits Align = CGF.CGM.getNaturalPointeeTypeAlignment(ArgType);
334:   llvm::Type *ElemTy = CGF.ConvertTypeForMem(ArgType->getPointeeType());
335:   return CGF.Builder.CreateCall(
336:       CGF.CGM.getIntrinsic(IntrinsicID, {ElemTy, Ptr->getType()}),
337:       {Ptr, ConstantInt::get(CGF.Builder.getInt32Ty(), Align.getQuantity())});
338: }
339: 
340: static Value *MakeLdg(CodeGenFunction &CGF, const CallExpr *E) {
```
- **EN**: This block contains preprocessor structure that shapes compilation of this unit.
- **CN**: 该代码块包含影响本编译单元构建方式的预处理结构。

### Lines 341-360
```cpp
341:   Value *Ptr = CGF.EmitScalarExpr(E->getArg(0));
342:   QualType ArgType = E->getArg(0)->getType();
343:   clang::CharUnits AlignV = CGF.CGM.getNaturalPointeeTypeAlignment(ArgType);
344:   llvm::Type *ElemTy = CGF.ConvertTypeForMem(ArgType->getPointeeType());
345: 
346:   // Use addrspace(1) for NVPTX ADDRESS_SPACE_GLOBAL
347:   auto *ASC = CGF.Builder.CreateAddrSpaceCast(Ptr, CGF.Builder.getPtrTy(1));
348:   auto *LD = CGF.Builder.CreateAlignedLoad(ElemTy, ASC, AlignV.getAsAlign());
349:   MDNode *MD = MDNode::get(CGF.Builder.getContext(), {});
350:   LD->setMetadata(LLVMContext::MD_invariant_load, MD);
351: 
352:   return LD;
353: }
354: 
355: static Value *MakeScopedAtomic(unsigned IntrinsicID, CodeGenFunction &CGF,
356:                                const CallExpr *E) {
357:   Value *Ptr = CGF.EmitScalarExpr(E->getArg(0));
358:   llvm::Type *ElemTy =
359:       CGF.ConvertTypeForMem(E->getArg(0)->getType()->getPointeeType());
360:   return CGF.Builder.CreateCall(
```
- **EN**: This block documents intent or context for the surrounding target builtin lowering code.
- **CN**: 该代码块说明周围 目标内建函数降级 代码的意图或上下文。

### Lines 361-380
```cpp
361:       CGF.CGM.getIntrinsic(IntrinsicID, {ElemTy, Ptr->getType()}),
362:       {Ptr, CGF.EmitScalarExpr(E->getArg(1))});
363: }
364: 
365: static Value *MakeCpAsync(unsigned IntrinsicID, unsigned IntrinsicIDS,
366:                           CodeGenFunction &CGF, const CallExpr *E,
367:                           int SrcSize) {
368:   return E->getNumArgs() == 3
369:              ? CGF.Builder.CreateCall(CGF.CGM.getIntrinsic(IntrinsicIDS),
370:                                       {CGF.EmitScalarExpr(E->getArg(0)),
371:                                        CGF.EmitScalarExpr(E->getArg(1)),
372:                                        CGF.EmitScalarExpr(E->getArg(2))})
373:              : CGF.Builder.CreateCall(CGF.CGM.getIntrinsic(IntrinsicID),
374:                                       {CGF.EmitScalarExpr(E->getArg(0)),
375:                                        CGF.EmitScalarExpr(E->getArg(1))});
376: }
377: 
378: static bool EnsureNativeHalfSupport(unsigned BuiltinID, const CallExpr *E,
379:                                     CodeGenFunction &CGF) {
380:   auto &C = CGF.CGM.getContext();
```
- **EN**: This block defines callable entry points like `EnsureNativeHalfSupport`; returns or forwards computed values for the surrounding target builtin lowering logic.
- **CN**: 该代码块定义可调用入口，例如 `EnsureNativeHalfSupport`；为周围的 目标内建函数降级 逻辑返回或转发计算结果。

### Lines 381-400
```cpp
381:   if (!C.getLangOpts().NativeHalfType &&
382:       C.getTargetInfo().useFP16ConversionIntrinsics()) {
383:     CGF.CGM.Error(E->getExprLoc(), C.BuiltinInfo.getQuotedName(BuiltinID) +
384:                                        " requires native half type support.");
385:     return false;
386:   }
387:   return true;
388: }
389: 
390: static Value *MakeHalfType(Function *Intrinsic, unsigned BuiltinID,
391:                            const CallExpr *E, CodeGenFunction &CGF) {
392:   if (!EnsureNativeHalfSupport(BuiltinID, E, CGF))
393:     return nullptr;
394: 
395:   SmallVector<Value *, 16> Args;
396:   auto *FTy = Intrinsic->getFunctionType();
397:   unsigned ICEArguments = 0;
398:   ASTContext::GetBuiltinTypeError Error;
399:   CGF.CGM.getContext().GetBuiltinType(BuiltinID, Error, &ICEArguments);
400:   assert(Error == ASTContext::GE_None && "Should not codegen an error");
```
- **EN**: This block uses control flow (if) to specialize target builtin lowering; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块通过控制流（if）细化 目标内建函数降级 行为；使用断言或不可达标记保护关键不变量。

### Lines 401-420
```cpp
401:   for (unsigned i = 0, e = E->getNumArgs(); i != e; ++i) {
402:     assert((ICEArguments & (1 << i)) == 0);
403:     auto *ArgValue = CGF.EmitScalarExpr(E->getArg(i));
404:     auto *PTy = FTy->getParamType(i);
405:     if (PTy != ArgValue->getType())
406:       ArgValue = CGF.Builder.CreateBitCast(ArgValue, PTy);
407:     Args.push_back(ArgValue);
408:   }
409: 
410:   return CGF.Builder.CreateCall(Intrinsic, Args);
411: }
412: 
413: static Value *MakeHalfType(unsigned IntrinsicID, unsigned BuiltinID,
414:                            const CallExpr *E, CodeGenFunction &CGF) {
415:   return MakeHalfType(CGF.CGM.getIntrinsic(IntrinsicID), BuiltinID, E, CGF);
416: }
417: 
418: static Value *MakeFMAOOB(unsigned IntrinsicID, llvm::Type *Ty,
419:                          const CallExpr *E, CodeGenFunction &CGF) {
420:   return CGF.Builder.CreateCall(CGF.CGM.getIntrinsic(IntrinsicID, {Ty}),
```
- **EN**: This block defines callable entry points like `MakeHalfType`; uses control flow (if, for) to specialize target builtin lowering; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块定义可调用入口，例如 `MakeHalfType`；通过控制流（if, for）细化 目标内建函数降级 行为；使用断言或不可达标记保护关键不变量。

### Lines 421-440
```cpp
421:                                 {CGF.EmitScalarExpr(E->getArg(0)),
422:                                  CGF.EmitScalarExpr(E->getArg(1)),
423:                                  CGF.EmitScalarExpr(E->getArg(2))});
424: }
425: 
426: } // namespace
427: 
428: Value *CodeGenFunction::EmitNVPTXBuiltinExpr(unsigned BuiltinID,
429:                                              const CallExpr *E) {
430:   switch (BuiltinID) {
431:   case NVPTX::BI__nvvm_atom_add_gen_i:
432:   case NVPTX::BI__nvvm_atom_add_gen_l:
433:   case NVPTX::BI__nvvm_atom_add_gen_ll:
434:     return MakeBinaryAtomicValue(*this, llvm::AtomicRMWInst::Add, E,
435:                                  AtomicOrdering::Monotonic);
436: 
437:   case NVPTX::BI__nvvm_atom_sub_gen_i:
438:   case NVPTX::BI__nvvm_atom_sub_gen_l:
439:   case NVPTX::BI__nvvm_atom_sub_gen_ll:
440:     return MakeBinaryAtomicValue(*this, llvm::AtomicRMWInst::Sub, E,
```
- **EN**: This block opens or references namespaces `Value`; defines callable entry points like `MakeBinaryAtomicValue`; uses control flow (switch, case) to specialize target builtin lowering.
- **CN**: 该代码块打开或引用命名空间 `Value`；定义可调用入口，例如 `MakeBinaryAtomicValue`；通过控制流（switch, case）细化 目标内建函数降级 行为。

### Lines 441-460
```cpp
441:                                  AtomicOrdering::Monotonic);
442: 
443:   case NVPTX::BI__nvvm_atom_and_gen_i:
444:   case NVPTX::BI__nvvm_atom_and_gen_l:
445:   case NVPTX::BI__nvvm_atom_and_gen_ll:
446:     return MakeBinaryAtomicValue(*this, llvm::AtomicRMWInst::And, E,
447:                                  AtomicOrdering::Monotonic);
448: 
449:   case NVPTX::BI__nvvm_atom_or_gen_i:
450:   case NVPTX::BI__nvvm_atom_or_gen_l:
451:   case NVPTX::BI__nvvm_atom_or_gen_ll:
452:     return MakeBinaryAtomicValue(*this, llvm::AtomicRMWInst::Or, E,
453:                                  AtomicOrdering::Monotonic);
454: 
455:   case NVPTX::BI__nvvm_atom_xor_gen_i:
456:   case NVPTX::BI__nvvm_atom_xor_gen_l:
457:   case NVPTX::BI__nvvm_atom_xor_gen_ll:
458:     return MakeBinaryAtomicValue(*this, llvm::AtomicRMWInst::Xor, E,
459:                                  AtomicOrdering::Monotonic);
460: 
```
- **EN**: This block spells out callable entry points like `MakeBinaryAtomicValue`; uses control flow (case) to specialize target builtin lowering.
- **CN**: 该代码块给出可调用入口的声明，例如 `MakeBinaryAtomicValue`；通过控制流（case）细化 目标内建函数降级 行为。

### Lines 461-480
```cpp
461:   case NVPTX::BI__nvvm_atom_xchg_gen_i:
462:   case NVPTX::BI__nvvm_atom_xchg_gen_l:
463:   case NVPTX::BI__nvvm_atom_xchg_gen_ll:
464:     return MakeBinaryAtomicValue(*this, llvm::AtomicRMWInst::Xchg, E,
465:                                  AtomicOrdering::Monotonic);
466: 
467:   case NVPTX::BI__nvvm_atom_max_gen_i:
468:   case NVPTX::BI__nvvm_atom_max_gen_l:
469:   case NVPTX::BI__nvvm_atom_max_gen_ll:
470:     return MakeBinaryAtomicValue(*this, llvm::AtomicRMWInst::Max, E,
471:                                  AtomicOrdering::Monotonic);
472: 
473:   case NVPTX::BI__nvvm_atom_max_gen_ui:
474:   case NVPTX::BI__nvvm_atom_max_gen_ul:
475:   case NVPTX::BI__nvvm_atom_max_gen_ull:
476:     return MakeBinaryAtomicValue(*this, llvm::AtomicRMWInst::UMax, E,
477:                                  AtomicOrdering::Monotonic);
478: 
479:   case NVPTX::BI__nvvm_atom_min_gen_i:
480:   case NVPTX::BI__nvvm_atom_min_gen_l:
```
- **EN**: This block spells out callable entry points like `MakeBinaryAtomicValue`; uses control flow (case) to specialize target builtin lowering.
- **CN**: 该代码块给出可调用入口的声明，例如 `MakeBinaryAtomicValue`；通过控制流（case）细化 目标内建函数降级 行为。

### Lines 481-500
```cpp
481:   case NVPTX::BI__nvvm_atom_min_gen_ll:
482:     return MakeBinaryAtomicValue(*this, llvm::AtomicRMWInst::Min, E,
483:                                  AtomicOrdering::Monotonic);
484: 
485:   case NVPTX::BI__nvvm_atom_min_gen_ui:
486:   case NVPTX::BI__nvvm_atom_min_gen_ul:
487:   case NVPTX::BI__nvvm_atom_min_gen_ull:
488:     return MakeBinaryAtomicValue(*this, llvm::AtomicRMWInst::UMin, E,
489:                                  AtomicOrdering::Monotonic);
490: 
491:   case NVPTX::BI__nvvm_atom_cas_gen_us:
492:   case NVPTX::BI__nvvm_atom_cas_gen_i:
493:   case NVPTX::BI__nvvm_atom_cas_gen_l:
494:   case NVPTX::BI__nvvm_atom_cas_gen_ll:
495:     // __nvvm_atom_cas_gen_* should return the old value rather than the
496:     // success flag.
497:     return MakeAtomicCmpXchgValue(*this, E, /*ReturnBool=*/false,
498:                                   AtomicOrdering::Monotonic,
499:                                   AtomicOrdering::Monotonic);
500: 
```
- **EN**: This block spells out callable entry points like `MakeBinaryAtomicValue`, `MakeAtomicCmpXchgValue`; uses control flow (case) to specialize target builtin lowering.
- **CN**: 该代码块给出可调用入口的声明，例如 `MakeBinaryAtomicValue`, `MakeAtomicCmpXchgValue`；通过控制流（case）细化 目标内建函数降级 行为。

### Lines 501-520
```cpp
501:   case NVPTX::BI__nvvm_atom_add_gen_f:
502:   case NVPTX::BI__nvvm_atom_add_gen_d: {
503:     Address DestAddr = EmitPointerWithAlignment(E->getArg(0));
504:     Value *Val = EmitScalarExpr(E->getArg(1));
505: 
506:     return Builder.CreateAtomicRMW(llvm::AtomicRMWInst::FAdd, DestAddr, Val,
507:                                    AtomicOrdering::Monotonic);
508:   }
509: 
510:   case NVPTX::BI__nvvm_atom_inc_gen_ui:
511:     return MakeBinaryAtomicValue(*this, llvm::AtomicRMWInst::UIncWrap, E,
512:                                  AtomicOrdering::Monotonic);
513: 
514:   case NVPTX::BI__nvvm_atom_dec_gen_ui:
515:     return MakeBinaryAtomicValue(*this, llvm::AtomicRMWInst::UDecWrap, E,
516:                                  AtomicOrdering::Monotonic);
517: 
518:   case NVPTX::BI__nvvm_ldg_c:
519:   case NVPTX::BI__nvvm_ldg_sc:
520:   case NVPTX::BI__nvvm_ldg_c2:
```
- **EN**: This block defines callable entry points like `MakeBinaryAtomicValue`; uses control flow (case) to specialize target builtin lowering.
- **CN**: 该代码块定义可调用入口，例如 `MakeBinaryAtomicValue`；通过控制流（case）细化 目标内建函数降级 行为。

### Lines 521-540
```cpp
521:   case NVPTX::BI__nvvm_ldg_sc2:
522:   case NVPTX::BI__nvvm_ldg_c4:
523:   case NVPTX::BI__nvvm_ldg_sc4:
524:   case NVPTX::BI__nvvm_ldg_s:
525:   case NVPTX::BI__nvvm_ldg_s2:
526:   case NVPTX::BI__nvvm_ldg_s4:
527:   case NVPTX::BI__nvvm_ldg_i:
528:   case NVPTX::BI__nvvm_ldg_i2:
529:   case NVPTX::BI__nvvm_ldg_i4:
530:   case NVPTX::BI__nvvm_ldg_l:
531:   case NVPTX::BI__nvvm_ldg_l2:
532:   case NVPTX::BI__nvvm_ldg_ll:
533:   case NVPTX::BI__nvvm_ldg_ll2:
534:   case NVPTX::BI__nvvm_ldg_uc:
535:   case NVPTX::BI__nvvm_ldg_uc2:
536:   case NVPTX::BI__nvvm_ldg_uc4:
537:   case NVPTX::BI__nvvm_ldg_us:
538:   case NVPTX::BI__nvvm_ldg_us2:
539:   case NVPTX::BI__nvvm_ldg_us4:
540:   case NVPTX::BI__nvvm_ldg_ui:
```
- **EN**: This block uses control flow (case) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（case）细化 目标内建函数降级 行为。

### Lines 541-560
```cpp
541:   case NVPTX::BI__nvvm_ldg_ui2:
542:   case NVPTX::BI__nvvm_ldg_ui4:
543:   case NVPTX::BI__nvvm_ldg_ul:
544:   case NVPTX::BI__nvvm_ldg_ul2:
545:   case NVPTX::BI__nvvm_ldg_ull:
546:   case NVPTX::BI__nvvm_ldg_ull2:
547:   case NVPTX::BI__nvvm_ldg_f:
548:   case NVPTX::BI__nvvm_ldg_f2:
549:   case NVPTX::BI__nvvm_ldg_f4:
550:   case NVPTX::BI__nvvm_ldg_d:
551:   case NVPTX::BI__nvvm_ldg_d2:
552:     // PTX Interoperability section 2.2: "For a vector with an even number of
553:     // elements, its alignment is set to number of elements times the alignment
554:     // of its member: n*alignof(t)."
555:     return MakeLdg(*this, E);
556: 
557:   case NVPTX::BI__nvvm_ldu_c:
558:   case NVPTX::BI__nvvm_ldu_sc:
559:   case NVPTX::BI__nvvm_ldu_c2:
560:   case NVPTX::BI__nvvm_ldu_sc2:
```
- **EN**: This block spells out callable entry points like `MakeLdg`; uses control flow (case) to specialize target builtin lowering.
- **CN**: 该代码块给出可调用入口的声明，例如 `MakeLdg`；通过控制流（case）细化 目标内建函数降级 行为。

### Lines 561-580
```cpp
561:   case NVPTX::BI__nvvm_ldu_c4:
562:   case NVPTX::BI__nvvm_ldu_sc4:
563:   case NVPTX::BI__nvvm_ldu_s:
564:   case NVPTX::BI__nvvm_ldu_s2:
565:   case NVPTX::BI__nvvm_ldu_s4:
566:   case NVPTX::BI__nvvm_ldu_i:
567:   case NVPTX::BI__nvvm_ldu_i2:
568:   case NVPTX::BI__nvvm_ldu_i4:
569:   case NVPTX::BI__nvvm_ldu_l:
570:   case NVPTX::BI__nvvm_ldu_l2:
571:   case NVPTX::BI__nvvm_ldu_ll:
572:   case NVPTX::BI__nvvm_ldu_ll2:
573:   case NVPTX::BI__nvvm_ldu_uc:
574:   case NVPTX::BI__nvvm_ldu_uc2:
575:   case NVPTX::BI__nvvm_ldu_uc4:
576:   case NVPTX::BI__nvvm_ldu_us:
577:   case NVPTX::BI__nvvm_ldu_us2:
578:   case NVPTX::BI__nvvm_ldu_us4:
579:   case NVPTX::BI__nvvm_ldu_ui:
580:   case NVPTX::BI__nvvm_ldu_ui2:
```
- **EN**: This block uses control flow (case) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（case）细化 目标内建函数降级 行为。

### Lines 581-600
```cpp
581:   case NVPTX::BI__nvvm_ldu_ui4:
582:   case NVPTX::BI__nvvm_ldu_ul:
583:   case NVPTX::BI__nvvm_ldu_ul2:
584:   case NVPTX::BI__nvvm_ldu_ull:
585:   case NVPTX::BI__nvvm_ldu_ull2:
586:     return MakeLdu(Intrinsic::nvvm_ldu_global_i, *this, E);
587:   case NVPTX::BI__nvvm_ldu_f:
588:   case NVPTX::BI__nvvm_ldu_f2:
589:   case NVPTX::BI__nvvm_ldu_f4:
590:   case NVPTX::BI__nvvm_ldu_d:
591:   case NVPTX::BI__nvvm_ldu_d2:
592:     return MakeLdu(Intrinsic::nvvm_ldu_global_f, *this, E);
593: 
594:   case NVPTX::BI__nvvm_atom_cta_add_gen_i:
595:   case NVPTX::BI__nvvm_atom_cta_add_gen_l:
596:   case NVPTX::BI__nvvm_atom_cta_add_gen_ll:
597:     return MakeScopedAtomic(Intrinsic::nvvm_atomic_add_gen_i_cta, *this, E);
598:   case NVPTX::BI__nvvm_atom_sys_add_gen_i:
599:   case NVPTX::BI__nvvm_atom_sys_add_gen_l:
600:   case NVPTX::BI__nvvm_atom_sys_add_gen_ll:
```
- **EN**: This block spells out callable entry points like `MakeLdu`, `MakeScopedAtomic`; uses control flow (case) to specialize target builtin lowering.
- **CN**: 该代码块给出可调用入口的声明，例如 `MakeLdu`, `MakeScopedAtomic`；通过控制流（case）细化 目标内建函数降级 行为。

### Lines 601-620
```cpp
601:     return MakeScopedAtomic(Intrinsic::nvvm_atomic_add_gen_i_sys, *this, E);
602:   case NVPTX::BI__nvvm_atom_cta_add_gen_f:
603:   case NVPTX::BI__nvvm_atom_cta_add_gen_d:
604:     return MakeScopedAtomic(Intrinsic::nvvm_atomic_add_gen_f_cta, *this, E);
605:   case NVPTX::BI__nvvm_atom_sys_add_gen_f:
606:   case NVPTX::BI__nvvm_atom_sys_add_gen_d:
607:     return MakeScopedAtomic(Intrinsic::nvvm_atomic_add_gen_f_sys, *this, E);
608:   case NVPTX::BI__nvvm_atom_cta_xchg_gen_i:
609:   case NVPTX::BI__nvvm_atom_cta_xchg_gen_l:
610:   case NVPTX::BI__nvvm_atom_cta_xchg_gen_ll:
611:     return MakeScopedAtomic(Intrinsic::nvvm_atomic_exch_gen_i_cta, *this, E);
612:   case NVPTX::BI__nvvm_atom_sys_xchg_gen_i:
613:   case NVPTX::BI__nvvm_atom_sys_xchg_gen_l:
614:   case NVPTX::BI__nvvm_atom_sys_xchg_gen_ll:
615:     return MakeScopedAtomic(Intrinsic::nvvm_atomic_exch_gen_i_sys, *this, E);
616:   case NVPTX::BI__nvvm_atom_cta_max_gen_i:
617:   case NVPTX::BI__nvvm_atom_cta_max_gen_ui:
618:   case NVPTX::BI__nvvm_atom_cta_max_gen_l:
619:   case NVPTX::BI__nvvm_atom_cta_max_gen_ul:
620:   case NVPTX::BI__nvvm_atom_cta_max_gen_ll:
```
- **EN**: This block spells out callable entry points like `MakeScopedAtomic`; uses control flow (case) to specialize target builtin lowering.
- **CN**: 该代码块给出可调用入口的声明，例如 `MakeScopedAtomic`；通过控制流（case）细化 目标内建函数降级 行为。

### Lines 621-640
```cpp
621:   case NVPTX::BI__nvvm_atom_cta_max_gen_ull:
622:     return MakeScopedAtomic(Intrinsic::nvvm_atomic_max_gen_i_cta, *this, E);
623:   case NVPTX::BI__nvvm_atom_sys_max_gen_i:
624:   case NVPTX::BI__nvvm_atom_sys_max_gen_ui:
625:   case NVPTX::BI__nvvm_atom_sys_max_gen_l:
626:   case NVPTX::BI__nvvm_atom_sys_max_gen_ul:
627:   case NVPTX::BI__nvvm_atom_sys_max_gen_ll:
628:   case NVPTX::BI__nvvm_atom_sys_max_gen_ull:
629:     return MakeScopedAtomic(Intrinsic::nvvm_atomic_max_gen_i_sys, *this, E);
630:   case NVPTX::BI__nvvm_atom_cta_min_gen_i:
631:   case NVPTX::BI__nvvm_atom_cta_min_gen_ui:
632:   case NVPTX::BI__nvvm_atom_cta_min_gen_l:
633:   case NVPTX::BI__nvvm_atom_cta_min_gen_ul:
634:   case NVPTX::BI__nvvm_atom_cta_min_gen_ll:
635:   case NVPTX::BI__nvvm_atom_cta_min_gen_ull:
636:     return MakeScopedAtomic(Intrinsic::nvvm_atomic_min_gen_i_cta, *this, E);
637:   case NVPTX::BI__nvvm_atom_sys_min_gen_i:
638:   case NVPTX::BI__nvvm_atom_sys_min_gen_ui:
639:   case NVPTX::BI__nvvm_atom_sys_min_gen_l:
640:   case NVPTX::BI__nvvm_atom_sys_min_gen_ul:
```
- **EN**: This block spells out callable entry points like `MakeScopedAtomic`; uses control flow (case) to specialize target builtin lowering.
- **CN**: 该代码块给出可调用入口的声明，例如 `MakeScopedAtomic`；通过控制流（case）细化 目标内建函数降级 行为。

### Lines 641-660
```cpp
641:   case NVPTX::BI__nvvm_atom_sys_min_gen_ll:
642:   case NVPTX::BI__nvvm_atom_sys_min_gen_ull:
643:     return MakeScopedAtomic(Intrinsic::nvvm_atomic_min_gen_i_sys, *this, E);
644:   case NVPTX::BI__nvvm_atom_cta_inc_gen_ui:
645:     return MakeScopedAtomic(Intrinsic::nvvm_atomic_inc_gen_i_cta, *this, E);
646:   case NVPTX::BI__nvvm_atom_cta_dec_gen_ui:
647:     return MakeScopedAtomic(Intrinsic::nvvm_atomic_dec_gen_i_cta, *this, E);
648:   case NVPTX::BI__nvvm_atom_sys_inc_gen_ui:
649:     return MakeScopedAtomic(Intrinsic::nvvm_atomic_inc_gen_i_sys, *this, E);
650:   case NVPTX::BI__nvvm_atom_sys_dec_gen_ui:
651:     return MakeScopedAtomic(Intrinsic::nvvm_atomic_dec_gen_i_sys, *this, E);
652:   case NVPTX::BI__nvvm_atom_cta_and_gen_i:
653:   case NVPTX::BI__nvvm_atom_cta_and_gen_l:
654:   case NVPTX::BI__nvvm_atom_cta_and_gen_ll:
655:     return MakeScopedAtomic(Intrinsic::nvvm_atomic_and_gen_i_cta, *this, E);
656:   case NVPTX::BI__nvvm_atom_sys_and_gen_i:
657:   case NVPTX::BI__nvvm_atom_sys_and_gen_l:
658:   case NVPTX::BI__nvvm_atom_sys_and_gen_ll:
659:     return MakeScopedAtomic(Intrinsic::nvvm_atomic_and_gen_i_sys, *this, E);
660:   case NVPTX::BI__nvvm_atom_cta_or_gen_i:
```
- **EN**: This block spells out callable entry points like `MakeScopedAtomic`; uses control flow (case) to specialize target builtin lowering.
- **CN**: 该代码块给出可调用入口的声明，例如 `MakeScopedAtomic`；通过控制流（case）细化 目标内建函数降级 行为。

### Lines 661-680
```cpp
661:   case NVPTX::BI__nvvm_atom_cta_or_gen_l:
662:   case NVPTX::BI__nvvm_atom_cta_or_gen_ll:
663:     return MakeScopedAtomic(Intrinsic::nvvm_atomic_or_gen_i_cta, *this, E);
664:   case NVPTX::BI__nvvm_atom_sys_or_gen_i:
665:   case NVPTX::BI__nvvm_atom_sys_or_gen_l:
666:   case NVPTX::BI__nvvm_atom_sys_or_gen_ll:
667:     return MakeScopedAtomic(Intrinsic::nvvm_atomic_or_gen_i_sys, *this, E);
668:   case NVPTX::BI__nvvm_atom_cta_xor_gen_i:
669:   case NVPTX::BI__nvvm_atom_cta_xor_gen_l:
670:   case NVPTX::BI__nvvm_atom_cta_xor_gen_ll:
671:     return MakeScopedAtomic(Intrinsic::nvvm_atomic_xor_gen_i_cta, *this, E);
672:   case NVPTX::BI__nvvm_atom_sys_xor_gen_i:
673:   case NVPTX::BI__nvvm_atom_sys_xor_gen_l:
674:   case NVPTX::BI__nvvm_atom_sys_xor_gen_ll:
675:     return MakeScopedAtomic(Intrinsic::nvvm_atomic_xor_gen_i_sys, *this, E);
676:   case NVPTX::BI__nvvm_atom_cta_cas_gen_us:
677:   case NVPTX::BI__nvvm_atom_cta_cas_gen_i:
678:   case NVPTX::BI__nvvm_atom_cta_cas_gen_l:
679:   case NVPTX::BI__nvvm_atom_cta_cas_gen_ll: {
680:     Value *Ptr = EmitScalarExpr(E->getArg(0));
```
- **EN**: This block defines callable entry points like `MakeScopedAtomic`; uses control flow (case) to specialize target builtin lowering.
- **CN**: 该代码块定义可调用入口，例如 `MakeScopedAtomic`；通过控制流（case）细化 目标内建函数降级 行为。

### Lines 681-700
```cpp
681:     llvm::Type *ElemTy =
682:         ConvertTypeForMem(E->getArg(0)->getType()->getPointeeType());
683:     return Builder.CreateCall(
684:         CGM.getIntrinsic(
685:             Intrinsic::nvvm_atomic_cas_gen_i_cta, {ElemTy, Ptr->getType()}),
686:         {Ptr, EmitScalarExpr(E->getArg(1)), EmitScalarExpr(E->getArg(2))});
687:   }
688:   case NVPTX::BI__nvvm_atom_sys_cas_gen_us:
689:   case NVPTX::BI__nvvm_atom_sys_cas_gen_i:
690:   case NVPTX::BI__nvvm_atom_sys_cas_gen_l:
691:   case NVPTX::BI__nvvm_atom_sys_cas_gen_ll: {
692:     Value *Ptr = EmitScalarExpr(E->getArg(0));
693:     llvm::Type *ElemTy =
694:         ConvertTypeForMem(E->getArg(0)->getType()->getPointeeType());
695:     return Builder.CreateCall(
696:         CGM.getIntrinsic(
697:             Intrinsic::nvvm_atomic_cas_gen_i_sys, {ElemTy, Ptr->getType()}),
698:         {Ptr, EmitScalarExpr(E->getArg(1)), EmitScalarExpr(E->getArg(2))});
699:   }
700:   case NVPTX::BI__nvvm_match_all_sync_i32p:
```
- **EN**: This block defines callable entry points like `ConvertTypeForMem`; uses control flow (case) to specialize target builtin lowering.
- **CN**: 该代码块定义可调用入口，例如 `ConvertTypeForMem`；通过控制流（case）细化 目标内建函数降级 行为。

### Lines 701-720
```cpp
701:   case NVPTX::BI__nvvm_match_all_sync_i64p: {
702:     Value *Mask = EmitScalarExpr(E->getArg(0));
703:     Value *Val = EmitScalarExpr(E->getArg(1));
704:     Address PredOutPtr = EmitPointerWithAlignment(E->getArg(2));
705:     Value *ResultPair = Builder.CreateCall(
706:         CGM.getIntrinsic(BuiltinID == NVPTX::BI__nvvm_match_all_sync_i32p
707:                              ? Intrinsic::nvvm_match_all_sync_i32p
708:                              : Intrinsic::nvvm_match_all_sync_i64p),
709:         {Mask, Val});
710:     Value *Pred = Builder.CreateZExt(Builder.CreateExtractValue(ResultPair, 1),
711:                                      PredOutPtr.getElementType());
712:     Builder.CreateStore(Pred, PredOutPtr);
713:     return Builder.CreateExtractValue(ResultPair, 0);
714:   }
715: 
716:   // FP MMA loads
717:   case NVPTX::BI__hmma_m16n16k16_ld_a:
718:   case NVPTX::BI__hmma_m16n16k16_ld_b:
719:   case NVPTX::BI__hmma_m16n16k16_ld_c_f16:
720:   case NVPTX::BI__hmma_m16n16k16_ld_c_f32:
```
- **EN**: This block uses control flow (case) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（case）细化 目标内建函数降级 行为。

### Lines 721-740
```cpp
721:   case NVPTX::BI__hmma_m32n8k16_ld_a:
722:   case NVPTX::BI__hmma_m32n8k16_ld_b:
723:   case NVPTX::BI__hmma_m32n8k16_ld_c_f16:
724:   case NVPTX::BI__hmma_m32n8k16_ld_c_f32:
725:   case NVPTX::BI__hmma_m8n32k16_ld_a:
726:   case NVPTX::BI__hmma_m8n32k16_ld_b:
727:   case NVPTX::BI__hmma_m8n32k16_ld_c_f16:
728:   case NVPTX::BI__hmma_m8n32k16_ld_c_f32:
729:   // Integer MMA loads.
730:   case NVPTX::BI__imma_m16n16k16_ld_a_s8:
731:   case NVPTX::BI__imma_m16n16k16_ld_a_u8:
732:   case NVPTX::BI__imma_m16n16k16_ld_b_s8:
733:   case NVPTX::BI__imma_m16n16k16_ld_b_u8:
734:   case NVPTX::BI__imma_m16n16k16_ld_c:
735:   case NVPTX::BI__imma_m32n8k16_ld_a_s8:
736:   case NVPTX::BI__imma_m32n8k16_ld_a_u8:
737:   case NVPTX::BI__imma_m32n8k16_ld_b_s8:
738:   case NVPTX::BI__imma_m32n8k16_ld_b_u8:
739:   case NVPTX::BI__imma_m32n8k16_ld_c:
740:   case NVPTX::BI__imma_m8n32k16_ld_a_s8:
```
- **EN**: This block uses control flow (case) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（case）细化 目标内建函数降级 行为。

### Lines 741-760
```cpp
741:   case NVPTX::BI__imma_m8n32k16_ld_a_u8:
742:   case NVPTX::BI__imma_m8n32k16_ld_b_s8:
743:   case NVPTX::BI__imma_m8n32k16_ld_b_u8:
744:   case NVPTX::BI__imma_m8n32k16_ld_c:
745:   // Sub-integer MMA loads.
746:   case NVPTX::BI__imma_m8n8k32_ld_a_s4:
747:   case NVPTX::BI__imma_m8n8k32_ld_a_u4:
748:   case NVPTX::BI__imma_m8n8k32_ld_b_s4:
749:   case NVPTX::BI__imma_m8n8k32_ld_b_u4:
750:   case NVPTX::BI__imma_m8n8k32_ld_c:
751:   case NVPTX::BI__bmma_m8n8k128_ld_a_b1:
752:   case NVPTX::BI__bmma_m8n8k128_ld_b_b1:
753:   case NVPTX::BI__bmma_m8n8k128_ld_c:
754:   // Double MMA loads.
755:   case NVPTX::BI__dmma_m8n8k4_ld_a:
756:   case NVPTX::BI__dmma_m8n8k4_ld_b:
757:   case NVPTX::BI__dmma_m8n8k4_ld_c:
758:   // Alternate float MMA loads.
759:   case NVPTX::BI__mma_bf16_m16n16k16_ld_a:
760:   case NVPTX::BI__mma_bf16_m16n16k16_ld_b:
```
- **EN**: This block uses control flow (case) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（case）细化 目标内建函数降级 行为。

### Lines 761-780
```cpp
761:   case NVPTX::BI__mma_bf16_m8n32k16_ld_a:
762:   case NVPTX::BI__mma_bf16_m8n32k16_ld_b:
763:   case NVPTX::BI__mma_bf16_m32n8k16_ld_a:
764:   case NVPTX::BI__mma_bf16_m32n8k16_ld_b:
765:   case NVPTX::BI__mma_tf32_m16n16k8_ld_a:
766:   case NVPTX::BI__mma_tf32_m16n16k8_ld_b:
767:   case NVPTX::BI__mma_tf32_m16n16k8_ld_c: {
768:     Address Dst = EmitPointerWithAlignment(E->getArg(0));
769:     Value *Src = EmitScalarExpr(E->getArg(1));
770:     Value *Ldm = EmitScalarExpr(E->getArg(2));
771:     std::optional<llvm::APSInt> isColMajorArg =
772:         E->getArg(3)->getIntegerConstantExpr(getContext());
773:     if (!isColMajorArg)
774:       return nullptr;
775:     bool isColMajor = isColMajorArg->getSExtValue();
776:     NVPTXMmaLdstInfo II = getNVPTXMmaLdstInfo(BuiltinID);
777:     unsigned IID = isColMajor ? II.IID_col : II.IID_row;
778:     if (IID == 0)
779:       return nullptr;
780: 
```
- **EN**: This block uses control flow (if, case) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（if, case）细化 目标内建函数降级 行为。

### Lines 781-800
```cpp
781:     Value *Result =
782:         Builder.CreateCall(CGM.getIntrinsic(IID, Src->getType()), {Src, Ldm});
783: 
784:     // Save returned values.
785:     assert(II.NumResults);
786:     if (II.NumResults == 1) {
787:       Builder.CreateAlignedStore(Result, Dst.emitRawPointer(*this),
788:                                  CharUnits::fromQuantity(4));
789:     } else {
790:       for (unsigned i = 0; i < II.NumResults; ++i) {
791:         Builder.CreateAlignedStore(
792:             Builder.CreateBitCast(Builder.CreateExtractValue(Result, i),
793:                                   Dst.getElementType()),
794:             Builder.CreateGEP(Dst.getElementType(), Dst.emitRawPointer(*this),
795:                               llvm::ConstantInt::get(IntTy, i)),
796:             CharUnits::fromQuantity(4));
797:       }
798:     }
799:     return Result;
800:   }
```
- **EN**: This block defines callable entry points like `fromQuantity`, `get`; uses control flow (if, for) to specialize target builtin lowering; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块定义可调用入口，例如 `fromQuantity`, `get`；通过控制流（if, for）细化 目标内建函数降级 行为；使用断言或不可达标记保护关键不变量。

### Lines 801-820
```cpp
801: 
802:   case NVPTX::BI__hmma_m16n16k16_st_c_f16:
803:   case NVPTX::BI__hmma_m16n16k16_st_c_f32:
804:   case NVPTX::BI__hmma_m32n8k16_st_c_f16:
805:   case NVPTX::BI__hmma_m32n8k16_st_c_f32:
806:   case NVPTX::BI__hmma_m8n32k16_st_c_f16:
807:   case NVPTX::BI__hmma_m8n32k16_st_c_f32:
808:   case NVPTX::BI__imma_m16n16k16_st_c_i32:
809:   case NVPTX::BI__imma_m32n8k16_st_c_i32:
810:   case NVPTX::BI__imma_m8n32k16_st_c_i32:
811:   case NVPTX::BI__imma_m8n8k32_st_c_i32:
812:   case NVPTX::BI__bmma_m8n8k128_st_c_i32:
813:   case NVPTX::BI__dmma_m8n8k4_st_c_f64:
814:   case NVPTX::BI__mma_m16n16k8_st_c_f32: {
815:     Value *Dst = EmitScalarExpr(E->getArg(0));
816:     Address Src = EmitPointerWithAlignment(E->getArg(1));
817:     Value *Ldm = EmitScalarExpr(E->getArg(2));
818:     std::optional<llvm::APSInt> isColMajorArg =
819:         E->getArg(3)->getIntegerConstantExpr(getContext());
820:     if (!isColMajorArg)
```
- **EN**: This block uses control flow (if, case) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（if, case）细化 目标内建函数降级 行为。

### Lines 821-840
```cpp
821:       return nullptr;
822:     bool isColMajor = isColMajorArg->getSExtValue();
823:     NVPTXMmaLdstInfo II = getNVPTXMmaLdstInfo(BuiltinID);
824:     unsigned IID = isColMajor ? II.IID_col : II.IID_row;
825:     if (IID == 0)
826:       return nullptr;
827:     Function *Intrinsic =
828:         CGM.getIntrinsic(IID, Dst->getType());
829:     llvm::Type *ParamType = Intrinsic->getFunctionType()->getParamType(1);
830:     SmallVector<Value *, 10> Values = {Dst};
831:     for (unsigned i = 0; i < II.NumResults; ++i) {
832:       Value *V = Builder.CreateAlignedLoad(
833:           Src.getElementType(),
834:           Builder.CreateGEP(Src.getElementType(), Src.emitRawPointer(*this),
835:                             llvm::ConstantInt::get(IntTy, i)),
836:           CharUnits::fromQuantity(4));
837:       Values.push_back(Builder.CreateBitCast(V, ParamType));
838:     }
839:     Values.push_back(Ldm);
840:     Value *Result = Builder.CreateCall(Intrinsic, Values);
```
- **EN**: This block defines callable entry points like `get`; uses control flow (if, for) to specialize target builtin lowering.
- **CN**: 该代码块定义可调用入口，例如 `get`；通过控制流（if, for）细化 目标内建函数降级 行为。

### Lines 841-860
```cpp
841:     return Result;
842:   }
843: 
844:   // BI__hmma_m16n16k16_mma_<Dtype><CType>(d, a, b, c, layout, satf) -->
845:   // Intrinsic::nvvm_wmma_m16n16k16_mma_sync<layout A,B><DType><CType><Satf>
846:   case NVPTX::BI__hmma_m16n16k16_mma_f16f16:
847:   case NVPTX::BI__hmma_m16n16k16_mma_f32f16:
848:   case NVPTX::BI__hmma_m16n16k16_mma_f32f32:
849:   case NVPTX::BI__hmma_m16n16k16_mma_f16f32:
850:   case NVPTX::BI__hmma_m32n8k16_mma_f16f16:
851:   case NVPTX::BI__hmma_m32n8k16_mma_f32f16:
852:   case NVPTX::BI__hmma_m32n8k16_mma_f32f32:
853:   case NVPTX::BI__hmma_m32n8k16_mma_f16f32:
854:   case NVPTX::BI__hmma_m8n32k16_mma_f16f16:
855:   case NVPTX::BI__hmma_m8n32k16_mma_f32f16:
856:   case NVPTX::BI__hmma_m8n32k16_mma_f32f32:
857:   case NVPTX::BI__hmma_m8n32k16_mma_f16f32:
858:   case NVPTX::BI__imma_m16n16k16_mma_s8:
859:   case NVPTX::BI__imma_m16n16k16_mma_u8:
860:   case NVPTX::BI__imma_m32n8k16_mma_s8:
```
- **EN**: This block uses control flow (case) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（case）细化 目标内建函数降级 行为。

### Lines 861-880
```cpp
861:   case NVPTX::BI__imma_m32n8k16_mma_u8:
862:   case NVPTX::BI__imma_m8n32k16_mma_s8:
863:   case NVPTX::BI__imma_m8n32k16_mma_u8:
864:   case NVPTX::BI__imma_m8n8k32_mma_s4:
865:   case NVPTX::BI__imma_m8n8k32_mma_u4:
866:   case NVPTX::BI__bmma_m8n8k128_mma_xor_popc_b1:
867:   case NVPTX::BI__bmma_m8n8k128_mma_and_popc_b1:
868:   case NVPTX::BI__dmma_m8n8k4_mma_f64:
869:   case NVPTX::BI__mma_bf16_m16n16k16_mma_f32:
870:   case NVPTX::BI__mma_bf16_m8n32k16_mma_f32:
871:   case NVPTX::BI__mma_bf16_m32n8k16_mma_f32:
872:   case NVPTX::BI__mma_tf32_m16n16k8_mma_f32: {
873:     Address Dst = EmitPointerWithAlignment(E->getArg(0));
874:     Address SrcA = EmitPointerWithAlignment(E->getArg(1));
875:     Address SrcB = EmitPointerWithAlignment(E->getArg(2));
876:     Address SrcC = EmitPointerWithAlignment(E->getArg(3));
877:     std::optional<llvm::APSInt> LayoutArg =
878:         E->getArg(4)->getIntegerConstantExpr(getContext());
879:     if (!LayoutArg)
880:       return nullptr;
```
- **EN**: This block uses control flow (if, case) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（if, case）细化 目标内建函数降级 行为。

### Lines 881-900
```cpp
881:     int Layout = LayoutArg->getSExtValue();
882:     if (Layout < 0 || Layout > 3)
883:       return nullptr;
884:     llvm::APSInt SatfArg;
885:     if (BuiltinID == NVPTX::BI__bmma_m8n8k128_mma_xor_popc_b1 ||
886:         BuiltinID == NVPTX::BI__bmma_m8n8k128_mma_and_popc_b1)
887:       SatfArg = 0;  // .b1 does not have satf argument.
888:     else if (std::optional<llvm::APSInt> OptSatfArg =
889:                  E->getArg(5)->getIntegerConstantExpr(getContext()))
890:       SatfArg = *OptSatfArg;
891:     else
892:       return nullptr;
893:     bool Satf = SatfArg.getSExtValue();
894:     NVPTXMmaInfo MI = getNVPTXMmaInfo(BuiltinID);
895:     unsigned IID = MI.getMMAIntrinsic(Layout, Satf);
896:     if (IID == 0)  // Unsupported combination of Layout/Satf.
897:       return nullptr;
898: 
899:     SmallVector<Value *, 24> Values;
900:     Function *Intrinsic = CGM.getIntrinsic(IID);
```
- **EN**: This block uses control flow (if) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（if）细化 目标内建函数降级 行为。

### Lines 901-920
```cpp
901:     llvm::Type *AType = Intrinsic->getFunctionType()->getParamType(0);
902:     // Load A
903:     for (unsigned i = 0; i < MI.NumEltsA; ++i) {
904:       Value *V = Builder.CreateAlignedLoad(
905:           SrcA.getElementType(),
906:           Builder.CreateGEP(SrcA.getElementType(), SrcA.emitRawPointer(*this),
907:                             llvm::ConstantInt::get(IntTy, i)),
908:           CharUnits::fromQuantity(4));
909:       Values.push_back(Builder.CreateBitCast(V, AType));
910:     }
911:     // Load B
912:     llvm::Type *BType = Intrinsic->getFunctionType()->getParamType(MI.NumEltsA);
913:     for (unsigned i = 0; i < MI.NumEltsB; ++i) {
914:       Value *V = Builder.CreateAlignedLoad(
915:           SrcB.getElementType(),
916:           Builder.CreateGEP(SrcB.getElementType(), SrcB.emitRawPointer(*this),
917:                             llvm::ConstantInt::get(IntTy, i)),
918:           CharUnits::fromQuantity(4));
919:       Values.push_back(Builder.CreateBitCast(V, BType));
920:     }
```
- **EN**: This block defines callable entry points like `get`; uses control flow (for) to specialize target builtin lowering.
- **CN**: 该代码块定义可调用入口，例如 `get`；通过控制流（for）细化 目标内建函数降级 行为。

### Lines 921-940
```cpp
921:     // Load C
922:     llvm::Type *CType =
923:         Intrinsic->getFunctionType()->getParamType(MI.NumEltsA + MI.NumEltsB);
924:     for (unsigned i = 0; i < MI.NumEltsC; ++i) {
925:       Value *V = Builder.CreateAlignedLoad(
926:           SrcC.getElementType(),
927:           Builder.CreateGEP(SrcC.getElementType(), SrcC.emitRawPointer(*this),
928:                             llvm::ConstantInt::get(IntTy, i)),
929:           CharUnits::fromQuantity(4));
930:       Values.push_back(Builder.CreateBitCast(V, CType));
931:     }
932:     Value *Result = Builder.CreateCall(Intrinsic, Values);
933:     llvm::Type *DType = Dst.getElementType();
934:     for (unsigned i = 0; i < MI.NumEltsD; ++i)
935:       Builder.CreateAlignedStore(
936:           Builder.CreateBitCast(Builder.CreateExtractValue(Result, i), DType),
937:           Builder.CreateGEP(Dst.getElementType(), Dst.emitRawPointer(*this),
938:                             llvm::ConstantInt::get(IntTy, i)),
939:           CharUnits::fromQuantity(4));
940:     return Result;
```
- **EN**: This block defines callable entry points like `get`; uses control flow (for) to specialize target builtin lowering.
- **CN**: 该代码块定义可调用入口，例如 `get`；通过控制流（for）细化 目标内建函数降级 行为。

### Lines 941-960
```cpp
941:   }
942:   // The following builtins require half type support
943:   case NVPTX::BI__nvvm_ex2_approx_f16:
944:     return MakeHalfType(
945:         CGM.getIntrinsic(Intrinsic::nvvm_ex2_approx, Builder.getHalfTy()),
946:         BuiltinID, E, *this);
947:   case NVPTX::BI__nvvm_ex2_approx_f16x2:
948:     return MakeHalfType(
949:         CGM.getIntrinsic(Intrinsic::nvvm_ex2_approx,
950:                          FixedVectorType::get(Builder.getHalfTy(), 2)),
951:         BuiltinID, E, *this);
952:   case NVPTX::BI__nvvm_ff2f16x2_rn:
953:     return MakeHalfType(Intrinsic::nvvm_ff2f16x2_rn, BuiltinID, E, *this);
954:   case NVPTX::BI__nvvm_ff2f16x2_rn_relu:
955:     return MakeHalfType(Intrinsic::nvvm_ff2f16x2_rn_relu, BuiltinID, E, *this);
956:   case NVPTX::BI__nvvm_ff2f16x2_rz:
957:     return MakeHalfType(Intrinsic::nvvm_ff2f16x2_rz, BuiltinID, E, *this);
958:   case NVPTX::BI__nvvm_ff2f16x2_rz_relu:
959:     return MakeHalfType(Intrinsic::nvvm_ff2f16x2_rz_relu, BuiltinID, E, *this);
960:   case NVPTX::BI__nvvm_fma_rn_f16:
```
- **EN**: This block spells out callable entry points like `MakeHalfType`; uses control flow (case) to specialize target builtin lowering.
- **CN**: 该代码块给出可调用入口的声明，例如 `MakeHalfType`；通过控制流（case）细化 目标内建函数降级 行为。

### Lines 961-980
```cpp
961:     return MakeHalfType(Intrinsic::nvvm_fma_rn_f16, BuiltinID, E, *this);
962:   case NVPTX::BI__nvvm_fma_rn_f16x2:
963:     return MakeHalfType(Intrinsic::nvvm_fma_rn_f16x2, BuiltinID, E, *this);
964:   case NVPTX::BI__nvvm_fma_rn_ftz_f16:
965:     return MakeHalfType(Intrinsic::nvvm_fma_rn_ftz_f16, BuiltinID, E, *this);
966:   case NVPTX::BI__nvvm_fma_rn_ftz_f16x2:
967:     return MakeHalfType(Intrinsic::nvvm_fma_rn_ftz_f16x2, BuiltinID, E, *this);
968:   case NVPTX::BI__nvvm_fma_rn_ftz_relu_f16:
969:     return MakeHalfType(Intrinsic::nvvm_fma_rn_ftz_relu_f16, BuiltinID, E,
970:                         *this);
971:   case NVPTX::BI__nvvm_fma_rn_ftz_relu_f16x2:
972:     return MakeHalfType(Intrinsic::nvvm_fma_rn_ftz_relu_f16x2, BuiltinID, E,
973:                         *this);
974:   case NVPTX::BI__nvvm_fma_rn_ftz_sat_f16:
975:     return MakeHalfType(Intrinsic::nvvm_fma_rn_ftz_sat_f16, BuiltinID, E,
976:                         *this);
977:   case NVPTX::BI__nvvm_fma_rn_ftz_sat_f16x2:
978:     return MakeHalfType(Intrinsic::nvvm_fma_rn_ftz_sat_f16x2, BuiltinID, E,
979:                         *this);
980:   case NVPTX::BI__nvvm_fma_rn_relu_f16:
```
- **EN**: This block spells out callable entry points like `MakeHalfType`; uses control flow (case) to specialize target builtin lowering.
- **CN**: 该代码块给出可调用入口的声明，例如 `MakeHalfType`；通过控制流（case）细化 目标内建函数降级 行为。

### Lines 981-1000
```cpp
 981:     return MakeHalfType(Intrinsic::nvvm_fma_rn_relu_f16, BuiltinID, E, *this);
 982:   case NVPTX::BI__nvvm_fma_rn_relu_f16x2:
 983:     return MakeHalfType(Intrinsic::nvvm_fma_rn_relu_f16x2, BuiltinID, E, *this);
 984:   case NVPTX::BI__nvvm_fma_rn_sat_f16:
 985:     return MakeHalfType(Intrinsic::nvvm_fma_rn_sat_f16, BuiltinID, E, *this);
 986:   case NVPTX::BI__nvvm_fma_rn_sat_f16x2:
 987:     return MakeHalfType(Intrinsic::nvvm_fma_rn_sat_f16x2, BuiltinID, E, *this);
 988:   case NVPTX::BI__nvvm_fma_rn_oob_f16:
 989:     return MakeFMAOOB(Intrinsic::nvvm_fma_rn_oob, Builder.getHalfTy(), E,
 990:                       *this);
 991:   case NVPTX::BI__nvvm_fma_rn_oob_f16x2:
 992:     return MakeFMAOOB(Intrinsic::nvvm_fma_rn_oob,
 993:                       llvm::FixedVectorType::get(Builder.getHalfTy(), 2), E,
 994:                       *this);
 995:   case NVPTX::BI__nvvm_fma_rn_oob_bf16:
 996:     return MakeFMAOOB(Intrinsic::nvvm_fma_rn_oob, Builder.getBFloatTy(), E,
 997:                       *this);
 998:   case NVPTX::BI__nvvm_fma_rn_oob_bf16x2:
 999:     return MakeFMAOOB(Intrinsic::nvvm_fma_rn_oob,
1000:                       llvm::FixedVectorType::get(Builder.getBFloatTy(), 2), E,
```
- **EN**: This block spells out callable entry points like `MakeHalfType`; uses control flow (case) to specialize target builtin lowering.
- **CN**: 该代码块给出可调用入口的声明，例如 `MakeHalfType`；通过控制流（case）细化 目标内建函数降级 行为。

### Lines 1001-1020
```cpp
1001:                       *this);
1002:   case NVPTX::BI__nvvm_fma_rn_oob_relu_f16:
1003:     return MakeFMAOOB(Intrinsic::nvvm_fma_rn_oob_relu, Builder.getHalfTy(), E,
1004:                       *this);
1005:   case NVPTX::BI__nvvm_fma_rn_oob_relu_f16x2:
1006:     return MakeFMAOOB(Intrinsic::nvvm_fma_rn_oob_relu,
1007:                       llvm::FixedVectorType::get(Builder.getHalfTy(), 2), E,
1008:                       *this);
1009:   case NVPTX::BI__nvvm_fma_rn_oob_relu_bf16:
1010:     return MakeFMAOOB(Intrinsic::nvvm_fma_rn_oob_relu, Builder.getBFloatTy(), E,
1011:                       *this);
1012:   case NVPTX::BI__nvvm_fma_rn_oob_relu_bf16x2:
1013:     return MakeFMAOOB(Intrinsic::nvvm_fma_rn_oob_relu,
1014:                       llvm::FixedVectorType::get(Builder.getBFloatTy(), 2), E,
1015:                       *this);
1016:   case NVPTX::BI__nvvm_fmax_f16:
1017:     return MakeHalfType(Intrinsic::nvvm_fmax_f16, BuiltinID, E, *this);
1018:   case NVPTX::BI__nvvm_fmax_f16x2:
1019:     return MakeHalfType(Intrinsic::nvvm_fmax_f16x2, BuiltinID, E, *this);
1020:   case NVPTX::BI__nvvm_fmax_ftz_f16:
```
- **EN**: This block spells out callable entry points like `MakeFMAOOB`, `MakeHalfType`; uses control flow (case) to specialize target builtin lowering.
- **CN**: 该代码块给出可调用入口的声明，例如 `MakeFMAOOB`, `MakeHalfType`；通过控制流（case）细化 目标内建函数降级 行为。

### Lines 1021-1040
```cpp
1021:     return MakeHalfType(Intrinsic::nvvm_fmax_ftz_f16, BuiltinID, E, *this);
1022:   case NVPTX::BI__nvvm_fmax_ftz_f16x2:
1023:     return MakeHalfType(Intrinsic::nvvm_fmax_ftz_f16x2, BuiltinID, E, *this);
1024:   case NVPTX::BI__nvvm_fmax_ftz_nan_f16:
1025:     return MakeHalfType(Intrinsic::nvvm_fmax_ftz_nan_f16, BuiltinID, E, *this);
1026:   case NVPTX::BI__nvvm_fmax_ftz_nan_f16x2:
1027:     return MakeHalfType(Intrinsic::nvvm_fmax_ftz_nan_f16x2, BuiltinID, E,
1028:                         *this);
1029:   case NVPTX::BI__nvvm_fmax_ftz_nan_xorsign_abs_f16:
1030:     return MakeHalfType(Intrinsic::nvvm_fmax_ftz_nan_xorsign_abs_f16, BuiltinID,
1031:                         E, *this);
1032:   case NVPTX::BI__nvvm_fmax_ftz_nan_xorsign_abs_f16x2:
1033:     return MakeHalfType(Intrinsic::nvvm_fmax_ftz_nan_xorsign_abs_f16x2,
1034:                         BuiltinID, E, *this);
1035:   case NVPTX::BI__nvvm_fmax_ftz_xorsign_abs_f16:
1036:     return MakeHalfType(Intrinsic::nvvm_fmax_ftz_xorsign_abs_f16, BuiltinID, E,
1037:                         *this);
1038:   case NVPTX::BI__nvvm_fmax_ftz_xorsign_abs_f16x2:
1039:     return MakeHalfType(Intrinsic::nvvm_fmax_ftz_xorsign_abs_f16x2, BuiltinID,
1040:                         E, *this);
```
- **EN**: This block spells out callable entry points like `MakeHalfType`; uses control flow (case) to specialize target builtin lowering.
- **CN**: 该代码块给出可调用入口的声明，例如 `MakeHalfType`；通过控制流（case）细化 目标内建函数降级 行为。

### Lines 1041-1060
```cpp
1041:   case NVPTX::BI__nvvm_fmax_nan_f16:
1042:     return MakeHalfType(Intrinsic::nvvm_fmax_nan_f16, BuiltinID, E, *this);
1043:   case NVPTX::BI__nvvm_fmax_nan_f16x2:
1044:     return MakeHalfType(Intrinsic::nvvm_fmax_nan_f16x2, BuiltinID, E, *this);
1045:   case NVPTX::BI__nvvm_fmax_nan_xorsign_abs_f16:
1046:     return MakeHalfType(Intrinsic::nvvm_fmax_nan_xorsign_abs_f16, BuiltinID, E,
1047:                         *this);
1048:   case NVPTX::BI__nvvm_fmax_nan_xorsign_abs_f16x2:
1049:     return MakeHalfType(Intrinsic::nvvm_fmax_nan_xorsign_abs_f16x2, BuiltinID,
1050:                         E, *this);
1051:   case NVPTX::BI__nvvm_fmax_xorsign_abs_f16:
1052:     return MakeHalfType(Intrinsic::nvvm_fmax_xorsign_abs_f16, BuiltinID, E,
1053:                         *this);
1054:   case NVPTX::BI__nvvm_fmax_xorsign_abs_f16x2:
1055:     return MakeHalfType(Intrinsic::nvvm_fmax_xorsign_abs_f16x2, BuiltinID, E,
1056:                         *this);
1057:   case NVPTX::BI__nvvm_fmin_f16:
1058:     return MakeHalfType(Intrinsic::nvvm_fmin_f16, BuiltinID, E, *this);
1059:   case NVPTX::BI__nvvm_fmin_f16x2:
1060:     return MakeHalfType(Intrinsic::nvvm_fmin_f16x2, BuiltinID, E, *this);
```
- **EN**: This block spells out callable entry points like `MakeHalfType`; uses control flow (case) to specialize target builtin lowering.
- **CN**: 该代码块给出可调用入口的声明，例如 `MakeHalfType`；通过控制流（case）细化 目标内建函数降级 行为。

### Lines 1061-1080
```cpp
1061:   case NVPTX::BI__nvvm_fmin_ftz_f16:
1062:     return MakeHalfType(Intrinsic::nvvm_fmin_ftz_f16, BuiltinID, E, *this);
1063:   case NVPTX::BI__nvvm_fmin_ftz_f16x2:
1064:     return MakeHalfType(Intrinsic::nvvm_fmin_ftz_f16x2, BuiltinID, E, *this);
1065:   case NVPTX::BI__nvvm_fmin_ftz_nan_f16:
1066:     return MakeHalfType(Intrinsic::nvvm_fmin_ftz_nan_f16, BuiltinID, E, *this);
1067:   case NVPTX::BI__nvvm_fmin_ftz_nan_f16x2:
1068:     return MakeHalfType(Intrinsic::nvvm_fmin_ftz_nan_f16x2, BuiltinID, E,
1069:                         *this);
1070:   case NVPTX::BI__nvvm_fmin_ftz_nan_xorsign_abs_f16:
1071:     return MakeHalfType(Intrinsic::nvvm_fmin_ftz_nan_xorsign_abs_f16, BuiltinID,
1072:                         E, *this);
1073:   case NVPTX::BI__nvvm_fmin_ftz_nan_xorsign_abs_f16x2:
1074:     return MakeHalfType(Intrinsic::nvvm_fmin_ftz_nan_xorsign_abs_f16x2,
1075:                         BuiltinID, E, *this);
1076:   case NVPTX::BI__nvvm_fmin_ftz_xorsign_abs_f16:
1077:     return MakeHalfType(Intrinsic::nvvm_fmin_ftz_xorsign_abs_f16, BuiltinID, E,
1078:                         *this);
1079:   case NVPTX::BI__nvvm_fmin_ftz_xorsign_abs_f16x2:
1080:     return MakeHalfType(Intrinsic::nvvm_fmin_ftz_xorsign_abs_f16x2, BuiltinID,
```
- **EN**: This block spells out callable entry points like `MakeHalfType`; uses control flow (case) to specialize target builtin lowering.
- **CN**: 该代码块给出可调用入口的声明，例如 `MakeHalfType`；通过控制流（case）细化 目标内建函数降级 行为。

### Lines 1081-1100
```cpp
1081:                         E, *this);
1082:   case NVPTX::BI__nvvm_fmin_nan_f16:
1083:     return MakeHalfType(Intrinsic::nvvm_fmin_nan_f16, BuiltinID, E, *this);
1084:   case NVPTX::BI__nvvm_fmin_nan_f16x2:
1085:     return MakeHalfType(Intrinsic::nvvm_fmin_nan_f16x2, BuiltinID, E, *this);
1086:   case NVPTX::BI__nvvm_fmin_nan_xorsign_abs_f16:
1087:     return MakeHalfType(Intrinsic::nvvm_fmin_nan_xorsign_abs_f16, BuiltinID, E,
1088:                         *this);
1089:   case NVPTX::BI__nvvm_fmin_nan_xorsign_abs_f16x2:
1090:     return MakeHalfType(Intrinsic::nvvm_fmin_nan_xorsign_abs_f16x2, BuiltinID,
1091:                         E, *this);
1092:   case NVPTX::BI__nvvm_fmin_xorsign_abs_f16:
1093:     return MakeHalfType(Intrinsic::nvvm_fmin_xorsign_abs_f16, BuiltinID, E,
1094:                         *this);
1095:   case NVPTX::BI__nvvm_fmin_xorsign_abs_f16x2:
1096:     return MakeHalfType(Intrinsic::nvvm_fmin_xorsign_abs_f16x2, BuiltinID, E,
1097:                         *this);
1098:   case NVPTX::BI__nvvm_fabs_f:
1099:   case NVPTX::BI__nvvm_abs_bf16:
1100:   case NVPTX::BI__nvvm_abs_bf16x2:
```
- **EN**: This block spells out callable entry points like `MakeHalfType`; uses control flow (case) to specialize target builtin lowering.
- **CN**: 该代码块给出可调用入口的声明，例如 `MakeHalfType`；通过控制流（case）细化 目标内建函数降级 行为。

### Lines 1101-1120
```cpp
1101:   case NVPTX::BI__nvvm_fabs_f16:
1102:   case NVPTX::BI__nvvm_fabs_f16x2:
1103:     return Builder.CreateUnaryIntrinsic(Intrinsic::nvvm_fabs,
1104:                                         EmitScalarExpr(E->getArg(0)));
1105:   case NVPTX::BI__nvvm_fabs_ftz_f:
1106:   case NVPTX::BI__nvvm_fabs_ftz_f16:
1107:   case NVPTX::BI__nvvm_fabs_ftz_f16x2:
1108:     return Builder.CreateUnaryIntrinsic(Intrinsic::nvvm_fabs_ftz,
1109:                                         EmitScalarExpr(E->getArg(0)));
1110:   case NVPTX::BI__nvvm_fabs_d:
1111:     return Builder.CreateFAbs(EmitScalarExpr(E->getArg(0)));
1112:   case NVPTX::BI__nvvm_ex2_approx_d:
1113:   case NVPTX::BI__nvvm_ex2_approx_f:
1114:     return Builder.CreateUnaryIntrinsic(Intrinsic::nvvm_ex2_approx,
1115:                                         EmitScalarExpr(E->getArg(0)));
1116:   case NVPTX::BI__nvvm_ex2_approx_ftz_f:
1117:     return Builder.CreateUnaryIntrinsic(Intrinsic::nvvm_ex2_approx_ftz,
1118:                                         EmitScalarExpr(E->getArg(0)));
1119:   case NVPTX::BI__nvvm_ldg_h:
1120:   case NVPTX::BI__nvvm_ldg_h2:
```
- **EN**: This block spells out callable entry points like `EmitScalarExpr`; uses control flow (case) to specialize target builtin lowering.
- **CN**: 该代码块给出可调用入口的声明，例如 `EmitScalarExpr`；通过控制流（case）细化 目标内建函数降级 行为。

### Lines 1121-1140
```cpp
1121:     return EnsureNativeHalfSupport(BuiltinID, E, *this) ? MakeLdg(*this, E)
1122:                                                         : nullptr;
1123:   case NVPTX::BI__nvvm_ldu_h:
1124:   case NVPTX::BI__nvvm_ldu_h2:
1125:     return EnsureNativeHalfSupport(BuiltinID, E, *this)
1126:                ? MakeLdu(Intrinsic::nvvm_ldu_global_f, *this, E)
1127:                : nullptr;
1128:   case NVPTX::BI__nvvm_cp_async_ca_shared_global_4:
1129:     return MakeCpAsync(Intrinsic::nvvm_cp_async_ca_shared_global_4,
1130:                        Intrinsic::nvvm_cp_async_ca_shared_global_4_s, *this, E,
1131:                        4);
1132:   case NVPTX::BI__nvvm_cp_async_ca_shared_global_8:
1133:     return MakeCpAsync(Intrinsic::nvvm_cp_async_ca_shared_global_8,
1134:                        Intrinsic::nvvm_cp_async_ca_shared_global_8_s, *this, E,
1135:                        8);
1136:   case NVPTX::BI__nvvm_cp_async_ca_shared_global_16:
1137:     return MakeCpAsync(Intrinsic::nvvm_cp_async_ca_shared_global_16,
1138:                        Intrinsic::nvvm_cp_async_ca_shared_global_16_s, *this, E,
1139:                        16);
1140:   case NVPTX::BI__nvvm_cp_async_cg_shared_global_16:
```
- **EN**: This block spells out callable entry points like `MakeCpAsync`; uses control flow (case) to specialize target builtin lowering.
- **CN**: 该代码块给出可调用入口的声明，例如 `MakeCpAsync`；通过控制流（case）细化 目标内建函数降级 行为。

### Lines 1141-1160
```cpp
1141:     return MakeCpAsync(Intrinsic::nvvm_cp_async_cg_shared_global_16,
1142:                        Intrinsic::nvvm_cp_async_cg_shared_global_16_s, *this, E,
1143:                        16);
1144:   case NVPTX::BI__nvvm_read_ptx_sreg_clusterid_x:
1145:     return Builder.CreateCall(
1146:         CGM.getIntrinsic(Intrinsic::nvvm_read_ptx_sreg_clusterid_x));
1147:   case NVPTX::BI__nvvm_read_ptx_sreg_clusterid_y:
1148:     return Builder.CreateCall(
1149:         CGM.getIntrinsic(Intrinsic::nvvm_read_ptx_sreg_clusterid_y));
1150:   case NVPTX::BI__nvvm_read_ptx_sreg_clusterid_z:
1151:     return Builder.CreateCall(
1152:         CGM.getIntrinsic(Intrinsic::nvvm_read_ptx_sreg_clusterid_z));
1153:   case NVPTX::BI__nvvm_read_ptx_sreg_clusterid_w:
1154:     return Builder.CreateCall(
1155:         CGM.getIntrinsic(Intrinsic::nvvm_read_ptx_sreg_clusterid_w));
1156:   case NVPTX::BI__nvvm_read_ptx_sreg_nclusterid_x:
1157:     return Builder.CreateCall(
1158:         CGM.getIntrinsic(Intrinsic::nvvm_read_ptx_sreg_nclusterid_x));
1159:   case NVPTX::BI__nvvm_read_ptx_sreg_nclusterid_y:
1160:     return Builder.CreateCall(
```
- **EN**: This block spells out callable entry points like `MakeCpAsync`; uses control flow (case) to specialize target builtin lowering.
- **CN**: 该代码块给出可调用入口的声明，例如 `MakeCpAsync`；通过控制流（case）细化 目标内建函数降级 行为。

### Lines 1161-1180
```cpp
1161:         CGM.getIntrinsic(Intrinsic::nvvm_read_ptx_sreg_nclusterid_y));
1162:   case NVPTX::BI__nvvm_read_ptx_sreg_nclusterid_z:
1163:     return Builder.CreateCall(
1164:         CGM.getIntrinsic(Intrinsic::nvvm_read_ptx_sreg_nclusterid_z));
1165:   case NVPTX::BI__nvvm_read_ptx_sreg_nclusterid_w:
1166:     return Builder.CreateCall(
1167:         CGM.getIntrinsic(Intrinsic::nvvm_read_ptx_sreg_nclusterid_w));
1168:   case NVPTX::BI__nvvm_read_ptx_sreg_cluster_ctaid_x:
1169:     return Builder.CreateCall(
1170:         CGM.getIntrinsic(Intrinsic::nvvm_read_ptx_sreg_cluster_ctaid_x));
1171:   case NVPTX::BI__nvvm_read_ptx_sreg_cluster_ctaid_y:
1172:     return Builder.CreateCall(
1173:         CGM.getIntrinsic(Intrinsic::nvvm_read_ptx_sreg_cluster_ctaid_y));
1174:   case NVPTX::BI__nvvm_read_ptx_sreg_cluster_ctaid_z:
1175:     return Builder.CreateCall(
1176:         CGM.getIntrinsic(Intrinsic::nvvm_read_ptx_sreg_cluster_ctaid_z));
1177:   case NVPTX::BI__nvvm_read_ptx_sreg_cluster_ctaid_w:
1178:     return Builder.CreateCall(
1179:         CGM.getIntrinsic(Intrinsic::nvvm_read_ptx_sreg_cluster_ctaid_w));
1180:   case NVPTX::BI__nvvm_read_ptx_sreg_cluster_nctaid_x:
```
- **EN**: This block uses control flow (case) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（case）细化 目标内建函数降级 行为。

### Lines 1181-1200
```cpp
1181:     return Builder.CreateCall(
1182:         CGM.getIntrinsic(Intrinsic::nvvm_read_ptx_sreg_cluster_nctaid_x));
1183:   case NVPTX::BI__nvvm_read_ptx_sreg_cluster_nctaid_y:
1184:     return Builder.CreateCall(
1185:         CGM.getIntrinsic(Intrinsic::nvvm_read_ptx_sreg_cluster_nctaid_y));
1186:   case NVPTX::BI__nvvm_read_ptx_sreg_cluster_nctaid_z:
1187:     return Builder.CreateCall(
1188:         CGM.getIntrinsic(Intrinsic::nvvm_read_ptx_sreg_cluster_nctaid_z));
1189:   case NVPTX::BI__nvvm_read_ptx_sreg_cluster_nctaid_w:
1190:     return Builder.CreateCall(
1191:         CGM.getIntrinsic(Intrinsic::nvvm_read_ptx_sreg_cluster_nctaid_w));
1192:   case NVPTX::BI__nvvm_read_ptx_sreg_cluster_ctarank:
1193:     return Builder.CreateCall(
1194:         CGM.getIntrinsic(Intrinsic::nvvm_read_ptx_sreg_cluster_ctarank));
1195:   case NVPTX::BI__nvvm_read_ptx_sreg_cluster_nctarank:
1196:     return Builder.CreateCall(
1197:         CGM.getIntrinsic(Intrinsic::nvvm_read_ptx_sreg_cluster_nctarank));
1198:   case NVPTX::BI__nvvm_is_explicit_cluster:
1199:     return Builder.CreateCall(
1200:         CGM.getIntrinsic(Intrinsic::nvvm_is_explicit_cluster));
```
- **EN**: This block uses control flow (case) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（case）细化 目标内建函数降级 行为。

### Lines 1201-1220
```cpp
1201:   case NVPTX::BI__nvvm_isspacep_shared_cluster:
1202:     return Builder.CreateCall(
1203:         CGM.getIntrinsic(Intrinsic::nvvm_isspacep_shared_cluster),
1204:         EmitScalarExpr(E->getArg(0)));
1205:   case NVPTX::BI__nvvm_mapa:
1206:     return Builder.CreateCall(
1207:         CGM.getIntrinsic(Intrinsic::nvvm_mapa),
1208:         {EmitScalarExpr(E->getArg(0)), EmitScalarExpr(E->getArg(1))});
1209:   case NVPTX::BI__nvvm_mapa_shared_cluster:
1210:     return Builder.CreateCall(
1211:         CGM.getIntrinsic(Intrinsic::nvvm_mapa_shared_cluster),
1212:         {EmitScalarExpr(E->getArg(0)), EmitScalarExpr(E->getArg(1))});
1213:   case NVPTX::BI__nvvm_getctarank:
1214:     return Builder.CreateCall(
1215:         CGM.getIntrinsic(Intrinsic::nvvm_getctarank),
1216:         EmitScalarExpr(E->getArg(0)));
1217:   case NVPTX::BI__nvvm_getctarank_shared_cluster:
1218:     return Builder.CreateCall(
1219:         CGM.getIntrinsic(Intrinsic::nvvm_getctarank_shared_cluster),
1220:         EmitScalarExpr(E->getArg(0)));
```
- **EN**: This block defines callable entry points like `EmitScalarExpr`; uses control flow (case) to specialize target builtin lowering.
- **CN**: 该代码块定义可调用入口，例如 `EmitScalarExpr`；通过控制流（case）细化 目标内建函数降级 行为。

### Lines 1221-1240
```cpp
1221:   case NVPTX::BI__nvvm_barrier_cluster_arrive:
1222:     return Builder.CreateCall(
1223:         CGM.getIntrinsic(Intrinsic::nvvm_barrier_cluster_arrive));
1224:   case NVPTX::BI__nvvm_barrier_cluster_arrive_relaxed:
1225:     return Builder.CreateCall(
1226:         CGM.getIntrinsic(Intrinsic::nvvm_barrier_cluster_arrive_relaxed));
1227:   case NVPTX::BI__nvvm_barrier_cluster_wait:
1228:     return Builder.CreateCall(
1229:         CGM.getIntrinsic(Intrinsic::nvvm_barrier_cluster_wait));
1230:   case NVPTX::BI__nvvm_fence_sc_cluster:
1231:     return Builder.CreateCall(
1232:         CGM.getIntrinsic(Intrinsic::nvvm_fence_sc_cluster));
1233:   case NVPTX::BI__nvvm_bar_sync:
1234:     return Builder.CreateCall(
1235:         CGM.getIntrinsic(Intrinsic::nvvm_barrier_cta_sync_aligned_all),
1236:         EmitScalarExpr(E->getArg(0)));
1237:   case NVPTX::BI__syncthreads:
1238:     return Builder.CreateCall(
1239:         CGM.getIntrinsic(Intrinsic::nvvm_barrier_cta_sync_aligned_all),
1240:         Builder.getInt32(0));
```
- **EN**: This block spells out callable entry points like `EmitScalarExpr`; uses control flow (case) to specialize target builtin lowering.
- **CN**: 该代码块给出可调用入口的声明，例如 `EmitScalarExpr`；通过控制流（case）细化 目标内建函数降级 行为。

### Lines 1241-1260
```cpp
1241:   case NVPTX::BI__nvvm_barrier_sync:
1242:     return Builder.CreateCall(
1243:         CGM.getIntrinsic(Intrinsic::nvvm_barrier_cta_sync_all),
1244:         EmitScalarExpr(E->getArg(0)));
1245:   case NVPTX::BI__nvvm_barrier_sync_cnt:
1246:     return Builder.CreateCall(
1247:         CGM.getIntrinsic(Intrinsic::nvvm_barrier_cta_sync_count),
1248:         {EmitScalarExpr(E->getArg(0)), EmitScalarExpr(E->getArg(1))});
1249:   case NVPTX::BI__nvvm_bar0_and:
1250:     return Builder.CreateZExt(
1251:         Builder.CreateIntrinsic(
1252:             Intrinsic::nvvm_barrier_cta_red_and_aligned_all, {},
1253:             {Builder.getInt32(0),
1254:              Builder.CreateICmpNE(EmitScalarExpr(E->getArg(0)),
1255:                                   Builder.getInt32(0))}),
1256:         Builder.getInt32Ty());
1257:   case NVPTX::BI__nvvm_bar0_or:
1258:     return Builder.CreateZExt(
1259:         Builder.CreateIntrinsic(
1260:             Intrinsic::nvvm_barrier_cta_red_or_aligned_all, {},
```
- **EN**: This block defines callable entry points like `EmitScalarExpr`; uses control flow (case) to specialize target builtin lowering.
- **CN**: 该代码块定义可调用入口，例如 `EmitScalarExpr`；通过控制流（case）细化 目标内建函数降级 行为。

### Lines 1261-1273
```cpp
1261:             {Builder.getInt32(0),
1262:              Builder.CreateICmpNE(EmitScalarExpr(E->getArg(0)),
1263:                                   Builder.getInt32(0))}),
1264:         Builder.getInt32Ty());
1265:   case NVPTX::BI__nvvm_bar0_popc:
1266:     return Builder.CreateIntrinsic(
1267:         Intrinsic::nvvm_barrier_cta_red_popc_aligned_all, {},
1268:         {Builder.getInt32(0), Builder.CreateICmpNE(EmitScalarExpr(E->getArg(0)),
1269:                                                    Builder.getInt32(0))});
1270:   default:
1271:     return nullptr;
1272:   }
1273: }
```
- **EN**: This block uses control flow (case) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（case）细化 目标内建函数降级 行为。

## Key Concepts / 关键概念

- **Intrinsic**: Frequently referenced symbol that likely anchors a key abstraction in this file. / 高频出现的符号，通常对应此文件中的关键抽象。
- **Builder**: Acts as a construction helper that incrementally assembles target builtin lowering state. / 充当构建辅助器，逐步组装 目标内建函数降级 状态。
- **BuiltinID**: Frequently referenced symbol that likely anchors a key abstraction in this file. / 高频出现的符号，通常对应此文件中的关键抽象。
- **getArg**: Frequently referenced symbol that likely anchors a key abstraction in this file. / 高频出现的符号，通常对应此文件中的关键抽象。
- **MMA_LDST**: Central symbol in this file's implementation of target builtin lowering. / 是该文件实现 目标内建函数降级 时的核心符号。
- **MakeHalfType**: Frequently referenced symbol that likely anchors a key abstraction in this file. / 高频出现的符号，通常对应此文件中的关键抽象。
- **CGM**: Central symbol in this file's implementation of target builtin lowering. / 是该文件实现 目标内建函数降级 时的核心符号。
- **CGF**: Central symbol in this file's implementation of target builtin lowering. / 是该文件实现 目标内建函数降级 时的核心符号。

## Dependencies / 依赖关系

- **Local CodeGen / 本地 CodeGen**: `CGBuiltin.h`
- **Clang libraries / Clang 库**: `clang/Basic/TargetBuiltins.h`
- **LLVM libraries / LLVM 库**: `llvm/IR/IntrinsicsNVPTX.h`
