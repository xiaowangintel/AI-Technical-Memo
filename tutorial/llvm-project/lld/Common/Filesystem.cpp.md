# Filesystem.cpp — Code Analysis / 代码分析

## Source / 来源

- File / 文件: `lld/Common/Filesystem.cpp`
- Repository / 仓库: `llvm-project`
- Purpose / 用途: Filesystem.cpp This file contains a few utility functions to handle files.. It also sits in code that provides shared linker infrastructure used across LLD targets. / 该文件提供各类 LLD 目标共享的链接基础设施。 源码头部说明其职责是：Filesystem.cpp This file contains a few utility functions to handle files.。

## Line-by-Line Analysis / 逐行分析

### Lines 1-12

```cpp
//===- Filesystem.cpp -----------------------------------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file contains a few utility functions to handle files.
//
//===----------------------------------------------------------------------===//
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 13-25

```cpp
#include "lld/Common/Filesystem.h"
#include "lld/Common/ErrorHandler.h"
#include "llvm/Config/llvm-config.h"
#include "llvm/Support/FileOutputBuffer.h"
#include "llvm/Support/FileSystem.h"
#include "llvm/Support/Parallel.h"
#include "llvm/Support/Path.h"
#include "llvm/Support/TimeProfiler.h"
#if LLVM_ON_UNIX
#include <unistd.h>
#endif
#include <thread>
```

- EN: Pulls in 10 header(s) from local project, LLVM, system dependencies needed by this range. Uses conditional compilation to choose platform- or feature-specific code paths. Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里引入 10 个头文件，为本段提供本地模块、LLVM 或系统层面的依赖。这里使用条件编译，根据平台或特性切换不同代码路径。这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 26-43

```cpp
using namespace llvm;
using namespace lld;

// Removes a given file asynchronously. This is a performance hack,
// so remove this when operating systems are improved.
//
// On Linux (and probably on other Unix-like systems), unlink(2) is a
// noticeably slow system call. As of 2016, unlink takes 250
// milliseconds to remove a 1 GB file on ext4 filesystem on my machine.
//
// To create a new result file, we first remove existing file. So, if
// you repeatedly link a 1 GB program in a regular compile-link-debug
// cycle, every cycle wastes 250 milliseconds only to remove a file.
// Since LLD can link a 1 GB binary in about 5 seconds, that waste
// actually counts.
//
// This function spawns a background thread to remove the file.
// The calling thread returns almost immediately.
```

- EN: Works inside namespace scope `llvm`, `lld` to organize symbols. Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `llvm`, `lld`.
- CN: 这里位于命名空间 `llvm`, `lld` 中，用于组织符号作用域。这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `llvm`, `lld`。

### Lines 44-61

```cpp
void lld::unlinkAsync(StringRef path) {
  if (!sys::fs::exists(path) || !sys::fs::is_regular_file(path))
    return;

// Removing a file is async on windows.
#if defined(_WIN32)
  // On Windows co-operative programs can be expected to open LLD's
  // output in FILE_SHARE_DELETE mode. This allows us to delete the
  // file (by moving it to a temporary filename and then deleting
  // it) so that we can link another output file that overwrites
  // the existing file, even if the current file is in use.
  //
  // This is done on a best effort basis - we do not error if the
  // operation fails. The consequence is merely that the user
  // experiences an inconvenient work-flow.
  //
  // The code here allows LLD to work on all versions of Windows.
  // However, at Windows 10 1903 it seems that the behavior of
```

- EN: Uses conditional compilation to choose platform- or feature-specific code paths. Declares or implements routines including `unlinkAsync`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `unlinkAsync`.
- CN: 这里使用条件编译，根据平台或特性切换不同代码路径。这里声明或实现函数，例如 `unlinkAsync`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `unlinkAsync`。

### Lines 62-79

```cpp
  // Windows has changed, so that we could simply delete the output
  // file. This code should be simplified once support for older
  // versions of Windows is dropped.
  //
  // Warning: It seems that the WINVER and _WIN32_WINNT preprocessor
  // defines affect the behavior of the Windows versions of the calls
  // we are using here. If this code stops working this is worth
  // bearing in mind.
  SmallString<128> tmpName;
  if (!sys::fs::createUniqueFile(path + "%%%%%%%%.tmp", tmpName)) {
    if (!sys::fs::rename(path, tmpName))
      path = tmpName;
    else
      sys::fs::remove(tmpName);
  }
  sys::fs::remove(path);
#else
  if (parallel::strategy.ThreadsRequested == 1)
```

- EN: Uses conditional compilation to choose platform- or feature-specific code paths. Declares or implements routines including `remove`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `remove`.
- CN: 这里使用条件编译，根据平台或特性切换不同代码路径。这里声明或实现函数，例如 `remove`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `remove`。

### Lines 80-89

```cpp
    return;

  // We cannot just remove path from a different thread because we are now going
  // to create path as a new file.
  // Instead we open the file and unlink it on this thread. The unlink is fast
  // since the open fd guarantees that it is not removing the last reference.
  int fd;
  std::error_code ec = sys::fs::openFileForRead(path, fd);
  sys::fs::remove(path);
```

- EN: Declares or implements routines including `openFileForRead`, `remove`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `openFileForRead`, `remove`.
- CN: 这里声明或实现函数，例如 `openFileForRead`, `remove`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `openFileForRead`, `remove`。

### Lines 90-105

```cpp
  if (ec)
    return;

  // close and therefore remove TempPath in background.
  std::mutex m;
  std::condition_variable cv;
  bool started = false;
  std::thread([&, fd] {
    {
      std::lock_guard<std::mutex> l(m);
      started = true;
      cv.notify_all();
    }
    ::close(fd);
  }).detach();
```

- EN: Declares or implements routines including `l`, `close`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `l`, `close`.
- CN: 这里声明或实现函数，例如 `l`, `close`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `l`, `close`。

### Lines 106-123

```cpp
  // GLIBC 2.26 and earlier have race condition that crashes an entire process
  // if the main thread calls exit(2) while other thread is starting up.
  std::unique_lock<std::mutex> l(m);
  cv.wait(l, [&] { return started; });
#endif
}

// Simulate file creation to see if Path is writable.
//
// Determining whether a file is writable or not is amazingly hard,
// and after all the only reliable way of doing that is to actually
// create a file. But we don't want to do that in this function
// because LLD shouldn't update any file if it will end in a failure.
// We also don't want to reimplement heuristics to determine if a
// file is writable. So we'll let FileOutputBuffer do the work.
//
// FileOutputBuffer doesn't touch a destination file until commit()
// is called. We use that class without calling commit() to predict
```

- EN: Uses conditional compilation to choose platform- or feature-specific code paths. Introduces type definitions such as `without`. Declares or implements routines including `l`. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里使用条件编译，根据平台或特性切换不同代码路径。这里引入类型定义，例如 `without`。这里声明或实现函数，例如 `l`。这里的注释记录了设计假设、不变量或使用说明。

### Lines 124-133

```cpp
// if the given file is writable.
std::error_code lld::tryCreateFile(StringRef path) {
  llvm::TimeTraceScope timeScope("Try create output file");
  if (path.empty())
    return std::error_code();
  if (path == "-")
    return std::error_code();
  return errorToErrorCode(FileOutputBuffer::create(path, 1).takeError());
}
```

- EN: Declares or implements routines including `tryCreateFile`, `timeScope`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `tryCreateFile`, `timeScope`.
- CN: 这里声明或实现函数，例如 `tryCreateFile`, `timeScope`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `tryCreateFile`, `timeScope`。

### Lines 134-145

```cpp
// Creates an empty file to and returns a raw_fd_ostream to write to it.
std::unique_ptr<raw_fd_ostream> lld::openFile(StringRef file) {
  std::error_code ec;
  auto ret =
      std::make_unique<raw_fd_ostream>(file, ec, sys::fs::OpenFlags::OF_None);
  if (ec) {
    error("cannot open " + file + ": " + ec.message());
    return nullptr;
  }
  return ret;
}
```

- EN: Declares or implements routines including `openFile`, `error`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `openFile`, `error`.
- CN: 这里声明或实现函数，例如 `openFile`, `error`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `openFile`, `error`。

### Lines 146-157

```cpp
// The merged bitcode after LTO is large. Try opening a file stream that
// supports reading, seeking and writing. Such a file allows BitcodeWriter to
// flush buffered data to reduce memory consumption. If this fails, open a file
// stream that supports only write.
std::unique_ptr<raw_fd_ostream> lld::openLTOOutputFile(StringRef file) {
  std::error_code ec;
  std::unique_ptr<raw_fd_ostream> fs =
      std::make_unique<raw_fd_stream>(file, ec);
  if (!ec)
    return fs;
  return openFile(file);
}
```

- EN: Declares or implements routines including `openLTOOutputFile`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `openLTOOutputFile`.
- CN: 这里声明或实现函数，例如 `openLTOOutputFile`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `openLTOOutputFile`。

## Key Concepts / 关键概念

- `without`: class or struct interface / 类或结构体接口
- `unlinkAsync`: function or method entry point / 函数或方法入口
- `remove`: function or method entry point / 函数或方法入口
- `openFileForRead`: function or method entry point / 函数或方法入口
- `l`: function or method entry point / 函数或方法入口
- `close`: function or method entry point / 函数或方法入口
- `llvm`: namespace scope / 命名空间作用域
- `lld`: namespace scope / 命名空间作用域

## Dependencies / 依赖关系

- Local headers / 本地头文件: `lld/Common/Filesystem.h`, `lld/Common/ErrorHandler.h`
- LLVM headers / LLVM 头文件: `llvm/Config/llvm-config.h`, `llvm/Support/FileOutputBuffer.h`, `llvm/Support/FileSystem.h`, `llvm/Support/Parallel.h`, `llvm/Support/Path.h`, `llvm/Support/TimeProfiler.h`
- System headers / 系统头文件: `unistd.h`, `thread`
- Directory context / 目录上下文: `lld/Common` neighbors usually cooperate with this file to provide the surrounding subsystem / `lld/Common` 下的相邻文件通常与本文件协作组成对应子系统
