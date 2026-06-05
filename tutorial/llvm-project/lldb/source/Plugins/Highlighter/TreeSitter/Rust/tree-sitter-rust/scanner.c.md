# scanner.c — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `lldb/source/Plugins/Highlighter/TreeSitter/Rust/tree-sitter-rust/scanner.c`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: Implements the LLDB support logic associated with `scanner`.
  - **CN**: 实现与 `scanner` 相关的 LLDB 支持逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-18
```c
#include "tree_sitter/alloc.h"
#include "tree_sitter/parser.h"

#include <wctype.h>

enum TokenType {
    STRING_CONTENT,
    RAW_STRING_LITERAL_START,
    RAW_STRING_LITERAL_CONTENT,
    RAW_STRING_LITERAL_END,
    FLOAT_LITERAL,
    BLOCK_OUTER_DOC_MARKER,
    BLOCK_INNER_DOC_MARKER,
    BLOCK_COMMENT_CONTENT,
    LINE_DOC_CONTENT,
    ERROR_SENTINEL
};

```
- **EN**: Pulls in the headers needed by this translation unit, including `tree_sitter/alloc.h`, `tree_sitter/parser.h`, `wctype.h`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `tree_sitter/alloc.h`, `tree_sitter/parser.h`, `wctype.h`。

### Lines 19-32
```c
typedef struct {
    uint8_t opening_hash_count;
} Scanner;

void *tree_sitter_rust_external_scanner_create() { return ts_calloc(1, sizeof(Scanner)); }

void tree_sitter_rust_external_scanner_destroy(void *payload) { ts_free((Scanner *)payload); }

unsigned tree_sitter_rust_external_scanner_serialize(void *payload, char *buffer) {
    Scanner *scanner = (Scanner *)payload;
    buffer[0] = (char)scanner->opening_hash_count;
    return 1;
}

```
- **EN**: Implements logic around `tree_sitter_rust_external_scanner_create`, `tree_sitter_rust_external_scanner_destroy`, `tree_sitter_rust_external_scanner_serialize`.
- **CN**: 围绕 `tree_sitter_rust_external_scanner_create`, `tree_sitter_rust_external_scanner_destroy`, `tree_sitter_rust_external_scanner_serialize` 实现具体逻辑。

### Lines 33-43
```c
void tree_sitter_rust_external_scanner_deserialize(void *payload, const char *buffer, unsigned length) {
    Scanner *scanner = (Scanner *)payload;
    scanner->opening_hash_count = 0;
    if (length == 1) {
        Scanner *scanner = (Scanner *)payload;
        scanner->opening_hash_count = buffer[0];
    }
}

static inline bool is_num_char(int32_t c) { return c == '_' || iswdigit(c); }

```
- **EN**: Implements logic around `tree_sitter_rust_external_scanner_deserialize`, `is_num_char`.
- **CN**: 围绕 `tree_sitter_rust_external_scanner_deserialize`, `is_num_char` 实现具体逻辑。

### Lines 44-63
```c
static inline void advance(TSLexer *lexer) { lexer->advance(lexer, false); }

static inline void skip(TSLexer *lexer) { lexer->advance(lexer, true); }

static inline bool process_string(TSLexer *lexer) {
    bool has_content = false;
    for (;;) {
        if (lexer->lookahead == '\"' || lexer->lookahead == '\\') {
            break;
        }
        if (lexer->eof(lexer)) {
            return false;
        }
        has_content = true;
        advance(lexer);
    }
    lexer->result_symbol = STRING_CONTENT;
    lexer->mark_end(lexer);
    return has_content;
}
```
- **EN**: Implements logic around `advance`, `skip`, `process_string`, `eof`, and 1 more symbols.
- **CN**: 围绕 `advance`, `skip`, `process_string`, `eof`, and 1 more symbols 实现具体逻辑。

### Lines 64-73
```c

static inline bool scan_raw_string_start(Scanner *scanner, TSLexer *lexer) {
    if (lexer->lookahead == 'b' || lexer->lookahead == 'c') {
        advance(lexer);
    }
    if (lexer->lookahead != 'r') {
        return false;
    }
    advance(lexer);

```
- **EN**: Implements logic around `scan_raw_string_start`, `advance`.
- **CN**: 围绕 `scan_raw_string_start`, `advance` 实现具体逻辑。

### Lines 74-85
```c
    uint8_t opening_hash_count = 0;
    while (lexer->lookahead == '#') {
        advance(lexer);
        opening_hash_count++;
    }

    if (lexer->lookahead != '"') {
        return false;
    }
    advance(lexer);
    scanner->opening_hash_count = opening_hash_count;

```
- **EN**: Implements logic around `advance`.
- **CN**: 围绕 `advance` 实现具体逻辑。

### Lines 86-105
```c
    lexer->result_symbol = RAW_STRING_LITERAL_START;
    return true;
}

static inline bool scan_raw_string_content(Scanner *scanner, TSLexer *lexer) {
    for (;;) {
        if (lexer->eof(lexer)) {
            return false;
        }
        if (lexer->lookahead == '"') {
            lexer->mark_end(lexer);
            advance(lexer);
            unsigned hash_count = 0;
            while (lexer->lookahead == '#' && hash_count < scanner->opening_hash_count) {
                advance(lexer);
                hash_count++;
            }
            if (hash_count == scanner->opening_hash_count) {
                lexer->result_symbol = RAW_STRING_LITERAL_CONTENT;
                return true;
```
- **EN**: Implements logic around `scan_raw_string_content`, `eof`, `mark_end`, `advance`.
- **CN**: 围绕 `scan_raw_string_content`, `eof`, `mark_end`, `advance` 实现具体逻辑。

### Lines 106-121
```c
            }
        } else {
            advance(lexer);
        }
    }
}

static inline bool scan_raw_string_end(Scanner *scanner, TSLexer *lexer) {
    advance(lexer);
    for (unsigned i = 0; i < scanner->opening_hash_count; i++) {
        advance(lexer);
    }
    lexer->result_symbol = RAW_STRING_LITERAL_END;
    return true;
}

```
- **EN**: Implements logic around `advance`, `scan_raw_string_end`.
- **CN**: 围绕 `advance`, `scan_raw_string_end` 实现具体逻辑。

### Lines 122-131
```c
static inline bool process_float_literal(TSLexer *lexer) {
    lexer->result_symbol = FLOAT_LITERAL;

    advance(lexer);
    while (is_num_char(lexer->lookahead)) {
        advance(lexer);
    }

    bool has_fraction = false, has_exponent = false;

```
- **EN**: Implements logic around `process_float_literal`, `advance`, `is_num_char`.
- **CN**: 围绕 `process_float_literal`, `advance`, `is_num_char` 实现具体逻辑。

### Lines 132-147
```c
    if (lexer->lookahead == '.') {
        has_fraction = true;
        advance(lexer);
        if (iswalpha(lexer->lookahead)) {
            // The dot is followed by a letter: 1.max(2) => not a float but an integer
            return false;
        }

        if (lexer->lookahead == '.') {
            return false;
        }
        while (is_num_char(lexer->lookahead)) {
            advance(lexer);
        }
    }

```
- **EN**: Implements logic around `advance`, `iswalpha`, `is_num_char`.
- **CN**: 围绕 `advance`, `iswalpha`, `is_num_char` 实现具体逻辑。

### Lines 148-163
```c
    lexer->mark_end(lexer);

    if (lexer->lookahead == 'e' || lexer->lookahead == 'E') {
        has_exponent = true;
        advance(lexer);
        if (lexer->lookahead == '+' || lexer->lookahead == '-') {
            advance(lexer);
        }
        if (!is_num_char(lexer->lookahead)) {
            return true;
        }
        advance(lexer);
        while (is_num_char(lexer->lookahead)) {
            advance(lexer);
        }

```
- **EN**: Implements logic around `mark_end`, `advance`, `is_num_char`.
- **CN**: 围绕 `mark_end`, `advance`, `is_num_char` 实现具体逻辑。

### Lines 164-178
```c
        lexer->mark_end(lexer);
    }

    if (!has_exponent && !has_fraction) {
        return false;
    }

    if (lexer->lookahead != 'u' && lexer->lookahead != 'i' && lexer->lookahead != 'f') {
        return true;
    }
    advance(lexer);
    if (!iswdigit(lexer->lookahead)) {
        return true;
    }

```
- **EN**: Implements logic around `mark_end`, `advance`, `iswdigit`.
- **CN**: 围绕 `mark_end`, `advance`, `iswdigit` 实现具体逻辑。

### Lines 179-198
```c
    while (iswdigit(lexer->lookahead)) {
        advance(lexer);
    }

    lexer->mark_end(lexer);
    return true;
}

static inline bool process_line_doc_content(TSLexer *lexer) {
    lexer->result_symbol = LINE_DOC_CONTENT;
    for (;;) {
        if (lexer->eof(lexer)) {
            return true;
        }
        if (lexer->lookahead == '\n') {
            // Include the newline in the doc content node.
            // Line endings are useful for markdown injection.
            advance(lexer);
            return true;
        }
```
- **EN**: Implements logic around `iswdigit`, `advance`, `mark_end`, `process_line_doc_content`, and 1 more symbols.
- **CN**: 围绕 `iswdigit`, `advance`, `mark_end`, `process_line_doc_content`, and 1 more symbols 实现具体逻辑。

### Lines 199-208
```c
        advance(lexer);
    }
}

typedef enum {
    LeftForwardSlash,
    LeftAsterisk,
    Continuing,
} BlockCommentState;

```
- **EN**: Implements logic around `advance`.
- **CN**: 围绕 `advance` 实现具体逻辑。

### Lines 209-220
```c
typedef struct {
    BlockCommentState state;
    unsigned nestingDepth;
} BlockCommentProcessing;

static inline void process_left_forward_slash(BlockCommentProcessing *processing, char current) {
    if (current == '*') {
        processing->nestingDepth += 1;
    }
    processing->state = Continuing;
};

```
- **EN**: Implements logic around `process_left_forward_slash`.
- **CN**: 围绕 `process_left_forward_slash` 实现具体逻辑。

### Lines 221-231
```c
static inline void process_left_asterisk(BlockCommentProcessing *processing, char current, TSLexer *lexer) {
    if (current == '*') {
        lexer->mark_end(lexer);
        processing->state = LeftAsterisk;
        return;
    }

    if (current == '/') {
        processing->nestingDepth -= 1;
    }

```
- **EN**: Implements logic around `process_left_asterisk`, `mark_end`.
- **CN**: 围绕 `process_left_asterisk`, `mark_end` 实现具体逻辑。

### Lines 232-245
```c
    processing->state = Continuing;
}

static inline void process_continuing(BlockCommentProcessing *processing, char current) {
    switch (current) {
        case '/':
            processing->state = LeftForwardSlash;
            break;
        case '*':
            processing->state = LeftAsterisk;
            break;
    }
}

```
- **EN**: Implements logic around `process_continuing`.
- **CN**: 围绕 `process_continuing` 实现具体逻辑。

### Lines 246-265
```c
static inline bool process_block_comment(TSLexer *lexer, const bool *valid_symbols) {
    char first = (char)lexer->lookahead;
    // The first character is stored so we can safely advance inside
    // these if blocks. However, because we only store one, we can only
    // safely advance 1 time. Since there's a chance that an advance could
    // happen in one state, we must advance in all states to ensure that
    // the program ends up in a sane state prior to processing the block
    // comment if need be.
    if (valid_symbols[BLOCK_INNER_DOC_MARKER] && first == '!') {
        lexer->result_symbol = BLOCK_INNER_DOC_MARKER;
        advance(lexer);
        return true;
    }
    if (valid_symbols[BLOCK_OUTER_DOC_MARKER] && first == '*') {
        advance(lexer);
        lexer->mark_end(lexer);
        // If the next token is a / that means that it's an empty block comment.
        if (lexer->lookahead == '/') {
            return false;
        }
```
- **EN**: Implements logic around `process_block_comment`, `advance`, `mark_end`.
- **CN**: 围绕 `process_block_comment`, `advance`, `mark_end` 实现具体逻辑。

### Lines 266-275
```c
        // If the next token is a * that means that this isn't a BLOCK_OUTER_DOC_MARKER
        // as BLOCK_OUTER_DOC_MARKER's only have 2 * not 3 or more.
        if (lexer->lookahead != '*') {
            lexer->result_symbol = BLOCK_OUTER_DOC_MARKER;
            return true;
        }
    } else {
        advance(lexer);
    }

```
- **EN**: Implements logic around `advance`.
- **CN**: 围绕 `advance` 实现具体逻辑。

### Lines 276-295
```c
    if (valid_symbols[BLOCK_COMMENT_CONTENT]) {
        BlockCommentProcessing processing = {Continuing, 1};
        // Manually set the current state based on the first character
        switch (first) {
            case '*':
                processing.state = LeftAsterisk;
                if (lexer->lookahead == '/') {
                    // This case can happen in an empty doc block comment
                    // like /*!*/. The comment has no contents, so bail.
                    return false;
                }
                break;
            case '/':
                processing.state = LeftForwardSlash;
                break;
            default:
                processing.state = Continuing;
                break;
        }

```
- **EN**: Finalizes a local computation, invariant check, or helper decision.
- **CN**: 完成局部计算、不变式检查或辅助决策。

### Lines 296-315
```c
        // For the purposes of actually parsing rust code, this
        // is incorrect as it considers an unterminated block comment
        // to be an error. However, for the purposes of syntax highlighting
        // this should be considered successful as otherwise you are not able
        // to syntax highlight a block of code prior to closing the
        // block comment
        while (!lexer->eof(lexer) && processing.nestingDepth != 0) {
            // Set first to the current lookahead as that is the second character
            // as we force an advance in the above code when we are checking if we
            // need to handle a block comment inner or outer doc comment signifier
            // node
            first = (char)lexer->lookahead;
            switch (processing.state) {
                case LeftForwardSlash:
                    process_left_forward_slash(&processing, first);
                    break;
                case LeftAsterisk:
                    process_left_asterisk(&processing, first, lexer);
                    break;
                case Continuing:
```
- **EN**: Implements logic around `eof`, `process_left_forward_slash`, `process_left_asterisk`; this block propagates recoverable errors, status objects, or diagnostics.
- **CN**: 围绕 `eof`, `process_left_forward_slash`, `process_left_asterisk` 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息。

### Lines 316-330
```c
                    lexer->mark_end(lexer);
                    process_continuing(&processing, first);
                    break;
                default:
                    break;
            }
            advance(lexer);
            if (first == '/' && processing.nestingDepth != 0) {
                lexer->mark_end(lexer);
            }
        }
        lexer->result_symbol = BLOCK_COMMENT_CONTENT;
        return true;
    }

```
- **EN**: Implements logic around `mark_end`, `process_continuing`, `advance`.
- **CN**: 围绕 `mark_end`, `process_continuing`, `advance` 实现具体逻辑。

### Lines 331-346
```c
    return false;
}

bool tree_sitter_rust_external_scanner_scan(void *payload, TSLexer *lexer, const bool *valid_symbols) {
    // The documentation states that if the lexical analysis fails for some reason
    // they will mark every state as valid and pass it to the external scanner
    // However, we can't do anything to help them recover in that case so we
    // should just fail.
    /*
      link: https://tree-sitter.github.io/tree-sitter/creating-parsers#external-scanners
      If a syntax error is encountered during regular parsing, Tree-sitter’s
      first action during error recovery will be to call the external scanner’s
      scan function with all tokens marked valid. The scanner should detect this
      case and handle it appropriately. One simple method of detection is to add
      an unused token to the end of the externals array, for example

```
- **EN**: Implements logic around `tree_sitter_rust_external_scanner_scan`; this block propagates recoverable errors, status objects, or diagnostics; supports documentation parsing, directives, or build-time rendering.
- **CN**: 围绕 `tree_sitter_rust_external_scanner_scan` 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息，并支持文档解析、指令或构建期渲染。

### Lines 347-357
```c
      externals: $ => [$.token1, $.token2, $.error_sentinel],

      then check whether that token is marked valid to determine whether
      Tree-sitter is in error correction mode.
    */
    if (valid_symbols[ERROR_SENTINEL]) {
        return false;
    }

    Scanner *scanner = (Scanner *)payload;

```
- **EN**: Finalizes a local computation, invariant check, or helper decision.
- **CN**: 完成局部计算、不变式检查或辅助决策。

### Lines 358-370
```c
    if (valid_symbols[BLOCK_COMMENT_CONTENT] || valid_symbols[BLOCK_INNER_DOC_MARKER] ||
        valid_symbols[BLOCK_OUTER_DOC_MARKER]) {
        return process_block_comment(lexer, valid_symbols);
    }

    if (valid_symbols[STRING_CONTENT] && !valid_symbols[FLOAT_LITERAL]) {
        return process_string(lexer);
    }

    if (valid_symbols[LINE_DOC_CONTENT]) {
        return process_line_doc_content(lexer);
    }

```
- **EN**: Implements logic around `process_block_comment`, `process_string`, `process_line_doc_content`.
- **CN**: 围绕 `process_block_comment`, `process_string`, `process_line_doc_content` 实现具体逻辑。

### Lines 371-383
```c
    while (iswspace(lexer->lookahead)) {
        skip(lexer);
    }

    if (valid_symbols[RAW_STRING_LITERAL_START] &&
        (lexer->lookahead == 'r' || lexer->lookahead == 'b' || lexer->lookahead == 'c')) {
        return scan_raw_string_start(scanner, lexer);
    }

    if (valid_symbols[RAW_STRING_LITERAL_CONTENT]) {
        return scan_raw_string_content(scanner, lexer);
    }

```
- **EN**: Implements logic around `iswspace`, `skip`, `scan_raw_string_start`, `scan_raw_string_content`.
- **CN**: 围绕 `iswspace`, `skip`, `scan_raw_string_start`, `scan_raw_string_content` 实现具体逻辑。

### Lines 384-393
```c
    if (valid_symbols[RAW_STRING_LITERAL_END] && lexer->lookahead == '"') {
        return scan_raw_string_end(scanner, lexer);
    }

    if (valid_symbols[FLOAT_LITERAL] && iswdigit(lexer->lookahead)) {
        return process_float_literal(lexer);
    }

    return false;
}
```
- **EN**: Implements logic around `scan_raw_string_end`, `iswdigit`, `process_float_literal`.
- **CN**: 围绕 `scan_raw_string_end`, `iswdigit`, `process_float_literal` 实现具体逻辑。

## Key Concepts / 关键概念

- **LLDB architecture / LLDB 架构**:
  - **EN**: Explains how LLDB organizes debugger APIs, runtime objects, and platform-specific helpers.
  - **CN**: 说明 LLDB 如何组织调试器 API、运行时对象以及平台相关辅助组件。
- **Symbolication / 符号化**:
  - **EN**: Relates raw addresses back to modules, sections, symbols, and source locations.
  - **CN**: 将原始地址关联回模块、节区、符号与源码位置。
- **Runtime control / 运行时控制**:
  - **EN**: Models debugger ownership over targets, processes, threads, and event delivery.
  - **CN**: 建模调试器对目标、进程、线程以及事件投递的控制。

## Dependencies / 依赖关系

- **Direct LLDB/LLVM includes / 直接的 LLDB/LLVM 包含**: `tree_sitter/alloc.h`, `tree_sitter/parser.h`
- **Standard-library headers / 标准库头文件**: `<wctype.h>`
