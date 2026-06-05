# LineEditor.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `llvm/lib/LineEditor/LineEditor.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: Implements line-editing support for interactive LLVM tools.
  - **CN**: 实现交互式 LLVM 工具所需的行编辑支持。

## Line-by-Line Analysis / 逐行分析

### Lines 1-20
```cpp
//===-- LineEditor.cpp - line editor --------------------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#include "llvm/LineEditor/LineEditor.h"
#include "llvm/ADT/STLExtras.h"
#include "llvm/ADT/SmallString.h"
#include "llvm/Config/config.h"
#include "llvm/Support/Path.h"
#include "llvm/Support/raw_ostream.h"
#include <algorithm>
#include <cassert>
#include <cstdio>
#ifdef HAVE_LIBEDIT
#include <histedit.h>
constexpr int DefaultHistorySize = 800;
```
- **EN**: Pulls in the headers needed by this translation unit, including `llvm/LineEditor/LineEditor.h`, `llvm/ADT/STLExtras.h`, `llvm/ADT/SmallString.h`, `llvm/Config/config.h`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `llvm/LineEditor/LineEditor.h`, `llvm/ADT/STLExtras.h`, `llvm/ADT/SmallString.h`, `llvm/Config/config.h`。

### Lines 21-33
```cpp
#endif

using namespace llvm;

std::string LineEditor::getDefaultHistoryPath(StringRef ProgName) {
  SmallString<32> Path;
  if (sys::path::home_directory(Path)) {
    sys::path::append(Path, "." + ProgName + "-history");
    return std::string(Path);
  }
  return std::string();
}

```
- **EN**: Introduces declarations for `llvm`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `llvm` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 34-53
```cpp
LineEditor::CompleterConcept::~CompleterConcept() = default;
LineEditor::ListCompleterConcept::~ListCompleterConcept() = default;

std::string LineEditor::ListCompleterConcept::getCommonPrefix(
    const std::vector<Completion> &Comps) {
  assert(!Comps.empty());

  std::string CommonPrefix = Comps[0].TypedText;
  for (const Completion &C : llvm::drop_begin(Comps)) {
    size_t Len = std::min(CommonPrefix.size(), C.TypedText.size());
    size_t CommonLen = 0;
    for (; CommonLen != Len; ++CommonLen) {
      if (CommonPrefix[CommonLen] != C.TypedText[CommonLen])
        break;
    }
    CommonPrefix.resize(CommonLen);
  }
  return CommonPrefix;
}

```
- **EN**: Implements logic around `~CompleterConcept`, `~ListCompleterConcept`, `getCommonPrefix`, `assert`, and 3 more symbols.
- **CN**: 围绕 `~CompleterConcept`, `~ListCompleterConcept`, `getCommonPrefix`, `assert`, and 3 more symbols 实现具体逻辑。

### Lines 54-64
```cpp
LineEditor::CompletionAction
LineEditor::ListCompleterConcept::complete(StringRef Buffer, size_t Pos) const {
  CompletionAction Action;
  std::vector<Completion> Comps = getCompletions(Buffer, Pos);
  if (Comps.empty()) {
    Action.Kind = CompletionAction::AK_ShowCompletions;
    return Action;
  }

  std::string CommonPrefix = getCommonPrefix(Comps);

```
- **EN**: Implements logic around `complete`, `getCompletions`, `empty`, `getCommonPrefix`.
- **CN**: 围绕 `complete`, `getCompletions`, `empty`, `getCommonPrefix` 实现具体逻辑。

### Lines 65-78
```cpp
  // If the common prefix is non-empty we can simply insert it. If there is a
  // single completion, this will insert the full completion. If there is more
  // than one, this might be enough information to jog the user's memory but if
  // not the user can also hit tab again to see the completions because the
  // common prefix will then be empty.
  if (CommonPrefix.empty()) {
    Action.Kind = CompletionAction::AK_ShowCompletions;
    for (const Completion &Comp : Comps)
      Action.Completions.push_back(Comp.DisplayText);
  } else {
    Action.Kind = CompletionAction::AK_Insert;
    Action.Text = CommonPrefix;
  }

```
- **EN**: Implements logic around `empty`, `push_back`.
- **CN**: 围绕 `empty`, `push_back` 实现具体逻辑。

### Lines 79-89
```cpp
  return Action;
}

LineEditor::CompletionAction LineEditor::getCompletionAction(StringRef Buffer,
                                                             size_t Pos) const {
  if (!Completer) {
    CompletionAction Action;
    Action.Kind = CompletionAction::AK_ShowCompletions;
    return Action;
  }

```
- **EN**: Implements logic around `getCompletionAction`.
- **CN**: 围绕 `getCompletionAction` 实现具体逻辑。

### Lines 90-99
```cpp
  return Completer->complete(Buffer, Pos);
}

#ifdef HAVE_LIBEDIT

// libedit-based implementation.

struct LineEditor::InternalData {
  LineEditor *LE;

```
- **EN**: Defines preprocessor-controlled structure, feature gates, or compile-time constants.
- **CN**: 定义受预处理器控制的结构、特性开关或编译期常量。

### Lines 100-109
```cpp
  History *Hist;
  EditLine *EL;

  unsigned PrevCount;
  std::string ContinuationOutput;

  FILE *Out;
};

namespace {
```
- **EN**: Contains supporting implementation details for the surrounding LLVM library component.
- **CN**: 包含周边 LLVM 库组件所需的辅助实现细节。

### Lines 110-128
```cpp

const char *ElGetPromptFn(EditLine *EL) {
  LineEditor::InternalData *Data;
  if (el_get(EL, EL_CLIENTDATA, &Data) == 0)
    return Data->LE->getPrompt().c_str();
  return "> ";
}

// Handles tab completion.
//
// This function is really horrible. But since the alternative is to get into
// the line editor business, here we are.
unsigned char ElCompletionFn(EditLine *EL, int ch) {
  LineEditor::InternalData *Data;
  if (el_get(EL, EL_CLIENTDATA, &Data) == 0) {
    if (!Data->ContinuationOutput.empty()) {
      // This is the continuation of the AK_ShowCompletions branch below.
      FILE *Out = Data->Out;

```
- **EN**: Implements logic around `ElGetPromptFn`, `el_get`, `getPrompt`, `ElCompletionFn`, and 1 more symbols.
- **CN**: 围绕 `ElGetPromptFn`, `el_get`, `getPrompt`, `ElCompletionFn`, and 1 more symbols 实现具体逻辑。

### Lines 129-139
```cpp
      // Print the required output (see below).
      ::fwrite(Data->ContinuationOutput.c_str(),
               Data->ContinuationOutput.size(), 1, Out);

      // Push a sequence of Ctrl-B characters to move the cursor back to its
      // original position.
      std::string Prevs(Data->PrevCount, '\02');
      ::el_push(EL, const_cast<char *>(Prevs.c_str()));

      Data->ContinuationOutput.clear();

```
- **EN**: Implements logic around `fwrite`, `size`, `Prevs`, `el_push`, and 1 more symbols; this block emits or serializes data to an external representation.
- **CN**: 围绕 `fwrite`, `size`, `Prevs`, `el_push`, and 1 more symbols 实现具体逻辑；该代码块把数据输出或序列化为外部表示。

### Lines 140-151
```cpp
      return CC_REFRESH;
    }

    const LineInfo *LI = ::el_line(EL);
    LineEditor::CompletionAction Action = Data->LE->getCompletionAction(
        StringRef(LI->buffer, LI->lastchar - LI->buffer),
        LI->cursor - LI->buffer);
    switch (Action.Kind) {
    case LineEditor::CompletionAction::AK_Insert:
      ::el_insertstr(EL, Action.Text.c_str());
      return CC_REFRESH;

```
- **EN**: Implements logic around `el_line`, `getCompletionAction`, `StringRef`, `el_insertstr`; this block uses `switch`-style dispatch.
- **CN**: 围绕 `el_line`, `getCompletionAction`, `StringRef`, `el_insertstr` 实现具体逻辑；该代码块使用 `switch` 风格分派。

### Lines 152-164
```cpp
    case LineEditor::CompletionAction::AK_ShowCompletions:
      if (Action.Completions.empty()) {
        return CC_REFRESH_BEEP;
      } else {
        // Push a Ctrl-E and a tab. The Ctrl-E causes libedit to move the cursor
        // to the end of the line, so that when we emit a newline we will be on
        // a new blank line. The tab causes libedit to call this function again
        // after moving the cursor. There doesn't seem to be anything we can do
        // from here to cause libedit to move the cursor immediately. This will
        // break horribly if the user has rebound their keys, so for now we do
        // not permit user rebinding.
        ::el_push(EL, const_cast<char *>("\05\t"));

```
- **EN**: Implements logic around `empty`, `el_push`; this block emits or serializes data to an external representation.
- **CN**: 围绕 `empty`, `el_push` 实现具体逻辑；该代码块把数据输出或序列化为外部表示。

### Lines 165-174
```cpp
        // This assembles the output for the continuation block above.
        raw_string_ostream OS(Data->ContinuationOutput);

        // Move cursor to a blank line.
        OS << "\n";

        // Emit the completions.
        for (const std::string &Completion : Action.Completions)
          OS << Completion << "\n";

```
- **EN**: Implements logic around `OS`; this block emits or serializes data to an external representation.
- **CN**: 围绕 `OS` 实现具体逻辑；该代码块把数据输出或序列化为外部表示。

### Lines 175-184
```cpp
        // Fool libedit into thinking nothing has changed. Reprint its prompt
        // and the user input. Note that the cursor will remain at the end of
        // the line after this.
        OS << Data->LE->getPrompt()
           << StringRef(LI->buffer, LI->lastchar - LI->buffer);

        // This is the number of characters we need to tell libedit to go back:
        // the distance between end of line and the original cursor position.
        Data->PrevCount = LI->lastchar - LI->cursor;

```
- **EN**: Implements logic around `getPrompt`, `StringRef`.
- **CN**: 围绕 `getPrompt`, `StringRef` 实现具体逻辑。

### Lines 185-200
```cpp
        return CC_REFRESH;
      }
    }
  }
  return CC_ERROR;
}

} // end anonymous namespace

LineEditor::LineEditor(StringRef ProgName, StringRef HistoryPath, FILE *In,
                       FILE *Out, FILE *Err)
    : Prompt((ProgName + "> ").str()), HistoryPath(std::string(HistoryPath)),
      Data(new InternalData) {
  if (HistoryPath.empty())
    this->HistoryPath = getDefaultHistoryPath(ProgName);

```
- **EN**: Implements logic around `LineEditor`, `Prompt`, `Data`, `empty`, and 1 more symbols.
- **CN**: 围绕 `LineEditor`, `Prompt`, `Data`, `empty`, and 1 more symbols 实现具体逻辑。

### Lines 201-220
```cpp
  Data->LE = this;
  Data->Out = Out;

  Data->Hist = ::history_init();
  assert(Data->Hist);

  Data->EL = ::el_init(ProgName.str().c_str(), In, Out, Err);
  assert(Data->EL);

  ::el_set(Data->EL, EL_PROMPT, ElGetPromptFn);
  ::el_set(Data->EL, EL_EDITOR, "emacs");
  ::el_set(Data->EL, EL_HIST, history, Data->Hist);
  ::el_set(Data->EL, EL_ADDFN, "tab_complete", "Tab completion function",
           ElCompletionFn);
  ::el_set(Data->EL, EL_BIND, "\t", "tab_complete", NULL);
  ::el_set(Data->EL, EL_BIND, "^r", "em-inc-search-prev",
           NULL); // Cycle through backwards search, entering string
  ::el_set(Data->EL, EL_BIND, "^w", "ed-delete-prev-word",
           NULL); // Delete previous word, behave like bash does.
  ::el_set(Data->EL, EL_BIND, "\033[3~", "ed-delete-next-char",
```
- **EN**: Implements logic around `history_init`, `assert`, `el_init`, `el_set`.
- **CN**: 围绕 `history_init`, `assert`, `el_init`, `el_set` 实现具体逻辑。

### Lines 221-232
```cpp
           NULL); // Fix the delete key.
  ::el_set(Data->EL, EL_CLIENTDATA, Data.get());

  setHistorySize(DefaultHistorySize);
  HistEvent HE;
  ::history(Data->Hist, &HE, H_SETUNIQUE, 1);
  loadHistory();
}

LineEditor::~LineEditor() {
  saveHistory();

```
- **EN**: Implements logic around `el_set`, `setHistorySize`, `history`, `loadHistory`, and 2 more symbols.
- **CN**: 围绕 `el_set`, `setHistorySize`, `history`, `loadHistory`, and 2 more symbols 实现具体逻辑。

### Lines 233-244
```cpp
  ::history_end(Data->Hist);
  ::el_end(Data->EL);
  ::fwrite("\n", 1, 1, Data->Out);
}

void LineEditor::saveHistory() {
  if (!HistoryPath.empty()) {
    HistEvent HE;
    ::history(Data->Hist, &HE, H_SAVE, HistoryPath.c_str());
  }
}

```
- **EN**: Implements logic around `history_end`, `el_end`, `fwrite`, `saveHistory`, and 2 more symbols.
- **CN**: 围绕 `history_end`, `el_end`, `fwrite`, `saveHistory`, and 2 more symbols 实现具体逻辑。

### Lines 245-256
```cpp
void LineEditor::loadHistory() {
  if (!HistoryPath.empty()) {
    HistEvent HE;
    ::history(Data->Hist, &HE, H_LOAD, HistoryPath.c_str());
  }
}

void LineEditor::setHistorySize(int size) {
  HistEvent HE;
  ::history(Data->Hist, &HE, H_SETSIZE, size);
}

```
- **EN**: Implements logic around `loadHistory`, `empty`, `history`, `setHistorySize`.
- **CN**: 围绕 `loadHistory`, `empty`, `history`, `setHistorySize` 实现具体逻辑。

### Lines 257-270
```cpp
std::optional<std::string> LineEditor::readLine() const {
  // Call el_gets to prompt the user and read the user's input.
  int LineLen = 0;
  const char *Line = ::el_gets(Data->EL, &LineLen);

  // Either of these may mean end-of-file.
  if (!Line || LineLen == 0)
    return std::nullopt;

  // Strip any newlines off the end of the string.
  while (LineLen > 0 &&
         (Line[LineLen - 1] == '\n' || Line[LineLen - 1] == '\r'))
    --LineLen;

```
- **EN**: Implements logic around `readLine`, `el_gets`.
- **CN**: 围绕 `readLine`, `el_gets` 实现具体逻辑。

### Lines 271-281
```cpp
  HistEvent HE;
  if (LineLen > 0)
    ::history(Data->Hist, &HE, H_ENTER, Line);

  return std::string(Line, LineLen);
}

#else // HAVE_LIBEDIT

// Simple fgets-based implementation.

```
- **EN**: Implements logic around `history`, `string`.
- **CN**: 围绕 `history`, `string` 实现具体逻辑。

### Lines 282-293
```cpp
struct LineEditor::InternalData {
  FILE *In;
  FILE *Out;
};

LineEditor::LineEditor(StringRef ProgName, StringRef HistoryPath, FILE *In,
                       FILE *Out, FILE *Err)
    : Prompt((ProgName + "> ").str()), Data(new InternalData) {
  Data->In = In;
  Data->Out = Out;
}

```
- **EN**: Introduces declarations for `LineEditor::InternalData`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `LineEditor::InternalData` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 294-304
```cpp
LineEditor::~LineEditor() {
  ::fwrite("\n", 1, 1, Data->Out);
}

void LineEditor::saveHistory() {}
void LineEditor::loadHistory() {}
void LineEditor::setHistorySize(int size) {}

std::optional<std::string> LineEditor::readLine() const {
  ::fprintf(Data->Out, "%s", Prompt.c_str());

```
- **EN**: Implements logic around `~LineEditor`, `fwrite`, `saveHistory`, `loadHistory`, and 3 more symbols.
- **CN**: 围绕 `~LineEditor`, `fwrite`, `saveHistory`, `loadHistory`, and 3 more symbols 实现具体逻辑。

### Lines 305-318
```cpp
  std::string Line;
  do {
    char Buf[64];
    char *Res = ::fgets(Buf, sizeof(Buf), Data->In);
    if (!Res) {
      if (Line.empty())
        return std::nullopt;
      else
        return Line;
    }
    Line.append(Buf);
  } while (Line.empty() ||
           (Line[Line.size() - 1] != '\n' && Line[Line.size() - 1] != '\r'));

```
- **EN**: Implements logic around `fgets`, `empty`, `append`, `size`.
- **CN**: 围绕 `fgets`, `empty`, `append`, `size` 实现具体逻辑。

### Lines 319-326
```cpp
  while (!Line.empty() &&
         (Line[Line.size() - 1] == '\n' || Line[Line.size() - 1] == '\r'))
    Line.resize(Line.size() - 1);

  return Line;
}

#endif // HAVE_LIBEDIT
```
- **EN**: Implements logic around `empty`, `size`, `resize`.
- **CN**: 围绕 `empty`, `size`, `resize` 实现具体逻辑。

## Key Concepts / 关键概念

- **Interactive editing / 交互式编辑**:
  - **EN**: Supports prompt-driven line editing and history management.
  - **CN**: 支持基于提示符的行编辑与历史管理。
- **Streaming output / 流式输出**:
  - **EN**: Writes diagnostics, serialized data, or textual representations through LLVM stream APIs.
  - **CN**: 通过 LLVM 流式 API 输出诊断、序列化数据或文本表示。

## Dependencies / 依赖关系

- **Direct LLVM/local includes / 直接的 LLVM/本地包含**: `llvm/LineEditor/LineEditor.h`, `llvm/ADT/STLExtras.h`, `llvm/ADT/SmallString.h`, `llvm/Config/config.h`, `llvm/Support/Path.h`, `llvm/Support/raw_ostream.h`
- **Standard-library headers / 标准库头文件**: `<algorithm>`, `<cassert>`, `<cstdio>`, `<histedit.h>`
- **Subsystem categories / 子系统类别**: LLVM ADT containers and utility types / LLVM ADT 容器与工具类型 (2), support-library helpers / Support 库辅助功能 (2), LLVM build configuration details / LLVM 构建配置细节 (1)
