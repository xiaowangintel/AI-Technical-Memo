# SPIRVSymbolicOperands.td — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/SPIRV/SPIRVSymbolicOperands.td`
- Repository: `llvm-project`
- Purpose (EN): This file defines symbolic named operands for various SPIR-V instructions.
- 目的（中文）: 该文件使用 TableGen DSL 描述目标后端元数据，并驱动自动生成代码。
- Language: TableGen DSL (TableGen DSL noted below)
- Note: This file is written in TableGen DSL, so many records are declarative descriptions consumed by LLVM code generators.
- 说明：该文件使用 TableGen DSL 编写，因此许多记录都是供 LLVM 代码生成器消费的声明式描述。

## Line-by-Line Analysis / 逐行分析

> Note / 说明: To keep the document readable, the source is analyzed in contiguous line ranges while preserving the original order. Each code block prefixes original line numbers.

### Lines 1-120
```tablegen
  1: //===- SPIRVSymbolicOperands.td ----------------------------*- tablegen -*-===//
  2: //
  3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
  4: // See https://llvm.org/LICENSE.txt for license information.
  5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
  6: //
  7: //===----------------------------------------------------------------------===//
  8: //
  9: // This file defines symbolic/named operands for various SPIR-V instructions.
 10: //
 11: //===----------------------------------------------------------------------===//
 12:
 13: include "llvm/TableGen/SearchableTable.td"
 14:
 15: //===----------------------------------------------------------------------===//
 16: // Lookup table containing symbolic operands with the following columns:
 17: // - Category (Extension/Capability/BuiltIn/etc.)
 18: // - Value (32-bit representation for binary emission)
 19: // - Mnemonic (String representation for textual emission)
 20: // - MinVersion
 21: // - MaxVersion
 22: //===----------------------------------------------------------------------===//
 23:
 24: // Forward-declare classes used in SymbolicOperand
 25: class OperandCategory;
 26:
 27: class SymbolicOperand<OperandCategory category, bits<32> value, string mnemonic, bits<32> minVersion, bits<32> maxVersion> {
 28:   OperandCategory Category = category;
 29:   bits<32> Value = value;
 30:   string Mnemonic = mnemonic;
 31:   bits<32> MinVersion = minVersion;
 32:   bits<32> MaxVersion = maxVersion;
 33: }
 34:
 35: def SymbolicOperands : GenericTable {
 36:   let FilterClass = "SymbolicOperand";
 37:   let Fields = ["Category", "Value", "Mnemonic", "MinVersion", "MaxVersion"];
 38:   string TypeOf_Category = "OperandCategory";
 39:   let PrimaryKey = ["Category", "Value"];
 40:   // Function for looking up symbolic operands based on category and value.
 41:   let PrimaryKeyName = "lookupSymbolicOperandByCategoryAndValue";
 42: }
 43:
 44: // Function for looking up symbolic operands based on just category.
 45: def lookupSymbolicOperandByCategory : SearchIndex {
 46:   let Table = SymbolicOperands;
 47:   let Key = ["Category"];
 48: }
 49:
 50: // Function for looking up symbolic operands based on category and mnemonic.
 51: def lookupSymbolicOperandByCategoryAndMnemonic : SearchIndex {
 52:   let Table = SymbolicOperands;
 53:   let Key = ["Category", "Mnemonic"];
 54: }
 55:
 56: //===----------------------------------------------------------------------===//
 57: // Lookup table for matching symbolic operands (category + 32-bit value) to
 58: // a SPIR-V extension.
 59: //===----------------------------------------------------------------------===//
 60:
 61: // Forward-declare classes used in ExtensionEntry
 62: class Extension;
 63:
 64: class ExtensionEntry<OperandCategory category, bits<32> value, Extension reqExtension> {
 65:   OperandCategory Category = category;
 66:   bits<32> Value = value;
 67:   Extension ReqExtension = reqExtension;
 68: }
 69:
 70: def ExtensionEntries : GenericTable {
 71:   let FilterClass = "ExtensionEntry";
 72:   let Fields = ["Category", "Value", "ReqExtension"];
 73:   string TypeOf_Category = "OperandCategory";
 74:   string TypeOf_ReqExtension = "Extension";
 75:   let PrimaryKey = ["Category", "Value"];
 76:   // Function for looking up the extension by category + value.
 77:   let PrimaryKeyName = "lookupExtensionByCategoryAndValue";
 78: }
 79:
 80: // Function to lookup symbolic operands enabled by a given extension.
 81: def lookupSymbolicOperandsEnabledByExtension : SearchIndex {
 82:   let Table = ExtensionEntries;
 83:   let Key = ["ReqExtension", "Category"];
 84: }
 85:
 86: //===----------------------------------------------------------------------===//
 87: // Lookup table for matching symbolic operands (category + 32-bit value) to
 88: // SPIR-V capabilities. If an operand requires more than one capability, there
 89: // will be multiple consecutive entries present in the table.
 90: //===----------------------------------------------------------------------===//
 91:
 92: // Forward-declare classes used in ExtensionEntry
 93: class Capability;
 94:
 95: class CapabilityEntry<OperandCategory category, bits<32> value, Capability reqCabaility> {
 96:   OperandCategory Category = category;
 97:   bits<32> Value = value;
 98:   Capability ReqCapability = reqCabaility;
 99: }
100:
101: def CapabilityEntries : GenericTable {
102:   let FilterClass = "CapabilityEntry";
103:   let Fields = ["Category", "Value", "ReqCapability"];
104:   string TypeOf_Category = "OperandCategory";
105:   string TypeOf_ReqCapability = "Capability";
106:   let PrimaryKey = ["Category", "Value"];
107:   // Function for looking up a (the first) capability by category + value. Next
108:   // capabilities should be consecutive.
109:   let PrimaryKeyName = "lookupCapabilityByCategoryAndValue";
110: }
111:
112: //===----------------------------------------------------------------------===//
113: // Lookup table for matching symbolic operands (category + 32-bit value) to
114: // SPIR-V environments. If an operand is allows in more than one environment,
115: // there will be multiple consecutive entries present in the table.
116: //===----------------------------------------------------------------------===//
117:
118: // Forward-declare classes used in ExtensionEntry
119: class Environment;
120:
```
- EN: This range uses TableGen DSL to describe records such as OperandCategory;, SymbolicOperand, SymbolicOperands, lookupSymbolicOperandByCategory; LLVM later expands these declarations into generated backend tables and helper code.
- CN: 这一段使用 TableGen DSL 描述 OperandCategory;、SymbolicOperand、SymbolicOperands、lookupSymbolicOperandByCategory 等记录；LLVM 后续会把这些声明展开为生成的后端表和辅助代码。

### Lines 121-240
```tablegen
121: class EnvironmentEntry<OperandCategory category, bits<32> value,
122:                        Environment allowedEnvironment> {
123:   OperandCategory Category = category;
124:   bits<32> Value = value;
125:   Environment AllowedEnvironment = allowedEnvironment;
126: }
127:
128: def EnvironmentEntries : GenericTable {
129:   let FilterClass = "EnvironmentEntry";
130:   let Fields = ["Category", "Value", "AllowedEnvironment"];
131:   string TypeOf_Category = "OperandCategory";
132:   string TypeOf_AllowedEnvironment = "Environment";
133:   let PrimaryKey = ["Category", "Value"];
134:   // Function for looking up a (the first) environment by category + value. Next
135:   // environment should be consecutive.
136:   let PrimaryKeyName = "lookupEnvironmentByCategoryAndValue";
137: }
138:
139: //===----------------------------------------------------------------------===//
140: // Multiclass used to define a SymbolicOperand and at the same time declare
141: // required extension and capabilities.
142: //===----------------------------------------------------------------------===//
143:
144: multiclass SymbolicOperandWithRequirements<
145:     OperandCategory category, bits<32> value, string mnemonic,
146:     bits<32> minVersion, bits<32> maxVersion, list<Extension> reqExtensions,
147:     list<Capability> reqCapabilities, list<Environment> allowedEnvironments> {
148:   assert !ge(!size(mnemonic), 1), "No mnemonic/string representation provided "
149:                                   "for symbolic operand with value "#value;
150:   def : SymbolicOperand<category, value, mnemonic, minVersion, maxVersion>;
151:
152:   assert !le(!size(reqExtensions), 1),
153:       "Too many required extensions for a symbolic/named operand: "#mnemonic;
154:   if !eq(!size(reqExtensions), 1) then {
155:     def : ExtensionEntry<category, value, reqExtensions[0]>;
156:   }
157:
158:   foreach capability = reqCapabilities in {
159:     def : CapabilityEntry<category, value, capability>;
160:   }
161:
162:   foreach environment = allowedEnvironments in {
163:     def : EnvironmentEntry<category, value, environment>;
164:   }
165: }
166:
167: //===----------------------------------------------------------------------===//
168: // Enum defining different categories of symbolic/named operands.
169: //===----------------------------------------------------------------------===//
170:
171: def OperandCategory : GenericEnum {
172:   let FilterClass = "OperandCategory";
173: }
174:
175: class OperandCategory;
176:
177: def ExtensionOperand : OperandCategory;
178: def CapabilityOperand : OperandCategory;
179: def SourceLanguageOperand : OperandCategory;
180: def AddressingModelOperand : OperandCategory;
181: def ExecutionModelOperand : OperandCategory;
182: def MemoryModelOperand : OperandCategory;
183: def ExecutionModeOperand : OperandCategory;
184: def StorageClassOperand : OperandCategory;
185: def DimOperand : OperandCategory;
186: def SamplerAddressingModeOperand : OperandCategory;
187: def SamplerFilterModeOperand : OperandCategory;
188: def ImageFormatOperand : OperandCategory;
189: def ImageChannelOrderOperand : OperandCategory;
190: def ImageChannelDataTypeOperand : OperandCategory;
191: def ImageOperandOperand : OperandCategory;
192: def FPFastMathModeOperand : OperandCategory;
193: def FPRoundingModeOperand : OperandCategory;
194: def LinkageTypeOperand : OperandCategory;
195: def AccessQualifierOperand : OperandCategory;
196: def FunctionParameterAttributeOperand : OperandCategory;
197: def DecorationOperand : OperandCategory;
198: def BuiltInOperand : OperandCategory;
199: def SelectionControlOperand : OperandCategory;
200: def LoopControlOperand : OperandCategory;
201: def FunctionControlOperand : OperandCategory;
202: def MemorySemanticsOperand : OperandCategory;
203: def MemoryOperandOperand : OperandCategory;
204: def ScopeOperand : OperandCategory;
205: def GroupOperationOperand : OperandCategory;
206: def KernelEnqueueFlagsOperand : OperandCategory;
207: def KernelProfilingInfoOperand : OperandCategory;
208: def OpcodeOperand : OperandCategory;
209: def CooperativeMatrixLayoutOperand : OperandCategory;
210: def CooperativeMatrixOperandsOperand : OperandCategory;
211: def SpecConstantOpOperandsOperand : OperandCategory;
212: def MatrixMultiplyAccumulateOperandsOperand : OperandCategory;
213: def FPEncodingOperand : OperandCategory;
214: def PackedVectorFormatsOperand : OperandCategory;
215:
216: //===----------------------------------------------------------------------===//
217: // Definition of the Environments
218: //===----------------------------------------------------------------------===//
219:
220: def Environment : GenericEnum, Operand<i32> {
221:   let FilterClass = "Environment";
222:   let ValueField = "Value";
223: }
224:
225: class Environment<bits<32> value> { bits<32> Value = value; }
226:
227: def EnvOpenCL : Environment<0>;
228: def EnvVulkan : Environment<1>;
229:
230: //===----------------------------------------------------------------------===//
231: // Multiclass used to define Extesions enum values and at the same time
232: // SymbolicOperand entries.
233: //===----------------------------------------------------------------------===//
234:
235: def Extension : GenericEnum, Operand<i32> {
236:   let FilterClass = "Extension";
237:   let NameField = "Name";
238:   let ValueField = "Value";
239:   let PrintMethod = "printExtension";
240:   let UnderlyingType = "uint32_t";
```
- EN: This range uses TableGen DSL to describe records such as EnvironmentEntry, EnvironmentEntries, SymbolicOperandWithRequirements, OperandCategory; LLVM later expands these declarations into generated backend tables and helper code.
- CN: 这一段使用 TableGen DSL 描述 EnvironmentEntry、EnvironmentEntries、SymbolicOperandWithRequirements、OperandCategory 等记录；LLVM 后续会把这些声明展开为生成的后端表和辅助代码。

### Lines 241-360
```tablegen
241: }
242:
243: class Extension<string name, bits<32> value> {
244:   string Name = name;
245:   bits<32> Value = value;
246: }
247:
248: multiclass ExtensionOperand<bits<32> value,
249:                             list<Environment> allowedEnvironments> {
250:   def NAME : Extension<NAME, value>;
251:   defm : SymbolicOperandWithRequirements<ExtensionOperand, value, NAME, 0,
252:                                          0, [], [], allowedEnvironments>;
253: }
254:
255: defm SPV_AMD_shader_explicit_vertex_parameter
256:     : ExtensionOperand<1, [EnvVulkan]>;
257: defm SPV_AMD_shader_trinary_minmax_extension : ExtensionOperand<2, [EnvVulkan]>;
258: defm SPV_AMD_gcn_shader : ExtensionOperand<3, [EnvVulkan]>;
259: defm SPV_KHR_shader_ballot : ExtensionOperand<4, [EnvVulkan]>;
260: defm SPV_AMD_shader_ballot : ExtensionOperand<5, [EnvVulkan]>;
261: defm SPV_AMD_gpu_shader_half_float : ExtensionOperand<6, [EnvVulkan]>;
262: defm SPV_KHR_shader_draw_parameters : ExtensionOperand<7, [EnvVulkan]>;
263: defm SPV_KHR_subgroup_vote : ExtensionOperand<8, [EnvVulkan]>;
264: defm SPV_KHR_16bit_storage : ExtensionOperand<9, [EnvVulkan]>;
265: defm SPV_KHR_device_group : ExtensionOperand<10, [EnvVulkan]>;
266: defm SPV_KHR_multiview : ExtensionOperand<11, [EnvVulkan]>;
267: defm SPV_NVX_multiview_per_view_attributes : ExtensionOperand<12, [EnvVulkan]>;
268: defm SPV_NV_viewport_array2 : ExtensionOperand<13, [EnvVulkan]>;
269: defm SPV_NV_stereo_view_rendering : ExtensionOperand<14, [EnvVulkan]>;
270: defm SPV_NV_sample_mask_override_coverage : ExtensionOperand<15, [EnvVulkan]>;
271: defm SPV_NV_geometry_shader_passthrough : ExtensionOperand<16, [EnvVulkan]>;
272: defm SPV_AMD_texture_gather_bias_lod : ExtensionOperand<17, [EnvVulkan]>;
273: defm SPV_KHR_storage_buffer_storage_class : ExtensionOperand<18, [EnvVulkan]>;
274: defm SPV_KHR_variable_pointers : ExtensionOperand<19, [EnvVulkan]>;
275: defm SPV_AMD_gpu_shader_int16 : ExtensionOperand<20, [EnvVulkan]>;
276: defm SPV_KHR_post_depth_coverage : ExtensionOperand<21, [EnvVulkan]>;
277: defm SPV_KHR_shader_atomic_counter_ops : ExtensionOperand<22, []>;
278: defm SPV_EXT_shader_stencil_export : ExtensionOperand<23, [EnvVulkan]>;
279: defm SPV_EXT_shader_viewport_index_layer : ExtensionOperand<24, [EnvVulkan]>;
280: defm SPV_AMD_shader_image_load_store_lod : ExtensionOperand<25, [EnvVulkan]>;
281: defm SPV_AMD_shader_fragment_mask : ExtensionOperand<26, [EnvVulkan]>;
282: defm SPV_EXT_fragment_fully_covered : ExtensionOperand<27, [EnvVulkan]>;
283: defm SPV_AMD_gpu_shader_half_float_fetch : ExtensionOperand<28, [EnvVulkan]>;
284: defm SPV_GOOGLE_decorate_string : ExtensionOperand<29, [EnvVulkan]>;
285: defm SPV_GOOGLE_hlsl_functionality1 : ExtensionOperand<30, [EnvVulkan]>;
286: defm SPV_NV_shader_subgroup_partitioned : ExtensionOperand<31, [EnvVulkan]>;
287: defm SPV_EXT_descriptor_indexing : ExtensionOperand<32, [EnvVulkan]>;
288: defm SPV_KHR_8bit_storage : ExtensionOperand<33, [EnvVulkan]>;
289: defm SPV_KHR_vulkan_memory_model : ExtensionOperand<34, [EnvVulkan]>;
290: defm SPV_NV_ray_tracing : ExtensionOperand<35, [EnvVulkan]>;
291: defm SPV_NV_compute_shader_derivatives : ExtensionOperand<36, [EnvVulkan]>;
292: defm SPV_NV_fragment_shader_barycentric : ExtensionOperand<37, [EnvVulkan]>;
293: defm SPV_NV_mesh_shader : ExtensionOperand<38, [EnvVulkan]>;
294: defm SPV_NV_shader_image_footprint : ExtensionOperand<39, [EnvVulkan]>;
295: defm SPV_NV_shading_rate : ExtensionOperand<40, [EnvVulkan]>;
296: defm SPV_INTEL_subgroups : ExtensionOperand<41, [EnvOpenCL]>;
297: defm SPV_INTEL_media_block_io : ExtensionOperand<42, [EnvOpenCL]>;
298: defm SPV_EXT_fragment_invocation_density : ExtensionOperand<44, [EnvVulkan]>;
299: defm SPV_KHR_no_integer_wrap_decoration : ExtensionOperand<45, [EnvOpenCL]>;
300: defm SPV_KHR_float_controls : ExtensionOperand<46, [EnvVulkan, EnvOpenCL]>;
301: defm SPV_EXT_physical_storage_buffer : ExtensionOperand<47, [EnvVulkan]>;
302: defm SPV_INTEL_fpga_memory_attributes : ExtensionOperand<48, [EnvOpenCL]>;
303: defm SPV_NV_cooperative_matrix : ExtensionOperand<49, [EnvVulkan]>;
304: defm SPV_INTEL_shader_integer_functions2
305:     : ExtensionOperand<50, [EnvVulkan, EnvOpenCL]>;
306: defm SPV_INTEL_fpga_loop_controls : ExtensionOperand<51, [EnvOpenCL]>;
307: defm SPV_EXT_fragment_shader_interlock : ExtensionOperand<52, [EnvVulkan]>;
308: defm SPV_NV_shader_sm_builtins : ExtensionOperand<53, [EnvVulkan]>;
309: defm SPV_KHR_shader_clock : ExtensionOperand<54, [EnvVulkan, EnvOpenCL]>;
310: defm SPV_INTEL_unstructured_loop_controls : ExtensionOperand<55, [EnvOpenCL]>;
311: defm SPV_EXT_demote_to_helper_invocation : ExtensionOperand<56, [EnvVulkan]>;
312: defm SPV_INTEL_fpga_reg : ExtensionOperand<57, [EnvOpenCL]>;
313: defm SPV_ALTERA_blocking_pipes : ExtensionOperand<58, [EnvOpenCL]>;
314: defm SPV_GOOGLE_user_type : ExtensionOperand<59, [EnvVulkan]>;
315: defm SPV_KHR_physical_storage_buffer : ExtensionOperand<60, [EnvVulkan]>;
316: defm SPV_INTEL_kernel_attributes : ExtensionOperand<61, [EnvOpenCL]>;
317: defm SPV_KHR_non_semantic_info : ExtensionOperand<62, [EnvVulkan, EnvOpenCL]>;
318: defm SPV_INTEL_io_pipes : ExtensionOperand<63, [EnvOpenCL]>;
319: defm SPV_KHR_ray_tracing : ExtensionOperand<64, [EnvVulkan]>;
320: defm SPV_KHR_ray_query : ExtensionOperand<65, [EnvVulkan]>;
321: defm SPV_INTEL_fpga_memory_accesses : ExtensionOperand<66, [EnvOpenCL]>;
322: defm SPV_ALTERA_arbitrary_precision_integers : ExtensionOperand<67, [EnvOpenCL]>;
323: defm SPV_EXT_shader_atomic_float_add
324:     : ExtensionOperand<68, [EnvVulkan, EnvOpenCL]>;
325: defm SPV_KHR_terminate_invocation : ExtensionOperand<69, [EnvVulkan]>;
326: defm SPV_KHR_fragment_shading_rate : ExtensionOperand<70, [EnvVulkan]>;
327: defm SPV_EXT_shader_image_int64 : ExtensionOperand<71, [EnvVulkan, EnvOpenCL]>;
328: defm SPV_INTEL_fp_fast_math_mode : ExtensionOperand<72, [EnvOpenCL]>;
329: defm SPV_INTEL_fpga_cluster_attributes : ExtensionOperand<73, [EnvOpenCL]>;
330: defm SPV_INTEL_loop_fuse : ExtensionOperand<74, [EnvOpenCL]>;
331: defm SPV_EXT_shader_atomic_float_min_max
332:     : ExtensionOperand<75, [EnvVulkan, EnvOpenCL]>;
333: defm SPV_KHR_workgroup_memory_explicit_layout
334:     : ExtensionOperand<76, [EnvVulkan]>;
335: defm SPV_KHR_linkonce_odr : ExtensionOperand<77, [EnvOpenCL]>;
336: defm SPV_KHR_expect_assume : ExtensionOperand<78, [EnvVulkan, EnvOpenCL]>;
337: defm SPV_INTEL_fpga_dsp_control : ExtensionOperand<79, [EnvOpenCL]>;
338: defm SPV_NV_bindless_texture : ExtensionOperand<80, [EnvVulkan]>;
339: defm SPV_INTEL_fpga_invocation_pipelining_attributes
340:     : ExtensionOperand<81, [EnvOpenCL]>;
341: defm SPV_KHR_subgroup_uniform_control_flow : ExtensionOperand<82, [EnvVulkan]>;
342: defm SPV_HUAWEI_subpass_shading : ExtensionOperand<83, [EnvVulkan]>;
343: defm SPV_KHR_integer_dot_product : ExtensionOperand<84, [EnvVulkan, EnvOpenCL]>;
344: defm SPV_EXT_shader_atomic_float16_add
345:     : ExtensionOperand<85, [EnvVulkan, EnvOpenCL]>;
346: defm SPV_INTEL_runtime_aligned : ExtensionOperand<86, [EnvOpenCL]>;
347: defm SPV_KHR_bit_instructions : ExtensionOperand<87, [EnvOpenCL]>;
348: defm SPV_NV_ray_tracing_motion_blur : ExtensionOperand<88, [EnvVulkan]>;
349: defm SPV_KHR_uniform_group_instructions : ExtensionOperand<89, [EnvOpenCL]>;
350: defm SPV_KHR_subgroup_rotate : ExtensionOperand<90, [EnvVulkan, EnvOpenCL]>;
351: defm SPV_INTEL_split_barrier : ExtensionOperand<91, [EnvOpenCL]>;
352: defm SPV_KHR_ray_cull_mask : ExtensionOperand<92, [EnvVulkan]>;
353: defm SPV_KHR_fragment_shader_barycentric : ExtensionOperand<93, [EnvVulkan]>;
354: defm SPV_EXT_relaxed_printf_string_address_space
355:     : ExtensionOperand<94, [EnvOpenCL]>;
356: defm SPV_EXT_mesh_shader : ExtensionOperand<96, [EnvVulkan]>;
357: defm SPV_ARM_core_builtins : ExtensionOperand<97, [EnvVulkan]>;
358: defm SPV_EXT_opacity_micromap : ExtensionOperand<98, [EnvVulkan]>;
359: defm SPV_NV_shader_invocation_reorder : ExtensionOperand<99, [EnvVulkan]>;
360: defm SPV_INTEL_usm_storage_classes : ExtensionOperand<100, [EnvOpenCL]>;
```
- EN: This range uses TableGen DSL to describe records such as Extension, ExtensionOperand, NAME, SPV_AMD_shader_explicit_vertex_parameter; LLVM later expands these declarations into generated backend tables and helper code.
- CN: 这一段使用 TableGen DSL 描述 Extension、ExtensionOperand、NAME、SPV_AMD_shader_explicit_vertex_parameter 等记录；LLVM 后续会把这些声明展开为生成的后端表和辅助代码。

### Lines 361-480
```tablegen
361: defm SPV_INTEL_fpga_latency_control : ExtensionOperand<101, [EnvOpenCL]>;
362: defm SPV_INTEL_fpga_argument_interfaces : ExtensionOperand<102, [EnvOpenCL]>;
363: defm SPV_INTEL_optnone : ExtensionOperand<103, [EnvOpenCL]>;
364: defm SPV_INTEL_function_pointers : ExtensionOperand<104, [EnvOpenCL]>;
365: defm SPV_INTEL_variable_length_array : ExtensionOperand<105, [EnvOpenCL]>;
366: defm SPV_INTEL_bfloat16_conversion : ExtensionOperand<106, [EnvOpenCL]>;
367: defm SPV_INTEL_inline_assembly : ExtensionOperand<107, [EnvOpenCL]>;
368: defm SPV_INTEL_cache_controls : ExtensionOperand<108, [EnvOpenCL]>;
369: defm SPV_INTEL_global_variable_host_access : ExtensionOperand<109, [EnvOpenCL]>;
370: defm SPV_INTEL_global_variable_fpga_decorations
371:     : ExtensionOperand<110, [EnvOpenCL]>;
372: defm SPV_KHR_cooperative_matrix : ExtensionOperand<111, [EnvVulkan, EnvOpenCL]>;
373: defm SPV_EXT_arithmetic_fence : ExtensionOperand<112, [EnvOpenCL]>;
374: defm SPV_EXT_optnone : ExtensionOperand<113, [EnvOpenCL]>;
375: defm SPV_INTEL_joint_matrix : ExtensionOperand<114, [EnvOpenCL]>;
376: defm SPV_INTEL_float_controls2 : ExtensionOperand<115, [EnvOpenCL]>;
377: defm SPV_INTEL_bindless_images : ExtensionOperand<116, [EnvOpenCL]>;
378: defm SPV_INTEL_long_composites : ExtensionOperand<117, [EnvOpenCL]>;
379: defm SPV_INTEL_memory_access_aliasing : ExtensionOperand<118, [EnvOpenCL]>;
380: defm SPV_INTEL_fp_max_error : ExtensionOperand<119, [EnvOpenCL]>;
381: defm SPV_INTEL_ternary_bitwise_function : ExtensionOperand<120, [EnvOpenCL]>;
382: defm SPV_INTEL_subgroup_matrix_multiply_accumulate
383:     : ExtensionOperand<121, [EnvOpenCL]>;
384: defm SPV_INTEL_2d_block_io : ExtensionOperand<122, [EnvOpenCL]>;
385: defm SPV_INTEL_int4 : ExtensionOperand<123, [EnvOpenCL]>;
386: defm SPV_KHR_float_controls2 : ExtensionOperand<124, [EnvVulkan, EnvOpenCL]>;
387: defm SPV_INTEL_tensor_float32_conversion : ExtensionOperand<125, [EnvOpenCL]>;
388: defm SPV_KHR_bfloat16 : ExtensionOperand<126, [EnvVulkan, EnvOpenCL]>;
389: defm SPV_INTEL_predicated_io : ExtensionOperand<127, [EnvOpenCL]>;
390: defm SPV_KHR_maximal_reconvergence : ExtensionOperand<128, [EnvVulkan]>;
391: defm SPV_INTEL_bfloat16_arithmetic
392:     : ExtensionOperand<129, [EnvVulkan, EnvOpenCL]>;
393: defm SPV_INTEL_16bit_atomics : ExtensionOperand<130, [EnvVulkan, EnvOpenCL]>;
394: defm SPV_ALTERA_arbitrary_precision_fixed_point : ExtensionOperand<131, [EnvOpenCL, EnvVulkan]>;
395: defm SPV_NV_shader_atomic_fp16_vector
396:     : ExtensionOperand<132, [EnvVulkan, EnvOpenCL]>;
397: defm SPV_EXT_image_raw10_raw12 :ExtensionOperand<133, [EnvOpenCL, EnvVulkan]>;
398: defm SPV_ALTERA_arbitrary_precision_floating_point: ExtensionOperand<134, [EnvOpenCL]>;
399: defm SPV_KHR_fma : ExtensionOperand<135, [EnvVulkan, EnvOpenCL]>;
400: defm SPV_INTEL_masked_gather_scatter : ExtensionOperand<136, [EnvOpenCL]>;
401: defm SPV_AMD_weak_linkage : ExtensionOperand<137, [EnvOpenCL]>;
402: // SPV_KHR_abort is currently only supported in the OpenCL environment.
403: // Vulkan/Shader support requires emitting explicit layout decorations on the
404: // Message Type, which is not yet implemented.
405: defm SPV_KHR_abort : ExtensionOperand<138, [EnvOpenCL]>;
406:
407: //===----------------------------------------------------------------------===//
408: // Multiclass used to define Capabilities enum values and at the same time
409: // SymbolicOperand entries with string mnemonics, versioning, extensions, and
410: // capabilities.
411: //===----------------------------------------------------------------------===//
412:
413: def Capability : GenericEnum, Operand<i32> {
414:   let FilterClass = "Capability";
415:   let NameField = "Name";
416:   let ValueField = "Value";
417:   let PrintMethod = !strconcat("printSymbolicOperand<OperandCategory::", FilterClass, "Operand>");
418: }
419:
420: class Capability<string name, bits<32> value> {
421:   string Name = name;
422:   bits<32> Value = value;
423: }
424:
425: multiclass CapabilityOperand<bits<32> value, bits<32> minVersion, bits<32> maxVersion, list<Extension> reqExtensions, list<Capability> reqCapabilities> {
426:   def NAME : Capability<NAME, value>;
427:   defm : SymbolicOperandWithRequirements<CapabilityOperand, value, NAME,
428:                                          minVersion, maxVersion, reqExtensions,
429:                                          reqCapabilities, []>;
430: }
431:
432: defm Matrix : CapabilityOperand<0, 0, 0, [], []>;
433: defm Shader : CapabilityOperand<1, 0, 0, [], [Matrix]>;
434: defm Geometry : CapabilityOperand<2, 0, 0, [], [Shader]>;
435: defm Tessellation : CapabilityOperand<3, 0, 0, [], [Shader]>;
436: defm Addresses : CapabilityOperand<4, 0, 0, [], []>;
437: defm Linkage : CapabilityOperand<5, 0, 0, [], []>;
438: defm Kernel : CapabilityOperand<6, 0, 0, [], []>;
439: defm Vector16 : CapabilityOperand<7, 0, 0, [], [Kernel]>;
440: defm Float16Buffer : CapabilityOperand<8, 0, 0, [], [Kernel]>;
441: defm Float16 : CapabilityOperand<9, 0, 0, [], []>;
442: defm Float64 : CapabilityOperand<10, 0, 0, [], []>;
443: defm Int64 : CapabilityOperand<11, 0, 0, [], []>;
444: defm Int64Atomics : CapabilityOperand<12, 0, 0, [], [Int64]>;
445: defm ImageBasic : CapabilityOperand<13, 0, 0, [], [Kernel]>;
446: defm ImageReadWrite : CapabilityOperand<14, 0, 0, [], [ImageBasic]>;
447: defm ImageMipmap : CapabilityOperand<15, 0, 0, [], [ImageBasic]>;
448: defm Pipes : CapabilityOperand<17, 0, 0, [], [Kernel]>;
449: defm Groups : CapabilityOperand<18, 0, 0, [], []>;
450: defm DeviceEnqueue : CapabilityOperand<19, 0, 0, [], []>;
451: defm LiteralSampler : CapabilityOperand<20, 0, 0, [], [Kernel]>;
452: defm AtomicStorage : CapabilityOperand<21, 0, 0, [], [Shader]>;
453: defm Int16 : CapabilityOperand<22, 0, 0, [], []>;
454: defm TessellationPointSize : CapabilityOperand<23, 0, 0, [], [Tessellation]>;
455: defm GeometryPointSize : CapabilityOperand<24, 0, 0, [], [Geometry]>;
456: defm ImageGatherExtended : CapabilityOperand<25, 0, 0, [], [Shader]>;
457: defm StorageImageMultisample : CapabilityOperand<27, 0, 0, [], [Shader]>;
458: defm UniformBufferArrayDynamicIndexing : CapabilityOperand<28, 0, 0, [], [Shader]>;
459: defm SampledImageArrayDynamicIndexing : CapabilityOperand<29, 0, 0, [], [Shader]>;
460: defm StorageBufferArrayDynamicIndexing : CapabilityOperand<30, 0, 0, [], [Shader]>;
461: defm StorageImageArrayDynamicIndexing : CapabilityOperand<31, 0, 0, [], [Shader]>;
462: defm ClipDistance : CapabilityOperand<32, 0, 0, [], [Shader]>;
463: defm CullDistance : CapabilityOperand<33, 0, 0, [], [Shader]>;
464: defm SampleRateShading : CapabilityOperand<35, 0, 0, [], [Shader]>;
465: defm SampledRect : CapabilityOperand<37, 0, 0, [], [Shader]>;
466: defm ImageRect : CapabilityOperand<36, 0, 0, [], [SampledRect]>;
467: defm GenericPointer : CapabilityOperand<38, 0, 0, [], [Addresses]>;
468: defm Int8 : CapabilityOperand<39, 0, 0, [], []>;
469: defm InputAttachment : CapabilityOperand<40, 0, 0, [], [Shader]>;
470: defm SparseResidency : CapabilityOperand<41, 0, 0, [], [Shader]>;
471: defm MinLod : CapabilityOperand<42, 0, 0, [], [Shader]>;
472: defm Sampled1D : CapabilityOperand<43, 0, 0, [], []>;
473: defm Image1D : CapabilityOperand<44, 0, 0, [], [Sampled1D]>;
474: defm SampledCubeArray : CapabilityOperand<45, 0, 0, [], [Shader]>;
475: defm ImageCubeArray : CapabilityOperand<34, 0, 0, [], [SampledCubeArray]>;
476: defm SampledBuffer : CapabilityOperand<46, 0, 0, [], []>;
477: defm ImageBuffer : CapabilityOperand<47, 0, 0, [], [SampledBuffer]>;
478: defm ImageMSArray : CapabilityOperand<48, 0, 0, [], [Shader]>;
479: defm StorageImageExtendedFormats : CapabilityOperand<49, 0, 0, [], [Shader]>;
480: defm ImageQuery : CapabilityOperand<50, 0, 0, [], [Shader]>;
```
- EN: This range uses TableGen DSL to describe records such as SPV_INTEL_fpga_latency_control, SPV_INTEL_fpga_argument_interfaces, SPV_INTEL_optnone, SPV_INTEL_function_pointers; LLVM later expands these declarations into generated backend tables and helper code.
- CN: 这一段使用 TableGen DSL 描述 SPV_INTEL_fpga_latency_control、SPV_INTEL_fpga_argument_interfaces、SPV_INTEL_optnone、SPV_INTEL_function_pointers 等记录；LLVM 后续会把这些声明展开为生成的后端表和辅助代码。

### Lines 481-600
```tablegen
481: defm DerivativeControl : CapabilityOperand<51, 0, 0, [], [Shader]>;
482: defm InterpolationFunction : CapabilityOperand<52, 0, 0, [], [Shader]>;
483: defm TransformFeedback : CapabilityOperand<53, 0, 0, [], [Shader]>;
484: defm GeometryStreams : CapabilityOperand<54, 0, 0, [], [Geometry]>;
485: defm StorageImageReadWithoutFormat : CapabilityOperand<55, 0, 0, [], [Shader]>;
486: defm StorageImageWriteWithoutFormat : CapabilityOperand<56, 0, 0, [], [Shader]>;
487: defm MultiViewport : CapabilityOperand<57, 0, 0, [], [Geometry]>;
488: defm SubgroupDispatch : CapabilityOperand<58, 0x10100, 0, [], [DeviceEnqueue]>;
489: defm NamedBarrier : CapabilityOperand<59, 0x10100, 0, [], [Kernel]>;
490: defm PipeStorage : CapabilityOperand<60, 0x10100, 0, [], [Pipes]>;
491: defm GroupNonUniform : CapabilityOperand<61, 0x10300, 0, [], []>;
492: defm GroupNonUniformVote : CapabilityOperand<62, 0x10300, 0, [], [GroupNonUniform]>;
493: defm GroupNonUniformArithmetic : CapabilityOperand<63, 0x10300, 0, [], [GroupNonUniform]>;
494: defm GroupNonUniformBallot : CapabilityOperand<64, 0x10300, 0, [], [GroupNonUniform]>;
495: defm GroupNonUniformShuffle : CapabilityOperand<65, 0x10300, 0, [], [GroupNonUniform]>;
496: defm GroupNonUniformShuffleRelative : CapabilityOperand<66, 0x10300, 0, [], [GroupNonUniform]>;
497: defm GroupNonUniformClustered : CapabilityOperand<67, 0x10300, 0, [], [GroupNonUniform]>;
498: defm GroupNonUniformQuad : CapabilityOperand<68, 0x10300, 0, [], [GroupNonUniform]>;
499: defm SubgroupBallotKHR : CapabilityOperand<4423, 0, 0, [SPV_KHR_shader_ballot], []>;
500: defm DrawParameters : CapabilityOperand<4427, 0x10300, 0, [SPV_KHR_shader_draw_parameters], [Shader]>;
501: defm SubgroupVoteKHR : CapabilityOperand<4431, 0, 0, [SPV_KHR_subgroup_vote], []>;
502: defm StorageBuffer16BitAccess : CapabilityOperand<4433, 0x10300, 0, [SPV_KHR_16bit_storage], []>;
503: defm StorageUniform16 : CapabilityOperand<4434, 0x10300, 0, [SPV_KHR_16bit_storage], [StorageBuffer16BitAccess]>;
504: defm StoragePushConstant16 : CapabilityOperand<4435, 0x10300, 0, [SPV_KHR_16bit_storage], []>;
505: defm StorageInputOutput16 : CapabilityOperand<4436, 0x10300, 0, [SPV_KHR_16bit_storage], []>;
506: defm DeviceGroup : CapabilityOperand<4437, 0x10300, 0, [SPV_KHR_device_group], []>;
507: defm MultiView : CapabilityOperand<4439, 0x10300, 0, [SPV_KHR_multiview], [Shader]>;
508: defm VariablePointersStorageBuffer : CapabilityOperand<4441, 0x10300, 0, [SPV_KHR_variable_pointers], [Shader]>;
509: defm VariablePointers : CapabilityOperand<4442, 0x10300, 0, [SPV_KHR_variable_pointers], [VariablePointersStorageBuffer]>;
510: defm AtomicStorageOps : CapabilityOperand<4445, 0, 0, [SPV_KHR_shader_atomic_counter_ops], []>;
511: defm SampleMaskPostDepthCoverage : CapabilityOperand<4447, 0, 0, [SPV_KHR_post_depth_coverage], []>;
512: defm StorageBuffer8BitAccess : CapabilityOperand<4448, 0, 0, [SPV_KHR_8bit_storage], []>;
513: defm UniformAndStorageBuffer8BitAccess : CapabilityOperand<4449, 0, 0, [SPV_KHR_8bit_storage], [StorageBuffer8BitAccess]>;
514: defm StoragePushConstant8 : CapabilityOperand<4450, 0, 0, [SPV_KHR_8bit_storage], []>;
515: defm DenormPreserve : CapabilityOperand<4464, 0x10400, 0, [SPV_KHR_float_controls], []>;
516: defm DenormFlushToZero : CapabilityOperand<4465, 0x10400, 0, [SPV_KHR_float_controls], []>;
517: defm SignedZeroInfNanPreserve : CapabilityOperand<4466, 0x10400, 0, [SPV_KHR_float_controls], []>;
518: defm RoundingModeRTE : CapabilityOperand<4467, 0x10400, 0, [SPV_KHR_float_controls], []>;
519: defm RoundingModeRTZ : CapabilityOperand<4468, 0x10400, 0, [SPV_KHR_float_controls], []>;
520: defm Float16ImageAMD : CapabilityOperand<5008, 0, 0, [], [Shader]>;
521: defm ImageGatherBiasLodAMD : CapabilityOperand<5009, 0, 0, [], [Shader]>;
522: defm FragmentMaskAMD : CapabilityOperand<5010, 0, 0, [], [Shader]>;
523: defm StencilExportEXT : CapabilityOperand<5013, 0, 0, [], [Shader]>;
524: defm ImageReadWriteLodAMD : CapabilityOperand<5015, 0, 0, [], [Shader]>;
525: defm Int64ImageEXT : CapabilityOperand<5016, 0, 0, [SPV_EXT_shader_image_int64], [Int64]>;
526: defm ShaderClockKHR : CapabilityOperand<5055, 0, 0, [SPV_KHR_shader_clock], []>;
527: defm BFloat16TypeKHR : CapabilityOperand<5116, 0, 0, [SPV_KHR_bfloat16], []>;
528: defm SampleMaskOverrideCoverageNV : CapabilityOperand<5249, 0, 0, [], [SampleRateShading]>;
529: defm GeometryShaderPassthroughNV : CapabilityOperand<5251, 0, 0, [], [Geometry]>;
530: defm ShaderViewportIndexLayerEXT : CapabilityOperand<5254, 0, 0, [], [MultiViewport]>;
531: defm ShaderViewportMaskNV : CapabilityOperand<5255, 0, 0, [], [ShaderViewportIndexLayerEXT]>;
532: defm ShaderStereoViewNV : CapabilityOperand<5259, 0, 0, [], [ShaderViewportMaskNV]>;
533: defm PerViewAttributesNV : CapabilityOperand<5260, 0, 0, [], [MultiView]>;
534: defm FragmentFullyCoveredEXT : CapabilityOperand<5265, 0, 0, [], [Shader]>;
535: defm MeshShadingNV : CapabilityOperand<5266, 0, 0, [], [Shader]>;
536: defm ShaderNonUniformEXT : CapabilityOperand<5301, 0, 0, [], [Shader]>;
537: defm RuntimeDescriptorArrayEXT : CapabilityOperand<5302, 0, 0, [], [Shader]>;
538: defm InputAttachmentArrayDynamicIndexingEXT : CapabilityOperand<5303, 0, 0, [], [InputAttachment]>;
539: defm UniformTexelBufferArrayDynamicIndexingEXT : CapabilityOperand<5304, 0, 0, [], [SampledBuffer]>;
540: defm StorageTexelBufferArrayDynamicIndexingEXT : CapabilityOperand<5305, 0, 0, [], [ImageBuffer]>;
541: defm UniformBufferArrayNonUniformIndexingEXT : CapabilityOperand<5306, 0, 0, [], [ShaderNonUniformEXT]>;
542: defm SampledImageArrayNonUniformIndexingEXT : CapabilityOperand<5307, 0, 0, [], [ShaderNonUniformEXT]>;
543: defm StorageBufferArrayNonUniformIndexingEXT : CapabilityOperand<5308, 0, 0, [], [ShaderNonUniformEXT]>;
544: defm StorageImageArrayNonUniformIndexingEXT : CapabilityOperand<5309, 0, 0, [], [ShaderNonUniformEXT]>;
545: defm InputAttachmentArrayNonUniformIndexingEXT : CapabilityOperand<5310, 0, 0, [], [InputAttachment, ShaderNonUniformEXT]>;
546: defm UniformTexelBufferArrayNonUniformIndexingEXT : CapabilityOperand<5311, 0, 0, [], [SampledBuffer, ShaderNonUniformEXT]>;
547: defm StorageTexelBufferArrayNonUniformIndexingEXT : CapabilityOperand<5312, 0, 0, [], [ImageBuffer, ShaderNonUniformEXT]>;
548: defm RayTracingNV : CapabilityOperand<5340, 0, 0, [], [Shader]>;
549: defm SubgroupShuffleINTEL : CapabilityOperand<5568, 0, 0, [SPV_INTEL_subgroups], []>;
550: defm SubgroupBufferBlockIOINTEL : CapabilityOperand<5569, 0, 0, [SPV_INTEL_subgroups], []>;
551: defm SubgroupImageBlockIOINTEL : CapabilityOperand<5570, 0, 0, [SPV_INTEL_subgroups], []>;
552: defm SubgroupImageMediaBlockIOINTEL : CapabilityOperand<5579, 0, 0, [SPV_INTEL_media_block_io], []>;
553: defm SubgroupAvcMotionEstimationINTEL : CapabilityOperand<5696, 0, 0, [], []>;
554: defm SubgroupAvcMotionEstimationIntraINTEL : CapabilityOperand<5697, 0, 0, [], []>;
555: defm SubgroupAvcMotionEstimationChromaINTEL : CapabilityOperand<5698, 0, 0, [], []>;
556: defm GroupNonUniformPartitionedNV : CapabilityOperand<5297, 0, 0, [], []>;
557: defm VulkanMemoryModelKHR : CapabilityOperand<5345, 0, 0, [], []>;
558: defm VulkanMemoryModelDeviceScopeKHR : CapabilityOperand<5346, 0, 0, [], []>;
559: defm ImageFootprintNV : CapabilityOperand<5282, 0, 0, [], []>;
560: defm FragmentBarycentricNV : CapabilityOperand<5284, 0, 0, [], []>;
561: defm ComputeDerivativeGroupQuadsNV : CapabilityOperand<5288, 0, 0, [], []>;
562: defm DemoteToHelperInvocation : CapabilityOperand<5379, 0x10600, 0, [SPV_EXT_demote_to_helper_invocation], []>;
563: defm ComputeDerivativeGroupLinearNV : CapabilityOperand<5350, 0, 0, [], []>;
564: defm FragmentDensityEXT : CapabilityOperand<5291, 0, 0, [], [Shader]>;
565: defm PhysicalStorageBufferAddressesEXT : CapabilityOperand<5347, 0, 0, [], [Shader]>;
566: defm CooperativeMatrixNV : CapabilityOperand<5357, 0, 0, [], [Shader]>;
567: defm ArbitraryPrecisionIntegersALTERA : CapabilityOperand<5844, 0, 0, [SPV_ALTERA_arbitrary_precision_integers], [Int8, Int16]>;
568: defm OptNoneINTEL : CapabilityOperand<6094, 0, 0, [SPV_INTEL_optnone], []>;
569: defm OptNoneEXT : CapabilityOperand<6094, 0, 0, [SPV_EXT_optnone], []>;
570: defm BitInstructions : CapabilityOperand<6025, 0, 0, [SPV_KHR_bit_instructions], []>;
571: defm ExpectAssumeKHR : CapabilityOperand<5629, 0, 0, [SPV_KHR_expect_assume], []>;
572: defm FunctionPointersINTEL : CapabilityOperand<5603, 0, 0, [SPV_INTEL_function_pointers], []>;
573: defm IndirectReferencesINTEL : CapabilityOperand<5604, 0, 0, [SPV_INTEL_function_pointers], []>;
574: defm AsmINTEL : CapabilityOperand<5606, 0, 0, [SPV_INTEL_inline_assembly], []>;
575: defm DotProductInputAll : CapabilityOperand<6016, 0x10600, 0, [SPV_KHR_integer_dot_product], []>;
576: defm DotProductInput4x8Bit : CapabilityOperand<6017, 0x10600, 0, [SPV_KHR_integer_dot_product], [Int8]>;
577: defm DotProductInput4x8BitPacked : CapabilityOperand<6018, 0x10600, 0, [SPV_KHR_integer_dot_product], []>;
578: defm DotProduct : CapabilityOperand<6019, 0x10600, 0, [SPV_KHR_integer_dot_product], []>;
579: defm GroupNonUniformRotateKHR : CapabilityOperand<6026, 0, 0, [SPV_KHR_subgroup_rotate], [GroupNonUniform]>;
580: defm FloatControls2
581:     : CapabilityOperand<6029, 0x10200, 0, [SPV_KHR_float_controls2], []>;
582: defm AtomicFloat32AddEXT : CapabilityOperand<6033, 0, 0, [SPV_EXT_shader_atomic_float_add], []>;
583: defm AtomicFloat64AddEXT : CapabilityOperand<6034, 0, 0, [SPV_EXT_shader_atomic_float_add], []>;
584: defm FmaKHR : CapabilityOperand<6030, 0, 0, [SPV_KHR_fma], []>;
585: defm AtomicFloat16AddEXT : CapabilityOperand<6095, 0, 0, [SPV_EXT_shader_atomic_float16_add], []>;
586: defm AtomicBFloat16AddINTEL : CapabilityOperand<6255, 0, 0, [SPV_INTEL_16bit_atomics], [BFloat16TypeKHR]>;
587: defm AtomicFloat16MinMaxEXT : CapabilityOperand<5616, 0, 0, [SPV_EXT_shader_atomic_float_min_max], []>;
588: defm AtomicFloat32MinMaxEXT : CapabilityOperand<5612, 0, 0, [SPV_EXT_shader_atomic_float_min_max], []>;
589: defm AtomicFloat64MinMaxEXT : CapabilityOperand<5613, 0, 0, [SPV_EXT_shader_atomic_float_min_max], []>;
590: defm AtomicBFloat16MinMaxINTEL : CapabilityOperand<6256, 0, 0, [SPV_INTEL_16bit_atomics], [BFloat16TypeKHR]>;
591: defm AtomicFloat16VectorNV : CapabilityOperand<5404, 0, 0, [SPV_NV_shader_atomic_fp16_vector], []>;
592: defm AtomicInt16CompareExchangeINTEL : CapabilityOperand<6260, 0, 0, [SPV_INTEL_16bit_atomics], [Int16]>;
593: defm Int16AtomicsINTEL : CapabilityOperand<6261, 0, 0, [SPV_INTEL_16bit_atomics], [AtomicInt16CompareExchangeINTEL]>;
594: defm AtomicBFloat16LoadStoreINTEL : CapabilityOperand<6262, 0, 0, [SPV_INTEL_16bit_atomics], [BFloat16TypeKHR]>;
595: defm VariableLengthArrayINTEL : CapabilityOperand<5817, 0, 0, [SPV_INTEL_variable_length_array], []>;
596: defm GroupUniformArithmeticKHR : CapabilityOperand<6400, 0, 0, [SPV_KHR_uniform_group_instructions], []>;
597: defm USMStorageClassesINTEL : CapabilityOperand<5935, 0, 0, [SPV_INTEL_usm_storage_classes], [Kernel]>;
598: defm BFloat16ArithmeticINTEL : CapabilityOperand<6226, 0, 0, [SPV_INTEL_bfloat16_arithmetic], []>;
599: defm BFloat16ConversionINTEL : CapabilityOperand<6115, 0, 0, [SPV_INTEL_bfloat16_conversion], []>;
600: defm GlobalVariableHostAccessINTEL : CapabilityOperand<6187, 0, 0, [SPV_INTEL_global_variable_host_access], []>;
```
- EN: This range uses TableGen DSL to describe records such as DerivativeControl, InterpolationFunction, TransformFeedback, GeometryStreams; LLVM later expands these declarations into generated backend tables and helper code.
- CN: 这一段使用 TableGen DSL 描述 DerivativeControl、InterpolationFunction、TransformFeedback、GeometryStreams 等记录；LLVM 后续会把这些声明展开为生成的后端表和辅助代码。

### Lines 601-720
```tablegen
601: defm HostAccessINTEL : CapabilityOperand<6188, 0, 0, [SPV_INTEL_global_variable_host_access], []>;
602: defm GlobalVariableFPGADecorationsINTEL : CapabilityOperand<6189, 0, 0, [SPV_INTEL_global_variable_fpga_decorations], []>;
603: defm CacheControlsINTEL : CapabilityOperand<6441, 0, 0, [SPV_INTEL_cache_controls], []>;
604: defm CooperativeMatrixKHR : CapabilityOperand<6022, 0, 0, [SPV_KHR_cooperative_matrix], []>;
605: defm ArithmeticFenceEXT : CapabilityOperand<6144, 0, 0, [SPV_EXT_arithmetic_fence], []>;
606: defm AbortKHR : CapabilityOperand<5120, 0, 0, [SPV_KHR_abort], []>;
607: defm SplitBarrierINTEL : CapabilityOperand<6141, 0, 0, [SPV_INTEL_split_barrier], []>;
608: defm CooperativeMatrixCheckedInstructionsINTEL : CapabilityOperand<6192, 0, 0, [SPV_INTEL_joint_matrix], []>;
609: defm CooperativeMatrixPrefetchINTEL : CapabilityOperand<6411, 0, 0, [SPV_INTEL_joint_matrix], []>;
610: defm PackedCooperativeMatrixINTEL : CapabilityOperand<6434, 0, 0, [SPV_INTEL_joint_matrix], []>;
611: defm CooperativeMatrixInvocationInstructionsINTEL : CapabilityOperand<6435, 0, 0, [SPV_INTEL_joint_matrix], []>;
612: defm CooperativeMatrixTF32ComponentTypeINTEL : CapabilityOperand<6436, 0, 0, [SPV_INTEL_joint_matrix], []>;
613: defm CooperativeMatrixBFloat16ComponentTypeINTEL : CapabilityOperand<6437, 0, 0, [SPV_INTEL_joint_matrix], []>;
614: defm RoundToInfinityINTEL : CapabilityOperand<5582, 0, 0, [SPV_INTEL_float_controls2], []>;
615: defm FloatingPointModeINTEL : CapabilityOperand<5583, 0, 0, [SPV_INTEL_float_controls2], []>;
616: defm FunctionFloatControlINTEL : CapabilityOperand<5821, 0, 0, [SPV_INTEL_float_controls2], []>;
617: defm KernelAttributesINTEL : CapabilityOperand<5892, 0, 0, [SPV_INTEL_kernel_attributes], [Kernel]>;
618: // TODO-SPIRV: add these once they are used / tested.
619: // defm FPGAKernelAttributesINTEL : CapabilityOperand<5897, 0, 0, [SPV_INTEL_kernel_attributes], [Kernel]>;
620: // defm FPGAKernelAttributesv2INTEL : CapabilityOperand<6161, 0, 0, [SPV_INTEL_kernel_attributes], [Kernel]>;
621: // END TODO-SPIRV
622: defm LongCompositesINTEL : CapabilityOperand<6089, 0, 0, [SPV_INTEL_long_composites], []>;
623: defm BindlessImagesINTEL : CapabilityOperand<6528, 0, 0, [SPV_INTEL_bindless_images], []>;
624: defm MemoryAccessAliasingINTEL : CapabilityOperand<5910, 0, 0, [SPV_INTEL_memory_access_aliasing], []>;
625: defm FPMaxErrorINTEL : CapabilityOperand<6169, 0, 0, [SPV_INTEL_fp_max_error], []>;
626: defm TernaryBitwiseFunctionINTEL : CapabilityOperand<6241, 0, 0, [SPV_INTEL_ternary_bitwise_function], []>;
627: defm SubgroupMatrixMultiplyAccumulateINTEL : CapabilityOperand<6236, 0, 0, [SPV_INTEL_subgroup_matrix_multiply_accumulate], []>;
628: defm Subgroup2DBlockIOINTEL : CapabilityOperand<6228, 0, 0, [SPV_INTEL_2d_block_io], []>;
629: defm Subgroup2DBlockTransformINTEL : CapabilityOperand<6229, 0, 0, [SPV_INTEL_2d_block_io], [Subgroup2DBlockIOINTEL]>;
630: defm Subgroup2DBlockTransposeINTEL : CapabilityOperand<6230, 0, 0, [SPV_INTEL_2d_block_io], [Subgroup2DBlockIOINTEL]>;
631: defm PredicatedIOINTEL : CapabilityOperand<6257, 0, 0, [SPV_INTEL_predicated_io], []>;
632: defm Int4TypeINTEL : CapabilityOperand<5112, 0, 0, [SPV_INTEL_int4], []>;
633: defm Int4CooperativeMatrixINTEL : CapabilityOperand<5114, 0, 0, [SPV_INTEL_int4], [Int4TypeINTEL, CooperativeMatrixKHR]>;
634: defm TensorFloat32RoundingINTEL : CapabilityOperand<6425, 0, 0, [SPV_INTEL_tensor_float32_conversion], []>;
635: defm MaskedGatherScatterINTEL : CapabilityOperand<6427, 0, 0, [SPV_INTEL_masked_gather_scatter], []>;
636: defm BFloat16DotProductKHR : CapabilityOperand<5117, 0, 0, [SPV_KHR_bfloat16], [BFloat16TypeKHR]>;
637: defm BFloat16CooperativeMatrixKHR : CapabilityOperand<5118, 0, 0, [SPV_KHR_bfloat16], [BFloat16TypeKHR, CooperativeMatrixKHR]>;
638: defm WeakLinkageAMD : CapabilityOperand<5181, 0, 0, [SPV_AMD_weak_linkage], [Linkage]>;
639: defm BlockingPipesALTERA : CapabilityOperand<5945, 0, 0, [SPV_ALTERA_blocking_pipes], []>;
640: defm ArbitraryPrecisionFixedPointALTERA : CapabilityOperand<5922, 0, 0, [SPV_ALTERA_arbitrary_precision_fixed_point], []>;
641: defm ArbitraryPrecisionFloatingPointALTERA : CapabilityOperand<5845, 0, 0,[SPV_ALTERA_arbitrary_precision_floating_point], []>;
642: defm UnstructuredLoopControlsINTEL : CapabilityOperand<5886, 0, 0, [SPV_INTEL_unstructured_loop_controls], []>;
643:
644: //===----------------------------------------------------------------------===//
645: // Multiclass used to define SourceLanguage enum values and at the same time
646: // SymbolicOperand entries.
647: //===----------------------------------------------------------------------===//
648:
649: def SourceLanguage : GenericEnum, Operand<i32> {
650:   let FilterClass = "SourceLanguage";
651:   let NameField = "Name";
652:   let ValueField = "Value";
653:   let PrintMethod = !strconcat("printSymbolicOperand<OperandCategory::", FilterClass, "Operand>");
654: }
655:
656: class SourceLanguage<string name, bits<32> value> {
657:   string Name = name;
658:   bits<32> Value = value;
659: }
660:
661: multiclass SourceLanguageOperand<bits<32> value> {
662:   def : SourceLanguage<NAME, value>;
663:   defm : SymbolicOperandWithRequirements<SourceLanguageOperand, value, NAME, 0,
664:                                          0, [], [], []>;
665: }
666:
667: defm Unknown : SourceLanguageOperand<0>;
668: defm ESSL : SourceLanguageOperand<1>;
669: defm GLSL : SourceLanguageOperand<2>;
670: defm OpenCL_C : SourceLanguageOperand<3>;
671: defm OpenCL_CPP : SourceLanguageOperand<4>;
672: defm HLSL : SourceLanguageOperand<5>;
673: defm CPP_for_OpenCL : SourceLanguageOperand<6>;
674:
675: //===----------------------------------------------------------------------===//
676: // Multiclass used to define AddressingModel enum values and at the same time
677: // SymbolicOperand entries with string mnemonics, and capabilities.
678: //===----------------------------------------------------------------------===//
679:
680: def AddressingModel : GenericEnum, Operand<i32> {
681:   let FilterClass = "AddressingModel";
682:   let NameField = "Name";
683:   let ValueField = "Value";
684:   let PrintMethod = !strconcat("printSymbolicOperand<OperandCategory::", FilterClass, "Operand>");
685: }
686:
687: class AddressingModel<string name, bits<32> value> {
688:   string Name = name;
689:   bits<32> Value = value;
690: }
691:
692: multiclass AddressingModelOperand<bits<32> value, list<Capability> reqCapabilities> {
693:   def : AddressingModel<NAME, value>;
694:   defm : SymbolicOperandWithRequirements<AddressingModelOperand, value, NAME, 0,
695:                                          0, [], reqCapabilities, []>;
696: }
697:
698: defm Logical : AddressingModelOperand<0, []>;
699: defm Physical32 : AddressingModelOperand<1, [Addresses]>;
700: defm Physical64 : AddressingModelOperand<2, [Addresses]>;
701: defm PhysicalStorageBuffer64EXT : AddressingModelOperand<5348, [PhysicalStorageBufferAddressesEXT]>;
702:
703: //===----------------------------------------------------------------------===//
704: // Multiclass used to define ExecutionModel enum values and at the same time
705: // SymbolicOperand entries with string mnemonics and capabilities.
706: //===----------------------------------------------------------------------===//
707:
708: def ExecutionModel : GenericEnum, Operand<i32> {
709:   let FilterClass = "ExecutionModel";
710:   let NameField = "Name";
711:   let ValueField = "Value";
712:   let PrintMethod = !strconcat("printSymbolicOperand<OperandCategory::", FilterClass, "Operand>");
713: }
714:
715: class ExecutionModel<string name, bits<32> value> {
716:   string Name = name;
717:   bits<32> Value = value;
718: }
719:
720: multiclass ExecutionModelOperand<bits<32> value, list<Capability> reqCapabilities> {
```
- EN: This range uses TableGen DSL to describe records such as HostAccessINTEL, GlobalVariableFPGADecorationsINTEL, CacheControlsINTEL, CooperativeMatrixKHR; LLVM later expands these declarations into generated backend tables and helper code.
- CN: 这一段使用 TableGen DSL 描述 HostAccessINTEL、GlobalVariableFPGADecorationsINTEL、CacheControlsINTEL、CooperativeMatrixKHR 等记录；LLVM 后续会把这些声明展开为生成的后端表和辅助代码。

### Lines 721-840
```tablegen
721:   def : ExecutionModel<NAME, value>;
722:   defm : SymbolicOperandWithRequirements<ExecutionModelOperand, value, NAME, 0,
723:                                          0, [], reqCapabilities, []>;
724: }
725:
726: defm Vertex : ExecutionModelOperand<0, [Shader]>;
727: defm TessellationControl: ExecutionModelOperand<1, [Tessellation]>;
728: defm TessellationEvaluation: ExecutionModelOperand<2, [Tessellation]>;
729: defm Geometry: ExecutionModelOperand<3, [Geometry]>;
730: defm Fragment: ExecutionModelOperand<4, [Shader]>;
731: defm GLCompute: ExecutionModelOperand<5, [Shader]>;
732: defm Kernel: ExecutionModelOperand<6, [Kernel]>;
733: defm TaskNV: ExecutionModelOperand<5267, [MeshShadingNV]>;
734: defm MeshNV: ExecutionModelOperand<5268, [MeshShadingNV]>;
735: defm RayGenerationNV: ExecutionModelOperand<5313, [RayTracingNV]>;
736: defm IntersectionNV: ExecutionModelOperand<5314, [RayTracingNV]>;
737: defm AnyHitNV: ExecutionModelOperand<5315, [RayTracingNV]>;
738: defm ClosestHitNV: ExecutionModelOperand<5316, [RayTracingNV]>;
739: defm MissNV: ExecutionModelOperand<5317, [RayTracingNV]>;
740: defm CallableNV : ExecutionModelOperand<5318, [RayTracingNV]>;
741:
742: //===----------------------------------------------------------------------===//
743: // Multiclass used to define MemoryModel enum values and at the same time
744: // SymbolicOperand entries with string mnemonics and capabilities.
745: //===----------------------------------------------------------------------===//
746:
747: def MemoryModel : GenericEnum, Operand<i32> {
748:   let FilterClass = "MemoryModel";
749:   let NameField = "Name";
750:   let ValueField = "Value";
751:   let PrintMethod = !strconcat("printSymbolicOperand<OperandCategory::", FilterClass, "Operand>");
752: }
753:
754: class MemoryModel<string name, bits<32> value> {
755:   string Name = name;
756:   bits<32> Value = value;
757: }
758:
759: multiclass MemoryModelOperand<bits<32> value, list<Capability> reqCapabilities> {
760:   def : MemoryModel<NAME, value>;
761:   defm : SymbolicOperandWithRequirements<MemoryModelOperand, value, NAME, 0,
762:                                          0, [], reqCapabilities, []>;
763: }
764:
765: defm Simple : MemoryModelOperand<0, [Shader]>;
766: defm GLSL450 : MemoryModelOperand<1, [Shader]>;
767: defm OpenCL : MemoryModelOperand<2, [Kernel]>;
768: defm VulkanKHR : MemoryModelOperand<3, [VulkanMemoryModelKHR]>;
769:
770: //===----------------------------------------------------------------------===//
771: // Multiclass used to define ExecutionMode enum values and at the same time
772: // SymbolicOperand entries with string mnemonics and capabilities.
773: //===----------------------------------------------------------------------===//
774:
775: def ExecutionMode : GenericEnum, Operand<i32> {
776:   let FilterClass = "ExecutionMode";
777:   let NameField = "Name";
778:   let ValueField = "Value";
779:   let PrintMethod = !strconcat("printSymbolicOperand<OperandCategory::", FilterClass, "Operand>");
780: }
781:
782: class ExecutionMode<string name, bits<32> value> {
783:   string Name = name;
784:   bits<32> Value = value;
785: }
786:
787: multiclass ExecutionModeOperand<bits<32> value, list<Capability> reqCapabilities> {
788:   def : ExecutionMode<NAME, value>;
789:   defm : SymbolicOperandWithRequirements<ExecutionModeOperand, value, NAME, 0,
790:                                          0, [], reqCapabilities, []>;
791: }
792:
793: defm Invocations : ExecutionModeOperand<0, [Geometry]>;
794: defm SpacingEqual : ExecutionModeOperand<1, [Tessellation]>;
795: defm SpacingFractionalEven : ExecutionModeOperand<2, [Tessellation]>;
796: defm SpacingFractionalOdd : ExecutionModeOperand<3, [Tessellation]>;
797: defm VertexOrderCw : ExecutionModeOperand<4, [Tessellation]>;
798: defm VertexOrderCcw : ExecutionModeOperand<5, [Tessellation]>;
799: defm PixelCenterInteger : ExecutionModeOperand<6, [Shader]>;
800: defm OriginUpperLeft : ExecutionModeOperand<7, [Shader]>;
801: defm OriginLowerLeft : ExecutionModeOperand<8, [Shader]>;
802: defm EarlyFragmentTests : ExecutionModeOperand<9, [Shader]>;
803: defm PointMode : ExecutionModeOperand<10, [Tessellation]>;
804: defm Xfb : ExecutionModeOperand<11, [TransformFeedback]>;
805: defm DepthReplacing : ExecutionModeOperand<12, [Shader]>;
806: defm DepthGreater : ExecutionModeOperand<14, [Shader]>;
807: defm DepthLess : ExecutionModeOperand<15, [Shader]>;
808: defm DepthUnchanged : ExecutionModeOperand<16, [Shader]>;
809: defm LocalSize : ExecutionModeOperand<17, []>;
810: defm LocalSizeHint : ExecutionModeOperand<18, [Kernel]>;
811: defm InputPoints : ExecutionModeOperand<19, [Geometry]>;
812: defm InputLines : ExecutionModeOperand<20, [Geometry]>;
813: defm InputLinesAdjacency : ExecutionModeOperand<21, [Geometry]>;
814: defm Triangles : ExecutionModeOperand<22, [Geometry]>;
815: defm InputTrianglesAdjacency : ExecutionModeOperand<23, [Geometry]>;
816: defm Quads : ExecutionModeOperand<24, [Tessellation]>;
817: defm Isolines : ExecutionModeOperand<25, [Tessellation]>;
818: defm OutputVertices : ExecutionModeOperand<26, [Geometry]>;
819: defm OutputPoints : ExecutionModeOperand<27, [Geometry]>;
820: defm OutputLineStrip : ExecutionModeOperand<28, [Geometry]>;
821: defm OutputTriangleStrip : ExecutionModeOperand<29, [Geometry]>;
822: defm VecTypeHint : ExecutionModeOperand<30, [Kernel]>;
823: defm ContractionOff : ExecutionModeOperand<31, [Kernel]>;
824: defm Initializer : ExecutionModeOperand<33, [Kernel]>;
825: defm Finalizer : ExecutionModeOperand<34, [Kernel]>;
826: defm SubgroupSize : ExecutionModeOperand<35, [SubgroupDispatch]>;
827: defm SubgroupsPerWorkgroup : ExecutionModeOperand<36, [SubgroupDispatch]>;
828: defm SubgroupsPerWorkgroupId : ExecutionModeOperand<37, [SubgroupDispatch]>;
829: defm LocalSizeId : ExecutionModeOperand<38, []>;
830: defm LocalSizeHintId : ExecutionModeOperand<39, [Kernel]>;
831: defm PostDepthCoverage : ExecutionModeOperand<4446, [SampleMaskPostDepthCoverage]>;
832: defm DenormPreserve : ExecutionModeOperand<4459, [DenormPreserve]>;
833: defm DenormFlushToZero : ExecutionModeOperand<4460, [DenormFlushToZero]>;
834: defm SignedZeroInfNanPreserve : ExecutionModeOperand<4461, [SignedZeroInfNanPreserve]>;
835: defm RoundingModeRTE : ExecutionModeOperand<4462, [RoundingModeRTE]>;
836: defm RoundingModeRTZ : ExecutionModeOperand<4463, [RoundingModeRTZ]>;
837: defm StencilRefReplacingEXT : ExecutionModeOperand<5027, [StencilExportEXT]>;
838: defm OutputLinesNV : ExecutionModeOperand<5269, [MeshShadingNV]>;
839: defm DerivativeGroupQuadsNV : ExecutionModeOperand<5289, [ComputeDerivativeGroupQuadsNV]>;
840: defm DerivativeGroupLinearNV : ExecutionModeOperand<5290, [ComputeDerivativeGroupLinearNV]>;
```
- EN: This range uses TableGen DSL to describe records such as Vertex, TessellationControl, TessellationEvaluation, Geometry; LLVM later expands these declarations into generated backend tables and helper code.
- CN: 这一段使用 TableGen DSL 描述 Vertex、TessellationControl、TessellationEvaluation、Geometry 等记录；LLVM 后续会把这些声明展开为生成的后端表和辅助代码。

### Lines 841-960
```tablegen
841: defm OutputTrianglesNV : ExecutionModeOperand<5298, [MeshShadingNV]>;
842: defm RoundingModeRTPINTEL : ExecutionModeOperand<5620, [RoundToInfinityINTEL]>;
843: defm RoundingModeRTNINTEL : ExecutionModeOperand<5621, [RoundToInfinityINTEL]>;
844: defm FloatingPointModeALTINTEL : ExecutionModeOperand<5622, [FloatingPointModeINTEL]>;
845: defm FloatingPointModeIEEEINTEL : ExecutionModeOperand<5623, [FloatingPointModeINTEL]>;
846: defm MaxWorkgroupSizeINTEL : ExecutionModeOperand<5893, [KernelAttributesINTEL]>;
847: // TODO-SPIRV: Add the following once they are used / tested.
848: // defm MaxWorkDimINTEL : ExecutionModeOperand<5894, [KernelAttributesINTEL]>;
849: // defm NoGlobalOffsetINTEL : ExecutionModeOperand<5895, [KernelAttributesINTEL]>;
850: // defm NumSIMDWorkitemsINTEL : ExecutionModeOperand<5896, [FPGAKernelAttributesINTEL]>;
851: // defm SchedulerTargetFmaxMhzINTEL : ExecutionModeOperand<5903, [FPGAKernelAttributesINTEL]>;
852: // defm StreamingInterfaceINTEL : ExecutionModeOperand<6154, [FPGAKernelAttributesv2INTEL]>;
853: // defm RegisterMapInterfaceINTEL : ExecutionModeOperand<6160, [FPGAKernelAttributesv2INTEL]>;
854: // END TODO-SPIRV
855: defm FPFastMathDefault : ExecutionModeOperand<6028, [FloatControls2]>;
856: defm MaximallyReconvergesKHR : ExecutionModeOperand<6023, [Shader]>;
857:
858: //===----------------------------------------------------------------------===//
859: // Multiclass used to define StorageClass enum values and at the same time
860: // SymbolicOperand entries with string mnemonics and capabilities.
861: //===----------------------------------------------------------------------===//
862:
863: def StorageClass : GenericEnum, Operand<i32> {
864:   let FilterClass = "StorageClass";
865:   let NameField = "Name";
866:   let ValueField = "Value";
867:   let PrintMethod = !strconcat("printSymbolicOperand<OperandCategory::", FilterClass, "Operand>");
868: }
869:
870: class StorageClass<string name, bits<32> value> {
871:   string Name = name;
872:   bits<32> Value = value;
873: }
874:
875: multiclass StorageClassOperand<bits<32> value, list<Extension> reqExtensions, list<Capability> reqCapabilities> {
876:   def : StorageClass<NAME, value>;
877:   defm : SymbolicOperandWithRequirements<StorageClassOperand, value, NAME, 0, 0,
878:                                          reqExtensions, reqCapabilities, []>;
879: }
880:
881: defm UniformConstant : StorageClassOperand<0, [], []>;
882: defm Input : StorageClassOperand<1, [], []>;
883: defm Uniform : StorageClassOperand<2, [], [Shader]>;
884: defm Output : StorageClassOperand<3, [], [Shader]>;
885: defm Workgroup : StorageClassOperand<4, [], []>;
886: defm CrossWorkgroup : StorageClassOperand<5, [], []>;
887: defm Private : StorageClassOperand<6, [], [Shader]>;
888: defm Function : StorageClassOperand<7, [], []>;
889: defm Generic : StorageClassOperand<8, [], [GenericPointer]>;
890: defm PushConstant : StorageClassOperand<9, [], [Shader]>;
891: defm AtomicCounter : StorageClassOperand<10, [], [AtomicStorage]>;
892: defm Image : StorageClassOperand<11, [], []>;
893: defm StorageBuffer : StorageClassOperand<12, [], [Shader]>;
894: defm CallableDataNV : StorageClassOperand<5328, [], [RayTracingNV]>;
895: defm IncomingCallableDataNV : StorageClassOperand<5329, [], [RayTracingNV]>;
896: defm RayPayloadNV : StorageClassOperand<5338, [], [RayTracingNV]>;
897: defm HitAttributeNV : StorageClassOperand<5339, [], [RayTracingNV]>;
898: defm IncomingRayPayloadNV : StorageClassOperand<5342, [], [RayTracingNV]>;
899: defm ShaderRecordBufferNV : StorageClassOperand<5343, [], [RayTracingNV]>;
900: defm PhysicalStorageBufferEXT : StorageClassOperand<5349, [], [PhysicalStorageBufferAddressesEXT]>;
901: defm CodeSectionINTEL : StorageClassOperand<5605, [SPV_INTEL_function_pointers], [FunctionPointersINTEL]>;
902: defm DeviceOnlyINTEL : StorageClassOperand<5936, [SPV_INTEL_usm_storage_classes], [USMStorageClassesINTEL]>;
903: defm HostOnlyINTEL : StorageClassOperand<5937, [SPV_INTEL_usm_storage_classes], [USMStorageClassesINTEL]>;
904:
905: //===----------------------------------------------------------------------===//
906: // Multiclass used to define Dim enum values and at the same time
907: // SymbolicOperand entries with string mnemonics and capabilities.
908: //===----------------------------------------------------------------------===//
909:
910: def Dim : GenericEnum, Operand<i32> {
911:   let FilterClass = "Dim";
912:   let NameField = "Name";
913:   let ValueField = "Value";
914:   let PrintMethod = !strconcat("printSymbolicOperand<OperandCategory::", FilterClass, "Operand>");
915: }
916:
917: class Dim<string name, bits<32> value> {
918:   string Name = name;
919:   bits<32> Value = value;
920: }
921:
922: multiclass DimOperand<bits<32> value, string mnemonic, list<Capability> reqCapabilities> {
923:   def NAME : Dim<NAME, value>;
924:   defm : SymbolicOperandWithRequirements<DimOperand, value, mnemonic, 0, 0, [],
925:                                          reqCapabilities, []>;
926: }
927:
928: defm DIM_1D : DimOperand<0, "1D", [Sampled1D, Image1D]>;
929: defm DIM_2D : DimOperand<1, "2D", [Shader, Kernel, ImageMSArray]>;
930: defm DIM_3D : DimOperand<2, "3D", []>;
931: defm DIM_Cube : DimOperand<3, "Cube", [Shader, ImageCubeArray]>;
932: defm DIM_Rect : DimOperand<4, "Rect", [SampledRect, ImageRect]>;
933: defm DIM_Buffer : DimOperand<5, "Buffer", [SampledBuffer, ImageBuffer]>;
934: defm DIM_SubpassData : DimOperand<6, "SubpassData", [InputAttachment]>;
935:
936: //===----------------------------------------------------------------------===//
937: // Multiclass used to define SamplerAddressingMode enum values and at the same
938: // time SymbolicOperand entries with string mnemonics and capabilities.
939: //===----------------------------------------------------------------------===//
940:
941: def SamplerAddressingMode : GenericEnum, Operand<i32> {
942:   let FilterClass = "SamplerAddressingMode";
943:   let NameField = "Name";
944:   let ValueField = "Value";
945:   let PrintMethod = !strconcat("printSymbolicOperand<OperandCategory::", FilterClass, "Operand>");
946: }
947:
948: class SamplerAddressingMode<string name, bits<32> value> {
949:   string Name = name;
950:   bits<32> Value = value;
951: }
952:
953: multiclass SamplerAddressingModeOperand<bits<32> value, list<Capability> reqCapabilities> {
954:   def : SamplerAddressingMode<NAME, value>;
955:   defm : SymbolicOperandWithRequirements<SamplerAddressingModeOperand, value,
956:                                          NAME, 0, 0, [], reqCapabilities, []>;
957: }
958:
959: defm None : SamplerAddressingModeOperand<0, [Kernel]>;
960: defm ClampToEdge : SamplerAddressingModeOperand<1, [Kernel]>;
```
- EN: This range uses TableGen DSL to describe records such as OutputTrianglesNV, RoundingModeRTPINTEL, RoundingModeRTNINTEL, FloatingPointModeALTINTEL; LLVM later expands these declarations into generated backend tables and helper code.
- CN: 这一段使用 TableGen DSL 描述 OutputTrianglesNV、RoundingModeRTPINTEL、RoundingModeRTNINTEL、FloatingPointModeALTINTEL 等记录；LLVM 后续会把这些声明展开为生成的后端表和辅助代码。

### Lines 961-1080
```tablegen
 961: defm Clamp : SamplerAddressingModeOperand<2, [Kernel]>;
 962: defm Repeat : SamplerAddressingModeOperand<3, [Kernel]>;
 963: defm RepeatMirrored : SamplerAddressingModeOperand<4, [Kernel]>;
 964:
 965: //===----------------------------------------------------------------------===//
 966: // Multiclass used to define SamplerFilterMode enum values and at the same
 967: // time SymbolicOperand entries with string mnemonics and capabilities.
 968: //===----------------------------------------------------------------------===//
 969:
 970: def SamplerFilterMode : GenericEnum, Operand<i32> {
 971:   let FilterClass = "SamplerFilterMode";
 972:   let NameField = "Name";
 973:   let ValueField = "Value";
 974:   let PrintMethod = !strconcat("printSymbolicOperand<OperandCategory::", FilterClass, "Operand>");
 975: }
 976:
 977: class SamplerFilterMode<string name, bits<32> value> {
 978:   string Name = name;
 979:   bits<32> Value = value;
 980: }
 981:
 982: multiclass SamplerFilterModeOperand<bits<32> value, list<Capability> reqCapabilities> {
 983:   def : SamplerFilterMode<NAME, value>;
 984:   defm : SymbolicOperandWithRequirements<SamplerFilterModeOperand, value, NAME,
 985:                                          0, 0, [], reqCapabilities, []>;
 986: }
 987:
 988: defm Nearest : SamplerFilterModeOperand<0, [Kernel]>;
 989: defm Linear : SamplerFilterModeOperand<1, [Kernel]>;
 990:
 991: //===----------------------------------------------------------------------===//
 992: // Multiclass used to define ImageFormat enum values and at the same time
 993: // SymbolicOperand entries with string mnemonics and capabilities.
 994: //===----------------------------------------------------------------------===//
 995:
 996: def ImageFormat : GenericEnum, Operand<i32> {
 997:   let FilterClass = "ImageFormat";
 998:   let NameField = "Name";
 999:   let ValueField = "Value";
1000:   let PrintMethod = !strconcat("printSymbolicOperand<OperandCategory::", FilterClass, "Operand>");
1001: }
1002:
1003: class ImageFormat<string name, bits<32> value> {
1004:   string Name = name;
1005:   bits<32> Value = value;
1006: }
1007:
1008: multiclass ImageFormatOperand<bits<32> value, list<Capability> reqCapabilities> {
1009:   def NAME : ImageFormat<NAME, value>;
1010:   defm : SymbolicOperandWithRequirements<ImageFormatOperand, value, NAME, 0,
1011:                                          0, [], reqCapabilities, []>;
1012: }
1013:
1014: defm Unknown : ImageFormatOperand<0, []>;
1015: defm Rgba32f : ImageFormatOperand<1, [Shader]>;
1016: defm Rgba16f : ImageFormatOperand<2, [Shader]>;
1017: defm R32f : ImageFormatOperand<3, [Shader]>;
1018: defm Rgba8 : ImageFormatOperand<4, [Shader]>;
1019: defm Rgba8Snorm : ImageFormatOperand<5, [Shader]>;
1020: defm Rg32f : ImageFormatOperand<6, [StorageImageExtendedFormats]>;
1021: defm Rg16f : ImageFormatOperand<7, [StorageImageExtendedFormats]>;
1022: defm R11fG11fB10f : ImageFormatOperand<8, [StorageImageExtendedFormats]>;
1023: defm R16f : ImageFormatOperand<9, [StorageImageExtendedFormats]>;
1024: defm Rgba16 : ImageFormatOperand<10, [StorageImageExtendedFormats]>;
1025: defm Rgb10A2 : ImageFormatOperand<11, [StorageImageExtendedFormats]>;
1026: defm Rg16 : ImageFormatOperand<12, [StorageImageExtendedFormats]>;
1027: defm Rg8 : ImageFormatOperand<13, [StorageImageExtendedFormats]>;
1028: defm R16 : ImageFormatOperand<14, [StorageImageExtendedFormats]>;
1029: defm R8 : ImageFormatOperand<15, [StorageImageExtendedFormats]>;
1030: defm Rgba16Snorm : ImageFormatOperand<16, [StorageImageExtendedFormats]>;
1031: defm Rg16Snorm : ImageFormatOperand<17, [StorageImageExtendedFormats]>;
1032: defm Rg8Snorm : ImageFormatOperand<18, [StorageImageExtendedFormats]>;
1033: defm R16Snorm : ImageFormatOperand<19, [StorageImageExtendedFormats]>;
1034: defm R8Snorm : ImageFormatOperand<20, [StorageImageExtendedFormats]>;
1035: defm Rgba32i : ImageFormatOperand<21, [Shader]>;
1036: defm Rgba16i : ImageFormatOperand<22, [Shader]>;
1037: defm Rgba8i : ImageFormatOperand<23, [Shader]>;
1038: defm R32i : ImageFormatOperand<24, [Shader]>;
1039: defm Rg32i : ImageFormatOperand<25, [StorageImageExtendedFormats]>;
1040: defm Rg16i : ImageFormatOperand<26, [StorageImageExtendedFormats]>;
1041: defm Rg8i : ImageFormatOperand<27, [StorageImageExtendedFormats]>;
1042: defm R16i : ImageFormatOperand<28, [StorageImageExtendedFormats]>;
1043: defm R8i : ImageFormatOperand<29, [StorageImageExtendedFormats]>;
1044: defm Rgba32ui : ImageFormatOperand<30, [Shader]>;
1045: defm Rgba16ui : ImageFormatOperand<31, [Shader]>;
1046: defm Rgba8ui : ImageFormatOperand<32, [Shader]>;
1047: defm R32ui : ImageFormatOperand<33, [Shader]>;
1048: defm Rgb10a2ui : ImageFormatOperand<34, [StorageImageExtendedFormats]>;
1049: defm Rg32ui : ImageFormatOperand<35, [StorageImageExtendedFormats]>;
1050: defm Rg16ui : ImageFormatOperand<36, [StorageImageExtendedFormats]>;
1051: defm Rg8ui : ImageFormatOperand<37, [StorageImageExtendedFormats]>;
1052: defm R16ui : ImageFormatOperand<38, [StorageImageExtendedFormats]>;
1053: defm R8ui : ImageFormatOperand<39, [StorageImageExtendedFormats]>;
1054:
1055: //===----------------------------------------------------------------------===//
1056: // Multiclass used to define ImageChannelOrder enum values and at the same time
1057: // SymbolicOperand entries with string mnemonics and capabilities.
1058: //===----------------------------------------------------------------------===//
1059:
1060: def ImageChannelOrder : GenericEnum, Operand<i32> {
1061:   let FilterClass = "ImageChannelOrder";
1062:   let NameField = "Name";
1063:   let ValueField = "Value";
1064:   let PrintMethod = !strconcat("printSymbolicOperand<OperandCategory::", FilterClass, "Operand>");
1065: }
1066:
1067: class ImageChannelOrder<string name, bits<32> value> {
1068:   string Name = name;
1069:   bits<32> Value = value;
1070: }
1071:
1072: multiclass ImageChannelOrderOperand<bits<32> value, list<Capability> reqCapabilities> {
1073:   def : ImageChannelOrder<NAME, value>;
1074:   defm : SymbolicOperandWithRequirements<ImageChannelOrderOperand, value, NAME,
1075:                                          0, 0, [], reqCapabilities, []>;
1076: }
1077:
1078: defm R : ImageChannelOrderOperand<0, [Kernel]>;
1079: defm A : ImageChannelOrderOperand<1, [Kernel]>;
1080: defm RG : ImageChannelOrderOperand<2, [Kernel]>;
```
- EN: This range uses TableGen DSL to describe records such as Clamp, Repeat, RepeatMirrored, SamplerFilterMode; LLVM later expands these declarations into generated backend tables and helper code.
- CN: 这一段使用 TableGen DSL 描述 Clamp、Repeat、RepeatMirrored、SamplerFilterMode 等记录；LLVM 后续会把这些声明展开为生成的后端表和辅助代码。

### Lines 1081-1200
```tablegen
1081: defm RA : ImageChannelOrderOperand<3, [Kernel]>;
1082: defm RGB : ImageChannelOrderOperand<4, [Kernel]>;
1083: defm RGBA : ImageChannelOrderOperand<5, [Kernel]>;
1084: defm BGRA : ImageChannelOrderOperand<6, [Kernel]>;
1085: defm ARGB : ImageChannelOrderOperand<7, [Kernel]>;
1086: defm Intensity : ImageChannelOrderOperand<8, [Kernel]>;
1087: defm Luminance : ImageChannelOrderOperand<9, [Kernel]>;
1088: defm Rx : ImageChannelOrderOperand<10, [Kernel]>;
1089: defm RGx : ImageChannelOrderOperand<11, [Kernel]>;
1090: defm RGBx : ImageChannelOrderOperand<12, [Kernel]>;
1091: defm Depth : ImageChannelOrderOperand<13, [Kernel]>;
1092: defm DepthStencil : ImageChannelOrderOperand<14, [Kernel]>;
1093: defm sRGB : ImageChannelOrderOperand<15, [Kernel]>;
1094: defm sRGBx : ImageChannelOrderOperand<16, [Kernel]>;
1095: defm sRGBA : ImageChannelOrderOperand<17, [Kernel]>;
1096: defm sBGRA : ImageChannelOrderOperand<18, [Kernel]>;
1097: defm ABGR : ImageChannelOrderOperand<19, [Kernel]>;
1098:
1099: //===----------------------------------------------------------------------===//
1100: // Multiclass used to define ImageChannelDataType enum values and at the same
1101: // time SymbolicOperand entries with string mnemonics and capabilities.
1102: //===----------------------------------------------------------------------===//
1103:
1104: def ImageChannelDataType : GenericEnum, Operand<i32> {
1105:   let FilterClass = "ImageChannelDataType";
1106:   let NameField = "Name";
1107:   let ValueField = "Value";
1108:   let PrintMethod = !strconcat("printSymbolicOperand<OperandCategory::", FilterClass, "Operand>");
1109: }
1110:
1111: class ImageChannelDataType<string name, bits<32> value> {
1112:   string Name = name;
1113:   bits<32> Value = value;
1114: }
1115:
1116: multiclass ImageChannelDataTypeOperand<bits<32> value, list<Capability> reqCapabilities> {
1117:   def : ImageChannelDataType<NAME, value>;
1118:   defm : SymbolicOperandWithRequirements<ImageChannelDataTypeOperand, value,
1119:                                          NAME, 0, 0, [], reqCapabilities, []>;
1120: }
1121:
1122: defm SnormInt8 : ImageChannelDataTypeOperand<0, []>;
1123: defm SnormInt16 : ImageChannelDataTypeOperand<1, []>;
1124: defm UnormInt8 : ImageChannelDataTypeOperand<2, [Kernel]>;
1125: defm UnormInt16 : ImageChannelDataTypeOperand<3, [Kernel]>;
1126: defm UnormShort565 : ImageChannelDataTypeOperand<4, [Kernel]>;
1127: defm UnormShort555 : ImageChannelDataTypeOperand<5, [Kernel]>;
1128: defm UnormInt101010 : ImageChannelDataTypeOperand<6, [Kernel]>;
1129: defm SignedInt8 : ImageChannelDataTypeOperand<7, [Kernel]>;
1130: defm SignedInt16 : ImageChannelDataTypeOperand<8, [Kernel]>;
1131: defm SignedInt32 : ImageChannelDataTypeOperand<9, [Kernel]>;
1132: defm UnsignedInt8 : ImageChannelDataTypeOperand<10, [Kernel]>;
1133: defm UnsignedInt16 : ImageChannelDataTypeOperand<11, [Kernel]>;
1134: defm UnsigendInt32 : ImageChannelDataTypeOperand<12, [Kernel]>;
1135: defm HalfFloat : ImageChannelDataTypeOperand<13, [Kernel]>;
1136: defm Float : ImageChannelDataTypeOperand<14, [Kernel]>;
1137: defm UnormInt24 : ImageChannelDataTypeOperand<15, [Kernel]>;
1138: defm UnormInt101010_2 : ImageChannelDataTypeOperand<16, [Kernel]>;
1139: defm UnsignedIntRaw10EXT : ImageChannelDataTypeOperand<17, [Kernel]>;
1140: defm UnsignedIntRaw12EXT : ImageChannelDataTypeOperand<18, [Kernel]>;
1141:
1142: //===----------------------------------------------------------------------===//
1143: // Multiclass used to define ImageOperand enum values and at the same time
1144: // SymbolicOperand entries with string mnemonics and capabilities.
1145: //===----------------------------------------------------------------------===//
1146:
1147: def ImageOperand : GenericEnum, Operand<i32> {
1148:   let FilterClass = "ImageOperand";
1149:   let NameField = "Name";
1150:   let ValueField = "Value";
1151:   let PrintMethod = !strconcat("printSymbolicOperand<OperandCategory::", FilterClass, "Operand>");
1152: }
1153:
1154: class ImageOperand<string name, bits<32> value> {
1155:   string Name = name;
1156:   bits<32> Value = value;
1157: }
1158:
1159: multiclass ImageOperandOperand<bits<32> value, list<Capability> reqCapabilities> {
1160:   def : ImageOperand<NAME, value>;
1161:   defm : SymbolicOperandWithRequirements<ImageOperandOperand, value, NAME, 0,
1162:                                          0, [], reqCapabilities, []>;
1163: }
1164:
1165: defm None : ImageOperandOperand<0x0, []>;
1166: defm Bias : ImageOperandOperand<0x1, [Shader]>;
1167: defm Lod : ImageOperandOperand<0x2, []>;
1168: defm Grad : ImageOperandOperand<0x4, []>;
1169: defm ConstOffset : ImageOperandOperand<0x8, []>;
1170: defm Offset : ImageOperandOperand<0x10, [ImageGatherExtended]>;
1171: defm ConstOffsets : ImageOperandOperand<0x20, [ImageGatherExtended]>;
1172: defm Sample : ImageOperandOperand<0x40, []>;
1173: defm MinLod : ImageOperandOperand<0x80, [MinLod]>;
1174: defm MakeTexelAvailableKHR : ImageOperandOperand<0x100, [VulkanMemoryModelKHR]>;
1175: defm MakeTexelVisibleKHR : ImageOperandOperand<0x200, [VulkanMemoryModelKHR]>;
1176: defm NonPrivateTexelKHR : ImageOperandOperand<0x400, [VulkanMemoryModelKHR]>;
1177: defm VolatileTexelKHR : ImageOperandOperand<0x800, [VulkanMemoryModelKHR]>;
1178: defm SignExtend : ImageOperandOperand<0x1000, []>;
1179: defm ZeroExtend : ImageOperandOperand<0x2000, []>;
1180:
1181: //===----------------------------------------------------------------------===//
1182: // Multiclass used to define FPFastMathMode enum values and at the same time
1183: // SymbolicOperand entries with string mnemonics and capabilities.
1184: //===----------------------------------------------------------------------===//
1185:
1186: def FPFastMathMode : GenericEnum, Operand<i32> {
1187:   let FilterClass = "FPFastMathMode";
1188:   let NameField = "Name";
1189:   let ValueField = "Value";
1190:   let PrintMethod = !strconcat("printSymbolicOperand<OperandCategory::", FilterClass, "Operand>");
1191: }
1192:
1193: class FPFastMathMode<string name, bits<32> value> {
1194:   string Name = name;
1195:   bits<32> Value = value;
1196: }
1197:
1198: multiclass FPFastMathModeOperand<bits<32> value, list<Capability> reqCapabilities> {
1199:   def : FPFastMathMode<NAME, value>;
1200:   defm : SymbolicOperandWithRequirements<FPFastMathModeOperand, value, NAME, 0,
```
- EN: This range uses TableGen DSL to describe records such as RA, RGB, RGBA, BGRA; LLVM later expands these declarations into generated backend tables and helper code.
- CN: 这一段使用 TableGen DSL 描述 RA、RGB、RGBA、BGRA 等记录；LLVM 后续会把这些声明展开为生成的后端表和辅助代码。

### Lines 1201-1320
```tablegen
1201:                                          0, [], reqCapabilities, []>;
1202: }
1203:
1204: defm None : FPFastMathModeOperand<0x0, []>;
1205: defm NotNaN : FPFastMathModeOperand<0x1, [Kernel]>;
1206: defm NotInf : FPFastMathModeOperand<0x2, [Kernel]>;
1207: defm NSZ : FPFastMathModeOperand<0x4, [Kernel]>;
1208: defm AllowRecip : FPFastMathModeOperand<0x8, [Kernel]>;
1209: defm Fast : FPFastMathModeOperand<0x10, [Kernel]>;
1210: defm AllowContract : FPFastMathModeOperand<0x10000, [FloatControls2]>;
1211: defm AllowReassoc : FPFastMathModeOperand<0x20000, [FloatControls2]>;
1212: defm AllowTransform : FPFastMathModeOperand<0x40000, [FloatControls2]>;
1213:
1214: //===----------------------------------------------------------------------===//
1215: // Multiclass used to define FPRoundingMode enum values and at the same time
1216: // SymbolicOperand entries with string mnemonics.
1217: //===----------------------------------------------------------------------===//
1218:
1219: def FPRoundingMode : GenericEnum, Operand<i32> {
1220:   let FilterClass = "FPRoundingMode";
1221:   let NameField = "Name";
1222:   let ValueField = "Value";
1223:   let PrintMethod = !strconcat("printSymbolicOperand<OperandCategory::", FilterClass, "Operand>");
1224: }
1225:
1226: class FPRoundingMode<string name, bits<32> value> {
1227:   string Name = name;
1228:   bits<32> Value = value;
1229: }
1230:
1231: multiclass FPRoundingModeOperand<bits<32> value> {
1232:   def NAME : FPRoundingMode<NAME, value>;
1233:   defm : SymbolicOperandWithRequirements<FPRoundingModeOperand, value, NAME, 0,
1234:                                          0, [], [], []>;
1235: }
1236:
1237: defm RTE : FPRoundingModeOperand<0>;
1238: defm RTZ : FPRoundingModeOperand<1>;
1239: defm RTP : FPRoundingModeOperand<2>;
1240: defm RTN : FPRoundingModeOperand<3>;
1241:
1242: //===----------------------------------------------------------------------===//
1243: // Multiclass used to define LinkageType enum values and at the same time
1244: // SymbolicOperand entries with string mnemonics and capabilities.
1245: //===----------------------------------------------------------------------===//
1246:
1247: def LinkageType : GenericEnum, Operand<i32> {
1248:   let FilterClass = "LinkageType";
1249:   let NameField = "Name";
1250:   let ValueField = "Value";
1251:   let PrintMethod = !strconcat("printSymbolicOperand<OperandCategory::", FilterClass, "Operand>");
1252: }
1253:
1254: class LinkageType<string name, bits<32> value> {
1255:   string Name = name;
1256:   bits<32> Value = value;
1257: }
1258:
1259: multiclass LinkageTypeOperand<bits<32> value, list<Capability> reqCapabilities> {
1260:   def : LinkageType<NAME, value>;
1261:   defm : SymbolicOperandWithRequirements<LinkageTypeOperand, value, NAME, 0,
1262:                                          0, [], reqCapabilities, []>;
1263: }
1264:
1265: defm Export : LinkageTypeOperand<0, [Linkage]>;
1266: defm Import : LinkageTypeOperand<1, [Linkage]>;
1267: defm LinkOnceODR : LinkageTypeOperand<2, [Linkage]>;
1268: defm WeakAMD : LinkageTypeOperand<3, [WeakLinkageAMD]>;
1269:
1270: //===----------------------------------------------------------------------===//
1271: // Multiclass used to define AccessQualifier enum values and at the same time
1272: // SymbolicOperand entries with string mnemonics and capabilities.
1273: //===----------------------------------------------------------------------===//
1274:
1275: def AccessQualifier : GenericEnum, Operand<i32> {
1276:   let FilterClass = "AccessQualifier";
1277:   let NameField = "Name";
1278:   let ValueField = "Value";
1279:   let PrintMethod = !strconcat("printSymbolicOperand<OperandCategory::", FilterClass, "Operand>");
1280: }
1281:
1282: class AccessQualifier<string name, bits<32> value> {
1283:   string Name = name;
1284:   bits<32> Value = value;
1285: }
1286:
1287: multiclass AccessQualifierOperand<bits<32> value, list<Capability> reqCapabilities> {
1288:   def NAME : AccessQualifier<NAME, value>;
1289:   defm : SymbolicOperandWithRequirements<AccessQualifierOperand, value, NAME, 0,
1290:                                          0, [], reqCapabilities, []>;
1291: }
1292:
1293: defm ReadOnly : AccessQualifierOperand<0, [Kernel]>;
1294: defm WriteOnly : AccessQualifierOperand<1, [Kernel]>;
1295: defm ReadWrite : AccessQualifierOperand<2, [Kernel]>;
1296: defm None : AccessQualifierOperand<3, []>;
1297:
1298: //===----------------------------------------------------------------------===//
1299: // Multiclass used to define FunctionParameterAttribute enum values and at the
1300: // same time SymbolicOperand entries with string mnemonics and capabilities.
1301: //===----------------------------------------------------------------------===//
1302:
1303: def FunctionParameterAttribute : GenericEnum, Operand<i32> {
1304:   let FilterClass = "FunctionParameterAttribute";
1305:   let NameField = "Name";
1306:   let ValueField = "Value";
1307:   let PrintMethod = !strconcat("printSymbolicOperand<OperandCategory::", FilterClass, "Operand>");
1308: }
1309:
1310: class FunctionParameterAttribute<string name, bits<32> value> {
1311:   string Name = name;
1312:   bits<32> Value = value;
1313: }
1314:
1315: multiclass FunctionParameterAttributeOperand<bits<32> value, list<Capability> reqCapabilities> {
1316:   def : FunctionParameterAttribute<NAME, value>;
1317:   defm : SymbolicOperandWithRequirements<FunctionParameterAttributeOperand,
1318:                                          value, NAME, 0, 0, [],
1319:                                          reqCapabilities, []>;
1320: }
```
- EN: This range uses TableGen DSL to describe records such as None, NotNaN, NotInf, NSZ; LLVM later expands these declarations into generated backend tables and helper code.
- CN: 这一段使用 TableGen DSL 描述 None、NotNaN、NotInf、NSZ 等记录；LLVM 后续会把这些声明展开为生成的后端表和辅助代码。

### Lines 1321-1440
```tablegen
1321:
1322: defm Zext : FunctionParameterAttributeOperand<0, [Kernel]>;
1323: defm Sext : FunctionParameterAttributeOperand<1, [Kernel]>;
1324: defm ByVal : FunctionParameterAttributeOperand<2, [Kernel]>;
1325: defm Sret : FunctionParameterAttributeOperand<3, [Kernel]>;
1326: defm NoAlias : FunctionParameterAttributeOperand<4, [Kernel]>;
1327: defm NoCapture : FunctionParameterAttributeOperand<5, [Kernel]>;
1328: defm NoWrite : FunctionParameterAttributeOperand<6, [Kernel]>;
1329: defm NoReadWrite : FunctionParameterAttributeOperand<7, [Kernel]>;
1330:
1331: //===----------------------------------------------------------------------===//
1332: // Multiclass used to define Decoration enum values and at the same time
1333: // SymbolicOperand entries with string mnemonics, versioning, extensions and
1334: // capabilities.
1335: //===----------------------------------------------------------------------===//
1336:
1337: def Decoration : GenericEnum, Operand<i32> {
1338:   let FilterClass = "Decoration";
1339:   let NameField = "Name";
1340:   let ValueField = "Value";
1341:   let PrintMethod = !strconcat("printSymbolicOperand<OperandCategory::", FilterClass, "Operand>");
1342: }
1343:
1344: class Decoration<string name, bits<32> value> {
1345:   string Name = name;
1346:   bits<32> Value = value;
1347: }
1348:
1349: multiclass DecorationOperand<bits<32> value, bits<32> minVersion, bits<32> maxVersion, list<Extension> reqExtensions, list<Capability> reqCapabilities> {
1350:   def : Decoration<NAME, value>;
1351:   defm : SymbolicOperandWithRequirements<DecorationOperand, value, NAME,
1352:                                          minVersion, maxVersion, reqExtensions,
1353:                                          reqCapabilities, []>;
1354: }
1355:
1356: defm RelaxedPrecision : DecorationOperand<0, 0, 0, [], [Shader]>;
1357: defm SpecId : DecorationOperand<1, 0, 0, [], [Shader, Kernel]>;
1358: defm Block : DecorationOperand<2, 0, 0, [], [Shader]>;
1359: defm BufferBlock : DecorationOperand<3, 0, 0, [], [Shader]>;
1360: defm RowMajor : DecorationOperand<4, 0, 0, [], [Matrix]>;
1361: defm ColMajor : DecorationOperand<5, 0, 0, [], [Matrix]>;
1362: defm ArrayStride : DecorationOperand<6, 0, 0, [], [Shader]>;
1363: defm MatrixStride : DecorationOperand<7, 0, 0, [], [Matrix]>;
1364: defm GLSLShared : DecorationOperand<8, 0, 0, [], [Shader]>;
1365: defm GLSLPacked : DecorationOperand<9, 0, 0, [], [Shader]>;
1366: defm CPacked : DecorationOperand<10, 0, 0, [], [Kernel]>;
1367: defm BuiltIn : DecorationOperand<11, 0, 0, [], []>;
1368: defm NoPerspective : DecorationOperand<13, 0, 0, [], [Shader]>;
1369: defm Flat : DecorationOperand<14, 0, 0, [], [Shader]>;
1370: defm Patch : DecorationOperand<15, 0, 0, [], [Tessellation]>;
1371: defm Centroid : DecorationOperand<16, 0, 0, [], [Shader]>;
1372: defm Sample : DecorationOperand<17, 0, 0, [], [SampleRateShading]>;
1373: defm Invariant : DecorationOperand<18, 0, 0, [], [Shader]>;
1374: defm Restrict : DecorationOperand<19, 0, 0, [], []>;
1375: defm Aliased : DecorationOperand<20, 0, 0, [], []>;
1376: defm Volatile : DecorationOperand<21, 0, 0, [], []>;
1377: defm Constant : DecorationOperand<22, 0, 0, [], [Kernel]>;
1378: defm Coherent : DecorationOperand<23, 0, 0, [], []>;
1379: defm NonWritable : DecorationOperand<24, 0, 0, [], []>;
1380: defm NonReadable : DecorationOperand<25, 0, 0, [], []>;
1381: defm Uniform : DecorationOperand<26, 0, 0, [], [Shader]>;
1382: defm UniformId : DecorationOperand<27, 0, 0, [], [Shader]>;
1383: defm SaturatedConversion : DecorationOperand<28, 0, 0, [], [Kernel]>;
1384: defm Stream : DecorationOperand<29, 0, 0, [], [GeometryStreams]>;
1385: defm Location : DecorationOperand<30, 0, 0, [], [Shader]>;
1386: defm Component : DecorationOperand<31, 0, 0, [], [Shader]>;
1387: defm Index : DecorationOperand<32, 0, 0, [], [Shader]>;
1388: defm Binding : DecorationOperand<33, 0, 0, [], [Shader]>;
1389: defm DescriptorSet : DecorationOperand<34, 0, 0, [], [Shader]>;
1390: defm Offset : DecorationOperand<35, 0, 0, [], [Shader]>;
1391: defm XfbBuffer : DecorationOperand<36, 0, 0, [], [TransformFeedback]>;
1392: defm XfbStride : DecorationOperand<37, 0, 0, [], [TransformFeedback]>;
1393: defm FuncParamAttr : DecorationOperand<38, 0, 0, [], [Kernel]>;
1394: defm FPRoundingMode : DecorationOperand<39, 0, 0, [], []>;
1395: defm FPFastMathMode : DecorationOperand<40, 0, 0, [], [Kernel, FloatControls2]>;
1396: defm LinkageAttributes : DecorationOperand<41, 0, 0, [], [Linkage]>;
1397: defm NoContraction : DecorationOperand<42, 0, 0, [], [Shader]>;
1398: defm InputAttachmentIndex : DecorationOperand<43, 0, 0, [], [InputAttachment]>;
1399: defm Alignment : DecorationOperand<44, 0, 0, [], [Kernel]>;
1400: defm MaxByteOffset : DecorationOperand<45, 0, 0, [], [Addresses]>;
1401: defm AlignmentId : DecorationOperand<46, 0, 0, [], [Kernel]>;
1402: defm MaxByteOffsetId : DecorationOperand<47, 0, 0, [], [Addresses]>;
1403: defm NoSignedWrap : DecorationOperand<4469, 0x10400, 0, [SPV_KHR_no_integer_wrap_decoration], []>;
1404: defm NoUnsignedWrap : DecorationOperand<4470, 0x10400, 0, [SPV_KHR_no_integer_wrap_decoration], []>;
1405: defm ExplicitInterpAMD : DecorationOperand<4999, 0, 0, [], []>;
1406: defm OverrideCoverageNV : DecorationOperand<5248, 0, 0, [], [SampleMaskOverrideCoverageNV]>;
1407: defm PassthroughNV : DecorationOperand<5250, 0, 0, [], [GeometryShaderPassthroughNV]>;
1408: defm ViewportRelativeNV : DecorationOperand<5252, 0, 0, [], [ShaderViewportMaskNV]>;
1409: defm SecondaryViewportRelativeNV : DecorationOperand<5256, 0, 0, [], [ShaderStereoViewNV]>;
1410: defm PerPrimitiveNV : DecorationOperand<5271, 0, 0, [], [MeshShadingNV]>;
1411: defm PerViewNV : DecorationOperand<5272, 0, 0, [], [MeshShadingNV]>;
1412: defm PerVertexNV : DecorationOperand<5273, 0, 0, [], [FragmentBarycentricNV]>;
1413: defm NonUniformEXT : DecorationOperand<5300, 0, 0, [], [ShaderNonUniformEXT]>;
1414: defm CountBuffer : DecorationOperand<5634, 0, 0, [], []>;
1415: defm UserSemantic : DecorationOperand<5635, 0, 0, [], []>;
1416: defm RestrictPointerEXT : DecorationOperand<5355, 0, 0, [], [PhysicalStorageBufferAddressesEXT]>;
1417: defm AliasedPointerEXT : DecorationOperand<5356, 0, 0, [], [PhysicalStorageBufferAddressesEXT]>;
1418: defm ReferencedIndirectlyINTEL : DecorationOperand<5602, 0, 0, [], [IndirectReferencesINTEL]>;
1419: defm ClobberINTEL : DecorationOperand<5607, 0, 0, [SPV_INTEL_inline_assembly], [AsmINTEL]>;
1420: defm SideEffectsINTEL : DecorationOperand<5608, 0, 0, [SPV_INTEL_inline_assembly], [AsmINTEL]>;
1421: defm ArgumentAttributeINTEL : DecorationOperand<6409, 0, 0, [], [FunctionPointersINTEL]>;
1422: defm CacheControlLoadINTEL : DecorationOperand<6442, 0, 0, [], [CacheControlsINTEL]>;
1423: defm CacheControlStoreINTEL : DecorationOperand<6443, 0, 0, [], [CacheControlsINTEL]>;
1424: defm HostAccessINTEL : DecorationOperand<6188, 0, 0, [], [GlobalVariableHostAccessINTEL]>;
1425: defm InitModeINTEL : DecorationOperand<6190, 0, 0, [], [GlobalVariableFPGADecorationsINTEL]>;
1426: defm ImplementInRegisterMapINTEL : DecorationOperand<6191, 0, 0, [], [GlobalVariableFPGADecorationsINTEL]>;
1427: defm FunctionRoundingModeINTEL : DecorationOperand<5822, 0, 0, [], [FunctionFloatControlINTEL]>;
1428: defm FunctionDenormModeINTEL : DecorationOperand<5823, 0, 0, [], [FunctionFloatControlINTEL]>;
1429: defm FunctionFloatingPointModeINTEL : DecorationOperand<6080, 0, 0, [], [FunctionFloatControlINTEL]>;
1430: defm AliasScopeINTEL : DecorationOperand<5914, 0, 0, [], [MemoryAccessAliasingINTEL]>;
1431: defm NoAliasINTEL : DecorationOperand<5915, 0, 0, [], [MemoryAccessAliasingINTEL]>;
1432: defm FPMaxErrorDecorationINTEL : DecorationOperand<6170, 0, 0, [], [FPMaxErrorINTEL]>;
1433:
1434: //===----------------------------------------------------------------------===//
1435: // Multiclass used to define BuiltIn enum values and at the same time
1436: // SymbolicOperand entries with string mnemonics, versioning, extensions and
1437: // capabilities.
1438: //===----------------------------------------------------------------------===//
1439:
1440: def BuiltIn : GenericEnum, Operand<i32> {
```
- EN: This range uses TableGen DSL to describe records such as Zext, Sext, ByVal, Sret; LLVM later expands these declarations into generated backend tables and helper code.
- CN: 这一段使用 TableGen DSL 描述 Zext、Sext、ByVal、Sret 等记录；LLVM 后续会把这些声明展开为生成的后端表和辅助代码。

### Lines 1441-1560
```tablegen
1441:   let FilterClass = "BuiltIn";
1442:   let NameField = "Name";
1443:   let ValueField = "Value";
1444:   let PrintMethod = !strconcat("printSymbolicOperand<OperandCategory::", FilterClass, "Operand>");
1445: }
1446:
1447: class BuiltIn<string name, bits<32> value> {
1448:   string Name = name;
1449:   bits<32> Value = value;
1450: }
1451:
1452: multiclass BuiltInOperand<bits<32> value, bits<32> minVersion, bits<32> maxVersion, list<Extension> reqExtensions, list<Capability> reqCapabilities> {
1453:   def NAME : BuiltIn<NAME, value>;
1454:   defm : SymbolicOperandWithRequirements<BuiltInOperand, value, NAME,
1455:                                          minVersion, maxVersion, reqExtensions,
1456:                                          reqCapabilities, []>;
1457: }
1458:
1459: defm Position : BuiltInOperand<0, 0, 0, [], [Shader]>;
1460: defm PointSize : BuiltInOperand<1, 0, 0, [], [Shader]>;
1461: defm ClipDistanceVariable : BuiltInOperand<3, 0, 0, [], [ClipDistance]>;
1462: defm CullDistanceVariable : BuiltInOperand<4, 0, 0, [], [CullDistance]>;
1463: defm VertexId : BuiltInOperand<5, 0, 0, [], [Shader]>;
1464: defm InstanceId : BuiltInOperand<6, 0, 0, [], [Shader]>;
1465: defm PrimitiveId : BuiltInOperand<7, 0, 0, [], [Geometry, Tessellation, RayTracingNV]>;
1466: defm InvocationId : BuiltInOperand<8, 0, 0, [], [Geometry, Tessellation]>;
1467: defm Layer : BuiltInOperand<9, 0, 0, [], [Geometry]>;
1468: defm ViewportIndex : BuiltInOperand<10, 0, 0, [], [MultiViewport]>;
1469: defm TessLevelOuter : BuiltInOperand<11, 0, 0, [], [Tessellation]>;
1470: defm TessLevelInner : BuiltInOperand<12, 0, 0, [], [Tessellation]>;
1471: defm TessCoord : BuiltInOperand<13, 0, 0, [], [Tessellation]>;
1472: defm PatchVertices : BuiltInOperand<14, 0, 0, [], [Tessellation]>;
1473: defm FragCoord : BuiltInOperand<15, 0, 0, [], [Shader]>;
1474: defm PointCoord : BuiltInOperand<16, 0, 0, [], [Shader]>;
1475: defm FrontFacing : BuiltInOperand<17, 0, 0, [], [Shader]>;
1476: defm SampleId : BuiltInOperand<18, 0, 0, [], [SampleRateShading]>;
1477: defm SamplePosition : BuiltInOperand<19, 0, 0, [], [SampleRateShading]>;
1478: defm SampleMask : BuiltInOperand<20, 0, 0, [], [Shader]>;
1479: defm FragDepth : BuiltInOperand<22, 0, 0, [], [Shader]>;
1480: defm HelperInvocation : BuiltInOperand<23, 0, 0, [], [Shader]>;
1481: defm NumWorkgroups : BuiltInOperand<24, 0, 0, [], []>;
1482: defm WorkgroupSize : BuiltInOperand<25, 0, 0, [], []>;
1483: defm WorkgroupId : BuiltInOperand<26, 0, 0, [], []>;
1484: defm LocalInvocationId : BuiltInOperand<27, 0, 0, [], []>;
1485: defm GlobalInvocationId : BuiltInOperand<28, 0, 0, [], []>;
1486: defm LocalInvocationIndex : BuiltInOperand<29, 0, 0, [], []>;
1487: defm WorkDim : BuiltInOperand<30, 0, 0, [], [Kernel]>;
1488: defm GlobalSize : BuiltInOperand<31, 0, 0, [], [Kernel]>;
1489: defm EnqueuedWorkgroupSize : BuiltInOperand<32, 0, 0, [], [Kernel]>;
1490: defm GlobalOffset : BuiltInOperand<33, 0, 0, [], [Kernel]>;
1491: defm GlobalLinearId : BuiltInOperand<34, 0, 0, [], [Kernel]>;
1492: defm SubgroupSize : BuiltInOperand<36, 0, 0, [], [Kernel, GroupNonUniform, SubgroupBallotKHR]>;
1493: defm SubgroupMaxSize : BuiltInOperand<37, 0, 0, [], [Kernel]>;
1494: defm NumSubgroups : BuiltInOperand<38, 0, 0, [], [Kernel, GroupNonUniform]>;
1495: defm NumEnqueuedSubgroups : BuiltInOperand<39, 0, 0, [], [Kernel]>;
1496: defm SubgroupId : BuiltInOperand<40, 0, 0, [], [Kernel, GroupNonUniform]>;
1497: defm SubgroupLocalInvocationId : BuiltInOperand<41, 0, 0, [], [Kernel, GroupNonUniform, SubgroupBallotKHR]>;
1498: defm VertexIndex : BuiltInOperand<42, 0, 0, [], [Shader]>;
1499: defm InstanceIndex : BuiltInOperand<43, 0, 0, [], [Shader]>;
1500: defm SubgroupEqMask : BuiltInOperand<4416, 0, 0, [], [SubgroupBallotKHR, GroupNonUniformBallot]>;
1501: defm SubgroupGeMask : BuiltInOperand<4417, 0, 0, [], [SubgroupBallotKHR, GroupNonUniformBallot]>;
1502: defm SubgroupGtMask : BuiltInOperand<4418, 0, 0, [], [SubgroupBallotKHR, GroupNonUniformBallot]>;
1503: defm SubgroupLeMask : BuiltInOperand<4419, 0, 0, [], [SubgroupBallotKHR, GroupNonUniformBallot]>;
1504: defm SubgroupLtMask : BuiltInOperand<4420, 0, 0, [], [SubgroupBallotKHR, GroupNonUniformBallot]>;
1505: defm BaseVertex : BuiltInOperand<4424, 0, 0, [], [DrawParameters]>;
1506: defm BaseInstance : BuiltInOperand<4425, 0, 0, [], [DrawParameters]>;
1507: defm DrawIndex : BuiltInOperand<4426, 0, 0, [], [DrawParameters, MeshShadingNV]>;
1508: defm DeviceIndex : BuiltInOperand<4438, 0, 0, [], [DeviceGroup]>;
1509: defm ViewIndex : BuiltInOperand<4440, 0, 0, [], [MultiView]>;
1510: defm BaryCoordNoPerspAMD : BuiltInOperand<4492, 0, 0, [], []>;
1511: defm BaryCoordNoPerspCentroidAMD : BuiltInOperand<4493, 0, 0, [], []>;
1512: defm BaryCoordNoPerspSampleAMD : BuiltInOperand<4494, 0, 0, [], []>;
1513: defm BaryCoordSmoothAMD : BuiltInOperand<4495, 0, 0, [], []>;
1514: defm BaryCoordSmoothCentroid : BuiltInOperand<4496, 0, 0, [], []>;
1515: defm BaryCoordSmoothSample : BuiltInOperand<4497, 0, 0, [], []>;
1516: defm BaryCoordPullModel : BuiltInOperand<4498, 0, 0, [], []>;
1517: defm FragStencilRefEXT : BuiltInOperand<5014, 0, 0, [], [StencilExportEXT]>;
1518: defm ViewportMaskNV : BuiltInOperand<5253, 0, 0, [], [ShaderViewportMaskNV, MeshShadingNV]>;
1519: defm SecondaryPositionNV : BuiltInOperand<5257, 0, 0, [], [ShaderStereoViewNV]>;
1520: defm SecondaryViewportMaskNV : BuiltInOperand<5258, 0, 0, [], [ShaderStereoViewNV]>;
1521: defm PositionPerViewNV : BuiltInOperand<5261, 0, 0, [], [PerViewAttributesNV, MeshShadingNV]>;
1522: defm ViewportMaskPerViewNV : BuiltInOperand<5262, 0, 0, [], [PerViewAttributesNV, MeshShadingNV]>;
1523: defm FullyCoveredEXT : BuiltInOperand<5264, 0, 0, [], [FragmentFullyCoveredEXT]>;
1524: defm TaskCountNV : BuiltInOperand<5274, 0, 0, [], [MeshShadingNV]>;
1525: defm PrimitiveCountNV : BuiltInOperand<5275, 0, 0, [], [MeshShadingNV]>;
1526: defm PrimitiveIndicesNV : BuiltInOperand<5276, 0, 0, [], [MeshShadingNV]>;
1527: defm ClipDistancePerViewNV : BuiltInOperand<5277, 0, 0, [], [MeshShadingNV]>;
1528: defm CullDistancePerViewNV : BuiltInOperand<5278, 0, 0, [], [MeshShadingNV]>;
1529: defm LayerPerViewNV : BuiltInOperand<5279, 0, 0, [], [MeshShadingNV]>;
1530: defm MeshViewCountNV : BuiltInOperand<5280, 0, 0, [], [MeshShadingNV]>;
1531: defm MeshViewIndices : BuiltInOperand<5281, 0, 0, [], [MeshShadingNV]>;
1532: defm BaryCoordNV : BuiltInOperand<5286, 0, 0, [], [FragmentBarycentricNV]>;
1533: defm BaryCoordNoPerspNV : BuiltInOperand<5287, 0, 0, [], [FragmentBarycentricNV]>;
1534: defm FragSizeEXT : BuiltInOperand<5292, 0, 0, [], [FragmentDensityEXT]>;
1535: defm FragInvocationCountEXT : BuiltInOperand<5293, 0, 0, [], [FragmentDensityEXT]>;
1536: defm LaunchIdNV : BuiltInOperand<5319, 0, 0, [], [RayTracingNV]>;
1537: defm LaunchSizeNV : BuiltInOperand<5320, 0, 0, [], [RayTracingNV]>;
1538: defm WorldRayOriginNV : BuiltInOperand<5321, 0, 0, [], [RayTracingNV]>;
1539: defm WorldRayDirectionNV : BuiltInOperand<5322, 0, 0, [], [RayTracingNV]>;
1540: defm ObjectRayOriginNV : BuiltInOperand<5323, 0, 0, [], [RayTracingNV]>;
1541: defm ObjectRayDirectionNV : BuiltInOperand<5324, 0, 0, [], [RayTracingNV]>;
1542: defm RayTminNV : BuiltInOperand<5325, 0, 0, [], [RayTracingNV]>;
1543: defm RayTmaxNV : BuiltInOperand<5326, 0, 0, [], [RayTracingNV]>;
1544: defm InstanceCustomIndexNV : BuiltInOperand<5327, 0, 0, [], [RayTracingNV]>;
1545: defm ObjectToWorldNV : BuiltInOperand<5330, 0, 0, [], [RayTracingNV]>;
1546: defm WorldToObjectNV : BuiltInOperand<5331, 0, 0, [], [RayTracingNV]>;
1547: defm HitTNV : BuiltInOperand<5332, 0, 0, [], [RayTracingNV]>;
1548: defm HitKindNV : BuiltInOperand<5333, 0, 0, [], [RayTracingNV]>;
1549: defm IncomingRayFlagsNV : BuiltInOperand<5351, 0, 0, [], [RayTracingNV]>;
1550:
1551: //===----------------------------------------------------------------------===//
1552: // Multiclass used to define SelectionControl enum values and at the same time
1553: // SymbolicOperand entries with string mnemonics.
1554: //===----------------------------------------------------------------------===//
1555:
1556: def SelectionControl : GenericEnum, Operand<i32> {
1557:   let FilterClass = "SelectionControl";
1558:   let NameField = "Name";
1559:   let ValueField = "Value";
1560:   let PrintMethod = !strconcat("printSymbolicOperand<OperandCategory::", FilterClass, "Operand>");
```
- EN: This range uses TableGen DSL to describe records such as BuiltIn, BuiltInOperand, NAME, Position; LLVM later expands these declarations into generated backend tables and helper code.
- CN: 这一段使用 TableGen DSL 描述 BuiltIn、BuiltInOperand、NAME、Position 等记录；LLVM 后续会把这些声明展开为生成的后端表和辅助代码。

### Lines 1561-1680
```tablegen
1561: }
1562:
1563: class SelectionControl<string name, bits<32> value> {
1564:   string Name = name;
1565:   bits<32> Value = value;
1566: }
1567:
1568: multiclass SelectionControlOperand<bits<32> value> {
1569:   def : SelectionControl<NAME, value>;
1570:   defm : SymbolicOperandWithRequirements<SelectionControlOperand, value, NAME,
1571:                                          0, 0, [], [], []>;
1572: }
1573:
1574: defm None : SelectionControlOperand<0x0>;
1575: defm Flatten : SelectionControlOperand<0x1>;
1576: defm DontFlatten : SelectionControlOperand<0x2>;
1577:
1578: //===----------------------------------------------------------------------===//
1579: // Multiclass used to define LoopControl enum values and at the same time
1580: // SymbolicOperand entries with string mnemonics.
1581: //===----------------------------------------------------------------------===//
1582:
1583: def LoopControl : GenericEnum, Operand<i32> {
1584:   let FilterClass = "LoopControl";
1585:   let NameField = "Name";
1586:   let ValueField = "Value";
1587:   let PrintMethod = !strconcat("printSymbolicOperand<OperandCategory::", FilterClass, "Operand>");
1588: }
1589:
1590: class LoopControl<string name, bits<32> value> {
1591:   string Name = name;
1592:   bits<32> Value = value;
1593: }
1594:
1595: multiclass LoopControlOperand<bits<32> value> {
1596:   def : LoopControl<NAME, value>;
1597:   defm : SymbolicOperandWithRequirements<LoopControlOperand, value, NAME, 0,
1598:                                          0, [], [], []>;
1599: }
1600:
1601: defm None : LoopControlOperand<0x0>;
1602: defm Unroll : LoopControlOperand<0x1>;
1603: defm DontUnroll : LoopControlOperand<0x2>;
1604: defm DependencyInfinite : LoopControlOperand<0x4>;
1605: defm DependencyLength : LoopControlOperand<0x8>;
1606: defm MinIterations : LoopControlOperand<0x10>;
1607: defm MaxIterations : LoopControlOperand<0x20>;
1608: defm IterationMultiple : LoopControlOperand<0x40>;
1609: defm PeelCount : LoopControlOperand<0x80>;
1610: defm PartialCount : LoopControlOperand<0x100>;
1611:
1612: //===----------------------------------------------------------------------===//
1613: // Multiclass used to define FunctionControl enum values and at the same time
1614: // SymbolicOperand entries with string mnemonics.
1615: //===----------------------------------------------------------------------===//
1616:
1617: def FunctionControl : GenericEnum, Operand<i32> {
1618:   let FilterClass = "FunctionControl";
1619:   let NameField = "Name";
1620:   let ValueField = "Value";
1621:   let PrintMethod = !strconcat("printSymbolicOperand<OperandCategory::", FilterClass, "Operand>");
1622: }
1623:
1624: class FunctionControl<string name, bits<32> value> {
1625:   string Name = name;
1626:   bits<32> Value = value;
1627: }
1628:
1629: multiclass FunctionControlOperand<bits<32> value> {
1630:   def : FunctionControl<NAME, value>;
1631:   defm : SymbolicOperandWithRequirements<FunctionControlOperand, value, NAME, 0,
1632:                                          0, [], [], []>;
1633: }
1634:
1635: defm None : FunctionControlOperand<0x0>;
1636: defm Inline : FunctionControlOperand<0x1>;
1637: defm DontInline : FunctionControlOperand<0x2>;
1638: defm Pure : FunctionControlOperand<0x4>;
1639: defm Const : FunctionControlOperand<0x8>;
1640: defm OptNoneEXT : FunctionControlOperand<0x10000>;
1641:
1642: //===----------------------------------------------------------------------===//
1643: // Multiclass used to define MemorySemantics enum values and at the same time
1644: // SymbolicOperand entries with string mnemonics, versioning, extensions and
1645: // capabilities.
1646: //===----------------------------------------------------------------------===//
1647:
1648: def MemorySemantics : GenericEnum, Operand<i32> {
1649:   let FilterClass = "MemorySemantics";
1650:   let NameField = "Name";
1651:   let ValueField = "Value";
1652:   let PrintMethod = !strconcat("printSymbolicOperand<OperandCategory::", FilterClass, "Operand>");
1653: }
1654:
1655: class MemorySemantics<string name, bits<32> value> {
1656:   string Name = name;
1657:   bits<32> Value = value;
1658: }
1659:
1660: multiclass MemorySemanticsOperand<bits<32> value, bits<32> minVersion, bits<32> maxVersion, list<Extension> reqExtensions, list<Capability> reqCapabilities> {
1661:   def : MemorySemantics<NAME, value>;
1662:   defm : SymbolicOperandWithRequirements<MemorySemanticsOperand, value, NAME,
1663:                                          minVersion, maxVersion, reqExtensions,
1664:                                          reqCapabilities, []>;
1665: }
1666:
1667: defm None : MemorySemanticsOperand<0x0, 0, 0, [], []>;
1668: defm Acquire : MemorySemanticsOperand<0x2, 0, 0, [], []>;
1669: defm Release : MemorySemanticsOperand<0x4, 0, 0, [], []>;
1670: defm AcquireRelease : MemorySemanticsOperand<0x8, 0, 0, [], []>;
1671: defm SequentiallyConsistent : MemorySemanticsOperand<0x10, 0, 0, [], []>;
1672: defm UniformMemory : MemorySemanticsOperand<0x40, 0, 0, [], [Shader]>;
1673: defm SubgroupMemory : MemorySemanticsOperand<0x80, 0, 0, [], []>;
1674: defm WorkgroupMemory : MemorySemanticsOperand<0x100, 0, 0, [], []>;
1675: defm CrossWorkgroupMemory : MemorySemanticsOperand<0x200, 0, 0, [], []>;
1676: defm AtomicCounterMemory : MemorySemanticsOperand<0x400, 0, 0, [], [AtomicStorage]>;
1677: defm ImageMemory : MemorySemanticsOperand<0x800, 0, 0, [], []>;
1678: defm OutputMemoryKHR : MemorySemanticsOperand<0x1000, 0, 0, [], [VulkanMemoryModelKHR]>;
1679: defm MakeAvailableKHR : MemorySemanticsOperand<0x2000, 0, 0, [], [VulkanMemoryModelKHR]>;
1680: defm MakeVisibleKHR : MemorySemanticsOperand<0x4000, 0, 0, [], [VulkanMemoryModelKHR]>;
```
- EN: This range uses TableGen DSL to describe records such as SelectionControl, SelectionControlOperand, None, Flatten; LLVM later expands these declarations into generated backend tables and helper code.
- CN: 这一段使用 TableGen DSL 描述 SelectionControl、SelectionControlOperand、None、Flatten 等记录；LLVM 后续会把这些声明展开为生成的后端表和辅助代码。

### Lines 1681-1800
```tablegen
1681: defm Volatile : MemorySemanticsOperand<0x8000, 0, 0, [], [VulkanMemoryModelKHR]>;
1682:
1683: //===----------------------------------------------------------------------===//
1684: // Multiclass used to define MemoryOperand enum values and at the same time
1685: // SymbolicOperand entries with string mnemonics, versioning, extensions and
1686: // capabilities.
1687: //===----------------------------------------------------------------------===//
1688:
1689: def MemoryOperand : GenericEnum, Operand<i32> {
1690:   let FilterClass = "MemoryOperand";
1691:   let NameField = "Name";
1692:   let ValueField = "Value";
1693:   let PrintMethod = !strconcat("printSymbolicOperand<OperandCategory::", FilterClass, "Operand>");
1694: }
1695:
1696: class MemoryOperand<string name, bits<32> value> {
1697:   string Name = name;
1698:   bits<32> Value = value;
1699: }
1700:
1701: multiclass MemoryOperandOperand<bits<32> value, bits<32> minVersion, bits<32> maxVersion, list<Extension> reqExtensions, list<Capability> reqCapabilities> {
1702:   def : MemoryOperand<NAME, value>;
1703:   defm : SymbolicOperandWithRequirements<MemoryOperandOperand, value, NAME,
1704:                                          minVersion, maxVersion, reqExtensions,
1705:                                          reqCapabilities, []>;
1706: }
1707:
1708: defm None : MemoryOperandOperand<0x0, 0, 0, [], []>;
1709: defm Volatile : MemoryOperandOperand<0x1, 0, 0, [], []>;
1710: defm Aligned : MemoryOperandOperand<0x2, 0, 0, [], [Kernel]>;
1711: defm Nontemporal : MemoryOperandOperand<0x4, 0, 0, [], []>;
1712: defm MakePointerAvailableKHR : MemoryOperandOperand<0x8, 0, 0, [], [VulkanMemoryModelKHR]>;
1713: defm MakePointerVisibleKHR : MemoryOperandOperand<0x10, 0, 0, [], [VulkanMemoryModelKHR]>;
1714: defm NonPrivatePointerKHR : MemoryOperandOperand<0x20, 0, 0, [], [VulkanMemoryModelKHR]>;
1715: defm AliasScopeINTELMask : MemoryOperandOperand<0x10000, 0, 0, [], [MemoryAccessAliasingINTEL]>;
1716: defm NoAliasINTELMask : MemoryOperandOperand<0x20000, 0, 0, [], [MemoryAccessAliasingINTEL]>;
1717:
1718: //===----------------------------------------------------------------------===//
1719: // Multiclass used to define Scope enum values and at the same time
1720: // SymbolicOperand entries with string mnemonics, versioning, extensions and
1721: // capabilities.
1722: //===----------------------------------------------------------------------===//
1723:
1724: def Scope : GenericEnum, Operand<i32> {
1725:   let FilterClass = "Scope";
1726:   let NameField = "Name";
1727:   let ValueField = "Value";
1728:   let PrintMethod = !strconcat("printSymbolicOperand<OperandCategory::", FilterClass, "Operand>");
1729: }
1730:
1731: class Scope<string name, bits<32> value> {
1732:   string Name = name;
1733:   bits<32> Value = value;
1734: }
1735:
1736: multiclass ScopeOperand<bits<32> value, bits<32> minVersion, bits<32> maxVersion, list<Extension> reqExtensions, list<Capability> reqCapabilities> {
1737:   def : Scope<NAME, value>;
1738:   defm : SymbolicOperandWithRequirements<ScopeOperand, value, NAME, minVersion,
1739:                                          maxVersion, reqExtensions,
1740:                                          reqCapabilities, []>;
1741: }
1742:
1743: defm CrossDevice : ScopeOperand<0, 0, 0, [], []>;
1744: defm Device : ScopeOperand<1, 0, 0, [], []>;
1745: defm Workgroup : ScopeOperand<2, 0, 0, [], []>;
1746: defm Subgroup : ScopeOperand<3, 0, 0, [], []>;
1747: defm Invocation : ScopeOperand<4, 0, 0, [], []>;
1748: defm QueueFamilyKHR : ScopeOperand<5, 0, 0, [], [VulkanMemoryModelKHR]>;
1749:
1750: //===----------------------------------------------------------------------===//
1751: // Multiclass used to define GroupOperation enum values and at the same time
1752: // SymbolicOperand entries with string mnemonics, versioning, extensions and
1753: // capabilities.
1754: //===----------------------------------------------------------------------===//
1755:
1756: def GroupOperation : GenericEnum, Operand<i32> {
1757:   let FilterClass = "GroupOperation";
1758:   let NameField = "Name";
1759:   let ValueField = "Value";
1760:   let PrintMethod = !strconcat("printSymbolicOperand<OperandCategory::", FilterClass, "Operand>");
1761: }
1762:
1763: class GroupOperation<string name, bits<32> value> {
1764:   string Name = name;
1765:   bits<32> Value = value;
1766: }
1767:
1768: multiclass GroupOperationOperand<bits<32> value, bits<32> minVersion, bits<32> maxVersion, list<Extension> reqExtensions, list<Capability> reqCapabilities> {
1769:   def NAME : GroupOperation<NAME, value>;
1770:   defm : SymbolicOperandWithRequirements<GroupOperationOperand, value, NAME,
1771:                                          minVersion, maxVersion, reqExtensions,
1772:                                          reqCapabilities, []>;
1773: }
1774:
1775: defm Reduce : GroupOperationOperand<0, 0, 0, [], [Kernel, GroupNonUniformArithmetic, GroupNonUniformBallot]>;
1776: defm InclusiveScan : GroupOperationOperand<1, 0, 0, [], [Kernel, GroupNonUniformArithmetic, GroupNonUniformBallot]>;
1777: defm ExclusiveScan : GroupOperationOperand<2, 0, 0, [], [Kernel, GroupNonUniformArithmetic, GroupNonUniformBallot]>;
1778: defm ClusteredReduce : GroupOperationOperand<3, 0, 0, [], [GroupNonUniformClustered]>;
1779: defm PartitionedReduceNV : GroupOperationOperand<6, 0, 0, [], [GroupNonUniformPartitionedNV]>;
1780: defm PartitionedInclusiveScanNV : GroupOperationOperand<7, 0, 0, [], [GroupNonUniformPartitionedNV]>;
1781: defm PartitionedExclusiveScanNV : GroupOperationOperand<8, 0, 0, [], [GroupNonUniformPartitionedNV]>;
1782:
1783: //===----------------------------------------------------------------------===//
1784: // Multiclass used to define KernelEnqueueFlags enum values and at the same time
1785: // SymbolicOperand entries with string mnemonics, versioning, extensions and
1786: // capabilities.
1787: //===----------------------------------------------------------------------===//
1788:
1789: def KernelEnqueueFlags : GenericEnum, Operand<i32> {
1790:   let FilterClass = "KernelEnqueueFlags";
1791:   let NameField = "Name";
1792:   let ValueField = "Value";
1793:   let PrintMethod = !strconcat("printSymbolicOperand<OperandCategory::", FilterClass, "Operand>");
1794: }
1795:
1796: class KernelEnqueueFlags<string name, bits<32> value> {
1797:   string Name = name;
1798:   bits<32> Value = value;
1799: }
1800:
```
- EN: This range uses TableGen DSL to describe records such as Volatile, MemoryOperand, MemoryOperandOperand, None; LLVM later expands these declarations into generated backend tables and helper code.
- CN: 这一段使用 TableGen DSL 描述 Volatile、MemoryOperand、MemoryOperandOperand、None 等记录；LLVM 后续会把这些声明展开为生成的后端表和辅助代码。

### Lines 1801-1920
```tablegen
1801: multiclass KernelEnqueueFlagsOperand<bits<32> value, bits<32> minVersion, bits<32> maxVersion, list<Extension> reqExtensions, list<Capability> reqCapabilities> {
1802:   def : KernelEnqueueFlags<NAME, value>;
1803:   defm : SymbolicOperandWithRequirements<KernelEnqueueFlagsOperand, value, NAME,
1804:                                          minVersion, maxVersion, reqExtensions,
1805:                                          reqCapabilities, []>;
1806: }
1807:
1808: defm NoWait : KernelEnqueueFlagsOperand<0, 0, 0, [], [Kernel]>;
1809: defm WaitKernel : KernelEnqueueFlagsOperand<1, 0, 0, [], [Kernel]>;
1810: defm WaitWorkGroup : KernelEnqueueFlagsOperand<2, 0, 0, [], [Kernel]>;
1811:
1812: //===----------------------------------------------------------------------===//
1813: // Multiclass used to define KernelProfilingInfo enum values and at the same time
1814: // SymbolicOperand entries with string mnemonics, versioning, extensions and
1815: // capabilities.
1816: //===----------------------------------------------------------------------===//
1817:
1818: def KernelProfilingInfo : GenericEnum, Operand<i32> {
1819:   let FilterClass = "KernelProfilingInfo";
1820:   let NameField = "Name";
1821:   let ValueField = "Value";
1822:   let PrintMethod = !strconcat("printSymbolicOperand<OperandCategory::", FilterClass, "Operand>");
1823: }
1824:
1825: class KernelProfilingInfo<string name, bits<32> value> {
1826:   string Name = name;
1827:   bits<32> Value = value;
1828: }
1829:
1830: multiclass KernelProfilingInfoOperand<bits<32> value, bits<32> minVersion, bits<32> maxVersion, list<Extension> reqExtensions, list<Capability> reqCapabilities> {
1831:   def : KernelProfilingInfo<NAME, value>;
1832:   defm : SymbolicOperandWithRequirements<KernelProfilingInfoOperand, value,
1833:                                          NAME, minVersion, maxVersion,
1834:                                          reqExtensions, reqCapabilities, []>;
1835: }
1836:
1837: defm None : KernelProfilingInfoOperand<0x0, 0, 0, [], []>;
1838: defm CmdExecTime : KernelProfilingInfoOperand<0x1, 0, 0, [], [Kernel]>;
1839:
1840: //===----------------------------------------------------------------------===//
1841: // Multiclass used to define Opcode enum values and at the same time
1842: // SymbolicOperand entries with string mnemonics and capabilities.
1843: //===----------------------------------------------------------------------===//
1844:
1845: def Opcode : GenericEnum, Operand<i32> {
1846:   let FilterClass = "Opcode";
1847:   let NameField = "Name";
1848:   let ValueField = "Value";
1849:   let PrintMethod = !strconcat("printSymbolicOperand<OperandCategory::", FilterClass, "Operand>");
1850: }
1851:
1852: class Opcode<string name, bits<32> value> {
1853:   string Name = name;
1854:   bits<32> Value = value;
1855: }
1856:
1857: multiclass OpcodeOperand<bits<32> value> {
1858:   def : Opcode<NAME, value>;
1859:   defm : SymbolicOperandWithRequirements<OpcodeOperand, value, NAME, 0,
1860:                                          0, [], [], []>;
1861: }
1862: // TODO: implement other mnemonics.
1863: defm InBoundsAccessChain : OpcodeOperand<66>;
1864: defm InBoundsPtrAccessChain : OpcodeOperand<70>;
1865: defm PtrCastToGeneric : OpcodeOperand<121>;
1866: defm GenericCastToPtr : OpcodeOperand<122>;
1867: defm GenericCastToPtrExplicit : OpcodeOperand<123>;
1868: defm Bitcast : OpcodeOperand<124>;
1869: defm ConvertPtrToU : OpcodeOperand<117>;
1870: defm ConvertUToPtr : OpcodeOperand<120>;
1871:
1872: //===----------------------------------------------------------------------===//
1873: // Multiclass used to define Cooperative Matrix Layout enum values and at the
1874: // same time SymbolicOperand entries extensions and capabilities.
1875: //===----------------------------------------------------------------------===//
1876:
1877: def CooperativeMatrixLayout : GenericEnum, Operand<i32> {
1878:   let FilterClass = "CooperativeMatrixLayout";
1879:   let NameField = "Name";
1880:   let ValueField = "Value";
1881: }
1882:
1883: class CooperativeMatrixLayout<string name, bits<32> value> {
1884:   string Name = name;
1885:   bits<32> Value = value;
1886: }
1887:
1888: multiclass CooperativeMatrixLayoutOperand<bits<32> value, list<Extension> reqExtensions, list<Capability> reqCapabilities> {
1889:   def : CooperativeMatrixLayout<NAME, value>;
1890:   defm : SymbolicOperandWithRequirements<CooperativeMatrixLayoutOperand, value,
1891:                                          NAME, 0, 0, reqExtensions,
1892:                                          reqCapabilities, []>;
1893: }
1894:
1895: defm RowMajorKHR : CooperativeMatrixLayoutOperand<0x0, [SPV_KHR_cooperative_matrix], [CooperativeMatrixKHR]>;
1896: defm ColumnMajorKHR : CooperativeMatrixLayoutOperand<0x1, [SPV_KHR_cooperative_matrix], [CooperativeMatrixKHR]>;
1897: defm PackedINTEL : CooperativeMatrixLayoutOperand<0x2, [SPV_INTEL_joint_matrix], [PackedCooperativeMatrixINTEL]>;
1898:
1899: //===----------------------------------------------------------------------===//
1900: // Multiclass used to define Cooperative Matrix Operands enum values and at the
1901: // same time SymbolicOperand entries with string mnemonics, extensions and
1902: // capabilities.
1903: //===----------------------------------------------------------------------===//
1904:
1905: def CooperativeMatrixOperands : GenericEnum, Operand<i32> {
1906:   let FilterClass = "CooperativeMatrixOperands";
1907:   let NameField = "Name";
1908:   let ValueField = "Value";
1909:   let PrintMethod = !strconcat("printSymbolicOperand<OperandCategory::", FilterClass, "Operand>");
1910: }
1911:
1912: class CooperativeMatrixOperands<string name, bits<32> value> {
1913:   string Name = name;
1914:   bits<32> Value = value;
1915: }
1916:
1917: multiclass CooperativeMatrixOperandsOperand<bits<32> value, list<Extension> reqExtensions, list<Capability> reqCapabilities> {
1918:   def : CooperativeMatrixOperands<NAME, value>;
1919:   defm : SymbolicOperandWithRequirements<CooperativeMatrixOperandsOperand,
1920:                                          value, NAME, 0, 0, reqExtensions,
```
- EN: This range uses TableGen DSL to describe records such as KernelEnqueueFlagsOperand, NoWait, WaitKernel, WaitWorkGroup; LLVM later expands these declarations into generated backend tables and helper code.
- CN: 这一段使用 TableGen DSL 描述 KernelEnqueueFlagsOperand、NoWait、WaitKernel、WaitWorkGroup 等记录；LLVM 后续会把这些声明展开为生成的后端表和辅助代码。

### Lines 1921-2040
```tablegen
1921:                                          reqCapabilities, []>;
1922: }
1923:
1924: defm NoneKHR : CooperativeMatrixOperandsOperand<0x0, [SPV_KHR_cooperative_matrix], [CooperativeMatrixKHR]>;
1925: defm MatrixASignedComponentsKHR : CooperativeMatrixOperandsOperand<0x1, [SPV_KHR_cooperative_matrix], [CooperativeMatrixKHR]>;
1926: defm MatrixBSignedComponentsKHR : CooperativeMatrixOperandsOperand<0x2, [SPV_KHR_cooperative_matrix], [CooperativeMatrixKHR]>;
1927: defm MatrixCSignedComponentsKHR : CooperativeMatrixOperandsOperand<0x4, [SPV_KHR_cooperative_matrix], [CooperativeMatrixKHR]>;
1928: defm MatrixResultSignedComponentsKHR : CooperativeMatrixOperandsOperand<0x8, [SPV_KHR_cooperative_matrix], [CooperativeMatrixKHR]>;
1929: defm SaturatingAccumulationKHR : CooperativeMatrixOperandsOperand<0x10, [SPV_KHR_cooperative_matrix], [CooperativeMatrixKHR]>;
1930: defm MatrixAAndBTF32ComponentsINTEL : CooperativeMatrixOperandsOperand<0x20, [SPV_INTEL_joint_matrix], [CooperativeMatrixTF32ComponentTypeINTEL]>;
1931: defm MatrixAAndBBFloat16ComponentsINTEL : CooperativeMatrixOperandsOperand<0x40, [SPV_INTEL_joint_matrix], [CooperativeMatrixBFloat16ComponentTypeINTEL]>;
1932: defm MatrixCBFloat16ComponentsINTEL : CooperativeMatrixOperandsOperand<0x80, [SPV_INTEL_joint_matrix], [CooperativeMatrixBFloat16ComponentTypeINTEL]>;
1933: defm MatrixResultBFloat16ComponentsINTEL : CooperativeMatrixOperandsOperand<0x100, [SPV_INTEL_joint_matrix], [CooperativeMatrixBFloat16ComponentTypeINTEL]>;
1934:
1935: //===----------------------------------------------------------------------===//
1936: // Multiclass used to define SpecConstant Operands enum values and at the
1937: // same time SymbolicOperand.
1938: //===----------------------------------------------------------------------===//
1939:
1940: def SpecConstantOpOperands : GenericEnum, Operand<i32> {
1941:   let FilterClass = "SpecConstantOpOperands";
1942:   let NameField = "Name";
1943:   let ValueField = "Value";
1944:   let PrintMethod = !strconcat("printSymbolicOperand<OperandCategory::", FilterClass, "Operand>");
1945: }
1946:
1947: class SpecConstantOpOperands<string name, bits<32> value> {
1948:   string Name = name;
1949:   bits<32> Value = value;
1950: }
1951:
1952: multiclass SpecConstantOpOperandsOperand<bits<32> value, list<Extension> reqExtensions, list<Capability> reqCapabilities> {
1953:   def : SpecConstantOpOperands<NAME, value>;
1954:   defm : SymbolicOperandWithRequirements<SpecConstantOpOperandsOperand, value,
1955:                                          NAME, 0, 0, reqExtensions,
1956:                                          reqCapabilities, []>;
1957: }
1958:
1959: // Conversion
1960: defm SConvert :  SpecConstantOpOperandsOperand<114, [], []>;
1961: defm FConvert :  SpecConstantOpOperandsOperand<115, [], []>;
1962: defm ConvertFToS :  SpecConstantOpOperandsOperand<110, [], [Kernel]>;
1963: defm ConvertSToF :  SpecConstantOpOperandsOperand<111, [], [Kernel]>;
1964: defm ConvertFToU :  SpecConstantOpOperandsOperand<109, [], [Kernel]>;
1965: defm ConvertUToF :  SpecConstantOpOperandsOperand<112, [], [Kernel]>;
1966: defm UConvert :  SpecConstantOpOperandsOperand<113, [], [Kernel]>;
1967: defm ConvertPtrToU :  SpecConstantOpOperandsOperand<117, [], [Kernel]>;
1968: defm ConvertUToPtr :  SpecConstantOpOperandsOperand<120, [], [Kernel]>;
1969: defm GenericCastToPtr :  SpecConstantOpOperandsOperand<122, [], [Kernel]>;
1970: defm PtrCastToGeneric :  SpecConstantOpOperandsOperand<121, [], [Kernel]>;
1971: defm Bitcast :  SpecConstantOpOperandsOperand<124, [], []>;
1972: defm QuantizeToF16 :  SpecConstantOpOperandsOperand<116, [], [Shader]>;
1973: // Arithmetic
1974: defm SNegate :  SpecConstantOpOperandsOperand<126, [], []>;
1975: defm Not :  SpecConstantOpOperandsOperand<200, [], []>;
1976: defm IAdd :  SpecConstantOpOperandsOperand<128, [], []>;
1977: defm ISub :  SpecConstantOpOperandsOperand<130, [], []>;
1978: defm IMul :  SpecConstantOpOperandsOperand<132, [], []>;
1979: defm UDiv :  SpecConstantOpOperandsOperand<134, [], []>;
1980: defm SDiv :  SpecConstantOpOperandsOperand<135, [], []>;
1981: defm UMod :  SpecConstantOpOperandsOperand<137, [], []>;
1982: defm SRem :  SpecConstantOpOperandsOperand<138, [], []>;
1983: defm SMod :  SpecConstantOpOperandsOperand<139, [], []>;
1984: defm ShiftRightLogical :  SpecConstantOpOperandsOperand<194, [], []>;
1985: defm ShiftRightArithmetic :  SpecConstantOpOperandsOperand<195, [], []>;
1986: defm ShiftLeftLogical :  SpecConstantOpOperandsOperand<196, [], []>;
1987: defm BitwiseOr :  SpecConstantOpOperandsOperand<197, [], []>;
1988: defm BitwiseAnd :  SpecConstantOpOperandsOperand<199, [], []>;
1989: defm BitwiseXor :  SpecConstantOpOperandsOperand<198, [], []>;
1990: defm FNegate :  SpecConstantOpOperandsOperand<127, [], [Kernel]>;
1991: defm FAdd :  SpecConstantOpOperandsOperand<129, [], [Kernel]>;
1992: defm FSub :  SpecConstantOpOperandsOperand<131, [], [Kernel]>;
1993: defm FMul :  SpecConstantOpOperandsOperand<133, [], [Kernel]>;
1994: defm FDiv :  SpecConstantOpOperandsOperand<136, [], [Kernel]>;
1995: defm FRem :  SpecConstantOpOperandsOperand<140, [], [Kernel]>;
1996: defm FMod :  SpecConstantOpOperandsOperand<141, [], [Kernel]>;
1997: // Composite;
1998: defm VectorShuffle :  SpecConstantOpOperandsOperand<79, [], []>;
1999: defm CompositeExtract :  SpecConstantOpOperandsOperand<81, [], []>;
2000: defm CompositeInsert :  SpecConstantOpOperandsOperand<82, [], []>;
2001: // Logical;
2002: defm LogicalOr :  SpecConstantOpOperandsOperand<166, [], []>;
2003: defm LogicalAnd :  SpecConstantOpOperandsOperand<167, [], []>;
2004: defm LogicalNot :  SpecConstantOpOperandsOperand<168, [], []>;
2005: defm LogicalEqual :  SpecConstantOpOperandsOperand<164, [], []>;
2006: defm LogicalNotEqual :  SpecConstantOpOperandsOperand<165, [], []>;
2007: defm Select :  SpecConstantOpOperandsOperand<169, [], []>;
2008: // Comparison;
2009: defm IEqual :  SpecConstantOpOperandsOperand<170, [], []>;
2010: defm INotEqual :  SpecConstantOpOperandsOperand<171, [], []>;
2011: defm ULessThan :  SpecConstantOpOperandsOperand<176, [], []>;
2012: defm SLessThan :  SpecConstantOpOperandsOperand<177, [], []>;
2013: defm UGreaterThan :  SpecConstantOpOperandsOperand<172, [], []>;
2014: defm SGreaterThan :  SpecConstantOpOperandsOperand<173, [], []>;
2015: defm ULessThanEqual :  SpecConstantOpOperandsOperand<178, [], []>;
2016: defm SLessThanEqual :  SpecConstantOpOperandsOperand<179, [], []>;
2017: defm UGreaterThanEqual :  SpecConstantOpOperandsOperand<174, [], []>;
2018: defm SGreaterThanEqual :  SpecConstantOpOperandsOperand<175, [], []>;
2019: // Memory
2020: defm AccessChain :  SpecConstantOpOperandsOperand<65, [], [Kernel]>;
2021: defm InBoundsAccessChain :  SpecConstantOpOperandsOperand<66, [], [Kernel]>;
2022: defm PtrAccessChain :  SpecConstantOpOperandsOperand<67, [], [Kernel]>;
2023: defm InBoundsPtrAccessChain :  SpecConstantOpOperandsOperand<70, [], [Kernel]>;
2024: defm CooperativeMatrixLengthKHR : SpecConstantOpOperandsOperand<4460, [], []>;
2025:
2026: //===----------------------------------------------------------------------===//
2027: // Multiclass used to define Matrix Multiply Accumulate Operands enum values and at the same time
2028: // SymbolicOperand entries with string mnemonics and capabilities.
2029: //===----------------------------------------------------------------------===//
2030: def MatrixMultiplyAccumulateOperands : GenericEnum, Operand<i32> {
2031:   let FilterClass = "MatrixMultiplyAccumulateOperands";
2032:   let NameField = "Name";
2033:   let ValueField = "Value";
2034:   let PrintMethod = !strconcat("printSymbolicOperand<OperandCategory::", FilterClass, "Operand>");
2035: }
2036:
2037: class MatrixMultiplyAccumulateOperands<string name, bits<32> value> {
2038:   string Name = name;
2039:   bits<32> Value = value;
2040: }
```
- EN: This range uses TableGen DSL to describe records such as NoneKHR, MatrixASignedComponentsKHR, MatrixBSignedComponentsKHR, MatrixCSignedComponentsKHR; LLVM later expands these declarations into generated backend tables and helper code.
- CN: 这一段使用 TableGen DSL 描述 NoneKHR、MatrixASignedComponentsKHR、MatrixBSignedComponentsKHR、MatrixCSignedComponentsKHR 等记录；LLVM 后续会把这些声明展开为生成的后端表和辅助代码。

### Lines 2041-2107
```tablegen
2041:
2042: multiclass  MatrixMultiplyAccumulateOperandsOperand<bits<32> value, list<Extension> reqExtensions> {
2043:   def : MatrixMultiplyAccumulateOperands<NAME, value>;
2044:   defm : SymbolicOperandWithRequirements<
2045:              MatrixMultiplyAccumulateOperandsOperand, value, NAME, 0, 0,
2046:              reqExtensions, [], []>;
2047: }
2048:
2049: defm None :  MatrixMultiplyAccumulateOperandsOperand<0x0, [SPV_INTEL_subgroup_matrix_multiply_accumulate]>;
2050: defm MatrixASignedComponentsINTEL :  MatrixMultiplyAccumulateOperandsOperand<0x1, [SPV_INTEL_subgroup_matrix_multiply_accumulate]>;
2051: defm MatrixBSignedComponentsINTEL :  MatrixMultiplyAccumulateOperandsOperand<0x2, [SPV_INTEL_subgroup_matrix_multiply_accumulate]>;
2052: defm MatrixCBFloat16INTEL :  MatrixMultiplyAccumulateOperandsOperand<0x4, [SPV_INTEL_subgroup_matrix_multiply_accumulate]>;
2053: defm MatrixResultBFloat16INTEL :  MatrixMultiplyAccumulateOperandsOperand<0x8, [SPV_INTEL_subgroup_matrix_multiply_accumulate]>;
2054: defm MatrixAPackedInt8INTEL :  MatrixMultiplyAccumulateOperandsOperand<0x10, [SPV_INTEL_subgroup_matrix_multiply_accumulate]>;
2055: defm MatrixBPackedInt8INTEL :  MatrixMultiplyAccumulateOperandsOperand<0x20, [SPV_INTEL_subgroup_matrix_multiply_accumulate]>;
2056: defm MatrixAPackedInt4INTEL :  MatrixMultiplyAccumulateOperandsOperand<0x40, [SPV_INTEL_subgroup_matrix_multiply_accumulate]>;
2057: defm MatrixBPackedInt4INTEL :  MatrixMultiplyAccumulateOperandsOperand<0x80, [SPV_INTEL_subgroup_matrix_multiply_accumulate]>;
2058: defm MatrixATF32INTEL :  MatrixMultiplyAccumulateOperandsOperand<0x100, [SPV_INTEL_subgroup_matrix_multiply_accumulate]>;
2059: defm MatrixBTF32INTEL :  MatrixMultiplyAccumulateOperandsOperand<0x200, [SPV_INTEL_subgroup_matrix_multiply_accumulate]>;
2060: defm MatrixAPackedFloat16INTEL :  MatrixMultiplyAccumulateOperandsOperand<0x400, [SPV_INTEL_subgroup_matrix_multiply_accumulate]>;
2061: defm MatrixBPackedFloat16INTEL :  MatrixMultiplyAccumulateOperandsOperand<0x800, [SPV_INTEL_subgroup_matrix_multiply_accumulate]>;
2062: defm MatrixAPackedBFloat16INTEL :  MatrixMultiplyAccumulateOperandsOperand<0x1000, [SPV_INTEL_subgroup_matrix_multiply_accumulate]>;
2063: defm MatrixBPackedBFloat16INTEL :  MatrixMultiplyAccumulateOperandsOperand<0x2000, [SPV_INTEL_subgroup_matrix_multiply_accumulate]>;
2064:
2065: //===----------------------------------------------------------------------===//
2066: // Multiclass used to define FPEncoding enum values and at the
2067: // same time SymbolicOperand entries with extensions.
2068: //===----------------------------------------------------------------------===//
2069: def FPEncoding : GenericEnum, Operand<i32> {
2070:   let FilterClass = "FPEncoding";
2071:   let NameField = "Name";
2072:   let ValueField = "Value";
2073:   let PrintMethod = !strconcat("printSymbolicOperand<OperandCategory::", FilterClass, "Operand>");
2074: }
2075:
2076: class FPEncoding<string name, bits<32> value> {
2077:   string Name = name;
2078:   bits<32> Value = value;
2079: }
2080:
2081: multiclass FPEncodingOperand<bits<32> value, list<Extension> reqExtensions>{
2082:   def NAME : FPEncoding<NAME, value>;
2083:   defm : SymbolicOperandWithRequirements<
2084:              FPEncodingOperand, value, NAME, 0, 0,
2085:              reqExtensions, [], []>;
2086: }
2087:
2088: defm BFloat16KHR : FPEncodingOperand<0, [SPV_KHR_bfloat16]>;
2089:
2090: def PackedVectorFormats : GenericEnum, Operand<i32> {
2091:   let FilterClass = "PackedVectorFormats";
2092:   let NameField = "Name";
2093:   let ValueField = "Value";
2094:   let PrintMethod = !strconcat("printSymbolicOperand<OperandCategory::", FilterClass, "Operand>");
2095: }
2096:
2097: class PackedVectorFormats<string name, bits<32> value> {
2098:   string Name = name;
2099:   bits<32> Value = value;
2100: }
2101:
2102: multiclass PackedVectorFormatsOperand<bits<32> value, list<Extension> reqExtensions> {
2103:   def NAME : BuiltIn<NAME, value>;
2104:   defm : SymbolicOperandWithRequirements<PackedVectorFormatsOperand, value, NAME, 0, 0, reqExtensions, [], []>;
2105: }
2106:
2107: defm PackedVectorFormat4x8Bit : PackedVectorFormatsOperand<0, [SPV_KHR_integer_dot_product]>;
```
- EN: This range uses TableGen DSL to describe records such as MatrixMultiplyAccumulateOperandsOperand, None, MatrixASignedComponentsINTEL, MatrixBSignedComponentsINTEL; LLVM later expands these declarations into generated backend tables and helper code.
- CN: 这一段使用 TableGen DSL 描述 MatrixMultiplyAccumulateOperandsOperand、None、MatrixASignedComponentsINTEL、MatrixBSignedComponentsINTEL 等记录；LLVM 后续会把这些声明展开为生成的后端表和辅助代码。

## Key Concepts / 关键概念

- EN: TableGen files use declarative records that LLVM expands into generated C++ tables and helper code.
  - CN: TableGen 文件使用声明式记录，LLVM 会将其展开为生成的 C++ 表和辅助代码。
- EN: SPIR-V backend code must preserve rich type information and encode capabilities/extensions explicitly.
  - CN: SPIR-V 后端需要保留丰富的类型信息，并显式编码 capability 与 extension。
- EN: Key symbols in this file include OperandCategory;, SymbolicOperand, SymbolicOperands, lookupSymbolicOperandByCategory, lookupSymbolicOperandByCategoryAndMnemonic, which anchor the file's main abstractions.
  - CN: 该文件中的关键符号包括 OperandCategory;, SymbolicOperand, SymbolicOperands, lookupSymbolicOperandByCategory, lookupSymbolicOperandByCategoryAndMnemonic，它们构成了本文件的核心抽象。

## Dependencies / 依赖关系

- LLVM infrastructure / LLVM 基础设施:
  - `llvm/TableGen/SearchableTable.td`
- TableGen pipeline / TableGen 流水线: records in this file are consumed by LLVM TableGen emitters to produce generated lookup tables and helper code.
- TableGen 流水线说明：该文件中的记录会被 LLVM 的 TableGen 生成器消费，进而产出查找表和辅助代码。
