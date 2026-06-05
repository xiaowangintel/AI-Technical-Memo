# NVPTXTargetStreamer.cpp — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/NVPTX/MCTargetDesc/NVPTXTargetStreamer.cpp`
- Repository: `llvm-project`
- Purpose (EN): This file implements the NVPTXTargetStreamer class.
- 目的（中文）: 该文件实现 LLVM 目标后端中的相关逻辑。
- Language: C++ source

## Line-by-Line Analysis / 逐行分析

> Note / 说明: To keep the document readable, the source is analyzed in contiguous line ranges while preserving the original order. Each code block prefixes original line numbers.

### Lines 1-40
```cpp
 1: //=====- NVPTXTargetStreamer.cpp - NVPTXTargetStreamer class ------------=====//
 2: //
 3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
 4: // See https://llvm.org/LICENSE.txt for license information.
 5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
 6: //
 7: //===----------------------------------------------------------------------===//
 8: //
 9: // This file implements the NVPTXTargetStreamer class.
10: //
11: //===----------------------------------------------------------------------===//
12:
13: #include "NVPTXTargetStreamer.h"
14: #include "NVPTXUtilities.h"
15: #include "llvm/MC/MCAsmInfo.h"
16: #include "llvm/MC/MCContext.h"
17: #include "llvm/MC/MCExpr.h"
18: #include "llvm/MC/MCObjectFileInfo.h"
19: #include "llvm/MC/MCSymbol.h"
20: #include "llvm/Support/Casting.h"
21: #include "llvm/Support/FormattedStream.h"
22:
23: using namespace llvm;
24:
25: //
26: // NVPTXTargetStreamer Implemenation
27: //
28: NVPTXTargetStreamer::NVPTXTargetStreamer(MCStreamer &S) : MCTargetStreamer(S) {}
29: NVPTXTargetStreamer::~NVPTXTargetStreamer() = default;
30:
31: void NVPTXTargetStreamer::outputDwarfFileDirectives() {
32:   for (const std::string &S : DwarfFiles)
33:     getStreamer().emitRawText(S);
34:   DwarfFiles.clear();
35: }
36:
37: void NVPTXTargetStreamer::closeLastSection() {
38:   if (HasSections)
39:     getStreamer().emitRawText("\t}");
40: }
```
- EN: This range implements operational logic in helpers such as NVPTXTargetStreamer::NVPTXTargetStreamer, NVPTXTargetStreamer::outputDwarfFileDirectives, getStreamer, clear, translating backend policy into executable code.
- CN: 这一段实现了 NVPTXTargetStreamer::NVPTXTargetStreamer、NVPTXTargetStreamer::outputDwarfFileDirectives、getStreamer、clear 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 41-80
```cpp
41:
42: void NVPTXTargetStreamer::emitDwarfFileDirective(StringRef Directive) {
43:   DwarfFiles.emplace_back(Directive);
44: }
45:
46: static bool isDwarfSection(const MCObjectFileInfo *FI,
47:                            const MCSection *Section) {
48:   // FIXME: the checks for the DWARF sections are very fragile and should be
49:   // fixed up in a followup patch.
50:   if (!Section || Section->isText())
51:     return false;
52:   return Section == FI->getDwarfAbbrevSection() ||
53:          Section == FI->getDwarfInfoSection() ||
54:          Section == FI->getDwarfMacinfoSection() ||
55:          Section == FI->getDwarfFrameSection() ||
56:          Section == FI->getDwarfAddrSection() ||
57:          Section == FI->getDwarfRangesSection() ||
58:          Section == FI->getDwarfARangesSection() ||
59:          Section == FI->getDwarfLocSection() ||
60:          Section == FI->getDwarfStrSection() ||
61:          Section == FI->getDwarfLineSection() ||
62:          Section == FI->getDwarfStrOffSection() ||
63:          Section == FI->getDwarfLineStrSection() ||
64:          Section == FI->getDwarfPubNamesSection() ||
65:          Section == FI->getDwarfPubTypesSection() ||
66:          Section == FI->getDwarfSwiftASTSection() ||
67:          Section == FI->getDwarfTypesDWOSection() ||
68:          Section == FI->getDwarfAbbrevDWOSection() ||
69:          Section == FI->getDwarfAccelObjCSection() ||
70:          Section == FI->getDwarfAccelNamesSection() ||
71:          Section == FI->getDwarfAccelTypesSection() ||
72:          Section == FI->getDwarfAccelNamespaceSection() ||
73:          Section == FI->getDwarfLocDWOSection() ||
74:          Section == FI->getDwarfStrDWOSection() ||
75:          Section == FI->getDwarfCUIndexSection() ||
76:          Section == FI->getDwarfInfoDWOSection() ||
77:          Section == FI->getDwarfLineDWOSection() ||
78:          Section == FI->getDwarfTUIndexSection() ||
79:          Section == FI->getDwarfStrOffDWOSection() ||
80:          Section == FI->getDwarfDebugNamesSection() ||
```
- EN: This range implements operational logic in helpers such as NVPTXTargetStreamer::emitDwarfFileDirective, emplace_back, translating backend policy into executable code.
- CN: 这一段实现了 NVPTXTargetStreamer::emitDwarfFileDirective、emplace_back 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 81-120
```cpp
 81:          Section == FI->getDwarfDebugInlineSection() ||
 82:          Section == FI->getDwarfGnuPubNamesSection() ||
 83:          Section == FI->getDwarfGnuPubTypesSection();
 84: }
 85:
 86: void NVPTXTargetStreamer::changeSection(const MCSection *CurSection,
 87:                                         MCSection *Section, uint32_t SubSection,
 88:                                         raw_ostream &OS) {
 89:   assert(!SubSection && "SubSection is not null!");
 90:   const MCObjectFileInfo *FI = getStreamer().getContext().getObjectFileInfo();
 91:   // Emit closing brace for DWARF sections only.
 92:   if (isDwarfSection(FI, CurSection))
 93:     OS << "\t}\n";
 94:   if (isDwarfSection(FI, Section)) {
 95:     // Emit DWARF .file directives in the outermost scope.
 96:     outputDwarfFileDirectives();
 97:     OS << "\t.section\t" << Section->getName() << '\n';
 98:     // DWARF sections are enclosed into braces - emit the open one.
 99:     OS << "\t{\n";
100:     HasSections = true;
101:   }
102: }
103:
104: void NVPTXTargetStreamer::emitRawBytes(StringRef Data) {
105:   MCTargetStreamer::emitRawBytes(Data);
106:   // TODO: enable this once the bug in the ptxas with the packed bytes is
107:   // resolved. Currently, (it is confirmed by NVidia) it causes a crash in
108:   // ptxas.
109: #if 0
110:   const MCAsmInfo &MAI = Streamer.getContext().getAsmInfo();
111:   const char *Directive = MAI.getData8bitsDirective();
112:   unsigned NumElements = Data.size();
113:   const unsigned MaxLen = 40;
114:   unsigned NumChunks = 1 + ((NumElements - 1) / MaxLen);
115:   // Split the very long directives into several parts if the limit is
116:   // specified.
117:   for (unsigned I = 0; I < NumChunks; ++I) {
118:     SmallString<128> Str;
119:     raw_svector_ostream OS(Str);
120:
```
- EN: This range implements operational logic in helpers such as getDwarfGnuPubTypesSection, assert, getStreamer, outputDwarfFileDirectives, translating backend policy into executable code.
- CN: 这一段实现了 getDwarfGnuPubTypesSection、assert、getStreamer、outputDwarfFileDirectives 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 121-160
```cpp
121:     const char *Label = Directive;
122:     for (auto It = std::next(Data.bytes_begin(), I * MaxLen),
123:               End = (I == NumChunks - 1)
124:                         ? Data.bytes_end()
125:                         : std::next(Data.bytes_begin(), (I + 1) * MaxLen);
126:          It != End; ++It) {
127:       OS << Label << (unsigned)*It;
128:       if (Label == Directive)
129:         Label = ",";
130:     }
131:     Streamer.emitRawText(OS.str());
132:   }
133: #endif
134: }
135:
136: void NVPTXTargetStreamer::emitValue(const MCExpr *Value) {
137:   if (Value->getKind() == MCExpr::SymbolRef) {
138:     const MCSymbolRefExpr &SRE = cast<MCSymbolRefExpr>(*Value);
139:     StringRef SymName = SRE.getSymbol().getName();
140:     if (!SymName.starts_with(".debug")) {
141:       Streamer.emitRawText(NVPTX::getValidPTXIdentifier(SymName));
142:       return;
143:     }
144:     // Fall through to the normal printing.
145:   }
146:   // Otherwise, print the Value normally.
147:   MCTargetStreamer::emitValue(Value);
148: }
149:
150: //
151: // NVPTXAsmTargetStreamer Implementation
152: //
153:
154: NVPTXAsmTargetStreamer::NVPTXAsmTargetStreamer(MCStreamer &S,
155:                                                formatted_raw_ostream &OS)
156:     : NVPTXTargetStreamer(S), OS(OS) {}
157: NVPTXAsmTargetStreamer::~NVPTXAsmTargetStreamer() = default;
158:
159: void NVPTXAsmTargetStreamer::emitBanner() {
160:   OS << "//\n"
```
- EN: This range implements operational logic in helpers such as bytes_end, std::next, emitRawText, NVPTXTargetStreamer::emitValue, translating backend policy into executable code.
- CN: 这一段实现了 bytes_end、std::next、emitRawText、NVPTXTargetStreamer::emitValue 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 161-184
```cpp
161:         "// Generated by LLVM NVPTX Back-End\n"
162:         "//\n"
163:         "\n";
164: }
165:
166: void NVPTXAsmTargetStreamer::emitVersionDirective(unsigned PTXVersion) {
167:   OS << ".version " << (PTXVersion / 10) << "." << (PTXVersion % 10) << "\n";
168: }
169:
170: void NVPTXAsmTargetStreamer::emitTargetDirective(StringRef Target,
171:                                                  bool TexModeIndependent,
172:                                                  bool HasDebug) {
173:   OS << ".target " << Target;
174:   if (TexModeIndependent)
175:     OS << ", texmode_independent";
176:   if (HasDebug)
177:     OS << ", debug";
178:   OS << "\n";
179: }
180:
181: void NVPTXAsmTargetStreamer::emitAddressSizeDirective(unsigned AddrSize) {
182:   OS << ".address_size " << AddrSize << "\n"
183:      << "\n";
184: }
```
- EN: This range implements operational logic in helpers such as NVPTXAsmTargetStreamer::emitVersionDirective, NVPTXAsmTargetStreamer::emitAddressSizeDirective, translating backend policy into executable code.
- CN: 这一段实现了 NVPTXAsmTargetStreamer::emitVersionDirective、NVPTXAsmTargetStreamer::emitAddressSizeDirective 等操作逻辑，把后端策略落实为可执行的代码路径。

## Key Concepts / 关键概念

- EN: NVPTX backend code reflects GPU execution concerns such as address spaces, intrinsics, and PTX-specific lowering.
  - CN: NVPTX 后端代码体现了 GPU 执行特性，例如地址空间、内建函数以及 PTX 特定降级。
- EN: Key symbols in this file include NVPTXTargetStreamer::NVPTXTargetStreamer, NVPTXTargetStreamer::outputDwarfFileDirectives, getStreamer, clear, NVPTXTargetStreamer::closeLastSection, which anchor the file's main abstractions.
  - CN: 该文件中的关键符号包括 NVPTXTargetStreamer::NVPTXTargetStreamer, NVPTXTargetStreamer::outputDwarfFileDirectives, getStreamer, clear, NVPTXTargetStreamer::closeLastSection，它们构成了本文件的核心抽象。

## Dependencies / 依赖关系

- Local backend headers / 本地后端头文件:
  - `NVPTXTargetStreamer.h`
  - `NVPTXUtilities.h`
- LLVM infrastructure / LLVM 基础设施:
  - `llvm/MC/MCAsmInfo.h`
  - `llvm/MC/MCContext.h`
  - `llvm/MC/MCExpr.h`
  - `llvm/MC/MCObjectFileInfo.h`
  - `llvm/MC/MCSymbol.h`
  - `llvm/Support/Casting.h`
  - `llvm/Support/FormattedStream.h`
