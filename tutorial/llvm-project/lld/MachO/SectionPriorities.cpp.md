# SectionPriorities.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `lld/MachO/SectionPriorities.cpp`
- **Repository / 仓库**: `/root/xw/llvm-project/`
- **Purpose (EN) / 用途（英文）**: This is based on the ELF port, see ELF/CallGraphSort.cpp for the details about the algorithm.
- **Purpose (CN) / 用途（中文）**: 实现 Mach-O 链接器的符号图、地址分配以及输出文件构建逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-13 / 第 1-13 行

```cpp
   1: //===- SectionPriorities.cpp ----------------------------------------------===//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: ///
   9: /// This is based on the ELF port, see ELF/CallGraphSort.cpp for the details
  10: /// about the algorithm.
  11: ///
  12: //===----------------------------------------------------------------------===//
  13: 
```

- **L1**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L2**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L3**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L4**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L5**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L6**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L7**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L8**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L9**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L10**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L11**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L12**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L13**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。

### Lines 14-31 / 第 14-31 行

```cpp
  14: #include "SectionPriorities.h"
  15: #include "BPSectionOrderer.h"
  16: #include "Config.h"
  17: #include "InputFiles.h"
  18: #include "Symbols.h"
  19: #include "Target.h"
  20: 
  21: #include "lld/Common/Args.h"
  22: #include "lld/Common/CommonLinkerContext.h"
  23: #include "lld/Common/ErrorHandler.h"
  24: #include "lld/Common/Utils.h"
  25: #include "llvm/ADT/DenseMap.h"
  26: #include "llvm/ADT/MapVector.h"
  27: #include "llvm/Support/Path.h"
  28: #include "llvm/Support/TimeProfiler.h"
  29: #include "llvm/Support/raw_ostream.h"
  30: #include "llvm/Support/xxhash.h"
  31: 
```

- **L14**: Includes \`SectionPriorities.h\` so this file can use declarations from that header. / 引入 \`SectionPriorities.h\`，使当前文件能够使用该头文件中的声明。
- **L15**: Includes \`BPSectionOrderer.h\` so this file can use declarations from that header. / 引入 \`BPSectionOrderer.h\`，使当前文件能够使用该头文件中的声明。
- **L16**: Includes \`Config.h\` so this file can use declarations from that header. / 引入 \`Config.h\`，使当前文件能够使用该头文件中的声明。
- **L17**: Includes \`InputFiles.h\` so this file can use declarations from that header. / 引入 \`InputFiles.h\`，使当前文件能够使用该头文件中的声明。
- **L18**: Includes \`Symbols.h\` so this file can use declarations from that header. / 引入 \`Symbols.h\`，使当前文件能够使用该头文件中的声明。
- **L19**: Includes \`Target.h\` so this file can use declarations from that header. / 引入 \`Target.h\`，使当前文件能够使用该头文件中的声明。
- **L20**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L21**: Includes \`lld/Common/Args.h\` so this file can use declarations from that header. / 引入 \`lld/Common/Args.h\`，使当前文件能够使用该头文件中的声明。
- **L22**: Includes \`lld/Common/CommonLinkerContext.h\` so this file can use declarations from that header. / 引入 \`lld/Common/CommonLinkerContext.h\`，使当前文件能够使用该头文件中的声明。
- **L23**: Includes \`lld/Common/ErrorHandler.h\` so this file can use declarations from that header. / 引入 \`lld/Common/ErrorHandler.h\`，使当前文件能够使用该头文件中的声明。
- **L24**: Includes \`lld/Common/Utils.h\` so this file can use declarations from that header. / 引入 \`lld/Common/Utils.h\`，使当前文件能够使用该头文件中的声明。
- **L25**: Includes \`llvm/ADT/DenseMap.h\` so this file can use declarations from that header. / 引入 \`llvm/ADT/DenseMap.h\`，使当前文件能够使用该头文件中的声明。
- **L26**: Includes \`llvm/ADT/MapVector.h\` so this file can use declarations from that header. / 引入 \`llvm/ADT/MapVector.h\`，使当前文件能够使用该头文件中的声明。
- **L27**: Includes \`llvm/Support/Path.h\` so this file can use declarations from that header. / 引入 \`llvm/Support/Path.h\`，使当前文件能够使用该头文件中的声明。
- **L28**: Includes \`llvm/Support/TimeProfiler.h\` so this file can use declarations from that header. / 引入 \`llvm/Support/TimeProfiler.h\`，使当前文件能够使用该头文件中的声明。
- **L29**: Includes \`llvm/Support/raw_ostream.h\` so this file can use declarations from that header. / 引入 \`llvm/Support/raw_ostream.h\`，使当前文件能够使用该头文件中的声明。
- **L30**: Includes \`llvm/Support/xxhash.h\` so this file can use declarations from that header. / 引入 \`llvm/Support/xxhash.h\`，使当前文件能够使用该头文件中的声明。
- **L31**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。

### Lines 32-41 / 第 32-41 行

```cpp
  32: #include <numeric>
  33: 
  34: using namespace llvm;
  35: using namespace llvm::MachO;
  36: using namespace llvm::sys;
  37: using namespace lld;
  38: using namespace lld::macho;
  39: 
  40: PriorityBuilder macho::priorityBuilder;
  41: 
```

- **L32**: Includes \`numeric\` so this file can use declarations from that header. / 引入 \`numeric\`，使当前文件能够使用该头文件中的声明。
- **L33**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L34**: Imports namespace \`llvm\` into the current scope for shorter symbol references. / 将命名空间 \`llvm\` 导入当前作用域，以便更简洁地引用符号。
- **L35**: Imports namespace \`llvm::MachO\` into the current scope for shorter symbol references. / 将命名空间 \`llvm::MachO\` 导入当前作用域，以便更简洁地引用符号。
- **L36**: Imports namespace \`llvm::sys\` into the current scope for shorter symbol references. / 将命名空间 \`llvm::sys\` 导入当前作用域，以便更简洁地引用符号。
- **L37**: Imports namespace \`lld\` into the current scope for shorter symbol references. / 将命名空间 \`lld\` 导入当前作用域，以便更简洁地引用符号。
- **L38**: Imports namespace \`lld::macho\` into the current scope for shorter symbol references. / 将命名空间 \`lld::macho\` 导入当前作用域，以便更简洁地引用符号。
- **L39**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L40**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L41**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。

### Lines 42-50 / 第 42-50 行

```cpp
  42: namespace {
  43: struct Edge {
  44:   int from;
  45:   uint64_t weight;
  46: };
  47: 
  48: struct Cluster {
  49:   Cluster(int sec, size_t s) : next(sec), prev(sec), size(s) {}
  50: 
```

- **L42**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L43**: Begins the declaration of struct \`Edge\`. / 开始声明 struct \`Edge\`。
- **L44**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L45**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L46**: Closes the current declaration or scoped construct. / 结束当前声明或带作用域的结构。
- **L47**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L48**: Begins the declaration of struct \`Cluster\`. / 开始声明 struct \`Cluster\`。
- **L49**: Defines function or method \`Cluster\`. / 定义函数或方法 \`Cluster\`。
- **L50**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。

### Lines 51-64 / 第 51-64 行

```cpp
  51:   double getDensity() const {
  52:     if (size == 0)
  53:       return 0;
  54:     return double(weight) / double(size);
  55:   }
  56: 
  57:   int next;
  58:   int prev;
  59:   uint64_t size;
  60:   uint64_t weight = 0;
  61:   uint64_t initialWeight = 0;
  62:   Edge bestPred = {-1, 0};
  63: };
  64: 
```

- **L51**: Defines function or method \`getDensity\`. / 定义函数或方法 \`getDensity\`。
- **L52**: Starts a conditional branch that runs only when the predicate is true. / 开始一个条件分支，仅在判定条件为真时执行。
- **L53**: Returns a value or exits the current function at this point. / 在此返回一个值，或直接结束当前函数。
- **L54**: Returns a value or exits the current function at this point. / 在此返回一个值，或直接结束当前函数。
- **L55**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L56**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L57**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L58**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L59**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L60**: Assigns or initializes state used by the surrounding logic. / 赋值或初始化周边逻辑要使用的状态。
- **L61**: Assigns or initializes state used by the surrounding logic. / 赋值或初始化周边逻辑要使用的状态。
- **L62**: Closes the current declaration or scoped construct. / 结束当前声明或带作用域的结构。
- **L63**: Closes the current declaration or scoped construct. / 结束当前声明或带作用域的结构。
- **L64**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。

### Lines 65-79 / 第 65-79 行

```cpp
  65: class CallGraphSort {
  66: public:
  67:   CallGraphSort(const MapVector<SectionPair, uint64_t> &profile);
  68: 
  69:   DenseMap<const InputSection *, int> run();
  70: 
  71: private:
  72:   std::vector<Cluster> clusters;
  73:   std::vector<const InputSection *> sections;
  74: };
  75: // Maximum amount the combined cluster density can be worse than the original
  76: // cluster to consider merging.
  77: constexpr int MAX_DENSITY_DEGRADATION = 8;
  78: } // end anonymous namespace
  79: 
```

- **L65**: Begins the declaration of class \`CallGraphSort\`. / 开始声明 class \`CallGraphSort\`。
- **L66**: Switches following class members to \`public\` access control. / 将后续类成员的访问控制切换为 \`public\`。
- **L67**: Declares function or method \`CallGraphSort\`. / 声明函数或方法 \`CallGraphSort\`。
- **L68**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L69**: Declares function or method \`run\`. / 声明函数或方法 \`run\`。
- **L70**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L71**: Switches following class members to \`private\` access control. / 将后续类成员的访问控制切换为 \`private\`。
- **L72**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L73**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L74**: Closes the current declaration or scoped construct. / 结束当前声明或带作用域的结构。
- **L75**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L76**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L77**: Assigns or initializes state used by the surrounding logic. / 赋值或初始化周边逻辑要使用的状态。
- **L78**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L79**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。

### Lines 80-93 / 第 80-93 行

```cpp
  80: // Take the edge list in callGraphProfile, resolve symbol names to Symbols, and
  81: // generate a graph between InputSections with the provided weights.
  82: CallGraphSort::CallGraphSort(const MapVector<SectionPair, uint64_t> &profile) {
  83:   DenseMap<const InputSection *, int> secToCluster;
  84: 
  85:   auto getOrCreateCluster = [&](const InputSection *isec) -> int {
  86:     auto res = secToCluster.try_emplace(isec, clusters.size());
  87:     if (res.second) {
  88:       sections.push_back(isec);
  89:       clusters.emplace_back(clusters.size(), isec->getSize());
  90:     }
  91:     return res.first->second;
  92:   };
  93: 
```

- **L80**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L81**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L82**: Defines function or method \`CallGraphSort\`. / 定义函数或方法 \`CallGraphSort\`。
- **L83**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L84**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L85**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L86**: Declares function or method \`try_emplace\`. / 声明函数或方法 \`try_emplace\`。
- **L87**: Starts a conditional branch that runs only when the predicate is true. / 开始一个条件分支，仅在判定条件为真时执行。
- **L88**: Declares function or method \`push_back\`. / 声明函数或方法 \`push_back\`。
- **L89**: Declares function or method \`emplace_back\`. / 声明函数或方法 \`emplace_back\`。
- **L90**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L91**: Returns a value or exits the current function at this point. / 在此返回一个值，或直接结束当前函数。
- **L92**: Closes the current declaration or scoped construct. / 结束当前声明或带作用域的结构。
- **L93**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。

### Lines 94-107 / 第 94-107 行

```cpp
  94:   // Create the graph
  95:   for (const std::pair<SectionPair, uint64_t> &c : profile) {
  96:     const auto fromSec = c.first.first->canonical();
  97:     const auto toSec = c.first.second->canonical();
  98:     uint64_t weight = c.second;
  99:     // Ignore edges between input sections belonging to different output
 100:     // sections.  This is done because otherwise we would end up with clusters
 101:     // containing input sections that can't actually be placed adjacently in the
 102:     // output.  This messes with the cluster size and density calculations.  We
 103:     // would also end up moving input sections in other output sections without
 104:     // moving them closer to what calls them.
 105:     if (fromSec->parent != toSec->parent)
 106:       continue;
 107: 
```

- **L94**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L95**: Starts a counted or iterator-based loop. / 开始一个计数式或迭代器式循环。
- **L96**: Declares function or method \`canonical\`. / 声明函数或方法 \`canonical\`。
- **L97**: Declares function or method \`canonical\`. / 声明函数或方法 \`canonical\`。
- **L98**: Assigns or initializes state used by the surrounding logic. / 赋值或初始化周边逻辑要使用的状态。
- **L99**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L100**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L101**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L102**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L103**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L104**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L105**: Starts a conditional branch that runs only when the predicate is true. / 开始一个条件分支，仅在判定条件为真时执行。
- **L106**: Skips directly to the next loop iteration. / 直接跳到下一次循环迭代。
- **L107**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。

### Lines 108-125 / 第 108-125 行

```cpp
 108:     int from = getOrCreateCluster(fromSec);
 109:     int to = getOrCreateCluster(toSec);
 110: 
 111:     clusters[to].weight += weight;
 112: 
 113:     if (from == to)
 114:       continue;
 115: 
 116:     // Remember the best edge.
 117:     Cluster &toC = clusters[to];
 118:     if (toC.bestPred.from == -1 || toC.bestPred.weight < weight) {
 119:       toC.bestPred.from = from;
 120:       toC.bestPred.weight = weight;
 121:     }
 122:   }
 123:   for (Cluster &c : clusters)
 124:     c.initialWeight = c.weight;
 125: }
```

- **L108**: Declares function or method \`getOrCreateCluster\`. / 声明函数或方法 \`getOrCreateCluster\`。
- **L109**: Declares function or method \`getOrCreateCluster\`. / 声明函数或方法 \`getOrCreateCluster\`。
- **L110**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L111**: Assigns or initializes state used by the surrounding logic. / 赋值或初始化周边逻辑要使用的状态。
- **L112**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L113**: Starts a conditional branch that runs only when the predicate is true. / 开始一个条件分支，仅在判定条件为真时执行。
- **L114**: Skips directly to the next loop iteration. / 直接跳到下一次循环迭代。
- **L115**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L116**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L117**: Assigns or initializes state used by the surrounding logic. / 赋值或初始化周边逻辑要使用的状态。
- **L118**: Starts a conditional branch that runs only when the predicate is true. / 开始一个条件分支，仅在判定条件为真时执行。
- **L119**: Assigns or initializes state used by the surrounding logic. / 赋值或初始化周边逻辑要使用的状态。
- **L120**: Assigns or initializes state used by the surrounding logic. / 赋值或初始化周边逻辑要使用的状态。
- **L121**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L122**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L123**: Starts a counted or iterator-based loop. / 开始一个计数式或迭代器式循环。
- **L124**: Assigns or initializes state used by the surrounding logic. / 赋值或初始化周边逻辑要使用的状态。
- **L125**: Closes the current scope or body. / 关闭当前作用域或代码体。

### Lines 126-143 / 第 126-143 行

```cpp
 126: 
 127: // It's bad to merge clusters which would degrade the density too much.
 128: static bool isNewDensityBad(Cluster &a, Cluster &b) {
 129:   double newDensity = double(a.weight + b.weight) / double(a.size + b.size);
 130:   return newDensity < a.getDensity() / MAX_DENSITY_DEGRADATION;
 131: }
 132: 
 133: // Find the leader of V's belonged cluster (represented as an equivalence
 134: // class). We apply union-find path-halving technique (simple to implement) in
 135: // the meantime as it decreases depths and the time complexity.
 136: static int getLeader(std::vector<int> &leaders, int v) {
 137:   while (leaders[v] != v) {
 138:     leaders[v] = leaders[leaders[v]];
 139:     v = leaders[v];
 140:   }
 141:   return v;
 142: }
 143: 
```

- **L126**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L127**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L128**: Defines function or method \`isNewDensityBad\`. / 定义函数或方法 \`isNewDensityBad\`。
- **L129**: Declares function or method \`double\`. / 声明函数或方法 \`double\`。
- **L130**: Returns a value or exits the current function at this point. / 在此返回一个值，或直接结束当前函数。
- **L131**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L132**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L133**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L134**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L135**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L136**: Defines function or method \`getLeader\`. / 定义函数或方法 \`getLeader\`。
- **L137**: Starts a loop that repeats while the condition remains true. / 开始一个在条件为真时持续执行的循环。
- **L138**: Assigns or initializes state used by the surrounding logic. / 赋值或初始化周边逻辑要使用的状态。
- **L139**: Assigns or initializes state used by the surrounding logic. / 赋值或初始化周边逻辑要使用的状态。
- **L140**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L141**: Returns a value or exits the current function at this point. / 在此返回一个值，或直接结束当前函数。
- **L142**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L143**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。

### Lines 144-156 / 第 144-156 行

```cpp
 144: static void mergeClusters(std::vector<Cluster> &cs, Cluster &into, int intoIdx,
 145:                           Cluster &from, int fromIdx) {
 146:   int tail1 = into.prev, tail2 = from.prev;
 147:   into.prev = tail2;
 148:   cs[tail2].next = intoIdx;
 149:   from.prev = tail1;
 150:   cs[tail1].next = fromIdx;
 151:   into.size += from.size;
 152:   into.weight += from.weight;
 153:   from.size = 0;
 154:   from.weight = 0;
 155: }
 156: 
```

- **L144**: Continues a multi-line declaration, call, or initializer. / 继续一个跨多行的声明、调用或初始化。
- **L145**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L146**: Assigns or initializes state used by the surrounding logic. / 赋值或初始化周边逻辑要使用的状态。
- **L147**: Assigns or initializes state used by the surrounding logic. / 赋值或初始化周边逻辑要使用的状态。
- **L148**: Assigns or initializes state used by the surrounding logic. / 赋值或初始化周边逻辑要使用的状态。
- **L149**: Assigns or initializes state used by the surrounding logic. / 赋值或初始化周边逻辑要使用的状态。
- **L150**: Assigns or initializes state used by the surrounding logic. / 赋值或初始化周边逻辑要使用的状态。
- **L151**: Assigns or initializes state used by the surrounding logic. / 赋值或初始化周边逻辑要使用的状态。
- **L152**: Assigns or initializes state used by the surrounding logic. / 赋值或初始化周边逻辑要使用的状态。
- **L153**: Assigns or initializes state used by the surrounding logic. / 赋值或初始化周边逻辑要使用的状态。
- **L154**: Assigns or initializes state used by the surrounding logic. / 赋值或初始化周边逻辑要使用的状态。
- **L155**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L156**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。

### Lines 157-166 / 第 157-166 行

```cpp
 157: // Group InputSections into clusters using the Call-Chain Clustering heuristic
 158: // then sort the clusters by density.
 159: DenseMap<const InputSection *, int> CallGraphSort::run() {
 160:   const uint64_t maxClusterSize = target->getPageSize();
 161: 
 162:   // Cluster indices sorted by density.
 163:   std::vector<int> sorted(clusters.size());
 164:   // For union-find.
 165:   std::vector<int> leaders(clusters.size());
 166: 
```

- **L157**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L158**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L159**: Defines function or method \`run\`. / 定义函数或方法 \`run\`。
- **L160**: Declares function or method \`getPageSize\`. / 声明函数或方法 \`getPageSize\`。
- **L161**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L162**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L163**: Declares function or method \`sorted\`. / 声明函数或方法 \`sorted\`。
- **L164**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L165**: Declares function or method \`leaders\`. / 声明函数或方法 \`leaders\`。
- **L166**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。

### Lines 167-178 / 第 167-178 行

```cpp
 167:   std::iota(leaders.begin(), leaders.end(), 0);
 168:   std::iota(sorted.begin(), sorted.end(), 0);
 169: 
 170:   llvm::stable_sort(sorted, [&](int a, int b) {
 171:     return clusters[a].getDensity() > clusters[b].getDensity();
 172:   });
 173: 
 174:   for (int l : sorted) {
 175:     // The cluster index is the same as the index of its leader here because
 176:     // clusters[L] has not been merged into another cluster yet.
 177:     Cluster &c = clusters[l];
 178: 
```

- **L167**: Declares function or method \`iota\`. / 声明函数或方法 \`iota\`。
- **L168**: Declares function or method \`iota\`. / 声明函数或方法 \`iota\`。
- **L169**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L170**: Defines function or method \`stable_sort\`. / 定义函数或方法 \`stable_sort\`。
- **L171**: Returns a value or exits the current function at this point. / 在此返回一个值，或直接结束当前函数。
- **L172**: Closes the current declaration or scoped construct. / 结束当前声明或带作用域的结构。
- **L173**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L174**: Starts a counted or iterator-based loop. / 开始一个计数式或迭代器式循环。
- **L175**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L176**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L177**: Assigns or initializes state used by the surrounding logic. / 赋值或初始化周边逻辑要使用的状态。
- **L178**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。

### Lines 179-187 / 第 179-187 行

```cpp
 179:     // Don't consider merging if the edge is unlikely.
 180:     if (c.bestPred.from == -1 || c.bestPred.weight * 10 <= c.initialWeight)
 181:       continue;
 182: 
 183:     int predL = getLeader(leaders, c.bestPred.from);
 184:     // Already in the same cluster.
 185:     if (l == predL)
 186:       continue;
 187: 
```

- **L179**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L180**: Starts a conditional branch that runs only when the predicate is true. / 开始一个条件分支，仅在判定条件为真时执行。
- **L181**: Skips directly to the next loop iteration. / 直接跳到下一次循环迭代。
- **L182**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L183**: Declares function or method \`getLeader\`. / 声明函数或方法 \`getLeader\`。
- **L184**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L185**: Starts a conditional branch that runs only when the predicate is true. / 开始一个条件分支，仅在判定条件为真时执行。
- **L186**: Skips directly to the next loop iteration. / 直接跳到下一次循环迭代。
- **L187**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。

### Lines 188-205 / 第 188-205 行

```cpp
 188:     Cluster *predC = &clusters[predL];
 189:     if (c.size + predC->size > maxClusterSize)
 190:       continue;
 191: 
 192:     if (isNewDensityBad(*predC, c))
 193:       continue;
 194: 
 195:     leaders[l] = predL;
 196:     mergeClusters(clusters, *predC, predL, c, l);
 197:   }
 198:   // Sort remaining non-empty clusters by density.
 199:   sorted.clear();
 200:   for (int i = 0, e = (int)clusters.size(); i != e; ++i)
 201:     if (clusters[i].size > 0)
 202:       sorted.push_back(i);
 203:   llvm::stable_sort(sorted, [&](int a, int b) {
 204:     return clusters[a].getDensity() > clusters[b].getDensity();
 205:   });
```

- **L188**: Assigns or initializes state used by the surrounding logic. / 赋值或初始化周边逻辑要使用的状态。
- **L189**: Starts a conditional branch that runs only when the predicate is true. / 开始一个条件分支，仅在判定条件为真时执行。
- **L190**: Skips directly to the next loop iteration. / 直接跳到下一次循环迭代。
- **L191**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L192**: Starts a conditional branch that runs only when the predicate is true. / 开始一个条件分支，仅在判定条件为真时执行。
- **L193**: Skips directly to the next loop iteration. / 直接跳到下一次循环迭代。
- **L194**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L195**: Assigns or initializes state used by the surrounding logic. / 赋值或初始化周边逻辑要使用的状态。
- **L196**: Declares function or method \`mergeClusters\`. / 声明函数或方法 \`mergeClusters\`。
- **L197**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L198**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L199**: Declares function or method \`clear\`. / 声明函数或方法 \`clear\`。
- **L200**: Starts a counted or iterator-based loop. / 开始一个计数式或迭代器式循环。
- **L201**: Starts a conditional branch that runs only when the predicate is true. / 开始一个条件分支，仅在判定条件为真时执行。
- **L202**: Declares function or method \`push_back\`. / 声明函数或方法 \`push_back\`。
- **L203**: Defines function or method \`stable_sort\`. / 定义函数或方法 \`stable_sort\`。
- **L204**: Returns a value or exits the current function at this point. / 在此返回一个值，或直接结束当前函数。
- **L205**: Closes the current declaration or scoped construct. / 结束当前声明或带作用域的结构。

### Lines 206-223 / 第 206-223 行

```cpp
 206: 
 207:   DenseMap<const InputSection *, int> orderMap;
 208: 
 209:   // Sections will be sorted by decreasing order. Absent sections will have
 210:   // priority 0 and be placed at the end of sections.
 211:   int curOrder = -clusters.size();
 212:   for (int leader : sorted) {
 213:     for (int i = leader;;) {
 214:       orderMap[sections[i]] = curOrder++;
 215:       i = clusters[i].next;
 216:       if (i == leader)
 217:         break;
 218:     }
 219:   }
 220:   if (!config->printSymbolOrder.empty()) {
 221:     std::error_code ec;
 222:     raw_fd_ostream os(config->printSymbolOrder, ec, sys::fs::OF_None);
 223:     if (ec) {
```

- **L206**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L207**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L208**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L209**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L210**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L211**: Declares function or method \`size\`. / 声明函数或方法 \`size\`。
- **L212**: Starts a counted or iterator-based loop. / 开始一个计数式或迭代器式循环。
- **L213**: Starts a counted or iterator-based loop. / 开始一个计数式或迭代器式循环。
- **L214**: Assigns or initializes state used by the surrounding logic. / 赋值或初始化周边逻辑要使用的状态。
- **L215**: Assigns or initializes state used by the surrounding logic. / 赋值或初始化周边逻辑要使用的状态。
- **L216**: Starts a conditional branch that runs only when the predicate is true. / 开始一个条件分支，仅在判定条件为真时执行。
- **L217**: Terminates the nearest loop or switch branch. / 终止最近一层的循环或 \`switch\` 分支。
- **L218**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L219**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L220**: Starts a conditional branch that runs only when the predicate is true. / 开始一个条件分支，仅在判定条件为真时执行。
- **L221**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L222**: Declares function or method \`os\`. / 声明函数或方法 \`os\`。
- **L223**: Starts a conditional branch that runs only when the predicate is true. / 开始一个条件分支，仅在判定条件为真时执行。

### Lines 224-241 / 第 224-241 行

```cpp
 224:       error("cannot open " + config->printSymbolOrder + ": " + ec.message());
 225:       return orderMap;
 226:     }
 227:     // Print the symbols ordered by C3, in the order of decreasing curOrder
 228:     // Instead of sorting all the orderMap, just repeat the loops above.
 229:     for (int leader : sorted)
 230:       for (int i = leader;;) {
 231:         const InputSection *isec = sections[i];
 232:         // Search all the symbols in the file of the section
 233:         // and find out a Defined symbol with name that is within the
 234:         // section.
 235:         for (Symbol *sym : isec->getFile()->symbols) {
 236:           if (auto *d = dyn_cast_or_null<Defined>(sym)) {
 237:             if (d->isec() == isec)
 238:               os << sym->getName() << "\n";
 239:           }
 240:         }
 241:         i = clusters[i].next;
```

- **L224**: Declares function or method \`error\`. / 声明函数或方法 \`error\`。
- **L225**: Returns a value or exits the current function at this point. / 在此返回一个值，或直接结束当前函数。
- **L226**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L227**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L228**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L229**: Starts a counted or iterator-based loop. / 开始一个计数式或迭代器式循环。
- **L230**: Starts a counted or iterator-based loop. / 开始一个计数式或迭代器式循环。
- **L231**: Assigns or initializes state used by the surrounding logic. / 赋值或初始化周边逻辑要使用的状态。
- **L232**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L233**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L234**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L235**: Starts a counted or iterator-based loop. / 开始一个计数式或迭代器式循环。
- **L236**: Starts a conditional branch that runs only when the predicate is true. / 开始一个条件分支，仅在判定条件为真时执行。
- **L237**: Starts a conditional branch that runs only when the predicate is true. / 开始一个条件分支，仅在判定条件为真时执行。
- **L238**: Calls a helper routine or completes a callable declaration. / 调用辅助例程或完成一个可调用对象的声明。
- **L239**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L240**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L241**: Assigns or initializes state used by the surrounding logic. / 赋值或初始化周边逻辑要使用的状态。

### Lines 242-257 / 第 242-257 行

```cpp
 242:         if (i == leader)
 243:           break;
 244:       }
 245:   }
 246: 
 247:   return orderMap;
 248: }
 249: 
 250: void macho::PriorityBuilder::SymbolPriorityEntry::setPriority(
 251:     int priority, StringRef objectFile) {
 252:   if (!objectFile.empty())
 253:     objectFiles.try_emplace(objectFile, priority);
 254:   else
 255:     anyObjectFile = std::min(anyObjectFile, priority);
 256: }
 257: 
```

- **L242**: Starts a conditional branch that runs only when the predicate is true. / 开始一个条件分支，仅在判定条件为真时执行。
- **L243**: Terminates the nearest loop or switch branch. / 终止最近一层的循环或 \`switch\` 分支。
- **L244**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L245**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L246**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L247**: Returns a value or exits the current function at this point. / 在此返回一个值，或直接结束当前函数。
- **L248**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L249**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L250**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L251**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L252**: Starts a conditional branch that runs only when the predicate is true. / 开始一个条件分支，仅在判定条件为真时执行。
- **L253**: Declares function or method \`try_emplace\`. / 声明函数或方法 \`try_emplace\`。
- **L254**: Begins the fallback branch for the preceding conditional logic. / 开始前一个条件逻辑的后备分支。
- **L255**: Declares function or method \`min\`. / 声明函数或方法 \`min\`。
- **L256**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L257**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。

### Lines 258-271 / 第 258-271 行

```cpp
 258: int macho::PriorityBuilder::SymbolPriorityEntry::getPriority(
 259:     const InputFile *f) const {
 260:   if (!f)
 261:     return anyObjectFile;
 262:   // We don't use toString(InputFile *) here because it returns the full path
 263:   // for object files, and we only want the basename.
 264:   StringRef basename = path::filename(f->getName());
 265:   StringRef filename =
 266:       f->archiveName.empty()
 267:           ? basename
 268:           : saver().save(path::filename(f->archiveName) + "(" + basename + ")");
 269:   return std::min(objectFiles.lookup(filename), anyObjectFile);
 270: }
 271: 
```

- **L258**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L259**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L260**: Starts a conditional branch that runs only when the predicate is true. / 开始一个条件分支，仅在判定条件为真时执行。
- **L261**: Returns a value or exits the current function at this point. / 在此返回一个值，或直接结束当前函数。
- **L262**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L263**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L264**: Declares function or method \`filename\`. / 声明函数或方法 \`filename\`。
- **L265**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L266**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L267**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L268**: Declares function or method \`saver\`. / 声明函数或方法 \`saver\`。
- **L269**: Returns a value or exits the current function at this point. / 在此返回一个值，或直接结束当前函数。
- **L270**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L271**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。

### Lines 272-280 / 第 272-280 行

```cpp
 272: std::optional<int>
 273: macho::PriorityBuilder::getCStringPriority(uint32_t hash,
 274:                                            const InputFile *f) const {
 275:   auto it = cStringPriorities.find(hash);
 276:   if (it == cStringPriorities.end())
 277:     return std::nullopt;
 278:   return it->second.getPriority(f);
 279: }
 280: 
```

- **L272**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L273**: Continues a multi-line declaration, call, or initializer. / 继续一个跨多行的声明、调用或初始化。
- **L274**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L275**: Declares function or method \`find\`. / 声明函数或方法 \`find\`。
- **L276**: Starts a conditional branch that runs only when the predicate is true. / 开始一个条件分支，仅在判定条件为真时执行。
- **L277**: Returns a value or exits the current function at this point. / 在此返回一个值，或直接结束当前函数。
- **L278**: Returns a value or exits the current function at this point. / 在此返回一个值，或直接结束当前函数。
- **L279**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L280**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。

### Lines 281-290 / 第 281-290 行

```cpp
 281: std::optional<int>
 282: macho::PriorityBuilder::getSymbolPriority(const Defined *sym) const {
 283:   if (sym->isAbsolute())
 284:     return std::nullopt;
 285:   auto it = priorities.find(utils::getRootSymbol(sym->getName()));
 286:   if (it == priorities.end())
 287:     return std::nullopt;
 288:   return it->second.getPriority(sym->isec()->getFile());
 289: }
 290: 
```

- **L281**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L282**: Defines function or method \`getSymbolPriority\`. / 定义函数或方法 \`getSymbolPriority\`。
- **L283**: Starts a conditional branch that runs only when the predicate is true. / 开始一个条件分支，仅在判定条件为真时执行。
- **L284**: Returns a value or exits the current function at this point. / 在此返回一个值，或直接结束当前函数。
- **L285**: Declares function or method \`find\`. / 声明函数或方法 \`find\`。
- **L286**: Starts a conditional branch that runs only when the predicate is true. / 开始一个条件分支，仅在判定条件为真时执行。
- **L287**: Returns a value or exits the current function at this point. / 在此返回一个值，或直接结束当前函数。
- **L288**: Returns a value or exits the current function at this point. / 在此返回一个值，或直接结束当前函数。
- **L289**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L290**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。

### Lines 291-308 / 第 291-308 行

```cpp
 291: void macho::PriorityBuilder::extractCallGraphProfile() {
 292:   TimeTraceScope timeScope("Extract call graph profile");
 293:   bool hasOrderFile = !priorities.empty();
 294:   for (const InputFile *file : inputFiles) {
 295:     auto *obj = dyn_cast_or_null<ObjFile>(file);
 296:     if (!obj)
 297:       continue;
 298:     for (const CallGraphEntry &entry : obj->callGraph) {
 299:       assert(entry.fromIndex < obj->symbols.size() &&
 300:              entry.toIndex < obj->symbols.size());
 301:       auto *fromSym = dyn_cast_or_null<Defined>(obj->symbols[entry.fromIndex]);
 302:       auto *toSym = dyn_cast_or_null<Defined>(obj->symbols[entry.toIndex]);
 303:       if (fromSym && toSym &&
 304:           (!hasOrderFile ||
 305:            (!getSymbolPriority(fromSym) && !getSymbolPriority(toSym))))
 306:         callGraphProfile[{fromSym->isec(), toSym->isec()}] += entry.count;
 307:     }
 308:   }
```

- **L291**: Defines function or method \`extractCallGraphProfile\`. / 定义函数或方法 \`extractCallGraphProfile\`。
- **L292**: Declares function or method \`timeScope\`. / 声明函数或方法 \`timeScope\`。
- **L293**: Declares function or method \`empty\`. / 声明函数或方法 \`empty\`。
- **L294**: Starts a counted or iterator-based loop. / 开始一个计数式或迭代器式循环。
- **L295**: Declares function or method \`dyn_cast_or_null\`. / 声明函数或方法 \`dyn_cast_or_null\`。
- **L296**: Starts a conditional branch that runs only when the predicate is true. / 开始一个条件分支，仅在判定条件为真时执行。
- **L297**: Skips directly to the next loop iteration. / 直接跳到下一次循环迭代。
- **L298**: Starts a counted or iterator-based loop. / 开始一个计数式或迭代器式循环。
- **L299**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L300**: Declares function or method \`size\`. / 声明函数或方法 \`size\`。
- **L301**: Declares function or method \`dyn_cast_or_null\`. / 声明函数或方法 \`dyn_cast_or_null\`。
- **L302**: Declares function or method \`dyn_cast_or_null\`. / 声明函数或方法 \`dyn_cast_or_null\`。
- **L303**: Starts a conditional branch that runs only when the predicate is true. / 开始一个条件分支，仅在判定条件为真时执行。
- **L304**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L305**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L306**: Assigns or initializes state used by the surrounding logic. / 赋值或初始化周边逻辑要使用的状态。
- **L307**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L308**: Closes the current scope or body. / 关闭当前作用域或代码体。

### Lines 309-319 / 第 309-319 行

```cpp
 309: }
 310: 
 311: void macho::PriorityBuilder::parseOrderFile(StringRef path) {
 312:   assert(callGraphProfile.empty() &&
 313:          "Order file must be parsed before call graph profile is processed");
 314:   std::optional<MemoryBufferRef> buffer = readFile(path);
 315:   if (!buffer) {
 316:     error("Could not read order file at " + path);
 317:     return;
 318:   }
 319: 
```

- **L309**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L310**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L311**: Defines function or method \`parseOrderFile\`. / 定义函数或方法 \`parseOrderFile\`。
- **L312**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L313**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L314**: Declares function or method \`readFile\`. / 声明函数或方法 \`readFile\`。
- **L315**: Starts a conditional branch that runs only when the predicate is true. / 开始一个条件分支，仅在判定条件为真时执行。
- **L316**: Declares function or method \`error\`. / 声明函数或方法 \`error\`。
- **L317**: Returns a value or exits the current function at this point. / 在此返回一个值，或直接结束当前函数。
- **L318**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L319**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。

### Lines 320-335 / 第 320-335 行

```cpp
 320:   int prio = std::numeric_limits<int>::min();
 321:   MemoryBufferRef mbref = *buffer;
 322:   for (StringRef line : args::getLines(mbref)) {
 323:     StringRef objectFile;
 324:     line = line.take_until([](char c) { return c == '#'; }); // ignore comments
 325:     line = line.ltrim();
 326: 
 327:     CPUType cpuType = StringSwitch<CPUType>(line)
 328:                           .StartsWith("i386:", CPU_TYPE_I386)
 329:                           .StartsWith("x86_64:", CPU_TYPE_X86_64)
 330:                           .StartsWith("arm:", CPU_TYPE_ARM)
 331:                           .StartsWith("arm64:", CPU_TYPE_ARM64)
 332:                           .StartsWith("ppc:", CPU_TYPE_POWERPC)
 333:                           .StartsWith("ppc64:", CPU_TYPE_POWERPC64)
 334:                           .Default(CPU_TYPE_ANY);
 335: 
```

- **L320**: Declares function or method \`min\`. / 声明函数或方法 \`min\`。
- **L321**: Assigns or initializes state used by the surrounding logic. / 赋值或初始化周边逻辑要使用的状态。
- **L322**: Starts a counted or iterator-based loop. / 开始一个计数式或迭代器式循环。
- **L323**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L324**: Defines function or method \`take_until\`. / 定义函数或方法 \`take_until\`。
- **L325**: Declares function or method \`ltrim\`. / 声明函数或方法 \`ltrim\`。
- **L326**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L327**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L328**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L329**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L330**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L331**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L332**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L333**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L334**: Declares function or method \`Default\`. / 声明函数或方法 \`Default\`。
- **L335**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。

### Lines 336-352 / 第 336-352 行

```cpp
 336:     if (cpuType != CPU_TYPE_ANY && cpuType != target->cpuType)
 337:       continue;
 338:     // Drop the CPU type as well as the colon
 339:     if (cpuType != CPU_TYPE_ANY)
 340:       line = line.drop_until([](char c) { return c == ':'; }).drop_front();
 341: 
 342:     constexpr std::array<StringRef, 2> fileEnds = {".o:", ".o):"};
 343:     for (StringRef fileEnd : fileEnds) {
 344:       size_t pos = line.find(fileEnd);
 345:       if (pos != StringRef::npos) {
 346:         // Split the string around the colon
 347:         objectFile = line.take_front(pos + fileEnd.size() - 1);
 348:         line = line.drop_front(pos + fileEnd.size());
 349:         break;
 350:       }
 351:     }
 352: 
```

- **L336**: Starts a conditional branch that runs only when the predicate is true. / 开始一个条件分支，仅在判定条件为真时执行。
- **L337**: Skips directly to the next loop iteration. / 直接跳到下一次循环迭代。
- **L338**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L339**: Starts a conditional branch that runs only when the predicate is true. / 开始一个条件分支，仅在判定条件为真时执行。
- **L340**: Declares function or method \`drop_until\`. / 声明函数或方法 \`drop_until\`。
- **L341**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L342**: Closes the current declaration or scoped construct. / 结束当前声明或带作用域的结构。
- **L343**: Starts a counted or iterator-based loop. / 开始一个计数式或迭代器式循环。
- **L344**: Declares function or method \`find\`. / 声明函数或方法 \`find\`。
- **L345**: Starts a conditional branch that runs only when the predicate is true. / 开始一个条件分支，仅在判定条件为真时执行。
- **L346**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L347**: Declares function or method \`take_front\`. / 声明函数或方法 \`take_front\`。
- **L348**: Declares function or method \`drop_front\`. / 声明函数或方法 \`drop_front\`。
- **L349**: Terminates the nearest loop or switch branch. / 终止最近一层的循环或 \`switch\` 分支。
- **L350**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L351**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L352**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。

### Lines 353-367 / 第 353-367 行

```cpp
 353:     // The rest of the line is either <symbol name> or
 354:     // cStringEntryPrefix<cstring hash>
 355:     line = line.trim();
 356:     if (line.consume_front(cStringEntryPrefix)) {
 357:       uint32_t hash = 0;
 358:       // Only accept hex (0x prefix) or decimal format
 359:       if (line.consume_front_insensitive("0x") ? !line.getAsInteger(16, hash)
 360:                                                : !line.getAsInteger(10, hash))
 361:         cStringPriorities[hash].setPriority(prio, objectFile);
 362:     } else {
 363:       StringRef symbol = utils::getRootSymbol(line);
 364:       if (!symbol.empty())
 365:         priorities[symbol].setPriority(prio, objectFile);
 366:     }
 367: 
```

- **L353**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L354**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L355**: Declares function or method \`trim\`. / 声明函数或方法 \`trim\`。
- **L356**: Starts a conditional branch that runs only when the predicate is true. / 开始一个条件分支，仅在判定条件为真时执行。
- **L357**: Assigns or initializes state used by the surrounding logic. / 赋值或初始化周边逻辑要使用的状态。
- **L358**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L359**: Starts a conditional branch that runs only when the predicate is true. / 开始一个条件分支，仅在判定条件为真时执行。
- **L360**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L361**: Declares function or method \`setPriority\`. / 声明函数或方法 \`setPriority\`。
- **L362**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L363**: Declares function or method \`getRootSymbol\`. / 声明函数或方法 \`getRootSymbol\`。
- **L364**: Starts a conditional branch that runs only when the predicate is true. / 开始一个条件分支，仅在判定条件为真时执行。
- **L365**: Declares function or method \`setPriority\`. / 声明函数或方法 \`setPriority\`。
- **L366**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L367**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。

### Lines 368-385 / 第 368-385 行

```cpp
 368:     ++prio;
 369:   }
 370: }
 371: 
 372: DenseMap<const InputSection *, int>
 373: macho::PriorityBuilder::buildInputSectionPriorities() {
 374:   DenseMap<const InputSection *, int> sectionPriorities;
 375:   if (config->bpStartupFunctionSort || config->bpFunctionOrderForCompression ||
 376:       config->bpDataOrderForCompression ||
 377:       !config->bpCompressionSortSpecs.empty()) {
 378:     TimeTraceScope timeScope("Balanced Partitioning Section Orderer");
 379:     sectionPriorities = runBalancedPartitioning(
 380:         config->bpStartupFunctionSort ? config->irpgoProfilePath : "",
 381:         config->bpCompressionSortSpecs, config->bpFunctionOrderForCompression,
 382:         config->bpDataOrderForCompression,
 383:         config->bpCompressionSortStartupFunctions,
 384:         config->bpVerboseSectionOrderer);
 385:   } else if (config->callGraphProfileSort) {
```

- **L368**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L369**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L370**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L371**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L372**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L373**: Defines function or method \`buildInputSectionPriorities\`. / 定义函数或方法 \`buildInputSectionPriorities\`。
- **L374**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L375**: Starts a conditional branch that runs only when the predicate is true. / 开始一个条件分支，仅在判定条件为真时执行。
- **L376**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L377**: Defines function or method \`empty\`. / 定义函数或方法 \`empty\`。
- **L378**: Declares function or method \`timeScope\`. / 声明函数或方法 \`timeScope\`。
- **L379**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L380**: Continues a multi-line declaration, call, or initializer. / 继续一个跨多行的声明、调用或初始化。
- **L381**: Continues a multi-line declaration, call, or initializer. / 继续一个跨多行的声明、调用或初始化。
- **L382**: Continues a multi-line declaration, call, or initializer. / 继续一个跨多行的声明、调用或初始化。
- **L383**: Continues a multi-line declaration, call, or initializer. / 继续一个跨多行的声明、调用或初始化。
- **L384**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L385**: Defines function or method \`if\`. / 定义函数或方法 \`if\`。

### Lines 386-395 / 第 386-395 行

```cpp
 386:     // Sort sections by the profile data provided by __LLVM,__cg_profile
 387:     // sections.
 388:     //
 389:     // This first builds a call graph based on the profile data then merges
 390:     // sections according to the C³ heuristic. All clusters are then sorted by a
 391:     // density metric to further improve locality.
 392:     TimeTraceScope timeScope("Call graph profile sort");
 393:     sectionPriorities = CallGraphSort(callGraphProfile).run();
 394:   }
 395: 
```

- **L386**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L387**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L388**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L389**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L390**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L391**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L392**: Declares function or method \`timeScope\`. / 声明函数或方法 \`timeScope\`。
- **L393**: Declares function or method \`CallGraphSort\`. / 声明函数或方法 \`CallGraphSort\`。
- **L394**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L395**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。

### Lines 396-406 / 第 396-406 行

```cpp
 396:   if (priorities.empty())
 397:     return sectionPriorities;
 398: 
 399:   auto addSym = [&](const Defined *sym) {
 400:     std::optional<int> symbolPriority = getSymbolPriority(sym);
 401:     if (!symbolPriority)
 402:       return;
 403:     int &priority = sectionPriorities[sym->isec()];
 404:     priority = std::min(priority, *symbolPriority);
 405:   };
 406: 
```

- **L396**: Starts a conditional branch that runs only when the predicate is true. / 开始一个条件分支，仅在判定条件为真时执行。
- **L397**: Returns a value or exits the current function at this point. / 在此返回一个值，或直接结束当前函数。
- **L398**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L399**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L400**: Declares function or method \`getSymbolPriority\`. / 声明函数或方法 \`getSymbolPriority\`。
- **L401**: Starts a conditional branch that runs only when the predicate is true. / 开始一个条件分支，仅在判定条件为真时执行。
- **L402**: Returns a value or exits the current function at this point. / 在此返回一个值，或直接结束当前函数。
- **L403**: Assigns or initializes state used by the surrounding logic. / 赋值或初始化周边逻辑要使用的状态。
- **L404**: Declares function or method \`min\`. / 声明函数或方法 \`min\`。
- **L405**: Closes the current declaration or scoped construct. / 结束当前声明或带作用域的结构。
- **L406**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。

### Lines 407-417 / 第 407-417 行

```cpp
 407:   // TODO: Make sure this handles weak symbols correctly.
 408:   for (const InputFile *file : inputFiles) {
 409:     if (isa<ObjFile>(file))
 410:       for (Symbol *sym : file->symbols)
 411:         if (auto *d = dyn_cast_or_null<Defined>(sym))
 412:           addSym(d);
 413:   }
 414: 
 415:   return sectionPriorities;
 416: }
 417: 
```

- **L407**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L408**: Starts a counted or iterator-based loop. / 开始一个计数式或迭代器式循环。
- **L409**: Starts a conditional branch that runs only when the predicate is true. / 开始一个条件分支，仅在判定条件为真时执行。
- **L410**: Starts a counted or iterator-based loop. / 开始一个计数式或迭代器式循环。
- **L411**: Starts a conditional branch that runs only when the predicate is true. / 开始一个条件分支，仅在判定条件为真时执行。
- **L412**: Declares function or method \`addSym\`. / 声明函数或方法 \`addSym\`。
- **L413**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L414**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L415**: Returns a value or exits the current function at this point. / 在此返回一个值，或直接结束当前函数。
- **L416**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L417**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。

### Lines 418-435 / 第 418-435 行

```cpp
 418: void macho::PriorityBuilder::forEachStringPiece(
 419:     ArrayRef<CStringInputSection *> inputs,
 420:     std::function<void(CStringInputSection &, StringPiece &, size_t)> f,
 421:     bool forceInputOrder, bool computeHash) const {
 422:   std::vector<std::tuple<int, CStringInputSection *, size_t>> orderedPieces;
 423:   std::vector<std::pair<CStringInputSection *, size_t>> unorderedPieces;
 424:   for (CStringInputSection *isec : inputs) {
 425:     for (const auto &[stringPieceIdx, piece] : llvm::enumerate(isec->pieces)) {
 426:       if (!piece.live)
 427:         continue;
 428:       // Process pieces in input order if we have no cstrings in our orderfile
 429:       if (forceInputOrder || cStringPriorities.empty()) {
 430:         f(*isec, piece, stringPieceIdx);
 431:         continue;
 432:       }
 433:       uint32_t hash =
 434:           computeHash
 435:               ? (xxh3_64bits(isec->getStringRef(stringPieceIdx)) & 0x7fffffff)
```

- **L418**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L419**: Continues a multi-line declaration, call, or initializer. / 继续一个跨多行的声明、调用或初始化。
- **L420**: Continues a multi-line declaration, call, or initializer. / 继续一个跨多行的声明、调用或初始化。
- **L421**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L422**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L423**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L424**: Starts a counted or iterator-based loop. / 开始一个计数式或迭代器式循环。
- **L425**: Starts a counted or iterator-based loop. / 开始一个计数式或迭代器式循环。
- **L426**: Starts a conditional branch that runs only when the predicate is true. / 开始一个条件分支，仅在判定条件为真时执行。
- **L427**: Skips directly to the next loop iteration. / 直接跳到下一次循环迭代。
- **L428**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L429**: Starts a conditional branch that runs only when the predicate is true. / 开始一个条件分支，仅在判定条件为真时执行。
- **L430**: Declares function or method \`f\`. / 声明函数或方法 \`f\`。
- **L431**: Skips directly to the next loop iteration. / 直接跳到下一次循环迭代。
- **L432**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L433**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L434**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L435**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。

### Lines 436-453 / 第 436-453 行

```cpp
 436:               : piece.hash;
 437:       if (auto priority = getCStringPriority(hash, isec->getFile()))
 438:         orderedPieces.emplace_back(*priority, isec, stringPieceIdx);
 439:       else
 440:         unorderedPieces.emplace_back(isec, stringPieceIdx);
 441:     }
 442:   }
 443:   if (orderedPieces.empty() && unorderedPieces.empty())
 444:     return;
 445:   llvm::stable_sort(orderedPieces, [](const auto &left, const auto &right) {
 446:     return std::get<0>(left) < std::get<0>(right);
 447:   });
 448:   for (auto &[priority, isec, pieceIdx] : orderedPieces)
 449:     f(*isec, isec->pieces[pieceIdx], pieceIdx);
 450:   // TODO: Add option to order the remaining cstrings for compression
 451:   for (auto &[isec, pieceIdx] : unorderedPieces)
 452:     f(*isec, isec->pieces[pieceIdx], pieceIdx);
 453: }
```

- **L436**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L437**: Starts a conditional branch that runs only when the predicate is true. / 开始一个条件分支，仅在判定条件为真时执行。
- **L438**: Declares function or method \`emplace_back\`. / 声明函数或方法 \`emplace_back\`。
- **L439**: Begins the fallback branch for the preceding conditional logic. / 开始前一个条件逻辑的后备分支。
- **L440**: Declares function or method \`emplace_back\`. / 声明函数或方法 \`emplace_back\`。
- **L441**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L442**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L443**: Starts a conditional branch that runs only when the predicate is true. / 开始一个条件分支，仅在判定条件为真时执行。
- **L444**: Returns a value or exits the current function at this point. / 在此返回一个值，或直接结束当前函数。
- **L445**: Defines function or method \`stable_sort\`. / 定义函数或方法 \`stable_sort\`。
- **L446**: Returns a value or exits the current function at this point. / 在此返回一个值，或直接结束当前函数。
- **L447**: Closes the current declaration or scoped construct. / 结束当前声明或带作用域的结构。
- **L448**: Starts a counted or iterator-based loop. / 开始一个计数式或迭代器式循环。
- **L449**: Declares function or method \`f\`. / 声明函数或方法 \`f\`。
- **L450**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L451**: Starts a counted or iterator-based loop. / 开始一个计数式或迭代器式循环。
- **L452**: Declares function or method \`f\`. / 声明函数或方法 \`f\`。
- **L453**: Closes the current scope or body. / 关闭当前作用域或代码体。

## Key Concepts / 关键概念

- **Role / 角色**: This is based on the ELF port, see ELF/CallGraphSort.cpp for the details about the algorithm. / 实现 Mach-O 链接器的符号图、地址分配以及输出文件构建逻辑。
- **Scale / 规模**: 453 lines, 17 direct includes, 3 named types, and 40 detected routines. / 共 453 行，含 17 个直接包含、3 个具名类型、40 个检测到的例程。
- **Mach-O linking / Mach-O 链接**: The code manages Mach-O atoms, symbols, segments, or output-file construction. / 该代码管理 Mach-O 原子、符号、段或输出文件构建。
- **Symbol management / 符号管理**: The code tracks declarations, references, visibility, or resolution of symbols. / 该代码跟踪符号的声明、引用、可见性或解析过程。
- **Section/segment layout / 节/段布局**: The file manipulates input or output sections, segments, or related layout state. / 该文件操作输入/输出节、段或相关布局状态。
- **Rich type modeling / 丰富的类型建模**: The file introduces or manipulates named types that capture subsystem state. / 该文件引入或操作用于表达子系统状态的具名类型。

## Dependencies / 依赖关系

- **LLVM / LLVM**: `llvm/ADT/DenseMap.h`, `llvm/ADT/MapVector.h`, `llvm/Support/Path.h`, `llvm/Support/TimeProfiler.h`, `llvm/Support/raw_ostream.h`, `llvm/Support/xxhash.h`.
- **lld / lld**: `lld/Common/Args.h`, `lld/Common/CommonLinkerContext.h`, `lld/Common/ErrorHandler.h`, `lld/Common/Utils.h`.
- **System or local / 系统或本地**: `SectionPriorities.h`, `BPSectionOrderer.h`, `Config.h`, `InputFiles.h`, `Symbols.h`, `Target.h`, `numeric`.
- **Header roles / 头文件角色**: standard-library or local support header / 标准库或本地支持头文件 (7), lld shared linker infrastructure / lld 共享链接基础设施 (4), support-library helpers / Support 库辅助功能 (4), LLVM ADT containers and utility types / LLVM ADT 容器与工具类型 (2).
- **Core types / 核心类型**: `Edge`, `Cluster`, `CallGraphSort`.
- **Visible routines / 可见例程**: `Cluster`, `getDensity`, `double`, `CallGraphSort`, `run`, `try_emplace`, `push_back`, `emplace_back`, `canonical`, `getOrCreateCluster`, `isNewDensityBad`, `getLeader`.
