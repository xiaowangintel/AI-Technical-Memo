# xray_function_call_trie.h — Code Analysis / 代码分析
## Source / 来源
- **File**: `compiler-rt/lib/xray/xray_function_call_trie.h`
- **Repository**: `llvm-project` (`/root/xw/llvm-project`)
- **Purpose (EN)**: This file is a part of XRay, a dynamic runtime instrumentation system.
- **目的（中文）**: 该头文件声明与 `XRay function call trie` 相关的接口、类型或常量。

## Line-by-Line Analysis / 逐行分析
### Line 1
````cpp
//===-- xray_function_call_trie.h ------------------------------*- C++ -*-===//
````
- **EN**: Banner comment delimiting a file header or major section.
- **CN**: 横幅注释，用于分隔文件头或主要章节。

### Line 2
````cpp
//
````
- **EN**: Comment formatting line in a banner or block comment.
- **CN**: 注释格式行，用于组织文件头或注释块。

### Line 3
````cpp
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
````
- **EN**: Comment documenting `Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.`.
- **CN**: 注释说明了 `Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.`。

### Line 4
````cpp
// See https://llvm.org/LICENSE.txt for license information.
````
- **EN**: Comment documenting `See https://llvm.org/LICENSE.txt for license information.`.
- **CN**: 注释说明了 `See https://llvm.org/LICENSE.txt for license information.`。

### Line 5
````cpp
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
````
- **EN**: Comment documenting `SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception`.
- **CN**: 注释说明了 `SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception`。

### Line 6
````cpp
//
````
- **EN**: Comment formatting line in a banner or block comment.
- **CN**: 注释格式行，用于组织文件头或注释块。

### Line 7
````cpp
//===----------------------------------------------------------------------===//
````
- **EN**: Banner comment delimiting a file header or major section.
- **CN**: 横幅注释，用于分隔文件头或主要章节。

### Line 8
````cpp
//
````
- **EN**: Comment formatting line in a banner or block comment.
- **CN**: 注释格式行，用于组织文件头或注释块。

### Line 9
````cpp
// This file is a part of XRay, a dynamic runtime instrumentation system.
````
- **EN**: Comment documenting `This file is a part of XRay, a dynamic runtime instrumentation system.`.
- **CN**: 注释说明了 `This file is a part of XRay, a dynamic runtime instrumentation system.`。

### Line 10
````cpp
//
````
- **EN**: Comment formatting line in a banner or block comment.
- **CN**: 注释格式行，用于组织文件头或注释块。

### Line 11
````cpp
// This file defines the interface for a function call trie.
````
- **EN**: Comment documenting `This file defines the interface for a function call trie.`.
- **CN**: 注释说明了 `This file defines the interface for a function call trie.`。

### Line 12
````cpp
//
````
- **EN**: Comment formatting line in a banner or block comment.
- **CN**: 注释格式行，用于组织文件头或注释块。

### Line 13
````cpp
//===----------------------------------------------------------------------===//
````
- **EN**: Banner comment delimiting a file header or major section.
- **CN**: 横幅注释，用于分隔文件头或主要章节。

### Line 14
````cpp
#ifndef XRAY_FUNCTION_CALL_TRIE_H
````
- **EN**: Starts a preprocessor condition: `#ifndef XRAY_FUNCTION_CALL_TRIE_H`.
- **CN**: 开始一个预处理条件：`#ifndef XRAY_FUNCTION_CALL_TRIE_H`。

### Line 15
````cpp
#define XRAY_FUNCTION_CALL_TRIE_H
````
- **EN**: Defines a macro or compile-time constant: `#define XRAY_FUNCTION_CALL_TRIE_H`.
- **CN**: 定义宏或编译期常量：`#define XRAY_FUNCTION_CALL_TRIE_H`。

### Line 16
````cpp
<blank>
````
- **EN**: Blank line separating logical sections.
- **CN**: 空行，用于分隔逻辑段落。

### Line 17
````cpp
#include "xray_buffer_queue.h"
````
- **EN**: Includes the local dependency `xray_buffer_queue.h`.
- **CN**: 引入本地依赖 `xray_buffer_queue.h`。

### Line 18
````cpp
#include "xray_defs.h"
````
- **EN**: Includes the local dependency `xray_defs.h`.
- **CN**: 引入本地依赖 `xray_defs.h`。

### Line 19
````cpp
#include "xray_profiling_flags.h"
````
- **EN**: Includes the local dependency `xray_profiling_flags.h`.
- **CN**: 引入本地依赖 `xray_profiling_flags.h`。

### Line 20
````cpp
#include "xray_segmented_array.h"
````
- **EN**: Includes the local dependency `xray_segmented_array.h`.
- **CN**: 引入本地依赖 `xray_segmented_array.h`。

### Line 21
````cpp
#include <limits>
````
- **EN**: Includes the system dependency `limits`.
- **CN**: 引入系统依赖 `limits`。

### Line 22
````cpp
#include <memory> // For placement new.
````
- **EN**: Includes the system dependency `memory`.
- **CN**: 引入系统依赖 `memory`。

### Line 23
````cpp
#include <utility>
````
- **EN**: Includes the system dependency `utility`.
- **CN**: 引入系统依赖 `utility`。

### Line 24
````cpp
<blank>
````
- **EN**: Blank line separating logical sections.
- **CN**: 空行，用于分隔逻辑段落。

### Line 25
````cpp
namespace __xray {
````
- **EN**: Opens namespace `__xray`.
- **CN**: 打开命名空间 `__xray`。

### Line 26
````cpp
<blank>
````
- **EN**: Blank line separating logical sections.
- **CN**: 空行，用于分隔逻辑段落。

### Line 27
````cpp
/// A FunctionCallTrie represents the stack traces of XRay instrumented
````
- **EN**: Comment documenting `/ A FunctionCallTrie represents the stack traces of XRay instrumented`.
- **CN**: 注释说明了 `/ A FunctionCallTrie represents the stack traces of XRay instrumented`。

### Line 28
````cpp
/// functions that we've encountered, where a node corresponds to a function and
````
- **EN**: Comment documenting `/ functions that we've encountered, where a node corresponds to a function and`.
- **CN**: 注释说明了 `/ functions that we've encountered, where a node corresponds to a function and`。

### Line 29
````cpp
/// the path from the root to the node its stack trace. Each node in the trie
````
- **EN**: Comment documenting `/ the path from the root to the node its stack trace. Each node in the trie`.
- **CN**: 注释说明了 `/ the path from the root to the node its stack trace. Each node in the trie`。

### Line 30
````cpp
/// will contain some useful values, including:
````
- **EN**: Comment documenting `/ will contain some useful values, including:`.
- **CN**: 注释说明了 `/ will contain some useful values, including:`。

### Line 31
````cpp
///
````
- **EN**: Banner comment delimiting a file header or major section.
- **CN**: 横幅注释，用于分隔文件头或主要章节。

### Line 32
````cpp
///   * The cumulative amount of time spent in this particular node/stack.
````
- **EN**: Comment documenting `/   * The cumulative amount of time spent in this particular node/stack.`.
- **CN**: 注释说明了 `/   * The cumulative amount of time spent in this particular node/stack.`。

### Line 33
````cpp
///   * The number of times this stack has appeared.
````
- **EN**: Comment documenting `/   * The number of times this stack has appeared.`.
- **CN**: 注释说明了 `/   * The number of times this stack has appeared.`。

### Line 34
````cpp
///   * A histogram of latencies for that particular node.
````
- **EN**: Comment documenting `/   * A histogram of latencies for that particular node.`.
- **CN**: 注释说明了 `/   * A histogram of latencies for that particular node.`。

### Line 35
````cpp
///
````
- **EN**: Banner comment delimiting a file header or major section.
- **CN**: 横幅注释，用于分隔文件头或主要章节。

### Line 36
````cpp
/// Each node in the trie will also contain a list of callees, represented using
````
- **EN**: Comment documenting `/ Each node in the trie will also contain a list of callees, represented using`.
- **CN**: 注释说明了 `/ Each node in the trie will also contain a list of callees, represented using`。

### Line 37
````cpp
/// a Array<NodeIdPair> -- each NodeIdPair instance will contain the function
````
- **EN**: Comment documenting `/ a Array<NodeIdPair> -- each NodeIdPair instance will contain the function`.
- **CN**: 注释说明了 `/ a Array<NodeIdPair> -- each NodeIdPair instance will contain the function`。

### Line 38
````cpp
/// ID of the callee, and a pointer to the node.
````
- **EN**: Comment documenting `/ ID of the callee, and a pointer to the node.`.
- **CN**: 注释说明了 `/ ID of the callee, and a pointer to the node.`。

### Line 39
````cpp
///
````
- **EN**: Banner comment delimiting a file header or major section.
- **CN**: 横幅注释，用于分隔文件头或主要章节。

### Line 40
````cpp
/// If we visualise this data structure, we'll find the following potential
````
- **EN**: Comment documenting `/ If we visualise this data structure, we'll find the following potential`.
- **CN**: 注释说明了 `/ If we visualise this data structure, we'll find the following potential`。

### Line 41
````cpp
/// representation:
````
- **EN**: Comment documenting `/ representation:`.
- **CN**: 注释说明了 `/ representation:`。

### Line 42
````cpp
///
````
- **EN**: Banner comment delimiting a file header or major section.
- **CN**: 横幅注释，用于分隔文件头或主要章节。

### Line 43
````cpp
///   [function id node] -> [callees] [cumulative time]
````
- **EN**: Comment documenting `/   [function id node] -> [callees] [cumulative time]`.
- **CN**: 注释说明了 `/   [function id node] -> [callees] [cumulative time]`。

### Line 44
````cpp
///                         [call counter] [latency histogram]
````
- **EN**: Comment documenting `/                         [call counter] [latency histogram]`.
- **CN**: 注释说明了 `/                         [call counter] [latency histogram]`。

### Line 45
````cpp
///
````
- **EN**: Banner comment delimiting a file header or major section.
- **CN**: 横幅注释，用于分隔文件头或主要章节。

### Line 46
````cpp
/// As an example, when we have a function in this pseudocode:
````
- **EN**: Comment documenting `/ As an example, when we have a function in this pseudocode:`.
- **CN**: 注释说明了 `/ As an example, when we have a function in this pseudocode:`。

### Line 47
````cpp
///
````
- **EN**: Banner comment delimiting a file header or major section.
- **CN**: 横幅注释，用于分隔文件头或主要章节。

### Line 48
````cpp
///   func f(N) {
````
- **EN**: Comment documenting `/   func f(N) {`.
- **CN**: 注释说明了 `/   func f(N) {`。

### Line 49
````cpp
///     g()
````
- **EN**: Comment documenting `/     g()`.
- **CN**: 注释说明了 `/     g()`。

### Line 50
````cpp
///     h()
````
- **EN**: Comment documenting `/     h()`.
- **CN**: 注释说明了 `/     h()`。

### Line 51
````cpp
///     for i := 1..N { j() }
````
- **EN**: Comment documenting `/     for i := 1..N { j() }`.
- **CN**: 注释说明了 `/     for i := 1..N { j() }`。

### Line 52
````cpp
///   }
````
- **EN**: Comment documenting `/   }`.
- **CN**: 注释说明了 `/   }`。

### Line 53
````cpp
///
````
- **EN**: Banner comment delimiting a file header or major section.
- **CN**: 横幅注释，用于分隔文件头或主要章节。

### Line 54
````cpp
/// We may end up with a trie of the following form:
````
- **EN**: Comment documenting `/ We may end up with a trie of the following form:`.
- **CN**: 注释说明了 `/ We may end up with a trie of the following form:`。

### Line 55
````cpp
///
````
- **EN**: Banner comment delimiting a file header or major section.
- **CN**: 横幅注释，用于分隔文件头或主要章节。

### Line 56
````cpp
///   f -> [ g, h, j ] [...] [1] [...]
````
- **EN**: Comment documenting `/   f -> [ g, h, j ] [...] [1] [...]`.
- **CN**: 注释说明了 `/   f -> [ g, h, j ] [...] [1] [...]`。

### Line 57
````cpp
///   g -> [ ... ] [...] [1] [...]
````
- **EN**: Comment documenting `/   g -> [ ... ] [...] [1] [...]`.
- **CN**: 注释说明了 `/   g -> [ ... ] [...] [1] [...]`。

### Line 58
````cpp
///   h -> [ ... ] [...] [1] [...]
````
- **EN**: Comment documenting `/   h -> [ ... ] [...] [1] [...]`.
- **CN**: 注释说明了 `/   h -> [ ... ] [...] [1] [...]`。

### Line 59
````cpp
///   j -> [ ... ] [...] [N] [...]
````
- **EN**: Comment documenting `/   j -> [ ... ] [...] [N] [...]`.
- **CN**: 注释说明了 `/   j -> [ ... ] [...] [N] [...]`。

### Line 60
````cpp
///
````
- **EN**: Banner comment delimiting a file header or major section.
- **CN**: 横幅注释，用于分隔文件头或主要章节。

### Line 61
````cpp
/// If for instance the function g() called j() like so:
````
- **EN**: Comment documenting `/ If for instance the function g() called j() like so:`.
- **CN**: 注释说明了 `/ If for instance the function g() called j() like so:`。

### Line 62
````cpp
///
````
- **EN**: Banner comment delimiting a file header or major section.
- **CN**: 横幅注释，用于分隔文件头或主要章节。

### Line 63
````cpp
///   func g() {
````
- **EN**: Comment documenting `/   func g() {`.
- **CN**: 注释说明了 `/   func g() {`。

### Line 64
````cpp
///     for i := 1..10 { j() }
````
- **EN**: Comment documenting `/     for i := 1..10 { j() }`.
- **CN**: 注释说明了 `/     for i := 1..10 { j() }`。

### Line 65
````cpp
///   }
````
- **EN**: Comment documenting `/   }`.
- **CN**: 注释说明了 `/   }`。

### Line 66
````cpp
///
````
- **EN**: Banner comment delimiting a file header or major section.
- **CN**: 横幅注释，用于分隔文件头或主要章节。

### Line 67
````cpp
/// We'll find the following updated trie:
````
- **EN**: Comment documenting `/ We'll find the following updated trie:`.
- **CN**: 注释说明了 `/ We'll find the following updated trie:`。

### Line 68
````cpp
///
````
- **EN**: Banner comment delimiting a file header or major section.
- **CN**: 横幅注释，用于分隔文件头或主要章节。

### Line 69
````cpp
///   f -> [ g, h, j ] [...] [1] [...]
````
- **EN**: Comment documenting `/   f -> [ g, h, j ] [...] [1] [...]`.
- **CN**: 注释说明了 `/   f -> [ g, h, j ] [...] [1] [...]`。

### Line 70
````cpp
///   g -> [ j' ] [...] [1] [...]
````
- **EN**: Comment documenting `/   g -> [ j' ] [...] [1] [...]`.
- **CN**: 注释说明了 `/   g -> [ j' ] [...] [1] [...]`。

### Line 71
````cpp
///   h -> [ ... ] [...] [1] [...]
````
- **EN**: Comment documenting `/   h -> [ ... ] [...] [1] [...]`.
- **CN**: 注释说明了 `/   h -> [ ... ] [...] [1] [...]`。

### Line 72
````cpp
///   j -> [ ... ] [...] [N] [...]
````
- **EN**: Comment documenting `/   j -> [ ... ] [...] [N] [...]`.
- **CN**: 注释说明了 `/   j -> [ ... ] [...] [N] [...]`。

### Line 73
````cpp
///   j' -> [ ... ] [...] [10] [...]
````
- **EN**: Comment documenting `/   j' -> [ ... ] [...] [10] [...]`.
- **CN**: 注释说明了 `/   j' -> [ ... ] [...] [10] [...]`。

### Line 74
````cpp
///
````
- **EN**: Banner comment delimiting a file header or major section.
- **CN**: 横幅注释，用于分隔文件头或主要章节。

### Line 75
````cpp
/// Note that we'll have a new node representing the path `f -> g -> j'` with
````
- **EN**: Comment documenting `/ Note that we'll have a new node representing the path `f -> g -> j'` with`.
- **CN**: 注释说明了 `/ Note that we'll have a new node representing the path `f -> g -> j'` with`。

### Line 76
````cpp
/// isolated data. This isolation gives us a means of representing the stack
````
- **EN**: Comment documenting `/ isolated data. This isolation gives us a means of representing the stack`.
- **CN**: 注释说明了 `/ isolated data. This isolation gives us a means of representing the stack`。

### Line 77
````cpp
/// traces as a path, as opposed to a key in a table. The alternative
````
- **EN**: Comment documenting `/ traces as a path, as opposed to a key in a table. The alternative`.
- **CN**: 注释说明了 `/ traces as a path, as opposed to a key in a table. The alternative`。

### Line 78
````cpp
/// implementation here would be to use a separate table for the path, and use
````
- **EN**: Comment documenting `/ implementation here would be to use a separate table for the path, and use`.
- **CN**: 注释说明了 `/ implementation here would be to use a separate table for the path, and use`。

### Line 79
````cpp
/// hashes of the path as an identifier to accumulate the information. We've
````
- **EN**: Comment documenting `/ hashes of the path as an identifier to accumulate the information. We've`.
- **CN**: 注释说明了 `/ hashes of the path as an identifier to accumulate the information. We've`。

### Line 80
````cpp
/// moved away from this approach as it takes a lot of time to compute the hash
````
- **EN**: Comment documenting `/ moved away from this approach as it takes a lot of time to compute the hash`.
- **CN**: 注释说明了 `/ moved away from this approach as it takes a lot of time to compute the hash`。

### Line 81
````cpp
/// every time we need to update a function's call information as we're handling
````
- **EN**: Comment documenting `/ every time we need to update a function's call information as we're handling`.
- **CN**: 注释说明了 `/ every time we need to update a function's call information as we're handling`。

### Line 82
````cpp
/// the entry and exit events.
````
- **EN**: Comment documenting `/ the entry and exit events.`.
- **CN**: 注释说明了 `/ the entry and exit events.`。

### Line 83
````cpp
///
````
- **EN**: Banner comment delimiting a file header or major section.
- **CN**: 横幅注释，用于分隔文件头或主要章节。

### Line 84
````cpp
/// This approach allows us to maintain a shadow stack, which represents the
````
- **EN**: Comment documenting `/ This approach allows us to maintain a shadow stack, which represents the`.
- **CN**: 注释说明了 `/ This approach allows us to maintain a shadow stack, which represents the`。

### Line 85
````cpp
/// currently executing path, and on function exits quickly compute the amount
````
- **EN**: Comment documenting `/ currently executing path, and on function exits quickly compute the amount`.
- **CN**: 注释说明了 `/ currently executing path, and on function exits quickly compute the amount`。

### Line 86
````cpp
/// of time elapsed from the entry, then update the counters for the node
````
- **EN**: Comment documenting `/ of time elapsed from the entry, then update the counters for the node`.
- **CN**: 注释说明了 `/ of time elapsed from the entry, then update the counters for the node`。

### Line 87
````cpp
/// already represented in the trie. This necessitates an efficient
````
- **EN**: Comment documenting `/ already represented in the trie. This necessitates an efficient`.
- **CN**: 注释说明了 `/ already represented in the trie. This necessitates an efficient`。

### Line 88
````cpp
/// representation of the various data structures (the list of callees must be
````
- **EN**: Comment documenting `/ representation of the various data structures (the list of callees must be`.
- **CN**: 注释说明了 `/ representation of the various data structures (the list of callees must be`。

### Line 89
````cpp
/// cache-aware and efficient to look up, and the histogram must be compact and
````
- **EN**: Comment documenting `/ cache-aware and efficient to look up, and the histogram must be compact and`.
- **CN**: 注释说明了 `/ cache-aware and efficient to look up, and the histogram must be compact and`。

### Line 90
````cpp
/// quick to update) to enable us to keep the overheads of this implementation
````
- **EN**: Comment documenting `/ quick to update) to enable us to keep the overheads of this implementation`.
- **CN**: 注释说明了 `/ quick to update) to enable us to keep the overheads of this implementation`。

### Line 91
````cpp
/// to the minimum.
````
- **EN**: Comment documenting `/ to the minimum.`.
- **CN**: 注释说明了 `/ to the minimum.`。

### Line 92
````cpp
class FunctionCallTrie {
````
- **EN**: Declares the class `FunctionCallTrie`.
- **CN**: 声明 class `FunctionCallTrie`。

### Line 93
````cpp
public:
````
- **EN**: Sets the C++ access level to `public`.
- **CN**: 将 C++ 访问级别设置为 `public`。

### Line 94
````cpp
  struct Node;
````
- **EN**: Declares the struct `Node`.
- **CN**: 声明 struct `Node`。

### Line 95
````cpp
<blank>
````
- **EN**: Blank line separating logical sections.
- **CN**: 空行，用于分隔逻辑段落。

### Line 96
````cpp
  // We use a NodeIdPair type instead of a std::pair<...> to not rely on the
````
- **EN**: Comment documenting `We use a NodeIdPair type instead of a std::pair<...> to not rely on the`.
- **CN**: 注释说明了 `We use a NodeIdPair type instead of a std::pair<...> to not rely on the`。

### Line 97
````cpp
  // standard library types in this header.
````
- **EN**: Comment documenting `standard library types in this header.`.
- **CN**: 注释说明了 `standard library types in this header.`。

### Line 98
````cpp
  struct NodeIdPair {
````
- **EN**: Declares the struct `NodeIdPair`.
- **CN**: 声明 struct `NodeIdPair`。

### Line 99
````cpp
    Node *NodePtr;
````
- **EN**: Executes or declares `Node *NodePtr;` within the current scope.
- **CN**: 在当前作用域中执行或声明 `Node *NodePtr;`。

### Line 100
````cpp
    int32_t FId;
````
- **EN**: Executes or declares `int32_t FId;` within the current scope.
- **CN**: 在当前作用域中执行或声明 `int32_t FId;`。

### Line 101
````cpp
  };
````
- **EN**: Closes a type definition or aggregate block.
- **CN**: 结束一个类型定义或聚合块。

### Line 102
````cpp
<blank>
````
- **EN**: Blank line separating logical sections.
- **CN**: 空行，用于分隔逻辑段落。

### Line 103
````cpp
  using NodeIdPairArray = Array<NodeIdPair>;
````
- **EN**: Introduces a type alias or using-declaration: `using NodeIdPairArray = Array<NodeIdPair>;`.
- **CN**: 引入类型别名或 using 声明：`using NodeIdPairArray = Array<NodeIdPair>;`。

### Line 104
````cpp
  using NodeIdPairAllocatorType = NodeIdPairArray::AllocatorType;
````
- **EN**: Introduces a type alias or using-declaration: `using NodeIdPairAllocatorType = NodeIdPairArray::AllocatorType;`.
- **CN**: 引入类型别名或 using 声明：`using NodeIdPairAllocatorType = NodeIdPairArray::AllocatorType;`。

### Line 105
````cpp
<blank>
````
- **EN**: Blank line separating logical sections.
- **CN**: 空行，用于分隔逻辑段落。

### Line 106
````cpp
  // A Node in the FunctionCallTrie gives us a list of callees, the cumulative
````
- **EN**: Comment documenting `A Node in the FunctionCallTrie gives us a list of callees, the cumulative`.
- **CN**: 注释说明了 `A Node in the FunctionCallTrie gives us a list of callees, the cumulative`。

### Line 107
````cpp
  // number of times this node actually appeared, the cumulative amount of time
````
- **EN**: Comment documenting `number of times this node actually appeared, the cumulative amount of time`.
- **CN**: 注释说明了 `number of times this node actually appeared, the cumulative amount of time`。

### Line 108
````cpp
  // for this particular node including its children call times, and just the
````
- **EN**: Comment documenting `for this particular node including its children call times, and just the`.
- **CN**: 注释说明了 `for this particular node including its children call times, and just the`。

### Line 109
````cpp
  // local time spent on this node. Each Node will have the ID of the XRay
````
- **EN**: Comment documenting `local time spent on this node. Each Node will have the ID of the XRay`.
- **CN**: 注释说明了 `local time spent on this node. Each Node will have the ID of the XRay`。

### Line 110
````cpp
  // instrumented function that it is associated to.
````
- **EN**: Comment documenting `instrumented function that it is associated to.`.
- **CN**: 注释说明了 `instrumented function that it is associated to.`。

### Line 111
````cpp
  struct Node {
````
- **EN**: Declares the struct `Node`.
- **CN**: 声明 struct `Node`。

### Line 112
````cpp
    Node *Parent;
````
- **EN**: Executes or declares `Node *Parent;` within the current scope.
- **CN**: 在当前作用域中执行或声明 `Node *Parent;`。

### Line 113
````cpp
    NodeIdPairArray Callees;
````
- **EN**: Executes or declares `NodeIdPairArray Callees;` within the current scope.
- **CN**: 在当前作用域中执行或声明 `NodeIdPairArray Callees;`。

### Line 114
````cpp
    uint64_t CallCount;
````
- **EN**: Executes or declares `uint64_t CallCount;` within the current scope.
- **CN**: 在当前作用域中执行或声明 `uint64_t CallCount;`。

### Line 115
````cpp
    uint64_t CumulativeLocalTime; // Typically in TSC deltas, not wall-time.
````
- **EN**: Carries part of the local implementation logic: `uint64_t CumulativeLocalTime; // Typically in TSC deltas, not wall-time.`.
- **CN**: 承载局部实现逻辑：`uint64_t CumulativeLocalTime; // Typically in TSC deltas, not wall-time.`。

### Line 116
````cpp
    int32_t FId;
````
- **EN**: Executes or declares `int32_t FId;` within the current scope.
- **CN**: 在当前作用域中执行或声明 `int32_t FId;`。

### Line 117
````cpp
<blank>
````
- **EN**: Blank line separating logical sections.
- **CN**: 空行，用于分隔逻辑段落。

### Line 118
````cpp
    // TODO: Include the compact histogram.
````
- **EN**: Comment recording follow-up work: `TODO: Include the compact histogram.`.
- **CN**: 注释记录后续待办事项：`TODO: Include the compact histogram.`。

### Line 119
````cpp
  };
````
- **EN**: Closes a type definition or aggregate block.
- **CN**: 结束一个类型定义或聚合块。

### Line 120
````cpp
<blank>
````
- **EN**: Blank line separating logical sections.
- **CN**: 空行，用于分隔逻辑段落。

### Line 121
````cpp
private:
````
- **EN**: Sets the C++ access level to `private`.
- **CN**: 将 C++ 访问级别设置为 `private`。

### Line 122
````cpp
  struct ShadowStackEntry {
````
- **EN**: Declares the struct `ShadowStackEntry`.
- **CN**: 声明 struct `ShadowStackEntry`。

### Line 123
````cpp
    uint64_t EntryTSC;
````
- **EN**: Executes or declares `uint64_t EntryTSC;` within the current scope.
- **CN**: 在当前作用域中执行或声明 `uint64_t EntryTSC;`。

### Line 124
````cpp
    Node *NodePtr;
````
- **EN**: Executes or declares `Node *NodePtr;` within the current scope.
- **CN**: 在当前作用域中执行或声明 `Node *NodePtr;`。

### Line 125
````cpp
    uint16_t EntryCPU;
````
- **EN**: Executes or declares `uint16_t EntryCPU;` within the current scope.
- **CN**: 在当前作用域中执行或声明 `uint16_t EntryCPU;`。

### Line 126
````cpp
  };
````
- **EN**: Closes a type definition or aggregate block.
- **CN**: 结束一个类型定义或聚合块。

### Line 127
````cpp
<blank>
````
- **EN**: Blank line separating logical sections.
- **CN**: 空行，用于分隔逻辑段落。

### Line 128
````cpp
  using NodeArray = Array<Node>;
````
- **EN**: Introduces a type alias or using-declaration: `using NodeArray = Array<Node>;`.
- **CN**: 引入类型别名或 using 声明：`using NodeArray = Array<Node>;`。

### Line 129
````cpp
  using RootArray = Array<Node *>;
````
- **EN**: Introduces a type alias or using-declaration: `using RootArray = Array<Node *>;`.
- **CN**: 引入类型别名或 using 声明：`using RootArray = Array<Node *>;`。

### Line 130
````cpp
  using ShadowStackArray = Array<ShadowStackEntry>;
````
- **EN**: Introduces a type alias or using-declaration: `using ShadowStackArray = Array<ShadowStackEntry>;`.
- **CN**: 引入类型别名或 using 声明：`using ShadowStackArray = Array<ShadowStackEntry>;`。

### Line 131
````cpp
<blank>
````
- **EN**: Blank line separating logical sections.
- **CN**: 空行，用于分隔逻辑段落。

### Line 132
````cpp
public:
````
- **EN**: Sets the C++ access level to `public`.
- **CN**: 将 C++ 访问级别设置为 `public`。

### Line 133
````cpp
  // We collate the allocators we need into a single struct, as a convenience to
````
- **EN**: Comment documenting `We collate the allocators we need into a single struct, as a convenience to`.
- **CN**: 注释说明了 `We collate the allocators we need into a single struct, as a convenience to`。

### Line 134
````cpp
  // allow us to initialize these as a group.
````
- **EN**: Comment documenting `allow us to initialize these as a group.`.
- **CN**: 注释说明了 `allow us to initialize these as a group.`。

### Line 135
````cpp
  struct Allocators {
````
- **EN**: Declares the struct `Allocators`.
- **CN**: 声明 struct `Allocators`。

### Line 136
````cpp
    using NodeAllocatorType = NodeArray::AllocatorType;
````
- **EN**: Introduces a type alias or using-declaration: `using NodeAllocatorType = NodeArray::AllocatorType;`.
- **CN**: 引入类型别名或 using 声明：`using NodeAllocatorType = NodeArray::AllocatorType;`。

### Line 137
````cpp
    using RootAllocatorType = RootArray::AllocatorType;
````
- **EN**: Introduces a type alias or using-declaration: `using RootAllocatorType = RootArray::AllocatorType;`.
- **CN**: 引入类型别名或 using 声明：`using RootAllocatorType = RootArray::AllocatorType;`。

### Line 138
````cpp
    using ShadowStackAllocatorType = ShadowStackArray::AllocatorType;
````
- **EN**: Introduces a type alias or using-declaration: `using ShadowStackAllocatorType = ShadowStackArray::AllocatorType;`.
- **CN**: 引入类型别名或 using 声明：`using ShadowStackAllocatorType = ShadowStackArray::AllocatorType;`。

### Line 139
````cpp
<blank>
````
- **EN**: Blank line separating logical sections.
- **CN**: 空行，用于分隔逻辑段落。

### Line 140
````cpp
    // Use hosted aligned storage members to allow for trivial move and init.
````
- **EN**: Comment documenting `Use hosted aligned storage members to allow for trivial move and init.`.
- **CN**: 注释说明了 `Use hosted aligned storage members to allow for trivial move and init.`。

### Line 141
````cpp
    // This also allows us to sidestep the potential-failing allocation issue.
````
- **EN**: Comment documenting `This also allows us to sidestep the potential-failing allocation issue.`.
- **CN**: 注释说明了 `This also allows us to sidestep the potential-failing allocation issue.`。

### Line 142
````cpp
    alignas(NodeAllocatorType) std::byte
````
- **EN**: Carries part of the local implementation logic: `alignas(NodeAllocatorType) std::byte`.
- **CN**: 承载局部实现逻辑：`alignas(NodeAllocatorType) std::byte`。

### Line 143
````cpp
        NodeAllocatorStorage[sizeof(NodeAllocatorType)];
````
- **EN**: Invokes a function-like statement: `NodeAllocatorStorage[sizeof(NodeAllocatorType)];`.
- **CN**: 调用一个类似函数的语句：`NodeAllocatorStorage[sizeof(NodeAllocatorType)];`。

### Line 144
````cpp
    alignas(RootAllocatorType) std::byte
````
- **EN**: Carries part of the local implementation logic: `alignas(RootAllocatorType) std::byte`.
- **CN**: 承载局部实现逻辑：`alignas(RootAllocatorType) std::byte`。

### Line 145
````cpp
        RootAllocatorStorage[sizeof(RootAllocatorType)];
````
- **EN**: Invokes a function-like statement: `RootAllocatorStorage[sizeof(RootAllocatorType)];`.
- **CN**: 调用一个类似函数的语句：`RootAllocatorStorage[sizeof(RootAllocatorType)];`。

### Line 146
````cpp
    alignas(ShadowStackAllocatorType) std::byte
````
- **EN**: Carries part of the local implementation logic: `alignas(ShadowStackAllocatorType) std::byte`.
- **CN**: 承载局部实现逻辑：`alignas(ShadowStackAllocatorType) std::byte`。

### Line 147
````cpp
        ShadowStackAllocatorStorage[sizeof(ShadowStackAllocatorType)];
````
- **EN**: Invokes a function-like statement: `ShadowStackAllocatorStorage[sizeof(ShadowStackAllocatorType)];`.
- **CN**: 调用一个类似函数的语句：`ShadowStackAllocatorStorage[sizeof(ShadowStackAllocatorType)];`。

### Line 148
````cpp
    alignas(NodeIdPairAllocatorType) std::byte
````
- **EN**: Carries part of the local implementation logic: `alignas(NodeIdPairAllocatorType) std::byte`.
- **CN**: 承载局部实现逻辑：`alignas(NodeIdPairAllocatorType) std::byte`。

### Line 149
````cpp
        NodeIdPairAllocatorStorage[sizeof(NodeIdPairAllocatorType)];
````
- **EN**: Invokes a function-like statement: `NodeIdPairAllocatorStorage[sizeof(NodeIdPairAllocatorType)];`.
- **CN**: 调用一个类似函数的语句：`NodeIdPairAllocatorStorage[sizeof(NodeIdPairAllocatorType)];`。

### Line 150
````cpp
<blank>
````
- **EN**: Blank line separating logical sections.
- **CN**: 空行，用于分隔逻辑段落。

### Line 151
````cpp
    NodeAllocatorType *NodeAllocator = nullptr;
````
- **EN**: Assigns or initializes state with `NodeAllocatorType *NodeAllocator = nullptr;`.
- **CN**: 使用 `NodeAllocatorType *NodeAllocator = nullptr;` 进行赋值或初始化。

### Line 152
````cpp
    RootAllocatorType *RootAllocator = nullptr;
````
- **EN**: Assigns or initializes state with `RootAllocatorType *RootAllocator = nullptr;`.
- **CN**: 使用 `RootAllocatorType *RootAllocator = nullptr;` 进行赋值或初始化。

### Line 153
````cpp
    ShadowStackAllocatorType *ShadowStackAllocator = nullptr;
````
- **EN**: Assigns or initializes state with `ShadowStackAllocatorType *ShadowStackAllocator = nullptr;`.
- **CN**: 使用 `ShadowStackAllocatorType *ShadowStackAllocator = nullptr;` 进行赋值或初始化。

### Line 154
````cpp
    NodeIdPairAllocatorType *NodeIdPairAllocator = nullptr;
````
- **EN**: Assigns or initializes state with `NodeIdPairAllocatorType *NodeIdPairAllocator = nullptr;`.
- **CN**: 使用 `NodeIdPairAllocatorType *NodeIdPairAllocator = nullptr;` 进行赋值或初始化。

### Line 155
````cpp
<blank>
````
- **EN**: Blank line separating logical sections.
- **CN**: 空行，用于分隔逻辑段落。

### Line 156
````cpp
    Allocators() = default;
````
- **EN**: Invokes a function-like statement: `Allocators() = default;`.
- **CN**: 调用一个类似函数的语句：`Allocators() = default;`。

### Line 157
````cpp
    Allocators(const Allocators &) = delete;
````
- **EN**: Invokes a function-like statement: `Allocators(const Allocators &) = delete;`.
- **CN**: 调用一个类似函数的语句：`Allocators(const Allocators &) = delete;`。

### Line 158
````cpp
    Allocators &operator=(const Allocators &) = delete;
````
- **EN**: Invokes a function-like statement: `Allocators &operator=(const Allocators &) = delete;`.
- **CN**: 调用一个类似函数的语句：`Allocators &operator=(const Allocators &) = delete;`。

### Line 159
````cpp
<blank>
````
- **EN**: Blank line separating logical sections.
- **CN**: 空行，用于分隔逻辑段落。

### Line 160
````cpp
    struct Buffers {
````
- **EN**: Declares the struct `Buffers`.
- **CN**: 声明 struct `Buffers`。

### Line 161
````cpp
      BufferQueue::Buffer NodeBuffer;
````
- **EN**: Executes or declares `BufferQueue::Buffer NodeBuffer;` within the current scope.
- **CN**: 在当前作用域中执行或声明 `BufferQueue::Buffer NodeBuffer;`。

### Line 162
````cpp
      BufferQueue::Buffer RootsBuffer;
````
- **EN**: Executes or declares `BufferQueue::Buffer RootsBuffer;` within the current scope.
- **CN**: 在当前作用域中执行或声明 `BufferQueue::Buffer RootsBuffer;`。

### Line 163
````cpp
      BufferQueue::Buffer ShadowStackBuffer;
````
- **EN**: Executes or declares `BufferQueue::Buffer ShadowStackBuffer;` within the current scope.
- **CN**: 在当前作用域中执行或声明 `BufferQueue::Buffer ShadowStackBuffer;`。

### Line 164
````cpp
      BufferQueue::Buffer NodeIdPairBuffer;
````
- **EN**: Executes or declares `BufferQueue::Buffer NodeIdPairBuffer;` within the current scope.
- **CN**: 在当前作用域中执行或声明 `BufferQueue::Buffer NodeIdPairBuffer;`。

### Line 165
````cpp
    };
````
- **EN**: Closes a type definition or aggregate block.
- **CN**: 结束一个类型定义或聚合块。

### Line 166
````cpp
<blank>
````
- **EN**: Blank line separating logical sections.
- **CN**: 空行，用于分隔逻辑段落。

### Line 167
````cpp
    explicit Allocators(Buffers &B) XRAY_NEVER_INSTRUMENT {
````
- **EN**: Carries part of the local implementation logic: `explicit Allocators(Buffers &B) XRAY_NEVER_INSTRUMENT {`.
- **CN**: 承载局部实现逻辑：`explicit Allocators(Buffers &B) XRAY_NEVER_INSTRUMENT {`。

### Line 168
````cpp
      new (&NodeAllocatorStorage)
````
- **EN**: Carries part of the local implementation logic: `new (&NodeAllocatorStorage)`.
- **CN**: 承载局部实现逻辑：`new (&NodeAllocatorStorage)`。

### Line 169
````cpp
          NodeAllocatorType(B.NodeBuffer.Data, B.NodeBuffer.Size);
````
- **EN**: Invokes a function-like statement: `NodeAllocatorType(B.NodeBuffer.Data, B.NodeBuffer.Size);`.
- **CN**: 调用一个类似函数的语句：`NodeAllocatorType(B.NodeBuffer.Data, B.NodeBuffer.Size);`。

### Line 170
````cpp
      NodeAllocator =
````
- **EN**: Carries part of the local implementation logic: `NodeAllocator =`.
- **CN**: 承载局部实现逻辑：`NodeAllocator =`。

### Line 171
````cpp
          reinterpret_cast<NodeAllocatorType *>(&NodeAllocatorStorage);
````
- **EN**: Invokes a function-like statement: `reinterpret_cast<NodeAllocatorType *>(&NodeAllocatorStorage);`.
- **CN**: 调用一个类似函数的语句：`reinterpret_cast<NodeAllocatorType *>(&NodeAllocatorStorage);`。

### Line 172
````cpp
<blank>
````
- **EN**: Blank line separating logical sections.
- **CN**: 空行，用于分隔逻辑段落。

### Line 173
````cpp
      new (&RootAllocatorStorage)
````
- **EN**: Carries part of the local implementation logic: `new (&RootAllocatorStorage)`.
- **CN**: 承载局部实现逻辑：`new (&RootAllocatorStorage)`。

### Line 174
````cpp
          RootAllocatorType(B.RootsBuffer.Data, B.RootsBuffer.Size);
````
- **EN**: Invokes a function-like statement: `RootAllocatorType(B.RootsBuffer.Data, B.RootsBuffer.Size);`.
- **CN**: 调用一个类似函数的语句：`RootAllocatorType(B.RootsBuffer.Data, B.RootsBuffer.Size);`。

### Line 175
````cpp
      RootAllocator =
````
- **EN**: Carries part of the local implementation logic: `RootAllocator =`.
- **CN**: 承载局部实现逻辑：`RootAllocator =`。

### Line 176
````cpp
          reinterpret_cast<RootAllocatorType *>(&RootAllocatorStorage);
````
- **EN**: Invokes a function-like statement: `reinterpret_cast<RootAllocatorType *>(&RootAllocatorStorage);`.
- **CN**: 调用一个类似函数的语句：`reinterpret_cast<RootAllocatorType *>(&RootAllocatorStorage);`。

### Line 177
````cpp
<blank>
````
- **EN**: Blank line separating logical sections.
- **CN**: 空行，用于分隔逻辑段落。

### Line 178
````cpp
      new (&ShadowStackAllocatorStorage) ShadowStackAllocatorType(
````
- **EN**: Carries part of the local implementation logic: `new (&ShadowStackAllocatorStorage) ShadowStackAllocatorType(`.
- **CN**: 承载局部实现逻辑：`new (&ShadowStackAllocatorStorage) ShadowStackAllocatorType(`。

### Line 179
````cpp
          B.ShadowStackBuffer.Data, B.ShadowStackBuffer.Size);
````
- **EN**: Executes or declares `B.ShadowStackBuffer.Data, B.ShadowStackBuffer.Size);` within the current scope.
- **CN**: 在当前作用域中执行或声明 `B.ShadowStackBuffer.Data, B.ShadowStackBuffer.Size);`。

### Line 180
````cpp
      ShadowStackAllocator = reinterpret_cast<ShadowStackAllocatorType *>(
````
- **EN**: Carries part of the local implementation logic: `ShadowStackAllocator = reinterpret_cast<ShadowStackAllocatorType *>(`.
- **CN**: 承载局部实现逻辑：`ShadowStackAllocator = reinterpret_cast<ShadowStackAllocatorType *>(`。

### Line 181
````cpp
          &ShadowStackAllocatorStorage);
````
- **EN**: Executes or declares `&ShadowStackAllocatorStorage);` within the current scope.
- **CN**: 在当前作用域中执行或声明 `&ShadowStackAllocatorStorage);`。

### Line 182
````cpp
<blank>
````
- **EN**: Blank line separating logical sections.
- **CN**: 空行，用于分隔逻辑段落。

### Line 183
````cpp
      new (&NodeIdPairAllocatorStorage) NodeIdPairAllocatorType(
````
- **EN**: Carries part of the local implementation logic: `new (&NodeIdPairAllocatorStorage) NodeIdPairAllocatorType(`.
- **CN**: 承载局部实现逻辑：`new (&NodeIdPairAllocatorStorage) NodeIdPairAllocatorType(`。

### Line 184
````cpp
          B.NodeIdPairBuffer.Data, B.NodeIdPairBuffer.Size);
````
- **EN**: Executes or declares `B.NodeIdPairBuffer.Data, B.NodeIdPairBuffer.Size);` within the current scope.
- **CN**: 在当前作用域中执行或声明 `B.NodeIdPairBuffer.Data, B.NodeIdPairBuffer.Size);`。

### Line 185
````cpp
      NodeIdPairAllocator = reinterpret_cast<NodeIdPairAllocatorType *>(
````
- **EN**: Carries part of the local implementation logic: `NodeIdPairAllocator = reinterpret_cast<NodeIdPairAllocatorType *>(`.
- **CN**: 承载局部实现逻辑：`NodeIdPairAllocator = reinterpret_cast<NodeIdPairAllocatorType *>(`。

### Line 186
````cpp
          &NodeIdPairAllocatorStorage);
````
- **EN**: Executes or declares `&NodeIdPairAllocatorStorage);` within the current scope.
- **CN**: 在当前作用域中执行或声明 `&NodeIdPairAllocatorStorage);`。

### Line 187
````cpp
    }
````
- **EN**: Closes the current scope block.
- **CN**: 结束当前作用域块。

### Line 188
````cpp
<blank>
````
- **EN**: Blank line separating logical sections.
- **CN**: 空行，用于分隔逻辑段落。

### Line 189
````cpp
    explicit Allocators(uptr Max) XRAY_NEVER_INSTRUMENT {
````
- **EN**: Carries part of the local implementation logic: `explicit Allocators(uptr Max) XRAY_NEVER_INSTRUMENT {`.
- **CN**: 承载局部实现逻辑：`explicit Allocators(uptr Max) XRAY_NEVER_INSTRUMENT {`。

### Line 190
````cpp
      new (&NodeAllocatorStorage) NodeAllocatorType(Max);
````
- **EN**: Invokes a function-like statement: `new (&NodeAllocatorStorage) NodeAllocatorType(Max);`.
- **CN**: 调用一个类似函数的语句：`new (&NodeAllocatorStorage) NodeAllocatorType(Max);`。

### Line 191
````cpp
      NodeAllocator =
````
- **EN**: Carries part of the local implementation logic: `NodeAllocator =`.
- **CN**: 承载局部实现逻辑：`NodeAllocator =`。

### Line 192
````cpp
          reinterpret_cast<NodeAllocatorType *>(&NodeAllocatorStorage);
````
- **EN**: Invokes a function-like statement: `reinterpret_cast<NodeAllocatorType *>(&NodeAllocatorStorage);`.
- **CN**: 调用一个类似函数的语句：`reinterpret_cast<NodeAllocatorType *>(&NodeAllocatorStorage);`。

### Line 193
````cpp
<blank>
````
- **EN**: Blank line separating logical sections.
- **CN**: 空行，用于分隔逻辑段落。

### Line 194
````cpp
      new (&RootAllocatorStorage) RootAllocatorType(Max);
````
- **EN**: Invokes a function-like statement: `new (&RootAllocatorStorage) RootAllocatorType(Max);`.
- **CN**: 调用一个类似函数的语句：`new (&RootAllocatorStorage) RootAllocatorType(Max);`。

### Line 195
````cpp
      RootAllocator =
````
- **EN**: Carries part of the local implementation logic: `RootAllocator =`.
- **CN**: 承载局部实现逻辑：`RootAllocator =`。

### Line 196
````cpp
          reinterpret_cast<RootAllocatorType *>(&RootAllocatorStorage);
````
- **EN**: Invokes a function-like statement: `reinterpret_cast<RootAllocatorType *>(&RootAllocatorStorage);`.
- **CN**: 调用一个类似函数的语句：`reinterpret_cast<RootAllocatorType *>(&RootAllocatorStorage);`。

### Line 197
````cpp
<blank>
````
- **EN**: Blank line separating logical sections.
- **CN**: 空行，用于分隔逻辑段落。

### Line 198
````cpp
      new (&ShadowStackAllocatorStorage) ShadowStackAllocatorType(Max);
````
- **EN**: Invokes a function-like statement: `new (&ShadowStackAllocatorStorage) ShadowStackAllocatorType(Max);`.
- **CN**: 调用一个类似函数的语句：`new (&ShadowStackAllocatorStorage) ShadowStackAllocatorType(Max);`。

### Line 199
````cpp
      ShadowStackAllocator = reinterpret_cast<ShadowStackAllocatorType *>(
````
- **EN**: Carries part of the local implementation logic: `ShadowStackAllocator = reinterpret_cast<ShadowStackAllocatorType *>(`.
- **CN**: 承载局部实现逻辑：`ShadowStackAllocator = reinterpret_cast<ShadowStackAllocatorType *>(`。

### Line 200
````cpp
          &ShadowStackAllocatorStorage);
````
- **EN**: Executes or declares `&ShadowStackAllocatorStorage);` within the current scope.
- **CN**: 在当前作用域中执行或声明 `&ShadowStackAllocatorStorage);`。

### Line 201
````cpp
<blank>
````
- **EN**: Blank line separating logical sections.
- **CN**: 空行，用于分隔逻辑段落。

### Line 202
````cpp
      new (&NodeIdPairAllocatorStorage) NodeIdPairAllocatorType(Max);
````
- **EN**: Invokes a function-like statement: `new (&NodeIdPairAllocatorStorage) NodeIdPairAllocatorType(Max);`.
- **CN**: 调用一个类似函数的语句：`new (&NodeIdPairAllocatorStorage) NodeIdPairAllocatorType(Max);`。

### Line 203
````cpp
      NodeIdPairAllocator = reinterpret_cast<NodeIdPairAllocatorType *>(
````
- **EN**: Carries part of the local implementation logic: `NodeIdPairAllocator = reinterpret_cast<NodeIdPairAllocatorType *>(`.
- **CN**: 承载局部实现逻辑：`NodeIdPairAllocator = reinterpret_cast<NodeIdPairAllocatorType *>(`。

### Line 204
````cpp
          &NodeIdPairAllocatorStorage);
````
- **EN**: Executes or declares `&NodeIdPairAllocatorStorage);` within the current scope.
- **CN**: 在当前作用域中执行或声明 `&NodeIdPairAllocatorStorage);`。

### Line 205
````cpp
    }
````
- **EN**: Closes the current scope block.
- **CN**: 结束当前作用域块。

### Line 206
````cpp
<blank>
````
- **EN**: Blank line separating logical sections.
- **CN**: 空行，用于分隔逻辑段落。

### Line 207
````cpp
    Allocators(Allocators &&O) XRAY_NEVER_INSTRUMENT {
````
- **EN**: Carries part of the local implementation logic: `Allocators(Allocators &&O) XRAY_NEVER_INSTRUMENT {`.
- **CN**: 承载局部实现逻辑：`Allocators(Allocators &&O) XRAY_NEVER_INSTRUMENT {`。

### Line 208
````cpp
      // Here we rely on the safety of memcpy'ing contents of the storage
````
- **EN**: Comment documenting `Here we rely on the safety of memcpy'ing contents of the storage`.
- **CN**: 注释说明了 `Here we rely on the safety of memcpy'ing contents of the storage`。

### Line 209
````cpp
      // members, and then pointing the source pointers to nullptr.
````
- **EN**: Comment documenting `members, and then pointing the source pointers to nullptr.`.
- **CN**: 注释说明了 `members, and then pointing the source pointers to nullptr.`。

### Line 210
````cpp
      internal_memcpy(&NodeAllocatorStorage, &O.NodeAllocatorStorage,
````
- **EN**: Carries part of the local implementation logic: `internal_memcpy(&NodeAllocatorStorage, &O.NodeAllocatorStorage,`.
- **CN**: 承载局部实现逻辑：`internal_memcpy(&NodeAllocatorStorage, &O.NodeAllocatorStorage,`。

### Line 211
````cpp
                      sizeof(NodeAllocatorType));
````
- **EN**: Declares an interface element or prototype: `sizeof(NodeAllocatorType));`.
- **CN**: 声明一个接口元素或原型：`sizeof(NodeAllocatorType));`。

### Line 212
````cpp
      internal_memcpy(&RootAllocatorStorage, &O.RootAllocatorStorage,
````
- **EN**: Carries part of the local implementation logic: `internal_memcpy(&RootAllocatorStorage, &O.RootAllocatorStorage,`.
- **CN**: 承载局部实现逻辑：`internal_memcpy(&RootAllocatorStorage, &O.RootAllocatorStorage,`。

### Line 213
````cpp
                      sizeof(RootAllocatorType));
````
- **EN**: Declares an interface element or prototype: `sizeof(RootAllocatorType));`.
- **CN**: 声明一个接口元素或原型：`sizeof(RootAllocatorType));`。

### Line 214
````cpp
      internal_memcpy(&ShadowStackAllocatorStorage,
````
- **EN**: Carries part of the local implementation logic: `internal_memcpy(&ShadowStackAllocatorStorage,`.
- **CN**: 承载局部实现逻辑：`internal_memcpy(&ShadowStackAllocatorStorage,`。

### Line 215
````cpp
                      &O.ShadowStackAllocatorStorage,
````
- **EN**: Carries part of the local implementation logic: `&O.ShadowStackAllocatorStorage,`.
- **CN**: 承载局部实现逻辑：`&O.ShadowStackAllocatorStorage,`。

### Line 216
````cpp
                      sizeof(ShadowStackAllocatorType));
````
- **EN**: Declares an interface element or prototype: `sizeof(ShadowStackAllocatorType));`.
- **CN**: 声明一个接口元素或原型：`sizeof(ShadowStackAllocatorType));`。

### Line 217
````cpp
      internal_memcpy(&NodeIdPairAllocatorStorage,
````
- **EN**: Carries part of the local implementation logic: `internal_memcpy(&NodeIdPairAllocatorStorage,`.
- **CN**: 承载局部实现逻辑：`internal_memcpy(&NodeIdPairAllocatorStorage,`。

### Line 218
````cpp
                      &O.NodeIdPairAllocatorStorage,
````
- **EN**: Carries part of the local implementation logic: `&O.NodeIdPairAllocatorStorage,`.
- **CN**: 承载局部实现逻辑：`&O.NodeIdPairAllocatorStorage,`。

### Line 219
````cpp
                      sizeof(NodeIdPairAllocatorType));
````
- **EN**: Declares an interface element or prototype: `sizeof(NodeIdPairAllocatorType));`.
- **CN**: 声明一个接口元素或原型：`sizeof(NodeIdPairAllocatorType));`。

### Line 220
````cpp
<blank>
````
- **EN**: Blank line separating logical sections.
- **CN**: 空行，用于分隔逻辑段落。

### Line 221
````cpp
      NodeAllocator =
````
- **EN**: Carries part of the local implementation logic: `NodeAllocator =`.
- **CN**: 承载局部实现逻辑：`NodeAllocator =`。

### Line 222
````cpp
          reinterpret_cast<NodeAllocatorType *>(&NodeAllocatorStorage);
````
- **EN**: Invokes a function-like statement: `reinterpret_cast<NodeAllocatorType *>(&NodeAllocatorStorage);`.
- **CN**: 调用一个类似函数的语句：`reinterpret_cast<NodeAllocatorType *>(&NodeAllocatorStorage);`。

### Line 223
````cpp
      RootAllocator =
````
- **EN**: Carries part of the local implementation logic: `RootAllocator =`.
- **CN**: 承载局部实现逻辑：`RootAllocator =`。

### Line 224
````cpp
          reinterpret_cast<RootAllocatorType *>(&RootAllocatorStorage);
````
- **EN**: Invokes a function-like statement: `reinterpret_cast<RootAllocatorType *>(&RootAllocatorStorage);`.
- **CN**: 调用一个类似函数的语句：`reinterpret_cast<RootAllocatorType *>(&RootAllocatorStorage);`。

### Line 225
````cpp
      ShadowStackAllocator = reinterpret_cast<ShadowStackAllocatorType *>(
````
- **EN**: Carries part of the local implementation logic: `ShadowStackAllocator = reinterpret_cast<ShadowStackAllocatorType *>(`.
- **CN**: 承载局部实现逻辑：`ShadowStackAllocator = reinterpret_cast<ShadowStackAllocatorType *>(`。

### Line 226
````cpp
          &ShadowStackAllocatorStorage);
````
- **EN**: Executes or declares `&ShadowStackAllocatorStorage);` within the current scope.
- **CN**: 在当前作用域中执行或声明 `&ShadowStackAllocatorStorage);`。

### Line 227
````cpp
      NodeIdPairAllocator = reinterpret_cast<NodeIdPairAllocatorType *>(
````
- **EN**: Carries part of the local implementation logic: `NodeIdPairAllocator = reinterpret_cast<NodeIdPairAllocatorType *>(`.
- **CN**: 承载局部实现逻辑：`NodeIdPairAllocator = reinterpret_cast<NodeIdPairAllocatorType *>(`。

### Line 228
````cpp
          &NodeIdPairAllocatorStorage);
````
- **EN**: Executes or declares `&NodeIdPairAllocatorStorage);` within the current scope.
- **CN**: 在当前作用域中执行或声明 `&NodeIdPairAllocatorStorage);`。

### Line 229
````cpp
<blank>
````
- **EN**: Blank line separating logical sections.
- **CN**: 空行，用于分隔逻辑段落。

### Line 230
````cpp
      O.NodeAllocator = nullptr;
````
- **EN**: Assigns or initializes state with `O.NodeAllocator = nullptr;`.
- **CN**: 使用 `O.NodeAllocator = nullptr;` 进行赋值或初始化。

### Line 231
````cpp
      O.RootAllocator = nullptr;
````
- **EN**: Assigns or initializes state with `O.RootAllocator = nullptr;`.
- **CN**: 使用 `O.RootAllocator = nullptr;` 进行赋值或初始化。

### Line 232
````cpp
      O.ShadowStackAllocator = nullptr;
````
- **EN**: Assigns or initializes state with `O.ShadowStackAllocator = nullptr;`.
- **CN**: 使用 `O.ShadowStackAllocator = nullptr;` 进行赋值或初始化。

### Line 233
````cpp
      O.NodeIdPairAllocator = nullptr;
````
- **EN**: Assigns or initializes state with `O.NodeIdPairAllocator = nullptr;`.
- **CN**: 使用 `O.NodeIdPairAllocator = nullptr;` 进行赋值或初始化。

### Line 234
````cpp
    }
````
- **EN**: Closes the current scope block.
- **CN**: 结束当前作用域块。

### Line 235
````cpp
<blank>
````
- **EN**: Blank line separating logical sections.
- **CN**: 空行，用于分隔逻辑段落。

### Line 236
````cpp
    Allocators &operator=(Allocators &&O) XRAY_NEVER_INSTRUMENT {
````
- **EN**: Carries part of the local implementation logic: `Allocators &operator=(Allocators &&O) XRAY_NEVER_INSTRUMENT {`.
- **CN**: 承载局部实现逻辑：`Allocators &operator=(Allocators &&O) XRAY_NEVER_INSTRUMENT {`。

### Line 237
````cpp
      // When moving into an existing instance, we ensure that we clean up the
````
- **EN**: Comment documenting `When moving into an existing instance, we ensure that we clean up the`.
- **CN**: 注释说明了 `When moving into an existing instance, we ensure that we clean up the`。

### Line 238
````cpp
      // current allocators.
````
- **EN**: Comment documenting `current allocators.`.
- **CN**: 注释说明了 `current allocators.`。

### Line 239
````cpp
      if (NodeAllocator)
````
- **EN**: Evaluates the conditional branch `if (NodeAllocator)`.
- **CN**: 计算条件分支 `if (NodeAllocator)`。

### Line 240
````cpp
        NodeAllocator->~NodeAllocatorType();
````
- **EN**: Invokes a function-like statement: `NodeAllocator->~NodeAllocatorType();`.
- **CN**: 调用一个类似函数的语句：`NodeAllocator->~NodeAllocatorType();`。

### Line 241
````cpp
      if (O.NodeAllocator) {
````
- **EN**: Evaluates the conditional branch `if (O.NodeAllocator) {`.
- **CN**: 计算条件分支 `if (O.NodeAllocator) {`。

### Line 242
````cpp
        new (&NodeAllocatorStorage)
````
- **EN**: Carries part of the local implementation logic: `new (&NodeAllocatorStorage)`.
- **CN**: 承载局部实现逻辑：`new (&NodeAllocatorStorage)`。

### Line 243
````cpp
            NodeAllocatorType(std::move(*O.NodeAllocator));
````
- **EN**: Declares an interface element or prototype: `NodeAllocatorType(std::move(*O.NodeAllocator));`.
- **CN**: 声明一个接口元素或原型：`NodeAllocatorType(std::move(*O.NodeAllocator));`。

### Line 244
````cpp
        NodeAllocator =
````
- **EN**: Carries part of the local implementation logic: `NodeAllocator =`.
- **CN**: 承载局部实现逻辑：`NodeAllocator =`。

### Line 245
````cpp
            reinterpret_cast<NodeAllocatorType *>(&NodeAllocatorStorage);
````
- **EN**: Invokes a function-like statement: `reinterpret_cast<NodeAllocatorType *>(&NodeAllocatorStorage);`.
- **CN**: 调用一个类似函数的语句：`reinterpret_cast<NodeAllocatorType *>(&NodeAllocatorStorage);`。

### Line 246
````cpp
        O.NodeAllocator = nullptr;
````
- **EN**: Assigns or initializes state with `O.NodeAllocator = nullptr;`.
- **CN**: 使用 `O.NodeAllocator = nullptr;` 进行赋值或初始化。

### Line 247
````cpp
      } else {
````
- **EN**: Carries part of the local implementation logic: `} else {`.
- **CN**: 承载局部实现逻辑：`} else {`。

### Line 248
````cpp
        NodeAllocator = nullptr;
````
- **EN**: Assigns or initializes state with `NodeAllocator = nullptr;`.
- **CN**: 使用 `NodeAllocator = nullptr;` 进行赋值或初始化。

### Line 249
````cpp
      }
````
- **EN**: Closes the current scope block.
- **CN**: 结束当前作用域块。

### Line 250
````cpp
<blank>
````
- **EN**: Blank line separating logical sections.
- **CN**: 空行，用于分隔逻辑段落。

### Line 251
````cpp
      if (RootAllocator)
````
- **EN**: Evaluates the conditional branch `if (RootAllocator)`.
- **CN**: 计算条件分支 `if (RootAllocator)`。

### Line 252
````cpp
        RootAllocator->~RootAllocatorType();
````
- **EN**: Invokes a function-like statement: `RootAllocator->~RootAllocatorType();`.
- **CN**: 调用一个类似函数的语句：`RootAllocator->~RootAllocatorType();`。

### Line 253
````cpp
      if (O.RootAllocator) {
````
- **EN**: Evaluates the conditional branch `if (O.RootAllocator) {`.
- **CN**: 计算条件分支 `if (O.RootAllocator) {`。

### Line 254
````cpp
        new (&RootAllocatorStorage)
````
- **EN**: Carries part of the local implementation logic: `new (&RootAllocatorStorage)`.
- **CN**: 承载局部实现逻辑：`new (&RootAllocatorStorage)`。

### Line 255
````cpp
            RootAllocatorType(std::move(*O.RootAllocator));
````
- **EN**: Declares an interface element or prototype: `RootAllocatorType(std::move(*O.RootAllocator));`.
- **CN**: 声明一个接口元素或原型：`RootAllocatorType(std::move(*O.RootAllocator));`。

### Line 256
````cpp
        RootAllocator =
````
- **EN**: Carries part of the local implementation logic: `RootAllocator =`.
- **CN**: 承载局部实现逻辑：`RootAllocator =`。

### Line 257
````cpp
            reinterpret_cast<RootAllocatorType *>(&RootAllocatorStorage);
````
- **EN**: Invokes a function-like statement: `reinterpret_cast<RootAllocatorType *>(&RootAllocatorStorage);`.
- **CN**: 调用一个类似函数的语句：`reinterpret_cast<RootAllocatorType *>(&RootAllocatorStorage);`。

### Line 258
````cpp
        O.RootAllocator = nullptr;
````
- **EN**: Assigns or initializes state with `O.RootAllocator = nullptr;`.
- **CN**: 使用 `O.RootAllocator = nullptr;` 进行赋值或初始化。

### Line 259
````cpp
      } else {
````
- **EN**: Carries part of the local implementation logic: `} else {`.
- **CN**: 承载局部实现逻辑：`} else {`。

### Line 260
````cpp
        RootAllocator = nullptr;
````
- **EN**: Assigns or initializes state with `RootAllocator = nullptr;`.
- **CN**: 使用 `RootAllocator = nullptr;` 进行赋值或初始化。

### Line 261
````cpp
      }
````
- **EN**: Closes the current scope block.
- **CN**: 结束当前作用域块。

### Line 262
````cpp
<blank>
````
- **EN**: Blank line separating logical sections.
- **CN**: 空行，用于分隔逻辑段落。

### Line 263
````cpp
      if (ShadowStackAllocator)
````
- **EN**: Evaluates the conditional branch `if (ShadowStackAllocator)`.
- **CN**: 计算条件分支 `if (ShadowStackAllocator)`。

### Line 264
````cpp
        ShadowStackAllocator->~ShadowStackAllocatorType();
````
- **EN**: Invokes a function-like statement: `ShadowStackAllocator->~ShadowStackAllocatorType();`.
- **CN**: 调用一个类似函数的语句：`ShadowStackAllocator->~ShadowStackAllocatorType();`。

### Line 265
````cpp
      if (O.ShadowStackAllocator) {
````
- **EN**: Evaluates the conditional branch `if (O.ShadowStackAllocator) {`.
- **CN**: 计算条件分支 `if (O.ShadowStackAllocator) {`。

### Line 266
````cpp
        new (&ShadowStackAllocatorStorage)
````
- **EN**: Carries part of the local implementation logic: `new (&ShadowStackAllocatorStorage)`.
- **CN**: 承载局部实现逻辑：`new (&ShadowStackAllocatorStorage)`。

### Line 267
````cpp
            ShadowStackAllocatorType(std::move(*O.ShadowStackAllocator));
````
- **EN**: Declares an interface element or prototype: `ShadowStackAllocatorType(std::move(*O.ShadowStackAllocator));`.
- **CN**: 声明一个接口元素或原型：`ShadowStackAllocatorType(std::move(*O.ShadowStackAllocator));`。

### Line 268
````cpp
        ShadowStackAllocator = reinterpret_cast<ShadowStackAllocatorType *>(
````
- **EN**: Carries part of the local implementation logic: `ShadowStackAllocator = reinterpret_cast<ShadowStackAllocatorType *>(`.
- **CN**: 承载局部实现逻辑：`ShadowStackAllocator = reinterpret_cast<ShadowStackAllocatorType *>(`。

### Line 269
````cpp
            &ShadowStackAllocatorStorage);
````
- **EN**: Executes or declares `&ShadowStackAllocatorStorage);` within the current scope.
- **CN**: 在当前作用域中执行或声明 `&ShadowStackAllocatorStorage);`。

### Line 270
````cpp
        O.ShadowStackAllocator = nullptr;
````
- **EN**: Assigns or initializes state with `O.ShadowStackAllocator = nullptr;`.
- **CN**: 使用 `O.ShadowStackAllocator = nullptr;` 进行赋值或初始化。

### Line 271
````cpp
      } else {
````
- **EN**: Carries part of the local implementation logic: `} else {`.
- **CN**: 承载局部实现逻辑：`} else {`。

### Line 272
````cpp
        ShadowStackAllocator = nullptr;
````
- **EN**: Assigns or initializes state with `ShadowStackAllocator = nullptr;`.
- **CN**: 使用 `ShadowStackAllocator = nullptr;` 进行赋值或初始化。

### Line 273
````cpp
      }
````
- **EN**: Closes the current scope block.
- **CN**: 结束当前作用域块。

### Line 274
````cpp
<blank>
````
- **EN**: Blank line separating logical sections.
- **CN**: 空行，用于分隔逻辑段落。

### Line 275
````cpp
      if (NodeIdPairAllocator)
````
- **EN**: Evaluates the conditional branch `if (NodeIdPairAllocator)`.
- **CN**: 计算条件分支 `if (NodeIdPairAllocator)`。

### Line 276
````cpp
        NodeIdPairAllocator->~NodeIdPairAllocatorType();
````
- **EN**: Invokes a function-like statement: `NodeIdPairAllocator->~NodeIdPairAllocatorType();`.
- **CN**: 调用一个类似函数的语句：`NodeIdPairAllocator->~NodeIdPairAllocatorType();`。

### Line 277
````cpp
      if (O.NodeIdPairAllocator) {
````
- **EN**: Evaluates the conditional branch `if (O.NodeIdPairAllocator) {`.
- **CN**: 计算条件分支 `if (O.NodeIdPairAllocator) {`。

### Line 278
````cpp
        new (&NodeIdPairAllocatorStorage)
````
- **EN**: Carries part of the local implementation logic: `new (&NodeIdPairAllocatorStorage)`.
- **CN**: 承载局部实现逻辑：`new (&NodeIdPairAllocatorStorage)`。

### Line 279
````cpp
            NodeIdPairAllocatorType(std::move(*O.NodeIdPairAllocator));
````
- **EN**: Declares an interface element or prototype: `NodeIdPairAllocatorType(std::move(*O.NodeIdPairAllocator));`.
- **CN**: 声明一个接口元素或原型：`NodeIdPairAllocatorType(std::move(*O.NodeIdPairAllocator));`。

### Line 280
````cpp
        NodeIdPairAllocator = reinterpret_cast<NodeIdPairAllocatorType *>(
````
- **EN**: Carries part of the local implementation logic: `NodeIdPairAllocator = reinterpret_cast<NodeIdPairAllocatorType *>(`.
- **CN**: 承载局部实现逻辑：`NodeIdPairAllocator = reinterpret_cast<NodeIdPairAllocatorType *>(`。

### Line 281
````cpp
            &NodeIdPairAllocatorStorage);
````
- **EN**: Executes or declares `&NodeIdPairAllocatorStorage);` within the current scope.
- **CN**: 在当前作用域中执行或声明 `&NodeIdPairAllocatorStorage);`。

### Line 282
````cpp
        O.NodeIdPairAllocator = nullptr;
````
- **EN**: Assigns or initializes state with `O.NodeIdPairAllocator = nullptr;`.
- **CN**: 使用 `O.NodeIdPairAllocator = nullptr;` 进行赋值或初始化。

### Line 283
````cpp
      } else {
````
- **EN**: Carries part of the local implementation logic: `} else {`.
- **CN**: 承载局部实现逻辑：`} else {`。

### Line 284
````cpp
        NodeIdPairAllocator = nullptr;
````
- **EN**: Assigns or initializes state with `NodeIdPairAllocator = nullptr;`.
- **CN**: 使用 `NodeIdPairAllocator = nullptr;` 进行赋值或初始化。

### Line 285
````cpp
      }
````
- **EN**: Closes the current scope block.
- **CN**: 结束当前作用域块。

### Line 286
````cpp
<blank>
````
- **EN**: Blank line separating logical sections.
- **CN**: 空行，用于分隔逻辑段落。

### Line 287
````cpp
      return *this;
````
- **EN**: Returns from the current function with `*this;`.
- **CN**: 使用 `*this;` 从当前函数返回。

### Line 288
````cpp
    }
````
- **EN**: Closes the current scope block.
- **CN**: 结束当前作用域块。

### Line 289
````cpp
<blank>
````
- **EN**: Blank line separating logical sections.
- **CN**: 空行，用于分隔逻辑段落。

### Line 290
````cpp
    ~Allocators() XRAY_NEVER_INSTRUMENT {
````
- **EN**: Carries part of the local implementation logic: `~Allocators() XRAY_NEVER_INSTRUMENT {`.
- **CN**: 承载局部实现逻辑：`~Allocators() XRAY_NEVER_INSTRUMENT {`。

### Line 291
````cpp
      if (NodeAllocator != nullptr)
````
- **EN**: Evaluates the conditional branch `if (NodeAllocator != nullptr)`.
- **CN**: 计算条件分支 `if (NodeAllocator != nullptr)`。

### Line 292
````cpp
        NodeAllocator->~NodeAllocatorType();
````
- **EN**: Invokes a function-like statement: `NodeAllocator->~NodeAllocatorType();`.
- **CN**: 调用一个类似函数的语句：`NodeAllocator->~NodeAllocatorType();`。

### Line 293
````cpp
      if (RootAllocator != nullptr)
````
- **EN**: Evaluates the conditional branch `if (RootAllocator != nullptr)`.
- **CN**: 计算条件分支 `if (RootAllocator != nullptr)`。

### Line 294
````cpp
        RootAllocator->~RootAllocatorType();
````
- **EN**: Invokes a function-like statement: `RootAllocator->~RootAllocatorType();`.
- **CN**: 调用一个类似函数的语句：`RootAllocator->~RootAllocatorType();`。

### Line 295
````cpp
      if (ShadowStackAllocator != nullptr)
````
- **EN**: Evaluates the conditional branch `if (ShadowStackAllocator != nullptr)`.
- **CN**: 计算条件分支 `if (ShadowStackAllocator != nullptr)`。

### Line 296
````cpp
        ShadowStackAllocator->~ShadowStackAllocatorType();
````
- **EN**: Invokes a function-like statement: `ShadowStackAllocator->~ShadowStackAllocatorType();`.
- **CN**: 调用一个类似函数的语句：`ShadowStackAllocator->~ShadowStackAllocatorType();`。

### Line 297
````cpp
      if (NodeIdPairAllocator != nullptr)
````
- **EN**: Evaluates the conditional branch `if (NodeIdPairAllocator != nullptr)`.
- **CN**: 计算条件分支 `if (NodeIdPairAllocator != nullptr)`。

### Line 298
````cpp
        NodeIdPairAllocator->~NodeIdPairAllocatorType();
````
- **EN**: Invokes a function-like statement: `NodeIdPairAllocator->~NodeIdPairAllocatorType();`.
- **CN**: 调用一个类似函数的语句：`NodeIdPairAllocator->~NodeIdPairAllocatorType();`。

### Line 299
````cpp
    }
````
- **EN**: Closes the current scope block.
- **CN**: 结束当前作用域块。

### Line 300
````cpp
  };
````
- **EN**: Closes a type definition or aggregate block.
- **CN**: 结束一个类型定义或聚合块。

### Line 301
````cpp
<blank>
````
- **EN**: Blank line separating logical sections.
- **CN**: 空行，用于分隔逻辑段落。

### Line 302
````cpp
  static Allocators InitAllocators() XRAY_NEVER_INSTRUMENT {
````
- **EN**: Carries part of the local implementation logic: `static Allocators InitAllocators() XRAY_NEVER_INSTRUMENT {`.
- **CN**: 承载局部实现逻辑：`static Allocators InitAllocators() XRAY_NEVER_INSTRUMENT {`。

### Line 303
````cpp
    return InitAllocatorsCustom(profilingFlags()->per_thread_allocator_max);
````
- **EN**: Returns from the current function with `InitAllocatorsCustom(profilingFlags()->per_thread_allocator_max);`.
- **CN**: 使用 `InitAllocatorsCustom(profilingFlags()->per_thread_allocator_max);` 从当前函数返回。

### Line 304
````cpp
  }
````
- **EN**: Closes the current scope block.
- **CN**: 结束当前作用域块。

### Line 305
````cpp
<blank>
````
- **EN**: Blank line separating logical sections.
- **CN**: 空行，用于分隔逻辑段落。

### Line 306
````cpp
  static Allocators InitAllocatorsCustom(uptr Max) XRAY_NEVER_INSTRUMENT {
````
- **EN**: Carries part of the local implementation logic: `static Allocators InitAllocatorsCustom(uptr Max) XRAY_NEVER_INSTRUMENT {`.
- **CN**: 承载局部实现逻辑：`static Allocators InitAllocatorsCustom(uptr Max) XRAY_NEVER_INSTRUMENT {`。

### Line 307
````cpp
    Allocators A(Max);
````
- **EN**: Invokes a function-like statement: `Allocators A(Max);`.
- **CN**: 调用一个类似函数的语句：`Allocators A(Max);`。

### Line 308
````cpp
    return A;
````
- **EN**: Returns from the current function with `A;`.
- **CN**: 使用 `A;` 从当前函数返回。

### Line 309
````cpp
  }
````
- **EN**: Closes the current scope block.
- **CN**: 结束当前作用域块。

### Line 310
````cpp
<blank>
````
- **EN**: Blank line separating logical sections.
- **CN**: 空行，用于分隔逻辑段落。

### Line 311
````cpp
  static Allocators
````
- **EN**: Carries part of the local implementation logic: `static Allocators`.
- **CN**: 承载局部实现逻辑：`static Allocators`。

### Line 312
````cpp
  InitAllocatorsFromBuffers(Allocators::Buffers &Bufs) XRAY_NEVER_INSTRUMENT {
````
- **EN**: Carries part of the local implementation logic: `InitAllocatorsFromBuffers(Allocators::Buffers &Bufs) XRAY_NEVER_INSTRUMENT {`.
- **CN**: 承载局部实现逻辑：`InitAllocatorsFromBuffers(Allocators::Buffers &Bufs) XRAY_NEVER_INSTRUMENT {`。

### Line 313
````cpp
    Allocators A(Bufs);
````
- **EN**: Invokes a function-like statement: `Allocators A(Bufs);`.
- **CN**: 调用一个类似函数的语句：`Allocators A(Bufs);`。

### Line 314
````cpp
    return A;
````
- **EN**: Returns from the current function with `A;`.
- **CN**: 使用 `A;` 从当前函数返回。

### Line 315
````cpp
  }
````
- **EN**: Closes the current scope block.
- **CN**: 结束当前作用域块。

### Line 316
````cpp
<blank>
````
- **EN**: Blank line separating logical sections.
- **CN**: 空行，用于分隔逻辑段落。

### Line 317
````cpp
private:
````
- **EN**: Sets the C++ access level to `private`.
- **CN**: 将 C++ 访问级别设置为 `private`。

### Line 318
````cpp
  NodeArray Nodes;
````
- **EN**: Executes or declares `NodeArray Nodes;` within the current scope.
- **CN**: 在当前作用域中执行或声明 `NodeArray Nodes;`。

### Line 319
````cpp
  RootArray Roots;
````
- **EN**: Executes or declares `RootArray Roots;` within the current scope.
- **CN**: 在当前作用域中执行或声明 `RootArray Roots;`。

### Line 320
````cpp
  ShadowStackArray ShadowStack;
````
- **EN**: Executes or declares `ShadowStackArray ShadowStack;` within the current scope.
- **CN**: 在当前作用域中执行或声明 `ShadowStackArray ShadowStack;`。

### Line 321
````cpp
  NodeIdPairAllocatorType *NodeIdPairAllocator;
````
- **EN**: Executes or declares `NodeIdPairAllocatorType *NodeIdPairAllocator;` within the current scope.
- **CN**: 在当前作用域中执行或声明 `NodeIdPairAllocatorType *NodeIdPairAllocator;`。

### Line 322
````cpp
  uint32_t OverflowedFunctions;
````
- **EN**: Executes or declares `uint32_t OverflowedFunctions;` within the current scope.
- **CN**: 在当前作用域中执行或声明 `uint32_t OverflowedFunctions;`。

### Line 323
````cpp
<blank>
````
- **EN**: Blank line separating logical sections.
- **CN**: 空行，用于分隔逻辑段落。

### Line 324
````cpp
public:
````
- **EN**: Sets the C++ access level to `public`.
- **CN**: 将 C++ 访问级别设置为 `public`。

### Line 325
````cpp
  explicit FunctionCallTrie(const Allocators &A) XRAY_NEVER_INSTRUMENT
````
- **EN**: Carries part of the local implementation logic: `explicit FunctionCallTrie(const Allocators &A) XRAY_NEVER_INSTRUMENT`.
- **CN**: 承载局部实现逻辑：`explicit FunctionCallTrie(const Allocators &A) XRAY_NEVER_INSTRUMENT`。

### Line 326
````cpp
      : Nodes(*A.NodeAllocator),
````
- **EN**: Carries part of the local implementation logic: `: Nodes(*A.NodeAllocator),`.
- **CN**: 承载局部实现逻辑：`: Nodes(*A.NodeAllocator),`。

### Line 327
````cpp
        Roots(*A.RootAllocator),
````
- **EN**: Carries part of the local implementation logic: `Roots(*A.RootAllocator),`.
- **CN**: 承载局部实现逻辑：`Roots(*A.RootAllocator),`。

### Line 328
````cpp
        ShadowStack(*A.ShadowStackAllocator),
````
- **EN**: Carries part of the local implementation logic: `ShadowStack(*A.ShadowStackAllocator),`.
- **CN**: 承载局部实现逻辑：`ShadowStack(*A.ShadowStackAllocator),`。

### Line 329
````cpp
        NodeIdPairAllocator(A.NodeIdPairAllocator),
````
- **EN**: Carries part of the local implementation logic: `NodeIdPairAllocator(A.NodeIdPairAllocator),`.
- **CN**: 承载局部实现逻辑：`NodeIdPairAllocator(A.NodeIdPairAllocator),`。

### Line 330
````cpp
        OverflowedFunctions(0) {}
````
- **EN**: Carries part of the local implementation logic: `OverflowedFunctions(0) {}`.
- **CN**: 承载局部实现逻辑：`OverflowedFunctions(0) {}`。

### Line 331
````cpp
<blank>
````
- **EN**: Blank line separating logical sections.
- **CN**: 空行，用于分隔逻辑段落。

### Line 332
````cpp
  FunctionCallTrie() = delete;
````
- **EN**: Invokes a function-like statement: `FunctionCallTrie() = delete;`.
- **CN**: 调用一个类似函数的语句：`FunctionCallTrie() = delete;`。

### Line 333
````cpp
  FunctionCallTrie(const FunctionCallTrie &) = delete;
````
- **EN**: Invokes a function-like statement: `FunctionCallTrie(const FunctionCallTrie &) = delete;`.
- **CN**: 调用一个类似函数的语句：`FunctionCallTrie(const FunctionCallTrie &) = delete;`。

### Line 334
````cpp
  FunctionCallTrie &operator=(const FunctionCallTrie &) = delete;
````
- **EN**: Invokes a function-like statement: `FunctionCallTrie &operator=(const FunctionCallTrie &) = delete;`.
- **CN**: 调用一个类似函数的语句：`FunctionCallTrie &operator=(const FunctionCallTrie &) = delete;`。

### Line 335
````cpp
<blank>
````
- **EN**: Blank line separating logical sections.
- **CN**: 空行，用于分隔逻辑段落。

### Line 336
````cpp
  FunctionCallTrie(FunctionCallTrie &&O) XRAY_NEVER_INSTRUMENT
````
- **EN**: Carries part of the local implementation logic: `FunctionCallTrie(FunctionCallTrie &&O) XRAY_NEVER_INSTRUMENT`.
- **CN**: 承载局部实现逻辑：`FunctionCallTrie(FunctionCallTrie &&O) XRAY_NEVER_INSTRUMENT`。

### Line 337
````cpp
      : Nodes(std::move(O.Nodes)),
````
- **EN**: Carries part of the local implementation logic: `: Nodes(std::move(O.Nodes)),`.
- **CN**: 承载局部实现逻辑：`: Nodes(std::move(O.Nodes)),`。

### Line 338
````cpp
        Roots(std::move(O.Roots)),
````
- **EN**: Carries part of the local implementation logic: `Roots(std::move(O.Roots)),`.
- **CN**: 承载局部实现逻辑：`Roots(std::move(O.Roots)),`。

### Line 339
````cpp
        ShadowStack(std::move(O.ShadowStack)),
````
- **EN**: Carries part of the local implementation logic: `ShadowStack(std::move(O.ShadowStack)),`.
- **CN**: 承载局部实现逻辑：`ShadowStack(std::move(O.ShadowStack)),`。

### Line 340
````cpp
        NodeIdPairAllocator(O.NodeIdPairAllocator),
````
- **EN**: Carries part of the local implementation logic: `NodeIdPairAllocator(O.NodeIdPairAllocator),`.
- **CN**: 承载局部实现逻辑：`NodeIdPairAllocator(O.NodeIdPairAllocator),`。

### Line 341
````cpp
        OverflowedFunctions(O.OverflowedFunctions) {}
````
- **EN**: Carries part of the local implementation logic: `OverflowedFunctions(O.OverflowedFunctions) {}`.
- **CN**: 承载局部实现逻辑：`OverflowedFunctions(O.OverflowedFunctions) {}`。

### Line 342
````cpp
<blank>
````
- **EN**: Blank line separating logical sections.
- **CN**: 空行，用于分隔逻辑段落。

### Line 343
````cpp
  FunctionCallTrie &operator=(FunctionCallTrie &&O) XRAY_NEVER_INSTRUMENT {
````
- **EN**: Carries part of the local implementation logic: `FunctionCallTrie &operator=(FunctionCallTrie &&O) XRAY_NEVER_INSTRUMENT {`.
- **CN**: 承载局部实现逻辑：`FunctionCallTrie &operator=(FunctionCallTrie &&O) XRAY_NEVER_INSTRUMENT {`。

### Line 344
````cpp
    Nodes = std::move(O.Nodes);
````
- **EN**: Declares an interface element or prototype: `Nodes = std::move(O.Nodes);`.
- **CN**: 声明一个接口元素或原型：`Nodes = std::move(O.Nodes);`。

### Line 345
````cpp
    Roots = std::move(O.Roots);
````
- **EN**: Declares an interface element or prototype: `Roots = std::move(O.Roots);`.
- **CN**: 声明一个接口元素或原型：`Roots = std::move(O.Roots);`。

### Line 346
````cpp
    ShadowStack = std::move(O.ShadowStack);
````
- **EN**: Declares an interface element or prototype: `ShadowStack = std::move(O.ShadowStack);`.
- **CN**: 声明一个接口元素或原型：`ShadowStack = std::move(O.ShadowStack);`。

### Line 347
````cpp
    NodeIdPairAllocator = O.NodeIdPairAllocator;
````
- **EN**: Assigns or initializes state with `NodeIdPairAllocator = O.NodeIdPairAllocator;`.
- **CN**: 使用 `NodeIdPairAllocator = O.NodeIdPairAllocator;` 进行赋值或初始化。

### Line 348
````cpp
    OverflowedFunctions = O.OverflowedFunctions;
````
- **EN**: Assigns or initializes state with `OverflowedFunctions = O.OverflowedFunctions;`.
- **CN**: 使用 `OverflowedFunctions = O.OverflowedFunctions;` 进行赋值或初始化。

### Line 349
````cpp
    return *this;
````
- **EN**: Returns from the current function with `*this;`.
- **CN**: 使用 `*this;` 从当前函数返回。

### Line 350
````cpp
  }
````
- **EN**: Closes the current scope block.
- **CN**: 结束当前作用域块。

### Line 351
````cpp
<blank>
````
- **EN**: Blank line separating logical sections.
- **CN**: 空行，用于分隔逻辑段落。

### Line 352
````cpp
  ~FunctionCallTrie() XRAY_NEVER_INSTRUMENT {}
````
- **EN**: Carries part of the local implementation logic: `~FunctionCallTrie() XRAY_NEVER_INSTRUMENT {}`.
- **CN**: 承载局部实现逻辑：`~FunctionCallTrie() XRAY_NEVER_INSTRUMENT {}`。

### Line 353
````cpp
<blank>
````
- **EN**: Blank line separating logical sections.
- **CN**: 空行，用于分隔逻辑段落。

### Line 354
````cpp
  void enterFunction(const int32_t FId, uint64_t TSC,
````
- **EN**: Carries part of the local implementation logic: `void enterFunction(const int32_t FId, uint64_t TSC,`.
- **CN**: 承载局部实现逻辑：`void enterFunction(const int32_t FId, uint64_t TSC,`。

### Line 355
````cpp
                     uint16_t CPU) XRAY_NEVER_INSTRUMENT {
````
- **EN**: Carries part of the local implementation logic: `uint16_t CPU) XRAY_NEVER_INSTRUMENT {`.
- **CN**: 承载局部实现逻辑：`uint16_t CPU) XRAY_NEVER_INSTRUMENT {`。

### Line 356
````cpp
    DCHECK_NE(FId, 0);
````
- **EN**: Invokes a function-like statement: `DCHECK_NE(FId, 0);`.
- **CN**: 调用一个类似函数的语句：`DCHECK_NE(FId, 0);`。

### Line 357
````cpp
<blank>
````
- **EN**: Blank line separating logical sections.
- **CN**: 空行，用于分隔逻辑段落。

### Line 358
````cpp
    // If we're already overflowed the function call stack, do not bother
````
- **EN**: Comment documenting `If we're already overflowed the function call stack, do not bother`.
- **CN**: 注释说明了 `If we're already overflowed the function call stack, do not bother`。

### Line 359
````cpp
    // attempting to record any more function entries.
````
- **EN**: Comment documenting `attempting to record any more function entries.`.
- **CN**: 注释说明了 `attempting to record any more function entries.`。

### Line 360
````cpp
    if (UNLIKELY(OverflowedFunctions)) {
````
- **EN**: Evaluates the conditional branch `if (UNLIKELY(OverflowedFunctions)) {`.
- **CN**: 计算条件分支 `if (UNLIKELY(OverflowedFunctions)) {`。

### Line 361
````cpp
      ++OverflowedFunctions;
````
- **EN**: Executes or declares `++OverflowedFunctions;` within the current scope.
- **CN**: 在当前作用域中执行或声明 `++OverflowedFunctions;`。

### Line 362
````cpp
      return;
````
- **EN**: Returns from the current function with `;`.
- **CN**: 使用 `;` 从当前函数返回。

### Line 363
````cpp
    }
````
- **EN**: Closes the current scope block.
- **CN**: 结束当前作用域块。

### Line 364
````cpp
<blank>
````
- **EN**: Blank line separating logical sections.
- **CN**: 空行，用于分隔逻辑段落。

### Line 365
````cpp
    // If this is the first function we've encountered, we want to set up the
````
- **EN**: Comment documenting `If this is the first function we've encountered, we want to set up the`.
- **CN**: 注释说明了 `If this is the first function we've encountered, we want to set up the`。

### Line 366
````cpp
    // node(s) and treat it as a root.
````
- **EN**: Comment documenting `node(s) and treat it as a root.`.
- **CN**: 注释说明了 `node(s) and treat it as a root.`。

### Line 367
````cpp
    if (UNLIKELY(ShadowStack.empty())) {
````
- **EN**: Evaluates the conditional branch `if (UNLIKELY(ShadowStack.empty())) {`.
- **CN**: 计算条件分支 `if (UNLIKELY(ShadowStack.empty())) {`。

### Line 368
````cpp
      auto *NewRoot = Nodes.AppendEmplace(
````
- **EN**: Carries part of the local implementation logic: `auto *NewRoot = Nodes.AppendEmplace(`.
- **CN**: 承载局部实现逻辑：`auto *NewRoot = Nodes.AppendEmplace(`。

### Line 369
````cpp
          nullptr, NodeIdPairArray(*NodeIdPairAllocator), 0u, 0u, FId);
````
- **EN**: Invokes a function-like statement: `nullptr, NodeIdPairArray(*NodeIdPairAllocator), 0u, 0u, FId);`.
- **CN**: 调用一个类似函数的语句：`nullptr, NodeIdPairArray(*NodeIdPairAllocator), 0u, 0u, FId);`。

### Line 370
````cpp
      if (UNLIKELY(NewRoot == nullptr))
````
- **EN**: Evaluates the conditional branch `if (UNLIKELY(NewRoot == nullptr))`.
- **CN**: 计算条件分支 `if (UNLIKELY(NewRoot == nullptr))`。

### Line 371
````cpp
        return;
````
- **EN**: Returns from the current function with `;`.
- **CN**: 使用 `;` 从当前函数返回。

### Line 372
````cpp
      if (Roots.AppendEmplace(NewRoot) == nullptr) {
````
- **EN**: Evaluates the conditional branch `if (Roots.AppendEmplace(NewRoot) == nullptr) {`.
- **CN**: 计算条件分支 `if (Roots.AppendEmplace(NewRoot) == nullptr) {`。

### Line 373
````cpp
        Nodes.trim(1);
````
- **EN**: Invokes a function-like statement: `Nodes.trim(1);`.
- **CN**: 调用一个类似函数的语句：`Nodes.trim(1);`。

### Line 374
````cpp
        return;
````
- **EN**: Returns from the current function with `;`.
- **CN**: 使用 `;` 从当前函数返回。

### Line 375
````cpp
      }
````
- **EN**: Closes the current scope block.
- **CN**: 结束当前作用域块。

### Line 376
````cpp
      if (ShadowStack.AppendEmplace(TSC, NewRoot, CPU) == nullptr) {
````
- **EN**: Evaluates the conditional branch `if (ShadowStack.AppendEmplace(TSC, NewRoot, CPU) == nullptr) {`.
- **CN**: 计算条件分支 `if (ShadowStack.AppendEmplace(TSC, NewRoot, CPU) == nullptr) {`。

### Line 377
````cpp
        Nodes.trim(1);
````
- **EN**: Invokes a function-like statement: `Nodes.trim(1);`.
- **CN**: 调用一个类似函数的语句：`Nodes.trim(1);`。

### Line 378
````cpp
        Roots.trim(1);
````
- **EN**: Invokes a function-like statement: `Roots.trim(1);`.
- **CN**: 调用一个类似函数的语句：`Roots.trim(1);`。

### Line 379
````cpp
        ++OverflowedFunctions;
````
- **EN**: Executes or declares `++OverflowedFunctions;` within the current scope.
- **CN**: 在当前作用域中执行或声明 `++OverflowedFunctions;`。

### Line 380
````cpp
        return;
````
- **EN**: Returns from the current function with `;`.
- **CN**: 使用 `;` 从当前函数返回。

### Line 381
````cpp
      }
````
- **EN**: Closes the current scope block.
- **CN**: 结束当前作用域块。

### Line 382
````cpp
      return;
````
- **EN**: Returns from the current function with `;`.
- **CN**: 使用 `;` 从当前函数返回。

### Line 383
````cpp
    }
````
- **EN**: Closes the current scope block.
- **CN**: 结束当前作用域块。

### Line 384
````cpp
<blank>
````
- **EN**: Blank line separating logical sections.
- **CN**: 空行，用于分隔逻辑段落。

### Line 385
````cpp
    // From this point on, we require that the stack is not empty.
````
- **EN**: Comment documenting `From this point on, we require that the stack is not empty.`.
- **CN**: 注释说明了 `From this point on, we require that the stack is not empty.`。

### Line 386
````cpp
    DCHECK(!ShadowStack.empty());
````
- **EN**: Invokes a function-like statement: `DCHECK(!ShadowStack.empty());`.
- **CN**: 调用一个类似函数的语句：`DCHECK(!ShadowStack.empty());`。

### Line 387
````cpp
    auto TopNode = ShadowStack.back().NodePtr;
````
- **EN**: Invokes a function-like statement: `auto TopNode = ShadowStack.back().NodePtr;`.
- **CN**: 调用一个类似函数的语句：`auto TopNode = ShadowStack.back().NodePtr;`。

### Line 388
````cpp
    DCHECK_NE(TopNode, nullptr);
````
- **EN**: Invokes a function-like statement: `DCHECK_NE(TopNode, nullptr);`.
- **CN**: 调用一个类似函数的语句：`DCHECK_NE(TopNode, nullptr);`。

### Line 389
````cpp
<blank>
````
- **EN**: Blank line separating logical sections.
- **CN**: 空行，用于分隔逻辑段落。

### Line 390
````cpp
    // If we've seen this callee before, then we access that node and place that
````
- **EN**: Comment documenting `If we've seen this callee before, then we access that node and place that`.
- **CN**: 注释说明了 `If we've seen this callee before, then we access that node and place that`。

### Line 391
````cpp
    // on the top of the stack.
````
- **EN**: Comment documenting `on the top of the stack.`.
- **CN**: 注释说明了 `on the top of the stack.`。

### Line 392
````cpp
    auto* Callee = TopNode->Callees.find_element(
````
- **EN**: Carries part of the local implementation logic: `auto* Callee = TopNode->Callees.find_element(`.
- **CN**: 承载局部实现逻辑：`auto* Callee = TopNode->Callees.find_element(`。

### Line 393
````cpp
        [FId](const NodeIdPair &NR) { return NR.FId == FId; });
````
- **EN**: Invokes a function-like statement: `[FId](const NodeIdPair &NR) { return NR.FId == FId; });`.
- **CN**: 调用一个类似函数的语句：`[FId](const NodeIdPair &NR) { return NR.FId == FId; });`。

### Line 394
````cpp
    if (Callee != nullptr) {
````
- **EN**: Evaluates the conditional branch `if (Callee != nullptr) {`.
- **CN**: 计算条件分支 `if (Callee != nullptr) {`。

### Line 395
````cpp
      CHECK_NE(Callee->NodePtr, nullptr);
````
- **EN**: Invokes a function-like statement: `CHECK_NE(Callee->NodePtr, nullptr);`.
- **CN**: 调用一个类似函数的语句：`CHECK_NE(Callee->NodePtr, nullptr);`。

### Line 396
````cpp
      if (ShadowStack.AppendEmplace(TSC, Callee->NodePtr, CPU) == nullptr)
````
- **EN**: Evaluates the conditional branch `if (ShadowStack.AppendEmplace(TSC, Callee->NodePtr, CPU) == nullptr)`.
- **CN**: 计算条件分支 `if (ShadowStack.AppendEmplace(TSC, Callee->NodePtr, CPU) == nullptr)`。

### Line 397
````cpp
        ++OverflowedFunctions;
````
- **EN**: Executes or declares `++OverflowedFunctions;` within the current scope.
- **CN**: 在当前作用域中执行或声明 `++OverflowedFunctions;`。

### Line 398
````cpp
      return;
````
- **EN**: Returns from the current function with `;`.
- **CN**: 使用 `;` 从当前函数返回。

### Line 399
````cpp
    }
````
- **EN**: Closes the current scope block.
- **CN**: 结束当前作用域块。

### Line 400
````cpp
<blank>
````
- **EN**: Blank line separating logical sections.
- **CN**: 空行，用于分隔逻辑段落。

### Line 401
````cpp
    // This means we've never seen this stack before, create a new node here.
````
- **EN**: Comment documenting `This means we've never seen this stack before, create a new node here.`.
- **CN**: 注释说明了 `This means we've never seen this stack before, create a new node here.`。

### Line 402
````cpp
    auto* NewNode = Nodes.AppendEmplace(
````
- **EN**: Carries part of the local implementation logic: `auto* NewNode = Nodes.AppendEmplace(`.
- **CN**: 承载局部实现逻辑：`auto* NewNode = Nodes.AppendEmplace(`。

### Line 403
````cpp
        TopNode, NodeIdPairArray(*NodeIdPairAllocator), 0u, 0u, FId);
````
- **EN**: Invokes a function-like statement: `TopNode, NodeIdPairArray(*NodeIdPairAllocator), 0u, 0u, FId);`.
- **CN**: 调用一个类似函数的语句：`TopNode, NodeIdPairArray(*NodeIdPairAllocator), 0u, 0u, FId);`。

### Line 404
````cpp
    if (UNLIKELY(NewNode == nullptr))
````
- **EN**: Evaluates the conditional branch `if (UNLIKELY(NewNode == nullptr))`.
- **CN**: 计算条件分支 `if (UNLIKELY(NewNode == nullptr))`。

### Line 405
````cpp
      return;
````
- **EN**: Returns from the current function with `;`.
- **CN**: 使用 `;` 从当前函数返回。

### Line 406
````cpp
    DCHECK_NE(NewNode, nullptr);
````
- **EN**: Invokes a function-like statement: `DCHECK_NE(NewNode, nullptr);`.
- **CN**: 调用一个类似函数的语句：`DCHECK_NE(NewNode, nullptr);`。

### Line 407
````cpp
    TopNode->Callees.AppendEmplace(NewNode, FId);
````
- **EN**: Invokes a function-like statement: `TopNode->Callees.AppendEmplace(NewNode, FId);`.
- **CN**: 调用一个类似函数的语句：`TopNode->Callees.AppendEmplace(NewNode, FId);`。

### Line 408
````cpp
    if (ShadowStack.AppendEmplace(TSC, NewNode, CPU) == nullptr)
````
- **EN**: Evaluates the conditional branch `if (ShadowStack.AppendEmplace(TSC, NewNode, CPU) == nullptr)`.
- **CN**: 计算条件分支 `if (ShadowStack.AppendEmplace(TSC, NewNode, CPU) == nullptr)`。

### Line 409
````cpp
      ++OverflowedFunctions;
````
- **EN**: Executes or declares `++OverflowedFunctions;` within the current scope.
- **CN**: 在当前作用域中执行或声明 `++OverflowedFunctions;`。

### Line 410
````cpp
    return;
````
- **EN**: Returns from the current function with `;`.
- **CN**: 使用 `;` 从当前函数返回。

### Line 411
````cpp
  }
````
- **EN**: Closes the current scope block.
- **CN**: 结束当前作用域块。

### Line 412
````cpp
<blank>
````
- **EN**: Blank line separating logical sections.
- **CN**: 空行，用于分隔逻辑段落。

### Line 413
````cpp
  void exitFunction(int32_t FId, uint64_t TSC,
````
- **EN**: Carries part of the local implementation logic: `void exitFunction(int32_t FId, uint64_t TSC,`.
- **CN**: 承载局部实现逻辑：`void exitFunction(int32_t FId, uint64_t TSC,`。

### Line 414
````cpp
                    uint16_t CPU) XRAY_NEVER_INSTRUMENT {
````
- **EN**: Carries part of the local implementation logic: `uint16_t CPU) XRAY_NEVER_INSTRUMENT {`.
- **CN**: 承载局部实现逻辑：`uint16_t CPU) XRAY_NEVER_INSTRUMENT {`。

### Line 415
````cpp
    // If we're exiting functions that have "overflowed" or don't fit into the
````
- **EN**: Comment documenting `If we're exiting functions that have "overflowed" or don't fit into the`.
- **CN**: 注释说明了 `If we're exiting functions that have "overflowed" or don't fit into the`。

### Line 416
````cpp
    // stack due to allocator constraints, we then decrement that count first.
````
- **EN**: Comment documenting `stack due to allocator constraints, we then decrement that count first.`.
- **CN**: 注释说明了 `stack due to allocator constraints, we then decrement that count first.`。

### Line 417
````cpp
    if (OverflowedFunctions) {
````
- **EN**: Evaluates the conditional branch `if (OverflowedFunctions) {`.
- **CN**: 计算条件分支 `if (OverflowedFunctions) {`。

### Line 418
````cpp
      --OverflowedFunctions;
````
- **EN**: Executes or declares `--OverflowedFunctions;` within the current scope.
- **CN**: 在当前作用域中执行或声明 `--OverflowedFunctions;`。

### Line 419
````cpp
      return;
````
- **EN**: Returns from the current function with `;`.
- **CN**: 使用 `;` 从当前函数返回。

### Line 420
````cpp
    }
````
- **EN**: Closes the current scope block.
- **CN**: 结束当前作用域块。

### Line 421
````cpp
<blank>
````
- **EN**: Blank line separating logical sections.
- **CN**: 空行，用于分隔逻辑段落。

### Line 422
````cpp
    // When we exit a function, we look up the ShadowStack to see whether we've
````
- **EN**: Comment documenting `When we exit a function, we look up the ShadowStack to see whether we've`.
- **CN**: 注释说明了 `When we exit a function, we look up the ShadowStack to see whether we've`。

### Line 423
````cpp
    // entered this function before. We do as little processing here as we can,
````
- **EN**: Comment documenting `entered this function before. We do as little processing here as we can,`.
- **CN**: 注释说明了 `entered this function before. We do as little processing here as we can,`。

### Line 424
````cpp
    // since most of the hard work would have already been done at function
````
- **EN**: Comment documenting `since most of the hard work would have already been done at function`.
- **CN**: 注释说明了 `since most of the hard work would have already been done at function`。

### Line 425
````cpp
    // entry.
````
- **EN**: Comment documenting `entry.`.
- **CN**: 注释说明了 `entry.`。

### Line 426
````cpp
    uint64_t CumulativeTreeTime = 0;
````
- **EN**: Assigns or initializes state with `uint64_t CumulativeTreeTime = 0;`.
- **CN**: 使用 `uint64_t CumulativeTreeTime = 0;` 进行赋值或初始化。

### Line 427
````cpp
<blank>
````
- **EN**: Blank line separating logical sections.
- **CN**: 空行，用于分隔逻辑段落。

### Line 428
````cpp
    while (!ShadowStack.empty()) {
````
- **EN**: Starts a `while` loop: `while (!ShadowStack.empty()) {`.
- **CN**: 开始一个 `while` 循环：`while (!ShadowStack.empty()) {`。

### Line 429
````cpp
      const auto &Top = ShadowStack.back();
````
- **EN**: Declares an interface element or prototype: `const auto &Top = ShadowStack.back();`.
- **CN**: 声明一个接口元素或原型：`const auto &Top = ShadowStack.back();`。

### Line 430
````cpp
      auto TopNode = Top.NodePtr;
````
- **EN**: Assigns or initializes state with `auto TopNode = Top.NodePtr;`.
- **CN**: 使用 `auto TopNode = Top.NodePtr;` 进行赋值或初始化。

### Line 431
````cpp
      DCHECK_NE(TopNode, nullptr);
````
- **EN**: Invokes a function-like statement: `DCHECK_NE(TopNode, nullptr);`.
- **CN**: 调用一个类似函数的语句：`DCHECK_NE(TopNode, nullptr);`。

### Line 432
````cpp
<blank>
````
- **EN**: Blank line separating logical sections.
- **CN**: 空行，用于分隔逻辑段落。

### Line 433
````cpp
      // We may encounter overflow on the TSC we're provided, which may end up
````
- **EN**: Comment documenting `We may encounter overflow on the TSC we're provided, which may end up`.
- **CN**: 注释说明了 `We may encounter overflow on the TSC we're provided, which may end up`。

### Line 434
````cpp
      // being less than the TSC when we first entered the function.
````
- **EN**: Comment documenting `being less than the TSC when we first entered the function.`.
- **CN**: 注释说明了 `being less than the TSC when we first entered the function.`。

### Line 435
````cpp
      //
````
- **EN**: Comment formatting line in a banner or block comment.
- **CN**: 注释格式行，用于组织文件头或注释块。

### Line 436
````cpp
      // To get the accurate measurement of cycles, we need to check whether
````
- **EN**: Comment documenting `To get the accurate measurement of cycles, we need to check whether`.
- **CN**: 注释说明了 `To get the accurate measurement of cycles, we need to check whether`。

### Line 437
````cpp
      // we've overflowed (TSC < Top.EntryTSC) and then account the difference
````
- **EN**: Comment documenting `we've overflowed (TSC < Top.EntryTSC) and then account the difference`.
- **CN**: 注释说明了 `we've overflowed (TSC < Top.EntryTSC) and then account the difference`。

### Line 438
````cpp
      // between the entry TSC and the max for the TSC counter (max of uint64_t)
````
- **EN**: Comment documenting `between the entry TSC and the max for the TSC counter (max of uint64_t)`.
- **CN**: 注释说明了 `between the entry TSC and the max for the TSC counter (max of uint64_t)`。

### Line 439
````cpp
      // then add the value of TSC. We can prove that the maximum delta we will
````
- **EN**: Comment documenting `then add the value of TSC. We can prove that the maximum delta we will`.
- **CN**: 注释说明了 `then add the value of TSC. We can prove that the maximum delta we will`。

### Line 440
````cpp
      // get is at most the 64-bit unsigned value, since the difference between
````
- **EN**: Comment documenting `get is at most the 64-bit unsigned value, since the difference between`.
- **CN**: 注释说明了 `get is at most the 64-bit unsigned value, since the difference between`。

### Line 441
````cpp
      // a TSC of 0 and a Top.EntryTSC of 1 is (numeric_limits<uint64_t>::max()
````
- **EN**: Comment documenting `a TSC of 0 and a Top.EntryTSC of 1 is (numeric_limits<uint64_t>::max()`.
- **CN**: 注释说明了 `a TSC of 0 and a Top.EntryTSC of 1 is (numeric_limits<uint64_t>::max()`。

### Line 442
````cpp
      // - 1) + 1.
````
- **EN**: Comment documenting `1) + 1.`.
- **CN**: 注释说明了 `1) + 1.`。

### Line 443
````cpp
      //
````
- **EN**: Comment formatting line in a banner or block comment.
- **CN**: 注释格式行，用于组织文件头或注释块。

### Line 444
````cpp
      // NOTE: This assumes that TSCs are synchronised across CPUs.
````
- **EN**: Comment documenting `NOTE: This assumes that TSCs are synchronised across CPUs.`.
- **CN**: 注释说明了 `NOTE: This assumes that TSCs are synchronised across CPUs.`。

### Line 445
````cpp
      // TODO: Count the number of times we've seen CPU migrations.
````
- **EN**: Comment recording follow-up work: `TODO: Count the number of times we've seen CPU migrations.`.
- **CN**: 注释记录后续待办事项：`TODO: Count the number of times we've seen CPU migrations.`。

### Line 446
````cpp
      uint64_t LocalTime =
````
- **EN**: Carries part of the local implementation logic: `uint64_t LocalTime =`.
- **CN**: 承载局部实现逻辑：`uint64_t LocalTime =`。

### Line 447
````cpp
          Top.EntryTSC > TSC
````
- **EN**: Carries part of the local implementation logic: `Top.EntryTSC > TSC`.
- **CN**: 承载局部实现逻辑：`Top.EntryTSC > TSC`。

### Line 448
````cpp
              ? (std::numeric_limits<uint64_t>::max() - Top.EntryTSC) + TSC
````
- **EN**: Carries part of the local implementation logic: `? (std::numeric_limits<uint64_t>::max() - Top.EntryTSC) + TSC`.
- **CN**: 承载局部实现逻辑：`? (std::numeric_limits<uint64_t>::max() - Top.EntryTSC) + TSC`。

### Line 449
````cpp
              : TSC - Top.EntryTSC;
````
- **EN**: Executes or declares `: TSC - Top.EntryTSC;` within the current scope.
- **CN**: 在当前作用域中执行或声明 `: TSC - Top.EntryTSC;`。

### Line 450
````cpp
      TopNode->CallCount++;
````
- **EN**: Executes or declares `TopNode->CallCount++;` within the current scope.
- **CN**: 在当前作用域中执行或声明 `TopNode->CallCount++;`。

### Line 451
````cpp
      TopNode->CumulativeLocalTime += LocalTime - CumulativeTreeTime;
````
- **EN**: Assigns or initializes state with `TopNode->CumulativeLocalTime += LocalTime - CumulativeTreeTime;`.
- **CN**: 使用 `TopNode->CumulativeLocalTime += LocalTime - CumulativeTreeTime;` 进行赋值或初始化。

### Line 452
````cpp
      CumulativeTreeTime += LocalTime;
````
- **EN**: Assigns or initializes state with `CumulativeTreeTime += LocalTime;`.
- **CN**: 使用 `CumulativeTreeTime += LocalTime;` 进行赋值或初始化。

### Line 453
````cpp
      ShadowStack.trim(1);
````
- **EN**: Invokes a function-like statement: `ShadowStack.trim(1);`.
- **CN**: 调用一个类似函数的语句：`ShadowStack.trim(1);`。

### Line 454
````cpp
<blank>
````
- **EN**: Blank line separating logical sections.
- **CN**: 空行，用于分隔逻辑段落。

### Line 455
````cpp
      // TODO: Update the histogram for the node.
````
- **EN**: Comment recording follow-up work: `TODO: Update the histogram for the node.`.
- **CN**: 注释记录后续待办事项：`TODO: Update the histogram for the node.`。

### Line 456
````cpp
      if (TopNode->FId == FId)
````
- **EN**: Evaluates the conditional branch `if (TopNode->FId == FId)`.
- **CN**: 计算条件分支 `if (TopNode->FId == FId)`。

### Line 457
````cpp
        break;
````
- **EN**: Breaks out of the current loop or switch.
- **CN**: 跳出当前循环或 switch。

### Line 458
````cpp
    }
````
- **EN**: Closes the current scope block.
- **CN**: 结束当前作用域块。

### Line 459
````cpp
  }
````
- **EN**: Closes the current scope block.
- **CN**: 结束当前作用域块。

### Line 460
````cpp
<blank>
````
- **EN**: Blank line separating logical sections.
- **CN**: 空行，用于分隔逻辑段落。

### Line 461
````cpp
  const RootArray &getRoots() const XRAY_NEVER_INSTRUMENT { return Roots; }
````
- **EN**: Carries part of the local implementation logic: `const RootArray &getRoots() const XRAY_NEVER_INSTRUMENT { return Roots; }`.
- **CN**: 承载局部实现逻辑：`const RootArray &getRoots() const XRAY_NEVER_INSTRUMENT { return Roots; }`。

### Line 462
````cpp
<blank>
````
- **EN**: Blank line separating logical sections.
- **CN**: 空行，用于分隔逻辑段落。

### Line 463
````cpp
  // The deepCopyInto operation will update the provided FunctionCallTrie by
````
- **EN**: Comment documenting `The deepCopyInto operation will update the provided FunctionCallTrie by`.
- **CN**: 注释说明了 `The deepCopyInto operation will update the provided FunctionCallTrie by`。

### Line 464
````cpp
  // re-creating the contents of this particular FunctionCallTrie in the other
````
- **EN**: Comment documenting `re-creating the contents of this particular FunctionCallTrie in the other`.
- **CN**: 注释说明了 `re-creating the contents of this particular FunctionCallTrie in the other`。

### Line 465
````cpp
  // FunctionCallTrie. It will do this using a Depth First Traversal from the
````
- **EN**: Comment documenting `FunctionCallTrie. It will do this using a Depth First Traversal from the`.
- **CN**: 注释说明了 `FunctionCallTrie. It will do this using a Depth First Traversal from the`。

### Line 466
````cpp
  // roots, and while doing so recreating the traversal in the provided
````
- **EN**: Comment documenting `roots, and while doing so recreating the traversal in the provided`.
- **CN**: 注释说明了 `roots, and while doing so recreating the traversal in the provided`。

### Line 467
````cpp
  // FunctionCallTrie.
````
- **EN**: Comment documenting `FunctionCallTrie.`.
- **CN**: 注释说明了 `FunctionCallTrie.`。

### Line 468
````cpp
  //
````
- **EN**: Comment formatting line in a banner or block comment.
- **CN**: 注释格式行，用于组织文件头或注释块。

### Line 469
````cpp
  // This operation will *not* destroy the state in `O`, and thus may cause some
````
- **EN**: Comment documenting `This operation will *not* destroy the state in `O`, and thus may cause some`.
- **CN**: 注释说明了 `This operation will *not* destroy the state in `O`, and thus may cause some`。

### Line 470
````cpp
  // duplicate entries in `O` if it is not empty.
````
- **EN**: Comment documenting `duplicate entries in `O` if it is not empty.`.
- **CN**: 注释说明了 `duplicate entries in `O` if it is not empty.`。

### Line 471
````cpp
  //
````
- **EN**: Comment formatting line in a banner or block comment.
- **CN**: 注释格式行，用于组织文件头或注释块。

### Line 472
````cpp
  // This function is *not* thread-safe, and may require external
````
- **EN**: Comment documenting `This function is *not* thread-safe, and may require external`.
- **CN**: 注释说明了 `This function is *not* thread-safe, and may require external`。

### Line 473
````cpp
  // synchronisation of both "this" and |O|.
````
- **EN**: Comment documenting `synchronisation of both "this" and |O|.`.
- **CN**: 注释说明了 `synchronisation of both "this" and |O|.`。

### Line 474
````cpp
  //
````
- **EN**: Comment formatting line in a banner or block comment.
- **CN**: 注释格式行，用于组织文件头或注释块。

### Line 475
````cpp
  // This function must *not* be called with a non-empty FunctionCallTrie |O|.
````
- **EN**: Comment documenting `This function must *not* be called with a non-empty FunctionCallTrie |O|.`.
- **CN**: 注释说明了 `This function must *not* be called with a non-empty FunctionCallTrie |O|.`。

### Line 476
````cpp
  void deepCopyInto(FunctionCallTrie &O) const XRAY_NEVER_INSTRUMENT {
````
- **EN**: Carries part of the local implementation logic: `void deepCopyInto(FunctionCallTrie &O) const XRAY_NEVER_INSTRUMENT {`.
- **CN**: 承载局部实现逻辑：`void deepCopyInto(FunctionCallTrie &O) const XRAY_NEVER_INSTRUMENT {`。

### Line 477
````cpp
    DCHECK(O.getRoots().empty());
````
- **EN**: Invokes a function-like statement: `DCHECK(O.getRoots().empty());`.
- **CN**: 调用一个类似函数的语句：`DCHECK(O.getRoots().empty());`。

### Line 478
````cpp
<blank>
````
- **EN**: Blank line separating logical sections.
- **CN**: 空行，用于分隔逻辑段落。

### Line 479
````cpp
    // We then push the root into a stack, to use as the parent marker for new
````
- **EN**: Comment documenting `We then push the root into a stack, to use as the parent marker for new`.
- **CN**: 注释说明了 `We then push the root into a stack, to use as the parent marker for new`。

### Line 480
````cpp
    // nodes we push in as we're traversing depth-first down the call tree.
````
- **EN**: Comment documenting `nodes we push in as we're traversing depth-first down the call tree.`.
- **CN**: 注释说明了 `nodes we push in as we're traversing depth-first down the call tree.`。

### Line 481
````cpp
    struct NodeAndParent {
````
- **EN**: Declares the struct `NodeAndParent`.
- **CN**: 声明 struct `NodeAndParent`。

### Line 482
````cpp
      FunctionCallTrie::Node *Node;
````
- **EN**: Executes or declares `FunctionCallTrie::Node *Node;` within the current scope.
- **CN**: 在当前作用域中执行或声明 `FunctionCallTrie::Node *Node;`。

### Line 483
````cpp
      FunctionCallTrie::Node *NewNode;
````
- **EN**: Executes or declares `FunctionCallTrie::Node *NewNode;` within the current scope.
- **CN**: 在当前作用域中执行或声明 `FunctionCallTrie::Node *NewNode;`。

### Line 484
````cpp
    };
````
- **EN**: Closes a type definition or aggregate block.
- **CN**: 结束一个类型定义或聚合块。

### Line 485
````cpp
    using Stack = Array<NodeAndParent>;
````
- **EN**: Introduces a type alias or using-declaration: `using Stack = Array<NodeAndParent>;`.
- **CN**: 引入类型别名或 using 声明：`using Stack = Array<NodeAndParent>;`。

### Line 486
````cpp
<blank>
````
- **EN**: Blank line separating logical sections.
- **CN**: 空行，用于分隔逻辑段落。

### Line 487
````cpp
    typename Stack::AllocatorType StackAllocator(
````
- **EN**: Carries part of the local implementation logic: `typename Stack::AllocatorType StackAllocator(`.
- **CN**: 承载局部实现逻辑：`typename Stack::AllocatorType StackAllocator(`。

### Line 488
````cpp
        profilingFlags()->stack_allocator_max);
````
- **EN**: Invokes a function-like statement: `profilingFlags()->stack_allocator_max);`.
- **CN**: 调用一个类似函数的语句：`profilingFlags()->stack_allocator_max);`。

### Line 489
````cpp
    Stack DFSStack(StackAllocator);
````
- **EN**: Invokes a function-like statement: `Stack DFSStack(StackAllocator);`.
- **CN**: 调用一个类似函数的语句：`Stack DFSStack(StackAllocator);`。

### Line 490
````cpp
<blank>
````
- **EN**: Blank line separating logical sections.
- **CN**: 空行，用于分隔逻辑段落。

### Line 491
````cpp
    for (const auto Root : getRoots()) {
````
- **EN**: Starts a `for` loop: `for (const auto Root : getRoots()) {`.
- **CN**: 开始一个 `for` 循环：`for (const auto Root : getRoots()) {`。

### Line 492
````cpp
      // Add a node in O for this root.
````
- **EN**: Comment documenting `Add a node in O for this root.`.
- **CN**: 注释说明了 `Add a node in O for this root.`。

### Line 493
````cpp
      auto NewRoot = O.Nodes.AppendEmplace(
````
- **EN**: Carries part of the local implementation logic: `auto NewRoot = O.Nodes.AppendEmplace(`.
- **CN**: 承载局部实现逻辑：`auto NewRoot = O.Nodes.AppendEmplace(`。

### Line 494
````cpp
          nullptr, NodeIdPairArray(*O.NodeIdPairAllocator), Root->CallCount,
````
- **EN**: Carries part of the local implementation logic: `nullptr, NodeIdPairArray(*O.NodeIdPairAllocator), Root->CallCount,`.
- **CN**: 承载局部实现逻辑：`nullptr, NodeIdPairArray(*O.NodeIdPairAllocator), Root->CallCount,`。

### Line 495
````cpp
          Root->CumulativeLocalTime, Root->FId);
````
- **EN**: Executes or declares `Root->CumulativeLocalTime, Root->FId);` within the current scope.
- **CN**: 在当前作用域中执行或声明 `Root->CumulativeLocalTime, Root->FId);`。

### Line 496
````cpp
<blank>
````
- **EN**: Blank line separating logical sections.
- **CN**: 空行，用于分隔逻辑段落。

### Line 497
````cpp
      // Because we cannot allocate more memory we should bail out right away.
````
- **EN**: Comment documenting `Because we cannot allocate more memory we should bail out right away.`.
- **CN**: 注释说明了 `Because we cannot allocate more memory we should bail out right away.`。

### Line 498
````cpp
      if (UNLIKELY(NewRoot == nullptr))
````
- **EN**: Evaluates the conditional branch `if (UNLIKELY(NewRoot == nullptr))`.
- **CN**: 计算条件分支 `if (UNLIKELY(NewRoot == nullptr))`。

### Line 499
````cpp
        return;
````
- **EN**: Returns from the current function with `;`.
- **CN**: 使用 `;` 从当前函数返回。

### Line 500
````cpp
<blank>
````
- **EN**: Blank line separating logical sections.
- **CN**: 空行，用于分隔逻辑段落。

### Line 501
````cpp
      if (UNLIKELY(O.Roots.Append(NewRoot) == nullptr))
````
- **EN**: Evaluates the conditional branch `if (UNLIKELY(O.Roots.Append(NewRoot) == nullptr))`.
- **CN**: 计算条件分支 `if (UNLIKELY(O.Roots.Append(NewRoot) == nullptr))`。

### Line 502
````cpp
        return;
````
- **EN**: Returns from the current function with `;`.
- **CN**: 使用 `;` 从当前函数返回。

### Line 503
````cpp
<blank>
````
- **EN**: Blank line separating logical sections.
- **CN**: 空行，用于分隔逻辑段落。

### Line 504
````cpp
      // TODO: Figure out what to do if we fail to allocate any more stack
````
- **EN**: Comment recording follow-up work: `TODO: Figure out what to do if we fail to allocate any more stack`.
- **CN**: 注释记录后续待办事项：`TODO: Figure out what to do if we fail to allocate any more stack`。

### Line 505
````cpp
      // space. Maybe warn or report once?
````
- **EN**: Comment documenting `space. Maybe warn or report once?`.
- **CN**: 注释说明了 `space. Maybe warn or report once?`。

### Line 506
````cpp
      if (DFSStack.AppendEmplace(Root, NewRoot) == nullptr)
````
- **EN**: Evaluates the conditional branch `if (DFSStack.AppendEmplace(Root, NewRoot) == nullptr)`.
- **CN**: 计算条件分支 `if (DFSStack.AppendEmplace(Root, NewRoot) == nullptr)`。

### Line 507
````cpp
        return;
````
- **EN**: Returns from the current function with `;`.
- **CN**: 使用 `;` 从当前函数返回。

### Line 508
````cpp
      while (!DFSStack.empty()) {
````
- **EN**: Starts a `while` loop: `while (!DFSStack.empty()) {`.
- **CN**: 开始一个 `while` 循环：`while (!DFSStack.empty()) {`。

### Line 509
````cpp
        NodeAndParent NP = DFSStack.back();
````
- **EN**: Invokes a function-like statement: `NodeAndParent NP = DFSStack.back();`.
- **CN**: 调用一个类似函数的语句：`NodeAndParent NP = DFSStack.back();`。

### Line 510
````cpp
        DCHECK_NE(NP.Node, nullptr);
````
- **EN**: Invokes a function-like statement: `DCHECK_NE(NP.Node, nullptr);`.
- **CN**: 调用一个类似函数的语句：`DCHECK_NE(NP.Node, nullptr);`。

### Line 511
````cpp
        DCHECK_NE(NP.NewNode, nullptr);
````
- **EN**: Invokes a function-like statement: `DCHECK_NE(NP.NewNode, nullptr);`.
- **CN**: 调用一个类似函数的语句：`DCHECK_NE(NP.NewNode, nullptr);`。

### Line 512
````cpp
        DFSStack.trim(1);
````
- **EN**: Invokes a function-like statement: `DFSStack.trim(1);`.
- **CN**: 调用一个类似函数的语句：`DFSStack.trim(1);`。

### Line 513
````cpp
        for (const auto Callee : NP.Node->Callees) {
````
- **EN**: Starts a `for` loop: `for (const auto Callee : NP.Node->Callees) {`.
- **CN**: 开始一个 `for` 循环：`for (const auto Callee : NP.Node->Callees) {`。

### Line 514
````cpp
          auto NewNode = O.Nodes.AppendEmplace(
````
- **EN**: Carries part of the local implementation logic: `auto NewNode = O.Nodes.AppendEmplace(`.
- **CN**: 承载局部实现逻辑：`auto NewNode = O.Nodes.AppendEmplace(`。

### Line 515
````cpp
              NP.NewNode, NodeIdPairArray(*O.NodeIdPairAllocator),
````
- **EN**: Carries part of the local implementation logic: `NP.NewNode, NodeIdPairArray(*O.NodeIdPairAllocator),`.
- **CN**: 承载局部实现逻辑：`NP.NewNode, NodeIdPairArray(*O.NodeIdPairAllocator),`。

### Line 516
````cpp
              Callee.NodePtr->CallCount, Callee.NodePtr->CumulativeLocalTime,
````
- **EN**: Carries part of the local implementation logic: `Callee.NodePtr->CallCount, Callee.NodePtr->CumulativeLocalTime,`.
- **CN**: 承载局部实现逻辑：`Callee.NodePtr->CallCount, Callee.NodePtr->CumulativeLocalTime,`。

### Line 517
````cpp
              Callee.FId);
````
- **EN**: Executes or declares `Callee.FId);` within the current scope.
- **CN**: 在当前作用域中执行或声明 `Callee.FId);`。

### Line 518
````cpp
          if (UNLIKELY(NewNode == nullptr))
````
- **EN**: Evaluates the conditional branch `if (UNLIKELY(NewNode == nullptr))`.
- **CN**: 计算条件分支 `if (UNLIKELY(NewNode == nullptr))`。

### Line 519
````cpp
            return;
````
- **EN**: Returns from the current function with `;`.
- **CN**: 使用 `;` 从当前函数返回。

### Line 520
````cpp
          if (UNLIKELY(NP.NewNode->Callees.AppendEmplace(NewNode, Callee.FId) ==
````
- **EN**: Evaluates the conditional branch `if (UNLIKELY(NP.NewNode->Callees.AppendEmplace(NewNode, Callee.FId) ==`.
- **CN**: 计算条件分支 `if (UNLIKELY(NP.NewNode->Callees.AppendEmplace(NewNode, Callee.FId) ==`。

### Line 521
````cpp
                       nullptr))
````
- **EN**: Carries part of the local implementation logic: `nullptr))`.
- **CN**: 承载局部实现逻辑：`nullptr))`。

### Line 522
````cpp
            return;
````
- **EN**: Returns from the current function with `;`.
- **CN**: 使用 `;` 从当前函数返回。

### Line 523
````cpp
          if (UNLIKELY(DFSStack.AppendEmplace(Callee.NodePtr, NewNode) ==
````
- **EN**: Evaluates the conditional branch `if (UNLIKELY(DFSStack.AppendEmplace(Callee.NodePtr, NewNode) ==`.
- **CN**: 计算条件分支 `if (UNLIKELY(DFSStack.AppendEmplace(Callee.NodePtr, NewNode) ==`。

### Line 524
````cpp
                       nullptr))
````
- **EN**: Carries part of the local implementation logic: `nullptr))`.
- **CN**: 承载局部实现逻辑：`nullptr))`。

### Line 525
````cpp
            return;
````
- **EN**: Returns from the current function with `;`.
- **CN**: 使用 `;` 从当前函数返回。

### Line 526
````cpp
        }
````
- **EN**: Closes the current scope block.
- **CN**: 结束当前作用域块。

### Line 527
````cpp
      }
````
- **EN**: Closes the current scope block.
- **CN**: 结束当前作用域块。

### Line 528
````cpp
    }
````
- **EN**: Closes the current scope block.
- **CN**: 结束当前作用域块。

### Line 529
````cpp
  }
````
- **EN**: Closes the current scope block.
- **CN**: 结束当前作用域块。

### Line 530
````cpp
<blank>
````
- **EN**: Blank line separating logical sections.
- **CN**: 空行，用于分隔逻辑段落。

### Line 531
````cpp
  // The mergeInto operation will update the provided FunctionCallTrie by
````
- **EN**: Comment documenting `The mergeInto operation will update the provided FunctionCallTrie by`.
- **CN**: 注释说明了 `The mergeInto operation will update the provided FunctionCallTrie by`。

### Line 532
````cpp
  // traversing the current trie's roots and updating (i.e. merging) the data in
````
- **EN**: Comment documenting `traversing the current trie's roots and updating (i.e. merging) the data in`.
- **CN**: 注释说明了 `traversing the current trie's roots and updating (i.e. merging) the data in`。

### Line 533
````cpp
  // the nodes with the data in the target's nodes. If the node doesn't exist in
````
- **EN**: Comment documenting `the nodes with the data in the target's nodes. If the node doesn't exist in`.
- **CN**: 注释说明了 `the nodes with the data in the target's nodes. If the node doesn't exist in`。

### Line 534
````cpp
  // the provided trie, we add a new one in the right position, and inherit the
````
- **EN**: Comment documenting `the provided trie, we add a new one in the right position, and inherit the`.
- **CN**: 注释说明了 `the provided trie, we add a new one in the right position, and inherit the`。

### Line 535
````cpp
  // data from the original (current) trie, along with all its callees.
````
- **EN**: Comment documenting `data from the original (current) trie, along with all its callees.`.
- **CN**: 注释说明了 `data from the original (current) trie, along with all its callees.`。

### Line 536
````cpp
  //
````
- **EN**: Comment formatting line in a banner or block comment.
- **CN**: 注释格式行，用于组织文件头或注释块。

### Line 537
````cpp
  // This function is *not* thread-safe, and may require external
````
- **EN**: Comment documenting `This function is *not* thread-safe, and may require external`.
- **CN**: 注释说明了 `This function is *not* thread-safe, and may require external`。

### Line 538
````cpp
  // synchronisation of both "this" and |O|.
````
- **EN**: Comment documenting `synchronisation of both "this" and |O|.`.
- **CN**: 注释说明了 `synchronisation of both "this" and |O|.`。

### Line 539
````cpp
  void mergeInto(FunctionCallTrie &O) const XRAY_NEVER_INSTRUMENT {
````
- **EN**: Carries part of the local implementation logic: `void mergeInto(FunctionCallTrie &O) const XRAY_NEVER_INSTRUMENT {`.
- **CN**: 承载局部实现逻辑：`void mergeInto(FunctionCallTrie &O) const XRAY_NEVER_INSTRUMENT {`。

### Line 540
````cpp
    struct NodeAndTarget {
````
- **EN**: Declares the struct `NodeAndTarget`.
- **CN**: 声明 struct `NodeAndTarget`。

### Line 541
````cpp
      FunctionCallTrie::Node *OrigNode;
````
- **EN**: Executes or declares `FunctionCallTrie::Node *OrigNode;` within the current scope.
- **CN**: 在当前作用域中执行或声明 `FunctionCallTrie::Node *OrigNode;`。

### Line 542
````cpp
      FunctionCallTrie::Node *TargetNode;
````
- **EN**: Executes or declares `FunctionCallTrie::Node *TargetNode;` within the current scope.
- **CN**: 在当前作用域中执行或声明 `FunctionCallTrie::Node *TargetNode;`。

### Line 543
````cpp
    };
````
- **EN**: Closes a type definition or aggregate block.
- **CN**: 结束一个类型定义或聚合块。

### Line 544
````cpp
    using Stack = Array<NodeAndTarget>;
````
- **EN**: Introduces a type alias or using-declaration: `using Stack = Array<NodeAndTarget>;`.
- **CN**: 引入类型别名或 using 声明：`using Stack = Array<NodeAndTarget>;`。

### Line 545
````cpp
    typename Stack::AllocatorType StackAllocator(
````
- **EN**: Carries part of the local implementation logic: `typename Stack::AllocatorType StackAllocator(`.
- **CN**: 承载局部实现逻辑：`typename Stack::AllocatorType StackAllocator(`。

### Line 546
````cpp
        profilingFlags()->stack_allocator_max);
````
- **EN**: Invokes a function-like statement: `profilingFlags()->stack_allocator_max);`.
- **CN**: 调用一个类似函数的语句：`profilingFlags()->stack_allocator_max);`。

### Line 547
````cpp
    Stack DFSStack(StackAllocator);
````
- **EN**: Invokes a function-like statement: `Stack DFSStack(StackAllocator);`.
- **CN**: 调用一个类似函数的语句：`Stack DFSStack(StackAllocator);`。

### Line 548
````cpp
<blank>
````
- **EN**: Blank line separating logical sections.
- **CN**: 空行，用于分隔逻辑段落。

### Line 549
````cpp
    for (const auto Root : getRoots()) {
````
- **EN**: Starts a `for` loop: `for (const auto Root : getRoots()) {`.
- **CN**: 开始一个 `for` 循环：`for (const auto Root : getRoots()) {`。

### Line 550
````cpp
      Node *TargetRoot = nullptr;
````
- **EN**: Assigns or initializes state with `Node *TargetRoot = nullptr;`.
- **CN**: 使用 `Node *TargetRoot = nullptr;` 进行赋值或初始化。

### Line 551
````cpp
      auto R = O.Roots.find_element(
````
- **EN**: Carries part of the local implementation logic: `auto R = O.Roots.find_element(`.
- **CN**: 承载局部实现逻辑：`auto R = O.Roots.find_element(`。

### Line 552
````cpp
          [&](const Node *Node) { return Node->FId == Root->FId; });
````
- **EN**: Invokes a function-like statement: `[&](const Node *Node) { return Node->FId == Root->FId; });`.
- **CN**: 调用一个类似函数的语句：`[&](const Node *Node) { return Node->FId == Root->FId; });`。

### Line 553
````cpp
      if (R == nullptr) {
````
- **EN**: Evaluates the conditional branch `if (R == nullptr) {`.
- **CN**: 计算条件分支 `if (R == nullptr) {`。

### Line 554
````cpp
        TargetRoot = O.Nodes.AppendEmplace(
````
- **EN**: Carries part of the local implementation logic: `TargetRoot = O.Nodes.AppendEmplace(`.
- **CN**: 承载局部实现逻辑：`TargetRoot = O.Nodes.AppendEmplace(`。

### Line 555
````cpp
            nullptr, NodeIdPairArray(*O.NodeIdPairAllocator), 0u, 0u,
````
- **EN**: Carries part of the local implementation logic: `nullptr, NodeIdPairArray(*O.NodeIdPairAllocator), 0u, 0u,`.
- **CN**: 承载局部实现逻辑：`nullptr, NodeIdPairArray(*O.NodeIdPairAllocator), 0u, 0u,`。

### Line 556
````cpp
            Root->FId);
````
- **EN**: Executes or declares `Root->FId);` within the current scope.
- **CN**: 在当前作用域中执行或声明 `Root->FId);`。

### Line 557
````cpp
        if (UNLIKELY(TargetRoot == nullptr))
````
- **EN**: Evaluates the conditional branch `if (UNLIKELY(TargetRoot == nullptr))`.
- **CN**: 计算条件分支 `if (UNLIKELY(TargetRoot == nullptr))`。

### Line 558
````cpp
          return;
````
- **EN**: Returns from the current function with `;`.
- **CN**: 使用 `;` 从当前函数返回。

### Line 559
````cpp
<blank>
````
- **EN**: Blank line separating logical sections.
- **CN**: 空行，用于分隔逻辑段落。

### Line 560
````cpp
        O.Roots.Append(TargetRoot);
````
- **EN**: Invokes a function-like statement: `O.Roots.Append(TargetRoot);`.
- **CN**: 调用一个类似函数的语句：`O.Roots.Append(TargetRoot);`。

### Line 561
````cpp
      } else {
````
- **EN**: Carries part of the local implementation logic: `} else {`.
- **CN**: 承载局部实现逻辑：`} else {`。

### Line 562
````cpp
        TargetRoot = *R;
````
- **EN**: Assigns or initializes state with `TargetRoot = *R;`.
- **CN**: 使用 `TargetRoot = *R;` 进行赋值或初始化。

### Line 563
````cpp
      }
````
- **EN**: Closes the current scope block.
- **CN**: 结束当前作用域块。

### Line 564
````cpp
<blank>
````
- **EN**: Blank line separating logical sections.
- **CN**: 空行，用于分隔逻辑段落。

### Line 565
````cpp
      DFSStack.AppendEmplace(Root, TargetRoot);
````
- **EN**: Invokes a function-like statement: `DFSStack.AppendEmplace(Root, TargetRoot);`.
- **CN**: 调用一个类似函数的语句：`DFSStack.AppendEmplace(Root, TargetRoot);`。

### Line 566
````cpp
      while (!DFSStack.empty()) {
````
- **EN**: Starts a `while` loop: `while (!DFSStack.empty()) {`.
- **CN**: 开始一个 `while` 循环：`while (!DFSStack.empty()) {`。

### Line 567
````cpp
        NodeAndTarget NT = DFSStack.back();
````
- **EN**: Invokes a function-like statement: `NodeAndTarget NT = DFSStack.back();`.
- **CN**: 调用一个类似函数的语句：`NodeAndTarget NT = DFSStack.back();`。

### Line 568
````cpp
        DCHECK_NE(NT.OrigNode, nullptr);
````
- **EN**: Invokes a function-like statement: `DCHECK_NE(NT.OrigNode, nullptr);`.
- **CN**: 调用一个类似函数的语句：`DCHECK_NE(NT.OrigNode, nullptr);`。

### Line 569
````cpp
        DCHECK_NE(NT.TargetNode, nullptr);
````
- **EN**: Invokes a function-like statement: `DCHECK_NE(NT.TargetNode, nullptr);`.
- **CN**: 调用一个类似函数的语句：`DCHECK_NE(NT.TargetNode, nullptr);`。

### Line 570
````cpp
        DFSStack.trim(1);
````
- **EN**: Invokes a function-like statement: `DFSStack.trim(1);`.
- **CN**: 调用一个类似函数的语句：`DFSStack.trim(1);`。

### Line 571
````cpp
        // TODO: Update the histogram as well when we have it ready.
````
- **EN**: Comment recording follow-up work: `TODO: Update the histogram as well when we have it ready.`.
- **CN**: 注释记录后续待办事项：`TODO: Update the histogram as well when we have it ready.`。

### Line 572
````cpp
        NT.TargetNode->CallCount += NT.OrigNode->CallCount;
````
- **EN**: Assigns or initializes state with `NT.TargetNode->CallCount += NT.OrigNode->CallCount;`.
- **CN**: 使用 `NT.TargetNode->CallCount += NT.OrigNode->CallCount;` 进行赋值或初始化。

### Line 573
````cpp
        NT.TargetNode->CumulativeLocalTime += NT.OrigNode->CumulativeLocalTime;
````
- **EN**: Assigns or initializes state with `NT.TargetNode->CumulativeLocalTime += NT.OrigNode->CumulativeLocalTime;`.
- **CN**: 使用 `NT.TargetNode->CumulativeLocalTime += NT.OrigNode->CumulativeLocalTime;` 进行赋值或初始化。

### Line 574
````cpp
        for (const auto Callee : NT.OrigNode->Callees) {
````
- **EN**: Starts a `for` loop: `for (const auto Callee : NT.OrigNode->Callees) {`.
- **CN**: 开始一个 `for` 循环：`for (const auto Callee : NT.OrigNode->Callees) {`。

### Line 575
````cpp
          auto TargetCallee = NT.TargetNode->Callees.find_element(
````
- **EN**: Carries part of the local implementation logic: `auto TargetCallee = NT.TargetNode->Callees.find_element(`.
- **CN**: 承载局部实现逻辑：`auto TargetCallee = NT.TargetNode->Callees.find_element(`。

### Line 576
````cpp
              [&](const FunctionCallTrie::NodeIdPair &C) {
````
- **EN**: Begins a function or method definition: `[&](const FunctionCallTrie::NodeIdPair &C) {`.
- **CN**: 开始一个函数或方法定义：`[&](const FunctionCallTrie::NodeIdPair &C) {`。

### Line 577
````cpp
                return C.FId == Callee.FId;
````
- **EN**: Returns from the current function with `C.FId == Callee.FId;`.
- **CN**: 使用 `C.FId == Callee.FId;` 从当前函数返回。

### Line 578
````cpp
              });
````
- **EN**: Executes or declares `});` within the current scope.
- **CN**: 在当前作用域中执行或声明 `});`。

### Line 579
````cpp
          if (TargetCallee == nullptr) {
````
- **EN**: Evaluates the conditional branch `if (TargetCallee == nullptr) {`.
- **CN**: 计算条件分支 `if (TargetCallee == nullptr) {`。

### Line 580
````cpp
            auto NewTargetNode = O.Nodes.AppendEmplace(
````
- **EN**: Carries part of the local implementation logic: `auto NewTargetNode = O.Nodes.AppendEmplace(`.
- **CN**: 承载局部实现逻辑：`auto NewTargetNode = O.Nodes.AppendEmplace(`。

### Line 581
````cpp
                NT.TargetNode, NodeIdPairArray(*O.NodeIdPairAllocator), 0u, 0u,
````
- **EN**: Carries part of the local implementation logic: `NT.TargetNode, NodeIdPairArray(*O.NodeIdPairAllocator), 0u, 0u,`.
- **CN**: 承载局部实现逻辑：`NT.TargetNode, NodeIdPairArray(*O.NodeIdPairAllocator), 0u, 0u,`。

### Line 582
````cpp
                Callee.FId);
````
- **EN**: Executes or declares `Callee.FId);` within the current scope.
- **CN**: 在当前作用域中执行或声明 `Callee.FId);`。

### Line 583
````cpp
<blank>
````
- **EN**: Blank line separating logical sections.
- **CN**: 空行，用于分隔逻辑段落。

### Line 584
````cpp
            if (UNLIKELY(NewTargetNode == nullptr))
````
- **EN**: Evaluates the conditional branch `if (UNLIKELY(NewTargetNode == nullptr))`.
- **CN**: 计算条件分支 `if (UNLIKELY(NewTargetNode == nullptr))`。

### Line 585
````cpp
              return;
````
- **EN**: Returns from the current function with `;`.
- **CN**: 使用 `;` 从当前函数返回。

### Line 586
````cpp
<blank>
````
- **EN**: Blank line separating logical sections.
- **CN**: 空行，用于分隔逻辑段落。

### Line 587
````cpp
            TargetCallee =
````
- **EN**: Carries part of the local implementation logic: `TargetCallee =`.
- **CN**: 承载局部实现逻辑：`TargetCallee =`。

### Line 588
````cpp
                NT.TargetNode->Callees.AppendEmplace(NewTargetNode, Callee.FId);
````
- **EN**: Invokes a function-like statement: `NT.TargetNode->Callees.AppendEmplace(NewTargetNode, Callee.FId);`.
- **CN**: 调用一个类似函数的语句：`NT.TargetNode->Callees.AppendEmplace(NewTargetNode, Callee.FId);`。

### Line 589
````cpp
          }
````
- **EN**: Closes the current scope block.
- **CN**: 结束当前作用域块。

### Line 590
````cpp
          DFSStack.AppendEmplace(Callee.NodePtr, TargetCallee->NodePtr);
````
- **EN**: Invokes a function-like statement: `DFSStack.AppendEmplace(Callee.NodePtr, TargetCallee->NodePtr);`.
- **CN**: 调用一个类似函数的语句：`DFSStack.AppendEmplace(Callee.NodePtr, TargetCallee->NodePtr);`。

### Line 591
````cpp
        }
````
- **EN**: Closes the current scope block.
- **CN**: 结束当前作用域块。

### Line 592
````cpp
      }
````
- **EN**: Closes the current scope block.
- **CN**: 结束当前作用域块。

### Line 593
````cpp
    }
````
- **EN**: Closes the current scope block.
- **CN**: 结束当前作用域块。

### Line 594
````cpp
  }
````
- **EN**: Closes the current scope block.
- **CN**: 结束当前作用域块。

### Line 595
````cpp
};
````
- **EN**: Closes a type definition or aggregate block.
- **CN**: 结束一个类型定义或聚合块。

### Line 596
````cpp
<blank>
````
- **EN**: Blank line separating logical sections.
- **CN**: 空行，用于分隔逻辑段落。

### Line 597
````cpp
} // namespace __xray
````
- **EN**: Closes namespace `__xray`.
- **CN**: 关闭命名空间 `__xray`。

### Line 598
````cpp
<blank>
````
- **EN**: Blank line separating logical sections.
- **CN**: 空行，用于分隔逻辑段落。

### Line 599
````cpp
#endif // XRAY_FUNCTION_CALL_TRIE_H
````
- **EN**: Ends the current preprocessor condition.
- **CN**: 结束当前的预处理条件。

## Key Concepts / 关键概念
- Preprocessor control / 预处理控制
- Namespaces / 命名空间
- Types and interfaces / 类型与接口
- Function logic / 函数逻辑
- Memory management / 内存管理
- Threading and synchronization / 线程与同步

## Dependencies / 依赖关系
- **Local headers / 本地头文件**: `xray_buffer_queue.h`, `xray_defs.h`, `xray_profiling_flags.h`, `xray_segmented_array.h`
- **System headers / 系统头文件**: `limits`, `memory`, `utility`
- **Compile-time conditions / 编译期条件**:
  - `#ifndef XRAY_FUNCTION_CALL_TRIE_H`
