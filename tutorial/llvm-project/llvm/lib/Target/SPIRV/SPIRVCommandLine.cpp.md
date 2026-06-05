# SPIRVCommandLine.cpp — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/SPIRV/SPIRVCommandLine.cpp`
- Repository: `llvm-project`
- Purpose (EN): This file contains definitions of classes and functions needed for processing, parsing, and using CLI options for the SPIR-V backend.
- 目的（中文）: 该文件用于支撑 LLVM 目标后端的相关功能。
- Language: C++ source

## Line-by-Line Analysis / 逐行分析

> Note / 说明: To keep the document readable, the source is analyzed in contiguous line ranges while preserving the original order. Each code block prefixes original line numbers.

### Lines 1-40
```cpp
 1: //===--- SPIRVCommandLine.cpp ---- Command Line Options ---------*- C++ -*-===//
 2: //
 3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
 4: // See https://llvm.org/LICENSE.txt for license information.
 5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
 6: //
 7: //===----------------------------------------------------------------------===//
 8: //
 9: // This file contains definitions of classes and functions needed for
10: // processing, parsing, and using CLI options for the SPIR-V backend.
11: //
12: //===----------------------------------------------------------------------===//
13:
14: #include "SPIRVCommandLine.h"
15: #include "MCTargetDesc/SPIRVBaseInfo.h"
16: #include "llvm/ADT/STLExtras.h"
17: #include "llvm/ADT/StringMap.h"
18: #include "llvm/TargetParser/Triple.h"
19:
20: #include <functional>
21: #include <iterator>
22: #include <string>
23: #include <utility>
24: #include <vector>
25:
26: #define DEBUG_TYPE "spirv-commandline"
27:
28: using namespace llvm;
29:
30: ExtensionSet SPIRVExtensionsParser::DisabledExtensions;
31:
32: static const StringMap<SPIRV::Extension::Extension> SPIRVExtensionMap = {
33:     {"SPV_EXT_shader_atomic_float_add",
34:      SPIRV::Extension::Extension::SPV_EXT_shader_atomic_float_add},
35:     {"SPV_EXT_shader_atomic_float16_add",
36:      SPIRV::Extension::Extension::SPV_EXT_shader_atomic_float16_add},
37:     {"SPV_EXT_shader_atomic_float_min_max",
38:      SPIRV::Extension::Extension::SPV_EXT_shader_atomic_float_min_max},
39:     {"SPV_INTEL_16bit_atomics",
40:      SPIRV::Extension::Extension::SPV_INTEL_16bit_atomics},
```
- EN: This range contains connective backend logic, including helper statements, control flow, and data movement needed by the surrounding implementation.
- CN: 这一段包含衔接性的后端逻辑，包括辅助语句、控制流和数据处理，用于支撑周围实现。

### Lines 41-80
```cpp
41:     {"SPV_NV_shader_atomic_fp16_vector",
42:      SPIRV::Extension::Extension::SPV_NV_shader_atomic_fp16_vector},
43:     {"SPV_EXT_arithmetic_fence",
44:      SPIRV::Extension::Extension::SPV_EXT_arithmetic_fence},
45:     {"SPV_EXT_demote_to_helper_invocation",
46:      SPIRV::Extension::Extension::SPV_EXT_demote_to_helper_invocation},
47:     {"SPV_EXT_descriptor_indexing",
48:      SPIRV::Extension::Extension::SPV_EXT_descriptor_indexing},
49:     {"SPV_EXT_fragment_fully_covered",
50:      SPIRV::Extension::Extension::SPV_EXT_fragment_fully_covered},
51:     {"SPV_EXT_fragment_invocation_density",
52:      SPIRV::Extension::Extension::SPV_EXT_fragment_invocation_density},
53:     {"SPV_EXT_fragment_shader_interlock",
54:      SPIRV::Extension::Extension::SPV_EXT_fragment_shader_interlock},
55:     {"SPV_EXT_mesh_shader", SPIRV::Extension::Extension::SPV_EXT_mesh_shader},
56:     {"SPV_EXT_shader_stencil_export",
57:      SPIRV::Extension::Extension::SPV_EXT_shader_stencil_export},
58:     {"SPV_EXT_shader_viewport_index_layer",
59:      SPIRV::Extension::Extension::SPV_EXT_shader_viewport_index_layer},
60:     {"SPV_GOOGLE_hlsl_functionality1",
61:      SPIRV::Extension::Extension::SPV_GOOGLE_hlsl_functionality1},
62:     {"SPV_GOOGLE_user_type", SPIRV::Extension::Extension::SPV_GOOGLE_user_type},
63:     {"SPV_ALTERA_arbitrary_precision_integers",
64:      SPIRV::Extension::Extension::SPV_ALTERA_arbitrary_precision_integers},
65:     {"SPV_ALTERA_arbitrary_precision_floating_point",
66:      SPIRV::Extension::Extension::
67:          SPV_ALTERA_arbitrary_precision_floating_point},
68:     {"SPV_INTEL_cache_controls",
69:      SPIRV::Extension::Extension::SPV_INTEL_cache_controls},
70:     {"SPV_INTEL_float_controls2",
71:      SPIRV::Extension::Extension::SPV_INTEL_float_controls2},
72:     {"SPV_INTEL_global_variable_fpga_decorations",
73:      SPIRV::Extension::Extension::SPV_INTEL_global_variable_fpga_decorations},
74:     {"SPV_INTEL_global_variable_host_access",
75:      SPIRV::Extension::Extension::SPV_INTEL_global_variable_host_access},
76:     {"SPV_INTEL_optnone", SPIRV::Extension::Extension::SPV_INTEL_optnone},
77:     {"SPV_EXT_optnone", SPIRV::Extension::Extension::SPV_EXT_optnone},
78:     {"SPV_INTEL_usm_storage_classes",
79:      SPIRV::Extension::Extension::SPV_INTEL_usm_storage_classes},
80:     {"SPV_INTEL_split_barrier",
```
- EN: This range contains connective backend logic, including helper statements, control flow, and data movement needed by the surrounding implementation.
- CN: 这一段包含衔接性的后端逻辑，包括辅助语句、控制流和数据处理，用于支撑周围实现。

### Lines 81-120
```cpp
 81:      SPIRV::Extension::Extension::SPV_INTEL_split_barrier},
 82:     {"SPV_INTEL_subgroups", SPIRV::Extension::Extension::SPV_INTEL_subgroups},
 83:     {"SPV_INTEL_media_block_io",
 84:      SPIRV::Extension::Extension::SPV_INTEL_media_block_io},
 85:     {"SPV_INTEL_memory_access_aliasing",
 86:      SPIRV::Extension::Extension::SPV_INTEL_memory_access_aliasing},
 87:     {"SPV_INTEL_joint_matrix",
 88:      SPIRV::Extension::Extension::SPV_INTEL_joint_matrix},
 89:     {"SPV_INTEL_masked_gather_scatter",
 90:      SPIRV::Extension::Extension::SPV_INTEL_masked_gather_scatter},
 91:     {"SPV_KHR_16bit_storage",
 92:      SPIRV::Extension::Extension::SPV_KHR_16bit_storage},
 93:     {"SPV_KHR_device_group", SPIRV::Extension::Extension::SPV_KHR_device_group},
 94:     {"SPV_KHR_fragment_shading_rate",
 95:      SPIRV::Extension::Extension::SPV_KHR_fragment_shading_rate},
 96:     {"SPV_KHR_multiview", SPIRV::Extension::Extension::SPV_KHR_multiview},
 97:     {"SPV_KHR_post_depth_coverage",
 98:      SPIRV::Extension::Extension::SPV_KHR_post_depth_coverage},
 99:     {"SPV_KHR_shader_draw_parameters",
100:      SPIRV::Extension::Extension::SPV_KHR_shader_draw_parameters},
101:     {"SPV_KHR_ray_tracing", SPIRV::Extension::Extension::SPV_KHR_ray_tracing},
102:     {"SPV_KHR_uniform_group_instructions",
103:      SPIRV::Extension::Extension::SPV_KHR_uniform_group_instructions},
104:     {"SPV_KHR_no_integer_wrap_decoration",
105:      SPIRV::Extension::Extension::SPV_KHR_no_integer_wrap_decoration},
106:     {"SPV_KHR_float_controls",
107:      SPIRV::Extension::Extension::SPV_KHR_float_controls},
108:     {"SPV_KHR_expect_assume",
109:      SPIRV::Extension::Extension::SPV_KHR_expect_assume},
110:     {"SPV_KHR_bit_instructions",
111:      SPIRV::Extension::Extension::SPV_KHR_bit_instructions},
112:     {"SPV_KHR_integer_dot_product",
113:      SPIRV::Extension::Extension::SPV_KHR_integer_dot_product},
114:     {"SPV_KHR_linkonce_odr", SPIRV::Extension::Extension::SPV_KHR_linkonce_odr},
115:     {"SPV_KHR_fma", SPIRV::Extension::Extension::SPV_KHR_fma},
116:     {"SPV_INTEL_inline_assembly",
117:      SPIRV::Extension::Extension::SPV_INTEL_inline_assembly},
118:     {"SPV_INTEL_bindless_images",
119:      SPIRV::Extension::Extension::SPV_INTEL_bindless_images},
120:     {"SPV_INTEL_bfloat16_arithmetic",
```
- EN: This range contains connective backend logic, including helper statements, control flow, and data movement needed by the surrounding implementation.
- CN: 这一段包含衔接性的后端逻辑，包括辅助语句、控制流和数据处理，用于支撑周围实现。

### Lines 121-160
```cpp
121:      SPIRV::Extension::Extension::SPV_INTEL_bfloat16_arithmetic},
122:     {"SPV_INTEL_bfloat16_conversion",
123:      SPIRV::Extension::Extension::SPV_INTEL_bfloat16_conversion},
124:     {"SPV_KHR_subgroup_rotate",
125:      SPIRV::Extension::Extension::SPV_KHR_subgroup_rotate},
126:     {"SPV_INTEL_variable_length_array",
127:      SPIRV::Extension::Extension::SPV_INTEL_variable_length_array},
128:     {"SPV_INTEL_function_pointers",
129:      SPIRV::Extension::Extension::SPV_INTEL_function_pointers},
130:     {"SPV_KHR_shader_clock", SPIRV::Extension::Extension::SPV_KHR_shader_clock},
131:     {"SPV_KHR_cooperative_matrix",
132:      SPIRV::Extension::Extension::SPV_KHR_cooperative_matrix},
133:     {"SPV_KHR_non_semantic_info",
134:      SPIRV::Extension::Extension::SPV_KHR_non_semantic_info},
135:     {"SPV_KHR_ray_query", SPIRV::Extension::Extension::SPV_KHR_ray_query},
136:     {"SPV_EXT_shader_image_int64",
137:      SPIRV::Extension::Extension::SPV_EXT_shader_image_int64},
138:     {"SPV_KHR_fragment_shader_barycentric",
139:      SPIRV::Extension::Extension::SPV_KHR_fragment_shader_barycentric},
140:     {"SPV_KHR_physical_storage_buffer",
141:      SPIRV::Extension::Extension::SPV_KHR_physical_storage_buffer},
142:     {"SPV_KHR_vulkan_memory_model",
143:      SPIRV::Extension::Extension::SPV_KHR_vulkan_memory_model},
144:     {"SPV_NV_shader_subgroup_partitioned",
145:      SPIRV::Extension::Extension::SPV_NV_shader_subgroup_partitioned},
146:     {"SPV_INTEL_long_composites",
147:      SPIRV::Extension::Extension::SPV_INTEL_long_composites},
148:     {"SPV_INTEL_fp_max_error",
149:      SPIRV::Extension::Extension::SPV_INTEL_fp_max_error},
150:     {"SPV_INTEL_subgroup_matrix_multiply_accumulate",
151:      SPIRV::Extension::Extension::
152:          SPV_INTEL_subgroup_matrix_multiply_accumulate},
153:     {"SPV_INTEL_ternary_bitwise_function",
154:      SPIRV::Extension::Extension::SPV_INTEL_ternary_bitwise_function},
155:     {"SPV_INTEL_2d_block_io",
156:      SPIRV::Extension::Extension::SPV_INTEL_2d_block_io},
157:     {"SPV_INTEL_int4", SPIRV::Extension::Extension::SPV_INTEL_int4},
158:     {"SPV_KHR_float_controls2",
159:      SPIRV::Extension::Extension::SPV_KHR_float_controls2},
160:     {"SPV_INTEL_tensor_float32_conversion",
```
- EN: This range contains connective backend logic, including helper statements, control flow, and data movement needed by the surrounding implementation.
- CN: 这一段包含衔接性的后端逻辑，包括辅助语句、控制流和数据处理，用于支撑周围实现。

### Lines 161-200
```cpp
161:      SPIRV::Extension::Extension::SPV_INTEL_tensor_float32_conversion},
162:     {"SPV_KHR_bfloat16", SPIRV::Extension::Extension::SPV_KHR_bfloat16},
163:     {"SPV_EXT_relaxed_printf_string_address_space",
164:      SPIRV::Extension::Extension::SPV_EXT_relaxed_printf_string_address_space},
165:     {"SPV_INTEL_predicated_io",
166:      SPIRV::Extension::Extension::SPV_INTEL_predicated_io},
167:     {"SPV_KHR_maximal_reconvergence",
168:      SPIRV::Extension::Extension::SPV_KHR_maximal_reconvergence},
169:     {"SPV_INTEL_kernel_attributes",
170:      SPIRV::Extension::Extension::SPV_INTEL_kernel_attributes},
171:     {"SPV_ALTERA_blocking_pipes",
172:      SPIRV::Extension::Extension::SPV_ALTERA_blocking_pipes},
173:     {"SPV_INTEL_int4", SPIRV::Extension::Extension::SPV_INTEL_int4},
174:     {"SPV_ALTERA_arbitrary_precision_fixed_point",
175:      SPIRV::Extension::Extension::SPV_ALTERA_arbitrary_precision_fixed_point},
176:     {"SPV_EXT_image_raw10_raw12",
177:      SPIRV::Extension::Extension::SPV_EXT_image_raw10_raw12},
178:     {"SPV_INTEL_unstructured_loop_controls",
179:      SPIRV::Extension::Extension::SPV_INTEL_unstructured_loop_controls},
180:     {"SPV_AMD_weak_linkage", SPIRV::Extension::Extension::SPV_AMD_weak_linkage},
181:     {"SPV_KHR_abort", SPIRV::Extension::Extension::SPV_KHR_abort}};
182:
183: bool SPIRVExtensionsParser::parse(cl::Option &O, StringRef ArgName,
184:                                   StringRef ArgValue, ExtensionSet &Vals) {
185:   SmallVector<StringRef, 10> Tokens;
186:   ArgValue.split(Tokens, ",", -1, false);
187:
188:   ExtensionSet EnabledExtensions;
189:
190:   auto M = partition(Tokens, [](auto &&T) { return T.starts_with('+'); });
191:
192:   if (std::any_of(M, Tokens.end(), equal_to("all")))
193:     for (auto &&El : make_second_range(SPIRVExtensionMap))
194:       Vals.insert(El);
195:
196:   for (auto &&Token : make_range(Tokens.begin(), M)) {
197:     StringRef ExtensionName = Token.substr(1);
198:     auto NameValuePair = SPIRVExtensionMap.find(ExtensionName);
199:
200:     if (NameValuePair == SPIRVExtensionMap.end())
```
- EN: This range implements operational logic in helpers such as split, partition, insert, substr, translating backend policy into executable code.
- CN: 这一段实现了 split、partition、insert、substr 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 201-240
```cpp
201:       return O.error("Unknown SPIR-V extension: " + Token.str());
202:
203:     EnabledExtensions.insert(NameValuePair->second);
204:   }
205:
206:   for (auto &&Token : make_range(M, Tokens.end())) {
207:     if (Token == "all")
208:       continue;
209:
210:     if (Token.size() == 3 && Token.upper() == "KHR") {
211:       for (const auto &[ExtensionName, ExtensionEnum] : SPIRVExtensionMap)
212:         if (StringRef(ExtensionName).starts_with("SPV_KHR_"))
213:           Vals.insert(ExtensionEnum);
214:       continue;
215:     }
216:
217:     if (Token.empty() || (!Token.starts_with("+") && !Token.starts_with("-")))
218:       return O.error("Invalid extension list format: " + Token);
219:
220:     auto NameValuePair = SPIRVExtensionMap.find(Token.substr(1));
221:
222:     if (NameValuePair == SPIRVExtensionMap.end())
223:       return O.error("Unknown SPIR-V extension: " + Token.str());
224:     if (EnabledExtensions.count(NameValuePair->second))
225:       return O.error(
226:           "Extension cannot be allowed and disallowed at the same time: " +
227:           NameValuePair->first());
228:     DisabledExtensions.insert(NameValuePair->second);
229:     Vals.erase(NameValuePair->second);
230:   }
231:
232:   Vals.insert(EnabledExtensions.begin(), EnabledExtensions.end());
233:
234:   return false;
235: }
236:
237: StringRef
238: SPIRVExtensionsParser::checkExtensions(const std::vector<std::string> &ExtNames,
239:                                        ExtensionSet &AllowedExtensions) {
240:   for (const auto &Ext : ExtNames) {
```
- EN: This range implements operational logic in helpers such as error, insert, find, first, translating backend policy into executable code.
- CN: 这一段实现了 error、insert、find、first 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 241-272
```cpp
241:     if (Ext == "all") {
242:       for (const auto &[ExtensionName, ExtensionEnum] : SPIRVExtensionMap)
243:         AllowedExtensions.insert(ExtensionEnum);
244:       break;
245:     }
246:     auto It = SPIRVExtensionMap.find(Ext);
247:     if (It == SPIRVExtensionMap.end())
248:       return Ext;
249:     AllowedExtensions.insert(It->second);
250:   }
251:   return StringRef();
252: }
253:
254: ExtensionSet SPIRVExtensionsParser::getValidExtensions(const Triple &TT) {
255:   ExtensionSet R;
256:   SPIRV::Environment::Environment CurrentEnvironment =
257:       SPIRV::Environment::Environment::EnvOpenCL;
258:   if (TT.getOS() == Triple::Vulkan)
259:     CurrentEnvironment = SPIRV::Environment::Environment::EnvVulkan;
260:
261:   for (const auto &[ExtensionName, ExtensionEnum] : SPIRVExtensionMap) {
262:     EnvironmentList AllowedEnv = getSymbolicOperandAllowedEnvironments(
263:         SPIRV::OperandCategory::OperandCategory::ExtensionOperand,
264:         ExtensionEnum);
265:
266:     if (llvm::is_contained(AllowedEnv, CurrentEnvironment) &&
267:         !llvm::is_contained(DisabledExtensions, ExtensionEnum))
268:       R.insert(ExtensionEnum);
269:   }
270:
271:   return R;
272: }
```
- EN: This range implements operational logic in helpers such as insert, find, StringRef, SPIRVExtensionsParser::getValidExtensions, translating backend policy into executable code.
- CN: 这一段实现了 insert、find、StringRef、SPIRVExtensionsParser::getValidExtensions 等操作逻辑，把后端策略落实为可执行的代码路径。

## Key Concepts / 关键概念

- EN: SPIR-V backend code must preserve rich type information and encode capabilities/extensions explicitly.
  - CN: SPIR-V 后端需要保留丰富的类型信息，并显式编码 capability 与 extension。
- EN: Key symbols in this file include split, partition, insert, substr, find, which anchor the file's main abstractions.
  - CN: 该文件中的关键符号包括 split, partition, insert, substr, find，它们构成了本文件的核心抽象。

## Dependencies / 依赖关系

- Local backend headers / 本地后端头文件:
  - `SPIRVCommandLine.h`
  - `MCTargetDesc/SPIRVBaseInfo.h`
- LLVM infrastructure / LLVM 基础设施:
  - `llvm/ADT/STLExtras.h`
  - `llvm/ADT/StringMap.h`
  - `llvm/TargetParser/Triple.h`
- System/standard headers / 系统或标准头文件:
  - `functional`
  - `iterator`
  - `string`
  - `utility`
  - `vector`
