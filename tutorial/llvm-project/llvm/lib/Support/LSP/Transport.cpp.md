# Transport.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `llvm/lib/Support/LSP/Transport.cpp`
- Repository: `llvm-project`
- Purpose (EN): Reply
- Purpose (CN): 该文件位于 LLVM 的 `Support/LSP` 目录中，主要实现与 `Transport` 相关的接口、数据结构和辅助逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-40

```cpp
//===--- JSONTransport.cpp - sending and receiving LSP messages over JSON -===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#include "llvm/Support/LSP/Transport.h"
#include "llvm/ADT/SmallString.h"
#include "llvm/Support/Error.h"
#include "llvm/Support/LSP/Logging.h"
#include "llvm/Support/LSP/Protocol.h"
#include <atomic>
#include <optional>
#include <system_error>
#include <utility>

using namespace llvm;
using namespace llvm::lsp;

//===----------------------------------------------------------------------===//
// Reply
//===----------------------------------------------------------------------===//

namespace {
/// Function object to reply to an LSP call.
/// Each instance must be called exactly once, otherwise:
///  - if there was no reply, an error reply is sent
///  - if there were multiple replies, only the first is sent
class Reply {
public:
  Reply(const llvm::json::Value &Id, StringRef Method, JSONTransport &Transport,
        std::mutex &TransportOutputMutex);
  Reply(Reply &&Other);
  Reply &operator=(Reply &&) = delete;
  Reply(const Reply &) = delete;
  Reply &operator=(const Reply &) = delete;

  void operator()(llvm::Expected<llvm::json::Value> Reply);
```
- EN: Introduces the file banner, comments, and the initial setup for this translation unit.
  CN: 这一段给出文件横幅、注释说明以及该编译单元的初始设置。
- EN: Brings in 9 direct dependencies, including `llvm/Support/LSP/Transport.h`, `llvm/ADT/SmallString.h`, `llvm/Support/Error.h`, `llvm/Support/LSP/Logging.h`.
  CN: 引入了 9 个直接依赖，其中包括 `llvm/Support/LSP/Transport.h`, `llvm/ADT/SmallString.h`, `llvm/Support/Error.h`, `llvm/Support/LSP/Logging.h`。
- EN: This section centers on `Reply`, `operator` and implements the local control flow and bookkeeping.
  CN: 这一段主要围绕 `Reply`, `operator` 等符号展开，负责实现局部控制流程与状态维护。

### Lines 41-80

```cpp

private:
  std::string Method;
  std::atomic<bool> Replied = {false};
  llvm::json::Value Id;
  JSONTransport *Transport;
  std::mutex &TransportOutputMutex;
};
} // namespace

Reply::Reply(const llvm::json::Value &Id, llvm::StringRef Method,
             JSONTransport &Transport, std::mutex &TransportOutputMutex)
    : Method(Method), Id(Id), Transport(&Transport),
      TransportOutputMutex(TransportOutputMutex) {}

Reply::Reply(Reply &&Other)
    : Method(Other.Method), Replied(Other.Replied.load()),
      Id(std::move(Other.Id)), Transport(Other.Transport),
      TransportOutputMutex(Other.TransportOutputMutex) {
  Other.Transport = nullptr;
}

void Reply::operator()(llvm::Expected<llvm::json::Value> Reply) {
  if (Replied.exchange(true)) {
    Logger::error("Replied twice to message {0}({1})", Method, Id);
    assert(false && "must reply to each call only once!");
    return;
  }
  assert(Transport && "expected valid transport to reply to");

  std::lock_guard<std::mutex> TransportLock(TransportOutputMutex);
  if (Reply) {
    Logger::info("--> reply:{0}({1})", Method, Id);
    Transport->reply(std::move(Id), std::move(Reply));
  } else {
    llvm::Error Error = Reply.takeError();
    Logger::info("--> reply:{0}({1}): {2}", Method, Id, Error);
    Transport->reply(std::move(Id), std::move(Error));
  }
}
```
- EN: This section centers on `Reply`, `operator`, `assert` and implements the local control flow and bookkeeping.
  CN: 这一段主要围绕 `Reply`, `operator`, `assert` 等符号展开，负责实现局部控制流程与状态维护。
- EN: In this range, the code checks conditions and handles edge cases and propagates LLVM-style errors and invariants.
  CN: 在这一段中，代码检查条件并处理边界情况，并传播 LLVM 风格的错误处理并维护不变量。

### Lines 81-120

```cpp

//===----------------------------------------------------------------------===//
// MessageHandler
//===----------------------------------------------------------------------===//

bool MessageHandler::onNotify(llvm::StringRef Method, llvm::json::Value Value) {
  Logger::info("--> {0}", Method);

  if (Method == "exit")
    return false;
  if (Method == "$cancel") {
    // TODO: Add support for cancelling requests.
  } else {
    auto It = NotificationHandlers.find(Method);
    if (It != NotificationHandlers.end())
      It->second(std::move(Value));
  }
  return true;
}

bool MessageHandler::onCall(llvm::StringRef Method, llvm::json::Value Params,
                            llvm::json::Value Id) {
  Logger::info("--> {0}({1})", Method, Id);

  Reply Reply(Id, Method, Transport, TransportOutputMutex);

  auto It = MethodHandlers.find(Method);
  if (It != MethodHandlers.end()) {
    It->second(std::move(Params), std::move(Reply));
  } else {
    Reply(llvm::make_error<LSPError>("method not found: " + Method.str(),
                                     ErrorCode::MethodNotFound));
  }
  return true;
}

bool MessageHandler::onReply(llvm::json::Value Id,
                             llvm::Expected<llvm::json::Value> Result) {
  // Find the response handler in the mapping. If it exists, move it out of the
  // mapping and erase it.
```
- EN: This section centers on `onNotify`, `onCall`, `Reply` and implements the local control flow and bookkeeping.
  CN: 这一段主要围绕 `onNotify`, `onCall`, `Reply` 等符号展开，负责实现局部控制流程与状态维护。
- EN: In this range, the code checks conditions and handles edge cases and propagates LLVM-style errors and invariants.
  CN: 在这一段中，代码检查条件并处理边界情况，并传播 LLVM 风格的错误处理并维护不变量。

### Lines 121-160

```cpp
  ResponseHandlerTy ResponseHandler;
  {
    std::lock_guard<std::mutex> responseHandlersLock(ResponseHandlersMutex);
    auto It = ResponseHandlers.find(debugString(Id));
    if (It != ResponseHandlers.end()) {
      ResponseHandler = std::move(It->second);
      ResponseHandlers.erase(It);
    }
  }

  // If we found a response handler, invoke it. Otherwise, log an error.
  if (ResponseHandler.second) {
    Logger::info("--> reply:{0}({1})", ResponseHandler.first, Id);
    ResponseHandler.second(std::move(Id), std::move(Result));
  } else {
    Logger::error(
        "received a reply with ID {0}, but there was no such outgoing request",
        Id);
    if (!Result)
      llvm::consumeError(Result.takeError());
  }
  return true;
}

//===----------------------------------------------------------------------===//
// JSONTransport
//===----------------------------------------------------------------------===//

/// Encode the given error as a JSON object.
static llvm::json::Object encodeError(llvm::Error Error) {
  std::string Message;
  ErrorCode Code = ErrorCode::UnknownErrorCode;
  auto HandlerFn = [&](const LSPError &LspError) -> llvm::Error {
    Message = LspError.message;
    Code = LspError.code;
    return llvm::Error::success();
  };
  if (llvm::Error Unhandled = llvm::handleErrors(std::move(Error), HandlerFn))
    Message = llvm::toString(std::move(Unhandled));

```
- EN: This section centers on `responseHandlersLock`, `encodeError`, `success` and implements the local control flow and bookkeeping.
  CN: 这一段主要围绕 `responseHandlersLock`, `encodeError`, `success` 等符号展开，负责实现局部控制流程与状态维护。
- EN: In this range, the code checks conditions and handles edge cases and propagates LLVM-style errors and invariants.
  CN: 在这一段中，代码检查条件并处理边界情况，并传播 LLVM 风格的错误处理并维护不变量。

### Lines 161-200

```cpp
  return llvm::json::Object{
      {"message", std::move(Message)},
      {"code", int64_t(Code)},
  };
}

/// Decode the given JSON object into an error.
llvm::Error decodeError(const llvm::json::Object &O) {
  StringRef Msg = O.getString("message").value_or("Unspecified error");
  if (std::optional<int64_t> Code = O.getInteger("code"))
    return llvm::make_error<LSPError>(Msg.str(), ErrorCode(*Code));
  return llvm::make_error<llvm::StringError>(llvm::inconvertibleErrorCode(),
                                             Msg.str());
}

void JSONTransport::notify(StringRef Method, llvm::json::Value Params) {
  sendMessage(llvm::json::Object{
      {"jsonrpc", "2.0"},
      {"method", Method},
      {"params", std::move(Params)},
  });
}
void JSONTransport::call(StringRef Method, llvm::json::Value Params,
                         llvm::json::Value Id) {
  sendMessage(llvm::json::Object{
      {"jsonrpc", "2.0"},
      {"id", std::move(Id)},
      {"method", Method},
      {"params", std::move(Params)},
  });
}
void JSONTransport::reply(llvm::json::Value Id,
                          llvm::Expected<llvm::json::Value> Result) {
  if (Result) {
    return sendMessage(llvm::json::Object{
        {"jsonrpc", "2.0"},
        {"id", std::move(Id)},
        {"result", std::move(*Result)},
    });
  }
```
- EN: This section centers on `decodeError`, `notify`, `call` and parses input and converts raw data into structured form.
  CN: 这一段主要围绕 `decodeError`, `notify`, `call` 等符号展开，负责解析输入并把原始数据转换成结构化形式。
- EN: In this range, the code checks conditions and handles edge cases and propagates LLVM-style errors and invariants.
  CN: 在这一段中，代码检查条件并处理边界情况，并传播 LLVM 风格的错误处理并维护不变量。

### Lines 201-240

```cpp

  sendMessage(llvm::json::Object{
      {"jsonrpc", "2.0"},
      {"id", std::move(Id)},
      {"error", encodeError(Result.takeError())},
  });
}

llvm::Error JSONTransport::run(MessageHandler &Handler) {
  std::string Json;
  while (!In->isEndOfInput()) {
    if (In->hasError()) {
      return llvm::errorCodeToError(
          std::error_code(errno, std::system_category()));
    }

    if (succeeded(In->readMessage(Json))) {
      if (llvm::Expected<llvm::json::Value> Doc = llvm::json::parse(Json)) {
        if (!handleMessage(std::move(*Doc), Handler))
          return llvm::Error::success();
      } else {
        Logger::error("JSON parse error: {0}", llvm::toString(Doc.takeError()));
      }
    }
  }
  return llvm::errorCodeToError(std::make_error_code(std::errc::io_error));
}

void JSONTransport::sendMessage(llvm::json::Value Msg) {
  OutputBuffer.clear();
  llvm::raw_svector_ostream os(OutputBuffer);
  os << llvm::formatv(PrettyOutput ? "{0:2}\n" : "{0}", Msg);
  Out << "Content-Length: " << OutputBuffer.size() << "\r\n\r\n"
      << OutputBuffer;
  Out.flush();
  Logger::debug(">>> {0}\n", OutputBuffer);
}

bool JSONTransport::handleMessage(llvm::json::Value Msg,
                                  MessageHandler &Handler) {
```
- EN: This section centers on `run`, `errorCodeToError`, `sendMessage` and implements the local control flow and bookkeeping.
  CN: 这一段主要围绕 `run`, `errorCodeToError`, `sendMessage` 等符号展开，负责实现局部控制流程与状态维护。
- EN: In this range, the code iterates over collections, ranges, or records and checks conditions and handles edge cases.
  CN: 在这一段中，代码遍历集合、区间或记录，并检查条件并处理边界情况。

### Lines 241-280

```cpp
  // Message must be an object with "jsonrpc":"2.0".
  llvm::json::Object *Object = Msg.getAsObject();
  if (!Object ||
      Object->getString("jsonrpc") != std::optional<StringRef>("2.0"))
    return false;

  // `id` may be any JSON value. If absent, this is a notification.
  std::optional<llvm::json::Value> Id;
  if (llvm::json::Value *I = Object->get("id"))
    Id = std::move(*I);
  std::optional<StringRef> Method = Object->getString("method");

  // This is a response.
  if (!Method) {
    if (!Id)
      return false;
    if (auto *Err = Object->getObject("error"))
      return Handler.onReply(std::move(*Id), decodeError(*Err));
    // result should be given, use null if not.
    llvm::json::Value Result = nullptr;
    if (llvm::json::Value *R = Object->get("result"))
      Result = std::move(*R);
    return Handler.onReply(std::move(*Id), std::move(Result));
  }

  // Params should be given, use null if not.
  llvm::json::Value Params = nullptr;
  if (llvm::json::Value *P = Object->get("params"))
    Params = std::move(*P);

  if (Id)
    return Handler.onCall(*Method, std::move(Params), std::move(*Id));
  return Handler.onNotify(*Method, std::move(Params));
}

/// Tries to read a line up to and including \n.
/// If failing, feof(), ferror(), or shutdownRequested() will be set.
LogicalResult readLine(std::FILE *In, SmallVectorImpl<char> &Out) {
  // Big enough to hold any reasonable header line. May not fit content lines
  // in delimited mode, but performance doesn't matter for that mode.
```
- EN: This section centers on `readLine` and parses input and converts raw data into structured form.
  CN: 这一段主要围绕 `readLine` 等符号展开，负责解析输入并把原始数据转换成结构化形式。
- EN: In this range, the code checks conditions and handles edge cases and propagates LLVM-style errors and invariants.
  CN: 在这一段中，代码检查条件并处理边界情况，并传播 LLVM 风格的错误处理并维护不变量。

### Lines 281-320

```cpp
  static constexpr int BufSize = 128;
  size_t Size = 0;
  Out.clear();
  for (;;) {
    Out.resize_for_overwrite(Size + BufSize);
    if (!std::fgets(&Out[Size], BufSize, In))
      return failure();

    clearerr(In);

    // If the line contained null bytes, anything after it (including \n) will
    // be ignored. Fortunately this is not a legal header or JSON.
    size_t Read = std::strlen(&Out[Size]);
    if (Read > 0 && Out[Size + Read - 1] == '\n') {
      Out.resize(Size + Read);
      return success();
    }
    Size += Read;
  }
}

// Returns std::nullopt when:
//  - ferror(), feof(), or shutdownRequested() are set.
//  - Content-Length is missing or empty (protocol error)
LogicalResult
JSONTransportInputOverFile::readStandardMessage(std::string &Json) {
  // A Language Server Protocol message starts with a set of HTTP headers,
  // delimited  by \r\n, and terminated by an empty line (\r\n).
  unsigned long long ContentLength = 0;
  llvm::SmallString<128> Line;
  while (true) {
    if (feof(In) || hasError() || failed(readLine(In, Line)))
      return failure();

    // Content-Length is a mandatory header, and the only one we handle.
    StringRef LineRef = Line;
    if (LineRef.consume_front("Content-Length: ")) {
      llvm::getAsUnsignedInteger(LineRef.trim(), 0, ContentLength);
    } else if (!LineRef.trim().empty()) {
      // It's another header, ignore it.
```
- EN: This section centers on `clearerr`, `success`, `readStandardMessage` and parses input and converts raw data into structured form.
  CN: 这一段主要围绕 `clearerr`, `success`, `readStandardMessage` 等符号展开，负责解析输入并把原始数据转换成结构化形式。
- EN: In this range, the code iterates over collections, ranges, or records and checks conditions and handles edge cases.
  CN: 在这一段中，代码遍历集合、区间或记录，并检查条件并处理边界情况。

### Lines 321-360

```cpp
      continue;
    } else {
      // An empty line indicates the end of headers. Go ahead and read the JSON.
      break;
    }
  }

  // The fuzzer likes crashing us by sending "Content-Length: 9999999999999999"
  if (ContentLength == 0 || ContentLength > 1 << 30)
    return failure();

  Json.resize(ContentLength);
  for (size_t Pos = 0, Read; Pos < ContentLength; Pos += Read) {
    Read = std::fread(&Json[Pos], 1, ContentLength - Pos, In);
    if (Read == 0)
      return failure();

    // If we're done, the error was transient. If we're not done, either it was
    // transient or we'll see it again on retry.
    clearerr(In);
    Pos += Read;
  }
  return success();
}

/// For lit tests we support a simplified syntax:
/// - messages are delimited by '// -----' on a line by itself
/// - lines starting with // are ignored.
/// This is a testing path, so favor simplicity over performance here.
/// When returning failure: feof(), ferror(), or shutdownRequested() will be
/// set.
LogicalResult
JSONTransportInputOverFile::readDelimitedMessage(std::string &Json) {
  Json.clear();
  llvm::SmallString<128> Line;
  while (succeeded(readLine(In, Line))) {
    StringRef LineRef = Line.str().trim();
    if (LineRef.starts_with("//")) {
      // Found a delimiter for the message.
      if (LineRef == "// -----")
```
- EN: This section centers on `clearerr`, `success`, `readDelimitedMessage` and parses input and converts raw data into structured form.
  CN: 这一段主要围绕 `clearerr`, `success`, `readDelimitedMessage` 等符号展开，负责解析输入并把原始数据转换成结构化形式。
- EN: In this range, the code iterates over collections, ranges, or records and checks conditions and handles edge cases.
  CN: 在这一段中，代码遍历集合、区间或记录，并检查条件并处理边界情况。

### Lines 361-369

```cpp
        break;
      continue;
    }

    Json += Line;
  }

  return failure(ferror(In));
}
```
- EN: This section centers on `failure` and implements the local control flow and bookkeeping.
  CN: 这一段主要围绕 `failure` 等符号展开，负责实现局部控制流程与状态维护。
- EN: This range returns the resulting value to its callers.
  CN: 这一段将结果返回给调用方。

## Key Concepts / 关键概念
- Domain / 领域: LLVM support utilities / LLVM 支撑工具
- Core symbols / 核心符号: `Reply`, `operator`, `assert`, `TransportLock` / 该文件围绕这些类型或函数组织主要逻辑。
- Data flow / 数据流: Parsing, decoding, and structural validation. / 重点关注解析、解码与结构校验。
- Error model / 错误模型: LLVM-style `Error`/`Expected` handling and invariant checks. / 使用 LLVM 风格的 `Error`/`Expected` 处理与不变量检查。
- Data structures / 数据结构: Relies on LLVM or STL containers for compact state management. / 依赖 LLVM 或 STL 容器管理紧凑状态。

## Dependencies / 依赖关系
- LLVM headers / LLVM 头文件: `llvm/Support/LSP/Transport.h`, `llvm/ADT/SmallString.h`, `llvm/Support/Error.h`, `llvm/Support/LSP/Logging.h`, `llvm/Support/LSP/Protocol.h`
- Standard library / 标准库: `atomic`, `optional`, `system_error`, `utility`
- Other/system headers / 其他或系统头文件: None / 无
- Related symbols / 相关符号: `Reply`, `operator`, `assert`, `TransportLock`, `onNotify`
