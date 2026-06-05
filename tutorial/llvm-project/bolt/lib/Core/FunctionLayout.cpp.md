# FunctionLayout.cpp — Code Analysis / 代码分析

## Source / 来源

- File / 文件: `bolt/lib/Core/FunctionLayout.cpp`
- Repository / 仓库: `llvm-project`
- Purpose / 用途: bolt/Core/FunctionLayout.cpp - Fragmented Function Layout -*- C++ -*. It also sits in code that implements BOLT core binary-analysis logic. / 该文件实现 BOLT 核心二进制分析逻辑。 源码头部说明其职责是：bolt/Core/FunctionLayout.cpp - Fragmented Function Layout -*- C++ -*。

## Line-by-Line Analysis / 逐行分析

### Lines 1-8

```cpp
//===- bolt/Core/FunctionLayout.cpp - Fragmented Function Layout -*- C++ -*-==//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 9-18

```cpp
#include "bolt/Core/FunctionLayout.h"
#include "bolt/Core/BinaryBasicBlock.h"
#include "llvm/ADT/STLExtras.h"
#include "llvm/ADT/edit_distance.h"
#include <algorithm>
#include <iterator>

using namespace llvm;
using namespace bolt;
```

- EN: Pulls in 6 header(s) from local project, LLVM, system dependencies needed by this range. Works inside namespace scope `llvm`, `bolt` to organize symbols. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `llvm`, `bolt`.
- CN: 这里引入 6 个头文件，为本段提供本地模块、LLVM 或系统层面的依赖。这里位于命名空间 `llvm`, `bolt` 中，用于组织符号作用域。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `llvm`, `bolt`。

### Lines 19-35

```cpp
FunctionFragment::FunctionFragment(FunctionLayout &Layout,
                                   const FragmentNum Num)
    : Layout(&Layout), Num(Num), StartIndex(Layout.block_size()) {}

FunctionFragment::iterator FunctionFragment::begin() {
  return iterator(Layout->block_begin() + StartIndex);
}
FunctionFragment::const_iterator FunctionFragment::begin() const {
  return const_iterator(Layout->block_begin() + StartIndex);
}
FunctionFragment::iterator FunctionFragment::end() {
  return iterator(Layout->block_begin() + StartIndex + Size);
}
FunctionFragment::const_iterator FunctionFragment::end() const {
  return const_iterator(Layout->block_begin() + StartIndex + Size);
}
```

- EN: Declares or implements routines including `Layout`, `begin`, `end`. Notable symbols here include `Layout`, `begin`, `end`.
- CN: 这里声明或实现函数，例如 `Layout`, `begin`, `end`。这里较值得关注的符号包括 `Layout`, `begin`, `end`。

### Lines 36-50

```cpp
BinaryBasicBlock *FunctionFragment::front() const { return *begin(); }

BinaryBasicBlock *FunctionFragment::back() const { return *std::prev(end()); }

FunctionLayout::FunctionLayout() { addFragment(); }

FunctionLayout::FunctionLayout(const FunctionLayout &Other)
    : Blocks(Other.Blocks) {
  for (FunctionFragment *const FF : Other.Fragments) {
    auto *Copy = new FunctionFragment(*FF);
    Copy->Layout = this;
    Fragments.emplace_back(Copy);
  }
}
```

- EN: Declares or implements routines including `front`, `back`, `FunctionLayout`, `Blocks`, `FunctionFragment`. Notable symbols here include `front`, `back`, `FunctionLayout`, `Blocks`, `FunctionFragment`.
- CN: 这里声明或实现函数，例如 `front`, `back`, `FunctionLayout`, `Blocks`, `FunctionFragment`。这里较值得关注的符号包括 `front`, `back`, `FunctionLayout`, `Blocks`, `FunctionFragment`。

### Lines 51-66

```cpp
FunctionLayout::FunctionLayout(FunctionLayout &&Other)
    : Fragments(std::move(Other.Fragments)), Blocks(std::move(Other.Blocks)) {
  for (FunctionFragment *const F : Fragments)
    F->Layout = this;
}

FunctionLayout &FunctionLayout::operator=(const FunctionLayout &Other) {
  Blocks = Other.Blocks;
  for (FunctionFragment *const FF : Other.Fragments) {
    auto *const Copy = new FunctionFragment(*FF);
    Copy->Layout = this;
    Fragments.emplace_back(Copy);
  }
  return *this;
}
```

- EN: Declares or implements routines including `FunctionLayout`, `Fragments`, `FunctionFragment`. Notable symbols here include `FunctionLayout`, `Fragments`, `FunctionFragment`.
- CN: 这里声明或实现函数，例如 `FunctionLayout`, `Fragments`, `FunctionFragment`。这里较值得关注的符号包括 `FunctionLayout`, `Fragments`, `FunctionFragment`。

### Lines 67-74

```cpp
FunctionLayout &FunctionLayout::operator=(FunctionLayout &&Other) {
  Fragments = std::move(Other.Fragments);
  Blocks = std::move(Other.Blocks);
  for (FunctionFragment *const FF : Fragments)
    FF->Layout = this;
  return *this;
}
```

- EN: Declares or implements routines including `move`. Notable symbols here include `move`.
- CN: 这里声明或实现函数，例如 `move`。这里较值得关注的符号包括 `move`。

### Lines 75-87

```cpp
FunctionLayout::~FunctionLayout() {
  for (FunctionFragment *const F : Fragments) {
    delete F;
  }
}

FunctionFragment &FunctionLayout::addFragment() {
  FunctionFragment *const FF =
      new FunctionFragment(*this, FragmentNum(Fragments.size()));
  Fragments.emplace_back(FF);
  return *FF;
}
```

- EN: Declares or implements routines including `FunctionLayout`, `addFragment`, `FunctionFragment`. Notable symbols here include `FunctionLayout`, `addFragment`, `FunctionFragment`.
- CN: 这里声明或实现函数，例如 `FunctionLayout`, `addFragment`, `FunctionFragment`。这里较值得关注的符号包括 `FunctionLayout`, `addFragment`, `FunctionFragment`。

### Lines 88-95

```cpp
FunctionFragment &FunctionLayout::getFragment(FragmentNum Num) {
  return *Fragments[Num.get()];
}

const FunctionFragment &FunctionLayout::getFragment(FragmentNum Num) const {
  return *Fragments[Num.get()];
}
```

- EN: Declares or implements routines including `getFragment`. Notable symbols here include `getFragment`.
- CN: 这里声明或实现函数，例如 `getFragment`。这里较值得关注的符号包括 `getFragment`。

### Lines 96-106

```cpp
const FunctionFragment &
FunctionLayout::findFragment(const BinaryBasicBlock *const BB) const {
  return getFragment(BB->getFragmentNum());
}

void FunctionLayout::addBasicBlock(BinaryBasicBlock *const BB) {
  BB->setLayoutIndex(Blocks.size());
  Blocks.emplace_back(BB);
  Fragments.back()->Size++;
}
```

- EN: Declares or implements routines including `findFragment`, `addBasicBlock`, `setLayoutIndex`. Notable symbols here include `findFragment`, `addBasicBlock`, `setLayoutIndex`.
- CN: 这里声明或实现函数，例如 `findFragment`, `addBasicBlock`, `setLayoutIndex`。这里较值得关注的符号包括 `findFragment`, `addBasicBlock`, `setLayoutIndex`。

### Lines 107-119

```cpp
void FunctionLayout::insertBasicBlocks(
    const BinaryBasicBlock *const InsertAfter,
    const ArrayRef<BinaryBasicBlock *> NewBlocks) {
  block_iterator InsertBeforePos = Blocks.begin();
  FragmentNum InsertFragmentNum = FragmentNum::main();
  unsigned LayoutIndex = 0;

  if (InsertAfter) {
    InsertBeforePos = std::next(findBasicBlockPos(InsertAfter));
    InsertFragmentNum = InsertAfter->getFragmentNum();
    LayoutIndex = InsertAfter->getLayoutIndex();
  }
```

- EN: Declares or implements routines including `main`, `next`, `getFragmentNum`, `getLayoutIndex`. Notable symbols here include `main`, `next`, `getFragmentNum`, `getLayoutIndex`.
- CN: 这里声明或实现函数，例如 `main`, `next`, `getFragmentNum`, `getLayoutIndex`。这里较值得关注的符号包括 `main`, `next`, `getFragmentNum`, `getLayoutIndex`。

### Lines 120-130

```cpp
  llvm::copy(NewBlocks, std::inserter(Blocks, InsertBeforePos));

  for (BinaryBasicBlock *const BB : NewBlocks) {
    BB->setFragmentNum(InsertFragmentNum);
    BB->setLayoutIndex(LayoutIndex++);
  }

  const fragment_iterator InsertFragment =
      fragment_begin() + InsertFragmentNum.get();
  InsertFragment->Size += NewBlocks.size();
```

- EN: Declares or implements routines including `copy`, `setFragmentNum`, `setLayoutIndex`, `fragment_begin`. Notable symbols here include `copy`, `setFragmentNum`, `setLayoutIndex`, `fragment_begin`.
- CN: 这里声明或实现函数，例如 `copy`, `setFragmentNum`, `setLayoutIndex`, `fragment_begin`。这里较值得关注的符号包括 `copy`, `setFragmentNum`, `setLayoutIndex`, `fragment_begin`。

### Lines 131-139

```cpp
  const fragment_iterator TailBegin = std::next(InsertFragment);
  auto const UpdateFragment = [&](FunctionFragment &FF) {
    FF.StartIndex += NewBlocks.size();
    for (BinaryBasicBlock *const BB : FF)
      BB->setLayoutIndex(LayoutIndex++);
  };
  std::for_each(TailBegin, fragment_end(), UpdateFragment);
}
```

- EN: Declares or implements routines including `next`, `setLayoutIndex`, `for_each`. Notable symbols here include `next`, `setLayoutIndex`, `for_each`.
- CN: 这里声明或实现函数，例如 `next`, `setLayoutIndex`, `for_each`。这里较值得关注的符号包括 `next`, `setLayoutIndex`, `for_each`。

### Lines 140-154

```cpp
void FunctionLayout::eraseBasicBlocks(
    const DenseSet<const BinaryBasicBlock *> ToErase) {
  const auto IsErased = [&](const BinaryBasicBlock *const BB) {
    return ToErase.contains(BB);
  };

  unsigned TotalErased = 0;
  for (FunctionFragment &FF : fragments()) {
    unsigned Erased = count_if(FF, IsErased);
    FF.Size -= Erased;
    FF.StartIndex -= TotalErased;
    TotalErased += Erased;
  }
  llvm::erase_if(Blocks, IsErased);
```

- EN: Declares or implements routines including `count_if`, `erase_if`. Notable symbols here include `count_if`, `erase_if`.
- CN: 这里声明或实现函数，例如 `count_if`, `erase_if`。这里较值得关注的符号包括 `count_if`, `erase_if`。

### Lines 155-165

```cpp
  // Remove empty fragments at the end
  const auto IsEmpty = [](const FunctionFragment *const FF) {
    return FF->empty();
  };
  const FragmentListType::iterator EmptyTailBegin =
      llvm::find_if_not(reverse(Fragments), IsEmpty).base();
  for (FunctionFragment *const FF :
       llvm::make_range(EmptyTailBegin, Fragments.end()))
    delete FF;
  Fragments.erase(EmptyTailBegin, Fragments.end());
```

- EN: Declares or implements routines including `find_if_not`, `make_range`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `find_if_not`, `make_range`.
- CN: 这里声明或实现函数，例如 `find_if_not`, `make_range`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `find_if_not`, `make_range`。

### Lines 166-183

```cpp
  updateLayoutIndices();
}

void FunctionLayout::updateLayoutIndices() const {
  unsigned BlockIndex = 0;
  for (const FunctionFragment &FF : fragments()) {
    for (BinaryBasicBlock *const BB : FF) {
      BB->setLayoutIndex(BlockIndex++);
      BB->setFragmentNum(FF.getFragmentNum());
    }
  }
}
void FunctionLayout::updateLayoutIndices(
    ArrayRef<BinaryBasicBlock *> Order) const {
  for (auto [Index, BB] : llvm::enumerate(Order))
    BB->setLayoutIndex(Index);
}
```

- EN: Declares or implements routines including `updateLayoutIndices`, `setLayoutIndex`, `setFragmentNum`. Notable symbols here include `updateLayoutIndices`, `setLayoutIndex`, `setFragmentNum`.
- CN: 这里声明或实现函数，例如 `updateLayoutIndices`, `setLayoutIndex`, `setFragmentNum`。这里较值得关注的符号包括 `updateLayoutIndices`, `setLayoutIndex`, `setFragmentNum`。

### Lines 184-196

```cpp
bool FunctionLayout::update(const ArrayRef<BinaryBasicBlock *> NewLayout) {
  const bool EqualBlockOrder = llvm::equal(Blocks, NewLayout);
  if (EqualBlockOrder) {
    const bool EqualPartitioning =
        llvm::all_of(fragments(), [](const FunctionFragment &FF) {
          return llvm::all_of(FF, [&](const BinaryBasicBlock *const BB) {
            return FF.Num == BB->getFragmentNum();
          });
        });
    if (EqualPartitioning)
      return false;
  }
```

- EN: Declares or implements routines including `update`, `equal`, `all_of`. Notable symbols here include `update`, `equal`, `all_of`.
- CN: 这里声明或实现函数，例如 `update`, `equal`, `all_of`。这里较值得关注的符号包括 `update`, `equal`, `all_of`。

### Lines 197-206

```cpp
  clear();

  // Generate fragments
  for (BinaryBasicBlock *const BB : NewLayout) {
    FragmentNum Num = BB->getFragmentNum();

    // Add empty fragments if necessary
    while (Fragments.back()->getFragmentNum() < Num)
      addFragment();
```

- EN: Declares or implements routines including `clear`, `getFragmentNum`, `addFragment`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `clear`, `getFragmentNum`, `addFragment`.
- CN: 这里声明或实现函数，例如 `clear`, `getFragmentNum`, `addFragment`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `clear`, `getFragmentNum`, `addFragment`。

### Lines 207-224

```cpp
    // Set the next fragment to point one past the current BB
    addBasicBlock(BB);
  }

  return true;
}

void FunctionLayout::clear() {
  Blocks = BasicBlockListType();
  // If the binary does not have relocations and is not split, the function will
  // be written to the output stream at its original file offset (see
  // `RewriteInstance::rewriteFile`). Hence, when the layout is cleared, retain
  // the main fragment, so that this information is not lost.
  for (FunctionFragment *const FF : llvm::drop_begin(Fragments))
    delete FF;
  Fragments = FragmentListType{Fragments.front()};
  getMainFragment().Size = 0;
}
```

- EN: Declares or implements routines including `addBasicBlock`, `clear`, `BasicBlockListType`, `getMainFragment`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `addBasicBlock`, `clear`, `BasicBlockListType`, `getMainFragment`.
- CN: 这里声明或实现函数，例如 `addBasicBlock`, `clear`, `BasicBlockListType`, `getMainFragment`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `addBasicBlock`, `clear`, `BasicBlockListType`, `getMainFragment`。

### Lines 225-232

```cpp

const BinaryBasicBlock *
FunctionLayout::getBasicBlockAfter(block_const_iterator BBIter,
                                   bool IgnoreSplits) const {
  const block_const_iterator BlockAfter = std::next(BBIter);
  if (BlockAfter == block_end())
    return nullptr;
```

- EN: Declares or implements routines including `next`. Notable symbols here include `next`.
- CN: 这里声明或实现函数，例如 `next`。这里较值得关注的符号包括 `next`。

### Lines 233-246

```cpp
  if (!IgnoreSplits)
    if (BlockAfter == getFragment((*BBIter)->getFragmentNum()).end())
      return nullptr;

  return *BlockAfter;
}

const BinaryBasicBlock *
FunctionLayout::getBasicBlockAfter(const BinaryBasicBlock *BB,
                                   bool IgnoreSplits) const {
  const block_const_iterator BBPos = find(blocks(), BB);
  if (BBPos == block_end())
    return nullptr;
```

- EN: Declares or implements routines including `find`. Notable symbols here include `find`.
- CN: 这里声明或实现函数，例如 `find`。这里较值得关注的符号包括 `find`。

### Lines 247-255

```cpp
  return getBasicBlockAfter(BBPos, IgnoreSplits);
}

bool FunctionLayout::isSplit() const {
  const unsigned NonEmptyFragCount = llvm::count_if(
      fragments(), [](const FunctionFragment &FF) { return !FF.empty(); });
  return NonEmptyFragCount >= 2;
}
```

- EN: Declares or implements routines including `isSplit`, `fragments`. Notable symbols here include `isSplit`, `fragments`.
- CN: 这里声明或实现函数，例如 `isSplit`, `fragments`。这里较值得关注的符号包括 `isSplit`, `fragments`。

### Lines 256-265

```cpp
uint64_t FunctionLayout::getEditDistance(
    const ArrayRef<const BinaryBasicBlock *> OldBlockOrder) const {
  return ComputeEditDistance<const BinaryBasicBlock *>(OldBlockOrder, Blocks);
}

FunctionLayout::block_const_iterator
FunctionLayout::findBasicBlockPos(const BinaryBasicBlock *BB) const {
  return block_const_iterator(find(Blocks, BB));
}
```

- EN: Declares or implements routines including `findBasicBlockPos`. Notable symbols here include `findBasicBlockPos`.
- CN: 这里声明或实现函数，例如 `findBasicBlockPos`。这里较值得关注的符号包括 `findBasicBlockPos`。

### Lines 266-269

```cpp
FunctionLayout::block_iterator
FunctionLayout::findBasicBlockPos(const BinaryBasicBlock *BB) {
  return find(Blocks, BB);
}
```

- EN: Declares or implements routines including `findBasicBlockPos`. Notable symbols here include `findBasicBlockPos`.
- CN: 这里声明或实现函数，例如 `findBasicBlockPos`。这里较值得关注的符号包括 `findBasicBlockPos`。

## Key Concepts / 关键概念

- `Layout`: function or method entry point / 函数或方法入口
- `begin`: function or method entry point / 函数或方法入口
- `end`: function or method entry point / 函数或方法入口
- `front`: function or method entry point / 函数或方法入口
- `back`: function or method entry point / 函数或方法入口
- `llvm`: namespace scope / 命名空间作用域
- `bolt`: namespace scope / 命名空间作用域

## Dependencies / 依赖关系

- Local headers / 本地头文件: `bolt/Core/FunctionLayout.h`, `bolt/Core/BinaryBasicBlock.h`
- LLVM headers / LLVM 头文件: `llvm/ADT/STLExtras.h`, `llvm/ADT/edit_distance.h`
- System headers / 系统头文件: `algorithm`, `iterator`
- Directory context / 目录上下文: `bolt/lib/Core` neighbors usually cooperate with this file to provide the surrounding subsystem / `bolt/lib/Core` 下的相邻文件通常与本文件协作组成对应子系统
