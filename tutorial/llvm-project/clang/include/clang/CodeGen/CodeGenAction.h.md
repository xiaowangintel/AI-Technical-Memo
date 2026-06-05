# CodeGenAction.h — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `clang/include/clang/CodeGen/CodeGenAction.h`
- **Repository / 仓库**: `/root/xw/llvm-project/`
- **Purpose (EN) / 用途（英文）**: LLVM Code Generation Frontend Action -*- C++.
- **Purpose (CN) / 用途（中文）**: 该文件主要涉及：LLVM Code Generation Frontend Action -*- C++。

## Line-by-Line Analysis / 逐行分析

### Lines 1-16 / 第 1-16 行

~~~~cpp
//===--- CodeGenAction.h - LLVM Code Generation Frontend Action -*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#ifndef LLVM_CLANG_CODEGEN_CODEGENACTION_H
#define LLVM_CLANG_CODEGEN_CODEGENACTION_H

#include "clang/Frontend/FrontendAction.h"
#include <memory>

namespace llvm {
  class LLVMContext;
~~~~

- **L1**: Banner comment marks a file or section boundary. / 横幅注释用于标记文件或章节边界。
- **L2**: Comment line provides visual separation or structure. / 注释行提供视觉分隔或结构信息。
- **L3**: Comment documents intent, constraints, or context: `Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.`. / 注释记录设计意图、约束或上下文：`Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.`。
- **L4**: Comment documents intent, constraints, or context: `See https://llvm.org/LICENSE.txt for license information.`. / 注释记录设计意图、约束或上下文：`See https://llvm.org/LICENSE.txt for license information.`。
- **L5**: Comment documents intent, constraints, or context: `SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception`. / 注释记录设计意图、约束或上下文：`SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception`。
- **L6**: Comment line provides visual separation or structure. / 注释行提供视觉分隔或结构信息。
- **L7**: Banner comment marks a file or section boundary. / 横幅注释用于标记文件或章节边界。
- **L8**: Blank line separates nearby declarations or logical regions. / 空行用于分隔附近的声明或逻辑区域。
- **L9**: Starts a preprocessor conditional region controlled by build or feature macros. / 开始一个由构建宏或特性宏控制的预处理条件区域。
- **L10**: Defines macro `LLVM_CLANG_CODEGEN_CODEGENACTION_H` for include guards, configuration, or generated declarations. / 定义宏 `LLVM_CLANG_CODEGEN_CODEGENACTION_H`，用于头文件保护、配置或生成声明。
- **L11**: Blank line separates nearby declarations or logical regions. / 空行用于分隔附近的声明或逻辑区域。
- **L12**: Includes `clang/Frontend/FrontendAction.h` so this file can use declarations from that dependency. / 引入 `clang/Frontend/FrontendAction.h`，使当前文件能够使用该依赖中的声明。
- **L13**: Includes `memory` so this file can use declarations from that dependency. / 引入 `memory`，使当前文件能够使用该依赖中的声明。
- **L14**: Blank line separates nearby declarations or logical regions. / 空行用于分隔附近的声明或逻辑区域。
- **L15**: Opens namespace `llvm` to scope related declarations. / 打开命名空间 `llvm` 以限制相关声明的作用域。
- **L16**: Declares TableGen class `LLVMContext`, which contributes reusable records or generated entities. / 声明 TableGen class `LLVMContext`，用于提供可复用记录或生成实体。

### Lines 17-32 / 第 17-32 行

~~~~cpp
  class Module;
}

namespace clang {
class BackendConsumer;
class CodeGenerator;

class CodeGenAction : public ASTFrontendAction {
private:
  // Let BackendConsumer access LinkModule.
  friend class BackendConsumer;

  /// Info about module to link into a module we're generating.
  struct LinkModule {
    /// The module to link in.
    std::unique_ptr<llvm::Module> Module;
~~~~

- **L17**: Declares TableGen class `Module`, which contributes reusable records or generated entities. / 声明 TableGen class `Module`，用于提供可复用记录或生成实体。
- **L18**: Closes the current scope or body. / 关闭当前作用域或主体。
- **L19**: Blank line separates nearby declarations or logical regions. / 空行用于分隔附近的声明或逻辑区域。
- **L20**: Opens namespace `clang` to scope related declarations. / 打开命名空间 `clang` 以限制相关声明的作用域。
- **L21**: Declares TableGen class `BackendConsumer`, which contributes reusable records or generated entities. / 声明 TableGen class `BackendConsumer`，用于提供可复用记录或生成实体。
- **L22**: Declares TableGen class `CodeGenerator`, which contributes reusable records or generated entities. / 声明 TableGen class `CodeGenerator`，用于提供可复用记录或生成实体。
- **L23**: Blank line separates nearby declarations or logical regions. / 空行用于分隔附近的声明或逻辑区域。
- **L24**: Declares TableGen class `CodeGenAction`, which contributes reusable records or generated entities. / 声明 TableGen class `CodeGenAction`，用于提供可复用记录或生成实体。
- **L25**: Switches subsequent class members to `private` access. / 将后续类成员切换到 `private` 访问级别。
- **L26**: Comment documents intent, constraints, or context: `Let BackendConsumer access LinkModule.`. / 注释记录设计意图、约束或上下文：`Let BackendConsumer access LinkModule.`。
- **L27**: Completes a declaration or executable statement in the current scope. / 在当前作用域中结束一条声明或可执行语句。
- **L28**: Blank line separates nearby declarations or logical regions. / 空行用于分隔附近的声明或逻辑区域。
- **L29**: Comment documents intent, constraints, or context: `Info about module to link into a module we're generating.`. / 注释记录设计意图、约束或上下文：`Info about module to link into a module we're generating.`。
- **L30**: Begins the declaration of struct `LinkModule`. / 开始声明 struct `LinkModule`。
- **L31**: Comment documents intent, constraints, or context: `The module to link in.`. / 注释记录设计意图、约束或上下文：`The module to link in.`。
- **L32**: Completes a declaration or executable statement in the current scope. / 在当前作用域中结束一条声明或可执行语句。

### Lines 33-48 / 第 33-48 行

~~~~cpp

    /// If true, we set attributes on Module's functions according to our
    /// CodeGenOptions and LangOptions, as though we were generating the
    /// function ourselves.
    bool PropagateAttrs;

    /// If true, we use LLVM module internalizer.
    bool Internalize;

    /// Bitwise combination of llvm::LinkerFlags used when we link the module.
    unsigned LinkFlags;
  };

  unsigned Act;
  std::unique_ptr<llvm::Module> TheModule;

~~~~

- **L33**: Blank line separates nearby declarations or logical regions. / 空行用于分隔附近的声明或逻辑区域。
- **L34**: Comment documents intent, constraints, or context: `If true, we set attributes on Module's functions according to our`. / 注释记录设计意图、约束或上下文：`If true, we set attributes on Module's functions according to our`。
- **L35**: Comment documents intent, constraints, or context: `CodeGenOptions and LangOptions, as though we were generating the`. / 注释记录设计意图、约束或上下文：`CodeGenOptions and LangOptions, as though we were generating the`。
- **L36**: Comment documents intent, constraints, or context: `function ourselves.`. / 注释记录设计意图、约束或上下文：`function ourselves.`。
- **L37**: Completes a declaration or executable statement in the current scope. / 在当前作用域中结束一条声明或可执行语句。
- **L38**: Blank line separates nearby declarations or logical regions. / 空行用于分隔附近的声明或逻辑区域。
- **L39**: Comment documents intent, constraints, or context: `If true, we use LLVM module internalizer.`. / 注释记录设计意图、约束或上下文：`If true, we use LLVM module internalizer.`。
- **L40**: Completes a declaration or executable statement in the current scope. / 在当前作用域中结束一条声明或可执行语句。
- **L41**: Blank line separates nearby declarations or logical regions. / 空行用于分隔附近的声明或逻辑区域。
- **L42**: Comment documents intent, constraints, or context: `Bitwise combination of llvm::LinkerFlags used when we link the module.`. / 注释记录设计意图、约束或上下文：`Bitwise combination of llvm::LinkerFlags used when we link the module.`。
- **L43**: Completes a declaration or executable statement in the current scope. / 在当前作用域中结束一条声明或可执行语句。
- **L44**: Closes the current declaration or scoped aggregate. / 结束当前声明或带作用域的聚合结构。
- **L45**: Blank line separates nearby declarations or logical regions. / 空行用于分隔附近的声明或逻辑区域。
- **L46**: Completes a declaration or executable statement in the current scope. / 在当前作用域中结束一条声明或可执行语句。
- **L47**: Completes a declaration or executable statement in the current scope. / 在当前作用域中结束一条声明或可执行语句。
- **L48**: Blank line separates nearby declarations or logical regions. / 空行用于分隔附近的声明或逻辑区域。

### Lines 49-64 / 第 49-64 行

~~~~cpp
  /// Bitcode modules to link in to our module.
  SmallVector<LinkModule, 4> LinkModules;
  llvm::LLVMContext *VMContext;
  bool OwnsVMContext;

  std::unique_ptr<llvm::Module> loadModule(llvm::MemoryBufferRef MBRef);

  /// Load bitcode modules to link into our module from the options.
  bool loadLinkModules(CompilerInstance &CI);

protected:
  bool BeginSourceFileAction(CompilerInstance &CI) override;

  /// Create a new code generation action.  If the optional \p _VMContext
  /// parameter is supplied, the action uses it without taking ownership,
  /// otherwise it creates a fresh LLVM context and takes ownership.
~~~~

- **L49**: Comment documents intent, constraints, or context: `Bitcode modules to link in to our module.`. / 注释记录设计意图、约束或上下文：`Bitcode modules to link in to our module.`。
- **L50**: Completes a declaration or executable statement in the current scope. / 在当前作用域中结束一条声明或可执行语句。
- **L51**: Completes a declaration or executable statement in the current scope. / 在当前作用域中结束一条声明或可执行语句。
- **L52**: Completes a declaration or executable statement in the current scope. / 在当前作用域中结束一条声明或可执行语句。
- **L53**: Blank line separates nearby declarations or logical regions. / 空行用于分隔附近的声明或逻辑区域。
- **L54**: Declares a function, method, or callable interface without defining it here. / 声明一个函数、方法或可调用接口，但不在此处给出定义。
- **L55**: Blank line separates nearby declarations or logical regions. / 空行用于分隔附近的声明或逻辑区域。
- **L56**: Comment documents intent, constraints, or context: `Load bitcode modules to link into our module from the options.`. / 注释记录设计意图、约束或上下文：`Load bitcode modules to link into our module from the options.`。
- **L57**: Declares a function, method, or callable interface without defining it here. / 声明一个函数、方法或可调用接口，但不在此处给出定义。
- **L58**: Blank line separates nearby declarations or logical regions. / 空行用于分隔附近的声明或逻辑区域。
- **L59**: Switches subsequent class members to `protected` access. / 将后续类成员切换到 `protected` 访问级别。
- **L60**: Declares a function, method, or callable interface without defining it here. / 声明一个函数、方法或可调用接口，但不在此处给出定义。
- **L61**: Blank line separates nearby declarations or logical regions. / 空行用于分隔附近的声明或逻辑区域。
- **L62**: Comment documents intent, constraints, or context: `Create a new code generation action. If the optional p _VMContext`. / 注释记录设计意图、约束或上下文：`Create a new code generation action. If the optional p _VMContext`。
- **L63**: Comment documents intent, constraints, or context: `parameter is supplied, the action uses it without taking ownership,`. / 注释记录设计意图、约束或上下文：`parameter is supplied, the action uses it without taking ownership,`。
- **L64**: Comment documents intent, constraints, or context: `otherwise it creates a fresh LLVM context and takes ownership.`. / 注释记录设计意图、约束或上下文：`otherwise it creates a fresh LLVM context and takes ownership.`。

### Lines 65-80 / 第 65-80 行

~~~~cpp
  CodeGenAction(unsigned _Act, llvm::LLVMContext *_VMContext = nullptr);

  bool hasIRSupport() const override;

  std::unique_ptr<ASTConsumer> CreateASTConsumer(CompilerInstance &CI,
                                                 StringRef InFile) override;

  void ExecuteAction() override;

  void EndSourceFileAction() override;

public:
  ~CodeGenAction() override;

  /// Take the generated LLVM module, for use after the action has been run.
  /// The result may be null on failure.
~~~~

- **L65**: Declares a function, method, or callable interface without defining it here. / 声明一个函数、方法或可调用接口，但不在此处给出定义。
- **L66**: Blank line separates nearby declarations or logical regions. / 空行用于分隔附近的声明或逻辑区域。
- **L67**: Declares a function, method, or callable interface without defining it here. / 声明一个函数、方法或可调用接口，但不在此处给出定义。
- **L68**: Blank line separates nearby declarations or logical regions. / 空行用于分隔附近的声明或逻辑区域。
- **L69**: Continues the current declaration, expression, or generated specification. / 继续当前的声明、表达式或生成规格。
- **L70**: Completes a declaration or executable statement in the current scope. / 在当前作用域中结束一条声明或可执行语句。
- **L71**: Blank line separates nearby declarations or logical regions. / 空行用于分隔附近的声明或逻辑区域。
- **L72**: Declares a function, method, or callable interface without defining it here. / 声明一个函数、方法或可调用接口，但不在此处给出定义。
- **L73**: Blank line separates nearby declarations or logical regions. / 空行用于分隔附近的声明或逻辑区域。
- **L74**: Declares a function, method, or callable interface without defining it here. / 声明一个函数、方法或可调用接口，但不在此处给出定义。
- **L75**: Blank line separates nearby declarations or logical regions. / 空行用于分隔附近的声明或逻辑区域。
- **L76**: Switches subsequent class members to `public` access. / 将后续类成员切换到 `public` 访问级别。
- **L77**: Declares a function, method, or callable interface without defining it here. / 声明一个函数、方法或可调用接口，但不在此处给出定义。
- **L78**: Blank line separates nearby declarations or logical regions. / 空行用于分隔附近的声明或逻辑区域。
- **L79**: Comment documents intent, constraints, or context: `Take the generated LLVM module, for use after the action has been run.`. / 注释记录设计意图、约束或上下文：`Take the generated LLVM module, for use after the action has been run.`。
- **L80**: Comment documents intent, constraints, or context: `The result may be null on failure.`. / 注释记录设计意图、约束或上下文：`The result may be null on failure.`。

### Lines 81-96 / 第 81-96 行

~~~~cpp
  std::unique_ptr<llvm::Module> takeModule();

  /// Take the LLVM context used by this action.
  llvm::LLVMContext *takeLLVMContext();

  CodeGenerator *getCodeGenerator() const;

  BackendConsumer *BEConsumer = nullptr;
};

class EmitAssemblyAction : public CodeGenAction {
  virtual void anchor();
public:
  EmitAssemblyAction(llvm::LLVMContext *_VMContext = nullptr);
};

~~~~

- **L81**: Declares a function, method, or callable interface without defining it here. / 声明一个函数、方法或可调用接口，但不在此处给出定义。
- **L82**: Blank line separates nearby declarations or logical regions. / 空行用于分隔附近的声明或逻辑区域。
- **L83**: Comment documents intent, constraints, or context: `Take the LLVM context used by this action.`. / 注释记录设计意图、约束或上下文：`Take the LLVM context used by this action.`。
- **L84**: Declares a function, method, or callable interface without defining it here. / 声明一个函数、方法或可调用接口，但不在此处给出定义。
- **L85**: Blank line separates nearby declarations or logical regions. / 空行用于分隔附近的声明或逻辑区域。
- **L86**: Declares a function, method, or callable interface without defining it here. / 声明一个函数、方法或可调用接口，但不在此处给出定义。
- **L87**: Blank line separates nearby declarations or logical regions. / 空行用于分隔附近的声明或逻辑区域。
- **L88**: Initializes or assigns a value used by the surrounding declarations or logic. / 对周边声明或逻辑使用的值进行初始化或赋值。
- **L89**: Closes the current declaration or scoped aggregate. / 结束当前声明或带作用域的聚合结构。
- **L90**: Blank line separates nearby declarations or logical regions. / 空行用于分隔附近的声明或逻辑区域。
- **L91**: Declares TableGen class `EmitAssemblyAction`, which contributes reusable records or generated entities. / 声明 TableGen class `EmitAssemblyAction`，用于提供可复用记录或生成实体。
- **L92**: Declares a function, method, or callable interface without defining it here. / 声明一个函数、方法或可调用接口，但不在此处给出定义。
- **L93**: Switches subsequent class members to `public` access. / 将后续类成员切换到 `public` 访问级别。
- **L94**: Declares a function, method, or callable interface without defining it here. / 声明一个函数、方法或可调用接口，但不在此处给出定义。
- **L95**: Closes the current declaration or scoped aggregate. / 结束当前声明或带作用域的聚合结构。
- **L96**: Blank line separates nearby declarations or logical regions. / 空行用于分隔附近的声明或逻辑区域。

### Lines 97-112 / 第 97-112 行

~~~~cpp
class EmitBCAction : public CodeGenAction {
  virtual void anchor();
public:
  EmitBCAction(llvm::LLVMContext *_VMContext = nullptr);
};

class EmitLLVMAction : public CodeGenAction {
  virtual void anchor();
public:
  EmitLLVMAction(llvm::LLVMContext *_VMContext = nullptr);
};

class EmitLLVMOnlyAction : public CodeGenAction {
  virtual void anchor();
public:
  EmitLLVMOnlyAction(llvm::LLVMContext *_VMContext = nullptr);
~~~~

- **L97**: Declares TableGen class `EmitBCAction`, which contributes reusable records or generated entities. / 声明 TableGen class `EmitBCAction`，用于提供可复用记录或生成实体。
- **L98**: Declares a function, method, or callable interface without defining it here. / 声明一个函数、方法或可调用接口，但不在此处给出定义。
- **L99**: Switches subsequent class members to `public` access. / 将后续类成员切换到 `public` 访问级别。
- **L100**: Declares a function, method, or callable interface without defining it here. / 声明一个函数、方法或可调用接口，但不在此处给出定义。
- **L101**: Closes the current declaration or scoped aggregate. / 结束当前声明或带作用域的聚合结构。
- **L102**: Blank line separates nearby declarations or logical regions. / 空行用于分隔附近的声明或逻辑区域。
- **L103**: Declares TableGen class `EmitLLVMAction`, which contributes reusable records or generated entities. / 声明 TableGen class `EmitLLVMAction`，用于提供可复用记录或生成实体。
- **L104**: Declares a function, method, or callable interface without defining it here. / 声明一个函数、方法或可调用接口，但不在此处给出定义。
- **L105**: Switches subsequent class members to `public` access. / 将后续类成员切换到 `public` 访问级别。
- **L106**: Declares a function, method, or callable interface without defining it here. / 声明一个函数、方法或可调用接口，但不在此处给出定义。
- **L107**: Closes the current declaration or scoped aggregate. / 结束当前声明或带作用域的聚合结构。
- **L108**: Blank line separates nearby declarations or logical regions. / 空行用于分隔附近的声明或逻辑区域。
- **L109**: Declares TableGen class `EmitLLVMOnlyAction`, which contributes reusable records or generated entities. / 声明 TableGen class `EmitLLVMOnlyAction`，用于提供可复用记录或生成实体。
- **L110**: Declares a function, method, or callable interface without defining it here. / 声明一个函数、方法或可调用接口，但不在此处给出定义。
- **L111**: Switches subsequent class members to `public` access. / 将后续类成员切换到 `public` 访问级别。
- **L112**: Declares a function, method, or callable interface without defining it here. / 声明一个函数、方法或可调用接口，但不在此处给出定义。

### Lines 113-128 / 第 113-128 行

~~~~cpp
};

class EmitCodeGenOnlyAction : public CodeGenAction {
  virtual void anchor();
public:
  EmitCodeGenOnlyAction(llvm::LLVMContext *_VMContext = nullptr);
};

class EmitObjAction : public CodeGenAction {
  virtual void anchor();
public:
  EmitObjAction(llvm::LLVMContext *_VMContext = nullptr);
};

}

~~~~

- **L113**: Closes the current declaration or scoped aggregate. / 结束当前声明或带作用域的聚合结构。
- **L114**: Blank line separates nearby declarations or logical regions. / 空行用于分隔附近的声明或逻辑区域。
- **L115**: Declares TableGen class `EmitCodeGenOnlyAction`, which contributes reusable records or generated entities. / 声明 TableGen class `EmitCodeGenOnlyAction`，用于提供可复用记录或生成实体。
- **L116**: Declares a function, method, or callable interface without defining it here. / 声明一个函数、方法或可调用接口，但不在此处给出定义。
- **L117**: Switches subsequent class members to `public` access. / 将后续类成员切换到 `public` 访问级别。
- **L118**: Declares a function, method, or callable interface without defining it here. / 声明一个函数、方法或可调用接口，但不在此处给出定义。
- **L119**: Closes the current declaration or scoped aggregate. / 结束当前声明或带作用域的聚合结构。
- **L120**: Blank line separates nearby declarations or logical regions. / 空行用于分隔附近的声明或逻辑区域。
- **L121**: Declares TableGen class `EmitObjAction`, which contributes reusable records or generated entities. / 声明 TableGen class `EmitObjAction`，用于提供可复用记录或生成实体。
- **L122**: Declares a function, method, or callable interface without defining it here. / 声明一个函数、方法或可调用接口，但不在此处给出定义。
- **L123**: Switches subsequent class members to `public` access. / 将后续类成员切换到 `public` 访问级别。
- **L124**: Declares a function, method, or callable interface without defining it here. / 声明一个函数、方法或可调用接口，但不在此处给出定义。
- **L125**: Closes the current declaration or scoped aggregate. / 结束当前声明或带作用域的聚合结构。
- **L126**: Blank line separates nearby declarations or logical regions. / 空行用于分隔附近的声明或逻辑区域。
- **L127**: Closes the current scope or body. / 关闭当前作用域或主体。
- **L128**: Blank line separates nearby declarations or logical regions. / 空行用于分隔附近的声明或逻辑区域。

### Line 129 / 第 129 行

~~~~cpp
#endif
~~~~

- **L129**: Ends the active preprocessor conditional region. / 结束当前的预处理条件区域。

## Key Concepts / 关键概念

- **File role / 文件角色**: This file is a header/interface file in Clang's **CodeGen** area. / 该文件是 Clang **CodeGen** 领域中的头文件/接口文件。
- **Scale / 规模**: The source contains 129 lines and 2 directly referenced includes. / 源文件共 129 行，直接引用了 2 个包含项。
- **Subsystem focus / 子系统重点**: IR emission, ABI-sensitive lowering, target-specific hooks. / IR 发射、ABI 相关降级、目标相关钩子。
- **Primary types/records / 主要类型或记录**: `LLVMContext`, `Module`, `BackendConsumer`, `CodeGenerator`, `CodeGenAction`, `LinkModule`, `EmitAssemblyAction`, `EmitBCAction`, `EmitLLVMAction`, `EmitLLVMOnlyAction`. / 主要类型或记录包括 `LLVMContext`, `Module`, `BackendConsumer`, `CodeGenerator`, `CodeGenAction`, `LinkModule`, `EmitAssemblyAction`, `EmitBCAction`, `EmitLLVMAction`, `EmitLLVMOnlyAction`。
- **Visible routines / 可见例程**: `loadModule`, `loadLinkModules`, `CodeGenAction`, `takeModule`, `takeLLVMContext`, `getCodeGenerator`, `anchor`, `EmitAssemblyAction`, `EmitBCAction`, `EmitLLVMAction`. / 可见的关键例程包括 `loadModule`, `loadLinkModules`, `CodeGenAction`, `takeModule`, `takeLLVMContext`, `getCodeGenerator`, `anchor`, `EmitAssemblyAction`, `EmitBCAction`, `EmitLLVMAction`。
- **Macros / 宏**: `LLVM_CLANG_CODEGEN_CODEGENACTION_H`. / 该文件中的宏包括 `LLVM_CLANG_CODEGEN_CODEGENACTION_H`。
- **Namespaces / 命名空间**: `llvm`, `clang`. / 涉及的命名空间包括 `llvm`, `clang`。

## Dependencies / 依赖关系

- **Clang headers / Clang 头文件**: `clang/Frontend/FrontendAction.h`.
- **System/other includes / 系统或其他包含项**: `memory`.
- **Core declarations / 核心声明**: `LLVMContext`, `Module`, `BackendConsumer`, `CodeGenerator`, `CodeGenAction`, `LinkModule`, `EmitAssemblyAction`, `EmitBCAction`, `EmitLLVMAction`, `EmitLLVMOnlyAction`.
- **Callable interfaces / 可调用接口**: `loadModule`, `loadLinkModules`, `CodeGenAction`, `takeModule`, `takeLLVMContext`, `getCodeGenerator`, `anchor`, `EmitAssemblyAction`, `EmitBCAction`, `EmitLLVMAction`.
- **Macro dependencies / 宏依赖**: `LLVM_CLANG_CODEGEN_CODEGENACTION_H`.
- **Namespaces / 命名空间**: `llvm`, `clang`.
