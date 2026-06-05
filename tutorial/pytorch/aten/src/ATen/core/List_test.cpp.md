# List_test.cpp — Code Analysis / 代码分析
## Source / 来源
- File: `aten/src/ATen/core/List_test.cpp`
- Repository: `pytorch/pytorch`
- Purpose (EN): This file implements core ATen/C10 abstractions, with primary focus on `c10`, `TEST`, `EXPECT_TRUE`.
- 用途（中文）: 该文件实现ATen/C10 核心抽象，核心关注对象是 `c10`, `TEST`, `EXPECT_TRUE`。

## Line-by-Line Analysis / 逐行分析

### Lines 1-24
```cpp
#include <ATen/core/List.h>
#include <gtest/gtest.h>

using namespace c10;
using std::string;

// TODO(NS): Remove me
// For some reason emplace tests in this file fails to compile with C++20 with the following warning
// aten/src/ATen/core/ivalue.h:240:3: error: array subscript 0 is outside array bounds of 'c10::IValue [0]'
#if defined(__GNUC__) && __GNUC__ == 13
#pragma GCC diagnostic ignored "-Warray-bounds"
#endif

// NOLINTBEGIN(performance-move-const-arg, bugprone-use-after-move, *analyzer*Move)
TEST(ListTestIValueBasedList, givenEmptyList_whenCallingEmpty_thenReturnsTrue) {
    List<string> list;
    EXPECT_TRUE(list.empty());
}

TEST(ListTestIValueBasedList, givenNonemptyList_whenCallingEmpty_thenReturnsFalse) {
    List<string> list({"3"});
    EXPECT_FALSE(list.empty());
}

```
- EN: Focus symbols: `c10`, `TEST`, `EXPECT_TRUE`, `empty`, `list`. This block sets up header dependencies and the initial compilation context. The file brings required declarations into scope before entering the main ATen logic.
- CN: 关注符号：`c10`, `TEST`, `EXPECT_TRUE`, `empty`, `list`。该代码块设置头文件依赖与初始编译上下文。文件在进入主要 ATen 逻辑前先引入所需声明。

### Lines 25-51
```cpp
TEST(ListTestIValueBasedList, givenEmptyList_whenCallingSize_thenReturnsZero) {
    List<string> list;
    EXPECT_EQ(0, list.size());
}

TEST(ListTestIValueBasedList, givenNonemptyList_whenCallingSize_thenReturnsNumberOfElements) {
    List<string> list({"3", "4"});
    EXPECT_EQ(2, list.size());
}

TEST(ListTestIValueBasedList, givenNonemptyList_whenCallingClear_thenIsEmpty) {
  List<string> list({"3", "4"});
  list.clear();
  EXPECT_TRUE(list.empty());
}

TEST(ListTestIValueBasedList, whenCallingGetWithExistingPosition_thenReturnsElement) {
  List<string> list({"3", "4"});
  EXPECT_EQ("3", list.get(0));
  EXPECT_EQ("4", list.get(1));
}

TEST(ListTestIValueBasedList, whenCallingGetWithNonExistingPosition_thenThrowsException) {
  List<string> list({"3", "4"});
  EXPECT_THROW(list.get(2), std::out_of_range);
}

```
- EN: Focus symbols: `TEST`, `EXPECT_EQ`, `size`, `list`, `clear`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`TEST`, `EXPECT_EQ`, `size`, `list`, `clear`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

### Lines 52-76
```cpp
TEST(ListTestIValueBasedList, whenCallingExtractWithExistingPosition_thenReturnsElement) {
  List<string> list({"3", "4"});
  EXPECT_EQ("3", list.extract(0));
  EXPECT_EQ("4", list.extract(1));
}

TEST(ListTestIValueBasedList, whenCallingExtractWithExistingPosition_thenListElementBecomesInvalid) {
  List<string> list({"3", "4"});
  list.extract(0);
  EXPECT_EQ("", list.get(0));
}

TEST(ListTestIValueBasedList, whenCallingExtractWithNonExistingPosition_thenThrowsException) {
  List<string> list({"3", "4"});
  EXPECT_THROW(list.extract(2), std::out_of_range);
}

TEST(ListTestIValueBasedList, whenCallingCopyingSetWithExistingPosition_thenChangesElement) {
  List<string> list({"3", "4"});
  string value = "5";
  list.set(1, value);
  EXPECT_EQ("3", list.get(0));
  EXPECT_EQ("5", list.get(1));
}

```
- EN: Focus symbols: `TEST`, `list`, `EXPECT_EQ`, `extract`, `get`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`TEST`, `list`, `EXPECT_EQ`, `extract`, `get`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

### Lines 77-102
```cpp
TEST(ListTestIValueBasedList, whenCallingMovingSetWithExistingPosition_thenChangesElement) {
  List<string> list({"3", "4"});
  string value = "5";
  list.set(1, std::move(value));
  EXPECT_EQ("3", list.get(0));
  EXPECT_EQ("5", list.get(1));
}

TEST(ListTestIValueBasedList, whenCallingCopyingSetWithNonExistingPosition_thenThrowsException) {
  List<string> list({"3", "4"});
  string value = "5";
  EXPECT_THROW(list.set(2, value), std::out_of_range);
}

TEST(ListTestIValueBasedList, whenCallingMovingSetWithNonExistingPosition_thenThrowsException) {
  List<string> list({"3", "4"});
  string value = "5";
  EXPECT_THROW(list.set(2, std::move(value)), std::out_of_range);
}

TEST(ListTestIValueBasedList, whenCallingAccessOperatorWithExistingPosition_thenReturnsElement) {
  List<string> list({"3", "4"});
  EXPECT_EQ("3", static_cast<string>(list[0]));
  EXPECT_EQ("4", static_cast<string>(list[1]));
}

```
- EN: Focus symbols: `TEST`, `list`, `set`, `move`, `EXPECT_EQ`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`TEST`, `list`, `set`, `move`, `EXPECT_EQ`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

### Lines 103-126
```cpp
TEST(ListTestIValueBasedList, whenAssigningToAccessOperatorWithExistingPosition_thenSetsElement) {
  List<string> list({"3", "4", "5"});
  list[1] = "6";
  EXPECT_EQ("3", list.get(0));
  EXPECT_EQ("6", list.get(1));
  EXPECT_EQ("5", list.get(2));
}

TEST(ListTestIValueBasedList, whenAssigningToAccessOperatorFromAccessOperator_thenSetsElement) {
  List<string> list({"3", "4", "5"});
  list[1] = list[2];
  EXPECT_EQ("3", list.get(0));
  EXPECT_EQ("5", list.get(1));
  EXPECT_EQ("5", list.get(2));
}

TEST(ListTestIValueBasedList, whenSwappingFromAccessOperator_thenSwapsElements) {
  List<string> list({"3", "4", "5"});
  swap(list[1], list[2]);
  EXPECT_EQ("3", list.get(0));
  EXPECT_EQ("5", list.get(1));
  EXPECT_EQ("4", list.get(2));
}

```
- EN: Focus symbols: `TEST`, `list`, `EXPECT_EQ`, `get`, `swap`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`TEST`, `list`, `EXPECT_EQ`, `get`, `swap`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

### Lines 127-154
```cpp
TEST(ListTestIValueBasedList, whenCallingAccessOperatorWithNonExistingPosition_thenThrowsException) {
  List<string> list({"3", "4"});
  EXPECT_THROW(list[2], std::out_of_range);
}

TEST(ListTestIValueBasedList, whenCallingInsertOnIteratorWithLValue_thenInsertsElement) {
  List<string> list({"3", "4", "6"});
  string v = "5";
  list.insert(list.begin() + 2, v);
  EXPECT_EQ(4, list.size());
  EXPECT_EQ("5", list.get(2));
}

TEST(ListTestIValueBasedList, whenCallingInsertOnIteratorWithRValue_thenInsertsElement) {
  List<string> list({"3", "4", "6"});
  string v = "5";
  list.insert(list.begin() + 2, std::move(v));
  EXPECT_EQ(4, list.size());
  EXPECT_EQ("5", list.get(2));
}

TEST(ListTestIValueBasedList, whenCallingInsertWithLValue_thenReturnsIteratorToNewElement) {
  List<string> list({"3", "4", "6"});
  string v = "5";
  List<string>::iterator result = list.insert(list.begin() + 2, v);
  EXPECT_EQ(list.begin() + 2, result);
}

```
- EN: Focus symbols: `TEST`, `list`, `EXPECT_THROW`, `insert`, `begin`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`TEST`, `list`, `EXPECT_THROW`, `insert`, `begin`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

### Lines 155-178
```cpp
TEST(ListTestIValueBasedList, whenCallingInsertWithRValue_thenReturnsIteratorToNewElement) {
  List<string> list({"3", "4", "6"});
  string v = "5";
  List<string>::iterator result = list.insert(list.begin() + 2, std::move(v));
  EXPECT_EQ(list.begin() + 2, result);
}

TEST(ListTestIValueBasedList, whenCallingEmplaceWithLValue_thenInsertsElement) {
  List<string> list({"3", "4", "6"});
  string v = "5";
  list.emplace(list.begin() + 2, v);
  EXPECT_EQ(4, list.size());
  EXPECT_EQ("5", list.get(2));
}

TEST(ListTestIValueBasedList, whenCallingEmplaceWithRValue_thenInsertsElement) {
  List<string> list({"3", "4", "6"});
  string v = "5";
  list.emplace(list.begin() + 2, std::move(v));
  EXPECT_EQ(4, list.size());
  EXPECT_EQ("5", list.get(2));
}

TEST(ListTestIValueBasedList, whenCallingEmplaceWithConstructorArg_thenInsertsElement) {
```
- EN: Focus symbols: `TEST`, `list`, `insert`, `begin`, `move`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`TEST`, `list`, `insert`, `begin`, `move`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

### Lines 179-202
```cpp
  List<string> list({"3", "4", "6"});
  list.emplace(list.begin() + 2, "5"); // const char* is a constructor arg to std::string
  EXPECT_EQ(4, list.size());
  EXPECT_EQ("5", list.get(2));
}

TEST(ListTestIValueBasedList, whenCallingPushBackWithLValue_ThenInsertsElement) {
  List<string> list;
  string v = "5";
  list.push_back(v);
  EXPECT_EQ(1, list.size());
  EXPECT_EQ("5", list.get(0));
}

TEST(ListTestIValueBasedList, whenCallingPushBackWithRValue_ThenInsertsElement) {
  List<string> list;
  string v = "5";
  list.push_back(std::move(v));
  EXPECT_EQ(1, list.size());
  EXPECT_EQ("5", list.get(0));
}

TEST(ListTestIValueBasedList, whenCallingEmplaceBackWithLValue_ThenInsertsElement) {
  List<string> list;
```
- EN: Focus symbols: `list`, `emplace`, `begin`, `EXPECT_EQ`, `size`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`list`, `emplace`, `begin`, `EXPECT_EQ`, `size`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

### Lines 203-226
```cpp
  string v = "5";
  list.emplace_back(v);
  EXPECT_EQ(1, list.size());
  EXPECT_EQ("5", list.get(0));
}

TEST(ListTestIValueBasedList, whenCallingEmplaceBackWithRValue_ThenInsertsElement) {
  List<string> list;
  string v = "5";
  list.emplace_back(std::move(v));
  EXPECT_EQ(1, list.size());
  EXPECT_EQ("5", list.get(0));
}

TEST(ListTestIValueBasedList, whenCallingEmplaceBackWithConstructorArg_ThenInsertsElement) {
  List<string> list;
  list.emplace_back("5");  // const char* is a constructor arg to std::string
  EXPECT_EQ(1, list.size());
  EXPECT_EQ("5", list.get(0));
}

TEST(ListTestIValueBasedList, givenEmptyList_whenIterating_thenBeginIsEnd) {
  List<string> list;
  const List<string> clist;
```
- EN: Focus symbols: `emplace_back`, `EXPECT_EQ`, `size`, `get`, `TEST`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`emplace_back`, `EXPECT_EQ`, `size`, `get`, `TEST`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

### Lines 227-251
```cpp
  EXPECT_EQ(list.begin(), list.end());
  EXPECT_EQ(list.begin(), list.end());
  EXPECT_EQ(clist.begin(), clist.end());
  EXPECT_EQ(clist.begin(), clist.end());
}

TEST(ListTestIValueBasedList, whenIterating_thenFindsElements) {
  List<string> list({"3", "5"});
  bool found_first = false;
  bool found_second = false;
  for (const auto && iter : list) {
    if (static_cast<string>(iter) == "3") {
      EXPECT_FALSE(found_first);
      found_first = true;
    } else if (static_cast<string>(iter) == "5") {
      EXPECT_FALSE(found_second);
      found_second = true;
    } else {
      ADD_FAILURE();
    }
  }
  EXPECT_TRUE(found_first);
  EXPECT_TRUE(found_second);
}

```
- EN: Focus symbols: `EXPECT_EQ`, `begin`, `end`, `TEST`, `list`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`EXPECT_EQ`, `begin`, `end`, `TEST`, `list`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

### Lines 252-277
```cpp
TEST(ListTestIValueBasedList, whenIteratingWithForeach_thenFindsElements) {
  List<string> list({"3", "5"});
  bool found_first = false;
  bool found_second = false;
  // NOLINTNEXTLINE(performance-implicit-conversion-in-loop)
  for (const string& elem : list) {
    if (elem == "3") {
      EXPECT_FALSE(found_first);
      found_first = true;
    } else if (elem == "5") {
      EXPECT_FALSE(found_second);
      found_second = true;
    } else {
      ADD_FAILURE();
    }
  }
  EXPECT_TRUE(found_first);
  EXPECT_TRUE(found_second);
}

TEST(ListTestIValueBasedList, givenOneElementList_whenErasing_thenListIsEmpty) {
  List<string> list({"3"});
  list.erase(list.begin());
  EXPECT_TRUE(list.empty());
}

```
- EN: Focus symbols: `TEST`, `list`, `EXPECT_FALSE`, `ADD_FAILURE`, `EXPECT_TRUE`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`TEST`, `list`, `EXPECT_FALSE`, `ADD_FAILURE`, `EXPECT_TRUE`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

### Lines 278-305
```cpp
TEST(ListTestIValueBasedList, givenList_whenErasing_thenReturnsIterator) {
  List<string> list({"1", "2", "3"});
  List<string>::iterator iter = list.erase(list.begin() + 1);
  EXPECT_EQ(list.begin() + 1, iter);
}

TEST(ListTestIValueBasedList, givenList_whenErasingFullRange_thenIsEmpty) {
  List<string> list({"1", "2", "3"});
  list.erase(list.begin(), list.end());
  EXPECT_TRUE(list.empty());
}

TEST(ListTestIValueBasedList, whenCallingReserve_thenDoesntCrash) {
  List<string> list;
  list.reserve(100);
}

TEST(ListTestIValueBasedList, whenCopyConstructingList_thenAreEqual) {
  List<string> list1({"3", "4"});

  // NOLINTNEXTLINE(performance-unnecessary-copy-initialization)
  List<string> list2(list1);

  EXPECT_EQ(2, list2.size());
  EXPECT_EQ("3", list2.get(0));
  EXPECT_EQ("4", list2.get(1));
}

```
- EN: Focus symbols: `TEST`, `list`, `erase`, `begin`, `EXPECT_EQ`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`TEST`, `list`, `erase`, `begin`, `EXPECT_EQ`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

### Lines 306-331
```cpp
TEST(ListTestIValueBasedList, whenCopyAssigningList_thenAreEqual) {
  List<string> list1({"3", "4"});

  List<string> list2;
  list2 = list1;

  EXPECT_EQ(2, list2.size());
  EXPECT_EQ("3", list2.get(0));
  EXPECT_EQ("4", list2.get(1));
}

TEST(ListTestIValueBasedList, whenCopyingList_thenAreEqual) {
  List<string> list1({"3", "4"});

  List<string> list2 = list1.copy();

  EXPECT_EQ(2, list2.size());
  EXPECT_EQ("3", list2.get(0));
  EXPECT_EQ("4", list2.get(1));
}

TEST(ListTestIValueBasedList, whenMoveConstructingList_thenNewIsCorrect) {
  List<string> list1({"3", "4"});

  List<string> list2(std::move(list1));

```
- EN: Focus symbols: `TEST`, `list1`, `EXPECT_EQ`, `size`, `get`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`TEST`, `list1`, `EXPECT_EQ`, `size`, `get`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

### Lines 332-356
```cpp
  EXPECT_EQ(2, list2.size());
  EXPECT_EQ("3", list2.get(0));
  EXPECT_EQ("4", list2.get(1));
}

TEST(ListTestIValueBasedList, whenMoveAssigningList_thenNewIsCorrect) {
  List<string> list1({"3", "4"});

  List<string> list2;
  list2 = std::move(list1);

  EXPECT_EQ(2, list2.size());
  EXPECT_EQ("3", list2.get(0));
  EXPECT_EQ("4", list2.get(1));
}

TEST(ListTestIValueBasedList, whenMoveConstructingList_thenOldIsUnchanged) {
  List<string> list1({"3", "4"});

  List<string> list2(std::move(list1));
  EXPECT_EQ(2, list1.size());
  EXPECT_EQ("3", list1.get(0));
  EXPECT_EQ("4", list1.get(1));
}

```
- EN: Focus symbols: `EXPECT_EQ`, `size`, `get`, `TEST`, `list1`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`EXPECT_EQ`, `size`, `get`, `TEST`, `list1`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

### Lines 357-384
```cpp
TEST(ListTestIValueBasedList, whenMoveAssigningList_thenOldIsUnchanged) {
  List<string> list1({"3", "4"});

  List<string> list2;
  list2 = std::move(list1);
  EXPECT_EQ(2, list1.size());
  EXPECT_EQ("3", list1.get(0));
  EXPECT_EQ("4", list1.get(1));
}

TEST(ListTestIValueBasedList, givenIterator_whenPostfixIncrementing_thenMovesToNextAndReturnsOldPosition) {
  List<string> list({"3", "4"});

  List<string>::iterator iter1 = list.begin();
  List<string>::iterator iter2 = iter1++;
  EXPECT_NE("3", static_cast<string>(*iter1));
  EXPECT_EQ("3", static_cast<string>(*iter2));
}

TEST(ListTestIValueBasedList, givenIterator_whenPrefixIncrementing_thenMovesToNextAndReturnsNewPosition) {
  List<string> list({"3", "4"});

  List<string>::iterator iter1 = list.begin();
  List<string>::iterator iter2 = ++iter1;
  EXPECT_NE("3", static_cast<string>(*iter1));
  EXPECT_NE("3", static_cast<string>(*iter2));
}

```
- EN: Focus symbols: `TEST`, `list1`, `move`, `EXPECT_EQ`, `size`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`TEST`, `list1`, `move`, `EXPECT_EQ`, `size`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

### Lines 385-411
```cpp
TEST(ListTestIValueBasedList, givenIterator_whenPostfixDecrementing_thenMovesToNextAndReturnsOldPosition) {
  List<string> list({"3", "4"});

  List<string>::iterator iter1 = list.end() - 1;
  List<string>::iterator iter2 = iter1--;
  EXPECT_NE("4", static_cast<string>(*iter1));
  EXPECT_EQ("4", static_cast<string>(*iter2));
}

TEST(ListTestIValueBasedList, givenIterator_whenPrefixDecrementing_thenMovesToNextAndReturnsNewPosition) {
  List<string> list({"3", "4"});

  List<string>::iterator iter1 = list.end() - 1;
  List<string>::iterator iter2 = --iter1;
  EXPECT_NE("4", static_cast<string>(*iter1));
  EXPECT_NE("4", static_cast<string>(*iter2));
}

TEST(ListTestIValueBasedList, givenIterator_whenIncreasing_thenMovesToNextAndReturnsNewPosition) {
  List<string> list({"3", "4", "5"});

  List<string>::iterator iter1 = list.begin();
  List<string>::iterator iter2 = iter1 += 2;
  EXPECT_EQ("5", static_cast<string>(*iter1));
  EXPECT_EQ("5", static_cast<string>(*iter2));
}

```
- EN: Focus symbols: `TEST`, `list`, `end`, `EXPECT_NE`, `EXPECT_EQ`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`TEST`, `list`, `end`, `EXPECT_NE`, `EXPECT_EQ`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

### Lines 412-438
```cpp
TEST(ListTestIValueBasedList, givenIterator_whenDecreasing_thenMovesToNextAndReturnsNewPosition) {
  List<string> list({"3", "4", "5"});

  List<string>::iterator iter1 = list.end();
  List<string>::iterator iter2 = iter1 -= 2;
  EXPECT_EQ("4", static_cast<string>(*iter1));
  EXPECT_EQ("4", static_cast<string>(*iter2));
}

TEST(ListTestIValueBasedList, givenIterator_whenAdding_thenReturnsNewIterator) {
  List<string> list({"3", "4", "5"});

  List<string>::iterator iter1 = list.begin();
  List<string>::iterator iter2 = iter1 + 2;
  EXPECT_EQ("3", static_cast<string>(*iter1));
  EXPECT_EQ("5", static_cast<string>(*iter2));
}

TEST(ListTestIValueBasedList, givenIterator_whenSubtracting_thenReturnsNewIterator) {
  List<string> list({"3", "4", "5"});

  List<string>::iterator iter1 = list.end() - 1;
  List<string>::iterator iter2 = iter1 - 2;
  EXPECT_EQ("5", static_cast<string>(*iter1));
  EXPECT_EQ("3", static_cast<string>(*iter2));
}

```
- EN: Focus symbols: `TEST`, `list`, `end`, `EXPECT_EQ`, `begin`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`TEST`, `list`, `end`, `EXPECT_EQ`, `begin`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

### Lines 439-463
```cpp
TEST(ListTestIValueBasedList, givenIterator_whenCalculatingDifference_thenReturnsCorrectNumber) {
  List<string> list({"3", "4"});
  EXPECT_EQ(2, list.end() - list.begin());
}

TEST(ListTestIValueBasedList, givenEqualIterators_thenAreEqual) {
  List<string> list({"3", "4"});

  List<string>::iterator iter1 = list.begin();
  List<string>::iterator iter2 = list.begin();
  EXPECT_TRUE(iter1 == iter2);
  EXPECT_FALSE(iter1 != iter2);
}

TEST(ListTestIValueBasedList, givenDifferentIterators_thenAreNotEqual) {
  List<string> list({"3", "4"});

  List<string>::iterator iter1 = list.begin();
  List<string>::iterator iter2 = list.begin();
  iter2++;

  EXPECT_FALSE(iter1 == iter2);
  EXPECT_TRUE(iter1 != iter2);
}

```
- EN: Focus symbols: `TEST`, `list`, `EXPECT_EQ`, `end`, `begin`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`TEST`, `list`, `EXPECT_EQ`, `end`, `begin`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

### Lines 464-490
```cpp
TEST(ListTestIValueBasedList, givenIterator_whenDereferencing_thenPointsToCorrectElement) {
  List<string> list({"3"});

  List<string>::iterator iter = list.begin();
  EXPECT_EQ("3", static_cast<string>(*iter));
}

TEST(ListTestIValueBasedList, givenIterator_whenAssigningNewValue_thenChangesValue) {
  List<string> list({"3"});

  List<string>::iterator iter = list.begin();
  *iter = "4";
  EXPECT_EQ("4", list.get(0));
}

TEST(ListTestIValueBasedList, givenIterator_whenAssigningNewValueFromIterator_thenChangesValue) {
  List<string> list({"3", "4"});

  List<string>::iterator iter = list.begin();
  *iter = *(iter + 1);
  EXPECT_EQ("4", list.get(0));
  EXPECT_EQ("4", list.get(1));
}

TEST(ListTestIValueBasedList, givenIterator_whenSwappingValuesFromIterator_thenChangesValue) {
  List<string> list({"3", "4"});

```
- EN: Focus symbols: `TEST`, `list`, `begin`, `EXPECT_EQ`, `get`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`TEST`, `list`, `begin`, `EXPECT_EQ`, `get`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

### Lines 491-518
```cpp
  List<string>::iterator iter = list.begin();
  swap(*iter, *(iter + 1));
  EXPECT_EQ("4", list.get(0));
  EXPECT_EQ("3", list.get(1));
}

TEST(ListTestIValueBasedList, givenOneElementList_whenCallingPopBack_thenIsEmpty) {
  List<string> list({"3"});
  list.pop_back();
  EXPECT_TRUE(list.empty());
}

TEST(ListTestIValueBasedList, givenEmptyList_whenCallingResize_thenResizesAndSetsEmptyValue) {
  List<string> list;
  list.resize(2);
  EXPECT_EQ(2, list.size());
  EXPECT_EQ("", list.get(0));
  EXPECT_EQ("", list.get(1));
}

TEST(ListTestIValueBasedList, givenEmptyList_whenCallingResizeWithValue_thenResizesAndSetsValue) {
  List<string> list;
  list.resize(2, "value");
  EXPECT_EQ(2, list.size());
  EXPECT_EQ("value", list.get(0));
  EXPECT_EQ("value", list.get(1));
}

```
- EN: Focus symbols: `begin`, `swap`, `EXPECT_EQ`, `get`, `TEST`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`begin`, `swap`, `EXPECT_EQ`, `get`, `TEST`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

### Lines 519-543
```cpp
TEST(ListTestIValueBasedList, isReferenceType) {
  List<string> list1;
  // NOLINTNEXTLINE(performance-unnecessary-copy-initialization)
  List<string> list2(list1);
  List<string> list3;
  list3 = list1;

  list1.push_back("three");
  EXPECT_EQ(1, list1.size());
  EXPECT_EQ(1, list2.size());
  EXPECT_EQ(1, list3.size());
}

TEST(ListTestIValueBasedList, copyHasSeparateStorage) {
  List<string> list1;
  List<string> list2(list1.copy());
  List<string> list3;
  list3 = list1.copy();

  list1.push_back("three");
  EXPECT_EQ(1, list1.size());
  EXPECT_EQ(0, list2.size());
  EXPECT_EQ(0, list3.size());
}

```
- EN: Focus symbols: `TEST`, `list2`, `push_back`, `EXPECT_EQ`, `size`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`TEST`, `list2`, `push_back`, `EXPECT_EQ`, `size`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

### Lines 544-567
```cpp
TEST(ListTestIValueBasedList, givenEqualLists_thenIsEqual) {
  List<string> list1({"first", "second"});
  List<string> list2({"first", "second"});

  EXPECT_EQ(list1, list2);
}

TEST(ListTestIValueBasedList, givenDifferentLists_thenIsNotEqual) {
  List<string> list1({"first", "second"});
  List<string> list2({"first", "not_second"});

  EXPECT_NE(list1, list2);
}

TEST(ListTestNonIValueBasedList, givenEmptyList_whenCallingEmpty_thenReturnsTrue) {
    List<int64_t> list;
    EXPECT_TRUE(list.empty());
}

TEST(ListTestNonIValueBasedList, givenNonemptyList_whenCallingEmpty_thenReturnsFalse) {
    List<int64_t> list({3});
    EXPECT_FALSE(list.empty());
}

```
- EN: Focus symbols: `TEST`, `list1`, `list2`, `EXPECT_EQ`, `EXPECT_NE`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`TEST`, `list1`, `list2`, `EXPECT_EQ`, `EXPECT_NE`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

### Lines 568-594
```cpp
TEST(ListTestNonIValueBasedList, givenEmptyList_whenCallingSize_thenReturnsZero) {
    List<int64_t> list;
    EXPECT_EQ(0, list.size());
}

TEST(ListTestNonIValueBasedList, givenNonemptyList_whenCallingSize_thenReturnsNumberOfElements) {
    List<int64_t> list({3, 4});
    EXPECT_EQ(2, list.size());
}

TEST(ListTestNonIValueBasedList, givenNonemptyList_whenCallingClear_thenIsEmpty) {
  List<int64_t> list({3, 4});
  list.clear();
  EXPECT_TRUE(list.empty());
}

TEST(ListTestNonIValueBasedList, whenCallingGetWithExistingPosition_thenReturnsElement) {
  List<int64_t> list({3, 4});
  EXPECT_EQ(3, list.get(0));
  EXPECT_EQ(4, list.get(1));
}

TEST(ListTestNonIValueBasedList, whenCallingGetWithNonExistingPosition_thenThrowsException) {
  List<int64_t> list({3, 4});
  EXPECT_THROW(list.get(2), std::out_of_range);
}

```
- EN: Focus symbols: `TEST`, `EXPECT_EQ`, `size`, `list`, `clear`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`TEST`, `EXPECT_EQ`, `size`, `list`, `clear`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

### Lines 595-622
```cpp
TEST(ListTestNonIValueBasedList, whenCallingExtractWithExistingPosition_thenReturnsElement) {
  List<int64_t> list({3, 4});
  EXPECT_EQ(3, list.extract(0));
  EXPECT_EQ(4, list.extract(1));
}

TEST(ListTestNonIValueBasedList, whenCallingExtractWithNonExistingPosition_thenThrowsException) {
  List<int64_t> list({3, 4});
  EXPECT_THROW(list.extract(2), std::out_of_range);
}

TEST(ListTestNonIValueBasedList, whenCallingCopyingSetWithExistingPosition_thenChangesElement) {
  List<int64_t> list({3, 4});
  int64_t value = 5;
  list.set(1, value);
  EXPECT_EQ(3, list.get(0));
  EXPECT_EQ(5, list.get(1));
}

TEST(ListTestNonIValueBasedList, whenCallingMovingSetWithExistingPosition_thenChangesElement) {
  List<int64_t> list({3, 4});
  int64_t value = 5;
  // NOLINTNEXTLINE(performance-move-const-arg)
  list.set(1, std::move(value));
  EXPECT_EQ(3, list.get(0));
  EXPECT_EQ(5, list.get(1));
}

```
- EN: Focus symbols: `TEST`, `list`, `EXPECT_EQ`, `extract`, `EXPECT_THROW`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`TEST`, `list`, `EXPECT_EQ`, `extract`, `EXPECT_THROW`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

### Lines 623-648
```cpp
TEST(ListTestNonIValueBasedList, whenCallingCopyingSetWithNonExistingPosition_thenThrowsException) {
  List<int64_t> list({3, 4});
  int64_t value = 5;
  EXPECT_THROW(list.set(2, value), std::out_of_range);
}

TEST(ListTestNonIValueBasedList, whenCallingMovingSetWithNonExistingPosition_thenThrowsException) {
  List<int64_t> list({3, 4});
  int64_t value = 5;
  EXPECT_THROW(list.set(2, std::move(value)), std::out_of_range);
}

TEST(ListTestNonIValueBasedList, whenCallingAccessOperatorWithExistingPosition_thenReturnsElement) {
  List<int64_t> list({3, 4});
  EXPECT_EQ(3, static_cast<int64_t>(list[0]));
  EXPECT_EQ(4, static_cast<int64_t>(list[1]));
}

TEST(ListTestNonIValueBasedList, whenAssigningToAccessOperatorWithExistingPosition_thenSetsElement) {
  List<int64_t> list({3, 4, 5});
  list[1] = 6;
  EXPECT_EQ(3, list.get(0));
  EXPECT_EQ(6, list.get(1));
  EXPECT_EQ(5, list.get(2));
}

```
- EN: Focus symbols: `TEST`, `list`, `EXPECT_THROW`, `set`, `move`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`TEST`, `list`, `EXPECT_THROW`, `set`, `move`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

### Lines 649-672
```cpp
TEST(ListTestNonIValueBasedList, whenAssigningToAccessOperatorFromAccessOperator_thenSetsElement) {
  List<int64_t> list({3, 4, 5});
  list[1] = list[2];
  EXPECT_EQ(3, list.get(0));
  EXPECT_EQ(5, list.get(1));
  EXPECT_EQ(5, list.get(2));
}

TEST(ListTestNonIValueBasedList, whenSwappingFromAccessOperator_thenSwapsElements) {
  List<int64_t> list({3, 4, 5});
  swap(list[1], list[2]);
  EXPECT_EQ(3, list.get(0));
  EXPECT_EQ(5, list.get(1));
  EXPECT_EQ(4, list.get(2));
}

TEST(ListTestNonIValueBasedList, whenCallingAccessOperatorWithNonExistingPosition_thenThrowsException) {
  List<int64_t> list({3, 4});
  EXPECT_THROW(list[2], std::out_of_range);
}

TEST(ListTestNonIValueBasedList, whenCallingInsertOnIteratorWithLValue_thenInsertsElement) {
  List<int64_t> list({3, 4, 6});
  int64_t v = 5;
```
- EN: Focus symbols: `TEST`, `list`, `EXPECT_EQ`, `get`, `swap`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`TEST`, `list`, `EXPECT_EQ`, `get`, `swap`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

### Lines 673-696
```cpp
  list.insert(list.begin() + 2, v);
  EXPECT_EQ(4, list.size());
  EXPECT_EQ(5, list.get(2));
}

TEST(ListTestNonIValueBasedList, whenCallingInsertOnIteratorWithRValue_thenInsertsElement) {
  List<int64_t> list({3, 4, 6});
  int64_t v = 5;
  // NOLINTNEXTLINE(performance-move-const-arg)
  list.insert(list.begin() + 2, std::move(v));
  EXPECT_EQ(4, list.size());
  EXPECT_EQ(5, list.get(2));
}

TEST(ListTestNonIValueBasedList, whenCallingInsertWithLValue_thenReturnsIteratorToNewElement) {
  List<int64_t> list({3, 4, 6});
  int64_t v = 5;
  List<int64_t>::iterator result = list.insert(list.begin() + 2, v);
  EXPECT_EQ(list.begin() + 2, result);
}

TEST(ListTestNonIValueBasedList, whenCallingInsertWithRValue_thenReturnsIteratorToNewElement) {
  List<int64_t> list({3, 4, 6});
  int64_t v = 5;
```
- EN: Focus symbols: `insert`, `begin`, `EXPECT_EQ`, `size`, `get`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`insert`, `begin`, `EXPECT_EQ`, `size`, `get`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

### Lines 697-720
```cpp
  // NOLINTNEXTLINE(performance-move-const-arg)
  List<int64_t>::iterator result = list.insert(list.begin() + 2, std::move(v));
  EXPECT_EQ(list.begin() + 2, result);
}

TEST(ListTestNonIValueBasedList, whenCallingEmplaceWithLValue_thenInsertsElement) {
  List<int64_t> list({3, 4, 6});
  int64_t v = 5;
  list.emplace(list.begin() + 2, v);
  EXPECT_EQ(4, list.size());
  EXPECT_EQ(5, list.get(2));
}

TEST(ListTestNonIValueBasedList, whenCallingEmplaceWithRValue_thenInsertsElement) {
  List<int64_t> list({3, 4, 6});
  int64_t v = 5;
  // NOLINTNEXTLINE(performance-move-const-arg)
  list.emplace(list.begin() + 2, std::move(v));
  EXPECT_EQ(4, list.size());
  EXPECT_EQ(5, list.get(2));
}

TEST(ListTestNonIValueBasedList, whenCallingEmplaceWithConstructorArg_thenInsertsElement) {
  List<int64_t> list({3, 4, 6});
```
- EN: Focus symbols: `insert`, `begin`, `move`, `EXPECT_EQ`, `TEST`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`insert`, `begin`, `move`, `EXPECT_EQ`, `TEST`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

### Lines 721-744
```cpp
  list.emplace(list.begin() + 2, 5); // const char* is a constructor arg to std::int64_t
  EXPECT_EQ(4, list.size());
  EXPECT_EQ(5, list.get(2));
}

TEST(ListTestNonIValueBasedList, whenCallingPushBackWithLValue_ThenInsertsElement) {
  List<int64_t> list;
  int64_t v = 5;
  list.push_back(v);
  EXPECT_EQ(1, list.size());
  EXPECT_EQ(5, list.get(0));
}

TEST(ListTestNonIValueBasedList, whenCallingPushBackWithRValue_ThenInsertsElement) {
  List<int64_t> list;
  int64_t v = 5;
  // NOLINTNEXTLINE(performance-move-const-arg)
  list.push_back(std::move(v));
  EXPECT_EQ(1, list.size());
  EXPECT_EQ(5, list.get(0));
}

TEST(ListTestNonIValueBasedList, whenCallingEmplaceBackWithLValue_ThenInsertsElement) {
  List<int64_t> list;
```
- EN: Focus symbols: `emplace`, `begin`, `EXPECT_EQ`, `size`, `get`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`emplace`, `begin`, `EXPECT_EQ`, `size`, `get`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

### Lines 745-768
```cpp
  int64_t v = 5;
  list.emplace_back(v);
  EXPECT_EQ(1, list.size());
  EXPECT_EQ(5, list.get(0));
}

TEST(ListTestNonIValueBasedList, whenCallingEmplaceBackWithRValue_ThenInsertsElement) {
  List<int64_t> list;
  int64_t v = 5;
  // NOLINTNEXTLINE(performance-move-const-arg)
  list.emplace_back(std::move(v));
  EXPECT_EQ(1, list.size());
  EXPECT_EQ(5, list.get(0));
}

TEST(ListTestNonIValueBasedList, whenCallingEmplaceBackWithConstructorArg_ThenInsertsElement) {
  List<int64_t> list;
  list.emplace_back(5);  // const char* is a constructor arg to std::int64_t
  EXPECT_EQ(1, list.size());
  EXPECT_EQ(5, list.get(0));
}

TEST(ListTestNonIValueBasedList, givenEmptyList_whenIterating_thenBeginIsEnd) {
  List<int64_t> list;
```
- EN: Focus symbols: `emplace_back`, `EXPECT_EQ`, `size`, `get`, `TEST`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`emplace_back`, `EXPECT_EQ`, `size`, `get`, `TEST`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

### Lines 769-794
```cpp
  const List<int64_t> clist;
  EXPECT_EQ(list.begin(), list.end());
  EXPECT_EQ(list.begin(), list.end());
  EXPECT_EQ(clist.begin(), clist.end());
  EXPECT_EQ(clist.begin(), clist.end());
}

TEST(ListTestNonIValueBasedList, whenIterating_thenFindsElements) {
  List<int64_t> list({3, 5});
  bool found_first = false;
  bool found_second = false;
  for (const auto && iter : list) {
    if (static_cast<int64_t>(iter) == 3) {
      EXPECT_FALSE(found_first);
      found_first = true;
    } else if (static_cast<int64_t>(iter) == 5) {
      EXPECT_FALSE(found_second);
      found_second = true;
    } else {
      ADD_FAILURE();
    }
  }
  EXPECT_TRUE(found_first);
  EXPECT_TRUE(found_second);
}

```
- EN: Focus symbols: `EXPECT_EQ`, `begin`, `end`, `TEST`, `list`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`EXPECT_EQ`, `begin`, `end`, `TEST`, `list`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

### Lines 795-820
```cpp
TEST(ListTestNonIValueBasedList, whenIteratingWithForeach_thenFindsElements) {
  List<int64_t> list({3, 5});
  bool found_first = false;
  bool found_second = false;
  // NOLINTNEXTLINE(performance-implicit-conversion-in-loop)
  for (const int64_t& elem : list) {
    if (elem == 3) {
      EXPECT_FALSE(found_first);
      found_first = true;
    } else if (elem == 5) {
      EXPECT_FALSE(found_second);
      found_second = true;
    } else {
      ADD_FAILURE();
    }
  }
  EXPECT_TRUE(found_first);
  EXPECT_TRUE(found_second);
}

TEST(ListTestNonIValueBasedList, givenOneElementList_whenErasing_thenListIsEmpty) {
  List<int64_t> list({3});
  list.erase(list.begin());
  EXPECT_TRUE(list.empty());
}

```
- EN: Focus symbols: `TEST`, `list`, `EXPECT_FALSE`, `ADD_FAILURE`, `EXPECT_TRUE`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`TEST`, `list`, `EXPECT_FALSE`, `ADD_FAILURE`, `EXPECT_TRUE`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

### Lines 821-848
```cpp
TEST(ListTestNonIValueBasedList, givenList_whenErasing_thenReturnsIterator) {
  List<int64_t> list({1, 2, 3});
  List<int64_t>::iterator iter = list.erase(list.begin() + 1);
  EXPECT_EQ(list.begin() + 1, iter);
}

TEST(ListTestNonIValueBasedList, givenList_whenErasingFullRange_thenIsEmpty) {
  List<int64_t> list({1, 2, 3});
  list.erase(list.begin(), list.end());
  EXPECT_TRUE(list.empty());
}

TEST(ListTestNonIValueBasedList, whenCallingReserve_thenDoesntCrash) {
  List<int64_t> list;
  list.reserve(100);
}

TEST(ListTestNonIValueBasedList, whenCopyConstructingList_thenAreEqual) {
  List<int64_t> list1({3, 4});

  // NOLINTNEXTLINE(performance-unnecessary-copy-initialization)
  List<int64_t> list2(list1);

  EXPECT_EQ(2, list2.size());
  EXPECT_EQ(3, list2.get(0));
  EXPECT_EQ(4, list2.get(1));
}

```
- EN: Focus symbols: `TEST`, `list`, `erase`, `begin`, `EXPECT_EQ`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`TEST`, `list`, `erase`, `begin`, `EXPECT_EQ`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

### Lines 849-874
```cpp
TEST(ListTestNonIValueBasedList, whenCopyAssigningList_thenAreEqual) {
  List<int64_t> list1({3, 4});

  List<int64_t> list2;
  list2 = list1;

  EXPECT_EQ(2, list2.size());
  EXPECT_EQ(3, list2.get(0));
  EXPECT_EQ(4, list2.get(1));
}

TEST(ListTestNonIValueBasedList, whenCopyingList_thenAreEqual) {
  List<int64_t> list1({3, 4});

  List<int64_t> list2 = list1.copy();

  EXPECT_EQ(2, list2.size());
  EXPECT_EQ(3, list2.get(0));
  EXPECT_EQ(4, list2.get(1));
}

TEST(ListTestNonIValueBasedList, whenMoveConstructingList_thenNewIsCorrect) {
  List<int64_t> list1({3, 4});

  List<int64_t> list2(std::move(list1));

```
- EN: Focus symbols: `TEST`, `list1`, `EXPECT_EQ`, `size`, `get`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`TEST`, `list1`, `EXPECT_EQ`, `size`, `get`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

### Lines 875-899
```cpp
  EXPECT_EQ(2, list2.size());
  EXPECT_EQ(3, list2.get(0));
  EXPECT_EQ(4, list2.get(1));
}

TEST(ListTestNonIValueBasedList, whenMoveAssigningList_thenNewIsCorrect) {
  List<int64_t> list1({3, 4});

  List<int64_t> list2;
  list2 = std::move(list1);

  EXPECT_EQ(2, list2.size());
  EXPECT_EQ(3, list2.get(0));
  EXPECT_EQ(4, list2.get(1));
}

TEST(ListTestNonIValueBasedList, whenMoveConstructingList_thenOldIsUnchanged) {
  List<int64_t> list1({3, 4});

  List<int64_t> list2(std::move(list1));
  EXPECT_EQ(2, list1.size());
  EXPECT_EQ(3, list1.get(0));
  EXPECT_EQ(4, list1.get(1));
}

```
- EN: Focus symbols: `EXPECT_EQ`, `size`, `get`, `TEST`, `list1`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`EXPECT_EQ`, `size`, `get`, `TEST`, `list1`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

### Lines 900-927
```cpp
TEST(ListTestNonIValueBasedList, whenMoveAssigningList_thenOldIsUnchanged) {
  List<int64_t> list1({3, 4});

  List<int64_t> list2;
  list2 = std::move(list1);
  EXPECT_EQ(2, list1.size());
  EXPECT_EQ(3, list1.get(0));
  EXPECT_EQ(4, list1.get(1));
}

TEST(ListTestNonIValueBasedList, givenIterator_whenPostfixIncrementing_thenMovesToNextAndReturnsOldPosition) {
  List<int64_t> list({3, 4});

  List<int64_t>::iterator iter1 = list.begin();
  List<int64_t>::iterator iter2 = iter1++;
  EXPECT_NE(3, static_cast<int64_t>(*iter1));
  EXPECT_EQ(3, static_cast<int64_t>(*iter2));
}

TEST(ListTestNonIValueBasedList, givenIterator_whenPrefixIncrementing_thenMovesToNextAndReturnsNewPosition) {
  List<int64_t> list({3, 4});

  List<int64_t>::iterator iter1 = list.begin();
  List<int64_t>::iterator iter2 = ++iter1;
  EXPECT_NE(3, static_cast<int64_t>(*iter1));
  EXPECT_NE(3, static_cast<int64_t>(*iter2));
}

```
- EN: Focus symbols: `TEST`, `list1`, `move`, `EXPECT_EQ`, `size`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`TEST`, `list1`, `move`, `EXPECT_EQ`, `size`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

### Lines 928-954
```cpp
TEST(ListTestNonIValueBasedList, givenIterator_whenPostfixDecrementing_thenMovesToNextAndReturnsOldPosition) {
  List<int64_t> list({3, 4});

  List<int64_t>::iterator iter1 = list.end() - 1;
  List<int64_t>::iterator iter2 = iter1--;
  EXPECT_NE(4, static_cast<int64_t>(*iter1));
  EXPECT_EQ(4, static_cast<int64_t>(*iter2));
}

TEST(ListTestNonIValueBasedList, givenIterator_whenPrefixDecrementing_thenMovesToNextAndReturnsNewPosition) {
  List<int64_t> list({3, 4});

  List<int64_t>::iterator iter1 = list.end() - 1;
  List<int64_t>::iterator iter2 = --iter1;
  EXPECT_NE(4, static_cast<int64_t>(*iter1));
  EXPECT_NE(4, static_cast<int64_t>(*iter2));
}

TEST(ListTestNonIValueBasedList, givenIterator_whenIncreasing_thenMovesToNextAndReturnsNewPosition) {
  List<int64_t> list({3, 4, 5});

  List<int64_t>::iterator iter1 = list.begin();
  List<int64_t>::iterator iter2 = iter1 += 2;
  EXPECT_EQ(5, static_cast<int64_t>(*iter1));
  EXPECT_EQ(5, static_cast<int64_t>(*iter2));
}

```
- EN: Focus symbols: `TEST`, `list`, `end`, `EXPECT_NE`, `EXPECT_EQ`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`TEST`, `list`, `end`, `EXPECT_NE`, `EXPECT_EQ`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

### Lines 955-981
```cpp
TEST(ListTestNonIValueBasedList, givenIterator_whenDecreasing_thenMovesToNextAndReturnsNewPosition) {
  List<int64_t> list({3, 4, 5});

  List<int64_t>::iterator iter1 = list.end();
  List<int64_t>::iterator iter2 = iter1 -= 2;
  EXPECT_EQ(4, static_cast<int64_t>(*iter1));
  EXPECT_EQ(4, static_cast<int64_t>(*iter2));
}

TEST(ListTestNonIValueBasedList, givenIterator_whenAdding_thenReturnsNewIterator) {
  List<int64_t> list({3, 4, 5});

  List<int64_t>::iterator iter1 = list.begin();
  List<int64_t>::iterator iter2 = iter1 + 2;
  EXPECT_EQ(3, static_cast<int64_t>(*iter1));
  EXPECT_EQ(5, static_cast<int64_t>(*iter2));
}

TEST(ListTestNonIValueBasedList, givenIterator_whenSubtracting_thenReturnsNewIterator) {
  List<int64_t> list({3, 4, 5});

  List<int64_t>::iterator iter1 = list.end() - 1;
  List<int64_t>::iterator iter2 = iter1 - 2;
  EXPECT_EQ(5, static_cast<int64_t>(*iter1));
  EXPECT_EQ(3, static_cast<int64_t>(*iter2));
}

```
- EN: Focus symbols: `TEST`, `list`, `end`, `EXPECT_EQ`, `begin`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`TEST`, `list`, `end`, `EXPECT_EQ`, `begin`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

### Lines 982-1006
```cpp
TEST(ListTestNonIValueBasedList, givenIterator_whenCalculatingDifference_thenReturnsCorrectNumber) {
  List<int64_t> list({3, 4});
  EXPECT_EQ(2, list.end() - list.begin());
}

TEST(ListTestNonIValueBasedList, givenEqualIterators_thenAreEqual) {
  List<int64_t> list({3, 4});

  List<int64_t>::iterator iter1 = list.begin();
  List<int64_t>::iterator iter2 = list.begin();
  EXPECT_TRUE(iter1 == iter2);
  EXPECT_FALSE(iter1 != iter2);
}

TEST(ListTestNonIValueBasedList, givenDifferentIterators_thenAreNotEqual) {
  List<int64_t> list({3, 4});

  List<int64_t>::iterator iter1 = list.begin();
  List<int64_t>::iterator iter2 = list.begin();
  iter2++;

  EXPECT_FALSE(iter1 == iter2);
  EXPECT_TRUE(iter1 != iter2);
}

```
- EN: Focus symbols: `TEST`, `list`, `EXPECT_EQ`, `end`, `begin`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`TEST`, `list`, `EXPECT_EQ`, `end`, `begin`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

### Lines 1007-1033
```cpp
TEST(ListTestNonIValueBasedList, givenIterator_whenDereferencing_thenPointsToCorrectElement) {
  List<int64_t> list({3});

  List<int64_t>::iterator iter = list.begin();
  EXPECT_EQ(3, static_cast<int64_t>(*iter));
}

TEST(ListTestNonIValueBasedList, givenIterator_whenAssigningNewValue_thenChangesValue) {
  List<int64_t> list({3});

  List<int64_t>::iterator iter = list.begin();
  *iter = 4;
  EXPECT_EQ(4, list.get(0));
}

TEST(ListTestNonIValueBasedList, givenIterator_whenAssigningNewValueFromIterator_thenChangesValue) {
  List<int64_t> list({3, 4});

  List<int64_t>::iterator iter = list.begin();
  *iter = *(iter + 1);
  EXPECT_EQ(4, list.get(0));
  EXPECT_EQ(4, list.get(1));
}

TEST(ListTestNonIValueBasedList, givenIterator_whenSwappingValuesFromIterator_thenChangesValue) {
  List<int64_t> list({3, 4});

```
- EN: Focus symbols: `TEST`, `list`, `begin`, `EXPECT_EQ`, `get`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`TEST`, `list`, `begin`, `EXPECT_EQ`, `get`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

### Lines 1034-1061
```cpp
  List<int64_t>::iterator iter = list.begin();
  swap(*iter, *(iter + 1));
  EXPECT_EQ(4, list.get(0));
  EXPECT_EQ(3, list.get(1));
}

TEST(ListTestNonIValueBasedList, givenOneElementList_whenCallingPopBack_thenIsEmpty) {
  List<int64_t> list({3});
  list.pop_back();
  EXPECT_TRUE(list.empty());
}

TEST(ListTestNonIValueBasedList, givenEmptyList_whenCallingResize_thenResizesAndSetsEmptyValue) {
  List<int64_t> list;
  list.resize(2);
  EXPECT_EQ(2, list.size());
  EXPECT_EQ(0, list.get(0));
  EXPECT_EQ(0, list.get(1));
}

TEST(ListTestNonIValueBasedList, givenEmptyList_whenCallingResizeWithValue_thenResizesAndSetsValue) {
  List<int64_t> list;
  list.resize(2, 5);
  EXPECT_EQ(2, list.size());
  EXPECT_EQ(5, list.get(0));
  EXPECT_EQ(5, list.get(1));
}

```
- EN: Focus symbols: `begin`, `swap`, `EXPECT_EQ`, `get`, `TEST`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`begin`, `swap`, `EXPECT_EQ`, `get`, `TEST`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

### Lines 1062-1086
```cpp
TEST(ListTestNonIValueBasedList, isReferenceType) {
  List<int64_t> list1;
  // NOLINTNEXTLINE(performance-unnecessary-copy-initialization)
  List<int64_t> list2(list1);
  List<int64_t> list3;
  list3 = list1;

  list1.push_back(3);
  EXPECT_EQ(1, list1.size());
  EXPECT_EQ(1, list2.size());
  EXPECT_EQ(1, list3.size());
}

TEST(ListTestNonIValueBasedList, copyHasSeparateStorage) {
  List<int64_t> list1;
  List<int64_t> list2(list1.copy());
  List<int64_t> list3;
  list3 = list1.copy();

  list1.push_back(3);
  EXPECT_EQ(1, list1.size());
  EXPECT_EQ(0, list2.size());
  EXPECT_EQ(0, list3.size());
}

```
- EN: Focus symbols: `TEST`, `list2`, `push_back`, `EXPECT_EQ`, `size`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`TEST`, `list2`, `push_back`, `EXPECT_EQ`, `size`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

### Lines 1087-1112
```cpp
TEST(ListTestNonIValueBasedList, givenEqualLists_thenIsEqual) {
  List<int64_t> list1({1, 3});
  List<int64_t> list2({1, 3});

  EXPECT_EQ(list1, list2);
}

TEST(ListTestNonIValueBasedList, givenDifferentLists_thenIsNotEqual) {
  List<int64_t> list1({1, 3});
  List<int64_t> list2({1, 2});

  EXPECT_NE(list1, list2);
}

TEST(ListTestNonIValueBasedList, isChecksIdentity) {
  List<int64_t> list1({1, 3});
  // NOLINTNEXTLINE(performance-unnecessary-copy-initialization)
  const auto list2 = list1;

  EXPECT_TRUE(list1.is(list2));
}

TEST(ListTestNonIValueBasedList, sameValueDifferentStorage_thenIsReturnsFalse) {
  List<int64_t> list1({1, 3});
  const auto list2 = list1.copy();

```
- EN: Focus symbols: `TEST`, `list1`, `list2`, `EXPECT_EQ`, `EXPECT_NE`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`TEST`, `list1`, `list2`, `EXPECT_EQ`, `EXPECT_NE`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

### Lines 1113-1136
```cpp
  EXPECT_FALSE(list1.is(list2));
}

TEST(ListTest, canAccessStringByReference) {
  List<std::string> list({"one", "two"});
  const auto& listRef = list;
  static_assert(std::is_same_v<decltype(listRef[1]), const std::string&>,
                "const List<std::string> access should be by const reference");
  std::string str = list[1];
  const std::string& strRef = listRef[1];
  EXPECT_EQ("two", str);
  EXPECT_EQ("two", strRef);
}

TEST(ListTest, canAccessOptionalStringByReference) {
  List<std::optional<std::string>> list({"one", "two", std::nullopt});
  const auto& listRef = list;
  static_assert(
      std::is_same_v<decltype(listRef[1]), std::optional<std::reference_wrapper<const std::string>>>,
      "List<std::optional<std::string>> access should be by const reference");
  std::optional<std::string> str1 = list[1];
  std::optional<std::string> str2 = list[2];
  auto const& strRef1 = listRef[1];
  auto const& strRef2 = listRef[2];
```
- EN: Focus symbols: `EXPECT_FALSE`, `is`, `TEST`, `list`, `static_assert`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`EXPECT_FALSE`, `is`, `TEST`, `list`, `static_assert`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

### Lines 1137-1160
```cpp
  EXPECT_EQ("two", str1);
  EXPECT_FALSE(str2.has_value());
  EXPECT_TRUE(strRef1.has_value());
  // NOLINTNEXTLINE(bugprone-unchecked-optional-access)
  EXPECT_EQ("two", strRef1.value().get());
  EXPECT_FALSE(strRef2.has_value());
}

TEST(ListTest, canAccessTensorByReference) {
  List<at::Tensor> list;
  const auto& listRef = list;
  static_assert(
      std::is_same_v<decltype(listRef[0]), const at::Tensor&>,
      "List<at::Tensor> access should be by const reference");
}

TEST(ListTest, toTypedList) {
  List<std::string> stringList({"one", "two"});
  auto genericList = impl::toList(std::move(stringList));
  EXPECT_EQ(genericList.size(), 2);
  stringList = c10::impl::toTypedList<std::string>(std::move(genericList));
  EXPECT_EQ(stringList.size(), 2);

  genericList = impl::toList(std::move(stringList));
```
- EN: Focus symbols: `EXPECT_EQ`, `EXPECT_FALSE`, `has_value`, `EXPECT_TRUE`, `value`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`EXPECT_EQ`, `EXPECT_FALSE`, `has_value`, `EXPECT_TRUE`, `value`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

### Lines 1161-1163
```cpp
  EXPECT_THROW(c10::impl::toTypedList<int64_t>(std::move(genericList)), c10::Error);
}
// NOLINTEND(performance-move-const-arg, bugprone-use-after-move, *analyzer*Move)
```
- EN: Focus symbols: `EXPECT_THROW`, `move`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`EXPECT_THROW`, `move`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

## Key Concepts / 关键概念
- core ATen/C10 abstractions / ATen/C10 核心抽象
- Conditional compilation / 条件编译
- Behavioral regression tests / 行为回归测试
- ATen/C10 internal layering / ATen/C10 内部分层

## Dependencies / 依赖关系
- Direct internal includes / 直接内部依赖: `ATen/core/List.h`
- External/system includes / 外部或系统头: `gtest/gtest.h`
- Inferred semantic dependencies / 推断出的语义依赖: tensor abstractions / 张量抽象; namespace scoping / 命名空间作用域
