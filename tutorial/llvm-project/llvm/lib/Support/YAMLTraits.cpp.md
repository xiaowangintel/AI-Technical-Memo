# YAMLTraits.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `llvm/lib/Support/YAMLTraits.cpp`
- Repository: `llvm-project`
- Purpose (EN): IO
- Purpose (CN): 该文件位于 LLVM 的 `Support` 目录中，主要实现与 `YAMLTraits` 相关的接口、数据结构和辅助逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-60

```cpp
//===- lib/Support/YAMLTraits.cpp -----------------------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#include "llvm/Support/YAMLTraits.h"
#include "llvm/ADT/STLExtras.h"
#include "llvm/ADT/SmallString.h"
#include "llvm/ADT/StringExtras.h"
#include "llvm/ADT/StringRef.h"
#include "llvm/ADT/Twine.h"
#include "llvm/Support/Casting.h"
#include "llvm/Support/Errc.h"
#include "llvm/Support/ErrorHandling.h"
#include "llvm/Support/Format.h"
#include "llvm/Support/LineIterator.h"
#include "llvm/Support/MemoryBuffer.h"
#include "llvm/Support/VersionTuple.h"
#include "llvm/Support/YAMLParser.h"
#include "llvm/Support/raw_ostream.h"
#include <cassert>
#include <cstdint>
#include <cstring>
#include <string>
#include <vector>

using namespace llvm;
using namespace yaml;

//===----------------------------------------------------------------------===//
//  IO
//===----------------------------------------------------------------------===//

IO::IO(void *Context) : Ctxt(Context) {}

IO::~IO() = default;

void *IO::getContext() const {
  return Ctxt;
}

void IO::setContext(void *Context) {
  Ctxt = Context;
}

void IO::setAllowUnknownKeys(bool Allow) {
  llvm_unreachable("Only supported for Input");
}

//===----------------------------------------------------------------------===//
//  Input
//===----------------------------------------------------------------------===//

Input::Input(StringRef InputContent, void *Ctxt,
             SourceMgr::DiagHandlerTy DiagHandler, void *DiagHandlerCtxt)
    : IO(Ctxt), Strm(new Stream(InputContent, SrcMgr, false, &EC)) {
  if (DiagHandler)
```
- EN: Introduces the file banner, comments, and the initial setup for this translation unit.
  CN: 这一段给出文件横幅、注释说明以及该编译单元的初始设置。
- EN: Brings in 20 direct dependencies, including `llvm/Support/YAMLTraits.h`, `llvm/ADT/STLExtras.h`, `llvm/ADT/SmallString.h`, `llvm/ADT/StringExtras.h`.
  CN: 引入了 20 个直接依赖，其中包括 `llvm/Support/YAMLTraits.h`, `llvm/ADT/STLExtras.h`, `llvm/ADT/SmallString.h`, `llvm/ADT/StringExtras.h`。
- EN: This section centers on `IO`, `setContext`, `setAllowUnknownKeys` and implements the local control flow and bookkeeping.
  CN: 这一段主要围绕 `IO`, `setContext`, `setAllowUnknownKeys` 等符号展开，负责实现局部控制流程与状态维护。

### Lines 61-120

```cpp
    SrcMgr.setDiagHandler(DiagHandler, DiagHandlerCtxt);
  DocIterator = Strm->begin();
}

Input::Input(MemoryBufferRef Input, void *Ctxt,
             SourceMgr::DiagHandlerTy DiagHandler, void *DiagHandlerCtxt)
    : IO(Ctxt), Strm(new Stream(Input, SrcMgr, false, &EC)) {
  if (DiagHandler)
    SrcMgr.setDiagHandler(DiagHandler, DiagHandlerCtxt);
  DocIterator = Strm->begin();
}

Input::~Input() = default;

std::error_code Input::error() { return EC; }

bool Input::outputting() const {
  return false;
}

bool Input::setCurrentDocument() {
  if (DocIterator != Strm->end()) {
    Node *N = DocIterator->getRoot();
    if (!N) {
      EC = make_error_code(errc::invalid_argument);
      return false;
    }

    if (isa<NullNode>(N)) {
      // Empty files are allowed and ignored
      ++DocIterator;
      return setCurrentDocument();
    }
    releaseHNodeBuffers();
    TopNode = createHNodes(N);
    CurrentNode = TopNode;
    return true;
  }
  return false;
}

bool Input::nextDocument() {
  return ++DocIterator != Strm->end();
}

const Node *Input::getCurrentNode() const {
  return CurrentNode ? CurrentNode->_node : nullptr;
}

bool Input::mapTag(StringRef Tag, bool Default) {
  // CurrentNode can be null if setCurrentDocument() was unable to
  // parse the document because it was invalid or empty.
  if (!CurrentNode)
    return false;

  std::string foundTag = CurrentNode->_node->getVerbatimTag();
  if (foundTag.empty()) {
    // If no tag found and 'Tag' is the default, say it was found.
    return Default;
  }
```
- EN: This section centers on `Input`, `error`, `outputting` and implements the local control flow and bookkeeping.
  CN: 这一段主要围绕 `Input`, `error`, `outputting` 等符号展开，负责实现局部控制流程与状态维护。
- EN: In this range, the code checks conditions and handles edge cases and returns the resulting value to its callers.
  CN: 在这一段中，代码检查条件并处理边界情况，并将结果返回给调用方。

### Lines 121-180

```cpp
  // Return true iff found tag matches supplied tag.
  return Tag == foundTag;
}

void Input::beginMapping() {
  if (EC)
    return;
  // CurrentNode can be null if the document is empty.
  MapHNode *MN = dyn_cast_or_null<MapHNode>(CurrentNode);
  if (MN) {
    MN->ValidKeys.clear();
  }
}

std::vector<StringRef> Input::keys() {
  MapHNode *MN = dyn_cast<MapHNode>(CurrentNode);
  std::vector<StringRef> Ret;
  if (!MN) {
    setError(CurrentNode, "not a mapping");
    return Ret;
  }
  for (auto &P : MN->Mapping)
    Ret.push_back(P.first());
  return Ret;
}

bool Input::preflightKey(StringRef Key, bool Required, bool, bool &UseDefault,
                         void *&SaveInfo) {
  UseDefault = false;
  if (EC)
    return false;

  // CurrentNode is null for empty documents, which is an error in case required
  // nodes are present.
  if (!CurrentNode) {
    if (Required)
      EC = make_error_code(errc::invalid_argument);
    else
      UseDefault = true;
    return false;
  }

  MapHNode *MN = dyn_cast<MapHNode>(CurrentNode);
  if (!MN) {
    if (Required || !isa<EmptyHNode>(CurrentNode))
      setError(CurrentNode, "not a mapping");
    else
      UseDefault = true;
    return false;
  }
  MN->ValidKeys.push_back(Key.str());
  HNode *Value = MN->Mapping[Key].first;
  if (!Value) {
    if (Required)
      setError(CurrentNode, Twine("missing required key '") + Key + "'");
    else
      UseDefault = true;
    return false;
  }
  SaveInfo = CurrentNode;
```
- EN: This section centers on `beginMapping`, `keys`, `setError` and implements the local control flow and bookkeeping.
  CN: 这一段主要围绕 `beginMapping`, `keys`, `setError` 等符号展开，负责实现局部控制流程与状态维护。
- EN: In this range, the code iterates over collections, ranges, or records and checks conditions and handles edge cases.
  CN: 在这一段中，代码遍历集合、区间或记录，并检查条件并处理边界情况。

### Lines 181-240

```cpp
  CurrentNode = Value;
  return true;
}

void Input::postflightKey(void *saveInfo) {
  CurrentNode = reinterpret_cast<HNode *>(saveInfo);
}

void Input::endMapping() {
  if (EC)
    return;
  // CurrentNode can be null if the document is empty.
  MapHNode *MN = dyn_cast_or_null<MapHNode>(CurrentNode);
  if (!MN)
    return;
  for (const auto &NN : MN->Mapping) {
    if (!is_contained(MN->ValidKeys, NN.first())) {
      const SMRange &ReportLoc = NN.second.second;
      if (!AllowUnknownKeys) {
        setError(ReportLoc, Twine("unknown key '") + NN.first() + "'");
        break;
      } else
        reportWarning(ReportLoc, Twine("unknown key '") + NN.first() + "'");
    }
  }
}

void Input::beginFlowMapping() { beginMapping(); }

void Input::endFlowMapping() { endMapping(); }

unsigned Input::beginSequence() {
  if (SequenceHNode *SQ = dyn_cast<SequenceHNode>(CurrentNode))
    return SQ->Entries.size();
  if (isa<EmptyHNode>(CurrentNode))
    return 0;
  // Treat case where there's a scalar "null" value as an empty sequence.
  if (ScalarHNode *SN = dyn_cast<ScalarHNode>(CurrentNode)) {
    if (isNull(SN->value()))
      return 0;
  }
  // Any other type of HNode is an error.
  setError(CurrentNode, "not a sequence");
  return 0;
}

void Input::endSequence() {
}

bool Input::preflightElement(unsigned Index, void *&SaveInfo) {
  if (EC)
    return false;
  if (SequenceHNode *SQ = dyn_cast<SequenceHNode>(CurrentNode)) {
    SaveInfo = CurrentNode;
    CurrentNode = SQ->Entries[Index];
    return true;
  }
  return false;
}

```
- EN: This section centers on `postflightKey`, `endMapping`, `setError` and implements the local control flow and bookkeeping.
  CN: 这一段主要围绕 `postflightKey`, `endMapping`, `setError` 等符号展开，负责实现局部控制流程与状态维护。
- EN: In this range, the code iterates over collections, ranges, or records and checks conditions and handles edge cases.
  CN: 在这一段中，代码遍历集合、区间或记录，并检查条件并处理边界情况。

### Lines 241-300

```cpp
void Input::postflightElement(void *SaveInfo) {
  CurrentNode = reinterpret_cast<HNode *>(SaveInfo);
}

unsigned Input::beginFlowSequence() { return beginSequence(); }

bool Input::preflightFlowElement(unsigned index, void *&SaveInfo) {
  if (EC)
    return false;
  if (SequenceHNode *SQ = dyn_cast<SequenceHNode>(CurrentNode)) {
    SaveInfo = CurrentNode;
    CurrentNode = SQ->Entries[index];
    return true;
  }
  return false;
}

void Input::postflightFlowElement(void *SaveInfo) {
  CurrentNode = reinterpret_cast<HNode *>(SaveInfo);
}

void Input::endFlowSequence() {
}

void Input::beginEnumScalar() {
  ScalarMatchFound = false;
}

bool Input::matchEnumScalar(StringRef Str, bool) {
  if (ScalarMatchFound)
    return false;
  if (ScalarHNode *SN = dyn_cast<ScalarHNode>(CurrentNode)) {
    if (SN->value() == Str) {
      ScalarMatchFound = true;
      return true;
    }
  }
  return false;
}

bool Input::matchEnumFallback() {
  if (ScalarMatchFound)
    return false;
  ScalarMatchFound = true;
  return true;
}

void Input::endEnumScalar() {
  if (!ScalarMatchFound) {
    setError(CurrentNode, "unknown enumerated scalar");
  }
}

bool Input::beginBitSetScalar(bool &DoClear) {
  BitValuesUsed.clear();
  if (SequenceHNode *SQ = dyn_cast<SequenceHNode>(CurrentNode)) {
    BitValuesUsed.resize(SQ->Entries.size());
  } else {
    setError(CurrentNode, "expected sequence of bit values");
  }
```
- EN: This section centers on `postflightElement`, `beginFlowSequence`, `preflightFlowElement` and implements the local control flow and bookkeeping.
  CN: 这一段主要围绕 `postflightElement`, `beginFlowSequence`, `preflightFlowElement` 等符号展开，负责实现局部控制流程与状态维护。
- EN: In this range, the code checks conditions and handles edge cases and propagates LLVM-style errors and invariants.
  CN: 在这一段中，代码检查条件并处理边界情况，并传播 LLVM 风格的错误处理并维护不变量。

### Lines 301-360

```cpp
  DoClear = true;
  return true;
}

bool Input::bitSetMatch(StringRef Str, bool) {
  if (EC)
    return false;
  if (SequenceHNode *SQ = dyn_cast<SequenceHNode>(CurrentNode)) {
    unsigned Index = 0;
    for (auto &N : SQ->Entries) {
      if (ScalarHNode *SN = dyn_cast<ScalarHNode>(N)) {
        if (SN->value() == Str) {
          BitValuesUsed[Index] = true;
          return true;
        }
      } else {
        setError(CurrentNode, "unexpected scalar in sequence of bit values");
      }
      ++Index;
    }
  } else {
    setError(CurrentNode, "expected sequence of bit values");
  }
  return false;
}

void Input::endBitSetScalar() {
  if (EC)
    return;
  if (SequenceHNode *SQ = dyn_cast<SequenceHNode>(CurrentNode)) {
    assert(BitValuesUsed.size() == SQ->Entries.size());
    for (unsigned i = 0; i < SQ->Entries.size(); ++i) {
      if (!BitValuesUsed[i]) {
        setError(SQ->Entries[i], "unknown bit value");
        return;
      }
    }
  }
}

void Input::scalarString(StringRef &S, QuotingType) {
  if (ScalarHNode *SN = dyn_cast<ScalarHNode>(CurrentNode)) {
    S = SN->value();
  } else {
    setError(CurrentNode, "unexpected scalar");
  }
}

void Input::blockScalarString(StringRef &S) { scalarString(S, QuotingType::None); }

void Input::scalarTag(std::string &Tag) {
  Tag = CurrentNode->_node->getVerbatimTag();
}

void Input::setError(HNode *hnode, const Twine &message) {
  assert(hnode && "HNode must not be NULL");
  setError(hnode->_node, message);
}

NodeKind Input::getNodeKind() {
```
- EN: This section centers on `bitSetMatch`, `setError`, `endBitSetScalar` and queries existing state and returns computed metadata.
  CN: 这一段主要围绕 `bitSetMatch`, `setError`, `endBitSetScalar` 等符号展开，负责查询现有状态并返回计算出的元数据。
- EN: In this range, the code iterates over collections, ranges, or records and checks conditions and handles edge cases.
  CN: 在这一段中，代码遍历集合、区间或记录，并检查条件并处理边界情况。

### Lines 361-420

```cpp
  if (isa<ScalarHNode>(CurrentNode))
    return NodeKind::Scalar;
  else if (isa<MapHNode>(CurrentNode))
    return NodeKind::Map;
  else if (isa<SequenceHNode>(CurrentNode))
    return NodeKind::Sequence;
  llvm_unreachable("Unsupported node kind");
}

void Input::setError(Node *node, const Twine &message) {
  Strm->printError(node, message);
  EC = make_error_code(errc::invalid_argument);
}

void Input::setError(const SMRange &range, const Twine &message) {
  Strm->printError(range, message);
  EC = make_error_code(errc::invalid_argument);
}

void Input::reportWarning(HNode *hnode, const Twine &message) {
  assert(hnode && "HNode must not be NULL");
  Strm->printError(hnode->_node, message, SourceMgr::DK_Warning);
}

void Input::reportWarning(Node *node, const Twine &message) {
  Strm->printError(node, message, SourceMgr::DK_Warning);
}

void Input::reportWarning(const SMRange &range, const Twine &message) {
  Strm->printError(range, message, SourceMgr::DK_Warning);
}

void Input::releaseHNodeBuffers() {
  EmptyHNodeAllocator.DestroyAll();
  ScalarHNodeAllocator.DestroyAll();
  SequenceHNodeAllocator.DestroyAll();
  MapHNodeAllocator.DestroyAll();
}

void Input::saveAliasHNode(Node *N, HNode *HN) {
  StringRef Anchor = N->getAnchor();
  if (!Anchor.empty())
    // YAML 1.2.2 - 3.2.2.2. Anchors and Aliases:
    //
    // An alias event refers to the most recent event in the serialization
    // having the specified anchor. Therefore, anchors need not be unique within
    // a serialization. In addition, an anchor need not have an alias node
    // referring to it.
    AliasMap[Anchor] = HN;
}

Input::HNode *Input::createHNodes(Node *N) {
  SmallString<128> StringStorage;
  switch (N->getType()) {
  case Node::NK_Scalar: {
    ScalarNode *SN = dyn_cast<ScalarNode>(N);
    StringRef KeyStr = SN->getValue(StringStorage);
    if (!StringStorage.empty()) {
      // Copy string to permanent storage
      KeyStr = StringStorage.str().copy(StringAllocator);
```
- EN: This section centers on `llvm_unreachable`, `setError`, `reportWarning` and implements the local control flow and bookkeeping.
  CN: 这一段主要围绕 `llvm_unreachable`, `setError`, `reportWarning` 等符号展开，负责实现局部控制流程与状态维护。
- EN: In this range, the code uses a `switch` to dispatch behavior by kind or encoding and checks conditions and handles edge cases.
  CN: 在这一段中，代码使用 `switch` 按类型或编码分派行为，并检查条件并处理边界情况。

### Lines 421-480

```cpp
    }
    auto *SHNode = new (ScalarHNodeAllocator.Allocate()) ScalarHNode(N, KeyStr);
    saveAliasHNode(SN, SHNode);
    return SHNode;
  }
  case Node::NK_BlockScalar: {
    BlockScalarNode *BSN = dyn_cast<BlockScalarNode>(N);
    StringRef ValueCopy = BSN->getValue().copy(StringAllocator);
    auto *BSHNode =
        new (ScalarHNodeAllocator.Allocate()) ScalarHNode(N, ValueCopy);
    saveAliasHNode(BSN, BSHNode);
    return BSHNode;
  }
  case Node::NK_Sequence: {
    SequenceNode *SQ = dyn_cast<SequenceNode>(N);
    auto SQHNode = new (SequenceHNodeAllocator.Allocate()) SequenceHNode(N);
    for (Node &SN : *SQ) {
      auto Entry = createHNodes(&SN);
      if (EC)
        break;
      SQHNode->Entries.push_back(Entry);
    }
    saveAliasHNode(SQ, SQHNode);
    return SQHNode;
  }
  case Node::NK_Mapping: {
    MappingNode *Map = dyn_cast<MappingNode>(N);
    auto mapHNode = new (MapHNodeAllocator.Allocate()) MapHNode(N);
    for (KeyValueNode &KVN : *Map) {
      Node *KeyNode = KVN.getKey();
      ScalarNode *Key = dyn_cast_if_present<ScalarNode>(KeyNode);
      Node *Value = KVN.getValue();
      if (!Key || !Value) {
        if (!Key)
          setError(KeyNode, "Map key must be a scalar");
        if (!Value)
          setError(KeyNode, "Map value must not be empty");
        break;
      }
      StringStorage.clear();
      StringRef KeyStr = Key->getValue(StringStorage);
      if (!StringStorage.empty()) {
        // Copy string to permanent storage
        KeyStr = StringStorage.str().copy(StringAllocator);
      }
      if (mapHNode->Mapping.count(KeyStr))
        // From YAML spec: "The content of a mapping node is an unordered set of
        // key/value node pairs, with the restriction that each of the keys is
        // unique."
        setError(KeyNode, Twine("duplicated mapping key '") + KeyStr + "'");
      auto ValueHNode = createHNodes(Value);
      if (EC)
        break;
      mapHNode->Mapping[KeyStr] =
          std::make_pair(std::move(ValueHNode), KeyNode->getSourceRange());
    }
    saveAliasHNode(Map, mapHNode);
    return std::move(mapHNode);
  }
  case Node::NK_Null:
```
- EN: This section centers on `saveAliasHNode`, `make_pair`, `move` and creates and initializes supporting objects or state.
  CN: 这一段主要围绕 `saveAliasHNode`, `make_pair`, `move` 等符号展开，负责创建并初始化辅助对象或状态。
- EN: In this range, the code iterates over collections, ranges, or records and checks conditions and handles edge cases.
  CN: 在这一段中，代码遍历集合、区间或记录，并检查条件并处理边界情况。

### Lines 481-540

```cpp
    // TODO: Anchor is not set for NullNode in the parser. Update the parser to
    // faithfully preserve anchors.
    return new (EmptyHNodeAllocator.Allocate()) EmptyHNode(N);
  case Node::NK_Alias: {
    AliasNode *AN = dyn_cast<AliasNode>(N);
    auto AliasName = AN->getName();
    auto AHN = AliasMap.find(AliasName);
    if (AHN == AliasMap.end()) {
      setError(AN, Twine("undefined alias '" + AliasName + "'"));
      return nullptr;
    }
    return AHN->second;
  }
  default:
    setError(N, "unknown node kind");
    return nullptr;
  }
}

void Input::setError(const Twine &Message) {
  setError(CurrentNode, Message);
}

void Input::setAllowUnknownKeys(bool Allow) { AllowUnknownKeys = Allow; }

bool Input::canElideEmptySequence() {
  return false;
}

//===----------------------------------------------------------------------===//
//  Output
//===----------------------------------------------------------------------===//

Output::Output(raw_ostream &yout, void *context, int WrapColumn)
    : IO(context), Out(yout), WrapColumn(WrapColumn) {}

Output::~Output() = default;

bool Output::outputting() const {
  return true;
}

void Output::beginMapping() {
  StateStack.push_back(inMapFirstKey);
  PaddingBeforeContainer = Padding;
  Padding = "\n";
}

bool Output::mapTag(StringRef Tag, bool Use) {
  if (Use) {
    // If this tag is being written inside a sequence we should write the start
    // of the sequence before writing the tag, otherwise the tag won't be
    // attached to the element in the sequence, but rather the sequence itself.
    bool SequenceElement = false;
    if (StateStack.size() > 1) {
      auto &E = StateStack[StateStack.size() - 2];
      SequenceElement = inSeqAnyElement(E) || inFlowSeqAnyElement(E);
    }
    if (SequenceElement && StateStack.back() == inMapFirstKey) {
      newLineCheck();
```
- EN: This section centers on `setError`, `setAllowUnknownKeys`, `canElideEmptySequence` and parses input and converts raw data into structured form.
  CN: 这一段主要围绕 `setError`, `setAllowUnknownKeys`, `canElideEmptySequence` 等符号展开，负责解析输入并把原始数据转换成结构化形式。
- EN: In this range, the code checks conditions and handles edge cases and propagates LLVM-style errors and invariants.
  CN: 在这一段中，代码检查条件并处理边界情况，并传播 LLVM 风格的错误处理并维护不变量。

### Lines 541-600

```cpp
    } else {
      output(" ");
    }
    output(Tag);
    if (SequenceElement) {
      // If we're writing the tag during the first element of a map, the tag
      // takes the place of the first element in the sequence.
      if (StateStack.back() == inMapFirstKey) {
        StateStack.pop_back();
        StateStack.push_back(inMapOtherKey);
      }
      // Tags inside maps in sequences should act as keys in the map from a
      // formatting perspective, so we always want a newline in a sequence.
      Padding = "\n";
    }
  }
  return Use;
}

void Output::endMapping() {
  // If we did not map anything, we should explicitly emit an empty map
  if (StateStack.back() == inMapFirstKey) {
    Padding = PaddingBeforeContainer;
    newLineCheck();
    output("{}");
    Padding = "\n";
  }
  StateStack.pop_back();
}

std::vector<StringRef> Output::keys() {
  report_fatal_error("invalid call");
}

bool Output::preflightKey(StringRef Key, bool Required, bool SameAsDefault,
                          bool &UseDefault, void *&SaveInfo) {
  UseDefault = false;
  SaveInfo = nullptr;
  if (Required || !SameAsDefault || WriteDefaultValues) {
    auto State = StateStack.back();
    if (State == inFlowMapFirstKey || State == inFlowMapOtherKey) {
      flowKey(Key);
    } else {
      newLineCheck();
      paddedKey(Key);
    }
    return true;
  }
  return false;
}

void Output::postflightKey(void *) {
  if (StateStack.back() == inMapFirstKey) {
    StateStack.pop_back();
    StateStack.push_back(inMapOtherKey);
  } else if (StateStack.back() == inFlowMapFirstKey) {
    StateStack.pop_back();
    StateStack.push_back(inFlowMapOtherKey);
  }
}
```
- EN: This section centers on `output`, `endMapping`, `newLineCheck` and writes, formats, or serializes results for downstream consumers.
  CN: 这一段主要围绕 `output`, `endMapping`, `newLineCheck` 等符号展开，负责为下游使用方写出、格式化或序列化结果。
- EN: In this range, the code checks conditions and handles edge cases and updates helper containers and temporary state.
  CN: 在这一段中，代码检查条件并处理边界情况，并更新辅助容器和临时状态。

### Lines 601-660

```cpp

void Output::beginFlowMapping() {
  StateStack.push_back(inFlowMapFirstKey);
  newLineCheck();
  ColumnAtMapFlowStart = Column;
  output("{ ");
}

void Output::endFlowMapping() {
  StateStack.pop_back();
  outputUpToEndOfLine(" }");
}

void Output::beginDocuments() {
  outputUpToEndOfLine("---");
}

bool Output::preflightDocument(unsigned index) {
  if (index > 0)
    outputUpToEndOfLine("\n---");
  return true;
}

void Output::postflightDocument() {
}

void Output::endDocuments() {
  output("\n...\n");
}

unsigned Output::beginSequence() {
  StateStack.push_back(inSeqFirstElement);
  PaddingBeforeContainer = Padding;
  Padding = "\n";
  return 0;
}

void Output::endSequence() {
  // If we did not emit anything, we should explicitly emit an empty sequence
  if (StateStack.back() == inSeqFirstElement) {
    Padding = PaddingBeforeContainer;
    newLineCheck(/*EmptySequence=*/true);
    output("[]");
    Padding = "\n";
  }
  StateStack.pop_back();
}

bool Output::preflightElement(unsigned, void *&SaveInfo) {
  SaveInfo = nullptr;
  return true;
}

void Output::postflightElement(void *) {
  if (StateStack.back() == inSeqFirstElement) {
    StateStack.pop_back();
    StateStack.push_back(inSeqOtherElement);
  } else if (StateStack.back() == inFlowSeqFirstElement) {
    StateStack.pop_back();
    StateStack.push_back(inFlowSeqOtherElement);
```
- EN: This section centers on `beginFlowMapping`, `newLineCheck`, `endFlowMapping` and writes, formats, or serializes results for downstream consumers.
  CN: 这一段主要围绕 `beginFlowMapping`, `newLineCheck`, `endFlowMapping` 等符号展开，负责为下游使用方写出、格式化或序列化结果。
- EN: In this range, the code checks conditions and handles edge cases and returns the resulting value to its callers.
  CN: 在这一段中，代码检查条件并处理边界情况，并将结果返回给调用方。

### Lines 661-720

```cpp
  }
}

unsigned Output::beginFlowSequence() {
  StateStack.push_back(inFlowSeqFirstElement);
  newLineCheck();
  ColumnAtFlowStart = Column;
  output("[ ");
  NeedFlowSequenceComma = false;
  return 0;
}

void Output::endFlowSequence() {
  StateStack.pop_back();
  outputUpToEndOfLine(" ]");
}

bool Output::preflightFlowElement(unsigned, void *&SaveInfo) {
  if (NeedFlowSequenceComma)
    output(", ");
  if (WrapColumn && Column > WrapColumn) {
    output("\n");
    for (int i = 0; i < ColumnAtFlowStart; ++i)
      output(" ");
    Column = ColumnAtFlowStart;
    output("  ");
  }
  SaveInfo = nullptr;
  return true;
}

void Output::postflightFlowElement(void *) {
  NeedFlowSequenceComma = true;
}

void Output::beginEnumScalar() {
  EnumerationMatchFound = false;
}

bool Output::matchEnumScalar(StringRef Str, bool Match) {
  if (Match && !EnumerationMatchFound) {
    newLineCheck();
    outputUpToEndOfLine(Str);
    EnumerationMatchFound = true;
  }
  return false;
}

bool Output::matchEnumFallback() {
  if (EnumerationMatchFound)
    return false;
  EnumerationMatchFound = true;
  return true;
}

void Output::endEnumScalar() {
  if (!EnumerationMatchFound)
    llvm_unreachable("bad runtime enum value");
}

```
- EN: This section centers on `beginFlowSequence`, `newLineCheck`, `output` and implements the local control flow and bookkeeping.
  CN: 这一段主要围绕 `beginFlowSequence`, `newLineCheck`, `output` 等符号展开，负责实现局部控制流程与状态维护。
- EN: In this range, the code iterates over collections, ranges, or records and checks conditions and handles edge cases.
  CN: 在这一段中，代码遍历集合、区间或记录，并检查条件并处理边界情况。

### Lines 721-780

```cpp
bool Output::beginBitSetScalar(bool &DoClear) {
  newLineCheck();
  output("[ ");
  NeedBitValueComma = false;
  DoClear = false;
  return true;
}

bool Output::bitSetMatch(StringRef Str, bool Matches) {
  if (Matches) {
    if (NeedBitValueComma)
      output(", ");
    output(Str);
    NeedBitValueComma = true;
  }
  return false;
}

void Output::endBitSetScalar() {
  outputUpToEndOfLine(" ]");
}

void Output::scalarString(StringRef &S, QuotingType MustQuote) {
  newLineCheck();
  if (S.empty()) {
    // Print '' for the empty string because leaving the field empty is not
    // allowed.
    outputUpToEndOfLine("''");
    return;
  }
  output(S, MustQuote);
  outputUpToEndOfLine("");
}

void Output::blockScalarString(StringRef &S) {
  if (!StateStack.empty())
    newLineCheck();
  output(" |");

  unsigned Indent = StateStack.empty() ? 1 : StateStack.size();

  auto Buffer = MemoryBuffer::getMemBuffer(S, "", false);
  for (line_iterator Lines(*Buffer, false); !Lines.is_at_end(); ++Lines) {
    outputNewLine();
    for (unsigned I = 0; I < Indent; ++I) {
      output("  ");
    }
    output(*Lines);
  }
  outputUpToEndOfLine("");
}

void Output::scalarTag(std::string &Tag) {
  if (Tag.empty())
    return;
  newLineCheck();
  output(Tag);
  output(" ");
}

```
- EN: This section centers on `beginBitSetScalar`, `newLineCheck`, `output` and implements the local control flow and bookkeeping.
  CN: 这一段主要围绕 `beginBitSetScalar`, `newLineCheck`, `output` 等符号展开，负责实现局部控制流程与状态维护。
- EN: In this range, the code iterates over collections, ranges, or records and checks conditions and handles edge cases.
  CN: 在这一段中，代码遍历集合、区间或记录，并检查条件并处理边界情况。

### Lines 781-840

```cpp
void Output::setError(const Twine &message) {
}

std::error_code Output::error() { return {}; }

bool Output::canElideEmptySequence() {
  // Normally, with an optional key/value where the value is an empty sequence,
  // the whole key/value can be not written.  But, that produces wrong yaml
  // if the key/value is the only thing in the map and the map is used in
  // a sequence.  This detects if the this sequence is the first key/value
  // in map that itself is embedded in a sequence.
  if (StateStack.size() < 2)
    return true;
  if (StateStack.back() != inMapFirstKey)
    return true;
  return !inSeqAnyElement(StateStack[StateStack.size() - 2]);
}

void Output::output(StringRef s) {
  Column += s.size();
  Out << s;
}

void Output::output(StringRef S, QuotingType MustQuote) {
  if (MustQuote == QuotingType::None) {
    // Only quote if we must.
    output(S);
    return;
  }

  StringLiteral Quote = MustQuote == QuotingType::Single ? StringLiteral("'")
                                                         : StringLiteral("\"");
  output(Quote); // Starting quote.

  // When using double-quoted strings (and only in that case), non-printable
  // characters may be present, and will be escaped using a variety of
  // unicode-scalar and special short-form escapes. This is handled in
  // yaml::escape.
  if (MustQuote == QuotingType::Double) {
    output(yaml::escape(S, /* EscapePrintable= */ false));
    output(Quote);
    return;
  }

  unsigned i = 0;
  unsigned j = 0;
  unsigned End = S.size();
  const char *Base = S.data();

  // When using single-quoted strings, any single quote ' must be doubled to be
  // escaped.
  while (j < End) {
    if (S[j] == '\'') {                   // Escape quotes.
      output(StringRef(&Base[i], j - i)); // "flush".
      output(StringLiteral("''"));        // Print it as ''
      i = j + 1;
    }
    ++j;
  }
  output(StringRef(&Base[i], j - i));
```
- EN: This section centers on `setError`, `error`, `canElideEmptySequence` and implements the local control flow and bookkeeping.
  CN: 这一段主要围绕 `setError`, `error`, `canElideEmptySequence` 等符号展开，负责实现局部控制流程与状态维护。
- EN: In this range, the code iterates over collections, ranges, or records and checks conditions and handles edge cases.
  CN: 在这一段中，代码遍历集合、区间或记录，并检查条件并处理边界情况。

### Lines 841-900

```cpp
  output(Quote); // Ending quote.
}

void Output::outputUpToEndOfLine(StringRef s) {
  output(s);
  if (StateStack.empty() || (!inFlowSeqAnyElement(StateStack.back()) &&
                             !inFlowMapAnyKey(StateStack.back())))
    Padding = "\n";
}

void Output::outputNewLine() {
  Out << "\n";
  Column = 0;
}

// if seq at top, indent as if map, then add "- "
// if seq in middle, use "- " if firstKey, else use "  "
//

void Output::newLineCheck(bool EmptySequence) {
  if (Padding != "\n") {
    output(Padding);
    Padding = {};
    return;
  }
  outputNewLine();
  Padding = {};

  if (StateStack.size() == 0 || EmptySequence)
    return;

  unsigned Indent = StateStack.size() - 1;
  bool PossiblyNestedSeq = false;
  auto I = StateStack.rbegin(), E = StateStack.rend();

  if (inSeqAnyElement(*I)) {
    PossiblyNestedSeq = true; // Not possibly but always.
    ++Indent;
  } else if (*I == inMapFirstKey || *I == inFlowMapFirstKey ||
             inFlowSeqAnyElement(*I)) {
    PossiblyNestedSeq = true;
    ++I; // Skip back().
  }

  unsigned OutputDashCount = 0;
  if (PossiblyNestedSeq) {
    // Count up consecutive inSeqFirstElement from the end, unless
    // inSeqFirstElement is the top of nested sequence.
    while (I != E) {
      // Don't count the top of nested sequence.
      if (!inSeqAnyElement(*I))
        break;

      ++OutputDashCount;

      // Stop counting if consecutive inSeqFirstElement ends.
      if (*I++ != inSeqFirstElement)
        break;
    }
  }
```
- EN: This section centers on `output`, `outputUpToEndOfLine`, `outputNewLine` and implements the local control flow and bookkeeping.
  CN: 这一段主要围绕 `output`, `outputUpToEndOfLine`, `outputNewLine` 等符号展开，负责实现局部控制流程与状态维护。
- EN: In this range, the code iterates over collections, ranges, or records and checks conditions and handles edge cases.
  CN: 在这一段中，代码遍历集合、区间或记录，并检查条件并处理边界情况。

### Lines 901-960

```cpp

  for (unsigned I = OutputDashCount; I < Indent; ++I)
    output("  ");

  for (unsigned I = 0; I < OutputDashCount; ++I)
    output("- ");
}

void Output::paddedKey(StringRef key) {
  output(key, needsQuotes(key, false));
  output(":");
  const char *spaces = "                ";
  if (key.size() < strlen(spaces))
    Padding = &spaces[key.size()];
  else
    Padding = " ";
}

void Output::flowKey(StringRef Key) {
  if (StateStack.back() == inFlowMapOtherKey)
    output(", ");
  if (WrapColumn && Column > WrapColumn) {
    output("\n");
    for (int I = 0; I < ColumnAtMapFlowStart; ++I)
      output(" ");
    Column = ColumnAtMapFlowStart;
    output("  ");
  }
  output(Key, needsQuotes(Key, false));
  output(": ");
}

NodeKind Output::getNodeKind() { report_fatal_error("invalid call"); }

bool Output::inSeqAnyElement(InState State) {
  return State == inSeqFirstElement || State == inSeqOtherElement;
}

bool Output::inFlowSeqAnyElement(InState State) {
  return State == inFlowSeqFirstElement || State == inFlowSeqOtherElement;
}

bool Output::inMapAnyKey(InState State) {
  return State == inMapFirstKey || State == inMapOtherKey;
}

bool Output::inFlowMapAnyKey(InState State) {
  return State == inFlowMapFirstKey || State == inFlowMapOtherKey;
}

//===----------------------------------------------------------------------===//
//  traits for built-in types
//===----------------------------------------------------------------------===//

void ScalarTraits<bool>::output(const bool &Val, void *, raw_ostream &Out) {
  Out << (Val ? "true" : "false");
}

StringRef ScalarTraits<bool>::input(StringRef Scalar, void *, bool &Val) {
  if (std::optional<bool> Parsed = parseBool(Scalar)) {
```
- EN: This section centers on `output`, `paddedKey`, `flowKey` and queries existing state and returns computed metadata.
  CN: 这一段主要围绕 `output`, `paddedKey`, `flowKey` 等符号展开，负责查询现有状态并返回计算出的元数据。
- EN: In this range, the code iterates over collections, ranges, or records and checks conditions and handles edge cases.
  CN: 在这一段中，代码遍历集合、区间或记录，并检查条件并处理边界情况。

### Lines 961-1020

```cpp
    Val = *Parsed;
    return StringRef();
  }
  return "invalid boolean";
}

void ScalarTraits<StringRef>::output(const StringRef &Val, void *,
                                     raw_ostream &Out) {
  Out << Val;
}

StringRef ScalarTraits<StringRef>::input(StringRef Scalar, void *,
                                         StringRef &Val) {
  Val = Scalar;
  return StringRef();
}

void ScalarTraits<std::string>::output(const std::string &Val, void *,
                                       raw_ostream &Out) {
  Out << Val;
}

StringRef ScalarTraits<std::string>::input(StringRef Scalar, void *,
                                           std::string &Val) {
  Val = Scalar.str();
  return StringRef();
}

void ScalarTraits<uint8_t>::output(const uint8_t &Val, void *,
                                   raw_ostream &Out) {
  // use temp uin32_t because ostream thinks uint8_t is a character
  uint32_t Num = Val;
  Out << Num;
}

StringRef ScalarTraits<uint8_t>::input(StringRef Scalar, void *, uint8_t &Val) {
  unsigned long long n;
  if (getAsUnsignedInteger(Scalar, 0, n))
    return "invalid number";
  if (n > 0xFF)
    return "out of range number";
  Val = n;
  return StringRef();
}

void ScalarTraits<uint16_t>::output(const uint16_t &Val, void *,
                                    raw_ostream &Out) {
  Out << Val;
}

StringRef ScalarTraits<uint16_t>::input(StringRef Scalar, void *,
                                        uint16_t &Val) {
  unsigned long long n;
  if (getAsUnsignedInteger(Scalar, 0, n))
    return "invalid number";
  if (n > 0xFFFF)
    return "out of range number";
  Val = n;
  return StringRef();
}
```
- EN: This section centers on `StringRef` and implements the local control flow and bookkeeping.
  CN: 这一段主要围绕 `StringRef` 等符号展开，负责实现局部控制流程与状态维护。
- EN: In this range, the code checks conditions and handles edge cases and returns the resulting value to its callers.
  CN: 在这一段中，代码检查条件并处理边界情况，并将结果返回给调用方。

### Lines 1021-1080

```cpp

void ScalarTraits<uint32_t>::output(const uint32_t &Val, void *,
                                    raw_ostream &Out) {
  Out << Val;
}

StringRef ScalarTraits<uint32_t>::input(StringRef Scalar, void *,
                                        uint32_t &Val) {
  unsigned long long n;
  if (getAsUnsignedInteger(Scalar, 0, n))
    return "invalid number";
  if (n > 0xFFFFFFFFUL)
    return "out of range number";
  Val = n;
  return StringRef();
}

void ScalarTraits<uint64_t>::output(const uint64_t &Val, void *,
                                    raw_ostream &Out) {
  Out << Val;
}

StringRef ScalarTraits<uint64_t>::input(StringRef Scalar, void *,
                                        uint64_t &Val) {
  unsigned long long N;
  if (getAsUnsignedInteger(Scalar, 0, N))
    return "invalid number";
  Val = N;
  return StringRef();
}

void ScalarTraits<int8_t>::output(const int8_t &Val, void *, raw_ostream &Out) {
  // use temp in32_t because ostream thinks int8_t is a character
  int32_t Num = Val;
  Out << Num;
}

StringRef ScalarTraits<int8_t>::input(StringRef Scalar, void *, int8_t &Val) {
  long long N;
  if (getAsSignedInteger(Scalar, 0, N))
    return "invalid number";
  if ((N > 127) || (N < -128))
    return "out of range number";
  Val = N;
  return StringRef();
}

void ScalarTraits<int16_t>::output(const int16_t &Val, void *,
                                   raw_ostream &Out) {
  Out << Val;
}

StringRef ScalarTraits<int16_t>::input(StringRef Scalar, void *, int16_t &Val) {
  long long N;
  if (getAsSignedInteger(Scalar, 0, N))
    return "invalid number";
  if ((N > INT16_MAX) || (N < INT16_MIN))
    return "out of range number";
  Val = N;
  return StringRef();
```
- EN: This section centers on `StringRef` and implements the local control flow and bookkeeping.
  CN: 这一段主要围绕 `StringRef` 等符号展开，负责实现局部控制流程与状态维护。
- EN: In this range, the code checks conditions and handles edge cases and returns the resulting value to its callers.
  CN: 在这一段中，代码检查条件并处理边界情况，并将结果返回给调用方。

### Lines 1081-1140

```cpp
}

void ScalarTraits<int32_t>::output(const int32_t &Val, void *,
                                   raw_ostream &Out) {
  Out << Val;
}

StringRef ScalarTraits<int32_t>::input(StringRef Scalar, void *, int32_t &Val) {
  long long N;
  if (getAsSignedInteger(Scalar, 0, N))
    return "invalid number";
  if ((N > INT32_MAX) || (N < INT32_MIN))
    return "out of range number";
  Val = N;
  return StringRef();
}

void ScalarTraits<int64_t>::output(const int64_t &Val, void *,
                                   raw_ostream &Out) {
  Out << Val;
}

StringRef ScalarTraits<int64_t>::input(StringRef Scalar, void *, int64_t &Val) {
  long long N;
  if (getAsSignedInteger(Scalar, 0, N))
    return "invalid number";
  Val = N;
  return StringRef();
}

void ScalarTraits<double>::output(const double &Val, void *, raw_ostream &Out) {
  Out << format("%g", Val);
}

StringRef ScalarTraits<double>::input(StringRef Scalar, void *, double &Val) {
  if (to_float(Scalar, Val))
    return StringRef();
  return "invalid floating point number";
}

void ScalarTraits<float>::output(const float &Val, void *, raw_ostream &Out) {
  Out << format("%g", Val);
}

StringRef ScalarTraits<float>::input(StringRef Scalar, void *, float &Val) {
  if (to_float(Scalar, Val))
    return StringRef();
  return "invalid floating point number";
}

void ScalarTraits<Hex8>::output(const Hex8 &Val, void *, raw_ostream &Out) {
  Out << format("0x%" PRIX8, (uint8_t)Val);
}

StringRef ScalarTraits<Hex8>::input(StringRef Scalar, void *, Hex8 &Val) {
  unsigned long long n;
  if (getAsUnsignedInteger(Scalar, 0, n))
    return "invalid hex8 number";
  if (n > 0xFF)
    return "out of range hex8 number";
```
- EN: This section centers on `StringRef`, `format` and writes, formats, or serializes results for downstream consumers.
  CN: 这一段主要围绕 `StringRef`, `format` 等符号展开，负责为下游使用方写出、格式化或序列化结果。
- EN: In this range, the code checks conditions and handles edge cases and returns the resulting value to its callers.
  CN: 在这一段中，代码检查条件并处理边界情况，并将结果返回给调用方。

### Lines 1141-1195

```cpp
  Val = n;
  return StringRef();
}

void ScalarTraits<Hex16>::output(const Hex16 &Val, void *, raw_ostream &Out) {
  Out << format("0x%" PRIX16, (uint16_t)Val);
}

StringRef ScalarTraits<Hex16>::input(StringRef Scalar, void *, Hex16 &Val) {
  unsigned long long n;
  if (getAsUnsignedInteger(Scalar, 0, n))
    return "invalid hex16 number";
  if (n > 0xFFFF)
    return "out of range hex16 number";
  Val = n;
  return StringRef();
}

void ScalarTraits<Hex32>::output(const Hex32 &Val, void *, raw_ostream &Out) {
  Out << format("0x%" PRIX32, (uint32_t)Val);
}

StringRef ScalarTraits<Hex32>::input(StringRef Scalar, void *, Hex32 &Val) {
  unsigned long long n;
  if (getAsUnsignedInteger(Scalar, 0, n))
    return "invalid hex32 number";
  if (n > 0xFFFFFFFFUL)
    return "out of range hex32 number";
  Val = n;
  return StringRef();
}

void ScalarTraits<Hex64>::output(const Hex64 &Val, void *, raw_ostream &Out) {
  Out << format("0x%" PRIX64, (uint64_t)Val);
}

StringRef ScalarTraits<Hex64>::input(StringRef Scalar, void *, Hex64 &Val) {
  unsigned long long Num;
  if (getAsUnsignedInteger(Scalar, 0, Num))
    return "invalid hex64 number";
  Val = Num;
  return StringRef();
}

void ScalarTraits<VersionTuple>::output(const VersionTuple &Val, void *,
                                        llvm::raw_ostream &Out) {
  Out << Val.getAsString();
}

StringRef ScalarTraits<VersionTuple>::input(StringRef Scalar, void *,
                                            VersionTuple &Val) {
  if (Val.tryParse(Scalar))
    return "invalid version format";
  return StringRef();
}
```
- EN: This section centers on `StringRef`, `format` and writes, formats, or serializes results for downstream consumers.
  CN: 这一段主要围绕 `StringRef`, `format` 等符号展开，负责为下游使用方写出、格式化或序列化结果。
- EN: In this range, the code checks conditions and handles edge cases and returns the resulting value to its callers.
  CN: 在这一段中，代码检查条件并处理边界情况，并将结果返回给调用方。

## Key Concepts / 关键概念
- Domain / 领域: YAML processing / YAML 处理
- Core symbols / 核心符号: `value`, `IO`, `setContext`, `setAllowUnknownKeys`, `llvm_unreachable` / 该文件围绕这些类型或函数组织主要逻辑。
- Data flow / 数据流: Parsing, decoding, and structural validation. / 重点关注解析、解码与结构校验。
- Error model / 错误模型: LLVM-style `Error`/`Expected` handling and invariant checks. / 使用 LLVM 风格的 `Error`/`Expected` 处理与不变量检查。
- Data structures / 数据结构: Relies on LLVM or STL containers for compact state management. / 依赖 LLVM 或 STL 容器管理紧凑状态。

## Dependencies / 依赖关系
- LLVM headers / LLVM 头文件: `llvm/Support/YAMLTraits.h`, `llvm/ADT/STLExtras.h`, `llvm/ADT/SmallString.h`, `llvm/ADT/StringExtras.h`, `llvm/ADT/StringRef.h`, `llvm/ADT/Twine.h`, `llvm/Support/Casting.h`, `llvm/Support/Errc.h`, `llvm/Support/ErrorHandling.h`, `llvm/Support/Format.h`, `llvm/Support/LineIterator.h`, `llvm/Support/MemoryBuffer.h`, `llvm/Support/VersionTuple.h`, `llvm/Support/YAMLParser.h`, `llvm/Support/raw_ostream.h`
- Standard library / 标准库: `cstdint`, `cstring`, `string`, `vector`
- Other/system headers / 其他或系统头文件: `cassert`
- Related symbols / 相关符号: `value`, `IO`, `setContext`, `setAllowUnknownKeys`, `llvm_unreachable`, `Input`
