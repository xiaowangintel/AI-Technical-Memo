# PseudoProbeRewriter.cpp — Code Analysis / 代码分析

## Source / 来源

- File / 文件: `bolt/lib/Rewrite/PseudoProbeRewriter.cpp`
- Repository / 仓库: `llvm-project`
- Purpose / 用途: bolt/Rewrite/PseudoProbeRewriter.cpp Implement support for pseudo probes.. It also sits in code that implements BOLT rewriting, patching, and binary emission logic. / 该文件实现 BOLT 重写、补丁与二进制输出逻辑。 源码头部说明其职责是：bolt/Rewrite/PseudoProbeRewriter.cpp Implement support for pseudo probes.。

## Line-by-Line Analysis / 逐行分析

### Lines 1-12

```cpp
//===- bolt/Rewrite/PseudoProbeRewriter.cpp -------------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// Implement support for pseudo probes.
//
//===----------------------------------------------------------------------===//
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 13-24

```cpp
#include "bolt/Core/BinaryFunction.h"
#include "bolt/Rewrite/MetadataRewriter.h"
#include "bolt/Rewrite/MetadataRewriters.h"
#include "bolt/Utils/CommandLineOpts.h"
#include "bolt/Utils/Utils.h"
#include "llvm/IR/Function.h"
#include "llvm/MC/MCPseudoProbe.h"
#include "llvm/Support/CommandLine.h"
#include "llvm/Support/Debug.h"
#include "llvm/Support/LEB128.h"
#include <memory>
```

- EN: Pulls in 11 header(s) from local project, LLVM, system dependencies needed by this range. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里引入 11 个头文件，为本段提供本地模块、LLVM 或系统层面的依赖。这里的注释记录了设计假设、不变量或使用说明。

### Lines 25-32

```cpp
#undef DEBUG_TYPE
#define DEBUG_TYPE "pseudo-probe-rewriter"

using namespace llvm;
using namespace bolt;

namespace opts {
```

- EN: Establishes header-guard or prologue macros so the file can be compiled safely. Works inside namespace scope `llvm`, `bolt`, `opts` to organize symbols. Defines macros such as `DEBUG_TYPE` for constants or compile-time switches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里建立头文件保护或前导宏，保证源码可被安全编译与重复包含控制。这里位于命名空间 `llvm`, `bolt`, `opts` 中，用于组织符号作用域。这里定义宏 `DEBUG_TYPE`，用于常量或编译期开关。这里的注释记录了设计假设、不变量或使用说明。

### Lines 33-40

```cpp
enum PrintPseudoProbesOptions {
  PPP_None = 0,
  PPP_Probes_Section_Decode = 0x1,
  PPP_Probes_Address_Conversion = 0x2,
  PPP_Encoded_Probes = 0x3,
  PPP_All = 0xf
};
```

- EN: Defines enumerations such as `PrintPseudoProbesOptions` to encode states or modes. Notable symbols here include `PrintPseudoProbesOptions`.
- CN: 这里定义枚举 `PrintPseudoProbesOptions`，用于表达状态或模式。这里较值得关注的符号包括 `PrintPseudoProbesOptions`。

### Lines 41-52

```cpp
static cl::opt<PrintPseudoProbesOptions> PrintPseudoProbes(
    "print-pseudo-probes", cl::desc("print pseudo probe info"),
    cl::init(PPP_None),
    cl::values(clEnumValN(PPP_Probes_Section_Decode, "decode",
                          "decode probes section from binary"),
               clEnumValN(PPP_Probes_Address_Conversion, "address_conversion",
                          "update address2ProbesMap with output block address"),
               clEnumValN(PPP_Encoded_Probes, "encoded_probes",
                          "display the encoded probes in binary section"),
               clEnumValN(PPP_All, "all", "enable all debugging printout")),
    cl::Hidden, cl::cat(BoltCategory));
```

- EN: Declares or implements routines including `desc`, `init`, `clEnumValN`, `cat`. Notable symbols here include `desc`, `init`, `clEnumValN`, `cat`.
- CN: 这里声明或实现函数，例如 `desc`, `init`, `clEnumValN`, `cat`。这里较值得关注的符号包括 `desc`, `init`, `clEnumValN`, `cat`。

### Lines 53-63

```cpp
extern cl::opt<bool> ProfileWritePseudoProbes;
extern cl::opt<bool> StaleMatchingWithPseudoProbes;
} // namespace opts

namespace {
class PseudoProbeRewriter final : public MetadataRewriter {
  /// .pseudo_probe_desc section.
  /// Contains information about pseudo probe description, like its related
  /// function
  ErrorOr<BinarySection &> PseudoProbeDescSection{std::errc::bad_address};
```

- EN: Works inside namespace scope `opts` to organize symbols. Introduces type definitions such as `PseudoProbeRewriter`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `PseudoProbeRewriter`, `opts`.
- CN: 这里位于命名空间 `opts` 中，用于组织符号作用域。这里引入类型定义，例如 `PseudoProbeRewriter`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `PseudoProbeRewriter`, `opts`。

### Lines 64-73

```cpp
  /// .pseudo_probe section.
  /// Contains information about pseudo probe details, like its address
  ErrorOr<BinarySection &> PseudoProbeSection{std::errc::bad_address};

  /// Update address of MCDecodedPseudoProbe.
  void updatePseudoProbes();

  /// Encode MCDecodedPseudoProbe.
  void encodePseudoProbes();
```

- EN: Declares or implements routines including `updatePseudoProbes`, `encodePseudoProbes`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `updatePseudoProbes`, `encodePseudoProbes`.
- CN: 这里声明或实现函数，例如 `updatePseudoProbes`, `encodePseudoProbes`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `updatePseudoProbes`, `encodePseudoProbes`。

### Lines 74-81

```cpp
  /// Parse .pseudo_probe_desc section and .pseudo_probe section
  /// Setup Pseudo probe decoder
  /// If \p ProfiledOnly is set, only parse records for functions with profile.
  void parsePseudoProbe(bool ProfiledOnly = false);

  /// PseudoProbe decoder
  std::shared_ptr<MCPseudoProbeDecoder> ProbeDecoderPtr;
```

- EN: Declares or implements routines including `parsePseudoProbe`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `parsePseudoProbe`.
- CN: 这里声明或实现函数，例如 `parsePseudoProbe`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `parsePseudoProbe`。

### Lines 82-91

```cpp
public:
  PseudoProbeRewriter(BinaryContext &BC)
      : MetadataRewriter("pseudo-probe-rewriter", BC),
        ProbeDecoderPtr(std::make_shared<MCPseudoProbeDecoder>()) {
    BC.setPseudoProbeDecoder(ProbeDecoderPtr);
  }

  Error preCFGInitializer() override;
  Error postEmitFinalizer() override;
```

- EN: Declares or implements routines including `PseudoProbeRewriter`, `MetadataRewriter`, `ProbeDecoderPtr`, `preCFGInitializer`, `postEmitFinalizer`. Notable symbols here include `PseudoProbeRewriter`, `MetadataRewriter`, `ProbeDecoderPtr`, `preCFGInitializer`, `postEmitFinalizer`.
- CN: 这里声明或实现函数，例如 `PseudoProbeRewriter`, `MetadataRewriter`, `ProbeDecoderPtr`, `preCFGInitializer`, `postEmitFinalizer`。这里较值得关注的符号包括 `PseudoProbeRewriter`, `MetadataRewriter`, `ProbeDecoderPtr`, `preCFGInitializer`, `postEmitFinalizer`。

### Lines 92-101

```cpp
  ~PseudoProbeRewriter() override { ProbeDecoderPtr.reset(); }
};

Error PseudoProbeRewriter::preCFGInitializer() {
  if (opts::ProfileWritePseudoProbes || opts::StaleMatchingWithPseudoProbes)
    parsePseudoProbe(opts::ProfileWritePseudoProbes);

  return Error::success();
}
```

- EN: Declares or implements routines including `PseudoProbeRewriter`, `preCFGInitializer`, `parsePseudoProbe`. Notable symbols here include `PseudoProbeRewriter`, `preCFGInitializer`, `parsePseudoProbe`.
- CN: 这里声明或实现函数，例如 `PseudoProbeRewriter`, `preCFGInitializer`, `parsePseudoProbe`。这里较值得关注的符号包括 `PseudoProbeRewriter`, `preCFGInitializer`, `parsePseudoProbe`。

### Lines 102-109

```cpp
Error PseudoProbeRewriter::postEmitFinalizer() {
  if (!opts::StaleMatchingWithPseudoProbes)
    parsePseudoProbe();
  updatePseudoProbes();

  return Error::success();
}
```

- EN: Declares or implements routines including `postEmitFinalizer`, `parsePseudoProbe`, `updatePseudoProbes`. Notable symbols here include `postEmitFinalizer`, `parsePseudoProbe`, `updatePseudoProbes`.
- CN: 这里声明或实现函数，例如 `postEmitFinalizer`, `parsePseudoProbe`, `updatePseudoProbes`。这里较值得关注的符号包括 `postEmitFinalizer`, `parsePseudoProbe`, `updatePseudoProbes`。

### Lines 110-119

```cpp
void PseudoProbeRewriter::parsePseudoProbe(bool ProfiledOnly) {
  MCPseudoProbeDecoder &ProbeDecoder(*ProbeDecoderPtr);
  PseudoProbeDescSection = BC.getUniqueSectionByName(".pseudo_probe_desc");
  PseudoProbeSection = BC.getUniqueSectionByName(".pseudo_probe");

  if (!PseudoProbeDescSection && !PseudoProbeSection) {
    // pesudo probe is not added to binary. It is normal and no warning needed.
    return;
  }
```

- EN: Declares or implements routines including `parsePseudoProbe`, `ProbeDecoder`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `parsePseudoProbe`, `ProbeDecoder`.
- CN: 这里声明或实现函数，例如 `parsePseudoProbe`, `ProbeDecoder`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `parsePseudoProbe`, `ProbeDecoder`。

### Lines 120-128

```cpp
  // If only one section is found, it might mean the ELF is corrupted.
  if (!PseudoProbeDescSection) {
    errs() << "BOLT-WARNING: fail in reading .pseudo_probe_desc binary\n";
    return;
  } else if (!PseudoProbeSection) {
    errs() << "BOLT-WARNING: fail in reading .pseudo_probe binary\n";
    return;
  }
```

- EN: Declares or implements routines including `errs`, `if`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `errs`, `if`.
- CN: 这里声明或实现函数，例如 `errs`, `if`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `errs`, `if`。

### Lines 129-136

```cpp
  StringRef Contents = PseudoProbeDescSection->getContents();
  if (!ProbeDecoder.buildGUID2FuncDescMap(
          reinterpret_cast<const uint8_t *>(Contents.data()), Contents.size(),
          /*IsMMapped*/ true)) {
    errs() << "BOLT-WARNING: fail in building GUID2FuncDescMap\n";
    return;
  }
```

- EN: Declares or implements routines including `getContents`, `errs`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `getContents`, `errs`.
- CN: 这里声明或实现函数，例如 `getContents`, `errs`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `getContents`, `errs`。

### Lines 137-154

```cpp
  MCPseudoProbeDecoder::Uint64Set GuidFilter;
  MCPseudoProbeDecoder::Uint64Map FuncStartAddrs;
  SmallVector<StringRef, 0> Suffixes(
      {".destroy", ".resume", ".llvm.", ".cold", ".warm"});
  for (const BinaryFunction *F : BC.getAllBinaryFunctions()) {
    bool HasProfile = F->hasProfileAvailable();
    for (const MCSymbol *Sym : F->getSymbols()) {
      StringRef SymName = Sym->getName();
      for (auto Name : {std::optional(NameResolver::restore(SymName)),
                        getCommonName(SymName, false, Suffixes)}) {
        if (!Name)
          continue;
        SymName = *Name;
        uint64_t GUID = Function::getGUIDAssumingExternalLinkage(SymName);
        FuncStartAddrs[GUID] = F->getAddress();
        if (ProfiledOnly && HasProfile)
          GuidFilter.insert(GUID);
      }
```

- EN: Declares or implements routines including `hasProfileAvailable`, `getName`, `getCommonName`, `getGUIDAssumingExternalLinkage`, `getAddress`. Notable symbols here include `hasProfileAvailable`, `getName`, `getCommonName`, `getGUIDAssumingExternalLinkage`, `getAddress`.
- CN: 这里声明或实现函数，例如 `hasProfileAvailable`, `getName`, `getCommonName`, `getGUIDAssumingExternalLinkage`, `getAddress`。这里较值得关注的符号包括 `hasProfileAvailable`, `getName`, `getCommonName`, `getGUIDAssumingExternalLinkage`, `getAddress`。

### Lines 155-164

```cpp
    }
  }
  Contents = PseudoProbeSection->getContents();
  if (!ProbeDecoder.buildAddress2ProbeMap(
          reinterpret_cast<const uint8_t *>(Contents.data()), Contents.size(),
          GuidFilter, FuncStartAddrs)) {
    errs() << "BOLT-WARNING: fail in building Address2ProbeMap\n";
    return;
  }
```

- EN: Declares or implements routines including `getContents`, `errs`. Notable symbols here include `getContents`, `errs`.
- CN: 这里声明或实现函数，例如 `getContents`, `errs`。这里较值得关注的符号包括 `getContents`, `errs`。

### Lines 165-172

```cpp
  if (opts::PrintPseudoProbes == opts::PrintPseudoProbesOptions::PPP_All ||
      opts::PrintPseudoProbes ==
          opts::PrintPseudoProbesOptions::PPP_Probes_Section_Decode) {
    outs() << "Report of decoding input pseudo probe binaries \n";
    ProbeDecoder.printGUID2FuncDescMap(outs());
    ProbeDecoder.printProbesForAllAddresses(outs());
  }
```

- EN: Declares or implements routines including `outs`. Notable symbols here include `outs`.
- CN: 这里声明或实现函数，例如 `outs`。这里较值得关注的符号包括 `outs`。

### Lines 173-190

```cpp
  const GUIDProbeFunctionMap &GUID2Func = ProbeDecoder.getGUID2FuncDescMap();
  // Checks GUID in GUID2Func and returns it if it's present or null otherwise.
  auto checkGUID = [&](StringRef SymName) -> uint64_t {
    uint64_t GUID = Function::getGUIDAssumingExternalLinkage(SymName);
    if (GUID2Func.find(GUID) == GUID2Func.end())
      return 0;
    return GUID;
  };
  for (BinaryFunction *F : BC.getAllBinaryFunctions()) {
    for (const MCSymbol *Sym : F->getSymbols()) {
      StringRef SymName = NameResolver::restore(Sym->getName());
      uint64_t GUID = checkGUID(SymName);
      std::optional<StringRef> CommonName =
          getCommonName(SymName, false, Suffixes);
      if (!GUID && CommonName)
        GUID = checkGUID(*CommonName);
      if (GUID)
        F->setGUID(GUID);
```

- EN: Declares or implements routines including `getGUIDAssumingExternalLinkage`, `restore`, `checkGUID`, `getCommonName`, `setGUID`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `getGUIDAssumingExternalLinkage`, `restore`, `checkGUID`, `getCommonName`, `setGUID`.
- CN: 这里声明或实现函数，例如 `getGUIDAssumingExternalLinkage`, `restore`, `checkGUID`, `getCommonName`, `setGUID`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `getGUIDAssumingExternalLinkage`, `restore`, `checkGUID`, `getCommonName`, `setGUID`。

### Lines 191-203

```cpp
    }
  }
}

void PseudoProbeRewriter::updatePseudoProbes() {
  MCPseudoProbeDecoder &ProbeDecoder(*ProbeDecoderPtr);
  // check if there is pseudo probe section decoded
  if (ProbeDecoder.getAddress2ProbesMap().empty())
    return;
  // input address converted to output
  AddressProbesMap &Address2ProbesMap = ProbeDecoder.getAddress2ProbesMap();
  const GUIDProbeFunctionMap &GUID2Func = ProbeDecoder.getGUID2FuncDescMap();
```

- EN: Declares or implements routines including `updatePseudoProbes`, `ProbeDecoder`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `updatePseudoProbes`, `ProbeDecoder`.
- CN: 这里声明或实现函数，例如 `updatePseudoProbes`, `ProbeDecoder`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `updatePseudoProbes`, `ProbeDecoder`。

### Lines 204-217

```cpp
  for (MCDecodedPseudoProbe &Probe : Address2ProbesMap) {
    uint64_t Address = Probe.getAddress();
    BinaryFunction *F = BC.getBinaryFunctionContainingAddress(Address);
    // If F is removed, eliminate all probes inside it from inline tree
    // Setting probes' addresses as INT64_MAX means elimination
    if (!F) {
      Probe.setAddress(INT64_MAX);
      continue;
    }
    // If F is not emitted, the function will remain in the same address as its
    // input
    if (!F->isEmitted())
      continue;
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 218-228

```cpp
    uint64_t Offset = Address - F->getAddress();
    const BinaryBasicBlock *BB = F->getBasicBlockContainingOffset(Offset);
    uint64_t BlkOutputAddress = BB->getOutputAddressRange().first;
    // Check if block output address is defined.
    // If not, such block is removed from binary. Then remove the probes from
    // inline tree
    if (BlkOutputAddress == 0) {
      Probe.setAddress(INT64_MAX);
      continue;
    }
```

- EN: Declares or implements routines including `getAddress`, `getBasicBlockContainingOffset`, `getOutputAddressRange`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `getAddress`, `getBasicBlockContainingOffset`, `getOutputAddressRange`.
- CN: 这里声明或实现函数，例如 `getAddress`, `getBasicBlockContainingOffset`, `getOutputAddressRange`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `getAddress`, `getBasicBlockContainingOffset`, `getOutputAddressRange`。

### Lines 229-243

```cpp
    if (Probe.isBlock()) {
      Probe.setAddress(BlkOutputAddress);
    } else if (Probe.isCall()) {
      // A call probe may be duplicated due to ICP
      // Go through output of InputOffsetToAddressMap to collect all related
      // probes
      auto CallOutputAddresses = BC.getIOAddressMap().lookupAll(Address);
      auto CallOutputAddress = CallOutputAddresses.first;
      if (CallOutputAddress == CallOutputAddresses.second) {
        Probe.setAddress(INT64_MAX);
      } else {
        Probe.setAddress(CallOutputAddress->second);
        CallOutputAddress = std::next(CallOutputAddress);
      }
```

- EN: Declares or implements routines including `if`, `next`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `if`, `next`.
- CN: 这里声明或实现函数，例如 `if`, `next`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `if`, `next`。

### Lines 244-261

```cpp
      while (CallOutputAddress != CallOutputAddresses.second) {
        ProbeDecoder.addInjectedProbe(Probe, CallOutputAddress->second);
        CallOutputAddress = std::next(CallOutputAddress);
      }
    }
  }

  if (opts::PrintPseudoProbes == opts::PrintPseudoProbesOptions::PPP_All ||
      opts::PrintPseudoProbes ==
          opts::PrintPseudoProbesOptions::PPP_Probes_Address_Conversion) {
    outs() << "Pseudo Probe Address Conversion results:\n";
    // table that correlates address to block
    std::unordered_map<uint64_t, StringRef> Addr2BlockNames;
    for (auto &F : BC.getBinaryFunctions())
      for (BinaryBasicBlock &BinaryBlock : F.second)
        Addr2BlockNames[BinaryBlock.getOutputAddressRange().first] =
            BinaryBlock.getName();
```

- EN: Declares or implements routines including `next`, `outs`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `next`, `outs`.
- CN: 这里声明或实现函数，例如 `next`, `outs`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `next`, `outs`。

### Lines 262-276

```cpp
    // scan all addresses -> correlate probe to block when print out
    for (MCDecodedPseudoProbe &Probe : Address2ProbesMap) {
      if (Probe.getAddress() == INT64_MAX)
        outs() << "Deleted Probe: ";
      else
        outs() << "Address: " << format_hex(Probe.getAddress(), 8) << " ";
      Probe.print(outs(), GUID2Func, true);
      // print block name only if the probe is block type and undeleted.
      if (Probe.isBlock() && Probe.getAddress() != INT64_MAX)
        outs() << format_hex(Probe.getAddress(), 8) << " Probe is in "
               << Addr2BlockNames[Probe.getAddress()] << "\n";
    }
    outs() << "=======================================\n";
  }
```

- EN: Declares or implements routines including `outs`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `outs`.
- CN: 这里声明或实现函数，例如 `outs`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `outs`。

### Lines 277-286

```cpp
  // encode pseudo probes with updated addresses
  encodePseudoProbes();
}

void PseudoProbeRewriter::encodePseudoProbes() {
  MCPseudoProbeDecoder &ProbeDecoder(*ProbeDecoderPtr);
  // Buffer for new pseudo probes section
  SmallString<8> Contents;
  MCDecodedPseudoProbe *LastProbe = nullptr;
```

- EN: Declares or implements routines including `encodePseudoProbes`, `ProbeDecoder`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `encodePseudoProbes`, `ProbeDecoder`.
- CN: 这里声明或实现函数，例如 `encodePseudoProbes`, `ProbeDecoder`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `encodePseudoProbes`, `ProbeDecoder`。

### Lines 287-296

```cpp
  auto EmitInt = [&](uint64_t Value, uint32_t Size) {
    const bool IsLittleEndian = BC.AsmInfo->isLittleEndian();
    uint64_t Swapped = support::endian::byte_swap(
        Value,
        IsLittleEndian ? llvm::endianness::little : llvm::endianness::big);
    unsigned Index = IsLittleEndian ? 0 : 8 - Size;
    auto Entry = StringRef(reinterpret_cast<char *>(&Swapped) + Index, Size);
    Contents.append(Entry.begin(), Entry.end());
  };
```

- EN: Declares or implements routines including `isLittleEndian`, `StringRef`. Notable symbols here include `isLittleEndian`, `StringRef`.
- CN: 这里声明或实现函数，例如 `isLittleEndian`, `StringRef`。这里较值得关注的符号包括 `isLittleEndian`, `StringRef`。

### Lines 297-310

```cpp
  auto EmitULEB128IntValue = [&](uint64_t Value) {
    SmallString<128> Tmp;
    raw_svector_ostream OSE(Tmp);
    encodeULEB128(Value, OSE, 0);
    Contents.append(OSE.str().begin(), OSE.str().end());
  };

  auto EmitSLEB128IntValue = [&](int64_t Value) {
    SmallString<128> Tmp;
    raw_svector_ostream OSE(Tmp);
    encodeSLEB128(Value, OSE);
    Contents.append(OSE.str().begin(), OSE.str().end());
  };
```

- EN: Declares or implements routines including `OSE`, `encodeULEB128`, `encodeSLEB128`. Notable symbols here include `OSE`, `encodeULEB128`, `encodeSLEB128`.
- CN: 这里声明或实现函数，例如 `OSE`, `encodeULEB128`, `encodeSLEB128`。这里较值得关注的符号包括 `OSE`, `encodeULEB128`, `encodeSLEB128`。

### Lines 311-328

```cpp
  // Emit individual pseudo probes in a inline tree node
  // Probe index, type, attribute, address type and address are encoded
  // Address of the first probe is absolute.
  // Other probes' address are represented by delta
  auto EmitDecodedPseudoProbe = [&](MCDecodedPseudoProbe *&CurProbe) {
    assert(!isSentinelProbe(CurProbe->getAttributes()) &&
           "Sentinel probes should not be emitted");
    EmitULEB128IntValue(CurProbe->getIndex());
    uint8_t PackedType = CurProbe->getType() | (CurProbe->getAttributes() << 4);
    uint8_t Flag =
        LastProbe ? ((int8_t)MCPseudoProbeFlag::AddressDelta << 7) : 0;
    EmitInt(Flag | PackedType, 1);
    if (LastProbe) {
      // Emit the delta between the address label and LastProbe.
      int64_t Delta = CurProbe->getAddress() - LastProbe->getAddress();
      EmitSLEB128IntValue(Delta);
    } else {
      // Emit absolute address for encoding the first pseudo probe.
```

- EN: Declares or implements routines including `assert`, `EmitULEB128IntValue`, `getType`, `EmitInt`, `getAddress`, and 1 more. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `assert`, `EmitULEB128IntValue`, `getType`, `EmitInt`, `getAddress`, `EmitSLEB128IntValue`.
- CN: 这里声明或实现函数，例如 `assert`, `EmitULEB128IntValue`, `getType`, `EmitInt`, `getAddress`, and 1 more。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `assert`, `EmitULEB128IntValue`, `getType`, `EmitInt`, `getAddress`, `EmitSLEB128IntValue`。

### Lines 329-337

```cpp
      uint32_t AddrSize = BC.AsmInfo->getCodePointerSize();
      EmitInt(CurProbe->getAddress(), AddrSize);
    }
  };

  std::map<InlineSite, MCDecodedPseudoProbeInlineTree *,
           std::greater<InlineSite>>
      Inlinees;
```

- EN: Declares or implements routines including `getCodePointerSize`, `EmitInt`. Notable symbols here include `getCodePointerSize`, `EmitInt`.
- CN: 这里声明或实现函数，例如 `getCodePointerSize`, `EmitInt`。这里较值得关注的符号包括 `getCodePointerSize`, `EmitInt`。

### Lines 338-349

```cpp
  // DFS of inline tree to emit pseudo probes in all tree node
  // Inline site index of a probe is emitted first.
  // Then tree node Guid, size of pseudo probes and children nodes, and detail
  // of contained probes are emitted Deleted probes are skipped Root node is not
  // encoded to binaries. It's a "wrapper" of inline trees of each function.
  std::list<std::pair<uint64_t, MCDecodedPseudoProbeInlineTree *>> NextNodes;
  const MCDecodedPseudoProbeInlineTree &Root =
      ProbeDecoder.getDummyInlineRoot();
  for (auto Child = Root.getChildren().begin();
       Child != Root.getChildren().end(); ++Child)
    Inlinees[Child->getInlineSite()] = &*Child;
```

- EN: Declares or implements routines including `getInlineSite`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `getInlineSite`.
- CN: 这里声明或实现函数，例如 `getInlineSite`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `getInlineSite`。

### Lines 350-360

```cpp
  for (auto Inlinee : Inlinees)
    // INT64_MAX is "placeholder" of unused callsite index field in the pair
    NextNodes.push_back({INT64_MAX, Inlinee.second});

  Inlinees.clear();

  while (!NextNodes.empty()) {
    uint64_t ProbeIndex = NextNodes.back().first;
    MCDecodedPseudoProbeInlineTree *Cur = NextNodes.back().second;
    NextNodes.pop_back();
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 361-378

```cpp
    if (Cur->Parent && !Cur->Parent->isRoot())
      // Emit probe inline site
      EmitULEB128IntValue(ProbeIndex);

    // Emit probes grouped by GUID.
    LLVM_DEBUG({
      dbgs().indent(MCPseudoProbeTable::DdgPrintIndent);
      dbgs() << "GUID: " << Cur->Guid << "\n";
    });
    // Emit Guid
    EmitInt(Cur->Guid, 8);
    // Emit number of probes in this node
    uint64_t Deleted = 0;
    for (MCDecodedPseudoProbe *&Probe :
         llvm::make_pointer_range(Cur->getProbes()))
      if (Probe->getAddress() == INT64_MAX)
        Deleted++;
    LLVM_DEBUG(dbgs() << "Deleted Probes:" << Deleted << "\n");
```

- EN: Declares or implements routines including `EmitULEB128IntValue`, `dbgs`, `EmitInt`, `make_pointer_range`, `LLVM_DEBUG`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `EmitULEB128IntValue`, `dbgs`, `EmitInt`, `make_pointer_range`, `LLVM_DEBUG`.
- CN: 这里声明或实现函数，例如 `EmitULEB128IntValue`, `dbgs`, `EmitInt`, `make_pointer_range`, `LLVM_DEBUG`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `EmitULEB128IntValue`, `dbgs`, `EmitInt`, `make_pointer_range`, `LLVM_DEBUG`。

### Lines 379-396

```cpp
    size_t InjectedProbes = ProbeDecoder.getNumInjectedProbes(Cur);
    uint64_t ProbesSize = Cur->getProbes().size() - Deleted + InjectedProbes;
    EmitULEB128IntValue(ProbesSize);
    // Emit number of direct inlinees
    EmitULEB128IntValue(Cur->getChildren().size());
    // Emit probes in this group
    for (MCDecodedPseudoProbe *&Probe :
         llvm::make_pointer_range(Cur->getProbes())) {
      if (Probe->getAddress() == INT64_MAX)
        continue;
      EmitDecodedPseudoProbe(Probe);
      LastProbe = Probe;
    }
    if (InjectedProbes) {
      for (MCDecodedPseudoProbe *&Probe :
           llvm::make_pointer_range(ProbeDecoder.getInjectedProbes(Cur))) {
        if (Probe->getAddress() == INT64_MAX)
          continue;
```

- EN: Declares or implements routines including `getProbes`, `EmitULEB128IntValue`, `make_pointer_range`, `EmitDecodedPseudoProbe`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `getProbes`, `EmitULEB128IntValue`, `make_pointer_range`, `EmitDecodedPseudoProbe`.
- CN: 这里声明或实现函数，例如 `getProbes`, `EmitULEB128IntValue`, `make_pointer_range`, `EmitDecodedPseudoProbe`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `getProbes`, `EmitULEB128IntValue`, `make_pointer_range`, `EmitDecodedPseudoProbe`。

### Lines 397-414

```cpp
        EmitDecodedPseudoProbe(Probe);
        LastProbe = Probe;
      }
    }

    for (auto Child = Cur->getChildren().begin();
         Child != Cur->getChildren().end(); ++Child)
      Inlinees[Child->getInlineSite()] = &*Child;
    for (const auto &Inlinee : Inlinees) {
      assert(Cur->Guid != 0 && "non root tree node must have nonzero Guid");
      NextNodes.push_back({std::get<1>(Inlinee.first), Inlinee.second});
      LLVM_DEBUG({
        dbgs().indent(MCPseudoProbeTable::DdgPrintIndent);
        dbgs() << "InlineSite: " << std::get<1>(Inlinee.first) << "\n";
      });
    }
    Inlinees.clear();
  }
```

- EN: Declares or implements routines including `EmitDecodedPseudoProbe`, `getChildren`, `getInlineSite`, `assert`, `dbgs`. Notable symbols here include `EmitDecodedPseudoProbe`, `getChildren`, `getInlineSite`, `assert`, `dbgs`.
- CN: 这里声明或实现函数，例如 `EmitDecodedPseudoProbe`, `getChildren`, `getInlineSite`, `assert`, `dbgs`。这里较值得关注的符号包括 `EmitDecodedPseudoProbe`, `getChildren`, `getInlineSite`, `assert`, `dbgs`。

### Lines 415-432

```cpp

  // Create buffer for new contents for the section
  // Freed when parent section is destroyed
  uint8_t *Output = new uint8_t[Contents.str().size()];
  memcpy(Output, Contents.str().data(), Contents.str().size());
  BC.registerOrUpdateSection(".pseudo_probe", PseudoProbeSection->getELFType(),
                             PseudoProbeSection->getELFFlags(), Output,
                             Contents.str().size(), 1);
  if (opts::PrintPseudoProbes == opts::PrintPseudoProbesOptions::PPP_All ||
      opts::PrintPseudoProbes ==
          opts::PrintPseudoProbesOptions::PPP_Encoded_Probes) {
    // create a dummy decoder;
    MCPseudoProbeDecoder DummyDecoder;
    StringRef DescContents = PseudoProbeDescSection->getContents();
    DummyDecoder.buildGUID2FuncDescMap(
        reinterpret_cast<const uint8_t *>(DescContents.data()),
        DescContents.size());
    StringRef ProbeContents = PseudoProbeSection->getOutputContents();
```

- EN: Declares or implements routines including `memcpy`, `getELFFlags`, `getContents`, `getOutputContents`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `memcpy`, `getELFFlags`, `getContents`, `getOutputContents`.
- CN: 这里声明或实现函数，例如 `memcpy`, `getELFFlags`, `getContents`, `getOutputContents`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `memcpy`, `getELFFlags`, `getContents`, `getOutputContents`。

### Lines 433-448

```cpp
    MCPseudoProbeDecoder::Uint64Set GuidFilter;
    MCPseudoProbeDecoder::Uint64Map FuncStartAddrs;
    for (const BinaryFunction *F : BC.getAllBinaryFunctions()) {
      const uint64_t Addr =
          F->isEmitted() ? F->getOutputAddress() : F->getAddress();
      FuncStartAddrs[Function::getGUIDAssumingExternalLinkage(
          NameResolver::restore(F->getOneName()))] = Addr;
    }
    DummyDecoder.buildAddress2ProbeMap(
        reinterpret_cast<const uint8_t *>(ProbeContents.data()),
        ProbeContents.size(), GuidFilter, FuncStartAddrs);
    DummyDecoder.printProbesForAllAddresses(outs());
  }
}
} // namespace
```

- EN: Declares or implements routines including `isEmitted`, `restore`. Notable symbols here include `isEmitted`, `restore`.
- CN: 这里声明或实现函数，例如 `isEmitted`, `restore`。这里较值得关注的符号包括 `isEmitted`, `restore`。

### Lines 449-452

```cpp
std::unique_ptr<MetadataRewriter>
llvm::bolt::createPseudoProbeRewriter(BinaryContext &BC) {
  return std::make_unique<PseudoProbeRewriter>(BC);
}
```

- EN: Declares or implements routines including `createPseudoProbeRewriter`. Notable symbols here include `createPseudoProbeRewriter`.
- CN: 这里声明或实现函数，例如 `createPseudoProbeRewriter`。这里较值得关注的符号包括 `createPseudoProbeRewriter`。

## Key Concepts / 关键概念

- `PseudoProbeRewriter`: class or struct interface / 类或结构体接口
- `PrintPseudoProbesOptions`: enumeration of modes or states / 模式或状态枚举
- `desc`: function or method entry point / 函数或方法入口
- `init`: function or method entry point / 函数或方法入口
- `clEnumValN`: function or method entry point / 函数或方法入口
- `cat`: function or method entry point / 函数或方法入口
- `updatePseudoProbes`: function or method entry point / 函数或方法入口
- `DEBUG_TYPE`: macro or compile-time switch / 宏或编译期开关

## Dependencies / 依赖关系

- Local headers / 本地头文件: `bolt/Core/BinaryFunction.h`, `bolt/Rewrite/MetadataRewriter.h`, `bolt/Rewrite/MetadataRewriters.h`, `bolt/Utils/CommandLineOpts.h`, `bolt/Utils/Utils.h`
- LLVM headers / LLVM 头文件: `llvm/IR/Function.h`, `llvm/MC/MCPseudoProbe.h`, `llvm/Support/CommandLine.h`, `llvm/Support/Debug.h`, `llvm/Support/LEB128.h`
- System headers / 系统头文件: `memory`
- Directory context / 目录上下文: `bolt/lib/Rewrite` neighbors usually cooperate with this file to provide the surrounding subsystem / `bolt/lib/Rewrite` 下的相邻文件通常与本文件协作组成对应子系统
