# Canonicalize.td — Code Analysis / 代码分析

## Source / 来源
- **File / 文件:** `lib/Dialect/Triton/IR/Canonicalize.td`
- **Purpose / 作用:** **EN:** Declares TableGen rules and generated canonicalization definitions for Triton. **CN:** 声明 Triton 的 TableGen 规则与生成式规范化定义。

## Line-by-Line Analysis / 逐行分析

### Lines 1-2

```tablegen
   1: #ifndef TT_PATTERNS
   2: #define TT_PATTERNS
```

- **EN:** Defines or closes a file guard so this header/TableGen fragment is included only once during compilation.
- **CN:** 这里定义或关闭文件保护宏，确保该头文件/TableGen 片段在编译过程中只被包含一次。
### Lines 4-5

```tablegen
   4: include "mlir/IR/PatternBase.td"
   5: include "triton/Dialect/Triton/IR/TritonOps.td"
```

- **EN:** Imports TableGen base definitions or dialect-specific pattern files needed by the declarations below.
- **CN:** 这里导入后续声明所需的 TableGen 基础定义或方言特定模式文件。
### Lines 7-10

```tablegen
   7: // broadcast(splat(x)) -> splat(x)
   8: def BroadcastSplatPattern :
   9:     Pat<(TT_BroadcastOp (TT_SplatOp $x)),
  10:         (TT_SplatOp $x)>;
```

- **EN:** Declares a TableGen canonicalization/rewrite rule that matches one IR shape and rewrites it into a simpler equivalent form.
- **CN:** 这里声明一个 TableGen 规范化/重写规则，用于匹配某种 IR 形态并改写成更简单的等价形式。
### Lines 12-15

```tablegen
  12: // broadcast(broadcast(x)) -> broadcast(x)
  13: def BroadcastBroadcastPattern :
  14:     Pat<(TT_BroadcastOp (TT_BroadcastOp $x)),
  15:         (TT_BroadcastOp $x)>;
```

- **EN:** Declares a TableGen canonicalization/rewrite rule that matches one IR shape and rewrites it into a simpler equivalent form.
- **CN:** 这里声明一个 TableGen 规范化/重写规则，用于匹配某种 IR 形态并改写成更简单的等价形式。
### Lines 17-17

```tablegen
  17: #endif
```

- **EN:** Defines or closes a file guard so this header/TableGen fragment is included only once during compilation.
- **CN:** 这里定义或关闭文件保护宏，确保该头文件/TableGen 片段在编译过程中只被包含一次。

## Key Concepts / 关键概念
- **EN:** The file defines IR semantics for canonicalize in its dialect layer.
  **CN:** 本文件在方言层为 Canonicalize 定义 IR 语义。

## Dependencies / 依赖关系
- **Internal Triton headers / Triton 内部头文件:** `triton/Dialect/Triton/IR/TritonOps.td`
- **MLIR headers / MLIR 头文件:** `mlir/IR/PatternBase.td`
- **LLVM headers / LLVM 头文件:** None
- **Standard/library headers / 标准或通用库头文件:** None
- **IR role / IR 角色:** The code is consumed by parsers, printers, verifiers, folders, and downstream passes that need stable dialect semantics. / 这些代码会被解析器、打印器、验证器、折叠逻辑以及需要稳定方言语义的下游 pass 使用。
