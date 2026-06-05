# fde.h — Code Analysis / 代码分析

## Source / 来源
- File / 文件: `torch/csrc/profiler/unwind/fde.h`
- Repository / 仓库: `pytorch`
- Purpose / 作用:
  - EN: Implements native stack unwinding and symbolization helpers for profiling.
  - CN: 实现用于 profiling 的原生栈展开与符号化辅助工具。

## Line-by-Line Analysis / 逐行分析

### Lines 1-48
```cpp
 1 | #pragma once
 2 | #include <c10/util/Exception.h>
 3 | #include <c10/util/irange.h>
 4 | #include <torch/csrc/profiler/unwind/action.h>
 5 | #include <torch/csrc/profiler/unwind/lexer.h>
 6 | #include <array>
 7 | #include <iostream>
 8 | #include <sstream>
 9 | #include <vector>
10 | 
11 | namespace torch::unwind {
12 | 
13 | struct TableState {
14 |   Action cfa;
15 |   std::array<Action, D_REG_SIZE> registers;
16 |   friend std::ostream& operator<<(std::ostream& out, const TableState& self) {
17 |     out << "cfa = " << self.cfa << "; ";
18 |     for (auto r : c10::irange(self.registers.size())) {
19 |       if (self.registers.at(r).kind != A_UNDEFINED) {
20 |         out << 'r' << r << " = " << self.registers.at(r) << "; ";
21 |       }
22 |     }
23 |     return out;
24 |   }
25 | };
26 | 
27 | // FDE - Frame Description Entry (Concept in ELF spec)
28 | // This format is explained well by
29 | // https://www.airs.com/blog/archives/460
30 | // Details of different dwarf actions are explained
31 | // in the spec document:
32 | // https://web.archive.org/web/20221129184704/https://dwarfstd.org/doc/DWARF4.doc
33 | // An overview of how DWARF unwinding works is given in
34 | // https://dl.acm.org/doi/pdf/10.1145/3360572
35 | // A similar implementation written in rust is:
36 | // https://github.com/mstange/framehop/
37 | 
38 | template <bool LOG = false>
39 | struct FDE {
40 |   FDE(void* data, const char* library_name, uint64_t load_bias)
41 |       : library_name_(library_name), load_bias_(load_bias) {
42 |     Lexer L(data);
43 |     auto length = L.read4or8Length();
44 |     void* fde_start = L.loc();
45 |     // NOLINTNEXTLINE(performance-no-int-to-ptr)
46 |     void* cie_data = (void*)((int64_t)fde_start - L.read<uint32_t>());
47 |     Lexer LC(cie_data);
48 |     auto cie_length = LC.read4or8Length();
```
- EN: Brings in project headers such as `<c10/util/Exception.h>`, `<c10/util/irange.h>`, `<torch/csrc/profiler/unwind/action.h>`, `<torch/csrc/profiler/unwind/lexer.h>` and system or third-party headers such as `<array>`, `<iostream>`, `<sstream>`, `<vector>` so this section can use their types, APIs, or macros. Uses preprocessor directives to guard compilation, platform branches, or exported declarations. Places the implementation in namespace scopes (`torch::unwind`) so ownership matches the PyTorch subsystem layout. Defines or extends data abstractions such as `TableState`, `FDE` that structure the state handled by this file.
- CN: 这里引入了项目头文件，例如 `<c10/util/Exception.h>`、`<c10/util/irange.h>`、`<torch/csrc/profiler/unwind/action.h>`、`<torch/csrc/profiler/unwind/lexer.h>`以及系统或第三方头文件，例如 `<array>`、`<iostream>`、`<sstream>`、`<vector>`，使本段代码能够使用相关类型、API 或宏。 使用预处理指令来控制编译开关、平台分支或导出声明。 把实现放入命名空间作用域（`torch::unwind`），使其归属与 PyTorch 子系统布局保持一致。 定义或扩展了 `TableState`、`FDE` 等数据抽象，用来组织本文件处理的状态。

### Lines 49-102
```cpp
 49 |     void* cie_start = LC.loc();
 50 |     auto zero = LC.read<uint32_t>();
 51 |     TORCH_INTERNAL_ASSERT(zero == 0, "expected 0 for CIE");
 52 |     auto version = LC.read<uint8_t>();
 53 |     TORCH_INTERNAL_ASSERT(
 54 |         version == 1 || version == 3, "non-1 version for CIE");
 55 |     augmentation_string_ = LC.readCString();
 56 |     if (hasAugmentation("eh")) {
 57 |       throw UnwindError("unsupported 'eh' augmentation string");
 58 |     }
 59 |     code_alignment_factor_ = static_cast<int64_t>(LC.readULEB128());
 60 |     data_alignment_factor_ = LC.readSLEB128();
 61 |     if (version == 1) {
 62 |       ra_register_ = LC.read<uint8_t>();
 63 |     } else {
 64 |       ra_register_ = static_cast<int64_t>(LC.readULEB128());
 65 |     }
 66 |     TORCH_INTERNAL_ASSERT(
 67 |         ra_register_ == D_EXPECTED_RA_REG,
 68 |         "unexpected ra register: ",
 69 |         ra_register_);
 70 |     if (augmentation_string_ && *augmentation_string_ == 'z') {
 71 |       augmentation_length_ = static_cast<int64_t>(LC.readULEB128());
 72 |       Lexer A(LC.loc());
 73 |       for (auto ap = augmentation_string_ + 1; *ap; ap++) {
 74 |         switch (*ap) {
 75 |           case 'L':
 76 |             lsda_enc = A.read<uint8_t>();
 77 |             break;
 78 |           case 'R':
 79 |             fde_enc = A.read<uint8_t>();
 80 |             break;
 81 |           case 'P': {
 82 |             uint8_t personality_enc = A.read<uint8_t>();
 83 |             A.readEncoded(personality_enc);
 84 |           } break;
 85 |           case 'S': {
 86 |             // signal handler
 87 |           } break;
 88 |           default: {
 89 |             throw UnwindError("unknown augmentation string");
 90 |           } break;
 91 |         }
 92 |       }
 93 |     }
 94 |     LC.skip(augmentation_length_);
 95 |     low_pc_ = L.readEncoded(fde_enc);
 96 |     high_pc_ = low_pc_ + L.readEncodedValue(fde_enc);
 97 | 
 98 |     if (hasAugmentation("z")) {
 99 |       augmentation_length_fde_ = static_cast<int64_t>(L.readULEB128());
100 |     }
101 |     L.readEncodedOr(lsda_enc, 0);
102 | 
```
- EN: Implements routines such as `UnwindError`, `A` that expose the key API or control flow of this region.
- CN: 实现了 `UnwindError`、`A` 等例程，它们构成了这一段的关键 API 或控制流程。

### Lines 103-153
```cpp
103 |     cie_begin_ = LC.loc();
104 |     fde_begin_ = L.loc();
105 |     cie_end_ = (void*)((const char*)cie_start + cie_length);
106 |     fde_end_ = (void*)((const char*)fde_start + length);
107 |   }
108 | 
109 |   // OP Code implementations
110 | 
111 |   void advance_raw(int64_t amount) {
112 |     auto previous_pc = current_pc_;
113 |     current_pc_ += amount;
114 |     if (LOG) {
115 |       (*out_) << (void*)(previous_pc - load_bias_) << '-'
116 |               << (void*)(current_pc_ - load_bias_) << ": " << state() << '\n';
117 |     }
118 |   }
119 | 
120 |   void advance_loc(int64_t amount) {
121 |     if (LOG) {
122 |       (*out_) << "advance_loc " << amount << '\n';
123 |     }
124 |     advance_raw(amount * code_alignment_factor_);
125 |   }
126 | 
127 |   void offset(int64_t reg, int64_t offset) {
128 |     if (LOG) {
129 |       (*out_) << "offset " << reg << ' ' << offset << '\n';
130 |     }
131 |     if (reg > (int64_t)state().registers.size()) {
132 |       if (LOG) {
133 |         (*out_) << "OFFSET OF BIG REGISTER " << reg << "ignored...\n";
134 |       }
135 |       return;
136 |     }
137 |     state().registers.at(reg) =
138 |         Action{A_LOAD_CFA_OFFSET, -1, offset * data_alignment_factor_};
139 |   }
140 | 
141 |   void restore(int64_t reg) {
142 |     if (LOG) {
143 |       (*out_) << "restore " << reg << '\n';
144 |     }
145 |     if (reg > (int64_t)state().registers.size()) {
146 |       if (LOG) {
147 |         (*out_) << "RESTORE OF BIG REGISTER " << reg << "ignored...\n";
148 |       }
149 |       return;
150 |     }
151 |     state().registers.at(reg) = initial_state_.registers.at(reg);
152 |   }
153 | 
```
- EN: Implements routines such as `advance_raw`, `advance_loc`, `offset`, `restore` that expose the key API or control flow of this region. At the statement level, this block returns computed state or forwards values to callers.
- CN: 实现了 `advance_raw`、`advance_loc`、`offset`、`restore` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段返回计算状态或把值转交给调用方。

### Lines 154-199
```cpp
154 |   void def_cfa(int64_t reg, int64_t off) {
155 |     if (LOG) {
156 |       (*out_) << "def_cfa " << reg << ' ' << off << '\n';
157 |     }
158 |     last_reg_ = reg;
159 |     last_offset_ = off;
160 |     state().cfa = Action::regPlusData(static_cast<int32_t>(reg), off);
161 |   }
162 |   void def_cfa_register(int64_t reg) {
163 |     def_cfa(reg, last_offset_);
164 |   }
165 |   void def_cfa_offset(int64_t off) {
166 |     def_cfa(last_reg_, off);
167 |   }
168 | 
169 |   void remember_state() {
170 |     if (LOG) {
171 |       (*out_) << "remember_state\n";
172 |     }
173 |     state_stack_.push_back(state());
174 |   }
175 |   void restore_state() {
176 |     if (LOG) {
177 |       (*out_) << "restore_state\n";
178 |     }
179 |     state_stack_.pop_back();
180 |   }
181 | 
182 |   void undefined(int64_t reg) {
183 |     if (LOG) {
184 |       (*out_) << "undefined " << reg << '\n';
185 |     }
186 |     state().registers.at(reg) = Action::undefined();
187 |   }
188 |   void register_(int64_t reg, int64_t rhs_reg) {
189 |     if (LOG) {
190 |       (*out_) << "register " << reg << ' ' << rhs_reg << '\n';
191 |     }
192 |     state().registers.at(reg) =
193 |         Action::regPlusData(static_cast<int32_t>(reg), 0);
194 |   }
195 | 
196 |   TableState& state() {
197 |     return state_stack_.back();
198 |   }
199 | 
```
- EN: Implements routines such as `def_cfa`, `def_cfa_register`, `def_cfa_offset`, `remember_state`, `restore_state` that expose the key API or control flow of this region. At the statement level, this block returns computed state or forwards values to callers.
- CN: 实现了 `def_cfa`、`def_cfa_register`、`def_cfa_offset`、`remember_state`、`restore_state` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段返回计算状态或把值转交给调用方。

### Lines 200-253
```cpp
200 |   void dump(std::ostream& out) {
201 |     out_ = &out;
202 |     out << "FDE(augmentation_string=" << augmentation_string_
203 |         << ", low_pc=" << (void*)(low_pc_ - load_bias_)
204 |         << ",high_pc=" << (void*)(high_pc_ - load_bias_)
205 |         << ",code_alignment_factor=" << code_alignment_factor_
206 |         << ", data_alignment_factor=" << data_alignment_factor_
207 |         << ", ra_register_=" << ra_register_ << ")\n";
208 |     readUpTo(high_pc_);
209 |     out_ = &std::cout;
210 |   }
211 | 
212 |   TableState readUpTo(uint64_t addr) {
213 |     if (addr < low_pc_ || addr > high_pc_) {
214 |       throw UnwindError("Address not in range");
215 |     }
216 |     if (LOG) {
217 |       // NOLINTNEXTLINE(performance-no-int-to-ptr)
218 |       (*out_) << "readUpTo " << (void*)addr << " for " << library_name_
219 |               << " at " << (void*)load_bias_ << '\n';
220 |     }
221 |     state_stack_.emplace_back();
222 |     current_pc_ = low_pc_;
223 |     // parse instructions...
224 |     Lexer LC(cie_begin_);
225 |     while (LC.loc() < cie_end_ && current_pc_ <= addr) {
226 |       readInstruction(LC);
227 |     }
228 |     if (current_pc_ > addr) {
229 |       return state();
230 |     }
231 | 
232 |     initial_state_ = state_stack_.back();
233 | 
234 |     if (LOG) {
235 |       (*out_) << "--\n";
236 |     }
237 | 
238 |     Lexer L(fde_begin_);
239 |     while (L.loc() < fde_end_ && current_pc_ <= addr) {
240 |       readInstruction(L);
241 |     }
242 |     // so that we print the full range in debugging
243 |     if (current_pc_ <= addr) {
244 |       advance_raw(addr - current_pc_);
245 |     }
246 |     return state();
247 |   }
248 | 
249 |   void dumpAddr2Line() {
250 |     std::cout << "addr2line -f -e " << library_name_ << ' '
251 |               << (void*)(low_pc_ - load_bias_) << '\n';
252 |   }
253 | 
```
- EN: Implements routines such as `dump`, `readUpTo`, `UnwindError`, `LC`, `state` that expose the key API or control flow of this region. At the statement level, this block returns computed state or forwards values to callers.
- CN: 实现了 `dump`、`readUpTo`、`UnwindError`、`LC`、`state` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段返回计算状态或把值转交给调用方。

### Lines 254-301
```cpp
254 |   void readInstruction(Lexer& L) {
255 |     uint8_t bc = L.read<uint8_t>();
256 |     auto op = bc >> 6;
257 |     auto lowbits = bc & 0x3F;
258 |     switch (op) {
259 |       case 0x0: {
260 |         switch (lowbits) {
261 |           case DW_CFA_nop: {
262 |             return; // nop
263 |           }
264 |           case DW_CFA_advance_loc1: {
265 |             auto delta = L.read<uint8_t>();
266 |             return advance_loc(delta);
267 |           }
268 |           case DW_CFA_advance_loc2: {
269 |             auto delta = L.read<uint16_t>();
270 |             return advance_loc(delta);
271 |           }
272 |           case DW_CFA_advance_loc4: {
273 |             auto delta = L.read<uint32_t>();
274 |             return advance_loc(delta);
275 |           }
276 |           case DW_CFA_offset_extended: {
277 |             auto reg = L.readULEB128();
278 |             auto off = L.readULEB128();
279 |             return offset(reg, off);
280 |           }
281 |           case DW_CFA_restore_extended: {
282 |             auto reg = L.readULEB128();
283 |             return restore(reg);
284 |           }
285 |           case DW_CFA_undefined: {
286 |             auto reg = L.readULEB128();
287 |             return undefined(reg);
288 |           }
289 |           case DW_CFA_register: {
290 |             auto reg = L.readULEB128();
291 |             auto rhs_reg = L.readULEB128();
292 |             return register_(reg, rhs_reg);
293 |           }
294 |           case DW_CFA_def_cfa: {
295 |             auto reg = L.readULEB128();
296 |             auto off = L.readULEB128();
297 |             return def_cfa(reg, off);
298 |           }
299 |           case DW_CFA_def_cfa_register: {
300 |             auto reg = L.readULEB128();
301 |             return def_cfa_register(reg);
```
- EN: Implements routines such as `readInstruction`, `advance_loc`, `offset`, `restore`, `undefined` that expose the key API or control flow of this region. At the statement level, this block returns computed state or forwards values to callers.
- CN: 实现了 `readInstruction`、`advance_loc`、`offset`、`restore`、`undefined` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段返回计算状态或把值转交给调用方。

### Lines 302-349
```cpp
302 |           }
303 |           case DW_CFA_def_cfa_offset: {
304 |             auto off = L.readULEB128();
305 |             return def_cfa_offset(off);
306 |           }
307 |           case DW_CFA_offset_extended_sf: {
308 |             auto reg = L.readULEB128();
309 |             auto off = L.readSLEB128();
310 |             return offset(reg, off);
311 |           }
312 |           case DW_CFA_remember_state: {
313 |             return remember_state();
314 |           }
315 |           case DW_CFA_restore_state: {
316 |             return restore_state();
317 |           }
318 |           case DW_CFA_GNU_args_size: {
319 |             // GNU_args_size, we do not need to know it..
320 |             L.readULEB128();
321 |             return;
322 |           }
323 |           case DW_CFA_expression: {
324 |             auto reg = L.readULEB128();
325 |             auto len = L.readULEB128();
326 |             // NOLINTNEXTLINE(performance-no-int-to-ptr)
327 |             auto end = (void*)((uint64_t)L.loc() + len);
328 |             auto op = L.read<uint8_t>();
329 |             if ((op & 0xF0) == 0x70) { // DW_bregX
330 |               auto rhs_reg = (op & 0xF);
331 |               auto addend = L.readSLEB128();
332 |               if (L.loc() == end) {
333 |                 state().registers.at(reg) =
334 |                     Action::regPlusDataDeref(rhs_reg, addend);
335 |                 return;
336 |               }
337 |             }
338 |             throw UnwindError("Unsupported dwarf expression");
339 |           }
340 |           case DW_CFA_def_cfa_expression: {
341 |             auto len = L.readULEB128();
342 |             // NOLINTNEXTLINE(performance-no-int-to-ptr)
343 |             auto end = (void*)((uint64_t)L.loc() + len);
344 |             auto op = L.read<uint8_t>();
345 |             if ((op & 0xF0) == 0x70) { // DW_bregX
346 |               auto rhs_reg = (op & 0xF);
347 |               auto addend = L.readSLEB128();
348 |               if (L.loc() != end) {
349 |                 auto op2 = L.read<uint8_t>();
```
- EN: Implements routines such as `def_cfa_offset`, `offset`, `remember_state`, `restore_state`, `UnwindError` that expose the key API or control flow of this region. At the statement level, this block returns computed state or forwards values to callers.
- CN: 实现了 `def_cfa_offset`、`offset`、`remember_state`、`restore_state`、`UnwindError` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段返回计算状态或把值转交给调用方。

### Lines 350-401
```cpp
350 |                 if (op2 == DW_OP_deref && L.loc() == end) { // deref
351 |                   state().cfa = Action::regPlusDataDeref(rhs_reg, addend);
352 |                   return;
353 |                 }
354 |               }
355 |             }
356 |             throw UnwindError("Unsupported def_cfa dwarf expression");
357 |           }
358 |           default: {
359 |             std::stringstream ss;
360 |             // NOLINTNEXTLINE(performance-no-int-to-ptr)
361 |             ss << "unknown op code " << (void*)(uint64_t)lowbits;
362 |             throw UnwindError(ss.str());
363 |           }
364 |         }
365 |       }
366 |       case DW_CFA_advance_loc: {
367 |         return advance_loc(lowbits);
368 |       }
369 |       case DW_CFA_offset: {
370 |         auto off = L.readULEB128();
371 |         return offset(lowbits, off);
372 |       }
373 |       case DW_CFA_restore: {
374 |         return restore(lowbits);
375 |       }
376 |     }
377 |   }
378 |   // used for debug printing
379 |   const char* library_name_;
380 |   uint64_t load_bias_;
381 | 
382 |   // parsed from the eh_string data structures:
383 |   const char* augmentation_string_ = nullptr;
384 |   int64_t augmentation_length_ = 0;
385 |   int64_t augmentation_length_fde_ = 0;
386 | 
387 |   int64_t code_alignment_factor_;
388 |   int64_t data_alignment_factor_;
389 |   void* cie_data_{nullptr};
390 | 
391 |   int64_t ra_register_;
392 |   uint8_t lsda_enc = DW_EH_PE_omit;
393 |   uint8_t fde_enc = DW_EH_PE_absptr;
394 |   uint64_t low_pc_ = UINT64_MAX;
395 |   uint64_t high_pc_ = UINT64_MAX;
396 | 
397 |   void* cie_begin_;
398 |   void* fde_begin_;
399 |   void* cie_end_;
400 |   void* fde_end_;
401 | 
```
- EN: Implements routines such as `UnwindError`, `advance_loc`, `offset`, `restore` that expose the key API or control flow of this region. At the statement level, this block declares abstract hooks that derived backends must provide; returns computed state or forwards values to callers; stores long-lived member state for later calls.
- CN: 实现了 `UnwindError`、`advance_loc`、`offset`、`restore` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段声明了需要由派生后端提供的抽象钩子；返回计算状态或把值转交给调用方；保存供后续调用使用的长期成员状态。

### Lines 402-418
```cpp
402 |   // state accumulated while parsing instructions
403 |   int64_t last_reg_ = 0;
404 |   int64_t last_offset_ = 0;
405 |   uint64_t current_pc_ = 0;
406 | 
407 |   TableState
408 |       initial_state_; // state after the initial instructions, used by restore
409 |   std::vector<TableState> state_stack_;
410 | 
411 |   std::ostream* out_ = &std::cout; // for debug dumping
412 |  private:
413 |   bool hasAugmentation(const char* s) {
414 |     return strstr(augmentation_string_, s) != nullptr;
415 |   }
416 | };
417 | 
418 | } // namespace torch::unwind
```
- EN: Implements routines such as `hasAugmentation` that expose the key API or control flow of this region. At the statement level, this block declares abstract hooks that derived backends must provide; returns computed state or forwards values to callers; stores long-lived member state for later calls.
- CN: 实现了 `hasAugmentation` 等例程，它们构成了这一段的关键 API 或控制流程。 在语句层面，这一段声明了需要由派生后端提供的抽象钩子；返回计算状态或把值转交给调用方；保存供后续调用使用的长期成员状态。

## Key Concepts / 关键概念

- **Primary types / 主要类型**
  - EN: `TableState`, `FDE`.
  - CN: `TableState`、`FDE`。
- **Important routines / 重要例程**
  - EN: `L`, `LC`, `UnwindError`, `A`, `advance_raw`, `advance_loc`, `offset`, `restore`.
  - CN: `L`、`LC`、`UnwindError`、`A`、`advance_raw`、`advance_loc`、`offset`、`restore`。
- **Subsystem scope / 子系统作用域**
  - EN: The code lives under `torch::unwind`, indicating the owning PyTorch subsystem.
  - CN: 代码位于 `torch::unwind` 命名空间下，这说明了其所属的 PyTorch 子系统。

## Dependencies / 依赖关系

- Local includes / 本地头文件: `<c10/util/Exception.h>`, `<c10/util/irange.h>`, `<torch/csrc/profiler/unwind/action.h>`, `<torch/csrc/profiler/unwind/lexer.h>`
- External includes / 外部头文件: `<array>`, `<iostream>`, `<sstream>`, `<vector>`
- Relationship / 关系:
  - EN: The unit relies on neighboring PyTorch headers for subsystem-specific types, dispatcher hooks, and utility helpers. Standard-library or third-party headers provide generic facilities such as containers, threading, math, or backend integration points. Its exported or visible routines are then consumed by callers in the same namespace and by higher-level runtime paths elsewhere in PyTorch.
  - CN: 该编译单元依赖相邻的 PyTorch 头文件，以获得子系统类型、分发器钩子和辅助工具。 标准库或第三方头文件提供了通用能力，例如容器、线程、数学支持或后端集成入口。 随后，这里导出或可见的例程会被同一命名空间中的调用方，以及 PyTorch 其他位置的更高层运行时路径所使用。
