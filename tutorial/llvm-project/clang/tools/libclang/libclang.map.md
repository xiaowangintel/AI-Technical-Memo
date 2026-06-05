# libclang.map — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `clang/tools/libclang/libclang.map`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: If you add a symbol to this file, make sure to add it with the correct version.  For example, if the LLVM main branch is LLVM 14.0.0, add new symbols with the version LLVM_14. On platforms where versions scripts are not used, this file will be used to
  - **CN**: 实现 libclang 的 C 接口、索引支持以及翻译单元服务。

## Line-by-Line Analysis / 逐行分析

### Lines 1-18

````text
# If you add a symbol to this file, make sure to add it with the correct
# version.  For example, if the LLVM main branch is LLVM 14.0.0, add new
# symbols with the version LLVM_14.
# On platforms where versions scripts are not used, this file will be used to
# generate a list of exports for libclang.so

LLVM_13 {
  global:
    clang_BlockCommandComment_getArgText;
    clang_BlockCommandComment_getCommandName;
    clang_BlockCommandComment_getNumArgs;
    clang_BlockCommandComment_getParagraph;
    clang_CXCursorSet_contains;
    clang_CXCursorSet_insert;
    clang_CXIndex_getGlobalOptions;
    clang_CXIndex_setGlobalOptions;
    clang_CXIndex_setInvocationEmissionPathOption;
    clang_CXRewriter_create;
````
- **L1 EN**: Defines a heading or emphasized marker: `# If you add a symbol to this file, make sure to add it with the correct`.
  **L1 CN**: 定义一个标题或强调标记：`# If you add a symbol to this file, make sure to add it with the correct`。
- **L2 EN**: Defines a heading or emphasized marker: `# version. For example, if the LLVM main branch is LLVM 14.0.0, add new`.
  **L2 CN**: 定义一个标题或强调标记：`# version. For example, if the LLVM main branch is LLVM 14.0.0, add new`。
- **L3 EN**: Defines a heading or emphasized marker: `# symbols with the version LLVM_14.`.
  **L3 CN**: 定义一个标题或强调标记：`# symbols with the version LLVM_14.`。
- **L4 EN**: Defines a heading or emphasized marker: `# On platforms where versions scripts are not used, this file will be used to`.
  **L4 CN**: 定义一个标题或强调标记：`# On platforms where versions scripts are not used, this file will be used to`。
- **L5 EN**: Defines a heading or emphasized marker: `# generate a list of exports for libclang.so`.
  **L5 CN**: 定义一个标题或强调标记：`# generate a list of exports for libclang.so`。
- **L6 EN**: Blank line separating nearby declarations or logic blocks.
  **L6 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L7 EN**: Provides textual content or support data: `LLVM_13 {`.
  **L7 CN**: 提供文本内容或支持数据：`LLVM_13 {`。
- **L8 EN**: Introduces a labeled text section: `global:`.
  **L8 CN**: 引入一个带标签的文本段落：`global:`。
- **L9 EN**: Provides textual content or support data: `clang_BlockCommandComment_getArgText;`.
  **L9 CN**: 提供文本内容或支持数据：`clang_BlockCommandComment_getArgText;`。
- **L10 EN**: Provides textual content or support data: `clang_BlockCommandComment_getCommandName;`.
  **L10 CN**: 提供文本内容或支持数据：`clang_BlockCommandComment_getCommandName;`。
- **L11 EN**: Provides textual content or support data: `clang_BlockCommandComment_getNumArgs;`.
  **L11 CN**: 提供文本内容或支持数据：`clang_BlockCommandComment_getNumArgs;`。
- **L12 EN**: Provides textual content or support data: `clang_BlockCommandComment_getParagraph;`.
  **L12 CN**: 提供文本内容或支持数据：`clang_BlockCommandComment_getParagraph;`。
- **L13 EN**: Provides textual content or support data: `clang_CXCursorSet_contains;`.
  **L13 CN**: 提供文本内容或支持数据：`clang_CXCursorSet_contains;`。
- **L14 EN**: Provides textual content or support data: `clang_CXCursorSet_insert;`.
  **L14 CN**: 提供文本内容或支持数据：`clang_CXCursorSet_insert;`。
- **L15 EN**: Provides textual content or support data: `clang_CXIndex_getGlobalOptions;`.
  **L15 CN**: 提供文本内容或支持数据：`clang_CXIndex_getGlobalOptions;`。
- **L16 EN**: Provides textual content or support data: `clang_CXIndex_setGlobalOptions;`.
  **L16 CN**: 提供文本内容或支持数据：`clang_CXIndex_setGlobalOptions;`。
- **L17 EN**: Provides textual content or support data: `clang_CXIndex_setInvocationEmissionPathOption;`.
  **L17 CN**: 提供文本内容或支持数据：`clang_CXIndex_setInvocationEmissionPathOption;`。
- **L18 EN**: Provides textual content or support data: `clang_CXRewriter_create;`.
  **L18 CN**: 提供文本内容或支持数据：`clang_CXRewriter_create;`。

### Lines 19-36

````text
    clang_CXRewriter_dispose;
    clang_CXRewriter_insertTextBefore;
    clang_CXRewriter_overwriteChangedFiles;
    clang_CXRewriter_removeText;
    clang_CXRewriter_replaceText;
    clang_CXRewriter_writeMainFileToStdOut;
    clang_CXXConstructor_isConvertingConstructor;
    clang_CXXConstructor_isCopyConstructor;
    clang_CXXConstructor_isDefaultConstructor;
    clang_CXXConstructor_isMoveConstructor;
    clang_CXXField_isMutable;
    clang_CXXMethod_isConst;
    clang_CXXMethod_isDefaulted;
    clang_CXXMethod_isPureVirtual;
    clang_CXXMethod_isStatic;
    clang_CXXMethod_isVirtual;
    clang_CXXRecord_isAbstract;
    clang_Comment_getChild;
````
- **L19 EN**: Provides textual content or support data: `clang_CXRewriter_dispose;`.
  **L19 CN**: 提供文本内容或支持数据：`clang_CXRewriter_dispose;`。
- **L20 EN**: Provides textual content or support data: `clang_CXRewriter_insertTextBefore;`.
  **L20 CN**: 提供文本内容或支持数据：`clang_CXRewriter_insertTextBefore;`。
- **L21 EN**: Provides textual content or support data: `clang_CXRewriter_overwriteChangedFiles;`.
  **L21 CN**: 提供文本内容或支持数据：`clang_CXRewriter_overwriteChangedFiles;`。
- **L22 EN**: Provides textual content or support data: `clang_CXRewriter_removeText;`.
  **L22 CN**: 提供文本内容或支持数据：`clang_CXRewriter_removeText;`。
- **L23 EN**: Provides textual content or support data: `clang_CXRewriter_replaceText;`.
  **L23 CN**: 提供文本内容或支持数据：`clang_CXRewriter_replaceText;`。
- **L24 EN**: Provides textual content or support data: `clang_CXRewriter_writeMainFileToStdOut;`.
  **L24 CN**: 提供文本内容或支持数据：`clang_CXRewriter_writeMainFileToStdOut;`。
- **L25 EN**: Provides textual content or support data: `clang_CXXConstructor_isConvertingConstructor;`.
  **L25 CN**: 提供文本内容或支持数据：`clang_CXXConstructor_isConvertingConstructor;`。
- **L26 EN**: Provides textual content or support data: `clang_CXXConstructor_isCopyConstructor;`.
  **L26 CN**: 提供文本内容或支持数据：`clang_CXXConstructor_isCopyConstructor;`。
- **L27 EN**: Provides textual content or support data: `clang_CXXConstructor_isDefaultConstructor;`.
  **L27 CN**: 提供文本内容或支持数据：`clang_CXXConstructor_isDefaultConstructor;`。
- **L28 EN**: Provides textual content or support data: `clang_CXXConstructor_isMoveConstructor;`.
  **L28 CN**: 提供文本内容或支持数据：`clang_CXXConstructor_isMoveConstructor;`。
- **L29 EN**: Provides textual content or support data: `clang_CXXField_isMutable;`.
  **L29 CN**: 提供文本内容或支持数据：`clang_CXXField_isMutable;`。
- **L30 EN**: Provides textual content or support data: `clang_CXXMethod_isConst;`.
  **L30 CN**: 提供文本内容或支持数据：`clang_CXXMethod_isConst;`。
- **L31 EN**: Provides textual content or support data: `clang_CXXMethod_isDefaulted;`.
  **L31 CN**: 提供文本内容或支持数据：`clang_CXXMethod_isDefaulted;`。
- **L32 EN**: Provides textual content or support data: `clang_CXXMethod_isPureVirtual;`.
  **L32 CN**: 提供文本内容或支持数据：`clang_CXXMethod_isPureVirtual;`。
- **L33 EN**: Provides textual content or support data: `clang_CXXMethod_isStatic;`.
  **L33 CN**: 提供文本内容或支持数据：`clang_CXXMethod_isStatic;`。
- **L34 EN**: Provides textual content or support data: `clang_CXXMethod_isVirtual;`.
  **L34 CN**: 提供文本内容或支持数据：`clang_CXXMethod_isVirtual;`。
- **L35 EN**: Provides textual content or support data: `clang_CXXRecord_isAbstract;`.
  **L35 CN**: 提供文本内容或支持数据：`clang_CXXRecord_isAbstract;`。
- **L36 EN**: Provides textual content or support data: `clang_Comment_getChild;`.
  **L36 CN**: 提供文本内容或支持数据：`clang_Comment_getChild;`。

### Lines 37-54

````text
    clang_Comment_getKind;
    clang_Comment_getNumChildren;
    clang_Comment_isWhitespace;
    clang_CompilationDatabase_dispose;
    clang_CompilationDatabase_fromDirectory;
    clang_CompilationDatabase_getAllCompileCommands;
    clang_CompilationDatabase_getCompileCommands;
    clang_CompileCommand_getArg;
    clang_CompileCommand_getDirectory;
    clang_CompileCommand_getFilename;
    clang_CompileCommand_getMappedSourceContent;
    clang_CompileCommand_getMappedSourcePath;
    clang_CompileCommand_getNumArgs;
    clang_CompileCommand_getNumMappedSources;
    clang_CompileCommands_dispose;
    clang_CompileCommands_getCommand;
    clang_CompileCommands_getSize;
    clang_Cursor_Evaluate;
````
- **L37 EN**: Provides textual content or support data: `clang_Comment_getKind;`.
  **L37 CN**: 提供文本内容或支持数据：`clang_Comment_getKind;`。
- **L38 EN**: Provides textual content or support data: `clang_Comment_getNumChildren;`.
  **L38 CN**: 提供文本内容或支持数据：`clang_Comment_getNumChildren;`。
- **L39 EN**: Provides textual content or support data: `clang_Comment_isWhitespace;`.
  **L39 CN**: 提供文本内容或支持数据：`clang_Comment_isWhitespace;`。
- **L40 EN**: Provides textual content or support data: `clang_CompilationDatabase_dispose;`.
  **L40 CN**: 提供文本内容或支持数据：`clang_CompilationDatabase_dispose;`。
- **L41 EN**: Provides textual content or support data: `clang_CompilationDatabase_fromDirectory;`.
  **L41 CN**: 提供文本内容或支持数据：`clang_CompilationDatabase_fromDirectory;`。
- **L42 EN**: Provides textual content or support data: `clang_CompilationDatabase_getAllCompileCommands;`.
  **L42 CN**: 提供文本内容或支持数据：`clang_CompilationDatabase_getAllCompileCommands;`。
- **L43 EN**: Provides textual content or support data: `clang_CompilationDatabase_getCompileCommands;`.
  **L43 CN**: 提供文本内容或支持数据：`clang_CompilationDatabase_getCompileCommands;`。
- **L44 EN**: Provides textual content or support data: `clang_CompileCommand_getArg;`.
  **L44 CN**: 提供文本内容或支持数据：`clang_CompileCommand_getArg;`。
- **L45 EN**: Provides textual content or support data: `clang_CompileCommand_getDirectory;`.
  **L45 CN**: 提供文本内容或支持数据：`clang_CompileCommand_getDirectory;`。
- **L46 EN**: Provides textual content or support data: `clang_CompileCommand_getFilename;`.
  **L46 CN**: 提供文本内容或支持数据：`clang_CompileCommand_getFilename;`。
- **L47 EN**: Provides textual content or support data: `clang_CompileCommand_getMappedSourceContent;`.
  **L47 CN**: 提供文本内容或支持数据：`clang_CompileCommand_getMappedSourceContent;`。
- **L48 EN**: Provides textual content or support data: `clang_CompileCommand_getMappedSourcePath;`.
  **L48 CN**: 提供文本内容或支持数据：`clang_CompileCommand_getMappedSourcePath;`。
- **L49 EN**: Provides textual content or support data: `clang_CompileCommand_getNumArgs;`.
  **L49 CN**: 提供文本内容或支持数据：`clang_CompileCommand_getNumArgs;`。
- **L50 EN**: Provides textual content or support data: `clang_CompileCommand_getNumMappedSources;`.
  **L50 CN**: 提供文本内容或支持数据：`clang_CompileCommand_getNumMappedSources;`。
- **L51 EN**: Provides textual content or support data: `clang_CompileCommands_dispose;`.
  **L51 CN**: 提供文本内容或支持数据：`clang_CompileCommands_dispose;`。
- **L52 EN**: Provides textual content or support data: `clang_CompileCommands_getCommand;`.
  **L52 CN**: 提供文本内容或支持数据：`clang_CompileCommands_getCommand;`。
- **L53 EN**: Provides textual content or support data: `clang_CompileCommands_getSize;`.
  **L53 CN**: 提供文本内容或支持数据：`clang_CompileCommands_getSize;`。
- **L54 EN**: Provides textual content or support data: `clang_Cursor_Evaluate;`.
  **L54 CN**: 提供文本内容或支持数据：`clang_Cursor_Evaluate;`。

### Lines 55-72

````text
    clang_Cursor_getArgument;
    clang_Cursor_getBriefCommentText;
    clang_Cursor_getCXXManglings;
    clang_Cursor_getCommentRange;
    clang_Cursor_getMangling;
    clang_Cursor_getModule;
    clang_Cursor_getNumArguments;
    clang_Cursor_getNumTemplateArguments;
    clang_Cursor_getObjCDeclQualifiers;
    clang_Cursor_getObjCManglings;
    clang_Cursor_getObjCPropertyAttributes;
    clang_Cursor_getObjCPropertyGetterName;
    clang_Cursor_getObjCPropertySetterName;
    clang_Cursor_getObjCSelectorIndex;
    clang_Cursor_getOffsetOfField;
    clang_Cursor_getParsedComment;
    clang_Cursor_getRawCommentText;
    clang_Cursor_getReceiverType;
````
- **L55 EN**: Provides textual content or support data: `clang_Cursor_getArgument;`.
  **L55 CN**: 提供文本内容或支持数据：`clang_Cursor_getArgument;`。
- **L56 EN**: Provides textual content or support data: `clang_Cursor_getBriefCommentText;`.
  **L56 CN**: 提供文本内容或支持数据：`clang_Cursor_getBriefCommentText;`。
- **L57 EN**: Provides textual content or support data: `clang_Cursor_getCXXManglings;`.
  **L57 CN**: 提供文本内容或支持数据：`clang_Cursor_getCXXManglings;`。
- **L58 EN**: Provides textual content or support data: `clang_Cursor_getCommentRange;`.
  **L58 CN**: 提供文本内容或支持数据：`clang_Cursor_getCommentRange;`。
- **L59 EN**: Provides textual content or support data: `clang_Cursor_getMangling;`.
  **L59 CN**: 提供文本内容或支持数据：`clang_Cursor_getMangling;`。
- **L60 EN**: Provides textual content or support data: `clang_Cursor_getModule;`.
  **L60 CN**: 提供文本内容或支持数据：`clang_Cursor_getModule;`。
- **L61 EN**: Provides textual content or support data: `clang_Cursor_getNumArguments;`.
  **L61 CN**: 提供文本内容或支持数据：`clang_Cursor_getNumArguments;`。
- **L62 EN**: Provides textual content or support data: `clang_Cursor_getNumTemplateArguments;`.
  **L62 CN**: 提供文本内容或支持数据：`clang_Cursor_getNumTemplateArguments;`。
- **L63 EN**: Provides textual content or support data: `clang_Cursor_getObjCDeclQualifiers;`.
  **L63 CN**: 提供文本内容或支持数据：`clang_Cursor_getObjCDeclQualifiers;`。
- **L64 EN**: Provides textual content or support data: `clang_Cursor_getObjCManglings;`.
  **L64 CN**: 提供文本内容或支持数据：`clang_Cursor_getObjCManglings;`。
- **L65 EN**: Provides textual content or support data: `clang_Cursor_getObjCPropertyAttributes;`.
  **L65 CN**: 提供文本内容或支持数据：`clang_Cursor_getObjCPropertyAttributes;`。
- **L66 EN**: Provides textual content or support data: `clang_Cursor_getObjCPropertyGetterName;`.
  **L66 CN**: 提供文本内容或支持数据：`clang_Cursor_getObjCPropertyGetterName;`。
- **L67 EN**: Provides textual content or support data: `clang_Cursor_getObjCPropertySetterName;`.
  **L67 CN**: 提供文本内容或支持数据：`clang_Cursor_getObjCPropertySetterName;`。
- **L68 EN**: Provides textual content or support data: `clang_Cursor_getObjCSelectorIndex;`.
  **L68 CN**: 提供文本内容或支持数据：`clang_Cursor_getObjCSelectorIndex;`。
- **L69 EN**: Provides textual content or support data: `clang_Cursor_getOffsetOfField;`.
  **L69 CN**: 提供文本内容或支持数据：`clang_Cursor_getOffsetOfField;`。
- **L70 EN**: Provides textual content or support data: `clang_Cursor_getParsedComment;`.
  **L70 CN**: 提供文本内容或支持数据：`clang_Cursor_getParsedComment;`。
- **L71 EN**: Provides textual content or support data: `clang_Cursor_getRawCommentText;`.
  **L71 CN**: 提供文本内容或支持数据：`clang_Cursor_getRawCommentText;`。
- **L72 EN**: Provides textual content or support data: `clang_Cursor_getReceiverType;`.
  **L72 CN**: 提供文本内容或支持数据：`clang_Cursor_getReceiverType;`。

### Lines 73-90

````text
    clang_Cursor_getSpellingNameRange;
    clang_Cursor_getStorageClass;
    clang_Cursor_getTemplateArgumentKind;
    clang_Cursor_getTemplateArgumentType;
    clang_Cursor_getTemplateArgumentUnsignedValue;
    clang_Cursor_getTemplateArgumentValue;
    clang_Cursor_getTranslationUnit;
    clang_Cursor_getVarDeclInitializer;
    clang_Cursor_hasAttrs;
    clang_Cursor_hasVarDeclExternalStorage;
    clang_Cursor_hasVarDeclGlobalStorage;
    clang_Cursor_isAnonymous;
    clang_Cursor_isAnonymousRecordDecl;
    clang_Cursor_isBitField;
    clang_Cursor_isDynamicCall;
    clang_Cursor_isExternalSymbol;
    clang_Cursor_isFunctionInlined;
    clang_Cursor_isInlineNamespace;
````
- **L73 EN**: Provides textual content or support data: `clang_Cursor_getSpellingNameRange;`.
  **L73 CN**: 提供文本内容或支持数据：`clang_Cursor_getSpellingNameRange;`。
- **L74 EN**: Provides textual content or support data: `clang_Cursor_getStorageClass;`.
  **L74 CN**: 提供文本内容或支持数据：`clang_Cursor_getStorageClass;`。
- **L75 EN**: Provides textual content or support data: `clang_Cursor_getTemplateArgumentKind;`.
  **L75 CN**: 提供文本内容或支持数据：`clang_Cursor_getTemplateArgumentKind;`。
- **L76 EN**: Provides textual content or support data: `clang_Cursor_getTemplateArgumentType;`.
  **L76 CN**: 提供文本内容或支持数据：`clang_Cursor_getTemplateArgumentType;`。
- **L77 EN**: Provides textual content or support data: `clang_Cursor_getTemplateArgumentUnsignedValue;`.
  **L77 CN**: 提供文本内容或支持数据：`clang_Cursor_getTemplateArgumentUnsignedValue;`。
- **L78 EN**: Provides textual content or support data: `clang_Cursor_getTemplateArgumentValue;`.
  **L78 CN**: 提供文本内容或支持数据：`clang_Cursor_getTemplateArgumentValue;`。
- **L79 EN**: Provides textual content or support data: `clang_Cursor_getTranslationUnit;`.
  **L79 CN**: 提供文本内容或支持数据：`clang_Cursor_getTranslationUnit;`。
- **L80 EN**: Provides textual content or support data: `clang_Cursor_getVarDeclInitializer;`.
  **L80 CN**: 提供文本内容或支持数据：`clang_Cursor_getVarDeclInitializer;`。
- **L81 EN**: Provides textual content or support data: `clang_Cursor_hasAttrs;`.
  **L81 CN**: 提供文本内容或支持数据：`clang_Cursor_hasAttrs;`。
- **L82 EN**: Provides textual content or support data: `clang_Cursor_hasVarDeclExternalStorage;`.
  **L82 CN**: 提供文本内容或支持数据：`clang_Cursor_hasVarDeclExternalStorage;`。
- **L83 EN**: Provides textual content or support data: `clang_Cursor_hasVarDeclGlobalStorage;`.
  **L83 CN**: 提供文本内容或支持数据：`clang_Cursor_hasVarDeclGlobalStorage;`。
- **L84 EN**: Provides textual content or support data: `clang_Cursor_isAnonymous;`.
  **L84 CN**: 提供文本内容或支持数据：`clang_Cursor_isAnonymous;`。
- **L85 EN**: Provides textual content or support data: `clang_Cursor_isAnonymousRecordDecl;`.
  **L85 CN**: 提供文本内容或支持数据：`clang_Cursor_isAnonymousRecordDecl;`。
- **L86 EN**: Provides textual content or support data: `clang_Cursor_isBitField;`.
  **L86 CN**: 提供文本内容或支持数据：`clang_Cursor_isBitField;`。
- **L87 EN**: Provides textual content or support data: `clang_Cursor_isDynamicCall;`.
  **L87 CN**: 提供文本内容或支持数据：`clang_Cursor_isDynamicCall;`。
- **L88 EN**: Provides textual content or support data: `clang_Cursor_isExternalSymbol;`.
  **L88 CN**: 提供文本内容或支持数据：`clang_Cursor_isExternalSymbol;`。
- **L89 EN**: Provides textual content or support data: `clang_Cursor_isFunctionInlined;`.
  **L89 CN**: 提供文本内容或支持数据：`clang_Cursor_isFunctionInlined;`。
- **L90 EN**: Provides textual content or support data: `clang_Cursor_isInlineNamespace;`.
  **L90 CN**: 提供文本内容或支持数据：`clang_Cursor_isInlineNamespace;`。

### Lines 91-108

````text
    clang_Cursor_isMacroBuiltin;
    clang_Cursor_isMacroFunctionLike;
    clang_Cursor_isNull;
    clang_Cursor_isObjCOptional;
    clang_Cursor_isVariadic;
    clang_EnumDecl_isScoped;
    clang_EvalResult_dispose;
    clang_EvalResult_getAsDouble;
    clang_EvalResult_getAsInt;
    clang_EvalResult_getAsLongLong;
    clang_EvalResult_getAsStr;
    clang_EvalResult_getAsUnsigned;
    clang_EvalResult_getKind;
    clang_EvalResult_isUnsignedInt;
    clang_File_isEqual;
    clang_File_tryGetRealPathName;
    clang_FullComment_getAsHTML;
    clang_FullComment_getAsXML;
````
- **L91 EN**: Provides textual content or support data: `clang_Cursor_isMacroBuiltin;`.
  **L91 CN**: 提供文本内容或支持数据：`clang_Cursor_isMacroBuiltin;`。
- **L92 EN**: Provides textual content or support data: `clang_Cursor_isMacroFunctionLike;`.
  **L92 CN**: 提供文本内容或支持数据：`clang_Cursor_isMacroFunctionLike;`。
- **L93 EN**: Provides textual content or support data: `clang_Cursor_isNull;`.
  **L93 CN**: 提供文本内容或支持数据：`clang_Cursor_isNull;`。
- **L94 EN**: Provides textual content or support data: `clang_Cursor_isObjCOptional;`.
  **L94 CN**: 提供文本内容或支持数据：`clang_Cursor_isObjCOptional;`。
- **L95 EN**: Provides textual content or support data: `clang_Cursor_isVariadic;`.
  **L95 CN**: 提供文本内容或支持数据：`clang_Cursor_isVariadic;`。
- **L96 EN**: Provides textual content or support data: `clang_EnumDecl_isScoped;`.
  **L96 CN**: 提供文本内容或支持数据：`clang_EnumDecl_isScoped;`。
- **L97 EN**: Provides textual content or support data: `clang_EvalResult_dispose;`.
  **L97 CN**: 提供文本内容或支持数据：`clang_EvalResult_dispose;`。
- **L98 EN**: Provides textual content or support data: `clang_EvalResult_getAsDouble;`.
  **L98 CN**: 提供文本内容或支持数据：`clang_EvalResult_getAsDouble;`。
- **L99 EN**: Provides textual content or support data: `clang_EvalResult_getAsInt;`.
  **L99 CN**: 提供文本内容或支持数据：`clang_EvalResult_getAsInt;`。
- **L100 EN**: Provides textual content or support data: `clang_EvalResult_getAsLongLong;`.
  **L100 CN**: 提供文本内容或支持数据：`clang_EvalResult_getAsLongLong;`。
- **L101 EN**: Provides textual content or support data: `clang_EvalResult_getAsStr;`.
  **L101 CN**: 提供文本内容或支持数据：`clang_EvalResult_getAsStr;`。
- **L102 EN**: Provides textual content or support data: `clang_EvalResult_getAsUnsigned;`.
  **L102 CN**: 提供文本内容或支持数据：`clang_EvalResult_getAsUnsigned;`。
- **L103 EN**: Provides textual content or support data: `clang_EvalResult_getKind;`.
  **L103 CN**: 提供文本内容或支持数据：`clang_EvalResult_getKind;`。
- **L104 EN**: Provides textual content or support data: `clang_EvalResult_isUnsignedInt;`.
  **L104 CN**: 提供文本内容或支持数据：`clang_EvalResult_isUnsignedInt;`。
- **L105 EN**: Provides textual content or support data: `clang_File_isEqual;`.
  **L105 CN**: 提供文本内容或支持数据：`clang_File_isEqual;`。
- **L106 EN**: Provides textual content or support data: `clang_File_tryGetRealPathName;`.
  **L106 CN**: 提供文本内容或支持数据：`clang_File_tryGetRealPathName;`。
- **L107 EN**: Provides textual content or support data: `clang_FullComment_getAsHTML;`.
  **L107 CN**: 提供文本内容或支持数据：`clang_FullComment_getAsHTML;`。
- **L108 EN**: Provides textual content or support data: `clang_FullComment_getAsXML;`.
  **L108 CN**: 提供文本内容或支持数据：`clang_FullComment_getAsXML;`。

### Lines 109-126

````text
    clang_HTMLStartTagComment_isSelfClosing;
    clang_HTMLStartTag_getAttrName;
    clang_HTMLStartTag_getAttrValue;
    clang_HTMLStartTag_getNumAttrs;
    clang_HTMLTagComment_getAsString;
    clang_HTMLTagComment_getTagName;
    clang_IndexAction_create;
    clang_IndexAction_dispose;
    clang_InlineCommandComment_getArgText;
    clang_InlineCommandComment_getCommandName;
    clang_InlineCommandComment_getNumArgs;
    clang_InlineCommandComment_getRenderKind;
    clang_InlineContentComment_hasTrailingNewline;
    clang_Location_isFromMainFile;
    clang_Location_isInSystemHeader;
    clang_ModuleMapDescriptor_create;
    clang_ModuleMapDescriptor_dispose;
    clang_ModuleMapDescriptor_setFrameworkModuleName;
````
- **L109 EN**: Provides textual content or support data: `clang_HTMLStartTagComment_isSelfClosing;`.
  **L109 CN**: 提供文本内容或支持数据：`clang_HTMLStartTagComment_isSelfClosing;`。
- **L110 EN**: Provides textual content or support data: `clang_HTMLStartTag_getAttrName;`.
  **L110 CN**: 提供文本内容或支持数据：`clang_HTMLStartTag_getAttrName;`。
- **L111 EN**: Provides textual content or support data: `clang_HTMLStartTag_getAttrValue;`.
  **L111 CN**: 提供文本内容或支持数据：`clang_HTMLStartTag_getAttrValue;`。
- **L112 EN**: Provides textual content or support data: `clang_HTMLStartTag_getNumAttrs;`.
  **L112 CN**: 提供文本内容或支持数据：`clang_HTMLStartTag_getNumAttrs;`。
- **L113 EN**: Provides textual content or support data: `clang_HTMLTagComment_getAsString;`.
  **L113 CN**: 提供文本内容或支持数据：`clang_HTMLTagComment_getAsString;`。
- **L114 EN**: Provides textual content or support data: `clang_HTMLTagComment_getTagName;`.
  **L114 CN**: 提供文本内容或支持数据：`clang_HTMLTagComment_getTagName;`。
- **L115 EN**: Provides textual content or support data: `clang_IndexAction_create;`.
  **L115 CN**: 提供文本内容或支持数据：`clang_IndexAction_create;`。
- **L116 EN**: Provides textual content or support data: `clang_IndexAction_dispose;`.
  **L116 CN**: 提供文本内容或支持数据：`clang_IndexAction_dispose;`。
- **L117 EN**: Provides textual content or support data: `clang_InlineCommandComment_getArgText;`.
  **L117 CN**: 提供文本内容或支持数据：`clang_InlineCommandComment_getArgText;`。
- **L118 EN**: Provides textual content or support data: `clang_InlineCommandComment_getCommandName;`.
  **L118 CN**: 提供文本内容或支持数据：`clang_InlineCommandComment_getCommandName;`。
- **L119 EN**: Provides textual content or support data: `clang_InlineCommandComment_getNumArgs;`.
  **L119 CN**: 提供文本内容或支持数据：`clang_InlineCommandComment_getNumArgs;`。
- **L120 EN**: Provides textual content or support data: `clang_InlineCommandComment_getRenderKind;`.
  **L120 CN**: 提供文本内容或支持数据：`clang_InlineCommandComment_getRenderKind;`。
- **L121 EN**: Provides textual content or support data: `clang_InlineContentComment_hasTrailingNewline;`.
  **L121 CN**: 提供文本内容或支持数据：`clang_InlineContentComment_hasTrailingNewline;`。
- **L122 EN**: Provides textual content or support data: `clang_Location_isFromMainFile;`.
  **L122 CN**: 提供文本内容或支持数据：`clang_Location_isFromMainFile;`。
- **L123 EN**: Provides textual content or support data: `clang_Location_isInSystemHeader;`.
  **L123 CN**: 提供文本内容或支持数据：`clang_Location_isInSystemHeader;`。
- **L124 EN**: Provides textual content or support data: `clang_ModuleMapDescriptor_create;`.
  **L124 CN**: 提供文本内容或支持数据：`clang_ModuleMapDescriptor_create;`。
- **L125 EN**: Provides textual content or support data: `clang_ModuleMapDescriptor_dispose;`.
  **L125 CN**: 提供文本内容或支持数据：`clang_ModuleMapDescriptor_dispose;`。
- **L126 EN**: Provides textual content or support data: `clang_ModuleMapDescriptor_setFrameworkModuleName;`.
  **L126 CN**: 提供文本内容或支持数据：`clang_ModuleMapDescriptor_setFrameworkModuleName;`。

### Lines 127-144

````text
    clang_ModuleMapDescriptor_setUmbrellaHeader;
    clang_ModuleMapDescriptor_writeToBuffer;
    clang_Module_getASTFile;
    clang_Module_getFullName;
    clang_Module_getName;
    clang_Module_getNumTopLevelHeaders;
    clang_Module_getParent;
    clang_Module_getTopLevelHeader;
    clang_Module_isSystem;
    clang_ParamCommandComment_getDirection;
    clang_ParamCommandComment_getParamIndex;
    clang_ParamCommandComment_getParamName;
    clang_ParamCommandComment_isDirectionExplicit;
    clang_ParamCommandComment_isParamIndexValid;
    clang_PrintingPolicy_dispose;
    clang_PrintingPolicy_getProperty;
    clang_PrintingPolicy_setProperty;
    clang_Range_isNull;
````
- **L127 EN**: Provides textual content or support data: `clang_ModuleMapDescriptor_setUmbrellaHeader;`.
  **L127 CN**: 提供文本内容或支持数据：`clang_ModuleMapDescriptor_setUmbrellaHeader;`。
- **L128 EN**: Provides textual content or support data: `clang_ModuleMapDescriptor_writeToBuffer;`.
  **L128 CN**: 提供文本内容或支持数据：`clang_ModuleMapDescriptor_writeToBuffer;`。
- **L129 EN**: Provides textual content or support data: `clang_Module_getASTFile;`.
  **L129 CN**: 提供文本内容或支持数据：`clang_Module_getASTFile;`。
- **L130 EN**: Provides textual content or support data: `clang_Module_getFullName;`.
  **L130 CN**: 提供文本内容或支持数据：`clang_Module_getFullName;`。
- **L131 EN**: Provides textual content or support data: `clang_Module_getName;`.
  **L131 CN**: 提供文本内容或支持数据：`clang_Module_getName;`。
- **L132 EN**: Provides textual content or support data: `clang_Module_getNumTopLevelHeaders;`.
  **L132 CN**: 提供文本内容或支持数据：`clang_Module_getNumTopLevelHeaders;`。
- **L133 EN**: Provides textual content or support data: `clang_Module_getParent;`.
  **L133 CN**: 提供文本内容或支持数据：`clang_Module_getParent;`。
- **L134 EN**: Provides textual content or support data: `clang_Module_getTopLevelHeader;`.
  **L134 CN**: 提供文本内容或支持数据：`clang_Module_getTopLevelHeader;`。
- **L135 EN**: Provides textual content or support data: `clang_Module_isSystem;`.
  **L135 CN**: 提供文本内容或支持数据：`clang_Module_isSystem;`。
- **L136 EN**: Provides textual content or support data: `clang_ParamCommandComment_getDirection;`.
  **L136 CN**: 提供文本内容或支持数据：`clang_ParamCommandComment_getDirection;`。
- **L137 EN**: Provides textual content or support data: `clang_ParamCommandComment_getParamIndex;`.
  **L137 CN**: 提供文本内容或支持数据：`clang_ParamCommandComment_getParamIndex;`。
- **L138 EN**: Provides textual content or support data: `clang_ParamCommandComment_getParamName;`.
  **L138 CN**: 提供文本内容或支持数据：`clang_ParamCommandComment_getParamName;`。
- **L139 EN**: Provides textual content or support data: `clang_ParamCommandComment_isDirectionExplicit;`.
  **L139 CN**: 提供文本内容或支持数据：`clang_ParamCommandComment_isDirectionExplicit;`。
- **L140 EN**: Provides textual content or support data: `clang_ParamCommandComment_isParamIndexValid;`.
  **L140 CN**: 提供文本内容或支持数据：`clang_ParamCommandComment_isParamIndexValid;`。
- **L141 EN**: Provides textual content or support data: `clang_PrintingPolicy_dispose;`.
  **L141 CN**: 提供文本内容或支持数据：`clang_PrintingPolicy_dispose;`。
- **L142 EN**: Provides textual content or support data: `clang_PrintingPolicy_getProperty;`.
  **L142 CN**: 提供文本内容或支持数据：`clang_PrintingPolicy_getProperty;`。
- **L143 EN**: Provides textual content or support data: `clang_PrintingPolicy_setProperty;`.
  **L143 CN**: 提供文本内容或支持数据：`clang_PrintingPolicy_setProperty;`。
- **L144 EN**: Provides textual content or support data: `clang_Range_isNull;`.
  **L144 CN**: 提供文本内容或支持数据：`clang_Range_isNull;`。

### Lines 145-162

````text
    clang_TParamCommandComment_getDepth;
    clang_TParamCommandComment_getIndex;
    clang_TParamCommandComment_getParamName;
    clang_TParamCommandComment_isParamPositionValid;
    clang_TargetInfo_dispose;
    clang_TargetInfo_getPointerWidth;
    clang_TargetInfo_getTriple;
    clang_TextComment_getText;
    clang_Type_getAlignOf;
    clang_Type_getCXXRefQualifier;
    clang_Type_getClassType;
    clang_Type_getModifiedType;
    clang_Type_getNamedType;
    clang_Type_getNullability;
    clang_Type_getNumObjCProtocolRefs;
    clang_Type_getNumObjCTypeArgs;
    clang_Type_getNumTemplateArguments;
    clang_Type_getObjCEncoding;
````
- **L145 EN**: Provides textual content or support data: `clang_TParamCommandComment_getDepth;`.
  **L145 CN**: 提供文本内容或支持数据：`clang_TParamCommandComment_getDepth;`。
- **L146 EN**: Provides textual content or support data: `clang_TParamCommandComment_getIndex;`.
  **L146 CN**: 提供文本内容或支持数据：`clang_TParamCommandComment_getIndex;`。
- **L147 EN**: Provides textual content or support data: `clang_TParamCommandComment_getParamName;`.
  **L147 CN**: 提供文本内容或支持数据：`clang_TParamCommandComment_getParamName;`。
- **L148 EN**: Provides textual content or support data: `clang_TParamCommandComment_isParamPositionValid;`.
  **L148 CN**: 提供文本内容或支持数据：`clang_TParamCommandComment_isParamPositionValid;`。
- **L149 EN**: Provides textual content or support data: `clang_TargetInfo_dispose;`.
  **L149 CN**: 提供文本内容或支持数据：`clang_TargetInfo_dispose;`。
- **L150 EN**: Provides textual content or support data: `clang_TargetInfo_getPointerWidth;`.
  **L150 CN**: 提供文本内容或支持数据：`clang_TargetInfo_getPointerWidth;`。
- **L151 EN**: Provides textual content or support data: `clang_TargetInfo_getTriple;`.
  **L151 CN**: 提供文本内容或支持数据：`clang_TargetInfo_getTriple;`。
- **L152 EN**: Provides textual content or support data: `clang_TextComment_getText;`.
  **L152 CN**: 提供文本内容或支持数据：`clang_TextComment_getText;`。
- **L153 EN**: Provides textual content or support data: `clang_Type_getAlignOf;`.
  **L153 CN**: 提供文本内容或支持数据：`clang_Type_getAlignOf;`。
- **L154 EN**: Provides textual content or support data: `clang_Type_getCXXRefQualifier;`.
  **L154 CN**: 提供文本内容或支持数据：`clang_Type_getCXXRefQualifier;`。
- **L155 EN**: Provides textual content or support data: `clang_Type_getClassType;`.
  **L155 CN**: 提供文本内容或支持数据：`clang_Type_getClassType;`。
- **L156 EN**: Provides textual content or support data: `clang_Type_getModifiedType;`.
  **L156 CN**: 提供文本内容或支持数据：`clang_Type_getModifiedType;`。
- **L157 EN**: Provides textual content or support data: `clang_Type_getNamedType;`.
  **L157 CN**: 提供文本内容或支持数据：`clang_Type_getNamedType;`。
- **L158 EN**: Provides textual content or support data: `clang_Type_getNullability;`.
  **L158 CN**: 提供文本内容或支持数据：`clang_Type_getNullability;`。
- **L159 EN**: Provides textual content or support data: `clang_Type_getNumObjCProtocolRefs;`.
  **L159 CN**: 提供文本内容或支持数据：`clang_Type_getNumObjCProtocolRefs;`。
- **L160 EN**: Provides textual content or support data: `clang_Type_getNumObjCTypeArgs;`.
  **L160 CN**: 提供文本内容或支持数据：`clang_Type_getNumObjCTypeArgs;`。
- **L161 EN**: Provides textual content or support data: `clang_Type_getNumTemplateArguments;`.
  **L161 CN**: 提供文本内容或支持数据：`clang_Type_getNumTemplateArguments;`。
- **L162 EN**: Provides textual content or support data: `clang_Type_getObjCEncoding;`.
  **L162 CN**: 提供文本内容或支持数据：`clang_Type_getObjCEncoding;`。

### Lines 163-180

````text
    clang_Type_getObjCObjectBaseType;
    clang_Type_getObjCProtocolDecl;
    clang_Type_getObjCTypeArg;
    clang_Type_getOffsetOf;
    clang_Type_getSizeOf;
    clang_Type_getTemplateArgumentAsType;
    clang_Type_getValueType;
    clang_Type_isTransparentTagTypedef;
    clang_Type_visitFields;
    clang_VerbatimBlockLineComment_getText;
    clang_VerbatimLineComment_getText;
    clang_VirtualFileOverlay_addFileMapping;
    clang_VirtualFileOverlay_create;
    clang_VirtualFileOverlay_dispose;
    clang_VirtualFileOverlay_setCaseSensitivity;
    clang_VirtualFileOverlay_writeToBuffer;
    clang_annotateTokens;
    clang_codeCompleteAt;
````
- **L163 EN**: Provides textual content or support data: `clang_Type_getObjCObjectBaseType;`.
  **L163 CN**: 提供文本内容或支持数据：`clang_Type_getObjCObjectBaseType;`。
- **L164 EN**: Provides textual content or support data: `clang_Type_getObjCProtocolDecl;`.
  **L164 CN**: 提供文本内容或支持数据：`clang_Type_getObjCProtocolDecl;`。
- **L165 EN**: Provides textual content or support data: `clang_Type_getObjCTypeArg;`.
  **L165 CN**: 提供文本内容或支持数据：`clang_Type_getObjCTypeArg;`。
- **L166 EN**: Provides textual content or support data: `clang_Type_getOffsetOf;`.
  **L166 CN**: 提供文本内容或支持数据：`clang_Type_getOffsetOf;`。
- **L167 EN**: Provides textual content or support data: `clang_Type_getSizeOf;`.
  **L167 CN**: 提供文本内容或支持数据：`clang_Type_getSizeOf;`。
- **L168 EN**: Provides textual content or support data: `clang_Type_getTemplateArgumentAsType;`.
  **L168 CN**: 提供文本内容或支持数据：`clang_Type_getTemplateArgumentAsType;`。
- **L169 EN**: Provides textual content or support data: `clang_Type_getValueType;`.
  **L169 CN**: 提供文本内容或支持数据：`clang_Type_getValueType;`。
- **L170 EN**: Provides textual content or support data: `clang_Type_isTransparentTagTypedef;`.
  **L170 CN**: 提供文本内容或支持数据：`clang_Type_isTransparentTagTypedef;`。
- **L171 EN**: Provides textual content or support data: `clang_Type_visitFields;`.
  **L171 CN**: 提供文本内容或支持数据：`clang_Type_visitFields;`。
- **L172 EN**: Provides textual content or support data: `clang_VerbatimBlockLineComment_getText;`.
  **L172 CN**: 提供文本内容或支持数据：`clang_VerbatimBlockLineComment_getText;`。
- **L173 EN**: Provides textual content or support data: `clang_VerbatimLineComment_getText;`.
  **L173 CN**: 提供文本内容或支持数据：`clang_VerbatimLineComment_getText;`。
- **L174 EN**: Provides textual content or support data: `clang_VirtualFileOverlay_addFileMapping;`.
  **L174 CN**: 提供文本内容或支持数据：`clang_VirtualFileOverlay_addFileMapping;`。
- **L175 EN**: Provides textual content or support data: `clang_VirtualFileOverlay_create;`.
  **L175 CN**: 提供文本内容或支持数据：`clang_VirtualFileOverlay_create;`。
- **L176 EN**: Provides textual content or support data: `clang_VirtualFileOverlay_dispose;`.
  **L176 CN**: 提供文本内容或支持数据：`clang_VirtualFileOverlay_dispose;`。
- **L177 EN**: Provides textual content or support data: `clang_VirtualFileOverlay_setCaseSensitivity;`.
  **L177 CN**: 提供文本内容或支持数据：`clang_VirtualFileOverlay_setCaseSensitivity;`。
- **L178 EN**: Provides textual content or support data: `clang_VirtualFileOverlay_writeToBuffer;`.
  **L178 CN**: 提供文本内容或支持数据：`clang_VirtualFileOverlay_writeToBuffer;`。
- **L179 EN**: Provides textual content or support data: `clang_annotateTokens;`.
  **L179 CN**: 提供文本内容或支持数据：`clang_annotateTokens;`。
- **L180 EN**: Provides textual content or support data: `clang_codeCompleteAt;`.
  **L180 CN**: 提供文本内容或支持数据：`clang_codeCompleteAt;`。

### Lines 181-198

````text
    clang_codeCompleteGetContainerKind;
    clang_codeCompleteGetContainerUSR;
    clang_codeCompleteGetContexts;
    clang_codeCompleteGetDiagnostic;
    clang_codeCompleteGetNumDiagnostics;
    clang_codeCompleteGetObjCSelector;
    clang_constructUSR_ObjCCategory;
    clang_constructUSR_ObjCClass;
    clang_constructUSR_ObjCIvar;
    clang_constructUSR_ObjCMethod;
    clang_constructUSR_ObjCProperty;
    clang_constructUSR_ObjCProtocol;
    clang_createCXCursorSet;
    clang_createIndex;
    clang_createTranslationUnit;
    clang_createTranslationUnit2;
    clang_createTranslationUnitFromSourceFile;
    clang_defaultCodeCompleteOptions;
````
- **L181 EN**: Provides textual content or support data: `clang_codeCompleteGetContainerKind;`.
  **L181 CN**: 提供文本内容或支持数据：`clang_codeCompleteGetContainerKind;`。
- **L182 EN**: Provides textual content or support data: `clang_codeCompleteGetContainerUSR;`.
  **L182 CN**: 提供文本内容或支持数据：`clang_codeCompleteGetContainerUSR;`。
- **L183 EN**: Provides textual content or support data: `clang_codeCompleteGetContexts;`.
  **L183 CN**: 提供文本内容或支持数据：`clang_codeCompleteGetContexts;`。
- **L184 EN**: Provides textual content or support data: `clang_codeCompleteGetDiagnostic;`.
  **L184 CN**: 提供文本内容或支持数据：`clang_codeCompleteGetDiagnostic;`。
- **L185 EN**: Provides textual content or support data: `clang_codeCompleteGetNumDiagnostics;`.
  **L185 CN**: 提供文本内容或支持数据：`clang_codeCompleteGetNumDiagnostics;`。
- **L186 EN**: Provides textual content or support data: `clang_codeCompleteGetObjCSelector;`.
  **L186 CN**: 提供文本内容或支持数据：`clang_codeCompleteGetObjCSelector;`。
- **L187 EN**: Provides textual content or support data: `clang_constructUSR_ObjCCategory;`.
  **L187 CN**: 提供文本内容或支持数据：`clang_constructUSR_ObjCCategory;`。
- **L188 EN**: Provides textual content or support data: `clang_constructUSR_ObjCClass;`.
  **L188 CN**: 提供文本内容或支持数据：`clang_constructUSR_ObjCClass;`。
- **L189 EN**: Provides textual content or support data: `clang_constructUSR_ObjCIvar;`.
  **L189 CN**: 提供文本内容或支持数据：`clang_constructUSR_ObjCIvar;`。
- **L190 EN**: Provides textual content or support data: `clang_constructUSR_ObjCMethod;`.
  **L190 CN**: 提供文本内容或支持数据：`clang_constructUSR_ObjCMethod;`。
- **L191 EN**: Provides textual content or support data: `clang_constructUSR_ObjCProperty;`.
  **L191 CN**: 提供文本内容或支持数据：`clang_constructUSR_ObjCProperty;`。
- **L192 EN**: Provides textual content or support data: `clang_constructUSR_ObjCProtocol;`.
  **L192 CN**: 提供文本内容或支持数据：`clang_constructUSR_ObjCProtocol;`。
- **L193 EN**: Provides textual content or support data: `clang_createCXCursorSet;`.
  **L193 CN**: 提供文本内容或支持数据：`clang_createCXCursorSet;`。
- **L194 EN**: Provides textual content or support data: `clang_createIndex;`.
  **L194 CN**: 提供文本内容或支持数据：`clang_createIndex;`。
- **L195 EN**: Provides textual content or support data: `clang_createTranslationUnit;`.
  **L195 CN**: 提供文本内容或支持数据：`clang_createTranslationUnit;`。
- **L196 EN**: Provides textual content or support data: `clang_createTranslationUnit2;`.
  **L196 CN**: 提供文本内容或支持数据：`clang_createTranslationUnit2;`。
- **L197 EN**: Provides textual content or support data: `clang_createTranslationUnitFromSourceFile;`.
  **L197 CN**: 提供文本内容或支持数据：`clang_createTranslationUnitFromSourceFile;`。
- **L198 EN**: Provides textual content or support data: `clang_defaultCodeCompleteOptions;`.
  **L198 CN**: 提供文本内容或支持数据：`clang_defaultCodeCompleteOptions;`。

### Lines 199-216

````text
    clang_defaultDiagnosticDisplayOptions;
    clang_defaultEditingTranslationUnitOptions;
    clang_defaultReparseOptions;
    clang_defaultSaveOptions;
    clang_disposeCXCursorSet;
    clang_disposeCXPlatformAvailability;
    clang_disposeCXTUResourceUsage;
    clang_disposeCodeCompleteResults;
    clang_disposeDiagnostic;
    clang_disposeDiagnosticSet;
    clang_disposeIndex;
    clang_disposeOverriddenCursors;
    clang_disposeSourceRangeList;
    clang_disposeString;
    clang_disposeStringSet;
    clang_disposeTokens;
    clang_disposeTranslationUnit;
    clang_enableStackTraces;
````
- **L199 EN**: Provides textual content or support data: `clang_defaultDiagnosticDisplayOptions;`.
  **L199 CN**: 提供文本内容或支持数据：`clang_defaultDiagnosticDisplayOptions;`。
- **L200 EN**: Provides textual content or support data: `clang_defaultEditingTranslationUnitOptions;`.
  **L200 CN**: 提供文本内容或支持数据：`clang_defaultEditingTranslationUnitOptions;`。
- **L201 EN**: Provides textual content or support data: `clang_defaultReparseOptions;`.
  **L201 CN**: 提供文本内容或支持数据：`clang_defaultReparseOptions;`。
- **L202 EN**: Provides textual content or support data: `clang_defaultSaveOptions;`.
  **L202 CN**: 提供文本内容或支持数据：`clang_defaultSaveOptions;`。
- **L203 EN**: Provides textual content or support data: `clang_disposeCXCursorSet;`.
  **L203 CN**: 提供文本内容或支持数据：`clang_disposeCXCursorSet;`。
- **L204 EN**: Provides textual content or support data: `clang_disposeCXPlatformAvailability;`.
  **L204 CN**: 提供文本内容或支持数据：`clang_disposeCXPlatformAvailability;`。
- **L205 EN**: Provides textual content or support data: `clang_disposeCXTUResourceUsage;`.
  **L205 CN**: 提供文本内容或支持数据：`clang_disposeCXTUResourceUsage;`。
- **L206 EN**: Provides textual content or support data: `clang_disposeCodeCompleteResults;`.
  **L206 CN**: 提供文本内容或支持数据：`clang_disposeCodeCompleteResults;`。
- **L207 EN**: Provides textual content or support data: `clang_disposeDiagnostic;`.
  **L207 CN**: 提供文本内容或支持数据：`clang_disposeDiagnostic;`。
- **L208 EN**: Provides textual content or support data: `clang_disposeDiagnosticSet;`.
  **L208 CN**: 提供文本内容或支持数据：`clang_disposeDiagnosticSet;`。
- **L209 EN**: Provides textual content or support data: `clang_disposeIndex;`.
  **L209 CN**: 提供文本内容或支持数据：`clang_disposeIndex;`。
- **L210 EN**: Provides textual content or support data: `clang_disposeOverriddenCursors;`.
  **L210 CN**: 提供文本内容或支持数据：`clang_disposeOverriddenCursors;`。
- **L211 EN**: Provides textual content or support data: `clang_disposeSourceRangeList;`.
  **L211 CN**: 提供文本内容或支持数据：`clang_disposeSourceRangeList;`。
- **L212 EN**: Provides textual content or support data: `clang_disposeString;`.
  **L212 CN**: 提供文本内容或支持数据：`clang_disposeString;`。
- **L213 EN**: Provides textual content or support data: `clang_disposeStringSet;`.
  **L213 CN**: 提供文本内容或支持数据：`clang_disposeStringSet;`。
- **L214 EN**: Provides textual content or support data: `clang_disposeTokens;`.
  **L214 CN**: 提供文本内容或支持数据：`clang_disposeTokens;`。
- **L215 EN**: Provides textual content or support data: `clang_disposeTranslationUnit;`.
  **L215 CN**: 提供文本内容或支持数据：`clang_disposeTranslationUnit;`。
- **L216 EN**: Provides textual content or support data: `clang_enableStackTraces;`.
  **L216 CN**: 提供文本内容或支持数据：`clang_enableStackTraces;`。

### Lines 217-234

````text
    clang_equalCursors;
    clang_equalLocations;
    clang_equalRanges;
    clang_equalTypes;
    clang_executeOnThread;
    clang_findIncludesInFile;
    clang_findIncludesInFileWithBlock;
    clang_findReferencesInFile;
    clang_findReferencesInFileWithBlock;
    clang_formatDiagnostic;
    clang_free;
    clang_getAddressSpace;
    clang_getAllSkippedRanges;
    clang_getArgType;
    clang_getArrayElementType;
    clang_getArraySize;
    clang_getBuildSessionTimestamp;
    clang_getCString;
````
- **L217 EN**: Provides textual content or support data: `clang_equalCursors;`.
  **L217 CN**: 提供文本内容或支持数据：`clang_equalCursors;`。
- **L218 EN**: Provides textual content or support data: `clang_equalLocations;`.
  **L218 CN**: 提供文本内容或支持数据：`clang_equalLocations;`。
- **L219 EN**: Provides textual content or support data: `clang_equalRanges;`.
  **L219 CN**: 提供文本内容或支持数据：`clang_equalRanges;`。
- **L220 EN**: Provides textual content or support data: `clang_equalTypes;`.
  **L220 CN**: 提供文本内容或支持数据：`clang_equalTypes;`。
- **L221 EN**: Provides textual content or support data: `clang_executeOnThread;`.
  **L221 CN**: 提供文本内容或支持数据：`clang_executeOnThread;`。
- **L222 EN**: Provides textual content or support data: `clang_findIncludesInFile;`.
  **L222 CN**: 提供文本内容或支持数据：`clang_findIncludesInFile;`。
- **L223 EN**: Provides textual content or support data: `clang_findIncludesInFileWithBlock;`.
  **L223 CN**: 提供文本内容或支持数据：`clang_findIncludesInFileWithBlock;`。
- **L224 EN**: Provides textual content or support data: `clang_findReferencesInFile;`.
  **L224 CN**: 提供文本内容或支持数据：`clang_findReferencesInFile;`。
- **L225 EN**: Provides textual content or support data: `clang_findReferencesInFileWithBlock;`.
  **L225 CN**: 提供文本内容或支持数据：`clang_findReferencesInFileWithBlock;`。
- **L226 EN**: Provides textual content or support data: `clang_formatDiagnostic;`.
  **L226 CN**: 提供文本内容或支持数据：`clang_formatDiagnostic;`。
- **L227 EN**: Provides textual content or support data: `clang_free;`.
  **L227 CN**: 提供文本内容或支持数据：`clang_free;`。
- **L228 EN**: Provides textual content or support data: `clang_getAddressSpace;`.
  **L228 CN**: 提供文本内容或支持数据：`clang_getAddressSpace;`。
- **L229 EN**: Provides textual content or support data: `clang_getAllSkippedRanges;`.
  **L229 CN**: 提供文本内容或支持数据：`clang_getAllSkippedRanges;`。
- **L230 EN**: Provides textual content or support data: `clang_getArgType;`.
  **L230 CN**: 提供文本内容或支持数据：`clang_getArgType;`。
- **L231 EN**: Provides textual content or support data: `clang_getArrayElementType;`.
  **L231 CN**: 提供文本内容或支持数据：`clang_getArrayElementType;`。
- **L232 EN**: Provides textual content or support data: `clang_getArraySize;`.
  **L232 CN**: 提供文本内容或支持数据：`clang_getArraySize;`。
- **L233 EN**: Provides textual content or support data: `clang_getBuildSessionTimestamp;`.
  **L233 CN**: 提供文本内容或支持数据：`clang_getBuildSessionTimestamp;`。
- **L234 EN**: Provides textual content or support data: `clang_getCString;`.
  **L234 CN**: 提供文本内容或支持数据：`clang_getCString;`。

### Lines 235-252

````text
    clang_getCXTUResourceUsage;
    clang_getCXXAccessSpecifier;
    clang_getCanonicalCursor;
    clang_getCanonicalType;
    clang_getChildDiagnostics;
    clang_getClangVersion;
    clang_getCompletionAnnotation;
    clang_getCompletionAvailability;
    clang_getCompletionBriefComment;
    clang_getCompletionChunkCompletionString;
    clang_getCompletionChunkKind;
    clang_getCompletionChunkText;
    clang_getCompletionFixIt;
    clang_getCompletionNumAnnotations;
    clang_getCompletionNumFixIts;
    clang_getCompletionParent;
    clang_getCompletionPriority;
    clang_getCursor;
````
- **L235 EN**: Provides textual content or support data: `clang_getCXTUResourceUsage;`.
  **L235 CN**: 提供文本内容或支持数据：`clang_getCXTUResourceUsage;`。
- **L236 EN**: Provides textual content or support data: `clang_getCXXAccessSpecifier;`.
  **L236 CN**: 提供文本内容或支持数据：`clang_getCXXAccessSpecifier;`。
- **L237 EN**: Provides textual content or support data: `clang_getCanonicalCursor;`.
  **L237 CN**: 提供文本内容或支持数据：`clang_getCanonicalCursor;`。
- **L238 EN**: Provides textual content or support data: `clang_getCanonicalType;`.
  **L238 CN**: 提供文本内容或支持数据：`clang_getCanonicalType;`。
- **L239 EN**: Provides textual content or support data: `clang_getChildDiagnostics;`.
  **L239 CN**: 提供文本内容或支持数据：`clang_getChildDiagnostics;`。
- **L240 EN**: Provides textual content or support data: `clang_getClangVersion;`.
  **L240 CN**: 提供文本内容或支持数据：`clang_getClangVersion;`。
- **L241 EN**: Provides textual content or support data: `clang_getCompletionAnnotation;`.
  **L241 CN**: 提供文本内容或支持数据：`clang_getCompletionAnnotation;`。
- **L242 EN**: Provides textual content or support data: `clang_getCompletionAvailability;`.
  **L242 CN**: 提供文本内容或支持数据：`clang_getCompletionAvailability;`。
- **L243 EN**: Provides textual content or support data: `clang_getCompletionBriefComment;`.
  **L243 CN**: 提供文本内容或支持数据：`clang_getCompletionBriefComment;`。
- **L244 EN**: Provides textual content or support data: `clang_getCompletionChunkCompletionString;`.
  **L244 CN**: 提供文本内容或支持数据：`clang_getCompletionChunkCompletionString;`。
- **L245 EN**: Provides textual content or support data: `clang_getCompletionChunkKind;`.
  **L245 CN**: 提供文本内容或支持数据：`clang_getCompletionChunkKind;`。
- **L246 EN**: Provides textual content or support data: `clang_getCompletionChunkText;`.
  **L246 CN**: 提供文本内容或支持数据：`clang_getCompletionChunkText;`。
- **L247 EN**: Provides textual content or support data: `clang_getCompletionFixIt;`.
  **L247 CN**: 提供文本内容或支持数据：`clang_getCompletionFixIt;`。
- **L248 EN**: Provides textual content or support data: `clang_getCompletionNumAnnotations;`.
  **L248 CN**: 提供文本内容或支持数据：`clang_getCompletionNumAnnotations;`。
- **L249 EN**: Provides textual content or support data: `clang_getCompletionNumFixIts;`.
  **L249 CN**: 提供文本内容或支持数据：`clang_getCompletionNumFixIts;`。
- **L250 EN**: Provides textual content or support data: `clang_getCompletionParent;`.
  **L250 CN**: 提供文本内容或支持数据：`clang_getCompletionParent;`。
- **L251 EN**: Provides textual content or support data: `clang_getCompletionPriority;`.
  **L251 CN**: 提供文本内容或支持数据：`clang_getCompletionPriority;`。
- **L252 EN**: Provides textual content or support data: `clang_getCursor;`.
  **L252 CN**: 提供文本内容或支持数据：`clang_getCursor;`。

### Lines 253-270

````text
    clang_getCursorAvailability;
    clang_getCursorCompletionString;
    clang_getCursorDefinition;
    clang_getCursorDisplayName;
    clang_getCursorExceptionSpecificationType;
    clang_getCursorExtent;
    clang_getCursorKind;
    clang_getCursorKindSpelling;
    clang_getCursorLanguage;
    clang_getCursorLexicalParent;
    clang_getCursorLinkage;
    clang_getCursorLocation;
    clang_getCursorPlatformAvailability;
    clang_getCursorPrettyPrinted;
    clang_getCursorPrintingPolicy;
    clang_getCursorReferenceNameRange;
    clang_getCursorReferenced;
    clang_getCursorResultType;
````
- **L253 EN**: Provides textual content or support data: `clang_getCursorAvailability;`.
  **L253 CN**: 提供文本内容或支持数据：`clang_getCursorAvailability;`。
- **L254 EN**: Provides textual content or support data: `clang_getCursorCompletionString;`.
  **L254 CN**: 提供文本内容或支持数据：`clang_getCursorCompletionString;`。
- **L255 EN**: Provides textual content or support data: `clang_getCursorDefinition;`.
  **L255 CN**: 提供文本内容或支持数据：`clang_getCursorDefinition;`。
- **L256 EN**: Provides textual content or support data: `clang_getCursorDisplayName;`.
  **L256 CN**: 提供文本内容或支持数据：`clang_getCursorDisplayName;`。
- **L257 EN**: Provides textual content or support data: `clang_getCursorExceptionSpecificationType;`.
  **L257 CN**: 提供文本内容或支持数据：`clang_getCursorExceptionSpecificationType;`。
- **L258 EN**: Provides textual content or support data: `clang_getCursorExtent;`.
  **L258 CN**: 提供文本内容或支持数据：`clang_getCursorExtent;`。
- **L259 EN**: Provides textual content or support data: `clang_getCursorKind;`.
  **L259 CN**: 提供文本内容或支持数据：`clang_getCursorKind;`。
- **L260 EN**: Provides textual content or support data: `clang_getCursorKindSpelling;`.
  **L260 CN**: 提供文本内容或支持数据：`clang_getCursorKindSpelling;`。
- **L261 EN**: Provides textual content or support data: `clang_getCursorLanguage;`.
  **L261 CN**: 提供文本内容或支持数据：`clang_getCursorLanguage;`。
- **L262 EN**: Provides textual content or support data: `clang_getCursorLexicalParent;`.
  **L262 CN**: 提供文本内容或支持数据：`clang_getCursorLexicalParent;`。
- **L263 EN**: Provides textual content or support data: `clang_getCursorLinkage;`.
  **L263 CN**: 提供文本内容或支持数据：`clang_getCursorLinkage;`。
- **L264 EN**: Provides textual content or support data: `clang_getCursorLocation;`.
  **L264 CN**: 提供文本内容或支持数据：`clang_getCursorLocation;`。
- **L265 EN**: Provides textual content or support data: `clang_getCursorPlatformAvailability;`.
  **L265 CN**: 提供文本内容或支持数据：`clang_getCursorPlatformAvailability;`。
- **L266 EN**: Provides textual content or support data: `clang_getCursorPrettyPrinted;`.
  **L266 CN**: 提供文本内容或支持数据：`clang_getCursorPrettyPrinted;`。
- **L267 EN**: Provides textual content or support data: `clang_getCursorPrintingPolicy;`.
  **L267 CN**: 提供文本内容或支持数据：`clang_getCursorPrintingPolicy;`。
- **L268 EN**: Provides textual content or support data: `clang_getCursorReferenceNameRange;`.
  **L268 CN**: 提供文本内容或支持数据：`clang_getCursorReferenceNameRange;`。
- **L269 EN**: Provides textual content or support data: `clang_getCursorReferenced;`.
  **L269 CN**: 提供文本内容或支持数据：`clang_getCursorReferenced;`。
- **L270 EN**: Provides textual content or support data: `clang_getCursorResultType;`.
  **L270 CN**: 提供文本内容或支持数据：`clang_getCursorResultType;`。

### Lines 271-288

````text
    clang_getCursorSemanticParent;
    clang_getCursorSpelling;
    clang_getCursorTLSKind;
    clang_getCursorType;
    clang_getCursorUSR;
    clang_getCursorVisibility;
    clang_getDeclObjCTypeEncoding;
    clang_getDefinitionSpellingAndExtent;
    clang_getDiagnostic;
    clang_getDiagnosticCategory;
    clang_getDiagnosticCategoryName;
    clang_getDiagnosticCategoryText;
    clang_getDiagnosticFixIt;
    clang_getDiagnosticInSet;
    clang_getDiagnosticLocation;
    clang_getDiagnosticNumFixIts;
    clang_getDiagnosticNumRanges;
    clang_getDiagnosticOption;
````
- **L271 EN**: Provides textual content or support data: `clang_getCursorSemanticParent;`.
  **L271 CN**: 提供文本内容或支持数据：`clang_getCursorSemanticParent;`。
- **L272 EN**: Provides textual content or support data: `clang_getCursorSpelling;`.
  **L272 CN**: 提供文本内容或支持数据：`clang_getCursorSpelling;`。
- **L273 EN**: Provides textual content or support data: `clang_getCursorTLSKind;`.
  **L273 CN**: 提供文本内容或支持数据：`clang_getCursorTLSKind;`。
- **L274 EN**: Provides textual content or support data: `clang_getCursorType;`.
  **L274 CN**: 提供文本内容或支持数据：`clang_getCursorType;`。
- **L275 EN**: Provides textual content or support data: `clang_getCursorUSR;`.
  **L275 CN**: 提供文本内容或支持数据：`clang_getCursorUSR;`。
- **L276 EN**: Provides textual content or support data: `clang_getCursorVisibility;`.
  **L276 CN**: 提供文本内容或支持数据：`clang_getCursorVisibility;`。
- **L277 EN**: Provides textual content or support data: `clang_getDeclObjCTypeEncoding;`.
  **L277 CN**: 提供文本内容或支持数据：`clang_getDeclObjCTypeEncoding;`。
- **L278 EN**: Provides textual content or support data: `clang_getDefinitionSpellingAndExtent;`.
  **L278 CN**: 提供文本内容或支持数据：`clang_getDefinitionSpellingAndExtent;`。
- **L279 EN**: Provides textual content or support data: `clang_getDiagnostic;`.
  **L279 CN**: 提供文本内容或支持数据：`clang_getDiagnostic;`。
- **L280 EN**: Provides textual content or support data: `clang_getDiagnosticCategory;`.
  **L280 CN**: 提供文本内容或支持数据：`clang_getDiagnosticCategory;`。
- **L281 EN**: Provides textual content or support data: `clang_getDiagnosticCategoryName;`.
  **L281 CN**: 提供文本内容或支持数据：`clang_getDiagnosticCategoryName;`。
- **L282 EN**: Provides textual content or support data: `clang_getDiagnosticCategoryText;`.
  **L282 CN**: 提供文本内容或支持数据：`clang_getDiagnosticCategoryText;`。
- **L283 EN**: Provides textual content or support data: `clang_getDiagnosticFixIt;`.
  **L283 CN**: 提供文本内容或支持数据：`clang_getDiagnosticFixIt;`。
- **L284 EN**: Provides textual content or support data: `clang_getDiagnosticInSet;`.
  **L284 CN**: 提供文本内容或支持数据：`clang_getDiagnosticInSet;`。
- **L285 EN**: Provides textual content or support data: `clang_getDiagnosticLocation;`.
  **L285 CN**: 提供文本内容或支持数据：`clang_getDiagnosticLocation;`。
- **L286 EN**: Provides textual content or support data: `clang_getDiagnosticNumFixIts;`.
  **L286 CN**: 提供文本内容或支持数据：`clang_getDiagnosticNumFixIts;`。
- **L287 EN**: Provides textual content or support data: `clang_getDiagnosticNumRanges;`.
  **L287 CN**: 提供文本内容或支持数据：`clang_getDiagnosticNumRanges;`。
- **L288 EN**: Provides textual content or support data: `clang_getDiagnosticOption;`.
  **L288 CN**: 提供文本内容或支持数据：`clang_getDiagnosticOption;`。

### Lines 289-306

````text
    clang_getDiagnosticRange;
    clang_getDiagnosticSetFromTU;
    clang_getDiagnosticSeverity;
    clang_getDiagnosticSpelling;
    clang_getElementType;
    clang_getEnumConstantDeclUnsignedValue;
    clang_getEnumConstantDeclValue;
    clang_getEnumDeclIntegerType;
    clang_getExceptionSpecificationType;
    clang_getExpansionLocation;
    clang_getFieldDeclBitWidth;
    clang_getFile;
    clang_getFileContents;
    clang_getFileLocation;
    clang_getFileName;
    clang_getFileTime;
    clang_getFileUniqueID;
    clang_getFunctionTypeCallingConv;
````
- **L289 EN**: Provides textual content or support data: `clang_getDiagnosticRange;`.
  **L289 CN**: 提供文本内容或支持数据：`clang_getDiagnosticRange;`。
- **L290 EN**: Provides textual content or support data: `clang_getDiagnosticSetFromTU;`.
  **L290 CN**: 提供文本内容或支持数据：`clang_getDiagnosticSetFromTU;`。
- **L291 EN**: Provides textual content or support data: `clang_getDiagnosticSeverity;`.
  **L291 CN**: 提供文本内容或支持数据：`clang_getDiagnosticSeverity;`。
- **L292 EN**: Provides textual content or support data: `clang_getDiagnosticSpelling;`.
  **L292 CN**: 提供文本内容或支持数据：`clang_getDiagnosticSpelling;`。
- **L293 EN**: Provides textual content or support data: `clang_getElementType;`.
  **L293 CN**: 提供文本内容或支持数据：`clang_getElementType;`。
- **L294 EN**: Provides textual content or support data: `clang_getEnumConstantDeclUnsignedValue;`.
  **L294 CN**: 提供文本内容或支持数据：`clang_getEnumConstantDeclUnsignedValue;`。
- **L295 EN**: Provides textual content or support data: `clang_getEnumConstantDeclValue;`.
  **L295 CN**: 提供文本内容或支持数据：`clang_getEnumConstantDeclValue;`。
- **L296 EN**: Provides textual content or support data: `clang_getEnumDeclIntegerType;`.
  **L296 CN**: 提供文本内容或支持数据：`clang_getEnumDeclIntegerType;`。
- **L297 EN**: Provides textual content or support data: `clang_getExceptionSpecificationType;`.
  **L297 CN**: 提供文本内容或支持数据：`clang_getExceptionSpecificationType;`。
- **L298 EN**: Provides textual content or support data: `clang_getExpansionLocation;`.
  **L298 CN**: 提供文本内容或支持数据：`clang_getExpansionLocation;`。
- **L299 EN**: Provides textual content or support data: `clang_getFieldDeclBitWidth;`.
  **L299 CN**: 提供文本内容或支持数据：`clang_getFieldDeclBitWidth;`。
- **L300 EN**: Provides textual content or support data: `clang_getFile;`.
  **L300 CN**: 提供文本内容或支持数据：`clang_getFile;`。
- **L301 EN**: Provides textual content or support data: `clang_getFileContents;`.
  **L301 CN**: 提供文本内容或支持数据：`clang_getFileContents;`。
- **L302 EN**: Provides textual content or support data: `clang_getFileLocation;`.
  **L302 CN**: 提供文本内容或支持数据：`clang_getFileLocation;`。
- **L303 EN**: Provides textual content or support data: `clang_getFileName;`.
  **L303 CN**: 提供文本内容或支持数据：`clang_getFileName;`。
- **L304 EN**: Provides textual content or support data: `clang_getFileTime;`.
  **L304 CN**: 提供文本内容或支持数据：`clang_getFileTime;`。
- **L305 EN**: Provides textual content or support data: `clang_getFileUniqueID;`.
  **L305 CN**: 提供文本内容或支持数据：`clang_getFileUniqueID;`。
- **L306 EN**: Provides textual content or support data: `clang_getFunctionTypeCallingConv;`.
  **L306 CN**: 提供文本内容或支持数据：`clang_getFunctionTypeCallingConv;`。

### Lines 307-324

````text
    clang_getIBOutletCollectionType;
    clang_getIncludedFile;
    clang_getInclusions;
    clang_getInstantiationLocation;
    clang_getLocation;
    clang_getLocationForOffset;
    clang_getModuleForFile;
    clang_getNullCursor;
    clang_getNullLocation;
    clang_getNullRange;
    clang_getNumArgTypes;
    clang_getNumCompletionChunks;
    clang_getNumDiagnostics;
    clang_getNumDiagnosticsInSet;
    clang_getNumElements;
    clang_getNumOverloadedDecls;
    clang_getOverloadedDecl;
    clang_getOverriddenCursors;
````
- **L307 EN**: Provides textual content or support data: `clang_getIBOutletCollectionType;`.
  **L307 CN**: 提供文本内容或支持数据：`clang_getIBOutletCollectionType;`。
- **L308 EN**: Provides textual content or support data: `clang_getIncludedFile;`.
  **L308 CN**: 提供文本内容或支持数据：`clang_getIncludedFile;`。
- **L309 EN**: Provides textual content or support data: `clang_getInclusions;`.
  **L309 CN**: 提供文本内容或支持数据：`clang_getInclusions;`。
- **L310 EN**: Provides textual content or support data: `clang_getInstantiationLocation;`.
  **L310 CN**: 提供文本内容或支持数据：`clang_getInstantiationLocation;`。
- **L311 EN**: Provides textual content or support data: `clang_getLocation;`.
  **L311 CN**: 提供文本内容或支持数据：`clang_getLocation;`。
- **L312 EN**: Provides textual content or support data: `clang_getLocationForOffset;`.
  **L312 CN**: 提供文本内容或支持数据：`clang_getLocationForOffset;`。
- **L313 EN**: Provides textual content or support data: `clang_getModuleForFile;`.
  **L313 CN**: 提供文本内容或支持数据：`clang_getModuleForFile;`。
- **L314 EN**: Provides textual content or support data: `clang_getNullCursor;`.
  **L314 CN**: 提供文本内容或支持数据：`clang_getNullCursor;`。
- **L315 EN**: Provides textual content or support data: `clang_getNullLocation;`.
  **L315 CN**: 提供文本内容或支持数据：`clang_getNullLocation;`。
- **L316 EN**: Provides textual content or support data: `clang_getNullRange;`.
  **L316 CN**: 提供文本内容或支持数据：`clang_getNullRange;`。
- **L317 EN**: Provides textual content or support data: `clang_getNumArgTypes;`.
  **L317 CN**: 提供文本内容或支持数据：`clang_getNumArgTypes;`。
- **L318 EN**: Provides textual content or support data: `clang_getNumCompletionChunks;`.
  **L318 CN**: 提供文本内容或支持数据：`clang_getNumCompletionChunks;`。
- **L319 EN**: Provides textual content or support data: `clang_getNumDiagnostics;`.
  **L319 CN**: 提供文本内容或支持数据：`clang_getNumDiagnostics;`。
- **L320 EN**: Provides textual content or support data: `clang_getNumDiagnosticsInSet;`.
  **L320 CN**: 提供文本内容或支持数据：`clang_getNumDiagnosticsInSet;`。
- **L321 EN**: Provides textual content or support data: `clang_getNumElements;`.
  **L321 CN**: 提供文本内容或支持数据：`clang_getNumElements;`。
- **L322 EN**: Provides textual content or support data: `clang_getNumOverloadedDecls;`.
  **L322 CN**: 提供文本内容或支持数据：`clang_getNumOverloadedDecls;`。
- **L323 EN**: Provides textual content or support data: `clang_getOverloadedDecl;`.
  **L323 CN**: 提供文本内容或支持数据：`clang_getOverloadedDecl;`。
- **L324 EN**: Provides textual content or support data: `clang_getOverriddenCursors;`.
  **L324 CN**: 提供文本内容或支持数据：`clang_getOverriddenCursors;`。

### Lines 325-342

````text
    clang_getPointeeType;
    clang_getPresumedLocation;
    clang_getRange;
    clang_getRangeEnd;
    clang_getRangeStart;
    clang_getRemappings;
    clang_getRemappingsFromFileList;
    clang_getResultType;
    clang_getSkippedRanges;
    clang_getSpecializedCursorTemplate;
    clang_getSpellingLocation;
    clang_getTUResourceUsageName;
    clang_getTemplateCursorKind;
    clang_getToken;
    clang_getTokenExtent;
    clang_getTokenKind;
    clang_getTokenLocation;
    clang_getTokenSpelling;
````
- **L325 EN**: Provides textual content or support data: `clang_getPointeeType;`.
  **L325 CN**: 提供文本内容或支持数据：`clang_getPointeeType;`。
- **L326 EN**: Provides textual content or support data: `clang_getPresumedLocation;`.
  **L326 CN**: 提供文本内容或支持数据：`clang_getPresumedLocation;`。
- **L327 EN**: Provides textual content or support data: `clang_getRange;`.
  **L327 CN**: 提供文本内容或支持数据：`clang_getRange;`。
- **L328 EN**: Provides textual content or support data: `clang_getRangeEnd;`.
  **L328 CN**: 提供文本内容或支持数据：`clang_getRangeEnd;`。
- **L329 EN**: Provides textual content or support data: `clang_getRangeStart;`.
  **L329 CN**: 提供文本内容或支持数据：`clang_getRangeStart;`。
- **L330 EN**: Provides textual content or support data: `clang_getRemappings;`.
  **L330 CN**: 提供文本内容或支持数据：`clang_getRemappings;`。
- **L331 EN**: Provides textual content or support data: `clang_getRemappingsFromFileList;`.
  **L331 CN**: 提供文本内容或支持数据：`clang_getRemappingsFromFileList;`。
- **L332 EN**: Provides textual content or support data: `clang_getResultType;`.
  **L332 CN**: 提供文本内容或支持数据：`clang_getResultType;`。
- **L333 EN**: Provides textual content or support data: `clang_getSkippedRanges;`.
  **L333 CN**: 提供文本内容或支持数据：`clang_getSkippedRanges;`。
- **L334 EN**: Provides textual content or support data: `clang_getSpecializedCursorTemplate;`.
  **L334 CN**: 提供文本内容或支持数据：`clang_getSpecializedCursorTemplate;`。
- **L335 EN**: Provides textual content or support data: `clang_getSpellingLocation;`.
  **L335 CN**: 提供文本内容或支持数据：`clang_getSpellingLocation;`。
- **L336 EN**: Provides textual content or support data: `clang_getTUResourceUsageName;`.
  **L336 CN**: 提供文本内容或支持数据：`clang_getTUResourceUsageName;`。
- **L337 EN**: Provides textual content or support data: `clang_getTemplateCursorKind;`.
  **L337 CN**: 提供文本内容或支持数据：`clang_getTemplateCursorKind;`。
- **L338 EN**: Provides textual content or support data: `clang_getToken;`.
  **L338 CN**: 提供文本内容或支持数据：`clang_getToken;`。
- **L339 EN**: Provides textual content or support data: `clang_getTokenExtent;`.
  **L339 CN**: 提供文本内容或支持数据：`clang_getTokenExtent;`。
- **L340 EN**: Provides textual content or support data: `clang_getTokenKind;`.
  **L340 CN**: 提供文本内容或支持数据：`clang_getTokenKind;`。
- **L341 EN**: Provides textual content or support data: `clang_getTokenLocation;`.
  **L341 CN**: 提供文本内容或支持数据：`clang_getTokenLocation;`。
- **L342 EN**: Provides textual content or support data: `clang_getTokenSpelling;`.
  **L342 CN**: 提供文本内容或支持数据：`clang_getTokenSpelling;`。

### Lines 343-360

````text
    clang_getTranslationUnitCursor;
    clang_getTranslationUnitSpelling;
    clang_getTranslationUnitTargetInfo;
    clang_getTypeDeclaration;
    clang_getTypeKindSpelling;
    clang_getTypeSpelling;
    clang_getTypedefDeclUnderlyingType;
    clang_getTypedefName;
    clang_hashCursor;
    clang_indexLoc_getCXSourceLocation;
    clang_indexLoc_getFileLocation;
    clang_indexSourceFile;
    clang_indexSourceFileFullArgv;
    clang_indexTranslationUnit;
    clang_index_getCXXClassDeclInfo;
    clang_index_getClientContainer;
    clang_index_getClientEntity;
    clang_index_getIBOutletCollectionAttrInfo;
````
- **L343 EN**: Provides textual content or support data: `clang_getTranslationUnitCursor;`.
  **L343 CN**: 提供文本内容或支持数据：`clang_getTranslationUnitCursor;`。
- **L344 EN**: Provides textual content or support data: `clang_getTranslationUnitSpelling;`.
  **L344 CN**: 提供文本内容或支持数据：`clang_getTranslationUnitSpelling;`。
- **L345 EN**: Provides textual content or support data: `clang_getTranslationUnitTargetInfo;`.
  **L345 CN**: 提供文本内容或支持数据：`clang_getTranslationUnitTargetInfo;`。
- **L346 EN**: Provides textual content or support data: `clang_getTypeDeclaration;`.
  **L346 CN**: 提供文本内容或支持数据：`clang_getTypeDeclaration;`。
- **L347 EN**: Provides textual content or support data: `clang_getTypeKindSpelling;`.
  **L347 CN**: 提供文本内容或支持数据：`clang_getTypeKindSpelling;`。
- **L348 EN**: Provides textual content or support data: `clang_getTypeSpelling;`.
  **L348 CN**: 提供文本内容或支持数据：`clang_getTypeSpelling;`。
- **L349 EN**: Provides textual content or support data: `clang_getTypedefDeclUnderlyingType;`.
  **L349 CN**: 提供文本内容或支持数据：`clang_getTypedefDeclUnderlyingType;`。
- **L350 EN**: Provides textual content or support data: `clang_getTypedefName;`.
  **L350 CN**: 提供文本内容或支持数据：`clang_getTypedefName;`。
- **L351 EN**: Provides textual content or support data: `clang_hashCursor;`.
  **L351 CN**: 提供文本内容或支持数据：`clang_hashCursor;`。
- **L352 EN**: Provides textual content or support data: `clang_indexLoc_getCXSourceLocation;`.
  **L352 CN**: 提供文本内容或支持数据：`clang_indexLoc_getCXSourceLocation;`。
- **L353 EN**: Provides textual content or support data: `clang_indexLoc_getFileLocation;`.
  **L353 CN**: 提供文本内容或支持数据：`clang_indexLoc_getFileLocation;`。
- **L354 EN**: Provides textual content or support data: `clang_indexSourceFile;`.
  **L354 CN**: 提供文本内容或支持数据：`clang_indexSourceFile;`。
- **L355 EN**: Provides textual content or support data: `clang_indexSourceFileFullArgv;`.
  **L355 CN**: 提供文本内容或支持数据：`clang_indexSourceFileFullArgv;`。
- **L356 EN**: Provides textual content or support data: `clang_indexTranslationUnit;`.
  **L356 CN**: 提供文本内容或支持数据：`clang_indexTranslationUnit;`。
- **L357 EN**: Provides textual content or support data: `clang_index_getCXXClassDeclInfo;`.
  **L357 CN**: 提供文本内容或支持数据：`clang_index_getCXXClassDeclInfo;`。
- **L358 EN**: Provides textual content or support data: `clang_index_getClientContainer;`.
  **L358 CN**: 提供文本内容或支持数据：`clang_index_getClientContainer;`。
- **L359 EN**: Provides textual content or support data: `clang_index_getClientEntity;`.
  **L359 CN**: 提供文本内容或支持数据：`clang_index_getClientEntity;`。
- **L360 EN**: Provides textual content or support data: `clang_index_getIBOutletCollectionAttrInfo;`.
  **L360 CN**: 提供文本内容或支持数据：`clang_index_getIBOutletCollectionAttrInfo;`。

### Lines 361-378

````text
    clang_index_getObjCCategoryDeclInfo;
    clang_index_getObjCContainerDeclInfo;
    clang_index_getObjCInterfaceDeclInfo;
    clang_index_getObjCPropertyDeclInfo;
    clang_index_getObjCProtocolRefListInfo;
    clang_index_isEntityObjCContainerKind;
    clang_index_setClientContainer;
    clang_index_setClientEntity;
    clang_install_aborting_llvm_fatal_error_handler;
    clang_isAttribute;
    clang_isConstQualifiedType;
    clang_isCursorDefinition;
    clang_isDeclaration;
    clang_isExpression;
    clang_isFileMultipleIncludeGuarded;
    clang_isFunctionTypeVariadic;
    clang_isInvalid;
    clang_isInvalidDeclaration;
````
- **L361 EN**: Provides textual content or support data: `clang_index_getObjCCategoryDeclInfo;`.
  **L361 CN**: 提供文本内容或支持数据：`clang_index_getObjCCategoryDeclInfo;`。
- **L362 EN**: Provides textual content or support data: `clang_index_getObjCContainerDeclInfo;`.
  **L362 CN**: 提供文本内容或支持数据：`clang_index_getObjCContainerDeclInfo;`。
- **L363 EN**: Provides textual content or support data: `clang_index_getObjCInterfaceDeclInfo;`.
  **L363 CN**: 提供文本内容或支持数据：`clang_index_getObjCInterfaceDeclInfo;`。
- **L364 EN**: Provides textual content or support data: `clang_index_getObjCPropertyDeclInfo;`.
  **L364 CN**: 提供文本内容或支持数据：`clang_index_getObjCPropertyDeclInfo;`。
- **L365 EN**: Provides textual content or support data: `clang_index_getObjCProtocolRefListInfo;`.
  **L365 CN**: 提供文本内容或支持数据：`clang_index_getObjCProtocolRefListInfo;`。
- **L366 EN**: Provides textual content or support data: `clang_index_isEntityObjCContainerKind;`.
  **L366 CN**: 提供文本内容或支持数据：`clang_index_isEntityObjCContainerKind;`。
- **L367 EN**: Provides textual content or support data: `clang_index_setClientContainer;`.
  **L367 CN**: 提供文本内容或支持数据：`clang_index_setClientContainer;`。
- **L368 EN**: Provides textual content or support data: `clang_index_setClientEntity;`.
  **L368 CN**: 提供文本内容或支持数据：`clang_index_setClientEntity;`。
- **L369 EN**: Provides textual content or support data: `clang_install_aborting_llvm_fatal_error_handler;`.
  **L369 CN**: 提供文本内容或支持数据：`clang_install_aborting_llvm_fatal_error_handler;`。
- **L370 EN**: Provides textual content or support data: `clang_isAttribute;`.
  **L370 CN**: 提供文本内容或支持数据：`clang_isAttribute;`。
- **L371 EN**: Provides textual content or support data: `clang_isConstQualifiedType;`.
  **L371 CN**: 提供文本内容或支持数据：`clang_isConstQualifiedType;`。
- **L372 EN**: Provides textual content or support data: `clang_isCursorDefinition;`.
  **L372 CN**: 提供文本内容或支持数据：`clang_isCursorDefinition;`。
- **L373 EN**: Provides textual content or support data: `clang_isDeclaration;`.
  **L373 CN**: 提供文本内容或支持数据：`clang_isDeclaration;`。
- **L374 EN**: Provides textual content or support data: `clang_isExpression;`.
  **L374 CN**: 提供文本内容或支持数据：`clang_isExpression;`。
- **L375 EN**: Provides textual content or support data: `clang_isFileMultipleIncludeGuarded;`.
  **L375 CN**: 提供文本内容或支持数据：`clang_isFileMultipleIncludeGuarded;`。
- **L376 EN**: Provides textual content or support data: `clang_isFunctionTypeVariadic;`.
  **L376 CN**: 提供文本内容或支持数据：`clang_isFunctionTypeVariadic;`。
- **L377 EN**: Provides textual content or support data: `clang_isInvalid;`.
  **L377 CN**: 提供文本内容或支持数据：`clang_isInvalid;`。
- **L378 EN**: Provides textual content or support data: `clang_isInvalidDeclaration;`.
  **L378 CN**: 提供文本内容或支持数据：`clang_isInvalidDeclaration;`。

### Lines 379-396

````text
    clang_isPODType;
    clang_isPreprocessing;
    clang_isReference;
    clang_isRestrictQualifiedType;
    clang_isStatement;
    clang_isTranslationUnit;
    clang_isUnexposed;
    clang_isVirtualBase;
    clang_isVolatileQualifiedType;
    clang_loadDiagnostics;
    clang_parseTranslationUnit;
    clang_parseTranslationUnit2;
    clang_parseTranslationUnit2FullArgv;
    clang_remap_dispose;
    clang_remap_getFilenames;
    clang_remap_getNumFiles;
    clang_reparseTranslationUnit;
    clang_saveTranslationUnit;
````
- **L379 EN**: Provides textual content or support data: `clang_isPODType;`.
  **L379 CN**: 提供文本内容或支持数据：`clang_isPODType;`。
- **L380 EN**: Provides textual content or support data: `clang_isPreprocessing;`.
  **L380 CN**: 提供文本内容或支持数据：`clang_isPreprocessing;`。
- **L381 EN**: Provides textual content or support data: `clang_isReference;`.
  **L381 CN**: 提供文本内容或支持数据：`clang_isReference;`。
- **L382 EN**: Provides textual content or support data: `clang_isRestrictQualifiedType;`.
  **L382 CN**: 提供文本内容或支持数据：`clang_isRestrictQualifiedType;`。
- **L383 EN**: Provides textual content or support data: `clang_isStatement;`.
  **L383 CN**: 提供文本内容或支持数据：`clang_isStatement;`。
- **L384 EN**: Provides textual content or support data: `clang_isTranslationUnit;`.
  **L384 CN**: 提供文本内容或支持数据：`clang_isTranslationUnit;`。
- **L385 EN**: Provides textual content or support data: `clang_isUnexposed;`.
  **L385 CN**: 提供文本内容或支持数据：`clang_isUnexposed;`。
- **L386 EN**: Provides textual content or support data: `clang_isVirtualBase;`.
  **L386 CN**: 提供文本内容或支持数据：`clang_isVirtualBase;`。
- **L387 EN**: Provides textual content or support data: `clang_isVolatileQualifiedType;`.
  **L387 CN**: 提供文本内容或支持数据：`clang_isVolatileQualifiedType;`。
- **L388 EN**: Provides textual content or support data: `clang_loadDiagnostics;`.
  **L388 CN**: 提供文本内容或支持数据：`clang_loadDiagnostics;`。
- **L389 EN**: Provides textual content or support data: `clang_parseTranslationUnit;`.
  **L389 CN**: 提供文本内容或支持数据：`clang_parseTranslationUnit;`。
- **L390 EN**: Provides textual content or support data: `clang_parseTranslationUnit2;`.
  **L390 CN**: 提供文本内容或支持数据：`clang_parseTranslationUnit2;`。
- **L391 EN**: Provides textual content or support data: `clang_parseTranslationUnit2FullArgv;`.
  **L391 CN**: 提供文本内容或支持数据：`clang_parseTranslationUnit2FullArgv;`。
- **L392 EN**: Provides textual content or support data: `clang_remap_dispose;`.
  **L392 CN**: 提供文本内容或支持数据：`clang_remap_dispose;`。
- **L393 EN**: Provides textual content or support data: `clang_remap_getFilenames;`.
  **L393 CN**: 提供文本内容或支持数据：`clang_remap_getFilenames;`。
- **L394 EN**: Provides textual content or support data: `clang_remap_getNumFiles;`.
  **L394 CN**: 提供文本内容或支持数据：`clang_remap_getNumFiles;`。
- **L395 EN**: Provides textual content or support data: `clang_reparseTranslationUnit;`.
  **L395 CN**: 提供文本内容或支持数据：`clang_reparseTranslationUnit;`。
- **L396 EN**: Provides textual content or support data: `clang_saveTranslationUnit;`.
  **L396 CN**: 提供文本内容或支持数据：`clang_saveTranslationUnit;`。

### Lines 397-414

````text
    clang_sortCodeCompletionResults;
    clang_suspendTranslationUnit;
    clang_toggleCrashRecovery;
    clang_tokenize;
    clang_uninstall_llvm_fatal_error_handler;
    clang_visitChildren;
    clang_visitChildrenWithBlock;

  local: *;
};

LLVM_16 {
  global:
    clang_getUnqualifiedType;
    clang_getNonReferenceType;
    clang_CXXMethod_isDeleted;
    clang_CXXMethod_isCopyAssignmentOperator;
    clang_CXXMethod_isMoveAssignmentOperator;
````
- **L397 EN**: Provides textual content or support data: `clang_sortCodeCompletionResults;`.
  **L397 CN**: 提供文本内容或支持数据：`clang_sortCodeCompletionResults;`。
- **L398 EN**: Provides textual content or support data: `clang_suspendTranslationUnit;`.
  **L398 CN**: 提供文本内容或支持数据：`clang_suspendTranslationUnit;`。
- **L399 EN**: Provides textual content or support data: `clang_toggleCrashRecovery;`.
  **L399 CN**: 提供文本内容或支持数据：`clang_toggleCrashRecovery;`。
- **L400 EN**: Provides textual content or support data: `clang_tokenize;`.
  **L400 CN**: 提供文本内容或支持数据：`clang_tokenize;`。
- **L401 EN**: Provides textual content or support data: `clang_uninstall_llvm_fatal_error_handler;`.
  **L401 CN**: 提供文本内容或支持数据：`clang_uninstall_llvm_fatal_error_handler;`。
- **L402 EN**: Provides textual content or support data: `clang_visitChildren;`.
  **L402 CN**: 提供文本内容或支持数据：`clang_visitChildren;`。
- **L403 EN**: Provides textual content or support data: `clang_visitChildrenWithBlock;`.
  **L403 CN**: 提供文本内容或支持数据：`clang_visitChildrenWithBlock;`。
- **L404 EN**: Blank line separating nearby declarations or logic blocks.
  **L404 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L405 EN**: Provides textual content or support data: `local: *;`.
  **L405 CN**: 提供文本内容或支持数据：`local: *;`。
- **L406 EN**: Provides textual content or support data: `};`.
  **L406 CN**: 提供文本内容或支持数据：`};`。
- **L407 EN**: Blank line separating nearby declarations or logic blocks.
  **L407 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L408 EN**: Provides textual content or support data: `LLVM_16 {`.
  **L408 CN**: 提供文本内容或支持数据：`LLVM_16 {`。
- **L409 EN**: Introduces a labeled text section: `global:`.
  **L409 CN**: 引入一个带标签的文本段落：`global:`。
- **L410 EN**: Provides textual content or support data: `clang_getUnqualifiedType;`.
  **L410 CN**: 提供文本内容或支持数据：`clang_getUnqualifiedType;`。
- **L411 EN**: Provides textual content or support data: `clang_getNonReferenceType;`.
  **L411 CN**: 提供文本内容或支持数据：`clang_getNonReferenceType;`。
- **L412 EN**: Provides textual content or support data: `clang_CXXMethod_isDeleted;`.
  **L412 CN**: 提供文本内容或支持数据：`clang_CXXMethod_isDeleted;`。
- **L413 EN**: Provides textual content or support data: `clang_CXXMethod_isCopyAssignmentOperator;`.
  **L413 CN**: 提供文本内容或支持数据：`clang_CXXMethod_isCopyAssignmentOperator;`。
- **L414 EN**: Provides textual content or support data: `clang_CXXMethod_isMoveAssignmentOperator;`.
  **L414 CN**: 提供文本内容或支持数据：`clang_CXXMethod_isMoveAssignmentOperator;`。

### Lines 415-432

````text
    clang_createAPISet;
    clang_disposeAPISet;
    clang_getSymbolGraphForCursor;
    clang_getSymbolGraphForUSR;
};

LLVM_17 {
  global:
    clang_CXXMethod_isExplicit;
    clang_createIndexWithOptions;
    clang_getBinaryOperatorKindSpelling;
    clang_getCursorBinaryOperatorKind;
    clang_getUnaryOperatorKindSpelling;
    clang_getCursorUnaryOperatorKind;
};

LLVM_19 {
  global:
````
- **L415 EN**: Provides textual content or support data: `clang_createAPISet;`.
  **L415 CN**: 提供文本内容或支持数据：`clang_createAPISet;`。
- **L416 EN**: Provides textual content or support data: `clang_disposeAPISet;`.
  **L416 CN**: 提供文本内容或支持数据：`clang_disposeAPISet;`。
- **L417 EN**: Provides textual content or support data: `clang_getSymbolGraphForCursor;`.
  **L417 CN**: 提供文本内容或支持数据：`clang_getSymbolGraphForCursor;`。
- **L418 EN**: Provides textual content or support data: `clang_getSymbolGraphForUSR;`.
  **L418 CN**: 提供文本内容或支持数据：`clang_getSymbolGraphForUSR;`。
- **L419 EN**: Provides textual content or support data: `};`.
  **L419 CN**: 提供文本内容或支持数据：`};`。
- **L420 EN**: Blank line separating nearby declarations or logic blocks.
  **L420 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L421 EN**: Provides textual content or support data: `LLVM_17 {`.
  **L421 CN**: 提供文本内容或支持数据：`LLVM_17 {`。
- **L422 EN**: Introduces a labeled text section: `global:`.
  **L422 CN**: 引入一个带标签的文本段落：`global:`。
- **L423 EN**: Provides textual content or support data: `clang_CXXMethod_isExplicit;`.
  **L423 CN**: 提供文本内容或支持数据：`clang_CXXMethod_isExplicit;`。
- **L424 EN**: Provides textual content or support data: `clang_createIndexWithOptions;`.
  **L424 CN**: 提供文本内容或支持数据：`clang_createIndexWithOptions;`。
- **L425 EN**: Provides textual content or support data: `clang_getBinaryOperatorKindSpelling;`.
  **L425 CN**: 提供文本内容或支持数据：`clang_getBinaryOperatorKindSpelling;`。
- **L426 EN**: Provides textual content or support data: `clang_getCursorBinaryOperatorKind;`.
  **L426 CN**: 提供文本内容或支持数据：`clang_getCursorBinaryOperatorKind;`。
- **L427 EN**: Provides textual content or support data: `clang_getUnaryOperatorKindSpelling;`.
  **L427 CN**: 提供文本内容或支持数据：`clang_getUnaryOperatorKindSpelling;`。
- **L428 EN**: Provides textual content or support data: `clang_getCursorUnaryOperatorKind;`.
  **L428 CN**: 提供文本内容或支持数据：`clang_getCursorUnaryOperatorKind;`。
- **L429 EN**: Provides textual content or support data: `};`.
  **L429 CN**: 提供文本内容或支持数据：`};`。
- **L430 EN**: Blank line separating nearby declarations or logic blocks.
  **L430 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L431 EN**: Provides textual content or support data: `LLVM_19 {`.
  **L431 CN**: 提供文本内容或支持数据：`LLVM_19 {`。
- **L432 EN**: Introduces a labeled text section: `global:`.
  **L432 CN**: 引入一个带标签的文本段落：`global:`。

### Lines 433-450

````text
    clang_Cursor_getBinaryOpcode;
    clang_Cursor_getBinaryOpcodeStr;
};

LLVM_20 {
  global:
    clang_getOffsetOfBase;
    clang_getTypePrettyPrinted;
    clang_isBeforeInTranslationUnit;
    clang_visitCXXBaseClasses;
};

LLVM_21 {
  global:
    clang_getFullyQualifiedName;
    clang_visitCXXMethods;
    clang_Cursor_getGCCAssemblyTemplate;
    clang_Cursor_isGCCAssemblyHasGoto;
````
- **L433 EN**: Provides textual content or support data: `clang_Cursor_getBinaryOpcode;`.
  **L433 CN**: 提供文本内容或支持数据：`clang_Cursor_getBinaryOpcode;`。
- **L434 EN**: Provides textual content or support data: `clang_Cursor_getBinaryOpcodeStr;`.
  **L434 CN**: 提供文本内容或支持数据：`clang_Cursor_getBinaryOpcodeStr;`。
- **L435 EN**: Provides textual content or support data: `};`.
  **L435 CN**: 提供文本内容或支持数据：`};`。
- **L436 EN**: Blank line separating nearby declarations or logic blocks.
  **L436 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L437 EN**: Provides textual content or support data: `LLVM_20 {`.
  **L437 CN**: 提供文本内容或支持数据：`LLVM_20 {`。
- **L438 EN**: Introduces a labeled text section: `global:`.
  **L438 CN**: 引入一个带标签的文本段落：`global:`。
- **L439 EN**: Provides textual content or support data: `clang_getOffsetOfBase;`.
  **L439 CN**: 提供文本内容或支持数据：`clang_getOffsetOfBase;`。
- **L440 EN**: Provides textual content or support data: `clang_getTypePrettyPrinted;`.
  **L440 CN**: 提供文本内容或支持数据：`clang_getTypePrettyPrinted;`。
- **L441 EN**: Provides textual content or support data: `clang_isBeforeInTranslationUnit;`.
  **L441 CN**: 提供文本内容或支持数据：`clang_isBeforeInTranslationUnit;`。
- **L442 EN**: Provides textual content or support data: `clang_visitCXXBaseClasses;`.
  **L442 CN**: 提供文本内容或支持数据：`clang_visitCXXBaseClasses;`。
- **L443 EN**: Provides textual content or support data: `};`.
  **L443 CN**: 提供文本内容或支持数据：`};`。
- **L444 EN**: Blank line separating nearby declarations or logic blocks.
  **L444 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L445 EN**: Provides textual content or support data: `LLVM_21 {`.
  **L445 CN**: 提供文本内容或支持数据：`LLVM_21 {`。
- **L446 EN**: Introduces a labeled text section: `global:`.
  **L446 CN**: 引入一个带标签的文本段落：`global:`。
- **L447 EN**: Provides textual content or support data: `clang_getFullyQualifiedName;`.
  **L447 CN**: 提供文本内容或支持数据：`clang_getFullyQualifiedName;`。
- **L448 EN**: Provides textual content or support data: `clang_visitCXXMethods;`.
  **L448 CN**: 提供文本内容或支持数据：`clang_visitCXXMethods;`。
- **L449 EN**: Provides textual content or support data: `clang_Cursor_getGCCAssemblyTemplate;`.
  **L449 CN**: 提供文本内容或支持数据：`clang_Cursor_getGCCAssemblyTemplate;`。
- **L450 EN**: Provides textual content or support data: `clang_Cursor_isGCCAssemblyHasGoto;`.
  **L450 CN**: 提供文本内容或支持数据：`clang_Cursor_isGCCAssemblyHasGoto;`。

### Lines 451-468

````text
    clang_Cursor_getGCCAssemblyNumOutputs;
    clang_Cursor_getGCCAssemblyNumInputs;
    clang_Cursor_getGCCAssemblyInput;
    clang_Cursor_getGCCAssemblyOutput;
    clang_Cursor_getGCCAssemblyNumClobbers;
    clang_Cursor_getGCCAssemblyClobber;
    clang_Cursor_isGCCAssemblyVolatile;
};

LLVM_23 {
  global:
    clang_ModuleCache_prune;
};

# Example of how to add a new symbol version entry.  If you do add a new symbol
# version, please update the example to depend on the version you added.
# LLVM_X {
# global:
````
- **L451 EN**: Provides textual content or support data: `clang_Cursor_getGCCAssemblyNumOutputs;`.
  **L451 CN**: 提供文本内容或支持数据：`clang_Cursor_getGCCAssemblyNumOutputs;`。
- **L452 EN**: Provides textual content or support data: `clang_Cursor_getGCCAssemblyNumInputs;`.
  **L452 CN**: 提供文本内容或支持数据：`clang_Cursor_getGCCAssemblyNumInputs;`。
- **L453 EN**: Provides textual content or support data: `clang_Cursor_getGCCAssemblyInput;`.
  **L453 CN**: 提供文本内容或支持数据：`clang_Cursor_getGCCAssemblyInput;`。
- **L454 EN**: Provides textual content or support data: `clang_Cursor_getGCCAssemblyOutput;`.
  **L454 CN**: 提供文本内容或支持数据：`clang_Cursor_getGCCAssemblyOutput;`。
- **L455 EN**: Provides textual content or support data: `clang_Cursor_getGCCAssemblyNumClobbers;`.
  **L455 CN**: 提供文本内容或支持数据：`clang_Cursor_getGCCAssemblyNumClobbers;`。
- **L456 EN**: Provides textual content or support data: `clang_Cursor_getGCCAssemblyClobber;`.
  **L456 CN**: 提供文本内容或支持数据：`clang_Cursor_getGCCAssemblyClobber;`。
- **L457 EN**: Provides textual content or support data: `clang_Cursor_isGCCAssemblyVolatile;`.
  **L457 CN**: 提供文本内容或支持数据：`clang_Cursor_isGCCAssemblyVolatile;`。
- **L458 EN**: Provides textual content or support data: `};`.
  **L458 CN**: 提供文本内容或支持数据：`};`。
- **L459 EN**: Blank line separating nearby declarations or logic blocks.
  **L459 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L460 EN**: Provides textual content or support data: `LLVM_23 {`.
  **L460 CN**: 提供文本内容或支持数据：`LLVM_23 {`。
- **L461 EN**: Introduces a labeled text section: `global:`.
  **L461 CN**: 引入一个带标签的文本段落：`global:`。
- **L462 EN**: Provides textual content or support data: `clang_ModuleCache_prune;`.
  **L462 CN**: 提供文本内容或支持数据：`clang_ModuleCache_prune;`。
- **L463 EN**: Provides textual content or support data: `};`.
  **L463 CN**: 提供文本内容或支持数据：`};`。
- **L464 EN**: Blank line separating nearby declarations or logic blocks.
  **L464 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L465 EN**: Defines a heading or emphasized marker: `# Example of how to add a new symbol version entry. If you do add a new symbol`.
  **L465 CN**: 定义一个标题或强调标记：`# Example of how to add a new symbol version entry. If you do add a new symbol`。
- **L466 EN**: Defines a heading or emphasized marker: `# version, please update the example to depend on the version you added.`.
  **L466 CN**: 定义一个标题或强调标记：`# version, please update the example to depend on the version you added.`。
- **L467 EN**: Defines a heading or emphasized marker: `# LLVM_X {`.
  **L467 CN**: 定义一个标题或强调标记：`# LLVM_X {`。
- **L468 EN**: Defines a heading or emphasized marker: `# global:`.
  **L468 CN**: 定义一个标题或强调标记：`# global:`。

### Lines 469-470

````text
#   clang_newsymbol;
# };
````
- **L469 EN**: Defines a heading or emphasized marker: `# clang_newsymbol;`.
  **L469 CN**: 定义一个标题或强调标记：`# clang_newsymbol;`。
- **L470 EN**: Defines a heading or emphasized marker: `# };`.
  **L470 CN**: 定义一个标题或强调标记：`# };`。

## Key Concepts / 关键概念

- **Clang tooling entry points / Clang 工具入口**:
  - **EN**: Shows how a command-line tool, helper script, or support asset plugs into Clang workflows.
  - **CN**: 展示命令行工具、辅助脚本或支持资源如何接入 Clang 工作流。
- **libclang C API / libclang C API**:
  - **EN**: Provides stable C-facing access to Clang parsing, indexing, and diagnostics.
  - **CN**: 提供面向 C 的稳定接口以访问 Clang 的解析、索引与诊断能力。
- **Source formatting / 源码格式化**:
  - **EN**: Normalizes source layout according to formatting policies and style rules.
  - **CN**: 按照格式策略与风格规则规范化源码布局。
- **Diagnostics / 诊断机制**:
  - **EN**: Surfaces diagnostic identifiers, categories, and reporting behavior.
  - **CN**: 呈现诊断 ID、类别以及报告行为。
- **Interactive compilation / 交互式编译**:
  - **EN**: Supports incremental parsing or execution in a REPL-style workflow.
  - **CN**: 支持 REPL 风格工作流中的增量解析或执行。
- **Protocol schemas / 协议模式**:
  - **EN**: Defines protobuf-based representations used to exchange structured test or fuzz data.
  - **CN**: 定义用于交换结构化测试或模糊数据的 protobuf 表示。

## Dependencies / 依赖关系

- **Local context / 本地上下文**: This file depends mainly on nearby tool-specific logic or assets. / 该文件主要依赖附近的工具专用逻辑或资源。
