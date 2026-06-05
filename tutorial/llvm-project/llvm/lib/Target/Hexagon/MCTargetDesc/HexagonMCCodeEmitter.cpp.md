# HexagonMCCodeEmitter.cpp — Code Analysis / 代码分析

## Source / 来源

- File / 文件: `llvm/lib/Target/Hexagon/MCTargetDesc/HexagonMCCodeEmitter.cpp`
- Repository / 仓库: `/root/xw/llvm-project/`
- Purpose / 作用 (EN): Implements Hexagon binary encoding of MC instructions.
- Purpose / 作用 (CN): 该文件实现 Hexagon 后端组件的核心逻辑。 重点涉及指令语义与选择。

## Line-by-Line Analysis / 逐行分析

### Lines 1-100 / 第 1-100 行

```cpp
     1: //===- HexagonMCCodeEmitter.cpp - Hexagon Target Descriptions -------------===//
     2: //
     3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
     4: // See https://llvm.org/LICENSE.txt for license information.
     5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
     6: //
     7: //===----------------------------------------------------------------------===//
     8: 
     9: #include "MCTargetDesc/HexagonMCCodeEmitter.h"
    10: #include "MCTargetDesc/HexagonBaseInfo.h"
    11: #include "MCTargetDesc/HexagonFixupKinds.h"
    12: #include "MCTargetDesc/HexagonMCExpr.h"
    13: #include "MCTargetDesc/HexagonMCInstrInfo.h"
    14: #include "MCTargetDesc/HexagonMCTargetDesc.h"
    15: #include "llvm/ADT/Statistic.h"
    16: #include "llvm/MC/MCContext.h"
    17: #include "llvm/MC/MCExpr.h"
    18: #include "llvm/MC/MCFixup.h"
    19: #include "llvm/MC/MCInst.h"
    20: #include "llvm/MC/MCInstrDesc.h"
    21: #include "llvm/MC/MCInstrInfo.h"
    22: #include "llvm/MC/MCRegisterInfo.h"
    23: #include "llvm/MC/MCSubtargetInfo.h"
    24: #include "llvm/Support/Casting.h"
    25: #include "llvm/Support/Compiler.h"
    26: #include "llvm/Support/Debug.h"
    27: #include "llvm/Support/EndianStream.h"
    28: #include "llvm/Support/raw_ostream.h"
    29: #include <cassert>
    30: #include <cstddef>
    31: #include <cstdint>
    32: #include <map>
    33: #include <string>
    34: #include <vector>
    35: 
    36: #define DEBUG_TYPE "mccodeemitter"
    37: 
    38: using namespace llvm;
    39: using namespace Hexagon;
    40: 
    41: STATISTIC(MCNumEmitted, "Number of MC instructions emitted");
    42: 
    43: static const unsigned fixup_Invalid = ~0u;
    44: 
    45: // clang-format off
    46: #define _ fixup_Invalid
    47: #define P(x) Hexagon::fixup_Hexagon##x
    48: static const std::map<unsigned, std::vector<unsigned>> ExtFixups = {
    49:   { HexagonMCExpr::VK_DTPREL,
    50:     { _,                _,              _,                      _,
    51:       _,                _,              P(_DTPREL_16_X),        P(_DTPREL_11_X),
    52:       P(_DTPREL_11_X),  P(_9_X),        _,                      P(_DTPREL_11_X),
    53:       P(_DTPREL_16_X),  _,              _,                      _,
    54:       P(_DTPREL_16_X),  _,              _,                      _,
    55:       _,                _,              _,                      _,
    56:       _,                _,              _,                      _,
    57:       _,                _,              _,                      _,
    58:       P(_DTPREL_32_6_X) }},
    59:   { HexagonMCExpr::VK_GOT,
    60:     { _,                _,              _,                      _,
    61:       _,                _,              P(_GOT_11_X),           _ /* [1] */,
    62:       _ /* [1] */,      P(_9_X),        _,                      P(_GOT_11_X),
    63:       P(_GOT_16_X),     _,              _,                      _,
    64:       P(_GOT_16_X),     _,              _,                      _,
    65:       _,                _,              _,                      _,
    66:       _,                _,              _,                      _,
    67:       _,                _,              _,                      _,
    68:       P(_GOT_32_6_X)    }},
    69:   { HexagonMCExpr::VK_GOTREL,
    70:     { _,                _,              _,                      _,
    71:       _,                _,              P(_GOTREL_11_X),        P(_GOTREL_11_X),
    72:       P(_GOTREL_11_X),  P(_9_X),        _,                      P(_GOTREL_11_X),
    73:       P(_GOTREL_16_X),  _,              _,                      _,
    74:       P(_GOTREL_16_X),  _,              _,                      _,
    75:       _,                _,              _,                      _,
    76:       _,                _,              _,                      _,
    77:       _,                _,              _,                      _,
    78:       P(_GOTREL_32_6_X) }},
    79:   { HexagonMCExpr::VK_TPREL,
    80:     { _,                _,              _,                      _,
    81:       _,                _,              P(_TPREL_16_X),         P(_TPREL_11_X),
    82:       P(_TPREL_11_X),   P(_9_X),        _,                      P(_TPREL_11_X),
    83:       P(_TPREL_16_X),   _,              _,                      _,
    84:       P(_TPREL_16_X),   _,              _,                      _,
    85:       _,                _,              _,                      _,
    86:       _,                _,              _,                      _,
    87:       _,                _,              _,                      _,
    88:       P(_TPREL_32_6_X)  }},
    89:   { HexagonMCExpr::VK_GD_GOT,
    90:     { _,                _,              _,                      _,
    91:       _,                _,              P(_GD_GOT_16_X),        P(_GD_GOT_11_X),
    92:       P(_GD_GOT_11_X),  P(_9_X),        _,                      P(_GD_GOT_11_X),
    93:       P(_GD_GOT_16_X),  _,              _,                      _,
    94:       P(_GD_GOT_16_X),  _,              _,                      _,
    95:       _,                _,              _,                      _,
    96:       _,                _,              _,                      _,
    97:       _,                _,              _,                      _,
    98:       P(_GD_GOT_32_6_X) }},
    99:   { HexagonMCExpr::VK_GD_PLT,
   100:     { _,                _,              _,                      _,
```
- EN: It imports headers such as MCTargetDesc/HexagonMCCodeEmitter.h, MCTargetDesc/HexagonBaseInfo.h, MCTargetDesc/HexagonFixupKinds.h, MCTargetDesc/HexagonMCExpr.h, ... (26 total), establishing the LLVM/Hexagon APIs used below. Header guards in this range prevent duplicate inclusion and define the interface boundary. It opens namespaces (llvm, Hexagon) to keep the implementation scoped to LLVM/Hexagon components. It declares or implements routines such as STATISTIC, translating Hexagon-specific policy into reusable code paths.
- CN: 这里引入了 MCTargetDesc/HexagonMCCodeEmitter.h, MCTargetDesc/HexagonBaseInfo.h, MCTargetDesc/HexagonFixupKinds.h, MCTargetDesc/HexagonMCExpr.h, ... (26 total) 等头文件，确定了后续代码依赖的 LLVM/Hexagon API。 这一段中的头文件保护宏用于防止重复包含，并明确接口边界。 这里打开了命名空间（llvm, Hexagon），把实现限定在 LLVM/Hexagon 组件作用域中。 这里声明或实现了 STATISTIC 等例程，把 Hexagon 特定策略落实为可复用的代码路径。

### Lines 101-200 / 第 101-200 行

```cpp
   101:       _,                _,              _,                      _,
   102:       _,                P(_9_X),        _,                      P(_GD_PLT_B22_PCREL_X),
   103:       _,                _,              _,                      _,
   104:       _,                _,              _,                      _,
   105:       _,                _,              P(_GD_PLT_B22_PCREL_X), _,
   106:       _,                _,              _,                      _,
   107:       _,                _,              _,                      _,
   108:       _                 }},
   109:   { HexagonMCExpr::VK_IE,
   110:     { _,                _,              _,                      _,
   111:       _,                _,              P(_IE_16_X),            _,
   112:       _,                P(_9_X),        _,                      _,
   113:       P(_IE_16_X),      _,              _,                      _,
   114:       P(_IE_16_X),      _,              _,                      _,
   115:       _,                _,              _,                      _,
   116:       _,                _,              _,                      _,
   117:       _,                _,              _,                      _,
   118:       P(_IE_32_6_X)     }},
   119:   { HexagonMCExpr::VK_IE_GOT,
   120:     { _,                _,              _,                      _,
   121:       _,                _,              P(_IE_GOT_11_X),        P(_IE_GOT_11_X),
   122:       P(_IE_GOT_11_X),  P(_9_X),        _,                      P(_IE_GOT_11_X),
   123:       P(_IE_GOT_16_X),  _,              _,                      _,
   124:       P(_IE_GOT_16_X),  _,              _,                      _,
   125:       _,                _,              _,                      _,
   126:       _,                _,              _,                      _,
   127:       _,                _,              _,                      _,
   128:       P(_IE_GOT_32_6_X) }},
   129:   { HexagonMCExpr::VK_LD_GOT,
   130:     { _,                _,              _,                      _,
   131:       _,                _,              P(_LD_GOT_11_X),        P(_LD_GOT_11_X),
   132:       P(_LD_GOT_11_X),  P(_9_X),        _,                      P(_LD_GOT_11_X),
   133:       P(_LD_GOT_16_X),  _,              _,                      _,
   134:       P(_LD_GOT_16_X),  _,              _,                      _,
   135:       _,                _,              _,                      _,
   136:       _,                _,              _,                      _,
   137:       _,                _,              _,                      _,
   138:       P(_LD_GOT_32_6_X) }},
   139:   { HexagonMCExpr::VK_LD_PLT,
   140:     { _,                _,              _,                      _,
   141:       _,                _,              _,                      _,
   142:       _,                P(_9_X),        _,                      P(_LD_PLT_B22_PCREL_X),
   143:       _,                _,              _,                      _,
   144:       _,                _,              _,                      _,
   145:       _,                _,              P(_LD_PLT_B22_PCREL_X), _,
   146:       _,                _,              _,                      _,
   147:       _,                _,              _,                      _,
   148:       _                 }},
   149:   { HexagonMCExpr::VK_PCREL,
   150:     { _,                _,              _,                      _,
   151:       _,                _,              P(_6_PCREL_X),          _,
   152:       _,                P(_9_X),        _,                      _,
   153:       _,                _,              _,                      _,
   154:       _,                _,              _,                      _,
   155:       _,                _,              _,                      _,
   156:       _,                _,              _,                      _,
   157:       _,                _,              _,                      _,
   158:       P(_32_PCREL)      }},
   159:   { HexagonMCExpr::VK_None,
   160:     { _,                _,              _,                      _,
   161:       _,                _,              P(_6_X),                P(_8_X),
   162:       P(_8_X),          P(_9_X),        P(_10_X),               P(_11_X),
   163:       P(_12_X),         P(_B13_PCREL),  _,                      P(_B15_PCREL_X),
   164:       P(_16_X),         _,              _,                      _,
   165:       _,                _,              P(_B22_PCREL_X),        _,
   166:       _,                _,              _,                      _,
   167:       _,                _,              _,                      _,
   168:       P(_32_6_X)        }},
   169: };
   170: // [1] The fixup is GOT_16_X for signed values and GOT_11_X for unsigned.
   171: 
   172: static const std::map<unsigned, std::vector<unsigned>> StdFixups = {
   173:   { HexagonMCExpr::VK_DTPREL,
   174:     { _,                _,              _,                      _,
   175:       _,                _,              _,                      _,
   176:       _,                _,              _,                      _,
   177:       _,                _,              _,                      _,
   178:       P(_DTPREL_16),    _,              _,                      _,
   179:       _,                _,              _,                      _,
   180:       _,                _,              _,                      _,
   181:       _,                _,              _,                      _,
   182:       P(_DTPREL_32)     }},
   183:   { HexagonMCExpr::VK_GOT,
   184:     { _,                _,              _,                      _,
   185:       _,                _,              _,                      _,
   186:       _,                _,              _,                      _,
   187:       _,                _,              _,                      _,
   188:       _,                _,              _,                      _,
   189:       _,                _,              _,                      _,
   190:       _,                _,              _,                      _,
   191:       _,                _,              _,                      _,
   192:       P(_GOT_32)        }},
   193:   { HexagonMCExpr::VK_GOTREL,
   194:     { _,                _,              _,                      _,
   195:       _,                _,              _,                      _,
   196:       _,                _,              _,                      _,
   197:       _,                _,              _,                      _,
   198:       _ /* [2] */,      _,              _,                      _,
   199:       _,                _,              _,                      _,
   200:       _,                _,              _,                      _,
```
- EN: Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonMCExpr, showing how the code connects to sibling backend components.
- CN: 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonMCExpr，说明了它与同级后端组件的连接关系。

### Lines 201-300 / 第 201-300 行

```cpp
   201:       _,                _,              _,                      _,
   202:       P(_GOTREL_32)     }},
   203:   { HexagonMCExpr::VK_PLT,
   204:     { _,                _,              _,                      _,
   205:       _,                _,              _,                      _,
   206:       _,                _,              _,                      _,
   207:       _,                _,              _,                      _,
   208:       _,                _,              _,                      _,
   209:       _,                _,              P(_PLT_B22_PCREL),      _,
   210:       _,                _,              _,                      _,
   211:       _,                _,              _,                      _,
   212:       _                 }},
   213:   { HexagonMCExpr::VK_TPREL,
   214:     { _,                _,              _,                      _,
   215:       _,                _,              _,                      _,
   216:       _,                _,              _,                      P(_TPREL_11_X),
   217:       _,                _,              _,                      _,
   218:       P(_TPREL_16),     _,              _,                      _,
   219:       _,                _,              _,                      _,
   220:       _,                _,              _,                      _,
   221:       _,                _,              _,                      _,
   222:       P(_TPREL_32)      }},
   223:   { HexagonMCExpr::VK_GD_GOT,
   224:     { _,                _,              _,                      _,
   225:       _,                _,              _,                      _,
   226:       _,                _,              _,                      _,
   227:       _,                _,              _,                      _,
   228:       P(_GD_GOT_16),    _,              _,                      _,
   229:       _,                _,              _,                      _,
   230:       _,                _,              _,                      _,
   231:       _,                _,              _,                      _,
   232:       P(_GD_GOT_32)     }},
   233:   { HexagonMCExpr::VK_GD_PLT,
   234:     { _,                _,              _,                      _,
   235:       _,                _,              _,                      _,
   236:       _,                _,              _,                      _,
   237:       _,                _,              _,                      _,
   238:       _,                _,              _,                      _,
   239:       _,                _,              P(_GD_PLT_B22_PCREL),   _,
   240:       _,                _,              _,                      _,
   241:       _,                _,              _,                      _,
   242:       _                 }},
   243:   { HexagonMCExpr::VK_GPREL,
   244:     { _,                _,              _,                      _,
   245:       _,                _,              _,                      _,
   246:       _,                _,              _,                      _,
   247:       _,                _,              _,                      _,
   248:       P(_GPREL16_0),    _,              _,                      _,
   249:       _,                _,              _,                      _,
   250:       _,                _,              _,                      _,
   251:       _,                _,              _,                      _,
   252:       _                 }},
   253:   { HexagonMCExpr::VK_HI16,
   254:     { _,                _,              _,                      _,
   255:       _,                _,              _,                      _,
   256:       _,                _,              _,                      _,
   257:       _,                _,              _,                      _,
   258:       P(_HI16),         _,              _,                      _,
   259:       _,                _,              _,                      _,
   260:       _,                _,              _,                      _,
   261:       _,                _,              _,                      _,
   262:       _                 }},
   263:   { HexagonMCExpr::VK_IE,
   264:     { _,                _,              _,                      _,
   265:       _,                _,              _,                      _,
   266:       _,                _,              _,                      _,
   267:       _,                _,              _,                      _,
   268:       _,                _,              _,                      _,
   269:       _,                _,              _,                      _,
   270:       _,                _,              _,                      _,
   271:       _,                _,              _,                      _,
   272:       P(_IE_32)         }},
   273:   { HexagonMCExpr::VK_IE_GOT,
   274:     { _,                _,              _,                      _,
   275:       _,                _,              _,                      _,
   276:       _,                _,              _,                      _,
   277:       _,                _,              _,                      _,
   278:       P(_IE_GOT_16),    _,              _,                      _,
   279:       _,                _,              _,                      _,
   280:       _,                _,              _,                      _,
   281:       _,                _,              _,                      _,
   282:       P(_IE_GOT_32)     }},
   283:   { HexagonMCExpr::VK_LD_GOT,
   284:     { _,                _,              _,                      _,
   285:       _,                _,              _,                      _,
   286:       _,                _,              _,                      _,
   287:       _,                _,              _,                      _,
   288:       P(_LD_GOT_16),    _,              _,                      _,
   289:       _,                _,              _,                      _,
   290:       _,                _,              _,                      _,
   291:       _,                _,              _,                      _,
   292:       P(_LD_GOT_32)     }},
   293:   { HexagonMCExpr::VK_LD_PLT,
   294:     { _,                _,              _,                      _,
   295:       _,                _,              _,                      _,
   296:       _,                _,              _,                      _,
   297:       _,                _,              _,                      _,
   298:       _,                _,              _,                      _,
   299:       _,                _,              P(_LD_PLT_B22_PCREL),   _,
   300:       _,                _,              _,                      _,
```
- EN: Notable Hexagon symbols referenced here include HexagonMCExpr, showing how the code connects to sibling backend components.
- CN: 这里引用的重要 Hexagon 符号包括 HexagonMCExpr，说明了它与同级后端组件的连接关系。

### Lines 301-400 / 第 301-400 行

```cpp
   301:       _,                _,              _,                      _,
   302:       _                 }},
   303:   { HexagonMCExpr::VK_LO16,
   304:     { _,                _,              _,                      _,
   305:       _,                _,              _,                      _,
   306:       _,                _,              _,                      _,
   307:       _,                _,              _,                      _,
   308:       P(_LO16),         _,              _,                      _,
   309:       _,                _,              _,                      _,
   310:       _,                _,              _,                      _,
   311:       _,                _,              _,                      _,
   312:       _                 }},
   313:   { HexagonMCExpr::VK_PCREL,
   314:     { _,                _,              _,                      _,
   315:       _,                _,              _,                      _,
   316:       _,                _,              _,                      _,
   317:       _,                _,              _,                      _,
   318:       _,                _,              _,                      _,
   319:       _,                _,              _,                      _,
   320:       _,                _,              _,                      _,
   321:       _,                _,              _,                      _,
   322:       P(_32_PCREL)      }},
   323:   { HexagonMCExpr::VK_None,
   324:     { _,                _,              _,                      _,
   325:       _,                _,              _,                      _,
   326:       _,                _,              _,                      _,
   327:       _,                P(_B13_PCREL),  _,                      P(_B15_PCREL),
   328:       _,                _,              _,                      _,
   329:       _,                _,              P(_B22_PCREL),          _,
   330:       _,                _,              _,                      _,
   331:       _,                _,              _,                      _,
   332:       P(_32)            }},
   333: };
   334: // clang-format on
   335: // [2] The actual fixup is LO16 or HI16, depending on the instruction.
   336: #undef P
   337: #undef _
   338: 
   339: static void addFixup(SmallVectorImpl<MCFixup> &Fixups, uint32_t Offset,
   340:                      const MCExpr *Value, uint16_t Kind) {
   341:   bool PCRel = false;
   342:   switch (Kind) {
   343:   case Hexagon::fixup_Hexagon_B22_PCREL:
   344:   case Hexagon::fixup_Hexagon_B15_PCREL:
   345:   case Hexagon::fixup_Hexagon_B7_PCREL:
   346:   case Hexagon::fixup_Hexagon_B13_PCREL:
   347:   case Hexagon::fixup_Hexagon_B9_PCREL:
   348:   case Hexagon::fixup_Hexagon_B32_PCREL_X:
   349:   case Hexagon::fixup_Hexagon_B22_PCREL_X:
   350:   case Hexagon::fixup_Hexagon_B15_PCREL_X:
   351:   case Hexagon::fixup_Hexagon_B13_PCREL_X:
   352:   case Hexagon::fixup_Hexagon_B9_PCREL_X:
   353:   case Hexagon::fixup_Hexagon_B7_PCREL_X:
   354:   case Hexagon::fixup_Hexagon_32_PCREL:
   355:   case Hexagon::fixup_Hexagon_PLT_B22_PCREL:
   356:   case Hexagon::fixup_Hexagon_GD_PLT_B22_PCREL:
   357:   case Hexagon::fixup_Hexagon_LD_PLT_B22_PCREL:
   358:   case Hexagon::fixup_Hexagon_6_PCREL_X:
   359:   case Hexagon::fixup_Hexagon_GD_PLT_B22_PCREL_X:
   360:   case Hexagon::fixup_Hexagon_GD_PLT_B32_PCREL_X:
   361:   case Hexagon::fixup_Hexagon_LD_PLT_B22_PCREL_X:
   362:   case Hexagon::fixup_Hexagon_LD_PLT_B32_PCREL_X:
   363:     PCRel = true;
   364:   }
   365:   Fixups.push_back(MCFixup::create(Offset, Value, Kind, PCRel));
   366: }
   367: 
   368: uint32_t HexagonMCCodeEmitter::parseBits(size_t Last, MCInst const &MCB,
   369:                                          MCInst const &MCI) const {
   370:   bool Duplex = HexagonMCInstrInfo::isDuplex(MCII, MCI);
   371:   if (State.Index == 0) {
   372:     if (HexagonMCInstrInfo::isInnerLoop(MCB)) {
   373:       assert(!Duplex);
   374:       assert(State.Index != Last);
   375:       return HexagonII::INST_PARSE_LOOP_END;
   376:     }
   377:   }
   378:   if (State.Index == 1) {
   379:     if (HexagonMCInstrInfo::isOuterLoop(MCB)) {
   380:       assert(!Duplex);
   381:       assert(State.Index != Last);
   382:       return HexagonII::INST_PARSE_LOOP_END;
   383:     }
   384:   }
   385:   if (Duplex) {
   386:     assert(State.Index == Last);
   387:     return HexagonII::INST_PARSE_DUPLEX;
   388:   }
   389:   if (State.Index == Last)
   390:     return HexagonII::INST_PARSE_PACKET_END;
   391:   return HexagonII::INST_PARSE_NOT_END;
   392: }
   393: 
   394: /// Emit the bundle.
   395: void HexagonMCCodeEmitter::encodeInstruction(const MCInst &MI,
   396:                                              SmallVectorImpl<char> &CB,
   397:                                              SmallVectorImpl<MCFixup> &Fixups,
   398:                                              const MCSubtargetInfo &STI) const {
   399:   MCInst &HMB = const_cast<MCInst &>(MI);
   400: 
```
- EN: It declares or implements routines such as addFixup, push_back, HexagonMCCodeEmitter::parseBits, HexagonMCInstrInfo::isDuplex, ... (6 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage. Notable Hexagon symbols referenced here include HexagonMCExpr, HexagonMCCodeEmitter, HexagonMCInstrInfo, HexagonII, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 addFixup, push_back, HexagonMCCodeEmitter::parseBits, HexagonMCInstrInfo::isDuplex, ... (6 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。 这里引用的重要 Hexagon 符号包括 HexagonMCExpr, HexagonMCCodeEmitter, HexagonMCInstrInfo, HexagonII，说明了它与同级后端组件的连接关系。

### Lines 401-500 / 第 401-500 行

```cpp
   401:   assert(HexagonMCInstrInfo::isBundle(HMB));
   402:   LLVM_DEBUG(dbgs() << "Encoding bundle\n";);
   403:   State.Addend = 0;
   404:   State.Extended = false;
   405:   State.Bundle = &MI;
   406:   State.Index = 0;
   407:   size_t Last = HexagonMCInstrInfo::bundleSize(HMB) - 1;
   408: 
   409:   for (auto &I : HexagonMCInstrInfo::bundleInstructions(HMB)) {
   410:     MCInst &HMI = const_cast<MCInst &>(*I.getInst());
   411: 
   412:     encodeSingleInstruction(HMI, CB, Fixups, STI, parseBits(Last, HMB, HMI));
   413:     State.Extended = HexagonMCInstrInfo::isImmext(HMI);
   414:     State.Addend += HEXAGON_INSTR_SIZE;
   415:     ++State.Index;
   416:   }
   417: }
   418: 
   419: static bool RegisterMatches(MCRegister Consumer, MCRegister Producer,
   420:                             MCRegister Producer2) {
   421:   return (Consumer == Producer) || (Consumer == Producer2) ||
   422:          HexagonMCInstrInfo::IsSingleConsumerRefPairProducer(Producer,
   423:                                                              Consumer);
   424: }
   425: 
   426: void HexagonMCCodeEmitter::encodeSingleInstruction(
   427:     const MCInst &MI, SmallVectorImpl<char> &CB,
   428:     SmallVectorImpl<MCFixup> &Fixups, const MCSubtargetInfo &STI,
   429:     uint32_t Parse) const {
   430:   assert(!HexagonMCInstrInfo::isBundle(MI));
   431:   uint64_t Binary;
   432: 
   433:   // Pseudo instructions don't get encoded and shouldn't be here
   434:   // in the first place!
   435:   assert(!HexagonMCInstrInfo::getDesc(MCII, MI).isPseudo() &&
   436:          "pseudo-instruction found");
   437:   LLVM_DEBUG(dbgs() << "Encoding insn `"
   438:                     << HexagonMCInstrInfo::getName(MCII, MI) << "'\n");
   439: 
   440:   Binary = getBinaryCodeForInstr(MI, Fixups, STI);
   441:   unsigned Opc = MI.getOpcode();
   442: 
   443:   // Check for unimplemented instructions. Immediate extenders
   444:   // are encoded as zero, so they need to be accounted for.
   445:   if (!Binary && Opc != DuplexIClass0 && Opc != A4_ext) {
   446:     LLVM_DEBUG(dbgs() << "Unimplemented inst `"
   447:                       << HexagonMCInstrInfo::getName(MCII, MI) << "'\n");
   448:     llvm_unreachable("Unimplemented Instruction");
   449:   }
   450:   Binary |= Parse;
   451: 
   452:   // if we need to emit a duplexed instruction
   453:   if (Opc >= Hexagon::DuplexIClass0 && Opc <= Hexagon::DuplexIClassF) {
   454:     assert(Parse == HexagonII::INST_PARSE_DUPLEX &&
   455:            "Emitting duplex without duplex parse bits");
   456:     unsigned DupIClass = MI.getOpcode() - Hexagon::DuplexIClass0;
   457:     // 29 is the bit position.
   458:     // 0b1110 =0xE bits are masked off and down shifted by 1 bit.
   459:     // Last bit is moved to bit position 13
   460:     Binary = ((DupIClass & 0xE) << (29 - 1)) | ((DupIClass & 0x1) << 13);
   461: 
   462:     const MCInst *Sub0 = MI.getOperand(0).getInst();
   463:     const MCInst *Sub1 = MI.getOperand(1).getInst();
   464: 
   465:     // Get subinstruction slot 0.
   466:     unsigned SubBits0 = getBinaryCodeForInstr(*Sub0, Fixups, STI);
   467:     // Get subinstruction slot 1.
   468:     State.SubInst1 = true;
   469:     unsigned SubBits1 = getBinaryCodeForInstr(*Sub1, Fixups, STI);
   470:     State.SubInst1 = false;
   471: 
   472:     Binary |= SubBits0 | (SubBits1 << 16);
   473:   }
   474:   support::endian::write<uint32_t>(CB, Binary, llvm::endianness::little);
   475:   ++MCNumEmitted;
   476: }
   477: 
   478: [[noreturn]] static void raise_relocation_error(unsigned Width, unsigned Kind) {
   479:   std::string Text;
   480:   raw_string_ostream Stream(Text);
   481:   Stream << "Unrecognized relocation combination: width=" << Width
   482:          << " kind=" << Kind;
   483:   report_fatal_error(Twine(Stream.str()));
   484: }
   485: 
   486: /// Some insns are not extended and thus have no bits. These cases require
   487: /// a more brute force method for determining the correct relocation.
   488: Hexagon::Fixups
   489: HexagonMCCodeEmitter::getFixupNoBits(MCInstrInfo const &MCII, const MCInst &MI,
   490:                                      const MCOperand &MO,
   491:                                      HexagonMCExpr::VariantKind VarKind) const {
   492:   const MCInstrDesc &MCID = HexagonMCInstrInfo::getDesc(MCII, MI);
   493:   unsigned InsnType = HexagonMCInstrInfo::getType(MCII, MI);
   494:   using namespace Hexagon;
   495: 
   496:   if (InsnType == HexagonII::TypeEXTENDER) {
   497:     if (VarKind == HexagonMCExpr::VK_None) {
   498:       auto Instrs = HexagonMCInstrInfo::bundleInstructions(*State.Bundle);
   499:       for (auto I = Instrs.begin(), N = Instrs.end(); I != N; ++I) {
   500:         if (I->getInst() != &MI)
```
- EN: It opens namespaces (Hexagon) to keep the implementation scoped to LLVM/Hexagon components. It declares or implements routines such as assert, getInst, encodeSingleInstruction, HexagonMCInstrInfo::isImmext, ... (19 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage.
- CN: 这里打开了命名空间（Hexagon），把实现限定在 LLVM/Hexagon 组件作用域中。 这里声明或实现了 assert, getInst, encodeSingleInstruction, HexagonMCInstrInfo::isImmext, ... (19 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。

### Lines 501-600 / 第 501-600 行

```cpp
   501:           continue;
   502:         assert(I+1 != N && "Extender cannot be last in packet");
   503:         const MCInst &NextI = *(I+1)->getInst();
   504:         const MCInstrDesc &NextD = HexagonMCInstrInfo::getDesc(MCII, NextI);
   505:         if (NextD.isBranch() || NextD.isCall() ||
   506:             HexagonMCInstrInfo::getType(MCII, NextI) == HexagonII::TypeCR)
   507:           return fixup_Hexagon_B32_PCREL_X;
   508:         return fixup_Hexagon_32_6_X;
   509:       }
   510:     }
   511: 
   512:     static const std::map<unsigned, unsigned> Relocs = {
   513:         {HexagonMCExpr::VK_GOTREL, fixup_Hexagon_GOTREL_32_6_X},
   514:         {HexagonMCExpr::VK_GOT, fixup_Hexagon_GOT_32_6_X},
   515:         {HexagonMCExpr::VK_TPREL, fixup_Hexagon_TPREL_32_6_X},
   516:         {HexagonMCExpr::VK_DTPREL, fixup_Hexagon_DTPREL_32_6_X},
   517:         {HexagonMCExpr::VK_GD_GOT, fixup_Hexagon_GD_GOT_32_6_X},
   518:         {HexagonMCExpr::VK_LD_GOT, fixup_Hexagon_LD_GOT_32_6_X},
   519:         {HexagonMCExpr::VK_IE, fixup_Hexagon_IE_32_6_X},
   520:         {HexagonMCExpr::VK_IE_GOT, fixup_Hexagon_IE_GOT_32_6_X},
   521:         {HexagonMCExpr::VK_PCREL, fixup_Hexagon_B32_PCREL_X},
   522:         {HexagonMCExpr::VK_GD_PLT, fixup_Hexagon_GD_PLT_B32_PCREL_X},
   523:         {HexagonMCExpr::VK_LD_PLT, fixup_Hexagon_LD_PLT_B32_PCREL_X},
   524:     };
   525: 
   526:     auto F = Relocs.find(VarKind);
   527:     if (F != Relocs.end())
   528:       return Hexagon::Fixups(F->second);
   529:     raise_relocation_error(0, VarKind);
   530:   }
   531: 
   532:   if (MCID.isBranch())
   533:     return fixup_Hexagon_B13_PCREL;
   534: 
   535:   static const std::map<unsigned, unsigned> RelocsLo = {
   536:       {HexagonMCExpr::VK_GOT, fixup_Hexagon_GOT_LO16},
   537:       {HexagonMCExpr::VK_GOTREL, fixup_Hexagon_GOTREL_LO16},
   538:       {HexagonMCExpr::VK_GD_GOT, fixup_Hexagon_GD_GOT_LO16},
   539:       {HexagonMCExpr::VK_LD_GOT, fixup_Hexagon_LD_GOT_LO16},
   540:       {HexagonMCExpr::VK_IE, fixup_Hexagon_IE_LO16},
   541:       {HexagonMCExpr::VK_IE_GOT, fixup_Hexagon_IE_GOT_LO16},
   542:       {HexagonMCExpr::VK_TPREL, fixup_Hexagon_TPREL_LO16},
   543:       {HexagonMCExpr::VK_DTPREL, fixup_Hexagon_DTPREL_LO16},
   544:       {HexagonMCExpr::VK_None, fixup_Hexagon_LO16},
   545:   };
   546: 
   547:   static const std::map<unsigned, unsigned> RelocsHi = {
   548:       {HexagonMCExpr::VK_GOT, fixup_Hexagon_GOT_HI16},
   549:       {HexagonMCExpr::VK_GOTREL, fixup_Hexagon_GOTREL_HI16},
   550:       {HexagonMCExpr::VK_GD_GOT, fixup_Hexagon_GD_GOT_HI16},
   551:       {HexagonMCExpr::VK_LD_GOT, fixup_Hexagon_LD_GOT_HI16},
   552:       {HexagonMCExpr::VK_IE, fixup_Hexagon_IE_HI16},
   553:       {HexagonMCExpr::VK_IE_GOT, fixup_Hexagon_IE_GOT_HI16},
   554:       {HexagonMCExpr::VK_TPREL, fixup_Hexagon_TPREL_HI16},
   555:       {HexagonMCExpr::VK_DTPREL, fixup_Hexagon_DTPREL_HI16},
   556:       {HexagonMCExpr::VK_None, fixup_Hexagon_HI16},
   557:   };
   558: 
   559:   switch (MCID.getOpcode()) {
   560:     case Hexagon::LO:
   561:     case Hexagon::A2_tfril: {
   562:       auto F = RelocsLo.find(VarKind);
   563:       if (F != RelocsLo.end())
   564:         return Hexagon::Fixups(F->second);
   565:       break;
   566:     }
   567:     case Hexagon::HI:
   568:     case Hexagon::A2_tfrih: {
   569:       auto F = RelocsHi.find(VarKind);
   570:       if (F != RelocsHi.end())
   571:         return Hexagon::Fixups(F->second);
   572:       break;
   573:     }
   574:   }
   575: 
   576:   raise_relocation_error(0, VarKind);
   577: }
   578: 
   579: static bool isPCRel(unsigned Kind) {
   580:   switch (Kind){
   581:   case fixup_Hexagon_B22_PCREL:
   582:   case fixup_Hexagon_B15_PCREL:
   583:   case fixup_Hexagon_B7_PCREL:
   584:   case fixup_Hexagon_B13_PCREL:
   585:   case fixup_Hexagon_B9_PCREL:
   586:   case fixup_Hexagon_B32_PCREL_X:
   587:   case fixup_Hexagon_B22_PCREL_X:
   588:   case fixup_Hexagon_B15_PCREL_X:
   589:   case fixup_Hexagon_B13_PCREL_X:
   590:   case fixup_Hexagon_B9_PCREL_X:
   591:   case fixup_Hexagon_B7_PCREL_X:
   592:   case fixup_Hexagon_32_PCREL:
   593:   case fixup_Hexagon_PLT_B22_PCREL:
   594:   case fixup_Hexagon_GD_PLT_B22_PCREL:
   595:   case fixup_Hexagon_LD_PLT_B22_PCREL:
   596:   case fixup_Hexagon_GD_PLT_B22_PCREL_X:
   597:   case fixup_Hexagon_LD_PLT_B22_PCREL_X:
   598:   case fixup_Hexagon_6_PCREL_X:
   599:     return true;
   600:   default:
```
- EN: It declares or implements routines such as assert, getInst, HexagonMCInstrInfo::getDesc, find, ... (6 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage. Notable Hexagon symbols referenced here include HexagonMCInstrInfo, HexagonII, HexagonMCExpr, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 assert, getInst, HexagonMCInstrInfo::getDesc, find, ... (6 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。 这里引用的重要 Hexagon 符号包括 HexagonMCInstrInfo, HexagonII, HexagonMCExpr，说明了它与同级后端组件的连接关系。

### Lines 601-700 / 第 601-700 行

```cpp
   601:     return false;
   602:   }
   603: }
   604: 
   605: unsigned HexagonMCCodeEmitter::getExprOpValue(const MCInst &MI,
   606:       const MCOperand &MO, const MCExpr *ME, SmallVectorImpl<MCFixup> &Fixups,
   607:       const MCSubtargetInfo &STI) const {
   608:   if (isa<HexagonMCExpr>(ME))
   609:     ME = &HexagonMCInstrInfo::getExpr(*ME);
   610:   int64_t Value;
   611:   if (ME->evaluateAsAbsolute(Value)) {
   612:     bool InstExtendable = HexagonMCInstrInfo::isExtendable(MCII, MI) ||
   613:                           HexagonMCInstrInfo::isExtended(MCII, MI);
   614:     // Only sub-instruction #1 can be extended in a duplex. If MI is a
   615:     // sub-instruction #0, it is not extended even if Extended is true
   616:     // (it can be true for the duplex as a whole).
   617:     bool IsSub0 = HexagonMCInstrInfo::isSubInstruction(MI) && !State.SubInst1;
   618:     if (State.Extended && InstExtendable && !IsSub0) {
   619:       unsigned OpIdx = ~0u;
   620:       for (unsigned I = 0, E = MI.getNumOperands(); I != E; ++I) {
   621:         if (&MO != &MI.getOperand(I))
   622:           continue;
   623:         OpIdx = I;
   624:         break;
   625:       }
   626:       assert(OpIdx != ~0u);
   627:       if (OpIdx == HexagonMCInstrInfo::getExtendableOp(MCII, MI)) {
   628:         unsigned Shift = HexagonMCInstrInfo::getExtentAlignment(MCII, MI);
   629:         Value = (Value & 0x3f) << Shift;
   630:       }
   631:     }
   632:     return Value;
   633:   }
   634:   assert(ME->getKind() == MCExpr::SymbolRef ||
   635:          ME->getKind() == MCExpr::Binary);
   636:   if (ME->getKind() == MCExpr::Binary) {
   637:     MCBinaryExpr const *Binary = cast<MCBinaryExpr>(ME);
   638:     getExprOpValue(MI, MO, Binary->getLHS(), Fixups, STI);
   639:     getExprOpValue(MI, MO, Binary->getRHS(), Fixups, STI);
   640:     return 0;
   641:   }
   642: 
   643:   unsigned FixupKind = fixup_Invalid;
   644:   const MCSymbolRefExpr *MCSRE = static_cast<const MCSymbolRefExpr *>(ME);
   645:   const MCInstrDesc &MCID = HexagonMCInstrInfo::getDesc(MCII, MI);
   646:   unsigned FixupWidth = HexagonMCInstrInfo::getExtentBits(MCII, MI) -
   647:                         HexagonMCInstrInfo::getExtentAlignment(MCII, MI);
   648:   auto VarKind = HexagonMCExpr::VariantKind(MCSRE->getSpecifier());
   649:   unsigned Opc = MCID.getOpcode();
   650:   unsigned IType = HexagonMCInstrInfo::getType(MCII, MI);
   651: 
   652:   LLVM_DEBUG(dbgs() << "----------------------------------------\n"
   653:                     << "Opcode Name: " << HexagonMCInstrInfo::getName(MCII, MI)
   654:                     << "\nOpcode: " << Opc << "\nRelocation bits: "
   655:                     << FixupWidth << "\nAddend: " << State.Addend
   656:                     << "\nVariant: " << unsigned(VarKind)
   657:                     << "\n----------------------------------------\n");
   658: 
   659:   // Pick the applicable fixup kind for the symbol.
   660:   // Handle special cases first, the rest will be looked up in the tables.
   661: 
   662:   if (FixupWidth == 16 && !State.Extended) {
   663:     if (VarKind == HexagonMCExpr::VK_None) {
   664:       if (HexagonMCInstrInfo::s27_2_reloc(*MO.getExpr())) {
   665:         // A2_iconst.
   666:         FixupKind = Hexagon::fixup_Hexagon_27_REG;
   667:       } else {
   668:         // Look for GP-relative fixups.
   669:         unsigned Shift = HexagonMCInstrInfo::getExtentAlignment(MCII, MI);
   670:         static const Hexagon::Fixups GPRelFixups[] = {
   671:           Hexagon::fixup_Hexagon_GPREL16_0, Hexagon::fixup_Hexagon_GPREL16_1,
   672:           Hexagon::fixup_Hexagon_GPREL16_2, Hexagon::fixup_Hexagon_GPREL16_3
   673:         };
   674:         assert(Shift < std::size(GPRelFixups));
   675:         auto UsesGP = [](const MCInstrDesc &D) {
   676:           return is_contained(D.implicit_uses(), Hexagon::GP);
   677:         };
   678:         if (UsesGP(MCID))
   679:           FixupKind = GPRelFixups[Shift];
   680:       }
   681:     } else if (VarKind == HexagonMCExpr::VK_GOTREL) {
   682:       // Select between LO/HI.
   683:       if (Opc == Hexagon::LO)
   684:         FixupKind = Hexagon::fixup_Hexagon_GOTREL_LO16;
   685:       else if (Opc == Hexagon::HI)
   686:         FixupKind = Hexagon::fixup_Hexagon_GOTREL_HI16;
   687:     }
   688:   } else {
   689:     bool BranchOrCR = MCID.isBranch() || IType == HexagonII::TypeCR;
   690:     switch (FixupWidth) {
   691:       case 9:
   692:         if (BranchOrCR)
   693:           FixupKind = State.Extended ? Hexagon::fixup_Hexagon_B9_PCREL_X
   694:                                      : Hexagon::fixup_Hexagon_B9_PCREL;
   695:         break;
   696:       case 8:
   697:       case 7:
   698:         if (State.Extended && VarKind == HexagonMCExpr::VK_GOT)
   699:           FixupKind = HexagonMCInstrInfo::isExtentSigned(MCII, MI)
   700:                         ? Hexagon::fixup_Hexagon_GOT_16_X
```
- EN: It declares or implements routines such as HexagonMCCodeEmitter::getExprOpValue, HexagonMCInstrInfo::isExtendable, assert, HexagonMCInstrInfo::getExtentAlignment, ... (13 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage. Notable Hexagon symbols referenced here include HexagonMCCodeEmitter, HexagonMCExpr, HexagonMCInstrInfo, HexagonII, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 HexagonMCCodeEmitter::getExprOpValue, HexagonMCInstrInfo::isExtendable, assert, HexagonMCInstrInfo::getExtentAlignment, ... (13 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。 这里引用的重要 Hexagon 符号包括 HexagonMCCodeEmitter, HexagonMCExpr, HexagonMCInstrInfo, HexagonII，说明了它与同级后端组件的连接关系。

### Lines 701-800 / 第 701-800 行

```cpp
   701:                         : Hexagon::fixup_Hexagon_GOT_11_X;
   702:         else if (FixupWidth == 7 && BranchOrCR)
   703:           FixupKind = State.Extended ? Hexagon::fixup_Hexagon_B7_PCREL_X
   704:                                      : Hexagon::fixup_Hexagon_B7_PCREL;
   705:         break;
   706:       case 0:
   707:         FixupKind = getFixupNoBits(MCII, MI, MO, VarKind);
   708:         break;
   709:     }
   710:   }
   711: 
   712:   if (FixupKind == fixup_Invalid) {
   713:     const auto &FixupTable = State.Extended ? ExtFixups : StdFixups;
   714: 
   715:     auto FindVK = FixupTable.find(VarKind);
   716:     if (FindVK != FixupTable.end())
   717:       FixupKind = FindVK->second[FixupWidth];
   718:   }
   719: 
   720:   if (FixupKind == fixup_Invalid)
   721:     raise_relocation_error(FixupWidth, VarKind);
   722: 
   723:   const MCExpr *FixupExpr = MO.getExpr();
   724:   if (State.Addend != 0 && isPCRel(FixupKind)) {
   725:     const MCExpr *C = MCConstantExpr::create(State.Addend, MCT);
   726:     FixupExpr = MCBinaryExpr::createAdd(FixupExpr, C, MCT);
   727:   }
   728: 
   729:   addFixup(Fixups, State.Addend, FixupExpr, FixupKind);
   730:   return 0;
   731: }
   732: 
   733: unsigned
   734: HexagonMCCodeEmitter::getMachineOpValue(MCInst const &MI, MCOperand const &MO,
   735:                                         SmallVectorImpl<MCFixup> &Fixups,
   736:                                         MCSubtargetInfo const &STI) const {
   737:   size_t OperandNumber = ~0U;
   738:   for (unsigned i = 0, n = MI.getNumOperands(); i < n; ++i)
   739:     if (&MI.getOperand(i) == &MO) {
   740:       OperandNumber = i;
   741:       break;
   742:     }
   743:   assert((OperandNumber != ~0U) && "Operand not found");
   744: 
   745:   if (HexagonMCInstrInfo::isNewValue(MCII, MI) &&
   746:       &MO == &HexagonMCInstrInfo::getNewValueOperand(MCII, MI)) {
   747:     // Calculate the new value distance to the associated producer
   748:     unsigned SOffset = 0;
   749:     unsigned VOffset = 0;
   750:     MCRegister UseReg = MO.getReg();
   751:     MCRegister DefReg1;
   752:     MCRegister DefReg2;
   753: 
   754:     auto Instrs = HexagonMCInstrInfo::bundleInstructions(*State.Bundle);
   755:     const MCOperand *I = Instrs.begin() + State.Index - 1;
   756: 
   757:     for (;; --I) {
   758:       assert(I != Instrs.begin() - 1 && "Couldn't find producer");
   759:       MCInst const &Inst = *I->getInst();
   760:       if (HexagonMCInstrInfo::isImmext(Inst))
   761:         continue;
   762: 
   763:       DefReg1 = MCRegister();
   764:       DefReg2 = MCRegister();
   765:       ++SOffset;
   766:       if (HexagonMCInstrInfo::isVector(MCII, Inst)) {
   767:         // Vector instructions don't count scalars.
   768:         ++VOffset;
   769:       }
   770:       if (HexagonMCInstrInfo::hasNewValue(MCII, Inst))
   771:         DefReg1 = HexagonMCInstrInfo::getNewValueOperand(MCII, Inst).getReg();
   772:       if (HexagonMCInstrInfo::hasNewValue2(MCII, Inst))
   773:         DefReg2 = HexagonMCInstrInfo::getNewValueOperand2(MCII, Inst).getReg();
   774:       if (!RegisterMatches(UseReg, DefReg1, DefReg2)) {
   775:         // This isn't the register we're looking for
   776:         continue;
   777:       }
   778:       if (!HexagonMCInstrInfo::isPredicated(MCII, Inst)) {
   779:         // Producer is unpredicated
   780:         break;
   781:       }
   782:       assert(HexagonMCInstrInfo::isPredicated(MCII, MI) &&
   783:              "Unpredicated consumer depending on predicated producer");
   784:       if (HexagonMCInstrInfo::isPredicatedTrue(MCII, Inst) ==
   785:           HexagonMCInstrInfo::isPredicatedTrue(MCII, MI))
   786:         // Producer predicate sense matched ours.
   787:         break;
   788:     }
   789:     // Hexagon PRM 10.11 Construct Nt from distance
   790:     unsigned Offset = HexagonMCInstrInfo::isVector(MCII, MI) ? VOffset
   791:                                                              : SOffset;
   792:     Offset <<= 1;
   793:     Offset |= HexagonMCInstrInfo::SubregisterBit(UseReg, DefReg1, DefReg2);
   794:     return Offset;
   795:   }
   796: 
   797:   assert(!MO.isImm());
   798:   if (MO.isReg()) {
   799:     MCRegister Reg = MO.getReg();
   800:     switch (HexagonMCInstrInfo::getDesc(MCII, MI)
```
- EN: It declares or implements routines such as getFixupNoBits, find, getExpr, MCConstantExpr::create, ... (13 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage. Notable Hexagon symbols referenced here include HexagonMCCodeEmitter, HexagonMCInstrInfo, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 getFixupNoBits, find, getExpr, MCConstantExpr::create, ... (13 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。 这里引用的重要 Hexagon 符号包括 HexagonMCCodeEmitter, HexagonMCInstrInfo，说明了它与同级后端组件的连接关系。

### Lines 801-820 / 第 801-820 行

```cpp
   801:                 .operands()[OperandNumber]
   802:                 .RegClass) {
   803:     case GeneralSubRegsRegClassID:
   804:     case GeneralDoubleLow8RegsRegClassID:
   805:       return HexagonMCInstrInfo::getDuplexRegisterNumbering(Reg);
   806:     default:
   807:       break;
   808:     }
   809:     return MCT.getRegisterInfo()->getEncodingValue(Reg);
   810:   }
   811: 
   812:   return getExprOpValue(MI, MO, MO.getExpr(), Fixups, STI);
   813: }
   814: 
   815: MCCodeEmitter *llvm::createHexagonMCCodeEmitter(MCInstrInfo const &MII,
   816:                                                 MCContext &MCT) {
   817:   return new HexagonMCCodeEmitter(MII, MCT);
   818: }
   819: 
   820: #include "HexagonGenMCCodeEmitter.inc"
```
- EN: It imports headers such as HexagonGenMCCodeEmitter.inc, establishing the LLVM/Hexagon APIs used below. It declares or implements routines such as operands, HexagonMCInstrInfo::getDuplexRegisterNumbering, getRegisterInfo, getExprOpValue, ... (6 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonMCInstrInfo, HexagonMCCodeEmitter, HexagonGenMCCodeEmitter, showing how the code connects to sibling backend components.
- CN: 这里引入了 HexagonGenMCCodeEmitter.inc 等头文件，确定了后续代码依赖的 LLVM/Hexagon API。 这里声明或实现了 operands, HexagonMCInstrInfo::getDuplexRegisterNumbering, getRegisterInfo, getExprOpValue, ... (6 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonMCInstrInfo, HexagonMCCodeEmitter, HexagonGenMCCodeEmitter，说明了它与同级后端组件的连接关系。

## Key Concepts / 关键概念

- register modeling / 寄存器建模
- instruction semantics / 指令语义
- MC-layer target description / MC 层目标描述
- subtarget feature gating / 子目标特性控制

## Dependencies / 依赖关系

- Direct includes / 直接包含: `MCTargetDesc/HexagonMCCodeEmitter.h, MCTargetDesc/HexagonBaseInfo.h, MCTargetDesc/HexagonFixupKinds.h, MCTargetDesc/HexagonMCExpr.h, MCTargetDesc/HexagonMCInstrInfo.h, MCTargetDesc/HexagonMCTargetDesc.h, llvm/ADT/Statistic.h, llvm/MC/MCContext.h, llvm/MC/MCExpr.h, llvm/MC/MCFixup.h, ... (27 total)`
- Hexagon symbols / Hexagon 符号: `HexagonMCCodeEmitter, HexagonBaseInfo, HexagonFixupKinds, HexagonMCExpr, HexagonMCInstrInfo, HexagonMCTargetDesc, HexagonII, HexagonGenMCCodeEmitter`
- Build role / 构建角色: compiled into LLVM's Hexagon backend library and linked with CodeGen/MC infrastructure. / 被编译进 LLVM 的 Hexagon 后端库，并与 CodeGen/MC 基础设施联动。
