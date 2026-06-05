# HexagonShuffler.h — Code Analysis / 代码分析

## Source / 来源

- File / 文件: `llvm/lib/Target/Hexagon/MCTargetDesc/HexagonShuffler.h`
- Repository / 仓库: `/root/xw/llvm-project/`
- Purpose / 作用 (EN): Declares Hexagon MC-layer target description glue.
- Purpose / 作用 (CN): 该文件声明 Hexagon 后端组件的接口、类型或辅助入口。

## Line-by-Line Analysis / 逐行分析

### Lines 1-50 / 第 1-50 行

```cpp
     1: //===- HexagonShuffler.h - Instruction bundle shuffling ---------*- C++ -*-===//
     2: //
     3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
     4: // See https://llvm.org/LICENSE.txt for license information.
     5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
     6: //
     7: //===----------------------------------------------------------------------===//
     8: //
     9: // This implements the shuffling of insns inside a bundle according to the
    10: // packet formation rules of the Hexagon ISA.
    11: //
    12: //===----------------------------------------------------------------------===//
    13: 
    14: #ifndef LLVM_LIB_TARGET_HEXAGON_MCTARGETDESC_HEXAGONSHUFFLER_H
    15: #define LLVM_LIB_TARGET_HEXAGON_MCTARGETDESC_HEXAGONSHUFFLER_H
    16: 
    17: #include "MCTargetDesc/HexagonMCInstrInfo.h"
    18: #include "MCTargetDesc/HexagonMCTargetDesc.h"
    19: #include "llvm/ADT/STLExtras.h"
    20: #include "llvm/ADT/SmallVector.h"
    21: #include "llvm/ADT/StringRef.h"
    22: #include "llvm/Support/MathExtras.h"
    23: #include "llvm/Support/SMLoc.h"
    24: #include <cstdint>
    25: #include <functional>
    26: #include <optional>
    27: #include <utility>
    28: 
    29: namespace llvm {
    30: 
    31: class MCContext;
    32: class MCInst;
    33: class MCInstrInfo;
    34: class MCSubtargetInfo;
    35: 
    36: // Insn resources.
    37: class HexagonResource {
    38:   // Mask of the slots or units that may execute the insn and
    39:   // the weight or priority that the insn requires to be assigned a slot.
    40:   unsigned Slots, Weight;
    41: 
    42: public:
    43:   HexagonResource(unsigned s) { setUnits(s); }
    44: 
    45:   void setUnits(unsigned s) {
    46:     Slots = s & ((1u << HEXAGON_PACKET_SIZE) - 1);
    47:     setWeight(s);
    48:   }
    49: 
    50:   void setAllUnits() {
```
- EN: It imports headers such as MCTargetDesc/HexagonMCInstrInfo.h, MCTargetDesc/HexagonMCTargetDesc.h, llvm/ADT/STLExtras.h, llvm/ADT/SmallVector.h, ... (11 total), establishing the LLVM/Hexagon APIs used below. Header guards in this range prevent duplicate inclusion and define the interface boundary. It opens namespaces (llvm) to keep the implementation scoped to LLVM/Hexagon components. It declares types such as MCContext, MCInst, MCInstrInfo, MCSubtargetInfo, ... (5 total), which carry the state or API of this component.
- CN: 这里引入了 MCTargetDesc/HexagonMCInstrInfo.h, MCTargetDesc/HexagonMCTargetDesc.h, llvm/ADT/STLExtras.h, llvm/ADT/SmallVector.h, ... (11 total) 等头文件，确定了后续代码依赖的 LLVM/Hexagon API。 这一段中的头文件保护宏用于防止重复包含，并明确接口边界。 这里打开了命名空间（llvm），把实现限定在 LLVM/Hexagon 组件作用域中。 这里声明了 MCContext, MCInst, MCInstrInfo, MCSubtargetInfo, ... (5 total) 等类型，用来承载该组件的状态或接口。

### Lines 51-100 / 第 51-100 行

```cpp
    51:     setUnits(((1u << HEXAGON_PACKET_SIZE) - 1));
    52:   }
    53:   unsigned setWeight(unsigned s);
    54: 
    55:   unsigned getUnits() const { return (Slots); }
    56:   unsigned getWeight() const { return (Weight); }
    57: 
    58:   // Check if the resources are in ascending slot order.
    59:   static bool lessUnits(const HexagonResource &A, const HexagonResource &B) {
    60:     return (llvm::popcount(A.getUnits()) < llvm::popcount(B.getUnits()));
    61:   }
    62: 
    63:   // Check if the resources are in ascending weight order.
    64:   static bool lessWeight(const HexagonResource &A, const HexagonResource &B) {
    65:     return (A.getWeight() < B.getWeight());
    66:   }
    67: };
    68: 
    69: // HVX insn resources.
    70: class HexagonCVIResource : public HexagonResource {
    71: public:
    72:   using UnitsAndLanes = std::pair<unsigned, unsigned>;
    73: 
    74: private:
    75:   // Count of adjacent slots that the insn requires to be executed.
    76:   unsigned Lanes;
    77:   // Flag whether the insn is a load or a store.
    78:   bool Load, Store;
    79:   // Flag whether the HVX resources are valid.
    80:   bool Valid;
    81: 
    82:   void setLanes(unsigned l) { Lanes = l; }
    83:   void setLoad(bool f = true) { Load = f; }
    84:   void setStore(bool f = true) { Store = f; }
    85: 
    86: public:
    87:   HexagonCVIResource(MCInstrInfo const &MCII,
    88:                      MCSubtargetInfo const &STI,
    89:                      unsigned s, MCInst const *id);
    90: 
    91:   bool isValid() const { return Valid; }
    92:   unsigned getLanes() const { return Lanes; }
    93:   bool mayLoad() const { return Load; }
    94:   bool mayStore() const { return Store; }
    95: };
    96: 
    97: // Handle to an insn used by the shuffling algorithm.
    98: class HexagonInstr {
    99:   friend class HexagonShuffler;
   100: 
```
- EN: It declares types such as HexagonCVIResource, HexagonInstr, which carry the state or API of this component. It defines declarative TableGen records like HexagonCVIResource, HexagonInstr; these records are consumed by TableGen instead of executed directly. It declares or implements routines such as setUnits, setWeight, getUnits, getWeight, ... (14 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal.
- CN: 这里声明了 HexagonCVIResource, HexagonInstr 等类型，用来承载该组件的状态或接口。 这里定义了 HexagonCVIResource, HexagonInstr 等声明式 TableGen 记录；这些记录由 TableGen 消费，而不是直接执行。 这里声明或实现了 setUnits, setWeight, getUnits, getWeight, ... (14 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。

### Lines 101-150 / 第 101-150 行

```cpp
   101:   MCInst const *ID;
   102:   MCInst const *Extender;
   103:   HexagonResource Core;
   104:   HexagonCVIResource CVI;
   105: 
   106: public:
   107:   HexagonInstr(MCInstrInfo const &MCII,
   108:                MCSubtargetInfo const &STI, MCInst const *id,
   109:                MCInst const *Extender, unsigned s)
   110:       : ID(id), Extender(Extender), Core(s), CVI(MCII, STI, s, id){};
   111: 
   112:   MCInst const &getDesc() const { return *ID; }
   113:   MCInst const *getExtender() const { return Extender; }
   114: 
   115:   // Check if the handles are in ascending order for shuffling purposes.
   116:   bool operator<(const HexagonInstr &B) const {
   117:     return (HexagonResource::lessWeight(B.Core, Core));
   118:   }
   119: 
   120:   // Check if the handles are in ascending order by core slots.
   121:   static bool lessCore(const HexagonInstr &A, const HexagonInstr &B) {
   122:     return (HexagonResource::lessUnits(A.Core, B.Core));
   123:   }
   124: 
   125:   // Check if the handles are in ascending order by HVX slots.
   126:   static bool lessCVI(const HexagonInstr &A, const HexagonInstr &B) {
   127:     return (HexagonResource::lessUnits(A.CVI, B.CVI));
   128:   }
   129: };
   130: 
   131: // Bundle shuffler.
   132: class HexagonShuffler {
   133:   using HexagonPacket =
   134:       SmallVector<HexagonInstr, HEXAGON_PRESHUFFLE_PACKET_SIZE>;
   135: 
   136:   struct HexagonPacketSummary {
   137:     // Number of memory operations, loads, solo loads, stores, solo stores,
   138:     // single stores.
   139:     unsigned memory;
   140:     unsigned loads;
   141:     unsigned load0;
   142:     unsigned stores;
   143:     unsigned store0;
   144:     unsigned store1;
   145:     unsigned NonZCVIloads;
   146:     unsigned AllCVIloads;
   147:     unsigned CVIstores;
   148:     // Number of duplex insns
   149:     unsigned duplex;
   150:     unsigned pSlot3Cnt;
```
- EN: It declares types such as HexagonShuffler, HexagonPacketSummary, which carry the state or API of this component. It defines declarative TableGen records like HexagonShuffler; these records are consumed by TableGen instead of executed directly. It declares or implements routines such as HexagonInstr, getDesc, getExtender, lessCore, ... (5 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal.
- CN: 这里声明了 HexagonShuffler, HexagonPacketSummary 等类型，用来承载该组件的状态或接口。 这里定义了 HexagonShuffler 等声明式 TableGen 记录；这些记录由 TableGen 消费，而不是直接执行。 这里声明或实现了 HexagonInstr, getDesc, getExtender, lessCore, ... (5 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。

### Lines 151-200 / 第 151-200 行

```cpp
   151:     std::optional<HexagonInstr *> PrefSlot3Inst;
   152:     unsigned memops;
   153:     unsigned ReservedSlotMask;
   154:     SmallVector<HexagonInstr *, HEXAGON_PRESHUFFLE_PACKET_SIZE> branchInsts;
   155:     std::optional<SMLoc> Slot1AOKLoc;
   156:     std::optional<SMLoc> NoSlot1StoreLoc;
   157:   };
   158:   // Insn handles in a bundle.
   159:   HexagonPacket Packet;
   160: 
   161: protected:
   162:   MCContext &Context;
   163:   int64_t BundleFlags;
   164:   MCInstrInfo const &MCII;
   165:   MCSubtargetInfo const &STI;
   166:   SMLoc Loc;
   167:   bool ReportErrors;
   168:   bool CheckFailure;
   169:   std::vector<std::pair<SMLoc, std::string>> AppliedRestrictions;
   170: 
   171:   bool applySlotRestrictions(HexagonPacketSummary const &Summary,
   172:                              const bool DoShuffle);
   173:   void restrictSlot1AOK(HexagonPacketSummary const &Summary);
   174:   void restrictNoSlot1Store(HexagonPacketSummary const &Summary);
   175:   void restrictNoSlot1();
   176:   bool restrictStoreLoadOrder(HexagonPacketSummary const &Summary);
   177:   void restrictBranchOrder(HexagonPacketSummary const &Summary);
   178:   void restrictPreferSlot3(HexagonPacketSummary const &Summary,
   179:                            const bool DoShuffle);
   180:   void permitNonSlot();
   181: 
   182:   std::optional<HexagonPacket> tryAuction(HexagonPacketSummary const &Summary);
   183: 
   184:   HexagonPacketSummary GetPacketSummary();
   185:   bool ValidPacketMemoryOps(HexagonPacketSummary const &Summary) const;
   186:   bool ValidResourceUsage(HexagonPacketSummary const &Summary);
   187: 
   188: public:
   189:   using iterator = HexagonPacket::iterator;
   190:   using const_iterator = HexagonPacket::const_iterator;
   191:   using packet_range = iterator_range<HexagonPacket::iterator>;
   192:   using const_packet_range = iterator_range<HexagonPacket::const_iterator>;
   193: 
   194:   HexagonShuffler(MCContext &Context, bool ReportErrors,
   195:                   MCInstrInfo const &MCII, MCSubtargetInfo const &STI);
   196: 
   197:   // Reset to initial state.
   198:   void reset();
   199:   // Check if the bundle may be validly shuffled.
   200:   bool check(const bool RequireShuffle = true);
```
- EN: It declares or implements routines such as applySlotRestrictions, restrictSlot1AOK, restrictNoSlot1Store, restrictNoSlot1, ... (15 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonInstr, HexagonPacket, HexagonPacketSummary, HexagonShuffler, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 applySlotRestrictions, restrictSlot1AOK, restrictNoSlot1Store, restrictNoSlot1, ... (15 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonInstr, HexagonPacket, HexagonPacketSummary, HexagonShuffler，说明了它与同级后端组件的连接关系。

### Lines 201-243 / 第 201-243 行

```cpp
   201:   // Reorder the insn handles in the bundle.
   202:   bool shuffle();
   203: 
   204:   unsigned size() const { return (Packet.size()); }
   205: 
   206:   bool isMemReorderDisabled() const {
   207:     return (BundleFlags & HexagonMCInstrInfo::memReorderDisabledMask) != 0;
   208:   }
   209: 
   210:   iterator begin() { return (Packet.begin()); }
   211:   iterator end() { return (Packet.end()); }
   212:   const_iterator cbegin() const { return (Packet.begin()); }
   213:   const_iterator cend() const { return (Packet.end()); }
   214:   packet_range insts(HexagonPacket &P) {
   215:     return make_range(P.begin(), P.end());
   216:   }
   217:   const_packet_range insts(HexagonPacket const &P) const {
   218:     return make_range(P.begin(), P.end());
   219:   }
   220:   packet_range insts() { return make_range(begin(), end()); }
   221:   const_packet_range insts() const { return make_range(cbegin(), cend()); }
   222: 
   223:   using InstPredicate = bool (*)(MCInstrInfo const &, MCInst const &);
   224: 
   225:   bool HasInstWith(InstPredicate Pred) const {
   226:     return llvm::any_of(insts(), [&](HexagonInstr const &I) {
   227:       MCInst const &Inst = I.getDesc();
   228:       return (*Pred)(MCII, Inst);
   229:     });
   230:   }
   231: 
   232:   // Add insn handle to the bundle .
   233:   void append(MCInst const &ID, MCInst const *Extender, unsigned S);
   234: 
   235:   // Return the error code for the last check or shuffling of the bundle.
   236:   void reportError(Twine const &Msg);
   237:   void reportResourceError(HexagonPacketSummary const &Summary, StringRef Err);
   238:   void reportResourceUsage(HexagonPacketSummary const &Summary);
   239: };
   240: 
   241: } // end namespace llvm
   242: 
   243: #endif //  LLVM_LIB_TARGET_HEXAGON_MCTARGETDESC_HEXAGONSHUFFLER_H
```
- EN: It opens namespaces (llvm) to keep the implementation scoped to LLVM/Hexagon components. It declares or implements routines such as shuffle, size, isMemReorderDisabled, begin, ... (17 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonMCInstrInfo, HexagonPacket, HexagonInstr, HexagonPacketSummary, showing how the code connects to sibling backend components.
- CN: 这里打开了命名空间（llvm），把实现限定在 LLVM/Hexagon 组件作用域中。 这里声明或实现了 shuffle, size, isMemReorderDisabled, begin, ... (17 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonMCInstrInfo, HexagonPacket, HexagonInstr, HexagonPacketSummary，说明了它与同级后端组件的连接关系。

## Key Concepts / 关键概念

- HVX vector ISA modeling / HVX 向量 ISA 建模
- instruction semantics / 指令语义
- MC-layer target description / MC 层目标描述
- subtarget feature gating / 子目标特性控制

## Dependencies / 依赖关系

- Direct includes / 直接包含: `MCTargetDesc/HexagonMCInstrInfo.h, MCTargetDesc/HexagonMCTargetDesc.h, llvm/ADT/STLExtras.h, llvm/ADT/SmallVector.h, llvm/ADT/StringRef.h, llvm/Support/MathExtras.h, llvm/Support/SMLoc.h, cstdint, functional, optional, ... (11 total)`
- Hexagon symbols / Hexagon 符号: `HexagonShuffler, HexagonMCInstrInfo, HexagonMCTargetDesc, HexagonResource, HexagonCVIResource, HexagonInstr, HexagonPacket, HexagonPacketSummary`
- Interface role / 接口角色: included by sibling Hexagon implementation files to share declarations safely. / 由同级 Hexagon 实现文件包含，以共享声明。
