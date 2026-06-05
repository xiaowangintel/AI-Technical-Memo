# HexagonISelDAGToDAGHVX.cpp — Code Analysis / 代码分析

## Source / 来源

- File / 文件: `llvm/lib/Target/Hexagon/HexagonISelDAGToDAGHVX.cpp`
- Repository / 仓库: `/root/xw/llvm-project/`
- Purpose / 作用 (EN): Implementation of permutation networks.
- Purpose / 作用 (CN): 该文件实现 Hexagon 后端组件的核心逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-200 / 第 1-200 行

```cpp
     1: //===-- HexagonISelDAGToDAGHVX.cpp ----------------------------------------===//
     2: //
     3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
     4: // See https://llvm.org/LICENSE.txt for license information.
     5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
     6: //
     7: //===----------------------------------------------------------------------===//
     8: 
     9: #include "HexagonISelDAGToDAG.h"
    10: #include "HexagonISelLowering.h"
    11: #include "llvm/ADT/BitVector.h"
    12: #include "llvm/ADT/SetVector.h"
    13: #include "llvm/CodeGen/SelectionDAGISel.h"
    14: #include "llvm/IR/Intrinsics.h"
    15: #include "llvm/IR/IntrinsicsHexagon.h"
    16: #include "llvm/Support/Debug.h"
    17: #include "llvm/Support/MathExtras.h"
    18: 
    19: #include <algorithm>
    20: #include <cmath>
    21: #include <deque>
    22: #include <functional>
    23: #include <map>
    24: #include <optional>
    25: #include <set>
    26: #include <unordered_map>
    27: #include <utility>
    28: #include <vector>
    29: 
    30: #define DEBUG_TYPE "hexagon-isel"
    31: using namespace llvm;
    32: 
    33: namespace {
    34: 
    35: // --------------------------------------------------------------------
    36: // Implementation of permutation networks.
    37: 
    38: // Implementation of the node routing through butterfly networks:
    39: // - Forward delta.
    40: // - Reverse delta.
    41: // - Benes.
    42: //
    43: //
    44: // Forward delta network consists of log(N) steps, where N is the number
    45: // of inputs. In each step, an input can stay in place, or it can get
    46: // routed to another position[1]. The step after that consists of two
    47: // networks, each half in size in terms of the number of nodes. In those
    48: // terms, in the given step, an input can go to either the upper or the
    49: // lower network in the next step.
    50: //
    51: // [1] Hexagon's vdelta/vrdelta allow an element to be routed to both
    52: // positions as long as there is no conflict.
    53: 
    54: // Here's a delta network for 8 inputs, only the switching routes are
    55: // shown:
    56: //
    57: //         Steps:
    58: //         |- 1 ---------------|- 2 -----|- 3 -|
    59: //
    60: // Inp[0] ***                 ***       ***   *** Out[0]
    61: //           \               /   \     /   \ /
    62: //            \             /     \   /     X
    63: //             \           /       \ /     / \
    64: // Inp[1] ***   \         /   ***   X   ***   *** Out[1]
    65: //           \   \       /   /   \ / \ /
    66: //            \   \     /   /     X   X
    67: //             \   \   /   /     / \ / \
    68: // Inp[2] ***   \   \ /   /   ***   X   ***   *** Out[2]
    69: //           \   \   X   /   /     / \     \ /
    70: //            \   \ / \ /   /     /   \     X
    71: //             \   X   X   /     /     \   / \
    72: // Inp[3] ***   \ / \ / \ /   ***       ***   *** Out[3]
    73: //           \   X   X   X   /
    74: //            \ / \ / \ / \ /
    75: //             X   X   X   X
    76: //            / \ / \ / \ / \
    77: //           /   X   X   X   \
    78: // Inp[4] ***   / \ / \ / \   ***       ***   *** Out[4]
    79: //             /   X   X   \     \     /   \ /
    80: //            /   / \ / \   \     \   /     X
    81: //           /   /   X   \   \     \ /     / \
    82: // Inp[5] ***   /   / \   \   ***   X   ***   *** Out[5]
    83: //             /   /   \   \     \ / \ /
    84: //            /   /     \   \     X   X
    85: //           /   /       \   \   / \ / \
    86: // Inp[6] ***   /         \   ***   X   ***   *** Out[6]
    87: //             /           \       / \     \ /
    88: //            /             \     /   \     X
    89: //           /               \   /     \   / \
    90: // Inp[7] ***                 ***       ***   *** Out[7]
    91: //
    92: //
    93: // Reverse delta network is same as delta network, with the steps in
    94: // the opposite order.
    95: //
    96: //
    97: // Benes network is a forward delta network immediately followed by
    98: // a reverse delta network.
    99: 
   100: enum class ColorKind { None, Red, Black };
   101: 
   102: // Graph coloring utility used to partition nodes into two groups:
   103: // they will correspond to nodes routed to the upper and lower networks.
   104: struct Coloring {
   105:   using Node = int;
   106:   using MapType = std::map<Node, ColorKind>;
   107:   static constexpr Node Ignore = Node(-1);
   108: 
   109:   Coloring(ArrayRef<Node> Ord) : Order(Ord) {
   110:     build();
   111:     if (!color())
   112:       Colors.clear();
   113:   }
   114: 
   115:   const MapType &colors() const {
   116:     return Colors;
   117:   }
   118: 
   119:   ColorKind other(ColorKind Color) {
   120:     if (Color == ColorKind::None)
   121:       return ColorKind::Red;
   122:     return Color == ColorKind::Red ? ColorKind::Black : ColorKind::Red;
   123:   }
   124: 
   125:   LLVM_DUMP_METHOD void dump() const;
   126: 
   127: private:
   128:   ArrayRef<Node> Order;
   129:   MapType Colors;
   130:   std::set<Node> Needed;
   131: 
   132:   using NodeSet = std::set<Node>;
   133:   std::map<Node,NodeSet> Edges;
   134: 
   135:   Node conj(Node Pos) {
   136:     Node Num = Order.size();
   137:     return (Pos < Num/2) ? Pos + Num/2 : Pos - Num/2;
   138:   }
   139: 
   140:   ColorKind getColor(Node N) {
   141:     auto F = Colors.find(N);
   142:     return F != Colors.end() ? F->second : ColorKind::None;
   143:   }
   144: 
   145:   std::pair<bool, ColorKind> getUniqueColor(const NodeSet &Nodes);
   146: 
   147:   void build();
   148:   bool color();
   149: };
   150: } // namespace
   151: 
   152: std::pair<bool, ColorKind> Coloring::getUniqueColor(const NodeSet &Nodes) {
   153:   auto Color = ColorKind::None;
   154:   for (Node N : Nodes) {
   155:     ColorKind ColorN = getColor(N);
   156:     if (ColorN == ColorKind::None)
   157:       continue;
   158:     if (Color == ColorKind::None)
   159:       Color = ColorN;
   160:     else if (Color != ColorKind::None && Color != ColorN)
   161:       return { false, ColorKind::None };
   162:   }
   163:   return { true, Color };
   164: }
   165: 
   166: void Coloring::build() {
   167:   // Add Order[P] and Order[conj(P)] to Edges.
   168:   for (unsigned P = 0; P != Order.size(); ++P) {
   169:     Node I = Order[P];
   170:     if (I != Ignore) {
   171:       Needed.insert(I);
   172:       Node PC = Order[conj(P)];
   173:       if (PC != Ignore && PC != I)
   174:         Edges[I].insert(PC);
   175:     }
   176:   }
   177:   // Add I and conj(I) to Edges.
   178:   for (unsigned I = 0; I != Order.size(); ++I) {
   179:     if (!Needed.count(I))
   180:       continue;
   181:     Node C = conj(I);
   182:     // This will create an entry in the edge table, even if I is not
   183:     // connected to any other node. This is necessary, because it still
   184:     // needs to be colored.
   185:     NodeSet &Is = Edges[I];
   186:     if (Needed.count(C))
   187:       Is.insert(C);
   188:   }
   189: }
   190: 
   191: bool Coloring::color() {
   192:   SetVector<Node> FirstQ;
   193:   auto Enqueue = [this,&FirstQ] (Node N) {
   194:     SetVector<Node> Q;
   195:     Q.insert(N);
   196:     for (unsigned I = 0; I != Q.size(); ++I) {
   197:       NodeSet &Ns = Edges[Q[I]];
   198:       Q.insert_range(Ns);
   199:     }
   200:     FirstQ.insert_range(Q);
```
- EN: It imports headers such as HexagonISelDAGToDAG.h, HexagonISelLowering.h, llvm/ADT/BitVector.h, llvm/ADT/SetVector.h, ... (19 total), establishing the LLVM/Hexagon APIs used below. Header guards in this range prevent duplicate inclusion and define the interface boundary. It opens namespaces (llvm, std) to keep the implementation scoped to LLVM/Hexagon components. It declares types such as ColorKind, Coloring, which carry the state or API of this component.
- CN: 这里引入了 HexagonISelDAGToDAG.h, HexagonISelLowering.h, llvm/ADT/BitVector.h, llvm/ADT/SetVector.h, ... (19 total) 等头文件，确定了后续代码依赖的 LLVM/Hexagon API。 这一段中的头文件保护宏用于防止重复包含，并明确接口边界。 这里打开了命名空间（llvm, std），把实现限定在 LLVM/Hexagon 组件作用域中。 这里声明了 ColorKind, Coloring 等类型，用来承载该组件的状态或接口。

### Lines 201-400 / 第 201-400 行

```cpp
   201:   };
   202:   for (Node N : Needed)
   203:     Enqueue(N);
   204: 
   205:   for (Node N : FirstQ) {
   206:     if (Colors.count(N))
   207:       continue;
   208:     NodeSet &Ns = Edges[N];
   209:     auto P = getUniqueColor(Ns);
   210:     if (!P.first)
   211:       return false;
   212:     Colors[N] = other(P.second);
   213:   }
   214: 
   215:   // First, color nodes that don't have any dups.
   216:   for (auto E : Edges) {
   217:     Node N = E.first;
   218:     if (!Needed.count(conj(N)) || Colors.count(N))
   219:       continue;
   220:     auto P = getUniqueColor(E.second);
   221:     if (!P.first)
   222:       return false;
   223:     Colors[N] = other(P.second);
   224:   }
   225: 
   226:   // Now, nodes that are still uncolored. Since the graph can be modified
   227:   // in this step, create a work queue.
   228:   std::vector<Node> WorkQ;
   229:   for (auto E : Edges) {
   230:     Node N = E.first;
   231:     if (!Colors.count(N))
   232:       WorkQ.push_back(N);
   233:   }
   234: 
   235:   for (Node N : WorkQ) {
   236:     NodeSet &Ns = Edges[N];
   237:     auto P = getUniqueColor(Ns);
   238:     if (P.first) {
   239:       Colors[N] = other(P.second);
   240:       continue;
   241:     }
   242: 
   243:     // Coloring failed. Split this node.
   244:     Node C = conj(N);
   245:     ColorKind ColorN = other(ColorKind::None);
   246:     ColorKind ColorC = other(ColorN);
   247:     NodeSet &Cs = Edges[C];
   248:     NodeSet CopyNs = Ns;
   249:     for (Node M : CopyNs) {
   250:       ColorKind ColorM = getColor(M);
   251:       if (ColorM == ColorC) {
   252:         // Connect M with C, disconnect M from N.
   253:         Cs.insert(M);
   254:         Edges[M].insert(C);
   255:         Ns.erase(M);
   256:         Edges[M].erase(N);
   257:       }
   258:     }
   259:     Colors[N] = ColorN;
   260:     Colors[C] = ColorC;
   261:   }
   262: 
   263:   // Explicitly assign "None" to all uncolored nodes.
   264:   for (unsigned I = 0; I != Order.size(); ++I)
   265:     Colors.try_emplace(I, ColorKind::None);
   266: 
   267:   return true;
   268: }
   269: 
   270: #if !defined(NDEBUG) || defined(LLVM_ENABLE_DUMP)
   271: void Coloring::dump() const {
   272:   dbgs() << "{ Order:   {";
   273:   for (Node P : Order) {
   274:     if (P != Ignore)
   275:       dbgs() << ' ' << P;
   276:     else
   277:       dbgs() << " -";
   278:   }
   279:   dbgs() << " }\n";
   280:   dbgs() << "  Needed: {";
   281:   for (Node N : Needed)
   282:     dbgs() << ' ' << N;
   283:   dbgs() << " }\n";
   284: 
   285:   dbgs() << "  Edges: {\n";
   286:   for (auto E : Edges) {
   287:     dbgs() << "    " << E.first << " -> {";
   288:     for (auto N : E.second)
   289:       dbgs() << ' ' << N;
   290:     dbgs() << " }\n";
   291:   }
   292:   dbgs() << "  }\n";
   293: 
   294:   auto ColorKindToName = [](ColorKind C) {
   295:     switch (C) {
   296:     case ColorKind::None:
   297:       return "None";
   298:     case ColorKind::Red:
   299:       return "Red";
   300:     case ColorKind::Black:
   301:       return "Black";
   302:     }
   303:     llvm_unreachable("all ColorKinds should be handled by the switch above");
   304:   };
   305: 
   306:   dbgs() << "  Colors: {\n";
   307:   for (auto C : Colors)
   308:     dbgs() << "    " << C.first << " -> " << ColorKindToName(C.second) << "\n";
   309:   dbgs() << "  }\n}\n";
   310: }
   311: #endif
   312: 
   313: namespace {
   314: // Base class of for reordering networks. They don't strictly need to be
   315: // permutations, as outputs with repeated occurrences of an input element
   316: // are allowed.
   317: struct PermNetwork {
   318:   using Controls = std::vector<uint8_t>;
   319:   using ElemType = int;
   320:   static constexpr ElemType Ignore = ElemType(-1);
   321: 
   322:   enum : uint8_t {
   323:     None,
   324:     Pass,
   325:     Switch
   326:   };
   327:   enum : uint8_t {
   328:     Forward,
   329:     Reverse
   330:   };
   331: 
   332:   PermNetwork(ArrayRef<ElemType> Ord, unsigned Mult = 1) {
   333:     Order.assign(Ord.data(), Ord.data()+Ord.size());
   334:     Log = 0;
   335: 
   336:     unsigned S = Order.size();
   337:     while (S >>= 1)
   338:       ++Log;
   339: 
   340:     Table.resize(Order.size());
   341:     for (RowType &Row : Table)
   342:       Row.resize(Mult*Log, None);
   343:   }
   344: 
   345:   void getControls(Controls &V, unsigned StartAt, uint8_t Dir) const {
   346:     unsigned Size = Order.size();
   347:     V.resize(Size);
   348:     for (unsigned I = 0; I != Size; ++I) {
   349:       unsigned W = 0;
   350:       for (unsigned L = 0; L != Log; ++L) {
   351:         unsigned C = ctl(I, StartAt+L) == Switch;
   352:         if (Dir == Forward)
   353:           W |= C << (Log-1-L);
   354:         else
   355:           W |= C << L;
   356:       }
   357:       assert(isUInt<8>(W));
   358:       V[I] = uint8_t(W);
   359:     }
   360:   }
   361: 
   362:   uint8_t ctl(ElemType Pos, unsigned Step) const {
   363:     return Table[Pos][Step];
   364:   }
   365:   unsigned size() const {
   366:     return Order.size();
   367:   }
   368:   unsigned steps() const {
   369:     return Log;
   370:   }
   371: 
   372: protected:
   373:   unsigned Log;
   374:   std::vector<ElemType> Order;
   375:   using RowType = std::vector<uint8_t>;
   376:   std::vector<RowType> Table;
   377: };
   378: 
   379: struct ForwardDeltaNetwork : public PermNetwork {
   380:   ForwardDeltaNetwork(ArrayRef<ElemType> Ord) : PermNetwork(Ord) {}
   381: 
   382:   bool run(Controls &V) {
   383:     if (!route(Order.data(), Table.data(), size(), 0))
   384:       return false;
   385:     getControls(V, 0, Forward);
   386:     return true;
   387:   }
   388: 
   389: private:
   390:   bool route(ElemType *P, RowType *T, unsigned Size, unsigned Step);
   391: };
   392: 
   393: struct ReverseDeltaNetwork : public PermNetwork {
   394:   ReverseDeltaNetwork(ArrayRef<ElemType> Ord) : PermNetwork(Ord) {}
   395: 
   396:   bool run(Controls &V) {
   397:     if (!route(Order.data(), Table.data(), size(), 0))
   398:       return false;
   399:     getControls(V, 0, Reverse);
   400:     return true;
```
- EN: It declares types such as PermNetwork, ForwardDeltaNetwork, ReverseDeltaNetwork, which carry the state or API of this component. It declares or implements routines such as getUniqueColor, other, conj, getColor, ... (23 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage.
- CN: 这里声明了 PermNetwork, ForwardDeltaNetwork, ReverseDeltaNetwork 等类型，用来承载该组件的状态或接口。 这里声明或实现了 getUniqueColor, other, conj, getColor, ... (23 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。

### Lines 401-600 / 第 401-600 行

```cpp
   401:   }
   402: 
   403: private:
   404:   bool route(ElemType *P, RowType *T, unsigned Size, unsigned Step);
   405: };
   406: 
   407: struct BenesNetwork : public PermNetwork {
   408:   BenesNetwork(ArrayRef<ElemType> Ord) : PermNetwork(Ord, 2) {}
   409: 
   410:   bool run(Controls &F, Controls &R) {
   411:     if (!route(Order.data(), Table.data(), size(), 0))
   412:       return false;
   413: 
   414:     getControls(F, 0, Forward);
   415:     getControls(R, Log, Reverse);
   416:     return true;
   417:   }
   418: 
   419: private:
   420:   bool route(ElemType *P, RowType *T, unsigned Size, unsigned Step);
   421: };
   422: } // namespace
   423: 
   424: bool ForwardDeltaNetwork::route(ElemType *P, RowType *T, unsigned Size,
   425:                                 unsigned Step) {
   426:   bool UseUp = false, UseDown = false;
   427:   ElemType Num = Size;
   428: 
   429:   // Cannot use coloring here, because coloring is used to determine
   430:   // the "big" switch, i.e. the one that changes halves, and in a forward
   431:   // network, a color can be simultaneously routed to both halves in the
   432:   // step we're working on.
   433:   for (ElemType J = 0; J != Num; ++J) {
   434:     ElemType I = P[J];
   435:     // I is the position in the input,
   436:     // J is the position in the output.
   437:     if (I == Ignore)
   438:       continue;
   439:     uint8_t S;
   440:     if (I < Num/2)
   441:       S = (J < Num/2) ? Pass : Switch;
   442:     else
   443:       S = (J < Num/2) ? Switch : Pass;
   444: 
   445:     // U is the element in the table that needs to be updated.
   446:     ElemType U = (S == Pass) ? I : (I < Num/2 ? I+Num/2 : I-Num/2);
   447:     if (U < Num/2)
   448:       UseUp = true;
   449:     else
   450:       UseDown = true;
   451:     if (T[U][Step] != S && T[U][Step] != None)
   452:       return false;
   453:     T[U][Step] = S;
   454:   }
   455: 
   456:   for (ElemType J = 0; J != Num; ++J)
   457:     if (P[J] != Ignore && P[J] >= Num/2)
   458:       P[J] -= Num/2;
   459: 
   460:   if (Step+1 < Log) {
   461:     if (UseUp   && !route(P,        T,        Size/2, Step+1))
   462:       return false;
   463:     if (UseDown && !route(P+Size/2, T+Size/2, Size/2, Step+1))
   464:       return false;
   465:   }
   466:   return true;
   467: }
   468: 
   469: bool ReverseDeltaNetwork::route(ElemType *P, RowType *T, unsigned Size,
   470:                                 unsigned Step) {
   471:   unsigned Pets = Log-1 - Step;
   472:   bool UseUp = false, UseDown = false;
   473:   ElemType Num = Size;
   474: 
   475:   // In this step half-switching occurs, so coloring can be used.
   476:   Coloring G({P,Size});
   477:   const Coloring::MapType &M = G.colors();
   478:   if (M.empty())
   479:     return false;
   480: 
   481:   ColorKind ColorUp = ColorKind::None;
   482:   for (ElemType J = 0; J != Num; ++J) {
   483:     ElemType I = P[J];
   484:     // I is the position in the input,
   485:     // J is the position in the output.
   486:     if (I == Ignore)
   487:       continue;
   488:     ColorKind C = M.at(I);
   489:     if (C == ColorKind::None)
   490:       continue;
   491:     // During "Step", inputs cannot switch halves, so if the "up" color
   492:     // is still unknown, make sure that it is selected in such a way that
   493:     // "I" will stay in the same half.
   494:     bool InpUp = I < Num/2;
   495:     if (ColorUp == ColorKind::None)
   496:       ColorUp = InpUp ? C : G.other(C);
   497:     if ((C == ColorUp) != InpUp) {
   498:       // If I should go to a different half than where is it now, give up.
   499:       return false;
   500:     }
   501: 
   502:     uint8_t S;
   503:     if (InpUp) {
   504:       S = (J < Num/2) ? Pass : Switch;
   505:       UseUp = true;
   506:     } else {
   507:       S = (J < Num/2) ? Switch : Pass;
   508:       UseDown = true;
   509:     }
   510:     T[J][Pets] = S;
   511:   }
   512: 
   513:   // Reorder the working permutation according to the computed switch table
   514:   // for the last step (i.e. Pets).
   515:   for (ElemType J = 0, E = Size / 2; J != E; ++J) {
   516:     ElemType PJ = P[J];         // Current values of P[J]
   517:     ElemType PC = P[J+Size/2];  // and P[conj(J)]
   518:     ElemType QJ = PJ;           // New values of P[J]
   519:     ElemType QC = PC;           // and P[conj(J)]
   520:     if (T[J][Pets] == Switch)
   521:       QC = PJ;
   522:     if (T[J+Size/2][Pets] == Switch)
   523:       QJ = PC;
   524:     P[J] = QJ;
   525:     P[J+Size/2] = QC;
   526:   }
   527: 
   528:   for (ElemType J = 0; J != Num; ++J)
   529:     if (P[J] != Ignore && P[J] >= Num/2)
   530:       P[J] -= Num/2;
   531: 
   532:   if (Step+1 < Log) {
   533:     if (UseUp && !route(P, T, Size/2, Step+1))
   534:       return false;
   535:     if (UseDown && !route(P+Size/2, T+Size/2, Size/2, Step+1))
   536:       return false;
   537:   }
   538:   return true;
   539: }
   540: 
   541: bool BenesNetwork::route(ElemType *P, RowType *T, unsigned Size,
   542:                          unsigned Step) {
   543:   Coloring G({P,Size});
   544:   const Coloring::MapType &M = G.colors();
   545:   if (M.empty())
   546:     return false;
   547:   ElemType Num = Size;
   548: 
   549:   unsigned Pets = 2*Log-1 - Step;
   550:   bool UseUp = false, UseDown = false;
   551: 
   552:   // Both assignments, i.e. Red->Up and Red->Down are valid, but they will
   553:   // result in different controls. Let's pick the one where the first
   554:   // control will be "Pass".
   555:   ColorKind ColorUp = ColorKind::None;
   556:   for (ElemType J = 0; J != Num; ++J) {
   557:     ElemType I = P[J];
   558:     if (I == Ignore)
   559:       continue;
   560:     ColorKind C = M.at(I);
   561:     if (C == ColorKind::None)
   562:       continue;
   563:     if (ColorUp == ColorKind::None) {
   564:       ColorUp = (I < Num / 2) ? ColorKind::Red : ColorKind::Black;
   565:     }
   566:     unsigned CI = (I < Num/2) ? I+Num/2 : I-Num/2;
   567:     if (C == ColorUp) {
   568:       if (I < Num/2)
   569:         T[I][Step] = Pass;
   570:       else
   571:         T[CI][Step] = Switch;
   572:       T[J][Pets] = (J < Num/2) ? Pass : Switch;
   573:       UseUp = true;
   574:     } else { // Down
   575:       if (I < Num/2)
   576:         T[CI][Step] = Switch;
   577:       else
   578:         T[I][Step] = Pass;
   579:       T[J][Pets] = (J < Num/2) ? Switch : Pass;
   580:       UseDown = true;
   581:     }
   582:   }
   583: 
   584:   // Reorder the working permutation according to the computed switch table
   585:   // for the last step (i.e. Pets).
   586:   for (ElemType J = 0; J != Num/2; ++J) {
   587:     ElemType PJ = P[J];         // Current values of P[J]
   588:     ElemType PC = P[J+Num/2];   // and P[conj(J)]
   589:     ElemType QJ = PJ;           // New values of P[J]
   590:     ElemType QC = PC;           // and P[conj(J)]
   591:     if (T[J][Pets] == Switch)
   592:       QC = PJ;
   593:     if (T[J+Num/2][Pets] == Switch)
   594:       QJ = PC;
   595:     P[J] = QJ;
   596:     P[J+Num/2] = QC;
   597:   }
   598: 
   599:   for (ElemType J = 0; J != Num; ++J)
   600:     if (P[J] != Ignore && P[J] >= Num/2)
```
- EN: It opens namespaces (bool) to keep the implementation scoped to LLVM/Hexagon components. It declares types such as BenesNetwork, which carry the state or API of this component. It declares or implements routines such as route, BenesNetwork, run, getControls, ... (9 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal.
- CN: 这里打开了命名空间（bool），把实现限定在 LLVM/Hexagon 组件作用域中。 这里声明了 BenesNetwork 等类型，用来承载该组件的状态或接口。 这里声明或实现了 route, BenesNetwork, run, getControls, ... (9 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。

### Lines 601-800 / 第 601-800 行

```cpp
   601:       P[J] -= Num/2;
   602: 
   603:   if (Step+1 < Log) {
   604:     if (UseUp && !route(P, T, Size/2, Step+1))
   605:       return false;
   606:     if (UseDown && !route(P+Size/2, T+Size/2, Size/2, Step+1))
   607:       return false;
   608:   }
   609:   return true;
   610: }
   611: 
   612: // --------------------------------------------------------------------
   613: // Support for building selection results (output instructions that are
   614: // parts of the final selection).
   615: 
   616: namespace {
   617: struct OpRef {
   618:   OpRef(SDValue V) : OpV(V) {}
   619:   bool isValue() const { return OpV.getNode() != nullptr; }
   620:   bool isValid() const { return isValue() || !(OpN & Invalid); }
   621:   bool isUndef() const { return OpN & Undef; }
   622:   static OpRef res(int N) { return OpRef(Whole | (N & Index)); }
   623:   static OpRef fail() { return OpRef(Invalid); }
   624: 
   625:   static OpRef lo(const OpRef &R) {
   626:     assert(!R.isValue());
   627:     return OpRef(R.OpN & (Undef | Index | LoHalf));
   628:   }
   629:   static OpRef hi(const OpRef &R) {
   630:     assert(!R.isValue());
   631:     return OpRef(R.OpN & (Undef | Index | HiHalf));
   632:   }
   633:   static OpRef undef(MVT Ty) { return OpRef(Undef | Ty.SimpleTy); }
   634: 
   635:   // Direct value.
   636:   SDValue OpV = SDValue();
   637: 
   638:   // Reference to the operand of the input node:
   639:   // If the 31st bit is 1, it's undef, otherwise, bits 28..0 are the
   640:   // operand index:
   641:   // If bit 30 is set, it's the high half of the operand.
   642:   // If bit 29 is set, it's the low half of the operand.
   643:   unsigned OpN = 0;
   644: 
   645:   enum : unsigned {
   646:     Invalid = 0x10000000,
   647:     LoHalf  = 0x20000000,
   648:     HiHalf  = 0x40000000,
   649:     Whole   = LoHalf | HiHalf,
   650:     Undef   = 0x80000000,
   651:     Index   = 0x0FFFFFFF,  // Mask of the index value.
   652:     IndexBits = 28,
   653:   };
   654: 
   655:   LLVM_DUMP_METHOD
   656:   void print(raw_ostream &OS, const SelectionDAG &G) const;
   657: 
   658: private:
   659:   OpRef(unsigned N) : OpN(N) {}
   660: };
   661: 
   662: struct NodeTemplate {
   663:   NodeTemplate() = default;
   664:   unsigned Opc = 0;
   665:   MVT Ty = MVT::Other;
   666:   std::vector<OpRef> Ops;
   667: 
   668:   LLVM_DUMP_METHOD void print(raw_ostream &OS, const SelectionDAG &G) const;
   669: };
   670: 
   671: struct ResultStack {
   672:   ResultStack(SDNode *Inp)
   673:     : InpNode(Inp), InpTy(Inp->getValueType(0).getSimpleVT()) {}
   674:   SDNode *InpNode;
   675:   MVT InpTy;
   676:   unsigned push(const NodeTemplate &Res) {
   677:     List.push_back(Res);
   678:     return List.size()-1;
   679:   }
   680:   unsigned push(unsigned Opc, MVT Ty, std::vector<OpRef> &&Ops) {
   681:     NodeTemplate Res;
   682:     Res.Opc = Opc;
   683:     Res.Ty = Ty;
   684:     Res.Ops = Ops;
   685:     return push(Res);
   686:   }
   687:   bool empty() const { return List.empty(); }
   688:   unsigned size() const { return List.size(); }
   689:   unsigned top() const { return size()-1; }
   690:   const NodeTemplate &operator[](unsigned I) const { return List[I]; }
   691:   unsigned reset(unsigned NewTop) {
   692:     List.resize(NewTop+1);
   693:     return NewTop;
   694:   }
   695: 
   696:   using BaseType = std::vector<NodeTemplate>;
   697:   BaseType::iterator begin() { return List.begin(); }
   698:   BaseType::iterator end()   { return List.end(); }
   699:   BaseType::const_iterator begin() const { return List.begin(); }
   700:   BaseType::const_iterator end() const   { return List.end(); }
   701: 
   702:   BaseType List;
   703: 
   704:   LLVM_DUMP_METHOD
   705:   void print(raw_ostream &OS, const SelectionDAG &G) const;
   706: };
   707: } // namespace
   708: 
   709: #if !defined(NDEBUG) || defined(LLVM_ENABLE_DUMP)
   710: void OpRef::print(raw_ostream &OS, const SelectionDAG &G) const {
   711:   if (isValue()) {
   712:     OpV.getNode()->print(OS, &G);
   713:     return;
   714:   }
   715:   if (OpN & Invalid) {
   716:     OS << "invalid";
   717:     return;
   718:   }
   719:   if (OpN & Undef) {
   720:     OS << "undef";
   721:     return;
   722:   }
   723:   if ((OpN & Whole) != Whole) {
   724:     assert((OpN & Whole) == LoHalf || (OpN & Whole) == HiHalf);
   725:     if (OpN & LoHalf)
   726:       OS << "lo ";
   727:     else
   728:       OS << "hi ";
   729:   }
   730:   OS << '#' << SignExtend32(OpN & Index, IndexBits);
   731: }
   732: 
   733: void NodeTemplate::print(raw_ostream &OS, const SelectionDAG &G) const {
   734:   const TargetInstrInfo &TII = *G.getSubtarget().getInstrInfo();
   735:   OS << format("%8s", EVT(Ty).getEVTString().c_str()) << "  "
   736:      << TII.getName(Opc);
   737:   bool Comma = false;
   738:   for (const auto &R : Ops) {
   739:     if (Comma)
   740:       OS << ',';
   741:     Comma = true;
   742:     OS << ' ';
   743:     R.print(OS, G);
   744:   }
   745: }
   746: 
   747: void ResultStack::print(raw_ostream &OS, const SelectionDAG &G) const {
   748:   OS << "Input node:\n";
   749: #ifndef NDEBUG
   750:   InpNode->dumpr(&G);
   751: #endif
   752:   OS << "Result templates:\n";
   753:   for (unsigned I = 0, E = List.size(); I != E; ++I) {
   754:     OS << '[' << I << "] ";
   755:     List[I].print(OS, G);
   756:     OS << '\n';
   757:   }
   758: }
   759: #endif
   760: 
   761: namespace {
   762: struct ShuffleMask {
   763:   ShuffleMask(ArrayRef<int> M) : Mask(M) {
   764:     for (int M : Mask) {
   765:       if (M == -1)
   766:         continue;
   767:       MinSrc = (MinSrc == -1) ? M : std::min(MinSrc, M);
   768:       MaxSrc = (MaxSrc == -1) ? M : std::max(MaxSrc, M);
   769:     }
   770:   }
   771: 
   772:   ArrayRef<int> Mask;
   773:   int MinSrc = -1, MaxSrc = -1;
   774: 
   775:   ShuffleMask lo() const {
   776:     size_t H = Mask.size()/2;
   777:     return ShuffleMask(Mask.take_front(H));
   778:   }
   779:   ShuffleMask hi() const {
   780:     size_t H = Mask.size()/2;
   781:     return ShuffleMask(Mask.take_back(H));
   782:   }
   783: 
   784:   void print(raw_ostream &OS) const {
   785:     OS << "MinSrc:" << MinSrc << ", MaxSrc:" << MaxSrc << " {";
   786:     for (int M : Mask)
   787:       OS << ' ' << M;
   788:     OS << " }";
   789:   }
   790: };
   791: 
   792: [[maybe_unused]]
   793: raw_ostream &operator<<(raw_ostream &OS, const ShuffleMask &SM) {
   794:   SM.print(OS);
   795:   return OS;
   796: }
   797: } // namespace
   798: 
   799: namespace shuffles {
   800: using MaskT = SmallVector<int, 128>;
```
- EN: Header guards in this range prevent duplicate inclusion and define the interface boundary. It opens namespaces (namespace) to keep the implementation scoped to LLVM/Hexagon components. It declares types such as OpRef, NodeTemplate, ResultStack, ShuffleMask, which carry the state or API of this component. It declares or implements routines such as OpRef, isValue, isValid, isUndef, ... (34 total), translating Hexagon-specific policy into reusable code paths.
- CN: 这一段中的头文件保护宏用于防止重复包含，并明确接口边界。 这里打开了命名空间（namespace），把实现限定在 LLVM/Hexagon 组件作用域中。 这里声明了 OpRef, NodeTemplate, ResultStack, ShuffleMask 等类型，用来承载该组件的状态或接口。 这里声明或实现了 OpRef, isValue, isValid, isUndef, ... (34 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。

### Lines 801-1000 / 第 801-1000 行

```cpp
   801: // Vdd = vshuffvdd(Vu, Vv, Rt)
   802: // Vdd = vdealvdd(Vu, Vv, Rt)
   803: // Vd  = vpack(Vu, Vv, Size, TakeOdd)
   804: // Vd  = vshuff(Vu, Vv, Size, TakeOdd)
   805: // Vd  = vdeal(Vu, Vv, Size, TakeOdd)
   806: // Vd  = vdealb4w(Vu, Vv)
   807: 
   808: ArrayRef<int> lo(ArrayRef<int> Vuu) { return Vuu.take_front(Vuu.size() / 2); }
   809: ArrayRef<int> hi(ArrayRef<int> Vuu) { return Vuu.take_back(Vuu.size() / 2); }
   810: 
   811: MaskT vshuffvdd(ArrayRef<int> Vu, ArrayRef<int> Vv, unsigned Rt) {
   812:   int Len = Vu.size();
   813:   MaskT Vdd(2 * Len);
   814:   llvm::copy(Vv, Vdd.begin());
   815:   llvm::copy(Vu, Vdd.begin() + Len);
   816: 
   817:   auto Vd0 = MutableArrayRef<int>(Vdd).take_front(Len);
   818:   auto Vd1 = MutableArrayRef<int>(Vdd).take_back(Len);
   819: 
   820:   for (int Offset = 1; Offset < Len; Offset *= 2) {
   821:     if ((Rt & Offset) == 0)
   822:       continue;
   823:     for (int i = 0; i != Len; ++i) {
   824:       if ((i & Offset) == 0)
   825:         std::swap(Vd1[i], Vd0[i + Offset]);
   826:     }
   827:   }
   828:   return Vdd;
   829: }
   830: 
   831: MaskT vdealvdd(ArrayRef<int> Vu, ArrayRef<int> Vv, unsigned Rt) {
   832:   int Len = Vu.size();
   833:   MaskT Vdd(2 * Len);
   834:   llvm::copy(Vv, Vdd.begin());
   835:   llvm::copy(Vu, Vdd.begin() + Len);
   836: 
   837:   auto Vd0 = MutableArrayRef<int>(Vdd).take_front(Len);
   838:   auto Vd1 = MutableArrayRef<int>(Vdd).take_back(Len);
   839: 
   840:   for (int Offset = Len / 2; Offset > 0; Offset /= 2) {
   841:     if ((Rt & Offset) == 0)
   842:       continue;
   843:     for (int i = 0; i != Len; ++i) {
   844:       if ((i & Offset) == 0)
   845:         std::swap(Vd1[i], Vd0[i + Offset]);
   846:     }
   847:   }
   848:   return Vdd;
   849: }
   850: 
   851: MaskT vpack(ArrayRef<int> Vu, ArrayRef<int> Vv, unsigned Size, bool TakeOdd) {
   852:   int Len = Vu.size();
   853:   MaskT Vd(Len);
   854:   auto Odd = static_cast<int>(TakeOdd);
   855:   for (int i = 0, e = Len / (2 * Size); i != e; ++i) {
   856:     for (int b = 0; b != static_cast<int>(Size); ++b) {
   857:       // clang-format off
   858:       Vd[i * Size + b]           = Vv[(2 * i + Odd) * Size + b];
   859:       Vd[i * Size + b + Len / 2] = Vu[(2 * i + Odd) * Size + b];
   860:       // clang-format on
   861:     }
   862:   }
   863:   return Vd;
   864: }
   865: 
   866: MaskT vshuff(ArrayRef<int> Vu, ArrayRef<int> Vv, unsigned Size, bool TakeOdd) {
   867:   int Len = Vu.size();
   868:   MaskT Vd(Len);
   869:   auto Odd = static_cast<int>(TakeOdd);
   870:   for (int i = 0, e = Len / (2 * Size); i != e; ++i) {
   871:     for (int b = 0; b != static_cast<int>(Size); ++b) {
   872:       Vd[(2 * i + 0) * Size + b] = Vv[(2 * i + Odd) * Size + b];
   873:       Vd[(2 * i + 1) * Size + b] = Vu[(2 * i + Odd) * Size + b];
   874:     }
   875:   }
   876:   return Vd;
   877: }
   878: 
   879: MaskT vdeal(ArrayRef<int> Vu, ArrayRef<int> Vv, unsigned Size, bool TakeOdd) {
   880:   int Len = Vu.size();
   881:   MaskT T = vdealvdd(Vu, Vv, Len - 2 * Size);
   882:   return vpack(hi(T), lo(T), Size, TakeOdd);
   883: }
   884: 
   885: MaskT vdealb4w(ArrayRef<int> Vu, ArrayRef<int> Vv) {
   886:   int Len = Vu.size();
   887:   MaskT Vd(Len);
   888:   for (int i = 0, e = Len / 4; i != e; ++i) {
   889:     Vd[0 * (Len / 4) + i] = Vv[4 * i + 0];
   890:     Vd[1 * (Len / 4) + i] = Vv[4 * i + 2];
   891:     Vd[2 * (Len / 4) + i] = Vu[4 * i + 0];
   892:     Vd[3 * (Len / 4) + i] = Vu[4 * i + 2];
   893:   }
   894:   return Vd;
   895: }
   896: 
   897: template <typename ShuffFunc, typename... OptArgs>
   898: auto mask(ShuffFunc S, unsigned Length, OptArgs... args) -> MaskT {
   899:   MaskT Vu(Length), Vv(Length);
   900:   std::iota(Vu.begin(), Vu.end(), Length); // High
   901:   std::iota(Vv.begin(), Vv.end(), 0);      // Low
   902:   return S(Vu, Vv, args...);
   903: }
   904: 
   905: } // namespace shuffles
   906: 
   907: // --------------------------------------------------------------------
   908: // The HvxSelector class.
   909: 
   910: static const HexagonTargetLowering &getHexagonLowering(SelectionDAG &G) {
   911:   return static_cast<const HexagonTargetLowering&>(G.getTargetLoweringInfo());
   912: }
   913: static const HexagonSubtarget &getHexagonSubtarget(SelectionDAG &G) {
   914:   return G.getSubtarget<HexagonSubtarget>();
   915: }
   916: 
   917: namespace llvm {
   918:   struct HvxSelector {
   919:     const HexagonTargetLowering &Lower;
   920:     HexagonDAGToDAGISel &ISel;
   921:     SelectionDAG &DAG;
   922:     const HexagonSubtarget &HST;
   923:     const unsigned HwLen;
   924: 
   925:     HvxSelector(HexagonDAGToDAGISel &HS, SelectionDAG &G)
   926:         : Lower(getHexagonLowering(G)), ISel(HS), DAG(G),
   927:           HST(getHexagonSubtarget(G)), HwLen(HST.getVectorLength()) {}
   928: 
   929:     MVT getSingleVT(MVT ElemTy) const {
   930:       assert(ElemTy != MVT::i1 && "Use getBoolVT for predicates");
   931:       unsigned NumElems = HwLen / (ElemTy.getSizeInBits() / 8);
   932:       return MVT::getVectorVT(ElemTy, NumElems);
   933:     }
   934: 
   935:     MVT getPairVT(MVT ElemTy) const {
   936:       assert(ElemTy != MVT::i1); // Suspicious: there are no predicate pairs.
   937:       unsigned NumElems = (2 * HwLen) / (ElemTy.getSizeInBits() / 8);
   938:       return MVT::getVectorVT(ElemTy, NumElems);
   939:     }
   940: 
   941:     MVT getBoolVT() const {
   942:       // Return HwLen x i1.
   943:       return MVT::getVectorVT(MVT::i1, HwLen);
   944:     }
   945: 
   946:     void selectExtractSubvector(SDNode *N);
   947:     void selectShuffle(SDNode *N);
   948:     void selectRor(SDNode *N);
   949:     void selectVAlign(SDNode *N);
   950: 
   951:     static SmallVector<uint32_t, 8> getPerfectCompletions(ShuffleMask SM,
   952:                                                           unsigned Width);
   953:     static SmallVector<uint32_t, 8> completeToPerfect(
   954:         ArrayRef<uint32_t> Completions, unsigned Width);
   955:     static std::optional<int> rotationDistance(ShuffleMask SM, unsigned WrapAt);
   956: 
   957:   private:
   958:     void select(SDNode *ISelN);
   959:     void materialize(const ResultStack &Results);
   960: 
   961:     SDValue getConst32(unsigned Val, const SDLoc &dl);
   962:     SDValue getSignedConst32(int Val, const SDLoc &dl);
   963:     SDValue getVectorConstant(ArrayRef<uint8_t> Data, const SDLoc &dl);
   964: 
   965:     enum : unsigned {
   966:       None,
   967:       PackMux,
   968:     };
   969:     OpRef concats(OpRef Va, OpRef Vb, ResultStack &Results);
   970:     OpRef funnels(OpRef Va, OpRef Vb, int Amount, ResultStack &Results);
   971: 
   972:     OpRef packs(ShuffleMask SM, OpRef Va, OpRef Vb, ResultStack &Results,
   973:                 MutableArrayRef<int> NewMask, unsigned Options = None);
   974:     OpRef packp(ShuffleMask SM, OpRef Va, OpRef Vb, ResultStack &Results,
   975:                 MutableArrayRef<int> NewMask);
   976:     OpRef vmuxs(ArrayRef<uint8_t> Bytes, OpRef Va, OpRef Vb,
   977:                 ResultStack &Results);
   978:     OpRef vmuxp(ArrayRef<uint8_t> Bytes, OpRef Va, OpRef Vb,
   979:                 ResultStack &Results);
   980: 
   981:     OpRef shuffs1(ShuffleMask SM, OpRef Va, ResultStack &Results);
   982:     OpRef shuffs2(ShuffleMask SM, OpRef Va, OpRef Vb, ResultStack &Results);
   983:     OpRef shuffp1(ShuffleMask SM, OpRef Va, ResultStack &Results);
   984:     OpRef shuffp2(ShuffleMask SM, OpRef Va, OpRef Vb, ResultStack &Results);
   985: 
   986:     OpRef butterfly(ShuffleMask SM, OpRef Va, ResultStack &Results);
   987:     OpRef contracting(ShuffleMask SM, OpRef Va, OpRef Vb, ResultStack &Results);
   988:     OpRef expanding(ShuffleMask SM, OpRef Va, ResultStack &Results);
   989:     OpRef perfect(ShuffleMask SM, OpRef Va, ResultStack &Results);
   990: 
   991:     bool selectVectorConstants(SDNode *N);
   992:     bool scalarizeShuffle(ArrayRef<int> Mask, const SDLoc &dl, MVT ResTy,
   993:                           SDValue Va, SDValue Vb, SDNode *N);
   994:   };
   995: } // namespace llvm
   996: 
   997: static void splitMask(ArrayRef<int> Mask, MutableArrayRef<int> MaskL,
   998:                       MutableArrayRef<int> MaskR) {
   999:   unsigned VecLen = Mask.size();
  1000:   assert(MaskL.size() == VecLen && MaskR.size() == VecLen);
```
- EN: It opens namespaces (shuffles, llvm) to keep the implementation scoped to LLVM/Hexagon components. It declares types such as HvxSelector, which carry the state or API of this component. It declares or implements routines such as vshuffvdd, take_front, hi, take_back, ... (58 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal.
- CN: 这里打开了命名空间（shuffles, llvm），把实现限定在 LLVM/Hexagon 组件作用域中。 这里声明了 HvxSelector 等类型，用来承载该组件的状态或接口。 这里声明或实现了 vshuffvdd, take_front, hi, take_back, ... (58 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。

### Lines 1001-1200 / 第 1001-1200 行

```cpp
  1001:   for (unsigned I = 0; I != VecLen; ++I) {
  1002:     int M = Mask[I];
  1003:     if (M < 0) {
  1004:       MaskL[I] = MaskR[I] = -1;
  1005:     } else if (unsigned(M) < VecLen) {
  1006:       MaskL[I] = M;
  1007:       MaskR[I] = -1;
  1008:     } else {
  1009:       MaskL[I] = -1;
  1010:       MaskR[I] = M-VecLen;
  1011:     }
  1012:   }
  1013: }
  1014: 
  1015: static std::pair<int,unsigned> findStrip(ArrayRef<int> A, int Inc,
  1016:                                          unsigned MaxLen) {
  1017:   assert(A.size() > 0 && A.size() >= MaxLen);
  1018:   int F = A[0];
  1019:   int E = F;
  1020:   for (unsigned I = 1; I != MaxLen; ++I) {
  1021:     if (A[I] - E != Inc)
  1022:       return { F, I };
  1023:     E = A[I];
  1024:   }
  1025:   return { F, MaxLen };
  1026: }
  1027: 
  1028: static bool isUndef(ArrayRef<int> Mask) {
  1029:   for (int Idx : Mask)
  1030:     if (Idx != -1)
  1031:       return false;
  1032:   return true;
  1033: }
  1034: 
  1035: static bool isIdentity(ArrayRef<int> Mask) {
  1036:   for (int I = 0, E = Mask.size(); I != E; ++I) {
  1037:     int M = Mask[I];
  1038:     if (M >= 0 && M != I)
  1039:       return false;
  1040:   }
  1041:   return true;
  1042: }
  1043: 
  1044: static bool isLowHalfOnly(ArrayRef<int> Mask) {
  1045:   int L = Mask.size();
  1046:   assert(L % 2 == 0);
  1047:   // Check if the second half of the mask is all-undef.
  1048:   return llvm::all_of(Mask.drop_front(L / 2), [](int M) { return M < 0; });
  1049: }
  1050: 
  1051: static SmallVector<unsigned, 4> getInputSegmentList(ShuffleMask SM,
  1052:                                                     unsigned SegLen) {
  1053:   assert(isPowerOf2_32(SegLen));
  1054:   SmallVector<unsigned, 4> SegList;
  1055:   if (SM.MaxSrc == -1)
  1056:     return SegList;
  1057: 
  1058:   unsigned Shift = Log2_32(SegLen);
  1059:   BitVector Segs(alignTo(SM.MaxSrc + 1, SegLen) >> Shift);
  1060: 
  1061:   for (int M : SM.Mask) {
  1062:     if (M >= 0)
  1063:       Segs.set(M >> Shift);
  1064:   }
  1065: 
  1066:   llvm::append_range(SegList, Segs.set_bits());
  1067:   return SegList;
  1068: }
  1069: 
  1070: static SmallVector<unsigned, 4> getOutputSegmentMap(ShuffleMask SM,
  1071:                                                     unsigned SegLen) {
  1072:   // Calculate the layout of the output segments in terms of the input
  1073:   // segments.
  1074:   // For example [1,3,1,0] means that the output consists of 4 output
  1075:   // segments, where the first output segment has only elements of the
  1076:   // input segment at index 1. The next output segment only has elements
  1077:   // of the input segment 3, etc.
  1078:   // If an output segment only has undef elements, the value will be ~0u.
  1079:   // If an output segment has elements from more than one input segment,
  1080:   // the corresponding value will be ~1u.
  1081:   unsigned MaskLen = SM.Mask.size();
  1082:   assert(MaskLen % SegLen == 0);
  1083:   SmallVector<unsigned, 4> Map(MaskLen / SegLen);
  1084: 
  1085:   for (int S = 0, E = Map.size(); S != E; ++S) {
  1086:     unsigned Idx = ~0u;
  1087:     for (int I = 0; I != static_cast<int>(SegLen); ++I) {
  1088:       int M = SM.Mask[S*SegLen + I];
  1089:       if (M < 0)
  1090:         continue;
  1091:       unsigned G = M / SegLen; // Input segment of this element.
  1092:       if (Idx == ~0u) {
  1093:         Idx = G;
  1094:       } else if (Idx != G) {
  1095:         Idx = ~1u;
  1096:         break;
  1097:       }
  1098:     }
  1099:     Map[S] = Idx;
  1100:   }
  1101: 
  1102:   return Map;
  1103: }
  1104: 
  1105: static void packSegmentMask(ArrayRef<int> Mask, ArrayRef<unsigned> OutSegMap,
  1106:                             unsigned SegLen, MutableArrayRef<int> PackedMask) {
  1107:   SmallVector<unsigned, 4> InvMap;
  1108:   for (int I = OutSegMap.size() - 1; I >= 0; --I) {
  1109:     unsigned S = OutSegMap[I];
  1110:     assert(S != ~0u && "Unexpected undef");
  1111:     assert(S != ~1u && "Unexpected multi");
  1112:     if (InvMap.size() <= S)
  1113:       InvMap.resize(S+1);
  1114:     InvMap[S] = I;
  1115:   }
  1116: 
  1117:   unsigned Shift = Log2_32(SegLen);
  1118:   for (int I = 0, E = Mask.size(); I != E; ++I) {
  1119:     int M = Mask[I];
  1120:     if (M >= 0) {
  1121:       int OutIdx = InvMap[M >> Shift];
  1122:       M = (M & (SegLen-1)) + SegLen*OutIdx;
  1123:     }
  1124:     PackedMask[I] = M;
  1125:   }
  1126: }
  1127: 
  1128: bool HvxSelector::selectVectorConstants(SDNode *N) {
  1129:   // Constant vectors are generated as loads from constant pools or as
  1130:   // splats of a constant value. Since they are generated during the
  1131:   // selection process, the main selection algorithm is not aware of them.
  1132:   // Select them directly here.
  1133:   SmallVector<SDNode*,4> Nodes;
  1134:   SetVector<SDNode*> WorkQ;
  1135: 
  1136:   // The DAG can change (due to CSE) during selection, so cache all the
  1137:   // unselected nodes first to avoid traversing a mutating DAG.
  1138:   WorkQ.insert(N);
  1139:   for (unsigned i = 0; i != WorkQ.size(); ++i) {
  1140:     SDNode *W = WorkQ[i];
  1141:     if (!W->isMachineOpcode() && W->getOpcode() == HexagonISD::ISEL)
  1142:       Nodes.push_back(W);
  1143:     for (unsigned j = 0, f = W->getNumOperands(); j != f; ++j)
  1144:       WorkQ.insert(W->getOperand(j).getNode());
  1145:   }
  1146: 
  1147:   for (SDNode *L : Nodes)
  1148:     select(L);
  1149: 
  1150:   return !Nodes.empty();
  1151: }
  1152: 
  1153: void HvxSelector::materialize(const ResultStack &Results) {
  1154:   DEBUG_WITH_TYPE("isel", {
  1155:     dbgs() << "Materializing\n";
  1156:     Results.print(dbgs(), DAG);
  1157:   });
  1158:   if (Results.empty())
  1159:     return;
  1160:   const SDLoc &dl(Results.InpNode);
  1161:   std::vector<SDValue> Output;
  1162: 
  1163:   for (unsigned I = 0, E = Results.size(); I != E; ++I) {
  1164:     const NodeTemplate &Node = Results[I];
  1165:     std::vector<SDValue> Ops;
  1166:     for (const OpRef &R : Node.Ops) {
  1167:       assert(R.isValid());
  1168:       if (R.isValue()) {
  1169:         Ops.push_back(R.OpV);
  1170:         continue;
  1171:       }
  1172:       if (R.OpN & OpRef::Undef) {
  1173:         MVT::SimpleValueType SVT = MVT::SimpleValueType(R.OpN & OpRef::Index);
  1174:         Ops.push_back(ISel.selectUndef(dl, MVT(SVT)));
  1175:         continue;
  1176:       }
  1177:       // R is an index of a result.
  1178:       unsigned Part = R.OpN & OpRef::Whole;
  1179:       int Idx = SignExtend32(R.OpN & OpRef::Index, OpRef::IndexBits);
  1180:       if (Idx < 0)
  1181:         Idx += I;
  1182:       assert(Idx >= 0 && unsigned(Idx) < Output.size());
  1183:       SDValue Op = Output[Idx];
  1184:       MVT OpTy = Op.getValueType().getSimpleVT();
  1185:       if (Part != OpRef::Whole) {
  1186:         assert(Part == OpRef::LoHalf || Part == OpRef::HiHalf);
  1187:         MVT HalfTy = MVT::getVectorVT(OpTy.getVectorElementType(),
  1188:                                       OpTy.getVectorNumElements()/2);
  1189:         unsigned Sub = (Part == OpRef::LoHalf) ? Hexagon::vsub_lo
  1190:                                                : Hexagon::vsub_hi;
  1191:         Op = DAG.getTargetExtractSubreg(Sub, dl, HalfTy, Op);
  1192:       }
  1193:       Ops.push_back(Op);
  1194:     } // for (Node : Results)
  1195: 
  1196:     assert(Node.Ty != MVT::Other);
  1197:     SDNode *ResN = (Node.Opc == TargetOpcode::COPY)
  1198:                       ? Ops.front().getNode()
  1199:                       : DAG.getMachineNode(Node.Opc, dl, Node.Ty, Ops);
  1200:     Output.push_back(SDValue(ResN, 0));
```
- EN: It declares or implements routines such as findStrip, assert, isUndef, isIdentity, ... (29 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage. Notable Hexagon symbols referenced here include HexagonISD, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 findStrip, assert, isUndef, isIdentity, ... (29 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。 这里引用的重要 Hexagon 符号包括 HexagonISD，说明了它与同级后端组件的连接关系。

### Lines 1201-1400 / 第 1201-1400 行

```cpp
  1201:   }
  1202: 
  1203:   SDNode *OutN = Output.back().getNode();
  1204:   SDNode *InpN = Results.InpNode;
  1205:   DEBUG_WITH_TYPE("isel", {
  1206:     dbgs() << "Generated node:\n";
  1207:     OutN->dumpr(&DAG);
  1208:   });
  1209: 
  1210:   ISel.ReplaceNode(InpN, OutN);
  1211:   selectVectorConstants(OutN);
  1212:   DAG.RemoveDeadNodes();
  1213: }
  1214: 
  1215: OpRef HvxSelector::concats(OpRef Lo, OpRef Hi, ResultStack &Results) {
  1216:   DEBUG_WITH_TYPE("isel", {dbgs() << __func__ << '\n';});
  1217:   const SDLoc &dl(Results.InpNode);
  1218:   Results.push(TargetOpcode::REG_SEQUENCE, getPairVT(MVT::i8), {
  1219:     getConst32(Hexagon::HvxWRRegClassID, dl),
  1220:     Lo, getConst32(Hexagon::vsub_lo, dl),
  1221:     Hi, getConst32(Hexagon::vsub_hi, dl),
  1222:   });
  1223:   return OpRef::res(Results.top());
  1224: }
  1225: 
  1226: OpRef HvxSelector::funnels(OpRef Va, OpRef Vb, int Amount,
  1227:                            ResultStack &Results) {
  1228:   // Do a funnel shift towards the low end (shift right) by Amount bytes.
  1229:   // If Amount < 0, treat it as shift left, i.e. do a shift right by
  1230:   // Amount + HwLen.
  1231:   auto VecLen = static_cast<int>(HwLen);
  1232: 
  1233:   if (Amount == 0)
  1234:     return Va;
  1235:   if (Amount == VecLen)
  1236:     return Vb;
  1237: 
  1238:   MVT Ty = getSingleVT(MVT::i8);
  1239:   const SDLoc &dl(Results.InpNode);
  1240: 
  1241:   if (Amount < 0)
  1242:     Amount += VecLen;
  1243:   if (Amount > VecLen) {
  1244:     Amount -= VecLen;
  1245:     std::swap(Va, Vb);
  1246:   }
  1247: 
  1248:   if (isUInt<3>(Amount)) {
  1249:     SDValue A = getConst32(Amount, dl);
  1250:     Results.push(Hexagon::V6_valignbi, Ty, {Vb, Va, A});
  1251:   } else if (isUInt<3>(VecLen - Amount)) {
  1252:     SDValue A = getConst32(VecLen - Amount, dl);
  1253:     Results.push(Hexagon::V6_vlalignbi, Ty, {Vb, Va, A});
  1254:   } else {
  1255:     SDValue A = getConst32(Amount, dl);
  1256:     Results.push(Hexagon::A2_tfrsi, Ty, {A});
  1257:     Results.push(Hexagon::V6_valignb, Ty, {Vb, Va, OpRef::res(-1)});
  1258:   }
  1259:   return OpRef::res(Results.top());
  1260: }
  1261: 
  1262: // Va, Vb are single vectors. If SM only uses two vector halves from Va/Vb,
  1263: // pack these halves into a single vector, and remap SM into NewMask to use
  1264: // the new vector instead.
  1265: OpRef HvxSelector::packs(ShuffleMask SM, OpRef Va, OpRef Vb,
  1266:                          ResultStack &Results, MutableArrayRef<int> NewMask,
  1267:                          unsigned Options) {
  1268:   DEBUG_WITH_TYPE("isel", {dbgs() << __func__ << '\n';});
  1269:   if (!Va.isValid() || !Vb.isValid())
  1270:     return OpRef::fail();
  1271: 
  1272:   if (Vb.isUndef()) {
  1273:     llvm::copy(SM.Mask, NewMask.begin());
  1274:     return Va;
  1275:   }
  1276:   if (Va.isUndef()) {
  1277:     llvm::copy(SM.Mask, NewMask.begin());
  1278:     ShuffleVectorSDNode::commuteMask(NewMask);
  1279:     return Vb;
  1280:   }
  1281: 
  1282:   MVT Ty = getSingleVT(MVT::i8);
  1283:   MVT PairTy = getPairVT(MVT::i8);
  1284:   OpRef Inp[2] = {Va, Vb};
  1285:   unsigned VecLen = SM.Mask.size();
  1286: 
  1287:   auto valign = [this](OpRef Lo, OpRef Hi, unsigned Amt, MVT Ty,
  1288:                        ResultStack &Results) {
  1289:     if (Amt == 0)
  1290:       return Lo;
  1291:     const SDLoc &dl(Results.InpNode);
  1292:     if (isUInt<3>(Amt) || isUInt<3>(HwLen - Amt)) {
  1293:       bool IsRight = isUInt<3>(Amt); // Right align.
  1294:       SDValue S = getConst32(IsRight ? Amt : HwLen - Amt, dl);
  1295:       unsigned Opc = IsRight ? Hexagon::V6_valignbi : Hexagon::V6_vlalignbi;
  1296:       Results.push(Opc, Ty, {Hi, Lo, S});
  1297:       return OpRef::res(Results.top());
  1298:     }
  1299:     Results.push(Hexagon::A2_tfrsi, MVT::i32, {getConst32(Amt, dl)});
  1300:     OpRef A = OpRef::res(Results.top());
  1301:     Results.push(Hexagon::V6_valignb, Ty, {Hi, Lo, A});
  1302:     return OpRef::res(Results.top());
  1303:   };
  1304: 
  1305:   // Segment is a vector half.
  1306:   unsigned SegLen = HwLen / 2;
  1307: 
  1308:   // Check if we can shuffle vector halves around to get the used elements
  1309:   // into a single vector.
  1310:   shuffles::MaskT MaskH(SM.Mask);
  1311:   SmallVector<unsigned, 4> SegList = getInputSegmentList(SM.Mask, SegLen);
  1312:   unsigned SegCount = SegList.size();
  1313:   SmallVector<unsigned, 4> SegMap = getOutputSegmentMap(SM.Mask, SegLen);
  1314: 
  1315:   if (SegList.empty())
  1316:     return OpRef::undef(Ty);
  1317: 
  1318:   // NOTE:
  1319:   // In the following part of the function, where the segments are rearranged,
  1320:   // the shuffle mask SM can be of any length that is a multiple of a vector
  1321:   // (i.e. a multiple of 2*SegLen), and non-zero.
  1322:   // The output segment map is computed, and it may have any even number of
  1323:   // entries, but the rearrangement of input segments will be done based only
  1324:   // on the first two (non-undef) entries in the segment map.
  1325:   // For example, if the output map is 3, 1, 1, 3 (it can have at most two
  1326:   // distinct entries!), the segments 1 and 3 of Va/Vb will be packaged into
  1327:   // a single vector V = 3:1. The output mask will then be updated to use
  1328:   // seg(0,V), seg(1,V), seg(1,V), seg(0,V).
  1329:   //
  1330:   // Picking the segments based on the output map is an optimization. For
  1331:   // correctness it is only necessary that Seg0 and Seg1 are the two input
  1332:   // segments that are used in the output.
  1333: 
  1334:   unsigned Seg0 = ~0u, Seg1 = ~0u;
  1335:   for (unsigned X : SegMap) {
  1336:     if (X == ~0u)
  1337:       continue;
  1338:     if (Seg0 == ~0u)
  1339:       Seg0 = X;
  1340:     else if (Seg1 != ~0u)
  1341:       break;
  1342:     if (X == ~1u || X != Seg0)
  1343:       Seg1 = X;
  1344:   }
  1345: 
  1346:   if (SegCount == 1) {
  1347:     unsigned SrcOp = SegList[0] / 2;
  1348:     for (int I = 0; I != static_cast<int>(VecLen); ++I) {
  1349:       int M = SM.Mask[I];
  1350:       if (M >= 0) {
  1351:         M -= SrcOp * HwLen;
  1352:         assert(M >= 0);
  1353:       }
  1354:       NewMask[I] = M;
  1355:     }
  1356:     return Inp[SrcOp];
  1357:   }
  1358: 
  1359:   if (SegCount == 2) {
  1360:     // Seg0 should not be undef here: this would imply a SegList
  1361:     // with <= 1 elements, which was checked earlier.
  1362:     assert(Seg0 != ~0u);
  1363: 
  1364:     // If Seg0 or Seg1 are "multi-defined", pick them from the input
  1365:     // segment list instead.
  1366:     if (Seg0 == ~1u || Seg1 == ~1u) {
  1367:       if (Seg0 == Seg1) {
  1368:         Seg0 = SegList[0];
  1369:         Seg1 = SegList[1];
  1370:       } else if (Seg0 == ~1u) {
  1371:         Seg0 = SegList[0] != Seg1 ? SegList[0] : SegList[1];
  1372:       } else {
  1373:         assert(Seg1 == ~1u);
  1374:         Seg1 = SegList[0] != Seg0 ? SegList[0] : SegList[1];
  1375:       }
  1376:     }
  1377:     assert(Seg0 != ~1u && Seg1 != ~1u);
  1378: 
  1379:     assert(Seg0 != Seg1 && "Expecting different segments");
  1380:     const SDLoc &dl(Results.InpNode);
  1381:     Results.push(Hexagon::A2_tfrsi, MVT::i32, {getConst32(SegLen, dl)});
  1382:     OpRef HL = OpRef::res(Results.top());
  1383: 
  1384:     // Va = AB, Vb = CD
  1385: 
  1386:     if (Seg0 / 2 == Seg1 / 2) {
  1387:       // Same input vector.
  1388:       Va = Inp[Seg0 / 2];
  1389:       if (Seg0 > Seg1) {
  1390:         // Swap halves.
  1391:         Results.push(Hexagon::V6_vror, Ty, {Inp[Seg0 / 2], HL});
  1392:         Va = OpRef::res(Results.top());
  1393:       }
  1394:       packSegmentMask(SM.Mask, {Seg0, Seg1}, SegLen, MaskH);
  1395:     } else if (Seg0 % 2 == Seg1 % 2) {
  1396:       // Picking AC, BD, CA, or DB.
  1397:       // vshuff(CD,AB,HL) -> BD:AC
  1398:       // vshuff(AB,CD,HL) -> DB:CA
  1399:       auto Vs = (Seg0 == 0 || Seg0 == 1) ? std::make_pair(Vb, Va)  // AC or BD
  1400:                                          : std::make_pair(Va, Vb); // CA or DB
```
- EN: It declares or implements routines such as back, dumpr, ReplaceNode, selectVectorConstants, ... (25 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage.
- CN: 这里声明或实现了 back, dumpr, ReplaceNode, selectVectorConstants, ... (25 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。

### Lines 1401-1600 / 第 1401-1600 行

```cpp
  1401:       Results.push(Hexagon::V6_vshuffvdd, PairTy, {Vs.first, Vs.second, HL});
  1402:       OpRef P = OpRef::res(Results.top());
  1403:       Va = (Seg0 == 0 || Seg0 == 2) ? OpRef::lo(P) : OpRef::hi(P);
  1404:       packSegmentMask(SM.Mask, {Seg0, Seg1}, SegLen, MaskH);
  1405:     } else {
  1406:       // Picking AD, BC, CB, or DA.
  1407:       if ((Seg0 == 0 && Seg1 == 3) || (Seg0 == 2 && Seg1 == 1)) {
  1408:         // AD or BC: this can be done using vmux.
  1409:         // Q = V6_pred_scalar2 SegLen
  1410:         // V = V6_vmux Q, (Va, Vb) or (Vb, Va)
  1411:         Results.push(Hexagon::V6_pred_scalar2, getBoolVT(), {HL});
  1412:         OpRef Qt = OpRef::res(Results.top());
  1413:         auto Vs = (Seg0 == 0) ? std::make_pair(Va, Vb)  // AD
  1414:                               : std::make_pair(Vb, Va); // CB
  1415:         Results.push(Hexagon::V6_vmux, Ty, {Qt, Vs.first, Vs.second});
  1416:         Va = OpRef::res(Results.top());
  1417:         packSegmentMask(SM.Mask, {Seg0, Seg1}, SegLen, MaskH);
  1418:       } else {
  1419:         // BC or DA: this could be done via valign by SegLen.
  1420:         // Do nothing here, because valign (if possible) will be generated
  1421:         // later on (make sure the Seg0 values are as expected).
  1422:         assert(Seg0 == 1 || Seg0 == 3);
  1423:       }
  1424:     }
  1425:   }
  1426: 
  1427:   // Check if the arguments can be packed by valign(Va,Vb) or valign(Vb,Va).
  1428:   // FIXME: maybe remove this?
  1429:   ShuffleMask SMH(MaskH);
  1430:   assert(SMH.Mask.size() == VecLen);
  1431:   shuffles::MaskT MaskA(SMH.Mask);
  1432: 
  1433:   if (SMH.MaxSrc - SMH.MinSrc >= static_cast<int>(HwLen)) {
  1434:     // valign(Lo=Va,Hi=Vb) won't work. Try swapping Va/Vb.
  1435:     shuffles::MaskT Swapped(SMH.Mask);
  1436:     ShuffleVectorSDNode::commuteMask(Swapped);
  1437:     ShuffleMask SW(Swapped);
  1438:     if (SW.MaxSrc - SW.MinSrc < static_cast<int>(HwLen)) {
  1439:       MaskA.assign(SW.Mask.begin(), SW.Mask.end());
  1440:       std::swap(Va, Vb);
  1441:     }
  1442:   }
  1443:   ShuffleMask SMA(MaskA);
  1444:   assert(SMA.Mask.size() == VecLen);
  1445: 
  1446:   if (SMA.MaxSrc - SMA.MinSrc < static_cast<int>(HwLen)) {
  1447:     int ShiftR = SMA.MinSrc;
  1448:     if (ShiftR >= static_cast<int>(HwLen)) {
  1449:       Va = Vb;
  1450:       Vb = OpRef::undef(Ty);
  1451:       ShiftR -= HwLen;
  1452:     }
  1453:     OpRef RetVal = valign(Va, Vb, ShiftR, Ty, Results);
  1454: 
  1455:     for (int I = 0; I != static_cast<int>(VecLen); ++I) {
  1456:       int M = SMA.Mask[I];
  1457:       if (M != -1)
  1458:         M -= SMA.MinSrc;
  1459:       NewMask[I] = M;
  1460:     }
  1461:     return RetVal;
  1462:   }
  1463: 
  1464:   // By here, packing by segment (half-vector) shuffling, and vector alignment
  1465:   // failed. Try vmux.
  1466:   // Note: since this is using the original mask, Va and Vb must not have been
  1467:   // modified.
  1468: 
  1469:   if (Options & PackMux) {
  1470:     // If elements picked from Va and Vb have all different (source) indexes
  1471:     // (relative to the start of the argument), do a mux, and update the mask.
  1472:     BitVector Picked(HwLen);
  1473:     SmallVector<uint8_t,128> MuxBytes(HwLen);
  1474:     bool CanMux = true;
  1475:     for (int I = 0; I != static_cast<int>(VecLen); ++I) {
  1476:       int M = SM.Mask[I];
  1477:       if (M == -1)
  1478:         continue;
  1479:       if (M >= static_cast<int>(HwLen))
  1480:         M -= HwLen;
  1481:       else
  1482:         MuxBytes[M] = 0xFF;
  1483:       if (Picked[M]) {
  1484:         CanMux = false;
  1485:         break;
  1486:       }
  1487:       NewMask[I] = M;
  1488:     }
  1489:     if (CanMux)
  1490:       return vmuxs(MuxBytes, Va, Vb, Results);
  1491:   }
  1492:   return OpRef::fail();
  1493: }
  1494: 
  1495: // Va, Vb are vector pairs. If SM only uses two single vectors from Va/Vb,
  1496: // pack these vectors into a pair, and remap SM into NewMask to use the
  1497: // new pair instead.
  1498: OpRef HvxSelector::packp(ShuffleMask SM, OpRef Va, OpRef Vb,
  1499:                          ResultStack &Results, MutableArrayRef<int> NewMask) {
  1500:   DEBUG_WITH_TYPE("isel", {dbgs() << __func__ << '\n';});
  1501:   SmallVector<unsigned, 4> SegList = getInputSegmentList(SM.Mask, HwLen);
  1502:   if (SegList.empty())
  1503:     return OpRef::undef(getPairVT(MVT::i8));
  1504: 
  1505:   // If more than two halves are used, bail.
  1506:   // TODO: be more aggressive here?
  1507:   unsigned SegCount = SegList.size();
  1508:   if (SegCount > 2)
  1509:     return OpRef::fail();
  1510: 
  1511:   MVT HalfTy = getSingleVT(MVT::i8);
  1512: 
  1513:   OpRef Inp[2] = { Va, Vb };
  1514:   OpRef Out[2] = { OpRef::undef(HalfTy), OpRef::undef(HalfTy) };
  1515: 
  1516:   // Really make sure we have at most 2 vectors used in the mask.
  1517:   assert(SegCount <= 2);
  1518: 
  1519:   for (int I = 0, E = SegList.size(); I != E; ++I) {
  1520:     unsigned S = SegList[I];
  1521:     OpRef Op = Inp[S / 2];
  1522:     Out[I] = (S & 1) ? OpRef::hi(Op) : OpRef::lo(Op);
  1523:   }
  1524: 
  1525:   // NOTE: Using SegList as the packing map here (not SegMap). This works,
  1526:   // because we're not concerned here about the order of the segments (i.e.
  1527:   // single vectors) in the output pair. Changing the order of vectors is
  1528:   // free (as opposed to changing the order of vector halves as in packs),
  1529:   // and so there is no extra cost added in case the order needs to be
  1530:   // changed later.
  1531:   packSegmentMask(SM.Mask, SegList, HwLen, NewMask);
  1532:   return concats(Out[0], Out[1], Results);
  1533: }
  1534: 
  1535: OpRef HvxSelector::vmuxs(ArrayRef<uint8_t> Bytes, OpRef Va, OpRef Vb,
  1536:                          ResultStack &Results) {
  1537:   DEBUG_WITH_TYPE("isel", {dbgs() << __func__ << '\n';});
  1538:   MVT ByteTy = getSingleVT(MVT::i8);
  1539:   MVT BoolTy = MVT::getVectorVT(MVT::i1, HwLen);
  1540:   const SDLoc &dl(Results.InpNode);
  1541:   SDValue B = getVectorConstant(Bytes, dl);
  1542:   Results.push(Hexagon::V6_vd0, ByteTy, {});
  1543:   Results.push(Hexagon::V6_veqb, BoolTy, {OpRef(B), OpRef::res(-1)});
  1544:   Results.push(Hexagon::V6_vmux, ByteTy, {OpRef::res(-1), Vb, Va});
  1545:   return OpRef::res(Results.top());
  1546: }
  1547: 
  1548: OpRef HvxSelector::vmuxp(ArrayRef<uint8_t> Bytes, OpRef Va, OpRef Vb,
  1549:                          ResultStack &Results) {
  1550:   DEBUG_WITH_TYPE("isel", {dbgs() << __func__ << '\n';});
  1551:   size_t S = Bytes.size() / 2;
  1552:   OpRef L = vmuxs(Bytes.take_front(S), OpRef::lo(Va), OpRef::lo(Vb), Results);
  1553:   OpRef H = vmuxs(Bytes.drop_front(S), OpRef::hi(Va), OpRef::hi(Vb), Results);
  1554:   return concats(L, H, Results);
  1555: }
  1556: 
  1557: OpRef HvxSelector::shuffs1(ShuffleMask SM, OpRef Va, ResultStack &Results) {
  1558:   DEBUG_WITH_TYPE("isel", {dbgs() << __func__ << '\n';});
  1559:   unsigned VecLen = SM.Mask.size();
  1560:   assert(HwLen == VecLen);
  1561:   (void)VecLen;
  1562:   assert(all_of(SM.Mask, [this](int M) { return M == -1 || M < int(HwLen); }));
  1563: 
  1564:   if (isIdentity(SM.Mask))
  1565:     return Va;
  1566:   if (isUndef(SM.Mask))
  1567:     return OpRef::undef(getSingleVT(MVT::i8));
  1568: 
  1569:   // First, check for rotations.
  1570:   if (auto Dist = rotationDistance(SM, VecLen)) {
  1571:     OpRef Rotate = funnels(Va, Va, *Dist, Results);
  1572:     if (Rotate.isValid())
  1573:       return Rotate;
  1574:   }
  1575:   unsigned HalfLen = HwLen / 2;
  1576:   assert(isPowerOf2_32(HalfLen));
  1577: 
  1578:   // Handle special case where the output is the same half of the input
  1579:   // repeated twice, i.e. if Va = AB, then handle the output of AA or BB.
  1580:   std::pair<int, unsigned> Strip1 = findStrip(SM.Mask, 1, HalfLen);
  1581:   if ((Strip1.first & ~HalfLen) == 0 && Strip1.second == HalfLen) {
  1582:     std::pair<int, unsigned> Strip2 =
  1583:         findStrip(SM.Mask.drop_front(HalfLen), 1, HalfLen);
  1584:     if (Strip1 == Strip2) {
  1585:       const SDLoc &dl(Results.InpNode);
  1586:       Results.push(Hexagon::A2_tfrsi, MVT::i32, {getConst32(HalfLen, dl)});
  1587:       Results.push(Hexagon::V6_vshuffvdd, getPairVT(MVT::i8),
  1588:                    {Va, Va, OpRef::res(Results.top())});
  1589:       OpRef S = OpRef::res(Results.top());
  1590:       return (Strip1.first == 0) ? OpRef::lo(S) : OpRef::hi(S);
  1591:     }
  1592:   }
  1593: 
  1594:   OpRef P = perfect(SM, Va, Results);
  1595:   if (P.isValid())
  1596:     return P;
  1597:   return butterfly(SM, Va, Results);
  1598: }
  1599: 
  1600: OpRef HvxSelector::shuffs2(ShuffleMask SM, OpRef Va, OpRef Vb,
```
- EN: It declares or implements routines such as OpRef::res, OpRef::lo, std::make_pair, valign, ... (36 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage.
- CN: 这里声明或实现了 OpRef::res, OpRef::lo, std::make_pair, valign, ... (36 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。

### Lines 1601-1800 / 第 1601-1800 行

```cpp
  1601:                            ResultStack &Results) {
  1602:   DEBUG_WITH_TYPE("isel", {dbgs() << __func__ << '\n';});
  1603:   if (isUndef(SM.Mask))
  1604:     return OpRef::undef(getSingleVT(MVT::i8));
  1605: 
  1606:   OpRef C = contracting(SM, Va, Vb, Results);
  1607:   if (C.isValid())
  1608:     return C;
  1609: 
  1610:   int VecLen = SM.Mask.size();
  1611:   shuffles::MaskT PackedMask(VecLen);
  1612:   OpRef P = packs(SM, Va, Vb, Results, PackedMask);
  1613:   if (P.isValid())
  1614:     return shuffs1(ShuffleMask(PackedMask), P, Results);
  1615: 
  1616:   // TODO: Before we split the mask, try perfect shuffle on concatenated
  1617:   // operands.
  1618: 
  1619:   shuffles::MaskT MaskL(VecLen), MaskR(VecLen);
  1620:   splitMask(SM.Mask, MaskL, MaskR);
  1621: 
  1622:   OpRef L = shuffs1(ShuffleMask(MaskL), Va, Results);
  1623:   OpRef R = shuffs1(ShuffleMask(MaskR), Vb, Results);
  1624:   if (!L.isValid() || !R.isValid())
  1625:     return OpRef::fail();
  1626: 
  1627:   SmallVector<uint8_t, 128> Bytes(VecLen);
  1628:   for (int I = 0; I != VecLen; ++I) {
  1629:     if (MaskL[I] != -1)
  1630:       Bytes[I] = 0xFF;
  1631:   }
  1632:   return vmuxs(Bytes, L, R, Results);
  1633: }
  1634: 
  1635: OpRef HvxSelector::shuffp1(ShuffleMask SM, OpRef Va, ResultStack &Results) {
  1636:   DEBUG_WITH_TYPE("isel", {dbgs() << __func__ << '\n';});
  1637:   int VecLen = SM.Mask.size();
  1638: 
  1639:   if (isIdentity(SM.Mask))
  1640:     return Va;
  1641:   if (isUndef(SM.Mask))
  1642:     return OpRef::undef(getPairVT(MVT::i8));
  1643: 
  1644:   shuffles::MaskT PackedMask(VecLen);
  1645:   OpRef P = packs(SM, OpRef::lo(Va), OpRef::hi(Va), Results, PackedMask);
  1646:   if (P.isValid()) {
  1647:     ShuffleMask PM(PackedMask);
  1648:     OpRef E = expanding(PM, P, Results);
  1649:     if (E.isValid())
  1650:       return E;
  1651: 
  1652:     OpRef L = shuffs1(PM.lo(), P, Results);
  1653:     OpRef H = shuffs1(PM.hi(), P, Results);
  1654:     if (L.isValid() && H.isValid())
  1655:       return concats(L, H, Results);
  1656:   }
  1657: 
  1658:   if (!isLowHalfOnly(SM.Mask)) {
  1659:     // Doing a perfect shuffle on a low-half mask (i.e. where the upper half
  1660:     // is all-undef) may produce a perfect shuffle that generates legitimate
  1661:     // upper half. This isn't wrong, but if the perfect shuffle was possible,
  1662:     // then there is a good chance that a shorter (contracting) code may be
  1663:     // used as well (e.g. V6_vshuffeb, etc).
  1664:     OpRef R = perfect(SM, Va, Results);
  1665:     if (R.isValid())
  1666:       return R;
  1667:     // TODO commute the mask and try the opposite order of the halves.
  1668:   }
  1669: 
  1670:   OpRef L = shuffs2(SM.lo(), OpRef::lo(Va), OpRef::hi(Va), Results);
  1671:   OpRef H = shuffs2(SM.hi(), OpRef::lo(Va), OpRef::hi(Va), Results);
  1672:   if (L.isValid() && H.isValid())
  1673:     return concats(L, H, Results);
  1674: 
  1675:   return OpRef::fail();
  1676: }
  1677: 
  1678: OpRef HvxSelector::shuffp2(ShuffleMask SM, OpRef Va, OpRef Vb,
  1679:                            ResultStack &Results) {
  1680:   DEBUG_WITH_TYPE("isel", {dbgs() << __func__ << '\n';});
  1681:   if (isUndef(SM.Mask))
  1682:     return OpRef::undef(getPairVT(MVT::i8));
  1683: 
  1684:   int VecLen = SM.Mask.size();
  1685:   SmallVector<int,256> PackedMask(VecLen);
  1686:   OpRef P = packp(SM, Va, Vb, Results, PackedMask);
  1687:   if (P.isValid())
  1688:     return shuffp1(ShuffleMask(PackedMask), P, Results);
  1689: 
  1690:   SmallVector<int,256> MaskL(VecLen), MaskR(VecLen);
  1691:   splitMask(SM.Mask, MaskL, MaskR);
  1692: 
  1693:   OpRef L = shuffp1(ShuffleMask(MaskL), Va, Results);
  1694:   OpRef R = shuffp1(ShuffleMask(MaskR), Vb, Results);
  1695:   if (!L.isValid() || !R.isValid())
  1696:     return OpRef::fail();
  1697: 
  1698:   // Mux the results.
  1699:   SmallVector<uint8_t,256> Bytes(VecLen);
  1700:   for (int I = 0; I != VecLen; ++I) {
  1701:     if (MaskL[I] != -1)
  1702:       Bytes[I] = 0xFF;
  1703:   }
  1704:   return vmuxp(Bytes, L, R, Results);
  1705: }
  1706: 
  1707: namespace {
  1708:   struct Deleter : public SelectionDAG::DAGNodeDeletedListener {
  1709:     template <typename T>
  1710:     Deleter(SelectionDAG &D, T &C)
  1711:       : SelectionDAG::DAGNodeDeletedListener(D, [&C] (SDNode *N, SDNode *E) {
  1712:                                                   C.erase(N);
  1713:                                                 }) {}
  1714:   };
  1715: 
  1716:   template <typename T>
  1717:   struct NullifyingVector : public T {
  1718:     DenseMap<SDNode*, SDNode**> Refs;
  1719:     NullifyingVector(T &&V) : T(V) {
  1720:       for (unsigned i = 0, e = T::size(); i != e; ++i) {
  1721:         SDNode *&N = T::operator[](i);
  1722:         Refs[N] = &N;
  1723:       }
  1724:     }
  1725:     void erase(SDNode *N) {
  1726:       auto F = Refs.find(N);
  1727:       if (F != Refs.end())
  1728:         *F->second = nullptr;
  1729:     }
  1730:   };
  1731: }
  1732: 
  1733: void HvxSelector::select(SDNode *ISelN) {
  1734:   // What's important here is to select the right set of nodes. The main
  1735:   // selection algorithm loops over nodes in a topological order, i.e. users
  1736:   // are visited before their operands.
  1737:   //
  1738:   // It is an error to have an unselected node with a selected operand, and
  1739:   // there is an assertion in the main selector code to enforce that.
  1740:   //
  1741:   // Such a situation could occur if we selected a node, which is both a
  1742:   // subnode of ISelN, and a subnode of an unrelated (and yet unselected)
  1743:   // node in the DAG.
  1744:   assert(ISelN->getOpcode() == HexagonISD::ISEL);
  1745:   SDNode *N0 = ISelN->getOperand(0).getNode();
  1746: 
  1747:   // There could have been nodes created (i.e. inserted into the DAG)
  1748:   // that are now dead. Remove them, in case they use any of the nodes
  1749:   // to select (and make them look shared).
  1750:   DAG.RemoveDeadNodes();
  1751: 
  1752:   SetVector<SDNode *> SubNodes;
  1753: 
  1754:   if (!N0->isMachineOpcode()) {
  1755:     // Don't want to select N0 if it's shared with another node, except if
  1756:     // it's shared with other ISELs.
  1757:     auto IsISelN = [](SDNode *T) { return T->getOpcode() == HexagonISD::ISEL; };
  1758:     if (llvm::all_of(N0->users(), IsISelN))
  1759:       SubNodes.insert(N0);
  1760:   }
  1761:   if (SubNodes.empty()) {
  1762:     ISel.ReplaceNode(ISelN, N0);
  1763:     return;
  1764:   }
  1765: 
  1766:   // Need to manually select the nodes that are dominated by the ISEL. Other
  1767:   // nodes are reachable from the rest of the DAG, and so will be selected
  1768:   // by the DAG selection routine.
  1769:   SetVector<SDNode*> Dom, NonDom;
  1770:   Dom.insert(N0);
  1771: 
  1772:   auto IsDomRec = [&Dom, &NonDom] (SDNode *T, auto Rec) -> bool {
  1773:     if (Dom.count(T))
  1774:       return true;
  1775:     if (T->use_empty() || NonDom.count(T))
  1776:       return false;
  1777:     for (SDNode *U : T->users()) {
  1778:       // If T is reachable from a known non-dominated node, then T itself
  1779:       // is non-dominated.
  1780:       if (!Rec(U, Rec)) {
  1781:         NonDom.insert(T);
  1782:         return false;
  1783:       }
  1784:     }
  1785:     Dom.insert(T);
  1786:     return true;
  1787:   };
  1788: 
  1789:   auto IsDom = [&IsDomRec] (SDNode *T) { return IsDomRec(T, IsDomRec); };
  1790: 
  1791:   // Add the rest of nodes dominated by ISEL to SubNodes.
  1792:   for (unsigned I = 0; I != SubNodes.size(); ++I) {
  1793:     for (SDValue Op : SubNodes[I]->ops()) {
  1794:       SDNode *O = Op.getNode();
  1795:       if (IsDom(O))
  1796:         SubNodes.insert(O);
  1797:     }
  1798:   }
  1799: 
  1800:   // Do a topological sort of nodes from Dom.
```
- EN: It declares types such as Deleter, NullifyingVector, which carry the state or API of this component. It declares or implements routines such as contracting, size, PackedMask, packs, ... (31 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage.
- CN: 这里声明了 Deleter, NullifyingVector 等类型，用来承载该组件的状态或接口。 这里声明或实现了 contracting, size, PackedMask, packs, ... (31 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。

### Lines 1801-2000 / 第 1801-2000 行

```cpp
  1801:   SetVector<SDNode*> TmpQ;
  1802: 
  1803:   std::map<SDNode *, unsigned> OpCount;
  1804:   for (SDNode *T : Dom) {
  1805:     unsigned NumDomOps = llvm::count_if(T->ops(), [&Dom](const SDUse &U) {
  1806:                              return Dom.count(U.getNode());
  1807:                            });
  1808: 
  1809:     OpCount.insert({T, NumDomOps});
  1810:     if (NumDomOps == 0)
  1811:       TmpQ.insert(T);
  1812:   }
  1813: 
  1814:   for (unsigned I = 0; I != TmpQ.size(); ++I) {
  1815:     SDNode *S = TmpQ[I];
  1816:     for (SDNode *U : S->users()) {
  1817:       if (U == ISelN)
  1818:         continue;
  1819:       auto F = OpCount.find(U);
  1820:       assert(F != OpCount.end());
  1821:       if (F->second > 0 && !--F->second)
  1822:         TmpQ.insert(F->first);
  1823:     }
  1824:   }
  1825: 
  1826:   // Remove the marker.
  1827:   ISel.ReplaceNode(ISelN, N0);
  1828: 
  1829:   assert(SubNodes.size() == TmpQ.size());
  1830:   NullifyingVector<decltype(TmpQ)::vector_type> Queue(TmpQ.takeVector());
  1831: 
  1832:   Deleter DUQ(DAG, Queue);
  1833:   for (SDNode *S : reverse(Queue)) {
  1834:     if (S == nullptr)
  1835:       continue;
  1836:     DEBUG_WITH_TYPE("isel", {dbgs() << "HVX selecting: "; S->dump(&DAG);});
  1837:     ISel.Select(S);
  1838:   }
  1839: }
  1840: 
  1841: bool HvxSelector::scalarizeShuffle(ArrayRef<int> Mask, const SDLoc &dl,
  1842:                                    MVT ResTy, SDValue Va, SDValue Vb,
  1843:                                    SDNode *N) {
  1844:   DEBUG_WITH_TYPE("isel", {dbgs() << __func__ << '\n';});
  1845:   MVT ElemTy = ResTy.getVectorElementType();
  1846:   assert(ElemTy == MVT::i8);
  1847:   unsigned VecLen = Mask.size();
  1848:   bool HavePairs = (2*HwLen == VecLen);
  1849:   MVT SingleTy = getSingleVT(MVT::i8);
  1850: 
  1851:   // The prior attempts to handle this shuffle may have left a bunch of
  1852:   // dead nodes in the DAG (such as constants). These nodes will be added
  1853:   // at the end of DAG's node list, which at that point had already been
  1854:   // sorted topologically. In the main selection loop, the node list is
  1855:   // traversed backwards from the root node, which means that any new
  1856:   // nodes (from the end of the list) will not be visited.
  1857:   // Scalarization will replace the shuffle node with the scalarized
  1858:   // expression, and if that expression reused any if the leftoever (dead)
  1859:   // nodes, these nodes would not be selected (since the "local" selection
  1860:   // only visits nodes that are not in AllNodes).
  1861:   // To avoid this issue, remove all dead nodes from the DAG now.
  1862: //  DAG.RemoveDeadNodes();
  1863: 
  1864:   SmallVector<SDValue,128> Ops;
  1865:   LLVMContext &Ctx = *DAG.getContext();
  1866:   MVT LegalTy = Lower.getTypeToTransformTo(Ctx, ElemTy).getSimpleVT();
  1867:   for (int I : Mask) {
  1868:     if (I < 0) {
  1869:       Ops.push_back(ISel.selectUndef(dl, LegalTy));
  1870:       continue;
  1871:     }
  1872:     SDValue Vec;
  1873:     unsigned M = I;
  1874:     if (M < VecLen) {
  1875:       Vec = Va;
  1876:     } else {
  1877:       Vec = Vb;
  1878:       M -= VecLen;
  1879:     }
  1880:     if (HavePairs) {
  1881:       if (M < HwLen) {
  1882:         Vec = DAG.getTargetExtractSubreg(Hexagon::vsub_lo, dl, SingleTy, Vec);
  1883:       } else {
  1884:         Vec = DAG.getTargetExtractSubreg(Hexagon::vsub_hi, dl, SingleTy, Vec);
  1885:         M -= HwLen;
  1886:       }
  1887:     }
  1888:     SDValue Idx = DAG.getConstant(M, dl, MVT::i32);
  1889:     SDValue Ex = DAG.getNode(ISD::EXTRACT_VECTOR_ELT, dl, LegalTy, {Vec, Idx});
  1890:     SDValue L = Lower.LowerOperation(Ex, DAG);
  1891:     assert(L.getNode());
  1892:     Ops.push_back(L);
  1893:   }
  1894: 
  1895:   SDValue LV;
  1896:   if (2*HwLen == VecLen) {
  1897:     SDValue B0 = DAG.getBuildVector(SingleTy, dl, {Ops.data(), HwLen});
  1898:     SDValue L0 = Lower.LowerOperation(B0, DAG);
  1899:     SDValue B1 = DAG.getBuildVector(SingleTy, dl, {Ops.data()+HwLen, HwLen});
  1900:     SDValue L1 = Lower.LowerOperation(B1, DAG);
  1901:     // XXX CONCAT_VECTORS is legal for HVX vectors. Legalizing (lowering)
  1902:     // functions may expect to be called only for illegal operations, so
  1903:     // make sure that they are not called for legal ones. Develop a better
  1904:     // mechanism for dealing with this.
  1905:     LV = DAG.getNode(ISD::CONCAT_VECTORS, dl, ResTy, {L0, L1});
  1906:   } else {
  1907:     SDValue BV = DAG.getBuildVector(ResTy, dl, Ops);
  1908:     LV = Lower.LowerOperation(BV, DAG);
  1909:   }
  1910: 
  1911:   assert(!N->use_empty());
  1912:   SDValue IS = DAG.getNode(HexagonISD::ISEL, dl, ResTy, LV);
  1913:   ISel.ReplaceNode(N, IS.getNode());
  1914:   select(IS.getNode());
  1915:   DAG.RemoveDeadNodes();
  1916:   return true;
  1917: }
  1918: 
  1919: SmallVector<uint32_t, 8> HvxSelector::getPerfectCompletions(ShuffleMask SM,
  1920:                                                             unsigned Width) {
  1921:   auto possibilities = [](ArrayRef<uint8_t> Bs, unsigned Width) -> uint32_t {
  1922:     unsigned Impossible = ~(1u << Width) + 1;
  1923:     for (unsigned I = 0, E = Bs.size(); I != E; ++I) {
  1924:       uint8_t B = Bs[I];
  1925:       if (B == 0xff)
  1926:         continue;
  1927:       if (~Impossible == 0)
  1928:         break;
  1929:       for (unsigned Log = 0; Log != Width; ++Log) {
  1930:         if (Impossible & (1u << Log))
  1931:           continue;
  1932:         unsigned Expected = (I >> Log) % 2;
  1933:         if (B != Expected)
  1934:           Impossible |= (1u << Log);
  1935:       }
  1936:     }
  1937:     return ~Impossible;
  1938:   };
  1939: 
  1940:   SmallVector<uint32_t, 8> Worklist(Width);
  1941: 
  1942:   for (unsigned BitIdx = 0; BitIdx != Width; ++BitIdx) {
  1943:     SmallVector<uint8_t> BitValues(SM.Mask.size());
  1944:     for (int i = 0, e = SM.Mask.size(); i != e; ++i) {
  1945:       int M = SM.Mask[i];
  1946:       if (M < 0)
  1947:         BitValues[i] = 0xff;
  1948:       else
  1949:         BitValues[i] = (M & (1u << BitIdx)) != 0;
  1950:     }
  1951:     Worklist[BitIdx] = possibilities(BitValues, Width);
  1952:   }
  1953: 
  1954:   // If there is a word P in Worklist that matches multiple possibilities,
  1955:   // then if any other word Q matches any of the possibilities matched by P,
  1956:   // then Q matches all the possibilities matched by P. In fact, P == Q.
  1957:   // In other words, for each words P, Q, the sets of possibilities matched
  1958:   // by P and Q are either equal or disjoint (no partial overlap).
  1959:   //
  1960:   // Illustration: For 4-bit values there are 4 complete sequences:
  1961:   // a:  0 1 0 1  0 1 0 1  0 1 0 1  0 1 0 1
  1962:   // b:  0 0 1 1  0 0 1 1  0 0 1 1  0 0 1 1
  1963:   // c:  0 0 0 0  1 1 1 1  0 0 0 0  1 1 1 1
  1964:   // d:  0 0 0 0  0 0 0 0  1 1 1 1  1 1 1 1
  1965:   //
  1966:   // Words containing unknown bits that match two of the complete
  1967:   // sequences:
  1968:   // ab: 0 u u 1  0 u u 1  0 u u 1  0 u u 1
  1969:   // ac: 0 u 0 u  u 1 u 1  0 u 0 u  u 1 u 1
  1970:   // ad: 0 u 0 u  0 u 0 u  u 1 u 1  u 1 u 1
  1971:   // bc: 0 0 u u  u u 1 1  0 0 u u  u u 1 1
  1972:   // bd: 0 0 u u  0 0 u u  u u 1 1  u u 1 1
  1973:   // cd: 0 0 0 0  u u u u  u u u u  1 1 1 1
  1974:   //
  1975:   // Proof of the claim above:
  1976:   // Let P be a word that matches s0 and s1. For that to happen, all known
  1977:   // bits in P must match s0 and s1 exactly.
  1978:   // Assume there is Q that matches s1. Note that since P and Q came from
  1979:   // the same shuffle mask, the positions of unknown bits in P and Q match
  1980:   // exactly, which makes the indices of known bits be exactly the same
  1981:   // between P and Q. Since P matches s0 and s1, the known bits of P much
  1982:   // match both s0 and s1. Also, since Q matches s1, the known bits in Q
  1983:   // are exactly the same as in s1, which means that they are exactly the
  1984:   // same as in P. This implies that P == Q.
  1985: 
  1986:   // There can be a situation where there are more entries with the same
  1987:   // bits set than there are set bits (e.g. value 9 occurring more than 2
  1988:   // times). In such cases it will be impossible to complete this to a
  1989:   // perfect shuffle.
  1990:   SmallVector<uint32_t, 8> Sorted(Worklist);
  1991:   llvm::sort(Sorted);
  1992: 
  1993:   for (unsigned I = 0, E = Sorted.size(); I != E;) {
  1994:     unsigned P = Sorted[I], Count = 1;
  1995:     while (++I != E && P == Sorted[I])
  1996:       ++Count;
  1997:     if ((unsigned)llvm::popcount(P) < Count) {
  1998:       // Reset all occurrences of P, if there are more occurrences of P
  1999:       // than there are bits in P.
  2000:       llvm::replace(Worklist, P, 0U);
```
- EN: It declares or implements routines such as llvm::count_if, count, size, find, ... (31 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage. Notable Hexagon symbols referenced here include HexagonISD, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 llvm::count_if, count, size, find, ... (31 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。 这里引用的重要 Hexagon 符号包括 HexagonISD，说明了它与同级后端组件的连接关系。

### Lines 2001-2200 / 第 2001-2200 行

```cpp
  2001:     }
  2002:   }
  2003: 
  2004:   return Worklist;
  2005: }
  2006: 
  2007: SmallVector<uint32_t, 8>
  2008: HvxSelector::completeToPerfect(ArrayRef<uint32_t> Completions, unsigned Width) {
  2009:   // Pick a completion if there are multiple possibilities. For now just
  2010:   // select any valid completion.
  2011:   SmallVector<uint32_t, 8> Comps(Completions);
  2012: 
  2013:   for (unsigned I = 0; I != Width; ++I) {
  2014:     uint32_t P = Comps[I];
  2015:     assert(P != 0);
  2016:     if (isPowerOf2_32(P))
  2017:       continue;
  2018:     // T = least significant bit of P.
  2019:     uint32_t T = P ^ ((P - 1) & P);
  2020:     // Clear T in all remaining words matching P.
  2021:     for (unsigned J = I + 1; J != Width; ++J) {
  2022:       if (Comps[J] == P)
  2023:         Comps[J] ^= T;
  2024:     }
  2025:     Comps[I] = T;
  2026:   }
  2027: 
  2028: #ifndef NDEBUG
  2029:   // Check that we have generated a valid completion.
  2030:   uint32_t OrAll = 0;
  2031:   for (uint32_t C : Comps) {
  2032:     assert(isPowerOf2_32(C));
  2033:     OrAll |= C;
  2034:   }
  2035:   assert(OrAll == (1u << Width) -1);
  2036: #endif
  2037: 
  2038:   return Comps;
  2039: }
  2040: 
  2041: std::optional<int> HvxSelector::rotationDistance(ShuffleMask SM,
  2042:                                                  unsigned WrapAt) {
  2043:   std::optional<int> Dist;
  2044:   for (int I = 0, E = SM.Mask.size(); I != E; ++I) {
  2045:     int M = SM.Mask[I];
  2046:     if (M < 0)
  2047:       continue;
  2048:     if (Dist) {
  2049:       if ((I + *Dist) % static_cast<int>(WrapAt) != M)
  2050:         return std::nullopt;
  2051:     } else {
  2052:       // Integer a%b operator assumes rounding towards zero by /, so it
  2053:       // "misbehaves" when a crosses 0 (the remainder also changes sign).
  2054:       // Add WrapAt in an attempt to keep I+Dist non-negative.
  2055:       Dist = M - I;
  2056:       if (Dist < 0)
  2057:         Dist = *Dist + WrapAt;
  2058:     }
  2059:   }
  2060:   return Dist;
  2061: }
  2062: 
  2063: OpRef HvxSelector::contracting(ShuffleMask SM, OpRef Va, OpRef Vb,
  2064:                                ResultStack &Results) {
  2065:   DEBUG_WITH_TYPE("isel", { dbgs() << __func__ << '\n'; });
  2066:   if (!Va.isValid() || !Vb.isValid())
  2067:     return OpRef::fail();
  2068: 
  2069:   // Contracting shuffles, i.e. instructions that always discard some bytes
  2070:   // from the operand vectors.
  2071:   //
  2072:   // Funnel shifts
  2073:   // V6_vshuff{e,o}b
  2074:   // V6_vshuf{e,o}h
  2075:   // V6_vdealb4w
  2076:   // V6_vpack{e,o}{b,h}
  2077: 
  2078:   int VecLen = SM.Mask.size();
  2079: 
  2080:   // First, check for funnel shifts.
  2081:   if (auto Dist = rotationDistance(SM, 2 * VecLen)) {
  2082:     OpRef Funnel = funnels(Va, Vb, *Dist, Results);
  2083:     if (Funnel.isValid())
  2084:       return Funnel;
  2085:   }
  2086: 
  2087:   MVT SingleTy = getSingleVT(MVT::i8);
  2088:   MVT PairTy = getPairVT(MVT::i8);
  2089: 
  2090:   auto same = [](ArrayRef<int> Mask1, ArrayRef<int> Mask2) -> bool {
  2091:     return Mask1 == Mask2;
  2092:   };
  2093: 
  2094:   using PackConfig = std::pair<unsigned, bool>;
  2095:   PackConfig Packs[] = {
  2096:       {1, false}, // byte, even
  2097:       {1, true},  // byte, odd
  2098:       {2, false}, // half, even
  2099:       {2, true},  // half, odd
  2100:   };
  2101: 
  2102:   { // Check vpack
  2103:     unsigned Opcodes[] = {
  2104:         Hexagon::V6_vpackeb,
  2105:         Hexagon::V6_vpackob,
  2106:         Hexagon::V6_vpackeh,
  2107:         Hexagon::V6_vpackoh,
  2108:     };
  2109:     for (int i = 0, e = std::size(Opcodes); i != e; ++i) {
  2110:       auto [Size, Odd] = Packs[i];
  2111:       if (same(SM.Mask, shuffles::mask(shuffles::vpack, HwLen, Size, Odd))) {
  2112:         Results.push(Opcodes[i], SingleTy, {Vb, Va});
  2113:         return OpRef::res(Results.top());
  2114:       }
  2115:     }
  2116:   }
  2117: 
  2118:   { // Check vshuff
  2119:     unsigned Opcodes[] = {
  2120:         Hexagon::V6_vshuffeb,
  2121:         Hexagon::V6_vshuffob,
  2122:         Hexagon::V6_vshufeh,
  2123:         Hexagon::V6_vshufoh,
  2124:     };
  2125:     for (int i = 0, e = std::size(Opcodes); i != e; ++i) {
  2126:       auto [Size, Odd] = Packs[i];
  2127:       if (same(SM.Mask, shuffles::mask(shuffles::vshuff, HwLen, Size, Odd))) {
  2128:         Results.push(Opcodes[i], SingleTy, {Vb, Va});
  2129:         return OpRef::res(Results.top());
  2130:       }
  2131:     }
  2132:   }
  2133: 
  2134:   { // Check vdeal
  2135:     // There is no "V6_vdealeb", etc, but the supposed behavior of vdealeb
  2136:     // is equivalent to "(V6_vpackeb (V6_vdealvdd Vu, Vv, -2))". Other such
  2137:     // variants of "deal" can be done similarly.
  2138:     unsigned Opcodes[] = {
  2139:         Hexagon::V6_vpackeb,
  2140:         Hexagon::V6_vpackob,
  2141:         Hexagon::V6_vpackeh,
  2142:         Hexagon::V6_vpackoh,
  2143:     };
  2144:     const SDLoc &dl(Results.InpNode);
  2145: 
  2146:     for (int i = 0, e = std::size(Opcodes); i != e; ++i) {
  2147:       auto [Size, Odd] = Packs[i];
  2148:       if (same(SM.Mask, shuffles::mask(shuffles::vdeal, HwLen, Size, Odd))) {
  2149:         Results.push(Hexagon::A2_tfrsi, MVT::i32,
  2150:                      {getSignedConst32(-2 * Size, dl)});
  2151:         Results.push(Hexagon::V6_vdealvdd, PairTy, {Vb, Va, OpRef::res(-1)});
  2152:         auto vdeal = OpRef::res(Results.top());
  2153:         Results.push(Opcodes[i], SingleTy,
  2154:                      {OpRef::hi(vdeal), OpRef::lo(vdeal)});
  2155:         return OpRef::res(Results.top());
  2156:       }
  2157:     }
  2158:   }
  2159: 
  2160:   if (same(SM.Mask, shuffles::mask(shuffles::vdealb4w, HwLen))) {
  2161:     Results.push(Hexagon::V6_vdealb4w, SingleTy, {Vb, Va});
  2162:     return OpRef::res(Results.top());
  2163:   }
  2164: 
  2165:   return OpRef::fail();
  2166: }
  2167: 
  2168: OpRef HvxSelector::expanding(ShuffleMask SM, OpRef Va, ResultStack &Results) {
  2169:   DEBUG_WITH_TYPE("isel", {dbgs() << __func__ << '\n';});
  2170:   // Expanding shuffles (using all elements and inserting into larger vector):
  2171:   //
  2172:   // V6_vunpacku{b,h} [*]
  2173:   //
  2174:   // [*] Only if the upper elements (filled with 0s) are "don't care" in Mask.
  2175:   //
  2176:   // Note: V6_vunpacko{b,h} are or-ing the high byte/half in the result, so
  2177:   // they are not shuffles.
  2178:   //
  2179:   // The argument is a single vector.
  2180: 
  2181:   int VecLen = SM.Mask.size();
  2182:   assert(2*HwLen == unsigned(VecLen) && "Expecting vector-pair type");
  2183: 
  2184:   std::pair<int,unsigned> Strip = findStrip(SM.Mask, 1, VecLen);
  2185: 
  2186:   // The patterns for the unpacks, in terms of the starting offsets of the
  2187:   // consecutive strips (L = length of the strip, N = VecLen):
  2188:   //
  2189:   // vunpacku:  0, -1, L, -1, 2L, -1 ...
  2190: 
  2191:   if (Strip.first != 0)
  2192:     return OpRef::fail();
  2193: 
  2194:   // The vunpackus only handle byte and half-word.
  2195:   if (Strip.second != 1 && Strip.second != 2)
  2196:     return OpRef::fail();
  2197: 
  2198:   int N = VecLen;
  2199:   int L = Strip.second;
  2200: 
```
- EN: Header guards in this range prevent duplicate inclusion and define the interface boundary. It declares or implements routines such as HvxSelector::completeToPerfect, Comps, assert, HvxSelector::rotationDistance, ... (15 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage.
- CN: 这一段中的头文件保护宏用于防止重复包含，并明确接口边界。 这里声明或实现了 HvxSelector::completeToPerfect, Comps, assert, HvxSelector::rotationDistance, ... (15 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。

### Lines 2201-2400 / 第 2201-2400 行

```cpp
  2201:   // First, check the non-ignored strips.
  2202:   for (int I = 2*L; I < N; I += 2*L) {
  2203:     auto S = findStrip(SM.Mask.drop_front(I), 1, N-I);
  2204:     if (S.second != unsigned(L))
  2205:       return OpRef::fail();
  2206:     if (2*S.first != I)
  2207:       return OpRef::fail();
  2208:   }
  2209:   // Check the -1s.
  2210:   for (int I = L; I < N; I += 2*L) {
  2211:     auto S = findStrip(SM.Mask.drop_front(I), 0, N-I);
  2212:     if (S.first != -1 || S.second != unsigned(L))
  2213:       return OpRef::fail();
  2214:   }
  2215: 
  2216:   unsigned Opc = Strip.second == 1 ? Hexagon::V6_vunpackub
  2217:                                    : Hexagon::V6_vunpackuh;
  2218:   Results.push(Opc, getPairVT(MVT::i8), {Va});
  2219:   return OpRef::res(Results.top());
  2220: }
  2221: 
  2222: OpRef HvxSelector::perfect(ShuffleMask SM, OpRef Va, ResultStack &Results) {
  2223:   DEBUG_WITH_TYPE("isel", { dbgs() << __func__ << '\n'; });
  2224:   // V6_vdeal{b,h}
  2225:   // V6_vshuff{b,h}
  2226: 
  2227:   // V6_vshufoe{b,h}  those are equivalent to vshuffvdd(..,{1,2})
  2228:   // V6_vshuffvdd (V6_vshuff)
  2229:   // V6_dealvdd (V6_vdeal)
  2230: 
  2231:   int VecLen = SM.Mask.size();
  2232:   assert(isPowerOf2_32(VecLen) && Log2_32(VecLen) <= 8);
  2233:   unsigned LogLen = Log2_32(VecLen);
  2234:   unsigned HwLog = Log2_32(HwLen);
  2235:   // The result length must be the same as the length of a single vector,
  2236:   // or a vector pair.
  2237:   assert(LogLen == HwLog || LogLen == HwLog + 1);
  2238:   bool HavePairs = LogLen == HwLog + 1;
  2239: 
  2240:   SmallVector<unsigned, 8> Perm(LogLen);
  2241: 
  2242:   // Check if this could be a perfect shuffle, or a combination of perfect
  2243:   // shuffles.
  2244:   //
  2245:   // Consider this permutation (using hex digits to make the ASCII diagrams
  2246:   // easier to read):
  2247:   //   { 0, 8, 1, 9, 2, A, 3, B, 4, C, 5, D, 6, E, 7, F }.
  2248:   // This is a "deal" operation: divide the input into two halves, and
  2249:   // create the output by picking elements by alternating between these two
  2250:   // halves:
  2251:   //   0 1 2 3 4 5 6 7    -->    0 8 1 9 2 A 3 B 4 C 5 D 6 E 7 F  [*]
  2252:   //   8 9 A B C D E F
  2253:   //
  2254:   // Aside from a few special explicit cases (V6_vdealb, etc.), HVX provides
  2255:   // a somwehat different mechanism that could be used to perform shuffle/
  2256:   // deal operations: a 2x2 transpose.
  2257:   // Consider the halves of inputs again, they can be interpreted as a 2x8
  2258:   // matrix. A 2x8 matrix can be looked at four 2x2 matrices concatenated
  2259:   // together. Now, when considering 2 elements at a time, it will be a 2x4
  2260:   // matrix (with elements 01, 23, 45, etc.), or two 2x2 matrices:
  2261:   //   01 23  45 67
  2262:   //   89 AB  CD EF
  2263:   // With groups of 4, this will become a single 2x2 matrix, and so on.
  2264:   //
  2265:   // The 2x2 transpose instruction works by transposing each of the 2x2
  2266:   // matrices (or "sub-matrices"), given a specific group size. For example,
  2267:   // if the group size is 1 (i.e. each element is its own group), there
  2268:   // will be four transposes of the four 2x2 matrices that form the 2x8.
  2269:   // For example, with the inputs as above, the result will be:
  2270:   //   0 8  2 A  4 C  6 E
  2271:   //   1 9  3 B  5 D  7 F
  2272:   // Now, this result can be transposed again, but with the group size of 2:
  2273:   //   08 19  4C 5D
  2274:   //   2A 3B  6E 7F
  2275:   // If we then transpose that result, but with the group size of 4, we get:
  2276:   //   0819 2A3B
  2277:   //   4C5D 6E7F
  2278:   // If we concatenate these two rows, it will be
  2279:   //   0 8 1 9 2 A 3 B 4 C 5 D 6 E 7 F
  2280:   // which is the same as the "deal" [*] above.
  2281:   //
  2282:   // In general, a "deal" of individual elements is a series of 2x2 transposes,
  2283:   // with changing group size. HVX has two instructions:
  2284:   //   Vdd = V6_vdealvdd Vu, Vv, Rt
  2285:   //   Vdd = V6_shufvdd  Vu, Vv, Rt
  2286:   // that perform exactly that. The register Rt controls which transposes are
  2287:   // going to happen: a bit at position n (counting from 0) indicates that a
  2288:   // transpose with a group size of 2^n will take place. If multiple bits are
  2289:   // set, multiple transposes will happen: vdealvdd will perform them starting
  2290:   // with the largest group size, vshuffvdd will do them in the reverse order.
  2291:   //
  2292:   // The main observation is that each 2x2 transpose corresponds to swapping
  2293:   // columns of bits in the binary representation of the values.
  2294:   //
  2295:   // The numbers {3,2,1,0} and the log2 of the number of contiguous 1 bits
  2296:   // in a given column. The * denote the columns that will be swapped.
  2297:   // The transpose with the group size 2^n corresponds to swapping columns
  2298:   // 3 (the highest log) and log2(n):
  2299:   //
  2300:   //     3 2 1 0         0 2 1 3         0 2 3 1
  2301:   //     *     *             * *           * *
  2302:   //  0  0 0 0 0      0  0 0 0 0      0  0 0 0 0      0  0 0 0 0
  2303:   //  1  0 0 0 1      8  1 0 0 0      8  1 0 0 0      8  1 0 0 0
  2304:   //  2  0 0 1 0      2  0 0 1 0      1  0 0 0 1      1  0 0 0 1
  2305:   //  3  0 0 1 1      A  1 0 1 0      9  1 0 0 1      9  1 0 0 1
  2306:   //  4  0 1 0 0      4  0 1 0 0      4  0 1 0 0      2  0 0 1 0
  2307:   //  5  0 1 0 1      C  1 1 0 0      C  1 1 0 0      A  1 0 1 0
  2308:   //  6  0 1 1 0      6  0 1 1 0      5  0 1 0 1      3  0 0 1 1
  2309:   //  7  0 1 1 1      E  1 1 1 0      D  1 1 0 1      B  1 0 1 1
  2310:   //  8  1 0 0 0      1  0 0 0 1      2  0 0 1 0      4  0 1 0 0
  2311:   //  9  1 0 0 1      9  1 0 0 1      A  1 0 1 0      C  1 1 0 0
  2312:   //  A  1 0 1 0      3  0 0 1 1      3  0 0 1 1      5  0 1 0 1
  2313:   //  B  1 0 1 1      B  1 0 1 1      B  1 0 1 1      D  1 1 0 1
  2314:   //  C  1 1 0 0      5  0 1 0 1      6  0 1 1 0      6  0 1 1 0
  2315:   //  D  1 1 0 1      D  1 1 0 1      E  1 1 1 0      E  1 1 1 0
  2316:   //  E  1 1 1 0      7  0 1 1 1      7  0 1 1 1      7  0 1 1 1
  2317:   //  F  1 1 1 1      F  1 1 1 1      F  1 1 1 1      F  1 1 1 1
  2318: 
  2319:   // There is one special case that is not a perfect shuffle, but can be
  2320:   // turned into one easily: when the shuffle operates on a vector pair,
  2321:   // but the two vectors in the pair are swapped. The code that identifies
  2322:   // perfect shuffles will reject it, unless the order is reversed.
  2323:   shuffles::MaskT MaskStorage(SM.Mask);
  2324:   bool InvertedPair = false;
  2325:   if (HavePairs && SM.Mask[0] >= int(HwLen)) {
  2326:     for (int i = 0, e = SM.Mask.size(); i != e; ++i) {
  2327:       int M = SM.Mask[i];
  2328:       MaskStorage[i] = M >= int(HwLen) ? M - HwLen : M + HwLen;
  2329:     }
  2330:     InvertedPair = true;
  2331:     SM = ShuffleMask(MaskStorage);
  2332:   }
  2333: 
  2334:   auto Comps = getPerfectCompletions(SM, LogLen);
  2335:   if (llvm::is_contained(Comps, 0))
  2336:     return OpRef::fail();
  2337: 
  2338:   auto Pick = completeToPerfect(Comps, LogLen);
  2339:   for (unsigned I = 0; I != LogLen; ++I)
  2340:     Perm[I] = Log2_32(Pick[I]);
  2341: 
  2342:   // Once we have Perm, represent it as cycles. Denote the maximum log2
  2343:   // (equal to log2(VecLen)-1) as M. The cycle containing M can then be
  2344:   // written as (M a1 a2 a3 ... an). That cycle can be broken up into
  2345:   // simple swaps as (M a1)(M a2)(M a3)...(M an), with the composition
  2346:   // order being from left to right. Any (contiguous) segment where the
  2347:   // values ai, ai+1...aj are either all increasing or all decreasing,
  2348:   // can be implemented via a single vshuffvdd/vdealvdd respectively.
  2349:   //
  2350:   // If there is a cycle (a1 a2 ... an) that does not involve M, it can
  2351:   // be written as (M an)(a1 a2 ... an)(M a1). The first two cycles can
  2352:   // then be folded to get (M a1 a2 ... an)(M a1), and the above procedure
  2353:   // can be used to generate a sequence of vshuffvdd/vdealvdd.
  2354:   //
  2355:   // Example:
  2356:   // Assume M = 4 and consider a permutation (0 1)(2 3). It can be written
  2357:   // as (4 0 1)(4 0) composed with (4 2 3)(4 2), or simply
  2358:   //   (4 0 1)(4 0)(4 2 3)(4 2).
  2359:   // It can then be expanded into swaps as
  2360:   //   (4 0)(4 1)(4 0)(4 2)(4 3)(4 2),
  2361:   // and broken up into "increasing" segments as
  2362:   //   [(4 0)(4 1)] [(4 0)(4 2)(4 3)] [(4 2)].
  2363:   // This is equivalent to
  2364:   //   (4 0 1)(4 0 2 3)(4 2),
  2365:   // which can be implemented as 3 vshufvdd instructions.
  2366: 
  2367:   using CycleType = SmallVector<unsigned, 8>;
  2368:   std::set<CycleType> Cycles;
  2369:   std::set<unsigned> All;
  2370: 
  2371:   for (unsigned I : Perm)
  2372:     All.insert(I);
  2373: 
  2374:   // If the cycle contains LogLen-1, move it to the front of the cycle.
  2375:   // Otherwise, return the cycle unchanged.
  2376:   auto canonicalize = [LogLen](const CycleType &C) -> CycleType {
  2377:     unsigned LogPos, N = C.size();
  2378:     for (LogPos = 0; LogPos != N; ++LogPos)
  2379:       if (C[LogPos] == LogLen - 1)
  2380:         break;
  2381:     if (LogPos == N)
  2382:       return C;
  2383: 
  2384:     CycleType NewC(C.begin() + LogPos, C.end());
  2385:     NewC.append(C.begin(), C.begin() + LogPos);
  2386:     return NewC;
  2387:   };
  2388: 
  2389:   auto pfs = [](const std::set<CycleType> &Cs, unsigned Len) {
  2390:     // Ordering: shuff: 5 0 1 2 3 4, deal: 5 4 3 2 1 0 (for Log=6),
  2391:     // for bytes zero is included, for halfwords is not.
  2392:     if (Cs.size() != 1)
  2393:       return 0u;
  2394:     const CycleType &C = *Cs.begin();
  2395:     if (C[0] != Len - 1)
  2396:       return 0u;
  2397:     int D = Len - C.size();
  2398:     if (D != 0 && D != 1)
  2399:       return 0u;
  2400: 
```
- EN: This range is dominated by comments or banner text that documents the surrounding section. It declares or implements routines such as findStrip, OpRef::res, HvxSelector::perfect, V6_vshuffvdd, ... (15 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage.
- CN: 这一段主要由注释或分隔说明组成，用来解释后续代码区域。 这里声明或实现了 findStrip, OpRef::res, HvxSelector::perfect, V6_vshuffvdd, ... (15 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。

### Lines 2401-2600 / 第 2401-2600 行

```cpp
  2401:     bool IsDeal = true, IsShuff = true;
  2402:     for (unsigned I = 1; I != Len - D; ++I) {
  2403:       if (C[I] != Len - 1 - I)
  2404:         IsDeal = false;
  2405:       if (C[I] != I - (1 - D)) // I-1, I
  2406:         IsShuff = false;
  2407:     }
  2408:     // At most one, IsDeal or IsShuff, can be non-zero.
  2409:     assert(!(IsDeal || IsShuff) || IsDeal != IsShuff);
  2410:     static unsigned Deals[] = {Hexagon::V6_vdealb, Hexagon::V6_vdealh};
  2411:     static unsigned Shufs[] = {Hexagon::V6_vshuffb, Hexagon::V6_vshuffh};
  2412:     return IsDeal ? Deals[D] : (IsShuff ? Shufs[D] : 0);
  2413:   };
  2414: 
  2415:   while (!All.empty()) {
  2416:     unsigned A = *All.begin();
  2417:     All.erase(A);
  2418:     CycleType C;
  2419:     C.push_back(A);
  2420:     for (unsigned B = Perm[A]; B != A; B = Perm[B]) {
  2421:       C.push_back(B);
  2422:       All.erase(B);
  2423:     }
  2424:     if (C.size() <= 1)
  2425:       continue;
  2426:     Cycles.insert(canonicalize(C));
  2427:   }
  2428: 
  2429:   MVT SingleTy = getSingleVT(MVT::i8);
  2430:   MVT PairTy = getPairVT(MVT::i8);
  2431: 
  2432:   // Recognize patterns for V6_vdeal{b,h} and V6_vshuff{b,h}.
  2433:   if (unsigned(VecLen) == HwLen) {
  2434:     if (unsigned SingleOpc = pfs(Cycles, LogLen)) {
  2435:       Results.push(SingleOpc, SingleTy, {Va});
  2436:       return OpRef::res(Results.top());
  2437:     }
  2438:   }
  2439: 
  2440:   // From the cycles, construct the sequence of values that will
  2441:   // then form the control values for vdealvdd/vshuffvdd, i.e.
  2442:   // (M a1 a2)(M a3 a4 a5)... -> a1 a2 a3 a4 a5
  2443:   // This essentially strips the M value from the cycles where
  2444:   // it's present, and performs the insertion of M (then stripping)
  2445:   // for cycles without M (as described in an earlier comment).
  2446:   SmallVector<unsigned, 8> SwapElems;
  2447:   // When the input is extended (i.e. single vector becomes a pair),
  2448:   // this is done by using an "undef" vector as the second input.
  2449:   // However, then we get
  2450:   //   input 1: GOODBITS
  2451:   //   input 2: ........
  2452:   // but we need
  2453:   //   input 1: ....BITS
  2454:   //   input 2: ....GOOD
  2455:   // Then at the end, this needs to be undone. To accomplish this,
  2456:   // artificially add "LogLen-1" at both ends of the sequence.
  2457:   if (!HavePairs)
  2458:     SwapElems.push_back(LogLen - 1);
  2459:   for (const CycleType &C : Cycles) {
  2460:     // Do the transformation: (a1..an) -> (M a1..an)(M a1).
  2461:     unsigned First = (C[0] == LogLen - 1) ? 1 : 0;
  2462:     SwapElems.append(C.begin() + First, C.end());
  2463:     if (First == 0)
  2464:       SwapElems.push_back(C[0]);
  2465:   }
  2466:   if (!HavePairs)
  2467:     SwapElems.push_back(LogLen - 1);
  2468: 
  2469:   const SDLoc &dl(Results.InpNode);
  2470:   OpRef Arg = HavePairs ? Va : concats(Va, OpRef::undef(SingleTy), Results);
  2471:   if (InvertedPair)
  2472:     Arg = concats(OpRef::hi(Arg), OpRef::lo(Arg), Results);
  2473: 
  2474:   for (unsigned I = 0, E = SwapElems.size(); I != E;) {
  2475:     bool IsInc = I == E - 1 || SwapElems[I] < SwapElems[I + 1];
  2476:     unsigned S = (1u << SwapElems[I]);
  2477:     if (I < E - 1) {
  2478:       while (++I < E - 1 && IsInc == (SwapElems[I] < SwapElems[I + 1]))
  2479:         S |= 1u << SwapElems[I];
  2480:       // The above loop will not add a bit for the final SwapElems[I+1],
  2481:       // so add it here.
  2482:       S |= 1u << SwapElems[I];
  2483:     }
  2484:     ++I;
  2485: 
  2486:     // Upper bits of the vdeal/vshuff parameter that do not cover any byte in
  2487:     // the vector are ignored. Technically, A2_tfrsi takes a signed value, which
  2488:     // is sign-extended to 32 bit if there is no extender. The practical
  2489:     // advantages are that signed values are smaller in common use cases and are
  2490:     // not sensitive to the vector size.
  2491:     int SS = SignExtend32(S, HwLog);
  2492: 
  2493:     NodeTemplate Res;
  2494:     Results.push(Hexagon::A2_tfrsi, MVT::i32, {getSignedConst32(SS, dl)});
  2495:     Res.Opc = IsInc ? Hexagon::V6_vshuffvdd : Hexagon::V6_vdealvdd;
  2496:     Res.Ty = PairTy;
  2497:     Res.Ops = {OpRef::hi(Arg), OpRef::lo(Arg), OpRef::res(-1)};
  2498:     Results.push(Res);
  2499:     Arg = OpRef::res(Results.top());
  2500:   }
  2501: 
  2502:   return HavePairs ? Arg : OpRef::lo(Arg);
  2503: }
  2504: 
  2505: OpRef HvxSelector::butterfly(ShuffleMask SM, OpRef Va, ResultStack &Results) {
  2506:   DEBUG_WITH_TYPE("isel", {dbgs() << __func__ << '\n';});
  2507:   // Butterfly shuffles.
  2508:   //
  2509:   // V6_vdelta
  2510:   // V6_vrdelta
  2511:   // V6_vror
  2512: 
  2513:   // The assumption here is that all elements picked by Mask are in the
  2514:   // first operand to the vector_shuffle. This assumption is enforced
  2515:   // by the caller.
  2516: 
  2517:   MVT ResTy = getSingleVT(MVT::i8);
  2518:   PermNetwork::Controls FC, RC;
  2519:   const SDLoc &dl(Results.InpNode);
  2520:   int VecLen = SM.Mask.size();
  2521: 
  2522:   for (int M : SM.Mask) {
  2523:     if (M != -1 && M >= VecLen)
  2524:       return OpRef::fail();
  2525:   }
  2526: 
  2527:   // Try the deltas/benes for both single vectors and vector pairs.
  2528:   ForwardDeltaNetwork FN(SM.Mask);
  2529:   if (FN.run(FC)) {
  2530:     SDValue Ctl = getVectorConstant(FC, dl);
  2531:     Results.push(Hexagon::V6_vdelta, ResTy, {Va, OpRef(Ctl)});
  2532:     return OpRef::res(Results.top());
  2533:   }
  2534: 
  2535:   // Try reverse delta.
  2536:   ReverseDeltaNetwork RN(SM.Mask);
  2537:   if (RN.run(RC)) {
  2538:     SDValue Ctl = getVectorConstant(RC, dl);
  2539:     Results.push(Hexagon::V6_vrdelta, ResTy, {Va, OpRef(Ctl)});
  2540:     return OpRef::res(Results.top());
  2541:   }
  2542: 
  2543:   // Do Benes.
  2544:   BenesNetwork BN(SM.Mask);
  2545:   if (BN.run(FC, RC)) {
  2546:     SDValue CtlF = getVectorConstant(FC, dl);
  2547:     SDValue CtlR = getVectorConstant(RC, dl);
  2548:     Results.push(Hexagon::V6_vdelta, ResTy, {Va, OpRef(CtlF)});
  2549:     Results.push(Hexagon::V6_vrdelta, ResTy,
  2550:                  {OpRef::res(-1), OpRef(CtlR)});
  2551:     return OpRef::res(Results.top());
  2552:   }
  2553: 
  2554:   return OpRef::fail();
  2555: }
  2556: 
  2557: SDValue HvxSelector::getConst32(unsigned Val, const SDLoc &dl) {
  2558:   return DAG.getTargetConstant(Val, dl, MVT::i32);
  2559: }
  2560: 
  2561: SDValue HvxSelector::getSignedConst32(int Val, const SDLoc &dl) {
  2562:   return DAG.getSignedTargetConstant(Val, dl, MVT::i32);
  2563: }
  2564: 
  2565: SDValue HvxSelector::getVectorConstant(ArrayRef<uint8_t> Data,
  2566:                                        const SDLoc &dl) {
  2567:   SmallVector<SDValue, 128> Elems;
  2568:   for (uint8_t C : Data)
  2569:     Elems.push_back(DAG.getConstant(C, dl, MVT::i8));
  2570:   MVT VecTy = MVT::getVectorVT(MVT::i8, Data.size());
  2571:   SDValue BV = DAG.getBuildVector(VecTy, dl, Elems);
  2572:   SDValue LV = Lower.LowerOperation(BV, DAG);
  2573:   DAG.RemoveDeadNode(BV.getNode());
  2574:   return DAG.getNode(HexagonISD::ISEL, dl, VecTy, LV);
  2575: }
  2576: 
  2577: void HvxSelector::selectExtractSubvector(SDNode *N) {
  2578:   SDValue Inp = N->getOperand(0);
  2579:   MVT ResTy = N->getValueType(0).getSimpleVT();
  2580:   unsigned Idx = N->getConstantOperandVal(1);
  2581: 
  2582:   [[maybe_unused]] MVT InpTy = Inp.getValueType().getSimpleVT();
  2583:   [[maybe_unused]] unsigned ResLen = ResTy.getVectorNumElements();
  2584:   assert(InpTy.getVectorElementType() == ResTy.getVectorElementType());
  2585:   assert(2 * ResLen == InpTy.getVectorNumElements());
  2586:   assert(Idx == 0 || Idx == ResLen);
  2587: 
  2588:   unsigned SubReg = Idx == 0 ? Hexagon::vsub_lo : Hexagon::vsub_hi;
  2589:   SDValue Ext = DAG.getTargetExtractSubreg(SubReg, SDLoc(N), ResTy, Inp);
  2590: 
  2591:   ISel.ReplaceNode(N, Ext.getNode());
  2592: }
  2593: 
  2594: void HvxSelector::selectShuffle(SDNode *N) {
  2595:   DEBUG_WITH_TYPE("isel", {
  2596:     dbgs() << "Starting " << __func__ << " on node:\n";
  2597:     N->dump(&DAG);
  2598:   });
  2599:   MVT ResTy = N->getValueType(0).getSimpleVT();
  2600:   // Assume that vector shuffles operate on vectors of bytes.
```
- EN: It declares or implements routines such as assert, begin, erase, push_back, ... (41 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage. Notable Hexagon symbols referenced here include HexagonISD, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 assert, begin, erase, push_back, ... (41 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。 这里引用的重要 Hexagon 符号包括 HexagonISD，说明了它与同级后端组件的连接关系。

### Lines 2601-2800 / 第 2601-2800 行

```cpp
  2601:   assert(ResTy.isVector() && ResTy.getVectorElementType() == MVT::i8);
  2602: 
  2603:   auto *SN = cast<ShuffleVectorSDNode>(N);
  2604:   std::vector<int> Mask(SN->getMask().begin(), SN->getMask().end());
  2605:   // This shouldn't really be necessary. Is it?
  2606:   for (int &Idx : Mask)
  2607:     if (Idx != -1 && Idx < 0)
  2608:       Idx = -1;
  2609: 
  2610:   unsigned VecLen = Mask.size();
  2611:   bool HavePairs = (2*HwLen == VecLen);
  2612:   assert(ResTy.getSizeInBits() / 8 == VecLen);
  2613: 
  2614:   // Vd = vector_shuffle Va, Vb, Mask
  2615:   //
  2616: 
  2617:   bool UseLeft = false, UseRight = false;
  2618:   for (unsigned I = 0; I != VecLen; ++I) {
  2619:     if (Mask[I] == -1)
  2620:       continue;
  2621:     unsigned Idx = Mask[I];
  2622:     assert(Idx < 2*VecLen);
  2623:     if (Idx < VecLen)
  2624:       UseLeft = true;
  2625:     else
  2626:       UseRight = true;
  2627:   }
  2628: 
  2629:   DEBUG_WITH_TYPE("isel", {
  2630:     dbgs() << "VecLen=" << VecLen << " HwLen=" << HwLen << " UseLeft="
  2631:            << UseLeft << " UseRight=" << UseRight << " HavePairs="
  2632:            << HavePairs << '\n';
  2633:   });
  2634:   // If the mask is all -1's, generate "undef".
  2635:   if (!UseLeft && !UseRight) {
  2636:     ISel.ReplaceNode(N, ISel.selectUndef(SDLoc(SN), ResTy).getNode());
  2637:     return;
  2638:   }
  2639: 
  2640:   SDValue Vec0 = N->getOperand(0);
  2641:   SDValue Vec1 = N->getOperand(1);
  2642:   assert(Vec0.getValueType() == ResTy && Vec1.getValueType() == ResTy);
  2643: 
  2644:   ResultStack Results(SN);
  2645:   OpRef Va = OpRef::undef(ResTy);
  2646:   OpRef Vb = OpRef::undef(ResTy);
  2647: 
  2648:   if (!Vec0.isUndef()) {
  2649:     Results.push(TargetOpcode::COPY, ResTy, {Vec0});
  2650:     Va = OpRef::OpRef::res(Results.top());
  2651:   }
  2652:   if (!Vec1.isUndef()) {
  2653:     Results.push(TargetOpcode::COPY, ResTy, {Vec1});
  2654:     Vb = OpRef::res(Results.top());
  2655:   }
  2656: 
  2657:   OpRef Res = !HavePairs ? shuffs2(ShuffleMask(Mask), Va, Vb, Results)
  2658:                          : shuffp2(ShuffleMask(Mask), Va, Vb, Results);
  2659: 
  2660:   bool Done = Res.isValid();
  2661:   if (Done) {
  2662:     // Make sure that Res is on the stack before materializing.
  2663:     Results.push(TargetOpcode::COPY, ResTy, {Res});
  2664:     materialize(Results);
  2665:   } else {
  2666:     Done = scalarizeShuffle(Mask, SDLoc(N), ResTy, Vec0, Vec1, N);
  2667:   }
  2668: 
  2669:   if (!Done) {
  2670: #ifndef NDEBUG
  2671:     dbgs() << "Unhandled shuffle:\n";
  2672:     SN->dumpr(&DAG);
  2673: #endif
  2674:     llvm_unreachable("Failed to select vector shuffle");
  2675:   }
  2676: }
  2677: 
  2678: void HvxSelector::selectRor(SDNode *N) {
  2679:   // If this is a rotation by less than 8, use V6_valignbi.
  2680:   MVT Ty = N->getValueType(0).getSimpleVT();
  2681:   const SDLoc &dl(N);
  2682:   SDValue VecV = N->getOperand(0);
  2683:   SDValue RotV = N->getOperand(1);
  2684:   SDNode *NewN = nullptr;
  2685: 
  2686:   if (auto *CN = dyn_cast<ConstantSDNode>(RotV.getNode())) {
  2687:     unsigned S = CN->getZExtValue() % HST.getVectorLength();
  2688:     if (S == 0) {
  2689:       NewN = VecV.getNode();
  2690:     } else if (isUInt<3>(S)) {
  2691:       NewN = DAG.getMachineNode(Hexagon::V6_valignbi, dl, Ty,
  2692:                                 {VecV, VecV, getConst32(S, dl)});
  2693:     }
  2694:   }
  2695: 
  2696:   if (!NewN)
  2697:     NewN = DAG.getMachineNode(Hexagon::V6_vror, dl, Ty, {VecV, RotV});
  2698: 
  2699:   ISel.ReplaceNode(N, NewN);
  2700: }
  2701: 
  2702: void HvxSelector::selectVAlign(SDNode *N) {
  2703:   SDValue Vv = N->getOperand(0);
  2704:   SDValue Vu = N->getOperand(1);
  2705:   SDValue Rt = N->getOperand(2);
  2706:   SDNode *NewN = DAG.getMachineNode(Hexagon::V6_valignb, SDLoc(N),
  2707:                                     N->getValueType(0), {Vv, Vu, Rt});
  2708:   ISel.ReplaceNode(N, NewN);
  2709:   DAG.RemoveDeadNode(N);
  2710: }
  2711: 
  2712: void HexagonDAGToDAGISel::PreprocessHvxISelDAG() {
  2713:   auto getNodes = [this]() -> std::vector<SDNode *> {
  2714:     std::vector<SDNode *> T;
  2715:     T.reserve(CurDAG->allnodes_size());
  2716:     for (SDNode &N : CurDAG->allnodes())
  2717:       T.push_back(&N);
  2718:     return T;
  2719:   };
  2720: 
  2721:   ppHvxShuffleOfShuffle(getNodes());
  2722: }
  2723: 
  2724: template <> struct std::hash<SDValue> {
  2725:   std::size_t operator()(SDValue V) const {
  2726:     return std::hash<const void *>()(V.getNode()) +
  2727:            std::hash<unsigned>()(V.getResNo());
  2728:   };
  2729: };
  2730: 
  2731: void HexagonDAGToDAGISel::ppHvxShuffleOfShuffle(std::vector<SDNode *> &&Nodes) {
  2732:   // Motivating case:
  2733:   //   t10: v64i32 = ...
  2734:   //         t46: v128i8 = vector_shuffle<...> t44, t45
  2735:   //         t48: v128i8 = vector_shuffle<...> t44, t45
  2736:   //       t42: v128i8 = vector_shuffle<...> t46, t48
  2737:   //     t12: v32i32 = extract_subvector t10, Constant:i32<0>
  2738:   //   t44: v128i8 = bitcast t12
  2739:   //     t15: v32i32 = extract_subvector t10, Constant:i32<32>
  2740:   //   t45: v128i8 = bitcast t15
  2741:   SelectionDAG &DAG = *CurDAG;
  2742:   unsigned HwLen = HST->getVectorLength();
  2743: 
  2744:   struct SubVectorInfo {
  2745:     SubVectorInfo(SDValue S, unsigned H) : Src(S), HalfIdx(H) {}
  2746:     SDValue Src;
  2747:     unsigned HalfIdx;
  2748:   };
  2749: 
  2750:   using MapType = std::unordered_map<SDValue, unsigned>;
  2751: 
  2752:   auto getMaskElt = [&](unsigned Idx, ShuffleVectorSDNode *Shuff0,
  2753:                         ShuffleVectorSDNode *Shuff1,
  2754:                         const MapType &OpMap) -> int {
  2755:     // Treat Shuff0 and Shuff1 as operands to another vector shuffle, and
  2756:     // Idx as a (non-undef) element of the top level shuffle's mask, that
  2757:     // is, index into concat(Shuff0, Shuff1).
  2758:     // Assuming that Shuff0 and Shuff1 both operate on subvectors of the
  2759:     // same source vector (as described by OpMap), return the index of
  2760:     // that source vector corresponding to Idx.
  2761:     ShuffleVectorSDNode *OpShuff = Idx < HwLen ? Shuff0 : Shuff1;
  2762:     if (Idx >= HwLen)
  2763:       Idx -= HwLen;
  2764: 
  2765:     // Get the mask index that M points at in the corresponding operand.
  2766:     int MaybeN = OpShuff->getMaskElt(Idx);
  2767:     if (MaybeN < 0)
  2768:       return -1;
  2769: 
  2770:     auto N = static_cast<unsigned>(MaybeN);
  2771:     unsigned SrcBase = N < HwLen ? OpMap.at(OpShuff->getOperand(0))
  2772:                                  : OpMap.at(OpShuff->getOperand(1));
  2773:     if (N >= HwLen)
  2774:       N -= HwLen;
  2775: 
  2776:     return N + SrcBase;
  2777:   };
  2778: 
  2779:   auto fold3 = [&](SDValue TopShuff, SDValue Inp, MapType &&OpMap) -> SDValue {
  2780:     // Fold all 3 shuffles into a single one.
  2781:     auto *This = cast<ShuffleVectorSDNode>(TopShuff);
  2782:     auto *S0 = cast<ShuffleVectorSDNode>(TopShuff.getOperand(0));
  2783:     auto *S1 = cast<ShuffleVectorSDNode>(TopShuff.getOperand(1));
  2784:     ArrayRef<int> TopMask = This->getMask();
  2785:     // This should be guaranteed by type checks in the caller, and the fact
  2786:     // that all shuffles should have been promoted to operate on MVT::i8.
  2787:     assert(TopMask.size() == S0->getMask().size() &&
  2788:            TopMask.size() == S1->getMask().size());
  2789:     assert(TopMask.size() == HwLen);
  2790: 
  2791:     SmallVector<int, 256> FoldedMask(2 * HwLen);
  2792:     for (unsigned I = 0; I != HwLen; ++I) {
  2793:       int MaybeM = TopMask[I];
  2794:       if (MaybeM >= 0) {
  2795:         FoldedMask[I] =
  2796:             getMaskElt(static_cast<unsigned>(MaybeM), S0, S1, OpMap);
  2797:       } else {
  2798:         FoldedMask[I] = -1;
  2799:       }
  2800:     }
```
- EN: Header guards in this range prevent duplicate inclusion and define the interface boundary. It declares types such as SubVectorInfo, which carry the state or API of this component. It declares or implements routines such as assert, cast<ShuffleVectorSDNode>, Mask, size, ... (34 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal.
- CN: 这一段中的头文件保护宏用于防止重复包含，并明确接口边界。 这里声明了 SubVectorInfo 等类型，用来承载该组件的状态或接口。 这里声明或实现了 assert, cast<ShuffleVectorSDNode>, Mask, size, ... (34 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。

### Lines 2801-3000 / 第 2801-3000 行

```cpp
  2801:     // The second half of the result will be all-undef.
  2802:     std::fill(FoldedMask.begin() + HwLen, FoldedMask.end(), -1);
  2803: 
  2804:     // Return
  2805:     //   FoldedShuffle = (Shuffle Inp, undef, FoldedMask)
  2806:     //   (LoHalf FoldedShuffle)
  2807:     const SDLoc &dl(TopShuff);
  2808:     MVT SingleTy = MVT::getVectorVT(MVT::i8, HwLen);
  2809:     MVT PairTy = MVT::getVectorVT(MVT::i8, 2 * HwLen);
  2810:     SDValue FoldedShuff =
  2811:         DAG.getVectorShuffle(PairTy, dl, DAG.getBitcast(PairTy, Inp),
  2812:                              DAG.getUNDEF(PairTy), FoldedMask);
  2813:     return DAG.getNode(ISD::EXTRACT_SUBVECTOR, dl, SingleTy, FoldedShuff,
  2814:                        DAG.getConstant(0, dl, MVT::i32));
  2815:   };
  2816: 
  2817:   auto getSourceInfo = [](SDValue V) -> std::optional<SubVectorInfo> {
  2818:     while (V.getOpcode() == ISD::BITCAST)
  2819:       V = V.getOperand(0);
  2820:     if (V.getOpcode() != ISD::EXTRACT_SUBVECTOR)
  2821:       return std::nullopt;
  2822:     return SubVectorInfo(V.getOperand(0),
  2823:                          !cast<ConstantSDNode>(V.getOperand(1))->isZero());
  2824:   };
  2825: 
  2826:   for (SDNode *N : Nodes) {
  2827:     if (N->getOpcode() != ISD::VECTOR_SHUFFLE)
  2828:       continue;
  2829:     EVT ResTy = N->getValueType(0);
  2830:     if (ResTy.getVectorElementType() != MVT::i8)
  2831:       continue;
  2832:     if (ResTy.getVectorNumElements() != HwLen)
  2833:       continue;
  2834: 
  2835:     SDValue V0 = N->getOperand(0);
  2836:     SDValue V1 = N->getOperand(1);
  2837:     if (V0.getOpcode() != ISD::VECTOR_SHUFFLE)
  2838:       continue;
  2839:     if (V1.getOpcode() != ISD::VECTOR_SHUFFLE)
  2840:       continue;
  2841:     if (V0.getValueType() != ResTy || V1.getValueType() != ResTy)
  2842:       continue;
  2843: 
  2844:     // Check if all operands of the two operand shuffles are extract_subvectors
  2845:     // from the same vector pair.
  2846:     auto V0A = getSourceInfo(V0.getOperand(0));
  2847:     if (!V0A.has_value())
  2848:       continue;
  2849:     auto V0B = getSourceInfo(V0.getOperand(1));
  2850:     if (!V0B.has_value() || V0B->Src != V0A->Src)
  2851:       continue;
  2852:     auto V1A = getSourceInfo(V1.getOperand(0));
  2853:     if (!V1A.has_value() || V1A->Src != V0A->Src)
  2854:       continue;
  2855:     auto V1B = getSourceInfo(V1.getOperand(1));
  2856:     if (!V1B.has_value() || V1B->Src != V0A->Src)
  2857:       continue;
  2858: 
  2859:     // The source must be a pair. This should be guaranteed here,
  2860:     // but check just in case.
  2861:     assert(V0A->Src.getValueType().getSizeInBits() == 16 * HwLen);
  2862: 
  2863:     MapType OpMap = {
  2864:         {V0.getOperand(0), V0A->HalfIdx * HwLen},
  2865:         {V0.getOperand(1), V0B->HalfIdx * HwLen},
  2866:         {V1.getOperand(0), V1A->HalfIdx * HwLen},
  2867:         {V1.getOperand(1), V1B->HalfIdx * HwLen},
  2868:     };
  2869:     SDValue NewS = fold3(SDValue(N, 0), V0A->Src, std::move(OpMap));
  2870:     ReplaceNode(N, NewS.getNode());
  2871:   }
  2872: }
  2873: 
  2874: void HexagonDAGToDAGISel::SelectHvxExtractSubvector(SDNode *N) {
  2875:   HvxSelector(*this, *CurDAG).selectExtractSubvector(N);
  2876: }
  2877: 
  2878: void HexagonDAGToDAGISel::SelectHvxShuffle(SDNode *N) {
  2879:   HvxSelector(*this, *CurDAG).selectShuffle(N);
  2880: }
  2881: 
  2882: void HexagonDAGToDAGISel::SelectHvxRor(SDNode *N) {
  2883:   HvxSelector(*this, *CurDAG).selectRor(N);
  2884: }
  2885: 
  2886: void HexagonDAGToDAGISel::SelectHvxVAlign(SDNode *N) {
  2887:   HvxSelector(*this, *CurDAG).selectVAlign(N);
  2888: }
  2889: 
  2890: void HexagonDAGToDAGISel::SelectV65GatherPred(SDNode *N) {
  2891:   const SDLoc &dl(N);
  2892:   SDValue Chain = N->getOperand(0);
  2893:   SDValue Address = N->getOperand(2);
  2894:   SDValue Predicate = N->getOperand(3);
  2895:   SDValue Base = N->getOperand(4);
  2896:   SDValue Modifier = N->getOperand(5);
  2897:   SDValue Offset = N->getOperand(6);
  2898:   SDValue ImmOperand = CurDAG->getTargetConstant(0, dl, MVT::i32);
  2899: 
  2900:   unsigned Opcode;
  2901:   unsigned IntNo = N->getConstantOperandVal(1);
  2902:   switch (IntNo) {
  2903:   default:
  2904:     llvm_unreachable("Unexpected HVX gather intrinsic.");
  2905:   case Intrinsic::hexagon_V6_vgathermhq:
  2906:   case Intrinsic::hexagon_V6_vgathermhq_128B:
  2907:     Opcode = Hexagon::V6_vgathermhq_pseudo;
  2908:     break;
  2909:   case Intrinsic::hexagon_V6_vgathermwq:
  2910:   case Intrinsic::hexagon_V6_vgathermwq_128B:
  2911:     Opcode = Hexagon::V6_vgathermwq_pseudo;
  2912:     break;
  2913:   case Intrinsic::hexagon_V6_vgathermhwq:
  2914:   case Intrinsic::hexagon_V6_vgathermhwq_128B:
  2915:     Opcode = Hexagon::V6_vgathermhwq_pseudo;
  2916:     break;
  2917:   }
  2918: 
  2919:   SDVTList VTs = CurDAG->getVTList(MVT::Other);
  2920:   SDValue Ops[] = { Address, ImmOperand,
  2921:                     Predicate, Base, Modifier, Offset, Chain };
  2922:   SDNode *Result = CurDAG->getMachineNode(Opcode, dl, VTs, Ops);
  2923: 
  2924:   MachineMemOperand *MemOp = cast<MemIntrinsicSDNode>(N)->getMemOperand();
  2925:   CurDAG->setNodeMemRefs(cast<MachineSDNode>(Result), {MemOp});
  2926: 
  2927:   ReplaceNode(N, Result);
  2928: }
  2929: 
  2930: void HexagonDAGToDAGISel::SelectV65Gather(SDNode *N) {
  2931:   const SDLoc &dl(N);
  2932:   SDValue Chain = N->getOperand(0);
  2933:   SDValue Address = N->getOperand(2);
  2934:   SDValue Base = N->getOperand(3);
  2935:   SDValue Modifier = N->getOperand(4);
  2936:   SDValue Offset = N->getOperand(5);
  2937:   SDValue ImmOperand = CurDAG->getTargetConstant(0, dl, MVT::i32);
  2938: 
  2939:   unsigned Opcode;
  2940:   unsigned IntNo = N->getConstantOperandVal(1);
  2941:   switch (IntNo) {
  2942:   default:
  2943:     llvm_unreachable("Unexpected HVX gather intrinsic.");
  2944:   case Intrinsic::hexagon_V6_vgathermh:
  2945:   case Intrinsic::hexagon_V6_vgathermh_128B:
  2946:     Opcode = Hexagon::V6_vgathermh_pseudo;
  2947:     break;
  2948:   case Intrinsic::hexagon_V6_vgathermw:
  2949:   case Intrinsic::hexagon_V6_vgathermw_128B:
  2950:     Opcode = Hexagon::V6_vgathermw_pseudo;
  2951:     break;
  2952:   case Intrinsic::hexagon_V6_vgathermhw:
  2953:   case Intrinsic::hexagon_V6_vgathermhw_128B:
  2954:     Opcode = Hexagon::V6_vgathermhw_pseudo;
  2955:     break;
  2956:   case Intrinsic::hexagon_V6_vgather_vscattermh:
  2957:   case Intrinsic::hexagon_V6_vgather_vscattermh_128B:
  2958:     Opcode = Hexagon::V6_vgather_vscatter_mh_pseudo;
  2959:     break;
  2960:   }
  2961: 
  2962:   SDVTList VTs = CurDAG->getVTList(MVT::Other);
  2963:   SDValue Ops[] = { Address, ImmOperand, Base, Modifier, Offset, Chain };
  2964:   SDNode *Result = CurDAG->getMachineNode(Opcode, dl, VTs, Ops);
  2965: 
  2966:   MachineMemOperand *MemOp = cast<MemIntrinsicSDNode>(N)->getMemOperand();
  2967:   CurDAG->setNodeMemRefs(cast<MachineSDNode>(Result), {MemOp});
  2968: 
  2969:   ReplaceNode(N, Result);
  2970: }
  2971: 
  2972: void HexagonDAGToDAGISel::SelectHVXDualOutput(SDNode *N) {
  2973:   unsigned IID = N->getConstantOperandVal(0);
  2974:   SDNode *Result;
  2975:   switch (IID) {
  2976:   case Intrinsic::hexagon_V6_vaddcarry: {
  2977:     std::array<SDValue, 3> Ops = {
  2978:         {N->getOperand(1), N->getOperand(2), N->getOperand(3)}};
  2979:     SDVTList VTs = CurDAG->getVTList(MVT::v16i32, MVT::v64i1);
  2980:     Result = CurDAG->getMachineNode(Hexagon::V6_vaddcarry, SDLoc(N), VTs, Ops);
  2981:     break;
  2982:   }
  2983:   case Intrinsic::hexagon_V6_vaddcarry_128B: {
  2984:     std::array<SDValue, 3> Ops = {
  2985:         {N->getOperand(1), N->getOperand(2), N->getOperand(3)}};
  2986:     SDVTList VTs = CurDAG->getVTList(MVT::v32i32, MVT::v128i1);
  2987:     Result = CurDAG->getMachineNode(Hexagon::V6_vaddcarry, SDLoc(N), VTs, Ops);
  2988:     break;
  2989:   }
  2990:   case Intrinsic::hexagon_V6_vsubcarry: {
  2991:     std::array<SDValue, 3> Ops = {
  2992:         {N->getOperand(1), N->getOperand(2), N->getOperand(3)}};
  2993:     SDVTList VTs = CurDAG->getVTList(MVT::v16i32, MVT::v64i1);
  2994:     Result = CurDAG->getMachineNode(Hexagon::V6_vsubcarry, SDLoc(N), VTs, Ops);
  2995:     break;
  2996:   }
  2997:   case Intrinsic::hexagon_V6_vsubcarry_128B: {
  2998:     std::array<SDValue, 3> Ops = {
  2999:         {N->getOperand(1), N->getOperand(2), N->getOperand(3)}};
  3000:     SDVTList VTs = CurDAG->getVTList(MVT::v32i32, MVT::v128i1);
```
- EN: It declares or implements routines such as std::fill, dl, MVT::getVectorVT, getVectorShuffle, ... (26 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage. Notable Hexagon symbols referenced here include HexagonDAGToDAGISel, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 std::fill, dl, MVT::getVectorVT, getVectorShuffle, ... (26 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。 这里引用的重要 Hexagon 符号包括 HexagonDAGToDAGISel，说明了它与同级后端组件的连接关系。

### Lines 3001-3011 / 第 3001-3011 行

```cpp
  3001:     Result = CurDAG->getMachineNode(Hexagon::V6_vsubcarry, SDLoc(N), VTs, Ops);
  3002:     break;
  3003:   }
  3004:   default:
  3005:     llvm_unreachable("Unexpected HVX dual output intrinsic.");
  3006:   }
  3007:   ReplaceUses(N, Result);
  3008:   ReplaceUses(SDValue(N, 0), SDValue(Result, 0));
  3009:   ReplaceUses(SDValue(N, 1), SDValue(Result, 1));
  3010:   CurDAG->RemoveDeadNode(N);
  3011: }
```
- EN: It declares or implements routines such as getMachineNode, llvm_unreachable, ReplaceUses, RemoveDeadNode, translating Hexagon-specific policy into reusable code paths.
- CN: 这里声明或实现了 getMachineNode, llvm_unreachable, ReplaceUses, RemoveDeadNode 等例程，把 Hexagon 特定策略落实为可复用的代码路径。

## Key Concepts / 关键概念

- HVX vector ISA modeling / HVX 向量 ISA 建模
- intrinsic mapping / Intrinsic 映射

## Dependencies / 依赖关系

- Direct includes / 直接包含: `HexagonISelDAGToDAG.h, HexagonISelLowering.h, llvm/ADT/BitVector.h, llvm/ADT/SetVector.h, llvm/CodeGen/SelectionDAGISel.h, llvm/IR/Intrinsics.h, llvm/IR/IntrinsicsHexagon.h, llvm/Support/Debug.h, llvm/Support/MathExtras.h, algorithm, ... (19 total)`
- Hexagon symbols / Hexagon 符号: `HexagonISelDAGToDAGHVX, HexagonISelDAGToDAG, HexagonISelLowering, HexagonTargetLowering, HexagonSubtarget, HexagonDAGToDAGISel, HexagonISD`
- Build role / 构建角色: compiled into LLVM's Hexagon backend library and linked with CodeGen/MC infrastructure. / 被编译进 LLVM 的 Hexagon 后端库，并与 CodeGen/MC 基础设施联动。
