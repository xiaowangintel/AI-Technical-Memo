# NVPTXIntrinsics.td — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/NVPTX/NVPTXIntrinsics.td`
- Repository: `llvm-project`
- Purpose (EN): NVPTXIntrinsics support code for the LLVM target backend.
- 目的（中文）: 该文件使用 TableGen DSL 描述目标后端元数据，并驱动自动生成代码。
- Language: TableGen DSL (TableGen DSL noted below)
- Note: This file is written in TableGen DSL, so many records are declarative descriptions consumed by LLVM code generators.
- 说明：该文件使用 TableGen DSL 编写，因此许多记录都是供 LLVM 代码生成器消费的声明式描述。

## Line-by-Line Analysis / 逐行分析

> Note / 说明: To keep the document readable, the source is analyzed in contiguous line ranges while preserving the original order. Each code block prefixes original line numbers.

### Lines 1-140
```tablegen
  1: //===- NVPTXIntrinsics.td - PTX Intrinsics Instructions -------*- tblgen -*-==//
  2: //
  3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
  4: // See https://llvm.org/LICENSE.txt for license information.
  5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
  6: //
  7: //===----------------------------------------------------------------------===//
  8:
  9: def AS_match {
 10:   code generic = [{
 11:    return cast<MemSDNode>(N)->getAddressSpace() == llvm::ADDRESS_SPACE_GENERIC;
 12:   }];
 13:   code shared = [{
 14:    return cast<MemSDNode>(N)->getAddressSpace() == llvm::ADDRESS_SPACE_SHARED;
 15:   }];
 16:   code shared_cluster = [{
 17:    return cast<MemSDNode>(N)->getAddressSpace() == llvm::ADDRESS_SPACE_SHARED_CLUSTER;
 18:   }];
 19:   code global = [{
 20:    return cast<MemSDNode>(N)->getAddressSpace() == llvm::ADDRESS_SPACE_GLOBAL;
 21:   }];
 22:   code const = [{
 23:    return cast<MemSDNode>(N)->getAddressSpace() == llvm::ADDRESS_SPACE_CONST;
 24:   }];
 25:   code param = [{
 26:    return cast<MemSDNode>(N)->getAddressSpace() == llvm::ADDRESS_SPACE_ENTRY_PARAM;
 27:   }];
 28: }
 29:
 30:
 31: //===----------------------------------------------------------------------===//
 32: // NVPTX Scope Constants
 33: // These map to the Scope enum in NVPTX.h
 34: //===----------------------------------------------------------------------===//
 35:
 36: def Scope_thread : PatLeaf<(i32 0)>;      // Thread = 0
 37: def Scope_cta : PatLeaf<(i32 1)>;         // Block = 1
 38: def Scope_cluster : PatLeaf<(i32 2)>;     // Cluster = 2
 39: def Scope_device : PatLeaf<(i32 3)>;      // Device = 3
 40: def Scope_sys : PatLeaf<(i32 4)>;         // System = 4
 41:
 42: //===----------------------------------------------------------------------===//
 43: // NVPTX Address Space Constants
 44: // These map to the AddressSpace enum in NVPTX.h
 45: //===----------------------------------------------------------------------===//
 46:
 47: def AddrSpace_gen : PatLeaf<(i32 0)>;        // Generic = 0
 48: def AddrSpace_global : PatLeaf<(i32 1)>;         // Global = 1
 49: def AddrSpace_shared : PatLeaf<(i32 3)>;         // Shared = 3
 50: def AddrSpace_const : PatLeaf<(i32 4)>;          // Const = 4
 51: def AddrSpace_local : PatLeaf<(i32 5)>;          // Local = 5
 52: def AddrSpace_shared_cluster : PatLeaf<(i32 7)>;  // SharedCluster = 7
 53: def AddrSpace_param : PatLeaf<(i32 101)>;        // Param = 101
 54:
 55: //===----------------------------------------------------------------------===//
 56: // NVPTX Ordering Constants
 57: // These map to the Ordering enum in NVPTX.h
 58: //===----------------------------------------------------------------------===//
 59:
 60: def Ordering_not_atomic : PatLeaf<(i32 0)>;           // NotAtomic = 0
 61: def Ordering_relaxed : PatLeaf<(i32 2)>;             // Relaxed = 1
 62: def Ordering_acquire : PatLeaf<(i32 4)>;             // Acquire = 4
 63: def Ordering_release : PatLeaf<(i32 5)>;             // Release = 5
 64: def Ordering_acquire_release : PatLeaf<(i32 6)>;      // AcquireRelease = 6
 65: def Ordering_sequentially_consistent : PatLeaf<(i32 7)>; // SequentiallyConsistent = 7
 66: def Ordering_volatile : PatLeaf<(i32 8)>;            // Volatile = 8
 67: def Ordering_relaxed_mmio : PatLeaf<(i32 9)>;         // RelaxedMMIO = 9
 68:
 69:
 70: // A node that will be replaced with the current PTX version.
 71: class PTX {
 72:   SDNodeXForm PTXVerXform = SDNodeXForm<imm, [{
 73:     return getI32Imm(Subtarget->getPTXVersion(), SDLoc(N));
 74:   }]>;
 75:   // (i32 0) will be XForm'ed to the currently used PTX version.
 76:   dag version = (PTXVerXform (i32 0));
 77: }
 78: def ptx : PTX;
 79:
 80: // Generates list of n sequential register names.
 81: // E.g. RegNames<3, "r">.ret -> ["r0", "r1", "r2" ]
 82: class RegSeq<int n, string prefix> {
 83:   list<string> ret = !if(n, !listconcat(RegSeq<!sub(n, 1), prefix>.ret,
 84:                                         [prefix # !sub(n, 1)]),
 85:                             []);
 86: }
 87:
 88: //-----------------------------------
 89: // Synchronization and shuffle functions
 90: //-----------------------------------
 91: let isConvergent = true in {
 92:
 93: def INT_BAR_WARP_SYNC_I : BasicNVPTXInst<(outs), (ins i32imm:$i), "bar.warp.sync",
 94:                              [(int_nvvm_bar_warp_sync imm:$i)]>,
 95:         Requires<[hasPTX<60>, hasSM<30>]>;
 96: def INT_BAR_WARP_SYNC_R : BasicNVPTXInst<(outs), (ins B32:$i), "bar.warp.sync",
 97:                              [(int_nvvm_bar_warp_sync i32:$i)]>,
 98:         Requires<[hasPTX<60>, hasSM<30>]>;
 99:
100: multiclass BARRIER_ALL<string asmstr, Intrinsic intrinsic, list<Predicate> requires = []> {
101:   let Predicates = requires in {
102:     def _i : BasicNVPTXInst<(outs), (ins i32imm:$i), asmstr, [(intrinsic imm:$i)]>;
103:     def _r : BasicNVPTXInst<(outs), (ins B32:$i), asmstr, [(intrinsic i32:$i)]>;
104:   }
105: }
106:
107: multiclass BARRIER_COUNT<string asmstr, Intrinsic intrinsic, list<Predicate> requires = []> {
108:   let Predicates = requires in {
109:     def _rr : BasicNVPTXInst<(outs), (ins B32:$i, B32:$j), asmstr, 
110:                              [(intrinsic i32:$i, i32:$j)]>;
111:     def _ri : BasicNVPTXInst<(outs), (ins B32:$i, i32imm:$j), asmstr, 
112:                              [(intrinsic i32:$i, imm:$j)]>;
113:     def _ir : BasicNVPTXInst<(outs), (ins i32imm:$i, B32:$j), asmstr,
114:                              [(intrinsic imm:$i, i32:$j)]>;
115:     def _ii : BasicNVPTXInst<(outs), (ins i32imm:$i, i32imm:$j), asmstr,
116:                              [(intrinsic imm:$i, imm:$j)]>;
117:   }
118: }
119:
120: multiclass BARRIER_RED_ALL<string asmstr, Intrinsic intrinsic, RegTyInfo dst_rt, list<Predicate> requires = []> {
121:   let Predicates = requires in {
122:     def _ip : BasicNVPTXInst<(outs dst_rt.RC:$dest), (ins i32imm:$i, B1:$pred), asmstr,
123:                              [(set dst_rt.Ty:$dest, (intrinsic imm:$i, i1:$pred))]>;
124:     def _rp : BasicNVPTXInst<(outs dst_rt.RC:$dest), (ins B32:$i, B1:$pred), asmstr, 
125:                              [(set dst_rt.Ty:$dest, (intrinsic i32:$i, i1:$pred))]>;
126:   }
127: }
128:
129: multiclass BARRIER_RED_COUNT<string asmstr, Intrinsic intrinsic, RegTyInfo dst_rt, list<Predicate> requires = []> {
130:   let Predicates = requires in {
131:     def _rrp : BasicNVPTXInst<(outs dst_rt.RC:$dest), (ins B32:$i, B32:$j, B1:$pred), asmstr, 
132:                               [(set dst_rt.Ty:$dest, (intrinsic i32:$i, i32:$j, i1:$pred))]>;
133:     def _rip : BasicNVPTXInst<(outs dst_rt.RC:$dest), (ins B32:$i, i32imm:$j, B1:$pred), asmstr, 
134:                               [(set dst_rt.Ty:$dest, (intrinsic i32:$i, imm:$j, i1:$pred))]>;
135:     def _irp : BasicNVPTXInst<(outs dst_rt.RC:$dest), (ins i32imm:$i, B32:$j, B1:$pred), asmstr,
136:                               [(set dst_rt.Ty:$dest, (intrinsic imm:$i, i32:$j, i1:$pred))]>;
137:     def _iip : BasicNVPTXInst<(outs dst_rt.RC:$dest), (ins i32imm:$i, i32imm:$j, B1:$pred), asmstr,
138:                               [(set dst_rt.Ty:$dest, (intrinsic imm:$i, imm:$j, i1:$pred))]>;
139:   }
140: }
```
- EN: This range uses TableGen DSL to describe records such as AS_match, Scope_thread, Scope_cta, Scope_cluster; LLVM later expands these declarations into generated backend tables and helper code.
- CN: 这一段使用 TableGen DSL 描述 AS_match、Scope_thread、Scope_cta、Scope_cluster 等记录；LLVM 后续会把这些声明展开为生成的后端表和辅助代码。

### Lines 141-280
```tablegen
141:
142:
143: // Note the "bar.sync" variants could be renamed to the equivalent corresponding
144: // "barrier.*.aligned" variants. We use the older syntax for compatibility with
145: // older versions of the PTX ISA.
146: defm BARRIER_CTA_SYNC_ALIGNED_ALL : BARRIER_ALL<"bar.sync", int_nvvm_barrier_cta_sync_aligned_all>;
147: defm BARRIER_CTA_SYNC_ALIGNED : BARRIER_COUNT<"bar.sync", int_nvvm_barrier_cta_sync_aligned_count>;
148: defm BARRIER_CTA_ARRIVE_ALIGNED : BARRIER_COUNT<"bar.arrive", int_nvvm_barrier_cta_arrive_aligned_count>;
149:
150: defm BARRIER_CTA_SYNC_ALL : BARRIER_ALL<"barrier.sync", int_nvvm_barrier_cta_sync_all, [hasPTX<60>]>;
151: defm BARRIER_CTA_SYNC : BARRIER_COUNT<"barrier.sync", int_nvvm_barrier_cta_sync_count, [hasPTX<60>]>;
152: defm BARRIER_CTA_ARRIVE : BARRIER_COUNT<"barrier.arrive", int_nvvm_barrier_cta_arrive_count, [hasPTX<60>]>;
153:
154: defm BARRIER_CTA_RED_POPC_ALIGNED_ALL : BARRIER_RED_ALL<"bar.red.popc.u32", int_nvvm_barrier_cta_red_popc_aligned_all, I32RT>;
155: defm BARRIER_CTA_RED_AND_ALIGNED_ALL : BARRIER_RED_ALL<"bar.red.and.pred", int_nvvm_barrier_cta_red_and_aligned_all, I1RT>;
156: defm BARRIER_CTA_RED_OR_ALIGNED_ALL : BARRIER_RED_ALL<"bar.red.or.pred", int_nvvm_barrier_cta_red_or_aligned_all, I1RT>;
157: defm BARRIER_CTA_RED_POPC_ALIGNED : BARRIER_RED_COUNT<"bar.red.popc.u32", int_nvvm_barrier_cta_red_popc_aligned_count, I32RT>;
158: defm BARRIER_CTA_RED_AND_ALIGNED : BARRIER_RED_COUNT<"bar.red.and.pred", int_nvvm_barrier_cta_red_and_aligned_count, I1RT>;
159: defm BARRIER_CTA_RED_OR_ALIGNED : BARRIER_RED_COUNT<"bar.red.or.pred", int_nvvm_barrier_cta_red_or_aligned_count, I1RT>;
160:
161: defm BARRIER_CTA_RED_POPC_ALL : BARRIER_RED_ALL<"barrier.red.popc.u32", int_nvvm_barrier_cta_red_popc_all, I32RT, [hasPTX<60>]>;
162: defm BARRIER_CTA_RED_AND_ALL : BARRIER_RED_ALL<"barrier.red.and.pred", int_nvvm_barrier_cta_red_and_all, I1RT, [hasPTX<60>]>;
163: defm BARRIER_CTA_RED_OR_ALL : BARRIER_RED_ALL<"barrier.red.or.pred", int_nvvm_barrier_cta_red_or_all, I1RT, [hasPTX<60>]>;
164: defm BARRIER_CTA_RED_POPC_COUNT : BARRIER_RED_COUNT<"barrier.red.popc.u32", int_nvvm_barrier_cta_red_popc_count, I32RT, [hasPTX<60>]>;
165: defm BARRIER_CTA_RED_AND_COUNT : BARRIER_RED_COUNT<"barrier.red.and.pred", int_nvvm_barrier_cta_red_and_count, I1RT, [hasPTX<60>]>;
166: defm BARRIER_CTA_RED_OR_COUNT : BARRIER_RED_COUNT<"barrier.red.or.pred", int_nvvm_barrier_cta_red_or_count, I1RT, [hasPTX<60>]>;
167:
168: class INT_BARRIER_CLUSTER<string variant, Intrinsic Intr,
169:                           list<Predicate> Preds = [hasPTX<78>, hasSM<90>]>:
170:         BasicNVPTXInst<(outs), (ins), "barrier.cluster."# variant, [(Intr)]>,
171:         Requires<Preds>;
172:
173: def barrier_cluster_arrive:
174:         INT_BARRIER_CLUSTER<"arrive", int_nvvm_barrier_cluster_arrive>;
175: def barrier_cluster_arrive_relaxed:
176:         INT_BARRIER_CLUSTER<"arrive.relaxed",
177:         int_nvvm_barrier_cluster_arrive_relaxed, [hasPTX<80>, hasSM<90>]>;
178: def barrier_cluster_wait:
179:         INT_BARRIER_CLUSTER<"wait", int_nvvm_barrier_cluster_wait>;
180:
181: // 'aligned' versions of the cluster barrier intrinsics
182: def barrier_cluster_arrive_aligned:
183:         INT_BARRIER_CLUSTER<"arrive.aligned", int_nvvm_barrier_cluster_arrive_aligned>;
184: def barrier_cluster_arrive_relaxed_aligned:
185:         INT_BARRIER_CLUSTER<"arrive.relaxed.aligned",
186:         int_nvvm_barrier_cluster_arrive_relaxed_aligned, [hasPTX<80>, hasSM<90>]>;
187: def barrier_cluster_wait_aligned:
188:         INT_BARRIER_CLUSTER<"wait.aligned", int_nvvm_barrier_cluster_wait_aligned>;
189:
190: foreach sync = [false, true] in {
191:   foreach mode = ["up", "down", "bfly", "idx"] in {
192:     foreach regclass = ["i32", "f32"] in {
193:       foreach return_pred = [false, true] in {
194:         foreach offset_imm = [false, true] in {
195:           foreach mask_imm = [false, true] in {
196:             foreach threadmask_imm = !if(sync, [0, 1], [0]) in {
197:               defvar Intr = !cast<Intrinsic>("int_nvvm_shfl_"
198:                                 # !if(sync, "sync_", "")
199:                                 # mode
200:                                 # "_" # regclass
201:                                 # !if(return_pred, "p", ""));
202:               defvar InOperandList = !con(
203:                 (ins B32:$src),
204:                 !dag(ins, !if(offset_imm, [i32imm], [B32]), ["offset"]),
205:                 !dag(ins, !if(mask_imm, [i32imm], [B32]), ["mask"]),
206:                 !if(sync,
207:                     !dag(ins, !if(threadmask_imm, [i32imm], [B32]), ["threadmask"]),
208:                     (ins)));
209:               defvar Pattern = !con(
210:                   (set B32:$dst),
211:                   !if(return_pred, (set B1:$pred), (set)),
212:                   (set !con(
213:                     !if(sync,
214:                       !dag(Intr, !if(threadmask_imm, [imm], [B32]), ["threadmask"]),
215:                       (Intr)),
216:                     (Intr B32:$src),
217:                     !dag(Intr, !if(offset_imm, [imm], [B32]), ["offset"]),
218:                     !dag(Intr, !if(mask_imm, [imm], [B32]), ["mask"]))));
219:
220:               def : BasicNVPTXInst<
221:                       !if(return_pred, (outs B32:$dst, B1:$pred),
222:                                        (outs B32:$dst)),
223:                       InOperandList,
224:                       "shfl." # !if(sync, "sync.", "") # mode # ".b32",
225:                       [Pattern]>,
226:                     Requires<!if(sync, [hasSM<30>, hasPTX<60>], [hasSM<30>, hasSHFL])>;
227:             }
228:           }
229:         }
230:       }
231:     }
232:   }
233: }
234:
235: // vote.{all,any,uni,ballot}
236: let Predicates = [hasPTX<60>, hasSM<30>] in {
237:   multiclass VOTE<string mode, RegTyInfo t, Intrinsic op> {
238:     def : BasicNVPTXInst<(outs t.RC:$dest), (ins B1:$pred),
239:                 "vote." # mode # "." # t.PtxType,
240:                 [(set t.Ty:$dest, (op i1:$pred))]>;
241:   }
242:
243:   defm VOTE_ALL : VOTE<"all", I1RT, int_nvvm_vote_all>;
244:   defm VOTE_ANY : VOTE<"any", I1RT, int_nvvm_vote_any>;
245:   defm VOTE_UNI : VOTE<"uni", I1RT, int_nvvm_vote_uni>;
246:   defm VOTE_BALLOT : VOTE<"ballot", I32RT, int_nvvm_vote_ballot>;
247:
248:   // vote.sync.{all,any,uni,ballot}
249:   multiclass VOTE_SYNC<string mode, RegTyInfo t, Intrinsic op> {
250:     def i : BasicNVPTXInst<(outs t.RC:$dest), (ins B1:$pred, i32imm:$mask),
251:                 "vote.sync." # mode # "." # t.PtxType,
252:                 [(set t.Ty:$dest, (op imm:$mask, i1:$pred))]>;
253:     def r : BasicNVPTXInst<(outs t.RC:$dest), (ins B1:$pred, B32:$mask),
254:                 "vote.sync." # mode # "." # t.PtxType,
255:                 [(set t.Ty:$dest, (op i32:$mask, i1:$pred))]>;
256:   }
257:
258:   defm VOTE_SYNC_ALL : VOTE_SYNC<"all", I1RT, int_nvvm_vote_all_sync>;
259:   defm VOTE_SYNC_ANY : VOTE_SYNC<"any", I1RT, int_nvvm_vote_any_sync>;
260:   defm VOTE_SYNC_UNI : VOTE_SYNC<"uni", I1RT, int_nvvm_vote_uni_sync>;
261:   defm VOTE_SYNC_BALLOT : VOTE_SYNC<"ballot", I32RT, int_nvvm_vote_ballot_sync>;
262: }
263: // elect.sync
264: let Predicates = [hasPTX<80>, hasSM<90>] in {
265: def INT_ELECT_SYNC_I : BasicNVPTXInst<(outs B32:$dest, B1:$pred), (ins i32imm:$mask),
266:             "elect.sync",
267:             [(set i32:$dest, i1:$pred, (int_nvvm_elect_sync imm:$mask))]>;
268: def INT_ELECT_SYNC_R : BasicNVPTXInst<(outs B32:$dest, B1:$pred), (ins B32:$mask),
269:             "elect.sync",
270:             [(set i32:$dest, i1:$pred, (int_nvvm_elect_sync i32:$mask))]>;
271: }
272:
273: let Predicates = [hasPTX<60>, hasSM<70>] in {
274:   multiclass MATCH_ANY_SYNC<Intrinsic op, RegTyInfo t> {
275:     def ii : BasicNVPTXInst<(outs B32:$dest), (ins t.Imm:$value, i32imm:$mask),
276:                 "match.any.sync." # t.PtxType,
277:                 [(set i32:$dest, (op imm:$mask, imm:$value))]>;
278:     def ir : BasicNVPTXInst<(outs B32:$dest), (ins t.Imm:$value, B32:$mask),
279:                 "match.any.sync." # t.PtxType,
280:                 [(set i32:$dest, (op i32:$mask, imm:$value))]>;
```
- EN: This range uses TableGen DSL to describe records such as BARRIER_CTA_SYNC_ALIGNED_ALL, BARRIER_CTA_SYNC_ALIGNED, BARRIER_CTA_ARRIVE_ALIGNED, BARRIER_CTA_SYNC_ALL; LLVM later expands these declarations into generated backend tables and helper code.
- CN: 这一段使用 TableGen DSL 描述 BARRIER_CTA_SYNC_ALIGNED_ALL、BARRIER_CTA_SYNC_ALIGNED、BARRIER_CTA_ARRIVE_ALIGNED、BARRIER_CTA_SYNC_ALL 等记录；LLVM 后续会把这些声明展开为生成的后端表和辅助代码。

### Lines 281-420
```tablegen
281:     def ri : BasicNVPTXInst<(outs B32:$dest), (ins t.RC:$value, i32imm:$mask),
282:                 "match.any.sync." # t.PtxType,
283:                 [(set i32:$dest, (op imm:$mask, t.Ty:$value))]>;
284:     def rr : BasicNVPTXInst<(outs B32:$dest), (ins t.RC:$value, B32:$mask),
285:                 "match.any.sync." # t.PtxType,
286:                 [(set i32:$dest, (op i32:$mask, t.Ty:$value))]>;
287:   }
288:
289:   defm MATCH_ANY_SYNC_32 : MATCH_ANY_SYNC<int_nvvm_match_any_sync_i32, I32RT>;
290:   defm MATCH_ANY_SYNC_64 : MATCH_ANY_SYNC<int_nvvm_match_any_sync_i64, I64RT>;
291:
292:   multiclass MATCH_ALLP_SYNC<RegTyInfo t, Intrinsic op> {
293:     def ii : BasicNVPTXInst<(outs B32:$dest, B1:$pred),
294:                       (ins t.Imm:$value, i32imm:$mask),
295:                 "match.all.sync." # t.PtxType,
296:                 [(set i32:$dest, i1:$pred, (op imm:$mask, imm:$value))]>;
297:     def ir : BasicNVPTXInst<(outs B32:$dest, B1:$pred),
298:                       (ins t.Imm:$value, B32:$mask),
299:                 "match.all.sync." # t.PtxType,
300:                 [(set i32:$dest, i1:$pred, (op i32:$mask, imm:$value))]>;
301:     def ri : BasicNVPTXInst<(outs B32:$dest, B1:$pred),
302:                       (ins t.RC:$value, i32imm:$mask),
303:                 "match.all.sync." # t.PtxType,
304:                 [(set i32:$dest, i1:$pred, (op imm:$mask, t.Ty:$value))]>;
305:     def rr : BasicNVPTXInst<(outs B32:$dest, B1:$pred),
306:                       (ins t.RC:$value, B32:$mask),
307:                 "match.all.sync." # t.PtxType,
308:                 [(set i32:$dest, i1:$pred, (op i32:$mask, t.Ty:$value))]>;
309:   }
310:   defm MATCH_ALLP_SYNC_32 : MATCH_ALLP_SYNC<I32RT, int_nvvm_match_all_sync_i32p>;
311:   defm MATCH_ALLP_SYNC_64 : MATCH_ALLP_SYNC<I64RT, int_nvvm_match_all_sync_i64p>;
312: }
313:
314: // activemask.b32
315: def ACTIVEMASK : BasicNVPTXInst<(outs B32:$dest), (ins),
316:                     "activemask.b32",
317:                     [(set i32:$dest, (int_nvvm_activemask))]>,
318:                  Requires<[hasPTX<62>, hasSM<30>]>;
319:
320: multiclass REDUX_SYNC<string BinOp, string PTXType, Intrinsic Intrin> {
321:   def : BasicNVPTXInst<(outs B32:$dst), (ins B32:$src, B32:$mask),
322:           "redux.sync." # BinOp # "." # PTXType,
323:           [(set i32:$dst, (Intrin i32:$src, B32:$mask))]>,
324:         Requires<[hasPTX<70>, hasSM<80>]>;
325: }
326:
327: defm REDUX_SYNC_UMIN : REDUX_SYNC<"min", "u32", int_nvvm_redux_sync_umin>;
328: defm REDUX_SYNC_UMAX : REDUX_SYNC<"max", "u32", int_nvvm_redux_sync_umax>;
329: defm REDUX_SYNC_ADD : REDUX_SYNC<"add", "s32", int_nvvm_redux_sync_add>;
330: defm REDUX_SYNC_MIN : REDUX_SYNC<"min", "s32", int_nvvm_redux_sync_min>;
331: defm REDUX_SYNC_MAX : REDUX_SYNC<"max", "s32", int_nvvm_redux_sync_max>;
332: defm REDUX_SYNC_AND : REDUX_SYNC<"and", "b32", int_nvvm_redux_sync_and>;
333: defm REDUX_SYNC_XOR : REDUX_SYNC<"xor", "b32", int_nvvm_redux_sync_xor>;
334: defm REDUX_SYNC_OR : REDUX_SYNC<"or", "b32", int_nvvm_redux_sync_or>;
335:
336: multiclass REDUX_SYNC_F<string BinOp, string abs, string NaN> {
337:   defvar intr_name = "int_nvvm_redux_sync_f" # BinOp # !subst(".", "_", abs) # !subst(".", "_", NaN);
338:
339:   def : BasicNVPTXInst<(outs B32:$dst),
340:                   (ins B32:$src, B32:$mask),
341:                   "redux.sync." # BinOp # abs # NaN # ".f32",
342:                   [(set f32:$dst, (!cast<Intrinsic>(intr_name) f32:$src, B32:$mask))]>,
343:                   Requires<[callSubtarget<"hasReduxSyncF32">]>;
344: }
345:
346: defm REDUX_SYNC_FMIN : REDUX_SYNC_F<"min", "", "">;
347: defm REDUX_SYNC_FMIN_ABS : REDUX_SYNC_F<"min", ".abs", "">;
348: defm REDUX_SYNC_FMIN_NAN: REDUX_SYNC_F<"min", "", ".NaN">;
349: defm REDUX_SYNC_FMIN_ABS_NAN: REDUX_SYNC_F<"min", ".abs", ".NaN">;
350: defm REDUX_SYNC_FMAX : REDUX_SYNC_F<"max", "", "">;
351: defm REDUX_SYNC_FMAX_ABS : REDUX_SYNC_F<"max", ".abs", "">;
352: defm REDUX_SYNC_FMAX_NAN: REDUX_SYNC_F<"max", "", ".NaN">;
353: defm REDUX_SYNC_FMAX_ABS_NAN: REDUX_SYNC_F<"max", ".abs", ".NaN">;
354:
355: } // isConvergent = true
356:
357: //-----------------------------------
358: // Explicit Memory Fence Functions
359: //-----------------------------------
360: class NullaryInst<string StrOp, Intrinsic IntOP> :
361:               BasicNVPTXInst<(outs), (ins), StrOp, [(IntOP)]>;
362:
363: def INT_MEMBAR_CTA : NullaryInst<"membar.cta", int_nvvm_membar_cta>;
364: def INT_MEMBAR_GL  : NullaryInst<"membar.gl",  int_nvvm_membar_gl>;
365: def INT_MEMBAR_SYS : NullaryInst<"membar.sys", int_nvvm_membar_sys>;
366:
367: def INT_FENCE_SC_CLUSTER:
368:        NullaryInst<"fence.sc.cluster", int_nvvm_fence_sc_cluster>,
369:        Requires<[hasPTX<78>, hasSM<90>]>;
370:
371: def INT_FENCE_MBARRIER_INIT_RELEASE_CLUSTER:
372:        NullaryInst<"fence.mbarrier_init.release.cluster",
373:         int_nvvm_fence_mbarrier_init_release_cluster>,
374:        Requires<[hasPTX<80>, hasSM<90>]>;
375:
376: let Predicates = [hasPTX<86>, hasSM<90>] in {
377: def INT_FENCE_ACQUIRE_SYNC_RESTRICT_CLUSTER_CLUSTER:
378:        NullaryInst<"fence.acquire.sync_restrict::shared::cluster.cluster",
379:         int_nvvm_fence_acquire_sync_restrict_space_cluster_scope_cluster>;
380:
381: def INT_FENCE_RELEASE_SYNC_RESTRICT_CTA_CLUSTER:
382:        NullaryInst<"fence.release.sync_restrict::shared::cta.cluster",
383:         int_nvvm_fence_release_sync_restrict_space_cta_scope_cluster>;
384: }
385:
386: // Proxy fence (uni-directional)
387: let Predicates = [hasPTX<86>, hasSM<90>] in {
388: def INT_NVVM_FENCE_PROXY_ASYNC_GENERIC_ACQUIRE_SYNC_RESTRICT_SPACE_CLUSTER_SCOPE_CLUSTER:
389:        NullaryInst<"fence.proxy.async::generic.acquire.sync_restrict::shared::cluster.cluster",
390:         int_nvvm_fence_proxy_async_generic_acquire_sync_restrict_space_cluster_scope_cluster>;
391:
392: def INT_NVVM_FENCE_PROXY_ASYNC_GENERIC_RELEASE_SYNC_RESTRICT_SPACE_CTA_SCOPE_CLUSTER:
393:        NullaryInst<"fence.proxy.async::generic.release.sync_restrict::shared::cta.cluster",
394:         int_nvvm_fence_proxy_async_generic_release_sync_restrict_space_cta_scope_cluster>;
395: }
396:
397: // Proxy fence (bi-directional)
398: foreach proxykind = ["alias", "async", "async.global", "async.shared_cta",
399:                       "async.shared_cluster"] in {
400:   defvar Preds = !if(!eq(proxykind, "alias"), [hasPTX<75>, hasSM<70>],
401:                                               [hasPTX<80>, hasSM<90>]);
402:   defvar Intr = IntrinsicName<"llvm.nvvm.fence.proxy." # proxykind>;
403:   def : NullaryInst<"fence.proxy." # !subst("_", "::", proxykind),
404:           !cast<Intrinsic>(Intr.record_name)>, Requires<Preds>;
405: }
406:
407: class FENCE_PROXY_TENSORMAP_GENERIC_RELEASE<string Scope, Intrinsic Intr> :
408:         NullaryInst<"fence.proxy.tensormap::generic.release." # Scope, Intr>,
409:         Requires<[hasPTX<83>, hasSM<90>]>;
410:
411: def INT_FENCE_PROXY_TENSORMAP_GENERIC_RELEASE_CTA:
412:       FENCE_PROXY_TENSORMAP_GENERIC_RELEASE<"cta",
413:         int_nvvm_fence_proxy_tensormap_generic_release_cta>;
414: def INT_FENCE_PROXY_TENSORMAP_GENERIC_RELEASE_CLUSTER:
415:       FENCE_PROXY_TENSORMAP_GENERIC_RELEASE<"cluster",
416:         int_nvvm_fence_proxy_tensormap_generic_release_cluster>;
417: def INT_FENCE_PROXY_TENSORMAP_GENERIC_RELEASE_GPU:
418:       FENCE_PROXY_TENSORMAP_GENERIC_RELEASE<"gpu",
419:         int_nvvm_fence_proxy_tensormap_generic_release_gpu>;
420: def INT_FENCE_PROXY_TENSORMAP_GENERIC_RELEASE_SYS:
```
- EN: This range uses TableGen DSL to describe records such as ri, rr, MATCH_ANY_SYNC_32, MATCH_ANY_SYNC_64; LLVM later expands these declarations into generated backend tables and helper code.
- CN: 这一段使用 TableGen DSL 描述 ri、rr、MATCH_ANY_SYNC_32、MATCH_ANY_SYNC_64 等记录；LLVM 后续会把这些声明展开为生成的后端表和辅助代码。

### Lines 421-560
```tablegen
421:       FENCE_PROXY_TENSORMAP_GENERIC_RELEASE<"sys",
422:         int_nvvm_fence_proxy_tensormap_generic_release_sys>;
423:
424: // fence.proxy.tensormap.acquire variants
425:
426: class FENCE_PROXY_TENSORMAP_GENERIC_ACQUIRE<string Scope, Intrinsic Intr> :
427:         NVPTXInst<(outs), (ins B64:$addr),
428:                   "fence.proxy.tensormap::generic.acquire." # Scope # " [$addr], 128;",
429:                   [(Intr i64:$addr, (i32 128))]>,
430:         Requires<[hasPTX<83>, hasSM<90>]>;
431:
432: def INT_FENCE_PROXY_TENSORMAP_GENERIC_ACQUIRE_CTA :
433:       FENCE_PROXY_TENSORMAP_GENERIC_ACQUIRE<"cta",
434:         int_nvvm_fence_proxy_tensormap_generic_acquire_cta>;
435: def INT_FENCE_PROXY_TENSORMAP_GENERIC_ACQUIRE_CLUSTER :
436:       FENCE_PROXY_TENSORMAP_GENERIC_ACQUIRE<"cluster",
437:         int_nvvm_fence_proxy_tensormap_generic_acquire_cluster>;
438: def INT_FENCE_PROXY_TENSORMAP_GENERIC_ACQUIRE_GPU :
439:       FENCE_PROXY_TENSORMAP_GENERIC_ACQUIRE<"gpu",
440:         int_nvvm_fence_proxy_tensormap_generic_acquire_gpu>;
441: def INT_FENCE_PROXY_TENSORMAP_GENERIC_ACQUIRE_SYS :
442:       FENCE_PROXY_TENSORMAP_GENERIC_ACQUIRE<"sys",
443:         int_nvvm_fence_proxy_tensormap_generic_acquire_sys>;
444:
445: //-----------------------------------
446: // Async Copy Functions
447: //-----------------------------------
448:
449: multiclass CP_ASYNC_MBARRIER_ARRIVE<string NoInc, string AddrSpace, Intrinsic Intrin> {
450:   def "" : BasicNVPTXInst<(outs), (ins ADDR:$addr),
451:             "cp.async.mbarrier.arrive" # NoInc # AddrSpace # ".b64",
452:             [(Intrin addr:$addr)]>,
453:     Requires<[hasPTX<70>, hasSM<80>]>;
454: }
455:
456: defm CP_ASYNC_MBARRIER_ARRIVE :
457:   CP_ASYNC_MBARRIER_ARRIVE<"", "", int_nvvm_cp_async_mbarrier_arrive>;
458: defm CP_ASYNC_MBARRIER_ARRIVE_SHARED :
459:   CP_ASYNC_MBARRIER_ARRIVE<"", ".shared", int_nvvm_cp_async_mbarrier_arrive_shared>;
460: defm CP_ASYNC_MBARRIER_ARRIVE_NOINC :
461:   CP_ASYNC_MBARRIER_ARRIVE<".noinc", "", int_nvvm_cp_async_mbarrier_arrive_noinc>;
462: defm CP_ASYNC_MBARRIER_ARRIVE_NOINC_SHARED :
463:   CP_ASYNC_MBARRIER_ARRIVE<".noinc", ".shared", int_nvvm_cp_async_mbarrier_arrive_noinc_shared>;
464:
465: multiclass CP_ASYNC_SHARED_GLOBAL_I<string cc, string cpsize, Intrinsic Intrin, Intrinsic IntrinS> {
466:   def "" : NVPTXInst<(outs), (ins ADDR:$dst, ADDR:$src),
467:             "cp.async." # cc # ".shared.global" # " [$dst], [$src], " # cpsize # ";",
468:             [(Intrin addr:$dst, addr:$src)]>,
469:     Requires<[hasPTX<70>, hasSM<80>]>;
470:
471:   // Variant with src_size parameter
472:   def _s : NVPTXInst<(outs), (ins ADDR:$dst, ADDR:$src, B32:$src_size),
473:              "cp.async." # cc # ".shared.global" # " [$dst], [$src], " # cpsize # ", $src_size;",
474:              [(IntrinS addr:$dst, addr:$src, i32:$src_size)]>,
475:     Requires<[hasPTX<70>, hasSM<80>]>;
476:   def _si: NVPTXInst<(outs), (ins ADDR:$dst, ADDR:$src, i32imm:$src_size),
477:              "cp.async." # cc # ".shared.global" # " [$dst], [$src], " # cpsize # ", $src_size;",
478:              [(IntrinS addr:$dst, addr:$src, imm:$src_size)]>,
479:     Requires<[hasPTX<70>, hasSM<80>]>;
480: }
481:
482: defm CP_ASYNC_CA_SHARED_GLOBAL_4 :
483:   CP_ASYNC_SHARED_GLOBAL_I<"ca", "4", int_nvvm_cp_async_ca_shared_global_4,
484:                                       int_nvvm_cp_async_ca_shared_global_4_s>;
485:
486: defm CP_ASYNC_CA_SHARED_GLOBAL_8 :
487:   CP_ASYNC_SHARED_GLOBAL_I<"ca", "8", int_nvvm_cp_async_ca_shared_global_8,
488:                                       int_nvvm_cp_async_ca_shared_global_8_s>;
489:
490: defm CP_ASYNC_CA_SHARED_GLOBAL_16 :
491:   CP_ASYNC_SHARED_GLOBAL_I<"ca", "16", int_nvvm_cp_async_ca_shared_global_16,
492:                                        int_nvvm_cp_async_ca_shared_global_16_s>;
493:
494: defm CP_ASYNC_CG_SHARED_GLOBAL_16 :
495:   CP_ASYNC_SHARED_GLOBAL_I<"cg", "16", int_nvvm_cp_async_cg_shared_global_16,
496:                                        int_nvvm_cp_async_cg_shared_global_16_s>;
497:
498: let Predicates = [hasPTX<70>, hasSM<80>] in {
499:   def CP_ASYNC_COMMIT_GROUP :
500:     NullaryInst<"cp.async.commit_group", int_nvvm_cp_async_commit_group>;
501:
502:   def CP_ASYNC_WAIT_GROUP :
503:     BasicNVPTXInst<(outs), (ins i32imm:$n), "cp.async.wait_group",
504:     [(int_nvvm_cp_async_wait_group timm:$n)]>;
505:
506:   def CP_ASYNC_WAIT_ALL :
507:     NullaryInst<"cp.async.wait_all", int_nvvm_cp_async_wait_all>;
508: }
509:
510: let Predicates = [hasPTX<80>, hasSM<90>] in {
511:   // cp.async.bulk variants of the commit/wait group
512:   def CP_ASYNC_BULK_COMMIT_GROUP :
513:     NullaryInst<"cp.async.bulk.commit_group", int_nvvm_cp_async_bulk_commit_group>;
514:
515:   def CP_ASYNC_BULK_WAIT_GROUP :
516:     BasicNVPTXInst<(outs), (ins i32imm:$n), "cp.async.bulk.wait_group",
517:     [(int_nvvm_cp_async_bulk_wait_group timm:$n)]>;
518:
519:   def CP_ASYNC_BULK_WAIT_GROUP_READ :
520:     BasicNVPTXInst<(outs), (ins i32imm:$n), "cp.async.bulk.wait_group.read",
521:     [(int_nvvm_cp_async_bulk_wait_group_read timm:$n)]>;
522: }
523:
524: //------------------------------
525: // TMA Async Bulk Copy Functions
526: //------------------------------
527:
528: class CpAsyncBulkStr<bit mc, bit ch, bit mask = 0> {
529:   // Shared to Global memory
530:   string S2G = "cp.async.bulk.global.shared::cta.bulk_group"
531:                # !if(ch, ".L2::cache_hint", "")
532:                # !if(mask, ".cp_mask", "");
533:
534:   // Global to Shared cluster memory
535:   string G2S = "cp.async.bulk.shared::cluster.global.mbarrier::complete_tx::bytes"
536:                # !if(mc, ".multicast::cluster", "")
537:                # !if(ch, ".L2::cache_hint", "");
538:
539:   // Global to Shared CTA memory
540:   string G2S_CTA = "cp.async.bulk.shared::cta.global.mbarrier::complete_tx::bytes"
541:                    # !if(ch, ".L2::cache_hint", "");
542:
543:   // Shared CTA to Cluster memory
544:   string C2C = "cp.async.bulk.shared::cluster.shared::cta.mbarrier::complete_tx::bytes";
545: }
546:
547: multiclass CP_ASYNC_BULK_S2G_INTR<bit has_ch> {
548:   def "" : NVPTXInst<(outs), (ins ADDR:$dst, ADDR:$src, B32:$size, B64:$ch),
549:       !if(has_ch,
550:           CpAsyncBulkStr<0, 1>.S2G # " [$dst], [$src], $size, $ch;",
551:           CpAsyncBulkStr<0, 0>.S2G # " [$dst], [$src], $size;"),
552:       [(int_nvvm_cp_async_bulk_shared_cta_to_global addr:$dst, addr:$src, i32:$size, i64:$ch, !if(has_ch, -1, 0))]>,
553:       Requires<[hasPTX<80>, hasSM<90>]>;
554:
555:   def _BM : NVPTXInst<(outs), (ins ADDR:$dst, ADDR:$src, B32:$size, B64:$ch, B16:$mask),
556:       !if(has_ch,
557:           CpAsyncBulkStr<0, 1, 1>.S2G # " [$dst], [$src], $size, $ch, $mask;",
558:           CpAsyncBulkStr<0, 0, 1>.S2G # " [$dst], [$src], $size, $mask;"),
559:       [(int_nvvm_cp_async_bulk_shared_cta_to_global_bytemask addr:$dst, addr:$src, i32:$size, i64:$ch, !if(has_ch, -1, 0), i16:$mask)]>,
560:       Requires<[hasPTX<86>, hasSM<100>]>;
```
- EN: This range uses TableGen DSL to describe records such as FENCE_PROXY_TENSORMAP_GENERIC_ACQUIRE, INT_FENCE_PROXY_TENSORMAP_GENERIC_ACQUIRE_CTA, INT_FENCE_PROXY_TENSORMAP_GENERIC_ACQUIRE_CLUSTER, INT_FENCE_PROXY_TENSORMAP_GENERIC_ACQUIRE_GPU; LLVM later expands these declarations into generated backend tables and helper code.
- CN: 这一段使用 TableGen DSL 描述 FENCE_PROXY_TENSORMAP_GENERIC_ACQUIRE、INT_FENCE_PROXY_TENSORMAP_GENERIC_ACQUIRE_CTA、INT_FENCE_PROXY_TENSORMAP_GENERIC_ACQUIRE_CLUSTER、INT_FENCE_PROXY_TENSORMAP_GENERIC_ACQUIRE_GPU 等记录；LLVM 后续会把这些声明展开为生成的后端表和辅助代码。

### Lines 561-700
```tablegen
561: }
562: defm CP_ASYNC_BULK_S2G    : CP_ASYNC_BULK_S2G_INTR<has_ch = 0>;
563: defm CP_ASYNC_BULK_S2G_CH : CP_ASYNC_BULK_S2G_INTR<has_ch = 1>;
564:
565: multiclass CP_ASYNC_BULK_G2S_INTR<bit has_ch> {
566:   defvar Intr = int_nvvm_cp_async_bulk_global_to_shared_cluster;
567:
568:   def "" : NVPTXInst<(outs),
569:       (ins ADDR:$dst, ADDR:$mbar, ADDR:$src,
570:            B32:$size, B16:$mask, B64:$ch),
571:       !if(has_ch,
572:           CpAsyncBulkStr<0, 1>.G2S # " [$dst], [$src], $size, [$mbar], $ch;",
573:           CpAsyncBulkStr<0, 0>.G2S # " [$dst], [$src], $size, [$mbar];"),
574:       [(Intr addr:$dst, addr:$mbar, addr:$src, i32:$size, i16:$mask, i64:$ch, 0, !if(has_ch, -1, 0))]>,
575:       Requires<[hasPTX<80>, hasSM<90>]>;
576:
577:   def _MC : NVPTXInst<(outs),
578:       (ins ADDR:$dst, ADDR:$mbar, ADDR:$src,
579:            B32:$size, B16:$mask, B64:$ch),
580:       !if(has_ch,
581:           CpAsyncBulkStr<1, 1>.G2S # " [$dst], [$src], $size, [$mbar], $mask, $ch;",
582:           CpAsyncBulkStr<1, 0>.G2S # " [$dst], [$src], $size, [$mbar], $mask;"),
583:       [(Intr addr:$dst, addr:$mbar, addr:$src, i32:$size, i16:$mask, i64:$ch, -1, !if(has_ch, -1, 0))]>,
584:       Requires<[hasPTX<80>, hasSM<90>]>;
585: }
586: defm CP_ASYNC_BULK_G2S    : CP_ASYNC_BULK_G2S_INTR<has_ch = 0>;
587: defm CP_ASYNC_BULK_G2S_CH : CP_ASYNC_BULK_G2S_INTR<has_ch = 1>;
588:
589: multiclass CP_ASYNC_BULK_G2S_CTA_INTR<bit has_ch> {
590:   defvar Intr = int_nvvm_cp_async_bulk_global_to_shared_cta;
591:
592:   def "" : NVPTXInst<(outs),
593:       (ins ADDR:$dst, ADDR:$mbar, ADDR:$src,
594:            B32:$size, B64:$ch),
595:       !if(has_ch,
596:           CpAsyncBulkStr<0, 1>.G2S_CTA # " [$dst], [$src], $size, [$mbar], $ch;",
597:           CpAsyncBulkStr<0, 0>.G2S_CTA # " [$dst], [$src], $size, [$mbar];"),
598:       [(Intr addr:$dst, addr:$mbar, addr:$src, i32:$size, i64:$ch, !if(has_ch, -1, 0))]>,
599:       Requires<[hasPTX<86>, hasSM<90>]>;
600: }
601: defm CP_ASYNC_BULK_G2S_CTA    : CP_ASYNC_BULK_G2S_CTA_INTR<has_ch = 0>;
602: defm CP_ASYNC_BULK_G2S_CTA_CH : CP_ASYNC_BULK_G2S_CTA_INTR<has_ch = 1>;
603:
604: def CP_ASYNC_BULK_CTA_TO_CLUSTER : NVPTXInst<(outs),
605:   (ins ADDR:$dst, ADDR:$mbar, ADDR:$src, B32:$size),
606:   CpAsyncBulkStr<0, 0>.C2C # " [$dst], [$src], $size, [$mbar];",
607:   [(int_nvvm_cp_async_bulk_shared_cta_to_cluster addr:$dst, addr:$mbar, addr:$src, i32:$size)]>,
608:   Requires<[hasPTX<80>, hasSM<90>]>;
609:
610: multiclass CP_ASYNC_BULK_PREFETCH_INTR<bit has_ch> {
611:   def "" : NVPTXInst<(outs), (ins ADDR:$src, B32:$size, B64:$ch),
612:       !if(has_ch,
613:           "cp.async.bulk.prefetch.L2.global.L2::cache_hint" # " [$src], $size, $ch;",
614:           "cp.async.bulk.prefetch.L2.global" # " [$src], $size;"),
615:       [(int_nvvm_cp_async_bulk_prefetch_L2 addr:$src, i32:$size, i64:$ch, !if(has_ch, -1, 0))]>,
616:       Requires<[hasPTX<80>, hasSM<90>]>;
617: }
618: defm CP_ASYNC_BULK_PREFETCH    : CP_ASYNC_BULK_PREFETCH_INTR<has_ch = 0>;
619: defm CP_ASYNC_BULK_PREFETCH_CH : CP_ASYNC_BULK_PREFETCH_INTR<has_ch = 1>;
620:
621: //-------------------------------------
622: // TMA Async Bulk Tensor Copy Functions
623: //-------------------------------------
624:
625: class TMA_DIMS_UTIL<int dim, string mode = ""> {
626:   // For example, when 'dim' is 3, this generates:
627:   // an ins_dag:    B32:$d0, B32:$d1, B32:$d2
628:   // with base_str: $d0, $d1, $d2
629:   dag ins_dag = !dag(ins, !listsplat(B32, dim), !foreach(i, !range(dim), "d" # i));
630:   string base_str = !interleave(!foreach(i, !range(dim), "$d" # i), ", ");
631:
632:   // Tile::Gather4/scatter4 actually operate on a 2D tensor,
633:   // though they take 5 co-ordinates.
634:   //
635:   // The scatter-gather happens over 4 rows with a fixed
636:   // column-index. The first co-ordinate represents the
637:   // col-index followed by four row-indices.
638:   int num_dims = !cond(
639:                    !eq(mode, "tile_scatter4") : 2,
640:                    !eq(mode, "tile_gather4")  : 2,
641:                    true : dim); // for all other modes
642: }
643:
644: class TMA_IM2COL_UTIL<int dim, string mode> {
645:   // For im2col_w/w_128 modes, number of offsets is always 2.
646:   // For im2col mode, offsets is (dim - 2).
647:   // For non-im2col modes (i.e. tile) there are no offsets.
648:   int offsets = !cond(
649:                   !eq(mode, "im2col") : !sub(dim, 2),
650:                   !eq(mode, "im2col_w") : 2,
651:                   !eq(mode, "im2col_w_128") : 2,
652:                   true : 0); // for all other modes
653:
654:   dag ins_dag = !if(!gt(offsets, 0),
655:     !dag(ins, !listsplat(B16, offsets), !foreach(i, !range(offsets), "im2col" # i)),
656:     (ins));
657:   string base_str = !interleave(!foreach(i, !range(offsets), "$im2col" # i), ", ");
658: }
659:
660: def CTAGroupFlags : Operand<i32> {
661:   let PrintMethod = "printCTAGroup";
662: }
663:
664: def tma_cta_group_imm0 : TImmLeaf<i32, [{return Imm == 0;}]>;
665: def tma_cta_group_imm_any : TImmLeaf<i32, [{return Imm >= 0;}]>;
666:
667: multiclass TMA_TENSOR_G2S_INTR<int dim, string mode, list<Predicate> pred,
668:                                TImmLeaf cta_group_type = tma_cta_group_imm_any> {
669:   defvar dims_dag = TMA_DIMS_UTIL<dim>.ins_dag;
670:   defvar dims_str = TMA_DIMS_UTIL<dim>.base_str;
671:   defvar asm_str_base = "$cg [$dst], [$tmap, {{" # dims_str # "}}], [$mbar]";
672:
673:   defvar im2col_dag = TMA_IM2COL_UTIL<dim, mode>.ins_dag;
674:   defvar im2col_str = TMA_IM2COL_UTIL<dim, mode>.base_str;
675:   defvar asm_str = !if(!empty(im2col_str),
676:                        asm_str_base,
677:                        asm_str_base # ", {{" # im2col_str # "}}");
678:
679:   defvar dim_val = TMA_DIMS_UTIL<dim, mode>.num_dims;
680:   defvar inst_name = "cp.async.bulk.tensor"
681:                      # "." # dim_val # "d"
682:                      # "." # "shared::cluster.global"
683:                      # "." # !subst("_", "::", mode)
684:                      # "." # "mbarrier::complete_tx::bytes";
685:   defvar intr = !cast<Intrinsic>(
686:                   "int_nvvm_cp_async_bulk_tensor_g2s_" # mode # "_" # dim_val # "d");
687:
688:   defvar ins_dag = !con(
689:                      (ins ADDR:$dst, ADDR:$mbar, B64:$tmap),
690:                      dims_dag, im2col_dag,
691:                      (ins B16:$mc, B64:$ch, CTAGroupFlags:$cg));
692:
693:   defvar intr_dag_base = !con(
694:                          (intr addr:$dst, addr:$mbar, B64:$tmap),
695:                          !setdagop(dims_dag, intr),
696:                          !setdagop(im2col_dag, intr),
697:                          (intr B16:$mc, B64:$ch));
698:   defvar intr_dag_no_hints   = !con(intr_dag_base, (intr 0,  0,  cta_group_type:$cg));
699:   defvar intr_dag_with_mc    = !con(intr_dag_base, (intr -1, 0,  cta_group_type:$cg));
700:   defvar intr_dag_with_ch    = !con(intr_dag_base, (intr 0, -1,  cta_group_type:$cg));
```
- EN: This range uses TableGen DSL to describe records such as CP_ASYNC_BULK_S2G, CP_ASYNC_BULK_S2G_CH, CP_ASYNC_BULK_G2S_INTR, ""; LLVM later expands these declarations into generated backend tables and helper code.
- CN: 这一段使用 TableGen DSL 描述 CP_ASYNC_BULK_S2G、CP_ASYNC_BULK_S2G_CH、CP_ASYNC_BULK_G2S_INTR、"" 等记录；LLVM 后续会把这些声明展开为生成的后端表和辅助代码。

### Lines 701-840
```tablegen
701:   defvar intr_dag_with_mc_ch = !con(intr_dag_base, (intr -1, -1, cta_group_type:$cg));
702:
703:   def "" : NVPTXInst<(outs), ins_dag,
704:              inst_name # asm_str # ";",
705:              [intr_dag_no_hints]>,
706:              Requires<pred>;
707:   def _MC : NVPTXInst<(outs), ins_dag,
708:               inst_name # ".multicast::cluster" # asm_str # ", $mc;",
709:               [intr_dag_with_mc]>,
710:               Requires<pred>;
711:   def _CH : NVPTXInst<(outs), ins_dag,
712:               inst_name # ".L2::cache_hint" # asm_str # ", $ch;",
713:               [intr_dag_with_ch]>,
714:               Requires<pred>;
715:   def _MC_CH : NVPTXInst<(outs), ins_dag,
716:                  inst_name # ".multicast::cluster.L2::cache_hint" # asm_str # ", $mc, $ch;",
717:                  [intr_dag_with_mc_ch]>,
718:                  Requires<pred>;
719: }
720:
721: foreach dim = 1...5 in {
722:   defm TMA_G2S_TILE_CG0_ # dim # "D"
723:       : TMA_TENSOR_G2S_INTR<dim, "tile", [hasPTX<80>, hasSM<90>],
724:                             tma_cta_group_imm0>;
725:   defm TMA_G2S_TILE_ # dim # "D"
726:       : TMA_TENSOR_G2S_INTR<dim, "tile",
727:                             [callSubtarget<"hasTMABlackwellSupport">]>;
728: }
729: foreach dim = 3...5 in {
730:   defm TMA_G2S_IM2COL_CG0_ # dim # "D"
731:       : TMA_TENSOR_G2S_INTR<dim, "im2col", [hasPTX<80>, hasSM<90>],
732:                             tma_cta_group_imm0>;
733:   defm TMA_G2S_IM2COL_ # dim # "D"
734:       : TMA_TENSOR_G2S_INTR<dim, "im2col",
735:                             [callSubtarget<"hasTMABlackwellSupport">]>;
736:   foreach mode = ["im2col_w", "im2col_w_128"] in {
737:     defm TMA_G2S_ # !toupper(mode) # "_" # dim # "D"
738:         : TMA_TENSOR_G2S_INTR<dim, mode,
739:                               [callSubtarget<"hasTMABlackwellSupport">]>;
740:   }
741: }
742: defm TMA_G2S_TILE_GATHER4_2D : TMA_TENSOR_G2S_INTR<5, "tile_gather4",
743:                                [callSubtarget<"hasTMABlackwellSupport">]>;
744:
745: multiclass TMA_TENSOR_G2S_CTA_INTR<int dim, string mode, list<Predicate> pred = []> {
746:   defvar dims_dag = TMA_DIMS_UTIL<dim>.ins_dag;
747:   defvar dims_str = TMA_DIMS_UTIL<dim>.base_str;
748:   defvar asm_str_base = " [$dst], [$tmap, {{" # dims_str # "}}], [$mbar]";
749:
750:   defvar im2col_dag = TMA_IM2COL_UTIL<dim, mode>.ins_dag;
751:   defvar im2col_str = TMA_IM2COL_UTIL<dim, mode>.base_str;
752:   defvar asm_str = !if(!empty(im2col_str),
753:                        asm_str_base,
754:                        asm_str_base # ", {{" # im2col_str # "}}");
755:
756:   defvar ins_dag = !con(
757:                      (ins ADDR:$dst, ADDR:$mbar, B64:$tmap),
758:                      dims_dag, im2col_dag,
759:                      (ins B64:$ch));
760:
761:   defvar dim_val = TMA_DIMS_UTIL<dim, mode>.num_dims;
762:   defvar intr = !cast<Intrinsic>(
763:                   "int_nvvm_cp_async_bulk_tensor_g2s_cta_" # mode # "_" # dim_val # "d");
764:   defvar intr_dag = !con(
765:                       (intr addr:$dst, addr:$mbar, B64:$tmap),
766:                       !setdagop(dims_dag, intr),
767:                       !setdagop(im2col_dag, intr),
768:                       (intr B64:$ch, 0));
769:   defvar intr_dag_with_ch = !con(
770:                               (intr addr:$dst, addr:$mbar, B64:$tmap),
771:                               !setdagop(dims_dag, intr),
772:                               !setdagop(im2col_dag, intr),
773:                               (intr B64:$ch, -1));
774:   defvar inst_name = "cp.async.bulk.tensor"
775:                      # "." # dim_val # "d"
776:                      # "." # "shared::cta.global"
777:                      # "." # !subst("_", "::", mode)
778:                      # "." # "mbarrier::complete_tx::bytes";
779:
780:   def "" : NVPTXInst<(outs), ins_dag,
781:              inst_name # asm_str # ";",
782:              [intr_dag]>,
783:              Requires<pred>;
784:   def _CH : NVPTXInst<(outs), ins_dag,
785:               inst_name # ".L2::cache_hint" # asm_str # ", $ch;",
786:               [intr_dag_with_ch]>,
787:               Requires<pred>;
788: }
789: foreach dim = 1...5 in {
790:   defm TMA_G2S_CTA_TILE_ # dim # "D"
791:     : TMA_TENSOR_G2S_CTA_INTR<dim, "tile", [hasPTX<86>, hasSM<90>]>;
792: }
793: foreach dim = 3...5 in {
794:   defm TMA_G2S_CTA_IM2COL_ # dim # "D"
795:     : TMA_TENSOR_G2S_CTA_INTR<dim, "im2col", [hasPTX<86>, hasSM<90>]>;
796:
797:   defm TMA_G2S_CTA_IM2COL_W_ # dim # "D"
798:     : TMA_TENSOR_G2S_CTA_INTR<dim, "im2col_w", [hasPTX<86>, hasSM<100>]>;
799:
800:   defm TMA_G2S_CTA_IM2COL_W_128_ # dim # "D"
801:     : TMA_TENSOR_G2S_CTA_INTR<dim, "im2col_w_128",
802:                               [callSubtarget<"hasTMABlackwellSupport">]>;
803: }
804: defm TMA_G2S_CTA_TILE_GATHER4_2D : TMA_TENSOR_G2S_CTA_INTR<5, "tile_gather4",
805:                                    [hasPTX<86>, hasSM<100>]>;
806:
807: multiclass TMA_TENSOR_S2G_INTR<int dim, string mode,
808:                                list<Predicate> pred = [hasPTX<80>, hasSM<90>]> {
809:   defvar dims_dag = TMA_DIMS_UTIL<dim>.ins_dag;
810:   defvar dims_str = TMA_DIMS_UTIL<dim>.base_str;
811:   defvar asm_str = " [$tmap, {{" # dims_str # "}}], [$src]";
812:
813:   defvar dim_val = TMA_DIMS_UTIL<dim, mode>.num_dims;
814:   defvar intr = !cast<Intrinsic>(
815:                   "int_nvvm_cp_async_bulk_tensor_s2g_" # mode # "_" # dim_val # "d");
816:
817:   defvar intr_dag = !con((intr addr:$src, B64:$tmap),
818:                          !setdagop(dims_dag, intr),
819:                          (intr B64:$ch, 0));
820:   defvar intr_dag_with_ch = !con((intr addr:$src, B64:$tmap),
821:                                  !setdagop(dims_dag, intr),
822:                                  (intr B64:$ch, -1));
823:
824:   // Fix-up the asm_str when it is im2col/scatter4.
825:   defvar mode_asm_str = !cond(
826:                           !eq(mode, "im2col") : "im2col_no_offs",
827:                           !eq(mode, "tile_scatter4") : "tile::scatter4",
828:                           true : mode);
829:   defvar prefix = "cp.async.bulk.tensor"
830:                   # "." # dim_val # "d"
831:                   # ".global.shared::cta"
832:                   # "." # mode_asm_str
833:                   # ".bulk_group";
834:
835:   def "" : NVPTXInst<(outs),
836:              !con((ins ADDR:$src, B64:$tmap), dims_dag, (ins B64:$ch)),
837:              prefix # asm_str # ";",
838:              [intr_dag]>,
839:              Requires<pred>;
840:   def _CH : NVPTXInst<(outs),
```
- EN: This range uses TableGen DSL to describe records such as "", _MC, _CH, _MC_CH; LLVM later expands these declarations into generated backend tables and helper code.
- CN: 这一段使用 TableGen DSL 描述 ""、_MC、_CH、_MC_CH 等记录；LLVM 后续会把这些声明展开为生成的后端表和辅助代码。

### Lines 841-980
```tablegen
841:               !con((ins ADDR:$src, B64:$tmap), dims_dag, (ins B64:$ch)),
842:               prefix # ".L2::cache_hint" # asm_str # ", $ch;",
843:               [intr_dag_with_ch]>,
844:               Requires<pred>;
845: }
846: foreach dim = 1...5 in {
847:   foreach mode = !if(!ge(dim, 3), ["tile", "im2col"], ["tile"]) in {
848:     defvar suffix = !toupper(mode) # "_" # dim # "D";
849:     defm TMA_TENSOR_S2G_ # suffix : TMA_TENSOR_S2G_INTR<dim, mode>;
850:   }
851: }
852: defm TMA_S2G_TILE_SCATTER4_2D : TMA_TENSOR_S2G_INTR<5, "tile_scatter4",
853:                                 [callSubtarget<"hasTMABlackwellSupport">]>;
854:
855: def TMAReductionFlags : Operand<i32> {
856:   let PrintMethod = "printTmaReductionMode";
857: }
858:
859: // TMA Copy from Shared to Global memory with Reduction
860: multiclass CP_ASYNC_BULK_TENSOR_REDUCE_INTR<int dim, bit shared32, string mode> {
861:   defvar dims_dag = TMA_DIMS_UTIL<dim>.ins_dag;
862:   defvar dims_str = TMA_DIMS_UTIL<dim>.base_str;
863:   defvar asm_str = " [$tmap, {{" # dims_str # "}}], [$src]";
864:   defvar rc = !if(shared32, B32, B64);
865:
866:   // For im2col mode, the actual asm_str is "im2col_no_offs"
867:   defvar mode_asm_str = !if(!eq(mode, "im2col"),
868:                             "im2col_no_offs", mode);
869:   defvar prefix = "cp.reduce.async.bulk.tensor" # "." # dim # "d" # ".global.shared::cta";
870:   defvar suffix = "." # mode_asm_str # ".bulk_group";
871:
872:   def "" : NVPTXInst<(outs),
873:             !con((ins rc:$src, B64:$tmap), dims_dag, (ins TMAReductionFlags:$red_op)),
874:             !strconcat(prefix, "${red_op}", suffix, asm_str, ";")>,
875:             Requires<[hasPTX<80>, hasSM<90>]>;
876:   def _CH : NVPTXInst<(outs),
877:                   !con((ins rc:$src, B64:$tmap), dims_dag, (ins B64:$ch, TMAReductionFlags:$red_op)),
878:                   !strconcat(prefix, "${red_op}", suffix, ".L2::cache_hint", asm_str, ", $ch;")>,
879:                   Requires<[hasPTX<80>, hasSM<90>]>;
880: }
881:
882: foreach dim = [1, 2, 3, 4, 5] in {
883:   foreach shared32 = [true, false] in {
884:     foreach mode = !if(!ge(dim, 3), ["tile", "im2col"], ["tile"]) in {
885:       defvar suffix = dim # "D"
886:                       # !if(shared32, "_SHARED32", "")
887:                       # "_" # !toupper(mode);
888:       defm CP_ASYNC_BULK_TENSOR_RED_ # suffix :
889:         CP_ASYNC_BULK_TENSOR_REDUCE_INTR<dim, shared32, mode>;
890:     }
891:   }
892: }
893:
894: // TMA Prefetch from Global memory to L2 cache
895: multiclass TMA_TENSOR_PREFETCH_INTR<int dim, string mode,
896:                                     list<Predicate> pred = [hasPTX<80>, hasSM<90>]> {
897:   defvar dims_dag = TMA_DIMS_UTIL<dim>.ins_dag;
898:   defvar dims_str = TMA_DIMS_UTIL<dim>.base_str;
899:   defvar asm_str_base = " [$tmap, {{" # dims_str # "}}]";
900:
901:   defvar im2col_dag = TMA_IM2COL_UTIL<dim, mode>.ins_dag;
902:   defvar im2col_str = TMA_IM2COL_UTIL<dim, mode>.base_str;
903:   defvar asm_str = !if(!empty(im2col_str),
904:                        asm_str_base,
905:                        asm_str_base # ", {{" # im2col_str # "}}");
906:
907:   defvar dim_val = TMA_DIMS_UTIL<dim, mode>.num_dims;
908:   defvar inst_name = "cp.async.bulk.prefetch.tensor"
909:                      # "." # dim_val # "d"
910:                      # "." # "L2.global"
911:                      # "." # !subst("_", "::", mode);
912:
913:   defvar intr = !cast<Intrinsic>(
914:                   "int_nvvm_cp_async_bulk_tensor_prefetch_" # mode # "_" # dim_val # "d");
915:
916:   defvar ins_dag  = !con((ins  B64:$tmap),
917:                          dims_dag,
918:                          im2col_dag,
919:                          (ins B64:$ch));
920:   defvar intr_dag = !con((intr B64:$tmap),
921:                          !setdagop(dims_dag, intr),
922:                          !setdagop(im2col_dag, intr),
923:                          (intr B64:$ch, 0));
924:   defvar intr_dag_with_ch = !con((intr B64:$tmap),
925:                                  !setdagop(dims_dag, intr),
926:                                  !setdagop(im2col_dag, intr),
927:                                  (intr B64:$ch, -1));
928:
929:   def "" : NVPTXInst<(outs), ins_dag,
930:              inst_name # asm_str # ";",
931:              [intr_dag]>,
932:              Requires<pred>;
933:   def _CH : NVPTXInst<(outs), ins_dag,
934:               inst_name # ".L2::cache_hint" # asm_str # ", $ch;",
935:               [intr_dag_with_ch]>,
936:               Requires<pred>;
937: }
938: foreach dim = 1...5 in {
939:   foreach mode = !if(!ge(dim, 3), ["tile", "im2col"], ["tile"]) in {
940:     defvar suffix = !toupper(mode) # "_" # dim # "D";
941:     defm TMA_TENSOR_PF_ # suffix : TMA_TENSOR_PREFETCH_INTR<dim, mode>;
942:   }
943: }
944: foreach dim = 3...5 in {
945:   foreach mode = ["im2col_w", "im2col_w_128"] in {
946:     defvar suffix = !toupper(mode) # "_" # dim # "D";
947:     defm TMA_TENSOR_PF_ # suffix : TMA_TENSOR_PREFETCH_INTR<dim, mode,
948:                                    [callSubtarget<"hasTMABlackwellSupport">]>;
949:   }
950: }
951: defm TMA_TENSOR_PF_TILE_GATHER4_2D : TMA_TENSOR_PREFETCH_INTR<5, "tile_gather4",
952:                                      [callSubtarget<"hasTMABlackwellSupport">]>;
953:
954: //Prefetchu and Prefetch
955:
956: defvar frag_pat = (int_nvvm_prefetch_tensormap node:$addr);
957:
958: multiclass PREFETCH_TENSORMAP_PATFRAG<string suffix, code predicate> {
959:   def !tolower(suffix) : PatFrag<!setdagop(frag_pat, ops), frag_pat, predicate>;
960: }
961:
962: defm prefetch_tensormap_ : PREFETCH_TENSORMAP_PATFRAG<"CONST", AS_match.const>;
963: defm prefetch_tensormap_ : PREFETCH_TENSORMAP_PATFRAG<"GENERIC", AS_match.generic>;
964: defm prefetch_tensormap_ : PREFETCH_TENSORMAP_PATFRAG<"PARAM", AS_match.param>;
965:
966: multiclass PREFETCH_TENSORMAP_INST<string addrspace_name, PatFrag pattern_frag> {
967:   def "" : BasicNVPTXInst<(outs), (ins ADDR:$addr),
968:            "prefetch" # addrspace_name # ".tensormap",
969:            [(pattern_frag addr:$addr)]>,
970:            Requires<[hasPTX<80>, hasSM<90>]>;
971: }
972:
973: defm PREFETCH_CONST_TENSORMAP   : PREFETCH_TENSORMAP_INST<".const", prefetch_tensormap_const>;
974: defm PREFETCH_GENERIC_TENSORMAP : PREFETCH_TENSORMAP_INST<"", prefetch_tensormap_generic>;
975: defm PREFETCH_PARAM_TENSORMAP   : PREFETCH_TENSORMAP_INST<".param", prefetch_tensormap_param>;
976:   
977: class PREFETCH_INTRS<string InstName, Intrinsic Intr> :
978:           BasicNVPTXInst<(outs), (ins ADDR:$addr),
979:           InstName,
980:           [(Intr addr:$addr)]>,
```
- EN: This range uses TableGen DSL to describe records such as TMA_TENSOR_S2G_, TMA_S2G_TILE_SCATTER4_2D, TMAReductionFlags, CP_ASYNC_BULK_TENSOR_REDUCE_INTR; LLVM later expands these declarations into generated backend tables and helper code.
- CN: 这一段使用 TableGen DSL 描述 TMA_TENSOR_S2G_、TMA_S2G_TILE_SCATTER4_2D、TMAReductionFlags、CP_ASYNC_BULK_TENSOR_REDUCE_INTR 等记录；LLVM 后续会把这些声明展开为生成的后端表和辅助代码。

### Lines 981-1120
```tablegen
 981:           Requires<[hasPTX<80>, hasSM<90>]>;
 982:
 983: def PREFETCHU_L1 : PREFETCH_INTRS<"prefetchu.L1", int_nvvm_prefetchu_L1>;   
 984: def PREFETCH_L1 : PREFETCH_INTRS<"prefetch.L1", int_nvvm_prefetch_L1>;
 985: def PREFETCH_L2 : PREFETCH_INTRS<"prefetch.L2", int_nvvm_prefetch_L2>;
 986: def PREFETCH_GLOBAL_L1 : PREFETCH_INTRS<"prefetch.global.L1", int_nvvm_prefetch_global_L1>;
 987: def PREFETCH_LOCAL_L1 : PREFETCH_INTRS<"prefetch.local.L1", int_nvvm_prefetch_local_L1>;
 988: def PREFETCH_GLOBAL_L2 : PREFETCH_INTRS<"prefetch.global.L2", int_nvvm_prefetch_global_L2>;
 989: def PREFETCH_LOCAL_L2 : PREFETCH_INTRS<"prefetch.local.L2", int_nvvm_prefetch_local_L2>;
 990: def PREFETCH_GLOBAL_L2_EVICT_NORMAL : PREFETCH_INTRS<"prefetch.global.L2::evict_normal", 
 991:                                       int_nvvm_prefetch_global_L2_evict_normal>;
 992: def PREFETCH_GLOBAL_L2_EVICT_LAST : PREFETCH_INTRS<"prefetch.global.L2::evict_last", 
 993:                                     int_nvvm_prefetch_global_L2_evict_last>;
 994:
 995: //Applypriority intrinsics
 996: class APPLYPRIORITY_L2_INTRS<string addrspace> :
 997:           BasicNVPTXInst<(outs), (ins ADDR:$addr, B64:$size),
 998:           StrJoin<".", ["applypriority", addrspace , "L2::evict_normal"]>.ret,
 999:           [(!cast<Intrinsic>(StrJoin<"_", ["int_nvvm_applypriority", addrspace , "L2_evict_normal"]>.ret)
1000:           addr:$addr, i64:$size)]>,
1001:           Requires<[hasPTX<74>, hasSM<80>]>;
1002:
1003: def APPLYPRIORITY_L2_EVICT_NORMAL        : APPLYPRIORITY_L2_INTRS<"">;
1004: def APPLYPRIORITY_GLOBAL_L2_EVICT_NORMAL : APPLYPRIORITY_L2_INTRS<"global">;
1005:
1006: //Discard Intrinsics
1007:
1008: def discard_size_imm : TImmLeaf<i64, [{ return Imm == 128; }]>;
1009:
1010: class DISCARD_L2_INTRS<string addrspace> :
1011:           BasicNVPTXInst<(outs), (ins ADDR:$addr, i64imm:$size),
1012:           StrJoin<".", ["discard", addrspace , "L2"]>.ret,
1013:           [(!cast<Intrinsic>(StrJoin<"_", ["int_nvvm_discard", addrspace , "L2"]>.ret)
1014:           addr:$addr, discard_size_imm:$size)]>,
1015:           Requires<[hasPTX<74>, hasSM<80>]>;
1016:
1017: def DISCARD_L2        : DISCARD_L2_INTRS<"">;
1018: def DISCARD_GLOBAL_L2 : DISCARD_L2_INTRS<"global">;
1019:
1020: //-----------------------------------
1021: // MBarrier Functions
1022: //-----------------------------------
1023:
1024: let Predicates = [hasPTX<70>, hasSM<80>] in {
1025:   class MBARRIER_INIT<string AddrSpace, Intrinsic Intrin> :
1026:             BasicNVPTXInst<(outs), (ins ADDR:$addr, B32:$count),
1027:             "mbarrier.init" # AddrSpace # ".b64",
1028:             [(Intrin addr:$addr, i32:$count)]>;
1029:
1030:   def MBARRIER_INIT : MBARRIER_INIT<"", int_nvvm_mbarrier_init>;
1031:   def MBARRIER_INIT_SHARED : MBARRIER_INIT<".shared",
1032:                                             int_nvvm_mbarrier_init_shared>;
1033:
1034:   class MBARRIER_INVAL<string AddrSpace, Intrinsic Intrin> :
1035:             BasicNVPTXInst<(outs), (ins ADDR:$addr),
1036:             "mbarrier.inval" # AddrSpace # ".b64",
1037:             [(Intrin addr:$addr)]>;
1038:
1039:   def MBARRIER_INVAL : MBARRIER_INVAL<"", int_nvvm_mbarrier_inval>;
1040:   def MBARRIER_INVAL_SHARED : MBARRIER_INVAL<".shared",
1041:                                               int_nvvm_mbarrier_inval_shared>;
1042:
1043:   class MBARRIER_ARRIVE<string AddrSpace, Intrinsic Intrin> :
1044:             BasicNVPTXInst<(outs B64:$state), (ins ADDR:$addr),
1045:             "mbarrier.arrive" # AddrSpace # ".b64",
1046:             [(set i64:$state, (Intrin addr:$addr))]>;
1047:
1048:   def MBARRIER_ARRIVE : MBARRIER_ARRIVE<"", int_nvvm_mbarrier_arrive>;
1049:   def MBARRIER_ARRIVE_SHARED :
1050:     MBARRIER_ARRIVE<".shared", int_nvvm_mbarrier_arrive_shared>;
1051:
1052:   class MBARRIER_ARRIVE_NOCOMPLETE<string AddrSpace, Intrinsic Intrin> :
1053:             BasicNVPTXInst<(outs B64:$state),
1054:             (ins ADDR:$addr, B32:$count),
1055:             "mbarrier.arrive.noComplete" # AddrSpace # ".b64",
1056:       [(set i64:$state, (Intrin addr:$addr, i32:$count))]>;
1057:
1058:   def MBARRIER_ARRIVE_NOCOMPLETE :
1059:     MBARRIER_ARRIVE_NOCOMPLETE<"", int_nvvm_mbarrier_arrive_noComplete>;
1060:   def MBARRIER_ARRIVE_NOCOMPLETE_SHARED :
1061:     MBARRIER_ARRIVE_NOCOMPLETE<".shared", int_nvvm_mbarrier_arrive_noComplete_shared>;
1062:
1063:   class MBARRIER_ARRIVE_DROP<string AddrSpace, Intrinsic Intrin> :
1064:             BasicNVPTXInst<(outs B64:$state), (ins ADDR:$addr),
1065:             "mbarrier.arrive_drop" # AddrSpace # ".b64",
1066:             [(set i64:$state, (Intrin addr:$addr))]>;
1067:
1068:   def MBARRIER_ARRIVE_DROP :
1069:     MBARRIER_ARRIVE_DROP<"", int_nvvm_mbarrier_arrive_drop>;
1070:   def MBARRIER_ARRIVE_DROP_SHARED :
1071:     MBARRIER_ARRIVE_DROP<".shared", int_nvvm_mbarrier_arrive_drop_shared>;
1072:
1073:   class MBARRIER_ARRIVE_DROP_NOCOMPLETE<string AddrSpace, Intrinsic Intrin> :
1074:             BasicNVPTXInst<(outs B64:$state),
1075:             (ins ADDR:$addr, B32:$count),
1076:             "mbarrier.arrive_drop.noComplete" # AddrSpace # ".b64",
1077:             [(set i64:$state, (Intrin addr:$addr, i32:$count))]>;
1078:
1079:   def MBARRIER_ARRIVE_DROP_NOCOMPLETE :
1080:     MBARRIER_ARRIVE_DROP_NOCOMPLETE<"", int_nvvm_mbarrier_arrive_drop_noComplete>;
1081:   def MBARRIER_ARRIVE_DROP_NOCOMPLETE_SHARED :
1082:     MBARRIER_ARRIVE_DROP_NOCOMPLETE<".shared",
1083:                         int_nvvm_mbarrier_arrive_drop_noComplete_shared>;
1084:
1085:   class MBARRIER_TEST_WAIT<string AddrSpace, Intrinsic Intrin> :
1086:             BasicNVPTXInst<(outs B1:$res), (ins ADDR:$addr, B64:$state),
1087:             "mbarrier.test_wait" # AddrSpace # ".b64",
1088:             [(set i1:$res, (Intrin addr:$addr, i64:$state))]>;
1089:
1090:   def MBARRIER_TEST_WAIT :
1091:     MBARRIER_TEST_WAIT<"", int_nvvm_mbarrier_test_wait>;
1092:   def MBARRIER_TEST_WAIT_SHARED :
1093:     MBARRIER_TEST_WAIT<".shared", int_nvvm_mbarrier_test_wait_shared>;
1094:
1095:   def MBARRIER_PENDING_COUNT :
1096:             BasicNVPTXInst<(outs B32:$res), (ins B64:$state),
1097:             "mbarrier.pending_count.b64",
1098:             [(set i32:$res, (int_nvvm_mbarrier_pending_count i64:$state))]>;
1099: }
1100:
1101: class MBAR_UTIL<string op, string scope,
1102:                 string space = "", string sem = "",
1103:                 bit tl = 0, bit parity = 0> {
1104:   // The mbarrier instructions in PTX ISA are of the general form:
1105:   // mbarrier.op.semantics.scope.space.b64 arg1, arg2 ...
1106:   // where:
1107:   // op -> arrive, expect_tx, complete_tx, arrive.expect_tx etc.
1108:   // semantics -> acquire, release, relaxed (default depends on the op)
1109:   // scope -> cta or cluster (default is cta-scope)
1110:   // space -> shared::cta or shared::cluster (default is shared::cta)
1111:   //
1112:   // The 'semantics' and 'scope' go together. If one is specified,
1113:   // then the other _must_ be specified. For example:
1114:   // (A) mbarrier.arrive             <args> (valid, release and cta are default)
1115:   // (B) mbarrier.arrive.release.cta <args> (valid, sem/scope mentioned explicitly)
1116:   // (C) mbarrier.arrive.release     <args> (invalid, needs scope)
1117:   // (D) mbarrier.arrive.cta         <args> (invalid, needs order)
1118:   //
1119:   // Wherever possible, we prefer form (A) to (B) since it is available
1120:   // from early PTX versions. In most cases, explicitly specifying the
```
- EN: This range uses TableGen DSL to describe records such as PREFETCHU_L1, PREFETCH_L1, PREFETCH_L2, PREFETCH_GLOBAL_L1; LLVM later expands these declarations into generated backend tables and helper code.
- CN: 这一段使用 TableGen DSL 描述 PREFETCHU_L1、PREFETCH_L1、PREFETCH_L2、PREFETCH_GLOBAL_L1 等记录；LLVM 后续会把这些声明展开为生成的后端表和辅助代码。

### Lines 1121-1260
```tablegen
1121:   // scope requires a later version of PTX.
1122:   string _scope_asm = !cond(
1123:                       !eq(scope, "scope_cluster") : "cluster",
1124:                       !eq(scope, "scope_cta") : !if(!empty(sem), "", "cta"),
1125:                       true : scope);
1126:   string _space_asm = !cond(
1127:                       !eq(space, "space_cta") : "shared",
1128:                       !eq(space, "space_cluster") : "shared::cluster",
1129:                       true : space);
1130:
1131:   string _parity = !if(parity, "parity", "");
1132:   string asm_str = StrJoin<".", ["mbarrier", op, _parity,
1133:                                   sem, _scope_asm, _space_asm, "b64"]>.ret;
1134:
1135:   string _intr_suffix = StrJoin<"_", [!subst(".", "_", op), _parity,
1136:                                       !if(tl, "tl", ""),
1137:                                       sem, scope, space]>.ret;
1138:   string intr_name = "int_nvvm_mbarrier_" # _intr_suffix;
1139:
1140:   // Predicate checks:
1141:   // These are used only for the "test_wait/try_wait" variants as they
1142:   // have evolved since sm80 and are complex. The predicates for the
1143:   // remaining instructions are straightforward and have already been
1144:   // applied directly.
1145:   Predicate _sm_pred = !cond(!or(
1146:                        !eq(op, "try_wait"),
1147:                        !eq(scope, "scope_cluster"),
1148:                        !eq(sem, "relaxed")) : hasSM<90>,
1149:                        true : hasSM<80>);
1150:   Predicate _ptx_pred = !cond(
1151:                         !eq(sem, "relaxed") : hasPTX<86>,
1152:                         !ne(_scope_asm, "") : hasPTX<80>,
1153:                         !eq(op, "try_wait") : hasPTX<78>,
1154:                         parity : hasPTX<71>,
1155:                         true   : hasPTX<70>);
1156:   list<Predicate> preds = [_ptx_pred, _sm_pred];
1157: }
1158:
1159: foreach op = ["expect_tx", "complete_tx"] in {
1160:   foreach scope = ["scope_cta", "scope_cluster"] in {
1161:     foreach space = ["space_cta", "space_cluster"] in {
1162:       defvar intr = !cast<Intrinsic>(MBAR_UTIL<op, scope, space>.intr_name);
1163:       defvar suffix = StrJoin<"_", [op, scope, space]>.ret;
1164:       def mbar_ # suffix : BasicNVPTXInst<(outs), (ins ADDR:$addr, B32:$tx_count),
1165:                            MBAR_UTIL<op, scope, space, "relaxed">.asm_str,
1166:                            [(intr addr:$addr, i32:$tx_count)]>,
1167:                            Requires<[hasPTX<80>, hasSM<90>]>;
1168:     } // space
1169:   } // scope
1170: } // op
1171:
1172: multiclass MBAR_ARR_INTR<string op, string scope, string sem,
1173:                          list<Predicate> pred = []> {
1174:   // When either of sem or scope is non-default, both have to
1175:   // be explicitly specified. So, explicitly state that
1176:   // sem is `release` when scope is `cluster`.
1177:   defvar asm_sem = !if(!and(!empty(sem), !eq(scope, "scope_cluster")),
1178:                     "release", sem);
1179:
1180:   defvar asm_cta  = MBAR_UTIL<op, scope, "space_cta", asm_sem>.asm_str;
1181:   defvar intr_cta = !cast<Intrinsic>(MBAR_UTIL<op, scope,
1182:                                      "space_cta", sem>.intr_name);
1183:
1184:   defvar asm_cluster  = MBAR_UTIL<op, scope, "space_cluster", asm_sem>.asm_str;
1185:   defvar intr_cluster = !cast<Intrinsic>(MBAR_UTIL<op, scope,
1186:                                          "space_cluster", sem>.intr_name);
1187:
1188:   def _CTA : NVPTXInst<(outs B64:$state),
1189:              (ins ADDR:$addr, B32:$tx_count),
1190:              asm_cta # " $state, [$addr], $tx_count;",
1191:              [(set i64:$state, (intr_cta addr:$addr, i32:$tx_count))]>,
1192:              Requires<pred>;
1193:   def _CLUSTER : NVPTXInst<(outs),
1194:                  (ins ADDR:$addr, B32:$tx_count),
1195:                  asm_cluster # " _, [$addr], $tx_count;",
1196:                  [(intr_cluster addr:$addr, i32:$tx_count)]>,
1197:                  Requires<pred>;
1198: }
1199: foreach op = ["arrive", "arrive.expect_tx",
1200:               "arrive_drop", "arrive_drop.expect_tx"] in {
1201:   foreach scope = ["scope_cta", "scope_cluster"] in {
1202:     defvar suffix = !subst(".", "_", op) # scope;
1203:     defm mbar_ # suffix # _release : MBAR_ARR_INTR<op, scope, "", [hasPTX<80>, hasSM<90>]>;
1204:     defm mbar_ # suffix # _relaxed : MBAR_ARR_INTR<op, scope, "relaxed", [hasPTX<86>, hasSM<90>]>;
1205:   } // scope
1206: } // op
1207:
1208: multiclass MBAR_WAIT_INTR<string op, string scope, string sem, bit time_limit> {
1209:   // When either of sem or scope is non-default, both have to
1210:   // be explicitly specified. So, explicitly state that the
1211:   // semantics is `acquire` when the scope is `cluster`.
1212:   defvar asm_sem = !if(!and(!empty(sem), !eq(scope, "scope_cluster")),
1213:                     "acquire", sem);
1214:
1215:   defvar asm_parity  = MBAR_UTIL<op, scope, "space_cta", asm_sem,
1216:                                  time_limit, 1>.asm_str;
1217:   defvar pred_parity = MBAR_UTIL<op, scope, "space_cta", asm_sem,
1218:                                  time_limit, 1>.preds;
1219:   defvar intr_parity = !cast<Intrinsic>(MBAR_UTIL<op, scope, "space_cta",
1220:                                         sem, time_limit, 1>.intr_name);
1221:
1222:   defvar asm_state  = MBAR_UTIL<op, scope, "space_cta", asm_sem,
1223:                                 time_limit>.asm_str;
1224:   defvar pred_state = MBAR_UTIL<op, scope, "space_cta", asm_sem,
1225:                                 time_limit>.preds;
1226:   defvar intr_state = !cast<Intrinsic>(MBAR_UTIL<op, scope, "space_cta",
1227:                                        sem, time_limit>.intr_name);
1228:
1229:   defvar ins_tl_dag = !if(time_limit, (ins B32:$tl), (ins));
1230:   defvar tl_suffix = !if(time_limit, ", $tl;", ";");
1231:   defvar intr_state_dag = !con((intr_state addr:$addr, i64:$state),
1232:                                !if(time_limit, (intr_state i32:$tl), (intr_state)));
1233:   defvar intr_parity_dag = !con((intr_parity addr:$addr, i32:$phase),
1234:                                !if(time_limit, (intr_parity i32:$tl), (intr_parity)));
1235:
1236:   def _STATE : NVPTXInst<(outs B1:$res), !con((ins ADDR:$addr, B64:$state), ins_tl_dag),
1237:                asm_state # " $res, [$addr], $state" # tl_suffix,
1238:                [(set i1:$res, intr_state_dag)]>,
1239:                Requires<pred_state>;
1240:   def _PARITY : NVPTXInst<(outs B1:$res), !con((ins ADDR:$addr, B32:$phase), ins_tl_dag),
1241:                 asm_parity # " $res, [$addr], $phase" # tl_suffix,
1242:                 [(set i1:$res, intr_parity_dag)]>,
1243:                 Requires<pred_parity>;
1244: }
1245: foreach op = ["test_wait", "try_wait"] in {
1246:   foreach scope = ["scope_cta", "scope_cluster"] in {
1247:     foreach time_limit = !if(!eq(op, "try_wait"), [true, false], [false]) in {
1248:       defvar suffix = StrJoin<"_", [op, scope, !if(time_limit, "tl", "")]>.ret;
1249:       defm mbar_ # suffix # "_acquire" : MBAR_WAIT_INTR<op, scope, "", time_limit>;
1250:       defm mbar_ # suffix # "_relaxed" : MBAR_WAIT_INTR<op, scope, "relaxed", time_limit>;
1251:     } // time_limit
1252:   } // scope
1253: } // op
1254:
1255: //-----------------------------------
1256: // Math Functions
1257: //-----------------------------------
1258:
1259: // Map min(1.0, max(0.0, x)) to sat(x)
1260: // Note that max(0.0, min(x, 1.0)) cannot be mapped to sat(x) because when x is
```
- EN: This range uses TableGen DSL to describe records such as mbar_, MBAR_ARR_INTR, _CTA, _CLUSTER; LLVM later expands these declarations into generated backend tables and helper code.
- CN: 这一段使用 TableGen DSL 描述 mbar_、MBAR_ARR_INTR、_CTA、_CLUSTER 等记录；LLVM 后续会把这些声明展开为生成的后端表和辅助代码。

### Lines 1261-1400
```tablegen
1261: // NaN
1262: // max(0.0, min(x, 1.0)) is 1.0 while sat(x) is 0.
1263: // Same story for fmax, fmin.
1264:
1265: def : Pat<(int_nvvm_fmin_f fpimm_1,
1266:             (int_nvvm_fmax_f fpimm_0, f32:$a)),
1267:           (CVT_f32_f32 $a, CvtSAT)>;
1268: def : Pat<(int_nvvm_fmin_f fpimm_1,
1269:             (int_nvvm_fmax_f f32:$a, fpimm_0)),
1270:           (CVT_f32_f32 $a, CvtSAT)>;
1271: def : Pat<(int_nvvm_fmin_f
1272:             (int_nvvm_fmax_f fpimm_0, f32:$a), fpimm_1),
1273:           (CVT_f32_f32 $a, CvtSAT)>;
1274: def : Pat<(int_nvvm_fmin_f
1275:             (int_nvvm_fmax_f f32:$a, fpimm_0), fpimm_1),
1276:           (CVT_f32_f32 $a, CvtSAT)>;
1277:
1278: def : Pat<(int_nvvm_fmin_d fpimm_1,
1279:             (int_nvvm_fmax_d fpimm_0, f64:$a)),
1280:           (CVT_f64_f64 $a, CvtSAT)>;
1281: def : Pat<(int_nvvm_fmin_d fpimm_1,
1282:             (int_nvvm_fmax_d f64:$a, fpimm_0)),
1283:           (CVT_f64_f64 $a, CvtSAT)>;
1284: def : Pat<(int_nvvm_fmin_d
1285:             (int_nvvm_fmax_d fpimm_0, f64:$a), fpimm_1),
1286:           (CVT_f64_f64 $a, CvtSAT)>;
1287: def : Pat<(int_nvvm_fmin_d
1288:             (int_nvvm_fmax_d f64:$a, fpimm_0), fpimm_1),
1289:           (CVT_f64_f64 $a, CvtSAT)>;
1290:
1291:
1292: // We need a full string for OpcStr here because we need to deal with case like
1293: // INT_PTX_RECIP.
1294: class F_MATH_1<string OpcStr, RegTyInfo dst, RegTyInfo src, Intrinsic IntOP,
1295:                list<Predicate> Preds = []>
1296:   : BasicNVPTXInst<(outs dst.RC:$dst),
1297:               (ins src.RC:$src0),
1298:               OpcStr,
1299:               [(set dst.Ty:$dst, (IntOP src.Ty:$src0))]>,
1300:     Requires<Preds>;
1301:
1302: // We need a full string for OpcStr here because we need to deal with the case
1303: // like INT_PTX_NATIVE_POWR_F.
1304: class F_MATH_2<string OpcStr, NVPTXRegClass t_regclass,
1305:   NVPTXRegClass s0_regclass, NVPTXRegClass s1_regclass, Intrinsic IntOP,
1306:   list<Predicate> Preds = []>
1307:             : BasicNVPTXInst<(outs t_regclass:$dst),
1308:               (ins s0_regclass:$src0, s1_regclass:$src1),
1309:             OpcStr,
1310:         [(set t_regclass:$dst, (IntOP s0_regclass:$src0, s1_regclass:$src1))]>,
1311:         Requires<Preds>;
1312:
1313: class F_MATH_3<string OpcStr, NVPTXRegClass t_regclass,
1314:   NVPTXRegClass s0_regclass, NVPTXRegClass s1_regclass,
1315:   NVPTXRegClass s2_regclass, Intrinsic IntOP, list<Predicate> Preds = []>
1316:             : BasicNVPTXInst<(outs t_regclass:$dst),
1317:               (ins s0_regclass:$src0, s1_regclass:$src1, s2_regclass:$src2),
1318:             OpcStr,
1319:         [(set t_regclass:$dst,
1320:           (IntOP s0_regclass:$src0, s1_regclass:$src1, s2_regclass:$src2))]>,
1321:           Requires<Preds>;
1322:
1323: //
1324: // MISC
1325: //
1326:
1327: def INT_NVVM_NANOSLEEP_I : BasicNVPTXInst<(outs), (ins i32imm:$i), "nanosleep.u32",
1328:                              [(int_nvvm_nanosleep imm:$i)]>,
1329:         Requires<[hasPTX<63>, hasSM<70>]>;
1330: def INT_NVVM_NANOSLEEP_R : BasicNVPTXInst<(outs), (ins B32:$i), "nanosleep.u32",
1331:                              [(int_nvvm_nanosleep i32:$i)]>,
1332:         Requires<[hasPTX<63>, hasSM<70>]>;
1333:
1334: def Hexu16imm : Operand<i16> {
1335:   let PrintMethod = "printHexUImm<16>";
1336: }
1337:
1338: let hasSideEffects = 1 in {
1339: // Performance Monitor events
1340: def INT_PM_EVENT_MASK : BasicNVPTXInst<(outs),
1341:                         (ins Hexu16imm:$mask),
1342:                         "pmevent.mask",
1343:                         [(int_nvvm_pm_event_mask timm:$mask)]>,
1344:                         Requires<[hasSM<20>, hasPTX<30>]>;
1345: } // hasSideEffects
1346:
1347: //
1348: // Min Max
1349: //
1350:
1351: def : Pat<(int_nvvm_fmin_f f32:$a, f32:$b), (MIN_f32_rr $a, $b, NoFTZ)>;
1352: def : Pat<(int_nvvm_fmin_ftz_f f32:$a, f32:$b), (MIN_f32_rr $a, $b, FTZ)>;
1353:
1354: let Predicates = [hasPTX<70>, hasSM<80>] in {
1355:   def : Pat<(int_nvvm_fmin_nan_f f32:$a, f32:$b), (MIN_NAN_f32_rr $a, $b, NoFTZ)>;
1356:   def : Pat<(int_nvvm_fmin_ftz_nan_f f32:$a, f32:$b), (MIN_NAN_f32_rr $a, $b, FTZ)>;
1357: }
1358:
1359: def INT_NVVM_FMIN_XORSIGN_ABS_F :
1360:  F_MATH_2<"min.xorsign.abs.f32", B32, B32, B32, int_nvvm_fmin_xorsign_abs_f,
1361:     [hasPTX<72>, hasSM<86>]>;
1362: def INT_NVVM_FMIN_FTZ_XORSIGN_ABS_F :
1363:   F_MATH_2<"min.ftz.xorsign.abs.f32", B32, B32, B32, int_nvvm_fmin_ftz_xorsign_abs_f,
1364:     [hasPTX<72>, hasSM<86>]>;
1365: def INT_NVVM_FMIN_NAN_XORSIGN_ABS_F :
1366:   F_MATH_2<"min.NaN.xorsign.abs.f32", B32, B32, B32, int_nvvm_fmin_nan_xorsign_abs_f,
1367:     [hasPTX<72>, hasSM<86>]>;
1368: def INT_NVVM_FMIN_FTZ_NAN_XORSIGN_ABS_F :
1369:   F_MATH_2<"min.ftz.NaN.xorsign.abs.f32", B32, B32, B32, int_nvvm_fmin_ftz_nan_xorsign_abs_f,
1370:     [hasPTX<72>, hasSM<86>]>;
1371:
1372:
1373: def : Pat<(int_nvvm_fmax_f f32:$a, f32:$b), (MAX_f32_rr $a, $b, NoFTZ)>;
1374: def : Pat<(int_nvvm_fmax_ftz_f f32:$a, f32:$b), (MAX_f32_rr $a, $b, FTZ)>;
1375:
1376: let Predicates = [hasPTX<70>, hasSM<80>] in {
1377:   def : Pat<(int_nvvm_fmax_nan_f f32:$a, f32:$b), (MAX_NAN_f32_rr $a, $b, NoFTZ)>;
1378:   def : Pat<(int_nvvm_fmax_ftz_nan_f f32:$a, f32:$b), (MAX_NAN_f32_rr $a, $b, FTZ)>;
1379: }
1380:
1381: def INT_NVVM_FMAX_XORSIGN_ABS_F :
1382:   F_MATH_2<"max.xorsign.abs.f32", B32, B32, B32, int_nvvm_fmax_xorsign_abs_f,
1383:     [hasPTX<72>, hasSM<86>]>;
1384: def INT_NVVM_FMAX_FTZ_XORSIGN_ABS_F :
1385:   F_MATH_2<"max.ftz.xorsign.abs.f32", B32, B32, B32, int_nvvm_fmax_ftz_xorsign_abs_f,
1386:     [hasPTX<72>, hasSM<86>]>;
1387: def INT_NVVM_FMAX_NAN_XORSIGN_ABS_F :
1388:   F_MATH_2<"max.NaN.xorsign.abs.f32", B32, B32, B32, int_nvvm_fmax_nan_xorsign_abs_f,
1389:     [hasPTX<72>, hasSM<86>]>;
1390: def INT_NVVM_FMAX_FTZ_NAN_XORSIGN_ABS_F :
1391:   F_MATH_2<"max.ftz.NaN.xorsign.abs.f32", B32, B32, B32, int_nvvm_fmax_ftz_nan_xorsign_abs_f,
1392:     [hasPTX<72>, hasSM<86>]>;
1393:
1394: def : Pat<(int_nvvm_fmin_d f64:$a, f64:$b), (MIN_f64_rr $a, $b)>;
1395: def : Pat<(int_nvvm_fmax_d f64:$a, f64:$b), (MAX_f64_rr $a, $b)>;
1396:
1397: //
1398: // Min Max f16, f16x2, bf16, bf16x2
1399: //
1400:
```
- EN: This range uses TableGen DSL to describe records such as F_MATH_1, F_MATH_2, F_MATH_3, INT_NVVM_NANOSLEEP_I; LLVM later expands these declarations into generated backend tables and helper code.
- CN: 这一段使用 TableGen DSL 描述 F_MATH_1、F_MATH_2、F_MATH_3、INT_NVVM_NANOSLEEP_I 等记录；LLVM 后续会把这些声明展开为生成的后端表和辅助代码。

### Lines 1401-1540
```tablegen
1401: class MIN_MAX_TUPLE<string V, Intrinsic I, NVPTXRegClass RC,
1402:                     list<Predicate> Preds = [hasPTX<70>, hasSM<80>]> {
1403:   string Variant = V;
1404:   Intrinsic Intr = I;
1405:   NVPTXRegClass RegClass = RC;
1406:   list<Predicate> Predicates = Preds;
1407: }
1408:
1409: multiclass MIN_MAX<string IntName> {
1410:   foreach P = [
1411:     MIN_MAX_TUPLE<"_f16", !if(!eq(IntName, "min"), int_nvvm_fmin_f16,
1412:       int_nvvm_fmax_f16), B16>,
1413:     MIN_MAX_TUPLE<"_ftz_f16", !if(!eq(IntName, "min"), int_nvvm_fmin_ftz_f16,
1414:       int_nvvm_fmax_ftz_f16), B16>,
1415:     MIN_MAX_TUPLE<"_NaN_f16", !if(!eq(IntName, "min"), int_nvvm_fmin_nan_f16,
1416:       int_nvvm_fmax_nan_f16), B16>,
1417:     MIN_MAX_TUPLE<"_ftz_NaN_f16", !if(!eq(IntName, "min"),
1418:       int_nvvm_fmin_ftz_nan_f16, int_nvvm_fmax_ftz_nan_f16), B16>,
1419:     MIN_MAX_TUPLE<"_xorsign_abs_f16", !if(!eq(IntName, "min"),
1420:       int_nvvm_fmin_xorsign_abs_f16, int_nvvm_fmax_xorsign_abs_f16),
1421:       B16, [hasPTX<72>, hasSM<86>]>,
1422:     MIN_MAX_TUPLE<"_ftz_xorsign_abs_f16", !if(!eq(IntName, "min"),
1423:       int_nvvm_fmin_ftz_xorsign_abs_f16, int_nvvm_fmax_ftz_xorsign_abs_f16),
1424:       B16, [hasPTX<72>, hasSM<86>]>,
1425:     MIN_MAX_TUPLE<"_NaN_xorsign_abs_f16", !if(!eq(IntName, "min"),
1426:       int_nvvm_fmin_nan_xorsign_abs_f16, int_nvvm_fmax_nan_xorsign_abs_f16),
1427:       B16, [hasPTX<72>, hasSM<86>]>,
1428:     MIN_MAX_TUPLE<"_ftz_NaN_xorsign_abs_f16", !if(!eq(IntName, "min"),
1429:       int_nvvm_fmin_ftz_nan_xorsign_abs_f16,
1430:       int_nvvm_fmax_ftz_nan_xorsign_abs_f16), B16, [hasPTX<72>, hasSM<86>]>,
1431:     MIN_MAX_TUPLE<"_f16x2", !if(!eq(IntName, "min"), int_nvvm_fmin_f16x2,
1432:       int_nvvm_fmax_f16x2), B32>,
1433:     MIN_MAX_TUPLE<"_ftz_f16x2", !if(!eq(IntName, "min"),
1434:       int_nvvm_fmin_ftz_f16x2, int_nvvm_fmax_ftz_f16x2), B32>,
1435:     MIN_MAX_TUPLE<"_NaN_f16x2", !if(!eq(IntName, "min"),
1436:       int_nvvm_fmin_nan_f16x2, int_nvvm_fmax_nan_f16x2), B32>,
1437:     MIN_MAX_TUPLE<"_ftz_NaN_f16x2", !if(!eq(IntName, "min"),
1438:       int_nvvm_fmin_ftz_nan_f16x2, int_nvvm_fmax_ftz_nan_f16x2), B32>,
1439:     MIN_MAX_TUPLE<"_xorsign_abs_f16x2", !if(!eq(IntName, "min"),
1440:       int_nvvm_fmin_xorsign_abs_f16x2, int_nvvm_fmax_xorsign_abs_f16x2),
1441:       B32, [hasPTX<72>, hasSM<86>]>,
1442:     MIN_MAX_TUPLE<"_ftz_xorsign_abs_f16x2", !if(!eq(IntName, "min"),
1443:       int_nvvm_fmin_ftz_xorsign_abs_f16x2, int_nvvm_fmax_ftz_xorsign_abs_f16x2),
1444:       B32, [hasPTX<72>, hasSM<86>]>,
1445:     MIN_MAX_TUPLE<"_NaN_xorsign_abs_f16x2", !if(!eq(IntName, "min"),
1446:       int_nvvm_fmin_nan_xorsign_abs_f16x2, int_nvvm_fmax_nan_xorsign_abs_f16x2),
1447:       B32, [hasPTX<72>, hasSM<86>]>,
1448:     MIN_MAX_TUPLE<"_ftz_NaN_xorsign_abs_f16x2", !if(!eq(IntName, "min"),
1449:       int_nvvm_fmin_ftz_nan_xorsign_abs_f16x2,
1450:       int_nvvm_fmax_ftz_nan_xorsign_abs_f16x2),
1451:       B32, [hasPTX<72>, hasSM<86>]>,
1452:     MIN_MAX_TUPLE<"_bf16", !if(!eq(IntName, "min"),
1453:       int_nvvm_fmin_bf16, int_nvvm_fmax_bf16), B16>,
1454:     MIN_MAX_TUPLE<"_NaN_bf16", !if(!eq(IntName, "min"), int_nvvm_fmin_nan_bf16,
1455:       int_nvvm_fmax_nan_bf16), B16>,
1456:     MIN_MAX_TUPLE<"_xorsign_abs_bf16", !if(!eq(IntName, "min"),
1457:       int_nvvm_fmin_xorsign_abs_bf16, int_nvvm_fmax_xorsign_abs_bf16),
1458:       B16, [hasPTX<72>, hasSM<86>]>,
1459:     MIN_MAX_TUPLE<"_NaN_xorsign_abs_bf16", !if(!eq(IntName, "min"),
1460:       int_nvvm_fmin_nan_xorsign_abs_bf16, int_nvvm_fmax_nan_xorsign_abs_bf16),
1461:       B16, [hasPTX<72>, hasSM<86>]>,
1462:     MIN_MAX_TUPLE<"_bf16x2", !if(!eq(IntName, "min"), int_nvvm_fmin_bf16x2,
1463:       int_nvvm_fmax_bf16x2), B32>,
1464:     MIN_MAX_TUPLE<"_NaN_bf16x2", !if(!eq(IntName, "min"),
1465:       int_nvvm_fmin_nan_bf16x2, int_nvvm_fmax_nan_bf16x2), B32>,
1466:     MIN_MAX_TUPLE<"_xorsign_abs_bf16x2", !if(!eq(IntName, "min"),
1467:       int_nvvm_fmin_xorsign_abs_bf16x2, int_nvvm_fmax_xorsign_abs_bf16x2),
1468:       B32, [hasPTX<72>, hasSM<86>]>,
1469:     MIN_MAX_TUPLE<"_NaN_xorsign_abs_bf16x2", !if(!eq(IntName, "min"),
1470:       int_nvvm_fmin_nan_xorsign_abs_bf16x2,
1471:       int_nvvm_fmax_nan_xorsign_abs_bf16x2),
1472:       B32, [hasPTX<72>, hasSM<86>]>] in {
1473:         def P.Variant : F_MATH_2<!strconcat(
1474:           IntName, !subst("_", ".", P.Variant)),
1475:           P.RegClass, P.RegClass, P.RegClass, P.Intr, P.Predicates>;
1476:   }
1477: }
1478:
1479: defm INT_NVVM_FMIN : MIN_MAX<"min">;
1480: defm INT_NVVM_FMAN : MIN_MAX<"max">;
1481:
1482: //
1483: // Multiplication
1484: //
1485:
1486: def : Pat<(int_nvvm_mulhi_s i16:$a, i16:$b), (MUL_HI_S16rr $a, $b)>;
1487: def : Pat<(int_nvvm_mulhi_us i16:$a, i16:$b), (MUL_HI_U16rr $a, $b)>;
1488: def : Pat<(int_nvvm_mulhi_i i32:$a, i32:$b), (MUL_HI_S32rr $a, $b)>;
1489: def : Pat<(int_nvvm_mulhi_ui i32:$a, i32:$b), (MUL_HI_U32rr $a, $b)>;
1490: def : Pat<(int_nvvm_mulhi_ll i64:$a, i64:$b), (MUL_HI_S64rr $a, $b)>;
1491: def : Pat<(int_nvvm_mulhi_ull i64:$a, i64:$b), (MUL_HI_U64rr $a, $b)>;
1492:
1493: def INT_NVVM_MUL_RN_FTZ_F : F_MATH_2<"mul.rn.ftz.f32", B32, B32, B32, int_nvvm_mul_rn_ftz_f>;
1494: def INT_NVVM_MUL_RN_F : F_MATH_2<"mul.rn.f32", B32, B32, B32, int_nvvm_mul_rn_f>;
1495: def INT_NVVM_MUL_RZ_FTZ_F : F_MATH_2<"mul.rz.ftz.f32", B32, B32, B32, int_nvvm_mul_rz_ftz_f>;
1496: def INT_NVVM_MUL_RZ_F : F_MATH_2<"mul.rz.f32", B32, B32, B32, int_nvvm_mul_rz_f>;
1497: def INT_NVVM_MUL_RM_FTZ_F : F_MATH_2<"mul.rm.ftz.f32", B32, B32, B32, int_nvvm_mul_rm_ftz_f>;
1498: def INT_NVVM_MUL_RM_F : F_MATH_2<"mul.rm.f32", B32, B32, B32, int_nvvm_mul_rm_f>;
1499: def INT_NVVM_MUL_RP_FTZ_F : F_MATH_2<"mul.rp.ftz.f32", B32, B32, B32, int_nvvm_mul_rp_ftz_f>;
1500: def INT_NVVM_MUL_RP_F : F_MATH_2<"mul.rp.f32", B32, B32, B32, int_nvvm_mul_rp_f>;
1501:
1502: def INT_NVVM_MUL_RN_D : F_MATH_2<"mul.rn.f64", B64, B64, B64, int_nvvm_mul_rn_d>;
1503: def INT_NVVM_MUL_RZ_D : F_MATH_2<"mul.rz.f64", B64, B64, B64, int_nvvm_mul_rz_d>;
1504: def INT_NVVM_MUL_RM_D : F_MATH_2<"mul.rm.f64", B64, B64, B64, int_nvvm_mul_rm_d>;
1505: def INT_NVVM_MUL_RP_D : F_MATH_2<"mul.rp.f64", B64, B64, B64, int_nvvm_mul_rp_d>;
1506:
1507: def INT_NVVM_MUL24_I : F_MATH_2<"mul24.lo.s32", B32, B32, B32, int_nvvm_mul24_i>;
1508: def INT_NVVM_MUL24_UI : F_MATH_2<"mul24.lo.u32", B32, B32, B32, int_nvvm_mul24_ui>;
1509:
1510: def INT_NVVM_MUL_RN_SAT_F16 : F_MATH_2<"mul.rn.sat.f16", B16, B16, B16, int_nvvm_mul_rn_sat_f16>;
1511: def INT_NVVM_MUL_RN_FTZ_SAT_F16 : F_MATH_2<"mul.rn.ftz.sat.f16", B16, B16, B16, int_nvvm_mul_rn_ftz_sat_f16>;
1512: def INT_NVVM_MUL_RN_SAT_F16X2 : F_MATH_2<"mul.rn.sat.f16x2", B32, B32, B32, int_nvvm_mul_rn_sat_v2f16>;
1513: def INT_NVVM_MUL_RN_FTZ_SAT_F16X2 : F_MATH_2<"mul.rn.ftz.sat.f16x2", B32, B32, B32, int_nvvm_mul_rn_ftz_sat_v2f16>;
1514:
1515: //
1516: // Div
1517: //
1518:
1519: def : Pat<(int_nvvm_div_approx_ftz_f f32:$a, f32:$b), (DIV_APPROX_F32_rr $a, $b, FTZ)>;
1520: def : Pat<(int_nvvm_div_approx_f f32:$a, f32:$b), (DIV_APPROX_F32_rr $a, $b, NoFTZ)>;
1521:
1522: def INT_NVVM_DIV_RN_FTZ_F : F_MATH_2<"div.rn.ftz.f32", B32, B32, B32, int_nvvm_div_rn_ftz_f>;
1523: def INT_NVVM_DIV_RN_F     : F_MATH_2<"div.rn.f32", B32, B32, B32, int_nvvm_div_rn_f>;
1524: def INT_NVVM_DIV_RZ_FTZ_F : F_MATH_2<"div.rz.ftz.f32", B32, B32, B32, int_nvvm_div_rz_ftz_f>;
1525: def INT_NVVM_DIV_RZ_F     : F_MATH_2<"div.rz.f32", B32, B32, B32, int_nvvm_div_rz_f>;
1526: def INT_NVVM_DIV_RM_FTZ_F : F_MATH_2<"div.rm.ftz.f32", B32, B32, B32, int_nvvm_div_rm_ftz_f>;
1527: def INT_NVVM_DIV_RM_F     : F_MATH_2<"div.rm.f32", B32, B32, B32, int_nvvm_div_rm_f>;
1528: def INT_NVVM_DIV_RP_FTZ_F : F_MATH_2<"div.rp.ftz.f32", B32, B32, B32, int_nvvm_div_rp_ftz_f>;
1529: def INT_NVVM_DIV_RP_F     : F_MATH_2<"div.rp.f32", B32, B32, B32, int_nvvm_div_rp_f>;
1530:
1531: def INT_NVVM_DIV_RN_D : F_MATH_2<"div.rn.f64", B64, B64, B64, int_nvvm_div_rn_d>;
1532: def INT_NVVM_DIV_RZ_D : F_MATH_2<"div.rz.f64", B64, B64, B64, int_nvvm_div_rz_d>;
1533: def INT_NVVM_DIV_RM_D : F_MATH_2<"div.rm.f64", B64, B64, B64, int_nvvm_div_rm_d>;
1534: def INT_NVVM_DIV_RP_D : F_MATH_2<"div.rp.f64", B64, B64, B64, int_nvvm_div_rp_d>;
1535:
1536: def : Pat<(int_nvvm_div_full f32:$a, f32:$b), (FDIV32rr $a, $b, NoFTZ)>;
1537: def : Pat<(int_nvvm_div_full f32:$a, fpimm:$b), (FDIV32ri $a, f32imm:$b, NoFTZ)>;
1538: def : Pat<(int_nvvm_div_full_ftz f32:$a, f32:$b), (FDIV32rr $a, $b, FTZ)>;
1539: def : Pat<(int_nvvm_div_full_ftz f32:$a, fpimm:$b), (FDIV32ri $a, f32imm:$b, FTZ)>;
1540:
```
- EN: This range uses TableGen DSL to describe records such as MIN_MAX_TUPLE, MIN_MAX, P.Variant, INT_NVVM_FMIN; LLVM later expands these declarations into generated backend tables and helper code.
- CN: 这一段使用 TableGen DSL 描述 MIN_MAX_TUPLE、MIN_MAX、P.Variant、INT_NVVM_FMIN 等记录；LLVM 后续会把这些声明展开为生成的后端表和辅助代码。

### Lines 1541-1680
```tablegen
1541: //
1542: // Sad
1543: //
1544:
1545: def INT_NVVM_SAD_S : F_MATH_3<"sad.s16", B16, B16, B16, B16, int_nvvm_sad_s>;
1546: def INT_NVVM_SAD_US : F_MATH_3<"sad.u16", B16, B16, B16, B16, int_nvvm_sad_us>;
1547: def INT_NVVM_SAD_I : F_MATH_3<"sad.s32", B32, B32, B32, B32, int_nvvm_sad_i>;
1548: def INT_NVVM_SAD_UI : F_MATH_3<"sad.u32", B32, B32, B32, B32, int_nvvm_sad_ui>;
1549: def INT_NVVM_SAD_LL : F_MATH_3<"sad.s64", B64, B64, B64, B64, int_nvvm_sad_ll>;
1550: def INT_NVVM_SAD_ULL : F_MATH_3<"sad.u64", B64, B64, B64, B64, int_nvvm_sad_ull>;
1551:
1552: //
1553: // Floor  Ceil
1554: //
1555:
1556: def : Pat<(int_nvvm_floor_ftz_f f32:$a), (CVT_f32_f32 $a, CvtRMI_FTZ)>;
1557: def : Pat<(int_nvvm_floor_f f32:$a),     (CVT_f32_f32 $a, CvtRMI)>;
1558: def : Pat<(int_nvvm_floor_d f64:$a),     (CVT_f64_f64 $a, CvtRMI)>;
1559:
1560: def : Pat<(int_nvvm_ceil_ftz_f f32:$a), (CVT_f32_f32 $a, CvtRPI_FTZ)>;
1561: def : Pat<(int_nvvm_ceil_f f32:$a),     (CVT_f32_f32 $a, CvtRPI)>;
1562: def : Pat<(int_nvvm_ceil_d f64:$a),     (CVT_f64_f64 $a, CvtRPI)>;
1563:
1564: //
1565: // Abs
1566: //
1567:
1568: multiclass F_ABS<string suffix, RegTyInfo RT, bit support_ftz, list<Predicate> preds = []> {
1569:   def "" : F_MATH_1<"abs." # suffix, RT, RT, int_nvvm_fabs, preds>;
1570:   if support_ftz then
1571:     def _FTZ : F_MATH_1<"abs.ftz." # suffix, RT, RT, int_nvvm_fabs_ftz, preds>;
1572: }
1573:
1574: defm ABS_F16 : F_ABS<"f16", F16RT, support_ftz = true, preds = [hasPTX<65>, hasSM<53>]>;
1575: defm ABS_F16X2 : F_ABS<"f16x2", F16X2RT, support_ftz = true, preds = [hasPTX<65>, hasSM<53>]>;
1576:
1577: defm ABS_BF16 : F_ABS<"bf16", BF16RT, support_ftz = false, preds = [hasPTX<70>, hasSM<80>]>;
1578: defm ABS_BF16X2 : F_ABS<"bf16x2", BF16X2RT, support_ftz = false, preds = [hasPTX<70>, hasSM<80>]>;
1579:
1580: defm ABS_F32 : F_ABS<"f32", F32RT, support_ftz = true>;
1581: defm ABS_F64 : F_ABS<"f64", F64RT, support_ftz = false>;
1582:
1583: //
1584: // copysign
1585: //
1586:
1587: def fcopysign_nvptx : SDNode<"NVPTXISD::FCOPYSIGN", SDTFPBinOp>;
1588:
1589: foreach t = [F32RT, F64RT] in
1590:   def COPYSIGN_ # t :
1591:       BasicNVPTXInst<(outs t.RC:$dst), (ins t.RC:$src0, t.RC:$src1),
1592:                 "copysign." # t.PtxType,
1593:               [(set t.Ty:$dst, (fcopysign_nvptx t.Ty:$src1, t.Ty:$src0))]>;
1594:
1595: //
1596: // Neg bf16, bf16x2
1597: //
1598:
1599: def INT_NVVM_NEG_BF16 : F_MATH_1<"neg.bf16", BF16RT,
1600:   BF16RT, int_nvvm_neg_bf16, [hasPTX<70>, hasSM<80>]>;
1601: def INT_NVVM_NEG_BF16X2 : F_MATH_1<"neg.bf16x2", BF16X2RT,
1602:   BF16X2RT, int_nvvm_neg_bf16x2, [hasPTX<70>, hasSM<80>]>;
1603:
1604: //
1605: // Round
1606: //
1607:
1608: def : Pat<(int_nvvm_round_ftz_f f32:$a), (CVT_f32_f32 $a, CvtRNI_FTZ)>;
1609: def : Pat<(int_nvvm_round_f f32:$a),     (CVT_f32_f32 $a, CvtRNI)>;
1610: def : Pat<(int_nvvm_round_d f64:$a),     (CVT_f64_f64 $a, CvtRNI)>;
1611:
1612: //
1613: // Trunc
1614: //
1615:
1616: def : Pat<(int_nvvm_trunc_ftz_f f32:$a), (CVT_f32_f32 $a, CvtRZI_FTZ)>;
1617: def : Pat<(int_nvvm_trunc_f f32:$a),     (CVT_f32_f32 $a, CvtRZI)>;
1618: def : Pat<(int_nvvm_trunc_d f64:$a),     (CVT_f64_f64 $a, CvtRZI)>;
1619:
1620: //
1621: // Saturate
1622: //
1623:
1624: def : Pat<(int_nvvm_saturate_ftz_f f32:$a), (CVT_f32_f32 $a, CvtSAT_FTZ)>;
1625: def : Pat<(int_nvvm_saturate_f f32:$a),     (CVT_f32_f32 $a, CvtSAT)>;
1626: def : Pat<(int_nvvm_saturate_d f64:$a),     (CVT_f64_f64 $a, CvtSAT)>;
1627:
1628: //
1629: // Exp2  Log2
1630: //
1631:
1632: def : Pat<(f32 (int_nvvm_ex2_approx_ftz f32:$a)), (EX2_APPROX_f32 $a, FTZ)>;
1633: def : Pat<(f32 (int_nvvm_ex2_approx f32:$a)), (EX2_APPROX_f32 $a, NoFTZ)>;
1634:
1635: let Predicates = [hasPTX<70>, hasSM<75>] in {
1636:   def : Pat<(f16 (int_nvvm_ex2_approx f16:$a)), (EX2_APPROX_f16 $a)>;
1637:   def : Pat<(v2f16 (int_nvvm_ex2_approx v2f16:$a)), (EX2_APPROX_f16x2 $a)>;
1638: }
1639:
1640: let Predicates = [hasPTX<78>, hasSM<90>] in {
1641:   def : Pat<(bf16 (int_nvvm_ex2_approx_ftz bf16:$a)), (EX2_APPROX_bf16 $a)>;
1642:   def : Pat<(v2bf16 (int_nvvm_ex2_approx_ftz v2bf16:$a)), (EX2_APPROX_bf16x2 $a)>;
1643: }
1644:
1645: def LG2_APPROX_f32 :
1646:   BasicFlagsNVPTXInst<(outs B32:$dst), (ins B32:$src), (ins FTZFlag:$ftz),
1647:     "lg2.approx$ftz.f32",
1648:     [(set f32:$dst, (flog2 f32:$src))]>;
1649:
1650: def LG2_APPROX_f64 :
1651:   BasicNVPTXInst<(outs B64:$dst), (ins B64:$src),
1652:     "lg2.approx.f64",
1653:     [(set f64:$dst, (flog2 f64:$src))]>;
1654:
1655: def : Pat<(int_nvvm_lg2_approx_ftz_f f32:$a), (LG2_APPROX_f32 $a, FTZ)>;
1656: def : Pat<(int_nvvm_lg2_approx_f f32:$a), (LG2_APPROX_f32 $a, NoFTZ)>;
1657: def : Pat<(int_nvvm_lg2_approx_d f64:$a), (LG2_APPROX_f64 $a)>;
1658:
1659: //
1660: // Sin  Cos
1661: //
1662:
1663: def : Pat<(int_nvvm_sin_approx_ftz_f f32:$a), (SIN_APPROX_f32 $a, FTZ)>;
1664: def : Pat<(int_nvvm_sin_approx_f f32:$a), (SIN_APPROX_f32 $a, NoFTZ)>;
1665: def : Pat<(int_nvvm_cos_approx_ftz_f f32:$a), (COS_APPROX_f32 $a, FTZ)>;
1666: def : Pat<(int_nvvm_cos_approx_f f32:$a), (COS_APPROX_f32 $a, NoFTZ)>;
1667:
1668: //
1669: // Fma
1670: //
1671:
1672: class FMA_TUPLE<string V, Intrinsic I, NVPTXRegClass RC,
1673:                 list<Predicate> Preds = []> {
1674:   string Variant = V;
1675:   Intrinsic Intr = I;
1676:   NVPTXRegClass RegClass = RC;
1677:   list<Predicate> Predicates = Preds;
1678: }
1679:
1680: multiclass FMA_INST {
```
- EN: This range uses TableGen DSL to describe records such as INT_NVVM_SAD_S, INT_NVVM_SAD_US, INT_NVVM_SAD_I, INT_NVVM_SAD_UI; LLVM later expands these declarations into generated backend tables and helper code.
- CN: 这一段使用 TableGen DSL 描述 INT_NVVM_SAD_S、INT_NVVM_SAD_US、INT_NVVM_SAD_I、INT_NVVM_SAD_UI 等记录；LLVM 后续会把这些声明展开为生成的后端表和辅助代码。

### Lines 1681-1820
```tablegen
1681:   foreach P = [
1682:     FMA_TUPLE<"_rn_f64", int_nvvm_fma_rn_d, B64>,
1683:     FMA_TUPLE<"_rz_f64", int_nvvm_fma_rz_d, B64>,
1684:     FMA_TUPLE<"_rm_f64", int_nvvm_fma_rm_d, B64>,
1685:     FMA_TUPLE<"_rp_f64", int_nvvm_fma_rp_d, B64>,
1686:
1687:     FMA_TUPLE<"_rn_ftz_f32", int_nvvm_fma_rn_ftz_f, B32>,
1688:     FMA_TUPLE<"_rn_ftz_sat_f32", int_nvvm_fma_rn_ftz_sat_f, B32>,
1689:     FMA_TUPLE<"_rn_f32", int_nvvm_fma_rn_f, B32>,
1690:     FMA_TUPLE<"_rn_sat_f32", int_nvvm_fma_rn_sat_f, B32>,
1691:     FMA_TUPLE<"_rz_ftz_f32", int_nvvm_fma_rz_ftz_f, B32>,
1692:     FMA_TUPLE<"_rz_ftz_sat_f32", int_nvvm_fma_rz_ftz_sat_f, B32>,
1693:     FMA_TUPLE<"_rz_f32", int_nvvm_fma_rz_f, B32>,
1694:     FMA_TUPLE<"_rz_sat_f32", int_nvvm_fma_rz_sat_f, B32>,
1695:     FMA_TUPLE<"_rm_f32", int_nvvm_fma_rm_f, B32>,
1696:     FMA_TUPLE<"_rm_sat_f32", int_nvvm_fma_rm_sat_f, B32>,
1697:     FMA_TUPLE<"_rm_ftz_f32", int_nvvm_fma_rm_ftz_f, B32>,
1698:     FMA_TUPLE<"_rm_ftz_sat_f32", int_nvvm_fma_rm_ftz_sat_f, B32>,
1699:     FMA_TUPLE<"_rp_f32", int_nvvm_fma_rp_f, B32>,
1700:     FMA_TUPLE<"_rp_sat_f32", int_nvvm_fma_rp_sat_f, B32>,
1701:     FMA_TUPLE<"_rp_ftz_f32", int_nvvm_fma_rp_ftz_f, B32>,
1702:     FMA_TUPLE<"_rp_ftz_sat_f32", int_nvvm_fma_rp_ftz_sat_f, B32>,
1703:
1704:     FMA_TUPLE<"_rn_f16", int_nvvm_fma_rn_f16, B16, [hasPTX<42>, hasSM<53>]>,
1705:     FMA_TUPLE<"_rn_ftz_f16", int_nvvm_fma_rn_ftz_f16, B16,
1706:       [hasPTX<42>, hasSM<53>]>,
1707:     FMA_TUPLE<"_rn_sat_f16", int_nvvm_fma_rn_sat_f16, B16,
1708:       [hasPTX<42>, hasSM<53>]>,
1709:     FMA_TUPLE<"_rn_ftz_sat_f16", int_nvvm_fma_rn_ftz_sat_f16, B16,
1710:       [hasPTX<42>, hasSM<53>]>,
1711:     FMA_TUPLE<"_rn_relu_f16", int_nvvm_fma_rn_relu_f16, B16,
1712:       [hasPTX<70>, hasSM<80>]>,
1713:     FMA_TUPLE<"_rn_ftz_relu_f16", int_nvvm_fma_rn_ftz_relu_f16, B16,
1714:       [hasPTX<70>, hasSM<80>]>,
1715:
1716:     FMA_TUPLE<"_rn_bf16", int_nvvm_fma_rn_bf16, B16, [hasPTX<70>, hasSM<80>]>,
1717:     FMA_TUPLE<"_rn_relu_bf16", int_nvvm_fma_rn_relu_bf16, B16,
1718:       [hasPTX<70>, hasSM<80>]>,
1719:
1720:     FMA_TUPLE<"_rn_f16x2", int_nvvm_fma_rn_f16x2, B32,
1721:       [hasPTX<42>, hasSM<53>]>,
1722:     FMA_TUPLE<"_rn_ftz_f16x2", int_nvvm_fma_rn_ftz_f16x2, B32,
1723:       [hasPTX<42>, hasSM<53>]>,
1724:     FMA_TUPLE<"_rn_sat_f16x2", int_nvvm_fma_rn_sat_f16x2, B32,
1725:       [hasPTX<42>, hasSM<53>]>,
1726:     FMA_TUPLE<"_rn_ftz_sat_f16x2", int_nvvm_fma_rn_ftz_sat_f16x2,
1727:       B32, [hasPTX<42>, hasSM<53>]>,
1728:     FMA_TUPLE<"_rn_relu_f16x2", int_nvvm_fma_rn_relu_f16x2, B32,
1729:       [hasPTX<70>, hasSM<80>]>,
1730:     FMA_TUPLE<"_rn_ftz_relu_f16x2", int_nvvm_fma_rn_ftz_relu_f16x2,
1731:       B32, [hasPTX<70>, hasSM<80>]>,
1732:
1733:     FMA_TUPLE<"_rn_bf16x2", int_nvvm_fma_rn_bf16x2, B32,
1734:       [hasPTX<70>, hasSM<80>]>,
1735:     FMA_TUPLE<"_rn_relu_bf16x2", int_nvvm_fma_rn_relu_bf16x2, B32,
1736:       [hasPTX<70>, hasSM<80>]>,
1737:   ] in {
1738:     def P.Variant :
1739:       F_MATH_3<!strconcat("fma", !subst("_", ".", P.Variant)),
1740:         P.RegClass, P.RegClass, P.RegClass, P.RegClass, P.Intr, P.Predicates>;
1741:   }
1742: }
1743:
1744: defm INT_NVVM_FMA : FMA_INST;
1745:
1746: foreach rnd = ["_rn", "_rz", "_rm", "_rp"] in {
1747:   foreach sat = ["", "_sat"] in {
1748:     foreach type = [f16, bf16] in {
1749:       def INT_NVVM_MIXED_FMA # rnd # sat # _f32_ # type : 
1750:         BasicNVPTXInst<(outs B32:$dst), (ins B16:$a, B16:$b, B32:$c),
1751:           !subst("_", ".", "fma" # rnd # sat # "_f32_" # type),
1752:           [(set f32:$dst, 
1753:            (!cast<Intrinsic>("int_nvvm_fma" # rnd # sat # "_f") 
1754:              (f32 (fpextend type:$a)),
1755:              (f32 (fpextend type:$b)),
1756:              f32:$c))]>,
1757:         Requires<[hasSM<100>, hasPTX<86>]>;
1758:     }
1759:   }
1760: }
1761:
1762: // Pattern for llvm.fma.f32 intrinsic when there is no FTZ flag
1763: let Predicates = [hasSM<100>, hasPTX<86>, doNoF32FTZ] in {
1764:   def : Pat<(f32 (fma (f32 (fpextend f16:$a)),
1765:                       (f32 (fpextend f16:$b)), f32:$c)),
1766:             (INT_NVVM_MIXED_FMA_rn_f32_f16 B16:$a, B16:$b, B32:$c)>;
1767:   def : Pat<(f32 (fma (f32 (fpextend bf16:$a)), 
1768:                       (f32 (fpextend bf16:$b)), f32:$c)),
1769:             (INT_NVVM_MIXED_FMA_rn_f32_bf16 B16:$a, B16:$b, B32:$c)>;
1770: }
1771:
1772: foreach ty = [F16RT, F16X2RT, BF16RT, BF16X2RT] in {
1773:   foreach relu = ["", "_relu"] in {
1774:     defvar Intr = !cast<Intrinsic>("int_nvvm_fma_rn_oob" # relu);
1775:     defvar suffix = !subst("_", ".", relu # "_" # ty.PtxType);
1776:     def INT_NVVM_FMA_OOB # relu # ty.PtxType : 
1777:       BasicNVPTXInst<(outs ty.RC:$dst), (ins ty.RC:$a, ty.RC:$b, ty.RC:$c),
1778:         "fma.rn.oob" # suffix,
1779:         [(set ty.Ty:$dst, (Intr ty.Ty:$a, ty.Ty:$b, ty.Ty:$c))]>,
1780:       Requires<[hasPTX<81>, hasSM<90>]>;
1781:   }
1782: }
1783:
1784: //
1785: // Rcp
1786: //
1787:
1788: def INT_NVVM_RCP_RN_FTZ_F : F_MATH_1<"rcp.rn.ftz.f32", F32RT, F32RT, int_nvvm_rcp_rn_ftz_f>;
1789: def INT_NVVM_RCP_RN_F : F_MATH_1<"rcp.rn.f32", F32RT, F32RT, int_nvvm_rcp_rn_f>;
1790: def INT_NVVM_RCP_RZ_FTZ_F : F_MATH_1<"rcp.rz.ftz.f32", F32RT, F32RT, int_nvvm_rcp_rz_ftz_f>;
1791: def INT_NVVM_RCP_RZ_F : F_MATH_1<"rcp.rz.f32", F32RT, F32RT, int_nvvm_rcp_rz_f>;
1792: def INT_NVVM_RCP_RM_FTZ_F : F_MATH_1<"rcp.rm.ftz.f32", F32RT, F32RT, int_nvvm_rcp_rm_ftz_f>;
1793: def INT_NVVM_RCP_RM_F : F_MATH_1<"rcp.rm.f32", F32RT, F32RT, int_nvvm_rcp_rm_f>;
1794: def INT_NVVM_RCP_RP_FTZ_F : F_MATH_1<"rcp.rp.ftz.f32", F32RT, F32RT, int_nvvm_rcp_rp_ftz_f>;
1795: def INT_NVVM_RCP_RP_F : F_MATH_1<"rcp.rp.f32", F32RT, F32RT, int_nvvm_rcp_rp_f>;
1796:
1797: def INT_NVVM_RCP_RN_D : F_MATH_1<"rcp.rn.f64", F64RT, F64RT, int_nvvm_rcp_rn_d>;
1798: def INT_NVVM_RCP_RZ_D : F_MATH_1<"rcp.rz.f64", F64RT, F64RT, int_nvvm_rcp_rz_d>;
1799: def INT_NVVM_RCP_RM_D : F_MATH_1<"rcp.rm.f64", F64RT, F64RT, int_nvvm_rcp_rm_d>;
1800: def INT_NVVM_RCP_RP_D : F_MATH_1<"rcp.rp.f64", F64RT, F64RT, int_nvvm_rcp_rp_d>;
1801:
1802: def INT_NVVM_RCP_APPROX_FTZ_F : F_MATH_1<"rcp.approx.ftz.f32",
1803:   F32RT, F32RT, int_nvvm_rcp_approx_ftz_f>;
1804: def INT_NVVM_RCP_APPROX_FTZ_D : F_MATH_1<"rcp.approx.ftz.f64",
1805:   F64RT, F64RT, int_nvvm_rcp_approx_ftz_d>;
1806:
1807: //
1808: // Sqrt
1809: //
1810:
1811: def INT_NVVM_SQRT_RN_FTZ_F : F_MATH_1<"sqrt.rn.ftz.f32",
1812:   F32RT, F32RT, int_nvvm_sqrt_rn_ftz_f>;
1813: def INT_NVVM_SQRT_RN_F : F_MATH_1<"sqrt.rn.f32", F32RT,
1814:   F32RT, int_nvvm_sqrt_rn_f>;
1815: def INT_NVVM_SQRT_RZ_FTZ_F : F_MATH_1<"sqrt.rz.ftz.f32",
1816:   F32RT, F32RT, int_nvvm_sqrt_rz_ftz_f>;
1817: def INT_NVVM_SQRT_RZ_F : F_MATH_1<"sqrt.rz.f32", F32RT,
1818:   F32RT, int_nvvm_sqrt_rz_f>;
1819: def INT_NVVM_SQRT_RM_FTZ_F : F_MATH_1<"sqrt.rm.ftz.f32",
1820:   F32RT, F32RT, int_nvvm_sqrt_rm_ftz_f>;
```
- EN: This range uses TableGen DSL to describe records such as P.Variant, INT_NVVM_FMA, INT_NVVM_MIXED_FMA, INT_NVVM_FMA_OOB; LLVM later expands these declarations into generated backend tables and helper code.
- CN: 这一段使用 TableGen DSL 描述 P.Variant、INT_NVVM_FMA、INT_NVVM_MIXED_FMA、INT_NVVM_FMA_OOB 等记录；LLVM 后续会把这些声明展开为生成的后端表和辅助代码。

### Lines 1821-1960
```tablegen
1821: def INT_NVVM_SQRT_RM_F : F_MATH_1<"sqrt.rm.f32", F32RT,
1822:   F32RT, int_nvvm_sqrt_rm_f>;
1823: def INT_NVVM_SQRT_RP_FTZ_F : F_MATH_1<"sqrt.rp.ftz.f32",
1824:   F32RT, F32RT, int_nvvm_sqrt_rp_ftz_f>;
1825: def INT_NVVM_SQRT_RP_F : F_MATH_1<"sqrt.rp.f32", F32RT,
1826:   F32RT, int_nvvm_sqrt_rp_f>;
1827: def INT_NVVM_SQRT_APPROX_FTZ_F : F_MATH_1<"sqrt.approx.ftz.f32",
1828:   F32RT, F32RT, int_nvvm_sqrt_approx_ftz_f>;
1829: def INT_NVVM_SQRT_APPROX_F : F_MATH_1<"sqrt.approx.f32",
1830:   F32RT, F32RT, int_nvvm_sqrt_approx_f>;
1831:
1832: def INT_NVVM_SQRT_RN_D : F_MATH_1<"sqrt.rn.f64", F64RT, F64RT, int_nvvm_sqrt_rn_d>;
1833: def INT_NVVM_SQRT_RZ_D : F_MATH_1<"sqrt.rz.f64", F64RT, F64RT, int_nvvm_sqrt_rz_d>;
1834: def INT_NVVM_SQRT_RM_D : F_MATH_1<"sqrt.rm.f64", F64RT, F64RT, int_nvvm_sqrt_rm_d>;
1835: def INT_NVVM_SQRT_RP_D : F_MATH_1<"sqrt.rp.f64", F64RT, F64RT, int_nvvm_sqrt_rp_d>;
1836:
1837: def fsqrt_approx : PatFrags<(ops node:$a),
1838:                             [(fsqrt node:$a),
1839:                              (int_nvvm_sqrt_f node:$a)], [{
1840:   return !usePrecSqrtF32(N);
1841: }]>;
1842:
1843: // nvvm_sqrt intrinsic
1844: def : Pat<(int_nvvm_sqrt_f f32:$a), (INT_NVVM_SQRT_RN_FTZ_F $a)>, Requires<[doF32FTZ]>;
1845: def : Pat<(int_nvvm_sqrt_f f32:$a), (INT_NVVM_SQRT_RN_F $a)>;
1846:
1847: def : Pat<(fsqrt_approx f32:$a), (INT_NVVM_SQRT_APPROX_FTZ_F $a)>, Requires<[doF32FTZ]>;
1848: def : Pat<(fsqrt_approx f32:$a), (INT_NVVM_SQRT_APPROX_F $a)>;
1849:
1850: //
1851: // Rsqrt
1852: //
1853:
1854: foreach t = [F32RT, F64RT] in {
1855:   def RSQRT_APPROX_ # t.Ty :
1856:     BasicFlagsNVPTXInst<(outs t.RC:$dst),
1857:                         (ins t.RC:$a), (ins FTZFlag:$ftz),
1858:                         "rsqrt.approx$ftz.f" # t.Size>;
1859: }
1860:
1861: def : Pat<(int_nvvm_rsqrt_approx_ftz_f f32:$a), (RSQRT_APPROX_f32 $a, FTZ)>;
1862: def : Pat<(int_nvvm_rsqrt_approx_ftz_d f64:$a), (RSQRT_APPROX_f64 $a, FTZ)>;
1863: def : Pat<(int_nvvm_rsqrt_approx_f f32:$a), (RSQRT_APPROX_f32 $a, NoFTZ)>;
1864: def : Pat<(int_nvvm_rsqrt_approx_d f64:$a), (RSQRT_APPROX_f64 $a, NoFTZ)>;
1865:
1866:
1867: // 1.0f / sqrt_approx -> rsqrt_approx
1868: let Predicates = [doRsqrtOpt] in {
1869:   def : Pat<(fdiv fpimm_1, (int_nvvm_sqrt_approx_f f32:$a)),
1870:           (RSQRT_APPROX_f32 $a, NoFTZ)>;
1871:   def : Pat<(fdiv fpimm_1, (int_nvvm_sqrt_approx_ftz_f f32:$a)),
1872:           (RSQRT_APPROX_f32 $a, FTZ)>;
1873:
1874:   // same for int_nvvm_sqrt_f when non-precision sqrt is requested
1875:   def : Pat<(fdiv fpimm_1, (fsqrt_approx f32:$a)),
1876:           (RSQRT_APPROX_f32 $a)>;
1877: }
1878: //
1879: // Add
1880: //
1881:
1882: def INT_NVVM_ADD_RN_SAT_F16 : F_MATH_2<"add.rn.sat.f16", B16, B16, B16, int_nvvm_add_rn_sat_f16>;
1883: def INT_NVVM_ADD_RN_FTZ_SAT_F16 : F_MATH_2<"add.rn.ftz.sat.f16", B16, B16, B16, int_nvvm_add_rn_ftz_sat_f16>;
1884: def INT_NVVM_ADD_RN_SAT_F16X2 : F_MATH_2<"add.rn.sat.f16x2", B32, B32, B32, int_nvvm_add_rn_sat_v2f16>;
1885: def INT_NVVM_ADD_RN_FTZ_SAT_F16X2 : F_MATH_2<"add.rn.ftz.sat.f16x2", B32, B32, B32, int_nvvm_add_rn_ftz_sat_v2f16>;
1886:
1887: def INT_NVVM_ADD_RN_FTZ_F : F_MATH_2<"add.rn.ftz.f32", B32, B32, B32, int_nvvm_add_rn_ftz_f>;
1888: def INT_NVVM_ADD_RN_SAT_FTZ_F : F_MATH_2<"add.rn.sat.ftz.f32", B32, B32, B32, int_nvvm_add_rn_ftz_sat_f>;
1889: def INT_NVVM_ADD_RN_F : F_MATH_2<"add.rn.f32", B32, B32, B32, int_nvvm_add_rn_f>;
1890: def INT_NVVM_ADD_RN_SAT_F : F_MATH_2<"add.rn.sat.f32", B32, B32, B32, int_nvvm_add_rn_sat_f>;
1891: def INT_NVVM_ADD_RZ_FTZ_F : F_MATH_2<"add.rz.ftz.f32", B32, B32, B32, int_nvvm_add_rz_ftz_f>;
1892: def INT_NVVM_ADD_RZ_SAT_FTZ_F : F_MATH_2<"add.rz.sat.ftz.f32", B32, B32, B32, int_nvvm_add_rz_ftz_sat_f>;
1893: def INT_NVVM_ADD_RZ_F : F_MATH_2<"add.rz.f32", B32, B32, B32, int_nvvm_add_rz_f>;
1894: def INT_NVVM_ADD_RZ_SAT_F : F_MATH_2<"add.rz.sat.f32", B32, B32, B32, int_nvvm_add_rz_sat_f>;
1895: def INT_NVVM_ADD_RM_FTZ_F : F_MATH_2<"add.rm.ftz.f32", B32, B32, B32, int_nvvm_add_rm_ftz_f>;
1896: def INT_NVVM_ADD_RM_SAT_FTZ_F : F_MATH_2<"add.rm.sat.ftz.f32", B32, B32, B32, int_nvvm_add_rm_ftz_sat_f>;
1897: def INT_NVVM_ADD_RM_F : F_MATH_2<"add.rm.f32", B32, B32, B32, int_nvvm_add_rm_f>;
1898: def INT_NVVM_ADD_RM_SAT_F : F_MATH_2<"add.rm.sat.f32", B32, B32, B32, int_nvvm_add_rm_sat_f>;
1899: def INT_NVVM_ADD_RP_FTZ_F : F_MATH_2<"add.rp.ftz.f32", B32, B32, B32, int_nvvm_add_rp_ftz_f>;
1900: def INT_NVVM_ADD_RP_SAT_FTZ_F : F_MATH_2<"add.rp.sat.ftz.f32", B32, B32, B32, int_nvvm_add_rp_ftz_sat_f>;
1901: def INT_NVVM_ADD_RP_F : F_MATH_2<"add.rp.f32", B32, B32, B32, int_nvvm_add_rp_f>;
1902: def INT_NVVM_ADD_RP_SAT_F : F_MATH_2<"add.rp.sat.f32", B32, B32, B32, int_nvvm_add_rp_sat_f>;
1903:
1904: def INT_NVVM_ADD_RN_D : F_MATH_2<"add.rn.f64", B64, B64, B64, int_nvvm_add_rn_d>;
1905: def INT_NVVM_ADD_RZ_D : F_MATH_2<"add.rz.f64", B64, B64, B64, int_nvvm_add_rz_d>;
1906: def INT_NVVM_ADD_RM_D : F_MATH_2<"add.rm.f64", B64, B64, B64, int_nvvm_add_rm_d>;
1907: def INT_NVVM_ADD_RP_D : F_MATH_2<"add.rp.f64", B64, B64, B64, int_nvvm_add_rp_d>;
1908:
1909: foreach rnd = ["_rn", "_rz", "_rm", "_rp"] in {
1910:   foreach sat = ["", "_sat"] in {
1911:     foreach type = [f16, bf16] in {
1912:       def INT_NVVM_MIXED_ADD # rnd # sat # _f32_ # type : 
1913:         BasicNVPTXInst<(outs B32:$dst), (ins B16:$a, B32:$b),
1914:           !subst("_", ".", "add" # rnd # sat # "_f32_" # type),
1915:           [(set f32:$dst, 
1916:            (!cast<Intrinsic>("int_nvvm_add" # rnd # sat # "_f") 
1917:              (f32 (fpextend type:$a)),
1918:              f32:$b))]>,
1919:         Requires<[hasSM<100>, hasPTX<86>]>;
1920:     }
1921:   }
1922: }
1923:
1924: // Pattern for fadd when there is no FTZ flag
1925: let Predicates = [hasSM<100>, hasPTX<86>, doNoF32FTZ] in {
1926:   def : Pat<(f32 (fadd (f32 (fpextend f16:$a)), f32:$b)),
1927:             (INT_NVVM_MIXED_ADD_rn_f32_f16 B16:$a, B32:$b)>;
1928:   def : Pat<(f32 (fadd (f32 (fpextend bf16:$a)), f32:$b)),
1929:             (INT_NVVM_MIXED_ADD_rn_f32_bf16 B16:$a, B32:$b)>;
1930: }
1931:
1932: //
1933: // Sub
1934: //
1935:
1936: def sub_rn_sat : SDNode<"NVPTXISD::SUB_RN_SAT", SDTFPBinOp>;
1937: def sub_rn_ftz_sat : 
1938:   SDNode<"NVPTXISD::SUB_RN_FTZ_SAT", SDTFPBinOp>;
1939:   
1940: class INT_NVVM_SUB_RN<RegTyInfo TyInfo, string variant> :
1941:   BasicNVPTXInst<(outs TyInfo.RC:$dst), (ins TyInfo.RC:$a, TyInfo.RC:$b),
1942:     !subst("_", ".", "sub.rn" # variant # "." # TyInfo.PtxType),
1943:     [(set TyInfo.Ty:$dst, 
1944:      (!cast<SDNode>("sub_rn" # variant) TyInfo.Ty:$a, TyInfo.Ty:$b))]>;
1945:
1946: def INT_NVVM_SUB_RN_SAT_F16 : INT_NVVM_SUB_RN<F16RT, "_sat">;
1947: def INT_NVVM_SUB_RN_FTZ_SAT_F16 : INT_NVVM_SUB_RN<F16RT, "_ftz_sat">;
1948: def INT_NVVM_SUB_RN_SAT_F16X2 : INT_NVVM_SUB_RN<F16X2RT, "_sat">;
1949: def INT_NVVM_SUB_RN_FTZ_SAT_F16X2 : INT_NVVM_SUB_RN<F16X2RT, "_ftz_sat">;
1950:
1951: foreach rnd = ["_rn", "_rz", "_rm", "_rp"] in {
1952:   foreach ftz = ["", "_ftz"] in {
1953:     foreach sat = ["", "_sat"] in {
1954:       defvar add_intrin = !cast<Intrinsic>("int_nvvm_add" # rnd # ftz # sat # "_f");
1955:       def INT_NVVM_SUB # rnd # ftz # sat # _F : 
1956:         BasicNVPTXInst<(outs B32:$dst), (ins B32:$a, B32:$b),
1957:           !subst("_", ".", "sub" # rnd # sat # ftz # "_f32"),
1958:           [(set f32:$dst, (add_intrin f32:$a, (f32 (fneg f32:$b))))]>;
1959:     }
1960:   }
```
- EN: This range uses TableGen DSL to describe records such as INT_NVVM_SQRT_RM_F, INT_NVVM_SQRT_RP_FTZ_F, INT_NVVM_SQRT_RP_F, INT_NVVM_SQRT_APPROX_FTZ_F; LLVM later expands these declarations into generated backend tables and helper code.
- CN: 这一段使用 TableGen DSL 描述 INT_NVVM_SQRT_RM_F、INT_NVVM_SQRT_RP_FTZ_F、INT_NVVM_SQRT_RP_F、INT_NVVM_SQRT_APPROX_FTZ_F 等记录；LLVM 后续会把这些声明展开为生成的后端表和辅助代码。

### Lines 1961-2100
```tablegen
1961:   
1962:   defvar add_intrin = !cast<Intrinsic>("int_nvvm_add" # rnd # "_d");
1963:   def INT_NVVM_SUB # rnd # _D : 
1964:     BasicNVPTXInst<(outs B64:$dst), (ins B64:$a, B64:$b),
1965:       !subst("_", ".", "sub" # rnd # "_f64"),
1966:       [(set f64:$dst, (add_intrin f64:$a, (f64 (fneg f64:$b))))]>;
1967: }
1968:
1969: foreach rnd = ["_rn", "_rz", "_rm", "_rp"] in {
1970:   foreach sat = ["", "_sat"] in {
1971:     foreach type = [f16, bf16] in {
1972:       def INT_NVVM_MIXED_SUB # rnd # sat # _f32_ # type : 
1973:         BasicNVPTXInst<(outs B32:$dst), (ins B16:$a, B32:$b),
1974:           !subst("_", ".", "sub" # rnd # sat # "_f32_" # type),
1975:           [(set f32:$dst, 
1976:            (!cast<Intrinsic>("int_nvvm_add" # rnd # sat # "_f") 
1977:              (f32 (fpextend type:$a)),
1978:              (f32 (fneg f32:$b))))]>,
1979:         Requires<[hasSM<100>, hasPTX<86>]>;
1980:     }
1981:   }
1982: }
1983:
1984: // Pattern for fsub when there is no FTZ flag
1985: let Predicates = [hasSM<100>, hasPTX<86>, doNoF32FTZ] in {
1986:   def : Pat<(f32 (fsub (f32 (fpextend f16:$a)), f32:$b)),
1987:             (INT_NVVM_MIXED_SUB_rn_f32_f16 B16:$a, B32:$b)>;
1988:   def : Pat<(f32 (fsub (f32 (fpextend bf16:$a)), f32:$b)),
1989:             (INT_NVVM_MIXED_SUB_rn_f32_bf16 B16:$a, B32:$b)>;
1990: }
1991:
1992: //
1993: // BFIND
1994: //
1995:
1996: foreach t = [I32RT, I64RT] in {
1997:   foreach sign = ["s", "u"] in {
1998:     defvar flo_intrin = !cast<Intrinsic>("int_nvvm_flo_" # sign);
1999:     def BFIND_ # sign # t.Size
2000:       : BasicNVPTXInst<(outs B32:$dst), (ins t.RC:$src),
2001:                   "bfind." # sign # t.Size,
2002:                   [(set i32:$dst, (flo_intrin t.Ty:$src, 0))]>;
2003:
2004:     def BFIND_SHIFTAMT_ # sign # t.Size
2005:       : BasicNVPTXInst<(outs B32:$dst), (ins t.RC:$src),
2006:                   "bfind.shiftamt." # sign # t.Size,
2007:                   [(set i32:$dst, (flo_intrin t.Ty:$src, -1))]>;
2008:   }
2009: }
2010:
2011: //
2012: // szext
2013: //
2014:
2015: foreach sign = ["s", "u"] in {
2016:   foreach mode = ["wrap", "clamp"] in {
2017:     defvar ext = !if(!eq(sign, "s"), "sext", "zext");
2018:     defvar intrin = !cast<Intrinsic>("int_nvvm_" # ext # "_" # mode);
2019:     defm SZEXT_ # sign # _ # mode
2020:       : I3Inst<"szext." # mode # "." # sign # "32",
2021:                intrin, I32RT, commutative = false,
2022:                requires = [hasSM<70>, hasPTX<76>]>;
2023:   }
2024: }
2025:
2026: //
2027: // BMSK
2028: //
2029:
2030: foreach mode = ["wrap", "clamp"] in {
2031:   defvar intrin = !cast<Intrinsic>("int_nvvm_bmsk_" # mode);
2032:   defm BMSK_ # mode
2033:     : I3Inst<"bmsk." # mode # ".b32",
2034:              intrin, I32RT, commutative = false,
2035:              requires = [hasSM<70>, hasPTX<76>]>;
2036: }
2037:
2038: //
2039: // Convert
2040: //
2041:
2042: def : Pat<(int_nvvm_d2f_rn_ftz f64:$a), (CVT_f32_f64 $a, CvtRN_FTZ)>;
2043: def : Pat<(int_nvvm_d2f_rn f64:$a),     (CVT_f32_f64 $a, CvtRN)>;
2044: def : Pat<(int_nvvm_d2f_rz_ftz f64:$a), (CVT_f32_f64 $a, CvtRZ_FTZ)>;
2045: def : Pat<(int_nvvm_d2f_rz f64:$a),     (CVT_f32_f64 $a, CvtRZ)>;
2046: def : Pat<(int_nvvm_d2f_rm_ftz f64:$a), (CVT_f32_f64 $a, CvtRM_FTZ)>;
2047: def : Pat<(int_nvvm_d2f_rm f64:$a),     (CVT_f32_f64 $a, CvtRM)>;
2048: def : Pat<(int_nvvm_d2f_rp_ftz f64:$a), (CVT_f32_f64 $a, CvtRP_FTZ)>;
2049: def : Pat<(int_nvvm_d2f_rp f64:$a),     (CVT_f32_f64 $a, CvtRP)>;
2050:
2051: def : Pat<(int_nvvm_d2i_rn f64:$a), (CVT_s32_f64 $a, CvtRNI)>;
2052: def : Pat<(int_nvvm_d2i_rz f64:$a), (CVT_s32_f64 $a, CvtRZI)>;
2053: def : Pat<(int_nvvm_d2i_rm f64:$a), (CVT_s32_f64 $a, CvtRMI)>;
2054: def : Pat<(int_nvvm_d2i_rp f64:$a), (CVT_s32_f64 $a, CvtRPI)>;
2055:
2056: def : Pat<(int_nvvm_d2ui_rn f64:$a), (CVT_u32_f64 $a, CvtRNI)>;
2057: def : Pat<(int_nvvm_d2ui_rz f64:$a), (CVT_u32_f64 $a, CvtRZI)>;
2058: def : Pat<(int_nvvm_d2ui_rm f64:$a), (CVT_u32_f64 $a, CvtRMI)>;
2059: def : Pat<(int_nvvm_d2ui_rp f64:$a), (CVT_u32_f64 $a, CvtRPI)>;
2060:
2061: def : Pat<(int_nvvm_i2d_rn i32:$a), (CVT_f64_s32 $a, CvtRN)>;
2062: def : Pat<(int_nvvm_i2d_rz i32:$a), (CVT_f64_s32 $a, CvtRZ)>;
2063: def : Pat<(int_nvvm_i2d_rm i32:$a), (CVT_f64_s32 $a, CvtRM)>;
2064: def : Pat<(int_nvvm_i2d_rp i32:$a), (CVT_f64_s32 $a, CvtRP)>;
2065:
2066: def : Pat<(int_nvvm_ui2d_rn i32:$a), (CVT_f64_u32 $a, CvtRN)>;
2067: def : Pat<(int_nvvm_ui2d_rz i32:$a), (CVT_f64_u32 $a, CvtRZ)>;
2068: def : Pat<(int_nvvm_ui2d_rm i32:$a), (CVT_f64_u32 $a, CvtRM)>;
2069: def : Pat<(int_nvvm_ui2d_rp i32:$a), (CVT_f64_u32 $a, CvtRP)>;
2070:
2071: def : Pat<(int_nvvm_f2i_rn_ftz f32:$a), (CVT_s32_f32 $a, CvtRNI_FTZ)>;
2072: def : Pat<(int_nvvm_f2i_rn f32:$a),     (CVT_s32_f32 $a, CvtRNI)>;
2073: def : Pat<(int_nvvm_f2i_rz_ftz f32:$a), (CVT_s32_f32 $a, CvtRZI_FTZ)>;
2074: def : Pat<(int_nvvm_f2i_rz f32:$a),     (CVT_s32_f32 $a, CvtRZI)>;
2075: def : Pat<(int_nvvm_f2i_rm_ftz f32:$a), (CVT_s32_f32 $a, CvtRMI_FTZ)>;
2076: def : Pat<(int_nvvm_f2i_rm f32:$a),     (CVT_s32_f32 $a, CvtRMI)>;
2077: def : Pat<(int_nvvm_f2i_rp_ftz f32:$a), (CVT_s32_f32 $a, CvtRPI_FTZ)>;
2078: def : Pat<(int_nvvm_f2i_rp f32:$a),     (CVT_s32_f32 $a, CvtRPI)>;
2079:
2080: def : Pat<(int_nvvm_f2ui_rn_ftz f32:$a), (CVT_u32_f32 $a, CvtRNI_FTZ)>;
2081: def : Pat<(int_nvvm_f2ui_rn f32:$a),     (CVT_u32_f32 $a, CvtRNI)>;
2082: def : Pat<(int_nvvm_f2ui_rz_ftz f32:$a), (CVT_u32_f32 $a, CvtRZI_FTZ)>;
2083: def : Pat<(int_nvvm_f2ui_rz f32:$a),     (CVT_u32_f32 $a, CvtRZI)>;
2084: def : Pat<(int_nvvm_f2ui_rm_ftz f32:$a), (CVT_u32_f32 $a, CvtRMI_FTZ)>;
2085: def : Pat<(int_nvvm_f2ui_rm f32:$a),     (CVT_u32_f32 $a, CvtRMI)>;
2086: def : Pat<(int_nvvm_f2ui_rp_ftz f32:$a), (CVT_u32_f32 $a, CvtRPI_FTZ)>;
2087: def : Pat<(int_nvvm_f2ui_rp f32:$a),     (CVT_u32_f32 $a, CvtRPI)>;
2088:
2089: def : Pat<(int_nvvm_i2f_rn i32:$a), (CVT_f32_s32 $a, CvtRN)>;
2090: def : Pat<(int_nvvm_i2f_rz i32:$a), (CVT_f32_s32 $a, CvtRZ)>;
2091: def : Pat<(int_nvvm_i2f_rm i32:$a), (CVT_f32_s32 $a, CvtRM)>;
2092: def : Pat<(int_nvvm_i2f_rp i32:$a), (CVT_f32_s32 $a, CvtRP)>;
2093:
2094: def : Pat<(int_nvvm_ui2f_rn i32:$a), (CVT_f32_u32 $a, CvtRN)>;
2095: def : Pat<(int_nvvm_ui2f_rz i32:$a), (CVT_f32_u32 $a, CvtRZ)>;
2096: def : Pat<(int_nvvm_ui2f_rm i32:$a), (CVT_f32_u32 $a, CvtRM)>;
2097: def : Pat<(int_nvvm_ui2f_rp i32:$a), (CVT_f32_u32 $a, CvtRP)>;
2098:
2099: def : Pat<(int_nvvm_ff2bf16x2_rn f32:$a, f32:$b),      (CVT_bf16x2_f32 $a, $b, CvtRN)>;
2100: def : Pat<(int_nvvm_ff2bf16x2_rn_relu f32:$a, f32:$b), (CVT_bf16x2_f32 $a, $b, CvtRN_RELU)>;
```
- EN: This range uses TableGen DSL to describe records such as INT_NVVM_SUB, INT_NVVM_MIXED_SUB, BFIND_, BFIND_SHIFTAMT_; LLVM later expands these declarations into generated backend tables and helper code.
- CN: 这一段使用 TableGen DSL 描述 INT_NVVM_SUB、INT_NVVM_MIXED_SUB、BFIND_、BFIND_SHIFTAMT_ 等记录；LLVM 后续会把这些声明展开为生成的后端表和辅助代码。

### Lines 2101-2240
```tablegen
2101: def : Pat<(int_nvvm_ff2bf16x2_rz f32:$a, f32:$b),      (CVT_bf16x2_f32 $a, $b, CvtRZ)>;
2102: def : Pat<(int_nvvm_ff2bf16x2_rz_relu f32:$a, f32:$b), (CVT_bf16x2_f32 $a, $b, CvtRZ_RELU)>;
2103: let Predicates = [hasPTX<81>, hasSM<80>] in {
2104:   def : Pat<(int_nvvm_ff2bf16x2_rn_satfinite f32:$a, f32:$b), (CVT_bf16x2_f32_sf $a, $b, CvtRN)>;
2105:   def : Pat<(int_nvvm_ff2bf16x2_rn_relu_satfinite f32:$a, f32:$b), (CVT_bf16x2_f32_sf $a, $b, CvtRN_RELU)>;
2106:   def : Pat<(int_nvvm_ff2bf16x2_rz_satfinite f32:$a, f32:$b), (CVT_bf16x2_f32_sf $a, $b, CvtRZ)>;
2107:   def : Pat<(int_nvvm_ff2bf16x2_rz_relu_satfinite f32:$a, f32:$b), (CVT_bf16x2_f32_sf $a, $b, CvtRZ_RELU)>;
2108: }
2109: let Predicates = [callSubtarget<"hasConvertWithStochasticRounding">] in {
2110: def : Pat<(int_nvvm_ff2bf16x2_rs f32:$a, f32:$b, i32:$c),
2111:           (CVT_bf16x2_f32_rs $a, $b, $c, CvtRS)>;
2112: def : Pat<(int_nvvm_ff2bf16x2_rs_relu f32:$a, f32:$b, i32:$c),
2113:           (CVT_bf16x2_f32_rs $a, $b, $c, CvtRS_RELU)>;
2114: def : Pat<(int_nvvm_ff2bf16x2_rs_satfinite f32:$a, f32:$b, i32:$c), 
2115:           (CVT_bf16x2_f32_rs_sf $a, $b, $c, CvtRS)>;
2116: def : Pat<(int_nvvm_ff2bf16x2_rs_relu_satfinite f32:$a, f32:$b, i32:$c),  
2117:           (CVT_bf16x2_f32_rs_sf $a, $b, $c, CvtRS_RELU)>;
2118: }
2119:
2120: def : Pat<(int_nvvm_ff2f16x2_rn f32:$a, f32:$b),      (CVT_f16x2_f32 $a, $b, CvtRN)>;
2121: def : Pat<(int_nvvm_ff2f16x2_rn_relu f32:$a, f32:$b), (CVT_f16x2_f32 $a, $b, CvtRN_RELU)>;
2122: def : Pat<(int_nvvm_ff2f16x2_rz f32:$a, f32:$b),      (CVT_f16x2_f32 $a, $b, CvtRZ)>;
2123: def : Pat<(int_nvvm_ff2f16x2_rz_relu f32:$a, f32:$b), (CVT_f16x2_f32 $a, $b, CvtRZ_RELU)>;
2124: let Predicates = [hasPTX<81>, hasSM<80>] in {
2125:   def : Pat<(int_nvvm_ff2f16x2_rn_satfinite f32:$a, f32:$b), (CVT_f16x2_f32_sf $a, $b, CvtRN)>;
2126:   def : Pat<(int_nvvm_ff2f16x2_rn_relu_satfinite f32:$a, f32:$b), (CVT_f16x2_f32_sf $a, $b, CvtRN_RELU)>;
2127:   def : Pat<(int_nvvm_ff2f16x2_rz_satfinite f32:$a, f32:$b), (CVT_f16x2_f32_sf $a, $b, CvtRZ)>;
2128:   def : Pat<(int_nvvm_ff2f16x2_rz_relu_satfinite f32:$a, f32:$b), (CVT_f16x2_f32_sf $a, $b, CvtRZ_RELU)>;
2129: }
2130:
2131: let Predicates = [callSubtarget<"hasConvertWithStochasticRounding">] in {
2132: def : Pat<(int_nvvm_ff2f16x2_rs f32:$a, f32:$b, i32:$c),
2133:           (CVT_f16x2_f32_rs $a, $b, $c, CvtRS)>;
2134: def : Pat<(int_nvvm_ff2f16x2_rs_relu f32:$a, f32:$b, i32:$c),
2135:           (CVT_f16x2_f32_rs $a, $b, $c, CvtRS_RELU)>;
2136: def : Pat<(int_nvvm_ff2f16x2_rs_satfinite f32:$a, f32:$b, i32:$c), 
2137:           (CVT_f16x2_f32_rs_sf $a, $b, $c, CvtRS)>;
2138: def : Pat<(int_nvvm_ff2f16x2_rs_relu_satfinite f32:$a, f32:$b, i32:$c), 
2139:           (CVT_f16x2_f32_rs_sf $a, $b, $c, CvtRS_RELU)>;
2140: }
2141: def : Pat<(int_nvvm_f2bf16_rn f32:$a),      (CVT_bf16_f32 $a, CvtRN)>;
2142: def : Pat<(int_nvvm_f2bf16_rn_relu f32:$a), (CVT_bf16_f32 $a, CvtRN_RELU)>;
2143: def : Pat<(int_nvvm_f2bf16_rz f32:$a),      (CVT_bf16_f32 $a, CvtRZ)>;
2144: def : Pat<(int_nvvm_f2bf16_rz_relu f32:$a), (CVT_bf16_f32 $a, CvtRZ_RELU)>;
2145: let Predicates = [hasPTX<81>, hasSM<80>] in {
2146:   def : Pat<(int_nvvm_f2bf16_rz_satfinite f32:$a), (CVT_bf16_f32_sf $a, CvtRZ)>;
2147:   def : Pat<(int_nvvm_f2bf16_rz_relu_satfinite f32:$a), (CVT_bf16_f32_sf $a, CvtRZ_RELU)>;
2148:   def : Pat<(int_nvvm_f2bf16_rn_satfinite f32:$a), (CVT_bf16_f32_sf $a, CvtRN)>;
2149:   def : Pat<(int_nvvm_f2bf16_rn_relu_satfinite f32:$a), (CVT_bf16_f32_sf $a, CvtRN_RELU)>;
2150: }
2151:
2152: def : Pat<(int_nvvm_f2f16_rn f32:$a),      (CVT_f16_f32 $a, CvtRN)>;
2153: def : Pat<(int_nvvm_f2f16_rn_relu f32:$a), (CVT_f16_f32 $a, CvtRN_RELU)>;
2154: def : Pat<(int_nvvm_f2f16_rz f32:$a),      (CVT_f16_f32 $a, CvtRZ)>;
2155: def : Pat<(int_nvvm_f2f16_rz_relu f32:$a), (CVT_f16_f32 $a, CvtRZ_RELU)>;
2156: let Predicates = [hasPTX<81>, hasSM<80>] in {
2157:   def : Pat<(int_nvvm_f2f16_rz_satfinite f32:$a), (CVT_f16_f32_sf $a, CvtRZ)>;
2158:   def : Pat<(int_nvvm_f2f16_rz_relu_satfinite f32:$a), (CVT_f16_f32_sf $a, CvtRZ_RELU)>;
2159:   def : Pat<(int_nvvm_f2f16_rn_satfinite f32:$a), (CVT_f16_f32_sf $a, CvtRN)>;
2160:   def : Pat<(int_nvvm_f2f16_rn_relu_satfinite f32:$a), (CVT_f16_f32_sf $a, CvtRN_RELU)>;
2161: }
2162:
2163: def : Pat<(int_nvvm_lohi_i2d i32:$a, i32:$b), (V2I32toI64 $a, $b)>;
2164: def : Pat<(int_nvvm_d2i_lo f64:$a), (I64toI32L $a)>;
2165: def : Pat<(int_nvvm_d2i_hi f64:$a), (I64toI32H $a)>;
2166:
2167: def : Pat<(int_nvvm_d2i_lo f64:$a), (I64toI32L_Sink $a)>, Requires<[hasPTX<71>]>;
2168: def : Pat<(int_nvvm_d2i_hi f64:$a), (I64toI32H_Sink $a)>, Requires<[hasPTX<71>]>;
2169:
2170: def : Pat<(int_nvvm_f2ll_rn_ftz f32:$a), (CVT_s64_f32 $a, CvtRNI_FTZ)>;
2171: def : Pat<(int_nvvm_f2ll_rn f32:$a),     (CVT_s64_f32 $a, CvtRNI)>;
2172: def : Pat<(int_nvvm_f2ll_rz_ftz f32:$a), (CVT_s64_f32 $a, CvtRZI_FTZ)>;
2173: def : Pat<(int_nvvm_f2ll_rz f32:$a),     (CVT_s64_f32 $a, CvtRZI)>;
2174: def : Pat<(int_nvvm_f2ll_rm_ftz f32:$a), (CVT_s64_f32 $a, CvtRMI_FTZ)>;
2175: def : Pat<(int_nvvm_f2ll_rm f32:$a),     (CVT_s64_f32 $a, CvtRMI)>;
2176: def : Pat<(int_nvvm_f2ll_rp_ftz f32:$a), (CVT_s64_f32 $a, CvtRPI_FTZ)>;
2177: def : Pat<(int_nvvm_f2ll_rp f32:$a),     (CVT_s64_f32 $a, CvtRPI)>;
2178:
2179: def : Pat<(int_nvvm_f2ull_rn_ftz f32:$a), (CVT_u64_f32 $a, CvtRNI_FTZ)>;
2180: def : Pat<(int_nvvm_f2ull_rn f32:$a),     (CVT_u64_f32 $a, CvtRNI)>;
2181: def : Pat<(int_nvvm_f2ull_rz_ftz f32:$a), (CVT_u64_f32 $a, CvtRZI_FTZ)>;
2182: def : Pat<(int_nvvm_f2ull_rz f32:$a),     (CVT_u64_f32 $a, CvtRZI)>;
2183: def : Pat<(int_nvvm_f2ull_rm_ftz f32:$a), (CVT_u64_f32 $a, CvtRMI_FTZ)>;
2184: def : Pat<(int_nvvm_f2ull_rm f32:$a),     (CVT_u64_f32 $a, CvtRMI)>;
2185: def : Pat<(int_nvvm_f2ull_rp_ftz f32:$a), (CVT_u64_f32 $a, CvtRPI_FTZ)>;
2186: def : Pat<(int_nvvm_f2ull_rp f32:$a),     (CVT_u64_f32 $a, CvtRPI)>;
2187:
2188: def : Pat<(int_nvvm_d2ll_rn f64:$a), (CVT_s64_f64 $a, CvtRNI)>;
2189: def : Pat<(int_nvvm_d2ll_rz f64:$a), (CVT_s64_f64 $a, CvtRZI)>;
2190: def : Pat<(int_nvvm_d2ll_rm f64:$a), (CVT_s64_f64 $a, CvtRMI)>;
2191: def : Pat<(int_nvvm_d2ll_rp f64:$a), (CVT_s64_f64 $a, CvtRPI)>;
2192:
2193: def : Pat<(int_nvvm_d2ull_rn f64:$a), (CVT_u64_f64 $a, CvtRNI)>;
2194: def : Pat<(int_nvvm_d2ull_rz f64:$a), (CVT_u64_f64 $a, CvtRZI)>;
2195: def : Pat<(int_nvvm_d2ull_rm f64:$a), (CVT_u64_f64 $a, CvtRMI)>;
2196: def : Pat<(int_nvvm_d2ull_rp f64:$a), (CVT_u64_f64 $a, CvtRPI)>;
2197:
2198: def : Pat<(int_nvvm_ll2f_rn i64:$a), (CVT_f32_s64 $a, CvtRN)>;
2199: def : Pat<(int_nvvm_ll2f_rz i64:$a), (CVT_f32_s64 $a, CvtRZ)>;
2200: def : Pat<(int_nvvm_ll2f_rm i64:$a), (CVT_f32_s64 $a, CvtRM)>;
2201: def : Pat<(int_nvvm_ll2f_rp i64:$a), (CVT_f32_s64 $a, CvtRP)>;
2202:
2203: def : Pat<(int_nvvm_ull2f_rn i64:$a), (CVT_f32_u64 $a, CvtRN)>;
2204: def : Pat<(int_nvvm_ull2f_rz i64:$a), (CVT_f32_u64 $a, CvtRZ)>;
2205: def : Pat<(int_nvvm_ull2f_rm i64:$a), (CVT_f32_u64 $a, CvtRM)>;
2206: def : Pat<(int_nvvm_ull2f_rp i64:$a), (CVT_f32_u64 $a, CvtRP)>;
2207:
2208: def : Pat<(int_nvvm_ll2d_rn i64:$a), (CVT_f64_s64 $a, CvtRN)>;
2209: def : Pat<(int_nvvm_ll2d_rz i64:$a), (CVT_f64_s64 $a, CvtRZ)>;
2210: def : Pat<(int_nvvm_ll2d_rm i64:$a), (CVT_f64_s64 $a, CvtRM)>;
2211: def : Pat<(int_nvvm_ll2d_rp i64:$a), (CVT_f64_s64 $a, CvtRP)>;
2212:
2213: def : Pat<(int_nvvm_ull2d_rn i64:$a), (CVT_f64_u64 $a, CvtRN)>;
2214: def : Pat<(int_nvvm_ull2d_rz i64:$a), (CVT_f64_u64 $a, CvtRZ)>;
2215: def : Pat<(int_nvvm_ull2d_rm i64:$a), (CVT_f64_u64 $a, CvtRM)>;
2216: def : Pat<(int_nvvm_ull2d_rp i64:$a), (CVT_f64_u64 $a, CvtRP)>;
2217:
2218:
2219: def : Pat<(int_nvvm_f2h_rn_ftz f32:$a), (CVT_f16_f32 $a, CvtRN_FTZ)>;
2220: def : Pat<(int_nvvm_f2h_rn f32:$a), (CVT_f16_f32 $a, CvtRN)>;
2221:
2222: let Predicates = [callSubtarget<"hasFP8ConversionSupport">] in {
2223:   def : Pat<(int_nvvm_ff_to_e4m3x2_rn f32:$a, f32:$b),
2224:             (CVT_e4m3x2_f32 $a, $b, CvtRN)>;
2225:   def : Pat<(int_nvvm_ff_to_e4m3x2_rn_relu f32:$a, f32:$b),
2226:             (CVT_e4m3x2_f32 $a, $b, CvtRN_RELU)>;
2227:   def : Pat<(int_nvvm_ff_to_e5m2x2_rn f32:$a, f32:$b),
2228:             (CVT_e5m2x2_f32 $a, $b, CvtRN)>;
2229:   def : Pat<(int_nvvm_ff_to_e5m2x2_rn_relu f32:$a, f32:$b),
2230:             (CVT_e5m2x2_f32 $a, $b, CvtRN_RELU)>;
2231: }
2232:
2233: let Predicates = [callSubtarget<"hasFP16X2ToNarrowFPConversionSupport">] in {
2234:   foreach dst_type = ["e4m3x2", "e5m2x2"] in {
2235:     foreach relu = ["", "_relu"] in {
2236:       defvar intrin = !cast<Intrinsic>("int_nvvm_bf16x2_to_" # dst_type # "_rn" # relu # "_satfinite");
2237:       defvar cvt_inst = !cast<NVPTXInst>("CVT_"# dst_type #"_bf16x2");
2238:       defvar cvt_mode = !cast<PatLeaf>("CvtRN" # !toupper(relu));
2239:       def : Pat<(intrin v2bf16:$a),
2240:                 (cvt_inst $a, cvt_mode)>;
```
- EN: This range uses TableGen DSL to describe records such as backend logic; LLVM later expands these declarations into generated backend tables and helper code.
- CN: 这一段使用 TableGen DSL 描述 后端逻辑 等记录；LLVM 后续会把这些声明展开为生成的后端表和辅助代码。

### Lines 2241-2380
```tablegen
2241:     }
2242:   }
2243: }
2244:
2245: let Predicates = [callSubtarget<"hasFP8ConversionSupport">] in {
2246:   def : Pat<(int_nvvm_f16x2_to_e4m3x2_rn v2f16:$a),
2247:             (CVT_e4m3x2_f16x2 $a, CvtRN)>;
2248:   def : Pat<(int_nvvm_f16x2_to_e4m3x2_rn_relu v2f16:$a),
2249:             (CVT_e4m3x2_f16x2 $a, CvtRN_RELU)>;
2250:   def : Pat<(int_nvvm_f16x2_to_e5m2x2_rn v2f16:$a),
2251:             (CVT_e5m2x2_f16x2 $a, CvtRN)>;
2252:   def : Pat<(int_nvvm_f16x2_to_e5m2x2_rn_relu v2f16:$a),
2253:             (CVT_e5m2x2_f16x2 $a, CvtRN_RELU)>;
2254:
2255:   def : Pat<(int_nvvm_e4m3x2_to_f16x2_rn i16:$a),
2256:             (CVT_f16x2_e4m3x2 $a, CvtRN)>;
2257:   def : Pat<(int_nvvm_e4m3x2_to_f16x2_rn_relu i16:$a),
2258:             (CVT_f16x2_e4m3x2 $a, CvtRN_RELU)>;
2259:   def : Pat<(int_nvvm_e5m2x2_to_f16x2_rn i16:$a),
2260:             (CVT_f16x2_e5m2x2 $a, CvtRN)>;
2261:   def : Pat<(int_nvvm_e5m2x2_to_f16x2_rn_relu i16:$a),
2262:             (CVT_f16x2_e5m2x2 $a, CvtRN_RELU)>;
2263: }
2264:
2265: let Predicates = [callSubtarget<"hasNarrowFPToBF16x2ConversionSupport">] in {
2266:   foreach src_type = ["e4m3x2", "e5m2x2"] in {
2267:     foreach relu = ["", "_relu"] in {
2268:       foreach satfinite = ["", "_satfinite"] in {
2269:         defvar intrin = !cast<Intrinsic>("int_nvvm_" # src_type # "_to_bf16x2_rn" # relu # satfinite # "_scale_n2_ue8m0");
2270:         defvar inst = !cast<NVPTXInst>("CVT_bf16x2_" # src_type # "_scale");
2271:         defvar mode = !cast<PatLeaf>("CvtRN" # !toupper(relu) # !toupper(satfinite));
2272:
2273:         def : Pat<(intrin i16:$a, i16:$b),
2274:                   (inst $a, $b, mode)>;
2275:       }
2276:     }
2277:   }
2278: } // let Predicates = [callSubtarget<"hasNarrowFPToBF16x2ConversionSupport">]
2279:
2280: let Predicates = [callSubtarget<"hasS2F6X2ConversionSupport">] in {
2281:   def : Pat<(int_nvvm_ff_to_s2f6x2_rn_satfinite_scale_n2_ue8m0 f32:$a, f32:$b, i16:$scale),
2282:             (CVT_s2f6x2_f32_sf_scale $a, $b, $scale, CvtRN)>;
2283:   def : Pat<(int_nvvm_ff_to_s2f6x2_rn_relu_satfinite_scale_n2_ue8m0 f32:$a, f32:$b, i16:$scale),
2284:             (CVT_s2f6x2_f32_sf_scale $a, $b, $scale, CvtRN_RELU)>;
2285:
2286:   def : Pat<(int_nvvm_bf16x2_to_s2f6x2_rn_satfinite_scale_n2_ue8m0 v2bf16:$a, i16:$scale),
2287:             (CVT_s2f6x2_bf16x2_sf_scale $a, $scale, CvtRN)>;
2288:   def : Pat<(int_nvvm_bf16x2_to_s2f6x2_rn_relu_satfinite_scale_n2_ue8m0 v2bf16:$a, i16:$scale),
2289:             (CVT_s2f6x2_bf16x2_sf_scale $a, $scale, CvtRN_RELU)>;
2290:
2291:   def : Pat<(int_nvvm_s2f6x2_to_bf16x2_rn_scale_n2_ue8m0 i16:$a, i16:$scale),
2292:             (CVT_bf16x2_s2f6x2_scale $a, $scale, CvtRN)>;
2293:   def : Pat<(int_nvvm_s2f6x2_to_bf16x2_rn_relu_scale_n2_ue8m0 i16:$a, i16:$scale),
2294:             (CVT_bf16x2_s2f6x2_scale $a, $scale, CvtRN_RELU)>;
2295:   
2296:   def : Pat<(int_nvvm_s2f6x2_to_bf16x2_rn_satfinite_scale_n2_ue8m0 i16:$a, i16:$scale),
2297:             (CVT_bf16x2_s2f6x2_sf_scale $a, $scale, CvtRN)>;
2298:   def : Pat<(int_nvvm_s2f6x2_to_bf16x2_rn_relu_satfinite_scale_n2_ue8m0 i16:$a, i16:$scale),
2299:             (CVT_bf16x2_s2f6x2_sf_scale $a, $scale, CvtRN_RELU)>;
2300: }
2301:
2302: let Predicates = [callSubtarget<"hasNarrowFPConversionSupport">] in {
2303:   def : Pat<(int_nvvm_ff_to_e2m3x2_rn_satfinite f32:$a, f32:$b),
2304:             (CVT_e2m3x2_f32_sf $a, $b, CvtRN)>;
2305:   def : Pat<(int_nvvm_ff_to_e2m3x2_rn_relu_satfinite f32:$a, f32:$b),
2306:             (CVT_e2m3x2_f32_sf $a, $b, CvtRN_RELU)>;
2307:   def : Pat<(int_nvvm_ff_to_e3m2x2_rn_satfinite f32:$a, f32:$b),
2308:             (CVT_e3m2x2_f32_sf $a, $b, CvtRN)>;
2309:   def : Pat<(int_nvvm_ff_to_e3m2x2_rn_relu_satfinite f32:$a, f32:$b),
2310:             (CVT_e3m2x2_f32_sf $a, $b, CvtRN_RELU)>;
2311:
2312:   def : Pat<(int_nvvm_e2m3x2_to_f16x2_rn i16:$a),
2313:             (CVT_f16x2_e2m3x2 $a, CvtRN)>;
2314:   def : Pat<(int_nvvm_e2m3x2_to_f16x2_rn_relu i16:$a),
2315:             (CVT_f16x2_e2m3x2 $a, CvtRN_RELU)>;
2316:   def : Pat<(int_nvvm_e3m2x2_to_f16x2_rn i16:$a),
2317:             (CVT_f16x2_e3m2x2 $a, CvtRN)>;
2318:   def : Pat<(int_nvvm_e3m2x2_to_f16x2_rn_relu i16:$a),
2319:             (CVT_f16x2_e3m2x2 $a, CvtRN_RELU)>;
2320: }
2321:
2322: let Predicates = [callSubtarget<"hasFP16X2ToNarrowFPConversionSupport">] in {
2323:   foreach src_type = ["f16x2", "bf16x2"] in {
2324:     foreach dst_type = ["e2m3x2", "e3m2x2"] in {
2325:       foreach relu = ["", "_relu"] in {
2326:         defvar intrin = !cast<Intrinsic>("int_nvvm_" # src_type # "_to_" # dst_type # "_rn" # relu # "_satfinite");
2327:         defvar cvt_inst = !cast<NVPTXInst>("CVT_" # dst_type # "_" # src_type # "_sf");
2328:         defvar cvt_mode = !cast<PatLeaf>("CvtRN" # !toupper(relu));
2329:         def : Pat<(intrin B32:$a), (cvt_inst $a, cvt_mode)>;
2330:       }
2331:     }
2332:   }
2333: }
2334:
2335: let Predicates = [callSubtarget<"hasNarrowFPToBF16x2ConversionSupport">] in {
2336:   foreach src_type = ["e2m3x2", "e3m2x2"] in {
2337:     foreach relu = ["", "_relu"] in {
2338:       foreach satfinite = ["", "_satfinite"] in {
2339:         defvar intrin = !cast<Intrinsic>("int_nvvm_" # src_type # "_to_bf16x2_rn" # relu # satfinite # "_scale_n2_ue8m0");
2340:         defvar inst = !cast<NVPTXInst>("CVT_bf16x2_" # src_type # "_scale");
2341:         defvar mode = !cast<PatLeaf>("CvtRN" # !toupper(relu) # !toupper(satfinite));
2342:         def : Pat<(intrin i16:$a, i16:$b),
2343:                   (inst $a, $b, mode)>;
2344:       }
2345:     }
2346:   }
2347: } // let Predicates = [callSubtarget<"hasNarrowFPToBF16x2ConversionSupport">]
2348:
2349: let Predicates = [callSubtarget<"hasNarrowFPConversionSupport">] in {
2350:   def : Pat<(int_nvvm_ff_to_e2m1x2_rn_satfinite f32:$a, f32:$b),
2351:             (CVT_e2m1x2_f32_sf $a, $b, CvtRN)>;
2352:   def : Pat<(int_nvvm_ff_to_e2m1x2_rn_relu_satfinite f32:$a, f32:$b),
2353:             (CVT_e2m1x2_f32_sf $a, $b, CvtRN_RELU)>;
2354:
2355:   def : Pat<(int_nvvm_e2m1x2_to_f16x2_rn i16:$a),
2356:             (CVT_f16x2_e2m1x2 $a, CvtRN)>;
2357:   def : Pat<(int_nvvm_e2m1x2_to_f16x2_rn_relu i16:$a),
2358:             (CVT_f16x2_e2m1x2 $a, CvtRN_RELU)>;
2359: }
2360:
2361: let Predicates = [callSubtarget<"hasFP16X2ToNarrowFPConversionSupport">] in {
2362:   foreach src_type = ["f16x2", "bf16x2"] in {
2363:     foreach relu = ["", "_relu"] in {
2364:       defvar intrin = !cast<Intrinsic>("int_nvvm_" # src_type # "_to_e2m1x2_rn" # relu # "_satfinite");
2365:       defvar cvt_inst = !cast<NVPTXInst>("CVT_e2m1x2_" # src_type # "_sf");
2366:       defvar cvt_mode = !cast<PatLeaf>("CvtRN" # !toupper(relu));
2367:       def : Pat<(intrin B32:$a), (cvt_inst $a, cvt_mode)>;
2368:     }
2369:   }
2370: }
2371:
2372: let Predicates = [callSubtarget<"hasNarrowFPToBF16x2ConversionSupport">] in {
2373:   foreach relu = ["", "_relu"] in {
2374:     foreach satfinite = ["", "_satfinite"] in {
2375:       defvar intrin = !cast<Intrinsic>("int_nvvm_e2m1x2_to_bf16x2_rn" # relu # satfinite # "_scale_n2_ue8m0");
2376:       defvar inst = !cast<NVPTXInst>("CVT_bf16x2_e2m1x2_scale");
2377:       defvar mode = !cast<PatLeaf>("CvtRN" # !toupper(relu) # !toupper(satfinite));
2378:       def : Pat<(intrin i16:$a, i16:$b),
2379:                 (inst $a, $b, mode)>;
2380:     }
```
- EN: This range uses TableGen DSL to describe records such as backend logic; LLVM later expands these declarations into generated backend tables and helper code.
- CN: 这一段使用 TableGen DSL 描述 后端逻辑 等记录；LLVM 后续会把这些声明展开为生成的后端表和辅助代码。

### Lines 2381-2520
```tablegen
2381:   }
2382: } // let Predicates = [callSubtarget<"hasNarrowFPToBF16x2ConversionSupport">]
2383:
2384: let Predicates = [callSubtarget<"hasNarrowFPConversionSupport">] in {
2385:   def : Pat<(int_nvvm_ff_to_ue8m0x2_rz f32:$a, f32:$b),
2386:             (CVT_ue8m0x2_f32 $a, $b, CvtRZ)>;
2387:   def : Pat<(int_nvvm_ff_to_ue8m0x2_rz_satfinite f32:$a, f32:$b),
2388:             (CVT_ue8m0x2_f32_sf $a, $b, CvtRZ)>;
2389:   def : Pat<(int_nvvm_ff_to_ue8m0x2_rp f32:$a, f32:$b),
2390:             (CVT_ue8m0x2_f32 $a, $b, CvtRP)>;
2391:   def : Pat<(int_nvvm_ff_to_ue8m0x2_rp_satfinite f32:$a, f32:$b),
2392:             (CVT_ue8m0x2_f32_sf $a, $b, CvtRP)>;
2393:
2394:   def : Pat<(int_nvvm_bf16x2_to_ue8m0x2_rz v2bf16:$a),
2395:             (CVT_ue8m0x2_bf16x2 $a, CvtRZ)>;
2396:   def : Pat<(int_nvvm_bf16x2_to_ue8m0x2_rz_satfinite v2bf16:$a),
2397:             (CVT_ue8m0x2_bf16x2_sf $a, CvtRZ)>;
2398:   def : Pat<(int_nvvm_bf16x2_to_ue8m0x2_rp v2bf16:$a),
2399:             (CVT_ue8m0x2_bf16x2 $a, CvtRP)>;
2400:   def : Pat<(int_nvvm_bf16x2_to_ue8m0x2_rp_satfinite v2bf16:$a),
2401:             (CVT_ue8m0x2_bf16x2_sf $a, CvtRP)>;
2402:
2403:   def : Pat<(int_nvvm_ue8m0x2_to_bf16x2 i16:$a),
2404:             (CVT_bf16x2_ue8m0x2 $a)>;
2405: }
2406:
2407: def SDT_CVT_F32X4_TO_FPX4_RS_VEC :
2408:   SDTypeProfile<1, 6, [SDTCisVec<0>, SDTCisFP<1>, SDTCisFP<2>, SDTCisFP<3>, 
2409:                        SDTCisFP<4>, SDTCisInt<5>, SDTCisInt<6>]>;
2410:
2411: def SDT_CVT_F32X4_TO_FPX4_RS_INT :
2412:   SDTypeProfile<1, 6, [SDTCisInt<0>, SDTCisFP<1>, SDTCisFP<2>, SDTCisFP<3>, 
2413:                        SDTCisFP<4>, SDTCisInt<5>, SDTCisInt<6>]>;
2414:
2415: class CVT_F32X4_TO_FPX4_RS_SF_NODE<string FPName, SDTypeProfile SDT> :
2416:   SDNode<"NVPTXISD::CVT_" # FPName # "X4_F32X4_RS_SF", SDT, []>;
2417:   
2418: multiclass CVT_F32X4_TO_FPX4_RS_SF_VEC<string FPName, VTVec RetTy> {
2419:   def : Pat<(RetTy (CVT_F32X4_TO_FPX4_RS_SF_NODE<!toupper(FPName),
2420:                       SDT_CVT_F32X4_TO_FPX4_RS_VEC>
2421:                    f32:$f1, f32:$f2, f32:$f3, f32:$f4, i32:$rbits, CvtRS)),
2422:             (!cast<NVPTXInst>("CVT_" # FPName # "x4_f32x4_rs_sf") 
2423:               $f1, $f2, $f3, $f4, $rbits, CvtRS)>;
2424:   
2425:   def : Pat<(RetTy (CVT_F32X4_TO_FPX4_RS_SF_NODE<!toupper(FPName), 
2426:                       SDT_CVT_F32X4_TO_FPX4_RS_VEC>
2427:                    f32:$f1, f32:$f2, f32:$f3, f32:$f4, i32:$rbits, CvtRS_RELU)),
2428:             (!cast<NVPTXInst>("CVT_" # FPName # "x4_f32x4_rs_sf") 
2429:               $f1, $f2, $f3, $f4, $rbits, CvtRS_RELU)>;
2430: }
2431:
2432: // RS rounding mode conversions
2433: let Predicates = [callSubtarget<"hasConvertWithStochasticRounding">] in {
2434: // FP8x4 conversions
2435: defm : CVT_F32X4_TO_FPX4_RS_SF_VEC<"e4m3", v4i8>;
2436: defm : CVT_F32X4_TO_FPX4_RS_SF_VEC<"e5m2", v4i8>;
2437:
2438: // FP6x4 conversions
2439: defm : CVT_F32X4_TO_FPX4_RS_SF_VEC<"e2m3", v4i8>;
2440: defm : CVT_F32X4_TO_FPX4_RS_SF_VEC<"e3m2", v4i8>;
2441:
2442: // FP4x4 conversions
2443: def : Pat<(i16 (CVT_F32X4_TO_FPX4_RS_SF_NODE<"E2M1", 
2444:                   SDT_CVT_F32X4_TO_FPX4_RS_INT>
2445:                 f32:$f1, f32:$f2, f32:$f3, f32:$f4, i32:$rbits, CvtRS)),
2446:           (CVT_e2m1x4_f32x4_rs_sf $f1, $f2, $f3, $f4, $rbits, CvtRS)>;
2447: def : Pat<(i16 (CVT_F32X4_TO_FPX4_RS_SF_NODE<"E2M1", 
2448:                   SDT_CVT_F32X4_TO_FPX4_RS_INT>
2449:                 f32:$f1, f32:$f2, f32:$f3, f32:$f4, i32:$rbits, CvtRS_RELU)),
2450:           (CVT_e2m1x4_f32x4_rs_sf $f1, $f2, $f3, $f4, $rbits, CvtRS_RELU)>;
2451: }
2452:
2453: //
2454: // FNS
2455: //
2456:
2457: class INT_FNS_MBO<dag ins, dag Operands>
2458:   : BasicNVPTXInst<(outs B32:$dst), ins,
2459:                "fns.b32",
2460:                [(set i32:$dst, Operands)]>,
2461:     Requires<[hasPTX<60>, hasSM<30>]>;
2462:
2463: def INT_FNS_rrr : INT_FNS_MBO<(ins B32:$mask, B32:$base, B32:$offset),
2464:                      (int_nvvm_fns i32:$mask, i32:$base, i32:$offset)>;
2465: def INT_FNS_rri : INT_FNS_MBO<(ins B32:$mask, B32:$base,    i32imm:$offset),
2466:                      (int_nvvm_fns i32:$mask, i32:$base,       imm:$offset)>;
2467: def INT_FNS_rir : INT_FNS_MBO<(ins B32:$mask,    i32imm:$base, B32:$offset),
2468:                      (int_nvvm_fns i32:$mask,       imm:$base, i32:$offset)>;
2469: def INT_FNS_rii : INT_FNS_MBO<(ins B32:$mask,    i32imm:$base,    i32imm:$offset),
2470:                      (int_nvvm_fns i32:$mask,       imm:$base,       imm:$offset)>;
2471: def INT_FNS_irr : INT_FNS_MBO<(ins    i32imm:$mask, B32:$base, B32:$offset),
2472:                      (int_nvvm_fns       imm:$mask, i32:$base, i32:$offset)>;
2473: def INT_FNS_iri : INT_FNS_MBO<(ins    i32imm:$mask, B32:$base,    i32imm:$offset),
2474:                      (int_nvvm_fns       imm:$mask, i32:$base,       imm:$offset)>;
2475: def INT_FNS_iir : INT_FNS_MBO<(ins    i32imm:$mask,    i32imm:$base, B32:$offset),
2476:                      (int_nvvm_fns       imm:$mask,       imm:$base, i32:$offset)>;
2477: def INT_FNS_iii : INT_FNS_MBO<(ins    i32imm:$mask,    i32imm:$base,    i32imm:$offset),
2478:                      (int_nvvm_fns       imm:$mask,       imm:$base,       imm:$offset)>;
2479:
2480: //-----------------------------------
2481: // Atomic Functions
2482: //-----------------------------------
2483:
2484: multiclass F_ATOMIC_2_INTRINSIC<RegTyInfo t, string sem_str, string as_str, string op_str,
2485:                       SDPatternOperator op, list<Predicate> preds> {
2486:   defvar asm_str = "atom" # sem_str # as_str # "." # op_str;
2487:   let mayLoad = 1, mayStore = 1, hasSideEffects = 1 in {
2488:     def r : BasicNVPTXInst<(outs t.RC:$dst), (ins ADDR:$addr, t.RC:$b),
2489:       asm_str,
2490:       [(set t.Ty:$dst, (op addr:$addr, t.Ty:$b))]>,
2491:     Requires<preds>;
2492:     if t.SupportsImm then
2493:       def i : BasicNVPTXInst<(outs t.RC:$dst), (ins ADDR:$addr, t.Imm:$b),
2494:         asm_str,
2495:         [(set t.Ty:$dst, (op addr:$addr, (t.Ty t.ImmNode:$b)))]>,
2496:       Requires<preds>;
2497:   }
2498: }
2499:
2500: class AtomicSDNodeXForm<SDNode atomic> {
2501:   SDNodeXForm GetSem = SDNodeXForm<atomic, [{
2502:     return getI32Imm(getMemOrder(cast<MemSDNode>(N)), SDLoc(N));
2503:   }]>;
2504:
2505:   SDNodeXForm GetScope = SDNodeXForm<atomic, [{
2506:     return getI32Imm(getAtomicScope(cast<MemSDNode>(N)), SDLoc(N));
2507:   }]>;
2508:
2509:   SDNodeXForm GetAddSp = SDNodeXForm<atomic, [{
2510:     return getI32Imm(getAddrSpace(cast<MemSDNode>(N)), SDLoc(N));
2511:   }]>;
2512: }
2513:
2514: multiclass F_ATOMIC_2<RegTyInfo t, SDPatternOperator op, string op_str, SDNode atomic, list<Predicate> preds = []> {
2515:   defvar asm_str = "atom${sem:sem}${scope:scope}${addsp:addsp}" # "." # op_str;
2516:   let mayLoad = 1, mayStore = 1, hasSideEffects = 1 in {
2517:     def _r : BasicFlagsNVPTXInst<(outs t.RC:$dst),
2518:       (ins ADDR:$addr, t.RC:$b),
2519:       (ins AtomicCode:$sem, AtomicCode:$scope, AtomicCode:$addsp),
2520:       asm_str>,
```
- EN: This range uses TableGen DSL to describe records such as SDT_CVT_F32X4_TO_FPX4_RS_VEC, SDT_CVT_F32X4_TO_FPX4_RS_INT, CVT_F32X4_TO_FPX4_RS_SF_NODE, CVT_F32X4_TO_FPX4_RS_SF_VEC; LLVM later expands these declarations into generated backend tables and helper code.
- CN: 这一段使用 TableGen DSL 描述 SDT_CVT_F32X4_TO_FPX4_RS_VEC、SDT_CVT_F32X4_TO_FPX4_RS_INT、CVT_F32X4_TO_FPX4_RS_SF_NODE、CVT_F32X4_TO_FPX4_RS_SF_VEC 等记录；LLVM 后续会把这些声明展开为生成的后端表和辅助代码。

### Lines 2521-2660
```tablegen
2521:       Requires<preds>;
2522:     if t.SupportsImm then
2523:       def _i : BasicFlagsNVPTXInst<(outs t.RC:$dst),
2524:         (ins ADDR:$addr, t.Imm:$b),
2525:         (ins AtomicCode:$sem, AtomicCode:$scope, AtomicCode:$addsp),
2526:         asm_str>,
2527:         Requires<preds>;
2528:   }
2529:
2530:   defvar XForm = AtomicSDNodeXForm<atomic>;
2531:
2532:   def : Pat<(op:$this addr:$addr, t.Ty:$b),
2533:   (!cast<Instruction>(NAME # _r) ADDR:$addr, t.Ty:$b, (XForm.GetSem $this), (XForm.GetScope $this), (XForm.GetAddSp $this))>;
2534:
2535:   if t.SupportsImm then
2536:     def : Pat<(op:$this addr:$addr, (t.Ty t.ImmNode:$b)),
2537:       (!cast<Instruction>(NAME # _i) ADDR:$addr, (t.Ty t.ImmNode:$b), (XForm.GetSem $this), (XForm.GetScope $this), (XForm.GetAddSp $this))>;
2538: }
2539:
2540: multiclass F_ATOMIC_3<RegTyInfo t, string op_str, SDPatternOperator op, SDNode atomic> {
2541:   defvar asm_str = "atom${sem:sem}${scope:scope}${addsp:addsp}" # "." # op_str;
2542:
2543:   let mayLoad = 1, mayStore = 1, hasSideEffects = 1 in {
2544:     def _rr : BasicFlagsNVPTXInst<(outs t.RC:$dst),
2545:       (ins ADDR:$addr, t.RC:$b, t.RC:$c),
2546:       (ins AtomicCode:$sem, AtomicCode:$scope, AtomicCode:$addsp),
2547:       asm_str>;
2548:
2549:     def _ir : BasicFlagsNVPTXInst<(outs t.RC:$dst),
2550:       (ins ADDR:$addr, t.Imm:$b, t.RC:$c),
2551:       (ins AtomicCode:$sem, AtomicCode:$scope, AtomicCode:$addsp),
2552:       asm_str>;
2553:
2554:     def _ri : BasicFlagsNVPTXInst<(outs t.RC:$dst),
2555:       (ins ADDR:$addr, t.RC:$b, t.Imm:$c),
2556:       (ins AtomicCode:$sem, AtomicCode:$scope, AtomicCode:$addsp),
2557:       asm_str>;
2558:
2559:     def _ii : BasicFlagsNVPTXInst<(outs t.RC:$dst),
2560:       (ins ADDR:$addr, t.Imm:$b, t.Imm:$c),
2561:       (ins AtomicCode:$sem, AtomicCode:$scope, AtomicCode:$addsp),
2562:       asm_str>;
2563:   }
2564:
2565:   defvar XForm = AtomicSDNodeXForm<atomic>;
2566:
2567:   def : Pat<(op:$this addr:$addr, t.Ty:$b, t.Ty:$c),
2568:         (!cast<Instruction>(NAME # _rr) ADDR:$addr, t.Ty:$b, t.Ty:$c, (XForm.GetSem $this), (XForm.GetScope $this), (XForm.GetAddSp $this))>;
2569:
2570:   def : Pat<(op:$this addr:$addr, (t.Ty t.ImmNode:$b), t.Ty:$c),
2571:         (!cast<Instruction>(NAME # _ir) ADDR:$addr, (t.Ty t.ImmNode:$b), t.Ty:$c, (XForm.GetSem $this), (XForm.GetScope $this), (XForm.GetAddSp $this))>;
2572:
2573:   def : Pat<(op:$this addr:$addr, t.Ty:$b, (t.Ty t.ImmNode:$c)),
2574:         (!cast<Instruction>(NAME # _ri) ADDR:$addr, t.Ty:$b, (t.Ty t.ImmNode:$c), (XForm.GetSem $this), (XForm.GetScope $this), (XForm.GetAddSp $this))>;
2575:
2576:   def : Pat<(op:$this addr:$addr, (t.Ty t.ImmNode:$b), (t.Ty t.ImmNode:$c)),
2577:         (!cast<Instruction>(NAME # _ii) ADDR:$addr, (t.Ty t.ImmNode:$b), (t.Ty t.ImmNode:$c), (XForm.GetSem $this), (XForm.GetScope $this), (XForm.GetAddSp $this))>;
2578: }
2579:
2580: defm atomic_load_fadd  : binary_atomic_op_fp<atomic_load_fadd>;
2581:
2582: // atom_add
2583: defm INT_PTX_ATOM_ADD_32 : F_ATOMIC_2<I32RT, atomic_load_add_i32, "add.u32", atomic_load_add>;
2584: defm INT_PTX_ATOM_ADD_64 : F_ATOMIC_2<I64RT, atomic_load_add_i64, "add.u64", atomic_load_add>;
2585:
2586: defm INT_PTX_ATOM_ADD_F16  : F_ATOMIC_2<F16RT, atomic_load_fadd, "add.noftz.f16", atomic_load_fadd, [hasSM<70>, hasPTX<63>]>;
2587: defm INT_PTX_ATOM_ADD_BF16 : F_ATOMIC_2<BF16RT, atomic_load_fadd, "add.noftz.bf16", atomic_load_fadd, [hasSM<90>, hasPTX<78>]>;
2588: defm INT_PTX_ATOM_ADD_F32  : F_ATOMIC_2<F32RT, atomic_load_fadd, "add.f32", atomic_load_fadd>;
2589: defm INT_PTX_ATOM_ADD_F64  : F_ATOMIC_2<F64RT, atomic_load_fadd, "add.f64", atomic_load_fadd, [hasAtomAddF64]>;
2590:
2591: // atom_swap
2592: defm INT_PTX_ATOM_SWAP_32 : F_ATOMIC_2<I32RT, atomic_swap_i32, "exch.b32", atomic_swap>;
2593: defm INT_PTX_ATOM_SWAP_64 : F_ATOMIC_2<I64RT, atomic_swap_i64, "exch.b64", atomic_swap>;
2594:
2595: // atom_max
2596: defm INT_PTX_ATOMIC_MAX_32 : F_ATOMIC_2<I32RT, atomic_load_max_i32, "max.s32", atomic_load_max>;
2597: defm INT_PTX_ATOMIC_MAX_64 : F_ATOMIC_2<I64RT, atomic_load_max_i64, "max.s64", atomic_load_max, [hasSM<32>]>;
2598: defm INT_PTX_ATOMIC_UMAX_32 : F_ATOMIC_2<I32RT, atomic_load_umax_i32, "max.u32", atomic_load_umax>;
2599: defm INT_PTX_ATOMIC_UMAX_64 : F_ATOMIC_2<I64RT, atomic_load_umax_i64, "max.u64", atomic_load_umax, [hasSM<32>]>;
2600:
2601: // atom_min
2602: defm INT_PTX_ATOMIC_MIN_32 : F_ATOMIC_2<I32RT, atomic_load_min_i32, "min.s32", atomic_load_min>;
2603: defm INT_PTX_ATOMIC_MIN_64 : F_ATOMIC_2<I64RT, atomic_load_min_i64, "min.s64", atomic_load_min, [hasSM<32>]>;
2604: defm INT_PTX_ATOMIC_UMIN_32 : F_ATOMIC_2<I32RT, atomic_load_umin_i32, "min.u32", atomic_load_umin>;
2605: defm INT_PTX_ATOMIC_UMIN_64 : F_ATOMIC_2<I64RT, atomic_load_umin_i64, "min.u64", atomic_load_umin, [hasSM<32>]>;
2606:
2607: // NOTE: The semantics for atomicrmw fmin (and fmax) upholds LangRef
2608: // requirements.  The LangRef requires the semantics of fmin/fmax to follow
2609: // llvm.minimum/llvm.maximum[1], but allows sNaNs to be treated as qNaNs[2],
2610: // that is the llvm.minnum(sNaN, NUM) -> qNaN case can be handled as (qNaN, NUM)
2611: // -> NUM. NVPTX implements this by lowering to atom.min.f32[3], which has those
2612: // semantics.
2613: // [1] https://llvm.org/docs/LangRef.html#llvm-implementation
2614: // [2] https://llvm.org/docs/LangRef.html#behavior-of-floating-point-nan-values
2615: // [3] https://docs.nvidia.com/cuda/parallel-thread-execution/index.html#half-precision-floating-point-instructions-min
2616:
2617: // atom_inc  atom_dec
2618: defm INT_PTX_ATOM_INC_32 : F_ATOMIC_2<I32RT, atomic_load_uinc_wrap_i32, "inc.u32", atomic_load_uinc_wrap>;
2619: defm INT_PTX_ATOM_DEC_32 : F_ATOMIC_2<I32RT, atomic_load_udec_wrap_i32, "dec.u32", atomic_load_udec_wrap>;
2620:
2621: // atom_and
2622: defm INT_PTX_ATOM_AND_32 : F_ATOMIC_2<I32RT, atomic_load_and_i32, "and.b32", atomic_load_and>;
2623: defm INT_PTX_ATOM_AND_64 : F_ATOMIC_2<I64RT, atomic_load_and_i64, "and.b64", atomic_load_and, [hasSM<32>]>;
2624:
2625: // atom_or
2626: defm INT_PTX_ATOM_OR_32 : F_ATOMIC_2<I32RT, atomic_load_or_i32, "or.b32", atomic_load_or>;
2627: defm INT_PTX_ATOM_OR_64 : F_ATOMIC_2<I64RT, atomic_load_or_i64, "or.b64", atomic_load_or, [hasSM<32>]>;
2628:
2629: // atom_xor
2630: defm INT_PTX_ATOM_XOR_32 : F_ATOMIC_2<I32RT, atomic_load_xor_i32, "xor.b32", atomic_load_xor>;
2631: defm INT_PTX_ATOM_XOR_64 : F_ATOMIC_2<I64RT, atomic_load_xor_i64, "xor.b64", atomic_load_xor, [hasSM<32>]>;
2632:
2633: // Define atom.cas for all combinations of size x addrspace x memory order
2634: // supported in PTX *and* on the hardware.
2635: foreach t = [I16RT, I32RT, I64RT] in {
2636:     defvar atomic_cmp_swap_pat = !cast<PatFrag>("atomic_cmp_swap_i"#t.Size);
2637:     defm INT_PTX_ATOM_CAS_#t.Size
2638:      : F_ATOMIC_3<t, "cas.b"#t.Size, atomic_cmp_swap_pat, atomic_cmp_swap>;
2639: }
2640:
2641: // Support for scoped atomic operations.  Matches
2642: // int_nvvm_atomic_{op}_{space}_{type}_{scope}
2643: // and converts it into the appropriate instruction.
2644: // NOTE: not all possible combinations are implemented
2645: //  'space' is limited to generic as it's the only one needed to support CUDA.
2646: //  'scope' = 'gpu' is default and is handled by regular atomic instructions.
2647:
2648: // Define instruction variants for all addressing modes.
2649:
2650: // Constructs intrinsic name and instruction asm strings.
2651: multiclass ATOM2N_impl<string OpStr, string IntTypeStr, string TypeStr,
2652:                        string ScopeStr, string SpaceStr,
2653:                        RegTyInfo t, list<Predicate> Preds> {
2654:   defm "" : F_ATOMIC_2_INTRINSIC<t,
2655:                        as_str = !if(!eq(ScopeStr, "gpu"), "", "." # ScopeStr),
2656:                        sem_str = !if(!eq(SpaceStr, "gen"), "", "." # SpaceStr),
2657:                        op_str = OpStr # "." # TypeStr,
2658:                        op = !cast<Intrinsic>(
2659:                               "int_nvvm_atomic_" # OpStr
2660:                               # "_" # SpaceStr # "_" # IntTypeStr
```
- EN: This range uses TableGen DSL to describe records such as _i, F_ATOMIC_3, _rr, _ir; LLVM later expands these declarations into generated backend tables and helper code.
- CN: 这一段使用 TableGen DSL 描述 _i、F_ATOMIC_3、_rr、_ir 等记录；LLVM 后续会把这些声明展开为生成的后端表和辅助代码。

### Lines 2661-2800
```tablegen
2661:                               # !if(!empty(ScopeStr), "", "_" # ScopeStr)),
2662:                        preds = Preds>;
2663: }
2664:
2665: // Constructs variants for different scopes of atomic op.
2666: multiclass ATOM2S_impl<string OpStr, string IntTypeStr, string TypeStr,
2667:                        RegTyInfo t, list<Predicate> Preds> {
2668:    // .gpu scope is default and is currently covered by existing
2669:    // atomics w/o explicitly specified scope.
2670:   foreach scope = ["cta", "sys"] in {
2671:     // For now we only need variants for generic space pointers.
2672:     foreach space = ["gen"] in {
2673:       defm _#scope#space : ATOM2N_impl<OpStr, IntTypeStr, TypeStr, scope, space,
2674:                          t, !listconcat(Preds, [hasAtomScope])>;
2675:     }
2676:   }
2677: }
2678:
2679: multiclass F_ATOMIC_3_INTRINSIC_PATTERN<RegTyInfo t, string OpStr, string InstructionName> {
2680:   foreach scope = ["cta", "sys"] in {
2681:     foreach space = ["gen"] in {
2682:       defvar intrinsic = !cast<SDPatternOperator>("int_nvvm_atomic_" # OpStr # "_" # space # "_i_" # scope);
2683:       def : Pat<(t.Ty (intrinsic addr:$addr, t.Ty:$b, t.Ty:$c)),
2684:             (!cast<Instruction>(InstructionName # "_rr") ADDR:$addr, t.Ty:$b, t.Ty:$c, Ordering_not_atomic, !cast<PatLeaf>("Scope_" # scope), !cast<PatLeaf>("AddrSpace_" # space))>;
2685:
2686:       def : Pat<(t.Ty (intrinsic addr:$addr, (t.Ty t.ImmNode:$b), t.Ty:$c)),
2687:             (!cast<Instruction>(InstructionName # "_ir") ADDR:$addr, (t.Ty t.ImmNode:$b), t.Ty:$c, Ordering_not_atomic, !cast<PatLeaf>("Scope_" # scope), !cast<PatLeaf>("AddrSpace_" # space))>;
2688:
2689:       def : Pat<(t.Ty (intrinsic addr:$addr, t.Ty:$b, (t.Ty t.ImmNode:$c))),
2690:             (!cast<Instruction>(InstructionName # "_ri") ADDR:$addr, t.Ty:$b, (t.Ty t.ImmNode:$c), Ordering_not_atomic, !cast<PatLeaf>("Scope_" # scope), !cast<PatLeaf>("AddrSpace_" # space))>;
2691:
2692:       def : Pat<(t.Ty (intrinsic addr:$addr, (t.Ty t.ImmNode:$b), (t.Ty t.ImmNode:$c))),
2693:             (!cast<Instruction>(InstructionName # "_ii") ADDR:$addr, (t.Ty t.ImmNode:$b), (t.Ty t.ImmNode:$c), Ordering_not_atomic, !cast<PatLeaf>("Scope_" # scope), !cast<PatLeaf>("AddrSpace_" # space))>;
2694:     }
2695:   }
2696: }
2697:
2698: // atom.add
2699: multiclass ATOM2_add_impl<string OpStr> {
2700:   defm _s32  : ATOM2S_impl<OpStr, "i", "s32", I32RT, []>;
2701:   defm _u32  : ATOM2S_impl<OpStr, "i", "u32", I32RT, []>;
2702:   defm _u64  : ATOM2S_impl<OpStr, "i", "u64", I64RT, []>;
2703:   defm _bf16 : ATOM2S_impl<OpStr, "f", "bf16", BF16RT, [hasSM<90>, hasPTX<78>]>;
2704:   defm _f16  : ATOM2S_impl<OpStr, "f", "f16", F16RT, []>;
2705:   defm _f32  : ATOM2S_impl<OpStr, "f", "f32", F32RT, []>;
2706:   defm _f64  : ATOM2S_impl<OpStr, "f", "f64", F64RT, []>;
2707: }
2708:
2709: // atom.{and,or,xor}
2710: multiclass ATOM2_bitwise_impl<string OpStr> {
2711:   defm _b32 : ATOM2S_impl<OpStr, "i", "b32", I32RT, []>;
2712:   defm _b64 : ATOM2S_impl<OpStr, "i", "b64", I64RT, [hasAtomBitwise64]>;
2713: }
2714:
2715: // atom.exch
2716: multiclass ATOM2_exch_impl<string OpStr> {
2717:   defm _b32 : ATOM2S_impl<OpStr, "i", "b32", I32RT, []>;
2718:   defm _b64 : ATOM2S_impl<OpStr, "i", "b64", I64RT, []>;
2719: }
2720:
2721: // atom.{min,max}
2722: multiclass ATOM2_minmax_impl<string OpStr> {
2723:   defm _s32 : ATOM2S_impl<OpStr, "i", "s32", I32RT, []>;
2724:   defm _u32 : ATOM2S_impl<OpStr, "i", "u32", I32RT, []>;
2725:   defm _s64 : ATOM2S_impl<OpStr, "i", "s64", I64RT, [hasAtomMinMax64]>;
2726:   defm _u64 : ATOM2S_impl<OpStr, "i", "u64", I64RT, [hasAtomMinMax64]>;
2727: }
2728:
2729: // atom.{inc,dec}
2730: multiclass ATOM2_incdec_impl<string OpStr> {
2731:   defm _u32 : ATOM2S_impl<OpStr, "i", "u32", I32RT, []>;
2732: }
2733:
2734: // atom.cas
2735: multiclass ATOM3_cas_impl<string OpStr> {
2736:   defm _b16 : F_ATOMIC_3_INTRINSIC_PATTERN<I16RT, OpStr, "INT_PTX_ATOM_CAS_16">;
2737:   defm _b32 : F_ATOMIC_3_INTRINSIC_PATTERN<I32RT, OpStr, "INT_PTX_ATOM_CAS_32">;
2738:   defm _b64 : F_ATOMIC_3_INTRINSIC_PATTERN<I64RT, OpStr, "INT_PTX_ATOM_CAS_64">;
2739: }
2740:
2741: defm INT_PTX_SATOM_ADD  : ATOM2_add_impl<"add">;
2742: defm INT_PTX_SATOM_AND  : ATOM2_bitwise_impl<"and">;
2743: defm INT_PTX_SATOM_CAS  : ATOM3_cas_impl<"cas">;
2744: defm INT_PTX_SATOM_DEC  : ATOM2_incdec_impl<"dec">;
2745: defm INT_PTX_SATOM_EXCH : ATOM2_exch_impl<"exch">;
2746: defm INT_PTX_SATOM_INC  : ATOM2_incdec_impl<"inc">;
2747: defm INT_PTX_SATOM_MAX  : ATOM2_minmax_impl<"max">;
2748: defm INT_PTX_SATOM_MIN  : ATOM2_minmax_impl<"min">;
2749: defm INT_PTX_SATOM_OR   : ATOM2_bitwise_impl<"or">;
2750: defm INT_PTX_SATOM_XOR  : ATOM2_bitwise_impl<"xor">;
2751:
2752: // atom.*.b128
2753:
2754: let mayLoad = true, mayStore = true, hasSideEffects = true,
2755:     Predicates = [hasAtomSwap128] in {
2756:   def ATOM_CAS_B128 :
2757:     NVPTXInst<
2758:         (outs B64:$dst0, B64:$dst1),
2759:         (ins ADDR:$addr, B64:$cmp0, B64:$cmp1, B64:$swap0, B64:$swap1,
2760:              AtomicCode:$sem, AtomicCode:$scope, AtomicCode:$addsp),
2761:         "{{\n\t"
2762:         ".reg .b128 cmp, swap, dst;\n\t"
2763:         "mov.b128 cmp, {$cmp0, $cmp1};\n\t"
2764:         "mov.b128 swap, {$swap0, $swap1};\n\t"
2765:         "atom${sem:sem}${scope:scope}${addsp:addsp}.cas.b128 dst, [$addr], cmp, swap;\n\t"
2766:         "mov.b128 {$dst0, $dst1}, dst;\n\t"
2767:         "}}">;
2768:
2769:   def ATOM_EXCH_B128 :
2770:     NVPTXInst<
2771:         (outs B64:$dst0, B64:$dst1),
2772:         (ins ADDR:$addr, B64:$amt0, B64:$amt1,
2773:              AtomicCode:$sem, AtomicCode:$scope, AtomicCode:$addsp),
2774:         "{{\n\t"
2775:         ".reg .b128 amt, dst;\n\t"
2776:         "mov.b128 amt, {$amt0, $amt1};\n\t"
2777:         "atom${sem:sem}${scope:scope}${addsp:addsp}.exch.b128 dst, [$addr], amt;\n\t"
2778:         "mov.b128 {$dst0, $dst1}, dst;\n\t"
2779:         "}}">;
2780: }
2781:
2782:
2783: //-----------------------------------
2784: // Support for ldu on sm_20 or later
2785: //-----------------------------------
2786:
2787: // Don't annotate ldu instructions as mayLoad, as they load from memory that is
2788: // read-only in a kernel.
2789:
2790: // Scalar
2791:
2792: class LDU_G<NVPTXRegClass regclass>
2793:   :  NVPTXInst<(outs regclass:$result), (ins i32imm:$fromWidth, ADDR:$src),
2794:                "ldu.global.b$fromWidth \t$result, [$src];">;
2795:
2796: def LDU_GLOBAL_i16 : LDU_G<B16>;
2797: def LDU_GLOBAL_i32 : LDU_G<B32>;
2798: def LDU_GLOBAL_i64 : LDU_G<B64>;
2799:
2800: // vector
```
- EN: This range uses TableGen DSL to describe records such as ATOM2S_impl, _#scope#space, F_ATOMIC_3_INTRINSIC_PATTERN, ATOM2_add_impl; LLVM later expands these declarations into generated backend tables and helper code.
- CN: 这一段使用 TableGen DSL 描述 ATOM2S_impl、_#scope#space、F_ATOMIC_3_INTRINSIC_PATTERN、ATOM2_add_impl 等记录；LLVM 后续会把这些声明展开为生成的后端表和辅助代码。

### Lines 2801-2940
```tablegen
2801:
2802: // Elementized vector ldu
2803: class VLDU_G_ELE_V2<NVPTXRegClass regclass>
2804:   : NVPTXInst<(outs regclass:$dst1, regclass:$dst2),
2805:               (ins i32imm:$fromWidth, ADDR:$src),
2806:               "ldu.global.v2.b$fromWidth \t{{$dst1, $dst2}}, [$src];">;
2807:
2808:
2809: class VLDU_G_ELE_V4<NVPTXRegClass regclass>
2810:   : NVPTXInst<(outs regclass:$dst1, regclass:$dst2, regclass:$dst3, regclass:$dst4),
2811:               (ins i32imm:$fromWidth, ADDR:$src),
2812:                "ldu.global.v4.b$fromWidth \t{{$dst1, $dst2, $dst3, $dst4}}, [$src];">;
2813:
2814:
2815: def LDU_GLOBAL_v2i16 : VLDU_G_ELE_V2<B16>;
2816: def LDU_GLOBAL_v2i32 : VLDU_G_ELE_V2<B32>;
2817: def LDU_GLOBAL_v2i64 : VLDU_G_ELE_V2<B64>;
2818:
2819: def LDU_GLOBAL_v4i16 : VLDU_G_ELE_V4<B16>;
2820: def LDU_GLOBAL_v4i32 : VLDU_G_ELE_V4<B32>;
2821:
2822:
2823: //-----------------------------------
2824: // Support for ldg on sm_35 or later
2825: //-----------------------------------
2826:
2827: // Don't annotate ld.global.nc as mayLoad, because these loads go through the
2828: // non-coherent texture cache, and therefore the values read must be read-only
2829: // during the lifetime of the kernel.
2830:
2831: class LDG_G<NVPTXRegClass regclass>
2832:   : NVPTXInst<(outs regclass:$result),
2833:               (ins AtomicCode:$Sign, i32imm:$fromWidth,
2834:                    UsedBytesMask:$usedBytes, ADDR:$src),
2835:                "${usedBytes}"
2836:                "ld.global.nc.${Sign:sign}$fromWidth \t$result, [$src];">;
2837:
2838: def LD_GLOBAL_NC_i16 : LDG_G<B16>;
2839: def LD_GLOBAL_NC_i32 : LDG_G<B32>;
2840: def LD_GLOBAL_NC_i64 : LDG_G<B64>;
2841:
2842: // vector
2843:
2844: // Elementized vector ldg
2845: class VLDG_G_ELE_V2<NVPTXRegClass regclass> :
2846:   NVPTXInst<(outs regclass:$dst1, regclass:$dst2),
2847:             (ins AtomicCode:$Sign, i32imm:$fromWidth, UsedBytesMask:$usedBytes,
2848:              ADDR:$src),
2849:             "${usedBytes}"
2850:             "ld.global.nc.v2.${Sign:sign}$fromWidth \t{{$dst1, $dst2}}, [$src];">;
2851:
2852:
2853: class VLDG_G_ELE_V4<NVPTXRegClass regclass> :
2854:   NVPTXInst<(outs regclass:$dst1, regclass:$dst2, regclass:$dst3, regclass:$dst4), 
2855:             (ins AtomicCode:$Sign, i32imm:$fromWidth, UsedBytesMask:$usedBytes,
2856:              ADDR:$src),
2857:             "${usedBytes}"
2858:             "ld.global.nc.v4.${Sign:sign}$fromWidth \t{{$dst1, $dst2, $dst3, $dst4}}, [$src];">;
2859:
2860: class VLDG_G_ELE_V8<NVPTXRegClass regclass> :
2861:   NVPTXInst<(outs regclass:$dst1, regclass:$dst2, regclass:$dst3, regclass:$dst4,
2862:                   regclass:$dst5, regclass:$dst6, regclass:$dst7, regclass:$dst8),
2863:             (ins AtomicCode:$Sign, i32imm:$fromWidth, UsedBytesMask:$usedBytes,
2864:              ADDR:$src),
2865:             "${usedBytes}"
2866:              "ld.global.nc.v8.${Sign:sign}$fromWidth \t{{$dst1, $dst2, $dst3, $dst4, $dst5, $dst6, $dst7, $dst8}}, [$src];">;
2867:
2868: // FIXME: 8-bit LDG should be fixed once LDG/LDU nodes are made into proper loads.
2869: def LD_GLOBAL_NC_v2i16 : VLDG_G_ELE_V2<B16>;
2870: def LD_GLOBAL_NC_v2i32 : VLDG_G_ELE_V2<B32>;
2871: def LD_GLOBAL_NC_v2i64 : VLDG_G_ELE_V2<B64>;
2872:
2873: def LD_GLOBAL_NC_v4i16 : VLDG_G_ELE_V4<B16>;
2874: def LD_GLOBAL_NC_v4i32 : VLDG_G_ELE_V4<B32>;
2875:
2876: def LD_GLOBAL_NC_v4i64 : VLDG_G_ELE_V4<B64>;
2877: def LD_GLOBAL_NC_v8i32 : VLDG_G_ELE_V8<B32>;
2878:
2879: multiclass NG_TO_G<string Str, bit Supports32 = 1, list<Predicate> Preds = []> {
2880:   if Supports32 then
2881:     def "" : BasicNVPTXInst<(outs B32:$result), (ins B32:$src),
2882:              "cvta." # Str # ".u32">, Requires<Preds>;
2883:   
2884:   def _64 : BasicNVPTXInst<(outs B64:$result), (ins B64:$src),
2885:               "cvta." # Str # ".u64">, Requires<Preds>;
2886: }
2887:
2888: multiclass G_TO_NG<string Str, bit Supports32 = 1, list<Predicate> Preds = []> {
2889:   if Supports32 then
2890:     def "" : BasicNVPTXInst<(outs B32:$result), (ins B32:$src),
2891:             "cvta.to." # Str # ".u32">, Requires<Preds>;
2892:   
2893:   def _64 : BasicNVPTXInst<(outs B64:$result), (ins B64:$src),
2894:             "cvta.to." # Str # ".u64">, Requires<Preds>;
2895: }
2896:
2897: foreach space = ["local", "shared", "global", "const", "param"] in {
2898:   defm cvta_#space : NG_TO_G<space>;
2899:   defm cvta_to_#space : G_TO_NG<space>;
2900: }
2901:
2902: defm cvta_shared_cluster : NG_TO_G<"shared::cluster", false, [hasClusters]>;
2903: defm cvta_to_shared_cluster : G_TO_NG<"shared::cluster", false, [hasClusters]>;
2904:
2905:
2906: // nvvm.move intrinsicc
2907: def nvvm_move_i16 : BasicNVPTXInst<(outs B16:$r), (ins B16:$s),
2908:                              "mov.b16",
2909:                              [(set i16:$r,
2910:                                (int_nvvm_move_i16 i16:$s))]>;
2911: def nvvm_move_i32 : BasicNVPTXInst<(outs B32:$r), (ins B32:$s),
2912:                              "mov.b32",
2913:                              [(set i32:$r,
2914:                                (int_nvvm_move_i32 i32:$s))]>;
2915: def nvvm_move_i64 : BasicNVPTXInst<(outs B64:$r), (ins B64:$s),
2916:                              "mov.b64",
2917:                              [(set i64:$r,
2918:                                (int_nvvm_move_i64 i64:$s))]>;
2919: def nvvm_move_float : BasicNVPTXInst<(outs B32:$r), (ins B32:$s),
2920:                              "mov.f32",
2921:                              [(set f32:$r,
2922:                                (int_nvvm_move_float f32:$s))]>;
2923: def nvvm_move_double : BasicNVPTXInst<(outs B64:$r), (ins B64:$s),
2924:                              "mov.f64",
2925:                              [(set f64:$r,
2926:                                (int_nvvm_move_double f64:$s))]>;
2927: def nvvm_move_ptr32 : BasicNVPTXInst<(outs B32:$r), (ins B32:$s),
2928:                              "mov.u32",
2929:                              [(set i32:$r,
2930:                                (int_nvvm_move_ptr i32:$s))]>;
2931: def nvvm_move_ptr64 : BasicNVPTXInst<(outs B64:$r), (ins B64:$s),
2932:                              "mov.u64",
2933:                              [(set i64:$r,
2934:                                (int_nvvm_move_ptr i64:$s))]>;
2935:
2936: // @TODO: Are these actually needed, or will we always just see symbols
2937: // copied to registers first?
2938: /*def nvvm_move_sym32 : NVPTXInst<(outs B32:$r), (ins ADDR_base:$s),
2939:                              "mov.u32 \t$r, $s;",
2940:                              [(set B32:$r,
```
- EN: This range uses TableGen DSL to describe records such as VLDU_G_ELE_V2, VLDU_G_ELE_V4, LDU_GLOBAL_v2i16, LDU_GLOBAL_v2i32; LLVM later expands these declarations into generated backend tables and helper code.
- CN: 这一段使用 TableGen DSL 描述 VLDU_G_ELE_V2、VLDU_G_ELE_V4、LDU_GLOBAL_v2i16、LDU_GLOBAL_v2i32 等记录；LLVM 后续会把这些声明展开为生成的后端表和辅助代码。

### Lines 2941-3080
```tablegen
2941:                              (int_nvvm_move_ptr texternalsym:$s))]>;
2942: def nvvm_move_sym64 : NVPTXInst<(outs B64:$r), (ins ADDR_base:$s),
2943:                              "mov.u64 \t$r, $s;",
2944:                              [(set B64:$r,
2945:                              (int_nvvm_move_ptr texternalsym:$s))]>;*/
2946:
2947: def texsurf_handles
2948:   : BasicNVPTXInst<(outs B64:$result), (ins ADDR_base:$src), "mov.u64">;
2949: def : Pat<(int_nvvm_texsurf_handle_internal globaladdr:$src),
2950:           (texsurf_handles (to_tglobaladdr $src))>;
2951:
2952: //-----------------------------------
2953: // Compiler Error Warn
2954: // - Just ignore them in codegen
2955: //-----------------------------------
2956:
2957: def INT_NVVM_COMPILER_WARN_32 : NVPTXInst<(outs), (ins B32:$a),
2958:                 "// llvm.nvvm.compiler.warn()",
2959:                 [(int_nvvm_compiler_warn i32:$a)]>;
2960: def INT_NVVM_COMPILER_WARN_64 : NVPTXInst<(outs), (ins B64:$a),
2961:                 "// llvm.nvvm.compiler.warn()",
2962:                 [(int_nvvm_compiler_warn i64:$a)]>;
2963: def INT_NVVM_COMPILER_ERROR_32 : NVPTXInst<(outs), (ins B32:$a),
2964:                 "// llvm.nvvm.compiler.error()",
2965:                 [(int_nvvm_compiler_error i32:$a)]>;
2966: def INT_NVVM_COMPILER_ERROR_64 : NVPTXInst<(outs), (ins B64:$a),
2967:                 "// llvm.nvvm.compiler.error()",
2968:                 [(int_nvvm_compiler_error i64:$a)]>;
2969:
2970:
2971: // isspacep
2972:
2973: multiclass ISSPACEP<string suffix, Intrinsic Intr, list<Predicate> Preds = []> {
2974:   def _32: BasicNVPTXInst<(outs B1:$d), (ins B32:$a),
2975:               "isspacep." # suffix,
2976:               [(set i1:$d, (Intr i32:$a))]>,
2977:     Requires<Preds>;
2978:   def _64: BasicNVPTXInst<(outs B1:$d), (ins B64:$a),
2979:               "isspacep." # suffix,
2980:               [(set i1:$d, (Intr i64:$a))]>,
2981:     Requires<Preds>;
2982: }
2983:
2984: defm isspace_const  : ISSPACEP<"const", int_nvvm_isspacep_const, [hasPTX<31>]>;
2985: defm isspace_global : ISSPACEP<"global", int_nvvm_isspacep_global>;
2986: defm isspace_local  : ISSPACEP<"local", int_nvvm_isspacep_local>;
2987: defm isspace_shared : ISSPACEP<"shared", int_nvvm_isspacep_shared>;
2988: defm isspace_shared_cluster : ISSPACEP<"shared::cluster",
2989:                                        int_nvvm_isspacep_shared_cluster,
2990:                                        [hasPTX<78>, hasSM<90>]>;
2991:
2992: // Special register reads
2993: def MOV_SPECIAL : BasicNVPTXInst<(outs B32:$d),
2994:                             (ins SpecialRegs:$r),
2995:                             "mov.b32", []>;
2996:
2997: def : Pat<(int_nvvm_read_ptx_sreg_envreg0), (MOV_SPECIAL ENVREG0)>;
2998: def : Pat<(int_nvvm_read_ptx_sreg_envreg1), (MOV_SPECIAL ENVREG1)>;
2999: def : Pat<(int_nvvm_read_ptx_sreg_envreg2), (MOV_SPECIAL ENVREG2)>;
3000: def : Pat<(int_nvvm_read_ptx_sreg_envreg3), (MOV_SPECIAL ENVREG3)>;
3001: def : Pat<(int_nvvm_read_ptx_sreg_envreg4), (MOV_SPECIAL ENVREG4)>;
3002: def : Pat<(int_nvvm_read_ptx_sreg_envreg5), (MOV_SPECIAL ENVREG5)>;
3003: def : Pat<(int_nvvm_read_ptx_sreg_envreg6), (MOV_SPECIAL ENVREG6)>;
3004: def : Pat<(int_nvvm_read_ptx_sreg_envreg7), (MOV_SPECIAL ENVREG7)>;
3005: def : Pat<(int_nvvm_read_ptx_sreg_envreg8), (MOV_SPECIAL ENVREG8)>;
3006: def : Pat<(int_nvvm_read_ptx_sreg_envreg9), (MOV_SPECIAL ENVREG9)>;
3007: def : Pat<(int_nvvm_read_ptx_sreg_envreg10), (MOV_SPECIAL ENVREG10)>;
3008: def : Pat<(int_nvvm_read_ptx_sreg_envreg11), (MOV_SPECIAL ENVREG11)>;
3009: def : Pat<(int_nvvm_read_ptx_sreg_envreg12), (MOV_SPECIAL ENVREG12)>;
3010: def : Pat<(int_nvvm_read_ptx_sreg_envreg13), (MOV_SPECIAL ENVREG13)>;
3011: def : Pat<(int_nvvm_read_ptx_sreg_envreg14), (MOV_SPECIAL ENVREG14)>;
3012: def : Pat<(int_nvvm_read_ptx_sreg_envreg15), (MOV_SPECIAL ENVREG15)>;
3013: def : Pat<(int_nvvm_read_ptx_sreg_envreg16), (MOV_SPECIAL ENVREG16)>;
3014: def : Pat<(int_nvvm_read_ptx_sreg_envreg17), (MOV_SPECIAL ENVREG17)>;
3015: def : Pat<(int_nvvm_read_ptx_sreg_envreg18), (MOV_SPECIAL ENVREG18)>;
3016: def : Pat<(int_nvvm_read_ptx_sreg_envreg19), (MOV_SPECIAL ENVREG19)>;
3017: def : Pat<(int_nvvm_read_ptx_sreg_envreg20), (MOV_SPECIAL ENVREG20)>;
3018: def : Pat<(int_nvvm_read_ptx_sreg_envreg21), (MOV_SPECIAL ENVREG21)>;
3019: def : Pat<(int_nvvm_read_ptx_sreg_envreg22), (MOV_SPECIAL ENVREG22)>;
3020: def : Pat<(int_nvvm_read_ptx_sreg_envreg23), (MOV_SPECIAL ENVREG23)>;
3021: def : Pat<(int_nvvm_read_ptx_sreg_envreg24), (MOV_SPECIAL ENVREG24)>;
3022: def : Pat<(int_nvvm_read_ptx_sreg_envreg25), (MOV_SPECIAL ENVREG25)>;
3023: def : Pat<(int_nvvm_read_ptx_sreg_envreg26), (MOV_SPECIAL ENVREG26)>;
3024: def : Pat<(int_nvvm_read_ptx_sreg_envreg27), (MOV_SPECIAL ENVREG27)>;
3025: def : Pat<(int_nvvm_read_ptx_sreg_envreg28), (MOV_SPECIAL ENVREG28)>;
3026: def : Pat<(int_nvvm_read_ptx_sreg_envreg29), (MOV_SPECIAL ENVREG29)>;
3027: def : Pat<(int_nvvm_read_ptx_sreg_envreg30), (MOV_SPECIAL ENVREG30)>;
3028: def : Pat<(int_nvvm_read_ptx_sreg_envreg31), (MOV_SPECIAL ENVREG31)>;
3029:
3030: //-----------------------------------
3031: // Texture Intrinsics
3032: //-----------------------------------
3033:
3034: // NOTE: For Fermi support, any new texture/surface/sampler intrinsics must be
3035: // also defined in NVPTXReplaceImageHandles.cpp
3036:
3037: // texmode_independent
3038: let IsTex = true, IsTexModeUnified = false in {
3039: // Texture fetch instructions using handles
3040:
3041: class TEX_1D_base<string inst, dag texsamp, list<dag> pattern = []>
3042:     : NVPTXInst<(outs B32:$r, B32:$g, B32:$b, B32:$a),
3043:                  !con(texsamp, (ins B32:$x)),
3044:                  inst # " \t\\{$r, $g, $b, $a\\}, [$t, $s, \\{$x\\}];",
3045:                  pattern>;
3046:
3047: multiclass TEX_1D<string inst, Intrinsic intr> {
3048:   def _RR : TEX_1D_base<inst, (ins B64:$t, B64:$s),
3049:       [(set B32:$r, B32:$g, B32:$b, B32:$a,
3050:             (intr i64:$t, i64:$s, B32:$x))]>;
3051:   def _RI : TEX_1D_base<inst, (ins B64:$t, i64imm:$s)>;
3052:   def _IR : TEX_1D_base<inst, (ins i64imm:$t, B64:$s)>;
3053:   def _II : TEX_1D_base<inst, (ins i64imm:$t, i64imm:$s)>;
3054: }
3055:
3056: defm TEX_1D_F32_S32 : TEX_1D<"tex.1d.v4.f32.s32", int_nvvm_tex_1d_v4f32_s32>;
3057: defm TEX_1D_F32_F32 : TEX_1D<"tex.1d.v4.f32.f32", int_nvvm_tex_1d_v4f32_f32>;
3058: defm TEX_1D_S32_S32 : TEX_1D<"tex.1d.v4.s32.s32", int_nvvm_tex_1d_v4s32_s32>;
3059: defm TEX_1D_S32_F32 : TEX_1D<"tex.1d.v4.s32.f32", int_nvvm_tex_1d_v4s32_f32>;
3060: defm TEX_1D_U32_S32 : TEX_1D<"tex.1d.v4.u32.s32", int_nvvm_tex_1d_v4u32_s32>;
3061: defm TEX_1D_U32_F32 : TEX_1D<"tex.1d.v4.u32.f32", int_nvvm_tex_1d_v4u32_f32>;
3062:
3063: class TEX_1D_LEVEL_base<string inst, dag texsamp, list<dag> pattern = []>
3064:     : NVPTXInst<(outs B32:$r, B32:$g, B32:$b, B32:$a),
3065:                  !con(texsamp, (ins B32:$x, B32:$lod)),
3066:                  inst # " \t\\{$r, $g, $b, $a\\}, [$t, $s, \\{$x\\}], $lod;",
3067:                  pattern>;
3068:
3069: multiclass TEX_1D_LEVEL<string inst, Intrinsic intr> {
3070:   def _RR : TEX_1D_LEVEL_base<inst, (ins B64:$t, B64:$s),
3071:       [(set B32:$r, B32:$g, B32:$b, B32:$a,
3072:             (intr i64:$t, i64:$s, B32:$x, B32:$lod))]>;
3073:   def _RI : TEX_1D_LEVEL_base<inst, (ins B64:$t, i64imm:$s)>;
3074:   def _IR : TEX_1D_LEVEL_base<inst, (ins i64imm:$t, B64:$s)>;
3075:   def _II : TEX_1D_LEVEL_base<inst, (ins i64imm:$t, i64imm:$s)>;
3076: }
3077:
3078: defm TEX_1D_F32_F32_LEVEL :
3079:   TEX_1D_LEVEL<"tex.level.1d.v4.f32.f32", int_nvvm_tex_1d_level_v4f32_f32>;
3080: defm TEX_1D_S32_F32_LEVEL :
```
- EN: This range uses TableGen DSL to describe records such as nvvm_move_sym64, texsurf_handles, INT_NVVM_COMPILER_WARN_32, INT_NVVM_COMPILER_WARN_64; LLVM later expands these declarations into generated backend tables and helper code.
- CN: 这一段使用 TableGen DSL 描述 nvvm_move_sym64、texsurf_handles、INT_NVVM_COMPILER_WARN_32、INT_NVVM_COMPILER_WARN_64 等记录；LLVM 后续会把这些声明展开为生成的后端表和辅助代码。

### Lines 3081-3220
```tablegen
3081:   TEX_1D_LEVEL<"tex.level.1d.v4.s32.f32", int_nvvm_tex_1d_level_v4s32_f32>;
3082: defm TEX_1D_U32_F32_LEVEL :
3083:   TEX_1D_LEVEL<"tex.level.1d.v4.u32.f32", int_nvvm_tex_1d_level_v4u32_f32>;
3084:
3085: class TEX_1D_GRAD_base<string inst, dag texsamp, list<dag> pattern = []>
3086:     : NVPTXInst<(outs B32:$r, B32:$g, B32:$b, B32:$a),
3087:                  !con(texsamp, (ins B32:$x, B32:$gradx, B32:$grady)),
3088:                  inst # " \t\\{$r, $g, $b, $a\\}, [$t, $s, \\{$x\\}],"
3089:                         " \\{$gradx\\}, \\{$grady\\};",
3090:                  pattern>;
3091:
3092: multiclass TEX_1D_GRAD<string inst, Intrinsic intr> {
3093:   def _RR : TEX_1D_GRAD_base<inst, (ins B64:$t, B64:$s),
3094:       [(set B32:$r, B32:$g, B32:$b, B32:$a,
3095:             (intr i64:$t, i64:$s, B32:$x, B32:$gradx, B32:$grady))]>;
3096:   def _RI : TEX_1D_GRAD_base<inst, (ins B64:$t, i64imm:$s)>;
3097:   def _IR : TEX_1D_GRAD_base<inst, (ins i64imm:$t, B64:$s)>;
3098:   def _II : TEX_1D_GRAD_base<inst, (ins i64imm:$t, i64imm:$s)>;
3099: }
3100:
3101: defm TEX_1D_F32_F32_GRAD
3102:   : TEX_1D_GRAD<"tex.grad.1d.v4.f32.f32", int_nvvm_tex_1d_grad_v4f32_f32>;
3103: defm TEX_1D_S32_F32_GRAD
3104:   : TEX_1D_GRAD<"tex.grad.1d.v4.s32.f32", int_nvvm_tex_1d_grad_v4s32_f32>;
3105: defm TEX_1D_U32_F32_GRAD
3106:   : TEX_1D_GRAD<"tex.grad.1d.v4.u32.f32", int_nvvm_tex_1d_grad_v4u32_f32>;
3107:
3108: class TEX_1D_ARRAY_base<string inst, dag texsamp, list<dag> pattern = []>
3109:     : NVPTXInst<(outs B32:$r, B32:$g, B32:$b, B32:$a),
3110:                  !con(texsamp, (ins B32:$l, B32:$x)),
3111:                  inst # " \t\\{$r, $g, $b, $a\\}, [$t, $s, \\{$l, $x\\}];",
3112:                  pattern>;
3113:
3114: multiclass TEX_1D_ARRAY<string inst, Intrinsic intr> {
3115:   def _RR : TEX_1D_ARRAY_base<inst, (ins B64:$t, B64:$s),
3116:       [(set B32:$r, B32:$g, B32:$b, B32:$a,
3117:             (intr i64:$t, i64:$s, B32:$l, B32:$x))]>;
3118:   def _RI : TEX_1D_ARRAY_base<inst, (ins B64:$t, i64imm:$s)>;
3119:   def _IR : TEX_1D_ARRAY_base<inst, (ins i64imm:$t, B64:$s)>;
3120:   def _II : TEX_1D_ARRAY_base<inst, (ins i64imm:$t, i64imm:$s)>;
3121: }
3122:
3123: defm TEX_1D_ARRAY_F32_F32
3124:   : TEX_1D_ARRAY<"tex.a1d.v4.f32.f32", int_nvvm_tex_1d_array_v4f32_f32>;
3125: defm TEX_1D_ARRAY_F32_S32
3126:   : TEX_1D_ARRAY<"tex.a1d.v4.f32.s32", int_nvvm_tex_1d_array_v4f32_s32>;
3127: defm TEX_1D_ARRAY_S32_S32
3128:   : TEX_1D_ARRAY<"tex.a1d.v4.s32.s32", int_nvvm_tex_1d_array_v4s32_s32>;
3129: defm TEX_1D_ARRAY_S32_F32
3130:   : TEX_1D_ARRAY<"tex.a1d.v4.s32.f32", int_nvvm_tex_1d_array_v4s32_f32>;
3131: defm TEX_1D_ARRAY_U32_S32
3132:   : TEX_1D_ARRAY<"tex.a1d.v4.u32.s32", int_nvvm_tex_1d_array_v4u32_s32>;
3133: defm TEX_1D_ARRAY_U32_F32
3134:   : TEX_1D_ARRAY<"tex.a1d.v4.u32.f32", int_nvvm_tex_1d_array_v4u32_f32>;
3135:
3136: class TEX_1D_ARRAY_LEVEL_base<string inst, dag texsamp, list<dag> pattern = []>
3137:     : NVPTXInst<(outs B32:$r, B32:$g, B32:$b, B32:$a),
3138:                  !con(texsamp, (ins B32:$l, B32:$x, B32:$lod)),
3139:                  inst # " \t\\{$r, $g, $b, $a\\},"
3140:                         " [$t, $s, \\{$l, $x\\}], $lod;",
3141:                  pattern>;
3142:
3143: multiclass TEX_1D_ARRAY_LEVEL<string inst, Intrinsic intr> {
3144:   def _RR : TEX_1D_ARRAY_LEVEL_base<inst, (ins B64:$t, B64:$s),
3145:       [(set B32:$r, B32:$g, B32:$b, B32:$a,
3146:             (intr i64:$t, i64:$s, B32:$l, B32:$x, B32:$lod))]>;
3147:   def _RI : TEX_1D_ARRAY_LEVEL_base<inst, (ins B64:$t, i64imm:$s)>;
3148:   def _IR : TEX_1D_ARRAY_LEVEL_base<inst, (ins i64imm:$t, B64:$s)>;
3149:   def _II : TEX_1D_ARRAY_LEVEL_base<inst, (ins i64imm:$t, i64imm:$s)>;
3150: }
3151:
3152: defm TEX_1D_ARRAY_F32_F32_LEVEL
3153:   : TEX_1D_ARRAY_LEVEL<"tex.level.a1d.v4.f32.f32", int_nvvm_tex_1d_array_level_v4f32_f32>;
3154: defm TEX_1D_ARRAY_S32_F32_LEVEL
3155:   : TEX_1D_ARRAY_LEVEL<"tex.level.a1d.v4.s32.f32", int_nvvm_tex_1d_array_level_v4s32_f32>;
3156: defm TEX_1D_ARRAY_U32_F32_LEVEL
3157:   : TEX_1D_ARRAY_LEVEL<"tex.level.a1d.v4.u32.f32", int_nvvm_tex_1d_array_level_v4u32_f32>;
3158:
3159: class TEX_1D_ARRAY_GRAD_base<string inst, dag texsamp, list<dag> pattern = []>
3160:     : NVPTXInst<(outs B32:$r, B32:$g, B32:$b, B32:$a),
3161:                  !con(texsamp, (ins B32:$l, B32:$x, B32:$gradx, B32:$grady)),
3162:                  inst # " \t\\{$r, $g, $b, $a\\}, [$t, $s, \\{$l, $x\\}],"
3163:                         " \\{$gradx\\}, \\{$grady\\};",
3164:                  pattern>;
3165:
3166: multiclass TEX_1D_ARRAY_GRAD<string inst, Intrinsic intr> {
3167:   def _RR : TEX_1D_ARRAY_GRAD_base<inst, (ins B64:$t, B64:$s),
3168:       [(set B32:$r, B32:$g, B32:$b, B32:$a,
3169:             (intr i64:$t, i64:$s, B32:$l, B32:$x,
3170:                   B32:$gradx, B32:$grady))]>;
3171:   def _RI : TEX_1D_ARRAY_GRAD_base<inst, (ins B64:$t, i64imm:$s)>;
3172:   def _IR : TEX_1D_ARRAY_GRAD_base<inst, (ins i64imm:$t, B64:$s)>;
3173:   def _II : TEX_1D_ARRAY_GRAD_base<inst, (ins i64imm:$t, i64imm:$s)>;
3174: }
3175:
3176: defm TEX_1D_ARRAY_F32_F32_GRAD
3177:   : TEX_1D_ARRAY_GRAD<"tex.grad.a1d.v4.f32.f32", int_nvvm_tex_1d_array_grad_v4f32_f32>;
3178: defm TEX_1D_ARRAY_S32_F32_GRAD
3179:   : TEX_1D_ARRAY_GRAD<"tex.grad.a1d.v4.s32.f32", int_nvvm_tex_1d_array_grad_v4s32_f32>;
3180: defm TEX_1D_ARRAY_U32_F32_GRAD
3181:   : TEX_1D_ARRAY_GRAD<"tex.grad.a1d.v4.u32.f32", int_nvvm_tex_1d_array_grad_v4u32_f32>;
3182:
3183: class TEX_2D_base<string inst, dag texsamp, list<dag> pattern = []>
3184:     : NVPTXInst<(outs B32:$r, B32:$g, B32:$b, B32:$a),
3185:                  !con(texsamp, (ins B32:$x, B32:$y)),
3186:                  inst # " \t\\{$r, $g, $b, $a\\}, [$t, $s, \\{$x, $y\\}];",
3187:                  pattern>;
3188:
3189: multiclass TEX_2D<string inst, Intrinsic intr> {
3190:   def _RR : TEX_2D_base<inst, (ins B64:$t, B64:$s),
3191:       [(set B32:$r, B32:$g, B32:$b, B32:$a,
3192:             (intr i64:$t, i64:$s, B32:$x, B32:$y))]>;
3193:   def _RI : TEX_2D_base<inst, (ins B64:$t, i64imm:$s)>;
3194:   def _IR : TEX_2D_base<inst, (ins i64imm:$t, B64:$s)>;
3195:   def _II : TEX_2D_base<inst, (ins i64imm:$t, i64imm:$s)>;
3196: }
3197:
3198: defm TEX_2D_F32_F32 : TEX_2D<"tex.2d.v4.f32.f32", int_nvvm_tex_2d_v4f32_f32>;
3199: defm TEX_2D_F32_S32 : TEX_2D<"tex.2d.v4.f32.s32", int_nvvm_tex_2d_v4f32_s32>;
3200: defm TEX_2D_S32_S32 : TEX_2D<"tex.2d.v4.s32.s32", int_nvvm_tex_2d_v4s32_s32>;
3201: defm TEX_2D_S32_F32 : TEX_2D<"tex.2d.v4.s32.f32", int_nvvm_tex_2d_v4s32_f32>;
3202: defm TEX_2D_U32_S32 : TEX_2D<"tex.2d.v4.u32.s32", int_nvvm_tex_2d_v4u32_s32>;
3203: defm TEX_2D_U32_F32 : TEX_2D<"tex.2d.v4.u32.f32", int_nvvm_tex_2d_v4u32_f32>;
3204:
3205: class TEX_2D_LEVEL_base<string inst, dag texsamp, list<dag> pattern = []>
3206:     : NVPTXInst<(outs B32:$r, B32:$g, B32:$b, B32:$a),
3207:                  !con(texsamp, (ins B32:$x, B32:$y, B32:$lod)),
3208:                  inst # " \t\\{$r, $g, $b, $a\\},"
3209:                         " [$t, $s, \\{$x, $y\\}], $lod;",
3210:                  pattern>;
3211:
3212: multiclass TEX_2D_LEVEL<string inst, Intrinsic intr> {
3213:   def _RR : TEX_2D_LEVEL_base<inst, (ins B64:$t, B64:$s),
3214:       [(set B32:$r, B32:$g, B32:$b, B32:$a,
3215:             (intr i64:$t, i64:$s, B32:$x, B32:$y, B32:$lod))]>;
3216:   def _RI : TEX_2D_LEVEL_base<inst, (ins B64:$t, i64imm:$s)>;
3217:   def _IR : TEX_2D_LEVEL_base<inst, (ins i64imm:$t, B64:$s)>;
3218:   def _II : TEX_2D_LEVEL_base<inst, (ins i64imm:$t, i64imm:$s)>;
3219: }
3220:
```
- EN: This range uses TableGen DSL to describe records such as TEX_1D_U32_F32_LEVEL, TEX_1D_GRAD_base, TEX_1D_GRAD, _RR; LLVM later expands these declarations into generated backend tables and helper code.
- CN: 这一段使用 TableGen DSL 描述 TEX_1D_U32_F32_LEVEL、TEX_1D_GRAD_base、TEX_1D_GRAD、_RR 等记录；LLVM 后续会把这些声明展开为生成的后端表和辅助代码。

### Lines 3221-3360
```tablegen
3221: defm TEX_2D_F32_F32_LEVEL :
3222:   TEX_2D_LEVEL<"tex.level.2d.v4.f32.f32", int_nvvm_tex_2d_level_v4f32_f32>;
3223: defm TEX_2D_S32_F32_LEVEL :
3224:   TEX_2D_LEVEL<"tex.level.2d.v4.s32.f32", int_nvvm_tex_2d_level_v4s32_f32>;
3225: defm TEX_2D_U32_F32_LEVEL :
3226:   TEX_2D_LEVEL<"tex.level.2d.v4.u32.f32", int_nvvm_tex_2d_level_v4u32_f32>;
3227:
3228: class TEX_2D_GRAD_base<string inst, dag texsamp, list<dag> pattern = []>
3229:     : NVPTXInst<(outs B32:$r, B32:$g, B32:$b, B32:$a),
3230:                  !con(texsamp, (ins B32:$x, B32:$y,
3231:                                     B32:$gradx0, B32:$gradx1,
3232:                                     B32:$grady0, B32:$grady1)),
3233:                  inst # " \t\\{$r, $g, $b, $a\\}, [$t, $s, \\{$x, $y\\}],"
3234:                         " \\{$gradx0, $gradx1\\}, \\{$grady0, $grady1\\};",
3235:                  pattern>;
3236:
3237: multiclass TEX_2D_GRAD<string inst, Intrinsic intr> {
3238:   def _RR : TEX_2D_GRAD_base<inst, (ins B64:$t, B64:$s),
3239:       [(set B32:$r, B32:$g, B32:$b, B32:$a,
3240:             (intr i64:$t, i64:$s, B32:$x, B32:$y,
3241:                   B32:$gradx0, B32:$gradx1,
3242:                   B32:$grady0, B32:$grady1))]>;
3243:   def _RI : TEX_2D_GRAD_base<inst, (ins B64:$t, i64imm:$s)>;
3244:   def _IR : TEX_2D_GRAD_base<inst, (ins i64imm:$t, B64:$s)>;
3245:   def _II : TEX_2D_GRAD_base<inst, (ins i64imm:$t, i64imm:$s)>;
3246: }
3247:
3248: defm TEX_2D_F32_F32_GRAD :
3249:   TEX_2D_GRAD<"tex.grad.2d.v4.f32.f32", int_nvvm_tex_2d_grad_v4f32_f32>;
3250: defm TEX_2D_S32_F32_GRAD :
3251:   TEX_2D_GRAD<"tex.grad.2d.v4.s32.f32", int_nvvm_tex_2d_grad_v4s32_f32>;
3252: defm TEX_2D_U32_F32_GRAD :
3253:   TEX_2D_GRAD<"tex.grad.2d.v4.u32.f32", int_nvvm_tex_2d_grad_v4u32_f32>;
3254:
3255: class TEX_2D_ARRAY_base<string inst, dag texsamp, list<dag> pattern = []>
3256:     : NVPTXInst<(outs B32:$r, B32:$g, B32:$b, B32:$a),
3257:                  !con(texsamp, (ins B32:$l, B32:$x, B32:$y)),
3258:                  inst # " \t\\{$r, $g, $b, $a\\},"
3259:                         " [$t, $s, \\{$l, $x, $y, $y\\}];",
3260:                  pattern>;
3261:
3262: multiclass TEX_2D_ARRAY<string inst, Intrinsic intr> {
3263:   def _RR : TEX_2D_ARRAY_base<inst, (ins B64:$t, B64:$s),
3264:       [(set B32:$r, B32:$g, B32:$b, B32:$a,
3265:             (intr i64:$t, i64:$s, B32:$l, B32:$x, B32:$y))]>;
3266:   def _RI : TEX_2D_ARRAY_base<inst, (ins B64:$t, i64imm:$s)>;
3267:   def _IR : TEX_2D_ARRAY_base<inst, (ins i64imm:$t, B64:$s)>;
3268:   def _II : TEX_2D_ARRAY_base<inst, (ins i64imm:$t, i64imm:$s)>;
3269: }
3270:
3271: defm TEX_2D_ARRAY_F32_F32
3272:   : TEX_2D_ARRAY<"tex.a2d.v4.f32.f32", int_nvvm_tex_2d_array_v4f32_f32>;
3273: defm TEX_2D_ARRAY_F32_S32
3274:   : TEX_2D_ARRAY<"tex.a2d.v4.f32.s32", int_nvvm_tex_2d_array_v4f32_s32>;
3275: defm TEX_2D_ARRAY_S32_S32
3276:   : TEX_2D_ARRAY<"tex.a2d.v4.s32.s32", int_nvvm_tex_2d_array_v4s32_s32>;
3277: defm TEX_2D_ARRAY_S32_F32
3278:   : TEX_2D_ARRAY<"tex.a2d.v4.s32.f32", int_nvvm_tex_2d_array_v4s32_f32>;
3279: defm TEX_2D_ARRAY_U32_S32
3280:   : TEX_2D_ARRAY<"tex.a2d.v4.u32.s32", int_nvvm_tex_2d_array_v4u32_s32>;
3281: defm TEX_2D_ARRAY_U32_F32
3282:   : TEX_2D_ARRAY<"tex.a2d.v4.u32.f32", int_nvvm_tex_2d_array_v4u32_f32>;
3283:
3284: class TEX_2D_ARRAY_LEVEL_base<string inst, dag texsamp, list<dag> pattern = []>
3285:     : NVPTXInst<(outs B32:$r, B32:$g, B32:$b, B32:$a),
3286:                  !con(texsamp, (ins B32:$l, B32:$x, B32:$y, B32:$lod)),
3287:                  inst # " \t\\{$r, $g, $b, $a\\},"
3288:                         " [$t, $s, \\{$l, $x, $y, $y\\}], $lod;",
3289:                  pattern>;
3290:
3291: multiclass TEX_2D_ARRAY_LEVEL<string inst, Intrinsic intr> {
3292:   def _RR : TEX_2D_ARRAY_LEVEL_base<inst, (ins B64:$t, B64:$s),
3293:       [(set B32:$r, B32:$g, B32:$b, B32:$a,
3294:             (intr i64:$t, i64:$s, B32:$l, B32:$x, B32:$y, B32:$lod))]>;
3295:   def _RI : TEX_2D_ARRAY_LEVEL_base<inst, (ins B64:$t, i64imm:$s)>;
3296:   def _IR : TEX_2D_ARRAY_LEVEL_base<inst, (ins i64imm:$t, B64:$s)>;
3297:   def _II : TEX_2D_ARRAY_LEVEL_base<inst, (ins i64imm:$t, i64imm:$s)>;
3298: }
3299:
3300: defm TEX_2D_ARRAY_F32_F32_LEVEL
3301:   : TEX_2D_ARRAY_LEVEL<"tex.level.a2d.v4.f32.f32", int_nvvm_tex_2d_array_level_v4f32_f32>;
3302: defm TEX_2D_ARRAY_S32_F32_LEVEL
3303:   : TEX_2D_ARRAY_LEVEL<"tex.level.a2d.v4.s32.f32", int_nvvm_tex_2d_array_level_v4s32_f32>;
3304: defm TEX_2D_ARRAY_U32_F32_LEVEL
3305:   : TEX_2D_ARRAY_LEVEL<"tex.level.a2d.v4.u32.f32", int_nvvm_tex_2d_array_level_v4u32_f32>;
3306:
3307: class TEX_2D_ARRAY_GRAD_base<string inst, dag texsamp, list<dag> pattern = []>
3308:     : NVPTXInst<(outs B32:$r, B32:$g, B32:$b, B32:$a),
3309:                  !con(texsamp, (ins B32:$l, B32:$x, B32:$y,
3310:                                     B32:$gradx0, B32:$gradx1,
3311:                                     B32:$grady0, B32:$grady1)),
3312:                  inst # " \t\\{$r, $g, $b, $a\\},"
3313:                         " [$t, $s, \\{$l, $x, $y, $y\\}],"
3314:                         " \\{$gradx0, $gradx1\\}, \\{$grady0, $grady1\\};",
3315:                  pattern>;
3316:
3317: multiclass TEX_2D_ARRAY_GRAD<string inst, Intrinsic intr> {
3318:   def _RR : TEX_2D_ARRAY_GRAD_base<inst, (ins B64:$t, B64:$s),
3319:       [(set B32:$r, B32:$g, B32:$b, B32:$a,
3320:             (intr i64:$t, i64:$s, B32:$l, B32:$x, B32:$y,
3321:                   B32:$gradx0, B32:$gradx1,
3322:                   B32:$grady0, B32:$grady1))]>;
3323:   def _RI : TEX_2D_ARRAY_GRAD_base<inst, (ins B64:$t, i64imm:$s)>;
3324:   def _IR : TEX_2D_ARRAY_GRAD_base<inst, (ins i64imm:$t, B64:$s)>;
3325:   def _II : TEX_2D_ARRAY_GRAD_base<inst, (ins i64imm:$t, i64imm:$s)>;
3326: }
3327:
3328: defm TEX_2D_ARRAY_F32_F32_GRAD
3329:   : TEX_2D_ARRAY_GRAD<"tex.grad.a2d.v4.f32.f32", int_nvvm_tex_2d_array_grad_v4f32_f32>;
3330: defm TEX_2D_ARRAY_S32_F32_GRAD
3331:   : TEX_2D_ARRAY_GRAD<"tex.grad.a2d.v4.s32.f32", int_nvvm_tex_2d_array_grad_v4s32_f32>;
3332: defm TEX_2D_ARRAY_U32_F32_GRAD
3333:   : TEX_2D_ARRAY_GRAD<"tex.grad.a2d.v4.u32.f32", int_nvvm_tex_2d_array_grad_v4u32_f32>;
3334:
3335: class TEX_3D_base<string inst, dag texsamp, list<dag> pattern = []>
3336:     : NVPTXInst<(outs B32:$r, B32:$g, B32:$b, B32:$a),
3337:                  !con(texsamp, (ins B32:$x, B32:$y, B32:$z)),
3338:                  inst # " \t\\{$r, $g, $b, $a\\},"
3339:                         " [$t, $s, \\{$x, $y, $z, $z\\}];",
3340:                  pattern>;
3341:
3342: multiclass TEX_3D<string inst, Intrinsic intr> {
3343:   def _RR : TEX_3D_base<inst, (ins B64:$t, B64:$s),
3344:       [(set B32:$r, B32:$g, B32:$b, B32:$a,
3345:             (intr i64:$t, i64:$s, B32:$x, B32:$y, B32:$z))]>;
3346:   def _RI : TEX_3D_base<inst, (ins B64:$t, i64imm:$s)>;
3347:   def _IR : TEX_3D_base<inst, (ins i64imm:$t, B64:$s)>;
3348:   def _II : TEX_3D_base<inst, (ins i64imm:$t, i64imm:$s)>;
3349: }
3350:
3351: defm TEX_3D_F32_F32 : TEX_3D<"tex.3d.v4.f32.f32", int_nvvm_tex_3d_v4f32_f32>;
3352: defm TEX_3D_F32_S32 : TEX_3D<"tex.3d.v4.f32.s32", int_nvvm_tex_3d_v4f32_s32>;
3353: defm TEX_3D_S32_S32 : TEX_3D<"tex.3d.v4.s32.s32", int_nvvm_tex_3d_v4s32_s32>;
3354: defm TEX_3D_S32_F32 : TEX_3D<"tex.3d.v4.s32.f32", int_nvvm_tex_3d_v4s32_f32>;
3355: defm TEX_3D_U32_S32 : TEX_3D<"tex.3d.v4.u32.s32", int_nvvm_tex_3d_v4u32_s32>;
3356: defm TEX_3D_U32_F32 : TEX_3D<"tex.3d.v4.u32.f32", int_nvvm_tex_3d_v4u32_f32>;
3357:
3358: class TEX_3D_LEVEL_base<string inst, dag texsamp, list<dag> pattern = []>
3359:     : NVPTXInst<(outs B32:$r, B32:$g, B32:$b, B32:$a),
3360:                  !con(texsamp, (ins B32:$x, B32:$y, B32:$z, B32:$lod)),
```
- EN: This range uses TableGen DSL to describe records such as TEX_2D_F32_F32_LEVEL, TEX_2D_S32_F32_LEVEL, TEX_2D_U32_F32_LEVEL, TEX_2D_GRAD_base; LLVM later expands these declarations into generated backend tables and helper code.
- CN: 这一段使用 TableGen DSL 描述 TEX_2D_F32_F32_LEVEL、TEX_2D_S32_F32_LEVEL、TEX_2D_U32_F32_LEVEL、TEX_2D_GRAD_base 等记录；LLVM 后续会把这些声明展开为生成的后端表和辅助代码。

### Lines 3361-3500
```tablegen
3361:                  inst # " \t\\{$r, $g, $b, $a\\},"
3362:                         " [$t, $s, \\{$x, $y, $z, $z\\}], $lod;",
3363:                  pattern>;
3364:
3365: multiclass TEX_3D_LEVEL<string inst, Intrinsic intr> {
3366:   def _RR : TEX_3D_LEVEL_base<inst, (ins B64:$t, B64:$s),
3367:       [(set B32:$r, B32:$g, B32:$b, B32:$a,
3368:             (intr i64:$t, i64:$s, B32:$x, B32:$y, B32:$z, B32:$lod))]>;
3369:   def _RI : TEX_3D_LEVEL_base<inst, (ins B64:$t, i64imm:$s)>;
3370:   def _IR : TEX_3D_LEVEL_base<inst, (ins i64imm:$t, B64:$s)>;
3371:   def _II : TEX_3D_LEVEL_base<inst, (ins i64imm:$t, i64imm:$s)>;
3372: }
3373:
3374: defm TEX_3D_F32_F32_LEVEL
3375:   : TEX_3D_LEVEL<"tex.level.3d.v4.f32.f32", int_nvvm_tex_3d_level_v4f32_f32>;
3376: defm TEX_3D_S32_F32_LEVEL
3377:   : TEX_3D_LEVEL<"tex.level.3d.v4.s32.f32", int_nvvm_tex_3d_level_v4s32_f32>;
3378: defm TEX_3D_U32_F32_LEVEL
3379:   : TEX_3D_LEVEL<"tex.level.3d.v4.u32.f32", int_nvvm_tex_3d_level_v4u32_f32>;
3380:
3381: class TEX_3D_GRAD_base<string inst, dag texsamp, list<dag> pattern = []>
3382:     : NVPTXInst<(outs B32:$r, B32:$g, B32:$b, B32:$a),
3383:                  !con(texsamp, (ins B32:$x, B32:$y, B32:$z,
3384:                                     B32:$gradx0, B32:$gradx1,
3385:                                     B32:$gradx2, B32:$grady0,
3386:                                     B32:$grady1, B32:$grady2)),
3387:                  inst # " \t\\{$r, $g, $b, $a\\},"
3388:                         " [$t, $s, \\{$x, $y, $z, $z\\}],"
3389:                         " \\{$gradx0, $gradx1, $gradx2, $gradx2\\},"
3390:                         " \\{$grady0, $grady1, $grady2, $grady2\\};",
3391:                  pattern>;
3392:
3393: multiclass TEX_3D_GRAD<string inst, Intrinsic intr> {
3394:   def _RR : TEX_3D_GRAD_base<inst, (ins B64:$t, B64:$s),
3395:       [(set B32:$r, B32:$g, B32:$b, B32:$a,
3396:             (intr i64:$t, i64:$s, B32:$x, B32:$y, B32:$z,
3397:                   B32:$gradx0, B32:$gradx1, B32:$gradx2,
3398:                   B32:$grady0, B32:$grady1, B32:$grady2))]>;
3399:   def _RI : TEX_3D_GRAD_base<inst, (ins B64:$t, i64imm:$s)>;
3400:   def _IR : TEX_3D_GRAD_base<inst, (ins i64imm:$t, B64:$s)>;
3401:   def _II : TEX_3D_GRAD_base<inst, (ins i64imm:$t, i64imm:$s)>;
3402: }
3403:
3404: defm TEX_3D_F32_F32_GRAD
3405:   : TEX_3D_GRAD<"tex.grad.3d.v4.f32.f32", int_nvvm_tex_3d_grad_v4f32_f32>;
3406: defm TEX_3D_S32_F32_GRAD
3407:   : TEX_3D_GRAD<"tex.grad.3d.v4.s32.f32", int_nvvm_tex_3d_grad_v4s32_f32>;
3408: defm TEX_3D_U32_F32_GRAD
3409:   : TEX_3D_GRAD<"tex.grad.3d.v4.u32.f32", int_nvvm_tex_3d_grad_v4u32_f32>;
3410:
3411: class TEX_CUBE_base<string inst, dag texsamp, list<dag> pattern = []>
3412:     : NVPTXInst<(outs B32:$r, B32:$g, B32:$b, B32:$a),
3413:                  !con(texsamp, (ins B32:$x, B32:$y, B32:$z)),
3414:                  inst # " \t\\{$r, $g, $b, $a\\},"
3415:                         " [$t, $s, \\{$x, $y, $z, $z\\}];",
3416:                  pattern>;
3417:
3418: multiclass TEX_CUBE<string inst, Intrinsic intr> {
3419:   def _RR : TEX_CUBE_base<inst, (ins B64:$t, B64:$s),
3420:       [(set B32:$r, B32:$g, B32:$b, B32:$a,
3421:             (intr i64:$t, i64:$s, B32:$x, B32:$y, B32:$z))]>;
3422:   def _RI : TEX_CUBE_base<inst, (ins B64:$t, i64imm:$s)>;
3423:   def _IR : TEX_CUBE_base<inst, (ins i64imm:$t, B64:$s)>;
3424:   def _II : TEX_CUBE_base<inst, (ins i64imm:$t, i64imm:$s)>;
3425: }
3426:
3427: defm TEX_CUBE_F32_F32
3428:   : TEX_CUBE<"tex.cube.v4.f32.f32", int_nvvm_tex_cube_v4f32_f32>;
3429: defm TEX_CUBE_S32_F32
3430:   : TEX_CUBE<"tex.cube.v4.s32.f32", int_nvvm_tex_cube_v4s32_f32>;
3431: defm TEX_CUBE_U32_F32
3432:   : TEX_CUBE<"tex.cube.v4.u32.f32", int_nvvm_tex_cube_v4u32_f32>;
3433:
3434: class TEX_CUBE_LEVEL_base<string inst, dag texsamp, list<dag> pattern = []>
3435:     : NVPTXInst<(outs B32:$r, B32:$g, B32:$b, B32:$a),
3436:                  !con(texsamp, (ins B32:$x, B32:$y, B32:$z, B32:$lod)),
3437:                  inst # " \t\\{$r, $g, $b, $a\\},"
3438:                         " [$t, $s, \\{$x, $y, $z, $z\\}], $lod;",
3439:                  pattern>;
3440:
3441: multiclass TEX_CUBE_LEVEL<string inst, Intrinsic intr> {
3442:   def _RR : TEX_CUBE_LEVEL_base<inst, (ins B64:$t, B64:$s),
3443:       [(set B32:$r, B32:$g, B32:$b, B32:$a,
3444:             (intr i64:$t, i64:$s, B32:$x, B32:$y, B32:$z,
3445:                   B32:$lod))]>;
3446:   def _RI : TEX_CUBE_LEVEL_base<inst, (ins B64:$t, i64imm:$s)>;
3447:   def _IR : TEX_CUBE_LEVEL_base<inst, (ins i64imm:$t, B64:$s)>;
3448:   def _II : TEX_CUBE_LEVEL_base<inst, (ins i64imm:$t, i64imm:$s)>;
3449: }
3450:
3451: defm TEX_CUBE_F32_F32_LEVEL
3452:   : TEX_CUBE_LEVEL<"tex.level.cube.v4.f32.f32", int_nvvm_tex_cube_level_v4f32_f32>;
3453: defm TEX_CUBE_S32_F32_LEVEL
3454:   : TEX_CUBE_LEVEL<"tex.level.cube.v4.s32.f32", int_nvvm_tex_cube_level_v4s32_f32>;
3455: defm TEX_CUBE_U32_F32_LEVEL
3456:   : TEX_CUBE_LEVEL<"tex.level.cube.v4.u32.f32", int_nvvm_tex_cube_level_v4u32_f32>;
3457:
3458: class TEX_CUBE_ARRAY_base<string inst, dag texsamp, list<dag> pattern = []>
3459:     : NVPTXInst<(outs B32:$r, B32:$g, B32:$b, B32:$a),
3460:                  !con(texsamp, (ins B32:$l, B32:$x, B32:$y, B32:$z)),
3461:                  inst # " \t\\{$r, $g, $b, $a\\},"
3462:                         " [$t, $s, \\{$l, $x, $y, $z\\}];",
3463:                  pattern>;
3464:
3465: multiclass TEX_CUBE_ARRAY<string inst, Intrinsic intr> {
3466:   def _RR : TEX_CUBE_ARRAY_base<inst, (ins B64:$t, B64:$s),
3467:       [(set B32:$r, B32:$g, B32:$b, B32:$a,
3468:             (intr i64:$t, i64:$s, B32:$l, B32:$x, B32:$y, B32:$z))]>;
3469:   def _RI : TEX_CUBE_ARRAY_base<inst, (ins B64:$t, i64imm:$s)>;
3470:   def _IR : TEX_CUBE_ARRAY_base<inst, (ins i64imm:$t, B64:$s)>;
3471:   def _II : TEX_CUBE_ARRAY_base<inst, (ins i64imm:$t, i64imm:$s)>;
3472: }
3473:
3474: defm TEX_CUBE_ARRAY_F32_F32
3475:   : TEX_CUBE_ARRAY<"tex.acube.v4.f32.f32", int_nvvm_tex_cube_array_v4f32_f32>;
3476: defm TEX_CUBE_ARRAY_S32_F32
3477:   : TEX_CUBE_ARRAY<"tex.acube.v4.s32.f32", int_nvvm_tex_cube_array_v4s32_f32>;
3478: defm TEX_CUBE_ARRAY_U32_F32
3479:   : TEX_CUBE_ARRAY<"tex.acube.v4.u32.f32", int_nvvm_tex_cube_array_v4u32_f32>;
3480:
3481: class TEX_CUBE_ARRAY_LEVEL_base<string inst, dag texsamp, list<dag> pattern = []>
3482:     : NVPTXInst<(outs B32:$r, B32:$g, B32:$b, B32:$a),
3483:                  !con(texsamp, (ins B32:$l, B32:$x, B32:$y, B32:$z, B32:$lod)),
3484:                  inst # " \t\\{$r, $g, $b, $a\\},"
3485:                         " [$t, $s, \\{$l, $x, $y, $z\\}], $lod;",
3486:                  pattern>;
3487:
3488: multiclass TEX_CUBE_ARRAY_LEVEL<string inst, Intrinsic intr> {
3489:   def _RR : TEX_CUBE_ARRAY_LEVEL_base<inst, (ins B64:$t, B64:$s),
3490:       [(set B32:$r, B32:$g, B32:$b, B32:$a,
3491:             (intr i64:$t, i64:$s, B32:$l, B32:$x, B32:$y, B32:$z,
3492:                   B32:$lod))]>;
3493:   def _RI : TEX_CUBE_ARRAY_LEVEL_base<inst, (ins B64:$t, i64imm:$s)>;
3494:   def _IR : TEX_CUBE_ARRAY_LEVEL_base<inst, (ins i64imm:$t, B64:$s)>;
3495:   def _II : TEX_CUBE_ARRAY_LEVEL_base<inst, (ins i64imm:$t, i64imm:$s)>;
3496: }
3497:
3498: defm TEX_CUBE_ARRAY_F32_F32_LEVEL
3499:   : TEX_CUBE_ARRAY_LEVEL<"tex.level.acube.v4.f32.f32",
3500:                          int_nvvm_tex_cube_array_level_v4f32_f32>;
```
- EN: This range uses TableGen DSL to describe records such as TEX_3D_LEVEL, _RR, _RI, _IR; LLVM later expands these declarations into generated backend tables and helper code.
- CN: 这一段使用 TableGen DSL 描述 TEX_3D_LEVEL、_RR、_RI、_IR 等记录；LLVM 后续会把这些声明展开为生成的后端表和辅助代码。

### Lines 3501-3640
```tablegen
3501: defm TEX_CUBE_ARRAY_S32_F32_LEVEL
3502:   : TEX_CUBE_ARRAY_LEVEL<"tex.level.acube.v4.s32.f32",
3503:                          int_nvvm_tex_cube_array_level_v4s32_f32>;
3504: defm TEX_CUBE_ARRAY_U32_F32_LEVEL
3505:   : TEX_CUBE_ARRAY_LEVEL<"tex.level.acube.v4.u32.f32",
3506:                          int_nvvm_tex_cube_array_level_v4u32_f32>;
3507:
3508: class TLD4_2D_base<string inst, dag texsamp, list<dag> pattern = []>
3509:     : NVPTXInst<(outs B32:$v0, B32:$v1, B32:$v2, B32:$v3),
3510:                  !con(texsamp, (ins B32:$x, B32:$y)),
3511:                  inst # " \t\\{$v0, $v1, $v2, $v3\\}, [$t, $s, \\{$x, $y\\}];",
3512:                  pattern>;
3513:
3514: multiclass TLD4_2D<string inst, Intrinsic intr> {
3515:   def _RR : TLD4_2D_base<inst, (ins B64:$t, B64:$s),
3516:       [(set B32:$v0, B32:$v1, B32:$v2, B32:$v3,
3517:             (intr i64:$t, i64:$s, B32:$x, B32:$y))]>;
3518:   def _RI : TLD4_2D_base<inst, (ins B64:$t, i64imm:$s)>;
3519:   def _IR : TLD4_2D_base<inst, (ins i64imm:$t, B64:$s)>;
3520:   def _II : TLD4_2D_base<inst, (ins i64imm:$t, i64imm:$s)>;
3521: }
3522:
3523: defm TLD4_R_2D_F32_F32
3524:   : TLD4_2D<"tld4.r.2d.v4.f32.f32", int_nvvm_tld4_r_2d_v4f32_f32>;
3525: defm TLD4_G_2D_F32_F32
3526:   : TLD4_2D<"tld4.g.2d.v4.f32.f32", int_nvvm_tld4_g_2d_v4f32_f32>;
3527: defm TLD4_B_2D_F32_F32
3528:   : TLD4_2D<"tld4.b.2d.v4.f32.f32", int_nvvm_tld4_b_2d_v4f32_f32>;
3529: defm TLD4_A_2D_F32_F32
3530:   : TLD4_2D<"tld4.a.2d.v4.f32.f32", int_nvvm_tld4_a_2d_v4f32_f32>;
3531:
3532: defm TLD4_R_2D_S32_F32
3533:   : TLD4_2D<"tld4.r.2d.v4.s32.f32", int_nvvm_tld4_r_2d_v4s32_f32>;
3534: defm TLD4_G_2D_S32_F32
3535:   : TLD4_2D<"tld4.g.2d.v4.s32.f32", int_nvvm_tld4_g_2d_v4s32_f32>;
3536: defm TLD4_B_2D_S32_F32
3537:   : TLD4_2D<"tld4.b.2d.v4.s32.f32", int_nvvm_tld4_b_2d_v4s32_f32>;
3538: defm TLD4_A_2D_S32_F32
3539:   : TLD4_2D<"tld4.a.2d.v4.s32.f32", int_nvvm_tld4_a_2d_v4s32_f32>;
3540:
3541: defm TLD4_R_2D_U32_F32
3542:   : TLD4_2D<"tld4.r.2d.v4.u32.f32", int_nvvm_tld4_r_2d_v4u32_f32>;
3543: defm TLD4_G_2D_U32_F32
3544:   : TLD4_2D<"tld4.g.2d.v4.u32.f32", int_nvvm_tld4_g_2d_v4u32_f32>;
3545: defm TLD4_B_2D_U32_F32
3546:   : TLD4_2D<"tld4.b.2d.v4.u32.f32", int_nvvm_tld4_b_2d_v4u32_f32>;
3547: defm TLD4_A_2D_U32_F32
3548:   : TLD4_2D<"tld4.a.2d.v4.u32.f32", int_nvvm_tld4_a_2d_v4u32_f32>;
3549:
3550: }
3551:
3552:
3553: // texmode_unified
3554: let IsTex = true, IsTexModeUnified = true in {
3555: // Texture fetch instructions using handles
3556:
3557: class TEX_UNIFIED_1D_base<string inst, dag tex, list<dag> pattern = []>
3558:     : NVPTXInst<(outs B32:$r, B32:$g, B32:$b, B32:$a),
3559:                  !con(tex, (ins B32:$x)),
3560:                  inst # " \t\\{$r, $g, $b, $a\\}, [$t, \\{$x\\}];",
3561:                  pattern>;
3562:
3563: multiclass TEX_UNIFIED_1D<string inst, Intrinsic intr> {
3564:   def _R : TEX_UNIFIED_1D_base<inst, (ins B64:$t),
3565:       [(set B32:$r, B32:$g, B32:$b, B32:$a, (intr i64:$t, B32:$x))]>;
3566:   def _I : TEX_UNIFIED_1D_base<inst, (ins i64imm:$t)>;
3567: }
3568:
3569: defm TEX_UNIFIED_1D_F32_S32
3570:   : TEX_UNIFIED_1D<"tex.1d.v4.f32.s32", int_nvvm_tex_unified_1d_v4f32_s32>;
3571: defm TEX_UNIFIED_1D_F32_F32
3572:   : TEX_UNIFIED_1D<"tex.1d.v4.f32.f32", int_nvvm_tex_unified_1d_v4f32_f32>;
3573: defm TEX_UNIFIED_1D_S32_S32
3574:   : TEX_UNIFIED_1D<"tex.1d.v4.s32.s32", int_nvvm_tex_unified_1d_v4s32_s32>;
3575: defm TEX_UNIFIED_1D_S32_F32
3576:   : TEX_UNIFIED_1D<"tex.1d.v4.s32.f32", int_nvvm_tex_unified_1d_v4s32_f32>;
3577: defm TEX_UNIFIED_1D_U32_S32
3578:   : TEX_UNIFIED_1D<"tex.1d.v4.u32.s32", int_nvvm_tex_unified_1d_v4u32_s32>;
3579: defm TEX_UNIFIED_1D_U32_F32
3580:   : TEX_UNIFIED_1D<"tex.1d.v4.u32.f32", int_nvvm_tex_unified_1d_v4u32_f32>;
3581:
3582: class TEX_UNIFIED_1D_LEVEL_base<string inst, dag tex, list<dag> pattern = []>
3583:     : NVPTXInst<(outs B32:$r, B32:$g, B32:$b, B32:$a),
3584:                  !con(tex, (ins B32:$x, B32:$lod)),
3585:                  inst # " \t\\{$r, $g, $b, $a\\}, [$t, \\{$x\\}], $lod;",
3586:                  pattern>;
3587:
3588: multiclass TEX_UNIFIED_1D_LEVEL<string inst, Intrinsic intr> {
3589:   def _R : TEX_UNIFIED_1D_LEVEL_base<inst, (ins B64:$t),
3590:       [(set B32:$r, B32:$g, B32:$b, B32:$a,
3591:             (intr i64:$t, B32:$x, B32:$lod))]>;
3592:   def _I : TEX_UNIFIED_1D_LEVEL_base<inst, (ins i64imm:$t)>;
3593: }
3594:
3595: defm TEX_UNIFIED_1D_F32_F32_LEVEL
3596:   : TEX_UNIFIED_1D_LEVEL<"tex.level.1d.v4.f32.f32", int_nvvm_tex_unified_1d_level_v4f32_f32>;
3597: defm TEX_UNIFIED_1D_S32_F32_LEVEL
3598:   : TEX_UNIFIED_1D_LEVEL<"tex.level.1d.v4.s32.f32", int_nvvm_tex_unified_1d_level_v4s32_f32>;
3599: defm TEX_UNIFIED_1D_U32_F32_LEVEL
3600:   : TEX_UNIFIED_1D_LEVEL<"tex.level.1d.v4.u32.f32", int_nvvm_tex_unified_1d_level_v4u32_f32>;
3601:
3602: class TEX_UNIFIED_1D_GRAD_base<string inst, dag tex, list<dag> pattern = []>
3603:     : NVPTXInst<(outs B32:$r, B32:$g, B32:$b, B32:$a),
3604:                  !con(tex, (ins B32:$x, B32:$gradx, B32:$grady)),
3605:                  inst # " \t\\{$r, $g, $b, $a\\},"
3606:                         " [$t, \\{$x\\}], \\{$gradx\\}, \\{$grady\\};",
3607:                  pattern>;
3608:
3609: multiclass TEX_UNIFIED_1D_GRAD<string inst, Intrinsic intr> {
3610:   def _R : TEX_UNIFIED_1D_GRAD_base<inst, (ins B64:$t),
3611:       [(set B32:$r, B32:$g, B32:$b, B32:$a,
3612:             (intr i64:$t, B32:$x, B32:$gradx, B32:$grady))]>;
3613:   def _I : TEX_UNIFIED_1D_GRAD_base<inst, (ins i64imm:$t)>;
3614: }
3615:
3616: defm TEX_UNIFIED_1D_F32_F32_GRAD
3617:   : TEX_UNIFIED_1D_GRAD<"tex.grad.1d.v4.f32.f32", int_nvvm_tex_unified_1d_grad_v4f32_f32>;
3618: defm TEX_UNIFIED_1D_S32_F32_GRAD
3619:   : TEX_UNIFIED_1D_GRAD<"tex.grad.1d.v4.s32.f32", int_nvvm_tex_unified_1d_grad_v4s32_f32>;
3620: defm TEX_UNIFIED_1D_U32_F32_GRAD
3621:   : TEX_UNIFIED_1D_GRAD<"tex.grad.1d.v4.u32.f32", int_nvvm_tex_unified_1d_grad_v4u32_f32>;
3622:
3623: class TEX_UNIFIED_1D_ARRAY_base<string inst, dag tex, list<dag> pattern = []>
3624:     : NVPTXInst<(outs B32:$r, B32:$g, B32:$b, B32:$a),
3625:                  !con(tex, (ins B32:$l, B32:$x)),
3626:                  inst # " \t\\{$r, $g, $b, $a\\}, [$t, \\{$l, $x\\}];",
3627:                  pattern>;
3628:
3629: multiclass TEX_UNIFIED_1D_ARRAY<string inst, Intrinsic intr> {
3630:   def _R : TEX_UNIFIED_1D_ARRAY_base<inst, (ins B64:$t),
3631:       [(set B32:$r, B32:$g, B32:$b, B32:$a, (intr i64:$t, B32:$l, B32:$x))]>;
3632:   def _I : TEX_UNIFIED_1D_ARRAY_base<inst, (ins i64imm:$t)>;
3633: }
3634:
3635: defm TEX_UNIFIED_1D_ARRAY_F32_S32
3636:   : TEX_UNIFIED_1D_ARRAY<"tex.a1d.v4.f32.s32", int_nvvm_tex_unified_1d_array_v4f32_s32>;
3637: defm TEX_UNIFIED_1D_ARRAY_F32_F32
3638:   : TEX_UNIFIED_1D_ARRAY<"tex.a1d.v4.f32.f32", int_nvvm_tex_unified_1d_array_v4f32_f32>;
3639: defm TEX_UNIFIED_1D_ARRAY_S32_S32
3640:   : TEX_UNIFIED_1D_ARRAY<"tex.a1d.v4.s32.s32", int_nvvm_tex_unified_1d_array_v4s32_s32>;
```
- EN: This range uses TableGen DSL to describe records such as TEX_CUBE_ARRAY_S32_F32_LEVEL, TEX_CUBE_ARRAY_U32_F32_LEVEL, TLD4_2D_base, TLD4_2D; LLVM later expands these declarations into generated backend tables and helper code.
- CN: 这一段使用 TableGen DSL 描述 TEX_CUBE_ARRAY_S32_F32_LEVEL、TEX_CUBE_ARRAY_U32_F32_LEVEL、TLD4_2D_base、TLD4_2D 等记录；LLVM 后续会把这些声明展开为生成的后端表和辅助代码。

### Lines 3641-3780
```tablegen
3641: defm TEX_UNIFIED_1D_ARRAY_S32_F32
3642:   : TEX_UNIFIED_1D_ARRAY<"tex.a1d.v4.s32.f32", int_nvvm_tex_unified_1d_array_v4s32_f32>;
3643: defm TEX_UNIFIED_1D_ARRAY_U32_S32
3644:   : TEX_UNIFIED_1D_ARRAY<"tex.a1d.v4.u32.s32", int_nvvm_tex_unified_1d_array_v4u32_s32>;
3645: defm TEX_UNIFIED_1D_ARRAY_U32_F32
3646:   : TEX_UNIFIED_1D_ARRAY<"tex.a1d.v4.u32.f32", int_nvvm_tex_unified_1d_array_v4u32_f32>;
3647:
3648: class TEX_UNIFIED_1D_ARRAY_LEVEL_base<string inst, dag tex, list<dag> pattern = []>
3649:     : NVPTXInst<(outs B32:$r, B32:$g, B32:$b, B32:$a),
3650:                  !con(tex, (ins B32:$l, B32:$x, B32:$lod)),
3651:                  inst # " \t\\{$r, $g, $b, $a\\}, [$t, \\{$l, $x\\}], $lod;",
3652:                  pattern>;
3653:
3654: multiclass TEX_UNIFIED_1D_ARRAY_LEVEL<string inst, Intrinsic intr> {
3655:   def _R : TEX_UNIFIED_1D_ARRAY_LEVEL_base<inst, (ins B64:$t),
3656:       [(set B32:$r, B32:$g, B32:$b, B32:$a,
3657:             (intr i64:$t, B32:$l, B32:$x, B32:$lod))]>;
3658:   def _I : TEX_UNIFIED_1D_ARRAY_LEVEL_base<inst, (ins i64imm:$t)>;
3659: }
3660:
3661: defm TEX_UNIFIED_1D_ARRAY_F32_F32_LEVEL
3662:   : TEX_UNIFIED_1D_ARRAY_LEVEL<"tex.level.a1d.v4.f32.f32",
3663:                                int_nvvm_tex_unified_1d_array_level_v4f32_f32>;
3664: defm TEX_UNIFIED_1D_ARRAY_S32_F32_LEVEL
3665:   : TEX_UNIFIED_1D_ARRAY_LEVEL<"tex.level.a1d.v4.s32.f32",
3666:                                int_nvvm_tex_unified_1d_array_level_v4s32_f32>;
3667: defm TEX_UNIFIED_1D_ARRAY_U32_F32_LEVEL
3668:   : TEX_UNIFIED_1D_ARRAY_LEVEL<"tex.level.a1d.v4.u32.f32",
3669:                                int_nvvm_tex_unified_1d_array_level_v4u32_f32>;
3670:
3671: class TEX_UNIFIED_1D_ARRAY_GRAD_base<string inst, dag tex, list<dag> pattern = []>
3672:     : NVPTXInst<(outs B32:$r, B32:$g, B32:$b, B32:$a),
3673:                  !con(tex, (ins B32:$l, B32:$x, B32:$gradx, B32:$grady)),
3674:                  inst # " \t\\{$r, $g, $b, $a\\},"
3675:                         "  [$t, \\{$l, $x\\}], \\{$gradx\\}, \\{$grady\\};",
3676:                  pattern>;
3677:
3678: multiclass TEX_UNIFIED_1D_ARRAY_GRAD<string inst, Intrinsic intr> {
3679:   def _R : TEX_UNIFIED_1D_ARRAY_GRAD_base<inst, (ins B64:$t),
3680:       [(set B32:$r, B32:$g, B32:$b, B32:$a,
3681:             (intr i64:$t, B32:$l, B32:$x, B32:$gradx, B32:$grady))]>;
3682:   def _I : TEX_UNIFIED_1D_ARRAY_GRAD_base<inst, (ins i64imm:$t)>;
3683: }
3684:
3685: defm TEX_UNIFIED_1D_ARRAY_F32_F32_GRAD
3686:   : TEX_UNIFIED_1D_ARRAY_GRAD<"tex.grad.a1d.v4.f32.f32",
3687:                               int_nvvm_tex_unified_1d_array_grad_v4f32_f32>;
3688: defm TEX_UNIFIED_1D_ARRAY_S32_F32_GRAD
3689:   : TEX_UNIFIED_1D_ARRAY_GRAD<"tex.grad.a1d.v4.s32.f32",
3690:                               int_nvvm_tex_unified_1d_array_grad_v4s32_f32>;
3691: defm TEX_UNIFIED_1D_ARRAY_U32_F32_GRAD
3692:   : TEX_UNIFIED_1D_ARRAY_GRAD<"tex.grad.a1d.v4.u32.f32",
3693:                               int_nvvm_tex_unified_1d_array_grad_v4u32_f32>;
3694:
3695: class TEX_UNIFIED_2D_base<string inst, dag tex, list<dag> pattern = []>
3696:     : NVPTXInst<(outs B32:$r, B32:$g, B32:$b, B32:$a),
3697:                  !con(tex, (ins B32:$x, B32:$y)),
3698:                  inst # " \t\\{$r, $g, $b, $a\\}, [$t, \\{$x, $y\\}];",
3699:                  pattern>;
3700:
3701: multiclass TEX_UNIFIED_2D<string inst, Intrinsic intr> {
3702:   def _R : TEX_UNIFIED_2D_base<inst, (ins B64:$t),
3703:       [(set B32:$r, B32:$g, B32:$b, B32:$a,
3704:             (intr i64:$t, B32:$x, B32:$y))]>;
3705:   def _I : TEX_UNIFIED_2D_base<inst, (ins i64imm:$t)>;
3706: }
3707:
3708: defm TEX_UNIFIED_2D_F32_S32
3709:   : TEX_UNIFIED_2D<"tex.2d.v4.f32.s32", int_nvvm_tex_unified_2d_v4f32_s32>;
3710: defm TEX_UNIFIED_2D_F32_F32
3711:   : TEX_UNIFIED_2D<"tex.2d.v4.f32.f32", int_nvvm_tex_unified_2d_v4f32_f32>;
3712: defm TEX_UNIFIED_2D_S32_S32
3713:   : TEX_UNIFIED_2D<"tex.2d.v4.s32.s32", int_nvvm_tex_unified_2d_v4s32_s32>;
3714: defm TEX_UNIFIED_2D_S32_F32
3715:   : TEX_UNIFIED_2D<"tex.2d.v4.s32.f32", int_nvvm_tex_unified_2d_v4s32_f32>;
3716: defm TEX_UNIFIED_2D_U32_S32
3717:   : TEX_UNIFIED_2D<"tex.2d.v4.u32.s32", int_nvvm_tex_unified_2d_v4u32_s32>;
3718: defm TEX_UNIFIED_2D_U32_F32
3719:   : TEX_UNIFIED_2D<"tex.2d.v4.u32.f32", int_nvvm_tex_unified_2d_v4u32_f32>;
3720:
3721: class TEX_UNIFIED_2D_LEVEL_base<string inst, dag tex, list<dag> pattern = []>
3722:     : NVPTXInst<(outs B32:$r, B32:$g, B32:$b, B32:$a),
3723:                  !con(tex, (ins B32:$x, B32:$y, B32:$lod)),
3724:                  inst # " \t\\{$r, $g, $b, $a\\}, [$t, \\{$x, $y\\}], $lod;",
3725:                  pattern>;
3726:
3727: multiclass TEX_UNIFIED_2D_LEVEL<string inst, Intrinsic intr> {
3728:   def _R : TEX_UNIFIED_2D_LEVEL_base<inst, (ins B64:$t),
3729:       [(set B32:$r, B32:$g, B32:$b, B32:$a,
3730:             (intr i64:$t, B32:$x, B32:$y, B32:$lod))]>;
3731:   def _I : TEX_UNIFIED_2D_LEVEL_base<inst, (ins i64imm:$t)>;
3732: }
3733:
3734: defm TEX_UNIFIED_2D_F32_F32_LEVEL
3735:   : TEX_UNIFIED_2D_LEVEL<"tex.level.2d.v4.f32.f32", int_nvvm_tex_unified_2d_level_v4f32_f32>;
3736: defm TEX_UNIFIED_2D_S32_F32_LEVEL
3737:   : TEX_UNIFIED_2D_LEVEL<"tex.level.2d.v4.s32.f32", int_nvvm_tex_unified_2d_level_v4s32_f32>;
3738: defm TEX_UNIFIED_2D_U32_F32_LEVEL
3739:   : TEX_UNIFIED_2D_LEVEL<"tex.level.2d.v4.u32.f32", int_nvvm_tex_unified_2d_level_v4u32_f32>;
3740:
3741: class TEX_UNIFIED_2D_GRAD_base<string inst, dag tex, list<dag> pattern = []>
3742:     : NVPTXInst<(outs B32:$r, B32:$g, B32:$b, B32:$a),
3743:                  !con(tex, (ins B32:$x, B32:$y,
3744:                                 B32:$gradx0, B32:$gradx1,
3745:                                 B32:$grady0, B32:$grady1)),
3746:                  inst # " \t\\{$r, $g, $b, $a\\}, [$t, \\{$x, $y\\}],"
3747:                         " \\{$gradx0, $gradx1\\}, \\{$grady0, $grady1\\};",
3748:                  pattern>;
3749: multiclass TEX_UNIFIED_2D_GRAD<string inst, Intrinsic intr> {
3750:   def _R : TEX_UNIFIED_2D_GRAD_base<inst, (ins B64:$t),
3751:       [(set B32:$r, B32:$g, B32:$b, B32:$a,
3752:             (intr i64:$t, B32:$x, B32:$y,
3753:                   B32:$gradx0, B32:$gradx1,
3754:                   B32:$grady0, B32:$grady1))]>;
3755:   def _I : TEX_UNIFIED_2D_GRAD_base<inst, (ins i64imm:$t)>;
3756: }
3757:
3758: defm TEX_UNIFIED_2D_F32_F32_GRAD
3759:   : TEX_UNIFIED_2D_GRAD<"tex.grad.2d.v4.f32.f32", int_nvvm_tex_unified_2d_grad_v4f32_f32>;
3760: defm TEX_UNIFIED_2D_S32_F32_GRAD
3761:   : TEX_UNIFIED_2D_GRAD<"tex.grad.2d.v4.s32.f32", int_nvvm_tex_unified_2d_grad_v4s32_f32>;
3762: defm TEX_UNIFIED_2D_U32_F32_GRAD
3763:   : TEX_UNIFIED_2D_GRAD<"tex.grad.2d.v4.u32.f32", int_nvvm_tex_unified_2d_grad_v4u32_f32>;
3764:
3765: class TEX_UNIFIED_2D_ARRAY_base<string inst, dag tex, list<dag> pattern = []>
3766:     : NVPTXInst<(outs B32:$r, B32:$g, B32:$b, B32:$a),
3767:                  !con(tex, (ins B32:$l, B32:$x, B32:$y)),
3768:                  inst # " \t\\{$r, $g, $b, $a\\}, [$t, \\{$l, $x, $y, $y\\}];",
3769:                  pattern>;
3770: multiclass TEX_UNIFIED_2D_ARRAY<string inst, Intrinsic intr> {
3771:   def _R : TEX_UNIFIED_2D_ARRAY_base<inst, (ins B64:$t),
3772:       [(set B32:$r, B32:$g, B32:$b, B32:$a,
3773:             (intr i64:$t, B32:$l, B32:$x, B32:$y))]>;
3774:   def _I : TEX_UNIFIED_2D_ARRAY_base<inst, (ins i64imm:$t)>;
3775: }
3776:
3777: defm TEX_UNIFIED_2D_ARRAY_F32_S32
3778:   : TEX_UNIFIED_2D_ARRAY<"tex.a2d.v4.f32.s32", int_nvvm_tex_unified_2d_array_v4f32_s32>;
3779: defm TEX_UNIFIED_2D_ARRAY_F32_F32
3780:   : TEX_UNIFIED_2D_ARRAY<"tex.a2d.v4.f32.f32", int_nvvm_tex_unified_2d_array_v4f32_f32>;
```
- EN: This range uses TableGen DSL to describe records such as TEX_UNIFIED_1D_ARRAY_S32_F32, TEX_UNIFIED_1D_ARRAY_U32_S32, TEX_UNIFIED_1D_ARRAY_U32_F32, TEX_UNIFIED_1D_ARRAY_LEVEL_base; LLVM later expands these declarations into generated backend tables and helper code.
- CN: 这一段使用 TableGen DSL 描述 TEX_UNIFIED_1D_ARRAY_S32_F32、TEX_UNIFIED_1D_ARRAY_U32_S32、TEX_UNIFIED_1D_ARRAY_U32_F32、TEX_UNIFIED_1D_ARRAY_LEVEL_base 等记录；LLVM 后续会把这些声明展开为生成的后端表和辅助代码。

### Lines 3781-3920
```tablegen
3781: defm TEX_UNIFIED_2D_ARRAY_S32_S32
3782:   : TEX_UNIFIED_2D_ARRAY<"tex.a2d.v4.s32.s32", int_nvvm_tex_unified_2d_array_v4s32_s32>;
3783: defm TEX_UNIFIED_2D_ARRAY_S32_F32
3784:   : TEX_UNIFIED_2D_ARRAY<"tex.a2d.v4.s32.f32", int_nvvm_tex_unified_2d_array_v4s32_f32>;
3785: defm TEX_UNIFIED_2D_ARRAY_U32_S32
3786:   : TEX_UNIFIED_2D_ARRAY<"tex.a2d.v4.u32.s32", int_nvvm_tex_unified_2d_array_v4u32_s32>;
3787: defm TEX_UNIFIED_2D_ARRAY_U32_F32
3788:   : TEX_UNIFIED_2D_ARRAY<"tex.a2d.v4.u32.f32", int_nvvm_tex_unified_2d_array_v4u32_f32>;
3789:
3790: class TEX_UNIFIED_2D_ARRAY_LEVEL_base<string inst, dag tex, list<dag> pattern = []>
3791:     : NVPTXInst<(outs B32:$r, B32:$g, B32:$b, B32:$a),
3792:                  !con(tex, (ins B32:$l, B32:$x, B32:$y, B32:$lod)),
3793:                  inst # " \t\\{$r, $g, $b, $a\\},"
3794:                         "  [$t, \\{$l, $x, $y, $y\\}], $lod;",
3795:                  pattern>;
3796: multiclass TEX_UNIFIED_2D_ARRAY_LEVEL<string inst, Intrinsic intr> {
3797:   def _R : TEX_UNIFIED_2D_ARRAY_LEVEL_base<inst, (ins B64:$t),
3798:       [(set B32:$r, B32:$g, B32:$b, B32:$a,
3799:             (intr i64:$t, B32:$l, B32:$x, B32:$y, B32:$lod))]>;
3800:   def _I : TEX_UNIFIED_2D_ARRAY_LEVEL_base<inst, (ins i64imm:$t)>;
3801: }
3802:
3803: defm TEX_UNIFIED_2D_ARRAY_F32_F32_LEVEL
3804:   : TEX_UNIFIED_2D_ARRAY_LEVEL<"tex.level.a2d.v4.f32.f32",
3805:                                int_nvvm_tex_unified_2d_array_level_v4f32_f32>;
3806: defm TEX_UNIFIED_2D_ARRAY_S32_F32_LEVEL
3807:   : TEX_UNIFIED_2D_ARRAY_LEVEL<"tex.level.a2d.v4.s32.f32",
3808:                                int_nvvm_tex_unified_2d_array_level_v4s32_f32>;
3809: defm TEX_UNIFIED_2D_ARRAY_U32_F32_LEVEL
3810:   : TEX_UNIFIED_2D_ARRAY_LEVEL<"tex.level.a2d.v4.u32.f32",
3811:                                int_nvvm_tex_unified_2d_array_level_v4u32_f32>;
3812:
3813: class TEX_UNIFIED_2D_ARRAY_GRAD_base<string inst, dag tex, list<dag> pattern = []>
3814:     : NVPTXInst<(outs B32:$r, B32:$g, B32:$b, B32:$a),
3815:                  !con(tex, (ins B32:$l, B32:$x, B32:$y,
3816:                                 B32:$gradx0, B32:$gradx1,
3817:                                 B32:$grady0, B32:$grady1)),
3818:                  inst # " \t\\{$r, $g, $b, $a\\}, [$t, \\{$l, $x, $y, $y\\}],"
3819:                         " \\{$gradx0, $gradx1\\}, \\{$grady0, $grady1\\};",
3820:                  pattern>;
3821: multiclass TEX_UNIFIED_2D_ARRAY_GRAD<string inst, Intrinsic intr> {
3822:   def _R : TEX_UNIFIED_2D_ARRAY_GRAD_base<inst, (ins B64:$t),
3823:       [(set B32:$r, B32:$g, B32:$b, B32:$a,
3824:             (intr i64:$t, B32:$l, B32:$x, B32:$y,
3825:                   B32:$gradx0, B32:$gradx1,
3826:                   B32:$grady0, B32:$grady1))]>;
3827:   def _I : TEX_UNIFIED_2D_ARRAY_GRAD_base<inst, (ins i64imm:$t)>;
3828: }
3829:
3830: defm TEX_UNIFIED_2D_ARRAY_F32_F32_GRAD
3831:   : TEX_UNIFIED_2D_ARRAY_GRAD<"tex.grad.a2d.v4.f32.f32",
3832:                               int_nvvm_tex_unified_2d_array_grad_v4f32_f32>;
3833: defm TEX_UNIFIED_2D_ARRAY_S32_F32_GRAD
3834:   : TEX_UNIFIED_2D_ARRAY_GRAD<"tex.grad.a2d.v4.s32.f32",
3835:                               int_nvvm_tex_unified_2d_array_grad_v4s32_f32>;
3836: defm TEX_UNIFIED_2D_ARRAY_U32_F32_GRAD
3837:   : TEX_UNIFIED_2D_ARRAY_GRAD<"tex.grad.a2d.v4.u32.f32",
3838:                               int_nvvm_tex_unified_2d_array_grad_v4u32_f32>;
3839:
3840: class TEX_UNIFIED_3D_base<string inst, dag tex, list<dag> pattern = []>
3841:     : NVPTXInst<(outs B32:$r, B32:$g, B32:$b, B32:$a),
3842:                  !con(tex, (ins B32:$x, B32:$y, B32:$z)),
3843:                  inst # " \t\\{$r, $g, $b, $a\\}, [$t, \\{$x, $y, $z, $z\\}];",
3844:                  pattern>;
3845: multiclass TEX_UNIFIED_3D<string inst, Intrinsic intr> {
3846:   def _R : TEX_UNIFIED_3D_base<inst, (ins B64:$t),
3847:       [(set B32:$r, B32:$g, B32:$b, B32:$a,
3848:             (intr i64:$t, B32:$x, B32:$y, B32:$z))]>;
3849:   def _I : TEX_UNIFIED_3D_base<inst, (ins i64imm:$t)>;
3850: }
3851:
3852: defm TEX_UNIFIED_3D_F32_S32
3853:   : TEX_UNIFIED_3D<"tex.3d.v4.f32.s32", int_nvvm_tex_unified_3d_v4f32_s32>;
3854: defm TEX_UNIFIED_3D_F32_F32
3855:   : TEX_UNIFIED_3D<"tex.3d.v4.f32.f32", int_nvvm_tex_unified_3d_v4f32_f32>;
3856: defm TEX_UNIFIED_3D_S32_S32
3857:   : TEX_UNIFIED_3D<"tex.3d.v4.s32.s32", int_nvvm_tex_unified_3d_v4s32_s32>;
3858: defm TEX_UNIFIED_3D_S32_F32
3859:   : TEX_UNIFIED_3D<"tex.3d.v4.s32.f32", int_nvvm_tex_unified_3d_v4s32_f32>;
3860: defm TEX_UNIFIED_3D_U32_S32
3861:   : TEX_UNIFIED_3D<"tex.3d.v4.u32.s32", int_nvvm_tex_unified_3d_v4u32_s32>;
3862: defm TEX_UNIFIED_3D_U32_F32
3863:   : TEX_UNIFIED_3D<"tex.3d.v4.u32.f32", int_nvvm_tex_unified_3d_v4u32_f32>;
3864:
3865: class TEX_UNIFIED_3D_LEVEL_base<string inst, dag tex, list<dag> pattern = []>
3866:     : NVPTXInst<(outs B32:$r, B32:$g, B32:$b, B32:$a),
3867:                  !con(tex, (ins B32:$x, B32:$y, B32:$z, B32:$lod)),
3868:                  inst # " \t\\{$r, $g, $b, $a\\},"
3869:                         " [$t, \\{$x, $y, $z, $z\\}], $lod;",
3870:                  pattern>;
3871: multiclass TEX_UNIFIED_3D_LEVEL<string inst, Intrinsic intr> {
3872:   def _R : TEX_UNIFIED_3D_LEVEL_base<inst, (ins B64:$t),
3873:       [(set B32:$r, B32:$g, B32:$b, B32:$a,
3874:             (intr i64:$t, B32:$x, B32:$y, B32:$z, B32:$lod))]>;
3875:   def _I : TEX_UNIFIED_3D_LEVEL_base<inst, (ins i64imm:$t)>;
3876: }
3877:
3878: defm TEX_UNIFIED_3D_F32_F32_LEVEL
3879:   : TEX_UNIFIED_3D_LEVEL<"tex.level.3d.v4.f32.f32", int_nvvm_tex_unified_3d_level_v4f32_f32>;
3880: defm TEX_UNIFIED_3D_S32_F32_LEVEL
3881:   : TEX_UNIFIED_3D_LEVEL<"tex.level.3d.v4.s32.f32", int_nvvm_tex_unified_3d_level_v4s32_f32>;
3882: defm TEX_UNIFIED_3D_U32_F32_LEVEL
3883:   : TEX_UNIFIED_3D_LEVEL<"tex.level.3d.v4.u32.f32", int_nvvm_tex_unified_3d_level_v4u32_f32>;
3884:
3885: class TEX_UNIFIED_3D_GRAD_base<string inst, dag tex, list<dag> pattern = []>
3886:     : NVPTXInst<(outs B32:$r, B32:$g, B32:$b, B32:$a),
3887:                  !con(tex, (ins B32:$x, B32:$y, B32:$z,
3888:                                 B32:$gradx0, B32:$gradx1,
3889:                                 B32:$gradx2, B32:$grady0,
3890:                                 B32:$grady1, B32:$grady2)),
3891:                  inst # " \t\\{$r, $g, $b, $a\\}, [$t, \\{$x, $y, $z, $z\\}],"
3892:                         " \\{$gradx0, $gradx1, $gradx2, $gradx2\\},"
3893:                         " \\{$grady0, $grady1, $grady2, $grady2\\};",
3894:                  pattern>;
3895: multiclass TEX_UNIFIED_3D_GRAD<string inst, Intrinsic intr> {
3896:   def _R : TEX_UNIFIED_3D_GRAD_base<inst, (ins B64:$t),
3897:       [(set B32:$r, B32:$g, B32:$b, B32:$a,
3898:             (intr i64:$t, B32:$x, B32:$y, B32:$z,
3899:                   B32:$gradx0, B32:$gradx1, B32:$gradx2,
3900:                   B32:$grady0, B32:$grady1, B32:$grady2))]>;
3901:   def _I : TEX_UNIFIED_3D_GRAD_base<inst, (ins i64imm:$t)>;
3902: }
3903:
3904: defm TEX_UNIFIED_3D_F32_F32_GRAD
3905:   : TEX_UNIFIED_3D_GRAD<"tex.grad.3d.v4.f32.f32", int_nvvm_tex_unified_3d_grad_v4f32_f32>;
3906: defm TEX_UNIFIED_3D_S32_F32_GRAD
3907:   : TEX_UNIFIED_3D_GRAD<"tex.grad.3d.v4.s32.f32", int_nvvm_tex_unified_3d_grad_v4s32_f32>;
3908: defm TEX_UNIFIED_3D_U32_F32_GRAD
3909:   : TEX_UNIFIED_3D_GRAD<"tex.grad.3d.v4.u32.f32", int_nvvm_tex_unified_3d_grad_v4u32_f32>;
3910:
3911: class TEX_UNIFIED_CUBE_base<string inst, dag tex, list<dag> pattern = []>
3912:     : NVPTXInst<(outs B32:$r, B32:$g, B32:$b, B32:$a),
3913:                  !con(tex, (ins B32:$x, B32:$y, B32:$z)),
3914:                  inst # " \t\\{$r, $g, $b, $a\\}, [$t, \\{$x, $y, $z, $z\\}];",
3915:                  pattern>;
3916: multiclass TEX_UNIFIED_CUBE<string inst, Intrinsic intr> {
3917:   def _R : TEX_UNIFIED_CUBE_base<inst, (ins B64:$t),
3918:       [(set B32:$r, B32:$g, B32:$b, B32:$a,
3919:             (intr i64:$t, B32:$x, B32:$y, B32:$z))]>;
3920:   def _I : TEX_UNIFIED_CUBE_base<inst, (ins i64imm:$t)>;
```
- EN: This range uses TableGen DSL to describe records such as TEX_UNIFIED_2D_ARRAY_S32_S32, TEX_UNIFIED_2D_ARRAY_S32_F32, TEX_UNIFIED_2D_ARRAY_U32_S32, TEX_UNIFIED_2D_ARRAY_U32_F32; LLVM later expands these declarations into generated backend tables and helper code.
- CN: 这一段使用 TableGen DSL 描述 TEX_UNIFIED_2D_ARRAY_S32_S32、TEX_UNIFIED_2D_ARRAY_S32_F32、TEX_UNIFIED_2D_ARRAY_U32_S32、TEX_UNIFIED_2D_ARRAY_U32_F32 等记录；LLVM 后续会把这些声明展开为生成的后端表和辅助代码。

### Lines 3921-4060
```tablegen
3921: }
3922:
3923: defm TEX_UNIFIED_CUBE_F32_F32
3924:   : TEX_UNIFIED_CUBE<"tex.cube.v4.f32.f32", int_nvvm_tex_unified_cube_v4f32_f32>;
3925: defm TEX_UNIFIED_CUBE_S32_F32
3926:   : TEX_UNIFIED_CUBE<"tex.cube.v4.s32.f32", int_nvvm_tex_unified_cube_v4s32_f32>;
3927: defm TEX_UNIFIED_CUBE_U32_F32
3928:   : TEX_UNIFIED_CUBE<"tex.cube.v4.u32.f32", int_nvvm_tex_unified_cube_v4u32_f32>;
3929:
3930: class TEX_UNIFIED_CUBE_LEVEL_base<string inst, dag tex, list<dag> pattern = []>
3931:     : NVPTXInst<(outs B32:$r, B32:$g, B32:$b, B32:$a),
3932:                  !con(tex, (ins B32:$x, B32:$y, B32:$z, B32:$lod)),
3933:                  inst # " \t\\{$r, $g, $b, $a\\},"
3934:                         " [$t, \\{$x, $y, $z, $z\\}], $lod;",
3935:                  pattern>;
3936: multiclass TEX_UNIFIED_CUBE_LEVEL<string inst, Intrinsic intr> {
3937:   def _R : TEX_UNIFIED_CUBE_LEVEL_base<inst, (ins B64:$t),
3938:       [(set B32:$r, B32:$g, B32:$b, B32:$a,
3939:             (intr i64:$t, B32:$x, B32:$y, B32:$z, B32:$lod))]>;
3940:   def _I : TEX_UNIFIED_CUBE_LEVEL_base<inst, (ins i64imm:$t)>;
3941: }
3942:
3943: defm TEX_UNIFIED_CUBE_F32_F32_LEVEL
3944:   : TEX_UNIFIED_CUBE_LEVEL<"tex.level.cube.v4.f32.f32",
3945:                            int_nvvm_tex_unified_cube_level_v4f32_f32>;
3946: defm TEX_UNIFIED_CUBE_S32_F32_LEVEL
3947:   : TEX_UNIFIED_CUBE_LEVEL<"tex.level.cube.v4.s32.f32",
3948:                            int_nvvm_tex_unified_cube_level_v4s32_f32>;
3949: defm TEX_UNIFIED_CUBE_U32_F32_LEVEL
3950:   : TEX_UNIFIED_CUBE_LEVEL<"tex.level.cube.v4.u32.f32",
3951:                            int_nvvm_tex_unified_cube_level_v4u32_f32>;
3952:
3953: class TEX_UNIFIED_CUBE_ARRAY_base<string inst, dag tex, list<dag> pattern = []>
3954:     : NVPTXInst<(outs B32:$r, B32:$g, B32:$b, B32:$a),
3955:                  !con(tex, (ins B32:$l, B32:$x, B32:$y, B32:$z)),
3956:                  inst # " \t\\{$r, $g, $b, $a\\}, [$t, \\{$l, $x, $y, $z\\}];",
3957:                  pattern>;
3958: multiclass TEX_UNIFIED_CUBE_ARRAY<string inst, Intrinsic intr> {
3959:   def _R : TEX_UNIFIED_CUBE_ARRAY_base<inst, (ins B64:$t),
3960:       [(set B32:$r, B32:$g, B32:$b, B32:$a,
3961:             (intr i64:$t, i32:$l, B32:$x, B32:$y, B32:$z))]>;
3962:   def _I : TEX_UNIFIED_CUBE_ARRAY_base<inst, (ins i64imm:$t)>;
3963: }
3964:
3965: defm TEX_UNIFIED_CUBE_ARRAY_F32_F32
3966:   : TEX_UNIFIED_CUBE_ARRAY<"tex.acube.v4.f32.f32", int_nvvm_tex_unified_cube_array_v4f32_f32>;
3967: defm TEX_UNIFIED_CUBE_ARRAY_S32_F32
3968:   : TEX_UNIFIED_CUBE_ARRAY<"tex.acube.v4.s32.f32", int_nvvm_tex_unified_cube_array_v4s32_f32>;
3969: defm TEX_UNIFIED_CUBE_ARRAY_U32_F32
3970:   : TEX_UNIFIED_CUBE_ARRAY<"tex.acube.v4.u32.f32", int_nvvm_tex_unified_cube_array_v4u32_f32>;
3971:
3972: class TEX_UNIFIED_CUBE_ARRAY_LEVEL_base<string inst, dag tex, list<dag> pattern = []>
3973:     : NVPTXInst<(outs B32:$r, B32:$g, B32:$b, B32:$a),
3974:                  !con(tex, (ins B32:$l, B32:$x, B32:$y, B32:$z, B32:$lod)),
3975:                  inst # " \t\\{$r, $g, $b, $a\\},"
3976:                         " [$t, \\{$l, $x, $y, $z\\}], $lod;",
3977:                  pattern>;
3978: multiclass TEX_UNIFIED_CUBE_ARRAY_LEVEL<string inst, Intrinsic intr> {
3979:   def _R : TEX_UNIFIED_CUBE_ARRAY_LEVEL_base<inst, (ins B64:$t),
3980:       [(set B32:$r, B32:$g, B32:$b, B32:$a,
3981:             (intr i64:$t, i32:$l, B32:$x, B32:$y, B32:$z, B32:$lod))]>;
3982:   def _I : TEX_UNIFIED_CUBE_ARRAY_LEVEL_base<inst, (ins i64imm:$t)>;
3983: }
3984:
3985: defm TEX_UNIFIED_CUBE_ARRAY_F32_F32_LEVEL
3986:   : TEX_UNIFIED_CUBE_ARRAY_LEVEL<"tex.level.acube.v4.f32.f32",
3987:                                  int_nvvm_tex_unified_cube_array_level_v4f32_f32>;
3988: defm TEX_UNIFIED_CUBE_ARRAY_S32_F32_LEVEL
3989:   : TEX_UNIFIED_CUBE_ARRAY_LEVEL<"tex.level.acube.v4.s32.f32",
3990:                                  int_nvvm_tex_unified_cube_array_level_v4s32_f32>;
3991: defm TEX_UNIFIED_CUBE_ARRAY_U32_F32_LEVEL
3992:   : TEX_UNIFIED_CUBE_ARRAY_LEVEL<"tex.level.acube.v4.u32.f32",
3993:                                  int_nvvm_tex_unified_cube_array_level_v4u32_f32>;
3994:
3995: class TEX_UNIFIED_CUBE_GRAD_base<string inst, dag tex, list<dag> pattern = []>
3996:     : NVPTXInst<(outs B32:$r, B32:$g, B32:$b, B32:$a),
3997:                  !con(tex, (ins B32:$x, B32:$y, B32:$z,
3998:                                 B32:$gradx0, B32:$gradx1,
3999:                                 B32:$gradx2, B32:$grady0,
4000:                                 B32:$grady1, B32:$grady2)),
4001:                  inst # " \t\\{$r, $g, $b, $a\\}, [$t, \\{$x, $y, $z, $z\\}],"
4002:                         " \\{$gradx0, $gradx1, $gradx2, $gradx2\\},"
4003:                         " \\{$grady0, $grady1, $grady2, $grady2\\};",
4004:                  pattern>;
4005:
4006: multiclass TEX_UNIFIED_CUBE_GRAD<string inst, Intrinsic intr> {
4007:   def _R : TEX_UNIFIED_CUBE_GRAD_base<inst, (ins B64:$t),
4008:       [(set B32:$r, B32:$g, B32:$b, B32:$a,
4009:             (intr i64:$t, B32:$x, B32:$y, B32:$z,
4010:                   B32:$gradx0, B32:$gradx1, B32:$gradx2,
4011:                   B32:$grady0, B32:$grady1, B32:$grady2))]>;
4012:   def _I : TEX_UNIFIED_CUBE_GRAD_base<inst, (ins i64imm:$t)>;
4013: }
4014:
4015: defm TEX_UNIFIED_CUBE_F32_F32_GRAD
4016:   : TEX_UNIFIED_CUBE_GRAD<"tex.grad.cube.v4.f32.f32", int_nvvm_tex_unified_cube_grad_v4f32_f32>;
4017: defm TEX_UNIFIED_CUBE_S32_F32_GRAD
4018:   : TEX_UNIFIED_CUBE_GRAD<"tex.grad.cube.v4.s32.f32", int_nvvm_tex_unified_cube_grad_v4s32_f32>;
4019: defm TEX_UNIFIED_CUBE_U32_F32_GRAD
4020:   : TEX_UNIFIED_CUBE_GRAD<"tex.grad.cube.v4.u32.f32", int_nvvm_tex_unified_cube_grad_v4u32_f32>;
4021:
4022: class TEX_UNIFIED_CUBE_ARRAY_GRAD_base<string inst, dag tex, list<dag> pattern = []>
4023:     : NVPTXInst<(outs B32:$r, B32:$g, B32:$b, B32:$a),
4024:                  !con(tex, (ins B32:$l, B32:$x, B32:$y, B32:$z,
4025:                                 B32:$gradx0, B32:$gradx1,
4026:                                 B32:$gradx2, B32:$grady0,
4027:                                 B32:$grady1, B32:$grady2)),
4028:                  inst # " \t\\{$r, $g, $b, $a\\}, [$t, \\{$l, $x, $y, $z\\}],"
4029:                         " \\{$gradx0, $gradx1, $gradx2, $gradx2\\},"
4030:                         " \\{$grady0, $grady1, $grady2, $grady2\\};",
4031:                  pattern>;
4032: multiclass TEX_UNIFIED_CUBE_ARRAY_GRAD<string inst, Intrinsic intr> {
4033:   def _R : TEX_UNIFIED_CUBE_ARRAY_GRAD_base<inst, (ins B64:$t),
4034:       [(set B32:$r, B32:$g, B32:$b, B32:$a,
4035:             (intr i64:$t, i32:$l, B32:$x, B32:$y, B32:$z,
4036:                   B32:$gradx0, B32:$gradx1, B32:$gradx2,
4037:                   B32:$grady0, B32:$grady1, B32:$grady2))]>;
4038:   def _I : TEX_UNIFIED_CUBE_ARRAY_GRAD_base<inst, (ins i64imm:$t)>;
4039: }
4040:
4041: defm TEX_UNIFIED_CUBE_ARRAY_F32_F32_GRAD
4042:   : TEX_UNIFIED_CUBE_ARRAY_GRAD<"tex.grad.acube.v4.f32.f32",
4043:                                 int_nvvm_tex_unified_cube_array_grad_v4f32_f32>;
4044: defm TEX_UNIFIED_CUBE_ARRAY_S32_F32_GRAD
4045:   : TEX_UNIFIED_CUBE_ARRAY_GRAD<"tex.grad.acube.v4.s32.f32",
4046:                                 int_nvvm_tex_unified_cube_array_grad_v4s32_f32>;
4047: defm TEX_UNIFIED_CUBE_ARRAY_U32_F32_GRAD
4048:   : TEX_UNIFIED_CUBE_ARRAY_GRAD<"tex.grad.acube.v4.u32.f32",
4049:                                 int_nvvm_tex_unified_cube_array_grad_v4u32_f32>;
4050:
4051: class TLD4_UNIFIED_2D_base<string inst, dag tex, list<dag> pattern = []>
4052:     : NVPTXInst<(outs B32:$v0, B32:$v1, B32:$v2, B32:$v3),
4053:                  !con(tex, (ins B32:$x, B32:$y)),
4054:                  inst # " \t\\{$v0, $v1, $v2, $v3\\}, [$t, \\{$x, $y\\}];",
4055:                  pattern>;
4056: multiclass TLD4_UNIFIED_2D<string inst, Intrinsic intr> {
4057:   def _R : TLD4_UNIFIED_2D_base<inst, (ins B64:$t),
4058:       [(set B32:$v0, B32:$v1, B32:$v2, B32:$v3,
4059:             (intr i64:$t, B32:$x, B32:$y))]>;
4060:   def _I : TLD4_UNIFIED_2D_base<inst, (ins i64imm:$t)>;
```
- EN: This range uses TableGen DSL to describe records such as TEX_UNIFIED_CUBE_F32_F32, TEX_UNIFIED_CUBE_S32_F32, TEX_UNIFIED_CUBE_U32_F32, TEX_UNIFIED_CUBE_LEVEL_base; LLVM later expands these declarations into generated backend tables and helper code.
- CN: 这一段使用 TableGen DSL 描述 TEX_UNIFIED_CUBE_F32_F32、TEX_UNIFIED_CUBE_S32_F32、TEX_UNIFIED_CUBE_U32_F32、TEX_UNIFIED_CUBE_LEVEL_base 等记录；LLVM 后续会把这些声明展开为生成的后端表和辅助代码。

### Lines 4061-4200
```tablegen
4061: }
4062:
4063: defm TLD4_UNIFIED_R_2D_F32_F32
4064:   : TLD4_UNIFIED_2D<"tld4.r.2d.v4.f32.f32", int_nvvm_tld4_unified_r_2d_v4f32_f32>;
4065: defm TLD4_UNIFIED_G_2D_F32_F32
4066:   : TLD4_UNIFIED_2D<"tld4.g.2d.v4.f32.f32", int_nvvm_tld4_unified_g_2d_v4f32_f32>;
4067: defm TLD4_UNIFIED_B_2D_F32_F32
4068:   : TLD4_UNIFIED_2D<"tld4.b.2d.v4.f32.f32", int_nvvm_tld4_unified_b_2d_v4f32_f32>;
4069: defm TLD4_UNIFIED_A_2D_F32_F32
4070:   : TLD4_UNIFIED_2D<"tld4.a.2d.v4.f32.f32", int_nvvm_tld4_unified_a_2d_v4f32_f32>;
4071:
4072: defm TLD4_UNIFIED_R_2D_S32_F32
4073:   : TLD4_UNIFIED_2D<"tld4.r.2d.v4.s32.f32", int_nvvm_tld4_unified_r_2d_v4s32_f32>;
4074: defm TLD4_UNIFIED_G_2D_S32_F32
4075:   : TLD4_UNIFIED_2D<"tld4.g.2d.v4.s32.f32", int_nvvm_tld4_unified_g_2d_v4s32_f32>;
4076: defm TLD4_UNIFIED_B_2D_S32_F32
4077:   : TLD4_UNIFIED_2D<"tld4.b.2d.v4.s32.f32", int_nvvm_tld4_unified_b_2d_v4s32_f32>;
4078: defm TLD4_UNIFIED_A_2D_S32_F32
4079:   : TLD4_UNIFIED_2D<"tld4.a.2d.v4.s32.f32", int_nvvm_tld4_unified_a_2d_v4s32_f32>;
4080:
4081: defm TLD4_UNIFIED_R_2D_U32_F32
4082:   : TLD4_UNIFIED_2D<"tld4.r.2d.v4.u32.f32", int_nvvm_tld4_unified_r_2d_v4u32_f32>;
4083: defm TLD4_UNIFIED_G_2D_U32_F32
4084:   : TLD4_UNIFIED_2D<"tld4.g.2d.v4.u32.f32", int_nvvm_tld4_unified_g_2d_v4u32_f32>;
4085: defm TLD4_UNIFIED_B_2D_U32_F32
4086:   : TLD4_UNIFIED_2D<"tld4.b.2d.v4.u32.f32", int_nvvm_tld4_unified_b_2d_v4u32_f32>;
4087: defm TLD4_UNIFIED_A_2D_U32_F32
4088:   : TLD4_UNIFIED_2D<"tld4.a.2d.v4.u32.f32", int_nvvm_tld4_unified_a_2d_v4u32_f32>;
4089:
4090: }
4091:
4092:
4093: //=== Surface load instructions
4094:
4095: let IsSuld = true in {
4096:
4097: class SULD_1D_base<string inst, NVPTXRegClass outtype, dag surf,
4098:                    list<dag> pattern = []>
4099:     : NVPTXInst<(outs outtype:$r),
4100:                 !con(surf, (ins B32:$x)),
4101:                 inst # " \\{$r\\}, [$s, \\{$x\\}];",
4102:                 pattern>;
4103: multiclass SULD_1D<string inst, NVPTXRegClass outtype> {
4104:   defvar intr = !cast<Intrinsic>("int_nvvm_" # !tolower(NAME));
4105:
4106:   def _R : SULD_1D_base<inst, outtype, (ins B64:$s),
4107:                         [(set outtype:$r, (intr i64:$s, i32:$x))]>;
4108:   def _I : SULD_1D_base<inst, outtype, (ins i64imm:$s)>;
4109: }
4110:
4111: foreach op = ["clamp", "trap", "zero"] in {
4112:   defvar op_upper = !toupper(op);
4113:   defm SULD_1D_I8_ # op_upper : SULD_1D<"suld.b.1d.b8." # op, B16>;
4114:   defm SULD_1D_I16_ # op_upper : SULD_1D<"suld.b.1d.b16." # op, B16>;
4115:   defm SULD_1D_I32_ # op_upper : SULD_1D<"suld.b.1d.b32." # op, B32>;
4116:   defm SULD_1D_I64_ # op_upper : SULD_1D<"suld.b.1d.b64." # op, B64>;
4117: }
4118:
4119: class SULD_1D_ARRAY_base<string inst, NVPTXRegClass outtype, dag surf,
4120:                          list<dag> pattern = []>
4121:     : NVPTXInst<(outs outtype:$r),
4122:                 !con(surf, (ins B32:$l, B32:$x)),
4123:                 inst # " \\{$r\\}, [$s, \\{$l, $x\\}];",
4124:                 pattern>;
4125: multiclass SULD_1D_ARRAY<string inst, NVPTXRegClass outtype> {
4126:   defvar intr = !cast<Intrinsic>("int_nvvm_" # !tolower(NAME));
4127:
4128:   def _R : SULD_1D_ARRAY_base<inst, outtype, (ins B64:$s),
4129:                               [(set outtype:$r,
4130:                                     (intr i64:$s, i32:$l, i32:$x))]>;
4131:   def _I : SULD_1D_ARRAY_base<inst, outtype, (ins i64imm:$s)>;
4132: }
4133:
4134: foreach op = ["clamp", "trap", "zero"] in {
4135:   defvar op_upper = !toupper(op);
4136:   defm SULD_1D_ARRAY_I8_ # op_upper : SULD_1D_ARRAY<"suld.b.a1d.b8." # op, B16>;
4137:   defm SULD_1D_ARRAY_I16_ # op_upper : SULD_1D_ARRAY<"suld.b.a1d.b16." # op, B16>;
4138:   defm SULD_1D_ARRAY_I32_ # op_upper : SULD_1D_ARRAY<"suld.b.a1d.b32." # op, B32>;
4139:   defm SULD_1D_ARRAY_I64_ # op_upper : SULD_1D_ARRAY<"suld.b.a1d.b64." # op, B64>;
4140: }
4141:
4142: class SULD_2D_base<string inst, NVPTXRegClass outtype, dag surf,
4143:                    list<dag> pattern = []>
4144:     : NVPTXInst<(outs outtype:$r),
4145:                 !con(surf, (ins B32:$x, B32:$y)),
4146:                 inst # " \\{$r\\}, [$s, \\{$x, $y\\}];",
4147:                 pattern>;
4148: multiclass SULD_2D<string inst, NVPTXRegClass outtype> {
4149:   defvar intr = !cast<Intrinsic>("int_nvvm_" # !tolower(NAME));
4150:
4151:   def _R : SULD_2D_base<inst, outtype, (ins B64:$s),
4152:                         [(set outtype:$r, (intr i64:$s, i32:$x, i32:$y))]>;
4153:   def _I : SULD_2D_base<inst, outtype, (ins i64imm:$s)>;
4154: }
4155:
4156: foreach op = ["clamp", "trap", "zero"] in {
4157:   defvar op_upper = !toupper(op);
4158:   defm SULD_2D_I8_ # op_upper : SULD_2D<"suld.b.2d.b8." # op, B16>;
4159:   defm SULD_2D_I16_ # op_upper : SULD_2D<"suld.b.2d.b16." # op, B16>;
4160:   defm SULD_2D_I32_ # op_upper : SULD_2D<"suld.b.2d.b32." # op, B32>;
4161:   defm SULD_2D_I64_ # op_upper : SULD_2D<"suld.b.2d.b64." # op, B64>;
4162: }
4163:
4164: class SULD_2D_ARRAY_base<string inst, NVPTXRegClass outtype, dag surf,
4165:                          list<dag> pattern = []>
4166:     : NVPTXInst<(outs outtype:$r),
4167:                 !con(surf, (ins B32:$l, B32:$x, B32:$y)),
4168:                 inst # " \\{$r\\}, [$s, \\{$l, $x, $y, $y\\}];",
4169:                 pattern>;
4170: multiclass SULD_2D_ARRAY<string inst, NVPTXRegClass outtype> {
4171:   defvar intr = !cast<Intrinsic>("int_nvvm_" # !tolower(NAME));
4172:
4173:   def _R : SULD_2D_ARRAY_base<inst, outtype, (ins B64:$s),
4174:                               [(set outtype:$r,
4175:                                     (intr i64:$s, i32:$l, i32:$x, i32:$y))]>;
4176:   def _I : SULD_2D_ARRAY_base<inst, outtype, (ins i64imm:$s)>;
4177: }
4178:
4179: foreach op = ["clamp", "trap", "zero"] in {
4180:   defvar op_upper = !toupper(op);
4181:   defm SULD_2D_ARRAY_I8_ # op_upper : SULD_2D_ARRAY<"suld.b.a2d.b8." # op, B16>;
4182:   defm SULD_2D_ARRAY_I16_ # op_upper : SULD_2D_ARRAY<"suld.b.a2d.b16." # op, B16>;
4183:   defm SULD_2D_ARRAY_I32_ # op_upper : SULD_2D_ARRAY<"suld.b.a2d.b32." # op, B32>;
4184:   defm SULD_2D_ARRAY_I64_ # op_upper : SULD_2D_ARRAY<"suld.b.a2d.b64." # op, B64>;
4185: }
4186:
4187: class SULD_3D_base<string inst, NVPTXRegClass outtype, dag surf,
4188:                    list<dag> pattern = []>
4189:     : NVPTXInst<(outs outtype:$r),
4190:                 !con(surf, (ins B32:$x, B32:$y, B32:$z)),
4191:                 inst # " \\{$r\\}, [$s, \\{$x, $y, $z, $z\\}];",
4192:                 pattern>;
4193: multiclass SULD_3D<string inst, NVPTXRegClass outtype> {
4194:   defvar intr = !cast<Intrinsic>("int_nvvm_" # !tolower(NAME));
4195:
4196:   def _R : SULD_3D_base<inst, outtype, (ins B64:$s),
4197:                         [(set outtype:$r,
4198:                               (intr i64:$s, i32:$x, i32:$y, i32:$z))]>;
4199:   def _I : SULD_3D_base<inst, outtype, (ins i64imm:$s)>;
4200: }
```
- EN: This range uses TableGen DSL to describe records such as TLD4_UNIFIED_R_2D_F32_F32, TLD4_UNIFIED_G_2D_F32_F32, TLD4_UNIFIED_B_2D_F32_F32, TLD4_UNIFIED_A_2D_F32_F32; LLVM later expands these declarations into generated backend tables and helper code.
- CN: 这一段使用 TableGen DSL 描述 TLD4_UNIFIED_R_2D_F32_F32、TLD4_UNIFIED_G_2D_F32_F32、TLD4_UNIFIED_B_2D_F32_F32、TLD4_UNIFIED_A_2D_F32_F32 等记录；LLVM 后续会把这些声明展开为生成的后端表和辅助代码。

### Lines 4201-4340
```tablegen
4201:
4202: foreach op = ["clamp", "trap", "zero"] in {
4203:   defvar op_upper = !toupper(op);
4204:   defm SULD_3D_I8_ # op_upper : SULD_3D<"suld.b.3d.b8." # op, B16>;
4205:   defm SULD_3D_I16_ # op_upper : SULD_3D<"suld.b.3d.b16." # op, B16>;
4206:   defm SULD_3D_I32_ # op_upper : SULD_3D<"suld.b.3d.b32." # op, B32>;
4207:   defm SULD_3D_I64_ # op_upper : SULD_3D<"suld.b.3d.b64." # op, B64>;
4208: }
4209: }
4210:
4211: let IsSuld = 2 in {
4212:
4213: class SULD_1D_V2_base<string inst, NVPTXRegClass outtype, dag surf,
4214:                       list<dag> pattern = []>
4215:     : NVPTXInst<(outs outtype:$r, outtype:$g),
4216:                 !con(surf, (ins B32:$x)),
4217:                 inst # " \\{$r, $g\\}, [$s, \\{$x\\}];",
4218:                 pattern>;
4219: multiclass SULD_1D_V2<string inst, NVPTXRegClass outtype> {
4220:   defvar intr = !cast<Intrinsic>("int_nvvm_" # !tolower(NAME));
4221:
4222:   def _R : SULD_1D_V2_base<inst, outtype, (ins B64:$s),
4223:                            [(set outtype:$r, outtype:$g,
4224:                                  (intr i64:$s, i32:$x))]>;
4225:   def _I : SULD_1D_V2_base<inst, outtype, (ins i64imm:$s)>;
4226: }
4227:
4228: foreach op = ["clamp", "trap", "zero"] in {
4229:   defvar op_upper = !toupper(op);
4230:   defm SULD_1D_V2I8_ # op_upper : SULD_1D_V2<"suld.b.1d.v2.b8." # op, B16>;
4231:   defm SULD_1D_V2I16_ # op_upper : SULD_1D_V2<"suld.b.1d.v2.b16." # op, B16>;
4232:   defm SULD_1D_V2I32_ # op_upper : SULD_1D_V2<"suld.b.1d.v2.b32." # op, B32>;
4233:   defm SULD_1D_V2I64_ # op_upper : SULD_1D_V2<"suld.b.1d.v2.b64." # op, B64>;
4234: }
4235:
4236: class SULD_1D_ARRAY_V2_base<string inst, NVPTXRegClass outtype, dag surf,
4237:                             list<dag> pattern = []>
4238:     : NVPTXInst<(outs outtype:$r, outtype:$g),
4239:                 !con(surf, (ins B32:$l, B32:$x)),
4240:                 inst # " \\{$r, $g\\}, [$s, \\{$l, $x\\}];",
4241:                 pattern>;
4242: multiclass SULD_1D_ARRAY_V2<string inst, NVPTXRegClass outtype> {
4243:   defvar intr = !cast<Intrinsic>("int_nvvm_" # !tolower(NAME));
4244:
4245:   def _R : SULD_1D_ARRAY_V2_base<inst, outtype, (ins B64:$s),
4246:                                  [(set outtype:$r, outtype:$g,
4247:                                        (intr i64:$s, i32:$l, i32:$x))]>;
4248:   def _I : SULD_1D_ARRAY_V2_base<inst, outtype, (ins i64imm:$s)>;
4249: }
4250:
4251: foreach op = ["clamp", "trap", "zero"] in {
4252:   defvar op_upper = !toupper(op);
4253:   defm SULD_1D_ARRAY_V2I8_ # op_upper : SULD_1D_ARRAY_V2<"suld.b.a1d.v2.b8." # op, B16>;
4254:   defm SULD_1D_ARRAY_V2I16_ # op_upper : SULD_1D_ARRAY_V2<"suld.b.a1d.v2.b16." # op, B16>;
4255:   defm SULD_1D_ARRAY_V2I32_ # op_upper : SULD_1D_ARRAY_V2<"suld.b.a1d.v2.b32." # op, B32>;
4256:   defm SULD_1D_ARRAY_V2I64_ # op_upper : SULD_1D_ARRAY_V2<"suld.b.a1d.v2.b64." # op, B64>;
4257: }
4258:
4259: class SULD_2D_V2_base<string inst, NVPTXRegClass outtype, dag surf,
4260:                       list<dag> pattern = []>
4261:     : NVPTXInst<(outs outtype:$r, outtype:$g),
4262:                 !con(surf, (ins B32:$x, B32:$y)),
4263:                 inst # " \\{$r, $g\\}, [$s, \\{$x, $y\\}];",
4264:                 pattern>;
4265: multiclass SULD_2D_V2<string inst, NVPTXRegClass outtype> {
4266:   defvar intr = !cast<Intrinsic>("int_nvvm_" # !tolower(NAME));
4267:
4268:   def _R : SULD_2D_V2_base<inst, outtype, (ins B64:$s),
4269:                            [(set outtype:$r, outtype:$g,
4270:                                  (intr i64:$s, i32:$x, i32:$y))]>;
4271:   def _I : SULD_2D_V2_base<inst, outtype, (ins i64imm:$s)>;
4272: }
4273:
4274: foreach op = ["clamp", "trap", "zero"] in {
4275:   defvar op_upper = !toupper(op);
4276:   defm SULD_2D_V2I8_ # op_upper : SULD_2D_V2<"suld.b.2d.v2.b8." # op, B16>;
4277:   defm SULD_2D_V2I16_ # op_upper : SULD_2D_V2<"suld.b.2d.v2.b16." # op, B16>;
4278:   defm SULD_2D_V2I32_ # op_upper : SULD_2D_V2<"suld.b.2d.v2.b32." # op, B32>;
4279:   defm SULD_2D_V2I64_ # op_upper : SULD_2D_V2<"suld.b.2d.v2.b64." # op, B64>;
4280: }
4281:
4282: class SULD_2D_ARRAY_V2_base<string inst, NVPTXRegClass outtype, dag surf,
4283:                             list<dag> pattern = []>
4284:     : NVPTXInst<(outs outtype:$r, outtype:$g),
4285:                 !con(surf, (ins B32:$l, B32:$x, B32:$y)),
4286:                 inst # " \\{$r, $g\\}, [$s, \\{$l, $x, $y, $y\\}];",
4287:                 pattern>;
4288: multiclass SULD_2D_ARRAY_V2<string inst, NVPTXRegClass outtype> {
4289:   defvar intr = !cast<Intrinsic>("int_nvvm_" # !tolower(NAME));
4290:
4291:   def _R : SULD_2D_ARRAY_V2_base<inst, outtype, (ins B64:$s),
4292:                                  [(set outtype:$r, outtype:$g,
4293:                                        (intr i64:$s, i32:$l, i32:$x, i32:$y))]>;
4294:   def _I : SULD_2D_ARRAY_V2_base<inst, outtype, (ins i64imm:$s)>;
4295: }
4296:
4297: foreach op = ["clamp", "trap", "zero"] in {
4298:   defvar op_upper = !toupper(op);
4299:   defm SULD_2D_ARRAY_V2I8_ # op_upper : SULD_2D_ARRAY_V2<"suld.b.a2d.v2.b8." # op, B16>;
4300:   defm SULD_2D_ARRAY_V2I16_ # op_upper : SULD_2D_ARRAY_V2<"suld.b.a2d.v2.b16." # op, B16>;
4301:   defm SULD_2D_ARRAY_V2I32_ # op_upper : SULD_2D_ARRAY_V2<"suld.b.a2d.v2.b32." # op, B32>;
4302:   defm SULD_2D_ARRAY_V2I64_ # op_upper : SULD_2D_ARRAY_V2<"suld.b.a2d.v2.b64." # op, B64>;
4303: }
4304:
4305: class SULD_3D_V2_base<string inst, NVPTXRegClass outtype, dag surf,
4306:                       list<dag> pattern = []>
4307:     : NVPTXInst<(outs outtype:$r, outtype:$g),
4308:                 !con(surf, (ins B32:$x, B32:$y, B32:$z)),
4309:                 inst # " \\{$r, $g\\}, [$s, \\{$x, $y, $z, $z\\}];",
4310:                 pattern>;
4311: multiclass SULD_3D_V2<string inst, NVPTXRegClass outtype> {
4312:   defvar intr = !cast<Intrinsic>("int_nvvm_" # !tolower(NAME));
4313:
4314:   def _R : SULD_3D_V2_base<inst, outtype, (ins B64:$s),
4315:                            [(set outtype:$r, outtype:$g,
4316:                                  (intr i64:$s, i32:$x, i32:$y, i32:$z))]>;
4317:   def _I : SULD_3D_V2_base<inst, outtype, (ins i64imm:$s)>;
4318: }
4319:
4320: foreach op = ["clamp", "trap", "zero"] in {
4321:   defvar op_upper = !toupper(op);
4322:   defm SULD_3D_V2I8_ # op_upper : SULD_3D_V2<"suld.b.3d.v2.b8." # op, B16>;
4323:   defm SULD_3D_V2I16_ # op_upper : SULD_3D_V2<"suld.b.3d.v2.b16." # op, B16>;
4324:   defm SULD_3D_V2I32_ # op_upper : SULD_3D_V2<"suld.b.3d.v2.b32." # op, B32>;
4325:   defm SULD_3D_V2I64_ # op_upper : SULD_3D_V2<"suld.b.3d.v2.b64." # op, B64>;
4326: }
4327:
4328: }
4329:
4330: let IsSuld = 3 in {
4331:
4332: class SULD_1D_V4_base<string inst, NVPTXRegClass outtype, dag surf,
4333:                       list<dag> pattern = []>
4334:     : NVPTXInst<(outs outtype:$r, outtype:$g, outtype:$b, outtype:$a),
4335:                 !con(surf, (ins B32:$x)),
4336:                 inst # " \\{$r, $g, $b, $a\\}, [$s, \\{$x\\}];",
4337:                 pattern>;
4338: multiclass SULD_1D_V4<string inst, NVPTXRegClass outtype> {
4339:   defvar intr = !cast<Intrinsic>("int_nvvm_" # !tolower(NAME));
4340:
```
- EN: This range uses TableGen DSL to describe records such as SULD_3D_I8_, SULD_3D_I16_, SULD_3D_I32_, SULD_3D_I64_; LLVM later expands these declarations into generated backend tables and helper code.
- CN: 这一段使用 TableGen DSL 描述 SULD_3D_I8_、SULD_3D_I16_、SULD_3D_I32_、SULD_3D_I64_ 等记录；LLVM 后续会把这些声明展开为生成的后端表和辅助代码。

### Lines 4341-4480
```tablegen
4341:   def _R : SULD_1D_V4_base<inst, outtype, (ins B64:$s),
4342:                            [(set outtype:$r, outtype:$g, outtype:$b, outtype:$a,
4343:                                  (intr i64:$s, i32:$x))]>;
4344:   def _I : SULD_1D_V4_base<inst, outtype, (ins i64imm:$s)>;
4345: }
4346:
4347: foreach op = ["clamp", "trap", "zero"] in {
4348:   defvar op_upper = !toupper(op);
4349:   defm SULD_1D_V4I8_ # op_upper : SULD_1D_V4<"suld.b.1d.v4.b8." # op, B16>;
4350:   defm SULD_1D_V4I16_ # op_upper : SULD_1D_V4<"suld.b.1d.v4.b16." # op, B16>;
4351:   defm SULD_1D_V4I32_ # op_upper : SULD_1D_V4<"suld.b.1d.v4.b32." # op, B32>;
4352: }
4353:
4354: class SULD_1D_ARRAY_V4_base<string inst, NVPTXRegClass outtype, dag surf,
4355:                             list<dag> pattern = []>
4356:     : NVPTXInst<(outs outtype:$r, outtype:$g, outtype:$b, outtype:$a),
4357:                 !con(surf, (ins B32:$l, B32:$x)),
4358:                 inst # " \\{$r, $g, $b, $a\\}, [$s, \\{$l, $x\\}];",
4359:                 pattern>;
4360: multiclass SULD_1D_ARRAY_V4<string inst, NVPTXRegClass outtype> {
4361:   defvar intr = !cast<Intrinsic>("int_nvvm_" # !tolower(NAME));
4362:
4363:   def _R : SULD_1D_ARRAY_V4_base<inst, outtype, (ins B64:$s),
4364:                                  [(set outtype:$r, outtype:$g, outtype:$b,
4365:                                        outtype:$a,
4366:                                        (intr i64:$s, i32:$l, i32:$x))]>;
4367:   def _I : SULD_1D_ARRAY_V4_base<inst, outtype, (ins i64imm:$s)>;
4368: }
4369:
4370: foreach op = ["clamp", "trap", "zero"] in {
4371:   defvar op_upper = !toupper(op);
4372:   defm SULD_1D_ARRAY_V4I8_ # op_upper : SULD_1D_ARRAY_V4<"suld.b.a1d.v4.b8." # op, B16>;
4373:   defm SULD_1D_ARRAY_V4I16_ # op_upper : SULD_1D_ARRAY_V4<"suld.b.a1d.v4.b16." # op, B16>;
4374:   defm SULD_1D_ARRAY_V4I32_ # op_upper : SULD_1D_ARRAY_V4<"suld.b.a1d.v4.b32." # op, B32>;
4375: }
4376:
4377: class SULD_2D_V4_base<string inst, NVPTXRegClass outtype, dag surf,
4378:                       list<dag> pattern = []>
4379:     : NVPTXInst<(outs outtype:$r, outtype:$g, outtype:$b, outtype:$a),
4380:                 !con(surf, (ins B32:$x, B32:$y)),
4381:                 inst # " \\{$r, $g, $b, $a\\}, [$s, \\{$x, $y\\}];",
4382:                 pattern>;
4383: multiclass SULD_2D_V4<string inst, NVPTXRegClass outtype> {
4384:   defvar intr = !cast<Intrinsic>("int_nvvm_" # !tolower(NAME));
4385:
4386:   def _R : SULD_2D_V4_base<inst, outtype, (ins B64:$s),
4387:                            [(set outtype:$r, outtype:$g, outtype:$b, outtype:$a,
4388:                                  (intr i64:$s, i32:$x, i32:$y))]>;
4389:   def _I : SULD_2D_V4_base<inst, outtype, (ins i64imm:$s)>;
4390: }
4391:
4392: foreach op = ["clamp", "trap", "zero"] in {
4393:   defvar op_upper = !toupper(op);
4394:   defm SULD_2D_V4I8_ # op_upper : SULD_2D_V4<"suld.b.2d.v4.b8." # op, B16>;
4395:   defm SULD_2D_V4I16_ # op_upper : SULD_2D_V4<"suld.b.2d.v4.b16." # op, B16>;
4396:   defm SULD_2D_V4I32_ # op_upper : SULD_2D_V4<"suld.b.2d.v4.b32." # op, B32>;
4397: }
4398:
4399: class SULD_2D_ARRAY_V4_base<string inst, NVPTXRegClass outtype, dag surf,
4400:                             list<dag> pattern = []>
4401:     : NVPTXInst<(outs outtype:$r, outtype:$g, outtype:$b, outtype:$a),
4402:                 !con(surf, (ins B32:$l, B32:$x, B32:$y)),
4403:                 inst # " \\{$r, $g, $b, $a\\}, [$s, \\{$l, $x, $y, $y\\}];",
4404:                 pattern>;
4405: multiclass SULD_2D_ARRAY_V4<string inst, NVPTXRegClass outtype> {
4406:   defvar intr = !cast<Intrinsic>("int_nvvm_" # !tolower(NAME));
4407:
4408:   def _R : SULD_2D_ARRAY_V4_base<inst, outtype, (ins B64:$s),
4409:                                  [(set outtype:$r, outtype:$g, outtype:$b,
4410:                                        outtype:$a,
4411:                                        (intr i64:$s, i32:$l, i32:$x, i32:$y))]>;
4412:   def _I : SULD_2D_ARRAY_V4_base<inst, outtype, (ins i64imm:$s)>;
4413: }
4414:
4415: foreach op = ["clamp", "trap", "zero"] in {
4416:   defvar op_upper = !toupper(op);
4417:   defm SULD_2D_ARRAY_V4I8_ # op_upper : SULD_2D_ARRAY_V4<"suld.b.a2d.v4.b8." # op, B16>;
4418:   defm SULD_2D_ARRAY_V4I16_ # op_upper : SULD_2D_ARRAY_V4<"suld.b.a2d.v4.b16." # op, B16>;
4419:   defm SULD_2D_ARRAY_V4I32_ # op_upper : SULD_2D_ARRAY_V4<"suld.b.a2d.v4.b32." # op, B32>;
4420: }
4421:
4422: class SULD_3D_V4_base<string inst, NVPTXRegClass outtype, dag surf,
4423:                       list<dag> pattern = []>
4424:     : NVPTXInst<(outs outtype:$r, outtype:$g, outtype:$b, outtype:$a),
4425:                 !con(surf, (ins B32:$x, B32:$y, B32:$z)),
4426:                 inst # " \\{$r, $g, $b, $a\\}, [$s, \\{$x, $y, $z, $z\\}];",
4427:                 pattern>;
4428: multiclass SULD_3D_V4<string inst, NVPTXRegClass outtype> {
4429:   defvar intr = !cast<Intrinsic>("int_nvvm_" # !tolower(NAME));
4430:
4431:   def _R : SULD_3D_V4_base<inst, outtype, (ins B64:$s),
4432:                            [(set outtype:$r, outtype:$g, outtype:$b, outtype:$a,
4433:                                  (intr i64:$s, i32:$x, i32:$y, i32:$z))]>;
4434:   def _I : SULD_3D_V4_base<inst, outtype, (ins i64imm:$s)>;
4435: }
4436:
4437: foreach op = ["clamp", "trap", "zero"] in {
4438:   defvar op_upper = !toupper(op);
4439:   defm SULD_3D_V4I8_ # op_upper : SULD_3D_V4<"suld.b.3d.v4.b8." # op, B16>;
4440:   defm SULD_3D_V4I16_ # op_upper : SULD_3D_V4<"suld.b.3d.v4.b16." # op, B16>;
4441:   defm SULD_3D_V4I32_ # op_upper : SULD_3D_V4<"suld.b.3d.v4.b32." # op, B32>;
4442: }
4443:
4444: }
4445:
4446: //-----------------------------------
4447: // Texture Query Intrinsics
4448: //-----------------------------------
4449:
4450: let IsSurfTexQuery = true in {
4451:   foreach query = ["channel_order", "channel_data_type", "width", "height", 
4452:                    "depth", "array_size", "num_samples", "num_mipmap_levels"] in {
4453:     def TXQ_ # !toupper(query) # _R
4454:       : NVPTXInst<(outs B32:$d), (ins B64:$a),
4455:                   "txq." # query # ".b32 \t$d, [$a];",
4456:                   [(set i32:$d, (!cast<Intrinsic>("int_nvvm_txq_" # query) i64:$a))]>;
4457:     def TXQ_ # !toupper(query) # _I
4458:       : NVPTXInst<(outs B32:$d), (ins i64imm:$a),
4459:                   "txq." # query # ".b32 \t$d, [$a];",
4460:                   []>;
4461:   }
4462: }
4463:
4464: //-----------------------------------
4465: // Surface Query Intrinsics
4466: //-----------------------------------
4467:
4468: let IsSurfTexQuery = true in {
4469:   foreach query = ["channel_order", "channel_data_type", "width", "height", "depth", "array_size"] in {
4470:     def SUQ_ # !toupper(query) # _R
4471:       : NVPTXInst<(outs B32:$d), (ins B64:$a),
4472:                   "suq." # query # ".b32 \t$d, [$a];",
4473:                   [(set i32:$d, (!cast<Intrinsic>("int_nvvm_suq_" # query) i64:$a))]>;
4474:     def SUQ_ # !toupper(query) # _I
4475:       : NVPTXInst<(outs B32:$d), (ins i64imm:$a),
4476:                   "suq." # query # ".b32 \t$d, [$a];",
4477:                   []>;
4478:   }
4479: }
4480:
```
- EN: This range uses TableGen DSL to describe records such as _R, _I, SULD_1D_V4I8_, SULD_1D_V4I16_; LLVM later expands these declarations into generated backend tables and helper code.
- CN: 这一段使用 TableGen DSL 描述 _R、_I、SULD_1D_V4I8_、SULD_1D_V4I16_ 等记录；LLVM 后续会把这些声明展开为生成的后端表和辅助代码。

### Lines 4481-4620
```tablegen
4481: //===- Handle Query -------------------------------------------------------===//
4482:
4483: // TODO: These intrinsics are not yet finalized, pending PTX ISA design work
4484: def ISTYPEP_SAMPLER
4485:   : BasicNVPTXInst<(outs B1:$d), (ins B64:$a),
4486:               "istypep.samplerref",
4487:               [(set i1:$d, (int_nvvm_istypep_sampler i64:$a))]>;
4488: def ISTYPEP_SURFACE
4489:   : BasicNVPTXInst<(outs B1:$d), (ins B64:$a),
4490:               "istypep.surfref",
4491:               [(set i1:$d, (int_nvvm_istypep_surface i64:$a))]>;
4492: def ISTYPEP_TEXTURE
4493:   : BasicNVPTXInst<(outs B1:$d), (ins B64:$a),
4494:               "istypep.texref",
4495:               [(set i1:$d, (int_nvvm_istypep_texture i64:$a))]>;
4496:
4497: //===- Surface Stores -----------------------------------------------------===//
4498:
4499: let IsSust = true in {
4500:
4501: class SUST_1D_base<string inst, NVPTXRegClass intype, dag surf, list<dag> pat>
4502:     : NVPTXInst<(outs),
4503:                 !con(surf, (ins B32:$x, intype:$r)),
4504:                 inst # " \t[$s, \\{$x\\}], \\{$r\\};", pat>;
4505: multiclass SUST_1D<string inst, NVPTXRegClass intype> {
4506:   defvar intr = !cast<Intrinsic>("int_nvvm_" # !tolower(NAME));
4507:
4508:   def _R : SUST_1D_base<inst, intype, (ins B64:$s),
4509:               [(intr B64:$s, B32:$x, intype:$r)]>;
4510:   def _I : SUST_1D_base<inst, intype, (ins i64imm:$s), []>;
4511: }
4512:
4513: foreach op = ["clamp", "trap", "zero"] in {
4514:   defvar op_upper = !toupper(op);
4515:   defm SUST_B_1D_I8_ # op_upper : SUST_1D<"sust.b.1d.b8." # op, B16>;
4516:   defm SUST_B_1D_I16_ # op_upper : SUST_1D<"sust.b.1d.b16." # op, B16>;
4517:   defm SUST_B_1D_I32_ # op_upper : SUST_1D<"sust.b.1d.b32." # op, B32>;
4518:   defm SUST_B_1D_I64_ # op_upper : SUST_1D<"sust.b.1d.b64." # op, B64>;
4519: }
4520:
4521: defm SUST_P_1D_I8_TRAP : SUST_1D<"sust.p.1d.b8.trap", B16>;
4522: defm SUST_P_1D_I16_TRAP : SUST_1D<"sust.p.1d.b16.trap", B16>;
4523: defm SUST_P_1D_I32_TRAP : SUST_1D<"sust.p.1d.b32.trap", B32>;
4524:
4525: class SUST_1D_V2_base<string inst, NVPTXRegClass intype, dag surf, list<dag> pat>
4526:     : NVPTXInst<(outs),
4527:                 !con(surf, (ins B32:$x, intype:$r, intype:$g)),
4528:                 inst # " \t[$s, \\{$x\\}], \\{$r, $g\\};",
4529:                 pat>;
4530: multiclass SUST_1D_V2<string inst, NVPTXRegClass intype> {
4531:   defvar intr = !cast<Intrinsic>("int_nvvm_" # !tolower(NAME));
4532:   def _R : SUST_1D_V2_base<inst, intype, (ins B64:$s),
4533:               [(intr B64:$s, B32:$x, intype:$r, intype:$g)]>;
4534:   def _I : SUST_1D_V2_base<inst, intype, (ins i64imm:$s), []>;
4535: }
4536:
4537: foreach op = ["clamp", "trap", "zero"] in {
4538:   defvar op_upper = !toupper(op);
4539:   defm SUST_B_1D_V2I8_ # op_upper : SUST_1D_V2<"sust.b.1d.v2.b8." # op, B16>;
4540:   defm SUST_B_1D_V2I16_ # op_upper : SUST_1D_V2<"sust.b.1d.v2.b16." # op, B16>;
4541:   defm SUST_B_1D_V2I32_ # op_upper : SUST_1D_V2<"sust.b.1d.v2.b32." # op, B32>;
4542:   defm SUST_B_1D_V2I64_ # op_upper : SUST_1D_V2<"sust.b.1d.v2.b64." # op, B64>;
4543: }
4544: defm SUST_P_1D_V2I8_TRAP : SUST_1D_V2<"sust.p.1d.v2.b8.trap", B16>;
4545: defm SUST_P_1D_V2I16_TRAP : SUST_1D_V2<"sust.p.1d.v2.b16.trap", B16>;
4546: defm SUST_P_1D_V2I32_TRAP : SUST_1D_V2<"sust.p.1d.v2.b32.trap", B32>;
4547:
4548: class SUST_1D_V4_base<string inst, NVPTXRegClass intype, dag surf, list<dag> pat>
4549:     : NVPTXInst<(outs),
4550:                 !con(surf, (ins B32:$x, intype:$r, intype:$g,
4551:                                 intype:$b, intype:$a)),
4552:                 inst # " \t[$s, \\{$x\\}], \\{$r, $g, $b, $a\\};",
4553:                 pat>;
4554: multiclass SUST_1D_V4<string inst, NVPTXRegClass intype> {
4555:   defvar intr = !cast<Intrinsic>("int_nvvm_" # !tolower(NAME));
4556:   def _R : SUST_1D_V4_base<inst, intype, (ins B64:$s),
4557:               [(intr B64:$s, B32:$x, intype:$r, intype:$g,
4558:                 intype:$b, intype:$a)]>;
4559:   def _I : SUST_1D_V4_base<inst, intype, (ins i64imm:$s), []>;
4560: }
4561:
4562: foreach op = ["clamp", "trap", "zero"] in {
4563:   defvar op_upper = !toupper(op);
4564:   defm SUST_B_1D_V4I8_ # op_upper : SUST_1D_V4<"sust.b.1d.v4.b8." # op, B16>;
4565:   defm SUST_B_1D_V4I16_ # op_upper : SUST_1D_V4<"sust.b.1d.v4.b16." # op, B16>;
4566:   defm SUST_B_1D_V4I32_ # op_upper : SUST_1D_V4<"sust.b.1d.v4.b32." # op, B32>;
4567: }
4568:
4569: defm SUST_P_1D_V4I8_TRAP : SUST_1D_V4<"sust.p.1d.v4.b8.trap", B16>;
4570: defm SUST_P_1D_V4I16_TRAP : SUST_1D_V4<"sust.p.1d.v4.b16.trap", B16>;
4571: defm SUST_P_1D_V4I32_TRAP : SUST_1D_V4<"sust.p.1d.v4.b32.trap", B32>;
4572:
4573: class SUST_1D_ARRAY_base<string inst, NVPTXRegClass intype, dag surf, list<dag> pat>
4574:     : NVPTXInst<(outs),
4575:                 !con(surf, (ins B32:$idx, B32:$x, intype:$r)),
4576:                 inst # " \t[$s, \\{$idx, $x\\}], \\{$r\\};",
4577:                 pat>;
4578: multiclass SUST_1D_ARRAY<string inst, NVPTXRegClass intype> {
4579:     defvar intr = !cast<Intrinsic>("int_nvvm_" # !tolower(NAME));
4580:   def _R : SUST_1D_ARRAY_base<inst, intype, (ins B64:$s),
4581:               [(intr B64:$s, B32:$idx, B32:$x, intype:$r)]>;
4582:   def _I : SUST_1D_ARRAY_base<inst, intype, (ins i64imm:$s), []>;
4583: }
4584:
4585: foreach op = ["clamp", "trap", "zero"] in {
4586:   defvar op_upper = !toupper(op);
4587:   defm SUST_B_1D_ARRAY_I8_ # op_upper : SUST_1D_ARRAY<"sust.b.a1d.b8." # op, B16>;
4588:   defm SUST_B_1D_ARRAY_I16_ # op_upper : SUST_1D_ARRAY<"sust.b.a1d.b16." # op, B16>;
4589:   defm SUST_B_1D_ARRAY_I32_ # op_upper : SUST_1D_ARRAY<"sust.b.a1d.b32." # op, B32>;
4590:   defm SUST_B_1D_ARRAY_I64_ # op_upper : SUST_1D_ARRAY<"sust.b.a1d.b64." # op, B64>;
4591: }
4592:
4593: defm SUST_P_1D_ARRAY_I8_TRAP : SUST_1D_ARRAY<"sust.p.a1d.b8.trap", B16>;
4594: defm SUST_P_1D_ARRAY_I16_TRAP : SUST_1D_ARRAY<"sust.p.a1d.b16.trap", B16>;
4595: defm SUST_P_1D_ARRAY_I32_TRAP : SUST_1D_ARRAY<"sust.p.a1d.b32.trap", B32>;
4596:
4597: class SUST_1D_ARRAY_V2_base<string inst, NVPTXRegClass intype, dag surf, list<dag> pat>
4598:     : NVPTXInst<(outs),
4599:                 !con(surf, (ins B32:$idx, B32:$x,
4600:                                 intype:$r, intype:$g)),
4601:                 inst # " \t[$s, \\{$idx, $x\\}], \\{$r, $g\\};",
4602:                 pat>;
4603: multiclass SUST_1D_ARRAY_V2<string inst, NVPTXRegClass intype> {
4604:   defvar intr = !cast<Intrinsic>("int_nvvm_" # !tolower(NAME));
4605:   def _R : SUST_1D_ARRAY_V2_base<inst, intype, (ins B64:$s),
4606:               [(intr B64:$s, B32:$idx, B32:$x,
4607:                 intype:$r, intype:$g)]>;
4608:   def _I : SUST_1D_ARRAY_V2_base<inst, intype, (ins i64imm:$s), []>;
4609: }
4610:
4611: foreach op = ["clamp", "trap", "zero"] in {
4612:   defvar op_upper = !toupper(op);
4613:   defm SUST_B_1D_ARRAY_V2I8_ # op_upper : SUST_1D_ARRAY_V2<"sust.b.a1d.v2.b8." # op, B16>;
4614:   defm SUST_B_1D_ARRAY_V2I16_ # op_upper : SUST_1D_ARRAY_V2<"sust.b.a1d.v2.b16." # op, B16>;
4615:   defm SUST_B_1D_ARRAY_V2I32_ # op_upper : SUST_1D_ARRAY_V2<"sust.b.a1d.v2.b32." # op, B32>;
4616:   defm SUST_B_1D_ARRAY_V2I64_ # op_upper : SUST_1D_ARRAY_V2<"sust.b.a1d.v2.b64." # op, B64>;
4617: }
4618:
4619: defm SUST_P_1D_ARRAY_V2I8_TRAP : SUST_1D_ARRAY_V2<"sust.p.a1d.v2.b8.trap", B16>;
4620: defm SUST_P_1D_ARRAY_V2I16_TRAP : SUST_1D_ARRAY_V2<"sust.p.a1d.v2.b16.trap", B16>;
```
- EN: This range uses TableGen DSL to describe records such as ISTYPEP_SAMPLER, ISTYPEP_SURFACE, ISTYPEP_TEXTURE, SUST_1D_base; LLVM later expands these declarations into generated backend tables and helper code.
- CN: 这一段使用 TableGen DSL 描述 ISTYPEP_SAMPLER、ISTYPEP_SURFACE、ISTYPEP_TEXTURE、SUST_1D_base 等记录；LLVM 后续会把这些声明展开为生成的后端表和辅助代码。

### Lines 4621-4760
```tablegen
4621: defm SUST_P_1D_ARRAY_V2I32_TRAP : SUST_1D_ARRAY_V2<"sust.p.a1d.v2.b32.trap", B32>;
4622:
4623: class SUST_1D_ARRAY_V4_base<string inst, NVPTXRegClass intype, dag surf, list<dag> pat>
4624:     : NVPTXInst<(outs),
4625:                 !con(surf, (ins B32:$idx, B32:$x,
4626:                                 intype:$r, intype:$g, intype:$b, intype:$a)),
4627:                 inst # " \t[$s, \\{$idx, $x\\}], \\{$r, $g, $b, $a\\};",
4628:                 pat>;
4629: multiclass SUST_1D_ARRAY_V4<string inst, NVPTXRegClass intype> {
4630:   defvar intr = !cast<Intrinsic>("int_nvvm_" # !tolower(NAME));
4631:   def _R : SUST_1D_ARRAY_V4_base<inst, intype, (ins B64:$s),
4632:               [(intr B64:$s, B32:$idx, B32:$x,
4633:                 intype:$r, intype:$g, intype:$b, intype:$a)]>;
4634:   def _I : SUST_1D_ARRAY_V4_base<inst, intype, (ins i64imm:$s), []>;
4635: }
4636:
4637: foreach op = ["clamp", "trap", "zero"] in {
4638:   defvar op_upper = !toupper(op);
4639:   defm SUST_B_1D_ARRAY_V4I8_ # op_upper : SUST_1D_ARRAY_V4<"sust.b.a1d.v4.b8." # op, B16>;
4640:   defm SUST_B_1D_ARRAY_V4I16_ # op_upper : SUST_1D_ARRAY_V4<"sust.b.a1d.v4.b16." # op, B16>;
4641:   defm SUST_B_1D_ARRAY_V4I32_ # op_upper : SUST_1D_ARRAY_V4<"sust.b.a1d.v4.b32." # op, B32>;
4642: }
4643:
4644: defm SUST_P_1D_ARRAY_V4I8_TRAP : SUST_1D_ARRAY_V4<"sust.p.a1d.v4.b8.trap", B16>;
4645: defm SUST_P_1D_ARRAY_V4I16_TRAP : SUST_1D_ARRAY_V4<"sust.p.a1d.v4.b16.trap", B16>;
4646: defm SUST_P_1D_ARRAY_V4I32_TRAP : SUST_1D_ARRAY_V4<"sust.p.a1d.v4.b32.trap", B32>;
4647:
4648: class SUST_2D_base<string inst, NVPTXRegClass intype, dag surf, list<dag> pat>
4649:     : NVPTXInst<(outs),
4650:                 !con(surf, (ins B32:$x, B32:$y, intype:$r)),
4651:                 inst # " \t[$s, \\{$x, $y\\}], \\{$r\\};",
4652:                 pat>;
4653: multiclass SUST_2D<string inst, NVPTXRegClass intype> {
4654:   defvar intr = !cast<Intrinsic>("int_nvvm_" # !tolower(NAME));
4655:   def _R : SUST_2D_base<inst, intype, (ins B64:$s),
4656:               [(intr B64:$s, B32:$x, B32:$y, intype:$r)]>;
4657:   def _I : SUST_2D_base<inst, intype, (ins i64imm:$s), []>;
4658: }
4659:
4660: foreach op = ["clamp", "trap", "zero"] in {
4661:   defvar op_upper = !toupper(op);
4662:   defm SUST_B_2D_I8_ # op_upper : SUST_2D<"sust.b.2d.b8." # op, B16>;
4663:   defm SUST_B_2D_I16_ # op_upper : SUST_2D<"sust.b.2d.b16." # op, B16>;
4664:   defm SUST_B_2D_I32_ # op_upper : SUST_2D<"sust.b.2d.b32." # op, B32>;
4665:   defm SUST_B_2D_I64_ # op_upper : SUST_2D<"sust.b.2d.b64." # op, B64>;
4666: }
4667:
4668: defm SUST_P_2D_I8_TRAP : SUST_2D<"sust.p.2d.b8.trap", B16>;
4669: defm SUST_P_2D_I16_TRAP : SUST_2D<"sust.p.2d.b16.trap", B16>;
4670: defm SUST_P_2D_I32_TRAP : SUST_2D<"sust.p.2d.b32.trap", B32>;
4671:
4672: class SUST_2D_V2_base<string inst, NVPTXRegClass intype, dag surf, list<dag> pat>
4673:     : NVPTXInst<(outs),
4674:                 !con(surf, (ins B32:$x, B32:$y,
4675:                                 intype:$r, intype:$g)),
4676:                 inst # " \t[$s, \\{$x, $y\\}], \\{$r, $g\\};",
4677:                 pat>;
4678: multiclass SUST_2D_V2<string inst, NVPTXRegClass intype> {
4679:   defvar intr = !cast<Intrinsic>("int_nvvm_" # !tolower(NAME));
4680:   def _R : SUST_2D_V2_base<inst, intype, (ins B64:$s),
4681:               [(intr B64:$s, B32:$x, B32:$y,
4682:                 intype:$r, intype:$g)]>;
4683:   def _I : SUST_2D_V2_base<inst, intype, (ins i64imm:$s), []>;
4684: }
4685:
4686: foreach op = ["clamp", "trap", "zero"] in {
4687:   defvar op_upper = !toupper(op);
4688:   defm SUST_B_2D_V2I8_ # op_upper : SUST_2D_V2<"sust.b.2d.v2.b8." # op, B16>;
4689:   defm SUST_B_2D_V2I16_ # op_upper : SUST_2D_V2<"sust.b.2d.v2.b16." # op, B16>;
4690:   defm SUST_B_2D_V2I32_ # op_upper : SUST_2D_V2<"sust.b.2d.v2.b32." # op, B32>;
4691:   defm SUST_B_2D_V2I64_ # op_upper : SUST_2D_V2<"sust.b.2d.v2.b64." # op, B64>;
4692: }
4693:
4694: defm SUST_P_2D_V2I8_TRAP : SUST_2D_V2<"sust.p.2d.v2.b8.trap", B16>;
4695: defm SUST_P_2D_V2I16_TRAP : SUST_2D_V2<"sust.p.2d.v2.b16.trap", B16>;
4696: defm SUST_P_2D_V2I32_TRAP : SUST_2D_V2<"sust.p.2d.v2.b32.trap", B32>;
4697:
4698: class SUST_2D_V4_base<string inst, NVPTXRegClass intype, dag surf, list<dag> pat>
4699:     : NVPTXInst<(outs),
4700:                 !con(surf, (ins B32:$x, B32:$y,
4701:                                 intype:$r, intype:$g, intype:$b, intype:$a)),
4702:                 inst # " \t[$s, \\{$x, $y\\}], \\{$r, $g, $b, $a\\};",
4703:                 pat>;
4704: multiclass SUST_2D_V4<string inst, NVPTXRegClass intype> {
4705:   defvar intr = !cast<Intrinsic>("int_nvvm_" # !tolower(NAME));
4706:   def _R : SUST_2D_V4_base<inst, intype, (ins B64:$s),
4707:               [(intr B64:$s, B32:$x, B32:$y,
4708:                 intype:$r, intype:$g, intype:$b, intype:$a)]>;
4709:   def _I : SUST_2D_V4_base<inst, intype, (ins i64imm:$s), []>;
4710: }
4711:
4712: foreach op = ["clamp", "trap", "zero"] in {
4713:   defvar op_upper = !toupper(op);
4714:   defm SUST_B_2D_V4I8_ # op_upper : SUST_2D_V4<"sust.b.2d.v4.b8." # op, B16>;
4715:   defm SUST_B_2D_V4I16_ # op_upper : SUST_2D_V4<"sust.b.2d.v4.b16." # op, B16>;
4716:   defm SUST_B_2D_V4I32_ # op_upper : SUST_2D_V4<"sust.b.2d.v4.b32." # op, B32>;
4717: }
4718:
4719: defm SUST_P_2D_V4I8_TRAP : SUST_2D_V4<"sust.p.2d.v4.b8.trap", B16>;
4720: defm SUST_P_2D_V4I16_TRAP : SUST_2D_V4<"sust.p.2d.v4.b16.trap", B16>;
4721: defm SUST_P_2D_V4I32_TRAP : SUST_2D_V4<"sust.p.2d.v4.b32.trap", B32>;
4722:
4723: class SUST_2D_ARRAY_base<string inst, NVPTXRegClass intype, dag surf, list<dag> pat>
4724:     : NVPTXInst<(outs),
4725:                 !con(surf, (ins B32:$idx, B32:$x, B32:$y,
4726:                                 intype:$r)),
4727:                 inst # " \t[$s, \\{$idx, $x, $y, $y\\}], \\{$r\\};",
4728:                 pat>;
4729: multiclass SUST_2D_ARRAY<string inst, NVPTXRegClass intype> {
4730:   defvar intr = !cast<Intrinsic>("int_nvvm_" # !tolower(NAME));
4731:   def _R : SUST_2D_ARRAY_base<inst, intype, (ins B64:$s),
4732:               [(intr B64:$s, B32:$idx, B32:$x, B32:$y,
4733:                 intype:$r)]>;
4734:   def _I : SUST_2D_ARRAY_base<inst, intype, (ins i64imm:$s), []>;
4735: }
4736:
4737: foreach op = ["clamp", "trap", "zero"] in {
4738:   defvar op_upper = !toupper(op);
4739:   defm SUST_B_2D_ARRAY_I8_ # op_upper : SUST_2D_ARRAY<"sust.b.a2d.b8." # op, B16>;
4740:   defm SUST_B_2D_ARRAY_I16_ # op_upper : SUST_2D_ARRAY<"sust.b.a2d.b16." # op, B16>;
4741:   defm SUST_B_2D_ARRAY_I32_ # op_upper : SUST_2D_ARRAY<"sust.b.a2d.b32." # op, B32>;
4742:   defm SUST_B_2D_ARRAY_I64_ # op_upper : SUST_2D_ARRAY<"sust.b.a2d.b64." # op, B64>;
4743: }
4744:
4745: defm SUST_P_2D_ARRAY_I8_TRAP : SUST_2D_ARRAY<"sust.p.a2d.b8.trap", B16>;
4746: defm SUST_P_2D_ARRAY_I16_TRAP : SUST_2D_ARRAY<"sust.p.a2d.b16.trap", B16>;
4747: defm SUST_P_2D_ARRAY_I32_TRAP : SUST_2D_ARRAY<"sust.p.a2d.b32.trap", B32>;
4748:
4749: class SUST_2D_ARRAY_V2_base<string inst, NVPTXRegClass intype, dag surf, list<dag> pat>
4750:     : NVPTXInst<(outs),
4751:                 !con(surf, (ins B32:$idx, B32:$x, B32:$y,
4752:                                 intype:$r, intype:$g)),
4753:                 inst # " \t[$s, \\{$idx, $x, $y, $y\\}], \\{$r, $g\\};",
4754:                 pat>;
4755: multiclass SUST_2D_ARRAY_V2<string inst, NVPTXRegClass intype> {
4756:   defvar intr = !cast<Intrinsic>("int_nvvm_" # !tolower(NAME));
4757:   def _R : SUST_2D_ARRAY_V2_base<inst, intype, (ins B64:$s),
4758:               [(intr B64:$s, B32:$idx, B32:$x, B32:$y,
4759:                 intype:$r, intype:$g)]>;
4760:   def _I : SUST_2D_ARRAY_V2_base<inst, intype, (ins i64imm:$s), []>;
```
- EN: This range uses TableGen DSL to describe records such as SUST_P_1D_ARRAY_V2I32_TRAP, SUST_1D_ARRAY_V4_base, SUST_1D_ARRAY_V4, _R; LLVM later expands these declarations into generated backend tables and helper code.
- CN: 这一段使用 TableGen DSL 描述 SUST_P_1D_ARRAY_V2I32_TRAP、SUST_1D_ARRAY_V4_base、SUST_1D_ARRAY_V4、_R 等记录；LLVM 后续会把这些声明展开为生成的后端表和辅助代码。

### Lines 4761-4900
```tablegen
4761: }
4762:
4763: foreach op = ["clamp", "trap", "zero"] in {
4764:   defvar op_upper = !toupper(op);
4765:   defm SUST_B_2D_ARRAY_V2I8_ # op_upper : SUST_2D_ARRAY_V2<"sust.b.a2d.v2.b8." # op, B16>;
4766:   defm SUST_B_2D_ARRAY_V2I16_ # op_upper : SUST_2D_ARRAY_V2<"sust.b.a2d.v2.b16." # op, B16>;
4767:   defm SUST_B_2D_ARRAY_V2I32_ # op_upper : SUST_2D_ARRAY_V2<"sust.b.a2d.v2.b32." # op, B32>;
4768:   defm SUST_B_2D_ARRAY_V2I64_ # op_upper : SUST_2D_ARRAY_V2<"sust.b.a2d.v2.b64." # op, B64>;
4769: }
4770:
4771: defm SUST_P_2D_ARRAY_V2I8_TRAP : SUST_2D_ARRAY_V2<"sust.p.a2d.v2.b8.trap", B16>;
4772: defm SUST_P_2D_ARRAY_V2I16_TRAP : SUST_2D_ARRAY_V2<"sust.p.a2d.v2.b16.trap", B16>;
4773: defm SUST_P_2D_ARRAY_V2I32_TRAP : SUST_2D_ARRAY_V2<"sust.p.a2d.v2.b32.trap", B32>;
4774:
4775: class SUST_2D_ARRAY_V4_base<string inst, NVPTXRegClass intype, dag surf, list<dag> pat>
4776:     : NVPTXInst<(outs),
4777:                 !con(surf, (ins B32:$idx, B32:$x, B32:$y,
4778:                                 intype:$r, intype:$g, intype:$b, intype:$a)),
4779:                 inst # " \t[$s, \\{$idx, $x, $y, $y\\}], \\{$r, $g, $b, $a\\};",
4780:                 pat>;
4781: multiclass SUST_2D_ARRAY_V4<string inst, NVPTXRegClass intype> {
4782:   defvar intr = !cast<Intrinsic>("int_nvvm_" # !tolower(NAME));
4783:   def _R : SUST_2D_ARRAY_V4_base<inst, intype, (ins B64:$s),
4784:               [(intr B64:$s, B32:$idx, B32:$x, B32:$y,
4785:                 intype:$r, intype:$g, intype:$b, intype:$a)]>;
4786:   def _I : SUST_2D_ARRAY_V4_base<inst, intype, (ins i64imm:$s), []>;
4787: }
4788:
4789: foreach op = ["clamp", "trap", "zero"] in {
4790:   defvar op_upper = !toupper(op);
4791:   defm SUST_B_2D_ARRAY_V4I8_ # op_upper : SUST_2D_ARRAY_V4<"sust.b.a2d.v4.b8." # op, B16>;
4792:   defm SUST_B_2D_ARRAY_V4I16_ # op_upper : SUST_2D_ARRAY_V4<"sust.b.a2d.v4.b16." # op, B16>;
4793:   defm SUST_B_2D_ARRAY_V4I32_ # op_upper : SUST_2D_ARRAY_V4<"sust.b.a2d.v4.b32." # op, B32>;
4794: }
4795:
4796: defm SUST_P_2D_ARRAY_V4I8_TRAP : SUST_2D_ARRAY_V4<"sust.p.a2d.v4.b8.trap", B16>;
4797: defm SUST_P_2D_ARRAY_V4I16_TRAP : SUST_2D_ARRAY_V4<"sust.p.a2d.v4.b16.trap", B16>;
4798: defm SUST_P_2D_ARRAY_V4I32_TRAP : SUST_2D_ARRAY_V4<"sust.p.a2d.v4.b32.trap", B32>;
4799:
4800: class SUST_3D_base<string inst, NVPTXRegClass intype, dag surf, list<dag> pat>
4801:     : NVPTXInst<(outs),
4802:                 !con(surf, (ins B32:$x, B32:$y, B32:$z,
4803:                                 intype:$r)),
4804:                 inst # " \t[$s, \\{$x, $y, $z, $z\\}], \\{$r\\};",
4805:                 pat>;
4806: multiclass SUST_3D<string inst, NVPTXRegClass intype> {
4807:   defvar intr = !cast<Intrinsic>("int_nvvm_" # !tolower(NAME));
4808:   def _R : SUST_3D_base<inst, intype, (ins B64:$s),
4809:               [(intr B64:$s, B32:$x, B32:$y, B32:$z,
4810:                 intype:$r)]>;
4811:   def _I : SUST_3D_base<inst, intype, (ins i64imm:$s), []>;
4812: }
4813:
4814: foreach op = ["clamp", "trap", "zero"] in {
4815:   defvar op_upper = !toupper(op);
4816:   defm SUST_B_3D_I8_ # op_upper : SUST_3D<"sust.b.3d.b8." # op, B16>;
4817:   defm SUST_B_3D_I16_ # op_upper : SUST_3D<"sust.b.3d.b16." # op, B16>;
4818:   defm SUST_B_3D_I32_ # op_upper : SUST_3D<"sust.b.3d.b32." # op, B32>;
4819:   defm SUST_B_3D_I64_ # op_upper : SUST_3D<"sust.b.3d.b64." # op, B64>;
4820: }
4821: defm SUST_P_3D_I8_TRAP : SUST_3D<"sust.p.3d.b8.trap", B16>;
4822: defm SUST_P_3D_I16_TRAP : SUST_3D<"sust.p.3d.b16.trap", B16>;
4823: defm SUST_P_3D_I32_TRAP : SUST_3D<"sust.p.3d.b32.trap", B32>;
4824:
4825: class SUST_3D_V2_base<string inst, NVPTXRegClass intype, dag surf, list<dag> pat>
4826:     : NVPTXInst<(outs),
4827:                 !con(surf, (ins B32:$x, B32:$y, B32:$z,
4828:                                 intype:$r, intype:$g)),
4829:                 inst # " \t[$s, \\{$x, $y, $z, $z\\}], \\{$r, $g\\};",
4830:                 pat>;
4831: multiclass SUST_3D_V2<string inst, NVPTXRegClass intype> {
4832:   defvar intr = !cast<Intrinsic>("int_nvvm_" # !tolower(NAME));
4833:   def _R : SUST_3D_V2_base<inst, intype, (ins B64:$s),
4834:               [(intr B64:$s, B32:$x, B32:$y, B32:$z,
4835:                 intype:$r, intype:$g)]>;
4836:   def _I : SUST_3D_V2_base<inst, intype, (ins i64imm:$s), []>;
4837: }
4838:
4839: foreach op = ["clamp", "trap", "zero"] in {
4840:   defvar op_upper = !toupper(op);
4841:   defm SUST_B_3D_V2I8_ # op_upper : SUST_3D_V2<"sust.b.3d.v2.b8." # op, B16>;
4842:   defm SUST_B_3D_V2I16_ # op_upper : SUST_3D_V2<"sust.b.3d.v2.b16." # op, B16>;
4843:   defm SUST_B_3D_V2I32_ # op_upper : SUST_3D_V2<"sust.b.3d.v2.b32." # op, B32>;
4844:   defm SUST_B_3D_V2I64_ # op_upper : SUST_3D_V2<"sust.b.3d.v2.b64." # op, B64>;
4845: }
4846: defm SUST_P_3D_V2I8_TRAP : SUST_3D_V2<"sust.p.3d.v2.b8.trap", B16>;
4847: defm SUST_P_3D_V2I16_TRAP : SUST_3D_V2<"sust.p.3d.v2.b16.trap", B16>;
4848: defm SUST_P_3D_V2I32_TRAP : SUST_3D_V2<"sust.p.3d.v2.b32.trap", B32>;
4849:
4850: class SUST_3D_V4_base<string inst, NVPTXRegClass intype, dag surf, list<dag> pat>
4851:     : NVPTXInst<(outs),
4852:                 !con(surf, (ins B32:$x, B32:$y, B32:$z,
4853:                                 intype:$r, intype:$g, intype:$b, intype:$a)),
4854:                 inst # " \t[$s, \\{$x, $y, $z, $z\\}], \\{$r, $g, $b, $a\\};",
4855:                 pat>;
4856: multiclass SUST_3D_V4<string inst, NVPTXRegClass intype> {
4857:   defvar intr = !cast<Intrinsic>("int_nvvm_" # !tolower(NAME));
4858:   def _R : SUST_3D_V4_base<inst, intype, (ins B64:$s),
4859:               [(intr B64:$s, B32:$x, B32:$y, B32:$z,
4860:                 intype:$r, intype:$g, intype:$b, intype:$a)]>;
4861:   def _I : SUST_3D_V4_base<inst, intype, (ins i64imm:$s), []>;
4862: }
4863:
4864: foreach op = ["clamp", "trap", "zero"] in {
4865:   defvar op_upper = !toupper(op);
4866:   defm SUST_B_3D_V4I8_ # op_upper : SUST_3D_V4<"sust.b.3d.v4.b8." # op, B16>;
4867:   defm SUST_B_3D_V4I16_ # op_upper : SUST_3D_V4<"sust.b.3d.v4.b16." # op, B16>;
4868:   defm SUST_B_3D_V4I32_ # op_upper : SUST_3D_V4<"sust.b.3d.v4.b32." # op, B32>;
4869: }
4870:
4871: defm SUST_P_3D_V4I8_TRAP : SUST_3D_V4<"sust.p.3d.v4.b8.trap", B16>;
4872: defm SUST_P_3D_V4I16_TRAP : SUST_3D_V4<"sust.p.3d.v4.b16.trap", B16>;
4873: defm SUST_P_3D_V4I32_TRAP : SUST_3D_V4<"sust.p.3d.v4.b32.trap", B32>;
4874:
4875: }
4876:
4877:
4878: //-----------------------------------
4879: // Read Special Registers
4880: //-----------------------------------
4881:
4882: class PTX_READ_SREG_R64<string regname, Intrinsic intop, list<Predicate> Preds=[]>
4883:   : NVPTXInst<(outs B64:$d), (ins),
4884:               "mov.u64 \t$d, %" # regname # ";",
4885:               [(set i64:$d, (intop))]>,
4886:     Requires<Preds>;
4887:
4888: class PTX_READ_SREG_R32<string regname, Intrinsic intop, list<Predicate> Preds=[]>
4889:   : NVPTXInst<(outs B32:$d), (ins),
4890:               "mov.u32 \t$d, %" # regname # ";",
4891:               [(set i32:$d, (intop))]>,
4892:     Requires<Preds>;
4893:
4894: multiclass PTX_READ_SREG_R32V4<string regname, list<Predicate> Preds=[]> {
4895:    foreach suffix = ["x", "y", "z", "w"] in {
4896:       defvar reg = regname # "." # suffix;
4897:       defvar intr = !cast<Intrinsic>("int_nvvm_read_ptx_sreg_" # regname # "_" # suffix);
4898:       def "_"#suffix :  PTX_READ_SREG_R32<reg, intr, Preds>;
4899:    }
4900: }
```
- EN: This range uses TableGen DSL to describe records such as SUST_B_2D_ARRAY_V2I8_, SUST_B_2D_ARRAY_V2I16_, SUST_B_2D_ARRAY_V2I32_, SUST_B_2D_ARRAY_V2I64_; LLVM later expands these declarations into generated backend tables and helper code.
- CN: 这一段使用 TableGen DSL 描述 SUST_B_2D_ARRAY_V2I8_、SUST_B_2D_ARRAY_V2I16_、SUST_B_2D_ARRAY_V2I32_、SUST_B_2D_ARRAY_V2I64_ 等记录；LLVM 后续会把这些声明展开为生成的后端表和辅助代码。

### Lines 4901-5040
```tablegen
4901:
4902: // TODO Add read vector-version of special registers
4903:
4904: defm INT_PTX_SREG_TID   : PTX_READ_SREG_R32V4<"tid">;
4905: defm INT_PTX_SREG_NTID  : PTX_READ_SREG_R32V4<"ntid">;
4906: defm INT_PTX_SREG_CTAID : PTX_READ_SREG_R32V4<"ctaid">;
4907: defm INT_PTX_SREG_NCTAID: PTX_READ_SREG_R32V4<"nctaid">;
4908:
4909: defm INT_PTX_SREG_CLUSTERID :
4910:        PTX_READ_SREG_R32V4<"clusterid", [hasSM<90>, hasPTX<78>]>;
4911: defm INT_PTX_SREG_NCLUSTERID :
4912:        PTX_READ_SREG_R32V4<"nclusterid", [hasSM<90>, hasPTX<78>]>;
4913: defm INT_PTX_SREG_CLUSTER_CTAID :
4914:        PTX_READ_SREG_R32V4<"cluster_ctaid", [hasSM<90>, hasPTX<78>]>;
4915: defm INT_PTX_SREG_CLUSTER_NCTAID:
4916:        PTX_READ_SREG_R32V4<"cluster_nctaid", [hasSM<90>, hasPTX<78>]>;
4917:
4918: def  INT_PTX_SREG_CLUSTER_CTARANK :
4919:        PTX_READ_SREG_R32<"cluster_ctarank",
4920:                          int_nvvm_read_ptx_sreg_cluster_ctarank,
4921:                          [hasSM<90>, hasPTX<78>]>;
4922: def  INT_PTX_SREG_CLUSTER_NCTARANK:
4923:        PTX_READ_SREG_R32<"cluster_nctarank",
4924:                          int_nvvm_read_ptx_sreg_cluster_nctarank,
4925:                          [hasSM<90>, hasPTX<78>]>;
4926:
4927: def INT_PTX_SREG_TOTAL_SMEM_SIZE :
4928:     PTX_READ_SREG_R32<"total_smem_size", int_nvvm_read_ptx_sreg_total_smem_size>;
4929: def INT_PTX_SREG_DYNAMIC_SMEM_SIZE :
4930:     PTX_READ_SREG_R32<"dynamic_smem_size", int_nvvm_read_ptx_sreg_dynamic_smem_size>;
4931: def INT_PTX_SREG_AGGR_SMEM_SIZE :
4932:     PTX_READ_SREG_R32<"aggr_smem_size",
4933:                       int_nvvm_read_ptx_sreg_aggr_smem_size,
4934:                       [hasSM<90>, hasPTX<81>]>;
4935:
4936: def SREG_LANEID : PTX_READ_SREG_R32<"laneid", int_nvvm_read_ptx_sreg_laneid>;
4937: def SREG_WARPID : PTX_READ_SREG_R32<"warpid", int_nvvm_read_ptx_sreg_warpid>;
4938: def SREG_NWARPID : PTX_READ_SREG_R32<"nwarpid", int_nvvm_read_ptx_sreg_nwarpid>;
4939: def SREG_SMID : PTX_READ_SREG_R32<"smid", int_nvvm_read_ptx_sreg_smid>;
4940: def SREG_NSMID : PTX_READ_SREG_R32<"nsmid", int_nvvm_read_ptx_sreg_nsmid>;
4941: def SREG_GRIDID : PTX_READ_SREG_R32<"gridid", int_nvvm_read_ptx_sreg_gridid>;
4942:
4943: def INT_PTX_SREG_LANEMASK_EQ :
4944:     PTX_READ_SREG_R32<"lanemask_eq", int_nvvm_read_ptx_sreg_lanemask_eq>;
4945: def INT_PTX_SREG_LANEMASK_LE :
4946:     PTX_READ_SREG_R32<"lanemask_le", int_nvvm_read_ptx_sreg_lanemask_le>;
4947: def INT_PTX_SREG_LANEMASK_LT :
4948:     PTX_READ_SREG_R32<"lanemask_lt", int_nvvm_read_ptx_sreg_lanemask_lt>;
4949: def INT_PTX_SREG_LANEMASK_GE :
4950:     PTX_READ_SREG_R32<"lanemask_ge", int_nvvm_read_ptx_sreg_lanemask_ge>;
4951: def INT_PTX_SREG_LANEMASK_GT :
4952:     PTX_READ_SREG_R32<"lanemask_gt", int_nvvm_read_ptx_sreg_lanemask_gt>;
4953:
4954: let hasSideEffects = 1 in {
4955:   def SREG_CLOCK : PTX_READ_SREG_R32<"clock", int_nvvm_read_ptx_sreg_clock>;
4956:   def SREG_CLOCK64 : PTX_READ_SREG_R64<"clock64", int_nvvm_read_ptx_sreg_clock64>;
4957:   def SREG_GLOBALTIMER : PTX_READ_SREG_R64<"globaltimer", int_nvvm_read_ptx_sreg_globaltimer>;
4958:   def SREG_GLOBALTIMER_LO : PTX_READ_SREG_R32<"globaltimer_lo", int_nvvm_read_ptx_sreg_globaltimer_lo>;
4959: }
4960:
4961: def: Pat <(i64 (readcyclecounter)), (SREG_CLOCK64)>;
4962: def: Pat <(i64 (readsteadycounter)), (SREG_GLOBALTIMER)>;
4963: def: Pat <(i32 (readsteadycounter)), (SREG_GLOBALTIMER_LO)>;
4964:
4965: def INT_PTX_SREG_PM0 : PTX_READ_SREG_R32<"pm0", int_nvvm_read_ptx_sreg_pm0>;
4966: def INT_PTX_SREG_PM1 : PTX_READ_SREG_R32<"pm1", int_nvvm_read_ptx_sreg_pm1>;
4967: def INT_PTX_SREG_PM2 : PTX_READ_SREG_R32<"pm2", int_nvvm_read_ptx_sreg_pm2>;
4968: def INT_PTX_SREG_PM3 : PTX_READ_SREG_R32<"pm3", int_nvvm_read_ptx_sreg_pm3>;
4969:
4970: // Reserved shared memory special register reads
4971: foreach suffix = ["begin", "end", "cap", "0", "1"] in {
4972:   defvar regname = "reserved_smem_offset_" # suffix;
4973:   defvar intr = !cast<Intrinsic>("int_nvvm_read_ptx_sreg_" # regname);
4974:   def "INT_PTX_SREG_RESERVED_SMEM_OFFSET_" # !toupper(suffix) :
4975:       PTX_READ_SREG_R32<regname, intr, [hasPTX<76>, hasSM<80>]>;
4976: }
4977:
4978: // TODO: It would be nice to use PTX_READ_SREG here, but it doesn't
4979: // handle the constant.
4980: def INT_PTX_SREG_WARPSIZE :
4981:     NVPTXInst<(outs B32:$dst), (ins), "mov.u32 \t$dst, WARP_SZ;",
4982:               [(set i32:$dst, (int_nvvm_read_ptx_sreg_warpsize))]>;
4983:
4984: // Helper class that represents a 'fragment' of an NVPTX *MMA instruction.
4985: // In addition to target-independent fields provided by WMMA_REGS, it adds
4986: // the fields commonly used to implement specific PTX instruction -- register
4987: // types and names, constraints, parts of assembly, etc.
4988: class WMMA_REGINFO<WMMA_REGS r, string op, string metadata = "",
4989:                    string kind = "", string stype = "", string scale = "">
4990:       : WMMA_REGS<r.geom, r.frag, r.ptx_elt_type,
4991:                   !or(!eq(op, "mma.sp"), !eq(op, "mma.sp.block_scale"))> {
4992:   // NVPTX register types used to carry fragment data.
4993:   NVPTXRegClass regclass = !cond(
4994:     !eq(ptx_elt_type, "e4m3") : B32,
4995:     !eq(ptx_elt_type, "e5m2") : B32,
4996:     !eq(ptx_elt_type, "e3m2") : B32,
4997:     !eq(ptx_elt_type, "e2m3") : B32,
4998:     !eq(ptx_elt_type, "e2m1") : B32,
4999:     !eq(ptx_elt_type, "f16") : B32,
5000:     !eq(ptx_elt_type, "f32") : B32,
5001:     !eq(ptx_elt_type, "f64") : B64,
5002:     !eq(ptx_elt_type, "bf16") : B32,
5003:     !eq(ptx_elt_type, "tf32") : B32,
5004:     !eq(ptx_elt_type, "s32") : B32,
5005:     !eq(ptx_elt_type, "b16") : B32,
5006:     !eq(ptx_elt_type, "b8") : B32,
5007:     !eq(ptx_elt_type, "b8x16.b6x16_p32") : B32,
5008:     !eq(ptx_elt_type, "b8x16.b4x16_p64") : B32,
5009:     !eq(ptx_elt_type, "s8") : B32,
5010:     !eq(ptx_elt_type, "u8") : B32,
5011:     !eq(ptx_elt_type, "s4") : B32,
5012:     !eq(ptx_elt_type, "u4") : B32,
5013:     !eq(ptx_elt_type, "b1") : B32);
5014:
5015:   // Instruction input/output arguments for the fragment.
5016:   list<NVPTXRegClass> ptx_regs = !listsplat(regclass, !size(regs));
5017:
5018:   // List of register names for the fragment -- ["ra0", "ra1",...]
5019:   list<string> reg_names = RegSeq<!size(ptx_regs), "r"#frag>.ret;
5020:
5021:   // Generates "{{$r0, $r1,.... $rN-1}}" for use in asm string construction.
5022:   string regstring = "{{$" # !interleave(reg_names, ", $") # "}}";
5023:
5024:   // Predicates for particular fragment variant. Technically those are
5025:   // per-instruction predicates, but currently all fragments that can be used in
5026:   // a given instruction are subject to the same constraints, so an instruction
5027:   // can use predicates from any of its fragments. If/when this is no
5028:   // longer the case, we can concat all per-fragment predicates to enforce that
5029:   // all fragments of the instruction are viable.
5030:   list<Predicate> Predicates = !cond(
5031:     !and(!eq(op, "mma.block_scale"),
5032:          !eq(kind, "mxf4nvf4"),
5033:          !eq(stype, "ue8m0"),
5034:          !eq(scale, ".scale_4x"))
5035:       : [callSubtarget<"hasMMAWithMXF4NVF4Scale4xE8M0">],
5036:
5037:     !and(!eq(op, "mma.sp.block_scale"),
5038:          !eq(kind, "mxf4nvf4"),
5039:          !eq(stype, "ue8m0"),
5040:          !eq(scale, ".scale_4x"))
```
- EN: This range uses TableGen DSL to describe records such as INT_PTX_SREG_TID, INT_PTX_SREG_NTID, INT_PTX_SREG_CTAID, INT_PTX_SREG_NCTAID; LLVM later expands these declarations into generated backend tables and helper code.
- CN: 这一段使用 TableGen DSL 描述 INT_PTX_SREG_TID、INT_PTX_SREG_NTID、INT_PTX_SREG_CTAID、INT_PTX_SREG_NCTAID 等记录；LLVM 后续会把这些声明展开为生成的后端表和辅助代码。

### Lines 5041-5180
```tablegen
5041:       : [callSubtarget<"hasMMASparseWithMXF4NVF4Scale4xE8M0">],
5042:
5043:     !and(!eq(op, "mma.sp.block_scale"),
5044:         !eq(kind, "mxf4nvf4"),
5045:         !eq(kind, "mxf4")) : [callSubtarget<"hasMMASparseBlockScaleF4">],
5046:
5047:     !or(!eq(op, "mma.block_scale"),
5048:         !eq(op, "mma.sp.block_scale")) : [callSubtarget<"hasMMABlockScale">],
5049:
5050:     !or(!eq(ptx_elt_type, "e3m2"),
5051:         !eq(ptx_elt_type, "e2m3"),
5052:         !eq(ptx_elt_type, "e2m1"),
5053:         !ne(kind, "")) : [callSubtarget<"hasMMABlockScale">],
5054:
5055:     !and(!or(!eq(ptx_elt_type,"e4m3"),
5056:              !eq(ptx_elt_type,"e5m2")),
5057:          !eq(geom, "m16n8k16")) : [hasSM<89>, hasPTX<87>],
5058:
5059:     !or(!eq(ptx_elt_type, "e4m3"),
5060:         !eq(ptx_elt_type, "e5m2")) : [hasSM<89>, hasPTX<84>],
5061:
5062:     !and(isSparse,
5063:          !ne(metadata, "sp")) : [hasSM<80>, hasPTX<85>],
5064:     isSparse : [hasSM<80>, hasPTX<71>],
5065:
5066:     // fp16 -> fp16/fp32 @ m16n16k16
5067:     !and(!eq(geom, "m16n16k16"),
5068:          !or(!eq(ptx_elt_type, "f16"),
5069:              !eq(ptx_elt_type, "f32"))) : [hasSM<70>, hasPTX<60>],
5070:
5071:     !and(!eq(geom, "m8n8k4"),
5072:          !eq(ptx_elt_type, "f64")) : [hasSM<80>, hasPTX<70>],
5073:
5074:     !and(!or(!eq(geom, "m16n8k4"),
5075:              !eq(geom, "m16n8k8"),
5076:              !eq(geom, "m16n8k16")),
5077:          !eq(ptx_elt_type, "f64")) : [hasSM<90>, hasPTX<78>],
5078:
5079:     // fp16 -> fp16/fp32 @ m8n32k16/m32n8k16
5080:     !and(!or(!eq(geom, "m8n32k16"),
5081:              !eq(geom, "m32n8k16")),
5082:          !or(!eq(ptx_elt_type, "f16"),
5083:              !eq(ptx_elt_type, "f32"))) : [hasSM<70>, hasPTX<61>],
5084:
5085:     // u8/s8 -> s32 @ m16n16k16/m8n32k16/m32n8k16
5086:     !and(!or(!eq(geom, "m16n16k16"),
5087:              !eq(geom, "m8n32k16"),
5088:              !eq(geom, "m32n8k16")),
5089:          !or(!eq(ptx_elt_type, "u8"),
5090:              !eq(ptx_elt_type, "s8"),
5091:              !eq(ptx_elt_type, "s32"))) : [hasSM<72>, hasPTX<63>],
5092:
5093:     !and(!or(!eq(geom, "m16n16k16"),
5094:              !eq(geom, "m8n32k16"),
5095:              !eq(geom, "m32n8k16")),
5096:          !eq(ptx_elt_type, "bf16")) : [hasSM<80>, hasPTX<70>],
5097:
5098:     !and(!eq(geom, "m16n16k8"),
5099:          !eq(ptx_elt_type, "tf32")) : [hasSM<80>, hasPTX<70>],
5100:
5101:     !and(!eq(geom, "m16n16k8"),
5102:          !eq(ptx_elt_type, "f32")) : [hasSM<80>, hasPTX<70>],
5103:
5104:     // b1 -> s32 @ m8n8k128(b1)
5105:     !and(!ne(op, "mma"),
5106:          !eq(geom, "m8n8k128")) : [hasSM<75>, hasPTX<63>],
5107:
5108:     // u4/s4 -> s32 @ m8n8k32 (u4/s4)
5109:     !and(!ne(op, "mma"),
5110:          !eq(geom, "m8n8k32")) : [hasSM<75>, hasPTX<63>],
5111:
5112:     !or(!eq(geom, "m16n8k8"),
5113:         !eq(geom, "m8n8k16")) : [hasSM<75>, hasPTX<65>],
5114:
5115:     !and(!ne(ptx_elt_type, "f64"),
5116:          !eq(geom, "m8n8k4")) : [hasSM<70>, hasPTX<64>],
5117:
5118:     // mma m8n8k32 requires higher PTX version
5119:     !and(!eq(op, "mma"),
5120:          !eq(geom, "m8n8k32")) : [hasSM<75>, hasPTX<65>],
5121:
5122:     !and(!eq(ptx_elt_type, "f64"),
5123:          !eq(geom, "m8n8k4")) : [hasSM<80>, hasPTX<70>],
5124:
5125:     !and(!eq(op, "mma"),
5126:          !or(!eq(geom, "m16n8k16"),
5127:              !eq(geom, "m16n8k4"),
5128:              !eq(geom, "m16n8k32"),
5129:              !eq(geom, "m16n8k64"),
5130:              !eq(geom, "m8n8k128"),
5131:              !eq(geom, "m16n8k128"),
5132:              !eq(geom, "m16n8k256"))) : [hasSM<80>, hasPTX<70>],
5133:
5134:     !and(!eq(op, "ldmatrix"),
5135:          !eq(ptx_elt_type, "b16"),
5136:          !eq(geom, "m8n8")) : [hasSM<75>, hasPTX<65>],
5137:
5138:     !and(!eq(op, "ldmatrix"),
5139:          !eq(ptx_elt_type, "b8"),
5140:          !eq(geom, "m16n16")) : [callSubtarget<"hasLdStmatrixBlackwellSupport">],
5141:
5142:     !and(!eq(op, "ldmatrix"),
5143:          !eq(ptx_elt_type, "b8x16.b6x16_p32"),
5144:          !eq(geom, "m16n16")) : [callSubtarget<"hasLdStmatrixBlackwellSupport">],
5145:
5146:     !and(!eq(op, "ldmatrix"),
5147:          !eq(ptx_elt_type, "b8x16.b4x16_p64"),
5148:          !eq(geom, "m16n16")) : [callSubtarget<"hasLdStmatrixBlackwellSupport">],
5149:
5150:     !and(!eq(op, "ldmatrix"),
5151:          !eq(ptx_elt_type, "b8x16.b6x16_p32"),
5152:          !eq(geom, "m8n16")) : [callSubtarget<"hasLdStmatrixBlackwellSupport">],
5153:
5154:     !and(!eq(op, "ldmatrix"),
5155:          !eq(ptx_elt_type, "b8x16.b4x16_p64"),
5156:          !eq(geom, "m8n16")) : [callSubtarget<"hasLdStmatrixBlackwellSupport">],
5157:
5158:     !and(!eq(op, "stmatrix"),!eq(ptx_elt_type, "b16"),
5159:          !eq(geom, "m8n8")) : [hasSM<90>, hasPTX<78>],
5160:
5161:     !and(!eq(op, "stmatrix"),
5162:          !eq(ptx_elt_type, "b8"),
5163:          !eq(geom, "m16n8")) : [callSubtarget<"hasLdStmatrixBlackwellSupport">]);
5164:
5165:   // template DAGs for instruction inputs/output.
5166:   dag Outs = !dag(outs, ptx_regs, reg_names);
5167:   dag Ins = !dag(ins, ptx_regs, reg_names);
5168: }
5169:
5170: // Convert dag of arguments into a dag to match given intrinsic.
5171: class BuildPatternI<Intrinsic Intr, dag Ins> {
5172:   // Build a dag pattern that matches the intrinsic call.
5173:   dag ret = !foreach(tmp, Ins,
5174:                           !subst(ADDR, addr,
5175:                           !subst(ins, Intr,
5176:                           !subst(i32imm, timm, tmp))));
5177: }
5178:
5179: // Same as above, but uses PatFrag instead of an Intrinsic.
5180: class BuildPatternPF<PatFrag Intr, dag Ins> {
```
- EN: This range uses TableGen DSL to describe records such as BuildPatternI, BuildPatternPF; LLVM later expands these declarations into generated backend tables and helper code.
- CN: 这一段使用 TableGen DSL 描述 BuildPatternI、BuildPatternPF 等记录；LLVM 后续会把这些声明展开为生成的后端表和辅助代码。

### Lines 5181-5320
```tablegen
5181:   // Build a dag pattern that matches the intrinsic call.
5182:   dag ret = !foreach(tmp, Ins,
5183:                           !subst(ADDR, addr,
5184:                           !subst(ins, Intr, tmp)));
5185: }
5186:
5187: // Common WMMA-related fields used for building patterns for all MMA instructions.
5188: class WMMA_INSTR<string _Intr, list<dag> _Args>
5189:   : NVPTXInst<(outs), (ins), "?", []> {
5190:   Intrinsic Intr = !cast<Intrinsic>(_Intr);
5191:   // Concatenate all arguments into a single dag.
5192:   dag Args = !foldl((ins), _Args, a, b, !con(a, b));
5193:   // Pre-build the pattern to match (intrinsic arg0, arg1, ...).
5194:   dag IntrinsicPattern = BuildPatternI<!cast<Intrinsic>(Intr), Args>.ret;
5195: }
5196:
5197: //
5198: // wmma.load.[a|b|c].sync.[row|col].m16n16k16[|.global|.shared].[f16|f32]
5199: //
5200:
5201: class WMMA_LOAD<WMMA_REGINFO Frag, string Layout, string Space, bit WithStride>
5202:   : WMMA_INSTR<WMMA_NAME_LDST<"load", Frag, Layout, WithStride>.record_name,
5203:                               [!con((ins ADDR:$src),
5204:                                     !if(WithStride, (ins B32:$ldm), (ins)))]>,
5205:     Requires<Frag.Predicates> {
5206:   // Load/store intrinsics are overloaded on pointer's address space.
5207:   // To match the right intrinsic, we need to build AS-constrained PatFrag.
5208:   // Operands is a dag equivalent in shape to Args, but using (ops node:$name, .....).
5209:   dag PFOperands = !if(WithStride, (ops node:$src, node:$ldm), (ops node:$src));
5210:   dag PFOperandsIntr = !if(WithStride, (Intr node:$src, node:$ldm), (Intr node:$src));
5211:   // Build PatFrag that only matches particular address space.
5212:   PatFrag IntrFrag = PatFrag<PFOperands,
5213:                              PFOperandsIntr,
5214:                              !cond(!eq(Space, ".shared"): AS_match.shared,
5215:                                    !eq(Space, ".global"): AS_match.global,
5216:                                    true: AS_match.generic)>;
5217:   // Build AS-constrained pattern.
5218:   let IntrinsicPattern = BuildPatternPF<IntrFrag, Args>.ret;
5219:
5220:   let OutOperandList = Frag.Outs;
5221:   let InOperandList = !con(Args, (ins MmaCode:$ptx));
5222:   let AsmString = "wmma.load."
5223:                   # Frag.frag
5224:                   # ".sync"
5225:                   # "${ptx:aligned}"
5226:                   # "." # Layout
5227:                   # "." # Frag.geom
5228:                   # Space
5229:                   # "." # Frag.ptx_elt_type # " \t"
5230:                   # Frag.regstring
5231:                   # ", [$src]"
5232:                   # !if(WithStride, ", $ldm", "")
5233:                   # ";";
5234: }
5235:
5236: //
5237: // wmma.store.d.sync.[row|col].m16n16k16[|.global|.shared].[f16|f32]
5238: //
5239: class WMMA_STORE_D<WMMA_REGINFO Frag, string Layout, string Space,
5240:                    bit WithStride>
5241:   : WMMA_INSTR<WMMA_NAME_LDST<"store", Frag, Layout, WithStride>.record_name,
5242:                [!con((ins ADDR:$dst),
5243:                      Frag.Ins,
5244:                      !if(WithStride, (ins B32:$ldm), (ins)))]>,
5245:     Requires<Frag.Predicates> {
5246:
5247:   // Load/store intrinsics are overloaded on pointer's address space.
5248:   // To match the right intrinsic, we need to build AS-constrained PatFrag.
5249:   // Operands is a dag equivalent in shape to Args, but using (ops node:$name, .....).
5250:   dag PFOperands = !con((ops node:$dst),
5251:                         !dag(ops, !listsplat(node, !size(Frag.regs)), Frag.reg_names),
5252:                         !if(WithStride, (ops node:$ldm), (ops)));
5253:   // Build PatFrag that only matches particular address space.
5254:   PatFrag IntrFrag = PatFrag<PFOperands,
5255:                              !foreach(tmp, PFOperands, !subst(ops, Intr, tmp)),
5256:                              !cond(!eq(Space, ".shared"): AS_match.shared,
5257:                                    !eq(Space, ".global"): AS_match.global,
5258:                                    true: AS_match.generic)>;
5259:   // Build AS-constrained pattern.
5260:   let IntrinsicPattern = BuildPatternPF<IntrFrag, Args>.ret;
5261:
5262:   let InOperandList  = !con(Args, (ins MmaCode:$ptx));
5263:   let OutOperandList = (outs);
5264:   let AsmString = "wmma.store.d.sync"
5265:                   # "${ptx:aligned}"
5266:                   # "." # Layout
5267:                   # "." # Frag.geom
5268:                   # Space
5269:                   # "." # Frag.ptx_elt_type
5270:                   # " \t[$dst],"
5271:                   # Frag.regstring
5272:                   # !if(WithStride, ", $ldm", "")
5273:                   # ";";
5274: }
5275:
5276: // Create all load/store variants
5277: defset list<WMMA_INSTR> MMA_LDSTs  = {
5278:   foreach layout = ["row", "col"] in {
5279:     foreach stride = [false, true] in {
5280:       foreach space = [".global", ".shared", ""] in {
5281:         foreach frag = NVVM_MMA_OPS.all_ld_ops in
5282:           if NVVM_WMMA_LDST_SUPPORTED<frag, layout>.ret then
5283:             def : WMMA_LOAD<WMMA_REGINFO<frag, "load">, layout, space, stride>;
5284:         foreach frag = NVVM_MMA_OPS.all_st_ops in
5285:           if NVVM_WMMA_LDST_SUPPORTED<frag, layout>.ret then
5286:             def : WMMA_STORE_D<WMMA_REGINFO<frag, "store">, layout, space, stride>;
5287:       } // space
5288:     } // stride
5289:   } // layout
5290: } // defset
5291:
5292: // B1 instruction variants need extra constraints.
5293: class MMA_OP_PREDICATES<WMMA_REGINFO FragA, string b1op> {
5294:   string Op = b1op;
5295:   WMMA_REGINFO Frag = FragA;
5296:   list<Predicate> ret = !listconcat(
5297:     FragA.Predicates,
5298:     !if(!eq(b1op, ".and.popc"), [hasSM<80>, hasPTX<71>], [])
5299:   );
5300: }
5301: // WMMA.MMA
5302: class WMMA_MMA<WMMA_REGINFO FragA, WMMA_REGINFO FragB,
5303:                WMMA_REGINFO FragC, WMMA_REGINFO FragD,
5304:                string ALayout, string BLayout, int Satfinite, string rnd, string b1op>
5305:   : WMMA_INSTR<WMMA_NAME<ALayout, BLayout, Satfinite, rnd, b1op, FragA, FragB, FragC, FragD>.record_name,
5306:                          [FragA.Ins, FragB.Ins, FragC.Ins]>,
5307:     // Requires does not seem to have effect on Instruction w/o Patterns.
5308:     // We set it here anyways and propagate to the Pat<> we construct below.
5309:     Requires<MMA_OP_PREDICATES<FragA, b1op>.ret> {
5310:   let OutOperandList = FragD.Outs;
5311:   let InOperandList  = !con(Args, (ins MmaCode:$ptx));
5312:   string TypeList = !cond(
5313:     !eq(FragA.ptx_elt_type, "f16") : "." # FragD.ptx_elt_type
5314:                                      # "." # FragC.ptx_elt_type,
5315:     1: "." # FragD.ptx_elt_type
5316:        # "." # FragA.ptx_elt_type
5317:        # "." # FragB.ptx_elt_type
5318:        # "." # FragC.ptx_elt_type,
5319:   );
5320:   let AsmString = "wmma.mma"
```
- EN: This range uses TableGen DSL to describe records such as WMMA_INSTR, WMMA_LOAD, WMMA_STORE_D, MMA_OP_PREDICATES; LLVM later expands these declarations into generated backend tables and helper code.
- CN: 这一段使用 TableGen DSL 描述 WMMA_INSTR、WMMA_LOAD、WMMA_STORE_D、MMA_OP_PREDICATES 等记录；LLVM 后续会把这些声明展开为生成的后端表和辅助代码。

### Lines 5321-5460
```tablegen
5321:                   # b1op
5322:                   # ".sync"
5323:                   # "${ptx:aligned}"
5324:                   # "." # ALayout
5325:                   # "." # BLayout
5326:                   # "." # FragA.geom
5327:                   # !if(!ne(rnd, ""), !strconcat(".", rnd), "")
5328:                   # TypeList
5329:                   # !if(Satfinite, ".satfinite", "") # "\n\t\t"
5330:                   # FragD.regstring # ",\n\t\t"
5331:                   # FragA.regstring # ",\n\t\t"
5332:                   # FragB.regstring # ",\n\t\t"
5333:                   # FragC.regstring # ";";
5334: }
5335:
5336: let isConvergent = true in {
5337: defset list<WMMA_INSTR> WMMAs  = {
5338:   foreach layout_a = ["row", "col"] in {
5339:     foreach layout_b = ["row", "col"] in {
5340:       foreach satf = [0, 1] in {
5341:         foreach rnd = ["", "rn", "rz", "rm", "rp"] in {
5342:           foreach op = NVVM_MMA_OPS.all_wmma_ops in {
5343:             foreach b1op = NVVM_MMA_B1OPS<op>.ret in {
5344:               if NVVM_WMMA_SUPPORTED<op, layout_a, layout_b, satf, rnd>.ret then {
5345:                 def : WMMA_MMA<WMMA_REGINFO<op[0], "wmma.mma">,
5346:                               WMMA_REGINFO<op[1], "wmma.mma">,
5347:                               WMMA_REGINFO<op[2], "wmma.mma">,
5348:                               WMMA_REGINFO<op[3], "wmma.mma">,
5349:                               layout_a, layout_b, satf, rnd, b1op>;
5350:               }
5351:             } // b1op
5352:           } // op
5353:         } // rnd
5354:       } // satf
5355:     } // layout_b
5356:   } // layout_a
5357: } // defset
5358: }
5359:
5360: // MMA
5361: class MMA<WMMA_REGINFO FragA, WMMA_REGINFO FragB,
5362:                WMMA_REGINFO FragC, WMMA_REGINFO FragD,
5363:                string ALayout, string BLayout, int Satfinite, string b1op, string Kind>
5364:   : WMMA_INSTR<MMA_NAME<ALayout, BLayout, Satfinite, b1op, Kind, FragA, FragB, FragC, FragD>.record_name,
5365:                         [FragA.Ins, FragB.Ins, FragC.Ins]>,
5366:     // Requires does not seem to have effect on Instruction w/o Patterns.
5367:     // We set it here anyways and propagate to the Pat<> we construct below.
5368:   Requires<MMA_OP_PREDICATES<FragA, b1op>.ret> {
5369:   let OutOperandList = FragD.Outs;
5370:   let InOperandList  = !con(Args, (ins MmaCode:$ptx));
5371:   string TypeList = "." # FragD.ptx_elt_type
5372:                     # "." # FragA.ptx_elt_type
5373:                     # "." # FragB.ptx_elt_type
5374:                     # "." # FragC.ptx_elt_type;
5375:   let AsmString = "mma.sync.aligned."
5376:                   # FragA.geom
5377:                   # "." # ALayout
5378:                   # "." # BLayout
5379:                   # !if(!ne(Kind, ""), "." # Kind, "")
5380:                   # !if(Satfinite, ".satfinite", "")
5381:                   # TypeList
5382:                   # b1op # "\n\t\t"
5383:                   # FragD.regstring # ",\n\t\t"
5384:                   # FragA.regstring # ",\n\t\t"
5385:                   # FragB.regstring # ",\n\t\t"
5386:                   # FragC.regstring # ";";
5387: }
5388:
5389: let isConvergent = true in {
5390: defset list<WMMA_INSTR> MMAs  = {
5391:   foreach layout_a = ["row", "col"] in {
5392:     foreach layout_b = ["row", "col"] in {
5393:       foreach satf = [0, 1] in {
5394:         foreach op = NVVM_MMA_OPS.all_mma_ops in {
5395:           foreach b1op = NVVM_MMA_B1OPS<op>.ret in {
5396:             foreach kind = ["", "kind::f8f6f4"] in {
5397:               if NVVM_MMA_SUPPORTED<op, layout_a, layout_b, kind, satf>.ret then {
5398:                 def : MMA<WMMA_REGINFO<op[0], "mma", "", kind>,
5399:                           WMMA_REGINFO<op[1], "mma", "", kind>,
5400:                           WMMA_REGINFO<op[2], "mma", "", kind>,
5401:                           WMMA_REGINFO<op[3], "mma", "", kind>,
5402:                           layout_a, layout_b, satf, b1op, kind>;
5403:               }
5404:             } // kind
5405:           } // b1op
5406:         } // op
5407:       } // satf
5408:     } // layout_b
5409:   } // layout_a
5410: } // defset
5411: }
5412:
5413: // MMA.block_scale
5414: class MMA_BLOCK_SCALE<WMMA_REGINFO FragA, WMMA_REGINFO FragB,
5415:                       WMMA_REGINFO FragC, WMMA_REGINFO FragD,
5416:                       string Kind, string SType, string ScaleVecSize>
5417:   : WMMA_INSTR<MMA_BLOCK_SCALE_NAME<Kind, SType, ScaleVecSize,
5418:                                     FragA, FragB, FragC, FragD>.record_name,
5419:                                     [FragA.Ins, FragB.Ins, FragC.Ins,
5420:                                      (ins B32:$scale_a, B16:$byte_id_a,
5421:                                           B16:$thread_id_a, B32:$scale_b,
5422:                                           B16:$byte_id_b, B16:$thread_id_b)]>,
5423:     // Requires does not seem to have effect on Instruction w/o Patterns.
5424:     // We set it here anyways and propagate to the Pat<> we construct below.
5425:   Requires<FragA.Predicates> {
5426:   let OutOperandList = FragD.Outs;
5427:   let InOperandList  = !con(Args, (ins MmaCode:$ptx));
5428:   string TypeList = !interleave([FragD.ptx_elt_type,
5429:                                  FragA.ptx_elt_type,
5430:                                  FragB.ptx_elt_type,
5431:                                  FragC.ptx_elt_type], ".");
5432:   string ScaleVecSizeStr = !cond(
5433:     !eq(ScaleVecSize, "") : "",
5434:     !eq(ScaleVecSize, ".scale_1x") : ".scale_vec::1X",
5435:     !eq(ScaleVecSize, ".scale_2x") : ".scale_vec::2X",
5436:     !eq(ScaleVecSize, ".scale_4x") : ".scale_vec::4X"
5437:   );
5438:   let AsmString = "mma.sync.aligned."
5439:                   # FragA.geom
5440:                   # ".row.col"
5441:                   # ".kind::" # Kind
5442:                   # ".block_scale"
5443:                   # ScaleVecSizeStr
5444:                   # "." # TypeList
5445:                   # "." # SType # " \n\t\t"
5446:                   # FragD.regstring # ",\n\t\t"
5447:                   # FragA.regstring # ",\n\t\t"
5448:                   # FragB.regstring # ",\n\t\t"
5449:                   # FragC.regstring # ",\n\t\t"
5450:                   # "$scale_a, {{$byte_id_a, $thread_id_a}}" # ",\n\t\t"
5451:                   # "$scale_b, {{$byte_id_b, $thread_id_b}};";
5452: }
5453:
5454: let isConvergent = true in {
5455: defset list<WMMA_INSTR> MMA_BLOCK_SCALEs  = {
5456:   foreach kind = ["mxf4", "mxf4nvf4", "mxf8f6f4"] in {
5457:     foreach scale_vec_size = ["", ".scale_1x", ".scale_2x", ".scale_4x"] in {
5458:       foreach stype = ["ue8m0", "ue4m3"] in {
5459:         foreach op = NVVM_MMA_OPS.all_mma_block_scale_ops in {
5460:           if NVVM_MMA_BLOCK_SCALE_SUPPORTED<op, kind, stype, scale_vec_size>.ret then {
```
- EN: This range uses TableGen DSL to describe records such as MMA, MMA_BLOCK_SCALE; LLVM later expands these declarations into generated backend tables and helper code.
- CN: 这一段使用 TableGen DSL 描述 MMA、MMA_BLOCK_SCALE 等记录；LLVM 后续会把这些声明展开为生成的后端表和辅助代码。

### Lines 5461-5600
```tablegen
5461:             def : MMA_BLOCK_SCALE<WMMA_REGINFO<op[0], "mma.block_scale", "", kind,
5462:                                                stype, scale_vec_size>,
5463:                                   WMMA_REGINFO<op[1], "mma.block_scale", "", kind,
5464:                                                stype, scale_vec_size>,
5465:                                   WMMA_REGINFO<op[2], "mma.block_scale", "", kind,
5466:                                                stype, scale_vec_size>,
5467:                                   WMMA_REGINFO<op[3], "mma.block_scale", "", kind,
5468:                                                stype, scale_vec_size>,
5469:                                   kind, stype, scale_vec_size>;
5470:           }
5471:         } // op
5472:       } // stype
5473:     } // scale_vec_size
5474:   } // kind
5475: } // defset
5476: }
5477:
5478: // MMA SP
5479: class MMA_SP<WMMA_REGINFO FragA, WMMA_REGINFO FragB,
5480:              WMMA_REGINFO FragC, WMMA_REGINFO FragD,
5481:              string Metadata, string Kind, int Satfinite>
5482:   : WMMA_INSTR<MMA_SP_NAME<Metadata, Kind, Satfinite,
5483:                            FragA, FragB, FragC, FragD>.record_name,
5484:                [FragA.Ins, FragB.Ins, FragC.Ins,
5485:                 (ins B32:$metadata, i32imm:$selector)]>,
5486:     // Requires does not seem to have effect on Instruction w/o Patterns.
5487:     // We set it here anyways and propagate to the Pat<> we construct below.
5488:     Requires<!listconcat(FragA.Predicates,
5489:                          FragB.Predicates,
5490:                          FragC.Predicates,
5491:                          FragD.Predicates)> {
5492:   let OutOperandList = FragD.Outs;
5493:   let InOperandList = !con(Args, (ins MmaCode:$ptx));
5494:   string TypeList = "." # FragD.ptx_elt_type
5495:                     # "." # FragA.ptx_elt_type
5496:                     # "." # FragB.ptx_elt_type
5497:                     # "." # FragC.ptx_elt_type;
5498:   let AsmString = "mma"
5499:                   # "." # Metadata
5500:                   # ".sync.aligned."
5501:                   # FragA.geom
5502:                   # ".row.col"
5503:                   # !if(!ne(Kind, ""), "." # Kind, "")
5504:                   # !if(Satfinite, ".satfinite", "")
5505:                   # TypeList # "\n\t\t"
5506:                   # FragD.regstring # ",\n\t\t"
5507:                   # FragA.regstring # ",\n\t\t"
5508:                   # FragB.regstring # ",\n\t\t"
5509:                   # FragC.regstring # ",\n\t\t"
5510:                   # "$metadata" # ",\n\t\t"
5511:                   # "$selector" # ";";
5512: }
5513:
5514: let isConvergent = true in {
5515: defset list<WMMA_INSTR> MMA_SPs = {
5516:   foreach metadata = ["sp", "sp::ordered_metadata"] in {
5517:     foreach kind = ["", "kind::f8f6f4"] in {
5518:       foreach satf = [0, 1] in {
5519:         foreach op = NVVM_MMA_OPS.all_mma_sp_ops in {
5520:           if NVVM_MMA_SP_SUPPORTED<op, metadata, kind, satf>.ret then {
5521:             def : MMA_SP<WMMA_REGINFO<op[0], "mma.sp", metadata, kind>,
5522:                          WMMA_REGINFO<op[1], "mma.sp", metadata, kind>,
5523:                          WMMA_REGINFO<op[2], "mma.sp", metadata, kind>,
5524:                          WMMA_REGINFO<op[3], "mma.sp", metadata, kind>,
5525:                          metadata, kind, satf>;
5526:           }
5527:         } // op
5528:       } // satf
5529:     } // kind
5530:   } // metadata
5531: } // defset
5532: }
5533:
5534: // MMA SP BLOCK SCALE
5535: class MMA_SP_BLOCK_SCALE<WMMA_REGINFO FragA, WMMA_REGINFO FragB,
5536:                          WMMA_REGINFO FragC, WMMA_REGINFO FragD,
5537:                          string Kind, string SType, string ScaleVecSize>
5538:   : WMMA_INSTR<MMA_SP_BLOCK_SCALE_NAME<Kind, SType, ScaleVecSize,
5539:                                        FragA, FragB, FragC, FragD>.record_name,
5540:                [FragA.Ins, FragB.Ins, FragC.Ins,
5541:                 (ins B32:$metadata, i32imm:$selector,
5542:                      B32:$scale_a, B16:$byte_id_a, B16:$thread_id_a,
5543:                      B32:$scale_b, B16:$byte_id_b, B16:$thread_id_b)]>,
5544:     // Requires does not seem to have effect on Instruction w/o Patterns.
5545:     // We set it here anyways and propagate to the Pat<> we construct below.
5546:     Requires<!listconcat(FragA.Predicates,
5547:                          FragB.Predicates,
5548:                          FragC.Predicates,
5549:                          FragD.Predicates)> {
5550:   let OutOperandList = FragD.Outs;
5551:   let InOperandList = !con(Args, (ins MmaCode:$ptx));
5552:   string TypeList = "." # FragD.ptx_elt_type
5553:                     # "." # FragA.ptx_elt_type
5554:                     # "." # FragB.ptx_elt_type
5555:                     # "." # FragC.ptx_elt_type;
5556:   string ScaleVecSizeStr = !cond(
5557:     !eq(ScaleVecSize, "") : "",
5558:     !eq(ScaleVecSize, ".scale_1x") : ".scale_vec::1X",
5559:     !eq(ScaleVecSize, ".scale_2x") : ".scale_vec::2X",
5560:     !eq(ScaleVecSize, ".scale_4x") : ".scale_vec::4X"
5561:   );
5562:   let AsmString = "mma.sp::ordered_metadata.sync.aligned."
5563:                   # FragA.geom
5564:                   # ".row.col"
5565:                   # ".kind::" # Kind
5566:                   # ".block_scale"
5567:                   # ScaleVecSizeStr
5568:                   # TypeList
5569:                   # "." # SType # "\n\t\t"
5570:                   # FragD.regstring # ",\n\t\t"
5571:                   # FragA.regstring # ",\n\t\t"
5572:                   # FragB.regstring # ",\n\t\t"
5573:                   # FragC.regstring # ",\n\t\t"
5574:                   # "$metadata" # ",\n\t\t"
5575:                   # "$selector" # ",\n\t\t"
5576:                   # "$scale_a, {{$byte_id_a, $thread_id_a}}" # ",\n\t\t"
5577:                   # "$scale_b, {{$byte_id_b, $thread_id_b}};";
5578: }
5579:
5580: let isConvergent = true in {
5581: defset list<WMMA_INSTR> MMA_SP_BLOCK_SCALEs = {
5582:   foreach kind = ["mxf4", "mxf4nvf4", "mxf8f6f4"] in {
5583:     foreach scale_vec_size = ["", ".scale_1x", ".scale_2x", ".scale_4x"] in {
5584:       foreach stype = ["ue8m0", "ue4m3"] in {
5585:       foreach op = NVVM_MMA_OPS.all_mma_sp_block_scale_ops in {
5586:         if NVVM_MMA_SP_BLOCK_SCALE_SUPPORTED<op, kind, stype, scale_vec_size>.ret then {
5587:           def : MMA_SP_BLOCK_SCALE<WMMA_REGINFO<op[0], "mma.sp.block_scale", "sp::ordered_metadata", kind,
5588:                                                 stype, scale_vec_size>,
5589:                        WMMA_REGINFO<op[1], "mma.sp.block_scale", "sp::ordered_metadata", kind,
5590:                                     stype, scale_vec_size>,
5591:                        WMMA_REGINFO<op[2], "mma.sp.block_scale", "sp::ordered_metadata", kind,
5592:                                     stype, scale_vec_size>,
5593:                        WMMA_REGINFO<op[3], "mma.sp.block_scale", "sp::ordered_metadata", kind,
5594:                                     stype, scale_vec_size>,
5595:                        kind, stype, scale_vec_size>;
5596:         }
5597:       } // op
5598:       } // stype
5599:     } // scale_vec_size
5600:   } // kind
```
- EN: This range uses TableGen DSL to describe records such as MMA_SP, MMA_SP_BLOCK_SCALE; LLVM later expands these declarations into generated backend tables and helper code.
- CN: 这一段使用 TableGen DSL 描述 MMA_SP、MMA_SP_BLOCK_SCALE 等记录；LLVM 后续会把这些声明展开为生成的后端表和辅助代码。

### Lines 5601-5740
```tablegen
5601: } // defset
5602: }
5603:
5604: //
5605: // ldmatrix.sync.aligned.m8n8[|.trans][|.shared].b16
5606: //
5607: class LDMATRIX<WMMA_REGINFO Frag, bit Transposed, string Space>
5608:   : WMMA_INSTR<LDMATRIX_NAME<Frag, Transposed>.record_name, [(ins ADDR:$src)]>,
5609:     Requires<Frag.Predicates> {
5610:   // Build PatFrag that only matches particular address space.
5611:   PatFrag IntrFrag = PatFrag<(ops node:$src), (Intr node:$src),
5612:                              !cond(!eq(Space, ".shared"): AS_match.shared,
5613:                                    true: AS_match.generic)>;
5614:   // Build AS-constrained pattern.
5615:   let IntrinsicPattern = BuildPatternPF<IntrFrag, Args>.ret;
5616:
5617:   let OutOperandList = Frag.Outs;
5618:   let InOperandList = !con(Args, (ins MmaCode:$ptx));
5619:   let AsmString = "ldmatrix.sync.aligned."
5620:                   # Frag.geom
5621:                   # "." # Frag.frag
5622:                   # !if(Transposed, ".trans", "")
5623:                   # Space
5624:                   # "." # Frag.ptx_elt_type
5625:                   # " " # Frag.regstring # ", [$src];";
5626: }
5627:
5628: // Create all ldmatrix variants
5629: defset list<WMMA_INSTR> LDMATRIXs  = {
5630:   foreach transposed = [false, true] in {
5631:     foreach space = [".shared", ""] in {
5632:       foreach frag = NVVM_MMA_OPS.all_ldmatrix_ops in
5633:         if NVVM_LDMATRIX_SUPPORTED<frag, transposed>.ret then
5634:           def : LDMATRIX<WMMA_REGINFO<frag, "ldmatrix">, transposed, space>;
5635:     } // space
5636:   } // transposed
5637: } // defset
5638:
5639: //
5640: // stmatrix.sync.aligned.m8n8[|.trans][|.shared].b16
5641: //
5642: class STMATRIX<WMMA_REGINFO Frag, bit Transposed, string Space>
5643:   : WMMA_INSTR<STMATRIX_NAME<Frag, Transposed>.record_name, [!con((ins ADDR:$dst), Frag.Ins)]>,
5644:     Requires<Frag.Predicates> {
5645:   // Build PatFrag that only matches particular address space.
5646:   dag PFOperands = !con((ops node:$dst),
5647:                         !dag(ops, !listsplat(node, !size(Frag.regs)), Frag.reg_names));
5648:   PatFrag IntrFrag = PatFrag<PFOperands,
5649:                              !foreach(tmp, PFOperands, !subst(ops, Intr, tmp)),
5650:                              !cond(!eq(Space, ".shared"): AS_match.shared,
5651:                                    true: AS_match.generic)>;
5652:   // Build AS-constrained pattern.
5653:   let IntrinsicPattern = BuildPatternPF<IntrFrag, Args>.ret;
5654:   let OutOperandList = (outs);
5655:   let InOperandList = !con(Args, (ins MmaCode:$ptx));
5656:   let AsmString = "stmatrix.sync.aligned."
5657:                   # Frag.geom
5658:                   # "." # Frag.frag
5659:                   # !if(Transposed, ".trans", "")
5660:                   # Space
5661:                   # "." # Frag.ptx_elt_type
5662:                   # " [$dst], " # Frag.regstring # ";";
5663: }
5664:
5665: // Create all stmatrix variants
5666: defset list<WMMA_INSTR> STMATRIXs = {
5667:   foreach transposed = [false, true] in {foreach space = [".shared", ""] in {
5668:       foreach frag = NVVM_MMA_OPS.all_stmatrix_ops in
5669:         if NVVM_STMATRIX_SUPPORTED<frag, transposed>.ret then
5670:           def : STMATRIX<WMMA_REGINFO<frag, "stmatrix">, transposed, space>;
5671:     } // space
5672:   } // transposed
5673: } // defset
5674:
5675: // movmatrix
5676: let isConvergent = true in {
5677: def MOVMATRIX_SYNC_ALIGNED_M8N8_TRANS_B16
5678:   : BasicNVPTXInst<(outs B32:$dst),
5679:               (ins B32:$src),
5680:               "movmatrix.sync.aligned.m8n8.trans.b16",
5681:               [(set B32:$dst,
5682:                 (int_nvvm_movmatrix_sync_aligned_m8n8_trans_b16 B32:$src))]>,
5683:     Requires<[hasSM<75>, hasPTX<78>]>;
5684: } // isConvergent = true
5685:
5686: // Constructing non-flat DAGs is still a pain. I can't !subst a dag node with a
5687: // dag, so the ptx.version must be appended *after* foreach replaces 'ins' with
5688: // the instruction record.
5689: class MMA_PAT<WMMA_INSTR wi>
5690:       : Pat<wi.IntrinsicPattern,
5691:             !con(!foreach(tmp, wi.Args, !subst(ins, wi, tmp)),
5692:                  (wi ptx.version))>,
5693:         Requires<wi.Predicates>;
5694:
5695: // Build intrinsic->instruction patterns for all MMA instructions.
5696: foreach mma = !listconcat(MMAs, MMA_BLOCK_SCALEs, WMMAs, MMA_LDSTs, LDMATRIXs,
5697:                           STMATRIXs, MMA_SPs, MMA_SP_BLOCK_SCALEs) in
5698:   def : MMA_PAT<mma>;
5699:
5700: multiclass MAPA<string suffix, Intrinsic Intr> {
5701:   let Predicates = [hasSM<90>, hasPTX<78>] in {
5702:     def _32: BasicNVPTXInst<(outs B32:$d), (ins B32:$a, B32:$b),
5703:                 "mapa" # suffix # ".u32",
5704:                 [(set i32:$d, (Intr i32:$a, i32:$b))]>;
5705:     def _32i: BasicNVPTXInst<(outs B32:$d), (ins B32:$a, i32imm:$b),
5706:                 "mapa" # suffix # ".u32",
5707:                 [(set i32:$d, (Intr i32:$a, imm:$b))]>;
5708:     def _64: BasicNVPTXInst<(outs B64:$d), (ins B64:$a, B32:$b),
5709:                 "mapa" # suffix # ".u64",
5710:                 [(set i64:$d, (Intr i64:$a, i32:$b))]>;
5711:     def _64i: BasicNVPTXInst<(outs B64:$d), (ins B64:$a, i32imm:$b),
5712:                 "mapa" # suffix # ".u64",
5713:                 [(set i64:$d, (Intr i64:$a, imm:$b))]>;
5714:   }
5715: }
5716:
5717:
5718: defm mapa  : MAPA<"", int_nvvm_mapa>;
5719: defm mapa_shared_cluster  : MAPA<".shared::cluster", int_nvvm_mapa_shared_cluster>;
5720:
5721:
5722: multiclass GETCTARANK<string suffix, Intrinsic Intr> {
5723:   let Predicates = [hasSM<90>, hasPTX<78>] in {
5724:     def _32: BasicNVPTXInst<(outs B32:$d), (ins B32:$a),
5725:                 "getctarank" # suffix # ".u32",
5726:                 [(set i32:$d, (Intr i32:$a))]>;
5727:     def _64: BasicNVPTXInst<(outs B32:$d), (ins B64:$a),
5728:                 "getctarank" # suffix # ".u64",
5729:                 [(set i32:$d, (Intr i64:$a))]>;
5730:   }
5731: }
5732:
5733: defm getctarank  : GETCTARANK<"", int_nvvm_getctarank>;
5734: defm getctarank_shared_cluster  : GETCTARANK<".shared::cluster", int_nvvm_getctarank_shared_cluster>;
5735:
5736: def is_explicit_cluster: NVPTXInst<(outs B1:$d), (ins),
5737:               "mov.pred\t$d, %is_explicit_cluster;",
5738:               [(set i1:$d, (int_nvvm_is_explicit_cluster))]>,
5739:     Requires<[hasSM<90>, hasPTX<78>]>;
5740:
```
- EN: This range uses TableGen DSL to describe records such as LDMATRIX, STMATRIX, MOVMATRIX_SYNC_ALIGNED_M8N8_TRANS_B16, MMA_PAT; LLVM later expands these declarations into generated backend tables and helper code.
- CN: 这一段使用 TableGen DSL 描述 LDMATRIX、STMATRIX、MOVMATRIX_SYNC_ALIGNED_M8N8_TRANS_B16、MMA_PAT 等记录；LLVM 后续会把这些声明展开为生成的后端表和辅助代码。

### Lines 5741-5880
```tablegen
5741: // setmaxnreg inc/dec intrinsics
5742: let isConvergent = true in {
5743: multiclass SET_MAXNREG<string Action, Intrinsic Intr> {
5744:   def : BasicNVPTXInst<(outs), (ins i32imm:$reg_count),
5745:           "setmaxnreg." # Action # ".sync.aligned.u32",
5746:           [(Intr timm:$reg_count)]>,
5747:     Requires<[callSubtarget<"hasSetMaxNRegSupport">]>;
5748: }
5749:
5750: defm INT_SET_MAXNREG_INC : SET_MAXNREG<"inc", int_nvvm_setmaxnreg_inc_sync_aligned_u32>;
5751: defm INT_SET_MAXNREG_DEC : SET_MAXNREG<"dec", int_nvvm_setmaxnreg_dec_sync_aligned_u32>;
5752:
5753: } // isConvergent
5754:
5755: //
5756: // WGMMA fence instructions
5757: //
5758: let isConvergent = true, Predicates = [hasSM90a, hasPTX<80>] in {
5759:   def WGMMA_FENCE_SYNC_ALIGNED : NullaryInst<"wgmma.fence.sync.aligned", int_nvvm_wgmma_fence_sync_aligned>;
5760:
5761:   def WGMMA_COMMIT_GROUP_SYNC_ALIGNED : NullaryInst<"wgmma.commit_group.sync.aligned", int_nvvm_wgmma_commit_group_sync_aligned>;
5762:
5763:   def WGMMA_WAIT_GROUP_SYNC_ALIGNED : BasicNVPTXInst<(outs), (ins i64imm:$n), "wgmma.wait_group.sync.aligned",
5764:                               [(int_nvvm_wgmma_wait_group_sync_aligned timm:$n)]>;
5765: }
5766:
5767: let Predicates = [hasSM<90>, hasPTX<78>] in {
5768:   def GRIDDEPCONTROL_LAUNCH_DEPENDENTS :
5769:         NullaryInst<"griddepcontrol.launch_dependents", int_nvvm_griddepcontrol_launch_dependents>;
5770:   def GRIDDEPCONTROL_WAIT :
5771:         NullaryInst<"griddepcontrol.wait", int_nvvm_griddepcontrol_wait>;
5772: }
5773:
5774: def EXIT : NullaryInst<"exit", int_nvvm_exit>;
5775:
5776: // Tcgen05 intrinsics
5777: let isConvergent = true in {
5778: let Predicates = [callSubtarget<"hasTcgen05InstSupport">] in {
5779: multiclass TCGEN05_ALLOC_INTR<string AS, string num, Intrinsic Intr> {
5780:   def "" : BasicNVPTXInst<(outs),
5781:              (ins ADDR:$dst, B32:$ncols),
5782:              "tcgen05.alloc.cta_group::" # num # ".sync.aligned" # AS # ".b32",
5783:              [(Intr addr:$dst, B32:$ncols)]>;
5784: }
5785:
5786: defm TCGEN05_ALLOC_CG1 : TCGEN05_ALLOC_INTR<"", "1", int_nvvm_tcgen05_alloc_cg1>;
5787: defm TCGEN05_ALLOC_CG2 : TCGEN05_ALLOC_INTR<"", "2", int_nvvm_tcgen05_alloc_cg2>;
5788:
5789: defm TCGEN05_ALLOC_S64_CG1 : TCGEN05_ALLOC_INTR<".shared::cta", "1", int_nvvm_tcgen05_alloc_shared_cg1>;
5790: defm TCGEN05_ALLOC_S64_CG2 : TCGEN05_ALLOC_INTR<".shared::cta", "2", int_nvvm_tcgen05_alloc_shared_cg2>;
5791:
5792: multiclass TCGEN05_DEALLOC_INTR<string num, Intrinsic Intr> {
5793:   def "" : BasicNVPTXInst<(outs),
5794:              (ins B32:$tmem_addr, B32:$ncols),
5795:              "tcgen05.dealloc.cta_group::" # num # ".sync.aligned.b32",
5796:              [(Intr B32:$tmem_addr, B32:$ncols)]>;
5797: }
5798: defm TCGEN05_DEALLOC_CG1: TCGEN05_DEALLOC_INTR<"1", int_nvvm_tcgen05_dealloc_cg1>;
5799: defm TCGEN05_DEALLOC_CG2: TCGEN05_DEALLOC_INTR<"2", int_nvvm_tcgen05_dealloc_cg2>;
5800:
5801: multiclass TCGEN05_RELINQ_PERMIT_INTR<string num, Intrinsic Intr> {
5802:   def "" : NullaryInst<"tcgen05.relinquish_alloc_permit.cta_group::" # num # ".sync.aligned", Intr>;
5803: }
5804: defm TCGEN05_RELINQ_CG1: TCGEN05_RELINQ_PERMIT_INTR<"1", int_nvvm_tcgen05_relinq_alloc_permit_cg1>;
5805: defm TCGEN05_RELINQ_CG2: TCGEN05_RELINQ_PERMIT_INTR<"2", int_nvvm_tcgen05_relinq_alloc_permit_cg2>;
5806:
5807: def tcgen05_wait_ld: NullaryInst<"tcgen05.wait::ld.sync.aligned", int_nvvm_tcgen05_wait_ld>;
5808: def tcgen05_wait_st: NullaryInst<"tcgen05.wait::st.sync.aligned", int_nvvm_tcgen05_wait_st>;
5809:
5810: multiclass TCGEN05_COMMIT_INTR<string AS, string num> {
5811:   defvar prefix = "tcgen05.commit.cta_group::" # num #".mbarrier::arrive::one.shared::cluster";
5812:
5813:   defvar intr_suffix = !if(!eq(AS, "shared"), "_shared", "") # "_cg" # num;
5814:   defvar Intr = !cast<Intrinsic>("int_nvvm_tcgen05_commit" # intr_suffix);
5815:   defvar IntrMC = !cast<Intrinsic>("int_nvvm_tcgen05_commit_mc" # intr_suffix);
5816:
5817:   def "" : BasicNVPTXInst<(outs), (ins ADDR:$mbar),
5818:              prefix # ".b64",
5819:              [(Intr addr:$mbar)]>;
5820:   def _MC : BasicNVPTXInst<(outs), (ins ADDR:$mbar, B16:$mc),
5821:                    prefix # ".multicast::cluster.b64",
5822:                    [(IntrMC addr:$mbar, B16:$mc)]>;
5823: }
5824:
5825: defm TCGEN05_COMMIT_CG1 : TCGEN05_COMMIT_INTR<"", "1">;
5826: defm TCGEN05_COMMIT_CG2 : TCGEN05_COMMIT_INTR<"", "2">;
5827: defm TCGEN05_COMMIT_S64_CG1 : TCGEN05_COMMIT_INTR<"shared", "1">;
5828: defm TCGEN05_COMMIT_S64_CG2 : TCGEN05_COMMIT_INTR<"shared", "2">;
5829:
5830: multiclass TCGEN05_CP_INTR<string shape, string src_fmt, string mc = ""> {
5831:   defvar dst_fmt = !if(!eq(src_fmt, ""), "", ".b8x16");
5832:   defvar fmt_asm = StrJoin<".", [dst_fmt, src_fmt]>.ret;
5833:   defvar fmt_intr = StrJoin<"_", [src_fmt]>.ret;
5834:
5835:   defvar shape_mc_asm = StrJoin<".", [shape, mc]>.ret;
5836:   defvar shape_mc_intr = !subst("::", "_", !subst(".", "_", shape_mc_asm));
5837:
5838:   defvar intr_prefix = StrJoin<"_", ["int_nvvm_tcgen05_cp", shape_mc_intr, fmt_intr]>.ret;
5839:   defvar IntrCG1 = !cast<Intrinsic>(intr_prefix # "_cg1");
5840:   defvar IntrCG2 = !cast<Intrinsic>(intr_prefix # "_cg2");
5841:
5842:   def _cg1 : BasicNVPTXInst<(outs),
5843:                     (ins ADDR:$tmem_addr, B64:$sdesc),
5844:                     "tcgen05.cp.cta_group::1." # shape_mc_asm # fmt_asm,
5845:                     [(IntrCG1 addr:$tmem_addr, B64:$sdesc)]>;
5846:   def _cg2 : BasicNVPTXInst<(outs),
5847:                     (ins ADDR:$tmem_addr, B64:$sdesc),
5848:                     "tcgen05.cp.cta_group::2." # shape_mc_asm # fmt_asm,
5849:                     [(IntrCG2 addr:$tmem_addr, B64:$sdesc)]>;
5850: }
5851:
5852: foreach src_fmt = ["", "b6x16_p32", "b4x16_p64"] in {
5853:   defm TCGEN05_CP_128x256b # src_fmt : TCGEN05_CP_INTR<"128x256b", src_fmt>;
5854:   defm TCGEN05_CP_4x256b # src_fmt   : TCGEN05_CP_INTR<"4x256b", src_fmt>;
5855:   defm TCGEN05_CP_128x128b # src_fmt : TCGEN05_CP_INTR<"128x128b", src_fmt>;
5856:   defm TCGEN05_CP_64x128_1 # src_fmt : TCGEN05_CP_INTR<"64x128b", src_fmt, "warpx2::02_13">;
5857:   defm TCGEN05_CP_64x128_2 # src_fmt : TCGEN05_CP_INTR<"64x128b", src_fmt, "warpx2::01_23">;
5858:   defm TCGEN05_CP_32x128 # src_fmt   : TCGEN05_CP_INTR<"32x128b", src_fmt, "warpx4">;
5859: }
5860: } // Predicates
5861:
5862: let Predicates = [callSubtarget<"hasTcgen05ShiftSupport">] in {
5863: multiclass TCGEN05_SHIFT_INTR<string num, Intrinsic Intr> {
5864:   def "" : BasicNVPTXInst<(outs),
5865:              (ins ADDR:$tmem_addr),
5866:              "tcgen05.shift.cta_group::" # num # ".down",
5867:              [(Intr addr:$tmem_addr)]>;
5868: }
5869: defm TCGEN05_SHIFT_CG1: TCGEN05_SHIFT_INTR<"1", int_nvvm_tcgen05_shift_down_cg1>;
5870: defm TCGEN05_SHIFT_CG2: TCGEN05_SHIFT_INTR<"2", int_nvvm_tcgen05_shift_down_cg2>;
5871: } // Predicates
5872:
5873: } // isConvergent
5874:
5875: let hasSideEffects = 1, Predicates = [callSubtarget<"hasTcgen05InstSupport">] in {
5876:
5877:   def tcgen05_fence_before_thread_sync: NullaryInst<
5878:     "tcgen05.fence::before_thread_sync", int_nvvm_tcgen05_fence_before_thread_sync>;
5879:
5880:   def tcgen05_fence_after_thread_sync: NullaryInst<
```
- EN: This range uses TableGen DSL to describe records such as SET_MAXNREG, INT_SET_MAXNREG_INC, INT_SET_MAXNREG_DEC, WGMMA_FENCE_SYNC_ALIGNED; LLVM later expands these declarations into generated backend tables and helper code.
- CN: 这一段使用 TableGen DSL 描述 SET_MAXNREG、INT_SET_MAXNREG_INC、INT_SET_MAXNREG_DEC、WGMMA_FENCE_SYNC_ALIGNED 等记录；LLVM 后续会把这些声明展开为生成的后端表和辅助代码。

### Lines 5881-6020
```tablegen
5881:     "tcgen05.fence::after_thread_sync", int_nvvm_tcgen05_fence_after_thread_sync>;
5882:
5883: } // hasSideEffects
5884:
5885: // name class for tcgen05.{ld, st}
5886: class TCGEN05_LDST_INST_NAME<string Op, string shape, int lg2Count, bit packOrUnpack> {
5887:   string name = "TCGEN05_" # Op
5888:                 # "_" # shape
5889:                 # "_x" # !shl(1, lg2Count)
5890:                 # !if(!eq(packOrUnpack, 1), !if(!eq(Op, "LD"), "_PACK", "_UNPACK"), "");
5891: }
5892:
5893: // reginfo class tcgen05.{ld, st}
5894: class TCGEN05_LDST_REGINFO<int Veclen> {
5895:   // create a list of types for load/store operands
5896:   list<NVPTXRegClass> regs = !listsplat(B32, Veclen);
5897:   // generate list of regnames for load/store operands
5898:   list<string> reg_names = !foreach(x, !range(0, Veclen), "r" # x);
5899:   string regstring = "{{" # !interleave(!foreach(n, !range(0, Veclen), "$r" # n), ", ") # "}}";
5900:   dag Ins = !dag(ins, regs, reg_names);
5901:   dag Outs = !dag(outs, regs, reg_names);
5902: }
5903:
5904: //
5905: // tcgen05.ld.sync.aligned.shape.x[1, 2, 4, 8, 16, 32, 64, 128][|.pack::16b].[b32]
5906: //
5907:
5908: class TCGEN05_LD_INST<string Shape, int Num, bit Pack> :
5909:         NVPTXInst<(outs), (ins), "?", []> {
5910:
5911:   TCGEN05_LDST_REGINFO Info = TCGEN05_LDST_REGINFO<
5912:                                 NVVM_TCGEN05_LDST_ACCESS_SIZE<Shape, Num>.veclen>;
5913:
5914:   let InOperandList = !con((ins B32:$taddr),
5915:                            !if(!eq(Shape, "16x32bx2"), (ins i64imm:$offset), (ins)));
5916:   let OutOperandList = Info.Outs;
5917:   let AsmString = "tcgen05.ld.sync.aligned"
5918:                   # "." # Shape
5919:                   # ".x" # !shl(1, Num)
5920:                   # !if(!eq(Pack, 1), ".pack::16b", "")
5921:                   # ".b32 "
5922:                   # Info.regstring # ", "
5923:                   # "[$taddr]"
5924:                   # !if(!eq(Shape, "16x32bx2"), ", $offset", "")
5925:                   # ";";
5926: }
5927:
5928: //
5929: // tcgen05.st.sync.aligned.shape.x[1, 2, 4, 8, 16, 32, 64, 128][|.unpack::16b].[b32]
5930: //
5931:
5932: class TCGEN05_ST_INST<string Shape, int Num, bit Unpack> :
5933:         NVPTXInst<(outs), (ins), "?", []> {
5934:
5935:   TCGEN05_LDST_REGINFO Info = TCGEN05_LDST_REGINFO<
5936:                                 NVVM_TCGEN05_LDST_ACCESS_SIZE<Shape, Num>.veclen>;
5937:
5938:   let InOperandList = !con((ins B32:$taddr),
5939:                            !if(!eq(Shape, "16x32bx2"), (ins i64imm:$offset), (ins)),
5940:                            Info.Ins);
5941:   let OutOperandList = (outs);
5942:   let AsmString = "tcgen05.st.sync.aligned"
5943:                   # "." # Shape
5944:                   # ".x" # !shl(1, Num)
5945:                   # !if(!eq(Unpack, 1), ".unpack::16b", "")
5946:                   # ".b32 [$taddr]"
5947:                   # !if(!eq(Shape, "16x32bx2"), ", $offset", "")
5948:                   # ", " # Info.regstring
5949:                   # ";";
5950: }
5951:
5952: let isConvergent = true, Predicates = [callSubtarget<"hasTcgen05InstSupport">] in {
5953:
5954: foreach shape = ["16x64b", "16x128b", "16x256b", "32x32b", "16x32bx2"] in {
5955:   foreach num = !range(0, 8) in {
5956:     foreach packOrUnpack = [false, true] in {
5957:       if NVVM_TCGEN05_LDST_ACCESS_SIZE<shape, num>.valid then {
5958:         def TCGEN05_LDST_INST_NAME<"LD", shape, num, packOrUnpack>.name :
5959:               TCGEN05_LD_INST<shape, num, packOrUnpack>;
5960:         def TCGEN05_LDST_INST_NAME<"ST", shape, num, packOrUnpack>.name :
5961:               TCGEN05_ST_INST<shape, num, packOrUnpack>;
5962:       }
5963:     }
5964:   }
5965: }
5966:
5967: } // isConvergent
5968:
5969: //
5970: // tcgen05.ld.red
5971: //
5972:
5973: class Tcgen05LdRedTypeProfile<int Num, bit hasOffset, ValueType RedValTy>:
5974:         SDTypeProfile<0, 0, []> {
5975:
5976:   list<ValueType> Results = !listsplat(RedValTy, !add(Num, 1)); // ret_type, red_val
5977:   list<ValueType> VTs = !listconcat(
5978:     Results,
5979:     [i32],                                // ptr
5980:     !if(!eq(hasOffset, 1), [i64], []),    // offset
5981:     [i32],                                // redOp
5982:     !if(!eq(RedValTy, f32), [i1, i1], []) // abs, nan
5983:   );
5984:   let Constraints = !foreach(x, !range(!size(VTs)), SDTCisVT<x, VTs[x]>);
5985:   let NumResults = !size(Results);
5986:   let NumOperands = !sub(!size(VTs), NumResults);
5987: }
5988:
5989: class Tcgen05LdRedSDNode<string Shape, int Num, string Type>:
5990:         SDNode<"NVPTXISD::TCGEN05_LD_RED_" # Shape # "_X" # Num # "_" # !toupper(Type),
5991:         Tcgen05LdRedTypeProfile<Num, !eq(Shape, "16x32bx2"), !cast<ValueType>(Type)>,
5992:                     [SDNPHasChain, SDNPSideEffect, SDNPMemOperand]>;
5993:
5994: class Tcgen05LdRedInst<string Shape, int Num, string RedOp, string Type,
5995:                        bit Abs = 0, bit Nan = 0>:
5996:          NVPTXInst<(outs), (ins), "?", []>,
5997:          Requires<[callSubtarget<"hasTcgen05LdRedSupport">]> {
5998:
5999:   bit IsFloat = !eq(Type, "f32");
6000:   string TypeStr = !if(IsFloat, "f32", "u32");
6001:
6002:   SDNode Opcode = Tcgen05LdRedSDNode<Shape, Num, Type>;
6003:
6004:   TCGEN05_LDST_REGINFO Info = TCGEN05_LDST_REGINFO<Num>;
6005:
6006:   let InOperandList = !con((ins B32:$addr),
6007:                            !if(!eq(Shape, "16x32bx2"), (ins i64imm:$offset),
6008:                                                        (ins)));
6009:
6010:   let OutOperandList = !con(Info.Outs, (outs B32:$redVal));
6011:
6012:   string AbsStr = !if(!and(!eq(IsFloat, 1), !eq(Abs, 1)), ".abs", "");
6013:   string NanStr = !if(!and(!eq(IsFloat, 1), !eq(Nan, 1)), ".NaN", "");
6014:
6015:   let AsmString = "tcgen05.ld.red.sync.aligned"
6016:                   # "." # Shape
6017:                   # ".x" # Num
6018:                   # "." # RedOp
6019:                   # AbsStr # NanStr
6020:                   # "." # TypeStr # " "
```
- EN: This range uses TableGen DSL to describe records such as TCGEN05_LDST_INST_NAME, TCGEN05_LDST_REGINFO, TCGEN05_LD_INST, TCGEN05_ST_INST; LLVM later expands these declarations into generated backend tables and helper code.
- CN: 这一段使用 TableGen DSL 描述 TCGEN05_LDST_INST_NAME、TCGEN05_LDST_REGINFO、TCGEN05_LD_INST、TCGEN05_ST_INST 等记录；LLVM 后续会把这些声明展开为生成的后端表和辅助代码。

### Lines 6021-6160
```tablegen
6021:                   # Info.regstring # ", $redVal, [$addr]"
6022:                   # !if(!eq(Shape, "16x32bx2"), ", $offset", "")
6023:                   # ";";
6024:
6025:   int RedOpVal = !cond(
6026:     !eq(RedOp, "min") : 0,
6027:     !eq(RedOp, "max") : 1,
6028:   );
6029:
6030:   int AbsVal = !if(!eq(Abs, 1), -1, 0);
6031:   int NanVal = !if(!eq(Nan, 1), -1, 0);
6032:
6033:   dag IntrinsicPattern = !con((Opcode i32:$addr),
6034:                               !if(!eq(Shape, "16x32bx2"), (Opcode i64:$offset),
6035:                                                           (Opcode)));
6036:
6037:   dag FlagOperands = !con((Opcode (i32 RedOpVal)),
6038:                           !if(!eq(IsFloat, 1), (Opcode (i1 AbsVal), (i1 NanVal)),
6039:                                                (Opcode)));
6040:
6041:   dag Results = !con(!foreach(tmp, OutOperandList,
6042:                                    !subst(outs, set,
6043:                                    !subst(B32, !if(!eq(IsFloat, 1), f32, i32), tmp))),
6044:                      // Unfortunately, there is no way to append a dag hence
6045:                      // creating a dummy argument so that it can be replaced with
6046:                      // the intrinsic arg dag
6047:                      (set 0));
6048:
6049:   let Pattern = [!setdagarg(Results, !sub(!size(Results), 1),
6050:                                      !con(IntrinsicPattern, FlagOperands))];
6051: }
6052:
6053: let isConvergent = true in {
6054:   foreach shape = ["32x32b", "16x32bx2"] in {
6055:     foreach num = 1...8 in {
6056:       foreach redop = ["min", "max"] in {
6057:         defvar access_size = NVVM_TCGEN05_LDST_ACCESS_SIZE<shape, num>;
6058:         if access_size.valid then {
6059:           foreach abs = [0, 1] in {
6060:             foreach nan = [0, 1] in {
6061:                 def : Tcgen05LdRedInst<shape, access_size.veclen, redop, "f32", abs, nan>;
6062:               } // nan
6063:             } // abs
6064:           def : Tcgen05LdRedInst<shape, access_size.veclen, redop, "i32">;
6065:         } // valid
6066:       } // redop
6067:     } // num
6068:   } // shape
6069: } // isConvergent
6070:
6071: // Bulk store instructions
6072: def st_bulk_imm : TImmLeaf<i64, [{ return Imm == 0; }]>;
6073:
6074: let Predicates = [hasSM<100>, hasPTX<86>] in {
6075:   def INT_NVVM_ST_BULK_GENERIC :
6076:     BasicNVPTXInst<(outs), (ins ADDR:$dest_addr, B64:$size, i64imm:$value),
6077:               "st.bulk",
6078:               [(int_nvvm_st_bulk addr:$dest_addr, i64:$size, st_bulk_imm:$value)]>;
6079:
6080:   def INT_NVVM_ST_BULK_SHARED_CTA:
6081:     BasicNVPTXInst<(outs), (ins ADDR:$dest_addr, B64:$size, i64imm:$value),
6082:               "st.bulk.shared::cta",
6083:               [(int_nvvm_st_bulk_shared_cta addr:$dest_addr, i64:$size, st_bulk_imm:$value)]>;
6084: }
6085:
6086: //
6087: // clusterlaunchcontorl Instructions
6088: //
6089:
6090: def CLUSTERLAUNCHCONTRL_TRY_CANCEL:
6091:       BasicNVPTXInst<(outs), (ins ADDR:$addr, ADDR:$mbar),
6092:                 "clusterlaunchcontrol.try_cancel.async.shared::cta.mbarrier::complete_tx::bytes.b128",
6093:                 [(int_nvvm_clusterlaunchcontrol_try_cancel_async_shared addr:$addr, addr:$mbar)]>,
6094:       Requires<[hasSM<100>, hasPTX<86>]>;
6095:
6096: def CLUSTERLAUNCHCONTRL_TRY_CANCEL_MULTICAST:
6097:       BasicNVPTXInst<(outs), (ins ADDR:$addr, ADDR:$mbar),
6098:                 "clusterlaunchcontrol.try_cancel.async.shared::cta.mbarrier::complete_tx::bytes" #
6099:                 ".multicast::cluster::all.b128",
6100:                 [(int_nvvm_clusterlaunchcontrol_try_cancel_async_multicast_shared addr:$addr, addr:$mbar)]>,
6101:       Requires<[callSubtarget<"hasClusterLaunchControlTryCancelMulticastSupport">]>;
6102:
6103: def SDTClusterLaunchControlQueryCancelIsCanceled: SDTypeProfile<1, 2, []>;
6104: def clusterlaunchcontrol_query_cancel_is_canceled:
6105:       SDNode<"NVPTXISD::CLUSTERLAUNCHCONTROL_QUERY_CANCEL_IS_CANCELED",
6106:              SDTClusterLaunchControlQueryCancelIsCanceled, []>;
6107:
6108: def CLUSTERLAUNCHCONTROL_QUERY_CANCEL_IS_CANCELED:
6109:   NVPTXInst<(outs B1:$pred), (ins B64:$try_cancel_response0, B64:$try_cancel_response1),
6110:             "{{\n\t" #
6111:                ".reg .b128 %clc_handle;\n\t" #
6112:                "mov.b128 %clc_handle, {$try_cancel_response0, $try_cancel_response1};\n\t" #
6113:                "clusterlaunchcontrol.query_cancel.is_canceled.pred.b128 $pred, %clc_handle;\n\t" #
6114:             "}}", [(set i1:$pred,
6115:                         (clusterlaunchcontrol_query_cancel_is_canceled i64:$try_cancel_response0, i64:$try_cancel_response1))]>,
6116:             Requires<[hasSM<100>, hasPTX<86>]>;
6117:
6118: class CLUSTERLAUNCHCONTROL_QUERY_CANCEL_GET_FIRST_CTAID<string Dim>:
6119:   NVPTXInst<(outs B32:$reg), (ins B64:$try_cancel_response0, B64:$try_cancel_response1),
6120:             "{{\n\t" #
6121:                ".reg .b128 %clc_handle;\n\t" #
6122:                "mov.b128 %clc_handle, {$try_cancel_response0, $try_cancel_response1};\n\t" #
6123:                "clusterlaunchcontrol.query_cancel.get_first_ctaid::" # Dim # ".b32.b128 $reg, %clc_handle;\n\t" #
6124:             "}}", [(set i32:$reg,
6125:                         (!cast<SDNode>("clusterlaunchcontrol_query_cancel_first_cta_id_" # Dim)
6126:                           i64:$try_cancel_response0, i64:$try_cancel_response1))]>,
6127:             Requires<[hasSM<100>, hasPTX<86>]>;
6128:
6129: foreach dim = ["x", "y", "z"] in {
6130:   def SDTClusterLaunchControlQueryCancelGetFirstCtaId # dim: SDTypeProfile<1, 2, []>;
6131:
6132:   def clusterlaunchcontrol_query_cancel_first_cta_id_ # dim :
6133:         SDNode<"NVPTXISD::CLUSTERLAUNCHCONTROL_QUERY_CANCEL_GET_FIRST_CTAID_" # !toupper(dim),
6134:                !cast<SDTypeProfile>("SDTClusterLaunchControlQueryCancelGetFirstCtaId" # dim), []>;
6135:
6136:   def CLUSTERLAUNCHCONTROL_QUERY_CANCEL_GET_FIRST_CTAID_ # dim:
6137:         CLUSTERLAUNCHCONTROL_QUERY_CANCEL_GET_FIRST_CTAID<dim>;
6138: }
6139:
6140: //
6141: // tcgen05.mma Helpers
6142: //
6143:
6144: class Tcgen05MMABase<bit IsSparse, string ASpace, string Kind, int CtaGroup,
6145:                      string CollectorUsage> :
6146:         NVPTXInst<(outs), (ins), "?", []>,
6147:         Requires<[]> {
6148:
6149:   int CollectorUsageVal = !cond(
6150:     !eq(CollectorUsage, "discard"): 0,
6151:     !eq(CollectorUsage, "lastuse"): 1,
6152:     !eq(CollectorUsage, "fill"): 2,
6153:     !eq(CollectorUsage, "use"): 3
6154:   );
6155:
6156:   int KindVal = !cond(
6157:                   !eq(Kind, "f16"): 0,
6158:                   !eq(Kind, "tf32"): 1,
6159:                   !eq(Kind, "f8f6f4"): 2,
6160:                   !eq(Kind, "i8"): 3,
```
- EN: This range uses TableGen DSL to describe records such as st_bulk_imm, INT_NVVM_ST_BULK_GENERIC, INT_NVVM_ST_BULK_SHARED_CTA, CLUSTERLAUNCHCONTRL_TRY_CANCEL; LLVM later expands these declarations into generated backend tables and helper code.
- CN: 这一段使用 TableGen DSL 描述 st_bulk_imm、INT_NVVM_ST_BULK_GENERIC、INT_NVVM_ST_BULK_SHARED_CTA、CLUSTERLAUNCHCONTRL_TRY_CANCEL 等记录；LLVM 后续会把这些声明展开为生成的后端表和辅助代码。

### Lines 6161-6300
```tablegen
6161:                 );
6162:
6163:   dag SparseMetadataIns = !if(IsSparse, (ins B32:$spmetadata), (ins));
6164:   string SparseMetadataStr = !if(IsSparse, ", [$spmetadata]", "");
6165:   dag SparseMetadataIntr = !if(IsSparse, (ins i32:$spmetadata), (ins));
6166:
6167:   string AOperandStr = !if(!eq(ASpace, "tensor"), "[$a]", "$a");
6168:   NVPTXRegClass ARegClass = !if(!eq(ASpace, "tensor"), B32, B64);
6169:
6170:   dag BasePatternArgs = !con((ins i32:$dtmem, ARegClass:$a, i64:$b,
6171:                                  i32:$idesc, i1:$enable_inp_d),
6172:                              SparseMetadataIntr);
6173:
6174:   dag BaseInOperandList = !con((ins B32:$dtmem, ARegClass:$a, B64:$b,
6175:                                     B32:$idesc, B1:$enable_inp_d),
6176:                                SparseMetadataIns);
6177:
6178:   let OutOperandList = (outs);
6179:
6180:   string Prefix = "tcgen05.mma";
6181:   string SpCtaKindStr = !if(IsSparse, ".sp", "")
6182:                         # ".cta_group::" # CtaGroup
6183:                         # ".kind::" # Kind;
6184:
6185:   string BaseOperandsStr = " [$dtmem], "
6186:                            # AOperandStr
6187:                            # ", $b"
6188:                            # SparseMetadataStr
6189:                            # ", $idesc";
6190:   string InputDStr = ", $enable_inp_d";
6191: }
6192:
6193: //
6194: // tcgen05.mma Instructions
6195: //
6196:
6197: class Tcgen05MMAInst<bit IsSparse, string ASpace, string Kind, int CtaGroup,
6198:                      string CollectorUsage, bit IsScaleInputD, bit IsAShift> :
6199:         Tcgen05MMABase<IsSparse, ASpace, Kind, CtaGroup, CollectorUsage> {
6200:
6201:   let Predicates = !cond(
6202:     IsScaleInputD : [callSubtarget<"hasTcgen05MMAScaleInputDImm">],
6203:     !eq(Kind, "i8") : [callSubtarget<"hasTcgen05MMAI8Kind">],
6204:     true : [callSubtarget<"hasTcgen05InstSupport">]
6205:   );
6206:
6207:   Intrinsic Intrin = !cast<Intrinsic>(
6208:                         NVVM_TCGEN05_MMA<IsSparse, ASpace, IsAShift, IsScaleInputD>.record_name
6209:                      );
6210:
6211:   dag ScaleInpIns = !if(IsScaleInputD, (ins i64imm:$scale_input_d), (ins));
6212:   string ScaleInpStr = !if(IsScaleInputD, ", $scale_input_d", "");
6213:   dag ScaleInpIntr = !if(IsScaleInputD, (Intrin i64:$scale_input_d), (Intrin));
6214:
6215:   let InOperandList = !con(BaseInOperandList, ScaleInpIns);
6216:
6217:   let AsmString = Prefix
6218:                   # SpCtaKindStr
6219:                   # !if(IsAShift, ".ashift", "")
6220:                   # ".collector::a::" # CollectorUsage
6221:                   # BaseOperandsStr
6222:                   # InputDStr
6223:                   # ScaleInpStr
6224:                   # ";";
6225:
6226:   dag IntrinsicPattern = !con(!foreach(tmp, BasePatternArgs, !subst(ins, Intrin, tmp)),
6227:                               ScaleInpIntr);
6228:
6229:   dag FlagOperands = (Intrin (i32 KindVal), (i32 CtaGroup),
6230:                              (i32 CollectorUsageVal));
6231:
6232:   let Pattern = [!con(IntrinsicPattern, FlagOperands)];
6233: }
6234:
6235: // tcgen05.mma
6236: foreach sparse = [0, 1] in {
6237:   foreach space = ["tensor", "shared"] in {
6238:     foreach kind = ["f16", "tf32", "f8f6f4", "i8"] in {
6239:       foreach cta_group = [1, 2] in {
6240:         foreach collector_usage = ["discard", "lastuse", "fill", "use"] in {
6241:           foreach scale_input_d = !if(!or(!eq(kind, "f16"),
6242:                                           !eq(kind, "tf32")), [0, 1], [0]) in {
6243:             foreach ashift = !if(!eq(space, "tensor"), [0, 1], [0]) in {
6244:
6245:               def : Tcgen05MMAInst<sparse, space, kind, cta_group,
6246:                                    collector_usage, scale_input_d, ashift>;
6247:             } // ashift
6248:           } // scale_input_d
6249:         } // collector_usage
6250:       } // cta_group
6251:     } // kind
6252:   } // space
6253: } // sparse
6254:
6255: //
6256: // tcgen05.mma.disable_output_lane Helpers
6257: //
6258:
6259: class Tcgen05MMADisableOutputLaneTypeProfile<bit IsSparse, string ASpace,
6260:                                              int CtaGroup, bit IsScaleInputD>:
6261:         SDTypeProfile<0, 0, []> {
6262:   int DisableOutputLaneVecSize = !mul(4, CtaGroup);
6263:
6264:   list<ValueType> VTs = !listconcat(
6265:     [i32],                                      // d
6266:     !if(!eq(ASpace, "tensor"), [i32], [i64]),   // a
6267:     [i64, i32, i1],                             // b, idesc, enable_inp_d
6268:     !if(IsSparse, [i32], []),                   // spmetadata
6269:     !if(IsScaleInputD, [i64], []),              // scale_input_d
6270:     !listsplat(i32, DisableOutputLaneVecSize),  // disable_output_lane
6271:     [i32, i32]                                  // kind, collector_usage
6272:   );
6273:   let Constraints = !foreach(x, !range(!size(VTs)), SDTCisVT<x, VTs[x]>);
6274:   let NumOperands = !size(Constraints);
6275: }
6276:
6277: class Tcgen05MMADisableOutputLaneSDNode<bit IsSparse, string ASpace,
6278:                                         int CtaGroup, bit IsScaleInput,
6279:                                         bit IsAShift>:
6280:         SDNode<"NVPTXISD::TCGEN05_MMA"
6281:                 # !if(IsSparse, "_SP", "")
6282:                 # "_" # !toupper(ASpace)
6283:                 # !if(IsScaleInput, "_SCALE_D", "")
6284:                 # "_DISABLE_OUTPUT_LANE_CG" # CtaGroup
6285:                 # !if(IsAShift, "_ASHIFT", ""),
6286:         Tcgen05MMADisableOutputLaneTypeProfile<IsSparse, ASpace, CtaGroup, IsScaleInput>,
6287:                     [SDNPHasChain, SDNPSideEffect, SDNPMemOperand]>;
6288:
6289: //
6290: // tcgen05.mma.disable_output_lane Instructions
6291: //
6292:
6293: class Tcgen05MMADisableOutputLaneInst<bit IsSparse, string ASpace, string Kind,
6294:                                       int CtaGroup, string CollectorUsage,
6295:                                       bit IsScaleInputD, bit IsAShift> :
6296:         Tcgen05MMABase<IsSparse, ASpace, Kind, CtaGroup, CollectorUsage> {
6297:
6298:   let Predicates = !cond(
6299:     IsScaleInputD : [callSubtarget<"hasTcgen05MMAScaleInputDImm">],
6300:     !eq(Kind, "i8") : [callSubtarget<"hasTcgen05MMAI8Kind">],
```
- EN: This range uses TableGen DSL to describe records such as Tcgen05MMAInst, Tcgen05MMADisableOutputLaneTypeProfile, Tcgen05MMADisableOutputLaneSDNode, Tcgen05MMADisableOutputLaneInst; LLVM later expands these declarations into generated backend tables and helper code.
- CN: 这一段使用 TableGen DSL 描述 Tcgen05MMAInst、Tcgen05MMADisableOutputLaneTypeProfile、Tcgen05MMADisableOutputLaneSDNode、Tcgen05MMADisableOutputLaneInst 等记录；LLVM 后续会把这些声明展开为生成的后端表和辅助代码。

### Lines 6301-6440
```tablegen
6301:     true : [callSubtarget<"hasTcgen05InstSupport">]
6302:   );
6303:
6304:   SDNode Opcode = Tcgen05MMADisableOutputLaneSDNode<IsSparse, ASpace, CtaGroup,
6305:                                                     IsScaleInputD, IsAShift>;
6306:
6307:   dag ScaleInpIns = !if(IsScaleInputD, (ins i64imm:$scale_input_d), (ins));
6308:   string ScaleInpStr = !if(IsScaleInputD, ", $scale_input_d", "");
6309:   dag ScaleInpIntr = !if(IsScaleInputD, (Opcode i64:$scale_input_d), (Opcode));
6310:
6311:   // disable output lane
6312:   int DisableOutputLaneVecSize = !mul(4, CtaGroup);
6313:
6314:   dag DisableOutputLaneIns = !dag(ins,
6315:                               !listsplat(B32, DisableOutputLaneVecSize),
6316:                               !foreach(x,
6317:                                       !range(DisableOutputLaneVecSize),
6318:                                       "disable_output_lane" # x));
6319:
6320:   dag DisableOutputLaneInput = !dag(Opcode,
6321:                                 !listsplat(i32, DisableOutputLaneVecSize),
6322:                                 !foreach(x,
6323:                                          !range(DisableOutputLaneVecSize),
6324:                                          "disable_output_lane" # x));
6325:
6326:   string DisableOutputLaneStr = "{{" #
6327:                                   !interleave(
6328:                                     !foreach(x,
6329:                                       !range(DisableOutputLaneVecSize),
6330:                                               "$disable_output_lane" # x),
6331:                                     ", ")
6332:                                 # "}}";
6333:
6334:   dag InOperandList = !con(BaseInOperandList,
6335:                            ScaleInpIns,
6336:                            DisableOutputLaneIns);
6337:
6338:   let AsmString = Prefix
6339:                   # SpCtaKindStr
6340:                   # !if(IsAShift, ".ashift", "")
6341:                   # ".collector::a::" # CollectorUsage
6342:                   # BaseOperandsStr
6343:                   # ", " # DisableOutputLaneStr
6344:                   # InputDStr
6345:                   # ScaleInpStr
6346:                   # ";";
6347:
6348:   dag IntrinsicPattern = !con(!foreach(tmp, BasePatternArgs, !subst(ins, Opcode, tmp)),
6349:                               ScaleInpIntr,
6350:                               DisableOutputLaneInput);
6351:
6352:   dag FlagOperands = (Opcode (i32 KindVal), (i32 CollectorUsageVal));
6353:
6354:   let Pattern = [!con(IntrinsicPattern, FlagOperands)];
6355: }
6356:
6357: // tcgen05.mma.disable_output_lane
6358: foreach sparse = [0, 1] in {
6359:   foreach space = ["tensor", "shared"] in {
6360:     foreach kind = ["f16", "tf32", "f8f6f4", "i8"] in {
6361:       foreach cta_group = [1, 2] in {
6362:         foreach collector_usage = ["fill", "use", "lastuse", "discard"] in {
6363:           foreach scale_input_d = !if(!or(!eq(kind, "f16"),
6364:                                           !eq(kind, "tf32")), [0, 1], [0]) in {
6365:             foreach ashift = !if(!eq(space, "tensor"), [0, 1], [0]) in {
6366:               def :
6367:                 Tcgen05MMADisableOutputLaneInst<sparse, space, kind, cta_group,
6368:                                                 collector_usage, scale_input_d,
6369:                                                 ashift>;
6370:             } // ashift
6371:           } // scale_input_d
6372:         } // collector_usage
6373:       } // cta_group
6374:     } // kind
6375:   } // space
6376: } // sparse
6377:
6378: //
6379: // tcgen05.mma.block_scale Instructions
6380: //
6381:
6382: class Tcgen05MMABlockScaleInst<bit IsSparse, string ASpace, string Kind,
6383:                                int CtaGroup, string ScaleVecSize,
6384:                                string CollectorUsage>:
6385:         Tcgen05MMABase<IsSparse, ASpace, Kind, CtaGroup, CollectorUsage> {
6386:
6387:   let Predicates = !cond(
6388:     !and(IsSparse,
6389:          !eq(Kind, "mxf4")) : [callSubtarget<"hasTcgen05MMASparseMxf4">],
6390:     !and(IsSparse,
6391:          !eq(Kind, "mxf4nvf4")) : [callSubtarget<"hasTcgen05MMASparseMxf4nvf4">],
6392:     !ne(ScaleVecSize, "") : [callSubtarget<"hasTcgen05InstSupport">, hasPTX<88>],
6393:     true : [callSubtarget<"hasTcgen05InstSupport">]
6394:   );
6395:
6396:   Intrinsic Intrin = !cast<Intrinsic>(
6397:                              NVVM_TCGEN05_MMA_BLOCKSCALE<IsSparse, ASpace, Kind, ScaleVecSize>.record_name);
6398:
6399:   let KindVal = !cond(
6400:                   !eq(Kind, "mxf8f6f4") : 0,
6401:                   !eq(Kind, "mxf4")     : 1,
6402:                   !eq(Kind, "mxf4nvf4") : 2,
6403:                 );
6404:
6405:   let InOperandList = !con(BaseInOperandList,
6406:                            (ins B32:$scale_a,
6407:                                 B32:$scale_b));
6408:   let AsmString = Prefix
6409:                   # SpCtaKindStr
6410:                   # ".block_scale" # ScaleVecSize
6411:                   # ".collector::a::" # CollectorUsage
6412:                   # BaseOperandsStr
6413:                   # ", [$scale_a], [$scale_b]"
6414:                   # InputDStr
6415:                   # ";";
6416:
6417:   dag IntrinsicPattern = !con(!foreach(tmp, BasePatternArgs, !subst(ins, Intrin, tmp)),
6418:                               (Intrin i32:$scale_a,
6419:                                       i32:$scale_b));
6420:
6421:   dag FlagOperands = (Intrin (i32 CtaGroup), (i32 CollectorUsageVal));
6422:
6423:   let Pattern = [!con(IntrinsicPattern, FlagOperands)];
6424: }
6425:
6426: // tcgen05.mma.block_scale
6427: foreach sparse = [0, 1] in {
6428:   foreach space = ["tensor", "shared"] in {
6429:     foreach kind = ["mxf8f6f4", "mxf4", "mxf4nvf4"] in {
6430:       foreach scale_vec_size = ["", ".block16", ".block32"] in {
6431:         foreach cta_group = [1, 2] in {
6432:           foreach collector_usage = ["fill", "use", "lastuse", "discard"] in {
6433:             if NVVM_TCGEN05_MMA_BLOCKSCALE_SUPPORTED<kind, scale_vec_size>.ret then {
6434:               def : Tcgen05MMABlockScaleInst<sparse, space, kind, cta_group,
6435:                                              scale_vec_size, collector_usage>;
6436:             }
6437:           } // collector_usage
6438:         } // cta_group
6439:       } // scale_vec_size
6440:     } // kind
```
- EN: This range uses TableGen DSL to describe records such as Tcgen05MMABlockScaleInst; LLVM later expands these declarations into generated backend tables and helper code.
- CN: 这一段使用 TableGen DSL 描述 Tcgen05MMABlockScaleInst 等记录；LLVM 后续会把这些声明展开为生成的后端表和辅助代码。

### Lines 6441-6567
```tablegen
6441:   } // space
6442: } // sparse
6443:
6444: //
6445: // tcgen05.mma.ws Instructions
6446: //
6447:
6448: class Tcgen05MMAWSInst<bit IsSparse, string ASpace, string Kind,
6449:                        int CollectorBufferB, string CollectorUsage,
6450:                        bit IsZeroColMask> :
6451:         Tcgen05MMABase<IsSparse, ASpace, Kind, /*CtaGroup=*/ 1, CollectorUsage> {
6452:
6453:   let Predicates = !cond(
6454:     !eq(Kind, "i8") : [callSubtarget<"hasTcgen05MMAI8Kind">],
6455:     true : [callSubtarget<"hasTcgen05InstSupport">]
6456:   );
6457:
6458:   Intrinsic Intrin = !cast<Intrinsic>(
6459:                             NVVM_TCGEN05_MMA_WS<IsSparse, ASpace, IsZeroColMask>.record_name);
6460:
6461:   dag ZeroColMaskIns = !if(IsZeroColMask, (ins B64:$zero_col_mask), (ins));
6462:   string ZeroColMaskStr = !if(IsZeroColMask, ", $zero_col_mask", "");
6463:   dag ZeroColMaskIntr = !if(IsZeroColMask,
6464:                             (Intrin i64:$zero_col_mask), (Intrin));
6465:
6466:   let InOperandList = !con(BaseInOperandList,
6467:                            ZeroColMaskIns);
6468:
6469:   let AsmString = Prefix
6470:                   # ".ws"
6471:                   # SpCtaKindStr
6472:                   # ".collector::b" # CollectorBufferB
6473:                   # "::" # CollectorUsage
6474:                   # BaseOperandsStr
6475:                   # InputDStr
6476:                   # ZeroColMaskStr
6477:                   # ";";
6478:
6479:   dag IntrinsicPattern = !con(!foreach(tmp, BasePatternArgs, !subst(ins, Intrin, tmp)),
6480:                               ZeroColMaskIntr);
6481:
6482:   dag FlagOperands = (Intrin (i32 KindVal), (i32 CollectorBufferB),
6483:                              (i32 CollectorUsageVal));
6484:
6485:   let Pattern = [!con(IntrinsicPattern, FlagOperands)];
6486: }
6487:
6488: // tcgen05.mma.ws
6489: foreach sparse = [0, 1] in {
6490:   foreach space = ["shared", "tensor"] in {
6491:     foreach kind = ["f16", "tf32", "f8f6f4", "i8"] in {
6492:       foreach collector_buffer_b = [0, 1, 2, 3] in {
6493:         foreach collector_usage_op = ["discard", "fill", "use", "lastuse"] in {
6494:           foreach zero_col_mask = [0, 1] in {
6495:               def : Tcgen05MMAWSInst<sparse, space, kind, collector_buffer_b,
6496:                                      collector_usage_op, zero_col_mask>;
6497:           } // zero_col_mask
6498:         } // collector_usage_op
6499:       } // collector_buffer_b
6500:     } // kind
6501:   } // space
6502: } // sparse
6503:
6504: //
6505: // tensormap.replace Instructions
6506: //
6507:
6508: class TensormapReplaceInst_2<string state_space, string field_name, 
6509:   string regclass_name, NVPTXRegClass val_RC, ValueType ValTy, Intrinsic Intrin,
6510:   code predicate> :
6511:   BasicNVPTXInst<(outs), 
6512:     (ins ADDR:$addr, val_RC:$val), 
6513:     "tensormap.replace.tile." # field_name # "." # state_space # ".b1024." # regclass_name,
6514:     [(PatFrag<(ops node:$addr, node:$val),
6515:        (Intrin node:$addr, node:$val), predicate>
6516:       addr:$addr, ValTy:$val)]>;
6517:
6518: class TensormapReplaceInst_3<string state_space, string field_name, 
6519:   string regclass_name, NVPTXRegClass val_RC, ValueType ValTy, Intrinsic Intrin,
6520:   code predicate> :
6521:   BasicNVPTXInst<(outs), 
6522:     (ins ADDR:$addr, B32:$ord, val_RC:$val), 
6523:     "tensormap.replace.tile." # field_name # "." # state_space # ".b1024." # regclass_name,
6524:     [(PatFrag<(ops node:$addr, node:$ord, node:$val),
6525:        (Intrin node:$addr, node:$ord, node:$val), predicate>
6526:       addr:$addr, i32:$ord, ValTy:$val)]>;
6527:
6528: foreach ss = ["GLOBAL", "SHARED_CTA"] in {
6529:   defvar pred = !if(!eq(ss, "GLOBAL"), AS_match.global, AS_match.shared);
6530:   defvar ss_ptx = !tolower(!subst("_", "::", ss));
6531:   let Predicates = [callSubtarget<"hasTensormapReplaceSupport">] in {
6532:     def TENSORMAP_REPLACE_TILE_GLOBAL_ADDRESS_ # ss : 
6533:       TensormapReplaceInst_2<ss_ptx, "global_address", "b64", B64, i64,
6534:         int_nvvm_tensormap_replace_global_address, pred>;
6535:
6536:     foreach field_name = ["INTERLEAVE_LAYOUT", "FILL_MODE", "RANK"] in {
6537:       defvar intrin = !cast<Intrinsic>("int_nvvm_tensormap_replace_" # !tolower(field_name));
6538:       def TENSORMAP_REPLACE_TILE_ # field_name # _ # ss : 
6539:         TensormapReplaceInst_2<ss_ptx, !tolower(field_name), "b32", B32, i32,
6540:           intrin, pred>;
6541:     } // field_name
6542:
6543:     def TENSORMAP_REPLACE_TILE_GLOBAL_STRIDE_ # ss : 
6544:       TensormapReplaceInst_3<ss_ptx, "global_stride", "b64", B64, i64, 
6545:         int_nvvm_tensormap_replace_global_stride, pred>;
6546:
6547:     foreach field_name = ["BOX_DIM", "GLOBAL_DIM", "ELEMENT_STRIDE"] in {
6548:       defvar intrin = !cast<Intrinsic>("int_nvvm_tensormap_replace_" # !tolower(field_name));
6549:       def TENSORMAP_REPLACE_TILE_ # field_name # _ # ss : 
6550:         TensormapReplaceInst_3<ss_ptx, !tolower(field_name), "b32", B32, i32, 
6551:           intrin, pred>;
6552:     } // field_name
6553:   } // hasTensormapReplaceSupport
6554:
6555:   def TENSORMAP_REPLACE_TILE_ELEMTYPE_ # ss : 
6556:     TensormapReplaceInst_2<ss_ptx, "elemtype", "b32", B32, i32, 
6557:       int_nvvm_tensormap_replace_elemtype, pred>;
6558:
6559:   def TENSORMAP_REPLACE_SWIZZLE_ATOMICITY_ # ss : 
6560:     TensormapReplaceInst_2<ss_ptx, "swizzle_atomicity", "b32", B32, i32, 
6561:       int_nvvm_tensormap_replace_swizzle_atomicity, pred>,
6562:     Requires<[callSubtarget<"hasTensormapReplaceSwizzleAtomicitySupport">]>;
6563:
6564:   def TENSORMAP_REPLACE_SWIZZLE_MODE_ # ss : 
6565:     TensormapReplaceInst_2<ss_ptx, "swizzle_mode", "b32", B32, i32, 
6566:       int_nvvm_tensormap_replace_swizzle_mode, pred>;
6567: } // state_space
```
- EN: This range uses TableGen DSL to describe records such as Tcgen05MMAWSInst, TensormapReplaceInst_2, TensormapReplaceInst_3, TENSORMAP_REPLACE_TILE_GLOBAL_ADDRESS_; LLVM later expands these declarations into generated backend tables and helper code.
- CN: 这一段使用 TableGen DSL 描述 Tcgen05MMAWSInst、TensormapReplaceInst_2、TensormapReplaceInst_3、TENSORMAP_REPLACE_TILE_GLOBAL_ADDRESS_ 等记录；LLVM 后续会把这些声明展开为生成的后端表和辅助代码。

## Key Concepts / 关键概念

- EN: Intrinsic handling connects target-specific builtins with LLVM IR or machine-level lowering paths.
  - CN: 内建函数处理负责把目标特定 builtin 与 LLVM IR 或机器级降级路径连接起来。
- EN: TableGen files use declarative records that LLVM expands into generated C++ tables and helper code.
  - CN: TableGen 文件使用声明式记录，LLVM 会将其展开为生成的 C++ 表和辅助代码。
- EN: NVPTX backend code reflects GPU execution concerns such as address spaces, intrinsics, and PTX-specific lowering.
  - CN: NVPTX 后端代码体现了 GPU 执行特性，例如地址空间、内建函数以及 PTX 特定降级。
- EN: Key symbols in this file include AS_match, Scope_thread, Scope_cta, Scope_cluster, Scope_device, which anchor the file's main abstractions.
  - CN: 该文件中的关键符号包括 AS_match, Scope_thread, Scope_cta, Scope_cluster, Scope_device，它们构成了本文件的核心抽象。

## Dependencies / 依赖关系

- TableGen pipeline / TableGen 流水线: records in this file are consumed by LLVM TableGen emitters to produce generated lookup tables and helper code.
- TableGen 流水线说明：该文件中的记录会被 LLVM 的 TableGen 生成器消费，进而产出查找表和辅助代码。
- This file has no explicit textual includes; its effective dependencies come from surrounding generated or linked LLVM components.
- 该文件没有显式文本 include；其实际依赖主要来自周边生成代码或链接到的 LLVM 组件。
